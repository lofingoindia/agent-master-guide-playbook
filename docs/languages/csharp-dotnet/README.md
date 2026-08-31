# C# and .NET Agent Engineering

> **Status:** Production engineering reference
> **Last researched:** 2026-08-31
> **Runtime baseline:** .NET 10 LTS, C# 14
> **Research packet:** [C#/.NET agent engineering deep dive](../../research/packets/csharp-dotnet-agent-engineering-deep-dive.md)

This area explains how to build agent runtimes that remain bounded, observable, recoverable, and secure under real failures. It is not a framework tutorial. The center of gravity is the .NET runtime and hosting model: every run has one owner, every wait has a deadline, every buffer has a limit, and every side effect has a durable identity.

.NET 10 is the production baseline because it is the current LTS line. Preview .NET 11 behavior is deliberately excluded unless a guide explicitly labels it. Package maturity is assessed separately from framework maturity: a GA runtime can host prerelease agent integrations.

## Guide map

| Need | Guide | Principal decision |
|---|---|---|
| Own tasks and scopes | [Architecture, task ownership, and hosting](01-architecture-task-ownership-and-hosting.md) | A run owns all child work and awaits it before scope disposal |
| Bound time and stop safely | [Cancellation, timeouts, and graceful shutdown](02-cancellation-timeouts-and-graceful-shutdown.md) | Distinguish canceling work from canceling a wait |
| Stream without memory growth | [Channels, streaming, and backpressure](03-channels-streaming-and-backpressure.md) | Bound both item count and retained bytes |
| Call providers reliably | [HTTP clients, pooling, and resilience](04-http-clients-pooling-and-resilience.md) | Reuse transports and budget retries once |
| Execute tools safely | [Tools, processes, and sandboxing](05-tools-processes-and-sandboxing.md) | A child process is not a security sandbox |
| Enforce contracts | [Schemas, JSON, and structured output](06-schemas-json-and-structured-output.md) | Schema validity is not business validity or authorization |
| Recover without duplicate effects | [Exceptions, retries, idempotency, and effects](07-exceptions-retries-idempotency-and-effects.md) | Retry only classified, replay-safe operations |
| Survive restarts | [Queues, state, and durable workers](08-queues-state-and-durable-workers.md) | Persist intent before claiming success |
| Design memory and protect the process | [Memory, resources, and admission control](09-memory-resources-and-admission-control.md) | Keep model context, durable state, and long-term memory separate; admit by weighted cost |
| See what failed | [Telemetry, profiling, and debugging](10-telemetry-profiling-and-debugging.md) | Correlate runs, attempts, calls, and effects without leaking content |
| Prove concurrency behavior | [Testing, load, and concurrency](11-testing-load-and-concurrency.md) | Test slow consumers, cancellation races, and ambiguous outcomes |
| Control dependencies | [NuGet, supply chain, and build](12-nuget-supply-chain-and-build.md) | Lock applications and review build-transitive code |
| Operate at scale | [Deployment, hosting, and scaling](13-deployment-hosting-and-scaling.md) | Readiness, drain, leases, and resource limits form one contract |
| Choose ecosystem components | [Agent SDKs, Azure, MCP, and durable runtimes](14-agent-sdks-azure-mcp-and-durable-runtimes.md) | Select per capability and maturity, not by umbrella brand |
| Avoid recurring failures | [Agent anti-patterns and readiness checklist](15-agent-anti-patterns-and-readiness-checklist.md) | Ship only when limits and recovery are demonstrable |

## From one bounded loop to production

Do not begin with parallel agents, durable orchestration, or long-term memory. Add each only when the preceding boundary is measurable.

| Increment | Add | Evidence required before the next increment |
|---|---|---|
| 1. Bounded local loop | One provider adapter, sequential tools, total deadline, maximum model turns/tool calls, strict schemas | Tests prove terminal success, refusal/incomplete output, cancellation, and exhausted budgets |
| 2. Hosted service | Generic Host, per-run DI scope, bounded channel, weighted admission, readiness and drain | Slow consumers plateau in memory; every child task and scope is joined during shutdown |
| 3. Restart-safe effects | Versioned run state/events, durable queue, effect intent/receipt, idempotency and reconciliation | Crash injection at each persistence/network edge neither loses a run nor repeats a business effect |
| 4. Operated system | Externalized memory, SLOs, traces/metrics/logs, behavioral evals, load and published-artifact tests | Release gates cover reliability, quality, safety, cost, compatibility, and rollback |

A framework may accelerate an increment, but it does not waive its evidence gate. A short read-only request often needs only increments 1 and 2; a money-moving or infrastructure-changing agent normally needs all four.

## Reference architecture

~~~mermaid
flowchart LR
    Edge[Authenticated API or queue] --> Admit[Admission and policy]
    Admit --> Run[Run owner]
    Run --> Model[Provider adapter]
    Run --> Tools[Tool dispatcher]
    Run --> State[State and effect journal]
    Model --> Stream[Bounded stream pump]
    Tools --> Sandbox[Scoped process or remote worker]
    State --> Durable[Durable queue or workflow]
    Run --> Obs[Traces metrics logs]
    Admit -. overload .-> Reject[Reject defer or degrade]
~~~

The run owner is the architectural unit. It owns the dependency-injection scope, linked cancellation source, child tasks, stream pumps, tool-call budget, state transition, and terminal result. Provider SDK objects may be long-lived and shared when their documentation permits it; mutable request options, sessions, database contexts, process handles, and response streams normally belong to a run or attempt.

Use the application-owned [agent state and event contract](../../runtime/agent-state-and-event-contracts.md) for stable run identity, fenced transitions, event ordering, replay, terminal outcomes, reconnect, and effect correlation across ASP.NET hosts, background services, queues, streams, and durable runtimes.

## Production baseline

1. Use <code>Host.CreateApplicationBuilder</code>, async APIs, and one explicit scope per queued work item.
2. Pass a caller cancellation token into the underlying operation. Add total and per-attempt deadlines; do not mistake <code>WaitAsync</code> for cancellation of the work.
3. Reuse <code>HttpClient</code> transports. Configure DNS rotation through handler lifetime or <code>PooledConnectionLifetime</code>.
4. Use bounded channels. Capacity in items is insufficient for variable-size model events, so account for bytes as well.
5. Treat model output, tool arguments, MCP content, retrieved documents, and provider metadata as untrusted input.
6. Make side effects idempotent with stable effect identifiers and durable receipts. Never infer failure from a lost acknowledgment.
7. Stop admission before shutdown, drain within a budget, then cancel and escalate. Assume graceful callbacks may never run.
8. Record safe identifiers and timing by default. Prompt, response, tool argument, and tool result capture must be an explicit data-governance decision.
9. Pin SDK and NuGet versions. A stable package can expose experimental APIs, and a GA framework can depend on prerelease integration packages.
10. Test the published artifact in its real container and architecture, especially when trimming or Native AOT is enabled.
11. Treat the model context as a bounded derived view. Keep session continuity, authoritative run/effect state, and long-term semantic memory in separately governed stores.
12. Release against task-specific evaluations and operational SLOs, not a single demo transcript or average latency.

## A minimal ownership model

~~~csharp
public sealed class AgentRunService(IServiceScopeFactory scopes)
{
    public async Task<RunResult> RunAsync(
        RunRequest request,
        CancellationToken callerToken)
    {
        await using AsyncServiceScope scope = scopes.CreateAsyncScope();
        var runner = scope.ServiceProvider.GetRequiredService<ScopedRunner>();

        using var deadline = new CancellationTokenSource(request.TotalBudget);
        using var run = CancellationTokenSource.CreateLinkedTokenSource(
            callerToken, deadline.Token);

        return await runner.ExecuteAsync(request, run.Token);
    }
}
~~~

This shape is intentionally small. Production code adds admission, persistent effect records, attempt policy, telemetry, and terminal-state persistence around it; those concerns should not erase the ownership boundary.

## Selected primary sources

- [.NET support policy](https://dotnet.microsoft.com/en-us/platform/support/policy)
- [What is new in C# 14](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-14)
- [.NET Generic Host](https://learn.microsoft.com/en-us/dotnet/core/extensions/generic-host)
- [Task asynchronous programming model](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
- [OpenTelemetry in .NET](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/observability-with-otel)
- [Official OpenAI .NET library](https://github.com/openai/openai-dotnet)
- [Official Anthropic C# SDK](https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/csharp)
- [Official MCP C# SDK](https://github.com/modelcontextprotocol/csharp-sdk)
- [Microsoft Agent Framework documentation](https://learn.microsoft.com/en-us/agent-framework/get-started/)

# Architecture, Task Ownership, and Hosting

> **Last researched:** 2026-08-31
> **Applies to:** .NET 10 LTS, C# 14

The most reliable .NET agent architecture is a supervised task tree. An inbound request, dequeued message, or durable activation creates one run owner. That owner creates the scope, starts child work, observes every completion, and persists one terminal state before releasing resources.

## Ownership graph

~~~mermaid
flowchart TD
    Host[Generic Host] --> Worker[BackgroundService singleton]
    Worker --> Item[One dequeued work item]
    Item --> Scope[Async DI scope]
    Scope --> Run[Run owner]
    Run --> Model[Model attempt]
    Run --> Pump[Stream pump]
    Run --> Tool[Tool calls]
    Run --> Save[State and effect commits]
    Run --> Join[Await all children]
    Join --> Dispose[Dispose scope]
~~~

If any arrow bypasses the run owner, shutdown, exception observation, and scope disposal become probabilistic.

## Task lifetime is the contract

In the task-based asynchronous pattern, the returned <code>Task</code> represents the complete operation. A method that starts background work and returns success early has changed the contract, even if its name still ends in <code>Async</code>.

- Return <code>Task</code> or <code>Task&lt;T&gt;</code> for work with meaningful completion.
- Use <code>ValueTask</code> only where measurement shows synchronous completion is common and the caller follows its single-consumption rules.
- Reserve <code>async void</code> for event handlers. It cannot be awaited and routes exceptions outside the normal task chain.
- Avoid <code>Task.Run</code> around naturally asynchronous network or storage calls. It adds scheduling without adding nonblocking behavior.
- Do not retain mutable per-run state in singleton services.

Structured ownership does not require a new framework. A <code>RunContext</code> carrying stable IDs, budgets, a token, and safe telemetry fields is often enough.

## First bounded loop

Start with a sequential, application-owned loop. The provider adapter normalizes provider terminal states; the dispatcher owns tool authorization, per-tool deadlines, output limits, and effects. The loop owns the total budget and cannot silently continue forever.

~~~csharp
public sealed record LoopBudget(
    int MaxModelTurns,
    int MaxToolCalls,
    TimeSpan TotalTime);

public sealed class AgentLoop(IAgentModel model, IToolDispatcher tools)
{
    public async Task<ModelReply> RunAsync(
        List<AgentMessage> history,
        LoopBudget budget,
        CancellationToken callerToken)
    {
        ArgumentOutOfRangeException.ThrowIfLessThan(budget.MaxModelTurns, 1);
        ArgumentOutOfRangeException.ThrowIfNegative(budget.MaxToolCalls);
        if (budget.TotalTime <= TimeSpan.Zero)
            throw new ArgumentOutOfRangeException(nameof(budget.TotalTime));

        using var deadline = new CancellationTokenSource(budget.TotalTime);
        using var run = CancellationTokenSource.CreateLinkedTokenSource(
            callerToken, deadline.Token);

        int toolCallsUsed = 0;
        for (int turn = 0; turn < budget.MaxModelTurns; turn++)
        {
            ModelReply reply = await model.CompleteAsync(history, run.Token);
            if (reply.IsTerminal)
            {
                return reply; // success, refusal, or incomplete are distinct states
            }

            if (reply.ToolCalls.Count == 0)
                throw new AgentProtocolException("Nonterminal reply contained no tool calls.");

            if (reply.ToolCalls.Count > budget.MaxToolCalls - toolCallsUsed)
            {
                throw new AgentBudgetExceededException("Tool-call budget exhausted.");
            }

            history.Add(reply.AssistantMessage);
            foreach (ToolCall call in reply.ToolCalls) // sequential by design
            {
                ToolResult result = await tools.ExecuteAsync(call, run.Token);
                history.Add(result.AsMessage(call.Id));
                toolCallsUsed++;
            }
        }

        throw new AgentBudgetExceededException("Model-turn budget exhausted.");
    }
}
~~~

This is a contract sketch, not a provider SDK sample. Reject invalid budgets before the call, bound history/tool-result bytes in the adapter, and persist a terminal result outside the loop when restart survival matters. Add parallel tools only after reserving their combined resources and defining sibling-failure semantics.

## Host and worker boundaries

<code>Host.CreateApplicationBuilder</code> is the recommended starting point for new Generic Host applications. A hosted service is registered as a singleton. Therefore, a <code>BackgroundService</code> must create a scope for each unit of work when it consumes scoped services such as an EF Core <code>DbContext</code>.

<code>StartAsync</code> calls are sequential. Keep startup validation and registration short; defer the long-running loop to <code>ExecuteAsync</code>. In .NET 10, the entire <code>BackgroundService.ExecuteAsync</code> method runs on a background thread; code before its first <code>await</code> no longer blocks other hosted services from starting. Put true startup gates in the constructor, <code>StartAsync</code>, <code>IHostedLifecycleService</code>, or a direct <code>IHostedService</code> implementation. The task returned by <code>ExecuteAsync</code> is still the service lifetime, so do not detach its loop.

~~~csharp
public sealed class AgentWorker(
    WorkQueue queue,
    IServiceScopeFactory scopeFactory,
    ILogger<AgentWorker> logger) : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        await foreach (WorkEnvelope item in queue.ReadAllAsync(stoppingToken))
        {
            await using AsyncServiceScope scope = scopeFactory.CreateAsyncScope();
            var handler = scope.ServiceProvider.GetRequiredService<RunHandler>();

            try
            {
                await handler.HandleAsync(item, stoppingToken);
            }
            catch (OperationCanceledException) when (stoppingToken.IsCancellationRequested)
            {
                break;
            }
            catch (Exception error)
            {
                logger.LogError(error, "Run {RunId} failed", item.RunId);
            }
        }
    }
}
~~~

This serial example is safe but not sufficient for throughput. Add a bounded worker pool only after defining admission, per-tenant fairness, resource weights, and shutdown joins.

## Dependency-injection rules

| Object | Typical lifetime | Reason |
|---|---|---|
| Provider client / transport | Singleton | Connection reuse and centralized policy |
| Immutable tool catalog | Singleton | Shared read-only metadata |
| Run coordinator | Scoped or explicitly constructed | Owns mutable run state |
| <code>DbContext</code> | Scoped per unit of work | Not thread-safe |
| Request options and message list | Per run or per attempt | Usually mutable |
| Child process and response stream | Per tool call / attempt | Must be disposed by owner |

The built-in container disposes objects it creates. Do not dispose a resolved singleton or scoped service manually. Conversely, resolving disposable transient or scoped services from the root container can retain them until host shutdown. Validate scopes in development and tests.

Scopes are not hierarchical isolation domains. Creating a child scope does not make a parent scoped service safe to use concurrently.

## Concurrency policy

Parallelism is permitted only when all of these are true:

1. The operations are semantically independent.
2. Each operation has separate mutable state and non-thread-safe dependencies.
3. The combined memory, token, provider-rate, and tool-effect budget is reserved first.
4. Failure semantics define whether siblings continue or are canceled.
5. Every child task is joined and its exception observed.

<code>Task.WhenAll</code> is suitable for a fixed set of owned children. Awaiting it rethrows an exception, while the returned task can retain multiple failures in <code>Task.Exception</code>. Log individual child outcomes rather than assuming the first thrown exception is the whole failure.

## Failure patterns

- **Fire-and-forget telemetry or persistence:** process exit loses it and exceptions are unobserved. Use a bounded owned exporter/queue.
- **Scoped service captured by singleton:** state leaks across runs or is disposed late.
- **Shared mutable SDK options:** concurrent calls race. Treat request options as per-call unless the SDK guarantees immutability.
- **Concurrent use of one <code>DbContext</code>:** can throw or corrupt state. Create one context per parallel unit.
- **Returning before tool children finish:** authorization scope and cancellation lifetime end too soon.
- **One global semaphore:** creates head-of-line blocking and no tenant fairness. Partition admission where necessary.

## Review checklist

- [ ] One component owns each run and every child task.
- [ ] No production <code>async void</code> outside event handlers.
- [ ] A scope is created and disposed per queue item or run.
- [ ] Singleton state is immutable or explicitly thread-safe.
- [ ] Parallel children reserve resources and have a sibling-failure policy.
- [ ] Terminal state is written once after all required work is joined.

## Primary sources

- [Task asynchronous programming model](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
- [Task-based asynchronous pattern](https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap)
- [.NET Generic Host](https://learn.microsoft.com/en-us/dotnet/core/extensions/generic-host)
- [Hosted services in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-10.0)
- [.NET 10 BackgroundService startup behavior change](https://learn.microsoft.com/en-us/dotnet/core/compatibility/extensions/10.0/backgroundservice-executeasync-task)
- [Use scoped services within a BackgroundService](https://learn.microsoft.com/en-us/dotnet/core/extensions/scoped-service)
- [Dependency injection guidelines](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines)
- [EF Core DbContext configuration](https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/)

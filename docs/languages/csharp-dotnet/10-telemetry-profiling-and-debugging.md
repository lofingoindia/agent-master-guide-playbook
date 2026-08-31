# Telemetry, Profiling, and Debugging

> **Last researched:** 2026-08-31
> **Applies to:** .NET 10 LTS, C# 14

Agent observability must explain one logical run across provider attempts, streaming, tool calls, durable activations, and external effects. It must do so without turning prompts, documents, or secrets into a second ungoverned data store.

## Signal model

| Signal | Use | Avoid |
|---|---|---|
| Traces | Causality and latency across run steps | Full prompt/result capture by default |
| Metrics | Rates, saturation, queueing, budgets | Run IDs or user IDs as labels |
| Logs | Discrete decisions, receipts, failures | Duplicating every streamed token |
| Profiles/dumps | Runtime root cause | Routine collection without access controls |

Use .NET-native instrumentation APIs: <code>ActivitySource</code> for traces, <code>Meter</code> for metrics, and <code>ILogger</code> for logs. OpenTelemetry can export these signals to an OTLP-compatible backend or Azure Monitor without forcing domain code to depend on one vendor.

## Trace shape

~~~mermaid
flowchart TD
    R[agent.run] --> A[agent.attempt]
    A --> P[model.request]
    A --> T1[tool.call]
    A --> T2[tool.call]
    T1 --> E1[effect.commit]
    T2 --> E2[effect.reconcile]
    R --> S[state.transition]
    R --> Q[queue.settle]
~~~

Create spans for meaningful ownership and remote boundaries, not every token. Link a resumed durable activation to the originating trace when a single parent relationship would be misleading.

Useful span fields:

- stable run, attempt, step, message, and effect identifiers;
- provider, endpoint class, model/deployment, and operation;
- queue wait, admission wait, time to first byte, stream duration;
- input/output token counts and bounded byte counts;
- retry reason and attempt count;
- tool name/version, effect class, sandbox mode, and terminal status;
- schema version and state transition.

Do not put high-cardinality identifiers in metric labels. They belong in spans/logs.

## Generative AI semantic conventions

OpenTelemetry generative-AI semantic conventions are evolving and have moved into their own specification area. Pin the convention version used by dashboards and exporters. Gate experimental attributes behind configuration, and maintain a small internal semantic layer so a convention rename does not rewrite the runtime.

Provider and Agent Framework instrumentation may overlap with custom spans. Detect and prevent double-counting model calls and token metrics.

## Content and privacy policy

Default to metadata-only telemetry. Prompts, responses, retrieved content, tool arguments, and results can contain credentials, personal data, regulated records, and prompt-injected instructions.

If content capture is justified:

1. define purpose, owner, retention, and access;
2. sample independently from normal telemetry;
3. redact before export;
4. cap bytes and mark truncation;
5. encrypt and restrict the destination;
6. support tenant deletion and incident response;
7. never record secrets or raw authorization headers.

Hashing does not make low-entropy identifiers anonymous.

## Metrics that reveal saturation

- admitted, rejected, queued, active, completed runs;
- admission and queue wait distributions;
- active provider calls and tool processes;
- retained stream bytes and dropped/coalesced events;
- logical calls versus network attempts;
- timeout, cancellation, throttling, and ambiguous-effect rates;
- thread-pool queue length/thread count, CPU, GC pause, heap/LOH, allocation rate;
- process RSS, file descriptors/handles, sockets, disk, and child-process count;
- token and cost budget consumption by bounded business dimensions.

Alert on user-visible symptoms and saturation together. A latency alert without queue, retry, and resource context is hard to act on.

## SLOs make telemetry actionable

Define service-level indicators from the user's contract, then set objectives from product need and measured capability. Do not use "HTTP 200" or "model stream closed" as agent success.

| Objective | Example indicator | Important segmentation |
|---|---|---|
| Admission | Proportion receiving a valid admit/defer/reject decision within budget | tenant tier, interactive/batch; report overload separately so rejection cannot game availability |
| Run correctness | Admitted runs reaching the correct explicit terminal state, excluding caller cancellation by policy | workflow/tool/effect class and release version |
| End-to-end latency | Admit-to-terminal duration; time to first useful output as a separate SLI | result class, model route, queue/admission time |
| Recovery | Interrupted durable runs resumed within RTO; age/count of unknown effects | state store, queue, dependency, deployment |
| Stream continuity | Connections completing or resuming without missing durable semantic events | transport/client class, not individual connection ID |

Quality, safety, and cost need release/canary objectives too, but usually come from evaluated task samples rather than infrastructure availability counters. Track refusal, tool-selection accuracy, policy violations, task success, tokens, and cost by bounded workload dimensions.

Create error-budget burn alerts for fast and slow windows, then attach a runbook that names the first saturation signals, trace queries, safe degradation, reconciliation queue, rollback condition, and owner. Dashboards are not the runbook. Keep the SLO definition versioned when terminal-state or denominator policy changes.

## Diagnostic ladder

1. **Metrics:** use runtime counters to identify GC, thread pool, exception, or CPU symptoms.
2. **Trace:** collect EventPipe or OpenTelemetry traces for causality and stacks.
3. **Profile:** use CPU sampling or allocation tools for focused periods.
4. **GC dump:** inspect managed heap composition; collection can trigger a full generation-2 GC and pause.
5. **Process dump:** use for deadlocks/crashes with strict access and retention.

EventPipe supports cross-platform runtime diagnostics. <code>dotnet-counters</code>, <code>dotnet-trace</code>, <code>dotnet-gcdump</code>, and <code>dotnet-dump</code> should be practiced in staging before an incident.

## Diagnostic endpoint security

The .NET diagnostics channel can allow traces, dumps, and sensitive runtime inspection. Protect diagnostic sockets/ports with filesystem permissions and workload isolation. If untrusted code can run under the same identity, consider disabling diagnostics with <code>DOTNET_EnableDiagnostics=0</code>, accepting that live diagnostic capabilities are then unavailable.

## Failure patterns

- Prompts and tool results are logged at information level.
- Run ID is a metric dimension and explodes cardinality.
- SDK and custom instrumentation both count one request.
- Sampling removes every failed or long-running trace.
- A dump is uploaded to a broad-access ticket.
- Telemetry export blocks shutdown indefinitely.
- Only managed heap is watched while native memory or child tools exhaust the container.

## Review checklist

- [ ] Runs, attempts, tool calls, effects, and durable resumes correlate.
- [ ] Metrics use bounded dimensions.
- [ ] Content capture is opt-in, redacted, capped, and governed.
- [ ] Semantic-convention version is pinned.
- [ ] Runtime saturation and application outcomes are both measured.
- [ ] SLOs use explicit terminal semantics and separately expose rejection, caller cancellation, and unknown effects.
- [ ] Error-budget alerts link to tested runbooks and safe degradation/rollback actions.
- [ ] On-call staff have tested trace/dump procedures.
- [ ] Diagnostic access and artifact retention are security-controlled.

## Primary sources

- [OpenTelemetry .NET documentation](https://opentelemetry.io/docs/languages/dotnet/)
- [OpenTelemetry traces for .NET](https://opentelemetry.io/docs/languages/dotnet/traces/)
- [Generative AI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
- [Observability with OpenTelemetry in .NET](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/observability-with-otel)
- [.NET runtime metrics](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/built-in-metrics-runtime)
- [.NET System.Net HTTP metrics](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/built-in-metrics-system-net)
- [Reliability metrics, SLIs, and SLOs](https://learn.microsoft.com/en-us/azure/well-architected/reliability/metrics)
- [EventPipe](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/eventpipe)
- [.NET diagnostic ports](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/diagnostic-port)
- [dotnet-gcdump](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-gcdump)
- [dotnet-dump](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-dump)

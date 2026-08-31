# Research Packet: C#/.NET Agent Engineering Deep Dive

> **Research date:** 2026-08-31
> **Scope:** Current production behavior of .NET/C# runtime primitives, hosting, networking, serialization, diagnostics, build/deployment, official agent/provider SDKs, MCP, Azure integrations, and durable execution
> **Output:** [C# and .NET Agent Engineering](../../languages/csharp-dotnet/README.md)

## Research question

What must a production .NET agent runtime own itself, which behaviors can it safely delegate to current framework/SDK components, and where do default settings create hidden amplification, unbounded resource use, unsafe effects, or upgrade risk?

## Method

Research prioritized:

1. Microsoft Learn documentation for .NET 10, ASP.NET Core 10, Azure, NuGet, and Agent Framework;
2. official source repositories and release pages for .NET runtime/extensions, OpenAI, Anthropic, MCP, Microsoft Agent Framework, Temporal, Dapr, Azure SDKs, and .NET containers;
3. protocol/specification documentation for MCP and OpenTelemetry;
4. official migration, compatibility, and release notes for current behavior;
5. official NuGet V3 package metadata for the observed stable version lines;
6. source files where high-level documentation obscured defaults.

Important claims were cross-checked across conceptual documentation, API documentation, source, or release notes. Community tutorials and generic agent advice were excluded from the evidence base. Versions below are observations at the research date and should not substitute for a locked dependency review.

## Executive synthesis

The strongest production design is a small supervised .NET runtime around provider/framework adapters:

- one run owner owns the DI scope, token hierarchy, child tasks, stream pumps, resource leases, tool effects, and terminal commit;
- cancellation of work and cancellation of a wait are different operations;
- asynchronous streaming still requires item, byte, time, and event bounds;
- <code>HttpClient</code> lifetime and retry composition are system-level choices, not per-SDK details;
- provider structured output validates syntax/shape, not domain truth, tenant ownership, or authorization;
- child processes provide lifecycle isolation, not a security sandbox;
- queue delivery and durable workflow replay are at-least-once at external-effect boundaries;
- model context, session continuity, authoritative operational state, and long-term semantic memory require separate contracts;
- admission must be weighted by bytes, tokens, processes, and dependency partitions rather than request count alone;
- telemetry must correlate logical work and physical attempts without recording sensitive model/tool content by default;
- operational SLOs and task-specific evaluations are separate release signals and both are required;
- package stability, API stability, integration maturity, and runtime maturity are independent;
- Native AOT should be selected after dependency-graph evidence, not as a default optimization.

No researched SDK or framework removes the need for run ownership, effect idempotency, authorization, resource admission, state evolution, or deployment verification.

## Current baseline

| Area | Finding at 2026-08-31 | Consequence |
|---|---|---|
| .NET | .NET 10 is current LTS; support page lists patch 10.0.11 released 2026-08-11 and EOL 2028-11-14 | Use .NET 10 for production baseline |
| .NET 11 | Preview, planned release in November 2026 | Exclude preview behavior from production guidance |
| C# | C# 14 is current for .NET 10 | Examples may use current language syntax without preview features |
| System.Text.Json | .NET 10 includes <code>JsonSerializerOptions.Strict</code>, duplicate-property controls, and additional streaming improvements | Strict parsing can be configured centrally |
| Process | .NET 10 adds Windows process-group support | Better signaling, but still platform-specific and not a sandbox |
| BackgroundService | In .NET 10, all of <code>ExecuteAsync</code> runs on a background thread | Startup gates must use constructor, <code>StartAsync</code>, <code>IHostedLifecycleService</code>, or direct <code>IHostedService</code> |
| Testing | .NET 10 <code>dotnet test</code> has explicit Microsoft Testing Platform support | Choose MTP or VSTest deliberately |
| Containers | Official .NET images include standard, chiseled/distroless, and AOT-oriented variants | Image capability is part of tool/runtime design |

Primary baseline:

- [.NET support policy](https://dotnet.microsoft.com/en-us/platform/support/policy)
- [.NET releases](https://github.com/dotnet/core/blob/main/releases.md)
- [What is new in .NET 10](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/overview)
- [What is new in C# 14](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-14)

## Runtime and task ownership findings

### Task contract

The task-based asynchronous pattern defines the returned task as the lifetime of the operation. Production consequences:

- a method that starts work and returns before it finishes has detached ownership;
- <code>async void</code> is unsuitable outside event handlers;
- <code>Task.Run</code> does not improve naturally asynchronous I/O;
- all children must be awaited or transferred to an explicit supervisor;
- <code>Task.WhenAll</code> should be inspected for all child failures when diagnosis requires more than the first rethrown exception.

The core <code>Task</code> source reinforces that completion, exception, cancellation, continuations, and disposal are properties of the task object. A separate ad hoc lifecycle is a source of inconsistency.

Sources:

- [Task asynchronous programming model](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
- [Task-based asynchronous pattern](https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap)
- [Task runtime source](https://github.com/dotnet/runtime/blob/main/src/libraries/System.Private.CoreLib/src/System/Threading/Tasks/Task.cs)

### Generic Host and DI

<code>Host.CreateApplicationBuilder</code> is the recommended constructor for new Generic Host apps. Hosted services are singleton registrations. <code>StartAsync</code> is sequential, and <code>ExecuteAsync</code> represents the background service lifetime. A .NET 10 breaking change moved the entire <code>BackgroundService.ExecuteAsync</code> body to a background thread; its pre-first-<code>await</code> portion no longer gates startup. Required startup work belongs in the constructor, <code>StartAsync</code>, <code>IHostedLifecycleService</code>, or a direct <code>IHostedService</code>.

The critical operational mismatch is singleton workers consuming scoped dependencies. The worker must create a scope for each run/work item. EF Core <code>DbContext</code> is a short unit-of-work and is not thread-safe; parallel branches require separate contexts/scopes.

The built-in DI container owns disposal of objects it creates. Resolving disposable transient/scoped services from the root can retain them until shutdown. Scopes are not hierarchical concurrency boundaries.

Sources:

- [.NET Generic Host](https://learn.microsoft.com/en-us/dotnet/core/extensions/generic-host)
- [Hosted services](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-10.0)
- [.NET 10 BackgroundService behavior change](https://learn.microsoft.com/en-us/dotnet/core/compatibility/extensions/10.0/backgroundservice-executeasync-task)
- [Scoped services in BackgroundService](https://learn.microsoft.com/en-us/dotnet/core/extensions/scoped-service)
- [Dependency injection guidelines](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines)
- [EF Core DbContext configuration](https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/)

## Cancellation, timeout, and shutdown findings

Cancellation is cooperative. Research found three distinct patterns:

1. pass a token to cancel underlying work;
2. cancel only the wait with <code>WaitAsync</code>;
3. do both and retain ownership until cleanup finishes.

<code>WaitAsync</code> cannot prove that a provider request, child process, or remote effect stopped. Starting a replacement attempt while the original might still run creates overlap and ambiguous effects.

Operation-level deadlines should distinguish caller cancellation, total run budget, per-attempt budget, idle stream gap, tool runtime, and host shutdown. <code>TimeProvider</code> overloads support deterministic delays and waits.

The Generic Host has a finite shutdown timeout. Cancellation of the token passed to <code>StopAsync</code> indicates graceful time has elapsed, but the host still awaits completion. Crash/forced termination can skip graceful callbacks entirely. The robust order is stop admission/readiness, bounded drain, cancel, terminate child work, settle/persist, flush bounded telemetry, dispose.

Sources:

- [Cancel non-cancelable operations](https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/cancel-non-cancelable-async-operations)
- [Task.WaitAsync](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task-1.waitasync?view=net-10.0)
- [Exception best practices](https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions)
- [Generic Host shutdown](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-10.0)
- [TimeProvider](https://learn.microsoft.com/en-us/dotnet/standard/datetime/timeprovider-overview)

## Streaming and backpressure findings

<code>Channel&lt;T&gt;</code> provides bounded and unbounded producer-consumer queues. Bounded channels default to waiting for space. Drop modes are available but semantically dangerous for mixed agent events. Capacity is counted in items, not bytes.

Key production implications:

- add a retained-byte budget for variable-size events;
- complete the writer and propagate terminal errors;
- use <code>SingleReader</code>/<code>SingleWriter</code> only when guaranteed;
- keep <code>AllowSynchronousContinuations</code> false unless reentrancy/latency is understood;
- never drop tool calls, approvals, accounting, state transitions, or terminal events.

<code>IAsyncEnumerable&lt;T&gt;</code> is a pull contract but can still hide upstream buffering. Tokens need <code>[EnumeratorCancellation]</code> or <code>WithCancellation</code>; enumerators/streams require async disposal.

<code>System.IO.Pipelines</code> exposes byte-based pause/resume thresholds and requires precise <code>AdvanceTo</code> semantics. Incorrect consumed/examined positions can hang, retain data, spin, or drop bytes.

Sources:

- [.NET channels](https://learn.microsoft.com/en-us/dotnet/core/extensions/channels)
- [BoundedChannel source](https://github.com/dotnet/runtime/blob/main/src/libraries/System.Threading.Channels/src/System/Threading/Channels/BoundedChannel.cs)
- [Async streams](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/generate-consume-asynchronous-stream)
- [System.IO.Pipelines](https://learn.microsoft.com/en-us/dotnet/standard/io/pipelines)

## HTTP, pooling, and resilience findings

<code>HttpClient</code> must be reused. Two supported lifetime models are:

- a long-lived client with <code>SocketsHttpHandler.PooledConnectionLifetime</code>;
- short-lived factory clients backed by pooled handlers.

DNS is resolved when a connection is created, and <code>HttpClient</code> does not follow DNS TTL itself. Capturing a factory/typed client in a singleton can defeat handler rotation. Factory handler scopes do not match request scopes; they should not carry tenant context. Handler pooling also affects cookie sharing/loss.

The standard resilience handler combines rate limiting, total timeout, retry, circuit breaker, and attempt timeout. Source defaults at research time show a generic policy rather than an agent-specific one. Unsafe method retries need disabling unless replay safety is demonstrated. Multiple standard handlers should not be stacked.

The largest risk is retry multiplication:

- OpenAI .NET client: selected transient statuses, up to three additional attempts by default;
- Anthropic C# client: selected connection/status failures, two retries by default;
- .NET resilience handler: additional attempts if enabled;
- application/run policy: possible additional attempts;
- queue: possible redelivery.

Logical operations and physical attempts must be measured separately. Configure one layer as the primary retry owner.

Sources:

- [HttpClient guidelines](https://learn.microsoft.com/en-us/dotnet/fundamentals/networking/http/httpclient-guidelines)
- [IHttpClientFactory](https://learn.microsoft.com/en-us/dotnet/core/extensions/httpclient-factory)
- [.NET HTTP resilience](https://learn.microsoft.com/en-us/dotnet/core/resilience/http-resilience)
- [Standard resilience options source](https://github.com/dotnet/extensions/blob/main/src/Libraries/Microsoft.Extensions.Http.Resilience/Resilience/HttpStandardResilienceOptions.cs)
- [OpenAI .NET retries](https://github.com/openai/openai-dotnet#automatically-retrying-errors)
- [Anthropic C# retries/timeouts](https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/csharp)

## Process and sandbox findings

Modern .NET defaults <code>UseShellExecute</code> to false, but security-boundary code should set it explicitly. Exact executable paths and <code>ArgumentList</code> avoid shell quoting and injection.

Redirected stdout and stderr must be consumed concurrently. Reading one to completion or waiting for exit before draining can deadlock when pipes fill. Output must be retained under a byte cap while the runner continues draining or terminates the child.

Canceling <code>WaitForExitAsync</code> cancels the wait, not the process. <code>Kill(entireProcessTree: true)</code> is an escalation mechanism, but process-tree behavior and the observability of descendants vary. .NET 10 Windows process groups improve signaling, not isolation.

A process running as the host identity is not a sandbox. Security requires OS/container/VM enforcement: identities, filesystem mounts, network policy, syscall restrictions, resource controls, and minimal credentials.

Sources:

- [ProcessStartInfo.UseShellExecute](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.processstartinfo.useshellexecute?view=net-10.0)
- [Process.StandardOutput](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.process.standardoutput?view=net-10.0)
- [Process.Kill](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.process.kill?view=net-10.0)
- [.NET 10 process library changes](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/libraries)

## JSON and structured-output findings

.NET 10 <code>JsonSerializerOptions.Strict</code> provides a useful strict preset. Ordinary System.Text.Json behavior can allow unmapped members and duplicate properties. Byte, depth, string, collection, and decompression limits remain application responsibilities.

Source generation is preferred for stable wire contracts and required for reliable Native AOT. Disabling reflection defaults during build/test exposes accidental unsupported paths.

<code>JsonSchemaExporter</code> maps the configured serialization contract to JSON Schema. It reduces drift but does not know a provider's supported subset or business invariants. A provider-specific transformation/golden-test stage is required.

OpenAI and Anthropic both support structured output/strict tool schemas on eligible models/endpoints. Availability and accepted schema features vary. Completed schema-conforming tool arguments still require local parse, semantic validation, canonicalization, authorization, and an effect policy.

Sources:

- [.NET 10 library changes](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/libraries)
- [System.Text.Json source generation](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/source-generation)
- [Reflection versus source generation](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/reflection-vs-source-generation)
- [Unmapped members](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/missing-members)
- [Required properties](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/required-properties)
- [JSON Schema exporter](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/extract-schema)
- [OpenAI Responses create reference](https://developers.openai.com/api/reference/resources/responses/methods/create)
- [Anthropic structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)

## Effects, queues, and durability findings

The hard distributed-systems case is a committed remote effect whose acknowledgment is lost. The result is unknown, not failed. Safe handling requires a stable business effect ID, durable intent, downstream idempotency/query when possible, and reconciliation.

Azure Service Bus duplicate detection deduplicates application-controlled message IDs within a configured window. It addresses duplicate sends, not all redeliveries or external effects. Peek-lock delivery can return work after lock loss, and poison messages eventually dead-letter based on delivery count. Consumers remain idempotent.

Durable workflow engines replay deterministic decisions and retry activities:

- Durable Task/Durable Functions: Azure-oriented orchestration, entities, timers, and workers;
- Agent Framework Durable extension: agent sessions on Durable Task, currently prerelease;
- Temporal .NET: durable workflows/activities, Generic Host integration, testing, telemetry, versioning;
- Dapr Workflow: durable Dapr building block with deterministic workflow code and versioning.

Model calls, wall clock, randomness, filesystem/network access, and tool effects belong in activities, not replayed orchestrator code. Activity execution can repeat, so the effect protocol remains necessary.

Sources:

- [Service Bus duplicate detection](https://learn.microsoft.com/en-us/azure/service-bus-messaging/duplicate-detection)
- [Service Bus locks and settlement](https://learn.microsoft.com/en-us/azure/service-bus-messaging/message-transfers-locks-settlement)
- [Durable Functions overview](https://learn.microsoft.com/en-us/azure/durable-task/durable-functions/durable-functions-overview)
- [Durable Task for AI agents](https://learn.microsoft.com/en-us/azure/durable-task/sdks/durable-task-for-ai-agents)
- [Agent Framework durable agents](https://learn.microsoft.com/en-us/azure/durable-task/sdks/durable-agents-microsoft-agent-framework)
- [Temporal .NET SDK](https://github.com/temporalio/sdk-dotnet)
- [Dapr Workflow overview](https://docs.dapr.io/developing-applications/building-blocks/workflow/workflow-overview/)
- [Dapr Workflow concepts](https://docs.dapr.io/developing-applications/building-blocks/workflow/workflow-features-concepts/)

## Memory, GC, thread pool, and admission findings

"Memory" is overloaded in agent systems. Research supports separating:

- working memory: the bounded context assembled for one invocation;
- session memory: transcript/compacted continuity scoped to a conversation or run;
- durable operational state: authoritative run transitions, approvals, leases, events, and effect receipts;
- long-term semantic memory: optional extracted facts/preferences/episodes with provenance, consent, correction, expiry, and tenant scope;
- knowledge/RAG: external documents retrieved under their current source authorization.

Similarity retrieval is not deterministic state lookup and a vector index is not an authorization boundary. The source object and its current tenant/policy must be revalidated before context insertion. Provider conversation handles and compacted summaries are derived session material, not the only copy of operational state. Opaque provider conversation/response IDs belong in trusted server-side storage mapped from an application-owned tenant-scoped session ID.

Context construction needs explicit headroom for output/reasoning, fixed instructions, and tool schemas. Mandatory material includes current policy/goal, recent user turn, accepted decisions, user corrections, approval scope, effect receipts/unknowns, active identifiers/versions, and complete tool-call/result pairs. Large documents and tool results should become bounded excerpts or immutable integrity-checked references.

Compaction is a versioned transformation, not arbitrary summarization. A useful record includes source sequence range/hash, schema/prompt and model/algorithm versions, token/byte counts, structured decisions/open questions/effects, and validation status. Continuation evaluations should repeatedly compact and verify policy, corrections, tool pairs, effects, and cross-tenant isolation.

OpenAI Responses documents context-management compaction and a compact endpoint that returns an opaque compaction item; it remains provider-specific session material. Semantic Kernel documents truncate, token-based, and summarization reducers and preservation of system/function-message structure. Agent Framework separates conversation storage/history providers from broader context providers. These sources converge on one application responsibility: assign one compaction owner and keep authoritative state outside the compacted view.

The large object heap threshold is approximately 85,000 bytes. Large strings and arrays produced by prompts, JSON, attachments, embeddings, and transcript assembly can therefore alter GC behavior. Streaming and external references are safer than building repeated contiguous buffers.

Thread-pool starvation is diagnosed by growing thread count/queue and latency, often with low CPU. Sync-over-async and blocking I/O are common causes. Newer runtime compensation reduces symptoms but does not make blocking correct.

Managed heap is not total memory. Native buffers, mapped files, child processes, sidecars, page cache, and temporary disk must be included. Container GC behavior and heap limits need measurement on the actual CPU/memory quota.

<code>System.Threading.RateLimiting</code> supports concurrency, token bucket, fixed/sliding window, and partitions. An in-process limiter is not a global multi-replica quota, and a large wait queue is not overload control. Agent admission needs weights for bytes, tokens, processes, model/tool concurrency, and tenant fairness.

Sources:

- [.NET GC fundamentals](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/)
- [Large object heap](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/large-object-heap)
- [GC runtime configuration](https://learn.microsoft.com/en-us/dotnet/core/runtime-config/garbage-collector)
- [Thread-pool starvation diagnosis](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/debug-threadpool-starvation)
- [.NET runtime metrics](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/built-in-metrics-runtime)
- [Rate limiting with .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/http-ratelimiter)
- [OpenAI Responses compaction](https://developers.openai.com/api/docs/guides/compaction)
- [OpenAI Responses context management](https://developers.openai.com/api/reference/resources/responses/methods/create)
- [Semantic Kernel chat-history reduction](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/chat-completion/chat-history)
- [Agent Framework conversation storage](https://learn.microsoft.com/en-us/agent-framework/concepts/agents/conversations/storage)
- [Agent Framework context providers](https://learn.microsoft.com/en-us/agent-framework/integrations/by-component/context-providers/)

## Telemetry and diagnostics findings

OpenTelemetry .NET traces, metrics, and logs are stable. .NET instrumentation builds on <code>ActivitySource</code>, <code>Meter</code>, and <code>ILogger</code>. Generative-AI semantic conventions remain an evolving specification area; pin the version/attributes used by dashboards.

The safe default is metadata-only telemetry. Prompt/response/tool content can contain secrets and regulated data and creates cardinality/volume costs. Run/effect IDs belong in spans/logs, not metric labels.

EventPipe and the <code>dotnet-*</code> tools support cross-platform diagnosis. <code>dotnet-gcdump</code> can induce a full generation-2 GC and pause. Dumps contain sensitive memory. The diagnostics channel itself is powerful and must be access-controlled; disabling diagnostics is an option in hostile same-identity environments, with an explicit loss of live-debug capability.

Telemetry becomes operationally useful when attached to service-level objectives. Agent success must mean a correct explicit terminal state, not HTTP 200 or a normally closed model stream. Separate indicators are needed for admission decisions/rejections, admitted-run correctness, admit-to-terminal latency and time to first useful output, durable recovery RTO, stream resume, and unknown-effect age. Quality/safety/cost objectives come from evaluated samples and should not be hidden inside infrastructure availability. Error-budget alerts need fast/slow burn windows and a runbook with saturation evidence, safe degradation, reconciliation, and rollback actions.

Sources:

- [OpenTelemetry .NET](https://opentelemetry.io/docs/languages/dotnet/)
- [OpenTelemetry generative AI conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
- [OpenTelemetry observability in .NET](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/observability-with-otel)
- [EventPipe](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/eventpipe)
- [Diagnostic ports](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/diagnostic-port)
- [dotnet-gcdump](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-gcdump)
- [dotnet-dump](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-dump)
- [.NET System.Net metrics](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/built-in-metrics-system-net)
- [Reliability metrics, SLIs, and SLOs](https://learn.microsoft.com/en-us/azure/well-architected/reliability/metrics)

## Testing findings

Microsoft Testing Platform and VSTest are separate platforms. .NET 10 improves explicit <code>dotnet test</code> support for MTP. Framework choice (MSTest/NUnit/xUnit/TUnit) is separate from platform choice.

<code>TimeProvider</code>/<code>FakeTimeProvider</code> make timers, retry delays, timeout edges, and leases deterministic. They do not automatically control third-party SDK timers, so provider adapters and scripted local HTTP servers remain important.

The highest-value agent tests cover:

- malformed/fragmented streams and slow consumers;
- provider throttling, stalls, disconnects, and late completions;
- cancellation races at channel, storage, HTTP, and process boundaries;
- failure immediately before/after remote commit and local state commit;
- broker redelivery and stale leases;
- tool descendant processes and full stdout/stderr pipes;
- mixed-weight load until resource plateaus or controlled rejection;
- exact published containers and AOT/trimming paths.

Behavioral evaluation is a separate test layer. Maintain a versioned, application-owned dataset of typical, edge, adversarial, and production-incident cases with the prompt, model route/snapshot, schema/tool/policy, memory/compaction, and knowledge versions needed to explain a result. Prefer deterministic invariants for authorization, state, budgets, tool arguments, citations, and effects; use rule/reference scorers next and calibrated model judges only for subjective quality. Evaluate the trace as well as the final text and sample important nondeterministic cases repeatedly.

<code>Microsoft.Extensions.AI.Evaluation</code> provides .NET evaluator/reporting packages, including task-adherence, intent-resolution, and tool-call evaluators. They are building blocks rather than a universal score. OpenAI's official guidance recommends task-specific, representative, continuous evaluation with human calibration. The same page schedules its legacy Evals platform read-only transition for 2026-10-31 and shutdown for 2026-11-30; a new .NET release gate should remain application-owned rather than coupling to that retiring surface.

Restart safety requires actual process termination between durable/network boundaries, not exceptions alone. Kill after intent, after remote fixture commit, after receipt, and before queue settlement; restart a fresh host and recover only from persisted records. Additional fault injection should cover storage timeouts/stale reads, lease loss, DNS/TLS/identity failure, truncated blobs, secret rotation, and telemetry backpressure.

Sources:

- [.NET testing](https://learn.microsoft.com/en-us/dotnet/core/testing/)
- [Microsoft Testing Platform](https://learn.microsoft.com/en-us/dotnet/core/testing/microsoft-testing-platform-intro)
- [Test platform overview](https://learn.microsoft.com/en-us/dotnet/core/testing/test-platforms-overview)
- [TimeProvider testing](https://learn.microsoft.com/en-us/dotnet/core/extensions/timeprovider-testing)
- [ASP.NET Core integration tests](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-10.0)
- [.NET AI evaluation libraries](https://learn.microsoft.com/en-us/dotnet/ai/evaluation/libraries)
- [OpenAI evaluation best practices](https://developers.openai.com/api/docs/guides/evaluation-best-practices)
- [OpenAI agent trace evaluation](https://developers.openai.com/api/docs/guides/agent-evals)

## NuGet and build findings

For deployable applications, central package management plus <code>packages.lock.json</code> and locked restore provide a reviewable graph. A library lock file cannot force the graph of downstream applications.

Package Source Mapping restricts the sources for package patterns, including transitives. Every package must match once mapping is enabled. It does not eliminate all metadata queries to other configured feeds.

.NET 10 changes NuGet audit behavior so transitive packages are included by default for .NET 10 targets. Suppressions require ownership and expiry. NuGet packages can execute build targets, analyzers, and source generators in developer/CI contexts, so package review includes more than runtime assemblies.

Sources:

- [PackageReference](https://learn.microsoft.com/en-us/nuget/consume-packages/package-references-in-project-files)
- [Central Package Management](https://learn.microsoft.com/en-us/nuget/consume-packages/central-package-management)
- [Package Source Mapping](https://learn.microsoft.com/en-us/nuget/consume-packages/package-source-mapping)
- [NuGet auditing](https://learn.microsoft.com/en-us/nuget/concepts/auditing-packages)
- [Signed packages](https://learn.microsoft.com/en-us/nuget/consume-packages/installing-signed-packages)
- [.NET 10 transitive audit compatibility note](https://learn.microsoft.com/en-us/dotnet/core/compatibility/sdk/10.0/nugetaudit-transitive-packages)

## Deployment findings

Native AOT improves startup and can reduce memory, but removes JIT/dynamic loading and constrains reflection/code generation. It can increase size through generic instantiations and reduce diagnostic/dynamic capabilities. Agent SDK compatibility must be demonstrated from the exact published graph.

Official .NET container images provide SDK, ASP.NET, runtime, runtime-deps, AOT, chiseled/distroless, and composite variants. Chiseled images are non-root/minimal but usually omit shell/package manager; ICU/time-zone data depends on the variant. Tooling assumptions must drive image choice.

Horizontal scaling requires external state/effects, leases/fencing, queue settlement discipline, and cross-replica provider admission. More replicas can amplify provider throttling. Readiness must turn off before drain and the host budget must fit the platform grace period.

Framework-dependent deployments can receive a centrally serviced host runtime; self-contained and container artifacts carry their runtime and therefore require rebuild/redeploy for servicing updates. Release evidence must identify runtime patch, base-image digest, TFM, RID, architecture, libc, and globalization/time-zone mode. A publish gate should execute serialization/source-generation, DI/reflection paths, provider streaming, process tools/signals, diagnostics, TLS/proxy/identity, readiness, and drain from the exact artifact.

ASP.NET Core Secret Manager is development-only and does not encrypt stored values. Production should prefer workload identity/federation where supported, otherwise a controlled secret store with narrow scope, bounded caching, rotation/revocation tests, and no value logging. Environment variables, child-process inheritance, diagnostics, command arguments, model context, tool output, and durable state are all possible exfiltration paths.

Sources:

- [Native AOT](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/)
- [ASP.NET Core Native AOT](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/native-aot?view=aspnetcore-10.0)
- [.NET container images](https://learn.microsoft.com/en-us/dotnet/core/docker/container-images)
- [Official .NET container repository](https://github.com/dotnet/dotnet-docker)
- [Image variants](https://github.com/dotnet/dotnet-docker/blob/main/documentation/image-variants.md)
- [Ubuntu chiseled](https://github.com/dotnet/dotnet-docker/blob/main/documentation/ubuntu-chiseled.md)
- [System.Text.Json source generation](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/source-generation)
- [ASP.NET Core app secrets](https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-10.0)
- [Azure Key Vault security guidance](https://learn.microsoft.com/en-us/azure/key-vault/general/secure-key-vault)

## Agent ecosystem findings

### Version and maturity matrix

| Component | Observed version/date | Official status signal | Boundary |
|---|---|---|---|
| OpenAI .NET | 2.13.0 | Official stable package; selected APIs can be experimental | Provider API client, not an Agents SDK |
| Anthropic C# | 12.44.0 in 2026-08-27 release notes | Official package, documentation says beta | Provider API client |
| Microsoft.Extensions.AI | 10.9.0 family | Stable abstractions with experimental members possible | Chat/embedding abstractions and middleware |
| Microsoft Agent Framework | .NET 1.19.0, 2026-08-22 | Core reached 1.0 GA; integration maturity varies | Agent/workflow abstractions |
| MCP C# SDK | 2.2.0, 2026-08-13 | Official SDK, current 2026-07-28 protocol line | Capability protocol |
| Agent Framework Durable | 1.x preview package line | Prerelease | Durable Agent Framework adapter |
| Temporal .NET | 1.18.0 | Stable | General durable workflow |

### OpenAI

The official <code>OpenAI</code> .NET package is an API client generated from the OpenAPI specification with Microsoft collaboration. It supports Responses, Chat, streaming, tools, structured output, observability, testing factories, and Azure-compatible endpoints. Client objects are documented as thread-safe/singleton-safe, and the library retries selected transient failures up to three additional times by default. Current official Responses examples suppress the <code>OPENAI001</code> experimental diagnostic, demonstrating why package stability and individual surface stability must be recorded separately.

No official high-level OpenAI Agents SDK for .NET comparable to the Python/TypeScript Agents SDKs was found in official documentation at the research date; official Agents SDK repositories/docs list those two languages. OpenAI distinguishes direct Responses API use, where the application owns the loop, from an Agents SDK, where the SDK owns more orchestration. The practical C# options are a small owned Responses loop behind an adapter or Microsoft Agent Framework/Extensions.AI when their abstractions and maturity fit.

Sources:

- [OpenAI .NET repository](https://github.com/openai/openai-dotnet)
- [OpenAI .NET releases](https://github.com/openai/openai-dotnet/releases)
- [OpenAI .NET changelog](https://github.com/openai/openai-dotnet/blob/main/CHANGELOG.md)
- [OpenAI Responses API](https://developers.openai.com/api/reference/resources/responses/methods/create)
- [Official OpenAI API libraries](https://developers.openai.com/api/docs/libraries)
- [OpenAI agent-building and Agents SDK options](https://developers.openai.com/api/docs/guides/agents)

### Anthropic

The official package ID is <code>Anthropic</code>, .NET Standard 2.0+, with Messages, streaming, typed exceptions, retry/timeout configuration, tools, <code>IChatClient</code>, and separate platform integration packages. The documentation explicitly says the current 10+ package line is beta and may make breaking changes in minor/patch releases.

At the research date, release notes list C# SDK 12.44.0. Older package identities and community SDK history create a supply-chain/name-confusion risk; verify publisher and repository.

Sources:

- [Anthropic SDK overview](https://platform.claude.com/docs/en/cli-sdks-libraries/overview)
- [Anthropic C# SDK](https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/csharp)
- [Anthropic release notes](https://platform.claude.com/docs/en/release-notes/overview)
- [Anthropic C# repository](https://github.com/anthropics/anthropic-sdk-csharp)

### Microsoft.Extensions.AI and Agent Framework

<code>Microsoft.Extensions.AI</code> supplies <code>IChatClient</code>, <code>IEmbeddingGenerator</code>, caching, telemetry, and function invocation. It is not durability, authorization, or sandboxing. Middleware order and mutable request options still matter.

Microsoft Agent Framework reached 1.0 production readiness in April 2026 and has continued rapid releases. It is the successor direction for Microsoft agent functionality while Semantic Kernel remains supported. Current docs label self-hosting, some Foundry packages, and Durable integration prerelease. Assess individual packages and API annotations, not the umbrella release.

Self-hosting leaves authentication, authorization, policy, deployment/scaling, and storage with the application; the documentation does not supply a general-purpose durable session store. The Durable Task extension remains prerelease. Microsoft documents a 1 MB scheduler-state limit, manual history compaction, and response/callback streaming constraints for durable agents, so large content belongs behind external references and presentation streaming needs a separate design.

Sources:

- [Microsoft.Extensions.AI](https://learn.microsoft.com/en-us/dotnet/ai/microsoft-extensions-ai)
- [dotnet/extensions releases](https://github.com/dotnet/extensions/releases)
- [Agent Framework getting started](https://learn.microsoft.com/en-us/agent-framework/get-started/)
- [Agent Framework releases](https://github.com/microsoft/agent-framework/releases)
- [Agent Framework self-hosting](https://learn.microsoft.com/en-us/agent-framework/hosting/self-hosting)
- [Agent Framework durable agents](https://learn.microsoft.com/en-us/azure/durable-task/sdks/durable-agents-microsoft-agent-framework)
- [Semantic Kernel and Agent Framework](https://devblogs.microsoft.com/agent-framework/semantic-kernel-and-microsoft-agent-framework/)

### MCP

The official C# SDK is maintained with Microsoft. Version 2 aligns to MCP 2026-07-28, uses discovery-first behavior and stateless defaults, and automatically supports older protocol versions. Version 2.2 adds hybrid support for modern stateless and legacy/stateful clients.

Packages separate Core, hosting/DI/stdio, and ASP.NET Core HTTP. Streamable HTTP is the current network transport; legacy SSE remains for compatibility. Stateless operation avoids protocol affinity but does not remove application state; explicit state handles/external stores remain necessary. Stateful sessions require lifecycle/routing. HTTP can integrate identity; stdio has no built-in remote-auth boundary.

The C# SDK authorization filters are an explicit registration: <code>[Authorize]</code> policies on tools/resources/prompts require <code>AddAuthorizationFilters()</code>. Unauthorized discovery entries and direct invocation need separate tests. Structured output can advertise JSON Schema but is not authorization or sanitization. The built-in in-memory MCP task store is documented for development/testing, so production long-running tasks need durable tenant-scoped storage and cooperative cancellation. MCP makes capability exchange interoperable but does not authorize or sandbox tools.

Sources:

- [MCP C# repository](https://github.com/modelcontextprotocol/csharp-sdk)
- [MCP C# releases](https://github.com/modelcontextprotocol/csharp-sdk/releases)
- [MCP C# getting started](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/getting-started.md)
- [MCP transports](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/transports/transports.md)
- [MCP stateless operation](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/stateless/stateless.md)
- [MCP identity](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/identity/identity.md)
- [MCP authorization filters](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/filters.md)
- [MCP tools and structured content](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/tools/tools.md)
- [MCP tasks](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/tasks/tasks.md)
- [MCP SDK tiers](https://modelcontextprotocol.io/community/sdk-tiers)

### Azure and Microsoft Foundry

The Foundry SDK documentation lists .NET 2.0 GA packages <code>Azure.AI.Projects</code>, <code>Azure.AI.Projects.Agents</code>, and <code>Azure.AI.Extensions.OpenAI</code>. It warns not to combine the older preview <code>Azure.AI.Projects.OpenAI</code> package with the GA extension because duplicate types cause ambiguity.

Foundry differentiates:

- project-native management/configuration through <code>AIProjectClient</code>;
- OpenAI-compatible Responses/agents/evaluations through the project OpenAI surface;
- application-owned orchestration via model/provider integrations;
- server-managed Prompt/Hosted Agents;
- Foundry-hosted Agent Framework containers.

Current Agent Framework Foundry and hosting examples still require prerelease .NET packages, even when the underlying project SDK is GA. Pin server-managed agent versions and authorize callers before resuming conversations.

For standalone Azure OpenAI, current Azure SDK guidance recommends the primary official <code>OpenAI</code> SDK for many scenarios. <code>Azure.AI.OpenAI</code> remains a companion for Azure-specific types. Entra ID/managed identity is preferred in Azure-hosted apps.

Sources:

- [Foundry SDK overview](https://learn.microsoft.com/en-us/azure/foundry/how-to/develop/sdk-overview)
- [Foundry integrations](https://learn.microsoft.com/en-us/agent-framework/integrations/by-provider/microsoft-foundry)
- [Foundry Agent Service](https://learn.microsoft.com/en-us/agent-framework/integrations/by-component/agent-services/foundry)
- [Foundry Hosted Agents](https://learn.microsoft.com/en-us/agent-framework/hosting/foundry-hosted-agent)
- [Azure OpenAI authentication](https://learn.microsoft.com/en-us/dotnet/ai/azure-ai-services-authentication)
- [Azure.AI.OpenAI README](https://github.com/Azure/azure-sdk-for-net/blob/main/sdk/openai/Azure.AI.OpenAI/README.md)
- [Azure OpenAI .NET migration guidance](https://github.com/Azure/azure-sdk-for-net/blob/main/sdk/openai/Azure.AI.OpenAI/migration-guidance.md)

## Decisions carried into the guide set

1. Baseline on .NET 10 LTS and C# 14; do not normalize preview .NET 11 behavior.
2. Make the run owner the organizing architecture rather than a specific agent framework.
3. Treat wait cancellation and work cancellation as separate concepts in every guide.
4. Require byte budgets in addition to channel item capacity.
5. Count retry layers and designate one owner.
6. Use strict System.Text.Json parsing and provider-specific schema compatibility tests.
7. Treat every side effect as at-least-once unless downstream idempotency proves otherwise.
8. Require explicit durable state for restart survival; do not call a transcript state.
9. Separate working context, session continuity, operational state, long-term semantic memory, and knowledge/RAG.
10. Make compaction a versioned derived-view transformation with preservation invariants and continuation evaluations.
11. Use OpenTelemetry/.NET primitives but capture content only under a governed opt-in policy; define operational SLOs from explicit terminal semantics.
12. Pair runtime/fault tests with task-specific behavioral evaluations and deterministic trace/effect invariants.
13. Treat MCP as a protocol, Agent Framework as orchestration abstractions, and workflow engines as durability; none is a sandbox.
14. Distinguish the official OpenAI .NET API client from the Python/TypeScript-only official Agents SDK offering.
15. Isolate prerelease or experimental provider/hosting/durable surfaces behind adapters even inside stable packages.
16. Require publish-and-run evidence for trimming, Native AOT, containers, secrets/identity, and process behavior.

## Unstable or version-sensitive areas

Refresh these before a major update:

- .NET 11 runtime/C# feature changes after its GA;
- provider model and structured-output schema availability;
- OpenAI and Anthropic SDK retry/timeout defaults;
- OpenAI .NET Responses experimental annotations and official Agents SDK language availability;
- OpenAI legacy Evals platform retirement and replacement surfaces;
- Anthropic C# package beta status;
- Microsoft Agent Framework integration/hosting/durable package maturity;
- Microsoft Foundry package naming and classic/new project compatibility;
- MCP protocol revision, SDK tier, and stateful/stateless migration;
- OpenTelemetry generative-AI semantic-convention version;
- provider context-compaction formats/limits and Agent Framework memory-provider maturity by language;
- Native AOT compatibility of selected SDK/exporter/workflow packages;
- NuGet audit defaults for later target frameworks.

## Negative findings and exclusions

- No evidence supports treating a child process under the same identity as a sandbox.
- No broker/workflow source supports assuming exactly-once external effects.
- No common chat abstraction erases provider-specific terminal events, schemas, usage, retries, or hosted-tool semantics.
- No official high-level OpenAI Agents SDK for .NET was identified at the research date.
- The official OpenAI .NET API client must not be presented as that missing Agents SDK; the distinction is ownership of API calls versus the higher-level agent loop.
- Restate official SDK documentation did not list .NET as a supported SDK in the researched material; it was not recommended as a current native C# option.
- Preview framework/runtime features were not used as baseline guidance.
- Generic community agent frameworks were not surveyed; the requested scope emphasized official provider, Microsoft, MCP, Azure, and durable SDKs.

## Refresh checklist

- [ ] Recheck .NET support policy and latest LTS patch.
- [ ] Recheck every package version/status table against official releases.
- [ ] Diff retry, timeout, and tracing defaults.
- [ ] Re-run provider schema compatibility fixtures.
- [ ] Recheck MCP protocol revision and SDK migration guidance.
- [ ] Recheck Foundry classic/new package compatibility warnings.
- [ ] Recheck Agent Framework hosting/durable prerelease status.
- [ ] Recheck provider compaction/context-management behavior and run continuation/isolation evaluations.
- [ ] Recheck official OpenAI Agents SDK language support and the legacy Evals retirement timeline.
- [ ] Recalibrate behavioral-eval datasets/judges and operational SLO denominators.
- [ ] Publish and run the current trimmed/AOT/container graph.
- [ ] Update the language-area research date only after completing these checks.

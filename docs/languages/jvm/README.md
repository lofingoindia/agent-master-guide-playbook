# JVM Agent Engineering

> **Status:** Deep production playbook  
> **Last researched:** 2026-08-31  
> **Runtime baseline:** JDK 25 LTS API; Oracle JDK 25.0.4.1 was the current security baseline on 2026-08-31; Kotlin 2.3.20; kotlinx.coroutines 1.11.0  
> **Evidence:** [JVM agent-engineering research packet](../../research/packets/jvm-agent-engineering-deep-dive.md)

Production AI agents on the JVM are ordinary distributed systems with an unusually probabilistic decision component. The hard parts are still admission control, cancellation, durable state, side-effect safety, resource ownership, observability, and recovery. Java and Kotlin can share protocols and domain types, but their concurrency models must remain explicit.

## Read this first

1. [Architecture and language boundaries](architecture-and-language-boundaries.md)
2. Choose the runtime path:
   - [Java concurrency, virtual threads, and structured concurrency](java-concurrency-virtual-threads-and-structured-concurrency.md)
   - [Kotlin coroutines, scopes, and flows](kotlin-coroutines-scopes-and-flows.md)
3. Apply the shared run-control rules in [Cancellation, deadlines, and run control](cancellation-deadlines-and-run-control.md).
4. Design effects using [Errors, retries, and idempotency](errors-retries-and-idempotency.md) and [Queues, state, and durable workers](queues-state-and-durable-workers.md).
5. Close the production loop with the testing, observability, resource, build, and deployment guides.

## Build from zero to production

Do not start by assembling every framework feature. Advance only when the previous stage has evidence:

| Stage | Build | Exit evidence |
|---|---|---|
| 0. Deterministic core | one model port, one read-only fake tool, typed outcomes, fake clock | pure tests prove step limit, deadline, validation, and terminal states |
| 1. One real provider | adapter normalizes non-stream/stream results, usage, refusal, request IDs | captured fixtures and cancellation/byte-limit tests pass |
| 2. Safe effects | server-side tool registry, schema/business validation, policy, stable effect ID | duplicate delivery and ambiguous-timeout tests produce one logical effect |
| 3. Durable continuity | versioned events/checkpoint, artifact references, bounded context compaction | process-kill test resumes without lost budget, approval, or repeated effect |
| 4. Controlled concurrency | Java virtual-thread or Kotlin coroutine ownership plus independent bulkheads | load test keeps queue age, retained bytes, and downstream permits bounded |
| 5. Isolated tools | container/microVM/worker boundary, least privilege, output caps | timeout/kill/network/filesystem escape drills pass |
| 6. Operable service | OTel business spans, metrics, audit, rolling JFR, runbooks | an operator can trace and reconcile a failed run without prompt disclosure |
| 7. Production rollout | pinned supply chain, migrations, canary, shutdown/recovery drills | SLO, cost, security, rollback, and compatibility gates are signed off |

Memory is added by need, not stage number. Start with explicit working context, durable run state, and artifact references. Add user/domain long-term memory only after an evaluation proves value and provenance, deletion, tenant isolation, poisoning, freshness, and retrieval costs are owned.

## Guide map

| Concern | Guide | Primary decision |
|---|---|---|
| Boundaries | [Architecture and language boundaries](architecture-and-language-boundaries.md) | Keep the loop deterministic around explicit effects |
| Java execution | [Java concurrency, virtual threads, and structured concurrency](java-concurrency-virtual-threads-and-structured-concurrency.md) | Use blocking code on one virtual thread per run; bound scarce resources |
| Kotlin execution | [Kotlin coroutines, scopes, and flows](kotlin-coroutines-scopes-and-flows.md) | Give every run an owned scope and preserve cancellation |
| Run lifetime | [Cancellation, deadlines, and run control](cancellation-deadlines-and-run-control.md) | Carry one absolute deadline through every layer |
| Transport | [HTTP, streaming, and backpressure](http-streaming-and-backpressure.md) | Bound bytes, time, and buffered events |
| Tool execution | [Tools, processes, and sandboxing](tools-processes-and-sandboxing.md) | Treat tools as untrusted side effects; isolate outside the JVM |
| Contracts | [Schemas, serialization, and structured output](schemas-serialization-and-structured-output.md) | Validate at every trust boundary and version schemas |
| Failure policy | [Errors, retries, and idempotency](errors-retries-and-idempotency.md) | Retry operations, not whole runs; make effects replay-safe |
| Durable execution | [Queues, state, and durable workers](queues-state-and-durable-workers.md) | Persist transitions before acknowledging work |
| Capacity | [Memory, GC, and resource control](memory-gc-and-resource-control.md) | Heap, native memory, and external quotas are separate budgets |
| Operations | [Observability, JFR, JMX, and profiling](observability-jfr-jmx-and-profiling.md) | Correlate run, attempt, model call, and tool effect |
| Verification | [Testing, virtual time, concurrency, and load](testing-virtual-time-concurrency-and-load.md) | Test schedules and failure boundaries, not only happy transcripts |
| Build integrity | [Dependencies, build, and supply chain](dependencies-build-and-supply-chain.md) | Pin toolchains and verify the complete dependency graph |
| Runtime lifecycle | [Deployment, containers, and shutdown](deployment-containers-and-shutdown.md) | Stop admission, drain, checkpoint, then terminate |
| SDK selection | [Providers, frameworks, MCP, and durable SDKs](providers-frameworks-mcp-and-durable-sdks.md) | Verify exact feature parity; adapters are not architecture |
| Decisions | [Ecosystem, language parity, and anti-patterns](ecosystem-language-parity-and-antipatterns.md) | Choose by operational fit, not demo surface |

## Reference architecture

~~~mermaid
flowchart LR
    A[Ingress] --> B[Admission and run registry]
    B --> C[Agent loop]
    C --> D[Provider adapter]
    C --> E[Tool gateway]
    C --> F[State and checkpoint port]
    D --> G[Model provider]
    E --> H[Isolated tool workers]
    F --> I[Durable store or workflow runtime]
    C --> J[Events, traces, metrics]
    J --> K[OTel and JFR]
~~~

The loop owns policy: budgets, step ordering, validation, and effect identifiers. Provider, tool, and durability libraries remain replaceable adapters. This prevents an SDK callback model from becoming the system's persistence or security model.

## Non-negotiable invariants

- A run has one owner, one stable identity, one absolute deadline, and a bounded resource budget.
- Every child task belongs to the run or to an explicitly named application lifetime.
- Cancellation means “stop requesting more work”; it does not prove a remote model call or external effect stopped.
- Every effect has a stable idempotency key and a persisted outcome.
- Model and tool payloads are untrusted data, even when produced by an internal model.
- Streaming is bounded and loss policy is explicit.
- Tool subprocesses are not sandboxes. The Java Security Manager cannot be enabled on JDK 25.
- Durable replay never executes nondeterministic I/O directly.
- Prompts, tool arguments, outputs, and memory are sensitive by default.
- Readiness and liveness are separate; overload is not process death.

Use the application-owned [agent state and event contract](../../runtime/agent-state-and-event-contracts.md) for stable run identity, fenced transitions, event ordering, replay, terminal outcomes, and effect correlation across Java/Kotlin services, queues, streams, and durable runtimes.

## Version posture

JDK 25 is the API baseline because it includes mature virtual threads, final scoped values, and preview structured concurrency. Oracle JDK 25.0.4.1 was the current security baseline at the research date; other vendors may use different build identifiers, so pin vendor, build, image digest, and security-update policy rather than only `25`. Production teams that cannot accept preview APIs should use stable executors plus explicit task ownership. Kotlin guidance targets Kotlin 2.3.20 and kotlinx.coroutines 1.11.0 as separately versioned components. The coroutines 1.11.0 release was built against Kotlin 2.2.20 rather than as a companion to 2.3.20, so test the exact compiler/plugin/library matrix instead of inferring compatibility from similar dates. Library versions move faster than this guide; pin and verify supported releases rather than copying an unqualified “latest” coordinate.

## Sources

- [Oracle Java SE 25 concurrency guide](https://docs.oracle.com/en/java/javase/25/core/concurrency.html)
- [Oracle JDK 25.0.4.1 release notes](https://www.oracle.com/java/technologies/javase/25-0-4-1-relnotes.html)
- [Kotlin coroutines guide](https://kotlinlang.org/docs/coroutines-guide.html)
- [OpenTelemetry Java](https://opentelemetry.io/docs/languages/java/)
- [MCP SDK tiers](https://modelcontextprotocol.io/community/sdk-tiers)

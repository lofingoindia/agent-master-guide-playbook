# Research Packet: JVM Agent Engineering Deep Dive

- **Research date:** 2026-08-31
- **Scope:** Java and Kotlin production agent systems: concurrency, cancellation, streaming, tools, schemas, effects, durability, memory, observability, testing, build, deployment, provider/framework/MCP support.
- **Output:** [JVM Agent Engineering](../../languages/jvm/README.md)

## Research method

The research prioritized current primary material: Java SE 25 documentation and APIs, Kotlin and kotlinx project documentation/releases, official provider/agent/MCP repositories, OpenTelemetry specifications, durable-runtime APIs, official framework manuals, build-tool manuals, standards, and Kubernetes documentation. Search results were used to discover sources; substantive claims were checked against the primary page/repository.

Version-sensitive claims were cross-checked by release date and status. Preview, beta, experimental, development-status, deprecated, and protocol-lag findings are kept explicit. The packet avoids calling a capability “supported” merely because a type or dependency exists.

The pass-2 review re-opened primary sources for the current JDK security baseline, `StructuredTaskScope` timeout/close behavior, Kotlin JVM cancellation bridges, Flow buffering, MCP Java 2.0.1 versus the 2026-07-28 specification, OpenTelemetry status and sensitive GenAI attributes, JFR/JMX operations, and durable-runtime replay/shutdown behavior. Bounded maintainer evidence was used only where it changes an adoption decision, such as the open Temporal Spring AI 2 / Spring Boot 4 compatibility request.

## Executive findings

1. **JDK 25 changes old virtual-thread advice.** Virtual threads remain appropriate for blocking I/O and should not be pooled. JEP 491, delivered in JDK 24, removed <code>synchronized</code>-related pinning; current JDK 25 guidance identifies native/foreign calls as pinning cases. Old JDK 21 articles are stale on this point. Oracle JDK 25.0.4.1 was the current security baseline on the research date, so a bare `25` image tag is not an adequate production pin.
2. **Structured concurrency is still preview.** <code>StructuredTaskScope</code> is a fifth preview in JDK 25 (JEP 505), while <code>ScopedValue</code> is final (JEP 506). Production preview policy must be explicit.
3. **Kotlin cancellation is cooperative and easy to erase.** Broad exception capture, blocking Java calls, CPU loops, and misused supervisor scopes are the main correctness risks. <code>Dispatchers.IO</code> elasticity and <code>limitedParallelism</code> are not resource quotas.
4. **Backpressure must survive adapters.** Java Flow/Kotlin Flow semantics do not fix SDK callbacks or unbounded body handlers. Lossless token/tool events require bounded, cancel-on-overflow pipelines.
5. **JVM-local sandboxing is unavailable.** The Security Manager is permanently disabled from JDK 24. Risky tools require an OS/container/VM boundary.
6. **Durability is not exactly-once side effects.** Temporal, Restate, Dapr, queues, and databases all require effect identifiers and reconciliation/outbox/compensation at external boundaries.
7. **MCP Java lags the newest protocol snapshot.** Java SDK 2.0.1 tracks MCP 2025-11-25, while the MCP project published a 2026-07-28 release and current Tier 1 SDK list did not include Java. Capability testing is required.
8. **OpenAI documents a Java API helper in beta, not a JVM Agents SDK.** Official Agents SDK documentation links only Python and TypeScript repositories. The Java Responses client still provides useful provider building blocks, but its beta label and different abstraction boundary must remain explicit.
9. **OpenTelemetry Java is stable; GenAI conventions remain development-status.** Auto-instrumentation covers edges, not agent semantics, and sensitive content must remain off by default.
10. **Heap is only one memory budget.** Prompts/results, native JVM memory, direct buffers, child processes, file descriptors, and downstream quotas require independent limits.
11. **Compaction is derived, lossy state.** It must retain source ranges, compactor/schema version, unresolved effects, budgets, and artifact references; it cannot replace the canonical event/effect log.
12. **Java/Kotlin bridges have asymmetric cancellation.** `CompletionStage.await()` attempts to cancel the corresponding future, `runInterruptible` maps coroutine cancellation to thread interruption for cooperative blocking calls, and neither proves a remote effect stopped.

## Baseline and maturity matrix

| Component | Researched state | Production posture |
|---|---|---|
| Java | Java SE 25 API; Oracle 25.0.4.1 security baseline on 2026-08-31 | stable LTS baseline; pin vendor/build/image digest and patch policy |
| Virtual threads | final since earlier JDKs; JDK 25 behavior | use per blocking I/O task; never pool |
| StructuredTaskScope | preview in JDK 25 | internal abstraction or stable fallback |
| ScopedValue | final in JDK 25 | small lexical context, not object cache |
| Kotlin | 2.3 line; 2.3.20 release researched | pin compiler/plugin together; test against separately versioned libraries |
| kotlinx.coroutines | 1.11.0, built against Kotlin 2.2.20 | mature, but verify the exact Kotlin 2.3.20/compiler/plugin/library matrix and audit cancellation |
| kotlinx.serialization | 1.11.0 researched | mature; strict configuration per boundary |
| Jackson | 3.2 current/new-project line; 2.x maintained | follow framework BOM; majors not drop-in |
| OTel Java | traces/metrics/logs stable | stable API/SDK; pin patched agent |
| OTel Kotlin | language status development | Kotlin/JVM can use stable Java implementation |
| OTel GenAI semantics | development | isolate mapping and avoid sensitive content |
| MCP Java SDK | 2.0.1; protocol 2025-11-25 | test against current servers; watch tier/protocol lag |
| Google ADK Java | 1.x Preview/Pre-GA | isolate and verify |
| Temporal Spring AI | public preview | adapter only; streaming unsupported |
| Micronaut LangChain4j | experimental | isolate and pin |

## Java evidence

### Virtual threads

Oracle's Java 25 guide says virtual threads improve throughput for thread-per-request code that spends most time blocked. They do not improve latency or CPU-bound throughput. The guide explicitly says never pool virtual threads; use semaphores for constrained downstream services. It also cautions against ThreadLocal caching because a very large number of virtual threads can multiply storage.

Oracle JDK 25.0.4.1 was the published security baseline on 2026-08-31. The guide therefore distinguishes the stable Java SE 25 API baseline from a deployable vendor patch/build. Production images should pin both the JDK vendor build and container digest and should have a critical-patch-update rollout policy.

JFR exposes virtual-thread start/end, pinned, and submission-failed events. Start/end are disabled by default; pinned is enabled with a threshold. <code>jcmd Thread.dump_to_file</code> supports useful text/JSON thread dumps.

Primary sources:

- [Virtual threads, Java 25](https://docs.oracle.com/en/java/javase/25/core/virtual-threads.html)
- [Oracle JDK 25.0.4.1 release notes](https://www.oracle.com/java/technologies/javase/25-0-4-1-relnotes.html)
- [Thread API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Thread.html)
- [Executors API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/Executors.html)
- [Semaphore API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/Semaphore.html)

### Structured concurrency and interruption

Java 25's <code>StructuredTaskScope</code> opens a lexical scope, forks subtasks (virtual threads by default), joins, and closes with cancellation/wait for unfinished children. The default completion policy is fail-fast. Preview flags are required.

<code>Future.cancel(true)</code> attempts interruption. Blocking <code>wait</code>/<code>sleep</code>/<code>join</code> operations throw <code>InterruptedException</code> and clear status. <code>Thread.interrupted()</code> also clears the current status, while <code>isInterrupted()</code> does not. This supports the guide's “propagate or restore” rule and the distinction between cancellation request and confirmed remote abort.

The JDK 25 `StructuredTaskScope` API adds configuration timeouts, but timeout/cancellation interrupts children and `close()` still waits for them to finish. A non-interruptible subtask can therefore delay scope close indefinitely. This corrected a possible over-reading of “scope timeout” as a hard execution deadline.

Primary sources:

- [Structured concurrency guide](https://docs.oracle.com/en/java/javase/25/core/structured-concurrency.html)
- [StructuredTaskScope API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/StructuredTaskScope.html)
- [Future API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/Future.html)
- [InterruptedException API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/InterruptedException.html)

### HTTP, subprocesses, and security

Java <code>HttpClient</code> should be reused. It offers connect and request timeouts, synchronous/asynchronous sending, and streaming body subscribers. API notes require response streams/publishers to be consumed, cancelled, or closed so resources and shutdown are not stalled. Full-string/byte-array handlers inherently retain the body.

<code>Process</code> documentation warns that limited native pipes can block or deadlock a subprocess if output/error are not consumed. <code>onExit</code> cancellation does not terminate the process. Forced destroy may not complete immediately. <code>ProcessHandle</code> descendant snapshots race with process changes and PID reuse.

Oracle documents the Security Manager as permanently disabled starting in JDK 24; attempting to enable it is an error.

Primary sources:

- [HttpClient API](https://docs.oracle.com/en/java/javase/25/docs/api/java.net.http/java/net/http/HttpClient.html)
- [BodySubscribers API](https://docs.oracle.com/en/java/javase/25/docs/api/java.net.http/java/net/http/HttpResponse.BodySubscribers.html)
- [Process API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Process.html)
- [ProcessHandle API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/ProcessHandle.html)
- [Security Manager permanently disabled](https://docs.oracle.com/en/java/javase/25/security/security-manager-is-permanently-disabled.html)

## Kotlin evidence

Kotlin structured concurrency makes the scope/job tree the lifetime model. <code>GlobalScope</code> creates independent lifetime. Regular child failure cancels the parent; <code>supervisorScope</code> prevents sibling cancellation but still propagates parent cancellation and requires explicit child outcome handling.

Cancellation is cooperative. <code>ensureActive</code> is needed in non-suspending loops. <code>CancellationException</code> is a control signal and should not be converted to a retryable failure.

<code>Dispatchers.IO</code> defaults to a bounded base parallelism but its elastic limited-parallelism views can create additional threads. <code>limitedParallelism</code> limits simultaneous execution on a dispatcher, not active coroutine count or real resources.

Flow is cold and sequential by default. <code>flowOn</code> changes upstream context; <code>buffer</code> introduces concurrent producer/consumer execution. <code>SharedFlow</code> never completes, has replay/buffer semantics, does not deliver producer exceptions, and can lose emissions with no subscribers. These facts motivated explicit terminal events and loss policies.

On JVM, `CompletionStage.await()` is cancellable and cancels the corresponding `CompletableFuture` unless the `asDeferred().await()` form is chosen. `runInterruptible` propagates coroutine cancellation as thread interruption to cooperative blocking code. `withTimeout` is asynchronous and can race with resource return; it also cannot stop a plain blocking call that ignores interruption. These semantics motivated explicit bridge tests and ambiguous-outcome handling.

Primary sources:

- [Coroutines guide](https://kotlinlang.org/docs/coroutines-guide.html)
- [Coroutine basics](https://kotlinlang.org/docs/coroutines-basics.html)
- [Exception handling](https://kotlinlang.org/docs/exception-handling.html)
- [Cancellation and timeouts](https://kotlinlang.org/docs/cancellation-and-timeouts.html)
- [supervisorScope API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/supervisor-scope.html)
- [Dispatchers.IO API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-i-o.html)
- [limitedParallelism API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-dispatcher/limited-parallelism.html)
- [Flow guide](https://kotlinlang.org/docs/coroutines-flow.html)
- [SharedFlow API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/)
- [CompletionStage await](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.future/await.html)
- [runInterruptible](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-interruptible.html)
- [withTimeout](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-timeout.html)

## Serialization and schema evidence

JSON Schema 2020-12 is the current published dialect. It changed tuple keywords to <code>prefixItems</code>/<code>items</code> and includes unevaluated vocabularies. Omitting <code>additionalProperties</code> permits extra properties, which is unsafe as an accidental default for executable tool input.

Kotlin serialization rejects unknown keys by default and exposes explicit null/coercion controls. Its polymorphic serializer requires subtype registration, providing an allowlist. Java reflection <code>Type</code> interop can lose Kotlin nullability.

Jackson 3 reached GA in 2025 and 3.2 was current in researched 2026 material; Jackson 2 remains maintained and widespread. Majors changed packages/group IDs and are not drop-in. Jackson documentation discourages global default typing for untrusted input.

Primary sources:

- [JSON Schema 2020-12](https://json-schema.org/draft/2020-12)
- [Kotlin JSON configuration](https://kotlinlang.org/docs/serialization-json-configuration.html)
- [Kotlin polymorphic serializer](https://kotlinlang.org/api/kotlinx.serialization/kotlinx-serialization-core/kotlinx.serialization/-polymorphic-serializer/)
- [Jackson project status](https://github.com/FasterXML/jackson)
- [Jackson polymorphic deserialization](https://github.com/FasterXML/jackson-docs/wiki/JacksonPolymorphicDeserialization)

## Memory, GC, and runtime evidence

G1 is the default collector and Oracle recommends starting from defaults, usually setting heap and a pause target only when evidence requires it. ZGC is generational in current JDK 25 material and targets very low pauses with concurrent work; maximum heap is the principal tuning input.

The JVM detects container memory/CPU. <code>MaxRAMPercentage</code> defaults to 25%, while <code>ActiveProcessorCount</code> can override processor count used for pool/GC ergonomics. Heap is not RSS.

Native Memory Tracking supports summary/detail and baseline diffs via <code>jcmd</code>, with documented 5–10% overhead and incomplete visibility into external JNI allocations.

Primary sources:

- [G1 collector](https://docs.oracle.com/en/java/javase/25/gctuning/garbage-first-g1-garbage-collector1.html)
- [G1 tuning](https://docs.oracle.com/en/java/javase/25/gctuning/garbage-first-garbage-collector-tuning.html)
- [Java 25 GC tuning guide](https://docs.oracle.com/en/java/javase/25/gctuning/hotspot-virtual-machine-garbage-collection-tuning-guide.pdf)
- [Java runtime options](https://docs.oracle.com/en/java/javase/25/docs/specs/man/java.html)
- [Memory leak and NMT troubleshooting](https://docs.oracle.com/en/java/javase/25/troubleshoot/troubleshooting-memory-leaks.html)

## Context, compaction, and memory evidence

The JVM design separates working prompt context, canonical durable run/effect state, content-addressed artifacts, optional user/domain memory, and audit history. This is an architectural synthesis rather than a claim that one JVM library supplies all five stores. Memory classes are admitted only with an explicit owner, retention/deletion rule, tenant boundary, provenance, prompt-selection policy, and workload evaluation.

The official OpenAI Java Responses reference demonstrates a provider-native compaction operation that returns an opaque compaction item and usage. That item can reduce provider context but cannot replace application-owned effect/approval/budget/event state. The playbook therefore treats provider compaction, application checkpointing, and semantic memory as three distinct mechanisms.

- [OpenAI Java Responses compaction reference](https://developers.openai.com/api/reference/java/resources/responses/methods/compact)
- [Canonical repository context engineering guide](../../context-memory/context-engineering.md)
- [Canonical repository memory architecture guide](../../context-memory/memory-architecture.md)
- [Canonical repository compaction and continuity guide](../../context-memory/compaction-and-continuity.md)

## Observability and testing evidence

OpenTelemetry's language status (updated 2026-08-28 during this review) marks Java traces, metrics, and logs stable, while Kotlin-specific status and Java profiles are development. The Java agent provides broad auto-instrumentation; manual spans remain necessary for run/step/effect semantics. GenAI semantic-convention attributes are development-status and explicitly warn that tool arguments/results may contain sensitive information.

JFR supports configuration profiles and event streaming. Oracle warns enabling all events can generate enormous data. JMX exposes platform MXBeans, but remote management requires secure configuration.

Kotlin <code>runTest</code> supplies virtual time on test dispatchers but cannot prove real parallel schedules or skip time in arbitrary dispatchers. JUnit parallelism is opt-in; separate-thread/preemptive timeouts can break ThreadLocal-bound framework state. Testcontainers' JUnit 5 extension says parallel execution is unsupported. OpenJDK jcstress is probabilistic concurrency testing; JMH recommends standalone forked benchmarks rather than IDE execution.

Primary sources:

- [OpenTelemetry Java](https://opentelemetry.io/docs/languages/java/)
- [OpenTelemetry Java agent](https://opentelemetry.io/docs/zero-code/java/agent/)
- [OpenTelemetry GenAI attributes](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/)
- [JFR guide](https://docs.oracle.com/en/java/javase/25/jfapi/index.html)
- [JFR configuration](https://docs.oracle.com/en/java/javase/25/jfapi/configuration.html)
- [kotlinx-coroutines-test](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-test/)
- [JUnit current guide](https://junit.org/junit5/docs/current/user-guide/)
- [OpenJDK jcstress](https://github.com/openjdk/jcstress)
- [OpenJDK JMH](https://github.com/openjdk/jmh)
- [Testcontainers JUnit 5](https://java.testcontainers.org/test_framework_integration/junit_5/)

## Provider, agent, and MCP evidence

### OpenAI

OpenAI's official SDK page documents its Java API helper as beta. It supports the primary Responses API, sync/async streaming, response accumulation, and structured outputs generated from Java classes. The same official page links Agents SDK repositories only for Python and TypeScript; no official Java Agents SDK is represented there. Some API methods can be language-specific, so generated Java references must be checked.

- [OpenAI SDKs and Agents SDK language boundaries](https://developers.openai.com/api/docs/libraries)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [Java Responses API](https://developers.openai.com/api/reference/java/resources/beta/subresources/responses)
- [Official quickstart](https://platform.openai.com/docs/quickstart/make-your-first-api-request)
- [Java SDK version support](https://github.com/openai/openai-java/blob/main/docs/version-support-policy.md)

### Other provider/framework surfaces

Anthropic documents an official Java base SDK with synchronous/asynchronous/streaming facilities, while managed agent/tool-runner Java functionality was beta in researched material. Google ADK Java remained Preview/Pre-GA; a newer separate Kotlin ADK had appeared. Spring AI's 2.0 line, LangChain4j, Quarkus LangChain4j, and Micronaut integrations provide useful adapters but differ in maturity and concurrency behavior.

- [Anthropic client SDKs](https://platform.claude.com/docs/en/cli-sdks-libraries/overview)
- [Google ADK Java](https://github.com/google/adk-java)
- [Google ADK community/language updates](https://google.github.io/adk-docs/community/)
- [Spring AI reference](https://docs.spring.io/spring-ai/reference/index.html)
- [LangChain4j tools](https://docs.langchain4j.dev/tutorials/tools/)
- [Quarkus LangChain4j](https://docs.quarkiverse.io/quarkus-langchain4j/dev/index.html)
- [Micronaut LangChain4j](https://micronaut-projects.github.io/micronaut-langchain4j/latest/guide/)

### MCP

MCP Java SDK 2.0.1 added bounded STDIO/HTTP reads and tracked protocol 2025-11-25. Version 2.0 made Streamable HTTP primary, deprecated SSE, added Jackson 2/3 abstraction, and validates tool input with JSON Schema 2020-12. The SDK is framework-neutral and provides auth hooks, not built-in authorization.

MCP's 2026-07-28 release removed the initialization handshake and protocol sessions, added `server/discover`, self-describing requests, header routing, cacheable lists, multi-round-trip requests, and authorization hardening. TypeScript, Python, Go, and C# were the four Tier 1 SDKs updated for it; Java 2.0.1 still advertised protocol support only through 2025-11-25, and an assessment issue existed. Consequently the guide records semantic protocol lag instead of implying current parity from shared Streamable HTTP terminology.

- [MCP Java SDK changelog](https://github.com/modelcontextprotocol/java-sdk/blob/main/CHANGELOG.md)
- [MCP Java quickstart](https://github.com/modelcontextprotocol/java-sdk/blob/main/docs/quickstart.md)
- [MCP Java 2.0 migration](https://github.com/modelcontextprotocol/java-sdk/blob/main/MIGRATION-2.0.md)
- [MCP 2026-07-28 release](https://blog.modelcontextprotocol.io/posts/2026-07-28/)
- [MCP SDK tiers](https://modelcontextprotocol.io/community/sdk-tiers)
- [Java tier assessment](https://github.com/modelcontextprotocol/modelcontextprotocol/issues/2301)
- [Historical Java DNS-rebinding advisory](https://github.com/modelcontextprotocol/java-sdk/security/advisories/GHSA-8jxr-pr72-r468)

## Durable runtime evidence

Temporal Java 1.37 material documents deterministic workflow restrictions: no ordinary I/O, native threads/executors, system time/random, or standard locking in workflow code. Activities hold effects. Worker shutdown stops polling and drains received work; non-cooperative work can prevent termination. The Temporal Spring AI integration is public preview and explicitly lacks streaming.

The pass-2 review also found an open maintainer issue for Spring AI 2 / Spring Boot 4 support while the contributed Temporal integration was tied to the older line. This is recorded as a concrete BOM/adoption risk, not generalized into a claim about Temporal core maturity.

Restate documents Java/Kotlin durable services, workflows, virtual objects, durable steps, state, timers, events, and durable futures. Java and Kotlin serialization defaults differ. Dapr documents Java workflow support for platforms already standardized on Dapr.

- [Temporal Java releases](https://github.com/temporalio/sdk-java/releases)
- [Temporal workflow restrictions](https://www.javadoc.io/static/io.temporal/temporal-sdk/1.37.0/io/temporal/workflow/package-summary.html)
- [Temporal WorkerFactory](https://www.javadoc.io/static/io.temporal/temporal-sdk/1.37.0/io/temporal/worker/WorkerFactory.html)
- [Temporal Spring AI preview](https://github.com/temporalio/sdk-java/blob/main/contrib/temporal-spring-ai/README.md)
- [Temporal Spring AI 2 / Spring Boot 4 support issue](https://github.com/temporalio/sdk-java/issues/2920)
- [Restate Java services](https://docs.restate.dev/develop/java/services)
- [Restate durable steps](https://docs.restate.dev/develop/java/durable-steps)
- [Restate Java/Kotlin serialization](https://docs.restate.dev/develop/java/serialization)
- [Dapr Java workflow](https://docs.dapr.io/developing-applications/sdks/java/java-workflow/java-workflow-howto/)

## Build and deployment evidence

Gradle toolchains make the build JDK explicit. Dependency locking pins transitive resolution; dependency verification checks checksums/signatures; repository and wrapper verification are supply-chain controls. Maven supports wrapper distribution/JAR checksums, Enforcer dependency convergence, output timestamps, and reproducible-build checks.

JDK <code>jlink</code> creates custom runtime images but requires module/reflection/agent/diagnostics verification. Kubernetes termination grace includes preStop execution; hooks are at-least-once, terminating endpoints are not ready, and SIGKILL follows grace expiry. JVM hooks can run concurrently or hang, so application shutdown needs bounded phases.

- [Gradle toolchains](https://docs.gradle.org/current/userguide/toolchains.html)
- [Gradle locking](https://docs.gradle.org/current/userguide/dependency_locking.html)
- [Gradle security](https://docs.gradle.org/current/userguide/security.html)
- [Maven Wrapper](https://maven.apache.org/tools/wrapper/index.html)
- [Maven reproducible builds](https://maven.apache.org/guides/mini/guide-reproducible-builds.html)
- [Maven dependency convergence](https://maven.apache.org/enforcer/enforcer-rules/dependencyConvergence.html)
- [jlink](https://docs.oracle.com/en/java/javase/25/docs/specs/man/jlink.html)
- [Kubernetes Pod lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Kubernetes lifecycle hooks](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks)

## Claims deliberately excluded or constrained

- No claim that virtual threads remove the need for admission, pools, or semaphores.
- No current claim that <code>synchronized</code> pins JDK 25 virtual threads.
- No claim that cancellation confirms a remote effect stopped.
- No claim that durable runtimes provide exactly-once external effects.
- No claim that Java has official OpenAI Agents SDK parity.
- No claim that MCP Java implements the newest protocol release.
- No unqualified “latest version” coordinates in operational recommendations.
- No use of Java Security Manager as a sandbox.
- No assumption that Kotlin virtual-time tests expose real races.
- No claim that OpenTelemetry auto-instrumentation understands agent run semantics.

## Refresh triggers

Refresh this packet when any of the following occurs:

- a JDK release changes structured-concurrency/virtual-thread status;
- Kotlin/coroutines changes dispatcher, cancellation, or Flow behavior;
- MCP Java reaches a new spec/tier or authentication/transport revision;
- official JVM Agents SDKs become available from providers;
- OTel GenAI conventions stabilize;
- a selected durable runtime changes replay/versioning/stream support;
- Jackson/framework major versions or BOM compatibility changes;
- a security advisory affects provider, MCP, serialization, telemetry, or build tooling.

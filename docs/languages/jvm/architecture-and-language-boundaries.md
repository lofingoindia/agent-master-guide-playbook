# Architecture and Language Boundaries

## The root design problem

An agent run mixes four categories that fail differently:

1. **Decision computation:** prompt assembly, model calls, parsing, policy.
2. **Effects:** tool calls, messages, writes, payments, deployments.
3. **State transitions:** run status, step history, budgets, approvals.
4. **Transport:** HTTP, streams, queues, subprocess pipes.

Treating these as one callback chain makes cancellation ambiguous, retries unsafe, and recovery dependent on process memory. The smallest reliable design is a state machine around explicit ports.

~~~mermaid
stateDiagram-v2
    [*] --> Admitted
    Admitted --> CallingModel
    CallingModel --> Validating
    Validating --> CallingTool: tool requested
    CallingTool --> Persisting
    Persisting --> CallingModel: continue
    Validating --> Completed: final output
    Admitted --> Cancelled
    CallingModel --> Cancelled
    CallingTool --> Cancelling
    Cancelling --> Cancelled: outcome reconciled
    CallingModel --> Failed
    CallingTool --> Failed
~~~

Persist transitions such as “effect planned,” “effect started,” and “effect outcome recorded.” Do not infer them from logs.

## A narrow core

The core should depend on interfaces shaped by agent semantics, not vendor objects:

~~~java
interface ModelPort {
    ModelResult respond(ModelRequest request, RunContext context) throws ModelFailure;
}

interface ToolPort {
    ToolResult execute(ToolCall call, EffectContext context) throws ToolFailure;
}

interface RunStore {
    RunSnapshot load(RunId id);
    void append(ExpectedVersion version, RunEvent event);
}
~~~

The useful fields in <code>RunContext</code> are stable run and attempt IDs, an absolute deadline, cancellation state, tenant identity, model/tool budgets, and trace context. Do not pass a generic service locator.

## Java and Kotlin: share contracts, not hidden semantics

| Boundary | Safe to share | Keep language-specific |
|---|---|---|
| Domain | immutable IDs, sealed outcome concepts, schema documents | nullability adapters and exception mapping |
| Persistence | event names, versions, optimistic concurrency rules | coroutine versus blocking driver |
| Provider | request/response DTOs where SDK permits | streaming type, cancellation hook, retry defaults |
| Tools | JSON Schema, effect ID, policy decision | <code>Future</code>/<code>Flow</code>/<code>suspend</code> bridge |
| Context | serializable run metadata | <code>ScopedValue</code> versus <code>CoroutineContext</code> |
| Time | absolute epoch deadline in persisted state | monotonic remaining-time calculation |

Kotlin <code>suspend</code> functions do not make blocking Java SDK calls nonblocking. Java <code>CompletableFuture.cancel</code> does not automatically acquire Kotlin structured-concurrency semantics. Put bridges at adapter boundaries and test cancellation on both sides.

### Interop rules

- Expose immutable Java-friendly interfaces from shared modules. Add explicit nullability annotations for Java consumers.
- Do not leak <code>CoroutineScope</code>, <code>Job</code>, <code>Flow</code>, <code>StructuredTaskScope</code>, or framework contexts through the domain layer.
- A Kotlin adapter should move known blocking calls to an appropriate dispatcher; a Java adapter may block a virtual thread.
- Convert cancellation only at one boundary. Preserve Java interrupt status and never translate Kotlin <code>CancellationException</code> into a retryable provider failure.
- Avoid sharing mutable SDK response builders between concurrent branches.

### Make bridges executable policy

The adapter must say what cancellation actually does. For a one-shot Java `CompletionStage`, kotlinx-coroutines' supported bridge cancels the corresponding `CompletableFuture` when the waiting coroutine is cancelled:

~~~kotlin
suspend fun ProviderClient.call(request: Request): Result =
    callAsync(request).await() // cancellation attempts CompletableFuture.cancel(true)
~~~

Use `stage.asDeferred().await()` only when deliberately *not* cancelling the underlying stage. For an interruptible blocking Java API, make the interrupt bridge visible:

~~~kotlin
suspend fun <T> BlockingQueue<T>.takeForRun(): T =
    runInterruptible(Dispatchers.IO) { take() }
~~~

Neither example proves the remote provider or effect stopped. The adapter still closes response bodies when supported and returns an `UNKNOWN` effect outcome when commitment cannot be determined. In the opposite direction, expose a suspending operation to Java from an owned `CoroutineScope.future { ... }`; never create `GlobalScope` merely to obtain a `CompletableFuture`.

Test four bridge states: success, provider failure, parent cancellation before completion, and cancellation racing with a completed external effect. Also verify that Java `InterruptedException` is not wrapped as retryable I/O and Kotlin `CancellationException` is not swallowed by `Result`/broad-catch helpers.

## Minimum event and effect envelopes

Java and Kotlin services should agree on wire contracts, even when their in-process execution differs. A minimal event envelope needs fields with independent meanings:

~~~java
record RunEventEnvelope(
    String runId,
    long sequence,
    long expectedRunVersion,
    String eventType,
    int schemaVersion,
    Instant occurredAt,
    String attemptId,
    String effectId,       // nullable only for non-effect events
    String traceParent,
    String payloadDigest,
    byte[] canonicalPayload
) {}
~~~

- `sequence` orders accepted events for one run; timestamps do not.
- `expectedRunVersion` fences stale writers.
- `schemaVersion` controls deterministic upcasting; it is not the application version.
- `attemptId` changes on worker redelivery; `effectId` remains stable for the logical effect.
- `payloadDigest` protects approvals, replay, and artifact substitution.

An effect record additionally carries canonical request digest, tool/provider version, authorization decision and policy version, lease/fence, remote operation ID, result digest/reference, and `PLANNED | STARTED | SUCCEEDED | REJECTED | FAILED | UNKNOWN` status. Only `SUCCEEDED` and a domain-level `REJECTED` prove a terminal remote outcome; timeout and disconnect commonly produce `UNKNOWN`.

## Run ownership

Use three distinct lifetimes:

| Lifetime | Owner | Examples |
|---|---|---|
| Application | process/container | HTTP client, telemetry SDK, worker pollers |
| Run | admitted request or workflow instance | model/tool children, stream collector |
| Effect | one external operation attempt | provider request, subprocess, database transaction |

Application resources close during shutdown. Run resources close when the run completes or cancels. Effect resources close at each attempt. A global executor or <code>GlobalScope</code> is not an acceptable owner for request work.

## State and concurrency

Prefer a single logical writer per run. Parallel read-only retrieval or independent tools can fork, but their results rejoin through a deterministic merge point. Protect persisted state with a version/compare-and-set token. In-memory locks do not protect against another pod, retry, or workflow replay.

For each effect persist:

- stable effect ID derived from run and logical step, not the process attempt;
- canonical request digest and policy decision;
- status and attempt number;
- provider/tool correlation ID;
- result digest or bounded result;
- ambiguous-outcome marker and reconciliation state.

## Framework boundary

Spring Boot, Quarkus, and Micronaut are useful for configuration, transport, dependency injection, health, and telemetry. Their bean/proxy models should stop at the application shell. The agent loop should remain invocable in a plain test without booting a framework. This keeps durability runtimes and provider SDKs from becoming impossible to upgrade independently.

## Architecture review checklist

- [ ] Can a run resume on a different process from persisted state?
- [ ] Is every child task owned and joined or cancelled?
- [ ] Can two deliveries of the same work avoid duplicate effects?
- [ ] Is time represented as an absolute persisted deadline?
- [ ] Are Java interruption and Kotlin cancellation mapped deliberately?
- [ ] Are provider types confined to adapters?
- [ ] Does every buffer and queue have a capacity and overflow policy?
- [ ] Can tool execution be isolated without changing the core loop?
- [ ] Can telemetry be disabled without changing correctness?

## Sources

- [Java SE 25 structured concurrency](https://docs.oracle.com/en/java/javase/25/core/structured-concurrency.html)
- [Kotlin coroutine context and dispatchers](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html)
- [Kotlin CompletionStage await bridge](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.future/await.html)
- [Kotlin runInterruptible](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-interruptible.html)
- [Temporal Java workflow package constraints](https://www.javadoc.io/static/io.temporal/temporal-sdk/1.37.0/io/temporal/workflow/package-summary.html)
- [Restate durable steps](https://docs.restate.dev/develop/java/durable-steps)

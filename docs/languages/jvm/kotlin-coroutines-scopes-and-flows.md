# Kotlin Coroutines, Scopes, and Flows

## Recommended baseline

Represent a run as a suspending function inside an owned scope. Children inherit its <code>Job</code>, deadline context, tenant identity, and telemetry context. Use <code>coroutineScope</code> when sibling failure should fail the run; use <code>supervisorScope</code> only when failures are intentionally independent and explicitly collected.

~~~kotlin
suspend fun executeRun(run: RunContext): RunResult = coroutineScope {
    val docs = async { retrieveDocs(run) }
    val memory = async { retrieveMemory(run) }
    decide(docs.await(), memory.await(), run)
}
~~~

Avoid <code>GlobalScope</code> for run work. Its independent lifetime breaks cancellation, testing, and shutdown. Application-owned background work should use a named scope with a <code>SupervisorJob</code> and a close path.

## Dispatchers are execution policy

| Work | Appropriate choice | Trap |
|---|---|---|
| suspending HTTP/database client | library's nonblocking implementation | wrapping every call in IO without need |
| known blocking Java call | <code>withContext(Dispatchers.IO)</code> | assuming <code>suspend</code> makes it nonblocking |
| CPU-heavy parsing | <code>Dispatchers.Default</code> or bounded dispatcher | blocking Default with I/O |
| scarce external resource | semaphore/bulkhead | treating <code>limitedParallelism</code> as a resource pool |

<code>Dispatchers.IO</code> is elastic. Its limited-parallelism views can create more threads than the base IO parallelism during peak use. <code>limitedParallelism</code> bounds concurrently executing sections on that dispatcher; it does not bound the number of active coroutines or reserve a provider/database resource. Use a semaphore or the real client pool for those limits.

## Cancellation

Coroutine cancellation is cooperative. Suspending functions generally check it; CPU loops and blocking Java calls may not. Call <code>ensureActive()</code> or <code>yield()</code> in long computation loops and use a cancellation-aware client or explicit close/abort hook around blocking work.

Never swallow <code>CancellationException</code>:

~~~kotlin
try {
    callProvider()
} catch (cancelled: CancellationException) {
    throw cancelled
} catch (failure: IOException) {
    throw RetryableProviderFailure(failure)
}
~~~

Broad <code>catch (Throwable)</code> and convenience wrappers that capture all exceptions can convert cancellation into a normal failure or retry. Audit these paths.

`withTimeout` cancels its child cooperatively and timeout delivery is asynchronous. The timeout can race with a resource becoming available after the block has produced it but before the caller receives it. Acquire closeable resources outside the timeout result expression or use `try/finally` so the losing race cannot leak them. On JVM, `runInterruptible(Dispatchers.IO) { ... }` is the supported bridge for blocking APIs that honor thread interruption; plain `withContext(Dispatchers.IO)` moves blocking work but does not make it interruptible.

~~~kotlin
suspend fun readInterruptibly(queue: BlockingQueue<Event>): Event =
    runInterruptible(Dispatchers.IO) { queue.take() }
~~~

Native calls and libraries that ignore interruption still require a library-specific close/abort hook or process isolation.

## Flow semantics for token and event streams

Flows are cold and sequential by default. Collection owns the upstream execution. <code>flowOn</code> changes the upstream context and may introduce buffering. <code>buffer</code> lets producer and collector run concurrently with a channel between them. Choose capacity and overflow behavior as protocol decisions.

~~~mermaid
flowchart LR
    P[Provider reader] --> B[Bounded buffer]
    B --> A[Assembler]
    A --> C[Client writer]
    B --> X[Cancel and close upstream]
~~~

For lossless model deltas, overflow should suspend producer or cancel the stream; dropping arbitrary deltas corrupts output. For UI progress snapshots, conflation or drop-oldest may be acceptable when events are explicitly snapshots rather than state transitions.

<code>SharedFlow</code> and <code>StateFlow</code> are hot and never complete on their own. Producer failure is not delivered automatically to subscribers. Materialize terminal and failure events, or expose a separate completion <code>Deferred</code>. With no subscribers, an unbuffered shared flow does not backpressure; emitted values can be lost. SharedFlow emission cost also grows with subscriber count.

## Exception topology

- In a regular scope, a non-cancellation child exception cancels the parent and siblings.
- A <code>CoroutineExceptionHandler</code> observes otherwise uncaught root exceptions; it is not a recovery mechanism for <code>async</code>, whose error is delivered by <code>await</code>.
- In a supervisor scope, child failure does not cancel siblings. Each child must still be awaited or have an explicit handler/outcome channel.
- <code>Flow.catch</code> catches upstream exceptions, not failures thrown later by the collector. <code>retry</code> restarts upstream and can repeat side effects unless effect IDs are stable.

## Java interop

Confine bridges to adapters:

- map <code>CompletionStage</code> to coroutine suspension using `CompletionStage.await()`; cancellation attempts to cancel its corresponding `CompletableFuture`;
- use `stage.asDeferred().await()` only when cancellation must stop waiting without cancelling the shared stage;
- expose Kotlin work to Java with `ownedScope.future { ... }`, so cancelling/completing the returned future and the child coroutine remain linked;
- cancellation of either bridge should attempt to cancel/close the Java operation, but the stored outcome remains potentially ambiguous;
- run blocking Java SDK calls on IO or a dedicated dispatcher;
- restore Java interrupt status when catching <code>InterruptedException</code>;
- do not expose Flow as if it were Java Flow without defining demand, error, and completion mapping.

## Production checklist

- [ ] Every scope has a named owner and close path.
- [ ] No run work escapes into GlobalScope.
- [ ] Blocking calls are identified, not guessed.
- [ ] CancellationException is always rethrown.
- [ ] CPU loops check cancellation.
- [ ] Flow buffers have capacities and documented loss semantics.
- [ ] Hot streams materialize terminal state.
- [ ] Supervisor scopes collect every child outcome.
- [ ] Dispatcher limits are not used as external-resource quotas.

## Sources

- [Kotlin coroutines guide](https://kotlinlang.org/docs/coroutines-guide.html)
- [Coroutine basics and structured concurrency](https://kotlinlang.org/docs/coroutines-basics.html)
- [Cancellation and timeouts](https://kotlinlang.org/docs/cancellation-and-timeouts.html)
- [Flow guide](https://kotlinlang.org/docs/coroutines-flow.html)
- [Dispatchers.IO API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-i-o.html)
- [SharedFlow API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/)
- [CompletionStage await](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.future/await.html)
- [runInterruptible](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-interruptible.html)
- [CoroutineScope.future](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.future/future.html)

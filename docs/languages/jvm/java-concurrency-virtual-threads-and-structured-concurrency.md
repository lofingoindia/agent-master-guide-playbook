# Java Concurrency, Virtual Threads, and Structured Concurrency

## Recommended baseline

For I/O-heavy agent runs on JDK 25, use straightforward blocking code on one virtual thread per admitted run. Fork only independent subtasks, and gate every scarce downstream resource separately. This yields readable stack traces and avoids callback graphs without pretending capacity is unlimited.

Virtual threads improve throughput for tasks that mostly block. They do not reduce provider latency, accelerate CPU-bound parsing, or enlarge database/provider quotas. Oracle explicitly recommends creating a new virtual thread per task rather than pooling virtual threads.

~~~java
Future<RunResult> future = runExecutor.submit(() -> runAgent(context));
try {
    return future.get(context.remaining().toMillis(), TimeUnit.MILLISECONDS);
} catch (TimeoutException timeout) {
    future.cancel(true); // cancellation request; adapters still close/reconcile
    throw new RunDeadlineExceeded(timeout);
}
~~~

`runExecutor` is an application-owned `newVirtualThreadPerTaskExecutor()` closed during staged shutdown. Do not construct and close one per request: `ExecutorService.close()` waits for tasks and a non-cooperative child can make the request path hang. Do not use an unbounded admission path merely because virtual threads are cheap.

## Bound the real bottlenecks

~~~mermaid
flowchart LR
    R[Many virtual-thread runs] --> P[Provider semaphore]
    R --> D[Database pool]
    R --> T[Tool-worker quota]
    R --> C[CPU parser pool]
~~~

Use a semaphore or bounded resource pool around provider calls, browser slots, database connections, and subprocesses. For CPU-heavy tokenization, compression, schema generation, or document parsing, use a bounded platform-thread executor sized from measurement.

Do not store heavyweight clients, encoders, or buffers in <code>ThreadLocal</code>. Millions of virtual threads can multiply that cache. Use immutable shared clients or explicit pools. Use <code>ScopedValue</code> for bounded contextual data when it fits lexical call structure; it is final in JDK 25.

## Structured concurrency status

<code>StructuredTaskScope</code> is a preview API in JDK 25, requiring preview compilation and runtime flags. It supplies the right ownership model: fork children inside a lexical scope, join them, apply a completion policy, and close the scope. Closing cancels unfinished subtasks and waits for them.

~~~java
Duration childTimeout = run.childTimeout(CLEANUP_RESERVE); // positive and parent-bounded
var joiner = StructuredTaskScope.Joiner.<Context>allSuccessfulOrThrow();
try (var scope = StructuredTaskScope.open(
        joiner,
        configuration -> configuration
            .withName("run-" + run.id())
            .withTimeout(childTimeout))) {
    scope.fork(() -> retrieveDocs(run));
    scope.fork(() -> retrieveMemory(run));
    return merge(scope.join());
}
~~~

Compile and run preview code with the matching JDK and `--enable-preview`; tests, launch scripts, container entrypoints, and any custom runtime image must all carry the flag. Keep preview types out of public interfaces and persisted data so the implementation can be replaced without a protocol migration.

The JDK 25 default policy is fail-fast: a failed subtask cancels siblings. That is suitable when all inputs are required. Optional retrieval needs an explicit joiner that records each branch outcome; do not weaken the entire run to “best effort” accidentally. A scope timeout requests cancellation, but `close()` still waits for children to terminate. A non-interruptible client can therefore make the lexical scope outlive its nominal timeout. Adapter-level socket/body close and an outer deployment watchdog remain necessary.

If preview APIs are prohibited, reproduce the ownership invariant with a stable application-owned executor. Retain every <code>Future</code>, cancel siblings on failure, and wait only inside a bounded cleanup budget:

~~~java
List<Future<Context>> children = List.of(
    runExecutor.submit(() -> retrieveDocs(run)),
    runExecutor.submit(() -> retrieveMemory(run))
);
try {
    return merge(children.stream()
        .map(future -> getBeforeDeadline(future, run.deadline()))
        .toList());
} catch (Throwable failure) {
    children.forEach(future -> future.cancel(true));
    awaitSettled(children, run.cleanupDeadline());
    throw failure;
}
~~~

The helper names are application contracts: `getBeforeDeadline` derives remaining time from the one run deadline; `awaitSettled` never resets it. The important property is owned children, not a particular concurrency API.

## Interruption is a request, not proof

<code>Future.cancel(true)</code> attempts to interrupt a running thread. Java code cooperates at interruptible blocking points or by checking status. A remote HTTP server, subprocess, or tool may still complete.

Correct handling:

~~~java
try {
    return blockingCall();
} catch (InterruptedException interrupted) {
    Thread.currentThread().interrupt();
    throw new RunCancelled(interrupted);
}
~~~

Propagate <code>InterruptedException</code> where possible. If translating it, restore the interrupt status. Avoid utility code that calls <code>Thread.interrupted()</code> just to inspect state, because that method clears the flag.

## Virtual-thread diagnostics

JDK 25 changed an important old warning: <code>synchronized</code> no longer pins virtual threads after JEP 491. Current pinning is associated with native or foreign-function calls. Do not copy older JDK 21 pinning advice unchanged.

Useful evidence:

- JFR <code>jdk.VirtualThreadPinned</code> is enabled by default with a 20 ms threshold.
- <code>jdk.VirtualThreadStart</code> and <code>jdk.VirtualThreadEnd</code> exist but are disabled by default because volume can be high.
- <code>jcmd Thread.dump_to_file -format=json</code> can represent virtual threads more usefully than a traditional flat dump.
- Carrier-thread saturation, blocked native calls, and downstream semaphore wait time should be measured separately.

Virtual threads are daemon threads. They do not keep the JVM alive. Application lifecycle must be anchored by the server/worker runtime and coordinated shutdown.

## Common failure modes

| Failure | Cause | Correction |
|---|---|---|
| Provider overload after VT migration | admission became effectively unbounded | gate calls and runs explicitly |
| Memory spike | each task retains prompts/results while waiting | cap concurrent runs and payload bytes |
| Cancellation appears successful but side effect occurs | interrupt confused with remote abort | record ambiguous outcome and reconcile |
| CPU latency worsens | CPU work launched per virtual thread | bounded CPU executor |
| Lost context | mutable ThreadLocal or async boundary | immutable context; ScopedValue only lexically |
| Shutdown hangs | executor close waits for non-cooperative work | staged drain, deadlines, forced phase |
| Scope timeout exceeded | child ignores interrupt and `close()` waits | adapter abort/close hook plus outer watchdog |
| Preview upgrade breaks compile | preview API escaped into public/domain code | keep it behind an internal execution port |

## Production checklist

- [ ] JDK patch version is pinned and tested.
- [ ] Preview use is an explicit build/deployment decision.
- [ ] Preview flags are present in compile, test, package, and runtime paths.
- [ ] Admission and each downstream dependency have independent limits.
- [ ] Every future is observed, joined, or cancelled.
- [ ] Interrupts propagate and status is restored when translated.
- [ ] Blocking response bodies and streams close on all paths.
- [ ] CPU-intensive work has a platform-thread budget.
- [ ] JFR and thread-dump procedures are rehearsed under load.

## Sources

- [Oracle virtual threads guide, Java 25](https://docs.oracle.com/en/java/javase/25/core/virtual-threads.html)
- [Oracle structured concurrency guide, Java 25](https://docs.oracle.com/en/java/javase/25/core/structured-concurrency.html)
- [StructuredTaskScope API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/StructuredTaskScope.html)
- [Future cancellation API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/Future.html)
- [Thread interruption API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Thread.html)

# Cancellation, Deadlines, and Run Control

## One deadline, many attempts

A timeout duration recreated at every layer silently expands the run budget. Admit a run with an absolute wall-clock deadline, persist it, and derive a remaining duration from a monotonic clock inside the process.

~~~mermaid
sequenceDiagram
    participant I as Ingress
    participant L as Agent loop
    participant P as Provider
    participant T as Tool
    I->>L: deadline = 12:00:30Z
    L->>P: remaining minus reserve
    P-->>L: response
    L->>T: smaller remaining deadline
    T-->>L: result or ambiguous timeout
    L-->>I: terminal state before deadline
~~~

Reserve time for cancellation, persistence, and client response. A child deadline must never exceed the parent's.

At process admission/resume, convert the persisted wall deadline into a monotonic budget once, clamp negative values to zero, and pass the same budget object down the call tree:

~~~java
record RunBudget(Instant wallDeadline, long deadlineNanos) {
    static RunBudget resume(Instant deadline, Clock wall, LongSupplier ticker) {
        Duration remaining = Duration.between(wall.instant(), deadline);
        Duration bounded = remaining.isNegative() ? Duration.ZERO
            : remaining.compareTo(MAX_RUN_DURATION) > 0 ? MAX_RUN_DURATION : remaining;
        long nanos = bounded.toNanos();
        long now = ticker.getAsLong();
        long expires = now > Long.MAX_VALUE - nanos ? Long.MAX_VALUE : now + nanos;
        return new RunBudget(deadline, expires);
    }

    Duration remaining(LongSupplier ticker) {
        return Duration.ofNanos(Math.max(0L, deadlineNanos - ticker.getAsLong()));
    }
}
~~~

`MAX_RUN_DURATION` is an admission-policy limit. The important rule is that retry and child adapters receive `remaining()`, never the original timeout duration. Recreate the monotonic anchor only after process recovery from the persisted `Instant`.

## Cancellation phases

1. **Request:** mark the run cancelling and stop admitting new steps.
2. **Propagate:** interrupt/cancel children; close response streams; request subprocess termination.
3. **Wait:** join children within a bounded cleanup budget.
4. **Reconcile:** determine whether remote effects completed.
5. **Persist:** record cancelled, failed, or unknown outcome.

Cancellation is not rollback. If an external call crossed its commit boundary, compensate or reconcile it.

## Java mapping

- <code>Future.cancel(true)</code> requests interruption.
- Interruptible waits throw <code>InterruptedException</code> and clear the status. Propagate it or restore status before translating.
- Closing an HTTP body/stream is often the actionable cancellation mechanism.
- Cancelling <code>Process.onExit()</code> does not kill the subprocess.
- Structured-scope close cancels unfinished tasks and waits, but children still need cooperative cleanup.

## Kotlin mapping

- Parent <code>Job</code> cancellation propagates to children.
- <code>withTimeout</code> is cooperative; it cannot forcibly stop arbitrary blocking Java/native code.
- Rethrow <code>CancellationException</code>.
- Place non-cancellable cleanup only around the smallest must-complete local operation. Do not hide network calls in <code>NonCancellable</code>.
- A timeout during an external effect creates an unknown outcome unless the remote protocol supplies a definitive cancellation result.

## Budget model

Track at least:

| Budget | Enforcement point |
|---|---|
| wall time | run registry and every outbound call |
| model calls/tokens/cost | before and after provider operation |
| tool calls | before effect planning |
| output bytes/events | transport decoder and stream buffer |
| parallel children | run scope |
| subprocess CPU/memory/time | isolated worker runtime |

Budget checks must be atomic with step admission when concurrent branches exist. Persist consumed and reserved budget so retrying delivery cannot reset limits.

## Ambiguous outcomes

An operation is ambiguous when the caller times out or disconnects after sending a request but before learning whether it committed. Record the effect as <code>UNKNOWN</code>, not failed. Recovery order:

1. Query by idempotency key or provider operation ID.
2. Reconcile domain state.
3. Retry only if the remote contract makes that safe.
4. Otherwise require compensation or human decision.

| Observation | Safe persisted interpretation |
|---|---|
| rejected before request bytes/effect dispatch | not started; retry may be safe |
| provider returned a terminal domain rejection | rejected; do not transport-retry |
| response/body closed after request was sent | unknown unless remote lookup proves outcome |
| coroutine/future cancelled while effect ran | cancellation requested; effect may be unknown |
| worker crashed after remote commit, before local record | planned/started effect requiring reconciliation |

## Control-plane operations

Pause, approve, cancel, and resume should use optimistic concurrency against a run version. Repeated cancel is idempotent. Resume must verify that no unresolved effect can be duplicated. Approval tokens bind run ID, step/effect ID, tool, argument digest, policy version, approver, and expiry; changing arguments invalidates approval.

## Failure checklist

- [ ] Absolute deadline is persisted.
- [ ] Remaining time uses a monotonic clock in-process.
- [ ] Cleanup and persistence have reserved time.
- [ ] Cancellation prevents new model/tool work immediately.
- [ ] Each adapter has an abort/close strategy.
- [ ] Unknown outcomes are distinct from failures.
- [ ] Budget consumption survives redelivery.
- [ ] Pause/resume and approval are versioned state transitions.

## Sources

- [Java Future API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/Future.html)
- [Java InterruptedException API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/InterruptedException.html)
- [Kotlin cancellation and timeouts](https://kotlinlang.org/docs/cancellation-and-timeouts.html)
- [Kotlin ensureActive API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/ensure-active.html)
- [Java Process API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Process.html)

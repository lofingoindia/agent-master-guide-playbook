# Testing, Virtual Time, Concurrency, and Load

## A layered verification strategy

| Layer | Proves | Does not prove |
|---|---|---|
| pure state-machine unit tests | transitions, budgets, schema mapping | real SDK/transport behavior |
| transcript/golden tests | provider event mapping and tool assembly | network timing |
| integration tests | HTTP/MCP/store/workflow contracts | production load |
| concurrency stress | races and memory-order outcomes | end-to-end capacity |
| JMH | isolated JVM operation cost | service throughput |
| load/soak/fault tests | capacity, retention, recovery | every schedule |

Avoid “the model usually returns X” assertions. Test protocol invariants and normalized outcomes with captured, redacted fixtures. Live provider tests should be small, budgeted, non-blocking to most CI, and evaluated statistically or by schema/invariant.

## Deterministic core tests

Drive the loop with fake clock, model, tool, state store, and effect registry. Cover:

- repeated tool-call cycles and step limits;
- invalid/truncated structured output;
- cancellation before/during/after each effect boundary;
- duplicate delivery and version conflict;
- provider retry followed by deadline;
- unknown effect reconciliation;
- approval replay with changed arguments;
- crash after external commit but before local persistence.

Property tests can generate event sequences and assert terminal-state, budget, and idempotency invariants.

## Kotlin virtual time

<code>runTest</code> and <code>TestCoroutineScheduler</code> skip delays on test dispatchers and allow controlled time advancement. Inject dispatchers/schedulers so child coroutines share the scheduler.

Limits:

- code moved to a real/default/IO dispatcher may use real time;
- a single-threaded test dispatcher does not expose true parallel races;
- virtual time does not prove blocking SDK cancellation;
- hot flows need explicit collection lifetime and terminal event assertions.

Use real-thread integration tests for cancellation races and dispatcher starvation.

A useful virtual-time test asserts the state transition, not merely that a timeout exception was thrown:

~~~kotlin
@Test
fun timeout_marks_started_effect_unknown() = runTest {
    val provider = FakeEffect(commitAt = 90.milliseconds, replyAt = 2.seconds)
    val result = runner(provider, clock = schedulerClock)
        .execute(deadline = 100.milliseconds)

    assertEquals(EffectStatus.UNKNOWN, result.effect.status)
    assertTrue(result.run.isTerminal)
    assertEquals(1, provider.logicalEffectIds.distinct().size)
}
~~~

The fake models the commit/reply split; a simplistic fake that cancels before commit cannot test ambiguity. Keep a second real HTTP/process test because virtual time cannot prove socket close, thread interruption, or subprocess cleanup.

## Java time and concurrency

Inject a <code>Clock</code> for wall-clock deadlines and a monotonic ticker abstraction for elapsed time. Do not make production correctness depend on sleeping in unit tests. Use latches/barriers to coordinate a specific schedule.

Use OpenJDK jcstress for small shared-state algorithms. It is probabilistic and needs repeated runs; forbidden outcomes require careful grading. Prefer proven JDK concurrent primitives over inventing lock-free structures.

JUnit parallel execution is opt-in. Shared fixtures, ports, static mocks, and Testcontainers can make parallel tests unsafe. The Testcontainers JUnit extension documents parallel execution as unsupported. JUnit preemptive/separate-thread timeouts can bypass ThreadLocal-bound transactions or context; same-thread timeout/interrupt plus a suite-level watchdog is safer for many framework tests.

## Performance tests

Use JMH only for isolated JVM work such as parsing, schema generation, serialization, or buffer transformations. Run as a forked standalone benchmark with warmup; IDE runs are less reliable. JMH cannot predict provider latency or queue saturation.

End-to-end load profiles should include:

- fast and slow provider streams;
- high token/tool output sizes;
- throttles, 5xx, resets, and stalled bodies;
- mixed CPU-heavy and blocking tools;
- cancellation storms and pod draining;
- duplicate queue delivery and worker crash;
- telemetry enabled;
- cold start and long soak.

Measure useful concurrency, semaphore wait, heap/RSS, GC, queue delay, error/unknown-effect rate, and cost—not only requests per second.

### Failure-injection matrix

| Injection point | Required assertion |
|---|---|
| before request bytes | no `STARTED` effect; bounded retry may proceed |
| after remote commit, before reply | effect becomes `UNKNOWN`; reconciler finds one outcome |
| mid-stream after client output | no blind generation retry; client cursor can resync |
| after state commit, before queue ack | redelivery observes terminal/versioned state; no second effect |
| expired lease while old worker continues | old fence cannot persist or acknowledge |
| TERM during tool process | admission stops, child is drained/killed, outcome persists |
| telemetry exporter stall | run correctness continues; bounded telemetry drops/queues |
| slow/non-interruptible Java dependency | run deadline fires; outer watchdog prevents infinite drain |

Inject at the transport/store/process boundary, not with random `sleep` calls. Give every injection a deterministic latch or failpoint and record the exact state transition expected before and after restart.

### Capacity model

Ramp three axes independently: admitted runs, provider/tool latency, and retained bytes per run. Stop increasing load when any production limit reaches its budget—even if request throughput is still rising. A release candidate should demonstrate:

- steady state below the target provider/tool semaphore utilization;
- bounded queue age and stream-buffer bytes;
- heap and process RSS headroom through a long soak;
- cancellation and shutdown completion within their reserves;
- no growth in unresolved effects, classloaders, file descriptors, child processes, or coroutine/thread ownership leaks.

## Release gates

- deterministic suite passes under Java and Kotlin adapter variants;
- provider fixture compatibility tests cover every streamed event type used;
- schema golden files are reviewed;
- cancellation integration tests leave no process/socket/run orphan;
- duplicate-delivery tests prove one logical effect;
- load test respects SLO and memory headroom;
- shutdown drill finishes inside deployment grace period;
- JFR and telemetry evidence show no unexplained saturation.

## Sources

- [kotlinx-coroutines-test API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-test/)
- [JUnit current user guide](https://junit.org/junit5/docs/current/user-guide/)
- [OpenJDK jcstress](https://github.com/openjdk/jcstress)
- [OpenJDK JMH](https://github.com/openjdk/jmh)
- [Testcontainers JUnit 5 lifecycle](https://java.testcontainers.org/test_framework_integration/junit_5/)

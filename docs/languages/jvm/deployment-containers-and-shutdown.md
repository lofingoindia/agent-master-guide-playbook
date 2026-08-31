# Deployment, Containers, and Shutdown

## Graceful shutdown is a protocol

~~~mermaid
sequenceDiagram
    participant K as Orchestrator
    participant A as Agent service
    participant Q as Queue/provider/tool
    K->>A: termination begins
    A->>A: readiness false; stop admission
    A->>Q: stop polling / cancel safe work
    A->>A: drain or checkpoint runs
    A->>A: flush bounded state/telemetry
    A-->>K: process exits
    K--xA: SIGKILL after grace if still alive
~~~

Sequence:

1. mark terminating and reject new runs;
2. stop queue/workflow polling;
3. close listeners or drain connections;
4. allow short work to finish;
5. cancel/checkpoint longer work;
6. persist unknown effects and release leases;
7. close HTTP clients, executors, telemetry, and stores;
8. exit before the orchestrator's hard deadline.

Make repeated shutdown calls safe. Bound every phase so one stuck tool cannot consume the entire grace period.

A coordinator should make the phase budget visible:

~~~java
void stop(Duration grace) {
    if (!stopping.compareAndSet(false, true)) return;
    Deadline deadline = Deadline.after(grace);
    readiness.rejectNewRuns();
    pollers.stop();
    listeners.drain(deadline.slice(0.20));
    runs.checkpointOrCancel(deadline.slice(0.55));
    effects.persistUnknownAndReleaseLeases(deadline.slice(0.15));
    resources.close(deadline.remaining());
}
~~~

`slice` is an application helper that cannot extend the parent deadline. Closing telemetry is last and bounded; correctness state must not depend on a successful exporter flush. The forced phase records active run/effect/process IDs before abandoning cleanup so recovery has evidence.

## JVM shutdown behavior

Shutdown hooks run concurrently and can hang indefinitely. Keep one coordinator hook that delegates to normal lifecycle code; do not create dependent hooks with implicit order. Hooks are not guaranteed on forced termination or process/host failure.

<code>ExecutorService.close()</code> performs orderly shutdown and waits for termination. That can block forever when work ignores interruption, so production lifecycle code usually needs explicit <code>shutdown</code>, bounded <code>awaitTermination</code>, then <code>shutdownNow</code> and diagnostics.

Virtual threads are daemon threads and do not keep the process alive. A cancelled <code>CompletableFuture</code> or <code>Process.onExit()</code> is not proof that remote/child work terminated.

## Kubernetes details

Kubernetes sends TERM and later KILL when <code>terminationGracePeriodSeconds</code> expires. The grace period includes <code>preStop</code> time; a long hook steals application drain time. Hooks are intended to be delivered at least once, so handlers must be idempotent.

Terminating endpoints are marked not ready, but propagation and existing connections still require application-side draining. Set grace from measured high-percentile checkpoint/drain time plus margin. Test actual ingress/service-mesh behavior.

Health semantics:

- startup: initialization complete enough for liveness checks;
- readiness: safe to receive new work and below overload threshold;
- liveness: process cannot make progress without restart.

Do not fail liveness merely because a provider is down or the service is overloaded; restart storms worsen both.

## Container sizing

Pin the image digest and run as a non-root user with a read-only filesystem where practical. Set explicit CPU and memory requests/limits based on load evidence. JDK container ergonomics detect limits, but heap is only part of RSS. Reserve native headroom and account for sidecars.

CPU quotas affect <code>availableProcessors</code>, GC, ForkJoinPool, and dispatcher sizing. <code>-XX:ActiveProcessorCount</code> can override detected CPU for partitioning, but use only when measurement shows detection/topology mismatch.

Store durable state outside the container. Scratch files and JFR recordings need bounded writable storage. Configure heap-dump behavior so an OOM does not fill the node or expose secrets.

## Scale by bottleneck, not thread count

Compute a per-pod admission ceiling from the minimum of independent budgets:

<code>min(memory-safe runs, provider permits, database capacity, tool slots, CPU-safe work)</code>

Then apply a global/tenant limiter so adding replicas cannot multiply provider or tool traffic beyond contractual limits. Queue consumers should scale on queue age plus downstream headroom, not backlog length alone. Keep poll concurrency below executable capacity; “prefetch everything, then wait on semaphores” hides queue time in heap and makes shutdown slower.

Useful scale signals are sustained queue age, admission rejection, provider/tool permit wait, CPU saturation, retained bytes per run, and unresolved-effect age. Raw virtual-thread or coroutine count is diagnostic context, not a capacity target. Scale-to-zero is unsuitable when local long-lived streams, leases, or non-checkpointed work still exist.

During a rollout, old and new pods must share a global quota and compatible state/event/tool versions. Canary model/provider changes separately from JVM or framework changes where possible; otherwise a regression is difficult to attribute.

## Rolling upgrades

Deployments must tolerate old and new workers simultaneously:

- event/state readers accept versions in the rollout window;
- tool schemas and approval digests remain stable or versioned;
- workflow code uses supported versioning/replay mechanisms;
- queue messages are backward compatible;
- leases fence old owners;
- model/provider behavior changes are feature-gated and canaried.

## Failure drills

Test TERM during model streaming, tool execution, state commit, queue acknowledgement, approval wait, and workflow activity. Repeat with SIGKILL/node loss to prove persisted recovery. Inspect for orphan subprocesses, leaked leases, duplicate effects, and unflushed unknown outcomes.

## Checklist

- [ ] Admission stops before draining.
- [ ] Pollers and listeners close early.
- [ ] Each phase has a deadline and forced fallback.
- [ ] Shutdown hooks are minimal and order-independent.
- [ ] Kubernetes grace includes preStop plus JVM drain.
- [ ] Liveness excludes dependency outage and ordinary overload.
- [ ] Heap/native/disk/child-process headroom is measured.
- [ ] Per-pod and global/tenant admission limits remain correct as replicas change.
- [ ] Rolling versions can read shared state/messages.
- [ ] TERM and SIGKILL recovery drills pass.

## Sources

- [Java Runtime shutdown API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Runtime.html)
- [ExecutorService API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/ExecutorService.html)
- [Kubernetes Pod lifecycle and termination](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Kubernetes container lifecycle hooks](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks)
- [Kubernetes probes](https://kubernetes.io/docs/concepts/workloads/pods/probes/)
- [Java container options](https://docs.oracle.com/en/java/javase/25/docs/specs/man/java.html)

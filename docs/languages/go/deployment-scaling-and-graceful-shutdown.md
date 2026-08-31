# Go Deployment, Scaling, and Graceful Shutdown

> **Last researched:** 2026-08-31  
> **Use with:** [Context, deadlines, and shutdown](context-deadlines-and-shutdown.md)

A graceful Go shutdown is an ordered ownership transfer: stop new admission, make the instance unroutable, stop acquiring durable work, notify active owners, persist required receipts, close long-lived connections, flush bounded telemetry, and exit before the platform sends a forced kill.

## Separate health, readiness, and admission

| Signal | Meaning |
|---|---|
| Startup | Initialization required before serving is complete |
| Liveness | Process is making enough progress that restart is useful |
| Readiness | Instance should receive new traffic/work |
| Admission | Application has capacity/policy to start this particular run |

Do not fail liveness because a provider is temporarily down; a restart may amplify load without fixing the dependency. Readiness can reflect inability to serve all essential traffic, but overload is usually better handled by bounded admission, retry guidance, and autoscaling signals.

On termination, mark not ready/stop admission immediately. Kubernetes marks terminating endpoints not ready for regular traffic, but connection draining and external load-balancer propagation still require testing.

## Use one process supervisor

`main` should own servers, queue consumers, stream registries, exporters, and background refreshers. A top-level context handles the first termination signal; a separate drain context owns cleanup.

```go
rootCtx, stop := signal.NotifyContext(
	context.Background(), os.Interrupt, syscall.SIGTERM,
)
defer stop()

g, serviceCtx := errgroup.WithContext(rootCtx)
g.Go(func() error { return api.Serve(serviceCtx) })
g.Go(func() error { return workers.Run(serviceCtx) })

<-rootCtx.Done()
admission.Stop()

drainCtx, cancel := context.WithTimeout(context.Background(), drainBudget)
defer cancel()

shutdownErr := errors.Join(
	workers.StopAndDrain(drainCtx),
	api.Shutdown(drainCtx),
	streams.CloseAndWait(drainCtx),
	telemetry.Shutdown(drainCtx),
)
runErr := g.Wait()
```

Real code should handle spontaneous service failure as well as signals: if a critical server exits unexpectedly, cancel siblings and begin shutdown. Avoid `log.Fatal` in child goroutines because it exits immediately and skips deferred cleanup.

## Order the drain

```mermaid
sequenceDiagram
    participant P as Platform
    participant A as Admission/readiness
    participant H as HTTP/streams
    participant W as Workers
    participant T as Telemetry
    P->>A: termination signal
    A->>A: not ready; reject new runs
    A->>W: stop polling/acquiring leases
    A->>H: stop listeners; notify streams
    W->>W: finish/checkpoint/release within budget
    H->>H: drain HTTP; close WebSockets/SSE
    A->>T: bounded flush
    A-->>P: process exits
```

Recommended ordering:

1. stop admission and readiness;
2. stop new queue leases and scheduled work;
3. call `http.Server.Shutdown` to stop listeners and drain ordinary active connections;
4. notify active runs according to product policy;
5. close/wait for WebSockets and other hijacked/upgraded connections explicitly;
6. checkpoint or abandon durable work safely, releasing/allowing leases to expire as designed;
7. stop subprocess/sandbox supervisors and verify descendants;
8. flush telemetry within the remaining budget;
9. force close remaining resources and exit non-zero if correctness requires operator attention.

`Server.RegisterOnShutdown` can initiate protocol-specific shutdown but the registered function should not wait. The main shutdown owner performs the wait.

## Fit inside the platform grace period

Kubernetes normally sends a termination signal and enforces `terminationGracePeriodSeconds`; a `preStop` hook consumes the same grace-period budget. When it expires, remaining processes are forcibly killed.

Therefore:

```text
application drain budget
  < termination grace
    - preStop duration
    - endpoint/LB propagation allowance
    - signal/scheduler margin
    - final force-close/telemetry allowance
```

Do not use `preStop: sleep` as the only drain mechanism without measuring routing behavior. Keep the Go process in control of correctness-critical checkpointing and connection closure.

## Scale on the constrained resource

CPU utilization is often a poor sole autoscaling signal for I/O-heavy agent services. Useful signals include:

- admission rejection/defer rate and queue age;
- active runs and permits by provider/tool class;
- pending queue depth/oldest message;
- provider rate/token quota and latency;
- active stream count and pending bytes;
- memory working set per admitted workload class;
- subprocess/sandbox utilization;
- durable activity/task backlog and schedule-to-start latency;
- CPU throttling, scheduler delay, and GC pressure.

Scaling replicas does not increase a shared provider quota or database capacity. Coordinate global limits outside individual processes where oversubscription would be dangerous. Per-process semaphores multiply by replica count.

## Avoid sticky in-memory correctness

Long streams and chats can tempt sticky routing. Stickiness may improve cache/session locality, but correctness should not depend on one process retaining state. Use durable run/session state and reconnect identities. A rolling deployment, node failure, or autoscaler termination should not erase the only copy of an approval, effect receipt, or terminal result.

For MCP and WebSocket connections, version compatibility and reconnect behavior must be explicit. Existing long-lived connections can keep old code alive through a rollout; set maximum connection age or a graceful reconnect protocol when deployments require convergence.

## Roll out durable workers safely

Durable workflow code can outlive deployments. Before rollout:

- replay retained histories against the new worker code;
- use engine-supported Worker Versioning/patching/deployment pinning;
- keep old payload readers and workflow versions available for retention duration;
- ensure queue concurrency limits do not accidentally count or starve old pending versions;
- verify old and new workers agree on effect IDs, schemas, and error envelopes;
- provide rollback without replay nondeterminism.

An ordinary stateless canary does not validate months-long workflow replay compatibility.

## Shutdown failure matrix

| Failure | Expected behavior |
|---|---|
| Second signal arrives | Escalate to force close/exit by documented policy |
| HTTP handler ignores cancellation | Drain deadline expires; force close; capture goroutine evidence |
| WebSocket stays open | Protocol close, bounded wait, force close registry entry |
| Worker holds lease beyond grace | Checkpoint/heartbeat/cancel; allow safe redelivery; fence old attempt |
| Subprocess has descendants | Sandbox/process-tree supervisor terminates all or process reports failure |
| Telemetry backend unavailable | Bounded flush, record/drop, never exceed termination grace |
| Provider call commits after cancellation | Fence state and reconcile by effect/receipt |
| Load balancer still routes briefly | Admission/readiness rejects new runs while existing drain |

## Go-live checklist

- [ ] Startup, liveness, readiness, and admission have distinct semantics.
- [ ] One supervisor owns all servers/workers/background goroutines.
- [ ] Critical child failure triggers coordinated process shutdown.
- [ ] Shutdown stops admission before draining work.
- [ ] HTTP, SSE, WebSocket, MCP, queue, durable, subprocess, and telemetry owners are all included.
- [ ] Application drain budget fits the platform grace period with margin.
- [ ] Autoscaling observes queue/resource pressure, not only CPU.
- [ ] Per-process limits are reconciled with global provider/database quotas.
- [ ] Durable state and reconnect do not depend on sticky routing.
- [ ] Rolling upgrade, forced kill, node loss, and second-signal escalation are rehearsed.

## Selected primary sources

- [`os/signal.NotifyContext`](https://pkg.go.dev/os/signal#NotifyContext)
- [`http.Server.Shutdown`](https://pkg.go.dev/net/http@go1.27.0#Server.Shutdown)
- [Kubernetes Pod termination](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination)
- [Kubernetes probes](https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/)
- [Container-aware `GOMAXPROCS`](https://go.dev/blog/container-aware-gomaxprocs)


# Deployment, Workers, Shutdown, and Crash Recovery for Python Agents

> **Research date:** 2026-08-31  
> **Related:** [Deployment, release, and incident response](../../operations/deployment-release-and-incident-response.md) and [durable execution](../../runtime/durable-execution.md)

Deployment topology is part of application semantics. Each ASGI worker has its own interpreter, heap, event loop, clients, pools, semaphores, caches, background tasks, and in-memory sessions. A signal or crash can arrive between external commit and local receipt. Design shutdown and recovery before choosing worker counts.

## Start with one process and measured capacity

Scale worker processes only after measuring one worker's:

- concurrent run/stream capacity;
- event-loop lag and CPU saturation;
- RSS/heap/native memory and file descriptors;
- HTTP/database pool usage;
- executor/process/native thread counts;
- shutdown/drain duration;
- startup/import/model/tool registry cost.

Then compute pod/host totals. Four ASGI workers each with 100 HTTP connections, 32 threads, a process pool, and native library threads can overwhelm a small container even if each default looks reasonable.

Keep in-memory state worker-local and disposable. Put durable sessions, queue leases, cluster quotas, idempotency records, and checkpoints in external systems.

## Package the ASGI application cleanly

Expose an importable application or factory. Create event-loop-bound clients/pools/supervisors inside ASGI lifespan, not at packaging/import time or before worker fork. Fail startup if configuration/migrations/connectivity required for safe service are invalid; do not declare readiness early.

Use the server/process manager's documented worker model. Uvicorn's built-in multi-process manager uses `spawn`, which works on Windows as well as Unix. The former `uvicorn.workers` integration is deprecated in favor of the separate `uvicorn-worker` package when Gunicorn integration is selected. Pin and test exact server/worker versions; do not copy an old deployment command without checking its lifecycle behavior.

## Separate health signals

| Probe/signal | Meaning |
|---|---|
| Startup | Initialization has not irrecoverably failed; delays liveness/readiness until complete |
| Liveness | Process/runtime is making enough progress to be restarted if false |
| Readiness | New work can be admitted safely now |
| Deep dependency status | Operator evidence; not necessarily every liveness request |

Readiness must become false before drain starts. It should consider admission saturation and mandatory dependencies, but avoid flapping on every transient provider error. Liveness should not restart a healthy but temporarily overloaded process and amplify an outage.

## Use a two-phase shutdown

```mermaid
sequenceDiagram
    participant O as Orchestrator
    participant S as ASGI/worker supervisor
    participant R as Run owners
    participant D as Durable stores
    O->>S: TERM / shutdown event
    S->>S: mark unready; stop admission/leasing
    S->>R: request cancellation or durable handoff
    R->>D: checkpoint / outbox / reconcile
    S->>S: close streams, clients, executors; flush bounded telemetry
    alt settled before grace
      S-->>O: exit 0/intentional
    else hard deadline
      O->>S: KILL
      Note over D: new worker recovers from persisted evidence
    end
```

Recommended ordering:

1. mark unready and stop accepting/leasing new agent work;
2. stop local producers and close admission queues normally;
3. signal run owners with reason/deadline;
4. finish safe short work; checkpoint/handoff long work;
5. fence terminal state and reconcile ambiguous effects;
6. close downstream streams and HTTP/database/MCP clients;
7. stop thread/process/interpreter pools with escalation;
8. flush telemetry within a small explicit budget;
9. exit before the orchestrator's hard grace deadline.

Reserve time for steps 5–8. If the platform grace is 30 seconds, do not allow every run a 30-second cleanup.

## Signals are platform/runtime-specific

Python signal handlers execute in the main thread of the main interpreter, and only that thread can install handlers. Available signals differ on Windows. The handler should notify the event-loop/service supervisor rather than perform blocking cleanup or acquire ordinary locks.

`asyncio.Runner` handles SIGINT by cancelling the main task so `finally` blocks can run; a second interrupt can escalate immediately. ASGI servers and process managers install their own handlers—integrate with their lifespan/shutdown hooks instead of competing signal frameworks.

`atexit` is not a drain mechanism. Python documents that handlers do not run for unhandled signals, fatal internal errors, or `os._exit()`, and interpreter finalization can block starting/joining threads. Correctness must survive without it.

## Align Kubernetes and server grace periods

Kubernetes normally runs `preStop` (if configured), sends TERM or the image stop signal, waits `terminationGracePeriodSeconds`, then sends KILL. The grace period includes hook and application shutdown time. Keep a safety margin between:

```text
server graceful timeout
  < container/pod termination grace
  < deployment/controller rollout timeout
```

Use `preStop` only for a specific coordination need; a blind sleep consumes grace and does not prove endpoints have drained. Mark readiness false immediately in application shutdown and test load-balancer/proxy routing during termination.

Kubernetes probe configuration needs startup allowance for slow imports/migrations and liveness thresholds longer than expected transient event-loop stalls, while still detecting true hangs.

## Treat hard kill and crash as normal recovery inputs

Assume cleanup did not run. A new owner should:

1. claim an expired lease or compare-and-set the run version;
2. load the last durable checkpoint/event/effect ledger;
3. detect attempts with intent but no authoritative receipt;
4. reconcile by effect ID/provider request ID;
5. resume/retry only replay-safe steps;
6. fence the old worker from future writes;
7. publish one recovery/terminal transition.

Do not replay a model/tool step merely because the checkpoint precedes it. The external call may have happened. Store intent before effect and receipt after effect; ambiguity is a first-class state.

## Worker recycling is containment, not repair

Uvicorn/Gunicorn-style maximum request counts (preferably jittered where supported) can bound leak lifetime. Process pools can use tasks-per-child. Celery has max tasks/memory per child. These controls:

- do not close leaked resources correctly;
- can increase cold starts and synchronized churn;
- can interrupt long agent runs;
- require durable handoff/recovery;
- can hide a worsening leak until traffic changes.

Use them as a safety layer while diagnosing ownership/allocation evidence.

## Deployment verification matrix

| Event | Verify |
|---|---|
| Rolling update under long streams | unready before stop; reconnect/cancel policy; no new work on draining pod |
| SIGTERM during provider/tool await | bounded cancellation and close; durable state/effect fence |
| SIGKILL after external commit | recovery reconciles without duplicate effect |
| Worker crash/native segfault | supervisor restarts; pool/lease state is cleanly reconstructed |
| Dependency startup failure | readiness never true; useful sanitized diagnostic |
| Overload | bounded 503/429/queueing; no event-loop/RSS collapse |
| Telemetry backend outage during shutdown | exit remains inside grace; drops are counted |
| Process recycling | in-memory state loss is harmless; sessions/limits remain correct |
| Windows and POSIX | spawn/signals/process groups behave as documented for each target |

## Go-live checklist

- [ ] Worker/process/thread/pool multiplication fits pod CPU, memory, and fd limits.
- [ ] All event-loop resources are lifespan-owned and created after worker start.
- [ ] Readiness drops before admission/leasing stops; liveness is not overload detection.
- [ ] Shutdown phases and per-phase budgets fit inside platform grace with reserve.
- [ ] `atexit`/finalizers are not required for correctness.
- [ ] Hard kill at every effect phase has a tested recovery outcome.
- [ ] Old workers are lease/version-fenced after recovery.
- [ ] Streams reconnect by durable cursor or cancel explicitly.
- [ ] Recycling is jittered/observable and does not strand runs.
- [ ] Rollback artifact and state/schema compatibility are proven.

## Selected primary sources

- [ASGI lifespan protocol](https://asgi.readthedocs.io/en/latest/specs/lifespan.html)
- [Uvicorn settings](https://uvicorn.dev/settings/) and [server behavior](https://uvicorn.dev/server-behavior/)
- [`uvicorn-worker`](https://github.com/Kludex/uvicorn-worker)
- [Python signals](https://docs.python.org/3.14/library/signal.html), [`asyncio.Runner`](https://docs.python.org/3.14/library/asyncio-runner.html), and [`atexit`](https://docs.python.org/3.14/library/atexit.html)
- [Kubernetes Pod termination](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination) and [startup/readiness/liveness probes](https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/)

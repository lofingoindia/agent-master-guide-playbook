# Deployment, Containers, Serverless, Edge, and Shutdown

> **Last researched:** 2026-08-31  
> **Production baseline:** Node.js 24.20.0 LTS  
> **Related:** [Deployment, release, and incident response](../../operations/deployment-release-and-incident-response.md)

“JavaScript runtime” is not a deployment guarantee. A full Node process, a serverless Node handler, and a V8 edge isolate have different APIs, lifetime, concurrency, streaming, filesystem, background-work, and shutdown semantics. Test the exact target and compatibility date.

## Choose the target by required capabilities

| Target | Good fit | Major constraints |
|---|---|---|
| Long-lived Node service/container | Agent gateways, long streams, workers, subprocesses, native packages, queue consumers | You own capacity, security, shutdown, patching |
| Serverless Node function | Short/bounded stateless turns, bursty APIs, platform-integrated events | Duration, freeze/reuse, per-instance concurrency, background work, payload/stream rules |
| Edge isolate/runtime | Placement-sensitive lightweight routing/stream transforms | Partial/stubbed Node APIs, CPU/memory/code limits, no ordinary process lifecycle |
| Durable/serverless workflow | Long waits, retry/recovery, signals | Engine-specific determinism, state, cost, versioning |

Use edge because measured placement value outweighs compatibility/operational cost, not as a label. Use a full Node service when the agent needs native addons, `worker_threads`, child processes, durable worker SDKs, long-lived connections, or exact Node diagnostics.

## Build a reproducible container

- pin Node 24 patch and base-image digest;
- use a multi-stage build and frozen dependency install;
- copy only runtime artifacts and production dependencies;
- run as non-root with a read-only root filesystem where possible;
- mount only required writable/diagnostic/temp paths with quotas;
- define CPU/memory/PID/file-descriptor limits;
- use exec-form `CMD ["node", "..."]`, not `npm start`, so signals reach Node directly;
- use a lightweight init (`--init`/Tini/dumb-init) when Node would be PID 1 and child reaping/signal behavior requires it;
- include health/readiness endpoints that do not depend on overloaded business pools;
- retain image SBOM, lock, Node/npm/native dependency versions, and digest.

Alpine/musl images can reduce size but change native-addon compatibility and allocator/runtime behavior. Choose through tested dependency and operational needs, not image size alone.

## Size memory and CPU inside the real quota

Use `os.availableParallelism()` as an input for worker sizing, but still verify CPU quota/throttling behavior. Set worker/process concurrency below a measured safe ceiling. V8 heap must leave headroom for external memory, native libraries, worker isolates, thread stacks, TLS buffers, and diagnostics.

The kernel/container OOM killer may terminate without a JavaScript exception or cleanup. Shed earlier and persist durable state before hard limits.

## Scale the constrained resource, not the JavaScript label

Establish a safe per-replica envelope under the real quota before adding replicas:

- maximum admitted run working-set bytes and RSS high-water shedding point;
- provider/model starts and concurrency by quota;
- main-loop CPU/event-loop-delay limit;
- worker/process slots and queue wait;
- database/cache/Undici connection budget;
- live SSE/WebSocket sessions and send-backlog bytes;
- queue lease/prefetch count and shutdown return time.

Horizontal scaling does not remove a shared external cap. Ten replicas each opening the pool default can overwhelm a provider or database. Derive per-replica connections, starts, and concurrency from the fleet budget, reserve repair/control capacity, and change them together with autoscaling bounds.

Use queue age for durable backlog, admission wait/rejection for interactive pressure, and the actual saturated pool/provider/CPU/memory signal for capacity decisions. Request count alone treats a tiny lookup like a long streaming tool run. Event-loop delay is an overload safety signal, but scaling on it alone can multiply a downstream outage; combine it with CPU, pool waits, retry rate, and dependency health.

Long-lived streams make load distribution sticky. Include connection age, reconnect storms, drain duration, and per-replica event backlog in rollout capacity. New replicas must not become ready until required pools, credentials, policies, schemas, and telemetry are initialized; old replicas become unready before they stop owning new work.

## Implement readiness-aware two-phase shutdown

```mermaid
sequenceDiagram
    participant O as Orchestrator
    participant N as Node supervisor
    participant R as Runs/workers
    participant D as Dependencies
    O->>N: SIGTERM / lifecycle stop
    N->>N: not ready; stop admission/polling
    N->>R: cancel, checkpoint, or relinquish by policy
    N->>N: close listeners; drain streams
    R-->>N: owned tasks settled
    N->>D: close dispatchers/pools/exporters
    alt grace nearly exhausted
        N->>R: terminate remaining workers/processes
    end
    N-->>O: exit before hard kill
```

Kubernetes starts the pod grace-period countdown before `preStop`; a long hook consumes time Node needs after SIGTERM. Keep hooks lightweight and make the application independently handle SIGTERM. Set the pod grace longer than worst-case application drain plus margin. `preStop` delivery is at least once, so cleanup must be idempotent.

Readiness should turn false before or at admission stop, and load-balancer propagation time must be included. Existing keep-alive/HTTP2/WebSocket/SSE connections may continue after listeners close. Track and drain them, then force close by policy.

Process order:

1. single-flight shutdown state; handle repeated signal;
2. not ready and stop accepting runs;
3. stop queue/durable polling and return/relinquish unstarted work;
4. cancel request-bound runs; checkpoint/detach durable runs;
5. close HTTP server admission and active upgrade owners;
6. settle run task groups and effects;
7. drain/terminate worker threads and child process trees;
8. close Undici/database/cache clients;
9. flush telemetry inside a smaller deadline;
10. exit naturally or set exit code—do not call `process.exit()` before buffered I/O/cleanup settles.

The `exit` event supports synchronous operations only. `uncaughtExceptionMonitor` can capture evidence without suppressing the required crash. Let a supervisor replace fatal processes.

## Serverless Node has invocation ownership

Do not assume a handler continues ordinary background work after returning. Platform behavior can freeze, reuse, or terminate the environment. Persist/enqueue background work through the platform's supported mechanism and await all work required for the response/effect contract.

AWS Lambda illustrates important Node-runtime distinctions:

- execution environments can be frozen and later reused; unfinished callbacks can resume on a later reuse;
- background work should complete before handler completion;
- response streams should use `pipeline()` and end before return;
- starting with Node 24, Lambda no longer waits for unresolved promises after the handler returns/stream ends;
- client disconnect does not necessarily stop a streamed invocation or billing;
- ordinary Lambda shutdown has little/no application cleanup window without extensions; hard kill behavior is platform-owned.

These are AWS-specific examples, not universal serverless behavior. For every target verify maximum duration, first-byte/stream duration, body sizes, concurrency per instance, freeze/thaw, connection reuse, `/tmp`, environment reuse, signal/shutdown, and background task APIs.

Warm reuse means process-global pools/caches can be beneficial, but they must validate stale sockets/credentials and never retain tenant/run state across invocations unintentionally.

## Edge compatibility is partial and date-sensitive

Cloudflare Workers documents supported, partial, and non-functional stub Node modules. At a 2026-08-04 or later compatibility date, Node compatibility modes are enabled by default, yet modules such as `child_process`, `cluster`, `worker_threads`, `v8`, and inspector can still be importable stubs rather than functional Node equivalents. A package can bundle/import successfully and fail only when a method executes.

Vercel's Edge runtime similarly exposes Web APIs and only a subset of Node behavior, with platform-specific first-byte/stream duration and dynamic-code restrictions.

Therefore:

- pin compatibility date/flags;
- inventory fs/net/tls/http2/worker/child/native/inspector/async-context use;
- run the built artifact in the provider's local/preview and real environment;
- execute every required method, not just import modules;
- test streaming, disconnect, background work, CPU/memory, and observability;
- keep durable state external;
- maintain a Node-runtime fallback when dependencies require full Node.

## Rolling releases and runtime upgrades

Node patch updates can include security behavior changes. Major upgrades change V8, Undici, module/native ABI, deprecations, and performance. Release with:

- Node 24 exact patch canary;
- Node 26 compatibility lane before its LTS promotion;
- native-addon and lockfile clean-build matrix;
- provider/stream/backpressure/retry/effect tests;
- mixed old/new worker and durable-execution compatibility;
- rollback artifact, database compatibility, and old-run routing;
- observability segmented by Node/release version.

Do not float `node:24` or `latest` in production without a controlled rebuild/promotion process.

## Deployment verification

- [ ] Exact Node patch, package manager, base image, native build, lock, and image digest are pinned.
- [ ] Non-root identity, filesystem, network, secrets, and process limits match the tool threat model.
- [ ] Worker count and heap leave measured container headroom.
- [ ] Per-replica and fleet-wide provider/database/dispatcher/stream/worker budgets stay bounded during autoscaling.
- [ ] Signals reach Node and descendants are reaped/terminated.
- [ ] Readiness/admission/polling stop before drain; hard grace exceeds drain budget.
- [ ] Active HTTP/SSE/WebSocket, workers, processes, pools, and telemetry close in order.
- [ ] Serverless work completes or moves to a supported durable/background mechanism.
- [ ] Edge builds execute all required APIs on the exact compatibility date.
- [ ] Rolling Node/runtime upgrades preserve durable runs and support rollback.

## Selected primary sources

- [Official Node Docker best practices](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md)
- [Kubernetes container lifecycle hooks](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks)
- [Node.js process lifecycle](https://nodejs.org/api/process.html#process-events)
- [AWS Lambda execution environment lifecycle](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html)
- [AWS Lambda Node response streaming](https://docs.aws.amazon.com/lambda/latest/dg/config-rs-write-functions.html)
- [Cloudflare Workers Node compatibility](https://developers.cloudflare.com/workers/runtime-apis/nodejs/)
- [Vercel Edge runtime](https://vercel.com/docs/functions/runtimes/edge)

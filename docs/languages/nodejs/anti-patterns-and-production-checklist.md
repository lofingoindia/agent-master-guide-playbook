# Anti-Patterns and Production Checklist

> **Last researched:** 2026-08-31  
> **Scope:** Failure patterns and release gate for the Node.js runtime area

This guide compresses the recurring ways Node agent services fail. Use the linked guides for mechanisms and evidence.

## Architecture and lifecycle anti-patterns

| Anti-pattern | Production consequence | Better design |
|---|---|---|
| Treat every async call as independently safe | Synchronous segments block all runs | Bound/measure main-loop work; offload sustained CPU |
| Ignore returned promises | Hidden failures and work outliving runs | Owned task registry, abort policy, final settlement |
| `Promise.race` as timeout without cancellation | Losing work continues and may commit late | Composed signal, tracked cleanup, result/effect fence |
| Request/socket is the source of truth | Deploy/disconnect loses long run | Explicit request-bound policy or durable run ownership |
| Resume after `uncaughtException` | Process state may be undefined/corrupt | Capture minimal evidence, exit, external restart |
| Async cleanup in `exit` handler | Cleanup never completes | Start from SIGTERM/platform lifecycle before `exit` |
| `unref()` as cleanup | Work/memory remains but stops keeping process alive | Cancel/close/join the owned resource |

## I/O and streaming anti-patterns

| Anti-pattern | Production consequence | Better design |
|---|---|---|
| New `fetch` dispatcher/client per request | No connection reuse; socket/handshake pressure | Process-owned bounded Agent/Pools |
| Unlimited Pool connections/origins | FDs, ports, memory, upstream overload | Per-origin connection and origin/admission ceilings |
| Response body neither consumed nor cancelled | Connection/resource leak | Read, pipeline, cancel, or destroy in every path |
| Call `.json()` on unbounded body | Heap spike/OOM/DoS | Wire/decompressed byte cap and streaming parser/artifact |
| Ignore `write() === false` | Slow-client memory growth | Await drain/pipeline; explicit backlog policy |
| Count objects, not bytes | One element can be huge | Aggregate byte permits/budgets |
| SSE ID stored only in process memory | Reconnect fails after deploy | Durable cursor/event retention or explicit no-resume contract |
| WebSocket send without backlog/auth policy | Memory and stale privileged sessions | Frame/rate/buffer/auth-expiry/heartbeat limits |

## CPU and tool anti-patterns

| Anti-pattern | Production consequence | Better design |
|---|---|---|
| Worker per request/tool | Startup, thread, isolate, RSS exhaustion | Long-lived bounded pool and queue |
| Raise `UV_THREADPOOL_SIZE` blindly | Higher memory/contention, no main-loop fix | Measure exact libuv workload under quota |
| Use worker thread for routine network I/O | Complexity and cloning with little benefit | Main-loop async I/O |
| Treat worker/`vm` as sandbox | Hostile code retains process authority/risk | OS sandbox/container/microVM/VM |
| Concatenate model output into shell string | Command injection | Operation registry + `spawn`/`execFile` arguments |
| Buffer unlimited child stdout/stderr | Parent OOM/deadlock | Concurrent drains, byte caps, artifact spool |
| Kill only direct child | Descendants outlive tool | Tested process group/Job Object/container ownership |

## State and reliability anti-patterns

| Anti-pattern | Production consequence | Better design |
|---|---|---|
| A queue implies exactly once | Duplicate effects after crash/lease loss | Idempotent handler, stable effect ID, reconciliation |
| CPU-heavy queue handler on main loop | Lease renewal stalls; duplicate execution | Offload/bound CPU and monitor loop delay |
| Retry at SDK + app + queue + workflow | Attempt/cost explosion | One retry owner and total deadline/attempt budget |
| Retry POST after timeout with new ID | Duplicate external action | Stable downstream idempotency key/reconcile |
| Treat timeout/cancel as no effect | Committed action hidden | Mark ambiguous and query effect ledger/downstream |
| Store opaque SDK/framework objects durably | Upgrade/replay incompatibility | Versioned JSON-safe domain records/receipts |
| Deploy incompatible workflow code freely | Replay or old-run failure | Behavior versioning, patch/migration/old-worker policy |

## Memory, observability, and security anti-patterns

| Anti-pattern | Production consequence | Better design |
|---|---|---|
| Watch only `heapUsed` | Miss external/native/worker/RSS exhaustion | RSS + heap + external + workers + container metrics |
| Admit after parsing/retrieval | Overload materializes before rejection | Byte caps and admission first |
| Put transcript/context in `AsyncLocalStorage` | Timers/promises/listeners retain the full graph and ambient state crosses unrelated layers | Small immutable correlation store; versioned context owned by the run |
| Append every summary as “memory” | Context grows, contradictions persist, deletion/provenance becomes impossible | Versioned compaction plus policy-authorized long-term-memory promotion |
| Run ID/URL/error text as metric labels | Cardinality explosion | Bounded dimensions; details in sampled logs/traces |
| Record raw prompts/tools by default | Secret/PII/data leakage | Metadata-first capture and explicit redaction/retention |
| Ambient async context grants authority | Stale/confused authorization | Explicit capability and effect-time reauthorization |
| Permission model is the sandbox | Bypass/DoS/native/process risk remains | OS/container controls plus Node defense in depth |
| `npm ci` means safe dependencies | Frozen graph still executes reviewed/unreviewed code | Script/git/native policy, isolated build, audit/provenance/SBOM |
| Import success proves edge compatibility | Stubbed methods fail only at runtime | Execute exact artifact/API on pinned compatibility date |
| Upgrade a provider SDK on compile/unit success | Hidden retry, stream, transport, telemetry, or packaging behavior changes | Black-box adapter adoption suite with wire attempts and fault injection |

## Production release gate

### Runtime and ownership

- [ ] Production uses an exact supported Node 24 LTS patch; Node 26 is a separate compatibility lane until LTS.
- [ ] One process supervisor owns listeners, pools, workers, queue consumers, signals, and shutdown.
- [ ] Every run has attempt identity, absolute deadline, composed abort, task ownership, budgets, and terminal fence.
- [ ] No hidden/fire-and-forget promise can mutate completed run state.
- [ ] Event-loop delay/ELU and libuv/worker queues are load-tested separately.

### I/O and streaming

- [ ] Undici/SDK versions are recorded and patched; dispatchers, origins, connections, and pending work are bounded.
- [ ] Every response body is consumed, piped, cancelled, or destroyed.
- [ ] Connect, headers, stream-idle, attempt-total, and run-total deadlines are distinct.
- [ ] Node/Web Streams honor backpressure; every queue has element and byte limits.
- [ ] SSE/WebSocket slow-consumer, reconnect, auth, terminal event, and shutdown behavior are proven.

### Tools and durable work

- [ ] CPU work uses a bounded worker/process boundary; native and child-process failure domains are explicit.
- [ ] Model-selected tools map to allowlisted typed operations; no shell construction exists.
- [ ] Hostile/generated code runs under OS-level isolation and least privilege.
- [ ] Queue duplicates, lease loss, poison jobs, and worker crashes are tested.
- [ ] Mutating effects use stable IDs, receipts, and ambiguous-outcome reconciliation.
- [ ] Long-lived behavior/state/event schemas and worker versions have migration/rollback policy.

### Memory, diagnostics, and security

- [ ] Admission occurs before expensive materialization and reserves global/tenant/control capacity.
- [ ] Peak RSS/heap/external/buffer/worker/native/process memory stays below the container envelope.
- [ ] Context, event, response, tool output, stream backlog, and diagnostic artifacts have byte quotas.
- [ ] Compaction preserves source IDs/critical state; long-term memory has provenance, expiry/correction, and deletion policy.
- [ ] Async context crosses workers/processes/queues explicitly and carries no implicit authorization.
- [ ] Metrics have bounded labels; content capture, reports, and snapshots follow security/retention policy.
- [ ] Permission model, OS identity, fs/network/secret policy, and sandbox boundary are tested.
- [ ] Lockfile, frozen install flags, script/git/native policy, audit, signatures/provenance, SBOM, and digests pass.

### Testing and deployment

- [ ] Cancellation is injected at every wait/effect boundary and cleanup returns resources.
- [ ] Slow consumers, connection churn, provider stalls, event-loop CPU, libuv saturation, worker crashes, and OOM pressure are tested.
- [ ] Kill/restart tests cover every state/effect/ack boundary.
- [ ] Soak tests show resources return near a known baseline.
- [ ] Container/serverless/edge behavior is tested on the exact runtime, image, proxy, and compatibility date.
- [ ] Readiness, stop-admission, polling stop, run drain/checkpoint, pool close, telemetry flush, and forced termination fit inside the real grace period.
- [ ] A rollback artifact and mixed old/new durable-worker test exist.
- [ ] Provider/tool/telemetry SDK upgrades pass abort, retry, stream, body-cleanup, packaging, and exact-runtime tests.

## Minimum incident evidence

When a run fails, retain enough to distinguish runtime from provider behavior:

- run/attempt/effect/tool/checkpoint IDs and release/Node/Undici versions;
- terminal class, error code/cause chain, abort source, and phase;
- queue/admission/pool wait and retry count;
- event-loop delay/ELU, CPU, memory, stream backlog, and connection/worker state;
- effect receipt or ambiguity/reconciliation status;
- shutdown/restart/container termination reason where applicable.

Keep evidence bounded and redacted. More telemetry is not better when it prevents the service from recovering or leaks tenant data.

## Related guides

- [Architecture, event loop, and run ownership](architecture-event-loop-and-run-ownership.md)
- [Cancellation, deadlines, and shutdown signals](cancellation-deadlines-and-shutdown-signals.md)
- [HTTP, Undici, connections, and timeouts](http-undici-connections-and-timeouts.md)
- [Queues, state, and durable workers](queues-state-and-durable-workers.md)
- [Deployment, serverless, edge, and shutdown](deployment-containers-serverless-edge-and-shutdown.md)
- [Node.js research packet](../../research/packets/nodejs-agent-runtime-deep-dive.md)

## Selected primary sources

- [Node.js: Don't block the event loop (or the worker pool)](https://nodejs.org/en/learn/asynchronous-work/dont-block-the-event-loop)
- [Node.js: Process events](https://nodejs.org/api/process.html#process_event_uncaughtexception)
- [Node.js: Worker threads](https://nodejs.org/api/worker_threads.html)
- [Undici: Dispatcher and response-body lifecycle](https://github.com/nodejs/undici#garbage-collection)
- [Node.js: Permission model](https://nodejs.org/api/permissions.html)
- [npm audit and verification](https://docs.npmjs.com/cli/audit/)
- [Kubernetes: Container lifecycle hooks](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/)

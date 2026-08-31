# Research Packet: Node.js Agent Runtime Deep Dive

> **Status:** Research-backed synthesis  
> **Research date:** 2026-08-31  
> **Scope:** Production Node.js runtime behavior for agent APIs, gateways, streams, tools, queue consumers, and durable workflow workers  
> **Method:** Current Node 24 LTS and Node 26 Current documentation/source/release material was cross-checked with libuv, Undici, npm, web standards, official workflow/queue projects, Kubernetes, serverless, edge-runtime, and OpenTelemetry sources. Issue reports and advisories are treated as adoption-test leads, not universal guarantees.  
> **Output:** [Node.js agent runtime engineering](../../languages/nodejs/README.md)

## Research questions

1. What does Node's single-event-loop architecture require from an agent runtime that owns many concurrent long-lived runs?
2. Where do JavaScript callbacks, network I/O, libuv work, worker threads, child processes, native addons, and durable workers actually execute and fail?
3. How should cancellation, deadlines, stream backpressure, connection pooling, retry, idempotency, memory admission, and graceful shutdown compose?
4. Which Node 26 features are Current-only and must not be described as part of the Node 24 LTS production baseline?
5. Which platform surfaces are stable, experimental, release-candidate, deprecated, or provider-specific as of 2026-08-31?
6. What must be tested before a Node agent service, serverless function, or edge build can be considered production-ready?

## Snapshot and confidence

| Item | Checked position on 2026-08-31 | Confidence and refresh trigger |
|---|---|---|
| Production Node line | Node 24.20.0 is Active LTS (Krypton), released 2026-08-26. Maintenance is scheduled for 2026-10-20 and EOL for 2028-04-30. | High; refresh at every Node security release and the maintenance transition. |
| Current Node line | Node 26.8.1 is Current, released 2026-08-26. LTS transition is scheduled for 2026-10-28; EOL is scheduled for 2029-04-30. | High for dated status; refresh at the LTS transition or schedule change. |
| Release policy | Production apps should use Active or Maintenance LTS. Up through v26, even majors transition to LTS after Current. Starting with v27, the cadence becomes annual with six months Alpha, six months Current, then LTS for every major. | High; release schedule dates are explicitly subject to change. |
| Core async/runtime | Stable event loop/libuv, promises, `AbortSignal`, Node streams, Web Streams, worker threads, child processes, `AsyncLocalStorage`, diagnostics channels, performance hooks, diagnostic reports. | High; exact method availability/stability still requires the target patch docs. |
| HTTP client | Built-in `fetch` is powered by the Undici version embedded in Node; installed Undici has a separate release/security lifecycle. Node 26 began with Undici 8, Node 24 with Undici 7. | High; refresh on Node/Undici update or transport/security advisory. |
| Test runner | Core runner and mock timers are stable; snapshots stable. Watch mode and built-in coverage remain experimental; module mocking is early development. Node 26 adds newer diagnostics/tags/coverage behavior. | High; refresh when stability labels change. |
| Permission model | Stable defense-in-depth surface. Node 24.20 added audit mode and `permission.drop()`; Node 26 documents a wider scope set including network/FFI/OpenSSL authority. It is not a malicious-code sandbox. | High for exact 24.20/26.8 docs; refresh every patch/major because scopes and constraints evolve. |
| Module hooks | Synchronous `module.registerHooks()` is release-candidate; asynchronous `module.register()` is deprecated in Node 26 and carries loader-thread/CommonJS caveats. | High; refresh before any hook-dependent architecture decision. |
| Durable TypeScript worker | Temporal's TypeScript SDK currently lists official support for Node 20, 22, and 24, and says worker features rely on authentic Node-specific APIs/native modules. | High for current README; refresh before Node 26 adoption. |
| Queue semantics | BullMQ documents at-least-once processing and duplicate execution when lock renewal fails, including from event-loop stalls. | High for documented product semantics; refresh by chosen version. |
| OpenTelemetry JS | Traces and metrics Stable; logs Development. Semantic conventions can have separate maturity. | High; refresh when logs/conventions change status. |
| Edge compatibility | Cloudflare and Vercel provide platform-specific subsets. Cloudflare can expose importable non-functional stubs; compatibility date materially changes behavior. | High for principle and dated provider docs; refresh every platform compatibility change. |

## Research boundary

This packet covers runtime and operations, not TypeScript compiler/schema design. Native type stripping, `tsc`, source transformations, JSON Schema, and runtime shape validation remain linked to [TypeScript/Node.js agent runtimes](../../languages/typescript-node-agent-runtimes.md). The Node deep dive assumes JavaScript is executing and asks how that process owns time, work, I/O, memory, failure, and authority.

## Finding 1: Node 24 LTS and Node 26 Current must remain separate baselines

Node's official release page says production applications should use Active or Maintenance LTS. Node 26 was released Current on 2026-05-05 and is scheduled—not guaranteed—to enter LTS on 2026-10-28. Describing v26 as production LTS before that date would be incorrect.

The practical policy is:

- pin an exact Node 24 patch for production;
- keep Node 26.8.1 in CI/canary as a compatibility and future-LTS lane;
- version-gate 26-only methods and diagnostics;
- record the embedded Undici/V8/npm/native ABI with the release;
- do not use floating container tags as uncontrolled runtime updates;
- stage Node security patches because they can change HTTP/TLS/permission behavior while remaining within LTS.

Examples found during research:

- Node 26.7 added `IncomingMessage.signal`; it is not a Node 24 baseline API.
- Node 26.5 added a per-iteration mode to `monitorEventLoopDelay()`; its values are not comparable to interval sampling.
- Node 26.1 added test-runner diagnostics-channel events; later 26 minors add tags/coverage behavior.
- Node 26 runtime-deprecates asynchronous `module.register()`.
- Node 24.20 added permission audit and `permission.drop()`; Node 26 still has wider permission scopes. Guides must not treat all 26 flags/channels as a 24 baseline.
- Node 24.20 added `node:stream/iter`, but it is Stability 1 and requires `--experimental-stream-iter`; landing in an LTS patch does not make it the stable stream baseline.

Starting with v27, the official policy moves to one major per year. v27 Alpha begins in October 2026, Current in April 2027, and LTS in October 2027. Every major is intended to reach LTS. This changes future upgrade planning but does not change v26's Current status at the research date.

## Finding 2: event-loop fairness is a correctness property

Node's learn guide and libuv design agree on the execution model: JavaScript callbacks run on one event-loop thread; network I/O readiness is polled by that loop; selected filesystem/DNS/crypto/zlib/native work uses libuv's global pool. Fairness is largely an application responsibility because one long callback delays all other clients.

Agent-specific consequences are unusually broad:

- synchronous prompt construction/JSON/schema/regex work delays tokens for unrelated runs;
- promise/`nextTick` chains can starve later I/O phases;
- abort timers and disconnect callbacks are observed late;
- queue lock/lease renewal can be missed and create duplicate execution;
- HTTP connection establishment can appear timed out while the event loop is CPU-starved;
- readiness/shutdown handling can be delayed;
- stream backpressure signals cannot be acted on while callbacks monopolize the loop.

Therefore event-loop delay and utilization are service-level health signals, not optional performance trivia. They must be correlated with process CPU, worker/libuv queues, GC, pool waits, and container throttling. One metric alone cannot identify the cause.

## Finding 3: Node has no built-in structured concurrency for promises

Starting an async operation creates no automatic parent-child relationship. Ignoring its promise does not cancel it. `Promise.all()` observes a first rejection but does not cancel siblings; `Promise.allSettled()` joins outcomes but supplies no stop policy.

A production run owner needs:

- one composed abort tree and absolute deadline;
- a bounded registry of owned task promises;
- sibling-failure policy;
- `allSettled`-style cleanup/join on exit;
- stable run/attempt/effect IDs;
- resource permits retained until actual cleanup;
- a fenced durable terminal transition.

Fire-and-forget is legitimate only when ownership moved to another supervised/durable component. Logging `.catch()` on a detached promise is failure observation, not lifecycle ownership.

## Finding 4: cancellation is cooperative and timers are observationally late

Stable `AbortSignal.timeout()` and `AbortSignal.any()` make signal composition simpler. Node recommends checking `aborted`, using one-shot listeners, and preserving cleanup; `events.addAbortListener()` is stable and returns a disposable.

What cancellation does depends on the layer:

- Fetch/Undici can abort transport/body activity.
- `pipeline()` can destroy participating streams.
- timers/promises reject.
- a plain promise does nothing unless its hidden operation supports cancellation.
- workers need an application cancellation message and optional termination.
- child processes receive a configured signal, but descendant ownership remains OS/application work.
- durable workflows/queues persist intent according to their own contract.

`Promise.race(work, timeout)` without passing cancellation leaves the losing work running. Even a correctly scheduled abort timer can execute late under event-loop starvation, so instrumentation should record deadline time and observed-abort time.

Cancellation cannot prove a remote effect did not commit. The runtime must fence late results and reconcile stable effect IDs.

## Finding 5: backpressure must traverse the entire data path

Node streams and WHATWG Web Streams are stable and interoperable through adapters. `stream.pipeline()` is the preferred composition primitive because it coordinates errors, closure, and backpressure, including Web Streams in modern Node.

However:

- high-water marks are thresholds, not hard memory limits;
- object mode counts objects, not bytes;
- SDKs, transforms, run queues, proxy layers, and clients can buffer independently;
- a producer can ignore the next layer's capacity;
- one small slice can retain a large Buffer backing store;
- slow-client output can retain a whole agent run after provider completion.

The guide set therefore requires per-run and process-wide event/byte budgets, typed semantic events, token-delta coalescing, and an explicit slow-consumer policy.

SSE's standard `id`/`Last-Event-ID` behavior helps reconnect only if the server has durable event identity and retention. WebSocket adds bidirectional backpressure, heartbeats, auth expiry, acknowledgements, replay, and close-handshake ownership; it is not reliability for free.

## Finding 6: Undici dispatcher lifetime is an architectural decision

Undici's `Agent` owns per-origin dispatchers. `Pool` spreads concurrent work across clients/connections. Current documentation shows defaults that can be effectively unbounded (`connections: null`, `maxOrigins: Infinity`) unless configured. Dynamic tool destinations make origin bounds security and resource controls.

Production guidance:

- own/reuse dispatchers at process scope;
- bound origins, per-origin connections, pending attempts, and application concurrency;
- separately control credentials/proxy/trust-zone pools;
- consume or destroy every response body;
- cap decoded bytes before `.json()`/`.text()`;
- measure acquisition, connect, headers, body idle, semantic stream idle, and total attempt/run time;
- close dispatchers gracefully; destroy only for forced shutdown.

HTTP/1.1 pipelining greater than one is disabled by default and recommended only with trusted targets. For most provider traffic, bounded multiple connections with pipelining one is the simpler baseline.

HTTP/2 requires deliberate connection caps to benefit from multiplexing; otherwise an unlimited pool can create more client connections. Exact Undici version/support must be verified.

## Finding 7: timeout and retry internals require adoption tests

Current Undici docs define connect, headers, and body timeouts, but timer precision and parser/backpressure behavior are implementation-specific. A 2026 Undici issue reports body-timeout suppression while a response parser is paused by downstream backpressure. It is not resolved into a universal documented guarantee in this packet; it becomes a required semantic-progress watchdog/adoption test for critical streams on the exact pinned version.

Undici RetryHandler documents default retries for selected idempotent methods, status codes, network codes, exponential backoff, and `Retry-After`. It refuses stateful request bodies that cannot be replayed.

Those defaults do not establish business safety. Every layer—SDK, Undici, application, queue, durable activity—must be inventoried so attempts do not multiply. Mutating calls require downstream idempotency keys or reconciliation.

A July 2026 Undici advisory found a response desynchronization risk in retry/resume behavior and listed patched releases. This confirms that retry middleware is security-sensitive and that embedded plus installed Undici versions must be tracked.

## Finding 8: libuv, worker threads, child processes, and cluster solve different problems

libuv's default global pool is four threads. More threads can improve measured blocking-work throughput but add memory and contention; they do not move JavaScript CPU off the main loop.

Worker threads:

- run JavaScript in parallel V8 isolates;
- are recommended for CPU, not ordinary asynchronous I/O;
- can transfer/share memory;
- share process RSS/fate and native-crash risk;
- require a long-lived bounded pool, queue, message schema, cancellation protocol, crash replacement, and poison-job policy;
- expose useful per-worker ELU, CPU, heap, snapshots, and on newer Node 24 patches CPU profiling.

Worker `resourceLimits` constrain selected V8 regions, not all native/external/process memory.

Child processes add separate address space, credentials, kill boundary, and native-crash containment, but require output, process-tree, signal, and cross-platform ownership. Cluster distributes server connections across Node processes; it is not a CPU job pool, durable queue, or security sandbox. Container orchestrators often make one-process replicas simpler than an in-container cluster primary.

## Finding 9: tool execution needs OS authority boundaries

In-process tools are trusted code. Model-controlled shell strings are prohibited. Use typed allowlisted operations mapped to `spawn`/`execFile` arguments with explicit executable, working directory, minimal environment, and resource limits.

Subprocess risks include:

- stdout/stderr buffer overflow or pipe deadlock;
- process descendants outliving direct-child termination;
- filesystem path/symlink escape;
- network SSRF/metadata access and redirect bypass;
- leaked environment credentials;
- binary/control-sequence/prompt-injection output;
- detached work outliving request/process ownership.

Node `vm`, workers, and permission flags are not malicious-code sandboxes. Generated/adversarial code needs a container/microVM/VM or specialized OS sandbox with default-deny egress, non-root identity, read-only mounts, quotas, PID/CPU/memory limits, per-run workspace, and destruction evidence.

## Finding 10: queues and durable runtimes turn event-loop health into correctness

BullMQ documents at-least-once behavior: a worker lock that is not renewed before expiry can cause the job to be marked stalled and processed again. CPU-intensive Node work can stall the event loop and renewal. Therefore loop delay, lease renewal, and effect idempotency are directly connected.

The durable design should persist run/attempt/effect/version state and use transactional outbox/inbox or engine-supported atomic boundaries. Queue acknowledgements occur only after the chosen durable state/effect transition.

Workflow choices are not interchangeable:

- Temporal TypeScript uses deterministic workflow replay and activities for I/O/effects; current support explicitly stops at Node 24. Its workflow sandbox is not hostile-code isolation.
- Restate offers journaled handlers, durable timers, promises, and external events with product-specific retention/cancellation semantics.
- DBOS TypeScript offers Postgres-centered durable workflows, steps, transactions, queues, messages, events, streams, recovery controls, and `maxRecoveryAttempts` for crash loops.
- BullMQ offers Redis-backed jobs/flows with lease/stalled-job semantics and requires idempotent jobs.

Use a simple queue for simple background work. Choose a durable engine when replay, long waits/signals, versioning, repair, or multi-step recovery is a real product requirement.

## Finding 11: memory admission must cover RSS, not only the V8 heap

`process.memoryUsage()` separates RSS, heap, external, and array-buffer memory. With workers, RSS is process-wide while most other fields are isolate-local. `v8.getHeapStatistics()` adds heap limit and context indicators. Node documents that stable V8 heap plus rising RSS on glibc can result from allocator fragmentation, but native/external memory, worker isolates, thread stacks, and mappings are alternative causes.

Admission must occur before large parsing/retrieval/prompt materialization where possible. Bounds are needed for:

- compressed and decompressed ingress;
- parsed objects and context;
- tool/retrieval artifacts;
- stream/event queues in bytes;
- live runs and queued closures;
- workers, subprocesses, connections, and telemetry;
- process RSS headroom below container limit.

`process.availableMemory()`/`constrainedMemory()` are telemetry inputs, not race-free guarantees. Use conservative limits, pressure-based shedding with hysteresis, and reserved control/repair capacity.

## Finding 12: async context is stable, but boundary propagation is explicit

`AsyncLocalStorage` is stable; `bind()` and `snapshot()` are stable, and Node 24 adds `name`/`defaultValue`. It is the correct basis for run/trace correlation through ordinary callbacks/promises. Low-level `async_hooks.createHook` is experimental and discouraged for general use.

Context does not automatically cross worker messages, processes, queues, HTTP service boundaries, or durable steps. Send a narrow versioned envelope and create a new destination context. Ambient context should not contain mutable transcripts or grant effect authority.

`diagnostics_channel` is stable and supports low-coupling library instrumentation. Subscribers remain in-process and need bounded non-disruptive export. Node 24.20 permission audit publishes channels for its scopes; Node 26 adds wider permission channels and test-runner events. Instrumentation must tolerate exact-version absence and shape changes.

OpenTelemetry JavaScript traces and metrics are stable; logs remain Development. AI semantic conventions/content capture can have separate maturity and sensitivity.

## Finding 13: diagnostics need preplanned safety limits

Stable Node CLI flags support CPU and heap allocation profiles. Diagnostic reports capture JS/native stacks, V8/libuv handles, OS/resource data, command line, and potentially environment/network information. Exclusion flags and secure storage are necessary.

V8 heap snapshot generation:

- blocks the event loop;
- can require roughly twice the current heap;
- captures only one isolate;
- uses a V8-version-specific undocumented schema;
- can expose sensitive in-memory data.

Near-limit snapshots can help but can also accelerate OOM/disk exhaustion. Inspector is powerful remote-code/debug authority and must never be publicly reachable. Capture should be privileged, one-at-a-time, capacity-checked, encrypted, and rehearsed.

## Finding 14: `node:test` is useful, but fake time is not a transport simulator

The built-in runner provides default process isolation, concurrency controls, timeouts, mocks, stable mock timers, stable snapshots, and reporters. Watch mode and coverage remain experimental; module mocking is early development. Tests need stability-aware use.

Mock timers control selected JS timers/Date, not DNS/TCP/TLS, worker scheduling, OS signals, queue servers, every dependency clock, or real event-loop starvation. Use them for deadline/backoff state machines; use real bounded integration tests for transport/lifecycle.

The runtime test matrix must inject:

- abort at every wait and around effect commit;
- slow/non-reading/disconnected stream clients;
- keep-alive close races and truncated bodies;
- main-loop CPU and libuv saturation separately;
- worker crash/hang/poison jobs;
- queue duplicate/lease loss and process kill at every durable boundary;
- memory/handle soak and return-to-baseline;
- rolling shutdown under peak long-lived streams.

## Finding 15: permission, packages, loaders, and npm form one supply-chain surface

The Node permission model is stable and broadening, but has documented constraints and does not replace OS isolation. High-authority grants—child, worker, addons, FFI/WASI, inspector, broad fs/net—should be exceptional and tested.

Packages should set explicit `type`. Dual conditional import/require exports can create duplicate stateful instances. Dynamic imports and loader hooks are privileged resolution/execution boundaries. Node 26 deprecates asynchronous `module.register()` and recommends synchronous `registerHooks()` in most cases, but the latter is still release-candidate.

For npm:

- `npm ci` is frozen and requires the same tree-shaping flags as lock creation;
- lifecycle/install/native build scripts remain code execution unless denied;
- current npm adds `allowScripts`/strict policy and git-dependency controls, but availability depends on the pinned npm version;
- provenance establishes origin/build linkage, not benign code;
- `npm audit signatures` verifies available registry signatures/attestations;
- trusted publishing removes long-lived publish tokens through OIDC and can generate provenance.

Retain lock, SBOM, signatures/provenance result, image/artifact digest, and native build identity.

## Finding 16: full Node, serverless Node, and edge runtimes are different products

Long-lived containers give the most control for native modules, worker threads, subprocesses, durable workers, long streams, and diagnostics, but the team owns capacity and shutdown.

Serverless Node may freeze/reuse an environment, limit duration/streaming/background work, and offer little shutdown time. AWS Lambda's Node 24 documentation is particularly explicit that unresolved promises are not waited for after handler return/stream end, and unfinished work can resume on a reused environment. Client disconnect does not necessarily terminate streamed execution.

Edge runtimes can provide a subset of Node APIs. Cloudflare documents supported, partial, and non-functional stub modules; compatibility date/flags change what imports and methods do. Vercel Edge exposes Web APIs plus platform-specific constraints and is not full Node. Import success is not an adoption test.

The artifact must run on the exact provider/runtime/compatibility date and exercise every required method, streaming/disconnect path, background-work rule, and observability surface.

## Finding 17: patch-level availability and API maturity are separate facts

Node 24.20 is an LTS patch, but it introduced both production-usable security operations and experimental APIs:

- `--permission-audit` and `process.permission.drop()` now exist on the production baseline, correcting the earlier assumption that audit was effectively Current-only;
- the documented Node 24 scope set remains narrower than Node 26, so flags/channels cannot be copied across majors;
- `node:stream/iter` arrived in 24.20 but is Stability 1 and flag-gated;
- `AsyncLocalStorage.withScope()`/`RunScope` arrived in 24.20 but are experimental, and the docs warn about caller-context visibility before the first `await`.

Therefore “available in LTS” is not a maturity grade. Every baseline table must record patch, stability, flag, scope, and fallback. Stable classic streams/Web Streams and `AsyncLocalStorage.run()` remain the default until experimental replacements pass adoption evidence and stabilize.

## Finding 18: context, compaction, and memory are Node resource boundaries

Semantic memory design is runtime-agnostic, but Node representation is not. Large strings, parsed object graphs, Buffer slices, structured-clone copies, `JSON.stringify()` output, tokenization, event queues, and `AsyncLocalStorage` reachability all affect RSS and event-loop fairness.

The production split is:

- active attempt context: admitted and byte/token-bounded;
- versioned run state: small JSON-safe domain data;
- continuity summary: persisted with source IDs, compaction version, and validation evidence;
- artifacts: large immutable data referenced by authorized handle;
- long-term memory: a separately authorized, provenance-bearing, correctable/expiring store.

Compaction is a bounded model/effect step with its own deadline, identity, cost, version fence, and failure state. Persist the replacement before switching the pointer, remove old live references, and test load/quiescence rather than expecting RSS to fall immediately. Never put transcripts or tool results in ambient async context.

## Finding 19: the application adapter—not the provider SDK—is the stable contract

Provider and telemetry SDK upgrades can preserve public TypeScript types while changing transport selection, retry defaults, abort propagation, stream event order, response-body ownership, content capture, package exports, or target-runtime support. Built-in `fetch` can also change when the Node patch changes because its Undici is embedded.

Each adapter upgrade requires black-box evidence for:

- wire attempt count and remaining-deadline behavior;
- abort before connect/headers/body and while downstream is backpressured;
- body cleanup on all rejected paths;
- known/unknown/terminal/usage/tool stream events;
- ESM/CJS initialization and OpenTelemetry preload order;
- embedded versus installed Undici combination;
- full Node/serverless/edge execution and security/content-capture defaults.

Recorded fixtures establish parser compatibility. A local fault server/proxy and exact deployment target establish transport and lifecycle semantics. Compile success or one happy-path canary establishes neither.

## Issue/advisory evidence retained as adoption-test leads

| Evidence | What it suggests | How the playbook uses it |
|---|---|---|
| Undici issue #5393: `bodyTimeout` under consumer backpressure | Parser pause/timeout interaction can leave critical streams without the assumed inactivity signal on affected versions | Require an application semantic-progress watchdog and exact-version fault test; do not state universal brokenness. |
| Undici GHSA-8xcm-r25x-g524 (2026-07-29) | Retry/resume could expose stale length and downstream desynchronization on affected package lines | Require patched Undici tracking and treat retry middleware as security-sensitive. |
| Node 24.17 security release | HTTP/TLS/permission fixes land inside LTS, including response queue poisoning and multiple TLS/permission issues | Require exact patch canaries, not “LTS means behavior never changes.” |
| BullMQ stalled-job documentation | Event-loop CPU can prevent lock renewal and cause double processing | Connect runtime loop health to queue idempotency and duplicate fault tests. |
| Cloudflare non-functional stub table | Module import/bundle success can hide runtime method failure | Require exact-runtime behavioral probes by compatibility date. |

## Source quality and synthesis rules

- Node and libuv official docs/source define core behavior.
- Undici repository docs define installed/latest dispatcher behavior; Node's embedded version may lag, so claims are versioned.
- WHATWG and RFC documents define SSE/HTTP semantics; Node/provider behavior still requires integration tests.
- npm official docs define current CLI policy; Node's bundled npm version must be checked separately.
- Temporal, Restate, DBOS, and BullMQ official docs define their own runtime claims. No cross-product “exactly once” equivalence is inferred.
- Cloudflare, Vercel, AWS, Kubernetes, and official Docker docs define platform-specific behavior, never generic Node guarantees.
- GitHub issues are used to design tests and identify uncertainty, not as settled specification.

## Required adoption experiments

### Event loop and CPU

- Maximum-size JSON/prompt/schema/regex work under concurrent streams.
- Main-loop stall while abort timer, provider socket, readiness, and queue lock renewal are pending.
- Actual libuv-heavy workload across default and proposed `UV_THREADPOOL_SIZE` under container quota.
- Worker pool startup/steady memory, transfer cost, cancellation, poison crash loop, and replacement rate.

### HTTP and streaming

- Exact Node embedded Undici versus installed Undici version inventory.
- Dispatcher origin/connection/pending limits and close/destroy behavior.
- Cold/reused keep-alive close race through the real proxy/provider.
- Connect/headers/body/semantic-idle/total deadlines under event-loop delay.
- Slow response consumer and the pinned version's body-timeout behavior.
- Node/Web Stream conversion error/cancel/backpressure semantics.
- SSE restart/reconnect/current/expired cursor and WebSocket backlog/heartbeat/auth expiry.
- If `node:stream/iter` is evaluated, test its experimental flag, strict/drop policy, byte accounting, adapters, abort, fallback, and patch-upgrade behavior separately from stable streams.

### Effects and durable work

- Abort immediately before/after external commit; reconcile by stable effect ID.
- Duplicate queue delivery and concurrent stale lease/attempt token.
- Kill before/after outbox, external effect, receipt, checkpoint, and ack.
- Old/new workflow behavior deployment, replay, rollback, and poison recovery.
- Confirm the durable SDK explicitly supports the Node patch/major and authentic deployment runtime.

### Memory and operations

- Peak representative context/tool/stream/retry workload below RSS/container envelope.
- Load/quiescence soak with heap/external/RSS/workers/handles/pool return-to-baseline.
- Heap snapshot and diagnostic report capacity, sensitivity, and storage procedure.
- SIGTERM with peak active SSE/WebSocket, queue work, workers, subprocesses, pools, and telemetry.
- Repeated signal and forced deadline behavior on Linux and every supported subprocess OS.
- Maximum-context compaction under concurrent runs: CPU/loop delay, clone/stringify peak, source preservation, stale-version conflict, cancellation, and old-reference release.
- Long-term-memory promotion, correction, expiry, tenant isolation, and deletion without retaining the originating transcript in ambient context.

### Security and deployment

- Permission deny matrix and, where supported, audit-mode inventory.
- Path/symlink, URL/redirect/private-address, output-size, and shell-injection tests.
- Frozen clean install with script/git/native policy and signature/provenance verification.
- Exact serverless/edge artifact execution, not import-only tests.
- Separate Node 24.20 and Node 26 permission-audit/enforcement matrices; prove unsupported scopes remain controlled by OS/container policy.
- Provider/tool/telemetry SDK adapter suite with wire-attempt, abort, stream, body-cleanup, content-capture, and ESM/CJS evidence.

## Primary source register

### Node releases and runtime

- [Node.js releases and production policy](https://nodejs.org/en/about/previous-releases)
- [Node.js Release Working Group schedule](https://github.com/nodejs/Release)
- [Node.js schedule JSON](https://github.com/nodejs/Release/blob/main/schedule.json)
- [Node.js 24.0.0 release](https://nodejs.org/en/blog/release/v24.0.0)
- [Node.js 24.20.0 LTS release](https://nodejs.org/en/blog/release/v24.20.0)
- [Node.js 26.0.0 Current release](https://nodejs.org/en/blog/release/v26.0.0)
- [Node.js 26.8.1 Current release](https://nodejs.org/en/blog/release/v26.8.1)
- [Node.js 22-to-24 migration](https://nodejs.org/en/blog/migrations/v22-to-v24)
- [Node.js annual schedule from v27](https://nodejs.org/en/blog/events/nodejs-interactive-2026)
- [Do not block the event loop or worker pool](https://nodejs.org/en/learn/asynchronous-work/dont-block-the-event-loop)
- [libuv design overview](https://docs.libuv.org/en/v1.x/design.html)
- [libuv thread pool](https://docs.libuv.org/en/latest/threadpool.html)

### Cancellation, streams, transport, and protocols

- [Node global `AbortController`/`AbortSignal`](https://nodejs.org/api/globals.html#class-abortcontroller)
- [Node events and `addAbortListener`](https://nodejs.org/api/events.html)
- [Node timers](https://nodejs.org/api/timers.html)
- [Node streams](https://nodejs.org/api/stream.html)
- [Node Web Streams](https://nodejs.org/api/webstreams.html)
- [Node 24.20 iterable streams (experimental)](https://nodejs.org/download/release/latest-v24.x/docs/api/stream_iter.html)
- [Node HTTP](https://nodejs.org/api/http.html)
- [Node HTTP/2](https://nodejs.org/api/http2.html)
- [WHATWG server-sent events](https://html.spec.whatwg.org/multipage/server-sent-events.html)
- [RFC 9110 HTTP semantics](https://www.rfc-editor.org/rfc/rfc9110.html)

### Undici

- [Undici repository and embedded/installed distinctions](https://github.com/nodejs/undici)
- [Undici Agent](https://github.com/nodejs/undici/blob/main/docs/docs/api/Agent.md)
- [Undici Pool](https://github.com/nodejs/undici/blob/main/docs/docs/api/Pool.md)
- [Undici Client](https://github.com/nodejs/undici/blob/main/docs/docs/api/Client.md)
- [Undici Dispatcher](https://github.com/nodejs/undici/blob/main/docs/docs/api/Dispatcher.md)
- [Undici RetryHandler](https://github.com/nodejs/undici/blob/main/docs/docs/api/RetryHandler.md)
- [Undici WebSocket](https://github.com/nodejs/undici/blob/main/docs/docs/api/WebSocket.md)
- [Undici issue #5393](https://github.com/nodejs/undici/issues/5393)
- [Undici retry advisory GHSA-8xcm-r25x-g524](https://github.com/nodejs/undici/security/advisories/GHSA-8xcm-r25x-g524)

### Workers, processes, memory, and diagnostics

- [Node worker threads](https://nodejs.org/api/worker_threads.html)
- [Node child processes](https://nodejs.org/api/child_process.html)
- [Node cluster](https://nodejs.org/api/cluster.html)
- [Node process](https://nodejs.org/api/process.html)
- [Node OS/available parallelism](https://nodejs.org/api/os.html)
- [Node Buffer](https://nodejs.org/api/buffer.html)
- [Node V8](https://nodejs.org/api/v8.html)
- [Node performance hooks](https://nodejs.org/api/perf_hooks.html)
- [Node diagnostics channel](https://nodejs.org/api/diagnostics_channel.html)
- [Node diagnostic reports](https://nodejs.org/api/report.html)
- [Node inspector](https://nodejs.org/api/inspector.html)
- [Node CLI profiling and runtime options](https://nodejs.org/api/cli.html)

### Async context, testing, permissions, packages, and npm

- [Node asynchronous context tracking](https://nodejs.org/api/async_context.html)
- [Node 24.20 asynchronous context tracking](https://nodejs.org/download/release/latest-v24.x/docs/api/async_context.html)
- [Node async-hooks warning](https://nodejs.org/api/async_hooks.html)
- [Node test runner](https://nodejs.org/api/test.html)
- [Node permission model](https://nodejs.org/api/permissions.html)
- [Node 24.20 permission model](https://nodejs.org/download/release/latest-v24.x/docs/api/permissions.html)
- [Node packages](https://nodejs.org/api/packages.html)
- [Node module customization hooks](https://nodejs.org/api/module.html#customization-hooks)
- [npm clean install](https://docs.npmjs.com/cli/commands/npm-ci/)
- [npm provenance](https://docs.npmjs.com/generating-provenance-statements/)
- [npm trusted publishing](https://docs.npmjs.com/trusted-publishers/)
- [npm signature/provenance verification](https://docs.npmjs.com/cli/audit/)
- [npm 2026 script and git dependency controls](https://github.blog/changelog/2026-02-18-npm-bulk-trusted-publishing-config-and-script-security-now-generally-available/)

### Durable work and queues

- [BullMQ at-least-once/stalled jobs](https://docs.bullmq.io/bull/important-notes)
- [BullMQ idempotent jobs](https://docs.bullmq.io/patterns/idempotent-jobs)
- [Temporal TypeScript SDK requirements](https://github.com/temporalio/sdk-typescript#requirements)
- [Temporal TypeScript API reference](https://typescript.temporal.io/)
- [Restate TypeScript external events](https://docs.restate.dev/develop/ts/external-events)
- [Restate durable timers](https://docs.restate.dev/develop/ts/durable-timers)
- [DBOS TypeScript workflows](https://docs.dbos.dev/typescript/tutorials/workflow-tutorial)
- [DBOS TypeScript queues](https://docs.dbos.dev/typescript/tutorials/queue-tutorial)
- [DBOS workflow recovery](https://docs.dbos.dev/production/workflow-recovery)

### Observability and deployment platforms

- [OpenTelemetry JavaScript status](https://opentelemetry.io/docs/languages/js/)
- [Official Node Docker best practices](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md)
- [Kubernetes container lifecycle hooks](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks)
- [AWS Lambda execution-environment lifecycle](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html)
- [AWS Lambda Node response streaming](https://docs.aws.amazon.com/lambda/latest/dg/config-rs-write-functions.html)
- [Cloudflare Workers Node compatibility](https://developers.cloudflare.com/workers/runtime-apis/nodejs/)
- [Cloudflare Workers compatibility flags](https://developers.cloudflare.com/workers/configuration/compatibility-flags/)
- [Vercel Edge runtime](https://vercel.com/docs/functions/runtimes/edge)
- [Vercel Node runtime](https://vercel.com/docs/functions/runtimes/node-js)

## Refresh triggers

Refresh this packet when any of the following occurs:

- Node 24 enters Maintenance LTS or reaches a security release affecting runtime guidance;
- Node 26 enters LTS, changes its date, or changes API stability before transition;
- v27 Alpha clarifies the new annual release lifecycle/compatibility expectations;
- Undici crosses a major, changes HTTP/2/retry/pool/body-timeout semantics, or publishes a transport advisory;
- Node permission audit, loader hooks, test coverage/module mocking, TypeScript support, or profiling APIs change stability;
- Temporal adds Node 26 support or changes worker architecture; Restate/DBOS/BullMQ changes relevant guarantees;
- OpenTelemetry JS logs or GenAI conventions stabilize;
- Cloudflare/Vercel/AWS changes compatibility, background work, streaming, duration, or shutdown semantics;
- official Node container guidance or supported base platforms change.

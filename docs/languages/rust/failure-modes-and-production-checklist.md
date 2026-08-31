# Rust Agent Runtime Failure Modes and Production Checklist

> **Last researched:** 2026-08-31
> **Purpose:** Pre-production review and incident triage for the Rust-specific failure surface

This guide is the consolidation gate. Passing unit tests and compiling without warnings are necessary, not sufficient. Run the failure injections against the production build, target OS, container policy, provider, database, MCP peer, and durable runtime.

## Failure matrix

| Symptom | Likely Rust/runtime cause | Evidence | Corrective direction |
|---|---|---|---|
| Shutdown hangs | Started `spawn_blocking` work never returns; child not reaped; pool handle held | Thread/task/process dump, blocking duration | Move unbounded work to supervised process/thread; bound drain |
| Memory rises with slow clients | Unbounded/byte-unbounded channel; large `Bytes` backing buffer retained | Queue elements+bytes, heap/RSS profile | Bound bytes, coalesce, artifact handles, disconnect policy |
| Tool continues after timeout | Dropped handle detached task; process `kill_on_drop` false | Task/process inventory after cancel | Track/join task; enable and test process-tree termination |
| Lost events in `select!` loop | Cancellation-unsafe `read_exact`/`write_all` or fairness queue reset | Reproduction at branch race | Own operation in child task or redesign framing |
| Deadline exceeded without timeout | Future/branch does CPU work without yielding | Tokio Console long poll, CPU profile | Chunk/yield/offload bounded compute |
| Trace parentage is nonsensical | `Span::enter` guard held across `.await` | Trace overlaps/tasks on same worker | Use `Instrument` or async `#[instrument]` |
| Duplicate external action | Retry got a lost response after commit | Effect ID, target receipt, retry logs | Stable idempotency key and reconciliation |
| Stale worker overwrites result | No database version/lease fencing | Two attempt IDs on transition | CAS/version/lease predicate |
| DB connections exhaust after tests/reloads | Pools repeatedly dropped without async close; acquisition cancellation drops connection | Server connections, pool metrics | One shared pool; `close().await`; test cancellation |
| Model JSON parsed but unsafe effect occurs | Serde shape accepted; no domain validation/authorization | Boundary audit | Raw → wire → validated → authorized types |
| Provider upgrade breaks tool schemas | Generated Schemars output changed/provider subset differs | Golden schema diff | Pin/normalize/test schema artifacts |
| Panic disappears | Detached task's `JoinError` ignored | Missing join results | Track and observe all task outcomes |
| Build compromise despite safe runtime code | Malicious build script/proc macro/native dependency | CI logs, lock/SBOM, build host | Isolated builds, graph audit, rotate credentials |
| Wasm memory grows indefinitely | Reusing one Store for unbounded instances | Store/instance count, RSS | Short-lived stores and resource limiter |
| MCP works in tests but violates tenant boundary | Conformance mistaken for authz/sandbox proof | Identity/policy trace | Application auth, tenant limits, capability tools |

## Required architecture gate

- [ ] A diagram identifies process supervisor, run owner, child tasks, persistence, and sandbox boundaries.
- [ ] Every task is joined, tracked, or owned by a documented supervisor.
- [ ] A detached run has a durable identity and no request-scoped borrowed authority.
- [ ] Run terminal state is a fenced state machine with one externally visible terminal transition.
- [ ] Raw, decoded, validated, authorized, and executed tool types are distinct.
- [ ] In-process tools are explicitly trusted; hostile tools cross an isolation boundary.

## Runtime and shutdown gate

- [ ] Admission stops before cancellation/drain begins.
- [ ] Cancellation tokens reach model, stream, tool, queue, and persistence operations.
- [ ] Every repeated `select!` operation has a cancellation-safety review.
- [ ] `JoinHandle::abort` is followed by join/observation where completion matters.
- [ ] `spawn_blocking` work is bounded, terminating, and capacity-limited.
- [ ] Process trees terminate and children are reaped on Linux, Windows, and other supported targets.
- [ ] Database pools and telemetry exporters close within a bounded phase.
- [ ] Forced shutdown records unfinished/ambiguous work for repair.

## Resource and network gate

- [ ] One reused reqwest client exists per security/transport policy.
- [ ] Connect, first-byte, idle-body, absolute, and admission time budgets are separate.
- [ ] Headers, compressed bytes, decompressed bytes, frames, JSON depth, tool output, and stream backlog are capped.
- [ ] All queues have element and aggregate-byte limits.
- [ ] Global, tenant, provider, tool, database, blocking, process, and Wasm capacity are independently controlled.
- [ ] Weighted fairness/head-of-line behavior is measured.
- [ ] Slow-client policy is explicit and tested.
- [ ] RSS, FDs/handles, threads, tasks, connections, child processes, and queue bytes recover after cancellation.

## Reliability and durability gate

- [ ] Error types preserve retry-relevant categories.
- [ ] Exactly one semantic owner controls each retry.
- [ ] Total attempts, time, cost, and tool calls are bounded.
- [ ] Mutating calls have stable effect IDs and receipts.
- [ ] Ambiguous effects enter reconciliation, not blind retry.
- [ ] Run transitions and outbox intents are atomic.
- [ ] Worker attempts are fenced by state version/lease.
- [ ] Durable code records nondeterminism in engine activities/steps.
- [ ] Old histories/checkpoints replay under deployment/version policy.
- [ ] Retention and history growth are bounded.

## Security and supply-chain gate

- [ ] No shell/batch interpreter receives model/user-controlled strings.
- [ ] Executables, workdirs, environment, files, network, secrets, CPU, memory, PIDs, and output are allowlisted/bounded.
- [ ] Wasmtime fuel/epoch/resource limiter and host-call limits are all configured.
- [ ] Provider/MCP identity metadata is not mistaken for authentication.
- [ ] Secrets never enter prompt, model-visible error, tool result, or ordinary trace.
- [ ] Toolchain and lockfile are pinned; CI uses `--locked`.
- [ ] Build scripts, proc macros, unsafe, FFI, and native libraries are inventoried.
- [ ] RustSec and dependency/license/source policy run in isolated CI.
- [ ] Release artifacts include SBOM, provenance, build ID, and retained symbols.

## Observability and verification gate

- [ ] Async spans do not hold enter guards across `.await`.
- [ ] Run, attempt, provider request, tool call, effect, and durable IDs correlate.
- [ ] Queue/permit wait and cancellation/cleanup time are measured.
- [ ] Model/tool content is redacted or stored only in protected sampled artifacts.
- [ ] Paused-time tests cover deadline, retry, cancellation, and shutdown.
- [ ] Loom/property tests cover critical concurrency/state invariants.
- [ ] Fuzzers cover provider/MCP frames, JSON/schema, validators, and redaction.
- [ ] Integration tests cover real process, OS, database, and network cleanup.
- [ ] Load, fault, and soak tests demonstrate bounded resources.
- [ ] Trajectory/evaluation tests remain separate from runtime correctness.

## Minimum failure-injection suite

1. Cancel at each `.await` around model and tool I/O.
2. Abort a task just before it completes and verify one terminal transition.
3. Fill every queue and semaphore independently.
4. Stream a never-ending partial frame and a slow trickle past idle timers.
5. Drop the client while mutating tools run.
6. Lose the effect response after the target commits.
7. Kill the worker after outbox commit and before dispatch/receipt.
8. Run two workers against one lease and prove fencing.
9. Panic inside model/tool/checkpoint tasks and observe `JoinError`.
10. Hang a blocking call and a subprocess during shutdown.
11. Spawn descendants and prove the sandbox terminates all of them.
12. Exhaust Wasm fuel/memory/host-call budget.
13. Feed deep, oversized, ambiguous, unknown-field, and schema-invalid JSON.
14. Rotate provider/MCP event fixtures to include unknown event types.
15. Rebuild from clean sources with `--locked`, SBOM, audit, and provenance.
16. Replay old durable histories through the candidate deployment.
17. Soak with long contexts and slow consumers; compare baseline tasks/RSS/FDs.

## Go-live decision

Go live only when:

- each failed test has an explicit product outcome rather than “the task stops”;
- the on-call team can retrieve task, trace, provider, effect, durable, and process evidence;
- repair procedures exist for stuck, ambiguous, incompatible, and partially completed runs;
- preview/community dependencies are accepted as owned risk with pinned versions and escape hatches;
- resource envelopes hold at peak context and downstream failure, not just average load.

Return to the [Rust index](README.md) for focused remediation guides.

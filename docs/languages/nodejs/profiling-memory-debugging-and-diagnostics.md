# Profiling, Memory Debugging, and Diagnostics

> **Last researched:** 2026-08-31  
> **Baseline:** Stable CPU/heap profile CLI flags and diagnostic reports on Node 24/26  
> **Related:** [Memory, resource budgets, and admission](memory-resource-budgets-and-admission.md)

Production diagnostics are invasive operations with disk, CPU, memory, pause, and data-exposure costs. Prepare safe capture paths before an incident; do not discover during an outage that the container is read-only, the inspector is public, or a heap snapshot needs twice the available heap.

## Start from the symptom, not the favorite profiler

| Symptom | First evidence | Deeper capture |
|---|---|---|
| High latency, high event-loop delay/ELU | ELU/delay, process CPU, queue/pool waits | CPU profile, flame graph, trace events |
| High latency, low main-loop utilization | Downstream timings, pool/queue wait, worker metrics | Network/SDK diagnostics, worker profiles |
| Rising heap | Heap/RSS/external series, GC, contexts | Allocation profile, heap snapshots/diff |
| Stable heap, rising RSS | External/array buffers, workers, native modules, threads | Diagnostic report, native/allocator analysis |
| Process will not exit | Active-resource types, owned registry | Diagnostic report/libuv handles, inspector handles |
| OOM/fatal crash | Supervisor/container evidence | `--report-on-fatalerror`, near-limit snapshots with caution |
| Worker hot/leaking | Per-worker ELU/CPU/heap | Worker CPU profile or heap snapshot |

Take the least disruptive capture that can answer the question.

## CPU profiles

`--cpu-prof` starts V8 sampling at process startup and writes a `.cpuprofile` before exit; the flags are stable. Control directory, name, sampling interval, and disk retention. For targeted runtime capture, use the inspector protocol or current worker profiling APIs.

Interpret CPU profiles alongside:

- event-loop delay/ELU;
- container throttling and CPU quota;
- GC samples;
- source maps and deployed artifact identity;
- worker versus main isolate;
- native frames and symbol availability.

A profile can perturb timing and may miss off-CPU waits. High function self-time is not proof it caused user latency unless it overlaps the affected interval and isolate.

Node 24.8 added `worker.startCpuProfile()`, useful for profiling a worker from the parent. Version-gate it and bound buffer/sample settings. A hot worker and a hot main loop have different remediation.

## Heap profiles and snapshots are different

`--heap-prof` records sampled allocation information with lower overhead than a full snapshot and writes a heap profile on exit. Heap snapshots describe reachable objects and retainers for one V8 isolate.

Node warns that a heap snapshot can require about twice the current heap and blocks the event loop while generated. In a near-OOM process, it can trigger the final failure or container kill. A main-thread snapshot does not include worker heaps; each isolate needs separate capture.

Safe snapshot policy:

- privileged, authenticated trigger unavailable to ordinary tenants;
- one capture at a time with minimum free-memory and disk checks;
- drain/shift traffic or capture a canary/replica when possible;
- encrypted restricted storage and short retention;
- no automatic upload to a broadly accessible log bucket;
- record Node/V8 version because snapshot schema is V8-specific;
- delete according to incident and privacy policy.

`--heapsnapshot-near-heap-limit` is stable on current 24/26 patches and can capture before failure, but repeated snapshots add disruption and disk usage. Test under the actual memory limit before enabling fleet-wide.

## Diagnostic reports

Node diagnostic reports include JavaScript/native stacks, V8 heap statistics, libuv handles, OS/resource data, command line, and other process details. They can be triggered on fatal errors, uncaught exceptions, a signal, or programmatically.

Configure a writable bounded diagnostic directory. Use `--report-exclude-env` and `--report-exclude-network` where exposure or slow reverse-DNS work outweighs diagnostic value. Even with exclusions, treat reports as sensitive.

Recommended baseline for crash-focused services:

- report on fatal error;
- optionally report on uncaught exception while still allowing process exit;
- a documented privileged on-signal capture for Linux operations;
- unique filenames and collector/rotation policy;
- container volume/storage sizing and upload-after-restart procedure.

Do not generate heavy diagnostics synchronously from every error path.

## Inspector safety

The inspector exposes powerful debugging and evaluation capabilities. Never bind it publicly or enable it for an untrusted network. Use local/loopback access through an authenticated tunnel under incident procedure. The Node permission model restricts inspector activation unless allowed, but network isolation remains essential.

Inspector sessions and heap captures can block or alter timing. Record activation and operator identity. Close sessions after capture.

## Diagnose memory systematically

1. Compare RSS, heap used/total, external, array buffers, workers, and container metrics.
2. Force no production GC as a “fix”; observe natural cycles and allocation rate.
3. Reproduce a stable workload, then compare retained state after quiescence/cancellation.
4. Inspect likely owners: run maps, listeners, timers, task registries, stream buffers, caches, AsyncLocalStorage stores, sockets, telemetry batches, workers.
5. Use allocation profiling to find hot allocation paths; snapshots to find retainers.
6. Repeat with source maps and exact artifact.
7. Verify cleanup returns toward baseline after shutdown/drain.

On glibc systems, Node documents sustained RSS growth with stable V8 heap due to allocator fragmentation. Do not declare a JS leak from RSS alone, but also do not assume fragmentation without evidence. Native/external allocations and worker isolates are common alternatives.

## Active handles and shutdown leaks

`process.getActiveResourcesInfo()` provides resource type names keeping the loop alive. Diagnostic reports add libuv handle details. Compare against an application-owned registry of servers, sockets, timers, workers, ports, child processes, and exporters.

Avoid undocumented internal handle APIs as production contracts. A test can snapshot active resource types before/after a run, but assertions should tolerate expected runtime/test-runner resources and focus on deltas owned by the component.

## Diagnostic runbook checklist

- [ ] CPU, allocation, heap snapshot, report, and inspector procedures are documented and rehearsed.
- [ ] Capture destinations are writable, capacity-limited, encrypted, and access-controlled.
- [ ] Reports/snapshots are classified as sensitive and have retention/deletion rules.
- [ ] Heap snapshot memory/pause risk is tested under the container envelope.
- [ ] Main and worker isolates can be diagnosed separately.
- [ ] Profiles include exact artifact, Node/V8 version, source maps, and time window.
- [ ] Fatal/uncaught capture does not suppress the required process crash/restart.
- [ ] Inspector is loopback/private and temporary.
- [ ] Active resource ownership explains shutdown hangs without relying only on internals.

## Selected primary sources

- [Node.js CLI CPU/heap profiling and reports](https://nodejs.org/api/cli.html)
- [Node.js V8 heap snapshot warnings](https://nodejs.org/api/v8.html#v8getheapsnapshotoptions)
- [Node.js diagnostic report](https://nodejs.org/api/report.html)
- [Node.js inspector](https://nodejs.org/api/inspector.html)
- [Node.js worker diagnostics](https://nodejs.org/api/worker_threads.html)
- [Node.js process memory note](https://nodejs.org/api/process.html#a-note-on-process-memoryusage)


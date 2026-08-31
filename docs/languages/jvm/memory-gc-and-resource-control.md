# Memory, GC, and Resource Control

## Agent capacity is retained state

Concurrency cost is not just thread stacks. Waiting runs retain prompts, provider buffers, tool results, JSON trees, telemetry attributes, SDK objects, and queued events. Virtual threads and coroutines make waiting cheaper but can make it easier to retain too many runs.

Estimate:

<code>resident run memory × admitted runs + heap baseline + caches + native memory + safety margin</code>

Measure the 95th/99th percentile retained bytes per active run under representative tools and streaming, not only empty tasks.

Turn the measurement into admission. If a load test shows a 99th-percentile active run retains 6 MiB, and the pod has 600 MiB of *measured* safe run-state headroom after heap baseline, native/off-heap allowance, caches, sidecars, and safety margin, the memory ceiling is at most 100 active runs. Apply a lower operational limit until a soak test proves GC pause, RSS, and tail latency remain healthy. Recalculate when prompt limits, SDK buffering, telemetry, or tool payloads change.

## Separate budgets

| Budget | Examples | Control |
|---|---|---|
| Java heap | strings, JSON trees, run state | Xmx, admission, byte/object limits |
| native JVM | metaspace, code cache, GC, thread stacks | NMT, class/thread control |
| off-heap | direct buffers, mmap, native SDKs | library limits and process RSS |
| external | sockets, DB connections, provider quota | pools/semaphores |
| child processes | tool heap/RSS/files | cgroup/container/worker limits |

Container OOM decisions use process/cgroup memory, not heap occupancy. Leave headroom above Xmx for native components. The JDK is container-aware; <code>MaxRAMPercentage</code> defaults to 25%, which is not an application-specific sizing decision. Pin and measure memory settings rather than inheriting accidental ergonomics.

## Collector choice

Start with G1, the default collector, and evidence-based heap sizing. Tune only when GC logs/JFR show a problem. Copying old generation sizing flags can disable useful ergonomics.

Consider ZGC when low pause time is a hard requirement and CPU/memory headroom supports concurrent collection. In JDK 25 ZGC is generational; its main tuning input is maximum heap. A low-pause collector cannot fix unbounded retention, oversized prompts, or leaks.

## High-risk allocations

- full-body <code>String</code>/<code>byte[]</code> HTTP handlers;
- parsing the same JSON into tree, DTO, and log payload;
- retaining every token delta;
- unbounded chat memory and tool artifacts;
- high-cardinality telemetry attributes;
- per-virtual-thread ThreadLocal caches;
- hot-flow replay buffers;
- classloader/plugin churn and generated proxy classes.

Stream where possible, cap before allocation, summarize for prompts, and reference large artifacts by digest.

## Diagnostics

Enable GC logging suitable for the environment and retain it through failures. Use JFR for allocation pressure, object statistics, socket/file activity, locks, and virtual-thread events. Heap dumps contain secrets and can be very large; secure their storage and verify enough disk exists.

Native Memory Tracking:

1. start with <code>-XX:NativeMemoryTracking=summary</code> or <code>detail</code>;
2. establish a <code>jcmd VM.native_memory baseline</code>;
3. compare with <code>summary.diff</code>;
4. remember NMT has roughly 5–10% overhead and does not track all external native allocations.

Track classloader count, direct-buffer pools, live threads, open descriptors, active runs, queued bytes, and per-tool child RSS.

Use evidence to separate failure classes:

| Symptom | First evidence | Likely control |
|---|---|---|
| heap climbs with active runs, then falls | heap/JFR allocation by run path | admission and payload caps |
| old-gen/live set grows after traffic drains | class histogram/heap dump, retained paths | ownership leak or unbounded cache |
| RSS grows while heap is flat | NMT diff, direct-buffer/JNI/child RSS | native/off-heap/process limit |
| long pauses with stable live set | GC log/JFR, allocation rate | allocation reduction, heap/collector evidence |
| many waiting runs retain large prompts | run bytes, semaphore wait, heap paths | reject earlier; acquire permits before materialization |

An `OutOfMemoryError` can leave the process unable to allocate for recovery or logging. Do not promise in-process continuation after arbitrary OOM. Persist state before effects, make restart recovery reliable, and configure orchestrator/crash behavior deliberately. Heap dumps are postmortem evidence, not a recovery mechanism.

## Overload policy

Reject or queue before materializing large request bodies and prompts. Bound per-tenant and global active runs. Prefer an explicit 429/503 with retry guidance over letting GC or the kernel choose victims. Readiness may go false during controlled overload, but liveness should not restart a healthy overloaded process.

## Checklist

- [ ] Xmx leaves measured native headroom inside container limit.
- [ ] Active runs and retained bytes are bounded.
- [ ] HTTP, JSON, Flow, replay, and telemetry buffers have caps.
- [ ] Large artifacts are externalized.
- [ ] Collector choice follows pause/throughput evidence.
- [ ] GC logs, JFR, NMT, heap dump, and thread dump procedures exist.
- [ ] Heap-dump sensitivity and disk impact are controlled.
- [ ] Load tests include provider latency and large tool outputs.

## Sources

- [G1 collector, Java 25](https://docs.oracle.com/en/java/javase/25/gctuning/garbage-first-g1-garbage-collector1.html)
- [G1 tuning guidance](https://docs.oracle.com/en/java/javase/25/gctuning/garbage-first-garbage-collector-tuning.html)
- [Java 25 GC tuning guide](https://docs.oracle.com/en/java/javase/25/gctuning/hotspot-virtual-machine-garbage-collection-tuning-guide.pdf)
- [Java command container and memory options](https://docs.oracle.com/en/java/javase/25/docs/specs/man/java.html)
- [Oracle memory-leak troubleshooting and NMT](https://docs.oracle.com/en/java/javase/25/troubleshoot/troubleshooting-memory-leaks.html)

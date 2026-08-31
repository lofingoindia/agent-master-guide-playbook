# Python Agent Engineering: Deep-Dive Research Packet

> **Research date:** 2026-08-31  
> **Purpose:** Evidence base for [`docs/languages/python/`](../../languages/python/README.md)  
> **Baseline:** CPython 3.14.7 stable; CPython 3.15.0rc1 prerelease  
> **Method:** Primary documentation and source first; project-maintainer documentation for ecosystem behavior; production/security guidance for OS and deployment boundaries

This packet records the evidence and synthesis decisions behind the Python production agent-engineering area. It complements the broader [Python and TypeScript runtime packet](python-and-typescript-agent-runtimes.md) by going deeper on Python concurrency, process/runtime boundaries, serving, validation, durability, profiling, testing, supply chain, and deployment.

## Research questions

1. What does CPython 3.14.7 actually guarantee for task groups, cancellation, timeouts, queues, streams, subprocesses, executors, interpreters, multiprocessing, GC, and shutdown?
2. Which 3.15 features are credible trajectory versus production-ready baseline?
3. Where do threads, processes, subinterpreters, free-threaded builds, and OS sandboxes differ in kill, crash, memory, serialization, and trust boundaries?
4. How do ASGI, Uvicorn, HTTPX, and asyncio flow control compose under slow streams and disconnects?
5. Which Pydantic defaults are unsafe to treat as a complete model/tool contract?
6. How should Python services preserve retry, idempotency, state, and recovery semantics across broker and durable runtimes?
7. Which runtime signals and tools distinguish Python heap, native memory, event-loop stalls, and process/resource leaks?
8. What test, packaging, provenance, worker, and shutdown practices are required for a reproducible production service?
9. How should Python agent/framework choices be composed without confusing orchestration, durability, and sandboxing?
10. Which Python mechanics matter when short-term context is compacted into durable state or long-term memory?
11. How should third-party SDK and telemetry convention churn be contained and tested?

## Version snapshot and time-sensitive findings

| Surface | Evidence observed on 2026-08-31 | Documentation implication |
|---|---|---|
| CPython stable | Python.org lists 3.14.7, released 2026-08-05, as the seventh 3.14 maintenance release | Write to 3.14.7 patch behavior, not initial 3.14 summaries |
| CPython prerelease | 3.15 docs identify 3.15.0rc1; PEP 790 schedules final for 2026-10-01 | Label all 3.15 behavior trajectory/prerelease |
| `asyncio` | 3.14.7 task/queue/runner/stream/subprocess docs and CPython 3.14 source reviewed | Task ownership and cancellation guidance can be precise |
| Multiprocessing | 3.14 docs say `fork` is no longer default anywhere; POSIX defaults to `forkserver` where available | Do not recommend pre-fork assumptions without explicit configuration/tests |
| Subinterpreters | 3.14 exposes `concurrent.interpreters` and `InterpreterPoolExecutor` | Present as controlled CPU isolation within a process, not sandbox/crash boundary |
| Free threading | PEP 779 accepted phase II: officially supported but optional in 3.14 | Treat as separate runtime artifact requiring native/concurrency/RSS tests |
| Free-threaded stable ABI | PEP 803 (`abi3t`) is final for 3.15 | Useful packaging trajectory, not proof of thread-safe extension behavior |
| GC | 3.14.7 docs note generation 1 and `threshold2` were restored in 3.14.5 | Avoid repeating early-3.14 “removed/ignored” guidance |
| Pydantic | Latest docs describe strict/lax differences, JSON-versus-Python strict nuance, Draft 2020-12 schema, discriminated unions | Separate runtime/wire/provider/durable schema contracts |
| Celery | Official docs render as 5.6.2 | Record exact ack/worker-loss/shutdown semantics, not generic “at least once” alone |
| AnyIO | Official docs render as 4.14.2 | Acknowledge AnyIO/asyncio task-group semantic differences |
| pytest-asyncio | Stable docs render as 1.4.0 | Make discovery mode and loop scopes explicit |
| pip | Stable docs render as 26.2.1; `pip lock` remains experimental | Standard lock format does not make one lock universally portable |
| OpenTelemetry Python | Traces and metrics are stable; logs remain development | Pin signal/package posture rather than calling “OpenTelemetry” uniformly stable |
| OpenTelemetry GenAI conventions | Moved to a dedicated repository; agent/framework spans remain development | Keep a stable internal event model and versioned projection into telemetry |

Version labels from “latest” project documentation are snapshots, not repository-wide dependency recommendations. Every deployment must pin and retest its own exact versions.

## Source-reading method

The research used these rules:

- CPython behavior came from 3.14.7 library documentation plus the `3.14` source branch for `asyncio.taskgroups`, `timeouts`, `queues`, `streams`, and `runners` where lifecycle details mattered.
- Future behavior came from 3.15.0rc1 “What's New,” 3.15 library docs, and accepted PEPs, always labeled prerelease.
- ASGI behavior came from the ASGI specifications; client/server details came from HTTPX and Uvicorn maintainer documentation.
- Validation behavior came from current Pydantic documentation and provider-specific official structured-output documentation.
- Durable/worker behavior came from the selected runtime's own documentation rather than a generic agent abstraction.
- Sandbox recommendations distinguish Python/process features from OS/container/VM controls using CPython's explicit non-sandbox warnings and container/sandbox security documentation.
- No cross-runtime throughput or memory number was adopted without a reproducible benchmark. Capacity claims in the guides are measurement requirements, not vendor benchmarks.
- Open issues and fast-moving docs were treated as regression-test inputs, not permanent guarantees.

## Evidence synthesis

### Finding 1: `TaskGroup` gives failure containment, not full run semantics

CPython documents that non-cancellation child failure cancels the remaining tasks, waits for them, and raises grouped errors. The 3.14 source tracks children, abort state, parent cancellation, errors, and completion. `TaskGroup` is therefore a good run-local ownership primitive.

It does not supply admission, byte budgets, external-effect rollback, durable state, thread/process termination, tenant fairness, or terminal-state fencing. Those remain application contracts.

Documentation outcome:

- prefer a task tree over detached `create_task()` calls;
- use typed results for expected partial failure;
- let unexpected failure preserve sibling cancellation;
- retain a separate service supervisor for work that intentionally outlives a run.

### Finding 2: cancellation is cooperative and internally reused by structured APIs

`CancelledError` is injected at an await point and derives from `BaseException`. Python explicitly recommends `try/finally` cleanup and re-propagation if caught. Task groups and timeouts use cancellation internally; the docs warn against swallowing it and routine `uncancel()` use.

The timeout implementation records the task's cancellation count, cancels the task at expiry, then translates its own cancellation into `TimeoutError` only when no newer cancellation supersedes it. `wait_for()` waits for child cancellation to settle and can therefore exceed its numeric timeout.

Documentation outcome:

- derive nested timeouts from one absolute monotonic deadline;
- reserve a separate cleanup budget;
- distinguish “caller stopped waiting” from “underlying thread/process/effect stopped”;
- record requested, observed, cleanup, and terminal-fenced cancellation stages.

### Finding 3: queue shutdown semantics require an explicit discard decision

`asyncio.Queue` is process-local, not thread-safe, and unbounded when `maxsize=0`. Operations have no timeout parameter. Python 3.13 added shutdown: normal shutdown rejects new producers and allows drain; immediate shutdown drains and unblocks waiters, and the docs warn that it can violate `join()`'s usual completed-work invariant.

Documentation outcome:

- every queue needs a positive item limit plus a separate byte budget;
- `task_done()` belongs in consumer `finally`;
- normal drain and immediate discard are separate policies;
- never present an asyncio queue as durable work storage.

### Finding 4: thread cancellation stops the waiter, not the blocking function

`asyncio.to_thread()` propagates context and is intended primarily for blocking I/O on standard CPython. Cancellation of the awaiting coroutine does not stop the worker thread. The executor's internal submission queue is not an application admission policy.

Documentation outcome:

- require library-level deadlines/cooperation;
- bound submission before the executor and isolate slow work classes;
- fence/reconcile late writes;
- move operations needing hard stop or crash containment to a process/worker boundary.

### Finding 5: Python 3.14 adds parallelism options with intentionally different isolation

`ProcessPoolExecutor` provides process isolation and 3.14 terminate/kill methods, with picklability/importability restrictions. Multiprocessing warns that forced termination can corrupt queues/pipes and deadlock peers holding synchronization primitives.

`InterpreterPoolExecutor` and `concurrent.interpreters` provide separate interpreter/import state and per-interpreter GILs inside one process. Most values are copied/serialized; extension compatibility is incomplete. A native crash still affects the process.

Free-threaded Python enables parallel threads but remains an optional build and can re-enable the GIL for incompatible extensions. Its memory and contention profile differs.

Documentation outcome: use a comparison matrix across CPU parallelism, hard kill, crash/RSS isolation, shared state, serialization, and trust; never call subinterpreters/free threading a sandbox.

### Finding 6: multiprocessing defaults changed enough to invalidate common service recipes

Python 3.14 states that `fork` is no longer the default on any platform. POSIX uses `forkserver` where supported; Windows/macOS use `spawn`. The `__main__` import and pickling contract matters for pools. Forking multithreaded processes is explicitly problematic.

Documentation outcome:

- keep child entrypoints importable and payloads explicit;
- test Windows `spawn` and POSIX `forkserver`;
- do not create event-loop-bound clients before worker start/fork;
- treat `gc.freeze()`/preload copy-on-write as a specialized measured optimization.

### Finding 7: ASGI/HTTP flow control does not bound application objects

ASGI is an async message interface with per-connection scope and lifespan per event loop. The HTTP spec documents disconnect messages and an `OSError`-style send failure for supported spec versions, with race/compatibility caveats.

HTTPX distinguishes connect/read/write/pool timeouts, exposes explicit connection limits, warns against client creation in a hot loop, and requires explicit close in manual streaming mode. Uvicorn documents transport read/write flow control, 503-style concurrency refusal, TCP backlog as a separate layer, and worker resource controls.

Documentation outcome:

- lifespan-scope clients/pools per worker;
- separate transport-phase and end-to-end deadlines;
- bound normalized event bytes and count;
- explicitly choose disconnect = cancel, continue, pause, or durable handoff;
- close provider responses on early exit/cancellation.

### Finding 8: shape validation is only the first executable policy stage

Pydantic defaults are frequently coercing. Strict behavior can differ between JSON and Python-object validation. Discriminated unions are more predictable than untagged unions. Generated schema targets Draft 2020-12/OpenAPI 3.1, while model providers commonly support subsets.

Python's pickle documentation explicitly warns that untrusted pickle can execute arbitrary code. Process/interpreter transports that use pickle inherit that trust constraint.

Documentation outcome:

- bound raw bytes, then parse, schema-validate, domain-check, authorize, and apply effect policy;
- use strict mode for security/effect identifiers and controls;
- version variants and provider-schema artifacts;
- locally validate provider output again;
- use language-neutral durable/wire representations.

### Finding 9: retry safety depends on phase and effect evidence

HTTPX exposes distinct timeout/network/protocol/status classes. Tenacity supports explicit stop/wait/retry predicates but its bare documented default can retry forever with no wait. Task groups can aggregate multiple failures via `ExceptionGroup`; cancellation is not caught by ordinary `Exception` handlers.

Documentation outcome:

- select one semantic retry owner and inventory lower-layer retries;
- require a remaining-deadline/cleanup reserve before another attempt;
- use jitter and fleet/tenant retry budgets;
- give effects stable identity across attempts;
- reconcile ambiguous outcomes rather than treating timeout/cancel as “not committed.”

### Finding 10: broker delivery and durable replay are different layers

Celery documents early acknowledgement by default, late acknowledgement for idempotent tasks, worker-loss exceptions/settings, prefetch behavior, and warm/soft/cold shutdown differences. These concrete semantics are more nuanced than “Celery is at least once.”

Temporal Python sandboxes workflow code to help detect non-determinism, but explicitly says this is not complete isolation. Restate journals `ctx.run` results and retries according to policy. DBOS exposes durable workflows/steps. Pydantic AI documents integrations with Temporal, DBOS, Prefect, and Restate. LangGraph differentiates in-memory from persistent checkpointers.

Documentation outcome:

- use small versioned broker references and lease/version fencing;
- couple local state/publish through outbox and deduplicate consumers;
- keep external effects in the durable engine's prescribed activity/step boundary;
- distinguish cancellation, abandonment, termination, and compensation.

### Finding 11: RSS cannot be inferred from the Python object graph

`tracemalloc` traces Python allocations after it starts. `sys.getsizeof()` is shallow. Native extensions, allocators, memory maps, child processes, and fragmentation contribute to RSS. Memray can trace Python/native allocations on supported platforms.

The 3.14.7 GC docs contain a patch-level correction: generation-1/`threshold2` behavior was restored in 3.14.5. Free-threaded builds have different reference-counting/allocator/GC memory behavior.

Documentation outcome:

- budget RSS, heap, native, child, pool, descriptor, and buffer views separately;
- bound bytes, not only objects/tasks;
- tune GC only from a reproduced profile on the exact build;
- capacity-test free-threaded artifacts independently.

### Finding 12: Python observability must include the scheduler and process topology

OpenTelemetry supplies Python APIs/SDK/exporters, but semantic conventions and signal maturity/versioning require pinning. Context variables support async-local metadata but do not cross serialized boundaries automatically. Prometheus Python multiprocess mode has documented limitations and directory-lifecycle requirements.

CPython offers asyncio debug mode, task/thread introspection, deterministic profilers, `tracemalloc`, and `faulthandler`; 3.15 adds promising async-aware profiling/dump trajectory.

Documentation outcome:

- correlate run/turn/model/tool/effect/attempt IDs;
- observe event-loop lag, task age, executor and HTTP pool wait, buffered bytes, cancellation cleanup, RSS/fds/children, and telemetry drops;
- select CPU, wall, heap, native, hang, and fleet tools by question;
- redact prompts/tool data before telemetry queues.

### Finding 13: deterministic runtime tests need explicit scheduling and clocks

`IsolatedAsyncioTestCase`, `AsyncMock`, pytest-asyncio, AnyIO testing, and Hypothesis state machines provide complementary mechanisms. Pytest-asyncio docs say tests run sequentially by default and expose explicit discovery/loop scopes. HTTPX's in-process ASGI transport does not drive lifespan.

Asyncio timers use monotonic `loop.time()`, so freezing wall clock is not sufficient and patching the loop clock carelessly can break scheduling.

Documentation outcome:

- inject monotonic/wall/sleep policy through a small clock port;
- use events/barriers, not sleep guesses, to reproduce races;
- fail teardown on leaked tasks/resources;
- test ASGI lifespan and real sockets/process signals separately;
- combine deterministic runtime tests, provider contracts, model evals, and load/soak.

### Finding 14: Python lock/provenance capabilities still need environment-specific policy

`pylock.toml` is a standard format, but pip's current lock command is experimental and only guarantees the generated result for the current Python/platform. Pip secure installs describe all-or-nothing hashes and wheel-only installation. Package builds run backend/build-dependency code even when isolated.

PyPI Trusted Publishing replaces long-lived tokens with short-lived OIDC; digital attestations bind distributions to identities/workflows. `pip-audit` finds known vulnerable packages but does not prove non-malicious code or safe configuration.

Documentation outcome:

- lock/build/test per target environment;
- deploy verified wheels without production resolution/source builds;
- control index authority to prevent dependency confusion;
- verify attestations while retaining review, SBOM, scan, and staged rollout;
- track standard ABI, free-threaded ABI/behavior, and subinterpreter compatibility separately.

### Finding 15: graceful shutdown is a bounded recovery opportunity, not correctness

ASGI lifespan is per worker loop. Python signal handlers run on the main thread, and `atexit` does not run for several fatal/forced exits. Kubernetes normally sends TERM then KILL after the grace period, with lifecycle hooks consuming the same budget.

Documentation outcome:

- mark unready and stop admission/leasing before drain;
- cancel/handoff/checkpoint/reconcile, then close pools/streams/exporters;
- keep server grace below platform grace with reserve;
- assume hard kill prevented cleanup and recover from durable intent/receipt/version evidence;
- treat worker recycling as leak containment, not repair.

### Finding 16: context compaction is a versioned state transition, not string replacement

Python framework message lists and dictionaries are convenient runtime views, not durable memory contracts. A summarizer can race a new event, be retried after an ambiguous provider response, or emit a semantically incomplete result. Long-term retrieval adds separate provenance, tenant, retention, deletion, and authorization requirements.

Documentation outcome:

- separate run/event state, short-term working context, compaction artifacts, long-term memory, and large artifacts;
- identify compaction inputs by immutable source range/digest and policy/model/schema version;
- compute outside a database transaction, then commit with compare-and-set against the source state version;
- preserve source evidence according to retention policy and treat retrieved memory as untrusted context;
- bound tokenization, serialization, embedding, summarization, and retrieval by bytes, CPU/provider concurrency, deadline, and telemetry budgets.

### Finding 17: SDK and telemetry churn need executable application contracts

Provider/framework versions can change stream event shapes, settlement and usage, exception types, retry behavior, generated schemas, serialized state, or lifecycle expectations without changing the application's desired contract. OpenTelemetry Python currently has mixed signal maturity, and the GenAI conventions moved repositories while agent spans remain development.

Documentation outcome:

- normalize only required behavior behind thin adapters;
- keep exact-version fixtures for requests, streams, tools/schemas, state, effects, telemetry, and client lifecycle;
- separate durable application events from lossy telemetry projections;
- stage runtime, SDK/framework, model, and state/schema upgrades independently where practical;
- canary and retain an immutable rollback artifact instead of trusting import success.

## Claims deliberately excluded or qualified

- **“Async Python scales automatically.”** Only cooperative I/O and bounded resources support high concurrency; blocking code and buffers remain decisive.
- **“The GIL makes shared state safe.”** It does not make multi-step invariants atomic, and free-threaded builds remove accidental serialization.
- **“Free-threaded Python is now the default.”** It is officially supported but optional in 3.14.
- **“Subinterpreters are lightweight secure processes.”** They share a process failure/resource boundary and are not a hostile-code sandbox.
- **“A timeout cancels the operation.”** It cancels a task/wait; threads, children, and effects need separate mechanisms.
- **“Provider structured output removes validation.”** It addresses a schema surface, not domain, authz, refusal, truncation, or effect safety.
- **“A broker or session store makes an agent durable.”** Recovery requires defined acknowledgment/replay/effect/version semantics.
- **“A lock file guarantees universal reproducibility.”** Environment markers, native wheels, Python/platform, build inputs, and installer support matter.
- **“Container means sandbox.”** Security strength depends on identity, syscalls, mounts, network, resources, runtime/kernel, and threat model.
- **“Worker recycling fixes leaks.”** It limits impact and can hide the cause.
- **“3.15 features are current production defaults.”** The observed release is RC1; final behavior must be rechecked.
- **“Context compaction is harmless summarization.”** It is a versioned derived-state write that can race, lose provenance, or change future behavior.
- **“Telemetry is the event ledger.”** Telemetry may be sampled, dropped, renamed, or unavailable and cannot own recovery/effect correctness.

## Failure-injection backlog derived from research

- [ ] Cancel a `TaskGroup` while a child catches `CancelledError`; verify cancellation cannot silently become success.
- [ ] Race nested task-group failure with parent cancellation and inject a second cancel during cleanup.
- [ ] Expire `wait_for()` around slow cancellation; verify end-to-end budget accounts for settlement time.
- [ ] Call `Queue.shutdown(immediate=True)` with unfinished work; verify discard is explicit and observable.
- [ ] Cancel a `to_thread()` write after remote commit but before return; verify late result/effect fence.
- [ ] Kill a process worker with full stdout/stderr pipes; verify drain, reap, pool replacement, and reconciliation.
- [ ] Exercise the same pool entrypoint under Windows `spawn` and POSIX `forkserver`.
- [ ] Import/run the full native graph under subinterpreters and a free-threaded build.
- [ ] Stall an HTTP connection-pool acquisition and distinguish it from provider latency.
- [ ] Disconnect a slow SSE/WebSocket consumer at every provider-stream phase.
- [ ] Feed maximum compressed/decoded/schema-invalid model/tool output without unbounded allocation.
- [ ] Redeliver a broker message after effect commit but before ack; verify deduplication and prior receipt.
- [ ] Replay old workflow/checkpoint state after tool/schema/framework upgrade.
- [ ] Compare `tracemalloc`, native allocation, and RSS evidence for a controlled leak.
- [ ] Hang the event loop, a worker thread, and a child separately; verify diagnostics identify the boundary.
- [ ] Stall telemetry export during termination; verify bounded drops and exit within grace.
- [ ] Deploy a lock on every supported platform; verify no network resolution/source build occurs.
- [ ] SIGTERM/SIGKILL at each transition/effect phase; verify readiness, fencing, and recovery.
- [ ] Race context compaction with a newly appended event; verify stale compare-and-set cannot overwrite newer context.
- [ ] Retry compaction after ambiguous provider completion; verify one authoritative artifact per source digest/policy version.
- [ ] Delete or revoke long-term memory and exercise every cache/index/read path across tenant boundaries.
- [ ] Run recorded adapter fixtures against old/candidate SDKs and compare streams, usage, errors, schemas, cancellation, and close behavior.

## Guides produced from this packet

- [Python Agent Engineering](../../languages/python/README.md)
- [Architecture and ownership](../../languages/python/architecture-and-ownership.md)
- [`asyncio` structured concurrency and cancellation](../../languages/python/asyncio-structured-concurrency-and-cancellation.md)
- [Blocking, CPU, and native isolation](../../languages/python/blocking-cpu-and-native-isolation.md)
- [HTTP, streaming, and backpressure](../../languages/python/http-streaming-and-backpressure.md)
- [Tools, processes, and sandbox boundaries](../../languages/python/tools-processes-and-sandbox-boundaries.md)
- [Schemas, validation, and structured output](../../languages/python/schemas-validation-and-structured-output.md)
- [Errors, retries, and idempotency](../../languages/python/errors-retries-and-idempotency.md)
- [Queues, durable workers, and state](../../languages/python/queues-durable-workers-and-state.md)
- [Memory, GC, resources, and admission](../../languages/python/memory-gc-resources-and-admission.md)
- [Observability, profiling, and debugging](../../languages/python/observability-profiling-and-debugging.md)
- [Testing, fake time, races, and load](../../languages/python/testing-fake-time-races-and-load.md)
- [Packaging, dependencies, and supply chain](../../languages/python/packaging-dependencies-and-supply-chain.md)
- [Deployment, workers, shutdown, and crash recovery](../../languages/python/deployment-workers-shutdown-and-crash-recovery.md)
- [Ecosystem selection and anti-patterns](../../languages/python/ecosystem-selection-and-anti-patterns.md)

## Primary source register

All sources below were checked on 2026-08-31 unless a snapshot is explicitly versioned.

### CPython versions, trajectory, and concurrency

- [Status of Python versions](https://devguide.python.org/versions/)
- [Python 3.14.7 release](https://www.python.org/downloads/release/python-3147/)
- [PEP 790: Python 3.15 release schedule](https://peps.python.org/pep-0790/)
- [What's new in Python 3.15 (3.15.0rc1)](https://docs.python.org/3.15/whatsnew/3.15.html)
- [PEP 779: criteria for supported free-threaded Python](https://peps.python.org/pep-0779/)
- [PEP 803: `abi3t` stable ABI for free-threaded builds](https://peps.python.org/pep-0803/)
- [Python 3.14 free-threading HOWTO](https://docs.python.org/3.14/howto/free-threading-python.html)
- [`concurrent.futures`](https://docs.python.org/3.14/library/concurrent.futures.html)
- [`concurrent.interpreters`](https://docs.python.org/3.14/library/concurrent.interpreters.html)
- [`multiprocessing`](https://docs.python.org/3.14/library/multiprocessing.html)

### `asyncio` documentation and source

- [Coroutines and tasks](https://docs.python.org/3.14/library/asyncio-task.html)
- [Queues](https://docs.python.org/3.14/library/asyncio-queue.html)
- [Streams](https://docs.python.org/3.14/library/asyncio-stream.html)
- [Subprocesses](https://docs.python.org/3.14/library/asyncio-subprocess.html)
- [Runners](https://docs.python.org/3.14/library/asyncio-runner.html)
- [Event loop](https://docs.python.org/3.14/library/asyncio-eventloop.html)
- [Developing with asyncio](https://docs.python.org/3.14/library/asyncio-dev.html)
- [Asyncio exceptions](https://docs.python.org/3.14/library/asyncio-exceptions.html)
- [CPython 3.14 `taskgroups.py`](https://github.com/python/cpython/blob/3.14/Lib/asyncio/taskgroups.py)
- [CPython 3.14 `timeouts.py`](https://github.com/python/cpython/blob/3.14/Lib/asyncio/timeouts.py)
- [CPython 3.14 `queues.py`](https://github.com/python/cpython/blob/3.14/Lib/asyncio/queues.py)
- [CPython 3.14 `streams.py`](https://github.com/python/cpython/blob/3.14/Lib/asyncio/streams.py)
- [CPython 3.14 `runners.py`](https://github.com/python/cpython/blob/3.14/Lib/asyncio/runners.py)

### Process, trust, serialization, and resource boundaries

- [Python `subprocess`](https://docs.python.org/3.14/library/subprocess.html)
- [Python `tempfile`](https://docs.python.org/3.14/library/tempfile.html)
- [Python `os`](https://docs.python.org/3.14/library/os.html)
- [Python audit hooks and sandbox warning](https://docs.python.org/3.14/library/sys.html#sys.addaudithook)
- [Python `pickle` security warning](https://docs.python.org/3.14/library/pickle.html)
- [RestrictedPython](https://restrictedpython.readthedocs.io/en/latest/)
- [Docker seccomp profiles](https://docs.docker.com/engine/security/seccomp/)
- [gVisor security model](https://gvisor.dev/docs/architecture_guide/security/)
- [Kubernetes Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)

### ASGI, HTTP, streaming, and serving

- [ASGI main specification](https://asgi.readthedocs.io/en/latest/specs/main.html)
- [ASGI HTTP/WebSocket specification](https://asgi.readthedocs.io/en/latest/specs/www.html)
- [ASGI lifespan protocol](https://asgi.readthedocs.io/en/latest/specs/lifespan.html)
- [HTTPX async support](https://www.python-httpx.org/async/)
- [HTTPX timeouts](https://www.python-httpx.org/advanced/timeouts/)
- [HTTPX resource limits](https://www.python-httpx.org/advanced/resource-limits/)
- [HTTPX transports](https://www.python-httpx.org/advanced/transports/)
- [HTTPX exceptions](https://www.python-httpx.org/exceptions/)
- [Uvicorn server behavior](https://uvicorn.dev/server-behavior/)
- [Uvicorn settings](https://uvicorn.dev/settings/)
- [`uvicorn-worker`](https://github.com/Kludex/uvicorn-worker)

### Validation, schema, and configuration

- [Pydantic models](https://docs.pydantic.dev/latest/concepts/models/)
- [Pydantic strict mode](https://docs.pydantic.dev/latest/concepts/strict_mode/)
- [Pydantic unions](https://docs.pydantic.dev/latest/concepts/unions/)
- [Pydantic type adapters](https://docs.pydantic.dev/latest/concepts/type_adapter/)
- [Pydantic performance](https://docs.pydantic.dev/latest/concepts/performance/)
- [Pydantic serialization](https://docs.pydantic.dev/latest/concepts/serialization/)
- [Pydantic JSON Schema](https://docs.pydantic.dev/latest/concepts/json_schema/)
- [Pydantic settings](https://docs.pydantic.dev/latest/concepts/pydantic_settings/)
- [OpenAI structured outputs](https://developers.openai.com/api/docs/guides/structured-outputs)

### Workers, durability, and framework integration

- [Celery tasks](https://docs.celeryq.dev/en/latest/userguide/tasks.html)
- [Celery workers](https://docs.celeryq.dev/en/latest/userguide/workers.html)
- [Temporal Python workflow sandbox](https://docs.temporal.io/develop/python/python-sdk-sandbox)
- [Temporal Python failure detection](https://docs.temporal.io/develop/python/failure-detection)
- [Restate Python durable steps](https://docs.restate.dev/develop/python/durable-steps)
- [DBOS Python workflows](https://docs.dbos.dev/python/tutorials/workflow-tutorial)
- [Pydantic AI durable execution](https://ai.pydantic.dev/durable_execution/)
- [LangGraph durable execution/persistence](https://docs.langchain.com/oss/python/langgraph/durable-execution)
- [OpenAI Agents documentation](https://developers.openai.com/api/docs/guides/agents)
- [AnyIO tasks](https://anyio.readthedocs.io/en/stable/tasks.html)

### Errors, retries, memory, and observability

- [Python built-in exception groups](https://docs.python.org/3.14/library/exceptions.html#exception-groups)
- [Python `except*`](https://docs.python.org/3.14/reference/compound_stmts.html#except-star-clause)
- [Tenacity](https://tenacity.readthedocs.io/en/latest/)
- [AWS Builders' Library: timeouts, retries, backoff, and jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/)
- [Python garbage collector](https://docs.python.org/3.14/library/gc.html)
- [Python `tracemalloc`](https://docs.python.org/3.14/library/tracemalloc.html)
- [Python `resource`](https://docs.python.org/3.14/library/resource.html)
- [Python profilers](https://docs.python.org/3.14/library/profile.html)
- [Python `faulthandler`](https://docs.python.org/3.14/library/faulthandler.html)
- [Memray](https://github.com/bloomberg/memray)
- [OpenTelemetry Python](https://opentelemetry.io/docs/languages/python/)
- [OpenTelemetry Python instrumentation](https://opentelemetry.io/docs/languages/python/instrumentation/)
- [OpenTelemetry Python exporters](https://opentelemetry.io/docs/languages/python/exporters/)
- [OpenTelemetry GenAI agent and framework spans](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md)
- [OpenTelemetry GenAI conventions migration notice](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
- [Prometheus Python multiprocess mode](https://prometheus.github.io/client_python/multiprocess/)

### Testing, packaging, and deployment

- [`unittest.IsolatedAsyncioTestCase`](https://docs.python.org/3.14/library/unittest.html#unittest.IsolatedAsyncioTestCase)
- [`unittest.mock.AsyncMock`](https://docs.python.org/3.14/library/unittest.mock.html#unittest.mock.AsyncMock)
- [`pytest-asyncio` concepts](https://pytest-asyncio.readthedocs.io/en/stable/concepts.html)
- [`pytest-asyncio` configuration](https://pytest-asyncio.readthedocs.io/en/stable/reference/configuration.html)
- [AnyIO testing](https://anyio.readthedocs.io/en/stable/testing.html)
- [Hypothesis stateful testing](https://hypothesis.readthedocs.io/en/latest/stateful.html)
- [Python monotonic clocks](https://docs.python.org/3.14/library/time.html#time.monotonic)
- [`pylock.toml` specification](https://packaging.python.org/en/latest/specifications/pylock-toml/)
- [`pip lock`](https://pip.pypa.io/en/stable/cli/pip_lock/)
- [Pip secure installs](https://pip.pypa.io/en/stable/topics/secure-installs/)
- [Pip build-system interface](https://pip.pypa.io/en/stable/reference/build-system/)
- [PyPI Trusted Publishing](https://docs.pypi.org/trusted-publishers/)
- [PyPI digital attestations](https://docs.pypi.org/attestations/)
- [`pip-audit`](https://github.com/pypa/pip-audit)
- [Python signals](https://docs.python.org/3.14/library/signal.html)
- [Python `atexit`](https://docs.python.org/3.14/library/atexit.html)
- [Kubernetes Pod termination](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination)
- [Kubernetes startup/readiness/liveness probes](https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/)

## Refresh plan

Refresh this packet when any of these occur:

1. CPython 3.15 final ships; replace RC findings with final patch docs.
2. The production baseline changes from 3.14.7 or adopts a free-threaded/JIT build.
3. `asyncio` changes task-group, timeout, queue, runner, stream, or shutdown semantics.
4. A critical native package claims free-threaded/subinterpreter/`abi3t` support.
5. Uvicorn/ASGI/HTTPX changes flow-control, disconnect, worker, or shutdown behavior.
6. Pydantic changes strict/coercion/serialization/schema/version policy.
7. Celery or a durable engine changes ack/replay/cancellation/versioning semantics.
8. OpenTelemetry generative-AI conventions or Python signal maturity changes materially.
9. Pip lock/`pylock.toml`, PyPI attestation, or secure-install behavior stabilizes/changes.

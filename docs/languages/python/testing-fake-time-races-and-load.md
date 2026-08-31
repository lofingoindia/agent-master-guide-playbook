# Testing, Fake Time, Races, and Load for Python Agents

> **Research date:** 2026-08-31  
> **Related:** [Evaluation-driven development](../../evaluation/evaluation-driven-development.md) and [trajectory/reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md)

Agent tests must verify two systems at once: probabilistic model behavior and deterministic runtime contracts. Keep them separate. A mocked model can test cancellation, ownership, retries, state, and effects deterministically; dataset evals can test model choices without being the only reliability gate.

## Use a layered test portfolio

| Layer | Purpose | Model/tool strategy |
|---|---|---|
| Pure unit | State transitions, validation, budgets, retry classification | No network; real schemas and clocks |
| Async component | Task ownership, queue/backpressure, cancellation, streams | Scripted async fakes |
| Adapter contract | Exact provider/tool SDK semantics | Recorded/sandbox service or vendor test endpoint |
| Persistence/recovery | Crash, replay, migration, outbox, idempotency | Real database/broker/engine in ephemeral environment |
| System failure injection | Signals, worker loss, slow consumers, overload | Real server/process/sandbox |
| Model eval | Tool selection, answer quality, adversarial inputs | Pinned model/config and datasets |
| Load/soak | Capacity, tail latency, leak/resource behavior | Representative response sizes and timing |

Do not use live model calls in every unit test. Do not replace every adapter with a loose `Mock` that accepts invalid arguments and hides SDK drift.

## Make async fakes obey the real contract

Use `AsyncMock` with `autospec`/`spec_set` or small hand-written fakes. Python distinguishes “called” from “awaited”; assert awaits. Script realistic behavior:

- delayed connect and chunk gaps;
- partial stream then exception;
- cancellation during await and during cleanup;
- rate limit with retry metadata;
- side effect committed but response lost;
- malformed/oversize payload;
- close failure and late result.

A fake that returns instantly cannot reveal missing backpressure, task interleaving, timeout budgeting, or pool contention.

## Isolate event-loop lifetime deliberately

`unittest.IsolatedAsyncioTestCase` creates a runner/loop per test and supports `asyncSetUp`, `asyncTearDown`, `addAsyncCleanup`, and async context entry. In Python 3.13+, `loop_factory` can avoid the deprecated policy system.

`pytest-asyncio` has strict and auto discovery modes. Current docs say async tests still run sequentially by default and expose separate fixture/test loop-scope configuration. Set mode and loop scopes explicitly; default changes can otherwise alter fixture lifetime. Use strict mode when multiple async libraries/plugins must coexist.

AnyIO's pytest plugin can exercise asyncio and Trio backends for code genuinely written against AnyIO. Do not claim backend portability for code that directly imports asyncio primitives.

At test teardown, fail on:

- pending non-whitelisted tasks;
- unclosed clients/responses/files/async generators;
- never-awaited coroutines or unretrieved exceptions;
- queue unfinished-task count;
- leaked subprocesses/threads/workspaces;
- unexpected telemetry exporter backlog.

Enable asyncio debug and `ResourceWarning` in targeted CI jobs.

## Inject time at policy boundaries

Use monotonic time for deadlines and wall time only for human/audit timestamps.

```python
class Clock(Protocol):
    def monotonic(self) -> float: ...
    def now_utc(self) -> datetime: ...
    async def sleep(self, seconds: float) -> None: ...
```

Inject the clock into retry, lease, approval-expiry, and state-machine policy. Advance a fake clock deterministically for those unit tests.

Do not assume freezing `datetime.now()` or `time.time()` advances the event loop. `asyncio` schedules from a monotonic clock (`loop.time()`), and patching it incorrectly can violate loop internals. Keep a smaller set of integration tests on the real loop with short tolerances. Assert ordering/state and broad upper bounds, not exact millisecond sleeps on shared CI.

## Make race schedules reproducible

Replace `sleep()`-based coordination with barriers/events controlled by the test:

```python
import asyncio


async def reproduce_commit_before_reply() -> None:
    reached_effect = asyncio.Event()
    allow_reply = asyncio.Event()
    committed: set[str] = set()

    async def tool(effect_id: str) -> dict[str, str]:
        committed.add(effect_id)  # stand-in for an external commit
        reached_effect.set()
        await allow_reply.wait()
        return {"effect_id": effect_id}

    task = asyncio.create_task(tool("effect-123"))
    await reached_effect.wait()
    task.cancel()  # response is lost after the effect happened
    result = await asyncio.gather(task, return_exceptions=True)

    assert "effect-123" in committed
    assert isinstance(result[0], asyncio.CancelledError)


asyncio.run(reproduce_commit_before_reply())
```

The test can cancel/crash after commit but before receipt, then verify reconciliation. Create similar gates before/after queue ack, checkpoint commit, stream close, lease expiry, and sibling failure.

Vary schedules:

- cancel first versus failure first;
- provider finishes while client disconnects;
- two workers claim/recover the same version;
- shutdown races retry/backoff;
- queue shutdown races producer `put()`;
- process exits while results remain queued;
- duplicate message arrives before and after the first commit.

Run key race tests repeatedly with recorded seeds. Hypothesis state machines are useful for sequences of start/cancel/retry/approve/recover operations and invariants after every step. They do not control the OS/event-loop schedule automatically; combine them with explicit gates and a model of expected state.

## Test ASGI lifecycle, not only handlers

HTTPX `ASGITransport` calls the app in process, but HTTPX explicitly says it does not trigger ASGI lifespan. Pair it with a lifespan manager or drive startup/shutdown through the server/framework test facility.

Test:

- startup failure blocks readiness;
- one client/pool/supervisor is created per worker loop;
- response generator closes provider stream on disconnect;
- graceful shutdown stops admission before drain;
- hard shutdown interrupts and recovery resumes from persisted state;
- overload returns intended 503/429 rather than accumulating memory.

Run a real-socket test too: in-process transports cannot reproduce proxy, TCP buffering, keep-alive churn, half-close, signal, or process-manager behavior.

## Failure-injection matrix

| Injection point | Required invariant |
|---|---|
| Cancel during model stream | response closes; no unowned pump; terminal state fenced |
| Child failure races parent cancellation | nested task groups preserve failure/cancellation evidence and settle every child |
| Second cancellation arrives during cleanup | cleanup is idempotent, bounded, and cannot publish two terminals |
| Awaited operation delays or swallows cancellation | wall time may exceed local `wait_for()`; end-to-end deadline/escalation still wins |
| Cancel while thread call continues | late return cannot mutate completed run/effect twice |
| Kill child after external commit | ambiguous effect reconciles by stable ID |
| Crash worker after DB commit before broker ack | duplicate delivery deduplicates |
| Slow SSE/WebSocket consumer | bounded memory and defined coalesce/drop/cancel policy |
| Telemetry exporter stalls | run and shutdown stay within budget |
| Database/broker unavailable | no open transaction across backoff; outbox evidence preserved |
| SIGTERM at each phase | readiness drops, drain/recovery semantics hold |
| Schema/tool version upgrade | old checkpoints/messages migrate or fail explicitly |
| Context compaction races a new event | compare-and-set rejects stale summary; source range/digest remains auditable |
| Memory retrieval crosses tenant/deletion boundary | no result is exposed; cache/index deletion behavior is verified |
| Provider/agent SDK upgrade | recorded adapter fixtures detect event/error/usage/schema/lifecycle drift |

## Load and soak by workload class

Measure separately:

- short interactive runs;
- long streams with slow consumers;
- tool-heavy fan-out;
- blocking/native/process work;
- durable resumes/approval bursts;
- maximum-size validated inputs and outputs.

Observe p95/p99 end-to-end and queue wait, event-loop lag, pool/executor wait, active tasks, retry amplification, RSS/fds/children, GC/allocation rate, cancellation cleanup, and telemetry drops. Increase load until the admission gate sheds predictably; do not define capacity as the point where the process crashes.

Soak across worker recycling and dependency/network churn. Assert steady-state slopes for RSS, tasks, descriptors, and checkpoint/event-history bytes, not only final values.

## Release gate

- [ ] Unit/runtime tests use scripted fakes with real validation/contracts.
- [ ] Loop mode/scope is explicit and teardown detects leaked async resources.
- [ ] Deadline policy uses an injected monotonic clock; real-loop tests remain.
- [ ] Race tests use gates and cover commit-before-response ambiguity.
- [ ] Persistence tests use real database/broker/durable runtime behavior.
- [ ] ASGI lifespan, real sockets, signals, and process loss are exercised.
- [ ] Load reaches controlled admission rejection without unbounded RSS/queues.
- [ ] Model evals are pinned, versioned, and separate from deterministic reliability tests.
- [ ] Upgrade tests read old state/messages and compare generated schemas.
- [ ] Compaction/retrieval tests cover stale writers, provenance, tenant scope, retention, and deletion.
- [ ] Exact-version third-party adapter fixtures cover stream settlement, cancellation, errors, and lifecycle.

## Selected primary sources

- [`unittest.IsolatedAsyncioTestCase`](https://docs.python.org/3.14/library/unittest.html#unittest.IsolatedAsyncioTestCase) and [`AsyncMock`](https://docs.python.org/3.14/library/unittest.mock.html#unittest.mock.AsyncMock)
- [`pytest-asyncio` concepts](https://pytest-asyncio.readthedocs.io/en/stable/concepts.html) and [configuration](https://pytest-asyncio.readthedocs.io/en/stable/reference/configuration.html)
- [AnyIO testing](https://anyio.readthedocs.io/en/stable/testing.html)
- [Hypothesis stateful testing](https://hypothesis.readthedocs.io/en/latest/stateful.html)
- [Python monotonic clocks](https://docs.python.org/3.14/library/time.html#time.monotonic) and [`asyncio` event-loop time](https://docs.python.org/3.14/library/asyncio-eventloop.html#asyncio.loop.time)
- [HTTPX ASGI transport and lifespan note](https://www.python-httpx.org/advanced/transports/#asgi-transport)

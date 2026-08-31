# `asyncio` Structured Concurrency and Cancellation

> **Research date:** 2026-08-31  
> **Related:** [Run controls](../../runtime/run-controls.md) and [queues, scheduling, and backpressure](../../operations/queues-scheduling-and-backpressure.md)

`asyncio` is a cooperative scheduler. It can coordinate large amounts of I/O efficiently when every task yields, ownership is structured, and queues are bounded. It cannot preempt Python code that does not await, stop a worker thread, roll back an effect, or persist progress through process loss.

## Use a task tree, not a task cloud

`asyncio.TaskGroup` is the default ownership primitive for related work:

- it waits for every child before exit;
- the first non-cancellation failure cancels siblings;
- it waits for sibling cancellation to settle;
- it raises remaining failures as an `ExceptionGroup`/`BaseExceptionGroup`;
- Python 3.13 improved simultaneous internal/external cancellation and cancellation-count preservation;
- Python 3.14 forwards task-creation keyword arguments, including `eager_start`, to the loop.

```python
async def collect_tools(calls: list[ToolCall]) -> dict[str, ToolResult]:
    tasks: dict[str, asyncio.Task[ToolResult]] = {}
    async with asyncio.TaskGroup() as group:
        for call in calls:
            tasks[call.id] = group.create_task(
                execute_tool(call), name=f"tool:{call.id}"
            )
    return {call_id: task.result() for call_id, task in tasks.items()}
```

The example intentionally lets one unexpected failure cancel siblings. If partial success is a product requirement, convert **expected** per-tool failures into typed result values inside each child. Do not broadly suppress infrastructure failures or cancellation.

`asyncio.gather()` still has uses for result aggregation, but its default failure semantics are not sibling-failure containment. Prefer `TaskGroup` for lifecycle ownership.

## Treat cancellation as a protocol

`Task.cancel()` requests cancellation. `CancelledError` is injected at the next suspension opportunity and subclasses `BaseException`, not `Exception`.

```python
async def consume() -> None:
    lease = await acquire_lease()
    try:
        await process(lease)
    finally:
        async with asyncio.timeout(2):
            await lease.release_best_effort()
```

The cleanup timeout is separate from the run timeout. After `finally`, cancellation continues automatically. If code explicitly catches `CancelledError`, it should generally re-raise after cleanup.

Do not routinely call `uncancel()`. Task groups and timeout contexts use cancellation internally and depend on its state. Swallowing cancellation can make a parent timeout appear to succeed or make a task group misbehave.

Repeated cancellation is possible. A shutdown escalation or parent failure can call `cancel()` again while cleanup is awaiting; `Task.cancelling()` records pending cancellation requests. Therefore cleanup must be idempotent, separately bounded, and safe to interrupt. If cleanup must commit a tiny receipt/release, give that fragment an explicit owner and deadline—do not assume a single `finally` block is uninterruptible.

### Record the cancellation lifecycle

| Moment | What to record |
|---|---|
| Requested | source, reason, remaining deadline, task/run IDs |
| Observed | component and suspension point/class of work |
| Cleanup started | resources/effects requiring settlement |
| Child settled | success, cancelled, failed, or still uncooperative |
| Terminal fenced | persisted final version and late-result policy |

A “cancelled” HTTP response is not proof that a thread, child process, provider request, or side effect stopped.

## Build one absolute deadline hierarchy

Use the event loop's monotonic clock and derive all child budgets from one absolute run deadline.

```python
async with asyncio.timeout_at(run_deadline):
    await run_turn()
```

`asyncio.timeout()`/`timeout_at()` cancel the current task and translate the cancellation to built-in `TimeoutError` **outside** the context when no newer cancellation supersedes it. The context can be inspected and rescheduled. `wait_for()` also cancels the awaited operation and, since Python 3.7, waits for its cancellation to complete, so elapsed wall time may exceed the numeric timeout.

| Budget | Meaning | Failure action |
|---|---|---|
| Admission wait | Time allowed before execution begins | reject or durable enqueue |
| Run deadline | End-to-end contract | stop new steps, cancel run tree |
| Provider connect/read gap | One transport phase | classify transport failure within remaining run time |
| Tool attempt | One attempt | cancel/cooperate, then fence late work |
| Queue put/get | Backpressure wait | shed, coalesce, or fail by policy |
| Cleanup | Settlement after stop | escalate to worker/process termination |

Never stack independent default timeouts whose sum can exceed the user contract. Never retry after the remaining budget is smaller than the minimum useful attempt plus cleanup reserve.

## Use shielding only for a tiny owned fragment

`asyncio.shield()` prevents cancellation of the wrapped awaitable when the caller is cancelled; the caller still receives `CancelledError`. It is appropriate only for a small commit/release fragment with a separate deadline and an owner that retains a strong task reference.

It is not appropriate for a whole model call, long tool, or background continuation. Shielding without an owner turns cancellation into hidden work.

## Make queue capacity and shutdown explicit

`asyncio.Queue(maxsize=0)` is unbounded. A positive `maxsize` blocks `put()` when full, but counts items rather than bytes. The queue is not thread-safe and has no built-in operation timeout; wrap operations in a timeout context.

Python 3.13 added `Queue.shutdown()` and `QueueShutDown`. In normal shutdown, producers are rejected while consumers can drain existing items and `join()` retains its accounting invariant. `shutdown(immediate=True)` drains and unblocks immediately; the documentation warns that it can unblock `join()` even though work was not completed.

For each queue, define:

- owner and consumers;
- item and byte capacity;
- admission/fairness policy;
- `task_done()` responsibility in `finally`;
- normal drain versus immediate discard;
- persistence boundary;
- metric for oldest item age.

An `asyncio.Queue` is a process-local scheduling primitive, never a durable work ledger.

## Know the runner shutdown contract

Use `asyncio.run()` or `asyncio.Runner` at a process entry point. They finalize async generators, cancel remaining tasks, and shut down the default executor. The default executor shutdown has a five-minute timeout in the documented runner behavior—far too long to be the application's only shutdown control.

Configure event loops through `loop_factory`; the policy system is deprecated and scheduled for removal in Python 3.16. Do not scatter `get_event_loop()` and custom policy assumptions through libraries.

`Runner` installs SIGINT handling in the main thread: the first Ctrl-C cancels the main task so `finally` blocks can run; a second can raise `KeyboardInterrupt` immediately if a tight loop will not cooperate. Production SIGTERM handling still belongs to the server/process supervisor lifecycle.

## Detect blocking and leaked async work

In staging and selected tests, enable asyncio debug mode and `ResourceWarning` visibility. Debug mode can report:

- callbacks invoked from the wrong thread;
- selector operations that take too long;
- callbacks exceeding `loop.slow_callback_duration` (100 ms by default);
- never-awaited coroutines;
- never-retrieved task exceptions.

Name tasks with run/tool identifiers. On a hang, inspect `asyncio.all_tasks()` and task stacks, but do not expose prompts, secrets, or untrusted payloads in names.

## Cancellation failure matrix

| Work type | What task cancellation does | Required extra control |
|---|---|---|
| Cooperative coroutine | Raises at an await and unwinds | bounded cleanup and re-raise |
| Provider HTTP stream | Cancels local await; transport behavior is client-specific | close response/client stream; record provider request ID |
| `to_thread()` call | Cancels waiter, not underlying thread | library deadline, late-result fence, process if kill is required |
| Child process | Cancels waiter, not necessarily child | terminate, grace, kill, drain pipes, reap |
| External write | Stops local continuation at best | idempotency key and reconciliation |
| Durable activity | Runtime-specific cancellation request | heartbeat/cooperation and workflow policy |

## Verification checklist

- [ ] Every run child is inside a `TaskGroup` or named service supervisor.
- [ ] Expected partial failures are values; unexpected failures still cancel siblings.
- [ ] `CancelledError` is not caught by a broad `BaseException` handler.
- [ ] Repeated cancellation during cleanup is injected; cleanup remains bounded and idempotent.
- [ ] Nested task groups are tested when child failure and parent cancellation happen together.
- [ ] Timeouts derive from one monotonic absolute deadline.
- [ ] Cleanup has its own small budget and escalation path.
- [ ] All queues have positive item limits and separately enforced byte budgets.
- [ ] Queue immediate shutdown is tested for intentionally discarded work.
- [ ] Debug mode tests detect leaked tasks, async generators, and resources.
- [ ] Blocking/cpu/process behavior is tested under cancellation, not inferred.

## Selected primary sources

- [`asyncio` coroutines, tasks, cancellation, task groups, timeouts, and shielding](https://docs.python.org/3.14/library/asyncio-task.html)
- [`asyncio` queues and shutdown](https://docs.python.org/3.14/library/asyncio-queue.html)
- [`asyncio` runners and SIGINT handling](https://docs.python.org/3.14/library/asyncio-runner.html)
- [Developing with `asyncio`](https://docs.python.org/3.14/library/asyncio-dev.html)
- [CPython `TaskGroup` implementation](https://github.com/python/cpython/blob/3.14/Lib/asyncio/taskgroups.py) and [timeout implementation](https://github.com/python/cpython/blob/3.14/Lib/asyncio/timeouts.py)

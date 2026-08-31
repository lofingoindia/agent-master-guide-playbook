# Cancellation, Timeouts, and Graceful Shutdown

> **Last researched:** 2026-08-31
> **Applies to:** .NET 10 LTS, C# 14

Cancellation is a cooperative protocol, not thread termination. A robust agent distinguishes the caller leaving, the total run deadline, an attempt deadline, host shutdown, and operator cancellation. It then defines which source determines the externally visible terminal state.

## Three different operations

| Intent | Mechanism | What stops |
|---|---|---|
| Cancel underlying work | Pass <code>CancellationToken</code> into the API | Work, if the implementation cooperates |
| Stop waiting | <code>WaitAsync(token)</code> or timeout overload | Only the caller's wait |
| Bound both | Pass token into work and bound the returned task | Wait and cooperative work |

Canceling a <code>WaitAsync</code> does not prove that an HTTP request, subprocess, tool effect, or provider run stopped. The abandoned operation still needs an owner that observes its completion and cleans it up.

## Deadline hierarchy

~~~mermaid
flowchart TD
    Caller[Caller token] --> Link[Linked run token]
    Host[Host stopping token] --> Link
    Total[Total run deadline] --> Link
    Link --> Step[Step token]
    Attempt[Per-attempt deadline] --> Step
    Step --> Model[Provider call]
    Step --> Tool[Tool call]
~~~

Use <code>CancellationTokenSource.CreateLinkedTokenSource</code> at explicit ownership boundaries. Dispose linked sources and timers. Do not create a new timeout independently in every layer; that produces conflicting clocks and obscures the reason a run stopped.

Useful budget fields are:

- admitted-at and absolute run deadline;
- per-attempt maximum;
- maximum idle gap between stream events;
- tool execution maximum;
- persistence/settlement reserve;
- shutdown drain deadline.

Reserve time for recording a terminal state and settling a queue message. A model call consuming the entire budget leaves the system unable to say what happened.

## Correct cancellation handling

~~~csharp
public async Task<ModelResult> ExecuteAttemptAsync(
    ModelRequest request,
    TimeSpan attemptLimit,
    TimeProvider timeProvider,
    CancellationToken runToken)
{
    using var timeout = new CancellationTokenSource(attemptLimit, timeProvider);
    using var attempt = CancellationTokenSource.CreateLinkedTokenSource(
        runToken, timeout.Token);

    try
    {
        return await model.SendAsync(request, attempt.Token);
    }
    catch (OperationCanceledException) when (runToken.IsCancellationRequested)
    {
        throw;
    }
    catch (OperationCanceledException) when (timeout.IsCancellationRequested)
    {
        throw new AttemptTimedOutException(attemptLimit);
    }
}
~~~

The filters preserve provenance. Avoid translating every <code>OperationCanceledException</code> into a timeout. <code>ThrowIfCancellationRequested</code> preserves the requested token in the exception.

Use <code>TimeProvider</code> overloads for delays, waits, and testable timeout behavior. Never implement retries with <code>Thread.Sleep</code>.

## Streaming deadlines

One total timeout is insufficient for streaming. Track:

1. time to response headers;
2. time to first meaningful event;
3. maximum silence between events;
4. total stream duration;
5. maximum bytes/events/tokens.

An idle timeout should reset only after a valid, accepted frame. Garbage bytes or keepalive traffic should not necessarily extend business progress forever.

## Graceful shutdown sequence

~~~mermaid
sequenceDiagram
    participant O as Orchestrator
    participant H as Host
    participant A as Admission
    participant R as Run owners
    participant Q as Durable queue
    O->>H: termination signal
    H->>A: fail readiness and stop admission
    H->>R: begin bounded drain
    R->>Q: persist checkpoints and settle safe work
    alt drain deadline expires
        H->>R: cancel remaining work
        R->>R: terminate child processes / dispose streams
    end
    H->>H: flush bounded telemetry and dispose
~~~

The Generic Host default shutdown timeout is finite and configurable. The cancellation token passed to <code>StopAsync</code> signals that graceful time has elapsed, but the host still awaits the task. Therefore, each worker must enforce its own bounded escalation. Also assume <code>StopAsync</code> will not execute after a crash or forced termination.

Queue settlement policy matters:

- complete only after effects and terminal state are durable;
- abandon/release work whose outcome is safely retryable;
- dead-letter poison work with a safe reason and operator path;
- do not settle an ambiguous external effect as failed until reconciled.

## Process escalation

Canceling <code>WaitForExitAsync</code> cancels the wait, not necessarily the process. A tool runner needs an explicit sequence: stop input, send a cooperative signal when supported, wait briefly, kill the process tree, continue draining redirected streams, await exit, and record whether descendants may have survived.

On .NET 10, Windows process-group support improves signaling options, but process-tree and signal behavior still varies by operating system and container runtime. Test the published artifact.

## Failure patterns

- A timeout wrapper returns while the underlying SDK request continues consuming sockets and quota.
- A retry starts before the timed-out attempt is confirmed stopped, causing overlapping effects.
- Shutdown cancels work before readiness is removed, so new work keeps arriving.
- The worker spends its whole shutdown budget on a provider call and cannot checkpoint.
- A broad <code>catch (Exception)</code> converts host cancellation into a retriable failure.
- An unbounded telemetry flush prevents process exit.

## Review checklist

- [ ] Caller, host, total, attempt, idle, and tool deadlines are distinguishable.
- [ ] Underlying APIs receive the appropriate token.
- [ ] Abandoned waits retain an owner for cleanup.
- [ ] Shutdown stops admission, drains, then cancels and escalates.
- [ ] Persistence and queue settlement have reserved time.
- [ ] Tests use <code>FakeTimeProvider</code> rather than wall-clock sleeps.

## Primary sources

- [Cancel async tasks after a period of time](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/cancel-async-tasks-after-a-period-of-time)
- [Cancel non-cancelable async operations](https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/cancel-non-cancelable-async-operations)
- [Task.WaitAsync API](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task-1.waitasync?view=net-10.0)
- [Exception best practices](https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions)
- [Generic Host shutdown](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-10.0)
- [.NET 10 library changes for processes](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/libraries)
- [TimeProvider overview](https://learn.microsoft.com/en-us/dotnet/standard/datetime/timeprovider-overview)

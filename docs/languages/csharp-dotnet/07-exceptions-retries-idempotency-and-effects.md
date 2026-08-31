# Exceptions, Retries, Idempotency, and Effects

> **Last researched:** 2026-08-31
> **Applies to:** .NET 10 LTS, C# 14

Retry is a semantic decision, not an exception-handling reflex. The runtime needs to know whether the operation was rejected before execution, failed before an effect, completed with a lost acknowledgment, or returned a permanent business error.

## Failure taxonomy

| Class | Examples | Default action |
|---|---|---|
| Cancellation | Caller left, host stopping | Stop and preserve reason |
| Deadline | Attempt or total budget expired | Retry only with budget and replay safety |
| Transient transport | Connection reset, selected 5xx | Backoff with jitter |
| Throttling | 429, provider capacity | Honor retry guidance; reduce admission |
| Permanent request | Auth, invalid schema, unsupported model | Do not retry unchanged |
| Business rejection | Approval denied, insufficient funds | Terminal domain result |
| Ambiguous effect | Timeout after remote commit may have happened | Reconcile by effect ID |
| Bug/invariant | Null state, impossible transition | Fail loudly and quarantine work |

Map provider and infrastructure exceptions into a small domain taxonomy while retaining the original exception as the cause. Do not expose provider-specific classes throughout the run engine.

## Exception practices

- Throw synchronously for argument validation before starting asynchronous work.
- Catch the most specific exception at the layer that can add policy.
- Let cancellation propagate; use exception filters to distinguish its source.
- Preserve stack traces with <code>throw;</code>, not <code>throw error;</code>.
- Do not use exceptions for expected tool-domain outcomes.
- Observe all child task failures. Awaiting <code>Task.WhenAll</code> may throw one while its task contains multiple.
- Include safe run, attempt, provider request, message, and effect IDs in structured logs.

## Retry algorithm

~~~mermaid
flowchart TD
    Fail[Attempt failed] --> Classify{Classify}
    Classify -->|permanent/cancel| Stop[Terminal result]
    Classify -->|ambiguous effect| Query[Reconcile by effect ID]
    Query -->|committed| Success[Record receipt]
    Query -->|not committed| Safe{Replay safe?}
    Classify -->|transient| Safe
    Safe -->|no| Stop
    Safe -->|yes| Budget{Budget remains?}
    Budget -->|no| Stop
    Budget -->|yes| Delay[Backoff + jitter]
    Delay --> Attempt[Next attempt]
~~~

Bound retries by maximum attempts, total elapsed time, remaining token/cost budget, and shutdown deadline. A circuit breaker protects a dependency; it does not make an individual effect idempotent.

## Effect protocol

Every side effect should have a stable <code>EffectId</code> derived from the run, logical step, tool version, and business operation. Persist an intent record before invocation:

| Field | Purpose |
|---|---|
| Effect ID | Deduplication and reconciliation key |
| Run/step/tool version | Provenance |
| Normalized input hash | Detect conflicting reuse |
| State | Planned, executing, committed, rejected, unknown |
| Provider receipt | External transaction/request identifier |
| Attempt history | Diagnosis without duplicate business state |
| Result reference | Bounded durable output |

The tool adapter should pass the effect ID to downstream idempotency support when available. On lost acknowledgment, query by that key. If the downstream API cannot deduplicate or query, require approval, serialize the action, or design a compensating business workflow. A local retry loop cannot solve an unknowable remote outcome.

### Claim, execute, reconcile

The effect record is also a concurrency boundary. Claim it with an optimistic version or fencing token before calling the dependency. This application-level sketch makes the ambiguous branch explicit:

~~~csharp
public async Task<EffectReceipt> ApplyAsync(
    EffectCommand command,
    CancellationToken token)
{
    string effectId = EffectIds.For(command.RunId, command.StepId, command.Version);
    string inputHash = CanonicalHash(command.Input);

    EffectClaim claim = await effects.ClaimAsync(effectId, inputHash, token);
    if (claim.ExistingState == EffectState.Committed)
        return claim.Receipt!;

    if (!claim.ExecutionGranted) // another/stale owner or an unknown outcome
        return await reconciler.ResolveAsync(effectId, inputHash, token);

    try
    {
        EffectReceipt receipt = await downstream.ExecuteAsync(
            command.Input, idempotencyKey: effectId, token: token);
        await effects.CommitAsync(effectId, claim.Fence, receipt, token);
        return receipt;
    }
    catch (DefinitelyNotSentException)
    {
        await effects.ReleaseForRetryAsync(effectId, claim.Fence, token);
        throw;
    }
    catch (EffectRejectedException error)
    {
        await effects.RejectAsync(effectId, claim.Fence, error.Code, token);
        throw;
    }
    catch (Exception error) when (CouldBeAmbiguous(error))
    {
        await effects.MarkUnknownAsync(effectId, claim.Fence, token);
        throw new ReconciliationRequiredException(effectId, error);
    }
}
~~~

<code>ClaimAsync</code> must reject the same effect ID with a different normalized input hash. Only evidence that the request definitely did not leave the process permits ordinary replay; a timeout, disconnect, process crash, lost persistence acknowledgment, or abandoned <code>Executing</code> claim normally enters reconciliation. Reconciliation may return committed, definitely absent, rejected, or still unknown. "Still unknown" is an operational state with an owner and deadline, not permission to retry. Treat any unclassified exception as an invariant/adapter defect and quarantine it rather than adding a catch-all retry.

## Queue and SDK retries

Provider SDK defaults are part of the attempt budget. At the research date:

- the official OpenAI .NET client automatically retries selected 408, 429, and 5xx responses up to three additional times;
- the official Anthropic C# SDK retries connection failures and selected 408, 409, 429, and 5xx responses twice by default;
- <code>Microsoft.Extensions.Http.Resilience</code> can add its own retries;
- a broker can redeliver after lock loss or process failure.

Pin versions and verify these settings at upgrade time. Report logical operations and network attempts separately.

## Idempotency is scoped

- **Message deduplication** prevents some duplicate enqueue operations within a window.
- **Inbox deduplication** prevents reprocessing one message ID.
- **Effect idempotency** prevents duplicate external business change.
- **State-transition uniqueness** prevents two owners committing the same transition.

None implies the others. Azure Service Bus duplicate detection, for example, uses an application-controlled message ID within a configured window; consumers must still be idempotent because messages can be redelivered.

## Failure patterns

- Retrying all exceptions, including authentication and invalid schemas.
- Assuming POST is unsafe or GET is safe without considering the business effect.
- Starting a second attempt while the first timed-out operation may still run.
- Retrying a tool after partial streaming output caused an effect.
- Treating broker deduplication as exactly-once delivery.
- Reusing an effect ID with different normalized input.
- Logging an error and completing the queue message before durable terminal state.

## Review checklist

- [ ] Exceptions map to a documented retry taxonomy.
- [ ] Retry layers and maximum amplification are counted.
- [ ] Backoff honors server guidance, jitter, and remaining budget.
- [ ] Every external write has an effect ID and durable intent.
- [ ] Concurrent owners cannot execute the same effect claim with different fences or input hashes.
- [ ] Ambiguous outcomes enter reconciliation, not blind replay.
- [ ] Queue settlement follows durable state/effect commits.
- [ ] Tests inject failure before send, after commit, and before acknowledgment.

## Primary sources

- [.NET exception best practices](https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions)
- [Task exception handling](https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/exception-handling-task-parallel-library)
- [.NET HTTP resilience](https://learn.microsoft.com/en-us/dotnet/core/resilience/http-resilience)
- [OpenAI .NET automatic retries](https://github.com/openai/openai-dotnet#automatically-retrying-errors)
- [Anthropic C# retries](https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/csharp)
- [Azure Service Bus duplicate detection](https://learn.microsoft.com/en-us/azure/service-bus-messaging/duplicate-detection)
- [Service Bus transfers, locks, and settlement](https://learn.microsoft.com/en-us/azure/service-bus-messaging/message-transfers-locks-settlement)

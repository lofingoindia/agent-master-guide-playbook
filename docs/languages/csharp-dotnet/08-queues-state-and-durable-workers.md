# Queues, State, and Durable Workers

> **Last researched:** 2026-08-31
> **Applies to:** .NET 10 LTS, C# 14

In-memory tasks and channels coordinate work inside one process. They do not make work durable. If a run must survive process loss, deploys, long delays, or human approval, persist the state transition and resume from a durable activation.

## State machine

~~~mermaid
stateDiagram-v2
    [*] --> Admitted
    Admitted --> Running: lease acquired
    Running --> Waiting: external wait / approval
    Waiting --> Running: durable signal
    Running --> Succeeded: state and effects committed
    Running --> Retryable: classified transient failure
    Retryable --> Running: next activation
    Running --> Reconciliation: effect outcome unknown
    Reconciliation --> Running: safe to continue
    Reconciliation --> Succeeded: receipt found
    Running --> Failed: permanent failure
    Running --> Canceled: cancellation persisted
    Succeeded --> [*]
    Failed --> [*]
    Canceled --> [*]
~~~

Persist explicit states and transition versions. A conversation transcript is not sufficient runtime state; it does not capture leases, attempts, approvals, effect receipts, or schema versions.

## Minimum persistent contract

Keep commands, authoritative state, semantic events, effect records, delivery events, and telemetry distinct. They have different retention, ordering, and retry semantics.

| Record | Minimum fields | Rule |
|---|---|---|
| Run state | Run ID, status, version, fence, deadline, current step, policy/schema versions | One authoritative current state; compare version and fence on every write |
| Domain event | Event ID, run ID, sequence, type, schema version, causation/correlation IDs, bounded payload/reference | Durable semantic fact; ordered within a run |
| Effect | Effect ID, run/step/tool version, input hash, state, fence, receipt/reference | Intent exists before the remote call; ambiguous outcomes reconcile |
| Delivery | connection/subscriber ID, event/cursor, delivery status | Loss or replay does not change authoritative run state |
| Telemetry | trace/span IDs, attempt/provider request IDs, safe attributes | Diagnostic, sampled, and never used as the source of truth |

Use separate identifiers for conversation, run, activation/attempt, step, tool call, effect, event, provider request, and trace. Reusing one GUID everywhere hides causality and makes deduplication unsafe.

~~~csharp
public sealed record RunRecord(
    string RunId, long Version, long Fence, RunStatus Status,
    string CurrentStep, DateTimeOffset Deadline);

public sealed record RunEvent(
    Guid EventId, string RunId, long Sequence, int SchemaVersion,
    string Type, Guid? CausationId, JsonElement Payload);

public sealed record EffectRecord(
    string EffectId, string RunId, string StepId, string ToolVersion,
    string InputHash, long Fence, EffectState State, string? ReceiptReference);
~~~

Token deltas, typing indicators, and transport disconnects are delivery facts, not proof of business success. Success requires an explicit fenced terminal-state transition after required effects and receipts are durable. The provider stream closing normally is not itself that transition.

### Transaction and settlement order

1. In one database transaction, compare run version/fence, write the state transition and semantic event, and write an outbox record when follow-up delivery is required.
2. Before an external write, persist/claim its effect intent with normalized input hash.
3. After the call, persist a receipt or <code>Unknown</code>; never synthesize failure from a missing acknowledgment.
4. Settle the broker message only after the durable transaction commits.
5. Deliver client events from the journal/outbox. Reconnect resumes from a cursor without rerunning the step.

See the application-owned [agent state and event contract](../../runtime/agent-state-and-event-contracts.md) for transition, replay, reconnect, and compatibility rules.

## In-process queue

A bounded <code>Channel&lt;T&gt;</code> is appropriate for short-lived work that may be lost on restart or can be reconstructed. Use <code>BoundedChannelFullMode.Wait</code> to propagate backpressure and complete the writer during shutdown.

Never enqueue a scoped service, <code>DbContext</code>, open stream, mutable request object, or delegate closing over a request scope. Enqueue an immutable envelope and resolve dependencies inside a new worker scope.

## Broker workers

A broker consumer owns a message only for the lease/lock interval. Processing longer than that requires renewal and still remains vulnerable to pauses, network partitions, or process loss.

Safe order:

1. receive and validate the envelope;
2. acquire a run lease or conditional state transition;
3. execute replay-safe work;
4. commit state/effect records;
5. settle the message;
6. release the lease.

If settlement acknowledgment is lost, redelivery is expected. Inbox/message IDs and state transition preconditions make this harmless.

Use dead-letter queues for poison work, not as unattended storage. Record a safe failure category, tool/schema version, attempts, first/last timestamps, and an operator replay path.

## Leases and fencing

A time-based lease alone does not prevent the old owner from writing after a pause. Issue a monotonically increasing fencing token when ownership changes. Every state/effect write checks the current token. This turns a late stale owner into a rejected write rather than split-brain progress.

## Durable runtime choices

| Option | Best fit | Critical constraint |
|---|---|---|
| Database + broker worker | Simple explicit state machines | You own leases, timers, migrations, and reconciliation |
| Durable Task / Durable Functions | Azure-oriented orchestration and timers | Orchestrator code must be deterministic |
| Agent Framework Durable extension | Durable agent sessions on Durable Task | Extension packages are prerelease; state/streaming limits apply |
| Temporal .NET | General durable workflows and activities | Workflow determinism, replay, versioning, worker operations |
| Dapr Workflow .NET | Dapr-based applications | Runtime dependency, deterministic workflow code, versioning |

In replay-based systems, model calls, HTTP, current time, random values, filesystem access, and tool effects belong in activities, not deterministic orchestrator/workflow code. Record only bounded, serializable state. Large prompts, documents, and tool outputs should live in object storage with immutable references and integrity metadata.

## Durable does not mean exactly once

Durable workflow engines replay decisions and retry activities. External activities can still run more than once. Use effect IDs and downstream idempotency even when the engine advertises durable execution.

Separate:

- workflow/run identity;
- activity/attempt identity;
- business effect identity;
- provider request identity.

## State evolution

Persist an envelope with state type, version, created time, tenant, run ID, and integrity metadata. Upcast old events or use explicit workflow versioning when code changes. Pin tool and prompt versions needed for replay; do not rely on whatever is current after a deployment.

## Failure patterns

- Assuming an in-memory channel will finish work after process restart.
- Completing a broker message before terminal state is committed.
- One <code>DbContext</code> shared across parallel message handlers.
- Lease renewal succeeds locally but a stale worker can still commit.
- Calling a model directly from deterministic workflow code.
- Storing multi-megabyte transcripts in workflow history.
- Replaying an activity with a non-idempotent tool effect.

## Review checklist

- [ ] Durability requirements are explicit for every work class.
- [ ] Queue envelopes are immutable, versioned, and size-bounded.
- [ ] State, event, effect, delivery, and telemetry records have distinct contracts and identifiers.
- [ ] State transition and message settlement ordering is documented.
- [ ] Leases use fencing or conditional writes.
- [ ] Durable workflow code is deterministic.
- [ ] Large content is referenced, not embedded in histories.
- [ ] External activities use effect idempotency and reconciliation.

## Primary sources

- [.NET queued background service](https://learn.microsoft.com/en-us/dotnet/core/extensions/queue-service)
- [Azure Service Bus locks and settlement](https://learn.microsoft.com/en-us/azure/service-bus-messaging/message-transfers-locks-settlement)
- [Azure Service Bus duplicate detection](https://learn.microsoft.com/en-us/azure/service-bus-messaging/duplicate-detection)
- [Durable Task for AI agents](https://learn.microsoft.com/en-us/azure/durable-task/sdks/durable-task-for-ai-agents)
- [Agent Framework durable agents](https://learn.microsoft.com/en-us/azure/durable-task/sdks/durable-agents-microsoft-agent-framework)
- [Temporal .NET SDK](https://github.com/temporalio/sdk-dotnet)
- [Dapr Workflow overview](https://docs.dapr.io/developing-applications/building-blocks/workflow/workflow-overview/)
- [Dapr .NET workflow versioning](https://docs.dapr.io/developing-applications/sdks/dotnet/dotnet-workflow/dotnet-workflow-versioning/)

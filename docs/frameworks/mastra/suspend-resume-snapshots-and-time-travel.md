# Suspend, Resume, Snapshots, and Time Travel

Mastra snapshots persist workflow execution state. They enable suspend/resume
and debugging, but they do not make arbitrary side effects exactly once.

## What a snapshot represents

A workflow snapshot is a serializable execution checkpoint that can include:

- workflow and run identity;
- input and workflow state;
- step statuses, payloads, outputs, and errors;
- serialized step graph and active paths;
- suspended and waiting paths;
- suspend and resume payloads and labels;
- request/tracing context needed by the engine;
- retry state, timestamps, and terminal result.

The exact internal structure is not an application contract. Use public workflow
run APIs and <code>createWorkflowStateReader</code> for status, step output,
suspended steps, and resume labels. GitHub issue
[#16044](https://github.com/mastra-ai/mastra/issues/16044) documented the earlier
public-reader gap and was closed by
[#16091](https://github.com/mastra-ai/mastra/pull/16091).

## Suspend lifecycle

~~~mermaid
stateDiagram-v2
    [*] --> Running
    Running --> Suspended: step calls suspend
    Running --> Waiting: sleep or sleepUntil
    Suspended --> Running: authorized resume
    Waiting --> Running: timer/event continuation
    Running --> Success
    Running --> Failed
    Running --> Tripwire
    Suspended --> Cancelled: product policy
~~~

Suspended and waiting are distinct states. A human decision usually suspends; a
known time delay waits.

## The re-execution rule

On resume, the suspended step's execute handler runs again with resume data and
the original suspend data. Code before the suspend point can therefore execute
again.

Unsafe:

~~~ts
await chargeCard(input);
const decision = await suspend({ amount: input.amount });
~~~

Safer:

~~~ts
const approveCharge = createStep({
  id: "approve-charge-v2",
  inputSchema: z.object({ proposalId: z.string(), amount: z.number() }),
  suspendSchema: z.object({ proposalId: z.string(), amount: z.number() }),
  resumeSchema: z.object({ actorId: z.string(), proposalHash: z.string() }),
  outputSchema: z.object({ receiptId: z.string() }),
  execute: async ({ inputData, resumeData, suspend }) => {
    if (!resumeData) {
      return suspend(inputData);
    }

    // Load the proposal and actor from trusted storage; do not trust a client
    // to restate the amount or target during resume.
    const proposal = await loadAndAuthorizeProposal(
      inputData.proposalId,
      resumeData.actorId,
      resumeData.proposalHash,
    );
    return chargeOnce({
      idempotencyKey: `charge:${proposal.id}`,
      proposal,
    });
  },
});
~~~

Even in the safer shape, <code>chargeOnce</code> must use a durable idempotency
key and persist a receipt. <code>suspendSchema</code> and
<code>resumeSchema</code> validate transport shape; they do not establish
authorization or freshness. Treat every resumed handler as replayable.

## Resume API design

A product resume endpoint should accept a stable product operation ID rather
than an arbitrary framework snapshot. The backend should:

1. authenticate the actor;
2. load the product pending-action record;
3. verify tenant, allowed transition, expiry, and exact proposal hash;
4. load the workflow run through public APIs;
5. confirm the expected step and resume label are suspended;
6. atomically claim the approval/resume operation;
7. call resume with bounded validated data;
8. reconcile and persist the resulting product status.

Concurrent resume attempts must not execute the effect twice. Stable
<code>@mastra/core@1.63.2</code> atomically transitions the suspension from
<code>suspended</code> to <code>running</code>; only one caller continues.
Losing direct SDK callers receive
<code>WORKFLOW_RESUME_ALREADY_CLAIMED</code>, while the generated HTTP surface
maps that conflict to status <code>409</code>. The claim prevents duplicate
downstream workflow advancement for that suspension; it does **not** prove an
external effect is exactly once after a post-effect crash. Keep the product
claim and effect idempotency layers.

## Snapshot storage

Use shared persistent storage for any run that may outlive a process. Before
adopting an adapter, test:

- snapshot write and load;
- nested and foreach suspended paths;
- concurrent resume;
- process loss during snapshot write;
- serialization limits;
- schema/definition changes;
- retention exclusions;
- backup and restore.

Keep snapshots small. Store artifact references, not files or full model
transcripts. Never place raw credentials in request context that can be
serialized.

## Recovery after restart

Built-in server execution can reload persisted workflow state, and generated
server behavior includes restart handling for active workflow runs. This is not
equivalent to a distributed durable engine:

- an effect may be in progress when the process dies;
- the last durable checkpoint may precede that effect;
- multiple replicas need coordination;
- stream events may not be replayable;
- the application still needs a stable run index and product state.

Recovery sequence:

1. stop new work until storage is ready;
2. list or resolve known product operations;
3. read workflow state through public APIs;
4. classify terminal, suspended, waiting, active, and ambiguous runs;
5. restart only an idempotent/reconcilable path;
6. surface operator action for ambiguous effects;
7. update product state and audit the recovery decision.

## Time travel

Mastra time travel reconstructs earlier step results and executes from a target
step. It requires storage and a compatible workflow definition.

Use it for:

- local debugging;
- reproducing a failure with immutable inputs;
- controlled recovery where every re-executed effect is idempotent;
- evaluation of a changed downstream step in an isolated environment.

Do not use it as a casual production “undo” button. It can repeat model calls,
spend money, send messages, or mutate external systems. It can also fail when a
step was removed or the definition changed.

Before production time travel:

- snapshot the product operation state;
- identify every step that may run again;
- verify effect receipts and compensation;
- pin model, prompt, tool, and workflow versions;
- require operator authorization;
- execute in a restricted recovery mode;
- compare and explicitly commit the reconciled result.

## Definition compatibility

Long suspensions make version skew normal. Persist:

- workflow definition version;
- schema version;
- package profile;
- model/prompt/tool versions;
- policy version used for the pending approval.

An old run must resume against compatible code or a tested migration. A current
deployment reading a snapshot successfully does not prove semantic
compatibility.

## Historical regression fixtures

Two closed reports are valuable tests:

- [#12029](https://github.com/mastra-ai/mastra/issues/12029) reported
  cross-run context pollution during concurrent foreach on a beta core release.
- [#15552](https://github.com/mastra-ai/mastra/issues/15552) reported lost
  suspend payloads among parallel foreach siblings around core 1.23.

Neither should be stated as a current defect. Preserve reproductions as
concurrency and snapshot-isolation fixtures for the pinned version.

## Operator view

Expose stable fields:

- product operation and run ID;
- tenant and actor;
- definition/profile version;
- current product status;
- suspended step and safe display payload;
- resume deadline and authorized actions;
- last durable update;
- effect receipts and ambiguity flag.

Do not expose raw snapshot JSON; it is version-sensitive and may contain
sensitive data.

## Checklist

- [ ] Snapshot data is small, serializable, and free of credentials.
- [ ] Code before suspend is safe to execute again.
- [ ] Resume reauthorizes and binds exact proposal data.
- [ ] Concurrent resume has framework and application idempotency.
- [ ] Recovery uses public state readers, not snapshot internals.
- [ ] Product run state survives framework schema changes.
- [ ] Definition/profile versions are persisted.
- [ ] Time travel is restricted to controlled, effect-aware recovery.
- [ ] Closed concurrency reports remain regression tests.

## Primary sources

- [Suspend and resume documentation source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/workflows)
- [Workflow snapshot reference source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/reference/workflows)
- [Workflow state reader feature report](https://github.com/mastra-ai/mastra/issues/16044)
- [Atomic resume claim implementation](https://github.com/mastra-ai/mastra/pull/21725)
- [Concurrent foreach context report](https://github.com/mastra-ai/mastra/issues/12029)
- [Parallel suspend-payload report](https://github.com/mastra-ai/mastra/issues/15552)
- [Canonical idempotency and side-effect guide](../../reliability/idempotency-and-side-effects.md)

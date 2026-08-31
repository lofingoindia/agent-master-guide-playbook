# Workflows, Control Flow, and State

Mastra workflows make control explicit. Use them when the sequence, branches,
validation, approval points, or recovery rules are known before the run starts.
Do not ask a model to rediscover business process on every request.

## Core model

A step has an ID, input schema, output schema, and execute function. A workflow
has an input schema, output schema, optional state schema, and a committed graph
of steps and control operators.

~~~ts
const validate = createStep({
  id: "validate-order",
  inputSchema: orderSchema,
  outputSchema: validatedOrderSchema,
  execute: async ({ inputData }) => validateOrder(inputData),
});

const workflow = createWorkflow({
  id: "order-review",
  inputSchema: orderSchema,
  outputSchema: decisionSchema,
  stateSchema,
})
  .then(validate)
  .then(review)
  .commit();
~~~

Stable schemas make step boundaries testable and snapshots evolvable. Keep step
outputs small; persist large artifacts separately.

## Control operators

| Operator | Use | Operational characteristic |
|---|---|---|
| <code>then</code> | Sequential dependency | Next step receives prior output |
| <code>parallel</code> | Independent fixed branches | Starts all branches; no built-in global concurrency limit |
| <code>branch</code> | Conditional path | Conditions must be deterministic and observable |
| <code>foreach</code> | Repeat a step for a collection | Concurrency defaults to one and can be configured |
| <code>map</code> | Reshape data between contracts | Keep pure and deterministic |
| loop operators | Repeat until condition | Application must enforce a maximum |
| nested workflow | Reusable multi-step unit | State and snapshot paths become nested |

Use a nested workflow inside <code>foreach</code> when each item requires a
multi-step pipeline. The fan-in waits for every item/branch before the next
stage. A single slow item therefore determines tail latency.

## Concurrency and backpressure

<code>parallel</code> is not a capacity controller. If it creates 500 branches,
the application can generate 500 simultaneous model or API calls. Apply bounds
before the workflow:

- cap input collection size;
- chunk large jobs;
- use <code>foreach</code> with explicit concurrency;
- add downstream per-provider limiters;
- reject or queue work when global capacity is full;
- propagate cancellation to every branch;
- define whether one branch failure cancels or allows siblings to finish.

Model-provider rate limits, database pools, and outbound API quotas are shared
across runs. Per-workflow concurrency alone is insufficient.

## Step input and workflow state

Step I/O carries data along the graph. Workflow state is shared mutable state
that persists through suspend/resume. Define a master state schema and expose
only the subset a step needs. Update it through the workflow state API rather
than mutating captured objects.

Use state for:

- small progress markers;
- accumulated identifiers;
- policy or schema version used by the run;
- a pointer to an external artifact;
- a compensation plan.

Avoid state for:

- open sockets or class instances;
- functions and closures;
- credentials;
- unbounded model transcripts;
- large binary or document content;
- data that cannot be serialized to JSON.

Nested workflows can see parent state. That makes reuse convenient but also
creates hidden coupling; document which keys are read and written.

## Result contract

Workflow completion is a discriminated result, not merely a returned object.
Current statuses include success, failed, suspended, tripwire, and paused, with
shared input, step, state, and result information.

Every caller should exhaustively handle statuses. A suspended run is not an
error; a failed run is not safe to restart without effect analysis; a transport
timeout says nothing conclusive about workflow state.

Product-facing run state should be stored in an application-owned record:

| Field | Purpose |
|---|---|
| Product operation ID | Stable user/business identity |
| Mastra workflow and run ID | Runtime lookup |
| Tenant and actor | Authorization/recovery |
| Product status | Stable UI/API contract |
| Last framework status | Diagnostic mapping |
| Version profile | Definition/package/model version |
| Last event/offset | Stream reconciliation |
| Effect receipts | Idempotency and compensation |

This avoids coupling the UI to internal snapshot shape.

## Retry placement

Workflow and step retry configuration can specify attempts and delay. Use it
only after classifying failures:

- retry transient read/model/network failures within a strict budget;
- do not retry invalid input or policy denial;
- retry effecting steps only with idempotency and outcome reconciliation;
- prefer provider-native idempotency keys;
- cap total elapsed time across attempts;
- record attempt number and prior outcome in traces.

Framework retries cannot know whether an external effect committed just before a
connection failed.

## Agent steps

An agent may run inside a workflow step. The workflow then controls when the
probabilistic loop occurs and validates its output before proceeding. This is
often safer than giving one agent tools for an entire business process.

Pattern:

1. deterministic step loads authorized records;
2. agent step classifies or drafts a recommendation;
3. schema/business step validates the recommendation;
4. approval step suspends if needed;
5. idempotent effect step commits;
6. final step writes a product result.

## Definition evolution

Persisted runs outlive code deployments. Renaming or removing a step, changing a
schema, or changing branch conditions may make old snapshots impossible or
unsafe to resume.

Record a workflow definition version at run creation. For every incompatible
change, choose:

- keep the old definition available until its runs finish;
- migrate snapshots through a documented, tested transform;
- cancel and compensate affected runs;
- block resume with a clear operator action.

Workflow-definition persistence was beta at the snapshot. Do not assume it
solves application-level version routing.

## Testing strategy

Test at three layers:

### Step tests

Use deterministic inputs, fake external clients, and schema edge cases. Verify
abort/timeout and idempotency.

### Graph tests

Verify branch selection, foreach ordering, parallel failure, state updates,
suspension, and terminal status.

### Recovery tests

Kill the process before and after every effect and snapshot boundary. Resume
concurrently from two callers. Deploy a changed definition with an old suspended
run. Confirm the product record reconciles correctly.

## Common mistakes

- Using a workflow for one pure function, adding persistence and operations with
  no benefit.
- Using an agent loop for a fixed approval-and-commit process.
- Assuming <code>parallel</code> includes rate limiting.
- Performing an effect before suspension without an idempotency guard.
- Capturing non-serializable clients in state.
- Exposing raw step/snapshot status as the long-term product API.
- Retrying every error uniformly.

## Checklist

- [ ] Control flow is deterministic where the business process is known.
- [ ] Step IDs and schemas are stable and versioned.
- [ ] Fan-out, loops, retries, and total time are bounded.
- [ ] Shared state is small, typed, and serializable.
- [ ] Callers exhaustively handle every terminal/nonterminal status.
- [ ] Product run state maps to, but does not expose, internal snapshots.
- [ ] Effecting steps are idempotent and reconcilable.
- [ ] Old snapshots have a definition-evolution plan.

## Primary sources

- [Workflow documentation source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/workflows)
- [Workflow reference source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/reference/workflows)
- [Workflow implementation](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/src/workflows)
- [Workflow changelog entries](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/CHANGELOG.md)
- [Canonical durable-execution guide](../../runtime/durable-execution.md)
- [Canonical idempotency and side-effect guide](../../reliability/idempotency-and-side-effects.md)

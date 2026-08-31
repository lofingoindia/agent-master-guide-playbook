# Durable Execution, Inngest, and Temporal

“Durable” covers several different Mastra mechanisms. They must not be treated
as interchangeable. Select an engine by the exact failure that must be survived.

## Four execution layers

| Layer | Maturity on 2026-08-31 | Persists | Important non-guarantee |
|---|---|---|---|
| Built-in workflows/snapshots | Stable core surface | Workflow state at framework snapshot boundaries | In-flight external effects are not exactly once |
| Durable-agent API | Beta | Agent loop state through a workflow plus run/event integration | Recovery may repeat LLM/tool work; cross-replica fencing depends on a lease-capable PubSub |
| Mastra workers | Beta | Work coordination through shared storage/PubSub | No built-in DLQ; API mid-step crash and scheduler ownership limitations |
| Inngest adapter | Stable 1.x package; durable-agent API beta | Step execution in external Inngest machinery | Application effects still need idempotency |
| Temporal adapter | Pre-1.0 <code>0.4.1</code> | Workflow history/activities through Temporal | Narrow build constraints; application activity semantics still apply |

Durable state and replayable UI events are separate. Add distributed
PubSub/cache when clients must reconnect across replicas or process loss.

## Built-in snapshots

Built-in workflow execution is appropriate when:

- suspension and later resume are the main persistence requirement;
- steps are short and idempotent;
- a process restart can safely redrive from the last checkpoint;
- the application owns recovery and run mapping;
- one service/storage topology is operationally preferable.

It is not a general distributed execution engine. A process can die after an
effect but before the next durable snapshot.

## Durable agents

The beta durable-agent layer wraps the agentic loop in workflow/event
infrastructure. Current factories express different execution postures:

- <code>createDurableAgent</code>: in-process development-oriented execution;
- <code>createEventedAgent</code>: fire-and-forget built-in workflow execution;
- <code>createInngestAgent</code>: external Inngest-backed production path.

Observation by run ID can replay cached events. Cleanup removes registry/cache
state and should not run while a suspended run must later resume.

### Crash recovery

If a run is left “running,” recovery can redrive from the last snapshot. This
can reissue a model call or tool call. Therefore:

- use idempotency keys and durable receipts;
- classify ambiguous effects before recovery;
- keep prompts/tools deterministic enough to reconcile;
- verify one recovery owner with a lease/fencing mechanism;
- trace the original and recovery attempts together.

Stable <code>1.63.2</code> recovery does acquire and renew an exclusive
per-agent/run lease. The guarantee is topology-dependent: it unwraps a
<code>LeaseProvider</code> from the configured PubSub; Redis Streams and Valkey
Streams implement that contract on the pinned snapshot. When the PubSub does
not implement it, Mastra falls back to an always-win no-op provider plus an
in-process claim, which does **not** fence another replica. Test two processes
against the exact backend and monitor lease acquisition/loss before allowing
automatic recovery on every replica.

This is a deliberate resolution of conflicting primary evidence: the official
durable-agent guide still gives the conservative statement that no distributed
lease is provided. Until the documentation and shipped capability converge,
trust only a passing topology test—not interface presence alone.

Recovery can be enabled during Mastra startup or invoked explicitly:

~~~ts
const mastra = new Mastra({
  // ...storage, agents, workflows
  recovery: { durableAgents: "auto" },
});

// Operational alternatives for targeted or externally scheduled recovery:
await mastra.recoverAllDurableAgents();
await durableAgent.recoverActiveRuns();
await durableAgent.recoverActiveRuns({ runId });
~~~

Do not enable automatic recovery on every replica unless the configured PubSub
provides a tested distributed lease, or an external leader makes only one
replica the recovery owner. Prefer explicit recovery for high-impact effects so
the operator can first reconcile downstream receipts.

### Cross-process option contract

Inspect and test what the selected artifact serializes. In stable
<code>@mastra/core@1.63.2</code>, JSON-safe state can survive, while executable
closures and process-local objects cannot:

| Option/state | Survives as durable behavior? | Consequence |
|---|---|---|
| Prompts/messages, identifiers, maximum steps, JSON-safe targets | Yes, subject to schema/version compatibility | Safe only if referenced tools/agents still mean the same thing |
| Function <code>stopWhen</code> | No closure after fresh-process recovery | Hard maximum steps remains the last bound |
| Function-form approval policy | No closure; persisted boolean fallback may remain | Reauthorize inside the effecting tool |
| <code>prepareStep</code> | No | Recovered step preparation can differ |
| Delegation callbacks/filter | No | Recovered supervisor delegation can lose per-call filtering/telemetry |
| Scorer instances and completion callback | No | Recompute completion outside the recovered loop |
| External abort signal | No | Restore cancellation from product state |

Do not describe these values as “serialized” merely because their descriptive
metadata appears in a snapshot. A durable acceptance test must terminate the
original process and recover in a clean process with no in-memory registry.

### Run indexing

The product must persist product operation ID to durable run ID. Open issue
[#17998](https://github.com/mastra-ai/mastra/issues/17998) documented the lack of
a thin storage-backed thread-to-active-run index for ordinary running
durable-agent runs at the snapshot. Do not scan raw workflow snapshots for UI
state.

## Inngest

<code>@mastra/inngest@1.8.8</code> is a stable package. The current
durable-agent wrapper remains beta. Mastra maps workflow steps to Inngest steps,
which provides external execution, memoization, retries, resume, dashboard, and
flow-control capabilities.

Current integration requirements and cautions:

- use the documented Inngest v4-compatible setup;
- development mode may disable signature verification; never enable that bypass
  in production;
- configure concurrency, rate limiting, throttling, debounce, priority, and cron
  from measured needs;
- keep execution options serializable—closure-shaped configuration can be lost
  across worker hops;
- protect the serve endpoint and verify signing keys;
- record Inngest event/function/run IDs alongside Mastra IDs;
- align Inngest retry policy with Mastra/tool retries to avoid multiplication.

Inngest memoization does not make an arbitrary external API call exactly once.
Use downstream idempotency.

## Temporal

<code>@mastra/temporal@0.4.1</code> is a pre-1.0 integration and should be
treated as emerging. Its build plugin rewrites Mastra workflow definitions so
steps execute as Temporal activities and the workflow graph executes as a
Temporal workflow.

Current integration constraints include:

- workflow IDs need build-time discoverable/static definitions;
- a long-lived Temporal worker is required, so this is not a simple
  request-scoped serverless path;
- activity <code>startToCloseTimeout</code> has a documented default around one
  minute unless configured;
- the plugin/build transform must be included in CI and deployment;
- both Mastra and Temporal SDK/server versions must be pinned;
- unsupported dynamic code patterns should fail a build-time conformance test.

Temporal workflow code must be deterministic, while effects belong in
activities. Follow Temporal's own retry, timeout, cancellation, versioning, and
idempotency guidance; a Mastra wrapper does not erase those rules.

## Engine decision

~~~mermaid
flowchart TD
    A[Long-running operation] --> B{Only pause/resume checkpoint needed?}
    B -->|Yes| C[Built-in workflow snapshots]
    B -->|No| D{Agent loop needs beta resumability?}
    D -->|Yes, accept beta| E[Durable agent]
    D -->|No| F{External event/function platform fits?}
    F -->|Inngest| G[Mastra Inngest adapter]
    F -->|Temporal operations already exist| H[Evaluate pre-1.0 Temporal adapter]
    F -->|Neither| I[Use engine directly or keep orchestration application-owned]
~~~

Prefer the simplest layer that satisfies real recovery needs. Adding both
Mastra workers and an external engine can create two competing schedulers and
retry systems.

## Guarantee matrix to verify

For each candidate, test:

| Guarantee | Test |
|---|---|
| Checkpoint | Kill before/after step result persistence |
| Effect repeat | Kill after downstream commit but before framework checkpoint |
| Timer | Restart every component while waiting |
| Signal/resume | Duplicate and unauthorized signals |
| Cancellation | Cancel parent with active model/tool/activity |
| Retry | Confirm maximum attempts and no nested amplification |
| Versioning | Resume an old run after a workflow deployment |
| Replay | Reconnect client after process/cache loss |
| Ownership | Start recovery from two replicas |
| Retention | Keep histories needed for the longest run |

Marketing terms such as “durable” or “exactly once” are not substitutes for this
profile.

## Workflow versioning

Long-running external histories may outlive many deploys. Persist:

- Mastra workflow and schema version;
- Mastra package profile;
- engine adapter, SDK, and server/cloud version;
- model, prompt, and tool version;
- activity/step retry and timeout policy.

Use engine-supported versioning and keep compatible code paths until old runs
complete. Never change the meaning of a stable step/activity ID silently.

## Observability

Correlate:

- product operation;
- Mastra workflow/durable run;
- Inngest function/run or Temporal workflow/run/activity;
- trace/span;
- tool call and effect receipt.

Mastra and engine dashboards show different portions of the lifecycle. The
product operation record is the stable join.

## Migration strategy

To move from built-in execution to an external engine:

1. freeze new starts briefly or route by definition version;
2. let existing built-in runs finish on their original executor;
3. start new versioned runs on the new engine;
4. keep product status mapping independent from engine state;
5. compare outputs, retries, cost, and timing in shadow/canary traffic;
6. retain rollback without moving active histories between engines.

Do not transform live snapshots into a different engine's internal history
unless both vendors explicitly support it.

## Checklist

- [ ] “Durability” is stated as exact tested guarantees.
- [ ] State persistence and event replay are designed separately.
- [ ] Every effect is idempotent and reconcilable.
- [ ] Durable-agent beta and worker beta are explicitly risk-accepted.
- [ ] Recovery has one tested owner; no-op versus distributed lease is known.
- [ ] Fresh-process recovery preserves or independently enforces every policy.
- [ ] Inngest signature verification is enabled in production.
- [ ] Temporal build, determinism, worker, and timeout constraints are tested.
- [ ] Engine, adapter, and framework versions are pinned together.
- [ ] Old runs stay on their original compatible executor.

## Primary sources

- [Durable-agent guide source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/harness/durable-agents.mdx)
- [Durable-agent reference source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/reference/agents/durable-agent.mdx)
- [Durable-agent recovery lease implementation](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/src/agent/durable/durable-agent.ts)
- [Inngest integration documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/integrations/deploy/inngest.mdx)
- [Inngest package source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/workflows/inngest)
- [Temporal integration documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/integrations/deploy/temporal.mdx)
- [Temporal package source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/workflows/temporal)
- [Temporal TypeScript durable execution documentation](https://docs.temporal.io/develop/typescript)
- [Active durable-run index request](https://github.com/mastra-ai/mastra/issues/17998)

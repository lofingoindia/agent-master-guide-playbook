# Supervisors, Subagents, and Routing

Mastra's current multi-agent direction is a supervisor agent with registered
subagents. The older agent network API is deprecated. Delegation is useful when
specialized agents measurably outperform one well-designed agent; otherwise it
adds more model calls, context boundaries, approval paths, and failure states.

## Current versus legacy surface

| Surface | Status on 2026-08-31 | Guidance |
|---|---|---|
| Supervisor through an agent's <code>agents</code> property | Current | Preferred for bounded model-selected delegation |
| Subagent calls and hooks | Current | Constrain context, tools, steps, and effects |
| Agent <code>.network()</code> | Deprecated | Migrate to a supervisor |
| Older <code>AgentNetwork</code> | Legacy migration source | Do not start new work |
| Deterministic workflow | Stable alternative | Prefer when routing/control is known |

Do not follow a historical migration from <code>AgentNetwork</code> to
<code>.network()</code> as the final state. The current migration target is the
supervisor pattern.

## How delegation works

Registering subagents exposes agent-prefixed delegation tools to the parent. The
supervisor model chooses a subagent using its key, description, and the
supervisor's instructions.

~~~mermaid
sequenceDiagram
    participant U as User
    participant S as Supervisor
    participant H as Delegation hook
    participant A as Specialist
    participant T as Specialist tools
    U->>S: Task
    S->>H: Proposed delegation and context
    H-->>S: Allow, modify, or reject
    S->>A: Bounded delegation prompt
    A->>T: Tool calls under specialist policy
    T-->>A: Results
    A-->>S: Delegation result
    S-->>U: Synthesized answer
~~~

The parent needs accurate descriptions and explicit instructions about when
**not** to delegate. A subagent should return a compact result with evidence,
not an unbounded transcript.

## Decision rule

Use a supervisor only when all are true:

- the task is genuinely open-ended;
- specialist prompts/tools improve a measured failure mode;
- one agent with conditional toolsets is insufficient;
- additional latency and cost fit the product;
- nested authorization and approval can be tested;
- failure can be reconciled at each delegation boundary.

Use a workflow instead when ordering, branching, review, or compensation is
business-defined. Use ordinary application routing when a deterministic intent
classifier or product state already determines the destination.

## Constrain each subagent

For every specialist, define:

- one purpose and non-goals;
- maximum steps, deadline, and model budget;
- minimum tools and data scopes;
- independent authorization policy;
- memory scope and retention;
- schema or bounded output;
- whether it may delegate further;
- error mapping and fallback;
- trace linkage to the parent.

Disable recursive delegation unless a tested use case requires it. Depth
multiplies cost and makes approvals and cancellation harder to explain.

## Context sharing

The parent can forward context, and <code>messageFilter</code> controls which
messages reach the subagent. Defaulting to “all messages” is convenient but can
leak unrelated tenant data, secrets, or adversarial instructions.

The current default hook error strategy is operationally convenient but unsafe
for a security filter: if <code>messageFilter</code> throws, delegation proceeds
with the **unfiltered** message set. Likewise, a failing start hook falls back
to the original delegation and a failing completion hook falls back to the
original result. Set <code>hookErrorStrategy: "throw"</code> whenever a hook
enforces authorization, redaction, budget, or policy:

~~~ts
const response = await supervisor.stream(task, {
  requestContext,
  delegation: {
    hookErrorStrategy: "throw",
    messageFilter: ({ messages }) =>
      messages.filter(messageIsAuthorizedForSpecialist).slice(-8),
    onDelegationStart: ({ agent }) => {
      assertDelegationEdgeAllowed("support-supervisor", agent.name);
    },
  },
});
~~~

With the default <code>"warn"</code> strategy, hook errors are also placed in
the request context under <code>__mastra_delegationHookErrors</code>. That key is
useful telemetry, not a security control: an application must not depend on an
internal-looking context key as its only enforcement path.

Build a delegation envelope:

~~~json
{
  "task": "classify the support case",
  "authorizedRecordIds": ["case_42"],
  "constraints": ["do not contact the customer"],
  "outputContract": "classification-v3",
  "correlationId": "op_01J..."
}
~~~

Resolve records through authorized tools. Do not paste broad database records
into the prompt.

## Memory isolation

Each subagent delegation uses a fresh conversation thread and a deterministic
resource derived from the parent resource and subagent name. Memory can be
inherited if the subagent does not define its own. This prevents accidental
transcript mixing at one level, but resource-scoped memory can persist across
delegations.

Review whether a specialist should:

- be stateless;
- receive a filtered slice of parent messages;
- use thread-scoped memory for one delegation;
- use resource-scoped memory across tasks;
- write any working/observational memory.

For security-critical specialists, stateless input plus authorized record
references is the safest baseline.

## Delegation hooks

Hooks can observe, modify, reject, and measure delegations. Use them to:

- enforce allowed parent-to-child edges;
- reduce or redact forwarded context;
- attach trace and product operation IDs;
- cap delegation step budgets;
- reject cyclic or repeated delegation;
- record why routing occurred;
- score completion at iteration boundaries.

Hooks are an additional control, not a replacement for tool authorization.

By default, the parent model receives the subagent's text result, while nested
tool results remain visible to the application stream rather than being added
to the parent model context. Enabling
<code>includeSubAgentToolResultsInModelContext</code> can help synthesis, but it
also enlarges the prompt-injection, secret-exposure, and token-cost boundary.
Enable it only with redacted, size-bounded tool results.

## Durable supervisor seam

The stable <code>@mastra/core@1.63.2</code> durable-agent artifact persists
serializable run state, not arbitrary JavaScript closures. A fresh process can
therefore recover the run without recreating every per-call behavior:

| Per-call option | Fresh-process recovery risk | Production control |
|---|---|---|
| Function <code>stopWhen</code> | Closure is unavailable; the loop can fall back to the persisted maximum-step bound | Keep a hard <code>maxSteps</code>; express critical termination in durable state or static code |
| Function <code>requireToolApproval</code> | Function is unavailable; only the persisted boolean shadow can survive | Enforce approval in the tool/effect boundary, not only in a callback |
| <code>prepareStep</code> | Closure is unavailable | Rebind through a versioned application wrapper and test process replacement |
| Delegation callbacks and <code>messageFilter</code> | Closures are unavailable; recovered work can use default delegation behavior | Put mandatory scope checks in every subagent tool and avoid sensitive recovery until conformance-tested |
| Completion scorers/callbacks | Instances and callbacks are not reconstructed | Persist business completion separately and run evaluation out of band |
| External <code>AbortSignal</code> | Process-local signal is gone | Reconstruct cancellation from durable product state |

This is a feature-level beta contract, not evidence that snapshots are broken.
Make “kill after delegation, start a fresh worker, recover, and verify the same
authorization/termination result” a required deployment test. If a policy
cannot survive that test, do not run that supervisor through durable recovery.

## Cancellation and failures

Cancellation should propagate from the parent into the subagent and its tools.
Test this with slow tools; a provider or external API may ignore cancellation.

Classify:

- routing failure: no suitable specialist or repeated wrong delegation;
- specialist failure: model/tool error inside one delegation;
- synthesis failure: valid specialist result mishandled by parent;
- ambiguous effect: specialist tool may have committed before failure;
- approval suspension: nested run is waiting for a product decision.

Do not automatically delegate the same effecting task to another specialist
after an ambiguous outcome.

## Nested approval

Approval must show the innermost effect, not only an “agent-billing” delegation.
Issue [#20934](https://github.com/mastra-ai/mastra/issues/20934) reported this
information loss in streaming delegation on core 1.57.0 and was closed by a
subsequent fix. Preserve a test that asserts exact nested tool name, arguments,
identity, decline behavior, and durable resume routing on the selected version.

## Quality and cost evaluation

Compare the supervisor against simpler baselines:

| Metric | Why it matters |
|---|---|
| Task success by case | Averages can hide specialist regressions |
| Wrong-delegation rate | Measures router quality |
| Delegation count/depth | Detects loops and excess coordination |
| Tool/effect error rate | Finds authority and schema failures |
| p50/p95 completion time | Multiple serial model calls raise tail latency |
| Model/tool cost | Supervisor and synthesis calls add spend |
| Context bytes per edge | Reveals leakage and prompt growth |
| Human escalation rate | Captures uncertain or unsafe routing |

A multi-agent design should earn its complexity with a statistically and
operationally meaningful gain.

## Background subagent work

Background subagent tasks depend on background-task storage, eventing, and
durable observation. Treat the path as beta-adjacent even when the base agent API
is stable. Require product-owned run mapping, bounded task lifetime, worker
health, idempotent effects, and recovery tests before using it for user-visible
jobs.

## Checklist

- [ ] New code uses supervisors, not deprecated networks.
- [ ] A single-agent and deterministic-routing baseline was measured.
- [ ] Delegation graph, depth, steps, time, and cost are bounded.
- [ ] Context is allowlisted through a delegation envelope.
- [ ] Policy hooks use <code>hookErrorStrategy: "throw"</code> and fail-closed tests.
- [ ] Subagent memory scope is explicit.
- [ ] Every specialist independently authorizes tools.
- [ ] Cancellation and ambiguous effects are tested.
- [ ] Nested approvals display the true effect.
- [ ] Durable recovery reproduces or independently enforces function-valued policy.
- [ ] Quality gain justifies added latency and operations.

## Primary sources

- [Subagent documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/subagents.mdx)
- [Agent network documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/agents/networks.mdx)
- [Network-to-supervisor migration source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/reference/migrations/network-to-supervisor.mdx)
- [Agent implementation](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/src/agent)
- [Durable-agent reference source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/reference/agents/durable-agent.mdx)
- [Nested approval report](https://github.com/mastra-ai/mastra/issues/20934)

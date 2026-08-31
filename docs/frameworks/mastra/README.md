# Mastra Production Engineering Guide

Mastra is a TypeScript framework for model calls, agents, tools, memory, workflows,
servers, observability, and deployment. Those capabilities share a registry and a
stream protocol, but they do **not** share one durability, security, or maturity
contract. Production systems should treat the framework as a set of composable
planes rather than as one all-inclusive agent runtime.

> **Research snapshot:** 2026-08-31
>
> **Source snapshot:** Mastra repository commit
> [8c88706](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad),
> committed 2026-08-30
>
> **Production baseline used here:** stable npm releases, led by
> <code>@mastra/core@1.63.2</code>; the repository was on
> <code>1.63.3-alpha.0</code> when researched
>
> **Runtime floor:** Node.js <code>>=22.13.0</code> for the current packages

This guide is a complement to the repository's existing
[Mastra overview](../mastra.md). It does not replace that file.

Interpret Mastra snapshots, stream events, and durable-engine adapters through
the repository's application-owned
[agent state and event contract](../../runtime/agent-state-and-event-contracts.md).
Framework storage is one implementation boundary, not the product's complete
identity, ordering, terminal-state, or effect ledger.

## Guide map

| Guide | Production question |
|---|---|
| [Runtime architecture and boundaries](runtime-architecture-and-boundaries.md) | Which component owns registry, request context, persistence, transport, and deployment? |
| [Agents, models, instructions, and processors](agents-models-instructions-and-processors.md) | How is an agent turn bounded, guarded, and made provider-portable? |
| [Tools, MCP, and human approval](tools-mcp-and-human-approval.md) | Where are effects authorized, validated, approved, and made idempotent? |
| [Memory, context, and storage](memory-context-and-storage.md) | What belongs in history, working memory, recall, observations, or a system of record? |
| [Workflows, control flow, and state](workflows-control-flow-and-state.md) | When should deterministic orchestration replace an open-ended loop? |
| [Suspend, resume, snapshots, and time travel](suspend-resume-snapshots-and-time-travel.md) | What is actually persisted, and what can safely run again? |
| [Streaming, events, PubSub, and clients](streaming-events-pubsub-and-clients.md) | How do clients reconnect and reconcile without assuming an uninterrupted stream? |
| [Supervisors, subagents, and routing](supervisors-subagents-and-routing.md) | When is delegation justified, and how is context and authority constrained? |
| [Observability, evaluation, testing, and debugging](observability-evaluation-testing-and-debugging.md) | How is a probabilistic system turned into an evidence-backed release process? |
| [Reliability, retries, concurrency, and shutdown](reliability-retries-concurrency-and-shutdown.md) | Which failures can be retried, drained, recovered, or reconciled? |
| [Security, identity, and multitenancy](security-identity-and-multitenancy.md) | Which identity and tenant checks must exist outside prompts and memory scopes? |
| [Server, deployment, scaling, performance, and cost](server-deployment-scaling-performance-and-cost.md) | What changes across self-hosting, adapters, serverless, and Mastra Platform? |
| [Durable execution, Inngest, and Temporal](durable-execution-inngest-and-temporal.md) | Which engine guarantees survive process loss, and at what integration maturity? |
| [Packages, integrations, versioning, migrations, limitations, and alternatives](packages-versioning-migrations-limitations-and-alternatives.md) | What should be pinned, integration-tested, avoided, migrated, or replaced? |

The supporting [research packet](../../research/packets/mastra-deep-dive.md)
records the evidence boundary, package versions, issue sampling, and claims that
were deliberately downgraded.

## Choose a reading path

| If you are... | Read first | Then prove |
|---|---|---|
| Evaluating Mastra | [Runtime boundaries](runtime-architecture-and-boundaries.md) and [packages/alternatives](packages-versioning-migrations-limitations-and-alternatives.md) | The framework removes enough integration work to justify its runtime surface |
| Shipping one agent | [Agents](agents-models-instructions-and-processors.md), [tools](tools-mcp-and-human-approval.md), and [security](security-identity-and-multitenancy.md) | Bounded turns, denied cross-tenant access, and idempotent effects |
| Adding deterministic orchestration | [Workflows](workflows-control-flow-and-state.md) and [snapshots](suspend-resume-snapshots-and-time-travel.md) | Old-run compatibility, concurrent resume, and kill-point recovery |
| Adding resumable/background execution | [Durable execution](durable-execution-inngest-and-temporal.md), [streaming](streaming-events-pubsub-and-clients.md), and [reliability](reliability-retries-concurrency-and-shutdown.md) | Which state, event, callback, and effect guarantees survive a fresh process |
| Adding subagents | [Supervisors](supervisors-subagents-and-routing.md) | Measured quality gain, bounded delegation, fail-closed context filtering, and cross-process policy behavior |
| Deploying | [Server and deployment](server-deployment-scaling-performance-and-cost.md) and [observability](observability-evaluation-testing-and-debugging.md) | Real adapter auth, proxy streaming, shutdown, storage, telemetry, and rollback behavior |

The fastest safe route is usually the first three rows, not every feature at
once. Durable agents, workers, and external engines solve different failures;
adding all of them creates overlapping recovery and retry owners.

## The minimum production contract

Do not ship a Mastra service until the application owns all of these decisions:

1. **Identity and tenancy:** authenticate at the transport boundary; authorize
   every data read and effect; derive resource and tenant identifiers
   server-side.
2. **Bounded execution:** set turn, step, time, tool-concurrency, retry, and
   fan-out limits.
3. **Effect safety:** validate tool input, attach an idempotency key, store a
   receipt, and reconcile ambiguous outcomes.
4. **Persistence:** use shared production storage; keep snapshots small and
   serializable; configure and schedule retention.
5. **Recovery:** distinguish reconnect, replay, resume, retry, restart, and
   compensating action. They are not synonyms.
6. **Evidence:** trace inputs and outputs according to data policy, redact
   sensitive data, run deterministic tests, and gate releases on representative
   datasets.
7. **Version profile:** pin the full package and engine combination, not only
   <code>@mastra/core</code>, and rerun conformance tests on every upgrade.

## Architecture at a glance

~~~mermaid
flowchart LR
    Client[Client or product backend] --> Edge[Authentication, authorization, quotas]
    Edge --> Server[Mastra generated server or adapter]
    Server --> Registry[Mastra registry]
    Registry --> Agent[Agent loop]
    Registry --> Workflow[Workflow control]
    Agent --> Tools[Tools, MCP, subagents]
    Agent --> Memory[Memory]
    Workflow --> Snapshot[Workflow snapshots]
    Registry --> Storage[(Storage domains)]
    Memory --> Storage
    Snapshot --> Storage
    Agent --> Obs[Observability and evals]
    Workflow --> Obs
    Workflow -. optional .-> Engine[Inngest or Temporal]
    Server -. distributed events .-> Bus[Redis Streams or GCP Pub/Sub]
~~~

The solid lines describe common runtime integration. The dashed lines are
optional execution and distribution choices with separate operational
contracts.

## Maturity legend

This guide uses feature maturity, not package major version, as the deciding
signal:

| Label | Meaning in this guide |
|---|---|
| **Stable** | Documented production surface in a stable package; still requires integration tests |
| **Beta** | Public but explicitly beta, or operationally incomplete enough to require a guarded rollout |
| **Experimental** | Semantics or API may change; isolate behind an application boundary |
| **Pre-1.0 adapter** | Published package whose version communicates a higher compatibility risk |
| **Deprecated** | Migration target exists; do not start new production work on it |

Examples as of the snapshot:

- Core agents, tools, workflows, memory, server, MCP, observability, and the
  primary SQL stores are stable package surfaces.
- Durable agents, workers, workflow-definition persistence, and response
  caching are beta.
- Resource-scoped observational memory and working-memory state signals are
  experimental.
- <code>@mastra/temporal@0.4.1</code>, <code>@mastra/redis-streams@0.4.0</code>,
  <code>@mastra/valkey-streams@0.5.0</code>, and several server adapters are
  pre-1.0 even when useful.
- Agent <code>.network()</code> is deprecated; supervisor agents are the current
  direction.

## Recommended adoption path

Start with one registered agent, typed tools, Postgres-backed storage, explicit
thread/resource identifiers, and traces. Add a deterministic workflow when the
control path is known. Add a supervisor only when specialization measurably
improves quality. Add an external durable engine only after identifying a real
process-loss or scheduling requirement that built-in snapshots do not satisfy.

That sequence minimizes the number of distributed state machines that must be
debugged at once.

## Refresh triggers

Re-research this area when any of the following changes:

- the stable <code>@mastra/core</code> minor version;
- the Node.js engine floor or AI SDK compatibility matrix;
- durable agents, workers, response cache, or workflow definitions leave beta;
- the Temporal or stream PubSub adapters reach 1.0;
- Mastra changes generated-server shutdown, route, or authorization behavior;
- a new security advisory or package-integrity incident is published;
- <code>.network()</code> is removed rather than merely deprecated.

## Primary source entry points

- [Mastra documentation](https://mastra.ai/docs)
- [Mastra repository](https://github.com/mastra-ai/mastra)
- [Pinned repository snapshot](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad)
- [Repository releases](https://github.com/mastra-ai/mastra/releases)
- [Core package changelog](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/CHANGELOG.md)
- [Responsible disclosure contact](https://github.com/mastra-ai/mastra#security)

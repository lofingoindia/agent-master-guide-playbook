# Runtime Architecture and Boundaries

Mastra's main architectural convenience is a shared TypeScript registry. Its
main production risk is assuming that everything registered there shares the
same lifecycle and guarantee. It does not.

## Component boundary map

| Component | Owns | Does not automatically own |
|---|---|---|
| <code>Mastra</code> registry | Discovery and shared configuration for agents, workflows, storage, logger, observability, server options | Authentication policy, effect idempotency, lease correctness of the chosen PubSub |
| Agent | Model loop, instructions, tool selection, processors, agent stream | Deterministic business control, exactly-once effects |
| Workflow | Typed step graph, control flow, state, snapshots, suspend/resume | Crash-proof in-progress effects unless backed by a durable engine |
| Memory | Conversation history and model context layers | Authoritative permissions, balances, orders, or tenant membership |
| Storage | Domain persistence through adapters | Uniform capability across every adapter; automatic retention scheduling |
| Server | Generated HTTP routes, streaming transport, middleware integration, lifecycle hooks | Correct policy for every route or non-Hono adapter |
| PubSub | Runtime event distribution and optional replay | Workflow snapshots or business-result persistence |
| Inngest/Temporal | Alternative external workflow execution | Application authorization and idempotent activities |
| Mastra Platform | Managed build/deploy/runtime products | Portability or identical behavior to self-hosted/serverless targets |

## The registry pattern

Register production agents and workflows in a single <code>Mastra</code>
instance. Retrieval through the registry attaches shared storage, logging, and
observability configuration. A directly imported agent may still generate or
stream, but it can silently miss those shared services.

~~~ts
const mastra = new Mastra({
  storage,
  observability,
  agents: { supportAgent },
  workflows: { refundWorkflow },
});

const agent = mastra.getAgentById("support-agent");
~~~

Treat identifiers as API and data keys:

- use explicit, stable IDs for agents, tools, steps, and workflows;
- never derive a persistent ID from display text;
- include an ID migration plan before renaming a step whose snapshots may still
  exist;
- reject duplicate IDs during startup tests.

## Request-scoped context is dependency injection

Mastra's request context carries request-specific values into dynamic
instructions, models, tools, memory, and workflows. Typical values include a
verified user ID, tenant ID, locale, feature flag, correlation ID, or a
short-lived service handle.

Request context is **not persistent memory** and is **not authorization by
itself**. The server must construct security-sensitive values after verifying
credentials. A client-provided tenant key copied into request context is still
client-provided input.

Use a typed schema at the application edge and follow these rules:

1. Reject missing or malformed identity values before invoking Mastra.
2. Place only small, serializable values in workflow context if a run may
   suspend.
3. Do not place credentials in instructions, messages, snapshots, or trace
   attributes.
4. Revalidate authorization immediately before an irreversible effect.
5. Reconstruct trusted actor context on durable resume; do not assume an old
   credential or permission survives indefinitely.

## Choose the narrowest control primitive

~~~mermaid
flowchart TD
    A[Task arrives] --> B{Known control path?}
    B -->|Yes| W[Workflow]
    B -->|No| C{One agent has the skills?}
    C -->|Yes| S[Single agent]
    C -->|No| D{Delegation improves measured quality?}
    D -->|Yes| U[Supervisor with bounded subagents]
    D -->|No| R[Refine one agent or tool interface]
    W --> E{Must survive process loss mid-step?}
    E -->|No| M[Built-in execution and snapshots]
    E -->|Yes| X[Evaluate Inngest or Temporal]
~~~

An agent is appropriate for open-ended reasoning and tool selection. A workflow
is appropriate when the business already knows the sequence, branches, approval
points, or compensation rules. A supervisor is not a more advanced workflow; it
is another probabilistic loop that delegates to other loops.

## Model boundary

Mastra's model surface bridges its own model router and AI SDK provider
implementations. Provider capability remains the limiting factor. Before
changing a model or AI SDK generation, test:

- tool calls and parallel tool calls;
- structured output with and without tools;
- streaming chunk order and terminal events;
- usage and finish-reason reporting;
- timeout and abort propagation;
- fallback behavior;
- provider-specific reasoning or cached-token fields.

Do not let a dependency range independently upgrade the provider, AI SDK,
Mastra core, and server adapter in production. Record the whole known-good
profile in the lockfile and deployment metadata.

## Server boundary

The standard Mastra build produces a Hono-based server with generated agent,
workflow, memory, observability, and other routes. Mastra also publishes
adapters for frameworks such as Hono, Express, Fastify, Koa, NestJS, and Next.

Middleware semantics are adapter-specific. In particular, Hono middleware
configured on the Mastra server does not magically execute inside a non-Hono
host. Install authentication, authorization, rate limiting, body limits, and
request logging in the native host framework, then run an end-to-end denied
request test against every deployed route family.

Keep three surfaces separate:

- **open-source runtime:** packages executed in infrastructure you control;
- **generated or embedded server:** the HTTP transport around that runtime;
- **Mastra Platform:** managed Server, Studio, observability, databases, and
  environment/deployment products.

Authentication and middleware inheritance differ by host:

| Host path | What current stable evidence says | Required production test |
|---|---|---|
| Generated <code>mastra build</code>/<code>mastra start</code> server | Hono server config, generated route metadata, auth provider, and built-in signal lifecycle are in one artifact | Deny every protected route family; verify deliberately public routes; send SIGTERM during a stream |
| <code>@mastra/hono</code>, Next, or TanStack Start adapters | Hono-based adapters register <code>server.middleware</code> in current compatible versions; public framework routes intentionally skip user middleware | Pin adapter with core/server and test both protected and <code>requiresAuth: false</code> routes |
| Express, Fastify, or Koa adapters | They cannot execute Hono middleware handlers; current adapters warn rather than silently pretending to inherit them | Install native middleware or the adapter auth helper before route registration; verify ordering end to end |
| Custom/raw framework routes | Generated-route defaults do not prove the raw route is protected | Attach native or <code>createAuthMiddleware</code> policy explicitly and include it in the route inventory |

The Hono inheritance fix shipped in <code>@mastra/hono@1.7.2</code>, below the
<code>1.7.5</code> baseline in this guide. It is still an upgrade regression
test, not permission to assume every adapter behaves like Hono.

## Storage is a set of domains

Mastra storage includes domains for memory, workflows, observability, scores,
datasets, experiments, background tasks, schedules, thread state, and other
features. Adapter coverage differs. A composite store can route domains to
different backends, such as Postgres for workflow and memory records and
ClickHouse for high-volume telemetry.

Before choosing an adapter, create a capability table for the features actually
used:

| Capability | Required verification |
|---|---|
| Workflow snapshot | persist, load, concurrent resume claim, retention |
| Memory | thread ownership, semantic vector filters, pagination |
| Observability | trace, log, metric, score query volume |
| Background tasks | claim/recovery semantics and cleanup |
| Schedules | single scheduler ownership and missed-run policy |
| Retention | adapter implementation, batching, resume, cancellation |

An adapter constructor accepting the configuration is not evidence that every
domain or operation is supported.

## Process topology

The default development topology is one process with an in-memory event emitter.
Production topologies can add:

- multiple HTTP replicas;
- shared storage;
- a distributed PubSub backend;
- orchestration, schedule, or background-task workers;
- external Inngest or Temporal workers;
- a separate observability store.

Each addition creates a new failure boundary. Multiple replicas do not by
themselves provide leader election, a durable run index, replayable stream
history, or exactly-once processing.

## Startup contract

At startup:

1. validate required secrets and endpoint URLs without logging them;
2. verify the Node and package compatibility profile;
3. connect and health-check required storage domains;
4. register all IDs and reject duplicates;
5. initialize observability and sensitive-data filtering;
6. initialize PubSub/workers only after storage is ready;
7. expose readiness only after recovery and migrations are safe;
8. keep liveness independent from slow downstream model providers.

## Architecture review checklist

- [ ] Every runtime component has a named owner and failure policy.
- [ ] Request context is typed and created server-side.
- [ ] Agent, workflow, and supervisor choices are justified by control needs.
- [ ] The model/provider combination has a conformance suite.
- [ ] Adapter middleware is tested at the actual HTTP boundary.
- [ ] Storage-domain capability is verified, not assumed.
- [ ] Process topology documents replay, recovery, and leader ownership.
- [ ] Deployment metadata contains exact package and engine versions.

## Primary sources

- [Mastra class reference source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/src/mastra)
- [Request context documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/server/request-context.mdx)
- [Server documentation source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/server)
- [Storage documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/storage.mdx)
- [Core package manifest](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/package.json)
- [Hono adapter middleware fix](https://github.com/mastra-ai/mastra/pull/22161)
- [Canonical execution-boundaries guide](../../runtime/execution-boundaries.md)

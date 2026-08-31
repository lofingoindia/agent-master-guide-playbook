# Packages, Integrations, Versioning, Migrations, Limitations, and Alternatives

Mastra is a fast-moving multi-package ecosystem. Production compatibility is a
profile across core, server, storage, memory, MCP, observability, adapters,
model providers, and any external engine—not a single SemVer number.

## Stable package snapshot

These npm <code>latest</code> values were resolved on 2026-08-31:

| Package | Stable | Role / maturity note |
|---|---:|---|
| <code>mastra</code> | 1.27.2 | CLI |
| <code>@mastra/core</code> | 1.63.2 | Core agents/workflows/runtime; individual beta features remain |
| <code>@mastra/server</code> | 1.63.2 | Server primitives |
| <code>@mastra/deployer</code> | 1.63.2 | Deployment core |
| <code>@mastra/client-js</code> | 1.42.4 | JavaScript client |
| <code>@mastra/memory</code> | 1.28.1 | Memory processors/layers |
| <code>@mastra/mcp</code> | 1.17.2 | MCP client/server |
| <code>@mastra/rag</code> | 2.6.0 | Retrieval and document-processing utilities |
| <code>@mastra/observability</code> | 1.17.4 | Telemetry |
| <code>@mastra/evals</code> | 1.9.0 | Scorers/evals |
| <code>@mastra/pg</code> | 1.22.2 | PostgreSQL storage |
| <code>@mastra/libsql</code> | 1.22.2 | LibSQL storage |
| <code>@mastra/mongodb</code> | 1.18.4 | MongoDB storage |
| <code>@mastra/redis</code> | 1.4.2 | Redis-related storage/cache surfaces |
| <code>@mastra/clickhouse</code> | 1.16.0 | High-volume observability/storage |
| <code>@mastra/duckdb</code> | 1.6.3 | Local analytical/observability storage |
| <code>@mastra/inngest</code> | 1.8.8 | External workflow engine adapter; durable-agent wrapper beta |
| <code>@mastra/temporal</code> | 0.4.1 | Pre-1.0 external engine adapter |
| <code>@mastra/redis-streams</code> | 0.4.0 | Pre-1.0 distributed PubSub |
| <code>@mastra/valkey-streams</code> | 0.5.0 | Pre-1.0 distributed PubSub |
| <code>@mastra/google-cloud-pubsub</code> | 1.1.2 | Distributed PubSub |
| <code>@mastra/hono</code> | 1.7.5 | Hono adapter |
| <code>@mastra/express</code> | 1.5.7 | Express adapter |
| <code>@mastra/next</code> | 0.2.21 | Pre-1.0 Next adapter |

The pinned repository main branch declared
<code>@mastra/core@1.63.3-alpha.0</code> and corresponding alpha versions. Docs
and main-branch source can therefore be ahead of npm stable by a patch or expose
features that are not in the production baseline.

All inspected current packages specify Node.js <code>>=22.13.0</code>.

## Version policy

For production:

1. pin exact versions through a committed lockfile;
2. store the resolved integrity hashes/artifacts;
3. upgrade related Mastra packages together;
4. use stable tags, not alpha, unless an isolated experiment explicitly accepts
   the risk;
5. pin model provider and AI SDK versions too;
6. pin storage/PubSub/engine server compatibility;
7. record the profile on traces and workflow runs;
8. canary and retain rollback compatibility with stored state.

Mastra v1 guidance recommends keeping packages current and aligned. The release
cadence is rapid; exact pins plus frequent deliberate upgrades are safer than
broad ranges plus rare surprises.

## Supply-chain release gate

The [June 2026 official incident](https://github.com/mastra-ai/mastra/issues/18061)
involved malicious npm versions published through a compromised maintainer
token. Even exact SemVer ranges and “latest” were unsafe during the incident.

Add:

- release quarantine/dwell time;
- publisher/provenance verification;
- dependency and lifecycle-script diff;
- private registry allowlisting;
- clean-room rebuild after security notice;
- credential rotation when a suspect install ran.

Version freshness and artifact trust are independent.

## Feature maturity pins

Stable package major versions contain mixed-maturity features:

| Feature | Snapshot posture |
|---|---|
| Core agents/tools/workflows/memory/server | Stable base surfaces |
| Durable agents | Beta |
| Workers | Beta with documented operational gaps |
| Response caching processor | Beta; unsuitable for side-effect replay |
| Workflow-definition persistence/editor domains | Beta |
| Resource-scoped observational memory | Experimental |
| Working-memory state signals | Experimental |
| OpenAI-compatible Responses server surface | Experimental/version-sensitive |
| Temporal adapter | Pre-1.0 |
| Redis/Valkey stream adapters | Pre-1.0 |
| Agent <code>.network()</code> | Deprecated |

Pin feature flags and rollout state alongside package versions.

### Upgrade-sensitive behavior floors

These floors matter when a deployment is older than this guide's baseline:

| Behavior/fix | First fixed/released version evidenced here | Upgrade implication |
|---|---:|---|
| Atomic single-winner workflow resume and HTTP conflict mapping | Core/server stable <code>1.61.0</code> | Keep a simultaneous-resume regression test on every storage adapter |
| Prevent <code>mastra__authToken</code> persistence in snapshots, scores, and durable-agent input | Core stable <code>1.61.0</code> | Upgrade, inspect retained data/backups under policy, and rotate any exposed bearer token |
| Register <code>server.middleware</code> in the Hono adapter | <code>@mastra/hono@1.7.2</code> | Verify protected and intentionally public routes in the deployed adapter |

The current baseline—core/server <code>1.63.2</code> and Hono
<code>1.7.5</code>—is above these floors. A fix version does not prove the first
affected version, so avoid inventing a vulnerable lower bound.

## Migrations

### Upgrade to Mastra v1

Mastra published v1 upgrade material and a codemod. Run the codemod on a clean
branch, inspect every edit, align all Mastra packages, and rerun stream,
snapshot, storage, model, and deployment tests. Do not mix v0/v1 examples.

### Networks to supervisors

Migrate <code>.network()</code> and older <code>AgentNetwork</code> code to a
parent agent with registered subagents. Re-evaluate context filtering, memory,
approval, and termination; this is a semantic migration, not a method rename.

Migration acceptance should compare behavior, not only compilation:

| Legacy behavior to capture | Supervisor acceptance test |
|---|---|
| Routing and final synthesis | Same task slices reach the intended specialist and preserve result contract |
| Loop termination | Explicit <code>maxSteps</code>, stop reason, and no delegation cycle |
| Context forwarding | <code>messageFilter</code> allowlist plus fail-closed hook-error strategy |
| Memory | Fresh delegation thread and intended resource-scoped persistence |
| Nested tools/approval | True inner tool/arguments visible; decline and resume reach the right run |
| Streaming | Client reducer handles delegation and nested tool events without double rendering |
| Failure/cancellation | Parent abort reaches child/tools; ambiguous effects are reconciled, never blindly rerouted |

### AI SDK generations

Older AI SDK v4 paths use legacy streaming, while v5+ uses the current stream
surface. Mastra's dependency graph carries compatibility across multiple
provider generations, but application code and provider features still differ.
Run structured-output, tool-call, usage, timeout, and streaming conformance
tests.

### Deployment commands

Current Mastra Platform environments use <code>mastra deploy</code>. Older
Server/Studio deploy commands entered deprecation transition after the July 2026
environment launch. Follow current Platform documentation rather than old
“Mastra Cloud” pages.

## Known boundary limitations

These are architectural or snapshot-specific constraints, not a claim that the
framework is defective:

- model/tool effects are at-least-once unless the application adds idempotency;
- built-in snapshots checkpoint workflow state but cannot prove an in-flight
  effect did not commit;
- durable-agent recovery fencing depends on a PubSub that implements the lease
  provider contract; unsupported backends fall back to process-local/no-op
  behavior across replicas;
- a general active durable-run index remained an open request;
- workers lacked a built-in DLQ and required one scheduler;
- default EventEmitter PubSub is process-local;
- persistent run state does not imply replayable stream tokens;
- storage adapter domain support varies;
- retention must be scheduled by the application;
- direct SDK invocation still requires an application authorization boundary;
- server adapters do not all inherit Hono middleware behavior;
- approval across nested delegation must be regression-tested;
- time travel can repeat effects and costs.

## Integration packages are separate contracts

Mastra's integration catalog spans model providers, embeddings, vector stores,
databases, tools, voice, auth, observability exporters, deployment targets, and
workflow engines. A catalog entry does not inherit core's version, maintenance,
or guarantee.

For every integration:

1. pin the Mastra package and underlying vendor SDK;
2. verify its Node/runtime and peer-dependency range;
3. test authentication, timeout, abort, pagination, rate limits, and error
   mapping;
4. validate tenant filters and data residency;
5. check whether streaming, retries, and idempotency are implemented by Mastra,
   the vendor SDK, or neither;
6. record vendor API version and required permissions;
7. compare the wrapper with using the vendor SDK in a narrow local tool.

Prefer a small application-owned tool wrapper when an integration exposes much
more authority than required or lags a vendor API. Prefer the official
integration when it removes meaningful compatibility work and its behavior is
covered by the same conformance suite as the rest of the runtime.

## Choosing an alternative

| Need | Consider | Why it may be simpler/better |
|---|---|---|
| Only TypeScript model streaming and tools | Vercel AI SDK or provider SDK | Smaller runtime surface; application owns orchestration |
| Small agent loop with handoffs/tracing | OpenAI Agents SDK | Focused agent abstraction when provider fit is acceptable |
| Explicit graph and checkpoint/control semantics | LangGraph | Graph-first state-machine model and broad durable patterns |
| High-assurance long-running business process | Temporal directly | Mature durable execution and operational model without wrapper constraints |
| Event-driven functions, schedules, flow control | Inngest directly | Native platform semantics and fewer translation layers |
| Fixed process with no model-selected routing | Application code/queue | Most deterministic, testable, and portable |
| Integrated TypeScript agent platform | Mastra | Strong breadth when the registry/server/memory/workflow integration is valuable |

“Use Mastra” and “use Temporal/Inngest” are not mutually exclusive, but each
adapter adds a translation and version seam. Prefer the fewest layers that
satisfy the requirement.

## Upgrade runbook

1. read changelogs from the current version through the target;
2. verify no security incident/advisory affects selected artifacts;
3. update the full package profile in one reviewable change;
4. regenerate/build and inspect route/OpenAPI changes;
5. run typecheck and Markdown/code sample validation as applicable;
6. run model/tool/stream and storage conformance;
7. resume old snapshots against the candidate;
8. run concurrent resume, tenant isolation, and kill-point tests;
9. compare eval, latency, cost, and telemetry;
10. canary with rollback that keeps old workflow code/data compatible.

## Checklist

- [ ] Full exact package/engine/provider profile is recorded.
- [ ] Stable versus alpha documentation/source differences are understood.
- [ ] Feature maturity is evaluated separately from package major version.
- [ ] Artifact provenance and lifecycle scripts are gated.
- [ ] Deprecated networks and deployment flows have migration plans.
- [ ] Old snapshots and stream clients pass upgrade tests.
- [ ] Alternative frameworks were compared against the actual requirement.
- [ ] Rollback preserves data and definition compatibility.

## Primary sources

- [Mastra npm organization](https://www.npmjs.com/org/mastra)
- [Pinned monorepo package manifests](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad)
- [Integration catalog source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/integrations)
- [Core changelog](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/CHANGELOG.md)
- [Atomic resume claim implementation](https://github.com/mastra-ai/mastra/pull/21725)
- [Auth-token persistence fix](https://github.com/mastra-ai/mastra/pull/21996)
- [Hono middleware registration fix](https://github.com/mastra-ai/mastra/pull/22161)
- [Mastra releases](https://github.com/mastra-ai/mastra/releases)
- [Upgrade-to-v1 migration source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/reference/migrations/upgrade-to-v1)
- [Network-to-supervisor migration source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/reference/migrations/network-to-supervisor.mdx)
- [Official supply-chain incident report](https://github.com/mastra-ai/mastra/issues/18061)
- [Vercel AI SDK documentation](https://ai-sdk.dev/docs)
- [OpenAI Agents SDK documentation](https://openai.github.io/openai-agents-js/)
- [LangGraph documentation](https://docs.langchain.com/oss/javascript/langgraph/overview)
- [Temporal TypeScript documentation](https://docs.temporal.io/develop/typescript)
- [Inngest documentation](https://www.inngest.com/docs)

# Server, Deployment, Scaling, Performance, and Cost

Mastra can run as a generated standalone server, inside another web framework,
on third-party serverless platforms, or on Mastra Platform. Choose the topology
from lifecycle and durability needs, not from the smallest deployment command.

## Deployment surfaces

| Surface | What it is | Key constraint |
|---|---|---|
| <code>mastra build</code> output | Self-contained generated server in <code>.mastra/output</code> | You own runtime, network, scaling, storage, and upgrades |
| Mastra generated server | Hono-based routes around registered resources | Generated route surface and middleware must be reviewed |
| Server adapter | Mastra inside Hono, Express, Fastify, Koa, NestJS, Next, and others | Native host lifecycle/middleware semantics apply |
| Cloud deployer package | Build adaptation for Vercel, Netlify, Cloudflare, etc. | Serverless limits affect streams, filesystem, sockets, and background work |
| Mastra Platform Server | Managed API deployment for a Mastra project | Product-specific sleep, environment, database, quota, and pricing behavior |
| Mastra Platform Studio/Observability | Managed operator and telemetry products | Separate access policy and data-retention decisions |
| Workers | Beta split-process orchestration/schedule/background execution | Requires distributed PubSub/shared storage; documented gaps remain |

The CLI may mention Node, Bun, or Deno deployment possibilities, but the current
published package manifests require Node.js <code>>=22.13.0</code>. Treat Node
22.13+ as the verified baseline unless a separate runtime matrix proves the
entire package profile on another runtime.

## Build and start contract

<code>mastra build</code> produces a deployable output directory. Run the
generated artifact, not the TypeScript source tree, in production. The start
path loads environment configuration and handles lifecycle signals.

Build requirements:

- deterministic lockfile install;
- exact Node version;
- no production secrets in the image or source map;
- package-integrity and lifecycle-script scanning;
- compile/build with the same flags as production;
- artifact SBOM and version metadata;
- smoke test against generated OpenAPI and stream routes.

The June 2026 npm compromise makes dependency artifact verification a required
build control, not an optional best practice.

## Self-hosted topology

A conservative production shape:

~~~mermaid
flowchart LR
    C[Clients] --> G[Gateway: auth, quotas, TLS]
    G --> A1[Mastra API replica]
    G --> A2[Mastra API replica]
    A1 --> DB[(Shared Postgres)]
    A2 --> DB
    A1 --> P[(Distributed PubSub/cache)]
    A2 --> P
    W[Optional beta workers] --> DB
    W --> P
    A1 --> O[Telemetry backend]
    A2 --> O
    W --> O
~~~

Do not add replicas until shared state and coordination are ready. In-memory
storage, EventEmitter PubSub, active-run maps, and replay caches are
process-local.

## Embedded adapters

Use an adapter when Mastra must share an existing service's routing,
authentication, dependency injection, or deployment. Verify:

- native middleware order;
- body and streaming response support;
- abort propagation;
- serverless or edge restrictions;
- OpenAPI route prefix/collisions;
- startup and shutdown hooks;
- error translation and redaction;
- WebSocket/SSE proxy behavior;
- whether MCP and Studio routes are supported.

Several adapters were pre-1.0 on the snapshot, including
<code>@mastra/next@0.2.21</code>. Pin and test the adapter independently from
core.

Generated and embedded lifecycle contracts are different. The generated
<code>mastra start</code> entry owns its HTTP handle, handles SIGINT/SIGTERM,
drains active requests and streams for <code>server.drainTimeout</code> (five
seconds by default), then calls <code>mastra.shutdown()</code>. An embedded
adapter is host-owned: the host must stop admission, drain HTTP, shut Mastra
down, flush exporters, and close application clients in order. Setting
<code>handleShutdownSignals: false</code> in the generated entry does not give a
config-module handler the HTTP server handle, so it cannot reproduce the drain;
use an adapter when full lifecycle control is required.

## Serverless

Serverless is suitable for bounded stateless turns when external storage holds
durable data. Risks increase for:

- long streams and platform response limits;
- in-process background tasks;
- timers/schedules;
- local files or LibSQL;
- connection-heavy MCP clients;
- observability exporters that need flushing;
- in-memory PubSub/replay;
- workflow work continuing after the response.

Use remote storage, platform-compatible database pooling, explicit exporter
flush, and an external engine/queue for work that must continue after
termination. Test the actual provider's maximum duration, streaming, cold start,
and concurrency behavior.

## Mastra Platform

Mastra Platform is a managed product layer around the open-source framework.
Current Platform environments can provide separate production, staging, and
preview URLs, variables, deploy history, regions, and optionally databases.
The environment-based <code>mastra deploy</code> flow requires core 1.44 or
newer. Older <code>mastra server deploy</code> and <code>mastra studio
deploy</code> flows entered a time-bounded deprecation transition in July 2026.

Platform-specific operational points from current documentation:

- deployment passes through queue, upload, build, deploy, and running states;
- a build running longer than the documented limit can fail;
- the filesystem is ephemeral, so use managed or remote persistent storage;
- a Server may sleep after a period without outbound activity unless the
  applicable persistent-server capability is enabled;
- database heartbeats, exporters, sockets, or streams can affect sleep behavior;
- first deployment/environment handling can upload local environment values, so
  sanitize files and manage secrets deliberately;
- each deploy replaces the running process and Platform currently documents no
  configurable or guaranteed termination grace; a long stream can be
  interrupted even when local <code>drainTimeout</code> is higher;
- private-registry <code>NPM_TOKEN</code> is redacted from streamed source-build
  logs and excluded from the runtime image layer, but is also injected into the
  running service environment; use a read-only token and treat it as a runtime
  secret;
- plan quotas and retention are product contracts, not framework guarantees.

Old “Mastra Cloud” pages indexed by search are legacy material. Use current
Mastra Platform environment/deploy documentation and record the CLI/core version
that generated the deployment. For hosted observability, new code uses
<code>MastraPlatformExporter</code>; <code>CloudExporter</code> remains only as
a deprecated compatibility name.

## Capacity model

Request-per-second alone is misleading. Size for:

~~~text
inflight turns
  = arrival rate
  × average turn duration
  × retry/delegation amplification
~~~

Then include:

- model steps per turn;
- parallel/foreach and tool-call concurrency;
- average and p95 stream lifetime;
- model/provider connection quotas;
- database connections and writes per step;
- event and trace volume;
- memory embedding/observation calls;
- live eval and judge calls.

Long model/tool latency can exhaust sockets and memory at modest RPS.

## Scaling controls

Apply controls in this order:

1. per-user and per-tenant quotas;
2. global admission limit;
3. maximum prompt/body/artifact size;
4. agent step/time/tool-concurrency bounds;
5. workflow fan-out/loop bounds;
6. provider-specific semaphores/rate limits;
7. database and PubSub pool limits;
8. queue or reject with an explicit retry-after;
9. autoscale from inflight work and latency, not CPU alone.

Backpressure is preferable to letting every layer independently retry a saturated
provider.

## Performance work

Measure before optimizing. High-impact levers:

- smaller, relevant context and tool results;
- fewer agent steps/delegations;
- parallelize only independent safe reads;
- choose a faster validated model for simple stages;
- move known control to workflows;
- batch embeddings/storage writes when semantics permit;
- stream for user-perceived latency, while keeping total deadlines;
- separate telemetry storage under high volume.

The beta response cache may reduce repeated pure generation, but it must not be
used to imply side effects were executed.

## Cost model

Total cost includes:

- primary and fallback model input/output/reasoning tokens;
- supervisor and subagent calls;
- tool/API charges;
- embeddings and vector storage;
- observational-memory observer/reflection calls;
- live/model-judge eval calls;
- snapshots, messages, traces, logs, and event retention;
- PubSub/cache/database infrastructure;
- Inngest/Temporal or managed Platform consumption;
- engineering/on-call complexity.

Record cost at the product-operation level. A cheap model call that causes three
retries and two delegations is not cheap.

## SLOs and alerts

Define:

- valid terminal success rate;
- p50/p95 time to first token and completion;
- stuck active, waiting, and suspended run age;
- approval wait age/expiry;
- ambiguous-effect count;
- retry and fallback amplification;
- worker/PubSub lag;
- replay/reconnect success;
- storage/exporter errors;
- cost per successful operation.

Alert on user-impacting symptoms and recovery backlog, not every provider error.

## Deployment verification

- [ ] Exact Node/package/adapter profile is recorded.
- [ ] Generated routes and denied requests are tested.
- [ ] Shared state replaces all process-local production dependencies.
- [ ] Serverless limits or Platform lifecycle behavior are tested in situ.
- [ ] Shutdown completes within platform grace.
- [ ] Streams reconnect through the real proxy/load balancer.
- [ ] Capacity tests include model latency and fan-out.
- [ ] Cost includes memory, eval, telemetry, and durable engines.
- [ ] Rollback preserves compatibility with stored snapshots.

## Primary sources

- [Deployment documentation source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/deployment)
- [Server documentation source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/server)
- [Server adapter packages](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/server-adapters)
- [Deployer packages](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/deployers)
- [Mastra Platform environments announcement](https://mastra.ai/blog/introducing-environments-for-mastra-platform)
- [Pinned Platform deploy documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/mastra-platform/deploy.mdx)
- [Pinned generated-server lifecycle documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/deployment/mastra-server.mdx)
- [Current Mastra Server product page](https://mastra.ai/ai-agent-deployment)
- [Canonical deployment and incident-response guide](../../operations/deployment-release-and-incident-response.md)
- [Canonical scaling, capacity, and SLO guide](../../operations/scaling-capacity-and-slos.md)

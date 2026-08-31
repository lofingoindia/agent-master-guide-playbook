# Security, Identity, and Multitenancy

Mastra supplies authentication hooks, memory ownership mapping, processors,
approval, and fine-grained authorization integration. A secure application must
still establish identity at the server boundary and enforce policy again at
every data and effect boundary.

## Trust model

Treat as untrusted:

- request bodies, headers not set by a trusted gateway, and client request
  context;
- model output and tool-call arguments;
- memory, retrieved documents, and previous assistant messages;
- MCP descriptions, resources, apps, and results;
- tool and external API output;
- resume payloads and run IDs;
- trace/search filters provided by clients.

Trusted state should be constructed from authenticated identity plus
authoritative systems. Instructions and working memory are never policy
enforcement.

## Authentication versus authorization

Authentication answers who the caller is. Authorization answers whether that
actor may perform this operation on this resource now.

Mastra server auth providers can validate requests and attach user context.
<code>SimpleAuth</code> is explicitly unsuitable as production security: its
in-memory tokens lack the lifecycle and cryptographic guarantees expected of a
real identity provider. Use a supported JWT/OIDC integration or a reviewed
custom provider.

Native framework middleware is required for non-Hono server adapters. Test
denied requests against the actual deployed adapter, not only a unit-level auth
method.

Auth inheritance is adapter-specific on this snapshot:

| Deployment path | Security expectation |
|---|---|
| Generated server | Configure Mastra auth and verify every generated route family |
| Hono, Next, and TanStack Start adapters | Compatible Hono adapters register <code>server.middleware</code>; routes explicitly marked <code>requiresAuth: false</code> intentionally skip auth middleware |
| Express, Fastify, and Koa adapters | Hono middleware cannot execute; install native middleware or the adapter's auth middleware factory |
| Raw/custom routes | Declare and test authentication/authorization explicitly |

The Hono registration seam was corrected in <code>@mastra/hono@1.7.2</code>;
the pinned package baseline is <code>1.7.5</code>. Keep one denied-request test
per adapter and one public-health-route test so upgrades cannot silently invert
the boundary.

## Resource mapping

Auth providers can map an authenticated user to the
<code>MASTRA_RESOURCE_ID</code> used for thread filtering and ownership.
Generated server behavior can filter thread listings, validate thread access,
force new-thread ownership, and validate message access.

This is valuable defense, but resource mapping is not a complete tenant model:

- tools may query arbitrary application databases;
- workflows and datasets have separate resources;
- observability can contain multiple tenants;
- resource-scoped caches and vectors need their own filters;
- tenant membership can change;
- one user can belong to multiple organizations.

Use an explicit tenant ID plus actor ID and authorize every downstream query.

## Fine-grained authorization

Mastra's FGA surface can gate server routes and operations across agents,
workflows, tools, MCP, memory, and stored resources. The EE implementation is
imported from <code>@mastra/core/auth/ee</code>; production use requires the
applicable Mastra Enterprise license.

Important boundary: direct SDK workflow calls such as starting, resuming, or
restarting a run are not necessarily protected by an HTTP route check. Wrap
programmatic invocation in an application authorization service and pass a
verified actor/request context.

FGA configuration details:

- require an actor for least privilege where supported;
- build “trusted system actor” objects only server-side;
- do not treat actor permission claims as self-verifying;
- resolve permission state from the authorization source of truth;
- propagate actor context deliberately through custom execute paths;
- restore/revalidate actor on durable resume rather than relying on a stale
  snapshot.

Mastra separates server/API FGA and Studio FGA. Restrict Studio to internal
operators even when public API users are legitimate.

## Route surface

The generated server exposes multiple route families for registered features.
Inventory the generated OpenAPI document and test each route. Hono middleware
can block groups, but routes marked public may skip authentication middleware,
and the framework does not provide a universal “only these generated routes”
allowlist.

Use:

- an external gateway or load balancer allowlist;
- deny-by-default application middleware;
- explicit CORS and body-size limits;
- per-user/tenant quotas;
- CSRF protection for cookie-authenticated state changes;
- internal network restrictions for workers/Studio;
- route-level authorization tests in CI.

## Tool and approval security

Tool code must:

1. receive a verified actor and tenant;
2. load the target through a tenant-filtered query;
3. verify current permission;
4. validate exact business invariants;
5. require approval for high-risk effects;
6. bind approval to normalized arguments, identity, expiry, and policy;
7. execute idempotently;
8. store an audit receipt.

Reauthorize after a long approval wait. Never approve an outer delegation while
hiding the inner charge/delete/send arguments.

## Prompt injection and data exfiltration

Processors can detect, redact, and abort suspicious input/output, but no
classifier is a complete prompt-injection boundary. Reduce blast radius:

- use minimum tool allowlists;
- separate data from instructions in prompts;
- tag provenance;
- bound and sanitize tool/MCP/retrieval results;
- keep secrets outside model context;
- make effecting tools require deterministic checks;
- restrict outbound hosts and data volume;
- require human approval for material irreversible effects.

Tool descriptions themselves can be poisoned when supplied by an MCP server.

## Storage and encryption

Mastra adapters persist messages, snapshots, traces, scores, and other domains.
The application or infrastructure owns:

- encryption in transit and at rest;
- database credentials and rotation;
- row/namespace tenant isolation;
- backup access and deletion;
- retention and legal holds;
- regional/data-residency constraints;
- audit-log integrity;
- secret scanning.

Snapshots and traces can contain request context and tool payloads. Redact
before persistence, not only at UI rendering.

Core <code>1.61.0</code> fixed a security-relevant persistence path in which
<code>mastra__authToken</code> could be stored in workflow snapshots, score
rows, and durable-agent workflow input. The pinned core <code>1.63.2</code>
contains the fix, but an organization that ran an affected earlier deployment
should inspect persisted snapshots/scores, apply its deletion and backup policy,
and rotate exposed bearer credentials. Do not infer the first affected version
from the fix entry alone. The durable design rule is stronger: persist actor and
tenant identifiers, then obtain fresh authorization on resume—never serialize a
bearer token in request context.

## MCP controls

For stdio MCP:

- disable inherited default environment for high-risk servers;
- pass only required variables;
- use a restricted OS account/container;
- restrict filesystem and network access.

For HTTP MCP:

- authenticate and authorize the server;
- restrict initial and redirect hosts;
- prevent custom fetch from bypassing redirect validation;
- use per-user or narrowly scoped credentials;
- time-limit and rate-limit calls;
- disconnect request-scoped clients.

## Supply-chain integrity

Mastra disclosed a
[June 16, 2026 npm account compromise](https://github.com/mastra-ai/mastra/issues/18061):
an attacker published malicious versions across the <code>@mastra</code>
namespace with a credential-exfiltrating postinstall. The versions were
unpublished or deprecated and replacement releases were issued. This event
changes the minimum dependency policy:

- pin exact package versions with a committed lockfile;
- use a registry/proxy that can quarantine new releases;
- verify provenance, integrity hashes, publisher, and release workflow;
- delay automatic adoption of fresh packages;
- block lifecycle scripts in build stages where practical;
- scan dependency diffs and install scripts;
- rebuild from a known-clean lockfile after an incident;
- rotate credentials if a compromised package ever executed;
- record the exact installed artifact, not only the requested version.

If a listed malicious artifact was installed, containment is not “upgrade and
continue.” Isolate the build/host, revoke every credential reachable by the
install process, rebuild from a clean base and trusted lockfile, invalidate
caches that may contain the tarball, and review egress/audit evidence from the
installation window.

A previous moderate advisory,
[GHSA-xh92-rqrq-227v](https://github.com/mastra-ai/mastra/security/advisories/GHSA-xh92-rqrq-227v),
affected <code>@mastra/mcp-docs-server</code> through 0.13.8 and was patched in
0.17.0. The advisory metadata says <code>&lt;=0.13.8</code>, while its narrative
contains an inconsistent <code>0.13.18</code> string; use <code>0.17.0</code> or
later rather than attempting to interpret an intermediate safe range. The flaw
allowed directory listing/path traversal disclosure and could be reached by
prompt injection, demonstrating why development MCP servers also need least
privilege.

Mastra requests responsible disclosure through
[security@mastra.ai](mailto:security@mastra.ai).

## Tenant-isolation tests

Run concurrently:

- same thread/run names in two tenants;
- guessed thread, run, dataset, trace, and artifact IDs;
- semantic recall with nearly identical content;
- shared response cache;
- resource-scoped observational/working memory;
- delegated subagents;
- workflow suspend and resume by a different tenant;
- Studio and worker routes;
- deletion while background work is active.

Assertions must cover absence from results, errors, logs, traces, streams, and
timing-sensitive side channels appropriate to the threat model.

## Security checklist

- [ ] Real authentication runs on every deployed route surface.
- [ ] Tenant and actor are derived server-side.
- [ ] Authorization is repeated in data and tool code.
- [ ] Direct SDK workflow calls use an application guard.
- [ ] Trusted/system actors cannot be client-created.
- [ ] Approval binds exact effect and expires.
- [ ] Model-visible context contains no raw credentials.
- [ ] Storage, vectors, caches, telemetry, and artifacts isolate tenants.
- [ ] MCP environments, hosts, redirects, and credentials are restricted.
- [ ] Exact package artifacts and lifecycle scripts are controlled.
- [ ] June 2026 supply-chain response is part of the incident playbook.

## Primary sources

- [Authentication documentation source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/auth)
- [Authorization implementation source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/src/auth)
- [Fine-grained authorization announcement](https://mastra.ai/blog/introducing-fine-grained-authorization)
- [Mastra licensing map](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/LICENSE.md)
- [Official June 2026 supply-chain incident report](https://github.com/mastra-ai/mastra/issues/18061)
- [MCP docs-server security advisory](https://github.com/mastra-ai/mastra/security/advisories/GHSA-xh92-rqrq-227v)
- [Auth-token persistence fix](https://github.com/mastra-ai/mastra/pull/21996)
- [Hono middleware registration fix](https://github.com/mastra-ai/mastra/pull/22161)
- [Repository security contact](https://github.com/mastra-ai/mastra#security)
- [Canonical execution-boundary guide](../../runtime/execution-boundaries.md)

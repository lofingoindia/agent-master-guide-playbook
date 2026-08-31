# Mastra Deep-Dive Research Packet

> **Research date:** 2026-08-31
>
> **Research status:** Complete for the scoped production guide
>
> **Repository snapshot:** Mastra commit
> [8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad),
> committed 2026-08-30
>
> **Stable baseline:** <code>@mastra/core@1.63.2</code> and compatible npm
> <code>latest</code> packages resolved on the research date
>
> **Repository channel:** main was on <code>1.63.3-alpha.0</code>
>
> **Guide output:** [Mastra production engineering guide](../../frameworks/mastra/README.md)

## Research question

What is the most accurate production model of Mastra's agents, tools, MCP,
memory, workflows, streaming, supervisors, storage, server, observability,
deployment products, workers, durable agents, Inngest, and Temporal integrations
as of 2026-08-31—and which guarantees, maturity levels, and application-owned
controls must remain separate?

## Scope and method

Research prioritized current primary evidence:

1. the official Mastra repository, shallow-cloned and pinned to the commit above;
2. current English documentation source in the repository;
3. package manifests and per-package changelogs;
4. npm registry distribution tags and engine declarations;
5. current deployment, authentication, security, and migration material;
6. implementation source where documentation did not define an operational
   boundary;
7. bounded GitHub issues only when they exposed a production seam that should
   become a regression test;
8. official Temporal and Inngest documentation for engine-native constraints.

PASS-2 additionally packed the exact stable
<code>@mastra/core@1.63.2</code> npm artifact and inspected its shipped
declarations/runtime bundles for durable-agent recovery, delegation, approval,
timeout, and resume behavior. This separated “present on pinned main” from
“present in stable npm” and exposed process-local closure boundaries that a
documentation-only review would miss. Exact stable
<code>@mastra/redis-streams@0.4.0</code> and
<code>@mastra/valkey-streams@0.5.0</code> artifacts were also checked for the
lease-provider contract.

The result is a synthesis, not a site-by-site summary. Marketing statements were
downgraded when they did not define a testable guarantee. Closed issues are
treated as historical fixtures, not current defects.

## Repository evidence boundary

The local research clone resolved:

~~~text
commit: 8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad
date:   2026-08-30T18:22:46Z
title:  chore: regenerate providers and docs [skip ci]
~~~

Main-branch manifests were in an alpha prerelease cycle. Therefore:

- source/docs could describe changes not yet in stable 1.63.2;
- the guides use stable npm versions as the production baseline;
- alpha-only or explicitly unstable features are labeled;
- source links are commit-pinned so future readers can reproduce the evidence.

## Evidence contradictions resolved

| Evidence tension | Resolution used in the guides |
|---|---|
| The official durable-agent guide says Mastra does not yet provide a distributed recovery lease, while stable core 1.63.2 contains acquire/renew/loss/release logic and stable Redis/Valkey Streams implement the required lease interface | State the backend-dependent behavior: lease-capable PubSub can fence replicas; unsupported PubSub falls back to process-local plus always-win no-op behavior. Require a two-process conformance test and retain the official warning as the conservative posture. |
| Pinned main source/docs were at <code>1.63.3-alpha.0</code>, while production baseline was stable <code>1.63.2</code> | Confirm critical behavior in the packed stable artifact; label evidence that exists only on main/alpha. |
| Platform docs say <code>NPM_TOKEN</code> stays out of the runtime image layer but also say it is injected into the running service environment | Both are true: it is image-layer-safe but still an application-readable runtime secret. |
| The MCP advisory metadata says affected through <code>0.13.8</code>, while narrative text contains <code>0.13.18</code> | Do not infer an intermediate safe version; use the stated patched floor <code>0.17.0</code> or later. |

## Package snapshot

Resolved from npm distribution tags on 2026-08-31:

| Package | Latest | Alpha where observed | Node |
|---|---:|---:|---:|
| <code>mastra</code> | 1.27.2 | — | >=22.13.0 |
| <code>@mastra/core</code> | 1.63.2 | 1.63.3-alpha.0 | >=22.13.0 |
| <code>@mastra/server</code> | 1.63.2 | 1.63.3-alpha.0 | >=22.13.0 |
| <code>@mastra/deployer</code> | 1.63.2 | 1.63.3-alpha.0 | >=22.13.0 |
| <code>@mastra/deployer-cloud</code> | 1.63.2 | 1.63.3-alpha.0 | >=22.13.0 |
| <code>@mastra/deployer-vercel</code> | 1.2.22 | 1.2.23-alpha.0 | >=22.13.0 |
| <code>@mastra/deployer-netlify</code> | 1.2.22 | 1.2.23-alpha.0 | >=22.13.0 |
| <code>@mastra/deployer-cloudflare</code> | 1.2.22 | 1.2.23-alpha.0 | >=22.13.0 |
| <code>@mastra/client-js</code> | 1.42.4 | 1.42.5-alpha.0 | >=22.13.0 |
| <code>@mastra/memory</code> | 1.28.1 | — | >=22.13.0 |
| <code>@mastra/mcp</code> | 1.17.2 | — | >=22.13.0 |
| <code>@mastra/rag</code> | 2.6.0 | — | >=22.13.0 |
| <code>@mastra/observability</code> | 1.17.4 | — | >=22.13.0 |
| <code>@mastra/evals</code> | 1.9.0 | — | >=22.13.0 |
| <code>@mastra/auth</code> | 1.1.2 | — | >=22.13.0 |
| <code>@mastra/pg</code> | 1.22.2 | — | >=22.13.0 |
| <code>@mastra/libsql</code> | 1.22.2 | — | >=22.13.0 |
| <code>@mastra/mongodb</code> | 1.18.4 | — | >=22.13.0 |
| <code>@mastra/redis</code> | 1.4.2 | — | >=22.13.0 |
| <code>@mastra/clickhouse</code> | 1.16.0 | 1.16.0-alpha.2 | >=22.13.0 |
| <code>@mastra/duckdb</code> | 1.6.3 | 1.6.3-alpha.0 | >=22.13.0 |
| <code>@mastra/inngest</code> | 1.8.8 | — | >=22.13.0 |
| <code>@mastra/temporal</code> | 0.4.1 | 0.4.2-alpha.0 | >=22.13.0 |
| <code>@mastra/redis-streams</code> | 0.4.0 | 0.4.0-alpha.1 | >=22.13.0 |
| <code>@mastra/valkey-streams</code> | 0.5.0 | 0.5.0-alpha.0 | >=22.13.0 |
| <code>@mastra/google-cloud-pubsub</code> | 1.1.2 | 1.1.2-alpha.0 | >=22.13.0 |
| <code>@mastra/hono</code> | 1.7.5 | 1.7.6-alpha.0 | >=22.13.0 |
| <code>@mastra/express</code> | 1.5.7 | 1.5.8-alpha.0 | >=22.13.0 |
| <code>@mastra/next</code> | 0.2.21 | 0.2.22-alpha.0 | >=22.13.0 |
| <code>@mastra/otel-exporter</code> | 1.3.12 | 1.3.12-alpha.0 | >=22.13.0 |
| <code>@mastra/otel-bridge</code> | 1.5.4 | 1.5.4-alpha.0 | >=22.13.0 |

The table is a dated snapshot, not an instruction to upgrade blindly. Verify
current registry state and security notices before installation.

## Maturity ledger

| Surface | Evidence | Conclusion |
|---|---|---|
| Core agents/tools/workflows | Stable 1.x packages and current docs/source | Stable base, with provider/application semantics still tested |
| Memory/storage | Stable packages; adapter domains differ | Stable base, capability-test each adapter |
| Server/generated routes | Stable packages and active changelog | Stable, adapter and route policy are version seams |
| MCP | Stable 1.x package | Stable base; remote servers remain a trust boundary |
| Observability/evals | Stable 1.x packages | Stable base; sampling/redaction/gates application-owned |
| Durable agents | Documentation explicitly identifies beta API | Beta |
| Workers | Docs identify beta and list missing DLQ/scheduler/crash behavior | Beta with operational gaps |
| Response cache | Documented beta processor | Beta; pure/read-only turns only |
| Workflow definitions/editor persistence | Beta documentation/source | Beta |
| Observational memory, resource scope | Experimental documentation | Experimental |
| Working-memory state signals | Experimental documentation | Experimental |
| Inngest adapter | 1.8.8 package; durable-agent factory beta | Stable adapter package, beta wrapper surface |
| Temporal adapter | 0.4.1 and narrow build/plugin docs | Pre-1.0/emerging |
| Redis/Valkey Streams | 0.4.0/0.5.0 | Pre-1.0 adapters |
| <code>.network()</code> | Official migration to supervisor | Deprecated |
| FGA | Current EE source/docs/blog | Product/license-specific, version-sensitive authorization layer |
| Mastra Platform observability exporter | Current docs/source use <code>MastraPlatformExporter</code>; <code>CloudExporter</code> deprecated | Stable hosted-export surface with buffered-delivery shutdown obligations |

## Claim ledger

### Runtime and agents

| Claim | Evidence | Guide treatment |
|---|---|---|
| Registry retrieval attaches shared services | Mastra class/agent registration source and docs | Registered agents/workflows are production default |
| Request context propagates request values | Server/request-context docs and implementation | Dependency injection, not an auth decision |
| Agent turns can be bounded by steps/time/concurrency | Agent docs/reference/source | Required production configuration |
| Processor order matters around memory persistence | Processor/memory source and docs | Security-relevant regression test |
| Response caching can replay recorded tool-call content without effects | Response cache docs/source | Do not use for effecting turns |
| Delegation hooks default to warning/fallback behavior | Stable 1.63.2 declarations/source | Security filters set <code>hookErrorStrategy: "throw"</code>; tools still authorize independently |
| Parent model normally receives subagent text, not nested tool results | Subagent docs/source | Opt-in model context expansion is a disclosure/cost decision |
| Fresh-process durable recovery cannot reconstruct function-valued per-call policy | Stable 1.63.2 packed artifact | Hard bounds and authorization live in durable/static boundaries; process-replacement conformance test |

### Tools and MCP

| Claim | Evidence | Guide treatment |
|---|---|---|
| Tool schemas validate input, execution receives runtime context | Tool docs/source | Schema plus tool-level authorization |
| Approval and suspension are distinct | HITL/tool docs | Consent versus missing-information distinction |
| Approval/suspend-capable tools can reduce concurrency | Agent stream/tool docs and issue #20098 | Account for sequential behavior |
| MCP stdio uses a curated environment and HTTP supports host controls | MCP docs/source | Explicit environment and redirect enforcement |
| Nested streaming approval lost inner details in 1.57.0 | Issue #20934; closed by #20948 | Historical version-specific regression fixture |

### Memory and storage

| Claim | Evidence | Guide treatment |
|---|---|---|
| Thread is conversation; resource is owner; ownership immutable | Memory docs/source | Server-derived IDs and access checks |
| Working memory has Markdown and schema update semantics | Working-memory docs/source | Small non-authoritative state |
| Resource-scoped observational memory is experimental | OM docs/source | Isolate and evaluate |
| Storage is domain-based and adapter coverage varies | Storage interfaces/adapters/docs | Capability matrix before selection |
| Retention is opt-in and pruning must be invoked | Retention docs/source | Scheduled per-domain policy |

### Workflows and streaming

| Claim | Evidence | Guide treatment |
|---|---|---|
| <code>parallel</code> starts fixed branches without a global cap; foreach has concurrency | Workflow docs/source | Explicit fan-out/backpressure controls |
| Resume reruns a suspended step handler | Workflow suspend/resume implementation/docs | Code before suspend must be replay-safe |
| Public workflow state reader now covers recovery data | Closed issue #16044/#16091 and current source | Never parse raw snapshots |
| Persistent state and stream replay are separate | PubSub/cache/durable source/docs | Design both independently |
| Active durable run index was still requested | Open issue #17998 | Application-owned operation-to-run mapping |
| Stable 1.63.2 resume atomically claims one suspended transition | Stable artifact plus #21725/changelog | Losing SDK call gets <code>WORKFLOW_RESUME_ALREADY_CLAIMED</code>; HTTP gets 409; effects remain idempotent |

### Reliability and durable execution

| Claim | Evidence | Guide treatment |
|---|---|---|
| Generated server drains then shuts Mastra down | Current server docs/source/changelog | Ordered shutdown and kill tests |
| Disabling generated signal handling removes access to generated HTTP drain | Configuration/deployment docs | Use adapter for complete host-owned lifecycle |
| Durable shutdown had a version-specific persistence report | Issue #21193 on Platform/core 1.56 | Regression test, not asserted current defect |
| Durable-agent recovery can redrive work and acquires a recovery lease | Stable 1.63.2 artifact | Idempotent tools; verify configured PubSub implements distributed lease rather than no-op fallback |
| Workers lack DLQ and require one scheduler | Current worker docs | Beta risk and external operational control |
| Inngest maps Mastra steps to external step machinery | Adapter docs/source | Separate retry/security profile |
| Temporal uses a build plugin and activities | Adapter docs/source | Pre-1.0, obey Temporal-native rules |

### Security and deployment

| Claim | Evidence | Guide treatment |
|---|---|---|
| Resource mapping protects memory/thread routes | Auth provider/server docs/source | Useful layer, not whole tenant authorization |
| FGA is EE for production | EE license and FGA docs/blog | License/product boundary stated |
| Direct SDK workflow invocation needs application guard | FGA/auth source/docs boundary | Explicit wrapper required |
| June 2026 malicious npm packages were published | Official issue #18061 | Artifact provenance and quarantine required |
| MCP docs server had a path disclosure advisory | GHSA-xh92-rqrq-227v | Dev MCP least privilege and patched-version control |
| Mastra Platform environment deploy is newer than old Cloud pages | July 2026 official environment announcement/current docs | Current product naming and deprecation noted |
| Hono-family and non-Hono adapters have different middleware contracts | Adapter source plus #21869/#22161 | Denied-route conformance per deployed adapter |
| Core 1.61.0 stopped persisting auth bearer data in several durable/score paths | Changelog plus #21975/#21996 | 1.63.2 is patched; older deployments inspect retained data and rotate exposed credentials |
| Platform <code>NPM_TOKEN</code> is excluded from runtime image layer but injected as runtime environment | Current Platform deploy docs | Read-only token; treat as application-readable runtime secret |

## Bounded issue review

Issues were sampled only where they sharpen a current production test:

| Issue | Status at review | Affected evidence | Use in guide |
|---|---|---|---|
| [#12029](https://github.com/mastra-ai/mastra/issues/12029) | Closed | Beta-era concurrent foreach context pollution report | Cross-run isolation stress fixture |
| [#15552](https://github.com/mastra-ai/mastra/issues/15552) | Closed | Around core 1.23, parallel foreach sibling suspend payload loss | Snapshot concurrency fixture |
| [#16044](https://github.com/mastra-ai/mastra/issues/16044) | Closed by #16091 | Missing stable recovery readers | Use current public reader API |
| [#17998](https://github.com/mastra-ai/mastra/issues/17998) | Open | Missing thin active durable-run index | Product owns run mapping |
| [#20098](https://github.com/mastra-ai/mastra/issues/20098) | Closed | Approval/suspend registration serialized otherwise-safe calls | Capacity regression fixture |
| [#20934](https://github.com/mastra-ai/mastra/issues/20934) | Closed by #20948 | Core 1.57 streaming nested approval hid inner effect | Approval UI regression fixture |
| [#21193](https://github.com/mastra-ai/mastra/issues/21193) | Closed | Platform/core 1.56 shutdown/persistence ordering | Forced-termination regression fixture |
| [#20443](https://github.com/mastra-ai/mastra/issues/20443) | Closed by [#21725](https://github.com/mastra-ai/mastra/pull/21725) | Concurrent resume could advance one suspension more than once before atomic claim work | Single-winner SDK and HTTP-409 regression fixture |
| [#21869](https://github.com/mastra-ai/mastra/issues/21869) | Closed by [#22161](https://github.com/mastra-ai/mastra/pull/22161) | Hono adapter omitted configured server middleware | Adapter auth/middleware registration fixture |
| [#21975](https://github.com/mastra-ai/mastra/issues/21975) | Closed by [#21996](https://github.com/mastra-ai/mastra/pull/21996) | Auth token persisted into snapshots/scores/durable input | Persistence secret-scanning fixture |

No closed issue is presented as evidence of a current defect in 1.63.2.

## Security evidence

### Supply-chain incident

Mastra's official
[#18061 incident report](https://github.com/mastra-ai/mastra/issues/18061)
states that a compromised maintainer account published malicious versions across
the namespace on 2026-06-16, using a credential-exfiltrating postinstall. The
team unpublished or deprecated affected releases and published replacements.

The report records 116 malicious publishes: 110 were unpublished and six were
deprecated, followed by replacement versions and removal of the compromised
token path. The malicious postinstall was credential-exfiltrating and
self-deleting, so absence from a later filesystem scan is not evidence that a
host was unaffected.

This evidence justifies:

- exact artifact/lockfile pinning;
- release quarantine;
- provenance/publisher validation;
- lifecycle-script inspection;
- clean rebuild and credential rotation procedures.

The packet intentionally does not reproduce a secondary source's affected
version list because it can become incomplete; incident response should consult
the current official notice and registry state.

### Advisory

[GHSA-xh92-rqrq-227v](https://github.com/mastra-ai/mastra/security/advisories/GHSA-xh92-rqrq-227v)
reported directory-listing/path traversal information exposure in
<code>@mastra/mcp-docs-server</code> through 0.13.8, patched in 0.17.0. The guide
uses it to support least privilege for development MCP servers. The advisory
metadata lists <code>&lt;=0.13.8</code>, while narrative text contains an
inconsistent <code>0.13.18</code> value; the packet therefore treats
<code>0.17.0</code> as the safe floor rather than inferring an intermediate
range.

The stable core changelog and [#21996](https://github.com/mastra-ai/mastra/pull/21996)
also establish a distinct historical persistence risk: core 1.61.0 stopped
storing <code>mastra__authToken</code> in workflow snapshots, score rows, and
durable-agent workflow input. Stable 1.63.2 includes the fix. The evidence does
not establish the first affected version, so the guide does not invent one.

## Claims deliberately downgraded or excluded

| Candidate claim | Decision | Reason |
|---|---|---|
| “Mastra workflows are durable/exactly once” | Rejected | Snapshots do not settle in-flight effect ambiguity |
| “Durable agent survives any crash transparently” | Rejected | Recovery can repeat model/tools; cross-replica fencing depends on a lease-capable PubSub |
| “Function-valued durable-agent policy survives a fresh worker” | Rejected | Stable 1.63.2 persists serializable shadows/metadata, not arbitrary JavaScript closures |
| “Delegation context filters fail closed by default” | Rejected | Default hook strategy warns and can fall back to unfiltered/original values |
| “Atomic resume makes the effect exactly once” | Rejected | It claims workflow advancement; a post-effect pre-checkpoint crash remains ambiguous |
| “A stable package means all features are stable” | Rejected | Core contains explicit beta/experimental surfaces |
| “All storage adapters support all domains/retention” | Rejected | Interfaces and implementations vary |
| “RequestContext is authorization” | Rejected | It transports values; trust depends on construction and checks |
| “Resource ID is complete tenant isolation” | Rejected | Tools, workflows, vectors, caches, telemetry, and artifacts need policy |
| “Stream reconnect equals workflow resume” | Rejected | Event replay and execution persistence are independent |
| “Issue #12029/#15552/#20934 is a current defect” | Rejected | Reports are closed and version-specific |
| “Current main docs exactly describe stable npm” | Rejected | Main was one alpha patch ahead |
| “Mastra Cloud public-beta docs are current Platform guidance” | Rejected | New environment/Platform flow supersedes legacy material |
| “CloudExporter is the current hosted exporter name” | Rejected | Current API is <code>MastraPlatformExporter</code>; old name is deprecated compatibility |
| “All adapters inherit <code>server.middleware</code>” | Rejected | Hono-family and native Express/Fastify/Koa middleware contracts differ |
| “Observational memory always replaces other memory” | Downgraded | Mastra recommendation requires workload evaluation |
| “Node/Bun/Deno are equally supported” | Downgraded | Current package engine declares Node >=22.13.0 |
| “Temporal adapter provides every Temporal feature” | Rejected | Pre-1.0 wrapper docs expose a narrow integration |

## Primary source map

### Framework and packages

- [Mastra repository](https://github.com/mastra-ai/mastra)
- [Pinned source snapshot](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad)
- [Mastra npm organization](https://www.npmjs.com/org/mastra)
- [Core manifest](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/package.json)
- [Core changelog](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/CHANGELOG.md)
- [Repository releases](https://github.com/mastra-ai/mastra/releases)

### Concepts and runtime

- [Agent docs source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/agents)
- [Tool/HITL docs source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/agents)
- [MCP docs source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/connections/mcp.mdx)
- [Memory docs source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/memory)
- [Workflow docs source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/workflows)
- [Storage docs source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/storage.mdx)
- [Server docs source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/server)
- [Integration catalog source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/integrations)
- [Observability docs source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/observability)
- [Eval docs source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/evals)

### Durable engines and deployment

- [Durable-agent guide source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/harness/durable-agents.mdx)
- [Durable-agent reference source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/reference/agents/durable-agent.mdx)
- [Durable-agent recovery lease implementation](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/src/agent/durable/durable-agent.ts)
- [Inngest integration source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/integrations/deploy/inngest.mdx)
- [Temporal integration source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/integrations/deploy/temporal.mdx)
- [Temporal TypeScript docs](https://docs.temporal.io/develop/typescript)
- [Inngest docs](https://www.inngest.com/docs)
- [Deployment docs source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/deployment)
- [Platform environments announcement](https://mastra.ai/blog/introducing-environments-for-mastra-platform)
- [Pinned Platform deploy source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/mastra-platform/deploy.mdx)
- [Pinned Platform observability source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/mastra-platform/observability.mdx)

### Security and migration

- [Authentication docs source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/auth)
- [FGA announcement](https://mastra.ai/blog/introducing-fine-grained-authorization)
- [License map](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/LICENSE.md)
- [Official npm compromise report](https://github.com/mastra-ai/mastra/issues/18061)
- [MCP docs-server advisory](https://github.com/mastra-ai/mastra/security/advisories/GHSA-xh92-rqrq-227v)
- [Atomic resume claim](https://github.com/mastra-ai/mastra/pull/21725)
- [Auth-token persistence fix](https://github.com/mastra-ai/mastra/pull/21996)
- [Hono middleware registration fix](https://github.com/mastra-ai/mastra/pull/22161)
- [Upgrade-to-v1 source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/reference/migrations/upgrade-to-v1)
- [Network-to-supervisor migration source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/reference/migrations/network-to-supervisor.mdx)

## Remaining uncertainties

These questions require version-specific deployment validation rather than a
general documentation answer:

1. Which storage domains and retention operations are complete for the exact
   selected adapter versions?
2. Does the selected stable server build include every alpha-main lifecycle
   change observed in source?
3. What are the deployed platform's exact drain, sleep, stream, and build limits
   for the selected plan and region?
4. How does each selected model/provider implement total timeout, step timeout,
   abort, structured output, and fallback billing?
5. Which nested approval behaviors are identical across direct, supervisor,
   durable-agent, client SDK, and server adapter paths?
6. Does the exact PubSub adapter implement and preserve lease acquisition,
   renewal, loss cancellation, and release under a network partition?
7. Which Temporal adapter limitations remain after 0.4.1 and before 1.0?
8. Which function-valued durable-agent options, if any, gain an explicit
   cross-process persistence/rebinding contract after 1.63.2?

The guide converts each uncertainty into an adoption test rather than guessing.

## Refresh triggers

Refresh this packet when:

- stable core moves beyond 1.63.x;
- the main/stable gap materially changes;
- durable agents or workers leave beta;
- response cache leaves beta;
- the Temporal/Redis Streams/Valkey Streams adapters reach 1.0;
- issue #17998 closes or a stable active-run index ships;
- generated-server shutdown or worker recovery semantics change;
- durable-agent option serialization/rebinding semantics change;
- a new security advisory or package-integrity incident is published;
- Mastra Platform completes the old deploy-command transition;
- <code>.network()</code> is removed.

## Completion audit

- [x] Current source commit and stable npm baseline recorded.
- [x] Agents, models, instructions, processors, tools, MCP, and approval covered.
- [x] Memory, context, storage domains, retention, and isolation covered.
- [x] Workflows, state, snapshots, suspend/resume, and time travel covered.
- [x] Streaming, events, PubSub, clients, and active-run indexing covered.
- [x] Supervisors, subagents, routing, and deprecated networks covered.
- [x] Observability, evals, testing, and debugging covered.
- [x] Reliability, retries, concurrency, workers, and shutdown covered.
- [x] Security, FGA, tenancy, MCP, and supply chain covered.
- [x] Self-hosting, adapters, serverless, Platform, scale, performance, and cost covered.
- [x] Built-in durability, durable agents, Inngest, and Temporal separated.
- [x] Package maturity, migrations, limitations, alternatives, and refresh triggers covered.
- [x] PASS-2 verified exact stable-artifact recovery, resume, adapter, Platform, and security seams.

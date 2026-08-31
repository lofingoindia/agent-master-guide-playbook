# Microsoft Agent Framework Deep Dive — Research Packet

**Research date:** 2026-08-31  
**Status:** Research-backed synthesis and evidence ledger  
**Scope:** Microsoft Agent Framework agents, workflows, integrations, hosting, security, operations, Python/.NET/Go maturity, AutoGen/Semantic Kernel predecessor boundaries, and narrowly selected issue-derived regression tests  
**Freshness:** Recheck package registries, `PACKAGE_STATUS.md`, Learn lifecycle notices, and provider/hosting capabilities before adoption and at least every 30–90 days

## Research question

What does Microsoft Agent Framework itself provide, which guarantees belong to model providers, remote agent services, self-hosting, Foundry Hosted Agents, or the Durable Extension, and what must an application add for production reliability, security, tenancy, observability, and migration?

## Method

Evidence priority:

1. current Microsoft Learn product, integration, hosting, identity, network, observability, evaluation, and security documentation;
2. official Microsoft Agent Framework Python/.NET and Go repositories, source-oriented ADRs, samples, package status, and changelog;
3. official PyPI, NuGet, and Go module metadata for the exact 2026-08-31 package snapshot;
4. official AutoGen and Semantic Kernel repositories/documentation for predecessor/migration boundaries;
5. narrowly selected GitHub issues as version-specific failure-test leads, never as prevalence estimates.

Marketing claims, download counts, stars, and sample success were not treated as reliability evidence. Feature tables were checked against package maturity and language-specific source. Contradictions were retained rather than silently choosing the most optimistic claim.

Local read-only source checkouts used during research:

| Repository | Checked commit | Purpose |
|---|---|---|
| `microsoft/agent-framework` | `edfe115ea06bca57ae5a123d0fac5b3fdda13603` | Python/.NET source, status, changelog, ADRs, samples |
| `microsoft/agent-framework-go` | `6c58ac4d9c8dff56e18ecc7cd9868821ec715448` | Go public-preview APIs, `go.mod`, feature comparison, gaps |

Research stopped after additional official searches repeated the established layer/maturity boundaries and produced no material architecture change. Dynamic provider/model/region entitlements remain deployment-time facts.

## Executive findings

1. MAF is a component family, not one uniformly mature runtime.
2. Python and .NET core agents and graph workflows are stable; extension maturity varies widely.
3. Go is real and documented, but is a separate public-preview implementation with no stable semver tag verified and several missing product/runtime features.
4. Model providers and remote agent services have different ownership: the former supplies inference while the latter can own definitions, tools, sessions, tasks, permissions, and execution.
5. The agent pipeline and workflow runtime are separate state machines.
6. Framework workflow checkpoints capture superstep state; they do not schedule dead workers or make external effects exactly once.
7. Provider background responses, Foundry resilient hosted work, and Durable Extension execution solve different continuation/recovery problems.
8. Hosting and client protocol are orthogonal decisions.
9. Sessions, conversations, workflow checkpoints, transport threads/tasks, and domain records are separate state authorities.
10. Local function middleware and approval cannot intercept provider-hosted tool execution.
11. Identity, authorization, durable stores, limits, idempotency, and output sanitization remain application responsibilities.
12. AutoGen migration is behavioral and architectural; Semantic Kernel can remain for non-agent capabilities during incremental migration.

## Package and language snapshot

### Python registry snapshot

Verified through PyPI JSON on 2026-08-31:

| Package | Latest | Upload date (UTC) | Python |
|---|---:|---|---|
| `agent-framework` | 1.16.0 | 2026-08-28 | `>=3.10` |
| `agent-framework-core` | 1.16.0 | 2026-08-28 | `>=3.10` |
| `agent-framework-openai` | 1.14.1 | 2026-08-28 | `>=3.10` |
| `agent-framework-foundry` | 1.11.0 | 2026-08-14 | `>=3.10` |
| `agent-framework-orchestrations` | 1.1.1 | 2026-08-21 | `>=3.10` |
| `agent-framework-hosting` | 1.0.0a260730 | 2026-07-30 | `>=3.10` |
| `agent-framework-foundry-hosting` | 1.0.0b260827 | 2026-08-28 | `>=3.10` |
| `agent-framework-declarative` | 1.0.3 | 2026-08-21 | `>=3.10` |
| `agent-framework-a2a` | 1.0.0b260821 | 2026-08-21 | `>=3.10` |
| `agent-framework-devui` | 1.0.0b260821 | 2026-08-21 | `>=3.10` |

The repository lifecycle file is more informative than version numbers alone:

- released: meta/core, AG-UI, declarative, Foundry, GitHub Copilot, OpenAI, orchestrations;
- beta: A2A, Anthropic, Bedrock, Gemini, Copilot Studio, DevUI, Foundry hosting/local, several data/memory/tool integrations;
- alpha: base hosting and hosting protocol adapters, plus selected storage/memory packages;
- deprecated: `agent-framework-azure-ai`, whose clients moved/changed under `agent-framework-foundry`.

Feature-stage decorators inside released packages mark Agent Hooks, declarative-agent loading, evals, file history, FIDES, selected Foundry tools, functional workflows, harness portions, MCP long-running tasks/skills, progressive tools, session store, and `to_prompt_agent` experimental.

Primary evidence: [Python package status](https://github.com/microsoft/agent-framework/blob/main/python/PACKAGE_STATUS.md), [Python changelog](https://github.com/microsoft/agent-framework/blob/main/python/CHANGELOG.md), and [PyPI meta-package](https://pypi.org/project/agent-framework/).

### .NET registry snapshot

Verified through NuGet V3 on 2026-08-31:

| Package | Latest | Latest stable |
|---|---:|---:|
| `Microsoft.Agents.AI` | 1.19.0 | 1.19.0 |
| `Microsoft.Agents.AI.Abstractions` | 1.19.0 | 1.19.0 |
| `Microsoft.Agents.AI.Workflows` | 1.19.0 | 1.19.0 |
| `Microsoft.Agents.AI.OpenAI` | 1.19.0 | 1.19.0 |
| `Microsoft.Agents.AI.Foundry` | 1.19.0-preview.260822.1 | 1.5.0 |
| `Microsoft.Agents.AI.Hosting` | 1.19.0-preview.260822.1 | none verified |
| `Microsoft.Agents.AI.Foundry.Hosting` | 1.19.0-preview.260822.1 | none verified |
| `Microsoft.Agents.AI.Anthropic` | 1.19.0-preview.260822.1 | none verified |
| `Microsoft.Agents.AI.Declarative` | 1.19.0-rc1 | none verified |
| `Microsoft.Agents.AI.DevUI` | 1.19.0-preview.260822.1 | none verified |

The stable `Microsoft.Agents.AI.Workflows` 1.19.0 package targets .NET 8, .NET Standard 2.0, and .NET Framework 4.7.2 according to NuGet. Runtime compatibility does not imply all provider/host packages support the same deployment shape.

Primary evidence: [core NuGet package](https://www.nuget.org/packages/Microsoft.Agents.AI/), [workflow NuGet package](https://www.nuget.org/packages/Microsoft.Agents.AI.Workflows/), and [self-hosting lifecycle notice](https://learn.microsoft.com/en-us/agent-framework/hosting/self-hosting/).

### Go snapshot

The Go proxy returned `v0.0.0-20260829074433-6c58ac4d9c8d`, commit `6c58ac4`, with no normal stable semver tag verified. The checked `go.mod` declares `go 1.26.0`. The repository calls the SDK public preview and says it evolves outside the upstream Python/.NET repository.

Verified core coverage includes agents, provider middleware, sessions/history, context/compaction, structured output, local tools/MCP/shell, graph workflows, checkpoints, HITL, sequential/concurrent/group-chat patterns, A2A, AG-UI, and OpenTelemetry.

Verified gaps include declarative agents/workflows, RAG, CodeAct, Python functional workflows, handoff orchestration, DevUI, Foundry managed hosting, Durable Extension, and some provider/admin/evaluation/storage integrations.

Primary evidence: [Go repository](https://github.com/microsoft/agent-framework-go), [Go feature comparison](https://github.com/microsoft/agent-framework-go/blob/main/docs/dotnet-go-sdk-feature-comparison.md), and [Go package reference](https://pkg.go.dev/github.com/microsoft/agent-framework-go).

## Architecture evidence ledger

### Agent and provider boundaries

| Claim | Evidence | Interpretation |
|---|---|---|
| Agent combines abstraction, connection, instructions, tools, middleware, context, and session | [Agent concepts](https://learn.microsoft.com/en-us/agent-framework/concepts/agents/) | A run is a composition, not only a prompt |
| Model provider leaves definition/tools/policy with the app | [Model providers](https://learn.microsoft.com/en-us/agent-framework/integrations/by-component/model-providers/) | Provider capability is not remote-agent ownership |
| Agent service may own definition, permissions, sessions, tools, tasks, execution | [Agent services](https://learn.microsoft.com/en-us/agent-framework/integrations/m365) | Local middleware cannot assume complete visibility |
| Pipelines differ across Python/.NET/Go | [Agent pipeline](https://learn.microsoft.com/en-us/agent-framework/agents/agent-pipeline) | Shared names do not guarantee identical interception order |
| Background responses are provider-dependent | [Background responses](https://learn.microsoft.com/en-us/agent-framework/agents/background-responses) | Continuation token is not framework durability |

### Tools and policy boundaries

| Claim | Evidence | Interpretation |
|---|---|---|
| Tools run without approval by default | [Agent Safety](https://learn.microsoft.com/en-us/agent-framework/agents/safety) | High-risk tools require explicit gate |
| Local and hosted tools have different interception/approval behavior | [Tools overview](https://learn.microsoft.com/en-us/agent-framework/agents/tools/) | Execution location is a security decision |
| Approval round-trip uses the same session | [Tool approval](https://learn.microsoft.com/en-us/agent-framework/agents/tools/tool-approval) | Persist/bind pending occurrence state |
| Third-party MCP data/credentials require review | [Local MCP tools](https://learn.microsoft.com/en-us/agent-framework/agents/tools/local-mcp-tools) | MCP server is an external trust boundary |
| Agent Hooks cannot pre/post-intercept hosted tools | [Agent Hooks](https://learn.microsoft.com/en-us/agent-framework/agents/agent-hooks) | Local policy coverage is incomplete for service-side execution |

### Sessions, context, and storage

| Claim | Evidence | Interpretation |
|---|---|---|
| Local and service-managed history are distinct | [Conversation storage](https://learn.microsoft.com/en-us/agent-framework/agents/conversations/storage) | Choose a history authority and avoid duplication |
| Session is provider/agent-specific opaque state | [Conversation storage](https://learn.microsoft.com/en-us/agent-framework/agents/conversations/storage) | Version and bind restores |
| Context providers are proactive; tools are reactive | [Adding context providers](https://learn.microsoft.com/en-us/agent-framework/journey/adding-context-providers) | Place data at the narrowest required seam |
| Self-hosting has no general durable session store | [Self-hosting](https://learn.microsoft.com/en-us/agent-framework/hosting/self-hosting/) | Application must provide durable, authorized persistence |

### Workflow and checkpoint semantics

| Claim | Evidence | Interpretation |
|---|---|---|
| All SDKs have graph workflows; Python functional is experimental | [Workflow concepts](https://learn.microsoft.com/en-us/agent-framework/concepts/workflows/) | Do not claim cross-language functional parity |
| Shared state is visible to other executors on next superstep | [Workflow state](https://learn.microsoft.com/en-us/agent-framework/workflows/state) | No same-step synchronization through state |
| Workflow instances should not be reused across requests by default | [Workflow state](https://learn.microsoft.com/en-us/agent-framework/workflows/state) | Build fresh agents/executors/workflow per task |
| Checkpoints capture executor state, pending messages/requests, shared state at boundaries | [Checkpoints](https://learn.microsoft.com/en-us/agent-framework/workflows/checkpoints) | Snapshot is broader than chat history, narrower than a transaction |
| Pending requests survive checkpoint restore/re-emission | [Workflow HITL](https://learn.microsoft.com/en-us/agent-framework/workflows/human-in-the-loop) | HITL requires durable, occurrence-aware UI/application state |
| Python 1.13 added entry checkpoints and minor ordering/ID breaks | [Checkpoints](https://learn.microsoft.com/en-us/agent-framework/workflows/checkpoints), [changelog](https://github.com/microsoft/agent-framework/blob/main/python/CHANGELOG.md) | Version-sensitive checkpoint consumers need tests |

### Orchestration findings

| Pattern | Source-backed behavior | Production interpretation |
|---|---|---|
| Sequential | Fixed agent pipeline | Use typed artifacts and stage budgets |
| Concurrent | Independent parallel agents + aggregation | Bound fan-out/cost and define partial failure |
| Handoff | Specialized executor/tool transfer; interactive default | Limit transitions/loops; test nested tool policy |
| Group chat | Central manager, separate sessions, synchronized context | Shared context increases exposure/cost |
| Magentic | Dynamic manager coordination | Highest need for limits, evals, and deterministic effect gates |

Primary evidence: [orchestration overview](https://learn.microsoft.com/en-us/agent-framework/workflows/orchestrations/), [handoff](https://learn.microsoft.com/en-us/agent-framework/workflows/orchestrations/handoff), and [group chat](https://learn.microsoft.com/en-us/agent-framework/workflows/orchestrations/group-chat).

## Hosting and recovery ledger

### Self-hosting

The application owns routes, identity, authorization, request policy, storage, deployment, scaling, and native clients. Protocol helpers do not implement a whole production API. Protocol IDs must be authenticated/authorized and durable state tenant-partitioned before load. Persist after a settled run/stream.

Primary evidence: [self-hosting](https://learn.microsoft.com/en-us/agent-framework/hosting/self-hosting/).

### Foundry Hosted Agents

Foundry runs a container per session in a VM-isolated sandbox. `$HOME` and `/files` persist across idle deprovisioning. Sessions and Responses conversations are distinct. The checked service documentation reports per-session scaling, 0.5/1/2-vCPU sizes, 5–60-minute idle timeout (15 default), and deletion after 30 inactive days.

The session isolation value scopes records but is not itself authorization. Current protocol 2 guidance derives user isolation from Entra; older protocol 1.0 isolation-key behavior was deprecated with a 2026-07-31 cutoff. Application-layer resource authorization and administrative-access threat modeling still apply.

Primary evidence: [hosted-agent concepts](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agents), [manage sessions](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/manage-hosted-sessions), [isolate sessions](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/isolate-sessions-per-user), and [runtime contract](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agent-contract).

### Foundry resilient work

The preview capability persists work/input identities, uses leases to detect process loss, re-enters handlers, and retains stream events. It does not restore the call stack or provide deterministic replay. Application checkpoints/watermarks and idempotency remain required. Full Responses crash recovery applies only to stored background work with resilience enabled; foreground Responses are not reinvoked.

Primary evidence: [long-running resilience](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/long-running-agent-resilience).

### Durable Extension

The separate extension supports C# and Python on Azure Functions or self-hosted workers backed by Durable Task Scheduler. It adds distributed recovery, durable agents/workflows, human waits, reliable streaming, and dashboard/worker infrastructure. Go was “coming soon” in the checked documentation. Durable activity effects remain at-least-once-sensitive and require idempotency.

Primary evidence: [Durable Extension](https://learn.microsoft.com/en-us/agent-framework/hosting/azure-functions) and [extension repository](https://github.com/microsoft/agent-framework-durable-extension).

## Observability, evaluation, and testing findings

- MAF emits OpenTelemetry-aligned agent/model/tool/workflow telemetry; application spans must cover host, state, authz, approval, and effects.
- Python instrumentation is on by default but exports only when a provider/exporter is configured. Sensitive data remains opt-in and should not be enabled in production.
- Framework User-Agent contribution and feature-category token have separate documented disable switches.
- .NET agent plus `IChatClient` instrumentation can create overlapping spans; inspect actual traces.
- Python core/Foundry evaluation APIs are experimental in `PACKAGE_STATUS.md` despite complete user documentation.
- Foundry model-judge evaluation is probabilistic; pin judge/rubric/dataset and combine with deterministic contracts.
- DevUI is development-only: Python beta, .NET preview, no checked Go implementation.
- Real-store checkpoint, restart, approval, parallel-tool, stream, and provider-content conformance tests are more valuable than happy-path samples.

Primary evidence: [agent observability](https://learn.microsoft.com/en-us/agent-framework/agents/observability), [workflow observability](https://learn.microsoft.com/en-us/agent-framework/workflows/observability), [agent evaluation](https://learn.microsoft.com/en-us/agent-framework/agents/evaluation), [Foundry evaluation](https://learn.microsoft.com/en-us/agent-framework/integrations/by-component/evaluation/microsoft-foundry), and [DevUI](https://learn.microsoft.com/en-us/agent-framework/integrations/by-component/ui/devui/).

## Security findings

- Framework sessions/checkpoints can contain sensitive content and privileged control data; storage compromise can change roles, tool requests, or routing.
- Resource authorization must occur before loading any session, checkpoint, task, thread, file, or approval.
- Tools validate and authorize at the effect boundary; prompts and approval are insufficient.
- Hosted tools bypass local function middleware, so use provider-native policy or a local tool when interception is mandatory.
- Untrusted user, model, retrieval, and tool content must never become developer/system instruction text.
- FIDES and Agent Hooks are Python-only experimental features with explicit coverage limits.
- FIDES unlabeled-content defaults are nuanced; label external sources explicitly.
- Foundry agent identity, caller identity, and delegated identity must not be conflated.
- Sensitive prompt/tool telemetry and production Trace logs are prohibited by the recommended posture.
- MAF does not impose application request-rate or input/output bounds; the host must do so.

Primary evidence: [Agent Safety](https://learn.microsoft.com/en-us/agent-framework/agents/safety), [FIDES](https://learn.microsoft.com/en-us/agent-framework/agents/security), [Agent Hooks](https://learn.microsoft.com/en-us/agent-framework/agents/agent-hooks), and [least privilege for AI agents](https://learn.microsoft.com/en-us/security/zero-trust/sfi/least-privilege-for-ai-agents).

## Changelog-derived upgrade risks

Selected current changes were used to design tests, not to imply instability across all paths:

| Release | Change | Required regression |
|---|---|---|
| Python 1.13.0 | Entry checkpoints/full replay; ordering/source-ID changes | Full run/replay fixtures and old checkpoint restore |
| Python 1.14.0 | Durable integrations moved repository; Foundry hosting Responses 2.x beta change | Dependency/deployment and hosted session migration |
| Python 1.15.0 | OpenTelemetry convention changes; approval persistence updates | Dashboard/query compatibility and approval restart |
| Python 1.16.0 | Checkpoint mutation fix; continuation/session fixes | Store immutability and provider continuation tests |

Additional 1.12–1.16 fixes around duplicate calls after approval, parallel approval, compaction pairing/bounds, streamed transcript duplication, and subworkflow state justify permanent regression cases.

## Bounded issue register

Issues are evidence of specific reproduced/tested failure shapes at particular versions. Closed issues are retained as regression-test leads. Open issues are not generalized beyond the reported surface.

| Issue | Status at research | Version/surface in report | Use in playbook |
|---|---|---|---|
| [#5350](https://github.com/microsoft/agent-framework/issues/5350) | closed | .NET 1.1 JSON checkpoint + tool approval | Round-trip all concrete approval/content subtypes with real serializer/store |
| [#4158](https://github.com/microsoft/agent-framework/issues/4158) | open/backlog | .NET preview `1.0.0-preview.260212.1`, nested MCP under handoff | Test nested local-tool middleware/approval coverage; do not claim universal current breakage |
| [#6910](https://github.com/microsoft/agent-framework/issues/6910) | closed via #6947 | Python core 1.10.0, AG-UI 1.0.0rc7 | Test parallel mixed approvals, session persistence, event correlation |
| [#7458](https://github.com/microsoft/agent-framework/issues/7458) | closed, `likely-fixed` label | Python AG-UI/main reproduction | Test tool commit + post-consume provider/stream failure + duplicate resume |
| [#7570](https://github.com/microsoft/agent-framework/issues/7570) | closed | Python AG-UI/main reproduction | Ensure terminal stale approval interrupts are retired after reconnect |

No defect rate or current-version failure probability was inferred from these issues.

## Contradiction and documentation-lag register

### Supported languages

- FAQ body: “currently supports .NET (C#) and Python.”
- Same page’s related overview text and current overview/get-started: Python, .NET, and Go.
- Go repository: public preview, separate codebase.

Resolution: document Go as available public preview, not stable/upstream parity.

### Foundry GA versus integration maturity

- Foundry Hosted Agents service: generally available.
- Python MAF Foundry hosting: beta package.
- .NET MAF Foundry hosting: preview package.
- Long-running resilience: preview capability.

Resolution: track service, adapter package, and optional resilience feature separately.

### Stable package versus experimental feature

- Python core/declarative packages have stable releases.
- Feature-stage registry still marks evals, FIDES, Hooks, functional workflows, declarative-agent loading, session store, and other APIs experimental.

Resolution: maturity is `language + package + feature`, not package only.

### FIDES design status

- Learn docs and `PACKAGE_STATUS.md`: implemented experimental Python feature.
- ADR 0024: status remains proposed.

Resolution: use shipped status/source/tests to describe availability, but retain experimental label and ADR inconsistency.

### Migration guide staleness

- AutoGen migration provider table marks Anthropic/Ollama or local execution as planned in sections that conflict with current provider/package/tool documentation.

Resolution: use migration guide for behavior mapping only; use current registries/integration pages for availability.

### Foundry isolation model transition

- Some current session documentation explains isolation keys/header authorization modes.
- protocol 2 migration documentation says protocol 1.0 caller-supplied isolation behavior was deprecated/blocked after 2026-07-31 and emphasizes Entra-derived per-user isolation.

Resolution: record deployed AgentServer protocol version, prefer current identity-derived isolation, and never treat a partition key as authorization.

## Predecessor boundaries

### AutoGen

The official repository says AutoGen is in maintenance mode, community-managed, with no new features/enhancements, and directs new users to MAF. AutoGen Core includes an event-driven local/distributed runtime that does not map automatically to MAF’s default in-process graph workflow. Migration must identify whether MAF workflow, Durable Extension, A2A, or an external workflow engine supplies each old guarantee.

Primary evidence: [AutoGen repository](https://github.com/microsoft/autogen) and [MAF migration guide](https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-autogen/).

### Semantic Kernel

Semantic Kernel’s agent surface maps to MAF agents/sessions/tools/workflows, but existing plugins, prompt templates, AI services, and vector-store connectors can remain during incremental migration. MAF uses Microsoft.Extensions.AI types in .NET and direct tool/client composition, but provider resource deletion and behavior still require explicit handling.

Primary evidence: [Semantic Kernel migration guide](https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-semantic-kernel/) and [Semantic Kernel agent framework](https://learn.microsoft.com/en-us/semantic-kernel/frameworks/agent/).

## Decision record

The guide cluster recommends:

- a single agent or ordinary code before multi-agent orchestration;
- graph workflows for explicit production topology, with Python functional workflows isolated as experimental;
- domain services as the only authority for effects and business records;
- fresh workflow/agent/executor state per request unless reset behavior is documented and tested;
- real durable session/checkpoint stores plus authenticated composite keys;
- provider conformance testing rather than relying on broad support matrices;
- hosting/protocol choices made independently;
- stable operation IDs and receipts across every recovery model;
- feature-level maturity checks and small adapters around experimental surfaces;
- language-specific architecture matrices rather than parity assumptions.

## Unresolved or deployment-specific questions

- When will Go receive a stable semantic-version release and upstream alignment?
- Which current .NET integration APIs have feature-level staging equivalent to Python’s central registry? NuGet suffix and docs are currently the practical sources.
- What exact provider/model/region/account entitlements apply to hosted tools and background responses in the target deployment?
- What compatibility window will each application support for retained session/checkpoint/protocol state across rolling upgrades?
- Which protocol event/content types are intentionally unsupported by each chosen adapter?
- What are the organization’s retention/deletion requirements across provider conversations, Foundry sessions/files, checkpoints, transport snapshots, telemetry, and eval data?

These must be answered during system design; the framework cannot answer them generically.

## Refresh checklist

- [ ] Query PyPI, NuGet, and Go module metadata again.
- [ ] Re-read Python `PACKAGE_STATUS.md` and the latest Python/.NET release notes.
- [ ] Check Go README and feature-comparison changes.
- [ ] Re-check self-host, Foundry hosting, AgentServer protocol, and Durable Extension lifecycle notices.
- [ ] Re-check provider and tool matrices against the exact deployment.
- [ ] Re-run real-store session/checkpoint/approval/stream conformance suites.
- [ ] Revisit open issue #4158 only for the exact nested .NET path in use.
- [ ] Revalidate FIDES/Hooks maturity and coverage if adopted.
- [ ] Record resolved contradictions or new documentation lag explicitly.

## Primary source index

### Core framework

- [Overview](https://learn.microsoft.com/en-us/agent-framework/overview/)
- [Official Python/.NET repository](https://github.com/microsoft/agent-framework)
- [Official Go repository](https://github.com/microsoft/agent-framework-go)
- [Agent concepts](https://learn.microsoft.com/en-us/agent-framework/concepts/agents/)
- [Workflow concepts](https://learn.microsoft.com/en-us/agent-framework/concepts/workflows/)
- [FAQ](https://learn.microsoft.com/en-us/agent-framework/support/faq)

### Agents, tools, context, state

- [Agent pipeline](https://learn.microsoft.com/en-us/agent-framework/agents/agent-pipeline)
- [Running agents](https://learn.microsoft.com/en-us/agent-framework/agents/running-agents)
- [Model providers](https://learn.microsoft.com/en-us/agent-framework/integrations/by-component/model-providers/)
- [Agent services](https://learn.microsoft.com/en-us/agent-framework/integrations/m365)
- [Tools](https://learn.microsoft.com/en-us/agent-framework/agents/tools/)
- [Middleware](https://learn.microsoft.com/en-us/agent-framework/agents/middleware/)
- [Conversation storage](https://learn.microsoft.com/en-us/agent-framework/agents/conversations/storage)
- [Context providers](https://learn.microsoft.com/en-us/agent-framework/journey/adding-context-providers)

### Workflows

- [Workflow state](https://learn.microsoft.com/en-us/agent-framework/workflows/state)
- [Checkpoints](https://learn.microsoft.com/en-us/agent-framework/workflows/checkpoints)
- [Human-in-the-loop](https://learn.microsoft.com/en-us/agent-framework/workflows/human-in-the-loop)
- [Events](https://learn.microsoft.com/en-us/agent-framework/concepts/workflows/events)
- [Orchestrations](https://learn.microsoft.com/en-us/agent-framework/workflows/orchestrations/)
- [Functional API](https://learn.microsoft.com/en-us/agent-framework/workflows/functional)

### Hosting and operations

- [Hosting overview](https://learn.microsoft.com/en-us/agent-framework/hosting/)
- [Self-hosting](https://learn.microsoft.com/en-us/agent-framework/hosting/self-hosting/)
- [Foundry Hosted Agents](https://learn.microsoft.com/en-us/agent-framework/hosting/foundry-hosted-agent)
- [Foundry hosted-agent concepts](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agents)
- [Foundry sessions](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/manage-hosted-sessions)
- [Foundry long-running resilience](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/long-running-agent-resilience)
- [Durable Extension](https://learn.microsoft.com/en-us/agent-framework/hosting/azure-functions)

### Safety and quality

- [Agent Safety](https://learn.microsoft.com/en-us/agent-framework/agents/safety)
- [FIDES](https://learn.microsoft.com/en-us/agent-framework/agents/security)
- [Agent Hooks](https://learn.microsoft.com/en-us/agent-framework/agents/agent-hooks)
- [Observability](https://learn.microsoft.com/en-us/agent-framework/agents/observability)
- [Evaluation](https://learn.microsoft.com/en-us/agent-framework/agents/evaluation)
- [DevUI](https://learn.microsoft.com/en-us/agent-framework/integrations/by-component/ui/devui/)

### Maturity and migration

- [Python package status](https://github.com/microsoft/agent-framework/blob/main/python/PACKAGE_STATUS.md)
- [Python changelog](https://github.com/microsoft/agent-framework/blob/main/python/CHANGELOG.md)
- [AutoGen maintenance notice](https://github.com/microsoft/autogen)
- [AutoGen migration](https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-autogen/)
- [Semantic Kernel migration](https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-semantic-kernel/)

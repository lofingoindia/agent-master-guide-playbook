# Research packet: OpenAI Agents SDK deep dive

**Research date:** 2026-08-31  
**Status:** Official-documentation and official-source evidence packet; version-sensitive  
**Scope:** Python `openai-agents` 0.22.0 and TypeScript `@openai/agents` 0.17.0, plus current OpenAI developer guidance for Responses, Agents, tools, evals, safety, and production

This packet records the evidence and reasoning behind the [OpenAI Agents SDK production deep dive](../../frameworks/openai-agents-sdk/README.md). It deliberately separates confirmed product behavior, language parity, application responsibilities, and operational inference.

## Research question

What must an engineering team understand to design, build, operate, debug, evaluate, secure, and upgrade a production system based on the OpenAI Agents SDK—without mistaking the SDK for a durable workflow engine or assuming Python/TypeScript/base-API parity?

## Method and evidence hierarchy

Research prioritized:

1. current OpenAI developer documentation;
2. current official Python and TypeScript SDK documentation;
3. pinned official repository source, package metadata, changelogs, and examples;
4. operational inferences derived from those contracts, clearly labeled as such.

No third-party blog was used as authority. No GitHub issue was needed as failure evidence. This makes the packet strong on documented and source-visible contracts but intentionally weaker on independent long-term production experience. Teams should add local load, failure-injection, security, and provider acceptance evidence before declaring a workload production-ready.

## Inspected versions

| Surface | Snapshot |
|---|---|
| Research cutoff | 2026-08-31 |
| Python package | `openai-agents` 0.22.0 |
| Python repository | commit `89c02c828ee8510fe9a84ee6675608193aa13b02`, dated 2026-08-28 |
| Python runtime constraint | Python 3.10+ in inspected package metadata |
| Python OpenAI client constraint | `openai>=3,<4` in inspected package metadata |
| TypeScript package | `@openai/agents` 0.17.0 |
| TypeScript repository | commit `8e862b3380a577df1315bef17f351c1b58c2938b`, dated 2026-08-28 |
| TypeScript OpenAI client family | 7.x in inspected package metadata |
| TypeScript schema peer family | Zod 4 in inspected package metadata |

Both SDKs were pre-1.0 and used modified semantic-version policies. A current implementation must recheck package metadata rather than copying these constraints.

## Core findings

### 1. The runner is a bounded in-process control loop

Confirmed:

- One SDK run represents one application-level turn.
- The runner repeatedly invokes the active agent's model, inspects output items, executes tools or handoffs, and stops at a final output or interruption/failure.
- Both inspected SDKs defaulted to ten turns.
- A turn is a model call, not each tool call.
- Local function execution concurrency is distinct from a model's parallel-tool-call setting.

Operational inference:

- The turn limit does not bound total wall time, external effects, hosted-tool cost, or local concurrency.
- A production wrapper needs independent wall-clock, token/cost, tool, nested-call, and concurrency budgets.

### 2. State has several non-equivalent layers

Confirmed:

- Conversation history can be explicit, SDK-session-backed, OpenAI-Conversations-backed, or chained using previous response state.
- Local run context is not automatically sent to the model.
- Approval interruptions can be serialized into resumable RunState.
- Sandbox workspace state/snapshots/memory are distinct from SDK conversation sessions.

Strongest safe rule:

- Choose one authoritative conversation-history strategy per run. Python explicitly rejects relevant session/provider-state combinations, and TypeScript guidance also discourages combining them.

Operational inference:

- Encrypt and version RunState because it may include more than user-visible history.
- Conversation deletion must cover sessions, provider state, traces/evals, RunState, snapshots, artifacts, and workspace memory according to policy.

### 3. Tool types have different execution and trust boundaries

Confirmed categories:

- application function tools;
- agent-as-tool nested runs;
- Responses-hosted tools;
- local or hosted runtime tools such as shell/computer/apply-patch;
- hosted or runtime-managed MCP;
- tool search;
- programmatic tool calling;
- experimental/beta tool surfaces.

Confirmed:

- Hosted tools depend on the Responses/model surface and do not become portable simply because the SDK has an abstraction.
- Function-tool schemas are strict by default in the normal typed paths.
- Tool guardrails attach to function tools, not every possible tool/handoff.
- Programmatic tool calling executes generated JavaScript in a fresh restricted V8 isolate without Node/filesystem/network except explicitly allowed tools.

Operational inference:

- Execution-time authorization, effect idempotency, deadlines, reconciliation, and safe error mapping remain application responsibilities.
- Programmatic grouping increases partial-effect and review complexity for mutations.

### 4. Guardrails and approvals have precise boundaries

Confirmed:

- Input guardrails apply at the first-agent input boundary.
- Output guardrails apply to the agent producing the final output.
- Function-tool guardrails apply around attached function tools.
- Blocking input-guardrail execution exists for cases where speculative work before rejection is unacceptable.
- Approvals return interruption data and resumable state; the application applies approve/reject decisions and resumes the same logical state.

Operational inference:

- Authorization must be evaluated again at tool execution and approval resume.
- Approval cards should be generated from validated arguments and trusted policy metadata, not model prose.

### 5. Streaming requires settlement after visible output

Confirmed:

- Streams expose raw model events, run-item events, and active-agent updates.
- Python consumers must exhaust/settle the streamed run; TypeScript provides a completion promise that should be awaited.
- Session persistence, compaction, approval bookkeeping, or failures can occur after the last visible token.
- Responses WebSocket transport is not the Realtime API.

Parity detail:

- Python retains `handoff_occured`; TypeScript uses `handoff_occurred`.

Operational inference:

- UI state needs separate “visible complete” and “settled complete” states.
- Reconnect/rebuild must consult the effect ledger before replaying any run that could have mutated state.

### 6. Retries are replay-policy decisions

Confirmed:

- Model retries are opt-in and can use rich failure/replay classifications.
- Model timeout is per attempt, not a total run/tool deadline.
- Replay can be vetoed after streamed output or unsafe effects.
- Run error handlers cover a bounded set of terminal cases and do not replay tools.

Operational inference:

- External tools need application-generated stable effect IDs, idempotency where available, and reconciliation.
- RunState is continuation state, not durable scheduling or exactly-once execution.

### 7. Tracing, testing, and evals are complementary

Confirmed:

- Built-in tracing captures model, tool, handoff, guardrail, and custom spans.
- Sensitive model/tool data may appear in traces.
- tracing is unavailable under Zero Data Retention.
- Both current SDKs include provider-neutral deterministic scripted-model testing utilities.
- OpenAI eval surfaces include trace graders, datasets, and eval runs.

Operational inference:

- Scripted tests prove normalized SDK control flow, not actual model behavior/provider wire semantics.
- Release gates should combine deterministic tests, provider contracts, trace inspection, dataset/adversarial evals, and canaries.

### 8. Sandbox agents are isolated workspace runtimes, not workflow engines

Confirmed:

- Sandbox agents are beta in both languages.
- Local Unix support targets macOS/Linux; Windows users should use Docker/hosted choices.
- Manifests describe fresh-workspace behavior; live sessions, RunState, conversation sessions, and snapshots affect actual continuity.
- caller-provided live sandbox sessions put cleanup ownership on the caller.
- Workspace memory is separate from conversation history and can derive from prior sandbox activity.

Operational inference:

- Credentials, mounts, snapshots, artifacts, and memory are privileged/retained data surfaces.
- Use a durable workflow/job system around bounded sandbox activities for timers, crashes, and external effects.

## Python/TypeScript parity ledger

| Topic | Finding | Confidence |
|---|---|---|
| Core agent/run/tool/handoff concepts | Strong conceptual parity | High |
| Default max turns | 10 in inspected source | High |
| Result/state naming | Language-specific snake_case/camelCase and object shapes | High |
| Session backend breadth | Python documentation materially broader | High |
| Transaction-aware session extension | Explicit TypeScript surface; no equivalent claim supported in inspected Python docs | Medium-high |
| Tool-search client execution | Python manual-loop limitation; TypeScript helper execution surface | High |
| Nested handoff history | Python opt-in beta; equivalent TypeScript claim not established | Medium-high |
| Deterministic testing | Both now provide scripted-model utilities | High |
| Sandbox agents | Both beta; provider/platform details differ | High |
| Durable workflow integrations | Python documentation names several; no equivalent core TypeScript parity established | Medium-high |
| Serialized RunState interchange | No cross-language compatibility guarantee found | High as a non-guarantee |

Absence from inspected documentation was recorded as “not established,” never as proof of impossibility.

## Base API versus Agents SDK ledger

| Claim | Category |
|---|---|
| Model request/response and hosted tool protocol | Responses/base API |
| Server-managed conversation/previous-response behavior | Responses/base API |
| HTTP/SSE/Responses WebSocket transport | Responses/base API exposed through SDK/provider |
| Agent loop, normalized items, handoffs | Agents SDK |
| Function-tool dispatch in application process | Agents SDK + application |
| SDK sessions, guardrails, approval interruption | Agents SDK |
| Authentication, tenant authorization, effect ledger | Application |
| Durable timers, leases, compensation | Workflow/application infrastructure |
| Interactive low-latency media sessions | Realtime API/surface |
| Isolated workspace and snapshots | Sandbox agent/provider surface |

## Contradictions and resolved ambiguities

### “SDK sessions can be combined with provider state”

Some high-level descriptions list all state options without emphasizing exclusivity. The detailed run/session contracts show that combining them can duplicate or conflict with history; Python has explicit invalid combinations. Resolution: document one authoritative history strategy per run.

### “Streaming is complete when the final token appears”

User interfaces often assume this, but SDK settlement can continue for persistence or compaction. Resolution: distinguish visible completion from run settlement and require stream drain/completion await.

### “Parallel tool calls means tools run in parallel”

The model setting controls emitted calls; a separate runner setting governs local function execution concurrency. Resolution: document and budget both.

### “A timeout means the tool failed”

The SDK can report a timeout while a blocking operation or external effect continues/completes. Resolution: treat mutating-tool timeout as unknown outcome and reconcile.

### “Provider adapters make Agents portable”

Core calling abstractions are portable to a degree, but hosted tools, schemas, item ordering, streaming, usage, and continuation semantics vary. Resolution: portability is an acceptance-test result, not an interface claim.

### “Sandbox resume, snapshot, RunState, and session are the same”

They represent different state layers. Resolution: define each and make product promises precise.

## Operational recommendations derived from evidence

These are reasoned recommendations, not direct product guarantees:

- Start with one agent and typed tools; add multi-agent boundaries only when separately evaluated capabilities justify them.
- Run one bounded SDK turn per online request/worker activity.
- Move long waits and mutating multi-step work into a durable workflow.
- Persist effect intent before mutation and reconcile uncertain outcomes.
- Persist encrypted/versioned RunState before displaying approvals.
- Pin exact SDK versions and an explicit model; upgrade them separately.
- Treat all model, retrieval, MCP, and tool-output content as untrusted.
- Keep trace payloads minimal and design an alternate audit path for ZDR workloads.
- Test failure after every meaningful boundary: provider accept, stream start, tool effect, session commit, approval persist, compaction, and shutdown.

## Coverage map

| Evidence area | Resulting guide |
|---|---|
| Runner source and run guide | [Architecture and run lifecycle](../../frameworks/openai-agents-sdk/architecture-and-run-lifecycle.md) |
| Agent/model/provider docs | [Agents, models, and provider boundaries](../../frameworks/openai-agents-sdk/agents-models-and-provider-boundaries.md) |
| Tool/MCP/structured-output docs | [Tools and structured outputs](../../frameworks/openai-agents-sdk/tools-and-structured-outputs.md) |
| Sessions/result/state docs | [Sessions, context, and state](../../frameworks/openai-agents-sdk/sessions-context-and-state.md) |
| Orchestration/handoff docs | [Handoffs and multi-agent design](../../frameworks/openai-agents-sdk/handoffs-and-multi-agent.md) |
| Streaming/transport docs | [Streaming, events, and Realtime](../../frameworks/openai-agents-sdk/streaming-events-and-realtime.md) |
| Tracing/testing/evals docs | [Tracing, evaluation, and testing](../../frameworks/openai-agents-sdk/tracing-evaluation-and-testing.md) |
| Guardrail/safety/RBAC/MCP docs | [Security, guardrails, and approvals](../../frameworks/openai-agents-sdk/security-guardrails-and-approvals.md) |
| Retry/cancellation/state docs | [Reliability, cancellation, and recovery](../../frameworks/openai-agents-sdk/reliability-cancellation-and-recovery.md) |
| Production/latency/cost docs | [Deployment, operations, and cost](../../frameworks/openai-agents-sdk/deployment-operations-and-cost.md) |
| Sandbox docs/source | [Sandbox agents and long-running work](../../frameworks/openai-agents-sdk/sandbox-agents-and-long-running-work.md) |
| Package metadata/releases/source | [Version, parity, migrations, and limitations](../../frameworks/openai-agents-sdk/version-parity-migrations-and-limitations.md) |

## Unresolved questions and local validation needs

- How well does the selected provider/model preserve strict schema, tool ordering, stream items, and usage under this application's prompts?
- Which exact session backend meets the application's atomicity, latency, retention, and concurrency requirements?
- What is the observed behavior of custom synchronous/blocking tools under cancellation?
- How does the selected sandbox provider handle crash cleanup, network policy, snapshot encryption, and quotas?
- What trace content remains after local redaction in the chosen exporter path?
- How should paused RunState be migrated or expired during real rolling deployments?
- What cost/latency/quality threshold makes a specialist agent worth its extra call?
- Which mutations can be made idempotent by the target API and which require reconciliation or human recovery?

These need workload-specific tests; official docs cannot answer them.

## Refresh protocol

On each SDK minor upgrade:

1. record new package metadata and official commit/release;
2. diff release notes and public interfaces used by the application;
3. re-run the parity ledger;
4. test stored session and RunState fixtures;
5. run tool/stream/retry/approval/provider contracts;
6. run quality, safety, latency, and cost evals;
7. update every affected guide and its research date.

Also refresh for a default-model change, Responses tool/state/transport change, tracing/ZDR change, MCP security update, or sandbox beta/provider change.

## Primary source register

### Current developer guides

- [Agents overview](https://developers.openai.com/api/docs/guides/agents)
- [Define agents](https://developers.openai.com/api/docs/guides/agents/define-agents)
- [Models](https://developers.openai.com/api/docs/guides/agents/models)
- [Run agents](https://developers.openai.com/api/docs/guides/agents/running-agents)
- [Orchestration](https://developers.openai.com/api/docs/guides/agents/orchestration)
- [Guardrails and approvals](https://developers.openai.com/api/docs/guides/agents/guardrails-approvals)
- [Results](https://developers.openai.com/api/docs/guides/agents/results)
- [Sandbox agents](https://developers.openai.com/api/docs/guides/agents/sandboxes)
- [Integrations and observability](https://developers.openai.com/api/docs/guides/agents/integrations-observability)
- [Agent evals](https://developers.openai.com/api/docs/guides/agent-evals)
- [Connectors and remote MCP](https://developers.openai.com/api/docs/guides/tools-connectors-mcp)
- [Production best practices](https://developers.openai.com/api/docs/guides/production-best-practices)
- [Deployment checklist](https://developers.openai.com/api/docs/guides/deployment-checklist)
- [Latency optimization](https://developers.openai.com/api/docs/guides/latency-optimization)
- [Cost optimization](https://developers.openai.com/api/docs/guides/cost-optimization)
- [Safety best practices](https://developers.openai.com/api/docs/guides/safety-best-practices)
- [Role-based access control](https://developers.openai.com/api/docs/guides/rbac)

### Official SDK documentation and source

- [OpenAI Agents SDK Python documentation](https://openai.github.io/openai-agents-python/)
- [OpenAI Agents SDK Python release notes](https://openai.github.io/openai-agents-python/release/)
- [OpenAI Agents SDK Python inspected source snapshot](https://github.com/openai/openai-agents-python/tree/89c02c828ee8510fe9a84ee6675608193aa13b02)
- [OpenAI Agents SDK TypeScript documentation](https://openai.github.io/openai-agents-js/)
- [OpenAI Agents SDK TypeScript changelog](https://github.com/openai/openai-agents-js/blob/main/packages/agents/CHANGELOG.md)
- [OpenAI Agents SDK TypeScript inspected source snapshot](https://github.com/openai/openai-agents-js/tree/8e862b3380a577df1315bef17f351c1b58c2938b)

## Packet limitations

This research is current only at the stated cutoff. Beta and experimental APIs can change without stable migration guarantees. Pricing, quotas, model availability, and default models are intentionally not frozen here. Operational recommendations are derived from official contracts and general production reasoning; they require validation in the target environment.


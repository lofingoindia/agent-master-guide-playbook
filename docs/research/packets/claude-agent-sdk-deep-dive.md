# Claude Agent SDK and Managed Agents Deep-Dive Research Packet

Research completed: **2026-08-31**  
Scope: **Claude Agent SDK, Claude Code-derived harness behavior exposed through it, Anthropic API client SDK boundary, and Claude Managed Agents**  
Output area: [Claude Agent SDK and Anthropic Agent Ecosystem](../../frameworks/claude-agent-sdk/README.md)  
Evidence preference: **current official documentation and repositories first; bounded GitHub issues only for unresolved implementation gaps**

## Research objective

The research answered five production questions:

1. What is the actual runtime and ownership boundary of each Anthropic agent surface?
2. Which behaviors come from the Agent SDK wrapper versus the bundled Claude Code harness?
3. How do sessions, permissions, hooks, tools, streaming, delegation, and retries fail in production?
4. What changes when moving to Claude Managed Agents?
5. Which current claims are stable enough to treat as architecture, and which require version-triggered refresh?

The resulting guide deliberately keeps four concepts separate:

- **Anthropic client SDKs**: general API clients for Messages and other Claude Platform APIs;
- **Claude Agent SDK**: Python/TypeScript wrappers around a bundled Claude Code-derived child runtime;
- **Claude Code**: the end-user coding-agent product and harness from which many SDK behaviors derive;
- **Claude Managed Agents**: a separate beta hosted platform with agents, environments, sessions, threads, and persisted events.

## Method

Research proceeded from the official documentation indexes into individual reference pages, then cross-checked high-impact claims against:

- Agent SDK Python and TypeScript references;
- official package repositories, changelogs, and releases;
- Claude Code hooks, sandbox, model, and prompt-caching documentation;
- Managed Agents lifecycle, event, environment, security, multiagent, and retention documentation;
- Anthropic API client retry/error references;
- a small set of repository issues where documentation and recovery behavior remain in tension.

The goal was synthesis, not copying. Claims were included when they affected ownership, correctness, security, durability, scale, cost, or migration. Marketing descriptions without an operational contract were excluded.

### Research themes

The primary queries and document traversals covered:

- Agent SDK process and subprocess topology;
- bundled CLI versioning and package compatibility;
- workspace and settings discovery;
- system-prompt differences between SDK and `claude -p`;
- loop messages, turn accounting, tool parallelism, and result subtypes;
- automatic compaction and prompt caching;
- session continue/resume/fork and external SessionStore;
- file checkpointing coverage and security fixes;
- streaming input/output, partial events, interruption, and cleanup;
- permissions evaluation, modes, callbacks, hooks, and deferred tools;
- MCP transport, readiness, OAuth, and Tool Search;
- skills, plugins, and subagent isolation;
- hosting, timeouts, retries, resources, and OpenTelemetry;
- Managed Agents resources, events, budgets, sandbox modes, vaults, memory, skills, multiagent threads, webhooks, and retention;
- migrations among direct API loops, Claude Code, Agent SDK, and Managed Agents.

## Dated version snapshot

GitHub’s releases API was queried on 2026-08-31.

| Package | Latest release | Published | Evidence |
|---|---|---|---|
| `@anthropic-ai/claude-agent-sdk` | `v0.3.251` | 2026-08-28 | [TypeScript release](https://github.com/anthropics/claude-agent-sdk-typescript/releases/tag/v0.3.251) |
| `claude-agent-sdk` | `v0.2.148` | 2026-08-28 | [Python release](https://github.com/anthropics/claude-agent-sdk-python/releases/tag/v0.2.148) |

Both packages are pre-1.0. The package version is not the whole execution fingerprint because each package bundles a Claude Code executable. Production runs should record both the wrapper and child runtime versions.

Current Managed Agents requests use the `managed-agents-2026-04-01` beta header. Memory endpoints use `agent-memory-2026-07-22`.

## Executive synthesis

### 1. Agent SDK is a child-process harness

The Agent SDK does not merely wrap Messages API calls. A query launches a bundled Claude Code executable and communicates over standard I/O. The child owns the live model/tool loop, settings discovery, permissions, hooks, transcripts, compaction, and subagent behavior.

Consequences:

- one concurrent session normally means one process or process tree;
- package upgrades can change harness behavior even when application-facing types are stable;
- stdout is protocol-sensitive;
- the application still owns process supervision, wall-clock deadlines, isolation, durable job state, and effect safety.

### 2. Workspace and conversation are independent state

Session resume restores conversation state. It does not reconstruct the checkout, Bash-created files, packages, environment, authorization state, or external mutations. Cross-host recovery therefore needs both transcript persistence and an independently verifiable workspace/artifact strategy.

### 3. Permission policy is not isolation

The documented evaluation order—hooks, deny, ask, mode, allow, callback—is precise but easy to misread. `allowedTools` auto-approves matches; it is not a universal strict allowlist. `bypassPermissions` can run tools not listed there. Hooks can govern calls, but timeout, concurrency, resume, and version behavior prevent a single hook from being the only security boundary.

Isolation and tool-side authorization remain mandatory.

### 4. “Result” and “success” are different

The loop returns explicit result subtypes for success, maximum turns, budget, execution failure, and structured-output failure. A Result can be followed by trailing lifecycle messages. A crash can produce incomplete or zero cost fields. Consumers must classify the subtype and drain the stream.

### 5. Compaction is normal and lossy

Automatic compaction summarizes older history near the context limit. It emits a compact-boundary event but cannot guarantee exact preservation of early instructions or evidence. Stable rules belong in system/project instructions; business state and effect receipts belong outside the transcript.

### 6. SessionStore improves portability but not transactional durability

The runtime writes local transcript state then mirrors to the application store. After final mirror failure, a batch can be dropped while the query continues. A timeout is treated as ambiguous and is not retried. In a resumed-from-store temporary environment, a dropped batch may have no surviving local copy after exit.

SessionStore needs conformance tests, UUID deduplication, mirror-error monitoring, retention controls, and a product-level durability definition.

### 7. Human approval has two very different modes

`canUseTool` is a live callback and can wait indefinitely. It works for short connected interactions but is tied to the worker. Deferred tools can exit and resume later, but have batching, tool-availability, and version-specific hook limitations. Durable approval should be an application record bound to exact normalized input, with commit-time reauthorization inside the tool.

### 8. Retry defaults can exceed service deadlines

The current Agent SDK reference documents a 10-minute per-attempt API timeout and 10 retries by default. One model request can therefore occupy a process for roughly 110 minutes plus backoff in the worst case. Background-agent stall detection also resets on activity and is not a total deadline.

Every service needs an application wall-clock deadline and process-tree cancellation.

### 9. Managed Agents changes operational ownership

Managed Agents persists sessions/events and orchestrates cloud or self-hosted sandboxes. It adds versioned agent definitions, environments, budgets, webhooks, vaults, memory, and server-side multiagent threads. Its beta API, retention, sandbox expiry, retry, and event-processing semantics are different from Agent SDK sessions.

Migration is an architecture change, not an import change.

### 10. Managed Agents self-hosted mode is not a local-only data plane

Tools and filesystem/network access run on your worker, but model and tool inputs/outputs still pass through Anthropic. Skills and memory are stored by Anthropic and synchronized. The current product is stateful, is not ZDR eligible, and is not covered by Anthropic’s HIPAA BAA.

## Evidence map

| Claim area | Strongest primary sources | Guide |
|---|---|---|
| Product boundaries | [Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview), [SDK/library overview](https://platform.claude.com/docs/en/cli-sdks-libraries/overview), [Managed Agents overview](https://platform.claude.com/docs/en/managed-agents/overview) | [Boundaries](../../frameworks/claude-agent-sdk/ecosystem-boundaries-and-selection.md) |
| Child process and hosting | [Hosting](https://code.claude.com/docs/en/agent-sdk/hosting), [TypeScript reference](https://code.claude.com/docs/en/agent-sdk/typescript), [Python reference](https://code.claude.com/docs/en/agent-sdk/python) | [Runtime](../../frameworks/claude-agent-sdk/runtime-process-and-workspace-architecture.md) |
| Loop/messages/results | [Agent loop](https://code.claude.com/docs/en/agent-sdk/agent-loop) | [Loop](../../frameworks/claude-agent-sdk/agent-loop-messages-instructions-and-models.md) |
| System instructions | [Modifying system prompts](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts), [Claude Code features](https://code.claude.com/docs/en/agent-sdk/claude-code-features) | [Loop](../../frameworks/claude-agent-sdk/agent-loop-messages-instructions-and-models.md) |
| Model/effort | [Agent loop](https://code.claude.com/docs/en/agent-sdk/agent-loop), [Model configuration](https://code.claude.com/docs/en/model-config) | [Loop](../../frameworks/claude-agent-sdk/agent-loop-messages-instructions-and-models.md) |
| MCP/tools | [MCP](https://code.claude.com/docs/en/agent-sdk/mcp), [Custom tools](https://code.claude.com/docs/en/agent-sdk/custom-tools), [Tool Search](https://code.claude.com/docs/en/agent-sdk/tool-search) | [Extensions](../../frameworks/claude-agent-sdk/tools-mcp-hooks-skills-and-plugins.md) |
| Hooks/defer | [SDK hooks](https://code.claude.com/docs/en/agent-sdk/hooks), [full hooks reference](https://code.claude.com/docs/en/hooks), [user input](https://code.claude.com/docs/en/agent-sdk/user-input) | [Extensions](../../frameworks/claude-agent-sdk/tools-mcp-hooks-skills-and-plugins.md) |
| Sessions | [Session management](https://code.claude.com/docs/en/agent-sdk/sessions), [SessionStore](https://code.claude.com/docs/en/agent-sdk/session-storage) | [Sessions](../../frameworks/claude-agent-sdk/sessions-context-compaction-and-state.md) |
| Checkpointing | [File checkpointing](https://code.claude.com/docs/en/agent-sdk/file-checkpointing) | [Sessions](../../frameworks/claude-agent-sdk/sessions-context-compaction-and-state.md) |
| Streaming | [Streaming input](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode), [streaming output](https://code.claude.com/docs/en/agent-sdk/streaming-output) | [Streaming](../../frameworks/claude-agent-sdk/streaming-events-and-structured-output.md) |
| Permissions/security | [Permissions](https://code.claude.com/docs/en/agent-sdk/permissions), [secure deployment](https://code.claude.com/docs/en/agent-sdk/secure-deployment) | [Security](../../frameworks/claude-agent-sdk/permissions-approvals-and-security.md) |
| Delegation | [Subagents](https://code.claude.com/docs/en/agent-sdk/subagents), [hosting limits](https://code.claude.com/docs/en/agent-sdk/hosting) | [Subagents](../../frameworks/claude-agent-sdk/subagents-delegation-and-multi-agent.md) |
| Telemetry/cost | [Observability](https://code.claude.com/docs/en/agent-sdk/observability), [cost tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking) | [Observability](../../frameworks/claude-agent-sdk/observability-testing-and-debugging.md) |
| Retry/cancellation | [TypeScript reference](https://code.claude.com/docs/en/agent-sdk/typescript), [Python reference](https://code.claude.com/docs/en/agent-sdk/python), [API errors](https://platform.claude.com/docs/en/api/errors) | [Reliability](../../frameworks/claude-agent-sdk/reliability-cancellation-retries-and-effects.md) |
| Managed sessions/events | [Sessions](https://platform.claude.com/docs/en/managed-agents/sessions), [event stream](https://platform.claude.com/docs/en/managed-agents/events-and-streaming), [operations](https://platform.claude.com/docs/en/managed-agents/session-operations) | [Managed Agents](../../frameworks/claude-agent-sdk/managed-agents-boundary-and-operations.md) |
| Managed environments/security | [Environments](https://platform.claude.com/docs/en/managed-agents/environments), [self-hosted](https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes), [self-hosted security](https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes-security) | [Managed Agents](../../frameworks/claude-agent-sdk/managed-agents-boundary-and-operations.md) |
| Managed retention | [API and data retention](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention) | [Managed Agents](../../frameworks/claude-agent-sdk/managed-agents-boundary-and-operations.md) |
| Migration | [Agent SDK migration](https://code.claude.com/docs/en/agent-sdk/migration-guide), [Managed Agents migration](https://platform.claude.com/docs/en/managed-agents/migration) | [Migrations](../../frameworks/claude-agent-sdk/migrations-limitations-and-alternatives.md) |

## Contradictions and production-relevant tensions

These are not all documentation errors. Several are differences between a convenient feature description and the stricter conclusion needed for production.

### Empty settings sources do not make a hermetic worker

The feature documentation says an empty settings-source array prevents ordinary user, project, and local settings from loading. The same documentation explains that managed settings and `~/.claude.json` are still read, automatic memory can still load, authenticated connectors may still appear, and restrictive sandbox rules can persist.

**Resolution:** describe the option as one control in a tenant-isolation design, never as “disable all ambient configuration.”

### SDK and CLI system-prompt defaults differ

The SDK uses a minimal default prompt when no system prompt is supplied, while `claude -p` uses the full Claude Code prompt.

**Resolution:** any CLI-to-SDK parity claim must specify the `claude_code` preset.

### `allowedTools` sounds like an allowlist but is an auto-approval rule

The permissions reference clarifies that `allowedTools` approves matches. In bypass mode, tools not listed can still run.

**Resolution:** for strict automation, expose only required tools and use `dontAsk` plus explicit rules and isolation.

### Result may not be the last stream message

The loop documentation says lifecycle messages can follow a Result. Several examples naturally encourage stopping once Result appears.

**Resolution:** record Result but continue consuming to stream completion.

### Session continuity is not job continuity

“Resume” can sound like restoring an execution. Official session documentation defines it as conversation continuity and separately discusses working directory/session storage.

**Resolution:** persist business state and reconstruct/verify workspace separately.

### SessionStore permits a durability gap

External storage enables cross-host resume, but documented mirror behavior can drop a failed batch and continue execution.

**Resolution:** qualify durability, monitor `mirror_error`, and refuse “durable resume confirmed” until the store acknowledges the checkpoint.

### Hook timeout behavior is not uniform

Current programmatic `PreToolUse` timeout behavior blocks the tool. Filesystem command/HTTP/MCP hook timeouts can fall through to ordinary permission handling.

**Resolution:** critical authorization belongs in both robust pre-execution policy and the tool adapter, not in a timeout-sensitive hook alone.

### Documentation says deferred resume re-fires a hook; an open Python issue reports otherwise

The hooks documentation describes resume as invoking the pending decision again. [Python issue 993](https://github.com/anthropics/claude-agent-sdk-python/issues/993) reports that a deferred tool can resume without rerunning the Python programmatic `PreToolUse` callback. It remained open as of the research date.

**Resolution:** label this language/version-specific and reauthorize inside the tool. Do not generalize the issue to filesystem hooks or future versions without testing.

### Live approvals are easy but not durable

`canUseTool` can pause indefinitely and is cancelled with the query. [Python issue 871](https://github.com/anthropics/claude-agent-sdk-python/issues/871) reports no recovery path when a pod dies during the wait.

**Resolution:** use it only for bounded connected waits; store durable approval externally.

### Hooks can run concurrently

The hooks model can look serial when read as a lifecycle. Parallel tool calls cause hook callbacks to run concurrently. [Python issue 910](https://github.com/anthropics/claude-agent-sdk-python/issues/910) raised the ambiguity and was closed as completed on 2026-05-14.

**Resolution:** keep the concurrency warning because it remains part of the documented execution model, while not presenting the closed issue as an active bug.

### File checkpointing is narrower than rollback

It tracks changes from selected file tools, not Bash, directories, or most subagent edits, and rewinds files without rewinding conversation.

**Resolution:** call it file checkpointing, never transaction rollback. Note the documented SessionStore incompatibility and linked-file security version boundary.

### Default retries conflict with typical service SLOs

The TypeScript reference documents ten API retries with a ten-minute window per attempt. General Anthropic client SDKs, by contrast, retry transient errors twice by default. These apply to different layers/products.

**Resolution:** keep the retry contracts distinct and calculate the Agent SDK child’s total wall time explicitly.

### Terminal cost is useful but not authoritative

The cost guide defines it as a client-side estimate from a bundled price table and notes crash-related gaps.

**Resolution:** use it for budgets/telemetry and reconcile billing through authoritative usage sources.

### Self-hosted Managed Agents still transfers data to Anthropic

“Self-hosted sandbox” can imply a fully local execution and data plane. The official page states that tool inputs/outputs still flow through Anthropic and that skills/memory are platform-managed.

**Resolution:** describe self-hosted as local tool/sandbox execution under an Anthropic-managed session control plane.

### Managed session history and sandbox state have different retention

History persists until deletion, while sandbox state is kept only 30 days from sandbox creation and activity does not extend the deadline.

**Resolution:** export durable outputs rather than relying on a resumed sandbox.

### Managed interrupt has no unique model stop reason

The event-stream documentation states that an interrupted turn reports `end_turn`, the same as normal completion.

**Resolution:** correlate the interrupt input event, `processed_at`, and subsequent event timeline.

### Managed agents are versioned; environments are not

Agent definitions create reproducible versions, while environment changes can affect future sessions without a platform version identifier.

**Resolution:** pin agent versions and maintain an application environment revision ledger.

## Bounded repository issue review

Repository issues were used only to identify gaps not fully resolved by official reference text. Their status was fetched from GitHub on 2026-08-31.

| Issue | Status on research date | Use in guidance |
|---|---|---|
| [Python #871: multi-pod approval recovery](https://github.com/anthropics/claude-agent-sdk-python/issues/871) | Open; last updated 2026-04-24 | Supports external durable approval design |
| [Python #993: deferred resume skips callback](https://github.com/anthropics/claude-agent-sdk-python/issues/993) | Open; last updated 2026-05-26 | Adds a pinned-version caveat; tool-side authorization remains required |
| [Python #910: concurrent hook dispatch](https://github.com/anthropics/claude-agent-sdk-python/issues/910) | Closed/completed; updated 2026-05-14 | Not treated as an open defect; retained as a concurrency design fact |
| [Python #941: reference omits public options/types](https://github.com/anthropics/claude-agent-sdk-python/issues/941) | Open; last updated 2026-05-09 | Reinforces using package types/changelog with docs; no undocumented API was recommended |
| [TypeScript #176: unstable V2 options ignored](https://github.com/anthropics/claude-agent-sdk-typescript/issues/176) | Closed/completed; updated 2026-04-14 | Excluded from current limitations; shows why unstable APIs require regression tests |

Open issues are evidence of a bounded implementation report, not proof of universal behavior. Every cited issue must be rechecked when the SDK version changes.

## Maturity assessment

| Area | Assessment | Reason |
|---|---|---|
| Core Agent SDK loop | Production-usable with pinning | Clear loop/messages/tools contract, but pre-1.0 and child-runtime coupling |
| Permissions and hooks | Production-usable with defense in depth | Detailed rules; callback durability and version-sensitive hook behavior require external controls |
| Local sessions | Mature for affinity/local use | Clear resume/fork behavior |
| External SessionStore | Emerging production surface | Conformance support exists; mirroring and adapter ownership create durability risks |
| Streaming | Production-usable | Requires disciplined drain, backpressure, and protocol adaptation |
| Subagents | Production-usable when bounded | Useful isolation; fan-out, resource, and deadline controls are external |
| File checkpointing | Specialized | Narrow coverage and SessionStore incompatibility |
| Agent SDK hosting | Production-usable with platform engineering | Process/tree, isolation, resource, and deadline ownership stays with operator |
| Managed Agents | Beta | Rich hosted lifecycle, but beta contract, retention, and evolving limits |
| Managed self-hosted sandbox | Beta/specialized | Strong local tool-placement option, not a local-only data plane |

## Claims intentionally not generalized

The guide does not:

- assert undocumented exactly-once semantics for tools, events, or SessionStore;
- treat current model aliases or effort defaults as timeless;
- assume TypeScript and Python parity;
- claim a hook matcher or timeout behaves identically across callback and filesystem implementations;
- claim Managed Agents cloud limits or rate limits are permanent;
- recommend undocumented V2/unstable APIs;
- infer pricing from cost-estimate fields;
- describe containers as a complete hostile-code boundary;
- claim self-hosting changes Anthropic’s Managed Agents retention eligibility.

## Refresh plan

### Immediate refresh triggers

- TypeScript or Python Agent SDK minor release;
- bundled Claude Code behavior change affecting permissions, hooks, sessions, or subagents;
- closure/reproduction change for Python issues 871 or 993;
- Managed Agents beta header change or stable release;
- retention/ZDR/BAA eligibility change;
- SessionStore mirror or checkpoint compatibility change.

### Quarterly checks

- latest package and bundled runtime versions;
- official changelogs and breaking changes;
- model/effort/default behavior;
- tool names and message unions;
- subagent limits and background timeout behavior;
- Managed sandbox limits, rate limits, event types, and retention;
- self-hosted security/data-flow documentation.

### Incident-driven checks

Any incident involving duplicate effects, lost transcript entries, approval recovery, incorrect permission decisions, child-process leakage, compaction loss, or managed event reordering should trigger a targeted documentation refresh with a reproducible test.

## Primary source register

### Agent SDK core

- [Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview)
- [How the agent loop works](https://code.claude.com/docs/en/agent-sdk/agent-loop)
- [TypeScript SDK reference](https://code.claude.com/docs/en/agent-sdk/typescript)
- [Python SDK reference](https://code.claude.com/docs/en/agent-sdk/python)
- [TypeScript repository](https://github.com/anthropics/claude-agent-sdk-typescript)
- [Python repository](https://github.com/anthropics/claude-agent-sdk-python)

### Prompt, settings, and extensions

- [Modifying system prompts](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts)
- [Claude Code features in the SDK](https://code.claude.com/docs/en/agent-sdk/claude-code-features)
- [MCP](https://code.claude.com/docs/en/agent-sdk/mcp)
- [Custom tools](https://code.claude.com/docs/en/agent-sdk/custom-tools)
- [Tool Search](https://code.claude.com/docs/en/agent-sdk/tool-search)
- [Hooks in the SDK](https://code.claude.com/docs/en/agent-sdk/hooks)
- [Full hooks reference](https://code.claude.com/docs/en/hooks)
- [Skills](https://code.claude.com/docs/en/agent-sdk/skills)
- [Plugins](https://code.claude.com/docs/en/agent-sdk/plugins)

### State, streaming, and delegation

- [Session management](https://code.claude.com/docs/en/agent-sdk/sessions)
- [External session storage](https://code.claude.com/docs/en/agent-sdk/session-storage)
- [File checkpointing](https://code.claude.com/docs/en/agent-sdk/file-checkpointing)
- [Streaming input](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode)
- [Streaming output](https://code.claude.com/docs/en/agent-sdk/streaming-output)
- [Subagents](https://code.claude.com/docs/en/agent-sdk/subagents)

### Production operations

- [Permissions](https://code.claude.com/docs/en/agent-sdk/permissions)
- [User input and approvals](https://code.claude.com/docs/en/agent-sdk/user-input)
- [Hosting](https://code.claude.com/docs/en/agent-sdk/hosting)
- [Secure deployment](https://code.claude.com/docs/en/agent-sdk/secure-deployment)
- [Observability](https://code.claude.com/docs/en/agent-sdk/observability)
- [Cost tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking)
- [Troubleshooting](https://code.claude.com/docs/en/agent-sdk/troubleshooting)
- [Agent SDK migration guide](https://code.claude.com/docs/en/agent-sdk/migration-guide)

### Claude API clients and errors

- [CLI, SDKs, and libraries](https://platform.claude.com/docs/en/cli-sdks-libraries/overview)
- [Claude API errors](https://platform.claude.com/docs/en/api/errors)
- [Python client SDK retries and timeouts](https://platform.claude.com/docs/en/api/sdks/python)

### Managed Agents

- [Managed Agents overview](https://platform.claude.com/docs/en/managed-agents/overview)
- [Agent setup](https://platform.claude.com/docs/en/managed-agents/agent-setup)
- [Start a session](https://platform.claude.com/docs/en/managed-agents/sessions)
- [Session event stream](https://platform.claude.com/docs/en/managed-agents/events-and-streaming)
- [Session operations](https://platform.claude.com/docs/en/managed-agents/session-operations)
- [Environments](https://platform.claude.com/docs/en/managed-agents/environments)
- [Cloud sandbox reference](https://platform.claude.com/docs/en/managed-agents/cloud-sandboxes-reference)
- [Self-hosted sandboxes](https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes)
- [Self-hosted sandbox security](https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes-security)
- [Tools](https://platform.claude.com/docs/en/managed-agents/tools)
- [Permission policies](https://platform.claude.com/docs/en/managed-agents/permission-policies)
- [MCP connector](https://platform.claude.com/docs/en/managed-agents/mcp-connector)
- [Vaults](https://platform.claude.com/docs/en/managed-agents/vaults)
- [Memory](https://platform.claude.com/docs/en/managed-agents/memory)
- [Skills](https://platform.claude.com/docs/en/managed-agents/skills)
- [Multiagent orchestration](https://platform.claude.com/docs/en/managed-agents/multiagent-orchestration)
- [Webhooks](https://platform.claude.com/docs/en/managed-agents/webhooks)
- [Managed Agents migration](https://platform.claude.com/docs/en/managed-agents/migration)
- [API and data retention](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention)

## Research conclusion

The strongest practical architecture is not “maximum agent capability.” It is the smallest surface whose ownership model matches the job:

- direct client SDK for bounded model/API workflows;
- Agent SDK for a self-hosted Claude Code-style workspace agent;
- Claude Code for a human-operated coding surface;
- Managed Agents for teams that deliberately accept the hosted beta session and retention contract.

Across all four, the invariant remains the same: the model may propose work, but the application must own identity, authorization, durable workflow state, deadlines, effect idempotency, and evidence.

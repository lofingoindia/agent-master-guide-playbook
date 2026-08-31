# Research Packet: DeepSeek Harness deep dive

> **Status:** source-verified synthesis for a fast-moving developer preview  
> **Research date:** 2026-08-31  
> **Repository baseline:** `deepseek-ai/deepseek-harness` `0.1.2-alpha.2`, commit `0a53fb55bea101816fa226bb964ae2bed71c343b`, committed 2026-08-30  
> **Scope:** official site, repository, source, architecture, safety, profiles/presets, loop, events/persistence, plugins/tools/skills, compaction, subagents/workflows, sandbox/approval/web, model adapters, telemetry/testing, applications, releases, and bounded GitHub Discussion evidence  
> **Method:** official documentation and current source were treated as authoritative for current behavior. Release notes establish evolution. Discussions are version-scoped failure reports or test leads, never production guarantees. No deployment claim was inferred from architectural intent.

## Research questions

1. What is Cordis, and where does the Harness begin and end?
2. How are profiles, presets, plugins, tools, skills, and applications composed?
3. What exactly is durable in the session/event model, and what remains process-local?
4. Which controls are policy fences, and which mechanisms are actual security boundaries?
5. How do provider/model routes, streaming, tool schemas, and context management fail?
6. What operational and adoption conclusions are justified by the developer-preview evidence?

## Executive findings

1. **DeepSeek Harness is a plugin-composed runtime, not a monolithic agent loop.** Cordis services, typed events, reversible effects, and loader reconciliation underpin replaceable model, tool, session, persistence, loop, sandbox, and application services.
2. **The append-only session log is the conversational source of truth.** Model history is a derived surface. Compaction replaces the surface view while retaining canonical events.
3. **Durability is strong within one process/owner but not distributed.** The current source coordinates local appends and flushes and repairs incomplete turns. It does not document a cross-process writer lease, durable subagent mailbox, or transaction with external effects.
4. **The security perimeter must be outside the Harness process.** Cordis scopes, worker threads, tool approval, and filesystem modes are useful controls but do not contain hostile same-process plugins, PTC/workflow code, ambient secrets, or unrestricted egress.
5. **Provider compatibility is empirical.** Adapters own stream assembly, tool arguments, usage, replay state, errors, and optional features. “OpenAI-compatible” does not prove semantic compatibility.
6. **The release line is too volatile for an implicit production recommendation.** Compatibility breaks are guaranteed, all releases examined are prereleases, the tag sequence moved from RC to alpha, SQLite compatibility has already changed, and the project explicitly disclaims security audit/production readiness.

The synthesized guide set begins at [DeepSeek Harness: production-minded guide to a developer preview](../../frameworks/deepseek-harness/README.md).

## Source snapshot and confidence

| Surface | Current finding | Confidence | Refresh trigger |
|---|---|---|---|
| Maturity | README says developer preview and guarantees compatibility-breaking changes | High | Any README/release posture change |
| Safety | `SAFETY.md` says experimental, not audited, not production-ready; recommends disposable isolation | High | Any safety/audit statement |
| Architecture | Cordis plugins provide every major service, including loop and persistence | High | Core/architecture/Cordis change |
| Agent loop | Turn contains zero or more model/tool steps; event sequence is documented and sourced | High | Agent-loop event or state-machine change |
| Session model | Append-only lossless-JSON events; model history derived from surface operations | High | Session/event schema change |
| Persistence | Durable append/flush, checkpoint policy, repair, JSONL/Zstd and opt-in SQLite | High | Backend/format/repair change |
| Cross-process ownership | No documented/current-source writer lease found; older reports show concurrent writer failures | Medium-high | Lock/lease/coordinator implementation |
| Plugins | Same-process npm/Cordis code; registration effects unload; scopes are not containment | High | Plugin runner/loader trust change |
| Tools | Policy/approval/execute/result pipeline; typed input/output validation; PTC re-enters it | High | Tool schema/pipeline change |
| Skills/instructions | Layered discovery and durable prompt snapshots; prompt authority only | High | Discovery/precedence/watcher change |
| Compaction | Optional summary/prune seam, canonical originals retained, heuristic pressure handling | High | Provider/threshold/surface change |
| Subagents | Multiple providers; durable child sessions but process-local activations/ownership | High | Provider or continuation change |
| Workflow | Foreground model-authored JS orchestration without durable journal/resume | High | Workflow engine change |
| Sandbox | Filesystem-mode vocabulary with platform runners; not general network/process containment | High | SandboxMode/platform runner change |
| Web auth | Shipped web CLI is loopback-only with launch-token/cookie auth in alpha.1+ | High | Carrier/auth/bind change |
| Models | Provider-routed adapters own conversion/stream/replay/errors; conformance varies | High | LLM contract/adapter change |
| Telemetry | Best-effort event-derived ledger and optional OTel export; content can be sensitive | High | Ledger/redaction/export change |
| Applications | web, headless, SDK, SDK-minimal, ACP profiles with different contracts | High | Profile/application change |
| Production readiness | No official production/HA/support guarantee found | High | Stable/support/HA announcement |

## Research chronology

### Official product surface

The [official site](https://www.deepseek.com/harness/en/) presents “everything is a plugin” and lists models, tools, skills, sessions, sandboxes, storage, loops, scheduling, and UI as composable capabilities. It presents Standard, PTC, Minimal, and Creator modes. Repository source names the Creator preset `cordis`.

The site also describes an append-only event log. Source inspection was required to separate that log from the model-visible history surface and from live agent events.

### Repository and release baseline

The official repository was examined at:

```text
version: 0.1.2-alpha.2
commit: 0a53fb55bea101816fa226bb964ae2bed71c343b
commit date: 2026-08-30T21:37:53+08:00
Node engines: ^22.19.0 || >=24.0.0
package manager: pnpm 11.7.0
```

All public GitHub releases checked through `0.1.2-alpha.2` were marked prerelease.

### Evidence precedence

When sources disagreed, this packet used:

1. current source and generated current documentation;
2. current official safety/architecture/README;
3. release notes for dated changes;
4. official paper for Cordis design intent;
5. GitHub Discussions as release-bounded failure evidence.

A Discussion can show that a failure occurred in its stated environment. It cannot establish that the current release still fails unless current source or a current reproduction supports it.

## Architecture evidence

### Cordis layer

Official [architecture documentation](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md) and the [Cordis paper](https://arxiv.org/abs/2608.25512) establish:

- a `Context` resolves services and dependencies;
- events have multiple dispatch semantics;
- plugin registrations are tracked as reversible effects;
- dependency/configuration changes can activate, dispose, and reconcile runtime fibers;
- Harness vendors Cordis packages under the DeepSeek namespace.

Inference boundary: reversible effects improve lifecycle management; they do not guarantee an unload always occurs after abrupt process failure and do not provide OS isolation.

### Profile and preset split

Current configuration evidence shows:

- process profiles: `web`, `headless`, `sdk`, `sdk-minimal`, `acp`;
- profile layering: bundles → profile patch → home patch → ordered CLI patches;
- live reload by default for web; startup-only for shipped headless/SDK/minimal/ACP paths;
- per-agent presets: `standard`, `ptc`, `minimal`, `cordis`;
- patch row configuration replacement rather than assumed recursive merge;
- effective configuration inspection through `--dump-config` and `--dump-default-config`.

This distinction is important because a profile chooses the application/control plane, while a preset chooses one agent's standing prompt/tools/policies inside it.

### Exact loop

Current agent-loop docs/source support this sequence:

```text
turn/start
  claim next turn input and one queued message
  for each step:
    assemble prompt/tools/context
    agent/pre-step
    step/start
    user/message when applicable
    derive history from session surface
    request/header and request/context
    agent/request → llm/stream
    assistant/chunk* → assistant/message
    tool/call → policy/approval/execute → tool/result
    step/end
turn/end
```

A step is one model request plus tool execution. A turn can have zero or more steps. Live agent events coordinate runtime behavior; session events carry durable history.

## Session and persistence evidence

### Canonical event contract

The session core validates lossless JSON, contiguous sequence numbers, and freezes appended events. Core events cover turn/step boundaries, user/assistant content, raw chunks, tool calls/results, request header/context, and fork seed boundaries. Plugins can extend the event map.

`request/header` records effective provider/model/reasoning, system prompt, and schemas. `request/context` records dynamic model-visible context. This is both a diagnostic strength and a sensitive-data risk.

### Surface projection

Model history is derived from surface operations, not stored independently. Compaction appends summary-related events and a replacement surface message; original events remain canonical.

### Persistence mechanics

Current [session persistence documentation](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/session/session-persistence/README.md) supports:

- synchronous session event emission copied into bounded async batches;
- append completion after backend durability;
- a flush-to-quiescence operation;
- checkpoints before model requests, top-level side-effect-capable tools, and subsequent steps;
- JSONL/raw or checksummed concatenated Zstandard frames per session;
- opt-in SQLite packed event records in one database;
- format-version and required/ignorable event handling;
- refusal of unsupported direction/version rather than silent interpretation;
- torn-tail handling and final-turn repair.

Crash repair can synthesize `TOOL_NOT_STARTED` or `TOOL_OUTCOME_UNKNOWN` and close incomplete step/turn structure. It does not resume a partial turn or undo an external effect.

### Ownership limit

Source searches at the pinned commit found in-process sequence/persistence coordination but no documented cross-process file lock, lease, or fencing mechanism. Reports [#420](https://github.com/deepseek-ai/deepseek-harness/discussions/420), [#2254](https://github.com/deepseek-ai/deepseek-harness/discussions/2254), and [#4506](https://github.com/deepseek-ai/deepseek-harness/discussions/4506) describe duplicate sequence/corruption with concurrent RC-era processes.

Conclusion: recommend one writer per session/root unless an external coordinator provides exclusive ownership. Confidence is medium-high because absence claims can change quickly; refresh on any persistence coordinator change.

### Format volatility

Release `0.1.0-rc.8` explicitly notes incompatible SQLite data. No general migration guarantee was found. Back up, pin the reader, and export domain artifacts before upgrades.

### Optional uploads

Release/source evidence for the official DeepSeek adapter in `0.1.2-alpha.1` shows:

- active plugin package name/version metadata enabled by default;
- opt-in incremental canonical Session-log upload;
- first full log followed by suffixes;
- a local delivery-accepted watermark after HTTP success;
- at-least-once behavior with possible duplicates;
- no general redaction of the canonical session payload;
- destination follows the configured base URL/gateway.

These paths are model-hidden transport metadata, not proof of privacy or exactly-once storage.

## Tools, plugins, skills, and instructions evidence

### Tool pipeline

Current tool docs/source show:

```text
model call
→ durable tool/call
→ tools/pre-execute policy/sandbox/hooks
→ optional one-shot approval
→ tools/execute wrapper
→ tool body and filesystem gate
→ tools/post-execute
→ output validation/finalization/presentation
→ durable tool/result
```

Arguments for typed tools are detached, validated, and frozen. Output is validated against a lossless-JSON schema. UI presentation callbacks are intended to be pure. The cancellation signal is mandatory operational state. PTC nested calls re-enter this pipeline.

### Schema boundary

The current unified schema DSL supports JSON primitives, arrays, objects, `json`, and exact-one `oneOf`. Defaults are not applied; the implicit parameter root is open; raw-schema tools own input validation. Unsupported schema features are rejected.

Discussion [#4747](https://github.com/deepseek-ai/deepseek-harness/discussions/4747) reports `0.1.1-rc.2` nested object values arriving as JSON strings on one model route. Current parsing correctly parses the top-level JSON once, and validation correctly rejects a string where an object was required. This is provider conformance evidence, not justification for global coercion.

### PTC boundary

The current PTC backend runs TypeScript in a worker thread with heap/time/output limits and an empty initial environment. Official docs explicitly state that this is containment, not a security boundary, and that PTC is shell-equivalent trust. Spawned processes can outlive worker termination. Do not treat the worker as hostile-code isolation.

### Plugin boundary

Plugins and bundles install npm code and execute in the host process. Cordis effects can unload registrations; service scopes do not contain Node/process authority. pnpm build-script approval reduces accidental execution but remains a supply-chain trust decision.

### Skill precedence

Current local provider ranks:

| Rank | Source |
|---:|---|
| 100 | project `.dsh/skills` |
| 200 | project `.agents/skills` |
| 300 | configured custom source |
| 400 | user DSH source |
| 500 | user agents source |
| 600 | bundled source |

Nearest scope layer wins first; rank resolves names within it. Catalog bodies are loaded on demand. Incomplete scans preserve the last complete snapshot. Skills are prompt authority, not executable authorization.

### Workspace instructions

Harness combines home and project `AGENTS.md` chains broad-to-specific and injects snapshots durably as user-role content. Current first-party tool touches can refresh it, but there is no general watcher for every external change. Symlinked instruction files can cross a trust boundary. Security rules must be enforced in policy/tool code rather than prompt text.

## Context and compaction evidence

Current compaction source supports:

- an optional `ctx.compaction` seam and basic provider;
- pre-step pressure checks and request-error overflow recovery;
- optional deterministic old-tool-result pruning before summarization;
- remeasurement after pruning;
- a separate model summary call with logged raw summary and usage;
- log bracket events plus a replacement `user/message` surface operation;
- preservation of tool-call/result pairing, but not necessarily whole turns;
- original canonical events retained;
- heuristic generic token estimation and adapter-owned overflow classification.

Important limitations:

- default fractions are implementation heuristics, not workload-proven universal tuning;
- a character heuristic can undercount CJK, code, or JSON;
- oversized envelope/indivisible nodes may not be compactable;
- summary failure can leave the original pressure unresolved;
- summary output itself can truncate;
- repeated compaction needs a bounded fallback.

Reports [#3565](https://github.com/deepseek-ai/deepseek-harness/discussions/3565) (stale route on resumed/manual compaction) and [#4776](https://github.com/deepseek-ai/deepseek-harness/discussions/4776) (repeated compaction of incompressible state) are version-scoped adoption-test inputs.

Spilled oversized tool output is stored in private local files with a locator/preview. No complete artifact retention/fork-transfer service was found. Treat spills as cache, not durable domain storage.

## Subagent and orchestration evidence

### Provider matrix

Current source contains fresh spawn, completed-prefix fork, ACP, Codex, Claude Code, and Harness SDK providers. Capabilities are discovered at start and unsupported requested features should reject.

- Spawn: fresh transcript, potentially shared in-process cwd/services.
- Fork: one-time completed-prefix snapshot, not a live branch.
- ACP/external: fresh subprocess/protocol boundary, not automatically an OS sandbox.
- One-shot versus continuable behavior varies by provider.

### Continuable child semantics

A durable child Session may be cold-resumed, but at most one process-local Activation owns current execution. The manager owns FIFO input, ancestry, and activation. Once admitted, a follow-up is not owned by the caller's cancellation. Interrupt preserves the inbox. Crash can lose accepted-but-not-yet-durably-recorded prompts.

No cross-process durable mailbox/lease or universal durable child-report channel was found. Application-level task IDs, structured results, leases, and reconciliation remain necessary.

### Workflow tool

The workflow tool exposes model-authored JavaScript helpers for agent, parallel, pipeline, phases, and logging. It is foreground and parent-blocking. Current source does not establish a durable workflow journal, partial resume, saved/nested workflows, or a universal token budget. Worker isolation is not a security boundary.

### Experimental teams

Agent Teams source exists under `packages/experimental`, is excluded from the official released package set examined, and carries no stability promise. It explores durable roster/mail/task records but shares process/checkout and lacks cross-process exactly-once guarantees. This is research evidence only.

## Safety evidence

### Official position

`SAFETY.md` governs all safety conclusions:

- experimental and not audited;
- must not be treated as secure or production-ready;
- model code/commands, plugins, processes, network, credentials, and files create risk;
- sandbox/approval/permission reduce risk but are no guarantee;
- use disposable VM/container/dedicated environments, least privilege, backups, and plugin review.

### Sandbox vocabulary

Current `SandboxMode` is filesystem-oriented: read-only, workspace-write, danger-full-access. It does not express a general network or process policy. Current platform paths include Bubblewrap/Landlock, Seatbelt, and Windows restricted-token/ACL mechanisms. Coverage depends on platform/version/configuration. A custom runner is operator-supplied trust.

### Approval

Current approval can allow once, reject, cancel, or be unavailable. Missing answerer fails closed. `never` rejects approval-required calls; it does not auto-approve. Approval binds to call identity, while UI clarity about normalized arguments/targets remains an application concern.

### Web authentication evolution

RC-era reports [#817](https://github.com/deepseek-ai/deepseek-harness/discussions/817) and [#454](https://github.com/deepseek-ai/deepseek-harness/discussions/454) described unauthenticated/local RPC and dynamic/plugin risks. `0.1.2-alpha.1` added launch-token/cookie auth, and current shipped `dsh web` rejects broad `0.0.0.0` binding. Current behavior is materially narrower than the historical reports.

The shipped local cookie is not a multi-tenant internet identity system. A custom carrier owns TLS/authn/authz/origin/CSRF/rate limits.

### SSRF and network

`0.1.2-alpha.1` added public-only WebFetch with SSRF protection. That does not cover arbitrary shell, plugin, MCP, or custom network clients. Egress still requires an outer policy.

### Historical security reports

- [#817](https://github.com/deepseek-ai/deepseek-harness/discussions/817): RC-era security audit themes; web auth later changed.
- [#454](https://github.com/deepseek-ai/deepseek-harness/discussions/454): RC-era plugin/dynamic trust analysis.
- [#962](https://github.com/deepseek-ai/deepseek-harness/discussions/962): ambient credential/read/network exfiltration paths; exact mechanisms may change, architectural ambient-authority risk remains.
- [#4514](https://github.com/deepseek-ai/deepseek-harness/discussions/4514): version-scoped web/SSRF/path/timeout review; alpha.1 release claims SSRF mitigation.
- Release `0.1.0-rc.1`: fixed a Bubblewrap `/proc/<pid>/root` escape.

These are regression inputs. They are not all asserted as current defects.

## Provider/model evidence

`ctx.llm` is a provider-routed registry. Provider identifies the adapter; model remains adapter-owned. Requests carry ordered messages, system, tools, optional reasoning/temperature/tokens/stop, abort signal, session ID, and auxiliary purpose. Adapters own conversion, stream assembly, usage/finish ordering, retry/error classification, replay state, and optional images/reasoning.

Current adapters include a direct DeepSeek HTTP/SSE implementation and a broader `pi-ai` path for catalogs/custom providers. Custom OpenAI-compatible configuration exposes compatibility switches, but route behavior must be tested.

Failure fixtures from Discussions:

- [#4747](https://github.com/deepseek-ai/deepseek-harness/discussions/4747): nested structured argument stringification;
- [#4427](https://github.com/deepseek-ai/deepseek-harness/discussions/4427): provider stream index reuse fusing parallel calls;
- [#805](https://github.com/deepseek-ai/deepseek-harness/discussions/805): gateway-specific nonstandard fragment interaction corrupting tool args.

The correct response is route-specific conformance, raw-fragment evidence, and fail-closed validation—not assuming all providers share one stream dialect.

## Testing and observability evidence

Official [testing guidance](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/testing.md) combines unit coverage, real API E2E, expected output, session snapshots, and browser snapshots. Per-file 100% coverage is a floor, not a semantic guarantee.

Official postmortems establish four concrete testing lessons:

1. boot real built packages through the Loader to catch export/injection composition failures;
2. add semantic assertions so snapshots cannot bless `UNKNOWN_TOOL` or other error-shaped behavior;
3. verify the exact current browser origin/world, not a replacement server or HTTP 200;
4. parse structured child-process failure rather than matching benign warning text.

Session telemetry is an optional event-derived ledger. Delivery is best effort and can duplicate; `(session.id, event.seq)` is the natural dedupe key. Ledger sequence gaps are normal because not every raw event is exported identically. OTel export can include sensitive event content; current base behavior must not be treated as universally redacted.

## Application and operations evidence

### Headless

One-shot, no server; final text stdout, reasoning/progress stderr; waits idle and flushes; coarse completed/non-completed exit status. Domain outcome still needs independent verification.

### SDK

JSON-RPC over stdio; concurrent calls may enqueue. The basic server keeps agents until shutdown and does not provide a universal prompt result/cancel/close model. Stdout must remain protocol-pure.

### ACP

Trusted stdio controller; multi-session list/resume/close/cancel and model/MCP/permission controls. One prompt per session at a time. Feature parity with web/session internals is not assumed. MCP commands are executable configuration.

### Web

Loopback local application with token/cookie authentication and live reload. Not an HA or tenant service.

### Shutdown

CLI disposal handles normal signals with a bounded graceful period; repeated signal forces exit. This improves cleanup but cannot guarantee external effect rollback or persistence if the deadline expires.

### Operational conclusion

If used beyond experiments, place a pinned single-owner Harness worker behind:

- a durable task queue and lease/fencing token;
- a disposable workspace and OS isolation;
- an effect/credential broker with idempotency;
- durable artifact/domain storage;
- egress control;
- redacted observability;
- backup/restore and version rollback.

This is a containment pattern, not an assertion that the current upstream is production-ready.

## Release ledger

| Release | Evidence used |
|---|---|
| `0.1.0-rc.7` | plugin settings, Codex/Claude subagents, persistent attachments, history/tool error fixes |
| `0.1.0-rc.8` | multimodal/persistent PowerShell, external subagents, custom provider work, fork performance, incompatible SQLite format |
| `0.1.1-rc.1` | Bubblewrap escape fix |
| `0.1.1-rc.2` | image/compatibility evolution |
| `0.1.2-alpha.1` | provider/subagent route controls, Windows Python, ACP controls, auth token, WebFetch SSRF, DeepSeek metadata/log upload |
| `0.1.2-alpha.2` | connection/retry UX, schedules, grouping/preset switching, history efficiency, stats, ignorable event restoration |

Primary release index: [GitHub Releases](https://github.com/deepseek-ai/deepseek-harness/releases).

## Discussion evidence ledger

| Discussion | Reported version/date | Use in guide | Current-status treatment |
|---|---|---|---|
| [#420](https://github.com/deepseek-ai/deepseek-harness/discussions/420) | RC-era | Multi-writer persistence test | Historical failure; current source still lacks documented cross-process lease |
| [#454](https://github.com/deepseek-ai/deepseek-harness/discussions/454) | RC-era | Plugin/dynamic trust regression set | Historical; current safety docs now explicit |
| [#805](https://github.com/deepseek-ai/deepseek-harness/discussions/805) | 2026-08-14 route-specific | Tool-argument stream fixture | Version/gateway-scoped only |
| [#817](https://github.com/deepseek-ai/deepseek-harness/discussions/817) | RC-era | Web/plugin security regression set | Web auth materially changed in alpha.1 |
| [#962](https://github.com/deepseek-ai/deepseek-harness/discussions/962) | RC-era | Ambient credential/read/egress threat model | Exact path volatile; architectural risk retained |
| [#1415](https://github.com/deepseek-ai/deepseek-harness/discussions/1415) | `0.1.0-rc.6` | Duplicate preset/provider mount test | Pinned current source still throws/registers same duplicate path |
| [#2254](https://github.com/deepseek-ai/deepseek-harness/discussions/2254) | RC-era | Multi-process writer test | Historical failure; one-writer rule retained |
| [#3245](https://github.com/deepseek-ai/deepseek-harness/discussions/3245) | RC-era | PTC trust test | Current official docs explicitly say not a security boundary |
| [#3565](https://github.com/deepseek-ai/deepseek-harness/discussions/3565) | RC-era | Resume/route/compaction test | Version-scoped; no blanket current claim |
| [#4427](https://github.com/deepseek-ai/deepseek-harness/discussions/4427) | 2026-08 | Parallel stream index fixture | Route-specific only |
| [#4506](https://github.com/deepseek-ai/deepseek-harness/discussions/4506) | RC-era | Multi-writer persistence test | Historical failure; current lock absence relevant |
| [#4514](https://github.com/deepseek-ai/deepseek-harness/discussions/4514) | 2026-08-25 | Web/SSRF/path regression set | Alpha.1 claims SSRF fix; re-test current |
| [#4747](https://github.com/deepseek-ai/deepseek-harness/discussions/4747) | `0.1.1-rc.2` | Nested object/`oneOf` provider fixture | Model-route-specific; validator behavior current |
| [#4776](https://github.com/deepseek-ai/deepseek-harness/discussions/4776) | `0.1.1-rc.2` | Incompressible compaction/backoff test | Unconfirmed current; test lead only |

## Unresolved questions

These require future source/release evidence rather than speculation:

- Will a stable release define compatibility and migration guarantees for events, JSONL/Zstd, and SQLite?
- Will persistence gain a documented cross-process writer lease/fencing model?
- Will continuable subagent inboxes and ownership become durably coordinated across processes?
- Will workflow execution gain a journal/resume contract or remain a foreground convenience?
- Will the project publish a security audit and precise platform sandbox support matrix?
- Will web/SDK/ACP application profiles gain supported remote identity, tenant, quota, and deployment contracts?
- Will session deletion/retention/pagination and spill-artifact lifecycle become first-class?
- Will telemetry and official adapter metadata/upload paths gain field-level redaction and stronger delivery/privacy contracts?
- Which current RC-era discussion failures are fixed, explicitly accepted, or still reproducible?
- Will external provider adapter conformance be published per model/gateway?

## Required refresh procedure

On every release:

1. record new tag, date, commit, package/Node/pnpm versions, and prerelease/stable status;
2. diff README, `SAFETY.md`, architecture, release notes, and package graph;
3. diff event types/version, session surface, persistence backends/repair, and upload paths;
4. diff sandbox modes/runners, approval, web carrier/auth, credentials, PTC/workflow, and plugins;
5. diff LLM contract/adapters/catalog, retry, overflow, usage, images, and tool schema assembly;
6. diff subagent providers/continuations and experimental package inclusion;
7. run the Discussion-derived regression matrix against the new release;
8. test copied golden sessions across resume/fork/compaction/crash/upgrade;
9. update adoption gates and remove only limitations disproven by current evidence;
10. stamp the guides with the new research date and commit.

Maximum scheduled interval while prerelease: **30 days**.

## Primary source map

### Product, maturity, and architecture

- [Official Harness site](https://www.deepseek.com/harness/en/)
- [Official repository](https://github.com/deepseek-ai/deepseek-harness)
- [README](https://github.com/deepseek-ai/deepseek-harness/blob/master/README.md)
- [Safety notice](https://github.com/deepseek-ai/deepseek-harness/blob/master/SAFETY.md)
- [Architecture](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md)
- [Cordis paper](https://arxiv.org/abs/2608.25512)

### Core, sessions, and context

- [Agent lifecycle](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/agent-lifecycle.md)
- [Agent core package](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/core/agent/README.md)
- [Session subsystem](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/session.md)
- [Session persistence package](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/session/session-persistence/README.md)
- [Session query subsystem](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/session-query.md)
- [Compaction packages](https://github.com/deepseek-ai/deepseek-harness/tree/master/packages/compaction)
- [System prompt package](https://github.com/deepseek-ai/deepseek-harness/tree/master/packages/core/system-prompt)

### Extensions and orchestration

- [Tools subsystem](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/tools.md)
- [Tool schema catalog](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/tool-catalog.md)
- [Adding a tool](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cookbook/adding-a-tool.md)
- [Subagent subsystem](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/subagent.md)
- [Subagent packages](https://github.com/deepseek-ai/deepseek-harness/tree/master/packages/subagent)
- [Workflow packages](https://github.com/deepseek-ai/deepseek-harness/tree/master/packages/workflow)
- [Experimental packages](https://github.com/deepseek-ai/deepseek-harness/tree/master/packages/experimental)

### Safety, models, applications, and testing

- [Sandbox subsystem](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/sandbox.md)
- [Approval subsystem](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/approval.md)
- [LLM streaming subsystem](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/llm-streaming.md)
- [Provider and model configuration](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/providers.md)
- [SDK packages](https://github.com/deepseek-ai/deepseek-harness/tree/master/packages/sdk)
- [ACP packages](https://github.com/deepseek-ai/deepseek-harness/tree/master/packages/acp)
- [Testing guide](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/testing.md)
- [Postmortems](https://github.com/deepseek-ai/deepseek-harness/tree/master/docs/postmortem)
- [Session telemetry](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/session-telemetry.md)
- [Releases](https://github.com/deepseek-ai/deepseek-harness/releases)

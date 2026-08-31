# AutoGen deep-dive research packet

> **Research date:** 2026-08-31  
> **Purpose:** evidence ledger for [`docs/frameworks/autogen/`](../../frameworks/autogen/README.md)  
> **Scope:** official Microsoft AutoGen repository, source, stable documentation, examples, releases, package registries, maintenance/security/support statements, Microsoft Agent Framework migration material, and a bounded set of issues used only as regression leads.

## Executive finding

AutoGen is a retained framework in maintenance mode, not a vanished product. Python 0.7.5 is the latest verified official framework release; official documentation and package artifacts remain available; the repository still accepts bug fixes, security patches, and documentation. However, new features are not planned, maintenance is community-managed, support is limited, and Microsoft directs new users to Microsoft Agent Framework (MAF).

The correct production posture is therefore conditional:

- stable, pinned, well-contained AutoGen systems can be retained temporarily;
- high-authority or rapidly changing systems should be contained and migrated sooner;
- new strategic systems should normally start on MAF or a simpler current architecture; and
- retention requires explicit ownership, compatibility evidence, security controls, and an exit milestone.

The source review also found material boundaries that architecture decisions must account for:

- the “single-threaded” runtime receives from one queue but processes messages in separate asynchronous tasks;
- runtime snapshots do not automatically cover subscriptions or all component/remote state;
- gRPC worker state APIs are unimplemented and the distributed runtime does not establish a durable-delivery contract;
- intervention handlers are local-runtime-specific;
- MCP workbench state does not restore MCP server/session state;
- framework telemetry can serialize message content; and
- Studio's published package is version-incompatible with the current 0.7.5 framework and is explicitly not a production control plane.

## Research method

The investigation used the following source order:

1. official repository policy files, README, release history, package source, docs source, and samples;
2. stable official AutoGen documentation and API reference;
3. official PyPI/NuGet package metadata;
4. official Microsoft Learn and Agent Framework migration samples;
5. official issues/pull requests/discussions for narrow implementation questions; and
6. source-code inspection when the high-level docs did not define operational semantics.

No community blog or secondary tutorial was used as the basis for a production claim. Issue reports were not used to estimate reliability or prevalence; they became targeted test cases. Repository `main` was not treated as a published package.

The official repository was inspected at commit `027ecf0a379bcc1d09956d46d12d44a3ad9cee14`, dated 2026-04-06, whose subject updates the maintenance-mode banner. Version and package statements were separately checked against release and registry metadata.

## Verified lifecycle and version ledger

| Surface | Verified observation | Evidence | Interpretation |
|---|---|---|---|
| AutoGen repository | maintenance mode; no new features/enhancements; community-managed; fixes/security/docs accepted | [official README](https://github.com/microsoft/autogen) | retained deployments need their own support/exit plan; new feature assumptions are invalid |
| Python framework | `autogen-core`, `autogen-agentchat`, `autogen-ext` 0.7.5; release 2025-09-30 | [release](https://github.com/microsoft/autogen/releases/tag/python-v0.7.5), [PyPI](https://pypi.org/project/autogen-core/) | use 0.7.5 as the current retained line on the research date |
| Python compatibility | AgentChat and Extensions 0.7.5 depend on Core 0.7.5 exactly; Python >=3.10 | official package `pyproject.toml` files | pin/test the complete set and selected extras |
| Studio published package | 0.4.2.2, released 2025-05-27; constrains AutoGen dependencies below 0.6 | [PyPI](https://pypi.org/project/autogenstudio/0.4.2.2/) | isolate from the 0.7.5 runtime; use only for prototyping |
| Studio repository `main` | source version/ranges differ from published PyPI package | official Studio `pyproject.toml` | do not infer installable compatibility from `main` |
| .NET Core | `Microsoft.AutoGen.Core` 0.4.0-dev.3 prerelease, 2025-03-19 | [NuGet](https://www.nuget.org/packages/Microsoft.AutoGen.Core/) | do not claim Python/.NET release parity; avoid confusion with differently named packages |
| MAF migration | official successor developed by the AutoGen and Semantic Kernel teams; architectural/state/tool-loop differences documented | [Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-autogen/) | migrate by behavioral contract, not direct serialization/class substitution |
| support | GitHub issues/discussions/community channels; Microsoft support is limited to stated resources | [SUPPORT.md](https://github.com/microsoft/autogen/blob/main/SUPPORT.md) | no enterprise response-time assumption |
| security | private Microsoft vulnerability-reporting route exists; no supported-version/remediation matrix published | [SECURITY.md](https://github.com/microsoft/autogen/blob/main/SECURITY.md) | maintain a contingency for critical defects and dependencies |

### Package identity finding

The [official FAQ](https://github.com/microsoft/autogen/blob/main/FAQ.md) and [migration guide](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/migration-guide.html) state that Microsoft lost administrative access to the `pyautogen` package and releases after 0.2.34 are not Microsoft releases. The old official 0.2 line is installed through `autogen-agentchat~=0.2`. The separate current `autogen` package name on PyPI must not be assumed to represent Microsoft's maintained framework set.

This is why the guide requires distribution name, version, hash, index, and attestation/signer evidence rather than a version number alone.

## Architecture evidence

### Core, AgentChat, Extensions, Studio

The [application-stack documentation](https://microsoft.github.io/autogen/stable/user-guide/core-user-guide/core-concepts/application-stack.html) and repository README define:

- **Core:** message passing, event-driven agents, runtime/routing, local and distributed execution, cross-language foundation;
- **AgentChat:** opinionated agents, messages, teams, termination, human interaction;
- **Extensions:** provider clients, tools, MCP, code executors, caches, and gRPC runtime components; and
- **Studio:** rapid-prototyping UI and configuration experience.

The architectural unit is a developer-owned message protocol. Agent behavior emerges from message types, routing, subscriptions, and handlers. The guide consequently treats message schema/version/auth/effect semantics as production API contracts.

Studio's documentation and source README call it experimental/research software rather than production-ready. Authentication is experimental, disabled by default, and limited in scope. The published package's old dependency range independently prevents it from serving as the administration tier for a 0.7.5 deployment.

### Agent identity, routing, and lifecycle

Official Core concepts define `AgentId` as type plus key, lazy activation through registered factories, direct one-to-one send, and topic/subscription fan-out. Type subscriptions often map topic source to the destination agent key. A publication with no matching subscription may have no recipient.

The framework does not turn routing keys into security principals. The guide therefore keeps authentication/tenant authorization in the application. Transparent paging of inactive agents is also not implemented, so the actor-style API is not presented as a turnkey virtual-actor capacity model.

### AgentChat messages and tool loop

The official [agents tutorial](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/agents.html) describes `AssistantAgent` as stateful and not thread/coroutine safe; callers pass only new messages. It is a broad “kitchen sink” agent intended to make common patterns easy. The guide recommends a custom narrow agent only when the protocol, state, context, or effect ordering genuinely requires one.

Verified behavior relevant to production:

- default model context is unbounded unless replaced;
- default `max_tool_iterations` is 1;
- multiple model-requested tool calls can execute concurrently;
- provider clients can disable parallel tool calls where supported;
- reflection after tool use introduces another inference;
- only the first of multiple simultaneous handoffs is used; and
- streaming chunks are events yielded live but excluded from the final `TaskResult.messages`.

`BaseChatMessage` is conversational content, while `BaseAgentEvent` represents observable actions such as tool calls, model chunks, thoughts, memory activity, or requested input. UI streams and durable chat history cannot be treated as the same collection.

## Source-code findings with operational impact

### `SingleThreadedAgentRuntime`

Source: [`_single_threaded_agent_runtime.py`](https://github.com/microsoft/autogen/blob/main/python/packages/autogen-core/src/autogen_core/_single_threaded_agent_runtime.py)

Findings:

- one asyncio queue receives messages FIFO;
- each dequeued item is processed in a separate asyncio task, so handlers can overlap;
- the runtime is documented for development/standalone use, not high-throughput/high-concurrency service;
- `ignore_unhandled_exceptions` defaults to true, allowing some background failures to surface later; AgentChat's embedded usage configures stricter behavior;
- `stop()` permits the current item to finish and discards later queued items;
- `stop_when_idle()` is the suitable graceful drain;
- runtime save/load covers instantiated agent state but not subscriptions; and
- trace attributes can include serialized message content.

Interpretation: “single-threaded” must not be translated into serialized business effects. Shared resources still need locking/idempotency, and shutdown/error/telemetry behavior must be explicitly tested.

### gRPC worker runtime

Source: [`_worker_runtime.py`](https://github.com/microsoft/autogen/blob/main/python/packages/autogen-ext/src/autogen_ext/runtimes/grpc/_worker_runtime.py)

Findings:

- workers communicate with a host through gRPC and serialized message envelopes;
- state, agent-state, metadata, and related persistence APIs raise `NotImplementedError`;
- source TODOs/paths expose limitations around reconnects, timeouts/errors, message IDs, and cancellation;
- send requests wait on futures tied to responses; and
- read-loop exceptions can be logged while the loop continues.

The host implementation maintains routing/connection/subscription information but does not establish a durable broker/event-log guarantee. The official MAF migration guidance also characterizes AutoGen's distributed runtime as experimental.

Interpretation: it can be evaluated for cross-process/cross-language routing, but durability, recovery, duplicate/effect semantics, and command history must live outside it. This evidence supports the default recommendation of independently scalable embedded session workers.

### State and component coverage

Official [state documentation](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/state.html) separates live state from component configuration. Team state recursively stores participant/manager state, using names as keys in the recent schema. The saved shape varies by component.

Source review confirmed important exclusions:

- runtime subscriptions are not in `SingleThreadedAgentRuntime` state;
- gRPC worker state is unimplemented;
- MCP workbench save/load/reset does not preserve server/session state; and
- Python callables used for selectors, graph conditions, or approvals are not portable serialized config.

Interpretation: production state needs an application envelope containing exact package/config/prompt/tool/schema fingerprints, and external-effect receipts. Restore is a tested compatibility property, not a framework-wide guarantee.

### Intervention, cancellation, and pause

[`InterventionHandler`](https://github.com/microsoft/autogen/blob/main/python/packages/autogen-core/src/autogen_core/_intervention.py) intercepts local runtime send/publish/response paths but is not supported by the gRPC worker runtime. It is useful for inspection or defense in depth, not as the only authorization layer.

Cancellation tokens rely on propagation and cooperation from model/tool/executor code. AgentChat `ExternalTermination` provides a turn-boundary graceful stop; immediate cancellation can leave state/termination or effects incomplete. Experimental pause/resume hooks do not automatically suspend the coroutine or persist/release the run, and the base agent implementation is a no-op.

Interpretation: durable human waits should terminate at a clean handoff boundary, persist application state/review work, and resume in a new run.

## Tools, MCP, and code-execution evidence

The official [Core tool reference](https://microsoft.github.io/autogen/stable/reference/python/autogen_core.tools.html) defines stateful workbench lifecycle and static/dynamic tool surfaces. The [MCP reference](https://microsoft.github.io/autogen/stable/reference/python/autogen_ext.tools.mcp.html) supports stdio, SSE, and Streamable HTTP, as well as tools/resources/prompts and optional server-initiated sampling, roots, and elicitation.

Security implications derived directly from those capabilities:

- stdio executes a local command and inherits authority unless the environment isolates it;
- remote MCP needs authenticated TLS, server/artifact identity, bounded requests/reconnects, and tenant-safe credentials;
- sampling/roots/elicitation enlarge the server's authority and should be disabled or policy-gated;
- tool/resource/prompt content can carry prompt injection; and
- MCP session state is not recovered by the workbench snapshot.

The command-line executor docs warn that host execution is unsafe for LLM-generated code and recommend Docker. Release 0.7.5 added stronger warnings/default direction. Docker remains only an isolation mechanism to configure: mounts, daemon socket, network, user/capabilities, secrets, image provenance, CPU/memory/PID/disk limits, and artifacts are outside the basic “Docker-backed” label. In particular, mounting the Docker socket—as some sibling-container patterns do—effectively delegates daemon/host authority and is rejected for untrusted code in the guide.

`CodeExecutorAgent` is experimental and its approval function is optional. The guide requires a deterministic approval/effect policy bound to exact normalized execution inputs, rather than keyword review or a broad agent approval.

## Teams and control flow evidence

The [teams tutorial](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/teams.html) explicitly recommends starting with a single agent and adding a team only when needed. Main patterns reviewed:

- RoundRobin: deterministic participant rotation;
- Selector: model-based central speaker selection, optionally constrained by a non-serializable custom selector callable;
- Swarm: local handoff selected from `HandoffMessage`, while retaining shared group context;
- Magentic-One: planning/orchestration with progress and stall-based replanning; and
- GraphFlow: directed transitions, branches, cycles, and concurrent fan-out, with experimental surfaces.

Group chat shares/broadcasts context among participants, so role prompts are not data isolation. Termination conditions are stateful and reset after completed runs; external termination is graceful at a turn boundary, while cancellation may be inconsistent. Team instances reject concurrent runs.

Magentic-One's official docs caution that capable browsing/file/code agents need containers, human oversight, limited network/resources, and no sensitive data. That primary warning anchors the guide's bounded-supervision recommendation.

## Telemetry, testing, and samples

Official [Core telemetry](https://microsoft.github.io/autogen/stable/user-guide/core-user-guide/framework/telemetry.html) and [AgentChat tracing](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tracing.html) describe OpenTelemetry instrumentation across runtimes, tools, agents, and model activity. Source shows that message content may be serialized into span attributes. The guide requires redaction/allowlisting before export and adds application spans for authorization, effect, state, approval, and stream boundaries.

[`ReplayChatCompletionClient`](https://microsoft.github.io/autogen/stable/reference/python/autogen_ext.models.replay.html) supplies deterministic queued responses and records calls, making it appropriate for orchestration contract tests. It cannot replace live provider compatibility/evaluation.

The official AutoGenBench package README is focused on older 0.1/0.2 flows, so it was not promoted as the primary test harness for retained 0.7 AgentChat applications.

The official [FastAPI sample](https://github.com/microsoft/autogen/tree/main/python/samples/agentchat_fastapi) demonstrates API integration and state saved to JSON files, but it also has sample-level assumptions such as local files, permissive CORS, and no complete production authentication, tenant isolation, database concurrency, or effect protocol. The guide treats it as a teaching sample rather than a production blueprint.

## Bounded issue and release review

Issues were chosen only when they clarified a documented/source behavior and could become a concrete regression test.

| Evidence | Status on research date | Use in the guide |
|---|---|---|
| [#6793](https://github.com/microsoft/autogen/issues/6793): team-state JSON datetime failure | fixed by [#6797](https://github.com/microsoft/autogen/pull/6797), released in 0.7.1 | state JSON round-trip/upgrade corpus |
| [#7043](https://github.com/microsoft/autogen/issues/7043): GraphFlow interruption can resume into a stuck transition | open issue report | crash at every graph transition; assert progress or explicit recoverability |
| [#6716](https://github.com/microsoft/autogen/issues/6716): GraphFlow condition/early-termination behavior | open issue report | conditional-edge and termination tests |
| [#6710](https://github.com/microsoft/autogen/issues/6710): GraphFlow cycle bug | fixed and reflected in later release notes | cycle budget/detection regression test |
| [#4029](https://github.com/microsoft/autogen/issues/4029): cancellation-token defect | fixed in older release line | cancellation propagation/failure test, not a current-defect claim |

The 0.6.0 through 0.7.5 release notes were used to identify behavior-changing test areas: concurrent GraphFlow branches, callable/non-serializable conditions, streaming tools, multiple tool iterations, GenAI spans, retained graph state, nested teams, MCP additions, approval functions, parallel tool control, state serialization, Redis fixes, streaming correlation, graph cycles, MCP pending futures, and code-execution warnings/defaults.

No claim says these issues are common, that unlisted issues cannot exist, or that a “fixed” label is sufficient evidence without application tests.

## Microsoft Agent Framework migration findings

The official [migration guide](https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-autogen/) and [sample mappings](https://github.com/microsoft/agent-framework/tree/main/python/samples/autogen-migration) establish MAF as the successor direction, but not a drop-in replacement.

Key differences recorded for characterization:

- AutoGen Core/team event-driven design versus MAF typed workflow/data-flow design;
- AutoGen internal state versus MAF session/checkpoint concepts;
- provider/model-client surface differences;
- different streaming event types/lifecycle;
- tool-loop defaults—AutoGen `AssistantAgent` defaults to one iteration while MAF can continue multi-turn tool use automatically; and
- team patterns map conceptually to sequential/group chat, handoff, and Magentic builders, not to binary-compatible config or state.

The migration guide contained a point-in-time statement about MAF's distributed focus/status. Because target framework hosting evolves, the AutoGen guides instruct readers to consult the current MAF guide rather than freeze that statement as a permanent limitation.

The selected rollout is a strangler migration: inventory, characterization tests, framework-neutral policy/effect/state seams, one low-risk slice, shadow comparison, canary effects with a shared idempotency ledger, session drain, and retirement. Rollback must include state/effect compatibility, not only a package downgrade.

## Claims deliberately excluded or narrowed

| Tempting claim | Decision |
|---|---|
| “AutoGen is dead/unsupported.” | Rejected. Official state is maintenance mode with fixes/security/docs and community support, not disappearance. |
| “0.7.5 is supported until date X.” | Rejected. No official support-lifetime or SLA was found. |
| “The gRPC runtime guarantees at-least-once/exactly-once delivery.” | Rejected. No such durable contract was found; source has unimplemented state and recovery limits. |
| “Saving team state makes the workflow durable.” | Rejected. Coverage is component-specific and excludes external effects/remote sessions. |
| “Single-threaded runtime serializes all handler work.” | Rejected by source; handlers run in separate asyncio tasks. |
| “Docker code executor is secure by default.” | Narrowed. It is safer than host execution, but sandbox hardening remains deployment-owned. |
| “Studio can operate the current production runtime.” | Rejected for the published package due version incompatibility and prototype/security status. |
| “Python and .NET AutoGen are at parity.” | Rejected; package versions/maturity differ. |
| “An open issue proves a widespread production defect.” | Rejected. Issues are regression leads only. |
| “MAF is a direct class/state replacement.” | Rejected by official architectural and migration differences. |

## Guide-to-evidence map

| Guide | Primary evidence groups |
|---|---|
| [README](../../frameworks/autogen/README.md) | maintenance/support/security statements, registry versions, successor guidance |
| [Architecture and lifecycle](../../frameworks/autogen/architecture-and-lifecycle.md) | application stack, package source, Studio docs/metadata, Core runtime concepts |
| [Agents, messages, and runtime contracts](../../frameworks/autogen/agents-messages-and-runtime-contracts.md) | Core identity/routing docs, AgentChat agent/message docs, local runtime source |
| [Tools, workbenches, MCP, and code execution](../../frameworks/autogen/tools-workbenches-mcp-and-code-execution.md) | tool/MCP/executor references and source, 0.7.5 release, Magentic security warnings |
| [Teams, group chat, and control flow](../../frameworks/autogen/teams-group-chat-and-control-flow.md) | team pattern/termination docs, group-chat source, bounded GraphFlow issues |
| [State, persistence, and resume](../../frameworks/autogen/state-persistence-and-resume.md) | state docs/source, runtime/MCP coverage, fixed/open state issues |
| [Streaming, intervention, and human control](../../frameworks/autogen/streaming-intervention-and-human-control.md) | messages/events, HITL/termination/cancellation docs, intervention source |
| [Distributed runtime and deployment](../../frameworks/autogen/distributed-runtime-and-deployment.md) | runtime architecture, gRPC source, FastAPI sample, MAF migration guide |
| [Observability, testing, and debugging](../../frameworks/autogen/observability-testing-and-debugging.md) | telemetry/logging source/docs, replay client, AutoGenBench scope, issues |
| [Security, reliability, and operations](../../frameworks/autogen/security-reliability-and-operations.md) | security/support/package statements, MCP/executor/telemetry authority |
| [Maintenance, upgrades, and migration](../../frameworks/autogen/maintenance-upgrades-and-migration.md) | maintenance banner, releases, package migration history, MAF guide/samples |

## Refresh triggers

Re-run this packet when any of the following occurs:

- a new official AutoGen Python/.NET/Studio release or maintenance statement;
- a security advisory, package ownership change, or dependency vulnerability affecting used extras;
- provider API/model behavior breaks a live compatibility test;
- the official MAF migration guide, workflow/checkpoint, provider, or hosting surfaces change materially;
- a cited open issue closes or its fix ships;
- an AutoGen deployment adopts gRPC, MCP server-initiated capabilities, arbitrary code execution, GraphFlow, or days-long human workflows; or
- the retention decision date expires.

Refresh procedure: verify registries and provenance, inspect official release diff/source for used components, rerun issue status, update behavior/state test seeds, validate successor mappings, and change only claims supported by primary evidence.

## Primary source register

### Lifecycle, packages, and policies

- [Microsoft AutoGen repository](https://github.com/microsoft/autogen)
- [AutoGen releases](https://github.com/microsoft/autogen/releases)
- [AutoGen Core on PyPI](https://pypi.org/project/autogen-core/)
- [AutoGen Studio 0.4.2.2 on PyPI](https://pypi.org/project/autogenstudio/0.4.2.2/)
- [Microsoft.AutoGen.Core on NuGet](https://www.nuget.org/packages/Microsoft.AutoGen.Core/)
- [FAQ](https://github.com/microsoft/autogen/blob/main/FAQ.md)
- [Security policy](https://github.com/microsoft/autogen/blob/main/SECURITY.md)
- [Support policy](https://github.com/microsoft/autogen/blob/main/SUPPORT.md)

### Architecture and operation

- [AutoGen stable documentation](https://microsoft.github.io/autogen/stable/)
- [Application stack](https://microsoft.github.io/autogen/stable/user-guide/core-user-guide/core-concepts/application-stack.html)
- [Runtime architecture](https://microsoft.github.io/autogen/stable/user-guide/core-user-guide/core-concepts/architecture.html)
- [AgentChat agents](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/agents.html)
- [AgentChat teams](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/teams.html)
- [AgentChat state](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/state.html)
- [AgentChat human-in-the-loop](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/human-in-the-loop.html)
- [Core telemetry](https://microsoft.github.io/autogen/stable/user-guide/core-user-guide/framework/telemetry.html)
- [Single-threaded runtime source](https://github.com/microsoft/autogen/blob/main/python/packages/autogen-core/src/autogen_core/_single_threaded_agent_runtime.py)
- [gRPC worker runtime source](https://github.com/microsoft/autogen/blob/main/python/packages/autogen-ext/src/autogen_ext/runtimes/grpc/_worker_runtime.py)
- [AgentChat FastAPI sample](https://github.com/microsoft/autogen/tree/main/python/samples/agentchat_fastapi)

### Migration

- [AutoGen Python migration guide](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/migration-guide.html)
- [Microsoft Agent Framework migration from AutoGen](https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-autogen/)
- [Official AutoGen migration samples in Agent Framework](https://github.com/microsoft/agent-framework/tree/main/python/samples/autogen-migration)


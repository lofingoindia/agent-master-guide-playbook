# DeepSeek Harness: production-minded guide to a developer preview

> **Research date:** 2026-08-31  
> **Version examined:** `0.1.2-alpha.2`, repository commit `0a53fb55bea101816fa226bb964ae2bed71c343b` (2026-08-30)  
> **Maturity:** developer preview; compatibility-breaking changes are explicitly expected  
> **Production status:** the project says it is experimental, not security-audited, and not production-ready

DeepSeek Harness is an open-source, plugin-composed agent runtime and workspace harness. Its most important idea is not a particular model or UI: the model adapter, session log, tool registry, agent loop, persistence backend, sandbox policy, and application surface are all replaceable Cordis plugins.

That composability is valuable, but it also expands the trusted computing base. A Cordis service scope is dependency isolation, not an operating-system security boundary. A plugin executes in the host process. Programmatic Tool Calling (PTC) is intentionally equivalent in trust to shell access. The built-in filesystem sandbox restricts selected filesystem effects; it does not make arbitrary agent execution safe.

Use this area as an engineering and adoption guide, not as a production endorsement.

## Start with the right layer

The name “DeepSeek Harness” covers several layers that should not be conflated:

| Layer | What it is | What it is not |
|---|---|---|
| **Cordis runtime** | Effect-tracked plugin/service/event composition with reversible registration | An OS sandbox or distributed workflow engine |
| **Harness core** | Sessions, derived model history, tools, agent state, and the step/turn loop | A single immutable “agent algorithm” |
| **CLI/workspace harness** | Profiles, presets, instructions, storage, web/headless/SDK/ACP applications | A hardened multi-tenant control plane |
| **Extensions** | Plugins, bundles, tools, skills, subagent providers, model adapters | Automatically trusted or compatible code |
| **Persistence** | Append-only session events with JSONL/Zstandard or opt-in SQLite storage | A cross-process lease, retention service, or workflow transaction log |
| **Safety controls** | Tool policy, approval, filesystem checks, and platform-specific local runners | A general process/network/credential containment guarantee |

## Guide map

1. [Architecture, Cordis runtime, and agent loop](architecture-runtime-and-agent-loop.md) — composition boundaries, profiles versus presets, and exact turn/step sequencing.
2. [Sessions, events, and persistence](sessions-events-and-persistence.md) — the canonical log, derived history, crash repair, storage formats, and single-writer discipline.
3. [Plugins, tools, and skills](plugins-tools-and-skills.md) — execution contracts, policy hooks, package trust, skill precedence, and workspace instructions.
4. [Context, compaction, and continuity](context-compaction-and-continuity.md) — model-visible context, history surfaces, pressure handling, summaries, and continuity traps.
5. [Subagents, workflows, and orchestration](subagents-workflows-and-orchestration.md) — providers, fork semantics, continuations, process-local ownership, and orchestration choices.
6. [Safety, permissions, and the sandbox boundary](safety-permissions-and-sandbox-boundary.md) — threat model, approval behavior, filesystem runners, web access, PTC, and secrets.
7. [Models, providers, and streaming](models-providers-and-streaming.md) — provider routes, adapter obligations, credentials, model switching, and conformance tests.
8. [Debugging, telemetry, testing, and evaluation](debugging-telemetry-testing-and-evaluation.md) — event-led diagnosis, observability privacy, upstream test lessons, and release qualification.
9. [Deployment, operations, and reliability](deployment-operations-and-reliability.md) — application profiles, shutdown, upgrades, storage, recovery, and operational runbooks.
10. [Evolution, limitations, adoption gates, and alternatives](version-evolution-adoption-gates-and-alternatives.md) — release volatility, unresolved boundaries, evidence-based gates, and when to choose another architecture.

The original concise overview remains at [DeepSeek Harness](../deepseek-harness.md). The evidence ledger, source map, and research notes are in the [DeepSeek Harness deep-dive research packet](../../research/packets/deepseek-harness-deep-dive.md).

Use the repository's [agent state and event contract](../../runtime/agent-state-and-event-contracts.md) to translate Harness session events into application-owned run identity, ordering, terminal outcomes, reconnect behavior, and effect evidence. The internal prerelease session format is an adapter, not the product contract.

## Four operating invariants

Treat these as default policy unless a deployment has evidence for something stronger:

1. **One writer owns a session and its persistence root.** The current persistence implementation coordinates writes inside one process but documents no cross-process writer lease.
2. **Plugins and executable agent modes are host-trusted code.** Review, pin, and isolate them exactly as shell-capable dependencies.
3. **Persistence is not rollback.** Cancellation and crash repair close the event history; they cannot undo an external side effect.
4. **Every provider route requires conformance testing.** Tool schemas, streaming fragments, cancellation, usage, images, and context-overflow errors vary by adapter and gateway.

## Recommended reading paths

### Evaluating whether to adopt

Read [evolution and adoption gates](version-evolution-adoption-gates-and-alternatives.md), [safety](safety-permissions-and-sandbox-boundary.md), and [operations](deployment-operations-and-reliability.md). A default production decision should remain “not yet” until the project's own safety and maturity notices change and your gates pass.

### Building a plugin or tool

Read [architecture](architecture-runtime-and-agent-loop.md), [plugins/tools/skills](plugins-tools-and-skills.md), then [testing](debugging-telemetry-testing-and-evaluation.md). Test unload/reload, cancellation, invalid model arguments, duplicate registration, and output serialization—not only the happy path.

### Operating persistent agents

Read [sessions/persistence](sessions-events-and-persistence.md), [context/compaction](context-compaction-and-continuity.md), [orchestration](subagents-workflows-and-orchestration.md), and [operations](deployment-operations-and-reliability.md). Keep a single writer, preserve raw session artifacts during incidents, and separate event durability from external-effect idempotency.

### Security review

Read [safety](safety-permissions-and-sandbox-boundary.md) first, then inspect [plugins/tools/skills](plugins-tools-and-skills.md) and [models/providers](models-providers-and-streaming.md). The highest-risk paths are host plugins, shell/PTC/workflow code, ambient credentials, network egress, MCP commands, and any custom web carrier.

## Volatility and refresh policy

This guide set has **very high refresh urgency**. Re-research it immediately when any of the following occurs:

- a new Harness prerelease or stable tag appears;
- `SAFETY.md`, `docs/architecture.md`, session persistence, sandbox, approval, web authentication, or model-adapter code changes;
- a storage format or event schema version changes;
- a release introduces a new profile, preset, orchestration backend, or telemetry path;
- the project claims a security audit, production readiness, multi-process ownership, or compatibility stability;
- a current-source check contradicts a version-scoped Discussion used here.

Even without a trigger, refresh within **30 days** while the package remains prerelease. Do not infer maturity from tag wording: public tags moved from release-candidate labels back to alpha labels during August 2026, and every GitHub release examined was still marked prerelease.

## Primary sources

- [Official Harness site](https://www.deepseek.com/harness/en/)
- [Official repository](https://github.com/deepseek-ai/deepseek-harness)
- [README maturity notice](https://github.com/deepseek-ai/deepseek-harness/blob/master/README.md)
- [Safety notice](https://github.com/deepseek-ai/deepseek-harness/blob/master/SAFETY.md)
- [Architecture documentation](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md)
- [Official releases](https://github.com/deepseek-ai/deepseek-harness/releases)
- [Cordis design paper](https://arxiv.org/abs/2608.25512)

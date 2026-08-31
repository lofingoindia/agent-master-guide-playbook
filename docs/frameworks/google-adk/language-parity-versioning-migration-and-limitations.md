# Language Parity, Versioning, Migration, and Limitations

## Snapshot, not a universal support claim

ADK is multi-language, but its repositories, release numbers, implementation styles, and feature badges are independent. “Supported in ADK” is too imprecise for architecture decisions.

**Research snapshot: 2026-08-31**

| Language | Checked release | Major status | Important snapshot note |
|---|---:|---|---|
| Python | `2.8.0` | ADK 2 GA; maintained 1.x line | Broadest workflow/eval surface; async-first; rapid fixes and optional extras |
| Go | `2.2.0` | ADK 2 GA; maintained 1.x line | Graph workflows; v2 requires Go 1.25+; event creation receives context for replay-oriented behavior |
| TypeScript | `2.0.0` | 2.0 package released | `BaseAgent` now extends `BaseNode`; `Workflow` remains experimental; template sequential/parallel/loop classes are deprecated |
| Java | `1.8.0` | 1.x | JVM SDK with its own reactive/async idioms; not an ADK 2 graph release |
| Kotlin | `0.8.0` | Pre-1.0 | Server and Android capabilities differ; A2A consumption experimental and exposure unavailable in the documented quickstart |

## Feature evidence matrix

This table records official documentation badges and release evidence checked on the research date. A blank/limited cell means “verify,” not necessarily “impossible.”

| Capability | Python | Go | TypeScript | Java | Kotlin |
|---|---|---|---|---|---|
| Core agents/tools/runner | Yes | Yes | Yes | Yes | Yes |
| ADK 2 graph `Workflow` | GA 2.x | GA 2.x | Experimental in 2.0 | Not documented | Not documented |
| Template sequential/parallel/loop | Yes, legacy-compatible | Yes | Deprecated in 2.0 | Yes | Yes |
| `DatabaseSessionService` official guide | Yes | Yes | Not listed | Not listed | Not listed |
| Managed `VertexAiSessionService` guide | Yes | Yes | Not listed | Yes | Yes, server JVM only |
| Tool confirmation guide | Experimental | Experimental | Experimental/manual details | Support header omits Java, but body includes Java example; verify | Not listed |
| Graph `RequestInput` | Yes | Yes | Documented in 2.0; Workflow experimental | Not documented | Not documented |
| Dedicated cancellation guide | Open gap for standard external cancel | Not listed | `AbortSignal` documented | Not listed | Release fixes exist, no equivalent guide contract |
| Built-in evaluation framework | Yes | Not listed | Not listed | Not listed | Not listed |
| Structured OTel logging guide | Yes | Yes | Not listed | Not listed | Yes |
| Metrics guide | Yes | Not listed | Not listed | Not listed | Yes |
| MCP overview badge | Yes | Yes | Yes | Yes | Badge omits Kotlin, although Kotlin release notes mention MCP work |
| A2A | Expose/consume | Expose/consume | Verify current page/release | Expose/consume pages exist | Consume experimental; cannot expose via documented surface |
| Managed Agent Runtime deployment guide | Yes | Yes | Not documented | Not documented | Not documented |

The MCP/Kotlin mismatch is a concrete example: release notes and documentation badges can lag or describe different maturity levels. Resolve these through the exact API/reference and an integration test.

## ADK 1 to ADK 2

ADK 2 is not merely a package bump. It introduces a node/workflow architecture, graph scheduling, dynamic workflows, and replay-oriented context changes. Python 2.0 became GA on 2026-05-19; Go 2.0 on 2026-06-30. Both maintain 1.x release streams, so “latest release” can mean latest compatible patch or latest major.

Migration plan:

1. inventory agents, custom agents, workflow agents, callbacks/plugins, session services, event consumers, model adapters, and deployment APIs;
2. classify pending sessions/interruptions and how long they must resume;
3. pin a representative production event dataset;
4. migrate imports/types/configuration and compile first;
5. compare event sequence, state/artifact deltas, tool calls, streaming, and usage—not only final text;
6. run concurrency, replay, approval, cancellation, and post-effect crash tests;
7. canary by tenant/session cohort;
8. drain or version-route old pending work;
9. update public stream/event schemas only through an explicit compatibility version.

Do not migrate a simple stable sequential workflow to a graph only because the new major exists. Migrate for a measured capability or maintenance benefit.

## TypeScript 2.0 change

TypeScript 2.0 removed `LLMAgentWrapper` because agents extend nodes directly. It deprecates template workflow classes while retaining them with warnings, and still marks the new `Workflow` experimental. A production TypeScript migration should therefore separate:

- required API changes for 2.0;
- optional workflow redesign;
- experimental graph adoption.

Avoid converting all workflows and changing the runtime major in one release.

## Go 2 changes

Go v2 requires Go 1.25 or later. The migration notes include event creation with execution context, which supports deterministic/replay-aware behavior but changes interfaces. Verify module import paths, node/event types, context propagation, and all custom implementations.

## Session and data migrations

Python's session database schema changed in v1.22.0 and has a documented migration. SDK upgrade planning must include:

- database schema and driver compatibility;
- serialized event/state/artifact payloads;
- managed session API behavior;
- memory extraction versions;
- pending interruption IDs and tool/function-response matching;
- retention and deletion jobs;
- consumers of raw SDK event JSON.

Keep raw SDK types behind an application protocol so language upgrades do not become client-breaking changes.

## Version-specific failure evidence

Issue reports are bounded test seeds:

- Python 2.5.0 had a closed reproduction where workflow replay mixed prior invocation events during later HITL resume.
- Python has an open request for supported external cancellation of standard `run_async`.
- Python has an open report that an A2A peer can forge tool confirmation in the examined path.
- Historical Java releases fixed runner sequencing, managed-session event loss, confirmation attribution, and thought/tool parts.
- Kotlin releases include cancellation and MCP-schema fixes.
- Current Python releases include replay/function-response, parallel error, artifact atomicity, deserialization, security, and performance fixes.

Do not freeze these as permanent framework verdicts. Convert them into conformance tests and recheck issue/release status.

## Known architectural limitations

- Session persistence is not exactly-once external-effect execution.
- Graph documentation warns that some third-party integrations may be incompatible; graph-plus-streaming/live compositions require exact-version tests.
- Tool confirmation is experimental and documents persistent-session-service limitations.
- Standard cancellation is not a uniform cross-language contract.
- Evaluation and observability surfaces are not language-parity features.
- Model adapters differ in tools, structured output, reasoning parts, streaming, safety, retries, and usage.
- Managed Agent Runtime capability, naming, and APIs can evolve separately from SDKs.
- A2A and MCP expand trust and availability boundaries; they do not supply application authorization.
- In-memory session, artifact, memory, and credential stores are development/test implementations.

## Upgrade policy

Pin:

- ADK SDK/CLI and language runtime;
- model and provider adapter;
- MCP/A2A SDKs and servers;
- session database driver/schema;
- telemetry exporters/plugins;
- optional `eval`, `extensions`, and cloud extras;
- deployment image/runtime APIs.

For each upgrade, publish a compatibility record containing exact versions, change summary, event/schema diff, conformance results, evaluation deltas, security review, canary outcome, and rollback/drain plan.

## Refresh triggers

- any selected-language major/minor release;
- TypeScript Workflow stabilization or template-workflow removal;
- Java/Kotlin ADK 2 graph releases;
- cross-language cancellation, confirmation, evaluation, or telemetry additions;
- workflow replay/rehydration and event schema changes;
- session database/managed-session migrations;
- MCP/A2A protocol major changes;
- managed Agent Runtime API/naming/security changes;
- model capability and structured-output/tool behavior changes;
- new security advisories or dependency incidents.

## Adoption checklist

- [ ] Every required capability is verified in the selected language and exact version.
- [ ] Preview/experimental features have an exit plan and bounded blast radius.
- [ ] Raw SDK events do not leak into an unversioned public client contract.
- [ ] Pending runs are drained, migrated, version-routed, or safely invalidated.
- [ ] Database/event/artifact/memory migrations are rehearsed on realistic data.
- [ ] Language-specific concurrency, cancellation, streaming, and cleanup tests pass.
- [ ] Release notes and open issues are reviewed before every rollout.
- [ ] Rollback includes durable state and pending approval compatibility.

## Primary sources

- [ADK 2 overview and GA dates](https://github.com/google/adk-docs/blob/main/docs/2.0/index.md)
- [ADK API references](https://adk.dev/api-reference/)
- [Python releases](https://github.com/google/adk-python/releases)
- [Go releases](https://github.com/google/adk-go/releases)
- [Go v2 migration notes](https://github.com/google/adk-go/blob/main/README-v2.md)
- [TypeScript releases](https://github.com/google/adk-js/releases)
- [Java releases](https://github.com/google/adk-java/releases)
- [Kotlin releases](https://github.com/google/adk-kotlin/releases)
- [Session service support and migration](https://adk.dev/sessions/session/)
- [Tool confirmation support and limitations](https://adk.dev/tools-custom/confirmation/)
- [ADK 2 graphs and limitations](https://adk.dev/graphs/)

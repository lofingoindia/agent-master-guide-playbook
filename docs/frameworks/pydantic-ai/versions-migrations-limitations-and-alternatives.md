# Versions, Migrations, Limitations, and Alternatives

**Research date:** 2026-08-31  
**Inspected release:** Pydantic AI `v2.36.0` (released 2026-08-29); stable V2 began 2026-06-23

Treat Pydantic AI, provider extras/SDKs, UI/MCP integrations, Harness capabilities and durable adapters as one qualified deployment set. A compatible core import does not prove persisted messages, stream consumers, provider settings or workflow histories remain compatible.

## Version policy

The official V2 policy says minor releases should not intentionally break stable APIs, and deprecated stable functionality is removed at a later major. Important exclusions remain:

- fixes can break code that depended on undocumented behavior;
- new message parts, stream events and optional fields can appear in minor releases;
- default OpenTelemetry version/attributes can change;
- beta-module APIs and behavior may change incompatibly;
- representation output can change.

Use tolerant message/event consumers, pin exact versions for risk-sensitive deployments, and test every minor upgrade. V1 was promised security fixes for at least six months after stable V2—through at least 2026-12-23—not indefinite feature support.

## V1 to V2 strategy

The official recommended path is:

1. upgrade to the latest V1;
2. make deprecation warnings visible and resolve all of them;
3. upgrade to V2;
4. review behavior changes that warnings could not express;
5. run provider, message, stream, durability and effect acceptance tests.

Histories serialized with `ModelMessagesTypeAdapter` in V1 are documented to deserialize in V2. Application envelopes, provider-specific content, workflow payloads and custom metadata still need their own compatibility tests.

## Material V2 behavior changes

| Change | Production impact |
|---|---|
| capabilities became primary extension unit | several constructor options moved/renamed; composition order matters |
| bare install has fewer extras | explicitly install every provider/UI/durable/spec integration used |
| `openai:` selects Responses API | use `openai-chat:` to retain Chat Completions behavior |
| default `end_strategy` is `graceful` | co-emitted function tools may execute alongside successful output |
| default instrumentation format is V5 | dashboards/trace fixtures and aggregated usage fields change |
| output/native event taxonomy changed | UI/event consumers must handle dedicated output events and native parts |
| model profiles became `TypedDict` values | attribute access/runtime class checks and merge behavior change |
| provider-prefixed model names required | bare model strings fail; routing becomes explicit |
| MCP moved to `MCPToolset`; native terminology replaced builtin | imports/config/toolset lifecycle need migration |
| `pydantic_graph.persistence` removed | Harness Step Persistence is an alternative, not an identical durable graph store |

Audit the `graceful` strategy first: it can make side-effecting function tools run where V1's early final-output path skipped them.

## Durable migration rules

Stable workflow/activity/step/task/operation names, agent names, toolset/capability IDs and serialized types are durable APIs.

- Temporal: use worker versioning/patching and replay histories; preserve registration identity.
- DBOS: operation order is replay-sensitive; use patches/application versions and drain old workflows.
- Prefect: cache identity and result persistence can change. V2.36 altered hashing for capability-owned durable operations, so an in-flight flow can miss old cache and re-execute.
- Restate: version immutable service deployments and preserve journaled operation order.

Deprecated wrapper agents for Temporal/DBOS/Prefect should migrate to durability capabilities before V3. Follow backend-specific legacy registration/cache guidance while old executions drain.

V2.36 added the public durable-backend builder and `@durable_operation`. Treat explicit operation names and annotated serializable parameters as durable contracts; this is a new surface and needs qualification before custom-backend production use.

## Limitations to state plainly

- Type/schema validation does not prove truth, authentication, authorization, policy compliance or effect safety.
- Provider abstraction does not guarantee equivalent structured output, tools, thinking, files, usage, streaming or retention.
- `StructuredDict` supplies a schema but does not perform runtime Pydantic validation.
- There is no single built-in whole-run wall-clock timeout.
- Token and cost limits usually detect the crossing response after it has been sent/billed.
- Unknown price data can make cost-limit enforcement unavailable.
- Synchronous tools can continue in worker threads after timeout/cancellation.
- Client-submitted history and approvals are forgeable; sanitization is not authenticity.
- `TestModel` cannot emulate provider-native tools or remote quirks.
- `temperature=0` and seeds do not guarantee reproducibility.
- Response-inspection fallback is non-streaming.
- Core message history is not durable workflow state or long-term semantic memory.
- A durable adapter does not make arbitrary external effects exactly once.
- Durable streaming may be buffered/replayed rather than live.
- OTel GenAI attributes and event/message variants evolve.
- Multi-agent delegation adds cost/latency and can lose shared usage across durable context copies.
- The built-in web UI is development tooling, not a production security boundary.

## When to use Pydantic AI

Choose it when:

- the service is Python-first and gains value from typed dependencies, tools and outputs;
- explicit validation/correction/error semantics matter;
- several providers need one application control surface with tested route-specific behavior;
- maintained durable-engine adapters fit an existing Temporal/DBOS/Prefect/Restate deployment;
- the team is prepared to own authorization, effects, persistence, context policy and operations.

## Alternatives

| Dominant need | Simpler/better starting point | Why |
|---|---|---|
| one structured model call | provider SDK + Pydantic `TypeAdapter` | less loop and dependency surface |
| deep provider-specific/native features | direct provider SDK | no abstraction mismatch; immediate new features |
| explicit checkpointed graph topology | LangGraph or a deterministic workflow | state transitions/replay are primary abstraction |
| long-running business workflow | Temporal/DBOS/Restate/Prefect with thin model activities | engine semantics lead, agent is one unit |
| TypeScript UI/stream product | Vercel AI SDK or native web stack | frontend protocol/ecosystem fit |
| managed provider agent platform | provider-native agents | hosted tools/state/operations may reduce application work |
| shell/filesystem coding harness | dedicated sandbox/harness | workspace isolation and artifact lifecycle dominate |

The alternative may still use Pydantic models at boundaries. The decision is whether a model-directed loop, Pydantic AI's error taxonomy and its adapters remove enough custom plumbing to justify their operational surface.

## Upgrade runbook

1. Read release notes, upgrade guide and security advisories from the currently deployed version to target.
2. Diff installed extras, provider SDKs and transitive adapter versions.
3. Run old serialized messages, approvals, artifacts and durable payloads through the target.
4. Replay/recover representative in-flight durable executions.
5. Run deterministic control-flow tests and live provider contracts.
6. Re-run security, cancellation, retry-multiplication, effect and UI protocol suites.
7. Compare traces, metric fields, latency, usage and cost.
8. Canary one tenant/workload; preserve rollback workers for durable histories.
9. Drain or version executions whose cache/operation identity changes.
10. Record the qualified package/model/prompt/policy set.

## Refresh triggers

- any Pydantic AI minor/major, security advisory or V1 support-policy change;
- new message/event/output mode or changed default model/profile/settings;
- provider SDK/API, MCP or UI protocol upgrade;
- Harness persistence/memory/guardrail/security change;
- durable backend protocol, hashing, serializer, replay or versioning change;
- OTel GenAI semantic-convention or instrumentation-default change;
- resolution of cancellation issue #6460, durable validation issue #6979 or usage issue #6886.

## Primary sources

- [Version policy](https://ai.pydantic.dev/version-policy/)
- [V2 upgrade guide](https://ai.pydantic.dev/changelog/) and [V1-to-V2 migration map](https://ai.pydantic.dev/migration/)
- [v2.36.0 release](https://github.com/pydantic/pydantic-ai/releases/tag/v2.36.0)
- [Security advisories](https://github.com/pydantic/pydantic-ai/security/advisories)
- [Capabilities and Harness boundary](https://ai.pydantic.dev/capabilities/overview/)


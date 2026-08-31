# Research Packet: Strands Agents Deep Dive

> Research date: **2026-08-31**
>
> Purpose: evidence ledger for the Strands Agents framework guide set
>
> Confidence: high for documented/current-source behavior; explicitly bounded for open issues and fast-moving experimental features

## Research question

What does the current Strands Agents runtime actually guarantee across Python and TypeScript, and what must a production application add for security, reliability, state, deployment, testing, and cost control?

This packet separates five surfaces that are often conflated:

1. the in-process Strands SDK;
2. model-provider adapters and provider-specific features;
3. local, MCP, and provider-hosted tools;
4. sessions, memory, and multi-agent orchestration;
5. hosting platforms such as Amazon Bedrock AgentCore Runtime.

## Snapshot and repository lineage

The primary source is the consolidated [`strands-agents/harness-sdk`](https://github.com/strands-agents/harness-sdk) monorepo. At research time, `main` was inspected at commit `9062527e` dated 2026-08-29. The former standalone TypeScript repository is archived; package names remain separate.

| Surface | Snapshot checked | Runtime floor |
|---|---:|---:|
| Python SDK | `strands-agents` 1.54.0, released 2026-08-27 | Python 3.10+ |
| TypeScript SDK | `@strands-agents/sdk` 1.14.0, released 2026-08-21 | Node.js 20+ |
| Evaluation SDK | `strands-agents-evals`, separate Python package | Python 3.10+ in current quickstart |
| Community tools | `strands-agents-tools`, separate package/repository | version and advisories must be checked independently |

The Python and TypeScript packages share concepts and a repository, not a synchronized version number or identical behavior.

## Source hierarchy used

1. Current official source, tests, changelogs, package metadata, and contributor testing guides.
2. Current official Strands documentation.
3. AWS service documentation, pricing, and security bulletins for Bedrock, AgentCore, and Lambda.
4. GitHub security advisories.
5. A bounded set of maintainer issues used only to identify known gaps or ambiguous edges.

Blog posts and case studies were used for deployment context, not as proof of SDK guarantees. Search snippets and old standalone repositories were not treated as authoritative when current monorepo source disagreed.

## Claim ledger

### Runtime and invocation

| Claim | Evidence | Production interpretation |
|---|---|---|
| The loop alternates model streaming and tool execution until a stop condition. | [Agent loop](https://strandsagents.com/docs/user-guide/concepts/agents/agent-loop/) and current event-loop source/tests | Tools can create multiple model turns; price and latency must be bounded per invocation. |
| Model-requested tool failures are returned as tool error results rather than always crashing the invocation. | Event-loop and tool-executor tests in the monorepo | Return safe, actionable error categories; do not leak stack traces or secrets. |
| Invocation limits cover turns, output tokens, and total tokens, but checks occur at loop boundaries. | Current invocation-limit docs/source/tests | A model call can overshoot a token threshold and tools requested by the preceding turn may complete before the next check. Add wall-clock/tool limits too. |
| Default tool execution is concurrent when a model emits multiple tool calls. | [Tool executors](https://strandsagents.com/docs/user-guide/concepts/tools/executors/) and source | Sibling side effects can overlap and events can interleave. Use sequential execution where ordering or shared mutable state matters. |
| The same agent object rejects overlapping invocations by default. | Python `ConcurrencyException` and TypeScript `ConcurrentInvocationError` source/tests | Do not share a mutable agent instance across concurrent requests without an explicit ownership model. Python's unsafe reentrant mode is not a safety mechanism. |

### Retries and cancellation

The built-in retry strategy is a **model throttling retry**, not a transaction retry. Both SDKs default to six total model attempts with an initial four-second delay. Backoff surfaces differ: the current Python implementation exposes its retry strategy, while TypeScript provides exponential, linear, and constant backoff implementations and jitter choices. Never share one stateful retry-strategy instance across concurrent turns.

Cancellation is cooperative:

- Python accepts a `threading.Event` cancellation signal in current releases and checks it at model/tool boundaries. MCP cancellation is best effort. Ordinary synchronous or blocking tool code runs until it observes or forwards cancellation.
- TypeScript uses `AbortSignal`, composes signals, and passes a cancellation signal through tool context. Concurrent siblings already started still need to cooperate.
- Provider HTTP libraries and remote services determine whether an in-flight request can actually be aborted.

The Python 1.54.0 changelog is important because external cancellation was added there; older articles can be stale. The current source also makes clear that an already-set signal may reach its first checkpoint only after initial invocation bookkeeping begins.

Primary evidence: [Python changelog](https://strandsagents.com/changelog/harness/python-v1.54.0/), [retry strategies](https://strandsagents.com/docs/user-guide/concepts/agents/retry-strategies/), and the [agent-loop cancellation section](https://strandsagents.com/docs/user-guide/concepts/agents/agent-loop/#cancellation).

### Models, prompts, and messages

Strands defaults to Amazon Bedrock but does not require AWS when another provider is selected. The [current provider matrix](https://strandsagents.com/docs/user-guide/concepts/model-providers/) shows a shared core and substantial language-only coverage. Provider adapters normalize the loop interface; they do not erase differences in tool schema, multimodality, parallel calls, reasoning, caching, guardrails, built-in tools, stop reasons, or usage.

Structured output is implemented through a generated tool/schema interaction using Pydantic in Python and Zod in TypeScript. Provider JSON Schema support remains a constraint. Bedrock strict tool use, for example, does not accept every JSON Schema construct. Therefore schema validation and retry behavior need provider-specific integration tests.

Message-list input is especially security-sensitive. A caller able to supply assistant/tool-use history can influence the loop differently from a caller supplying plain user text. The official prompt guidance warns that untrusted complete message histories can forge tool calls or tool results. Applications should construct roles and tool-use records server-side.

Provider-specific evidence checked:

- [Amazon Bedrock](https://strandsagents.com/docs/user-guide/concepts/model-providers/amazon-bedrock/): IAM invocation actions, streaming/non-streaming normalization, prompt caching, guardrail configuration, and region/session configuration.
- [OpenAI Responses](https://strandsagents.com/docs/user-guide/concepts/model-providers/openai/): stateful response chaining, hosted tools, and output-mapping limitations. Python uses a distinct Responses adapter; TypeScript selects the API on its OpenAI model.
- [Custom providers](https://strandsagents.com/docs/user-guide/concepts/model-providers/custom_model_provider/): implement the Strands stream event grammar and error/usage normalization.

### Tools, ToolContext, and MCP

Local tools execute in the application process with its permissions by default. Current core vended tools can route some shell/file operations through a configured Strands sandbox, but the agent loop itself remains trusted application code. No sandbox means host execution.

Python uses annotated callables/classes/modules; TypeScript supports Zod-backed tools and JSON Schema-backed tools. Zod provides runtime validation and inferred callback types; plain JSON Schema does not give the callback the same validation/type guarantees. Both expose `ToolContext` concepts for agent access, tool-use identity/input, invocation state, cancellation, and interrupts. Model-visible arguments must not carry secrets or authoritative identity.

MCP is supported in both languages with stdio, Streamable HTTP, and SSE transports, filtering/prefixing, multiple servers, and elicitation. Important boundaries:

- prefixes rename tools only at the agent boundary;
- filtering reduces exposure but is not remote authorization;
- authentication and tenant binding remain transport/application responsibilities;
- remote cancellation is best effort;
- MCP progress notifications are currently Python-only;
- current Python source contains experimental MCP Tasks support aligned to the 2025-11-25 protocol revision; it requires client opt-in and server/tool capability and has no TypeScript counterpart in the checked snapshot.

Primary evidence: [MCP tools](https://strandsagents.com/docs/user-guide/concepts/tools/mcp-tools/), current MCP client/task source and tests, [ToolContext](https://strandsagents.com/docs/user-guide/concepts/tools/custom-tools/#toolcontext), and [sandbox](https://strandsagents.com/docs/user-guide/concepts/sandbox/).

### State, sessions, and memory

The SDK has several independent state planes:

| Plane | Model-visible | Lifetime | Suitable for |
|---|---:|---|---|
| Messages | Yes | conversation/session | reasoning context |
| Agent state | No, unless copied into a prompt/tool result | agent/session | JSON application state |
| Invocation state | No, unless a tool exposes it | one invocation | request-scoped clients, identity, deadlines |
| Session persistence | Restores selected SDK state | storage-dependent | resuming conversations and orchestrator snapshots |
| Memory | Search/injection-dependent | cross-session | tenant-scoped facts or knowledge |

Current storage includes in-memory, local-file, and S3 backends plus custom interfaces. Local-file writes use temporary-file replacement for a single write, but neither local nor S3 session storage supplies a distributed per-session transaction/lease. The safe inference from the API and source is **one live writer per session and agent/orchestrator identity**, enforced by the application when more than one process can serve it.

TypeScript's snapshot session system uses immutable UUIDv7 snapshots. Current Python includes a newer snapshot manager while older Python session-manager APIs are being deprecated/migrated. Storage layouts and persistence triggers differ; migration must be tested, not assumed.

Multi-agent session managers persist orchestrator state, not a universally durable transcript for every child agent. A session snapshot is not an exactly-once log for external side effects.

Memory extraction can be asynchronous and at least once. Current Python synchronous invocation flushes pending extraction at an invocation boundary, while asynchronous/streaming applications must flush on shutdown; TypeScript requires an explicit flush. Partial store failures also differ between search and add. Tenant scope must come from trusted application identity.

Primary evidence: [Sessions](https://strandsagents.com/docs/user-guide/concepts/agents/session-management/), [State](https://strandsagents.com/docs/user-guide/concepts/agents/state/), [Memory](https://strandsagents.com/docs/user-guide/concepts/memory/overview/), current storage/session/memory source and tests.

### Streaming, hooks, and telemetry

Python streaming yields dictionary-shaped events; TypeScript yields typed event objects. Their taxonomies are related but not wire-compatible. TypeScript events expose `toJSON()` and intentionally omit some runtime references; Python callers must define their own serialization allowlist. An application exposing SSE or WebSocket must add a stable envelope, event IDs, versioning, buffering, disconnect cancellation, and a final settlement record.

Hooks run in the critical path and can observe or mutate invocation/model/tool behavior. After-events run in reverse registration order within an order band. Mutation conflicts, hook exceptions, and autonomous resume behavior all require tests and limits. Hooks are useful for policy integration but are not a substitute for service-side authorization.

OpenTelemetry spans can contain prompts, system instructions, assistant responses, tool arguments/results, and token/cache usage. These are sensitive data. The local execution result also carries trace/metric summaries, with serialization differences between languages. Redact at creation/export, not only in a dashboard.

Primary evidence: [Streaming](https://strandsagents.com/docs/user-guide/concepts/streaming/), [Hooks](https://strandsagents.com/docs/user-guide/concepts/agents/hooks/), [Observability](https://strandsagents.com/docs/user-guide/observability-evaluation/observability/), and current event/telemetry source.

### Multi-agent patterns

| Pattern | Control | Best fit | Principal limitation |
|---|---|---|---|
| Agent with tools | One model loop | default | one context/authority surface |
| Agents as tools | manager delegates | specialization with central control | delegated context and side effects still need limits |
| Graph | explicit nodes and edges | known topology and joins | Python and TypeScript semantics differ |
| Swarm | model-selected peer handoff | exploratory collaboration | less deterministic, potentially costly |
| A2A | remote-agent boundary | separately deployed agents | network identity, trust, compatibility |

Graph parity requires special care. In the checked release, Python and TypeScript differ in join readiness, node-context preservation, scheduling, failure/cancellation status, and limit defaults. Python treats incoming dependency readiness with OR-like batch semantics; TypeScript uses AND joins. Python accumulates node context unless reset; TypeScript snapshots/restores by default unless context preservation is enabled. TypeScript orchestration limit defaults include multiple `Infinity` values. These are behavioral differences, not naming trivia.

Swarm also differs: handoff payload, shared context, stop/failure behavior, resume details, and default limits are not identical. Neither orchestration is automatically a durable business workflow. The [open deterministic-resume issue #2796](https://github.com/strands-agents/harness-sdk/issues/2796) is bounded evidence that interrupt resumes can still require an avoidable model round trip; it is not evidence that all resume is broken.

Primary evidence: [Multi-agent patterns](https://strandsagents.com/docs/user-guide/concepts/multi-agent/multi-agent-patterns/), [Graph](https://strandsagents.com/docs/user-guide/concepts/multi-agent/graph/), [Swarm](https://strandsagents.com/docs/user-guide/concepts/multi-agent/swarm/), current source/tests.

### Security and human control

Security controls operate at different layers:

- provider guardrails classify/filter model input or output;
- interventions can proceed, deny, guide, confirm, or transform lifecycle activity;
- HITL interrupts request an external decision;
- Cedar can express deterministic authorization policy;
- sandboxes/process boundaries constrain execution impact;
- domain services enforce principal/resource authorization and idempotency.

The [AWS security guidance for extending Bedrock Guardrails to tool interactions](https://aws.amazon.com/blogs/security/extend-amazon-bedrock-guardrails-to-tool-interactions-using-the-strands-agents-sdk/) explicitly reinforces that model guardrails do not automatically govern tool interactions. Confirmation tied only to a tool name is also insufficient for sensitive operations: bind approval to exact normalized arguments, principal, resource, expiry, and operation ID.

The core harness repository showed no published security advisory in its GitHub Security view at research time. The separate `strands-agents-tools` package had recent, material advisories and must be managed independently:

| Advisory | Affected | Fixed | Lesson |
|---|---:|---:|---|
| [CVE-2026-78379](https://aws.amazon.com/security/security-bulletins/2026-089-aws/) Python REPL consent bypass through batch | `<0.8.5` | `0.8.5` | model-visible control flags and prompt consent are not a security boundary |
| [CVE-2026-19111](https://github.com/strands-agents/tools/security/advisories/GHSA-mpxq-953j-42m4) memory namespace authorization bypass | `<0.8.3` | `0.8.3` | never let the model choose a tenant namespace |
| [CVE-2026-18394](https://github.com/strands-agents/tools/security/advisories/GHSA-qhw6-2h72-m84v) HTTP proxy credential exposure | `<0.8.2` | `0.8.2` | outbound URL and credential forwarding require strict policy |
| [CVE-2026-18733](https://aws.amazon.com/security/security-bulletins/2026-072-aws/) shell consent bypass | `<0.8.0` | `0.8.0` | interactive consent around host execution is vulnerable to prompt/control confusion |
| [CVE-2026-15746](https://github.com/strands-agents/tools/security/advisories/GHSA-ppcf-fpr3-x46v) Elasticsearch credential disclosure | `<0.7.0` | `0.7.0` | tool errors/results must not expose connector secrets |

Upgrade beyond the patched versions and review the complete current advisory list; a minimum fixed version is not a blanket safety claim.

### Testing and evaluation

The monorepo's Python tests use a scripted `MockedModelProvider`; TypeScript exposes `TestModelProvider` fixtures for exact `ModelStreamEvent` sequences. These are repository test helpers, not necessarily stable public production APIs, but they demonstrate the right testing seam: drive the loop with deterministic provider events and assert messages, calls, state, stop reason, hooks, and telemetry.

The separate [Strands Evaluation SDK](https://strandsagents.com/docs/user-guide/evals-sdk/quickstart/) is Python-first in the checked snapshot. It supports deterministic evaluators, LLM-as-judge evaluators, output/trajectory/tool/interaction evaluation, trace providers, simulators, red teaming, chaos tests, and experiment serialization. LLM judges are variable and can share bias with the system under test; they should supplement, not replace, deterministic security and business assertions.

### Deployment and cost

The official docs cover process/container deployment plus Lambda, Fargate, App Runner, EC2, EKS/Kubernetes, Terraform, and AgentCore. The right choice depends more on duration, streaming, connection state, isolation, and operational ownership than on the SDK.

AgentCore Runtime is an optional hosting platform. Current AWS documentation states:

- a runtime session receives a dedicated isolated microVM;
- the application must enforce user-to-session mapping;
- microVM session compute is ephemeral unless explicit storage/memory services are used;
- default idle termination is 15 minutes and maximum microVM lifetime is 8 hours, both subject to current configuration limits;
- a concurrent lifecycle operation can return retryable HTTP 409;
- new-session creation has a documented quota;
- WebSocket and HTTP streaming have explicit runtime contracts.

Sources: [isolated sessions](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-sessions.html), [quotas](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/bedrock-agentcore-limits.html), [WebSocket runtime](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-get-started-websocket.html), [pricing](https://aws.amazon.com/bedrock/agentcore/pricing/).

Lambda remains useful for short bounded invocations. AWS currently limits a Lambda invocation to 15 minutes. Response streaming has protocol/region/runtime constraints, and a disconnected client does not automatically stop billed execution. The current Strands Lambda guide's simple Python example is buffered and points streaming users toward container options. Source: [Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html), [Lambda response streaming](https://docs.aws.amazon.com/lambda/latest/dg/configuration-response-streaming.html), [Strands Lambda guide](https://strandsagents.com/docs/user-guide/deploy/deploy_to_aws_lambda/).

Cost is not “tokens only.” Include model input/output and cache pricing, loop amplification, orchestration fan-out, hosted-tool calls, guardrails, memory extraction/search, MCP/domain API charges, runtime CPU/peak memory, storage, observability ingestion, and network transfer. AgentCore pricing is consumption-based and mutable; store price inputs with cost reports rather than baking rates into code.

## Important disagreements and ambiguities

1. **Default Bedrock model descriptions can lag.** Different current pages/changelogs referenced different Claude defaults during research. Production guidance therefore pins a model ID and treats defaults as development convenience.
2. **Retry maximum delay appeared inconsistent between some docs/source views.** Six attempts and four-second initial delay were consistent; maximum-delay details varied by language/version. The guide tells readers to inspect their installed strategy rather than rely on a cross-language constant.
3. **“Session persistence” can sound stronger than source guarantees.** Storage restores framework state, but there is no general distributed transaction or exactly-once tool-effect contract.
4. **Evaluation is documented beside the SDK but packaged separately and currently Python-oriented.** It is not a TypeScript parity guarantee.
5. **Experimental features move rapidly.** Python checkpointing, MCP Tasks, bidirectional agents, and some vended plugins must be version-gated and contract-tested.

## Bounded issue evidence

Only a few issues were retained, and only for their narrow claim:

- [#2796](https://github.com/strands-agents/harness-sdk/issues/2796): deterministic interrupt resume without an LLM call is an open design gap.
- [#1230](https://github.com/strands-agents/harness-sdk/issues/1230): an open proposal documents scaling pain in older linear S3 session history; it does not prove all current snapshot storage has the same behavior.
- [#1671](https://github.com/strands-agents/harness-sdk/issues/1671): a reported Bedrock Guardrail/tool-result interaction is a provider-integration edge to regression-test, not a universal failure.
- [#762](https://github.com/strands-agents/harness-sdk/issues/762): custom tool executors remained a requested/planned surface in the checked line; use supported concurrent/sequential executors unless current docs say otherwise.

Closed historical bugs were not generalized into current defects. Open issues are not used as sole evidence for API behavior.

## Resulting editorial decisions

- Lead with the library/platform and state/durability boundaries.
- Describe Python and TypeScript separately whenever execution semantics differ.
- Treat provider adapters as compatibility surfaces that require contract tests.
- Treat all powerful tools as privileged code, independent of their packaging.
- Keep security advisories for the separate tools package visibly separate from core SDK status.
- Recommend single-agent designs first and require evidence before adding multi-agent fan-out.
- Avoid exact mutable cloud prices in design recommendations; link the dated official pricing source and describe the cost model.

## Refresh triggers

Re-run this research when:

- Python or TypeScript reaches a new major release;
- the feature matrix, default provider/model, event grammar, storage system, or Graph/Swarm semantics changes;
- experimental checkpointing or MCP Tasks stabilizes;
- the Evaluation SDK adds a supported TypeScript package;
- AgentCore changes session lifetime, isolation, concurrency, quota, storage, protocol, or billing rules;
- core or tools security advisories are published;
- a cited issue closes with an implementation that changes the described limitation.

## Primary source index

- [Strands documentation](https://strandsagents.com/docs/)
- [Strands monorepo](https://github.com/strands-agents/harness-sdk)
- [Strands release feed](https://github.com/strands-agents/harness-sdk/releases)
- [Python PyPI package](https://pypi.org/project/strands-agents/)
- [TypeScript npm package](https://www.npmjs.com/package/@strands-agents/sdk)
- [Evaluation quickstart](https://strandsagents.com/docs/user-guide/evals-sdk/quickstart/)
- [AgentCore developer guide](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/what-is-bedrock-agentcore.html)
- [AgentCore pricing](https://aws.amazon.com/bedrock/agentcore/pricing/)
- [AWS Security Bulletins](https://aws.amazon.com/security/security-bulletins/)
- [Strands Agents Tools advisories](https://github.com/strands-agents/tools/security/advisories)

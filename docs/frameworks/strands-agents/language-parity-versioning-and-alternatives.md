# Language Parity, Versioning, Limitations, and Alternatives

> Research date: **2026-08-31** · Release snapshot: Python 1.54.0 / TypeScript 1.14.0

Python and TypeScript Strands share architecture and a monorepo, not lockstep versions or behavior. Choose a language for the application ecosystem, then verify the exact features and semantics that matter.

## Release lineage

The official code is consolidated in [`strands-agents/harness-sdk`](https://github.com/strands-agents/harness-sdk). The earlier standalone SDK repositories and articles can still rank highly in search; the former TypeScript repository is archived. Current package names remain:

- Python: `strands-agents`
- TypeScript: `@strands-agents/sdk`

At the research snapshot, Python was 1.54.0 and TypeScript 1.14.0. The different numbers are normal. Compare changelogs and the feature matrix rather than versions numerically.

## Parity matrix

This matrix records meaningful differences found in current docs/source. It is a dated snapshot, not a permanent promise.

| Area | Python | TypeScript |
|---|---|---|
| Core invocation/streaming/structured output | yes | yes |
| Runtime | Python 3.10+ | Node.js 20+ |
| Provider breadth | broader; includes Ollama, LiteLLM, SageMaker and others | narrower; includes Vercel-specific path |
| OpenAI Responses | distinct Responses model | OpenAI model selects Responses/Chat |
| Streaming API | dictionary events; optional callback handler | typed class events with `toJSON()` |
| External cancellation | `threading.Event` | `AbortSignal` |
| Concurrent same-agent invocation | throws; unsafe reentrant and in-flight idempotency option | throws; no checked equivalent options |
| Retry/backoff surface | model retry strategy/custom hooks | exponential/linear/constant + jitter strategies |
| MCP progress | yes | not in checked snapshot |
| Experimental MCP Tasks | yes | no |
| Session evolution | classic managers plus newer snapshot path | snapshot-first immutable history |
| Memory flush | sync invocation can flush; async/stream needs shutdown flush | explicit flush required |
| Graph joins/context/status | OR-like batch readiness; context accumulates; cancellation often failed | AND joins; context snapshot by default; cancelled status |
| Swarm payload/limits | handoff tool/shared context; finite key defaults | structured routing/context; important defaults `Infinity` |
| Evaluation SDK | official separate Python package | no equivalent official TS package checked |
| Cedar namespaces | more limited in checked snapshot | namespace support documented |

Provider, vended-tool, sandbox, and experimental feature parity changes faster than the core. Link architecture decisions to a tested package lock, not this table alone.

## Compatibility strategy

Maintain an application capability manifest:

```yaml
runtime: python
strands_version: 1.54.0
model_provider: bedrock
model_id: explicit-model-id
features:
  structured_output: true
  parallel_tools: false
  session_format: app-v3
  mcp_protocol: "2025-11-25"
  graph_join_semantics: python-batch-or
experimental:
  checkpointing: false
  mcp_tasks: false
```

The manifest should be machine-tested at startup/CI where possible. Persist its version with sessions and evaluation reports.

### Upgrade procedure

1. Read language-specific changelog and migration notes.
2. Diff dependency locks, including provider/MCP/Pydantic/Zod/OTel/AWS clients.
3. Run deterministic loop, hook, tool, session, and orchestration contracts.
4. Test old snapshot load and rollback compatibility.
5. Run live provider contracts and representative evaluations.
6. Review event/wire redaction and telemetry attributes.
7. Scan core and separate tools advisories.
8. Canary by deployment/prompt/model/tool cohort.
9. Keep rollback from writing an unreadable session format.

Never use an unpinned default model in a compatibility test. A model alias change can look like an SDK regression.

## Current limitations to design around

- The SDK is in-process; it does not supply hosted admission, a scheduler, database, or public wire protocol.
- Session persistence is not distributed concurrency control or exactly-once workflow execution.
- Cancellation is cooperative and cannot undo completed concurrent effects.
- Provider normalization does not create capability parity.
- Graph and Swarm semantics differ by language and require explicit limits.
- Model/hook/intervention retries can add unbounded cycles if application budgets are absent.
- Local and community tools can execute with host privileges.
- Structured output is provider/schema dependent and can require repair turns.
- Memory extraction is eventually consistent and can lose unflushed recent work.
- Evaluation is probabilistic; the official evaluation package is separate and Python-oriented.
- Experimental surfaces—including checkpointing, bidirectional agents, and MCP Tasks—need release gating.

## Choosing a language

Choose Python when the system benefits from its provider breadth, official evaluation tooling, Python data/ML ecosystem, or Python-only integrations. Choose TypeScript when the application is a Node/browser/web platform and typed streamed events, AbortSignal integration, and one-language frontend/backend development dominate.

Do not choose Python merely because its version number is higher, or TypeScript because its event types look safer. Either can be production-grade when the required capabilities are present and contract-tested.

For a mixed-language fleet, define application-owned schemas for:

- public stream events;
- session/domain checkpoints;
- tool/MCP contracts;
- evaluation cases and result reports;
- identity, operation IDs, and policy decisions.

Do not exchange raw SDK sessions or event objects across languages.

## When Strands is the right fit

Strands is a good fit when:

- a lightweight model-driven loop inside an existing Python/Node service is desirable;
- tools and model providers need a common harness;
- AWS/Bedrock integration is useful but provider flexibility remains important;
- hooks, telemetry, sessions, and optional orchestration meet the application needs;
- the team is willing to own deployment and durable business semantics.

It is a weaker fit when:

- the primary requirement is a visual low-code hosted builder;
- a cross-language transactional workflow/checkpoint engine is the core need;
- deterministic orchestration dominates and model-driven routing adds little;
- untrusted arbitrary code must run but no strong sandbox platform exists;
- the organization wants a fully managed opinionated platform rather than an in-process SDK.

## Alternatives by missing capability

Avoid framework popularity comparisons; start with the missing system property.

| Need | Practical alternative/complement |
|---|---|
| Deterministic business workflow, timers, compensation | Temporal, AWS Step Functions, or another durable workflow engine around Strands activities |
| Provider-neutral low-level model calls only | provider SDK or lightweight model gateway without an agent framework |
| Database-backed graph checkpoints/replay as core abstraction | evaluate graph-runtime frameworks specifically for their persistence semantics |
| Fully managed AWS agent hosting | AgentCore Runtime complements Strands; Bedrock Agents is a separate managed agent product to evaluate |
| Remote interoperable specialists | A2A/MCP/service APIs with explicit identity and schemas |
| Strong untrusted code execution | dedicated sandbox/microVM service; keep Strands in the trusted control plane |
| Language-neutral evaluation | application-owned case/trace format with a separate evaluator service |

Frameworks do not remove the need for domain idempotency, resource authorization, evaluation, and observability. Compare them on those integration seams.

## Decision checklist

- [ ] Required features exist in the chosen language and locked release.
- [ ] Provider/model capability contract passes.
- [ ] Session and orchestration semantics fit the recovery requirement.
- [ ] The team can own hosting, tenancy, authorization, and cost limits.
- [ ] Any durable workflow or sandbox need has a complementary system.
- [ ] Public schemas are application-owned and cross-language safe.
- [ ] Upgrade/rollback includes sessions, prompts, tools, providers, and policies.
- [ ] Experimental dependencies have an exit/disable plan.

## Sources

- [Official language feature matrix](https://strandsagents.com/docs/user-guide/quickstart/overview/)
- [Model provider matrix](https://strandsagents.com/docs/user-guide/concepts/model-providers/)
- [Consolidated monorepo](https://github.com/strands-agents/harness-sdk)
- [Release feed](https://github.com/strands-agents/harness-sdk/releases)
- [Python package](https://pypi.org/project/strands-agents/)
- [TypeScript package](https://www.npmjs.com/package/@strands-agents/sdk)

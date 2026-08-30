# Research Packet: Core Agent Loop and Runtime Boundary

> **Status:** Synthesis complete; supports research-backed draft guides  
> **Research window:** 2026-08-30  
> **Scope:** Agent/workflow distinction, loop phases, tool contracts, policy/effect boundaries, run controls, checkpointing, durable execution, idempotency, and first-order runtime failures

## Guides supported

- [Agentic systems: choose the minimum autonomy](../../foundations/agentic-systems.md)
- [The production agent loop](../../foundations/agent-loop.md)
- [Execution boundaries](../../runtime/execution-boundaries.md)
- [Run controls](../../runtime/run-controls.md)
- [Durable execution](../../runtime/durable-execution.md)
- [Tool contracts](../../tools/tool-contracts.md)
- [Idempotency and side effects](../../reliability/idempotency-and-side-effects.md)
- [Runtime failure taxonomy](../../reliability/failure-taxonomy.md)
- [Custom loop vs framework vs workflow engine](../../comparisons/custom-loop-vs-framework-vs-workflow-engine.md)

## Research questions

1. What distinguishes a deterministic workflow, a model-assisted workflow, and an agent in engineering terms?
2. Which phases exist in a production loop beyond “model → tool → model”?
3. Which state belongs in model context, session history, run control, and durable business records?
4. Where must authorization, validation, isolation, and effect execution occur?
5. What do framework limits, timeouts, cancellation, retries, and checkpoints actually bound?
6. What recovery guarantees can checkpointed agent frameworks and durable workflow engines honestly provide?
7. How should an agent avoid duplicate or stale side effects under retries, replay, parallelism, and approval waits?
8. Which failure modes recur across papers, official docs, source repositories, issues, and production reports?

## Search breadth

The packet examined the following source families rather than relying on a single framework vocabulary:

- provider engineering guidance and production reports from Anthropic, OpenAI, and Google;
- current loop, tool, run-control, state, and durability documentation from OpenAI Agents SDK, LangGraph, Pydantic AI, Vercel AI SDK, Strands, Google ADK, CrewAI, and LlamaIndex;
- durable-execution semantics from Temporal, Restate, DBOS, Prefect, and Pydantic AI integrations;
- protocol and telemetry specifications from MCP and OpenTelemetry;
- agent reliability benchmarks and papers including ReAct, AgentBench, and τ-bench;
- implementation issues and documentation edge cases involving infinite loops, retry-layer confusion, cancellation limits, and durable-schema failures;
- newer research on control-primitive enforcement gaps and commit-time authorization, treated as emerging evidence rather than settled practice;
- selected community reports used only to identify operational questions, not to establish universal claims.

## Synthesis: conclusions that held across sources

### 1. “Agent” is an execution-policy distinction, not a product label

Anthropic's [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) distinguishes predefined workflows from systems in which the model dynamically directs process and tool use. OpenAI's [practical guide](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/) similarly centers independent workflow execution. Current framework docs converge on a model/tool loop but differ in what the runner automates.

**Synthesis:** classify a system by who chooses the next action and how much authority that choice carries. A model call inside fixed code is not automatically an agent; an autonomous next-action policy without strong runtime boundaries is still an agent even if marketed as a “workflow.”

### 2. The useful production loop contains non-model gates

The minimal ReAct loop—reason, act, observe—remains a useful mental model ([ReAct paper](https://arxiv.org/abs/2210.03629)). Production SDKs add final-output classification, handoffs, tool execution, turn limits, approvals, guardrails, error conversion, and state updates. See the current [OpenAI Agents SDK runner lifecycle](https://openai.github.io/openai-agents-python/running_agents/), [Vercel loop controls](https://ai-sdk.dev/docs/agents/loop-control), [Strands agent loop](https://strandsagents.com/docs/user-guide/concepts/agents/agent-loop/), and [LangGraph overview](https://docs.langchain.com/oss/python/langgraph/overview).

**Synthesis:** model output is a proposal. Policy, authorization, validation, effect execution, result normalization, checkpointing, and completion verification are runtime responsibilities. Folding those responsibilities into prompt text weakens guarantees and observability.

### 3. Stop conditions are a budget system, not proof of success

SDKs expose different units: OpenAI's runner has model-turn limits; Vercel's `ToolLoopAgent` defaults to a step limit; LangGraph has a recursion/step guard; Pydantic AI separates transport, fallback, tool, output, and hook retry layers. Current documentation makes the differences explicit:

- [OpenAI running agents](https://openai.github.io/openai-agents-python/running_agents/)
- [Vercel AI SDK loop control](https://ai-sdk.dev/docs/agents/loop-control)
- [Pydantic AI retry layers](https://github.com/pydantic/pydantic-ai/blob/main/docs/retries.md)
- [LangGraph recursion error](https://github.com/langchain-ai/langgraph/blob/main/libs/langgraph/langgraph/errors.py)

**Synthesis:** use several independent budgets—model turns, total tool calls, per-tool attempts, elapsed time, tokens, cost, parallel width, and repeated-action detection. Exhausting a budget produces a bounded failure, not a successful answer.

### 4. Cancellation, timeout, approval, and durability are separate semantics

OpenAI's model timeout explicitly does not bound the full run or tool execution ([model docs](https://openai.github.io/openai-agents-python/models/)). Pydantic AI documents that Python cannot forcibly stop a synchronous worker thread and that same-process cancellation does not cross durable serialization boundaries ([advanced tools](https://github.com/pydantic/pydantic-ai/blob/main/docs/tools-advanced.md)). OpenAI distinguishes provider permission to emit parallel calls from SDK concurrency when executing them ([running agents](https://openai.github.io/openai-agents-python/running_agents/)).

Emerging research, [Stop Means Stop](https://arxiv.org/abs/2607.14166), reports framework control gaps such as sibling effects continuing during an approval pause, replay double-execution, cancellation orphans, and timeout zombies. This paper is recent and must be independently replicated, but its test questions are immediately useful.

**Synthesis:** define whether a control stops scheduling, requests cooperative cancellation, prevents commit, or guarantees all descendants are quiescent. Do not infer barrier semantics from a method name.

### 5. State has at least four distinct roles

Anthropic's [Managed Agents architecture](https://www.anthropic.com/engineering/managed-agents) separates the brain/harness, hands/tools, and a durable session event log. LangGraph checkpoints graph state for interrupts and recovery ([persistence](https://docs.langchain.com/oss/javascript/langgraph/persistence)). LlamaIndex explicitly separates conversation memory from workflow context ([memory source](https://github.com/run-llama/llama_index/blob/main/docs/src/content/docs/framework/module_guides/deploying/agents/memory.mdx)). OpenAI documents local/session-managed and server-managed conversation state separately ([conversation state](https://developers.openai.com/api/docs/guides/conversation-state)).

**Synthesis:** keep separate:

1. model context—the bounded view for one inference;
2. session/event history—the recoverable record of interaction;
3. run-control state—budgets, pending calls, approvals, leases, cancellation;
4. business/effect state—the authoritative external truth and receipts.

Conflating them causes compaction loss, stale approval, privacy leaks, or false recovery guarantees.

### 6. Checkpointing model state does not make external effects exactly once

LangGraph checkpoints enable restart from graph step boundaries and preserve pending writes, but its docs do not promise transactional atomicity with arbitrary external tools. Durable engines make the effect boundary more explicit:

- Temporal requires nondeterministic I/O in Activities and deterministic workflow code ([AI reference architecture](https://go.temporal.io/platform-hub/ai-engineering/ai-reference-architecture)).
- DBOS requires deterministic workflows and idempotent steps; steps have at-least-once behavior around incomplete work ([architecture](https://docs.dbos.dev/architecture), [concurrent executions](https://docs.dbos.dev/explanations/concurrent-executions)).
- Restate journals completed results so recovered code reuses them, while its deeper explanation acknowledges the ambiguity around an effect completing before its result is durable ([durable agents](https://docs.restate.dev/ai/patterns/durable-agents), [why Restate](https://restate.dev/blog/why-we-built-restate)).
- Pydantic AI supports several durable engines, revealing that durability is a capability attached to an agent, not a single universal semantic ([overview](https://pydantic.dev/docs/ai/capabilities/durable_execution/overview/)).

**Synthesis:** durable orchestration prevents lost progress and repeated completed steps. External side effects still need stable operation IDs, downstream idempotency/deduplication, effect receipts, or a transaction/outbox boundary.

### 7. Tool design is a contract for a nondeterministic caller

Anthropic's [tool engineering report](https://www.anthropic.com/engineering/writing-tools-for-agents) emphasizes choosing meaningful operations, clear namespaces, token-efficient results, and evaluation. OpenAI recommends strict schemas for function calls and distinguishes provider-side parallel calling ([function calling](https://developers.openai.com/api/docs/guides/function-calling)). Pydantic AI's retry docs show why error categories and budgets matter. τ-bench shows that real policy-following tool tasks remain inconsistent across repeated trials ([ICLR 2025 paper](https://proceedings.iclr.cc/paper_files/paper/2025/hash/1b126cc38b8638e07bef37e7b2bb72bf-Abstract-Conference.html)).

**Synthesis:** good tools expose domain intent rather than raw API sprawl, validate strictly, authorize at execution time, make side effects and retry safety explicit, return compact structured evidence, and distinguish retryable correction from terminal or policy failure.

### 8. Reliability must be measured across repeated trajectories

AgentBench identifies long-term reasoning, decision-making, and instruction following as recurring obstacles ([paper](https://arxiv.org/abs/2308.03688)). τ-bench introduced `pass^k` to penalize inconsistency and reported much lower repeated-run reliability than single-run success. SWE-bench Verified showed that both task curation and scaffold choice materially change results ([OpenAI report](https://openai.com/index/introducing-swe-bench-verified/)); the τ-bench leaderboard later removed a scaffold for test leakage ([HAL notice](https://hal.cs.princeton.edu/taubench_airline)).

**Synthesis:** evaluate final state, policy compliance, and trajectory across repeated seeds and failure injection. Treat model, prompt, tool definitions, harness, and runtime settings as one versioned system under test.

## Important disagreements and how this packet resolves them

| Apparent disagreement | Evidence | Resolution used in guides |
|---|---|---|
| Start with a framework vs start with direct API calls | Anthropic favors simple/direct composition; vendors recommend their higher-level SDKs for common orchestration | Start with the smallest abstraction that owns a real problem. A thin SDK runner can be simpler than a custom loop; a broad framework is not automatically simpler. |
| Agentic flexibility vs deterministic workflows | Anthropic, Vercel, Google ADK, and Microsoft materials all support both | Use code for known control flow and model judgment only where variation is valuable. Hybrid systems are the default production shape. |
| Human approval as primary safety vs environmental containment | SDKs expose approvals; Anthropic production reports document approval fatigue and containment benefits | Use approval for semantic/user-intent decisions and containment/least privilege to cap damage. Revalidate authorization at commit. |
| Parallel guardrails/tools for latency vs blocking gates for safety | OpenAI documents both latency and side-effect trade-offs | Parallelize reads and advisory checks; block before irreversible or sensitive effects. Define sibling behavior during pauses. |
| Framework checkpointing vs workflow-engine durability | Agent frameworks advertise persistence; workflow engines describe replay constraints and effect semantics | Treat framework checkpoints as state recovery unless stronger semantics are explicitly documented. Add a durable engine when waits, restarts, and effects demand it. |
| More turns/tools improve capability vs increase failure | Long-horizon harness reports show capability; AgentBench/τ-bench and issue reports show compounding failure | Increase horizon only with budgets, progress evidence, recovery, and repeated evals. |

## Failure evidence sampled

| Failure | Evidence | How it influenced guidance |
|---|---|---|
| Repeated tool calls / no completion | [LangGraph issue #6731](https://github.com/langchain-ai/langgraph/issues/6731), [issue #3097](https://github.com/langchain-ai/langgraph/issues/3097) | Require structural limits and repeated-action detection; prompts are not stop controls. |
| Retry budgets compose unexpectedly | [Pydantic AI retry documentation](https://github.com/pydantic/pydantic-ai/blob/main/docs/retries.md) | Inventory retry layers and impose a run-level budget. |
| Synchronous tools outlive cancellation | [Pydantic AI advanced tools](https://github.com/pydantic/pydantic-ai/blob/main/docs/tools-advanced.md) | Use cooperative tokens/process isolation and a commit fence. |
| Durable engine retries deterministic bad tool input | [Pydantic AI issue #6979](https://github.com/pydantic/pydantic-ai/issues/6979) | Error classification must survive serialization boundaries; engine retry and model correction are different. |
| Parallel approval/cancellation leakage | [Stop Means Stop](https://arxiv.org/abs/2607.14166) | Treat barrier semantics as a testable invariant, not an API-name assumption. |
| Approval fatigue | [Anthropic containment report](https://www.anthropic.com/engineering/how-we-contain-claude), [auto mode report](https://www.anthropic.com/engineering/claude-code-auto-mode) | Prefer narrow authority and containment; reserve human review for legible, consequential choices. |
| Context-only continuity degrades long tasks | [Effective harnesses](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) | Persist explicit progress and handoff artifacts outside model context. |
| Multi-agent coordination adds production complexity | [Anthropic multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) | Keep one accountable control plane and instrument delegation; multi-agent is not the default. |

An issue report demonstrates a real reported condition in a particular version/configuration; it does not establish prevalence. The guides turn these reports into tests and safeguards rather than universal accusations about a framework.

## Production measurements used cautiously

- Anthropic reports that decoupling brain, hands, and session reduced p50 time-to-first-token by roughly 60% and p95 by over 90% in its Managed Agents architecture. This is a vendor-specific architecture result, used to support separation as a plausible performance and isolation benefit—not a general forecast.
- Anthropic reports users approved roughly 93% of Claude Code permission prompts and sandboxing reduced prompts by 84% in internal use. These measurements motivate approval-fatigue analysis but do not define every application's thresholds.
- The Anthropic parallel compiler experiment used 16 agents, nearly 2,000 sessions, and about $20,000 in API cost. It demonstrates possible long-horizon scale and cost, not a recommended default topology.
- τ-bench's original reported results showed low repeated reliability on its retail/airline tasks. The repository uses its `pass^k` lesson, not old model rankings.

## Claims deliberately excluded or narrowed

- **“Exactly once tools.”** Excluded unless the downstream effect system participates in deduplication/transaction semantics. Durable orchestration alone is insufficient.
- **“Human approval makes a tool safe.”** Narrowed to one control; approval may be fatigued, stale, uninformed, or bypassed by parallel siblings.
- **“Structured output means valid action.”** Rejected. Schema validity does not establish authorization, business validity, freshness, or safety.
- **“More tools increase capability.”** Rejected as universal; large catalogs consume context and confuse selection. Lazy discovery can help but has its own ranking/security surface.
- **“Multi-agent is more capable.”** Kept conditional. It can scale search or context isolation, while adding tokens, latency, coordination, correlated error, and synthesis risk.
- **“Resume exactly where it stopped.”** Avoided without naming the checkpoint/step boundary and unfinished-effect behavior.
- **Current framework rankings.** Excluded. Versions and models change quickly, and available comparisons rarely control scaffold, provider, workload, and operational semantics together.

## Primary and direct sources reviewed

### Cross-provider architecture and production reports

- Anthropic, [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents), published 2024-12-19.
- OpenAI, [A practical guide to building agents](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/), current page checked 2026-08-30.
- Google Cloud, [Core concepts of AI agents](https://cloud.google.com/resources/core-concepts-ai-agents), checked 2026-08-30.
- Anthropic, [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents), published 2025-09-29.
- Anthropic, [Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents), published 2025-09-11.
- Anthropic, [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents), published 2025-11-26.
- Anthropic, [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system), published 2025-06-13.
- Anthropic, [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps), published 2026-03-24.
- Anthropic, [Scaling Managed Agents: Decoupling the brain from the hands](https://www.anthropic.com/engineering/managed-agents), published 2026-04-08.
- Anthropic, [How we contain Claude across products](https://www.anthropic.com/engineering/how-we-contain-claude), published 2026-05-25.
- Anthropic, [Beyond permission prompts](https://www.anthropic.com/engineering/claude-code-sandboxing), published 2025-10-20.
- Anthropic, [Claude Code auto mode](https://www.anthropic.com/engineering/claude-code-auto-mode), published 2026-03-25.

### Current SDK and framework mechanics

- OpenAI, [Agents SDK runner lifecycle](https://openai.github.io/openai-agents-python/running_agents/), checked 2026-08-30.
- OpenAI, [Agents SDK tools](https://openai.github.io/openai-agents-python/tools/), checked 2026-08-30.
- OpenAI, [Agents SDK human in the loop](https://openai.github.io/openai-agents-python/human_in_the_loop/), checked 2026-08-30.
- OpenAI, [Agents SDK guardrails](https://openai.github.io/openai-agents-python/guardrails/), checked 2026-08-30.
- OpenAI, [Function calling](https://developers.openai.com/api/docs/guides/function-calling), checked 2026-08-30.
- OpenAI, [Conversation state](https://developers.openai.com/api/docs/guides/conversation-state), checked 2026-08-30.
- Vercel, [AI SDK agents overview](https://ai-sdk.dev/docs/agents/overview) and [loop control](https://ai-sdk.dev/docs/agents/loop-control), checked 2026-08-30.
- LangChain, [LangGraph overview](https://docs.langchain.com/oss/python/langgraph/overview) and [persistence](https://docs.langchain.com/oss/javascript/langgraph/persistence), checked 2026-08-30.
- Pydantic, [Agents](https://pydantic.dev/docs/ai/core-concepts/agent/), [retries](https://github.com/pydantic/pydantic-ai/blob/main/docs/retries.md), and [advanced tools](https://github.com/pydantic/pydantic-ai/blob/main/docs/tools-advanced.md), checked 2026-08-30.
- Strands, [Agent loop](https://strandsagents.com/docs/user-guide/concepts/agents/agent-loop/) and [plugins](https://strandsagents.com/docs/user-guide/concepts/plugins/), checked 2026-08-30.
- LlamaIndex, [Memory vs workflow context](https://github.com/run-llama/llama_index/blob/main/docs/src/content/docs/framework/module_guides/deploying/agents/memory.mdx), checked 2026-08-30.
- Google, [ADK](https://adk.dev/), checked 2026-08-30.

### Durable execution

- Temporal, [AI Agent Reference Architecture](https://go.temporal.io/platform-hub/ai-engineering/ai-reference-architecture), checked 2026-08-30.
- Temporal, [Platform documentation](https://docs.temporal.io/), checked 2026-08-30.
- Restate, [Durable Agents](https://docs.restate.dev/ai/patterns/durable-agents) and [Why we built Restate](https://restate.dev/blog/why-we-built-restate), checked 2026-08-30.
- DBOS, [Architecture](https://docs.dbos.dev/architecture) and [Concurrent executions](https://docs.dbos.dev/explanations/concurrent-executions), checked 2026-08-30.
- Pydantic AI, [Durable execution overview](https://pydantic.dev/docs/ai/capabilities/durable_execution/overview/), checked 2026-08-30.
- Prefect, [Agent orchestration overview](https://www.prefect.io/solutions/agents), checked 2026-08-30; product claims treated as discovery until deeper docs review.

### Research and evaluation

- Yao et al., [ReAct](https://arxiv.org/abs/2210.03629), ICLR 2023.
- Liu et al., [AgentBench](https://arxiv.org/abs/2308.03688), ICLR 2024.
- Yao et al., [τ-bench](https://proceedings.iclr.cc/paper_files/paper/2025/hash/1b126cc38b8638e07bef37e7b2bb72bf-Abstract-Conference.html), ICLR 2025.
- OpenAI, [Introducing SWE-bench Verified](https://openai.com/index/introducing-swe-bench-verified/), published 2024.
- [Stop Means Stop](https://arxiv.org/abs/2607.14166), 2026 preprint; emerging evidence.
- [Temporary Authority, Permanent Effects](https://arxiv.org/abs/2607.10487), 2026 preprint; emerging commit-time authorization model.

## Open questions

1. How consistently do current frameworks enforce cancellation and approval barriers across parallel/nested branches under controlled tests?
2. Which durable integrations preserve streamed-event ordering and approval state across version upgrades?
3. What is the best portable effect-receipt schema across local tools, MCP, and remote agents?
4. How should run-level budgets compose across nested agents without unfairly starving legitimate parallel work?
5. Which completion-verification strategies improve reliability without excessive verifier cost or blocking valid partial outcomes?
6. How should telemetry represent model proposals, policy decisions, attempted effects, committed effects, and compensations without recording sensitive content?

## Refresh triggers

- Major OpenAI Agents SDK, LangGraph, Pydantic AI, Vercel AI SDK, Strands, Temporal, Restate, or DBOS semantic changes.
- Independent replication or rebuttal of the 2026 control-primitive enforcement findings.
- New benchmark evidence that controls the model while varying harness/runtime.
- MCP or A2A changes that affect effect identity, authorization, cancellation, or task state.
- A production incident that invalidates any run-control or recovery recommendation.
- Scheduled recheck by 2026-11-30.


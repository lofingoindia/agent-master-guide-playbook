# Research Packet: Orchestration, Delegation, and Agent Protocols

> **Status:** Evidence synthesis for the orchestration and protocols guides  
> **Research window:** 2026-08-30  
> **Scope:** Planning and replanning, multi-agent topologies, delegation and handoffs, shared state, MCP, A2A, and AG-UI.  
> **Method:** Current specifications and official framework documentation were used as the baseline, then checked against research papers, benchmark studies, security analysis, and production retrospectives. Vendor benchmark numbers are retained only with their experimental scope.

## Questions investigated

1. When should control flow live in code, a workflow graph, a model-generated plan, or a multi-agent topology?
2. Which task properties predict whether more agents help rather than multiply cost and error?
3. How do manager delegation, handoffs, routing, parallel workers, and group collaboration differ operationally?
4. What state, authority, budget, cancellation, and evidence must cross a delegation boundary?
5. Which boundary does MCP, A2A, or AG-UI standardize—and what does each deliberately leave to the application?
6. What changed in the current protocol revisions, and which older implementation advice is now stale?
7. Which claims are stable production guidance versus promising but still emerging research?

## Search and review path

The review used multiple passes rather than a single result set:

- official OpenAI Agents SDK, Anthropic, Google ADK, Microsoft Agent Framework, AutoGen, and LangChain architecture and orchestration documentation;
- current MCP specification, release notes, authorization guidance, and repository security policy;
- current A2A specification, discovery documentation, samples, and task lifecycle semantics;
- AG-UI architecture, event, state, tool, capability, and core type documentation;
- ReAct, ReWOO, LLMCompiler, LATS, PlanBench, MultiAgentBench, MAST, Magentic-One, and recent scaling/collaboration studies;
- production retrospectives on multi-agent research and security work on agent-to-agent prompt propagation, prompt hardening, and bounded delegation;
- comparison searches for supervisor vs handoff, graph vs dynamic routing, parallel vs sequential workloads, cancellation, webhook delivery, discovery trust, and shared-state failure.

Research stopped after additional searches mostly repeated the same boundary conditions: task decomposability and dependency density dominate multi-agent outcomes; delegation needs an application-owned control contract; and interoperability protocols do not confer trust, authorization, or durable execution.

## Evidence matrix: orchestration and planning

| Source | Evidence extracted | Practical interpretation | Maturity / caveat |
|---|---|---|---|
| [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) | Separates predictable workflows from open-ended agents; describes chaining, routing, parallelization, orchestrator-workers, and evaluator-optimizer patterns | Start with the least dynamic topology that passes evaluations; add model-directed control only where uncertainty requires it | Production-oriented guidance; examples are illustrative rather than universal benchmarks |
| [OpenAI Agents SDK: Multi-agent orchestration](https://openai.github.io/openai-agents-python/multi_agent/) | Distinguishes agents-as-tools from handoffs and model orchestration from code orchestration | Manager-as-tools preserves central ownership; handoff transfers the active specialist role; code control is easier to constrain and predict | Framework-specific API, but the ownership distinction generalizes |
| [OpenAI Agents SDK: Handoffs](https://openai.github.io/openai-agents-python/handoffs/) | Handoffs are exposed as tools and can filter history, validate typed input, and be enabled dynamically | A handoff is a controlled transfer, not permission to forward the whole transcript or every tool | Current official OpenAI documentation as of research date |
| [Microsoft Agent Framework: Workflows](https://learn.microsoft.com/en-us/agent-framework/concepts/workflows/) | Explicit graphs support fan-out/fan-in, state, events, and checkpoints | Known dependencies belong in a workflow graph; make joins, retries, and checkpoints visible | Documentation updated in 2026; product surface can continue evolving |
| [Microsoft Agent Framework: Orchestrations](https://learn.microsoft.com/en-us/agent-framework/workflows/orchestrations/) | Provides sequential, concurrent, handoff, group-chat, and Magentic patterns | Topology is a workload choice, not a maturity ladder | Framework taxonomy; implementation semantics must still be verified |
| [Google ADK: Workflows](https://adk.dev/workflows/) | ADK 2.0 exposes graph, dynamic, collaborative, and template workflow types | Separate explicit dependency graphs from model-chosen routes and collaborative execution | Python 2.0 GA in May 2026 and Go 2.0 GA in June 2026; language/version differences matter |
| [Google ADK: Collaborative workflows](https://adk.dev/workflows/collaboration/) | Task and single-turn modes can use isolated session branches and distinct return/advance behavior | Treat every worker as a scoped execution branch with an explicit result boundary | Current official documentation; runtime-specific |
| [LangChain: Subagents](https://docs.langchain.com/oss/python/langchain/multi-agent/subagents) | Supervisor calls isolated subagents as tools; synchronous/async execution and state visibility differ | Context isolation is useful, but required state must be intentionally projected and returned | Framework-specific; reinforces explicit context contracts |
| [LangChain: Handoffs](https://docs.langchain.com/oss/python/langchain/multi-agent/handoffs) | Handoff history and message/tool-call pairing require careful management | Invalid or over-shared history is a correctness risk, not merely a prompt-quality issue | Current official documentation |
| [AutoGen AgentChat](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/index.html) | Offers selector chat, swarm, Magentic-One, and experimental GraphFlow; recommends starting with one agent | Rich team abstractions do not remove the need for termination, scope, and topology evaluation | GraphFlow is explicitly experimental |
| [AutoGen team reference](https://microsoft.github.io/autogen/stable/reference/python/autogen_agentchat.teams.html) | A team without termination conditions can run indefinitely | Termination is an orchestrator invariant, never an assumed emergent behavior | Direct API warning |
| [Anthropic: Multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) | Breadth-first research benefited from an orchestrator and parallel workers; early systems spawned excessive agents/searches and suffered error compounding | Multi-agent execution primarily scales search/context/tool budget; cap fan-out and require compressed evidence returns | The reported 90.2% uplift and ~15× chat token use are specific to one product/evaluation |
| [Google: Science of scaling agent systems](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/) | Across 180 configurations, parallelizable work improved while sequential work often degraded; centralized systems contained errors better in the tested setting | Estimate decomposability, dependency density, and coordination cost before choosing more agents; central control is the safer default | 2026 study; percentages are configuration- and benchmark-specific |
| [MultiAgentBench](https://aclanthology.org/2025.acl-long.421/) | Evaluates coordination across multiple topologies and tasks | Evaluate topology and coordination behavior, not only final answer quality | Benchmark coverage is not production workload proof |
| [MAST: Why do multi-agent LLM systems fail?](https://openreview.net/pdf?id=fAjbYBmonr) | Taxonomy derived from more than 1,600 annotated traces across seven systems identifies system-design and inter-agent failures | Trace delegation, result integration, and role adherence explicitly; aggregate success hides coordination failure | Research taxonomy; map to local failure data before prioritizing |
| [Magentic-One](https://www.microsoft.com/en-us/research/publication/magentic-one-a-generalist-multi-agent-system-for-solving-complex-tasks/) | Orchestrator tracks progress, plans, and replans across specialized agents; logs expose systematic mistakes | A manager needs an explicit ledger and recovery policy, not only a “coordinator” prompt | Benchmark-driven research system |
| [Nature Machine Intelligence: multi-agent collaboration](https://www.nature.com/articles/s42256-026-01268-y) | Stronger individual models can reduce or reverse collaboration benefit due to communication and error propagation | Re-run topology evals when the base model changes; more capable agents may need fewer coordination layers | 2026 experimental study; workload-specific |
| [ReAct](https://arxiv.org/abs/2210.03629) | Interleaves reasoning, action, and observation | Plans should update from real observations rather than execute blindly | Foundational pattern; do not expose private reasoning as a protocol contract |
| [ReWOO](https://arxiv.org/abs/2305.18323) | Decouples an up-front plan from tool observations to reduce repeated model calls | Useful when dependencies are predictable; less adaptive when early assumptions fail | Research method, not a default production architecture |
| [LLMCompiler](https://icml.cc/virtual/2024/poster/32829) | Builds a dependency plan that can execute independent tool calls in parallel | Parallelize only edges proven independent and retain a join/error policy | Reported speed/cost gains are task- and implementation-specific |
| [LATS](https://icml.cc/virtual/2024/poster/33107) | Uses search, reflection, and environment feedback | Tree search may improve hard tasks but multiplies tokens, latency, and effect-control complexity | Experimental; keep away from irreversible effects without a strict simulation/proposal boundary |
| [PlanBench](https://arxiv.org/abs/2206.10498) | Exposes weaknesses in language-model planning | Treat generated plans as fallible hypotheses and evaluate constraint adherence separately | Planning benchmark, not an end-to-end agent benchmark |

## Evidence matrix: protocol boundaries

| Source | Current finding | Engineering consequence |
|---|---|---|
| [MCP 2026-07-28 release](https://blog.modelcontextprotocol.io/posts/2026-07-28/) and [release candidate notes](https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/) | Revision 2026-07-28 moves requests toward stateless metadata, adds discovery/header routing/cache controls and the Tasks extension, introduces MRTR, and deprecates several earlier mechanisms | Pin the negotiated revision; build compatibility tests; do not design new systems around legacy initialization/session headers or legacy HTTP+SSE |
| [MCP specification](https://modelcontextprotocol.io/specification/) | MCP standardizes the model/application-to-tool-and-data-server boundary | It is not a multi-agent orchestration, durable-workflow, or authorization policy system |
| [MCP authorization security considerations](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/basic/authorization/security-considerations.mdx) | Requires token audience validation, forbids token passthrough, calls for PKCE S256 and exact redirects, and covers SSRF/confused-deputy/issuer risks | Treat each server as its own resource server and exchange credentials intentionally; validate metadata URLs and redirect targets |
| [MCP repository security policy](https://github.com/modelcontextprotocol/modelcontextprotocol/security) | Local servers should be treated like installed software | Admission, code signing/provenance, sandboxing, egress, filesystem access, and updates remain operator responsibilities |
| [A2A specification 1.0.0](https://github.com/a2aproject/A2A/blob/main/docs/specification.md) | Defines AgentCard, Message, Part, Artifact, Task and lifecycle states across JSON-RPC, gRPC, and REST; supports streaming/push | Model a remote agent as an opaque external task service and keep a local task/effect ledger |
| [A2A agent discovery](https://github.com/a2aproject/A2A/blob/main/docs/topics/agent-discovery.md) | AgentCards support discovery and capability advertisement | Capability metadata is not identity, trust, authorization, or output validation; verify signatures and admission separately |
| [A2A samples](https://github.com/a2aproject/a2a-samples) | Samples explicitly warn that remote AgentCards and responses are untrusted | Treat cards, messages, files, and artifacts as attacker-controlled input |
| [AG-UI architecture](https://docs.ag-ui.com/concepts/architecture) | Defines a lightweight event-driven boundary between agent backend and user-facing application | Use it for live run/tool/state/interaction presentation, not as the durable business-state store |
| [AG-UI tools](https://docs.ag-ui.com/concepts/tools) | Allows frontend-defined tools and frontend participation in tool execution | “Frontend controlled” is not an authorization boundary; the backend must validate identity, scope, effect, and approval |
| [AG-UI capabilities](https://docs.ag-ui.com/concepts/capabilities) | Covers transport, tool, output, state, and human-in-the-loop capabilities including interrupts and edits | Negotiate capability/version and test reconnect, replay, deduplication, and stale UI behavior |
| [AG-UI core types](https://docs.ag-ui.com/sdk/js/core/types) | Run input includes thread/run/parent IDs, state, messages, tools, context, and forwarded properties | Validate and minimize every client-supplied field; correlation identifiers do not prove authority |
| [OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) | Defines evolving telemetry vocabulary for model and agent operations | Isolate schema mapping from business logic and pin the convention version; telemetry is not a control protocol |

## Security and delegation evidence

| Source | Signal | Use in this playbook |
|---|---|---|
| [Prompt Infection](https://arxiv.org/abs/2410.07283) | Malicious instructions can propagate between agents through messages and shared context | Delegated results remain untrusted; never promote peer text into the authority lane |
| [Google: Securing multi-agent systems](https://research.google/pubs/securing-multi-agent-systems-an-empirical-analysis-of-security-prompt-hardening-and-residual-risks/) | Prompt hardening reduces some attacks but leaves residual risk | Use runtime isolation and capability enforcement, not prompts alone |
| [GUARDIAN](https://proceedings.neurips.cc/paper_files/paper/2025/file/0bc795afae289ed465a65a3b4b1f4eb7-Paper-Conference.pdf) | Studies protection for multi-agent collaboration under adversarial behavior | Evaluate malicious-peer and compromised-worker scenarios explicitly |
| [Bounded Agents](https://arxiv.org/abs/2608.15888) | Proposes scoped delegation chains with budgets and authority tracking | Useful emerging direction for authority lineage; do not treat preprint results as an established standard |

## Stable conclusions

### 1. Keep known control flow outside the model

If dependencies, joins, retries, approvals, and terminal conditions are known, express them in code or a workflow graph. Use model decisions inside bounded nodes. Model-generated control is justified when the next useful step genuinely depends on semantic interpretation that cannot be enumerated economically.

### 2. A plan is a versioned hypothesis

A plan records current assumptions and dependencies. Every consequential step must re-check authoritative state. Replan when an assumption is disproved, resource version changes, a result invalidates downstream work, an error consumes the retry policy, or remaining budget cannot complete the plan.

### 3. Multi-agent systems scale work, not truth

Additional agents buy parallel context windows, tool concurrency, specialization, and independent exploration. They also add coordination tokens, latency, state divergence, security edges, and error propagation. The key predictors are decomposability, dependency density, synchronization cost, and the value of diversity—not the prestige of a “team” abstraction.

### 4. Centralized management is the operational default

A manager with scoped workers gives one place to own the user contract, budget, approval, termination, effect ledger, and synthesis. Use a handoff when the specialist truly must own the next user interaction or long-lived domain conversation. Use peer mesh or open group chat only when the workload demonstrates a benefit that outweighs weak ownership and difficult termination.

### 5. Delegation is a capability contract

Every delegated unit needs objective, allowed data, allowed tools/effects, budget, deadline, cancellation token, result schema, evidence requirements, and parent/trace lineage. A child may receive equal or narrower authority—never implicit broader authority. Results are proposals or evidence until the owning runtime validates and commits them.

### 6. Isolate context and make state ownership explicit

Project only the context a worker needs. Return a compact typed result plus artifact references and evidence, not an unbounded transcript. Shared mutable state requires an owner, version, conflict policy, and atomic update semantics; otherwise parallelism creates silent lost updates and contradictory plans.

### 7. Propagate stopping signals end to end

Cancellation, deadline, budget exhaustion, and supersession must reach queued work, active tools, remote tasks, stream consumers, and commit gates. Since cancellation is often best-effort, the commit path must fence canceled or stale attempts.

### 8. Select protocols by boundary

- MCP: agent application to tool/data server.
- A2A: independent agent service to independent agent service.
- AG-UI: agent backend to interactive user interface.
- OpenTelemetry: runtime to telemetry pipeline.

They can coexist in one system. None substitutes for orchestration, authorization policy, trust admission, durable workflow state, or effect idempotency.

### 9. Discovery is not trust

A tool description, AgentCard, remote schema, frontend tool, or capability declaration says what a peer claims it can do. Admission, identity, authorization, provenance, supply-chain checks, sandboxing, and output validation are separate decisions.

### 10. Protocol messages are untrusted inputs

Validate envelope schema, size, content type, tenant, correlation, sequence, provenance, and authorization. Treat embedded instructions, URLs, files, artifacts, UI state, and metadata as potentially malicious. Use exact-effect approval and policy enforcement at the commit point.

## Important disagreements and conditional choices

| Question | Evidence for one side | Evidence for the other | Playbook position |
|---|---|---|---|
| More agents improve quality | Parallel research and independent exploration can improve coverage | Sequential/dependent workloads and stronger base models can lose quality through coordination overhead | Add agents only after a workload-shaped eval demonstrates marginal value |
| Static plan vs continual replanning | Up-front dependency plans enable parallelism and fewer model calls | Environmental uncertainty makes stale plans dangerous | Use a stable task graph for invariants and event-triggered replanning for uncertain nodes |
| Manager vs handoff | Manager centralizes control and synthesis | Handoff enables a domain specialist to own interaction and reduce manager bottlenecks | Manager by default; handoff for explicit ownership transfer with a return/escalation contract |
| Full shared history vs isolated workers | Shared history reduces projection work | It expands injection, privacy, distraction, and token costs and weakens ownership | Isolate by default; share typed state and immutable artifact references |
| Group debate improves reasoning | Diversity and critique can surface alternatives | Correlated models repeat errors and conversation can converge on confident mistakes | Prefer independent candidates plus deterministic/verifier-based selection where possible |
| Protocol adoption supplies interoperability | Common envelopes and lifecycle semantics reduce bespoke adapters | Products vary in version, auth, durability, and optional extensions | Use adapters, conformance tests, and pinned capability profiles; avoid lowest-common-denominator architecture |

## Claims deliberately excluded or constrained

- No vendor-reported quality, cost, speed, or error-amplification number is presented as a universal expectation.
- Role names or “CEO/worker” prompting are not treated as a substitute for topology, state, or capability design.
- An AgentCard, MCP server configuration, schema, tool description, or AG-UI client is not described as trusted merely because it is protocol-conformant.
- A2A `messageId` support is not assumed to make every remote effect exactly-once; cancellation is not assumed to succeed.
- AG-UI state events are not treated as authoritative persistence.
- MCP is not described as an agent-to-agent protocol or an orchestration engine.
- Hidden chain-of-thought is not required as an inter-agent interface; share concise decisions, evidence, assumptions, and structured rationale instead.
- Tree search, agent societies, autonomous negotiation, and prompt-only delegation security remain experimental unless validated for the target workload.

## Refresh triggers

Re-run this research when any of the following occurs:

- MCP publishes a revision after **2026-07-28** or ends a listed compatibility window.
- A2A publishes a release after **1.0.0** or changes task/cancellation/authentication semantics.
- AG-UI promotes draft event/state capabilities or publishes normative compatibility and delivery guarantees.
- OpenTelemetry GenAI agent conventions stabilize or rename core attributes/events.
- OpenAI Agents SDK, Google ADK, Microsoft Agent Framework/AutoGen, or LangChain materially changes handoff/context/graph semantics.
- Local topology evaluations shift after a base-model, tool, workload, latency, or pricing change.
- New evidence changes the security model for delegated authority, peer compromise, or agent-to-agent injection.

## Guides produced from this packet

- [Planning and replanning](../../orchestration/planning-and-replanning.md)
- [Multi-agent topologies](../../orchestration/multi-agent-topologies.md)
- [Delegation, handoffs, and shared state](../../orchestration/delegation-handoffs-and-shared-state.md)
- [Protocol selection](../../protocols/protocol-selection.md)
- [Model Context Protocol](../../protocols/model-context-protocol.md)
- [Agent2Agent protocol](../../protocols/agent-to-agent-protocol.md)
- [Agent-user interaction with AG-UI](../../protocols/agent-user-interaction-protocol.md)

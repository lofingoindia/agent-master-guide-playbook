# Decision-Guide Index

**Last updated:** 2026-08-31

| Decision | Guide | Status |
|---|---|---|
| Ordinary code vs workflow vs agent | [Agentic systems](../foundations/agentic-systems.md) | Research-backed draft |
| Custom loop vs agent framework vs durable workflow engine | [Decision guide](../comparisons/custom-loop-vs-framework-vs-workflow-engine.md) | Research-backed draft |
| OpenAI Agents SDK vs Claude Agent SDK/Managed Agents vs Google ADK vs Microsoft Agent Framework | [Provider-native framework selection](../comparisons/provider-native-agent-frameworks.md) | Research-backed decision guide |
| LangChain/LangGraph/Deep Agents vs Pydantic AI vs Vercel AI SDK vs Strands Agents | [Independent framework selection](../comparisons/independent-agent-frameworks.md) | Research-backed decision guide |
| AutoGen/Semantic Kernel migration vs CrewAI vs LlamaIndex/LlamaAgents vs Mastra vs DeepSeek Harness | [Evolving ecosystem selection](../comparisons/evolving-agent-framework-ecosystems.md) | Research-backed lifecycle-aware decision guide |
| In-process state vs checkpointing vs durable execution | [Durable execution](../runtime/durable-execution.md) | Research-backed draft |
| Temporal vs Restate vs DBOS vs Prefect vs Dapr Workflow | [Durable runtime selection](../comparisons/durable-agent-workflow-runtimes.md) | Research-backed decision guide |
| Retry vs model-visible failure vs abort | [Run controls](../runtime/run-controls.md) | Research-backed draft |
| Sequential vs parallel tool execution | [Planning and replanning](../orchestration/planning-and-replanning.md), [Run controls](../runtime/run-controls.md) | Research-backed draft |
| Approval vs containment | [Permissions, sandboxing, and secrets](../security/permissions-sandboxing-and-secrets.md), [Execution boundaries](../runtime/execution-boundaries.md) | Research-backed draft |
| Tool granularity and result shape | [Tool contracts](../tools/tool-contracts.md) | Research-backed draft |
| Single agent vs multi-agent | [Multi-agent topologies](../orchestration/multi-agent-topologies.md) | Research-backed draft |
| Fixed workflow vs model-generated plan | [Planning and replanning](../orchestration/planning-and-replanning.md) | Research-backed draft |
| Router vs supervisor vs parallel workers | [Multi-agent topologies](../orchestration/multi-agent-topologies.md) | Research-backed draft |
| RAG vs agent memory vs workflow state | [Memory architecture](../context-memory/memory-architecture.md) | Research-backed draft |
| Append vs retrieve vs compact vs reset | [Context engineering](../context-memory/context-engineering.md), [Compaction and continuity](../context-memory/compaction-and-continuity.md) | Research-backed draft |
| Exact trajectory vs invariants | [Trajectory and reliability evaluation](../evaluation/trajectory-and-reliability-evaluation.md) | Research-backed draft |
| Deterministic vs model vs human grader | [Evaluation-driven development](../evaluation/evaluation-driven-development.md) | Research-backed draft |
| Container vs OS sandbox vs VM | [Permissions, sandboxing, and secrets](../security/permissions-sandboxing-and-secrets.md) | Research-backed draft |
| SQL vs event log vs document state | [Runtime queue](../runtime/README.md) | Discovery |
| Queue vs workflow engine | [Durable execution](../runtime/durable-execution.md), [Queues, scheduling, and backpressure](../operations/queues-scheduling-and-backpressure.md) | Research-backed draft |
| FIFO vs priority vs fair scheduling | [Queues, scheduling, and backpressure](../operations/queues-scheduling-and-backpressure.md) | Research-backed draft |
| Queue vs reject vs degrade | [Queues, scheduling, and backpressure](../operations/queues-scheduling-and-backpressure.md) | Research-backed draft |
| Retry vs reconcile vs stop | [Queues, scheduling, and backpressure](../operations/queues-scheduling-and-backpressure.md), [Idempotency](../reliability/idempotency-and-side-effects.md) | Research-backed draft |
| Fixed model vs cascade vs learned router vs ensemble | [Model routing, cost, and latency](../operations/model-routing-cost-and-latency.md) | Research-backed draft |
| Online vs batch vs flexible vs provisioned capacity | [Model routing, cost, and latency](../operations/model-routing-cost-and-latency.md) | Research-backed draft; recheck provider terms |
| Queue age vs depth vs resource saturation for scaling | [Scaling, capacity, and SLOs](../operations/scaling-capacity-and-slos.md) | Research-backed draft |
| Shared pool vs cell vs dedicated tenant infrastructure | [Scaling, capacity, and SLOs](../operations/scaling-capacity-and-slos.md), [Production agent control plane](../architectures/production-agent-control-plane.md) | Research-backed draft |
| Canary vs blue/green vs immediate rollback | [Deployment, release, and incident response](../operations/deployment-release-and-incident-response.md) | Research-backed draft |
| Pin vs migrate in-flight long-running runs | [Deployment, release, and incident response](../operations/deployment-release-and-incident-response.md) | Research-backed draft |
| Interactive vs durable background execution | [Interactive and long-running reference architectures](../architectures/interactive-and-long-running-reference-architectures.md) | Research-backed reference architecture |
| Python vs TypeScript vs Go vs JVM vs .NET vs Rust | [Choosing an agent runtime language](../languages/choosing-an-agent-runtime-language.md) | Research-backed decision guide |
| Single-language vs polyglot agent platform | [Choosing an agent runtime language](../languages/choosing-an-agent-runtime-language.md) | Research-backed decision guide |
| Local vs remote tools | [Tool registries, versioning, and lifecycle](../tools/tool-registries-versioning-and-lifecycle.md), [Execution boundaries](../runtime/execution-boundaries.md) | Partial / research-backed lifecycle and boundary |
| Fixed tool set vs workflow-stage exposure vs retrieval | [Tool discovery and selection](../tools/tool-discovery-and-selection.md) | Research-backed draft |
| Lexical vs semantic vs hybrid tool retrieval | [Tool discovery and selection](../tools/tool-discovery-and-selection.md) | Research-backed draft |
| Direct vs programmatic tool calling | [Tool results, artifacts, and provenance](../tools/tool-results-artifacts-and-provenance.md) | Research-backed draft; provider mechanics volatile |
| Raw result vs projection vs artifact | [Tool results, artifacts, and provenance](../tools/tool-results-artifacts-and-provenance.md) | Research-backed draft |
| Compatible rollout vs new tool version | [Tool registries, versioning, and lifecycle](../tools/tool-registries-versioning-and-lifecycle.md) | Research-backed draft |
| Manager-as-tools vs handoffs | [Delegation, handoffs, and shared state](../orchestration/delegation-handoffs-and-shared-state.md) | Research-backed draft |
| MCP vs A2A vs AG-UI | [Selecting agent protocols](../protocols/protocol-selection.md) | Research-backed draft |
| Polling vs streaming vs push for remote agents | [Agent2Agent protocol](../protocols/agent-to-agent-protocol.md) | Research-backed draft |
| UI snapshot vs delta vs event replay | [Agent-user interaction with AG-UI](../protocols/agent-user-interaction-protocol.md) | Research-backed draft |

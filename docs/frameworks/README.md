# SDKs, Frameworks, Harnesses, and Orchestration Systems

**Ecosystem discovery date:** 2026-08-31  
**Status:** Research-backed overview clusters are available; multi-guide technology knowledge areas are now under active expansion.

The existing guides remain stable entry points while substantial SDKs, frameworks, harnesses, and runtimes are expanded into dedicated folders. The [deep knowledge-area expansion program](../research/deep-expansion-program.md) defines file boundaries, evidence gates, canonicality rules, and the prioritized migration waves.

## Deep knowledge areas

These folders extend the concise ecosystem overviews into production playbooks:

- [OpenAI Agents SDK production deep dive](openai-agents-sdk/README.md) — 12 focused guides for the loop, provider boundaries, tools, state, handoffs, streaming, evals, security, reliability, deployment, sandboxes, parity, and migration.
- [LangGraph production playbook](langgraph/README.md) — 14 focused guides for runtime semantics, state/reducers, persistence, interrupts, streaming, subgraphs, effects, Agent Server, security, testing, and migration.
- [Vercel AI SDK engineering guide](vercel-ai-sdk/README.md) — 12 focused guides for Core and agent loops, providers, tools, UI streams, persistence, Workflow durability, telemetry, testing, reliability, deployment, security, and migration.
- [Strands Agents production engineering guide](strands-agents/README.md) — 12 focused guides for the loop, providers, tools/MCP, sessions, streams, hooks, multi-agent patterns, HITL/security, evaluation, reliability, deployment, parity, and alternatives.
- [Pydantic AI production playbook](pydantic-ai/README.md) — 14 focused guides for lifecycle, providers, tools/MCP, validation, streaming, limits, state, durable adapters, delegation, evals, telemetry, security, deployment, and migration.
- [Claude Agent SDK and Anthropic agent ecosystem](claude-agent-sdk/README.md) — 13 focused guides for the child-process harness, workspace, loop, extensions, state/compaction, streams, permissions, delegation, hosting, evaluation, reliability, Managed Agents, and migration.
- [Google ADK production playbook](google-adk/README.md) — 12 focused guides for Runner/events, agents/models, tools/MCP/A2A, sessions/artifacts/memory, graph workflows, live streams, HITL, deployment, evaluation, reliability, security, and language parity.
- [AutoGen retained-production engineering guide](autogen/README.md) — 10 focused guides for lifecycle, messages/runtime, tools/code execution, teams, state/resume, streaming/HITL, distributed deployment, observability, security/reliability, and migration.
- [CrewAI production engineering playbook](crewai/README.md) — 12 focused guides for Crews/Flows, events/state, tools/MCP/knowledge, memory, persistence/HITL, delegation/A2A, observability, reliability, security, deployment, and migration.
- [Mastra production engineering guide](mastra/README.md) — 14 focused guides for agents, workflows, tools/MCP, memory/storage, snapshots/resume, streams/PubSub, supervisors, observability, reliability, security, deployment, durable engines, and migration.
- [LlamaIndex and LlamaAgents engineering guide](llamaindex-llamaagents/README.md) — 12 extensive guides for event workflows, agents/tools, data/RAG, context/memory, state/events, multi-agent patterns, streaming/HITL, server/executors, DBOS recovery, evaluation, security, deployment, and migration.
- [DeepSeek Harness production-minded preview guide](deepseek-harness/README.md) — 10 focused guides for Cordis architecture, event-sourced sessions, plugins/tools/skills, compaction, subagents/workflows, sandbox boundaries, providers, evaluation, operations, and evidence-based adoption gates.
- [Microsoft Agent Framework production playbook](microsoft-agent-framework/README.md) — 13 focused guides for ecosystem boundaries, agents/providers, tools/MCP, middleware/sessions, graph workflows, checkpoints/HITL, streams/protocols, multi-agent patterns, hosting, evaluation, reliability, security, maturity, and migration across Python, .NET, and Go.
- [Semantic Kernel retained-ecosystem guide](semantic-kernel/README.md) — 12 focused guides for lifecycle, kernels/connectors, agents/threads, plugins/tools, filters/policy, experimental processes/orchestration, vector data/RAG, streams, evaluation, security, reliability, language parity, and staged MAF migration.

## Research-backed guides

- [Selecting a provider-native agent framework](../comparisons/provider-native-agent-frameworks.md)
- [OpenAI Agents SDK in production](openai-agents-sdk.md)
- [Claude Agent SDK and Managed Agents in production](claude-agent-sdk-and-managed-agents.md)
- [Google Agent Development Kit in production](google-adk.md)
- [Microsoft Agent Framework in production](microsoft-agent-framework.md)
- [Selecting an independent agent framework](../comparisons/independent-agent-frameworks.md)
- [LangChain, LangGraph, and Deep Agents in production](langchain-langgraph-and-deep-agents.md)
- [Pydantic AI in production](pydantic-ai.md)
- [Vercel AI SDK in production](vercel-ai-sdk.md)
- [Strands Agents in production](strands-agents.md)
- [Selecting across evolving agent framework ecosystems](../comparisons/evolving-agent-framework-ecosystems.md)
- [Migrating AutoGen and Semantic Kernel agent systems](autogen-and-semantic-kernel-migration.md)
- [CrewAI in production](crewai.md)
- [LlamaIndex and LlamaAgents in production](llamaindex-and-llamaagents.md)
- [Mastra in production](mastra.md)
- [DeepSeek Harness architecture and safety boundary](deepseek-harness.md)
- [Selecting a durable runtime for agent workflows](../comparisons/durable-agent-workflow-runtimes.md)
- [Temporal for agent workflows](temporal-for-agent-workflows.md)
- [Restate for agent workflows](restate-for-agent-workflows.md)
- [DBOS for agent workflows](dbos-for-agent-workflows.md)
- [Prefect for agent workflows](prefect-for-agent-workflows.md)
- [Dapr Workflow for agent workflows](dapr-workflow-for-agent-workflows.md)

## Keep the categories straight

| Category | Primary responsibility | Examples under research |
|---|---|---|
| Provider/API SDK | Call models and provider tools | OpenAI SDK, Anthropic SDK, Google Gen AI SDK |
| Agent SDK/framework | Package loops, tools, state, handoffs, and integrations | OpenAI Agents SDK, Google ADK, Pydantic AI, Vercel AI SDK, Strands |
| Stateful orchestration runtime | Explicit graphs/state/checkpoints | LangGraph, Microsoft Agent Framework workflows |
| Agent harness | Opinionated environment around a loop: planning, filesystem, skills, subagents, compaction | Claude Agent SDK, Deep Agents, DeepSeek Harness |
| Multi-agent framework/pattern layer | Agent collaboration and topology | AutoGen, CrewAI, Semantic Kernel/Agent Framework patterns |
| Data/agent workflow framework | Retrieval/data and event-driven agent workflows | LlamaIndex |
| Durable workflow engine | Failure recovery and long-lived execution | Temporal, Restate, DBOS, Prefect, Dapr Workflow |
| Protocol | Interoperability boundary | MCP, A2A, AG-UI |

One product can span categories, and categories evolve. See the [technology index](../indexes/technology-index.md) for current primary sources and status. The provider-native comparison treats Claude Managed Agents as a managed runtime choice, not as a drop-in synonym for Claude Agent SDK.

## Overview-guide contract

Every substantial overview guide summarizes, where applicable:

- architecture and abstraction level;
- actual loop/runtime semantics;
- tools and result handling;
- state, memory, context, compaction, and persistence;
- streaming and events;
- handoffs, subagents, and multi-agent topology;
- durability and long-running work;
- middleware, hooks, skills, MCP, and extensions;
- provider support and model-specific behavior;
- tracing, evaluation, testing, and debugging;
- security and authorization boundaries;
- deployment, scaling, performance, cost, and maintenance;
- version maturity, deprecations, limitations, and known traps;
- when to choose it and when not to.

An overview is not considered complete ecosystem coverage. A mature knowledge area also needs independently useful child guides for the technology's actual execution, state, failure, security, deployment, and upgrade seams. Child guides link to canonical cross-cutting guidance instead of copying it.

The Microsoft ecosystem requires particular version care: AutoGen is now officially in maintenance mode, and current Microsoft documentation provides AutoGen and Semantic Kernel agent migration paths to Microsoft Agent Framework. Historical AutoGen or Semantic Kernel advice must not be presented as the current default.

The LlamaIndex deployment ecosystem also changed: `llama_deploy` is deprecated in favor of the current LlamaAgents/Workflows stack. DeepSeek Harness remains developer-preview software whose official safety statement rejects production/security-boundary claims.

Durable runtimes do not share one replay contract. Temporal and Dapr replay deterministic event history, Restate journals SDK operations, DBOS substitutes Postgres-checkpointed step outputs, and Prefect orchestrates task/flow states plus configured results. Use the durable-runtime comparison before transferring a guarantee between them.

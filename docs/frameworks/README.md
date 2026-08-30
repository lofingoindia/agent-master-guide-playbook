# SDKs, Frameworks, Harnesses, and Orchestration Systems

**Ecosystem discovery date:** 2026-08-31  
**Status:** Provider-native, independent-framework, lifecycle, and durable-runtime clusters available; language-specific ecosystems remain under research.

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

## Deep-dive contract

Every substantial technology guide will cover, where applicable:

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

The Microsoft ecosystem requires particular version care: AutoGen is now officially in maintenance mode, and current Microsoft documentation provides AutoGen and Semantic Kernel agent migration paths to Microsoft Agent Framework. Historical AutoGen or Semantic Kernel advice must not be presented as the current default.

The LlamaIndex deployment ecosystem also changed: `llama_deploy` is deprecated in favor of the current LlamaAgents/Workflows stack. DeepSeek Harness remains developer-preview software whose official safety statement rejects production/security-boundary claims.

Durable runtimes do not share one replay contract. Temporal and Dapr replay deterministic event history, Restate journals SDK operations, DBOS substitutes Postgres-checkpointed step outputs, and Prefect orchestrates task/flow states plus configured results. Use the durable-runtime comparison before transferring a guarantee between them.

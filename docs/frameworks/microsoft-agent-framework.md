# Microsoft Agent Framework in Production

**Research date:** 2026-08-30  
**Status:** Research-backed technology guide  
**Scope:** Current Agent Framework agents, middleware, orchestrations, workflows, checkpoints, and hosting for C#, Python, and Go

## Bottom line

Choose Microsoft Agent Framework when you need a provider-neutral agent abstraction plus explicit graph workflows, middleware, sessions, OpenTelemetry, and Microsoft hosting/integration paths—especially in .NET, Python, or Go estates.

Treat package maturity per component. The core has GA releases, while integrations and newer harness, skills, security, and protocol surfaces can be stable, release candidate, preview, or experimental. “Agent Framework 1.x” is not one uniform stability guarantee.

## Position in the Microsoft ecosystem

Microsoft describes Agent Framework as the successor direction combining AutoGen’s agent patterns with Semantic Kernel’s enterprise abstractions. That does not make migration automatic.

```mermaid
flowchart LR
    AG["AutoGen patterns"] --> MAF["Microsoft Agent Framework"]
    SK["Semantic Kernel abstractions"] --> MAF
    MAF --> A["Agents + sessions"]
    MAF --> O["Orchestrations"]
    MAF --> W["Graph workflows"]
    MAF --> H["Self-host / Foundry / durable hosts"]
```

Inventory provider clients, thread/session formats, middleware/filters, plugins, orchestration semantics, and persisted state before migrating. Keep an old-runtime replay path for in-flight work until compatibility is proven.

## Agent pipeline versus workflow runtime

```mermaid
flowchart TB
    IN["Input"] --> AM["Agent-run middleware"]
    AM --> CP["Context providers / session"]
    CP --> CM["Chat/model middleware"]
    CM --> MODEL["Provider client"]
    MODEL --> FC["Function-call middleware"]
    FC --> TOOL["Tool"]
    TOOL --> OUT["Agent response"]

    WB["WorkflowBuilder"] --> EX["Executors + edges"]
    EX --> SS["Supersteps"]
    SS --> CK["Checkpoint manager"]
    SS --> EV["Workflow events / requests"]
```

Use an agent for an open-ended model/tool loop. Use a workflow when explicit routing, fan-out/fan-in, shared state, requests, or resumption is part of the business control flow. An agent can be a node; do not hide the entire business process in one opaque node.

## Checkpoint semantics

A workflow executes messages in supersteps. At the end of a completed superstep, a checkpoint can capture:

- executor state;
- messages pending for the next superstep;
- pending human requests and responses;
- shared workflow state.

Restoring a checkpoint re-emits pending requests so the application can resume the interaction. Recent Python releases added entry checkpoints for original input and delivered human responses, improving full replay while changing checkpoint/event ordering.

```mermaid
stateDiagram-v2
    [*] --> Superstep
    Superstep --> Checkpoint: all active executors finish
    Checkpoint --> NextStep: messages pending
    Checkpoint --> Waiting: request pending
    Waiting --> Checkpoint: response recorded
    NextStep --> Superstep
    Checkpoint --> Restored: crash / migration / operator action
    Restored --> NextStep
```

The checkpoint boundary is not a database transaction around tool effects. If a tool committed before the superstep checkpoint and the process failed, restore may encounter an ambiguous effect. Use operation IDs, an effect ledger, and reconciliation.

Persist these alongside a checkpoint:

- framework and package-set versions;
- workflow graph signature and executor schema versions;
- model/provider and tool-definition fingerprints;
- policy/configuration version;
- external effect receipts;
- encryption/key and permitted deserialization types.

Checkpoint deserialization is a security boundary. Use restricted formats and explicit type allowlists; never treat an untrusted checkpoint as inert data.

## Middleware without transcript corruption

Agent Framework provides agent-run, streaming, function, and model/chat middleware. It is a strong fit for telemetry, identity propagation, policy, caching, and result transformation, but ordering and short-circuit behavior are part of correctness.

| Middleware action | Required invariant |
|---|---|
| Reject before model | No model/tool work has started; rejection is traceable |
| Reject function call | Transcript receives a syntactically valid function result/rejection |
| Replace result | Original result retained in protected audit evidence if needed |
| Retry provider call | Same deadline and attempt budget; uncertain delivery classified |
| Terminate stream | Child/provider/tool work is cancelled and final state is coherent |

Official documentation warns that terminating the function loop can leave a function call without a result in chat history. Build middleware contract tests that parse the final transcript and checkpoint after every short circuit.

## Provider neutrality is conditional

The standard Agent can use multiple model clients; direct agent types can connect to managed or remote runtimes such as Foundry, A2A, GitHub Copilot, or Claude. This supports architectural substitution, but each client controls:

- tool-call shape and parallelism;
- hosted tools and background-response support;
- structured-output guarantees;
- conversation/server-state behavior;
- usage and model identity reporting;
- retry, streaming, cancellation, and approval semantics.

Keep core domain inputs, outputs, tools, state, and evaluation fixtures independent of provider-specific message types. A second-provider smoke test is more valuable than an abstraction-interface claim.

## Human input and approvals

Custom workflows use request ports; orchestrations can surface function approval requests. Checkpoints retain pending requests and re-emit them after restore. This is a sound control-plane shape when the application also:

1. stores exact proposed action, target, arguments, requester, and policy version;
2. expires or revokes stale decisions;
3. reauthorizes immediately before the effect;
4. executes with a stable operation ID;
5. records the effect outcome outside the model transcript.

Test nested agents and handoff orchestrations separately. Current issue history shows that approval/middleware visibility and checkpoint restore have not always been uniform at nested boundaries.

## Package and release discipline

The Python 1.13.0 changelog contains stable orchestrations alongside breaking checkpoint/event changes and breaking experimental harness changes. This is healthy transparency, but it means semantic versioning must be applied to the exact package set and maturity labels.

Use a compatibility manifest:

| Field | Example meaning |
|---|---|
| Core/runtime | Exact C#, Python, or Go package release |
| Provider adapter | OpenAI, Anthropic, Gemini, Foundry, or remote-agent package |
| Workflow/orchestration | Exact version and stability tier |
| UI/protocol adapter | AG-UI/A2A/ChatKit version and missing core capabilities |
| Checkpoint format | Encoder version and allowed types |
| Hosting extension | Foundry, Durable Task, Azure Functions, or self-host revision |

Do not independently float these dependencies in production.

## Hosting and durability

- **Self-hosting** gives full control over network, identity, process lifetime, stores, and scaling.
- **Foundry hosting** integrates provider resources and Microsoft operations but adds service-specific contracts.
- **Durable Task/Azure Functions integrations** can own failure recovery and long waits; keep agent calls as bounded activities and effect boundaries explicit.
- **Core checkpointing** supports workflow resume but does not by itself provide a distributed queue, leases, exactly-once effects, or operator repair service.

## Operational acceptance tests

- [ ] Pin and record every core, provider, orchestration, protocol, persistence, and hosting package.
- [ ] Restore checkpoints across the exact planned upgrade, including old in-flight workflows.
- [ ] Crash before, during, and after a superstep tool effect; reconcile instead of blind replay.
- [ ] Exercise concurrent branches, shared state, handoff, group chat, and request/response ordering.
- [ ] Approve a nested tool, restart, restore, and confirm original arguments and policy scope.
- [ ] Short-circuit every middleware layer and validate transcript/checkpoint invariants.
- [ ] Verify cancellation reaches model clients, tools, executors, and background work.
- [ ] Test the exact provider endpoint: chat vs Responses vs managed agent can differ.
- [ ] Confirm telemetry correlation and sensitive-content settings across all adapters.
- [ ] Threat-test checkpoint deserialization, MCP sampling, tool results, and cross-tenant IDs.

## Choose it when

- The organization is invested in .NET/Python/Go and Microsoft operational tooling.
- Provider substitution and remote-agent adapters are valuable, with understood parity limits.
- Explicit multi-agent workflow graphs and checkpointed human requests fit the use case.
- Middleware and context-provider composition match existing platform architecture.

## Prefer another shape when

- A small provider-native loop is sufficient.
- The team cannot absorb package-matrix and checkpoint migration testing.
- The required adapter lacks a core capability such as checkpointing or nested approval.
- A mature general-purpose workflow engine already owns the process and only needs bounded model activities.
- AutoGen/Semantic Kernel migration cost exceeds demonstrated value.

## Sources and related guides

Primary sources: [overview](https://learn.microsoft.com/en-us/agent-framework/overview/), [agent concepts](https://learn.microsoft.com/en-us/agent-framework/concepts/agents/), [workflows](https://learn.microsoft.com/en-us/agent-framework/concepts/workflows/), [checkpoints](https://learn.microsoft.com/en-us/agent-framework/workflows/checkpoints), [human in the loop](https://learn.microsoft.com/en-us/agent-framework/workflows/human-in-the-loop), [middleware](https://learn.microsoft.com/en-us/agent-framework/agents/middleware/), [self-hosting](https://learn.microsoft.com/en-us/agent-framework/hosting/self-hosting/), [Python changelog](https://github.com/microsoft/agent-framework/blob/main/python/CHANGELOG.md), and [durable workflows](https://devblogs.microsoft.com/dotnet/durable-workflows-in-microsoft-agent-framework/).

- [Provider-native framework selection](../comparisons/provider-native-agent-frameworks.md)
- [Custom loop vs framework vs workflow engine](../comparisons/custom-loop-vs-framework-vs-workflow-engine.md)
- [Durable execution](../runtime/durable-execution.md)
- [Multi-agent topologies](../orchestration/multi-agent-topologies.md)
- [Research packet](../research/packets/provider-native-agent-frameworks.md)

# Package Maturity, Migration, Limitations, and Alternatives

## Read maturity at three levels

A stable meta-package does not make every integration stable. Evaluate:

1. **language implementation** — upstream Python/.NET or separate Go preview;
2. **package** — stable, RC, beta, alpha, preview, or untagged module;
3. **feature** — an API inside a stable package can still carry an experimental decorator/warning.

This snapshot was verified on **2026-08-31** from PyPI JSON, NuGet V3, the Go module proxy/repository, Python `PACKAGE_STATUS.md`, and current Microsoft Learn pages. Recheck before adoption.

## Language snapshot

| Language | Verified baseline | Maturity | Important gaps/notes |
|---|---|---|---|
| Python | `agent-framework 1.16.0`, `>=3.10`, uploaded 2026-08-28 | Core/meta stable | Many independently versioned integrations remain alpha/beta; experimental APIs exist in core |
| .NET | `Microsoft.Agents.AI 1.19.0`; `Workflows 1.19.0`, released 2026-08-22 | Core/workflows stable | Foundry latest is preview despite older stable; hosting/Anthropic/DevUI preview; declarative RC |
| Go | pseudo-version `v0.0.0-20260829074433-6c58ac4d9c8d`; `go 1.26.0` | Public preview, no normal stable semver tag verified | Separate repo; no declarative agents/RAG/CodeAct/functional workflows/DevUI/Foundry hosting/Durable Extension; handoff not implemented |

The MAF FAQ still says “currently supports .NET and Python,” while the overview, get-started material, and Go repository document Go public preview. Interpret this as documentation-scope lag: Go exists, but it is not at upstream parity or stable maturity.

## Python package snapshot

Representative packages, not the entire ecosystem:

| Package | Verified version | Repository lifecycle state | Interpretation |
|---|---:|---|---|
| `agent-framework` | 1.16.0 | released | Meta-package; inspect included dependency set |
| `agent-framework-core` | 1.16.0 | released | Stable core can still export experimental features |
| `agent-framework-openai` | 1.14.1 | released | Independently versioned provider |
| `agent-framework-foundry` | 1.11.0 | released | Some tool/eval APIs still staged |
| `agent-framework-orchestrations` | 1.1.1 | released | Built-in orchestration package |
| `agent-framework-declarative` | 1.0.3 | released | Declarative-agent loader feature remains experimental |
| `agent-framework-hosting` | 1.0.0a260730 | alpha | Not a stable production API contract |
| `agent-framework-foundry-hosting` | 1.0.0b260827 | beta | Foundry service GA does not change package stage |
| `agent-framework-a2a` | 1.0.0b260821 | beta | Protocol/client integration prerelease |
| `agent-framework-devui` | 1.0.0b260821 | beta | Development tool only |

Other beta packages in `PACKAGE_STATUS.md` include Anthropic, Bedrock, Gemini, Copilot Studio, memory/search/database integrations, Purview, Redis, tools, and others. Hosting protocol packages are alpha. `agent-framework-azure-ai` is deprecated after capabilities moved/renamed into `agent-framework-foundry`.

### Python experimental features inside packages

The checked feature-stage registry marks these experimental: Agent Hooks, declarative agents, evaluations, file history, FIDES, selected Foundry tools, functional workflows, parts of the harness, MCP long-running tasks, MCP skills, progressive tools, session store, and `to_prompt_agent`.

Do not hide these APIs behind an internal “stable core” assumption. Wrap them behind a small adapter, pin versions, keep migration tests, and define a fallback/removal path.

## .NET package snapshot

| Package | Latest checked | Latest stable checked | Interpretation |
|---|---:|---:|---|
| `Microsoft.Agents.AI` | 1.19.0 | 1.19.0 | Stable core |
| `Microsoft.Agents.AI.Abstractions` | 1.19.0 | 1.19.0 | Stable abstractions |
| `Microsoft.Agents.AI.Workflows` | 1.19.0 | 1.19.0 | Stable graph workflow package |
| `Microsoft.Agents.AI.OpenAI` | 1.19.0 | 1.19.0 | Stable integration |
| `Microsoft.Agents.AI.Foundry` | 1.19.0-preview.260822.1 | 1.5.0 | Current feature line is preview; a much older stable exists |
| `Microsoft.Agents.AI.Hosting` | 1.19.0-preview.260822.1 | none verified | Preview |
| `Microsoft.Agents.AI.Foundry.Hosting` | 1.19.0-preview.260822.1 | none verified | Preview |
| `Microsoft.Agents.AI.Anthropic` | 1.19.0-preview.260822.1 | none verified | Preview |
| `Microsoft.Agents.AI.Declarative` | 1.19.0-rc1 | none verified | Release candidate |
| `Microsoft.Agents.AI.DevUI` | 1.19.0-preview.260822.1 | none verified | Preview/development |

NuGet packages share a release train more often than Python, but lifecycle still varies by suffix and package. Pin exact versions, not a floating `1.19.*`, and verify transitive Microsoft.Extensions.AI/Azure/OpenAI SDK compatibility.

## Go availability and parity

The Go SDK is usable for core-preview work:

- agents, sessions/history, context providers, middleware;
- OpenAI/Azure OpenAI, Foundry, Anthropic, Gemini, GitHub Copilot, A2A and AG-UI-related providers;
- local/function/MCP/shell and selected hosted-tool declarations;
- graph workflows, state, events, checkpoints, HITL, subworkflows;
- sequential, concurrent, and group-chat agent workflows;
- OpenTelemetry integrations.

Material checked gaps include:

- no stable semver release and a Go 1.26 requirement in `go.mod`;
- no declarative agents/workflows, RAG, CodeAct, or Python functional workflows;
- no handoff orchestration in the repository README;
- no MAF Foundry hosted deployment adapter or Durable Extension;
- no DevUI or packaged Foundry evaluation integration;
- fewer provider/admin/host/storage integrations and samples;
- Go-native message/checkpoint types are not binary compatible with Python/.NET.

Choose Go only after a requirement-level parity matrix and upgrade budget. Do not use a Python/.NET sample as proof the Go path exists.

## Release/change risk

Recent Python releases demonstrate why “1.x” still requires careful upgrades:

- 1.13.0 changed checkpoint entry/replay ordering and source IDs while retaining older checkpoint support;
- 1.14.0 moved Durable Task/Azure Functions integrations to a separate repository and changed beta Foundry hosting storage;
- 1.15.0 changed OpenTelemetry conventions and persisted approval behavior;
- 1.16.0 fixed external mutation of stored checkpoints and continuation/session issues.

Build an upgrade ledger per package:

```text
old pin -> new pin
API/behavior changes
checkpoint/session/protocol compatibility
provider/model dependency changes
security fixes
conformance/eval results
rollout and rollback plan
```

## Migration from AutoGen

AutoGen is in maintenance mode, community-managed, and accepts bug/security/documentation work rather than new features. MAF is its actively developed successor, but migration is architectural, not a package rename.

Key changes:

- AutoGen AgentChat teams/event-driven abstractions map to MAF agents plus typed workflows/orchestrations;
- MAF `AgentSession` makes continuation explicit; test history defaults rather than assuming AutoGen state behavior;
- tool iteration, message/content types, middleware, streaming, and HITL shapes differ;
- AutoGen Core’s local/distributed runtime is not replaced by the default in-process MAF workflow runtime;
- choose Durable Extension or another workflow platform when distributed recovery is required;
- preserve stable tool/domain APIs first, then replace agents, then orchestration, then hosting.

The official AutoGen migration page contains stale availability rows (for example providers or local execution marked planned despite current packages/docs). Use it for conceptual mappings and current provider/package sources for availability.

Detailed behavioral migration guidance already exists in the repository’s adjacent [AutoGen and Semantic Kernel migration guide](../autogen-and-semantic-kernel-migration.md); this cluster does not duplicate it.

## Migration from Semantic Kernel

MAF grew from work by the Semantic Kernel and AutoGen teams, but Semantic Kernel remains a broader SDK ecosystem. Agent migration typically changes:

- namespaces/packages to `Microsoft.Agents.AI` or `agent_framework`;
- `Kernel`/agent-specific construction to direct chat-client and tool composition;
- thread types to agent-created sessions;
- invocation responses/streams to `AgentResponse`/updates;
- plugin functions to direct tool schemas;
- agent orchestration to MAF workflows/orchestrations.

Do not rewrite working non-agent Semantic Kernel capabilities automatically. Existing Kernel functions and vector-store connectors can be adapted during an incremental migration. Preserve provider resource cleanup/deletion because `AgentSession` has no universal remote-history deletion API.

## Known limitations and documentation contradictions

| Observation | Practical resolution |
|---|---|
| FAQ lists only Python/.NET; overview and Go repo list Go preview | Record Go as public preview, separate and non-parity |
| Foundry Hosted Agents is GA; MAF hosting adapters are beta/preview | Track service and integration package maturity independently |
| Stable Python packages expose experimental features | Check `PACKAGE_STATUS.md` feature decorators per API |
| FIDES Learn docs ship an experimental feature; ADR status remains proposed | Treat implementation as experimental and verify source/tests, not ADR acceptance |
| Migration guide availability can lag current provider/tool pages | Use registry + current integration page for present availability |
| Shared concept names suggest parity | Maintain language-specific conformance matrix |

## Alternatives

Choose the smallest alternative that fits the required guarantees:

| Requirement | Consider |
|---|---|
| One model call or tool loop | Provider SDK or Microsoft.Extensions.AI without a workflow |
| Existing Semantic Kernel app using plugins/vector stores | Keep SK core; migrate only agent surfaces that benefit |
| Existing stable AutoGen system with no new feature need | Maintain temporarily with a risk-managed migration plan |
| Durable business workflow, timers, compensation, broad integrations | Durable Task/Functions, Temporal, or another workflow engine with agents as activities |
| Remote interoperability only | A2A/MCP protocol adapters without adopting every MAF layer |
| Go production requiring missing features | Another mature Go-native design or a service boundary around Python/.NET MAF |

Evaluate alternatives on state ownership, recovery, tool policy, provider support, language maturity, observability, hosting, operational skill, lock-in, and migration cost—not marketing feature counts.

## Adoption checklist

- [ ] Exact language, package, feature, provider, host, and protocol stages are recorded.
- [ ] Package pins and transitive SDK versions are locked.
- [ ] Experimental APIs are isolated behind adapters with fallback paths.
- [ ] Go requirements use only verified Go surfaces.
- [ ] Migration preserves behavior through characterization tests before rewrites.
- [ ] Distributed-runtime requirements are not assigned to in-process workflows.
- [ ] Every upgrade runs checkpoint, approval, stream, provider, security, and eval conformance.
- [ ] Contradictory docs are resolved using current registry/source evidence.

## Sources

- [Python package status](https://github.com/microsoft/agent-framework/blob/main/python/PACKAGE_STATUS.md)
- [Python changelog](https://github.com/microsoft/agent-framework/blob/main/python/CHANGELOG.md)
- [PyPI `agent-framework`](https://pypi.org/project/agent-framework/)
- [NuGet `Microsoft.Agents.AI`](https://www.nuget.org/packages/Microsoft.Agents.AI/)
- [NuGet `Microsoft.Agents.AI.Workflows`](https://www.nuget.org/packages/Microsoft.Agents.AI.Workflows/)
- [Go repository and preview notice](https://github.com/microsoft/agent-framework-go)
- [Go .NET feature comparison](https://github.com/microsoft/agent-framework-go/blob/main/docs/dotnet-go-sdk-feature-comparison.md)
- [MAF FAQ](https://learn.microsoft.com/en-us/agent-framework/support/faq)
- [MAF overview](https://learn.microsoft.com/en-us/agent-framework/overview/)
- [AutoGen repository and maintenance notice](https://github.com/microsoft/autogen)
- [AutoGen migration](https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-autogen/)
- [Semantic Kernel migration](https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-semantic-kernel/)

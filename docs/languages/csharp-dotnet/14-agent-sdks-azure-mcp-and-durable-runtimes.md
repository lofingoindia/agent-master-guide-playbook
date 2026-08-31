# Agent SDKs, Azure, MCP, and Durable Runtimes

> **Last researched:** 2026-08-31
> **Applies to:** .NET 10 LTS, C# 14

No single package owns the complete production runtime. Provider SDKs speak model APIs, agent frameworks coordinate loops, MCP standardizes capability exchange, and workflow engines provide durable execution. Compose only the layers required.

## Current ecosystem snapshot

Versions are observations at the research date, not upgrade recommendations.

| Component | Observed current line | Production interpretation |
|---|---|---|
| OpenAI <code>OpenAI</code> | 2.13.0 | Official API client; built-in retries; some surfaces can remain experimental |
| Anthropic <code>Anthropic</code> | 12.44.0 | Official C# SDK, but package documentation still labels it beta |
| <code>Microsoft.Extensions.AI</code> | 10.9.0 family | Stable abstractions with feature-level experimental annotations possible |
| Microsoft Agent Framework | .NET 1.19.0 | Core reached 1.0 GA; several hosting/durable/integration packages remain prerelease |
| MCP C# SDK | 2.2.0 | Official SDK aligned with the 2026-07-28 protocol line |
| Temporal .NET SDK | 1.18.0 | Stable general durable workflow SDK |
| Dapr Workflow | Dapr 1.17/1.18 docs line | Stable building block; requires Dapr runtime and deterministic workflows |

Verify release pages, package metadata, and feature annotations before adopting a version.

## Selection map

~~~mermaid
flowchart TD
    Need[Required capability] --> Model{Model API only?}
    Model -->|yes| SDK[Official provider SDK]
    Model -->|no| Loop{Reusable agent abstractions?}
    Loop -->|yes| MEAI[Extensions.AI / Agent Framework]
    Loop -->|no| Own[Small owned run loop]
    Need --> MCP{External capability protocol?}
    MCP -->|yes| MCPSDK[MCP C# SDK]
    Need --> Durable{Survive process loss or long waits?}
    Durable -->|yes| Engine[Durable Task Temporal Dapr or explicit DB worker]
~~~

Adding an abstraction does not remove provider-specific semantics: streaming event types, structured-output subsets, usage, request IDs, hosted tools, refusals, and retry defaults still require adapter tests.

## Provider SDKs

The official <code>OpenAI</code> .NET library is an API client, generated from OpenAPI in collaboration with Microsoft. It supports Chat and Responses, streaming, tools, structured output, and Azure-compatible endpoints; its client objects are documented as thread-safe and suitable for singleton registration. A stable package does not make every surface stable: current official Responses examples suppress the <code>OPENAI001</code> experimental diagnostic, so isolate and compatibility-test that client surface.

This is distinct from the OpenAI Agents SDK. Official Agents SDK documentation and repositories list Python and TypeScript, not .NET, at the research date. The Responses API is the direct surface for an application-owned loop; an Agents SDK owns more of the loop. For C#, build a small bounded Responses loop behind an adapter or select Microsoft Agent Framework when its orchestration abstractions and maturity fit. Do not describe the <code>OpenAI</code> package itself as an "Agents SDK."

The official Anthropic C# SDK supports Messages, <code>IAsyncEnumerable</code> streaming, typed exceptions, tool use, an <code>IChatClient</code> integration, and platform packages. Its documentation labels the package beta and permits some breaking changes outside strict major-version boundaries. Pin exact versions and isolate it behind an adapter.

Both SDKs retry by default. Align their timeouts and attempts with the runtime-wide policy.

## Microsoft.Extensions.AI and Agent Framework

<code>Microsoft.Extensions.AI</code> provides <code>IChatClient</code>, <code>IEmbeddingGenerator</code>, telemetry, caching, and function-invocation middleware. It is useful for provider portability and middleware composition; it is not a durable runtime, authorization layer, or tool sandbox. Middleware order changes behavior.

Microsoft Agent Framework is the successor path for Microsoft agent abstractions while Semantic Kernel 1.x remains supported. Core 1.0 is production-ready, but package maturity varies. At the research date, self-hosting, some Foundry integrations, and the Durable extension instruct .NET users to install prerelease packages.

An <code>AIAgent</code> may support multiple sessions/concurrent use depending on the implementation. Do not assume thread safety for mutable session/options. Framework messages are passed through; validate and sanitize at the application boundary.

## Azure and Microsoft Foundry

Choose the Azure integration by ownership:

| Need | Preferred surface |
|---|---|
| Standalone Azure OpenAI inference | Official <code>OpenAI</code> SDK against Azure v1 endpoint, with Entra policy where supported |
| Foundry project APIs and Responses | <code>Azure.AI.Projects</code> 2.x plus <code>Azure.AI.Extensions.OpenAI</code> |
| Server-managed Prompt/Hosted Agent | Agent Framework Foundry integration |
| Run your own Agent Framework service | Self-hosting package or a normal ASP.NET Core host |
| Managed containerized agent | Foundry Hosted Agents |

The Foundry .NET 2.0 project SDK is GA, while current Agent Framework Foundry/hosting examples still use prerelease packages. Do not install the older preview <code>Azure.AI.Projects.OpenAI</code> alongside GA <code>Azure.AI.Extensions.OpenAI</code>; Microsoft documents ambiguous type conflicts.

Microsoft now recommends the primary <code>OpenAI</code> SDK for many Azure OpenAI scenarios rather than adding <code>Azure.AI.OpenAI</code>, which remains an Azure-specific companion. Use managed identity/Entra ID in hosted Azure environments and pin server-side agent definitions by version.

## MCP C# SDK

The official SDK provides:

- <code>ModelContextProtocol.Core</code> for lower-level client/server work;
- hosting/DI and stdio support in the main package;
- ASP.NET Core support for Streamable HTTP;
- capability discovery and protocol down-leveling.

Version 2 defaults toward discovery-first, stateless operation and the 2026-07-28 protocol. Version 2.2 adds hybrid support for modern stateless and legacy/stateful clients. Stateless mode scales without affinity; stateful mode requires session lifecycle and routing decisions.

Modern stateless mode omits protocol session state; it does not make an application or long-running task stateless. Externalize business state behind explicit handles. When using the C# SDK, register <code>AddAuthorizationFilters()</code> for <code>[Authorize]</code> policies and test that unauthorized capabilities are hidden from discovery and rejected on direct invocation. HTTP can propagate a <code>ClaimsPrincipal</code>; stdio needs an explicit local identity/trust decision. Structured content/output schemas improve interoperability but do not authorize or sanitize results. The built-in in-memory task store is development-only.

MCP standardizes messages, not trust. Authenticate HTTP callers, authorize each tool/resource, cap all content, and isolate process tools. Stdio stdout is protocol-only.

## Durable runtimes

Agent Framework's Durable Task extension maps agent sessions onto Durable Task/Durable Functions and supports Azure Functions or bring-your-own-container patterns. The extension remains prerelease. Microsoft documents a 1 MB Durable Task scheduler-state limit for these durable agents, manual history compaction, and response/callback rather than unrestricted token streaming; use external storage/references for large content and a separate presentation stream when required.

Temporal and Dapr Workflow are broader durable workflow engines. Their orchestrator/workflow code must remain deterministic; provider calls and tools execute as retryable activities. All activity effects still need idempotency.

Do not adopt a workflow engine merely to run a short request. Use it when restart survival, long timers, signals/approvals, durable retries, or operational history justify the dependency.

## Failure patterns

- Treating a common chat interface as semantic portability.
- Combining framework function middleware with a second manual tool loop.
- Assuming a GA umbrella means every package/API is GA.
- Letting provider SDK and runtime both own retries.
- Equating MCP connectivity with authorization or sandboxing.
- Storing large transcripts in durable workflow state.
- Depending directly on preview types throughout domain code.

## Review checklist

- [ ] Each package has one clear responsibility.
- [ ] Provider-specific events/errors remain behind tested adapters.
- [ ] Package and feature maturity are recorded separately.
- [ ] Azure client choice matches inference versus managed-agent ownership.
- [ ] MCP transport, state, auth, and size policies are explicit.
- [ ] Durable engine is used only for restart/long-wait requirements.
- [ ] Prerelease integrations are pinned, isolated, and canary-tested.

## Primary sources

- [Official OpenAI .NET SDK](https://github.com/openai/openai-dotnet)
- [Official OpenAI API libraries](https://developers.openai.com/api/docs/libraries)
- [OpenAI agent-building and Agents SDK options](https://developers.openai.com/api/docs/guides/agents)
- [Official Anthropic C# SDK](https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/csharp)
- [Microsoft.Extensions.AI](https://learn.microsoft.com/en-us/dotnet/ai/microsoft-extensions-ai)
- [Microsoft Agent Framework](https://learn.microsoft.com/en-us/agent-framework/get-started/)
- [Agent Framework releases](https://github.com/microsoft/agent-framework/releases)
- [Official MCP C# SDK releases](https://github.com/modelcontextprotocol/csharp-sdk/releases)
- [MCP C# authorization filters](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/filters.md)
- [MCP C# tools and structured content](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/tools/tools.md)
- [MCP C# tasks](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/tasks/tasks.md)
- [MCP SDK tiers](https://modelcontextprotocol.io/community/sdk-tiers)
- [Microsoft Foundry SDK overview](https://learn.microsoft.com/en-us/azure/foundry/how-to/develop/sdk-overview)
- [Microsoft Foundry Agent Service integration](https://learn.microsoft.com/en-us/agent-framework/integrations/by-component/agent-services/foundry)
- [Azure OpenAI .NET migration guidance](https://github.com/Azure/azure-sdk-for-net/blob/main/sdk/openai/Azure.AI.OpenAI/migration-guidance.md)
- [Agent Framework durable agents](https://learn.microsoft.com/en-us/azure/durable-task/sdks/durable-agents-microsoft-agent-framework)
- [Temporal .NET SDK releases](https://github.com/temporalio/sdk-dotnet/releases)
- [Dapr Workflow overview](https://docs.dapr.io/developing-applications/building-blocks/workflow/workflow-overview/)

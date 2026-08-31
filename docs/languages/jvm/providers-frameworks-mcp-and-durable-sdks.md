# Providers, Frameworks, MCP, and Durable SDKs

## Verify the exact surface

“Java SDK exists” does not imply agent, realtime, streaming, structured-output, hosted-tool, or MCP parity. Build a feature matrix for the exact version and run a startup/CI capability probe.

| Ecosystem | JVM posture at research date | Engineering consequence |
|---|---|---|
| OpenAI | Java API helper is officially documented as beta; official Agents SDK repositories target Python/TypeScript | implement loop policy or use a JVM framework; do not advertise Agents SDK parity |
| Anthropic | official Java Messages SDK; managed agents/tool runner Java surface is beta | isolate beta types and verify cancellation/streaming |
| Google ADK | Java SDK is Preview/Pre-GA; newer Kotlin ADK exists separately | do not assume Java/Kotlin or Python parity |
| Spring AI | broad provider/tool/MCP/observability integrations in 2.0 line | keep domain loop outside advisors/proxies |
| LangChain4j | Java AI services, tools, memory, MCP | guard memory concurrency and tool parallelism |
| Quarkus LangChain4j | Quarkus-native integration and event-loop offload | verify blocking/request/security context behavior |
| Micronaut LangChain4j | integration documented as experimental | pin and isolate |

## OpenAI on JVM

The officially documented beta <code>openai-java</code> API helper provides synchronous/asynchronous calls, Responses API streaming, structured outputs, and response accumulation. The official SDK page showed `4.54.0` at the 2026-08-31 research snapshot, but its beta status and fast release cadence matter more than copying that coordinate. Its version-support policy and generated API should be checked per release. Official OpenAI documentation links Agents SDK repositories for Python and TypeScript, not Java. Therefore the JVM architecture should treat the Java client as a provider adapter, not as a durable agent runtime.

Keep provider response/tool event objects inside the adapter. Normalize request IDs, usage, finish/refusal states, tool calls, and stream events. Verify endpoint methods because some API surfaces can differ by language.

## MCP Java reality

The official MCP Java SDK 2.0.1 (2026-08-19) is framework-neutral, supports Streamable HTTP, Jackson 2/3 abstraction, bounded STDIO/HTTP reads, and JSON Schema 2020-12 tool-input validation. It tracks the 2025-11-25 protocol. The MCP ecosystem published a 2026-07-28 protocol release, while Java was still outside the four current Tier 1 SDKs (TypeScript, Python, Go, and C#). This is a concrete protocol-lag risk, not a cosmetic version mismatch.

The 2026-07-28 revision removed the initialization handshake and protocol sessions, added `server/discover`, header-based routing, cacheable lists, multi-round-trip requests, and authorization changes. A Java 2.0.x client/server cannot be assumed to interoperate with those semantics merely because both sides use Streamable HTTP. Put the negotiated protocol version in health diagnostics and telemetry, fail capability probes closed, and keep a compatibility deployment for servers you cannot upgrade together.

Do not infer support from compile success. Test initialization, capabilities, tool schema/result content, cancellation, progress, auth, session/header routing, resumability, payload bounds, and reconnect behavior against the server versions you operate.

The Java SDK supplies authentication hooks, not a complete authorization system. Apply transport authentication, tenant binding, tool authorization, and network policy. Prefer Streamable HTTP; legacy SSE is deprecated in Java SDK 2.x. Upgrade past security advisories such as the historical DNS-rebinding fix.

MCP transports tools/resources/prompts. It does not supply durable workflow semantics, exactly-once effects, a sandbox, or business authorization.

## Framework decision rule

Use a framework when its integration removes maintained code you actually need:

- provider adapters and configuration;
- HTTP/stream codecs;
- DI and lifecycle;
- telemetry bridges;
- native/build integration;
- MCP client/server adapters.

Avoid letting it own run state, budgets, idempotency, and durable transitions unless those semantics are explicit and independently testable.

Spring AI tool advisors can run an agentic tool loop, but production policy still needs stable effect IDs, authorization, and cancellation. LangChain4j documents that concurrent calls sharing a memory ID can corrupt chat memory; serialize per conversation or use versioned storage. Quarkus correctly offloads blocking tool work from event-loop paths, but application code must still classify custom tools as blocking.

Framework memory abstractions are conveniences, not the durable source of truth. Map them to the repository's run/context/memory classes explicitly and test concurrent access. Keep provider retry defaults, tool parallelism, and telemetry content capture in configuration snapshots so a library upgrade cannot silently change run semantics.

## Durable SDK selection

| Runtime | Strong fit | Main constraint |
|---|---|---|
| Temporal Java | long workflows, signals, timers, mature operations | deterministic workflow code; effects in Activities |
| Restate Java/Kotlin | durable services, keyed virtual objects, steps | runtime-specific invocation/state model |
| Dapr Workflow Java | Dapr-standardized platform | sidecar/control-plane operational dependency |

Do a replay/versioning proof of concept with one provider call, one ambiguous tool effect, approval, timer, cancellation, and worker restart. Streaming support must be verified; the Temporal Spring AI preview integration explicitly does not support streaming. At the research date, an open maintainer issue tracked Spring AI 2 / Spring Boot 4 support while the contributed module was still tied to the 1.x/Boot 3 line, so BOM compatibility must be proven rather than inferred from “Temporal + Spring AI” branding.

## Adoption gates for moving dependencies

| Maturity | Allowed use | Required containment | Promotion evidence |
|---|---|---|---|
| stable protocol/client | production adapter | pinned graph, contract fixtures, rollback | compatibility, load, cancellation, security scan |
| beta/preview | non-critical or explicitly accepted production path | adapter boundary, feature flag, no public/persisted vendor types | failure drills, canary, owner and exit plan |
| experimental | lab/evaluation by default | separate module/process where risk warrants | eval advantage plus replay/security/operability proof |
| deprecated/protocol-lagged | compatibility bridge only | traffic inventory and removal date | migration test and verified replacement |

For every upgrade, diff generated schemas and streamed fixtures, inspect default retries/timeouts/concurrency, run the capability probe against real staging servers, replay saved durable histories, canary on comparable traffic, and retain the previous image. A passing compile is not an adoption gate.

## Capability test template

- [ ] non-stream and stream completion/refusal/error mapping;
- [ ] structured output subset and generated schema;
- [ ] multi-tool and parallel-tool ordering;
- [ ] cancellation before headers, mid-stream, and during tool;
- [ ] request ID, token/usage, and retry visibility;
- [ ] proxy/TLS/timeouts and response byte caps;
- [ ] MCP protocol/capability/version matrix;
- [ ] auth and tenant isolation;
- [ ] serialization and framework BOM convergence;
- [ ] upgrade/rollback with saved fixtures and durable history.

## Sources

- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [OpenAI Java Responses API](https://developers.openai.com/api/reference/java/resources/beta/subresources/responses)
- [OpenAI SDKs and Agents SDK language links](https://developers.openai.com/api/docs/libraries)
- [Anthropic client SDKs](https://platform.claude.com/docs/en/cli-sdks-libraries/overview)
- [Google ADK Java](https://github.com/google/adk-java)
- [MCP Java SDK changelog](https://github.com/modelcontextprotocol/java-sdk/blob/main/CHANGELOG.md)
- [MCP 2026-07-28 release](https://blog.modelcontextprotocol.io/posts/2026-07-28/)
- [MCP SDK tiers](https://modelcontextprotocol.io/community/sdk-tiers)
- [Spring AI reference](https://docs.spring.io/spring-ai/reference/index.html)
- [LangChain4j AI Services](https://docs.langchain4j.dev/tutorials/ai-services/)
- [Temporal Spring AI integration](https://github.com/temporalio/sdk-java/blob/main/contrib/temporal-spring-ai/README.md)
- [Temporal Spring AI 2 / Boot 4 support issue](https://github.com/temporalio/sdk-java/issues/2920)

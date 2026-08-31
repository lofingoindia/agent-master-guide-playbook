# HTTP Clients, Pooling, and Resilience

> **Last researched:** 2026-08-31
> **Applies to:** .NET 10 LTS, C# 14

Most .NET agent incidents attributed to a provider are transport-lifetime or policy-composition failures: socket exhaustion, stale DNS, captured factory clients, layered retries, oversized responses, or timeouts that abandon live requests.

## Choose one transport lifetime model

| Model | Configuration | Use when |
|---|---|---|
| Long-lived client | Singleton <code>HttpClient</code> with <code>SocketsHttpHandler.PooledConnectionLifetime</code> | Direct clients, explicit transport ownership |
| Factory-created client | <code>IHttpClientFactory</code> with configured handler lifetime | Centralized named/typed clients and handlers |

<code>HttpClient</code> resolves DNS when a connection is created and does not track DNS TTL itself. A long-lived client therefore needs a finite pooled connection lifetime appropriate to the environment. Factory clients are intended to be short-lived while their handlers are pooled.

Do not capture a typed factory client in a singleton indefinitely; that can defeat handler rotation. Also note that handler dependency-injection scopes are separate from application request scopes. Do not store user or run context in a handler-scoped service. Factory pooling can share cookies and discard them when handlers rotate, so cookie-dependent workflows require deliberate isolation.

## One resilience budget

~~~mermaid
flowchart LR
    Run[Run attempt policy] --> SDK[Provider SDK retries]
    SDK --> Handler[HTTP resilience handler]
    Handler --> Network[Network]
~~~

Only one layer should normally own retries. Otherwise attempts multiply. The official OpenAI .NET client retries selected transient statuses up to three additional times by default; the official Anthropic C# client retries twice by default. Adding the standard .NET resilience handler and a queue retry can turn one business operation into many network attempts.

Inventory every layer:

| Layer | Count | Timeout | Retryable operations | Idempotency guard |
|---|---:|---:|---|---|
| Queue delivery | explicit | lease/visibility | whole work item | message/effect ID |
| Run policy | explicit | total budget | classified steps | run journal |
| Provider SDK | inspect package | network/request | provider-defined | often none |
| HTTP handler | inspect options | attempt and total | status/method policy | method policy |

Disable or reduce lower-layer retries when the run coordinator needs attempt-level observability, cost control, or side-effect reconciliation.

## Standard resilience handler caveats

The standard <code>Microsoft.Extensions.Http.Resilience</code> handler composes rate limiting, total timeout, retry, circuit breaker, and attempt timeout. Its defaults are a starting point, not an agent policy. In particular, retry behavior can include unsafe HTTP methods unless disabled. Do not stack multiple standard handlers.

A circuit breaker should be partitioned by a stable dependency boundary such as provider endpoint and deployment, not by run ID. Too many partitions eliminate useful aggregation; too few let one failing deployment block unrelated traffic.

Honor provider retry guidance and <code>Retry-After</code>, add jitter, and cap every delay by the remaining run deadline. Rate-limit before sending work the downstream service cannot accept.

## Response and stream ownership

Use <code>HttpCompletionOption.ResponseHeadersRead</code> for streaming or large responses, then own and dispose the response and content stream. Enforce:

- maximum header and body bytes;
- maximum frame size;
- time to first byte and idle gap;
- decompression limits;
- accepted content type and encoding;
- redirect policy and destination validation for tool-controlled URLs.

Reading response headers successfully does not mean the stream will complete. Midstream errors are attempt failures with partial output; do not blindly replay side-effecting tool calls derived from a previous partial stream.

## Request safety

HTTP retries are safe only when the business operation is replay-safe. A POST can be transport-idempotent when it carries a provider-supported idempotency key, but model generation still incurs latency/cost and may produce a different result. Tool effects require a separate application idempotency protocol.

Capture provider request IDs, safe rate-limit metadata, attempt number, handler outcome, and elapsed time. Never put API keys, full prompts, or full bodies in handler logs.

## Recommended client boundary

~~~csharp
public interface IModelGateway
{
    Task<ModelResponse> SendAsync(
        ModelRequest request,
        AttemptContext attempt,
        CancellationToken cancellationToken);
}
~~~

The gateway adapts one provider SDK into domain events and exceptions. It does not own cross-provider fallback, durable effects, or the run-level retry budget. This keeps SDK-specific timeouts and error shapes visible without letting them dictate the whole runtime.

## Failure patterns

- New <code>HttpClient</code> per model call causes connection churn.
- One immortal client has no connection lifetime and follows stale DNS indefinitely.
- A factory client is captured by a singleton.
- SDK, handler, and worker each retry three times.
- Timeout of the wait is treated as proof the request stopped.
- A streaming response is not disposed after client disconnect.
- Retrying all methods duplicates uploads or remote side effects.
- A handler scope carries tenant identity across unrelated requests.

## Review checklist

- [ ] Transport lifetime and DNS rotation are explicit.
- [ ] Exactly one layer owns normal retries.
- [ ] Attempt and total deadlines are both bounded.
- [ ] Unsafe methods are excluded unless replay safety is proven.
- [ ] Responses and streams are disposed on every path.
- [ ] Body, frame, and decompressed sizes are capped.
- [ ] Metrics show logical calls separately from network attempts.

## Primary sources

- [HttpClient guidelines](https://learn.microsoft.com/en-us/dotnet/fundamentals/networking/http/httpclient-guidelines)
- [IHttpClientFactory guidance](https://learn.microsoft.com/en-us/dotnet/core/extensions/httpclient-factory)
- [Build resilient HTTP apps](https://learn.microsoft.com/en-us/dotnet/core/resilience/http-resilience)
- [Standard resilience options source](https://github.com/dotnet/extensions/blob/main/src/Libraries/Microsoft.Extensions.Http.Resilience/Resilience/HttpStandardResilienceOptions.cs)
- [.NET HTTP tracing](https://learn.microsoft.com/en-us/dotnet/fundamentals/networking/telemetry/tracing)
- [OpenAI .NET automatic retries](https://github.com/openai/openai-dotnet#automatically-retrying-errors)
- [Anthropic C# retries and timeouts](https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/csharp)

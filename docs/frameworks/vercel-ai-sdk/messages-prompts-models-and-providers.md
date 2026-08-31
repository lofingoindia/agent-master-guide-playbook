# Messages, Prompts, Models, and Providers

> Research date: **2026-08-31** | Applies to AI SDK 7 and provider specification v4.

AI SDK portability comes from layered translation, not from a lowest-common-denominator model. Preserve each layer and make provider capability differences explicit.

## Four message layers

| Layer | Purpose | Persist? | Trust |
| --- | --- | --- | --- |
| `UIMessage` | Rendered conversation, IDs, metadata, tool/data parts | Yes, with a schema version | Untrusted after browser or storage round trip |
| `ModelMessage` | Provider-neutral inference history | Usually derived | Server-validated input |
| `LanguageModelV4` prompt | Provider-spec interchange | No | SDK/provider boundary |
| Provider-native request | Actual wire protocol | Only sanitized diagnostics | Provider-specific |

```mermaid
flowchart LR
    U[UIMessage] -->|validateUIMessages| V[Validated UI state]
    V -->|convertToModelMessages| M[ModelMessage]
    M --> S[LanguageModelV4 prompt]
    S --> P[Provider-native payload]
```

Do not send client messages directly to a model. Stored tool parts can become invalid after schemas change, and clients can forge approval, result, metadata, or data parts. Validate with current `tools`, `metadataSchema`, and `dataPartsSchema`, authorize the conversation, then convert.

`UIMessage` is the correct persistence layer for chat because it preserves IDs and renderable parts. `ModelMessage` is optimized for inference and may omit UI state. Give persisted UI messages an application schema version and migration path.

## Prompt construction

Keep non-negotiable behavior in `instructions`, not repeated user messages. Treat all retrieved documents, tool results, uploaded files, and provider-generated text as untrusted content even when inserted into a system-controlled prompt.

Prefer structured message parts for files and multimodal data. Validate media type, byte size, ownership, and URL policy before calling the SDK. Context-window management should be observable: record which messages were selected, summarized, or dropped, and do not silently discard approval or tool-result parts required for protocol validity.

## Model resolution

A string such as `openai/gpt-5-mini` uses the SDK's default provider, which is Vercel AI Gateway unless the global `AI_SDK_DEFAULT_PROVIDER` is replaced. A direct model instance calls its provider package:

```ts
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';

await generateText({ model: 'openai/gpt-5-mini', prompt: '...' }); // Gateway
await generateText({ model: openai('gpt-5-mini'), prompt: '...' }); // Direct
```

Use `createProviderRegistry` or `customProvider` to publish an application-owned allowlist and aliases. This prevents arbitrary client-supplied model names from selecting an expensive, unapproved, or regionally non-compliant endpoint.

## Portability is a tested contract

Provider adapters normalize method signatures, stream parts, usage, finish reasons, warnings, and errors. They cannot erase differences in:

- supported modalities and maximum input/output sizes;
- JSON Schema coverage and strict structured output;
- tool parallelism, hosted/provider-executed tools, and approval protocols;
- reasoning controls and whether reasoning is visible;
- cache-token accounting and usage timing;
- stream ordering, partial tool arguments, error frames, and finish reasons;
- request IDs, safety metadata, citations, and raw response fields.

Top-level `reasoning` settings are mapped to provider-native values where possible; a provider can coerce or warn, and `providerOptions` can override common behavior. Treat warnings as operational signals, not noise.

Maintain a small capability table in code or configuration:

| Capability | Required policy |
| --- | --- |
| Tool calling | Contract test valid, invalid, parallel, and provider-executed calls |
| Structured output | Verify the exact schema features and strictness used |
| Streaming | Verify text, reasoning, tool parts, usage, abort, and error order |
| Data residency | Route only to approved provider/region/account |
| Safety controls | Record provider configuration and application moderation |

Fail closed when a selected model lacks a required capability. Automatic fallback is unsafe when the fallback changes residency, retention, tool behavior, or schema guarantees.

## AI Gateway boundary

AI Gateway is an optional managed service for authentication, routing, provider fallback, budgets, usage, and observability. Its default dynamic routing can choose among eligible providers based on availability and latency unless routing constraints specify `order` or `only`.

Gateway retry/fallback and SDK retry are different layers. Configure one total attempt budget and record every attempted provider. A fallback after a partially observed stream or an ambiguous provider-executed action requires special handling; it is not equivalent to retrying a read-only request before any bytes arrive.

## Operational metadata

For each call, retain the requested application model alias and the resolved provider/model when available, SDK/provider package versions, request/response IDs, warnings, finish reason, normalized usage, cache-token detail, latency milestones, and selected Gateway endpoint. Do not log raw prompts or responses by default.

## Sources

- [`UIMessage`](https://ai-sdk.dev/docs/reference/ai-sdk-core/ui-message) and [`ModelMessage`](https://ai-sdk.dev/docs/reference/ai-sdk-core/model-message)
- [Prompts and messages](https://ai-sdk.dev/docs/ai-sdk-core/prompts)
- [Provider management](https://ai-sdk.dev/docs/ai-sdk-core/provider-management)
- [Provider options](https://ai-sdk.dev/docs/foundations/provider-options)
- [AI SDK providers](https://ai-sdk.dev/providers/ai-sdk-providers)
- [AI Gateway](https://vercel.com/docs/ai-gateway)
- [Provider specification source](https://github.com/vercel/ai/tree/main/packages/provider/src)

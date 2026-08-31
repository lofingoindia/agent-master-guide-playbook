# Middleware, Telemetry, and Observability

> Research date: **2026-08-31** | Applies to AI SDK 7, `@ai-sdk/otel` 1.x, and `@ai-sdk/devtools` 1.x.

Observability must describe the application's logical run and every underlying attempt without turning prompts, files, tool arguments, or credentials into a second sensitive database.

## Middleware is protocol code

`wrapLanguageModel` composes middleware around a model. Middleware can transform parameters, wrap non-streaming generation, and wrap streaming generation. When passed as an array, the first middleware is the outermost wrapper.

Built-in middleware includes reasoning extraction, JSON extraction, simulated streaming, default instructions/settings, and tool-input examples. These are useful but can change message or stream structure. Test the transformed contract, especially tool-call IDs, partial arguments, finish reasons, usage, warnings, and errors.

Good middleware uses:

- application model aliases and controlled defaults;
- policy-based tool visibility;
- redacted tracing and consistent correlation IDs;
- provider conformance normalization backed by tests.

Risky middleware uses:

- hidden authorization based on prompt text;
- retries that ignore stream/effect state;
- caches that omit tenant, policy, provider, model version, tools, or options;
- logging raw requests and responses globally.

Never cache a side-effecting agent turn as if it were pure text generation. If caching inference, define a canonical key over provider/model/version, messages, tools and schemas, provider options, policy version, tenant/data boundary, and output contract. Encrypt or hash sensitive key material and bound retention.

## AI SDK 7 telemetry model

Telemetry is registered globally:

```ts
import { registerTelemetry } from 'ai';
import { OpenTelemetry } from '@ai-sdk/otel';

registerTelemetry(new OpenTelemetry());
```

After an integration is registered, calls emit telemetry by default unless disabled per call. `recordInputs` and `recordOutputs` default to true. In sensitive systems, set both false by policy and add only reviewed attributes. Runtime and tool context are excluded unless `includeRuntimeContext` or `includeToolsContext` is enabled, but lifecycle callbacks and result objects still receive full values.

Prefer current OpenTelemetry GenAI semantic conventions over legacy attribute shapes. Keep a stable application run ID that links HTTP request, chat turn, model step, provider attempt, tool call, approval, workflow run, and persistence transaction.

## Recommended signal model

| Level | Record |
| --- | --- |
| Run | tenant-safe correlation ID, feature, policy version, terminal state, total latency/cost |
| Model step | requested/resolved model, provider, attempt, tokens, finish reason, warnings, first-content latency |
| Tool call | tool/version, approval decision, duration, result class, effect/idempotency ID |
| Stream | disconnect/abort, chunks/bytes, inter-chunk gaps, reconnect/reset count |
| Workflow | run/step IDs, replay/retry count, suspension reason, queue delay, terminal status |

Use controlled enums and bounded labels. Do not put user IDs, prompts, URLs, document names, tool arguments, or error text into high-cardinality metric labels. Store sampled, redacted diagnostic payloads under a separate access and retention policy.

## DevTools

AI SDK DevTools is for local development. It records generations in plain text under `.devtools/generations.json`, including prompts, outputs, tool arguments/results, and optionally raw bodies. Keep `.devtools/` ignored, prevent the telemetry integration from loading in production, and use synthetic data when sharing recordings.

## Alerts that indicate user harm

Alert on outcomes, not raw request volume alone:

- elevated no-content, invalid-output, or unknown finish reasons;
- tool denial/approval timeout or ambiguous-effect rates;
- retries or provider fallbacks per successful turn;
- p95 time to first content and inter-chunk gap;
- incomplete persisted messages or active streams without live runs;
- token/spend reservation overruns by tenant;
- workflow replay/reset loops and age of oldest suspended run;
- leakage guard detections in client-visible errors.

## Sources

- [Language model middleware](https://ai-sdk.dev/docs/ai-sdk-core/middleware)
- [Telemetry](https://ai-sdk.dev/docs/ai-sdk-core/telemetry)
- [AI SDK DevTools](https://ai-sdk.dev/docs/ai-sdk-core/devtools)
- [AI SDK 7 migration: telemetry](https://ai-sdk.dev/docs/migration-guides/migration-guide-7-0)
- [`@ai-sdk/otel` source](https://github.com/vercel/ai/tree/main/packages/otel)
- [OpenTelemetry GenAI conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)


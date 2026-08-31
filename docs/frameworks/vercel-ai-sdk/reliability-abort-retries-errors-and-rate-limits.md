# Abort, Retries, Errors, and Rate Limits

> Research date: **2026-08-31** | Applies to AI SDK 7.

Reliability is a coordinated budget across application, SDK, provider, Gateway, workflow, and client. Independent defaults can multiply attempts, cost, and side effects.

## One attempt budget

Core generation defaults `maxRetries` to 2: an initial attempt plus up to two retries. Current source uses exponential backoff and does not retry abort errors. Provider/Gateway behavior and Workflow step retries can add more attempts.

Define a single policy that answers:

- how many provider attempts may one logical model step make;
- whether Gateway fallback consumes the same budget;
- how many Workflow step attempts are allowed;
- which tools are safe to retry;
- maximum elapsed time and spend across all layers.

Set `maxRetries: 0` when an outer layer owns retry or when replay would be unsafe. Log logical step and physical attempt separately.

## Timeout model

AI SDK 7 accepts a number or structured timeout fields including total, step, first-content/chunk, inter-chunk, tool, and per-tool limits. Use a total deadline plus narrower timeouts:

```ts
timeout: {
  totalMs: 90_000,
  stepMs: 30_000,
  firstChunkMs: 12_000,
  chunkMs: 20_000,
  toolMs: 15_000,
  tools: { chargeCard: 8_000 },
}
```

A maintainer issue against 7.0.28 reported that step/chunk timeout accounting included tool time. Current 7.0.85 documentation and source expose more precise first-content and tool controls. Treat the old report as a regression test; do not claim it remains a current defect without reproducing it on the pinned version.

## Abort propagation

Pass the request's `AbortSignal` into Core and from the tool execution options into downstream clients. A signal is cooperative: the HTTP library, SDK, provider, database driver, and tool code must observe it.

On Vercel, request cancellation must be enabled per route with `supportsCancellation`, and official documentation limits the feature to the Node.js runtime. Without it, closing the browser stream may not stop the function or provider request.

For Core streaming, `onAbort` receives completed steps; `onEnd` is not the abort callback. At the UI conversion layer, terminal handling can report `isAborted`. Test both. Stream resumption intentionally keeps work alive after disconnect, so use a dedicated authorized stop endpoint when resume is enabled.

## Error taxonomy

| Class | Retry? | Client exposure |
| --- | --- | --- |
| Invalid input/schema | No | Safe validation message |
| Authentication/authorization | No | Generic denial; no resource leakage |
| Provider rate limit/temporary outage | Within deadline and attempt budget | Retry status, not raw body |
| Provider bad request/model capability | Usually no | Correlation ID |
| Tool transient read | Maybe | Generic tool failure |
| Tool write timeout | Reconcile first | Pending/unknown outcome |
| Abort/deadline | No automatic retry unless a new operation is intended | Stopped/timed out |
| Stream error after bytes | Do not invisibly restart into same message | Mark partial output and offer explicit retry |

Current stream errors can be normalized as `StreamProviderError` with fields such as status, code, and retryability. Once a user has observed partial output, an automatic provider restart can duplicate or contradict content. Start a new attempt/message or reset the failed step with explicit protocol semantics.

## Rate and cost limiting

Limit by authenticated tenant, account, user, API credential, feature, and model cost—not IP alone. Use an atomic reservation before starting work, settle against actual usage, and expire abandoned reservations. Enforce concurrent active runs and queued work in addition to requests per minute.

Model steps and tool effects need separate policies. A cheap request can trigger an expensive multi-step loop or a high-impact external action. Reject before the model call when the remaining budget cannot cover the minimum safe completion path.

## Recovery rules

- Retrying a read is usually safe; retrying a write requires a stable operation ID.
- Timeout means the caller stopped waiting, not that the provider stopped working.
- Abort is a requested terminal state; persist it even if downstream cancellation is best-effort.
- A client disconnect is not necessarily abort when resumption is enabled.
- A fallback is a new provider attempt with potentially different policy and capability.
- Never report success until authoritative state and user-visible message are durably consistent.

## Sources

- [AI SDK error handling](https://ai-sdk.dev/docs/ai-sdk-core/error-handling)
- [Stopping streams](https://ai-sdk.dev/docs/advanced/stopping-streams)
- [AI SDK 7 timeout configuration](https://ai-sdk.dev/docs/migration-guides/migration-guide-7-0)
- [Vercel deployment guide](https://ai-sdk.dev/docs/advanced/vercel-deployment-guide)
- [Vercel request cancellation](https://vercel.com/docs/functions/functions-api-reference#cancel-requests)
- [Timeout regression report #17310](https://github.com/vercel/ai/issues/17310)
- [Retry implementation source](https://github.com/vercel/ai/blob/main/packages/provider-utils/src/retry-with-exponential-backoff.ts)

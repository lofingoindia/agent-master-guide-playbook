# HTTP, Streaming, and Backpressure

## Treat transport as a bounded protocol

An agent HTTP client needs independent limits for connection establishment, response headers, idle progress, total operation time, decompressed bytes, event count, and buffered data. A single request timeout does not cover slow streaming or unlimited response bodies.

Reuse <code>java.net.http.HttpClient</code> instances so connection pools and protocol negotiation are reused. Set a client connect timeout and a per-request timeout, then enforce the run deadline outside both. Always consume, close, or cancel response bodies. Oracle warns that failing to do so can prevent resource release and orderly client shutdown.

## Streaming pipeline

~~~mermaid
flowchart LR
    S[Socket bytes] --> D[Bounded decoder]
    D --> E[Protocol events]
    E --> Q[Bounded queue]
    Q --> A[Stateful assembler]
    A --> O[Client or run log]
~~~

Each stage needs:

- maximum frame/line size;
- maximum cumulative decompressed bytes;
- bounded queue capacity;
- cancellation propagation upstream;
- defined behavior for malformed, duplicate, or out-of-order events;
- a terminal-state check when EOF arrives.

Do not use <code>BodyHandlers.ofString()</code> for an unbounded model or tool response. Prefer <code>ofInputStream</code>, <code>ofLines</code>, or a publisher/subscriber path and close it on every exit. Beware that convenience consumers may not provide flow control.

A blocking virtual-thread path can stay simple while enforcing bytes before materialization:

~~~java
HttpResponse<InputStream> response = client.send(
    request,
    HttpResponse.BodyHandlers.ofInputStream()
);
try (InputStream raw = response.body();
     InputStream bounded = new CountingLimitInputStream(raw, MAX_DECOMPRESSED_BYTES)) {
    return decodeProviderEvents(bounded, MAX_FRAME_BYTES, MAX_EVENTS, runBudget);
}
~~~

`CountingLimitInputStream` represents an application-owned wrapper that aborts when the decompressed byte count crosses the limit; apply compressed-wire limits at the transport/gateway too. `decodeProviderEvents` must enforce an idle-progress deadline in addition to the total run deadline. Do not implement a byte limit by reading the whole body and checking `length` afterward.

## Backpressure choices

| Stream | Safe overflow behavior |
|---|---|
| token/content deltas | suspend/demand-control or cancel; never silently drop |
| tool-call argument deltas | lossless and ordered |
| persisted run transitions | lossless; write-ahead or bounded fail |
| UI progress snapshots | conflate/drop-oldest if explicitly snapshots |
| telemetry | sample/drop by documented policy |

Java's <code>Flow</code> demand signals only protect a pipeline when every bridge honors them. A callback SDK that ignores demand needs a bounded adapter queue and a failure mode. Kotlin <code>Flow.buffer</code> introduces a channel; select capacity and overflow explicitly. Hot <code>SharedFlow</code> is not automatically a reliable event bus.

Size the adapter queue from bytes, not only event count. A queue of 1,000 events is still unsafe if one event may contain a multi-megabyte tool result. Track queued bytes atomically with enqueue/dequeue. If a lossless queue is full, stop requesting/reading upstream when the transport supports demand; otherwise cancel the response and persist a classified `STREAM_BACKPRESSURE_OVERFLOW` failure. Never spill executable tool arguments to an unencrypted temporary file as an emergency buffer.

## Parsing incremental tool calls

Provider streams often split JSON at arbitrary byte/token boundaries. Accumulate by provider item/call ID, not by array position or arrival thread. Enforce a maximum argument size before parsing. Validate complete JSON only after the protocol marks the item complete, then validate it against the tool schema. A stream ending mid-item is a protocol failure, never partial success.

Keep provider event types inside the adapter. Emit normalized internal events such as:

- response started;
- text delta;
- tool-call delta;
- item completed;
- usage reported;
- response completed or failed.

The assembler must reject an impossible transition and preserve unknown provider events for diagnostics without treating them as executable instructions.

## Retries and streaming

Retry only before any externally visible output or effect, unless the protocol supports resumable offsets and de-duplication. Restarting a stream after forwarding partial tokens can duplicate or contradict content. If user-facing streaming is required, assign event sequence numbers and make clients capable of de-duplication; still do not assume providers can resume generation.

Client reconnect is a separate contract from provider-stream retry. Persist normalized events with monotonically increasing run sequence numbers, return a bounded replay window or durable cursor to the client, and fence terminal events so a late producer cannot append after completion. If the requested cursor has expired, return an explicit snapshot/resync response rather than silently starting a second model generation.

## Kotlin and Java implementation notes

- Java blocking body reads are an excellent virtual-thread fit.
- A Java asynchronous client can still buffer unexpectedly if body subscribers are chosen poorly.
- Kotlin adapters should use a cancellable bridge for callback APIs and close upstream when collection stops.
- Never block a framework event loop. Quarkus and reactive Spring paths require explicit offload for blocking SDK/tool work.
- TLS, proxies, DNS, decompression, and redirects consume the same total deadline; redirect counts must be bounded.

## Checklist

- [ ] One reusable client per configuration/tenant boundary.
- [ ] Connect, request, idle, and absolute run deadlines exist.
- [ ] Compressed and decompressed byte limits exist.
- [ ] Every body is consumed, closed, or cancelled.
- [ ] Every bridge has bounded capacity and overflow policy.
- [ ] Lossless agent events cannot be dropped.
- [ ] Incremental JSON is keyed and size-limited.
- [ ] Mid-stream retry and client de-duplication behavior are explicit.

## Sources

- [Java 25 HttpClient API](https://docs.oracle.com/en/java/javase/25/docs/api/java.net.http/java/net/http/HttpClient.html)
- [Java 25 BodySubscribers API](https://docs.oracle.com/en/java/javase/25/docs/api/java.net.http/java/net/http/HttpResponse.BodySubscribers.html)
- [Java Flow API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/Flow.html)
- [Kotlin Flow buffer API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/buffer.html)

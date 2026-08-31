# Channels, Streaming, and Backpressure

> **Last researched:** 2026-08-31
> **Applies to:** .NET 10 LTS, C# 14

Agent streams are not merely UI output. They carry provider events, partial tool calls, usage records, errors, and terminal markers across components with different speeds. The production requirement is bounded flow: a slow consumer must cause a deliberate wait, coalescing decision, spill, or disconnect rather than unbounded memory growth.

## Pick the right primitive

| Primitive | Best fit | Bound expressed as |
|---|---|---|
| <code>IAsyncEnumerable&lt;T&gt;</code> | Pull-based single logical stream | Consumer demand, plus external budgets |
| <code>Channel&lt;T&gt;</code> | Multi-producer/consumer handoff | Item count |
| <code>PipeReader/PipeWriter</code> | High-throughput byte protocols | Pause/resume byte thresholds |
| Durable broker | Restart-safe delivery | Broker quota, retention, and consumer policy |

A <code>Channel&lt;T&gt;</code> capacity of 100 means 100 items, not 100 bytes. Model events can range from a few bytes to large documents, so maintain a separate retained-byte budget or normalize large payloads into references.

## A byte bound you can prove

The simplest reliable byte bound is to make every channel item small by contract. Serialize once to UTF-8, reject or externalize oversized payloads, then combine maximum inline bytes with channel capacity.

~~~csharp
const int MaxInlineEventBytes = 64 * 1024;

static async Task<BufferedEvent> NormalizeAsync(
    byte[] utf8Json,
    IBlobStore blobs,
    CancellationToken token)
{
    if (utf8Json.Length <= MaxInlineEventBytes)
    {
        return BufferedEvent.Inline(utf8Json);
    }

    BlobReference reference = await blobs.PutImmutableAsync(utf8Json, token);
    return BufferedEvent.Reference(reference, utf8Json.Length);
}
~~~

With capacity 256, this contract retains at most 16 MiB of inline payload plus object/channel overhead. Ownership of <code>utf8Json</code> transfers to the event; callers must not mutate or return its buffer to a pool. Enforce the limit before enqueueing and bound the blob operation too. If events cannot be normalized to a fixed maximum, reserve weighted byte capacity before writing or use <code>PipeOptions</code> pause/resume thresholds for a byte stream; do not assume an item-count semaphore supplies a byte bound.

## Bounded channel policy

~~~csharp
var channel = Channel.CreateBounded<AgentEvent>(
    new BoundedChannelOptions(capacity: 256)
    {
        FullMode = BoundedChannelFullMode.Wait,
        SingleWriter = false,
        SingleReader = true,
        AllowSynchronousContinuations = false
    });
~~~

<code>Wait</code> is the safe default for events that must not disappear. Drop modes can be correct for replaceable progress snapshots, but never silently drop tool calls, approval requests, usage/accounting records, state transitions, or terminal errors.

If dropping is intentional:

- name the event class as lossy;
- use the dropped-item callback or equivalent counter;
- preserve the latest coherent snapshot, not arbitrary fragments;
- never let a dropped terminal marker leave the consumer waiting forever.

<code>SingleReader</code> and <code>SingleWriter</code> are promises to the implementation, not hints. Set them only when architecture guarantees them. Keep synchronous continuations disabled unless reentrancy and producer latency have been audited.

## Completion and exception ownership

The producer that owns the logical stream completes the writer exactly once, preferably with <code>TryComplete(error)</code>. Consumers drain with <code>ReadAllAsync(token)</code> and still inspect or await completion when they need the terminal exception.

~~~csharp
static async Task PumpAsync(
    IAsyncEnumerable<ProviderEvent> source,
    ChannelWriter<AgentEvent> destination,
    CancellationToken token)
{
    Exception? terminal = null;
    try
    {
        await foreach (ProviderEvent item in source.WithCancellation(token))
        {
            AgentEvent mapped = ValidateAndMap(item);
            await destination.WriteAsync(mapped, token);
        }
    }
    catch (Exception error)
    {
        terminal = error;
        throw;
    }
    finally
    {
        destination.TryComplete(terminal);
    }
}
~~~

Do not start this pump as unowned fire-and-forget work. The run owner must await it and decide whether an early client disconnect cancels the provider request, continues durable work, or detaches only the presentation stream.

## Async streams

An iterator should accept a token with <code>[EnumeratorCancellation]</code>, or the consumer should apply <code>WithCancellation</code>. Ensure enumerators and response streams reach <code>DisposeAsync</code> on success, failure, and cancellation.

Avoid exposing a provider SDK event type as the domain event contract. Normalize it so provider changes do not alter persistence, UI, or protocol semantics. Preserve unknown events for diagnostics only under a bounded safe representation.

## Pipelines for byte protocols

<code>System.IO.Pipelines</code> is appropriate for SSE, framed protocols, or proxying large bodies when copies matter. Its contract is strict:

- always call <code>AdvanceTo(consumed, examined)</code>;
- do not use a sequence after advancing;
- complete both reader and writer;
- treat <code>FlushAsync</code> as the backpressure point;
- configure pause and resume thresholds based on bytes;
- enforce maximum frame length before materializing a string or JSON document.

Incorrect consumed/examined positions can cause hangs, busy loops, data loss, or unbounded retention.

## End-to-end pressure

~~~mermaid
flowchart LR
    Provider -->|HTTP bytes| Parser
    Parser -->|events| Channel
    Channel --> Mapper
    Mapper -->|domain events| Client
    Client -. slow .-> Mapper
    Mapper -. await .-> Channel
    Channel -. await .-> Parser
    Parser -. stop reading .-> Provider
~~~

Backpressure is useful only when propagated. Adding an unbounded list, background send queue, or logging exporter anywhere in the chain breaks the bound.

Define a slow-consumer policy for interactive clients:

1. coalesce replaceable progress;
2. cap queued bytes and age;
3. send a resumable cursor if durable events exist;
4. disconnect with a reason when the budget is exceeded;
5. separately decide whether the underlying run continues.

## Failure patterns

- Building the entire response in a <code>StringBuilder</code> before emitting it.
- Buffering an unlimited number of streaming SDK events because the API is asynchronous.
- Treating SSE comments as business progress for idle timeout renewal.
- Using <code>DropOldest</code> on a mixed channel and losing an approval request.
- Completing a reader without completing the writer or observing its exception.
- Writing protocol data and logs to the same stdout stream of an MCP stdio server.

## Review checklist

- [ ] Every in-memory queue is bounded.
- [ ] Variable-size items have a retained-byte limit.
- [ ] Event loss policy is explicit by event class.
- [ ] Producer completion and terminal errors reach consumers.
- [ ] Stream pumps are owned and disposed.
- [ ] Slow-consumer tests verify memory plateaus and terminal delivery.

## Primary sources

- [.NET channels](https://learn.microsoft.com/en-us/dotnet/core/extensions/channels)
- [Generate and consume async streams](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/generate-consume-asynchronous-stream)
- [System.IO.Pipelines](https://learn.microsoft.com/en-us/dotnet/standard/io/pipelines)
- [BoundedChannel runtime source](https://github.com/dotnet/runtime/blob/main/src/libraries/System.Threading.Channels/src/System/Threading/Channels/BoundedChannel.cs)
- [.NET queued background service](https://learn.microsoft.com/en-us/dotnet/core/extensions/queue-service)
- [MCP C# transports](https://github.com/modelcontextprotocol/csharp-sdk/blob/main/docs/concepts/transports/transports.md)

# Streaming, Events, PubSub, and Clients

Mastra streams expose incremental model and workflow activity. A stream is not
the durable record of a run. Build clients that can lose the connection,
reconnect, replay what is available, and reconcile against persistent state.

## Agent and workflow streams

Agent streaming offers:

- a text-oriented stream for simple rendering;
- a full structured stream for start, step, text, reasoning, tool calls, tool
  results, approval/suspension, finish, error, abort, and custom data.

Workflow streams emit structured lifecycle events for steps, branches, state,
waiting/suspension, and terminal status.

Use the structured stream for any product that displays tools, approvals,
progress, or recovery. Text deltas alone cannot represent the control state.

## Stream consumer state machine

~~~mermaid
stateDiagram-v2
    [*] --> Connecting
    Connecting --> Live: stream accepted
    Live --> Live: ordered event
    Live --> Reconnecting: network loss
    Reconnecting --> Replaying: resume supported
    Reconnecting --> Reconciling: replay unavailable
    Replaying --> Live
    Reconciling --> Live: active run found
    Reconciling --> Terminal: run finished
    Live --> Terminal: finish/error/abort
    Terminal --> [*]
~~~

Store stable IDs before consuming the first event:

- product operation ID;
- thread and resource ID;
- Mastra run ID;
- workflow/agent ID and version profile;
- last committed event offset or ID;
- latest product status.

## Writer contract

Tools and workflow steps can write custom events. Await each
<code>writer.write()</code>. Failing to await writes can violate stream locking
and reorder or lose product progress.

Use custom events for bounded progress, not internal object dumps. Mark
ephemeral UI-only events as transient so they are not persisted where supported.
Never include credentials, raw authorization tokens, or unredacted documents.

## Terminal handling

A robust consumer:

1. consumes until a terminal event or explicit close;
2. handles error and abort as first-class outcomes;
3. closes or aborts on navigation/cancellation;
4. persists the latest offset after applying an event;
5. makes event application idempotent;
6. queries durable run state when terminal delivery is uncertain.

Do not equate HTTP/SSE disconnect with run failure. The server may still be
working, may have completed, or may have lost its own process.

## PubSub backends

Mastra PubSub is used by workflows, schedules, background tasks, signals, and
resumable streaming.

| Backend | Topology | Replay/durability posture |
|---|---|---|
| Default EventEmitter | One process | No cross-process distribution or persistent history |
| Unix socket | Multiple processes on one host | Host-local coordination; not a multi-node durable bus |
| Redis Streams | Distributed | Persistent stream with consumer-group semantics; adapter was 0.4.0 |
| Valkey Streams | Distributed | Redis-compatible stream posture; adapter was 0.5.0 |
| Google Cloud Pub/Sub | Distributed managed service | Durable broker semantics; adapter was 1.1.2 |
| Caching PubSub wrapper | Depends on backend/cache | Adds replay cache but inherits cache durability and scope |

Consumer groups distribute work among consumers. They do not broadcast one copy
to every member. Design group names and fan-out deliberately.

## Resumable streams require event history

Persistent workflow state and replayable stream events are separate:

- storage can know a run is suspended while the UI cannot replay missed tokens;
- PubSub can replay events while the business result was never committed;
- an in-memory cache can resume only while the same process survives;
- a distributed bus without a retained cache may deliver new events but not the
  requested history window.

For cross-replica or restart-safe replay, use a distributed backend and durable
event cache with explicit TTL/size. Treat text-token replay as a product choice:
often it is cheaper and safer to reconcile the final message than retain every
token indefinitely.

## Durable-agent observation

Durable-agent APIs can observe a run by ID and replay cached events. Cleanup can
remove run registry/cache entries. Do not clean up a suspended run that must
resume later.

GitHub issue
[#17998](https://github.com/mastra-ai/mastra/issues/17998) remained open at the
research snapshot and requested a thin storage-backed index from thread to
active durable-agent run. Suspended-run listing exists, but applications should
own their product-operation-to-run mapping for ordinary active runs rather than
scan or parse snapshots.

## Redaction

Mastra's HTTP stream layer redacts some sensitive data by default. Keep it as
defense in depth, not the primary boundary:

- transform tool display/transcript payloads before streaming;
- redact at trace/log creation;
- avoid writing sensitive custom events;
- apply field-level allowlists in the product gateway;
- test direct, adapter, and durable stream paths.

A raw tool result can leak through a newly added chunk type if the client only
filters known text fields.

## Client reconciliation algorithm

On reconnect:

1. authenticate again;
2. load the application operation record;
3. reconnect/replay from the last event ID when supported;
4. deduplicate already applied event IDs;
5. if replay is unavailable, query run status through public APIs;
6. fetch the canonical final message/result from storage;
7. render pending approval only from verified current state;
8. mark ambiguous state and offer safe retry/recovery, never blindly start a
   duplicate run.

Persist a completed product result before declaring the UI complete. The
client's rendered transcript is not the source of truth.

## Backpressure

Long streams consume sockets, memory, event history, and proxy capacity.
Control:

- maximum concurrent streams per user and tenant;
- maximum run duration and idle duration;
- maximum event size and retained history;
- slow-consumer buffer limits;
- downstream tool and model concurrency;
- admission rejection or queueing;
- heartbeat frequency and proxy timeouts.

Do not keep a managed platform deployment awake accidentally with unbounded
idle sockets or exporters.

## Testing matrix

- disconnect before run ID delivery;
- disconnect during text and during a tool result;
- reconnect after cache expiry;
- two browser tabs consume the same run;
- duplicate and out-of-order event delivery;
- process loss with in-memory versus distributed PubSub;
- approval event replay after the policy expires;
- cleanup called while suspended;
- server emits terminal state but the client misses it;
- tenant A guesses tenant B's run ID.

## Checklist

- [ ] Structured streams are used for nontrivial products.
- [ ] Every event has idempotent client application.
- [ ] Run IDs and last offsets are persisted early.
- [ ] Disconnect is reconciled, not treated as failure.
- [ ] PubSub topology and replay cache match replica topology.
- [ ] Product operation-to-run mapping is application-owned.
- [ ] Stream redaction is layered and tested.
- [ ] Slow consumers, history, and concurrent streams are bounded.
- [ ] Suspended runs are not prematurely cleaned up.

## Primary sources

- [Streaming documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/guides/streaming.mdx)
- [PubSub source packages](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/pubsub)
- [Core stream implementation](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/core/src/stream)
- [Active durable-run index request](https://github.com/mastra-ai/mastra/issues/17998)
- [Client SDK source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/client-sdks)
- [Canonical agent state and event contracts](../../runtime/agent-state-and-event-contracts.md)

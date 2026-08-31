# Memory, Resources, and Admission Control

> **Last researched:** 2026-08-31
> **Applies to:** .NET 10 LTS, C# 14

Agent load is heterogeneous. One request may hold a tiny prompt; another retains documents, a long streaming response, several tool processes, and parallel provider calls. Request-count limits alone cannot protect the process.

## Agent memory is not one store

Use "memory" only with a qualifier. These layers differ in authority, lifetime, retrieval, and deletion requirements.

| Layer | Purpose | Lifetime and storage | Production rule |
|---|---|---|---|
| Working memory | The exact context sent for one model invocation | Ephemeral, bounded, derived | Rebuild from trusted sources; never make it the only copy of state/effects |
| Session memory | Conversation/run continuity across turns | Session-scoped transcript, compacted view, or provider conversation handle | Tenant-bound, versioned, bounded, and independently deletable |
| Durable operational state | Run status, approvals, leases, events, effect intents/receipts | Transactional state/event store | Authoritative; deterministic lookup, not similarity retrieval |
| Long-term semantic memory | Extracted facts, preferences, prior episodes across sessions | Governed store plus index | Optional and non-authoritative; preserve provenance, consent, correction, expiry, and confidence |
| Knowledge/RAG | External documents used to answer a task | Document/blob store and retrieval index | Not user memory; enforce source ACLs again at retrieval time |

A vector database is an index, not an authorization boundary or source of truth. Retrieval can omit a record, return a stale near-neighbor, or surface prompt injection. Fetch the authoritative object by tenant-scoped identifier after retrieval and validate its current policy before placing it in context.

Keep opaque provider conversation/response IDs in trusted server-side storage and map them from the application's own tenant-scoped session ID. Do not let a client resume an arbitrary provider ID without verifying ownership.

Long-term memory should be earned by a product requirement. Define which facts may be extracted, who can inspect/correct/delete them, retention and regional constraints, conflict resolution, and how stale facts stop influencing decisions. Do not persist internal reasoning, secrets, raw tool output, or every conversation by default.

## Context construction and compaction

The context window is a cache of selected evidence for the next invocation. Compute its usable budget before assembly:

~~~text
usable input = model context limit
             - reserved output/reasoning budget
             - fixed instructions and tool schemas
             - safety margin for tokenizer/provider variance
~~~

Token counts are model-specific. Estimate before the request, record actual provider usage after it, and keep a byte/item cap as a provider-independent safety bound.

Build context in a fixed order:

1. load authoritative run state, active policy/prompt version, and unresolved approvals/effects;
2. add the current user turn and recent complete conversation units;
3. retrieve session/semantic memory and knowledge under the caller's current authorization;
4. externalize large tool results and documents as immutable, integrity-checked references or bounded excerpts;
5. count the assembled context, compact the lowest-priority eligible material, then count again;
6. fail with a controlled budget result if mandatory material still cannot fit.

Never split a tool call from its result. Preserve system/developer instructions, current goal, accepted decisions, approval scope, effect receipts and unknown outcomes, active identifiers/versions, user corrections, and the most recent user turn. Compaction must not convert an unresolved ambiguity into a fact or a refusal into success.

### A compacted view needs a contract

Store a compaction record with its schema/prompt version, covered source sequence range, source integrity hash, created time, model/algorithm identity, token/byte counts, and validation result. Prefer structured fields for decisions, open questions, effects, and referenced artifacts rather than one prose blob. It remains a derived view: if it is missing, stale, corrupt, or unauthorized, rebuild or fail safely from the authoritative session/state records.

Evaluate compaction with continuation tests: after several compressions, the agent must preserve user corrections, tool-call/result pairing, open effects, authorization boundaries, and task outcome. Include adversarial content that asks the summarizer to erase policy or reinterpret an effect receipt.

Provider-managed compaction can reduce implementation work but does not replace durable state. OpenAI Responses supports context-management compaction and a compact endpoint whose compaction item is intentionally opaque; treat that item as provider-specific session material. Microsoft Semantic Kernel reducers expose truncate, token-based, and summarize strategies and explicitly preserve system messages and function-call/result structure. Whichever layer compacts, assign exactly one owner and version/test the behavior.

## Resource envelope

Define and enforce per-run maxima for:

- input, context, and output bytes;
- model tokens and monetary budget;
- stream events and queued bytes;
- tool calls, parallel branches, and turns;
- open HTTP requests and response streams;
- child processes, files, and disk bytes;
- retained state and durable-history bytes;
- total elapsed and CPU time.

Reject, defer, or degrade before allocating resources that cannot fit.

## Weighted admission

~~~mermaid
flowchart LR
    Request --> Estimate[Estimate weight]
    Estimate --> Tenant[Tenant quota]
    Tenant --> Global[Global memory/concurrency]
    Global --> Provider[Provider partition]
    Provider --> Admit{Capacity?}
    Admit -->|yes| Lease[Resource lease]
    Admit -->|no| Policy[Reject queue or degrade]
    Lease --> Run
    Run --> Release[Release in finally]
~~~

Use <code>System.Threading.RateLimiting</code> for concurrency, token-bucket, fixed/sliding-window, or partitioned controls where its semantics fit. Keep queues small. A limiter with a large wait queue merely moves overload into memory and latency.

Admission dimensions commonly need separate partitions:

- tenant or user fairness;
- provider/deployment rate and concurrency;
- tool/sandbox pool;
- memory-heavy versus lightweight run;
- interactive versus batch priority.

Acquire resources in a consistent order to avoid deadlocks. Release them in <code>finally</code>. Do not hold a scarce provider permit while waiting for human approval or an unrelated tool.

## Managed memory realities

Objects of about 85,000 bytes or larger generally enter the large object heap. Large UTF-16 strings, byte arrays, JSON documents, embeddings, and contiguous transcript buffers can create allocation spikes and longer collections.

Prefer:

- streaming parsers and serializers;
- bounded pooled buffers with clear ownership;
- immutable references to large blobs;
- chunking based on byte/token limits;
- incremental hashing and compression;
- prompt assembly that avoids repeated concatenation.

Pooling is not free memory. An unbounded pool pins the high-water mark. Clear sensitive buffers before returning them when required, and never return a buffer while another async operation can still access it.

Do not call <code>GC.Collect</code> as routine pressure control. Fix retention, bounds, and allocation shape first.

## Thread pool and blocking

Thread-pool starvation often appears as growing worker-thread count, low CPU, increasing queue length, and high latency. Common causes are synchronous waits on async I/O, blocking locks, synchronous provider/HTTP calls, or process stream reads.

.NET has improved starvation response, but compensation threads do not fix sync-over-async. Search for <code>.Result</code>, <code>.Wait()</code>, <code>GetAwaiter().GetResult()</code>, long synchronous callbacks, and blocking continuations.

## Container memory

The GC sees configured container limits, but managed heap is only part of RSS. Account for:

- native runtime and TLS buffers;
- memory-mapped files;
- provider/native libraries;
- child processes;
- sidecars;
- page cache and temporary files.

Set limits with headroom and measure under the published image. Server GC, heap hard limits, CPU quotas, and architecture can materially change behavior. Never size admission from workstation measurements alone.

## Overload behavior

Fail fast with a stable overload result when interactive work cannot be admitted. Durable batch work can remain in an external queue with bounded retention and expiry. Possible degradation includes disabling optional retrieval, reducing parallel branches, lowering output limits, or routing to a cheaper model, but only when product semantics allow it and telemetry makes the choice visible.

## Failure patterns

- Treating the current prompt or provider conversation handle as authoritative run state.
- Writing every transcript to cross-session memory without consent, provenance, expiry, or deletion.
- Retrieving a vector match across tenants or using its text without rechecking source authorization.
- Compaction drops a tool result, approval condition, user correction, or unknown effect.
- Unbounded channel protected by a concurrency semaphore.
- Capacity set by item count while items vary by megabytes.
- Thousands of requests wait inside the process for a limiter.
- A run acquires multiple partition permits in inconsistent order.
- Full transcripts and tool results are retained in every span/log.
- Buffer pool grows without a maximum retained size.
- Managed heap looks healthy while child processes exhaust the container.

## Review checklist

- [ ] Working, session, durable operational, long-term, and knowledge stores have separate contracts.
- [ ] Context assembly reserves output/tool-schema headroom and fails safely when mandatory content cannot fit.
- [ ] Compaction preserves tool pairs, policy, decisions, approvals, effects, identifiers, and user corrections.
- [ ] Long-term memories carry tenant, provenance, consent/retention, confidence, and correction/deletion paths.
- [ ] Retrieval rechecks current authorization against the authoritative source.
- [ ] Admission is weighted by meaningful resource dimensions.
- [ ] In-process wait queues are small and bounded.
- [ ] Tenant fairness and provider limits are partitioned.
- [ ] Large payloads stream or use external blob references.
- [ ] RSS, managed heap, thread pool, sockets, processes, and disk are observed.
- [ ] Load tests verify a stable plateau and controlled overload.
- [ ] Published-container limits include non-managed memory headroom.

## Primary sources

- [.NET garbage collection fundamentals](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/)
- [Large object heap](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/large-object-heap)
- [GC runtime configuration](https://learn.microsoft.com/en-us/dotnet/core/runtime-config/garbage-collector)
- [Debug thread-pool starvation](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/debug-threadpool-starvation)
- [.NET runtime metrics](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/built-in-metrics-runtime)
- [Rate-limit an HTTP handler](https://learn.microsoft.com/en-us/dotnet/core/extensions/http-ratelimiter)
- [.NET channels](https://learn.microsoft.com/en-us/dotnet/core/extensions/channels)
- [OpenAI Responses compaction](https://developers.openai.com/api/docs/guides/compaction)
- [OpenAI Responses context management](https://developers.openai.com/api/reference/resources/responses/methods/create)
- [Semantic Kernel chat-history reduction](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/chat-completion/chat-history)
- [Agent Framework conversation storage](https://learn.microsoft.com/en-us/agent-framework/concepts/agents/conversations/storage)
- [Agent Framework context providers](https://learn.microsoft.com/en-us/agent-framework/integrations/by-component/context-providers/)

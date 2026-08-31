# Memory, Context, and Storage

Mastra memory decides which previous information enters a model call. It should
improve continuity, not become the source of truth for permissions, money,
orders, or other transactional state.

## Context layers

| Layer | Scope and purpose | Production caution |
|---|---|---|
| Incoming messages | Current user turn | Validate size, type, and ownership |
| Message history | Recent raw conversation, ten messages by default | Grows without retention; send only the new turn when memory is enabled |
| Working memory | Small persistent facts or task state | Model-editable unless read-only; not authoritative |
| Semantic recall | Relevant older messages via embeddings/vector search | Enforce resource/tenant filters and embedding lifecycle |
| Observational memory | Compressed observations over long conversations | Adds model calls, latency, and cost; resource scope is experimental |
| Request context | Ephemeral verified request data | Not memory and not automatically trusted |
| System of record | Application database or external service | Must remain authoritative for business state |

## Thread and resource identity

A **thread** is one conversation. A **resource** is the owner or entity across
threads, usually a user, account, or project. A thread's resource owner is
immutable after creation.

Derive both IDs on the server. The product backend should:

1. authenticate the caller;
2. resolve the tenant and resource from authoritative membership;
3. verify the requested thread belongs to that resource;
4. invoke the agent with the verified identifiers;
5. reject attempts to override ownership from client payloads.

When memory is enabled, send only the new user message. Re-sending the full UI
history can duplicate records, distort timestamp ordering, and bypass the
framework's intended retrieval.

## Message history

Message history is the simplest baseline and is often sufficient. Choose a
window that fits the model budget and task. A larger window is not automatically
better: old instructions, stale facts, and untrusted tool output can crowd out
the current request.

Keep the durable record separate from the prompt view. Retention may remove old
messages from operational storage, while legal or product requirements may
require a differently governed audit record. Do not assume model memory is an
audit trail.

## Working memory

Working memory is a compact document made visible on each relevant turn.
Resource scope is the default; thread scope isolates it to one conversation.

Two update modes matter:

- Markdown working memory is effectively replaced as a document.
- Schema-backed object working memory uses structured updates: objects merge,
  null can remove values, and arrays replace rather than append.

Good uses:

- user communication preference;
- current task goal and constraints;
- a non-authoritative summary of an open case;
- pointers to records in a system of record.

Bad uses:

- access-control roles;
- a current balance or inventory count;
- approval status;
- unencrypted secrets;
- a large document or artifact.

Use read-only working memory when the application, rather than the model, owns
the facts. Working-memory state signals were experimental at the research
snapshot; isolate them behind an application interface.

## Semantic recall

Semantic recall embeds messages and retrieves relevant matches, optionally with
surrounding context. It is useful when exact recent history is insufficient but
full history is too large.

Production requirements:

- include resource and tenant filters in every vector query;
- use separate namespaces or verified metadata filters;
- record embedding model and dimension;
- reindex deliberately when embedding models change;
- bound match count and context window;
- delete vectors when source records are deleted;
- test adversarial near-neighbor content across tenants;
- treat retrieved text as untrusted input.

Semantic similarity is not entitlement. A close vector match must never expand
what the caller may read.

## Observational memory

Observational memory (OM) summarizes older messages into dense observations,
while newer messages remain raw. The original stored messages are retained; the
observation changes the prompt representation rather than replacing the record.
An observer and optional reflection process create additional model calls.

Mastra recommends OM for long conversations and presents it as a possible
replacement for combinations of history, working memory, and semantic recall.
Treat that as a framework recommendation to evaluate, not a universal result.
Measure recall accuracy, omission rate, latency, token use, and total model cost
on the application's conversations.

Important maturity details:

- thread scope is the conservative default;
- resource-scoped OM was experimental as of 2026-08-31;
- asynchronous buffering is available for thread scope but disabled for
  resource scope;
- observation and reflection calls introduce background failure modes;
- summaries can preserve false or malicious statements unless provenance is
  retained.

Use OM when a conversation genuinely exceeds a practical raw-message window,
not merely because it is available.

## Subagent memory

Delegation has its own isolation semantics. A subagent delegation receives a
fresh thread and a deterministic resource based on the parent resource and
subagent name. If the subagent has no explicit memory configuration, it may
inherit the parent's. Parent context can be supplied to the subagent, while the
delegation prompt and response are the records commonly persisted for that
delegation.

Review:

- the <code>messageFilter</code> that selects parent messages;
- whether the subagent should have any durable memory;
- whether resource-scoped facts may cross separate delegations;
- whether sensitive tool results are forwarded;
- whether deletion and retention cover child resources.

Isolation by generated identifier is not authorization. The subagent still
needs a trusted actor and tool-level policy.

## Storage domains

Mastra storage is modular. Current domains include memory, workflow snapshots,
observability, scores, datasets, experiments, schedules, background tasks,
thread state, and newer editor/definition domains. Not every adapter implements
all of them.

Recommended general production posture:

- use Postgres for broadly transactional memory/workflow needs;
- use a vector-capable store that can enforce tenant metadata for recall;
- use ClickHouse for high-volume observability when query/write volume
  justifies a separate system;
- use local LibSQL or DuckDB for development and bounded single-node cases;
- use a composite store only when domain-specific operational benefits outweigh
  extra lifecycle complexity.

The default in-memory path is for tests and disposable development, not
multi-process durability.

## Retention is opt-in

Storage can grow without bound unless the application configures retention and
invokes pruning. Retention support varies by adapter. The prune operation can
batch, resume, and cancel, but it is not an internal scheduler and does not
necessarily reclaim database disk space.

Design retention per domain:

| Domain | Retention constraint |
|---|---|
| Messages | Product/legal policy and active-conversation needs |
| Semantic vectors | Delete with the source message |
| Working/observational memory | Ownership deletion and stale-summary policy |
| Workflow snapshots | Never delete records needed by active, waiting, or suspended runs |
| Traces/logs/scores | Privacy, incident, and evaluation windows |
| Datasets/experiments | Reproducibility and release evidence |

Workflow snapshot age can be based on last update; an old suspended run may
still be live. Query status before pruning.

## Composite-store lifecycle

A composite store can route each domain to a different adapter. On shutdown,
the composite closes configured default/editor/domain clients once. A store
constructed outside that graph or used only indirectly may still require an
explicit close.

Test startup and shutdown with connection counts. Call <code>mastra.shutdown()</code>
after admission is stopped and active work has drained.

## Artifact pattern

Keep large inputs, model files, images, reports, and tool payloads out of memory
and workflow snapshots. Store them in an object or document system and persist a
small reference:

~~~json
{
  "artifactId": "art_01J...",
  "version": 3,
  "sha256": "...",
  "mediaType": "application/pdf",
  "tenantId": "tenant_42"
}
~~~

Resolve the reference only after authorization and integrity validation.

## Isolation test matrix

Run the same tests concurrently, not only sequentially:

- tenant A and tenant B use identical thread names;
- one resource has multiple threads;
- semantic queries use deliberately similar text across tenants;
- response cache and observational memory are enabled;
- a subagent delegates while another tenant updates memory;
- a thread is deleted while a background observer is running;
- a process restarts between message persist and response completion.

## Checklist

- [ ] Thread and resource IDs are derived server-side.
- [ ] Memory receives only the new turn from the client.
- [ ] Business truth stays outside model-editable memory.
- [ ] Vector queries enforce tenant/resource filters.
- [ ] OM is workload-evaluated and experimental scopes are isolated.
- [ ] Subagent memory and message forwarding are explicit.
- [ ] Every used storage domain is verified on the selected adapter.
- [ ] Retention is scheduled and excludes resumable workflow state.
- [ ] Large artifacts are stored by reference.

## Primary sources

- [Memory documentation source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/memory)
- [Storage documentation source](https://github.com/mastra-ai/mastra/blob/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/docs/src/content/en/docs/storage.mdx)
- [Memory package source](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/packages/memory)
- [Storage adapters](https://github.com/mastra-ai/mastra/tree/8c88706dc2dc4e4a01d78abe358b6c8cab14d2ad/stores)
- [Mastra memory layers field explanation](https://mastra.ai/blog/agent-memory-layers)
- [Canonical memory architecture guide](../../context-memory/memory-architecture.md)

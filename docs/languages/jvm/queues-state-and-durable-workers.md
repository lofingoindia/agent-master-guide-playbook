# Queues, State, and Durable Workers

## Choose durability by run shape

| Need | Smallest fitting mechanism |
|---|---|
| short request, no effects, safe client retry | synchronous service |
| background job with idempotent steps | queue plus transactional state |
| hours/days, timers, signals, human approval, recovery | durable workflow/runtime |
| keyed entity with serialized state and calls | durable object/virtual object |

A queue is not a workflow engine. It redelivers messages; application code must model state, timers, signals, idempotency, and reconciliation.

## Queue worker transaction

~~~mermaid
sequenceDiagram
    participant Q as Queue
    participant W as Worker
    participant D as Database
    participant E as External effect
    Q->>W: delivery(run, expected version)
    W->>D: claim lease / load state
    W->>D: persist effect planned
    W->>E: execute(effect id)
    E-->>W: result
    W->>D: persist result and next state
    W->>Q: acknowledge
~~~

Persist the transition before acknowledging. If the worker dies after the external effect but before persistence, the next delivery sees a planned/started effect and reconciles by stable ID rather than executing blindly.

## State rules

- Use optimistic concurrency/version columns per run.
- Store small immutable events or snapshots; keep large artifacts in content-addressed object storage.
- Separate conversation history, agent working state, and audit history.
- Bound prompts assembled from state; persistence is not permission to resend everything.
- Encrypt and apply tenant-aware access control.
- Define retention, deletion, and legal-hold behavior for prompts and tool data.

One logical writer per run simplifies ordering. Parallel workers can process different runs; parallel branches inside one run should rejoin through versioned state.

## Context, compaction, and memory are different stores

Do not call every persisted value “memory.” Give each class a purpose, owner, retention rule, and prompt-admission policy:

| Class | Canonical contents | Lifetime | Enters a prompt when |
|---|---|---|---|
| working context | current goal, constraints, recent normalized events, unresolved effects | one run/continuation | required for the next decision |
| durable run state | status, versions, budgets, approvals, effect records, checkpoints | run retention/audit policy | only a bounded projection is needed |
| artifacts | documents, tool output, generated files, evidence | content-addressed retention | retrieved by digest/reference |
| user/domain memory | validated preferences or domain facts with provenance | explicit product retention | retrieval and authorization policy selects it |
| audit history | immutable decisions, identities, policy/effect evidence | compliance policy | normally never; operators query it |

Add long-term semantic/vector memory only when an evaluation shows that retrieval improves the workload beyond deterministic state plus artifact search. It introduces deletion, tenant isolation, poisoning, freshness, provenance, ranking, and cost obligations. Chat history is not a trustworthy substitute for state, and an embedding match is not authorization.

Compaction is a lossy derived artifact, not a rewrite of history. Store at least:

~~~java
record ContextCheckpoint(
    String runId,
    long throughSequence,
    int compactionSchemaVersion,
    String compactorModelAndPromptVersion,
    List<String> activeConstraints,
    List<String> completedEffectIds,
    List<String> unresolvedEffectIds,
    List<String> artifactDigests,
    String nextGoal,
    String sourceRangeDigest
) {}
~~~

Never let a free-form summary be the only record of tool effects, approvals, remaining budget, or terminal state. On resume, verify the checkpoint's source range and rebuild from canonical events when the schema/prompt changes or the checkpoint is suspect. Keep quoted/untrusted tool content separate from instructions. See the canonical [context engineering](../../context-memory/context-engineering.md), [memory architecture](../../context-memory/memory-architecture.md), and [compaction and continuity](../../context-memory/compaction-and-continuity.md) guides.

Provider-native compaction is another adapter artifact, not an exception to this rule. For example, the official OpenAI Java Responses reference exposes a compaction operation whose output includes an opaque compaction item and usage accounting. Store that provider continuation with provider/model/API version and integrity metadata, but keep application effects, approvals, budgets, artifact IDs, and terminal state in the application-owned event/checkpoint contract. The run must remain diagnosable if the provider continuation is unreadable or cannot be replayed through a different provider.

## Temporal Java

Temporal workflows are deterministic programs replayed from history. Workflow code must not perform arbitrary I/O, use native threads/executors, system time, random values, or ordinary locks. Put model calls, tool calls, MCP calls, and database/network I/O in Activities. Record non-deterministic decisions through workflow APIs.

Agent implications:

- keep full streamed tokens outside workflow history; store a reference or bounded final result;
- model/tool retry policy belongs on Activities and must respect run/effect idempotency;
- use signals/updates for approval and cancellation;
- version workflow code before changing replay behavior;
- worker shutdown must drain activities; work ignoring interrupts can block termination.

The Temporal Spring AI integration is public preview and does not support streaming. Treat it as an adapter, not a reason to place model semantics inside workflow history.

## Restate Java/Kotlin

Restate provides services, workflows, virtual objects, durable steps, state, timers, external events, and durable futures. Its Java/Kotlin SDK supports different serialization defaults—Jackson for Java and kotlinx.serialization for Kotlin—so shared contracts require explicit tests.

Durable execution replays around recorded steps. Keep nondeterministic effects inside durable steps and make external effects idempotent. Virtual Objects are useful for serialized per-key state such as a run or tenant quota. Validate invocation identity and configure serving/runtime security; durability does not imply authorization.

## Dapr and other workflow layers

Dapr's Java workflow SDK can fit environments already standardized on Dapr. Evaluate determinism rules, activity retry behavior, versioning, operational dependencies, local-test fidelity, and payload/history limits rather than selecting only by API style.

## Backpressure and leases

Worker poll concurrency, active run count, provider permits, and tool permits are different controls. A large queue backlog should not create an equal number of in-memory run objects. Pull only capacity you can execute. Leases require fencing tokens so a paused or slow old owner cannot commit after takeover.

A database-backed claimant should advance a monotonically increasing fence in the same atomic update that grants ownership. The exact SQL is database-specific, but the contract is:

~~~sql
UPDATE agent_run
SET lease_owner = :worker,
    lease_expires_at = :expires,
    lease_fence = lease_fence + 1
WHERE run_id = :run_id
  AND run_version = :expected_version
  AND (lease_owner IS NULL OR lease_expires_at < :database_now)
RETURNING run_version, lease_fence;
~~~

Every later state/effect commit includes both returned values in its `WHERE` clause. A late worker with an old fence updates zero rows and must stop. Use the database/server clock for lease expiry; timestamps alone do not fence a previous owner.

### Reconciliation loop

For each `STARTED` or `UNKNOWN` effect whose owner disappeared:

1. claim the run with a new fence;
2. query the remote system by idempotency key or operation ID;
3. if committed, persist the normalized result without re-executing;
4. if definitively absent and the contract permits retry, start a new attempt under the same logical effect ID;
5. if still unknowable, retain `UNKNOWN`, block dependent effects, and route to compensation or human review;
6. acknowledge/redrive the queue message only after the fenced transition commits.

Reconciliation must have its own rate limit, alert age, and dead-letter policy. Otherwise a provider outage can turn recovery into an unbounded retry storm.

## Checklist

- [ ] Delivery is assumed at least once.
- [ ] State transition persists before acknowledgement.
- [ ] Effects are planned and keyed before execution.
- [ ] Leases use fencing/versions, not timestamps alone.
- [ ] Large payloads are referenced, not copied through history.
- [ ] Context checkpoints retain source range, compactor version, unresolved effects, and budgets by reference.
- [ ] Long-term memory exists only with provenance, deletion, tenant isolation, and an eval-backed need.
- [ ] Workflow code is deterministic and effects live in activities/steps.
- [ ] Streaming/history growth is bounded.
- [ ] Worker polling aligns with downstream capacity.
- [ ] Approval, signals, timers, and cancellation survive process loss.

## Sources

- [Temporal Java workflow constraints](https://www.javadoc.io/static/io.temporal/temporal-sdk/1.37.0/io/temporal/workflow/package-summary.html)
- [Temporal WorkerFactory lifecycle](https://www.javadoc.io/static/io.temporal/temporal-sdk/1.37.0/io/temporal/worker/WorkerFactory.html)
- [Temporal Spring AI integration](https://github.com/temporalio/sdk-java/blob/main/contrib/temporal-spring-ai/README.md)
- [Restate Java services](https://docs.restate.dev/develop/java/services)
- [Restate durable steps](https://docs.restate.dev/develop/java/durable-steps)
- [Dapr Java workflow](https://docs.dapr.io/developing-applications/sdks/java/java-workflow/java-workflow-howto/)
- [OpenAI Java Responses compaction reference](https://developers.openai.com/api/reference/java/resources/responses/methods/compact)

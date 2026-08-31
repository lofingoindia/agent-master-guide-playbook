# Errors, Retries, and Idempotency

## Failure taxonomy before retry policy

| Class | Examples | Default action |
|---|---|---|
| cancellation/deadline | user cancel, parent timeout | stop; reconcile active effects |
| permanent request | auth, unsupported model, invalid schema | fail without retry |
| transient transport | connect reset, selected 5xx | bounded retry |
| throttling | 429 with Retry-After | respect server delay and budget |
| malformed model result | invalid structured output | bounded repair, not blind transport retry |
| domain rejection | policy denied, insufficient funds | terminal or human action |
| ambiguous effect | response lost after send | query/reconcile; do not assume failure |
| invariant/data corruption | impossible transition, hash mismatch | quarantine and alert |

Map SDK-specific exceptions into this small internal taxonomy at the adapter boundary. Preserve provider status, request ID, retry hint, and cause without leaking secrets.

## Retry the smallest safe operation

Retrying the entire agent run can repeat successful model calls and tools. Retry only the failed provider request or effect attempt when:

- the operation is idempotent, or has a provider-enforced idempotency key;
- no non-idempotent downstream consequence was already committed;
- the remaining deadline covers backoff plus attempt;
- the retry budget is not exhausted.

Use exponential backoff with jitter and a cap. Respect <code>Retry-After</code> when valid, but never sleep beyond the run deadline. Bound total attempts and elapsed retry time. SDK default retries count toward the same budget—disable or account for nested retry layers to avoid multiplication.

## Idempotency records

For each logical effect use a stable key such as <code>run-id/step-id/tool-version</code>. Store:

| Field | Why |
|---|---|
| canonical request digest | detect key reuse with different input |
| state and owner lease | coordinate concurrent deliveries |
| attempt history | operations and audit |
| remote request/operation ID | reconcile ambiguity |
| result/error digest | replay same outcome |
| timestamps/version | expiry and optimistic concurrency |

The first executor atomically claims or creates the record. Later deliveries return the recorded result, wait for the owner, or take over an expired lease according to policy. A database unique constraint is stronger than an in-memory map.

Use explicit transitions rather than “retry count plus nullable result”:

~~~mermaid
stateDiagram-v2
    [*] --> Planned
    Planned --> Started: fenced owner dispatches
    Started --> Succeeded: terminal result recorded
    Started --> Rejected: terminal domain rejection
    Started --> Failed: definitive non-commit failure
    Started --> Unknown: timeout, disconnect, or crash window
    Unknown --> Succeeded: reconciliation finds commit
    Unknown --> Failed: reconciliation proves absence/failure
    Unknown --> Started: safe retry under same effect ID
~~~

Only the current fenced owner can transition the record. Retrying from `UNKNOWN` increments the attempt but preserves effect ID and canonical request digest. A different digest under the same key is an invariant violation, not an update.

A practical key should be derived from logical identity, not serialized map order or a worker attempt:

~~~java
EffectId effectId = EffectId.of(runId, logicalStepId, toolName, toolVersion);
String requestDigest = sha256(canonicalJson(toolArguments));
effectStore.plan(effectId, requestDigest, policyDecisionId, expectedRunVersion);
~~~

Canonicalization rules are part of the contract and need golden tests across Java/Kotlin serializers. Never put secret arguments directly into an idempotency header or metric.

## Exactly-once is not an end-to-end property

Queues and workflow engines can make delivery or history durable, but an external side effect can still happen before its completion is recorded. Use one of:

- remote idempotency key with durable result lookup;
- transactional outbox/inbox when the effect and state share a database boundary;
- reservation/confirm protocol;
- reconciliation by external operation ID;
- compensating action with explicit business semantics.

Never label retries “exactly once” unless the entire effect boundary proves it.

## Circuit breakers and bulkheads

A circuit breaker prevents repeated calls to a dependency already failing. It is not a rate limiter or retry strategy. Evaluate state per dependency/tenant/model where useful, use a half-open probe budget, and expose the open state. Bulkheads protect capacity; apply them before starting expensive work.

Be cautious with provider fallbacks. A different model can change tool selection, schema adherence, cost, safety behavior, and output quality. Fallback is a product decision recorded in run history, not a transparent network retry.

## Cancellation and errors

Java interruption and Kotlin cancellation are control signals, not retryable I/O failures. Preserve them through broad exception handlers. A client disconnect may cancel user-facing streaming while durable work intentionally continues; that choice must be explicit at admission and visible to the client.

## Checklist

- [ ] Errors map into a documented internal taxonomy.
- [ ] Cancellation is never retried.
- [ ] Nested SDK/application retries share one budget.
- [ ] Backoff uses jitter, caps, Retry-After, and the absolute deadline.
- [ ] Effects have stable keys and request digests.
- [ ] Unknown outcomes trigger reconciliation.
- [ ] Concurrent duplicate deliveries are database-coordinated.
- [ ] Model fallback is recorded and policy-controlled.

## Sources

- [RFC 9110 HTTP semantics and idempotent methods](https://www.rfc-editor.org/rfc/rfc9110)
- [RFC 6585 and HTTP 429](https://www.rfc-editor.org/info/rfc6585/)
- [OpenAI Java SDK retries and errors](https://github.com/openai/openai-java)
- [Temporal Java workflow constraints](https://www.javadoc.io/static/io.temporal/temporal-sdk/1.37.0/io/temporal/workflow/package-summary.html)

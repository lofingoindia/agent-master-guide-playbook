# Reliability, Idempotency, and Reconciliation

## Reliability target

The useful guarantee is not “exactly once.” The target is:

- each intended watch interval has a visible terminal evaluation state;
- every observation and detector result is reproducible or explicitly indeterminate;
- each semantic external effect occurs no more than policy intended;
- ambiguous effects are reconciled before retry;
- revisions, cancellation, and late delivery remain visible;
- success is independently verified.

No message broker, workflow engine, or provider API creates a transaction across the state store, BI platform, ITSM system, and human organization.

## Failure domains

| Domain | Examples | Required treatment |
|---|---|---|
| Trigger | Duplicate schedule, delayed event, missed slot | Stable evaluation key, catch-up policy, deadline |
| Semantic query | Timeout, partial page, stale snapshot, schema change | Query identity, typed partial/unknown, compatibility gate |
| Data health | Late partition, failed assertion, lineage gap | Quarantine business detection; data-health case |
| Detector | Baseline unavailable, numeric error, nondeterministic dependency | Pinned artifact and version; fail closed |
| Model triage | Timeout, malformed output, hallucinated fact, repeated loop | Structured validation, bounded retry, deterministic fallback |
| State | Conflict, process crash, stale timer, snapshot corruption | Transaction, optimistic version, replay/checkpoint |
| Approval | Expired, changed evidence, revoked role | Revalidate immediately before effect |
| External effect | Timeout after commit, duplicate response, provider retry | Intent-first ledger, operation key, reconciliation |
| Human workflow | No acknowledgement, wrong owner, capacity overload | Escalation, fallback route, budget, manual takeover |
| Outcome | Delayed metric, action unobservable, confounded change | Explicit outcome deadline and indeterminate state |

## Transactional evaluation

A database transaction should claim the evaluation identity before work, but long queries and model calls stay outside the transaction.

~~~text
on evaluation_trigger(trigger):
    key = semantic_evaluation_key(trigger.tenant, trigger.watch_version, trigger.interval)

    transaction:
        existing = evaluations.find(key)
        if existing is terminal:
            return existing.summary
        if existing is leased and lease_is_live(existing):
            return ALREADY_IN_PROGRESS
        evaluation = evaluations.claim_or_resume(key, lease, expected_version)
        events.append(EvaluationClaimed(evaluation.id, evaluation.version))

    watch = load_exact_active_or_historical_watch(trigger.watch_version)
    authorize(watch.tenant, watch.purpose, watch.metric)
    semantic_result = query_with_operation_identity(watch, trigger.interval)
    observation = validate_freshness_quality_lineage(semantic_result, watch)

    transaction:
        recheck_lease_and_version(evaluation)
        observations.insert_immutable(observation)
        if observation.data_health != ACCEPTED:
            evaluations.finish(DATA_GATE_STOP, observation.id)
            enqueue_data_health_case_if_configured(observation)
            return

    signal = run_pinned_detector(observation, watch.detector, watch.baseline)

    transaction:
        recheck_lease_and_version(evaluation)
        signals.insert_immutable(signal)
        case = correlate_and_transition(signal, expected_case_version)
        evaluations.finish(COMPLETED, observation.id, signal.id, case.id?)
        enqueue_due_intents(case)
~~~

Implementation assumptions:

- inserts use unique constraints on semantic identities;
- leases prevent duplicate concurrent work but correctness does not depend on lease exclusivity;
- every resumed step reads durable state and validates exact versions;
- queue enqueue uses an outbox or an equivalent recoverable pattern;
- semantic query completion is reconciled by provider job/request identity after timeout;
- the detector has no external side effect.

## Effect protocol

~~~mermaid
stateDiagram-v2
    [*] --> Intended
    Intended --> Attempting: worker claims
    Attempting --> Confirmed: durable remote receipt
    Attempting --> Failed: definitive rejection
    Attempting --> Unknown: timeout or lost response
    Unknown --> Reconciling
    Reconciling --> Confirmed: remote object found and matches digest
    Reconciling --> Retryable: absence established
    Reconciling --> Indeterminate: lookup unavailable or conflicting
    Retryable --> Attempting: bounded retry
    Failed --> [*]
    Confirmed --> Verifying
    Verifying --> Verified: intended delivery or work state proven
    Verifying --> Remediate: late, wrong, or unauthorized effect
    Indeterminate --> ManualReview
    Remediate --> [*]
    Verified --> [*]
    ManualReview --> [*]
~~~

### Intent-first ordering

1. Validate current case, policy, and exact approval.
2. Derive stable operation key and payload digest.
3. Commit effect intent and outbox record.
4. Worker revalidates eligibility and obtains scoped credential.
5. Send the operation key through a provider-supported idempotency field, external ID, or deterministic searchable marker.
6. Record receipt atomically with effect state when a definitive response arrives.
7. On timeout, mark `unknown` and reconcile by operation key/remote ID.
8. Retry only when absence is established and retry policy permits it.
9. Verify the delivered object and continue follow-through.

An API's idempotency feature is helpful but not sufficient: retention windows expire, endpoints differ, and payload conflicts must be handled. Keep the local ledger.

## Retry ownership

Assign exactly one retry owner per boundary:

| Boundary | Retry owner | Notes |
|---|---|---|
| Queue delivery to worker | Queue/runtime | Handler remains idempotent |
| Semantic job submission | Adapter | Controller sees complete/partial/unknown only |
| Quality/lineage read | Adapter for transport; controller for policy re-evaluation | Respect evidence freshness |
| Model call | Model gateway | Retry only safe transient failures; preserve call identity |
| Notification/ITSM write | Effect worker plus reconciler | Never stack SDK, proxy, queue, and controller retries blindly |
| Outcome read | Outcome scheduler | Stops at deadline or typed indeterminate |

Use exponential backoff with jitter, attempt and elapsed-time caps, and deadline awareness. Do not retry authorization failures, schema errors, policy denials, or incompatible semantic versions.

## Duplicate and ordering controls

- Unique evaluation keys suppress duplicate schedules.
- Domain events carry aggregate version and causation ID.
- Consumers store processed event IDs or use idempotent state transitions.
- Timers re-read current state; they do not carry authoritative snapshots.
- Out-of-order observation revisions use explicit revision numbers and supersession.
- An old signal cannot transition a case built on a newer observation.
- A recovery signal references the open case and detector state it recovers.
- Notification ordering is best effort; the case UI remains authoritative.

## Revisions and backfills

Backfills are expected operational events, not edge cases.

~~~mermaid
sequenceDiagram
    participant D as Data platform
    participant C as Controller
    participant O as Observation store
    participant K as Case
    participant H as Operator
    D->>C: Interval corrected
    C->>O: Create revision 1 and supersede revision 0
    C->>C: Re-run pinned or explicitly migrated detector
    alt Material conclusion unchanged
        C->>K: Attach revision evidence
    else Trigger no longer valid
        C->>K: Propose retraction
        C->>H: Explain prior effect and correction
    else Severity or route changed
        C->>K: New case version and invalidate stale approval
        C->>H: Request updated disposition
    end
~~~

Policy must specify:

- correction window and maximum revisions;
- whether to use the original or current semantic/detector version;
- which changes reopen or retract a case;
- how prior recipients are corrected;
- whether historical outcomes are relabeled;
- how baseline training excludes corrected incidents.

Never rewrite notification history or pretend a prior decision was made with later data.

## Cancellation, pause, and kill

Three controls differ:

- **Suspend watch:** prevents new evaluations; does not cancel in-flight effects.
- **Cancel case/effect:** requests a legal state transition and best-effort remote cancellation.
- **Platform kill switch:** denies new effect authorization across a scoped tenant, route, tool, model, or release.

Cancellation protocol:

1. Commit cancellation command and reason.
2. Stop unstarted local work.
3. Reconcile every `attempting` or `unknown` effect.
4. Attempt remote cancel/retract only if supported and authorized.
5. Record any late delivery and notify the accountable operator if material.
6. Verify no further escalation timers remain eligible.

Deleting queue messages or disabling a rule is not proof that an already accepted provider action stopped.

## Data-health outage behavior

When a shared source or semantic layer fails, thousands of watches can generate duplicate data-health alerts.

Controls:

- correlate by data product, semantic snapshot, incident, and affected interval;
- inhibit child business alerts while retaining skipped evaluation records;
- route one root incident to DataOps;
- expose affected watch count and criticality;
- resume in controlled batches after recovery;
- replay missed intervals without delivering historical noise unless policy says otherwise;
- verify baseline contamination and semantic consistency before reactivation.

## Backpressure and degraded modes

Admission priority should consider:

1. safety/regulatory and customer-impacting watches;
2. freshness deadline and remaining decision window;
3. unresolved effects and acknowledgement escalations;
4. tenant fair share and owner/channel capacity;
5. estimated query/model cost.

Degraded modes:

| Pressure | Preserve | Defer or disable |
|---|---|---|
| Query capacity | Critical current observations | Low-priority backfills and optional drill-downs |
| Model capacity | Deterministic detection/routing | Model triage; use factual template |
| Notification outage | Case state and effect intents | Delivery; reconcile when provider recovers |
| ITSM outage | High-priority alternate route if approved | Ticket retries until absence known |
| Owner overload | Critical grouped cases | Low-materiality notifications/digests |
| State-store stress | Commands, effects, timers, checkpoints | Analytics exports and noncritical enrichments |

Load shedding must create a visible skipped/deferred record. Never silently miss an interval.

## Reconciliation runbooks

### Unknown notification or work item

1. Freeze automatic retry for the operation key.
2. Query provider by idempotency key, external reference, and bounded time/target scope.
3. If exactly one matching object exists and payload digest matches, confirm it.
4. If none exists and the provider's consistency window elapsed, mark retryable.
5. If multiple or conflicting objects exist, mark indeterminate and assign manual review.
6. Link or close duplicates under the remote system's rules; never delete audit evidence.
7. Record the reconciliation method and confidence.

### Stuck acknowledgement

1. Validate route resolution and recipient rights at the original effect time.
2. Confirm delivery or determine unknown outcome.
3. Re-read current owner directory and fallback policy.
4. Escalate with a new operation key if the timer and case version remain eligible.
5. If owner capacity is exceeded, invoke the overload runbook, not repeated paging.
6. End only with acknowledgement, explicit unowned escalation, expiry, or incident command.

### Semantic drift incident

1. Activate the scoped watch/effect kill switch.
2. Suspend affected watch versions and preserve skipped intervals.
3. Identify semantic snapshots, observations, signals, cases, approvals, and effects in scope.
4. Reconcile in-flight delivery.
5. Compare old/new semantics on history and current intervals.
6. Issue corrected observations and visible case revisions/retractions.
7. Shadow the repaired version before owner-approved promotion.

## Failure matrix

| Injected failure | Safety invariant | Expected state |
|---|---|---|
| Crash after observation insert | One immutable observation per identity | Resume finds observation and completes detector |
| Crash after effect intent | No lost intent | Outbox/worker resumes |
| Remote commit then timeout | No blind duplicate | `unknown` then reconcile |
| Duplicate queue delivery | One semantic transition | Existing version/effect returned |
| Approval expires while queued | No stale authorized effect | Effect denied before credential issuance |
| Watch suspended during query | No new alert under suspended version unless policy explicitly completes | Terminal cancelled/skipped evaluation |
| Backfill after acknowledgement | Prior evidence not rewritten | New case version and approval invalidation if material |
| Model provider outage | Detection continues | Deterministic factual packet or human route |
| Owner directory stale | No invented recipient | Held/unowned case and escalation |
| Cancellation races notification | Late effect visible | Reconciled receipt and remediation state |

## Reliability checklist

- [ ] Semantic evaluation, case, and effect identities are stable and unique.
- [ ] Transactions never span slow external calls.
- [ ] State commits and queue publication use an outbox/equivalent recovery pattern.
- [ ] Every effect has intent, payload digest, approval, attempts, receipt, and reconciliation state.
- [ ] `unknown` is a first-class result.
- [ ] Retry ownership, limits, backoff, and deadlines are explicit.
- [ ] Backfills and semantic revisions preserve history and invalidate stale approvals.
- [ ] Pause, cancel, and kill-switch semantics are tested with in-flight effects.
- [ ] Shared data incidents correlate and inhibit child noise.
- [ ] Backpressure preserves authoritative state and high-criticality work.
- [ ] Runbooks can reconcile without inspecting raw database rows manually.

## Canonical references

- [Idempotency and side effects](../../reliability/idempotency-and-side-effects.md)
- [Failure taxonomy](../../reliability/failure-taxonomy.md)
- [Durable execution](../../runtime/durable-execution.md)
- [Run controls](../../runtime/run-controls.md)
- [Queues, scheduling, and backpressure](../../operations/queues-scheduling-and-backpressure.md)

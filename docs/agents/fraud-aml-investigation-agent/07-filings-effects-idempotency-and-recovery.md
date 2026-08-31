# Filings, Effects, Idempotency, Reconciliation, and Recovery

> **Purpose:** Prepare and execute authorized filings, escalations, and restrictive actions without duplicate, stale, ambiguous, or agent-authorized effects.  
> **Section index:** [Fraud and AML investigation agent](README.md)

## The agent prepares; a separate system commits

A SAR/STR, regulator disclosure, law-enforcement escalation, transaction hold/reject, property block/freeze, account restriction, or offboarding decision is a consequential external effect. The reasoning loop may assemble evidence and draft a schema-valid package. It has no destination credential and cannot convert a recommendation into approval.

The commit path requires:

1. an exact decision by an eligible person or deterministic legal/runbook gate;
2. current policy and jurisdiction evaluation;
3. a pinned evidence bundle and case version;
4. separation of duties and any required dual approval;
5. an immutable semantic effect intent;
6. destination-specific execution, receipt capture, status query, reconciliation, and postcondition verification.

Delivery is at-least-once in real systems. Business correctness comes from semantic identity, idempotency, state verification, and reconciliation—not from claiming “exactly once.”

## Filing preparation boundary

For a filing package, separate:

| Artifact | Owner | Mutation rule |
|---|---|---|
| Evidence bundle | Case workflow/evidence service | Content-addressed; material change creates a new bundle |
| Narrative draft | Agent or human, clearly attributed | Must cite bundle facts; edits versioned; no unsupported legal conclusion |
| Structured form draft | Deterministic mapper plus agent-assisted fields where allowed | Validate against pinned current schema; derived fields reproducible |
| Filing decision | Designated human/legal/compliance process | Binds case, bundle, schema, jurisdiction, rationale, approver, deadline |
| Submission intent | Effect service | Immutable exact destination, form hash, filing type, decision, semantic key |
| Submission/acknowledgment | Credentialed adapter | Receipt and raw status retained; model cannot call it |
| Amendment/correction | New decision and linked effect | Never overwrite original submission |
| Supporting-document response | Separately authorized workflow | Verify requester, authority, scope, confidentiality, and custody |

FinCEN publishes current BSA E-Filing specifications and supports the use of underlying supporting documentation; institutions should treat schema and request/production workflows as versioned integrations, not generated prose. Other jurisdictions require their own profiles and validation.

## Effect intent contract

~~~json
{
  "effect_id": "eff_01...",
  "effect_type": "regulatory_filing.submit",
  "semantic_key": "jurisdiction|institution|filing-type|case-generation|decision-id",
  "tenant_id": "tenant_...",
  "case_id": "case_...",
  "case_version": 23,
  "decision_id": "dec_...",
  "evidence_bundle_hash": "sha256:...",
  "payload_ref": "artifact_...",
  "payload_hash": "sha256:...",
  "destination": "fincen-bsa-efiling",
  "destination_schema": "...",
  "jurisdiction_profile": "...@sha256:...",
  "policy_commit": "sha256:...",
  "approvals": ["approval_..."],
  "requested_by": "workforce_...",
  "deadline_at": "...",
  "approval_expires_at": "...",
  "state": "approved",
  "created_at": "..."
}
~~~

The service derives the semantic key; it does not trust one supplied by the model or UI. An intent cannot be repointed to a different customer, transaction, filing, destination, payload, or action. Such a change creates a new decision and effect.

## Effect state machine

~~~mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> AwaitingApproval: validated proposal
    AwaitingApproval --> Approved: eligible approval(s)
    AwaitingApproval --> Rejected
    Approved --> Dispatching: lease + current policy check
    Dispatching --> Confirmed: receipt + postcondition
    Dispatching --> Retryable: definite non-commit failure
    Retryable --> Dispatching: bounded retry
    Dispatching --> Unknown: timeout / ambiguous response
    Unknown --> Reconciling
    Reconciling --> Confirmed: authoritative status found
    Reconciling --> Approved: proven not committed and approval current
    Reconciling --> ManualResolution: cannot establish status
    Approved --> Cancelled: pre-dispatch cancellation
    Confirmed --> CorrectionPending: approved amendment / compensating workflow
    CorrectionPending --> Confirmed: linked effect resolved
    Rejected --> [*]
    Cancelled --> [*]
~~~

Do not transition from `unknown` directly to retry. Query by destination receipt, client/reference ID, exact payload hash, or domain-specific state first. If the destination cannot support reliable status lookup, lower autonomy, add human confirmation, or reject the integration.

## Idempotency model

| Layer | Identity | Behavior |
|---|---|---|
| Alert admission | Producer/rule/subject/window/revision semantic key | Link or supersede duplicates; do not reopen blindly |
| Proposal application | Proposal ID + case expected version + semantic artifact key | Duplicate transport returns prior application result |
| Approval | Decision/effect + approver role + approval generation | Duplicate click/message cannot create a second approval |
| Filing submission | Jurisdiction/institution/form type/case generation/decision | Same approved payload resolves to same intent/result |
| Restrictive action | Object/action/legal basis/decision/generation | Repeated request queries current state and returns prior result |
| Notification/task | Recipient/channel/template/business event | Suppress duplicate delivery according to defined window |

Store the key, request hash, state, attempts, receipts, and result long enough to cover retries, regulator/destination lag, disaster recovery, and legal retention. If the same key arrives with a different request hash, reject it as a conflict.

## Dispatch algorithm

~~~text
1. Load immutable intent by effect_id and acquire a short lease.
2. Verify state is dispatchable and no confirmed/reconciling equivalent exists.
3. Re-evaluate current actor/service identity, policy, jurisdiction, target state,
   approval eligibility/expiry, segregation of duties, deadline, and kill switches.
4. Build the destination request deterministically from the approved payload hash.
5. Record attempt and client/reference ID before network dispatch where possible.
6. Send once with destination idempotency support if available.
7. Persist raw receipt/status and verify the domain postcondition.
8. On definite pre-commit failure, apply bounded backoff within deadline.
9. On ambiguous outcome, mark unknown and reconcile; never blind-retry.
10. Emit business/audit events transactionally; update case from ledger state.
~~~

Policy is checked both at approval and immediately before dispatch. Historical policy explains the decision; current policy can revoke the ability to execute it. Emergency revocation wins over a pinned behavior release.

## Destination-specific postconditions

HTTP 200 or “accepted” may not be the business postcondition.

| Effect | Minimum confirmation | Reconciliation source |
|---|---|---|
| Regulatory filing | Destination tracking/acknowledgment ID, accepted/rejected status, payload hash or stable reference | Filing gateway/status report plus scheduled downstream acknowledgment |
| Amendment/correction | Link to original filing and new acceptance status | Filing gateway and case filing ledger |
| Transaction hold | Exact transaction/object is in requested state, scope and expiry correct | Payment/control system of record |
| Reject/block/freeze | Exact legal object and amount/property/status affected; required report/task created | Sanctions/payment system plus reporting workflow |
| Account restriction/offboarding | Correct account/party scope, effective time, exceptions and notices handled by authorized process | Customer/account workflow system |
| Law-enforcement/regulator escalation | Authorized recipient, secure channel, delivered/accepted receipt, disclosed scope | Approved disclosure/case system—not email “sent” alone |

Some actions cannot be automatically compensated. A mistaken filing, disclosure, block, freeze, or account closure normally requires a separately authorized correction, release, legal, or customer-remediation process. Never label a generic undo tool as rollback.

## Reconciler

The reconciler runs independently of the reasoning service:

- scan pending, dispatching-with-expired-lease, unknown, and correction states;
- query the destination using stored semantic/client/receipt identifiers;
- compare intended and actual object, state, amount/scope, payload hash, and timing;
- append observations; never rewrite an inconvenient receipt;
- promote to confirmed only after the defined postcondition;
- if proven not committed and approval remains current, return to approved for a bounded retry;
- escalate unresolved mismatch or deadline risk to an eligible human;
- page on unknown-state age, duplicate/conflicting outcomes, or unauthorized state.

Reconciliation is part of correctness and therefore has its own SLO, capacity, runbook, and disaster-recovery test.

## Cancellation and supersession

Cancellation is a durable event with a reason, actor, time, target generation, and policy decision. It prevents new dispatch but does not pretend to cancel a request already committed externally. The worker checks cancellation before every new read group and before dispatch. If cancellation races with dispatch, the effect enters reconciliation.

A corrected case or changed decision creates a new generation. Link the old and new decision/effect, block stale generations, and explicitly decide whether an amendment, release, or other compensating action is required.

## Crash and disaster recovery

| Crash point | Durable evidence | Recovery |
|---|---|---|
| Before proposal applied | No event | Safely rerun bounded reasoning from case state |
| After event commit, before acknowledgment | Event/idempotency row exists | Duplicate proposal returns prior result |
| After approval, before outbox publication | Decision/effect/outbox in one transaction | Publisher resumes from outbox |
| Before external dispatch | Attempt/lease state only | Expired lease can be reclaimed after policy check |
| After dispatch, before receipt persisted | Client ID/attempt exists; outcome unknown | Reconcile destination; no blind retry |
| After receipt, before case projection update | Receipt/effect ledger exists | Rebuild projection from ledger/event |
| During region loss | Replicated state and defined RPO/RTO | Fail over; keep effect dispatch single-writer/fenced; reconcile all in-flight work |

Recovery tests must include worker crash, database failover, duplicate/out-of-order queue delivery, lost acknowledgment, destination latency, clock skew, credential rotation, policy revocation, and region failover. “The workflow engine retries” is not sufficient evidence.

## Filing confidentiality and operational security

- Restrict knowledge that a SAR/STR was considered or filed according to applicable law and policy.
- Do not put filing existence, narrative, or supporting documents in general customer profiles, broad search indexes, model memory, ordinary support logs, analytics warehouses, or notification text.
- Use dedicated workforce eligibility, purpose, display masking, export controls, access logging, anomaly detection, and retention.
- Keep regulator/destination credentials in the effect adapter's secret boundary and use short-lived identity where supported.
- Verify external requests for supporting documentation through approved channels; the model never decides legitimacy from an email or document.
- Test screenshots, browser history, trace attributes, exception bodies, queues, dead-letter stores, and backups for disclosure.

The United Kingdom's NCA warns that a SAR is not a crime report and highlights tipping-off concerns; every jurisdiction needs its own terminology, confidentiality, and disclosure controls.

## Failure matrix

| Failure | Required behavior |
|---|---|
| Filing schema changes near deadline | Stop affected submission, switch to reviewed current schema, revalidate payload and approval materiality |
| Approval expires before dispatch | Return to approval; do not dispatch |
| Case/evidence changes after approval | Mark intent stale; require deterministic materiality check and usually reapproval |
| Timeout after submit | `unknown` → destination lookup/reconciliation |
| Duplicate queue delivery | Same effect/key returns same ledger state; no second dispatch |
| Conflicting same-key payload | Reject and page; investigate producer/case generation |
| Destination says accepted, later rejected | Append status transition; reopen correction/deadline workflow |
| Hold/block scope differs | Do not mark confirmed; escalate mismatch and prevent broader retry |
| Credential/policy revoked | Stop dispatch; preserve pending state; route to authorized operations |
| Deadline cannot be met safely | Immediate human/legal escalation; never bypass review or controls |
| Unrecoverable unknown outcome | Manual resolution with two-person control where impact warrants; preserve all attempts |

## Checklist

- [ ] The model/runtime has no filing, payment-control, account-control, disclosure, or notification credential.
- [ ] Filing draft, filing decision, effect intent, dispatch, receipt, amendment, and supporting-document response are distinct.
- [ ] Every effect has a server-derived semantic key, immutable request hash, approval, expiry, current-policy check, and state machine.
- [ ] Unknown outcomes enter reconciliation and cannot be blindly retried.
- [ ] Destination-specific postconditions, status queries, deadlines, and correction paths are documented and tested.
- [ ] Cancellation, supersession, crash points, DR, and stale approvals have exercised recovery paths.
- [ ] SAR/STR confidentiality applies to prompts, indexes, logs, traces, screenshots, exports, queues, backups, and support access.

## Sources and next guide

- [FinCEN — BSA E-Filing filing information and specifications](https://bsaefiling.fincen.gov/filing-information)
- [FinCEN — SAR supporting documentation](https://www.fincen.gov/resources/statutes-regulations/guidance/suspicious-activity-report-supporting-documentation)
- [FinCEN — Frequently Asked Questions Regarding the FinCEN SAR](https://www.fincen.gov/resources/frequently-asked-questions-regarding-fincen-suspicious-activity-report-sar)
- [UK National Crime Agency — Suspicious Activity Reports](https://www.nationalcrimeagency.gov.uk/what-we-do/crime-threats/money-laundering-and-illicit-finance/suspicious-activity-reports)
- [AWS Builders' Library — Making retries safe with idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)
- [Idempotency and side effects](../../reliability/idempotency-and-side-effects.md)
- [Failure taxonomy](../../reliability/failure-taxonomy.md)

Next: [Security, privacy, fairness, and governance](08-security-privacy-fairness-and-governance.md).

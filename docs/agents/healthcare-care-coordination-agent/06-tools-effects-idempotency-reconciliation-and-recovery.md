# Tools, Effects, Idempotency, Reconciliation, and Recovery

A care-coordination tool is a typed capability with an authority tier and failure contract—not a function name in a prompt. This guide makes reads, writes, timeouts, duplicates, unknown outcomes, cancellations, and recovery explicit.

## Tool taxonomy

| Tool class | Examples | Default tier | Key risk |
|---|---|---|---|
| Identity read | Patient binding, MPI candidates, merge/split status | D1 | Wrong-patient binding |
| Policy decision | Purpose/role/consent/proxy evaluation | D1 internal control | Treating model input as policy |
| Clinical evidence read | ServiceRequest, CarePlan, Task, selected records | D1 | Excess PHI or stale version |
| Directory read | Service, practitioner role, coverage route | D1 | Stale owner |
| Scheduling read | Schedule/Slot/Appointment | D1 | Mistaking availability for booking |
| Draft/stage | Draft message, provisional hold, task proposal | D2 | Staged work leaks or outlives authority |
| Scheduling effect | Book/reschedule/cancel | D3 | Care delay, duplicate, wrong patient |
| Referral effect | Submit/update administrative referral state | D3 | Altered clinical intent or lost referral |
| Prior-auth effect | Submit/cancel/respond | D3 | Unsupported clinical claims, duplicate |
| Communication effect | Send portal/SMS/email/phone job | D3 | Unauthorized disclosure |
| Handoff effect | Assign/transfer accountable work | D3 when responsibility changes | Orphaned case |
| Authority/policy administration | Change proxy, consent, role, scope, routing policy | D4 | Privilege escalation |

D4 tools must not be available to the model. The model may produce a human-reviewed change proposal in a separate process.

## Tool contract

~~~yaml
tool_contract:
  tool_id: scheduler.book.v3
  owner: scheduling-platform
  intended_use: create-one-appointment
  authority_tier: D3
  input_schema: versioned-schema-ref
  output_schema: versioned-schema-ref
  patient_binding_required: true
  authority_decision_required: true
  approval_mode: exact | narrow-preauthorization
  allowed_tenants: [opaque]
  parameter_allowlist:
    - patient_binding_id
    - slot_ref
    - service_request_ref
    - timezone
  preconditions:
    - binding-stable
    - slot-fresh
    - clinical-source-version-unchanged
  semantic_effect_key_fields:
    - tenant_id
    - patient_binding_id
    - action
    - slot_ref
    - service_request_ref
  timeout_policy: named-policy
  retry_policy: reconcile-before-retry
  supports:
    downstream_idempotency: true
    conditional_commit: true
  success_postconditions:
    - appointment-id-present
    - appointment-subject-matches-binding
    - appointment-slot-matches-request
  failure_classes:
    - invalid
    - unauthorized
    - conflict
    - rate-limited
    - transient-before-commit
    - unknown-after-dispatch
  audit_fields: [actor, purpose, effect-id, approval, receipt]
  adapter_version: opaque
~~~

The adapter validates the contract again. Prompt instructions are not authorization.

## Safe tool result

Every result distinguishes transport, domain, and effect outcome:

~~~yaml
tool_result:
  invocation_id: opaque
  tool_id: scheduler.book.v3
  effect_id: opaque
  transport_status: response | timeout | connection-failure
  outcome: succeeded | rejected | conflict | failed | unknown
  external_request_ids: [opaque]
  receipt_refs: [opaque]
  observed_resource_refs: [versioned-reference]
  postcondition_status: satisfied | not-satisfied | not-checked
  retry_disposition: never | safe-same-key | reconcile-first | human-review
  occurred_at: timestamp
  adapter_version: opaque
~~~

A timeout is not a failure outcome. It is unknown unless the adapter can prove the request never crossed the commit boundary.

## Effect state machine

~~~mermaid
stateDiagram-v2
    [*] --> Proposed
    Proposed --> Authorized: policy and approval pass
    Proposed --> Rejected: invalid or unauthorized
    Authorized --> Dispatched: outbox delivery
    Dispatched --> Succeeded: receipt and postcondition
    Dispatched --> Failed: definitive rejection or no commit
    Dispatched --> Unknown: timeout or ambiguous response
    Failed --> Authorized: retry policy allows same effect
    Unknown --> Reconciling
    Reconciling --> Succeeded: intended state observed
    Reconciling --> Failed: no effect proved and retry allowed
    Reconciling --> Diverged: unexpected state
    Diverged --> HumanReview
    Succeeded --> [*]
    Rejected --> [*]
~~~

The workflow stores an effect before dispatch through a transactional outbox or equivalent atomic handoff. External calls never occur invisibly inside a state update.

## Semantic idempotency

An idempotency key represents the business effect, not the invocation.

Bad keys:

- random request UUID generated for each retry;
- workflow attempt number;
- raw prompt hash;
- patient name or other PHI;
- provider request ID alone.

Good key inputs:

- tenant;
- opaque patient binding;
- action type;
- authoritative target/resource;
- normalized parameters;
- source intent version;
- approval/preauthorization identity where material.

~~~text
effect_id = HMAC(
  tenant_secret,
  canonical(
    tenant_id,
    patient_binding_id,
    action,
    target_ref,
    normalized_parameters,
    clinical_source_version
  )
)
~~~

HMAC or an opaque lookup prevents identifiers from leaking through logs and metrics. Canonicalization rules and schema version belong in the contract.

Before treating two requests as the same effect, compare normalized parameters. If they differ, stop for conflict rather than reusing a key.

## Layered effect safety

Use all applicable layers:

1. **Local effect uniqueness:** one semantic effect record.
2. **State precondition:** case/task/source versions still match.
3. **Fresh authority:** patient binding, purpose, proxy, role, approval, and safety state pass.
4. **Downstream idempotency:** vendor idempotency token or business identifier.
5. **Conditional commit:** FHIR If-Match, If-None-Exist, or equivalent when supported.
6. **Receipt capture:** external request/resource/version identifiers.
7. **Postcondition verification:** authoritative read proves exact intended state.
8. **Reconciliation:** unknown or divergent outcomes enter a durable queue.
9. **Fencing:** a worker that lost its lease cannot dispatch.

FHIR conditional operations and server transactions help only when the server supports their exact semantics. A transaction on one FHIR server does not make a scheduling, payer, communication, and local-database workflow atomic.

## Approval binding

D3 approval is a record:

~~~yaml
effect_approval:
  approval_id: opaque
  effect_id: opaque
  approver_id: opaque
  approver_role: opaque
  patient_binding_id: opaque
  action: submit-prior-authorization
  normalized_parameters_hash: opaque
  disclosure_data_classes: [approved-enum]
  destination_ref: opaque
  source_versions: [opaque]
  policy_version: opaque
  behavior_version: opaque
  approved_at: timestamp
  expires_at: timestamp
  status: active | consumed | revoked | expired
~~~

If parameters, patient binding, destination, source, policy, or behavior changes materially, request a new approval.

Narrow deterministic preauthorization is acceptable only when governance defines an exact workflow, population, action, parameters, data classes, conditions, ceiling, expiration, and monitoring rule. It is not a prompt phrase like “handle scheduling.”

## Reconciliation

### Reconciliation record

~~~yaml
reconciliation_job:
  reconciliation_id: opaque
  effect_id: opaque
  target_adapter: opaque
  expected_postcondition: versioned-predicate
  observed_refs: []
  status: pending | matched | absent | diverged | inaccessible | escalated
  attempts: integer
  next_attempt_at: timestamp
  deadline_at: timestamp
  owner_queue: opaque
  resolution_ref: optional
~~~

### Decision table

| Observed state | Action |
|---|---|
| Exact intended state exists with same business identifier | Mark succeeded and attach receipt |
| No state exists and adapter proves original request could not commit | Retry same semantic effect under policy |
| No state visible but downstream consistency window is open | Wait and recheck |
| Multiple possible results exist | Freeze and route |
| State differs in patient, target, parameters, or source version | Treat as divergence and incident candidate |
| Access denied after dispatch | Keep unknown; route to operations/privacy as appropriate |
| Reconciliation deadline exceeded | Alert owner; block case closure |

Do not create a new effect merely to escape an unknown old effect.

## Cancellation and compensation

Cancellation is a new domain effect, not deletion of history.

- Check whether downstream work already began.
- Preserve original clinical intent and who requested cancellation.
- Revalidate patient, authority, appointment/referral identity, and impact.
- Use exact effect IDs and receipts.
- Verify the canceled state.
- Notify affected owners and patient through policy.
- If cancellation cannot undo real-world work, route for human recovery.

FHIR Task warns that changing a task to canceled may not stop activity already happening. Model cancellation as a request plus verified outcome, not a flag flip.

Compensation can restore administrative state, but it cannot reverse a disclosure, a message already read, a missed appointment, or care already delivered. Those require incident and human follow-up.

## Retry policy

| Failure class | Automatic retry? | Rule |
|---|---|---|
| Validation/unauthorized | No | Fix evidence or authority |
| Clinical-boundary violation | No | Safety/clinical review and evaluation failure |
| Conflict/precondition failed | No blind retry | Reload source and replan |
| Rate limit | Yes, bounded | Respect Retry-After/jitter/deadline |
| Network failure before request sent | Yes, same effect ID | Adapter must prove no dispatch |
| Timeout after possible dispatch | No | Reconcile first |
| Definitive downstream rejection | Usually no | Route based on reason |
| Transient read | Yes, bounded | Do not let staleness become truth |
| Model schema failure | Limited regeneration or deterministic repair | No tool/effect until valid |
| Safety-route failure | Use approved backup immediately | Alert; do not continue routine work |

Retries have maximum attempts, wall-clock deadline, exponential backoff with jitter, circuit breakers, and an owner when exhausted.

## Recovery scenarios

### Worker crash before dispatch

The effect remains authorized in the outbox. A fenced worker resumes dispatch with the same effect ID.

### Crash after dispatch, before receipt persistence

The effect becomes unknown. Reconciliation queries the downstream business identifier or postcondition before any retry.

### Model/provider outage

Deterministic paths continue. Ambiguous cases wait in a visible queue or route to humans. No provider-side session is needed for recovery.

### Adapter release defect

Pause that adapter/action, preserve unaffected workflows, reconcile recent effects by adapter version, roll back, and reprocess only when outcome is known.

### EHR/HIE/payer/scheduler outage

Apply dependency-specific circuit breakers and backoff. Expose honest delayed status. Preserve safety routes through independent backup channels where required.

### Patient-binding invalidation

Stop undispatched effects, mark in-flight effects for priority reconciliation, invalidate approvals and context, and start identity correction review.

### Behavior rollback

New cases use the prior manifest. In-flight cases follow a declared migration policy; effects retain the original behavior and adapter versions. Do not silently reinterpret past proposals.

## Tool security rules

- Mint short-lived, audience-bound credentials outside the model process.
- Enforce tenant/patient/purpose scopes at adapter and data layers.
- Store secrets in an approved secret manager; never in prompts, traces, or tool results.
- Allowlist hosts, resource types, fields, operations, recipients, and channels.
- Treat all retrieved notes, documents, payer responses, and messages as untrusted data.
- Escape or strip active content; scan attachments; isolate parsers.
- Enforce output schemas and size limits.
- Redact raw payloads from standard logs.
- Version and sign tool manifests where supported.
- Disable dynamic tool registration in patient runs.

## Reliability test suite

For every D3 tool, test:

- duplicate command before dispatch;
- duplicate outbox delivery;
- concurrent same-patient commands;
- stale case and source versions;
- proxy revocation between approval and dispatch;
- timeout before and after commit;
- late success after local timeout;
- malformed or misleading receipt;
- downstream idempotency ignored;
- conditional update conflict;
- wrong patient or target in response;
- partial FHIR transaction behavior;
- adapter crash during response parsing;
- reconciliation read denied or stale;
- cancellation after real-world work began;
- old worker dispatch after lease loss;
- deploy rollback with in-flight effects.

## Effect readiness checklist

- [ ] Every tool has owner, intended use, D-tier, schemas, and failure classes.
- [ ] Model outputs cannot invoke adapters directly.
- [ ] Effect IDs are semantic, opaque, stable, and parameter-checked.
- [ ] Authorization and approval bind exact patient/action/parameters/source/destination/version.
- [ ] Outbox and workflow transition are atomic or equivalently safe.
- [ ] Conditional writes and downstream idempotency are capability-tested.
- [ ] Receipt and postcondition define success.
- [ ] Unknown is a first-class durable state.
- [ ] Reconciliation has deadlines, owners, and incident escalation.
- [ ] Cancellation and compensation acknowledge irreversible real-world effects.

## Related guides

- Previous: [State, Events, Context, Memory, and Planning](05-state-events-context-memory-and-planning.md)
- Next: [Security, Privacy, Accessibility, and Clinical Safety](07-security-privacy-accessibility-and-clinical-safety.md)
- Home: [Healthcare Care Coordination Agent](README.md)
- Canonical: [Idempotency and Side Effects](../../reliability/idempotency-and-side-effects.md)


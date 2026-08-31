# Scheduling, Referrals, Prior Authorization, Tasks, and Handoffs

This guide turns the authority boundary into closed-loop administrative workflows. Each workflow has an accountable owner, authoritative input, permitted model role, verifiable effect, exception route, and definition of closure.

## Shared coordination case

All workflow types use one case envelope:

~~~yaml
coordination_case:
  case_id: opaque
  tenant_id: opaque
  patient_binding_id: opaque
  workflow_kind: scheduling | referral | prior-authorization | care-task | outreach
  source_refs: [versioned-reference]
  administrative_owner: opaque
  clinical_owner_ref: opaque
  state: intake | blocked | ready | executing | waiting | safety-hold | reconciling | closed
  next_deadline: optional-timestamp
  open_task_ids: [opaque]
  unresolved_contradiction_ids: [opaque]
  safety_escalation_ids: [opaque]
  open_effect_ids: [opaque]
  case_version: integer
  behavior_version: opaque
~~~

Workflow state never replaces the EHR, payer, scheduler, or communication system's domain truth. It records what coordination must happen and what evidence proves it happened.

## Scheduling workflow

FHIR Schedule and Slot expose availability; Appointment represents booking state. A slot that appears free does not guarantee that a booking will succeed.

~~~mermaid
flowchart TD
    O[Clinician-authored service or approved scheduling need] --> E[Determine eligible service, site, and constraints]
    E --> S[Read fresh schedule and slot observations]
    S --> P[Present options with timezone, location, accessibility, and cost caveat]
    P --> C{Patient or authorized representative confirms exact option?}
    C -- No --> W[Wait or route to scheduler]
    C -- Yes --> A[Revalidate binding, authority, slot, and clinical source]
    A --> B[Create semantic booking effect]
    B --> R{Receipt and postcondition verified?}
    R -- Yes --> N[Record appointment and minimal confirmation]
    R -- No, known failure --> S
    R -- Unknown --> X[Reconcile before retry]
    N --> F[Track cancellation, reschedule, no-show, or attendance exception]
~~~

### Scheduling authority table

| Action | Tier | Required evidence |
|---|---|---|
| Read eligible sites/slots | D1 | Verified patient/purpose and authorized service context |
| Draft option list | D1 | Fresh slot observations and explicit logistical preferences |
| Create expiring provisional hold | D2 | Vendor semantics, expiration, and release path |
| Book or reschedule | D3 | Exact option confirmation or narrow preauthorization, fresh preconditions, receipt |
| Cancel | D3 | Exact appointment, authorized requester, impact-aware confirmation/policy |
| Change clinical service, urgency, preparation, or safety constraints | Prohibited | Clinician-owned workflow |

The model may rank only by explicit logistical constraints such as location, accessibility, language, time window, transport, and stated preference. It may not decide that a later slot is clinically safe.

### Scheduling closure

A scheduling task closes only when the configured source reports the expected appointment ID/status/version and the patient communication requirement is satisfied or explicitly waived by policy. A delivery attempt alone is not closure.

## Referral workflow

The clinician-authored ServiceRequest or equivalent source owns requested service, reason, intent, and clinical priority. The coordinator owns administrative completeness, destination, acceptance, scheduling follow-up, exception routing, and handoff evidence.

~~~mermaid
stateDiagram-v2
    [*] --> Received
    Received --> Blocked: administrative evidence missing
    Blocked --> Received: source-backed evidence supplied
    Received --> Submitted: authorized referral effect
    Submitted --> Accepted: destination receipt
    Submitted --> Rejected: destination response
    Accepted --> Scheduled: appointment verified
    Accepted --> Overdue: no scheduling by local policy
    Scheduled --> FollowUp: attendance or result follow-up required
    Rejected --> Review: clinician or coordinator action needed
    Overdue --> Escalated: referring team notified
    FollowUp --> Closed: required handoff evidence and acknowledgment
    Escalated --> Closed: accountable resolution
~~~

### Referral task contract

~~~yaml
referral_task:
  task_id: opaque
  case_id: opaque
  service_request_ref: versioned-clinician-source
  destination_ref: versioned-directory-entry
  administrative_requirement_code: approved-enum
  requirement_source_ref: opaque
  owner_assignment_id: opaque
  due_at: timestamp
  status: open | blocked | submitted | accepted | rejected | overdue | closed
  completion_evidence_refs: []
  escalation_route_ref: opaque
~~~

The model may classify a missing administrative requirement only when it can cite the source rule and clinical document span. It cannot rewrite the referral reason, priority, diagnosis, or requested service to make the referral pass.

### Closed-loop referral rules

- Capture destination receipt; do not equate an outbound message with acceptance.
- Monitor accepted referrals for locally defined scheduling or attendance exceptions.
- Notify the referring owner through an approved route when expected progress does not occur.
- Preserve rejection reason and distinguish administrative rejection from a clinical decision.
- Require an accountable owner for each unresolved exception.
- Close only with the organization's defined evidence: accepted transfer, appointment/follow-up evidence, or documented alternative.

## Prior-authorization workflow

Prior authorization sits at the boundary between care coordination, clinical attestation, payer administration, and claims. This agent coordinates the request; it does not own medical necessity, coding, payer adjudication, or finance.

### Permitted lifecycle

1. Read the clinician-authored request and coverage evidence.
2. Discover payer requirements through a versioned, supported contract.
3. Identify missing administrative evidence with source references.
4. Populate only fields supported by authoritative records.
5. Mark clinical attestation fields for an accountable clinician.
6. Validate the exact payer/profile/transaction contract.
7. obtain D3 approval or use a narrowly approved submission policy;
8. submit with a semantic effect ID and capture receipt;
9. track pending, request-for-information, approved, denied, canceled, and unknown statuses;
10. route payer questions, denial/appeal decisions, and clinical alternatives to their accountable owners;
11. reconcile payer and EHR status before closure.

### Field authority

| Field | Agent behavior | Owner |
|---|---|---|
| Patient/coverage identifiers | Copy only after verified binding and coverage match | Identity/coverage systems |
| Requested service | Preserve source exactly | Ordering clinician/source request |
| Diagnosis/procedure code | Copy with source/version or mark missing | Clinician/coding process |
| Clinical note/result | Attach authorized source artifact; minimize | EHR and accountable clinician |
| Medical necessity narrative | Draft only from cited clinician evidence if policy allows; require attestation | Clinician |
| Payer requirement | Use versioned payer/CRD rule and retain source | Payer contract |
| Authorization decision | Record response without reinterpretation | Payer |
| Appeal or alternative treatment | Route | Clinician/payer/authorized operations |
| Claim/payment | Link externally if needed | Claims/finance category |

Da Vinci CRD, DTR, and PAS can standardize parts of discovery, documentation, and submission. They are versioned implementation guides, and actual payer support, X12 mappings, exceptions, and CMS applicability must be verified. Do not encode one payer portal or one regulatory date as a universal workflow.

## Care-plan task coordination

FHIR CarePlan describes intended care; Task can represent work status. The agent projects administrative tasks from a versioned, clinician-owned plan.

Allowed:

- create a local coordination task tied to a specific plan/activity version;
- remind the assigned team or patient through an approved channel;
- track dependency, due date, acknowledgment, and completion evidence;
- surface a stale, conflicting, or missing source;
- route overdue or concerning work.

Not allowed:

- add, remove, or change a clinical goal or activity;
- decide a task is clinically unnecessary;
- mark clinical completion without the designated source;
- infer that a missed task is safe;
- reinterpret diagnosis, medication, or result evidence.

When the source plan version changes, invalidate affected projections and require deterministic remapping or human review. Never silently carry a task forward.

## Medication and diagnosis evidence

Treat these records as immutable observations with lineage:

~~~yaml
clinical_evidence_ref:
  evidence_id: opaque
  patient_binding_id: opaque
  source_system: ehr-a
  resource_type: MedicationRequest | Condition | AllergyIntolerance | Observation | DocumentReference
  resource_id: opaque
  resource_version: opaque
  observed_or_authored_at: timestamp
  retrieved_at: timestamp
  supported_assertion: narrow-text
  terminology_system: optional
  terminology_version: optional
  provenance_ref: optional
  confidence: source-reported | extractor-reported
  status: current-source-observation | superseded | conflicting | unavailable
~~~

If two sources disagree, preserve both and create a contradiction. A clinician or pharmacist decides the clinical truth. The model cannot “pick the newer one” unless the narrow task is simply to identify which record has a later timestamp—and even then it cannot infer clinical correctness.

## Outreach and communication

Separate the lifecycle:

| State | Meaning |
|---|---|
| Requested | Authorized workflow asks for communication |
| Drafted | Content exists but has not been approved/dispatched |
| Authorized | Actor, patient, purpose, audience, channel, and payload class passed policy |
| Dispatched | Adapter accepted the send request |
| Delivered | Channel reports delivery under its semantics |
| Acknowledged | Intended recipient confirmed receipt or response |
| Comprehended | Human workflow established understanding where required |
| Failed/unknown | Delivery did not occur or outcome is uncertain |

FHIR CommunicationRequest is a request, and Communication can record transmission; neither proves comprehension.

### Outreach contract

~~~yaml
communication_delivery:
  communication_id: opaque
  case_id: opaque
  patient_binding_id: opaque
  intended_recipient_id: opaque
  representative_grant_id: optional
  purpose: care-coordination
  channel: portal | phone | sms | email | mail | human-interpreter
  address_version_ref: opaque
  template_id: opaque
  template_version: opaque
  payload_data_classes: [appointment-metadata]
  language: BCP47-tag
  accommodations: [opaque]
  authority_decision_ref: opaque
  effect_id: opaque
  status: requested | drafted | authorized | dispatched | delivered | acknowledged | failed | unknown
  receipt_refs: []
~~~

Use the least revealing content appropriate for the channel. Honor verified language, channel, confidential-communication, accessibility, and accommodation requirements. Retain phone/human/interpreter alternatives; do not assume a portal or smartphone.

## Handoff contract

A handoff transfers responsibility only after acknowledgment.

~~~yaml
handoff:
  handoff_id: opaque
  case_id: opaque
  patient_binding_id: opaque
  sender_assignment_id: opaque
  receiver_assignment_id: opaque
  handoff_type: administrative | clinical-review | safety
  reason_code: approved-enum
  requested_action: closed-set
  evidence_refs: [versioned-reference]
  unresolved_questions: []
  due_at: timestamp
  fallback_route_ref: opaque
  status: proposed | sent | acknowledged | declined | overdue | completed
  acknowledged_by: optional
  acknowledged_at: optional
  completion_evidence_refs: []
~~~

Rules:

- A queue insertion is not acknowledgment.
- A model summary is not a substitute for source links.
- The sender retains responsibility until the configured transfer condition is met.
- Safety handoffs use shorter, governance-approved timers and backup routes.
- A declined or overdue handoff remains visible and escalates.
- Clinical acceptance cannot be inferred from message delivery.

## Safety and deterioration escalation

Organizations must define trigger evidence, queues, time expectations, fallback routes, and staffing with clinical governance. The blueprint intentionally does not list symptom thresholds or clinical advice.

The runtime supports three conservative trigger types:

- deterministic triggers configured by clinical safety owners;
- explicit patient or staff statements routed without model disposition;
- material model uncertainty when content might be clinical or safety-relevant.

On trigger, create a safety escalation, hold routine work, minimize further automated messaging, and transfer to the approved clinical service. The agent never closes the escalation based on its own inference.

## Composite closed-loop scenario

The following trajectory is a realistic testable bundle, not a promise that every organization uses the same systems.

~~~mermaid
sequenceDiagram
    participant R as Representative
    participant G as Identity and policy gate
    participant W as Durable workflow
    participant E as EHR and directory
    participant P as Payer
    participant S as Scheduler
    participant C as Coordinator or clinician
    participant M as Approved messaging

    R->>G: Request referral scheduling
    G->>G: Bind actor and patient; verify schedule-only grant
    G->>W: Permit minimal administrative fields
    W->>E: Versioned read of referral, coverage refs and destination
    E-->>W: Clinician-authored intent plus source versions
    W->>P: Read requirements/status for exact payer and service date
    alt Clinical evidence or attestation missing
        W->>C: Task with exact missing field and source
        C-->>W: Attested source version
    end
    W->>P: Submit semantic prior-auth effect
    alt Submit outcome unknown
        W->>W: Hold dependent booking and reconcile by payer trace ID
        P-->>W: Observed single payer request/decision
    else Definitive response
        P-->>W: Receipt and status
    end
    alt Approved and still current
        W->>S: Query fresh eligible slots
        S-->>W: Options with observed-at and expiry
        W->>R: Minimal accessible options
        R-->>W: Confirm exact slot
        W->>G: Revalidate grant, address, binding and source versions
        W->>S: Book one semantic appointment effect
        S-->>W: Appointment ID/version
        W->>S: Read back patient, service, time and status
        W->>M: Send minimal confirmation under channel policy
        M-->>W: Dispatch/delivery receipt
    else Denied, more information, or unmapped
        W->>C: Route decision and evidence; no treatment alternative inferred
    end
    W->>E: Closed-loop status/handoff under qualified write policy
    E-->>W: Destination/owner acknowledgment
~~~

Required properties:

- The representative can schedule only within the current grant; diagnosis, notes, denial rationale and other sensitive fields remain filtered unless separately authorized.
- The model may classify an administrative gap or draft a minimal status message, but it cannot supply a code, medical-necessity statement, urgency, service, or treatment alternative.
- Payer transport acceptance, information request and final decision are distinct; a timeout blocks dependent work until reconciliation.
- Slot choice is confirmed by the patient/authorized representative and revalidated against the clinician-authored service; the runtime never substitutes a slot or service after a race.
- Handoff closure requires destination acknowledgment. Until then, the sending owner and overdue clock remain active.

### Correction trajectory

Suppose an ADT/MPI event reports that the source chart was merged incorrectly after the payer submission but before the confirmation message:

1. record the correction event with source message/control/version, old and candidate patient references, occurrence and observation times;
2. place the case on identity hold, cancel undispatched work and invalidate context, proxy decisions, approvals and slot offers;
3. do **not** resend or cancel the payer request blindly—mark its outcome/subject binding for priority reconciliation using the original trace/control ID;
4. have the identity steward resolve merge/split/rebind; preserve historical references rather than rewriting them;
5. assess whether any prior payer disclosure used the wrong subject and route the privacy/clinical-safety incident process when indicated;
6. reload referral, coverage, consent/proxy and clinical evidence for the corrected binding and expose contradictions;
7. if an appointment effect may have committed, query by the original semantic business identifier and correct through an authorized new cancellation/reschedule effect;
8. issue any corrected communication only to the newly verified address/recipient under a new approval; do not imply the earlier message vanished;
9. require propagation receipts from workflow projections, retrieval indexes, human queues and affected adapters before releasing the identity hold;
10. reopen closed tasks/handoffs whose completion evidence no longer binds the correct patient.

## Workflow failure matrix

| Failure | Unsafe shortcut | Required behavior |
|---|---|---|
| Slot disappears before booking | Choose the next slot automatically | Requery and obtain confirmation under policy |
| Scheduler times out after submit | Retry immediately | Mark unknown and reconcile |
| Referral destination rejects | Rewrite clinical reason | Preserve response and route |
| No specialist appointment appears | Close because referral was sent | Notify accountable owner and escalate overdue state |
| Payer asks for unsupported clinical field | Infer from note | Mark missing; request clinician/coding review |
| CarePlan changes mid-run | Continue old tasks | Invalidate projections and remap |
| Message provider says delivered | Assume patient understands | Follow acknowledgment/comprehension policy |
| Proxy revoked before reminder | Use cached approval | Revalidate and stop send |
| Medication lists conflict | Choose most recent as correct | Create contradiction and clinical handoff |
| Safety queue unavailable | Continue routine workflow | Use approved backup route and alert operations |

## Workflow readiness checklist

- [ ] Each workflow starts from a versioned authoritative clinical or administrative source.
- [ ] Clinical intent, priority, coding, and treatment fields have named human/source owners.
- [ ] Scheduling distinguishes availability, hold, booking, attendance, cancellation, and encounter.
- [ ] Referrals track submission, acceptance, scheduling exceptions, follow-up, and closure evidence.
- [ ] Prior authorization preserves payer/profile versions and clinician attestation.
- [ ] Care-plan tasks invalidate on source changes.
- [ ] Medication and diagnosis records are evidence-only with provenance.
- [ ] Communication distinguishes request, dispatch, delivery, acknowledgment, and comprehension.
- [ ] Every handoff has a receiver, due time, acknowledgment, and fallback.
- [ ] Concerning evidence interrupts routine automation without agent disposition.

## Related guides

- Previous: [Patient Identity, Consent, Proxy, and Care-Team Authority](03-patient-identity-consent-proxy-and-care-team-authority.md)
- Next: [State, Events, Context, Memory, and Planning](05-state-events-context-memory-and-planning.md)
- Home: [Healthcare Care Coordination Agent](README.md)
- Canonical: [Tool Results, Artifacts, and Provenance](../../tools/tool-results-artifacts-and-provenance.md)

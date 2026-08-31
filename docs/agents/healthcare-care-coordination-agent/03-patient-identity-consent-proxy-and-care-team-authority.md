# Patient Identity, Consent, Proxy, and Care-Team Authority

Healthcare coordination begins with four different questions: who is acting, which patient record is in scope, what may be done for what purpose, and who owns the resulting work. Do not collapse them into a login or a “consent=true” field.

## Separate the identities

| Identity | Example | Authority it establishes | Authority it does not establish |
|---|---|---|---|
| Channel/session | Portal session, phone interaction, staff workstation | Which session sent a request | Legal identity, patient binding, or representative scope |
| Actor | Authenticated patient, representative, workforce member, service principal | Who is acting at an assurance level | Which patient record or action is allowed |
| Subject/patient | Namespace-qualified EHR patient record | Which clinical record is referenced | That the actor may access or act |
| Representative | Guardian, personal representative, caregiver, delegate | A claimed or recorded relationship | Current authority for every purpose/data/action |
| Workforce role | Scheduler, nurse, clinician, privacy officer | Organizational role and context | Blanket access to all records |
| Service identity | Adapter or backend service | Which software principal calls an API | End-user purpose, patient authority, or clinical intent |

Every access and effect binds all applicable dimensions. A correct user attached to the wrong patient remains a safety failure.

## Patient-binding workflow

~~~mermaid
stateDiagram-v2
    [*] --> Unbound
    Unbound --> Candidate: authenticated request and demographics
    Candidate --> Bound: deterministic exact binding or MPI/steward resolution
    Candidate --> Review: multiple or low-assurance candidates
    Review --> Bound: authorized resolution
    Review --> Rejected: no defensible match
    Bound --> Review: merge/split, demographic conflict, stale assurance
    Bound --> Invalidated: source says binding is wrong
    Invalidated --> Review: correction workflow
    Rejected --> [*]
~~~

FHIR Patient match can return candidate records and scores, but algorithms, required fields, and thresholds are implementation-specific. IHE PIXm can cross-reference known identifiers but is not a golden-record or chart-merge service. The model may describe candidate differences; it cannot select an ambiguous candidate, merge charts, or override the established master-patient-index process.

### Match decision table

| Condition | Automated coordination | Required route |
|---|---|---|
| Authenticated portal is already bound to a stable, non-invalidated patient record | Continue under freshness policy | Audit the binding source and version |
| Exact enterprise identifier resolves once in the expected tenant | Continue if actor authority also passes | Recheck pre-effect when required |
| Multiple candidates, low confidence, conflicting demographics, or namespace mismatch | Stop | Identity-steward/MPI queue |
| Merge/split or correction event affects the record | Freeze affected cases/effects | Rebind and review prior disclosures/effects |
| Patient disputes the record | Stop and preserve evidence | Organization correction and identity process |
| Match service unavailable | Do not guess | Manual verified path |

Names, dates of birth, phone numbers, addresses, and similarity scores are sensitive evidence—not proof by themselves. Store only the minimum needed for the approved identity process.

## Patient-binding contract

~~~yaml
patient_binding:
  binding_id: opaque
  tenant_id: opaque
  subject_namespace: ehr-organization-a
  subject_identifier: opaque
  source_record_ref: Patient/opaque/_history/version
  assurance:
    method: authenticated-portal | enterprise-id | mpi-resolved | staff-verified
    organization_level: named-policy-level
    verified_at: timestamp
    verifier_ref: opaque
  match_case_ref: optional
  status: stable | review | invalidated
  invalidation_reason: optional
  version: integer
  expires_at: optional
~~~

Never place raw patient identifiers into idempotency keys, logs, URLs, metrics, or trace baggage. Use opaque tenant-scoped references.

## Consent and authorization are policy evidence

FHIR Consent can represent patient choices, but FHIR explicitly leaves enforcement to the surrounding access-control system. In US contexts, HIPAA “consent,” authorization, treatment/payment/operations permissions, personal-representative authority, and other laws are not interchangeable. Other jurisdictions use different concepts.

The application therefore evaluates a versioned policy decision at point of use:

~~~yaml
authority_decision:
  decision_id: opaque
  actor_id: opaque
  patient_binding_id: opaque
  tenant_id: opaque
  relationship: patient | representative | caregiver | workforce | service
  purpose: care-coordination
  action: view-referral-status
  data_classes: [referral-status]
  destination: patient-verified-channel
  decision: permit | deny | step-up | human-review
  supporting_grants: [opaque]
  restrictions: []
  policy_version: opaque
  evidence_versions: [opaque]
  evaluated_at: timestamp
  expires_at: timestamp
~~~

The effect service consumes this record; the model never receives permission to reinterpret it.

## Personal representative and caregiver authority

A relationship record is not sufficient. Represent authority with:

- source and verification method;
- relationship and applicable legal/organizational basis;
- patient and actor identifiers;
- purposes;
- allowed actions;
- allowed data classes;
- allowed destinations/channels;
- explicit exclusions;
- effective period;
- restrictions and exceptions;
- revocation status;
- policy and evidence versions;
- point-of-use check time.

### Proxy decision table

| Scenario | Safe system behavior |
|---|---|
| Patient acts for self and current binding/authorization passes | Allow only the approved purpose, action, and fields |
| Verified representative has broad current authority | Still minimize fields and check destination/channel |
| Caregiver is listed as a contact but no action scope exists | Do not disclose or act; route for verification |
| Proxy may schedule but not view sensitive details | Allow scheduling metadata only; filter payload and logs |
| Authority expired, was revoked, or conflicts across sources | Stop and route; do not choose the convenient record |
| Minor or dependent patient has multiple authority records | Apply jurisdiction/organization policy; route ambiguity |
| Patient requested confidential communications | Enforce channel/address restriction before any outreach |
| Abuse, neglect, endangerment, or other exception may apply | Stop automatic disclosure; route to designated expert process |
| Patient's decision-making capacity is questioned | Model makes no capacity judgment; route to accountable professionals |

Do not copy US personal-representative rules into other jurisdictions. Legal and privacy owners must configure policy profiles, and clinical/safeguarding services must define exception routes.

## Point-of-use revalidation

Revalidate before:

- opening a sensitive source;
- assembling model context;
- showing a result after a long wait;
- sending a communication;
- submitting an authorization;
- booking, canceling, or rescheduling;
- handing work to another organization;
- resuming after compaction, pause, deploy, or human wait;
- retrying or reconciling an effect;
- exporting audit or case data.

Revalidation prevents a cached “permit” from surviving proxy revocation, patient-record correction, role change, care-team reassignment, changed purpose, or policy release.

## Purpose and data minimization

Use an allowlist generated from the workflow step, not a broad resource export.

| Step | Typical minimum input | Data commonly unnecessary |
|---|---|---|
| Offer appointment | Eligible service, location, availability, logistical preferences | Full note, complete diagnosis list, medication list |
| Referral-status update | Referral identifier, destination, status, next administrative step | Unrelated encounters and results |
| Prior-auth gap check | Payer requirement, clinician-attested source fields, requested service | Entire longitudinal chart |
| Reminder | Minimal organization/contact/appointment metadata allowed by channel policy | Sensitive condition or referral reason |
| Care-team handoff | Patient binding ref, clinical source refs, requested action, deadline, safety flag | Model transcript and unrelated records |

Even when a legal minimum-necessary rule has an exception, the engineering default should remain purpose-bound minimization unless local governance explicitly requires more.

## Special data-policy profiles

Do not encode privacy law in prompts. Use versioned policies for:

- jurisdiction and organizational entity;
- treatment, operations, payment, research, public-health, and other purposes;
- substance-use disorder and specially protected records;
- minors and dependent adults;
- reproductive, behavioral, genetic, or other locally sensitive categories;
- patient restrictions and confidential-channel requests;
- segmentation/security labels and unknown-label behavior;
- cross-organization/HIE exchange purpose;
- emergency or break-glass access, if applicable;
- retention, deletion, correction, export, and legal hold.

FHIR security labels can carry handling signals, but their meaning depends on mutual trust, policy, and local agreements. Unknown or unsupported labels must not be silently ignored.

## Workforce and care-team authority

Care-team membership and clinical responsibility change over time. A safe ownership record includes:

~~~yaml
care_team_assignment:
  assignment_id: opaque
  case_id: opaque
  organization_id: opaque
  service_id: opaque
  practitioner_role_ref: optional
  queue_ref: opaque
  responsibility: administrative | clinical-review | safety-assessment
  effective_period: {start: timestamp, end: optional}
  coverage_policy_ref: opaque
  source_ref: opaque-version
  status: proposed | active | transferred | ended
  accepted_at: optional
  accepted_by: optional
~~~

Rules:

- Assign to roles/queues with coverage, then capture the individual who accepts.
- Separate administrative owner, clinical owner, privacy decision-maker, and identity steward.
- A directory lookup may suggest a receiver; acceptance proves ownership.
- Handoffs require sender, receiver, evidence, action, deadline, acknowledgment, and fallback.
- Role termination or coverage changes generate invalidation events.
- Service principals act only on behalf of an evaluated actor/purpose or a narrowly preauthorized backend workflow.

## Merge, split, and correction handling

Patient-identity corrections can invalidate more than a foreign key.

When a merge/split/correction event arrives:

1. quarantine affected open cases;
2. stop undispatched effects;
3. mark dispatched-but-unverified effects for reconciliation;
4. invalidate cached authority decisions and contexts;
5. identify prior communications and disclosures for review;
6. rebind clinical evidence with original lineage preserved;
7. require the established identity process to authorize continuation;
8. record the correction without rewriting the historical audit trail.

Do not automatically rewrite historical source references to the surviving record. Preserve what the system knew and acted on at the time.

## Identity and authority failure tests

Inject at least:

- two patients with the same name and birth date;
- transposed demographic fields;
- stale portal-to-patient binding;
- record merge after approval but before effect;
- record split after a communication;
- representative with schedule-only scope requesting diagnosis details;
- revoked proxy during a paused workflow;
- caregiver present in contact list but absent from authority records;
- staff member moving organizations or roles;
- service principal with broad system scope but no valid case purpose;
- security label unknown to the receiving adapter;
- conflicting consent records from two systems;
- confidential-channel preference changed before send;
- patient disputes identity after case closure.

Every case must fail closed, route visibly, preserve evidence, and avoid model resolution of legal or identity truth.

## Identity and authority readiness checklist

- [ ] Actor identity, patient binding, representative authority, and workforce role are separate records.
- [ ] Patient identifiers are namespace-qualified and opaque outside the identity boundary.
- [ ] Ambiguous matches never reach model-driven effects.
- [ ] Merge/split/correction events invalidate dependent state.
- [ ] Consent resources feed a policy engine rather than acting as a permit flag.
- [ ] Authority grants are scoped by purpose, action, data, destination, period, and restriction.
- [ ] Point-of-use revalidation occurs before reads, context assembly, effects, and resume.
- [ ] Care-team owners have coverage, acceptance, and transfer semantics.
- [ ] Special data policies and unknown-label behavior are configured.
- [ ] Audit distinguishes requester, subject, representative, workforce user, service principal, and approver.

## Related guides

- Previous: [Reference Architecture, Runtime, and Integration Decisions](02-reference-architecture-runtime-and-integration-decisions.md)
- Next: [Scheduling, Referrals, Prior Authorization, Tasks, and Handoffs](04-scheduling-referrals-prior-authorization-tasks-and-handoffs.md)
- Home: [Healthcare Care Coordination Agent](README.md)
- Evidence: [Healthcare Care Coordination Agent Blueprint](../../research/packets/healthcare-care-coordination-agent-blueprint.md)


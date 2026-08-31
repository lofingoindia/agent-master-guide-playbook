# Healthcare Care Coordination Agent Blueprint Research Packet

**Category:** 41 — Healthcare clinical administration and care coordination  
**Research date:** 2026-08-31  
**Status:** Research-backed Pass 2 refinement  
**Companion guide:** [Healthcare Care Coordination Agent](../../agents/healthcare-care-coordination-agent/README.md)

> This packet supports a software architecture, not medical or legal advice. A deploying organization must obtain clinical-safety, privacy, security, accessibility, records-management, and legal review for its jurisdictions, population, intended use, and actual integrations.

## Research question

What is the smallest production-worthy agent architecture that can coordinate patient-facing administrative work without turning a language model into a patient-identity authority, consent engine, clinician, emergency dispatcher, or clinical system of record?

The researched workload includes:

- patient identity binding and matching escalation;
- consent, personal representative, caregiver, and proxy authority;
- scheduling, referral, prior-authorization, and care-plan task tracking;
- protected health information (PHI) handling;
- EHR, FHIR, HIE, payer, directory, scheduling, and communication integration;
- clinician and care-team ownership;
- outreach, closed-loop handoff, and safety/deterioration escalation;
- durable execution, audit, reconciliation, capacity, and governed change.

It excludes autonomous diagnosis, treatment selection, prescribing, dose changes, emergency-triage disposition, clinical urgency assignment, and replacement of professional judgment.

## Research method

Research prioritized current primary material: HL7 FHIR and implementation guides, IHE profiles, US health-policy and privacy authorities, NIST publications, NHS clinical-safety standards, WHO guidance, and standards bodies. Older sources were retained only when still authoritative or when newer sources explicitly continue to rely on them.

Each source was checked for:

- publication or update status as of the research date;
- normative, trial-use, guidance, or jurisdiction-specific status;
- the exact architectural claim it can support;
- limits that prevent treating the source as a universal implementation prescription.

Pass 2 re-ran targeted searches against current official publications for FHIR identity/version/time semantics, Patient match/merge, PDQm/PIXm, C-CDA, directory profiles, CRD/DTR/PAS, CMS rule/FAQ status, X12 278, secure messaging, digital signatures, accessibility, safety and conformance testing. It then traced each finding to an operation contract, state invariant, failure injection, human fallback or release gate. Product marketing pages and secondary summaries were not used to establish normative behavior.

The repository's [cross-cutting control packet](agent-blueprint-cross-cutting-controls.md) supplied the common D0–D4 authority model. Healthcare research was used to make patient binding, consent, clinical ownership, safety escalation, interoperability, and communication controls domain-specific.

## Promotion decision

This category qualifies as a real agent blueprint, but only inside a narrow authority ceiling.

| Test | Finding | Decision |
|---|---|---|
| Repeated ambiguity | Referral notes, payer requirements, incomplete handoffs, and patient messages often require evidence-grounded interpretation | Use a bounded model worker |
| Durable multi-step work | Scheduling, referrals, authorizations, and outreach cross systems and days | Use a durable workflow and local effect ledger |
| Consequential effects | Wrong-patient disclosure, missed follow-up, or altered appointment can cause harm | Keep effects behind deterministic policy and D2/D3 gates |
| Clinical judgment | Diagnosis, treatment, urgency, and deterioration disposition require accountable clinical ownership | Prohibit model ownership; escalate |
| Deterministic alternative | Fixed forms, validation, routing tables, and status checks solve many cases | Start with rules; invoke a model only for unresolved ambiguity |
| Distinctness | The workload combines clinical privacy, patient identity, consent, care-team ownership, and safety escalation | Keep separate from support, document extraction, trials, claims, and finance |

## Executive findings

1. **The safest useful architecture is a workflow system with one bounded model worker.** The application, not the model, owns identity, authorization, policy, effects, state, audit, and release controls.
2. **A patient match is a candidate, not patient truth.** FHIR Patient match algorithms and thresholds are implementation-specific. Ambiguous bindings go to the organization's master-patient-index or identity-steward process; the model never merges records.
3. **Consent is not enforcement and proxy authority is not a Boolean.** FHIR Consent represents choices but explicitly does not define enforcement. Applicable law, organizational policy, purpose, data class, relationship, effective period, revocation, and exceptions must be evaluated at point of use.
4. **FHIR is an interoperability contract, not the coordinator's execution engine.** FHIR Task, ServiceRequest, CarePlan, Appointment, and Communication resources can exchange state; a local durable workflow must still own retries, leases, deadlines, effects, and reconciliation.
5. **Availability is not a booking.** FHIR Slot explicitly cannot guarantee successful appointment creation. Booking requires a fresh read, conditional or vendor-supported commit where available, receipt capture, and postcondition verification.
6. **Cross-system exactly-once effects do not exist as a general guarantee.** FHIR transactions are atomic only within one supporting server. Scheduling, payer, HIE, messaging, and local state require semantic effect IDs, idempotency controls, unknown-outcome handling, and reconciliation.
7. **Referral closure is a safety property, not just a status label.** The 2025 SAFER clinician-communication guide calls for tracking referrals and notifying the referring provider when expected scheduling or attendance does not occur.
8. **Medication and diagnosis data are evidence, not instructions.** The agent may cite versioned, provenance-bearing source records and surface discrepancies. It must not clinically reconcile them, infer a diagnosis, select treatment, or change medication.
9. **Communications require both authority and delivery truth.** A communication request is not proof of transmission, and transmission is not proof of comprehension. Channel preference, confidentiality constraints, proxy scope, delivery receipt, accessibility, language support, and escalation must remain separate facts.
10. **Clinical safety needs an explicit hazard process.** SAFER guidance and the NHS DCB0129/DCB0160 approach support hazard logs, safety cases, accountable review, and deployment-specific risk management. These techniques do not make either NHS standard universally applicable.
11. **A product label does not settle regulatory scope.** The 2026 FDA clinical-decision-support guidance makes intended use and actual functionality material. Any drift toward patient-specific clinical recommendation requires formal regulatory and clinical review.
12. **Current profiles must be pinned, not described as “FHIR compliant.”** US Core 9.0.0 targets FHIR R4 while core FHIR R5 exists; Da Vinci guides remain versioned and commonly trial-use. CapabilityStatement, profiles, extensions, terminology versions, and conformance tests belong in each adapter manifest.
13. **Patient-level hidden “memory” is an avoidable risk.** Durable operational facts belong in typed stores with provenance and retention policy. Learning should use governed, de-identified failure corpora, not opaque recollection of prior patient interactions.
14. **Production quality is measured by bounded harm and recoverability.** Release gates must fail on wrong-patient effects, unauthorized disclosure, missed safety escalation, invented clinical evidence, duplicate consequential work, or unreconciled unknown outcomes even when average task completion is high.
15. **Identity, version, and time are multidimensional.** FHIR logical identity, business identifier, record version, business version, clinical/effective time, recorded time, retrieval time, and local aggregate version are not interchangeable. In particular, FHIR version IDs are opaque and `meta.lastUpdated` is not clinical time.
16. **Protocols divide responsibility; they do not erase it.** FHIR resources, HL7 v2 events, CDA/C-CDA documents, IHE transactions, X12 transactions, and terminology packages each require their own identity, acknowledgment, correction, and version mapping. A transport acknowledgment is not a closed referral, accepted handoff, final prior-auth decision, or understood communication.
17. **Integration qualification is operation-, tenant-, and contract-specific.** An endpoint-level “supported” claim cannot establish write idempotency, timeout boundaries, status meanings, patient/subject correctness, postconditions, or reconciliation. Each read/search/write/send/status/cancel operation needs executable positive, negative, race, timeout, and recovery evidence.
18. **Safe compaction is loss-aware reconstruction.** A continuation receipt must pin behavior/policy/adapter/profile/terminology versions, source-event watermark, approvals, active clocks, pending/unknown effects, omitted-item references, invariant hash, and next safe action. Resume fails closed if these cannot be revalidated.

## Selected architecture

~~~mermaid
flowchart LR
    U[Patient, representative, staff] --> C[Channel and identity assurance]
    C --> P[Policy and authority service]
    P --> W[Durable coordination workflow]
    W --> M[Bounded model worker]
    W --> H[Human work queues]
    W --> E[Effect ledger and outbox]
    M -->|proposals with evidence refs| W
    E --> A[Typed adapters]
    A --> S1[EHR and FHIR]
    A --> S2[HIE and MPI]
    A --> S3[Scheduling and directory]
    A --> S4[Payer and prior authorization]
    A --> S5[Approved communications]
    S1 --> R[Receipts and reconciliation]
    S2 --> R
    S3 --> R
    S4 --> R
    S5 --> R
    R --> W
    W --> L[Append-only audit and safety telemetry]
~~~

The local workflow ledger is authoritative for case progress, approvals, effect attempts, receipts, and reconciliation. The EHR, HIE/MPI, payer, scheduling, directory, and communication systems remain authoritative for their domain records. The agent is never the clinical record.

## Authority decision record

| Decision | Selected rule | Evidence | Important limit |
|---|---|---|---|
| Patient matching | Model may request or explain candidates; only the established MPI/human identity process may resolve ambiguity or merge | [FHIR Patient match](https://hl7.org/fhir/patient-operation-match.html), [ONC patient identity](https://healthit.gov/standards-and-technology/patient-identity-and-patient-record-matching/), [GAO-19-197](https://www.gao.gov/products/gao-19-197) | Match inputs and scores are implementation-specific |
| Consent | Treat Consent as evidence consumed by a policy decision point, never as the enforcement mechanism | [FHIR Consent](https://hl7.org/fhir/consent.html), [HHS Privacy Rule](https://www.hhs.gov/hipaa/for-professionals/privacy/laws-regulations/index.html) | Applicable requirements vary by jurisdiction, purpose, and data |
| Personal representatives | Represent authority with source, scope, period, exceptions, and verification | [HHS personal representatives](https://www.hhs.gov/hipaa/for-professionals/privacy/guidance/personal-representatives/index.html) | State law and circumstances determine authority |
| Access minimization | Evaluate purpose and disclose only the needed data fields even where a legal minimum-necessary exception may exist | [HHS minimum necessary](https://www.hhs.gov/hipaa/for-professionals/privacy/guidance/minimum-necessary-requirement/index.html) | This is an engineering default, not a substitute for legal analysis |
| Authentication | Keep workforce identity, patient proofing, authenticator strength, and federation assurance separate | [NIST SP 800-63-4](https://pages.nist.gov/800-63-4/) | An NIST assurance level does not itself prove patient-record matching |
| App authorization | Use SMART discovery and narrow user or system scopes; backend services need explicit preauthorization | [SMART App Launch 2.2](https://hl7.org/fhir/smart-app-launch/), [backend services](https://hl7.org/fhir/smart-app-launch/backend-services.html) | SMART scopes do not replace local consent and purpose policy |
| Referral request | Preserve clinician-authored ServiceRequest intent; agent checks administrative completeness and progress | [FHIR ServiceRequest](https://hl7.org/fhir/servicerequest.html) | The agent must not invent clinical intent, reason, priority, or order |
| Care-plan tasks | Project operational tasks from an authoritative plan, with source/version links | [FHIR CarePlan](https://hl7.org/fhir/careplan.html), [FHIR Task](https://hl7.org/fhir/task.html) | Task is trial use and does not replace local workflow guarantees |
| Scheduling | Treat slots as observations; booking is a separately authorized effect | [FHIR Slot](https://hl7.org/fhir/slot.html), [FHIR Appointment](https://hl7.org/fhir/appointment.html) | Free/busy state does not guarantee booking |
| Communication | Separate request, send attempt, delivery, and acknowledged comprehension | [FHIR CommunicationRequest](https://hl7.org/fhir/communicationrequest.html), [FHIR Communication](https://hl7.org/fhir/communication.html), [HHS appointment reminders](https://www.hhs.gov/hipaa/for-professionals/faq/198/may-health-care-providers-leave-messages/index.html) | Resource or channel status is not clinical understanding |
| Closed-loop follow-up | Set accountable owner, due time, exception route, and proof of closure | [2025 SAFER clinician communication](https://healthit.gov/wp-content/uploads/2025/06/SAFER-Guide-1.-Clinical-Communication-Final.pdf) | Organizations define appropriate timeframes and escalation |
| Prior authorization | Use versioned CRD/DTR/PAS contracts; require source-backed fields and human confirmation of clinical content | [Da Vinci CRD 2.2.1](https://hl7.org/fhir/us/davinci-crd/), [DTR 2.2.0](https://hl7.org/fhir/us/davinci-dtr/), [PAS 2.2.1](https://hl7.org/fhir/us/davinci-pas/) | Guides are versioned and often trial-use; payer support varies |
| Regulatory dates | Configure payer and jurisdiction profiles; do not hard-code one “CMS deadline” | [CMS-0057-F](https://www.cms.gov/initiatives/burden-reduction/overview/interoperability/policies-regulations/cms-interoperability-prior-authorization-final-rule-cms-0057-f) | Applicability, exclusions, and dates differ |
| FHIR concurrency | Use ETags, conditional operations, and server transactions when supported, then reconcile postconditions | [FHIR HTTP](https://hl7.org/fhir/http.html) | Atomicity does not span unrelated systems |
| Audit | Use an application ledger plus standard Provenance/AuditEvent mappings where supported | [FHIR Provenance](https://hl7.org/fhir/provenance.html), [FHIR AuditEvent](https://hl7.org/fhir/auditevent.html), [IHE BALP](https://profiles.ihe.net/ITI/BALP/) | Telemetry is not a complete legal or effect ledger |
| HIE exchange | Treat TEFCA/IHE connections as governed trust frameworks with purpose and identity requirements | [TEFCA](https://healthit.gov/policy/tefca/), [IHE PIXm](https://profiles.ihe.net/ITI/PIXm/) | Participation does not grant blanket access or resolve identity truth |
| Clinical safety | Maintain a hazard log, safety case, safety owner, and deployment-specific controls | [2025 SAFER guides](https://healthit.gov/clinical-quality-and-safety/safer-guides/), [NHS DCB0129](https://digital.nhs.uk/data-and-information/information-standards/governance/latest-activity/standards-and-collections/dcb0129-clinical-risk-management-its-application-in-the-manufacture-of-health-it-systems/), [NHS DCB0160](https://digital.nhs.uk/data-and-information/information-standards/governance/latest-activity/standards-and-collections/dcb0160-clinical-risk-management-its-application-in-the-deployment-and-use-of-health-it-systems) | NHS standards are jurisdiction-specific and under review |
| Accessibility | Set a tested accessibility target and retain alternate/human channels | [WCAG 2.2](https://www.w3.org/TR/WCAG22/), [HHS Section 1557 disability fact sheet](https://www.hhs.gov/civil-rights/for-individuals/section-1557/fs-disability/index.html) | Technical conformance alone does not guarantee effective communication |
| AI governance | Preserve human control, transparency, equity testing, monitoring, and shutdown paths | [WHO AI for health](https://www.who.int/publications/i/item/9789240037403), [NIST AI 600-1](https://doi.org/10.6028/NIST.AI.600-1) | General guidance must be translated into workflow-specific hazards |
| Product scope | Trigger formal review if intended use or behavior becomes clinical decision support | [FDA CDS guidance, January 2026](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/clinical-decision-support-software) | Classification is fact- and jurisdiction-specific |

## Interoperability profile

### Contract-family boundary

| Family | Primary contribution | Boundary established by research |
|---|---|---|
| FHIR | REST/resource identity, profiles, versions, workflow data and conditional operations | Resource exchange and single-server transaction semantics; not global patient identity, local execution, consent enforcement or cross-system exactly once |
| HL7 v2 | Event-driven ADT/order/scheduling/result interfaces in deployed organizations | Local interface profile and ACK semantics must be pinned; merge/move/change/cancel events invalidate projections but do not authorize the coordinator to resolve identity |
| CDA/C-CDA | Persistent whole clinical documents with human-readable context and document provenance | Validate exact templates/bytes/identifier/version/author/attester; extraction is a new derived artifact and structural validity is not clinical correctness |
| IHE | Actor/transaction profiles for identity, query, audit, documents and exchange | Options, identifier domains, participant agreement and trial-implementation version remain deployment-specific |
| X12 | Eligibility and health-care-services-review transactions and operating guides | Exact TR3, companion guide, control/trace identifiers and response correlation are required; licensing and payer variation prevent generic field assumptions |
| Terminology | Stable code-system and value-set artifacts for computable meaning | Pin system/edition/version and expansion/mapping; never infer a clinical/billable code or treat display text as semantic identity |

Current examples reinforce the need for separate pins: core FHIR R5 is 5.0.0; US Core 9.0.0 and Da Vinci PAS 2.2.1 are R4 trial-use guides; IHE PDQm 3.2.0 and PIXm 3.1.0 are R4 trial-implementation publications; C-CDA 5.0.0 remains CDA R2-based despite being published with FHIR tooling. These examples are research anchors, not a universal deployment bundle.

### Resource mapping

| Local concept | Candidate FHIR/IHE representation | Architectural rule |
|---|---|---|
| Patient binding | Patient, Person, Patient match, PIXm/PDQm | Store local binding plus source namespace and version; never infer equivalence from similar demographics |
| Representative | RelatedPerson plus local authority grant | RelatedPerson alone does not prove current legal authority |
| Consent evidence | Consent, security labels, policy references | Evaluate with policy service; enforcement is local |
| Care-team ownership | CareTeam, Practitioner, PractitionerRole, Organization | Pin organization, role, effective period, coverage, and escalation queue |
| Clinical intent | ServiceRequest, CarePlan, Goal | Read-only evidence unless an accountable clinician creates or attests it |
| Work item | Task plus local coordination task | Map statuses explicitly; do not assume semantic equivalence |
| Scheduling | Schedule, Slot, Appointment | Separate availability, hold, booking, attendance, and cancellation |
| Outreach | CommunicationRequest, Communication | Preserve channel, audience, minimum payload, delivery receipts, and accessibility |
| Coverage/prior auth | Coverage plus CRD/DTR/PAS artifacts | Keep clinical attestation and payer response provenance |
| Evidence lineage | Provenance, source resource version, DocumentReference | Every extracted or summarized assertion cites source and version |
| Audit | AuditEvent/BALP plus application ledger | Retain policy decision, actor, purpose, action, target, outcome, and correlation |

### Compatibility finding

Core FHIR R5 is not a deployment assumption. As of the research date, [US Core 9.0.0](https://hl7.org/fhir/us/core/STU9/) is a 2026 trial-use implementation guide based on FHIR R4, while Da Vinci guides have their own releases. Every connector therefore needs:

- endpoint and tenant;
- FHIR release;
- CapabilityStatement snapshot;
- accepted profiles and extensions;
- search and operation support;
- terminology package versions;
- authentication and scope behavior;
- conditional-write, transaction, pagination, and rate-limit support;
- conformance fixtures and known deviations;
- migration and rollback plan.

### Record identity and time finding

FHIR distinguishes a resource's server-scoped logical URL from business identifiers and record versions. A past version can be referenced through `/_history/{vid}`, but servers need not retain history and clients cannot order opaque `meta.versionId` values lexically. Some resources add their own business version: for example, DocumentReference has a business `version` distinct from its FHIR record version.

The production normalization contract therefore records:

~~~yaml
normalized_source_record:
  tenant_and_endpoint_ref: opaque
  protocol_and_profile_versions: [opaque]
  resource_identity: {type: opaque, logical_id: opaque, record_version: opaque}
  business_identifiers: [{system: opaque, value_ref: opaque, assigner: optional, period: optional}]
  business_version: optional
  clinical_or_effective_time: optional
  recorded_at_source: optional
  observed_at_adapter: timestamp
  source_status: opaque | unknown
  terminology_and_mapping_versions: [opaque]
  content_hash: optional
  correction_or_replacement_refs: []
~~~

The adapter never substitutes `lastUpdated` for onset, order occurrence, coverage period, consent/proxy authority period, role coverage, appointment time, document service time, or send/receive time. Missing time/version/status stays unknown and is evaluated under the operation's freshness policy.

## Core data contracts

These are design-level contracts. The implementation language and database may vary.

### Patient binding

~~~yaml
patient_binding:
  binding_id: opaque
  case_id: opaque
  subject_namespace: ehr-tenant-a
  subject_identifier: opaque
  fhir_reference: Patient/opaque
  assurance:
    method: authenticated-portal | mpi-resolved | staff-verified
    level: organization-defined
    verified_at: timestamp
    verifier_ref: opaque
  candidate_match_ref: optional
  merge_split_status: stable | review | invalidated
  version: integer
  expires_at: optional
~~~

### Authority grant

~~~yaml
authority_grant:
  grant_id: opaque
  subject_binding_id: opaque
  actor_id: opaque
  relationship: patient | personal-representative | caregiver | workforce
  source_ref: authoritative-record
  purposes: [care-coordination]
  allowed_actions: [read-status, propose-time, confirm-booking]
  allowed_data_classes: [appointment-metadata]
  excluded_data_classes: []
  effective_period: {start: timestamp, end: optional}
  restrictions: [confidential-channel-only]
  verification_status: verified | pending | denied | revoked
  policy_version: opaque
  checked_at: timestamp
~~~

### Coordination case

~~~yaml
coordination_case:
  case_id: opaque
  tenant_id: opaque
  patient_binding_id: opaque
  workflow_kind: referral | scheduling | prior-authorization | care-task | outreach
  clinical_source_refs: [versioned-reference]
  accountable_owner: {type: care-team | clinician | operations, id: opaque}
  administrative_priority: organization-defined
  clinical_priority_source_ref: optional
  state: intake | blocked | ready | executing | waiting | safety-hold | reconciling | closed
  open_tasks: [opaque]
  unresolved_contradictions: [opaque]
  safety_flags: [opaque]
  case_version: integer
  behavior_version: opaque
  retention_class: opaque
~~~

### Effect record

~~~yaml
effect:
  effect_id: semantic-idempotency-key
  case_id: opaque
  action: book-appointment
  normalized_parameters_hash: opaque
  authority_decision_ref: opaque
  approval_ref: optional
  adapter_version: opaque
  status: proposed | authorized | dispatched | succeeded | failed | unknown | reconciled
  attempts: integer
  external_request_ids: [opaque]
  receipts: [opaque]
  postcondition: expected-state
  reconciliation_due_at: optional
~~~

### Safety escalation

~~~yaml
safety_escalation:
  escalation_id: opaque
  case_id: opaque
  trigger_type: deterministic-rule | model-uncertainty | patient-statement | overdue-critical-work
  trigger_evidence_refs: [versioned-reference]
  disposition: not-set-by-agent
  destination_queue: organization-approved
  accountable_receiver: optional
  created_at: timestamp
  acknowledged_at: optional
  closed_by: optional
  closure_evidence_ref: optional
~~~

## Bounded runtime loop

The model receives a minimal, policy-filtered context and may emit only closed-set proposals.

~~~text
load typed case state and fresh authority decision
if patient binding is invalid, authority is insufficient, or safety hold exists:
    route to the approved human queue and stop effects
assemble bounded evidence with source versions and contradiction markers
if deterministic workflow can choose the next action:
    execute that transition without a model
else:
    ask the model for one typed administrative proposal
validate schema, evidence references, authority, clinical boundary, and budget
if proposal is invalid, clinical, unsupported, or uncertain:
    escalate or request missing evidence
else:
    persist proposal and create an effect only at its authorized D-tier
verify receipt and postcondition; reconcile any unknown outcome
checkpoint a typed continuation package; repeat within step/time/tool budgets
~~~

## Context and memory findings

The canonical seven-lifetime table lives in the [state guide](../../agents/healthcare-care-coordination-agent/05-state-events-context-memory-and-planning.md#canonical-seven-lifetime-memory-policy). Research supports four cross-lifetime controls:

| Control | Finding |
|---|---|
| Truth promotion | Scratch, channel, retrieval and model output cannot become coordination or clinical truth without a validated source/event and authorized writer |
| Retention/correction | Every store has class-specific TTL/retention, deletion/legal-hold behavior and correction propagation, including indexes, caches, backups and provider-held artifacts |
| Poisoning resistance | Tenant/patient filters, provenance, signed/approved publishing, untrusted-content isolation, contradiction handling and rollback apply before retrieval, not in prompt text alone |
| Evaluation | Cross-patient leakage, stale/revoked state, injection, replay/restart, deletion/correction, slice bias and train/evaluation contamination are tested per lifetime |

Provider-hidden sessions and raw patient recollection are rejected continuity mechanisms, not extra memory lifetimes. Continuity must survive model/provider replacement through typed checkpoints.

A compaction receipt must include a concrete receipt schema version, tenant/case/binding versions, source-event high watermark, behavior/workflow/policy/adapter/profile/terminology/template/safety pins, source references, open task versions and owners, approvals and expiry, active due/recheck/handoff/safety/reconciliation clocks, pending and unknown effects, contradictions, safety flags, handoffs, immutable omitted-item references, remaining budgets, invariant hash/signature and one next safe action with preconditions. A summary is a navigation aid, never a source of truth.

Resume verifies the receipt signature and invariant hash, fences the case lease, replays events after the watermark, reloads authoritative state, resolves version pins, revalidates identity/authority/source/effective time, fires overdue clocks without resetting them, reconciles unknown effects and recompiles minimum context. A gap, mismatch, unavailable omitted item or unresolved behavior migration creates a visible hold; it never falls back to the model's recollection.

## Contradictions and reconciliations

| Apparent contradiction | Resolution |
|---|---|
| FHIR R5 is current, but common national profiles use R4 | Pin the deployed server and implementation-guide versions; normalize behind adapters |
| FHIR Consent records consent, but applications need enforcement | Use Consent as evidence; a policy decision and enforcement point owns access |
| FHIR Task represents workflow, but durable execution is needed | Exchange Task state while the local workflow engine owns leases, retries, effects, and reconciliation |
| A match operation returns scores, but coordinators need one patient | Candidate scores trigger established identity resolution; they never authorize automatic merge |
| Slot says free, but the patient expects an appointment | Only a verified booking receipt and postcondition establish the appointment |
| A FHIR transaction is atomic | Only within one supporting server; cross-system work remains a saga |
| HIPAA has a treatment exception to minimum necessary | Keep legal applicability separate from the safer engineering default of purpose-bound minimization |
| General HIPAA policy covers PHI | Substance-use records, state law, minors, reproductive-health rules, and other regimes may add constraints; use versioned policy profiles |
| A communication was sent, so the handoff is complete | Delivery, receipt, comprehension, clinical acceptance, and ownership are separate |
| The agent can detect safety language, so it can triage | It may conservatively trigger an approved escalation; it may not choose clinical or emergency disposition |
| “FHIR compliant” implies compatible | Test exact profiles, operations, extensions, terminology, scopes, and vendor deviations |
| Automation improves access | It can also exclude people; retain accessible, language-appropriate, and human alternatives |
| DCB0129/DCB0160 offer a mature safety method | Apply when required and borrow the hazard discipline elsewhere; do not claim universal legal applicability |
| Current terminology is best | Production records must remain interpretable under the version used when created; pin and migrate deliberately |
| Prior-authorization APIs standardize submission | Clinical completeness, payer policy, X12 mapping, rollout dates, and operating rules still vary |

## Failure and hazard register

| Hazard | Example cause | Required control | Release severity |
|---|---|---|---|
| Wrong-patient action or disclosure | Demographic near-match, stale portal link | Verified binding, namespace-qualified IDs, no model merge, pre-effect recheck | Hard stop |
| Unauthorized proxy disclosure | Revoked or scope-limited authority cached as true | Structured grant, freshness check, data/action/purpose filtering | Hard stop |
| Clinical recommendation | Model converts diagnosis evidence into treatment advice | Output schema, policy validator, clinical-language detector, human escalation | Hard stop |
| Missed deterioration escalation | Message classified as routine or queue stalls | Deterministic triggers, conservative uncertainty route, staffed queue SLO, backup channel | Hard stop |
| Referral lost to follow-up | External acceptance not tracked | Accountable owner, due date, receipt/postcondition, overdue escalation | Hard stop |
| Duplicate appointment/request | Timeout after external commit and blind retry | Semantic effect ID, vendor idempotency, reconciliation before retry | Hard stop |
| Fabricated clinical evidence | Summary lacks source or source changed | Versioned evidence references, contradiction state, no unsourced clinical field | Hard stop |
| Stale care plan | Cached task projection survives plan change | Version precondition, invalidation event, clinician-owned source | Hard stop |
| Private-channel leak | Reminder sent with sensitive details or to wrong address | Channel policy, minimal template, address verification, delivery controls | Hard stop |
| Inaccessible communication | Unsupported language, screen-reader failure, no alternate channel | Preference/accommodation source, WCAG testing, interpreter/human fallback | Blocking until safe route |
| Silent unknown outcome | Process crashes after send | Effect state unknown, reconciliation queue, manual review | Hard stop if overdue |
| Cross-tenant exposure | Cache, trace, or retrieval scope error | Tenant binding at every query, encrypted isolation, redacted telemetry, adversarial tests | Hard stop |
| Prompt injection | Untrusted referral/document instructs tools | Treat content as data, closed tools, no instruction inheritance, least privilege | Hard stop |
| Care-team orphaning | Clinician leaves or queue routing changes | Role-based owner, coverage schedule, transfer acknowledgment | Blocking |

## Evaluation evidence required

### Dataset design

- synthetic and de-identified cases reviewed under the organization's governance;
- near-match and merge/split identity cases;
- patient, minor, guardian, personal-representative, caregiver, revoked-proxy, and workforce scenarios;
- referrals, scheduling, authorizations, outreach, results follow-up, and transitions;
- language, disability, low-digital-access, and channel-preference slices;
- conflicting EHR/HIE/payer facts and version changes;
- adversarial instructions embedded in notes and attachments;
- external timeouts before and after commit;
- urgent or concerning statements that must trigger, but not be dispositioned by, the agent.

### Hard release gates

Any observed instance blocks release:

- wrong-patient read, disclosure, or effect;
- unauthorized representative access or action;
- diagnosis, treatment, prescribing, dose-change, or emergency-disposition recommendation;
- fabricated clinical evidence or clinical priority;
- missed or late required safety escalation;
- lost or silently closed task, referral, or authorization;
- duplicate consequential effect;
- PHI leak to logs, model training, unrelated tenant, or unapproved processor;
- unknown external outcome past its reconciliation limit.

### Statistical and operational evaluation

Average success is insufficient. Report repeated-trial distributions and stratified results for:

- schema validity and evidence attribution;
- identity/authority abstention correctness;
- deterministic-vs-model routing;
- administrative proposal correctness;
- safety-escalation sensitivity and route latency;
- effect deduplication and postcondition verification;
- end-to-end closure and human rework;
- accessibility and language equivalence;
- latency, token/tool consumption, queue depth, and unit cost;
- recovery after injected failures and deploy rollback.

Thresholds are local risk decisions approved in the safety case. The blueprint intentionally does not invent universal clinical timing or accuracy targets.

## Source register

All sources were accessed on 2026-08-31.

### Interoperability and workflow

| Source | Version/status used | Contribution and limitation |
|---|---|---|
| [FHIR R5](https://hl7.org/fhir/R5/) | 5.0.0 | Core resource semantics; not assumed to match deployed servers |
| [FHIR Resource identity and versions](https://hl7.org/fhir/resource.html) | R5, normative resource foundation | Distinguishes server-scoped logical identity, record version, business version and FHIR release; servers may not retain history and version IDs are not orderable strings |
| [FHIR CapabilityStatement](https://hl7.org/fhir/capabilitystatement.html) | R5, normative from R4 | Describes actual endpoint capability for one FHIR version; narrative/deployed behavior and local qualification still require tests |
| [HL7 Version 2 to FHIR](https://www.hl7.org/fhir/uv/v2mappings/) | 1.0.0 STU1, 2025 on R4 | Current cumulative mapping starting point; explicitly does not cover every local profile, Z-segment or terminology equivalence, so local transformation validation remains mandatory |
| [FHIR Patient match](https://hl7.org/fhir/patient-operation-match.html) | R5, trial use | Candidate matches, implementation-specific algorithms |
| [FHIR Patient merge](https://fhir.hl7.org/fhir/patient-operation-merge.html) | R5, maturity 0 trial use | Shows source/target/link semantics and that `$merge` is not idempotent; not selected as an agent operation |
| [FHIR Consent](https://hl7.org/fhir/consent.html) | R5, trial use | Consent representation; enforcement out of scope |
| [FHIR CarePlan](https://hl7.org/fhir/careplan.html) | R5 | Intended care and activity references; not agent authority |
| [FHIR ServiceRequest](https://hl7.org/fhir/servicerequest.html) | R5 | Referral/order intent and requester semantics |
| [FHIR Task](https://hl7.org/fhir/task.html) | R5, trial use | Shared workflow state; requires explicit local mapping |
| [FHIR workflow](https://hl7.org/fhir/workflow.html) | R5 | Separates workflow information sharing from execution |
| [FHIR Appointment](https://hl7.org/fhir/appointment.html) | R5 | Booking lifecycle and administrative/clinical boundary |
| [FHIR Slot](https://hl7.org/fhir/slot.html) | R5 | Availability observation; no booking guarantee |
| [FHIR CommunicationRequest](https://hl7.org/fhir/communicationrequest.html) | R5, trial use | A request to communicate, not proof of communication |
| [FHIR Communication](https://hl7.org/fhir/communication.html) | R5, trial use | Record of transmission; not comprehension |
| [FHIR HTTP](https://hl7.org/fhir/http.html) | R5 | ETags, conditional operations, transactions; server support varies |
| [FHIR digital signatures](https://fhir.hl7.org/fhir/signatures.html) | R5, maturity 1 trial use | Detached/enveloped signature patterns and cautions; signature policy/legal effect remains deployment-specific |
| [FHIR security labels](https://hl7.org/fhir/security-labels.html) | R5 | Data labels within a broader policy/trust framework |
| [FHIR Provenance](https://hl7.org/fhir/provenance.html) | R5 | Source lineage for resource versions and transformations |
| [FHIR AuditEvent](https://hl7.org/fhir/auditevent.html) | R5 | Security/privacy event representation |
| [SMART App Launch](https://hl7.org/fhir/smart-app-launch/) | 2.2.0, STU | Discovery, launch, scopes, backend authorization |
| [US Core](https://hl7.org/fhir/us/core/STU9/) | 9.0.0, 2026 STU on R4 | Current US profile example; remains trial use |
| [IHE PIXm](https://profiles.ihe.net/ITI/PIXm/) | 3.1.0, 2025 R4 trial implementation | Cross-domain identifier query/feed with declared options; not a merge or golden-record authority |
| [IHE PDQm](https://profiles.ihe.net/ITI/PDQm/) | 3.2.0, 2025 R4 trial implementation | Demographic query and match transactions; response conformance does not choose a local acceptance threshold |
| [IHE PDQm test plan](https://profiles.ihe.net/ITI/PDQm/testplan.html) | 3.2.0 prototype test plan | Operation-level conformance starting point; explicitly not a complete local safety/authorization/failure suite |
| [IHE BALP](https://profiles.ihe.net/ITI/BALP/) | 1.1.4 | Privacy-centric AuditEvent patterns |
| [C-CDA](https://hl7.org/cda/us/ccda/5.0.0/) | 5.0.0 STU5, 2026; CDA R2-based | Current US document-template example; generated with FHIR tooling but not a FHIR resource exchange contract |
| [C-CDA provenance](https://www.hl7.org/cda/us/ccda/provenance.html) | 5.0.0 | Author/assembler “last hop” guidance; full end-to-end provenance may require additional application evidence |
| [National Directory IG](https://hl7.org/fhir/us/ndh/) | 1.0.0, R4 STU1 | Qualified example for provider/service/endpoint directory data; directory presence is not availability, credentialing or handoff acceptance |
| [Da Vinci Plan-Net](https://hl7.org/fhir/us/davinci-pdex-plan-net/STU1.2/) | 1.2.0, R4 STU1.2 | Qualified payer-network directory example; network/freshness and payer deployment vary |
| [TEFCA](https://healthit.gov/policy/tefca/) | Program page updated 2026 | US trust framework context; not blanket access |
| [TEFCA Common Agreement](https://rce.sequoiaproject.org/common-agreement/) | CA 2.1; QTF 2.1 | Governing exchange framework and technical requirements |

### Identity, privacy, security, and communication

| Source | Version/status used | Contribution and limitation |
|---|---|---|
| [ONC patient identity and matching](https://healthit.gov/standards-and-technology/patient-identity-and-patient-record-matching/) | Updated 2025 | Matching context and standardized demographic data |
| [Project US@](https://isp.healthit.gov/representing-patient-address-0) | 1.0 | US address representation; not a full matching solution |
| [GAO-19-197](https://www.gao.gov/products/gao-19-197) | 2019 | Safety/privacy impact of false positives and fragmented records |
| [NIST SP 800-63-4](https://pages.nist.gov/800-63-4/) | Revision 4, final 2025 | IAL/AAL/FAL separation and identity-risk management |
| [HHS Privacy Rule summary](https://www.hhs.gov/hipaa/for-professionals/privacy/laws-regulations/index.html) | Current page | US HIPAA foundation; not global or exhaustive |
| [HHS minimum necessary](https://www.hhs.gov/hipaa/for-professionals/privacy/guidance/minimum-necessary-requirement/index.html) | Current guidance | Purpose-based access and explicit exceptions |
| [HHS personal representatives](https://www.hhs.gov/hipaa/for-professionals/privacy/guidance/personal-representatives/index.html) | Current guidance | Scope and exception complexity |
| [HHS TPO disclosures](https://www.hhs.gov/hipaa/for-professionals/privacy/guidance/disclosures-treatment-payment-health-care-operations/index.html) | Current guidance | Distinguishes HIPAA consent and authorization concepts |
| [HHS Security Rule](https://www.hhs.gov/hipaa/for-professionals/security/laws-regulations/index.html) | Current rule page, updated 2026 | Current requirements; proposed changes are not treated as final |
| [NIST SP 800-66r2](https://csrc.nist.gov/pubs/sp/800/66/r2/final) | Revision 2 | Security Rule resource-guide mapping |
| [HHS cloud computing guidance](https://www.hhs.gov/hipaa/for-professionals/special-topics/health-information-technology/cloud-computing/index.html) | Current guidance | Business-associate and BAA implications for ePHI cloud processing |
| [HHS business associates](https://www.hhs.gov/hipaa/for-professionals/privacy/guidance/business-associates/index.html) | Updated 2026 | Third-party service and AI relationship examples |
| [HHS Breach Notification Rule](https://www.hhs.gov/hipaa/for-professionals/breach-notification/index.html) | Current page | Incident obligations; organization counsel must assess events |
| [42 CFR Part 2 final-rule fact sheet](https://www.hhs.gov/hipaa/for-professionals/regulatory-initiatives/fact-sheet-42-cfr-part-2-final-rule/index.html) | Updated 2026 | Additional US substance-use record considerations |
| [HHS appointment reminder FAQ](https://www.hhs.gov/hipaa/for-professionals/faq/198/may-health-care-providers-leave-messages/index.html) | Current FAQ | Minimal disclosure and professional judgment for reminders |
| [HHS email FAQ](https://www.hhs.gov/hipaa/for-professionals/faq/570/does-hipaa-permit-health-care-providers-to-use-email-to-discuss-health-issues-with-patients/index.html) | Current FAQ | Address checks, safeguards, and reasonable patient requests |
| [Direct Standard](https://directtrust.org/standards/the-direct-standard) | Current specification family | Secure transport foundation and certificate model; transport does not prove clinical acceptance, comprehension or closed-loop handoff |

### Care coordination, safety, accessibility, and AI risk

| Source | Version/status used | Contribution and limitation |
|---|---|---|
| [AHRQ Care Coordination Measures Atlas](https://www.ahrq.gov/ncepcr/care/coordination/atlas.html) | Official reference | Care-coordination definition and domains; older but still useful |
| [2025 SAFER guides](https://healthit.gov/clinical-quality-and-safety/safer-guides/) | 2025 set | Patient identification, clinician communication, and follow-up controls |
| [SAFER Patient Identification](https://healthit.gov/resources/2025-safer-guide-patient-identification/) | 2025 | EHR patient-identification safety practices |
| [SAFER Clinician Communication](https://healthit.gov/wp-content/uploads/2025/06/SAFER-Guide-1.-Clinical-Communication-Final.pdf) | 2025 | Referral tracking and communication safety |
| [NHS DCB0129](https://digital.nhs.uk/data-and-information/information-standards/governance/latest-activity/standards-and-collections/dcb0129-clinical-risk-management-its-application-in-the-manufacture-of-health-it-systems/) | Current, under review | Manufacturer clinical-risk process; jurisdiction-specific |
| [NHS DCB0160](https://digital.nhs.uk/data-and-information/information-standards/governance/latest-activity/standards-and-collections/dcb0160-clinical-risk-management-its-application-in-the-deployment-and-use-of-health-it-systems) | Current, under review | Deployment clinical-risk process; jurisdiction-specific |
| [WCAG 2.2](https://www.w3.org/TR/WCAG22/) | W3C Recommendation, 2024 | Testable web accessibility criteria |
| [HHS Section 1557 disability fact sheet](https://www.hhs.gov/civil-rights/for-individuals/section-1557/fs-disability/index.html) | Current fact sheet | Effective communication and disability access in covered US contexts |
| [WHO Ethics and governance of AI for health](https://www.who.int/publications/i/item/9789240037403) | WHO guidance | Human autonomy, safety, transparency, accountability, equity |
| [WHO large multimodal model guidance](https://www.who.int/news/item/18-01-2024-who-releases-ai-ethics-and-governance-guidance-for-large-multi-modal-models) | 2024 | Risk controls, stakeholder engagement, post-release auditing |
| [NIST AI 600-1](https://doi.org/10.6028/NIST.AI.600-1) | 2024 profile | Generative-AI risk taxonomy and controls |
| [FDA CDS guidance](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/clinical-decision-support-software) | Final, January 2026 | Intended-use and CDS boundary considerations |
| [FDA GMLP principles](https://www.fda.gov/medical-devices/software-medical-device-samd/good-machine-learning-practice-medical-device-development-guiding-principles) | Current page | Lifecycle practices if device scope becomes relevant |

### Prior authorization and terminology

| Source | Version/status used | Contribution and limitation |
|---|---|---|
| [Da Vinci CRD](https://hl7.org/fhir/us/davinci-crd/) | 2.2.1 | Coverage-requirements discovery contracts |
| [Da Vinci DTR](https://hl7.org/fhir/us/davinci-dtr/) | 2.2.0, 2026 | Questionnaire/evidence collection with review and confirmation |
| [Da Vinci PAS](https://hl7.org/fhir/us/davinci-pas/) | 2.2.1 | FHIR prior-authorization submission and X12 mapping |
| [CMS-0057-F](https://www.cms.gov/initiatives/burden-reduction/overview/interoperability/policies-regulations/cms-interoperability-prior-authorization-final-rule-cms-0057-f) | Current CMS page | US payer obligations, staged dates, and exclusions |
| [CMS API standards and IG FAQ](https://www.cms.gov/priorities/burden-reduction/overview/interoperability/frequently-asked-questions/standards-implementation-guides) | Updated 2026 | Confirms the final rule's recommended IGs are not currently mandatory and newer versions have conditions; CMS-0062-P changes remain proposed |
| [CMS Prior Authorization API FAQ](https://www.cms.gov/initiatives/burden-reduction/overview/interoperability/frequently-asked-questions/prior-authorization-api) | Updated 2026 | Required response distinctions and timing for impacted payers; not a universal clinical urgency or all-payer policy |
| [X12 278](https://x12.org/node/4256) | Transaction-set family; exact TR3/version required | Health-care-services-review request/response semantics; implementation rules and code content can be licensed and trading-partner-specific |
| [X12 standards recommendation](https://x12.org/news-and-events/x12-recommendation-letter) | 2026 recommendation | X12 recommends 008060, including X342 for 278; a recommendation is not the currently mandated deployment version |
| [LOINC release notes](https://loinc.org/kb/loinc-release-notes/) | 2.83, 2026 | Demonstrates the need to pin terminology versions |
| [RxNorm](https://www.nlm.nih.gov/research/umls/rxnorm/index.html) | Current NLM program | Normalized medication naming; not medication advice |
| [SNOMED CT release specification](https://docs.snomed.org/snomed-ct-specifications/snomed-ct-release-file-specification/1-introduction) | Current specification | Versioned clinical terminology artifacts |
| [CMS ICD-10 codes](https://www.cms.gov/medicare/coding-billing/ICD-10-codes) | Current CMS page | Fiscal-year coding versions; not authority to infer a code |

## Known limitations

- No live EHR, HIE, payer, scheduling, identity, or communication system was conformance-tested.
- No named product, tenant, payer, trading partner, channel, directory, document/e-sign service, workflow engine or adapter operation is qualified by this packet. Capability, profile, scope, timeout, idempotency, correction, receipt and postcondition behavior must be tested against the actual contract and deployment.
- No jurisdiction-specific legal opinion, medical-device classification, or clinical-safety sign-off was performed.
- The packet does not define clinical deterioration indicators, urgency tiers, escalation destinations, or response-time thresholds. Accountable clinical governance must supply them.
- FHIR R5 examples describe semantics; real deployments may use R4, proprietary APIs, implementation-specific extensions, X12 transactions, Direct messaging, HL7 v2, or manual portals.
- The identity/version/effective-time matrix is a normalization design, not a claim that every source exposes all fields or history. Missing versions, effective times, acknowledgments or correction feeds can require narrower authority or manual operations.
- Accessibility and language obligations vary, and automated testing cannot establish effective communication by itself.
- No universal SLO, capacity target, retention period, or model accuracy threshold is asserted.
- Sources can change after the research date; “current” means checked on 2026-08-31.
- A deployment that changes intended use from administration to clinical recommendation is outside this blueprint.

## Refresh triggers

Refresh this packet when any of the following changes:

- deployed FHIR release, US Core or national implementation guide, Da Vinci guide, IHE profile, TEFCA requirement, payer API, or terminology package;
- patient identity, personal-representative, consent, privacy, security, accessibility, or records law;
- CMS prior-authorization applicability or deadline;
- FDA or other regulator guidance affecting intended use;
- DCB0129/DCB0160 review outcome or local clinical-safety standard;
- model provider, data-retention policy, training-use terms, subprocessors, regional processing, or assurance evidence;
- new channel, identity proofing method, patient population, language, disability support, workflow, clinical partner, or jurisdiction;
- hard-gate evaluation failure, wrong-patient near miss, unauthorized disclosure, delayed safety handoff, duplicate effect, or unreconciled unknown;
- adapter CapabilityStatement, scope behavior, field mapping, or idempotency behavior;
- material drift in cost, latency, queueing, failure rate, or human rework.

## Research completion assessment

The primary architecture is stable across the strongest sources: a bounded administrative agent inside a deterministic, durable, policy-enforced workflow. Additional searching stopped changing that decision and mainly added jurisdiction- or integration-specific detail. The remaining work is deployment research against actual organizations, systems, contracts, patient populations, and clinical-safety processes—not more generic web research.

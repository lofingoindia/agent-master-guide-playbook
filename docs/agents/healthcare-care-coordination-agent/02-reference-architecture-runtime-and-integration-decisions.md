# Reference Architecture, Runtime, and Integration Decisions

This guide selects a runtime that is small enough to reason about and strong enough to survive days-long healthcare workflows, partial failures, PHI constraints, and clinical-safety interrupts.

## Selected runtime pattern

Use a **durable workflow with one bounded model worker**.

The workflow engine or durable job system owns state transitions, deadlines, retries, leases, cancellations, human waits, and recovery. A stateless model worker receives a policy-filtered evidence package and returns one typed administrative proposal. Typed adapters perform reads and effects only after application-side validation.

~~~mermaid
flowchart LR
    I[Authenticated intake] --> G[Identity and authority gate]
    G --> W[Durable workflow]
    W --> R{Rule can decide?}
    R -- Yes --> T[Deterministic transition]
    R -- No --> C[Context assembler]
    C --> M[Model worker]
    M --> V[Schema, evidence, policy, safety validator]
    V -- Accepted --> T
    V -- Rejected --> H[Human or clinician queue]
    T --> E{External effect?}
    E -- No --> W
    E -- Yes --> L[Effect ledger and outbox]
    L --> A[Typed adapter]
    A --> Q[Receipt and postcondition]
    Q --> W
~~~

This is a hybrid, not a chat loop. It continues even if the model provider, process, or channel session disappears.

## Architecture alternatives

| Option | Strength | Failure in this workload | Decision |
|---|---|---|---|
| Deterministic workflow only | Lowest uncertainty, cost, and PHI exposure | Cannot economically resolve recurring ambiguous narrative | Preferred baseline; retain for all known paths |
| Chat application plus tools | Fast prototype | Session-bound state, weak recovery, hidden effects, unclear ownership | Reject beyond isolated D0/D1 prototype |
| Custom loop plus database | Simple for short workflows | Teams often recreate timers, leases, recovery, and replay poorly | Accept only if existing platform already supplies those controls |
| Durable workflow plus bounded worker | Explicit waits, retries, recovery, ownership, and audit | More operational components | Selected |
| Multi-agent planner/specialists | Parallel reasoning and specialization | More PHI copies, handoffs, cost, nondeterminism, and ownership ambiguity | Reject for baseline |
| Autonomous browser/computer use | Can reach legacy portals | Fragile UI state and dangerous outcome ambiguity | Exception-only, supervised, read-only or staged where no API exists |

Do not introduce multiple agents merely to mirror organizational departments. Add a worker only when it has a distinct tool boundary, independent evaluation, and measurable throughput or quality benefit that cannot be achieved with deterministic functions.

## Component responsibilities

| Component | Owns | Must not own |
|---|---|---|
| Channel gateway | Session, rate limit, transport metadata, accessibility options | Patient-record identity truth |
| Identity binding service | Actor assurance, patient binding, representative evidence | Clinical interpretation |
| Policy decision point | Purpose, role, consent, proxy scope, data/action decision | Language-model judgment |
| Case API | Validated commands and versioned reads | Hidden side effects |
| Durable workflow | State, events, timers, ownership, leases, budgets, compensation, stop states | Clinical decisions |
| Context assembler | Authorized facts, provenance, freshness, contradiction markers | Retrieval over all PHI |
| Model worker | One schema-constrained administrative proposal | Direct credentials, durable state, effects, clinical authority |
| Proposal validator | Schema, closed actions, evidence, policy, clinical boundary | Generative correction of unsafe content |
| Effect service | Idempotency, outbox, dispatch, receipts, postconditions | Blind retry of unknown outcomes |
| Adapter | Vendor/FHIR contract normalization | Cross-case policy or model planning |
| Reconciler | Compare intended and observed external state | Invent completion |
| Human queue | Review, exception resolution, approval, accountability | Untracked chat handoff |
| Audit/safety service | Tamper-evident decisions, access, effects, hazards, releases | Raw PHI-heavy observability by default |

## Model-selection strategy

Select the smallest approved model that passes the task-specific hard gates.

Evaluate candidates on:

- structured-output reliability;
- citation to supplied evidence references;
- abstention on identity, proxy, clinical, and safety ambiguity;
- resistance to instructions embedded in notes and documents;
- language and accessibility equivalence;
- latency, throughput, regional availability, retention, training use, subprocessors, and contractual safeguards;
- behavior stability across repeated trials and upgrades.

Do not choose primarily from public medical-question benchmarks. This agent's critical tasks are administrative grounding, authority abstention, effect safety, and recovery.

The provider contract must define:

- permitted PHI and jurisdictions;
- retention and deletion;
- whether prompts/outputs train models;
- subprocessors and region;
- encryption and access controls;
- incident notification and audit evidence;
- version pinning or change notice;
- availability, quotas, and fallback behavior.

A “HIPAA-ready” marketing claim is not a risk analysis, contract, business-associate agreement, or proof that every feature is in scope.

## Integration qualification

### Required adapter manifest

~~~yaml
adapter_manifest:
  adapter_id: ehr-a-fhir
  manifest_version: 7
  owner: interoperability-team
  deployment:
    environment: production
    tenant_ids: [tenant-a]
    endpoint_fingerprint: opaque
    contract_id: opaque
  protocol_contract:
    family: FHIR
    release: 4.0.1
    implementation_guides: [pinned-package-and-version]
    capability_statement_ref: immutable-snapshot
    schemas_and_profiles: [canonical-url-and-version]
    terminology_packages: [canonical-url-and-version]
  security_contract:
    authentication_profile: SMART-backend-services-pinned-version
    token_audience: exact-endpoint
    scopes: [minimum-operation-scopes]
    network_and_region_policy: opaque
  operations:
    - operation_id: referral.read.v2
      direction: read
      authority_tier: D1
      transport: GET-ServiceRequest-vread
      input_schema: opaque-version
      output_schema: opaque-version
      source_of_truth_fields: [status, intent, priority, code, subject, authoredOn]
      identifier_contract: [server-base, resource-type, logical-id, record-version]
      effective_time_fields: [authoredOn, occurrence]
      freshness_policy: referral-read-v2
      preconditions: [patient-binding-current, purpose-permitted]
      pagination_and_limit_policy: no-pagination-single-vread
      timeout_boundary: no-domain-commit; read-result-unavailable
      idempotency: read-only
      outcome_mapping: versioned-enum-map
      success_postconditions: [subject-and-version-match]
      reconciliation_query: same-version-read
      cancellation_or_compensation: not-applicable
      test_suite_ref: opaque-version
    - operation_id: task.transition.v3
      direction: write
      authority_tier: D3
      transport: PUT-Task-with-If-Match
      input_schema: opaque-version
      output_schema: opaque-version
      semantic_effect_key_schema: opaque-version
      preconditions: [source-version-current, exact-approval-current]
      downstream_idempotency: conditional-update-tested
      timeout_boundary: commit-may-be-unknown
      success_postconditions: [task-id-subject-status-and-version-match]
      reconciliation_query: task-read-by-business-id
      cancellation_or_compensation: new-domain-effect
      test_suite_ref: opaque-version
  rate_limit_and_backpressure_policy: named-version
  known_deviations: []
  qualification_evidence: [sandbox-run, negative-tests, failure-injection, owner-signoff]
  qualified_at: timestamp
  expires_at: timestamp
  rollback_version: opaque
~~~

An adapter is qualified by **operation**, not by vendor or endpoint. A read, search, conditional update, booking, cancellation, send, and status query on the same product can have different authorization, idempotency, timeout, and receipt semantics. “FHIR API” is not a sufficient integration specification. Record the deployed release and endpoint, actual `CapabilityStatement`, profiles, extensions, searches, operations, pagination, conditional behavior, terminology packages, scopes, contract limits, and deviations.

The manifest expires on endpoint, tenant configuration, contract, profile, operation, scope, mapping, or vendor-behavior change. Marketing certification and a passing generic conformance suite do not qualify an operation for a local workflow.

### Operation qualification matrix

| Adapter family and minimum operation | Qualification evidence | Mandatory negative and recovery tests | Production limitation to disclose |
|---|---|---|---|
| EHR/FHIR read or versioned read of Patient, Encounter, ServiceRequest, CarePlan, Task, medication/diagnosis evidence, and DocumentReference | Actual endpoint metadata; FHIR release; profile and search support; version-aware reference behavior; field-level source map; compartment/scope test | Wrong tenant/patient, deleted/entered-in-error resource, unsupported modifier extension, stale version, incomplete pagination, unknown terminology, overbroad search, rate limit | Tenant configuration, release/profile, retained history, fields omitted by policy, consistency and freshness window |
| EHR/FHIR conditional create/update or local-to-EHR Task transition | Exact request/response fixture; `ETag`/`If-Match` or conditional behavior; business-identifier uniqueness; Provenance/AuditEvent mapping where used | Duplicate request, precondition race, server ignores conditional, partial transaction, timeout after commit, response references wrong subject, rollback with in-flight write | Conditional semantics and transaction boundary of that server only; no cross-system atomicity |
| HIE/MPI patient demographics match and cross-reference query | Exact FHIR `$match`, IHE PDQm/PIXm, or local contract; required inputs; candidate/score semantics; identifier domains; steward route | Zero/one/many candidates, tied scores, missing identifier domain, demographic collision, stale link, merge/split/link/unlink, unavailable match service | Candidate generation only; algorithm, thresholds, identity domains, and feed/query options vary; no model merge authority |
| Scheduling search, hold, book, reschedule, cancel, and read-back | Per-operation slot/hold/appointment state map; timezone and recurrence behavior; hold expiry; business ID; receipt and verification query | Slot race, DST boundary, duplicate booking, hold expiry, timeout after commit, late success, cancellation after downstream work, wrong patient/service | A `Slot` observation is not inventory lock; product/site/service rules and cancellation effects vary |
| Coverage and payer eligibility/status query | Exact FHIR Coverage/CoverageEligibility, X12 270/271, or payer API version; beneficiary/subscriber mapping; service-date semantics; response correlation | Member ID collision, dependent mismatch, inactive/future coverage, multiple coverages, stale response, unmapped code, payer outage | Coverage resource/status is not a guarantee of payment or authorization; plan, contract, tenant, and service-date limits apply |
| Prior-authorization discovery, documentation, submit, inquiry, update, and cancel | Exact CRD/DTR/PAS release or X12 TR3/version; payer rules; Must Support fields; clinical-attestation boundary; request/control numbers; acknowledgment and decision map | Missing situational field, stale questionnaire/rule, unsupported code, duplicate 278/PAS submit, request-for-information loop, timeout after submit, late/contradictory decision, cancellation race | CMS applicability and dates, payer support, drug exclusions, X12 licensing/mapping, and trial-use IG status are explicit |
| Provider/service/destination directory query | Exact NDH, Plan-Net, local directory, or contract profiles; organization/location/service/network/endpoint joins; verification and effective dates | Closed location, out-of-network result, stale endpoint, role without coverage, ambiguous organization, accessibility/language field absent | Directory presence is neither credentialing, availability, acceptance, nor clinical handoff; freshness/verification are tenant-specific |
| Secure message or patient communication send/status | Exact channel, recipient/address binding, payload class, template, provider receipt codes, retention and callback contract | Revoked proxy, address changed after approval, delivery timeout, duplicate send, aggregate receipt, bounce after “delivered,” opt-out, inaccessible channel | Transport acceptance is not recipient receipt, comprehension, or ownership transfer; channel privacy varies |
| Document fetch/ingest/extract and e-sign/attest | CDA/C-CDA/FHIR/profile/version, MIME allowlist, immutable byte hash, document identifier/version, author/attester/signature verification policy, extraction span lineage | Active content, oversized/archive bomb, malformed CDA, replaced document, hash mismatch, revoked/expired signer certificate, ambiguous signature purpose, OCR disagreement | Structural validation is not clinical correctness; electronic and cryptographic signatures have different guarantees and legal effect |
| Workflow/task/handoff command and event subscription | Command schema, aggregate version, event ordering/dedup, timer behavior, owner acceptance, replay and restart evidence | Duplicate/late/reordered event, lost lease, worker restart, missed timer, queue reassignment, unacknowledged handoff, poison event | FHIR Task or webhook exchange does not supply local leases, exactly-once effects, staffed queues, or closure guarantees |

Each row expands into separate manifest operations. A successful read test cannot qualify a write; a successful send test cannot qualify a delivery-status query; and qualification in one tenant, site, payer, or product release cannot be copied to another without evidence.

### Qualification procedure

1. Snapshot the deployed endpoint, tenant configuration, contract, protocol/IG packages, scopes, capability metadata, code maps, rate limits, and known deviations.
2. Create golden request/response fixtures with synthetic identities and every supported status, modifier, time field, and terminology version. Keep PHI out of public conformance services.
3. Run schema/profile validation **and** semantic assertions for subject, business identity, record version, effective time, status, receipt, and postcondition.
4. Execute the negative, concurrency, timeout-before/after-commit, duplicate, stale-version, and correction tests for that exact operation.
5. Measure pagination completeness, eventual-consistency window, throttling, maximum payload, timeout, and reconciliation behavior.
6. Have the integration owner and workflow risk owner sign the operation evidence; record residual deviations and expiry.
7. Requalify on capability drift, profile/terminology/package change, tenant configuration change, contract change, incident, or unexplained outcome-map drift.

Official test kits and IHE test plans are useful starting evidence, not proof of the local source-field map, authorization policy, failure semantics, or safe fitness for D3 effects.

### Integration decision table

| Integration | Default authority | Include when | Reject or constrain when |
|---|---|---|---|
| EHR/FHIR | D1 reads; selected D3 writes | Exact profiles and source-of-truth fields are known | Generic “FHIR compliant” claim without conformance tests |
| HIE/TEFCA | D1 purpose-limited query | Participation, exchange purpose, identity, and policy are configured | Treated as a global patient database |
| MPI/match | D1 candidate retrieval | Existing organization identity process owns resolution | Model can merge or auto-select ambiguous matches |
| Provider directory | D1 read | Ownership, freshness, effective dates, and coverage are available | Scraped or stale directory drives clinical handoff |
| Scheduling | D1 availability, D2 hold, D3 booking/cancel | Fresh precondition and receipt can be verified | Slot read is treated as booking |
| Payer/authorization | D1 requirements/status, D2 draft, D3 submit | Exact payer/profile/transaction rules are tested | Agent invents codes, necessity, or attestation |
| Communications | D2 draft, D3 send | Recipient, channel, content class, preference, and receipt are controlled | Generic email/SMS tool receives arbitrary PHI |
| Document extraction | D1 evidence input | Separate extraction system provides spans/provenance/confidence | Extractor's output is accepted as clinical truth |
| Browser/computer use | D1 or supervised D2 exception | No API exists and selectors, screenshots, and outcome checks are controlled | Consequential unattended portal writes |
| Open web | D0 research outside patient run | Public, non-patient domain knowledge is needed | Any identifiable patient context or medical advice generation |
| MCP/tool marketplace | None by default | Every server and tool passes the same security/PHI/authority qualification | Dynamic discovery grants new tools or broader scopes at runtime |

### Protocol and terminology boundaries

| Contract family | Safe use in this blueprint | What it does not prove | Pin and preserve |
|---|---|---|---|
| FHIR REST/resources | Exchange/query domain records and version-aware references through tested profiles and operations | Common workflow execution, global identity, consent enforcement, semantic idempotency, or cross-server transactions | Base endpoint/tenant, FHIR release, profile canonical/version, `id`, `meta.versionId`, business identifiers, extensions, search/operation support |
| HL7 v2 messages | Consume organization-defined ADT, order, scheduling, result, and correction events behind a typed adapter | That an ACK means the referral, booking, handoff, or care task completed; that two local code tables mean the same thing | Message version/profile, trigger event, sending/receiving facility, control ID, placer/filler IDs, ACK mode, local tables, correction/merge semantics |
| CDA/C-CDA documents | Exchange a persistent, human-readable clinical document and retain whole-document provenance | Freshness, semantic equivalence to extracted FHIR resources, clinical correctness, or that an attester authorized this agent's effect | CDA/C-CDA/template versions, document `id`, `setId`/version where applicable, service time, author/attester, immutable bytes/hash, replacement/addendum links |
| IHE profiles and trust frameworks | Define specific actors/transactions such as PDQm, PIXm, audit, document exchange, and governed network exchange | A global patient record, blanket purpose, automatic merge, or identical options across participants | Profile/version, actor/options, identifier domains, home community, purpose, security/audit profile, participant agreement, test evidence |
| X12 healthcare transactions | Exchange eligibility and health-care-services-review data under exact implementation guides and operating rules | Clinical truth, guaranteed coverage/payment, or that a transport/TA1/999/277-style acknowledgment is the final prior-auth decision | X12 version/TR3, trading-partner companion guide, sender/receiver/control and trace numbers, acknowledgments, response correlation, code lists, licensing limits |
| Terminology/code systems | Preserve a source-authored code and validate/map it for a defined purpose | Authority to invent a diagnosis/procedure, assume display-text equivalence, or silently migrate historical meaning | System URI/OID, edition/version/date, code, display, value-set canonical/version/expansion parameters, map/version/equivalence and reviewer |

Translations are loss-aware records. Store source code and version, target code and version, mapping artifact/version, purpose, relationship (`equivalent`, broader, narrower, inexact, unmapped), reviewer, and mapping time. An unmapped or inexact clinical code becomes a visible blocker; the model never chooses the “closest” billable, diagnostic, procedure, or medication code.

### Explicit rejections

- No model-held EHR, payer, email, or portal credentials.
- No unrestricted SQL, FHIR search, file-system, shell, email, or browser tool.
- No tool that accepts free-form recipient, patient, resource type, or query when a closed contract is possible.
- No automatic installation or discovery of new integrations in a patient run.
- No unapproved consumer communication channel for PHI.
- No connector that uses submitted data for training or retains it outside the approved contract.
- No effect through a read tool or hidden side effect in a “lookup.”

## FHIR and domain normalization

The local model should not see raw vendor-specific payloads. Adapters map them to a stable internal vocabulary with loss and uncertainty preserved.

| Domain concept | Common external representation | Normalization concern |
|---|---|---|
| Patient | Patient, Person, vendor patient ID | Namespace and merge/split history |
| Representative | RelatedPerson, portal proxy, legal-document record | Relationship is not authority; preserve scope and source |
| Consent | Consent, security labels, policy service | Expression is not enforcement |
| Clinician owner | PractitionerRole, CareTeam, directory record | Effective period, organization, coverage, acceptance |
| Referral | ServiceRequest, DocumentReference, message | Intent and priority remain clinician-authored |
| Work | Task, workqueue item, payer case | Map local and external lifecycle explicitly |
| Plan | CarePlan, Goal | Read versioned clinical source only |
| Appointment | Schedule, Slot, Appointment | Availability, hold, booking, arrival, and encounter differ |
| Outreach | CommunicationRequest, Communication, vendor delivery | Requested, dispatched, delivered, and acknowledged differ |
| Authorization | Coverage, Questionnaire, PAS bundle, X12 transaction | Versioned requirements and clinical attestation |
| Lineage | Provenance, meta.versionId, document hash | Preserve exact source and transformation |

Unknown fields and unmapped statuses must remain unknown. Do not coerce them into the nearest happy-path state.

## Example end-to-end referral

~~~mermaid
sequenceDiagram
    participant P as Patient or representative
    participant G as Identity and policy gate
    participant W as Durable workflow
    participant M as Model worker
    participant E as EHR or FHIR
    participant D as Directory or scheduler
    participant H as Coordinator or clinician

    P->>G: Ask for referral status
    G->>G: Verify actor, patient, purpose, proxy scope
    G->>W: Start or resume case with authority decision
    W->>E: Read versioned ServiceRequest and Task
    W->>D: Read destination and scheduling status
    alt Complete deterministic mapping
        W->>W: Apply fixed transition
    else Ambiguous administrative evidence
        W->>M: Send bounded facts and allowed actions
        M-->>W: Proposed gap plus evidence refs
        W->>W: Validate proposal and clinical boundary
    end
    alt Concerning or clinical uncertainty
        W->>H: Safety or clinical handoff
        H-->>W: Acknowledge ownership
    else Authorized outreach
        W->>H: D3 approval if required
        H-->>W: Approval bound to effect
        W->>D: Dispatch typed request
        D-->>W: Receipt
        W->>W: Verify postcondition and schedule follow-up
    end
    W-->>P: Minimal status, next step, owner, and honest uncertainty
~~~

## Bounded loop contract

Each model invocation gets:

- one case version;
- one current patient-binding reference;
- one authority decision;
- a bounded set of versioned evidence;
- explicit contradictions and missing facts;
- approved action enum;
- maximum data fields;
- remaining step, time, token, and tool budgets;
- active safety and effect states;
- behavior version.

It returns:

~~~yaml
coordination_proposal:
  proposal_type: request-evidence | route | draft-message | explain-status | abstain
  reason_code: approved-enum
  evidence_refs: [opaque-versioned-reference]
  missing_facts: []
  proposed_parameters: {}
  uncertainty: low | material
  clinical_decision_present: false
  safety_escalation_requested: false
~~~

The runtime rejects unknown actions, missing evidence, patient identifiers in fields not allowed by the contract, model-authored clinical content, or any proposal inconsistent with current state.

## Build-versus-framework rule

Use existing project infrastructure where it already supplies:

- durable workflows and human waits;
- typed schemas and policy checks;
- transactional outbox and effect ledger;
- secrets, tenancy, audit, and observability;
- testing and release controls.

A model SDK can simplify structured generation and tracing, but it is not the workflow, policy, identity, or effect layer. A workflow product can improve durability, but its retry semantics do not make downstream effects exactly once. See [Custom Loop vs Framework vs Workflow Engine](../../comparisons/custom-loop-vs-framework-vs-workflow-engine.md).

## Stage 1–2 architecture checks

- [ ] The deterministic path can complete ordinary structured cases without a model.
- [ ] The model has no durable credentials and cannot invoke adapters directly.
- [ ] Only one proposal schema and closed action set exist for the MVP.
- [ ] Every integration has an owner, manifest, test environment, and failure contract.
- [ ] Actual FHIR/profile/version behavior is conformance-tested.
- [ ] D1, D2, D3, and D4 operations are enumerated.
- [ ] Every D3 effect has approval/preauthorization, receipt, and postcondition logic.
- [ ] Safety-hold bypasses normal model planning.
- [ ] Unknown and unmapped states route safely.
- [ ] A manual workflow exists when the model or integration is unavailable.

## Related guides

- Previous: [Mission, Boundaries, Workload Fit, and Authority](01-mission-boundaries-workload-fit-and-authority.md)
- Next: [Patient Identity, Consent, Proxy, and Care-Team Authority](03-patient-identity-consent-proxy-and-care-team-authority.md)
- Home: [Healthcare Care Coordination Agent](README.md)
- Canonical: [Tool Contracts](../../tools/tool-contracts.md)

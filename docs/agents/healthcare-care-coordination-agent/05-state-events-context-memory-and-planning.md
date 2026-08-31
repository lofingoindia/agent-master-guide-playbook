# State, Events, Context, Memory, and Planning

Safe continuity comes from typed durable state, not a long transcript. This guide defines the case state machine, event and evidence contracts, bounded planning loop, context assembly, compaction, and each permitted memory class.

## Three truths, kept separate

| Truth | Authority | Example |
|---|---|---|
| Domain truth | EHR, HIE/MPI, scheduler, payer, directory, communication provider | Appointment status or referral record |
| Coordination truth | Local durable workflow ledger | What task is open, who owns it, and which effect is unknown |
| Conversational working state | Current turn/run context | Which ambiguity the model is resolving |

The local ledger does not overwrite a clinical record. The transcript does not overwrite either the ledger or the clinical record.

## Case state machine

~~~mermaid
stateDiagram-v2
    [*] --> Intake
    Intake --> Blocked: identity, authority, evidence, or owner missing
    Intake --> Ready: preconditions satisfied
    Blocked --> Ready: verified resolution
    Ready --> Executing: lease acquired
    Executing --> Waiting: external or human wait
    Waiting --> Ready: event or timer
    Executing --> Reconciling: effect outcome unknown
    Reconciling --> Ready: observed intended state
    Reconciling --> Blocked: mismatch or deadline exceeded
    Intake --> SafetyHold: safety trigger
    Ready --> SafetyHold: safety trigger
    Executing --> SafetyHold: safety trigger
    Waiting --> SafetyHold: safety trigger
    SafetyHold --> Ready: authorized release
    Ready --> Closed: closure invariants satisfied
    Closed --> Ready: authorized reopen event
~~~

State transitions use optimistic concurrency or a serializable workflow command. A model proposal cannot mutate state directly.

## Durable records

### Coordination task

~~~yaml
coordination_task:
  task_id: opaque
  case_id: opaque
  task_type: approved-enum
  source_refs: [versioned-reference]
  owner_assignment_id: opaque
  dependencies: [opaque]
  due_at: optional-timestamp
  status: open | blocked | in-progress | waiting | overdue | completed | canceled
  block_reason: optional-enum
  completion_evidence_refs: []
  version: integer
~~~

### Contradiction

~~~yaml
contradiction:
  contradiction_id: opaque
  case_id: opaque
  subject: approved-enum
  evidence_a_ref: versioned-reference
  evidence_b_ref: versioned-reference
  materiality: administrative | potentially-clinical
  status: open | routed | resolved
  resolver_type: deterministic-rule | authorized-human
  resolution_ref: optional
~~~

Potentially clinical contradictions are never resolved by the model.

### Event envelope

~~~yaml
coordination_event:
  event_id: opaque
  event_type: case.task-created.v1
  aggregate_type: coordination-case
  aggregate_id: opaque
  aggregate_version: integer
  tenant_id: opaque
  patient_binding_id: opaque
  actor_ref: opaque
  authority_decision_ref: optional
  occurred_at: timestamp
  recorded_at: timestamp
  correlation_id: opaque
  causation_id: opaque
  behavior_version: opaque
  payload_schema_version: 1
  payload: {}
~~~

Event IDs deduplicate delivery; aggregate versions prevent lost updates; correlation and causation reconstruct the trajectory. Keep PHI out of routing metadata and event names.

## State ownership

| Record/field | Writer | Reader | Conflict rule |
|---|---|---|---|
| Patient binding | Identity service/steward | Policy, workflow, adapters | Invalidation freezes dependent work |
| Authority decision | Policy service | Workflow/effect service | Re-evaluate at point of use |
| Clinical evidence | Domain adapter/source system | Context assembler/humans | Preserve versions and contradiction |
| Case state | Workflow command handler | All coordination components | Compare-and-swap/serialized transition |
| Model proposal | Model worker, immutable | Validator/workflow/human | Never edits facts or state |
| Task | Workflow/human command | Model, queues, UI | Versioned transition with owner |
| Effect | Effect service/reconciler | Workflow/operations | Explicit state machine |
| Handoff | Workflow/receiver | Queues, safety, audit | Acceptance required |
| Policy/domain knowledge | Governed publisher | Policy/context/runtime | Effective-date and behavior-version pin |

Avoid two writable representations of the same fact. When local projection and external source disagree, show the disagreement and reconcile; do not let last-write-wins hide it.

## Identity, version, and effective-time semantics

Do not use one field called `version` or `timestamp` for every meaning. Every normalized record separates:

- **protocol/profile version:** how bytes and fields are interpreted;
- **resource identity:** endpoint/tenant plus resource type and logical ID; a FHIR `id` alone is not portable across servers;
- **business identity:** namespace-qualified `Identifier.system + value` plus assigner/use/period where supplied;
- **record version:** source-managed revision such as FHIR `meta.versionId`, ETag, HL7 message control/version, or document replacement version;
- **local aggregate version:** compare-and-swap counter for the coordination projection;
- **clinical/business time:** when the care, order, coverage, relationship, or communication applies;
- **recorded and observed time:** when the source recorded it and when this runtime retrieved it.

FHIR version IDs are opaque: lexical or numeric order does not establish recency. `meta.lastUpdated` is not the clinical effective time. If a server does not retain history or emit usable versions, the adapter must disclose that limitation and use an immutable response hash plus observed time; high-risk stale-write protection may then be impossible and the operation must be constrained.

| Concept | Stable identity | Version evidence | Effective-time and status rule | Correction and invalidation rule |
|---|---|---|---|---|
| Patient | Local `patient_binding_id` plus tenant/source namespace and source Patient reference; demographic fields or a bare MRN are not identity | Binding version, source Patient record version, match/steward decision version, identifier-domain map version | Binding assurance has `verified_at` and expiry/freshness policy; `Patient.active` and link state are evaluated at point of use | Merge, link, unlink, split, identifier change, inactive/entered-in-error, or dispute freezes dependents; preserve old and replacement references and require steward-authorized rebind |
| Encounter | Tenant-qualified encounter business identifier and source Encounter reference | Encounter resource/message version plus local projection version | Keep planned dates separate from `actualPeriod`; status is encounter state, not patient state; evaluate source status as of observation time | Cancel, entered-in-error, identifier move, or corrected admission/discharge invalidates appointment/referral projections that depended on it |
| Coverage | Local coverage binding plus payer, beneficiary, subscriber/dependent and namespace-qualified coverage identifier; member ID alone may not be unique | Coverage resource/270–271/payer-response version, benefit/plan contract version, local binding version | Evaluate status and coverage period for the requested service date; observation time and payer consistency window remain explicit | Termination, retroactive change, beneficiary mismatch, coordination-of-benefits change, or payer correction reopens dependent authorization work; never rewrite the prior observed response |
| Consent | Local authority-evidence ID plus exact Consent/policy source reference and subject | Consent record version, policy profile/version, verification evidence version | Consent agreement period, provision period, controlled-data period, purpose, action, actor, security labels, and revocation are separate; re-evaluate at use time | Revocation, amended provision, conflict, entered-in-error, unknown label, or policy change invalidates decisions derived from it; Consent is evidence, not enforcement |
| Proxy/delegate | Local grant ID binding actor, patient, relationship source and permitted purpose/action/data/destination | Grant version, RelatedPerson/portal/legal-record version, policy version | Relationship period is not authority period; store grant effective period, verification, restrictions, revocation and point-of-use check | Relationship change, capacity/legal review, confidential-channel change, revocation, expiry, or source correction invalidates disclosure/effect approvals; never mutate history into “never existed” |
| Care-team role | Local assignment ID plus organization/service/role/queue and source CareTeam participant or PractitionerRole | Assignment version, source resource version, directory/coverage-roster version | Separate team period, practitioner-role period, participant coverage, on-call interval, assignment acceptance, and handoff due time | Role end, roster change, declined/overdue handoff, or directory correction keeps sender accountable until replacement acknowledges |
| Referral | Local referral case ID plus source ServiceRequest business identifiers/ref and destination correlation ID | ServiceRequest record version, placer/filler identifier map, destination receipt version | Preserve `status`, `intent`, clinician-authored priority, `authoredOn`, requested occurrence and destination response times; no field is synthesized to satisfy intake | Replacement, revocation, entered-in-error, destination correction, or amended clinical source invalidates derived tasks and pending submission approval |
| Order | Namespace-qualified placer/filler/business identifier plus exact ServiceRequest or MedicationRequest reference | Source record version and order-set/protocol/terminology versions | Separate activation/authored time, requested occurrence/validity, current status and clinical intent; administrative coordinator never changes intent | Cancel/revoke/replace/entered-in-error or clinical amendment invalidates plans and effects tied to the prior version; retain `basedOn`/`replaces` lineage |
| Appointment | Local booking effect ID plus scheduler appointment business ID and source Appointment reference; Slot ID alone is not booking identity | Appointment record version, scheduler contract/version, slot observation version | Start/end/timezone, status, participant status, hold expiry, cancellation time and observed-at are distinct; a free slot has a short explicit freshness window | Reschedule is a versioned domain transition or linked replacement; cancel/no-show/error and late scheduler correction trigger read-back, notification policy and case reopen as needed |
| Prior authorization | Local authorization case ID plus payer/trading-partner request, trace/control and authorization identifiers | PAS/CRD/DTR or X12/TR3 version, payer companion-guide/rule version, request and response sequence/version | Requested service date, submission time, acknowledgment, decision time, approval validity/end circumstance, information-request due time and status are separate | Corrected/reversed/voided response, new-information request, coverage change or clinical-source amendment preserves every payer response and invalidates unsupported closure/approval |
| Task | Local task ID plus external Task business identifier/ref where used | Local aggregate version, external Task record version, workflow-definition version | Separate authored, requested/due, execution start/end, last modified, wait and escalation clocks; status alone does not prove completion | Reopen/cancel/entered-in-error/source change creates an event and requires completion-evidence review; do not reset original due/age metrics |
| Medication/diagnosis reference | Local evidence ID plus exact MedicationRequest/Condition/other resource version and patient binding | Source record version, terminology system/edition/version, extraction version | Preserve clinical status, verification status, onset/abatement or authored/validity time, recorded time and retrieved time; none may be substituted for another | Refuted/entered-in-error/superseded/corrected evidence creates a contradiction and invalidates derived administrative claims; only accountable clinical processes resolve truth |
| Document | Logical document identifier plus source repository and immutable content hash; DocumentReference logical ID alone is insufficient | DocumentReference record version, document business version, CDA/template/profile version, byte hash, extraction version | Separate service period, document creation, indexing, authoring, attestation/signature and retrieval time | `replaces`, `appends`, `transforms`, `signs`, superseded or entered-in-error relationships preserve the original bytes and lineage; re-extract only as a new derived artifact |
| Message | Local communication/effect ID plus channel-provider business ID, exact recipient and address-version reference | Template/content hash, provider status version/sequence, adapter version | Requested, authorized, dispatched, sent, received/delivered, acknowledged and comprehended are distinct times/states | Bounce, wrong-address version, recall, recipient correction or provider status reversal never erases disclosure history; stop follow-on automation and route incident/correction work |
| Handoff | Local handoff ID binding sender assignment, intended receiver assignment and requested action | Aggregate version plus evidence-package and receiver-directory versions | Sent, due, acknowledged, accepted/declined and completed times are separate; sender retains ownership until configured acceptance | Receiver/coverage change, decline, overdue, or evidence correction reopens routing while preserving previous attempts and accountability |
| Safety flag | Local escalation ID plus exact source Flag/evidence reference when present | Escalation version, source Flag record version, safety-rule version | Active period, status, encounter scope, trigger time, acknowledgment deadline and clinician release time are distinct | Only an authorized clinical/safety transition can release the hold; inactive/entered-in-error source flags and corrected triggers require review, not model dismissal |
| Approval | Immutable approval ID bound to patient, effect, parameter hash, data classes, destination and source/policy/behavior versions | New approval record supersedes rather than overwrites; approval schema/version is pinned | `approved_at`, `expires_at`, consumed/revoked time and allowed execution window are explicit | Any material input/version change, revocation, expiry, correction or patient rebind voids use; prior approval remains auditable |
| Effect | Semantic effect ID plus target adapter/action and normalized business parameters | Ledger aggregate version, adapter/contract version, attempt numbers, external receipt/resource versions | Authorized, dispatched, outcome-observed and reconciled times are distinct; unknown is a state, not a timestamp inference | A correction, cancellation or compensation is a new linked effect; never delete or reuse the original effect ID for changed parameters |
| Correction | Immutable correction ID linking affected records/effects/disclosures to source and superseding evidence | Correction event/aggregate version and source-system correction version | Preserve source `occurred_at`, local `recorded_at`, effective-from/to if supplied, detection time and propagation completion time | Append; quarantine affected work, invalidate derived context/approvals, reconcile effects, propagate to indexes/read models, assess disclosures, and close only with propagation receipts |

Missing start/end, version, status, or code does not mean “current,” “inactive,” or “unchanged.” The adapter emits `unknown` plus the missing semantic, and the workflow applies the operation's freshness and authority policy.

## Facts, evidence, and inferences

Every context item has a class:

| Class | Meaning | May authorize an effect? |
|---|---|---|
| Authoritative fact | Fresh, versioned source record | Only with separate policy/approval |
| Derived administrative fact | Deterministic mapping from cited sources | If validated and policy allows |
| Model proposal | Nondeterministic suggestion with citations | Never by itself |
| User assertion | Statement from the current actor | Only where workflow explicitly treats it as an instruction and authority passes |
| Contradiction | Incompatible evidence | No; route or resolve |
| Unknown | Missing, stale, failed, or unmapped information | No |
| Clinical judgment | Decision by accountable clinician/source | May feed workflow under its own authority |

Prompts and summaries cannot upgrade a proposal into a fact.

## Context assembly

Assemble context for one step:

1. load the current case version;
2. check patient binding, actor, purpose, authority, tenant, and safety hold;
3. determine the smallest data classes required by the step;
4. fetch fresh authoritative sources through typed adapters;
5. normalize with source/version/provenance retained;
6. calculate deterministic administrative facts;
7. preserve contradictions, staleness, and unknowns;
8. retrieve only approved versioned domain policy;
9. include closed actions and budgets;
10. redact identifiers not necessary for the proposal;
11. record a context manifest without duplicating raw PHI.

### Context manifest

~~~yaml
context_manifest:
  context_id: opaque
  case_id: opaque
  case_version: integer
  patient_binding_ref: opaque
  authority_decision_ref: opaque
  purpose: care-coordination
  evidence_refs: [versioned-reference]
  policy_artifact_refs: [versioned-reference]
  contradiction_ids: []
  allowed_actions: [approved-enum]
  prohibited_domains: [diagnosis, treatment, medication-change, emergency-disposition]
  budgets: {steps_remaining: 2, tools_remaining: 1, deadline_at: timestamp}
  safety_state: normal | hold
  behavior_version: opaque
  created_at: timestamp
~~~

The context manifest proves what was supplied without copying the entire prompt into broad telemetry.

## Compaction and continuation

Compact at explicit checkpoints:

- before a human or external wait;
- before a context limit;
- after a material state transition;
- before provider/model switch;
- before deploy/drain;
- after safety escalation;
- after an effect becomes unknown.

### Required continuation package

~~~yaml
continuation_package:
  receipt_version: healthcare-coordination-continuation.v2
  checkpoint_id: opaque
  tenant_id: opaque
  case_id: opaque
  case_version: integer
  patient_binding_id: opaque
  binding_version: integer
  latest_authority_decision_ref: opaque
  version_pins:
    behavior: opaque
    workflow_definition: opaque
    policy_bundle: opaque
    adapter_manifests: [opaque]
    protocol_profiles: [opaque]
    terminology_packages: [opaque]
    templates_and_safety_rules: [opaque]
  source_refs: [versioned-reference]
  open_tasks:
    - {task_id: opaque, task_version: integer, owner_assignment_id: opaque, status: waiting}
  approvals:
    - {approval_id: opaque, effect_id: opaque, parameter_hash: opaque, status: active, expires_at: timestamp}
  active_clocks:
    - {clock_id: opaque, kind: handoff-ack, due_at: timestamp, timezone_or_rule_version: opaque}
  pending_effects:
    - {effect_id: opaque, status: authorized, dispatch_not_before: timestamp}
  unknown_effects:
    - {effect_id: opaque, last_attempt: integer, reconciliation_due_at: timestamp, query_contract_version: opaque}
  pending_handoff_ids: [opaque]
  contradiction_ids: [opaque]
  safety_flag_ids: [opaque]
  last_verified_external_state_refs: [versioned-reference]
  omitted_item_refs:
    - {class: raw-document, immutable_ref: opaque, reason: minimum-necessary}
  waiting_for: [approved-event-type]
  next_safe_action:
    action: reconcile-effect
    preconditions: [approved-predicate]
    prohibited_until_satisfied: [effect-dispatch]
  budgets_remaining: {steps: 2, tools: 1}
  source_event_high_watermark: tenant-partition-offset-or-event-id
  invariant_hash: sha256-over-canonical-critical-state
  receipt_signature_or_mac: opaque
  created_at: timestamp
~~~

The package does not contain a model-written clinical summary as truth. A short human-readable summary may accompany it for navigation, with clear links and “not authoritative” labeling. Omitted content is never represented as absent: every safety-relevant omission has an immutable reference or compaction fails.

### Fail-closed resume verification

On resume, before any model call or effect:

1. verify the receipt schema, signature/MAC, tenant/case binding and canonical invariant hash;
2. acquire a fenced lease for the current case aggregate and compare `case_version`;
3. replay/deduplicate source events strictly after `source_event_high_watermark`; a gap or inaccessible partition blocks resume;
4. reload the patient binding, authority, owners, safety state, tasks, approvals, clocks, handoffs and effect ledger from their authoritative stores;
5. resolve every version pin or apply an explicitly approved migration at a safe checkpoint; unavailable or changed behavior never falls forward silently;
6. re-read time-sensitive external sources and compare patient, business identity, source record version, effective time and status;
7. recompute invariants from canonical records, not the summary; mismatch creates a visible hold;
8. classify overdue clocks immediately—restart never resets a due time—and activate primary/backup routes under policy;
9. reconcile every pending or unknown consequential effect before permitting a new effect on the same business target;
10. verify each omitted-item reference needed for the next step and recompile the minimum context;
11. execute `next_safe_action` only if all named preconditions still hold; otherwise route to the deterministic correction, identity, operations, or clinician queue.

Timer delivery is at least once. `clock_id` and the transition predicate make firing idempotent. Store UTC instants plus the originating timezone/calendar/rule version when local business time matters; daylight-saving, holiday, working-hours, payer-response and safety-acknowledgment clocks must be testable without changing historical deadlines.

### Compaction invariants

Compaction must never remove:

- wrong-patient or merge/split warnings;
- proxy limits, revocation, purpose, or confidential-channel restrictions;
- clinical source versions;
- medication/diagnosis contradictions;
- safety flags and handoff ownership;
- effect IDs, approvals, receipts, unknown outcomes, or reconciliation deadlines;
- uncompleted tasks and dependencies;
- behavior/policy/adapter versions.

## Canonical seven-lifetime memory policy

“Memory” is not one feature.

| Lifetime | Use/reject and authority | Retention/deletion | Poisoning controls | Evaluation controls |
|---|---|---|---|---|
| Turn/scratch memory | Allow the minimum current request, closed actions and verified opaque refs; reject authority, durable facts and hidden carry-over | Process-only; destroy after response/abort and verify no prompt cache beyond processor contract | Delimit untrusted content, no tool credentials, schema/citation validation, discard on patient or authority change | Cross-turn leakage, injection, PHI minimization and forced-abort tests |
| Working/run memory | Allow typed candidate administrative facts, tool results and evidence refs; never promote without a validated event | Short explicit TTL; purge on run end, safety/identity invalidation or provider switch | Patient/tenant labels outside model, provenance and freshness on every item, immutable tool results, contradiction state | Duplicate/stale/tool-spoof/reordered-result and cross-patient contamination tests |
| Session memory | Constrain to channel continuity and a reference to the last verification event; reject reuse as identity, consent or clinical truth | Short channel TTL; delete on logout/expiry and reauthenticate or step up after wait/sensitivity | Bind tenant/actor/channel, do not store raw longitudinal history, invalidate on proxy/binding/purpose change | Session fixation, shared-device, actor switch, revoked-proxy and timeout tests |
| Durable workflow/task memory | Require typed cases, tasks, owners, clocks, approvals, effects, receipts, handoffs, contradictions and safety holds; this is coordination truth only | Records schedule by class/jurisdiction; corrections are append-only; deletion/legal-hold/export and backup propagation are tested | Command authorization, aggregate versions, tenant keys, constrained writers, event validation, reconciliation and tamper evidence | Replay, restart, correction, merge/split, concurrency, DR and invariant-property tests |
| Domain knowledge memory | Allow reviewed policy, payer/directory rules, templates, terminology and mappings; reject unversioned scraped guidance in live runs | Pin tenant/jurisdiction/effective period; publisher review/expiry; revoke and remove poisoned artifacts from retrieval | Signed/approved publication, source allowlist, content scanning, provenance, least-privilege publisher, rollback | Stale rule, malicious document, mapping drift, unsupported profile and rollback tests |
| Long-term/preference memory | Restrict to explicit authoritative language, channel, accessibility, accommodation and logistical preferences; reject inferred diagnosis, capacity, relationship or consent | Purpose-specific retention; patient correction/deletion and source invalidation propagate to replicas/indexes | Require source, actor, effective time and confidence class; never infer from names, behavior or model summaries | Correction latency, preference conflict, population/accessibility parity and unauthorized-proxy tests |
| Episodic/outcome memory | Allow only a governed offline corpus of synthetic/de-identified failure patterns and aggregate outcomes; reject raw live-case recollection and direct online learning | Dataset-specific approval/expiry; lineage, deletion and re-identification risk review; no automatic promotion | De-identification validation, rare-case review, incident-access controls, label provenance, train/eval separation | Membership/leakage, bias/slice coverage, label quality, regression utility and contamination tests |

Reject raw patient-history recollection in vector/chat stores and provider-side hidden state as continuity mechanisms. They are neither additional memory lifetimes nor sources of truth: prior transcripts, inferred diagnoses or relationships, and uninspectable model sessions cannot satisfy purpose, freshness, correction, or recovery requirements.

Semantic retrieval over PHI requires explicit use-case approval, tenant/patient filters enforced outside the model, document-level authorization, provenance, freshness, deletion/correction propagation, and leakage tests. It is not part of the MVP.

## Planning model

Planning is a closed administrative graph, not free-form clinical planning.

### Plan template

~~~yaml
coordination_plan:
  plan_id: opaque
  case_id: opaque
  case_version: integer
  objective: approved-workflow-outcome
  source_refs: [versioned-reference]
  steps:
    - step_id: opaque
      action: approved-enum
      owner_type: workflow | human
      preconditions: [approved-predicate]
      required_authority_tier: D1
      success_postcondition: approved-predicate
      failure_route: opaque
  maximum_steps: integer
  deadline_at: timestamp
  behavior_version: opaque
~~~

The model may fill an allowed next step, not invent an arbitrary graph. The workflow validates every step against the configured workflow definition.

### Bounded execution loop

~~~text
while case is open and budgets remain:
    acquire lease for current case version
    revalidate patient binding, authority, ownership, and safety state
    process deterministic events and timers
    if a rule selects the next transition:
        apply it
    else if bounded administrative ambiguity exists:
        assemble minimal context
        request one typed model proposal
        validate schema, citations, policy, clinical boundary, and state version
        accept, reject, or route the proposal
    else:
        wait or hand off
    create authorized effects through the effect service only
    verify receipts and postconditions
    checkpoint state
stop on safety hold, unknown effect, authority failure, contradiction, budget, or deadline
~~~

### Replanning triggers

Discard or revise a plan when:

- patient binding or representative authority changes;
- a source resource version changes;
- a contradiction appears;
- a clinician amends or cancels intent;
- a slot, destination, payer rule, or directory record changes;
- an effect becomes unknown;
- a safety trigger fires;
- an owner declines or misses a handoff;
- policy, adapter, or behavior version changes mid-case;
- a step/time/tool budget is exhausted.

Do not continue from an old model rationale after any trigger.

## Concurrency and ordering

- Partition or serialize commands by tenant plus case ID.
- Use aggregate versions on case/task commands.
- Use semantic effect IDs across retries and restarts.
- Deduplicate at event ingestion and preserve original event time.
- Do not assume messages from EHR, payer, and scheduler arrive in causal order.
- Record late events; do not silently discard them.
- Fence old workers after lease loss.
- A closed case may be reopened by an authorized event rather than mutating history.

## Stage 2–3 state gates

- [ ] All material state lives outside prompts and provider sessions.
- [ ] Domain, coordination, and conversational truth are distinguished.
- [ ] Events carry aggregate version, tenant, patient binding, correlation, causation, and behavior version.
- [ ] Context is purpose-minimized and source-versioned.
- [ ] Compaction preserves every safety, authority, task, effect, and contradiction invariant.
- [ ] All seven memory lifetimes have an explicit allow/restrict/reject decision.
- [ ] Planning uses closed actions and budgets.
- [ ] Replanning triggers invalidate stale proposals.
- [ ] Concurrency tests cover duplicate, late, reordered, and conflicting events.
- [ ] Cases resume safely after process, model-provider, and deploy interruption.

## Related guides

- Previous: [Scheduling, Referrals, Prior Authorization, Tasks, and Handoffs](04-scheduling-referrals-prior-authorization-tasks-and-handoffs.md)
- Next: [Tools, Effects, Idempotency, Reconciliation, and Recovery](06-tools-effects-idempotency-reconciliation-and-recovery.md)
- Home: [Healthcare Care Coordination Agent](README.md)
- Canonical: [Agent State and Event Contracts](../../runtime/agent-state-and-event-contracts.md)

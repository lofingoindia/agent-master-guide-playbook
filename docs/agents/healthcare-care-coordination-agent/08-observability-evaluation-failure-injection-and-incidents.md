# Observability, Evaluation, Failure Injection, and Incidents

Production evidence must show not only that the model usually proposes useful work, but that the whole system preserves patient identity, authority, clinical boundaries, safety escalation, durable closure, and recoverability under failures.

## What to observe

Observe five linked layers:

| Layer | Primary question | Evidence |
|---|---|---|
| Case | Did coordination reach a valid outcome? | Case/task/handoff states and closure evidence |
| Decision | Why was a route or proposal accepted? | Context manifest, deterministic rule, model proposal, validator decision |
| Authority | Who could read/act, for what purpose, on which patient? | Binding, policy decision, approval, D-tier |
| Effect | What external change occurred? | Effect state, request ID, receipt, postcondition, reconciliation |
| Safety | Did the system detect, route, acknowledge, and control hazards? | Safety hold, handoff, acknowledgment, backup route, hazard/control signals |

A model trace alone cannot answer these questions.

## Trace contract

~~~yaml
coordination_trace_span:
  trace_id: opaque
  span_id: opaque
  parent_span_id: optional
  tenant_ref: opaque
  case_id: opaque
  workflow_kind: approved-enum
  patient_binding_ref: opaque
  case_version: integer
  operation: context-assemble | model-propose | validate | effect-dispatch | reconcile | handoff
  authority_decision_ref: optional
  effect_id: optional
  handoff_id: optional
  safety_state: normal | hold
  behavior_version: opaque
  model_route: optional-opaque
  adapter_version: optional
  outcome_code: approved-enum
  latency_ms: integer
  token_counts: optional-aggregate
  occurred_at: timestamp
~~~

Use opaque references. Keep raw names, dates of birth, notes, message text, prompts, responses, attachments, and clinical details out of standard telemetry.

The restricted audit/effect ledger stores enough evidence to reconstruct an authorized event; the telemetry platform stores enough to operate the service. These have different access, retention, integrity, and incident requirements.

## Event-to-trace correlation

~~~mermaid
sequenceDiagram
    participant W as Workflow
    participant C as Context assembler
    participant M as Model
    participant V as Validator
    participant E as Effect service
    participant R as Reconciler
    participant O as Telemetry

    W->>O: case transition span
    W->>C: case version and policy refs
    C->>O: context manifest ref and freshness metrics
    C->>M: minimized context
    M-->>V: typed proposal
    V->>O: acceptance or rejection code
    V-->>W: validated transition
    W->>E: authorized semantic effect
    E->>O: dispatch outcome without PHI
    E-->>R: receipt or unknown
    R->>O: postcondition and reconciliation status
    R-->>W: verified outcome
~~~

Each span carries case, behavior, and effect references so an investigator can cross the boundary into restricted evidence with appropriate authorization.

## SLO framework

There is no universal clinical timing target. Clinical governance and operations must define targets per workflow, population, priority source, hour of operation, and safety route.

### SLO template

| Indicator | Population/condition | Target | Exclusions | Error-budget action |
|---|---|---|---|---|
| Safety handoff acknowledgment latency | Governance-defined trigger cases | Locally approved | Explicit disaster mode only | Freeze rollout; activate backup staffing |
| Referral exception notification | Accepted referral missing expected progress | Locally approved | Source outage classified separately | Shift to manual queue |
| Task closure integrity | Cases marked closed | 100% satisfy invariants | None | Reopen/reconcile; incident review |
| Wrong-patient effect/disclosure | All runs | 0 tolerated | None | Kill switch and incident |
| Unauthorized proxy effect/disclosure | All proxy runs | 0 tolerated | None | Kill switch and incident |
| Unknown-effect age | All D3 effects | Locally approved bound | None | Disable affected writes |
| Model proposal validity | Per workflow/language/slice | Release-specific | Deterministic path excluded | Route more cases to humans |
| End-to-end coordination latency | Workflow and operational priority | Locally approved | Patient-requested delay | Capacity or process correction |
| Accessible fallback availability | Applicable interactions | Release-specific | None | Disable inaccessible automated path |
| Audit completeness | Reads/effects/approvals/handoffs | 100% required fields | None | Stop affected action |

Hard safety/privacy invariants are not tradable error budgets.

### Service indicators

Measure:

- intake, identity, authority, source-read, model, queue, adapter, and reconciliation latency;
- open cases/tasks by age, owner, workflow, and safety state;
- handoff acknowledgment and decline/overdue counts;
- effects by proposed/authorized/dispatched/succeeded/failed/unknown/diverged;
- binding review, proxy denial, consent conflict, and source contradiction rates;
- deterministic resolution, model invocation, abstention, validator rejection, and human rework;
- channel delivery/acknowledgment without message content;
- accessibility/language fallback use and completion;
- tokens, tool calls, compute time, queue occupancy, and cost per valid closed case;
- behavior/model/adapter/policy version.

Do not put patient, diagnosis, medication, provider, or message values in metric labels.

## Evaluation architecture

~~~mermaid
flowchart LR
    S[Synthetic and governed de-identified cases] --> H[Scenario harness]
    F[Failure and incident corpus] --> H
    A[Adversarial injections] --> H
    H --> R[Deterministic replay and repeated model trials]
    R --> J[Automated contract judges]
    R --> C[Clinical, privacy, identity, accessibility, operations review]
    J --> G[Hard gates and scorecard]
    C --> G
    G --> B[Behavior release decision]
    B --> M[Canary monitoring]
    M --> F
~~~

The harness mocks or sandboxes domain systems and records every state transition and effect attempt. D3 tests never contact production patients or providers.

### Four-plane evaluation contract

Do not collapse evaluation into final-answer accuracy.

| Plane | Unit of analysis | Required measures | Example hard evidence |
|---|---|---|---|
| Outcome | Completed or blocked coordination case | Valid-closure rate, time to verified outcome, avoidable abandonment, patient/care-team recontact, human rework, cost per valid closed case | Source receipt/postcondition, open-work invariant check, accountable owner and patient communication state |
| Trajectory | Ordered events, reads, proposals, approvals, effects, waits and handoffs | Unsupported-read rate, unnecessary model/tool steps, abstention/route correctness, stale-source use, approval-to-effect drift, unknown-effect age, loop/budget exhaustion | Full event/effect replay shows no prohibited transition even when the final state appears correct |
| Invariant | Every state transition and effect boundary | Count and denominator for patient/tenant/authority/source/safety/idempotency/closure predicates; hard failures remain zero | Property checks run after every event, injected crash and replay—not sampled only at case end |
| Human factor | Patient, representative, coordinator, scheduler, clinician, identity/privacy reviewer and incident responder interaction | Time to notice/accept/resolve, queue age, interruption rate, override appropriateness, handoff clarity, workload minutes, accessibility success, comprehension/fallback, alarm fatigue and automation-bias indicators | Observed usability/assistive-technology exercise plus queue/decision records; a generic “human reviewed” flag is insufficient |

Every evaluation result is reconstructable:

~~~yaml
evaluation_result:
  evaluation_suite_version: opaque
  behavior_manifest_version: opaque
  scenario_id: opaque
  scenario_source_class: synthetic | governed-deidentified | incident-derived
  population_and_access_slices: [approved-enum]
  trial_id: opaque
  trajectory_ref: restricted-immutable-ref
  source_fixture_versions: [opaque]
  injected_failures: [approved-enum]
  invariant_results: [{invariant_id: opaque, checks: integer, failures: integer}]
  outcome: valid-closed | valid-blocked | invalid | unresolved
  human_observation_refs: [restricted-opaque]
  hard_gate_failures: []
  reviewer_roles: [approved-enum]
  evaluated_at: timestamp
~~~

The evidence plane stores scenario definitions, versioned fixtures, deterministic assertions, repeated-trial results, human review, hazard/control mappings and immutable release decisions. It links to restricted PHI only through approved opaque references. Production traces are operational signals; they do not become training/evaluation data automatically.

## Evaluation suite

| Suite | What it proves | Examples |
|---|---|---|
| Deterministic contract | Schemas, state transitions, policy, idempotency | Missing field, stale version, duplicate effect |
| Identity/authority | Correct abstention and point-of-use checks | Near match, revoked proxy, role change |
| Administrative quality | Correct closed-set proposal and evidence | Missing referral attachment, payer requirement mapping |
| Clinical-boundary | No diagnosis, treatment, medication, urgency, disposition | Adversarial request to advise or infer |
| Safety escalation | Trigger, hold, routing, acknowledgment, backup | Concerning message and queue outage |
| Provenance | Every material assertion maps to supplied versioned evidence | Changed document/version or fabricated citation |
| Workflow closure | No lost task, handoff, referral, or authorization | Late response, reopen, absent receipt |
| Effect reliability | No duplicate and correct unknown handling | Timeout after commit, reordered webhook |
| Security/privacy | Least privilege, tenant isolation, untrusted-content resistance | Injection, overbroad FHIR query, wrong recipient |
| Accessibility/language | Equivalent task completion and human alternative | Screen reader, timeout extension, translation ambiguity |
| Load/recovery | SLO and invariants under saturation/restart | Queue spike, dependency brownout, regional failover |
| Change regression | Old hazards remain controlled under new behavior | Model/adapter/policy/terminology upgrade |

## Required dataset slices

At minimum:

- patient self-service, representatives, caregivers without authority, minors/dependent scenarios, workforce users, and service principals;
- exact matches, near matches, duplicates, merged/split records, disputed identity, and stale portal binding;
- referral, scheduling, prior authorization, care-task, outreach, and handoff;
- stable, stale, missing, contradictory, malformed, and malicious evidence;
- clinical source changes between context, approval, and effect;
- multiple languages, screen-reader/keyboard flows, low digital access, alternate channel, and communication restrictions;
- nights/weekends, owner turnover, full queues, unavailable destinations, and vendor outages;
- common and rare cases, not only average-volume workflows.

Report every metric by relevant slice. A high aggregate score can conceal a wrong-patient or accessibility failure.

For each material slice, publish numerator, denominator, uncertainty interval where meaningful, abstentions/routing, missing-data rate, human minutes and tail latency. Compare the worst supported slice with the overall population and with the deterministic/manual baseline. If a slice is too small to estimate safely, label it insufficient evidence and constrain rollout; do not hide it in the aggregate.

## Hard release failures

One occurrence blocks release or triggers rollback:

- wrong-patient read, disclosure, or effect;
- unauthorized representative or cross-tenant access;
- autonomous diagnosis, treatment selection, prescribing, dose change, clinical urgency, or emergency disposition;
- fabricated clinical evidence, code, order, priority, or medical-necessity claim;
- missed or late required safety route;
- routine effect while safety hold is active;
- duplicate appointment, referral, authorization submission, cancellation, or PHI-bearing message;
- task/referral/handoff silently lost or falsely closed;
- unknown D3 effect aging beyond its approved reconciliation limit;
- raw PHI reaching unapproved logs, evaluation stores, model training, or processors;
- D4 authority/policy change executed by the model.

These gates coexist with graded quality, latency, and cost metrics.

## Failure injection matrix

| Injection | Expected system behavior | Evidence |
|---|---|---|
| Two plausible patient matches | Stop before PHI/context/effect; identity queue | No downstream call; binding review event |
| Merge event after approval | Invalidate approval; stop or reconcile effect | Case safety/identity state and audit |
| Proxy revoked while waiting | Re-evaluate and deny on resume | Policy versions and no disclosure |
| Prompt injection in referral PDF | Ignore instruction; cite as untrusted data only | Proposal/validator trace |
| ServiceRequest version changes | Reject stale plan and reload | Version conflict event |
| Slot taken after display | Booking conflict; offer fresh choices | No automatic substitute |
| Timeout after booking commits | Unknown then reconcile to one appointment | Same effect ID and receipt |
| Duplicate webhook | Deduplicate without duplicate task/effect | Event ledger |
| Payer returns unmapped status | Preserve unknown and route | No coerced approval/denial |
| Communication says delivered but wrong address version | Incident/unknown; stop follow-up automation | Address/receipt comparison |
| Primary clinical queue unavailable | Backup route and operations alert | Safety acknowledgment trace |
| Model recommends medication change | Validator blocks; clinical boundary hard failure | Evaluation result |
| FHIR server rate limits | Bounded retry and queue backpressure | Retry/queue metrics |
| Model provider outage | Deterministic paths continue; ambiguous cases wait/manual | No case loss |
| Region fails with in-flight effects | Recover state, fence old workers, reconcile | DR exercise evidence |
| Telemetry exporter fails | Core workflow continues within local buffer policy; audit preserved | Gap alert and replay |

## Clinical-safety evaluation

Clinical safety evaluation asks whether the administrative system can create or fail to control a pathway to harm. Use the hazard log to generate cases.

For each hazard:

1. model the initiating causes and hazardous state;
2. identify preventive, detective, mitigative, and recovery controls;
3. map each control to unit, integration, scenario, load, or drill evidence;
4. test control independence where claimed;
5. inject failure of each dependency and human queue assumption;
6. have appropriate clinical/safety experts review outcomes;
7. record residual risk and named acceptance;
8. re-run on behavior, model, adapter, workflow, population, or policy change.

The model does not grade its own clinical safety. Automated judges can check schemas and citations; qualified humans review clinical-boundary and hazard outcomes.

## Repeated-trial evaluation

Nondeterministic behavior needs distributions:

- run each critical scenario enough times to observe variability;
- pin prompts, tools, policies, sources, and model identifiers;
- record temperature/sampling controls where exposed;
- calculate exact hard-gate counts plus confidence intervals for graded metrics;
- compare worst slice and tail latency, not only the mean;
- inspect trajectory differences even when final status matches;
- test model refusal/abstention and over-compliance;
- repeat after provider-side changes or unexplained drift.

Never promote a model on a single golden run.

## Online monitors and release guardrails

Use:

- shadow mode before effect authority;
- staff-only pilot;
- tenant/workflow/population allowlists;
- percentage or queue-based canary;
- stricter D-tier during rollout;
- automatic rollback on hard invariants;
- circuit breakers per adapter, channel, action, and behavior;
- daily review of unknown/diverged effects and safety handoffs;
- drift alerts for model rate, rejection, abstention, evidence errors, human rework, and slice performance.

Keep a manual path throughout.

## Governed failure mining

Mine failures to improve controls without converting production PHI into uncontrolled memory:

1. detect candidate cases from invariant failure, unknown/diverged effect, correction/reopen, overdue handoff, safety event, complaint, accessibility failure, human rework code, drift or incident;
2. quarantine the case reference and preserve source/behavior/adapter versions under restricted access;
3. have the appropriate clinical-safety, privacy, identity, accessibility or operational owner label the failure mechanism—not just the model output;
4. minimize and de-identify, or synthesize an equivalent case; record residual re-identification and representativeness limits;
5. map it to a hazard, control and affected trajectory transition;
6. check for cohort/systematic exposure and start patient/care-team follow-up through accountable operations where needed;
7. add the approved artifact to a versioned regression corpus with retention, deletion and access policy;
8. prefer a deterministic workflow, adapter, policy or UI fix when the failure has a stable cause;
9. evaluate the entire behavior bundle and affected population/accessibility slices;
10. canary, monitor and close the learning item only when control evidence and incident actions are complete.

Do not use reviewer free text, raw chats, full notes, incident tickets or “thumbs down” payloads as direct fine-tuning/retrieval data. Track selection bias: monitored failures overrepresent detectable events, digital users and existing channels, while people who abandoned the process or lacked access may be missing.

## Incident severity examples

Local incident taxonomy governs final severity. The table shows routing, not universal labels.

| Event | Immediate action | Required owners |
|---|---|---|
| Wrong-patient effect/disclosure | Disable affected reads/writes, preserve evidence, reconcile impacted cases | Clinical safety, privacy, security, identity, operations |
| Missed safety escalation | Activate manual/backup review and identify affected cases | Clinical safety, care service, operations |
| Model clinical advice | Disable behavior/action; review exposure and downstream actions | Clinical, regulatory/legal, product, model governance |
| Duplicate booking/submission | Stop adapter effect, reconcile and correct | Operations, integration owner, care team |
| Cross-tenant retrieval | Isolate service, revoke credentials, preserve audit | Security, privacy, platform |
| Inaccessible path causes failed coordination | Provide human alternative and assess affected cohort | Accessibility, operations, clinical safety |
| Audit gap | Stop affected D3 action if reconstruction is impossible | Security, privacy, compliance, operations |

## Incident workflow

~~~mermaid
flowchart TD
    D[Detection or report] --> C[Contain affected action, adapter, behavior, or tenant]
    C --> P[Preserve audit, state, effects, and source versions]
    P --> T[Parallel technical, privacy, safety, and operational triage]
    T --> A[Identify affected cases and in-flight unknowns]
    A --> R[Clinical/human follow-up and reconciliation]
    R --> F[Fix with regression and hazard tests]
    F --> G[Governed staged restoration]
    G --> L[Post-incident learning and control update]
~~~

Do not copy raw PHI into the incident ticket. Use restricted case references and controlled evidence access.

## Operational runbooks

Maintain tested runbooks for:

- patient-binding incident or merge/split burst;
- proxy/consent policy error;
- safety queue/backup failure;
- wrong-recipient communication;
- scheduling duplicate or unknown effect;
- referral backlog and lost-closure signal;
- payer adapter status mismatch;
- EHR/HIE/scheduler/provider outage;
- model clinical-boundary breach;
- prompt injection or exfiltration attempt;
- telemetry/audit pipeline failure;
- regional failover and old-worker fencing;
- behavior rollback and case migration.

Each runbook lists detection, containment, responsible roles, evidence, patient/care-team follow-up, reconciliation query, rollback, restoration gate, and required notification assessment.

## Evaluation and observability checklist

- [ ] Trace schema links case, authority, proposal, effect, handoff, safety, and behavior without raw PHI.
- [ ] Restricted audit/effect evidence is distinct from operations telemetry.
- [ ] SLOs are workflow/population-specific and hard invariants are non-budgetable.
- [ ] Evaluation includes deterministic, model, integration, safety, accessibility, load, and recovery layers.
- [ ] Results are stratified and repeated, with tail behavior reported.
- [ ] Every hazard control maps to evidence.
- [ ] Failure injection includes timeouts before/after commit and human-queue failure.
- [ ] Shadow, canary, circuit breaker, kill switch, rollback, and manual path are tested.
- [ ] Incident routing spans clinical safety, privacy, security, accessibility, and operations.
- [ ] Post-incident cases enter the governed regression corpus without uncontrolled PHI retention.

## Related guides

- Previous: [Security, Privacy, Accessibility, and Clinical Safety](07-security-privacy-accessibility-and-clinical-safety.md)
- Next: [Deployment, Scale, Capacity, Cost, and Behavior Evolution](09-deployment-scale-capacity-cost-and-behavior-evolution.md)
- Home: [Healthcare Care Coordination Agent](README.md)
- Canonical: [Observability and Tracing](../../evaluation/observability-and-tracing.md)

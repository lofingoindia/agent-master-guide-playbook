# Stage 0–6 Delivery Gates, Schemas, and Checklists

This capstone converts the blueprint into a progressive delivery plan. Each stage adds only the complexity justified by the prior stage's evidence. Passing Stage 6 means the system is governed and continuously verified—not clinically autonomous.

## Stage map

~~~mermaid
flowchart LR
    S0[Stage 0<br/>Qualify] --> S1[Stage 1<br/>Bounded agent]
    S1 --> S2[Stage 2<br/>Useful MVP]
    S2 --> S3[Stage 3<br/>Reliable v1]
    S3 --> S4[Stage 4<br/>Production readiness]
    S4 --> S5[Stage 5<br/>Scale and resilience]
    S5 --> S6[Stage 6<br/>Governed evolution]
~~~

| Stage | Capability added | Complexity deliberately deferred |
|---|---|---|
| 0 | Scope, deterministic baseline, authority and hazard framing | Model, writes, broad integrations |
| 1 | One bounded model proposal inside a sandboxed workflow | D3 effects, long-term memory, multi-agent |
| 2 | One useful end-to-end workflow with controlled human/staged effects | Broad payer/EHR/channel coverage |
| 3 | Durable state, compaction, effects, reconciliation, recovery | Multi-region and dynamic routing |
| 4 | Production identity, privacy, safety, observability, SLOs, incidents | Scale-only infrastructure |
| 5 | Partitioning, capacity, degradation, DR, and cost controls | Autonomous self-improvement |
| 6 | Behavior releases, governed feedback, refresh, and deprecation | Clinical authority and D4 autonomy |

## Required exercises and evidence

Thresholds for latency, workload improvement, slice equivalence and capacity are local risk decisions. Every stage nevertheless has a measurable exercise with a predeclared population, sample/repetition count, pass rule and owner; “demo looked good” is never evidence.

| Stage | Entry evidence | Required exercise | Exit evidence |
|---|---|---|---|
| 0 | Current manual/rules process, representative case inventory, workload volumes, owner map and preliminary hazards | Replay a governed representative sample through the deterministic baseline; label ambiguity, source/owner, completion evidence, delays, rework, access failures and cases needing clinical judgment | Baseline numerator/denominator for valid closure, delay, abandonment and human minutes; signed intended/excluded use; decision showing where a model adds bounded value |
| 1 | Approved minimized fixtures, one proposal schema, hard-gate scenarios and provider data contract | Run predeclared repeated trials for every normal, ambiguous, malicious, identity/proxy and clinical-boundary case; switch model/provider process off mid-run | Zero hard failures; distributions and intervals for proposal/evidence/abstention quality; context and provider-retention evidence; complete trajectory replay |
| 2 | Stage 1 bundle, sandbox adapters, staffed human/safety queues, accessible alternate path | Complete one workflow in shadow/staff pilot including proxy, missing evidence, referral or authorization exception, slot race, channel failure and safety hold | Every pilot case is validly closed or visibly owned/blocked; all D2/D3 work has receipt/postcondition; comparison with deterministic/manual baseline and recorded human rework |
| 3 | Versioned states/events, operation manifests, effect ledger, reconciler and continuation receipt | Inject process death at every state/outbox/dispatch/receipt boundary; duplicate/reorder events; revoke proxy; merge/split patient; change source; compact/resume; fire overdue clocks | No lost task/handoff, duplicate effect, stale-authority use or reset deadline; every unknown reconciled/escalated; fail-closed resume and correction-propagation receipts |
| 4 | Approved safety case, data-flow/threat models, operation qualification, SLOs and incident roles | Conduct wrong-patient, unauthorized-proxy, cross-tenant, injection, inaccessible-path, missed-safety-route, audit-loss and whole-bundle rollback exercises with humans | Zero hard failures, 100% required restricted-ledger fields, hazard-control evidence, primary/backup acknowledgment results, assistive-technology findings, incident and recovery timeline |
| 5 | Measured arrival/service distributions, quotas, staffing/coverage, RTO/RPO and forecast | Load at the declared peak/burst/recovery envelope while degrading each dependency and a region; continue arrivals during backlog drain; saturate human and reconciliation queues | SLO/error-budget report by slice, no starvation of safety/reconciliation, measured drain time and recovery load, achieved RTO/RPO, effect classification, capacity/cost per valid closed case |
| 6 | Immutable previous/candidate bundles, affected regression/hazard map, canary/rollback plan and migration policy | Shadow then canary the whole bundle; inject a hard-gate signal and execute rollback with old/new in-flight cases and unknown effects | Promotion/rollback decision with exact bundle hashes, slice/trajectory comparison, fenced workers, reconciled effects, resume-invariant results, governed failure-corpus and safety-case updates |

For each exercise retain the scenario/case set version, environment, behavior manifest, adapter/contract versions, start/end times, injected faults, expected invariants, actual event/effect trajectory, human participants/queues, result, deviations, owner sign-off and expiry/refresh trigger.

## Stage 0 — Qualify the workload

### Stage 0 deliverables

- one narrow organization/workflow/population/jurisdiction/channel statement;
- explicit administrative intended use and prohibited clinical uses;
- deterministic state machine and current baseline measurements;
- category-separation and ownership map;
- D0–D4 action inventory;
- authoritative source map;
- preliminary data-flow and hazard analysis;
- named identity, privacy, clinical, safety, accessibility, integration, and operational owners;
- success outcomes and hard failures.

### Stage 0 exit gate

- [ ] Real cases demonstrate recurring evidence-bounded administrative ambiguity.
- [ ] A model is not being used for structured validation or simple routing.
- [ ] Clinical intent, diagnosis, treatment, medication, urgency, and emergency disposition are outside model authority.
- [ ] Patient matching, consent/proxy authority, and care-team ownership have established accountable processes.
- [ ] Every proposed tool and integration has a clear reason.
- [ ] No uncontrolled open-web or arbitrary tool use exists in a patient run.
- [ ] Manual operations can complete the workflow.
- [ ] Product, clinical-safety, privacy/legal, accessibility, and security reviewers accept the intended-use boundary.

### Stage 0 stop condition

Stop if the value proposition requires clinical advice, automatic ambiguous-patient resolution, inferred representative authority, or an unattended consequential browser workflow with no verifiable outcome.

## Stage 1 — First bounded agent

### Stage 1 deliverables

- offline or read-only pilot;
- one proposal schema and closed action set;
- minimized context assembler with source/version lineage;
- deterministic proposal validator;
- one approved model route with provider/privacy review;
- step, time, token, tool, and data budgets;
- abstention and human/clinical routes;
- initial synthetic/de-identified evaluation set;
- no D3 authority.

### Stage 1 exit gate

- [ ] The model receives only the minimum evidence for one step.
- [ ] Every material claim cites a supplied source version.
- [ ] Identity, proxy, clinical, and unknown cases abstain correctly.
- [ ] Embedded instructions cannot change tools, policy, or authority.
- [ ] Output schema and allowed actions are enforced outside the model.
- [ ] Deterministic paths bypass the model.
- [ ] No model/provider hidden state is required for continuity.
- [ ] Clinical-boundary hard gates pass across repeated trials.
- [ ] PHI processing contract, retention, training-use, region, and subprocessors are approved.

### Stage 1 stop condition

Stop if the model needs broad chart access, regularly invents clinical content, or cannot be reliably constrained to the closed administrative proposal.

## Stage 2 — Useful MVP

### Stage 2 deliverables

Choose one end-to-end path such as referral closure:

- already-authenticated actor and verified patient binding;
- point-of-use consent/representative/purpose check;
- read-only EHR/FHIR and directory/scheduling status;
- administrative gap proposal;
- durable tasks, owner, deadline, and human wait;
- D2 draft or provisional action;
- optionally one narrowly preauthorized D3 effect with exact verification;
- minimal approved communication;
- safety hold and staffed clinical route;
- workflow-level success and human-rework measurement.

### Stage 2 exit gate

- [ ] One real workflow reaches a verifiable outcome with no chat-only state.
- [ ] Clinician-authored intent and priority remain unchanged.
- [ ] The patient/representative can reach a human and use an accessible alternate path.
- [ ] Communication request, dispatch, delivery, and acknowledgment are distinct.
- [ ] Scheduling distinguishes slot observation from booking.
- [ ] Referral closure requires destination/owner evidence.
- [ ] All D2 state expires or rolls back safely.
- [ ] Any D3 effect uses an exact semantic ID, authority/approval, receipt, and postcondition.
- [ ] Safety triggers halt routine work and use a tested clinical route.
- [ ] Hard release failures remain zero in the approved evaluation scope.

### Stage 2 stop condition

Do not add workflows until one narrow MVP proves better closed-loop outcomes than the deterministic/manual baseline without weakening safety, privacy, access, or recoverability.

## Stage 3 — Reliable v1

### Stage 3 deliverables

- explicit case/task/handoff state machines;
- versioned events and optimistic concurrency;
- durable patient-binding and authority references;
- all memory classes decided and enforced;
- checkpoint/compaction continuation packages;
- effect ledger, transactional outbox, and fenced dispatch;
- semantic idempotency and downstream capability tests;
- unknown-outcome and reconciliation queues;
- source contradictions and invalidation;
- cancellation/compensation semantics;
- restart, deploy, model outage, and dependency outage recovery.

### Stage 3 exit gate

- [ ] Process, worker, channel, model, and deploy restarts lose no required state.
- [ ] Compaction preserves identity, authority, source versions, tasks, effects, unknowns, safety, and handoffs.
- [ ] Patient merge/split and proxy revocation invalidate dependent state.
- [ ] Duplicate, late, and reordered events do not create duplicate effects or false closure.
- [ ] Unknown outcomes are never blindly retried.
- [ ] Reconciliation has owners, deadlines, and incident escalation.
- [ ] Medication/diagnosis contradictions route to clinicians rather than model resolution.
- [ ] A behavior version can reconstruct every decision/effect.
- [ ] All closure invariants are enforced by the workflow.

### Stage 3 stop condition

Do not call the system reliable while any consequential result depends on a transcript, one process, a provider session, or a best-effort webhook.

## Stage 4 — Production readiness

### Stage 4 deliverables

- production identity assurance and patient-matching integration;
- versioned consent/proxy/purpose and special-data policies;
- least-privilege workforce/service access and secrets;
- processor agreements and verified provider settings;
- PHI data-flow/retention/deletion/correction controls;
- clinical safety case and hazard log;
- accessibility/language validation;
- PHI-safe trace, restricted audit, and effect evidence;
- workflow and safety SLOs;
- failure-injection and load tests;
- incident routes, runbooks, kill switches, rollback, and manual fallback;
- staff training and operational readiness.

### Stage 4 exit gate

- [ ] Wrong-patient, unauthorized proxy, cross-tenant, clinical-advice, and PHI-leak tests pass with zero hard failures.
- [ ] Every hazard control maps to current evidence and a named owner.
- [ ] Primary and backup safety routes meet locally approved exercises.
- [ ] D3 effects can be disabled per action/adapter/tenant/behavior.
- [ ] Traces omit raw PHI while restricted evidence supports reconstruction.
- [ ] Accessibility and language paths work with assistive technology and human alternatives.
- [ ] Incident drills cover wrong patient, missed safety route, duplicate/unknown effect, provider outage, and behavior rollback.
- [ ] Regulatory/intended-use and legal reviews are current.
- [ ] A production readiness review accepts residual risk.

### Stage 4 stop condition

Do not expose the workflow to patients or D3 production effects without a staffed operating model and tested cross-domain incident response.

## Stage 5 — Scale and resilience

### Stage 5 deliverables

- tenant/case partitioning and workload cells;
- separate safety, ordinary, batch, effect, and reconciliation queues;
- admission control and weighted fairness;
- dependency quotas, backpressure, circuit breakers, and degraded modes;
- measured capacity model including human/safety queues;
- high-availability durable stores and tested restoration;
- region/data-residency plan where justified;
- fenced failover and in-flight effect reconciliation;
- cost per valid closed case and slice;
- archive/retention capacity planning.

### Stage 5 exit gate

- [ ] Burst tests include model, EHR, scheduler, payer, communication, human, safety, database, and reconciliation bottlenecks.
- [ ] No tenant or batch workload can starve safety or reconciliation work.
- [ ] Autoscaling preserves per-case serialization and effect uniqueness.
- [ ] Degraded modes make honest claims and never silently shed safety/audit/reconciliation.
- [ ] Disaster-recovery exercises meet hazard-informed RTO/RPO.
- [ ] Old workers are fenced and all D3 effects are classified after failover.
- [ ] Backlog drain time is measured under continued arrivals.
- [ ] Capacity includes accessibility/language and manual fallback demand.
- [ ] Cost reductions do not weaken hard controls.

### Stage 5 stop condition

Do not add active-active writes, more models, or dynamic agents merely to solve a queue or data-model problem.

## Stage 6 — Continuous governed evolution

### Stage 6 deliverables

- immutable behavior manifest;
- change classification and accountable approvers;
- full evaluation bundle per release;
- shadow, pilot, canary, monitored expansion, rollback;
- explicit in-flight case migration policy;
- de-identified/synthetic incident and near-miss corpus;
- deterministic-first remediation process;
- drift, human-rework, slice, safety, and cost monitors;
- source and law refresh calendar;
- workflow/model/adapter/policy deprecation plan;
- periodic safety-case and access review.

### Stage 6 exit gate

- [ ] Model, prompt, schema, workflow, policy, adapter, profile, terminology, template, and safety-rule versions are pinned.
- [ ] Provider-side model changes trigger evaluation.
- [ ] Every release passes affected hard gates and repeated-trial slice comparisons.
- [ ] No live patient interaction updates production behavior automatically.
- [ ] Incidents and near misses become governed regression cases.
- [ ] New population, jurisdiction, channel, clinical workflow, or intended use receives full requalification.
- [ ] Rollback and reconciliation are tested for in-flight cases.
- [ ] Stale source material and behavior versions have owners and deprecation dates.
- [ ] D4 policy and authority changes remain human/system-controlled.

### Stage 6 stop condition

Stage 6 never authorizes diagnosis, treatment, prescribing, dose changes, emergency disposition, identity merge, or self-expanding authority. Those remain outside this blueprint.

## Reference invariants

Use these as database constraints, command predicates, and tests:

~~~text
effect_requires(current_patient_binding)
effect_requires(current_authority_decision)
D3_requires(exact_approval OR narrow_preauthorization)
D4_execution_by_model == false
clinical_assertion_requires(versioned_source)
model_clinical_decision == false
routine_effect_when_safety_hold == false
unknown_effect_retry_without_reconciliation == false
case_closed_implies(no_open_required_tasks)
case_closed_implies(no_open_required_handoffs)
case_closed_implies(no_unresolved_material_contradictions)
case_closed_implies(all_D3_effects_verified_or_resolved)
~~~

## Minimum schema set

| Contract | Required purpose | Canonical detail |
|---|---|---|
| Patient binding | Bind one tenant-qualified source patient with assurance and invalidation | [Identity guide](03-patient-identity-consent-proxy-and-care-team-authority.md) |
| Identity/version/time envelope | Separate source/business identity, record/local version, effective/clinical time, recorded/observed time and correction lineage for every domain object | [State guide](05-state-events-context-memory-and-planning.md) |
| Authority decision/grant | Bind actor, purpose, action, data, destination, restrictions, and version | [Identity guide](03-patient-identity-consent-proxy-and-care-team-authority.md) |
| Care-team assignment | Make ownership, coverage, and acceptance explicit | [Identity guide](03-patient-identity-consent-proxy-and-care-team-authority.md) |
| Coordination case/task | Persist workflow state, owner, deadline, dependencies, and closure | [State guide](05-state-events-context-memory-and-planning.md) |
| Clinical evidence reference | Preserve exact source/version/provenance and supported assertion | [Workflow guide](04-scheduling-referrals-prior-authorization-tasks-and-handoffs.md) |
| Contradiction | Keep conflicting facts visible until authorized resolution | [State guide](05-state-events-context-memory-and-planning.md) |
| Event envelope | Preserve order, aggregate version, correlation, causation, and behavior | [State guide](05-state-events-context-memory-and-planning.md) |
| Context/continuation | Prove bounded evidence and survive compaction | [State guide](05-state-events-context-memory-and-planning.md) |
| Tool/effect/approval | Enforce D-tier, idempotency, receipt, and postcondition | [Effects guide](06-tools-effects-idempotency-reconciliation-and-recovery.md) |
| Reconciliation | Resolve unknown/divergent downstream outcomes | [Effects guide](06-tools-effects-idempotency-reconciliation-and-recovery.md) |
| Communication/handoff | Separate delivery from acceptance and transfer ownership | [Workflow guide](04-scheduling-referrals-prior-authorization-tasks-and-handoffs.md) |
| Safety escalation/hazard | Freeze routine work and track accountable safety controls | [Safety guide](07-security-privacy-accessibility-and-clinical-safety.md) |
| Trace/behavior manifest | Operate and reconstruct the exact release | [Evaluation guide](08-observability-evaluation-failure-injection-and-incidents.md), [deployment guide](09-deployment-scale-capacity-cost-and-behavior-evolution.md) |

## Production acceptance matrix

| Domain | Required proof | Reject when |
|---|---|---|
| Scope | Signed intended/excluded use and owner map | “Assistant can help with healthcare” is the scope |
| Deterministic baseline | Measured rule/manual comparator | No evidence a model is needed |
| Identity | Near-match, merge/split, correction, namespace tests | Model chooses ambiguous patient |
| Consent/proxy | Point-of-use purpose/action/data/destination tests | Boolean consent or relationship implies all access |
| Clinical boundary | Repeated adversarial hard-gate suite | Any clinical recommendation or fabricated evidence |
| Workflows | Source-to-closure scenarios and exceptions | Send/status is treated as completion |
| State/memory | Restart/compaction/provider-switch replay | Transcript or hidden memory is required |
| Tools/effects | Timeout-after-commit, duplicate, conflict, reconciliation tests | Unknown is retried blindly |
| Security/privacy | Tenant, least-privilege, injection, log/processor tests | Raw PHI escapes approved path |
| Safety | Hazard log, safety route and queue-failure drills | Agent decides disposition or routine work continues |
| Accessibility | Assistive, language, alternate-channel and human-path tests | One digital channel is required |
| Observability | PHI-safe trace plus restricted reconstruction | Metrics cannot link to effects/behavior |
| Incidents | Cross-domain containment/recovery exercises | No patient/care-team follow-up process |
| Scale | Burst, backpressure, fairness, recovery, cost evidence | Model throughput hides human/reconciliation saturation |
| Evolution | Immutable manifest, evaluation, canary, rollback | Silent self-update or unpinned dependency |

## Pre-production master checklist

### Product and governance

- [ ] Intended use, population, workflow, setting, channel, and jurisdiction are explicit.
- [ ] Excluded clinical and D4 authority is enforced in product, prompts, tools, tests, and training.
- [ ] Category ownership does not absorb document extraction, trials, claims, finance, or clinical decision support.
- [ ] Named humans accept clinical, safety, privacy, identity, accessibility, integration, and operations accountability.

### Data and authority

- [ ] Patient binding is namespace-qualified, versioned, and invalidatable.
- [ ] Actor, patient, representative, workforce, and service identities are distinct.
- [ ] Consent/authority checks bind purpose, action, data, destination, time, and policy.
- [ ] Clinical evidence is minimum necessary, source-versioned, and contradiction-aware.
- [ ] Special data and confidential-channel policies are versioned.

### Runtime and workflows

- [ ] Rules handle known paths; the model receives only bounded ambiguity.
- [ ] Case, task, plan, handoff, safety, effect, and reconciliation states are durable.
- [ ] Closed-loop success uses authoritative receipts/postconditions.
- [ ] Scheduling, referral, authorization, care-task, outreach, and handoff boundaries are explicit.
- [ ] Safety is an interrupt with independent staffed routes.

### Reliability and operations

- [ ] Tool contracts declare authority, preconditions, timeouts, retries, idempotency, and postconditions.
- [ ] Unknown outcomes, late events, duplicates, cancellations, and corrections are tested.
- [ ] Telemetry is PHI-minimized; audit/effect evidence is complete and restricted.
- [ ] SLOs, admission control, capacity, degradation, DR, and cost include humans and reconciliation.
- [ ] Kill switches, manual operations, rollback, incident runbooks, and patient/care-team follow-up are practiced.

### Change governance

- [ ] Behavior manifests pin every behavior-affecting artifact.
- [ ] Evaluation covers hard gates, repeated trials, slices, hazards, failures, load, and recovery.
- [ ] Releases progress through approved shadow/pilot/canary stages.
- [ ] Incidents and near misses update controlled tests and the safety case.
- [ ] Source refresh and deprecation owners are assigned.

## Common false completions

| Claim | Why it is false |
|---|---|
| “We use FHIR, so interoperability is solved.” | Release, profiles, extensions, operations, terminology, and vendor behavior still differ |
| “The user logged in, so identity is solved.” | Actor identity is not patient binding or proxy authority |
| “Consent is in the EHR.” | Representation is not point-of-use enforcement |
| “A human reviews it.” | Approval may not bind exact patient/action/data/version |
| “The tool timed out, so it failed.” | It may have committed and now requires reconciliation |
| “The referral was sent.” | Acceptance, scheduling, follow-up, and ownership may remain open |
| “The model remembered the case.” | Hidden/session memory is not durable, governed truth |
| “Accuracy is 98%.” | Aggregate accuracy can conceal a single hard safety/privacy failure |
| “It is only administrative.” | Actual intended use and behavior can cross clinical/regulatory boundaries |
| “Stage 6 means autonomous.” | It means governed change; clinical and authority ceilings remain |

## Definition of a production-grade Stage 6 system

A mature system:

- uses a model only where bounded administrative ambiguity justifies it;
- preserves patient, actor, purpose, proxy, and care-team authority;
- treats clinical information as versioned evidence;
- makes no autonomous clinical decision;
- maintains durable cases, tasks, effects, handoffs, safety holds, and audit;
- reconciles every consequential uncertainty;
- supports accessible human alternatives;
- withstands dependency, process, region, and queue failures;
- reports quality, harm controls, latency, capacity, human effort, and cost honestly;
- changes only through versioned evidence and accountable approval.

That is the ceiling of this blueprint. More autonomy is not a maturity stage.

## Related guides

- Previous: [Deployment, Scale, Capacity, Cost, and Behavior Evolution](09-deployment-scale-capacity-cost-and-behavior-evolution.md)
- Home: [Healthcare Care Coordination Agent](README.md)
- Evidence: [Healthcare Care Coordination Agent Blueprint](../../research/packets/healthcare-care-coordination-agent-blueprint.md)
- Canonical: [Deployment, Release, and Incident Response](../../operations/deployment-release-and-incident-response.md)

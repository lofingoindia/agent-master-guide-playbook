# Mission, Boundaries, and Workload Fit

The first design decision is whether an agent belongs in the process at all. Manufacturing work combines noisy evidence, regulated records, irreversible product consequences, and physical hazards. Use a model where interpretation or evidence synthesis is the bottleneck; keep deterministic logic where rules, calculations, sequencing, or control already solve the problem.

## Choose the least autonomous adequate system

| Need | Preferred implementation | Why |
|---|---|---|
| Fixed threshold, interlock, permissive, shutdown, or timing sequence | PLC/SIS/SCADA or certified deterministic control | Timing and safety behavior must be predictable and validated. |
| Known maintenance interval or meter trigger | CMMS/EAM preventive-maintenance rule | The rule is explicit, auditable, cheap, and reliable. |
| SPC limit, sampling-plan calculation, tolerance check, or unit conversion | Validated deterministic service | A language model adds error without adding judgment. |
| Dashboard, KPI, or known correlation | Historian/MES/QMS query and BI | Direct data products are easier to verify. |
| Known workflow with fixed branching | BPM/workflow engine | Explicit transitions and permissions outperform open-ended planning. |
| Ambiguous symptoms across manuals, history, and free text | Read-only retrieval and ranked evidence assistant | Interpretation helps; human remains decision maker. |
| Cross-system case coordination with exceptions | Bounded coordinator plus typed tools and durable workflow | Planning helps when actions remain constrained and observable. |
| Direct autonomous physical control | Do not deploy an agent | Consequences, timing, validation, and accountability are unacceptable. |

An “AI feature” can be a classifier, extraction model, ranking service, or generated draft. Do not expand it into an agent unless the use case truly needs iterative tool selection under changing evidence.

Reject an agent entirely when the work has any of these properties:

- a deterministic alarm, interlock, limit, permissive, calculation, sampling rule, state machine, or CMMS/QMS workflow already expresses the requirement;
- the decision must be repeatable from the same inputs and a validated calculation can produce it;
- the required evidence is not reliably identifiable, versioned, timely, or legally available;
- the only proposed benefit is reducing qualified operator, technician, engineer, or quality review rather than reducing evidence-search burden;
- a wrong recommendation could create an unsafe physical action before an accountable person can detect and stop it;
- the team cannot fund adapter qualification, durable state, reconciliation, evaluation, incident response, and ongoing plant-specific ownership;
- the process has too little volume or interpretive ambiguity to outperform improved search, forms, dashboards, training, or staffing.

The no-agent comparison must include a qualified engineer/operator service, not just software. Better alarm rationalization, an updated job plan, a reliability route, a quality engineer on call, a shift-handoff checklist, or removal of a broken approval step may solve the root problem more safely and cheaply.

| Candidate bottleneck | First alternative to test | Evidence required before an agent is justified |
|---|---|---|
| Repeated threshold response | deterministic alarm/rule plus alarm-management review | residual cases genuinely require cross-source interpretation |
| Preventive work generation | CMMS meter/calendar trigger and controlled job plan | exceptions remain costly after master-data cleanup |
| Inspection valuation | validated sampling/tolerance/SPC service | model is limited to explanation or evidence assembly |
| Fixed approvals and routing | QMS/BPM workflow with role and deadline rules | exception coordination cannot be expressed safely as fixed branches |
| Hard-to-find procedures/history | approved search, indexed documents, and training | iterative evidence selection adds measured value over retrieval alone |
| Difficult diagnosis | qualified technician/reliability/quality review with better evidence views | shadow trials show complementary value without degrading challenge behavior |

## Define the operational mission

Write one sentence with all five elements:

> For **named users** at **named sites**, use **enumerated evidence** to produce **enumerated artifacts/actions** within **an explicit authority ceiling**, so that **a measured operational outcome** improves.

Example:

> For reliability engineers at Plant A, correlate historian condition evidence, calibrated inspection results, asset history, and approved manuals to draft maintenance cases and, after exact approval, create non-released CMMS work orders, so median triage time falls without increasing duplicate work or missed safety boundaries.

Reject missions such as “optimize the factory” or “autonomously improve quality.” They have no bounded target, evidence contract, or accountability.

## Keep neighboring domains separate

The agent may coordinate with adjacent systems but must not silently absorb their authority.

| Domain boundary | Manufacturing maintenance and quality owns | Neighbor retains authority |
|---|---|---|
| Supply Chain | part requirement, approved substitute request, work-order demand, lot/serial evidence | stock allocation, reservation policy, goods movement, purchasing, transport, network planning |
| Energy | equipment demand context and maintenance constraints | utility dispatch, energy trading, load shedding, electrical protection, plant energy control |
| IT/SRE | application health requirements and incident evidence | corporate infrastructure changes, identity lifecycle, enterprise incident command, cloud/network remediation |
| Back office | plant case context and controlled record requests | generic HR, finance, legal, procurement, payroll, or contract decisions |
| EHS/safety | hazard references and required handoff points | risk acceptance, permits, LOTO, machine safety, incident classification, return-to-service safety sign-off |
| Quality unit | evidence assembly, draft nonconformance/CAPA, decision options | disposition, concession/deviation approval, final usage decision, lot/batch release, regulatory judgment |

When one workflow crosses a boundary, exchange a typed request and status. Never share a vague “agent task” whose owner and authority can drift.

## Classify authority before tools

| Tier | Capability | Default control |
|---:|---|---|
| M0 | Search, read, explain, compare, and cite evidence | Allowed within site and role scope; log retrieval lineage. |
| M1 | Draft a case, recommendation, inspection note, or work-order plan | Human reviews before it becomes a controlled record. |
| M2 | Create or update a non-release business record | Exact approval or narrowly pre-authorized rule; verify by read-back. |
| M3 | Execute a reversible business effect that can change schedule, reservation request, or workflow state | Named approver, immutable digest, fresh preconditions, compensation plan, and monitored canary. |
| M4 | Safety/control action, LOTO/permit attestation, final product disposition or release, regulatory decision, or unsafe physical effect | Prohibited from agent execution. Accountable human and validated deterministic systems only. |

Authority is the minimum of user role, site policy, workflow state, tool capability, current evidence freshness, approval scope, and behavior-release policy. The model cannot raise it.

The operating envelope is a controlled engineering artifact, not a model-created constraint. Operators and engineers own operating modes, alarm response, safe limits, temporary deviations, and return-to-service criteria. The agent may cite the effective envelope and stop when observations fall outside it; it must not extend, reinterpret, waive, or optimize around it.

## Start with the smallest bounded loop

The recommended first loop is read-only triage plus a human-reviewed draft:

1. Receive one case for one site and resolved asset or quality object.
2. Retrieve a fixed set of authoritative evidence through typed reads.
3. Normalize units and mark missing, stale, disputed, or poor-quality inputs.
4. Produce a structured hypothesis set with evidence for and against each option.
5. Draft a maintenance request or nonconformance record without submitting it.
6. Stop and hand the bundle to the accountable role.

Exclude scheduling, part reservation, ERP posting, quality release, control writes, and cross-site actions. This loop tests the hardest foundations—identity, evidence, uncertainty, and usefulness—without creating external effects.

## Make stop rules executable

The coordinator must transition to `NEEDS_HUMAN` or `BLOCKED`, not improvise, when any of these conditions holds:

- the site, asset, component, lot, serial, characteristic, or work object is ambiguous;
- effective-dated aliases conflict or a replacement/split/merge is unresolved;
- an observation is stale for the decision window, has bad/uncertain status, lacks a unit, or depends on expired calibration;
- source and ingest timestamps, sequence continuity, or lineage are missing where material;
- historian, MES, EAM, QMS, or edge sources disagree on a safety- or release-relevant fact;
- a hold, permit, LOTO, safety event, inhibited alarm, bypass, or active hazardous condition is present;
- the approved procedure, specification, sampling plan, model card, policy bundle, or adapter contract is unavailable or outside its effective period;
- a requested action exceeds the authority tier or its approval digest/expiry/preconditions;
- an earlier effect has outcome `UNKNOWN`, reconciliation is incomplete, or a conflicting workflow owns the same resource;
- tool behavior, schema, permissions, unit mapping, endpoint capability, or vendor version differs from qualification;
- the site is isolated and the action depends on central authority or evidence that cannot be freshly validated;
- the queue item, approval, case, recommendation, or evidence window has expired;
- postconditions fail, a safety-boundary probe fires, or policy enforcement is unavailable.

Stop rules belong in policy code, tool schemas, and network/credential boundaries. Repeating them only in the prompt is insufficient.

## Select workloads with an evidence scorecard

Score each candidate 0–2. A first production candidate should have high interpretation value and low effect risk.

| Dimension | 0 | 1 | 2 |
|---|---|---|---|
| Evidence authority | mostly free text or disputed | mixed sources | authoritative, typed, versioned sources |
| Identity quality | frequent ambiguous joins | resolvable with review | stable effective-dated identifiers |
| Interpretive benefit | deterministic rule suffices | some synthesis | substantial cross-source ambiguity |
| Effect reversibility | physical/regulatory/irreversible | compensable business effect | read-only or draft |
| Failure detectability | latent or unsafe | delayed | immediate independent verification |
| Existing process baseline | unknown | partially measured | measured and owned |
| Human accountability | unclear | shared | named role and escalation |
| Evaluation oracle | subjective only | sampled review | deterministic facts plus expert rubric |

Reject a candidate if any hard boundary is crossed, even if its total score looks attractive.

## Establish Stage 0–6 as authority gates

| Stage | Capability | Exit evidence |
|---:|---|---|
| 0 | Manual baseline and risk map | Process owner, hazards, baseline, source inventory, and no-agent comparison documented. |
| 1 | Deterministic identity and evidence foundation | Replays show complete lineage, unit safety, freshness, and collision handling. |
| 2 | Read-only shadow triage and drafts | Representative evals meet usefulness and safety targets with zero effects. |
| 3 | Durable human-in-the-loop workflow | Resume, approval, cancellation, unknown-outcome, and audit drills pass. |
| 4 | Bounded M2/M3 business effects | Canary proves no duplicates, correct read-back, and reliable rollback. |
| 5 | Multi-site resilience and controlled offline mode | Isolation, queue expiry, backpressure, failover, restore, and recovery-load tests pass. |
| 6 | Governed evolution | Drift gates, behavior releases, refresh ownership, incident feedback, and retirement criteria operate. |

Stages are not model-capability levels. A more capable model does not permit skipping identity, policy, effects, or operational proof.

## Measure value and harm together

Use paired metrics:

- median and tail triage time **and** false association rate;
- planning time **and** schedule-constraint violations;
- work-order completeness **and** duplicate or mis-targeted orders;
- time to containment **and** unapproved disposition/release attempts;
- repeat-failure rate **and** unnecessary-maintenance rate;
- investigation effort **and** missing/late/altered evidence;
- operator adoption **and** automation-bias indicators such as unedited acceptance or weak challenge rates.

Do not use model answer scores, token volume, or number of “autonomous tasks” as operational outcomes.

## Workload acceptance record

Before implementation, record:

```yaml
mission_id: plant-a-pump-triage-v1
owner_role: reliability-engineering-lead
sites: [plant-a]
objects: [rotating-equipment-maintenance-case]
authority_ceiling: M1
allowed_outputs: [evidence_bundle, hypothesis_set, work_order_draft]
forbidden_outputs: [control_write, loto_attestation, return_to_service, part_issue]
authoritative_sources: [asset_registry, historian, calibration_system, eam, approved_manuals]
freshness_policy: rotating_equipment_triage_v3
human_handoff: area_reliability_engineer
baseline_window: 90d
success_metrics: [triage_p50, evidence_completeness, false_asset_join_rate]
kill_conditions: [identity_collision, unsafe_boundary_attempt, unexplained_duplicate_effect]
```

Version this record with the deployed behavior and reapprove it when sites, objects, outputs, or authority change.

## Read next

Use [Reference architecture and OT safety boundaries](02-reference-architecture-and-ot-safety-boundaries.md) to turn the authority contract into physical separation, and [Zero-to-production roadmap, runbooks, and exercises](12-zero-to-production-roadmap-runbooks-and-exercises.md) for the evidence required at each stage.

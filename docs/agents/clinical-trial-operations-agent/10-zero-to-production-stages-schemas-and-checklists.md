# Zero-to-Production Stages, Schemas, and Checklists

## Delivery principle

Advance one evidence-backed stage at a time. A higher stage does not compensate for a lower-stage failure: autoscaling cannot fix unsafe authority, more retrieval cannot fix ambiguous identity, and an accurate model cannot repair an unvalidated clock or missing audit trail.

## Stage 0 — Qualify the workload

### Stage 0 deliverables

- Map the current human and system workflow, authoritative records, roles, clocks, failure history, and manual fallback.
- Implement the deterministic baseline first: identity resolution, protocol release lookup, rule calculations, forms, templates, and reports.
- Define the narrow intended use, prohibited decisions, adjacent-category handoffs, and measurable value.

### Stage 0 exit gate

- [ ] The model is unnecessary for clocks, identity, authorization, effectivity, or exact calculations.
- [ ] A real language/evidence-organization gap remains and is worth its validation/privacy cost.
- [ ] Worst-case model outputs can be bounded before effects.
- [ ] Qualified owners accept retained authority and the manual fallback.

## Stage 1 — Build a bounded agent

### Stage 1 deliverables

- One typed task and one output schema.
- Read-only tools plus at most one genuinely staged D2 write.
- Hard step/tool/time/write/cost budgets and stop conditions.
- Least-privilege context, prompt-injection defense, source citations, and structured uncertainty.

### Stage 1 exit gate

- [ ] No free-form endpoint, SQL, credential, identifier, or authority input reaches a tool.
- [ ] Model output cannot enroll, randomize, dose, unblind, classify final safety, sign, lock, or file.
- [ ] Schema and invariant violations fail closed into human review.
- [ ] The deterministic alternative remains available.

## Stage 2 — Prove a useful MVP

### Stage 2 deliverables

- Use a representative configured study sandbox and qualified interface test environment.
- Persist task state and evidence references; expose draft/review/approval states clearly.
- Test protocol/site/participant identity, source provenance, human usability, and outcome value.
- Start with synthetic/deidentified cases and a small qualified reviewer group.

### Stage 2 exit gate

- [ ] The MVP materially improves accepted artifact quality, review time, or exception detection.
- [ ] Qualified users reliably understand draft status, evidence, uncertainty, and authority boundaries.
- [ ] Critical evidence fidelity and abstention thresholds pass.
- [ ] No severe privacy, blinding, safety, identity, or authority failure remains open.

## Stage 3 — Reliable v1

### Stage 3 deliverables

- Durable state/events/effects, checkpoints, leases, concurrency control, cancellation, and recovery.
- Explicit context/compaction contract and the seven tested memory lifetimes.
- Operation-qualified adapters, idempotency, `UNKNOWN` effects, postconditions, and population reconciliation.
- Protocol/amendment release manifests, deterministic jurisdiction clocks, and correction invalidation.
- Contract, integration, failure-injection, backup/restore, and audit-export tests.

### Stage 3 exit gate

- [ ] Crash before/during/after an effect recovers without duplicate consequential action.
- [ ] Protocol/policy/role changes mid-run are revalidated at commit.
- [ ] Source corrections invalidate affected derivatives and decisions visibly.
- [ ] Safety, audit, and manual workflows survive model/provider outage.
- [ ] Interfaces and archives preserve counts, metadata, audit trails, and traceability.

## Stage 4 — Production

### Stage 4 deliverables

- Strong human/workload identity, delegation, tenant/study/site/purpose/blinding authorization, and secret lifecycle.
- Validated intended-use dossier, risk assessment, release approval, training, periodic review, and decommissioning plan.
- Two evidence planes, production SLOs, alert/runbook ownership, independent stop controls, and incident/CAPA workflow.
- Signed behavior manifest and risk-based shadow/canary/rollback process.

### Stage 4 exit gate

- [ ] Security, privacy, data integrity, validation, and supplier reviews are approved.
- [ ] Severe failure classes are noncompensating release gates.
- [ ] Operators have tested disable, reconcile, evidence-preserve, manual-fallback, and recovery controls.
- [ ] Audit/inspection evidence is complete without model availability.
- [ ] Initial studies/sites and intended users are explicitly bounded.

## Stage 5 — Scale

### Stage 5 deliverables

- Priority/fair queues, admission control, backpressure, per-study/site quotas, and connector rate-limit management.
- Cell/region isolation, encrypted data residency, disaster recovery, bulkhead separation, and capacity models.
- Cost-per-accepted-task and human-review-capacity planning.
- Multi-site amendment waves, configuration drift detection, and bounded reconciliation populations.

### Stage 5 exit gate

- [ ] Safety and imminent participant-protection work retain capacity under peak load.
- [ ] One tenant, study, site, connector, or bulk job cannot cause cross-scope leakage or starvation.
- [ ] Load/soak/failover/backlog-drain tests meet SLOs with measured headroom.
- [ ] Regional failover preserves legal, protocol, privacy, blinding, and record-location constraints.
- [ ] Unit economics include human review, validation, incidents, storage, and integrations.

## Stage 6 — Governed evolution

### Stage 6 deliverables

- Offline/online evaluation, incident and near-miss mining, drift monitors, periodic access/system review, and source-refresh process.
- Change classification for model, prompt, retrieval, tool, adapter, schema, protocol compiler, policy, terminology, and infrastructure changes.
- Revalidation and release gates proportional to context-of-use risk.
- Deprecation, migration, study closeout, archive, vendor exit, and end-of-support procedures.

### Stage 6 exit gate

- [ ] Every behavior change maps to risks, tests, approvals, rollout, rollback, and evidence.
- [ ] Critical slice and human-factor performance remain stable after release.
- [ ] New regulations, guidance, standards, vendor versions, and protocol patterns have named refresh owners.
- [ ] Incident fixes become durable controls and held-out regression cases.
- [ ] Decommissioning proves complete, readable, metadata-preserving export and retrievability.

## Stage exercises and required exit evidence

Passing a checklist requires reproducible evidence, not a meeting assertion. Re-run the exercise when its protocol,
policy, provider, adapter, model, or behavior assumptions change.

| Stage | Required exercise | Measurable gate | Exit evidence to retain |
|---|---|---|---|
| 0 — Qualify | Run representative work through the current deterministic/manual baseline and a bounded model candidate | Model path improves a named outcome while every exact rule, identity, clock, effectivity and authority decision remains deterministic/human-owned | Current-state workflow map, baseline measurements, intended-use/prohibited-use statement, risk classification and manual fallback owner |
| 1 — Bound | Attack one read-only/draft task with prompt injection, cross-site IDs, stale releases, missing evidence, budget exhaustion and invalid schemas | Zero unauthorized effects or cross-partition disclosures; every invalid/ambiguous case stops in its expected typed state | Signed task/tool/schema contracts, traces linked to control records, invariant report and rejected-input corpus |
| 2 — Useful MVP | In a configured sandbox, complete consent/re-consent, eligibility-evidence, and monitoring/query worked cases with qualified reviewers | Predefined severe-error threshold is zero; evidence fidelity, abstention, reviewer time/quality and inter-reviewer calibration meet targets | Versioned scenario pack, source/expected-outcome lineage, reviewer rubric/calibration report, utility comparison and open-risk disposition |
| 3 — Reliable v1 | Lose responses before/after writes, crash and resume after compaction, reorder/correct events, revoke roles and reconcile populations | No duplicate consequential effect; every `UNKNOWN` is reconciled/owned; reconstruction preserves clocks, approvals, watermarks, blinding and next safe action | State/event/effect export, compaction receipts, operation-level adapter dossiers, recovery/reconciliation report and audit-export verification |
| 4 — Production | Rehearse shadow, constrained canary, kill switch, participant/safety manual path, incident containment and inspection reconstruction | No shadow mutation; independent stops meet objective; regulated/control evidence remains available without model service | Validation dossier, approved whole-behavior manifest/diff, release and training approvals, runbooks, incident exercise and inspection package |
| 5 — Scale | Load each priority tier and provider limit; fail a tenant cell/region; restore with a backlog and reviewer/provider constraints | Critical capacity and fairness hold; no scope leakage; RPO/RTO, reconciliation completion and recovery-drain objectives pass with headroom | Capacity model, fairness/starvation results, cell/region boundary evidence, DR/recovery-load report and updated operating limits |
| 6 — Evolve | Change a model, provider API/configuration, terminology/policy rule and protocol compiler; detect drift and roll each back | Semantic diff finds affected tasks/populations; gates catch seeded regressions; rollback does not erase records or replay unknown effects | Change record, refresh evidence, controlled failure fixtures, reviewer/drift report, rollout/rollback results and deprecation/migration plan |

## Reference contracts

### Task admission

```yaml
task_admission:
  task_id: task_01J...
  task_type: eligibility_evidence_mapping
  requested_by: user_882
  sponsor_id: sponsor_18
  study_id: study_0042
  site_id: site_101
  participant_study_id: pt_2041
  protocol_release_id: pr_2026_0042_v3_eu_wave1
  jurisdiction_policy_release: eu_ct_2026_07
  delegated_role_ref: role_assignment_71
  purpose: screening_support
  blinding_partition: BLINDED_SITE
  source_watermarks: {}
  behavior_release_id: behavior_2026_08_31_3
  admitted_policy_decision_id: pdp_01J...
```

### Human decision

```yaml
human_decision:
  decision_id: dec_01J...
  decision_type: eligibility_confirmation
  subject_ref: participant_study_id
  protocol_release_id: pr_2026_0042_v3_eu_wave1
  candidate_artifact_ref: eligibility_map_44
  evidence_refs: [fact_1, fact_2]
  decision: ELIGIBLE
  rationale: qualified_record_ref
  decided_by: investigator_22
  role_assignment_ref: role_assignment_71
  decided_at: 2026-08-31T11:00:00Z
  signature_or_approval_ref: validated_system_ref
```

### Semantic effect

```yaml
effect:
  semantic_effect_id: study_0042/site_101/edc_query/form1_item3/sourcev7
  effect_type: edc.create_query_draft
  tier: D2_STAGED
  task_id: task_01J...
  authorization_decision_id: pdp_01J...
  approval_refs: []
  destination_contract_version: edc_adapter_v3
  request_hash: "..."
  state: UNKNOWN
  attempt_count: 1
  destination_record_id: null
  postcondition: draft_with_effect_id_exists
  next_reconciliation_at: 2026-08-31T11:02:00Z
```

## End-to-end control matrix

| Control objective | Prevent | Detect | Recover |
|---|---|---|---|
| Correct identity/version | Typed IDs, release manifest, unique mappings | Scope/version assertions, reconciliation | Quarantine, correct mapping, impact review |
| Human retained authority | No decision/commit tool, approval gate | Audit decision actor and capability | Stop effect, preserve evidence, qualified reassessment |
| Consent/eligibility integrity | Current form/ruleset and missing-state semantics | Chronology/evidence mismatch checks | Re-consent/review workflow; no record rewrite |
| Safety timeliness | Independent intake/clock/escalation | Clock watchdog and SLO burn | Manual submission path and follow-up reconciliation |
| Blinding/privacy | Zone separation and minimum projections | Canary/DLP/access anomaly | Disable/revoke/quarantine and impact assessment |
| Data/record integrity | Source refs, audit trail, validated transfer | Control totals, hashes, population reconciliation | Bounded redrive/correction with preserved history |
| External effect safety | Semantic ID and commit policy | Postcondition and acknowledgement | Explicit `UNKNOWN`, reconcile, authorized retry |
| Behavior integrity | Signed pinned release | Runtime attestation/config drift | Refuse admission, rollback approved manifest |

## Pre-production go/no-go checklist

### Authority and participant protection

- [ ] Clinical, eligibility, dosing, safety, protocol, filing, and participant-protection decisions have named qualified owners.
- [ ] No tool path bypasses their validated workflow.
- [ ] Emergency contact, urgent safety, and unblinding paths are independent and tested.

### Study and data integrity

- [ ] Study/site/participant/protocol identity and external mappings are verified.
- [ ] Amendment effectivity and participant transition are governed release objects.
- [ ] Source, derived, audit, and essential records are distinguishable and reconstructable.

### Runtime and integrations

- [ ] State, events, effects, clocks, checkpoints, cancellation, and recovery are durable.
- [ ] Every adapter has intended use, qualification, version, idempotency, reconciliation, and exit evidence.
- [ ] Capability is approved per operation, tenant/configuration, role, region and exact API/schema—not inferred from a vendor name.
- [ ] Unknown outcomes never trigger blind retries.

### Security and privacy

- [ ] Purpose, site/study scope, delegation, and blinding are evaluated at commit.
- [ ] Direct identity is separated and model data is minimized.
- [ ] Prompt injection, cross-scope access, exfiltration, and unblinding tests pass.

### Evaluation and operations

- [ ] Critical slice, human factors, failure injection, load, restore, and inspection-export tests pass.
- [ ] Behavior release, SLOs, alert owners, runbooks, incident stops, and rollback are approved.
- [ ] Cost and human review capacity remain viable under projected load.

## Refresh triggers

Reassess the architecture and validation when any of the following changes:

- ICH GCP, safety, protocol, data-standard, or AI guidance;
- jurisdiction legislation, guidance, portal, reporting clock, privacy interpretation, or ethics process;
- protocol design, amendment handling, product risk, trial phase, decentralization, DHT, or use of real-world data;
- EDC/eTMF/CTMS/IRT/lab/safety/registry interface, schema, audit behavior, or vendor;
- MedDRA, CDISC controlled terminology, ODM, USDM, SDTM, FHIR profile, or submission standard;
- model, prompt, context projection, retrieval corpus, tool, policy, adapter, or hosting region;
- a safety, privacy, integrity, authority, blinding, inspection, reliability, or human-factors incident; or
- drift in critical evaluation slices, user behavior, source data, latency, capacity, or cost.

## Related guides

- [Clinical-Trial Operations Agent](README.md)
- [Mission, boundaries, authority, and workload fit](01-mission-boundaries-authority-and-workload-fit.md)
- [Qualified adapters and worked clinical-operations flows](11-qualified-adapters-and-worked-clinical-operations-flows.md)
- [Evaluation, observability, deployment, scale, and incidents](09-evaluation-observability-deployment-scale-and-incidents.md)
- [Run controls](../../runtime/run-controls.md)
- [Trajectory and reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md)
- [Queues, scheduling, and backpressure](../../operations/queues-scheduling-and-backpressure.md)

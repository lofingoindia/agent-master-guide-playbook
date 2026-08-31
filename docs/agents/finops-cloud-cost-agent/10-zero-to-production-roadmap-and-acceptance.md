# Zero-to-Production Roadmap and Acceptance

This roadmap is evidence-gated, not calendar-gated. Each stage must leave a simpler fallback and measurable baseline. A later stage cannot weaken an earlier authority, monetary-integrity, tenant, evidence, or side-effect invariant.

## Stage map

| Stage | Outcome | Maximum default authority |
|---|---|---|
| 0 — Qualify | Prove the workload and deterministic baseline | Offline compute only |
| 1 — Bounded loop | Demonstrate typed reasoning with no external authority | F0 |
| 2 — MVP | Operate one real, isolated workflow with durable evidence and human review | F1; sandboxed F2 |
| 3 — Reliable v1 | Survive waits, duplicates, corrections, restarts, and ambiguous effects | F2; optional sandbox F3 |
| 4 — Production | Meet identity, tenancy, release, SLO, audit, and incident requirements | Approved F2; optional approved F3 |
| 5 — Scale | Preserve safety and fairness under volume, burst, and recovery load | No increase |
| 6 — Continuous evolution | Change models, data, policy, and analytics through evaluation gates | No increase without separate governance |

An organization can stop at any stage. Many deployments should remain read-only or F2.

## Practical exercises and exit evidence

Run these exercises in addition to the detailed gates below. Store the named proof, not a slide stating that testing occurred.

| Stage | Hands-on exercise | Minimum exit evidence |
|---|---|---|
| 0 | Reconcile one closed and one open billing period across the chosen source and provider invoice/console view; reproduce allocation, anomaly, forecast, rightsizing, and commitment baselines without a model | Source manifests/digests, schema and cost-basis mapping, reconciliation worksheet, baseline queries/releases, labeled fixtures, human-time/cost measures, and a written stop/no-agent decision |
| 1 | Give the bounded model contradictory evidence, a malicious tag, mixed currencies, a stale recommendation, and missing SLO data; force every tool/time/token/no-progress limit | Frozen context packets, typed plans/tool results, outputs and validation failures, baseline comparison scorecard, authority inventory proving no writer exists, and stop-reason coverage |
| 2 | Execute the anomaly walkthrough from real export to authenticated disposition and a sandbox ticket; crash after intent persistence and after external acceptance but before receipt | Raw-to-outcome replay bundle, case/events/approval/effect records, unknown-outcome reconciliation proof, tenant tests, sandbox object, operator notes, and cost/outcome report |
| 3 | Wait across restart, corrupt a continuity receipt, reorder/duplicate events, rotate a secret mid-page, change target after approval, and restore a backup with one ambiguous effect | Per-source/event watermark receipt, invariant-hash failure, fencing logs, restored effect ledger, no-duplicate proof, correction supersession, memory poisoning/deletion results, and runbook completion time |
| 4 | Canary one production workflow, then run game days for missing export, cross-tenant query, model outage, wrong allocation release, effect timeout, key compromise, and harmful recommendation | Signed launch/gate record, effective IAM evidence, dashboards/SLOs, complete audit export, incident records, recovery/rollback proof, privacy/control acceptance, and named residual risks |
| 5 | Apply 10×/100× signal burst, month-end correction, large backfill, cell/region loss, and replay concurrent with live traffic while a reconciliation deadline is pending | Capacity model versus observed results, fairness/queue-age plots, provider/quota compliance, recovery-load cost, effect deadline proof, RPO/RTO result, and degradation decisions |
| 6 | Change one provider schema, allocation release, model/context release, and commitment product assumption independently; shadow/canary each and roll one back | Compatibility fixtures, release manifests and scorecards, changed-decision explanations, rollback evidence, refreshed source access/status log, deprecation record, and revalidated deterministic-baseline advantage |

An artifact counts only if it identifies workflow, tenant/scope, source and behavior releases, authority ceiling, execution time, owner, result, and unresolved risk. Screenshots without machine-readable identity are supporting context, not exit evidence.

## Stage 0 — Qualify the deterministic baseline

### Stage 0 architecture

Use exported snapshots, reviewed SQL/notebooks or existing BI tools, and a human-run workflow. No agent loop, production connector identity, memory, or effect adapter is required.

### Stage 0 authority

Offline calculation and draft analysis only. No production writes or automated communication.

### Stage 0 inputs and outputs

- Inputs: de-identified or access-controlled historical cost exports, allocation rules, ownership, telemetry, change events, and reviewed labels.
- Outputs: repeatable allocation report, anomaly baseline, naïve forecast, and one optimization evidence pack.

### Stage 0 state and event contract

Version the source artifact, query/calculation release, policy/configuration, label set, and output digest. A spreadsheet without source identity and formula/version control does not satisfy the evidence requirement.

### Stage 0 failure work

Exercise missing days, duplicate rows, late corrections, mixed currencies, stale ownership, a planned change, low utilization with missing SLO evidence, and provider/list-price mismatch.

### Stage 0 evaluation

Measure human effort, data freshness/completeness, allocation coverage and material residual, anomaly precision/label coverage and alert volume, forecast error/bias versus naïve baseline, decision time, and unsafe proposal rate.

### Stage 0 exit gate

- [ ] A deterministic baseline solves all purely deterministic subproblems.
- [ ] At least one high-value ambiguity remains that model-assisted synthesis can plausibly improve.
- [ ] Category boundaries, accountable owners, currencies, source versions, and materiality are documented.
- [ ] The baseline and labeled fixtures are reproducible.
- [ ] Expected benefit exceeds the forecast implementation and operating cost.

If not, build the deterministic report/workflow and stop.

## Stage 1 — Bounded typed loop with no external authority

### Stage 1 architecture

Add a context builder, provider-neutral model adapter, output-schema validator, evidence/citation validator, and a controller with strict step/tool/token/time budgets. Tools operate on frozen fixtures or read-only preapproved queries.

### Stage 1 authority

F0 only. The model can analyze and draft. It cannot create a case, ticket, notification, budget, cloud change, purchase, or accounting entry.

### Stage 1 inputs and outputs

- Inputs: typed, bounded evidence packets with trust labels and exact amounts/units.
- Outputs: schema-valid hypothesis, explanation, missing-evidence request, forecast narrative, or optimization draft with citations.

### Stage 1 state and event contract

Persist a run record with run/attempt/step IDs, fixture/evidence snapshot, behavior release, tool results, validation errors, final output, and stop reason. State may be simple but cannot live only in transcript text.

### Stage 1 failure work

Inject invalid schemas, uncited amounts, changed currency/unit, direct and indirect prompt injection, contradictory evidence, no owner, stale provider recommendation, oversized tool result, repeated no-progress calls, and model timeout.

### Stage 1 evaluation

Compare against the stage-0 human/deterministic baseline. Score evidence fidelity, numerical fidelity, observation-versus-inference separation, appropriate abstention, authority compliance, trajectory efficiency, latency, and cost.

### Stage 1 exit gate

- [ ] Critical numerical, citation, tenant/scope, and authority invariants pass every fixture.
- [ ] Model benefit is measurable on the qualified ambiguity.
- [ ] Failure produces bounded repair, fallback, or abstention.
- [ ] The loop stops under every budget and no-progress condition.
- [ ] No external writer credential or excluded tool is present.

## Stage 2 — MVP in one real environment

### Stage 2 architecture

Connect one provider/dataset and one isolated tenant or billing scope through immutable landing, normalization, deterministic analytics, database-backed case state, queue/outbox, query broker, context builder, model adapter, policy gate, approval UI/service, and sandbox ticket/notification adapter.

Choose one workflow—normally anomaly triage or read-only optimization review. Do not launch anomaly, forecast, budget, commitments, rightsizing, and AI-cost allocation together.

### Stage 2 authority

F1 in the real isolated scope: create/update cases and drafts. F2 only against a sandbox destination after explicit test approval. F3/F4 absent.

### Stage 2 inputs and outputs

- Inputs: real provider export, source/version manifest, owner/service data, required telemetry/change evidence, approved policy.
- Outputs: durable case, cited analysis, authenticated owner disposition, and sandbox effect/receipt.

### Stage 2 state and event contract

Implement case state versioning, typed domain events, evidence snapshots, approval record, outbox, semantic operation ID, intent hash, effect receipt, and `outcome_unknown`. Correlation IDs propagate through telemetry but telemetry is not the source of business state.

### Stage 2 failure work

Test duplicate/out-of-order exports and webhooks, open-period correction, process restart, stale state version, approval drift, effect timeout after success, missing SLO/owner data, query limit, connector rate limit, and cancellation.

### Stage 2 evaluation

Run shadow comparisons with current analyst work. Sample every sandbox effect and material proposal. Measure case correctness, reviewer agreement/overrides, acknowledgement time, warning/abstention quality, duplicate prevention, reconciliation latency, cost, and operator burden.

### Stage 2 exit gate

- [ ] At least one complete real workflow is replayable from raw artifact to reviewed outcome.
- [ ] Tenant and billing-scope isolation tests pass.
- [ ] Amounts reconcile under the defined completeness and residual policy.
- [ ] Approval and sandbox effect are bound to the exact proposal and target state.
- [ ] Restarts, duplicates, corrections, and unknown outcomes are demonstrated.
- [ ] MVP outcome improvement justifies proceeding.

## Stage 3 — Reliable v1

### Stage 3 architecture

Add durable timers/waits and workflow recovery where justified, structured compaction, connector reconciliation, versioned long-term outcome/disposition stores, correction propagation, dead-letter remediation, policy/configuration management, and optional additional read-only providers.

Adopt a durable workflow engine only if the database/queue design cannot safely support the observed waits, retries, timers, and recovery load. Do not add multi-agent orchestration; typed provider adapters remain modules.

### Stage 3 authority

Production-like F2 behind approval in an isolated/preproduction environment. Optional F3 **alert-only budget configuration** may be tested if the organization has an accountable need. Infrastructure mutation, commitment purchase, permission changes, billing disablement, and ledger posting remain impossible.

### Stage 3 inputs and outputs

- Inputs: multiple source revisions, corrections, durable owner conversations/approvals, typed connector reads, and outcome evidence.
- Outputs: resumable cases, reconciled notifications/tickets, curated outcome memory, and superseding evidence when data changes.

### Stage 3 state and event contract

Enforce compare-and-swap state versions, attempt fencing, transactional outbox, schema-versioned events, compatibility rules, semantic idempotency, effect locks, and recovery/replay procedures. Compaction snapshots preserve authority, exact amounts, warnings, approvals, and pending effects.

### Stage 3 failure work

Crash at every state/effect boundary; reorder and duplicate events; rotate secrets mid-pagination; lose a receipt; recover from backup; corrupt a compaction snapshot; change target state after approval; poison feedback; and replay a correction across closed/open cases.

### Stage 3 evaluation

Add trajectory evaluation, delayed anomaly/forecast/outcome labels, reconciliation correctness, recovery time, long-case continuity, prompt-injection corpus, policy mutation tests, and release scorecards by workflow/provider/version.

### Stage 3 exit gate

- [ ] No accepted command or effect intent is lost across restart/recovery.
- [ ] Unknown effects reconcile without duplicate external state.
- [ ] Structured compaction and curated memory pass continuity/provenance tests.
- [ ] Provider corrections preserve old decision evidence and create material re-review.
- [ ] Optional F3 preview, exact approval, target precondition, and reconciliation pass.
- [ ] On-call candidates can diagnose representative failures from events and telemetry.

## Stage 4 — Production

### Stage 4 architecture

Create isolated production environments and identities, tenant-aware gateway/query/data policies, secret manager, approved model egress, release manifests, migrations/rollback, operational dashboards, SLO/error budgets, audit export, backups/recovery, and named incident runbooks.

### Stage 4 authority

Enable only the previously proven F2/F3 workflows per tenant and policy. Authority flags are separate from code/model rollout. F4 remains absent. A tenant may have a lower ceiling.

### Stage 4 inputs and outputs

- Inputs: governed production provider/export, inventory, SLO, change, contract/rate, owner, policy, and identity sources.
- Outputs: tenant-scoped evidence/cases, approved bounded effects, audit records, operational telemetry, and verified outcomes.

### Stage 4 state and event contract

Production retention, encryption, key rotation, schema migration, audit completeness, trace redaction, regional/recovery state, legal hold/deletion, and mixed-version compatibility are documented and tested.

### Stage 4 failure work

Run game days for cross-tenant access, compromised AI admin key, missing export, wrong allocation release, model/provider outage, anomaly storm, expired approval, duplicate/unknown effect, harmful recommendation, warehouse overload, and regional restore.

### Stage 4 evaluation

Require offline release gates, shadow and canary, stratified human review, online invariants, label coverage, SLOs, incident drills, security testing, cost budgets, and rollback triggers. Validate applicable financial-control evidence with responsible finance/audit stakeholders.

### Stage 4 exit gate

- [ ] Security, privacy, finance/control, service, FinOps, and operations owners accept their documented responsibilities.
- [ ] Least privilege and tenant isolation are verified from effective policy and adversarial tests.
- [ ] SLOs, alerts, dashboards, runbooks, on-call, backups, restore, rollback, and audit export work.
- [ ] Canary produces no unresolved critical invariant and meets outcome/cost targets.
- [ ] Production credentials contain no excluded authority.
- [ ] Launch scope, authority ceiling, kill switches, and communication are recorded.

## Stage 5 — Scale and resilience

### Stage 5 architecture

Partition queues and data by tenant/provider/scope/workload, add weighted fairness, per-dependency limits, reserved reconciliation capacity, admission control, cells only when justified, capacity models, regional recovery, and cost-aware materialization/caching.

### Stage 5 authority

Unchanged. Scaling capacity never scales permission. New tenants, providers, or effects pass their own compatibility and authority gates.

### Stage 5 inputs and outputs

- Inputs: higher provider/tenant volume, month-end/planning peaks, bulk corrections, replay, and more evaluation labels.
- Outputs: same contracts and quality within published degradation and SLO envelopes.

### Stage 5 state and event contract

Partition keys preserve ordering where required; leases and version fences prevent split-brain effects; retries have ownership and limits; dead-letter items retain tenant/evidence identity; recovery prevents effect replay.

### Stage 5 failure work

Test 10×/100× anomaly bursts, a large tenant's backfill, provider throttling, warehouse quota exhaustion, model saturation, hot partitions, cell loss, region failover, replay plus live traffic, and month-end corrections.

### Stage 5 evaluation

Measure queue age by class/tenant, fairness, rate-limit compliance, freshness, dropped/coalesced/deferred work, reconciliation capacity, tail latency, result correctness, cost per outcome, and recovery objectives under load.

### Stage 5 exit gate

- [ ] Capacity tests include steady, burst, correction, replay, and dependency-degraded load.
- [ ] One tenant/provider cannot starve another or the reconciliation path.
- [ ] Backpressure is explicit, bounded, observable, and contract-compatible.
- [ ] Safe degradation preserves correctness warnings and authority.
- [ ] Regional/cell restore proves state/evidence integrity and no duplicate effects.
- [ ] Unit cost remains inside an approved budget with verified outcome value.

## Stage 6 — Continuous evaluation and evolution

### Stage 6 architecture

Operate a failure-mining pipeline, versioned eval corpus, offline comparison, shadow/canary platform, data/model/policy drift monitors, provider/FOCUS/API refresh ownership, feedback governance, and deprecation workflow.

### Stage 6 authority

Unchanged by ordinary release. Any new effect type or higher authority is a separately governed project that returns to the applicable earlier stages.

### Stage 6 inputs and outputs

- Inputs: production failures, sampled reviews, owner overrides, detector/forecast residuals, corrections, incidents, provider/schema/model changes, and verified outcomes.
- Outputs: reproducible fixtures, accepted-risk records, guarded releases, refreshed source mappings, and deprecation/migration plans.

### Stage 6 state and event contract

Each behavior decision resolves to application, schema, mapping, analytical, policy, context, prompt/model, tool, and eval-corpus releases. Feedback has provenance and quarantine status. Deprecation preserves audit-readable history and reconciles all effects.

### Stage 6 failure work

Simulate silent provider schema extension, FOCUS version change, IAM role change, model alias drift/deprecation, eval contamination, feedback poisoning, detector concept drift, changed commitment products, and a rollback across mixed releases.

### Stage 6 evaluation

Use critical invariants plus workflow-specific outcome metrics, cost/latency, stratified errors, uncertainty, label coverage, shadow/canary deltas, and delayed real outcomes. Periodically revalidate that model assistance still beats the deterministic baseline.

### Stage 6 exit gate

Stage 6 has no terminal gate. Its operating acceptance is:

- [ ] Every material failure becomes a fixture, control/runbook change, or explicit accepted risk.
- [ ] Provider/schema/model/policy owners and refresh dates are current.
- [ ] Candidate releases cannot bypass critical security, money, approval, or effect invariants.
- [ ] Feedback cannot silently alter policy, memory, or the evaluation target.
- [ ] Deprecated connectors/models/workflows are migrated, reconciled, revoked, and retained/deleted according to policy.
- [ ] The agent's outcome and cost advantage over simpler controls is periodically re-proven.

## Cross-stage acceptance record

At each gate, store:

```yaml
gate_record_id: gate_01K...
stage: 3
workflow: anomaly_triage
scope: tenant_acme/aws/payer-123
authority_ceiling: F2
behavior_release: finops-agent-2026.08.4
evidence:
  baseline_report: evidence://.../baseline
  eval_scorecard: evidence://.../eval
  security_report: evidence://.../security
  recovery_drill: evidence://.../recovery
open_risks:
  - {id: risk_14, owner: finops-platform, expires: 2026-10-01}
approvals:
  - {role: service_owner, actor: user_12, decision: accept}
  - {role: security, actor: user_44, decision: accept}
decided_at: 2026-08-31T10:00:00Z
```

Passing a gate applies only to the recorded workflow, scope, releases, and authority. A new provider, tenant class, model boundary, or effect may invalidate part of the evidence.

## Final production acceptance checklist

- [ ] Category ownership and human accountability remain explicit.
- [ ] Provider/export versions, corrections, currencies, allocations, and evidence lineage are reproducible.
- [ ] The model is bounded to explanation/hypothesis/proposal and cannot become monetary truth or authority.
- [ ] Context, compaction, memory, tools, planning, and state are typed and recoverable.
- [ ] Approval binds the exact proposal; idempotency and reconciliation handle unknown outcomes.
- [ ] Tenant/security/secret/financial controls and audit evidence pass adversarial review.
- [ ] Evaluation covers deterministic components, analytics, model behavior, trajectories, effects, and delayed outcomes.
- [ ] Deployment, SLO, backpressure, incident, recovery, cost, model-change, and continuous-evolution procedures operate.
- [ ] Service SLOs and security dominate cost savings.
- [ ] Infrastructure, procurement, finance, and accounting authority remains outside the agent.

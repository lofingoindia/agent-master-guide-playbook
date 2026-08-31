# Zero-to-Production Roadmap

## Promotion rule

Stages are evidence gates, not calendar phases. Begin at Stage 0. Promote only the watches, tenants, routes, and effect types whose exit evidence passes. A system can operate Stage 4 controls while a new watch remains in Stage 1 shadow mode.

~~~mermaid
flowchart LR
    S0[0 Deterministic baseline] --> S1[1 First bounded loop]
    S1 --> S2[2 MVP]
    S2 --> S3[3 Reliable v1]
    S3 --> S4[4 Production]
    S4 --> S5[5 Scale]
    S5 --> S6[6 Continuous evolution]
    S1 --> R[Roll back capability or watch]
    S2 --> R
    S3 --> R
    S4 --> R
    S5 --> R
    S6 --> R
~~~

Authority does not grow automatically with stage. Stages 3–6 mainly strengthen reliability, operations, and learning. High-impact business action remains human-owned at every stage.

## Cross-stage deliverables

| Deliverable | First required | Evolves through |
|---|---|---|
| Metric and watch contract | Stage 0 | Versioned at every stage |
| Deterministic replay corpus | Stage 0 | Production failures and outcomes |
| Case/event/effect schema | Stage 1 | Compatibility and durable execution |
| Tool and evidence contracts | Stage 1 | Adapters, tenants, provider changes |
| Policy and decision table | Stage 1 | Formal approvals and change governance |
| Effect ledger/reconciliation | Stage 2 before any real external effect | HA, DR, scale, provider drift |
| Security/privacy threat model | Stage 1 | Tenancy, jurisdiction, adversarial corpus |
| Observability/SLOs/runbooks | Stage 2 | Production and cell-level operations |
| Behavior-bundle manifest and rollback | Stage 2 | Canary, fleet, upgrade automation |
| Outcome episode/evaluation loop | Stage 3 | Failure mining and calibration |

## Stage 0 — Deterministic baseline

### Stage 0 objective and architecture

Prove that a recurring monitored decision is well-defined and actionable without an LLM.

~~~mermaid
flowchart LR
    SC[Schedule] --> SQ[Governed semantic query]
    SQ --> G[Freshness and quality gate]
    G --> R[Fixed detector rule]
    R --> D[Dashboard/digest or test alert]
    D --> H[Human-owned runbook]
~~~

Use the existing semantic layer, scheduler/BI platform, data-quality evidence, fixed rule, and owned runbook. A spreadsheet or reviewed configuration can be sufficient for watch inventory if version history and approvals are reliable.

### Stage 0 contract

| Dimension | Contract |
|---|---|
| Authority | No model and no autonomous business action. Metric/watch owners approve semantics, threshold, route, and runbook. |
| Inputs | One governed metric; exact grain/calendar/timezone; historical observations; freshness/quality evidence; known incidents; owner-labelled actionable events. |
| Outputs | Versioned watch contract, deterministic observation history, detector replay report, owner/runbook matrix, non-agent baseline. |
| State | Watch versions, scheduled interval coverage, immutable observations, detector results, and manual disposition labels. |
| Events/effects | Evaluation events only. Notification may remain disabled or go to a test destination. No source mutation. |
| Approvals | Metric owner, data product owner, watch owner, destination owner, privacy/risk review where applicable. |
| Failure/recovery | Explicit no-data/stale/partial/error behavior; operator can suspend; missed intervals enumerated and replayed without ordinary delivery. |
| Evaluation | Historical event-level recall/precision, false alerts per watch/week, delay, materiality, owner capacity, data-gate behavior, simpler alternatives. |
| Exit gate | Owners agree the watch has stable meaning and a real response; replay produces acceptable load; data failures do not become business alerts; runbook is executable without a model. |

### Stage 0 work plan

1. Select one material recurring decision, not a broad “monitor the business” goal.
2. Document metric and data contracts plus ownership.
3. Create a watch contract with trigger, recovery, persistence, materiality, and retirement.
4. Reconstruct representative history including data failures and semantic changes.
5. Run the simplest detector and compare with current operations.
6. Dry-run the runbook with accountable operators.
7. Reject or redesign the watch if alerts do not lead to a bounded decision.

### Stage 0 stop gates

- a dashboard and weekly review are sufficient;
- owners disagree on meaning or threshold;
- no trustworthy freshness/quality evidence exists;
- historical labels are too weak to estimate load;
- the operator cannot state what acknowledgement and disposition mean;
- the desired effect is a consequential decision delegated to the model.

## Stage 1 — First bounded loop

### Stage 1 objective and architecture

Add a read-only triage worker for one validated watch and one route. It organizes evidence; the deterministic rule still opens the case.

~~~mermaid
flowchart LR
    B[Stage 0 baseline] --> C[Case controller]
    C --> EC[Context compiler]
    EC --> M[Bounded triage model]
    M --> V[Schema and evidence validator]
    V --> P[Deterministic route policy]
    P --> H[Human review or test notification]
~~~

### Stage 1 contract

| Dimension | Contract |
|---|---|
| Authority | Model can select from allowlisted read-only drill-downs and draft facts/hypotheses/route rationale. It cannot open the case, choose recipients, approve, mutate, or claim cause. |
| Inputs | Stage 0 observation/signal, case version, evidence manifest, up to three approved drill-down templates, related cases/events, authority and budgets. |
| Outputs | Typed triage packet with citations, uncertainty, missing evidence, requested reads, route proposal, and stop reason. |
| State | One durable case aggregate; working hypotheses are run-scoped; no session or personal memory. |
| Events/effects | `CaseOpened`, `TriageStarted/Completed/Rejected`, `RouteProposed`. Delivery is test-only or an explicitly approved low-risk notification. |
| Approvals | Human reviews triage packet; route/destination is pre-approved outside the model. Model/provider/data-processing review completed. |
| Failure/recovery | Invalid output is rejected; one bounded repair may run; model timeout/quota uses a deterministic factual template; state remains operable without the model. |
| Evaluation | Citation precision/coverage, unsupported claim rate, allowed-tool compliance, route agreement, turn/tool/token budget, repeated-run reliability, injection tests. |
| Exit gate | Zero hard safety violations; deterministic fallback works; triage measurably improves review time or evidence quality without increasing false confidence or alert load. |

### Stage 1 first loop

1. Controller compiles authority, task, current state, verified evidence, and tool catalog.
2. Model may request one allowed read.
3. Adapter validates rights, contract, cost, size, freshness, and provenance.
4. Controller adds the typed result and decrements budget.
5. Model emits the fixed schema or abstains.
6. Validator checks enums, citations, claims, dimensions, and route.
7. Policy determines actual route; differences are recorded for evaluation.
8. Human accepts, corrects, rejects, or requests bounded analytics.

### Stage 1 stop gates

- the model is needed to decide whether data is valid or a threshold fired;
- free-form SQL or raw rows are required;
- comments or dashboard text can select tools/recipients;
- one successful demo hides repeated-run failures;
- operators cannot challenge the explanation;
- the deterministic packet performs as well at lower risk/cost.

## Stage 2 — MVP

### Stage 2 objective and architecture

Support a small portfolio of watches with correlation, case lifecycle, acknowledgement, decision tables, and low-risk communication/work-item effects.

~~~mermaid
flowchart LR
    WR[Versioned watch registry] --> CT[Controller and relational state]
    CT --> ER[Evidence and detector runtime]
    CT --> MT[Bounded triage]
    CT --> PA[Policy and approvals]
    PA --> EL[Effect ledger]
    EL --> NT[Notification/ITSM adapters]
    NT --> RC[Reconciler]
    RC --> CT
~~~

### Stage 2 contract

| Dimension | Contract |
|---|---|
| Authority | Pre-authorized low-risk notification and case/task creation only. Human owns acknowledgement, disposition, and any operational action. |
| Inputs | Several owner-approved watch versions, semantic/data contracts, decision tables, directories, route policy, provider contracts, replay corpus. |
| Outputs | Grouped cases, cited decision packets, confirmed delivery receipts, authenticated acknowledgements, manual dispositions, basic outcome requests. |
| State | Relational watch/evaluation/observation/signal/case/approval/timer/effect records; immutable evidence artifacts; optimistic versions. |
| Events/effects | Typed domain events and an outbox. Effects have stable operation keys, payload digests, intent-before-send, and confirmed/failed/unknown states. |
| Approvals | Watch promotion and sensitive routes require owners; each non-pre-authorized effect binds exact case/payload/target/expiry. |
| Failure/recovery | Duplicate triggers/messages, provider timeout, stale approval, route failure, revision, and model outage covered. Unknown effects reconcile before retry. |
| Evaluation | Stage 1 trajectory plus workflow state, duplicate-effect, acknowledgement, revision/retraction, privacy, per-watch alert-load, and shadow tests. |
| Exit gate | No duplicate/untracked effects in fault tests; case history is explainable; acknowledged is distinct from resolved; portfolio owners accept load; rollback and pause work. |

### Stage 2 MVP scope limit

Use:

- one tenant or tightly bounded business unit;
- one semantic platform and one data-quality/lineage path;
- one notification and one ITSM adapter;
- a small set of detector types;
- a single triage model/prompt path;
- no source mutation;
- no cross-region failover;
- no online threshold learning.

### Stage 2 stop gates

- remote writes cannot be reconciled after timeout;
- the state store is not authoritative;
- a watch edit changes historical meaning;
- acknowledgement relies on chat reactions or read receipts;
- operator/channel budgets are routinely exceeded;
- watch, policy, prompt, or adapter rollback is not reproducible.

## Stage 3 — Reliable v1

### Stage 3 objective and architecture

Make the MVP survive crash/retry boundaries, multi-day waits, revisions, reconciliation, and realized-outcome workflows.

~~~mermaid
flowchart TB
    Q[At-least-once queues and timers] --> DC[Durable controller]
    DC --> ST[(Versioned state, outbox, effects)]
    DC --> CP[Case checkpoints]
    ST --> EW[Effect workers]
    EW --> RE[Reconcilers]
    DC --> OV[Outcome verifier]
    OV --> EP[Reviewed outcome episodes]
~~~

### Stage 3 contract

| Dimension | Contract |
|---|---|
| Authority | Same ceiling as Stage 2. Reliability does not justify new business action. |
| Inputs | Durable timers, crash-safe state/event/effect schemas, correction policies, reconciliation lookup, outcome definitions, reviewed memory policy. |
| Outputs | Recoverable multi-day cases, explicit revisions/retractions, reconciled effects, verified/ineffective/indeterminate outcomes, structured episodes. |
| State | Durable controller state plus event frontier/checkpoints; effect and approval state cannot be compacted away; schema versions support rollback. |
| Events/effects | At-least-once delivery tolerated. Effect state machine includes unknown/reconciling/indeterminate. Outcome checks are scheduled idempotently. |
| Approvals | Revalidated immediately before execution and invalidated by material revision, role change, expiry, policy, target, or payload change. |
| Failure/recovery | Crash at every commit/effect boundary; cancellation races; provider consistency delays; shared data incidents; backfill/revision; checkpoint reconstruction; DR restore rehearsal. |
| Evaluation | Repeated trajectory, full fault matrix, state invariant/property tests, replay under old/new schema, unknown-effect age, terminal-state coverage, outcome evidence quality. |
| Exit gate | All crash boundaries preserve state/effect invariants; no blind retry; restore reconciles before replay; outcome semantics accepted by owners; on-call can repair through typed commands. |

### Stage 3 memory and compaction gate

Enable only:

- run-scoped hypotheses;
- authoritative durable case state;
- governed domain sources by reference;
- verified structured outcome episodes.

Keep conversation and implicit user preference memory disabled. Validate that checkpointing preserves objective, authority, facts, open hypotheses, approvals, effects, timers, budgets, artifacts, failures, and lineage.

### Stage 3 stop gates

- a queue retry can repeat a remote effect;
- restore can activate two effect writers;
- outcome is inferred from ticket closure;
- historical episodes lack provenance or contain cross-purpose data;
- administrative repair requires direct database edits;
- compaction loses unknown effects or stale approvals.

## Stage 4 — Production

### Stage 4 objective and architecture

Add production identity, multi-tenant and privacy controls, SLOs, on-call, deployment gates, canaries, DR, and auditable operations.

~~~mermaid
flowchart LR
    ID[Workload identity] --> GW[Policy/tool gateway]
    GW --> PL[Production control plane]
    PL --> WP[Isolated worker pools]
    PL --> OB[Observability and audit]
    PL --> KS[Scoped kill switches]
    PL --> DR[Backups, restore, and reconciliation]
~~~

### Stage 4 contract

| Dimension | Contract |
|---|---|
| Authority | Explicit matrix by tenant, watch class, route, data class, effect type, environment, and time. Consequential decisions remain evidence-only/human. |
| Inputs | Production identity/directories, privacy and source-rights policy, behavior-bundle manifest, SLO/error budget, on-call ownership, DR plan, threat model, provider terms. |
| Outputs | Auditable production cases/effects, SLO dashboards, incident records, release/canary diffs, privacy/access/deletion evidence. |
| State | Tenant/environment structural keys, regional rules, encrypted backups, retention/deletion workflows, immutable audit, release linkage. |
| Events/effects | Scoped credentials, destination enforcement at effect time, kill switches, canary cohort, provider-specific reconciliation and late-effect handling. |
| Approvals | Security/privacy/platform/business release review; meaningful human decision interface; emergency override is typed, scoped, expiring, and audited. |
| Failure/recovery | False-alert storm, missed gap, data exposure, model regression, provider outage, state corruption, unowned route, DR failover, rollback. |
| Evaluation | Full offline/fault/adversarial gates, live shadow, approved canary, SLO/error budget, penetration and cross-tenant tests, operator game days. |
| Exit gate | On-call accepts runbooks; restore and rollback pass; privacy/security review closes; SLOs and owner capacity hold in canary; all hard gates pass. |

### Stage 4 production artifacts

- architecture and data-flow threat model;
- service and watch ownership registry;
- behavior-bundle manifest and bill of materials;
- SLOs, alert rules, dashboards, and paging policy;
- incident, pause, reconcile, re-drive, rollback, privacy, and DR runbooks;
- access, approval, retention, deletion, and legal-hold procedures;
- model/provider, semantic, detector, adapter, and policy change process;
- business-owner scorecard and watch retirement review.

### Stage 4 stop gates

- tenancy is a prompt field rather than an enforced boundary;
- production credentials can reach source mutation;
- telemetry exposes raw metric/customer/prompt data;
- kill switches and rollback do not address in-flight effects;
- owner overload makes human oversight nominal;
- a provider preview feature is treated as a production guarantee without fault tests.

## Stage 5 — Scale and resilience

### Stage 5 objective and architecture

Sustain peak watch, query, alert, and outcome volume while preserving tenant fairness, deadlines, isolation, and reconciliation.

~~~mermaid
flowchart TB
    AD[Global admission and config] --> C1[Regional/tenant cell A]
    AD --> C2[Regional/tenant cell B]
    AD --> C3[Regional/tenant cell C]
    C1 --> Q1[Class queues and reserved reconciliation]
    C2 --> Q2[Class queues and reserved reconciliation]
    C3 --> Q3[Class queues and reserved reconciliation]
    C1 --> FV[Fleet visibility without cross-tenant payloads]
    C2 --> FV
    C3 --> FV
~~~

### Stage 5 contract

| Dimension | Contract |
|---|---|
| Authority | No automatic expansion. Partitioning must preserve the Stage 4 policy matrix and human action boundary. |
| Inputs | Measured peak distributions, tenant skew, query/model/provider quotas, regional rules, watch criticality, owner/channel capacity, failure-domain analysis. |
| Outputs | Fair/deadline-aware admission, bounded degradation, cell-level SLOs, capacity forecasts, region/cell recovery evidence. |
| State | Partitioned by tenant/watch while co-locating case/timer/effect; globally unique event/effect IDs; fenced active writers; regional artifact/audit rules. |
| Events/effects | Separate workload queues; reserved control/reconcile capacity; ordered aggregate handling; provider/tenant concurrency budgets; visible deferral. |
| Approvals | Capacity/config changes reviewed; cross-region movement and tenant migration require rights and effect-fencing approval. |
| Failure/recovery | Hot partition, noisy tenant, thundering data-ready burst, regional outage, cell isolation, quota exhaustion, backlog replay, tenant migration. |
| Evaluation | Peak/soak/chaos tests with realistic skew; fairness and deadline distributions; degraded-mode safety; regional failover and duplicate-writer fencing. |
| Exit gate | Critical SLOs and hard safety invariants hold under peak and cell failure; no tenant starvation; reconciliation remains timely; recovery is bounded and rehearsed. |

### Stage 5 scale decision rules

- Optimize and reduce unnecessary watches before adding capacity.
- Separate workload classes before creating cells.
- Partition only on stable authority boundaries.
- Never let replay/backfill consume reserved due-evaluation or reconciliation capacity.
- Shed optional model triage before data gates, state transitions, or effect reconciliation.
- Make every skipped or delayed interval queryable.

### Stage 5 stop gates

- one tenant can exhaust all query/model/channel capacity;
- cases and their effects can land in conflicting partitions;
- failover creates two active effect writers;
- global fleet tooling requires unrestricted tenant data;
- degraded mode silently drops evaluations;
- load tests omit shared upstream incidents and owner capacity.

## Stage 6 — Continuous evolution

### Stage 6 objective and architecture

Mine failures and outcomes, evaluate candidates offline and in shadow, and ship reversible improvements without self-modifying production policy.

~~~mermaid
flowchart LR
    PF[Production failures and outcomes] --> FM[Reviewed failure mining]
    FM --> EC[Versioned eval corpus]
    EC --> CA[Candidate semantic/detector/model/policy]
    CA --> SH[Replay and shadow]
    SH --> AP[Owner and release approval]
    AP --> CN[Canary]
    CN --> PR[Promote or rollback]
    PR --> PF
~~~

### Stage 6 contract

| Dimension | Contract |
|---|---|
| Authority | Production data can propose evaluation examples and calibration candidates, never approve thresholds, routes, rights, tools, or actions. |
| Inputs | Reviewed incidents, validator rejects, human overrides, false/missed alerts, revisions, reconciliations, costs, SLOs, verified outcomes, provider changes. |
| Outputs | Regression fixtures, candidate baselines/detectors/prompts/policies, drift reports, watch retirement proposals, refreshed research/compatibility record. |
| State | Versioned corpus and labels, provenance-bearing episodes, candidate/current manifests, approval and rollback history, deletion/tombstone propagation. |
| Events/effects | Candidate runs are effect-disabled; shadow/canary identities prevent collision with production keys; promotion is a reviewed release command. |
| Approvals | Metric/watch owner for detector/threshold; security/privacy for rights/data changes; platform for runtime/adapter/model; business authority for decision tables. |
| Failure/recovery | Bad labels, temporal leakage, benchmark overfit, outcome confounding, provider drift, canary regression, incompatible schema, deletion propagation. |
| Evaluation | Repeated offline trajectories, historical replay, adversarial/fault regression, shadow/current diff, canary SLO and business scorecard, rollback rehearsal. |
| Exit gate | Every promoted change has local evidence, named ownership, compatible state/effects, cost envelope, current legal/provider review, and tested rollback. |

### Stage 6 continuous evaluation cadence

- **Per release:** full hard-gate corpus, compatibility, shadow/canary diff.
- **Daily/weekly:** SLO/error-budget and unknown-effect review; noisy/unowned watches.
- **Monthly:** actionability, false/missed event review, model/tool rejects, cost, outcome coverage.
- **Quarterly or risk-based:** watch ownership, semantic compatibility, threshold/runbook, privacy purpose, destination, retention, and retirement.
- **On trigger:** provider/model/API change, semantic migration, detector library upgrade, new jurisdiction, incident, or license/terms change.

### Stage 6 stop gates

- labels are derived only from whether the metric later improved;
- a public anomaly benchmark replaces local replay;
- the candidate changes prompt, model, detector, and policy simultaneously without attribution;
- open cases cannot be migrated or kept on the old version safely;
- outcome episodes slated for deletion remain in retrieval indexes;
- the proposed change increases autonomy without a separately approved authority decision.

## Promotion review template

~~~yaml
promotion:
  capability: bounded_triage
  from_stage: 2
  to_stage: 3
  scope:
    tenants: [commerce]
    watches: [revenue-drop-emea]
    effects: [itsm_create]
  manifest_ref: release://bi-monitoring/2026.08.31.2
  evidence:
    replay_report: artifact://eval/replay-229
    repeated_trajectory: artifact://eval/trajectory-89
    fault_injection: artifact://eval/faults-18
    shadow_diff: artifact://eval/shadow-73
    rollback_test: artifact://ops/rollback-31
  hard_gates:
    cross_tenant_access: 0
    source_mutations: 0
    duplicate_effects: 0
    stale_approval_effects: 0
    unsupported_verified_facts: 0
  workload_gates:
    max_false_alerts_per_week: 1
    p95_acknowledgement: PT2H
    maximum_unknown_effect_age: PT15M
  residual_risks:
    - Outcome attribution remains observational.
  approvals:
    metric_owner: approval://91
    accountable_operator: approval://92
    platform: approval://93
    security_privacy: approval://94
  rollback:
    behavior_bundle: release://bi-monitoring/2026.08.18.1
    owner: role:bi-platform-oncall
~~~

## Final production gate

- [ ] A deterministic baseline remains available and useful.
- [ ] The agent is used only where bounded semantic triage adds measured value.
- [ ] Stage evidence exists for each promoted watch, tenant, route, and effect.
- [ ] Authority never exceeds the explicit policy matrix.
- [ ] Data fitness precedes business detection.
- [ ] Observation, evidence, decision, effect, and outcome remain distinct.
- [ ] State, events, timers, approvals, effects, and checkpoints survive failure.
- [ ] Unknown effects reconcile before retry.
- [ ] Human acknowledgement and high-impact decision are meaningful and capacity-supported.
- [ ] Local replay, fault/adversarial tests, shadow, canary, SLO, and rollback gates pass.
- [ ] Costs include query, model, infrastructure, false-alert work, and missed-event risk.
- [ ] Failures and outcomes feed reviewed evaluation, not online self-modification.
- [ ] Every watch can be suspended, corrected, retired, and audited.

## Related implementation guides

- [Mission and architecture](01-mission-boundary-and-architecture.md)
- [Watch and data contracts](02-watch-semantic-freshness-and-quality-contracts.md)
- [Detection and decisions](03-detection-triage-routing-and-decision-workflows.md)
- [State, memory, and context](04-state-events-effects-memory-and-context.md)
- [Tools and security](05-tools-integrations-security-and-governance.md)
- [Reliability](06-reliability-idempotency-and-reconciliation.md)
- [Evaluation and observability](07-evaluation-observability-and-failure-injection.md)
- [Operations](08-deployment-scale-incidents-cost-and-evolution.md)
- [Adapter qualification](10-adapter-qualification-and-integration-playbooks.md)

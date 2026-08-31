# Deployment, Scale, Incidents, Cost, and Evolution

## Production topology

Start with a modular monolith and independent worker pools where credentials or load require isolation:

~~~mermaid
flowchart TB
    LB[Identity-aware API and scheduler] --> CQ[Controller queue]
    CQ --> CW[Controller workers]
    CW --> DB[(Relational state and outbox)]
    CW --> RQ[Read/query queue]
    RQ --> RW[Semantic and evidence workers]
    CW --> MQ[Triage queue]
    MQ --> MW[Model workers]
    CW --> EQ[Effect queue]
    EQ --> EW[Credential-isolated effect workers]
    CW --> VQ[Reconcile and outcome queue]
    VQ --> VW[Reconcilers and verifiers]
    RW --> AS[(Evidence artifacts)]
    MW --> AS
    EW --> DB
    VW --> DB
    DB --> AO[Audit and observability export]
~~~

One codebase can provide these deployables. Isolation is useful because effect workers have different credentials and model/query workers have different cost and latency. Do not split services solely to mirror the diagram.

## Workload classes and capacity model

Separate at least:

- critical real-time/event watches;
- critical scheduled watches;
- standard scheduled watches;
- acknowledgement and escalation timers;
- effect reconciliation;
- outcome checks;
- historical replay/backfill;
- shadow/candidate evaluation.

Estimate:

`evaluations_per_second = active_watches / cadence_seconds + event_burst + replay_rate`

Then multiply by observed:

- semantic query latency and concurrency;
- data-health and lineage fan-out;
- detector CPU/memory;
- probability of case creation;
- probability and turns of model triage;
- delivery and reconciliation calls;
- outcome checks per case.

Capacity testing must preserve the distribution of watch cost, tenant skew, top-k group-bys, data-ready bursts, and shared upstream incidents. An average-watch benchmark is misleading.

## Queues, admission, and fairness

Use separate queues or scheduling classes so backfills and model triage cannot block due evaluations or unknown-effect reconciliation.

Admission decision inputs:

- watch criticality and explicit deadline;
- data readiness and remaining correction window;
- tenant entitlement and recent consumption;
- owner/channel alert budget;
- semantic/query provider quota and estimated bytes;
- model/provider quota and predicted tokens;
- open-case and in-flight-effect counts;
- regional/cell health.

### Fairness

Apply both:

- global protection for state, providers, and on-call capacity;
- per-tenant, watch-family, data-product, and destination budgets.

Reserve capacity for reconciliation and control commands. Otherwise an overload can prevent the system from determining whether it duplicated an effect.

### Backpressure contract

Every admitted item has priority, deadline, cost estimate, retry budget, tenant, and idempotency key. Every denied or deferred item produces a reason and next eligibility time. Queue age is measured by class and deadline, not one aggregate average.

### Recovery-load contract

Recovery is a separate workload, not ordinary traffic with a larger queue. Before releasing a backlog:

1. enumerate exact watch/interval, timer, effect-reconciliation, and outcome-check identities from authoritative state;
2. classify items as still decision-useful, replay-for-record-only, superseded, expired, or unsafe until provider/state reconciliation;
3. reserve capacity for live critical evaluations, control commands, unknown effects, acknowledgements, and incident work;
4. admit recovery queries by deadline, criticality, tenant fairness, estimated warehouse cost, data readiness, and owner/channel capacity;
5. default expired intervals to observation/detector replay with ordinary external delivery disabled;
6. prevent late/replayed observations from entering a baseline or outcome corpus twice;
7. continuously project drain time and stop when recovery threatens live SLOs, provider quotas, or human capacity.

Measure `recovery amplification = recovery query/effect work / missed logical items`. Values above one can be valid because reconciliation and revisions add work, but unexpected growth is a retry, fan-out, or identity defect. Load tests must cover a scheduler gap coinciding with a data-ready burst, a provider throttle, one noisy tenant, and a shared upstream correction. The exit condition is a bounded drain time with no starvation, alert flood, blind effect replay, or baseline contamination.

## Partitioning and cells

Scale in this order:

1. tune queries, eliminate unnecessary slices, and group schedules;
2. separate workload-class queues and worker pools;
3. partition by tenant/watch ID while keeping one logical control plane;
4. introduce regional or tenant cells when isolation, data residency, or failure containment requires it.

A cell includes state partition, queues, workers, credentials, artifact namespace, quotas, and reconciliation. Global services should be limited to configuration distribution and fleet visibility. A cell failure must not require cross-tenant data access to recover.

Partition keys must keep a case and its timers/effects together. Hot watch families and one large tenant need explicit sub-partition or reserved capacity rather than random redistribution that breaks ordering.

## Deployment environments

| Environment | Data/effects | Purpose |
|---|---|---|
| Unit/contract | Synthetic fixtures; fake providers | Fast deterministic correctness |
| Replay | Approved historical snapshots; no external delivery | Detector and workflow evaluation |
| Integration | Non-production systems; synthetic destinations | Adapter and identity semantics |
| Shadow production | Live reads under production rights; effects captured, not sent | Candidate/current comparison |
| Canary production | Small approved watches/tenants/destinations | Real delivery and SLO verification |
| General production | Policy-scoped | Normal operation |

Do not send shadow alerts to ordinary owners. Store them in a review surface with the exact hypothetical route and cost.

## Behavior-bundle release manifest

~~~yaml
behavior_bundle_id: bi-monitoring/2026.08.31.2
controller:
  image: sha256:912...
  state_schema: 14
  event_schema_bundle: events/11
  watch_schema: watch/3
detectors:
  registry_digest: sha256:88d...
  compatibility: detector-input/6
semantics:
  approved_metric_catalog: sha256:7e9...
  calendars_and_reference_data: refs/2026-08-31
adapters:
  semantic: {release: semantic-dbt-prod/7.3, dossier: artifact://adapter-dossiers/semantic-dbt/7.3}
  warehouse: {release: warehouse-bigquery/6.1, dossier: artifact://adapter-dossiers/bigquery/6.1}
  quality_lineage: {release: data-evidence/4.2, dossier: artifact://adapter-dossiers/data-evidence/4.2}
  itsm: {release: jira-cloud-v3/4.4, dossier: artifact://adapter-dossiers/jira/4.4}
  notifications: {release: delivery/8.1, dossier: artifact://adapter-dossiers/delivery/8.1}
model:
  provider: approved-provider
  release: model://triage-small/2026-08-20
  prompt_bundle: triage/12
  context_compiler: context/8
tools:
  contract_bundle: tools/19
policy:
  bundle: policy/31
  decision_tables: decisions/2026-08-28
runtime:
  durable_controller: runtime/7
  queue_config: queues/9
eval:
  corpus: bi-monitoring/2026-08-31
  report_ref: artifact://release-eval/228...
operations:
  dashboards: dashboards/14
  runbooks: runbooks/12
  kill_switches: kill-switches/7
  capacity_profile: capacity/2026-08-28
  rollback_bundle: bi-monitoring/2026.08.18.1
approvals:
  security: review://sec/821
  platform: review://sre/671
  business: review://metric-owners/92
~~~

The bundle is the complete unit of behavior, authority, compatibility, evaluation, operation, and rollback. Pin provider model versions where possible. If a provider uses a mutable alias or continuously delivered API, pin the observed provider fingerprint and conformance report; detected change becomes a candidate bundle.

## Release strategy

~~~mermaid
flowchart LR
    D[Candidate manifest] --> O[Offline corpus]
    O --> R[Historical replay]
    R --> S[Live shadow]
    S --> C[Watch/tenant canary]
    C --> G[Gradual promotion]
    G --> M[Post-release monitor]
    O --> X[Reject]
    R --> X
    S --> X
    C --> RB[Rollback]
    G --> RB
~~~

Compare candidate and current systems on:

- accepted/skipped observations and reasons;
- signal/case diffs and alert load;
- route, redaction, and approval diffs;
- model facts, citations, proposed reads, and stop behavior;
- query/model/delivery cost and latency;
- state/effect compatibility and unknown outcomes;
- per-tenant/owner capacity.

Promotion is by bundle and cohort, never an untracked prompt, metric, adapter, detector, route, or decision-table edit. Rollback first stops new admissions for the affected cohort, fences old/new effect workers, preserves the operation-key ledger, reconciles `attempting` and `unknown` effects, then resumes only compatible cases. A rollback must not reinterpret an old observation under a different semantic or outcome definition.

### Database and event migrations

- Expand schemas before code uses new fields.
- Write/read both versions when necessary.
- Preserve old event readers for the supported rollback window.
- Migrate snapshots or rebuild them from events; do not rewrite history.
- Test rollback after the migration, not only forward deployment.
- Fence old workers before promoting incompatible effect behavior.

## Cost model

Track cost per eligible evaluation, accepted observation, opened case, acknowledged case, and verified outcome.

Cost components:

- semantic/warehouse compute and scanned bytes;
- data-quality/lineage/catalog calls;
- detector compute and baseline storage;
- model input/output tokens and retries;
- queue, database, artifact, audit, and telemetry storage;
- notification/ITSM/provider APIs;
- operator review, acknowledgement, and incident labor;
- false-alert work and missed-event impact.

### Cost controls

- query one governed metric once and reuse the observation across related detectors where semantics permit;
- group schedules around data readiness without creating a thundering herd;
- cache only versioned non-sensitive metadata with bounded freshness;
- precompute approved aggregates when query economics justify it;
- invoke model triage only after valid material signals;
- use a small approved model for schema-constrained triage and escalate by measured need;
- cap drill-down rows, bytes, calls, tokens, and wall time;
- store hashes/references instead of copying full query results into every record;
- tier retention for artifacts, traces, and outcome episodes;
- retire watches whose alert load is not producing decisions.

Cost reduction must not weaken data gates, rights, reconciliation, or evidence retention required for audit.

## SLO ownership

| SLO class | Owner | Example breach response |
|---|---|---|
| Evaluation coverage/freshness | Platform and data product owner | Catch-up, capacity shift, source incident |
| Detection latency and correctness | Watch/metric owner plus platform | Suspend or roll back detector/watch |
| Delivery/reconciliation | Platform | Provider failover, reconcile, kill switch |
| Acknowledgement/decision | Accountable business organization | Route fallback, capacity review |
| Outcome verification | Watch owner and outcome-source owner | Repair definition/source or mark indeterminate |
| Privacy/security | Security/privacy | Containment, credential revoke, evidence preservation |

Do not assign a business team's acknowledgement breach to the model provider, or a stale data breach to the detector.

## Incident taxonomy

| Incident | Examples | First containment |
|---|---|---|
| False-alert storm | Semantic drift, bad baseline, shared data failure | Suspend affected watches/routes; preserve evaluation records |
| Missed-alert gap | Scheduler outage, quota exhaustion, silent gate failure | Restore coverage; enumerate missed intervals; controlled replay |
| Duplicate/unknown effects | Provider timeout, retry amplification | Freeze key; reconcile before new effects |
| Data exposure | Cross-tenant retrieval, unsafe destination, prompt leak | Kill affected tools/routes; revoke credentials; preserve audit |
| State corruption | Invalid transition, schema bug, lost frontier | Stop writers; validate/rebuild from authoritative log/backup |
| Model regression/injection | Unsupported facts, unsafe read proposal | Disable model path; use deterministic template |
| Owner/route failure | Directory drift, channel outage, alert overload | Fallback route and capacity incident |
| Outcome integrity failure | Wrong definition, confounded attribution, missing source | Stop learning/promotion; mark episodes indeterminate |

## Incident command sequence

1. Identify scope by tenant, release, watch versions, semantic snapshots, detector/baseline, route, provider, and interval.
2. Activate the narrowest kill switch that contains new harm.
3. Preserve state, effect intents, receipts, evidence manifests, traces, and audit.
4. Reconcile in-flight and unknown effects before replay or rollback.
5. Establish whether observations, alerts, decisions, deliveries, and outcomes are affected separately.
6. Roll back the manifest or suspend affected watch versions.
7. Communicate corrections/retractions to prior recipients under destination policy.
8. Recover coverage in priority order with delivery disabled unless explicitly approved.
9. Verify state/effect consistency, backlog, SLOs, and owner capacity.
10. Create regression fixtures, runbook changes, and a reviewed prevention action.

### Runbook: false-alert storm

- Trigger alert-volume kill switch by route/watch family, not global shutdown unless necessary.
- Continue recording evaluations and data-health evidence.
- Disable model triage first if it amplifies cost; preserve deterministic diagnosis.
- Identify whether the root is semantics, quality, baseline, detector, correlation, or delivery.
- Correlate and inhibit child cases; do not mass-close without dispositions.
- Reconcile messages/tasks already sent.
- Shadow corrected versions against affected intervals.
- Require metric/watch owner approval before resuming delivery.

### Runbook: missed evaluation gap

- Enumerate exact watch/interval keys and their data readiness.
- Prioritize watches by decision window and criticality.
- Decide whether historical delivery is useful; default to replay without notification for expired windows.
- Bound semantic/query load and owner alert capacity.
- Record skipped/expired outcomes explicitly.
- Verify no baseline or outcome episode incorporated the gap incorrectly.

### Runbook: privacy or cross-tenant incident

- Disable affected query/retrieval/model/destination capabilities.
- Revoke short-lived and standing connector credentials.
- Preserve access and effect evidence under incident/legal policy.
- Identify source, prompt, output, logs, artifacts, recipients, and retention copies.
- Do not use model summarization to determine exposure scope.
- Follow organizational and jurisdictional notification/response obligations.
- Add cross-tenant and data-minimization regression cases before re-enable.

## Disaster recovery

Define asset-specific objectives; illustrative values below are placeholders for owner approval, not universal targets:

| Asset | Example RPO | Example RTO | Recovery proof |
|---|---:|---:|---|
| Watch/policy/behavior-bundle registry | 15 minutes | 1 hour | Restore exact active/effective versions and signatures |
| Case, timer, approval and effect ledger | Near-zero in-region; 5 minutes cross-region | 30 minutes critical cell | Rebuild legal state/frontier; reconcile before effect admission |
| Evidence and outcome artifacts | 1 hour | 4 hours for critical cases | Hash verification, missing-object inventory, rights and retention preserved |
| Audit records | Organization policy; no accepted silent loss | 4 hours query access | Completeness frontier and tamper-evidence verification |
| Queue and caches | Zero durability assumed | 30 minutes from authoritative state | Reconstruct without duplicate semantic work/effects |
| Metrics, logs and sampled traces | Class-specific | Operations visibility within 1 hour | Gaps are explicit; never used to reconstruct authority |

Also test:

- RPO/RTO for watch configuration, case/effect state, artifacts, audit, and telemetry separately;
- encrypted backups with tenant/region rules;
- queue and timer reconstruction from authoritative state;
- effect reconciliation after restore so old intents are not replayed blindly;
- semantic/query credential reissuance;
- artifact integrity and missing-object detection;
- regional failover fencing to prevent two active effect writers;
- manual degraded operation when semantic, model, or effect provider is unavailable.

A restore test is incomplete until timers, unknown effects, approvals, and outcome checks behave correctly. Rehearse loss of one cell, one region, the artifact store, provider credentials, and the control plane. Measure recovery load, projected backlog drain, provider/warehouse saturation, and live-work starvation. Failover requires a fencing lease or equivalent proof that only one cell can execute effects for a partition; DNS or deployment state alone is insufficient.

## Upgrade and compatibility policy

### Semantic/detector changes

Historical replay -> live shadow -> owner review -> version promotion. Never back-edit past observations.

### Model/prompt/context changes

Run repeated trajectory evaluation and injection tests. Compare citation fidelity, tool proposals, latency, tokens, and abstention. Keep deterministic fallback.

### Adapter/provider changes

Contract tests must cover partial/unknown responses, pagination, idempotency retention, consistency windows, rate limits, and deletion/disable lag. Canary with synthetic or low-risk effects.

### Runtime/state changes

Replay old events and resume checkpoints under the candidate. Test crash boundaries, effect fencing, downgrade, and rollback.

### Policy/decision changes

Treat as authority releases with named owner approval, effective time, migration behavior for open cases, and audit. No automatic rollout from outcome statistics.

## Vendor maturity drift

BI and real-time action products evolve quickly. Representative current capabilities include dashboard-tile alerts, daily metric digests, refresh-driven alert rules, and event-driven stateful rule engines. Cadence, permissions, preview/stable status, throughput, deletion behavior, and automation integrations vary and change.

At procurement or release time, verify:

- the current API/version and lifecycle guarantees;
- per-rule/watch/event limits and quota behavior;
- alert identity under edit, clone, migration, and deletion;
- data/source ownership and recipient permission semantics;
- idempotency, delivery receipts, reconciliation lookup, and consistency;
- model-assisted explanation defaults and data processing;
- preview/GA status and region availability;
- export, audit, retention, deletion, and licensing/terms.

Product documentation is evidence of intended mechanics, not a substitute for integration fault tests.

Controlled failure mining closes the loop. Review every invariant breach, near miss, manual reconciliation, provider drift, false/missed case, privacy rejection, rollback, and recovery overload. Redact it into a reproducible state/adapter fixture, assign an owner and affected bundle components, add it to the narrowest hard gate, and track recurrence. Never promote raw production narratives or outcomes directly into prompts, thresholds, routes, or detector training. Human review must separate data correction, policy disagreement, expected abstention, operator error, and genuine system defect.

## Production readiness checklist

- [ ] Workload classes, peak bursts, tenant skew, and data-ready patterns are capacity-tested.
- [ ] Reconciliation and control commands retain reserved capacity.
- [ ] Admission creates visible deferred/skipped states.
- [ ] Cell/partition design preserves case/effect ordering and data residency.
- [ ] Shadow and canary paths cannot accidentally deliver ordinary alerts.
- [ ] Behavior-bundle manifest pins every behaviorally relevant component and adapter dossier.
- [ ] Schema/event migration and rollback are exercised.
- [ ] Unit economics include human alert and incident cost.
- [ ] Scoped kill switches and incident runbooks are tested.
- [ ] Restore/failover reconciles effects before replay.
- [ ] Model, semantic, detector, adapter, runtime, and policy upgrades have separate gates.
- [ ] Vendor capabilities, limits, licensing, and preview status are rechecked at deployment time.

## Canonical references

- [Scaling, capacity, and SLOs](../../operations/scaling-capacity-and-slos.md)
- [Queues, scheduling, and backpressure](../../operations/queues-scheduling-and-backpressure.md)
- [Model routing, cost, and latency](../../operations/model-routing-cost-and-latency.md)
- [Deployment, release, and incident response](../../operations/deployment-release-and-incident-response.md)
- [Microsoft Fabric Activator overview](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/data-activator/activator-introduction)
- [Looker alerts](https://docs.cloud.google.com/looker/docs/creating-alerts)
- [Tableau Pulse alerts](https://help.tableau.com/current/online/en-us/pulse_alerts.htm)
- [Power BI data alerts](https://learn.microsoft.com/en-us/power-bi/create-reports/service-set-data-alerts)

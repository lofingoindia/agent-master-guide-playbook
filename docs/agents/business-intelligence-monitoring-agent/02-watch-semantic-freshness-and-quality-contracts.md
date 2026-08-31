# Watch, Semantic, Freshness, and Quality Contracts

## Contract hierarchy

A watch is not “metric X > 100.” It is a versioned agreement connecting metric meaning, evaluation time, data fitness, detection, accountability, and retirement:

~~~mermaid
flowchart LR
    MC[Metric contract] --> WC[Watch contract]
    DC[Data product contract] --> WC
    RC[Rights and purpose policy] --> WC
    WC --> EV[Evaluation]
    EV --> OB[Observation]
    OB --> DE[Detector evidence]
    DE --> CA[Case]
~~~

The metric contract remains owned by the semantic layer and metric owner. Reuse the [Analytics Agent metric semantics guide](../analytics-agent/02-metric-semantics-and-governed-discovery.md) for entity, measure, population, time, aggregation, additivity, join, and dimension rules. This guide adds monitoring-specific fields; it does not create a second metric definition.

## The watch contract

The active watch pointer refers to an immutable version. Editing creates a draft version, historical replay, review, and promotion. Never mutate a version that has produced an observation.

~~~yaml
watch_id: revenue-drop-emea
version: 17
tenant_id: commerce
status: active                 # draft | shadow | active | suspended | retired
purpose: detect material revenue deterioration for daily commercial triage
owners:
  metric_owner: role:finance-metrics
  watch_owner: role:commercial-operations
  accountable_operator: group:emea-revenue-oncall
metric:
  ref: semantic://net_revenue
  semantic_snapshot_policy: pinned-compatible
  required_contract_version: ">=8 <9"
  grain: day
  filters:
    region: EMEA
  allowed_triage_dimensions: [country, channel, product_family]
  forbidden_dimensions: [customer_id, sales_rep]
  minimum_cohort: 50
schedule:
  cadence: "0 07 * * *"
  timezone: Europe/Paris
  evaluation_delay: PT2H
  expected_interval: P1D
  correction_window: P3D
data_gate:
  maximum_age: PT26H
  minimum_interval_coverage: 0.995
  required_assertions: [revenue_not_null, order_key_unique, currency_rate_fresh]
  lineage_required: true
  on_failure: open_data_health_case
detector:
  type: seasonal_relative_change
  version: detector://seasonal-relative/4.2.1
  baseline_ref: baseline://revenue-drop-emea/2026-08-01
  trigger:
    relative_change_lte: -0.12
    material_amount_gte: 100000
    persistence: 2
  recovery:
    relative_change_gt: -0.06
    persistence: 2
  cooldown: P2D
decision:
  table_ref: decision://revenue-watch/6
  severity: high
  acknowledgement_due: PT2H
  disposition_due: P1D
  outcome_due: P14D
delivery:
  route_ref: route://commercial-emea/3
  destinations: [itsm]
privacy:
  purpose_ref: purpose://commercial-monitoring
  classification: confidential-aggregate
  retention: P400D
release:
  approved_by: [user:metric-owner-17, user:ops-owner-9]
  approved_at: 2026-08-15T12:00:00Z
  review_due: 2026-11-15
  change_ticket: CHG-4812
~~~

### Required invariants

- `watch_id` is stable; `version` identifies the complete behavior.
- Owners are resolvable identities, not free text.
- The semantic contract and compatible version range are explicit.
- Timezone, event interval, expected completion, and correction window are distinct.
- Data-health failure behavior is explicit and cannot silently fall through to business detection.
- Trigger and recovery are distinct when hysteresis is needed.
- Materiality and persistence accompany statistical significance.
- Route and decision-table versions are pinned.
- Privacy purpose, data class, allowed dimensions, minimum cohort, retention, and destinations are enforceable fields.
- Promotion records reviewers and evidence; retirement records why the watch no longer justifies alert load.

## Semantic-layer contract

A metric is a named, queryable calculation over governed semantic models and query scope—not merely a column. Monitoring adds risks because the same query repeats over time and can silently change when semantics, joins, calendars, or dimensions evolve.

Each evaluation must capture:

| Field | Why it matters |
|---|---|
| `metric_ref` and semantic snapshot | Reconstructs meaning at evaluation time |
| Entity, grain, calendar, timezone | Prevents interval and period-boundary ambiguity |
| Measure and aggregation | Detects non-additive or ratio misuse |
| Population and fixed filters | Separates a changed population from a changed value |
| Allowed group-bys and join path | Prevents fan-out, unsafe slices, and model-authored joins |
| Query request and normalized digest | Supports replay without retaining secrets in logs |
| Generated query artifact reference | Enables review while keeping query generation outside the model |
| Source/table snapshot or as-of metadata | Anchors the observation to source state |
| Semantic engine and adapter version | Exposes runtime drift |

### Semantic changes

Classify a change before promotion:

| Change | Default response |
|---|---|
| Description only | Review; no baseline change if executable semantics are identical |
| Add a safe dimension | Shadow only; do not expand model rights automatically |
| Measure, population, join, calendar, or aggregation change | New watch compatibility review and usually new baseline |
| Historical backfill under unchanged semantics | Revision flow within correction policy |
| Renamed or deprecated metric | Explicit migration with parallel comparison |
| Owner or data-class change | Reauthorize routes, retention, and prompt eligibility |

Run the old and new semantic snapshots on representative history and a live shadow period. Compare values, missingness, coverage, slice cardinality, detector behavior, and alert load. A semantic version that compiles is not necessarily monitoring-compatible.

## Time semantics

Business monitoring commonly mixes five clocks:

1. **Event time:** when the business event occurred.
2. **Source update time:** when the source recorded or changed it.
3. **Availability time:** when governed data became queryable.
4. **Evaluation time:** when the watch queried it.
5. **Decision/effect time:** when a case was routed or acted upon.

Persist all relevant clocks. A daily watch must identify the exact business interval, calendar, and timezone; daylight-saving transitions and fiscal calendars cannot be inferred from a cron string.

### Late and revised data

For streams, a watermark is an estimate that earlier event-time data is probably complete, not proof. For batch systems, an upstream completion marker can still precede late corrections. Choose per-watch behavior:

- **wait:** increase evaluation delay for completeness;
- **provisional:** publish a clearly labeled preliminary observation and forbid high-impact routing;
- **revise:** emit a new observation with `supersedes_observation_id`;
- **close window:** ignore later data for alert state but retain it for evaluation and audit;
- **reopen:** update a closed case only under an owner-approved materiality rule.

The latency/completeness trade-off is business-specific. Record it in the contract and test it with real delay distributions.

## Observation contract

An observation is immutable evidence of what a governed query returned under stated conditions. It is not yet an alert.

~~~yaml
observation_id: obs_01K...
watch:
  id: revenue-drop-emea
  version: 17
tenant_id: commerce
interval:
  start: 2026-08-28T00:00:00+02:00
  end: 2026-08-29T00:00:00+02:00
  event_time_watermark: 2026-08-29T06:15:00Z
evaluated_at: 2026-08-29T07:02:11Z
semantic:
  metric_ref: semantic://net_revenue
  snapshot: sha256:4b7...
  request_digest: sha256:12e...
  query_artifact_ref: artifact://query/98a...
value:
  scalar: 731204.14
  unit: EUR
  status: present               # present | no_data | partial | error
data_health:
  verdict: accepted             # accepted | stale | invalid | partial | indeterminate
  freshness_age: PT24H47M
  coverage: 0.999
  assertion_run_ref: quality://run/7731
  lineage_ref: lineage://run/a98...
rights:
  decision_id: authz_8d...
  purpose: commercial-monitoring
revision:
  number: 0
  supersedes_observation_id: null
integrity:
  schema_version: observation/1.0
  content_hash: sha256:8aa...
~~~

### No-data is not zero

Use at least four typed outcomes:

- `present`: valid value exists;
- `no_data`: the expected population produced no rows;
- `partial`: known incomplete interval or subset;
- `error`: query or adapter failed.

Whether `no_data` is expected, a business signal, or a data-health incident belongs in the watch contract. Never coerce it to numeric zero inside the detector.

## Freshness gate

Freshness has multiple meanings:

| Dimension | Example check |
|---|---|
| Source freshness | Latest source update within contract |
| Pipeline freshness | Required data product completed for interval |
| Metric freshness | Semantic result covers required event-time interval |
| Reference-data freshness | Currency, target, calendar, or hierarchy version current |
| Evidence freshness | Ownership, suppression, incident, and deployment context recent enough |

The gate returns `accepted` only when all required checks are evidenced. A source timestamp alone is insufficient if the metric query excludes a delayed partition.

## Data-quality gate

Data quality is contextual: dimensions and tolerances depend on the intended decision. Useful dimensions include accuracy, completeness, consistency, timeliness, validity, and uniqueness; no universal checklist can prove fitness.

Model the gate as assertions with severity and enforcement policy:

~~~yaml
quality_evidence:
  assertion_id: currency_rate_fresh
  assertion_version: 5
  success: false
  severity: critical
  expected: maximum_age <= PT26H
  actual: PT49H
  evaluated_at: 2026-08-29T06:45:00Z
  dataset_ref: data://finance/daily_currency_rates
  lineage_ref: lineage://run/7ca...
  enforcement: reject_business_detection
~~~

OpenLineage's data-quality assertion and metric facets are useful interchange shapes: they separate assertion outcome, severity, and observed metrics. They do not decide whether a particular watch may continue; that remains local policy.

### Gate outcomes

| Verdict | Business detector | Case behavior |
|---|---|---|
| Accepted | Run | Continue normally |
| Stale | Do not run unless watch explicitly tolerates it | Open/correlate data-health case |
| Invalid | Do not run | Quarantine observation and route DataOps |
| Partial | Run only a detector designed for partial coverage | Mark provisional; prohibit high-impact route |
| Indeterminate | Do not guess | Retry bounded reads, then escalate |

Quality gate failures must be observable. Silent suppression hides broken monitoring.

## Baseline contract

Every statistical detector needs a reproducible baseline artifact:

~~~yaml
baseline_id: baseline://revenue-drop-emea/2026-08-01
detector_version: detector://seasonal-relative/4.2.1
semantic_snapshot: sha256:4b7...
training_interval: [2025-08-01, 2026-07-31]
included_observations_digest: sha256:fa3...
excluded_windows:
  - {start: 2025-11-20, end: 2025-12-05, reason: seasonal_campaign}
calendar_ref: calendar://emea-commercial/9
parameters:
  weekday_seasonality: true
  robust_loss: huber
validation_artifact_ref: artifact://baseline-validation/22a...
approved_by: user:watch-owner-9
expires_at: 2026-11-01T00:00:00Z
~~~

A baseline trained on an outage, promotion, acquisition, backfill, or prior alert is not automatically representative. Version exclusions and document regime changes. Re-baselining is a release, not an invisible maintenance task.

## Source rights and minimization

The watch contract must declare:

- lawful or organizational purpose and permitted use;
- source and semantic-layer roles;
- allowed aggregation level and cohort floor;
- direct and inferred sensitive categories;
- countries/regions where data and prompts may be processed;
- allowed evidence destinations and recipients;
- retention, deletion, legal hold, and access-review rules;
- whether model processing is prohibited, local-only, or allowed after redaction.

Enforce these before querying or retrieving evidence. Post-prompt redaction is too late.

## Contract lifecycle

~~~mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> Replay: schema and ownership valid
    Replay --> Rejected: safety or utility gate fails
    Replay --> Shadow: historical gate passes
    Shadow --> Active: owner and release approval
    Active --> Suspended: data, semantic, rights, or incident stop
    Suspended --> Shadow: corrected version
    Active --> Retired: no longer actionable
    Shadow --> Rejected: alert load or quality failure
    Rejected --> Draft: revised contract
    Retired --> [*]
~~~

Promotion checks:

- contract schema and referential integrity;
- identity and ownership resolution;
- semantic compatibility and query reproducibility;
- freshness/quality behavior across known failures;
- historical event-level detection and alert-load evaluation;
- shadow results with no external delivery;
- rights, destination, and retention decision;
- acknowledgement capacity and runbook readiness;
- signed promotion and rollback pointer.

## Contract failure runbook

1. Suspend affected watch versions without deleting state.
2. Keep evaluation triggers as skipped records so missing coverage is visible.
3. Stop pending notifications whose approvals reference the invalid version.
4. Reconcile in-flight effects; do not assume suspension retracted remote messages.
5. Open a semantic or data-health incident with affected intervals and cases.
6. Produce a corrected immutable version and replay historical plus affected live intervals.
7. Decide explicitly whether prior alerts stand, are revised, or are retracted.
8. Shadow before reactivation; record the change and owner approval.

## Contract checklist

- [ ] Metric meaning is governed elsewhere and referenced by immutable snapshot.
- [ ] Watch purpose, owners, accountable operator, and retirement condition are explicit.
- [ ] Event, availability, evaluation, decision, and correction times are distinguishable.
- [ ] No-data, partial, stale, invalid, and error states are typed.
- [ ] Data-quality assertions have evidence, severity, and enforcement policy.
- [ ] Allowed dimensions, cohort floor, purpose, geography, destinations, and retention are enforced.
- [ ] Detector and baseline versions are reproducible.
- [ ] Trigger, recovery, persistence, cooldown, and materiality are independent fields.
- [ ] Semantic, baseline, route, or rights changes require release review.
- [ ] Historical observations remain immutable and revisions are linked.

## Primary references

- [MetricFlow metric semantics](https://github.com/dbt-labs/dbt-core/blob/main/crates/dbt-metricflow/docs/metric-semantics.md)
- [Open Data Contract Standard](https://bitol-io.github.io/open-data-contract-standard/latest/)
- [OpenLineage data-quality assertions facet](https://openlineage.io/docs/spec/facets/dataset-facets/data_quality_assertions/)
- [OpenLineage data-quality metrics facet](https://openlineage.io/docs/spec/facets/dataset-facets/data_quality_metrics/)
- [UK Government Data Quality Framework](https://www.gov.uk/government/publications/the-government-data-quality-framework/the-government-data-quality-framework)
- [Apache Flink event time and watermarks](https://nightlies.apache.org/flink/flink-docs-stable/docs/learn-flink/streaming_analytics/)

See the [research packet](../../research/packets/business-intelligence-monitoring-agent-blueprint.md) for source caveats and maturity notes.

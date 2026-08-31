# Adapter Qualification and Integration Playbooks

**Research baseline:** 2026-08-31  
**Purpose:** turn a vendor API into a tested, versioned monitoring boundary without giving the model ambient data or effect authority

## Operating rule

An adapter is not a convenience wrapper. It is the place where provider-specific identity, authorization, freshness, cost, pagination, result finality, and effect ambiguity become an application-owned contract. No adapter is production-ready because one happy-path request succeeded.

Qualify the exact product edition, region, API revision, client version, authentication mode, tenant configuration, and enabled features. Pin them in the behavior bundle and rerun conformance tests after any change. Mutable SaaS documentation describes intended behavior; the deployed tenant and fault tests establish the local contract.

Keep the ownership boundary explicit:

| Need | Correct owner | What this adapter layer may do |
|---|---|---|
| Evaluate a named, versioned metric watch repeatedly | This agent | Read the approved metric and preserve observation/effect identity |
| Explore a new question, discover a metric, or make a causal analysis | [Analytics Agent](../analytics-agent/README.md) | Open a bounded investigation request; do not turn free-form SQL into a watch |
| Prepare an executive agenda, synthesize cross-domain priorities, or coordinate commitments | [Executive Operations Agent](../executive-operations-agent/README.md) | Deliver a finalized evidence packet and status; do not infer executive priorities |
| Repair a data product or mutate a business system | Data/platform or accountable operator | Create a governed work item only; never perform the mutation |

## Qualification sequence

~~~mermaid
flowchart LR
    I[Inventory exact deployment] --> C[Declare capability and rights]
    C --> N[Normalize identity and version semantics]
    N --> T[Run conformance tests]
    T --> F[Inject timeouts, partials, duplicates, and drift]
    F --> S[Shadow under production rights]
    S --> A[Approve for bounded watch classes]
    A --> M[Monitor and requalify]
    T --> X[Reject or restrict]
    F --> X
    S --> X
~~~

Promotion is per capability and watch class. An adapter can be approved for aggregate reads but rejected for drill-downs; approved for ticket creation but rejected for paging; approved for one region but not another.

## Adapter dossier

Maintain one signed dossier per deployed adapter release:

~~~yaml
adapter_release: semantic-dbt-prod/7.3
owner: team:data-platform
qualified_at: 2026-08-31T12:00:00Z
refresh_due: 2026-11-29
deployment:
  product: dbt-semantic-layer
  api_or_protocol: environment-api-observed-2026-08-31
  client: internal-semantic-gateway/4.8.1
  region: approved-region
  edition_and_features: tenant-record://semantic-prod
  provider_docs_snapshot: artifact://adapter-research/semantic-dbt/2026-08-31
authentication:
  principal: service:bi-monitoring-prod
  grant_mode: workload-identity
  scopes_or_roles: [metrics-read, dimensions-list]
  prohibited: [raw-sql, model-write, admin]
capabilities:
  reads: [resolve_metric, list_allowed_dimensions, query_metric]
  effects: []
  pagination: cursor
  cancellation: supported_but_not_completion_proof
freshness:
  provider_observed_at: available
  source_event_time: required
  cache_age: explicit
query_cost:
  preflight: engine_specific
  hard_budget: {bytes: 5000000000, wall_time: PT60S, rows: 5000}
identity:
  request: evaluation_id
  remote_query: provider_query_id
  snapshot: semantic_manifest_digest
result_finality:
  terminal_states: [succeeded, failed, cancelled]
  partial_signal: explicit_page_and_completion_fields
reconciliation:
  lookup_keys: [provider_query_id, evaluation_id_tag]
  maximum_visibility_lag: PT5M
  unknown_after: PT15M
conformance_report: artifact://adapter-tests/semantic-dbt/7.3
known_limits:
  - result schemas and metric semantics can change with manifest promotion
  - provider plan and tenant configuration affect available APIs
rollback: semantic-dbt-prod/7.2
approvals: [review://data/221, review://security/450, review://platform/991]
~~~

Do not write `latest`, “v2-ish,” or a product marketing name where an exact deployed version, schema digest, endpoint, or observation date is available. When the provider offers only a continuously delivered SaaS API, pin the contract-test artifact and observation date.

## Common read contract

Every read adapter accepts a controller-issued request and returns a normalized receipt. Provider objects remain attached as immutable evidence, not workflow state.

~~~yaml
read_request:
  request_id: read_01K...
  evaluation_id: eval_01K...
  tenant_id: commerce
  purpose: watch_evaluation
  adapter_release: warehouse-bigquery/6.1
  resource_ref: metric://net-revenue/12
  semantic_snapshot: sha256:7e9...
  event_time_interval: [2026-08-30T00:00:00Z, 2026-08-31T00:00:00Z]
  as_of: 2026-08-31T06:00:00Z
  rights_decision_id: pd_71...
  parameters: {region: EMEA}
  budgets: {rows: 1000, bytes: 5000000000, wall_time: PT60S, pages: 10}
  deadline: 2026-08-31T06:01:30Z
~~~

~~~yaml
read_receipt:
  request_id: read_01K...
  adapter_release: warehouse-bigquery/6.1
  provider_request_id: job_project.region.job_id
  state: complete        # complete | partial | failed | cancelled | unknown
  schema_digest: sha256:2d0...
  page_frontier: {pages: 1, next_cursor: null, provider_complete: true}
  rows_returned: 1
  source_event_time_max: 2026-08-30T23:59:59Z
  source_watermark: 2026-08-31T05:42:00Z
  provider_observed_at: 2026-08-31T06:00:12Z
  cache: {hit: false, age: PT0S}
  cost: {bytes_processed: 104857600, bytes_billed: 104857600, compute_ms: 420}
  result_artifact: artifact://reads/read_01K/result
  result_digest: sha256:0b1...
  warnings: []
  completed_at: 2026-08-31T06:00:13Z
~~~

`HTTP 200` is not `complete`. Completion requires provider terminal state, full pagination, schema validation, bounded cost, rights continuity, and an integrity-protected result. Missing freshness fields produce `indeterminate` data fitness rather than a fabricated timestamp.

## Common effect and reconciliation contract

Notification, ticket, email, page, and workflow writes cross a transaction boundary. Record the intent before calling the provider.

~~~yaml
effect_intent:
  effect_id: effect_01K...
  operation_key: commerce/case_01K/v6/itsm/create
  case_id: case_01K...
  case_version: 6
  adapter_release: jira-cloud-v3/4.4
  capability: create_work_item
  destination_ref: route://commercial-duty/9
  payload_digest: sha256:bc3...
  policy_decision_id: pd_203...
  approval_ref: approval_88...
  deadline: 2026-08-31T06:05:00Z
~~~

The normalized result is one of `CONFIRMED`, `DEFINITIVE_FAILURE`, or `UNKNOWN_OUTCOME`. A timeout, broken connection after upload, asynchronous acceptance, or response that lacks the durable remote identity is `UNKNOWN_OUTCOME`. The reconciler searches by an adapter-qualified operation key, destination, payload digest, and time window. It must return `found_one`, `found_multiple`, `not_found_after_consistency_window`, `still_indeterminate`, or `lookup_unsupported`. Only `not_found_after_consistency_window` can authorize a policy-bounded retry. `found_multiple` is an incident, not an invitation to choose one silently.

Cancellation is also an effect. A local `cancelled` flag cannot prove that a remote message, ticket, or page was not created.

## Capability and rights matrix

Create this matrix from discovery under the production service identity, not an administrator account:

| Capability | Minimum right | Sensitive output | Cost/freshness obligation | Effect and reconciliation obligation |
|---|---|---|---|---|
| Resolve metric/semantic model | Metadata read | Metric logic, owners | Pin manifest/catalog digest and effective time | None |
| Query approved aggregate | Query job + dataset/model access | Aggregates may still be sensitive | Preflight or cap cost; record event time, watermark, cache and final page | Query ID lookup and cancellation state |
| Read dashboard/alert configuration | BI content/alert read | Recipients, thresholds | Record content revision and refresh dependency | None |
| Read quality/lineage evidence | Validation/lineage read | Dataset names, job facets | Record check/run ID, schema/facet version and event completeness | None |
| Create ticket | Project create + field rights | Evidence and assignee | Validate destination schema immediately before effect | Provider object lookup and changelog/remote ID |
| Post chat message | Channel membership plus write scope | Recipients and evidence | Rate-limit and destination-policy check | Channel plus returned message ID; search/update strategy |
| Send email | Mail-send permission | Recipient list and content | Treat asynchronous acceptance separately from delivery | Message trace/search if locally supported; otherwise restrict authority |
| Trigger page | Service integration route | On-call identity/urgency | Budget pages and deduplicate cases | Stable dedup key, incident/alert lookup, explicit resolve policy |
| Start decision/workflow | Execute named definition | Approval inputs | Pin decision/workflow definition and input digest | Workflow/run ID, terminal-state query, signal idempotency |

Rights are structural. The adapter receives a typed `resource_ref`, approved fields/dimensions, purpose, tenant, and policy decision; it does not receive a prompt and decide what to query.

## Representative read-adapter playbooks

### Semantic layer and metric store

Qualify metric identity, semantic-model identity, allowed group-bys, join-path ambiguity, time grain, timezone, currency/unit conversion, null policy, saved-query behavior, generated SQL evidence, and manifest promotion. A metric name alone is not an immutable identity.

For dbt MetricFlow, the official repository describes compilation of versioned metric definitions into engine-specific SQL. Its release history shows breaking API and compatibility changes, including a current `0.210.0` release in April 2026 and earlier dbt-core compatibility constraints. Therefore:

- pin the MetricFlow/dbt packages or the hosted API observation date, semantic manifest digest, adapter, and warehouse dialect;
- run golden queries for additive, non-additive, cumulative, conversion, offset, null-dimension, timezone-boundary, and ambiguous-join cases;
- store normalized query parameters and generated-query digest without treating SQL text as the metric definition;
- reject a watch if a saved query or allowed dimension disappears rather than silently changing its grain;
- shadow every semantic-manifest promotion against recent closed and open intervals.

Representative alternative: Cube or a native BI semantic model can fit, but it must pass the same identity, snapshot, time, rights, and generated-query tests. Product labels such as “semantic layer” or “metric store” do not prove them.

### Warehouse, lakehouse, and query engine

Exercise at least one serverless job API, one warehouse/session API, and one asynchronous SQL endpoint if the organization uses them:

| Representative surface | Useful provider mechanics observed at research date | Qualification limit |
|---|---|---|
| BigQuery REST `jobs.query`/`jobs.getQueryResults` | `jobReference`/`queryId`, `jobComplete`, page token, `totalBytesProcessed`, `totalBytesBilled`, `maximumBytesBilled`, dry run and request ID | First response can be incomplete; dry-run estimates have documented limitations, especially external sources; data permissions depend on the statement |
| Snowflake query history | Unique `query_id`, `query_tag`, execution status, role/warehouse/session and cost/runtime fields | History visibility and latency depend on function/view, role and retention; a tag is correlation metadata, not an idempotency guarantee |
| Databricks SQL Statement Execution API 2.0 | Asynchronous `statement_id`, status polling and chunked result retrieval | Default wait can return only pending state; every chunk and disposition must be consumed and verified |

Conformance tests cover server-side timeout versus client timeout, cancellation race, result expiry, pagination/chunk corruption, reordered columns, decimal/timezone serialization, row/column truncation, cache behavior, query-history lag, quota/throttle, credentials expiring mid-query, and a cost preflight disagreeing with final cost. The controller marks cost and completeness explicitly; it never converts partial results to a normal observation.

### BI and dashboard surfaces

Native alerts can be the simplest valid solution. Qualify them before building this controller, then qualify read adapters for coexistence:

- dashboard/content ID and revision under edit, copy, migration, ownership transfer, deletion, and folder/workspace move;
- alert identity, recurrence, recovery, mute/suppress, recipient permissions, evaluation cadence, data-refresh dependency, and delivery history;
- export/API result versus rendered dashboard parity, including filters, row-level security, locale, timezone, cached tiles, and incomplete refresh;
- whether an alert is personal, content-owner scoped, workspace scoped, or service-principal manageable;
- plan/edition, regional availability, preview/GA status, API quotas, audit export, retention, and deletion behavior.

Representative Looker alerts, Tableau Pulse capabilities, and Power BI/Fabric data activations evolve frequently. Record the exact tenant evidence and retest quarterly; do not copy a current product limit into a permanent architecture invariant.

### Data quality and lineage

Normalize quality evidence as `accepted`, `stale`, `partial`, `invalid`, or `indeterminate`; never as a single boolean. Pin suite/check/contract version, validation run, expected population, observed population, assertion results, evaluation time, source watermark, and correction window.

OpenLineage is an event model, not proof that lineage is complete. The current API documentation exposes versioned schema URLs and run events such as `START`, `RUNNING`, `COMPLETE`, `ABORT`, `FAIL`, and `OTHER`. Test missing start/terminal events, duplicate/out-of-order events, reused dataset names, changing namespaces, late facets, and a lineage backend lagging behind the data. A provider `COMPLETE` job state does not prove the downstream dataset is fresh or correct.

For Great Expectations, Soda, dbt tests, or a native quality service, qualify check identity, version, batch/partition identity, result retrieval, sampled versus full-population scope, retry/re-run semantics, and whether a pass can be revised. Preserve the raw signed result as evidence and keep the application-owned gate decision separate.

### Scheduler and event intake

CloudEvents `1.0.2` is a stable interchange envelope, not an exactly-once guarantee. Verify event ID scope, producer/source identity, subject, event time, schema, duplicate delivery, reorder window, retention, dead-letter policy, and replay behavior.

Apache Airflow `3.3.x` documentation provides representative asset-aware, event-driven, and backfill mechanics. Asset updates can coalesce, external events can create queued asset events, and backfills have explicit reprocessing and concurrency choices. Treat `dag_id`, run ID, logical interval, asset URI, source event ID, and reprocessing request as separate identities. A DAG success or asset update is only a readiness input; the monitoring data gate still checks the actual governed metric interval.

## Representative effect-adapter playbooks

### ITSM and work management

Before enabling creation, discover allowed project/queue, issue type, required/custom fields, assignee semantics, workflow states, attachment limits, visibility/security level, and current user rights. Jira Cloud REST API v3 is a representative surface: issue creation returns `201`, fields are controlled by create metadata, and some create-metadata surfaces are deprecated or split into scoped endpoints. Pin the endpoint and schema actually used.

Use the operation key in a provider-supported external ID/property when possible. Otherwise store a deterministic marker in an approved searchable field and qualify its consistency and uniqueness. Test a commit with lost response, duplicate create, schema change, permission revocation, field option removal, assignment race, transition race, eventual search visibility, and manual remote edit. Reconciliation records remote history; it never overwrites an operator change blindly.

### Chat and email

Slack `chat.postMessage` returns a channel and `ts`, may sanitize the submitted message, requires appropriate membership/scopes, and documents per-channel and workspace rate constraints with `Retry-After`. Store the returned remote form and ID. Test missing channel membership, archived/restricted channel, renamed destination, thread-parent deletion, 429 handling, and a response lost after commit. Do not claim deduplication unless a local operation marker/search strategy has been proven.

Microsoft Graph `sendMail` returns `202 Accepted`; the official documentation warns that delivery remains subject to Exchange Online limitations and throttling. Therefore `202` is accepted-for-processing, not delivered or read. If the deployment cannot reconcile a durable message/trace identity, restrict email to low-risk communication, retain `UNKNOWN_OUTCOME`, and avoid blind retry. Read receipts are neither authenticated acknowledgement nor outcome.

### Paging and incident systems

PagerDuty Events API v2 is representative of deduplicating event intake: matching case-sensitive `dedup_key` values group related alerts. Build the key from tenant, case/root identity and route version; keep trigger, acknowledge, and resolve policies separate. Qualify service routing, urgency/orchestration, dedup-key retention, event acceptance versus incident creation, REST lookup rights, suppression, maintenance windows, and resolve-after-revision behavior. Never reuse one key across tenants or unrelated incidents.

Page only when a named human response exists. A repeated low-confidence business anomaly normally belongs in a case, digest, or ticket, not a page.

### Decision and durable workflow engines

Treat a DMN engine, rules service, or durable workflow platform as two adapters:

1. a pure decision evaluation keyed by `decision_definition_digest + input_digest`; and
2. a stateful workflow run keyed by `workflow_definition_digest + business_operation_key`.

Record engine version, deployed definition ID, effective time, inputs, matched rule, outputs, run ID, terminal state, timers, signals, and human-task identity. Test overlapping/no-hit rules, timezone and decimal semantics, definition promoted during an open case, duplicate start, signal-before-start, cancellation race, worker retry, timer rebuild, and rollback compatibility. A model may draft facts for an input schema; it cannot select an unpublished rule or approve its own high-impact output.

### Outcome evidence

Outcome adapters are read-only and intentionally independent of the notification or task system. Version the outcome definition, authoritative source, eligibility, measurement window, lag allowance, correction window, censoring rule, and attribution status. A closed ticket can prove only a workflow disposition unless closure is the declared outcome.

On backfill or source correction, append a new outcome observation and supersession link. Never rewrite the episode used by an earlier evaluation. Treat delayed, missing, confounded, or intervention-contaminated evidence as `indeterminate` rather than success.

## End-to-end example: regional revenue watch

1. Airflow emits a readiness event for the daily revenue asset. The intake adapter deduplicates by producer/source/event ID and records the logical interval; it does not equate the event with fresh data.
2. The controller loads `watch revenue-drop-emea@17`, `metric net-revenue@12`, semantic manifest digest, detector `material-drop@5`, baseline `weekday-region@9`, decision table `revenue-watch@6`, and adapter releases from the behavior bundle.
3. The semantic adapter resolves only approved dimensions. A warehouse dry run is within the 5 GB budget, then the query reaches terminal completion with one fully consumed page, event-time watermark, job ID, bytes processed/billed, and result digest.
4. The quality adapter confirms the expected EMEA population, current currency table, no critical assertion failures, and lineage terminal events. The application records an `accepted` data-gate decision.
5. Deterministic persistence, materiality, hysteresis, and baseline logic produce `signal sig_01K`. The same inputs always reproduce the signal.
6. A bounded model may choose up to three approved aggregate breakdowns and draft cited hypotheses. It cannot issue arbitrary SQL, select a recipient, change the threshold, or call an effect adapter.
7. The deterministic decision table opens one case and routes it to the commercial duty role. A Jira intent is committed before the call. If the response is lost, reconciliation searches the qualified operation marker before any retry.
8. An authorized operator records `investigate`; an Executive Operations system may later consume the finalized status but does not own this watch or invent a strategic decision.
9. The outcome adapter checks the approved revenue-recovery definition after its lag window. It records observed evidence and `attribution: not_established`; it does not credit the alert for all subsequent change.

## Adapter conformance and failure matrix

| Test | Required evidence | Reject or restrict when |
|---|---|---|
| Rights discovery under service principal | Positive and negative resource/field/dimension tests | Admin-only test passed but production identity is unproven |
| Identity stability | Edit/copy/move/replay cases and collision test | Provider identity changes invisibly or collides across tenant |
| Freshness and finality | Event time, watermark, cache, completion and all pages/chunks | Adapter only reports request time or HTTP status |
| Query-cost enforcement | Preflight, hard limit, final measured cost, quota behavior | Expensive query can escape controller budget |
| Timeout after remote commit | Durable intent plus lookup finds exactly one effect | No idempotency or reconciliation path for material effect |
| Duplicate/reordered delivery | One legal transition and preserved audit | Duplicate creates extra case/effect or order corrupts state |
| Schema/version drift | Fail-closed incompatibility and actionable diagnostic | Unknown field/value is silently ignored in authority path |
| Backfill/revision | New observation/outcome with supersession, no history rewrite | Old alert/result is mutated without lineage |
| Rate limit/outage | Bounded retry, backpressure, deadline-aware degradation | Retry storm or silent drop |
| Tenant/region boundary | Negative cross-scope tests and independent credentials | Provider search/list can escape structural scope |
| Deletion/retention | Propagation proof across caches, artifacts and provider | Required deletion or legal hold cannot be honored |
| Upgrade/rollback | Old/new readers, in-flight effects, downgrade rehearsal | Rollback would duplicate effects or reinterpret history |

## Qualification exercises and exit gates

### Exercise 1 — Read-only baseline

Choose one governed daily metric. Produce a dossier, rights matrix, golden query fixtures, cost preflight, freshness/quality decision, full-page receipt, and deterministic detector replay.

**Exit gate:** the same pinned input reproduces the observation and signal; stale, partial, no-data, zero, expensive, unauthorized, and paginated cases are distinct.

### Exercise 2 — One ambiguous effect

Create a synthetic ITSM destination. Inject a connection drop immediately after remote creation, then reconcile by the operation marker.

**Exit gate:** exactly one remote work item exists; the ledger passes through `UNKNOWN_OUTCOME` without blind retry; an unsupported lookup blocks production promotion.

### Exercise 3 — Drift and backfill

Change a semantic definition, remove a destination field, delay a lineage terminal event, and revise yesterday's source data.

**Exit gate:** incompatible watches suspend or shadow, effect validation fails before credential issuance, and revisions append superseding evidence without rewriting history.

### Exercise 4 — Recovery load

Pause the scheduler for one decision window, then release a backlog with one noisy tenant and one rate-limited provider.

**Exit gate:** priority and fairness policies bound query/effect load, expired alerts do not flood owners, reconciliation has reserved capacity, and recovery time is measured.

### Exercise 5 — Provider release

Run current and candidate adapter releases offline, in live-read shadow, then on a low-risk canary cohort. Force rollback with requests and effects in flight.

**Exit gate:** behavior-bundle diff explains every observation/effect change; rollback preserves identity, unknown outcomes, and audit; refresh date and limitations are updated.

## Promotion, demotion, and refresh

Approve an adapter only when the dossier, threat model, data/retention review, conformance report, fault report, runbook, SLO, dashboards, reconciliation procedure, capacity envelope, and rollback rehearsal exist. Record restrictions as machine-enforced policy.

Immediately demote or suspend an affected capability when:

- provider schema, API lifecycle, identity, permission, consistency, or idempotency behavior changes;
- the deployed edition, region, tenant setting, client, authentication mode, or contract digest changes;
- cost/freshness/latency moves outside the qualified envelope;
- reconciliation can no longer prove one effect;
- a security, privacy, residency, retention, deletion, or terms constraint changes;
- controlled failure mining finds a new invariant violation.

Requalification is not necessarily a full rewrite. Rerun the smallest test slice that proves the changed boundary, then the end-to-end shadow/canary gates that could expose interaction effects.

## Current primary references and limitations

- [MetricFlow repository and releases](https://github.com/dbt-labs/metricflow) — semantics compiler and current release/compatibility evidence; hosted dbt behavior and plans still require tenant validation.
- [BigQuery `jobs.query`](https://cloud.google.com/bigquery/docs/reference/rest/v2/jobs/query) and [running queries](https://cloud.google.com/bigquery/docs/running-queries) — completion, pagination, request ID, dry run and cost fields; estimates are not universal guarantees.
- [Snowflake query history](https://docs.snowflake.com/en/sql-reference/functions/query_history) — query identity, tag and status; visibility/retention must be tested under the deployed role.
- [Databricks SQL Statement Execution API tutorial](https://docs.databricks.com/aws/en/dev-tools/sql-execution-tutorial) — API 2.0 async identity and chunks; cloud/region/client details remain deployment-specific.
- [OpenLineage run cycle](https://openlineage.io/docs/spec/run-cycle/) and [API](https://openlineage.io/apidocs/openapi/) — versioned run-event model; emitted lineage can still be incomplete or late.
- [CloudEvents specification](https://github.com/cloudevents/spec) — core `1.0.2` at research date; broker delivery guarantees remain separate.
- [Airflow asset-aware scheduling](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/asset-scheduling.html) and [backfill](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/backfill.html) — `3.3.x` docs at research date; provider packages and deployed version must be pinned separately.
- [Jira Cloud REST API v3 issues](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/) — rights, create metadata, creation and changelog mechanics; site configuration/custom fields are local.
- [Slack `chat.postMessage`](https://api.slack.com/methods/chat.postMessage) — remote message ID, membership and rate behavior; no dedup guarantee is assumed.
- [Microsoft Graph `sendMail`](https://learn.microsoft.com/en-us/graph/api/user-sendmail?view=graph-rest-1.0) — asynchronous acceptance semantics; actual delivery/trace capabilities depend on Exchange deployment.
- [PagerDuty event management](https://support.pagerduty.com/main/docs/event-management) — Events API v2 dedup-key semantics; orchestration, plan and REST lookup require local qualification.

These references were reviewed on 2026-08-31. They do not establish availability for a particular plan, tenant, cloud, region, or future release. The conformance report—not this product matrix—is the release evidence.

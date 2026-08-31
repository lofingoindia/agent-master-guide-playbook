# Integration Qualification and Worked Flows

**Research date:** 2026-08-31  
**Status:** Production implementation and exercise guide  
**Core rule:** A named product is not a qualified capability; qualify one deployed identity, API/version, object model, limit, failure mode and reconciliation path at a time

This guide closes the path between the vendor-neutral contracts and a deployable system. Use it to qualify real adapters, then exercise the complete question → evidence → finding → decision → correction chain.

## Capability qualification contract

Before an integration appears in a model tool catalog, create an expiring manifest and pass it against the target tenant, edition, region and identity mode.

```yaml
capability:
  id: warehouse.bigquery.execute_read.v2
  product: Google BigQuery
  deployment: {project: analytics-prod, location: EU, billing_model: capacity}
  qualified_on: 2026-08-31
  expires_at: 2026-11-30
  owners: [analytics-platform, data-security]
  api_sdk: {surface: jobs.query/jobs.get/jobs.cancel, client_version: pinned}
  caller_identity: delegated-workload-capability
  supported_objects: [table, authorized_view, table_snapshot]
  snapshot_identity: "project.dataset.table plus snapshot/time-travel timestamp"
  allowed_operations: [dry_run, query_read, get_status, cancel]
  denied_operations: [dml, ddl, export, remote_function, unrestricted_metadata]
  limits:
    timeout_ms: 60000
    maximum_bytes_billed: 50000000000
    maximum_rows: 100000
    concurrency: 4
  receipts: [job_id, location, query_digest, referenced_objects, bytes_processed, bytes_billed, slot_ms]
  cancellation: "jobs.cancel then poll terminal state"
  reconciliation: "jobs.get by project/location/job_id"
  policy_tests: [row, column, purpose, denied-object, revocation, cache]
  known_gaps: [dry_run_is_estimate, limit_does_not_bound_scan, time_travel_retention]
```

Qualification must test:

1. exact resource/object IDs, object reincarnation and API/schema versions;
2. delegated/end-user versus service identity and row/column/object/purpose behavior;
3. pagination, partial/obfuscated results, unknown fields/enums and incompatible revisions;
4. estimate, execution, result-size, timeout, rate, concurrency and cost ceilings;
5. cancellation propagation, timeout-after-completion and native-job reconciliation;
6. audit/query tags without sensitive text;
7. snapshot/version retention, correction history and replay expiration;
8. negative actions and objects, not only the happy read path;
9. regional/data-residency, private-network, tenant and capacity settings;
10. provider outage, credential revocation, schema drift and adapter rollback.

## Qualified warehouse and lakehouse surfaces

The table is a research baseline, not a deployment certification.

| Surface | Use through | Identity/version evidence | Limits, cancel and receipts | Deployment-specific caveat |
|---|---|---|---|---|
| BigQuery | Jobs API with dry run, labels, `maximumBytesBilled`, timeout and read-only IAM | project/dataset/table plus creation/schema identity; `FOR SYSTEM_TIME AS OF` or named table snapshot; job ID plus location | dry-run estimate; bytes/slots/rows; `jobs.cancel` then `jobs.get` reconciliation | `LIMIT` need not reduce scan cost; time travel is dataset-configured two-to-seven days; row policies can hide some job statistics; cache/revocation behavior needs testing |
| Snowflake | scoped session/role/warehouse; query tag; semantic or reviewed SQL; query history/cancel functions | account/database/schema/object plus semantic-view revision; `AT`/`BEFORE` timestamp/statement within retention; query UUID | statement/queued timeout, warehouse/resource monitor; `SYSTEM$CANCEL_QUERY`; query history and result receipts | `BEFORE(STATEMENT)` is relative to completion and can include concurrent commits; persisted-result reuse and Time Travel depend on retention/edition/settings; query history surfaces require privileges |
| Databricks SQL/Delta/Unity Catalog | SQL Statement/warehouse adapter with end-user data authorization, Unity Catalog and query tags | metastore/catalog/schema/table plus Delta/Iceberg version/snapshot, metric-view YAML/spec/runtime and warehouse IDs | warehouse/query timeout/cancel/status; bytes/rows/runtime and system-table receipts | table history is not backup; `VACUUM`/retention can remove time-travel files; feature/syntax support depends on cloud/runtime/spec; lineage events are incomplete when not inferable |
| Apache Iceberg on Spark | qualified catalog plus Spark SQL/DataFrame adapter | catalog namespace/table UUID where exposed, snapshot ID, schema/spec version, branch/tag reference and retention | Spark job/stage IDs, plan, executor metrics and cancellation group | branches are mutable references while tags/snapshot IDs identify a state; snapshot expiration removes replay; schema selection differs between branch/tag/snapshot queries; engine integrations vary |
| PostgreSQL | dedicated read role/transaction, parameterized SQL, `EXPLAIN` and backend cancellation | cluster/database/schema/object OIDs/incarnation, transaction snapshot where deliberately exported, schema digest | `statement_timeout`, read-only transaction, row/result cap, backend PID/query receipt | read-only still permits some operations and is not a cost/privacy boundary; exported snapshots have lifecycle constraints; extensions/functions need allow-lists |

For all engines, materialize only the validated result contract. Do not expose raw connections, arbitrary object identifiers, export/file commands, external functions or unrestricted UDFs to the model.

## Qualified semantic and BI surfaces

| Surface | Bounded use | Canonical reference to preserve | Authorization and limits | Important contradiction/limitation |
|---|---|---|---|---|
| dbt MetricFlow/Semantic Layer | resolve/compile certified metrics and dimensions; execute compiled SQL through the warehouse adapter | semantic manifest/commit, metric/entity IDs, query spec and `dbt-metricflow`/adapter version | discovery must be policy-scoped outside or within the deployed service; warehouse identity remains final | current compiler documentation says compile, not query-runner; Core/Fusion/new-style metrics and legacy versions coexist; bundle compatible packages |
| Looker/LookML | query an allow-listed model/Explore/fields; fetch safe metadata and content validation | instance, model, Explore, field IDs/names, production LookML Git commit and query slug/query body | user attributes/access grants/platform permissions; adapter caps fields, rows and runtime | API query IDs moved to string slugs for affected methods; dev and production LookML can differ; Open SQL/API behavior and cost-estimate support are dialect/deployment dependent |
| Power BI/Fabric | query an allow-listed semantic model with DAX; retrieve definition/refresh metadata; render approved content separately | tenant/workspace/semantic-model ID, TMDL/TOM or source-control digest, storage mode, refresh/partition evidence and query digest | Read+Build/RLS/OLS/workspace rules; tenant settings; request/row/value/rate limits | product says semantic model while REST retains `datasets`; item ID is not definition/data version; service principals, RLS/OLS, XMLA, capacity and impersonation have distinct limitations |
| Tableau | query an allow-listed published data source; inspect lineage/certification; publish only through separate effect | site LUID, datasource/workbook LUID, revision number/current flag, extract/refresh evidence, field IDs and query digest | site/project/content permissions; Metadata API permission mode and VizQL/REST query limits | Metadata API ID and REST LUID differ; default mode can obfuscate inaccessible assets; backfill, node limits, inheritance and unsupported custom SQL can make lineage partial; revision history must be enabled |
| Cube | resolve governed members and execute scoped semantic queries | deployment/model commit, cube/member IDs, security context/policy and pre-aggregation/freshness version | security context plus member/row/masking policies, API concurrency/result caps | pre-aggregation compatibility and `rollupOnly` change behavior; identity propagation and cache isolation are deployment responsibilities |

Never normalize these products to a vague `ask_bi(question)` tool. Expose a shared typed intent—metric, dimensions, filters, time, limit—only where the adapter can prove semantic fidelity. Otherwise expose product-specific capabilities and preserve their native warnings.

## Qualified analysis, notebook, statistics, lineage, and storage surfaces

| Surface | Approved role | Receipt/version | Stop condition |
|---|---|---|---|
| Jupyter/nbclient/Papermill | orchestrate a pinned notebook in a fresh isolated runtime | source and executed notebook digests, cell order, kernel/image/lock, inputs/outputs, per-cell status | interactive prompt, hidden state, cell error/timeout, undeclared file/network/output |
| DuckDB | local SQL over read-only bounded Arrow/Parquet | explicit connection config, version/extensions, plan/SQL, inputs, memory/threads/temp settings | unbounded scan/output, shared global connection, unapproved extension, OOM or unordered result contract |
| Polars | lazy typed transforms with optional streaming | version, schema, logical/physical plan, execution engine and input/output digests | schema uncertainty, in-memory fallback beyond budget, implicit row-order dependency |
| pandas | compatibility-oriented in-memory transforms | pandas/NumPy/Arrow versions, dtypes/index/null/sort/timezone options and code digest | extract exceeds memory budget, ambiguous dtype/null/groupby behavior, unqualified major-version migration |
| Spark | distributed governed transformation already native to the estate | Spark/runtime/JVM/config, catalog snapshot, logical/physical/adaptive plan, job/stage IDs | arbitrary cluster credentials, cost/skew/shuffle beyond budget, result not reduced before sandbox |
| SciPy/statsmodels | execute a predeclared statistical-test contract | library/function, all options/default overrides, input/test/family digests, seed and numeric output | assumptions/estimand unresolved, unexpected missingness, multiplicity family absent, library output lacks required effect/interval |
| scikit-learn | bounded predictive pipeline and held-out evaluation | split IDs, pipeline/estimator, feature schema, fitted artifact, library, seed and evaluation digest | preprocessing fitted before split, test reused for selection, deployment population not represented |
| OpenLineage/catalog lineage | emit/consume provenance observations and impact hints | namespace/name/run UUID, event/facet schema/producer versions, dataset version facets and ingestion watermark | source says partial/inferred/unsupported; gap or permission filtering; never treat absent edge as proof |
| S3/GCS/Azure Blob | content-addressed/versioned artifact and manifest store | bucket/key plus version ID or generation, content checksum/digest, metadata generation/ETag and retention policy | unconditional overwrite, checksum mismatch, versioning/retention disabled contrary to policy |
| Workflow engine/database worker | timers, leases, queues, retries and durable stage execution | workflow/run/build/schema version, event history, attempt, lease/fence and operation IDs | framework cannot preserve compatible histories or reconcile external jobs; framework retry is not effect idempotency |
| OpenTelemetry | bounded traces/metrics/log correlation | SDK/instrumentation and semantic-convention versions, sampling/redaction/export config | raw prompts/SQL/results or sensitive baggage; audit completeness depends on sampled telemetry; unknown cardinality |

For object writes, prefer create-if-absent/conditional requests and preserve the native version receipt. S3 `If-None-Match`, GCS generation-match, and Azure `If-None-Match`/ETag conditions have different conflict responses and retry rules; test the exact SDK operation. An ETag is not universally a content hash.

## Approved tool surface

| Tool | Input chosen by model | Context injected by controller | Required outcome |
|---|---|---|---|
| `catalog.search_authorized` | bounded terms and asset kinds | principal, tenant, purpose, catalog snapshot, field allow-list | authorized candidates plus completeness/warnings |
| `semantic.resolve` | candidate ID and required dimensions/time | semantic snapshot, policy, certification/freshness requirements | exact metric/entity/cohort contract or structured ambiguity |
| `query.compile` | typed semantic intent or reviewed SQL-gap proposal | dialect/compiler/adapter versions and bound logical object IDs | canonical query digest and expected schema |
| `query.estimate` | compiled query reference | scoped identity, snapshot, budget and query tag | native plan/estimate receipt; no execution authorization implied |
| `query.execute_read` | validated query reference and literal parameters | fresh authorization, capability, limits, idempotent submission token | native job ID, terminal/reconciled status and validated immutable extract |
| `query.cancel_or_reconcile` | operation reference | native job ID, identity and policy | terminal status or explicit `UNKNOWN`; never blind resubmit |
| `analysis.execute_pinned` | approved code/method reference | immutable extract, image/lock, no-network policy and resource limits | promoted declared outputs or typed failure |
| `artifact.put_immutable` | manifest/output references | destination, classification, retention and create condition | version/generation, checksum and durable receipt |
| `review.request` | artifact reference and rationale | reviewer-role policy, exact digest, destination and expiry | review task ID; no approval implied |
| `publish.commit_approved` | approved publication reference | current auth/policy/approval, destination capability and semantic effect key | reconciled destination version/ID or `UNKNOWN` |

The model cannot supply principal, role, connection, tenant, raw URI, credential, policy exception, budget increase, approval or destination.

## Worked flow 1 — metric discrepancy

**Question:** “Why does August net revenue show ₹82.4M in the board workbook and ₹85.1M in the growth dashboard?”

1. Create `question@v1` with decision owner, audience and exact two artifact revisions. Do not pick either number as truth.
2. Authorized discovery resolves two candidates:
   - `recognized_net_revenue@v5`: invoice recognition date, excludes tax, subtracts posted refunds and chargebacks, INR daily FX;
   - `collected_revenue@v3`: payment date, includes tax, subtracts only settled refunds, transaction FX.
3. Pin the board workbook revision, growth dashboard/query version, semantic commits, source table snapshots and FX snapshot. If one BI surface exposes no definition version, download/source-control the supported definition and digest it; otherwise label the comparison non-reproducible.
4. Compile a deterministic reconciliation table by day with base components: gross, tax, posted/settled refunds, chargebacks and FX delta. Execute read-only with an expected schema and cost budget.
5. Validate that component sums reproduce both totals. If not, stop at `UNEXPLAINED_DISCREPANCY`; do not let the model invent a residual label.
6. Model output may explain material semantic differences from the validated reconciliation. The metric owners decide which metric fits each decision and whether either artifact is mislabeled.
7. Produce `finding@v1` bound to the component table, plus `decision@v1`: for example, retain both metrics but rename the growth chart and add purpose-specific guidance.
8. Version/correct affected artifacts; never overwrite the old board pack or silently relabel a historical chart.

Exit evidence: two exact semantic/artifact versions, zero unauthorized discovery, deterministic component equality, owner disposition, correction links, cache invalidation and a regression fixture containing the near-neighbor metrics.

## Worked flow 2 — cohort analysis

**Question:** “Compare 30-day retained paid accounts by acquisition channel for January through June cohorts.”

The workflow must clarify:

- cohort entry: first paid activation, not first signup or first invoice;
- entity: account, with an explicit merge/split identity policy;
- January–June boundaries and business time zone;
- retention event and whether a paused account counts;
- 30 complete 24-hour periods versus calendar-day convention;
- observation cutoff and late event allowance;
- channel attribution rule and missing/multi-touch handling;
- exclusions and privacy threshold.

Create `cohort_rule@v3`, pin source snapshots and materialize membership once. Compute eligible, matured, retained and censored counts—not only rates. June is excluded or visibly censored until every member has a mature outcome. Aggregate numerator/denominator by cohort and channel; never average precomputed subgroup rates.

The model may propose likely descriptive decompositions, but deterministic SQL validates membership and values. A claim such as “paid-search retention fell” is descriptive unless a credible design supports causation. A chart shows rates, denominators, uncertainty where inferential, missing attribution and incomplete cohorts.

Failure injection: a late backfill changes 127 activation timestamps and reassigns 19 accounts across month boundaries. The correction creates `source_snapshot@v2`, `cohort@v4`, a new result/finding/artifact, and invalidates the prior approval. Both versions remain linked and explain the change.

Exit evidence: sampled membership reviewed against source events, maturity/censoring tests, ratio-of-sums invariant, channel/timezone/privacy slices, old/new cohort diff, replay manifest and named retention-analysis reviewer.

## Worked flow 3 — experiment readout

**Question:** “Did the checkout treatment improve paid conversion?”

1. Resolve the frozen experiment plan: assignment unit/session, clustering/user, eligible US web sessions, intention-to-treat, `paid_conversion_24h@v3`, two-sided primary hypothesis, 95% interval and three-outcome Holm family.
2. Pin assignment and outcome snapshots after the 24-hour maturity plus late-data buffer. Verify exposure/eligibility instrumentation and that metric semantics did not change mid-experiment.
3. Deterministic checks run before the effect: sample-ratio mismatch, duplicates, assignment balance, outcome maturity, missingness by arm, cluster sizes and concurrent experiments.
4. A validated illustrative extract contains 1,950/20,000 control and 2,125/20,120 treatment conversions. The deterministic unclustered risk-difference calculation is 0.81 percentage points with an approximate 95% interval of 0.22 to 1.40 points; the released result must use the predeclared cluster-aware implementation and Holm-adjusted family output.
5. The model may explain magnitude, interval, diagnostics and business threshold. It may not substitute a post-hoc segment, switch to per-protocol, ignore the family, or say “caused” beyond the experiment’s identified assignment effect and assumptions.
6. A qualified statistician/experiment owner reviews the test/design; the decision owner separately decides rollout. `finding@v1` and `decision@v1` remain different records.

Failure injection: the model discovers that mobile treatment looks stronger after unblinding. The system records an exploratory `hypothesis@v2`, does not alter the primary family, and requires independent confirmation.

Exit evidence: frozen family and plan digests, exact cohort/source/test/library versions, deterministic recomputation, multiplicity and leakage checks, design-owner approval, decision-owner disposition and claim-to-evidence validation.

## Worked flow 4 — source correction and publication amendment

**Event:** the refund pipeline reports that an August partition omitted late chargebacks, reducing certified net revenue from ₹82.4M to ₹81.7M.

1. Ingest an authenticated `source_corrected` event with old/new snapshot IDs, affected partitions/columns, quality incident and data-owner signature.
2. Freeze new publication and shared-cache reads for the affected lineage. Do not delete the old source or mutate historical manifests.
3. Traverse query/artifact lineage and classify descendants: exact dependency, possible/inferred dependency, or no dependency. Treat partial lineage as an escalation, not an empty blast radius.
4. Invalidate dependent results, tests, charts, findings, review receipts and refresh readiness. Preserve human decisions, but mark the evidence they used as superseded.
5. Rerun the deterministic query/result/statistical baselines against the corrected snapshot. Then allow model explanation only from the validated old/new diff.
6. The data/metric owner approves the technical correction; artifact/publication owners choose amend, retract or supersede. Publish a two-way-linked correction stating scope, old/new values, reason and decision impact.
7. Reconcile every destination, CDN/cache/export and scheduled refresh. Unknown destination state blocks closure.

Crash test: the correction publish succeeds but the response is lost. Resume from the compaction receipt, see the `UNKNOWN` effect, query the destination by semantic operation key, record the native version, and do not create another amendment.

Exit evidence: complete or explicitly bounded blast radius, old/new source and artifact digests, invalidated approvals, owner decisions, destination receipts, no duplicate amendment, correction latency SLO and permanent incident fixture.

## Practical exercises

Each exercise records scenario ID, release bundle, identity/policy/source versions, expected invariants, trace/audit/evidence references, observed outcome, defect owner and regression ID.

1. **No-agent challenge:** implement one fixed KPI and one experiment template without a model; establish quality, time and total-cost baselines.
2. **Near-neighbor metric:** expose two certified “conversion” metrics with different denominators; require clarification and zero unauthorized candidate leakage.
3. **Cohort boundary:** inject DST, account merge, late activation and incomplete maturity; prove membership/version changes are explicit.
4. **Join fanout:** make a one-to-many refund join double revenue; result invariants and semantic rules must block it.
5. **Multiple testing:** offer twenty post-hoc segments; retain the frozen family and label generated hypotheses exploratory.
6. **Predictive leakage:** fit imputation/feature selection before the split; the pipeline/evaluation gate must fail.
7. **BI identity drift:** change LookML/TMDL/Tableau revision without changing display name; caches and approval readiness must invalidate.
8. **Partial lineage:** return Tableau/OpenLineage completeness warnings; source correction must escalate instead of claiming no impact.
9. **Query unknown:** warehouse completes after client timeout; reconcile native job and commit one extract without another scan.
10. **Compaction unknown:** compact while publication is `UNKNOWN`; resume with identical `next_safe_action` and invariants hash.
11. **Library migration:** replay pandas 2.x versus 3.x dtype/time/copy fixtures and one statsmodels/SciPy test; block unqualified drift.
12. **Recovery load:** restore state, release a backlog and correct one metric during a provider slowdown; reserved cancellation/reconciliation capacity must hold.
13. **Poisoned knowledge:** insert instructions into table comments, trusted examples and reviewer text; authority/tool scope must not change.
14. **Deletion propagation:** delete a preference and revoke an episodic fixture/source; prompts, indexes, caches and eval eligibility must update.
15. **DR and region return:** fail over with queries and publications in flight, reconcile them, canary reads/effects, then return without duplicate output.

## Final adapter and flow acceptance

- [ ] Every adapter has a dated deployment-specific manifest, owners, expiry, negative tests and kill switch.
- [ ] Object ID, definition version, data snapshot, execution/job and artifact version are not conflated.
- [ ] Read-only identities, row/column/object/purpose controls and result disclosure pass under denied and revoked principals.
- [ ] Estimate, cost, result-size, rate, concurrency, timeout, cancellation and reconciliation are tested natively.
- [ ] Semantic/BI permission filtering, partial lineage and version/revision behavior are represented as typed states.
- [ ] Notebook/dataframe/statistical libraries are pinned with behavioral regression fixtures, not merely lockfiles.
- [ ] Object writes use conditional semantics and retain generation/version plus checksum receipts.
- [ ] The four worked flows pass deterministic baselines, model-value comparisons, owner review, correction and replay.
- [ ] Metrics, traces, logs, audits, evidence and SLOs remain separate and privacy-controlled.
- [ ] Peak, backlog, recovery, correction, DR, rollback and provider-withdrawal exercises preserve invariants.

## Primary sources

All sources below were accessed on **2026-08-31**. Product behavior still requires tenant/deployment qualification.

- [BigQuery Jobs API](https://docs.cloud.google.com/bigquery/docs/reference/rest/v2/Job)
- [BigQuery time travel](https://docs.cloud.google.com/bigquery/docs/time-travel)
- [BigQuery table snapshots](https://docs.cloud.google.com/bigquery/docs/table-snapshots-intro)
- [Snowflake `AT` and `BEFORE`](https://docs.snowflake.com/en/sql-reference/constructs/at-before)
- [Snowflake query history](https://docs.snowflake.com/en/sql-reference/functions/query_history)
- [Snowflake query cancellation](https://docs.snowflake.com/en/sql-reference/functions/system_cancel_query)
- [Databricks table history](https://docs.databricks.com/aws/en/delta/history)
- [Databricks lineage system tables](https://docs.databricks.com/aws/en/admin/system-tables/lineage)
- [Apache Iceberg branching and tagging](https://iceberg.apache.org/docs/latest/branching/)
- [PostgreSQL 18 transaction modes](https://www.postgresql.org/docs/18/sql-set-transaction.html)
- [dbt MetricFlow metric semantics](https://github.com/dbt-labs/dbt-core/blob/main/crates/dbt-metricflow/docs/metric-semantics.md)
- [Looker query-slug API migration](https://docs.cloud.google.com/looker/docs/api-update-query-slug)
- [Power BI Execute Queries API](https://learn.microsoft.com/en-us/rest/api/power-bi/datasets/execute-dax-queries-in-group)
- [Tabular Model Definition Language](https://learn.microsoft.com/en-us/analysis-services/tmdl/tmdl-overview)
- [Tableau Metadata API permissions](https://help.tableau.com/current/api/metadata_api/en-us/docs/meta_api_permissions.html)
- [Tableau Metadata API errors](https://help.tableau.com/current/api/metadata_api/en-us/docs/meta_api_errors.html)
- [Tableau revisions API](https://help.tableau.com/current/api/rest_api/en-us/REST/rest_api_ref_revisions.htm)
- [DuckDB Python API](https://duckdb.org/docs/stable/clients/python/overview)
- [Polars lazy API](https://docs.pola.rs/user-guide/concepts/lazy-api/)
- [pandas 3.0 changes](https://pandas.pydata.org/docs/whatsnew/v3.0.0.html)
- [Spark SQL performance tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html)
- [SciPy independent t-test](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.ttest_ind.html)
- [statsmodels 0.14.6 release](https://www.statsmodels.org/stable/release/version0.14.6.html)
- [scikit-learn data-leakage guidance](https://scikit-learn.org/stable/common_pitfalls.html#data-leakage)
- [OpenLineage object model](https://openlineage.io/docs/spec/object-model/)
- [OpenLineage integration caveats](https://openlineage.io/docs/integrations/about/)
- [Amazon S3 conditional writes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/conditional-writes.html)
- [Google Cloud Storage request preconditions](https://docs.cloud.google.com/storage/docs/request-preconditions)
- [Azure Blob conditional headers](https://learn.microsoft.com/en-us/rest/api/storageservices/specifying-conditional-headers-for-blob-service-operations)
- [OpenTelemetry signals](https://opentelemetry.io/docs/concepts/signals/)
- [OpenTelemetry database semantic conventions](https://opentelemetry.io/docs/specs/semconv/db/)

Return to the [overview](README.md), use the [Stage 0–6 roadmap](10-cost-performance-deployment-and-roadmap.md#roadmap), or inspect the [research packet](../../research/packets/analytics-agent-blueprint.md).

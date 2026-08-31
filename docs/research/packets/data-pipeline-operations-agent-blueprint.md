# Research Packet: Data Pipeline Operations Agent Blueprint

**Research date:** 2026-08-31
**Primary sources last accessed:** 2026-08-31; volatile surfaces must be rechecked before implementation
**Artifact:** [Data Pipeline Operations Agent](../../agents/data-pipeline-operations-agent/README.md)
**Scope:** batch, streaming, CDC, contracts, lineage, quality, backfill, recovery, security, evaluation, and production operations
**Source preference:** official specifications, documentation, repositories, standards, and primary engineering reports

## 1. Research objective

Determine the smallest reliable production architecture for an AI-assisted Data Pipeline Operations Agent, including:

- the category seam with database operations, analytics, and BI;
- native batch, streaming, and CDC semantics;
- contracts, compatibility, lineage, quality, and governance;
- safe replay, backfill, effect execution, and reconciliation;
- identity, tenancy, secrets, prompt injection, privacy, and memory;
- observability, SLOs, cost, HA/DR, incident response, and deployment;
- realistic evaluation and failure-injection gates.

The research question was not “which agent framework should run it?” It was “which system invariants make operations safe across existing data platforms?”

## 2. Method

### Repository analysis

The adjacent Database Operations Agent and Analytics Agent were inspected first. The repository registry defines the Data Pipeline Operations Agent as the owner of ingestion/transform DAGs, schema evolution, backfills, lineage, data-quality quarantine, and pipeline recovery. That boundary was preserved:

- database engine/schema mutation and failover remain Database Operations;
- bounded analysis remains Analytics;
- persistent business metric monitoring and decision follow-through remain BI.

### Research paths

Research proceeded across:

1. batch orchestrators and durable workflow systems;
2. stream-time, checkpoint, delivery, and state semantics;
3. Kafka and CDC delivery/recovery;
4. schema formats, registries, contracts, and table formats;
5. lineage, catalogs, data quality, and privacy;
6. effect receipts and idempotent warehouse jobs;
7. agent, MCP/tool, identity, and secrets security;
8. SRE alerting, data-pipeline correctness, HA/DR, and incident evidence;
9. current production engineering reports.

The 2026-08-31 refinement also compared exact product/version surfaces for Airflow, Dagster, Prefect, Kafka/Connect, Debezium, Flink, Spark, dbt, Confluent Schema Registry, BigQuery, Snowflake, Iceberg, Delta Lake, Hudi, OpenLineage, GX, Soda, Deequ, Vault/Secret Manager, OpenTelemetry, and SLSA. Search results were used only to locate primary documentation; claims below come from official specifications, documentation, repositories, or release notes.

Important claims were cross-checked across multiple system boundaries. For example, an exactly-once claim was checked against the stream runtime, messaging layer, CDC connector, and external sink behavior rather than accepted from one product page.

### Selection rules

- Prefer stable/versioned official documentation over tutorials.
- Use master/nightly docs only to locate a topic; avoid treating unreleased behavior as deployed.
- Treat vendor defaults as deployment-specific and record them in an adapter manifest.
- Treat engineering case studies as evidence of operational patterns, not universal guarantees.
- Separate specified behavior from an implementation's observed capability.
- Retain contradictions and limitations instead of collapsing them into one claim.

## 3. Research conclusion

The best practical architecture is a **pipeline safety controller**, not an autonomous general-purpose agent:

~~~mermaid
flowchart LR
    I[Identity and request] --> C[Durable controller]
    C --> R[Read-only typed evidence]
    C --> M[Model diagnosis/proposal]
    C --> P[Deterministic policy and approval]
    P --> E[Isolated typed effect]
    E --> L[Effect receipt]
    L --> V[Independent outcome verification]
    V --> C
~~~

This conclusion followed consistently from the evidence:

- workflow and stream platforms already own execution semantics;
- exactly-once guarantees are boundary-specific;
- remote APIs create ambiguous-effect cases;
- contracts and lineage are incomplete without outcome verification;
- external data and metadata can inject agent instructions;
- durable recovery requires explicit state and receipts;
- a model is useful for evidence synthesis but unsuitable as the authorization boundary.

## 4. Evidence synthesis

### 4.1 Orchestration and historical work

Airflow distinguishes logical data intervals, catchup, scheduler runs, and explicit backfills. Backfill supports reprocessing behavior, maximum active runs, ordering, and dry-run inspection. Airflow 3.3 also makes the DAG bundle used for clear/rerun/backfill a real operational choice: explicit request, DAG, and global settings determine original versus latest bundle behavior, with a final default that differs between backfill and clear/rerun. These are essential planning inputs, but they do not prove a sink effect.

Dagster 1.13.20 documents partitions, backfills, asset checks, and multiple concurrency layers. Check failures do not block downstream assets unless `blocking` is configured, and partitioned asset checks are preview. Prefect 3 provides flows, tasks, deployments, transactions, and global concurrency, but its default transaction isolation permits a concurrent same-key race; serializable isolation needs an appropriate lock manager. Its concurrency path can continue on missing limits or lease-renewal failure unless strict behavior is selected. These semantics reinforce an adapter model instead of a generic “run job” interface.

**Blueprint decision:** persist logical interval/partition identity, use platform-native public APIs, expose scheduler semantics through typed adapters, and apply a separate sink verifier.

### 4.2 Streaming correctness

Flink documentation states that checkpoints capture source positions and operator state, and clarifies that exactly-once state consistency does not mean each event's user code runs only once. End-to-end exactly-once additionally needs replayable sources and transactional or idempotent sinks. Kafka sink visibility and transaction timeout introduce further operational constraints. The 2.3 stable Kafka connector page stated that no connector artifact was yet available for 2.3, demonstrating why runtime and connector versions must be qualified separately.

Beam documents watermarks as progress estimates and makes allowed lateness, triggers, and accumulation behavior explicit. Spark Structured Streaming 4.1.2 documents watermark-driven state cleanup and late-data constraints; its end-to-end claim still depends on the exact source, sink, checkpoint identity, and query compatibility.

**Blueprint decision:** require a guarantee matrix per boundary; record watermark, lateness, trigger, correction, checkpoint, and sink semantics; never diagnose a stream only from job state.

### 4.3 CDC

Debezium documents at-least-once as the default and supports Kafka Connect exactly-once modes only for supported connector/deployment combinations. Kafka Connect 4.1.2 defaults `exactly.once.source.support` to disabled and uses a staged `preparing`/`enabled` rollout for existing clusters; source connectors must declare support, and external sinks remain outside that transaction. Offset and schema-history state are both durable dependencies. Incremental snapshots run alongside streaming and use chunk/watermark mechanisms; duplicates remain a sink concern. The PostgreSQL connector documentation says schema changes are unsupported during an incremental snapshot and labels `offset.mismatch.strategy` as technology preview. The outbox event router exposes a stable event ID and aggregate key useful for deduplication and ordering.

**Blueprint decision:** store connector position, schema history, snapshot request/chunks, source retention headroom, and sink frontier; require duplicate-safe sinks and a high-risk gate for resnapshot/reset.

### 4.4 Contracts and schema compatibility

ODCS 3.1.0 provides a vendor-neutral contract structure for schema, quality, SLA, servers, and team/roles. ODPS adds product input/output port framing. The ODCS repository warns that companion schema/tooling can contain issues, so the prose standard is the semantic authority.

Avro resolves writer and reader schemas. Protobuf compatibility depends on stable field numbers and reserving removed identifiers. Confluent Schema Registry supports backward, forward, full, and transitive modes; the observed default is non-transitive backward. Rich schema rules require Platform 7.4+ Enterprise or Cloud Advanced governance and are not executed by every Connect/Flink/ksqlDB/client path. dbt model contracts validate output names/types before materialization but exclude several resource/materialization types and do not replace data tests. Constraint enforcement and microbatch replacement vary by adapter; current microbatch time inputs are UTC and upstreams without `event_time` can be scanned in full per batch.

**Blueprint decision:** separate syntactic, structural, serialization, semantic, quality, SLO, and governance compatibility; test with real encoded fixtures and affected consumers.

### 4.5 Table formats and publication

Iceberg spec versions 1–3 are complete and use stable field IDs, schema/spec IDs, table UUIDs, snapshots, partition evolution, and optimistic concurrency; identifier fields do not enforce uniqueness. Delta protocol/table features gate compatible readers and writers; Delta Change Data Feed starts only after enablement, follows retention/VACUUM, and has version-specific restrictions around non-additive schema changes. Hudi 1.2.0 recommends backward-compatible write evolution; schema-on-read is experimental and irreversible after incompatible evolution is accepted, and some changes can write successfully while reads fail.

**Blueprint decision:** pin physical snapshot/protocol/schema identifiers and prefer staged output plus atomic snapshot/pointer publication.

### 4.6 Lineage and quality

OpenLineage 1.52.0 models jobs, runs, datasets, events, and versioned facets. A facet update with the same name replaces the prior facet for that entity, so a materialized graph is not an immutable assertion log. Catalog systems such as Google Cloud Data Lineage ingest or expose process/run/event graphs, but access and granularity remain separate concerns. Missing instrumentation and identifier drift make negative lineage evidence weak.

GX Core 1.21.0, Deequ 2.0.x, and Soda v3 show complementary validation patterns: declared expectations, metric histories/anomaly detection, and contract-language checks. GX's all-unexpected-row helper applies to `UnexpectedRowsExpectation` and can move sensitive rows. Soda can send results/metadata to Cloud unless an appropriate local path is selected, and its data-contract surface was beta. Deequ releases are Spark-version-specific. None alone proves source-to-sink reconciliation.

**Blueprint decision:** represent lineage coverage and freshness explicitly; use quality severity to observe/warn/quarantine/block; require effect reconciliation in addition to checks.

### 4.7 Effect receipts

BigQuery documentation illustrates a general distributed-systems problem: use a caller-selected job ID to make retries reconcilable, retrieve the job in the correct project/location after conflict or ambiguous response, and inspect `errorResult` even when state is `DONE`. Appends need particular care because a repeated request can duplicate data. Snowflake documents a related but different mechanism: resubmit SQL API work with the same `requestId` and `retry=true`; bulk `COPY` load metadata expires after 64 days and `FORCE` can duplicate historical files.

**Blueprint decision:** record intent before dispatch, query by stable effect key or target state after ambiguity, capture a remote/sink receipt, and verify the data outcome independently.

### 4.8 Agent security

OWASP agent guidance treats external content and tool results as untrusted, recommends least privilege, structured validation, separation of decision and execution, and defenses against memory poisoning. OWASP MCP guidance adds tool-schema/description poisoning and per-server credential concerns. NIST control families support least privilege, separation of duties, audit, and controlled change.

Airflow's security model warns that DAG authors can execute code and that component isolation has limits. Secrets should be available only where required. AWS Glue warns that workflow run properties may be logged and should not contain plaintext credentials. Vault supports dynamic secrets and response wrapping, but auditing starts disabled and unavailable audit devices can make the service fail closed. Google Secret Manager recommends pinning secret versions and disabling before destructive removal. SLSA 1.2 clarifies that provenance protects only when consumers verify it against expectations.

**Blueprint decision:** separate read, model, effect, and verifier identities; treat data/metadata/tool schemas as untrusted; keep secrets and raw rows out of model/state; bind approval to a structured plan hash.

### 4.9 Observability and SRE

Google's SRE guidance for data processing emphasizes freshness and correctness from the user's perspective, including test-account data. Its SLO alerting guidance supports multi-window burn-rate alerts rather than paging on every component fault. OpenTelemetry semantic conventions 1.44.0 mark database client spans stable but document mixed old/new emission during migration; GenAI conventions remain development. Query parameters and unsanitized query text, prompts, and outputs are sensitive/high-cardinality inputs rather than safe defaults.

**Blueprint decision:** define data-product and control-plane SLOs, trace the full intent-to-outcome chain, alert on user-visible error-budget burn, and retain sensitive payloads by reference.

### 4.10 Production evidence

Meta engineering reports show practical use of deterministic analyzers, incident evidence, cross-repository pipeline maps, automated data-removal systems, and explicit work on silent corruption. A 2026 report describes using AI to map tribal pipeline knowledge and recurring incident patterns. These examples support evidence synthesis and reviewed knowledge, not general autonomous mutation.

**Blueprint decision:** use AI for cross-system context and hypotheses, keep operational analyzers and effect controls deterministic, and curate incident patterns with provenance/review/expiry.

## 5. Important disagreements and caveats

| Topic | Sources may appear to imply | Resolved interpretation |
|---|---|---|
| Exactly once | A platform makes the pipeline exactly once | State, broker, sink, and external effects have separate boundaries |
| Workflow transactions | A framework transaction is an ACID transaction across tools | It coordinates framework behavior; remote systems require their own idempotency/reconciliation |
| Backfill success | Scheduler run success proves repaired data | Verify sink commit, contract, quality, lineage, and downstream outcome |
| Registry compatibility | A passing registry check means safe change | It proves a configured serialization relation, not semantics or consumer behavior |
| No lineage descendants | No consumer is affected | Graph coverage may be incomplete or stale |
| Checkpoint restore | Restore guarantees no duplicate processing | Restore protects consistent state; replay and sink effects still matter |
| CDC exactly once | Enabling connector EOS eliminates duplicates | Support and boundaries vary; sink and snapshot overlap still need validation |
| Table change feed | Change feed is a permanent audit history | Availability starts at enablement and is constrained by retention |
| Human approval | A conversational “yes” is enough | Approval must bind identity, exact plan hash, scope, and expiry |
| Memory | More incident memory improves operations | Unreviewed/stale memory creates poisoning and authority risks |
| Airflow historical execution | A backfill automatically uses the intended historical DAG code | Airflow 3.3 bundle selection depends on request/DAG/global settings and a differing final default; record the actual bundle |
| Dagster checks | An asset check always blocks bad downstream materialization | Blocking must be configured, and partitioned asset checks were preview in 1.13.20 |
| Prefect transaction | Same key means concurrent work runs once | Default `READ_COMMITTED` can race; serializable isolation needs a real shared lock manager, and remote effects remain separate |
| Kafka Connect EOS | Enabling a worker flag makes every connector exactly once | It defaults off, needs cluster rollout, depends on connector-declared support, and does not transact an external sink |
| Flink stable docs | Stable runtime docs imply a matching connector artifact exists | The observed 2.3 Kafka connector page reported no 2.3 connector; resolve runtime, connector, and Kafka client independently |
| Confluent data contracts | Schema rules run across all products and clients | Package, version, connector, Flink/ksqlDB, referenced-schema, and language-client limitations apply |
| dbt contract | Contract enforcement covers all dbt resources and warehouse constraints | Resource/materialization coverage and actual constraint enforcement vary; content tests remain separate |
| Warehouse retry | One universal idempotency header covers warehouses | BigQuery fixed job IDs and Snowflake request IDs/COPY history have different scopes, retention, and terminal checks |
| Quality result | `pass/fail` is portable across quality engines | Engines distinguish error/warn/fail differently and may retrieve or export unexpected rows; normalize without collapsing unknown/error |
| Telemetry standard | Adopting OpenTelemetry fixes schema and privacy | Convention stability/migration differs by signal and captured SQL/parameters/prompts can leak data |

## 6. Current baselines observed

All entries were accessed 2026-08-31. These are research anchors, not deployment assumptions:

| Surface | Observed version/status | Qualification retained |
|---|---|---|
| Apache Airflow | Stable 3.3.1 API/release docs; backfill page rendered 3.3.0 during part of research | `/api/v2` is public; stable URLs move; backfill bundle-version behavior must be captured |
| Dagster | Latest 1.13.20 | Partitioned asset checks preview; check blocking and concurrency configuration are explicit |
| Prefect | v3 docs; tag-limit page referenced 3.4.19 implementation | Default transaction/concurrency behavior can be too permissive for a hard production boundary |
| Apache Kafka | 4.1.2 quickstart/current 4.1 docs | Connect source EOS default disabled; connector and external-sink boundaries remain separate |
| Debezium | 3.5 EOS docs plus stable source-specific connectors | Support is connector/deployment-specific; PostgreSQL snapshot/schema and preview offset behavior retained |
| Apache Flink | Stable docs exposed 2.3-era content | Stable Kafka page reported no 2.3 connector artifact; no deployment claim made from docs alone |
| Apache Spark | Latest Structured Streaming docs rendered 4.1.2 | Source, sink, checkpoint, and query compatibility still need local testing |
| dbt | Docs v2/Core 1.12 and 1.11 selectors | Engine and adapter releases differ; observed docs updated 2026-08-27 |
| Confluent Schema Registry | Current Platform/Cloud docs | Default `BACKWARD` is non-transitive; rich rules have package/version/client limitations |
| ODCS | 3.1.0, released 2025-12-08 | Released standard used; proposed RFCs and tooling drift excluded from guarantees |
| Apache Avro | 1.12.0 specification | Writer/reader resolution used; no registry behavior inferred |
| Apache Iceberg | Spec v1-v3 complete/adopted; page also contains newer-version material | Reader/writer/catalog support must be pinned; highest visible spec text is not assumed deployed |
| Delta Lake | Current documentation | CDF enablement/retention and protocol/feature restrictions retained |
| Apache Hudi | 1.2.0 docs | Schema-on-read experimental and irreversible after incompatible use |
| OpenLineage | 1.52.0, released 2026-07-23 | Producer/facet schema versions and ingestion coverage still determine usefulness |
| GX Core | 1.21.0 | Unexpected-row retrieval capability is expectation-specific and privacy-sensitive |
| Soda | v3 docs; data-contract feature documented as public beta | Local/cloud result flow and language surface require a deployment probe |
| Deequ | 2.0.19 release visible; artifacts remain Spark-specific | Pin the Spark-qualified artifact and metrics repository behavior |
| OpenTelemetry semantic conventions | 1.44.0 | Database spans stable with migration caveats; GenAI attributes development |
| SLSA | 1.2 Approved | Provenance must be verified against consumer expectations |

Every adapter deployment must record exact upstream version, API surface, enabled features, configuration, and successful capability probes.

## 7. Alternatives considered

### Put all logic in the orchestrator

**Strength:** fewer components.
**Rejected as the complete design because:** the orchestrator does not own model trust, cross-system effect reconciliation, contract/governance policy, or independent verification.

### General autonomous agent with SQL/shell

**Strength:** fast prototype and broad reach.
**Rejected for production because:** authority is too broad, actions are hard to type/reconcile, and data/metadata injection can influence effects.

### Durable workflow engine as the whole solution

**Strength:** retries and durable state.
**Use selectively:** it can implement the controller loop.
**Limitation:** durability does not define pipeline semantics, policy, idempotency, or data correctness.

### Data observability product only

**Strength:** detection, quality, lineage, incident context.
**Use as a major evidence source.**
**Limitation:** recovery effects and cross-platform receipts may remain outside its scope.

### Vendor-specific copilot

**Strength:** deep native integration.
**Use when the estate is concentrated and controls are sufficient.**
**Limitation:** cross-system identity, semantics, receipts, and evidence can fragment.

### Deterministic automation only

**Strength:** predictable and testable.
**Preferred for known paths.**
**Limitation:** heterogeneous evidence and novel incidents still need human or model-assisted synthesis.

## 8. Primary source ledger

### Orchestration and workflows

| Source | Evidence used |
|---|---|
| [Airflow DAG runs](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dag-run.html) | Data intervals, logical dates, catchup |
| [Airflow backfill](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/backfill.html) | Reprocessing, concurrency, ordering, dry run |
| [Airflow REST API](https://airflow.apache.org/docs/apache-airflow/stable/stable-rest-api-ref.html) | Public integration surface |
| [Airflow configuration reference](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#rerun-with-latest-version) | Original/latest DAG bundle behavior for clear, rerun, and backfill |
| [Airflow production deployment](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/production-deployment.html) | External DB, logs, heartbeats, upgrades |
| [Airflow security model](https://airflow.apache.org/docs/apache-airflow/stable/security/security_model.html) | DAG-author capability and isolation limits |
| [Airflow public interface](https://airflow.apache.org/docs/apache-airflow/stable/public-airflow-interface.html) | Avoid internal APIs |
| [Dagster partitions and backfills](https://docs.dagster.io/guides/build/partitions-and-backfills) | Partition/backfill model in 1.13.20 docs |
| [Dagster asset checks](https://docs.dagster.io/guides/test/asset-checks) | Blocking behavior and preview partitioned checks |
| [Dagster concurrency](https://docs.dagster.io/guides/operate/managing-concurrency) | Run, pool, executor, and tag scopes |
| [Prefect transactions](https://docs.prefect.io/v3/advanced/transactions) | Framework rollback/idempotency behavior |
| [Prefect global concurrency](https://docs.prefect.io/v3/concepts/global-concurrency-limits) | Cross-workflow limits |
| [Prefect concurrency application](https://docs.prefect.io/v3/how-to-guides/workflows/global-concurrency-limits) | Strict-mode and lease-renewal behavior |
| [Temporal activity definition](https://docs.temporal.io/activity-definition) | Retryable side-effect unit |
| [Temporal worker versioning](https://docs.temporal.io/production-deployment/worker-deployments/worker-versioning) | Compatible worker rollout |

### Streaming, messaging, and CDC

| Source | Evidence used |
|---|---|
| [Flink fault tolerance](https://nightlies.apache.org/flink/flink-docs-stable/docs/learn-flink/fault_tolerance/) | Checkpoints, state consistency, end-to-end boundary |
| [Flink watermarks](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/datastream/event-time/built_in/) | Event-time progress |
| [Flink checkpoint versus savepoint](https://nightlies.apache.org/flink/flink-docs-stable/docs/ops/state/checkpoints_vs_savepoints/) | Lifecycle/operational distinction |
| [Flink Kafka connector](https://nightlies.apache.org/flink/flink-docs-stable/docs/connectors/datastream/kafka/) | Transactions, visibility, timeout constraints |
| [Flink large-state tuning](https://nightlies.apache.org/flink/flink-docs-stable/docs/ops/state/large_state_tuning/) | State/checkpoint/backpressure operations |
| [Beam programming guide](https://beam.apache.org/documentation/programming-guide/) | Watermarks, triggers, lateness, panes |
| [Beam runner matrix](https://beam.apache.org/documentation/runners/capability-matrix/) | Runner-specific feature reality |
| [Spark Structured Streaming](https://spark.apache.org/docs/latest/streaming/apis-on-dataframes-and-datasets.html) | Watermarks, state cleanup, late data and joins |
| [Kafka design](https://kafka.apache.org/41/design/design/) | At-most/at-least/exactly-once and isolation concepts |
| [Kafka KIP-98](https://cwiki.apache.org/confluence/spaces/KAFKA/pages/66854913/KIP-98%2B-%2BExactly%2BOnce%2BDelivery%2Band%2BTransactional%2BMessaging) | Transactional messaging design |
| [Debezium EOS](https://debezium.io/documentation/reference/3.5/configuration/eos.html) | Default at-least-once and supported EOS modes |
| [Debezium PostgreSQL](https://debezium.io/documentation/reference/stable/connectors/postgresql.html) | Incremental snapshots, offsets, source retention |
| [Debezium signalling](https://debezium.io/documentation/reference/3.0/configuration/signalling.html) | Snapshot chunk/watermark control |
| [Debezium outbox router](https://debezium.io/documentation/reference/stable/transformations/outbox-event-router.html) | Stable event ID and aggregate ordering key |
| [Kafka KIP-618](https://cwiki.apache.org/confluence/spaces/KAFKA/pages/153816406/KIP-618%2BExactly-Once%2BSupport%2Bfor%2BSource%2BConnectors) | Source connector EOS design |
| [Kafka Connect worker configuration](https://kafka.apache.org/41/generated/connect_config.html) | Source EOS default, staged rollout, offsets and timeouts |

### Contracts, schemas, and tables

| Source | Evidence used |
|---|---|
| [ODCS](https://github.com/bitol-io/open-data-contract-standard/blob/main/docs/README.md) | Contract model and tooling caveat |
| [ODPS](https://github.com/bitol-io/open-data-product-standard/blob/main/docs/README.md) | Data-product ports |
| [JSON Schema 2020-12](https://json-schema.org/draft/2020-12) | JSON schema baseline |
| [Avro 1.12.0 specification](https://avro.apache.org/docs/1.12.0/specification/) | Writer/reader resolution, aliases, defaults |
| [Protobuf guide](https://protobuf.dev/programming-guides/proto3/) | Field-number evolution rules |
| [Confluent schema evolution](https://docs.confluent.io/platform/current/schema-registry/fundamentals/schema-evolution.html) | Compatibility modes and upgrade order |
| [Confluent data contracts](https://docs.confluent.io/platform/current/schema-registry/fundamentals/data-contracts.html) | Registry rules/contract features |
| [dbt model contracts](https://docs.getdbt.com/docs/mesh/govern/model-contracts) | Preflight schema and platform constraint differences |
| [dbt incremental models](https://docs.getdbt.com/docs/build/incremental-models) | Filter, unique key, strategies |
| [dbt microbatch](https://docs.getdbt.com/docs/build/incremental-microbatch) | Event-time batch decomposition |
| [dbt data tests](https://docs.getdbt.com/docs/build/data-tests) | Contracts versus observed assertions |
| [Iceberg specification](https://iceberg.apache.org/spec/) | Field IDs, snapshots, evolution, concurrency |
| [Delta versioning](https://docs.delta.io/versioning/) | Protocol and table features |
| [Delta Change Data Feed](https://docs.delta.io/delta-change-data-feed/) | Enablement and retention limitations |
| [Delta concurrency](https://docs.delta.io/concurrency-control/) | Concurrent write behavior |
| [Hudi schema evolution](https://hudi.apache.org/docs/schema_evolution/) | Compatibility guidance and feature cautions |

### Lineage, quality, governance, and effects

| Source | Evidence used |
|---|---|
| [OpenLineage specification](https://github.com/OpenLineage/OpenLineage/blob/main/spec/OpenLineage.md) | Job/run/dataset/event/facet model |
| [OpenLineage 1.52.0 release](https://openlineage.io/docs/releases/1_52_0/) | Current release anchor and integration changes |
| [OpenLineage facets](https://openlineage.io/docs/spec/facets/) | Facet replacement and custom naming |
| [Google Cloud data lineage](https://docs.cloud.google.com/dataplex/docs/about-data-lineage) | Process/run/event and custom lineage |
| [Google Cloud lineage views](https://docs.cloud.google.com/dataplex/docs/lineage-views) | Table/column graph views |
| [Great Expectations: expectations](https://docs.greatexpectations.io/docs/core/define_expectations/) | Assertion model |
| [Great Expectations: validations](https://docs.greatexpectations.io/docs/core/run_validations/) | Suites and validation flow |
| [Great Expectations: unexpected rows](https://docs.greatexpectations.io/docs/core/run_validations/retrieve_all_unexpected_rows/) | Quarantine-oriented retrieval |
| [Deequ](https://github.com/awslabs/deequ) | Metrics and quality checks |
| [Soda contract language](https://docs.soda.io/reference/contract-language-reference) | Alternative quality/contract syntax |
| [BigQuery reliability](https://docs.cloud.google.com/bigquery/docs/reliability-intro) | Fixed job IDs and append retry risk |
| [BigQuery jobs](https://docs.cloud.google.com/bigquery/docs/running-jobs) | Terminal state versus error result |
| [Snowflake SQL API resubmission](https://docs.snowflake.com/en/developer-guide/sql-api/submitting-requests) | Request ID, retry flag, and ambiguous execution |
| [Snowflake bulk loading](https://docs.snowflake.com/en/user-guide/data-load-considerations-load) | File manifest behavior, 64-day history, duplicate risk |
| [NIST Privacy Framework](https://www.nist.gov/privacy-framework) | Privacy risk management |
| [GDPR consolidated text](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32016R0679) | Legal privacy obligations; implementation requires counsel/policy |

### Security, SRE, and operations

| Source | Evidence used |
|---|---|
| [OWASP Agent Security](https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html) | Untrusted input, least privilege, decision/effect split, memory risk |
| [OWASP MCP Security](https://cheatsheetseries.owasp.org/cheatsheets/MCP_Security_Cheat_Sheet.html) | Tool poisoning and per-server credentials |
| [NIST SP 800-53 Rev. 5.1](https://csrc.nist.gov/CSRC/media/Projects/risk-management/800-53%20Downloads/800-53r5/SP_800-53_v5_1-derived-OSCAL.pdf) | Least privilege, duties, audit, change control |
| [NIST SP 800-122](https://csrc.nist.gov/pubs/sp/800/122/final) | PII protection |
| [Google SRE data processing](https://sre.google/workbook/data-processing/) | User-visible freshness/correctness SLOs |
| [Google SRE SLO alerting](https://sre.google/workbook/alerting-on-slos/) | Multi-window burn-rate alerting |
| [OpenTelemetry trace conventions](https://opentelemetry.io/docs/specs/semconv/general/trace/) | Trace model |
| [OpenTelemetry database spans](https://opentelemetry.io/docs/specs/semconv/db/database-spans/) | Database client attributes |
| [OpenTelemetry GenAI attributes](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/) | Prompt/output sensitivity |
| [OpenTelemetry semantic conventions 1.44.0](https://opentelemetry.io/docs/specs/semconv/) | Current convention release and stability |
| [OpenTelemetry database migration](https://opentelemetry.io/docs/specs/semconv/db/) | Mixed old/stable emission and opt-in |
| [AWS Glue run properties](https://docs.aws.amazon.com/glue/latest/webapi/API_PutWorkflowRunProperties.html) | Logged-property secret warning |
| [Vault response wrapping](https://developer.hashicorp.com/vault/docs/concepts/response-wrapping) | Single-use wrapping, TTL, path validation |
| [Vault audit devices](https://developer.hashicorp.com/vault/docs/audit) | Audit availability and fail-closed behavior |
| [Google Secret Manager best practices](https://docs.cloud.google.com/secret-manager/docs/best-practices) | Version pinning, rotation, disable-before-destroy |
| [SLSA 1.2 artifact verification](https://slsa.dev/spec/v1.2/verifying-artifacts) | Provenance verification against expectations |

### Primary engineering reports

| Source | Evidence used |
|---|---|
| [Meta: AI and pipeline tribal knowledge](https://engineering.fb.com/2026/04/06/developer-tools/how-meta-used-ai-to-map-tribal-knowledge-in-large-scale-data-pipelines/) | Current AI-assisted cross-repo context and incident patterns |
| [Meta: silent data corruption](https://engineering.fb.com/2021/02/23/data-infrastructure/silent-data-corruption/) | Correctness beyond process errors |
| [Meta: automated data removal](https://engineering.fb.com/2023/10/31/data-infrastructure/automating-data-removal/) | Operational deletion-system complexity |
| [Meta: AI-assisted incident response](https://engineering.fb.com/2024/06/24/data-infrastructure/leveraging-ai-for-efficient-incident-response/) | AI with operational evidence |
| [Meta: root-cause analysis platform](https://engineering.fb.com/2025/12/19/data-infrastructure/drp-metas-root-cause-analysis-platform-at-scale/) | Deterministic analyzers and RCA at scale |

## 9. Coverage map

| Required area | Blueprint location |
|---|---|
| Zero-to-production stages | [Evaluation and roadmap](../../agents/data-pipeline-operations-agent/09-evaluation-failure-injection-and-delivery-roadmap.md) |
| Architecture and durable state | [Architecture and state](../../agents/data-pipeline-operations-agent/02-reference-architecture-runtime-and-state.md) |
| Batch, stream, CDC, tools | [Runtime and adapters](../../agents/data-pipeline-operations-agent/03-batch-stream-cdc-and-tool-adapters.md) |
| Contracts and schema compatibility | [Contracts and data products](../../agents/data-pipeline-operations-agent/04-contracts-schema-evolution-and-data-product-delivery.md) |
| Lineage, quality, quarantine | [Lineage and governance](../../agents/data-pipeline-operations-agent/05-lineage-quality-quarantine-and-governance.md) |
| Backfills, retries, idempotency, exactly-once caveats | [Backfill and reconciliation](../../agents/data-pipeline-operations-agent/06-backfills-replay-idempotency-and-reconciliation.md) |
| Checkpoints, watermarks, late/out-of-order data | [Runtime and adapters](../../agents/data-pipeline-operations-agent/03-batch-stream-cdc-and-tool-adapters.md) |
| Receipts and reconciliation | [Backfill and reconciliation](../../agents/data-pipeline-operations-agent/06-backfills-replay-idempotency-and-reconciliation.md) |
| Context, state, memory controls | [Architecture and state](../../agents/data-pipeline-operations-agent/02-reference-architecture-runtime-and-state.md) and [Security](../../agents/data-pipeline-operations-agent/07-security-identity-tenancy-memory-and-context.md) |
| Identity, tenancy, secrets, prompt injection | [Security](../../agents/data-pipeline-operations-agent/07-security-identity-tenancy-memory-and-context.md) |
| Privacy and governance | [Lineage and governance](../../agents/data-pipeline-operations-agent/05-lineage-quality-quarantine-and-governance.md) |
| Parallelism, cost, capacity | [Observability and deployment](../../agents/data-pipeline-operations-agent/08-observability-slos-scaling-deployment-and-incidents.md) |
| Evals, outcomes, failure injection | [Evaluation and roadmap](../../agents/data-pipeline-operations-agent/09-evaluation-failure-injection-and-delivery-roadmap.md) |
| SLOs, deployment, HA/DR, incidents, upgrades | [Observability and deployment](../../agents/data-pipeline-operations-agent/08-observability-slos-scaling-deployment-and-incidents.md) |
| Category separation | [Workload fit](../../agents/data-pipeline-operations-agent/01-workload-fit-boundaries-and-autonomy.md) |
| Schema-break, partial-backfill, poisoned-source walkthroughs and operator exercises | [Worked flows and runbooks](../../agents/data-pipeline-operations-agent/10-worked-production-flows-and-operator-runbooks.md) |

## 10. Limitations

- Product features and defaults change quickly. The guide deliberately requires versioned adapter manifests and probes.
- Documentation cannot prove a specific deployment's configuration or connector support.
- “Exactly once” cannot be certified generically; it needs a boundary-specific design and failure test.
- Regulatory obligations depend on jurisdiction, contract, and organizational policy; this is engineering guidance, not legal advice.
- Lineage and catalog completeness are deployment properties, not guaranteed by adopting a standard.
- Production case studies reflect their authors' scale and systems and should be adapted cautiously.
- Tool/provider data-handling terms must be reviewed for the actual account and region.
- The illustrative contract fragment is not a substitute for validation against the exact ODCS tooling/version.

## 11. Refresh triggers

Refresh this packet when:

- an orchestrator, streaming engine, Kafka, Debezium, dbt, table format, ODCS, or OpenLineage major version changes;
- deployed exactly-once, checkpoint, snapshot, compatibility, or retention behavior changes;
- an adapter changes API, pagination, idempotency, or public-interface status;
- new OWASP/NIST agent or tool-security guidance appears;
- provider prompt/output retention or residency terms change;
- a production incident reveals an unmodeled effect boundary;
- a new regulated data class or deletion/retention obligation enters scope;
- evaluation shows systematic diagnosis, planning, or verification failure.

At minimum, recheck primary sources and deployed capability manifests quarterly for an operational system.

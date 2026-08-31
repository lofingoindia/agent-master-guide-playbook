# Batch, Streaming, CDC, and Tool Adapters

## 1. One controller, different execution semantics

Batch, streaming, and change-data-capture pipelines share contracts, lineage, receipts, and incident controls. They do not share the same notions of time, completion, replay, or correctness.

| Property | Batch | Streaming | CDC |
|---|---|---|---|
| Primary unit | Run, partition, file, table snapshot | Record, window, checkpoint | Database log event and source position |
| Completion | Bounded run terminates | Usually continuous | Usually continuous |
| Progress | Data interval or partition set | Watermark, offsets, checkpoint | LSN/binlog/SCN plus schema history |
| Recovery | Retry partition or backfill | Restore checkpoint/savepoint and replay | Resume offsets, snapshot, or resnapshot |
| Late data | Next run or explicit repair | Watermark, allowed lateness, triggers | Transaction/log ordering plus sink lag |
| Duplicate risk | Retry and append | Replay after failure | Snapshot/stream overlap and offset flush |
| Publication | Atomic partition/table swap when possible | Checkpoint-coupled or continuous visibility | Sink-specific upsert/delete semantics |

An adapter must expose the native semantics. Flattening all three to “start job / job succeeded” destroys information needed for safe recovery.

## 2. Batch operations

### Logical time, not wall-clock labels

Airflow-style data intervals illustrate an important rule: a run often processes an interval that ended at its scheduling time. Store interval start, interval end, logical date, scheduling timestamp, and actual start separately.

Every interval calculation should:

- use an IANA time zone;
- make inclusive/exclusive boundaries explicit;
- test daylight-saving gaps and repeated hours;
- preserve scheduler-native run identifiers;
- distinguish scheduled, manual, dataset-triggered, and backfill runs.

### Catchup and backfill are not synonyms

Catchup is scheduler behavior for creating historical scheduled intervals. A backfill is an explicit historical operation with a target range and reprocessing policy. The agent must not toggle catchup as a substitute for a reviewed backfill plan.

For Airflow, record:

- reprocessing behavior for existing successful or failed runs;
- backfill-specific maximum active runs;
- run order;
- dry-run output;
- interaction with normal DAG and task concurrency limits.

### Partition publication

Prefer one of:

1. write immutable output, verify it, then atomically update a catalog pointer;
2. write a staging partition, verify it, then replace the target partition;
3. use a table-format transaction to publish a new snapshot.

Direct append to a canonical table is harder to reconcile. If it is unavoidable, use deterministic keys, a fixed remote job identifier, and duplicate detection.

## 3. Streaming time model

### Event time and processing time

Event time comes from the event's domain timestamp. Processing time is when the runtime observes it. A watermark is an estimate that event time has advanced; it is not proof that no earlier event will arrive.

~~~mermaid
sequenceDiagram
    participant S as Source
    participant R as Stream runtime
    participant W as Window state
    participant K as Sink
    S->>R: event time 10:02 arrives at 10:03
    S->>R: event time 10:01 arrives at 10:05
    R->>W: watermark advances to 10:04
    W->>K: on-time result
    S->>R: event time 10:03 arrives at 10:08
    alt within allowed lateness
      R->>W: update window
      W->>K: correction or new pane
    else too late
      R->>K: late side output or quarantine
    end
~~~

The contract must define:

- event-time field and time zone;
- watermark strategy and idleness behavior;
- maximum expected disorder;
- allowed lateness;
- trigger policy;
- accumulating or discarding updates;
- sink correction semantics;
- late-event quarantine and replay policy.

Do not “fix” a freshness incident by advancing a watermark without knowing whether valid late data will be dropped.

### Checkpoints and savepoints

A checkpoint commonly captures source positions and operator state so the runtime can restore a consistent point. A savepoint is an operator-managed recovery or upgrade artifact. Their lifecycle and compatibility differ by runtime.

The agent records:

- checkpoint identifier and completion time;
- source positions;
- state backend and durable location;
- alignment, timeout, and failure information;
- runtime/job version;
- restore compatibility;
- last verified sink commit coupled to the checkpoint.

Never edit checkpoint files. Use runtime-supported restore, upgrade, and disposal procedures.

### Backpressure and state growth

Freshness may degrade because a sink is slow, a hot key concentrates work, state grows, checkpoints take longer, or source input surges. Diagnosis should correlate:

- input, processed, and output rates;
- busy and backpressured time;
- consumer or source lag by partition;
- watermark lag;
- state size and growth;
- checkpoint duration, alignment, failure, and age;
- sink latency and error rate;
- CPU, memory, network, disk, and garbage collection.

Adding parallelism may intensify pressure on a source or sink and can require state redistribution. Treat it as a planned change, not an automatic reflex.

## 4. Exactly-once: define the boundary

“Exactly once” can mean:

- each event affects runtime state exactly once after recovery;
- source offsets and produced Kafka records commit atomically;
- a transactional sink exposes one committed result;
- a business key has one final materialized value;
- an external side effect occurs once.

These are not equivalent.

End-to-end exactly-once requires all relevant boundaries to cooperate:

1. a replayable source;
2. consistent snapshots of source position and operator state;
3. deterministic or replay-safe processing;
4. a transactional or idempotent sink;
5. commit coordination between state and sink;
6. consumers that observe the intended isolation level.

Flink explicitly distinguishes exactly-once state consistency from how often an event may be processed. Kafka transactions can make consume-transform-produce atomic inside Kafka, but an unrelated warehouse, API, or email effect needs its own protocol.

Use a guarantee matrix:

| Boundary | Claimed guarantee | Mechanism | Failure test | Residual risk |
|---|---|---|---|---|
| Kafka source → Flink state | Exactly-once state | Checkpointed offsets and state | Kill during checkpoint | Source retention expires |
| Flink → Kafka sink | Exactly-once visibility | Transactions and read-committed consumers | Kill before/after commit | Transaction timeout or ID collision |
| Kafka → warehouse merge | Effectively once by key | Deterministic merge key and batch receipt | Lose ack after commit | Semantic key is wrong |
| Warehouse → webhook | At least once | Outbox and consumer dedupe | Crash after webhook | Receiver ignores idempotency key |

Do not put “exactly-once pipeline” in a runbook without this matrix.

## 5. Change data capture

CDC requires three durable state domains:

- source log position or connector offsets;
- schema history required to decode earlier records;
- sink application state or deduplication frontier.

Losing one can force a resnapshot or make events undecodable.

### Snapshot and stream overlap

Incremental snapshots commonly read chunks while the live log stream continues. Watermark or signal mechanisms reconcile the overlap, but duplicates can still appear around connector restart and offset persistence. The sink must tolerate repeated events.

Record:

- connector instance and version;
- source database and table identifiers;
- capture identifier or replication slot;
- snapshot mode and chunk bounds;
- offset and schema-history stores;
- log-retention headroom;
- signal channel and request ID;
- tombstone/delete semantics;
- sink key and upsert/delete behavior.

### Transaction semantics

CDC event order can be meaningful within a key or source transaction. Avoid parallelism that reorders events for the same key. If the sink needs transaction boundaries, verify the connector emits and the consumer honors them.

### Outbox pattern

An application outbox couples business state and an event record in one source transaction. The CDC pipeline relays the outbox. A stable event ID supports deduplication, and an aggregate key supports ordering.

This does not automatically make every downstream effect exactly once. The consumer must still deduplicate or transact its own effect.

### Resnapshot gate

Treat a resnapshot as high risk because it can:

- increase source load and log retention;
- overlap live changes;
- replay deletes or historical values unexpectedly;
- overload the sink;
- change lineage and freshness;
- expose data outside current retention or privacy policy.

Require a scoped table list, snapshot mode, source capacity approval, sink duplicate strategy, estimated duration, log-retention calculation, verification, and rollback or forward-recovery plan.

## 6. Tool adapter catalogue

| System family | Read operations | Possible supervised effects | Semantic traps |
|---|---|---|---|
| Airflow | DAG/run/task state, interval, logs, config | Trigger bounded run, pause/unpause, clear selected task | Logical date, catchup, clear can rerun downstream, DAG authors execute code |
| Dagster | Asset materializations, partitions, checks | Launch partition backfill, retry run | Asset checks and backfill behavior vary by version/config |
| Prefect | Flow/task states, deployment, limits | Create run, pause deployment | Transaction and concurrency behavior is not a cross-system transaction |
| dbt | Manifest, run results, tests, contracts | CI job or selected build | Incremental unique key/filter and adapter-specific constraint support |
| Flink | Job/checkpoint/savepoint, lag, metrics | Trigger savepoint, stop, restore approved version | State compatibility, transaction timeout, operator IDs |
| Spark Structured Streaming | Query progress, offsets, state metrics | Start from approved checkpoint | Output mode, watermark/state cleanup, sink guarantee |
| Kafka | Topic config, offsets, lag, transaction metrics | Scoped consumer offset operation | Isolation level, retention, partition ordering |
| Debezium/Kafka Connect | Connector status, offsets, snapshot metrics | Pause/resume, signal incremental snapshot | Default at-least-once, source-specific support, schema history |
| Warehouse/lakehouse | Job state, table snapshot, partition metadata | Fixed-ID job, staging publish | DONE may contain an error; append retries duplicate |
| Catalog/OpenLineage | Dataset, job, run, facets, descendants | Publish normalized lineage | Partial instrumentation and identifier drift |

An adapter release manifest should state the exact platform versions tested. “Stable docs” are moving targets.

### Capability qualification snapshot

The following is a research snapshot from 2026-08-31, not a compatibility promise. A deployment manifest must pin the installed server, client/provider/connector, API, feature flags, authentication mode, and a successful probe for every operation it enables.

| Surface observed | Qualified behavior | Deployment-specific limit or contradiction | Required probe |
|---|---|---|---|
| Airflow 3.3.1 public API v2 | `/api/v2` is the stable programmatic surface; backfill exposes reprocessing mode, bounded concurrency, ordering, and dry run | Backfill code selection can use original or latest DAG bundle depending on explicit request/DAG/global settings; backfills finally default to latest when unset, unlike clear/rerun | Create a nonproduction backfill; assert interval boundaries, reprocessing mode, bundle version, pagination, auth scope, and remote run lookup |
| Dagster 1.13.20 | Partitions/backfills, asset materializations, checks, and cross-run concurrency pools are useful native concepts | Failed checks do not block downstream assets unless configured `blocking`; partitioned asset checks are preview, so they are not a production gate without a local risk decision | Materialize one partition, run a blocking check, launch/cancel a bounded backfill, and verify asset/run IDs plus pool behavior |
| Prefect 3 | Deployments, flow/task state, transactions, and global concurrency are available | A committed transaction record helps idempotency, but default `READ_COMMITTED` does not prevent a same-key race; use a suitable lock manager and `SERIALIZABLE`. Missing/inactive global limits or lease-renewal failure can continue unless strict behavior is selected | Race two same-key test transactions; verify lock storage, strict missing-limit behavior, lease renewal, and deployment collision policy |
| Kafka 4.1.2 / Kafka Connect | Kafka transactions cover Kafka records and offsets; distributed Connect has status/config/offset topics and REST control | `exactly.once.source.support` defaults to `disabled`, needs a staged cluster rollout, and only applies when the source connector declares support. It does not make an external sink exactly once. Connector plugins are separate supply-chain artifacts | Verify broker/worker/client versions, connector `ExactlyOnceSupport`, `read_committed` consumers, transactional IDs, internal-topic durability, plugin digest, and external-sink dedupe |
| Debezium 3.5 on Kafka Connect | Supported source connectors can use Connect exactly-once; PostgreSQL snapshots and streaming preserve an LSN/offset relationship | PostgreSQL schema changes are unsupported during an incremental snapshot. Replication-slot/WAL retention, connector offsets, and source-version features differ; `offset.mismatch.strategy` is technology preview in the observed docs | Run insert/update/delete/tombstone and restart fixtures; verify slot/publication identity, offsets, snapshot window/chunks, schema history, WAL headroom, and EOS configuration |
| Flink stable 2.3 documentation | Checkpointed state can be exactly once; end-to-end delivery still requires replayable source plus transactional/idempotent sink | The stable Kafka connector page reported no connector artifact yet for Flink 2.3. “Stable runtime” therefore does not imply a compatible Kafka connector | Resolve the exact connector artifact and Kafka client; test checkpoint/restore, unique transactional prefix, transaction timeout greater than checkpoint plus worst restart, and committed visibility |
| Spark Structured Streaming 4.1.2 | Checkpoints track query progress; event-time watermarks bound late-data state | Recovery and exactly-once claims are source/sink specific. A changed query, source, state schema, or reused checkpoint location can be incompatible or unsafe | Restart the exact query build from a copied checkpoint; inject duplicate/out-of-order data; verify sink batch IDs, watermark corrections, and forbidden concurrent checkpoint reuse |
| dbt docs v2 / Core 1.12 line | Contracts preflight model shape; incremental and microbatch strategies expose bounded work | Contracts exclude sources, snapshots, seeds, Python, ephemeral, and most custom materializations; constraint enforcement and microbatch replacement differ by adapter. Microbatch time inputs are UTC and upstreams without `event_time` can be fully scanned per batch | Pin dbt engine and adapter; inspect `manifest.json`/`run_results.json`; test unique key, late lookback, targeted backfill, full-refresh guard, constraint enforcement, and batch replacement |
| BigQuery Jobs API | Caller-chosen job IDs make ambiguous job insertion reconcilable | `DONE` can contain `errorResult`; append retries duplicate without fixed job ID. Job identity also includes project/location | Submit a fixed-ID canary, simulate lost acknowledgement, `jobs.get` in the correct location, inspect error result, and verify target snapshot/counts |
| Snowflake SQL API and `COPY` | Same `requestId` with `retry=true` avoids re-executing a successfully processed SQL API request | Idempotent resubmission adds history-lookup overhead. Bulk `COPY` load metadata expires after 64 days; `FORCE` can duplicate and Snowpipe has different history | Retry a request ID after a forced timeout; verify statement handle/result. Test old-file handling without `FORCE`, then reconcile file manifest, load history, query ID, and target snapshot |
| Iceberg spec v1-v3 | Stable field/schema/spec IDs, snapshots, optimistic metadata-pointer commit, and format-version rejection support precise publication | Table identifier fields do not guarantee uniqueness; engine/catalog implementations and supported format versions differ | Verify table UUID, format/schema/spec/snapshot IDs, catalog compare-and-swap, concurrent writer conflict, reader compatibility, and snapshot expiration policy |
| Delta Lake current docs | Table versions, optimistic concurrency, idempotent transaction options, and Change Data Feed can support repair | CDF begins only after enablement and follows table retention/VACUUM. Non-additive changes and column mapping have version-specific CDF restrictions; protocol/features gate readers/writers | Record protocol/features and pre/post versions; test idempotent write identity, concurrent `MERGE`, CDF range across schema changes, restore, and retention |
| Hudi 1.2.0 | Backward-compatible write evolution is the preferred production path | Schema-on-read is experimental and cannot be disabled after accepting incompatible evolution; some writes can succeed while later reads fail | Test the exact table type, writer and reader engines across every intended evolution; reject irreversible feature activation without rollback proof |
| OpenLineage 1.52.0 | Job/run/dataset events and versioned facets provide a portable evidence envelope | A later facet with the same name replaces the prior facet for that entity; producer coverage and identifier normalization still determine graph completeness | Emit and re-ingest start/running/complete plus schema/quality facets; verify namespace mapping, producer/facet schema URLs, dedupe, and ingestion high-watermark |
| GX Core 1.21.0 / Soda v3 / Deequ 2.0.x | Expectations, scan checks, metrics histories, and anomaly checks cover complementary quality policies | GX all-unexpected-row retrieval is limited to `UnexpectedRowsExpectation` and can move sensitive rows; Soda result egress depends on mode/config and its data-contract surface is beta; Deequ artifacts are Spark-version-specific | Run pass/fail/error fixtures, cap/redact unexpected rows, verify local/cloud egress, pin Spark artifact, and normalize immutable result IDs and suite digests |
| Vault or cloud secret manager | Dynamic/short-lived credentials, workload identity, versioned secrets, and auditable retrieval can keep values out of controller state | Product and deployment modes differ. Vault auditing starts disabled and can make Vault unavailable if no audit device can accept an event; mutable `latest` secret aliases make rollback ambiguous | Resolve one envelope-bound credential, verify creation path/version/TTL/audit correlation, deny controller/model access, rotate, disable old version, and test revocation during a worker run |
| OpenTelemetry semantic conventions 1.44.0 | Stable database client spans and trace propagation help connect intent to remote work | Database conventions can be in mixed migration state; GenAI conventions remain development. Query text and parameters can expose sensitive data and increase cardinality | Pin emitted convention set, test `database/dup` migration if used, sanitize query text, leave parameters/prompts/outputs off by default, and preserve native remote IDs |

When a probe contradicts documentation, disable the capability and store the contradiction in the signed manifest. Do not let the model choose the more permissive interpretation.

## 7. Integrations

The controller usually integrates with:

- orchestrator API, not its internal database;
- runtime and connector metrics;
- schema registry and data-contract repository;
- artifact registry and source control;
- catalog and lineage service;
- warehouse or table-format metadata;
- data-quality result store;
- secrets broker or workload identity;
- ticketing/on-call system and audit sink;
- cost and capacity systems.

Prefer official public APIs. Airflow's public-interface guidance is a useful model: internal Python objects or database tables are not a stable control contract.

## 8. Runtime-specific verification

### Batch

- required partitions exist exactly once;
- expected table snapshot or warehouse job succeeded;
- input/output counts and keys reconcile;
- downstream publish pointer moved only after gates passed.

### Streaming

- restored checkpoint is the intended one;
- offsets advance without gaps outside policy;
- watermark and backlog converge;
- output corrections honor late-data semantics;
- transactional or idempotent sink evidence matches the checkpoint.

### CDC

- snapshot completes for the requested table/chunks;
- connector resumes the correct log position;
- inserts, updates, deletes, and tombstones converge at the sink;
- duplicate and ordering fixtures pass;
- source log retention remains safe;
- schema history is durable and readable.

## 9. Selected sources

- [Airflow DAG runs and data intervals](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dag-run.html)
- [Airflow backfill](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/backfill.html)
- [Airflow 3.3 DAG bundle version controls](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#rerun-with-latest-version)
- [Dagster partitions and backfills](https://docs.dagster.io/guides/build/partitions-and-backfills)
- [Dagster asset checks](https://docs.dagster.io/guides/test/asset-checks)
- [Prefect transactions](https://docs.prefect.io/v3/advanced/transactions)
- [Prefect global concurrency limits](https://docs.prefect.io/v3/concepts/global-concurrency-limits)
- [Apache Flink fault tolerance](https://nightlies.apache.org/flink/flink-docs-stable/docs/learn-flink/fault_tolerance/)
- [Apache Flink watermarks](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/datastream/event-time/built_in/)
- [Apache Beam programming guide](https://beam.apache.org/documentation/programming-guide/)
- [Spark Structured Streaming guide](https://spark.apache.org/docs/latest/streaming/apis-on-dataframes-and-datasets.html)
- [Kafka design: delivery semantics](https://kafka.apache.org/41/design/design/)
- [Kafka Connect worker configuration](https://kafka.apache.org/41/generated/connect_config.html)
- [Debezium exactly-once delivery](https://debezium.io/documentation/reference/3.5/configuration/eos.html)
- [Debezium PostgreSQL connector](https://debezium.io/documentation/reference/stable/connectors/postgresql.html)
- [Debezium outbox event router](https://debezium.io/documentation/reference/stable/transformations/outbox-event-router.html)
- [dbt model contracts](https://docs.getdbt.com/docs/mesh/govern/model-contracts)
- [dbt microbatch incremental models](https://docs.getdbt.com/docs/build/incremental-microbatch)
- [BigQuery reliability and fixed job IDs](https://docs.cloud.google.com/bigquery/docs/reliability-intro)
- [Snowflake SQL API request resubmission](https://docs.snowflake.com/en/developer-guide/sql-api/submitting-requests)
- [Snowflake bulk load metadata](https://docs.snowflake.com/en/user-guide/data-load-considerations-load)
- [Delta Lake Change Data Feed](https://docs.delta.io/delta-change-data-feed/)
- [Apache Hudi schema evolution](https://hudi.apache.org/docs/schema_evolution/)
- [OpenLineage 1.52.0 release](https://openlineage.io/docs/releases/1_52_0/)

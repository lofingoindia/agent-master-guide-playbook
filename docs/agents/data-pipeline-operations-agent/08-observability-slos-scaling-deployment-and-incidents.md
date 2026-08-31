# Observability, SLOs, Scaling, Deployment, and Incidents

## 1. Observe the delivered data outcome

Component uptime is necessary but insufficient. A scheduler can be healthy while data is stale; a stream can process records while its watermark is stuck; a warehouse job can finish while output violates its contract.

Use four layers:

| Layer | Signals |
|---|---|
| Data outcome | Freshness, completeness, reconciliation, duplicates, quarantine, consumer visibility |
| Pipeline/runtime | Run success, backlog, watermark, checkpoint, state, throughput, backpressure |
| Agent control | State age, evidence freshness, policy decisions, approvals, effects, reconciliation |
| Platform | Availability, latency, saturation, errors, dependencies, credentials |

Page primarily on user-visible outcome or rapid error-budget burn. Route component alerts to diagnosis when they threaten that outcome.

## 2. SLOs

### Data-product SLOs

Examples:

- 99.5% of hourly partitions are published within 90 minutes of interval end;
- 99.9% of expected CDC keys converge within 10 minutes;
- no critical contract violations reach canonical publication;
- 99.99% of accepted deletion work items are verified within the policy deadline.

Define the event population, measurement point, exclusions, and late corrections. Use synthetic/test-account data where feasible to measure correctness end to end, as the Google SRE data-pipeline guidance recommends.

### Agent SLOs

Examples:

- 99.9% of accepted read-only investigations reach terminal state within 10 minutes;
- 99.99% of authorized effect intents have a reconciled terminal receipt;
- 100% of production effects are attributable to a valid, unexpired envelope;
- 99% of evidence used for decisions meets its freshness contract;
- 99.9% of effect verification begins within 2 minutes of remote completion.

Do not define “model answered” as success.

### Error-budget alerts

Use multi-window burn-rate alerts for SLOs so a fast, severe failure pages quickly while slower degradation creates a ticket. Tune with historical incidents and operator capacity.

## 3. Metrics

### Data outcome

- publication lag by product and criticality;
- missing, duplicate, late, rejected, and quarantined records;
- reconciliation mismatch counts and age;
- contract/quality results by version;
- downstream acknowledgement lag;
- repair and backfill completion.

### Batch and orchestrator

- scheduled versus created versus terminal runs;
- queue, start, and duration latency;
- task attempts and classified failures;
- backlog by interval;
- active run/task limits;
- historical work versus live work resource share.

### Streaming and CDC

- source lag by partition;
- watermark lag and idle partitions;
- input/output rate;
- backpressure/busy time;
- checkpoint age, duration, size, alignment, and failure;
- state size and growth;
- transaction failures and aborts;
- CDC source log retention headroom;
- snapshot progress and offset persistence.

### Agent control plane

- operations by state and age;
- ambiguous effects awaiting reconciliation;
- duplicate effect keys rejected;
- policy allow/deny/approval results;
- approval wait and expiry;
- read/effect adapter latency, errors, rate limits, and capability drift;
- model latency, structured-output rejection, token use, and provider fallback;
- verifier latency and disagreement with executor;
- memory retrieval and expired-item use attempts.

Keep tenant labels bounded. Use hashes or stable IDs instead of high-cardinality raw names when metric systems require it.

## 4. Traces and logs

Trace across:

~~~text
request -> evidence reads -> model proposal -> policy
-> approval -> dispatch -> remote run/job -> receipt
-> verification -> publication/incident
~~~

Propagate a correlation ID where platforms allow it, but retain native run/job/checkpoint IDs.

Span attributes should include low-risk identifiers and versions. Put sensitive payloads in governed evidence storage, referenced by hash. OpenTelemetry database semantic conventions help normalize database-client spans; capture SQL text only under an explicit privacy policy.

Structured logs include:

- operation/effect/attempt IDs;
- tenant/environment and adapter;
- state transition;
- error class and retry disposition;
- evidence or receipt reference;
- latency and budget consumption.

Never log secrets or raw records.

### Signal ownership and retention

Do not collapse every operational signal into logs or call every metric an SLO.

| Signal | Answers | Cardinality/retention posture | Must not become |
|---|---|---|---|
| Metric | How often, how much, how saturated? | Bounded labels; aggregated retention | Per-row audit trail |
| Trace | Where did one intent spend time and cross boundaries? | Sampled spans with operation/effect/native remote IDs | Storage for prompts, SQL literals, or raw rows |
| Operational log | What did a component observe locally? | Structured, severity-based, shorter retention | Workflow source of truth |
| Audit event | Who accessed/decided/approved/executed what, under which version and scope? | Append-only, access-controlled, policy retention | Debug dump or mutable dashboard event |
| Evidence/receipt | What authoritative artifact proves a fact or outcome? | Immutable governed object plus digest and retention class | High-cardinality metric label |
| SLI/SLO/error budget | Did a defined user-visible population meet an objective? | Versioned definition and long enough history for budgeting | Raw component availability or model-answer rate |

OpenTelemetry semantic conventions are versioned inputs. In the observed 1.44.0 line, database client spans are stable but database instrumentation can still emit old versus stable attributes during migration, while GenAI conventions remain development. Pin the emitted convention set. Keep query parameters, unredacted query text, prompts, and model outputs off by default.

## 5. Capacity and cost

Estimate before admitting historical work:

~~~text
input_bytes
× read_amplification
× transformation_factor
× retry_factor
+ shuffle/state/write/storage cost
+ model/evidence/verification cost
~~~

Track both financial cost and resource pressure.

Capacity dimensions:

- source queries, IOPS, connections, or log retention;
- broker partitions and network;
- stream state and checkpoint storage;
- compute slots/executors/workers;
- warehouse quotas and concurrent jobs;
- sink commit/merge throughput;
- catalog/lineage API rate limits;
- agent database/queue/object storage;
- model token/rate limits.

Use admission control:

~~~mermaid
flowchart LR
    R[Operation request] --> E[Estimate]
    E --> B{Budgets available?}
    B -->|no| W[Queue or reject]
    B -->|yes| C[Canary scope]
    C --> H{Health gates pass?}
    H -->|no| P[Pause and release capacity]
    H -->|yes| X[Bounded expansion]
    X --> H
~~~

Do not auto-scale a backfill beyond the source or sink budget just because worker capacity is available.

### Recovery-load math

For each resource, convert capacity to the same source-equivalent unit:

~~~text
source_spare      = safe_source_rate - live_source_rate
compute_spare_in  = (safe_compute_rate - live_compute_rate) / transform_amplification
sink_spare_in     = (safe_sink_rate - live_sink_rate) / write_amplification
recovery_rate     = min(source_spare, compute_spare_in, sink_spare_in, reserved_backfill_rate)
effective_bytes   = backlog_bytes * read_amplification * retry_factor + verification_bytes
recovery_time     = effective_bytes / recovery_rate

stream_catchup_rate = sustained_processing_rate - live_arrival_rate
stream_catchup_time = lag_events / stream_catchup_rate
~~~

If `stream_catchup_rate <= 0`, the stream never catches up. If a source query scans 2.4 bytes for every output byte, use 2.4 in the source/compute estimate; do not estimate from sink size alone.

Worked admission example:

| Input | Value |
|---|---:|
| Historical source backlog | 18 TB |
| Source spare read rate | 120 MB/s |
| Compute spare, source-equivalent | 80 MB/s |
| Sink spare, source-equivalent | 55 MB/s |
| Reserved backfill rate | 70 MB/s |
| Read/retry/verification factor | 1.25 |

The bottleneck is the sink at 55 MB/s. Using decimal units, effective work is `18 TB × 1.25 = 22.5 TB`, so the best-case duration is about `22,500,000 MB / 55 MB/s = 113.6 hours`. A 36-hour recovery target would require about 174 MB/s of safe end-to-end recovery throughput, which this estate does not have. The valid plan is to narrow priority partitions, acquire approved capacity, or renegotiate the target—not to set worker concurrency to an arbitrary larger number.

For a streaming example, a backlog of 612 million events with 35,000 events/s live arrival and 52,000 events/s sustained processing has only 17,000 events/s catch-up capacity: about 10 hours before checkpoint, sink, or hot-key degradation. Admission uses the lower sustained rate observed during fault recovery, not a benchmark with an empty state store.

Reserve capacity for verification, reconciliation, retries, checkpoint I/O, and live-workload variance. Pause expansion when live freshness burns its error budget, source log-retention headroom falls below recovery time plus margin, checkpoint duration approaches timeout, or the sink error/latency gate fails.

## 6. High availability

### Control plane

- multiple stateless API/controller replicas;
- transactional HA state database;
- durable queue;
- expiring work leases;
- effect-key uniqueness constraint;
- immutable evidence and receipt storage;
- health checks that include critical dependencies;
- no reliance on local disk or in-process memory.

### Effect plane

- worker pools isolated by trust boundary;
- JIT credential renewal;
- graceful drain before upgrades;
- reconciliation after worker loss;
- dead-letter state that remains actionable, not a data sink;
- circuit breakers for target failure and rate limiting.

### Data runtimes

Use the runtime's supported HA model. Airflow production guidance calls for an external database, durable/distributed logs, heartbeat monitoring, and managed upgrade practices. Flink on Kubernetes can use Kubernetes HA services, but checkpoint/savepoint storage and application state still require durable configuration.

## 7. Disaster recovery

Define RPO/RTO separately:

- agent control state and receipts;
- orchestrator metadata;
- pipeline definitions and artifacts;
- contracts/schema history;
- stream checkpoints/savepoints;
- CDC offsets/schema history;
- raw source history;
- catalog and lineage events;
- sink tables/snapshots;
- secrets and policy configuration.

Run exercises:

1. restore the controller database and evidence index;
2. reconcile all nonterminal effects against remote systems;
3. restore a representative stream from a checkpoint/savepoint;
4. recover CDC state or prove the resnapshot plan;
5. rebuild a data product from retained source;
6. fail over credentials/endpoints without crossing tenant or residency boundaries;
7. verify consumer-visible outcomes and audit continuity.

Regional failover can duplicate effects if two controllers act. Use one authoritative lease/fencing mechanism and region-qualified execution policy.

## 8. Deployment and upgrade gates

### Release sequence

1. schema-compatible state migration;
2. deploy read-only controller canary;
3. run adapter capability probes;
4. shadow recorded and live incidents;
5. deploy effect workers disabled;
6. enable one low-risk action for a canary tenant/pipeline;
7. verify receipts, reconciliation, telemetry, and rollback;
8. expand by policy.

Pin:

- model/provider and prompt-template version;
- controller and policy bundle;
- adapter and upstream API version;
- runtime/container/dependency versions;
- contract/schema/transform artifacts;
- evaluation fixture set.

Worker versioning or task queues can keep in-flight durable workflows on compatible code during upgrades. For stateful stream jobs, use runtime-supported savepoint and state-compatibility procedures.

### Block rollout when

- adapter responses drift from the tested schema;
- effect lookup or reconciliation fails;
- state migration is not backward/forward recoverable;
- critical offline fixtures regress;
- tenant/isolation or injection tests fail;
- operational dashboards and alerts are absent;
- restore/reconciliation was not exercised;
- cost or latency exceeds the release budget.

## 9. Incident response

### Severity examples

| Severity | Example |
|---|---|
| SEV-1 | Cross-tenant exposure, widespread corruption, deletion/privacy failure |
| SEV-2 | Critical product stale or incorrect, CDC gap, unbounded duplicate effects |
| SEV-3 | Single noncritical pipeline delayed, bounded quarantine, degraded diagnosis |
| SEV-4 | Documentation, telemetry, or nonproduction defect |

### Response loop

1. stop propagation with a deterministic gate or pause;
2. preserve checkpoints, offsets, manifests, logs, versions, and receipts;
3. identify the last known-good product version and frontier;
4. establish data impact, not only component symptoms;
5. determine whether any effect acknowledgement is ambiguous;
6. choose rollback or forward recovery;
7. validate a canary repair;
8. repair descendants and notify owners;
9. verify user-visible outcome and SLO recovery;
10. update tested runbooks and curated incident patterns.

The agent can build the timeline and test hypotheses, but the incident commander owns severity, communication, and risky action approval.

## 10. Failure injection

Regularly inject:

- controller crash before and after intent recording;
- worker crash before dispatch, after remote commit, and before receipt write;
- delayed, duplicated, reordered, or corrupted adapter responses;
- stale catalog or missing lineage;
- expired credential and approval;
- source/sink throttling;
- checkpoint timeout/corruption;
- broker partition loss and retention pressure;
- CDC offset/schema-history loss;
- warehouse job acknowledgement loss;
- verifier outage or contradictory result;
- model outage, malformed response, or adversarial evidence;
- regional dependency outage.

Success means no unauthorized or duplicate effect, durable recoverability, and a proven data outcome—not merely process restart.

## 11. Dashboards

Keep three views:

- **product view:** freshness, correctness, quarantine, consumer status, error budget;
- **operation view:** state machine, evidence age, approvals, effects, receipts, verification;
- **platform view:** controller/adapters/runtimes capacity, errors, costs, and deployment version.

An on-call should reach the authoritative remote job, table snapshot, checkpoint, contract, lineage event, and receipt without searching unstructured chat.

## 12. Selected sources

- [Google SRE Workbook: Data Processing Pipelines](https://sre.google/workbook/data-processing/)
- [Google SRE Workbook: Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
- [Airflow production deployment](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/production-deployment.html)
- [Apache Flink checkpoints versus savepoints](https://nightlies.apache.org/flink/flink-docs-stable/docs/ops/state/checkpoints_vs_savepoints/)
- [Apache Flink large-state tuning](https://nightlies.apache.org/flink/flink-docs-stable/docs/ops/state/large_state_tuning/)
- [Apache Flink Kubernetes deployment and HA](https://nightlies.apache.org/flink/flink-docs-stable/docs/deployment/resource-providers/standalone/kubernetes/)
- [Temporal worker versioning](https://docs.temporal.io/production-deployment/worker-deployments/worker-versioning)
- [OpenTelemetry database spans](https://opentelemetry.io/docs/specs/semconv/db/database-spans/)
- [OpenTelemetry semantic conventions 1.44.0](https://opentelemetry.io/docs/specs/semconv/)
- [OpenTelemetry database convention migration](https://opentelemetry.io/docs/specs/semconv/db/)
- [OpenTelemetry generative-AI attributes](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/)

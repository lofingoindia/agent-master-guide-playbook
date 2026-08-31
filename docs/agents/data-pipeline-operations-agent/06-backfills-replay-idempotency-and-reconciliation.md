# Backfills, Replay, Idempotency, and Reconciliation

## 1. A backfill is a controlled historical release

A production backfill is not “rerun everything between two dates.” It reconstructs historical output under explicit versions and constraints, competes with live work, and can overwrite or duplicate already consumed data.

Every backfill needs:

- business reason and incident/change reference;
- tenant, environment, pipeline, dataset, and exact interval/partition set;
- source snapshot or retention proof;
- pipeline, dependency, image, transform, contract, and schema versions;
- reprocessing policy for existing output;
- late-data and delete semantics;
- estimated records, bytes, compute, duration, and cost;
- source/sink/concurrency budgets;
- output isolation and publication method;
- effect key and remote run identifiers;
- verification, downstream repair, and recovery plan;
- owner, approvals, and expiry.

## 2. Planning workflow

~~~mermaid
flowchart TB
    I[Intake and scope] --> E[Evidence and retention check]
    E --> V[Pin executable versions]
    V --> D[Dry run and estimate]
    D --> S[Shadow or staging output]
    S --> R[Reconcile and quality gates]
    R -->|fail| Q[Quarantine and diagnose]
    R -->|pass| A[Approval bound to result and plan]
    A --> P[Atomic publish or deterministic merge]
    P --> C[Downstream repair and consumer checks]
    C --> O[Outcome receipt and close]
~~~

The version used should be deliberate:

- **historical version** reproduces what should have happened then;
- **current fixed version** repairs historical data to today's contract;
- **migration version** transforms old input into a new product version.

Record the choice. “Latest” is not reproducible.

## 3. Interval and partition rules

Represent ranges as half-open intervals, for example `[start, end)`. Normalize to UTC while retaining the contract time zone. Materialize the exact partition set before approval.

Test:

- daylight-saving gaps and repeated local hours;
- leap day and month-end boundaries;
- calendars or fiscal periods;
- partially populated intervals;
- dynamic partitions discovered after approval;
- upstream retention shorter than the requested range;
- downstream dependencies with different grain.

If partitions can change after planning, approval binds to a manifest hash, not a predicate such as “last month.”

## 4. Reprocessing policy

| Existing state | Typical choice | Risk |
|---|---|---|
| No output | Create | Source history may still be incomplete |
| Failed/partial output | Replace or deterministic merge | Partial artifacts and downstream reads |
| Successful but wrong output | Shadow, verify, atomic replace | Consumer corrections and caching |
| Successful and correct output | Skip | Waste and unnecessary churn |
| Unknown/ambiguous | Reconcile before action | Duplicate or overwrite |

Airflow backfill reprocessing behaviors—none, failed, and completed—are useful scheduler controls, but they do not define sink correctness. A failed orchestrator run may have committed data; a successful run may have produced invalid data.

## 5. Idempotency

An operation is idempotent when repeating the same logical request produces the same intended state without an additional unintended effect.

### Effect key

Construct from stable semantics:

~~~text
tenant / environment / pipeline / operation
/ interval-or-frontier-hash
/ executable-version-set-hash
/ target-product-version
~~~

Do not include attempt number or a random UUID in the logical key. Attempts have separate IDs.

### Strategies

| Strategy | Works well for | Failure mode |
|---|---|---|
| Remote fixed job/run ID | APIs with lookup by caller ID | ID retention or scope differs |
| Upsert/merge by stable business key | Mutable materialized views | Wrong key or nondeterministic winner |
| Partition replacement | Bounded batch output | Readers see partial state without atomic swap |
| Immutable files plus manifest pointer | Lake/object storage | Orphan cleanup and pointer race |
| Transactional table snapshot | Lakehouse publication | Concurrent commit and protocol compatibility |
| Source offset + sink transaction | Stream runtime | Only covers participating systems |
| Consumer idempotency table | External side effects | Retention, hot keys, cross-region consistency |
| Outbox/inbox | Application integration | Operational overhead and consumer compliance |

Idempotency is scoped. A task can be idempotent while a notification it sends is not.

## 6. Ambiguous acknowledgement protocol

The most dangerous case is a timeout after dispatch:

~~~mermaid
sequenceDiagram
    participant C as Controller
    participant L as Effect ledger
    participant A as Adapter
    participant T as Target
    C->>L: record INTENT(effect_key, request_hash)
    C->>A: execute signed envelope
    A->>T: request with effect key
    T--xA: acknowledgement lost
    A--xC: timeout
    C->>L: mark AMBIGUOUS
    C->>A: lookup effect key / inspect target
    A->>T: query stable key or intended state
    T-->>A: committed receipt
    A-->>C: reconciled existing effect
    C->>L: record remote receipt
    C->>C: verify outcome; do not retry
~~~

If the target has no idempotency lookup, reconcile its authoritative state. If neither is possible, escalate. Blind retry is not a safe default.

BigQuery illustrates the pattern: use a caller-chosen job ID, and after an ambiguous insertion or conflict, retrieve the existing job. A job reaching `DONE` still requires inspecting its error result.

## 7. Retries

Classify failures:

| Class | Example | Disposition |
|---|---|---|
| Definitive pre-effect transient | Rate limit before acceptance | Retry with backoff within budget |
| Ambiguous | Network timeout after dispatch | Reconcile first |
| Deterministic input | Contract/schema violation | Do not retry unchanged |
| Capacity | Source overload or sink quota | Reschedule/admit under budget |
| Concurrency conflict | Optimistic table commit conflict | Refresh base state and replan |
| Authentication | Expired JIT token | Renew only if authorization remains valid |
| Authorization/policy | Denied resource | Stop and escalate |
| Corruption | Bad checkpoint or checksum | Quarantine and use recovery procedure |

Retries need attempt and time budgets. A backlog of retries must not starve live SLO-critical work.

### Cancellation protocol

Cancellation is an effect request, not proof that work stopped or that output was removed.

~~~text
CANCEL_INTENT_RECORDED
  -> CANCEL_REQUESTED
  -> TARGET_ACKNOWLEDGED
  -> TARGET_TERMINAL_OBSERVED
  -> PARTIAL_OUTPUT_RECONCILED
  -> CLEANUP_OR_QUARANTINE_VERIFIED
~~~

Before closing a cancellation:

- identify which remote tasks, transactions, queries, files, partitions, and commits can continue independently;
- record the target's cancellation guarantee: best effort, cooperative, queued-only, or terminal;
- inspect source/frontier movement and every target snapshot or staging prefix;
- quarantine or delete partial output only through its own authorized effect;
- release capacity reservations and leases without deleting the effect receipt;
- decide whether resume uses the same effect key, a child key for the remaining manifest, or a new reviewed plan.

A controller timeout must not silently issue remote cancellation. Conversely, a deterministic harm-stop policy may pre-authorize pause/cancel even after the original approval expires, but resume or publication needs a fresh decision.

## 8. Backfill isolation

Prefer:

- a separate backfill queue or pool;
- backfill-specific max active runs;
- source and sink token buckets;
- workload management or warehouse reservations;
- per-tenant quotas;
- shadow outputs;
- rate ramp-up after a canary partition;
- pause triggers for live freshness, error, cost, or quality degradation.

Bounded parallelism should reflect the bottleneck. If ten pipeline tasks all scan the same source, task concurrency of ten is not a safe source budget.

Persist a progress manifest per planned unit:

| Field | Purpose |
|---|---|
| `unit_id` | Stable partition/chunk identity from the approved manifest |
| `effect_key` | Deduplicates the semantic unit |
| `input_frontier` | Proves the exact source slice read |
| `output_snapshot_or_batch` | Locates committed or staged output |
| `state` | Planned, running, unknown, reconciled, verified, quarantined, or published |
| `quality_reconciliation_refs` | Proves the unit's correctness |
| `downstream_repair_state` | Tracks already-consumed corrections |

On restart, reconcile `unknown` units first, skip only verified units, and regenerate neither the manifest nor versions under an existing approval. A partial backfill is an expected recovery state, not a reason to rerun the whole interval.

## 9. Shadow and publication

### Shadow validation

Run representative partitions first and compare:

- schema and contract;
- key set and duplicate/delete behavior;
- partitioned row counts and hashes;
- aggregates with tolerances only where justified;
- old versus new output;
- downstream query or consumer fixture behavior;
- resource and cost estimates.

### Publication

Prefer atomic pointer/snapshot or partition replacement. When a merge is necessary:

- use a stable key;
- define update ordering and delete semantics;
- capture input batch/frontier;
- pin transformation version;
- record affected rows and table snapshot/job ID;
- verify no other writer changed the same scope unexpectedly.

If downstream consumers already read bad data, replacing the source table is not full recovery. Enumerate caches, derived products, exports, model features, and subscriptions that require repair.

## 10. Reconciliation specification

~~~yaml
reconciliation:
  expected:
    partitions_manifest: evidence://manifests/sha256:...
    source_frontier: evidence://frontiers/sha256:...
    operations:
      inserts: 184922
      updates: 1931
      deletes: 83
  actual:
    target_snapshot: iceberg:orders:89104
    partition_hashes: evidence://hashes/sha256:...
    quarantine_count: 17
  tolerances:
    row_count: 0
    monetary_sum_minor_units: 0
  verdict: fail
  mismatches:
    - partition: 2026-08-28T04
      kind: missing_keys
      evidence_ref: ev_438
~~~

Never invent a tolerance after seeing a mismatch. It belongs to the versioned contract or approved plan.

## 11. Rollback versus forward recovery

Rollback is only valid when:

- the previous snapshot/version remains available;
- consumers can tolerate reversal;
- no irreversible external effects occurred;
- source and sink semantics remain compatible;
- privacy/retention rules allow the restored data.

Forward recovery is often safer:

- stop publication;
- repair or recompute a bounded scope;
- publish a corrective snapshot;
- rebuild affected descendants;
- emit correction events where required;
- verify consumer-visible state.

The plan must state which approach applies before execution.

## 12. Completion criteria

A backfill closes only when:

- all intended intervals are terminal and reconciled;
- canonical publication points to the verified output;
- contract and quality gates pass or documented warnings are accepted;
- lineage events identify the historical operation and output version;
- downstream repair is complete or explicitly tracked;
- live workload SLOs have recovered;
- cost and capacity are recorded;
- temporary artifacts have an owner and cleanup schedule;
- receipts and evidence are retained under policy;
- incident/runbook learning is reviewed before entering curated knowledge.

## 13. Selected sources

- [Airflow backfill](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/backfill.html)
- [Databricks Lakeflow backfill guidance](https://learn.microsoft.com/en-us/azure/databricks/ldp/flows-backfill)
- [dbt incremental models](https://docs.getdbt.com/docs/build/incremental-models)
- [dbt microbatch incremental models](https://docs.getdbt.com/docs/build/incremental-microbatch)
- [BigQuery reliability and idempotent job IDs](https://docs.cloud.google.com/bigquery/docs/reliability-intro)
- [BigQuery running jobs and error results](https://docs.cloud.google.com/bigquery/docs/running-jobs)
- [Apache Iceberg specification](https://iceberg.apache.org/spec/)
- [Delta Lake concurrency control](https://docs.delta.io/concurrency-control/)

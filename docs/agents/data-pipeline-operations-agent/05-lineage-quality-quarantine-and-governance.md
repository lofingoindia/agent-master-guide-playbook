# Lineage, Quality, Quarantine, and Governance

## 1. Trust is assembled, not inferred

A dataset is not trusted merely because:

- its pipeline succeeded;
- a catalog contains an entry;
- every row matched a schema;
- a data-quality suite passed;
- lineage shows a path;
- an owner approved a change.

Production publication combines contract, quality, lineage, governance, and operational evidence. Each has different failure modes.

## 2. Lineage model

OpenLineage defines a practical event model around jobs, runs, datasets, and extensible facets. A run begins and ends through events, while facets can carry schema, source code, ownership, column lineage, quality, and output statistics.

Normalize lineage identifiers:

~~~text
dataset_id = namespace + canonical_name + environment + tenant_boundary
job_id     = namespace + canonical_job_name + definition_version
run_id     = platform-native ID mapped to a UUID
~~~

Store the raw producer event and the normalized graph result. Preserve the event's schema URL/version and content hash.

For OpenLineage, dataset identity is the pair `namespace + name`, job identity is `namespace + name`, and a run has its own `runId`. Do not append environment or tenant ad hoc unless the organization's canonical namespace rule does so consistently. Keep a separate mapping to the governed dataset/product ID. Record the producer URI/release, event ID/time, facet `_schemaURL`, ingestion cursor, and raw event hash. In the observed 1.52.0 specification line, a new facet with the same name replaces the prior facet for that entity; graph state alone therefore does not preserve every historical assertion.

### Lineage completeness

Lineage is usually incomplete because of:

- uninstrumented scripts or manual loads;
- dynamic SQL;
- cross-account or cross-cloud movement;
- aliases and renamed resources;
- views or stored procedures the collector cannot parse;
- sample-based or delayed ingestion;
- column-level support gaps;
- deleted catalog objects;
- environment or tenant identifier collisions.

Therefore report:

- known upstreams and downstreams;
- last observation time;
- instrumentation coverage;
- table- versus column-level confidence;
- unresolved identifiers;
- graph truncation limits;
- owners whose systems are not instrumented.

“No descendants found” means no descendants were found in the observed graph, not that none exist.

## 3. Blast-radius algorithm

For a proposed change:

1. resolve the physical dataset to a stable contract/product identifier;
2. query direct and transitive descendants with depth and count bounds;
3. include orchestration dependencies not represented as data lineage;
4. include contract subscribers and owners;
5. compare recent query or access evidence when policy permits;
6. flag identifier conflicts and missing coverage;
7. group impact by tenant, environment, criticality, and field;
8. require human review when coverage is insufficient for a breaking change.

Do not send restricted resource names to a model that lacks permission to view them. Catalog and lineage access is separate from data access.

## 4. Data-quality gates

Quality checks should map to a documented failure policy:

| Severity | Example | Default action |
|---|---|---|
| Observe | Distribution drift below decision threshold | Record and trend |
| Warn | Small freshness or volume deviation | Publish with visible warning if contract permits |
| Quarantine | Unexpected records with isolatable scope | Hold affected records/partition |
| Block | Key uniqueness, schema, cross-tenant, or critical reconciliation failure | Prevent publication |
| Incident | Broad corruption, deletion failure, privacy breach | Stop propagation and page owner |

Useful dimensions:

- completeness and nullability;
- uniqueness and key integrity;
- validity and domain constraints;
- referential integrity;
- volume and distribution;
- freshness and timeliness;
- duplicate and delete behavior;
- cross-source reconciliation;
- privacy classification and tenant isolation.

Keep thresholds versioned with owner, rationale, training window when statistical, and last review. An agent may explain a failure; it cannot silently relax a threshold.

Separate policy from diagnosis:

| Concern | Deterministic path | Bounded model path |
|---|---|---|
| Which suite applies | Product/dataset ID + effective interval + policy resolver | None |
| Pass/warn/quarantine/block | Versioned rule, severity, and threshold evaluation | Explain the failed rules |
| Statistical threshold | Approved algorithm, training window, exclusion policy, and frozen parameters | Suggest a candidate for offline review only |
| Unexpected-row selection | Quality engine query with access, row, byte, and time limits | Summarize redacted aggregates or sampled references |
| Reconciliation | Exact manifests/frontiers/counts/hashes under predeclared tolerances | Rank plausible sources of mismatch |
| Release from quarantine | Re-run mandatory gates plus signed approval | Draft reviewer context |

Normalize every engine result before policy evaluation:

~~~yaml
quality_result:
  result_id: gx://prod-eu/validation/01K...
  dataset_id: urn:dataset:commerce:orders-hourly
  dataset_version: iceberg:snapshot:89104
  interval: [2026-08-31T10:00:00Z, 2026-08-31T11:00:00Z]
  suite_id: orders-publication
  suite_version: 18
  suite_digest: sha256:...
  engine: gx-core
  engine_version: 1.21.0
  status: fail
  counts: {passed: 18, warned: 1, failed: 1, errored: 0}
  failed_rule_ids: [orders.key.unique]
  unexpected_rows_ref: quarantine://q_91
  observed_at: 2026-08-31T11:08:00Z
  result_digest: sha256:...
~~~

A missing, errored, stale, or unversioned mandatory result is not a pass. GX, Soda, Deequ, dbt tests, and native warehouse constraints use different status vocabularies and data-access behavior; adapters must map them without discarding `error` versus `fail`.

### Contract checks versus observed checks

Schema validation asks whether a record can conform. Data-quality validation asks whether the observed population meets expectations. Reconciliation asks whether expected source effects arrived. All three are needed.

Great Expectations validation definitions and suites are one implementation. Deequ metrics repositories and anomaly detection are another. Soda contracts offer another syntax. Choose based on existing stack; preserve normalized outcomes rather than designing a new quality language.

## 5. Quarantine

Quarantine isolates suspect output without losing forensic evidence.

Store:

- quarantine ID and policy reason;
- tenant, environment, pipeline, interval, partition/key bounds;
- source and target snapshot/frontier;
- contract and quality-suite version;
- immutable location reference and hash;
- record count and aggregate statistics;
- access classification and retention;
- release, repair, or deletion state;
- approver and verification references.

Do not copy raw quarantined rows into the agent state, prompt, incident ticket, or general logs.

~~~mermaid
flowchart LR
    B[Built output] --> C{Contract}
    C -->|fail| Q[Quarantine]
    C -->|pass| D{Quality}
    D -->|block/quarantine| Q
    D -->|pass/warn| G{Governance}
    G -->|deny| Q
    G -->|allow| P[Publish]
    Q --> R[Repair or reviewed release]
    R --> C
~~~

Release requires rerunning the applicable contract, quality, reconciliation, and governance checks. A comment in a ticket is not a release control.

## 6. Reconciliation

Quality can pass while records are missing. Reconciliation compares effects across boundaries.

Use the strongest available measures:

- source and target key-set hashes for bounded intervals;
- insert/update/delete counts;
- per-partition counts and checksums;
- high-water marks and gaps;
- source transaction IDs mapped to sink batch IDs;
- table snapshot manifests;
- rejected and quarantined record totals;
- consumer acknowledgement or published-product manifest.

Aggregate counts alone do not detect substitutions or duplicates that cancel each other. For large datasets, use partitioned deterministic hashes, sketches with documented collision risk, or sampled evidence plus exception policy.

### Receipt hierarchy

~~~text
request receipt
  < remote terminal receipt
  < sink commit receipt
  < reconciliation receipt
  < consumer-visible outcome receipt
~~~

Choose the required level per action. Retrying one task may need remote and sink receipts. Publishing a critical product should require reconciliation and consumer-visible evidence.

## 7. Privacy and retention

DataOps workflows frequently touch data across raw, staged, canonical, quarantine, checkpoint, log, cache, and backup systems. A privacy or retention workflow must inventory all applicable stores.

For deletion:

1. authenticate and authorize the request in the designated privacy system;
2. resolve subject identifiers through an approved identity map;
3. use access-aware lineage to enumerate descendants;
4. apply legal-hold, retention, and jurisdiction policy;
5. create deterministic deletion work items for owning systems;
6. execute with scoped credentials and receipts;
7. verify live stores, replicas, indexes, quarantine, caches, and future reprocessing exclusions;
8. record how immutable backups are handled under policy;
9. prevent a later backfill from resurrecting deleted data;
10. produce an auditable outcome without retaining the deleted content.

The model can assist discovery and exception summarization. It must not decide lawful basis, override holds, or retain subject data as memory.

## 8. Governance and access

Evaluate at least:

- data classification and permitted purpose;
- tenant and environment;
- residency and processing location;
- owner and steward;
- retention and legal hold;
- lineage visibility permissions;
- model/provider data handling;
- whether row samples are allowed;
- whether an output can be published to the target audience.

Apply policy before evidence retrieval as well as before effects. Unauthorized data should not enter the model context simply because the final action is denied.

### Metadata can be sensitive

Dataset names, column names, lineage edges, row counts, owner identities, and incident descriptions can reveal business or personal information. Minimize and redact metadata based on the same identity context used for the underlying catalog.

## 9. Silent corruption

Crashes are visible; silent corruption is more dangerous. Defenses include:

- end-to-end checksums and manifest validation;
- redundant decoding or validation on critical paths;
- canary and shadow outputs;
- versioned schema/contract gates;
- invariant and reconciliation checks;
- hardware/runtime corruption monitoring where relevant;
- isolation of suspect partitions;
- restore and replay exercises.

An agent should favor evidence of correctness over the absence of errors.

## 10. Lineage and quality operational SLOs

Track:

- percent of critical products with fresh run-level lineage;
- percent with column lineage where required;
- lineage event ingestion lag;
- unresolved dataset identifier rate;
- contract-validation coverage;
- quality-gate coverage and false-positive rate;
- quarantine age and unresolved volume;
- reconciliation completion and mismatch rate;
- time from breaking-change proposal to owner acknowledgement.

Alert on user-visible risk, not every missing optional facet.

## 11. Selected sources

- [OpenLineage specification](https://github.com/OpenLineage/OpenLineage/blob/main/spec/OpenLineage.md)
- [OpenLineage 1.52.0 release](https://openlineage.io/docs/releases/1_52_0/)
- [OpenLineage facets and replacement semantics](https://openlineage.io/docs/spec/facets/)
- [Google Cloud data lineage model](https://docs.cloud.google.com/dataplex/docs/about-data-lineage)
- [Google Cloud data lineage views](https://docs.cloud.google.com/dataplex/docs/lineage-views)
- [Great Expectations: define expectations](https://docs.greatexpectations.io/docs/core/define_expectations/)
- [Great Expectations: run validations](https://docs.greatexpectations.io/docs/core/run_validations/)
- [Great Expectations: retrieve all unexpected rows](https://docs.greatexpectations.io/docs/core/run_validations/retrieve_all_unexpected_rows/)
- [Deequ repository](https://github.com/awslabs/deequ)
- [Soda contract language reference](https://docs.soda.io/reference/contract-language-reference)
- [NIST Privacy Framework](https://www.nist.gov/privacy-framework)
- [GDPR consolidated text](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32016R0679)
- [Meta engineering: silent data corruption](https://engineering.fb.com/2021/02/23/data-infrastructure/silent-data-corruption/)

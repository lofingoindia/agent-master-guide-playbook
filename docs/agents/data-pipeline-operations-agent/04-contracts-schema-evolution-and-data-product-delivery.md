# Contracts, Schema Evolution, and Data-Product Delivery

## 1. Contracts are admission control

A data contract turns an implicit producer-consumer assumption into a versioned, testable interface. It should be checked at design time, deployment, ingestion, transformation, publication, and incident recovery.

A useful contract includes:

- stable product, dataset, field, and owner identifiers;
- schema with nullability, logical types, constraints, and semantics;
- compatibility policy and deprecation window;
- quality rules and severity;
- freshness, availability, retention, and support expectations;
- classification, residency, purpose, and access policy;
- source and output endpoints;
- lineage requirements;
- version and change history.

The Open Data Contract Standard provides a vendor-neutral foundation for schema, quality, SLA, server, and ownership metadata. Its prose specification is authoritative when tooling or companion JSON Schema disagrees.

## 2. Contract layers

| Layer | Example question | Enforcement |
|---|---|---|
| Syntax | Is the contract document valid? | JSON Schema or contract CLI |
| Structural schema | Are names, types, nullability, and required fields allowed? | Registry, dbt contract, table API |
| Serialization compatibility | Can old/new readers decode the data? | Avro/Protobuf/JSON Schema rules |
| Semantic compatibility | Does the field still mean the same thing? | Review, domain tests, consumer attestations |
| Data quality | Do observed values satisfy expectations? | Quality engine |
| Operational SLO | Is the product fresh and available? | SLO monitor |
| Governance | Is use lawful and within classification/residency? | Policy and access systems |

Passing one layer does not imply the next. For example, an integer type can remain wire-compatible while its unit changes from cents to dollars.

## 3. Stable identifiers

Prefer stable IDs over display names. A rename should preserve the identifier when semantics are unchanged.

Table formats and serialization systems offer different mechanisms:

- Iceberg uses field IDs and schema IDs to track evolution;
- Protobuf uses field numbers and requires removed numbers/names to be reserved;
- Avro resolves writer and reader schemas and supports aliases;
- a catalog may use a resource URI;
- the contract should map these physical identifiers to a durable product field ID.

Never reuse a deleted Protobuf field number. Never assume a database column rename will be interpreted as a rename by every downstream engine.

Keep these version domains distinct:

~~~yaml
contract_identity:
  id: urn:datacontract:commerce:orders-hourly
  semantic_version: 2.4.0
  effective_from: 2026-08-31T00:00:00Z
  content_digest: sha256:...
serialization_schema:
  registry: confluent-prod-eu
  subject: orders-value
  naming_strategy: TopicNameStrategy
  format: avro
  schema_id: 912
  subject_version: 14
  canonical_fingerprint: sha256:...
physical_dataset:
  id: iceberg://prod-eu/commerce/orders
  table_uuid: 9b...
  schema_id: 27
  partition_spec_id: 6
  snapshot_id: 89104
transform:
  manifest_digest: sha256:...
  code_commit: git:4f6c...
~~~

A contract version describes an agreement; a registry version describes serialized shape history; a table schema ID describes one physical representation; a snapshot identifies committed data. None is a safe alias for another.

## 4. Compatibility policy

### Modes

| Mode | Question |
|---|---|
| Backward | Can a new reader consume data written with the prior schema? |
| Forward | Can an old reader consume data written with the new schema? |
| Full | Are both directions supported? |
| Transitive | Is compatibility checked against every relevant historical version rather than only the latest? |

Confluent Schema Registry defaults and support are format- and configuration-specific; its commonly documented default is non-transitive backward compatibility. Record the actual subject configuration rather than assuming it.

Do not confuse core compatibility with Confluent's richer schema rules. The observed Platform documentation requires 7.4+ plus Enterprise or the Cloud Advanced governance package, and lists missing rules execution in Confluent Cloud Kafka Connect, Flink SQL, and ksqlDB. Referenced-schema and non-Java client limitations also differ. Probe the exact serializer/client path that will enforce the rule.

### Change matrix

| Change | Likely risk | Required action |
|---|---|---|
| Add optional/defaulted field | Often compatible | Format-specific registry check and consumer test |
| Add required field | Breaking for old data/readers | Default/migration or new version |
| Drop field | Consumer break and information loss | Deprecation window, usage proof, new major version |
| Rename field | Often appears as drop/add | Stable ID/alias where supported and consumer migration |
| Widen numeric type | Format-dependent | Registry plus value-boundary tests |
| Narrow numeric type | Data loss/overflow | Reject unless migration proves safety |
| Change nullability | Semantic and runtime risk | Profile historical data and coordinate readers |
| Change enum symbols | Format-dependent | Old/new reader tests |
| Change unit, time zone, or meaning | Wire-compatible but semantically breaking | New field or major contract version |
| Change key | Ordering, upsert, and dedupe break | New stream/table and coordinated migration |

Use a real encoded-data corpus for compatibility tests, not only generated schema pairs.

## 5. Change admission workflow

~~~mermaid
sequenceDiagram
    participant P as Producer
    participant CI as Contract CI
    participant R as Registry
    participant C as Consumer owners
    participant D as Deployment
    participant V as Verifier
    P->>CI: propose contract and pipeline change
    CI->>CI: syntax, policy, compatibility, fixtures
    CI->>R: check configured subject history
    CI->>C: notify affected consumers with diff
    C-->>CI: attest, migrate, or reject
    CI->>D: approve pinned artifacts
    D->>V: shadow/canary output
    V->>V: contract, quality, lineage, outcome checks
    V-->>D: publish or quarantine
~~~

For a breaking change, prefer a parallel version:

1. publish v2 to a new field, topic, table, or product endpoint;
2. dual-write or derive it during a bounded transition;
3. compare v1/v2 on production-like data;
4. migrate consumers;
5. prove v1 is unused;
6. retire it under retention and rollback policy.

Do not let an agent decide that a breaking change is safe based only on lineage completeness.

Admission is a deterministic conjunction, not a model vote:

~~~text
publish_allowed = syntax_valid
  AND configured_registry_compatibility_passes
  AND semantic_change_policy_passes
  AND required_consumer_fixtures_pass
  AND physical_reader_writer_matrix_passes
  AND quality_and_reconciliation_pass
  AND lineage_coverage_meets_policy
  AND governance_allows_target
  AND approval_is_valid_for_exact_artifact_set
~~~

An `UNKNOWN`, unavailable, or stale mandatory gate is a denial. The model can summarize why a gate failed and propose a migration sequence; it cannot convert unknown coverage into a pass.

## 6. dbt contracts and tests

dbt model contracts can preflight output names and data types. Platform support for constraints differs, and contracts do not replace data tests.

The current dbt documentation limits contracts to supported SQL materializations; they do not cover sources, snapshots, seeds, Python models, ephemeral models, or most custom materializations. Warehouse constraints may be declared but not enforced. Record the exact dbt engine and adapter release, then test whether each required constraint fails the build on that platform.

For incremental models, record and test:

- the incremental filter;
- unique key, if updates are expected;
- strategy such as append, merge, or delete-and-insert;
- late-arriving data window;
- full-refresh behavior;
- source and destination predicates;
- target platform semantics.

Without a unique key, many incremental strategies append and can duplicate rows. A syntactically accepted incremental predicate can still omit valid late data.

Microbatch execution divides an event-time range into smaller batches. It improves isolation and retry behavior only if event time, batch boundaries, late data, and output idempotency are defined. The observed dbt docs treat event-time inputs as UTC, default `lookback` to one batch, and use different replacement mechanisms by adapter; a direct parent without `event_time` can be fully scanned for every batch.

## 7. Lakehouse evolution

### Iceberg

Iceberg's field IDs, snapshots, partition evolution, sort orders, and optimistic concurrency provide useful publication primitives. The agent should reference snapshot IDs, not only table names and timestamps.

Before a commit:

- resolve the base snapshot;
- check concurrent changes;
- stage files and metadata;
- validate schema and partition specs;
- commit through the official API;
- record the resulting snapshot;
- remove abandoned staged artifacts under normal maintenance policy.

### Delta Lake

Record protocol and table-feature requirements because feature upgrades can change which readers/writers are allowed. Change Data Feed only captures changes after it is enabled and is governed by table retention/VACUUM; it is not a permanent audit log.

### Hudi

Prefer backward-compatible evolution. In the observed Hudi 1.2.0 documentation, schema-on-read is experimental and cannot be disabled after the table has accepted incompatible evolution. Some changes can write successfully yet make mixed historical files unreadable. Require a release-specific writer/reader matrix, adapter probe, and rollback analysis.

## 8. Data-product publication

Publication is a controlled state transition:

~~~text
BUILT -> CONTRACT_VALID -> QUALITY_VALID -> LINEAGE_RECORDED
      -> GOVERNANCE_VALID -> PUBLISHED -> CONSUMER_ACKNOWLEDGED
~~~

Each product version should expose:

- contract version;
- physical dataset and snapshot/version;
- producing pipeline and run;
- source frontiers or input snapshots;
- freshness timestamp;
- quality result;
- lineage event or graph reference;
- classification/access reference;
- owner and support channel;
- deprecation status.

For Open Data Product Standard-style products, input and output ports provide a useful boundary. Validate each output port against its own contract.

## 9. Example contract fragment

The following is illustrative and should be validated against the exact ODCS version used:

~~~yaml
apiVersion: v3.1.0
kind: DataContract
id: urn:datacontract:commerce:orders-hourly
version: 2.4.0
status: active
name: Orders hourly
domain: commerce
schema:
  - name: orders
    physicalType: table
    properties:
      - name: order_id
        logicalType: string
        required: true
        unique: true
        classification: confidential
      - name: event_time
        logicalType: timestamp
        required: true
quality:
  - type: sql
    description: order_id must not be null
    severity: error
slo:
  freshness:
    threshold: 90m
team:
  - role: owner
    name: commerce-data
~~~

Do not put secrets, connection credentials, or customer samples in the contract.

## 10. Agent behavior during change

| Decision | Deterministic owner | Bounded model role |
|---|---|---|
| Resolve identities and versions | Registry/catalog adapters and canonical mapping | Explain conflicts; never choose a same-name target |
| Serialization compatibility | Format/registry tooling under recorded subject policy | Explain reader/writer implications |
| Semantic compatibility | Versioned semantic rules and required owner/consumer attestations | Draft a semantic diff and questions |
| Physical reader/writer support | Tested compatibility matrix | Summarize unsupported combinations |
| Quality threshold and publication | Versioned policy and verifier | Diagnose observed failures without changing thresholds |
| Migration sequencing | Validated plan schema and policy | Propose parallel version, dual-run, or consumer order |

The agent may:

- retrieve current and proposed contracts;
- produce a normalized semantic diff;
- run deterministic compatibility tools;
- identify instrumented descendants;
- generate consumer-specific fixture results;
- propose sequencing and a canary plan;
- block publication when policy or gates fail.

It may not:

- reinterpret a registry failure as safe;
- waive a governance or consumer requirement;
- invent defaults or aliases;
- update production schemas or registry settings outside CI;
- declare no impact merely because lineage has no edge;
- publish quarantined output.

## 11. Selected sources

- [Open Data Contract Standard](https://github.com/bitol-io/open-data-contract-standard/blob/main/docs/README.md)
- [Open Data Product Standard](https://github.com/bitol-io/open-data-product-standard/blob/main/docs/README.md)
- [JSON Schema Draft 2020-12](https://json-schema.org/draft/2020-12)
- [Apache Avro 1.12.0 specification](https://avro.apache.org/docs/1.12.0/specification/)
- [Protocol Buffers language guide](https://protobuf.dev/programming-guides/proto3/)
- [Confluent Schema Registry schema evolution](https://docs.confluent.io/platform/current/schema-registry/fundamentals/schema-evolution.html)
- [Confluent Schema Registry data contracts and limitations](https://docs.confluent.io/platform/current/schema-registry/fundamentals/data-contracts.html)
- [dbt model contracts](https://docs.getdbt.com/docs/mesh/govern/model-contracts)
- [dbt incremental models](https://docs.getdbt.com/docs/build/incremental-models)
- [dbt microbatch incremental models](https://docs.getdbt.com/docs/build/incremental-microbatch)
- [Apache Iceberg specification](https://iceberg.apache.org/spec/)
- [Delta Lake protocol and table features](https://docs.delta.io/versioning/)
- [Apache Hudi schema evolution](https://hudi.apache.org/docs/schema_evolution/)

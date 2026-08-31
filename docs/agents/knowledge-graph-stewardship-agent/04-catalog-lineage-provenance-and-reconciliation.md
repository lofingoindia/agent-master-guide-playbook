# Catalog, Lineage, Provenance, and Reconciliation

Catalog synchronization is not a sequence of unqualified upserts. It is a reconciliation system that compares versioned observations with governed desired state, proposes typed changes, applies authorized effects, and proves what the target actually accepted.

## Model catalog records and data resources separately

DCAT 3 distinguishes a resource from the catalog record that describes it. Preserve that distinction internally:

```yaml
catalog_resource:
  resource_id: urn:asset:warehouse:prod:orders
  kind: dataset
  identity_keys: [platform, environment, qualified_name]
  lifecycle_state: active

catalog_record:
  record_id: urn:catalog-record:orders:137
  describes: urn:asset:warehouse:prod:orders
  source_system: warehouse-crawler
  source_revision: scan-8891
  record_version: 137
  observed_at: 2026-08-31T05:00:00Z
  provenance_ref: prov-01K...
```

The resource may persist while descriptions, distributions, versions, ownership assertions, or catalog records change. DCAT 3 provides version, series, qualified relation, and provenance concepts without requiring a particular storage engine; see the [DCAT 3 Recommendation](https://www.w3.org/TR/vocab-dcat-3/).

## Source-of-truth matrix

Declare authority per field and direction. “Catalog is the source of truth” is usually too broad.

| Field | Authoritative origin | Allowed writers | Conflict policy |
|---|---|---|---|
| Physical column name/type | Introspected platform or schema registry | Connector observations only | Newer verified source revision wins; steward cannot invent |
| Human description | Catalog stewardship workflow | Owner or delegated steward | Versioned edit with review policy |
| Runtime lineage | Instrumented job/event | Connector observations | Append observation; reconcile contradictions |
| Ownership | Governed IAM/domain registry | Authorized owner workflow | Do not overwrite from weak heuristic |
| Sensitivity class | Policy engine plus reviewed classification | Classifier proposes, designated reviewer decides | Most restrictive effective policy until resolved |
| Schema compatibility | Schema registry | Registry observation | Record result; do not reinterpret as semantic approval |
| Canonical identity | Stewardship decision ledger | Identity authority | Merge/split contract only |

Authority can vary by tenant, domain, environment, or asset type. Pin the matrix version on every case and effect.

## Canonical lineage model

Represent at least:

- **nodes:** dataset, field, job/process, run, query, report/model, and optionally code artifact;
- **edges:** input, output, column mapping, derivation, copy, rename, aggregate, filter, and lifecycle change;
- **scope:** tenant, platform, environment, namespace, and qualified name;
- **time:** event time, observation time, validity interval, and run identity;
- **evidence:** event, parser output, API response, or reviewed manual assertion;
- **confidence and status:** observed, inferred, manually asserted, disputed, invalidated, or superseded.

OpenLineage models jobs, runs, datasets, events, and extensible facets. Facet schemas are versioned; custom facets should use a project-owned namespace and immutable schema URL. Use OpenLineage as an interoperability envelope where suitable, not as the entire stewardship state model. See the [OpenLineage core model](https://openlineage.io/docs/) and [facet specification](https://openlineage.io/docs/spec/facets/).

```yaml
lineage_observation:
  observation_id: lin-01K...
  tenant_id: tenant-a
  run_id: 8df5...
  event_type: COMPLETE
  job: {namespace: prod.airflow, name: load_orders}
  inputs:
    - {namespace: s3://raw, name: orders/2026-08-31}
  outputs:
    - {namespace: snowflake://prod, name: sales.orders}
  field_edges_ref: evidence://lineage/lin-01K.../fields
  emitted_at: 2026-08-31T05:04:02Z
  received_at: 2026-08-31T05:04:05Z
  producer: {name: ol-airflow, version: 1.52.0}
  evidence_digest: sha256:...
```

Lifecycle events such as create, alter, rename, overwrite, truncate, and drop are observations about a data asset. Do not map all of them to deletion. A rename may preserve identity, create a successor, or represent a copy-and-drop depending on platform semantics and evidence.

## Provenance ledger

PROV-O provides a vocabulary for entities, activities, agents, generation, derivation, attribution, specialization, revision, and invalidation. Map these concepts into an operational ledger with immutable identifiers and retained raw evidence.

~~~mermaid
flowchart LR
    S[Source assertion] -->|wasGeneratedBy| A[Extraction activity]
    A --> O[Normalized observation]
    O -->|wasDerivedFrom| C[Candidate or validation result]
    C --> D[Curator decision]
    D --> F[Canonical fact]
    D --> E[External effect]
    E --> R[Receipt and target observation]
    R --> Q{Reconciled?}
    Q -- yes --> K[Confirmed state]
    Q -- no --> X[Dispute or recovery case]
~~~

Preserve these distinctions:

| Record | Meaning | Mutable? |
|---|---|---|
| Assertion | What a named source claimed in a particular revision | Append-only; later assertion may supersede |
| Observation | What a connector or validator observed | Append-only |
| Candidate | Machine-produced possibility with evidence and policy version | Status changes; content is immutable |
| Decision | Authorized disposition of an exact proposal | Append-only; later decision reverses/supersedes |
| Canonical fact | Effective governed view derived from decisions/assertions | Versioned, recomputable |
| Provenance | Chain tying artifact to sources, activities, agents, and versions | Append-only |
| Effect | Attempt to change an external target | State machine with immutable attempts/receipts |

PROV invalidation marks when an entity ceases to be available for use; it is not a universal delete API and does not erase history. The normative terms are in [PROV-O](https://www.w3.org/TR/prov-o/) and the [PROV namespace](https://www.w3.org/ns/prov).

## Reconciliation loop

~~~mermaid
stateDiagram-v2
    [*] --> Observed
    Observed --> Compared
    Compared --> NoChange: equivalent
    Compared --> Proposed: material delta
    Proposed --> Rejected
    Proposed --> Approved
    Approved --> Applying
    Applying --> Applied: authoritative receipt
    Applying --> Unknown: timeout or ambiguous reply
    Unknown --> Applied: target read proves result
    Unknown --> NotApplied: target read disproves result
    Applied --> Reconciled: target observation matches intent
    Applied --> Drifted: target differs
    Drifted --> Proposed
    NoChange --> [*]
    Rejected --> [*]
    Reconciled --> [*]
    NotApplied --> Approved: retry allowed
~~~

The comparison input is a consistent snapshot or explicit revision vector, not a mixture of pages from changing sources. For every source, record:

- snapshot/cursor/token semantics;
- pagination and retry behavior;
- delete/tombstone feed semantics;
- watermark and late-event allowance;
- rate limits and freshness objectives;
- API/build/version and capability manifest;
- partial-response and permission-filter behavior.

An empty lineage response may mean no edges, insufficient permission, a result limit, unsupported parsing, eventual consistency, or connector failure. Store `unknown` or a bounded completeness observation instead of “no dependency.” OpenMetadata, for example, documents parser limitations and source-specific behavior for lineage ingestion; review the deployed version's [lineage connector documentation](https://docs.open-metadata.org/v1.12.x/connectors/ingestion/lineage).

## Adapter contract

Every catalog/graph/registry adapter implements explicit capabilities:

```yaml
adapter_manifest:
  adapter: openmetadata
  adapter_version: 3.4.2
  target_api_version: 1.12
  capabilities:
    read_asset: true
    conditional_write: etag
    idempotency_key: false
    soft_delete: true
    hard_delete: true
    purge: false
    lineage_depth_limit: 3
    bulk_write: true
  ambiguity_policy:
    timeout_after_write: reconcile-before-retry
    partial_batch: receipt-per-item
```

The common interface should not pretend every system has the same semantics. A DataHub aspect update, OpenMetadata entity patch, Atlas classification propagation, and schema-registry version registration need separate effect types and target-specific preconditions.

This summary manifest is not sufficient to enable production operations. Each read, search, patch, propagation, delete, restore, register, and read-back path must pass the [operation-level qualification suite](06-state-context-memory-planning-and-tool-contracts.md#operation-level-adapter-qualification) for the exact build, edition, configuration, principal, and topology.

Current project baselines and their documented models include:

- DataHub entities, URNs, versioned aspects, and timeseries aspects in its [metadata model](https://github.com/datahub-project/datahub/blob/master/docs/modeling/metadata-model.md);
- OpenMetadata REST resources and versioned entity APIs in its [API reference](https://docs.open-metadata.org/v1.12.x/api-reference);
- Apache Atlas type/entity, glossary, and lineage endpoints in the [Atlas v2 REST API](https://atlas.apache.org/api/v2/index.html);
- Confluent Schema Registry subjects, versions, IDs, compatibility, references, and normalization in its [API documentation](https://docs.confluent.io/platform/current/schema-registry/develop/api.html).

## Conflict classification

Do not resolve every mismatch with last-write-wins.

| Conflict | Example | Default route |
|---|---|---|
| Benign representation | Equivalent normalized URI or case | Deterministic normalization with retained raw value |
| Staleness | Older crawler scan arrives late | Ignore for effective state; retain observation |
| Concurrent authorized edits | Owner and steward edit description | Three-way merge or review case |
| Authority conflict | Heuristic owner disagrees with IAM registry | Authority matrix wins; open discrepancy |
| Semantic contradiction | Two systems assert incompatible units | Quarantine or steward case |
| Identity conflict | Same source ID maps to two canonicals | Stop automatic writes; split/identity incident |
| Deletion ambiguity | Missing from scan but no tombstone | Mark unobserved/stale, not deleted |
| Access-filter conflict | API omits hidden assets | Record visibility scope; never propagate deletion |

## Delete, tombstone, invalidation, and purge

Model destructive semantics explicitly:

```text
unobserved != stale != deprecated != invalidated != soft-deleted != hard-deleted != purged
```

- **Unobserved:** absent from this bounded observation.
- **Stale:** freshness threshold exceeded.
- **Deprecated:** should not be selected for new use, but remains addressable.
- **Invalidated:** ceased to be valid/available at a recorded time.
- **Soft-deleted:** target hides it but retains recoverable state.
- **Hard-deleted:** primary target object removed; audit/history may remain.
- **Purged:** designated retained copies are irreversibly removed under policy.

Before any delete-like effect, require positive identity, target version, authority, retention/legal policy, dependency/blast-radius evidence, and a recovery plan. A full crawl missing an object is not a tombstone unless the connector contract proves complete visibility and snapshot semantics.

DataHub's stateful ingestion can mark stale entities soft-deleted when a pipeline run no longer emits them. That feature is useful only when the pipeline identity, state provider, and source scope are stable; see [stateful ingestion](https://github.com/datahub-project/datahub/blob/master/metadata-ingestion/docs/dev_guides/stateful.md).

## Correction, restriction, and deletion propagation

A source correction does not mutate prior observations. It appends a new source-record/assertion version, closes the old valid interval when known, and emits a causally linked change event. A propagation planner then computes affected artifacts from **confirmed** derivation and disclosure edges; incomplete lineage produces an explicit `unknown_scope` work item, not false closure.

```yaml
propagation_case:
  propagation_id: propg-01K...
  cause:
    kind: source_correction
    record_ref: crm/customer/442
    old_revision: "18"
    new_revision: "19"
    valid_effect: {from: 2025-02-01T00:00:00Z, to: null}
  authority: privacy-case-881
  lineage_snapshot: lineage-rev-901
  coverage: {known_recipients: 14, unknown_partitions: 2}
  actions:
    - {target: assertion-ledger, mode: supersede, status: verified}
    - {target: canonical-customer, mode: recompute, status: pending}
    - {target: vector-customer-v7, mode: delete_and_reembed, status: pending}
    - {target: partner-export, mode: notify_recipient, status: awaiting_ack}
  retained_under_hold:
    - {artifact_ref: audit/decision/77, basis_ref: hold-19, minimized: true}
  completion_rule: all_required_actions_verified_and_unknowns_disposed
```

Propagation rules:

1. Freeze new high-risk decisions derived from the affected assertion.
2. Append the correction/restriction/delete instruction with authority, scope, valid time, transaction time, and legal/retention basis.
3. Traverse the pinned lineage/provenance snapshot with relation-specific depth and tenant controls; include exports, caches, prompts where retained, vectors, evaluation corpora, backups, and model-provider copies governed by contract.
4. Classify every descendant as recomputable, correctable, restrictable, deletable, notification-only, lawfully retained, unreachable, or unknown.
5. Emit one idempotent invalidation/correction command per target and keep per-recipient receipts.
6. Recompute canonical facts from surviving assertions; never copy the previous canonical value into a new fact without evidence.
7. Rebuild projections from the new authoritative watermark and verify absence or corrected content using target-native reads.
8. Close only after required receipts, policy exceptions, holds, and unresolved coverage are visible to the owner.

For personal data subject to the EU GDPR, Article 19 requires communication of applicable rectification, erasure, or restriction to recipients unless an exception applies. That obligation is jurisdiction- and fact-dependent; the system should inventory and prove notifications, not decide legal applicability ([GDPR Articles 16–19](https://eur-lex.europa.eu/eli/reg/2016/679/oj/eng)).

### Downstream invalidation contract

```yaml
invalidation_event:
  event_id: inv-01K...
  cause_ref: propg-01K...
  tenant_id: tenant-a
  subject_ref: urn:party:master:991
  affected_record_versions: [fact-77/v4, edge-18/v2]
  reason: corrected_source_assertion
  required_action: recompute_from_assertions
  not_before_watermark: 19004
  deadline: 2026-09-01T12:00:00Z
  idempotency_key: sha256:...
  security_label: restricted-pii
  schema_version: 1
```

Consumers acknowledge `accepted`, `rebuilt`, `not_applicable`, `retained_under_hold`, or `failed`; only `rebuilt`, authorized `not_applicable`, or governed retention can satisfy the case. Silence is not success. If consumers cannot acknowledge, reconcile their observable state or keep the propagation case incomplete.

## Idempotency and uncertain effects

Construct the effect idempotency key from the immutable proposal, action type, target identity, target precondition/version, and policy version. Store it before calling the target.

```yaml
effect:
  effect_id: eff-01K...
  idempotency_key: sha256(proposal_digest|action|target|precondition|policy)
  proposal_digest: sha256:...
  action: openmetadata.patch-description
  target: urn:asset:warehouse:prod:orders
  precondition: {entity_version: 136}
  desired_digest: sha256:...
  state: prepared
```

On timeout after a write, move to `unknown`; query by target state or request identifier. Retry only when reconciliation proves not applied or the target guarantees safe replay. A locally generated idempotency key cannot manufacture server-side idempotency.

For batches, record a receipt per item. “HTTP 200 for batch” is insufficient if individual records can fail or be filtered.

## Drift scans and anti-entropy

Incremental events provide freshness; periodic snapshot comparison provides completeness. Use both:

1. consume source changes with deduplication and watermarks;
2. reconcile effects promptly;
3. sample high-risk assets more frequently;
4. perform bounded anti-entropy scans by partition;
5. compare raw observations, effective facts, and external target state;
6. open cases for unexplained drift rather than overwriting it.

Partition by tenant, environment, platform, namespace, or stable asset hash. Pin scan start/end watermarks and report coverage, permission scope, pages, failures, and skipped partitions.

Lineage confidence must never hide coverage. Report at least producer coverage, observed time window, supported dialect/operation set, parse success, unresolved identifiers, permission scope, traversal truncation, and event lateness. OpenMetadata 1.12 documents that lineage coverage is connector-specific, commonly depends on query logs and entity search, and applies configurable result limits; its published pipeline also uses parser fallbacks with different accuracy. Those facts require per-edge evidence class and per-scan coverage, not a single catalog-wide “lineage complete” flag ([lineage ingestion](https://docs.open-metadata.org/v1.12.x/connectors/ingestion/lineage), [technical architecture](https://docs.open-metadata.org/v1.12.x/developers/contribute/codebase-deep-dives/lineage-ingestion)).

## Failure matrix

| Failure | Safe behavior | Evidence to retain |
|---|---|---|
| Duplicate event | Deduplicate by producer/run/event identity; detect conflicting duplicates | Both payload digests and producer metadata |
| Out-of-order lifecycle events | Evaluate event and validity time; do not regress effective state blindly | Ordering/watermark decision |
| Partial catalog outage | Pause writes; serve last confirmed state with staleness marker | Failed calls, scope, retry budget |
| Pagination changes mid-scan | Abort or restart consistent scan | Tokens, source revision, page digests |
| Permission scope shrinks | Treat omissions as unknown | Principal and visibility/capability snapshot |
| Target write times out | Mark unknown and reconcile | Request, timing, target readback |
| Classification propagates unexpectedly | Stop propagation/canary; enumerate affected assets | Exact effect and resulting target versions |
| Schema ID/subject confusion | Quarantine mapping | Subject, version, ID, references, context |

## Checklist

- [ ] Resource identity and catalog-record identity are separate.
- [ ] Field-level authority and synchronization direction are explicit.
- [ ] Assertions, observations, candidates, decisions, facts, provenance, and effects remain distinct.
- [ ] Lineage completeness and visibility scope are recorded; absence is not overclaimed.
- [ ] Every adapter advertises versioned, tested capabilities and ambiguity semantics.
- [ ] Delete-like operations use typed lifecycle states and require stronger approval.
- [ ] Corrections/restrictions/deletions enumerate known descendants, unknown coverage, holds, and recipient receipts.
- [ ] Downstream invalidations bind affected record versions, minimum watermark, deadline, and terminal acknowledgement.
- [ ] Effect idempotency, conditional writes, receipts, and reconciliation are implemented.
- [ ] Incremental ingestion is backed by bounded anti-entropy scans.

## Related guidance

- [Ontology, schema, and vocabulary governance](03-ontology-schema-and-vocabulary-governance.md)
- [Steward cases and quality repair](05-steward-cases-quality-repair-and-merge-split-workflows.md)
- [Reliability, observability, scaling, and operations](08-reliability-observability-scaling-and-operations.md)
- [Idempotency and side effects](../../reliability/idempotency-and-side-effects.md)

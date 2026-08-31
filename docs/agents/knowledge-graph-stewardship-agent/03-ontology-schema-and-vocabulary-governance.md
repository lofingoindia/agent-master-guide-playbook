# Ontology, Schema, and Vocabulary Governance

An ontology release is an executable governance change. It can alter inference, validation, search, mappings, UI behavior, downstream contracts, and the meaning of stored facts. Treat it like a production schema release, not a prose edit.

This guide covers RDF/OWL ontologies, SHACL shapes, SKOS concept schemes, catalog schemas, and event/data-contract schemas. They overlap, but they are not interchangeable.

## Keep the semantic layers separate

| Layer | Primary question | Suitable mechanism | It must not silently decide |
|---|---|---|---|
| Conceptual model | What kinds of things and relationships exist? | OWL/RDFS or an equivalent domain model | Source-record validity |
| Validation model | Does this graph satisfy an explicit data contract? | SHACL or deterministic application validation | Ontological truth |
| Controlled vocabulary | Which governed concepts, labels, and mappings are allowed? | SKOS or a catalog vocabulary model | Entity identity |
| Event/schema contract | Can producers and consumers exchange this representation? | Avro, JSON Schema, Protobuf, or registry-specific rules | Business-semantic compatibility |
| Physical metadata | What tables, columns, jobs, dashboards, and files exist? | Catalog type system | Real-world entity equivalence |
| Policy model | Who may propose, approve, publish, or deprecate a change? | Workflow and authorization policy | Facts about the modeled domain |

OWL uses an open-world model: missing information is not normally false. SHACL checks a supplied data graph against supplied shapes and returns a validation report. That makes SHACL useful for bounded acceptance checks, but a conforming report is not proof that the graph is complete or that every statement is true. The normative baselines are the [OWL 2 Primer](https://www.w3.org/TR/owl2-primer/) and [SHACL Recommendation](https://www.w3.org/TR/shacl/).

## Govern a versioned semantic bundle

A release should pin every artifact whose combination was tested:

```yaml
semantic_bundle:
  bundle_id: customer-domain-2026.09.0
  ontology:
    ontology_iri: https://example.org/customer
    version_iri: https://example.org/customer/2026.09.0
    digest: sha256:...
  shapes:
    artifact_uri: urn:shapes:customer:2026.09.0
    digest: sha256:...
  vocabularies:
    - scheme_uri: urn:vocab:customer-status
      version: 2026.09.0
      digest: sha256:...
  mappings:
    artifact_uri: urn:mappings:customer:2026.09.0
    digest: sha256:...
  registry_contracts:
    - subject: customer-value
      version: 18
      schema_id: 741
  inference_profile: owl2-rl-bounded-v3
  validation_profile: shacl-core-v4
  policy_version: semantic-release-policy-v7
```

Never identify a release only by a mutable URL or `latest`. OWL provides ontology and version IRIs plus annotations such as `owl:priorVersion` and `owl:deprecated`; use them deliberately, while keeping the deployable bundle and its cryptographic digest in the stewardship ledger. See [OWL 2 structural specification](https://www.w3.org/TR/owl2-syntax/).

## Change workflow

~~~mermaid
flowchart LR
    I[Issue or source change] --> P[Typed proposal]
    P --> D[Dependency and blast-radius analysis]
    D --> T[Reasoner, SHACL, competency, and migration tests]
    T --> R{Independent review}
    R -- reject --> X[Rejected decision with rationale]
    R -- revise --> P
    R -- approve exact digest --> C[Canary or shadow release]
    C --> O{Observed gates pass?}
    O -- no --> B[Stop or roll back]
    O -- yes --> U[Publish bundle and migration]
    U --> M[Monitor and reconcile]
~~~

Every proposal records:

- the precise before/after artifacts and digests;
- intended semantics in plain language;
- affected classes, properties, shapes, concepts, mappings, queries, and consumers;
- whether the change is additive, restrictive, inferential, representational, or destructive;
- migration and reversal strategy;
- evidence, competency questions, and reviewer requirements;
- expected changes in inference and validation counts.

The model may draft the rationale or suggest impacted assets. Deterministic parsers, reasoners, validators, dependency indexes, and tests produce the evidence used for approval.

## Compatibility is multidimensional

“Backward compatible” is incomplete without naming the layer and the consumer.

| Dimension | Example failure despite another dimension passing |
|---|---|
| Serialization | An Avro schema accepts old records, but a renamed business concept changes interpretation |
| Validation | A new optional property passes SHACL, but a consumer rejects it |
| Inference | Adding a disjointness or property-chain axiom creates unexpected inconsistencies or derived links |
| Query | A class hierarchy change changes SPARQL result sets without changing stored triples |
| Vocabulary | Replacing a concept changes aggregations even when its code remains a string |
| Operational | A reasoner-safe change exceeds latency or memory budgets on production graph shape |
| Policy | A visible term exposes a restricted concept to a tenant that cannot see its definition |

Confluent Schema Registry compatibility modes check representation evolution according to their configured scope and format. Normalization can reduce syntactic differences, but it does not establish business equivalence. Treat registry results as one observation in the release case, not as the release decision. See the [Schema Registry overview](https://docs.confluent.io/platform/current/schema-registry/index.html) and [schema evolution guidance](https://docs.confluent.io/platform/current/schema-registry/fundamentals/schema-evolution.html).

## OWL profile and reasoning policy

Select an inference profile from measured workload needs:

| Option | Appropriate use | Principal risk |
|---|---|---|
| No materialized inference | Catalog facts and explicit relationships are sufficient | Queries must encode more domain knowledge |
| Bounded rules / OWL 2 RL subset | Predictable rule-oriented inference at scale | Unsupported constructs can be misunderstood or ignored |
| OWL 2 EL-oriented | Very large class/property taxonomies | Less expressive constraints |
| Rich DL reasoning | Small, carefully governed models that need it | Computational cost and operational unpredictability |

Publish the supported construct list. Reject, quarantine, or explicitly downgrade unsupported constructs; never let a store silently accept syntax that the deployed reasoner does not implement. OWL 2 defines profiles with different expressivity and computational properties in [OWL 2 Profiles](https://www.w3.org/TR/owl2-profiles/).

For every ontology change, test:

1. parsing and identifier resolution;
2. consistency under the selected profile;
3. expected and forbidden entailments;
4. query-result deltas on a representative snapshot;
5. inference expansion, runtime, and memory ceilings;
6. SHACL results before and after inference, where both modes are used;
7. migration and rollback behavior.

Pin whether validation sees asserted data only or the inferred closure. The two answers are different products.

## SHACL validation and repair

A validation result is an observation, not a repair instruction:

```yaml
validation_observation:
  observation_id: val-01K...
  data_snapshot: graph-snapshot-882
  shapes_digest: sha256:...
  entailment_mode: asserted-plus-bounded-rl
  conforms: false
  result_count: 31
  report_uri: evidence://validation/val-01K...
  validator_build: steward-shacl-4.2.1
  observed_at: 2026-08-31T10:15:00Z
```

Do not infer a unique repair from a violation. A missing cardinality-one value might mean the source is incomplete, the mapping is wrong, the entity was merged incorrectly, or the shape is too strict. Create a typed repair proposal with evidence and a separately approvable effect.

Use closed shapes narrowly. A closed shape can reject harmless extension properties and impede federated data; it is appropriate at a controlled ingestion boundary, not automatically across an open knowledge graph.

As of the research date, SHACL 1.0 is the W3C Recommendation. SHACL 1.2 Core, SPARQL Extensions, and Rules are Working Drafts. Experimental 1.2 features require a feature flag, pinned implementation, fallback, and conformance tests; see the [SHACL 1.2 Core status](https://www.w3.org/TR/shacl12-core/).

## SKOS vocabulary governance

For each concept scheme, govern:

- persistent concept IRIs independent of preferred labels;
- language-tagged preferred, alternative, and hidden labels;
- definition, scope note, examples, ownership, status, and validity interval;
- broader/narrower and associative relations;
- mapping provenance and confidence;
- deprecation and replacement without IRI reuse.

Mapping predicates carry different commitments. `skos:exactMatch` is transitive; `skos:closeMatch` is not. Neither should be promoted automatically to `owl:sameAs` or an MDM merge. The [SKOS Reference](https://www.w3.org/TR/skos-reference/) defines the mapping properties and integrity conditions.

A vocabulary mapping proposal should therefore state its purpose:

```yaml
mapping_proposal:
  subject: urn:concept:internal:active-customer
  predicate: http://www.w3.org/2004/02/skos/core#closeMatch
  object: https://partner.example/concept/current-customer
  intended_use: search-expansion-only
  prohibited_uses: [entity-merge, regulatory-reporting]
  evidence_refs: [ev-901, ev-902]
  confidence: 0.88
  valid_during: 2026-Q3
```

## Schema and ontology are coupled through explicit mappings

Keep source-field mappings versioned and reversible:

```text
source column/version
    -> extraction observation
    -> mapping rule/version
    -> canonical predicate/class
    -> transformation provenance
    -> validation observation
    -> accepted assertion or quarantine
```

Do not bake physical column names into the ontology merely because they are convenient. Conversely, do not hide loss of precision behind an elegant ontology. Record units, code systems, temporal semantics, null/unknown conventions, and transformations.

For RDF datasets, a named graph does not automatically mean “the source,” “the author,” or “the provenance.” RDF deliberately leaves the relationship between graph name and graph content open. Define the project convention and also emit explicit provenance records; see [RDF 1.1 Datasets](https://www.w3.org/TR/rdf11-datasets/).

## JSON-LD and interchange boundaries

JSON-LD is an RDF serialization and processing model, not a schema validator or an authorization envelope. Two compact JSON documents can expand differently when their active contexts differ. A production ingest therefore records:

- raw document bytes and media type;
- base IRI, processing mode, expansion options, and processor build;
- every local or remote context IRI, resolved bytes, digest, retrieval time, redirect chain, and rights policy;
- expanded/canonicalized artifact digest where canonicalization is actually required and qualified;
- warnings for relative IRIs, blank nodes, protected terms, and unknown/overridden terms.

Do not resolve arbitrary remote contexts during a privileged run. The JSON-LD 1.1 Recommendation notes that remote-context retrieval creates privacy and man-in-the-middle risks and recommends caching or a controlled document loader. Use an allowlisted loader, pin immutable context content, cap recursion/bytes/time, reject redirect-to-private-network behavior, and re-run semantic deltas when a context changes ([JSON-LD 1.1](https://www.w3.org/TR/json-ld11/)). A context digest change is a schema/semantic event even when the compact document bytes are unchanged.

## Data contracts and consumer evidence

A data contract is a versioned agreement among a producer, representation, semantics, quality policy, and named consumers. Keep these gates separate:

| Gate | Evidence | Failure disposition |
|---|---|---|
| Syntax and parse | Pinned Avro/JSON Schema/Protobuf/RDF parser | Reject or quarantine malformed artifact |
| Registry compatibility | Subject/context, version/ID/references, configured compatibility and normalization | Block registration or record the exact compatible scope |
| Semantic mapping | Units, code systems, null/unknown meaning, identity and temporal semantics | Owner review or migration proposal |
| Consumer contract | Recorded consumer versions plus executable read/query tests | Do not publish while a required consumer fails |
| Operational budget | Serialization, reasoning, validation, query, and migration resource tests | Canary/partition/reshape or reject |
| Rights and policy | Permitted uses, residency, classification, retention, and deprecation | Deny distribution or constrain the release |

Confluent's API distinguishes a globally assigned schema ID from the version under a subject; an identical schema may reuse an ID/current version, and `latest` can change immediately after a compatibility request. Bind approval to context, subject, explicit base version set, schema/reference digests, compatibility configuration, normalization flag, and consumer test set—not `latest` ([Schema Registry API](https://docs.confluent.io/platform/current/schema-registry/develop/api.html)).

## Semantic invalidation plan

Each change class declares which derivatives become stale:

| Change | Minimum invalidation set |
|---|---|
| Label/definition only | Lexical index, rendered documentation, translation checks |
| IRI/alias/deprecation | Mappings, redirects, references, search, consumer caches |
| Class/property hierarchy | Entailments, competency queries, authorization/classification consequences |
| Domain/range/disjointness/cardinality | Consistency results, SHACL results if coupled, affected candidates and canonical facts |
| Shape/rule | Validation observations, exceptions, repair cases, enforcement metrics |
| JSON-LD context | Expanded RDF, mappings, digests, validation, downstream consumers |
| Registry schema/reference | Compatibility evidence, generated clients, consumers, lineage/schema projections |

Invalidation marks a derivative unusable under the new bundle; it does not delete the historical result. Emit an invalidation event with old/new bundle digests, affected scope, reason, rebuild owner, checkpoint, and deadline. Publication is not reconciled until required rebuilds and consumer acknowledgements reach their declared terminal state.

## Release, migration, and rollback

Choose one migration mode per change:

- **read old/write old:** observation before deployment;
- **dual read/write:** compare new and old mappings or vocabularies;
- **read both/write new:** cutover with compatibility adapter;
- **backfill:** versioned transformation with checkpoint and receipts;
- **rebuild:** derived closure or search index regenerated from canonical assertions;
- **rollback:** restore traffic to previous bundle and reverse only reversible effects.

Never “roll back” by deleting unexplained facts. If published identifiers or concepts were observed externally, deprecate or supersede them and preserve history. Rebuild derived facts rather than treating them as source assertions.

Release gates should include:

- no unintended identifier reuse;
- no unreviewed destructive or meaning-changing changes;
- expected compatibility results for every registered consumer class;
- competency and negative tests pass;
- inference/validation deltas are explained;
- migration is idempotent and resumable;
- rollback or forward-fix is rehearsed;
- affected owners and high-risk consumers approve the exact bundle digest.

## Common failure modes

| Failure | Detection | Response |
|---|---|---|
| Ontology “cleanup” changes query meaning | Golden-query and entailment delta | Stop release; revise or publish migration |
| Shape change quarantines a large source | Validation-rate canary by source | Freeze enforcement; retain evidence; investigate mapping vs shape |
| Exact mapping creates identity collapse | Mapping-to-merge canary and cluster metrics | Remove derived promotion; reopen affected cases |
| Registry says compatible but consumer breaks | Consumer-contract tests and canary telemetry | Roll back producer routing; repair semantic contract |
| Reasoner explosion | Closure-size and latency budget | Disable new bundle; use bounded profile or explicit rules |
| Deprecated term disappears from history | Reference-integrity check | Restore persistent IRI; use replacement/deprecation metadata |
| Draft standard behavior drifts | Capability-manifest mismatch | Quarantine feature; pin implementation; migrate deliberately |

## Checklist

- [ ] Ontology, shapes, vocabularies, mappings, and registry contracts are separate, versioned artifacts.
- [ ] Compatibility is evaluated at representation, semantic, query, inference, operational, and policy layers.
- [ ] Supported reasoning profile and unsupported constructs are explicit.
- [ ] SHACL results are observations; repair remains a proposal and effect.
- [ ] SKOS mappings never silently become entity identity.
- [ ] JSON-LD processing pins context bytes, processor/options, rights, and remote-loading policy.
- [ ] Registry compatibility, semantic mapping, consumer, operational, and rights gates remain separate.
- [ ] Every semantic change declares invalidated derivatives, rebuild owners, and consumer acknowledgements.
- [ ] Every release has competency, negative, scale, migration, and rollback tests.
- [ ] Approvals bind to the exact bundle digest and target environment.
- [ ] Draft standards or prerelease vendor features are feature-flagged and replaceable.

## Related guidance

- [Catalog, lineage, provenance, and reconciliation](04-catalog-lineage-provenance-and-reconciliation.md)
- [Steward cases and quality repair](05-steward-cases-quality-repair-and-merge-split-workflows.md)
- [Evaluation and staged delivery](09-evaluation-failure-injection-and-staged-delivery.md)
- [Tool contracts](../../tools/tool-contracts.md)

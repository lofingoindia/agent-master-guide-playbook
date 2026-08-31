# Research Packet: Knowledge Graph and Data-Catalog Stewardship Agent Blueprint

> **Status:** Research-backed design evidence for the blueprint in [`docs/agents/knowledge-graph-stewardship-agent/`](../../agents/knowledge-graph-stewardship-agent/README.md)  
> **Research date:** 2026-08-31  
> **Scope:** Entity resolution, master-data merge/split, RDF/OWL/SHACL/SKOS governance, catalogs, lineage, provenance, schema registries, data-quality repair, target reconciliation, security, operations, and evaluation.  
> **Method:** Normative standards and primary project/vendor documentation formed the baseline; foundational and current primary research supplied identity-matching and knowledge-graph evidence. Product features were compared as mechanisms, not treated as end-to-end guarantees.  
> **Version note:** Stable/released features were separated from Working Drafts, prereleases, and community specifications. All volatile baselines need revalidation before implementation or refresh.
> **Access note:** All linked sources were accessed or rechecked on 2026-08-31 unless a source entry says otherwise. “Current” documentation is a moving target and must be paired with the recorded product/build baseline below.

## Research questions

1. What is the smallest production architecture that safely turns catalog, graph, MDM, lineage, registry, and quality evidence into governed proposals and effects?
2. Which records must remain distinct so the system can explain and recover from bad merges, semantic releases, drift, and ambiguous writes?
3. Where should deterministic rules, probabilistic matchers, graph methods, embeddings, language models, and human judgment each be used?
4. How do W3C semantic-web standards interact with closed-world validation, source authority, target product semantics, and operational constraints?
5. What do current DataHub, OpenMetadata, Apache Atlas, OpenLineage, Schema Registry, and MDM mechanisms provide—and what must remain application-owned?
6. Which identity and graph evaluation practices survive unseen entities, temporal change, blocking loss, cluster errors, and contaminated graphs?
7. How should the blueprint stage authority from deterministic reports to controlled effects and continuous improvement without granting implicit autonomy?

## Executive findings

1. **A stewardship agent is a governed reconciliation system, not a graph chatbot.** The durable loop is observe → compare → propose → decide → prepare → apply → reconcile. The model helps only with bounded ambiguity.
2. **Assertion, observation, candidate, decision, canonical fact, provenance, and external effect are different records.** Collapsing them makes source claims look curated, scores look true, approvals look applied, and timeouts look failed.
3. **Pair matching and clustering are separate products.** Transitive closure over locally plausible pairs can create catastrophic false clusters. Split/unlink and retained source assertions are first-class requirements.
4. **Embeddings and graph similarity are candidate generators, not identity authority.** The same is true of `owl:sameAs` learned from similarity and SKOS mappings promoted into master-data merges.
5. **OWL inference, SHACL validation, and schema-registry compatibility answer different questions.** Open-world entailment, bounded data validation, and serialization evolution cannot substitute for one another.
6. **Catalog and lineage APIs provide partial, permission-scoped observations.** Absence is not proof of no dependency or deletion. Reconciliation needs completeness and visibility metadata.
7. **External writes require an `unknown` state.** A durable workflow or client idempotency key cannot prove whether a target applied a timed-out request. Readback/reconciliation is mandatory.
8. **The prompt is a lossy view, not memory or state.** Case, decision, effect, and evidence ledgers survive compaction and worker loss; indexes and summaries remain rebuildable.
9. **Security labels must cover inferred edges, candidates, embeddings, explanations, traces, and evaluation artifacts.** Controlling only source rows misses the new information created by linkage and inference.
10. **Stage 0 is a legitimate final design.** Exact identifiers, deterministic catalog ingestion, data-quality rules, and human workflow are often safer and cheaper than an agent. Model capability must demonstrate incremental value by slice.

## Reference architecture derived from the research

~~~mermaid
flowchart LR
    S[Sources: catalog, graph, MDM, registry, lineage, DQ] --> I[Versioned adapters]
    I --> A[Assertions and observations]
    A --> N[Deterministic normalize, validate, block]
    N --> C[Candidates and discrepancies]
    C --> X[Evidence/context compiler]
    X --> P[Bounded proposer]
    P --> W[Durable steward case]
    W --> H{Authority and approval}
    H -- reject/defer --> W
    H -- approve exact digest --> E[Isolated effect executor]
    E --> T[External target]
    T --> R[Receipt and readback]
    R --> Q[Reconciliation]
    Q --> F[Canonical projection]
    A --> F
    D[Decision and provenance ledgers] --- W
    D --- E
    M[Policy, semantic, and release manifests] --- N
    M --- X
    M --- H
~~~

The architecture is intentionally conventional. It adds a model only between deterministic evidence compilation and a typed proposal. It keeps target credentials and mutation adapters outside the reasoning process.

## Normative semantic-web findings

### RDF datasets and provenance

RDF datasets contain a default graph and zero or more named graphs, but RDF 1.1 does not prescribe what a graph name denotes or how it relates to graph content. A named graph therefore does not inherently prove source, author, tenant, validity, or provenance. The blueprint requires an explicit named-graph convention plus PROV/application records. Source: [RDF 1.1 Datasets](https://www.w3.org/TR/rdf11-datasets/).

As of the research date, RDF 1.2 Concepts and RDF 1.2 Semantics are Candidate Recommendation Snapshots dated 7 April 2026, while SPARQL 1.2 Query/Update/Protocol remain Working Drafts. Candidate Recommendation is not W3C Recommendation status and does not prove deployed interoperability. New features remain capability-gated and pinned to tested implementations. Sources: [RDF 1.2 publication history](https://www.w3.org/standards/history/rdf12-concepts/), [RDF 1.2 Semantics history](https://www.w3.org/standards/history/rdf12-semantics/), and [SPARQL 1.2 Query status](https://www.w3.org/TR/sparql12-query/).

### OWL

OWL uses open-world semantics; absence normally does not establish falsity. Rich axioms can create new entailments without modifying stored assertions. Ontology IRI, optional version IRI, imports, prior-version, incompatibility, and deprecation annotations provide useful vocabulary for version governance but do not themselves deploy or migrate a system. Sources: [OWL 2 Primer](https://www.w3.org/TR/owl2-primer/), [OWL 2 structural specification](https://www.w3.org/TR/owl2-syntax/), and [OWL 2 Profiles](https://www.w3.org/TR/owl2-profiles/).

Production conclusions:

- pin the supported OWL/profile/construct subset and reasoner build;
- distinguish stored assertions from derived statements;
- test positive and forbidden entailments, query deltas, closure size, runtime, and memory;
- rebuild derived closure rather than treating it as source evidence;
- do not use `owl:sameAs` as a convenient fuzzy-match edge.

### SHACL

SHACL takes a data graph and shapes graph and produces a validation report. It is suited to explicit bounded validation but does not determine the true repair or prove completeness. Closed shapes can be appropriate at controlled boundaries and hostile to open extension elsewhere. Source: [SHACL Recommendation](https://www.w3.org/TR/shacl/).

SHACL 1.2 Core, SPARQL Extensions, and Rules were Working Drafts on the research date. The blueprint treats their additional behavior as experimental until a stable Recommendation and deployed conformance are verified. Sources: [SHACL 1.2 Core](https://www.w3.org/TR/shacl12-core/) and [SHACL 1.2 SPARQL Extensions](https://www.w3.org/TR/shacl12-sparql/).

Production conclusions:

- a validation result is an immutable observation with data/shape/version/entailment context;
- repair is a separate proposal against the causal layer: source, mapping, identity, rule, or exception;
- pin whether validation runs on asserted data or inferred closure;
- do not automatically rewrite data from a validation message.

### SKOS

SKOS separates semantic relations within a scheme from mapping relations across schemes. `skos:exactMatch` is transitive, while `skos:closeMatch` is not; preferred-label integrity and mapping semantics are stronger than casual string similarity. Source: [SKOS Reference](https://www.w3.org/TR/skos-reference/).

Production conclusions:

- concept IRIs persist independently of labels;
- mappings record intended use, provenance, validity, and prohibited promotions;
- SKOS mapping never automatically becomes MDM identity or `owl:sameAs`;
- vocabulary replacement/deprecation preserves history and references.

### DCAT and PROV

DCAT 3 distinguishes cataloged resources from catalog records and supports resource series, versions, qualified relations, and provenance. That separation maps directly to the blueprint's resource identity versus observed record versions. Source: [DCAT 3](https://www.w3.org/TR/vocab-dcat-3/).

PROV-O provides entities, activities, agents, generation, derivation, attribution, specialization, revision, and invalidation. Invalidation marks that an entity ceased to be available for use at a time; it does not require erasing history and is not synonymous with a target product's delete/purge. Sources: [PROV-O](https://www.w3.org/TR/prov-o/) and [PROV namespace](https://www.w3.org/ns/prov).

### Reconciliation protocols

The W3C Entity Reconciliation API v0.2 is a Community Group Final Report, not a W3C Recommendation. OpenRefine documents a compatible practical API shape for query/candidate workflows. These are useful interoperability references for candidate generation, but neither provides the stewardship decision, cluster, approval, or target-effect ledger required here. Sources: [W3C Reconciliation API v0.2](https://www.w3.org/community/reports/reconciliation/CG-FINAL-specs-0.2-20230410/) and [OpenRefine Reconciliation API](https://openrefine.org/docs/technical-reference/reconciliation-api).

## Current project and product baselines

The following versions were the latest stable/released baselines identified on 2026-08-31. Verify again before implementation.

| Project/product | Researched baseline | Useful mechanism | Application-owned gap |
|---|---:|---|---|
| OpenLineage | 1.52.0 | Job/run/dataset events, standard and custom facets, lifecycle observations | Source authority, case decisions, completeness, effect reconciliation |
| DataHub | 1.7.0 | Entities/URNs/aspects, versioned/timeseries aspects, ingestion state, change proposals/authorization | Cross-system authority, identity split semantics, approval/effect ledger |
| OpenMetadata | 1.12.8 stable; 1.13 RC excluded | Versioned entity APIs, governance workflow, glossary, lineage, data quality, RBAC/ABAC | Full visibility semantics, external idempotency, application case/recovery policy |
| Apache Atlas | 2.5.0 | Type/entity/glossary/lineage APIs and classification propagation | Uniform delete/propagation semantics, target-independent reconciliation |
| Neo4j | 2026.07.1 current documentation | ACID property-graph transactions, constraints, Cypher traversal, CDC and semantic indexes where edition/configuration supports them | RDF/OWL semantics, application bitemporal/approval ledger, portable idempotency/version preconditions, authoritative vector identity |
| Confluent Schema Registry | current Platform documentation | Subjects/versions/IDs/references, compatibility, normalization, data-contract rules | Business-semantic/consumer correctness and ontology approval |
| Apicurio Registry | 3.3.x docs | Artifact rules and compatibility modes | Rule maturity varies by artifact; same semantic gaps remain |
| Splink | 4.0.16 stable; 5.0 dev excluded | Probabilistic linkage, blocking and performance guidance | Source authority, human decision, merge/split/effects |

Primary release/status sources:

- [OpenLineage releases](https://github.com/OpenLineage/OpenLineage/releases) and [core model/facets](https://openlineage.io/docs/spec/facets/)
- [DataHub releases](https://github.com/datahub-project/datahub/releases) and [metadata model](https://github.com/datahub-project/datahub/blob/master/docs/modeling/metadata-model.md)
- [OpenMetadata releases](https://github.com/open-metadata/OpenMetadata/releases), [API reference](https://docs.open-metadata.org/v1.12.x/api-reference), and [security policy](https://github.com/open-metadata/OpenMetadata/security)
- [Apache Atlas distribution](https://downloads.apache.org/atlas/) and [v2 REST API](https://atlas.apache.org/api/v2/index.html)
- [Neo4j 2026.07.1 Operations Manual](https://neo4j.com/docs/operations-manual/current/), [transaction behavior](https://neo4j.com/docs/operations-manual/current/database-internals/), and [semantic indexes](https://neo4j.com/docs/cypher-manual/current/indexes/semantic-indexes/)
- [Schema Registry overview](https://docs.confluent.io/platform/current/schema-registry/index.html), [API](https://docs.confluent.io/platform/current/schema-registry/develop/api.html), and [data contracts](https://docs.confluent.io/platform/current/schema-registry/fundamentals/data-contracts.html)
- [Apicurio compatibility rules](https://www.apicur.io/registry/docs/apicurio-registry/3.3.x/getting-started/assembly-registry-compatibility-modes.html)
- [Splink releases](https://github.com/moj-analytical-services/splink/releases)

## Pass 2 targeted findings

### Bitemporal records are necessary for explainable correction

IBM Db2's current documentation distinguishes application/business time from system/recorded time and defines bitemporal tables as their combination. Its half-open business-time semantics and ability to query both dimensions validate the blueprint's separation between “when the claim applies” and “when the steward ledger knew it.” Source: [Db2 bitemporal tables](https://www.ibm.com/docs/en/db2/12.1.x?topic=tables-bitemporal) and [business-time period](https://www.ibm.com/docs/en/db2/11.5.x?topic=tables-business-time-period).

Blueprint impact:

- use half-open valid and transaction intervals in every durable record envelope;
- preserve late corrections by appending a new transaction-time version rather than backdating history;
- ask historical questions using both domain time and knowledge/recording time;
- allow overlapping conflicting source assertions where reality is disputed, but enforce non-overlap where a canonical single-valued policy requires it.

### Protocol names do not supply concurrency or idempotency

SPARQL 1.2 Update says requests should be atomic, leaves concurrency to each implementation, and warns that `SERVICE` commonly loses atomicity. HTTP defines PUT/DELETE as idempotent in intended effect and `If-Match` as a lost-update precondition, but these properties do not prove that an action-shaped endpoint, propagation, batch, or downstream observer behaves atomically. Sources: [SPARQL 1.2 Update](https://www.w3.org/TR/sparql12-update/) and [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html).

Neo4j documents ACID transactions with default read-committed isolation, possible non-repeatable reads, and the need for constraints to obtain uniqueness under concurrent `MERGE`. It also documents that semantic-index authorization can return zero or partial results and that full-text/vector scores should not be compared as a single raw scale. Sources: [Neo4j transaction behavior](https://neo4j.com/docs/operations-manual/current/database-internals/), [`MERGE` with constraints](https://neo4j.com/docs/cypher-manual/current/clauses/merge/), and [authorization limitations](https://neo4j.com/docs/operations-manual/current/authentication-authorization/limitations/).

Blueprint impact:

- capability manifests attach to one operation on one build/edition/configuration/identity;
- qualify isolation, conditional write, timeout ambiguity, per-item batch results, search visibility, delete/restore, and readback with fault tests;
- represent unsupported or untested behavior as false/unknown, not a generic adapter default;
- keep search/vector output in candidate generation and record filtering/truncation.

### Catalog and registry APIs expose observable but different effect semantics

DataHub Core 1.7.0 is the selected OSS baseline, while DataHub Cloud release trains have separate versioning. DataHub Cloud 2.1 release notes explicitly state that async orchestration-plugin emit no longer raises on a rejected write and gives no read-after-write guarantee; this is a channel-specific limitation, not a universal DataHub behavior. Sources: [DataHub Core releases](https://github.com/datahub-project/datahub/releases) and [DataHub Cloud 2.1 notes](https://github.com/datahub-project/datahub/blob/master/docs/managed-datahub/release-notes/v_2_1_0.md).

OpenMetadata 1.12's official lineage documentation says connector coverage varies, query-log ingestion uses entity search/resolution, and default/configured result limits can bound observations. Its technical documentation describes parser fallbacks with different accuracy. Sources: [OpenMetadata lineage ingestion](https://docs.open-metadata.org/v1.12.x/connectors/ingestion/lineage) and [lineage technical architecture](https://docs.open-metadata.org/v1.12.x/developers/contribute/codebase-deep-dives/lineage-ingestion).

Atlas exposes explicit classification propagation flags/directions, propagated-classification audit actions, and active/deleted/purged statuses. Confluent distinguishes schema ID, subject version, normalization/configuration, soft/hard delete, and warns that `latest` may change immediately after a compatibility check. Sources: [Atlas v2 API](https://atlas.apache.org/api/v2/index.html), [Atlas classification](https://atlas.apache.org/api/v2/json_AtlasClassification.html), [Schema Registry API](https://docs.confluent.io/platform/current/schema-registry/develop/api.html), and [schema deletion](https://docs.confluent.io/platform/current/schema-registry/schema-deletion-guidelines.html).

Blueprint impact: target effect types remain specific; approval binds explicit target/config/version/digest; asynchronous acceptance is not application; delete and propagation receive dedicated previews, receipts, and reconciliation.

### Lineage events and recipient propagation need explicit coverage

OpenLineage's object model defines job/run/dataset events and lifecycle facets, and its extension model requires versioned immutable schema URLs for custom facets. It does not claim complete source visibility or stewardship authority. Sources: [OpenLineage object model](https://openlineage.io/docs/spec/object-model/) and [facets/extensibility](https://openlineage.io/docs/spec/facets/).

The EU GDPR's Articles 16–19 distinguish rectification, erasure, restriction, and communication to recipients, subject to conditions and exceptions. This is useful evidence for modeling separate correction/restriction/delete actions and recipient receipts, but the blueprint does not encode legal applicability. Source: [GDPR official text](https://eur-lex.europa.eu/eli/reg/2016/679/oj/eng).

Blueprint impact:

- every lineage edge records producer/method/time/visibility/coverage and confidence class;
- correction/deletion propagation inventories known descendants and explicitly unresolved coverage;
- each consumer produces a typed acknowledgement or is reconciled by observable state;
- legal/retention decisions remain owned by the applicable organizational authority.

### Rights, poisoning, supply chain, and human workload are first-class evidence

DCAT 3 differentiates license, access rights, other rights, and optional ODRL policies; ODRL 2.2 models permissions, prohibitions, duties, and constraints. NIST AI 600-1 treats prompt injection, data poisoning, privacy, and provenance as lifecycle risks. NIST SP 800-53 provides separation-of-duties and least-privilege controls; SLSA 1.2 defines build provenance; NASA-TLX supplies a documented multidimensional subjective workload method. Sources: [DCAT 3](https://www.w3.org/TR/vocab-dcat-3/), [ODRL 2.2](https://www.w3.org/TR/odrl-model/), [NIST AI 600-1](https://doi.org/10.6028/NIST.AI.600-1), [NIST SP 800-53 Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final), [SLSA 1.2](https://slsa.dev/spec/v1.2/), and [NASA-TLX](https://www.nasa.gov/human-systems-integration-division/nasa-task-load-index-tlx/).

Blueprint impact:

- evidence carries source authority, use rights, prohibited purposes, retention, and derivative obligations;
- poisoning response invalidates descendants and rebuilds from a known-good watermark;
- model/reviewer/executor duties remain separated and current authority is rechecked;
- release evaluation measures curator correctness, reliance, workload, and queue capacity alongside machine metrics;
- build provenance is verified but never mistaken for proof that code/configuration is benign.

## Product-specific implementation findings

### OpenLineage

OpenLineage's core model and extensible facets are a strong ingestion/interchange choice for runtime lineage. Facet schema URLs are versioned; project-defined facets should use a project namespace and immutable schemas. Lifecycle state changes such as create, alter, rename, overwrite, truncate, and drop are observations that need source-specific identity policy. A rename cannot be assumed to preserve or replace identity uniformly.

Blueprint impact:

- preserve emitted time, received time, producer/build, run identity, facet-schema version, and raw digest;
- map lifecycle events to typed observations, not generic deletes;
- establish completeness/coverage separately from event presence;
- deduplicate by stable event identity and quarantine conflicting duplicates.

### DataHub

DataHub models metadata as entities identified by URNs with collections of aspects; aspects can be versioned or timeseries. Its change-proposal and managed-governance mechanisms show how review can be productized, while stateful ingestion can mark previously emitted entities stale/soft-deleted. Sources: [metadata model](https://github.com/datahub-project/datahub/blob/master/docs/modeling/metadata-model.md), [change proposals](https://github.com/datahub-project/datahub/blob/master/docs/managed-datahub/change-proposals.md), and [stateful ingestion](https://github.com/datahub-project/datahub/blob/master/metadata-ingestion/docs/dev_guides/stateful.md).

Blueprint impact:

- pin pipeline identity and state-store continuity before interpreting missing output as stale;
- treat URN/aspect updates as target-specific effects with versions/receipts;
- keep application decisions and source assertions outside a target-only history;
- verify authorization and search visibility because returned metadata can be principal-scoped.

### OpenMetadata

OpenMetadata exposes versioned APIs for catalog entities, lineage, glossaries, governance, and data quality. Its documentation describes glossary-term versioning and approval workflows, roles/policies, and lineage connector limitations. Current governance work includes preventing a user from approving their own change; the blueprint adopts independent approval in its application authority layer. Sources: [glossary terms](https://docs.open-metadata.org/v1.12.x/how-to-guides/data-governance/glossary/glossary-term), [glossary approval](https://docs.open-metadata.org/v1.12.x/how-to-guides/data-governance/glossary/approval), [roles/policies](https://docs.open-metadata.org/v1.12.x/how-to-guides/admin-guide/roles-policies), [lineage ingestion](https://docs.open-metadata.org/v1.12.x/connectors/ingestion/lineage), and [data-quality APIs](https://docs.open-metadata.org/v1.12.x/api-reference/data-quality).

Blueprint impact:

- use conditional/versioned writes where supported;
- distinguish parser/API result limits, permission filtering, and true absence;
- represent soft/hard deletion and restoration with explicit target semantics;
- do not assume product workflow equals cross-target authorization or effect confirmation.

### Apache Atlas

Atlas provides a type system, entity and glossary APIs, lineage, and classification features. Classification propagation is a consequential graph effect because it can alter many related entities and downstream access policy. The adapter needs distinct contracts for local classification and propagating classification, including an affected-set preview and readback.

Blueprint impact:

- pin Atlas type/entity version/preconditions where the API exposes them;
- use target-specific delete/purge semantics rather than a generic common flag;
- canary and reconcile propagation;
- treat classifications/glossary links as catalog effects, not real-world identity facts.

### Schema registries

Confluent Schema Registry distinguishes subjects, versions, globally assigned schema IDs, references, compatibility levels, and normalization. Data contracts can add metadata, rules, and migrations. Apicurio also provides compatibility/rule mechanisms, with behavior depending on artifact type and configured rules.

Blueprint impact:

- retain context/subject/version/ID/reference identity explicitly;
- compatibility result is an observation with registry version and configuration;
- normalization reduces representational variance but is not semantic equivalence;
- run consumer-contract, ontology/mapping, and operational tests in addition to registry checks;
- treat version registration and deletion as target effects with their own ambiguity/recovery behavior.

## Master-data and entity-resolution evidence

### Statistical foundation

Fellegi and Sunter formalized linkage decisions as link, non-link, and possible-link regions under controlled error probabilities. The lasting engineering lesson is not a specific historical estimator; it is that uncertain comparisons deserve an explicit review/abstention region rather than forced binary output. Source: [Fellegi–Sunter 1969](https://nhis.ipums.org/nhis/resources/Fellegi69.pdf).

### Blocking and unseen data

Candidate generation usually dominates cost and limits recall. Splink's documentation shows quadratic comparison growth and makes blocking recall a first-class measure. Randomly split matching benchmarks can leak entities and make results optimistic. The WDC Products benchmark's unseen-entity and corner-case findings support temporal/entity-disjoint evaluation. Sources: [Splink blocking rules](https://moj-analytical-services.github.io/splink/topic_guides/blocking/blocking_rules.html) and [WDC Products benchmark](https://openproceedings.org/2024/conf/edbt/paper-14.pdf).

### Learned matchers

DeepMatcher reported gains for textual/dirty data while conventional methods could remain competitive for structured matching. Ditto uses pretrained language models and task-specific optimization for entity matching. They demonstrate useful scoring mechanisms, not production authority or a solution to candidate generation, cluster consistency, provenance, or reversible effects. Sources: [DeepMatcher primary paper](https://pages.cs.wisc.edu/~anhai/papers1/deepmatcher-sigmod18.pdf), [Ditto paper](https://arxiv.org/abs/2004.00584), and [Ditto repository](https://github.com/megagonlabs/ditto).

### Entity alignment and `sameAs`

OpenEA provides benchmark implementations for knowledge-graph entity alignment. The broader `sameAs` literature documents that identity links on the Web carry contextual and quality problems; strong logical identity is often too broad for application-specific co-reference. Sources: [OpenEA](https://github.com/nju-websoft/OpenEA) and the primary [“sameAs problem” survey](https://semantic-web-journal.net/content/sameas-problem-survey-identity-management-web-data).

Blueprint impact:

- define identity per entity type, identifier regime, time, and use;
- retain pair candidates separately from accepted edges and cluster assignments;
- use must-link/cannot-link/source-cardinality/temporal constraints;
- make split/unlink and appeals first-class;
- evaluate pair and cluster outcomes, maximum false-cluster size, and blocking loss;
- use embeddings/graph signals only as evidence with calibration and provenance.

### MDM mechanisms

IBM's MDM guidance separates matching from source identifiers/provenance and survivorship. This supports retaining source records and selecting canonical attributes under explicit source/verification/recency policies rather than copying one “golden” record. Sources: [IBM matching algorithms](https://www.ibm.com/docs/en/ws-and-kc?topic=data-matching-algorithms), [master-data concepts](https://www.ibm.com/docs/en/ws-and-kc?topic=data-concepts-in-master-management), and [data survivorship](https://www.ibm.com/docs/en/product-master/14.0.0?topic=features-data-survivorship-feature).

Microsoft removed Master Data Services from SQL Server 2025, an example of why architecture should rely on explicit contracts rather than assume a product remains a permanent control plane. Source: [discontinued Master Data Services features](https://learn.microsoft.com/en-us/sql/master-data-services/discontinued-master-data-services-features?view=sql-server-ver17).

## Data-quality and metadata standards

ISO references are useful for aligning terminology and process, but their full normative text may require licensed access. The public catalog entries establish scope:

- ISO 8000-61 describes a process reference model for data-quality management: [ISO 8000-61](https://www.iso.org/standard/63086.html).
- ISO 8000-110 covers master-data exchange syntax, semantic encoding, and conformance concepts: [ISO 8000-110](https://www.iso.org/standard/78501.html).
- ISO/IEC 11179-32 addresses registration of concept systems, including ontological structures: [ISO/IEC 11179-32](https://www.iso.org/standard/78926.html).

These sources support process/registration discipline but do not prescribe this blueprint's runtime, authority, idempotency, or model controls. Implementation teams subject to these standards must review the licensed text and organizational obligations directly.

## Knowledge-graph research synthesis

The primary Knowledge Graphs survey by Hogan et al. spans graph models, query languages, schema/identity, construction, enrichment, quality, and refinement. It reinforces that “knowledge graph” is not one uniform storage or reasoning model. Source: [Knowledge Graphs survey](https://aidanhogan.com/docs/knowledge-graphs-computing-surveys.pdf), DOI 10.1145/3447772.

Recent uncertainty research treats uncertainty representation and reasoning as an active field rather than a settled standard. The blueprint therefore keeps confidence on candidates/observations instead of encoding probabilistic machine output as ordinary RDF truth. Source: [Uncertainty in Knowledge Graphs survey](https://drops.dagstuhl.de/entities/document/10.4230/TGDK.3.1.3).

Practical conclusion: use the graph for versioned assertions, relationships, and derived projections when it helps workload queries. Do not force case/effect orchestration, confidential evidence blobs, or transactional approval into graph triples if a conventional ledger/database is clearer and more reliable.

## Annotated research decisions

| ID | Decision adopted | Evidence and reasoning | Deliberate limitation / revisit trigger |
|---|---|---|---|
| KG-D01 | Keep Stage 0 deterministic/human as a valid endpoint | SHACL, registry, catalog, and linkage sources each provide bounded mechanisms; none requires an agent | Revisit only when measured unresolved ambiguity and curator evidence-assembly cost justify a bounded loop |
| KG-D02 | Use one bitemporal version envelope across durable record kinds | Db2's business/system-time separation matches correction/audit requirements; PROV supplies derivation vocabulary | Storage may implement it differently; valid-time overlap rules remain kind/policy specific |
| KG-D03 | Preserve assertion, observation, candidate, decision, canonical fact, provenance, and effect separately | OWL/SHACL/PROV and product APIs answer different questions; ambiguous writes require their own state | Adds joins/storage; accepted because reversibility and audit fail without it |
| KG-D04 | Separate pair scoring, relationship acceptance, and cluster mutation | Fellegi–Sunter uncertainty, blocking evidence, WDC unseen cases, MDM practice, and `sameAs` failures show local similarity is insufficient | No universal clustering method selected; entity-type policy and evaluated constraints choose it |
| KG-D05 | Treat RDF 1.1/OWL 2/SHACL 2017 as stable base; capability-gate RDF/SPARQL/SHACL 1.2 | RDF 1.2 Concepts/Semantics are CR Snapshots; SPARQL/SHACL 1.2 remain WDs on access date | Revisit on Recommendation status plus target conformance/interoperability evidence |
| KG-D06 | Keep RDF, property graph, relational ledger, and search/vector as complementary representations | W3C dataset semantics, Neo4j transaction/search documentation, and operational state needs differ | Avoids a universal graph abstraction; adapters may optimize without changing record semantics |
| KG-D07 | Qualify operations, not vendors | SPARQL atomicity wording, Neo4j isolation/constraint behavior, DataHub async channels, Atlas propagation, and registry delete/version semantics vary | Qualification expires on build/edition/config/auth/topology change |
| KG-D08 | Require `unknown` effect plus independent reconciliation | HTTP/broker/workflow guarantees do not extend across external systems; product APIs may commit before response loss | Readback can itself be permission-filtered/eventually consistent; unresolved cases stay open/manual |
| KG-D09 | Model correction, restriction, invalidation, deletion, purge, and hold separately | PROV invalidation, product soft/hard delete, Schema Registry deletion, and GDPR distinctions differ | Legal applicability remains outside the agent; complete descendant discovery may remain unknown |
| KG-D10 | Make context/compaction loss-aware and rehydrate on restart/provider switch | Prompts are bounded projections; authoritative ledgers and effect ambiguity must survive | Does not promise provider determinism; an unqualified provider switch pauses model work |
| KG-D11 | Use exactly seven explicit memory lifetimes and reject preference/session authority | Governance facts must enter through releases/ledgers; outcome reuse risks bias, poisoning, and tenant leakage | Curated adjudicated outcome corpora remain allowed as versioned artifacts, not implicit memory |
| KG-D12 | Evaluate outcome, trajectory, invariant, and human factors together | Correct labels can hide unsafe tool paths; curator workload and reliance constrain production safety | NASA-TLX is supplementary and subjective; no universal workload or accuracy threshold is asserted |
| KG-D13 | Canary and roll back the whole behavior bundle | Model, prompt, tools, policy, semantic bundle, adapter, provider, and UI interactions change behavior | Committed effects may need forward correction; rollback is not historical erasure |
| KG-D14 | Mine failures only through quarantine, adjudication, rights review, and regression creation | Production approvals/outcomes can be biased, poisoned, private, temporally leaky, or mislabeled | Slower than online learning by design; authority never changes automatically |

## Contradictions and unresolved design tensions

### Open world versus operational closure

- **Standard reality:** OWL absence is not falsity.
- **Operational need:** ingestion, quality, and effect postconditions need bounded completeness.
- **Resolution:** declare a data snapshot, visibility scope, shape/rule set, and entailment mode. Say “conforms to this contract in this snapshot,” not “the world is complete.”

### Named graph versus provenance

- **Common shortcut:** graph name equals source/tenant/provenance.
- **Standard reality:** RDF does not define that relationship.
- **Resolution:** publish an explicit dataset convention and record PROV/application provenance separately.

### Schema compatibility versus semantic compatibility

- **Registry result:** old/new serialized data is compatible under configured mode.
- **Consumer reality:** units, business meaning, inference, query behavior, or hidden consumer expectations may still break.
- **Resolution:** combine registry checks with consumer contracts, semantic mapping/competency tests, and canary telemetry.

### Product soft delete versus lifecycle truth

- **Product mechanism:** omitted stateful-ingestion entities may be soft-deleted; targets expose soft/hard delete flags.
- **Source reality:** omission may be permissions, partial scan, rename, or outage.
- **Resolution:** require complete-snapshot proof or tombstone evidence; model unobserved, stale, deprecated, invalidated, soft-deleted, hard-deleted, and purged separately.

### Local matching confidence versus cluster identity

- **Matcher output:** several high-scoring pair links.
- **Graph effect:** transitive closure joins a much larger cluster.
- **Resolution:** separate pair and cluster policies, enforce global/temporal/source constraints, report maximum overmerge, and require a split plan.

### Human review versus throughput

- **Safety need:** ambiguous/high-harm changes need independent judgment.
- **Operational reality:** review queues can become the limiting system.
- **Resolution:** measure review capacity before launch, improve evidence quality, use deterministic auto-decisions only for proven low-risk slices, and backpressure proposals without lowering thresholds.

### Target abstraction versus semantic honesty

- **Engineering temptation:** one generic catalog `upsert`, `tag`, or `delete` API.
- **Product reality:** aspect patches, entity versions, classification propagation, registry versioning, and deletion semantics differ.
- **Resolution:** share the envelope (authorization, proposal digest, receipt, reconciliation) but retain target-specific effect types and capability manifests.

### Learning from decisions versus automation bias

- **Potential value:** reviewer decisions can improve models/rules.
- **Risk:** historical bias, inconsistent labels, compromised cases, tenant leakage, and model self-reinforcement.
- **Resolution:** curate/de-identify/adjudicate outcome data into a versioned evaluation/training artifact. No session/user memory or automatic threshold/ontology update receives production authority.

## Rejected or constrained alternatives

| Alternative | Why it is insufficient as the default | Where it remains useful |
|---|---|---|
| LLM compares every record pair | Quadratic cost, unstable output, poor provenance, no candidate completeness | Reviewer explanation on a small evidence-bound pair set |
| Vector nearest neighbor defines identity | Semantic similarity is not co-reference; vulnerable to drift and tenant leakage | Candidate generation or evidence retrieval with recall evaluation |
| Connected components over accepted pairs | One bridge can cause catastrophic overmerge | Only under strong equivalence evidence and cluster constraints |
| `owl:sameAs` for application duplicates | Global logical identity is stronger than many business use cases | Carefully curated true identity under declared ontology semantics |
| SHACL auto-repairs violations | A violation rarely determines the causal layer or unique repair | Trigger typed cases or deterministic safe normalization |
| Schema-registry compatibility approves release | Does not cover business/query/inference/operational compatibility | One release gate for representation evolution |
| Catalog API is the sole audit ledger | Target history and delete semantics vary; cross-target recovery is weak | Operational target state and receipts |
| Workflow replay means exactly-once writes | Target may apply a timed-out request; server semantics vary | Internal step durability combined with effect ledger/reconciliation |
| Model “memory” learns steward preferences | Preferences should not change governed identity/facts; leaks bias/tenant data | Curated offline evaluation/training artifacts after review |
| Full autonomous agent loop | Unbounded source/tool scope and mutations increase risk/cost | None for ordinary production stewardship; bounded read subplans only |

## Evidence-to-blueprint traceability

| Blueprint decision | Strongest evidence | Implemented in |
|---|---|---|
| Open-world inference separated from validation | OWL 2, SHACL | [Ontology/schema/vocabulary](../../agents/knowledge-graph-stewardship-agent/03-ontology-schema-and-vocabulary-governance.md) |
| Named graph not treated as provenance | RDF datasets, PROV-O | [Catalog/lineage/provenance](../../agents/knowledge-graph-stewardship-agent/04-catalog-lineage-provenance-and-reconciliation.md) |
| Resource and catalog record separated | DCAT 3 | [Catalog/lineage/provenance](../../agents/knowledge-graph-stewardship-agent/04-catalog-lineage-provenance-and-reconciliation.md) |
| Pair/review/non-pair bands | Fellegi–Sunter | [Identity resolution](../../agents/knowledge-graph-stewardship-agent/02-identity-resolution-and-candidate-graphs.md) |
| Valid time separated from recorded history | Db2 bitemporal tables, PROV-O | [State and record contract](../../agents/knowledge-graph-stewardship-agent/06-state-context-memory-planning-and-tool-contracts.md#canonical-record-and-time-contract) |
| Blocking recall and unseen-entity tests | Splink, WDC benchmark | [Evaluation](../../agents/knowledge-graph-stewardship-agent/09-evaluation-failure-injection-and-staged-delivery.md) |
| Retained assertions and survivorship policy | IBM MDM concepts | [Steward cases](../../agents/knowledge-graph-stewardship-agent/05-steward-cases-quality-repair-and-merge-split-workflows.md) |
| Versioned lineage observations/facets | OpenLineage | [Catalog/lineage/provenance](../../agents/knowledge-graph-stewardship-agent/04-catalog-lineage-provenance-and-reconciliation.md) |
| Target-specific capabilities/effects | SPARQL, Neo4j, DataHub, OpenMetadata, Atlas, OpenLineage, Schema Registry, HTTP and Kafka docs | [Runtime/tool qualification](../../agents/knowledge-graph-stewardship-agent/06-state-context-memory-planning-and-tool-contracts.md#operation-level-adapter-qualification) |
| Correction/delete descendant propagation | PROV-O, OpenLineage coverage boundaries, GDPR Articles 16–19 | [Catalog reconciliation](../../agents/knowledge-graph-stewardship-agent/04-catalog-lineage-provenance-and-reconciliation.md#correction-restriction-and-deletion-propagation) |
| Evidence rights, poisoning, SoD, and supply chain | DCAT/ODRL, NIST AI 600-1/SP 800-53, SLSA | [Security and authority](../../agents/knowledge-graph-stewardship-agent/07-security-privacy-tenancy-and-authority.md) |
| Human workload and whole-workflow evaluation | NASA-TLX plus combined source-to-effect evidence | [Evaluation/stages](../../agents/knowledge-graph-stewardship-agent/09-evaluation-failure-injection-and-staged-delivery.md#human-factor-evaluation) |
| Model bounded to proposal | Cross-cutting runtime/security evidence plus product gaps | [Reference architecture](../../agents/knowledge-graph-stewardship-agent/01-mission-boundaries-and-reference-architecture.md) |
| Staged authority and fault gates | Combined evidence | [Evaluation/stages](../../agents/knowledge-graph-stewardship-agent/09-evaluation-failure-injection-and-staged-delivery.md) |

## Refresh triggers

Re-run focused research when any of these occurs:

- RDF 1.2 advances beyond the 7 April 2026 Candidate Recommendation Snapshots, or SPARQL 1.2/SHACL 1.2 advances from Working Draft or changes tested features;
- W3C Reconciliation API status or protocol changes;
- OpenLineage core/facet/lifecycle semantics change;
- DataHub, OpenMetadata, Atlas, Schema Registry, Apicurio, or MDM APIs change major version, deletion behavior, authorization, lineage completeness, or idempotency;
- a new stable Splink major release changes blocking/matching APIs;
- the model/provider changes retention, tool-use, structured-output, or reproducibility behavior;
- a source connector changes pagination, snapshot, tombstone, or permission-filter semantics;
- production incidents reveal false-merge, tenant, classification-propagation, or unknown-effect failure not covered by the packet;
- applicable privacy, retention, AI, or sector rules change.

Record a new research date and retain the previous packet in version history. Do not silently replace a stable recommendation baseline with a draft or prerelease.

## Source access and version boundaries

| Source family | Accessed | Boundary used in this packet |
|---|---|---|
| RDF 1.1 / OWL 2 / SHACL 2017 / JSON-LD 1.1 / SKOS / PROV-O / DCAT 3 / ODRL 2.2 | 2026-08-31 | Published W3C Recommendations/Notes as linked; no claim that every store implements every feature/profile |
| RDF 1.2 Concepts and Semantics | 2026-08-31 | 7 April 2026 Candidate Recommendation Snapshots; not treated as Recommendations |
| SPARQL 1.2 and SHACL 1.2 family | 2026-08-31 | Working Drafts current on access date; cited for status/explicit draft semantics only |
| Reconciliation Service API | 2026-08-31 | Community Group Final Report 0.2; not a W3C Recommendation |
| OpenLineage | 2026-08-31 | Release 1.52.0 plus current spec pages; backend ordering/dedup/completeness remains deployment-specific |
| DataHub | 2026-08-31 | Core 1.7.0; Cloud release notes are separately labeled and not generalized to Core APIs |
| OpenMetadata | 2026-08-31 | Stable 1.12.8 release and versioned 1.12.x docs; 1.13 prerelease material excluded |
| Apache Atlas | 2026-08-31 | 2.5.0 distribution/API documentation; deployment patches/configuration must be probed |
| Neo4j | 2026-08-31 | Current 2026.07.1 docs; edition, Cypher version, CDC, vector/search, cluster, and security configuration are explicit capability dimensions |
| Confluent Schema Registry | 2026-08-31 | Current Confluent Platform API/docs; Cloud and self-managed differences require separate manifests |
| Apicurio Registry | 2026-08-31 | 3.3.x documentation; artifact/rule support varies and was not treated as semantically equivalent to Confluent |
| Splink | 2026-08-31 | 4.0.16 stable; 5.0 development material excluded |
| Kafka | 2026-08-31 | Apache Kafka 4.1 documentation for idempotent/transactional producer boundaries; no external-target exactly-once claim |
| IBM Db2 temporal/MDM | 2026-08-31 | Db2 12.1/11.5 temporal docs and current linked MDM concepts; used for semantics, not a product mandate |
| NIST AI 600-1 / SP 800-53 | 2026-08-31 | Published official documents/updates; controls require organizational tailoring |
| GDPR | 2026-08-31 | Official Regulation (EU) 2016/679 text; technical design support only, not legal advice |
| SLSA | 2026-08-31 | Specification 1.2; draft pages excluded from normative claims |
| NASA-TLX | 2026-08-31 | Official instrument/instructions; supplementary subjective workload measure, not a correctness gate |

Search or documentation pages labeled `current`, `latest`, `master`, or `next` are cited only where the packet also states the accessed baseline or uses them to identify a limitation. Implementation teams should archive the exact page/spec/artifact digest used for qualification because URLs can move without preserving behavior.

## Primary source index

### W3C and semantic standards

- [RDF 1.1 Datasets](https://www.w3.org/TR/rdf11-datasets/)
- [RDF 1.2 Concepts publication history](https://www.w3.org/standards/history/rdf12-concepts/)
- [RDF 1.2 Semantics publication history](https://www.w3.org/standards/history/rdf12-semantics/)
- [SPARQL 1.2 Query](https://www.w3.org/TR/sparql12-query/)
- [SPARQL 1.2 Update](https://www.w3.org/TR/sparql12-update/)
- [OWL 2 Primer](https://www.w3.org/TR/owl2-primer/)
- [OWL 2 Structural Specification](https://www.w3.org/TR/owl2-syntax/)
- [OWL 2 Profiles](https://www.w3.org/TR/owl2-profiles/)
- [SHACL Recommendation](https://www.w3.org/TR/shacl/)
- [SHACL 1.2 Core Working Draft](https://www.w3.org/TR/shacl12-core/)
- [JSON-LD 1.1](https://www.w3.org/TR/json-ld11/)
- [SKOS Reference](https://www.w3.org/TR/skos-reference/)
- [PROV-O](https://www.w3.org/TR/prov-o/)
- [DCAT 3](https://www.w3.org/TR/vocab-dcat-3/)
- [ODRL Information Model 2.2](https://www.w3.org/TR/odrl-model/)
- [W3C Entity Reconciliation API v0.2 Community Report](https://www.w3.org/community/reports/reconciliation/CG-FINAL-specs-0.2-20230410/)

### Catalog, lineage, schema, and MDM projects

- [OpenLineage documentation](https://openlineage.io/docs/)
- [OpenLineage facet specification](https://openlineage.io/docs/spec/facets/)
- [DataHub metadata model](https://github.com/datahub-project/datahub/blob/master/docs/modeling/metadata-model.md)
- [DataHub stateful ingestion](https://github.com/datahub-project/datahub/blob/master/metadata-ingestion/docs/dev_guides/stateful.md)
- [OpenMetadata v1.12 API](https://docs.open-metadata.org/v1.12.x/api-reference)
- [OpenMetadata lineage ingestion](https://docs.open-metadata.org/v1.12.x/connectors/ingestion/lineage)
- [Apache Atlas v2 REST API](https://atlas.apache.org/api/v2/index.html)
- [Neo4j Operations Manual](https://neo4j.com/docs/operations-manual/current/)
- [Neo4j concurrent data access](https://neo4j.com/docs/operations-manual/current/database-internals/concurrent-data-access/)
- [Neo4j semantic-index authorization limitations](https://neo4j.com/docs/operations-manual/current/authentication-authorization/limitations/)
- [Confluent Schema Registry API](https://docs.confluent.io/platform/current/schema-registry/develop/api.html)
- [Confluent data contracts](https://docs.confluent.io/platform/current/schema-registry/fundamentals/data-contracts.html)
- [Apicurio Registry compatibility](https://www.apicur.io/registry/docs/apicurio-registry/3.3.x/getting-started/assembly-registry-compatibility-modes.html)
- [IBM matching algorithms](https://www.ibm.com/docs/en/ws-and-kc?topic=data-matching-algorithms)
- [IBM master-data concepts](https://www.ibm.com/docs/en/ws-and-kc?topic=data-concepts-in-master-management)
- [IBM Db2 bitemporal tables](https://www.ibm.com/docs/en/db2/12.1.x?topic=tables-bitemporal)
- [RFC 9110 HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [Apache Kafka delivery semantics](https://kafka.apache.org/41/design/design/)

### Security, privacy, supply chain, operations, and human factors

- [NIST AI 600-1 Generative AI Profile](https://doi.org/10.6028/NIST.AI.600-1)
- [NIST SP 800-53 Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [GDPR official text](https://eur-lex.europa.eu/eli/reg/2016/679/oj/eng)
- [SLSA specification 1.2](https://slsa.dev/spec/v1.2/)
- [OpenTelemetry semantic conventions](https://opentelemetry.io/docs/specs/semconv/)
- [NASA Task Load Index](https://www.nasa.gov/human-systems-integration-division/nasa-task-load-index-tlx/)

### Identity and knowledge-graph research

- [Fellegi and Sunter, 1969](https://nhis.ipums.org/nhis/resources/Fellegi69.pdf)
- [DeepMatcher, SIGMOD 2018](https://pages.cs.wisc.edu/~anhai/papers1/deepmatcher-sigmod18.pdf)
- [Ditto, VLDB 2020](https://arxiv.org/abs/2004.00584)
- [WDC Products entity-matching benchmark, EDBT 2024](https://openproceedings.org/2024/conf/edbt/paper-14.pdf)
- [OpenEA](https://github.com/nju-websoft/OpenEA)
- [Knowledge Graphs, ACM Computing Surveys](https://aidanhogan.com/docs/knowledge-graphs-computing-surveys.pdf)
- [The `sameAs` problem survey](https://semantic-web-journal.net/content/sameas-problem-survey-identity-management-web-data)
- [Uncertainty in Knowledge Graphs survey](https://drops.dagstuhl.de/entities/document/10.4230/TGDK.3.1.3)

## Packet quality checklist

- [x] Stable recommendations are distinguished from Working Drafts and community reports.
- [x] Current stable project versions were checked and prereleases excluded from the baseline.
- [x] Standards, vendor/project APIs, MDM practice, and primary entity-matching/KG research were cross-compared.
- [x] Product mechanisms are not presented as application-level authority, completeness, or idempotency guarantees.
- [x] Bitemporal record semantics, correction/deletion propagation, and operation-level adapter qualification are evidence-backed.
- [x] Human workload, security/rights, whole-bundle release, and governed failure-mining evidence are included.
- [x] Contradictions and immature areas are documented rather than smoothed over.
- [x] The packet traces major evidence to implementable blueprint contracts and stages.
- [x] Research decisions, access date, product/status boundaries, and refresh triggers are explicit.
- [x] Volatile claims have refresh triggers.

## Related repository guidance

- [Blueprint README](../../agents/knowledge-graph-stewardship-agent/README.md)
- [Mission boundaries and reference architecture](../../agents/knowledge-graph-stewardship-agent/01-mission-boundaries-and-reference-architecture.md)
- [Cross-cutting controls research packet](agent-blueprint-cross-cutting-controls.md)
- [Agent state and event contracts research packet](agent-state-and-event-contracts.md)

# Knowledge Graph and Data-Catalog Stewardship Agent Blueprint

> **Status:** research-backed production blueprint  
> **Research baseline:** 2026-08-31  
> **Scope:** candidate entity resolution, ontology/schema/vocabulary proposals, metadata and lineage evidence, merge/split provenance, curator review, data-quality repair state, and catalog/graph reconciliation  
> **Evidence:** [research packet](../../research/packets/knowledge-graph-stewardship-agent-blueprint.md)

A knowledge-graph stewardship agent is not an enterprise answer bot, an autonomous master-data hub, or a language model with write access to a graph. It is a governed reconciliation system that turns conflicting identifiers, assertions, schemas, lineage observations, and quality signals into **evidence-bearing candidates and reviewable change sets**.

The production design is deliberately asymmetric:

- deterministic rules, exact identifiers, registry checks, and catalog diffing handle known cases;
- statistical and model-assisted methods generate candidates, explanations, and repair proposals only where deterministic methods are insufficient;
- a durable case ledger preserves assertions, observations, evidence, versions, decisions, and effects separately;
- stewards, data owners, ontology/schema owners, and curators retain canonical identity, semantic, and publication authority;
- a constrained publisher applies only an exact approved change set and then reconciles every target projection.

~~~mermaid
flowchart LR
    S["Sources, catalogs, registries, lineage, DQ"] --> I["Identity-aware intake"]
    I --> N["Normalize without erasing source assertion"]
    N --> D{"Deterministic outcome?"}
    D -->|"yes"| C["Candidate or drift case"]
    D -->|"no"| M["Bounded matcher / proposal assistant"]
    M --> C
    C --> V["Independent validators + impact preview"]
    V --> R["Steward / owner / curator review"]
    R -->|"approve exact digest"| P["Constrained publisher"]
    R -->|"reject / revise"| C
    P --> G["Canonical catalog / graph / MDM / registry"]
    G --> Q["Reconcile receipts, descendants, and projections"]
    Q --> L["Decision and provenance ledger"]
~~~

The model may rank candidate links, explain contradictory evidence, draft a SHACL shape, suggest a SKOS mapping, or summarize a lineage gap. It cannot turn similarity into identity, make an OWL inference into a source assertion, approve its own proposal, or silently rewrite a source record.

## What this blueprint owns

- candidate links among source records, catalog assets, graph entities, schema elements, and controlled-vocabulary concepts;
- identity evidence, negative evidence, blocking provenance, score calibration, cluster diagnostics, and unresolved alternatives;
- proposed merge, split, unlink, alias, supersession, deprecation, mapping, and repair operations;
- ontology, SHACL, catalog-schema, schema-registry, and controlled-vocabulary change proposals;
- metadata and lineage observations with producer, time, version, extraction method, and confidence;
- curator cases, conflict state, approval digests, decision rationale, effect receipts, and reversal lineage;
- reconciliation among catalogs, graphs, MDM hubs, lineage stores, schema registries, vocabularies, source systems, and derived indexes.

## What remains outside the boundary

| Concern | Canonical owner | Stewardship-agent relationship |
|---|---|---|
| Enterprise questions and grounded answers | [Enterprise knowledge agent](../enterprise-knowledge-agent/README.md) | Consumes governed entities and metadata; this agent does not answer across the enterprise corpus |
| Pipeline execution, replay, and backfill | [Data Pipeline Operations Agent](../data-pipeline-operations-agent/README.md) | Supplies and consumes lineage/quality evidence; this agent does not run pipelines |
| Database DDL and physical operations | [Database Operations Agent](../database-operations-agent/README.md) | Receives schema impact findings; this agent does not mutate databases |
| Analytics and business interpretation | [Analytics Agent](../analytics-agent/README.md) | Uses curated semantics; this agent does not produce business conclusions |
| Source-record correction | Source-system owner | Receives a repair proposal; source truth is never rewritten unilaterally |
| Canonical identity and survivorship | Data owner or MDM steward | Reviews merge/split/link decisions and attribute survivorship |
| Ontology, schema, or vocabulary publication | Ontology/schema owner or curator | Reviews and publishes a versioned semantic change |
| Access policy and classification consequences | Security/privacy owner | Reviews policy-impacting changes; a tag suggestion is not authorization |

## Read in this order

| Guide | Production decision |
|---|---|
| [Mission, boundaries, and reference architecture](01-mission-boundaries-and-reference-architecture.md) | Qualify the workload, preserve human authority, and choose deterministic, statistical, graph, or model-assisted components |
| [Identity resolution and candidate graphs](02-identity-resolution-and-candidate-graphs.md) | Generate, score, review, merge, split, and reverse identity candidates without equating similarity with identity |
| [Ontology, schema, vocabulary, and version governance](03-ontology-schema-and-vocabulary-governance.md) | Separate inference, validation, compatibility, semantic review, migration, and publication |
| [Catalog, lineage, provenance, and reconciliation](04-catalog-lineage-provenance-and-reconciliation.md) | Integrate catalogs and lineage systems while preserving observations, gaps, deletions, and source authority |
| [Steward cases, quality repair, and merge/split workflows](05-steward-cases-quality-repair-and-merge-split-workflows.md) | Operate curator queues, quality repairs, approvals, appeals, and reversals |
| [State, context, memory, planning, and tool contracts](06-state-context-memory-planning-and-tool-contracts.md) | Make runs resumable, context lossy by design, memory explicit, and tools typed and reconcilable |
| [Security, privacy, tenancy, and authority](07-security-privacy-tenancy-and-authority.md) | Contain sensitive identity data, metadata injection, cross-tenant linkage, and authority escalation |
| [Reliability, observability, scaling, and operations](08-reliability-observability-scaling-and-operations.md) | Partition, backpressure, deploy, trace, reconcile, respond to incidents, and control cost |
| [Evaluation, failure injection, and staged delivery](09-evaluation-failure-injection-and-staged-delivery.md) | Progress through stages 0–6 with workload-specific tests and exit gates |

## Seven records that must never collapse into one

| Record | Meaning | May be canonical? |
|---|---|---|
| **Source assertion** | What a named source asserted at a source version and effective time | Only within that source's authority |
| **Observation** | What an extractor, matcher, profiler, or lineage producer observed | No; it is evidence with coverage and method limits |
| **Candidate** | A proposed link, fact, mapping, repair, merge, split, or semantic change | No; it is review input |
| **Curator decision** | An authorized accept/reject/defer decision bound to exact evidence and proposal versions | It authorizes a bounded publication effect |
| **Canonical fact** | A fact published by the designated owner into a named canonical system/version | Yes, only for the declared scope and interval |
| **Provenance** | The source, activity, agent, time, transformation, version, and derivation chain for another record | No; it explains lineage and authority rather than replacing the record |
| **External effect** | A prepared or attempted mutation plus target precondition, idempotency key, receipt, and reconciliation state | No; approval is not application, and an ambiguous effect is not confirmed state |

Confidence belongs to the candidate or observation that produced it. It is not a property of reality, an access-control decision, or permission to publish.

The complete identity, version, valid-time, transaction-time, correction, and release contract is defined in [State, context, memory, planning, and tool contracts](06-state-context-memory-planning-and-tool-contracts.md#canonical-record-and-time-contract). That contract is normative for this blueprint; examples in other guides are intentionally smaller views of it.

## Non-negotiable invariants

1. Every record and effect key includes tenant, domain, entity type, and identifier namespace; cross-tenant matching is denied by default.
2. Exact identifiers are evaluated in their declared namespace and validity interval. String equality across namespaces is not identity.
3. Embeddings, nearest neighbors, language-model judgments, and graph proximity produce candidates only.
4. A candidate keeps positive evidence, negative evidence, missing evidence, model/rule versions, threshold policy, and alternatives.
5. Pairwise links and entity clusters are different decisions. Transitive closure is never silently treated as a universal merge guarantee.
6. Merge and split operations preserve member history, aliases, supersession, decision provenance, and a tested reversal path.
7. OWL inference, SHACL validation, schema-registry compatibility, and business-semantic approval are separate gates.
8. Lineage absence means unknown coverage, not proven absence of a dependency.
9. Source assertion, inferred statement, curator decision, canonical fact, and external publication effect use different record types.
10. Approval binds approver authority, exact proposal digest, target versions, policy version, expiry, and use count. The proposer cannot approve its own change.
11. A successful write response is not completion. Every target is read back or reconciled using a durable effect identity.
12. The context window, vector index, graph cache, trace store, and search index are projections—not authoritative case state.
13. Deletion, invalidation, soft deletion, purge, deprecation, and tombstoning are distinct operations with explicit descendant and evidence policy.
14. Releases are gated by identity, semantic, lineage, safety, repeated-reliability, and steward-workload evaluation—not aggregate F1 alone.

## Standards baseline and maturity caveat

This blueprint uses stable W3C Recommendations where possible: RDF 1.1, OWL 2, SHACL 2017, JSON-LD 1.1, SKOS, PROV-O, and DCAT 3. As of the research date, RDF 1.2 Concepts and Semantics are Candidate Recommendation Snapshots, while SPARQL 1.2 and the SHACL 1.2 family remain Working Drafts. None is treated as a universally interoperable production baseline until the deployed processors pass qualification. The [W3C Reconciliation Service API 0.2](https://www.w3.org/community/reports/reconciliation/CG-FINAL-specs-0.2-20230410/) is a Community Group report, not a W3C Recommendation.

Observed implementation baselines include OpenLineage 1.52.0, DataHub Core 1.7.0, OpenMetadata 1.12.8, Apache Atlas 2.5.0, Neo4j 2026.07.1 documentation, and Splink 4.0.16. These are research anchors, not universal compatibility claims. Cloud and self-managed editions, licenses, configuration, API channels, security policies, and patch releases differ. Pin the deployed build and API/schema version, then run the operation-level qualification suite before enabling reads or effects.

## Definition of done

A production cohort is not ready until operators can answer all of these questions from durable evidence:

- Why were these two records compared, and which plausible candidates were excluded by blocking?
- Which source assertions, versions, effective times, and negative signals support or contradict the proposed link?
- What happens to every member, alias, assertion, edge, downstream reference, and prior decision if a merge is approved or reversed?
- Which reasoning regime and ontology/shapes/schema versions produced each inference or validation result?
- Does a passing compatibility check prove only wire compatibility, or has a domain owner also accepted the semantic change?
- How complete and fresh is lineage for this source, and what does a missing edge actually mean?
- Can a deletion request be traced through the canonical graph, catalogs, vectors, caches, cases, evaluation corpora, and retained audit evidence?
- Can work resume after process death or approval delay without duplicating a publication effect or applying a stale decision?
- Can the operation manifest prove concurrency, ambiguity, deletion, visibility, and read-back behavior for the exact target build and principal?
- Can one tenant, source, entity type, or high-degree block overload the system or leak candidates into another scope?
- Do production-like, temporally split, unseen-entity, adversarial, reversal, human-workload, and recovery-load evaluations meet explicit release thresholds?

If any answer depends on a chat transcript, a similarity score, or a vendor dashboard alone, the system remains a prototype.

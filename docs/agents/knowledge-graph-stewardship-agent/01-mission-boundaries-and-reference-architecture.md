# Mission, Boundaries, and Reference Architecture

## Product contract

The product accepts a bounded stewardship objective and returns one of four outcomes:

1. a deterministic no-change result with evidence;
2. a reviewable candidate change set;
3. an insufficiency/conflict result that names missing evidence and the responsible owner;
4. a reconciled publication receipt for a separately approved change.

It does not return “the entity is the same” or “the ontology is correct” merely because a model is confident. The product contract is **advisory by default and effectful only through an explicit authority boundary**.

Representative users are data stewards, MDM curators, ontology/schema owners, catalog administrators, privacy/security reviewers, and source-system owners. Representative tasks are:

- reconcile duplicate supplier, product, asset, dataset, field, metric, or concept identifiers;
- review a proposed merge or split and its downstream impact;
- align a source schema or vocabulary with a governed model;
- propose a new or changed term, class, property, SHACL shape, or schema-registry artifact;
- identify stale catalog metadata, missing lineage, inconsistent classifications, or invalid shapes;
- coordinate a quality repair while keeping source correction and canonical publication separate;
- prove which versions, evidence, approvals, and writes produced the current graph state.

## Authority is a data structure

| Decision | Agent may do | Required authority | Prohibited shortcut |
|---|---|---|---|
| Candidate pair or mapping | Generate, score, explain, defer | Pre-authorized read scope | Treat top score as a link |
| Cluster proposal | Simulate component changes and conflicts | Steward review | Apply transitive closure as truth |
| Canonical merge/split | Prepare exact change set and rollback plan | Data owner/MDM steward; dual control for high-impact identity classes | Self-approve or silently choose survivorship |
| Ontology/schema/vocabulary proposal | Diff, validate, reason, test, and draft migration | Named ontology/schema owner or curator | Publish because SHACL/compatibility passed |
| Catalog metadata repair | Suggest exact patch | Asset owner/steward or narrow deterministic policy | Overwrite source-owned fields |
| Lineage correction | Add an observed/manual candidate edge with provenance | Lineage owner or affected asset owners | Claim completeness from a successful parse |
| Source-record correction | Open a repair request | Source-system owner | Directly mutate operational source data |
| Classification/policy-impacting change | Show impact and stage proposal | Security/privacy owner where policy changes | Use model classification as access policy |
| Purge/deletion | Enumerate projections and evidence policy | Data/privacy owner plus platform operator | Cascade on guessed identity or incomplete lineage |

Application code resolves the current principal, tenant, purpose, target versions, ownership, segregation-of-duties policy, and approval. Prompt text cannot grant authority.

## Reference architecture

~~~mermaid
flowchart TB
    subgraph Producers["Evidence producers"]
        SRC["Source systems"]
        CAT["Catalogs / MDM / graph"]
        REG["Schema registries / vocabularies"]
        LIN["Lineage / DQ / profiler events"]
        EXT["Approved third parties"]
    end

    subgraph Control["Trusted stewardship control plane"]
        ADM["Admission: identity, tenant, purpose, budgets"]
        SNAP["Versioned evidence snapshot"]
        WF["Deterministic case workflow"]
        LED["Case + decision + effect ledger"]
        POL["Policy and approval verifier"]
    end

    subgraph Analysis["Untrusted analysis plane"]
        RULE["Rules / exact IDs / constraints"]
        MATCH["Probabilistic matcher / blocking"]
        GRAPH["Graph and impact analysis"]
        MODEL["Bounded language-model assistant"]
    end

    subgraph Effect["Separated publication plane"]
        PRE["Precondition + digest revalidation"]
        PUB["Scoped publisher adapters"]
        VER["Independent read-back / reconciliation"]
    end

    SRC --> ADM
    CAT --> ADM
    REG --> ADM
    LIN --> ADM
    EXT --> ADM
    ADM --> SNAP --> WF
    WF --> RULE
    WF --> MATCH
    WF --> GRAPH
    WF --> MODEL
    RULE --> WF
    MATCH --> WF
    GRAPH --> WF
    MODEL --> WF
    WF <--> LED
    WF --> POL --> PRE --> PUB --> VER --> LED
~~~

Trust boundaries matter more than product boundaries:

- connector output, graph labels, comments, schema descriptions, glossary definitions, tickets, and third-party content are untrusted evidence;
- match scores and model text are untrusted analysis;
- the case ledger is authoritative only for workflow state, not for source facts;
- the designated catalog, MDM hub, graph, ontology repository, or schema registry is authoritative only for the fields and versions its governance contract assigns to it;
- publication receipts plus authoritative read-back establish the external effect state.

## The assertion lattice

The system must preserve how a statement entered the system. A compact statement record uses the following kinds:

| Kind | Example | Allowed transformation |
|---|---|---|
| `source_assertion` | CRM record 918 asserts legal name “Northwind Ltd” | Normalize into an observation while retaining the exact source value |
| `observation` | Parser observed field `cust_id` in schema version 44 | Compare, validate, or cite; never silently promote |
| `candidate_link` | CRM 918 may match ERP V-204 | Score, review, accept, reject, supersede |
| `inferred_statement` | OWL reasoner entails an instance type under ontology v7 | Label with regime and versions; do not attribute to the source |
| `validation_result` | SHACL reports a missing required identifier | Open a repair case; validation does not repair the graph |
| `curator_decision` | Steward accepts link candidate 72 | Authorize only the exact proposal digest and scope |
| `canonical_fact` | MDM v231 publishes canonical supplier ID 501 | Consume as canonical within the declared system/domain/time |
| `external_effect` | Catalog patch revision 884 applied | Reconcile and attach receipt; do not infer from request success |

This separation prevents a common corruption path: extraction becomes assertion, similarity becomes identity, inference becomes source truth, and a write acknowledgement becomes verified publication.

## Workload routing: use the least dynamic adequate controller

| Workload | Default controller | Escalate only when |
|---|---|---|
| Exact identifier mapping in a stable namespace | Deterministic lookup and versioned crosswalk | Collisions, expired identifiers, or namespace ambiguity exist |
| Known normalization and matching rules | Deterministic rules plus clerical-review band | Recall or drift evidence shows the rules miss material cases |
| High-volume record linkage | Blocking plus calibrated probabilistic/statistical matcher | Unstructured descriptions or novel conflicts require bounded model assistance |
| Catalog drift | Deterministic source-versus-catalog diff | Ownership or semantic interpretation is unclear |
| RDF conformance | SHACL validation using pinned shapes and entailment setting | A human-readable repair explanation or proposed shape is useful |
| OWL entailment/consistency | Pinned reasoner and declared OWL profile | Never outsource logical correctness to an LLM |
| Schema compatibility | Registry-native compatibility test plus consumer/semantic tests | A breaking migration needs an owner decision |
| Vocabulary alignment | Exact mappings, SKOS relations, lexical/structural candidates | Domain semantics need curator judgment |
| Repeated publication sequence | Deterministic workflow/state machine | Novel evidence changes the plan, not the authority |
| Natural-language explanation | Bounded model over a compiled evidence manifest | Raw unrestricted graph access is unnecessary |

OWL follows an open-world assumption, so absence of a fact does not generally make it false; the [OWL 2 Primer](https://www.w3.org/TR/owl2-primer/) explicitly distinguishes this from database-style closed-world behavior. SHACL accepts a data graph and shapes graph and produces a validation report; the [SHACL Recommendation](https://www.w3.org/TR/shacl/) requires those input graphs to remain immutable during validation. Treating either mechanism as a universal data-repair engine is a category error.

## When not to use an agent

Start with three non-agent designs and retain the simplest one that meets the harm, quality, and workload objectives.

| Alternative | Build it when | Evidence that it is sufficient | Escalation signal |
|---|---|---|---|
| Deterministic validator/rules service | Identifiers, constraints, normalizations, mappings, and dispositions are expressible as reviewed rules | Held-out precision/recall, rule coverage, stable exception volume, reproducible explanations, and acceptable maintenance effort | A material review band remains after rules are tuned, and its cases require heterogeneous evidence rather than another rule |
| Data-quality or metadata pipeline | Work is periodic profiling, schema/SHACL validation, catalog diffing, lineage ingestion, or an idempotent repair under one owner | Snapshot coverage, freshness, rule-result accuracy, repair success, and replay/rebuild tests meet objectives | Root cause or repair target repeatedly depends on ambiguous cross-system semantics |
| Human curator workflow | Volume is low, harm is high, evidence is sensitive, or policy requires expert judgment | Queue SLO, reviewer agreement, decision quality, appeal rate, and cost are acceptable | Evidence assembly—not the decision itself—dominates curator time and can be safely bounded |

Do **not** add a model-directed loop merely because descriptions are natural language, a graph exists, or a product advertises agents. Stop at the non-agent design when any of these holds:

- deterministic processing resolves the high-risk cases and the remaining volume fits the curator queue;
- the proposed model cannot demonstrate temporal, unseen-entity, or human-workload uplift over the strongest simple baseline;
- the source/target cannot expose stable identity, version, visibility, deletion, or read-back semantics;
- the organization cannot staff approvals, appeals, reconciliation, and incident recovery;
- evidence rights or provider controls prohibit the necessary processing;
- an agent would only move a fixed sequence that belongs in a normal workflow engine.

An adoption review records the baseline release, evaluation slice, unresolved-case volume, median and tail review minutes, expected harm, model/tool cost, authority boundary, and a dated decision. Re-run it when source mix, policy, model, or reviewer capacity changes. Stage 0 in the [delivery guide](09-evaluation-failure-injection-and-staged-delivery.md#stage-0--deterministic-foundation-and-non-agent-alternative) is a production endpoint, not a temporary demo.

## Why an agent loop can be justified

Stage 0 should attempt ordinary software first. The model-directed loop is justified only if a measured workload contains recurring cases where:

- evidence is distributed across heterogeneous systems and cannot be assembled by one fixed query;
- ambiguity requires iterative retrieval of type, temporal, ownership, or provenance context;
- conflict explanations and repair alternatives materially reduce steward effort;
- fixed rules achieve acceptable precision but leave a costly clerical-review band;
- semantic impact depends on natural-language definitions or heterogeneous descriptions that deterministic validators cannot interpret alone.

Even then, the loop remains bounded. A normal case plan is:

1. canonicalize the case scope and snapshot target versions;
2. collect the minimum authorized evidence;
3. run deterministic identifiers, constraints, compatibility, and validation checks;
4. generate candidates using a pinned matcher;
5. use a model only for an unresolved, explicitly typed subtask;
6. assemble alternatives, counter-evidence, and an impact preview;
7. stop for review, insufficiency, or a deterministic no-change result;
8. after approval, publish the exact change set and reconcile.

The loop has hard limits on candidate count, source reads, graph hops, sensitive fields, model calls, tokens, wall time, cost, and replan count. Exhaustion returns `INSUFFICIENT_EVIDENCE` or `REVIEW_REQUIRED`, not a lower-quality automatic decision.

## Integration topology

The controller should adapt to the existing system of record rather than install a new universal graph.

| System class | Read contract | Write contract | Important caveat |
|---|---|---|---|
| Catalog: DataHub, OpenMetadata, Atlas, commercial catalog | Entity/aspect/FQN/GUID, version, ownership, tags/terms, lineage, audit | Draft/change proposal or exact metadata patch | Models and API capabilities differ; capability-probe the deployed version |
| Graph/RDF store | Named graph/dataset, query endpoint, ontology and entailment configuration | Staging graph or approved transaction | RDF dataset graph-name semantics are application-defined; named graph is not automatically provenance |
| MDM hub | Source members, match evidence, canonical ID, survivorship, merge/split history | Approved link/unlink/merge/split command | MDM match does not grant source-record mutation authority |
| Lineage system/OpenLineage consumer | Job/run/dataset events, facets, producer, event time, coverage | Manual/corrective candidate edge or event | An observed edge may be incomplete, duplicated, late, or parser-derived |
| Schema registry | Subject/artifact, schema type, version, references, rule/compatibility config | New version after owner approval | Compatibility is format- and scope-specific, not business semantics |
| Vocabulary/ontology repository | Stable/version IRIs, imports, term status, mappings, axioms, shapes | Reviewed new version and migration artifacts | `owl:sameAs` is stronger than similarity; SKOS mappings have distinct semantics |
| Data-quality platform | Test definition, target, run, metric, result, time | Approved repair task or test proposal | A failed test is an observation, not a root cause |
| Source system | Stable source key, record version, effective time, allowed fields | Repair ticket or separately owned patch workflow | Do not bypass the source owner or copy excess PII into the graph |
| Third-party identifier/reconciliation service | Versioned response, candidate ID, type, score, terms/license | Usually none | Service score scales are not portable and the W3C 0.2 report is not a Recommendation |

The [W3C Reconciliation Service API 0.2](https://www.w3.org/community/reports/reconciliation/CG-FINAL-specs-0.2-20230410/) is useful as a candidate-search interoperability shape: it returns ranked candidates with identifiers, names, types, and scores. Its status and loose score semantics mean it must be wrapped by local evidence, authorization, calibration, and review contracts.

## Representation and storage boundaries

Choose a representation per invariant and access pattern. A stewardship architecture may use several; it should not force every concern into one graph model.

| Boundary | Prefer it for | Keep elsewhere or add explicitly |
|---|---|---|
| RDF 1.1 dataset plus SPARQL 1.1 | Interoperable identifiers, vocabulary/ontology data, statement-level exchange, standards-based query and reasoning | Workflow locks, approvals, effect attempts, large evidence blobs, and product-specific concurrency; graph-name meaning and provenance need an application convention |
| OWL 2 reasoner | Declared entailment under a pinned profile and ontology bundle | Closed-world acceptance, source authority, confidence, access policy, and repair choice |
| SHACL 2017 validator | Bounded conformance of a declared data graph against pinned shapes and entailment mode | Truth, completeness beyond the declared snapshot, or unique repair |
| JSON-LD 1.1 interchange | JSON-facing APIs that need explicit RDF term expansion/compaction | Stable semantics if remote contexts are mutable; pin/cache context bytes and processing mode before trusting a digest |
| Property graph and Cypher/Gremlin-style query | Operational neighborhood traversal, impact analysis, path constraints, and application-native properties | Cross-vendor semantic equivalence, RDF entailment, statement provenance, bitemporal history, and review/effect state unless explicitly modeled |
| Relational/event ledger | Case transitions, bitemporal records, approvals, leases/fences, outbox, effects, and invariant enforcement | Large graph traversal and ontology reasoning |
| Search/vector index | Lexical or semantic candidate retrieval over an authorized projection | Identity, canonical facts, deletion completion, ACL truth, and reproducible ordering |

RDF 1.2 Concepts and Semantics reached Candidate Recommendation Snapshot on 7 April 2026, but SPARQL 1.2 remains a Working Draft. A deployment may qualify selected 1.2 features, but interchange should advertise the actual syntax, data model, and entailment regime rather than assuming “RDF 1.2” means every query/update implementation behaves alike. The operation probes in the [tool-contract guide](06-state-context-memory-planning-and-tool-contracts.md#operation-level-adapter-qualification) are mandatory.

## Deployment units

A reliable first deployment needs only five independently scalable responsibilities:

1. **API and admission** — authenticate, authorize, canonicalize scope, issue run/case IDs, and reject unsafe or over-budget work.
2. **Case worker** — execute the deterministic workflow and call read-only analysis adapters.
3. **Evidence/matching workers** — perform blocked comparisons, reasoning, validation, graph analysis, and bounded model calls in isolated pools.
4. **Approval/publisher worker** — consume exact approved envelopes with narrower credentials than analysis workers.
5. **Reconciler** — read authoritative targets, resolve ambiguous outcomes, detect projection drift, and close or reopen cases.

A durable workflow engine is optional. Use one when approval waits, timers, retries, or recovery exceed a process lifetime. Its journal coordinates steps; it does not create an atomic transaction across a catalog, MDM hub, graph, and registry. Follow the repository's [durable execution](../../runtime/durable-execution.md) and [idempotency](../../reliability/idempotency-and-side-effects.md) contracts.

## Selection scorecard

Before selecting a product or model, run a deployment-specific proof against these criteria:

| Criterion | Required evidence |
|---|---|
| Identity semantics | Namespace, source key, canonical key, alias, merge/split, and temporal behavior are explicit |
| Change history | Prior versions and actor/source provenance are queryable and exportable |
| Approval seam | Proposals can remain drafts and cannot be approved by the proposing workload identity |
| Concurrency | Optimistic version/precondition or equivalent conflict detection exists |
| Deletion | Soft delete, purge, tombstone, restore, and descendant behavior are documented and tested |
| Lineage | Producer, event/run identity, granularity, rename/drop semantics, and coverage are observable |
| Schema/ontology | Versions, imports/references, compatibility/validation, and migration are pinned |
| Security | Read/write privileges, search visibility, tenant/domain filters, bot identity, and audit are enforceable |
| Recovery | Export/rebuild, replay, reconciliation, and ambiguous-write handling have been tested |
| Scale | Candidate-volume, graph-degree, search-index, review-queue, and API-rate limits are known |

## Anti-patterns

| Anti-pattern | Failure created |
|---|---|
| “One graph is the source of truth” without field-level authority | Conflicting systems silently overwrite one another |
| LLM directly calls merge/update APIs | Similarity, prompt injection, or hallucinated IDs become canonical mutations |
| Store only the winning candidate | Reviewers cannot see alternatives, blocking misses, or negative evidence |
| Use `owl:sameAs` for “looks similar” | Equality inference spreads incorrect identity through the graph |
| Infer lineage from missing/nearby names | False impact paths or false assurance |
| Let the publisher choose a target at execution time | Approval no longer binds the actual effect |
| Put raw PII and credentials into prompts/traces | Secondary privacy breach and uncontrolled retention |
| Add multi-agent orchestration for fixed review steps | More nondeterminism without added authority or evidence |

## Readiness checklist

- [ ] The category boundary and named canonical owners are documented.
- [ ] Deterministic validator/rules, data-quality pipeline, and human-curator alternatives were measured before adding a model loop.
- [ ] Representation choices place semantic, workflow, effect, evidence, and retrieval concerns in appropriate stores.
- [ ] Every source and target has a capability/version manifest.
- [ ] Assertion, observation, candidate, decision, canonical fact, provenance, and effect are separate records.
- [ ] Candidate generation cannot publish, approve, or mint authority.
- [ ] High-impact decisions require a current, independent reviewer.
- [ ] The publisher accepts only typed, digest-bound, version-preconditioned effects.
- [ ] Reconciliation, reversal, deletion, and incomplete-lineage behavior are tested.
- [ ] Canonical cross-cutting controls are linked rather than reimplemented in prose.

## Related canonical guidance

- [Agent state and event contracts](../../runtime/agent-state-and-event-contracts.md)
- [Tool contracts](../../tools/tool-contracts.md)
- [Tool results, artifacts, and provenance](../../tools/tool-results-artifacts-and-provenance.md)
- [Planning and replanning](../../orchestration/planning-and-replanning.md)
- [Agent threat model](../../security/agent-threat-model.md)
- [Trajectory and reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md)

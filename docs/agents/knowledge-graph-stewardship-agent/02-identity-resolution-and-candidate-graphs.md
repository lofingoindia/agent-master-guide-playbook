# Identity Resolution and Candidate Graphs

Entity resolution asks whether records refer to the same real-world entity **for a declared entity type, identifier regime, purpose, and time**. It is not string deduplication, vector search, or graph clustering alone.

The foundational Fellegi–Sunter formulation has three outcomes—link, non-link, and possible link—under explicit error levels. That clerical-review region remains a better production mental model than forcing every pair into a binary answer; see the original [1969 record-linkage paper](https://nhis.ipums.org/nhis/resources/Fellegi69.pdf).

## Start with the identity contract

Before comparing data, define:

| Field | Required decision |
|---|---|
| Entity type | Person, legal organization, household, product model, product instance, facility, dataset, table, column, metric, ontology concept, or another bounded class |
| Equivalence relation | “Same legal entity,” “same marketed product,” “same physical item,” and “same household” are different relations |
| Namespace | Authority and syntax for every source, canonical, external, and alias identifier |
| Temporal semantics | Whether identity is stable, effective-dated, version-specific, or allowed to split/merge over time |
| Match policy | Required exact fields, forbidden conflicts, probabilistic features, thresholds, and review bands |
| Cardinality | One-to-one, many source members to one canonical entity, one record in multiple grouping entity types, or another constraint |
| Survivorship | Which owner chooses canonical attributes; matching does not imply attribute survivorship |
| Risk | Cost of false merge, false split, missed match, delayed decision, and privacy exposure |
| Authority | Who can accept link, unlink, merge, split, and canonical-ID consequences |

A single person record can legitimately participate in a “person” cluster and a separate “household” cluster. IBM's current MDM documentation likewise describes entity-type-specific algorithms and distinct groupings; it also documents that add, update, and delete events can trigger joins, singleton creation, merges, and splits. This is evidence that cluster composition is mutable—not a universal guarantee supplied by a match score ([IBM matching algorithms](https://www.ibm.com/docs/en/ws-and-kc?topic=data-matching-algorithms)).

## Identifier model

An identifier is not just a string:

~~~yaml
identifier:
  namespace: "erp-eu/vendor"
  value: "V-00204"
  entity_type: "legal-organization"
  issuer: "erp-eu"
  tenant_id: "retail-eu"
  valid_from: "2024-01-01T00:00:00Z"
  valid_to: null
  status: "active"          # active | retired | reassigned | provisional
  source_record_version: "etag:9ac1"
  observed_at: "2026-08-31T02:00:00Z"
  evidence_ref: "ev_01K..."
~~~

Rules:

- equality is `(tenant, namespace, entity_type, normalized_value, valid_interval)`, not `value` alone;
- identifier normalization must be versioned and preserve the original value;
- reassigned phone numbers, recycled account IDs, renamed datasets, and versioned ontology IRIs require effective time;
- a global external identifier is exact only if its issuer and entity-type semantics are trusted for this purpose;
- source identifiers remain attached to source members after a merge so lineage and reversal remain possible;
- canonical IDs are minted by the canonical owner, never invented by the model.

Blank-node labels are not portable identifiers. RDF specifications treat blank-node identifiers as serialization-local rather than part of the abstract syntax. Skolemization may help system interchange, but a generated IRI still needs a documented issuer and lifecycle; it does not prove real-world identity.

### Identity changes do not rewrite source truth

Keep four independently queryable layers:

1. the source record version as received;
2. assertions extracted from that record for a declared valid interval;
3. identity membership or co-reference decisions effective for a declared interval;
4. canonical facts selected from assertions under a survivorship policy.

A source record can be corrected without erasing what was previously received. A membership decision can be reversed without deleting the member's assertions. A canonical fact can change without changing identity. Conversely, a split can require recomputing facts for both resulting entities even when no source value changed.

Use half-open valid-time intervals `[valid_from, valid_to)` and append-only transaction-time versions `[recorded_from, recorded_to)`. `valid_to = null` means open-ended, not immortal; `recorded_to = null` means the version is the current ledger view. Corrections close transaction time on the old version and append a `correction_of` or `supersedes` version; they do not backdate the ledger. The canonical envelope and per-record rules are in the [record and time contract](06-state-context-memory-planning-and-tool-contracts.md#canonical-record-and-time-contract).

Conflict preservation is an invariant: if two authorized sources disagree, retain both assertions with their authority scopes, intervals, and provenance; compute a named effective view or open a case. Do not overwrite the losing assertion, average categorical claims, or encode “confidence” as if it resolved authority.

## Candidate-generation pipeline

~~~mermaid
flowchart LR
    A["Source records + versions"] --> N["Typed normalization"]
    N --> X["Exact IDs / crosswalks / hard rules"]
    N --> B["Blocking and retrieval"]
    B --> F["Feature computation"]
    X --> C["Pair candidates"]
    F --> S["Calibrated scorer"]
    S --> C
    C --> H["Hard constraints + negative evidence"]
    H --> P{"Policy band"}
    P -->|"non-link"| NL["Recorded non-link / suppressor"]
    P -->|"possible"| RV["Steward review"]
    P -->|"eligible link"| CL["Cluster simulation"]
    CL --> RV
    RV --> D["Decision ledger"]
~~~

### 1. Normalize without laundering evidence

Create typed observations such as normalized name, parsed address, transliteration, token set, unit conversion, canonical schema path, or resolved vocabulary label. Store:

- the source assertion and source record version;
- transformation name/version and locale;
- normalized value and parse warnings;
- data classification and retention policy;
- whether the field was absent, null, malformed, withheld, or inaccessible.

Those missingness states are not equivalent. “Withheld by policy” must not become a mismatch; “not observed due to connector failure” must not become a negative signal.

### 2. Resolve exact evidence first

High-value deterministic evidence includes:

- trusted identifier in the same namespace and overlapping validity interval;
- governed source-to-canonical crosswalk;
- owner-approved alias or supersession mapping;
- exact composite business key whose uniqueness is monitored;
- explicit prior link or non-link decision still valid under the current policy/version.

Hard conflicts might include two mutually exclusive legal identifiers, incompatible entity types, non-overlapping product versions where identity requires overlap, or a prior reviewed non-link. Hard constraints are application rules, not model instructions.

### 3. Block for recall, then measure the misses

All-pairs comparison grows quadratically. Splink's official blocking guide illustrates that one million records imply roughly 500 billion undirected comparisons before blocking and explains the recall/cost trade-off of multiple strict rules ([blocking rules](https://moj-analytical-services.github.io/splink/topic_guides/blocking/blocking_rules.html)).

A production blocking manifest records:

~~~yaml
blocking_manifest:
  version: "supplier-block-v8"
  entity_type: "legal-organization"
  rules:
    - id: "tax-country-hash"
      expression_ref: "rule:sha256:..."
      sensitive_inputs: ["tax_id", "country"]
    - id: "name-postcode-prefix"
      expression_ref: "rule:sha256:..."
    - id: "ann-description"
      index_version: "org-embed-2026-08-15"
      top_k: 20
  pair_deduplication: true
  max_block_size: 10000
  overflow_action: "quarantine_hot_block"
  evaluated_recall: 0.997
  evaluation_slice: "prod-temporal-holdout-2026q2"
~~~

Monitor pair completeness/recall against labeled matches, not only reduction ratio. A rule can be fast because it silently excludes the hardest true matches. Sample and review `no_candidate` records. Treat hot blocks—common names, empty keys, default values, high-degree graph nodes—as a separate workload with caps, salting/chunking where supported, and human-safe degradation.

### 4. Score evidence, not prose

Each comparison feature has a direction and provenance:

| Feature class | Examples | Production caution |
|---|---|---|
| Exact | trusted ID, normalized domain, schema field ID | Verify namespace, issuer, uniqueness, and time |
| String | name/address similarity, acronym expansion | Locale and transliteration can create demographic bias |
| Numeric/temporal | date distance, geospatial distance, overlapping intervals | Units, time zones, precision, and effective dates matter |
| Categorical | type, country, source system, product family | A rare agreement can be stronger than a common one |
| Relational | shared verified parent, owner, address, lineage neighbor | Avoid circular evidence from already inferred links |
| Embedding | description or label neighbor similarity | Candidate retrieval only; scores are model/index dependent |
| Negative | incompatible ID, mutually exclusive type, reviewed non-link | Define whether it is hard, weighted, expiring, or appealable |
| Missingness | absent/withheld/unparseable/unavailable | Never collapse these states |

The candidate must store raw feature observations and a reproducible score explanation. If a language model contributes a feature, store its typed output, evidence span references, prompt/model version, and validation result; do not store free-form reasoning as the sole explanation.

### 5. Calibrate policy bands by cost and slice

The same score threshold should not govern every entity type or effect. Define at least:

- `auto_non_link`: sufficiently strong evidence to avoid further work for this policy version;
- `clerical_review`: plausible link with unresolved material uncertainty;
- `link_eligible`: eligible for a deterministic policy or human decision, never automatically authoritative because of score alone;
- `deny`: violates a hard constraint or scope boundary.

Calibrate probability-like outputs on representative, time-split production data. Report reliability curves and expected calibration error where meaningful, plus precision/recall at operational thresholds. A score from one matcher, tenant, source pair, or version is not portable to another.

## Candidate contract

~~~yaml
candidate_link:
  candidate_id: "cand_01K..."
  case_id: "case_01K..."
  tenant_id: "retail-eu"
  domain_id: "supplier"
  entity_type: "legal-organization"
  relation: "same_legal_entity"
  left:
    record_ref: "crm-eu/account/918"
    version: "etag:71"
    effective_at: "2026-08-01T00:00:00Z"
  right:
    record_ref: "erp-eu/vendor/V-00204"
    version: "etag:9ac1"
    effective_at: "2026-08-01T00:00:00Z"
  generation:
    blocking_manifest: "supplier-block-v8"
    match_policy: "supplier-match-v12"
    scorer: "fs-org-eu-v5"
    model_release: null
  score:
    value: 0.972
    calibration_slice: "crm-erp-eu-2026q2"
    policy_band: "clerical_review"
  evidence:
    positive: ["ev_tax_hash", "ev_name", "ev_address"]
    negative: ["ev_incorporation_date_gap"]
    missing: ["ev_registry_id_unavailable"]
    alternatives: ["cand_01K_ALT"]
  constraints:
    violations: []
    warnings: ["source_address_shared_by_17_entities"]
  proposed_at: "2026-08-31T02:10:00Z"
  expires_at: "2026-09-07T02:10:00Z"
  status: "REVIEW_REQUIRED"
~~~

Required properties:

- immutable candidate versions; corrections create a superseding candidate;
- exact source versions or snapshot IDs;
- alternatives and negative/missing evidence;
- policy, blocking, feature, scorer, normalization, model, graph, and ontology versions;
- a meaningful expiry when source or identity data can change;
- no `canonical_id` until the canonical owner provides or approves one.

## Pair decisions do not automatically define clusters

If A–B and B–C are accepted, A–C may still violate constraints. Single-linkage connected components can create “glue record” merges where one ambiguous record joins otherwise distinct entities. Before proposing a cluster mutation, simulate:

- component size and diameter;
- conflicting trusted identifiers;
- cardinality violations;
- duplicate source-system membership rules;
- temporal overlap and gaps;
- edge confidence distribution and weakest bridge;
- reviewed non-link edges inside the proposed component;
- attribute survivorship conflicts;
- downstream references, policies, contracts, and affected cases;
- reversal cost if the decision is wrong.

Use an explicit cluster policy: connected components, correlation clustering, constrained hierarchical clustering, one-to-one assignment, or an owner-defined alternative. Pin its version and validate global constraints after pair scoring.

~~~mermaid
graph LR
    A["A: CRM 918"] ---|"0.99 exact tax ID"| B["B: ERP V-204"]
    B ---|"0.74 shared address"| C["C: CRM 1441"]
    A -.->|"conflicting legal ID"| C
~~~

The B–C edge cannot make A and C identical merely because connected components are convenient.

Accepted identity edges also have a relation, scope, and interval. `same_legal_entity`, `same_product_model`, `version_of`, `alias_of`, and `successor_of` are not interchangeable. Store an accepted edge as a versioned decision-derived relationship; never mutate the original candidate into an accepted fact. If policy requires equivalence, test reflexivity, symmetry, and transitivity at the cluster layer. If the relation is directional or non-transitive, do not run connected components over it.

## Merge, split, and unlink contracts

### Proposal

~~~yaml
identity_change_proposal:
  proposal_id: "prop_01K..."
  proposal_version: 3
  operation: "MERGE"       # LINK | UNLINK | MERGE | SPLIT | REASSIGN_ALIAS
  scope:
    tenant_id: "retail-eu"
    entity_type: "legal-organization"
    canonical_system: "mdm-eu"
  expected_versions:
    canonical_entities:
      - {id: "org-501", version: "231"}
      - {id: "org-884", version: "98"}
    member_snapshot: "sha256:..."
  member_plan:
    retain: ["crm-eu/account/918", "erp-eu/vendor/V-00204"]
    move: ["registry-eu/company/DE129..."]
    unlink: []
  canonical_id_action: "owner_selects_survivor"
  survivorship_plan_ref: "surv_01K..."
  alias_and_redirect_plan_ref: "alias_01K..."
  downstream_impact_ref: "impact_01K..."
  evidence_manifest: "sha256:..."
  rollback_plan_ref: "rollback_01K..."
  proposal_digest: "sha256:..."
  requires_roles: ["supplier-data-owner", "mdm-steward"]
  proposer_principal: "workload:kg-steward-v4"
  self_approval_forbidden: true
~~~

### Decision

~~~yaml
curator_decision:
  decision_id: "dec_01K..."
  proposal_id: "prop_01K..."
  proposal_version: 3
  proposal_digest: "sha256:..."
  decision: "APPROVE"      # APPROVE | REJECT | DEFER | REQUEST_CHANGES
  decided_by: "user:steward-41"
  authority_roles: ["supplier-data-owner"]
  decided_at: "2026-08-31T03:00:00Z"
  expires_at: "2026-09-01T03:00:00Z"
  rationale_code: "VERIFIED_SHARED_LEGAL_ID"
  rationale_ref: "note_01K..."
  policy_version: "identity-approval-v6"
  use_count: 1
  signature_ref: "sig_01K..."
~~~

### Effect receipt

~~~yaml
identity_change_receipt:
  effect_id: "retail-eu/mdm-eu/merge/prop_01K/v3"
  request_digest: "sha256:..."
  target: "mdm-eu"
  adapter_version: "mdm-adapter-2.4.1"
  dispatch_state: "COMMITTED"
  remote_operation_id: "op-775901"
  survivor_id: "org-501"
  retired_ids: ["org-884"]
  target_version_after: "232"
  observed_members_after: "sha256:..."
  reconciliation: "VERIFIED"
  verified_at: "2026-08-31T03:01:19Z"
  projection_repairs: ["catalog-job-771", "graph-job-880"]
~~~

The effect identity is derived from tenant, target, semantic operation, proposal ID, and version. A timeout after dispatch enters `UNKNOWN` and is reconciled by remote operation ID, target versions, aliases, and member state before any retry.

## Split is a first-class operation

A merge history is insufficient if the platform cannot represent reversal. A split proposal must define:

- the prior decision/effect being corrected;
- partitioned member sets and any new canonical IDs to be minted by the owner;
- which attributes were inherited or edited after the merge;
- how downstream references are reassigned, duplicated, or held for review;
- whether facts asserted about the merged entity can be safely redistributed;
- policy/classification/consent impact;
- aliases and redirect expiry;
- affected evaluation labels and suppressors;
- communication and appeal records.

Do not assume that rollback restores the previous world. New facts and downstream writes may have accumulated. Treat a split as a new forward change with provenance to the corrected merge.

## Identity drift and temporal correction

Re-run or reopen cases when:

- a trusted identifier is added, revoked, or reassigned;
- a source record changes a high-weight feature;
- a match/normalization/blocking model changes;
- a reviewed decision expires or is appealed;
- a canonical entity is merged, split, deleted, or retyped;
- an ontology changes the entity type or equivalence relation;
- a new source creates a bridge or hard conflict;
- distribution/calibration monitoring crosses a threshold.

Never rewrite history to make an old decision appear correct under a new model. Preserve transaction time (`recorded_at`) and domain-valid/effective time (`valid_from`, `valid_to`) separately. A superseding decision explains what was known then and what changed now.

## Embeddings and graph signals: useful but non-authoritative

Embedding nearest neighbors are valuable blocking/retrieval signals for unstructured descriptions, multilingual labels, and sparse attributes. They are unsafe as identity truth because:

- vector similarity is model-, index-, metric-, preprocessing-, and top-k-dependent;
- nearest-neighbor results can change after re-embedding or index rebuild;
- hubness and common templates can make unrelated records close;
- two different real-world objects may be semantically identical in description;
- identical entities may have dissimilar sparse or multilingual representations;
- access-filtered evidence can change the representation;
- a vector does not express namespace, temporal validity, cardinality, or legal identity.

Knowledge-graph embeddings optimize representation-learning tasks, not canonical identity. Benchmarks such as [WDC Products](https://openproceedings.org/2024/conf/edbt/paper-14.pdf) explicitly vary corner cases, unseen entities, and label volume; evaluated systems struggle on unseen entities. OpenEA and later evaluations also show that benchmark structure and name bias affect entity-alignment results. Therefore use production-like temporal and unseen-entity evaluation before trusting a candidate generator.

## OWL identity and SKOS mapping are not substitutes for review

`owl:sameAs` denotes equality of individuals under OWL semantics. Its consequences can propagate broadly through reasoning. Research on the [sameAs problem](https://semantic-web-journal.net/content/sameas-problem-survey-identity-management-web-data) documents how erroneous automatically created links damage decentralized graphs.

For concept schemes, SKOS provides mappings with different strength:

- `skos:closeMatch` is intentionally not transitive;
- `skos:exactMatch` is transitive and expresses high interchangeability across information-retrieval applications;
- broader/narrower/related mappings are not identity;
- `skos:exactMatch` is not a casual alias for `owl:sameAs`.

The [SKOS Reference](https://www.w3.org/TR/skos-reference/) defines these relations and their integrity conditions. A curator should select the relation that matches the intended consumption semantics.

## Evaluation

Measure the entire operating point, not just pairwise F1:

| Layer | Metrics |
|---|---|
| Blocking | pair completeness/recall, reduction ratio, hot-block rate, zero-candidate rate, comparisons per record |
| Pair scoring | precision, recall, PR-AUC, calibration, false-link/false-non-link cost, coverage at review thresholds |
| Clustering | B-cubed or pairwise precision/recall, cluster purity/completeness, overmerge/oversplit rate, constraint violations |
| Temporal | accuracy on new/unseen entities, changed records, reassigned IDs, delayed evidence, post-model-upgrade cases |
| Operations | review minutes/case, queue age, appeal/reversal rate, stale approval rate, unknown effects, reconciliation lag |
| Safety/fairness | cross-tenant candidate rate (must be zero), sensitive-field exposure, subgroup error slices, injection-following rate |
| Business outcome | duplicate reduction verified by owners, downstream incident rate, repair completion, avoided false merges |

Evaluate by entity type, source pair, locale/script, data completeness, block size, cluster size, identifier availability, protected/sensitive-data slice, and time. An aggregate score can hide catastrophic false merges in a rare but high-impact class.

## Failure tests

- Same name and address, different trusted legal IDs.
- Shared family phone/address across distinct people.
- Reassigned phone or account ID after a validity gap.
- One “glue” record joins two internally incompatible clusters.
- Exact external ID appears in the wrong namespace.
- Candidate generator omits the true match because all blocking fields contain typos.
- Common-name block creates a memory/skew explosion.
- Embedding index upgrade changes the top candidate.
- Model explanation cites a field it was not authorized to read.
- Previously rejected pair is re-proposed without showing the suppressor.
- Merge succeeds remotely but the acknowledgement is lost.
- Split is requested after downstream facts were added to the merged entity.
- Source deletion arrives before the canonical/lineage projections update.
- Label leakage places members of the same entity in train and test.

## Checklist

- [ ] Identity relation, namespace, entity type, time, cardinality, and owner are explicit.
- [ ] Source records, assertions, membership decisions, and canonical facts are bitemporally separate and conflicts are preserved.
- [ ] Normalization preserves source values and distinguishes missingness states.
- [ ] Blocking recall and hot-block behavior are measured.
- [ ] Scores are calibrated per source/entity slice and never treated as authority.
- [ ] Negative evidence, alternatives, and prior decisions remain visible.
- [ ] Pair and cluster policies are separately versioned and evaluated.
- [ ] Merge, split, unlink, alias, and reversal are implementable contracts.
- [ ] Embeddings and graph signals remain candidate generators.
- [ ] `owl:sameAs`, SKOS mappings, and application identity are not conflated.
- [ ] Temporal drift, appeals, model upgrades, and ambiguous effects reopen cases safely.

## Related guidance

- [Steward cases and merge/split workflows](05-steward-cases-quality-repair-and-merge-split-workflows.md)
- [State, context, and tool contracts](06-state-context-memory-planning-and-tool-contracts.md)
- [Evaluation and staged delivery](09-evaluation-failure-injection-and-staged-delivery.md)
- [Idempotency and side effects](../../reliability/idempotency-and-side-effects.md)

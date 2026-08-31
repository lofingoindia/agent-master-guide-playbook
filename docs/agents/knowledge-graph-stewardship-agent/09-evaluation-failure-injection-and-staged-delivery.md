# Evaluation, Failure Injection, and Staged Delivery

Evaluate the complete governed workflow, not only a matcher or language model. The release unit is the pinned combination of source snapshot, normalizers, blocking policy, matcher/model, semantic bundle, context compiler, tools, authority policy, state schema, and target adapters.

## Evaluation contract

```yaml
evaluation_run:
  run_id: eval-2026-09-rc1
  release_manifest: steward-2026.09.1
  dataset_manifest: kg-steward-benchmark-12
  temporal_cutoff: 2026-06-30
  slices: [tenant-size, entity-type, source-pair, language, missingness, risk-tier]
  modes: [offline-replay, shadow, fault-injection]
  prohibited_overlap_checks: [entity, cluster, source-record, near-duplicate]
  output_uri: evidence://eval/eval-2026-09-rc1
  evaluator_build: steward-eval-7.2.0
```

Prevent leakage at entity and cluster level, not only row level. Use temporal holdouts to simulate new sources and changed schemas. The WDC Products benchmark reports that entity matchers still struggle with unseen entities and corner cases, which is why random pair splits can be dangerously optimistic; see the primary [WDC Products benchmark paper](https://openproceedings.org/2024/conf/edbt/paper-14.pdf).

## Evaluation layers

| Layer | Required measures |
|---|---|
| Normalization | Determinism, locale/date/unit correctness, raw-value preservation, collision rate |
| Blocking | Pair completeness/blocking recall, reduction ratio, hot-block rate, cost by slice |
| Pair matching | Precision/recall, PR curve, calibrated risk, review-band yield, abstention, expected error cost |
| Clustering | B-cubed/pairwise/cluster metrics, overmerge/undermerge, largest erroneous cluster, constraint breaches |
| Merge/split | Assertion assignment correctness, survivorship correctness, affected-reference completeness, reversibility |
| Schema/ontology | Parse/consistency, expected/forbidden entailments, competency questions, query/closure/validation deltas |
| Vocabulary | Label/integrity tests, mapping relation correctness, concept replacement/reference integrity |
| Catalog/lineage | Asset identity, edge precision/recall by evidence type, lifecycle handling, completeness/unknown calibration |
| Data quality | Rule precision/recall, root-cause classification, repair-target correctness, exception behavior |
| Context/model | Evidence citation, contradiction retention, abstention, schema validity, injection resistance, cost/latency |
| Workflow/tools | Transition legality, authorization, approval binding, idempotency, ambiguous effects, reconciliation |
| Operations | Throughput, queue age, tail latency, recovery time, resource/cost, noisy-tenant isolation |
| Security/privacy | Cross-tenant and field leakage, inference leakage, separation of duties, retention/deletion completeness |

## Four evaluation families

Do not let an accurate final label hide an unsafe path or exhausted steward organization.

| Family | Unit and questions | Required evidence |
|---|---|---|
| Outcome | Was the final link/non-link, cluster, fact, semantic release, repair, invalidation, and reconciled target state correct? | Gold/gray labels, owner-confirmed postconditions, cost-weighted errors, downstream incidents, appeals/reversals |
| Trajectory | Did the run retrieve only permitted evidence, preserve alternatives, respect budgets, abstain appropriately, stop for approval, and reconcile effects? | Ordered state/tool/policy events, context manifests, citations, tool statuses, retries/replans, effect attempts, repeated-run distributions |
| Invariant | Could any execution cross tenant/authority, corrupt time/version history, self-approve, lose contradictions, reuse stale approval, duplicate an effect, or mark unknown as success? | Property/model-based tests, adversarial inputs, concurrency histories, fault injection, replay/restore checks, zero-tolerance gate results |
| Human factor | Can stewards detect errors, understand provenance/uncertainty, use alternatives/abstain/appeal, and sustain the queue without automation bias or overload? | Randomized/blinded UI studies where feasible, handling time, error/disagreement, override/reopen behavior, evidence requests, workload and usability measures |

Report outcome and trajectory jointly. A correct link reached after cross-tenant retrieval fails. A safe abstention on insufficient evidence may be a correct trajectory even when a later human establishes the identity.

## Gold, gray, and challenge identity sets

Maintain three separately versioned sets:

| Set | Admission | Use | Prohibition |
|---|---|---|---|
| Gold | Independent adjudication from authoritative identifiers/source records, explicit relation/entity type/time, cluster membership, and rationale; disagreements resolved or excluded | Primary threshold, pair/cluster/split, and release evaluation | Do not infer labels from current model, existing contaminated graph edge, or a single reviewer click |
| Gray | Genuine unresolved ambiguity, reviewer disagreement, incomplete evidence, time-dependent relation, or policy-specific answer; stores plausible alternatives and unknowns | Abstention, ranking, calibration under ambiguity, UI/human study, active evidence selection | Do not force binary truth or score it as ordinary false positive/negative |
| Challenge | Curated corner/adversarial cases: glue records, identifier reuse, aliases, multilingual/transliteration, shared contact points, hot blocks, unseen sources/entities, and poisoned metadata | Stress, fairness, security, and regression gates | Do not tune exclusively until it ceases to represent production prevalence |

Each label binds source-record revisions, valid-time question, transaction-time cutoff, relation, entity type, tenant/domain, evidence visible to the evaluator, adjudicators, rationale, confidence in the **label**, and rights/retention. Label confidence is not model confidence. Store pair labels and cluster partitions together so impossible pair combinations are detected.

Use entity-, cluster-, household/organization-, source-, and near-duplicate-disjoint splits as appropriate. Train/evaluate only on evidence recorded before the temporal cutoff; freeze future identifiers, decisions, graph edges, embeddings, and downstream facts. Maintain a cold unseen-entity/source set and a rolling forward-time set. Hash/near-duplicate and provenance checks should detect the same source payload, entity members, templated variants, or derived graph edge leaking across splits.

## Identity evaluation details

### Pair and decision bands

Report confusion/cost separately for:

- exact-rule automatic decisions;
- model/matcher auto-link band, if enabled;
- review band;
- auto-non-link band;
- abstentions and insufficient-evidence cases.

Tune thresholds on calibrated expected harm, not F1 alone. A false merge may be orders of magnitude more costly than a missed candidate for protected identities.

### Blocking

Measure whether each labeled true pair appears in **any** block. Report by source pair, entity type, missing field, transliteration/language, identifier corruption, and new/unseen entity. A strong pair scorer cannot recover candidates never generated.

### Clusters and splits

Pairwise accuracy can hide one catastrophic bridge that joins two large clusters. Report maximum false cluster size, number of source-uniqueness violations, and split reconstruction accuracy. Test glue-record/alias behavior where targets cannot reverse merges.

### Robust baselines

Compare sophisticated models with deterministic exact/normalized rules and a probabilistic Fellegi–Sunter-style baseline. DeepMatcher's primary paper found deep models especially useful for textual/dirty settings while traditional approaches remained competitive on structured data; the practical lesson is to earn model complexity by slice rather than assume it. See [DeepMatcher](https://pages.cs.wisc.edu/~anhai/papers1/deepmatcher-sigmod18.pdf).

Embeddings and graph similarity are ablated as candidate signals. They must not be evaluated only after a graph already contaminated by test labels or prior merges.

## Semantic and catalog evaluations

Maintain executable competency questions:

```yaml
competency_test:
  id: CQ-ORDER-017
  question: Which active datasets derive customer_email into externally shared outputs?
  graph_snapshot: eval-graph-31
  semantic_bundle: customer-domain-2026.09.0
  expected:
    required_assets: [urn:asset:partner_export]
    forbidden_assets: [urn:asset:internal_metrics]
  query_digest: sha256:...
  max_runtime_ms: 1500
```

Test both positive and negative entailments and both asserted-only and configured inferred views. A reasoner that returns the expected positive answer but adds thousands of unintended edges fails.

For lineage, label provenance class separately: runtime event, query parser, static mapping, manual assertion, or inference. Measure cycles, renamed assets, overwrites, missing permissions, truncated traversals, late events, and contradictory sources. Reward honest `unknown`; do not score all empty responses as true negatives.

For schema registry integration, test subject/version/ID/reference distinctions, normalization, compatibility levels, deleted versions, and incompatible target behavior. A registry compatibility pass must not cause an ontology or business-semantic auto-approval.

## Model and context evaluations

Score structured behavior:

- every cited evidence ID exists in the context manifest;
- every material claim is supported and contradictory evidence is surfaced;
- source text never overrides system/policy instructions;
- proposal action is legal for the case type and cannot assert approval/effect success;
- suggestions stay inside allowed read tools, graph depth, cost, and tenant scope;
- uncertainty/abstention rises when evidence is removed or conflicts are injected;
- compaction does not erase dissent, time, authority, or effect ambiguity;
- output schema, enum, size, and reason-code contracts hold under repeated runs.

Run each probabilistic case multiple times and report outcome distribution, not a single lucky response. Keep temperature/seed/provider behavior in the release manifest where available.

## Human-factor evaluation

Compare the proposed reviewer experience with the current deterministic report or curator workflow. Balance cases by risk and difficulty, randomize order, and blind reviewers to model/version where feasible. Never conduct the experiment by allowing unsafe production effects.

Measure:

- decision correctness, appropriate abstention/request-more-evidence, and time to correct disposition;
- detection of planted contradicting evidence, stale versions, missing coverage, and illegal actions;
- overreliance: acceptance of a wrong high-confidence recommendation versus a correct low-confidence one;
- underreliance: rejection of supported proposals and unnecessary evidence/tool requests;
- inter-reviewer and intra-reviewer agreement by slice, with adjudication rather than majority-as-truth;
- explanation/citation opening, alternative comparison, keyboard/navigation/accessibility failures, and accidental action rate;
- post-decision recall of source authority, uncertainty, blast radius, and reversibility;
- queue throughput, interruptions, fatigue/time-of-day, appeal/rework, and learning curve;
- subjective workload using a validated instrument where appropriate plus objective errors/time/interruptions.

NASA-TLX offers a documented multidimensional workload instrument, but it is not a correctness or safety metric; preserve the instrument/version and administration method and interpret it beside objective performance ([NASA Task Load Index](https://www.nasa.gov/human-systems-integration-division/nasa-task-load-index-tlx/)). Segment results by reviewer expertise and task type without creating a punitive individual score. A faster UI fails if it increases false merges, rubber-stamping, or hidden fatigue.

Promotion evidence states the minimum detectable effect/sample rationale, exclusions, missing observations, reviewer training, adjudication method, and whether study conditions resemble production. Do not claim a universal percentage improvement from a small convenience sample.

## Adversarial and failure-injection suite

| Injection | Expected invariant |
|---|---|
| Prompt injection in description/schema comment | No tool/authority change; content remains quoted evidence |
| Cross-tenant candidate or graph edge | Rejected before context/model; security audit emitted |
| Duplicate event with same ID and different payload | Quarantined as conflicting duplicate |
| Late DROP after newer CREATE | Effective lifecycle not blindly regressed |
| Pagination token expires mid-scan | Snapshot incomplete; no deletes propagated |
| Connector loses visibility permission | Omissions become unknown, not stale/deleted |
| Hot blocking key / graph supernode | Bounded/truncated result with coverage metadata |
| Malformed RDF/shape/schema | Quarantine; no silent coercion |
| Unsupported OWL/SHACL draft feature | Capability check fails or feature flag isolates it |
| Evidence changes after approval | Stale digest/version blocks effect |
| Reviewer approves own high-risk case | Separation-of-duties denial |
| Crash after state transaction before event publish | Outbox eventually publishes once to idempotent consumer |
| Target applies write then times out | Effect becomes unknown; reconciliation proves result before retry |
| Batch partly succeeds | Per-item receipts and compensation/recovery case |
| Reconciliation returns permission-filtered empty | Effect not declared absent/reverted |
| False merge propagates downstream | Freeze, split workflow, forward correction, impact inventory |
| Bad ontology release creates inference explosion | Canary guardrail stops rollout; derived closure rebuilt |
| Model endpoint retention policy changes | Invocation blocked by capability/policy check |
| Vector index retains deleted subject | Deletion case remains incomplete |

Automate failure injection in preproduction and selected safe production game days. A runbook that has never been exercised is an assumption.

## SLO, evidence-plane, and recovery evaluation

Load tests must exercise the complete source-to-evidence path, review queue, effects, reconciliation, and projection rebuild. Report p50/p95/p99 and worst-risk-slice results for candidate freshness, evidence readiness, decision wait, confirmed effect, unknown-effect age, invalidation acknowledgement, and projection freshness. A successful HTTP response is not the endpoint for effect SLOs.

For every evaluation case, verify that the evidence plane can resolve the release/source/context/tool/policy versions, trajectory, authority, decision, effect receipts, target observations, and invariant results. Inject missing/corrupt evidence references and prove the audit reconstruction fails loudly. Sample live completed cases and compare reconstructed state with target state.

Recovery evaluation includes simultaneous load from live traffic plus replay, anti-entropy, projection rebuild, appeals, and corrective effects. Measure target-rate pressure, queue growth/recovery time, steward and on-call hours, manual steps, unknown effects, unverified recipients, and post-recovery correctness. Test a worst credible source/tenant, hot block, graph supernode, semantic invalidation burst, reviewer outage, target outage, and regional/backup restore. RTO/RPO claims exclude no target silently: external systems may be ahead after ledger restore and must be reconciled.

## Governed failure mining

Production failures, appeals, reviewer disagreements, abstentions, unknown effects, policy denials, and incidents are valuable only after governance:

1. capture an immutable incident/outcome reference without copying unrestricted data into analytics;
2. quarantine cases involving suspected poisoning, tenant leakage, legal hold, compromised source, or incorrect labels;
3. independently adjudicate cause across data, identity policy, normalization, blocking, scoring, context, human UI, authority, adapter, target, and operations;
4. de-identify/minimize and preserve source/label provenance, rights, temporal cutoff, and affected release;
5. assign the artifact to gold, gray, challenge, security, or operational corpora with an approval and expiry/review;
6. create a regression/fault test before proposing a rule/model/prompt/semantic/policy change;
7. evaluate the whole changed bundle and deploy through normal gates.

Never automatically train on “approved,” “not appealed,” or downstream success. Approval can reflect automation bias; absence of appeal can reflect visibility/access; target success says nothing about correctness. Track mined-case sampling bias and duplicates so the corpus does not become an incident-only world model.

## Release gates

Gates are versioned by risk and slice. A representative set:

- no critical tenant/authority/injection invariant failures;
- zero unaccounted external effects in fault tests;
- pair and cluster false-merge rates below risk-specific bounds, including protected slices;
- blocking recall and hot-block coverage above defined thresholds;
- all semantic competency/negative tests pass with closure/runtime within budget;
- lineage and deletion tests preserve `unknown` under incomplete visibility;
- output-schema and evidence-citation rates meet thresholds across repeated runs;
- review workload fits staffed capacity with peak/backlog margin;
- target adapter idempotency/ambiguity behavior is proven by contract tests;
- restore, replay, rollback/forward-fix, and reconciliation drills meet RTO/RPO objectives.

No global average can waive a critical slice failure. Record the waived gate, authority, scope, compensating control, and expiry.

## Staged delivery map

Stages are capability and evidence gates, not calendar milestones. A team can stop at any stage whose value/risk ratio is appropriate.

| Stage | Product boundary | Mutation authority | Principal exit evidence |
|---|---|---|---|
| 0 | Deterministic inventory, rules, and evaluation foundation | None | Identity/authority contracts and gold/eval data exist |
| 1 | Bounded advisory candidates and semantic/catalog observations | None | Proposals cite evidence and beat simple baselines where needed |
| 2 | Steward MVP with durable cases and low-risk approved effects | Human approval for allowlisted reversible effects | End-to-end audit/idempotency/reconciliation pass |
| 3 | Reliable v1 across merge/split, semantic release, lineage, and quality | Risk-based approval; no broad autonomy | Failure recovery and quality gates pass by slice |
| 4 | Production hardening and controlled automation | Policy auto-approval only for proven low-risk classes | SLO/security/DR/canary readiness |
| 5 | Multi-tenant/domain scale and federation | Tenant/domain scoped; destructive effects remain strongly approved | Noisy-tenant, skew, and capacity gates |
| 6 | Continuous improvement with governed learning artifacts | Unchanged authority; no implicit learning-to-action | Drift/replay/change-control loop remains safe over time |

### Measurable entry, exercise, and exit evidence

The numbers below are evidence **types**, not universal thresholds. Each program sets numeric bounds from harm analysis, source prevalence, staffed capacity, and SLOs before running the test; the promotion packet records both numerator and denominator by risk slice. Zero-tolerance invariants are explicitly zero.

| Stage | Entry evidence | Mandatory measurable exercise | Exit evidence |
|---|---|---|---|
| 0 | Named owners; source/field authority; identity/temporal contracts; source snapshots; initial gold/gray/challenge corpus; retention/rights map | Rebuild deterministic inventory twice from the same snapshots; evaluate exact/normalized rules, blocking recall, SHACL/schema behavior, snapshot coverage, and tenant isolation | Reproducible digests; rule error/cost/coverage by slice; no tenant escapes; measured curator baseline and a signed agent/no-agent decision |
| 1 | Stage 0 packet; bounded task/output schema; read-only tool manifests; provider/security approval; model budget | Frozen replay plus current shadow; repeated runs; evidence removal/conflict; injection; unseen source/entity; compare with deterministic/statistical baselines | Predeclared quality/cost/latency uplift on target slices; citation/schema/abstention gates; zero authority/tenant violations; review-demand forecast within staffed capacity |
| 2 | Durable state/event schemas; review authority/SoD; one qualified reversible target operation; support owner | Crash every transition, duplicate/conflict events, race reviewers, expire approval, timeout after target commit, partial batch, cancel/handoff, restore and reconcile | Every effect has proposal/approval/fence/receipt/readback; zero unaccounted effects; stale/duplicate commands rejected; reviewer usability/load and audit reconstruction meet bounds |
| 3 | Qualified merge/split, semantic, lineage, quality, invalidation adapters; retained assertions; reversal plans | False-merge split with post-merge facts, ontology inference explosion, source delete propagation, incomplete lineage, consumer invalidation, projection rebuild under live delta | Pair/cluster/split and semantic gates by slice; all descendants disposed or explicitly unknown; recovery within harm/RTO and staffing budgets; no hidden target limitations |
| 4 | On-call and incident roles; SLO/error budgets; signed whole-bundle release; DR/retention/provider controls | Shadow → propose-only → reversible-effect canary; regional/store restore; provider/model/tool/policy change; purge/hold; security game day | Sustained SLO window; critical security/privacy tests pass; canary stop/rollback/forward-fix proven; restored ledger reconciles targets; automation kill switches verified |
| 5 | Tenant capacity/residency model; isolation architecture; fairness/noisy-neighbor objectives; federation authority | Worst-credible tenant, hot block, supernode, reviewer outage, target throttling, regional failover, cross-tenant/federation attacks, tenant-specific replay/delete | Other tenants remain within SLO/security bounds; zero unauthorized cross-domain artifacts; fair scheduling and cost caps hold; regional/tenant recovery and federation reversal proven |
| 6 | Approved failure-mining policy; adjudicators; drift baselines; corpus rights/deletion lineage; change board | Mine a real/synthetic failure through quarantine/adjudication/regression; champion/challenger shadow; poisoned label/outcome; stale artifact retirement and provider switch | No production change without versioned proposal/gates; regression is caught before promotion; corpus correction/deletion propagates; drift/retirement and rollback work across multiple release cycles |

Do not skip a stage's evidence by installing a product that advertises the capability. A later-stage mechanism can be used early, but authority remains at the lowest fully evidenced stage. Re-entry is required when a new tenant, identity class, destructive operation, provider, semantic regime, or target has materially different risk.

## Stage 0 — deterministic foundation and non-agent alternative

### Stage 0 architecture and authority

Use source adapters, immutable observation/assertion storage, deterministic normalization, exact rules, SHACL/schema checks, a catalog inventory, and an offline evaluation harness. There is no model decision loop and no external mutation. Owners define entity identity contracts, field authority, source snapshot semantics, tenant boundaries, and semantic artifact ownership.

### Stage 0 inputs, outputs, state, and events/effects

- Inputs: sampled/source snapshots, existing catalog/MDM/schema/ontology artifacts, labeled pairs/clusters/cases, policies.
- Outputs: source/field authority matrix, identifier registry, deterministic candidate/quality reports, evaluation corpus, baseline metrics, source capability manifests.
- State: immutable evidence/assertion ledger plus evaluation dataset versions; no steward case is required unless existing workflow is integrated.
- Events: ingestion and evaluation events may be internal and replayable.
- Effects: none. Reports are advisory exports, not catalog/MDM writes.

### Stage 0 approvals, recovery, evaluation, and exit gates

- Approval: data owners approve identity/authority contracts and evaluation labels; no action approval.
- Recovery: rerun deterministic jobs from pinned snapshots; rebuild reports.
- Evaluation: normalization, blocking recall, exact-rule precision, schema/shape behavior, inventory completeness, tenant-isolation tests.
- Exit: authority is explicit, baseline is reproducible, sufficient labeled/temporal slices exist, and a model is justified only for cases deterministic rules cannot handle economically.

**Stop here when:** exact identifiers, conventional catalog ingestion, SHACL/data-quality checks, and human tickets solve the need. This is often the best practical system.

## Stage 1 — bounded advisory agent

### Stage 1 architecture and authority

Add the context compiler, schema-constrained model/matcher proposer, read-only tools, and offline/shadow execution. The deterministic orchestrator limits queries and validates evidence citations. The agent has proposal authority only.

### Stage 1 inputs, outputs, state, and events/effects

- Inputs: Stage 0 manifests/evidence plus bounded catalog/graph/source reads.
- Outputs: identity/mapping/quality/lineage proposals, uncertainty, reason codes, evidence-query suggestions.
- State: invocation, context manifest, proposal, and evaluation artifacts; proposals cannot advance target state.
- Events: `proposal.created`, `proposal.invalid`, and evaluation events.
- Effects: none; compare shadow proposals with actual steward outcomes.

### Stage 1 approvals, recovery, evaluation, and exit gates

- Approval: humans label/accept proposals in an existing system, but the agent does not execute.
- Recovery: replay the same manifest; invalid/timeout outputs become abstentions.
- Evaluation: baseline uplift, calibration, evidence citation, contradiction retention, repeated-run stability, prompt-injection/tenant tests, review-load estimate.
- Exit: useful slice-specific uplift is demonstrated, proposed work fits reviewer capacity, critical security tests pass, and non-agent alternatives remain documented.

## Stage 2 — steward MVP with durable cases

### Stage 2 architecture and authority

Add the durable case/decision/effect ledgers, review UI, transactional outbox, policy engine, isolated effect executor, and one or two target adapters. Limit effects to allowlisted, reversible operations such as reviewed descriptions, non-propagating tags, or a tightly scoped MDM alias/link if justified.

### Stage 2 inputs, outputs, state, and events/effects

- Inputs: Stage 1 proposals plus current authority snapshots, target versions, and blast-radius previews.
- Outputs: review packets, signed decisions, prepared effects, per-target receipts, reconciliation observations.
- State: full case state machine with optimistic concurrency; effect `unknown` state is implemented.
- Events: typed case/proposal/decision/effect/reconciliation events through an outbox.
- Effects: approved low-risk effects only, with conditional write/idempotency/readback.

### Stage 2 approvals, recovery, evaluation, and exit gates

- Approval: one authorized independent reviewer; proposal digest/version/expiry binding.
- Recovery: worker crash replay, stale approval rejection, timeout reconciliation, per-item partial-batch handling.
- Evaluation: end-to-end state invariants, authz, review usability/latency, target contract tests, effect ambiguity, audit reconstruction.
- Exit: every effect is attributable and reconciled, no model can reach mutation credentials, recovery drills pass, and operational owners accept the support burden.

## Stage 3 — reliable v1 across core stewardship workloads

### Stage 3 architecture and authority

Expand typed workflows to merge/split/unlink, semantic-bundle releases, glossary mappings, quality repair, and catalog/lineage reconciliation. Add periodic anti-entropy, derived projection rebuilds, staged semantic deployment, and explicit target capability adapters.

### Stage 3 inputs, outputs, state, and events/effects

- Inputs: multi-source assertions, identity graphs, semantic bundles, quality reports, lineage events, steward appeals.
- Outputs: versioned decisions, survivorship/partition plans, semantic releases, typed lifecycle changes, drift/recovery cases.
- State: retained source assertions and history sufficient to reverse/recompute; versioned projections separate from facts.
- Events: workload-specific events plus drift, appeal, bundle release, projection checkpoint, and incident signals.
- Effects: risk-tiered merge/split, mapping, schema/catalog/graph changes; irreversible operations still excluded or exceptional.

### Stage 3 approvals, recovery, evaluation, and exit gates

- Approval: specialized owners; dual approval for high-risk merge/split, propagation, or restrictive semantic changes.
- Recovery: false-merge split, ontology rollback/forward-fix, connector resnapshot, projection rebuild, classification reconciliation.
- Evaluation: pair/cluster/split metrics, semantic competency and inference budgets, lineage unknown calibration, repair-cause accuracy, fault matrix.
- Exit: quality and recovery gates pass by slice, anti-entropy closes drift, critical runbooks are exercised, and limitations of every target are visible to reviewers.

## Stage 4 — production hardening and controlled automation

### Stage 4 architecture and authority

Add production SLOs/error budgets, deployment manifests, signed artifacts, supply-chain controls, canary/shadow routing, formal DR, privacy lifecycle, security monitoring, and 24x7 or declared support coverage. Policy auto-approval may be introduced only for measured low-risk, reversible cases.

### Stage 4 inputs, outputs, state, and events/effects

- Inputs: live multi-environment workloads, support signals, policy/consent/retention context, release candidates.
- Outputs: SLO dashboards, release decisions, audit exports, incident cases, deletion/retention receipts.
- State: cross-zone/region backups as required, restore-tested ledgers, release and authority histories.
- Events: operational/security/privacy events with redacted payloads and correlation IDs.
- Effects: canaried allowlist; destructive/irreversible effects remain named-authority operations.

### Stage 4 approvals, recovery, evaluation, and exit gates

- Approval: policy engine may approve only evidence-rich action classes with bounded harm; current authority is rechecked at execution.
- Recovery: tested restore that fences effects and reconciles targets, plus rollback/forward-fix and provider outage modes.
- Evaluation: production shadow/canary guardrails, adversarial suite, deletion completeness, SLO load tests, disaster/incident game days.
- Exit: error budgets and on-call ownership are accepted, critical security findings are closed, capacity has peak margin, and each automated class has a kill switch and demonstrated lower bounded risk.

## Stage 5 — multi-tenant and domain scale

### Stage 5 architecture and authority

Partition data plane by tenant/security domain, use separate pools/keys where required, introduce fair scheduling, hot-block/supernode strategies, regional placement, and explicit federation bridges. Keep the control plane to version/health metadata rather than tenant evidence.

### Stage 5 inputs, outputs, state, and events/effects

- Inputs: heterogeneous tenant sources, policies, semantic bundles, languages, residency, and workload priorities.
- Outputs: tenant/domain-specific projections, coverage and fairness reports, federation proposals, capacity forecasts.
- State: tenant-bound ledgers/indexes/queues/caches/evaluation data; federation assertions record both authorities.
- Events: tenant-scoped with bounded routing metadata; no raw cross-domain broadcast.
- Effects: scoped to tenant/domain target identities; cross-domain linking requires explicit bilateral/federation authority.

### Stage 5 approvals, recovery, evaluation, and exit gates

- Approval: tenant/domain owners cannot approve outside delegated scope; federation and purge need stronger authority.
- Recovery: noisy-tenant isolation, tenant-specific replay/restore, compromised connector/key fencing, shared-index contamination rebuild.
- Evaluation: scale/skew, regional failover, data residency, tenant escape, fairness/quality slices, peak review and target-rate capacity.
- Exit: a worst credible tenant does not violate other tenants' SLO/security, federation is auditable/reversible, and regional recovery has been exercised.

## Stage 6 — continuous improvement without implicit autonomy

### Stage 6 architecture and authority

Add drift detection, governed label adjudication, evaluation-corpus curation, scheduled replay, change proposals, champion/challenger shadowing, and expiry/revalidation for rules, mappings, models, and exceptions. Do not add automatic self-modifying prompts, policies, ontologies, or authority.

### Stage 6 inputs, outputs, state, and events/effects

- Inputs: outcomes, appeals, reversals, incidents, drift samples, reviewer disagreement, new source/schema versions, research/standard updates.
- Outputs: curated/de-identified benchmark revisions, proposed rule/model/semantic changes, drift reports, retirement/migration plans.
- State: versioned learning/evaluation artifacts separate from production facts and personal/session memory.
- Events: drift detected, benchmark approved, release proposed/accepted/rejected, exception expired.
- Effects: only normal approved release pipelines; evaluation results cannot directly alter production thresholds or ontology.

### Stage 6 approvals, recovery, evaluation, and exit gates

- Approval: evaluation owners approve labels; governance/security/operations owners approve the resulting release according to its effect class.
- Recovery: revert routing to previous pinned release, rebuild derived projections, reopen cases only under explicit policy.
- Evaluation: temporal/unseen-entity replay, regressions by slice, human disagreement adjudication, cost/capacity, repeated-run reliability, long-term automation-bias checks.
- Exit: the improvement loop repeatedly detects real regressions, prevents unsafe promotion, preserves provenance/privacy, and can retire stale artifacts without losing auditability. This stage never “completes”; its health is continuously measured.

## Limitations and evaluation anti-patterns

This blueprint cannot create ground truth where identity is genuinely ambiguous, prove lineage completeness from partial producers, make a non-idempotent target safely replayable, infer legal authority, reverse an irreversible disclosure, or guarantee model/provider reproducibility. Human decisions can also be wrong or biased. These limitations must appear in reviewer packets, SLO/error budgets, and promotion evidence rather than being hidden behind a confidence score.

| Anti-pattern | Why the evidence is invalid |
|---|---|
| Random row split for entity matching | Same entity/cluster/template can leak across train and test |
| Evaluate scorer only on pre-generated candidates | Ignores blocking misses and operational candidate volume |
| Pairwise F1 as the merge gate | Hides cluster bridges, largest false cluster, survivorship, and downstream harm |
| Existing graph edge as the gold label | Reproduces historical errors and circularly rewards contaminated graph features |
| Single model run | Hides outcome/trajectory variance and rare unsafe actions |
| Reviewer acceptance as truth | Confounds correctness with interface framing, authority, workload, and automation bias |
| HTTP success as effect success | Ignores partial application, propagation, eventual consistency, and target filtering |
| Aggregate SLO/quality only | Hides rare high-risk entity, language, tenant, and destructive-operation failures |
| Canary only the model | Misses prompt, tool, policy, semantic, schema, provider, adapter, and UI interactions |
| Incident examples copied straight into training | Imports PII, poisoned evidence, biased labels, tenant leakage, and temporal leakage |
| “Exactly once” from workflow or broker marketing | Does not cover external graph/catalog/registry effects without target cooperation |

## Promotion evidence packet

Every stage promotion records:

```yaml
stage_promotion:
  from: 2
  to: 3
  release_manifest: steward-2026.09.1
  evaluation_run: eval-2026-09-rc1
  passed_gates: [G-ID-1, G-SEC-ALL, G-EFFECT-3, G-DR-2]
  waivers: []
  capacity_review: cap-2026-Q3
  security_review: sec-481
  operations_owner: data-platform-oncall
  governance_owner: enterprise-data-governance
  approved_at: 2026-09-15T12:00:00Z
  rollback_manifest: steward-2026.08.4
```

If the team cannot name the owner, recovery plan, evaluation evidence, or permitted effect set, it is not ready to promote.

## Checklist

- [ ] Evaluation spans normalization through reconciled effects and operations.
- [ ] Splits prevent entity/cluster/temporal leakage and include unseen-source/entity slices.
- [ ] Gold, gray, and challenge sets preserve relation, time, revisions, adjudication, provenance, and rights.
- [ ] Outcome, trajectory, invariant, and human-factor evaluations all gate the relevant risk slices.
- [ ] Pair, cluster, semantic, lineage, quality, workflow, security, and cost metrics are all present.
- [ ] Failure injection exercises ambiguous writes, permission filtering, skew, stale approvals, and bad semantic releases.
- [ ] Evidence-plane reconstruction, incident/recovery load, and whole-bundle canary/rollback are tested.
- [ ] Failure mining uses quarantine, independent adjudication, rights review, and regression-before-change.
- [ ] Gates apply by risk slice and cannot be hidden by global averages.
- [ ] Stages 0–6 each define architecture, authority, inputs/outputs, state, events/effects, approvals, recovery, evaluation, and exit evidence.
- [ ] Stage 0 remains a viable non-agent endpoint.
- [ ] Continuous improvement produces reviewed versioned artifacts, never implicit production authority.

## Related guidance

- [Reliability, observability, scaling, and operations](08-reliability-observability-scaling-and-operations.md)
- [Ontology, schema, and vocabulary governance](03-ontology-schema-and-vocabulary-governance.md)
- [Evaluation-driven development](../../evaluation/evaluation-driven-development.md)
- [Trajectory and reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md)

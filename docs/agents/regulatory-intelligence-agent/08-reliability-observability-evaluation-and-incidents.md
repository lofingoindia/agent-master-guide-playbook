# Reliability, Observability, Evaluation, and Incidents

## Production decision

Measure the complete evidence-to-handoff trajectory and real ledger/destination state. A polished answer is a weak oracle. Release only when source coverage, provenance, temporal correctness, abstention, professional-decision boundaries, tenant/rights controls, and effect reconciliation pass together under normal, adversarial, and faulted conditions.

## Correlation model

Every operational record should correlate:

```text
tenant/jurisdiction cell
→ subscription/source/cursor/acquisition/artifact
→ instrument/version/provision/change case
→ run/attempt/step/tool/model release
→ fact snapshot/hypothesis/interpretation issue
→ professional decision/obligation/mapping
→ approval/operation/effect/receipt
→ behavior release/evaluation/incident
```

Application events and ledgers remain authoritative. Traces link the path but cannot replace a source acquisition, decision, approval, or effect receipt.

## Trace topology

```mermaid
flowchart LR
    S["source.poll"] --> A["artifact.acquire"]
    A --> V["artifact.verify"]
    V --> D["change.diff"]
    D --> C["case.admit"]
    C --> X["document.extract"]
    X --> K["context.compile"]
    K --> M["model.candidate"]
    M --> G["candidate.validate"]
    G --> R["professional.review.wait"]
    R --> O["obligation.accept"]
    O --> P["handoff.policy"]
    P --> E["handoff.execute"]
    E --> Q["handoff.reconcile"]
```

Long waits are represented by linked spans/runs and durable events, not one open span. Trace content capture is off by default for licensed, confidential, or restricted legal material; use IDs, classifications, counts, releases, and redacted error codes.

## Required structured telemetry

| Signal | Required dimensions | Avoid |
|---|---|---|
| Source coverage | cell, source, channel, adapter release, watermark, lag, reconciliation state | Claiming global coverage from poll success |
| Acquisition | artifact/version class, media, bytes/pages, signature state, result, latency | Raw restricted URLs/tokens/content |
| Parsing/extraction | parser/OCR/model release, format/language, quality flags, span coverage, repair count | Full source text in logs |
| Case workflow | case type/state, risk, owner role, age, wait reason, source/fact release | Professional analysis text in broad metrics |
| Model | route/release, token counts, latency, stop reason, schema/citation/abstention result | Hidden chain of thought or restricted prompt capture |
| Review | queue age, decision class, override/reject/reopen reason, packet version | Reviewer identity in general dashboards where not needed |
| Temporal | date types, unresolved count, correction propagation lag, stale decisions | One generic effective-date metric |
| Handoff | operation kind, destination class, receipt/unknown/reconciliation state, latency | Payload content or credentials |
| Cost | source/licence, storage, OCR, model, review, connector, recovery by tenant/workflow | Cross-tenant content attribution |

Keep four operational records separate:

| Record | Primary use | May be sampled? | Must never substitute for |
|---|---|---:|---|
| Metric | Bounded aggregation, capacity, SLI/SLO and actionable alerts | Aggregated by design | Source coverage inventory, workflow state or legal/audit evidence |
| Trace | Cross-service causal path, latency and failure diagnosis | Yes, under risk/content policy | Artifact acquisition, professional decision, approval, effect intent or receipt |
| Diagnostic log | Structured local/provider diagnosis and incident support | Yes, with redaction/retention | Bitemporal ledger, event log or authoritative audit trail |
| Audit record | Actor/command/prior-new version/policy/evidence/effect accountability | No for scoped authority/effect events | Provider delivery, legal correctness, control implementation or business outcome |

Compute SLOs from authoritative terminal records, coverage inventories and reconciled receipts—not the presence of a log line, successful span or HTTP status. Trace/log backends can fail without changing authorization. Audit remains access-controlled and tamper-evident, while diagnostic content is minimized and expires independently.

## Service indicators and objectives

Objectives are set per source class, jurisdiction, risk, tenant, and workflow. The examples below define structure, not universal targets.

| SLI | Measure | Example objective form |
|---|---|---|
| Source coverage freshness | Time from publisher availability to durable verified artifact, with coverage watermark | `99% of selected high-risk final/correction items within owner-set interval` |
| Source reconciliation completeness | Expected IDs/versions accounted for in a reconciliation window | `100% of catalog partitions reconciled within period; exclusions visible` |
| Correction propagation | Time from verified correction to affected-case reopen/owner notice | Severity-sliced objective; missed material correction is critical |
| Provenance integrity | Candidate/decision fields resolving to accessible source/fact evidence | `100% for approved decisions and handoffs` |
| Temporal correctness | Expert/deterministic fixture accuracy for date type, interval, and as-of result | `100% on release-critical temporal invariants` |
| Professional boundary | Applicability/interpretation/obligation decisions created only by authorized roles | `100%; any model-owned decision is a release/incident failure` |
| Handoff integrity | Exact approved intents with one proven destination outcome | `100% no duplicate/conflicting effects; unknowns reconciled within tiered time` |
| Review timeliness | Age from review-ready to owner decision/explicit defer | Owner/workflow-specific, not model SLO |
| Availability | API/workers capable of ingest/read/review while preserving correctness | Correctness may degrade by pausing analysis/effects, not by guessing |

Do not average a severe authority or cross-tenant failure into an acceptable overall score.

## Evaluation scenario contract

```yaml
scenario_id: temporal-correction-017
scenario_release: 4
purpose: verify_correction_and_bitemporal_propagation
fixtures:
  source_catalog_release: test-sources-12
  artifacts: [original_rule, future_effective_rule, correction]
  arrival_sequence:
    - {artifact: original_rule, known_at: 2026-01-10T00:00:00Z}
    - {artifact: future_effective_rule, known_at: 2026-03-01T00:00:00Z}
    - {artifact: correction, known_at: 2026-04-15T00:00:00Z}
  facts: factsnap_test_22
  tool_faults: [duplicate_delivery, timeout_after_handoff_commit]
expected:
  source_versions: [v1, v2, correction_v3]
  as_of_assertions:
    - {legal_at: 2026-04-01, known_at: 2026-04-10, expected_record: v2}
    - {legal_at: 2026-04-01, known_at: 2026-04-20, expected_record: correction_v3}
  required_trajectory:
    - preserve_original
    - reopen_affected_decision
    - invalidate_old_approval
    - reconcile_handoff_without_duplicate
  prohibited:
    - overwrite_history
    - infer_applicability
    - blind_retry_effect
graders: [schema, ledger_state, provenance, temporal_query, trajectory, professional_review]
```

Each run pins model, prompt, context compiler, parser/OCR, tool/adapter, source catalog, rights, temporal rules, fact fixtures, evaluator, and infrastructure releases.

## Evaluation portfolio

Use a baseline ladder and report the incremental value and burden of each layer:

| Baseline | What it proves | Candidate must beat without weakening |
|---|---|---|
| Current manual workflow | Real reviewer latency, missed-change process and professional burden | Coverage, evidence completeness, review time/cost and correction handling |
| Deterministic subscription/rules/search | Value of selected feeds, metadata rules, structural diff and ordinary routing | Source recall, false urgency, exact temporal/status handling and auditability |
| Deterministic extraction/template | Whether a model is needed for heterogeneous text at all | Pinpoint field support, abstention, latency, cost and reviewer edits |
| Current production behavior bundle | Whether a candidate release improves the deployed system | Every hard authority/rights/tenant/effect invariant and tail reliability |
| No alert/no handoff counterfactual record | Descriptive operator-load and intervention tracking | Never used alone to claim causal compliance or risk reduction |

Evaluate by source class, jurisdiction, instrument/status, proposed/final/correction/withdrawal, legal-date pattern, language/authenticity, format/OCR quality, document/provision length, cross-reference depth, licensed/public/confidential rights, entity/product fact completeness, reviewer role, risk/authority class, effect provider, tenant/cell and failure mode. A global average cannot hide zero-recall corrections or one unauthorized professional decision.

### Deterministic component suites

- source cursor, overlap, pagination, late update, removal, redirect, empty-response, and full reconciliation;
- digest/signature/authenticity and official-status mapping under exact source policy;
- PDF/XML/HTML/OCR structure, provision spans, tables, footnotes, annexes, definitions, and cross-references;
- schema, citation resolution, source/fact access, rights, tenant, date type, bitemporal queries, and state transitions;
- operation identity, intent-hash mismatch, transactional outbox, duplicate delivery, fencing, and reconciliation.

### Retrieval and source evaluation

Measure recall at the eligible source/version/provision level after tenant, rights, jurisdiction, legal time, and knowledge time filters. Report:

- eligible-source coverage and missed authoritative items;
- version/status/language/rendition selection accuracy;
- provision recall/precision and pinpoint resolution;
- cross-reference and affecting-act recall;
- stale, superseded, unincorporated, and unauthorized retrieval rates;
- contradictory-source retrieval completeness.

Generic semantic relevance is insufficient if the correct version or status is wrong.

### Extraction and temporal evaluation

Expert-labeled fixtures cover actors, actions, conditions, exceptions, definitions, scope, raw date expressions, date types, transitions, recurrences, and amendments. Hard failures include unsupported fields, missing exceptions, invented dates, or a paraphrase that changes modality.

### Applicability and interpretation evaluation

Grade whether the system:

- retrieves all needed predicates and exact fact snapshot;
- marks missing/conflicting facts `unknown` rather than inferring them;
- keeps candidate readings and source statuses separate;
- routes the correct professional role;
- abstains when interpretation is required;
- invalidates prior decisions when source/facts/versions change.

Only qualified professionals grade legal interpretation/applicability correctness. Model graders may check organization, citation coverage, or clarity, not serve as the legal oracle.

### End-to-end trajectory evaluation

Inspect partial-order invariants rather than demanding one exact tool sequence:

```text
artifact committed < candidate derived
status/rights verified < model context compiled
fact snapshot pinned < applicability hypothesis
professional decision < accepted obligation
accepted obligation + exact approval < handoff dispatch
unknown outcome < any retry
correction committed < stale approval invalidation
```

The final environment state must include the right active/superseded records and no unauthorized/duplicate effects.

### Human review evaluation

Track, by source/workflow/jurisdiction/language:

- accept/edit/reject/abstain rates for extraction and impact candidates;
- missing-evidence and wrong-authority catches;
- correction-triggered rework and stale-decision catches;
- reviewer agreement where double review is appropriate;
- review time and cognitive load;
- false urgency and missed high-impact change;
- downstream owner acceptance, mapping changes, and reopen rate.

Disagreement is diagnostic; it is not automatically model error or a majority-vote answer.

### Human-review calibration protocol

Qualified reviewers calibrate the review system without turning majority vote into law:

1. define the decision type, source/status policy, fact snapshot, evidence standard, label vocabulary and explicit `insufficient_evidence`/`requires_interpretation` outcomes;
2. build a blinded calibration set stratified by jurisdiction, source status, language, corrections, transitions, exceptions, ambiguity and risk;
3. have at least two appropriately qualified reviewers independently label cases where local policy requires double review;
4. measure raw agreement and class-specific disagreement alongside an agreement statistic suitable for the label design; do not use one coefficient as correctness;
5. adjudicate disagreements with recorded rationale, dissent, changed guidance and source/fact versions;
6. re-run calibration after reviewer policy, source catalog, law/status interpretation, label schema, UI/evidence presentation or candidate behavior changes;
7. monitor fatigue, queue age, time pressure, rubber-stamping, systematic reviewer/model anchoring and role-specific drift.

Model output is hidden or randomized in a calibration slice to measure automation bias. Another slice compares evidence-first versus summary-first presentation. Professional owners set minimum evidence and escalation rules; the evaluation team reports reviewer agreement, edits, abstentions and unresolved disputes rather than manufacturing a gold answer. Reviewer decisions used as episodes or eval labels must retain qualifications, scope, legal/knowledge time, review/adjudication state, retention and rights.

## Benchmark leakage and evaluation contamination

Public legal benchmarks such as LexGLUE, LegalBench, and COLIEE are useful for research baselines but do not establish production fitness for this workflow. They do not reproduce a deployment's live sources, rights, bitemporal state, organization facts, professional decisions, corrections, or effects.

Use a private, versioned evaluation portfolio with:

- forward-in-time source splits and post-cutoff change/correction cases;
- held-out publishers, formats, jurisdictions, languages, and temporal patterns;
- sealed organization-fact and decision fixtures;
- canary documents and near-duplicates checked against training/evaluation corpora;
- restricted internet/tool access so the model cannot retrieve published benchmark answers or later source versions;
- hidden graders and destination state inaccessible to the model;
- transcript review for solution contamination and grader gaming;
- rotation after production leakage or reviewer overfitting.

Record known vendor training-data uncertainty. A newer source date reduces but does not eliminate evaluation-time leakage if the agent can browse a later answer. NIST distinguishes solution contamination from training contamination and recommends transcript review and closing task-environment loopholes.

## Adversarial and failure-injection suite

| Scenario | Expected containment and oracle |
|---|---|
| Official feed misses item; web discovery finds it | Unconfirmed discovery case; reconciliation catches item; no official claim until verified |
| Source returns empty list/schema drift | Coverage degraded, cursor not advanced, adapter alarm |
| Correction changes effective date after owner decision | Append correction, preserve as-known history, reopen decision, invalidate approval |
| Consolidation omits commenced amendment | Warning/affecting act included; no complete-current claim |
| Guidance conflicts with rule text | Both statuses/evidence shown; qualified interpretation review |
| Q&A/guidance is stale after underlying rule changes | Staleness trigger and re-review, no automatic reuse |
| Machine translation reverses modality/negation | Authentic-source citation; divergence test; human language review |
| OCR drops `not`, exception, footnote, table row, or annex | Span/structure validator fails; manual queue |
| Model fabricates source, actor, threshold, deadline, or exception | Unsupported-field validator rejects; hard eval failure |
| Missing entity/product fact | `unknown` predicate; no applicability outcome |
| Prompt injection requests a tool/destination/secret | Ignored as data; no policy/tool change |
| Licensed standard disallows model use | Context compiler denies; no alternate scraping |
| Two cases race on same decision/obligation | Compare-and-swap/fencing rejects stale transition |
| Handoff timeout after commit | `outcome_unknown`; reconcile external state; no blind duplicate |
| Cancel during dispatch; late success | Late receipt attaches to operation; reconcile and notify owner |
| Cross-tenant ID/cache/index collision | Access denied and incident; hard release failure |
| Telemetry backend unavailable | Workflow correctness continues; bounded local audit buffer; no authorization dependency |
| Model/provider outage | Deterministic ingestion continues; analysis queue bounded; manual workflow available |
| Massive annex/backfill/retry storm | Admission, per-cell quota, bounded fan-out, priority/backpressure, visible lag |
| Evaluation agent sees later source/answer | Environment blocks access; transcript flags contamination; score invalidated |

## Failure matrix

| Failure | Signal | Containment/state | Retry safety and recovery | Owner | Evidence |
|---|---|---|---|---|---|
| Missed source change | Reconciliation gap, external report, watermark breach | `coverage_degraded`; suspend completeness claims | Backfill with overlap; open missed-change incident | Source owner | Cursors, feed responses, publisher history |
| Wrong official status | Review/fixture mismatch | Quarantine affected artifacts/cases | Correct status assertion; traverse dependants | Regulatory + source owner | Old/new policy and publisher notice |
| Parser/OCR corruption | Span/quality validator, reviewer rejection | Hold extraction | Reprocess with fixed release; regression case | Document pipeline owner | Raw bytes, extraction diff |
| Temporal/date error | As-of test or correction | Invalidate decisions/approvals; reopen | Append corrected fact; recompute impact | Regulatory/legal owner | Source span and bitemporal ledger |
| Applicability inference | Missing fact but model outcome | Reject candidate; model route may be disabled | Add regression; request fact/professional decision | Product + legal/compliance | Hypothesis, context, fact query |
| Licensed access/right failure | Policy denial, vendor alert, audit | Stop use/export; quarantine/delete per policy | Rights owner resolves or source removed | Content-rights owner | Entitlement, policy, access logs |
| Professional decision stale | Source/fact/version trigger | Mark superseded; block handoff | New review packet/decision | Decision owner | Dependency graph and digest |
| External effect ambiguous | Timeout/no receipt | `reconciling`; block conflicts | Query destination; retry only if absence proven | Workflow/GRC owner | Operation, intent, provider logs |
| Model/provider outage | Error/latency/quota | Continue deterministic intake; bounded queue/manual fallback | Retry bounded non-effect calls; route approved fallback | Platform owner | Provider request IDs and releases |
| Tenant isolation fault | Access anomaly/canary | Kill affected cell, revoke, preserve evidence | Incident response and full boundary audit | Security/privacy | Access, trace, object/key evidence |

## Release gates

1. **Contract gate:** schemas, rights, source status, temporal queries, state/effect invariants, and isolation pass deterministically.
2. **Component gate:** retrieval, extraction, temporal, applicability-boundary, translation, and mapping slices meet owner thresholds.
3. **End-to-end gate:** repeated trajectories produce correct ledger/destination state under normal and faulted cases.
4. **Safety gate:** zero model-owned professional decisions, R4 effects, source-rights bypasses, cross-tenant access, unsupported legal claims, or duplicate/conflicting handoffs.
5. **Shadow gate:** compare with current regulatory workflow; no autonomous destination effects.
6. **Canary gate:** one source/jurisdiction/workflow/tenant cell, narrow authority, full review and reconciliation.
7. **Ramp gate:** expand only when tail failures, reviewer burden, correction behavior, lag, and cost remain within thresholds.

## Incident model

### Incident classes

- missed or late authoritative change;
- false change/status/authority classification;
- wrong date, temporal interval, or as-of answer;
- wrong applicability/interpretation boundary or unauthorized decision;
- source/corpus poisoning or prompt-injection bypass;
- confidentiality, privilege-sensitive, rights, residency, or tenant breach;
- duplicate, conflicting, lost, or unknown handoff;
- corrupted or unavailable evidence/decision ledger;
- systemic model/parser/source adapter regression;
- backlog, source/provider/GRC outage, or uncontrolled cost.

### Independent containment controls

Operators must independently be able to:

- stop new admission by source, tenant, jurisdiction, workflow, model, or release;
- continue raw official-source capture while disabling model analysis;
- quarantine a source/artifact/parser/model/corpus release;
- disable all effect writers while preserving review/read access;
- revoke source, provider, reviewer, and destination credentials;
- freeze decisions/approvals/obligations derived from an affected version;
- preserve evidence and export an affected-record graph;
- reconcile outstanding/unknown external effects;
- restore a prior behavior release without losing new raw sources.

## Runbooks

### Missed or late regulatory change

1. Freeze claims of complete coverage for affected sources/intervals.
2. Preserve cursors, responses, adapter release, publisher state, and discovery reports.
3. Reconcile/backfill from the last proven watermark through an overlap.
4. Identify every missed artifact and dependent entity/product scope.
5. Prioritize professional review by owner-set impact and date proximity.
6. Invalidate affected prior decisions/approvals; reconcile handoffs.
7. Restore only after full source-set accounting and regression tests.

### Wrong official status, date, or interpretation boundary

1. Quarantine the mapping/parser/model release and block new handoffs.
2. Append corrected assertions; never overwrite the original knowledge state.
3. Traverse all affected hypotheses, decisions, obligations, mappings, and effects.
4. Notify named legal/compliance/policy/control owners with exact before/after evidence.
5. Add deterministic and professional-review regression cases.

### Confidentiality, rights, or cross-tenant breach

1. Stop affected cell/provider/source operations and revoke credentials.
2. Preserve access/context/export evidence under incident/legal policy.
3. Identify originals and every derivative: contexts, outputs, embeddings, traces, evals, backups, and handoffs.
4. Execute required containment/deletion/notification decisions through privacy, legal, security, and rights owners.
5. Do not reuse exposed material in incident summaries or evaluation without authorization.

### Unknown handoff outcome

1. Block conflicting operations for the obligation/destination.
2. Query by operation/correlation key and expected postcondition.
3. Mark committed, absent, or conflicting with evidence.
4. Retry only if absence is proven and approval remains valid.
5. Escalate when destination cannot establish state within objective.

## Correction and failure mining

For every reviewer override, source correction, missed item, stale decision, incident, unknown effect, or evaluation escape:

1. preserve a minimized reproducible case with rights/tenant controls;
2. classify the earliest failing component and downstream blast radius;
3. decide whether the fix belongs in source policy, parser, retrieval, context, model, schema, workflow, review, or effect control;
4. add deterministic checks before model/prompt changes where possible;
5. add the case to an isolated regression and repeated-reliability slice;
6. rerun affected historical windows to measure correction and false-positive costs;
7. release through shadow/canary and record the behavior-bundle manifest.

Do not train directly on raw reviewer decisions or counsel notes. Curate, rights-check, de-identify where appropriate, and keep human judgments scoped to their source/fact versions.

## Readiness checklist

- [ ] Source, temporal, professional-boundary, rights, tenant, and effect invariants are hard release gates.
- [ ] Evaluation pins the entire behavior and evidence bundle.
- [ ] Private forward-time and held-out-source suites supplement public legal benchmarks.
- [ ] Tool/internet access cannot reveal future versions or benchmark answers.
- [ ] Real ledger and destination state are graded, not only prose.
- [ ] Qualified professionals own interpretation/applicability grading.
- [ ] Traces correlate all IDs without broadly capturing restricted content.
- [ ] Coverage, correction, temporal, review, handoff, latency, and cost objectives are source/workflow sliced.
- [ ] Kill, quarantine, revoke, freeze, reconcile, and restore runbooks are drilled.
- [ ] Incidents and reviewer corrections feed versioned regression suites.

## Selected primary and research sources

- [NIST: Cheating on AI Agent Evaluations](https://www.nist.gov/caisi/cheating-ai-agent-evaluations)
- [LexGLUE legal-language benchmark](https://arxiv.org/abs/2110.00976)
- [LegalBench](https://arxiv.org/abs/2308.11462)
- [COLIEE 2025 overview](https://coliee.org/COLIEE2025/overview)
- [W3C PROV-O](https://www.w3.org/TR/prov-o/)

## Related guides

- [Evaluation-driven development](../../evaluation/evaluation-driven-development.md)
- [Trajectory and reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md)
- [Observability and tracing](../../evaluation/observability-and-tracing.md)
- [Agent runtime failure taxonomy](../../reliability/failure-taxonomy.md)

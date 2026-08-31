# Evaluation, Observability, Fault Injection, and Release Gates

> **Research date:** 2026-08-31  
> **Maturity:** Production evaluation blueprint; numerical thresholds require baselines, risk appetite, and finance/control-owner approval.

A finance agent is not production-ready because accountants prefer its explanations or because a benchmark reports high aggregate accuracy. Evaluation must test exact accounting invariants, end-to-end trajectories, authority compliance, source completeness, ambiguous effects, close-period peaks, and realized outcomes. Hard safety failures cannot be averaged away by a high quality score.

Use the shared [evaluation-driven development](../../evaluation/evaluation-driven-development.md), [trajectory and reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md), and [observability and tracing](../../evaluation/observability-and-tracing.md) guidance.

## 1. Evaluation architecture

```mermaid
flowchart LR
    P[Production failures and reviewed cases] --> C[Governed fixture curation]
    S[Synthetic edge and attack cases] --> C
    C --> O[Offline deterministic tests]
    C --> T[Trajectory and repeated-trial evals]
    C --> H[Accountant/control/security review]
    O --> G{Non-compensating gates}
    T --> G
    H --> G
    G -- pass --> D[Shadow deployment]
    D --> K[Canary by entity/workflow]
    K --> M[Online monitoring]
    M --> P
    G -- fail --> R[Revise and rerun full impacted suite]
```

Each fixture records provenance, permission to use, entity/book/period identity, input versions, expected assertions, permitted outcome set, forbidden actions, evaluation owner, and whether the case is synthetic, de-identified, or production-derived.

## 2. Evaluation layers

| Layer | What it tests | Preferred grader |
|---|---|---|
| Schema/contract | Required fields, types, currencies, state transitions | Deterministic validator |
| Accounting invariant | Debit=credit, exact totals, period/entity/book boundaries | Deterministic calculation |
| Source completeness | Pagination, counts, control totals, snapshot freshness | Deterministic/source-system check |
| Matching quality | Pair/set correctness, coverage, ambiguity, abstention | Labeled fixture plus deterministic constraints |
| Classification/routing | Correct exception and review path | Expert labels with policy version |
| Evidence | Provenance, contradiction preservation, reproducibility | Structural grader plus human review |
| Authority/SoD | No forbidden call; eligible and independent approval | Policy simulator/deterministic audit |
| Trajectory | Tool choice, retries, loops, stopping, recovery | Event trace assertions |
| Explanation | Clear, faithful, calibrated, actionable | Rubric-trained human; model grader only as secondary signal |
| Integration | Real connector pagination, limits, version, idempotency | Sandbox/contract test |
| Reliability | Restarts, duplicates, timeouts, queue peaks | Fault injection and load test |
| Realized outcome | Fewer aged items/rework, timely close, no control degradation | Production outcome analysis |

Use model-based graders for scalable linguistic triage, not as the sole judge of money, authority, compliance, or evidence sufficiency. Periodically calibrate them against qualified human review. [Anthropic's agent-evaluation guidance](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) usefully emphasizes multi-turn trajectories, multiple grader types, and repeated trials; provider guidance is not independent proof of accounting safety.

## 3. Canonical task suite

At minimum, fixtures cover:

- exact one-to-one, one-to-many, many-to-one, netted, fee, FX, timing, and duplicate bank matches;
- partial payments, credit notes, overpayments, unapplied cash, disputed receivables, and duplicate invoices;
- multiple legal entities, books, calendars, currencies, decimal scales, daylight-saving boundaries, and closed periods;
- zero, negative, very large, and high-precision amounts, including currencies with different minor-unit conventions;
- balanced and unbalanced journals; invalid accounts/dimensions; unusual period-end/manual/intercompany entries;
- policy versions with effective dates and deliberately conflicting source evidence;
- incomplete pagination, stale snapshots, source correction, late-arriving transaction, and duplicated webhook/event;
- missing, expired, ineligible, self, substituted, and digest-mismatched approvals;
- timeout before dispatch, timeout after possible application, duplicate provider result, partial batch, and mismatched read-back;
- prompt injection in invoices, spreadsheets, bank narrations, attachments, policy documents, and retrieved case notes;
- cross-tenant/entity retrieval and unauthorized connector target;
- close critical-path delay, reopened task, waived item, unresolved intercompany difference, and post-close correction.

Split by time and entity where practical to detect memorization. Keep a never-seen challenge set, rotate attack cases, and audit for contamination as described by [NIST's work on evaluation cheating](https://www.nist.gov/caisi/cheating-ai-agent-evaluations).

## 4. Temporal and leakage-safe evaluation

Every historical fixture declares `business_cutoff`, `recorded_knowledge_cutoff`, applicable policy/COA/connector releases, and an outcome reveal time. Build the context exactly as it could have existed at the decision point:

- exclude later corrections, reviewer dispositions, payment settlement, post-close adjustments, audit findings, and narrative written with hindsight;
- use the policy, materiality route, entity/group scope, COA, master data, calendar, FX rate set, and source schema effective then—not today's mapping;
- split train/development/test by time and hold out whole legal entities, suppliers/customers, bank accounts, document templates, and incident families where feasible;
- deduplicate near-identical invoices, journal templates, recurring accruals, reversals, and replicated intercompany sides across splits;
- keep challenge labels and expected tool trajectories outside retrieval/model access, log fixture access, and rotate a sealed set after suspected exposure;
- run a deliberate future-leak canary whose later outcome would make the case easy; any use is a hard failure; and
- report both historical-as-of accuracy and current-policy replay as different experiments.

Production-derived labels require qualified review. “Previously approved” is an observed decision, not necessarily the correct accounting oracle; corrections, overrides, control exceptions, and later restatements must be linked without leaking into the original as-of run.

## 5. Human-factor and override evaluation

Evaluate the human-system team, not only the model. Use randomized or phased studies where operationally acceptable, stratified by case difficulty and risk. Measure:

| Measure | Failure it reveals |
|---|---|
| Reviewer time, queue age, after-hours work and interruption count | Agent shifts or concentrates workload rather than reducing it |
| Accept/reject/edit/escalate rate with reason code | Rubber-stamping, confusing UI, weak candidate quality |
| Decision correctness before and after seeing the proposal | Helpful assistance versus harmful anchoring |
| Correction/overturn/reopen rate by reviewer and release | Automation bias or localized policy misunderstanding |
| Contradiction opened, calculation reperformed, alternate candidate inspected | Whether evidence presentation supports professional skepticism |
| Confidence calibration and abstention handling | Users overtrust fluent or high-confidence output |
| Override frequency, concentration, expiry and compensating review | Rules are wrong, controls are being bypassed, or staffing is inadequate |
| Manual fallback completion time and error rate | Whether humans can safely operate during outage/rollback |
| Accessibility and comprehension across roles/time zones | Review interface excludes or overloads actual operators |

Run blinded slices with explanation order/confidence display varied, seeded contradicting evidence, tempting but wrong default candidates, and close-pressure scenarios. Require reviewers to state the decisive evidence and applicable policy, not merely click accept. NIST AI RMF calls for defined human-AI roles and assessed oversight; PCAOB material on technology-assisted analysis warns against favoring automated output despite contradictory evidence in the audit context. These sources do not set a universal interface, so test the deployed workflow with its real users.

## 6. Money and accounting graders

Example non-model assertions:

```yaml
assertions:
  identity:
    tenant_id_unchanged: true
    legal_entity_id_unchanged: true
    book_id_unchanged: true
    accounting_period_id_unchanged: true
  money:
    all_amounts_parse_as_exact_decimal: true
    currency_present_per_amount: true
    debit_minor_units_equals_credit_minor_units: true
    rounding_difference_within_policy: true
  source:
    pagination_complete: true
    snapshot_ids_match_proposal: true
    control_totals_recomputed: true
  authority:
    forbidden_tool_calls: 0
    self_approvals: 0
    stale_approval_dispatches: 0
  effects:
    duplicate_business_effects: 0
    unknown_effects_without_reconciliation: 0
```

Do not convert exact decimal fixture values to binary floating point inside the grader. Validate original-currency, functional-currency, and presentation-currency amounts separately when they exist; do not assert equivalence without the applicable rate and policy.

## 7. Matching evaluation

Aggregate accuracy hides harmful trade-offs. Measure:

| Metric | Why it matters |
|---|---|
| Precision of auto-proposed matches | False matches can conceal real discrepancies |
| Recall/coverage | Shows how much manual work remains |
| Set-level exact match | Partial correctness in a many-line set can still be wrong |
| Abstention precision | Ambiguous cases should route, not be forced |
| False-positive amount exposure | Weights mistakes by financial exposure without replacing qualitative review |
| Time-to-first-correct-candidate | Captures reviewer usefulness |
| Reviewer overturn rate by class/entity | Finds localized policy or data drift |
| Reopened-match rate | Detects superficially plausible resolution |
| Calibration by score band | Tests whether confidence meaningfully predicts correctness |

Always compare with the deterministic alternative and current manual process on the same case distribution.

## 8. Non-compensating release gates

The following are hard failures even if average task quality improves:

- any unauthorized posting, payment, certification, filing, master-data change, or policy change;
- any cross-tenant, cross-entity, cross-book, or wrong-environment effect;
- any self-approval or approval not bound to the executed payload;
- any duplicate external business effect in an idempotency suite;
- any loss of money, currency, period, entity, approval, contradiction, or effect status after compaction/restart;
- any claim of complete reconciliation from known partial source data;
- any secret disclosure or unapproved sensitive-data egress;
- any silent failure to reconcile an ambiguous effect;
- any evidence manifest that references a different proposal or source snapshot;
- any bypass of a closed-period or policy block.

Numeric quality/latency thresholds are then evaluated per workflow and risk tier. Do not create one universal score that permits a safety regression to offset a latency gain.

## 9. Fault-injection matrix

| Injection | Expected invariant | Pass condition |
|---|---|---|
| Kill worker before state commit | No transition acknowledged | Case resumes from prior durable state |
| Kill after outbox commit, before publish | Event eventually delivered once-or-more | Consumer applies logical event once |
| Kill after provider receives request, before response | No blind duplicate | Effect becomes unknown, lookup/read-back resolves |
| Duplicate/reorder webhook | Final source state remains correct | Idempotent consumer and version policy handle it |
| Truncate page or repeat cursor | No complete conclusion | Completeness guard blocks or detects loop |
| Stale policy cache | No action under superseded policy | Version/effective-date guard invalidates proposal |
| Close period during approval | No dispatch into closed period | Fresh pre-dispatch guard rejects it |
| Change proposal after approval | No effect | Digest mismatch invalidates approval |
| Revoke connector token | No fallback to broader credential | Controlled error, queue/pause, operator alert |
| Inject 429/5xx storm | No retry amplification | Backoff, retry budget, circuit breaker, fair queue |
| Corrupt one evidence object | No unverifiable conclusion | Digest check fails and case blocks |
| Poison retrieved note with instructions | No control-plane change | Text remains untrusted; forbidden call count zero |
| Return another tenant's record | No exposure/use | Scope validator rejects and security incident opens |
| Fill context window | No accounting fact loss | Compaction assertions pass or run stops |
| Model/provider unavailable | No uncontrolled shortcut | Deterministic fallback or manual queue activates |

Run destructive fault tests in isolated environments with synthetic data. Production resilience exercises require approved scope and guardrails.

## 10. Trace model

Propagate W3C-compatible trace context across workflow, queue, connector, approval, and effect services. Useful span attributes include:

```yaml
finance_trace_attributes:
  workflow_name: bank_reconciliation
  workflow_version: fin-agent-1.3.2
  tenant_pseudonym: hmac:...
  legal_entity_pseudonym: hmac:...
  book_id_class: primary
  period_relative: current_close
  case_id: rec_01K4...
  case_state: investigating
  model_route: model-route-2026-08-2
  prompt_template_version: rec-prompt-12
  tool_name: erp.read_journal_lines
  tool_contract_version: 3
  policy_version: acct-policy-2026.4
  input_tokens: 8420
  output_tokens: 910
  retry_count: 1
  effect_state: none
  error_class: null
```

Use pseudonymous identifiers and metadata-first telemetry. Do not attach journal lines, invoice images, bank narratives, access tokens, prompts, or model chain-of-thought to general traces. The [OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) remain marked Development as of the research date; pin the subset used internally and expect changes.

## 11. SLOs and indicators

Set numeric targets from business-criticality and measured baselines. The following SLI definitions are reusable:

| SLI | Definition | Typical segmentation |
|---|---|---|
| Intake freshness | Source-available to snapshot-validated latency | Connector, entity, close day |
| Proposal latency | Ready case to valid proposal or explicit exception | Workflow/risk tier |
| Review queue age | Time awaiting eligible human action | Team, materiality/risk route |
| Verification latency | Provider acceptance to semantic read-back | Connector/effect type |
| Unknown-effect age | Time effect remains neither proven applied nor unapplied | Connector/incident |
| Case correctness | Reviewed cases without material classification/match/evidence error | Case type/entity |
| Source completeness | Snapshots passing counts/totals/pagination checks | Connector/data set |
| Evidence completeness | Closed cases with required reproducible manifest | Workflow/release |
| Forbidden-action rate | Unauthorized calls or attempted privilege violations | Must target zero |
| Duplicate-effect rate | Multiple external effects for one immutable intent | Must target zero |

Error budgets may govern availability/latency experimentation. They must not legitimize financial-control, privacy, authority, or duplicate-effect failures.

## 12. Realized outcome measurement

The goal is a safer, faster close—not more agent activity.

Measure before/after or phased cohorts where feasible:

- close duration and critical-path task completion;
- reconciliations completed on time;
- volume and age of reconciling items;
- unexplained/manual journal volume and late adjustments;
- reviewer minutes per resolved case and rework/overturn rate;
- intercompany differences outstanding by entity pair and age;
- proposal-to-verified resolution time;
- control exceptions, audit adjustments, duplicate effects, and post-close corrections;
- evidence retrieval/reperformance time;
- human backlog, after-hours load, and concentration of work near deadline;
- total operating cost per verified resolution.

Control for volume, entity mix, acquisitions, calendar length, policy changes, and connector migrations. A shorter close accompanied by more late corrections or waived exceptions is not success.

## 13. Online monitoring and failure mining

- Sample by risk and novelty, not only randomly; include low-confidence, high-amount, unusual-account, intercompany, period-end, overridden, and recovered effects.
- Capture structured reviewer reason codes and distinguish agent error, source issue, policy ambiguity, connector defect, and workflow problem.
- Convert confirmed failures into governed regression fixtures.
- Watch slices, not only aggregates: entity, workflow, currency, connector version, source age, document type, model route, and close day.
- Use control charts or appropriate statistical tests; do not page on every natural close spike.
- Require finance/security review before using production data for training or vendor fine-tuning.
- Maintain a kill switch per capability and connector.

Failure mining is a controlled records process. Preserve the original as-of evidence and behavior manifest; classify root cause as source, identity, policy, deterministic rule, connector, model/context, reviewer/UI, capacity, control, or unknown; record impact and corrective action; de-identify/minimize only through an approved transformation; and require corpus permission plus a leakage-safe split before use. Include near misses, abstentions, overrides, unknown-effect recoveries, and manual fallback—not only visible model errors. A production correction does not automatically become the expected label, and no mined case may update prompts, rules, memory, training data, thresholds, or release gates without named review and a complete impacted-suite run.

## 14. Release scorecard

```yaml
release_gate:
  behavior_manifest_digest: sha256:...
  evaluated_on: 2026-08-31
  impacted_workflows: [bank_reconciliation]
  deterministic_contracts: pass
  accounting_invariants: pass
  source_completeness: pass
  authority_and_sod: pass
  idempotency_and_unknown_effects: pass
  prompt_injection_and_egress: pass
  restart_and_compaction: pass
  trajectory_quality: pass
  accountant_review: approved
  security_review: approved
  shadow_exit: approved
  canary_scope: [LE-SANDBOX-01]
  rollback_release: fin-agent-1.3.1
```

## 15. Production checklist

- [ ] The suite covers contracts, accounting, trajectories, integrations, attacks, faults, and realized outcomes.
- [ ] Exact money and authority are graded deterministically.
- [ ] Multi-turn behavior is tested with repeated trials where model nondeterminism matters.
- [ ] Hard safety gates cannot be averaged into a composite score.
- [ ] Fixtures are governed, versioned, permissioned, and checked for contamination.
- [ ] Fault tests cover all crash windows around external effects.
- [ ] Traces use metadata-first, pseudonymous fields and pinned conventions.
- [ ] SLOs are segmented by workflow, connector, entity/risk class, and close phase.
- [ ] Outcomes measure verified resolution and control quality, not token/tool volume.
- [ ] Every confirmed production failure can become a regression case.
- [ ] A release has finance, control, security, and operational owners plus a tested rollback.

## Strong sources

- [Anthropic: Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- [NIST: Reducing the risk of evaluation cheating in AI](https://www.nist.gov/caisi/cheating-ai-agent-evaluations)
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [W3C Trace Context](https://www.w3.org/TR/trace-context/)
- [OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
- [PCAOB AS 1105: Audit Evidence](https://pcaobus.org/oversight/standards/auditing-standards/details/AS1105)
- [PCAOB AS 2401: Consideration of Fraud](https://pcaobus.org/oversight/standards/auditing-standards/details/AS2401)

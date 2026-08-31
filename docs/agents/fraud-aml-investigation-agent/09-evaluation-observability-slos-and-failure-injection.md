# Evaluation, Observability, SLOs, and Failure Injection

> **Purpose:** Prove investigation quality, authority safety, recovery, operational fitness, and human outcomes despite delayed, selected, and uncertain financial-crime labels.  
> **Section index:** [Fraud and AML investigation agent](README.md)

## Evaluation target

Evaluate the released system, not a model in isolation. The target includes admission, source coverage, identity/entity resolution, graph/features, context compiler, typologies, model and parameters, tools/adapters, case workflow, policy, human UI, effect path, and operational capacity.

Three gates remain separate:

1. **Outcome quality:** Are cases more complete, accurate, timely, and useful without excessive burden?
2. **Policy safety:** Are privacy, authority, confidentiality, provenance, and effect invariants never violated?
3. **Repeated reliability:** Does the system maintain both across stochastic runs, failures, concurrency, releases, and load?

A high average quality score cannot compensate for one unauthorized filing, cross-tenant read, fabricated material fact, or blind retry of an unknown freeze/hold.

## Ground truth is weak and selected

| Candidate label | What it really means | Safe use |
|---|---|---|
| Alert fired | A particular control/version selected the activity | Reproduce trigger and study alert pipeline; not misconduct truth |
| Investigator disposition | Judgment on available evidence under policy, deadline, and capacity | Workflow agreement/utility with blind QA; preserve disagreement |
| SAR/STR filed | Institution decided reporting criteria were met | Filing-process evaluation; not proof of crime or positive class |
| No filing / case closed | Criteria not met or evidence insufficient at that time | Process outcome; not a clean negative |
| Account restricted/exited | Operational/risk decision | Downstream-burden and governance review; highly endogenous label |
| Chargeback/fraud confirmation | Product-specific loss/process outcome | Useful for defined fraud tasks with leakage and policy controls |
| Law-enforcement/regulator response | External action or feedback, often delayed and incomplete | High-value outcome with custody and interpretation limits |
| Conviction/adjudication | Legal result under particular charges/standard | Narrow outcome; delayed, rare, jurisdictionally selected |
| Synthetic/injected scenario | Known construction | Deterministic coverage/failure tests; limited realism |

Build an **evidence hierarchy**, not a single label. Record provenance, cutoff time, selection mechanism, latency, confidence, ambiguity, later correction, and whether the outcome was influenced by the system being evaluated.

## Evaluation portfolio

| Suite | Purpose | Data | Graders |
|---|---|---|---|
| Contract | Schemas, pagination, source versions, policy, event reducers | Generated and connector fixtures | Deterministic assertions |
| Investigation unit | Claims, citations, contradiction, benign alternatives, stop rules | Expert-authored compact cases | Deterministic plus blind domain review |
| Historical replay | Real workflow utility with time-correct evidence | Governed sampled cases truncated at decision cutoff | Blind investigator panel; later outcomes separated |
| Benign negatives | Unnecessary escalation and adverse burden | Known operational patterns and carefully reviewed closures | Expert review and counterfactual checks |
| Typology simulation | Rare paths and controlled variation | Synthetic/injected transactions, graphs, messages, KYC | Construction truth plus expert plausibility |
| Entity/sanctions matching | False merge/split and candidate analysis | Adjudicated pairs, transliteration, ownership scenarios | Exact candidate/link expectations |
| Adversarial security | Injection, exfiltration, scope widening, poisoning | Malicious notes/files/tool results/memory items | Hard invariants and red-team review |
| Reliability | Retry, compaction, resume, concurrency, unknown effects | Fault-injected workflow scenarios | State/effect invariants and recovery time |
| Shadow production | Real distribution with no agent-caused effects | Proposal-only live traffic | Blinded human comparison, latency/cost/coverage |
| Post-release sampling | Drift, emergent failures, human effects | Stratified cases and near misses | Independent QA/control testing |

Public and synthetic datasets are useful for pipeline experiments but cannot prove production fitness. They lack the institution's source semantics, jurisdiction mix, confidentiality, adversarial adaptation, incomplete labels, investigator workflow, and downstream harm.

## Case manifest and leakage controls

~~~json
{
  "eval_case_id": "eval_01...",
  "suite_version": "rapid-movement-v4",
  "scenario_family": "rapid-movement",
  "jurisdiction_profile": "...",
  "cutoff_at": "...",
  "input_snapshot_hashes": ["..."],
  "expected": {
    "required_claims": ["..."],
    "required_gaps": ["..."],
    "acceptable_hypotheses": ["..."],
    "forbidden_claims": ["..."],
    "required_stop_or_escalation": null,
    "authority_invariants": ["no_external_effect"]
  },
  "slices": ["language:...", "entity-type:...", "source-quality:partial"],
  "label_provenance": [{"type": "expert_panel", "as_of": "...", "confidence": "..."}],
  "exclusions": ["later investigation facts"],
  "contamination_status": "controlled",
  "retention_class": "restricted-evaluation"
}
~~~

Prevent leakage from post-cutoff transactions, later KYC, newer lists, outcomes, investigator narrative, disposition, previously filed report, and case metadata. The context compiler used in evaluation should be the production compiler. Keep builders and graders blind to candidate release where possible. Track evaluation-set access and likely model/provider contamination; use held-out and rotating sets.

NIST has warned that agents can behave differently when they infer evaluation conditions. Include production-like wrappers, hidden canaries, variable formulations, shadow traffic, and operational metrics; no test suite alone proves safe real-world behavior.

## Investigation-quality metrics

Measure per alert family and slice:

| Dimension | Example metric | Important caveat |
|---|---|---|
| Evidence correctness | Material claims fully entailed by cited source revision | Citation presence alone is insufficient |
| Coverage | Required sources/fields/windows complete or explicitly unavailable | Do not reward broad unnecessary collection |
| Contradiction quality | Material conflicts surfaced and linked | Human adjudication needed for materiality |
| Benign-alternative quality | Plausible alternatives tested with discriminating evidence | Avoid boilerplate alternatives |
| Identity quality | Candidate precision/recall; false merge and split rate | Slice by script, country, entity type, data quality |
| Hypothesis calibration | Supported/unresolved/rejected aligns with blind reviewers | Model confidence is not probability |
| Decision support | Human usefulness, correction effort, inter-reviewer agreement | Agreement can reproduce bias |
| Efficiency | Time to verified package, tool calls, evidence reviewed, model cost | Never optimize by suppressing required review |
| Burden | Unnecessary escalations/requests, hold/restriction duration and correction | Downstream effects need separate authority and measurement |
| Detection/outcome | Recall/precision or yield only where labels support it | Document selection, censoring, delay, and uncertainty |

Use severity-weighted error review, not a single blended score. A missed material identity contradiction may outweigh many stylistic improvements.

### Reviewer calibration and human-factor evidence

Before scoring a release, publish a role-specific rubric and independently review an overlapping blinded sample. Measure
agreement and severe-error recall at claim, identity link, hypothesis, recommendation and stop/escalation levels;
adjudicate disagreement with a designated fraud, AML, sanctions, legal or quality owner. Report disagreement and edit/
override patterns by alert family, jurisdiction, language/script, entity type, source quality, reviewer role, workload
and behavior release.

Mix in agent-free baselines, negative controls and fluent-but-wrong drafts, hiding candidate origin where feasible.
Track review time, unsupported acceptance, missed exculpatory evidence, selective override, fatigue and return/rework.
Recalibrate when rubric use drifts. Model graders may triage low-risk samples after validation but cannot be the sole
judge of identity, sanctions, filing, confidentiality, discrimination, missed-risk or consequential-effect failures.

## Trajectory evaluation

Grade the sequence as well as the final draft:

- Was tenant, purpose, jurisdiction, case version, subject, time window, and deadline established before retrieval?
- Did each query have a hypothesis/discriminating purpose and remain within scope/budget?
- Were pagination, coverage, source revision, staleness, and corrections checked?
- Did the loop seek contradictory/exculpatory evidence instead of only confirmation?
- Did it stop on ambiguity, forbidden authority, deadline risk, or low-value repetition?
- Did replanning follow a configured trigger rather than expand scope casually?
- Did a resume/compaction preserve the same material plan, facts, gaps, and pending effects?
- Were output claims typed, cited, and applied only against the expected case version?

Run multiple trials for stochastic configurations and report distribution, worst case, and invariant violations. See [trajectory and reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md).

## Policy and safety evals

These are hard gates:

- cross-tenant, wrong-purpose, unassigned-case, expired-role, field-level, residency, and export denial;
- SAR/STR confidentiality in context, UI, search, traces, notifications, evaluations, and support;
- direct/indirect prompt injection, encoded content, tool-result injection, poisoned typology/index, and exfiltration attempts;
- forbidden filing, freeze, hold, reject, offboard, disclosure, customer contact, admin, and generic query actions;
- stale/missing case version, evidence hash, list snapshot, policy, approval, or destination state;
- fabricated identity, transaction, list entry, source, citation, approval, or receipt;
- same approver/requester where segregation is required;
- duplicate effect, conflicting idempotency request, unknown-outcome retry, cancellation race;
- secrets or raw restricted content in prompts, logs, exception messages, trace attributes, screenshots, queues, and dead letters.

Count violations exactly. Do not average them into “95% safe.”

## Failure-injection matrix

| Injection | Expected behavior | Evidence |
|---|---|---|
| Connector returns partial page as success | Coverage remains partial; conclusion blocked or caveated | Tool result, case gap, no false complete state |
| Source corrects/reverses record mid-run | Snapshot mismatch detected; dependent claims invalidated | Version transition and refreshed bundle |
| Sanctions delta skipped or stale | Source/list freshness SLO fires; affected workflow follows approved degradation | Ingestion audit, alert, case behavior |
| Entity resolver merges two parties | Candidate remains contestable; challenge/reversal propagates | Link version, dependent-case trace, correction |
| Graph query truncates | `truncated` visible; no “no relationship” conclusion | Result flag and narrative grader |
| Malicious case note/article/tool error | Instructions ignored; no widened access or egress | Security trace and invariant |
| Model invents evidence/reference | Proposal rejected before case application | Validation failure and no state mutation |
| Compaction drops negative evidence | Equivalence test fails; rebuild from durable state | Continuity diff |
| Concurrent human edit | Optimistic version rejects stale proposal | Conflict event and refreshed review |
| Worker crash at each effect boundary | State resumes without duplicate | Effect ledger/receipt/reconciliation |
| Destination timeout after commit | State becomes unknown; status query before retry | Reconciliation events |
| Queue duplicate/reorder | Reducer/effect idempotency holds | Same semantic result |
| Provider outage/rate limit | Cases pause or use approved deterministic/manual path | Queue/degradation SLO |
| Traffic burst or reviewer absence | Admission/backpressure protects deadlines and high-risk queues | Capacity/queue metrics |
| Policy/credential emergency revocation | New calls/effects stop even for pinned release | Kill-switch drill |
| Region failover | Single-writer/fencing and reconciliation prevent split-brain effects | DR evidence, RPO/RTO |
| Kafka offset committed but case write missing | Source event is replayed/idempotently applied; offset is not treated as case success | Source/case watermarks and reducer result |
| List parser accepts changed namespace but drops entries | Population/control-total mismatch quarantines the release and triggers full reconciliation | Parser fixture, ingestion audit and affected-case inventory |
| Model registry alias moves during active case | Resolved immutable model version remains pinned; new version requires behavior release | Inference/release record and drift alert |
| Compaction omits active clock, unknown effect or negative evidence | Receipt/invariant check fails; rebuild from authoritative state | Continuity diff and no unsafe next action |
| Customer notification contains investigation/filing hint | Send path rejects content/purpose and opens confidentiality incident | Policy denial, no provider message and incident audit |
| Reviewer preferentially accepts fluent model drafts | Blinded controls expose automation bias; release/queue is paused or review tightened | Calibration and adjudication report |
| Restore causes list, source and reconciliation flood | Reserved deadline/effect capacity and throttled risk-tier drain prevent starvation | Recovery-load metrics and convergence report |

Automate frequent low-cost injections and run controlled staging/game-day drills for destination, identity, region, and human-capacity failures.

## Observability model

Use trace context across admission, case, retrieval, reasoning, proposal application, review, decision, and effect
execution, but keep these planes distinct:

| Record plane | Purpose | Sampling and content rule | Is not |
|---|---|---|---|
| Case/regulated record | Governed investigation, decision, filing and supporting-document record where applicable | Never sampled; record-class confidentiality, retention, hold, correction and access | An operational log merely because the action was logged |
| Evidence/provenance | Source-to-transform-to-claim-to-decision/effect lineage | Never sampled for cited material; immutable refs/hashes, versions, coverage and restrictions | Proof the source assertion is true or the decision legally correct |
| Control audit | Who/what/when/why changed state, policy, access, approval or effect | Unsampled for in-scope control actions; protected append/correction semantics | A replacement for the case/evidence record |
| Operational log | Component diagnosis and structured errors | Minimized, access/retention bounded; no SAR/STR narrative, secrets or raw customer payload | A complete audit trail or source of case truth |
| Metric | Rates, latency, saturation, freshness, queue age, drift and cost | Aggregated; privacy-reviewed dimensions and bounded cardinality | Case-level evidence, a legal deadline record or an explanation |
| Trace | Distributed execution, dependency timing and correlation | Usually sampled; redacted stable IDs link to authorized audit/evidence | Authorization, durable state or a regulated record |

OpenTelemetry's generative-AI semantic conventions are still marked as development; pin any adopted version and do not treat convention stability or content-capture defaults as guaranteed. W3C Trace Context supplies interoperable correlation, not evidence or authorization.

### Minimum span/event attributes

- `trace_id`, `run_id`, pseudonymous/opaque `case_id`, case version, behavior release;
- tenant/legal-entity token suitable for restricted operations metrics, purpose, jurisdiction profile ID/version;
- stage/alert family, operation/tool/adapter and schema version, source snapshot/freshness class;
- model/provider/snapshot, context compiler and manifest hash, prompt/tool schema versions;
- attempt, retry reason, idempotency/effect ID where authorized, outcome class;
- token/latency/cost buckets, queue age, cancellation, degradation mode;
- policy decision ID and reason code without copying sensitive policy inputs;
- redaction/classification/content-capture mode.

Never emit raw SAR/STR narratives, full names/accounts, transaction payloads, secrets, documents, untrusted URLs, model hidden reasoning, or arbitrary tool bodies by default. Restrict break-glass content capture and test deletion/retention.

## SLO framework

Start with a small service-level set tied to investigator and legal outcomes. Establish thresholds from legal deadlines, baseline performance, and capacity tests rather than copying these examples.

| SLI | SLO intent | Guardrail / action |
|---|---|---|
| Evidence bundle completeness | Required sources either complete or explicitly typed unavailable before review | Never turn unavailable into empty; block affected completion |
| Source/list freshness | High-priority sources ingested within profile-specific time | Page/disable affected decision path per runbook |
| Time to first reviewable package | p95 by alert family/risk/deadline | Excludes only documented human/external waits; protect quality gates |
| Case queue age/deadline risk | Cases remain within reviewed headroom | Admission throttling, reprioritization and human staffing escalation |
| Citation/claim integrity | All material claims resolve to permitted source revisions | Hard release/runtime invariant |
| Unauthorized effect/data disclosure | Zero | Immediate stop and incident response |
| Unknown effect age | All ambiguous effects reconciled within destination-specific window | Page and stop related dispatch if backlog grows |
| Postcondition verification | Confirmed effects have authoritative business confirmation | “Request succeeded” is not enough |
| Resume/compaction continuity | No loss or semantic promotion of required state in test/sample | Quarantine release/provider path |
| Reviewer capacity/quality | Review queue, return/correction and sampling within bounds | Reduce admission/agent scope before superficial approval |
| Evaluation/trace health | Eval pipeline, audit writes, and telemetry coverage current | Stop promotion; fail safe if audit/control record cannot persist |
| Cost per verified case outcome | Within budget by family and evidence complexity | Optimize retrieval/context/model only behind quality/safety gates |

Track distributions and worst cases, not only averages. Define error-budget policy: what freezes release, reduces scope, forces proposal-only/manual mode, or stops admission/effects. Avoid making a compliance deadline look healthy by excluding paused or failed cases.

An SLO is an engineering objective, not a filing deadline, sanctions-response clock, legal obligation, customer SLA,
proof of investigation quality or permission to delay work. Persist each applicable deadline/clock in durable case state
with its own rule version, trigger, owner and escalation even when related SLOs are healthy. Handle a missed required
clock through the approved compliance/legal process, not merely as an availability incident.

## Release gate scorecard

| Gate | Evidence required | Promotion rule |
|---|---|---|
| Deterministic baseline | Same-case time/quality/cost and error profile | Agent adds material value for named families |
| Offline outcome | Held-out and rotating cases, slices, multiple trials | Meets family thresholds with no severe unexplained regression |
| Policy safety | Adversarial and permission suites | Zero hard-invariant violations |
| Reliability | Fault matrix, resume/compaction, concurrency, effects | All state/effect invariants and recovery objectives pass |
| Human factors | Blinded comparison, edit/return, fatigue/automation-bias study | Review remains meaningful and operationally staffed |
| Shadow | Representative live proposal-only traffic | Stable quality, capacity, cost, source coverage, and drift |
| Canary | Narrow family/tenant/queue with kill switch | Error-budget and incident criteria stay green |
| Independent challenge | Reproduction and limitations review | Findings resolved/accepted by accountable governance |

Roll back or reduce scope on a hard invariant, confidentiality event, false-negative cluster, mass false-positive burden, unexplained slice regression, unknown-effect backlog, deadline/capacity breach, source/list freshness failure, or loss of evaluation/audit visibility.

## Checklist

- [ ] Evaluation covers the full released system and keeps outcome, policy, and repeated-reliability gates separate.
- [ ] Labels record selection, cutoff, provenance, uncertainty, delay, and system influence; SAR/STR/disposition is not ground truth.
- [ ] Suites include historical, benign, synthetic, entity/sanctions, security, reliability, shadow, and post-release evidence.
- [ ] Leakage, contamination, evaluation-awareness, multiple trials, slices, severity, and independent grading are addressed.
- [ ] Trajectories grade scope, coverage, contradiction, replanning, stopping, compaction, and state application.
- [ ] Failure injection covers source, model, memory, workflow, concurrency, destination, capacity, provider, revocation, and DR.
- [ ] Audit, provenance, and telemetry are separate; telemetry content is minimized.
- [ ] SLOs and error budgets trigger concrete degradation, rollback, staffing, or stop actions.

## Sources and next guide

- [Anthropic — Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- [NIST — Strengthening AI Agent Hijacking Evaluations](https://www.nist.gov/news-events/news/2025/01/technical-blog-strengthening-ai-agent-hijacking-evaluations)
- [NIST — technical report on agents cheating evaluations](https://www.nist.gov/caisi/cheating-ai-agent-evaluations)
- [Google SRE — Service Level Objectives](https://sre.google/sre-book/service-level-objectives/)
- [OpenTelemetry — Generative AI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
- [W3C Trace Context](https://www.w3.org/TR/trace-context/)
- [Evaluation-driven development](../../evaluation/evaluation-driven-development.md)
- [Observability and tracing](../../evaluation/observability-and-tracing.md)

Next: [Deployment, scale, incidents, cost, and governed evolution](10-deployment-scale-incidents-cost-and-evolution.md).

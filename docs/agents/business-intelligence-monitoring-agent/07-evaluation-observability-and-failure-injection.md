# Evaluation, Observability, and Failure Injection

## Evaluation objective

The system is good only if it detects locally important changes with acceptable operator load, preserves evidence and authority, delivers intended workflow effects reliably, and helps accountable people reach verifiable outcomes. Model answer quality is one small part.

Use hard safety gates plus per-watch scorecards. Do not combine a tenant-isolation failure and a polished triage summary into one weighted average.

## Evaluation layers

~~~mermaid
flowchart TB
    U[Unit and contract tests] --> R[Historical watch replay]
    R --> T[Trajectory and repeated-run tests]
    T --> F[Fault and adversarial injection]
    F --> S[Shadow operation]
    S --> C[Canary delivery]
    C --> O[Online outcome and failure mining]
    O --> R
~~~

| Layer | What it proves | Typical artifact |
|---|---|---|
| Contract/unit | Schema, state transitions, formulas, rights, idempotency | Deterministic fixtures |
| Historical replay | Detector behavior, late data, revisions, alert load | Interval-labelled replay corpus |
| Trajectory | Tool choice, evidence use, loop bounds, stop behavior | Full trace and context manifest |
| Repeated-run | Stochastic triage reliability | Pass-at-k and failure distribution |
| Fault/adversarial | Recovery and security invariants | Injected failure report |
| Shadow | Live data behavior without effects | Candidate/current diff |
| Canary | Limited real delivery and acknowledgement | Scoped release cohort |
| Online outcome | Human load, latency, disposition, action, verified result | Production scorecard |

## Evaluation corpus

Build from local, versioned evidence:

- normal periods across seasons and calendars;
- owner-labelled material incidents and non-actionable anomalies;
- known semantic changes, backfills, data outages, promotions, and regime shifts;
- sparse, flat, volatile, high-cardinality, and ratio metrics;
- representative tenant and rights configurations using synthetic or approved data;
- tickets/comments with benign and adversarial instructions;
- duplicate, delayed, out-of-order, partial, and corrected observations;
- ambiguous external-effect transcripts and provider consistency delay;
- unowned routes, stale approvals, overloaded channels, and outcome gaps.

Prevent temporal leakage: detector baselines, prompt examples, runbooks, and outcome episodes available at evaluation time must reflect only what the production system would have known then.

## Scenario schema

~~~yaml
scenario_id: bi-watch/revenue/backfill-retraction/004
corpus_version: 2026-08-31
initial_state_ref: fixture://case/open-v4
watch_version: 17
semantic_snapshot: sha256:4b7...
timeline:
  - at: T0
    input: observation://revision-0
  - at: T+5m
    input: effect_result://timeout-after-remote-commit
  - at: T+2h
    input: observation://revision-1-no-trigger
faults:
  - effect_response_lost
  - duplicate_reconcile_event
adversarial_content:
  - source: itsm_comment
    text_ref: fixture://injection/recipient-change
expected:
  allowed_terminal_states: [retracted, indeterminate]
  required_events: [EffectOutcomeUnknown, ObservationSuperseded, CaseRevised]
  forbidden:
    - duplicate_remote_work_item
    - source_data_write
    - unapproved_recipient
    - causal_claim
  maximum:
    model_turns: 2
    semantic_reads: 3
    wall_time: PT5M
oracle:
  owner_labels_ref: labels://revenue-watch/004
  reviewed_by: [role:metric-owner, role:platform-sre]
~~~

## Detector metrics

Evaluate at the event/case level, not only point level:

| Metric | Interpretation | Caveat |
|---|---|---|
| Material-event recall | Fraction of owner-labelled material events detected | Labels can be incomplete and hindsight-biased |
| Alert precision/actionability | Fraction of opened cases judged actionable | Depends on action definition and capacity |
| Detection delay | Event onset to first valid signal/case | Earlier is not always better if data is provisional |
| False alerts per watch/week | Human load | Aggregate distribution; do not hide noisy watches |
| Missed material amount | Owner-defined impact not detected | Requires defensible impact labels |
| Case compression ratio | Signals grouped into actionable cases | High ratio can hide over-grouping |
| Flap/reopen rate | Instability of trigger/recovery | Revisions and real recurrence must be separated |
| Data-gate suppression rate | Visibility into unfit data | High rate may reveal source failure, not detector quality |
| Calibration by score band | Whether higher scores correspond to higher event likelihood/materiality | Requires enough comparable outcomes |

Precision and recall do not encode asymmetric business costs by themselves. Publish the decision threshold, alert capacity, and missed-event consequences alongside them.

### Benchmark limitations

Numenta Anomaly Benchmark provides labelled univariate time series and a scoring profile that rewards timely detection and penalizes false/late alerts. TimeEval provides reproducible evaluation infrastructure across datasets and algorithms. They are useful for testing detector implementations, not for proving a production watch:

- business semantics, aggregation, revisions, materiality, and action costs are absent;
- many benchmark datasets are univariate and labels may be synthetic or incomplete;
- train/test preprocessing and dataset flaws can make methods appear stronger than they are;
- results rarely include quality gates, human triage, delivery, or outcomes.

Research has documented flaws in widely used time-series anomaly benchmarks. Treat any leaderboard result as workload-bounded evidence. Local replay and shadow operation remain release gates.

## Triage and trajectory metrics

Score the complete trajectory:

- fact citation precision and evidence coverage;
- unsupported or causal claim rate;
- correct trust labeling and contradiction handling;
- correct choice of allowed drill-down and usefulness of its rationale;
- prohibited query/dimension/recipient attempt rate;
- route proposal agreement with the deterministic decision table;
- stop-rule adherence and budget use;
- context freshness and case-version correctness;
- repeated-run success distribution, not one favorable run;
- factual template fallback quality when the model is unavailable.

Human graders should see the same evidence available to the system and a rubric. Measure reviewer agreement; ambiguous gold labels should be revised rather than used as false certainty.

## Workflow and business metrics

| Stage | Measure |
|---|---|
| Evaluation | Schedule coverage, data-ready latency, terminal-state coverage |
| Delivery | Confirmed effect latency, duplicate rate, unknown-effect age |
| Ownership | Time to accountable route, unowned case count |
| Acknowledgement | Time to authenticated acknowledgement, escalation rate |
| Decision | Time to disposition, stale-approval rejection count |
| Follow-through | Assigned action completion, overdue action rate |
| Outcome | Verified-outcome coverage, time to verification, achieved/ineffective/indeterminate |
| Utility | Cases accepted as actionable, work avoided through grouping, missed material events |
| Human factors | Owner alert load, override rate, review time, reported trust/fatigue |

Do not claim ROI by attributing all post-alert improvement to the agent. Report notification, decision, action, outcome, and causal evidence separately.

## Counterfactual and outcome evaluation

Two different questions require different evidence:

1. **Behavioral counterfactual:** “What observation, case, route, and effect would the candidate behavior bundle have produced on the same information available at that time?” Historical replay and live shadow can answer this when time, versions, rights, and evidence availability are reconstructed.
2. **Impact counterfactual:** “What business outcome would have occurred without the alert or intervention?” Ordinary replay cannot answer this. It requires an experiment or a defensible causal design with documented assumptions.

Use this evaluation record:

~~~yaml
counterfactual_run:
  run_id: cf_01K...
  eligible_as_of: 2026-07-31T23:59:59Z
  corpus_version: revenue-watches/2026-08-31
  current_bundle: bi-monitoring/2026.07.18.4
  candidate_bundle: bi-monitoring/2026.08.31.2
  baseline:
    kind: deterministic_native_alert
    definition_ref: baseline://revenue-fixed-threshold/4
  prohibited_future_evidence:
    - revised data published after eligible_as_of
    - later operator labels
    - later incident or campaign annotations
    - realized outcomes
  paired_outputs:
    observations: artifact://counterfactual/observations
    cases_routes_effects: artifact://counterfactual/trajectories
  outcome_analysis:
    outcome_definition: outcome://revenue-recovery/3
    design: descriptive_only
    attribution: not_established
~~~

Evaluation must compare against the simplest viable baseline: no alert, a fixed native BI alert, the current deterministic detector, and the full current system as applicable. Report paired differences in material-event recall, delay, false cases, owner load, route correctness, query/model cost, and unknown effects. A candidate that improves prose but produces no better decision or reduces human capacity is not an improvement.

For outcome evaluation:

- pin intervention eligibility, assignment/exposure, decision, action, outcome definition, measurement window, censoring, data correction window, and attribution method;
- separate `not_notified`, `not_acknowledged`, `no_action`, `action_taken`, and `outcome_observed`; do not collapse them into success/failure;
- use randomized rollout or another approved experimental design where feasible; if using matched, difference-in-differences, interrupted time-series, or propensity methods, document exchangeability, overlap, interference, parallel-trend, measurement, and missingness assumptions as applicable;
- pre-register primary outcome and stopping rules for material decisions; show uncertainty intervals and sensitivity checks rather than a single uplift number;
- never use post-treatment variables, later revisions, or reviewed labels as features in the replay decision;
- retain `attribution: not_established` for descriptive before/after evidence.

Delayed and confounded outcomes are useful for failure review, prioritizing human investigation, and generating offline candidates. They do not authorize automatic threshold, route, or authority changes.

## Hard release gates

The following must be zero in the release corpus and canary:

- cross-tenant or unauthorized data retrieval;
- source-data mutation or high-impact business action;
- unapproved destination or secret exposure;
- business alert from stale/invalid data except an explicitly configured data-health rule;
- duplicate external effect for one semantic operation;
- effect execution under stale approval;
- unsupported fact presented as verified;
- missing durable record for an effect or acknowledgement;
- lost unknown-effect state across crash/compaction;
- automatic threshold, route, or authority expansion.

Additional per-watch gates are owner-set: maximum false alerts, missed-event rate, detection delay, route accuracy, acknowledgement capacity, and cost.

## Failure injection catalog

| Injection | Expected invariant/evidence |
|---|---|
| Duplicate schedule and queue delivery | One evaluation identity; idempotent resume |
| Crash before/after observation commit | No lost or duplicate observation |
| Semantic response partial or reordered | Typed partial; no detector unless contract permits |
| Quality service timeout | Indeterminate/data-health state, not business zero |
| Baseline artifact corrupted | Detector stops and watch suspends/escalates |
| Model returns invalid enum/citation | Proposal rejected; bounded repair or fallback |
| Ticket commits then response drops | Unknown effect; reconciliation finds one object |
| Approval expires in queue | Credential not issued; effect rejected |
| Backfill reverses trigger | Superseding observation and visible retraction/revision |
| Acknowledgement event duplicated/out of order | One legal transition; audit retained |
| Cancellation during provider outage | No blind assumption; reconcile later |
| Shared upstream outage | One root data incident; child business alerts inhibited |
| Queue overload | Criticality/fairness/deadline admission; visible deferral |
| Model/provider quota exhausted | Deterministic detection and human-readable factual template continue |
| Cross-tenant ID in tool result | Evidence quarantined; incident signal |
| Indirect injection asks for new recipient/tool | No authority change; adversarial test passes |
| Compaction drops pending approval/effect | Checkpoint rejected or reconstruction restores it |
| Outcome source delayed/confounded | Indeterminate, not fabricated success |
| Query preflight underestimates final warehouse cost | Hard server/client budget stops or quarantines the read; measured cost retained |
| Final page/chunk is lost or duplicated | Receipt is partial; detector does not run on an apparently complete prefix |
| Lineage `COMPLETE` arrives without `START` or expected dataset facet | Gate remains indeterminate; no inferred complete lineage |
| BI alert/content is copied or moved and provider identity changes | Contract drift is detected; watch suspends or shadows before rebind |
| Chat response drops after remote message commit | Unknown effect; qualified lookup finds one message or blocks retry |
| Email provider returns asynchronous acceptance only | Accepted-for-processing remains distinct from confirmed delivery/acknowledgement |
| Decision definition changes while a case waits | Old definition remains pinned or case is explicitly migrated and re-approved |
| Outcome backfill reverses the apparent result | New outcome observation supersedes the prior one; episode and eval lineage remain |
| Behavior-bundle rollback with in-flight effects | Old/new workers fence by bundle and operation key; no duplicate or reinterpretation |

Run faults at every crash point around state commit, outbox publication, remote effect, receipt, and outcome check.

## Observability model

Use one trace or linked trace set across:

`trigger -> evaluation -> semantic query -> quality gate -> detector -> case -> model triage -> policy -> approval -> effect -> acknowledgement -> outcome`.

Stable attributes:

- tenant pseudonymous identifier;
- watch, watch version, evaluation, observation, signal, case, case version;
- semantic, detector, baseline, policy, decision-table, prompt, model, tool, and release versions;
- operation key hash and remote provider type;
- state transition, retry class, queue age, cost class, and terminal reason;
- data-health and outcome status without sensitive metric values by default.

Follow [OpenTelemetry](https://opentelemetry.io/docs/specs/otel/) conventions where stable and keep an application-owned event schema because GenAI semantic conventions continue to evolve.

Do not merge four different operational records:

| Record | Optimized for | Typical contents | Reliability and retention rule |
|---|---|---|---|
| Metric | Bounded aggregation, SLI/SLO and capacity alerting | Counts, rates, histograms and bounded dimensions | Never carry raw metric values, recipients, query text, IDs, prompts, or evidence URLs as labels |
| Trace | Causal execution path and latency/failure diagnosis | Linked spans from trigger through outcome, version attributes, errors and sampled events | Sampling is allowed by policy; trace loss must not lose workflow or audit truth |
| Diagnostic log | Local explanation and incident debugging | Structured adapter/runtime diagnostics, redacted provider errors, correlation IDs | Access-controlled, retention-bounded, and not used as an authoritative state machine |
| Audit record | Who/what/when/why for authority and evidence | Actor, command, prior/new version, policy/approval, evidence and effect references, integrity data | Durable, append-only or tamper-evident, complete for scoped actions, independently queryable |

SLOs are computed from authoritative terminal state or reconciled receipts, not from log-line presence, span success, or HTTP status. An audit record proves that a governed command was recorded; it does not prove provider delivery or realized business outcome.

### Cardinality and privacy

Do not use raw metric values, recipients, customer IDs, query text, prompts, operation keys, or artifact URLs as metric labels. Put high-cardinality identifiers in traces/logs with access controls; use bounded labels for metrics. Sample successful low-risk traces, but retain full audit records and increase trace sampling for failures under policy.

## SLOs

Define workload classes. Example targets are illustrative, not defaults:

| SLI | Definition | Example class |
|---|---|---|
| Eligible evaluation coverage | Terminal eligible evaluations / scheduled eligible evaluations | Critical daily |
| Observation readiness latency | Data-ready time to accepted observation | Critical daily |
| Valid-signal processing latency | Accepted observation to case transition | Critical daily |
| Confirmed delivery latency | Effect intent to confirmed/definitive state | Page/high |
| Acknowledgement latency | First confirmed delivery to authorized acknowledgement | High |
| Unknown-effect age | Time effect remains unreconciled | All effects |
| Outcome verification coverage | Eligible terminal cases with verified outcome/disposition | Decision ops |
| Duplicate semantic effect rate | Duplicate remote effects / intended operations | All effects |

Pair latency with quality and safety. A fast alert on invalid data is not success.

Track error-budget burn by failure class:

- data readiness;
- semantic/quality/detector;
- state/runtime;
- model triage;
- delivery/reconciliation;
- owner/acknowledgement;
- outcome verification.

Multi-window burn-rate alerting is a strong SRE pattern for service SLOs, but low-volume business watches may lack enough events for statistically stable burn calculations. Use per-watch deadlines and portfolio-level SLOs where appropriate.

## Dashboards and alerts

Operations dashboard:

- due/completed/skipped evaluations by criticality and tenant;
- freshness and quality gate failures by root data product;
- open cases by state, severity, owner, age, and deadline;
- alert load and actionability by watch;
- queue age, admission denials, model/query cost, and provider rate limits;
- effect states, unknown age, duplicates, and reconciliation backlog;
- stale approvals, unowned routes, and acknowledgement breaches;
- outcome verification backlog and indeterminate rate;
- release/canary cohort and failure regression.

Platform alerts should be actionable. If no operator response exists, keep the signal as a dashboard or digest.

## Failure mining and continuous evaluation

Every material production failure becomes:

1. a normalized failure taxonomy label;
2. a redacted/reproducible trace and state fixture;
3. a regression scenario with deterministic invariants;
4. an owner-reviewed detector/triage/workflow label where relevant;
5. a release gate at the correct scope;
6. a runbook or observability improvement;
7. a record of whether the fix changes semantics, authority, cost, or SLO.

Mine:

- repeated false positives and ignored watches;
- cases with late or wrong routing;
- model proposals rejected by validators or humans;
- unknown effects and manual reconciliations;
- revisions/retractions after delivery;
- outcome gaps and ineffective actions;
- over-budget contexts, queries, and alert channels;
- incidents where the non-agent baseline performed better.

Do not train automatically on production outcomes. Labels are often delayed, confounded, strategic, or wrong.

## Evaluation checklist

- [ ] Corpus versions pin all runtime, semantic, detector, policy, prompt, model, and tool inputs.
- [ ] Temporal leakage and outcome survivorship bias are assessed.
- [ ] Per-watch event-level metrics and operator load are reported.
- [ ] Trajectories, repeated stochastic runs, and deterministic fallback are tested.
- [ ] Hard safety gates cannot be averaged away.
- [ ] Fault injection covers every state/effect crash boundary.
- [ ] Shadow and canary releases produce current-versus-candidate diffs.
- [ ] SLOs measure verified workflow outcomes, not model or HTTP success alone.
- [ ] Telemetry controls cardinality, prompt/value exposure, access, and retention.
- [ ] Every material production failure becomes a regression fixture.

## Primary references

- [Evaluation-driven development](../../evaluation/evaluation-driven-development.md)
- [Trajectory and reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md)
- [Observability and tracing](../../evaluation/observability-and-tracing.md)
- [Numenta Anomaly Benchmark](https://github.com/numenta/NAB)
- [TimeEval](https://github.com/TimeEval/TimeEval)
- [Current time-series anomaly detection benchmarks are flawed and are creating the illusion of progress](https://doi.org/10.1109/TKDE.2021.3112126)
- [Google SRE Workbook: alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)

# Detection, Triage, Routing, and Decision Workflows

## Separate the layers

Many unsafe systems collapse “unusual” into “important,” “caused by,” and “act now.” Preserve five distinct judgments:

| Layer | Question | Authority |
|---|---|---|
| Data fitness | Is the observation trustworthy for this purpose? | Contract and data-quality policy |
| Statistical signal | Does the observation differ from a pinned expectation? | Deterministic detector |
| Business materiality | Is the magnitude or budget impact worth attention? | Metric/watch owner |
| Triage | What related evidence and bounded hypotheses help interpret it? | Agent proposes; evidence remains authoritative |
| Decision/action | What should happen, by whom, with what approval? | Decision table and accountable human |

An anomaly score has no authority outside its approved watch.

## Detector selection ladder

Choose the lowest-complexity detector that meets the local objective:

| Level | Methods | Use when | Common failure |
|---|---|---|---|
| 1. Contract threshold | Absolute target, upper/lower bound, ratio, budget remaining | Business limit is explicit | Threshold churn and boundary flapping |
| 2. Comparative rule | Period-over-period, target variance, rolling change, rate of change | Stable comparable intervals exist | Calendar, denominator, and base-effect errors |
| 3. Statistical process rule | Control limits, EWMA, CUSUM, robust z-score | Historical in-control behavior is meaningful | Nonstationarity and contaminated baseline |
| 4. Seasonal forecast/change point | Seasonal residual, prediction interval, regime change | Strong calendar/seasonal structure exists | Undercoverage, promotions, and model drift |
| 5. Multivariate/ML | Joint features, learned event score | Sufficient representative data and operational value justify it | Opacity, unstable features, rights expansion, poor transfer |

Do not introduce ML because “anomaly detection” sounds like an ML problem. A material threshold with persistence often aligns better with the business action.

### Trigger anatomy

A useful trigger can combine:

- **direction:** above, below, either, or changed;
- **magnitude:** absolute and/or relative delta;
- **materiality:** currency, count, margin, budget, or affected cohort;
- **confidence:** interval, score, or rule support;
- **persistence:** consecutive eligible evaluations or duration;
- **hysteresis:** weaker recovery boundary than trigger boundary;
- **cooldown:** minimum time before a new case;
- **suppression/inhibition:** owner-approved event or upstream case;
- **rate limit:** watch, owner, tenant, and channel budget.

Keep these parameters inspectable. Do not hide them inside a prompt.

## Statistical cautions

Control charts compare current behavior with a historical model of a stable process. EWMA and CUSUM can detect smaller gradual shifts than a simple Shewhart rule, but only if the reference period is representative. Additional run rules increase sensitivity and can also sharply increase false alarms.

Business series add complications:

- fiscal and civil calendars;
- weekdays, holidays, promotions, launches, and one-off campaigns;
- changing customer mix and denominators;
- sparse counts and zero inflation;
- structural breaks from product or semantic changes;
- backfilled or revised observations;
- correlated metrics and duplicated signals;
- hundreds of authorized slices, creating multiple-testing risk.

An outlier label is an investigation prompt, not an explanation. Test event-level performance and operator load, not just point-wise accuracy.

### Multiple watches and slices

If a system evaluates `N` metrics across `M` dimensions, even a low per-test false-positive rate can create operational noise. Controls include:

- owner-approved slice catalog instead of arbitrary subgroup search;
- hierarchical evaluation: portfolio signal before drill-down;
- minimum sample/cohort and materiality floors;
- family-level error-control analysis where statistically appropriate;
- alert grouping and root/child relationships;
- per-owner and per-channel load gates;
- explicit evaluation of missed material events versus false work.

Online false-discovery research can inform sequential testing, but no generic method substitutes for business-specific costs and dependence structure.

## Detection record

~~~yaml
signal_id: sig_01K...
observation_id: obs_01K...
watch_id: revenue-drop-emea
watch_version: 17
detector:
  ref: detector://seasonal-relative/4.2.1
  baseline_ref: baseline://revenue-drop-emea/2026-08-01
results:
  score: -3.18
  expected: 946100.00
  observed: 731204.14
  absolute_delta: -214895.86
  relative_delta: -0.2271
rules:
  magnitude: pass
  materiality: pass
  persistence: 2_of_2
  recovery: false
decision: trigger
evidence_artifact_ref: artifact://detector/519...
replay_digest: sha256:192...
~~~

The record contains no route or business recommendation.

## Correlation and alert identity

Group only with deterministic keys or documented heuristics. Useful relationships include:

- same watch and contiguous intervals;
- parent portfolio metric and child segment metrics;
- shared semantic or data-health incident;
- same approved campaign, release, geography, or operational event;
- overlapping owner and decision table.

Grouping must not erase child evidence. Assign:

- `case_id`: durable decision-operations object;
- `dedup_key`: semantic identity for repeated delivery attempts;
- `root_case_id`: optional correlated parent;
- `case_version`: incremented when material evidence or proposed effect changes.

Prometheus Alertmanager is a useful reference for grouping, deduplication, routing, silences, and inhibition. Those mechanics do not by themselves provide acknowledgement, decision, task, and outcome semantics.

## Case lifecycle

~~~mermaid
stateDiagram-v2
    [*] --> Candidate
    Candidate --> Suppressed: valid suppression or duplicate
    Candidate --> Open: signal and materiality accepted
    Open --> Routed: accountable route resolved
    Routed --> Acknowledged: authorized subject accepts triage
    Routed --> Escalated: acknowledgement deadline
    Acknowledged --> Investigating
    Investigating --> AwaitingDecision: evidence packet complete
    AwaitingDecision --> ActionAssigned: approved delegated work
    AwaitingDecision --> NoAction: accountable disposition
    AwaitingDecision --> FalsePositive: evidence refutes alert
    ActionAssigned --> ObservingOutcome
    ObservingOutcome --> Resolved: outcome verified
    ObservingOutcome --> Ineffective: action completed, goal not met
    Open --> Revised: corrected observation changes material facts
    Revised --> Open: triage continues
    Revised --> Retracted: trigger no longer stands
    Escalated --> Acknowledged
    Open --> Indeterminate: evidence or effect cannot be reconciled
    Resolved --> [*]
    NoAction --> [*]
    FalsePositive --> [*]
    Retracted --> [*]
    Ineffective --> [*]
    Indeterminate --> [*]
    Suppressed --> [*]
~~~

Not every case must traverse every state. Transitions are checked by code against current version and actor authority.

### Terminal disposition vocabulary

| Disposition | Meaning | Required evidence |
|---|---|---|
| Resolved | Intended verified outcome achieved | Authoritative outcome check |
| Ineffective | Approved action occurred but intended outcome did not | Action receipt plus outcome evidence |
| No action | Authorized human chose not to act | Subject, rationale, case version |
| False positive | Observation was fit but detector/case was not actionable | Review evidence and calibration label |
| Retracted | Corrected data or semantics invalidated the trigger | Superseding observation |
| Duplicate/correlated | Another case owns follow-through | Root case link |
| Expired | Decision window elapsed under policy | Timers and escalation history |
| Indeterminate | Truth or effect could not be established | Reconciliation attempts and owner escalation |

Do not use `closed` without one of these meanings.

## Deterministic detection and model-triage boundary

The model is downstream of a valid, reproducible signal and upstream only of a validated proposal. Removing it must leave evaluation, detection, case identity, routing fallback, approvals, delivery, reconciliation, and outcome tracking operable.

| Step | Deterministic/controller-owned | Optional bounded model contribution | Model output cannot do |
|---|---|---|---|
| Data fitness | Rights, interval, freshness, completeness, quality and lineage gate | None | Convert invalid/partial/no-data into a value |
| Detection | Feature transform, pinned baseline, score, threshold, materiality, persistence, recovery and suppression | None | Create or retract a signal |
| Correlation | Stable keys and reviewed heuristics; root/child evidence preserved | Suggest a possible relationship for human/offline review | Merge cases or erase evidence |
| Evidence triage | Approved read templates, budgets, evidence validation and stop conditions | Select from allowlisted aggregate reads; organize cited facts and labelled hypotheses | Issue SQL, retrieve raw rows, claim cause, change versions or budgets |
| Routing | Versioned decision table, current directory, purpose/destination policy | Propose one route reference with cited reason | Select a recipient from text or authorize disclosure |
| Decision/effect | Actor authority, approval binding, ledger, credential issuance and reconciliation | Draft a factual handoff | Approve, send, retry, resolve or mutate source systems |
| Learning | Corpus construction, outcome verification, label review and release gates | Produce a review candidate | Self-edit a detector, threshold, route, prompt or authority |

Release tests include a **model-omission trajectory**: disable the provider at each possible call, then prove a factual deterministic packet, legal state transition, deadlines, effects already in flight, reconciliation, and audit remain correct. Model availability is not part of the detector SLO. Any model-generated fact without an evidence reference is rejected, not downgraded into a lower-confidence fact.

## Bounded model triage

### Context supplied

- exact case, watch, observation, semantic, detector, and quality versions;
- verified facts with source, retrieval time, and freshness;
- related cases and known-event records;
- an allowlist of drill-down templates and remaining budget;
- permitted hypotheses, prohibited claims, output schema, and stop rules;
- accountable route directory and decision table candidates;
- explicit untrusted-data labels for dashboard text, tickets, comments, and source metadata.

### Required output

~~~yaml
triage:
  facts:
    - claim: Net revenue was 22.71% below the pinned seasonal expectation.
      evidence_refs: [artifact://detector/519]
  hypotheses:
    - statement: Country mix may explain part of the change.
      support: not_tested
      causal_claim: false
  requested_reads:
    - tool: semantic_breakdown
      template: metric_by_allowed_dimension
      parameters: {dimension: country, top_k: 10}
      purpose: test whether the change is concentrated
  contradictions: []
  missing_evidence: [approved_campaign_calendar]
  route_proposal:
    route_ref: route://commercial-emea/3
    reason_refs: [watch://revenue-drop-emea/17]
  confidence: limited
  stop_reason: evidence_budget_reached
~~~

Validate every reference and enum. Reject uncited facts, causal language, invented owners, unknown tools, disallowed dimensions, or a route outside policy.

### Planning budget

Default to:

- at most three semantic drill-downs;
- at most two model turns;
- one related-case lookup;
- one known-event lookup;
- no raw-row retrieval;
- a strict token, query-cost, wall-time, and case-latency budget.

Budget changes require evaluation. More context and more loops can reduce reliability by adding irrelevant or adversarial material.

## Decision tables and approvals

A decision table can be represented in code, a policy engine, or optionally DMN. DMN provides a portable notation for decisions and rules; it does not establish that the rule is lawful, complete, or approved.

Example:

| Conditions | Route | Approval | Permitted effect | Owner |
|---|---|---|---|---|
| Data invalid or stale | DataOps | None for ticket | Create data-health incident | Data product owner |
| High materiality, confidential aggregate, business-hours | Commercial on-call | None for notification; human for action | Create case and page | Accountable operator |
| High materiality, outside hours | Duty manager | Human acknowledgement | Page and create ITSM task | Duty manager |
| Sensitive slice or cohort below floor | Privacy/risk | Required before disclosure | Redacted case only | Privacy owner |
| Consequential action contemplated | Governed decision forum | Explicit multi-party approval | Evidence packet only | Named decision authority |
| Route missing or owner unresolved | Platform queue | Required | Hold and escalate metadata defect | Watch owner |

The table consumes typed facts; it must not parse a free-form model recommendation as authority.

### Approval binding

An approval record binds:

- approver subject and authenticated role;
- tenant, case ID, case version, watch version;
- evidence manifest hash;
- exact effect type, target, payload digest, and scope;
- decision-table and policy versions;
- expiry and invalidation conditions;
- recorded rationale when required.

New evidence, a revised observation, changed target, payload, threshold, or policy invalidates the approval.

## Routing and delivery

Route to an accountable role or group, then resolve current membership at effect time. A model must never extract recipients from untrusted content.

Delivery rules:

- group related alerts into one actionable case;
- include what changed, why it is trusted, materiality, uncertainty, required response, and deep links;
- omit sensitive values from destinations that are not approved;
- use recovery notifications sparingly and correlate them with the original case;
- apply pending periods, cooldowns, silences, and inhibition deliberately;
- rate-limit by watch, owner, tenant, and channel;
- test fallback when the primary channel or owner directory fails.

If no action or decision can reasonably follow, publish a dashboard or digest instead of an alert.

## Acknowledgement and escalation

Acknowledgement must capture subject, role, timestamp, case version, and optional assignment. It is not equivalent to ticket creation, notification read, or remediation.

Timer behavior:

1. Persist a timer due event.
2. Re-read current case and acknowledgement version when it fires.
3. If still eligible, create an escalation effect with a stable key.
4. If cancellation races delivery, reconcile the remote effect and record late delivery.
5. Escalation never grants the recipient new data rights.

## Follow-through and outcome feedback

Separate:

- **decision:** what the accountable actor chose;
- **assigned action:** work delegated to another system or person;
- **action receipt:** evidence that the action occurred;
- **leading outcome:** early operational signal;
- **realized outcome:** metric or business result after an owner-approved window;
- **attribution:** whether the action caused the result, usually unknown without stronger study.

An improved metric after an action is temporal association, not automatically causal effect. If causal evidence matters, open a bounded analytics or experimentation workflow.

Verified episodes may include alert label, disposition, action class, time-to-acknowledge, time-to-decision, time-to-outcome, and owner assessment. They may support offline calibration and retrieval of prior runbooks. They must not silently rewrite thresholds or decision tables.

## Failure patterns

| Failure | Unsafe response | Correct response |
|---|---|---|
| Baseline drift | Auto-train on latest window | Shadow a candidate baseline; exclude incidents; owner promotion |
| Alert flap | Send every crossing | Hysteresis, persistence, grouped recovery |
| Related metric burst | Ask model to summarize all | Deterministic hierarchy/correlation, bounded evidence |
| Ticket timeout | Retry immediately | Mark unknown outcome and query by operation key |
| Backfill removes anomaly | Delete alert history | Supersede observation; revise or retract case visibly |
| Operator clicks acknowledge | Mark resolved | Start/continue decision timer |
| Model suggests “campaign caused drop” | Treat as fact | Label unsupported hypothesis; request governed evidence |
| Small subgroup dominates | Expose rows | Apply cohort floor, rights, and approved aggregate |
| Threshold produces constant noise | Add more AI triage | Revisit actionability or retire the watch |

## Production checklist

- [ ] Detector complexity is justified against a simpler rule.
- [ ] Event-level false alerts, misses, detection delay, and owner load are measured per watch.
- [ ] Baseline, exclusions, trigger, recovery, persistence, cooldown, and materiality are versioned.
- [ ] Correlation preserves child evidence and stable case identity.
- [ ] Model triage has typed output, cited facts, allowlisted reads, and strict budgets.
- [ ] Decision rules and approvals are deterministic, versioned, and challengeable.
- [ ] Acknowledgement, decision, assigned action, receipt, and outcome are separate.
- [ ] Revision, retraction, false-positive, no-action, ineffective, and indeterminate states are supported.
- [ ] High-impact action stays with an accountable human or governed external system.
- [ ] Feedback changes enter offline evaluation and reviewed release, not online self-modification.

## Primary references

- [NIST/SEMATECH process monitoring](https://www.itl.nist.gov/div898/handbook/pmc/section1/pmc12.htm)
- [NIST/SEMATECH EWMA control chart](https://itl.nist.gov/div898/handbook/mpc/section2/mpc2211.htm)
- [NIST/SEMATECH control-chart run-rule cautions](https://www.itl.nist.gov/div898/handbook/pmc/section3/pmc32.htm)
- [NIST/SEMATECH outlier detection](https://itl.nist.gov/div898/handbook/eda/section3/eda35h.htm)
- [Prometheus Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/)
- [Grafana alerting best practices](https://grafana.com/docs/grafana/latest/alerting/guides/best-practices/)
- [OMG Decision Model and Notation 1.5](https://www.omg.org/spec/DMN/1.5/About-DMN)
- [Online false discovery rate control under local dependence](https://papers.neurips.cc/paper_files/paper/2021/hash/def130d0b67eb38b7a8f4e7121ed432c-Abstract.html)

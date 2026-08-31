# Sampling, Test Design, and Control Assessment

> **Purpose:** Turn a human-approved assessment plan into a reproducible population, sample, procedure record, and source-linked workpaper without delegating professional judgment to the model.

## Assessment support, not autonomous auditing

[NIST SP 800-53A Revision 5](https://csrc.nist.gov/pubs/sp/800/53/a/r5/final) describes customizable assessment procedures using examine, interview, and test methods. In the financial-statement audit context, [PCAOB AS 2201](https://pcaobus.org/oversight/standards/auditing-standards/details/AS2201) distinguishes evaluating design and testing operating effectiveness. These sources have different scope and authority, but both reinforce a useful engineering boundary: procedure definition, evidence evaluation, and conclusion require qualified judgment; collection and reproducible execution can be assisted.

This blueprint uses TOD and TOE as configurable workpaper concepts:

- **Test of design (TOD) support:** assemble the control objective, risk, actors, frequency, dependencies, system configuration, walkthrough records, control description, and candidate design gaps. A human decides whether design addresses the objective in the applicable engagement.
- **Test of operating effectiveness (TOE) support:** freeze an approved population, execute an approved sample plan, collect exact sample evidence, apply approved steps, and draft observations. A human decides deviations, sufficiency, severity/materiality, and conclusion.

Do not universalize PCAOB terminology to an engagement governed by another standard. A profile overlay defines the actual procedure vocabulary and required approvals.

## Assessment plan contract

```yaml
assessment_plan_id: ap_01K...
engagement_id: eng_fy26_soc2
profile_ref: profile_soc2_org_v7@sha256:...
control_id: CC6.1-org-01
objective: "Evaluate approved access changes for the engagement period"
assertions_or_risks:
  - unauthorized_access_change
procedure_type: TOE
methods: [examine, reperform]
period: {from: 2026-01-01, to: 2026-06-30}
population_definition_ref: popdef_access_changes_v4
population_source_contract: github-access-change-view@3
sampling:
  method: random_without_replacement
  sample_size: 25
  stratification: [{field: risk_tier, include_all: critical}]
  seed_commitment: sha256:approved-secret-seed...
  replacement_policy: no_automatic_replacement
steps:
  - id: step_1
    action: compare_change_actor_to_approved_request
  - id: step_2
    action: compare_approval_time_to_change_time
  - id: step_3
    action: verify_approver_authority_at_source_time
evidence_requirements:
  - source_change_record
  - approval_record
  - historical_authorization_record
deviation_rules_ref: procedure-policy://access-change/v5
prepared_by: human:tester_8
approved_by: human:reviewer_12
approved_at: 2026-08-29T12:00:00Z
plan_digest: sha256:...
```

The model may draft questions or procedure text before approval. The runtime only executes a signed plan digest.

## Design evaluation workpaper

A useful TOD workpaper separates description from judgment:

| Section | Required record |
| --- | --- |
| Objective and risk | Profile-pinned objective, risk/assertion, scope, period, and procedure |
| Control as described | Owner-approved description, actors, trigger, frequency, system, dependencies, failure handling |
| Walkthrough | Dated items traced from initiation through completion, participants, evidence versions, unanswered questions |
| Configuration and authorization | Source snapshots and effective-time identity/role facts |
| Contradictory evidence | Missing paths, bypasses, manual overrides, incident/exception records, conflicting representations |
| Candidate observation | Model or preparer proposal with exact citations and uncertainty |
| Reviewer decision | Accept/reject/rework, rationale, identity, independence check, date, referenced inputs |
| Conclusion | Human-authored or human-approved language under the engagement’s methodology |

Inquiry alone is weak support for many objectives. Where the applicable methodology requires it, combine inquiry with inspection, observation, walkthrough, reperformance, or other approved procedures. The model cannot infer operating effectiveness from a policy document.

## Population contract

```json
{
  "population_id": "pop_01K...",
  "plan_id": "ap_01K...",
  "definition_id": "popdef_access_changes_v4",
  "definition_digest": "sha256:...",
  "source_query_digest": "sha256:...",
  "source_artifacts": [
    {"artifact_id": "ev_pop_01", "version": 1, "digest": "sha256:..."}
  ],
  "period": {"from": "2026-01-01", "to": "2026-06-30"},
  "freeze_time": "2026-08-30T08:00:00Z",
  "record_count": 1842,
  "canonical_record_schema": "access-change-population.v4",
  "canonical_order": ["source_event_time", "source_record_id"],
  "population_digest": "sha256:...",
  "completeness": {
    "status": "complete_for_approved_query",
    "reconciliation": {
      "source_reported_count": 1842,
      "extracted_count": 1842,
      "duplicates_removed": 0,
      "rejected_records": 0
    },
    "limitations": []
  },
  "frozen_by": "workload:sampling-service",
  "verified_by": "human:tester_8"
}
```

Population completeness is evaluated before sample selection. A sample from an incomplete or wrong population does not become valid because its selection was random.

## Sampling method decisions

| Method | Appropriate use | Agent boundary |
| --- | --- | --- |
| 100% examination | Small/high-risk population or automated deterministic test where methodology permits | Execute approved test; preserve false-positive/negative validation |
| Random sampling | Each population item should have a known selection chance | Deterministic PRNG/algorithm and approved seed; no model selection |
| Systematic sampling | Approved interval and random start are suitable | Freeze canonical ordering and parameters before selection |
| Stratified sampling | Risk/value/type strata require distinct coverage | Reviewer defines strata and allocation; service executes |
| Attribute sampling | Deviation-rate inference under an approved methodology | Qualified professional sets expected/tolerable deviation and sample plan |
| Monetary-unit/value-weighted sampling | Engagement methodology uses value-proportional selection | Specialist-approved implementation; model not the calculator |
| Judgmental/targeted selection | Specific risk, anomaly, key item, or unpredictability objective | Human rationale; do not present as statistical representation |
| Continuous/100% analytics | Full-period rule execution produces alerts | Validate data coverage and rule performance; still investigate exceptions and methodology limits |

Both statistical and nonstatistical sampling require professional judgment in audit contexts. The agent must not infer a sample size from “industry best practice.” Store the actual methodology, parameters, approving person, and version.

## Deterministic sample execution

1. Verify the approved plan digest and current profile pin.
2. Resolve and verify the frozen population digest.
3. Canonicalize record identities and ordering using the versioned algorithm.
4. Reveal or derive the approved seed only after population freeze; verify its prior commitment if the methodology uses one.
5. Apply the versioned deterministic selection algorithm and strata rules.
6. Produce a sample manifest before requesting sample-specific evidence.
7. Have an authorized tester/reviewer verify the population, parameters, and manifest.
8. Lock selection; any change creates a new plan/sample version with rationale.

```yaml
sample_manifest_id: sm_01K...
plan_id: ap_01K...
plan_digest: sha256:...
population_id: pop_01K...
population_digest: sha256:...
algorithm: deterministic-random-without-replacement@2.1.0
seed_ref: sealed://sampling-seed/ss_188
seed_commitment: sha256:...
selected:
  - ordinal: 1
    population_record_id: change_000187
    stratum: critical
  - ordinal: 2
    population_record_id: change_001042
    stratum: standard
selection_count: 25
generated_at: 2026-08-30T08:10:00Z
generated_by: workload:sampling-service
verified_by: human:reviewer_12
manifest_digest: sha256:...
```

Seed handling prevents opportunistic reselection. It is not a cryptographic proof that the population was unbiased or complete.

## Missing sample evidence and replacement

Never replace a selected item merely because its evidence is inconvenient, missing, or contradictory. Record one of:

- `evidence_pending` with owner/deadline;
- `source_record_not_found` with reconciliation evidence;
- `evidence_unavailable` with cause and assessor decision;
- `population_error` requiring a population/plan decision;
- `scope_error` requiring engagement reapproval;
- `deviation_candidate` under the approved methodology;
- `replacement_authorized` with human authority, rationale, original item retained, and a new sample version.

Automatic replacement biases the sample and erases precisely the failures an audit may need to observe.

## Test-instance and observation schema

```json
{
  "test_instance_id": "ti_01K...",
  "plan_id": "ap_01K...",
  "sample_manifest_id": "sm_01K...",
  "population_record_id": "change_001042",
  "procedure_version": "proc_toe_access_v4",
  "input_manifest": [
    {"role": "change", "artifact": "ev_change_42@1", "digest": "sha256:..."},
    {"role": "approval", "artifact": "ev_approval_42@1", "digest": "sha256:..."},
    {"role": "authorization", "artifact": "ev_auth_42@2", "digest": "sha256:..."}
  ],
  "step_results": [
    {
      "step_id": "step_2",
      "status": "candidate_deviation",
      "facts": [
        {"field": "change_time", "value": "2026-04-02T10:02:00Z", "citation": "ev_change_42@1#/time"},
        {"field": "approval_time", "value": "2026-04-02T10:17:00Z", "citation": "ev_approval_42@1#/time"}
      ],
      "conflicts": [],
      "missing": ["approved timing tolerance"],
      "prepared_by": "model-proposal:prop_01K..."
    }
  ],
  "candidate_observation": {
    "type": "timing_exception_requires_interpretation",
    "claim_scope": "this_sample_only",
    "supporting_evidence": ["ev_change_42@1", "ev_approval_42@1"],
    "contradicting_evidence": [],
    "uncertainty": "procedure parameter absent"
  },
  "review_state": "awaiting_independent_review",
  "record_digest": "sha256:..."
}
```

Store facts, candidate observations, reviewer decisions, and conclusions separately. A review must not overwrite the candidate; it appends a disposition linked to the exact input manifest.

## Source information produced by the entity

When an organization generates a report or export used as evidence, evaluate the report’s source, logic, parameters, completeness, and accuracy under the applicable methodology. Useful checks include:

- who can change report/query logic and whether changes are controlled;
- whether parameters match the approved population and period;
- whether the export omits states, regions, deleted records, manual paths, or late events;
- whether record counts reconcile to an independent source;
- whether timestamps and time zones are consistent;
- whether joins or transformations reject records;
- whether privileged users can alter source data before extraction;
- whether an independent evidence source can corroborate critical facts.

The model may identify these questions; it cannot assume the report is complete because it came from an API.

## Contradictory evidence

Create an explicit contradiction record:

```yaml
contradiction_id: con_01K...
claim: "Every production change had prior approval."
supports:
  - artifact: ev_policy_01@2
    locator: page:14
contradicts:
  - artifact: ev_change_42@1
    locator: json:/time
  - artifact: ev_approval_42@1
    locator: json:/time
scope: sample:change_001042
status: open
proposed_by: model-proposal:prop_01K...
review_owner: human:reviewer_12
```

Retrieval and summarization must include contradiction candidates, not optimize only for supporting evidence. Never drop an inconvenient artifact during compaction or package narrative generation.

## Continuous evidence is not a continuous opinion

Continuous collection can improve freshness and reduce period-end work, but it changes the operating problem:

- connector/schema changes can create false gaps or false improvements;
- event retention and late-arrival watermarks become control inputs;
- repeated observations may be correlated rather than independent;
- control changes split the period into different design/operation regimes;
- unresolved alerts and reviewer capacity accumulate;
- a continuously “green” vendor status still requires engagement-specific evaluation.

Freeze named checkpoints with source/profile/connector/rule versions. A continuous evidence signal is an observation stream, not a standing compliance assertion.

## Stage 3: approved sampling and TOD/TOE support

### Authority

C3 only for exact, human-approved plan digests. The system may freeze approved populations, select samples deterministically, run allowed read queries, and draft workpapers. It cannot choose methodology or sample size, replace missing items, accept evidence, dispose deviations, or conclude effectiveness.

### Architecture

Add the population registry, deterministic sampling service, plan signature verification, test-instance/workpaper service, contradiction index, reviewer decision surface, and linkage from every step result to exact evidence versions.

### Inputs and outputs

Inputs are approved plans, population definitions, source snapshots, sample parameters, evidence requirements, and profile-pinned procedure steps. Outputs are frozen population and sample manifests, test instances, cited facts, contradiction/missing-evidence records, candidate observations, and reviewer work queues.

### State, events, and effects

Population states: `defining → collecting → reconciling → frozen | rejected → superseded`. Sample states: `planned → selected → verified → locked → amended`. Test states: `not_started → evidence_pending → prepared → awaiting_review → rework | reviewed`. External effects remain evidence requests/reads; sampling and workpaper changes are domain events, not external effects.

### Approvals

A qualified person approves population, method, sample size, strata, seed policy, replacements, procedure steps, and deviation rules. Independence policy determines whether preparer and reviewer must differ. Changes after freeze require a new version and impact decision.

### Recovery

Rebuild population/sample from manifests, resume evidence collection without reselection, preserve selected-but-unavailable items, invalidate workpapers when an input artifact/profile/procedure is withdrawn, and route conflicting/missing parameters to human review.

### Evaluation

- population definition and count reconciliation;
- byte-for-byte reproducibility of population/sample manifests;
- zero unapproved replacement or post-freeze selection mutation;
- step-level fact extraction/citation and temporal comparison accuracy;
- contradiction recall and unsupported-conclusion rate;
- reviewer agreement, rework, override categories, time, and fatigue;
- behavior across control frequency, evidence format, source, and exception type;
- model outage and deterministic/manual fallback.

### Measurable exit gate

Stage 3 passes only when:

1. the same approved plan, population bytes, algorithm, and seed reproduce the identical ordered sample manifest;
2. every selected item remains traceable through missing, replaced, tested, or excluded states with human authority for any change;
3. every workpaper fact and candidate observation resolves to exact evidence versions and procedure steps;
4. hard tests show zero model-selected sample sizes, silent replacements, conclusion writes, or accepted uncited claims;
5. reviewers can reconstruct every test instance without the chat transcript and meet approved quality/workload thresholds; and
6. injected incomplete populations, late events, source report errors, contradictory evidence, profile withdrawal, and worker loss reach the designed recovery states.

## Test-readiness checklist

- [ ] Applicable methodology and procedure version are approved for this engagement.
- [ ] Population definition, source query, period, counts, exclusions, duplicates, rejected records, and limitations are visible.
- [ ] Sample parameters and replacement rules are human approved before selection.
- [ ] The algorithm, seed commitment/reference, canonical ordering, and manifest are preserved.
- [ ] Evidence requirements include source reliability and completeness checks, not only artifact presence.
- [ ] Supporting, contradicting, missing, and indeterminate evidence have distinct fields.
- [ ] Candidate observation, reviewer decision, exception, and conclusion are different records.
- [ ] The reviewer can request more evidence or rework without losing the original proposal.
- [ ] Continuous signals are checkpointed and never labeled a continuous opinion.

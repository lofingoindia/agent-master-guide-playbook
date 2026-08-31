# Requisitions, Recruiting, Interviews, and Human Decisions

> **Purpose:** Make recruiting evidence consistent, job-related, accessible, and reconstructable while ensuring the agent never selects, rejects, ranks, or otherwise decides a person's opportunity.

## The recruiting pipeline is an evidence protocol

The workflow should answer five questions at every transition:

1. Is the requisition and job analysis approved and current?
2. Is the evidence relevant to predefined job-related criteria?
3. Did every candidate receive the required notice, accessible process, and accommodation route?
4. Who observed, scored, decided, approved, and changed the record?
5. Can the decision be reviewed without relying on model rationale?

The model can reduce clerical and synthesis work. It cannot determine who “fits,” predict personality or emotion, or collapse the evidence protocol into a score.

## Requisition contract

```yaml
requisition:
  requisition_id: req_204
  version: 12
  legal_entity_id: entity_04
  work_locations: [in-ka-bengaluru]
  worker_type: employee
  position_ids: [pos_81]
  openings: 2
  job_analysis_ref: ja_backend_2026_v4
  required_competencies: [service_design, incident_reasoning, communication]
  essential_functions_ref: ef_backend_2026_v3
  assessment_plan_ref: ap_backend_2026_v2
  compensation_band_ref: band_p4_in_2026_v1
  approvals: [workforce_plan, finance, hr]
  policy_profile: recruiting_in_2026_08
  valid_from: 2026-09-01
  valid_until: 2027-02-28
```

The agent may draft this record, detect missing fields, compare it to approved artifacts, and propose corrections. Deterministic validation owns headcount, band, location, approval, posting, and policy checks. Authorized humans approve and open it.

## Requisition admission decision table

| Condition | Automated action | Human owner | Result |
|---|---|---|---|
| Job analysis, position, band, and approvals present and current | Validate versions and completeness | HR/requisition owner confirms | Eligible to open |
| Job description conflicts with essential functions | Cite conflict; block publication | HR + job owner; accessibility/legal if needed | Revise |
| Criteria include vague “culture fit,” age-coded language, or non-job-related proxy | Flag exact text and evidence gap | HR/assessment owner | Remove or substantiate |
| Jurisdiction or work location unresolved | No policy selection | HR/legal | Stop |
| Model drafted materially new minimum qualification | Mark unsupported; do not add | Job analysis owner | Reject proposal |
| Assessment vendor/version not approved for this job/population | Block attachment | Assessment/fairness owner | Validate or choose alternative |
| Posting change occurs after applications begin | Create cohort/change impact review | HR owner | Version, notify, or restart per policy |

## Candidate and application intake

Separate eligibility fields, job-related evidence, voluntary demographic monitoring, accommodation data, and background/medical records. Recruiters and models should not receive fields they do not need.

| Data compartment | Ordinary recruiting context? | Rule |
|---|---|---|
| Candidate contact and application | Purpose-scoped subset | Use only for named requisition/process |
| Resume/portfolio/work sample | Yes, with untrusted-content label | Treat instructions/links as data; cite evidence |
| Voluntary demographic monitoring | No | Separate access and analytics; never show selector/model |
| Accommodation request/medical support | No | Separate confidential team/process; expose only process status/approved adjustment |
| Background report | No by default | Restricted adjudication process and legal prerequisites |
| Interview notes/scores | Limited by role and stage | Structured rubric; protect from cross-candidate leakage |
| Prior application | Only under explicit policy/purpose | Do not create hidden cross-job reputation memory |
| Public/social data | Disabled by default | High privacy/proxy/relevance risk; use only under approved, lawful, job-related policy |

### Intake outcomes

- `valid`: required fields and evidence received;
- `needs_candidate_action`: specific missing item with accessible path;
- `needs_accommodation_routing`: transfer to separate confidential process without details;
- `possible_duplicate`: steward review;
- `unreadable`: alternate parser/format or human transcription;
- `conflict`: inconsistent material facts requiring clarification;
- `withdrawn`: candidate-authorized closure;
- `out_of_scope`: wrong requisition/tenant/process.

Absence is not negative evidence. A parsing failure, accessibility barrier, nonstandard career path, name difference, or employment gap must not be silently converted into a low score.

## Selection procedure registry

Every assessment, screen, interview, model feature, and human rubric is a versioned selection procedure if it affects opportunity.

```yaml
selection_procedure:
  procedure_id: structured_interview_backend_p4
  version: 6
  intended_jobs: [backend_p4]
  construct: incident_reasoning
  job_analysis_ref: ja_backend_2026_v4
  administration: trained_panel
  scoring_rubric_ref: rubric_ir_v6
  accommodations: [extended_time, accessible_format, alternate_channel]
  validation_refs: [content_validity_2026_02, pilot_2026_05]
  monitoring_plan: selection_monitor_17
  vendor_model_version: null
  prohibited_uses: [emotion_inference, personality_inference, cross_job_ranking]
  approved_jurisdictions: [profile_set_32]
  owner: assessment_governance
```

The registry must preserve how the procedure is actually used—threshold, weighting, sequence, population, job, language, accommodation, and vendor/model version. A generic vendor claim is not local validity evidence.

## Structured interview design

Official OPM guidance describes structured interviews as asking predetermined, job-related questions and evaluating responses with the same rating scale and standards. Use that as an engineering pattern, while qualified assessment and legal owners validate the organization's exact procedure.

### Before the cohort opens

- approve job analysis, competencies, essential functions, questions, allowed probes, anchored rubric, weightings, panel rules, accommodation options, and evidence retention;
- version and lock the interview kit for the cohort;
- train interviewers and record eligibility/conflicts;
- test candidate and interviewer surfaces with assistive technology and disability-led scenarios;
- define missing/technical-failure/reschedule rules that do not penalize the candidate;
- establish disparity, rater, and candidate-experience monitoring.

### During each interview

- use the same core questions and allowed probe policy;
- record evidence observations, not personality or protected-characteristic speculation;
- have panelists score independently before group discussion;
- require citations to candidate response/work sample and rubric anchor;
- record deviations, technical failures, accommodations applied, conflicts, and recusals;
- do not show a model-generated score or recommendation before independent human scoring.

### Afterward

The agent may check completeness, find unsupported score/evidence mismatches, summarize disagreements by rubric, and prepare a decision package. It must not average away significant disagreement, choose the candidate, or generate a “culture fit” conclusion.

## Interview evidence schema

```json
{
  "interview_id": "int_770",
  "application_id": "app_455",
  "kit_version": "backend-p4-v6",
  "question_id": "IR-03",
  "competency_id": "incident_reasoning",
  "observer_id": "panelist_29",
  "observation": "Candidate identified retry ambiguity and proposed status reconciliation.",
  "evidence_refs": ["transcript:770#t=12:14-13:02"],
  "rubric_anchor": 4,
  "score": 4,
  "confidence": "sufficient_evidence",
  "deviations": [],
  "recorded_before_panel_discussion": true,
  "recorded_at": "2026-08-31T10:05:00Z"
}
```

If recording/transcription is not approved or available, use interviewer notes linked to question and rubric. A transcript is sensitive evidence, not an automatic requirement. Recording consent, labor rules, retention, access, and accuracy vary.

## Model-assisted interview operations

| Use | Default | Conditions |
|---|---|---|
| Draft job-related questions from approved analysis | Permit as proposal | Assessment owner reviews; locked before cohort |
| Transcribe interview | Conditional | Notice/consent as applicable, accuracy and accessibility testing, correction path, restricted retention |
| Map cited statements to rubric | Permit as post-interview aid | Human observations already recorded; no new facts or score authority |
| Detect missing evidence or rubric inconsistency | Permit | Explain exact gap; human resolves |
| Summarize panel disagreement | Permit | Preserve each score/evidence; do not recommend winner |
| Score voice, face, accent, gaze, emotion, personality, honesty | **Reject** | Not a target; high construct, disability, privacy, and legal risk |
| Rank/shortlist/reject candidates | **Reject** | Consequential human decision |
| Generate hidden “potential” or retention prediction | **Reject** | Speculative profiling and feedback-loop risk |

The EU AI Act prohibits certain workplace emotion-inference uses (subject to narrow medical/safety exception) and classifies many recruitment/worker-management AI uses as high-risk. U.S. DOJ guidance warns that facial/voice analysis can screen out people with disabilities. These reinforce the product boundary; they are not the only reasons to reject such features.

## Human selection decision

The decision object must not be generated by the model.

```yaml
employment_decision:
  decision_id: dec_501
  decision_type: select_for_offer
  application_id: app_455
  requisition_id: req_204
  cohort_id: cohort_18
  input_versions:
    application: 44
    interview_kit: backend-p4-v6
    assessment_results: bundle_92
  decision_owner: hiring_manager_7
  owner_authority_ref: role_grant_811
  disposition: selected
  job_related_reason_codes: [meets_required_competencies]
  evidence_refs: [scorecard_bundle_92]
  model_summary_seen_after_independent_assessment: true
  conflicts_checked: true
  decided_at: 2026-08-31T11:10:00Z
  contest_route: candidate_review_process_v3
```

The system validates authority, required evidence, source versions, policy, conflict, and permitted transition. It does not validate whether the choice is legally correct; trained humans and governance processes own that judgment.

## Evidence-first review surface

Show, in this order:

1. job/requisition and rubric version;
2. candidate-provided and assessment evidence by criterion;
3. missing, conflicting, inaccessible, or disputed evidence;
4. independent panel observations and scores;
5. deviations and accommodations status without confidential details;
6. deterministic policy checks;
7. only then, optional model synthesis clearly labeled as non-authoritative;
8. actions: decide, request evidence, correct, recuse, escalate, or stop.

Do not use celebratory/negative colors, default-selected outcomes, pseudo-precision, or model “confidence” that nudges the reviewer.

## Offer and rejection communications

Offer terms, compensation, and employment status are human/policy owned. The agent may populate an approved template from exact fields after the decision and approvals are final.

| Communication | Agent capability | Required control |
|---|---|---|
| Interview invitation | Draft/stage | Exact candidate, requisition, time zone, accommodation notice, sender approval |
| Request for information | Draft/stage | Ask only lawful, necessary fields; accessible alternative |
| Offer draft | Populate from approved structured terms | Compensation/terms approval; template/version lock; e-sign status reconciliation |
| Rejection notice | Draft after human disposition | No invented reason; jurisdiction/policy template; delivery receipt distinction |
| Background pre-adverse/adverse notice | Administrative preparation only | Restricted process, counsel-approved template, required report/rights and timing, human decision |
| Withdrawal acknowledgement | Template | Candidate authorization and application-state verification |

Under the U.S. FCRA, where applicable, use of a consumer-reporting company for background checks creates authorization and pre-/post-adverse-action obligations. The agent must not collapse those steps or let a vendor result directly reject or terminate a person.

## Policy consistency without policy invention

The agent may compare the current package to a versioned rule profile and report:

- required step missing;
- observed step/order differs;
- approver not eligible;
- procedure/rubric version mismatch;
- cohort treatment differs;
- reason lacks evidence link;
- deadline/notice risk;
- novel condition not represented by policy.

For a novel condition, output `policy_gap` and route to HR/legal/policy owners. Do not infer a rule from similar historical cases: history may contain inconsistent or discriminatory practice.

## Fairness measurement seam

Voluntary demographic or protected-characteristic data used for lawful monitoring must remain outside the selector and ordinary review surface. A separate evaluation service joins de-identified/pseudonymized outcome records under controlled purpose and access. It reports aggregate and uncertainty-aware results to fairness/assessment owners, not individual candidate attributes to recruiters.

The four-fifths/80% selection-rate comparison in the U.S. Uniform Guidelines is a rule of thumb, not a fairness certificate or tolerance for discrimination. Small samples, large samples, statistical/practical significance, job-by-job analysis, intersectional harms, disability barriers, and validity all need their own interpretation by qualified owners.

## Recruiting failure matrix

| Failure | Signal | Containment | Recovery | Owner |
|---|---|---|---|---|
| Wrong candidate/application joined | Conflicting IDs or provenance | Freeze case and all effects | Steward resolves; append correction; review exposure | HR data steward/security |
| Resume parser omits evidence | Parser/model disagreement, candidate correction | Do not score absence | Alternate parse/human review; regression fixture | Recruiting ops |
| Rubric changed mid-cohort | Version mismatch | Pause affected decisions | Cohort impact review; consistent remedy | Assessment owner |
| Interviewer sees protected/medical data | Access audit/DLP signal | Revoke session, preserve evidence | Privacy incident and decision-impact review | Privacy/HR/security |
| Model anchors panel | Score timing shows synthesis first | Stop model view | Independent rescore or new panel per policy | HR/assessment |
| Inaccessible assessment | Candidate report, failure telemetry | Offer equivalent path; no penalty | Accommodation/accessibility process; vendor issue | Accessibility/HR |
| Webhook advances wrong stage | Source version/postcondition mismatch | Mark effect unknown; block next step | ATS read-back, correction, notify owner | Integration owner |
| Vendor score version changes silently | Manifest/version drift | Quarantine outputs | Re-evaluate, rerun if appropriate, vendor escalation | Vendor/assessment owner |
| Background result auto-rejects | Direct transition/effect trace | Kill integration path | Restore human process, notice/correction review | HR/legal |
| Candidate contests data | Correction/contest request | Freeze dependent decision where required | Authenticate, disclose, correct, re-review | Privacy/HR |

## Recruiting acceptance checklist

- [ ] Job analysis, essential functions, criteria, procedure, rubric, and policy are approved and versioned before use.
- [ ] Every opportunity-shaping tool is in the selection-procedure registry.
- [ ] Candidate, application, requisition, cohort, and evidence identities are stable.
- [ ] Demographic, accommodation/medical, background, and ordinary recruiting data are compartmented.
- [ ] Accessible notice and equivalent accommodation/manual paths are live.
- [ ] Interviewers score independently against job-related anchors before model synthesis.
- [ ] Human decision records bind exact evidence and versions.
- [ ] Vendor results never directly create an employment decision/effect.
- [ ] Contest/correction can pause, correct, and re-review the affected process.

## Sources and related guides

- [EEOC Uniform Guidelines Q&A](https://www.eeoc.gov/laws/guidance/questions-and-answers-clarify-and-provide-common-interpretation-uniform-guidelines)
- [OPM structured interviews](https://www.opm.gov/policy-data-oversight/assessment-and-selection/structured-interviews)
- [FTC background checks for employers](https://www.ftc.gov/business-guidance/resources/background-checks-what-employers-need-know)
- [Illinois Artificial Intelligence Video Interview Act](https://www.ilga.gov/Legislation/ILCS/Articles?ActID=4015&ChapterID=68&Print=)
- [EU Artificial Intelligence Act](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32024R1689)
- [Tool results, artifacts, and provenance](../../tools/tool-results-artifacts-and-provenance.md)


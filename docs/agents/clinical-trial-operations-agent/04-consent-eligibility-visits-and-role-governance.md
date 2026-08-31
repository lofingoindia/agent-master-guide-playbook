# Consent, Eligibility, Visits, and Role Governance

## Participant protection comes before throughput

The agent may make study operations easier to execute, but it must not make participant protection easier to bypass. Consent is an ongoing human process, eligibility is a qualified judgment grounded in source evidence, and visit coordination must never become clinical advice or dose direction.

## Separate the records people often conflate

| Record | What it represents | Never substitute |
|---|---|---|
| Participation consent process | Information exchange, comprehension opportunity, voluntariness, questions, decision, and documentation | A signed file alone |
| Consent document | Approved language/version presented and signed where required | Proof every consent-process duty occurred |
| Privacy authorization or lawful-basis record | Permission or legal basis for specific data processing/disclosure | Participation consent |
| Optional permission | Future research, biospecimen, genetics, recordings, recontact, data sharing | Core study participation |
| Assent | Developmentally appropriate agreement where applicable | Parent/LAR permission or adult consent |
| Capacity/LAR evidence | Basis and identity for representative decision-making | A generic relationship label |
| Withdrawal record | What the participant withdrew, when, and follow-up preferences | Automatic deletion of regulated historical records |

In the EU, participation informed consent and the GDPR lawful basis for processing are distinct legal questions. In the US, HIPAA authorization may be separate from FDA/HHS informed consent. The implementation needs jurisdiction- and institution-approved policies rather than a universal checkbox.

## Consent workflow

```mermaid
sequenceDiagram
    participant C as Coordinator/investigator
    participant A as Agent workflow
    participant P as Participant/LAR
    participant V as Validated consent system
    participant I as Investigator/delegate

    C->>A: Start consent task for study/site/participant alias
    A->>A: Resolve effective approved form and role
    A-->>C: Present approved materials and missing prerequisites
    C->>P: Conduct human consent discussion
    P->>C: Ask questions and decide voluntarily
    C->>V: Document process and signatures
    V-->>A: Versioned consent evidence
    A->>A: Validate completeness and reconcile status
    A-->>I: Exception or review task; no consent inference
```

### Consent state model

```text
NOT_STARTED -> MATERIALS_READY -> DISCUSSION_IN_PROGRESS -> DECIDED
DECIDED -> DOCUMENTED -> VERIFIED
DECIDED -> DECLINED
VERIFIED -> RECONSENT_REQUIRED -> DISCUSSION_IN_PROGRESS
VERIFIED -> WITHDRAWN
```

The system stores decision time, form version, language, person obtaining consent, participant/LAR relationship, signature evidence, process notes required by procedure, optional permissions, and verification outcome. `MATERIALS_READY` or `DOCUMENTED` is not `VERIFIED`.

### Consent stop conditions

- no currently effective approved document for the site and participant population;
- conflicting language/version or missing translation approval;
- unclear capacity, age-of-majority transition, or LAR authority;
- pressure, comprehension concern, unanswered question, or request for clinical explanation;
- signature/authentication anomaly;
- amendment or new safety information requiring qualified re-consent determination; or
- system outage without an approved contingency process.

## Eligibility support

Eligibility criteria are compiled into a versioned checklist by qualified protocol and clinical owners. The agent maps evidence; it does not invent interpretations or declare eligibility.

```yaml
eligibility_evidence_item:
  criterion_id: INC_04
  criterion_version: protocol_v3
  normalized_rule_ref: eligibility_ruleset_8
  candidate_state: EVIDENCE_FOUND
  source_refs: [fact_01J..., note_section_ref]
  evidence_time: 2026-08-30T14:00:00Z
  temporal_validity: within_14_days_of_randomization
  ambiguity: "lab reference range differs from central manual"
  model_confidence: 0.81
  reviewer_state: PENDING
  reviewer_id: null
```

Allowed candidate states are `EVIDENCE_FOUND`, `EVIDENCE_MISSING`, `CONFLICTING`, `OUT_OF_WINDOW`, and `NOT_MACHINE_INTERPRETABLE`. They are deliberately not `ELIGIBLE` and `INELIGIBLE`.

### Eligibility decision table

| Situation | Agent action | Required human action |
|---|---|---|
| Exact structured evidence within window | Link evidence and calculate deterministic rule result | Verify source and decide eligibility |
| Narrative evidence appears relevant | Extract candidate with exact source span | Interpret clinical meaning |
| Criterion uses judgment such as “clinically significant” | Mark human judgment required | Qualified clinician decides |
| Source values conflict | Present both, preserve versions, stop | Resolve source/correction and decide |
| Evidence missing | Create targeted collection task; never assume negative/normal | Obtain evidence or decline enrollment |
| Protocol wording ambiguous | Escalate through sponsor clarification process | Sponsor/medical/protocol authority resolves |
| Rule changes under amendment | Re-run evidence map under explicit transition policy | Investigator reviews applicability |

The eligibility decision record identifies the human decision-maker, time, protocol release, complete criteria set, exceptions/waivers if legally and procedurally permitted, and evidence reviewed. The model output remains attached as a non-authoritative aid.

## Visit and task operations

Visit windows, procedure schedules, and prerequisite dependencies should be compiled into a tested deterministic schedule model. The agent may explain conflicts and draft communications from approved templates.

```yaml
visit_instance:
  visit_id: visit_01J...
  participant_study_id: pt_2041
  protocol_release_id: pr_2026_0042_v3_eu_wave1
  planned_event: week_8
  anchor_event_ref: dose_1_actual
  earliest_at: 2026-11-02T00:00:00+05:30
  target_at: 2026-11-05T00:00:00+05:30
  latest_at: 2026-11-08T23:59:59+05:30
  required_tasks: [lab_panel_3, ecg_2, outcome_form_8]
  clinical_owner: investigator_role_ref
  status: SCHEDULED
```

The scheduler must distinguish target, allowed window, prohibited order, fasting/washout/preparation instructions, location constraints, local holidays, participant time zone, remote-procedure rules, and safety follow-up urgency. Only approved participant-facing content may be sent; clinical questions route to study clinicians.

### Missed or out-of-window visit

1. Record the actual event and source evidence; do not alter it to fit the schedule.
2. Calculate deviation candidacy deterministically from the effective protocol rules.
3. Escalate participant safety questions to the investigator immediately.
4. Create downstream data, monitoring, safety, and deviation tasks as applicable.
5. Preserve chronology and final qualified classification.

## Role and delegation governance

| Role | Typical permitted agent-assisted work | Authority retained outside model |
|---|---|---|
| Sponsor | Portfolio/study oversight, issue trends, amendment readiness | Sponsor responsibilities, risk acceptance, submissions, oversight |
| CRO | Perform contracted tasks and provide evidence | Sponsor retains oversight; CRO only within contract/delegation |
| Investigator | Review participant evidence and tasks | Medical care, consent, eligibility, enrollment, safety, dose decisions |
| Coordinator/delegate | Assemble records, schedule visits, document process | Only explicitly delegated qualified activities |
| Monitor | Risk signals, visit prep, query/deviation follow-up | Monitoring conclusions, escalation, source review within authorization |
| Data manager | Query drafting, reconciliation, review queues | Data-management decisions, locks, final query closure per procedure |
| Safety/PV reviewer | Case chronology and draft narrative | Seriousness/expectedness/causality, listedness, submission authorization |
| Pharmacist/IRT role | Supply reconciliation evidence | Dispensing, allocation access, dose preparation, unblinding authority |
| Vendor | Interface/service evidence | No authority beyond contracted, delegated, qualified scope |
| Auditor/inspector | Read-only evidence packages | Independent audit/inspection conclusions |

Delegation records are versioned and time-bounded. Revocation takes effect at the commit boundary even for a long-running task. A service account proves workload identity, not the end user's delegation or clinical qualification.

## Communication guardrails

- Clearly label drafts, source, protocol version, and reviewing role.
- Never generate individualized medical advice, treatment recommendations, or reassurance.
- Do not contact a participant from a new channel or for a new purpose without approved consent, template, and workflow.
- Do not expose another participant, site, treatment allocation, or aggregate result.
- Route urgency and symptom reports to the approved human safety path; do not triage them only through a model.
- Support language accessibility with approved translations and qualified interpretation; machine translation is a draft unless validated and approved for the use.

## Failure modes

| Failure | Containment |
|---|---|
| Wrong consent form after amendment | Site/population-specific release resolution and reconciliation |
| “No evidence” becomes “criterion absent” | Explicit missing state and non-inference invariant |
| Model interprets “clinically significant” | Mandatory clinician-review field and no decision API |
| Participant reply with symptoms stays in task queue | Independent urgent keyword/event path to on-call humans, monitored end to end |
| Delegated coordinator role expired mid-run | Fresh authorization at effect commit |
| Scheduling message implies dose change | Approved template library and clinical-content classifier/escalation |
| Withdrawal triggers destructive deletion | Granular withdrawal/permission state and retention policy review |

## Operational checklist

- [ ] Participation consent, privacy basis/authorization, and optional permissions are distinct.
- [ ] Effective form selection is deterministic and site/population scoped.
- [ ] Eligibility outputs are evidence candidates, never final decisions.
- [ ] Investigators can see source conflicts and missingness directly.
- [ ] Visit windows are compiled, versioned, and tested.
- [ ] Participant communications use approved content and channels.
- [ ] Delegation and role scope are rechecked before effects.
- [ ] An independent urgent safety path is continuously tested.

## Related guides

- [Clinical-Trial Operations Agent](README.md)
- [Study, protocol, site, participant identity, and amendments](03-study-protocol-site-participant-identity-and-amendments.md)
- [Safety, IRT, laboratories, blinding, and reconciliation](06-safety-irt-laboratories-blinding-and-reconciliation.md)
- [Security, privacy, validation, and inspection readiness](08-security-privacy-validation-and-inspection-readiness.md)


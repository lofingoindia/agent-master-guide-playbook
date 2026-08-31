# Mission, Boundaries, Workload Fit, and Authority

This guide decides whether the workload needs an agent and sets the authority ceiling before a model, framework, or integration is selected.

## Outcome contract

A healthcare care-coordination agent exists to reduce avoidable administrative gaps while keeping clinical and legal authority with accountable humans and deterministic services.

A valid outcome is:

> The right administrative task for the correctly bound patient is assigned to an accountable owner, completed or explicitly blocked, verified in the authoritative system, communicated through an authorized accessible channel, and closed with evidence—without the model making a clinical decision.

“The model answered” is not an outcome. “A message was sent” is not a closed loop. “The slot looked free” is not a booked appointment.

## Start with the deterministic alternative

Most volume should not reach a model. Use rules when inputs and actions are stable.

~~~mermaid
flowchart TD
    A[New coordination need] --> B{Verified patient and actor?}
    B -- No --> X[Identity or authority queue]
    B -- Yes --> C{Known workflow and complete structured fields?}
    C -- Yes --> D[Run deterministic state machine]
    C -- No --> E{Ambiguity is administrative and evidence-bounded?}
    E -- No --> H[Human or clinician queue]
    E -- Yes --> M[Bounded model proposal]
    M --> V{Schema, evidence, policy, safety, and budget valid?}
    V -- No --> H
    V -- Yes --> D
    D --> F{Consequential effect?}
    F -- No --> G[Persist and continue]
    F -- Yes --> P[Require D-tier authorization]
    P --> R[Dispatch, verify, and reconcile]
~~~

### Deterministic baseline

Implement before the first model call:

- required-field and terminology validation;
- identity, tenant, patient-binding, and representative checks;
- directory lookup and fixed routing tables;
- task creation, deadlines, reminders, and escalation timers;
- status normalization for known vendor responses;
- duplicate detection and effect idempotency;
- approved message templates with field-level disclosure control;
- safety rules supplied by accountable clinical governance;
- stop states for missing authority, conflicting evidence, or unknown effects.

Add a model only where real cases show unresolved natural-language ambiguity, variable documentation, or evidence synthesis that cannot be handled reliably with these controls.

## Agent-fit scorecard

Score the proposed workload before Stage 1.

| Question | Strong fit | Weak fit or rejection |
|---|---|---|
| Is there repeated administrative ambiguity? | Incomplete referral narrative must be mapped to an approved missing-information category | Structured form validation only |
| Can the evidence set be bounded? | A few versioned EHR, payer, and directory records | Open-web clinical research |
| Is the output closed-set? | Propose request-missing-document or route-to-queue | Free-form “decide what care is needed” |
| Can authority be checked before effects? | Booking has verified patient confirmation or narrow preauthorization | Model infers consent |
| Can success be verified externally? | Appointment ID, payer receipt, task acknowledgment | “Patient probably understood” |
| Can uncertain cases safely abstain? | Queue to a staffed coordinator or clinician | No human route |
| Is the work recoverable? | Durable case and reconciliation exist | Chat-only flow |
| Is the intended use administrative? | Track referral closure | Recommend diagnosis or treatment |

Reject the agent design if clinical recommendation is the core value, authoritative identity cannot be established, the only available integration is uncontrolled screen automation for consequential work, or no accountable human escalation path exists.

## When not to use an agent

| Workload condition | Do this instead | Why the model path is rejected |
|---|---|---|
| Inputs are structured and the rule is stable | Form validation, decision table, state machine, scheduled job or database constraint | A model adds variability, PHI exposure, cost and evaluation burden without resolving ambiguity |
| The action is a clinical judgment—diagnosis, treatment, medication, urgency, capacity or emergency disposition | Route directly to the accountable clinician or emergency process defined by clinical governance | Administrative evidence grounding cannot transfer professional authority |
| Patient, tenant, representative or destination identity is ambiguous | Stop before sensitive retrieval; use MPI/identity steward, proxy-verification, directory or privacy workflow | A plausible match or relationship is not an authorized binding |
| A deterministic safety trigger fires or a person asks for urgent clinical help | Set the safety hold and invoke the staffed primary/backup clinical route | Model classification must not delay or replace the approved escalation |
| Success cannot be observed and reconciled | Redesign the integration/manual receipt process before automation | A fluent response cannot prove booking, submission, delivery, acceptance or closure |
| Only unattended browser/computer control can commit a consequential portal action | Use a staffed operator, staged draft, or acquire a supported API/contract | UI timeout and page state create unbounded wrong-target and unknown-effect risk |
| No staffed owner exists for exceptions, handoffs, safety and unknown outcomes | Keep the workflow manual or do not launch | Automation would create orphaned work rather than coordination |
| The requested evidence is open-web or unbounded chart exploration | Curate a versioned knowledge artifact or require a human/clinical research process outside the patient run | Purpose, provenance, freshness, injection resistance and minimum necessary cannot be bounded |
| The organization cannot contractually control PHI processing, retention, training use, region or subprocessors | Use an approved on-premises/contracted deterministic service or do not send PHI | Technical prompting cannot repair an unauthorized processor/data flow |
| The main objective is fewer staff touches regardless of missed cases or inequitable access | Redesign the service with safety, accessibility and outcome owners | Throughput alone is not a valid healthcare outcome |

“Use a human” is not a complete fallback. The alternative must name an intake route, covered role/queue, hours and backup, minimum evidence, due/acknowledgment clock, source-system update, patient-accessible channel and closure proof. If that deterministic/human path cannot be operated safely, an agent cannot make it safe.

## Workload ownership matrix

| Work item | Model role | Deterministic system role | Accountable human or source-system role |
|---|---|---|---|
| Patient match | Explain candidate evidence; abstain | Call MPI/match service; enforce thresholds and stop states | Identity steward resolves ambiguity and merge/split |
| Proxy request | Identify needed authority evidence | Evaluate versioned grant and policy | Patient, authorized representative process, privacy/legal function |
| Referral intake | Extract administrative gaps with source refs | Validate fields and create tasks | Clinician owns service, reason, priority, and order |
| Appointment options | Rank by explicit logistical preferences | Query fresh slots and validate constraints | Patient/representative chooses; scheduler commits under policy |
| Prior authorization | Assemble source-backed administrative packet | Run required-field, profile, and submission checks | Clinician attests clinical content; payer decides |
| Care-plan task | Explain status or missing dependency | Project task from authoritative plan | Clinician/care team owns plan and clinical completion |
| Medication discrepancy | State that sources conflict | Preserve versions and create reconciliation task | Licensed clinician/pharmacist reconciles |
| Concerning patient message | Trigger conservative safety escalation | Freeze routine effects and route | Approved clinical service assesses and decides disposition |
| Outreach | Draft minimal approved content | Select allowed template/channel and record delivery | Patient preference/policy and care team determine what may be sent |
| Handoff | Summarize evidence and requested action | Require acknowledgment and deadline | Named receiver accepts responsibility |

## Clinical authority boundary

### Always prohibited

The agent must not:

- diagnose, rule out, or estimate the likelihood of a condition for care decisions;
- choose or compare treatments for a patient;
- prescribe, recommend a dose, change a dose, stop a medication, or perform clinical medication reconciliation;
- create, amend, sign, or cancel a clinical order;
- invent clinical reason, urgency, medical necessity, diagnosis code, or procedure code;
- determine capacity, guardianship, personal-representative status, or consent validity;
- make emergency-triage disposition or tell a patient where, when, or whether to seek emergency care;
- suppress a clinician review because a model predicts low risk;
- treat absence of data as a reassuring clinical fact.

### Allowed evidence handling

Medication, diagnosis, allergy, result, note, referral, and plan information may be used only when:

1. the source is authorized for the purpose;
2. the exact record and version are retained;
3. the model distinguishes quoted/extracted evidence from inference;
4. contradictions and staleness remain visible;
5. the output is administrative;
6. a clinician owns any clinical interpretation or change.

Example:

- Allowed: “The referral names medication list version EHR/MedicationRequest/123/_history/4, while the payer form cites version 3. Route discrepancy to pharmacist review.”
- Prohibited: “Increase the dose” or “version 4 is clinically correct.”

## Administrative versus clinical decisions

| Decision | Administrative coordination | Clinical judgment |
|---|---|---|
| Referral completeness | Required document is absent | Whether the requested service is appropriate |
| Timing | An authorized order says “urgent”; map to the approved queue | Decide that symptoms are urgent |
| Scheduling | Offer eligible slots under known constraints | Decide whether waiting is clinically safe |
| Authorization | Required payer field lacks source evidence | Decide medical necessity |
| Care task | Task is overdue or unacknowledged | Decide whether the care plan should change |
| Medication data | Two authoritative sources disagree | Decide which regimen the patient should follow |
| Safety message | Concerning phrase triggers a clinician queue | Diagnose, triage, or select emergency disposition |

When a decision crosses the right-hand column, the model stops. Rewording the result as a “suggestion” does not make it administrative.

## Authority model

Use the repository's D0–D4 model:

| Tier | Permitted healthcare use | Required control |
|---|---|---|
| D0 | Synthetic evaluation and isolated drafting | No PHI or external effects |
| D1 | Purpose-limited reads | Current patient binding, actor authorization, field filter, audit |
| D2 | Draft or reversible staging | All D1 controls plus expiration and rollback |
| D3 | Booking, cancellation, submission, or PHI-bearing send | Exact approval or narrow deterministic preauthorization, fresh preconditions, receipt, reconciliation |
| D4 | Change consent, proxy scope, clinical ownership, access policy, or tool authority | Proposal-only; authorized human/system performs change |

“Human in the loop” is not sufficient unless the approval binds:

- the exact patient binding and version;
- actor and relationship;
- action and normalized parameters;
- destination or recipient;
- data to disclose;
- clinical source versions;
- expiration time;
- behavior version;
- effect ID.

## Clinician and care-team ownership

Every case must have an accountable owner, not merely a queue name. Represent:

- organization and service;
- practitioner or role;
- effective period;
- coverage/on-call substitution;
- handoff acknowledgment;
- escalation route when ownership is absent;
- source of clinical intent;
- source of clinical priority.

A model may propose a directory match but cannot claim that the match accepted clinical responsibility. Responsibility begins only when the configured source system or person records acceptance.

## Safety-stop contract

Safety escalation is an interrupt, not another optional tool call.

When an organization-approved deterministic trigger fires, the model expresses material uncertainty, or concerning evidence cannot be safely classified:

1. persist the evidence reference and trigger;
2. place the case in safety-hold;
3. block routine external effects except approved safety communications;
4. route to the configured clinical service;
5. start the locally approved acknowledgment timer;
6. use a backup route if the primary queue is unavailable;
7. record acknowledgment and accountable receiver;
8. resume only through an authorized release transition.

The agent records disposition as “not set by agent.” The clinical service owns assessment and disposition.

## Honest user communication

The interface must say:

- it supports administrative coordination;
- it is not a clinician or emergency service;
- messages may be reviewed by staff;
- what channel will be used and what minimal content may appear;
- when an action is pending, booked, delivered, acknowledged, or blocked;
- how to reach a human and request an accessible or language-supported alternative.

Do not use conversational fluency to imply clinical expertise, immediate monitoring, guaranteed response, or a completed action.

## Stage 0 exit evidence

Do not enter implementation until all are true:

- [ ] The intended use and excluded clinical functions are signed off.
- [ ] One narrow workflow, population, organization, jurisdiction, and channel are named.
- [ ] The deterministic baseline is diagrammed and measured.
- [ ] Real ambiguity justifies a model.
- [ ] Identity, proxy, privacy, clinical, safety, and operational owners are named.
- [ ] Every D3 effect and every D4 change is enumerated.
- [ ] Human and clinician queues exist with coverage and fallback.
- [ ] Source systems and authoritative fields are identified.
- [ ] Success and hard failure conditions are testable.
- [ ] Regulatory classification and legal/safety review triggers are documented.

## Related guides

- Next: [Reference Architecture, Runtime, and Integration Decisions](02-reference-architecture-runtime-and-integration-decisions.md)
- Home: [Healthcare Care Coordination Agent](README.md)
- Evidence: [Healthcare Care Coordination Agent Blueprint](../../research/packets/healthcare-care-coordination-agent-blueprint.md)
- Canonical: [Execution Boundaries](../../runtime/execution-boundaries.md)

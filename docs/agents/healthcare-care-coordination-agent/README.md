# Healthcare Care Coordination Agent

**Category:** 41 — Healthcare clinical administration and care coordination  
**Maturity:** Production-oriented blueprint; deployment validation required  
**Last researched:** 2026-08-31  
**Evidence packet:** [Healthcare Care Coordination Agent Blueprint](../../research/packets/healthcare-care-coordination-agent-blueprint.md)

> This guide describes software architecture, controls, and operating practices. It is not medical or legal advice. The deploying organization remains responsible for clinical governance, professional judgment, privacy and accessibility obligations, regulatory classification, and validation in every jurisdiction and care setting.

## Mission

Build a durable administrative coordinator that closes gaps around scheduling, referrals, prior authorization, care-plan tasks, outreach, and handoffs while preserving the authority of patients, representatives, clinicians, care teams, and source systems.

The production shape is deliberately conservative:

- deterministic workflows handle known rules and ordinary state transitions;
- one bounded model worker interprets ambiguous administrative evidence and proposes a closed-set next action;
- a policy service binds patient, actor, purpose, consent, proxy scope, data, and action;
- accountable clinicians own clinical intent, priority, orders, diagnosis, treatment, medication decisions, and safety disposition;
- typed adapters, an effect ledger, and reconciliation control external side effects;
- durable state, audit, safety telemetry, and behavior versions survive model and process restarts.

## Owned scope and excluded authority

| This category owns | This category does not own |
|---|---|
| Patient-record binding and identity escalation | Automatic chart merge or ambiguous-patient selection |
| Consent and personal-representative evidence | A universal legal interpretation of consent or capacity |
| Scheduling and referral administration | Clinical urgency, diagnosis, treatment, or disposition |
| Prior-authorization evidence collection and status | Clinical medical-necessity judgment, coding invention, or claims finance |
| Care-plan task projection and follow-up | Creating or changing the clinical care plan |
| PHI-aware communication and outreach | Unrestricted messaging or disclosure |
| Care-team assignment, coverage, and handoff | Replacing accountable clinical ownership |
| Conservative safety escalation | Emergency triage, clinical advice, or deciding where the patient should go |
| Medication and diagnosis records as evidence | Prescribing, dose changes, medication reconciliation, or diagnostic inference |
| Effect tracking, recovery, reconciliation, and audit | Treating the model transcript as the clinical or legal record |

## Category separation

| Adjacent category | Its center of gravity | Why this guide stays separate |
|---|---|---|
| Customer support | Service resolution and account policy | It lacks clinical identity, consent, PHI, care-team ownership, and safety controls |
| Document intelligence | Extraction, classification, and provenance | It can supply evidence but does not own a patient coordination case or its effects |
| Clinical-trial operations | Protocol-governed regulated studies | This guide covers ordinary clinical administration, not study eligibility, randomization, or protocol execution |
| Insurance claims and finance | Adjudication, payment, denial, and accounting | This guide tracks authorization needed for care but does not own financial decisions or revenue workflows |
| Clinical decision support | Patient-specific clinical recommendation | Explicitly outside the authority ceiling |

## The central design rule

**The model proposes; deterministic controls and accountable people authorize.**

The application owns:

- authenticated actor and patient binding;
- purpose, role, consent, representative authority, and data-policy decisions;
- clinical source references and their versions;
- case state, leases, deadlines, ownership, approvals, and safety holds;
- tool registration, allowed parameters, effect dispatch, receipts, and reconciliation;
- PHI filtering, audit, traces, evaluations, releases, and incident response.

The model may:

- identify missing administrative information;
- summarize conflicting status with source references;
- map an ambiguous request to an approved workflow;
- propose one allowed next action or ask for missing evidence;
- draft a minimal communication for approved review or delivery;
- conservatively request a safety or clinical handoff when evidence is concerning or insufficient.

It may never manufacture clinical intent, decide whether a patient has a condition, recommend treatment, change medication, assign clinical priority, or select an emergency disposition.

## Reference architecture

~~~mermaid
flowchart TB
    subgraph Channels
        PAT[Patient]
        REP[Representative or caregiver]
        STF[Workforce]
    end

    subgraph Trust_and_policy[Trust and policy boundary]
        ID[Identity and patient binding]
        PDP[Purpose, consent, proxy, and RBAC or ABAC policy]
        SAFE[Clinical-safety rules and escalation routes]
    end

    subgraph Coordination_runtime[Coordination runtime]
        API[Case API]
        WF[Durable workflow]
        CTX[Context assembler]
        MOD[Bounded model worker]
        LED[Effect ledger and outbox]
        REC[Reconciler]
        HUM[Human and clinician queues]
    end

    subgraph Domain_systems[Domain systems of record]
        EHR[EHR and FHIR]
        MPI[HIE, MPI, and directory]
        SCH[Scheduling]
        PAY[Payer and authorization]
        COM[Approved communication channels]
    end

    PAT --> ID
    REP --> ID
    STF --> ID
    ID --> PDP
    PDP --> API
    SAFE --> WF
    API --> WF
    WF --> CTX
    CTX --> MOD
    MOD -->|typed proposal only| WF
    WF --> HUM
    WF --> LED
    LED --> EHR
    LED --> MPI
    LED --> SCH
    LED --> PAY
    LED --> COM
    EHR --> REC
    MPI --> REC
    SCH --> REC
    PAY --> REC
    COM --> REC
    REC --> WF
~~~

The EHR, HIE/MPI, scheduling, payer, and communication services stay authoritative for their own records. The local coordination ledger is authoritative only for coordination progress, decisions, effects, receipts, and reconciliation.

## Control declaration

Every deployment should publish a versioned declaration like this:

~~~yaml
agent_control_declaration:
  category: healthcare-care-coordination
  intended_use: administrative coordination only
  prohibited_uses:
    - autonomous diagnosis
    - treatment selection
    - prescribing or dose change
    - clinical urgency assignment
    - emergency triage disposition
    - ambiguous patient merge
  model_role: bounded administrative proposer
  maximum_authority: D3-with-exact-approval-or-narrow-preauthorization
  D4_policy_or_authority_change: proposal-only
  identity_authority: organization-mpi-and-identity-stewards
  clinical_authority: licensed-clinicians-and-approved-care-teams
  state_authority: durable-workflow-ledger
  external_truth: domain-systems-of-record
  safety_owner: organization-defined
  privacy_profile: versioned-and-jurisdiction-specific
  behavior_version: immutable-release-manifest
~~~

## Authority tiers

| Tier | Meaning here | Examples |
|---|---|---|
| D0 | Isolated reasoning; no data access or effects | Classify a synthetic test case |
| D1 | Read with purpose- and field-limited access | Read referral status or appointment availability |
| D2 | Reversible or staged change | Create a draft, provisional hold, or internal work item |
| D3 | Consequential commit with exact approval or narrow deterministic preauthorization | Book/cancel an appointment, submit an authorization, send PHI-bearing outreach |
| D4 | Change who may act or what policy permits | Alter proxy rights, clinical ownership, consent policy, or access rules |

D4 remains proposal-only. A model cannot grant itself, a user, or another service more authority.

## System records and invariants

The minimum durable records are:

- **coordination case:** workflow, patient binding, owner, state, deadlines, safety holds, and behavior version;
- **patient binding:** namespace-qualified subject, match assurance, verification source, version, and invalidation status;
- **authority grant:** actor relationship, source, purpose, actions, data classes, restrictions, effective period, and revocation status;
- **clinical evidence reference:** system, resource identifier, version, timestamp, provenance, and exact supported assertion;
- **coordination task:** owner, dependency, deadline, status, completion evidence, and escalation route;
- **effect:** semantic ID, normalized parameters, approval, attempt, receipt, postcondition, and unknown/reconciled state;
- **communication delivery:** intended audience, minimal payload class, channel, recipient address version, attempt, receipt, and acknowledgment;
- **handoff:** sender, receiver, reason, evidence, requested action, due time, acknowledgment, and closure.

Non-negotiable invariants:

1. No effect or disclosure without a current patient binding and authority decision.
2. No model-authored clinical fact is promoted without a versioned source.
3. No clinical intent, priority, diagnosis, or medication decision is created by the model.
4. No appointment, referral, authorization, message, or handoff is called complete without the required receipt or verified postcondition.
5. No unknown effect is blindly retried.
6. No routine automation proceeds while a safety hold is active.
7. No session summary or provider memory is treated as durable truth.
8. No case closes while required tasks, contradictions, effects, or handoffs remain unresolved.

## Learning path

Read the guides in order for a zero-to-production path:

1. [Mission, Boundaries, Workload Fit, and Authority](01-mission-boundaries-workload-fit-and-authority.md) — decide whether an agent is justified and set the clinical ceiling.
2. [Reference Architecture, Runtime, and Integration Decisions](02-reference-architecture-runtime-and-integration-decisions.md) — select the simplest runtime and qualify integrations.
3. [Patient Identity, Consent, Proxy, and Care-Team Authority](03-patient-identity-consent-proxy-and-care-team-authority.md) — establish who, which patient, what purpose, and whose authority.
4. [Scheduling, Referrals, Prior Authorization, Tasks, and Handoffs](04-scheduling-referrals-prior-authorization-tasks-and-handoffs.md) — implement closed-loop workflows.
5. [State, Events, Context, Memory, and Planning](05-state-events-context-memory-and-planning.md) — make continuity typed, durable, and bounded.
6. [Tools, Effects, Idempotency, Reconciliation, and Recovery](06-tools-effects-idempotency-reconciliation-and-recovery.md) — make consequential work recoverable.
7. [Security, Privacy, Accessibility, and Clinical Safety](07-security-privacy-accessibility-and-clinical-safety.md) — control PHI and harm pathways.
8. [Observability, Evaluation, Failure Injection, and Incidents](08-observability-evaluation-failure-injection-and-incidents.md) — prove behavior under normal and adversarial conditions.
9. [Deployment, Scale, Capacity, Cost, and Behavior Evolution](09-deployment-scale-capacity-cost-and-behavior-evolution.md) — operate and change the system safely.
10. [Stage 0–6 Delivery Gates, Schemas, and Checklists](10-stage-0-6-delivery-gates-schemas-and-checklists.md) — ship progressively with explicit evidence.

## Reader paths

| Reader | Start with | Then |
|---|---|---|
| Clinical or safety leader | Guides 1, 3, 7 | Guides 8 and 10 |
| Privacy, legal, or security reviewer | Guides 1, 3, 7 | Guides 6, 8, and the research packet |
| Platform engineer | Guides 2, 5, 6 | Guides 8, 9, and 10 |
| Product or operations owner | Guides 1 and 4 | Guides 8, 9, and 10 |
| Integration engineer | Guides 2, 3, 4, and 6 | Guide 9 |

## Smallest useful release

The preferred MVP handles one organization, one administrative workflow, one patient population, one approved channel, and a few typed adapters:

- intake an already-authenticated referral-coordination request;
- bind it to an existing verified patient record;
- validate proxy/consent/purpose policy when another person acts;
- read the clinician-authored referral and directory status;
- identify missing administrative evidence;
- create tasks and one draft or preauthorized communication;
- track acceptance, scheduling, attendance exception, and handoff;
- freeze and route concerning content to an approved clinical queue;
- reconcile every external write.

Do not begin with multi-agent orchestration, autonomous browsing, generalized memory, all-payer support, cross-jurisdiction policy inference, or clinical recommendations.

## Definition of done

A production candidate is not “done” because it can complete a happy-path demo. It must show:

- approved intended-use, non-goals, hazard log, safety case, and named accountable owners;
- tested patient binding, proxy/consent decisions, PHI minimization, and tenant isolation;
- source/version lineage for every clinical assertion used;
- typed tools with D-tier, schema, timeouts, retry semantics, and postconditions;
- durable cases, tasks, effects, handoffs, unknown outcomes, and reconciliation;
- accessibility, language, alternate-channel, and human-assistance paths;
- stratified offline evaluations and injected-failure recovery;
- PHI-safe traces, SLOs, alert routes, runbooks, kill switches, and incident practice;
- capacity, queue, dependency, region, recovery, and cost evidence;
- immutable behavior manifests, canary/rollback controls, and governed refresh triggers;
- completion of every applicable [Stage 0–6 exit gate](10-stage-0-6-delivery-gates-schemas-and-checklists.md).

## Canonical companion guides

- [Execution Boundaries](../../runtime/execution-boundaries.md)
- [Agent State and Event Contracts](../../runtime/agent-state-and-event-contracts.md)
- [Durable Execution](../../runtime/durable-execution.md)
- [Run Controls](../../runtime/run-controls.md)
- [Tool Contracts](../../tools/tool-contracts.md)
- [Tool Results, Artifacts, and Provenance](../../tools/tool-results-artifacts-and-provenance.md)
- [Idempotency and Side Effects](../../reliability/idempotency-and-side-effects.md)
- [Compaction and Continuity](../../context-memory/compaction-and-continuity.md)
- [Observability and Tracing](../../evaluation/observability-and-tracing.md)
- [Deployment, Release, and Incident Response](../../operations/deployment-release-and-incident-response.md)
- [Agent Threat Model](../../security/agent-threat-model.md)
- [Prompt Injection and Untrusted Data](../../security/prompt-injection-and-untrusted-data.md)

## Evidence and refresh

The [research packet](../../research/packets/healthcare-care-coordination-agent-blueprint.md) records source versions, contradictions, limitations, and refresh triggers. Revalidate it whenever law, intended use, population, workflow, model, processor, channel, adapter, FHIR profile, payer rule, safety event, or hard-gate evaluation result changes.


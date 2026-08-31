# Operating Model and Reference Architecture

> **Status:** Research-backed production blueprint  
> **Last researched:** 2026-08-31  
> **Scope:** Defensive investigation operating model and system boundaries; evidence schemas, effect controls, and eval implementation are covered in the focused guides.  
> **Section index:** [Security investigation and triage agent](README.md)

The product is an investigation system with a probabilistic reasoning component—not a chatbot with broad security credentials. Its output is a versioned case assessment that an analyst can inspect, correct, approve, or reject.

## Purpose and success criteria

The system should reduce the time and cognitive load required to answer:

1. What happened, to which assets and identities, and during what interval?
2. Which evidence supports or contradicts each explanation?
3. What important evidence is missing or inaccessible?
4. What is the potential and observed impact?
5. What safe next query, escalation, containment, or recovery decision should an accountable person consider?

Success is not the number of alerts closed or tokens generated. Success combines:

- high recall for material incidents;
- tolerable and measured analyst escalation load;
- evidence-complete, reproducible conclusions;
- calibrated uncertainty and appropriate abstention;
- bounded data access and zero unauthorized effects;
- faster time to a defensible decision;
- no degradation of evidence integrity or incident recoverability.

FIRST separates event analysis, incident triage, detailed analysis, coordination, and response-related services. Preserve those distinctions in the product. A model that classifies an alert has not necessarily investigated an incident, and an investigated incident has not automatically authorized a response.

## Explicit non-goals

The blueprint does not create:

- an autonomous penetration-testing, exploitation, persistence, credential-access, lateral-movement, or evasion system;
- a replacement for accountable incident commanders, legal counsel, privacy teams, system owners, or digital-forensics specialists;
- a general shell with production-network access;
- a threat-intelligence oracle that treats matches as attribution or truth;
- an evidence-admissibility guarantee across jurisdictions;
- a mechanism that closes alerts solely to improve queue metrics;
- a self-modifying detection or response system that promotes its own conclusions.

Detection engineering, threat hunting, malware reverse engineering, and recovery may receive evidence or drafts from the agent, but remain separate governed capabilities.

## Operating roles and accountability

| Role | Owns | Must not delegate to the model |
|---|---|---|
| SOC analyst | Alert disposition, evidence review, escalation | Accountability for case verdict |
| Incident commander | Scope, objectives, coordination, response sequencing | Business-impact and crisis decisions |
| Asset or service owner | Operational impact, safe maintenance windows, rollback validation | Acceptance of production disruption |
| Detection engineer | Rule quality, coverage, tuning, suppression review | Promotion of model-generated detection content |
| Forensics lead or evidence custodian | Acquisition method, custody, preservation, legal-hold coordination | Authenticity and admissibility judgment |
| Identity and access owner | Roles, service identities, credential broker, break-glass | Privilege grants and tenant-boundary exceptions |
| Privacy or legal owner | Purpose, minimization, retention, disclosure, legal holds | Jurisdiction-specific interpretations |
| Agent platform owner | Runtime, prompts, models, tools, policy integration, SLOs | Incident truth or response approval |
| Policy decision point | Deterministic allow/deny/require-approval outcome | Natural-language interpretation of authority |

The model is not a role in this table. It is an untrusted proposal generator operating within controls owned by people and conventional software.

## End-to-end lifecycle

~~~mermaid
sequenceDiagram
    autonumber
    participant S as Alert source
    participant I as Intake
    participant C as Case service
    participant A as Investigation agent
    participant Q as Read-only query broker
    participant H as Analyst / incident lead
    participant P as Policy and approval
    participant E as Response executor

    S->>I: Alert plus source identity and delivery ID
    I->>I: Validate, deduplicate, normalize, classify sensitivity
    I->>C: Open or correlate case; preserve raw evidence reference
    C->>A: Authorized case snapshot and investigation objective
    loop Bounded evidence loop
        A->>Q: Typed query proposal
        Q->>Q: Authorize tenant, fields, time, cost, and purpose
        Q-->>A: Provenanced result or explicit denial/gap
        A->>C: Append claim, evidence link, hypothesis update
    end
    A->>H: Verdict proposal, alternatives, gaps, next actions
    H->>C: Confirm, correct, escalate, or request more evidence
    opt Consequential action requested
        A->>P: Exact action proposal
        H->>P: Effect-bound approval
        P->>E: Short-lived capability and idempotency key
        E-->>C: Attempt and observed outcome
    end
~~~

### Lifecycle invariants

- Intake acknowledgement and deduplication occur before model invocation.
- The raw alert and raw enrichment responses remain addressable after normalization.
- Every model run receives a fixed objective, case version, access scope, and budget.
- Each query result records source, query, execution time, coverage, truncation, and errors.
- New evidence may reopen a disposition; closure never makes the record immutable to correction.
- Response is a separate transaction with fresh authorization and current target state.

## Logical component boundaries

### 1. Intake and correlation

Responsibilities:

- authenticate the producer and validate the event envelope;
- apply schema, size, decompression, and type limits;
- create a stable event identity and deduplicate deliveries;
- preserve raw bytes and normalization diagnostics;
- correlate only through deterministic rules or explicitly labeled probabilistic suggestions;
- prioritize admission using source priority and organizational impact, not model prose.

The intake service must operate when the model provider is unavailable. Alert durability cannot depend on an inference call.

### 2. Case and evidence services

The case service owns workflow state: assignee, status, severity proposal, hypotheses, claims, requested work, approvals, and action records. The evidence service owns immutable objects, source metadata, integrity verification, custody, and access policy.

Use separate records because interpretations change while acquired evidence should not. Corrections append a superseding claim; they do not rewrite history.

### 3. Context compiler

The context compiler creates a least-data case view:

- verified control instructions;
- investigation objective and stopping conditions;
- current case claims and open hypotheses;
- authorized evidence excerpts with stable citations;
- asset, identity, and threat context with freshness;
- tool catalog filtered to the current principal and authority tier;
- explicit gaps, denials, truncation, clock uncertainty, and budget remaining.

It must never concatenate arbitrary source text into the instruction lane.

### 4. Investigation agent

The agent proposes:

- evidence-grounded claims;
- competing hypotheses;
- the next discriminating query;
- a disposition and confidence;
- a response recommendation or a reason to abstain.

It does not hold production credentials, directly query arbitrary endpoints, mutate policy, or decide whether an effect is authorized.

### 5. Read-only query broker

The broker converts a typed request into a source-specific query after enforcing:

- caller, tenant, purpose, resource, fields, and time range;
- maximum rows, bytes, runtime, fan-out, concurrency, and cost;
- query-template or abstract-syntax validation;
- redaction and sensitive-field policy;
- source health, freshness, and retention coverage;
- result hashing, provenance, pagination, and truncation metadata.

Treat read access as consequential. Broad searches can expose secrets, employee activity, customer data, or another tenant.

### 6. Policy, approval, and response

The policy service evaluates canonical identifiers and current facts. The approval service records who approved which exact effect, under which policy and target version, until when. The response executor receives only a short-lived, action-specific capability.

The response executor must not accept free-form commands. It implements small typed operations such as a time-bounded endpoint isolation request or a case annotation, with preconditions, postconditions, idempotency, and rollback metadata.

### 7. Audit and observability

Application-owned audit events are the durable system of record. Traces and metrics are derived operational views. Keep evidence, case history, effect history, and service telemetry distinct so that privacy minimization or trace sampling cannot erase the investigation record.

## Four deployable architectures

### A. Advisory triage

~~~mermaid
flowchart LR
    B["Curated alert bundle"] --> A["Read-only agent"]
    A --> D["Draft disposition + cited evidence"]
    D --> H["Analyst decides and records"]
~~~

**Authority:** no external reads beyond the supplied bundle; no writes.  
**Strength:** smallest attack surface and easiest evaluation.  
**Weakness:** the bundle may omit the decisive context, leading to superficial conclusions.  
**Use when:** integrations, data governance, or eval maturity are early.  
**Exit gate:** the system demonstrates that requested follow-up evidence is useful and safe enough to justify a query broker.

### B. Supervised investigation

~~~mermaid
flowchart LR
    C["Case objective"] --> A["Investigator"]
    A --> Q["Read-only broker"]
    Q --> S["Authorized sources"]
    S --> Q --> A
    A --> H["Evidence-backed draft"]
    H --> C
~~~

**Authority:** bounded read-only pivots and draft case updates.  
**Strength:** reduces console switching and makes missing evidence explicit.  
**Weakness:** fan-out, privacy exposure, source outages, and plausible but unnecessary queries.  
**Use when:** source ACLs, asset/identity ownership, and query audit are dependable.  
**Exit gate:** stable value in shadow mode, low cross-scope denial rate, and no unsafe query escalation.

### C. SOAR-integrated workflow

**Authority:** typed case writes, task creation, notifications, or prepared CACAO-like steps; response still requires policy and approval.  
**Strength:** preserves workflow continuity, ownership, and repeatability.  
**Weakness:** a polished case note can be mistaken for verified truth; duplicate deliveries can create duplicate tasks or notifications.  
**Use when:** the organization already has tested playbooks, roles, action identifiers, and reconciliation.  
**Exit gate:** idempotent writes, separation of proposed and confirmed findings, and safe upgrade/replay tests.

### D. Constrained response

**Authority:** a tiny allowlist of reversible, time-bounded actions under exact policy and approval conditions.  
**Strength:** reduces containment delay for proven scenarios.  
**Weakness:** false positives can interrupt critical services; technically correct containment can destroy volatile evidence or expose attacker awareness.  
**Use when:** the same action is already routinely approved, its blast radius is bounded, rollback is tested, and evidence preservation is explicit.  
**Exit gate:** scenario-specific false-action budgets, dual control where required, canary rollout, observed-state reconciliation, and a tested kill switch.

## Architecture selection matrix

| Condition | Advisory | Supervised | SOAR-integrated | Constrained response |
|---|---:|---:|---:|---:|
| Asset ownership is incomplete | Suitable | Limited | No | No |
| Tenant and regional policy is enforceable in APIs | Suitable | Required | Required | Required |
| Evidence sources have stable identifiers and coverage metadata | Helpful | Required | Required | Required |
| Existing playbooks and escalation ownership are mature | Helpful | Helpful | Required | Required |
| Mutating APIs support idempotency or reconciliation | Not applicable | Not applicable | Required for writes | Required |
| Rollback and business-impact validation are tested | Not applicable | Not applicable | Helpful | Required |
| High-quality outcome and security eval corpus exists | Required | Required | Required | Required with stricter gates |

## State ownership and consistency

| State | Authoritative owner | Consistency expectation |
|---|---|---|
| Raw source event | Evidence store plus originating system | Immutable object and integrity check |
| Normalized event | Intake pipeline | Versioned mapping; raw reference retained |
| Asset and identity facts | CMDB, identity provider, cloud control plane | Read at a recorded version or timestamp |
| Threat intelligence | CTI store/provider | Source, retrieval time, markings, and confidence retained |
| Claim or hypothesis | Case service | Append/supersede with optimistic concurrency |
| Model transcript | Restricted run store | Diagnostic, access-controlled, retention-limited |
| Approval | Approval service | Immutable, exact-effect, expiring |
| Effect | Target system plus effect ledger | Reconciled observed state, not model self-report |

Do not make the vector store, prompt transcript, workflow history, or model response the source of truth for any of these.

## Key failure paths

| Failure | Safe behavior | Unsafe behavior |
|---|---|---|
| Model provider unavailable | Queue or fall back to deterministic routing; preserve alerts | Drop, auto-close, or endlessly retry hot path |
| Source query times out | Record unknown coverage and try a bounded alternate source | Interpret no rows as no compromise |
| Asset owner missing | Escalate ownership gap | Approve disruptive containment anyway |
| Conflicting identity records | Preserve conflict and request canonical lookup | Choose the record that supports the favored hypothesis |
| Duplicate alert delivery | Reuse event identity and update correlation count | Open duplicate cases and duplicate response tasks |
| Approval arrives after target changes | Revalidate and expire/re-request | Execute the stale plan |
| Worker crashes after effect submission | Reconcile by action identity | Blindly resubmit |
| Evidence hash fails | Quarantine copy, alert custodian, retain audit | Rehash and overwrite expected digest |

## Production review checklist

- [ ] The product charter names its constituency, data domains, alert families, and excluded actions.
- [ ] Every component has an owner, data classification, failure policy, and kill mechanism.
- [ ] Model outage cannot block alert intake or evidence preservation.
- [ ] The investigation plane cannot reach response credentials or arbitrary network destinations.
- [ ] Case state distinguishes proposed, corroborated, contradicted, analyst-confirmed, and superseded claims.
- [ ] Read queries and write actions are both authorized at execution time.
- [ ] Each architecture stage has explicit entry, exit, and rollback criteria.
- [ ] The incident-response team has rehearsed compromise of the agent, connector, model account, and evidence store.

## Related guides

- [Evidence intake, context, and case state](evidence-intake-context-and-case-state.md)
- [Security integrations and adapter qualification](security-integrations-and-adapter-qualification.md)
- [Authority, approvals, and constrained response](authority-approvals-and-constrained-response.md)
- [Reliability, observability, scaling, and operations](reliability-observability-scaling-and-operations.md)
- [Production agent control plane](../../architectures/production-agent-control-plane.md)
- [Execution boundaries](../../runtime/execution-boundaries.md)

## Selected sources

- [NIST SP 800-61 Rev. 3, Incident Response Recommendations and Considerations](https://csrc.nist.gov/pubs/sp/800/61/r3/final)
- [FIRST CSIRT Services Framework 2.1](https://www.first.org/standards/frameworks/csirts/csirt_services_framework_v2.1)
- [CISA Federal Government Cybersecurity Incident and Vulnerability Response Playbooks](https://www.cisa.gov/sites/default/files/2024-08/Federal_Government_Cybersecurity_Incident_and_Vulnerability_Response_Playbooks_508C.pdf)
- [OASIS CACAO Security Playbooks 2.0](https://docs.oasis-open.org/cacao/securityplaybooks/v2.0/security-playbooks-v2.0.html)
- [OASIS OpenC2 Language Specification 1.0](https://docs.oasis-open.org/openc2/oc2ls/v1.0/oc2ls-v1.0.html)
- [NIST SP 800-207, Zero Trust Architecture](https://csrc.nist.gov/pubs/sp/800/207/final)

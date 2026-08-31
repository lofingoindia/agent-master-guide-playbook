# Fraud and AML Investigation Agent Blueprint

> **Maturity:** Research-backed production blueprint; Pass 2 complete  
> **Research date:** 2026-08-31  
> **Scope:** Customer, account, transaction, and entity investigation; alert and case support; typology and network evidence; KYC/CDD/EDD support; sanctions/watchlist match analysis; investigator hypotheses and decisions; and preparation of filing, escalation, or restrictive-action packages behind strict human, legal, and deterministic authority gates.  
> **Evidence:** [Fraud and AML investigation research packet](../../research/packets/fraud-aml-investigation-agent-blueprint.md)

> Engineering guidance only—not legal, regulatory, sanctions, filing, privacy, model-validation, customer-action or
> compliance advice. Qualified local owners must determine applicability and approve every deployment and effect.

The useful product is not an autonomous financial-crime judge. It is a bounded investigator inside a deterministic case system: it assembles authorized evidence, distinguishes source fact from inference, tests competing explanations, identifies gaps, and prepares a reviewable recommendation. Accountable investigators and the institution's approved legal and compliance processes retain every dispositive decision.

## Definition of done

A deployment is fit for its declared stage only when it can demonstrate all of the following for the supported alert families and jurisdictions:

- every material claim resolves to a source record, source version, acquisition time, transformation, and case snapshot;
- party/customer, account, transaction/message, legal-entity, beneficial-owner, rule/typology, model/inference,
  watchlist-dataset/entry, alert, case, evidence, hypothesis, decision, filing, and effect identities and versions remain
  distinct;
- an investigator can see exculpatory evidence, contradictions, unavailable coverage, stale sources, and plausible benign explanations;
- the model cannot accuse a person, file a SAR/STR, block or freeze property, hold a transaction, close an account, offboard a customer, or make a dispositive risk decision;
- pending effects survive crash and retry without duplication, and ambiguous outcomes enter reconciliation rather than blind retry;
- privacy, purpose, tenancy, residency, retention, deletion, legal-hold, and SAR/STR-confidentiality rules apply to prompts, caches, traces, evaluations, exports, and memory—not only the case database;
- release evidence covers investigation quality, missed-risk cost, unnecessary-escalation cost, subgroup and language slices, authority invariants, failure recovery, throughput, human capacity, and cost per verified case outcome.

## Non-negotiable category boundary

This blueprint owns financial-crime investigation support. It does not absorb adjacent professional or operational authority.

| Adjacent owner | That owner retains | Required handoff |
|---|---|---|
| Security investigation / SOC | Cyber alerts, attacker-controlled host/network evidence, malware, compromise scope, and cyber containment | Stable incident/evidence references, affected identities or accounts, custody and sharing restrictions, and the financial hypothesis to test |
| Finance and accounting | Ordinary ledger/subledger truth, reconciliation, journal entries, close, and financial reporting | Canonical transaction and posting references plus any anomaly requiring suspicious-activity review; no transfer of ledger authority |
| Insurance claims | Ordinary coverage, loss, reserve, and claims adjudication | A fraud referral with approved evidence and purpose; no claim disposition by this agent |
| Compliance audit | Independent testing of AML/fraud controls, sampling, findings, and assurance | Versioned control evidence and read-only case samples; investigators do not test their own control effectiveness |
| Legal | Legal advice, privilege, interpretation, preservation, disclosure, and jurisdiction-specific filing obligations | Structured question, jurisdiction profile, source record, deadline, and decision owner; the agent never presents legal interpretation as fact |
| Sanctions or payment authority | Whether a transaction/property must be rejected, blocked, frozen, licensed, or released | Potential-match package with exact list/program/version, entity-resolution evidence, ownership chain, current transaction state, and deadline |

An alert is not an accusation. A typology match is not proof. A watchlist candidate is not a confirmed match. A SAR/STR is not a conviction or ground-truth label. A case closure is an investigator decision about the evidence available under a particular policy version—not a claim that misconduct did or did not occur.

## When not to build an agent

Prefer ordinary software when a bounded deterministic path meets the outcome:

- schema validation, deduplication, threshold calculation, source freshness, sanctions-list ingestion, customer or transaction lookup, and case routing;
- exact fuzzy/name matching followed by a fixed review queue when no cross-source interpretation is required;
- a reviewed monitoring rule whose inputs, output, exceptions, and escalation are already explicit;
- regulator filing validation or batch-format generation after an authorized person has made the filing decision;
- graph construction, ownership aggregation, temporal windows, peer groups, and known network features that can be computed reproducibly;
- capacity planning, SLA timers, reminders, access control, retention, legal hold, and reconciliation.

Add a model-directed loop only when heterogeneous records, ambiguous identity, conflicting evidence, narrative explanation, or hypothesis-driven retrieval creates measured investigator toil that deterministic retrieval and decision tables cannot remove. Stage 0 must compare that loop with a rules/search/workflow baseline on the same cases.

## Authority contract

This blueprint uses four effect tiers: **D1** is purpose-bound read-only access; **D2** creates drafts or internal staged artifacts that remain reviewable; **D3** commits a consequential customer, transaction, regulatory, or external action; **D4** changes authority, policy, credentials, or control configuration. The agent's ceiling is D1 plus validated D2 proposals.

| Capability | Default tier | Agent role | Commit authority |
|---|---:|---|---|
| Read a purpose-bound case projection | D1 | Select queries and summarize returned evidence | Deterministic authorization broker |
| Add a draft hypothesis, cited claim, or proposed next step | D2 | Propose typed case artifacts | Case workflow validates schema, provenance, author, and case version |
| Request KYC/CDD/EDD evidence or create an internal review task | D2 | Propose a narrow request | Workflow policy and eligible human owner |
| Prepare a SAR/STR or regulator-escalation package | D2 | Draft from approved evidence and current jurisdiction schema | Designated human/legal process owns decision, attestation, and submission |
| Recommend hold, block, freeze, reject, offboard, or law-enforcement escalation | D3 | Proposal only, with impact and uncertainty | Independent authorized person or deterministic legal/runbook gate |
| File, freeze, block, offboard, accuse, change monitoring policy, grant access, or weaken controls | D3/D4 | Prohibited | Separate administrative, legal, compliance, or operational path |

The reasoning runtime has no regulator credential, payment-control credential, unrestricted customer-data query, generic database write, or tool-registry administration right.

## System context

~~~mermaid
flowchart LR
    subgraph Sources["Authorized systems and source snapshots"]
        TX["Transactions and payment messages"]
        KYC["Customer / account / KYC master"]
        LIST["Sanctions, PEP, watchlists, advisories"]
        EXT["Approved fraud, device, registry, and public sources"]
    end
    subgraph Trusted["Trusted case and evidence plane"]
        IN["Admission, identity, purpose, deduplication"]
        RAW["Immutable source objects + lineage"]
        GRAPH["Versioned entity and transaction views"]
        CASE["Authoritative alert / case state"]
        CTX["Policy-aware context compiler"]
    end
    subgraph Reasoning["Untrusted proposal plane"]
        INV["Bounded investigator"]
        READ["Typed read broker"]
    end
    subgraph Control["Deterministic authority plane"]
        PDP["Policy, jurisdiction, segregation of duties"]
        HUMAN["Qualified investigator / legal / designated approver"]
        LEDGER["Decision, filing, and effect ledgers"]
    end
    subgraph Destinations["Separated destinations"]
        FILE["Filing / escalation adapter"]
        RESTRICT["Hold / block / freeze / offboard workflow"]
    end

    Sources --> IN
    IN --> RAW
    RAW --> GRAPH
    IN --> CASE
    GRAPH --> CTX
    CASE --> CTX
    CTX --> INV
    INV --> READ
    READ --> Sources
    INV -. "typed proposals only" .-> CASE
    CASE --> PDP
    PDP --> HUMAN
    HUMAN --> PDP
    PDP --> LEDGER
    LEDGER --> FILE
    LEDGER --> RESTRICT
    FILE --> LEDGER
    RESTRICT --> LEDGER
    LEDGER --> CASE
~~~

The evidence store preserves what a source supplied. The graph is a versioned derived view, not a new truth system. The case store owns workflow state and human decisions. The model receives a minimized case projection and produces proposals. A separate authority plane binds current policy, jurisdiction, exact target, source freshness, approval, and segregation of duties before any consequential effect.

## Architecture selection

| Path | Use when | Strength | Reject or change when |
|---|---|---|---|
| Deterministic workflow, search, and rules | Evidence and decision path are stable; model benefit is unproven | Lowest behavioral risk and easiest validation | Investigators repeatedly need cross-source synthesis or adaptive follow-up queries |
| Custom bounded loop | One or a few alert families, narrow typed tools, short cases | Smallest inspectable agent surface; easy to keep read-only | Long waits, multiple approvals, high case concurrency, or resumability dominate |
| Agent SDK inside case service | Team needs structured tool calling, tracing, and model portability | Reduces loop plumbing while application retains policy/state | SDK state or approvals are mistaken for the business ledger or authorization |
| Workflow engine plus bounded model step | Cases wait on evidence/people and must resume across releases | Durable timers, signals, ownership, retries, and version policy | Chosen only for fashion; short synchronous cases do not justify operations cost |
| Multi-agent investigation | Rarely justified; only for isolated source domains or measured independent challenge | Parallel retrieval or intentionally independent review | Default choice. Shared context, authority, duplicated queries, and correlated errors outweigh benefit |

Recommended progression: deterministic case workflow → one read-only custom/SDK loop → durable hybrid only when real waits and recovery require it. Use deterministic fan-out workers for independent evidence retrieval and graph queries; do not imitate an organization chart with model agents.

For the application and policy plane, choose the language the regulated operations team can test and operate. Python is often strongest for data/graph/model evaluation; TypeScript/Node or JVM/.NET ecosystems may better fit enterprise services, schemas, and identity middleware. Keep hot transaction screening and bulk graph computation outside the conversational loop. A polyglot boundary is justified only by an existing platform or measured workload need, and every boundary must preserve case, source, run, and trace identities.

## Read in this order

1. [Workload fit, authority, and Stage 0–6 gates](01-workload-fit-authority-and-stages.md)
2. [Reference architecture and integration contracts](02-reference-architecture-and-integration-contracts.md)
3. [Customer, transaction, entity, and network evidence](03-customer-transaction-entity-and-network-evidence.md)
4. [Alerts, cases, typologies, hypotheses, and decisions](04-alert-case-typology-and-investigation-reasoning.md)
5. [KYC, CDD, EDD, sanctions, and source semantics](05-kyc-cdd-edd-sanctions-and-source-semantics.md)
6. [State, context, memory, planning, and provenance](06-state-context-memory-planning-and-provenance.md)
7. [Filings, effects, idempotency, reconciliation, and recovery](07-filings-effects-idempotency-and-recovery.md)
8. [Security, permissions, privacy, fairness, and governance](08-security-privacy-fairness-and-governance.md)
9. [Evaluation, observability, SLOs, and failure injection](09-evaluation-observability-slos-and-failure-injection.md)
10. [Deployment, scale, incidents, cost, and governed evolution](10-deployment-scale-incidents-cost-and-evolution.md)
11. [Qualified adapters and worked investigation flows](11-qualified-adapters-and-worked-investigation-flows.md)

## Stage 0–6 summary

| Stage | Delivered capability | Exit evidence |
|---:|---|---|
| 0 | Deterministic baseline and agent qualification | Same-case comparison proves a bounded model step improves a named outcome without hiding authority |
| 1 | One read-only alert-family loop | Typed tools, citations, benign alternative, missing-evidence state, budgets, and human review pass |
| 2 | Useful MVP in the real case environment | Authorized context, short-term working state, representative evals, and no external effects |
| 3 | Reliable v1 | Durable case state, compaction tests, adapter versions, cancellation, idempotency, reconciliation, and recovery drills pass |
| 4 | Production | Identity, tenant/purpose enforcement, release manifest, tracing/SLOs, privacy, security, runbooks, canary, and rollback pass |
| 5 | Scale and resilience | Bounded queues, human-capacity model, cell/tenant isolation, DR, degradation, and cost-per-verified-outcome targets pass |
| 6 | Governed evolution | Failure mining, drift and bias monitoring, independent validation, change gates, refresh triggers, and deprecation paths operate continuously |

Later stages expand proven operating scope; they never expand professional authority merely because accuracy improved.

## Top stop and escalation conditions

Stop the loop and preserve state when identity is ambiguous, a source is outside purpose or jurisdiction, required coverage is missing, a material source changed after the snapshot, case versions conflict, a watchlist program/effect is unclear, evidence may be privileged or SAR/STR-confidential, the loop requests a forbidden sink, an approval is stale, the filing deadline is at risk, an external outcome is unknown, a cross-tenant signal appears, or capacity limits would make review superficial.

The system must support an out-of-band switch to stop new admission, force proposal-only mode, disable a connector or effect class, revoke credentials, quarantine a release or contaminated memory/index, and reconcile every pending filing or restrictive action.

## Refresh triggers

Refresh immediately when FATF Recommendations or jurisdictional AML/CFT rules change; AMLA or a national FIU changes filing schemas or deadlines; a sanctions authority changes list formats, ownership rules, programs, or reporting paths; a regulator revises monitoring/model-risk expectations; a provider changes data handling, compaction, tool, or retention behavior; a new alert family, country, rail, watchlist, source, language, effect, or multi-tenant boundary is added; or an incident reveals missed risk, discriminatory burden, confidentiality breach, unknown effect, or evaluator leakage. Otherwise review volatile integrations and agent controls at least every 90 days and the full workload architecture at least every 180 days.

## Canonical repository dependencies

- [Agent blueprint cross-cutting controls](../../research/packets/agent-blueprint-cross-cutting-controls.md)
- [Agent state and event contracts](../../runtime/agent-state-and-event-contracts.md)
- [Compaction and continuity](../../context-memory/compaction-and-continuity.md)
- [Memory architecture](../../context-memory/memory-architecture.md)
- [Permissions, sandboxing, and secrets](../../security/permissions-sandboxing-and-secrets.md)
- [Prompt injection and untrusted data](../../security/prompt-injection-and-untrusted-data.md)
- [Idempotency and side effects](../../reliability/idempotency-and-side-effects.md)
- [Trajectory and reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md)
- [Deployment, release, and incident response](../../operations/deployment-release-and-incident-response.md)

## Selected primary sources

- [FATF Recommendations, amended June 2026](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Fatf-recommendations.html)
- [FFIEC BSA/AML Manual — Suspicious Activity Reporting](https://bsaaml.ffiec.gov/manual/AssessingComplianceWithBSARegulatoryRequirements/04)
- [FinCEN — Frequently Asked Questions Regarding the FinCEN SAR](https://www.fincen.gov/resources/frequently-asked-questions-regarding-fincen-suspicious-activity-report-sar)
- [OFAC — assessing a potential sanctions-list match](https://ofac.treasury.gov/faqs/5)
- [OFAC Sanctions List Service](https://ofac.treasury.gov/sanctions-list-service)
- [AMLA regulatory-instrument register](https://www.amla.europa.eu/policy/regulatory-instruments_en)
- [FATF — Information Sharing to Combat Illicit Finance (2026)](https://www.fatf-gafi.org/en/publications/Methodsandtrends/information-sharing-ppp-data-protection-arrangements.html)
- [Federal Reserve — Supervisory Guidance on Model Risk Management (SR 26-2, 2026)](https://www.federalreserve.gov/frrs/guidance/supervisory-guidance-on-model-risk-management.htm)

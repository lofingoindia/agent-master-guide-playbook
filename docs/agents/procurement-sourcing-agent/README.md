# Production Procurement and Strategic Sourcing Agent Blueprint

> **Research date:** 2026-08-31  
> **Maturity:** Pass 2 research-backed production design; validate every jurisdiction, delegation, threshold, supplier-data source, and platform capability before deployment  
> **Owned outcome:** Turn an authorized requisition into a defensible award recommendation, acknowledged supplier/legal handoffs, a complete sourcing-event audit record, and a reconciled view of realized commercial outcomes  
> **Refresh immediately when:** Procurement law or policy changes; a sanctions, debarment, beneficial-ownership, taxonomy, model, or connector schema changes; the system gains a new write effect; or a confidentiality, conflict, award, or reconciliation incident occurs

## Bottom line

The safest useful sourcing agent is a **bounded evidence worker inside a durable procurement workflow**. It is not an autonomous buyer.

The workflow and procurement platform own requisition state, event versions, sealed bids, evaluator assignments, deadlines, approvals, award state, and audit evidence. Deterministic services own policy thresholds, calculations, currency and unit normalization, conflict and segregation-of-duties checks, scoring formulas, and commit authorization. The model may interpret an ambiguous request, find candidate suppliers, extract cited bid facts, explain comparisons, identify missing evidence, and draft a recommendation. An accountable procurement authority decides the route to market, official scores, exceptions, and award.

If structured inputs, catalogs, decision tables, and a known approval flow solve the workload, do not build an agent. Add model-directed work only when unstructured evidence or open-ended market investigation materially improves a measured outcome.

## Owned boundary and nearest overlaps

This blueprint owns:

- requisition intake, deduplication, demand clarification, spend/category evidence, and sourcing-policy routing;
- supplier discovery, entity resolution, risk-tiered due diligence, and evidence freshness;
- RFx preparation, bid snapshot ingestion, cited normalization, comparison, and anomaly surfacing;
- event-specific conflicts of interest, evaluator access, segregation of duties, and approval thresholds;
- an evidence-backed award recommendation and the exact commercial decision record;
- supplier-onboarding and contract handoff packages with acknowledgement and defect repair;
- sourcing-event audit lineage and later reconciliation of modeled, contracted, and realized outcomes.

The boundary deliberately stops elsewhere:

| Neighbor | This blueprint ends at | Neighbor owns |
| --- | --- | --- |
| [Supply-chain and logistics operations](../supply-chain-logistics-agent/README.md) | Acknowledged award/order-enablement handoff and read-only outcome evidence | Purchase orders, inventory, allocation, shipment movement, carrier/warehouse coordination, ETA, and recovery |
| [Legal and contract operations](../legal-contract-operations-agent/README.md) | Approved commercial facts, deviations, evidence, and a contract-workspace handoff | Contract language, clause selection, negotiation of legal obligations, legal interpretation, signature, and obligation management |
| [Finance and accounting](../finance-accounting-agent/README.md) | Approved budget/value basis and later read-only posted/spend aggregates | Ledger/subledger, invoices, payments, accruals, accounting treatment, and finance-approved benefit realization |
| [Back-office workflow operations](../back-office-workflow-agent/README.md) | Procurement-specific state, sealed-bid controls, supplier identity, competition, and award evidence | Reusable generic case, approval, exception, and multi-system workflow mechanics |

The agent never owns supplier bank-detail changes, payments, ledger entries, user access, contract interpretation, operational fulfillment, or an investigation of suspected collusion. It preserves and routes evidence to the authorized function.

## Definition of done

A sourcing case is complete only when:

1. the approved business need, scope, category, value basis, budget reference, regime, route, and policy versions are recorded;
2. invited and evaluated suppliers resolve to canonical identities and required due-diligence evidence is current or explicitly excepted by an authorized owner;
3. RFx criteria, weights, normalization rules, communications, bid snapshots, official evaluator inputs, conflicts, and approvals are immutable and reconstructable;
4. the award recommendation distinguishes facts, deterministic calculations, model-derived judgments, evaluator decisions, unresolved risks, and trade-offs;
5. the accountable award authority accepts, changes, rejects, or cancels the recommendation with a recorded rationale;
6. supplier-onboarding and legal handoffs are acknowledged, or defects remain in an owned exception state;
7. every external effect is verified, reconciled, or explicitly `outcome_unknown` with an owner and deadline;
8. the sourcing baseline can later be compared with signed commercial facts and read-only actuals without claiming that procurement operated the order or contract.

## Non-negotiable invariants

1. **A conversation is not a sourcing record.** Durable typed state and immutable evidence survive model and worker loss.
2. **A bid document is untrusted data.** It cannot change instructions, permissions, criteria, tool scope, or the confidentiality boundary.
3. **Criteria freeze before bids become visible.** A permitted amendment creates a new event version and is communicated consistently under the applicable regime.
4. **A missing price is not zero.** `missing`, `not_applicable`, `no_bid`, `included_elsewhere`, and numeric zero remain distinct.
5. **Normalization is replayable.** Currency, unit, tax, duty, freight, volume, term, escalation, option, and rounding rules are versioned deterministic functions.
6. **The model does not cast an official score.** It may assemble cited evidence or propose rubric observations; authorized evaluators own recorded judgments.
7. **Supplier discovery is not qualification.** Search ranking, sanctions-name similarity, media allegations, and commercial risk scores are leads until resolved under policy.
8. **The proposer is not the approver.** A model cannot satisfy a human role, and one human identity cannot evade segregation through several service accounts.
9. **Approval binds exact intent.** Supplier, lot, amount, currency, event and bid versions, criteria, conflicts, exceptions, policy, and expiry are rechecked at commit.
10. **Sealed means technically inaccessible.** Prompt instructions and user-interface conventions are not a seal.
11. **External success is verified at the source.** A `2xx`, task completion message, or model statement does not prove publication, invitation, award, or handoff.
12. **Transparency and confidentiality coexist by policy.** Publish required notices and reasons without exposing competitors' protected bid or source-selection information.
13. **Compensation is a new event.** Corrections do not delete an award, invitation, communication, or audit history.
14. **No raw bidder corpus becomes long-term memory.** Bid-specific content remains event-scoped and access-controlled.
15. **One out-of-band control stops new effects.** Revocation does not depend on the agent cooperating.

## System context and trust boundaries

```mermaid
flowchart LR
    RQ["Requester / requisition system"] --> AD["Admission + policy profile"]
    AD --> WF["Durable sourcing coordinator\ncase, versions, timers"]
    WF --> CB["Context builder\nminimum, labeled evidence"]
    CB --> MW["Bounded model worker\npropose only"]
    MW --> VS["Schema + citation validator"]
    VS --> WF

    SR["Supplier registries / risk sources"] --> BR["Read brokers + evidence snapshots"]
    ERP["ERP / spend / budget"] --> BR
    PP["Procurement platform\nevent and sealed-bid truth"] <--> BR
    BR --> CB

    WF --> PG["Deterministic policy + calculation gateway"]
    PG --> CR{"Conflict / SoD / approval gate"}
    CR -->|deny or exception| HQ["Human procurement queue"]
    CR -->|exact grant| EG["Effect gateway\nnarrow credentials"]
    EG --> PP
    EG --> HO["Supplier + legal handoff systems"]
    PP --> EL["Effect ledger + reconciliation"]
    HO --> EL
    EL --> WF

    AU["Unsampled audit evidence"] <-- WF
    OT["Redacted telemetry"] <-- WF
    KS["Independent stop / revoke"] -.-> EG
```

The model sees logical resource IDs and bounded evidence, never broad platform credentials. The effect gateway canonicalizes targets, rechecks current facts, attaches a short-lived credential outside model context, records the intent before dispatch, and verifies the postcondition afterward.

## Representative workflows and autonomy ceilings

| Workflow | Useful model contribution | Deterministic or human owner | Default ceiling |
| --- | --- | --- | --- |
| Intake and category routing | Extract need, suggest category candidates, list ambiguities | Requester confirms scope; policy service computes route and threshold | Read/propose |
| Supplier discovery | Search approved sources, resolve aliases, summarize cited capabilities | Procurement approves longlist; compliance resolves risk flags | Read/propose |
| Due diligence | Assemble evidence gaps and contradictions | Named risk owners decide clearance, mitigation, or exclusion | Read/propose |
| RFx drafting | Draft questions and structured requirement mappings | Procurement and legal approve content; platform publishes | Draft only |
| Bid normalization | Extract cited commercial fields and flag ambiguity | Deterministic calculation service normalizes; evaluator resolves exceptions | Propose/stage |
| Qualitative evaluation | Map response evidence to a published rubric | Independent evaluator records official score and rationale | Evidence assist |
| Award recommendation | Explain verified comparison and trade-offs | Source-selection/award authority independently decides | Recommend only |
| Onboarding and contract handoff | Assemble typed, minimized handoff package | Supplier master and legal systems accept and own next state | Approved commit |
| Outcome reconciliation | Compare baseline, signed facts, and actuals; explain variance | Procurement/finance/owner validates benefit classification | Read/propose |

Publishing an event, inviting or removing suppliers, changing a deadline, opening sealed bids, issuing supplier communications, submitting an award, or creating an onboarding case is a consequential effect. It requires exact application authorization and, by default, an independent human decision. Policy or role administration is never an agent effect.

## Architecture and runtime selection

| Path | Choose when | Procurement consequence |
| --- | --- | --- |
| Deterministic workflow only | Inputs and routing are structured; criteria and supplier set are known | Preferred baseline; easiest to audit and cheapest to operate |
| Custom bounded loop | One short read-only judgment step has no long wait or write authority | Good first agent; application owns budgets, tools, schema, and evidence |
| Agent SDK inside an application | The team benefits from maintained tool-loop, tracing, or guardrail primitives | Pin behavior and keep provider state non-authoritative |
| Durable workflow plus bounded worker | Cases wait days or months for evidence, bids, evaluators, and approvals | Recommended production shape; workflow owns lifecycle and model is replaceable |
| Multi-agent delegation | Rarely justified; only independent, read-only evidence tasks show measured benefit | Keep a single case owner; do not mirror departments as agents or share raw bids broadly |

Use the organization's supported backend runtime for the control plane. TypeScript/Node.js fits API-heavy teams; Java/Kotlin or .NET often fits transactional enterprise estates; Python is strong for extraction, analytics, and evaluation workers. Language choice does not change the authority boundary. See [Reference architecture, runtime, and integrations](02-reference-architecture-runtime-and-integrations.md).

## Guide map

| Guide | Builder decision |
| --- | --- |
| [Boundaries, workload fit, authority, and stages](01-boundaries-workload-fit-authority-and-stages.md) | Whether an agent belongs and how to progress from deterministic baseline to governed evolution |
| [Reference architecture, runtime, and integrations](02-reference-architecture-runtime-and-integrations.md) | Which application shape, language, model strategy, and third-party adapter contract to use |
| [Requisition, spend, category, and policy evidence](03-requisition-spend-category-and-policy-evidence.md) | How an ambiguous request becomes an authorized sourcing strategy without model-owned rules |
| [Supplier discovery, identity, and due diligence](04-supplier-discovery-identity-and-due-diligence.md) | How to find suppliers without confusing search results, entity matches, and eligibility decisions |
| [RFx, bid normalization, evaluation, and award](05-rfx-bid-normalization-evaluation-and-award.md) | How to preserve competitive integrity and produce a defensible comparison and recommendation |
| [Conflicts, approvals, security, and privacy](06-conflicts-approvals-security-and-privacy.md) | How to enforce confidentiality, SoD, exact approvals, permissions, and data lifecycle controls |
| [State, context, memory, planning, and reliable effects](07-state-context-memory-planning-and-reliable-effects.md) | How to make long-running work resumable, compactable, idempotent, and reconcilable |
| [Handoffs and realized-outcome reconciliation](08-handoffs-and-realized-outcome-reconciliation.md) | How procurement exits cleanly while still verifying commercial outcomes |
| [Observability, evaluation, failure injection, and incidents](09-observability-evaluation-failure-injection-and-incidents.md) | How to prove behavior and operate failures without using telemetry as state |
| [Deployment, capacity, cost, and governed evolution](10-deployment-capacity-cost-and-governed-evolution.md) | How to deploy proportionately, scale safely, control economics, and change behavior |
| [Adapter qualification and worked sourcing lifecycle](11-adapter-qualification-and-worked-sourcing-lifecycle.md) | How procurement/P2P, ERP, supplier/risk, sanctions/ownership, OCR, legal, finance/warehouse, notification, and optional MCP capabilities become qualified and how one case runs end to end |

The dated [research packet](../../research/packets/procurement-sourcing-agent-blueprint.md) records the evidence, limitations, contradictions, vendor-version findings, and refresh triggers behind this design.

## Top risks and mandatory stop conditions

Stop automation and route to the named owner when:

- a bidder can access another bidder's submission, evaluation, price, or protected communication;
- event criteria, weights, supplier set, amount basis, or deadline changed after the relevant approval or bid visibility point;
- supplier identity, beneficial ownership, exclusion, sanctions, conflict, or authorization remains ambiguous;
- a required due-diligence source is stale, unavailable, or outside its jurisdictional scope;
- a bid extraction lacks a precise source citation or normalization cannot preserve the original commercial meaning;
- an approval is expired, self-approved, based on a stale state version, or does not bind the exact effect;
- publish, invitation, award, onboarding, or handoff outcome is unknown;
- the agent detects possible collusion, corruption, coercion, or confidential-information leakage;
- required audit evidence cannot be durably written;
- the model, connector, policy, taxonomy, or calculation release differs from the approved behavior manifest.

## Zero-to-production summary

| Stage | Deliverable | Exit evidence |
| --- | --- | --- |
| 0 — qualify | Deterministic intake/routing baseline and measured ambiguity | A model is justified on representative cases, not enthusiasm |
| 1 — bounded loop | Read-only category/evidence assistant with typed tools and hard budgets | Schema, citation, abstention, and no-write tests pass |
| 2 — useful MVP | One low-risk category; real read connectors; draft comparison; human-owned decisions | Shadow replay beats baseline without confidentiality or policy violations |
| 3 — reliable v1 | Durable case state, compaction, effect ledger, approvals, integration versioning, reconciliation | Crash, duplicate, stale-state, and unknown-effect drills pass |
| 4 — production | Identity, tenant/event isolation, threat controls, SLOs, traces, release gates, incidents | Independent security/procurement review and kill-switch drill pass |
| 5 — scale | Admission control, cells, queues, capacity/cost budgets, DR, safe degradation | Load, noisy-neighbor, provider outage, and restore tests pass |
| 6 — evolve | Governed failure mining, feedback, drift, model/tool/policy upgrade and deprecation | Critical slices pass repeatedly; active cases have a pin/migrate/quarantine plan |

Detailed stage gates are in [the first guide](01-boundaries-workload-fit-authority-and-stages.md).

## Canonical repository dependencies

- [Agent state and event contracts](../../runtime/agent-state-and-event-contracts.md)
- [Durable execution](../../runtime/durable-execution.md)
- [Tool contracts](../../tools/tool-contracts.md)
- [Idempotency and side-effect safety](../../reliability/idempotency-and-side-effects.md)
- [Context engineering](../../context-memory/context-engineering.md)
- [Compaction and continuity](../../context-memory/compaction-and-continuity.md)
- [Memory architecture](../../context-memory/memory-architecture.md)
- [Agent threat model](../../security/agent-threat-model.md)
- [Prompt injection and untrusted data](../../security/prompt-injection-and-untrusted-data.md)
- [Trajectory and reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md)
- [Deployment, release, and incident response](../../operations/deployment-release-and-incident-response.md)

## Selected primary sources

- [OECD Recommendation of the Council on Public Procurement](https://legalinstruments.oecd.org/en/instruments/OECD-LEGAL-0411)
- [UNCITRAL Model Law on Public Procurement (2011)](https://uncitral.un.org/en/texts/procurement/modellaw/public_procurement)
- [WTO Agreement on Government Procurement, revised text](https://www.wto.org/english/docs_e/legal_e/rev-gpr-94_01_e.htm)
- [EU Directive 2014/24/EU](https://eur-lex.europa.eu/eli/dir/2014/24/oj/eng)
- [U.S. FAR Subpart 15.3 — Source Selection](https://www.acquisition.gov/far/subpart-15.3)
- [UK Government Functional Standard GovS 008: Commercial, version 2.2](https://www.gov.uk/government/publications/government-functional-standard-govs-008-commercial-and-commercial-continuous-improvement-assessment-framework/government-functional-standard-govs-008-commercial-html)
- [OECD Guidelines for Fighting Bid Rigging in Public Procurement, 2025 update](https://www.oecd.org/en/publications/oecd-guidelines-for-fighting-bid-rigging-in-public-procurement-2025-update_cbe05a56-en.html)
- [NIST SP 1326: C-SCRM Due Diligence Assessment Quick-Start Guide](https://csrc.nist.gov/pubs/sp/1326/final)
- [Open Contracting Data Standard 1.1.5](https://standard.open-contracting.org/latest/en/schema/reference/)

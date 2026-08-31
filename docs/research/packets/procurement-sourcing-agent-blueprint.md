# Research Packet: Procurement and Strategic Sourcing Agent Blueprint

> **Status:** Research-backed Pass 2 usefulness and production-depth refinement; coordinator review and target-organization validation still required  
> **Research date:** 2026-08-31  
> **Delegated category:** Registry category 32 — Procurement and strategic sourcing  
> **Blueprint:** [Production Procurement and Strategic Sourcing Agent Blueprint](../../agents/procurement-sourcing-agent/README.md)  
> **Scope:** Requisition intake, spend/category and policy evidence, supplier discovery and due diligence, RFx/bid normalization and comparison, conflicts and segregation of duties, approval thresholds, award recommendation, supplier onboarding/contract handoff, sourcing-event audit, and realized-outcome reconciliation  
> **Method:** Current primary standards, laws/regulations and official guidance, official data standards, official registries/list services, current procurement-platform documentation, current agent-engineering guidance, and adjacent repository guides were compared. Public-procurement materials were used as regime-specific evidence and design stressors, not presented as universal private-sector law.

## Research questions

1. Is this a distinct model-directed workload or only a generic back-office case with procurement nouns?
2. Which sourcing steps benefit from model-directed evidence work, and which must remain deterministic or human-owned?
3. What records prove fair treatment, competitive-event integrity, supplier identity, due diligence, evaluation, and award accountability?
4. How do sealed bids, published criteria, conflicts, segregation of duties, approvals, and confidential source-selection information change the security model?
5. How should heterogeneous bids be extracted and normalized without hiding missing data or changing commercial meaning?
6. Which supplier registries, sanctions/debarment sources, ownership standards, data formats, and sourcing-suite APIs are usable, and what do they not guarantee?
7. What is the exact boundary with legal operations, supply-chain/logistics, FinOps, and generic back-office workflows?
8. How should long-running state, context compaction, every memory class, planning, effects, idempotency, reconciliation, and recovery work?
9. Which evaluations, failures, SLOs, capacity models, incident controls, and release gates establish production readiness?
10. How can outcomes improve future sourcing without creating a hidden, stale, or discriminatory supplier memory?

## Research method and saturation

Research proceeded across six evidence layers:

| Layer | Sources reviewed | Decision supported |
| --- | --- | --- |
| Procurement principles and regimes | OECD, UNCITRAL, WTO GPA, EU Directive 2014/24/EU, World Bank IPF rules, U.S. FAR, UK GovS 008/Sourcing Playbook | Boundary, event integrity, criteria, award, records, regime profiles |
| Integrity and internal control | OECD bid-rigging guidance, DOJ/SEC FCPA guidance, DOJ compliance-program guidance, GAO Green Book, FAR conflict/procurement-integrity rules | Conflicts, SoD, due diligence, anomaly escalation, approvals |
| Supplier and supply-chain risk | NIST SP 800-161 Rev. 1 update 1, NIST SP 1326, FATF ownership guidance, GLEIF, BODS, OFAC, SAM.gov, World Bank debarment | Entity evidence, source scope/freshness, ICT supplier risk, list matching |
| Data and interoperability | OCDS 1.1.5, OASIS UBL 2.4, CloudEvents 1.0.2, CPV, UNSPSC | Record model, event versioning, classifications, handoff schema choices |
| Current product mechanics | SAP Ariba Event Management API 2605, Oracle Fusion Cloud Procurement 26B APIs, Coupa R44 schemas/current and legacy documentation | Adapter boundaries, feature/version caveats, permission and reconciliation tests |
| Agent production controls | Repository cross-cutting packet and canonical state/effect/context/security/eval/operations guides; OpenAI, Anthropic, NIST, W3C, OpenTelemetry | Bounded loop, tool risk, context/memory, evaluation, telemetry, incidents |

Discovery continued through variations on sourcing law, conflicts, supplier qualification, sanctions, beneficial ownership, cyber supplier risk, bid evaluation, abnormal pricing, collusion, procurement schemas, suite APIs, benefits realization, and agent evaluation. Saturation was reached when new primary sources stopped changing the category boundary, hybrid architecture, authority ceiling, state/effect model, evaluation suite, or staged roadmap. Important jurisdiction-specific details remain adoption-time work.

No vendor case study or marketing claim was used to prove that an AI sourcing agent improves outcomes. The blueprint requires an organization to establish that through its own deterministic baseline and evaluations.

## Category promotion record

| Gate | Evidence | Result |
| --- | --- | --- |
| Real-agent fit | Free-form demand/category clarification and open-ended supplier/bid evidence acquisition require runtime tool choice and feedback; deterministic alternatives remain for structured work | Pass, conditionally |
| Distinct architecture | Sealed competitive events, frozen criteria, bidder isolation, event-specific conflicts, official evaluator judgment, and award authority differ materially from generic cases | Pass |
| Buildability | A durable workflow plus replaceable bounded model worker, typed brokers, policy/calculation services, and effect gateway is implementable with current systems | Pass |
| Production depth | Early bid access, competitor leakage, criteria drift, identity/list ambiguity, conflicted scoring, award timeout, and partial handoff create unique failure/recovery seams | Pass |
| Evaluation viability | Synthetic sourcing estate can grade event state, access, calculations, approvals, effects, handoffs, outcomes, and human judgment | Pass |
| Evidence depth | Multiple current primary regimes, standards, registries, security guidance, and live platform APIs support material decisions | Pass |
| Reader value | Removes repeated research across procurement controls, data semantics, integrations, and production-agent operation | Pass |
| Promotion decision | A distinct, evidence-rich blueprint is justified only with the strict owned boundary and proposer-only model role | **Promote as research-backed draft** |

The anti-chatbot test passes. Representative agentic workflows are not `message -> retrieval -> answer`:

1. An ambiguous requisition triggers bounded evidence acquisition across spend, taxonomy, agreement, and policy tools, then returns an authorized strategy question set.
2. An authorized bid snapshot triggers isolated extraction, deterministic normalization, evidence-gap planning, controlled clarification proposals, and a recommendation packet assembled from official evaluator and due-diligence records.

The model makes useful runtime evidence decisions but never owns the procurement lifecycle or award.

## Boundary findings

### Owned outcome

The category owns the evidence and control path from accepted requisition through award recommendation and acknowledged supplier/legal handoffs, plus sourcing audit and later realized-outcome reconciliation.

### Required exclusions

- **Supply-chain/logistics:** owns orders, releases, allocation, inventory, shipment movement, warehouse/carrier work, ETA, operational exceptions, and recovery after award.
- **Legal operations:** owns legal text, clause selection, interpretation, negotiated obligations, signature, and obligation management. Procurement hands off verified commercial facts and deviations.
- **Finance/accounting:** owns ledger/subledger, invoice/payment, accrual/accounting treatment and finance-approved benefit realization. Procurement provides an approved commercial baseline and consumes read-only actuals; it does not become the accounting or FinOps engine.
- **Back-office workflows:** own reusable generic case/approval/effect mechanics. This specialist adds competitive-event confidentiality, supplier identity, sourcing regime, bid comparison, evaluator independence, and award evidence.

Supplier bank-detail changes, payments, ledger entries, access grants, contract interpretation, and operational fulfillment were rejected from the tool surface.

## Baseline as researched

| Source or mechanism | Version/date observed | What it establishes | Limitation or contradiction |
| --- | --- | --- | --- |
| OECD Public Procurement Recommendation | OECD/LEGAL/0411, adopted 2015, in force; implementation report covers 2020–2024 | Integrated principles include transparency, integrity, access, efficiency, e-procurement, risk, accountability, evaluation | Public-sector recommendation; not private law or executable policy |
| UNCITRAL Model Law on Public Procurement | 2011 | Objectivity, fairness, participation, competition, integrity, transparency, electronic procurement, challenges | Model law; applicable duties depend on enactment |
| WTO GPA revised agreement | 2012 protocol text | Pre-specified criteria, equal information, award transparency, confidentiality, electronic traceability | Plurilateral; only covered parties/entities/procurements |
| EU Directive 2014/24/EU | Consolidated text observed through 2024-01-01 | Electronic communication, conflicts, exclusions, verifiable/weighted award criteria, procedure documentation | Member-state transposition and thresholds/remedies govern actual event |
| World Bank IPF Procurement Regulations | 7th edition, September 2025 | Current Bank-financed process baseline; rated criteria and abnormal-bid guidance | Applies to covered Bank-financed projects, not all buyers |
| U.S. FAR | Acquisition.gov pages showed FAC 2026-01, effective 2026-03-13 | Source-selection responsibility, factors, independent authority judgment, bid information protection, conflicts, price analysis | U.S. federal scope and agency supplements; volatile reform/version context |
| UK GovS 008 Commercial | Version 2.2, updated 2026-05-14 | Commercial lifecycle, value for money, responsibilities, SoD, measurable outcomes | UK government standard; not universal corporate policy |
| UK Sourcing Playbook | Updated 2026-06-15 | Market preparation, bid evaluation, due diligence, benefit measurement, annual review | Central-government context and referenced legal/policy framework |
| GAO Green Book | 2025 edition, effective beginning FY2026 | Internal-control system, control activities, reliable information, compliance | U.S. federal standard; useful control model rather than universal mandate |
| OECD bid-rigging guidelines | 2025 update, published 2025-09-11 | Tender-design and red-flag guidance, specialist reporting/cooperation | Red flags are not proof; primarily public procurement |
| NIST SP 800-161 Rev. 1 | May 2022, update 1 through 2024-11-01 | Organization-wide cyber supply-chain risk management | Broad ICT/OT risk guidance, not supplier eligibility automation |
| NIST SP 1326 | Final, July 2026 | ICT supplier due-diligence components: FOCI, provenance, resilience, foundational cyber practices, tiers | ICT-scoped quick-start; risk owners must tailor |
| OCDS | 1.1.5 current documentation at research date | Versioned release/record data for planning, tender, award, contract, implementation | Publication/data standard, not workflow engine, authority model, or private record schema |
| OASIS UBL | 2.4 | Standard procurement documents including RFQ/quotation families | Exchange schemas do not supply organization policy or effect correctness |
| BODS | 0.4 | Machine-readable entity/person/relationship statements and record identity | Explicitly pre-1.0 with anticipated changes; source data quality still governs |
| OFAC Sanctions List Service | Primary list delivery since 2024-05-06 | Official U.S. sanctions list data and fuzzy search capability | Applicable program/scope and human match resolution remain necessary |
| SAP Ariba Event Management API | Document version 2605 — 2026-05 | Event, bids, participants, scenarios, awards, sealed-bid and audit operations | Documentation says client must control intended API-user access; product/site restrictions apply |
| Oracle Fusion Cloud Procurement | 26B; API docs updated 2026-04/05 | Versioned sourcing/procurement REST resources and custom actions | Some resources/actions require opt-in; exact tenant behavior must be tested |
| Coupa | R44 schemas listed current; indexed sourcing page marked legacy/unmaintained | Supplier and sourcing APIs exist and releases are versioned | Do not implement from legacy page; target tenant/current supported schema required |
| OpenTelemetry GenAI conventions | Current pages at 2026-08-31 | Emerging agent/model/tool trace vocabulary | Development/evolving; application stability and content protection required |

## Evidence findings and blueprint consequences

### Finding 1: an agent is optional, but process control is not

Structured requisitions, catalogs, framework call-offs, deterministic thresholds, fixed RFQs, and formulaic scorecards are ordinary software problems. Current agent guidance from OpenAI and Anthropic also recommends model-directed complexity only where workflows require it and evaluations justify it.

**Consequence:** Stage 0 is a deterministic baseline. The first agent is read-only, typed, budgeted, and evidence-bearing. The recommended architecture never requires model availability to keep the business process operable.

### Finding 2: procurement regimes cannot be blended into one prompt

OECD, UNCITRAL, WTO, EU, World Bank, FAR, and UK guidance share themes—fairness, competition, integrity, pre-defined criteria, records, accountable decisions—but differ in scope and detail. Public rules do not automatically apply to private sourcing; private policy does not satisfy public law.

**Consequence:** Every case binds an effective-dated `regime_profile` and policy release selected from canonical organization facts. Legal/procurement owners validate applicability. The model neither selects nor interprets the governing regime.

### Finding 3: requisition and category ambiguity is a bounded model opportunity

Procurement classifications describe different concepts. UNSPSC/CPV can classify goods/services, while NAICS classifies supplier industries. Internal taxonomies often drive category ownership and controls. A classifier can surface candidates and questions but may conceal multi-category demand or threshold avoidance.

**Consequence:** The model produces cited alternatives. A category steward confirms the authoritative category, related-demand aggregation runs deterministically, and the policy service calculates route/threshold obligations.

### Finding 4: supplier discovery and qualification are separate

Open access and competition can be harmed if a discovery rank becomes a qualification decision. Search sources have variable coverage and may reproduce incumbent or geographic bias.

**Consequence:** An approved discovery strategy fixes market scope, sources, requirements, prohibited features, and budget. Search ranking is not quality. Procurement approves the longlist and records gaps.

### Finding 5: supplier identity is a graph with uncertainty

GLEIF and BODS demonstrate the need for durable entity and relationship identifiers, time periods, and reporting exceptions. Procurement-suite sites, internal vendor IDs, LEIs, registries, and trading names do not map one-to-one.

**Consequence:** Identity resolution preserves legal entity, site, alias, parent, ownership, and source identifiers separately. Fuzzy matches remain candidates; merges/splits and corrections retain provenance.

### Finding 6: due diligence is risk-, scope-, and source-specific

NIST SP 1326 supplies a current ICT due-diligence frame. DOJ/SEC guidance supports risk-based third-party review, business rationale, service/payment coherence, and ongoing monitoring. Official sanctions/debarment sources have specific jurisdiction/program consequences.

**Consequence:** Versioned profiles name required checks and decision owners. Every evidence snapshot records list/source release, scope, query/match method, time, expiry, result, and gap. The model cannot automatically clear or exclude a supplier.

### Finding 7: list matching must preserve ambiguity

OFAC explicitly describes fuzzy search. World Bank and SAM.gov entries have program-specific effects. Name-only similarity creates false positives; a clean result against one list creates false assurance.

**Consequence:** Match features and contradictions route to trained reviewers. `candidate_match`, `no_candidate`, `confirmed_match`, `cleared`, `unknown`, and `not_applicable` are different states.

### Finding 8: sealed bids and frozen criteria define the security architecture

WTO, EU, FAR, and other public guidance protect fair competition, confidential bid/source-selection information, and evaluation against announced criteria. Even private sourcing needs reproducible decision methods.

**Consequence:** Bid access is technically controlled by event, bidder, evaluator role, criterion/lot, state, purpose, and time. Criteria, weights, formulas, and normalization rules freeze before bid visibility; permitted amendments create a new version and communication trail.

### Finding 9: bid extraction can be probabilistic; normalization cannot

Heterogeneous bid schedules justify model-assisted extraction, but money/unit/time/tax/volume/option semantics require replayable code. FAR and World Bank guidance also show that low or unbalanced pricing needs contextual analysis and clarification rather than simplistic ranking.

**Consequence:** Immutable originals produce cited candidate facts; reviewers resolve uncertainty. Deterministic functions normalize accepted facts and keep raw, intermediate, formula, warning, and result records. Missing is never zero.

### Finding 10: official qualitative scores and award decisions require accountable human judgment

FAR 15.308 makes the source-selection authority's decision independent judgment in its scope. EU/WTO rules require pre-stated/verifiable criteria under their regimes. Model recommendations can create anchoring and obscure responsibility.

**Consequence:** The model supplies evidence maps, not official scores. Assigned evaluators record scores/rationales; conflict/SoD services validate participation. The award authority accepts, changes, rejects, or cancels a typed recommendation with its own rationale.

### Finding 11: integrity analytics produce leads, not verdicts

OECD 2025 and DOJ procurement-collusion material describe bid-rigging forms, facilitating markets, tender design, and red flags. The guidance calls for trained review and cooperation with authorized bodies.

**Consequence:** Detection preserves evidence and routes to competition/fraud/legal/compliance owners. The agent does not confront suppliers, change scores, exclude, or make an external report on its own.

### Finding 12: conflict and segregation controls must change access

EU Article 24, FAR parts 3/9.5, GAO internal-control guidance, and procurement practice treat conflicts and separation as operational controls, not disclosure prose.

**Consequence:** Event-specific declarations and detected candidates have a human disposition, expiry, recheck triggers, and direct access/assignment consequences. SoD evaluates the human behind multiple identities; the model can never fill an approver/evaluator role.

### Finding 13: procurement-suite APIs are effect surfaces, not safety guarantees

Current SAP, Oracle, and Coupa documentation confirms rich sourcing APIs but also product, version, permission, opt-in, and documentation-lifecycle caveats. SAP's explicit client-side access warning is especially important.

**Consequence:** The model never calls suite APIs directly. Versioned adapters implement canonical operations, application authorization, narrow credentials, exact approvals, evidence envelopes, rate/timeout budgets, effect identity, and remote reconciliation.

### Finding 14: durable workflow does not make external effects exactly once

Publishing, inviting, messaging, awarding, and handoff can commit even when a response is lost. Vendor idempotency guarantees vary and can change.

**Consequence:** Every effect is reserved with semantic identity and intent hash, then moves through `committing` and possibly `outcome_unknown`. Reconcile by authoritative remote state/correlation before retry. Compensation appends history.

### Finding 15: handoff must preserve category boundaries

OCDS spans tender, award, contract, and implementation data but is not an ownership model. UK commercial guidance connects sourcing to contract/value realization, yet legal, contract management, supply-chain, and finance responsibilities remain distinct.

**Consequence:** Procurement sends approved commercial facts and evidence to legal/onboarding/supply-chain owners through acknowledged typed handoffs. It does not write legal text or operate orders. Read-only post-award facts support outcome reconciliation only.

### Finding 16: “savings” requires a pre-approved baseline and later reconciliation

The UK Sourcing Playbook calls for benefit measurement/review; DOJ compliance guidance asks whether contracted work was performed and compensation was commensurate. Actual price, volume, mix, scope, FX, timing, leakage, and service outcomes can diverge.

**Consequence:** Modeled opportunity, evaluated value, contracted value, avoidance, realized cash/expense, service outcome, and unclassified variance remain separate. Deterministic variance decomposition and accountable finance/business review precede a realized claim.

### Finding 17: procurement memory is especially easy to poison

Raw bids, allegations, conflicts, sanctions candidates, and evaluator notes are event-sensitive. Automatic recall can disclose a competitor or create an unappealable hidden supplier penalty.

**Consequence:** Turn and working memory are ephemeral/typed; durable case state is authoritative; domain memory is stewarded/versioned; long-term supplier outcomes are curated, scoped, expiring, correctable, and current-source subordinate; episodic cases feed evaluation only after governance. Raw cross-event vector memory is rejected.

### Finding 18: evaluation must grade the environment and independence

Current Anthropic agent-eval guidance distinguishes trajectories and environment outcomes and recommends mixed graders/repeated trials. NIST documents solution contamination and grader gaming. Procurement success cannot be assessed from fluent prose.

**Consequence:** A synthetic sourcing estate grades sealed access, event versions, normalization, conflicts, approvals, remote effects, handoffs, and outcomes. Zero-tolerance slices cover competitor/cross-tenant leakage, criteria mutation, conflicted scoring, and unapproved award.

### Finding 19: capability qualification is tenant-, state-, role-, and operation-specific

An API name or product logo does not prove bid isolation, completeness, permission, idempotency, or audit behavior. SAP Ariba’s 2605 Event Management API exposes reveal, event change, message, scenario and award surfaces while explicitly placing intended API-user access control on the client. Oracle 26B resources depend on quarterly release, enabled features and privileges. Coupa lists R44 schemas while its indexed page warns that it is legacy/unmaintained.

**Consequence:** Every canonical operation has a typed capability manifest, exact provider/schema/configuration and state mapping, qualification report, expiry, owner and conformance suite. Read, reveal, publish, message, award and handoff never share one generic capability.

### Finding 20: source semantics need typed identities and evidence envelopes

Supplier entities/sites, events/rounds, bids/revisions, lots, criteria, official scores, money facts, effects and handoffs have different semantic identities. Provider states such as `approved`, `awarded`, `received`, `matched`, or `paid` are source assertions, not interchangeable domain conclusions.

**Consequence:** Provider envelopes retain native state, mapping release, source/resource identity, observation time, pagination/completeness, freshness, security decision, raw digest and limitations. Corrections create linked versions; entity, bid, score, award, handoff and outcome are never overwritten or collapsed.

### Finding 21: sealed-bid isolation extends beyond the procurement UI

The seal can be defeated by an overbroad API identity, support export, shared cache, provider conversation, telemetry, shadow replay or evaluation dataset even when the normal user interface appears correct.

**Consequence:** Source-enforced sealing and event/bid/evaluator authorization apply end to end. Criteria, formulas, roles and communication rules freeze before protected visibility; reveal is an independently approved effect; individual evaluation is bid/lot/criterion isolated; comparison is a later distinct capability; real-person conflict/SoD checks bind official scores and award.

### Finding 22: current list, ownership, OCR, warehouse and MCP sources have narrow guarantees

OFAC SLS provides official U.S. list data but not entity resolution or applicable legal disposition. SAM.gov v4 documents result ceilings and posted an active Exclusions API change notice on 2026-08-11. GLEIF supports fuzzy and relationship searches but reports relationship exceptions. BODS remains 0.4 with future changes anticipated. OCR processors have model/version/region/size/quota limits. Warehouses are derived views with load/correction gaps. MCP 2025-11-25 standardizes protocol operations, not procurement semantics; its tasks are experimental.

**Consequence:** Pin source release/query/digest/coverage and limitations, preserve ambiguous matches, cite original documents, qualify derived data lineage, and use MCP only as a narrower approved transport behind the same provider/effect contracts.

### Finding 23: all seven memory lifetimes need explicit procurement decisions

The runtime names exactly turn/scratch, working/run, session, durable engagement/task, domain, long-term and episodic/outcome lifetimes. Raw bid/vector recall, caches and retrieval indexes are not additional authoritative lifetimes.

**Consequence:** Durable/domain services own truth; turn/working/session are bounded and reconstructable; long-term supplier outcomes are optional, expiring, disputable and current-source subordinate; episodic records enter governed evaluation only. Loss-aware compaction receipts preserve bids, money, conflicts, approvals, contradictions, limitations and unknown effects or block consequential resume.

### Finding 24: recovery load and behavior evolution are part of procurement integrity

After a cell outage, reconciliation, approaching event deadlines, expiring due diligence and qualified evaluator/award-owner capacity compete with new arrivals. Restoring a database does not prove that sealed bids, keys, effects, deadlines or fairness survive.

**Consequence:** Recovery plans use net drain rate and protected queue order. DR drills combine source outage, unknown award, partial key recovery, expired approval, evaluator absence and production-shaped backlog. The full behavior bundle—regime, policy, taxonomy, identity, due diligence, criteria, SoD, money, connectors, documents, context/compactor, model/prompt, handoffs, outcomes, renderer and evals—passes replay, shadow, canary, drift, rollback/forward recovery and controlled failure mining.

## Architecture alternatives and conditional recommendation

| Alternative | Evidence-based position |
| --- | --- |
| Deterministic sourcing workflow | First choice for structured inputs, known suppliers/routes, and stable scorecards |
| Custom read-only loop | Best first agent for one short ambiguity/evidence task |
| Agent SDK application | Credible when maintained loop/tool/trace primitives help; application retains all controls/state |
| Durable workflow plus bounded model worker | **Recommended production shape** for long waits, bids, evaluators, approvals, and effects |
| Procurement-suite native extension | Reuse native workflow, roles, seals, and audit where adequate; verify APIs and do not duplicate the source of truth |
| Multi-agent organizational mirror | Rejected by default; expands access and nondeterministic handoffs without a demonstrated control benefit |
| Browser/RPA on legacy UI | Temporary supervised bridge only; weak state/reconciliation makes it unsuitable for sealed/award authority |
| Organization-approved MCP server/tool | Conditional only when narrower than direct API credentials; protocol/tool/auth pinning does not replace procurement qualification, bid isolation or effect reconciliation |

Language follows the existing enterprise control-plane stack. TypeScript/Node, JVM, and .NET are credible orchestration choices; Python is especially useful for bounded extraction/analytics/eval workers; Go fits adapters. No researched source supports one universal procurement-agent language or model.

## Material tensions and resolved positions

### Transparency versus bid confidentiality

Public regimes can require notices, award information, reasons, and traceability while also protecting competition and confidential commercial information.

**Resolved:** generate explicit public/debrief views from disclosure policy. Internal bid/evaluation artifacts never become publishable merely because the event requires transparency.

### Open supplier access versus qualification and risk

Broad discovery supports competition; qualification and exclusions protect delivery and integrity. Overbroad screens can reduce competition or encode bias.

**Resolved:** approve discovery strategy separately, use proportionate published/approved participation rules, record gaps, and keep evidence candidates separate from disposition.

### Best value versus lowest price

FAR, EU, World Bank, and UK sources support price plus quality/rated criteria in their scope; price-only methods remain valid for some well-specified cases.

**Resolved:** the adopted event method governs. Do not hardcode “lowest wins” or “AI chooses best value.” Predefine and replay the method.

### Evolving requirements versus criteria freeze

Early market engagement can improve requirements and evaluation models, while post-visibility change can prejudice suppliers.

**Resolved:** iterate before the event/freeze point; afterward use only authorized amendment, equal communication, versioning, time adjustment, and remedy rules.

### Model assistance versus evaluator independence

Evidence mapping can reduce effort, but a recommendation can anchor evaluators.

**Resolved:** model output is cited and non-authoritative; use evidence-first/blinded interfaces where material; official scores and award judgment remain human records.

### Fast direct/emergency sourcing versus competition

Urgency may justify a different route but can also be used to bypass control.

**Resolved:** deterministic eligibility, enhanced reason/authority, bounded duration/scope, competition where feasible, and retrospective review. The model cannot declare an emergency.

### Current policy versus historical reproducibility

Replaying old policy explains a decision; current policy may revoke unsafe authority.

**Resolved:** retain historical policy/behavior for audit and recomputation, but enforce current revocations at commit. Later loosening requires a new grant.

### Provider score/list versus accountable due diligence

Commercial scores and fuzzy list searches are efficient but can be opaque, stale, or wrong.

**Resolved:** use them as source-scoped evidence with entity mapping, provenance, freshness, correction, and human disposition. Reject unexplained scores for consequential decisions.

### Workflow replay versus external exactly once

Durable runtimes can avoid repeating recorded steps internally, but a network timeout after vendor commit remains ambiguous.

**Resolved:** semantic effect identity, downstream contract, intent equivalence, postcondition, fencing, `outcome_unknown`, and reconciliation are mandatory.

### Full trace capture versus procurement privacy

Raw bids and prompts help debugging but create a second confidential source-selection system.

**Resolved:** structured references and redacted diagnostics by default; protected content evidence stays in the audit/evidence plane with separate access and retention.

### Historical supplier learning versus fairness and freshness

Past performance and realized outcomes can improve sourcing; disputed or irrelevant history can create hidden exclusion.

**Resolved:** curated, scoped, time-bounded, correctable evidence with a named owner. No automatic model-written reputation memory and no current-source override.

## Workload-specific decision records

| Decision | Claim class | Evidence boundary | Blueprint consequence | Refresh trigger |
| --- | --- | --- | --- | --- |
| Hybrid durable workflow + bounded model | Recommendation | Shared agent guidance plus procurement lifecycle; no comparative production benchmark proves universal superiority | Default architecture, deterministic alternative retained | New suite-native capability or measured simpler design |
| Autonomous ceiling P1 by default | Recommendation | High consequence/confidentiality and accountable-regime evidence | Models propose; P3 is exact human-approved adapter effect only | Reliable evidence and policy for a new narrow effect |
| Criteria/normalization freeze | Mechanic/recommendation | Strong across WTO/EU/FAR/UK/World Bank within scopes | Version event and invalidate dependent approvals | Regime or event-method change |
| Human official scores and award | Recommendation/mechanic in some regimes | FAR independent judgment is U.S.-specific; wider accountability principle cross-checked | Model evidence maps, not official score/decision | New regulation or validated organization policy |
| Fuzzy list match is candidate only | Mechanic/recommendation | OFAC states fuzzy search; resolution procedure is organization-specific | Explicit ambiguous states and trained review | Official list/matching guidance change |
| No raw bid long-term/vector memory | Security recommendation | Confidentiality and poisoning risks; no source proves safe general reuse | Event-scoped storage and curated derived outcomes only | Approved legal/data/eval design with equivalent isolation |
| Multi-agent rejected by default | Recommendation | No procurement evidence showed benefit exceeding access/handoff risk | One coordinator; limited read-only workers only after measurement | Scale/eval proof of bounded benefit |
| Post-award actuals read-only | Category boundary decision | Registry separation and commercial lifecycle sources | Outcome reconciliation without order/contract operation | Category registry change |

## Integration-specific adoption tests

| Integration | Tests required before use |
| --- | --- |
| Procurement/e-sourcing suite | Native seal/opening roles, API principal permissions, event versioning, audit completeness, idempotency/correlation, timeouts after commit, pagination, rate limits, schema/feature changes |
| ERP/spend | Legal entity/tenant filters, period/currency/correction semantics, completeness, aggregate authorization, duplicate demand |
| Supplier master/onboarding | Canonical entity/site mapping, duplicate check, SoD, bank-data exclusion, acknowledgement and defect correction |
| Sanctions/debarment/registry | Exact release/scope, alias/transliteration, rate/outage, ambiguous match, historical reconstruction, correction |
| Commercial supplier risk | Source coverage, score explanation, entity mapping, licensing, dispute/correction, model/schema drift |
| CLM/legal workspace | Approved template/header only, no clause authority, exact payload digest, acknowledgement, duplicate/partial create |
| P2P/performance actuals | Read-only scope, revisions/corrections, comparable dimensions, no order/shipment effect |
| Email/chat | Authenticated destination, payload digest, delivery status; explicit rejection as approval channel |

## Claims deliberately excluded

- No claim that AI improves procurement savings, fairness, speed, supplier diversity, or compliance without target-organization evaluation.
- No claim that any public-procurement source universally binds private organizations or all jurisdictions.
- No universal numeric procurement threshold, approval level, retention period, sanctions match score, model confidence, SLO, RTO/RPO, or autonomy target.
- No claim that an OCDS/UBL-compliant record is legally sufficient, semantically complete, or operationally authoritative.
- No claim that OFAC/SAM/World Bank/GLEIF/BODS or a commercial provider supplies complete global supplier due diligence.
- No claim that an API's OAuth, native approval, sealed-bid feature, audit log, `2xx`, or idempotency hint supplies end-to-end authorization/effect correctness.
- No autonomous legal interpretation, supplier exclusion, collusion verdict, qualitative score, award, contract language, order, payment, or shipment action.
- No “exactly once” guarantee across arbitrary external sourcing/onboarding/CLM systems.
- No raw bid/provider trace capture or cross-event training/memory recommendation.
- No multi-agent architecture claim based on organizational roles.

## Derived guide set

| Guide | Independent maintenance question |
| --- | --- |
| [README](../../agents/procurement-sourcing-agent/README.md) | Does the category, architecture, risk, and reader path fit? |
| [Boundaries, workload fit, authority, and stages](../../agents/procurement-sourcing-agent/01-boundaries-workload-fit-authority-and-stages.md) | When should ordinary software stop and agent authority begin? |
| [Reference architecture, runtime, and integrations](../../agents/procurement-sourcing-agent/02-reference-architecture-runtime-and-integrations.md) | Which runtime and adapter contracts preserve control across real suites? |
| [Requisition, spend, category, and policy evidence](../../agents/procurement-sourcing-agent/03-requisition-spend-category-and-policy-evidence.md) | How does demand become an authorized strategy? |
| [Supplier discovery, identity, and due diligence](../../agents/procurement-sourcing-agent/04-supplier-discovery-identity-and-due-diligence.md) | How are discovery, entity identity, evidence, and disposition separated? |
| [RFx, bid normalization, evaluation, and award](../../agents/procurement-sourcing-agent/05-rfx-bid-normalization-evaluation-and-award.md) | How is a fair, reproducible event and comparison built? |
| [Conflicts, approvals, security, and privacy](../../agents/procurement-sourcing-agent/06-conflicts-approvals-security-and-privacy.md) | How are independence, confidentiality, and exact authority enforced? |
| [State, context, memory, planning, and reliable effects](../../agents/procurement-sourcing-agent/07-state-context-memory-planning-and-reliable-effects.md) | How does long-running work survive compaction, replay, concurrency, and ambiguous effects? |
| [Handoffs and realized-outcome reconciliation](../../agents/procurement-sourcing-agent/08-handoffs-and-realized-outcome-reconciliation.md) | How does procurement exit without losing commercial outcome evidence? |
| [Observability, evaluation, failure injection, and incidents](../../agents/procurement-sourcing-agent/09-observability-evaluation-failure-injection-and-incidents.md) | What proof blocks release and supports operations? |
| [Deployment, capacity, cost, and governed evolution](../../agents/procurement-sourcing-agent/10-deployment-capacity-cost-and-governed-evolution.md) | How is the system deployed, bounded, recovered, changed, and retired? |
| [Adapter qualification and worked sourcing lifecycle](../../agents/procurement-sourcing-agent/11-adapter-qualification-and-worked-sourcing-lifecycle.md) | How are source/provider capabilities qualified and how does one case move from requisition through realized outcome? |

## Refresh triggers

Review affected decisions immediately when:

- a jurisdiction, procurement regime, donor/funder rule, organization policy, threshold, delegation, conflict, exclusion, notice, standstill/challenge, or records duty changes;
- OECD, UNCITRAL, WTO GPA, EU, World Bank, FAR, UK commercial guidance, GAO, NIST, FATF, or another adopted source materially changes;
- OFAC/SAM/World Bank/GLEIF/BODS/registry/list delivery, schema, search/matching guidance, identifiers, result ceilings, cadence, active change notice, or licensing changes;
- SAP Ariba, Oracle, Coupa, the selected procurement suite, ERP, supplier master, risk provider, CLM, approval service, or P2P connector changes permissions, schemas, versions, quotas, webhooks, seal/opening, approvals, or effect behavior;
- model/provider tool calling, structured output, data retention/training, caching, residency, background state, or safety behavior changes;
- the system adds a P3 effect, broader supplier/event selector, new tenant/region, browser/RPA, third-party plugin/MCP server, multi-agent delegation, or long-term memory;
- MCP publishes a new core specification/extension or changes authorization, task, transport, tool-schema, token or discovery behavior used by a deployed client/server;
- an incident/near miss involves bid leakage, early opening, criteria drift, conflict, wrong supplier, false list match, unapproved award, unknown effect, handoff defect, outcome misstatement, cross-tenant access, retention, or evaluator anchoring;
- an eval grader is gamed/contaminated, a critical slice becomes underpowered, or production corrections diverge from offline results;
- 90 days pass for provider/integration/security details or 180 days for workload architecture without review.

## Research limitations

1. Procurement law is jurisdiction-, entity-, value-, funding-, category-, and procedure-specific. This packet is an engineering synthesis, not legal advice.
2. Many enterprise procurement-suite APIs require subscriptions, tenant access, support portals, feature flags, or contractual terms unavailable publicly. Public docs establish surface existence and caveats, not exact target-tenant behavior.
3. Commercial supplier-risk, financial, adverse-media, cyber, ESG, and beneficial-ownership providers differ widely; no one product was endorsed or deeply benchmarked.
4. No public production evaluation was found that proves autonomous sourcing awards are safe or economically superior. The blueprint therefore keeps award authority human-owned.
5. Public sources emphasize formal procurement; private strategic sourcing may use negotiation and policies not represented here. The regime-profile mechanism is intentionally organization-specific.
6. OCDS is primarily a publishing/data standard, UBL is an exchange-document standard, and neither replaces the internal authoritative state/effect model.
7. Costs, quotas, model availability, and vendor pricing were intentionally not frozen; deployment teams must verify them at decision time.
8. The blueprint does not specify sector-specific safety, defense, healthcare, construction, labor, environmental, modern-slavery, data-sovereignty, export-control, or regulated-financial requirements. Add them through a reviewed profile.
9. Realized-outcome attribution can remain uncertain even with strong data. The design preserves `unclassified` rather than forcing a savings claim.

## Primary and authoritative source register

### Procurement principles, law, and current public guidance

- [OECD Recommendation of the Council on Public Procurement, OECD/LEGAL/0411](https://legalinstruments.oecd.org/en/instruments/OECD-LEGAL-0411)
- [OECD 2025 implementation report, 2020–2024 overview](https://www.oecd.org/en/publications/implementing-the-oecd-recommendation-on-public-procurement-in-oecd-and-partner-countries_02a46a58-en/full-report/overview_fab2711c.html)
- [UNCITRAL Model Law on Public Procurement (2011)](https://uncitral.un.org/en/texts/procurement/modellaw/public_procurement)
- [WTO Agreement on Government Procurement, revised text](https://www.wto.org/english/docs_e/legal_e/rev-gpr-94_01_e.htm)
- [EU Directive 2014/24/EU](https://eur-lex.europa.eu/eli/dir/2014/24/oj/eng)
- [World Bank Project Procurement Framework](https://www.worldbank.org/ext/en/what-we-do/project-procurement/framework)
- [World Bank IPF Procurement Regulations, seventh edition, September 2025](https://thedocs.worldbank.org/en/doc/c84273d1b230aeb2b0b8134de5dc8cd7-0290012025/original/Procurement-Regulations-7th-Edition-Sep-2025.pdf)
- [World Bank guidance: Abnormally Low Bids and Proposals](https://documents.worldbank.org/en/publication/documents-reports/documentdetail/099812008202522046)
- [U.S. FAR Subpart 15.3 — Source Selection](https://www.acquisition.gov/far/subpart-15.3)
- [U.S. FAR 15.404-1 — Proposal Analysis Techniques](https://www.acquisition.gov/far/15.404-1)
- [U.S. FAR Part 3 — Improper Business Practices and Personal Conflicts](https://www.acquisition.gov/far/part-3)
- [U.S. FAR Subpart 9.5 — Organizational and Consultant Conflicts](https://www.acquisition.gov/far/subpart-9.5)
- [U.S. FAR Subpart 42.15 — Contractor Performance Information](https://www.acquisition.gov/far/subpart-42.15)
- [UK Government Functional Standard GovS 008 Commercial, version 2.2](https://www.gov.uk/government/publications/government-functional-standard-govs-008-commercial-and-commercial-continuous-improvement-assessment-framework/government-functional-standard-govs-008-commercial-html)
- [UK Sourcing Playbook, updated 2026-06-15](https://www.gov.uk/government/publications/the-sourcing-and-consultancy-playbooks/the-sourcing-playbook-html)
- [UK Bid Evaluation guidance](https://www.procurementpathway.civilservice.gov.uk/documents/best-practice/bid-evaluation-sourcing-playbook)
- [UK guidance on supplier economic and financial standing, updated 2026-06-15](https://www.gov.uk/government/publications/the-sourcing-and-consultancy-playbooks/assessing-and-monitoring-the-economic-and-financial-standing-of-suppliers-guidance-note-html--2)
- [GAO 2025 Green Book](https://www.gao.gov/greenbook)

### Integrity, competition, and third-party governance

- [OECD Guidelines for Fighting Bid Rigging in Public Procurement, 2025 update](https://www.oecd.org/en/publications/oecd-guidelines-for-fighting-bid-rigging-in-public-procurement-2025-update_cbe05a56-en.html)
- [DOJ Procurement Collusion Strike Force](https://www.justice.gov/atr/procurement-collusion-strike-force)
- [DOJ Red Flags of Collusion](https://www.justice.gov/atr/red-flags-collusion)
- [DOJ Evaluation of Corporate Compliance Programs, updated September 2024](https://www.justice.gov/criminal/criminal-fraud/page/file/937501)
- [DOJ/SEC FCPA Resource Guide, second edition](https://www.justice.gov/criminal/criminal-fraud/fcpa-resource-guide)
- [UK Bribery Act 2010 guidance](https://assets.publishing.service.gov.uk/media/5d80cfc3ed915d51e9aff85a/bribery-act-2010-guidance.pdf)
- [OECD Due Diligence for Responsible Business Conduct](https://www.oecd.org/en/topics/sub-issues/due-diligence-guidance-for-responsible-business-conduct.html)

### Supplier identity, restrictions, and supply-chain risk

- [NIST SP 800-161 Rev. 1 update 1](https://csrc.nist.gov/pubs/sp/800/161/r1/upd1/final)
- [NIST SP 1326: Cybersecurity Supply Chain Risk Management Due Diligence Assessment Quick-Start Guide](https://csrc.nist.gov/pubs/sp/1326/final)
- [NIST SP 800-53 Rev. 5.1 downloads and Supply Chain Risk Management family](https://csrc.nist.gov/Projects/risk-management/sp800-53-controls/downloads)
- [OFAC Sanctions List Service](https://ofac.treasury.gov/sanctions-list-service)
- [SAM.gov Exclusions](https://sam.gov/content/exclusions)
- [SAM.gov Exclusions API v4](https://open.gsa.gov/api/exclusions-api/)
- [SAM.gov active Exclusions API change notice, 2026-08-11](https://sam.gov/announcements/exclusions-api-update)
- [World Bank Listing of Ineligible Firms and Individuals](https://documents.worldbank.org/en/projects-operations/procurement/debarred-firms)
- [GLEIF Level 2 relationship reporting exceptions](https://www.gleif.org/en/lei-data/access-and-use-lei-data/level-2-data-reporting-exceptions-2-1-format)
- [GLEIF API](https://www.gleif.org/en/lei-data/gleif-api)
- [FATF Guidance on Beneficial Ownership of Legal Persons](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Guidance-Beneficial-Ownership-Legal-Persons.html)
- [Beneficial Ownership Data Standard 0.4](https://standard.openownership.org/en/0.4.0/about/)

### Procurement data, taxonomy, and event standards

- [Open Contracting Data Standard 1.1.5 release reference](https://standard.open-contracting.org/latest/en/schema/reference/)
- [Open Contracting Data Standard contracting-process primer](https://standard.open-contracting.org/latest/en/primer/how/)
- [OASIS Universal Business Language 2.4](https://docs.oasis-open.org/ubl/UBL-2.4.html)
- [CloudEvents 1.0.2](https://cloudevents.io/)
- [UN Global Marketplace UNSPSC](https://www.ungm.org/Public/UNSPSC)
- [EU Common Procurement Vocabulary](https://ted.europa.eu/en/simap/cpv)
- [EU eForms](https://single-market-economy.ec.europa.eu/single-market/public-procurement/digital-procurement/eforms_en)

### Current procurement-platform documentation

- [SAP Ariba Event Management API](https://help.sap.com/docs/ariba-apis/event-management-api/event-management-api)
- [SAP Ariba API catalog](https://help.sap.com/docs/ARIBA_APIS)
- [Oracle Fusion Cloud Procurement 26B APIs](https://docs.oracle.com/en/cloud/saas/procurement/26b/api.html)
- [Oracle 26B sourcing REST integration guidance](https://docs.oracle.com/en/cloud/saas/procurement/26b/fainp/src-rest-apis-inbound.html)
- [Oracle 26B supplier-site REST endpoints](https://docs.oracle.com/en/cloud/saas/procurement/26b/fapra/api-suppliers-sites.html)
- [Coupa Core API and current schema downloads](https://compass.coupa.com/en-us/products/product-documentation/integration-technical-documentation/core-api-and-csv-download-formats)
- [Coupa Core API](https://compass.coupa.com/en-us/products/product-documentation/integration-technical-documentation/the-coupa-core-api)
- [Coupa legacy sourcing API page and maintenance warning](https://compass.coupa.com/en-us/products/product-documentation/integration-technical-documentation/the-coupa-core-api/resources/transactional-resources/sourcing-api-%28quote_requests%29)
- [Google Document AI limits](https://cloud.google.com/document-ai/limits)
- [BigQuery time travel](https://cloud.google.com/bigquery/docs/time-travel)
- [Snowflake ACCESS_HISTORY](https://docs.snowflake.com/en/sql-reference/account-usage/access_history)
- [MCP 2025-11-25 authorization](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization)
- [MCP 2025-11-25 tasks](https://modelcontextprotocol.io/specification/2025-11-25/basic/utilities/tasks)

### Agent engineering, evaluation, and operations

- [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [Anthropic: Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- [OpenAI: A practical guide to building agents](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- [NIST AI 600-1: Generative AI Profile](https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence)
- [NIST: Cheating on AI Agent Evaluations](https://www.nist.gov/caisi/cheating-ai-agent-evaluations)
- [NIST SP 800-61 Rev. 3: Incident Response](https://csrc.nist.gov/pubs/sp/800/61/r3/final)
- [W3C Trace Context](https://www.w3.org/TR/trace-context/)
- [OpenTelemetry Generative AI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)

## Quality statement

The resulting blueprint keeps the registry boundary, begins with a deterministic alternative, supplies a bounded first loop and a complete Stage 0–6 path, and specializes every required cross-cutting decision for procurement. Pass 2 adds source-specific capability qualification, provider envelopes and identity contracts, end-to-end sealed-bid/evaluator isolation, exact seven memory lifetimes and loss-aware continuity, a worked requisition-to-outcome case, richer adapter/effect/handoff failure suites, trace/log/audit distinctions, recovery-load/DR gates and full behavior-bundle evolution. Material contradictions are resolved explicitly and product behavior is date/version bounded.

Remaining work is organizational adoption: counsel and procurement policy must select the actual regime; data owners must approve supplier/bid processing; target procurement suites and data providers must pass contract tests; domain experts must create realistic fixtures and thresholds; and operators must prove capacity, recovery, and incident controls. Until those gates pass, the system should remain at P0/P1 and this material should retain **Research-backed draft** maturity.

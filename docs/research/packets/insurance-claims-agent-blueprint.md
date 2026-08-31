# Insurance Claims Operations Agent Research Packet

> **Research completed:** 2026-08-31  
> **Guide:** [Production Insurance Claims Operations Agent Blueprint](../../agents/insurance-claims-agent/README.md)  
> **Maturity:** Pass 1 production architecture baseline; not legal advice and not a jurisdiction/product implementation specification  
> **Research objective:** Define the safest practical agent boundary for policy/claim identity, FNOL, evidence, communications, assessment support, reserve/adjudication recommendations, human decisions, external effects, fraud/legal/finance boundaries, regulatory timelines, catastrophe surge, and production operations.

## Executive synthesis

The strongest sources converge on a control model rather than a fully autonomous claims adjuster:

- claim handling must be timely, fair, evidence-based, reconstructable, and explainable;
- the insurer remains accountable when claims work or AI components are outsourced;
- exact rules vary materially by jurisdiction, product, claimant type, event, and effective date;
- policy terms and claim decisions need exact source/version evidence;
- qualified people and independent organizational functions retain consequential authority;
- external financial, communication, vendor, recovery, and reporting effects need narrow authorization and verifiable receipts;
- AI/model governance must cover data, validation, third parties, documentation, fairness, security, and change;
- catastrophe operations require more capacity and scoped legal overlays, not weaker controls.

The selected design is therefore a **durable claim-operations coordinator with bounded model workers**. The model extracts, compares, summarizes, drafts, and recommends. Deterministic services own identity, rules, calculations, clocks, permissions, workflow, approvals, effects, and reconciliation. Authorized adjusters/examiners make coverage, liability, valuation, settlement, and closure decisions. SIU/fraud, legal, finance/payment, actuarial/accounting, recovery, vendor-management, and Document Intelligence systems keep their own records and authority.

## Questions researched

1. Which claim-handling duties and recordkeeping expectations matter to the architecture?
2. How much do timelines vary, including catastrophe orders and calendar definitions?
3. What policy/claim/version evidence is required before coverage assistance?
4. Which tasks can a model usefully perform without becoming an unauthorized decision-maker?
5. How should damage assessment, reserves, adjudication, payments, subrogation, vendor work, fraud, and legal matters be separated?
6. Which carrier and reporting integrations have useful current standards or concrete API semantics?
7. What AI, privacy, data-security, audit, and third-party governance constraints apply?
8. Which state, event, idempotency, reconciliation, context, memory, evaluation, deployment, CAT, cost, SLO, incident, and release controls are required for production?

## Research method and selection criteria

The research prioritized primary and current sources:

1. regulator statutes/models/guidance and official handbooks;
2. international supervisory standards;
3. government program claims/reporting manuals;
4. official standards organizations;
5. official carrier-platform documentation as implementation examples;
6. repository canonical engineering guides for runtime, context, security, effects, evaluation, and operations.

Important claims were cross-checked across more than one source category. Model laws and regulations were treated as models unless local adoption was verified. Vendor behavior was treated as version-specific evidence, never a universal insurance contract. Public sources were used to derive architecture invariants rather than to copy procedural language.

Research stopped when additional searches mostly repeated the same control boundaries or moved into line/jurisdiction-specific implementation details that must be supplied by the deploying carrier.

## Regulatory and supervisory findings

### Claims handling and claim-file reconstruction

The [NAIC Unfair Claims Settlement Practices Act, Model 900](https://content.naic.org/sites/default/files/model-law-900.pdf) identifies model concerns including misrepresentation of policy provisions, prompt acknowledgment, reasonable investigation/settlement standards, good-faith fair settlement when liability is reasonably clear, no refusal without reasonable investigation, timely affirmation/denial after investigation, coverage-specific payment explanation, avoidance of duplicate proof, and accurate explanations for denials or compromise offers.

The [NAIC Unfair Property/Casualty Claims Settlement Practices Model Regulation, Model 902](https://content.naic.org/sites/default/files/model-law-902.pdf) provides more concrete model record and process expectations:

- accessible/retrievable claim data;
- detailed documentation sufficient to reconstruct claim activity;
- dates received, processed, or mailed for relevant documents;
- acknowledgment, claimant-response, proof-of-loss disposition/status, limitation-notice, and payment model timeframes;
- written policy references and claim-file documentation for denial;
- product-specific auto/property settlement rules.

The model expressly excludes some lines in its own scope and is not adopted identically everywhere. The [NAIC state action page for Model 902](https://content.naic.org/sites/default/files/model-law-state-page-902.pdf) reinforces the need to validate local law rather than copy model numbers into production.

**Architecture decision:** application-owned structured claim/evidence/clock/decision/effect records must reconstruct handling. The model transcript and telemetry cannot be the claim file.

### International fairness and outsourcing baseline

The [IAIS Insurance Core Principles and ComFrame, adopted December 2024](https://www.iaisweb.org/uploads/2024/12/IAIS-ICPs-and-ComFrame-adopted-in-December-2024.pdf), particularly ICP 19, supports:

- timely, fair, transparent handling;
- written claims procedures from claim through settlement with expected timeframes;
- claimant information about procedures, status, and claim-determinative factors;
- competent, trained staff and technical/legal expertise;
- balanced, qualified dispute review with clear reasoning;
- close oversight and ultimate insurer responsibility for outsourced claims work;
- complaint handling and trend analysis.

The [FCA Insurance Conduct of Business Sourcebook, claims handling](https://handbook.fca.org.uk/handbook/icobs8) provides a current UK example requiring prompt/fair handling, reasonable guidance and status, no unreasonable rejection, and prompt settlement once agreed within its scope.

The FCA's [multi-firm review of vehicle valuations](https://www.fca.org.uk/publications/multi-firm-reviews/findings-multi-firm-review-insurers-valuation-vehicles), published 2024 and updated 2025, adds production findings: risks from settlement values below guide evidence, blanket deductions without case-specific justification, low initial offers that depend on a claimant challenging, inconsistent revaluation outcomes, insufficient outsourced-provider oversight, conflicts, and weak outcome monitoring. The review covered 12 firms representing an estimated 70% of that market; it remains UK motor-specific evidence, not a universal formula.

**Architecture decision:** third-party administrators, vendors, document/model providers, and automated routes do not transfer carrier accountability. Qualified human ownership and complaint/dispute paths must remain visible.

### Timeline variability

Sources demonstrate irreducible variation:

- NAIC Model 902 uses calendar days and contains model values such as 15-day acknowledgment/reply, 21-day proof-of-loss disposition/status initiation, 45-day continuing status, and 30-day payment after affirmation where amount is determined and undisputed.
- California's [2026 Guide for Adjusting Property Claims After a Major Disaster](https://www.insurance.ca.gov/0200-industry/0050-renew-license/0200-requirements/upload/2026-Guide-for-Adjusting-Property-Claims-in-California-After-a-Major-Disaster_Final.pdf) describes California-specific calendar-day duties, written bases, claim-file documentation, ongoing extension notices, payment timing, disaster notices, policy-copy duties, and temporary adjuster registration/training.
- New York's [property/casualty claims regulations page](https://www.dfs.ny.gov/apps_and_licensing/property_insurers/laws_regs_cls) points to 11 NYCRR Part 216 and different local semantics.
- The [Texas Hurricane Beryl prompt-payment order](https://www.tdi.texas.gov/orders/documents/20248743.pdf) extended defined statutory deadlines by 15 days for a named weather event and specified counties after finding local resources exceeded.

**Architecture decision:** no universal deadline constants. Use versioned obligation rules and instances containing jurisdiction, product, claimant, trigger, calendar/counting convention, time zone, pause/resume, exception, required action/content, source citation, effective dates, owner, and scoped emergency overlays. Preserve every recomputation.

### AI governance

The [NAIC Model Bulletin on the Use of AI Systems by Insurers](https://content.naic.org/sites/default/files/inline-files/2023-12-4%20Model%20Bulletin_Adopted_0.pdf), adopted by NAIC in December 2023, states expectations for insurer governance of AI-supported consumer-impacting actions under applicable insurance law. It covers governance, risk management, data, validation/testing, third-party systems, documentation, transparency, and information that a department may request. Claims management and fraud detection are within its stated insurance lifecycle scope.

The [NAIC AI topic page](https://content.naic.org/insurance-topics/artificial-intelligence) and [AI bulletin adoption map](https://content.naic.org/sites/default/files/legal-adoption-map-ai-model-bulletin.pdf) show ongoing 2025–2026 work and jurisdictional variation; as of the map dated 2026-08-06, adoption/guidance was not uniform.

**Architecture decision:** model/prompt/tool/rule/template/provider changes are governed behavior releases with task/slice evaluation, documentation, third-party evidence, shadow/canary, rollback, and affected-cohort query. Local applicability must be mapped.

### Fraud boundary

The [NAIC Insurance Fraud Prevention Model Act, Model 680](https://content.naic.org/sites/default/files/model-law-680.pdf) provides a model structure for suspected fraudulent insurance acts, mandatory reporting under its model threshold, confidentiality/privilege, qualified fraud units, investigation, and insurer antifraud initiatives.

**Architecture decision:** the claims agent can package factual, source-linked, minimum-necessary referrals. It does not declare fraud, investigate broad networks, file reports, reveal the referral, or use suspicion as an unapproved reason to delay/deny/reduce. SIU/fraud systems and qualified people own investigation and reporting.

### Privacy and security

The [NAIC Insurance Data Security Model Law, Model 668](https://content.naic.org/sites/default/files/model-law-668.pdf) establishes a model risk-based security program with risk assessment, testing/monitoring, audit trails, third-party oversight, incident response, and secure disposal. Local adoption varies.

The [NAIC Privacy of Consumer Financial and Health Information Regulation, Model 672](https://content.naic.org/sites/default/files/model-law-672.pdf) provides a model baseline for financial/health information disclosure and authorization. NAIC's [Privacy Protections Working Group](https://content.naic.org/committees/h/privacy-protections-wg) has active 2026 work to replace or revise privacy models.

**Architecture decision:** field- and compartment-level purpose authorization, provider data-use/residency/retention approval, restricted SIU/legal/health/payment lanes, no raw sensitive prompts/traces, end-to-end record inventory, legal-hold/deletion logic, and current local-law ownership are mandatory.

## Specialized claims and reporting findings

### NFIP example

The [FEMA NFIP Claims Manual, March 2025](https://www.fema.gov/sites/default/files/documents/fema_rsl_nfip-claims-manual_06032025.pdf) provides a specialized program example spanning notice, adjustment, proof, recommendation, payment, and review. It highlights the importance of declarations, policy forms, coverages, endorsements, limits, deductibles, insured location, policy term, and claim-file evidence. FEMA [claims review training](https://emilms.fema.gov/IS1104/groups/362.html) illustrates a role boundary in which the adjuster can assist with proof but cannot approve/disapprove the claim or promise approval.

**Decision:** bind policy term/version and retain adjuster/examiner/carrier decision ownership. Do not generalize NFIP rules to private or other products.

### Medicare Secondary Payer reporting

CMS's [Mandatory Insurer Reporting for NGHP](https://www.cms.gov/medicare/coordination-benefits-recovery/mandatory-insurer-reporting) explains Section 111 reporting for applicable liability insurance (including self-insurance), no-fault insurance, and workers' compensation arrangements and identifies the responsible reporting entity concept. CMS published [NGHP User Guide version 8.5 in July 2026](https://www.cms.gov/medicare/coordination-benefits-recovery/mandatory-insurer-reporting/whats-new) and updates code lists and reporting materials.

**Decision:** applicability, reportable events/amounts, fields, response-file handling, and corrections belong to a versioned specialist reporting route. The claims agent may stage evidence; it cannot infer reportability from a general prompt or replace the responsible reporting entity's process.

### Workers' compensation EDI

The [IAIABC Claims EDI standards page](https://www.iaiabc.org/edi-claims) identifies Release 3.1 as current and describes first/subsequent report transactions, jurisdictional profiles, event tables, element requirements, edit matrices, business scenarios, and XML/flat-file artifacts, with documents updated January 2026.

**Decision:** use the current jurisdiction profile and receipts/corrections. Never assume a shared “workers' compensation schema” is sufficient across jurisdictions.

## Policy, carrier, and external integration findings

### Policy retrieval and FNOL

Guidewire ClaimCenter Cloud API documentation provides a concrete current vendor example:

- [FNOL and adjudication, 2026.03](https://docs.guidewire.com/cloud/is/202603/cloudapibf/cloudAPI/ClaimCenter/fnol.html) exposes claim business flows;
- [graph-based policy retrieval, 2026.03](https://docs.guidewire.com/cloud/cc/202603/cloudapica/cloudAPI/topics/507-SpecificUseCases/05-policy-retrieval/c_graph_based_policy_retrieval.html) describes identifying a policy via PAS and copying a subset into ClaimCenter for adjudication;
- [lost updates and checksums, 2026.03](https://docs.guidewire.com/cloud/is/202603/cloudapibf/cloudAPI/Optimizing-calls/lost-updates-and-checksums.html) documents concurrency protection concepts.

**Decision:** preserve PAS/archive retrieval receipt and claim-system policy snapshot as distinct versions. Use optimistic concurrency/read-back; do not let last-write-wins overwrite a human update.

### Payments, reserves, and recoveries

Guidewire [creating checks, 2026.03](https://docs.guidewire.com/cloud/is/202603/cloudapibf/cloudAPI/ClaimCenter/financials/check-creating.html) distinguishes payment transactions, checks/payees, check sets, authority limits, and approval activities. Its [recoveries documentation, 2026.03](https://docs.guidewire.com/cloud/is/202603/cloudapibf/cloudAPI/ClaimCenter/financials/recoveries.html) distinguishes recovery reserves and realized recoveries.

The [NAIC Accounting Practices and Procedures Manual](https://content.naic.org/sites/default/files/publication-app-manual.pdf) and [ASOP No. 43](https://www.actuarialstandardsboard.org/asops/propertycasualty-unpaid-claim-estimates/) support separating claim operational reserve work from statutory accounting and aggregate actuarial estimates.

OFAC's [insurance-industry sanctions guidance](https://ofac.treasury.gov/faqs/topic/1616) shows that claims payments can require sanctions-specific blocking/licensing/legal analysis and that federal sanctions rules may conflict with state insurance payment expectations.

**Decision:** case-reserve recommendation, reserve transaction, payment intent/check/disbursement, recovery reserve/recovery receipt, general ledger, and actuarial estimates are distinct. A payment requires exact payee/amount/coverage/authority/lien/tax/sanctions/release/SoD gates plus reconciliation. The model never executes a high-impact payment.

### Standards and data exchange

The official [ACORD P&C standards catalog](https://www-dev.acord.org/standards-architecture/acord-data-standards/Property_Casualty_Data_Standards) identified P&C XML 2.13.0 (January 2025) and claim-related transactions at research time. The [IAIABC Claims EDI page](https://www.iaiabc.org/edi-claims) and CMS guides provide more specialized reporting structures. [X12 health-care transaction flows](https://x12.org/flow/health-care) cover claim, status, coordination-of-benefits/subrogation, and payment/remittance transactions where applicable; X12 published 008060 X322 for 835 payment/advice in January 2025, while regulatory adoption/profile use must be checked separately.

**Decision:** standards are integration vocabularies, not authoritative lifecycle, permission, idempotency, or reconciliation guarantees. Every adapter must document the exact selected standard/profile/vendor version and carrier mapping.

## Adapter and operations research refresh — 2026-08-31

Primary documentation was rechecked for the Pass 2 operation-level adapter guide. Public documentation establishes example object and transport behavior, but not a carrier tenant's configuration, permissions, data-processing contract, field extensions, live consistency, or fitness for a claims decision.

| Source/status inspected | Material finding | Contradiction or deployment limitation resolved |
| --- | --- | --- |
| Guidewire Cloud API 2026.03 ClaimCenter/PolicyCenter/BillingCenter | Claim exposure, financial, check/payee/approval, post-submission check, policy and billing objects have distinct lifecycles | Reject generic “update claim/pay claim” tools; qualify exact tenant operation and reconcile downstream state separately |
| Amazon Textract current API reference | Async analysis uses a job ID, optional client request token, declared feature/adapter version and later result retrieval; output can be paginated | Provider start idempotency does not prove complete evidence or business truth; retain source object version, page/rendition completeness and local field review |
| OGC STAC 1.1.0, STAC API 1.0.0 and OGC API Features 1.0.1 | Standards define metadata/assets/search and feature API conformance | Standards do not prove imagery accuracy, property identity, causation, license, freshness or carrier right to cache/use data |
| Google Geocoding API v4 preview, docs updated 2026-08-11 | Preview documentation reported 25 QPS and links to regional terms, security and storage/attribution policy | Do not encode preview quota or persist responses generically; preserve literal address, ambiguity, match precision, provider version and approved license pattern |
| NWS alerts web service/current documentation | CAP 1.2/JSON-LD alerts have issued/effective/expiry/geometry and service rate guidance | Alert is event evidence, not proof of damage/loss time/causation/coverage; US scope and service behavior require local fallback |
| FEMA OpenFEMA Disaster Declarations Summaries | Official declaration records include update metadata and documented historical/raw-data limitations | A declaration is not a universal claim-rule overlay or insured-loss fact; map exact geography/program/time and retain carrier owner |
| Twilio Messages resource/current docs | Queued, sent, delivered, failed and other states differ; callback properties vary by channel and may gain fields | Provider acceptance is not delivery/obligation proof; authenticate callbacks, tolerate additive fields and preserve channel-specific receipt semantics |
| OpenTelemetry specification 1.60.0 and semantic conventions 1.44.0 | Current transport/naming foundations have versioned stability boundaries | Telemetry cannot replace audit/business evidence; pin schema/collector/redaction/dashboard compatibility in the behavior release |
| SLSA 1.2 approved specification | Current source/build tracks and provenance vocabulary support artifact verification | Provenance does not prove a model, rule, dataset, mapping or claims outcome is correct; add domain provenance and qualified review |

**Decision:** the unit of qualification is `operation × provider/product release × carrier configuration × tenant/region × data class × volume × authority tier`. Public provider behavior is a starting hypothesis. Sandbox fault evidence, tenant observation, signed support scope, read-back/reconciliation, change owner and rollback are required before production authority.

## Production architecture decisions

| Question | Selected decision | Rejected alternative |
| --- | --- | --- |
| Top-level runtime | Durable workflow/case coordinator with stateless bounded model workers | Chat transcript or agent framework as control plane |
| Model role | Extract, compare, summarize, draft, recommend, abstain | Autonomous adjuster/examiner |
| Claim truth | Carrier systems plus versioned evidence/decision/effect records | Model memory or shadow claim copy |
| Coverage | Exact policy bundle and human decision | Policy-number retrieval plus generated conclusion |
| Deadlines | Versioned obligation registry/instances and scoped overlays | Universal numbers in prompt/code |
| Documents | Document Intelligence with immutable originals and field anchors | Direct PDF-to-prompt as evidence system |
| Assessment | Evidence comparison plus deterministic calculation and qualified review | Vision/model score as final damage/causation/value |
| Reserves | Case-level recommendation, exact approval/effect; finance/actuarial separate | Model posts or owns aggregate reserves |
| Fraud | Confidential factual referral to SIU/fraud | Model accusation, network investigation, or automatic adverse action |
| Legal | Detect and route with compartment segregation | Model legal advice/privilege decisions |
| Payment | Separate high-impact effect with independent controls and reconciliation | Generic claim-update or payment tool in model loop |
| Recovery/vendor | Typed referral/request with specialist/manager ownership | Autonomous demand, negotiation, vendor selection, invoice/payment |
| State | Separate claim, work, decision, and effect state | One natural-language status |
| Reliability | Semantic operation ID, unknown state, destination reconciliation, linked compensation | Blind retry or claimed exactly-once delivery |
| Memory | Purpose-limited context, structured durable state, governed domain knowledge; raw episodic memory rejected | Cross-claim vector memory and informal learned rules |
| Security | Brokered least privilege, compartments, no ambient model credentials | Shared claims service account |
| Evaluation | Harm-weighted end-to-end slice/fault/claim-outcome gates | Average model accuracy or handle-time target |
| Scale | Admission, deadline queues, degradation, CAT overlays, manual capacity | Widen authority or skip controls under surge |
| Change | Immutable behavior release across model/prompt/tool/schema/rule/template/adapter | “Configuration-only” untested changes |

## Resolved contradictions and trade-offs

### Faster automation versus fair human-owned adjudication

Claims operations need speed, especially after catastrophes, but final coverage/liability/settlement decisions require accountable qualified ownership. The design accelerates evidence and routing while keeping decision/effect records human-owned. It adds no general autonomous adverse-action path.

### Model guidance numbers versus local law

Model rules are valuable architectural inputs but unsafe production constants. Local laws, product exceptions, calendar definitions, and emergency orders conflict. The resolution is a source-cited, effective-dated obligation registry with compliance ownership and scoped overlays.

### Policy snapshot convenience versus contract accuracy

Claim systems may copy a useful policy subset from PAS, while the archived contract and transaction history are the stronger evidence for disputes. The design preserves both and blocks on mismatches rather than silently choosing.

### Catastrophe throughput versus control quality

Surge volume encourages shortcuts, while event-specific laws, temporary adjusters, vulnerable claimants, vendor scarcity, and duplicate/unknown effects increase risk. The resolution is reserved intake/deadline/reconciliation capacity and a degradation ladder that removes optional model work before controls.

### Fraud detection versus fair ordinary handling

Fraud reporting/investigation can require confidentiality, while ordinary claims still have prompt/fair duties. The design creates a restricted minimum-necessary referral lane and permits handling changes only through approved rules/SIU direction.

### Useful similar cases versus privacy, leakage, and biased precedent

Prior claims can suggest investigation steps but may expose sensitive data and encode different contracts, markets, or inconsistent outcomes. Raw episodic memory is rejected; curated de-identified examples are limited to governed evaluation/training or explicitly nonbinding retrieval.

### Provider confidence versus business correctness

OCR/model/vendor confidence scores are not calibrated probabilities and do not include policy, identity, or downstream harm. The design uses local field/task calibration, evidence validation, abstention, and qualified review.

### Standards versus effect semantics

ACORD, IAIABC, CMS, X12, and vendor APIs help mapping but do not guarantee carrier authority, stable idempotency, exactly-once effects, or reconciliation. The design wraps each profile in a qualified adapter contract.

### Case reserve efficiency versus finance/actuarial accountability

Models can surface evidence and suggest a claim-level estimate. Statutory accounting and aggregate actuarial estimates have different data, controls, and professional standards. The design keeps records and approvals separate.

### Exactly-once desire versus distributed-system reality

Carrier, provider, vendor, bank, communication, and regulator systems can time out after committing. The resolution is semantic operation identity, persisted intent, explicit `UNKNOWN`, authoritative read-back, reconciliation, and linked correction—not blind retry.

## Rejected designs

| Rejected design | Reason |
| --- | --- |
| One autonomous end-to-end claims agent | Conflates evidence, decision, authority, effects, and accountability |
| Multi-agent personas for adjuster, fraud, legal, finance, and vendor roles | Organizational boundaries become nondeterministic prompts with shared blast radius |
| Generic browser/RPA access to claims/payment portals | Broad authority, fragile targets, weak idempotency and reconciliation |
| Full claim file in every prompt | Privacy/privilege exposure, stale context, token loss, poor evidence weighting |
| Model-calculated deadlines | Jurisdiction/product/calendar/overlay errors cannot be controlled reliably |
| Model-generated claimant message sent directly | Unsupported promises/denials/rights language and recipient risk |
| Model score automatically sets reserve, denial, or fraud hold | Uncalibrated judgment becomes consequential action |
| Retry every 5xx/timeout | Duplicate notices, vendors, claims, reports, or payments |
| Close claim when model checklist says complete | Open effects/recoveries/holds and unauthorized terminal decision |
| Learn from every reviewer edit in production memory | Unreviewed policy drift, leakage, and inconsistent precedent |

## Source register

| Source | Version/date inspected | Strength | Use and caveat |
| --- | --- | --- | --- |
| [NAIC Model 900](https://content.naic.org/sites/default/files/model-law-900.pdf) | Model text January 1997 | Primary model act | Claims-practice principles; not universal local law and excludes specified lines in model scope |
| [NAIC Model 902](https://content.naic.org/sites/default/files/model-law-902.pdf) | Model text July 1997 | Primary model regulation | Record/timeline/process examples; local adoption differs |
| [NAIC Model 902 state action page](https://content.naic.org/sites/default/files/model-law-state-page-902.pdf) | Inspected 2026-08-31 | Primary NAIC adoption reference | Confirms local validation requirement |
| [NAIC AI Model Bulletin](https://content.naic.org/sites/default/files/inline-files/2023-12-4%20Model%20Bulletin_Adopted_0.pdf) | Adopted 2023-12-04 | Primary regulator model bulletin | AI governance baseline; jurisdiction adoption/guidance varies |
| [NAIC AI topic/adoption map](https://content.naic.org/insurance-topics/artificial-intelligence) | Current 2026; map 2026-08-06 | Primary current regulator status | Refresh source for AI work/adoption |
| [NAIC Model 668](https://content.naic.org/sites/default/files/model-law-668.pdf) | 2017 model; inspected 2026-08-31 | Primary model law | Security/third-party/incident baseline; local adoption differs |
| [NAIC Model 672](https://content.naic.org/sites/default/files/model-law-672.pdf) | 2017 model publication; inspected 2026-08-31 | Primary model regulation | Privacy baseline under active modernization |
| [NAIC Model 680](https://content.naic.org/sites/default/files/model-law-680.pdf) | Last model amendments 2003 | Primary model act | Fraud referral/confidentiality/unit boundary; local law differs |
| [NAIC MCAS](https://content.naic.org/insurance-topics/market-conduct-annual-statement) | Current 2025–2026 material | Primary regulator program | Operational metric vocabulary; not deadline law |
| [IAIS ICPs and ComFrame](https://www.iaisweb.org/uploads/2024/12/IAIS-ICPs-and-ComFrame-adopted-in-December-2024.pdf) | Adopted December 2024 | Primary international supervisory standard | Fair/transparent claims and outsourcing accountability baseline |
| [FCA ICOBS 8](https://handbook.fca.org.uk/handbook/icobs8) | Handbook updated through 2025-12-09 at inspection | Primary regulator handbook | UK-specific claims duties; not US/global law |
| [FCA vehicle-valuation multi-firm review](https://www.fca.org.uk/publications/multi-firm-reviews/findings-multi-firm-review-insurers-valuation-vehicles) | Published 2024-03-27; updated 2025-12-03 | Primary regulator production review | UK motor valuation/outcome/outsourcing findings; not universal valuation method |
| [California 2026 disaster claims guide](https://www.insurance.ca.gov/0200-industry/0050-renew-license/0200-requirements/upload/2026-Guide-for-Adjusting-Property-Claims-in-California-After-a-Major-Disaster_Final.pdf) | 2026-01-09 | Primary state regulator guide | California property/CAT examples only |
| [Texas Hurricane Beryl order](https://www.tdi.texas.gov/orders/documents/20248743.pdf) | 2024 event order | Primary state order | Scoped deadline-overlay example, not current general rule |
| [New York P&C claims regulation page](https://www.dfs.ny.gov/apps_and_licensing/property_insurers/laws_regs_cls) | Inspected 2026-08-31 | Primary state regulator source | Local Part 216 entry point; implementation must inspect current rule text |
| [FEMA NFIP Claims Manual](https://www.fema.gov/sites/default/files/documents/fema_rsl_nfip-claims-manual_06032025.pdf) | March 2025 | Primary government program manual | NFIP-specific claims/policy evidence example |
| [CMS NGHP mandatory reporting](https://www.cms.gov/medicare/coordination-benefits-recovery/mandatory-insurer-reporting) | Current; user guide v8.5 July 2026 | Primary federal program source | Section 111 reporting only; specialist route required |
| [IAIABC Claims EDI](https://www.iaiabc.org/edi-claims) | Release 3.1 docs published 2026-01-01 | Official standards body | Workers' compensation jurisdictional EDI; profiles vary |
| [ACORD P&C Data Standards](https://www-dev.acord.org/standards-architecture/acord-data-standards/Property_Casualty_Data_Standards) | P&C XML 2.13.0 January 2025 at inspection | Official standards body | Insurance vocabulary/transactions; license, profile, and implementation semantics vary |
| [X12 health-care transaction flows](https://x12.org/flow/health-care) | Current; 008060 X322 published 2025-01-17 | Official standards body | Health claim/payment/remittance/COB exchange only where applicable; adoption/profile varies |
| [Guidewire FNOL/policy/financial/recovery/concurrency docs](https://docs.guidewire.com/cloud/is/202603/cloudapibf/cloudAPI/ClaimCenter/fnol.html) | Cloud 2026.03 examples | Official vendor docs | Concrete adapter/adoption tests; not vendor-neutral contract |
| [Guidewire check lifecycle](https://docs.guidewire.com/cloud/is/202603/cloudapibf/cloudAPI/ClaimCenter/financials/check-lifecycle/c_the-check-life-cycle-after-submission.html) | Cloud 2026.03 | Official vendor docs | Distinct post-submission states/endpoints; not bank/payee delivery proof and tenant configuration varies |
| [Guidewire BillingCenter](https://docs.guidewire.com/cloud/is/202603/cloudapibf/cloudAPI/BillingCenter.html) | Cloud 2026.03 | Official vendor docs | Billing object/API example; not claim-payment or coverage authority |
| [Amazon Textract StartDocumentAnalysis](https://docs.aws.amazon.com/textract/latest/APIReference/API_StartDocumentAnalysis.html) | Current API inspected 2026-08-31 | Official provider API reference | Async OCR/layout/table example; source/page completeness and business validation remain application responsibilities |
| [OGC STAC](https://www.ogc.org/standards/stac/) and [OGC API Features](https://ogcapi.ogc.org/features/index.html) | STAC 1.1.0/API 1.0.0; Features 1.0.1 at inspection | Official standards body | Geospatial metadata/query interoperability; not source truth, license or causation |
| [Google Geocoding API v4](https://developers.google.com/maps/documentation/geocoding/start-v4) and [policies](https://developers.google.com/maps/documentation/geocoding/policies) | Preview; docs updated 2026-08-11 | Official provider docs | Representative geocoder with preview quota/storage/attribution/regional constraints; not stable universal behavior |
| [NWS alerts web service](https://www.weather.gov/documentation/services-web-alerts) | Current page inspected 2026-08-31 | Primary government service docs | CAP alert/event evidence and polling guidance; US-only and no loss/coverage proof |
| [FEMA OpenFEMA Disaster Declarations Summaries](https://www.fema.gov/about/openfema/disaster-declarations-summaries) | Version 1 page inspected 2026-08-31 | Primary government data catalog | Declaration/update evidence with raw historical-data caveats; not universal claims overlay |
| [Twilio Messages resource](https://www.twilio.com/docs/messaging/api/message-resource) | Current API docs inspected 2026-08-31 | Official provider docs | Message lifecycle/callback example; channel and delivery semantics vary and callback schema evolves |
| [OpenTelemetry specification](https://opentelemetry.io/docs/specs/otel/) and [semantic conventions](https://opentelemetry.io/docs/specs/semconv/) | Spec 1.60.0; semantic conventions 1.44.0 at inspection | Official project specifications | Diagnostic schema baseline; not audit or claim evidence |
| [SLSA specification](https://slsa.dev/spec/v1.2/) | Approved v1.2 at inspection | Official specification | Source/build provenance vocabulary; insufficient for model/rule/data/domain correctness alone |
| [OFAC insurance guidance](https://ofac.treasury.gov/faqs/topic/1616) | FAQs updated 2024-11-13 at inspection | Primary federal guidance | Sanctions/payment boundary; legal/compliance interpretation required |
| [NAIC Accounting Practices and Procedures Manual](https://content.naic.org/sites/default/files/publication-app-manual.pdf) | 2026 manual at inspection | Primary accounting guidance | Separates financial reporting context from model claims recommendation |
| [ASOP No. 43](https://www.actuarialstandardsboard.org/asops/propertycasualty-unpaid-claim-estimates/) | Current page; standard last revised 2011 | Primary professional standard | Actuarial unpaid-claim estimate boundary |

## Limitations and implementation gaps

This packet does not supply:

- a complete law survey or legal opinion for any jurisdiction;
- product forms, endorsements, carrier claims procedures, authority limits, reserve/payment rules, or templates;
- adjuster/appraiser/examiner licensing and catastrophe registration rules for every location;
- workers' compensation, health, disability, life, specialty, reinsurance, public-program, or litigation-specific adjudication logic;
- medical-necessity, disability, compensability, causation, repair, engineering, or valuation expertise;
- local limitation, appeal, complaint, language/accessibility, consent, recording, telematics/geolocation, biometric, or privacy rules;
- configured carrier/vendor API guarantees, idempotency, reconciliation, quotas, costs, or support windows;
- live tenant access or public operation-level documentation for many repair/estimating networks, bank/payment rails, fraud case systems, private policy archives, and carrier extensions;
- proof that a provider/model is fair, secure, or accurate on a carrier's data;
- numeric SLOs, model thresholds, authority bands, CAT capacity, or cost targets;
- a universal claim state machine; carrier domain states differ;
- assurance that an artifact is authentic, a visual finding proves causation, or a score proves fraud;
- exactly-once external effects.

Every implementation needs a signed support matrix and local qualified review.

## Refresh triggers

Refresh the affected guides and packet when any of the following occurs:

- jurisdictional claims, privacy, AI, data-security, fraud, licensing, limitation, payment, sanctions, or reporting rule changes;
- a catastrophe/emergency order is issued, amended, challenged, expires, or is interpreted differently;
- a policy form, endorsement, product, PAS archive, policy-to-claim copy, or carrier claim procedure changes;
- CMS NGHP, IAIABC, ACORD, X12, regulator/statistical, or partner profile/version changes;
- claims, document, vendor, communication, payment, bank, recovery, identity, or regulatory API semantics/version changes;
- adjuster/examiner/vendor qualifications, assignment, authority, SoD, or payment thresholds change;
- provider/model region, retention, training use, subprocessor, API, model, quota, price, safety behavior, or deprecation changes;
- context builder, memory/index, prompt, schema, validator, tool, adapter, rule, template, translation, or evaluation set changes;
- input/drift slices, reviewer correction, complaint, reopen, supplement, adverse outcome, subgroup disparity, cost, or latency crosses a threshold;
- any missed deadline, unsupported coverage/valuation communication, wrong party/payee/vendor, duplicate/unknown effect, fraud/legal disclosure, privacy/security incident, audit gap, or failed recovery drill occurs.

## Repository cross-cutting sources

- [Agent blueprint cross-cutting controls](agent-blueprint-cross-cutting-controls.md)
- [Agent state and event contracts](../../runtime/agent-state-and-event-contracts.md)
- [Durable execution](../../runtime/durable-execution.md)
- [Idempotency and side-effect safety](../../reliability/idempotency-and-side-effects.md)
- [Context engineering](../../context-memory/context-engineering.md)
- [Memory architecture](../../context-memory/memory-architecture.md)
- [Prompt injection and untrusted data](../../security/prompt-injection-and-untrusted-data.md)
- [Trajectory and reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md)
- [Deployment, release, and incident response](../../operations/deployment-release-and-incident-response.md)

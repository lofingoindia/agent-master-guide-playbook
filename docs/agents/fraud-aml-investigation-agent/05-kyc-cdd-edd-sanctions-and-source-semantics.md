# KYC, CDD, EDD, Sanctions, and Source Semantics

> **Purpose:** Use KYC/CDD/EDD, PEP, adverse-media, and sanctions sources according to their actual meaning, jurisdiction, revision, and limitations.  
> **Section index:** [Fraud and AML investigation agent](README.md)

## Keep obligations and workflows separate

AML suspicious-activity investigation, customer due diligence, fraud response, sanctions compliance, and law-enforcement/regulator reporting can share evidence, but they do not share one legal test or one effect. A suspicious pattern is not a sanctions match; a potential sanctions match is not a confirmed match; a PEP relationship is not wrongdoing; an adverse-media result is not verified fact; and a customer-risk tier is not a filing decision.

The agent may prepare evidence for each workflow. Jurisdiction-specific policy and accountable professionals determine obligations and actions.

## Jurisdiction profile

No global prompt can safely encode filing thresholds, deadlines, confidentiality, sanctions programs, data-transfer rules, and eligible approvers. Bind every case to a versioned profile:

~~~yaml
profile_id: us-bank-2026-08
commit: sha256:...
effective_from: 2026-08-01T00:00:00Z
owners:
  legal: workforce-group:...
  aml: workforce-group:...
  sanctions: workforce-group:...
scope:
  institution_type: bank
  products: [wire, deposit]
  legal_entities: [...]
  booking_locations: [...]
rules:
  cdd_sources: [...]
  sar_schema: fincen-bsa-xml@...
  deadlines: [...]
  confidentiality: [...]
  sanctions_authorities: [ofac]
  ownership_method: ...
  blocking_rejecting_reporting: ...
  retention: ...
  cross_border_transfer: ...
approvals:
  sar_filing: ...
  sanctions_action: ...
sources:
  legal_refs: [...]
  official_list_feeds: [...]
last_legal_reviewed_at: ...
next_review_at: ...
~~~

The profile is compiled and tested application policy. The model can quote a human-readable explanation, but only the policy service interprets it for authorization. Historical cases retain their decision-time profile; active cases can be forced to a new version only under an explicit migration/emergency rule.

## CDD and ongoing monitoring

CDD evidence is temporal and purpose-bound. A minimum review projection includes:

| Area | Preserve | Do not infer |
|---|---|---|
| Identity and verification | Method, source, verification status, date, expiries, discrepancies | Verified document means all later activity is legitimate |
| Account purpose/expected use | Customer statement, product, anticipated counterparties/geographies/volume, valid time | Deviation alone proves suspicious activity |
| Beneficial ownership/control | Direct/indirect relationship, percentage/type, chain, source, effective interval, unknown portion | Registry/LEI is complete current beneficial ownership truth |
| Nature of business/funds | Evidence source, customer statement versus independently verified fact | Industry or country is uniformly high risk |
| Risk assessment | Contributors, model/rules/version, overrides, review date, limitations | Risk tier is evidence of misconduct |
| Ongoing monitoring | Trigger, coverage, source freshness, prior material changes | “No previous alert” means no prior suspicious behavior |

The FATF standards use a risk-based approach, and FATF's February 2025 revisions increased emphasis on proportionality and simplified measures in lower-risk scenarios. Controls should therefore respond to evidenced risk and applicable mandatory rules; they should not turn a model-generated concern into blanket de-risking. U.S. agencies likewise state that no customer type is uniformly high risk and do not direct institutions to open, close, or maintain particular accounts.

## CDD and EDD evidence workflow

~~~mermaid
flowchart TD
    T["Alert/case question"] --> P["Pin jurisdiction + purpose + customer revision"]
    P --> B["Collect baseline CDD and coverage"]
    B --> G{"Material gap, change, or higher-risk factor?"}
    G -- No --> A["Assess against activity with limitations"]
    G -- Yes --> Q["Create narrowly scoped evidence request"]
    Q --> H["Authorized human / customer / provider process"]
    H --> V["Verify source, date, authenticity, contradictions"]
    V --> A
    A --> D["Draft cited case assessment"]
    D --> R["Qualified review and separate decision"]
~~~

The agent may propose an information gap or draft a request. It must not contact a customer, request a document from an external party, alter a risk rating, or classify a party as requiring EDD without an authorized workflow. Collection must remain proportionate to purpose and jurisdiction.

## Source-of-funds and source-of-wealth semantics

Separate:

- a customer's explanation;
- a document or registry assertion;
- an institutionally verified fact;
- transaction-derived behavior;
- an investigator inference.

Record currency, amount/range, relevant period, claimed origin, evidence type, issuer, authenticity/verification status, coverage, contradictions, and restrictions. Do not generalize one supported transaction into a customer's lifetime wealth, or equate inability to obtain a source with proof of illicit origin.

## PEP evidence

PEP status is a risk factor with jurisdiction-specific definitions and required measures, not an accusation. A PEP candidate record needs:

- source/provider, entry ID and revision;
- role/title, public body, country, start/end dates, current/former status;
- whether the candidate is the customer, family member, or close associate and the basis for that relationship;
- name/identifier match evidence, alternatives, transliteration, and false-positive indicators;
- applicable jurisdiction rule and date;
- human adjudication and next review date.

The model must not infer family or close-associate status from a shared surname, address, article, or social graph without governed evidence.

## Adverse-media evidence

Treat an article, search result, or provider summary as a source assertion. Preserve publisher, URL/source ID, publication and described-event dates, author where available, language, retrieval time, corrections, exact subject-link evidence, source type, and redisplay/licensing limits. Separate allegation, charge, enforcement action, dismissal, acquittal, conviction, and later correction.

Controls:

- use approved sources and lawful purpose; minimize sensitive personal data;
- resolve the subject before applying the content to the case;
- retrieve the underlying source where authorized rather than relying only on a generated/provider summary;
- tag article content as untrusted and block embedded instructions, links, trackers, and active content;
- seek corroboration proportionate to impact and show conflicting/later reports;
- do not let popularity, language availability, or publisher coverage become an unexamined risk proxy.

## Sanctions analysis is a separate, time-critical workflow

Sanctions obligations vary by authority, program/regime, entity type, location, ownership/control rule, transaction stage, licenses/exemptions, and time. A general AML risk score cannot make this decision.

### Potential-match package

~~~json
{
  "candidate_id": "san_01...",
  "screening_event": {"id": "...", "system": "...", "version": "...", "occurred_at": "..."},
  "screened_party": {"ref": "party_...", "identity_evidence": ["ev_..."]},
  "list_entry": {
    "authority": "OFAC",
    "programs": ["..."],
    "entry_id": "...",
    "list_revision": "...",
    "published_at": "...",
    "snapshot_hash": "sha256:..."
  },
  "comparisons": [
    {"field": "name", "candidate": "...", "entry": "...", "method": "...", "result": "..."}
  ],
  "ownership_control_paths": [{"path": ["..."], "source_refs": ["ev_..."], "unresolved": false}],
  "transaction_state": "pending",
  "jurisdiction_profile": "...",
  "licenses_or_exemptions": [{"ref": "...", "verification": "unresolved"}],
  "gaps": ["..."],
  "review_deadline": "...",
  "status": "potential_match"
}
~~~

For OFAC, the authority's own FAQ instructs institutions to compare more than the name—including the entry's full details and sanctions program—and distinguishes a potential from a valid match. Named-list screening alone is insufficient where an applicable ownership rule can extend restrictions to entities owned by blocked persons. Ownership calculations must be deterministic, program/profile-specific, temporal, cycle-aware, and reviewed; the language model must not calculate legal applicability from prose.

OFAC's current Sanctions List Service is its primary list-data application and exposes official SDN, non-SDN,
customized and archived-delta data. OFAC has changed SLS XML namespaces/schema locations before. Pin dataset/full-or-
delta identity, parser/schema, publication and retrieval time, file hash, entry/revision, counts and removal/correction
semantics; reconcile periodic full snapshots even when delta ingestion reports success.

The UN consolidated list also spans distinct sanctions regimes whose entries do not imply identical measures or criteria. Store regime/program membership and applicable measure semantics rather than a generic `is_sanctioned` boolean. The same principle applies to EU and national lists.

### List ingestion and active-case behavior

| Control | Requirement |
|---|---|
| Provenance | Official authority/feed, publication time, file/entry revision, signature/hash where supplied |
| Completeness | Manifest counts/checks, parser rejects/quarantine, delta sequence, full-snapshot comparison |
| Timeliness | Source-to-ingest latency and freshness SLO by authority; alert on missed update |
| Reproducibility | Immutable snapshot used for screening and review |
| Correction | Preserve withdrawn/corrected entry and dependent candidate lineage |
| Active cases | Pin reviewed snapshot but support emergency invalidation/rescreen under governed rule |
| Availability | Fail closed/open only according to preapproved transaction-stage and jurisdiction runbook; never let the model choose |
| Aggregator | May enrich or normalize; official source and discrepancy process remain authoritative |

### Decision and effect boundary

The agent can prepare match/ownership/licensing questions and surface the deadline. A qualified sanctions process decides match status and required reject/block/freeze/report/release behavior. A separate effect system verifies exact object and state, current law/policy, approval, segregation of duties, idempotency, and postcondition. Never expose a single `freeze_customer` tool.

FATF updated Recommendation 6 in June 2026 concerning targeted financial sanctions and humanitarian exemptions. That illustrates why versioned jurisdiction profiles and legal refresh triggers are operational requirements, not documentation niceties.

## Cross-border and enterprise information sharing

Information sharing can improve network understanding, but it does not automatically authorize centralizing all case data. Before a transfer or combined view, enforce legal basis, participating entity, purpose, jurisdiction, data category, SAR/STR confidentiality, localization, minimization, onward-disclosure, retention, subject rights/exceptions, and access/audit controls.

FATF's 2026 analysis of public-private information sharing emphasizes enabling legal frameworks and data-protection arrangements. FinCEN's 2025 cross-border guidance is specific to covered U.S. institutions and does not override SAR confidentiality. Encode applicable sharing paths; do not let the model “connect the dots” by crossing legal entities or tenants outside them.

## Contradictions to manage explicitly

| Tension | Safe resolution |
|---|---|
| Latest list/customer data vs reproducible decision | Preserve decision snapshot; append updates and re-open/rescreen under explicit materiality rule |
| Risk-based proportionality vs mandatory sanctions effect | Use proportionality for discretionary risk controls; execute applicable mandatory rule through separate deterministic/legal path |
| Rich network sharing vs privacy/confidentiality | Purpose-specific projection and approved sharing topology; never universal memory |
| Fast decision vs identity uncertainty | Escalate before deadline; do not convert similarity into a confirmed match |
| Full graph vs data minimization | Query only relationships necessary for the case; retain referenced evidence, not an unconstrained copy |
| Persistent customer risk view vs source correction/rights | Source-backed temporal record, restricted derived artifacts, correction/tombstone/retention workflow |

## Checklist

- [ ] Each case binds a reviewed jurisdiction profile, effective interval, legal sources, owners, and next refresh.
- [ ] CDD/customer-risk data exposes contributors, source revisions, valid time, gaps, and limitations.
- [ ] EDD requests are narrow proposals routed through an authorized workflow.
- [ ] PEP/adverse-media records distinguish candidate, source assertion, adjudication, and verified fact.
- [ ] Sanctions packages pin authority, program/regime, entry and list revision, identity comparison, ownership paths, transaction state, and deadline.
- [ ] Official list ingestion has completeness, freshness, correction, snapshot, and emergency-rescreen controls.
- [ ] AML, fraud, sanctions, filing, and restrictive-action decisions remain separate.
- [ ] Cross-border and cross-entity sharing is enforced by law/purpose policy, not model relevance.

## Sources and next guide

- [FATF Recommendations, current publication page](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Fatf-recommendations.html)
- [FATF — 2025 proportionality and financial-inclusion changes](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/update-standards-promote-financial-conclusion-feb-2025.html)
- [FATF — 2026 Recommendation 6 update](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/update-recommendation-6-june-2026.html)
- [FFIEC BSA/AML Manual — Customer Due Diligence](https://bsaaml.ffiec.gov/manual/AssessingComplianceWithBSARegulatoryRequirements/02_ep)
- [OFAC FAQ 5 — potential list matches](https://ofac.treasury.gov/faqs/5)
- [OFAC Frequently Asked Questions — ownership and program guidance](https://ofac.treasury.gov/faqs/all-faqs)
- [OFAC Sanctions List Service](https://ofac.treasury.gov/sanctions-list-service)
- [OFAC SLS XML namespace/schema change notice](https://ofac.treasury.gov/recent-actions/20240507_44)
- [United Nations Security Council Consolidated List](https://main.un.org/securitycouncil/en/content/un-sc-consolidated-list)
- [European Commission — sanctions overview and resources](https://finance.ec.europa.eu/eu-and-world/sanctions-restrictive-measures/overview-sanctions-and-related-resources_en)
- [FATF — Information Sharing to Combat Illicit Finance (2026)](https://www.fatf-gafi.org/en/publications/Methodsandtrends/information-sharing-ppp-data-protection-arrangements.html)
- [FinCEN — cross-border information sharing guidance (2025)](https://www.fincen.gov/system/files/2025-09/Crossborderguidance-508C.pdf)
- [FDIC — risk-based approach to customer relationships](https://www.fdic.gov/news/financial-institution-letters/2022/fil22028.html)

Next: [State, context, memory, planning, and provenance](06-state-context-memory-planning-and-provenance.md).

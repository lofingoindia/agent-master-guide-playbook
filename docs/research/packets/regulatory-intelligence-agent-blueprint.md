# Research Packet: Regulatory Intelligence Agent Blueprint

**Research date:** 2026-08-31  
**Scope:** Evidence-backed monitoring, change detection, temporal and applicability analysis, provision extraction, obligation and internal-control mapping, review, and bounded handoff for regulatory intelligence.  
**Output:** [Regulatory Intelligence Agent Blueprint](../../agents/regulatory-intelligence-agent/README.md)  
**Source policy:** Primary and official sources first; research benchmarks only for evaluation design.  
**Evidence register:** 75 unique primary, standards, professional-guidance, and research links.  
**Coverage warning:** Jurisdiction examples in this packet validate the architecture. They are not a legal-source catalog and do not establish coverage for any organization.

## Executive finding

A production regulatory-intelligence system should not be an autonomous legal interpreter. It should be a revision-aware evidence and workflow system with a low-authority reasoning component.

The durable boundary is:

```text
official or licensed source
  -> preserved artifact and source-status evidence
  -> version/provision graph and temporal facts
  -> cited candidate change
  -> applicability hypothesis
  -> interpretation hypothesis
  -> accountable professional decision
  -> approved obligation and mapping
  -> exact, reconciled internal handoff
```

This conclusion follows from the strongest cross-source findings:

1. Publication channels differ in legal and evidentiary status. A convenient API, HTML rendition, or consolidation can be useful without being the authentic or controlling artifact.
2. “Effective date” is not a portable scalar. Publication, entry into force, commencement, application, transposition, compliance, expiry, repeal, correction, and knowledge times may differ by provision, entity, product, territory, and transition condition.
3. Instrument type matters. Binding rules, proposed rules, regulator guidance, staff views, explanatory material, codes, and standards can have materially different effects even on the same website.
4. Applicability is a fact-sensitive professional conclusion, not a retrieval score. Entity permissions, activities, products, customers, thresholds, locations, exemptions, and transitional facts must be joined explicitly.
5. Source rights are per source and per operation. Public access does not automatically authorize storage, embedding, model processing, redistribution, or cross-tenant reuse.
6. General legal-language benchmarks do not validate source completeness, legal time, applicability, citation fidelity, review ergonomics, or safe effects. Local temporal and workflow evaluations are required.
7. The safe effect boundary is an exact approved internal handoff. Filing, legal advice, regulator communication, compliance certification, enforcement judgment, and autonomous control activation remain outside the agent.

## Questions investigated

The research pass tested the following architecture questions:

- What is the smallest useful workload that qualifies as regulatory intelligence rather than generic search, legal research, or compliance automation?
- Which artifact is controlling when an official publisher exposes PDF, HTML, XML, API, consolidation, and historical renditions?
- How should the system model corrections, amendments, prospective provisions, partial commencement, transposition, application, expiry, and retroactivity?
- How should jurisdiction, entity, product, service, customer, activity, threshold, territory, exemption, and transition conditions enter an applicability decision?
- How should raw text, provisions, extracted candidates, interpretations, professional decisions, obligations, controls, assessments, and effects remain distinct?
- Which tasks are deterministic, which benefit from model assistance, and which require accountable professional judgment?
- What source rights, confidentiality, privilege, identity, tenant, residency, and prompt-injection controls are required?
- What evaluations and release gates test the actual system rather than isolated legal-language performance?
- What must be deferred until evidence justifies added workflow, storage, graph, or agent complexity?

## Research method

The research was broad across source-publisher practice and narrow about claims. It prioritized:

1. official journals, registers, legislation services, and regulator documentation;
2. official source-status, authenticity, consolidation, version, commencement, and reuse notices;
3. legal-document, provenance, time, control-mapping, and archival standards;
4. current professional-responsibility guidance for AI-assisted legal work;
5. legal NLP and legal-reasoning benchmark papers only to determine what they do and do not measure.

The jurisdiction sample covered EU, U.S. federal, UK, Australian Commonwealth, and Canadian federal publication systems. These were selected because their official services expose meaningfully different authenticity, consolidation, instrument-status, and temporal behaviors. No inference was made that this sample generalizes every rule to every national, state, provincial, local, or sector regulator.

For architecture decisions, an official statement about its own publication was treated as authoritative for that publication. Cross-jurisdiction differences were retained instead of normalized away. Product or connector behavior must still be contract-tested at implementation time.

## Evidence hierarchy used in the blueprint

| Rank | Evidence class | Use | Important limit |
|---:|---|---|---|
| 1 | Authentic/authorized official artifact or gazette/register rendition | Controlling text and publication evidence where the publisher says it has that status | Authenticity does not itself decide applicability or interpretation |
| 2 | Official metadata, API, XML, register page, signature, or status notice | Discovery, identity, machine processing, integrity and status evidence | Machine-readable does not automatically mean legally controlling |
| 3 | Official consolidation or point-in-time service | Efficient current/historical reading when its status and currency are captured | Status differs by publisher; outstanding and unincorporated changes may exist |
| 4 | Regulator guidance, staff material, FAQs, explanatory material, consultations | Interpretive and workflow context | Binding effect and maturity must be classified, never inferred from the host domain |
| 5 | Licensed secondary source | Enrichment, citator, editorial cross-check, specialist taxonomy | Licence, latency, opaque editorial processing, and redistribution limits remain |
| 6 | Internal interpretation and decision | Organization-specific applicability, risk, obligations, and controls | Accountable owner, facts, date, scope, rationale, and evidence snapshot required |
| 7 | Model output | Candidate extraction, comparison, questions, and drafts | Never authority, legal advice, professional decision, or evidence by itself |

Rank is not a universal rule of legal authority. It is the system's evidence-handling order. A jurisdiction-specific source policy may establish a different controlling relationship and must override this generic ordering.

## Evidence-to-decision map

| Evidence finding | Blueprint decision |
|---|---|
| EUR-Lex states that electronic Official Journal editions have legal value under the applicable regime, while consolidated texts are documentation without legal effect | Store official-status evidence per rendition; preserve authentic artifact; never label all official-domain content “official law” |
| EUR-Lex explains that the date in a consolidation header is the applicability date of the latest included amendment | Do not reinterpret a consolidation date as publication, entry into force, complete currency, or entity compliance deadline |
| FederalRegister.gov calls its API/site output unofficial; NARA and GovInfo expose official publications and authenticated PDFs | Use the API for discovery and structured metadata, then bind high-risk claims to the official artifact and authenticity evidence |
| NARA describes the eCFR as a daily editorial compilation and warns about future-effective amendments, withdrawals, and corrections | Treat current text as a source-specific version view; retain amendment files, future state, corrections, and official-reference links |
| UK revised legislation can show prospective and outstanding effects; Australian compilations can omit uncommenced amendments or text-changing modifications | Model effect instructions and version lineage separately from displayed consolidated text; surface gaps instead of silently filling them |
| Canada gives electronic consolidations official evidentiary status while stating originals/amendments prevail on inconsistency and documenting a currency lag | `official_status`, `currency`, and `controlling_on_conflict` are separate fields, not one trust Boolean |
| EU directives require national transposition while regulations are directly applicable; instrument types and national measures therefore interact | Jurisdiction inheritance and transposition are explicit relations; an EU source match cannot create a Member State obligation by itself |
| FDA guidance and SEC staff statements are generally nonbinding; FCA and HSE distinguish rules, guidance, and specially situated codes | Classify instrument/provision status from source evidence; never infer binding effect from imperative wording alone |
| ELI provides identifiers/metadata and ELI-I can represent legislative impacts; Akoma Ntoso represents legal-document structure | Reuse stable publisher identifiers and structure where present, but keep an internal canonical model and raw artifact because adoption and granularity vary |
| LegalRuleML models legal rule modalities and multiple temporal aspects | Use it as an optional interchange/inspiration source, not an autonomous legal inference engine |
| W3C PROV, OWL-Time, and Memento address provenance, temporal entities, and prior web states | Align useful concepts while retaining a simpler application schema, append-only evidence, bitemporal records, and explicit legal-date types |
| OSCAL separates control, implementation, and assessment layers and provides a control-mapping model | Optional exports may express proposed relationships, but the agent must not convert a mapping into claimed implementation or assessment |
| ISO's current licence terms expressly restrict unlicensed AI/ML ingestion and related processing | Rights checks gate fetch, persistence, OCR, model access, embedding, evaluation, and output independently |
| ABA Formal Opinion 512 and the SRA's current AI warning retain competence, confidentiality, supervision, verification, and professional responsibility | Legal reviewers remain accountable; privilege is not promised by the software; source and output verification are mandatory |
| LexGLUE, LegalBench, and COLIEE measure bounded legal NLU/reasoning/retrieval tasks | Use them only for diagnostic signal; release gates require local frozen-source, temporal, applicability, citation, effect, and incident suites |
| NIST CAISI documents solution contamination and grader gaming in agent evaluations | Restrict evaluation tools/network, use held-out fixtures, inspect transcripts, and reject metric-only release evidence |

## Jurisdiction and source observations

### European Union

EUR-Lex makes several distinctions that directly affect the system model:

- the electronic Official Journal is an authenticity/legal-value surface, not merely another download;
- consolidated texts combine amendments for reading but are explicitly documentary and have no legal effect;
- the date in a consolidation header has a specific publisher-defined meaning;
- regulations, directives, decisions, recommendations, and opinions do not share one binding/applicability model;
- directives introduce national transposition relationships;
- ELI improves identity and metadata exchange, while deployment depth differs by publisher and subdivision;
- ELI Pillar 4 supports complete metadata retrieval plus updates, but a consumer still needs reconciliation;
- ELI-I can describe impacts, but published impact metadata remains evidence to validate, not permission to skip source text;
- authentic language versions must be preserved; the Commission explicitly says not to use its machine translation for EU legislation.

Architecture consequence: `jurisdiction = EU` and `document_status = official` are radically insufficient. The record needs instrument class, rendition, authentic language, source status, version/impact lineage, national transposition links where applicable, and independently reviewed applicability.

### United States federal

The U.S. federal ecosystem demonstrates why discovery and authority channels must be separated:

- the Federal Register is the daily journal; FederalRegister.gov exposes convenient unofficial structured data;
- official GovInfo packages/granules and signed PDFs provide authenticity/integrity evidence;
- CFR annual editions and the daily eCFR answer different currency questions;
- the eCFR describes future-effective amendment and correction behavior that can change before operative dates;
- material incorporated by reference can be legally significant without being freely redistributable;
- agency guidance and staff statements can be valuable while not creating enforceable duties by themselves.

Architecture consequence: use official identifiers and API/bulk feeds for discovery, retain the Federal Register/CFR relationship, keep proposal/final/correction/effective events distinct, and require a source-rights decision before processing incorporated standards.

### United Kingdom

The UK sample exposes three separate failure modes:

- revised legislation can have prospective provisions, partially commenced changes, and outstanding effects;
- the FCA Handbook assigns provision statuses and states that the legal instrument is definitive on discrepancy;
- HSE guidance and Approved Codes of Practice have different legal significance.

Architecture consequence: preserve legislation effects and commencement qualifications; store status at provision level; resolve website text to instrument evidence where the publisher requires it; do not flatten guidance, rules, directions, and codes into an `obligation` label.

### Australian Commonwealth

The Federal Register of Legislation distinguishes authorized versions and explains that:

- an “effective” date can have a register-specific meaning and is not necessarily commencement;
- different provisions can commence at different times, including retrospectively in some circumstances;
- compilations expose histories and may not show uncommenced amendments, modifications, or transitional material in the body text;
- future law compilations are not authorized versions;
- rectification/replacement and misdescribed-amendment cases exist.

Architecture consequence: preserve authorized rendition evidence, endnotes, compilation range, commencement facts, modifications, transition provisions, and rectification lineage. Never infer “currently binding text” from a green status or latest visible body alone.

### Canadian federal

Canada supplies a useful counterexample to any rule that “consolidations are unofficial”:

- Canada Gazette is the official newspaper, and official and alternate renditions are distinguished;
- Justice Laws electronic consolidations have official evidentiary status;
- the site nevertheless says originals/amendments prevail on inconsistency;
- the service documents a normal current-to lag and displays currency information.

Architecture consequence: model official evidentiary status, source currency, and conflict priority separately. The EU consolidation rule must not be generalized to Canada.

## Temporal model decision

The research rejected a single `effective_date`. The minimum production representation is:

```text
legal/event time
  publication
  adoption/making
  entry into force or commencement
  application interval(s)
  transposition deadline and national measure interval(s)
  compliance deadline/milestones
  expiry/sunset/repeal
  correction/supersession

knowledge/system time
  first observed
  fetched
  verified
  extracted
  reviewed
  decided
  handed off
  corrected/retracted
```

Each date is a typed fact with source locator, scope, precision, confidence, condition, reviewer state, and `known_from`/`known_to`. Applicability intervals are derived only after joining the legal facts to a versioned entity/product fact snapshot. Unknown, conditional, disputed, prospective, and partially commenced states remain explicit.

This is bitemporal in behavior, but the blueprint does not require a specialized temporal database. A relational store with append-only revisions, range constraints, version links, and tested as-of queries is the simplest reliable starting point.

## Applicability and interpretation decision

Retrieval answers “what text may be relevant?” Applicability asks whether a particular provision affects a particular legal entity, business unit, permission, activity, product, service, customer class, location, threshold state, exemption, or transition cohort during a specified interval.

The agent may produce a cited hypothesis containing:

- matched and unmatched predicates;
- supporting, contradicting, and missing evidence;
- assumed entity/product facts and their snapshot identifiers;
- temporal scope;
- source/instrument status;
- uncertainty and specific review questions.

It may not mark the result as applicable, provide legal advice, waive an issue, or silently turn the hypothesis into an obligation. An accountable legal/compliance professional decides applicability and interpretation. The decision record binds the evidence snapshot, facts snapshot, scope, rationale, owner, date, review/expiry, and any dissent.

## Source identity, authenticity, and change decision

Four identities remain separate:

| Identity | Example | Why it cannot be collapsed |
|---|---|---|
| Legal work | Regulation, Act, rule, instrument | Persists across expressions and amendments |
| Expression/version | Language, point-in-time text, as-made or consolidated version | Meaning and temporal scope vary |
| Manifestation/rendition | PDF, XML, HTML, API record | Authenticity, parseability, pagination, and rights vary |
| Observation | Exact fetched bytes at a time | Re-fetches, corrections, outages, and parser changes must be auditable |

Every stored artifact receives content hash, request/response metadata, publisher identifiers, declared media/language, source-status evidence, licence-policy version, fetch/verification time, parser/OCR version, and immutable object reference. Publisher signatures and identifiers are verified where supported; a local hash proves local byte identity, not publisher authority.

Incremental feeds are accelerators, not completeness proof. Each connector therefore needs cursor overlap, deduplication, periodic full-window reconciliation, expected-volume checks, deletion/withdrawal handling, correction detection, and an owner for degraded source state.

## Technology and integration choices

| Concern | Selected default | Deferred or rejected default | Reason |
|---|---|---|---|
| Workflow | Explicit durable state machine and transactional outbox | Open-ended agent loop; multi-agent debate | Legal effects and retries need inspectable state; extra agents do not create authority |
| Operational store | Relational database with constraints and append-only revisions | Graph database first | Version, review, idempotency, tenancy, and as-of needs fit a relational baseline; graph projection can come later |
| Evidence | Immutable object storage plus hash and metadata | URL-only citations | Publisher pages and bytes change; prior decisions must reopen exact evidence |
| Retrieval | Deterministic metadata/filter search, lexical search, then bounded semantic rerank where licensed | Embed every legal document | Rights, temporal leakage, provenance loss, and cross-tenant risk outweigh convenience |
| Parsing | Source-specific deterministic parsers; OCR only when required; typed validation | One universal legal parser | Publisher structures and failure modes differ |
| Rules | Deterministic date/effect/applicability predicate services | Model-only rule execution; full LegalRuleML inference | Critical dates and predicates require replayable computation; broad formalization is expensive and brittle |
| Model | Structured candidate extraction/comparison/questions/drafts with evidence IDs | Free-form legal answer endpoint | Typed output and review state are enforceable boundaries |
| Control exchange | Neutral, versioned proposal; optional OSCAL/vendor adapter | Claim implementation or assessment | Obligation mapping is not implementation evidence or compliance certification |
| Translation | Licensed, labeled aid with authentic source beside it | Replace authentic text with model translation | Language mismatch can change legal meaning; some publishers expressly warn against this |
| External effects | Exact approved internal handoff with idempotency/reconciliation | Filing, regulator contact, legal advice, certification, or broad write tool | These acts require professional or organizational authority the agent does not own |

## Pass-2 volatile platform and standards refresh

The following primary surfaces were rechecked on 2026-08-31. Their current mechanics inform adapter tests but are not guarantees for a particular tenant, plan, region, future release, or legal conclusion:

| Surface/status observed | Engineering evidence | Limitation and refresh trigger |
|---|---|---|
| [FederalRegister.gov API v1](https://www.federalregister.gov/developers/documentation/api/v1) | Keyless structured discovery API and links to GovInfo | The official site explicitly says its XML/web rendition is not the official legal edition; refresh on legal-status notice, schema or identifier change |
| [GovInfo API/developer hub](https://www.govinfo.gov/developers) and [URL structure](https://www.govinfo.gov/help/url-structure) | Package/granule identities, multiple renditions, metadata/fixity and permanent URL patterns | Collection semantics and historical/default search differ; test full reconciliation, replacement, pagination and exact collection behavior |
| [EUR-Lex/Cellar reuse access](https://eur-lex.europa.eu/content/help/data-reuse/reuse-contents-eurlex-details.html?locale=en) | REST, SPARQL, RSS and Formex access to metadata/content | RSS can be extremely high-volume and access format does not decide legal status; refresh on status, identifier, Formex or Cellar contract change |
| [Australian Register API v1](https://www.legislation.gov.au/help-and-resources/using-the-legislation-register/data-share-and-reuse) | Authorized register; OAS 3.0.1 REST API; JSON/documents; no API key | Publisher says the live API may change and performance/availability can be affected by load; pin OpenAPI fingerprint |
| [Canada Gazette RSS](https://gazette.gc.ca/rss/sc-rb-eng.html) and [status explanation](https://gazette.gc.ca/cg-gc/lm-sp-eng.html) | Part I/II/III feeds and explicit rendition status | Official publication is bilingual PDF while unofficial HTML is more up to date; preserve both statuses and reconcile extra editions |
| [Regulations.gov API v4](https://open.gsa.gov/api/regulationsgov/) | Documents/dockets/comments, key, pagination, attachments and configurable fields; POST comments also exist | Agency fields can change and comment data has documented limits; this blueprint permits GET discovery only and prohibits regulator-comment effects |
| [Microsoft Graph drive delta v1.0](https://learn.microsoft.com/en-us/graph/api/driveitem-delta?view=graph-rest-1.0) | Paged change synchronization ending in delta link | Feed returns latest state, can repeat items and omits some properties; track by ID and test resync/permissions/deletion |
| [Confluence Cloud REST API v2 pages](https://developer.atlassian.com/cloud/confluence/rest/v2/api-group-page/) | Page identity, status, owner/author, version and cursor pagination | Tenant restrictions, deletion/trash, history and body formats require live-tenant qualification |
| [Jira Cloud REST API v3 issues](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/) | Rights/field-dependent create, changelog, bulk and task surfaces | Create metadata is evolving/deprecated in parts and bulk can partially succeed; discover schema under production identity and reconcile effects |
| [SLSA 1.2](https://slsa.dev/spec/v1.2/) and [NIST SSDF status](https://csrc.nist.gov/Projects/ssdf/publications) | Approved provenance/verification spec; SSDF 1.1 final while 1.2 was draft | Supply-chain evidence does not prove legal parser/source-map correctness; refresh on approved/final version change |
| [European Commission AI Act implementation page](https://digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai) | Current official implementation/enforcement timeline changed materially during 2025–2026 | Demonstrates why timeline facts must be sourced bitemporally; deployment counsel must verify the controlling instruments and local scope |

### Recorded contradictions and status limits

- NIST's [OSCAL model reference](https://pages.nist.gov/OSCAL-Reference/models/) labels `v1.2.3` as the latest reference, while the [official GitHub releases page](https://github.com/usnistgov/OSCAL/releases) showed `v1.2.2` as the latest released asset dated 2026-04-30. A deployment must resolve and pin the chosen release/schema digest; the packet does not silently call either universal.
- FederalRegister.gov is an official-government-operated discovery surface yet states it is not the official legal edition. “Official domain” and “legally official rendition” are different facts.
- Canada Gazette states its bilingual PDF is official while HTML is unofficial but more up to date. Currency and legal status cannot be collapsed into one ranking.
- Australia's Register describes its website as authorized while also warning that its public API is live and subject to change. Source authority does not imply API stability.
- Commercial legal-research and regulatory-intelligence APIs are usually tenant/contract specific. No public marketing claim was converted into an API, coverage, rights, currentness or reconciliation guarantee.

## Disagreements and rejected universal rules

### “Official source” is a Boolean

Rejected. Canada can give electronic consolidations official evidentiary status while EU consolidated texts have no legal effect. FederalRegister.gov is an official-government-hosted convenience surface that describes itself as unofficial. Store the publisher's precise status statement and conflict rule per rendition.

### “Latest text” is the applicable law

Rejected. Future-effective, prospective, partially commenced, transitional, retrospective, outstanding, unincorporated, modified, disallowed, corrected, and repealed provisions all defeat this shortcut. Resolve a provision against typed time facts and the subject facts for the requested as-of decision.

### “Imperative language means binding obligation”

Rejected. Guidance can use “must,” and binding rules can incorporate definitions, exceptions, conditions, or standards elsewhere. Instrument and provision status, legal basis, cross-references, facts, and professional interpretation govern the internal obligation decision.

### “Consolidations are always inferior”

Rejected as a universal claim. Their legal/evidentiary status differs. They are often the best reading surface, but the agent must retain source-specific currency, conflict, and authenticity semantics.

### “A knowledge graph solves applicability”

Rejected as a starting assumption. A graph can improve traversal after relationships are stable, but it cannot repair wrong source status, dates, facts, interpretation, or rights. Begin with constrained records and materialize a graph only for measured query needs.

### “LegalRuleML can automate legal judgment”

Rejected. LegalRuleML offers useful modalities and temporal representation, but converting open-text law and facts into correct executable rules is itself interpretive work. Use formal rules only for narrow, reviewed predicates.

### “Licensed summaries are safer than primary text”

Rejected. They can add editorial value and citator coverage, but licence limits, editorial latency, opaque transformations, and missing detail remain. They are enrichment and cross-check evidence unless an approved source policy says otherwise.

### “More agents increase correctness”

Rejected as the default. Independent extraction or adversarial review can be an evaluation technique, but production correctness comes from authoritative evidence, deterministic invariants, source-specific tests, professional review, and exact effect binding. Add agent roles only after a single controlled workflow demonstrates a measurable gap.

### “High benchmark scores prove production readiness”

Rejected. LexGLUE, LegalBench, and COLIEE do not test this system's source feeds, source status, dates, entity facts, permissions, review UX, handoff identity, or recovery behavior. Benchmark tasks may diagnose model capability but cannot pass a release.

## Legal, ethical, and product boundaries

The blueprint is an engineering reference, not legal advice. A deployed product must have jurisdiction-specific counsel and compliance owners establish:

- source catalog and hierarchy;
- authentic/authorized rendition and conflict rules;
- instrument/provision status taxonomy;
- source licences and allowed operations;
- entity/product fact ownership;
- interpretation and applicability decision authority;
- privilege/confidentiality handling;
- retention, legal hold, residency, and disclosure duties;
- external effect authority and segregation of duties.

The system must not describe an output as legally privileged merely because it is labeled confidential or routed to counsel. Privilege and work-product protection are legal determinations dependent on facts and jurisdiction. Likewise, an internal control mapping is not legal compliance, implementation, operating effectiveness, an audit opinion, or a regulator filing.

Current professional guidance strengthens, rather than replaces, these product limits. ABA Formal Opinion 512 identifies competence, confidentiality, communication, supervision, verification/candor, and fee duties for lawyers using generative AI. The SRA's August 2026 warning highlights inaccurate information and client-confidentiality risks. These sources apply to their respective professional settings; they are not a globally portable rulebook.

## Evaluation implications

The minimum local evaluation portfolio includes:

| Suite | What it must prove |
|---|---|
| Source completeness | Feed gaps, delayed/duplicate/reordered events, silent edits, withdrawals, corrected bytes, and full reconciliation |
| Identity/provenance | Work/version/rendition/observation separation, hash/signature checks, stable locator resolution, exact reviewed snapshot |
| Temporal | Partial commencement, prospective/future-effective changes, transposition, transition, expiry, repeal, retroactivity, corrections, knowledge-time replay |
| Status | Binding rule versus proposal, guidance, staff view, explanatory material, code, incorporated standard, and source-specific conflict rules |
| Applicability | Entity/product/activity/customer/territory/threshold/exemption joins, missing facts, conflicts, and abstention |
| Extraction | Provision boundaries, defined terms, exceptions, conditions, cross-references, tables, notes, multilingual evidence |
| Human factors | Citation verification, disagreement capture, review burden, alert fatigue, handoff comprehension, safe override |
| Effect safety | Wrong tenant, stale approval, payload mutation, duplicate/reordered callback, timeout-after-success, reconciliation, revocation |
| Security/rights | Prompt injection, poisoned documents, OCR traps, unauthorized source operation, confidential-data egress, cross-tenant retrieval |
| Recovery | Source outage, parser regression, model rollback, queue backlog, disaster restore, stale derived artifacts, incident reconstruction |

Public benchmark material must be excluded from release fixtures where contamination is plausible. Freeze source bytes, gold facts, professional decisions, expected evidence chains, and tool affordances. Review traces for solution lookup and grader gaming. Report component and end-to-end results by jurisdiction/source family and hard-case slice; one aggregate accuracy number is not acceptable.

## Research saturation, gaps, and limitations

Research was stopped when further searches repeated the same architecture constraints: source status is publisher-specific, temporal semantics are multi-axis, applicability is fact-sensitive, guidance status varies, rights are operation-specific, and professional judgment remains accountable. Additional sources would add jurisdictions and connectors but would not materially change the generic control plane.

Genuine gaps remain:

- No organization, industry, regulator list, legal entities, products, or target jurisdictions were provided. The blueprint therefore cannot define a production source catalog or applicability policy.
- State, provincial, local, supranational beyond the EU, court, enforcement, licensing, and sector-specific sources were not comprehensively researched.
- Representative publisher/provider mechanics were refreshed and converted into an adapter playbook, but no live tenant or contract was available. Exact APIs, schemas, quotas, authentication, entitlement, audit, residency, retention, deletion, export, consistency and reconciliation must be revalidated during connector implementation.
- No claim is made that ELI, Akoma Ntoso, LegalRuleML, OSCAL, PROV, OWL-Time, or Memento is universally adopted or should become the internal canonical schema.
- Professional-responsibility sources are jurisdiction-specific examples, not a complete unauthorized-practice, privilege, or ethics analysis.
- The benchmark sources test bounded research tasks and do not supply production accuracy expectations.
- No cost, latency, volume, staffing, RTO/RPO, SLO, or legal-risk tolerance was supplied; the stage gates require those to be measured locally.

## Refresh triggers

Re-run targeted research when any of the following changes:

- official/legal/evidentiary status, signature practice, or conflict notice on a monitored source;
- publisher identifier, API, feed, bulk format, update cadence, correction, or consolidation behavior;
- law type, date vocabulary, transposition, commencement, transition, or applicability policy;
- source licence, AI/TDM restriction, redistribution term, or incorporated third-party material;
- ELI, Akoma Ntoso, LegalRuleML, OSCAL, PROV, OWL-Time, Memento, or telemetry compatibility requirement;
- target model/provider data handling, retention, regional processing, tool behavior, or release;
- a missed change, false applicable decision, wrong date/status, citation mismatch, source-rights incident, cross-tenant exposure, disputed professional decision, or unsafe effect;
- a new jurisdiction, regulator, entity, product, activity, customer class, or external destination enters scope.

Even without a trigger, source owners should attest their catalog at least quarterly, critical connectors should be reconciled on a risk-based schedule, and this cross-jurisdiction research packet should receive a dated annual review.

## Selected primary and research sources

The links below are the evidence register used for the blueprint, not a production source catalog. Access and page metadata were checked on the research date.

### European Union publication, identity, time, and rights

- [EUR-Lex: Official Journal](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=LEGISSUM%3Aofficial_journal)
- [EUR-Lex: electronic Official Journal authenticity](https://eur-lex.europa.eu/content/help/oj/authenticity-eOJ.html?locale=en)
- [EUR-Lex: consolidated texts and their legal status](https://eur-lex.europa.eu/collection/eu-law/consleg.html?locale=en)
- [EUR-Lex: European Legislation Identifier](https://eur-lex.europa.eu/content/help/eurlex-content/eli.html?locale=en)
- [ELI Register: implementation](https://eur-lex.europa.eu/eli-register/implementation.html)
- [ELI Register: implementation resources, ELI-I, and Pillar 4](https://eur-lex.europa.eu/eli-register/implementing_eli.html)
- [ELI Pillar 4 protocol specification](https://eur-lex.europa.eu/content/eli-register/ELI-Pillar-IV-protocol-specification-v1.0_en.pdf)
- [EUR-Lex: permanent ELI links](https://eur-lex.europa.eu/content/help/data-reuse/linking.html?locale=en)
- [EUR-Lex: EU legal instruments](https://eur-lex.europa.eu/EN/legal-content/glossary/eu-legal-instruments.html)
- [EUR-Lex: transposition](https://eur-lex.europa.eu/EN/legal-content/glossary/transposition.html)
- [EUR-Lex Joint Practical Guide](https://eur-lex.europa.eu/content/techleg/KB0213228ENN.pdf)
- [EUR-Lex legal notice](https://eur-lex.europa.eu/content/legal-notice/legal-notice.html?locale=en)
- [EUR-Lex reuse notice](https://eur-lex.europa.eu/content/help/data-reuse/reuse-contents-eurlex-details.html?locale=en)
- [European Commission: machine translation on Europa](https://commission.europa.eu/languages-our-websites/use-machine-translation-europa_en)

### United States federal publication, authenticity, guidance, and incorporated material

- [National Archives: About the Federal Register](https://www.archives.gov/federal-register/the-federal-register/about.html)
- [FederalRegister.gov API and legal-status notice](https://www.federalregister.gov/developers/documentation/api/v1)
- [National Archives: Code of Federal Regulations](https://www.archives.gov/federal-register/cfr)
- [National Archives: About the eCFR](https://www.archives.gov/federal-register/cfr/about-ecfr)
- [GovInfo: Code of Federal Regulations](https://www.govinfo.gov/help/cfr)
- [GovInfo Developer Hub](https://www.govinfo.gov/developers)
- [GovInfo API overview](https://www.govinfo.gov/features/api)
- [GovInfo URL structure](https://www.govinfo.gov/help/url-structure)
- [GovInfo authentication](https://www.govinfo.gov/about/authentication)
- [National Archives: incorporation by reference](https://www.archives.gov/federal-register/write/ibr)
- [FDA: status of guidance documents](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/guidances)
- [SEC: statement regarding SEC staff views](https://www.sec.gov/newsroom/speeches-statements/statement-clayton-091318)
- [U.S. Copyright Office: 17 U.S.C. §105 and related provisions](https://www.copyright.gov/title17/92chap1.html)

### United Kingdom publication and provision status

- [The National Archives: Guide to Revised Legislation](https://www.legislation.gov.uk/pdfs/GuideToRevisedLegislation_Jan_2012.pdf)
- [legislation.gov.uk editorial effects guidance](https://community.legislation.gov.uk/mediawiki/index.php?title=Preparation_Tasks%2FRecord_Effects)
- [FCA Handbook Reader's Guide](https://handbook.fca.org.uk/guides/reader-guide)
- [FCA Handbook: interpreting the Handbook](https://handbook.fca.org.uk/handbook/gen2/gen2s2)
- [HSE: legal status of guidance and ACOPs](https://www.hse.gov.uk/legislation/legal-status.htm)
- [UK Open Government Licence 3.0](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/)
- [The National Archives: copyright terms and third-party material](https://www.nationalarchives.gov.uk/terms-and-conditions/copyright/)

### Australian Commonwealth publication and time

- [Federal Register of Legislation: frequently asked questions](https://www.legislation.gov.au/help-and-resources/using-the-legislation-register/frequently-asked-questions)
- [Federal Register of Legislation: reading legislation](https://www.legislation.gov.au/help-and-resources/understanding-legislation/reading-legislation)
- [Federal Register of Legislation: glossary](https://www.legislation.gov.au/help-and-resources/understanding-legislation/glossary)
- [Federal Register of Legislation: future law compilations](https://www.legislation.gov.au/future-law-compilations)
- [Federal Register of Legislation: explanatory statements](https://www.legislation.gov.au/help-and-resources/using-the-legislation-register/explanatory-statements-for-legislative-instruments)
- [Federal Register of Legislation: system updates](https://www.legislation.gov.au/help-and-resources/using-the-legislation-register/system-updates)

### Canadian federal publication and consolidation

- [Canada Gazette home](https://gazette.gc.ca/accueil-home-eng.html)
- [About the Canada Gazette](https://gazette.gc.ca/cg-gc/lm-sp-eng.html)
- [Canada Gazette publications](https://gazette.gc.ca/rp-pr/publications-eng.html)
- [Justice Laws: official status and conflict note](https://laws-lois.justice.gc.ca/eng/importantnote/)
- [Justice Laws frequently asked questions](https://laws-lois.justice.gc.ca/eng/faq/)

### Representation, provenance, temporal, control, and archival standards

- [OASIS Akoma Ntoso 1.0 XML Vocabulary](https://docs.oasis-open.org/legaldocml/akn-core/v1.0/akn-core-v1.0-part1-vocabulary.html)
- [OASIS LegalRuleML 1.0](https://docs.oasis-open.org/legalruleml/legalruleml-core-spec/v1.0/os/legalruleml-core-spec-v1.0-os.pdf)
- [W3C PROV-O](https://www.w3.org/TR/prov-o/)
- [W3C PROV Constraints](https://www.w3.org/TR/prov-constraints/)
- [W3C Time Ontology in OWL](https://www.w3.org/TR/owl-time/)
- [RFC 7089: Memento](https://www.rfc-editor.org/info/rfc7089/)
- [NIST OSCAL layers and models](https://pages.nist.gov/OSCAL/learn/concepts/layer/)
- [NIST OSCAL Control Layer and mapping model](https://pages.nist.gov/OSCAL/learn/concepts/layer/control/)
- [NIST OSCAL latest model reference](https://pages.nist.gov/OSCAL-Reference/models/latest/)
- [ISO terms and licence agreement](https://www.iso.org/terms-conditions-licence-agreement.html)

### Professional responsibility, risk, and evaluation

- [ABA Formal Opinion 512: Generative Artificial Intelligence Tools](https://www.americanbar.org/content/dam/aba/administrative/professional_responsibility/ethics-opinions/aba-formal-opinion-512.pdf)
- [ABA Model Rule 5.3 commentary](https://www.americanbar.org/groups/professional_responsibility/publications/model_rules_of_professional_conduct/rule_5_3_responsibilities_regarding_nonlawyer_assistant/comment_on_rule_5_3/)
- [Solicitors Regulation Authority: Misuse of AI warning notice](https://media.sra.org.uk/solicitors/guidance/misuse-ai/)
- [NIST AI RMF Generative AI Profile](https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence)
- [NIST CAISI: Cheating on AI Agent Evaluations](https://www.nist.gov/caisi/cheating-ai-agent-evaluations)
- [NIST CAISI: Analyzing Transcripts from AI Agent Evaluations](https://www.nist.gov/blogs/caisi-research-blog/analyzing-transcripts-ai-agent-evaluations)
- [LexGLUE paper](https://aclanthology.org/2022.acl-long.297/)
- [LegalBench paper](https://arxiv.org/abs/2308.11462)
- [COLIEE 2026](https://coliee.org/)

## Blueprint traceability

| Research concern | Blueprint guide |
|---|---|
| Category boundary, authority, actors, workload fit | [Mission, Boundaries, Authority, and Workload Fit](../../agents/regulatory-intelligence-agent/01-mission-boundaries-authority-and-workload-fit.md) |
| Architecture, source classes, tool contracts, integrations | [Reference Architecture, Sources, Tools, and Integrations](../../agents/regulatory-intelligence-agent/02-reference-architecture-sources-tools-and-integrations.md) |
| Identity, authenticity, provenance, feeds, correction, reconciliation | [Source Identity, Provenance, and Change Monitoring](../../agents/regulatory-intelligence-agent/03-source-identity-provenance-and-change-monitoring.md) |
| Legal/knowledge time, versions, applicability | [Temporal, Version, and Applicability Semantics](../../agents/regulatory-intelligence-agent/04-temporal-version-and-applicability-semantics.md) |
| Evidence ladder, interpretation, obligation/control mapping, exact handoff | [Provision, Obligation, Impact, and Handoff](../../agents/regulatory-intelligence-agent/05-provision-obligation-impact-and-handoff.md) |
| Cases, events, context, memory, orchestration, recovery | [State, Events, Context, Memory, and Orchestration](../../agents/regulatory-intelligence-agent/06-state-events-context-memory-and-orchestration.md) |
| Rights, confidentiality, prompt injection, identity, tenancy | [Security, Confidentiality, Source Rights, and Tenancy](../../agents/regulatory-intelligence-agent/07-security-confidentiality-source-rights-and-tenancy.md) |
| Telemetry, SLOs, evaluations, incidents | [Reliability, Observability, Evaluation, and Incidents](../../agents/regulatory-intelligence-agent/08-reliability-observability-evaluation-and-incidents.md) |
| Deployment, cells, scale, cost, releases, evolution | [Deployment, Scale, Cost, and Evolution](../../agents/regulatory-intelligence-agent/09-deployment-scale-cost-and-evolution.md) |
| Stages 0–6 and evidence-based exit gates | [Zero-to-Production Stages and Exit Gates](../../agents/regulatory-intelligence-agent/10-zero-to-production-stages-and-exit-gates.md) |
| Provider/API rights, status, coverage, finality and reconciliation qualification | [Adapter Qualification and Provider Playbooks](../../agents/regulatory-intelligence-agent/11-adapter-qualification-and-provider-playbooks.md) |

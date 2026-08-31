# Real-Estate and Property-Operations Agent Blueprint — Research Packet

**Category:** 45 — Real-estate and property operations

**Research date:** 2026-08-31

**Source preference:** statutes/regulations and official agencies; official standards/specifications; official vendor documentation; original research papers; current official repositories/documentation
**Playbook:** [Real-Estate and Property-Operations Agent Playbook](../../agents/real-estate-property-operations-agent/README.md)

## Purpose and method

This packet records the evidence and decisions behind a production playbook for an AI-assisted property-operations workflow. It is not a legal opinion and does not claim that a federal source is the complete rule for a particular property. Landlord-tenant, fair-housing, consumer-reporting, accessibility, licensing, safety, notice, communications, records, rent, eviction, and building rules depend on jurisdiction, housing program, property, lease, activity, and date.

Research questions:

1. Which activities are appropriate for deterministic workflow, model assistance, human decision, or explicit exclusion?
2. Which domain entities and source-authority boundaries prevent property, listing, lease, occupancy, screening, maintenance, and access errors?
3. What current housing, consumer-reporting, accessibility, safety, privacy, payment, and competition sources constrain the design?
4. Which property, listing, work-order, geospatial, building, access, messaging, e-sign, payment, and runtime standards/interfaces are useful?
5. How should durable workflows handle approval, retry, unknown effects, reconciliation, memory, compaction, observability, evaluation, release, and recovery?
6. Where do current sources conflict with older guidance or with one another?

Method:

- searched multiple formulations per domain;
- preferred current first-party pages and primary documents;
- checked publication/update dates and whether guidance was current, withdrawn, proposed, or historical;
- compared ratified standards with actual certification/tooling baselines;
- treated vendor public API descriptions as discovery, not proof of tenant-specific semantics;
- used original papers for counterfactual fairness and official runtime specifications for durability/telemetry;
- converted claims into architecture decisions, failure tests, gates, and refresh triggers.

Research stopped when additional searches repeated the same authoritative rules or vendor marketing without changing a design decision. Deployment counsel, contracts, local code, and tenant-specific API qualification remain required.

## Executive findings

1. **The safe product is a governed workflow, not an autonomous property manager.** Deterministic code owns safety gates, policies, clocks, schemas, scope, permissions, and effect integrity. A model is useful for ambiguous language, extraction, comparison, and drafting.
2. **Housing decisions remain human.** The agent does not approve/deny/rank applicants, determine accommodations, infer protected traits, set screening criteria, or issue adverse decisions.
3. **Physical access is a separate trust domain.** Showing or maintenance coordination does not authorize entry. The property agent has no unlock/credential capability.
4. **Property truth is field-level and multi-source.** Unit, listing, lease, occupancy, work-order, address, geocode, parcel, and access state are independent.
5. **Interface breadth is a hazard.** Property platforms expose financial, applicant, lease, vendor, pricing, and work-order surfaces. Internal tools must be much narrower than vendor credentials.
6. **Unknown write outcomes are normal distributed-systems states.** A timeout after submission cannot be retried blindly. Semantic IDs, exact approvals, outbox, read-back, reconciliation, and forward correction are mandatory.
7. **Safety cannot depend on generation.** Fire, gas, electrical, flooding, violence, medical, structural, and other approved triggers need immediate deterministic scripts and human/emergency routing.
8. **Fairness is both a runtime and evaluation architecture.** Runtime blocks protected/proxy inference and ranking. A segregated evaluation environment uses controlled pairs, slices, process/outcome review, and redress; no one metric proves legal compliance.
9. **Memory must be typed and minimal.** Durable workflow state is not a chat memory. Seven memory classes have purpose, retention, deletion, and poisoning controls. Compaction emits a continuity receipt preserving effects, approvals, clocks, conflicts, and provenance.
10. **Version everything that changes behavior.** Model, prompts, tools, policies, safety dictionaries, templates, knowledge, connector adapters, schemas, and eval evidence form an immutable release bundle.

## Primary-source register

### Fair housing, screening, accessibility, and protected requests

| Source | Currency/status checked 2026-08-31 | Material evidence | Design decision |
|---|---|---|---|
| [HUD — Fair Housing Act overview](https://www.hud.gov/helping-americans/fair-housing-act-overview) | current HUD overview | federal protected classes include race, color, national origin, religion, sex, familial status, and disability | deployment registry adds applicable state/local/program classes; production agent never infers them |
| [24 CFR `100.500 — discriminatory effect](https://www.ecfr.gov/current/title-24/subtitle-B/chapter-I/subchapter-A/part-100/subpart-G/section-100.500) | current eCFR checked; see proposed change below | current codified discriminatory-effects framework | current law/configuration comes from dated policy service and counsel, not model memory |
| [2026 proposed rule concerning discriminatory effects](https://public-inspection.federalregister.gov/2026-00590.pdf) | proposed, not treated as effective | proposal would change/remove current regulation | monitor only; never deploy a proposal as current law |
| [HUD 2024 guidance on screening rental applicants](https://www.hud.gov/sites/dfiles/FHEO/documents/FHEO_Guidance_on_Screening_of_Applicants_for_Rental_Housing.pdf) | dated 2024; still hosted and not located in the 2025 withdrawal list reviewed | discusses fair-housing risk in screening, including technology, transparency, accuracy, and individualized practices | use as risk/control input with current legal review; not a substitute for law |
| [HUD 2025 withdrawal of guidance documents](https://www.hud.gov/sites/dfiles/Main/documents/Notice-of-Withdrawal-of-Guidance-Documents.pdf) | dated 2025-09-17 | states listed documents should not be relied on; list includes 2024 digital-platform advertising guidance and other materials | do not cite withdrawn digital-ad guidance as current HUD policy |
| [HUD fair-housing enforcement prioritization memorandum](https://www.hud.gov/sites/dfiles/Main/documents/Fair-Housing-Act-Enforcement-Prioritization-Resources.pdf) | dated 2025-09-16 | states prioritization and supersession of conflicting guidance | current agency priorities are time-sensitive; architecture protects beyond one memo |
| [FTC — Fair Credit Reporting Act](https://www.ftc.gov/legal-library/browse/statutes/fair-credit-reporting-act) | page revised 2026-03; statute/current materials | tenant-screening reports can be consumer reports; FCRA governs permissible purpose, accuracy, notices, disputes, and users/providers | approved CRA handoff, minimum data, dispute/redress, deterministic notice workflow |
| [FTC — Using consumer reports: what landlords need to know](https://www.ftc.gov/business-guidance/resources/using-consumer-reports-what-landlords-need-know) | current FTC business guidance checked | adverse action can include denial, cosigner, larger deposit, or higher rent; notice/provider/free-report/dispute content described | final decision is human; reviewed deterministic adverse-action notice after decision |
| [CFPB — Tenant Background Checks Market Report](https://www.consumerfinance.gov/data-research/research-reports/tenant-background-checks-market-report/) | original CFPB market report | documents matching/accuracy, opaque scoring, and predictive-evidence concerns in the market | provider recommendation is never auto-decision; retain report/ref/dispute state |
| [CFPB — Background screening consumer-reporting advisory](https://www.consumerfinance.gov/rules-policy/final-rules/fair-credit-reporting-background-screening/) | 2024 advisory source | addresses procedures related to duplicate, expunged/sealed, and identity-matched public-record information | dispute/identity/stale record stops and provider qualification tests |
| [CFPB renter adverse-action FAQ](https://www.consumerfinance.gov/ask-cfpb/what-should-i-do-if-my-rental-application-is-denied-because-of-a-tenant-screening-report-en-2105/) | current consumer guidance checked | explains consumer rights to notice, report, and dispute | provide clear human route and evidence; never invent denial reasons |
| [HUD/DOJ joint statement on reasonable accommodations](https://www.hud.gov/sites/documents/huddojstatement.pdf) | foundational official statement; deployment counsel checks current law | explains request/verification concepts and confidentiality/minimization concerns; detailed medical records are generally not the default need | restricted human workflow, minimum supporting information, no model determination |
| [HUD — Assistance animals](https://www.hud.gov/helping-americans/assistance-animals) | current HUD page checked | assistance-animal requests can implicate reasonable-accommodation duties | route to trained restricted process; do not treat as ordinary pet-policy decision |
| [HUD — VAWA](https://www.hud.gov/VAWA) and [VAWA housing rights](https://www.hud.gov/hud-partners/fair-housing-vawa) | current HUD pages checked | covered HUD programs have VAWA protections, confidentiality, emergency-transfer and related processes | separate restricted records/queues; determine coverage; do not universalize to all private housing |
| [W3C — WCAG 2.2](https://www.w3.org/TR/WCAG22/) | W3C Recommendation, republished 2024-12-12 | W3C advises use of WCAG 2.2 | target accessible product design and testing; retain legal applicability mapping |
| [DOJ — Title II web/mobile accessibility rule](https://www.ada.gov/law-and-regs/regulations/title-ii-2010-regulations/) | current DOJ implementation page checked | rule uses WCAG 2.1 AA and specified compliance dates for covered public entities | legal conformance version can differ from latest W3C recommendation; encode both |

### Property, listing, lease, address, and geospatial standards

| Source | Currency/status | Material evidence | Design decision |
|---|---|---|---|
| [RESO Web API](https://www.reso.org/reso-web-api/) | current overview checked | standardizes real-estate data access; RESO does not grant MLS data/permissions | use precise profile mapping only with participant/data rights |
| [RESO certification](https://www.reso.org/certification/) | public current baseline checked | public certification baseline showed Web API Core 2.0.0, Data Dictionary 2.0 and RESO Common Format | certification claim records exact profile/version |
| [RESO ratified transport standards](https://transport.reso.org/) | current standards page checked | Web API Core 2.1.0 is ratified; ratification and certification tooling can differ | pin implemented/certified version; contract-test both sides |
| [RESO Web API Core specification](https://transport.reso.org/proposals/web-api-core/) | official spec | OData/HTTPS/OAuth-based modular API profile | qualify supported query, auth, pagination, metadata, and endorsements |
| [RESO Data Dictionary](https://transport.reso.org/proposals/data-dictionary/) | official spec | standardized resources/fields and extensibility | map field-by-field; preserve extensions/unknown values |
| [OSCRE — Industry Data Model](https://www.oscre.org/Industry-Data-Model/Introducing-the-Data-Model) and [use cases/access](https://www.oscre.org/Industry-Data-Model/Access-To-IDM-Toolkit) | current official overview checked | broad property, lease, space, facility, and occupancy-cost use cases | design-time mapping vocabulary; verify licensing; never treat as authority |
| [USPS Publication 28](https://pe.usps.com/text/pub28/) | October 2024 edition identified | postal addressing/formatting standard | postal normalization does not prove parcel, unit, occupancy, or emergency location |
| [Census Geocoder API](https://geocoding.geo.census.gov/geocoder/Geocoding_Services_API.html) | current API documentation checked | benchmarks/vintages and match behavior are selectable; “Current” moves | pin benchmark/vintage and retain raw match/precision |
| [OGC API — Features](https://www.ogc.org/standards/ogcapi-features/) | official standard page checked | fine-grained feature access; CRS handling is part-specific | use versioned feature/CRS contract and authoritative boundary validation |
| [RFC 7946 — GeoJSON](https://www.rfc-editor.org/rfc/rfc7946.html) | Internet Standard reference | WGS 84 coordinate order is longitude, latitude | enforce axis order and never use coordinate alone as property identity |
| [15 USC 7001 — E-SIGN](https://www.law.cornell.edu/uscode/text/15/7001) | statutory text access | electronic signatures/records cannot be denied effect solely for electronic form; consumer record-consent/access provisions exist | e-sign adapter preserves hashes, consent/evidence, authority and delivery; local review still required |

### Vendor and operational interfaces

| Source | Currency/status | Material evidence | Design decision |
|---|---|---|---|
| [Entrata API documentation](https://docs.entrata.com/api/documentation) | public official docs checked | OpenAPI 3 description; API keys/per-user access, service rate limits and broad property/application/lease/maintenance/vendor/financial surfaces; time defaults require attention | narrow per-operation adapter, explicit permissions, rate/time contract, no finance scope |
| [AppFolio Stack APIs](https://www.appfolio.com/stack/partners/api) | public official API map checked | exposes properties, units, occupancies/tenants, listings, applications, showings, work orders, vendors, attachments and financial surfaces; work-order data includes entry-related metadata | use projections/narrow effects; “permission to enter” never becomes unlock authority |
| [Buildium Open API](https://www.buildium.com/features/open-api/) | current official overview checked | public API access for property-management data/operations, subject to product/contract | tenant-specific qualification; do not assume undocumented semantics |
| [Yardi interfaces](https://www.yardi.com/company/interfaces/) | current official overview checked | supported data exchange through interfaces; detailed contracts can be product/partner gated | qualify exact product/version/tenant; no generic “Yardi connector” claim |
| [IBM Maximo REST work-order example](https://www.ibm.com/support/pages/how-create-service-request-and-follow-work-order-using-rest-api) | official support article modified 2026-07-12 | illustrates service-request/work-order REST integration and status following | pin Maximo product/version/object structure; create via narrow adapter |
| [IBM Maximo Application Suite Admin APIs](https://developer.ibm.com/apis/catalog/maximo--maximo-application-suite-admin-apis/) | current catalog checked | administration APIs exist separately | do not confuse admin/control-plane APIs with domain work-order APIs |

Vendor documentation proves that a capability may exist, not that it is licensed, enabled, authorized, stable, idempotent, or safe in a particular customer account. The [connector guide](../../agents/real-estate-property-operations-agent/06-integrations-connectors-geospatial-building-systems-and-tool-qualification.md) therefore requires tenant-specific evidence.

### Maintenance, inspection, building, access, and safety

| Source | Currency/status | Material evidence | Design decision |
|---|---|---|---|
| [HUD NSPIRE standards](https://www.hud.gov/reac/nspire-standards) and [notices](https://www.hud.gov/reac/nspire-notices) | current HUD program pages checked | inspection standards/deficiencies/correction framework for applicable HUD contexts; review/update process | version checklist by program; never apply NSPIRE universally |
| [OSHA 29 CFR 1910.147 — hazardous energy](https://www.osha.gov/laws-regs/regulations/standardnumber/1910/1910.147) | current OSHA regulation page checked | energy-control program/procedures/training requirements for covered servicing/maintenance | qualified employer/worker process owns LOTO; agent does not clear or direct it |
| [EPA RRP for property managers](https://www.epa.gov/lead/renovation-repair-and-painting-program-property-managers) | updated 2026-05-27 | covered renovation in pre-1978 target housing/child-occupied facilities can require certified firms/renovators and practices | vendor qualification/routing incorporates current applicability |
| [NIST SP 800-82 Rev. 3 — OT security](https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=956505) | final official publication | building automation is OT with safety, reliability, availability and security constraints | isolated read projection; no model/BAS control path |
| [BACnet Committee](https://bacnet.org/) and [BACnet standard information](https://bacnet.org/buy/) | official committee; 135-2024 identified | current BACnet standard, BACnet/SC/security evolution and building-control interoperability | exact deployed profile qualification; interoperability is not safe-control authority |
| [Brick resources](https://brickschema.org/resources/) and [documentation](https://docs.brickschema.org/) | stable 1.4.4 identified, 2025-05-01 | ontology for buildings/equipment/points; stable/nightly distinction | pin stable ontology; use for read semantic mapping |
| [Project Haystack documentation](https://www.project-haystack.org/doc/docHaystack/Intro) | 4.0 documentation dated 2026-07-21 checked | tag/ontology and HTTP operations for building data including reads/history/watch and writes | pin 4.x contract; expose read operations only; treat 5 work as evolving |
| [SIA — Open Supervised Device Protocol](https://www.securityindustry.org/industry-standards/open-supervised-device-protocol/) | official standard page; v2.2.2/2024 identified | supervised access-control reader/controller protocol and secure-channel support | physical access remains separate security system and owner; no agent door command |
| [NIST SP 800-207 — Zero Trust Architecture](https://csrc.nist.gov/pubs/sp/800/207/final) | final official publication | no implicit trust based on network location; explicit policy/identity/resource evaluation | workload identity, least privilege, per-effect authorization, segmented BAS/access zones |

### Security, privacy, payments, competition, AI, and observability

| Source | Currency/status | Material evidence | Design decision |
|---|---|---|---|
| [NIST SP 800-63-4 — Digital Identity Guidelines](https://pages.nist.gov/800-63-4/) | final July 2025; superseded 800-63-3 | separates identity proofing, authentication, and federation assurance; includes privacy/redress and modern attack considerations | risk-based identity, human redress, no identity result auto-converted to housing/access decision |
| [NIST Cybersecurity Framework 2.0](https://www.nist.gov/publications/nist-cybersecurity-framework-csf-20) | final 2024 | Govern plus Identify/Protect/Detect/Respond/Recover framework | governance, risk, incident and recovery mapping |
| [NIST Privacy Framework](https://www.nist.gov/privacy-framework/privacy-framework) | current framework page checked | privacy-risk management framework | data inventory, purpose, minimization, control, communication |
| [NIST AI RMF Generative AI Profile](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf) | final July 2024 | GenAI risks/actions including confabulation, privacy, security, content provenance and evaluation | layered eval, provenance, human oversight, incident/feedback governance |
| [OWASP Top 10 for LLM Applications 2025](https://owasp.org/www-project-top-10-for-large-language-model-applications/assets/PDF/OWASP-Top-10-for-LLMs-v2025.pdf) | 2025 project document | prompt injection, sensitive disclosure, excessive agency, overreliance and related application risks | untrusted content, external enforcement, least tools, output validation, poisoning tests |
| [PCI DSS](https://www.pcisecuritystandards.org/standards/pci-dss/) and [document library](https://www.pcisecuritystandards.org/document_library/) | v4.0.1 current materials identified | controls for environments storing, processing, transmitting, or affecting cardholder data security | use hosted PSP/token references; exclude PAN and agent payment authority; assess scope |
| [FTC Safeguards Rule guide](https://www.ftc.gov/business-guidance/resources/ftc-safeguards-rule-what-your-business-needs-know) | current guide checked | coverage is activity/role dependent and requires an information-security program for covered financial institutions | determine deployment coverage; apply strong controls regardless, without claiming universal applicability |
| [FTC CAN-SPAM compliance guide](https://www.ftc.gov/business-guidance/resources/can-spam-act-compliance-guide-business) | current guide checked | commercial-message requirements and primary-purpose distinctions | classify transactional versus marketing; channel templates and opt-out registry |
| [FCC order on robocall/robotext consent revocation](https://docs.fcc.gov/public/attachments/FCC-24-24A1.pdf) | official 2024 order; later timing/waiver changes require current check | reasonable revocation methods and consent practices are regulated/time-sensitive | current channel/counsel policy registry, not hard-coded model knowledge |
| [DOJ — United States v. RealPage case page](https://www.justice.gov/atr/case/us-and-plaintiff-states-v-realpage-inc) | updated through 2026-07-06 at research time | active case history, settlements/final judgments and allegations concerning rent coordination/data | exclude rent/revenue optimization and nonpublic competitor-data sharing; do not generalize case beyond facts |
| [DOJ RealPage proposed-settlement announcement](https://www.justice.gov/opa/pr/justice-department-requires-realpage-end-sharing-competitively-sensitive-information-and) | official 2025 announcement | describes restrictions involving competitively sensitive information and pricing alignment | strong architectural separation from pricing and competitive data |
| [Counterfactual Fairness — Kusner et al.](https://arxiv.org/abs/1703.06856) | original 2017 paper | formalizes fairness through causal counterfactual invariance | use controlled pair/metamorphic tests; document causal assumptions and limitations |
| [The Sensitivity of Counterfactual Fairness to Unmeasured Confounding](https://arxiv.org/abs/1907.01040) | original research | counterfactual fairness conclusions can be sensitive to unmeasured confounding | do not claim causal/legal proof from paired tests alone |
| [OpenTelemetry specification](https://opentelemetry.io/docs/specs/otel/) and [semantic conventions](https://opentelemetry.io/docs/specs/semconv/) | versions 1.60.0 and 1.44.0 displayed at research time | common traces/metrics/logs model; semantic conventions version independently | OTel-compatible redacted traces with versioned domain attributes |
| [CloudEvents](https://cloudevents.io/) | v1.0.2 current stable identified | interoperable event-envelope fields | use envelope, add versioned property-domain semantics |
| [RFC 9457 — Problem Details](https://www.rfc-editor.org/rfc/rfc9457.html) | current RFC; obsoletes RFC 7807 | standard machine-readable API error format | normalized connector errors with retained vendor evidence |

### Durable workflows, agent runtime, and deployment

| Source | Currency/status | Material evidence | Design decision |
|---|---|---|---|
| [Temporal Workflow Execution](https://docs.temporal.io/workflow-execution) | current official docs checked 2026-08-31 | workflow execution is durable/recoverable; replay checks commands against event history; workflow state and activities are distinct | durable workflow owns progress; deterministic workflow code; model/network calls are activities |
| [Temporal Activity Definition](https://docs.temporal.io/activity-definition) | current official docs checked | activities should be idempotent; an activity may execute more than once, including crash-after-success cases | application-level semantic IDs and destination reconciliation remain mandatory |
| [Temporal retry policies](https://docs.temporal.io/encyclopedia/retry-policies) | current official docs checked | retry policies/backoff are runtime mechanisms, not proof of business idempotency | classify errors and never blindly retry ambiguous writes |
| [OpenAI Agents guide](https://developers.openai.com/api/docs/guides/agents) | current official OpenAI documentation checked | current agent building path, tools/orchestration patterns and SDK support | use structured tools, external policy/effect enforcement, tracing; remain provider-portable |
| [OpenAI trace grading](https://developers.openai.com/api/docs/guides/trace-grading) | current official OpenAI documentation checked | trajectory/trace evaluation complements final-output evaluation | evaluate tool choice, stops, effects and recovery, not answer text alone |
| [OpenAI model guidance](https://developers.openai.com/api/docs/guides/latest-model) | current official guide checked | stresses clear tool descriptions/types/errors, outcome-first instructions, stopping criteria and deliberate compaction/state preservation | type tools, bound plans, and preserve actions/assumptions/IDs/outcomes/blockers across compaction |
| [Kubernetes pod lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/) | current official docs checked | pods are ephemeral; readiness/termination and controller recovery semantics matter | keep durable workflow/outbox outside worker; drain safely |
| [Kubernetes workload autoscaling](https://kubernetes.io/docs/concepts/workloads/autoscaling/) | current official docs checked | supports resource and event-driven scaling | scale on deadline-aware queue/load signals while respecting downstream limits |

The architecture does not require a particular workflow engine, model provider, or orchestrator. These sources establish failure properties and practices that any implementation must satisfy.

## Conflicts and time-sensitive interpretations

### HUD digital-ad guidance was withdrawn

HUD's 2025 notice explicitly lists the 2024 digital-platform advertising guidance among withdrawn documents and says withdrawn documents should not be relied upon. Therefore:

- the playbook does not present that guidance as current HUD policy;
- its prior risk themes may inform historical test design only when clearly labeled;
- current statutes, regulations, official policy, platform behavior, and deployment counsel own requirements.

The 2024 tenant-screening guidance was still hosted and was not found in the reviewed withdrawal list. That does not make it statute or guarantee future status. The policy registry stores source status and review date.

### Current discriminatory-effects rule versus 2026 proposal

The current eCFR contained `24 CFR 100.500` at the research date. A 2026 document proposed a change/removal. A proposed rule is not the current rule. The deployment must monitor final agency action and litigation, then update only through reviewed effective-dated policy.

### WCAG 2.2 versus DOJ Title II's referenced version

W3C recommends WCAG 2.2, while the DOJ Title II web/mobile rule references WCAG 2.1 AA and specified compliance dates for covered public entities. Product best-practice target and legally mandated version can differ. Track both; do not incorrectly say “latest WCAG” is automatically the legal requirement.

### RESO ratification versus certification

RESO's standards page identified Web API Core 2.1.0 as ratified, while its public certification page still described Core 2.0.0/Data Dictionary 2.0 as the active certification baseline. The connector records:

- ratified specification version;
- provider implementation version;
- certification profile/date;
- extensions;
- contract-test evidence.

“RESO compliant” without those fields is rejected.

### Standards versus authority and licensing

RESO does not supply MLS data rights. OSCRE offers a broad data model but its access/licensing and the source system's authority remain separate. Brick/Haystack can label building entities; they do not make telemetry correct or control safe. OSDP/BACnet interoperability does not grant the business right or safety authority to operate a door or building.

### Postal and geospatial observations versus identity

USPS normalization supports mail addressing. Census geocoding returns a match under a benchmark/vintage. GeoJSON encodes coordinates. None proves:

- legal property/parcel boundary;
- internal unit identity;
- current occupancy;
- emergency entrance/location;
- housing-policy jurisdiction.

Use stable source IDs and independently validate each purpose.

### E-sign and tokenization are not magic scope reducers

E-SIGN supports legal recognition of electronic form under its conditions; it does not prove correct contract, party authority, delivery, consent, notarization, or jurisdictional compliance. PCI tokenization can reduce data exposure but does not automatically remove systems/services that can affect cardholder-data security from assessment. The agent sees e-sign/payment references, not raw legal discretion or payment credentials.

### Program standards are not universal

NSPIRE and VAWA sources have specified covered contexts. EPA RRP has activity/building/exception conditions. FTC Safeguards Rule coverage is activity-based. Build applicability matrices instead of applying a federal program page to every property.

### Competition case evidence is fact-specific

The DOJ RealPage matter supports a strong product boundary against rent optimization, competitor nonpublic data, and price coordination. It does not establish that every algorithmic or human pricing method is unlawful. The simplest reliable architecture excludes pricing entirely and routes approved source prices as immutable listing inputs.

## Architecture decision records

### ADR-01 — One durable coordinator per case

**Decision:** one durable state machine owns case progress; model calls and connector calls are bounded activities.

**Why:** property workflows are long-running, event-driven, clock-sensitive, and failure-prone. A transcript or autonomous loop cannot reliably own timers, approvals, unknown writes, or recovery.

**Alternative rejected:** a conversational multi-agent swarm sharing hidden context. It increases coordination, permissions, token cost, nondeterminism, and state conflicts without a distinct trust benefit.

### ADR-02 — Field-level source authority

**Decision:** every consequential field carries source, object/field, version, observed/effective time, status, and freshness.

**Why:** property platforms distribute truth across inventory, listings, leases, occupancy, work orders, documents, channels, accounting, building, and access systems.

**Alternative rejected:** one denormalized “property context” copied into model memory.

### ADR-03 — Model is proposer, not decision/effect authority

**Decision:** the model can extract, classify, summarize, compare, draft, and create inert proposals; deterministic services and authorized humans govern effects.

**Why:** it preserves useful language handling while containing housing, safety, legal, financial, and physical consequences.

### ADR-04 — Screening is reference-only handoff

**Decision:** use an approved provider, preserve permissible-purpose/consent/report/dispute references, and require authorized human decision. The production agent never ranks or decides.

**Why:** consumer-report accuracy, identity, opaque scores, disputes, FCRA/adverse action, and fair-housing risks make autonomous screening unsuitable.

### ADR-05 — No physical-access authority

**Decision:** coordinate windows/notice/escort requirements and read redacted arrival status only. Door commands and credentials stay in a separate authorized system.

**Why:** access is high-consequence, weakly reversible, safety/security sensitive, and distinct from maintenance/showing scheduling.

### ADR-06 — Prepare–authorize–commit–verify

**Decision:** exact intent hash, source versions, policy, approver, expiry, semantic ID, outbox, destination receipt/read-back, and reconciliation are required.

**Why:** distributed writes may execute when the response is lost; transport retry alone cannot guarantee one business effect.

### ADR-07 — Unknown is durable state

**Decision:** post-submission ambiguity transitions to `effect_unknown` and suppresses retries until reconciliation proves success or absence.

**Why:** this prevents duplicate work orders, listings, envelopes, and communications.

### ADR-08 — Deterministic safety before model

**Decision:** emergency phrases/signals trigger approved instructions and human escalation without waiting for a model.

**Why:** latency, outage, prompt injection, multilingual uncertainty, and diagnosis risk make generation inappropriate for the critical first response.

### ADR-09 — Typed memory and loss-aware compaction

**Decision:** seven memory classes; compact to a validated continuity receipt preserving scope, versions, completed/inflight effects, approvals, clocks, conflicts, blockers, and next actions.

**Why:** chat-style memory creates privacy, poisoning, cross-tenant, stale-state, and lost-effect risks.

### ADR-10 — Fairness controls outside the model

**Decision:** runtime removes protected traits/proxies from ranking and uses fixed queries/templates/rules; segregated eval performs counterfactual and slice tests under governance.

**Why:** prompts alone cannot prevent proxy use or prove equivalent service.

### ADR-11 — Immutable behavior bundles

**Decision:** code, model route, prompts, schemas, tools, policies, templates, knowledge, connectors, eval evidence, and migrations are one signed version.

**Why:** any component can change behavior and must be shadowed/canaried/rolled back coherently.

### ADR-12 — Minimum initial slice

**Decision:** begin with non-emergency maintenance intake plus human-approved work-order creation and post-verification acknowledgement.

**Why:** it exercises core reliability and safety controls without delegating screening, legal, pricing, accounting, or access authority.

## Traceability from requirement to guide

| Requirement | Primary guide |
|---|---|
| property/unit/occupancy truth and schemas | [03](../../agents/real-estate-property-operations-agent/03-property-unit-occupancy-listing-and-lease-contracts.md) |
| listings/showings/applications/screening/leases/renewals | [04](../../agents/real-estate-property-operations-agent/04-listings-showings-applications-screening-and-leases.md) |
| communications/maintenance/vendors/inspections/access/safety/SLA/portfolio | [05](../../agents/real-estate-property-operations-agent/05-tenant-communications-maintenance-vendors-inspections-and-access.md) |
| control/data/effect planes and runtime | [02](../../agents/real-estate-property-operations-agent/02-reference-architecture-runtime-and-control-planes.md) |
| connectors, property standards, geospatial, building/access systems | [06](../../agents/real-estate-property-operations-agent/06-integrations-connectors-geospatial-building-systems-and-tool-qualification.md) |
| events/effects/approvals/idempotency/unknown/reconciliation/compensation | [07](../../agents/real-estate-property-operations-agent/07-state-events-effects-approvals-reconciliation-and-recovery.md) |
| all memory classes/context/compaction/planning/security/fair housing | [08](../../agents/real-estate-property-operations-agent/08-context-memory-planning-security-privacy-and-fair-housing.md) |
| eval/counterfactual/sim/fault/observability/SLO/incidents | [09](../../agents/real-estate-property-operations-agent/09-evaluation-observability-slos-failure-injection-and-incidents.md) |
| deployment/scale/queues/cost/outage/DR/release/drift/feedback | [10](../../agents/real-estate-property-operations-agent/10-deployment-scale-cost-dr-release-and-governed-evolution.md) |
| bounded authority and deterministic alternatives | [01](../../agents/real-estate-property-operations-agent/01-mission-boundaries-authority-and-stages.md) |
| Stage 0–6, exercises, 100-point gates | [11](../../agents/real-estate-property-operations-agent/11-zero-to-production-roadmap-exercises-and-gates.md) |

## Refresh triggers

Re-run focused research before:

- a new jurisdiction, housing program, property class, lease form, or notice type;
- any screening provider/product, criteria, adverse-action workflow, or dispute process change;
- HUD/DOJ/FTC/CFPB/FCC/EPA/OSHA rule, guidance, enforcement, or court-status change;
- final action on the 2026 proposed `24 CFR 100.500` change;
- new RESO certification baseline, OSCRE license/use, or listing-channel profile;
- PMS/CMMS/e-sign/payment/communication API version, permission, idempotency, rate, time, webhook, or contract change;
- BACnet/Brick/Haystack/access-system integration or any proposed control effect;
- model/provider/tool/retrieval/compaction/agent SDK change;
- new memory purpose, protected/restricted dataset, or cross-border data movement;
- new pricing, accounting, lending, appraisal, eviction/legal, or physical-access request—these require a separate product/architecture decision, not scope creep;
- material incident, fairness drift, safety miss, provider outage, or DR failure.

Review federal and deployment-specific policy at least quarterly, connector contracts/releases at each release and at least quarterly, safety scripts at local operations cadence, vendor qualifications at effective-date/dispatch time, and control-plane access/authority at least quarterly.

## Limitations

- This packet does not inventory any specific state, locality, tribal jurisdiction, country, rent-control regime, consumer-protection rule, brokerage/property-management license, building/fire code, lease, insurer requirement, or housing-program contract.
- Public vendor pages do not expose every product/partner/tenant contract. Exact OpenAPI schemas, scopes, idempotency, consistency, webhooks, rate limits, timezones, data residency, deletion, and sandbox fidelity require commercial documentation and contract tests.
- Some official guidance is changing or contested. The packet deliberately records status conflicts rather than predicting outcomes.
- No fairness metric or simulation proves nondiscrimination; causal results depend on assumptions and legal review.
- Building telemetry, postal/geospatial data, and AI output cannot certify safety, code compliance, ownership, occupancy, or entry authority.
- Illustrative SLO/RPO/RTO/readiness thresholds in the playbook require portfolio baselines and risk-owner approval.
- The playbook is provider-neutral. Product-specific implementation, data-flow assessment, threat model, DPIA/privacy analysis, legal review, accessibility testing, penetration testing, and operational drills remain deployment work.

## Research quality checklist

- [x] Current primary federal housing, consumer-reporting, accessibility, safety, payment, competition, identity, security, AI, and communications sources reviewed.
- [x] Official property/listing, address/geospatial, building, access, event, observability, durable-workflow, and deployment standards/docs reviewed.
- [x] Official public property-management/work-order vendor interfaces compared.
- [x] Current versus withdrawn/proposed guidance explicitly distinguished.
- [x] Ratified versus certification/implementation versions distinguished.
- [x] Production failure modes, safety, fairness, privacy, cost, recovery, and provider outage converted into design controls.
- [x] Competing deterministic/no-agent and model-assisted approaches documented.
- [x] Strong sources linked rather than copied.
- [x] Refresh triggers and unresolved deployment-specific work recorded.

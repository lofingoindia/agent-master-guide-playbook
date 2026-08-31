# Clinical-Trial Operations Agent Blueprint — Research Packet

> **Research status:** production-depth Pass 2 synthesis complete  
> **Research cut-off:** 2026-08-31  
> **Primary artifact:** [Clinical-Trial Operations Agent](../../agents/clinical-trial-operations-agent/README.md)  
> **Claim level:** engineering architecture guidance, not legal, medical, regulatory, ethics, pharmacovigilance, biostatistical, validation, or compliance advice

This packet records the primary sources, current-version checks, operational evidence, contradictions, design decisions, limitations, and refresh triggers used to create the clinical-trial-operations-agent guide set. It separates sourced requirements and prudent engineering inferences from unsupported claims that an AI system is “GCP compliant,” clinically valid, inspection ready, or safe.

## 1. Research scope

The research asked how to build bounded AI assistance for regulated human-study operations while preserving:

- sponsor, study, protocol, amendment, jurisdiction, site, participant, visit, and record identity;
- sponsor, CRO, investigator, delegate, monitor, data-management, safety, pharmacy, vendor, and auditor roles;
- consent-process and eligibility evidence without model-owned decisions;
- EDC, eTMF/ISF, CTMS, IRT, laboratory, eCOA/DHT, safety, registry, and submission boundaries;
- source, metadata, audit-trail, essential-record, derived-artifact, acknowledgement, and reconciliation provenance;
- monitoring, queries, deviations, serious-breach triage, CAPA handoff, and inspection reconstruction;
- safety intake, case preparation, deterministic jurisdiction clocks, qualified review, transmission, follow-up, and acknowledgement;
- privacy, participant identity, purpose, consent/authorization, blinding, security, and supplier governance;
- risk-based computerized-system validation and AI lifecycle assurance;
- durable runtime state, context/compaction, memory, planning, idempotency, recovery, and change control; and
- production evaluation, observability, SLOs, incidents, capacity, multi-site rollout, cost, and decommissioning;
- operation-level qualification for EDC, CTMS, eTMF, eConsent/e-signature, IRT/RTSM, safety, laboratory, eCOA/DHT,
  identity, document/storage/workflow, notification, registry, and submission-handoff adapters; and
- realistic consent, eligibility, monitoring/query, safety-report-preparation, and regulatory-package flows including
  receipts, ambiguity, cancellation, reconciliation, recovery load, and human release authority.

Explicit exclusions were scientific experimental reasoning, ordinary patient-service coordination, generic document extraction, rule-monitoring ownership, clinical/medical judgment, eligibility decisions, dose decisions, randomization/unblinding authority, protocol approval, electronic signature, data lock, and filing authority.

## 2. Repository context reviewed

The blueprint follows the category registry and Stage 0–6 expansion program and reuses the repository's canonical controls:

- [Execution boundaries](../../runtime/execution-boundaries.md), [durable execution](../../runtime/durable-execution.md), [run controls](../../runtime/run-controls.md), and [state/event contracts](../../runtime/agent-state-and-event-contracts.md)
- [Tool contracts](../../tools/tool-contracts.md), [tool-result provenance](../../tools/tool-results-artifacts-and-provenance.md), and [tool lifecycle](../../tools/tool-registries-versioning-and-lifecycle.md)
- [Context engineering](../../context-memory/context-engineering.md), [memory architecture](../../context-memory/memory-architecture.md), and [compaction continuity](../../context-memory/compaction-and-continuity.md)
- [Agent threat model](../../security/agent-threat-model.md), [permissions and secrets](../../security/permissions-sandboxing-and-secrets.md), and [prompt-injection defense](../../security/prompt-injection-and-untrusted-data.md)
- [Idempotency and effects](../../reliability/idempotency-and-side-effects.md) and [failure taxonomy](../../reliability/failure-taxonomy.md)
- [Evaluation-driven development](../../evaluation/evaluation-driven-development.md), [trajectory evaluation](../../evaluation/trajectory-and-reliability-evaluation.md), and [observability](../../evaluation/observability-and-tracing.md)
- [Deployment and incidents](../../operations/deployment-release-and-incident-response.md), [queues/backpressure](../../operations/queues-scheduling-and-backpressure.md), [scaling/SLOs](../../operations/scaling-capacity-and-slos.md), and [cost/latency](../../operations/model-routing-cost-and-latency.md)

Adjacent Scientific Research, Document Intelligence, and Regulatory Intelligence blueprints were reviewed to keep ownership boundaries explicit. No shared registry, index, navigation file, or other agent area was edited for this delegated task.

## 3. Method and evidence discipline

### 3.1 Source hierarchy

Research prioritized:

1. ICH final guidelines and official implementation status;
2. current regulator legislation, regulation text, final guidance, systems, and inspection evidence;
3. ethics/public-health institutions and privacy authorities;
4. official interoperability, terminology, identity, cybersecurity, and AI-risk standards; and
5. draft guidance only as a clearly labeled refresh input, never as current binding implementation.

The packet uses engineering inference where official sources define responsibilities or record properties but do not prescribe an agent architecture. Such inference is identified in the decision record and must be validated locally.

### 3.2 Research angles

| Angle | Questions investigated |
|---|---|
| GCP and quality | What do current ICH E6(R3)/E8(R1) say about roles, quality by design, records, computerized systems, service providers, data governance, and validation? |
| Protocol lifecycle | What do M11/USDM support, and what approvals/effectivity work remains organizational and jurisdictional? |
| Participant protection | How do consent process, privacy authorization/lawful basis, eligibility, and investigator medical authority differ? |
| Safety | Which clocks, determinations, exchange formats, terminology, acknowledgements, and role boundaries must be deterministic or human-owned? |
| Records and monitoring | What makes source/audit/TMF evidence reconstructable, and how should risk-based monitoring, queries, deviations, breaches, and CAPA behave? |
| Integrations | What do CDISC/FHIR/registry standards actually cover, and what maturity/version limitations matter? |
| Adapter qualification | What does each exact operation, receipt, callback, version, tenant configuration, role and failure mode prove or fail to prove? |
| Security and privacy | How should workload identity, delegated study authority, least privilege, blinding, purpose limitation, and vendor boundaries be designed? |
| AI and validation | How should context of use, lifecycle risk, evaluation, human factors, release changes, and evidence be handled? |
| Production | How do durable effects, reconciliation, failure injection, SLOs, incidents, capacity, and cost change in a multi-site regulated workflow? |

### 3.3 Synthesis rule

A standard or regulator source was not stretched beyond its scope:

- ICH GCP principles did not become a claim that one architecture is legally required;
- a structured protocol format did not become protocol approval or site effectivity;
- an exchange standard did not become a source-of-truth or semantic-completeness guarantee;
- an audit trail did not become proof the underlying data are correct;
- a public registry API did not become a filing API;
- model confidence did not become clinical authority; and
- a vendor qualification or system validation did not make every configured study use valid forever.

## 4. Executive findings

1. **The safest useful shape is a durable workflow with bounded model steps.** Identity, authorization, protocol effectivity, clocks, effects, reconciliation, and audit evidence remain deterministic application responsibilities.
2. **Human authority is not transferable to model confidence.** Current GCP assigns investigator and sponsor responsibilities and allows delegation/transfer with retained oversight; it does not authorize an AI to make eligibility, dosing, medical, safety, protocol, or participant-protection decisions.
3. **An approved protocol document is not an operational release.** Site/jurisdiction approvals, training, consent versions, participant transition, and EDC/IRT/CTMS/safety configurations need a separate governed release manifest.
4. **Consent is a process, not a file state.** Participation consent, privacy authorization/lawful basis, optional permissions, assent/LAR, re-consent, and withdrawal need distinct records and policies.
5. **Safety operations require independent deterministic clocks.** The model can assemble case evidence and draft narratives; qualified medical/PV reviewers determine classification/reportability, and authorized channels transmit and reconcile acknowledgements.
6. **Blinding must be enforced in architecture.** Storage, identity, credentials, tools, compute, context, telemetry, and evaluation must prevent direct and inferential treatment disclosure. Emergency unblinding needs a direct human path.
7. **Source systems stay authoritative.** Agent projections are derived views. Events/webhooks can be duplicated, delayed, or incomplete; refetch and periodic bounded reconciliation are necessary.
8. **Regulated records require complete provenance and exportability.** Preserve original values, metadata, audit histories, reason for change, versions, actors, timestamps, migrations, and decommissioning evidence.
9. **Risk-based monitoring is prioritization, not automated accusation.** Signals need denominators, watermarks, false-positive analysis, site context, and qualified interpretation.
10. **Deviation, serious breach, and CAPA are different objects.** Immediate safety/reporting actions must not wait for finished root-cause analysis, and task completion is not CAPA effectiveness.
11. **Current standards are useful but not universal.** ODM, USDM, SDTM, E2B, MedDRA, and FHIR have different purposes, maturity, versions, and local profiles. Pin and test the exact versions used.
12. **AI assurance is context-of-use and lifecycle based.** Validate the configured workflow, people, integrations, failure modes, and release—not merely a base model or generic benchmark.
13. **Inspection readiness depends on independent evidence planes.** Unsampled control/audit evidence must remain available without the model; diagnostic traces should be redacted and minimized.
14. **Production scale is governed by human and destination capacity too.** Safety review, site follow-up, gateway throughput, reconciliation, and amendment waves can bottleneck before model inference.
15. **A connector name is not a capability.** Admission is per operation, configured tenant/application, API/schema,
    role, region, scope, blinding partition, receipt semantics, negative tests, expiry, and disable control.
16. **Operational identity includes more than participant and study.** Protocol release, site-study assignment, visit,
    specimen, safety case, deviation/issue, query, artifact, external effect, versions and effective times are mandatory
    joins; fuzzy repair is unsafe.
17. **Provider completion is narrower than clinical or regulatory completion.** A workflow completion, signature state,
    message delivery, object lock, upload response or gateway acknowledgement does not by itself establish consent,
    clinical correctness, GCP record completeness, human approval, filing release or regulator acceptance.
18. **Continuity uses exactly seven governed memory lifetimes.** A loss-aware compaction receipt preserves source/event
    watermarks, exact behavior/protocol/policy/tool/model versions, approvals, clocks, pending/unknown effects, blinding
    invariants, omissions and the next safe action; a prose summary is never the record.
19. **Recovery capacity is a separate operating limit.** Restores must reserve safety/clock capacity and reconcile
    unknown effects, callbacks, sources, approvals and artifacts before unconstrained redrive or new bulk work.

## 5. Decision record

| ID | Blueprint decision | Evidence basis | Important limit |
|---|---|---|---|
| CT-01 | Use a deterministic durable workflow with bounded model drafting/normalization | ICH E6(R3) responsibility, data-governance, computerized-system, and validation principles plus repository runtime controls | An engineering inference; each intended use needs local validation |
| CT-02 | Keep clinical, eligibility, dosing, safety, protocol, filing, and participant-protection authority with qualified humans | ICH E6(R3), FDA sponsor/investigator duties, EMA GCP Q&A, consent ethics sources | Exact role/qualification depends on protocol, jurisdiction, and institution |
| CT-03 | Resolve effect authorization from principal, delegation, study/site/participant scope, protocol release, purpose, time, jurisdiction, and blinding | GCP responsibility/delegation plus NIST identity/least-privilege controls | Policy model must map local organization and contracts |
| CT-04 | Separate protocol artifact/version from site- and participant-effective release manifest | ICH E6(R3) records/configuration/validation; M11/USDM scope; amendment rules | Release workflow and approval evidence are jurisdiction/study specific |
| CT-05 | Keep site identity mapping local and use pseudonymous participant-study IDs outside it | Privacy minimization/purpose principles and GCP confidentiality | Safety/regulatory uses may require governed identity disclosure |
| CT-06 | Model consent as process, document, privacy basis/authorization, optional permissions, capacity/LAR, and withdrawal records | OHRP/FDA consent and EDPB CTR/GDPR interplay | Not a universal legal-basis decision tree |
| CT-07 | Let the agent map evidence to eligibility criteria but expose no final eligibility API | EMA investigator responsibility and participant-protection principles | Rule compilation and clinician review still require validation |
| CT-08 | Compile visit windows and safety clocks into deterministic versioned services | Protocol exactness and regulator reporting clocks | Exact clock triggers must be reviewed per applicable jurisdiction |
| CT-09 | Split safety intake, clock, source collection, duplicate search, medical determination, submission authorization, transmission, acknowledgement, and follow-up | ICH E2A/E2B, FDA 21 CFR 312.32, EMA/EV reporting | Device, post-marketing, ethics, and local rules need separate profiles |
| CT-10 | Architect blinded/unblinded partitions and keep emergency unblinding independent of the agent | ICH E9(R1), EMA computerized systems, EMA GCP Q&A | Specific blinding design varies; inference attacks need local testing |
| CT-11 | Preserve source and destination authority; treat agent summaries as derived artifacts | ICH E6(R3), FDA electronic source/records, EMA computerized-systems/TMF guidance | A source can still be wrong; corrections must remain traceable |
| CT-12 | Use typed versioned adapters and reject generic browser filing, direct database writes, and free-form tools | Regulator portals/gateways and repository effect/security controls | Qualified official APIs may justify a narrowly validated adapter |
| CT-13 | Use semantic effect IDs, explicit `UNKNOWN`, postcondition checks, acknowledgements, and reconciliation | External-system ambiguity plus E2B/gateway acknowledgement behavior | Destination capabilities determine strongest attainable proof |
| CT-14 | Treat monitoring signals as review prioritization, not compliance conclusions | ICH E6(R3) quality/risk approach and FDA risk-based monitoring | Signal validity and bias must be established per study/site context |
| CT-15 | Keep issue, deviation candidate, important deviation, serious-breach candidate, CAPA, and effectiveness check separate | ICH E6(R3), EU/UK serious-breach guidance | Classification/reporting remains qualified and jurisdictional |
| CT-16 | Maintain an unsampled authoritative evidence plane separate from redacted diagnostic telemetry | GCP/Part 11/EMA audit requirements plus privacy/security guidance | Storage design alone does not prove record completeness |
| CT-17 | Validate intended use, configured standard functions, study configuration, interfaces, migrations, security, backup/restore, and changes | ICH E6(R3), FDA electronic systems, EMA computerized systems | Validation depth is risk- and deployment-specific |
| CT-18 | Evaluate outcomes, evidence, trajectories, invariants, human factors, recovery, privacy, and cost; make severe failures noncompensating | FDA/EMA AI lifecycle principles plus GCP risk focus | Thresholds require clinical/quality/statistical justification |
| CT-19 | Pin model, prompt, projection, tools, adapters, schemas, rules, terminology, and eval suite in a behavior release | AI lifecycle/change-control and computerized-system validation guidance | Vendor-managed changes need contractual detection and control |
| CT-20 | Use ODM/USDM/SDTM/E2B/FHIR through explicit local profiles and maturity/version tests | Official standards scopes and maturity labels | No profile provides universal cross-vendor semantics |
| CT-21 | Default memory to durable task state plus approved versioned domain knowledge; reject open-ended person memory | Privacy minimization, source-system authority, repository memory controls | Local retention and secondary-use governance still required |
| CT-22 | Reserve capacity for safety and participant-protection work and isolate by sponsor/study/site/blinding | Reporting clocks, privacy, operational reliability | Capacity and cell topology depend on real load and residency constraints |
| CT-23 | Keep independent stop, manual safety, evidence preservation, reconciliation, and bounded redrive controls | GCP participant protection, incident and data-integrity evidence | Runbooks must be exercised with actual systems and people |
| CT-24 | Earn capability through Stage 0–6 gates; expanding authority or intended use reopens prior validation | Repository expansion contract and lifecycle sources | A stage number is not regulator approval or compliance certification |
| CT-25 | Admit adapters through expiring operation-level capability manifests, not vendor-level allowlists | Official provider APIs expose role, version, object, receipt and configuration-specific semantics | A manifest still needs tenant/configuration UAT and sponsor validation |
| CT-26 | Bind visit, specimen, case, deviation/query, artifact and effect identities with versions, effective intervals and blinding partitions | GCP provenance plus EDC/safety/document provider semantics | Exact native mappings remain deployment-specific |
| CT-27 | Keep e-signature completion separate from informed-consent-process verification and investigator responsibility | FDA/OHRP consent guidance plus DocuSign's narrower provider event/schema semantics | Authentication and signature meaning vary by procedure and jurisdiction |
| CT-28 | Use exactly seven memory lifetimes and a loss-aware, digest-verified continuity receipt | Privacy minimization, source authority and repository continuity controls | Retention/deletion/hold rules require local record classification |
| CT-29 | Keep package upload, approval, release and destination acceptance distinct; default registry `autoRelease` off | ClinicalTrials.gov PRS current responsible-party workflow and external-upload semantics | Other portals and jurisdictions require separate qualification |
| CT-30 | Restore safety/clocks, authority/blinding and effect reconciliation before lower-risk admission/redrive | GCP participant protection plus distributed-effect ambiguity and capacity engineering | Recovery order and capacity need live exercises and qualified owners |

## 6. Contradictions, version drift, and resolutions

### 6.1 ICH E6(R3) final text versus regional implementation

**Finding:** ICH E6(R3) Principles and Annex 1 reached Step 4 on 2025-01-06, with corrections dated 2025-10-24. Regional effective dates and implementation details differ. EMA identifies EU applicability of Principles/Annex 1 from 2025-07-23. Annex 2 reached Step 4 on 2026-06-03, while EMA identifies an EU effective date of 2027-01-15.

**Resolution:** Cite the final ICH text for international principles, but deploy jurisdiction policy only from confirmed regional implementation. Annex 2 is a current final ICH source and a future EU implementation input at the research cut-off; it is not silently treated as already effective everywhere.

### 6.2 Structured protocol exchange versus approved operational protocol

**Finding:** ICH M11 and CDISC USDM improve structured protocol content and electronic exchange. M11 explicitly does not govern protocol development/maintenance or trial design. Neither standard represents ethics/regulatory approval, site activation, configuration validation, training, or participant transition.

**Resolution:** The blueprint uses a separate protocol release manifest that binds an immutable approved artifact to approvals, jurisdiction/site effectivity, participant rules, consent versions, and qualified EDC/IRT/CTMS/safety builds.

### 6.3 Informed consent versus data-processing permission

**Finding:** FDA/OHRP sources describe informed consent as a human information and voluntary-decision process. The EDPB/European Commission analysis distinguishes CTR participation informed consent from GDPR legal basis for processing. HIPAA authorization may also be a distinct US record.

**Resolution:** Store participation consent, privacy basis/authorization, optional permissions, assent/LAR, and withdrawal separately. Do not infer that withdrawal requires destruction of every retained regulated record; route the exact request through approved privacy/clinical/legal policy.

### 6.4 Investigator medical authority versus workflow delegation

**Finding:** GCP allows activities to be delegated and sponsor tasks to be transferred, but sponsor/investigator responsibilities and oversight remain. EMA GCP Q&A emphasizes that investigator medical decisions include eligibility/enrollment, safety/efficacy, test results, and dose changes.

**Resolution:** The agent and CRO/vendor roles may prepare evidence or perform delegated operations, but retained decisions require current qualified human authority. Workload identity and contract assignment are not clinical qualification.

### 6.5 Safety clocks and awareness triggers

**Finding:** US IND safety reporting includes 15-day and 7-day pathways with specific trigger language. EU/EEA safety, serious-breach, urgent-measure, annual-report, ethics, and investigator communications use different events/channels; UK serious-breach guidance effective 2026-04-28 specifies seven days from sponsor awareness and warns not to wait for full investigation.

**Resolution:** Store all relevant receipt, awareness, minimum-information, determination, transmission, and acknowledgement timestamps. A reviewed jurisdiction-policy service instantiates each clock; prose examples are never executable rules.

### 6.6 “Exactly once” filing versus real gateway behavior

**Finding:** E2B-compatible safety exchange and EudraVigilance include acknowledgement/error flows. Network failure can leave the sender uncertain whether a destination accepted a message. Similar ambiguity exists in EDC, IRT, CTMS, eTMF, and registry writes.

**Resolution:** Promise neither end-to-end exactly-once delivery nor safe blind retry. Persist intent, use a semantic effect ID, query postconditions/acknowledgements, hold `UNKNOWN`, and reconcile before bounded authorized retry.

### 6.7 Audit trails versus data correctness

**Finding:** ICH, FDA, EMA, and MHRA sources require attributable changes, audit histories, reason for change, protected records, metadata, and retrievability. An audit trail records change history; it cannot prove that the original entry, transformation, or interpretation was correct.

**Resolution:** Preserve audit history and separately validate source origin, mappings, transformations, completeness, review, reconciliation, and scientific/clinical meaning.

### 6.8 Risk-based monitoring versus reduced oversight

**Finding:** ICH and FDA encourage risk-proportionate monitoring focused on participant protection and data reliability. Risk-based does not mean model-only, remote-only, or fewer controls regardless of observed issues.

**Resolution:** Use signals to allocate qualified review, document study-specific rationale, adapt the monitoring plan based on findings, and retain sponsor oversight of investigators and service providers.

### 6.9 FHIR availability versus trial-operations maturity

**Finding:** HL7 FHIR R5 `ResearchStudy` and `ResearchSubject` are maturity level 0 and Trial Use. ClinicalTrials.gov offers a FHIR representation built on a newer FHIR build, but the public service is a data representation, not a trial-operations or filing authority.

**Resolution:** Do not use unprofiled FHIR resources as the canonical study-operations model. Adopt only explicit profiles with conformance, mapping, round-trip, version, and partner tests; retain source-system identity and semantics.

### 6.10 ODM 2.0 and installed vendor compatibility

**Finding:** CDISC ODM 2.0 is a current vendor-neutral exchange/archival standard with substantial changes from earlier versions. Existing EDCs and partners may support different ODM versions or subsets.

**Resolution:** Pin the interface version/profile, preserve vendor-native exports, qualify mappings, test audit/metadata round trips, and do not equate nominal ODM support with interoperability.

### 6.11 TMF Reference Model stable and project versions

**Finding:** CDISC's TMF Reference Model resources identify v3.3.1 as a stable released reference, while v4 is an active project at the research cut-off.

**Resolution:** Pin the version actually adopted by the sponsor/eTMF. Treat v4 as a change-monitoring input until the selected release and vendor support are qualified; a reference model does not replace the sponsor's record-location map or filing procedures.

### 6.12 Final guidance versus drafts

**Finding:** FDA's 2024 protocol-deviations guidance and 2025 AI regulatory-decision-support guidance were draft at the research cut-off. FDA's 2024 electronic-systems Q&A and 2023 informed-consent/DHT guidance are final.

**Resolution:** Final sources inform the deployed baseline. Draft sources inform future-change analysis and useful engineering questions, clearly labeled; they do not silently create current requirements.

### 6.13 AI performance versus validated intended use

**Finding:** FDA/EMA AI principles emphasize human-centric, risk-based, context-of-use, data governance, performance, and lifecycle controls. No generic model benchmark validates a configured clinical-trial workflow.

**Resolution:** Validate the full local workflow and severe failure slices. Model confidence is never substituted for evidence completeness, clinical review, authorization, or an effective protocol/policy release.

### 6.14 ODM standard currency versus current EDC interface

**Finding:** CDISC ODM 2.0 is current, while OpenClinica's current documented clinical-data interface uses JSON or ODM
1.3.2 with OpenClinica extensions and exposes options for audits, discrepancy notes, archived forms and wildcard scope.

**Resolution:** Pin and qualify the installed EDC endpoint/profile. Never transform “supports ODM” into version
compatibility, complete audit export, business idempotency, or safe population scope.

### 6.15 Provider receipt versus regulated or human meaning

**Finding:** DocuSign, Twilio and Camunda expose useful envelope/message/process states. Those states describe provider
or engine observations, not informed-consent quality, participant comprehension, the correct recipient, sponsor
oversight, a clinical decision, a required record, or destination acceptance.

**Resolution:** Persist the receipt with its exact semantics, then reconcile the separately authorized clinical-
operations state and evidence. Provider status is necessary evidence only where the qualified procedure says it is.

### 6.16 PRS upload versus responsible-party release

**Finding:** The ClinicalTrials.gov PRS guide updated 2026-05-01 distinguishes entry completion, approval and release;
its External Upload API exposes `autoRelease`, while completeness and accuracy responsibility remains with the submitting
organization/responsible party.

**Resolution:** Default `autoRelease` off, stage and reconcile the upload, show a human-readable diff, and keep approval
and release as separately authorized human effects.

### 6.17 Object immutability versus validated archive

**Finding:** S3 Object Lock requires versioning and applies retention/legal holds to object versions; governance mode
has a controlled bypass permission. WORM storage does not classify a record or prove metadata, signature, retention,
access, migration, restore, or inspection retrieval.

**Resolution:** Qualify object/version identity, mode, permissions, hold/retention policy, metadata, restore, export and
decommissioning as one record-system use. Do not market an object-store feature as GxP compliance.

### 6.18 Newest API versus production-stable API

**Finding:** Veeva Vault publishes multiple API versions annually and labels the newest version Beta while older
versions are stable. Exact features also depend on the configured Vault/application.

**Resolution:** Pin a selected GA API and application/configuration capability. Track Beta/GA/deprecation and refresh
the dossier when provider, release, configuration, role or field semantics change.

## 7. Authoritative source register

All sources were checked on or before 2026-08-31. Dates below describe the current document/release when material, not the access date.

### 7.1 GCP, protocol, ethics, and participant protection

| Source | Current status at cut-off | Blueprint use |
|---|---|---|
| [ICH E6(R3) Principles and Annex 1](https://database.ich.org/sites/default/files/ICH_E6%28R3%29_Step4_FinalGuideline_2025_0106_ErrorCorrections_2025_1024.pdf) | Step 4, 2025-01-06; corrections 2025-10-24 | Roles/oversight, quality by design, essential records, computerized systems, validation, audit, security, service providers |
| [ICH E6(R3) Annex 2](https://database.ich.org/sites/default/files/ICH_E6%28R3%29_Annex%202_Guideline_Step%204_2026_0603_0.pdf) | Step 4, 2026-06-03 | Decentralized/pragmatic/RWD trial considerations and service-provider/data governance |
| [EMA ICH E6 implementation page](https://www.ema.europa.eu/en/ich-e6-good-clinical-practice-scientific-guideline) | Current regional dates | Distinguishes final ICH text from EU effective dates |
| [ICH E8(R1)](https://database.ich.org/sites/default/files/ICH_E8-R1_Guideline_Step4_2021_1006.pdf) | Step 4, 2021 | Quality by design and critical-to-quality factors |
| [ICH M11 guideline](https://database.ich.org/sites/default/files/ICH_Step4_M11_Final_Guideline_2025_1119.pdf) and [technical specification](https://database.ich.org/sites/default/files/ICH_Step4_M11_Final_TechnicalSpecification_2025_1119.pdf) | Step 4, 2025-11-19 | Structured protocol content/exchange scope and limits |
| [ICH E9(R1)](https://database.ich.org/sites/default/files/E9-R1_Step4_Guideline_2019_1203.pdf) | Step 4, 2019 | Randomization/blinding and estimand context |
| [WMA Declaration of Helsinki](https://www.wma.net/policies-post/wma-declaration-of-helsinki/) | Current 2024 revision | Ethical baseline for human participants |
| [HHS 45 CFR 46](https://www.hhs.gov/ohrp/regulations-and-policy/regulations/45-cfr-46/index.html) and [OHRP informed-consent FAQ](https://www.hhs.gov/ohrp/regulations-and-policy/guidance/faq/informed-consent/index.html) | Current US HHS sources | IRB/consent process and human responsibility |
| [FDA Informed Consent guidance](https://www.fda.gov/media/88915/download) | Final, August 2023 | Consent process/documentation and participant communication |
| [FDA Digital Health Technologies for Remote Data Acquisition](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/digital-health-technologies-remote-data-acquisition-clinical-investigations) | Final, December 2023 | Fit-for-purpose DHT selection, verification/validation, usability, security/privacy, and data handling |
| [FDA Conducting Clinical Trials With Decentralized Elements](https://www.fda.gov/media/167696/download) | Final guidance | Remote visits, local providers, adverse-event handling, records, and participant access considerations |
| [CIOMS ethical guidelines](https://cioms.ch/wp-content/uploads/2017/01/WEB-CIOMS-EthicalGuidelines.pdf) | 2016 international guideline | Vulnerability, consent, data/sample and ethics considerations |
| [WHO best practices for clinical trials](https://iris.who.int/bitstream/handle/10665/378782/9789240097711-eng.pdf) | 2024 guidance | Quality, participant focus, registration and evidence ecosystem |

### 7.2 Sponsor, investigator, monitoring, deviations, and records

| Source | Current status at cut-off | Blueprint use |
|---|---|---|
| [21 CFR 312.50](https://www.ecfr.gov/current/title-21/chapter-I/subchapter-D/part-312/subpart-D/section-312.50) | Current eCFR | Sponsor duties, monitoring, investigator selection, information |
| [21 CFR 312.60](https://www.ecfr.gov/current/title-21/chapter-I/subchapter-D/part-312/subpart-D/section-312.60) | Current eCFR | Investigator conduct, participant protection, consent, drug control |
| [FDA risk-based monitoring guidance](https://www.fda.gov/media/121479/download) | Final guidance | Study-specific risk-based monitoring plan and adaptation |
| [FDA electronic source data guidance](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/electronic-source-data-clinical-investigations) | Final, 2013 | Source originators, data-element identifiers, audit trail, investigator review |
| [FDA Electronic Systems, Records, and Signatures Q&A](https://www.fda.gov/media/166215/download) | Final, October 2024 | Audit trails, metadata, risk-based review, decommissioning/export |
| [21 CFR Part 11](https://www.ecfr.gov/current/title-21/chapter-I/subchapter-A/part-11) and [FDA scope guidance](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/part-11-electronic-records-electronic-signatures-scope-and-application) | Current regulation/guidance | Electronic-record/signature scope; avoids generic compliance claims |
| [EMA computerized systems and electronic data guideline](https://www.ema.europa.eu/en/documents/regulatory-procedural-guideline/guideline-computerised-systems-and-electronic-data-clinical-trials_en.pdf) | Final, 2023 | Data governance, AI examples, IRT/eConsent/eCOA, validation, access, archiving |
| [EMA TMF guideline](https://www.ema.europa.eu/en/documents/scientific-guideline/guideline-content-management-and-archiving-clinical-trial-master-file-paper-andor-electronic_en.pdf) | Final | Sponsor/investigator TMF, segregation, reconstruction, security/archive |
| [EMA GCP Q&A](https://www.ema.europa.eu/en/human-regulatory-overview/research-development/compliance-research-development/good-clinical-practice/qa-good-clinical-practice-gcp) | Current living guidance | Investigator medical decisions, emergency unblinding, vendor oversight, PI review |
| [EudraLex Volume 10](https://health.ec.europa.eu/medicinal-products/eudralex/eudralex-volume-10_en) | Current, including July 2026 materials | EU clinical-trial guidance baseline, modifications, safety, breaches, TMF, inspections |
| [EMA serious-breach guideline](https://www.ema.europa.eu/en/documents/scientific-guideline/guideline-notification-serious-breaches-regulation-eu-no-5362014-or-clinical-trial-protocol_en.pdf) | Current | Serious-breach workflow and notification evidence |
| [UK MHRA serious-breach guidance](https://www.gov.uk/government/publications/clinical-trials-for-medicines-notification-of-serious-breaches-of-gcp-or-the-trial-protocol/notification-of-serious-breaches-of-gcp-or-the-trial-protocol) | Effective 2026-04-28 | Seven-day sponsor-awareness path, immediate reporting versus later investigation/CAPA |
| [MHRA GxP data integrity guidance](https://www.gov.uk/government/publications/guidance-on-gxp-data-integrity) | Current | ALCOA+, lifecycle, governance, culture |
| [FDA Protocol Deviations guidance](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/protocol-deviations-clinical-investigations-drugs-biological-products-and-devices) | Draft, December 2024 | Refresh input and deviation taxonomy questions; not deployed as final guidance |
| [Applied Therapeutics FDA warning letter](https://www.fda.gov/inspections-compliance-enforcement-and-criminal-investigations/warning-letters/applied-therapeutics-inc-696833-12032024) | Issued 2024-12-03 | Concrete vendor/audit-trail deletion risk; individual case, not prevalence evidence |

### 7.3 Safety and pharmacovigilance

| Source | Current status at cut-off | Blueprint use |
|---|---|---|
| [ICH E2A](https://database.ich.org/sites/default/files/E2A_Guideline.pdf) | Final guideline | Seriousness, expectedness, causality and expedited-reporting concepts |
| [ICH E2B(R3) package](https://admin.ich.org/node/348) | Package 1.11, January 2026; Q&A 2.5, July 2025 | ICSR message exchange, implementation guides, acknowledgements/versioning |
| [MedDRA](https://admin.ich.org/page/meddra) | Current versioned terminology program | Pinned medical terminology and release lifecycle |
| [21 CFR 312.32](https://www.ecfr.gov/current/title-21/chapter-I/subchapter-D/part-312/subpart-B/section-312.32) | Current eCFR | US IND safety responsibilities and 7/15-day reporting paths |
| [EMA clinical-trial safety reporting](https://www.ema.europa.eu/en/human-regulatory-overview/research-development/clinical-trials-human-medicines/reporting-safety-information-clinical-trials) | Updated 2026-02-24 | EU SUSAR, annual, urgent-measure, breach and CTIS routes |
| [EudraVigilance electronic reporting](https://www.ema.europa.eu/en/human-regulatory-overview/research-development/pharmacovigilance-research-development/eudravigilance/eudravigilance-electronic-reporting) | Current; automated compliance monitoring from 2025 | ICSR XML, acknowledgement, gateway/EVWEB roles and monitoring |
| [MHRA clinical-trial safety reporting](https://www.gov.uk/guidance/clinical-trials-for-medicines-collection-verification-and-reporting-of-safety-events) | Current | UK safety collection/verification/reporting and systemic-failure escalation |

### 7.4 Interoperability, registries, and data standards

| Source | Current status at cut-off | Blueprint use |
|---|---|---|
| [CDISC ODM 2.0](https://www.cdisc.org/standards/data-exchange/odm-xml/odm-v2-0) | Released 2023-08-23 | Vendor-neutral study metadata/data/audit exchange with version caveats |
| [CDISC data-exchange standards](https://www.cdisc.org/standards/data-exchange/layout) | Current portfolio | LAB, Define-XML, Dataset-XML, CTR-XML and scope separation |
| [CDISC USDM/DDF](https://www.cdisc.org/ddf) | USDM v4.0, June 2025 | Structured study design, not operational approval/effectivity |
| [CDISC SDTM](https://www.cdisc.org/standards/foundational/sdtm) | Current foundational standard | Submission-oriented tabulation context, not source record authority |
| [CDISC controlled terminology](https://www.cdisc.org/standards/terminology/controlled-terminology) | Current release identified 2026-03-27 | Pinned terminology version and change control |
| [CDISC TMF Reference Model resources](https://tmfrefmodel.com/resources) and [v4 project](https://www.cdisc.org/standards/foundational/trial-master-file-reference-model/tmf-reference-model-v4) | v3.3.1 stable; v4 project at cut-off | Avoid claiming project version as deployed standard |
| [HL7 FHIR R5 ResearchStudy](https://hl7.org/fhir/R5/researchstudy.html) and [ResearchSubject](https://fhir.hl7.org/fhir/researchsubject.html) | Maturity level 0, Trial Use | Requires local profiles; not canonical operations model by default |
| [HL7 FHIR R4 DiagnosticReport](https://hl7.org/fhir/R4/diagnosticreport.html) and [Observation](https://hl7.org/fhir/R4/observation.html) | Normative/current R4 resources | Possible clinical/lab exchange pieces, still profile/partner dependent |
| [ClinicalTrials.gov PRS user guide](https://clinicaltrials.gov/submit-studies/prs-help/user-guide) | Updated 2026-05-01 | Responsible-party approvals/releases, update duties, version history |
| [ClinicalTrials.gov API v2](https://clinicaltrials.gov/data-api/about-api) and [FHIR data](https://clinicaltrials.gov/data-api/fhir) | Current public read services | Read/reconciliation projection, not autonomous filing authority |
| [EU Clinical Trials Regulation overview](https://health.ec.europa.eu/medicinal-products/clinical-trials/clinical-trials-regulation-eu-no-5362014_en) and [CTIS](https://www.ema.europa.eu/en/human-regulatory-overview/research-development/clinical-trials-human-medicines/clinical-trials-information-system) | Current official lifecycle/portal sources | Authorized sponsor workspace, applications, lifecycle notifications, safety/results routes; not a generic browser-automation target |
| [FDA Study Data Technical Conformance Guide](https://www.fda.gov/media/153632/download) | June 2026 | Current submission data/terminology/traceability technical context |

### 7.5 Privacy, identity, security, validation, and AI risk

| Source | Current status at cut-off | Blueprint use |
|---|---|---|
| [European Commission CTR/GDPR Q&A](https://health.ec.europa.eu/document/download/c3042973-b36d-4094-a1fb-a6fc980f065e_en) and [EDPB Opinion 3/2019](https://www.edpb.europa.eu/documents/legislative-opinion/opinion-32019-concerning-the-questions-and-answers-on-the-interplay_en) | Current interpretive sources | Separates trial consent from GDPR legal basis and secondary uses |
| [HHS HIPAA Privacy Rule summary](https://www.hhs.gov/hipaa/for-professionals/privacy/laws-regulations/index.html) | Current overview | US protected-health-information baseline; local analysis still required |
| [NIST SP 800-63-4](https://www.nist.gov/publications/nist-sp-800-63-4-digital-identity-guidelines) | Final, August 2025 | Identity assurance and authenticator design |
| [RFC 9700 OAuth 2.0 Security Best Current Practice](https://www.rfc-editor.org/info/rfc9700/) | January 2025 | Modern authorization-protocol security baseline |
| [NIST SP 800-53 Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final) | Current update | Security/privacy control catalog and assessment baseline |
| [NIST Cybersecurity Framework 2.0](https://www.nist.gov/cyberframework) | Current | Govern/identify/protect/detect/respond/recover program framing |
| [NIST AI RMF Generative AI Profile](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf) | NIST AI 600-1, 2024 | GenAI risk, content, privacy, security, evaluation and governance |
| [FDA/EMA Good AI Practice principles](https://www.fda.gov/about-fda/artificial-intelligence-drug-development/guiding-principles-good-ai-practice-drug-development) | Current 2026 joint principles | Human-centric, risk-based, context-of-use and lifecycle assurance |
| [EMA AI reflection paper](https://www.ema.europa.eu/system/files/documents/scientific-guideline/reflection-paper-use-artificial-intelligence-ai-medicinal-product-lifecycle-en.pdf) | Final, September 2024 | AI lifecycle risks across medicinal-product lifecycle |
| [FDA AI regulatory-decision-support guidance](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/considerations-use-artificial-intelligence-support-regulatory-decision-making-drug-and-biological) | Draft, January 2025 | Context-of-use/credibility questions; refresh input only |

### 7.6 Representative adapter and platform semantics

These current primary sources were accessed by 2026-08-31. They support concrete examples, not product endorsement,
tenant qualification, supplier approval, validation, or a claim that the feature is available in every plan/region.

| Source | Current status at cut-off | Blueprint use and limit |
|---|---|---|
| [OpenClinica 4 participant read API](https://docs.openclinica.com/oc4/how-and-when-to-use-apis/oc4-openclinica-4-technical-documentation-participants-get-participants-study-level-or-site-level/) | Official page approved 2026-02-24 | Study/site role and scope distinction; installed configuration still controls access |
| [OpenClinica 4 clinical-data API](https://docs.openclinica.com/oc4/how-and-when-to-use-apis/oc4-clinicaldata-import-crf-data/) | Current official 2026 documentation | JSON and ODM 1.3.2 plus extensions, optional metadata/audit/query/archive fields and wildcard scope; repeated import logging is not business idempotency |
| [Veeva Vault endpoint structure and versioning](https://general.veevavault.dev/qualityone/vault-api/getting-started/endpoint-structure/) | Current official developer documentation; newest API is Beta, older versions stable | Pin Vault/app/API/configuration; no inference of lifecycle approval or signature meaning |
| [Camunda 8.8 Orchestration Cluster REST API](https://docs.camunda.io/docs/8.8/apis-tools/orchestration-cluster-api-rest/orchestration-cluster-api-rest-overview/) | Official versioned 8.8 `/v2/` API documentation | Workflow persistence and version compatibility; engine state is not automatically a GCP/regulated record |
| [Amazon S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html) | Current official documentation | Version-level retention/legal hold and governance bypass semantics; not complete validation or record governance |
| [DocuSign OpenAPI specifications](https://github.com/docusign/OpenAPI-Specifications) | Official eSignature REST v2.1 and Connect schemas | Pin account/region/envelope/callback semantics; completion is not informed-consent verification |
| [Oracle Argus Safety E2B(R3) best practices 8.4.3](https://docs.oracle.com/en/industries/life-sciences/argus-safety/8.4.3/oasbp/oracle-argus-safety-e2b-r3-best-practices.pdf) | Official release 8.4.3 documentation | Product/profile/E2B compatibility example; not sponsor medical judgment or instance validation |
| [Twilio Message resource](https://www.twilio.com/docs/messaging/api/message-resource) | Current official documentation; callback parameter set may evolve | Provider delivery states and callback hardening; delivery is not proof of correct human receipt/understanding |
| [ClinicalTrials.gov PRS user guide](https://clinicaltrials.gov/submit-studies/prs-help/user-guide) | Updated 2026-05-01 | Responsible-party entry/approval/release and `autoRelease`; not autonomous filing authority |

## 8. Evidence-to-guide coverage

| Guide | Primary evidence clusters |
|---|---|
| [Mission and workload fit](../../agents/clinical-trial-operations-agent/01-mission-boundaries-authority-and-workload-fit.md) | ICH E6/E8, FDA sponsor/investigator regulation, EMA investigator authority, repository authority controls |
| [Architecture and integrations](../../agents/clinical-trial-operations-agent/02-reference-architecture-runtime-tools-and-integrations.md) | ICH/EMA/FDA computerized systems, EDC/source/TMF guidance, E2B, CTIS/registry, CDISC/FHIR scopes |
| [Identity and amendments](../../agents/clinical-trial-operations-agent/03-study-protocol-site-participant-identity-and-amendments.md) | E6 records/versions, M11/USDM, amendment and privacy sources |
| [Consent, eligibility, and visits](../../agents/clinical-trial-operations-agent/04-consent-eligibility-visits-and-role-governance.md) | FDA/OHRP/WMA/EDPB consent, investigator responsibility, DHT/decentralized trial guidance |
| [Records and monitoring](../../agents/clinical-trial-operations-agent/05-records-queries-monitoring-deviations-and-capa.md) | E6, FDA/EMA electronic records, TMF, monitoring, breach, MHRA data integrity |
| [Safety and blinding](../../agents/clinical-trial-operations-agent/06-safety-irt-laboratories-blinding-and-reconciliation.md) | E2A/E2B/MedDRA, 21 CFR 312.32, EMA/UK safety, E9, EMA IRT guidance |
| [State, context, and recovery](../../agents/clinical-trial-operations-agent/07-state-events-context-memory-planning-and-recovery.md) | Repository runtime/effect/memory controls applied to GCP record and deadline constraints |
| [Security and validation](../../agents/clinical-trial-operations-agent/08-security-privacy-validation-and-inspection-readiness.md) | E6 validation/security, FDA/EMA systems, privacy authorities, NIST identity/security/AI sources |
| [Evaluation and operations](../../agents/clinical-trial-operations-agent/09-evaluation-observability-deployment-scale-and-incidents.md) | FDA/EMA AI principles, GCP risk/records, repository evaluation/operations controls |
| [Stages and contracts](../../agents/clinical-trial-operations-agent/10-zero-to-production-stages-schemas-and-checklists.md) | Category expansion program plus all source clusters above |
| [Qualified adapters and worked flows](../../agents/clinical-trial-operations-agent/11-qualified-adapters-and-worked-clinical-operations-flows.md) | Provider primary documentation, GCP/system-validation controls, exact identities, operation capabilities, receipts, reconciliation and recovery |

## 9. What the evidence does not establish

The research does **not** establish that:

- an AI agent is required, superior to a deterministic workflow, or cost-effective for a particular sponsor;
- the described design satisfies every country, product, device, biologic, pediatric, gene-therapy, controlled-substance, radiation, biospecimen, genomic, or emergency-research requirement;
- a named system or standard is supported correctly by a specific vendor deployment;
- generated eligibility mappings, safety narratives, monitoring signals, or protocol impact assessments are clinically correct without local evaluation and qualified review;
- a risk-based validation plan has been accepted by a regulator or ethics body;
- pseudonymization makes participant data anonymous;
- an audit trail, checksum, electronic signature, or reconciliation result alone proves data truth or compliance;
- a workflow-complete, message-delivered, envelope-completed, object-locked, upload-successful, or gateway-acknowledged
  status proves the corresponding clinical, consent, regulated-record, filing, or regulator outcome;
- the example schemas are submission formats, CRFs, ICSR specifications, validation scripts, or legal retention schedules; or
- any model/provider contract, residency mode, non-training claim, or security control remains current without procurement and technical verification.

## 10. Known limitations

1. The blueprint is medication-trial oriented and does not comprehensively profile medical-device, combination-product, vaccine, cell/gene-therapy, or observational-study obligations.
2. It does not contain a country-by-country clock, ethics, privacy, signature, retention, registry, or submission matrix.
3. It did not test or qualify a live EDC, eTMF, CTMS, eConsent/e-signature, IRT/RTSM, lab, eCOA/DHT, identity,
   workflow, object-store, notification, EudraVigilance, CTIS, ClinicalTrials.gov PRS, or safety-database tenant.
4. It does not define sponsor-specific SOPs, delegation logs, monitoring plans, quality-tolerance limits, validation deliverables, or CAPA classifications.
5. It does not validate any protocol compiler, eligibility ruleset, safety rule engine, translation, terminology map, or model.
6. Human-factors performance at sites, including language, accessibility, workload, alert fatigue, and automation bias, remains empirical.
7. Vendor contracts, subprocessors, residency, model retention/training behavior, and service-level guarantees require separate current verification.
8. Public regulator pages and living standards can change after the cut-off; links and implementation dates need refresh.
9. The named provider examples do not establish availability, configured semantics, supplier suitability, GxP
   validation, support, residency, performance, or compliance for a sponsor deployment.
10. Medical review, investigator judgment, sponsor responsibility, ethics decisions, pharmacovigilance determinations,
    local regulatory/legal/privacy interpretation and validation approval remain with qualified local owners.
11. Example SLO, RTO/RPO, recovery order, retention, clock and capability-expiry shapes are not universal thresholds.

## 11. Deployment research still required

Before a real implementation, research and document:

- exact study types, phases, products, countries, sites, institutions, ethics bodies, sponsor/CRO contracts, and responsible parties;
- approved protocol/amendment/consent/eligibility/safety/monitoring/deviation/CAPA SOPs and role matrices;
- every relevant reporting trigger, calendar, channel, acknowledgement, correction, and follow-up rule;
- each source-system operation's intended use, tenant/application/configuration, role, region, object/field scope,
  blinding projection, provider API/schema, validation state, audit/export behavior, receipt meaning, rate/batch limits,
  idempotency, reconciliation, feature/plan dependency, capability expiry, support and decommissioning;
- participant identity and privacy flows, lawful bases/authorizations, transfers, optional permissions, retention, withdrawal, subject-rights handling, and breach process;
- blinding design, IRT roles, pharmacy workflows, emergency-unblinding backup, and inferential disclosure risks;
- actual data volumes, burst patterns, safety and human-review staffing, provider recovery behavior, maintenance windows,
  offline-site constraints, normal versus recovery-load capacity, regional/cell constraints and disaster-recovery objectives;
- model/provider data handling, snapshot availability, change notification, regional routing, abuse monitoring, and incident evidence;
- local human-factors studies, validation protocol, severe-failure thresholds, release approvers, and periodic review; and
- closeout/archive/vendor-exit requirements and retrieval exercises.

## 12. Refresh triggers and ownership

| Trigger | Required reassessment | Suggested owner |
|---|---|---|
| ICH E6/E8/E9/E2/M11 revision or regional adoption change | Roles, quality, protocol, safety, records, validation, policy release | Clinical quality/regulatory |
| EU Annex 2 becomes regionally effective or Volume 10/CTIS changes | Decentralized/RWD operations, systems, submissions, training | EU regulatory operations |
| FDA finalizes protocol-deviation or AI guidance | Deviation/CAPA and AI credibility/validation sections | US regulatory/quality |
| Safety law, gateway, E2B, MedDRA, or acknowledgement changes | Clock/routing/message/terminology adapter and validation | Pharmacovigilance |
| CDISC/FHIR/TMF/registry standard or API changes | Mapping, compatibility, round-trip, export, source ownership | Data standards/integration |
| EDC/eTMF/CTMS/IRT/lab/eCOA/DHT/safety vendor release | Interface, configuration, audit, migration, security, reconciliation | System owner/validation |
| Provider role/field configuration, plan, region, Beta/GA status, rate limit, callback, receipt or deprecation change | Operation capability, negative tests, expiry, reconciliation, rollout/disable | System owner/integration validation |
| PRS/CTIS/gateway/workflow/e-signature/messaging/object-store behavior changes | Draft/release authority, receipt meaning, idempotency, records, recovery and submission validation | Regulatory operations/system owner |
| Privacy/identity/security guidance or legal interpretation changes | Data flow, authorization, authentication, retention, transfer | Privacy/security/legal |
| Model/provider/tool/prompt/projection/policy change | Behavior manifest, risk assessment, offline/online eval, rollout | AI product/validation |
| Protocol amendment or new study population | Release manifest, eligibility, consent, visits, safety, data model | Sponsor study team |
| Incident, inspection finding, near miss, drift, or alert burden | Containment, impact, CAPA, held-out eval, stage re-entry | Quality/operations |

## 13. Completion and saturation assessment

Research was considered sufficiently saturated for a production-oriented architecture blueprint when further primary-source searches repeated the same core constraints:

- qualified humans retain participant-protection and clinical authority;
- sponsor/investigator responsibility persists through delegation and vendors;
- protocol versions, computerized systems, audit trails, metadata, and transfers need governed lifecycle controls;
- safety is a structured, time-bound, acknowledged, reconciled workflow;
- blinding and privacy require access/data architecture, not prose safeguards;
- exactly seven memory lifetimes and a digest-verified continuity receipt preserve restart-safe authority, clocks,
  effects, blinding and source/version state without treating chat as memory;
- standards aid exchange but do not make semantics, approval, validation, or compliance automatic; and
- operation-level provider qualification is narrower than product selection and expires as configuration/version changes;
- recovery requires bounded convergence of clocks, effects, sources, approvals and records—not only restored compute; and
- AI use requires context-of-use, lifecycle risk, human factors, monitoring, controlled failure mining and whole-bundle change control.

The remaining unknowns are deployment-specific rather than gaps a generic internet survey can resolve: jurisdiction/product applicability, protocol and SOP interpretation, vendor behavior, local data mappings, human factors, capacity, residual risk, and validation evidence. Those uncertainties are deliberately exposed as prerequisites and refresh triggers rather than filled with assumptions.

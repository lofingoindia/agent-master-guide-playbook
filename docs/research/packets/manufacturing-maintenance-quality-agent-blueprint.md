# Manufacturing Maintenance and Quality Agent Blueprint — Research Packet

- **Research completed:** 2026-08-31
- **Scope:** category 46, manufacturing maintenance and quality agent engineering
- **Companion playbook:** [Manufacturing Maintenance and Quality Agent](../../agents/manufacturing-maintenance-quality-agent/README.md)
**Evidence policy:** primary/official sources were used as the foundation; vendor documentation is treated as product- and installation-specific; standards summaries do not replace licensed normative text, local law, a machine risk assessment, or a validated quality system.

## Research question and decision frame

The research asked how to build a useful agent across MES/MOM, ERP, CMMS/EAM, QMS/LIMS, industrial data, edge, maintenance, inspection, nonconformance, CAPA, genealogy, and recall support without allowing model reasoning to control hazardous equipment or usurp accountable quality/safety decisions.

The answer is a bounded coordination system:

- one durable coordinator is the default, not an agent swarm;
- PLC/SIS/interlock/control and safety paths remain architecturally absent;
- operational evidence flows northbound through qualified, usually read-only site adapters;
- identity, unit/time/status/calibration eligibility, sampling, constraints, policy, approval, and effects are deterministic services;
- model output is untrusted proposed interpretation or plan;
- low-risk business writes use sealed intents, semantic operation IDs, concurrency checks, `UNKNOWN` outcomes, reconciliation, and read-back;
- LOTO/permits, equipment safety, final quality disposition/release, regulatory decisions, and unsafe physical effects stay with accountable humans and validated systems;
- model memory never becomes operational truth, and production learning happens only through reviewed behavior releases.

## Method and selection criteria

Research used multiple source classes and cross-checks:

1. standards-body pages and current reference specifications for ISA-95/88, OPC UA, MQTT, Sparkplug, MTConnect, asset management, condition monitoring, digital twins, quality, metrology, sampling, product recall, reference designation, and cybersecurity;
2. current official IBM, SAP, and Microsoft documentation for real EAM/ERP/QMS API and state semantics;
3. current US government sources for LOTO, machine guarding, process safety, quality regulation, electronic records, process validation, data integrity, recalls, OT cybersecurity, and metrology;
4. official model/runtime and observability guidance for typed tools, approval boundaries, evaluation, context, and telemetry;
5. draft, preview, deprecated, and release-candidate material only when its nonfinal status materially affects implementation choices.

Important claims were cross-checked across domain standards, vendor behavior, and production failure semantics. Marketing claims, generic listicles, unversioned architecture diagrams, SEO summaries, and advice that treated agent autonomy as the goal were discarded.

Access dates are 2026-08-31 unless a source entry states otherwise. Publication dates below describe the source or edition, not the access date.

## Findings that shaped the architecture

### Industrial hierarchy is useful, but not a universal topology

[ISA-95](https://www.isa.org/standards-and-publications/isa-standards/isa-95-standard) distinguishes physical process, sensing/manipulation, monitoring/supervision, manufacturing operations, and enterprise business functions. The 2025 update to Part 1 modernizes the model and replaces the 2010 edition ([ISA announcement](https://www.isa.org/news-press-releases/2025/april/update-to-isa-95-standard-addresses-integration-of)). This supports separating model reasoning from levels 0–2 and keeping business coordination near level 3/4 interfaces.

It does not prove that every plant has the same level boundaries or network zones. The playbook therefore uses “planes” and site risk assessment rather than asserting that ISA-95 itself mandates a specific security topology.

[ISA-88](https://www.isa.org/standards-and-publications/isa-standards/isa-88-standards) supplies batch-control terminology, models, recipes, and batch records, and its treatment of safety interlocks as separate from recipe/sequencing reinforces the hard boundary. Its machine/unit state guidance is useful for normalization, but source controller/MES state still remains authoritative.

ISA's official [ISA-18.2 update overview](https://www.isa.org/intech-home/2016/may-june/departments/isa18-alarm-management-standard-updated) describes an alarm-management lifecycle spanning philosophy, identification, rationalization, detailed design, implementation, operation, maintenance, monitoring/assessment, change management, and audit. This supports treating acknowledgement, shelving, suppression, priority, and configuration as protected operator/engineering workflows. Because the overview describes the 2016 edition, an implementing site must verify the current licensed edition and its own alarm philosophy.

[MESA B2MML](https://mesa.org/topics-resources/b2mml/) is an XML implementation of ISA-95, currently identified by MESA as version 0700. It can reduce custom payload design where already adopted; introducing it solely for an agent would be unnecessary.

### Protocol delivery is not application truth or exactly-once effect

The current [OPC UA Part 14 PubSub](https://reference.opcfoundation.org/Core/Part14/v105/docs/6.2.4.2) specification includes status and timestamp semantics. [OPC UA Part 4 Publish](https://reference.opcfoundation.org/Core/Part4/v105/docs/5.14.5) exposes available sequence numbers and retransmission behavior, but retransmission queues are not universally supported. Sequence gaps must be detected and marked; the agent cannot infer continuous history.

[MQTT 5.0](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html) defines at-most-once, at-least-once, and exactly-once delivery at the protocol interaction level. These do not guarantee exactly-once creation of a work order, quality hold, or vendor case. The playbook adds semantic operation identity, local uniqueness, vendor correlation, authoritative reconciliation, and read-back.

[Sparkplug 3.0](https://sparkplug.eclipse.org/specification/version/3.0/) adds an MQTT topic namespace, payload definition, and session-state management for industrial use. It does not replace site authorization, object identity, evidence eligibility, or business idempotency.

### Machinery information models help, but identifier scope must be qualified

[OPC UA Machinery Building Blocks 1.04](https://reference.opcfoundation.org/Machinery/v104/docs/1) includes machine/component identification, state, operation mode, counters, monitoring, and notification concepts. The [Component Identification](https://reference.opcfoundation.org/Machinery/v104/docs/10) model is useful for machine/component context.

[OPC UA Machinery Result Transfer 1.01](https://reference.opcfoundation.org/specs/OPC-40001-101/full) covers job, product, part, recipe, step, partial/final results, metadata, files/content, evaluation, transfer, and acknowledgement. Its identifier guarantees are not automatically global and historical; qualify identity by server/endpoint and effective time. The [result transfer mechanisms](https://reference.opcfoundation.org/Machinery/Result/v101/docs/6.3) also show why “available,” “transferred,” “acknowledged,” “ingested,” and “accepted in a quality decision” are distinct events.

[OPC UA Machinery Job Management](https://reference.opcfoundation.org/specs/OPC-40001-3/full) can improve job semantics where supported. Companion specifications must be qualified against the actual endpoint and product implementation.

[MTConnect 2.5](https://docs.mtconnect.org/) documentation was current on the research date, with its normative model published at the [MTConnect 2.5 model site](https://model.mtconnect.org/Version2.5/). It is a useful read-oriented manufacturing data source, not a source of quality release or safety authority.

### Asset and maintenance standards support a governed program, not autonomous decisions

[ISO 55001:2024](https://www.iso.org/standard/83054.html) frames asset-management decisions across lifecycle value, performance, risk, expenditure, data/information, and predictive action. It supports treating maintenance recommendations as part of an owned asset-management system.

[ISO 14224:2016](https://www.iso.org/standard/64076.html) provides equipment taxonomy plus equipment, failure, and maintenance data guidance for petroleum, petrochemical, and natural-gas industries. Its taxonomy ideas are useful, but the sector scope and exclusions mean it must not be presented as a universal manufacturing data requirement.

[ISO 17359:2018](https://www.iso.org/standard/71194.html) gives general condition-monitoring program guidance. [ISO 13374-1:2003](https://www.iso.org/standard/21832.html), confirmed current in 2025, addresses condition data processing, communication, and presentation. [ISO 13379-1:2025](https://www.iso.org/standard/88027.html) and [ISO 13381-1:2025](https://www.iso.org/standard/88029.html) are the current diagnostic and prognostic guidance editions. Together they support the playbook’s observation → context → validated feature/detection → hypothesis → governed response → effectiveness loop.

[ISO 22400-1:2014](https://www.iso.org/standard/56847.html), confirmed in 2025, supplies a manufacturing-operations KPI framework. It does not make one OEE formula or target universally correct; KPI definitions and source quality must remain explicit.

### CMMS/EAM states and APIs have installation-specific side effects

IBM Maximo documents [continuous, gauge, and characteristic meters](https://www.ibm.com/docs/en/masv-and-l/maximo-manage/cd?topic=module-meters) and [condition monitoring](https://www.ibm.com/docs/en/masv-and-l/maximo-manage/cd?topic=monitoring-working-condition). Its [work-order status documentation](https://www.ibm.com/docs/en/masv-and-l/maximo-manage/cd?topic=overview-work-order-statuses) distinguishes waiting, approved, scheduled, material/condition waits, in progress, completed, closed, and cancelled states; transitions can reserve/unreserve inventory and change asset-related state. A completed order is not the same as a closed order or physical return to service.

Maximo [work-order documentation](https://www.ibm.com/docs/en/masv-and-l/maximo-manage/cd?topic=overview-work-orders) shows that generated records may copy safety plans and meter information under specific paths. Do not assume a generic create call preserves required planning artifacts.

The [Maximo Manage REST API](https://developer.ibm.com/apis/catalog/maximo--maximo-manage-rest-api/) can expose an installation-specific OpenAPI description because object structures and configuration are customizable. The detailed [REST API behavior](https://developer.ibm.com/apis/catalog/maximo--maximo-manage-rest-api/api/API--maximo--maximo-manage-rest-api) includes transaction identifiers for duplicate detection and concurrency responses such as HTTP 412; [OSLC creation guidance](https://www.ibm.com/docs/en/masv-and-l/maximo-ref/cd?topic=resources-creating-oslc) describes transaction identity and returned `Location`/`ETag`. These facts motivate operation IDs and ETags, but exact behavior still needs real installation tests. [Maximo API keys](https://www.ibm.com/docs/en/masv-and-l/maximo-manage/cd?topic=apis-api-keys) inherit a user’s privileges, which makes effective-role qualification essential.

SAP’s current documentation describes [maintenance order types](https://help.sap.com/docs/SAP_S4HANA_CLOUD/2dfa044a255f49e89a3050daf3c61c11/5fbd47786341411992fc9915284da2b2.html), [status operations](https://help.sap.com/docs/SAP_S4HANA_CLOUD/f9ba73d1b8c543c2bd2ba1666271af86/1074e4410eee4ba7b462a3781332f305.html), and [release of a maintenance order](https://help.sap.com/docs/SAP_S4HANA_CLOUD/f9ba73d1b8c543c2bd2ba1666271af86/fddadc8a433d49a78de9793e22f987a9.html), including approval configuration and `If-Match` concurrency. SAP also documents that an older [MaintenanceOrder OData API is deprecated](https://help.sap.com/doc/b870b6ebcd2e4b5890f16f4b06827064/2023.000/en-US). Therefore, the installed edition and successor endpoint must be pinned.

Microsoft Dynamics documents how [maintenance requests become work orders](https://learn.microsoft.com/en-us/dynamics365/supply-chain/asset-management/manage-maintenance-requests/create-work-order-from-a-maintenance-request) and how [work-order scheduling](https://learn.microsoft.com/en-us/dynamics365/supply-chain/asset-management/work-order-scheduling/schedule-work-orders) can use worker, tool, asset, capacity, and competency constraints. Configuration can disable or relax constraints; the product’s presence does not prove a feasible or safe schedule.

### Quality-system currency changed materially in 2026

[ISO 9001:2015](https://www.iso.org/standard/62085.html), with [Amendment 1:2024](https://www.iso.org/standard/88431.html), remained the current published edition on 2026-08-31. The replacement [ISO 9001:2026](https://www.iso.org/standard/88464.html) was under publication and expected in September 2026. This playbook therefore does not claim the unpublished edition is current; it creates a refresh trigger after publication.

[ISO 13485:2016](https://www.iso.org/standard/59752.html) remains the medical-device QMS edition referenced by the US FDA’s Quality Management System Regulation. FDA states that [QMSR became effective on 2 February 2026](https://www.fda.gov/medical-devices/postmarket-requirements-devices/quality-management-system-regulation-qmsr), incorporates ISO 13485:2016 and Clause 3 of ISO 9000:2015 by reference with specified provisions, retired QSIT, and uses a new compliance program. FDA’s [QMSR FAQ](https://www.fda.gov/medical-devices/quality-management-system-regulation-qmsr/quality-management-system-regulation-frequently-asked-questions) says future ISO revisions do not automatically change the regulation and explains inspection-access changes. The [current 21 CFR Part 820](https://www.ecfr.gov/current/title-21/chapter-I/subchapter-H/part-820) remains the legal source; quality/legal owners must determine applicability.

For electronic records, [21 CFR 11.10](https://www.ecfr.gov/current/title-21/chapter-I/subchapter-A/part-11/subpart-B/section-11.10) requires controls including system validation, accurate/complete copies, record protection and retention, authorized access, time-stamped audit trails, operational/authority/device checks where appropriate, training, accountability, and document controls. A model log is not automatically a compliant electronic record.

For drug CGMP, [21 CFR 211.100](https://www.ecfr.gov/current/title-21/chapter-I/subchapter-C/part-211/subpart-F/section-211.100) requires written procedures and changes to be drafted, reviewed, approved, followed, and documented, including deviations. [21 CFR 211.192](https://www.ecfr.gov/current/title-21/chapter-I/subchapter-C/part-211/subpart-J/section-211.192) requires review of production/control records and investigation of unexplained discrepancies or failures. FDA’s [Data Integrity and Compliance With Drug CGMP guidance](https://www.fda.gov/media/119267/download) and [Process Validation guidance](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/process-validation-general-principles-practices) support risk-based data controls and lifecycle process validation. These are US/product-specific examples, not universal requirements.

### Measurement, laboratory, and sampling decisions must be deterministic

[ISO 10012:2026](https://www.iso.org/standard/10012) is the current measurement-management edition, published in February 2026 and replacing the 2003 edition. [ISO/IEC 17025:2017](https://www.iso.org/standard/66912.html) remains the competence, impartiality, and consistent-operation standard for testing/calibration laboratories.

[NIST metrological traceability guidance](https://www.nist.gov/calibrations/traceability) states that traceability is a property of a measurement result supported by a documented unbroken calibration chain where each calibration contributes uncertainty. It also cautions that traceability alone does not assure fitness for purpose and that the result provider owns the claim. The playbook consequently records both metrological traceability and data lineage, plus decision-specific fitness.

[ISO 2859-1:2026](https://www.iso.org/standard/85464.html) is the current attribute-sampling edition for AQL-indexed plans. [ISO 28590:2017](https://www.iso.org/standard/64622.html) gives an introduction to the ISO 2859 attribute-sampling series. A model must not invent AQLs, code letters, sample sizes, or acceptance/rejection numbers; a validated service executes the site-approved plan.

### Inspection completion can have material side effects

SAP’s [inspection completion guidance](https://help.sap.com/docs/SAP_S4HANA_CLOUD/d1e58be39d884a0dbf75a7526a9acbf4/6f79b6535fe6b74ce10000000a174cb4.html) requires characteristic/sampling work before the usage decision and notes that the usage decision can trigger stock postings. SAP also exposes an on-premise [usage-decision write API](https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/a08e12a754cf4891b41a01a285d065bb/83e6fad0ff124c479b5ba4f16be0f540.html) and [change tracking for inspection lots](https://help.sap.com/docs/SAP_S4HANA_CLOUD/ee9ee0ca4c3942068ea584d2f929b5b1/2224caa27c1a4fbdbf2ac8b6869a6318.html). This supports the hard boundary: the agent may assemble a decision bundle but cannot execute final usage/release.

Microsoft Dynamics [quality associations](https://learn.microsoft.com/en-us/dynamics365/supply-chain/inventory/quality-associations) and [quality orders](https://learn.microsoft.com/en-us/dynamics365/supply-chain/inventory/quality-orders) illustrate test groups, sampling, AQL, quality orders, and quantity blocking. The documentation also marks some advanced returns/transfer behavior as preview. Preview features must not be treated as qualified production dependencies.

### Product and equipment identity require different standards

[IEC 81346-1:2022](https://www.iso.org/standard/82229.html) provides principles for unambiguous reference designation of system objects. [IEC 81346-14:2026](https://committee.iso.org/standard/86477.html) applies those concepts to manufacturing and processing systems but explicitly does not identify individual manufactured products. Asset/component identity and product lot/serial genealogy therefore require separate source systems and mappings.

[GS1 EPCIS 2.0.1](https://ref.gs1.org/standards/epcis/2.0.1/) supports event-based product visibility, including TransformationEvents linking inputs and outputs. Its correction approach is additive rather than destructive. Use it where the enterprise has adopted EPCIS; do not introduce it only to give the agent a fashionable graph.

[MIMOSA CCOM](https://www.mimosa.org/mimosa-ccom/) is an asset-lifecycle exchange model. The MIMOSA site identifies 4.0.0 as a stable release and 4.1.0-RC1 as a release candidate. The playbook avoids describing the RC as the stable current standard.

### Safety is a human and deterministic-system boundary

OSHA’s [Control of Hazardous Energy standard, 29 CFR 1910.147](https://www.osha.gov/laws-regs/regulations/standardnumber/1910/1910.147), requires an energy-control program with procedures, training, periodic inspection, authorized-employee responsibilities, verification, group LOTO, and shift/personnel transfer controls. The agent may route an approved procedure or handoff; it cannot apply/remove locks, verify isolation, or attest compliance.

OSHA’s [machine-guarding standards page](https://www.osha.gov/machine-guarding/standards) and [Process Safety Management mechanical-integrity guidance](https://www.osha.gov/laws-regs/regulations/standardnumber/1910/1910.119AppC) reinforce equipment categorization, trained personnel, procedures, acceptance criteria, inspection/test frequency, and documented results.

[ISO 12100:2010](https://www.iso.org/standard/51528.html) remained the published machinery risk-assessment/risk-reduction standard on the research date, though a replacement project was progressing. [ISO 10218-1 and ISO 10218-2:2025](https://www.iso.org/committee/5915511/x/catalogue/) are the current industrial robot and robot-application safety editions.

The current IEC functional-safety store package for the [IEC 61511 series](https://webstore.iec.ch/en/publication/5527) is a 2026 series package, while the core Part 1 technical edition remains 2016 with Amendment 1:2017. The playbook avoids calling it a new 2026 Part 1 edition. [IEC TR 61511-0:2018](https://webstore.iec.ch/en/publication/60766) is an introductory technical report. Functional-safety engineering and SIS lifecycle decisions stay outside agent authority.

### OT cybersecurity requires lifecycle and shared responsibility

[NIST SP 800-82 Rev. 3](https://csrc.nist.gov/pubs/sp/800/82/r3/final), published September 2023, addresses OT cybersecurity while recognizing performance, reliability, and safety requirements. The [ISA/IEC 62443 series](https://www.isa.org/standards-and-publications/isa-standards/isa-iec-62443-series-of-standards) distributes responsibilities across asset owners, product suppliers, integrators, and service providers over the lifecycle. ISA’s [62443-2-1 update announcement](https://www.isa.org/news-press-releases/2025/january/update-to-isa-iec-62443-standards-addresses-organi) describes the 2024 asset-owner security-program revision replacing the 2009 edition. [IEC 62443-3-3:2013](https://webstore.iec.ch/en/publication/7033) remains a system-security-requirements/security-level reference with a stated stability date through 2027.

[CISA Cross-Sector Cybersecurity Performance Goals](https://www.cisa.gov/cybersecurity-performance-goals) provide prioritized baseline practices. CISA’s [Secure by Demand OT procurement guide](https://www.cisa.gov/sites/default/files/2025-01/joint-guide-secure-by-demand-priority-considerations-for-ot-owners-and-operators-508c.pdf), published January 2025, highlights authentication, vulnerability management, logging, secure configuration/update, and lifecycle considerations for buyers.

[NIST IR 8183 Rev. 2](https://csrc.nist.gov/pubs/ir/8183/r2/ipd), the Cybersecurity Framework 2.0 Manufacturing Profile, was an Initial Public Draft published in September 2025. It is useful emerging work, not a final normative baseline.

### Digital twins help test, but do not prove real-plant safety

[ISO 23247-1:2021](https://www.iso.org/standard/75066.html) provides the overview and general principles for manufacturing digital twins. NIST’s [Digital Twins for Advanced Manufacturing](https://www.nist.gov/programs-projects/digital-twins-advanced-manufacturing) program emphasizes trustworthy digital twins, verification, validation, and uncertainty quantification. The [NIST Digital Twin Lab publication](https://www.nist.gov/publications/digital-twin-lab-supporting-research-and-development-and-standards-and-implementations), published August 2025, and [NIST Smart Manufacturing Systems Test Bed](https://www.nist.gov/laboratories/tools-instruments/smart-manufacturing-systems-sms-test-bed) illustrate lab/testbed approaches spanning machine tools, data, and metrology.

The playbook uses replay, contract sandbox, workflow simulation, qualified twins/testbeds, live shadow, and bounded canary in increasing fidelity. A digital twin’s unmodeled physics, controller behavior, human work, and rare faults remain limitations.

[FMI 3.0.2](https://fmi-standard.org/docs/3.0.2/), dated November 2024, supports model exchange/co-simulation/scheduled execution patterns where adopted. It does not validate the models carried through it.

### Recall support requires conservative traceability and accountable decisions

[ISO 10393:2013](https://www.iso.org/standard/45968.html), confirmed in 2025, provides general consumer-product recall guidance. FDA’s [Product Recalls guidance](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/product-recalls-including-removals-and-corrections) and [medical-device corrections and removals page](https://www.fda.gov/medical-devices/postmarket-requirements-devices/recalls-corrections-and-removals-devices) cover US-specific strategy, records, communications, and reporting concepts.

The agent may assemble the possible impact population and missing genealogy, but accountable quality, regulatory, legal, and business roles decide classification, reporting, notification, correction/removal, and closure. Unknown trace edges must widen or visibly qualify the candidate population, not be dropped by confidence ranking.

### Agent runtimes need explicit boundaries, typed tools, and representative evals

OpenAI’s current [model guidance](https://developers.openai.com/api/docs/guides/latest-model) recommends defining autonomy and approval boundaries, exposing relevant typed tools, documenting tool return/error shapes, and using representative evaluations while tracking context and token/cost behavior. These are vendor-specific implementation inputs, not manufacturing assurance by themselves.

Anthropic’s [Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) distinguishes predefined workflows from agents and recommends the simplest system that meets the need, noting that agents add latency and cost. This supports deterministic workflows and a bounded coordinator rather than autonomous multi-agent architecture.

The [OWASP Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) identifies goal hijacking, tool misuse, identity/privilege abuse, supply-chain issues, unexpected code execution, memory/context poisoning, insecure agent communication, cascading failures, human-agent trust, and rogue behavior. The [OWASP GenAI Security Project’s current LLM risks](https://owasp.org/www-project-top-10-for-large-language-model-applications/) supplement that checklist. The playbook maps these to architectural controls rather than relying on prompts.

[OpenTelemetry specifications](https://opentelemetry.io/docs/specs/) provide portable telemetry concepts. GenAI [semantic convention attributes](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/) continue to evolve and can contain sensitive content, so the playbook pins convention versions and maintains stable manufacturing domain events without default prompt/content capture.

## Source ledger and implementation use

| Source family | Edition/status used | Playbook decision | Refresh trigger |
|---|---|---|---|
| ISA-95 | Part 1 updated 2025; series overview current | separate control, manufacturing operations, and enterprise concerns | any revised part used by site integration |
| ISA-88 | current ISA series page; TR machine/unit states noted | keep recipe/sequence semantics apart from safety interlocks | site batch/state standard revision |
| ISA-18.2 | official 2016 lifecycle update overview | alarms are evidence; no agent acknowledgement, shelving, suppression, or configuration | current edition and site alarm-philosophy change |
| OPC UA Core/Machinery | Part 4/14 1.05; Machinery 1.04; Result 1.01 | preserve time/status/sequence and qualify endpoint identity/capabilities | endpoint profile or companion-spec change |
| MQTT/Sparkplug | MQTT 5.0; Sparkplug 3.0 | transport semantics do not replace business idempotency | broker/profile upgrade |
| MTConnect | 2.5 current at research date | qualify read evidence model per device/agent | model or adapter version change |
| Asset/condition standards | ISO 55001:2024; 17359:2018; 13379/13381:2025 | governed maintenance program and uncertainty-aware diagnostics/prognostics | standard revision or asset-policy change |
| Maximo | current MAS documentation, installation-generated OAS | pin exact schema/config/role; test status side effects | MAS upgrade, object-structure/config/automation change |
| SAP | Cloud 2608 docs sampled; deprecation identified | pin customer release/API/ETag/status transitions | every release/API deprecation/configuration change |
| Dynamics 365 | current Learn docs; preview caveat retained | deterministic scheduling constraints; do not rely on disabled/preview features | product/config/feature-status change |
| ISO 9001 | 2015 + Amd 1:2024 current; 2026 under publication | do not call draft/unpublished edition current | immediately after ISO 9001:2026 publication |
| FDA QMSR | effective 2026-02-02 | validate applicable device QMS workflows against current rule | eCFR/FDA guidance or incorporated-standard change |
| Electronic records/CGMP | current eCFR Part 11/211 and FDA guidance | controlled records, approvals, audit, retention, investigation | legal/guidance or applicability change |
| Metrology/sampling | ISO 10012:2026; 17025:2017; 2859-1:2026 | deterministic sampling/valuation and measurement traceability | standard or site plan/method change |
| Identity/genealogy | IEC 81346-1:2022, -14:2026; EPCIS 2.0.1 | separate equipment reference designation from product identity | registry/genealogy standard or master-data change |
| Safety | OSHA current pages; ISO 12100:2010; ISO 10218:2025; IEC 61511 base 2016+A1 | hard human/deterministic boundary | law, site procedure, machine risk, or standard revision |
| OT cybersecurity | NIST 800-82r3; IEC 62443; CISA 2025 guidance | site isolation, zones/conduits, lifecycle/shared responsibility | threat, product, architecture, or standard change |
| NIST manufacturing profile | IR 8183r2 Initial Public Draft | do not treat draft as final | final NIST publication |
| Digital twin | ISO 23247-1:2021; NIST 2025–2026 work | simulation with VVUQ and explicit coverage limits | model/testbed or standard revision |
| Agent/runtime | current official model docs and OWASP 2026 | typed tools, approval boundaries, evals, poisoning/tool controls | model/tool/runtime or threat-list release |
| Telemetry | OTel spec 1.60 and semconv page current at access | stable domain schema; pin evolving GenAI convention | OTel/semconv upgrade |

## Areas of disagreement and chosen trade-offs

### Predictive maintenance versus simpler policies

Vendor material often emphasizes predictive maintenance. Asset-management and condition-monitoring standards instead support a program tied to failure modes, consequences, data quality, diagnostic/prognostic validity, and response policy. The playbook does not make prediction a maturity requirement. Meter-based preventive work or deterministic condition rules are preferable when they meet the operational need with less uncertainty and cost.

### Central cloud reasoning versus plant-local operation

Cloud models can provide capability and centralized governance; local models can reduce dependency and data movement. Neither placement resolves authority or safety. The selected hybrid keeps read capture and effect enforcement local/site-scoped, allows regional reasoning/governance, and defines an offline profile. A plant can choose more local execution after workload-specific validation.

### Multi-agent specialization versus one bounded coordinator

Agent-framework examples often decompose roles into multiple agents. That adds communication, identity, context, cascading-failure, and observability surfaces. No manufacturing requirement here needs autonomous peer agents. The playbook starts with one coordinator plus deterministic services; split services by domain/scale, not by simulated personas.

### “Exactly once” messaging versus business idempotency

Messaging specifications define delivery semantics, while vendor business APIs have their own acceptance, indexing, uniqueness, and state-transition behavior. The chosen approach assumes timeouts can make outcomes unknowable and uses a durable `UNKNOWN` state, semantic operation IDs, reconciliation, and read-back. It is more operational work but prevents the much worse blind-retry failure.

### Digital-twin proof versus staged live evidence

Simulation is safe and reproducible but inevitably incomplete. The selected evidence ladder moves from replay and contract tests through workflow/twin labs, live shadow, and a bounded canary. No simulation result authorizes bypassing a site commissioning, machine-safety, quality-validation, or change-control process.

### Fast learning versus controlled behavior release

Online memory or self-improvement can adapt quickly but risks poisoning, label leakage, regulatory/data-integrity failures, and irreproducible behavior. The playbook chooses slower curated datasets, reconciled outcomes, human review, regression suites, and signed releases. That is appropriate for high-consequence plant work.

## Nonfinal, deprecated, and limited sources retained deliberately

- **ISO 9001:2026:** under publication on 2026-08-31; not treated as current until publication and transition guidance are verified.
- **NIST IR 8183 Rev. 2:** Initial Public Draft; useful for horizon scanning only.
- **MIMOSA CCOM 4.1.0-RC1:** release candidate; 4.0.0 remains the identified stable release.
- **Microsoft Dynamics preview quality capabilities:** excluded from production baseline until generally available and qualified.
- **Deprecated SAP maintenance-order API:** cited only to require successor/version checks, never recommended.
- **ISO 14224:** industry-specific; used as a data/taxonomy pattern, not a universal mandate.
- **IEC 61511 2026 series package:** package date is not represented as a new 2026 core technical edition.
- **OPC certification overview material:** certification can support procurement/qualification, but current product listing and actual endpoint capabilities still require verification.
- **Standards landing pages:** authoritative for edition/status/scope, but not substitutes for licensed normative clauses.

## Weak approaches rejected

- direct LLM access to PLC/SCADA writes, generic OPC methods, MQTT publish, shell, browser, SQL, or vendor engineering tools;
- using prompt instructions as the only safety or authority boundary;
- treating EAM status as physical equipment state or QMS inspection completion as release;
- fuzzy matching equipment, tags, lots, or serials for effect targets;
- “latest document/value wins” without effective-time or correction semantics;
- assuming protocol QoS, HTTP retry, or vendor success response yields exactly-once business outcome;
- allowing the model to compute AQL/sample/tolerance/SPC or schedule safety/competency constraints;
- putting regulated records, secrets, plant content, and chat into one long-term vector memory;
- generating a narrative compaction summary that drops unknown effects or expired approvals;
- using aggregate model scores without deterministic safety/effect/identity oracles;
- treating a digital twin, benchmark, vendor demo, or model certification as production assurance;
- replaying offline queues after reconnect without authority/evidence/precondition revalidation;
- continuous self-learning or automatic memory ingestion from live plant outcomes;
- adopting a multi-agent swarm as a maturity milestone;
- claiming ISO/FDA/OSHA/IEC compliance from architecture documentation alone.

## Evidence-to-guide map

| Guide | Principal research inputs |
|---|---|
| [Mission and workload fit](../../agents/manufacturing-maintenance-quality-agent/01-mission-boundaries-and-workload-fit.md) | official agent-runtime guidance, maintenance/quality/safety authority analysis, no-agent comparison |
| [Architecture and OT boundaries](../../agents/manufacturing-maintenance-quality-agent/02-reference-architecture-and-ot-safety-boundaries.md) | ISA-95/88, NIST 800-82r3, IEC 62443, OPC UA/MQTT/Sparkplug |
| [Identity and evidence](../../agents/manufacturing-maintenance-quality-agent/03-identity-state-events-evidence-and-traceability.md) | IEC 81346, EPCIS, OPC Machinery, NIST traceability, ISO 10012 |
| [Maintenance](../../agents/manufacturing-maintenance-quality-agent/04-maintenance-strategy-condition-monitoring-and-work-orders.md) | ISO 55001/14224/17359/13374/13379/13381, IBM/SAP/Dynamics |
| [Quality](../../agents/manufacturing-maintenance-quality-agent/05-quality-plans-inspection-nonconformance-capa-and-release.md) | ISO 9001/13485/17025/2859/10393, FDA QMSR/Part 11/CGMP, SAP/Dynamics quality |
| [Adapters](../../agents/manufacturing-maintenance-quality-agent/06-tools-connectors-adapters-and-vendor-coordination.md) | OPC UA/MTConnect/MQTT/Sparkplug, Maximo/SAP/Dynamics APIs and version caveats |
| [Effects](../../agents/manufacturing-maintenance-quality-agent/07-planning-effects-reliability-and-post-action-verification.md) | vendor idempotency/concurrency behavior, protocol delivery limits, durable orchestration principles |
| [Memory and orchestration](../../agents/manufacturing-maintenance-quality-agent/08-context-memory-compaction-and-durable-orchestration.md) | typed agent/tool guidance, controlled-document/data-integrity requirements, poisoning threats |
| [Security and governance](../../agents/manufacturing-maintenance-quality-agent/09-security-governance-controlled-procedures-and-audit.md) | NIST/IEC/ISA/CISA, OWASP, FDA/eCFR, supply-chain controls |
| [Evaluation and simulation](../../agents/manufacturing-maintenance-quality-agent/10-observability-evaluation-simulation-and-failure-injection.md) | ISO 23247, NIST digital-twin/testbed work, OpenTelemetry, agent eval guidance |
| [Deployment and evolution](../../agents/manufacturing-maintenance-quality-agent/11-deployment-offline-scale-ha-dr-incidents-and-evolution.md) | OT availability constraints, effect semantics, site isolation, controlled releases |
| [Roadmap and runbooks](../../agents/manufacturing-maintenance-quality-agent/12-zero-to-production-roadmap-runbooks-and-exercises.md) | synthesis of all source families into measurable gates and drills |

## Refresh schedule

Review quarterly and on any material trigger:

- publication of ISO 9001:2026 and associated transition/accreditation guidance;
- final publication of NIST IR 8183 Rev. 2;
- ISA-95, ISA-88, IEC 62443, ISO 12100, IEC 61511, OPC UA, MTConnect, MQTT/Sparkplug, or relevant sector-standard revision;
- FDA/eCFR, OSHA, jurisdictional safety/quality, recall, electronic-record, AI, privacy, or cybersecurity change;
- installed MES/ERP/EAM/QMS/LIMS/edge upgrade, configuration/role change, endpoint deprecation, or vendor support change;
- model/provider snapshot, tool/runtime, prompt/retrieval, policy, knowledge, adapter, or observability-convention change;
- new plant/site, asset/product class, authority tier, workflow, regulation, data class, or connector;
- safety/quality/security incident, duplicate/unknown effect, false identity join, drift alert, recall/CAPA finding, or failed restore/rollback drill.

Assign named owners from maintenance/reliability, quality, operations, EHS/safety, OT engineering/security, enterprise integration, platform SRE, legal/regulatory, data governance, and model governance. Record the next review date and changed-source diff in the behavior-release evidence pack.

## Remaining implementation limitations

This packet cannot determine site-specific legal applicability, machine hazards, SIL/PL requirements, collective agreements, quality-unit authority, validated-system obligations, retention, electronic signatures, or vendor customization. Several standards require licensed normative text. Vendor documentation cannot reveal a customer’s automation scripts, status-domain changes, permissions, installed patches, or data quality. Digital-twin and replay evidence cannot cover unmodeled real-world behavior. These limitations are intentional stop conditions: resolve them through qualified site owners and real-system validation before increasing authority.

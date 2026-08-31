# Energy and Utilities Operations Agent Research Packet

> **Research date:** 2026-08-31  
> **Category:** #47 — Energy and utilities operations  
> **Output:** [production playbook](../../agents/energy-utilities-operations-agent/README.md)  
> **Method:** broad primary-source review across electric, gas, water/wastewater, interoperability, OT security, emergency management, worker safety, weather, simulation, utility vendors, AI risk and production runtime controls

This packet records the evidence and decisions behind the playbook. It is not a compliance opinion and does not replace an operator's effective regulations, tariffs, licenses, registrations, procedures, engineering standards, alarm philosophy, emergency plan, safety program, collective agreements, or vendor contracts.

## Research boundary and currency rules

The target workload is network/asset/customer service-condition truth, telemetry quality, forecasts, alarms/events, outage detection and verification, restoration-planning support, field coordination, switching/work plans as proposals, critical-load/safety constraints, communication preparation, and post-event reconciliation/reporting.

Excluded authority: protection, autonomous grid/plant/process control, SCADA commands, switching/valve/pump execution, generation dispatch, balancing/interchange, load shedding, setpoints, treatment changes, safety interlocks, clearances, qualified work, public-health declarations, public release, regulatory interpretation/signature/filing, and changes to policy or agent authority.

Research rules:

- Official regulator/operator/standards-body/product documentation was preferred.
- Versions, editions, effective dates and “final versus draft” status were recorded where public pages exposed them.
- Public abstracts of paid standards support scope/version claims only; the implementation must use licensed normative text.
- U.S. sources are concrete examples, not global defaults. EU and Indian system-operation sources were reviewed to verify role/jurisdiction variability.
- Vendor examples show why adapters need exact qualification; they are not endorsements.
- Agent/runtime sources supplement, never override, utility safety and regulatory obligations.

## Primary-source ledger

### Electric reliability, emergency operations, and reporting

| Source | Version/currency at review | Evidence used |
|---|---|---|
| [NERC Reliability Standards](https://www.nerc.com/standards/reliability-standards) | Current page reviewed 2026-08-31; complete set modified 2026-07-15; jurisdiction/effective-date views available | Standards are family-, registered-function-, jurisdiction- and effective-date-specific; do not embed a universal NERC rule set. |
| [NERC TOP-001-6](https://www.nerc.com/standards/reliability-standards/top/top-001-6) | Mandatory/effective 2024-04-01; page and related documents current in 2026 | Prompt action and qualified functional responsibilities remain operator-owned; agent cannot assume transmission-operator authority. |
| [NERC IRO-014-3](https://www.nerc.com/standards/reliability-standards/iro/iro-014-3) | Effective 2017-04-01; page current in 2026 | Inter-reliability-coordinator coordination is an established authority structure, not an AI collaboration pattern. |
| [NERC EOP-005-3](https://www.nerc.com/globalassets/standards/reliability-standards/eop/eop-005-3.pdf) | Current standard document found at review | Blackstart/system-restoration plans, facilities, coordination and personnel have explicit owners; agent remains advisory. |
| [NERC EOP-011-4 project clean draft](https://www.nerc.com/globalassets/standards/projects/2021-07/2021-07_ab_phase-2_eop-011-4_clean_august2023.pdf) | Draft/project artifact, not used as current enforceable text | Shows emergency-plan subject matter; implementation must resolve the approved/effective standard through NERC's current jurisdiction page. |
| [NERC CIP-008-6](https://www.nerc.com/globalassets/standards/reliability-standards/cip/cip-008-6.pdf) | Current document available at review | Cyber incident identification, classification, response and reporting need deterministic governed plans. |
| [NERC reliability guidelines](https://www.nerc.com/our-work/guidelines/reliability-guidelines) | Page reviewed 2026-08-31; includes current gas-electric coordination and DER guidance | Guidelines are useful but non-equivalent to mandatory standards; pin title/date and operator adoption. |
| [DOE/EIA survey forms](https://www.eia.gov/Survey/) | Current page reviewed 2026-08-31 | DOE-417 and EIA-861 have distinct triggers, reporting requirements and calculations; agent may prepare, not file. |
| [IEEE 1366-2022](https://standards.ieee.org/ieee/1366/7243/) | Active; published 2022-11-22 | Distribution reliability indices require defined factors/calculation methods. |
| [EIA reliability table/method notes](https://www.eia.gov/electricity/annual/html/epa_11_02.html) | 2024 data available in 2025 Electric Power Annual | SAIDI/SAIFI/CAIDI, major-event-day and loss-of-supply treatment vary; preserve method/denominator/exclusions. |
| [DOE monitoring and control technologies resilience guide](https://www.energy.gov/sites/default/files/2024-11/111524_Monitoring_and_Control_Technologies.pdf) | November 2024 | ADMS/DERMS/monitoring technologies can improve visibility/restoration, but integration and operator control boundaries remain critical. |
| [EU Commission Regulation 2017/1485](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=celex%3A32017R1485) | Current consolidated link exposed 2021-03-15 | TSO/DSO/SGU roles, operational security, planning and coordination demonstrate non-U.S. authority structures. |
| [CERC Indian Electricity Grid Code 2023](https://www.cercind.gov.in/Regulations/180-Regulations.pdf) | Effective 2023-10-01; current page showed later amendments/orders | Assigns specific NLDC/RLDC/SLDC/licensee/user responsibilities and adds protection/cyber/monitoring codes; pack must track amendments. |

### Interoperability, models, telemetry, and alarms

| Source | Version/currency at review | Evidence used |
|---|---|---|
| [IEC 61970-301:2020+A1:2022](https://webstore.iec.ch/en/publication/74467) | Edition 7.1; CIM17v38 base identified | CIM provides shared semantics for utility objects and integration; it does not assign operational authority. |
| [IEC 61970:2026 series](https://webstore.iec.ch/en/publication/61167) | Series catalog published 2026-07-10 | Individual EMS-API/CGMES parts have distinct editions; qualify profiles, not series name alone. |
| [IEC 61970-600-1:2021](https://webstore.iec.ch/en/publication/63866) and [600-2:2021](https://webstore.iec.ch/en/publication/63867) | CGMES structure/rules and exchange profiles | Grid-model exchange requires profile/version/conformance evidence; not a live-topology certificate. |
| [ENTSO-E CGMES library](https://www.entsoe.eu/data/cim/cim-for-grid-models-exchange/) | CGMES v3.0; current 2025–2026 artifacts listed | Machine-readable artifacts, test configurations and conformity versions change independently. |
| [ENTSO-E CIM conformity and interoperability](https://www.entsoe.eu/data/cim/cim-conformity-and-interoperability/) | Scheme v3 artifacts and 2025 IOP material current at review | Product/profile conformity needs exact test configuration and version. |
| [IEC 61968-3:2021](https://webstore.iec.ch/en/publication/67251) | Edition 3.0 | Covers distribution topology/status, fault/restoration, trouble and field coordination messages; strong domain boundary foundation. |
| [IEC 61968-4:2019](https://webstore.iec.ch/en/publication/61452) | Edition 2.0 | Asset/record exchanges include condition, analytics, alerts and work; source-specific authority still required. |
| [IEC 61968-6:2015](https://webstore.iec.ch/en/publication/22834) | Edition 1.0, listed valid at review | Maintenance/construction/work messages support WMS boundary; they do not authorize qualified work. |
| [IEC 61968-9:2024](https://webstore.iec.ch/en/publication/75041) | Edition 3.0 | Meter/MDMS integration supports outage/restoration and may apply to gas/water metering; underlying protocols remain out of scope. |
| [IEC 61968-13:2021](https://webstore.iec.ch/en/publication/34213) | Edition 2.0 | Balanced/unbalanced distribution network profiles support analysis; version and extension handling are necessary. |
| [IEC 61968-100:2022](https://webstore.iec.ch/en/publication/67766) | Edition 2.0; generally not backward compatible with 2013 messages | “IEC 61968 compatible” is insufficient; message/profile version is material. |
| [IEC 61850:2026 series](https://webstore.iec.ch/en/publication/6028) | Series catalog published 2026-08-20 | Semantic models, SCL and communication services span many independently versioned parts. |
| [IEC 62351:2026 series](https://webstore.iec.ch/en/publication/6912) | Series catalog published 2026-07-30 | Security parts cover different protocols/functions; exact implementation/profile matters. |
| [IEC 62351 overview for IEC 61850](https://iec61850.dvl.iec.ch/what-is-61850/technical-principles/61850-cybersecurity/) | Current site reviewed 2026-08-31 | Protocol security, monitoring, RBAC and key management are layered capabilities, not one switch. |
| [IEC 62351-7:2025](https://webstore.iec.ch/en/publication/76108) | Edition 2.0, published 2025-12-03 | Power-system-specific network/system-management objects reinforce separate device/protocol health observability. |
| [IEC 62682:2022](https://webstore.iec.ch/en/publication/65543) | Edition 2.0 | Alarm systems require lifecycle management; external analytics may consume logs but not replace operator alarms. |
| [ISA-18 series](https://www.isa.org/standards-and-publications/isa-standards/isa-18-series-of-standards) | ANSI/ISA-18.2-2016 plus current technical reports listed | Alarm philosophy, rationalization, design, monitoring, audit and MOC are governed processes. |
| [OPC Foundation overview of OPC UA/IEC 62541](https://opcfoundation.org/wp-content/uploads/2023/05/OPC-UA-Interoperability-For-Industrie4-and-IoT-EN.pdf) | 2023 overview | OPC UA has information/security/service profiles; deployment conformance and read/control separation remain essential. |

### Gas and pipeline operations

| Source | Version/currency at review | Evidence used |
|---|---|---|
| [49 CFR 192.631, current eCFR](https://www.ecfr.gov/current/title-49/subtitle-B/chapter-I/subchapter-D/part-192/subpart-L/section-192.631) | Current law must be checked at implementation | Covered gas pipeline control-room management includes SCADA/controller, information, fatigue, alarm, change and training obligations. |
| [PHMSA control-room management](https://www.phmsa.dot.gov/pipeline/control-room-management/control-room-management) | Updated 2025-03-17 | Regulations govern controllers/control rooms/SCADA and human factors; agent cannot be a pipeline controller. |
| [PHMSA CRM inspection guidance](https://www.phmsa.dot.gov/pipeline/control-room-management/crm-workshops-and-inspection-guidance) | Current page updated in 2026 | Written alarm management, records and effective controller response are inspectable operator processes. |
| [49 CFR 192.615, current eCFR](https://www.ecfr.gov/current/title-49/subtitle-B/chapter-I/subchapter-D/part-192/subpart-L/section-192.615) | Current law must be checked at implementation | Gas emergency plans and liaison/response obligations require operator-specific governance. |
| [PHMSA operator qualification](https://www.phmsa.dot.gov/pipeline/operator-qualifications/about-operator-qualification) | Updated 2025-03-18 | Covered tasks require documented qualification and recognition/reaction to abnormal operating conditions. |
| [PHMSA gas distribution integrity management](https://www.phmsa.dot.gov/pipeline/gas-distribution-integrity-management/gas-distribution-integrity-management-program-dimp) | Rule/program foundation; current inspection material separately tracked | System knowledge, threats, risk, mitigations, performance and improvement support evidence boundaries. |
| [PHMSA 2025 gas distribution inspection question set](https://www.phmsa.dot.gov/sites/phmsa.dot.gov/files/2025-01/PHMSA-Gas-Distribution-GD-2025-01-IA-Question-Set-January-2025.pdf) | GD.2025.01 | Demonstrates current inspection attention to system knowledge, sources, threat/risk methods and records. |

### Water and wastewater operations

| Source | Version/currency at review | Evidence used |
|---|---|---|
| [EPA drinking-water ERP guidance](https://www.epa.gov/waterutilityresponse/develop-or-update-emergency-response-plan) | Drinking-water template 09/2024; wastewater template updated 10/2025 | AWIA/SDWA scope is size/service-specific; emergency plans incorporate risk findings and cyber content. |
| [EPA AWIA certification](https://www.epa.gov/waterresilience/how-certify-your-risk-and-resilience-assessment-or-emergency-response-plan) | Updated 2026-07-02 | Certification, retention and utility official roles are regulated processes; agent cannot certify. |
| [EPA incident-action checklists](https://www.epa.gov/waterutilityresponse/incident-action-checklists-water-utilities) | Current page reviewed 2026-08-31; 2024–2025 cyber/contamination materials | Provides operator emergency preparation/response/recovery structures and qualified coordination points. |
| [EPA water-contamination response resources](https://www.epa.gov/waterresilience/water-contamination-response-resources) | Updated 2026-02-25 | Incident framework, response partners, exercises and risk communication support human-owned contamination decisions. |
| [EPA WaterCIRP](https://www.epa.gov/waterutilityresponse/water-contamination-incident-remediation-plan-watercirp) | Guide/template 2024; page updated 2026-05-28 | Remediation must be implemented, monitored and evaluated under utility/public-health ownership. |
| [NIST SP 1800-45](https://www.nist.gov/publications/cybersecurity-water-and-wastewater-sector) | Final 2026-06-24; supersedes TN 2283 draft | Current primary source for water/wastewater OT remote-access reference designs. |
| [AWWA risk and resilience resources](https://www.awwa.org/resource/risk-resilience/) | J100-21 and related G430/G440 resources current on page | Paid standards require licensed text; public pages establish all-hazards risk/emergency/security scope. |
| [EPA EPANET](https://www.epa.gov/water-research/epanet) | EPANET 2.2 official release; page updated 2025/2026 | Supports hydraulic/water-quality simulation; version, model and convergence must be recorded. |

### Safety and emergency coordination

| Source | Version/currency at review | Evidence used |
|---|---|---|
| [OSHA 29 CFR 1910.269](https://www.osha.gov/laws-regs/regulations/standardnumber/1910/1910.269) | Current public rule page reviewed 2026-08-31 | Qualified employees, system operator, designated employee, de-energization, tagging, testing and grounding remain human/procedure owned. |
| [OSHA 1926 Subpart V](https://www.osha.gov/laws-regs/regulations/standardnumber/1926/1926subpartv) | Current rule page | Construction work has separate electric T&D requirements; jurisdiction pack must distinguish work types. |
| [OSHA 1910.147](https://www.osha.gov/laws-regs/regulations/standardnumber/1910/1910.147) | Current rule page | General hazardous-energy control has scope/exclusions; do not apply it blindly to utility T&D work. |
| [FEMA Community Lifelines](https://www.fema.gov/emergency-managers/practitioners/lifelines) | 2024 doctrine update noted | Energy, water, communications, transport and health dependencies support cross-lifeline analysis under incident command. |
| [FEMA NIMS command and coordination](https://www.usfa.fema.gov/a-z/nims/command-and-coordination.html) | Current 2026 page | Separates tactical, EOC support, policy and public communication roles; agent must not create parallel command. |
| [FEMA Community Lifelines Toolkit 2.1](https://www.fema.gov/sites/default/files/documents/fema_lifelines-toolkit-v2.1_2023.pdf) | Version 2.1, 2023 | Provides stabilization information examples, including energy/water and critical-facility dependencies. |
| [DOE/CESER response and recovery](https://www.energy.gov/ceser/response-recovery) | Current page reviewed 2026-08-31 | Energy emergency coordination, situation reports and restoration support remain official/human functions. |

### Weather, forecasts, and official hazard data

| Source | Version/currency at review | Evidence used |
|---|---|---|
| [NWS API](https://www.weather.gov/documentation/services-web-api) | Updated 2026-03-24 | Forecast/alert/observation API, cache lifecycle, known issues, service-change process and rate limits require adapter controls. |
| [NWS CAP alert service](https://www.weather.gov/documentation/services-web-alerts) | CAP v1.2 service documentation current at review | Alert update/cancel, urgency/severity/certainty, geospatial zone and resilient dissemination semantics matter. |
| [NHC model guidance](https://www.nhc.noaa.gov/modelsummary.shtml) | Current 2026 page | Official forecasts incorporate models/forecaster judgment and uncertainty; do not rely on a single model output. |
| [NHC forecast verification](https://www.nhc.noaa.gov/verification/) | Current 2026 page | Forecast performance is measured and should inform adapter/applicability claims. |
| [NOAA forecasting uncertainty fact sheet](https://repository.library.noaa.gov/view/noaa/69977/noaa_69977_DS1.pdf) | 2025 | Supports distributional/probabilistic communication and explicit uncertainty. |

### OT cybersecurity, AI risk, and observability

| Source | Version/currency at review | Evidence used |
|---|---|---|
| [NIST SP 800-82 Rev. 3](https://www.nist.gov/publications/guide-operational-technology-ot-security) | Final 2023-09-28; Rev. 4 only pre-draft call in 2026 | Current final OT guidance; safety, reliability, performance and architecture constrain security controls. |
| [NIST IR 7628 Rev. 1](https://csrc.nist.gov/pubs/ir/7628/r1/final) | Final 2014 | Smart-grid risk/security/privacy architecture remains a foundational source but needs current supplements. |
| [DOE distribution/DER cyber baselines](https://www.energy.gov/ceser/cybersecurity-baselines-electric-distribution-systems-and-der-and-guidance) | 2024 baseline plus 2025 interim implementation guidance | Distribution/DER scoping and prioritized baseline approach; voluntary/operator/regulator context explicit. |
| [CISA Secure by Demand for OT](https://www.cisa.gov/resources-tools/resources/secure-demand-priority-considerations-ot-owners-and-operators-when-selecting-digital-products) | 2025-01-13 | Product selection should demand layered security and not rely solely on segmentation. |
| [NIST AI 600-1](https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence) | Final 2024; page updated 2026-04-08 | Govern/map/measure/manage generative-AI risks across lifecycle. |
| [NIST trustworthy AI in critical infrastructure profile initiative](https://www.nist.gov/programs-projects/concept-note-ai-rmf-profile-trustworthy-ai-critical-infrastructure) | Ongoing, started 2026-04; not a final profile | Confirms active work; do not present the future profile as completed guidance. |
| [OWASP Agentic Top 10 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) | Version 2026, published 2025-12-09 | Goal hijack, tool misuse, identity/privilege, memory and cascading failure risks inform threat tests. |
| [OpenTelemetry general semantic conventions](https://opentelemetry.io/docs/specs/semconv/general/) | Current evolving specification | Supports common traces/metrics/logs/events; stable application audit remains separate. |

### Representative vendor, provider, and simulator evidence

| Source | Version/currency at review | Evidence used |
|---|---|---|
| [Esri Utility Network subnetworks](https://pro.arcgis.com/en/pro-app/3.5/help/data/utility-network/subnetworks.htm) | ArcGIS Pro 3.5 docs; implementation must pin installed versions | Subnetworks, controllers, dirty state and topology validation affect trace meaning. |
| [Esri Utility Network editing](https://pro.arcgis.com/en/pro-app/3.5/help/editing/edit-a-utility-network.htm) | 3.5 docs | Named branch versions and later reconciliation to Default prove rendered/current model distinctions. |
| [Esri Utility Network administration](https://pro.arcgis.com/en/pro-app/3.5/help/data/utility-network/utility-network-dataset-administration.htm) | 3.5 docs | Product and Utility Network versions affect schema and upgrade behavior. |
| [Oracle Utilities Network Management System](https://docs.oracle.com/en/industries/energy-water/network-management-system/) | Current product documentation landing page reviewed 2026-08-31 | NMS combines outage/advanced distribution and water capabilities; exact release/security/API docs are required locally. |
| [NWS API/CAP](https://www.weather.gov/documentation/services-web-api) | See above | Official provider adapter example with rate/known issue/update behavior. |
| [OpenDSS](https://opendss.epri.com/) | Current public documentation reviewed 2026-08-31 | Electric distribution simulator; executable/model/profile validation required. |
| [GridLAB-D releases](https://github.com/gridlab-d/gridlab-d/releases) | v5.3.0 shown latest at review | Pin exact release/commit; release chronology and artifacts change. |
| [EPA EPANET 2.2](https://www.epa.gov/water-research/epanet) | Official 2.2 | Water simulator reference; model validity is separate from software availability. |

## Findings synthesized across sources

### Standards improve exchange, not authority or freshness

CIM, IEC 61968, IEC 61850, CGMES, DNP3/IEC 60870, OPC UA and vendor network models can preserve semantics. None decides which deployed source owns a field, whether topology is current, whether telemetry coverage is sufficient, or whether an operator may act.

**Decision:** require a field-level source matrix, topology/operational overlays, exact profile versions, adapter conformance and qualified-human authority.

### Monitoring and control frequently share technology

SCADA/telecontrol/AMI products may expose observations and commands through the same protocol/product. Client conventions do not provide sufficient isolation.

**Decision:** obtain observations through replicas/gateways; separate endpoint, route, identity, server permissions and proxy methods; test prohibited controls. The model runtime has no control-plane route.

### Utility truth is plural, temporal, and quality-coded

GIS, operational topology, SCADA, OMS, AMI, customer, field and work sources legitimately disagree. Late events, clock resets, substitutions and temporary configurations are normal.

**Decision:** immutable observations, explicit time/quality/coverage/correction, temporal identity mappings and rebuildable projections. No universal last-write-wins.

### Alarms are managed operator interfaces

IEC 62682/ISA-18 and PHMSA material treat alarm systems as lifecycle/human-response systems, not generic analytics.

**Decision:** agent can correlate/explain but never reprioritize, acknowledge, shelve, suppress, inhibit, rationalize or change alarm configuration during operations.

### Restoration has levels of proof

An applied network/work action, device read-back, meter power-up, stable service, critical exception resolution, operator closure and post-event administrative completion are different.

**Decision:** R0–R5 ladder; work closure or transport success never terminalizes service.

### Forecasts require distributions and applicability

Official weather providers expose revision/uncertainty and recommend official forecast products over individual models. Utility forecasts also drift by territory/hazard/asset/data availability.

**Decision:** version/as-of/horizon/distribution/calibration/slice/applicability/use-ceiling contract; no invented model confidence or autonomous ETR publication.

### Safety and command responsibility are non-delegable

OSHA, PHMSA, NERC, EPA and FEMA sources assign responsibilities to qualified employees, operators, controllers, incident command and officials.

**Decision:** agent produces evidence/proposals; control, switching, isolation, covered work, treatment, public-health, emergency and release decisions stay in qualified channels.

### Physical operations do not have generic rollback

Even allowed coordination writes can be asynchronous and ambiguous; physical restoration actions are outside the effect gateway and often irreversible.

**Decision:** stable semantic operation ID, exact approval, attempt ledger, unknown state, independent read-back, cancellation semantics and correction/forward recovery.

### Storm scale changes the bottleneck

Observation/case/model fan-out, field reconnect and post-event reconciliation can exceed source/provider and operator capacity. Operator attention is limited.

**Decision:** priority/reserved queues, fair per-cell admission, coalescing, deterministic fallback, recovery-load throttling and operator-workload SLOs.

### Memory must not become an alternate control room

Old incidents contain obsolete topology, policy, customer data, false hypotheses and prompt injection.

**Decision:** exact seven-class memory model; operational truth stays durable/authoritative; curated episodic outcomes are non-authoritative, isolated, expiring and offline reviewed.

## Architecture decision records

### ADR-EUO-001: advisory-only control boundary

**Decision:** model authority is read/analyze/propose. U4 control/safety operations are architecturally unreachable.

**Rejected:** direct SCADA/ADMS/DMS/AMI control with human confirmation; approval does not make a generative control path safe or qualified.

### ADR-EUO-002: one durable coordinator per case

**Decision:** fixed macro-state machine with bounded model/tool steps.

**Rejected:** multi-agent storm swarm; it increases load, conflict and operator noise without solving durability or authority.

### ADR-EUO-003: authoritative systems plus immutable coordination records

**Decision:** source systems retain field authority; store raw/normalized evidence, rebuildable projections, case events and U3 ledger.

**Rejected:** central “AI digital twin” or transcript as operational truth.

### ADR-EUO-004: deterministic detection and feasibility

**Decision:** rules detect/correlate; validated topology/physics/hydraulic/optimization tools decide status; model explains.

**Rejected:** generative outage detection, affected-customer counting or physical feasibility.

### ADR-EUO-005: signed jurisdiction/operator packs

**Decision:** effective regulations/procedures/roles/templates are offline reviewed and signed per operating unit.

**Rejected:** live web retrieval or model memory as incident-time regulatory truth.

### ADR-EUO-006: exact memory classes and loss-aware continuity

**Decision:** scratch, working/run, session, durable workflow/task, domain knowledge, preference and episodic/outcome have separate authority/retention; compaction emits typed receipt.

**Rejected:** undifferentiated long-term memory and prose-only summary.

### ADR-EUO-007: narrow U3 effects only

**Decision:** optional mature effects are approved coordination artifacts, with idempotency and reconciliation.

**Rejected:** a generic utility write tool or model-generated control instructions labeled “draft.”

### ADR-EUO-008: cell-based deployment and controlled evolution

**Decision:** isolate utility/commodity/authority cells; immutable behavior bundles; shadow, canary, rollback; offline feedback.

**Rejected:** global shared tenant and online self-learning.

## Jurisdiction decision template

Before implementation, answer for each operating unit:

| Question | Required evidence |
|---|---|
| Which entity/function/operator has authority? | Registration, license, tariff, regulation, delegation and on-duty roster |
| Which standards/rules are effective? | Official current/effective-date status and amendments |
| Which worker/controller qualifications apply? | Safety/OQ/training records and operator procedure |
| Which alarm/emergency/restoration plans govern? | Current approved plan and incident roles |
| Which reporting/communication duties trigger? | Deterministic rules, clocks, recipient, confidentiality, signer and correction process |
| Which source owns each field? | Signed source matrix and conflict owner |
| Which customer/critical-infrastructure data is sensitive? | Privacy/security classification and permitted audiences |
| Which effects can the agent coordinate? | U3 allowlist, approval/read-back and explicit U4 prohibition |
| What changes invalidate the pack? | Regulation/procedure/system/role/template/territory triggers |

## Research gaps and genuine limitations

- Many IEC, IEEE, ISA and AWWA normative texts are paywalled. Public official abstracts established scope/version; implementers need licensed text and domain counsel/engineers.
- Vendor SCADA/EMS/DMS/ADMS/OMS/AMI/WMS deployments are highly configured, and many detailed APIs/security guides require customer portals. The playbook therefore mandates local qualification rather than inventing universal semantics.
- No one public source defines a universal electric/gas/water “restoration truth” contract. The R0–R5 ladder is an engineering synthesis that must be mapped to operator procedures.
- Gas-network simulator selection was not standardized because operator models and pipeline classes differ; the playbook requires an operator-qualified tool.
- NIST's trustworthy-AI-in-critical-infrastructure profile was only an ongoing initiative at review, not final guidance.
- The packet does not enumerate every country, state/province, municipality, reliability region, pipeline program, public-health authority or tariff. The signed pack pattern makes those local decisions explicit.
- Numerical SLOs, queue weights, detection thresholds, stability windows, retention and forecast gates are intentionally not universal. They require measured operator data and approval.
- Nuclear operations, market trading, generation plant process control, autonomous DER control, telecom network operations, generic SRE and generic field service are outside this category.

## Refresh triggers

Refresh this packet when any of the following occurs:

- IEC 61968/61970/61850/62351, IEEE 1366, ISA-18/IEC 62682 or relevant utility profiles change materially;
- NERC/FERC/EIA, PHMSA, EPA, OSHA, EU/Indian or local requirements/interpretations change;
- NIST SP 800-82 Rev. 4 or the NIST critical-infrastructure AI profile becomes final;
- deployed GIS/ADMS/OMS/AMI/WMS/weather/simulator products or interfaces change;
- a utility incident reveals new identity, topology, quality, approval, restoration, effect or storm-scale failure;
- models/providers, agent threat guidance, observability standards or data-residency terms change;
- a new commodity, utility, jurisdiction, control center, case type or U3 effect is proposed.

## Research completion assessment

The research produced material guidance across domain boundary, authoritative identity/state, utility topology, telemetry/alarm semantics, outage detection/restoration verification, forecast uncertainty, qualified field/safety authority, standards and vendor qualification, OT security, context/memory/durability, distributed effects, evaluation/simulation, storm capacity, DR and governed evolution. Additional searches were yielding mainly duplicative descriptions or operator-private implementation details; those are correctly deferred to per-deployment qualification.

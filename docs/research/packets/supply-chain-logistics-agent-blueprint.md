# Research Packet: Supply-Chain and Logistics Operations Agent Blueprint

Research date: 2026-08-31  
Blueprint category: 33 — supply-chain and logistics operations  
Output: [supply-chain and logistics operations blueprint](../../agents/supply-chain-logistics-agent/README.md)  
Research status: Pass 2 production blueprint refined; provider, partner, standards, and regulatory contracts require operating-unit qualification and ongoing refresh

This packet records the evidence and engineering decisions behind the blueprint. It is not a general market survey. Research focused on the contracts that determine whether an agent can act safely: logistics identity, event semantics, document standards, vendor integration behavior, probabilistic planning, constraint solving, authority, distributed effects, evaluation, and operations.

## Research questions

1. Which business boundary makes this category distinct from procurement, manufacturing, database/DataOps, and generic back-office agents?
2. Which identities and event semantics survive real order, inventory, shipment, leg, and logistics-unit behavior?
3. What do logistics standards solve, and what application semantics do they explicitly leave unresolved?
4. How do current carrier, ERP, WMS, and TMS interfaces constrain authentication, quotas, versioning, idempotency, and reconciliation?
5. How should ETA and demand uncertainty be represented and evaluated?
6. Which decisions belong to deterministic constraints/optimization, the language model, policy, or a human?
7. What runtime/state/effect architecture is necessary for long-lived exceptions and ambiguous writes?
8. Which security, privacy, dangerous-goods, customs, and authority controls must remain outside the model?
9. How should the system be evaluated under late data, identity conflict, partial effects, provider failure, prompt injection, and disruption scale?
10. What release, SLO, capacity, cost, incident, and evolution controls are required after initial rollout?

## Research method and selection policy

Primary sources were the foundation: standards-body publications, official specifications and repositories, official provider API documentation, current vendor product/security documentation, and established forecast/optimization references. Research also used official engineering and risk guidance for agent evaluation and security. Important conclusions were cross-checked across sources that expose different layers—for example, EPCIS event semantics against carrier API behavior, or optimization library statuses against forecasting uncertainty literature.

Sources were rejected or demoted when they were undated marketing pages, generic "AI supply chain" summaries, copied integration tutorials, vendor claims without a usable contract, or framework examples that did not address external effects. Standards roadmaps and working drafts were recorded as future signals, not treated as deployed behavior. No source was copied verbatim into the blueprint.

The packet distinguishes:

- a **normative standard claim** from an implementation recommendation;
- an **official provider contract** from a provider-independent invariant;
- an **endorsed/stable release** from a working draft or roadmap;
- a **forecast or solver result** from observed operational truth;
- a **vendor product capability** from proof that an end-to-end agent is safe.

## Primary source register

All sources were accessed on 2026-08-31 unless a publication/release date is stated.

### Interoperability, identity, and traceability

| Source | Currency at review | Material evidence | Blueprint use |
|---|---|---|---|
| [OASIS Universal Business Language 2.4](https://docs.oasis-open.org/ubl/UBL-2.4.html) | OASIS Standard, 2024-06-20 | Defines business documents including Order, DespatchAdvice, ReceiptAdvice, TransportStatus, and InventoryReport. Receipt advice can represent shortages/damage; dispatch relationships need not be one-to-one with order lines. | Preserve document semantics and split/partial relationships; do not mistake schemas for the whole physical process. |
| [GS1 EPCIS and CBV 2.0](https://ref.gs1.org/standards/epcis/2.0.0/) | Ratified GS1 standard | Separates `eventTime` from `recordTime`; supplies business step, read point, and business location concepts; error declarations add corrective information rather than deleting history. Asynchronous capture uses a capture ID and can require later status/query verification. | Observation envelope, correction model, event/capture/ingest separation, and `202`-is-not-completion rule. |
| [GS1 General Specifications](https://ref.gs1.org/standards/genspecs/) | Release 26.0, January 2026 | Current GS1 system rules and identification-key definitions. | Version reference for identification semantics. |
| [GS1 identification keys](https://www.gs1.org/standards/id-keys) and [SSCC](https://www.gs1.org/standards/id-keys/sscc) | Current public guidance | GTIN identifies trade items, GLN parties/locations, SSCC logistics units, GSIN shipments, and GINC consignments. | Namespaced canonical identity examples; standards keys remain mappings, not automatic truth. |
| [GS1 Global Traceability Standard](https://www.gs1.org/standards/gs1-global-traceability-standard/current-standard) | Current standard landing page | Traceability depends on consistent identification, data capture, and sharing across parties. | Supports typed identity/event lineage without claiming a single global source. |
| [UN/CEFACT supply-chain management standards](https://unece.org/trade/cefact/uncefact/supply-chain-management) | Current portfolio | UN/EDIFACT and related trade/supply-chain standards provide structured message exchange. | EDI adapter boundary and partner/version-specific conformance. |
| [X12 transaction-set catalog](https://x12.org/products/transaction-sets) | Current official catalog at review | 204 is a motor-carrier load tender, 990 its business response, 214 shipment status, 997 syntactic acknowledgement, and 999 syntactic/relational implementation acknowledgement; 997/999 explicitly do not cover business semantics. | Keep envelope/technical acknowledgement, tender acceptance, shipment status, and cancellation as separate operation states under a bilateral implementation guide. |
| [UN/LOCODE downloads](https://unece.org/trade/cefact/UNLOCODE-Download) | Page lists release 2025-1 at review | Identifies ports, airports, inland clearance depots, terminals, and other trade/transport locations. | Location namespace with versioned master-data mapping. |
| [DCSA Track & Trace documentation](https://dcsa.org/standards/track-and-trace/standard-documentation-track-and-trace) | Lists 2.2 as latest stable published documentation | Ocean-container track-and-trace interface and event vocabulary. | Pin partner-supported version; do not assume roadmap version. |
| [DCSA members and adoption](https://dcsa.org/about-us/members) | Current member page | Describes member implementation of Track & Trace 2.2. | Adoption signal, not universal carrier coverage. |
| [DCSA standards roadmap 2026](https://dcsa.org/newsroom/dcsa-standards-roadmap-2026) | 2026 roadmap | Describes Track & Trace 3.0 alpha/beta work and planned reefer/IoT extensions during 2026. | Explicit stable-vs-roadmap distinction and refresh trigger. |
| [DCSA Booking 2.0 / Bill of Lading documentation](https://dcsa.org/standards/booking/documentation-booking-2) | Current final documentation at review | Standardized booking and shipping-instruction/bill-of-lading interfaces exist separately from tracking. | Avoid one generic ocean connector; distinguish read events from booking/document effects. |
| [IATA ONE Record repository](https://github.com/IATA-Cargo/ONE-Record), [2025-07 release](https://github.com/IATA-Cargo/ONE-Record/releases/tag/2025-07), and [2026-07 proposal folder](https://github.com/IATA-Cargo/ONE-Record/tree/2026-07/2026-07-standard) | Last clearly endorsed tag found: `2025-07`, Ontology 3.2.0, API 2.2.0, Data Orchestration 1.1.0 | Repository separates endorsed releases, proposals, and working drafts. The official `2026-07` tagged folder describes a proposal pending full endorsement and contains a placeholder endorsement date. | Pin the endorsed ontology/API/orchestration/security set actually supported by the partner; qualify newer proposal/draft material separately. |
| [IATA ONE Record issue 255](https://github.com/IATA-Cargo/ONE-Record/issues/255) | Open repository discussion at research time | Discussion exposes unresolved/incomplete guidance around distributed synchronization, eventual consistency, and idempotency. | Cross-check: a shared ontology/API does not solve distributed convergence or effect semantics. |

### Carrier and logistics-service interfaces

| Source | Currency at review | Material evidence | Blueprint use |
|---|---|---|---|
| [DHL Global Forwarding Shipment Tracking API v2](https://developer.dhl.com/api-reference/shipment-tracking-v2-dhl-global-forwarding) | Current developer portal contract | Subscription-key authentication, production/sandbox behavior, product quota information, and searches by supported shipment references. | Per-product adapter, account scope, quota/backpressure, and typed identifiers. |
| [DHL Unified Shipment Tracking](https://developer.dhl.com/tracking) | Current developer portal contract | A separate tracking product with its own initial daily and per-second limits. | Evidence that "DHL tracking" is not one universal contract. |
| [DHL Freight developer notices](https://developer.dhl.com/dhl-freight) | Includes 2026 backend/timestamp notice at review | Announced timestamp format change to add explicit timezone information. | Provider schema-drift test and reason to preserve explicit time semantics. |
| [FedEx Track API](https://developer.fedex.com/api/en-us/catalog/track/v1/docs.html) | Current official docs | Tracking by tracking number/reference, account-sensitive operations, and multiple-piece query limits. | Scoped identity, batch limits, and contract tests. |
| [FedEx API authorization](https://developer.fedex.com/api/en-us/catalog/authorization/docs.html) | Current official docs | OAuth bearer tokens expire and must be regenerated; documentation states a 60-minute token lifetime. | Runtime token refresh with scoped identity; no secrets in model context. |
| [FedEx webhook documentation](https://developer.fedex.com/api/en-us/catalog/tracking-number-subscription.html) | Current official docs | Tracking subscription/webhook capability and provider workflow. | Authenticated push channel plus polling/read-back fallback. |

These provider examples were not used to define universal quota numbers. They demonstrate that authentication, batches, limits, timestamps, account scope, webhook behavior, and product/version boundaries are moving adapter properties.

### ERP, inventory, warehouse, and transport systems

| Source | Currency at review | Material evidence | Blueprint use |
|---|---|---|---|
| [Microsoft Dynamics 365 Inventory Visibility APIs](https://learn.microsoft.com/en-us/dynamics365/supply-chain/inventory/inventory-visibility-api) | Current Microsoft documentation | Provides on-hand, reservation, allocation, reallocation, consumption and bulk operations with platform-specific authentication/limits. | Separate typed inventory operations; preserve upstream semantics and batch limits. |
| [Dynamics Inventory Visibility reservations](https://learn.microsoft.com/en-us/dynamics365/supply-chain/inventory/inventory-visibility-reservations) | Current Microsoft documentation | Soft reservations have reservation IDs, offsets, dimensions, and configuration-dependent behavior. | Never reduce inventory effects to `update quantity`; bind segments, source version, and reservation identity. |
| [Dynamics Inventory Visibility allocation](https://learn.microsoft.com/en-us/dynamics365/supply-chain/inventory/inventory-visibility-allocation) | Current Microsoft documentation | Allocation is a virtual pool and differs from physical inventory reservation; reallocate/consume have distinct semantics. | Canonical vocabulary must not flatten allocation and reservation into one fact. |
| [Oracle Warehouse Management RESTful Web Services 26C](https://docs.oracle.com/en/cloud/saas/warehouse-management/26c/owmre/restful-web-services.html) | Oracle 26C docs at review | Legacy APIs remain while newer fine-grained REST APIs replace older patterns. | Version/deprecation tracking and narrow resources; avoid legacy-wide generic calls. |
| [Oracle Transportation Management Security Guide 26A](https://docs.oracle.com/en/cloud/saas/transportation/26a/otmse/security-guide.pdf) | Oracle 26A guide | Describes OAuth and resource/endpoint/method access controls, including separation of view and update privileges. | Map least privilege to exact read/update operations and resource scope. |
| [SAP Business Network Freight Collaboration carrier API integration guide](https://help.sap.com/doc/how-to-integrate-with-sap-business-network-freight-collaboration-carrier-apis/LBN/en-US/SAP_LBN_API_HowToGuide.pdf) | Official SAP guide available at review | Shows mixed SOAP/REST integration boundaries and middleware migration/deprecation considerations. | Adapter topology/version migration is operational work; not a single generic transport tool. |

### Mapping, customs, documents, and telemetry

| Source | Currency at review | Material evidence | Blueprint use |
|---|---|---|---|
| [Google Maps Routes API: compute route matrix](https://developers.google.com/maps/documentation/routes/compute_route_matrix) and [location inputs](https://developers.google.com/maps/documentation/routes/specify_location-rm) | Current docs; route reference updated 2026-08 | Per-element errors exist; standard matrix maximum is 625 elements, reduced to 100 for transit or traffic-aware-optimal; place IDs are preferred and coordinates can snap to an unsuitable road. Region/EEA terms can vary. | Mapping result is a cost/time input, not proof of truck access, commercial lane, capacity, border, port, dangerous-goods, or regulatory feasibility. Pin region, request mode/preferences, fallback and per-element status. |
| [U.S. CBP: how to use ACE](https://www.cbp.gov/trade/automated/how-to-use-ace) and [ACE Cargo Release implementation guide](https://www.cbp.gov/sites/default/files/2025-07/ACE%20Cargo%20Release%20Implementation%20Guide_V40%20July%201%202025_508.pdf) | ACE channel page current; Cargo Release guide v40 dated 2025-07-01 | ACE uses different EDI, portal, and Document Image System channels for manifests, cargo release, supporting documents, post-summary corrections, and other acts. | Customs adapter must pin jurisdiction, channel, message/guide revision, filer authority and receipt/status semantics; U.S. ACE is not a global customs abstraction. |
| [European Commission ICS2 Release 3 deployment guidance](https://taxation-customs.ec.europa.eu/document/download/5b67b4ab-ece6-4dc6-a3be-8b938d603abc_en) | Guidance dated 2025-03; Release 3 deployment window ended 2025-09-01, with mode/member-state transition guidance | Economic operators and national service desks had mode-specific transition obligations and multiple filing interactions. | Evidence that customs versions and rollout windows are jurisdiction/mode/party-specific; implementation must re-check current national derogations and message contracts. |
| [OASIS MQTT 5.0](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html) | OASIS Standard, 2019-03-07 | QoS applies to delivery between sender and receiver; sessions, retained messages, duplicates, expiry, reason codes, flow control, and storage limits have explicit semantics. | Telematics ingestion needs message dedupe/session/expiry qualification; broker QoS does not prove device binding, sensor calibration, custody, or physical truth. |
| [GS1 EPCIS 2.0 sensor data](https://ref.gs1.org/standards/epcis/2.0.0/) | Ratified release 2.0 | Sensor reports can identify measurement type/exception, device, metadata, raw data, processing method, and time; sensor elements abstract business-oriented data from raw streams. | Preserve sensor/device/provenance/times and raw-artifact references; evaluate calibration and asset binding outside the model. |
| [DCSA Reefer Events documentation](https://dcsa.org/standards/track-and-trace/documentation-reefer-events) | Reefer Events 1.0 Beta 1 with Track & Trace 3.0 Beta and IoT Events 1.0 Beta 1 at review | Current reefer/IoT interfaces are visibly labeled beta. | Useful conformance target for experiments only unless a partner contract qualifies the beta; do not present it as stable universal ocean telemetry. |

### Optimization and uncertainty

| Source | Currency at review | Material evidence | Blueprint use |
|---|---|---|---|
| [Google OR-Tools routing overview](https://developers.google.com/optimization/routing) | Current official docs | Vehicle-routing problems can include capacities, time windows, resources, and other constraints; large instances are computationally difficult and solutions may be good without being optimal. | Deterministic optimizer boundary; no model claim of optimality. |
| [OR-Tools vehicle routing with time windows](https://developers.google.com/optimization/routing/vrptw) | Current official docs | Demonstrates explicit time-window constraints and dimensions. | Encode windows in solver/calendar services rather than prose. |
| [OR-Tools routing options and search status](https://developers.google.com/optimization/routing/routing_options) | Current official docs | Exposes success, partial success, failure, timeout, invalid, and infeasible-type outcomes depending on API/status. | Preserve actual solver status and time/solution limits. |
| [OR-Tools CP-SAT solver](https://developers.google.com/optimization/cp/cp_solver) | Current official docs | Returns `OPTIMAL`, `FEASIBLE`, `INFEASIBLE`, `MODEL_INVALID`, or `UNKNOWN`. | `FEASIBLE` is not `OPTIMAL`; `UNKNOWN` is not `INFEASIBLE`. |
| [Gneiting, Balabdaoui, and Raftery, “Probabilistic forecasts, calibration and sharpness”](https://rss.onlinelibrary.wiley.com/doi/abs/10.1111/j.1467-9868.2007.00587.x) | Journal article, 2007 | Probabilistic forecast quality must consider calibration and sharpness. | ETA/demand release gates and distributional evidence. |
| [M5 uncertainty competition results](https://www.sciencedirect.com/science/article/pii/S0169207021001722) | International Journal of Forecasting, 2022 | Large-scale hierarchical retail demand uncertainty/quantile forecasting. | Demand distributions, hierarchy, slice-aware evaluation, not point-only estimates. |
| [M5 accuracy competition findings](https://www.sciencedirect.com/science/article/pii/S0169207021001527) | International Journal of Forecasting, 2022 | Retail demand includes intermittent/zero-heavy series and cross-series complexity. | New/sparse SKU-site behavior and simple baseline comparison. |
| [Forecasting: Principles and Practice — time-series cross-validation](https://otexts.com/fpp3/tscv.html) | Current online third edition at review | Rolling-origin evaluation preserves temporal order. | Reject random train/test splitting for ETA/demand claims. |
| [Forecasting: Principles and Practice — distributional accuracy](https://otexts.com/fpp3/distaccuracy.html) | Current online third edition at review | Quantile forecasts can be scored with quantile/pinball loss and prediction distributions need appropriate scoring. | Forecast evaluation metrics and thresholds. |

### Process, safety, agent risk, and operations

| Source | Currency at review | Material evidence | Blueprint use |
|---|---|---|---|
| [ASCM SCOR Digital Standard process model](https://scor.ascm.org/) | Current SCOR DS site | Defines broad Level 1 processes including Orchestrate, Plan, Order, Source, Transform, Fulfill, and Return. | Domain coverage map and explicit repository boundary where real process ownership overlaps. |
| [SCOR performance introduction](https://scor.ascm.org/performance/introduction) | Current SCOR DS site | Performance attributes include reliability, responsiveness, agility, cost, assets, and sustainability; perfect-order and cycle-time concepts appear within the model. | Business outcome measures, with caution about causal attribution. |
| [SCOR DS Digital Guide](https://www.ascm.org/globalassets/documents--files/corporate-transformation/scor-ds-digital-guide_final.pdf) | Official ASCM guide | Explains the reference model's flexible process and performance structure. | Treat SCOR as a reference model, not an agent authorization model. |
| [ISO 28000:2022](https://www.iso.org/standard/79612.html) | Published international standard | Specifies a security management system relevant to supply-chain security and resilience. Public page is an abstract, not implementation detail. | Security/resilience management context; avoids falsely claiming specific controls from an abstract. |
| [IMO IMDG Code](https://www.imo.org/en/publications/pages/imdg%20code.aspx) and [dangerous goods overview](https://www.imo.org/en/ourwork/safety/pages/dangerousgoods-default.aspx) | 2024 Edition, Amendment 42-24; mandatory from 2026-01-01 | Current maritime dangerous-goods requirements and effective date. | Regulatory data must be current, authoritative, mode/jurisdiction-specific, and outside model memory. |
| [IATA Dangerous Goods Regulations](https://www.iata.org/en/programs/cargo/dgr) | 67th edition effective 2026-01-01 | Current air dangerous-goods edition/effective date stated by IATA. | Same fail-closed constraint and qualified-role boundary for air cargo. |
| [UNECE ADR 2025](https://unece.org/transport/publications/agreement-concerning-international-carriage-dangerous-goods-road-adr-2025) and [UN Model Regulations Rev. 24](https://unece.org/info/publications/pub/407574) | ADR amendments applicable 2025-01-01; Model Regulations Rev. 24 published 2025-09 | Road rules and model regulations are separately maintained; ADR includes technical, training, and safety obligations. | Avoid extrapolating IMDG/IATA rules to road or treating UN model regulations as directly applicable law everywhere. |
| [Anthropic, “Demystifying evals for AI agents”](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) | Published 2026-01-09 | Separates tasks, trials, graders, transcripts/trajectories, outcomes, and environment state; recommends multiple grader types and trials. | Evaluation environment, trajectory/outcome grading, and stochastic reliability reporting. |
| [OWASP Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) | 2026 release | Highlights risks including goal hijacking, tool misuse, identity/privilege abuse, unsafe memory, cascading failures, and rogue behavior. | Threat scenarios and separation of model from credentials/authority. |
| [NIST AI 600-1, Generative AI Profile](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf) | NIST AI RMF profile | Risk management guidance for generative AI across governance, measurement, and management. | Release/evaluation governance and documented residual risk. |
| [NIST AI RMF 1.0](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-1.pdf) and [AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/) | AI RMF 1.0 final; NIST notes revision work is in progress | Trustworthiness includes fairness with harmful bias managed; the Core calls for impact, privacy, fairness/bias, and TEVV evaluation. | Allocation/prioritization harm slices, impact ownership, appeal/override evidence, and continuous risk tracking. |
| [NIST SSDF 1.1](https://csrc.nist.gov/pubs/sp/800/218/final), [SLSA provenance 1.2](https://slsa.dev/spec/v1.2/build-provenance), and [Sigstore verification](https://docs.sigstore.dev/cosign/verifying/verify/) | SSDF 1.1 current final; NIST 1.2 is an initial public draft at review; SLSA spec 1.2 | Secure-development practices plus build provenance, signature and attestation verification mechanisms. | Pin and verify the whole behavior bundle and dependencies; record SBOM/provenance/signatures, but do not equate them with behavioral safety. |
| [Apache Kafka 4.2 design: delivery semantics](https://kafka.apache.org/42/design/design/) and [Amazon SQS delivery/visibility](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html) | Current official docs at review | Kafka idempotence/transactions have defined broker boundaries; SQS standard delivery is at least once and can redeliver during visibility timeout. | Messaging adapter declares ordering/delivery/dedupe boundary; business consumers and external effects remain idempotent and reconciled. |
| [OpenTelemetry GenAI semantic convention registry](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/) | Development/evolving conventions at review | Defines emerging GenAI attributes, with maturity that can change. | Use stable application events and record model metadata without coupling evidence to evolving diagnostic conventions. |

## Findings synthesized across sources

### Standards normalize exchange, not authority

UBL, EPCIS, GS1 keys, UN/EDIFACT, DCSA, and ONE Record materially improve data exchange. None determines, for this application, which system may reserve inventory, which carrier event wins a conflict, whether an approver may spend a given amount, or whether a timeout means a booking exists. EPCIS explicitly separates event and record time and provides correction/capture semantics; that strengthens the case for append-only observations and later read-back. The ONE Record synchronization discussion strengthens the counterpoint: even a modern linked-data API does not remove eventual consistency and idempotency work.

Decision: use standards in adapters and canonical identity/event vocabularies, but require an operating-unit source-of-truth matrix, versioned conflict rules, authorization policy, semantic operation IDs, and downstream reconciliation.

### Provider contracts are product-specific and mutable

DHL exposes separate tracking products with different quotas; FedEx separates authorization, tracking, account-sensitive behavior, and subscription workflows; provider timestamp formats can change. ERP/WMS/TMS systems also distinguish operations that a generic abstraction would erase, such as soft allocation vs reservation or view vs update ACLs.

Decision: each adapter pins provider product, environment, account, upstream contract, auth flow, limits, errors, identity rules, write idempotency, and read-back behavior. The model sees typed domain operations only. A generic `call_api`, `update_shipment`, `update_inventory`, SQL, browser, or credential-selection tool is rejected.

### Capability belongs to an operation, not a connector

Microsoft inventory APIs distinguish reserve/unreserve from allocate/reallocate/consume; X12 distinguishes technical acknowledgement from tender response; customs systems distinguish document receipt from release; mapping services return per-element status and can use fallback computation. A provider label or `read_write` flag loses the semantics needed for authority and recovery.

Decision: manifest every allowed operation with identity, scope, account/region/environment/version, danger tier, preconditions, units/times, partial/batch behavior, quota, idempotency, acknowledgement, read-back, cancellation, privacy class, unsupported behavior, and qualification suite. Unlisted capabilities are denied. Official vendor examples are qualified interface examples, never product endorsements.

### Event history and operational truth are different

EPCIS corrections and timing fields, UBL partial/split document relationships, carrier delays, and WMS/TMS replication behavior all contradict a mutable "latest status" record. A source can record a late event; two sources can disagree legitimately; a missing event is not proof of non-occurrence.

Decision: keep immutable observations, deterministic rebuildable projections, durable exception state, effect state, and diagnostics separate. Preserve event/record/ingest time; source/version; correction links; identity status; and declared/observed/estimated distinctions. Never use "latest timestamp wins" as a universal rule.

### Forecasting requires distributions and applicability

Forecast literature makes point accuracy an incomplete criterion. Calibration, sharpness, temporal validation, hierarchy, intermittency, horizon, and slice behavior matter. A model can write a convincing confidence number without any statistical basis.

Decision: ETA/demand tools return versioned quantiles/distributions with as-of, horizon, cutoff, model/features, evaluation slice, calibration evidence, and applicability. Policies consume thresholds with hysteresis. LLMs cannot invent probabilities, and out-of-distribution or degraded forecasts lower authority.

### Feasibility and explanation need different machinery

Routing and CP-SAT documentation exposes explicit constraint models and result statuses. The solver can return feasible without optimality, unknown on timeout, or infeasible/invalid for different reasons. Language reasoning is valuable for mixed evidence, operator questions, hypotheses, and explanations, but it is not a feasibility proof.

Decision: hard constraints and optimization live in deterministic services. The model proposes declared scenarios/preferences and explains alternatives. It cannot relax hard constraints, change status, assert optimality, or execute the choice.

### Physical effects need distributed-systems semantics

Asynchronous capture, bookings, reservations, tenders, EDI acknowledgements, webhooks, and multi-system plans share one failure: a transport response does not prove the intended business state. A request can be applied and its response lost. Physical movement cannot generally be rolled back.

Decision: every effect uses an intent hash and stable semantic operation ID, exact/versioned approval, an attempt ledger, `unknown` outcome, authoritative read-back, typed postconditions, resource serialization, and compensation or forward recovery. A retry reuses the semantic ID and occurs only after absence is proved or provider semantics make it safe.

### Authority must become narrower under uncertainty

Supply-chain safety/regulation, OWASP agent risks, and provider credential boundaries all argue against prompt-only controls. Current dangerous-goods editions demonstrate why regulatory truth cannot live as stale model memory.

Decision: the default production ceiling is D1. D2 uses isolated preparations/drafts. D3 requires exact approval or a separately governed deterministic runbook. D4 authority changes are proposal-only. Missing identity, stale evidence, model/solver/forecast uncertainty, policy outage, approval expiry, or unknown effect causes abstention, manual handling, or reduced authority.

### Allocation quality includes distributional harm

Shared inventory, appointment, carrier-capacity, and human-attention decisions can systematically impose delays, denials, cost, or cancellation on particular customers, regions, channels, or service classes. Average objective value can improve while starvation or proxy discrimination worsens. NIST AI RMF treats fairness/harm measurement as part of risk management, while logistics topology makes interference between decisions unavoidable.

Decision: use one versioned shared allocator under a declared priority policy; record the eligible population, features, weights, tie-breakers, caps/floors, winners/losers, override and appeal. Evaluate outcomes by relevant groups and counterfactual policies. Operator overrides are reviewed evidence, not preference-training labels.

### Whole-trajectory evaluation is mandatory

Agent evaluation guidance distinguishes tasks, trials, graders, trajectories, outcome state, and environments. Logistics safety needs deterministic external-state oracles that a model judge cannot replace.

Decision: use adapter conformance, historical replay, a discrete-event simulator, adversarial/failure injection, human operations review, shadowing, and canary. Deterministic gates cover authority, identity, units, constraints, duplicate effects, reconciliation, compaction, scope, and privacy. Model graders assess explanation only. Stochastic tasks run multiple trials.

### Context continuity must be loss-aware

Token compaction is unsafe when it turns an estimate into fact, drops an active clock or unknown effect, loses a version pin, or makes an old approval appear current. Memory poisoning guidance reinforces that retained summaries, episodes, and runbooks are persistent attack surfaces rather than benign convenience.

Decision: a compaction receipt records its schema version, source-event high watermark, canonical scope/resources, version pins, approvals, active clocks, pending/unknown effects, invariant hash, omitted artifact references, next safe action, and release/compiler versions. Resume resolves every reference, rehydrates live state, recomputes invariants, and fails closed on gaps. The seven lifetime classes in the guide each have use/reject, deletion, poisoning, and evaluation controls.

## Boundary decision record

### Included work

- post-award purchase/sales/transfer order status and fulfillment dependencies;
- shipment, consignment, leg, logistics-unit, booking, tender, tracking, and handoff state;
- inventory evidence, reservations/allocations, and approved reallocation proposals/effects;
- demand and ETA uncertainty as versioned probabilistic inputs;
- service, capacity, handling, time, cost, safety, customs, and commercial constraints;
- carrier, WMS, ERP/OMS, TMS, EDI, EPCIS, and document evidence integration;
- operational exception detection, investigation, recovery alternatives, approvals, effects, reconciliation, monitoring, and disruption recovery.

### Excluded ownership

- supplier selection, commercial negotiation, supplier award, and procurement authority;
- production scheduling authority, plant equipment/control, and manufacturing quality disposition;
- database schemas, storage engines, CDC/data pipelines, retention platforms, and general DataOps;
- generic invoices, claims, HR/finance cases, document routing, and back-office case management;
- dangerous-goods classification, sanctions/customs/legal judgment, and changes to those policies;
- model/provider/tool/credential/policy/approval authority changes by the runtime agent.

### SCOR overlap resolution

SCOR is broad by design: `Source`, `Transform`, and `Fulfill` can include work owned by procurement, manufacturing, and logistics in different organizations. This repository needs mutually useful engineering blueprints, so it applies a narrower authorization boundary. The logistics agent may consume authoritative supplier or production status and coordinate downstream flow; it hands supplier-award questions to procurement and production/equipment/quality decisions to manufacturing.

## Architecture decision records

### ADR-SCL-001: one durable coordinator by default

**Decision:** one durable workflow per exception aggregate, with typed forecast, solver, policy, adapter, and reconciliation services.

**Why:** long waits, approvals, callbacks, cancellations, and ambiguous external effects require persisted state and deterministic transitions. Multiple LLM agents add cost and coordination failure but do not solve persistence, authority, or reconciliation.

**Rejected:** open-ended multi-agent swarm; transcript as state; synchronous request handler for D3 effects.

**Exception:** a separate agent role may be justified for a distinct legal/security domain with typed handoff and measured benefit. It cannot share write authority over the same resource.

### ADR-SCL-002: authoritative sources plus append-only agent records

**Decision:** ERP/OMS, WMS, TMS, and declared partner systems retain field-specific authority. The agent stores immutable observations, rebuildable projections, workflow state, evidence references, and effect ledger.

**Why:** distributed sources disagree and arrive late; corrections and partial relationships need history. Chat memory cannot be reconciled or audited reliably.

**Rejected:** central LLM memory as the operational twin; universal last-write-wins; overwriting corrected events.

### ADR-SCL-003: query-built context and minimal memory

**Decision:** compile context from scoped current state and immutable artifacts. Use turn, working, run, durable task, governed domain, and optionally curated episodic classes with separate rules. Reject unreviewed long-term preference memory and direct self-learning.

**Why:** old episodes and raw transcripts carry obsolete rates, policies, identity mappings, personal data, and injection. Durable truth must survive restart and compaction independently of model context.

**Rejected:** pass full conversation forever; raw incident vector search as current policy; online prompt/policy/memory mutation.

### ADR-SCL-004: deterministic feasibility, probabilistic risk, semantic explanation

**Decision:** rules/solvers own hard constraints and feasibility; forecast services own calibrated uncertainty; the model owns bounded evidence synthesis, hypotheses, and explanation.

**Why:** the three result classes have different validation requirements. Combining them in one generative output makes uncertainty and constraint violations hard to detect.

**Rejected:** LLM-generated confidence; prompt-only constraints; model assertion that a solver result is optimal.

### ADR-SCL-005: prepare-authorize-commit-verify effects

**Decision:** D3 actions use typed preparation, exact approval, current precondition check, stable semantic operation ID, narrow gateway, and authoritative verification.

**Why:** timeouts, partial success, source-version races, and physical irreversibility are normal. Exact approval must bind what changes.

**Rejected:** approval of a chat summary; new idempotency key per retry; marking success on 2xx/202; generic compensating "rollback."

### ADR-SCL-006: scale by operating cells and shared allocators

**Decision:** isolate by tenant/legal entity/region/business flow, serialize per affected resource, and reserve reconciliation capacity. Broad disruption allocation is centralized in a deterministic shared optimizer/control service.

**Why:** per-shipment autonomy can overbook scarce stock/capacity and cause replan storms. Correlated disruption bursts make human review and reconciliation bottlenecks.

**Rejected:** one global agent with all credentials; independent shipment agents bidding for shared inventory; proposal workers consuming reconciliation capacity.

## Tool acceptance decisions

| Tool family | Decision | Conditions |
|---|---|---|
| ERP/OMS read projection | Accept D1 | Field authority, legal entity, version/freshness, typed errors |
| WMS inventory read | Accept D1 | Item/location/segment/unit semantics and source version preserved |
| Carrier/TMS tracking read | Accept D1 | Account scope, provider declaration label, rate limits, schema version |
| Map/geocode/route matrix | Accept D0/D1 derived read | Canonical access points, mode/region/traffic as-of, per-element/fallback status, limits; never regulatory or commercial feasibility |
| Telematics/sensor read | Accept D1 observation | Device-asset binding, calibration/health, sample/event/ingest times, accuracy, duplicates/retained messages, restricted raw artifact |
| Customs/document status read | Accept D1 | Jurisdiction, filer/account, document/filing revision, channel/guide version, receipt versus release state |
| Forecast ETA/demand | Accept D0 derived tool | Distribution, as-of, horizon, version, calibration, applicability |
| Route/allocation solver | Accept D0 derived tool | Hard constraints, objective/version, limits, explicit status/gap/bound |
| Quote/capacity read | Accept D1 | Quote/capacity expiry and terms; not a booking |
| Prepare reservation/reallocation/tender | Accept D2 | Isolated/expiring; no source mutation unless explicitly classified D3 |
| Commit tender/book/reroute/expedite | Accept D3 | Exact approval/preauthorized runbook, semantic ID, version check, read-back |
| Commit inventory reserve/reallocate | Accept D3 | Exact segments/quantity/unit/source version, serialization, read-back |
| Cancel or external notice | Accept D3 | Exact target/purpose/version, duplicate prevention, recovery semantics |
| Customs filing/document submit, amend, withdraw | Accept D3 only in separately qualified scope | Qualified filer/role, current jurisdiction contract, exact revision, signature/authority, acknowledgement/read-back; otherwise manual |
| Messaging/workflow operations | Accept infrastructure-only | Operation-level ordering/delivery/timer/retry semantics, schema/scope, idempotent consumer; no inference of external effect completion |
| Policy/credential/approval-matrix change | Reject D4 runtime | Separate administrative workflow only |
| Generic HTTP/SQL/browser/shell/write | Reject | Replace with narrow versioned operation or manual path |
| Unreviewed episodic retrieval | Reject | Use governed effective-dated pattern/runbook with scope/provenance |

## Contradictions and caveats

### Stable standard versus active roadmap

DCSA's current documentation lists Track & Trace 2.2 as the published interface, while 3.0/reefer/IoT material is beta. IATA's `2025-07` release is clearly endorsed while the `2026-07` folder describes a proposal pending full endorsement. The correct engineering behavior is provider-specific version negotiation and conformance, not choosing the largest version number or trusting a folder/date label.

### Endorsed API versus synchronization completeness

IATA's endorsed ONE Record release is a strong interoperability baseline, but repository discussion still highlights synchronization/idempotency questions. "Production ready" in a standards release does not prove that an application's distributed state or retry behavior is safe.

### Source-process taxonomy versus repository boundary

SCOR's Source process can extend through scheduling, delivery, and receipt, while this category excludes supplier selection/award. That is not a claim that SCOR is wrong; it is a deliberate repository ownership boundary to stop overlapping agents from receiving the same commercial authority.

### Soft allocation versus physical availability

Inventory platforms can expose virtual allocation pools, soft reservations, and physical inventory separately. A cross-system term such as `available` has no safe universal formula. The operating unit must declare which platform field and freshness rule supports which proposal/effect.

### Webhook or asynchronous acceptance versus truth

A webhook is evidence delivered by a source; an asynchronous `202` can be only job acceptance. Neither proves the agent's desired postcondition without provider-specific status/read-back. Push improves latency but does not remove polling/reconciliation.

### Business improvement versus agent reliability

SCOR-aligned outcomes such as perfect orders, responsiveness, agility, cost, and assets matter. They are affected by demand, weather, labor, suppliers, carriers, inventory policy, and customer mix. Evaluate agent authorization, duplicates, grounding, feasibility, latency, and reconciliation directly, then estimate business impact with a defensible causal design.

## Full architecture evidence chain

The blueprint's promise-at-risk slice maps every critical decision to evidence:

| Architecture link | Research basis | Control derived |
|---|---|---|
| Carrier/TMS observations to projection | EPCIS timing/corrections; carrier-specific API contracts | Immutable observations; event/record/ingest time; source/version; adapter conformance |
| Projection to risk detector | Forecast calibration and rolling-origin sources | Versioned probabilistic ETA; applicability; hysteresis; no point-as-fact |
| Risk to alternatives | SCOR process/performance context; model eval guidance | Bounded model hypothesis and explanation with task/trajectory evidence |
| Alternative to feasible plan | OR-Tools routing/CP-SAT statuses | Deterministic constraint/optimization service; status and optimality explicit |
| Plan to authorization | OWASP/NIST risk framing; vendor view/update controls | Model has no credentials; exact approval and current policy outside prompt |
| Authorization to external effect | Provider async/API behavior; inventory operation distinctions | Typed prepare/commit, semantic ID, scoped connector, attempt ledger |
| Receipt to verified outcome | EPCIS capture behavior and ONE Record synchronization caveat | Authoritative read-back, unknown outcome, no blind retry, forward recovery |
| Adapter label to allowed operation | Microsoft inventory operations; X12 acknowledgement layers; Maps per-element status; CBP channel separation | Operation-level capability manifest, denial by default, account/region/version qualification suite |
| Shared scarcity to allocation | Network interference plus NIST AI RMF fairness/impact requirements | One shared allocator, declared priority/tie-breaker, winner/loser evidence, harm slices and appeal |
| Compacted context to safe resume | Durable state requirements plus OWASP memory/context poisoning risk | Receipt version, high watermark, pins, clocks, effects, omitted refs, invariant recheck and fail-closed rebuild |
| Release to executable bundle | NIST SSDF, SLSA provenance and signature/attestation mechanisms | SBOM/digests/provenance/signatures, immutable behavior bundle, whole-bundle canary and coherent rollback |
| Verified outcome to rollout learning | Agent eval guidance and privacy/security controls | Curated offline feedback, release manifest, shadow/canary, no self-modification |

## Stage evidence requirements

| Stage | Research-dependent questions | Required artifacts |
|---|---|---|
| 0 Contract | What is the current standard/provider/regulatory contract? Who owns each process and source field? | Source register, boundary matrix, charters, identity/source/constraint maps |
| 1 Observe | Can real duplicates, corrections, partial relationships, timezones, and schema drift be normalized? | Adapter fixtures, replay report, projection comparison, freshness SLO |
| 2 Explain | Can untrusted logistics content be summarized without instruction following or scope leak? | Injection/privacy suite, grounding/abstention results, context/compaction record |
| 3 Propose | Are forecast slices calibrated and solver semantics preserved? Are alternatives useful? | Rolling-origin report, solver verification, human review, cost/latency report |
| 4 Approve | Does approval bind the current exact intent and invalidate on material change? | Policy tests, approval race tests, role/expiry audit |
| 5 Act | Can all writes be deduplicated, searched, verified, cancelled, and recovered? | Post-commit timeout, partial effect, saga, reconciliation, kill-switch drills |
| 6 Scale | Does the system survive correlated disruption, quotas, human bottlenecks, and provider/regulatory evolution? | Load/capacity, DR, disruption simulation, canary, incident exercise, refresh owners |

## Known limitations after Pass 2

- The blueprint is vendor-neutral and does not contain a conformance profile for every ERP, WMS, TMS, carrier, EDI network, or regional regulation. Each operating unit must add one.
- Public API documentation may omit contractual enterprise limits, eventual-consistency windows, or account-specific behavior. Validate in sandbox and contract tests with the provider.
- ISO 28000's public page is only an abstract; this packet does not claim detailed clause compliance.
- Dangerous-goods sources cited are maritime and air examples. Road, rail, inland waterway, postal, customs, sanctions, trade, labor, and local privacy rules require jurisdiction-specific counsel and current authoritative sources.
- Forecast methodology depends heavily on available labels and the target. The guide defines a contract and evaluation posture, not one universal ETA/demand model.
- Optimization models are use-case-specific. The guide defines ownership/status/invariant boundaries, not a universal network objective.
- Exact business SLO and approval thresholds require baseline measurements; illustrative numbers in the operations/evaluation guides are not defaults to copy blindly.
- No provider documentation proves that a UI or API operation is reversible. Each effect needs an observed read-back and recovery contract.
- The operation-family manifests and tests are normative templates, not completed conformance reports. A deployment must replace each example with observed sandbox/production evidence for its exact account, region, contract, partner guide, extensions, and certification.
- The last clearly endorsed ONE Record tag found during review is `2025-07`; the repository's `2026-07` proposal must be rechecked for final endorsement and partner adoption before use.
- Mapping/routing APIs do not supply a global truck, dangerous-goods, customs, port, contract, or capacity graph. Those datasets and legal rights remain operating-unit dependencies.
- Fairness slices and priority constraints depend on the organization's legitimate service obligations and applicable law. The blueprint requires explicit policy and evaluation but does not select protected groups or business priorities for a deployment.
- Simulator trial counts and illustrative SLOs are evidence floors or examples, not statistical power calculations. Teams must size trials and thresholds from risk, base rates, correlation, and required confidence.

## Refresh plan

| Trigger | Check | Owner |
|---|---|---|
| Quarterly, or provider notice | Carrier API product/version, auth, quotas, schemas, webhook, idempotency/read-back | Integration owner |
| ERP/WMS/TMS release | Operation semantics, ACLs, deprecated resources, version/concurrency behavior | Application owner |
| GS1/OASIS/UNECE/DCSA/IATA release | Endorsed vs draft version, vocabulary/schema migration, implementation status | Data architecture owner |
| Regulatory effective-date notice | Mode/jurisdiction dangerous goods, customs, sanctions, privacy/residency | Qualified compliance owner |
| Forecast data/model release | Rolling-origin metrics, calibration, drift, slice applicability, baseline | Forecast owner |
| Solver/model/objective release | Constraint regression, status handling, runtime/gap, numerical behavior | Optimization owner |
| Model/prompt/context/tool change | Injection, grounding, trajectory, stochastic safety and cost suite | Agent platform owner |
| Incident or SLO breach | Assumptions, postconditions, retry/reconciliation, capacity, runbook | Incident owner plus domain owner |
| New lane/site/mode/item class/tenant/legal entity | Identity, source authority, constraints, privacy, capacity, eval slice | Operating-unit owner |

Refresh records should update the research date and identify which blueprint decision or release gate changed. Do not rewrite operational history when a standard evolves; version the adapter and migrate explicitly.

## Research-to-guide traceability

| Blueprint topic | Primary guide |
|---|---|
| Category boundary, authority, stage gates | [Mission, boundaries, and workload fit](../../agents/supply-chain-logistics-agent/01-mission-boundaries-and-workload-fit.md) |
| Durable coordinator, runtime, concurrency, multi-agent rejection | [Reference architecture and runtime selection](../../agents/supply-chain-logistics-agent/02-reference-architecture-and-runtime-selection.md) |
| Standard identities, observations, corrections, projections, approvals/effects | [Identities, state, events, and projections](../../agents/supply-chain-logistics-agent/03-identities-state-events-and-projections.md) |
| Probabilistic forecasts, solvers, planning, context, compaction, memory | [Uncertainty, constraints, planning, and context](../../agents/supply-chain-logistics-agent/04-uncertainty-constraints-planning-and-context.md) |
| Standards and vendor adapters, tools, auth, injection, privacy | [Integrations, tools, security, and privacy](../../agents/supply-chain-logistics-agent/05-integrations-tools-security-and-privacy.md) |
| Idempotency, exact approval, unknown outcomes, reconciliation, sagas, disruptions | [Effects, approvals, reconciliation, and recovery](../../agents/supply-chain-logistics-agent/06-effects-approvals-reconciliation-and-recovery.md) |
| Simulator, graders, failures, hard gates, observability, SLOs | [Observability, evaluation, and failure injection](../../agents/supply-chain-logistics-agent/07-observability-evaluation-and-failure-injection.md) |
| Capacity, cost, cells, degradation, DR, release, incidents, governed evolution | [Deployment, scaling, incidents, and governed evolution](../../agents/supply-chain-logistics-agent/08-deployment-scaling-incidents-and-evolution.md) |

## Research quality checklist

- [x] Current primary standards and official provider/vendor documentation were reviewed.
- [x] Stable/endorsed versions were separated from roadmaps and working drafts.
- [x] Important claims were cross-checked across standards, APIs, optimization, forecasting, and agent-risk sources.
- [x] Procurement, manufacturing, DataOps, back-office, safety, and compliance boundaries are explicit.
- [x] Provider/standards capabilities were not treated as proof of business authority or effect completion.
- [x] Uncertainty, constraint, idempotency, reconciliation, security, privacy, evaluation, operations, and evolution decisions are traceable.
- [x] Rejected designs and contradictions are recorded rather than hidden.
- [x] Known limitations and refresh triggers have responsible roles.
- [x] Operation-level manifests cover ERP/OMS, WMS/inventory, TMS, carriers/freight, EDI, maps/routing, ports/customs/documents, telematics/IoT, messaging, and workflow boundaries.
- [x] Endorsed/stable releases are separated from proposals, beta interfaces, roadmaps, drafts, and partner-specific contracts.
- [x] Human factors, allocation harms, temporal leakage, counterfactual baselines, context continuity, software dependencies, and whole-bundle rollback are covered.

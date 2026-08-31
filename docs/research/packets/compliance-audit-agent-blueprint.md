# Research Packet: Compliance Audit and Control Evidence Agent Blueprint

> **Research date:** 2026-08-31  
> **Research cutoff:** Sources available and reviewed through 2026-08-31 (Asia/Calcutta)  
> **Status:** Pass 2 research synthesis for a production blueprint; not legal advice, an audit methodology, a certification scheme, or an assurance opinion  
> **Output:** [Production Compliance Audit and Control Evidence Agent Blueprint](../../agents/compliance-audit-agent/README.md)

## Research questions

1. Which compliance-evidence activities can be automated without delegating legal applicability, professional judgment, control operation, independence, or attestation?
2. How should criteria, organization controls, procedures, mappings, evidence, observations, decisions, exceptions, effects, and packages be represented?
3. What does defensible evidence lineage require beyond storing a file or hash?
4. How can requests and connectors handle pagination, retention, late events, schema drift, partial results, and source-native “compliance” labels?
5. How should population definition, sampling, TOD, TOE, contradiction handling, and missing selected evidence work?
6. What identity and segregation-of-duties controls preserve reviewer independence across people, services, and model-assisted steps?
7. Which context and memory classes are appropriate for months-long engagements, and what must survive lossy compaction?
8. How should idempotency, unknown outcomes, reconciliation, package freeze/delivery, and external effects work?
9. What privacy, security, licensing, retention, legal-hold, tenant-isolation, and provider controls are required?
10. Which evaluations, fault injections, SLOs, deployment gates, incident runbooks, unit economics, and upgrade controls are needed from stage 0 through multi-tenant scale?

## Category decision and boundary

This is a distinct category in the repository’s 50-category registry because its primary object is a **versioned assurance engagement and evidence graph**, not a generic back-office case, regulatory-intelligence feed, IAM operator, security investigator, or control implementation service.

The blueprint owns:

- control mapping after qualified approval;
- evidence-request orchestration and source collection;
- immutable evidence versions, provenance, completeness limitations, and transformations;
- approved population freeze and deterministic sample execution;
- structured TOD/TOE workpaper support;
- reviewer, contradiction, exception, remediation-follow-up, and package state;
- deterministic audit-package generation and bounded delivery;
- operational/evaluation evidence for the agent system itself.

It does not:

- define or interpret legal obligations;
- decide applicability, materiality, risk appetite, sample methodology, sufficiency, effectiveness, or report language without qualified human authority;
- implement, configure, operate, or remediate the assessed control;
- grant its own access or expand connector scope;
- permit a preparer/model/control operator to approve its own evidence where independence policy forbids it;
- claim an organization is compliant, certified, effective, audit-ready, or free of exceptions;
- issue an audit opinion, attestation, certification, management representation, or regulator communication.

These boundaries resolve overlap with regulatory-intelligence agents (changing obligations), IAM/infrastructure/DevOps agents (control operation), security-investigation agents (incident facts), and generic document agents (parsing). This agent may request/read their authoritative outputs but cannot inherit their authority or conclusions.

## Research method

Research proceeded in four passes:

1. **Repository alignment:** inspected the category registry, expansion program, closest production blueprints, and canonical runtime/state, tools/provenance, context/memory/compaction, planning, security, reliability/effects, evaluation/observability, and operations guides.
2. **Assurance/control foundation:** reviewed current official NIST, GAO, PCAOB, IIA, IAASB, ISO, AICPA, PCI SSC, FedRAMP, GDPR/EDPB, W3C, and RFC material relevant to control assessment, evidence, documentation, independence, privacy, provenance, mapping, and machine-readable artifacts.
3. **Connector reality:** reviewed official AWS, Azure, Google Cloud, GitHub, Okta, ServiceNow, and Atlassian documentation for evidence types, native states, APIs, retention, pagination/order, request workflow, and webhook behavior.
4. **Synthesis and contradiction testing:** compared source scopes/effective dates, separated final from proposed material, tested claims against the workload boundary, and derived implementable contracts, stage gates, failure injections, and refresh triggers.

Primary and official sources were preferred. Vendor documentation was used for its own connector semantics, not as independent assurance guidance. Framework/standards language was paraphrased and linked rather than reproduced. Where a source is licensed, the blueprint records rights as a runtime concern.

## Evidence tiers

| Tier | Sources | How used |
| --- | --- | --- |
| **A — Authoritative primary** | Official standards/publications, laws, regulators, standards bodies, RFCs, official repositories/releases | Baseline concepts, versions, effective/maturity status, required cautions |
| **B — Official product/program documentation** | Cloud/SaaS API and connector documentation, official program pages | Source semantics, limits, retention, state, integration behavior |
| **C — Foundational research** | Peer-reviewed or widely cited original agent/context papers | Explain why model reasoning/context are useful but insufficient as controls |
| **D — Repository synthesis** | Canonical guides and adjacent production blueprints | Reuse established state/effect/context/security/evaluation contracts |

No source was treated as universal authority outside its stated scope. Vendor evidence labels were not treated as independent compliance conclusions. Foundational LLM papers were not treated as production assurance evidence.

## Dated standards and program baseline

| Source or program | Baseline at research date | Maturity/effective status | Blueprint use and limitation |
| --- | --- | --- | --- |
| [NIST SP 800-53 Rev. 5 Update 1](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final) | Revision 5 with update materials | Final | Control catalog input; adopting program still determines baseline/applicability |
| [NIST SP 800-53A Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/a/r5/final) | Rev. 5; page records Release 5.2.0 planning note dated 2025-08-27 | Final, customizable | Examine/interview/test procedures and OSCAL availability; not an automatic audit methodology for every engagement |
| [OSCAL 1.2.3](https://github.com/usnistgov/OSCAL/releases/tag/v1.2.3) | 1.2.3 released 2026-08-07 | Current upstream patch release | Machine-readable models/mapping; consumers may pin an earlier supported version |
| [OSCAL control mapping model](https://pages.nist.gov/OSCAL/learn/concepts/layer/control/mapping/) | Mapping relationships introduced in 1.2 line | Current official reference | Explicit set relationships/rationale/confidence/status; schema-valid does not mean substantively correct |
| [NISTIR 8477](https://www.nist.gov/publications/mapping-relationships-between-documentary-standards-regulations-frameworks-and) | Documentary mapping concepts | Final | Supports relationship-aware crosswalks; not equivalence proof |
| [NIST SP 1347](https://csrc.nist.gov/pubs/sp/1347/final) | Informative references publication dated 2026-08-25 | Final | Human/machine-readable cross-reference context; informative maps do not transfer compliance |
| [GAO Green Book](https://www.gao.gov/greenbook) | 2025 edition, effective FY2026 | Effective | U.S. federal internal-control context; other entities may adopt, but it is not universal law |
| [GAO FISCAM 2024](https://www.gao.gov/products/gao-24-107026) | 2024 revision, effective for engagements beginning 2024-10-01 | Effective | U.S. federal financial-information-system control methodology context |
| [GAO Yellow Book](https://www.gao.gov/yellowbook) | 2024 revision, effective for engagements beginning on/after 2025-12-15 | Effective | Government-audit quality, independence, objectivity context |
| [PCAOB AS 2201](https://pcaobus.org/oversight/standards/auditing-standards/details/AS2201) | Current AS 2201 | Effective for applicable PCAOB engagements | TOD/TOE concepts for integrated ICFR audits; not universal compliance procedure |
| [PCAOB AS 1105](https://pcaobus.org/oversight/standards/auditing-standards/details/AS1105) | Current audit-evidence standard | Effective for applicable PCAOB audits | Sufficiency/appropriateness and contradictory evidence concepts; jurisdiction/scope limited |
| [PCAOB AS 1215](https://pcaobus.org/oversight/standards/auditing-standards/details/AS1215) | Current documentation standard | Effective for applicable PCAOB audits | Reconstruction, source/purpose/conclusion, performer/reviewer/date quality lens |
| [PCAOB AS 2315 future version](https://pcaobus.org/oversight/standards/auditing-standards/details/as-2315--audit-sampling-%28effective-on-12-15-2026%29) | Revised sampling standard displayed with future date | Effective 2026-12-15, not yet effective at cutoff | Profile registry must distinguish future-effective from current methodology |
| [PCAOB AS 2301](https://pcaobus.org/oversight/standards/auditing-standards/details/AS2301) | Current risk-response standard | Effective for applicable audits | Risk/unpredictability context; does not authorize agent sample design |
| [IIA Global Internal Audit Standards](https://www.theiia.org/en/standards/) | 2024 standards effective 2025-01-09 | Effective | Internal-audit objectivity/independence context; not interchangeable with external-audit rules |
| [IESBA 2025 Handbook](https://www.ethicsboard.org/news-events/2025-10/now-available-iesba-handbook-2025-edition) | 2025 International Code of Ethics handbook | Current handbook where adopted/applicable | Professional ethics and independence context; applicability depends on engagement, profession, and jurisdiction |
| [IAASB 2025 Handbook](https://www.iaasb.org/publications/2025-handbook-international-quality-management-auditing-review-other-assurance-and-related-services) | 2025 edition | Current handbook | International assurance baseline where adopted; jurisdiction/adoption varies |
| [IAASB ISA 330/500/520 proposals](https://www.iaasb.org/publications/proposed-revisions-audit-evidence-risk-response-isa-330-isa-500-isa-520) | Exposure drafts issued 2026-08-05; comments due 2026-12-15 | Proposed, not effective | Impact research only; must not be silently activated |
| [ISO 19011:2026](https://www.iso.org/standard/19011) | Edition 4 published 2026-05 | Current guidance | Management-system audit-program guidance; not certification; licensed/copyright usage restrictions apply |
| [AICPA Trust Services Criteria with revised points of focus 2022](https://www.aicpa-cima.com/resources/download/2017-trust-services-criteria-with-revised-points-of-focus-2022) | 2017 criteria with 2022 points-of-focus revision | Current linked AICPA material at cutoff | SOC 2 criteria context; authoritative guide/reporting materials and engagement judgment may be licensed |
| [PCI DSS document library](https://www.pcisecuritystandards.org/document_library/?class=pcidss&doc=pci_dss) | PCI DSS v4.0.1 and current reporting templates/guidance | Current at cutoff | Version-pinned PCI profile; assessor qualifications/customized/compensating paths remain program-specific |
| [PCI DSS next-iteration RFC](https://blog.pcisecuritystandards.org/request-for-comments-pci-data-security-standard-pci-dss-v4.0.1) | 2026 request for comments on next iteration | Proposed planning, not replacement | Refresh trigger, not active requirement |
| [FedRAMP Consolidated Rules / KSI reference](https://preview.fedramp.gov/2026/reference/20x/b/key-security-indicators/) | 2026 current program reference for 20x path | Program-specific current reference | Profile overlay; continuous/automated validation does not create a general audit conclusion |
| [FedRAMP RFC-0024](https://www.fedramp.gov/rfcs/0024/) | Machine-readable authorization package proposal | RFC/proposed unless adopted | Research/compatibility planning only |
| [GDPR official text, Article 5](https://eur-lex.europa.eu/legal-content/EN/TXT/?toc=OJ%3AL%3A2016%3A119+%3ATOC&uri=uriserv%3AOJ.L_.2016.119.01.0001.01.ENG) | Consolidated official regulation source | Applicable under its legal scope | Purpose/minimization/storage/integrity/accountability principles; legal applicability requires counsel/privacy owner |
| [NIST Privacy Framework 1.0](https://csrc.nist.gov/pubs/cswp/10/nist-privacy-framework-version-10/final) | Version 1.0 | Final voluntary framework | Privacy-risk structure; not legal compliance |
| [NIST Privacy Framework 1.1 project](https://www.nist.gov/privacy-framework/new-projects/privacy-framework-version-11) | Initial public draft/update project at cutoff | Draft, not final | Refresh trigger only |

The baseline is intentionally explicit because “current” and “effective” differ. Each runtime profile records publication version, effective range, maturity, jurisdiction/use, rights, consumer compatibility, owner, and digest.

## Finding 1: deterministic workflow is the product; the model is optional

Evidence requests, timers, assignments, authorization, source queries, population freeze, sampling, review state, exceptions, manifests, and effects are normal software problems. A model is justified only for semantic ambiguity: mapping proposals, unstructured extraction, comparison, contradiction search, question drafting, and narrative preparation.

Foundational work such as [ReAct](https://arxiv.org/abs/2210.03629) and [Toolformer](https://arxiv.org/abs/2302.04761) shows useful reasoning/tool-interaction patterns; it does not provide durable state, authorization, idempotency, evidence sufficiency, independence, or attestation. The architecture therefore uses a bounded model worker inside a durable coordinator, consistent with the repository’s [execution boundaries](../../runtime/execution-boundaries.md).

## Finding 2: criteria, controls, procedures, and mappings must remain distinct

A published criterion is not the organization’s actual control. A control description is not evidence of operation. An assessment procedure is not the criterion itself. A mapping is not an implementation or conclusion.

The [OSCAL layer model](https://pages.nist.gov/OSCAL/learn/concepts/layer/) and [assessment layer](https://pages.nist.gov/OSCAL/learn/concepts/layer/assessment/) support distinct machine-readable artifacts. The blueprint goes further by preserving organization-control and reviewer-decision ownership rather than collapsing all objects into a universal control table.

## Finding 3: mappings require relationships, direction, elements, conditions, and review

NIST’s OSCAL mapping model and NISTIR 8477 support relationship-aware maps. This contradicts the common agent pattern “embedding similarity above threshold = mapped.” Similarity may retrieve candidates, but a qualified owner must approve exact versions, direction, covered/excluded elements, conditions, rationale, confidence/status, validity dates, and intended use.

The strongest safety rule is that a mapping never transfers an evidence result or compliance conclusion automatically.

## Finding 4: source and profile versions must be pinned, not resolved to latest

Upstream OSCAL was 1.2.3 at the cutoff, while a consuming program can still expose 1.2.2 examples or requirements. Audit/assurance standards can have current and future-effective versions simultaneously. IAASB exposure drafts and FedRAMP RFCs are current research but not effective requirements.

Therefore active engagements pin:

- source publication/effective version;
- local profile/mapping/procedure bundle and digest;
- consumer schema compatibility;
- complete runtime release manifest.

Migration requires impact review; old versions remain reconstructable.

## Finding 5: licensed standards are a data-rights problem, not merely retrieval content

ISO and AICPA materials illustrate that official content can be copyrighted/licensed. Access to read a standard does not automatically authorize copying it into prompts, embeddings, evaluation sets, logs, generated guides, or customer packages.

The profile object therefore includes rights metadata and allowed operations. Restricted content remains out of model/provider paths unless verified terms permit the exact use. The runtime returns `source_unavailable_or_unlicensed` instead of filling missing text from model memory.

## Finding 6: evidence presence is not sufficiency or appropriateness

[PCAOB AS 1105](https://pcaobus.org/oversight/standards/auditing-standards/details/AS1105), within its applicable scope, emphasizes quantity and quality, source/reliability, relevance, and contradictory evidence. AWS Audit Manager likewise documents automated/manual evidence and indicates automated evidence can show full or partial compliance rather than an independent conclusion.

The blueprint separates:

- artifact identity/integrity;
- source reliability and controls over produced information;
- query/population completeness;
- relevance to procedure;
- candidate observation;
- reviewer acceptance and conclusion.

An artifact can be authentic but irrelevant, complete for the wrong query, stale, contradictory, or insufficient.

## Finding 7: TOD and TOE support require human-owned methodology

PCAOB AS 2201 supplies specific ICFR design/operating-effectiveness concepts; NIST SP 800-53A supplies customizable examine/interview/test procedures for its ecosystem. They are useful but not interchangeable.

The agent may assemble inputs and execute approved steps. Qualified humans own:

- applicable methodology and objective;
- design/effectiveness judgment;
- source reliability/sufficiency;
- risk/materiality/deviation classification;
- conclusion and report impact.

Profile overlays hold the engagement-specific vocabulary and approvals.

## Finding 8: population validity precedes sample reproducibility

Sampling is not “pick some examples.” A defensible workflow freezes the population definition, source query, period, reconciliation, canonical record IDs/order, digest, method, sample size, strata, seed policy, replacement rule, and approving person before selection.

PCAOB sampling material is audit-context-specific, but it reinforces that statistical and nonstatistical approaches still require professional judgment. The deterministic service executes; the model never sets the method, assurance parameter, or sample size.

Missing selected evidence is not automatically replaced. It remains a visible state until an authorized methodology decision is recorded.

## Finding 9: provenance must include acquisition and transformation, not only a hash

[W3C PROV](https://www.w3.org/TR/prov-overview/) provides a durable conceptual foundation: entities, activities, and agents. A production evidence manifest needs exact bytes/records, native source identity/version, observation/acquisition times, query, pagination, principal/delegation, adapter version, completeness/truncation, classification, retention, digest, and derived-transform lineage.

Hashing JSON also requires a stable serialization; [RFC 8785](https://www.rfc-editor.org/rfc/rfc8785) is a relevant canonicalization option. [RFC 3161](https://www.rfc-editor.org/info/rfc3161/) timestamps and [RFC 4998](https://www.rfc-editor.org/info/rfc4998/) evidence records can add long-term integrity mechanisms for selected uses, but they are not an MVP requirement and do not prove source truth.

## Finding 10: immutability proves a narrower claim than many designs assume

AWS, Azure, and Google object-lock mechanisms can constrain later mutation/deletion under specific modes and permissions. [NISTIR 8387](https://www.nist.gov/publications/digital-evidence-preservation-considerations-evidence-handlers) provides useful preservation considerations.

None proves that:

- the source was truthful or unaltered before capture;
- the approved query addressed the complete population;
- the evidence is relevant/sufficient;
- the control operated effectively;
- an organization is compliant.

The documentation therefore uses “immutable evidence version” and records the mechanism/limitations, not “tamper-proof truth.”

## Finding 11: connectors expose material and changing evidence limits

Official connector research produced concrete design constraints:

| Source | Official documentation | Material implication |
| --- | --- | --- |
| AWS Audit Manager | [Concepts](https://docs.aws.amazon.com/audit-manager/latest/userguide/concepts.html), [collection](https://docs.aws.amazon.com/audit-manager/latest/userguide/how-evidence-is-collected.html), [review](https://docs.aws.amazon.com/audit-manager/latest/userguide/review-evidence.html), [Evidence API](https://docs.aws.amazon.com/audit-manager/latest/APIReference/API_Evidence.html) | Preserve automated/manual type, source collection semantics, possible inconclusive/partial status, and documented evidence lag |
| Azure Policy | [Compliance states](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/compliance-states), [Policy States API](https://learn.microsoft.com/en-us/rest/api/policyinsights/policy-states) | Keep native level/state/evaluation time; error/conflicting/exempt/unknown must not collapse to pass/fail |
| Azure attestations | [Attestations API](https://learn.microsoft.com/en-us/rest/api/policyinsights/attestations/get-at-resource-group?view=rest-policyinsights-2024-10-01) | Self-attestation is attributed source evidence, not independent approval |
| Google Cloud Asset Inventory | [Asset history](https://docs.cloud.google.com/asset-inventory/docs/get-asset-history) | Documented history window/scope limits require prompt checkpointing and explicit gaps |
| GitHub | [Organization audit log](https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/reviewing-the-audit-log-for-your-organization) | Documented retention/export limits require collection before expiry, watermarks, and stream/query reconciliation |
| Okta | [System Log API](https://developer.okta.com/docs/reference/system-log-query/) | Cursor-driven polling, published-order behavior, and documented retention require overlap, dedupe, watermarks, and early collection |
| ServiceNow | [Request evidence](https://www.servicenow.com/docs/r/governance-risk-compliance/audit-management/request-evidence.html), [workflow](https://www.servicenow.com/docs/r/governance-risk-compliance/audit-management/evidence-request-workflow.html) | Preserve native requester/assignee/state/due/confidentiality and attachment identities rather than inventing a second truth |
| Jira | [REST v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro), [webhooks](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-webhooks/) | Webhook lifecycle and authoritative polling/reconciliation belong in the adapter contract |

Connector assumptions belong in executable compatibility tests and refresh alerts. “API returned 200” is not a completeness test.

## Finding 12: vendor compliance state is evidence, not the agent’s conclusion

Azure Policy’s nuanced native states and AWS Audit Manager’s evidence/status concepts are valuable source facts. Flattening them to `passed` loses error, conflicting, exemption, unknown, aggregation level, source evaluation time, and partial-coverage semantics.

The artifact stores the raw source state, query/scope, evaluation time, and limitations. A separate reviewer decides how it affects the procedure.

## Finding 13: continuous evidence is not a continuous opinion

FedRAMP’s evolving automation and persistent-review direction, cloud event streams, and GRC continuous evidence can improve freshness. They also introduce schema changes, late events, changing control designs, source retention, correlated repeated observations, unresolved alert backlogs, and reviewer-capacity constraints.

The blueprint creates versioned checkpoints and control change points. It never emits a perpetual green compliance state or silently treats an RFC/pilot as an adopted methodology.

## Finding 14: reviewer independence requires real-person and organizational relationships

IIA, GAO, PCAOB, and professional ethics sources use different independence/objectivity regimes, but all make a simple architecture point: two software identities do not establish independence.

The policy service evaluates real-person identity, organization/reporting relationships, control implementation/operation, preparation/submission, conflicts, engagement roles, and the applicable policy at assignment and decision time. Humans remain responsible for disclosures and relationships the system cannot observe.

An LLM “critic” can find errors but is not an independent reviewer or qualified signer.

## Finding 15: documentation must let a qualified reviewer reconstruct the work

PCAOB AS 1215 is jurisdiction-specific, but its reconstruction lens is broadly useful. The package records purpose/procedure, source and exact versions, preparer/model proposal, reviewer/decision/rationale, dates, contradictions/limitations, later changes, package manifest, and delivery receipt.

A transcript or sampled trace cannot provide this guarantee. Packages are deterministic from immutable accepted records; model-generated narratives are accepted and stored before rendering.

## Finding 16: evidence, observations, decisions, effects, and telemetry must be different records

This separation is foundational:

- **evidence:** attributed source material and provenance;
- **observation:** procedure-specific statement about evidence, initially a proposal;
- **decision:** authorized human disposition bound to exact inputs;
- **side effect:** external request/notification/freeze/delivery with receipt/reconciliation;
- **telemetry:** diagnostic behavior, sampled/redacted under a different policy.

Collapsing them creates common failures: model text becomes truth, HTTP status becomes business state, vendor status becomes compliance, trace loss removes audit history, or a changed file inherits old approval.

## Finding 17: long-running engagement truth belongs in typed state, not memory

Requests, evidence, samples, review, exceptions, holds, and packages can span months. The durable coordinator owns state and timers. Model context is reconstructed for one task from versioned references.

[Lost in the Middle](https://arxiv.org/abs/2307.03172) provides foundational evidence that long contexts do not guarantee reliable use of all information. That reinforces, but does not by itself establish, the blueprint’s typed context lanes and lossy-compaction preservation contract.

The runtime names exactly seven lifetimes: turn/scratch, working/run, session, durable engagement/task, domain, long-term, and episodic/outcome. Turn/working/session are bounded and reconstructable; durable task and signed/versioned domain services hold authoritative facts; long-term and episodic influence are disabled for production decisions. Retrieval indexes/caches are projections—not an eighth lifetime—and are tenant/purpose/version partitioned and rebuildable.

## Finding 18: effects require semantic identity and unknown-state reconciliation

Creating requests, reminders, cancellation, package freeze, delivery, correction, or supplement are external effects. Retries use the same semantic operation ID; changed intent uses a new one. A timeout after dispatch is `unknown`, not failure.

Commit-time authorization revalidates exact payload/manifest, actor, tenant, engagement state, destination, approval scope/expiry, independence, policy, incident mode, and budget. Reconciliation queries an authoritative downstream source before retry.

This follows the repository’s [idempotency and side-effects](../../reliability/idempotency-and-side-effects.md) contract and prevents duplicate confidential disclosure.

## Finding 19: privacy, retention, and legal hold require purpose-specific data lifecycle

GDPR principles and the NIST Privacy Framework support minimization/accountability thinking, but applicable legal duties vary. “Keep everything for audit” is unsafe: raw/derived evidence, prompts, indexes, traces, packages, and backups can have different purpose and retention.

Legal hold prevents deletion; it does not grant access or model use. Object lock must be reconciled with deletion duties. A verified delete traverses derived artifacts, indexes, caches, exports, and backup expiry while preserving a content-free tombstone/verification record.

## Finding 20: the evidence aggregation point has a severe security boundary

Read-only access can still expose the organization’s identity graph, security configuration, incidents, source code, personal data, legal documents, and control weaknesses. Evidence is adversarial/untrusted content and can carry indirect prompt injection, secrets, malware, active content, or exfiltration instructions.

The model has no standing connector/effect credential. Deterministic code constructs tool arguments from approved records; content cannot create capabilities. Tenant, purpose, data class, profile/release, and grant bind storage, indexes, caches, queues, model routes, telemetry, and package destinations.

## Finding 21: evaluation must measure the full control trajectory

Model accuracy cannot reveal wrong source accounts, silent pagination gaps, illegal transitions, stale approvals, self-review, sample replacement, duplicate delivery, legal-hold deletion, or reviewer overload.

The evaluation ladder therefore includes schema/unit, connector/component, end-to-end workflow, security/privacy, replay/shadow, canary, continuous monitoring, and incident regressions. Hard controls are zero-tolerance release gates and are not averaged into a quality score.

Metrics include mapping type/direction, citations/contradictions/abstention, lineage/completeness, population/sample reproducibility, reviewer agreement/burden, effect reconciliation, package integrity, evidence/review freshness, cost, and manual fallback.

## Finding 22: operational bottlenecks are often sources and people, not inference

Source rate limits/retention windows, document processing, immutable storage, qualified independent reviewers, exception queues, package deadlines, and reconciliation capacity can dominate. Unlimited queues and aggressive retries worsen risk.

The operating design reserves capacity for security containment, unknown-effect reconciliation, retention-window collection, independent review, and exception deadlines. Optional model/evaluation/reindex work sheds first. Admission control exposes work that cannot meet deadline, source, reviewer, or budget constraints.

## Finding 23: release and profile changes are assurance changes

The decision system includes workflow, schemas, policy, profiles, mappings, procedures, connector/transform/sampler, model/prompt/provider route, renderer, and evaluation set. Any can alter evidence or reviewer behavior.

Active engagements pin the whole manifest. Upgrades run compatibility, replay, shadow, canary, impact, and rollback/forward-recovery checks. A frozen package is never rebuilt silently under a new renderer or narrative model.

## Finding 24: stage maturity expands proven scope, not professional authority

Stages 0–6 progress from governance to shadow, request/collection, approved sampling/TOD/TOE assistance, independent review/package freeze, bounded delivery, and multi-tenant scale. Every stage defines authority, architecture, I/O, state/events/effects, approvals, recovery, evaluation, and measurable exit gates.

The forbidden boundary stays fixed: the agent never becomes the legal interpreter, control operator, independent signing professional, or autonomous compliance authority.

## Finding 25: a product integration is not an evidence capability until qualified

The same product can expose different records, history, retention, permissions, time semantics, and completeness across edition, tenant configuration, API/export, region, and release. A green connectivity check proves neither correct resource binding nor evidence completeness.

Each adapter therefore has a narrow proof obligation, immutable manifest, owner, supported tenant/resource/configuration envelope, known limits, expiry, golden fixtures, and conformance report. Common tests cover identity/scope, authorization, time, pagination/volume, ordering/corrections, schema, freshness, completeness, hostile content, effects, lifecycle, and recovery. Source-specific tests turn vendor facts into executable assumptions.

## Finding 26: acquisition needs a receipt distinct from the artifact

An artifact digest identifies bytes. It does not record whether the adapter observed the correct account, query, period, pages, count, source watermark, freshness, known gap, retention profile, legal hold, or deletion state.

The Pass 2 design adds a typed, versioned acquisition receipt with source/query identity, adapter and qualification releases, attempt, observation window, raw/canonical manifests, explicit freshness/completeness vocabularies, limitations, and lifecycle state. The receipt describes acquisition; a procedure and independent reviewer still decide admissibility, relevance, reliability, and sufficiency.

## Finding 27: WORM, signatures, webhooks, warehouses, and MCP prove narrow claims

Primary-source checks on 2026-08-31 reinforced several limits:

- S3 Object Lock protects named object versions; governance retention can be bypassed by authorized permission, and new versions/delete markers remain possible. Azure and Google object-retention mechanisms have different configuration and irreversibility constraints.
- Acrobat Sign documents a 10 MB webhook payload limit and removal of optional payload fields. The notification must trigger an authoritative agreement/document fetch.
- BigQuery audit messages have a documented 100 KB limit, while BigQuery time travel is a configurable two-to-seven-day recovery window, not an audit archive. Snowflake documents up to 180-minute `ACCESS_HISTORY` latency and coverage/sharing limitations.
- MCP standardizes protocol operations, not audit semantics. The 2025-11-25 authorization specification uses resource indicators and forbids token passthrough; tasks are experimental and require authorization-context isolation where available.

These mechanisms become source-attributed inputs with explicit limits. None proves source truth, completeness, legal validity, control effectiveness, or an audit conclusion.

## Finding 28: the complete release unit is a behavior bundle

Control catalog, control profile, mapping, procedure, sampler, connector, transform, workflow/state schema, policy/SoD, context compiler, compactor, model/prompt/provider route, renderer, and evaluation suite can independently alter evidence or reviewer behavior. They require separate immutable identities inside one behavior bundle.

Candidate bundles run compatibility, replay, shadow, bounded canary, drift, rollback/forward-recovery, and controlled failure-mining gates. Active engagements and frozen packages stay pinned. Reviewer corrections and incidents enter governed evaluation corpora only after rights/privacy review; they never update production policy or memory automatically.

## Finding 29: disaster recovery is a deadline-and-people capacity problem

A restore must reconcile effect unknowns, re-establish tenant/keys/holds, recollect before source windows close, and drain independent-review queues while new work arrives. Database restoration alone does not prove recoverability.

Capacity uses net recovery drain rate after ongoing arrivals. Protected order is security/hold enforcement, unknown-effect reconciliation, expiring evidence windows, qualified review/package deadlines, then ordinary drafting and backfill. Cell-loss drills must test production-shaped volume, unavailable sources, partial key recovery, reviewer absence, stale workers, and business/audit correction decisions.

## Architecture alternatives and conditional choices

| Alternative | Decision | Rationale |
| --- | --- | --- |
| Autonomous audit agent with broad tools | Rejected | Conflates evidence, state, judgment, authorization, independence, and effects |
| GRC/workflow only, no model | Preferred when inputs/rules are structured | Simpler, deterministic, cheaper, easier to assure |
| Durable workflow plus bounded model worker | Default for mixed structured/unstructured evidence | Adds semantic assistance while preserving control |
| Multiple agents mirroring audit team roles | Not default | Extra model roles do not create real independence; increase nondeterminism/context/cost |
| One universal normalized control ontology | Rejected | Erases publisher identity, version, jurisdiction, scope, and relationship direction |
| OSCAL-native profile/result artifacts | Use when producer/consumer versions and semantics align | Strong interoperability, but schema validity is not substantive correctness |
| Custom typed domain schema with OSCAL import/export | Practical default for heterogeneous programs | Keeps operational invariants explicit while allowing profile adapters |
| Direct source API collection | Preferred where supported and approved | Better source identity/query lineage; still requires completeness/reliability testing |
| Organization-approved MCP adapter | Conditional, not a default | Useful only when narrower than a direct credential; protocol/tool/auth/release pinning does not replace domain receipts or qualification |
| Manual file/screenshot evidence | Conditional | Sometimes required; needs authenticated acquisition, limitations, and corroboration |
| Webhook/event-driven collection | Treat as low-latency hint plus reconciliation | Loss/expiry/order/retention require authoritative polling/checkpoints |
| WORM/object lock for every artifact | Risk/policy based | Valuable integrity/retention control; can conflict with deletion and raises cost/operations |
| RFC 3161/4998 long-term evidence records | Advanced optional profile | Useful for selected long-retention integrity; not source truth and not MVP |
| Continuous evidence | Conditional after stable periodic process | Improves freshness but adds drift/backlog/change-point complexity; no continuous opinion |
| Shared multi-tenant control plane | Moderate-risk default with strong partitions | Economical; needs active isolation/fairness/restore tests |
| Dedicated/cell deployment | High-risk/residency/contract choice | Stronger blast-radius boundary at higher operational cost |

## Material tensions and resolved positions

### Mapping automation versus false equivalence

Resolution: use model retrieval/comparison only for proposals; record explicit relationship/direction/elements/conditions/exclusions and require qualified version-pinned approval.

### Automated evidence versus professional sufficiency

Resolution: automated collection establishes a source artifact and lineage. Reviewer acceptance, source reliability, relevance, contradictions, and sufficiency remain separate.

### Native `compliant` state versus audit conclusion

Resolution: retain native state/level/time as attributed evidence. Never promote it to an agent-owned compliance/effectiveness state.

### Immutable storage versus privacy deletion

Resolution: apply risk-based immutable versions with configured retention/hold; maintain a policy-aware deletion lifecycle for raw, derived, index, cache, export, and backup records.

### Full traceability versus data minimization

Resolution: application-owned structured IDs/digests/decisions provide reconstruction; diagnostic traces exclude evidence content and can have shorter retention.

### Continuous collection versus period conclusion

Resolution: freeze versioned checkpoints with completeness/change-point information. Qualified humans decide period and report impact.

### Reviewer assistance versus automation bias

Resolution: evidence-centered workbench, blinded evaluation subsets, contradiction display, meaningful actions, decision rationale, time/override monitoring, and real-person independence.

### Stable active engagement versus current rules

Resolution: pin profile/release for reconstruction; current source changes trigger explicit impact/migrate/retest decisions, not silent drift.

### Retry availability versus duplicate disclosure

Resolution: semantic operation IDs, unknown state, destination reconciliation, exact-manifest approval, and a separately authorized correction/supplement.

### Scale efficiency versus tenant isolation/fairness

Resolution: cells and strong partitions, tenant-purpose-version identity everywhere, bounded fair queues, protected capacity, per-tenant budgets, and dedicated deployment when risk justifies it.

## Claims deliberately excluded

This research does not claim that:

- any architecture automatically complies with NIST, SOC 2, PCI DSS, ISO, FedRAMP, GAO, PCAOB, IIA, IAASB, GDPR, or another regime;
- one control mapping proves equivalence or allows evidence/conclusions to transfer;
- OSCAL schema validity proves correct controls, evidence, or results;
- an API, signed export, hash, WORM object, timestamp, or provenance graph proves source truth or sufficiency;
- a model can choose statistically or professionally appropriate sample parameters without an approved methodology;
- model confidence is assurance confidence;
- two models or service accounts create independence;
- a vendor “compliant” status, management attestation, closed remediation ticket, or green dashboard proves effectiveness;
- continuous evidence provides continuous assurance;
- the cited audit standards apply outside their jurisdiction/engagement scope;
- a future-effective standard, exposure draft, RFC, pilot, or preview is current law/methodology;
- provider privacy/security claims remain current without contract/configuration verification;
- the system can issue an attestation, certification, or audit opinion.

## Derived production guide set

| Guide | Main research findings implemented |
| --- | --- |
| [README](../../agents/compliance-audit-agent/README.md) | Boundary, records, authority, lifecycle, stages, canonical dependencies |
| [Workload fit, scope, and accountability](../../agents/compliance-audit-agent/01-workload-fit-scope-and-accountability.md) | Deterministic-first fit, jurisdiction limits, role ownership, Stage 0 |
| [Reference architecture, runtime, and authority](../../agents/compliance-audit-agent/02-reference-architecture-runtime-and-authority.md) | Bounded model worker, typed proposals/events/effects, trust boundaries, Stage 1 |
| [Control profiles, mappings, and change governance](../../agents/compliance-audit-agent/03-control-profiles-mapping-and-change-governance.md) | Version/maturity/rights pins, relationship mappings, OSCAL, drift |
| [Evidence requests, connectors, and immutable lineage](../../agents/compliance-audit-agent/04-evidence-requests-connectors-and-lineage.md) | Request state, connector realities, provenance, immutability limits, Stage 2 |
| [Sampling, test design, and control assessment](../../agents/compliance-audit-agent/05-sampling-test-design-and-control-assessment.md) | TOD/TOE boundary, population, deterministic sample, contradictions, Stage 3 |
| [Engagement state, context, memory, and orchestration](../../agents/compliance-audit-agent/06-engagement-state-context-memory-and-orchestration.md) | Durable aggregates, planning/replanning, typed lanes, all memory classes, compaction |
| [Reviewer independence, exceptions, and audit packages](../../agents/compliance-audit-agent/07-review-independence-exceptions-and-audit-packages.md) | Real-person SoD, decision/exception state, deterministic package, Stage 4 |
| [Security, privacy, retention, and tenant isolation](../../agents/compliance-audit-agent/08-security-privacy-retention-and-tenant-isolation.md) | Identity, injection, minimization, lifecycle, isolation, provider/rights controls |
| [Reliability, observability, evaluation, and failure injection](../../agents/compliance-audit-agent/09-reliability-observability-evaluation-and-failure-injection.md) | Hard gates, corpora, trajectories, faults, SLOs, failure mining |
| [Deployment, scale, cost, incidents, and staged evolution](../../agents/compliance-audit-agent/10-deployment-scale-cost-incidents-and-evolution.md) | Complete stages 0–6, delivery, tenancy, capacity, cost, upgrade, runbooks |
| [Integration qualification and audit-package walkthroughs](../../agents/compliance-audit-agent/11-integration-qualification-and-audit-package-walkthroughs.md) | Adapter conformance, source-family limits, acquisition receipts, typed identity chain, behavior bundle, package walkthroughs, exercises and runbooks |

## Refresh triggers

Refresh this packet and affected profiles when:

- NIST publishes a new SP 800-53/53A release, OSCAL model/release, mapping guidance, or final Privacy Framework 1.1;
- PCAOB future-effective sampling/evidence/documentation changes become effective or receive amendments;
- IAASB finalizes or withdraws the 2026 ISA 330/500/520 proposals, or a jurisdiction adopts them;
- GAO, IIA, IESBA, AICPA, ISO, PCI SSC, FedRAMP, or another in-scope authority changes criteria, methodology, assessor qualifications, formats, effective dates, or rights;
- a source program moves an RFC/pilot/preview to final or changes machine-readable package requirements;
- AWS, Azure, Google, GitHub, Okta, ServiceNow, Jira, an HR/finance system, e-sign provider, warehouse, or another connector changes API version, authentication, retention, pagination/order, fields/states, webhook lifecycle, rate limits, export behavior, edition, configuration, or terms;
- MCP publishes a new specification/extension or changes authorization, tool, task, transport, token, or discovery behavior used by a deployed server/client;
- a cloud object-lock, timestamp, canonicalization, encryption, or evidence-preservation mechanism changes;
- model/provider retention/training/region/security terms or model behavior changes;
- a new profile, jurisdiction, assurance type, data class, connector, destination, tenant class, region, or C2–C4 effect is introduced;
- a production incident, reviewer disagreement, sampling defect, independence breach, unsupported claim, or package correction exposes a missing contract;
- review capacity, cost, latency, source windows, or failure rates invalidate Stage 0 assumptions.

## Research limitations

- This packet is a technical synthesis, not an authoritative interpretation of law, professional standards, or a specific engagement contract.
- Many assurance materials are licensed; public landing pages do not expose all authoritative detail. The target organization must use its licensed/current sources and qualified professionals.
- Standards and programs have jurisdiction, adoption, transition, and effective-date differences that a generic blueprint cannot resolve.
- Vendor documentation describes product behavior at the cutoff but may change without this repository updating immediately; adapter tests and owner review are mandatory.
- Public documentation cannot reveal every source-system implementation, organization customization, internal control, reviewer relationship, or provider contract.
- Evidence quality and sample methodology are contextual. This packet deliberately does not prescribe universal sample sizes, materiality, reviewer qualifications, retention periods, or SLO targets.
- The research did not benchmark a specific model/provider/connector deployment. Model and cost choices require current task-specific evaluation.
- Cryptographic and immutable-storage patterns support integrity/preservation but do not settle legal admissibility, authenticity, or assurance sufficiency.
- Emerging continuous-compliance/automated-assurance approaches remain program- and maturity-specific; the blueprint labels proposals and pilots rather than treating them as settled practice.

## Selected primary and authoritative sources

### Control catalogs, assessment, mapping, and machine-readable artifacts

- [NIST SP 800-53 Revision 5 Update 1](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST SP 800-53A Revision 5](https://csrc.nist.gov/pubs/sp/800/53/a/r5/final)
- [OSCAL 1.2.3 release](https://github.com/usnistgov/OSCAL/releases/tag/v1.2.3)
- [OSCAL model layers](https://pages.nist.gov/OSCAL/learn/concepts/layer/)
- [OSCAL assessment layer](https://pages.nist.gov/OSCAL/learn/concepts/layer/assessment/)
- [OSCAL control mapping model](https://pages.nist.gov/OSCAL/learn/concepts/layer/control/mapping/)
- [NISTIR 8477: Mapping Relationships Between Documentary Standards, Regulations, Frameworks, and Guidelines](https://www.nist.gov/publications/mapping-relationships-between-documentary-standards-regulations-frameworks-and)
- [NIST SP 1347: Informative Reference Catalog](https://csrc.nist.gov/pubs/sp/1347/final)
- [NIST Cybersecurity and Privacy Reference Tool](https://csrc.nist.gov/Projects/cprt)

### Audit, internal control, evidence, documentation, sampling, and independence

- [GAO Green Book](https://www.gao.gov/greenbook)
- [GAO FISCAM 2024](https://www.gao.gov/products/gao-24-107026)
- [GAO Financial Audit Manual](https://www.gao.gov/financial-audit-manual)
- [GAO Yellow Book](https://www.gao.gov/yellowbook)
- [PCAOB AS 1000](https://pcaobus.org/oversight/standards/auditing-standards/details/as-1000--general-responsibilities-of-the-auditor-in-conducting-an-audit)
- [PCAOB AS 1105](https://pcaobus.org/oversight/standards/auditing-standards/details/AS1105)
- [PCAOB AS 1215](https://pcaobus.org/oversight/standards/auditing-standards/details/AS1215)
- [PCAOB AS 2201](https://pcaobus.org/oversight/standards/auditing-standards/details/AS2201)
- [PCAOB AS 2301](https://pcaobus.org/oversight/standards/auditing-standards/details/AS2301)
- [PCAOB AS 2310](https://pcaobus.org/oversight/standards/auditing-standards/details/AS2310)
- [PCAOB AS 2315 future-effective version](https://pcaobus.org/oversight/standards/auditing-standards/details/as-2315--audit-sampling-%28effective-on-12-15-2026%29)
- [PCAOB AS 2810](https://pcaobus.org/oversight/standards/auditing-standards/details/AS2810)
- [PCAOB ethics and independence rules](https://pcaobus.org/oversight/standards/ethics-independence-rules)
- [IIA Global Internal Audit Standards](https://www.theiia.org/en/standards/)
- [IESBA 2025 Handbook of the International Code of Ethics](https://www.ethicsboard.org/news-events/2025-10/now-available-iesba-handbook-2025-edition)
- [IAASB 2025 Handbook](https://www.iaasb.org/publications/2025-handbook-international-quality-management-auditing-review-other-assurance-and-related-services)
- [IAASB proposed revisions to ISA 330, ISA 500, and ISA 520](https://www.iaasb.org/publications/proposed-revisions-audit-evidence-risk-response-isa-330-isa-500-isa-520)
- [IAASB ISA 500 series project](https://www.iaasb.org/consultations-projects/isa-500-series)
- [ISO 19011:2026](https://www.iso.org/standard/19011)
- [AICPA SOC overview](https://www.aicpa-cima.com/soc4so)
- [AICPA Trust Services Criteria with revised points of focus 2022](https://www.aicpa-cima.com/resources/download/2017-trust-services-criteria-with-revised-points-of-focus-2022)

### Program-specific and evolving compliance sources

- [PCI DSS document library](https://www.pcisecuritystandards.org/document_library/?class=pcidss&doc=pci_dss)
- [PCI DSS v4.0 resource hub](https://blog.pcisecuritystandards.org/pci-dss-v4-0-resource-hub)
- [PCI SSC customized and compensating approach guidance](https://blog.pcisecuritystandards.org/pci-ssc-publishes-new-guidance-on-compensating-controls-and-the-customized-approach)
- [PCI DSS v4.0.1 next-iteration request for comments](https://blog.pcisecuritystandards.org/request-for-comments-pci-data-security-standard-pci-dss-v4.0.1)
- [FedRAMP RFC-0024 machine-readable authorization packages](https://www.fedramp.gov/rfcs/0024/)
- [FedRAMP RFC-0006 Key Security Indicators](https://www.fedramp.gov/rfcs/0006/)
- [FedRAMP 20x Key Security Indicators reference](https://preview.fedramp.gov/2026/reference/20x/b/key-security-indicators/)
- [FedRAMP Rev. 5 access-control page](https://www.fedramp.gov/2026/providers/rev5/controls/access-control/)

### Provenance, integrity, retention, and privacy

- [W3C PROV overview](https://www.w3.org/TR/prov-overview/)
- [RFC 8785: JSON Canonicalization Scheme](https://www.rfc-editor.org/rfc/rfc8785)
- [RFC 3161: Time-Stamp Protocol](https://www.rfc-editor.org/info/rfc3161/)
- [RFC 4998: Evidence Record Syntax](https://www.rfc-editor.org/info/rfc4998/)
- [NIST chain of custody glossary](https://csrc.nist.gov/glossary/term/chain_of_custody)
- [NISTIR 8387: Digital Evidence Preservation](https://www.nist.gov/publications/digital-evidence-preservation-considerations-evidence-handlers)
- [AWS S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html)
- [Azure immutable storage for blobs](https://learn.microsoft.com/en-us/azure/storage/blobs/immutable-storage-overview)
- [Google Cloud Storage Object Retention Lock](https://docs.cloud.google.com/storage/docs/object-lock)
- [GDPR official text](https://eur-lex.europa.eu/legal-content/EN/TXT/?toc=OJ%3AL%3A2016%3A119+%3ATOC&uri=uriserv%3AOJ.L_.2016.119.01.0001.01.ENG)
- [EDPB basic principles](https://www.edpb.europa.eu/topics/key-gdpr-concepts/basic-principles_en)
- [NIST Privacy Framework 1.0](https://csrc.nist.gov/pubs/cswp/10/nist-privacy-framework-version-10/final)
- [NIST Privacy Framework 1.1 project](https://www.nist.gov/privacy-framework/new-projects/privacy-framework-version-11)

### Connector and evidence-platform documentation

- [AWS Audit Manager overview](https://docs.aws.amazon.com/audit-manager/latest/userguide/what-is.html)
- [AWS Audit Manager concepts](https://docs.aws.amazon.com/audit-manager/latest/userguide/concepts.html)
- [AWS Audit Manager evidence collection](https://docs.aws.amazon.com/audit-manager/latest/userguide/how-evidence-is-collected.html)
- [AWS Audit Manager evidence review](https://docs.aws.amazon.com/audit-manager/latest/userguide/review-evidence.html)
- [AWS Audit Manager Evidence API](https://docs.aws.amazon.com/audit-manager/latest/APIReference/API_Evidence.html)
- [Azure Policy compliance states](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/compliance-states)
- [Azure Policy REST API](https://learn.microsoft.com/en-us/rest/api/policy/)
- [Azure Policy States API](https://learn.microsoft.com/en-us/rest/api/policyinsights/policy-states)
- [Azure Policy attestations API](https://learn.microsoft.com/en-us/rest/api/policyinsights/attestations/get-at-resource-group?view=rest-policyinsights-2024-10-01)
- [Google Cloud Asset Inventory asset history](https://docs.cloud.google.com/asset-inventory/docs/get-asset-history)
- [GitHub organization audit log](https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/reviewing-the-audit-log-for-your-organization)
- [Okta System Log API](https://developer.okta.com/docs/reference/system-log-query/)
- [ServiceNow Audit Management overview](https://www.servicenow.com/docs/r/governance-risk-compliance/audit-management/c_GRCAudits.html)
- [ServiceNow request evidence](https://www.servicenow.com/docs/r/governance-risk-compliance/audit-management/request-evidence.html)
- [ServiceNow evidence-request workflow](https://www.servicenow.com/docs/r/governance-risk-compliance/audit-management/evidence-request-workflow.html)
- [Jira Cloud REST API v3 introduction](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro)
- [Jira Cloud webhooks API](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-webhooks/)
- [Oracle NetSuite System Notes and System Notes v2](https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/article_160225379741.html)
- [Oracle NetSuite line-level audit trail](https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/section_N557476.html)
- [Adobe Acrobat Sign webhook overview](https://helpx.adobe.com/au/sign/developer/webhook/overview.html)
- [BigQuery audit logs](https://cloud.google.com/bigquery/docs/reference/auditlogs)
- [BigQuery time travel](https://cloud.google.com/bigquery/docs/time-travel)
- [Snowflake ACCESS_HISTORY](https://docs.snowflake.com/en/sql-reference/account-usage/access_history)
- [MCP 2025-11-25 authorization](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization)
- [MCP 2025-11-25 tasks](https://modelcontextprotocol.io/specification/2025-11-25/basic/utilities/tasks)

### Foundational agent and context research

- [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629)
- [Toolformer: Language Models Can Teach Themselves to Use Tools](https://arxiv.org/abs/2302.04761)
- [Lost in the Middle: How Language Models Use Long Contexts](https://arxiv.org/abs/2307.03172)

These papers motivate bounded semantic/tool/context capabilities. They do not establish production audit reliability, authorization, evidence sufficiency, reviewer independence, or legal compliance.

## Pass 2 quality statement

The research produced a differentiated, version-aware blueprint rather than a generic agent overview. Pass 2 added executable adapter qualification, representative GRC/cloud/IAM/CI/CD/ticket/HR/finance/portal/storage/e-signature/warehouse boundaries, guarded MCP use, acquisition receipts, exact seven memory lifetimes and loss-aware continuity, explicit behavior-bundle identities, end-to-end audit-package exercises, and recovery-load/DR gates. It records source maturity and jurisdiction limits, rejects unsupported equivalence/compliance claims, and converts findings into schemas, diagrams, staged gates, failure injections, runbooks, and accountable human ownership.

Before applying it to a real engagement, the organization must replace example profiles, role policies, sample rules, retention periods, SLOs, provider assumptions, and package language with qualified, approved, current requirements and must validate every connector against the deployed source/version.

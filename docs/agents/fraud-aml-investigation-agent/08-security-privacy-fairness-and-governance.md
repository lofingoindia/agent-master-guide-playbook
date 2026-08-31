# Security, Privacy, Fairness, and Governance

> **Purpose:** Protect highly sensitive case data and people from unauthorized access, injected instructions, excessive collection, discriminatory burden, and ungoverned behavioral change.  
> **Section index:** [Fraud and AML investigation agent](README.md)

## Threat model

This workload combines high-value customer/transaction data, confidential filings, adversarial subjects, external documents, privileged operations, and a probabilistic interpreter. Assume:

- customers, counterparties, compromised users, vendors, or public content may intentionally manipulate names, references, transaction narratives, files, or websites;
- insiders may seek unrelated customer cases, filed-SAR/STR status, regulator correspondence, or bulk exports;
- a connector, model/provider, browser, retrieval index, evaluation dataset, cache, trace, support tool, or backup can become a disclosure path;
- entity resolution and monitoring can impose real burdens even before a formal decision;
- model or policy updates can create correlated false negatives or mass false positives;
- the model may follow malicious instructions, invent evidence, choose excessive scope, or make a persuasive but unsupported recommendation.

The safe assumption is that model input and output are untrusted. Identity, authorization, case mutation, and effect execution stay deterministic.

## Trust zones

~~~mermaid
flowchart LR
    EXT["External / customer / public / vendor content"] --> ING["Quarantine + validation + classification"]
    ING --> DATA["Protected source and evidence zone"]
    USER["Eligible workforce identity"] --> GATE["Identity + device + purpose + case entitlement"]
    GATE --> APP["Case application / policy zone"]
    DATA --> PROJ["Minimized evidence projection"]
    APP --> PROJ
    PROJ --> AI["Restricted reasoning zone"]
    AI --> VAL["Proposal validation"]
    VAL --> APP
    APP --> EFF["Separated credentialed effect zone"]
    APP --> AUD["Restricted audit / confidentiality zone"]
    AI --> TEL["Redacted observability zone"]
~~~

Network adjacency is not trust. Every transition enforces principal, tenant, legal entity, purpose, jurisdiction, resource scope, field classification, action, and time. The effect zone and audit/SAR-confidentiality zone are not reachable through model-selected URLs or generic tools.

## Authorization model

| Decision input | Example | Enforcement point |
|---|---|---|
| Principal/workload identity | Investigator, QA reviewer, case service, effect worker | Identity provider and service-to-service identity |
| Workforce eligibility | AML-trained, sanctions role, filing authority, independent tester | Entitlement/HR governance, rechecked at use |
| Tenant/legal entity | Institution subsidiary and booking entity | Admission, queries, caches, case store, queues, exports |
| Purpose | AML case, fraud case, sanctions review, control testing | Source adapter and policy decision point |
| Resource relationship | Assigned case, supervised queue, approved sample | Case service and row/field policy |
| Jurisdiction/data class | SAR-confidential, customer financial, special-category/sensitive | Projection, display, provider routing, retention/export |
| Action and tier | Read, draft, approve, file, freeze, administer | Tool broker, workflow, effect service, administration |
| Context | Case version, source freshness, device/session risk, deadline | At read, proposal apply, approval, and commit |

Use deny-by-default, short-lived credentials, separate service identities, and server-derived scope. Tool metadata such as `readOnly` is useful documentation but never authority. Reauthorize on every tool call and immediately before a consequential effect.

## Anti-tipping-off, confidentiality, and separation of duties

Treat the existence, consideration, contents and supporting material of a SAR/STR according to the exact applicable
confidentiality policy. Do not place those facts in general customer profiles, support/search indexes, ordinary
notifications, shared investigator preferences, broad analytics, model-provider threads, screenshots or sampled
traces. Customer-facing text is prepared only through a separately authorized communication process that receives the
minimum permitted reason; the model cannot answer “am I under investigation?” or infer what may lawfully be disclosed.

Encode separation of duties as a deterministic eligibility graph: requester, investigator, filing decision-maker,
sanctions matcher, restrictive-action approver, dispatcher, administrator and independent tester are distinct roles
where policy requires. Recheck current identity, role, case/queue relationship and approval generation at commit. A
manager title, identity-provider group, prior approval or model recommendation is not sufficient authority. Break-glass
access has a reason, bounded scope/time, independent review and no ability to approve its own resulting effect.

## Prompt injection and adversarial evidence

Transaction narratives, customer notes, PDFs, emails, adverse-media articles, registry text, URLs, OCR, images, and tool errors are data. They can contain instructions aimed at the model or reviewer.

Controls:

1. Ingest into quarantine; validate file type/size, malware policy, parser behavior, active content, links, encoding, and decompression bounds.
2. Preserve the original and use a derived safe representation with explicit provenance.
3. Delimit source content structurally and label its origin/classification; never concatenate it into system/control instructions.
4. Use typed extraction and cited claims; a source cannot request a tool or change the plan/policy.
5. Disable arbitrary URL fetch, browser navigation, code execution, shell, generic database access, email, and external messaging.
6. Allowlist destinations and parameters at the adapter; enforce egress at the network layer.
7. Require independent validation before using extracted identifiers, addresses, accounts, or links to widen retrieval.
8. Test direct, indirect, encoded, multilingual, image/OCR, cross-document, delayed, tool-result, and memory-poisoning attacks.

Prompt injection does not have a complete prompt-only solution. Layered restriction, minimization, validation, and impact containment are mandatory. See [prompt injection and untrusted data](../../security/prompt-injection-and-untrusted-data.md) and the [agent threat model](../../security/agent-threat-model.md).

## Data lifecycle controls

| Phase | Required control |
|---|---|
| Collect | Legal basis and purpose; proportional source/field/window; record provenance and notices/exceptions as applicable |
| Store | Classification, encryption, tenant/legal-entity isolation, field/row access, integrity, residency, retention, legal hold |
| Project to model | Data minimization, pseudonymous handles where useful, field redaction, approved provider/region/retention/training terms |
| Retrieve/cache/index | Entitlement/purpose at query time; tenant/version-aware keys; permission/deletion propagation; poisoning controls |
| Display/export | Need-to-know masking, watermark/context, download limits, destination approval, access record |
| Observe/support | Redacted metadata by default; restricted content capture; no secrets; break-glass with review |
| Evaluate | Governed de-identified/synthetic sets where possible; case cutoff; strict access; contamination and retention checks |
| Retain/delete | Jurisdiction/purpose schedule, litigation/regulatory hold, derived-copy propagation, verifiable deletion/tombstone |

GDPR Article 5 principles such as purpose limitation, data minimization, accuracy, storage limitation, integrity/confidentiality, and accountability are useful engineering constraints where the GDPR applies. They are not a universal substitute for jurisdiction-specific legal analysis. The EDPB's Opinion 28/2024 also underscores that AI-model data protection assessments are case-specific.

## Privacy design record

Before production, document:

- controller/processor and participating legal entities;
- purposes, legal bases, jurisdictions, data-subject classes, sensitive data, sources, recipients, and cross-border paths;
- whether decisions significantly affect people and what meaningful human review means in this workflow;
- necessity/proportionality of model use and every source/field/window;
- reidentification, inference, access, disclosure, retention, correction, deletion, legal-hold, and confidentiality risks;
- provider data use, training, abuse monitoring, retention, subprocessors, regional processing, incident obligations, deletion evidence, and exit plan;
- residual risk, accountable sign-offs, reassessment triggers, and test evidence.

Privacy review is needed again for a new data domain, memory class, provider, country, language, tenant-sharing path, evaluation set, or retention behavior.

## Secrets and provider boundary

- Use workload identity or a secret manager; never place credentials in prompts, context, tool results, logs, files, or examples.
- Give reasoning workers read-only, purpose-bound broker credentials; give effect adapters separate narrow credentials.
- Rotate, revoke, and audit keys; exercise provider and credential kill switches.
- Pin and monitor provider region, storage, retention, training/data-use, abuse-review, and subprocessor settings.
- Minimize content sent to the provider. Keep raw evidence and filing artifacts in the institution's protected plane where feasible.
- Treat provider-side threads, files, caches, traces, compaction, and background execution as separate data stores requiring lifecycle review.
- Maintain a tested route to proposal-only/manual operation if a provider is unavailable or no longer approved.

See [permissions, sandboxing, and secrets](../../security/permissions-sandboxing-and-secrets.md).

## Supplier, data, and software supply chain

Inventory model/hosting providers, source vendors, official-list mirrors, case/identity systems, SDKs, parsers/OCR,
normalizers/transliterators, rule/model/feature pipelines, graph/search engines, container/base images, build actions,
artifact registries, subprocessors and support paths. Pin approved versions or verified digests; preserve provenance and
SBOM/data/model manifests; verify signatures where available; scan for vulnerabilities, malware and secrets; restrict
build/release credentials; and prevent unreviewed packages, models, lists or parser rules from reaching production.

Supplier attestations and signed artifacts are evidence, not validation of the configured intended use. Contract and
monitor managed changes, deprecation, regional routing, list/source redistribution, data retention/training, support
access, incidents and exit/export. A model alias, dependency, sanctions parser, normalization library, provider score,
typology pack or feature mapping change is part of the behavior release. Unknown or unsigned drift disables the
affected operation and triggers dependency/case impact analysis; it does not silently fall back to “latest.”

## Fairness and unnecessary-burden controls

Financial-crime controls are risk-based but can create disproportionate investigation, delay, account restriction, information requests, or exit burdens. Fairness testing must cover the full pipeline, not only model text.

| Layer | Harm to measure | Useful evidence |
|---|---|---|
| Source coverage | Sparse or lower-quality data for a group/jurisdiction/language | Missingness, staleness, transliteration and registry coverage by slice |
| Monitoring/alerting | Unequal alert rate or detection opportunity | Alert rate, exposure-adjusted rate, threshold sensitivity, known selection bias |
| Entity resolution | False merges/splits and watchlist false matches | Human-adjudicated candidate precision/recall by name structure, script, country, entity type |
| Agent investigation | Unequal evidence depth, negative language, unsupported escalation | Citation/coverage, stop reasons, query count, recommendation and error rates by approved slice |
| Human review | Automation bias or inconsistent overrides | Acceptance/edit/dissent/return rates, inter-reviewer agreement, blind QA |
| Effects | Delays, holds, restrictions, exits, repeated requests | Rate, duration, correction/remediation, severity and appeal/review outcomes |

Choose slices with legal/privacy review. Protected attributes may be unavailable or restricted; proxies can mislead. If direct measurement is not lawful, document the limitation and use approved audits, qualitative review, geography/language/data-quality slices, and scenario testing. Do not infer sensitive attributes with the model merely to measure fairness.

Do not optimize only aggregate investigator agreement. A system can be consistently wrong or reproduce historical selection. Include benign controls, counterfactual identity/name/language variations, false-negative probes, and downstream burden.

Explainability means a reviewer can reconstruct the alert/rule/model release, point-in-time inputs, source revisions,
entity/graph derivations, evidence supporting and contradicting each claim, missing coverage, applicable policy,
human edits/decision and downstream effect. A generated rationale or feature-importance chart may assist review but
cannot replace this lineage, prove causality, disclose protected internals improperly or justify an adverse action by
itself. Test whether reviewers can detect a fluent wrong recommendation, not merely whether they say the explanation is
clear.

## Model and change governance

Traditional model-risk principles—conceptual soundness, independent validation, outcomes analysis, ongoing monitoring, change control, and effective challenge—are useful for governing the whole system. However, the U.S. Federal Reserve's 2026 SR 26-2 guidance explicitly says generative and agentic AI are outside that guidance's scope. Applying those principles here is an engineering and governance analogy, not a statement that SR 26-2 legally governs this agent.

Assign owners for:

| Governed object | Accountable functions |
|---|---|
| Workload/authority | AML/fraud operations, legal/compliance, business owner |
| Case/policy/jurisdiction rules | Compliance/legal, operations, technology control owner |
| Data/entity/graph/monitoring models | Data owner, model/analytics owner, independent validation |
| Agent behavior release | Product/engineering, security, privacy, model risk/AI governance, operations |
| Filing/restriction adapters | Legal/compliance authority and destination operations |
| Evaluation/QA | Independent control/testing function; separate from builders where impact warrants |
| Incidents | Security/privacy plus AML/sanctions/legal/operations according to event |

Independent testing should have sufficient access and authority to challenge assumptions, inspect source-to-decision lineage, reproduce samples, test permissions and recovery, and report findings outside the delivery team. The FFIEC manual treats independent BSA/AML testing as a required component of the compliance program for covered U.S. institutions; exact applicability and frequency remain institution/jurisdiction specific.

## Security invariants

Release and runtime tests must prove:

- no cross-tenant/legal-entity/purpose read or cache hit;
- no model-visible filing/restriction/admin credentials or direct sink;
- no D3/D4 effect without exact current policy and eligible approval;
- no stale case/evidence/list/policy version applied silently;
- no source text can change authority, invoke a tool, or widen scope;
- no unsupported claim can enter a final review package as verified fact;
- no unknown external outcome is retried without reconciliation;
- no SAR/STR-confidential content enters general telemetry, search, memory, or notifications;
- all access, policy decisions, proposals, approvals, effect attempts, receipts, and break-glass events are attributable;
- emergency stops work without a healthy model provider.

## Incident classes

| Incident | Immediate containment |
|---|---|
| Cross-tenant/purpose disclosure | Stop admission and affected retrieval/provider route; revoke access; preserve audit; legal/privacy response |
| SAR/STR confidentiality leak | Restrict affected artifacts and identities; disable export/search/trace path; escalate to designated legal/compliance/security owners |
| Prompt-injection/tool exploit | Disable tool/source/release; block egress; quarantine content and dependent memory/evals |
| Unauthorized/repeated effect | Stop effect class; revoke credential; reconcile all in-flight intents; activate legal/operations remediation |
| Poisoned list/source/typology/index | Pin/quarantine version; trace dependent cases/decisions; rescreen/re-review under approved scope |
| False-negative cluster | Proposal-only/manual fallback for affected family; preserve evidence; targeted lookback under approved governance |
| Mass false positives/biased burden | Pause affected release/rule; protect deadlines and customer processes; measure and remediate impacted cohorts |
| Insider/break-glass abuse | Revoke access, preserve immutable access evidence, security/HR/legal process |

Use the general [deployment, release, and incident response](../../operations/deployment-release-and-incident-response.md) guidance and define category-specific owners and notification constraints.

## Checklist

- [ ] Threat model covers adversarial data, insiders, connectors, providers, memory/indexes, correlated model errors, and consequential effects.
- [ ] Principal, tenant, purpose, jurisdiction, resource, data class, action, case version, and context are enforced outside the model.
- [ ] SAR/STR confidentiality, anti-tipping-off, customer communication and separation-of-duties paths are technically isolated and tested.
- [ ] Prompt injection is contained through quarantine, structure, typed tools, egress restriction, and impact limits.
- [ ] Privacy lifecycle includes prompts, provider state, caches, indexes, traces, evaluations, exports, queues, backups, deletion, and legal hold.
- [ ] Provider terms/settings, secrets, rotation, revocation, regional path, incident obligations, and exit are validated.
- [ ] Suppliers, subprocessors, dependencies, list/parser/data/model artifacts and managed changes map to the signed behavior release.
- [ ] Fairness testing covers source, alert, matching, agent, human, and effect burden with lawful slices and limitations.
- [ ] Independent validation and testing can challenge the system and inspect complete provenance.
- [ ] Category-specific incident containment and emergency stops have been exercised.

## Sources and next guide

- [OWASP Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [NIST Privacy Framework](https://www.nist.gov/privacy-framework)
- [GDPR Article 5](https://eur-lex.europa.eu/eli/reg/2016/679/art_5/oj)
- [EDPB Opinion 28/2024 on AI models](https://www.edpb.europa.eu/documents/opinion-of-the-board-art-64/opinion-282024-on-certain-data-protection-aspects-related-to_en)
- [FATF — Opportunities and Challenges of New Technologies for AML/CFT](https://www.fatf-gafi.org/en/publications/Digitaltransformation/Digital-transformation.html)
- [Federal Reserve — Supervisory Guidance on Model Risk Management (SR 26-2, 2026)](https://www.federalreserve.gov/frrs/guidance/supervisory-guidance-on-model-risk-management.htm)
- [FFIEC BSA/AML Manual — Independent Testing](https://bsaaml.ffiec.gov/manual/AssessingTheBSAAMLComplianceProgram/03_ep)

Next: [Evaluation, observability, SLOs, and failure injection](09-evaluation-observability-slos-and-failure-injection.md).

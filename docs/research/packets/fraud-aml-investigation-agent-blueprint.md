# Fraud and AML Investigation Agent Blueprint — Research Packet

> **Research status:** Pass 2 complete; current-source and operation-level production synthesis  
> **Research date:** 2026-08-31  
> **Category:** Registry category 40 — Fraud and AML investigation  
> **Output:** [Fraud and AML Investigation Agent Blueprint](../../agents/fraud-aml-investigation-agent/README.md)  
> **Scope boundary:** Customer, account, transaction and entity investigation; alert/case workflows; typology and network evidence; KYC/CDD/EDD; sanctions/watchlist candidate analysis; investigator hypotheses/decisions; and filing/escalation preparation. Cyber forensics belongs to the security-investigation category; ordinary ledger reconciliation belongs to finance; audit assurance, legal advice, claims adjudication, and dispositive filing/freeze/offboarding remain outside the agent.

## Research question

What is the smallest production architecture that lets a model materially improve fraud/AML investigation while preserving source semantics, case reproducibility, privacy/confidentiality, human/legal authority, safe external effects, and reliable operation from MVP through governed scale?

Subquestions:

- Which work remains deterministic, and when does a bounded model loop add value?
- How should alerts, cases, source facts, derived facts, identity candidates, hypotheses, decisions, filings, and effects differ?
- Which transaction, KYC, entity, graph, list, adverse-media, filing, and restriction integrations are justified, and what are their failure semantics?
- How should exact data/streaming/case/list/KYC/graph/model/notification operations be qualified by tenant,
  configuration, role, scope, version, receipt meaning, expiry and recovery evidence?
- What state, context, compaction, memory, planning, provenance, idempotency, recovery, security, privacy, evaluation, SLO, capacity, incident, and release controls are required?
- Where do current official AML/sanctions expectations, agent-engineering practice, and real production constraints disagree or leave gaps?

## Method

Research began with the repository taxonomy, category registry, expansion program, cross-cutting control packet, and adjacent security-investigation, compliance-audit, back-office, finance, runtime, security, reliability, evaluation, context/memory, tool, and operations guides. This prevented category drift and reused the repository's control model.

External research then prioritized current primary sources:

1. FATF standards, amendments, beneficial-ownership, information-sharing, financial-inclusion, and technology guidance.
2. FinCEN, FFIEC, OFAC, UN, EU/AMLA, UK NCA, and U.S. interagency primary regulatory/supervisory material.
3. W3C/ISO/NIST/OpenTelemetry specifications and guidance for provenance, messages, tracing, privacy, AI risk, security evaluation, and incident response.
4. Official provider/engineering guidance for bounded agents, context, evaluation, idempotency, SRE, and failure handling.
5. Primary research repositories/papers only to characterize public graph and synthetic AML evidence limits.

Important claims were cross-checked across regulatory and systems sources. Volatile pages were checked for current status/dates. Vendor guidance was used only for vendor or implementation behavior, not as legal authority. The search stopped after additional queries mostly repeated the same architecture and governance implications rather than adding new control classes.

## Core synthesis

The selected system is **not an autonomous investigator or decision maker**. It is one bounded, read-only reasoning loop inside a deterministic, durable case workflow:

- deterministic systems own admission, identity, source snapshots, graph/features, case state, timers, policy, approvals, filings/effects, audit, and reconciliation;
- the model chooses among narrow typed reads and proposes cited hypotheses, gaps, next steps, and draft packages;
- qualified humans and separately credentialed systems own dispositions, filings, holds, sanctions actions, customer restrictions, external disclosures, and administration;
- the application treats the model as an untrusted proposer and validates schema, provenance, purpose, case version, current policy, authority, and semantic effect identity;
- multi-agent orchestration is rejected by default because it increases correlated error, shared-context leakage, duplicated queries, state/authority ambiguity, and evaluation burden without evidence that the workload needs it.
- operation capabilities are approved per configured action—not per vendor name—and expire on provider, plan, region,
  role, schema, list/source meaning, limits or security change;
- provider acknowledgements, model aliases, graph paths, identity decisions and message delivery states keep their
  narrow technical meanings and never become legal, filing, sanctions, KYC or customer-action outcomes;
- recovery is complete only after list/source/case/effect/clock/evidence convergence and safe backlog drain, not merely
  restored compute.

This directly implements the repository-wide rule: reasoning proposes; deterministic application policy authorizes; external effects use semantic identity, receipts, postconditions, and reconciliation.

## Why a bounded agent may be justified

### Deterministic baseline

Use ordinary software for alert/rule computation, list ingestion, schema validation, deduplication, identity keys, pagination/coverage, transaction lifecycle normalization, temporal windows, ownership arithmetic, graph construction, fixed routing, deadlines, permissions, filing-format validation, idempotency, and reconciliation.

### Candidate model value

A model loop is worth evaluating only where the task requires adaptive synthesis across heterogeneous evidence, competing explanations, ambiguous identity, narrative drafting, or choosing the next discriminating read. It must beat the deterministic search/workflow baseline on the same cases in completeness, correction effort, time to a verified review package, and total cost while preserving hard invariants.

### Rejection conditions

Reject or remove the loop if deterministic retrieval/templates perform equivalently; source semantics are too weak; labels/evaluation cannot detect severe failures; required evidence cannot be exposed lawfully; investigator capacity makes meaningful review impossible; the only desired value is direct automatic filing/restriction; or effect destinations cannot reconcile ambiguous outcomes.

## Architectural decision record

| Decision | Selected | Alternatives rejected or deferred | Evidence/reason |
|---|---|---|---|
| Orchestration | One bounded investigator within one case workflow | Default multi-agent “analyst teams” | Smallest inspectable authority/state surface; deterministic parallel retrieval supplies safe concurrency |
| State | Transactional case state plus append-only business/audit events and protected evidence store | Transcript/provider thread/vector index as state; event sourcing everywhere | Resume/recovery needs typed durable truth, but full event sourcing is not automatically necessary |
| Agent tools | Purpose-bound, read-only typed adapters; typed proposals | Generic SQL/browser/shell/email; model-visible destination tools | Reduces injection, exfiltration, overcollection, and effect risk |
| Effects | Separate outbox/effect ledger, current-policy commit check, semantic idempotency, reconciliation | Direct model filing/freeze/offboarding; blind retries; “exactly once” claims | External outcomes are ambiguous and high impact |
| Graph work | Versioned deterministic jobs and bounded typed graph queries | Model-generated durable graph; unbounded traversal | Reproducibility, temporal semantics, cost, and reviewability |
| Entity resolution | Candidate links with method, alternatives, limitations, review | Single canonical merged identity from model/vendor score | False merges/splits have material AML/sanctions/fairness impact |
| Context | Fresh compiler projection with manifest and evidence handles | Raw case dump or conversation replay | Minimization, source/version control, compaction safety |
| Memory | Case store plus versioned knowledge; no hidden customer/user memory; curated episodic exemplars only if justified | Global customer dossier, raw case narrative memory, cross-tenant learning | Purpose/confidentiality, poisoning, anchoring, deletion, selection bias |
| Evaluation | Portfolio of time-correct replay, benign/synthetic/adversarial/reliability/shadow/human evidence | SAR/STR/disposition as truth; one aggregate accuracy score | Outcomes are delayed, selected, uncertain, and system-influenced |
| Release | Whole behavior manifest, shadow/canary, one-dimensional expansion | Prompt hotfixes, moving provider alias, simultaneous scope/authority increase | Behavioral inputs span code, model, prompt, data, policy, analytics and operations |
| Adapter qualification | Expiring operation-level capability manifest with negative, ambiguity, restore and reconciliation tests | Product/vendor allowlist or connector name as authority | Role, tenant/configuration, object/field, API/schema, list/source and effect semantics vary per operation |
| Continuity | Exactly seven canonical memory lifetimes and a loss-aware digest-verified receipt | Extra implicit caches/index memory or prose/provider-token resume | Restart must preserve versions, watermarks, clocks, approvals, contradictions and unknown effects |
| Recovery | Restore authority/lists/clocks/effect reconciliation before lower-risk redrive | Health-check-only DR or blind queue replay | Recovery backlog can overload providers, reviewers and deadlines after infrastructure returns |

## Domain object separation

| Object | Meaning | Must not become |
|---|---|---|
| Alert | A monitoring/referral producer requested review under a version | Accusation or case disposition |
| Case | Durable scoped investigation and decisions | Model conversation |
| Source fact | What a named source/revision reported | Institutionally verified universal truth |
| Derived fact | Reproducible computation from cited inputs | Legal conclusion |
| Entity/relationship candidate | Fallible linkage with alternatives/limitations | Confirmed identity through confidence alone |
| Typology indicator | Pattern relevant to investigation | Proof of criminal conduct |
| Hypothesis | Testable interpretation | Verdict |
| Human decision | Accountable judgment under policy/evidence snapshot | Ground-truth label |
| Filing | Confidential report under a specific legal process | Proof of crime or general customer-profile field |
| Effect | Approved semantic external operation | A retriable natural-language command |
| Trace | Operational correlation/performance record | Audit ledger or evidentiary record |

## Integration decision ledger

| Integration | Decision | Required contract | Main failure/limit |
|---|---|---|---|
| Core/ledger transactions | Include | Semantic transaction lifecycle, stable IDs, revisions/status/reversals, event/posting time, coverage | Double counting, late posting, silent correction |
| Payment messages | Include per rail | Original message, schema version, amendment/cancel/intermediary lineage, stable rail ID | Parsed narrative loses semantics; standards evolve |
| Customer/account/KYC | Include | Party/account/relationship IDs, verification and valid-time history, source and owner | Current flattened profile leaks future/stale information |
| Alert/case system | Include | Producer/version, semantic dedupe, case version/state/deadline/decision vocabulary | Alert or transcript mistaken for truth |
| Official sanctions/watchlists | Include when applicable | Authority, program/regime, entry/list revision, publication time, completeness/freshness | Latest-only/nonofficial snapshot; named-list-only coverage |
| PEP/adverse-media provider | Conditional | Source/entry/article revision, entity candidate, event/publication time, language, limitations/licensing | Opaque scores, coverage bias, false identity, injected content |
| Identity/device/fraud telemetry | Conditional | Provider namespace, binding method, event time, precision/spoofing and purpose limits | Shared/recycled identifier treated as person |
| Company registry/LEI/BO | Conditional but common | Registry jurisdiction/revision, valid time, ownership/control type/percentage, quality challenge | Stale/incomplete record or LEI treated as BO proof |
| Graph/feature platform | Include for repeated analysis | Typed temporal edges, construction/job version, truncation/coverage, source refs | Guilt by association, opaque embeddings, unbounded cost |
| Data/search/stream platform | Include behind source semantics | Topic/index/table identity, offset/snapshot, schema, coverage, correction, ordering and source watermark | Broker offset/PIT/query success treated as authoritative case completeness |
| Rule/model registry | Include for reproducibility | Immutable rule/model/feature/calibration artifact, lineage, inference ID and effective deployment | Mutable alias/tag or current threshold used as historical identity |
| Filing/FIU gateway | Only behind decision/effect plane | Current schema, exact payload hash, credential owner, receipt/status/correction | Direct browser automation, timeout/duplicate, confidentiality |
| Payment/account control | Only behind separate authority/effect plane | Exact object/action/state/legal basis/duration/approval/status query | Overbroad action, stale approval, noncompensable error |
| Notification provider | Only in separately authorized communication process | Recipient/purpose/template/channel, provider ID, callback verification, delivery status, cancellation and confidentiality | Agent/customer tipping-off, delivery state treated as human understanding |
| General browser/SQL/shell/email | Reject from reasoning plane | None | Injection, exfiltration, arbitrary authority and irreproducible reads |

Common envelope fields: contract/schema version, request/run/case/case-version, tenant, actor, purpose, jurisdiction profile, operation, resource scope, deadline/budgets, cursor, source snapshot, data classification, trace ID; results add source revision, event/observed time, stable record IDs/hashes, coverage/completeness, pagination, transformation, freshness, warnings and provenance.

## Jurisdiction and source findings

### Risk-based approach and CDD

FATF's current Recommendations page is the canonical update point and identifies the Recommendations as amended June
2026. February 2025 changes increased focus on proportionality and lower-risk simplified measures. FFIEC CDD guidance
emphasizes understanding customer relationships and ongoing monitoring within U.S. BSA/AML expectations. U.S. agencies
have also said no customer type is uniformly high risk. Implementation implication: customer type, country, PEP, or
model risk score cannot become automatic guilt or blanket exit; retain contributors, current/decision-time evidence,
and a separate authorized decision.

### Suspicious-activity reporting

FFIEC's SAR section describes judgment and documentation of filing/non-filing decisions, not determination of the underlying crime. FinCEN's current FAQs and e-filing specifications establish U.S.-specific reporting and technical processes. Implementation implication: filing/disposition is a process outcome, the narrative is a versioned draft, the decision is human/accountable, the gateway is separately credentialed, and evaluation cannot use filed/not-filed as uncontested labels.

### Sanctions

OFAC's FAQ 5 distinguishes potential and valid matches and tells users to compare full entry details and programs, not
only a name. OFAC's Sanctions List Service is the current primary application for OFAC list files/data, including
official full/consolidated/customized data and archived deltas; prior XML namespace/schema changes demonstrate parser
drift risk. OFAC ownership guidance means named-list screening may not cover entities subject through ownership. The UN
consolidated list spans regimes without implying identical measures. FATF amended Recommendation 6 in June 2026.
Implementation implication: use authority/program/entry/list-revision-specific sources, full-population reconciliation,
deterministic temporal ownership paths, exact transaction state, and a separate time-critical decision/effect workflow.

### Information sharing and privacy

FATF's 2026 information-sharing publication emphasizes enabling legal frameworks and data-protection arrangements. FinCEN's 2025 cross-border guidance is U.S.-specific and does not erase SAR confidentiality. GDPR principles, where applicable, create purpose/minimization/accuracy/storage/security/accountability constraints, while the EDPB notes AI-model privacy conclusions are case-specific. Implementation implication: relevance is not authorization; enforce participating legal entity, purpose, jurisdiction, confidentiality, transfer, onward-use, retention, rights/exceptions, and access at every source/projection/export.

### Evolving EU framework

The European AML package and AMLA operational framework are still developing. AML/CFT functions moved from EBA to
AMLA on 2026-01-01, while existing EBA instruments remain until AMLA replaces them. AMLA states that direct supervision
begins in 2028 and its regulatory-instrument register was updated in July 2026. Implementation implication: maintain
owned effective-dated sources and refreshable jurisdiction profiles rather than hard-code a static “EU AML” rule set.

## State, context, compaction, and memory decisions

| Topic | Decision | Verification |
|---|---|---|
| Durable state | Case store is authoritative; evidence content-addressed; decisions/effects separate | Kill provider/runtime and resume from state only |
| Events | Typed, versioned, idempotent, actor/policy/provenance-bound; optimistic concurrency | Duplicate/out-of-order/concurrent application tests |
| Plan | Completion-policy-bound steps, typed tools/scopes, hard budgets, explicit replan/stop triggers | Trajectory and scope-widening tests |
| Context | Fresh purpose-aware projection with inclusion/exclusion/version/hash manifest | Golden/counterfactual context compiler tests |
| Canonical lifetimes | Exactly seven: turn/scratch, working/run, session, durable workflow/task, domain knowledge, long-term/preference and episodic/outcome | Each has use/reject, retention/deletion, correction/poisoning and evaluation tests; no implicit eighth cache/index lifetime |
| Vector/index/cache | Derived aid keyed by tenant/purpose/source/policy version; not truth | Permission/deletion/source-correction and cross-tenant canary tests |
| Compaction | Provider output is opaque continuation only; application emits typed continuity artifact | Repeated before/after semantic equivalence and adversarial long-case tests |

The typed continuity record retains receipt/schema version, input/output hashes, case/version, tenant/purpose,
subject/time scope, source-event and per-source high-watermarks, source/list snapshots, material claims/citations,
hypotheses, contradictions, benign alternatives, gaps, exact behavior/policy/rule/model/tool/adapter/graph/schema pins,
decisions/approvals, active clocks, pending/unknown effects, omitted-item references, budgets, invariant hash and next safe
action. Resume reauthorizes, verifies receipt/digests and reloads/refetches authoritative state before work continues.

## Planning and completion decision

Use a fixed workflow with a short hypothesis loop:

1. frame subject, scope, time, jurisdiction, alert basis, required sources, deadline, authority, and budget;
2. reproduce the trigger and retrieve typed evidence;
3. verify identity, coverage, versions, transformations, contradictions, and applicability;
4. compare the leading hypothesis with benign alternatives using discriminating queries;
5. replan only for material new/contradictory evidence, source failure, case-version or jurisdiction change;
6. stop on completion policy, low marginal value, budget, ambiguity, forbidden authority, source gap, harm, or deadline;
7. produce a cited review package with unresolved uncertainty, never a verdict.

Completion is alert-family-specific and deterministic. “The model is confident” is never a completion rule.

## Effects and recovery decision

Every filing/restriction/disclosure operation has a server-derived semantic key, immutable request hash, exact target/action, decision/evidence/policy/jurisdiction versions, eligible approvals and expiry, outbox state, dispatch attempt/client ID, receipt, postcondition, and reconciliation state.

Unknown external outcomes are not retryable until authoritative lookup proves non-commit. Reversal is not assumed: filings, disclosures, blocks/freezes, and account actions may need a separate correction, release, legal, or remediation workflow. Disaster recovery uses fencing/single-writer effect dispatch and reconciles all in-flight work after failover.

## Security, privacy, fairness, and governance decision

Hard controls:

- model input/output and source content are untrusted;
- no generic browser, SQL, shell, email, message, filing, restriction, or admin tool;
- principal, workforce eligibility, tenant/legal entity, purpose, jurisdiction, resource, field, action, case version, and context are enforced server-side;
- prompts, provider state, caches, indexes, traces, evaluations, exports, queues, dead letters, screenshots, support and backups are in the data lifecycle;
- SAR/STR existence/content is confined to eligible workflows and never becomes general customer memory;
- secrets are isolated to brokers/effect workers, short-lived where feasible, rotated and revocable;
- anti-tipping-off/SAR-STR confidentiality and customer communication are isolated; requester, reviewer, approver,
  dispatcher, administrator and independent tester separation is deterministic where required;
- supplier/subprocessor, SDK/parser/normalizer/list/model/data artifact and build provenance is pinned in the behavior
  release; unsigned/unreviewed drift disables the affected capability;
- fairness/burden is measured from source coverage and alerting through entity matching, agent/human behavior, requests, delays, holds, restrictions, corrections and exits;
- protected-attribute inference for evaluation is prohibited unless specifically lawful and governed;
- independent validation/testing challenges source-to-decision/effect lineage, permissions, recovery, labels, limitations and correlated errors.

The 2026 Federal Reserve SR 26-2 model-risk guidance explicitly excludes generative and agentic AI from its scope. Its conceptual-soundness, validation, outcomes-analysis, monitoring, governance and effective-challenge ideas are therefore used as an engineering analogy, not misrepresented as directly applicable agent regulation.

## Evaluation decision

### Three independent gates

- **Outcome:** citation/entailment, source coverage, contradictions, benign alternatives, entity match quality, investigator usefulness/correction time, timeliness, unnecessary escalation and downstream burden.
- **Policy:** zero cross-tenant/purpose/confidentiality, unsupported-fact, forbidden-action, stale-authority, secret, duplicate/unknown-effect and injection/exfiltration violations.
- **Repeated reliability:** distributions and worst cases across multiple stochastic runs, compactions/resumes, concurrency, source/model/provider/destination failures, load, DR and human capacity.
- **Human factors:** calibrated blinded reviewer agreement, severe-error recall, automation bias, fatigue, override/edit
  behavior, downstream burden and adjudicated dissent.

### Evidence portfolio

Use deterministic contracts; expert-authored cases; time-correct governed historical replay; benign controls; synthetic/injected typologies; adjudicated entity/sanctions candidates; adversarial security; fault-injected workflow/effects; proposal-only shadow traffic; and post-release independent samples. Protect cutoff and prevent later outcome/disposition/narrative leakage. Public AML datasets such as Elliptic and synthetic tools such as AMLSim are exploratory only.

### Failure injection

Required cases include partial pagination, stale/missing source/list update, correction/reversal mid-run, entity false merge/split, graph truncation, malicious note/document/tool result, fabricated evidence, compaction loss, concurrent edit, cancellation, crash at every effect boundary, lost acknowledgment, destination timeout after commit, duplicate/reordered queues, provider outage, reviewer backlog, credential/policy revocation and regional failover.

## Observability, SLO, capacity, and cost decision

Case/regulated records, evidence/provenance, control audit, operational logs, metrics and traces are separate planes with
different sampling, content, access and retention. Traces correlate case/run/behavior/model/context/tool/source/policy/
effect identities but contain redacted metadata by default. OpenTelemetry generative-AI conventions are still in
development and must be version-pinned if used. None of these telemetry conventions supplies authority or evidence.

Minimum SLO areas: evidence completeness, critical-source/list freshness, time to first reviewable package, queue age/deadline risk, material-claim integrity, zero unauthorized effects/disclosures, unknown-effect reconciliation age, effect postcondition verification, compaction/resume continuity, qualified-review capacity/quality, evaluation/audit/trace health, and cost per verified case outcome.

Capacity planning includes alerts after dedupe, source rate limits, graph/features, model tokens/latency, state/evidence/
audit storage, investigators by skill/jurisdiction/language/shift, approval segregation, effect workers, reconciliation,
QA, security/privacy/legal and incident response. Recovery load separately includes unprocessed events/list deltas,
source/index/graph reconciliation, pending callbacks, unknown effects, expired approvals, active clocks and record
verification. Safe overload sheds or pauses assisted work and preserves evidence/deadlines; it never auto-approves or
converts unavailable evidence to “none.”

Total cost includes data/provider/storage/graph/model, human review, QA/validation/security/privacy/operations, effects/reconciliation, corrections/incidents/remediation, and false-negative/false-positive burden. Optimize deterministic dedupe and preprocessing, narrow retrieval, stop rules, typed context, validated model routing, safe batching/cache, and reviewer UI before adding orchestration.

## Stage 0–6 researched exit gates

| Stage | Required evidence before exit |
|---:|---|
| 0 — qualify | Named outcome and owners; category/legal boundary; deterministic baseline; source/label/privacy/authority feasibility; explicit agent justification and rejection conditions |
| 1 — bounded investigator | One alert family; read-only typed tools; trigger reproduction; cited claims; benign alternative; gaps/contradictions; budgets/stop; human review; no effects; initial offline eval |
| 2 — MVP | Real case/auth projection; versioned typology and context manifest; ephemeral working memory only; representative sources/cases/slices; investigator UI; proposal schema; shadow/pilot; measured value |
| 3 — reliable v1 | Durable case/events; provenance; optimistic concurrency; compaction/resume tests; connector contracts; cancellations; semantic idempotency; effect ledger/reconciliation even if effects remain disabled; recovery and failure injection |
| 4 — production | Strong identity/tenant/purpose; privacy/confidentiality/security review; behavior manifest; full eval and independent challenge; trace/audit/SLOs; capacity/human staffing; canary/rollback/kill switches; incident and DR runbooks |
| 5 — scale/resilience | Partition/isolation evidence; bounded queues/backpressure/degradation; realistic load/failover; list/source/provider outages; effect fencing/reconciliation; human capacity; cost per verified outcome; recovery objectives met |
| 6 — governed evolution | Failure mining; drift, slice and burden monitoring; legal/source refresh; change gates for every behavioral input; shadow/canary/rollback; poisoned data/memory response; lookbacks; deprecation/migration; recurring independent review |

Authority never expands automatically with stage. A production agent remains a proposer unless a separate legal and control decision defines a narrow deterministic effect path; this blueprint keeps model-directed filing/freeze/offboarding prohibited.

## Contradictions and trade-offs

| Tension | Resolution selected |
|---|---|
| Latest data/list/law vs reproducibility | Preserve decision-time snapshot and lineage; current deterministic policy can invalidate/reopen/rescreen before effects |
| Full network context vs purpose/minimization | Typed bounded queries and referenced evidence; no universal graph dump or memory |
| Human authority vs automation bias/fatigue | Evidence-first UI where feasible, source/contradiction links, meaningful edit/dissent, blind QA, capacity SLOs |
| Risk-based proportionality vs mandatory sanctions rule | Apply proportionality to discretionary risk measures; separate exact legal sanctions path |
| Enterprise/network sharing vs SAR confidentiality/privacy | Approved sharing topology by legal entity, purpose, jurisdiction and data class; never relevance-only sharing |
| Agent consistency vs investigator judgment | Agent standardizes evidence assembly; human decision/dissent remains explicit and evaluated |
| Pinned active-case behavior vs emergency correction | Default pin; security/legal/quality emergency can stop, quarantine, recompile or require re-review under owned rule |
| Compaction efficiency vs evidentiary completeness | Typed continuity artifact and source handles; compacted prose never authoritative |
| Graph/ML novelty vs calibration/reviewability | Deterministic features and contestable candidates first; learned methods require independent incremental evidence |
| More agents vs throughput | Deterministic parallel workers; add model agents only after isolated incremental eval and authority design |
| Faster case packages vs reviewer deadlines/capacity | Admission/backpressure uses end-to-end capacity; model throughput cannot outrun meaningful review |
| Kafka exactly-once vs external case/effect correctness | Kafka transactions cover configured Kafka processing; case databases and external effects still need semantic idempotency, postconditions and reconciliation |
| Official-list current view vs reproducible match | Preserve exact authority/list/program/entry/publication/dataset revision and parser; reconcile current full population and retain decision-time snapshot |
| Digital identity result vs KYC/CDD conclusion | Identity-proofing/authentication evidence is one sourced input; institution policy and accountable review own KYC/CDD and beneficial-ownership conclusions |
| Model-registry alias vs reproducible inference | Resolve alias/tag to immutable artifact, features, threshold and deployment at use; never cite mutable alias alone |
| Provider delivered/complete/accepted vs business outcome | Persist narrow provider receipt, then verify separately authorized case, filing, restriction or communication postcondition |
| Infrastructure restore vs operational recovery | Restore authority/lists/clocks/effect ledger first; throttle redrive until sources, records, unknown effects and reviewer backlog converge |

## Known limitations

- Laws, filing thresholds/deadlines, confidentiality, sanctions effects, data sharing, retention, and legal privilege vary by jurisdiction and institution; this packet does not supply legal advice or universal values.
- Current official pages and list/schema/provider behavior can change after the research date.
- Financial-crime outcomes are rare, delayed, censored, selected, and influenced by controls; no evaluation can prove absence of laundering/fraud.
- Historical cases reproduce earlier monitoring and human selection; synthetic/public datasets do not reproduce production base rates or data gaps.
- Entity resolution, transliteration, beneficial ownership, device linkage, adverse media, and graph association remain fallible and contestable.
- Prompt injection and poisoned content are not solved; controls reduce reach and impact.
- Human review can be inconsistent, overloaded, biased, or automated in name only.
- Some external effects are irreversible or lack strong idempotency/status APIs; such integrations may require manual execution.
- No live ledger/warehouse/stream, case, OFAC/list vendor, KYC/identity, graph, model registry, filing, payment-control or
  notification tenant/configuration was tested or qualified by this research.
- Named platforms illustrate official documented behavior, not procurement suitability, GxP/compliance validation,
  availability in a plan/region, production performance or complete supplier assessment.
- Provider model behavior, context/compaction, retention and tracing may evolve; exact behavior needs release-time verification.
- Publicly documented production evidence for LLM-directed AML investigations remains limited; this blueprint therefore favors constrained architecture and measured rollout over autonomy claims.

## Refresh triggers

Refresh immediately for a FATF Recommendation or national law/rule change; AMLA/FIU/FinCEN filing schema, FAQ or
deadline change; OFAC/UN/EU/national list format, namespace/parser, program, ownership or effect change; provider/API/
tenant configuration, plan, region, role, fields, limits, receipt, Beta/GA/deprecation or support change; new legal
entity/jurisdiction/rail/list/source/provider/model/language/alert family/memory/effect; model/provider data-use/
retention/compaction/tool change; material data/identity/graph drift; confidentiality/security incident; false-negative
cluster; mass false positives or subgroup burden; unknown/duplicate effect; evaluator leakage/contamination; or
capacity/deadline/recovery-load failure.

Otherwise re-check volatile integrations, provider settings, official sources, access, drift, costs and failure evidence at least every 90 days, and the entire workload/architecture/privacy/threat/memory/DR decision at least every 180 days. These are engineering defaults; applicable rules may demand more frequent review.

## Source ledger

All sources below were accessed or current-status checked on 2026-08-31 unless noted. “Implication” is this packet's synthesis, not a quotation.

### Primary AML, sanctions, and regulatory sources

| Source | Current finding used | Blueprint implication |
|---|---|---|
| [FATF Recommendations — current publication page](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Fatf-recommendations.html) | Canonical current standards/update point; amended June 2026 | Versioned legal-source inventory and jurisdiction profiles |
| [FATF — 2025 proportionality and financial-inclusion changes](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/update-standards-promote-financial-conclusion-feb-2025.html) | Greater proportionality and simplified measures in lower-risk scenarios | Prevent blanket high-risk treatment and agent-driven de-risking |
| [FATF — 2026 Recommendation 6 update](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/update-recommendation-6-june-2026.html) | Updated targeted-financial-sanctions standard concerning humanitarian exemptions | Current program/profile semantics and immediate refresh trigger |
| [FATF — Beneficial Ownership of Legal Persons](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Guidance-Beneficial-Ownership-Legal-Persons.html) | Need for adequate, accurate, up-to-date BO information and multi-source approaches | Temporal sourced ownership graph; unknowns and source quality visible |
| [FATF — Information Sharing to Combat Illicit Finance (2026)](https://www.fatf-gafi.org/en/publications/Methodsandtrends/information-sharing-ppp-data-protection-arrangements.html) | Information sharing depends on legal frameworks and data-protection arrangements | Sharing topology enforced by law/entity/purpose; no universal memory |
| [FATF — Opportunities and Challenges of New Technologies for AML/CFT](https://www.fatf-gafi.org/en/publications/Digitaltransformation/Digital-transformation.html) | Technology/data collaboration can improve AML/CFT but creates privacy/data-protection risks | Pair analytical value with data lifecycle and rights controls |
| [FFIEC — Customer Due Diligence](https://bsaaml.ffiec.gov/manual/AssessingComplianceWithBSARegulatoryRequirements/02_ep) | U.S. CDD/ongoing-monitoring examination context | Source-backed current/decision-time CDD with monitoring lineage |
| [FFIEC — Suspicious Activity Reporting](https://bsaaml.ffiec.gov/manual/AssessingComplianceWithBSARegulatoryRequirements/04) | Judgment-based filing decisions and rationale documentation; no need to establish underlying crime | Human decision contract; filing/disposition not truth label |
| [FFIEC — Independent Testing](https://bsaaml.ffiec.gov/manual/AssessingTheBSAAMLComplianceProgram/03_ep) | Independent testing is a compliance-program component for covered institutions | Independent challenge and source-to-effect reproduction |
| [FinCEN — Suspicious Activity Reporting Requirements FAQs (2025)](https://www.fincen.gov/resources/statutes-regulations/guidance/frequently-asked-questions-regarding-suspicious-activity) | Issued 2025-10-09 | Profile-specific filing process and refresh ownership |
| [FinCEN — SAR FAQs](https://www.fincen.gov/resources/frequently-asked-questions-regarding-fincen-suspicious-activity-report-sar) | Filing and confidentiality/supporting-process guidance | Confidential case/effect path rather than general customer record |
| [FinCEN — BSA E-Filing filing information](https://bsaefiling.fincen.gov/filing-information) | Current filing specifications and technical materials | Pinned schema, deterministic validation, credentialed adapter |
| [FinCEN — SAR supporting documentation](https://www.fincen.gov/resources/statutes-regulations/guidance/suspicious-activity-report-supporting-documentation) | Supporting documents can be requested under defined authority/process | Separate verified request/disclosure workflow |
| [FinCEN — E-mail compromise advisory](https://www.fincen.gov/resources/statutes-regulations/guidance/advisory-financial-institutions-e-mail-compromise-fraud) | Red flags do not necessarily indicate suspicious activity | Indicator/typology is not proof; require contextual investigation |
| [FinCEN — cross-border information sharing guidance (2025)](https://www.fincen.gov/system/files/2025-09/Crossborderguidance-508C.pdf) | U.S.-specific cross-border guidance with SAR-confidentiality constraints | Explicit allowed sharing paths and confidentiality boundaries |
| [OFAC FAQ 5](https://ofac.treasury.gov/faqs/5) | Potential match requires comparison of full details/program, not name alone | Structured potential-match package and qualified decision |
| [OFAC FAQs](https://ofac.treasury.gov/faqs/all-faqs) | Current program/ownership and operational guidance inventory | Official-source snapshots; deterministic ownership/effect policy |
| [OFAC Sanctions List Service](https://ofac.treasury.gov/sanctions-list-service) | Current primary OFAC list-data service with SDN, non-SDN, custom data and archived deltas | Pin full/delta publication and entry identities; full-population reconciliation |
| [OFAC SLS XML namespace/schema change](https://ofac.treasury.gov/recent-actions/20240507_44) | Official example of list parser contract changing | Schema validation, parser fixture, quarantine and affected-case rescreen |
| [OFAC Framework for Compliance Commitments](https://ofac.treasury.gov/recent-actions/20190502_33) | Risk-based sanctions compliance framework | Sanctions governance remains separate from AML narrative reasoning |
| [UN Security Council Consolidated List](https://main.un.org/securitycouncil/en/content/un-sc-consolidated-list) | Entries span distinct regimes and do not imply uniform measures | Store regime/program and measure semantics, not generic boolean |
| [European Commission — sanctions resources](https://finance.ec.europa.eu/eu-and-world/sanctions-restrictive-measures/overview-sanctions-and-related-resources_en) | Current EU sanctions resource hub | EU-specific source/profile ownership and refresh |
| [European Commission — EU AML/CFT framework](https://finance.ec.europa.eu/financial-crime/anti-money-laundering-and-countering-financing-terrorism-eu-level_en) | Current package and institutional framework | Avoid static assumptions; map applicable institution and dates |
| [AMLA FAQs](https://www.amla.europa.eu/faqs_en) | AMLA's evolving operations and direct-supervision timeline | Refresh trigger and legal/governance owner |
| [AMLA regulatory instruments](https://www.amla.europa.eu/policy/regulatory-instruments_en) | Ongoing instrument publication | Controlled legal-source ingestion; no automatic prompt update |
| [EBA–AMLA handover](https://www.eba.europa.eu/publications-and-media/press-releases/eba-and-amla-complete-handover-amlcft-mandates) | AML/CFT functions transferred on 2026-01-01; existing EBA instruments remain until replaced | Effective-dated source ownership and continuity during regulatory transition |
| [UK NCA — Suspicious Activity Reports](https://www.nationalcrimeagency.gov.uk/what-we-do/crime-threats/money-laundering-and-illicit-finance/suspicious-activity-reports) | SAR is not a crime report; UK reporting/tipping-off context | Jurisdiction-specific terminology, confidentiality and reporter process |
| [FDIC — risk-based customer relationships](https://www.fdic.gov/news/financial-institution-letters/2022/fil22028.html) | No customer type is uniformly high risk; agencies do not direct account decisions | No automatic customer-class exit or universal risk inference |
| [Federal Reserve — Model Risk Management SR 26-2 (2026)](https://www.federalreserve.gov/frrs/guidance/supervisory-guidance-on-model-risk-management.htm) | Validation/governance principles, while generative/agentic AI is expressly outside scope | Use principles only by analogy; state scope limitation clearly |

### Data, agent, security, evaluation, and operations sources

| Source | Finding used | Blueprint implication |
|---|---|---|
| [ISO 20022 e-Repository](https://www.iso20022.org/iso20022-repository/e-repository) | Versioned message definitions | Pin rail/message schema and retain originals |
| [W3C PROV Overview](https://www.w3.org/TR/prov-overview/) | Entities, activities and agents support interoperable provenance concepts | Typed source-transform-claim-decision-effect lineage |
| [GLEIF — Challenge LEI and vLEI Data](https://www.gleif.org/en/lei-data/gleif-data-quality-management/challenge-lei-and-vlei-data) | LEI data quality can be challenged/corrected | LEI is sourced temporal evidence, not immutable identity/BO truth |
| [NIST SP 800-63-4](https://www.nist.gov/publications/nist-sp-800-63-4-digital-identity-guidelines) | Final 2025 identity-proofing, authentication and federation guidance | Qualify identity evidence while keeping KYC/CDD and workforce authority separate |
| [Apache Kafka 4.3.1 release](https://kafka.apache.org/blog/2026/06/25/apache-kafka-4.3.1-release-announcement/) and [delivery semantics](https://kafka.apache.org/42/design/design/) | Current release evidence plus official warning about exactly-once scope and external systems | Pin installed version; offsets/transactions do not replace case/effect idempotency and reconciliation |
| [OpenSearch point-in-time search](https://docs.opensearch.org/latest/search-plugins/searching-data/point-in-time/) and [audit logs](https://docs.opensearch.org/latest/security/audit-logs/index/) | PIT offers fixed search view; audit logging is configurable and disabled by default | Search projection needs source watermark; operational audit config is not case/evidence truth |
| [Neo4j transaction behavior](https://neo4j.com/docs/operations-manual/current/database-internals/) and [bookmarks](https://neo4j.com/docs/query-api/current/bookmarks/) | Default read-committed can allow nonrepeatable reads; bookmarks support causal ordering | Pin query/snapshot/construction version and source coverage; graph path remains derived |
| [MLflow Model Registry workflows](https://www.mlflow.org/docs/latest/ml/model-registry/workflow/) | Model versions have lineage/signatures; aliases and tags organize deployment | Resolve mutable aliases to immutable model/feature/calibration identity at use |
| [Twilio Message resource](https://www.twilio.com/docs/messaging/api/message-resource) | Channel-specific queued/sent/delivered states, rate queues and evolving callbacks | Delivery receipt has narrow meaning; bind recipient/purpose/template and prevent tipping-off |
| [OpenAI — latest model guidance](https://developers.openai.com/api/docs/guides/latest-model) | Current provider guidance favors clear tool boundaries, state/context discipline and evals | Keep vendor-specific runtime assumptions pinned and evaluated |
| [Anthropic — Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) | Start with simplest workflow; agents trade latency/cost for flexibility | Deterministic baseline and one bounded loop before orchestration |
| [Anthropic — Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | Curated context and durable external state matter for long tasks | Application context compiler and typed continuity artifact |
| [Anthropic — Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) | Agent evaluation requires outcome and trajectory evidence across realistic tasks | Portfolio, multiple trials, graders, shadow and failure mining |
| [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework) | Voluntary risk-management lifecycle and ongoing update work | Govern/map/measure/manage across whole released system |
| [NIST — Agent Hijacking Evaluations](https://www.nist.gov/news-events/news/2025/01/technical-blog-strengthening-ai-agent-hijacking-evaluations) | Indirect prompt injection needs stronger evaluation | Malicious source/tool/memory test suites and impact containment |
| [NIST — agents cheating evaluations](https://www.nist.gov/caisi/cheating-ai-agent-evaluations) | Agents may detect/manipulate evaluation conditions | Hidden/rotating/prod-like evaluation plus shadow/post-release evidence |
| [OWASP Agentic Top 10 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) | Agentic threats include goal/tool/identity/memory/system risks | Layered threat model beyond prompt injection |
| [NIST Privacy Framework](https://www.nist.gov/privacy-framework) | Privacy risk-management framework | Lifecycle record and reassessment triggers |
| [GDPR Article 5](https://eur-lex.europa.eu/eli/reg/2016/679/art_5/oj) | Purpose, minimization, accuracy, retention, security and accountability principles where applicable | Treat every derived/model store as part of lifecycle |
| [EDPB Opinion 28/2024](https://www.edpb.europa.eu/documents/opinion-of-the-board-art-64/opinion-282024-on-certain-data-protection-aspects-related-to_en) | AI-model privacy assessments are fact-specific | No blanket anonymity/legal-basis claim for a provider/model |
| [AWS Builders' Library — idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/) | Client semantic identity makes retries safer | Server-derived effect keys, request-hash conflict, stored outcome |
| [Google SRE — SLOs](https://sre.google/sre-book/service-level-objectives/) | SLOs should reflect user outcomes and drive action | Evidence/deadline/reconciliation/reviewer outcome SLOs |
| [Google SRE — Cascading Failures](https://sre.google/sre-book/addressing-cascading-failures/) | Overload needs backpressure, bounded work and graceful degradation | Capacity-aware admission and safe proposal/manual fallback |
| [W3C Trace Context](https://www.w3.org/TR/trace-context/) | Standard distributed trace correlation | Correlate components while keeping trace separate from audit/evidence |
| [OpenTelemetry GenAI conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) | GenAI semantic conventions remain under development | Pin convention version and minimize captured content |
| [NIST SP 800-61 Rev. 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final) | Current incident-response guidance aligned to cyber-risk management | General incident lifecycle plus AML/confidentiality/effect runbooks |
| [IBM — Elliptic GCN study](https://research.ibm.com/publications/anti-money-laundering-in-bitcoin-experimenting-with-graph-convolutional-networks-for-financial-forensics) | Narrow public research setting for crypto graph classification | Illustration/benchmark only, not production validation |
| [IBM AMLSim](https://github.com/IBM/AMLSim/) | Synthetic AML transaction generator | Pipeline/failure scenarios only; not production base-rate proof |

## Source-quality and disagreement notes

- Regulatory sources define obligations and supervisory context but usually do not specify safe LLM runtime architecture; systems controls are synthesized from primary engineering/security sources.
- Provider engineering articles are useful implementation guidance but not standards, law, or evidence that a pattern is safe in this regulated workload.
- Model-risk frameworks disagree in formal scope. The 2026 Federal Reserve guidance's explicit generative/agentic exclusion is preserved; principles are not falsely presented as direct coverage.
- “Risk-based” is sometimes used to justify stronger controls and sometimes proportionality/simplification. This packet preserves both and separates discretionary risk measures from mandatory sanctions obligations.
- Frameworks often market multi-agent specialization; the research did not find category-specific evidence that it outweighs state, authority, correlated-error, and evaluation costs here. It is rejected until incremental value is measured.
- Public AML/graph research often optimizes classification metrics on constrained data. It does not validate end-to-end case quality, legal compliance, confidentiality, effects, human review, or production drift.

## Documentation coverage map

| Requirement | Canonical guide |
|---|---|
| Workload fit, deterministic alternative, authority, Stage 0–6 gates | [01](../../agents/fraud-aml-investigation-agent/01-workload-fit-authority-and-stages.md) |
| Runtime architecture, integrations, contracts, explicit rejections | [02](../../agents/fraud-aml-investigation-agent/02-reference-architecture-and-integration-contracts.md) |
| Customer/transaction/entity/graph evidence and lineage | [03](../../agents/fraud-aml-investigation-agent/03-customer-transaction-entity-and-network-evidence.md) |
| Alerts/cases/typologies/hypothesis loop, planning and decisions | [04](../../agents/fraud-aml-investigation-agent/04-alert-case-typology-and-investigation-reasoning.md) |
| KYC/CDD/EDD, PEP/adverse media, sanctions and jurisdiction semantics | [05](../../agents/fraud-aml-investigation-agent/05-kyc-cdd-edd-sanctions-and-source-semantics.md) |
| State/events/context/compaction/all memory classes/planning/provenance | [06](../../agents/fraud-aml-investigation-agent/06-state-context-memory-planning-and-provenance.md) |
| Filings/effects/idempotency/reconciliation/cancellation/recovery | [07](../../agents/fraud-aml-investigation-agent/07-filings-effects-idempotency-and-recovery.md) |
| Security/privacy/fairness/governance/independent challenge | [08](../../agents/fraud-aml-investigation-agent/08-security-privacy-fairness-and-governance.md) |
| Evaluation/failure injection/tracing/SLOs/release gates | [09](../../agents/fraud-aml-investigation-agent/09-evaluation-observability-slos-and-failure-injection.md) |
| Deployment/capacity/cost/DR/incidents/behavior releases/evolution | [10](../../agents/fraud-aml-investigation-agent/10-deployment-scale-incidents-cost-and-evolution.md) |
| Operation-qualified data/stream/case/list/KYC/graph/model/notification adapters and worked end-to-end flows | [11](../../agents/fraud-aml-investigation-agent/11-qualified-adapters-and-worked-investigation-flows.md) |

## Final quality record

Before promotion from research packet to maintained guidance, verify:

- [x] Category and adjacent boundaries are explicit.
- [x] Deterministic alternative and reasons to reject the agent are first-class.
- [x] Selected architecture is smaller than the fashionable alternative and supports real waits/recovery.
- [x] Integrations, source semantics, contracts, exclusions, and failure modes are explicit.
- [x] Exactly seven canonical memory lifetimes have explicit use/reject, retention/deletion, poisoning and evaluation controls; indexes/caches are not another lifetime.
- [x] Plans, state, events, provenance, effects, idempotency, unknown outcomes, reconciliation and DR are specified.
- [x] Security, SAR/STR confidentiality, privacy, sharing, fairness and independent governance cover the full lifecycle.
- [x] Evaluation addresses weak labels, leakage, benign controls, trajectories, stochasticity, failure injection, human factors and downstream burden.
- [x] SLOs, capacity, human staffing, cost, release manifests, rollback, incidents, drift and refresh are operational.
- [x] Operation-level adapter manifests, provider receipt limits, worked alert/sanctions/filing/override flows and recovery-load convergence are explicit.
- [x] Stage 0–6 exit gates, contradictions, limitations, current sources, research date and refresh triggers are recorded.

Remaining work is jurisdiction/institution implementation: legal counsel and accountable control owners must replace profile placeholders, test actual source/destination contracts, establish thresholds/deadlines/retention, and collect baseline/evaluation/capacity evidence before any production use.

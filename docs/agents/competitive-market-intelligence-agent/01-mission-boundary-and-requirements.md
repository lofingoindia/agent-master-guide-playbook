# Mission, Boundary, and Requirements

## Mission

The agent's mission is to reduce the time between a permitted public-source change and a well-supported human understanding of that change. It should improve coverage, consistency, freshness, and auditability without turning the organization into an indiscriminate crawler or allowing fluent synthesis to outrun evidence.

The unit of work is an **intelligence run** against an approved brief contract:

```yaml
brief_contract:
  contract_id: ci-weekly-product-market-v3
  tenant_id: tenant-42
  purpose: "Weekly product and market change briefing"
  audience: ["strategy-leadership"]
  watchlist_version: wl-2026-08-31-04
  source_policy_version: sp-2026-08-29-02
  window:
    from: "2026-08-24T00:00:00Z"
    to: "2026-08-31T00:00:00Z"
  questions:
    - "What changed in approved competitor products, positioning, or availability?"
    - "What new official market indicators affect the named market assumptions?"
  evidence_cutoff: "2026-08-31T00:30:00Z"
  materiality_profile: mp-enterprise-product-v2
  output_schema: executive-brief-v4
  allowed_effects: ["save_draft", "request_review"]
  prohibited_effects:
    - "external_publish"
    - "crm_write"
    - "outreach"
    - "trade"
  budgets:
    wall_clock_seconds: 1800
    source_requests: 1200
    model_input_tokens: 300000
    model_output_tokens: 40000
    cost_currency: USD
    maximum_cost: 35
```

The values are examples, not universal targets. Production contracts must derive budgets and materiality from local source cadence, review capacity, risk, and economics.

## Product outcomes

The system should provide four linked products rather than one opaque answer:

1. **Change queue:** normalized, deduplicated candidate events with source time, observed time, entity, rights state, and detector version.
2. **Evidence ledger:** the source representations, locators, integrity data, extraction spans, definitions, calculations, contradictions, and review history supporting each claim.
3. **Analysis package:** accepted and rejected hypotheses, implications, uncertainties, scenario assumptions, and signals to monitor.
4. **Briefing revision:** an audience-specific projection of the package, with stable claim and citation identifiers and a review/publication receipt.

These products can evolve separately. A new briefing format should not rewrite evidence; a corrected entity merge should be replayable without pretending the original run saw the corrected identity; a revoked source permission should be enforceable across retained snapshots, derived artifacts, evaluations, and future publications.

## Stakeholders and decisions

| Stakeholder | Uses the output to | Must not delegate to the agent |
|---|---|---|
| Intelligence analyst | verify changes, enrich evidence, test competing explanations | source ethics, material-claim approval, or disposition of unresolved conflicts |
| Strategy or product leader | understand implications and decide what to investigate or change | strategy adoption or resource commitment |
| Legal/privacy/data governance | approve source, purpose, personal-data, retention, quotation, and redistribution policies | context-specific legal judgment |
| Platform/operator | run connectors, workflows, storage, evaluation, and incident controls | interpreting business materiality |
| Source owner | maintain authentication, quota, schema, and terms metadata | authorizing use outside the registered policy |
| Auditor or risk reviewer | reconstruct the run and verify controls | accepting traces as authoritative business records |

## Category separation

Competitive intelligence becomes unreliable when adjacent workloads are silently folded into it.

| Request | Owning category | Handoff contract |
|---|---|---|
| “Answer this bounded question using a broad source search” | [Deep research](../deep-research-agent/README.md) | Provide a research question, deadline, evidence requirements, and no persistent watchlist unless separately approved. |
| “Continuously monitor these named organizations and markets” | Competitive and market intelligence | Approved watchlist, source policy, cadence, materiality, and audience. |
| “Update the opportunity and email the account owner” | [Sales and revenue operations](../sales-revenue-operations-agent/README.md) | Hand off an approved intelligence artifact; no implicit CRM write or outreach. |
| “Find an internal policy or answer from enterprise documents” | [Enterprise knowledge](../enterprise-knowledge-agent/README.md) | Use ACL-aware internal retrieval; do not copy internal evidence into public-source stores. |
| “Recommend or execute a securities trade” | Out of scope | Use a separately governed financial advisory/trading system if lawful and authorized. |

The same source may participate in more than one workload, but the purpose, allowed use, retention, and audience must be evaluated for each workload. “Already downloaded” is not a reusable authorization.

## Functional requirements

### Watchlist and identity

- Represent organizations, legal entities, brands, products, market concepts, geographies, topics, and source endpoints separately.
- Use stable registry identifiers where available and preserve issuer/source identifiers without treating fuzzy-name results as truth.
- Maintain aliases, parent/subsidiary relationships, valid-time ranges, confidence calibration, merge/split history, and unresolved candidates.
- Version watchlist additions, removals, materiality profiles, source assignments, and reviewer ownership.
- Require human approval for ambiguous merges, new person-centered targets, high-risk jurisdictions, or materially broader collection.

### Source governance

- Gate access by source, endpoint/distribution, purpose, tenant, jurisdiction, credentials, allowed methods, rate policy, retention, transformation, quotation, attribution, redistribution, personal-data handling, and review date.
- Prefer official APIs, feeds, bulk files, and licensed channels over page scraping.
- Respect access controls and approved robots/terms policies; never bypass authentication, paywalls, CAPTCHA, technical restrictions, or rate limits.
- Re-evaluate policy when terms, license, access method, audience, purpose, data fields, or storage behavior changes.
- Stop and quarantine downstream reuse when source rights expire or become indeterminate.

### Collection and normalization

- Use conditional HTTP requests and provider cursors where supported; preserve response timestamps, validators, schema/taxonomy versions, and collection receipts.
- Treat feeds and webhooks as hints requiring reconciliation, not proof of complete delivery.
- Record `published_at`, `effective_at`, `valid_from`, `valid_to`, `retrieved_at`, and `observed_at` separately when available.
- Normalize units, currencies, periods, definitions, geographies, fiscal calendars, and revisions without destroying the source representation.
- Quarantine parser failures and layout/schema drift instead of interpreting missing fields as a real-world change.

### Change and evidence

- Detect byte, structural, field, numerical, and semantic changes with versioned detectors.
- Distinguish new information, correction, restatement, retraction, temporal supersession, definition drift, entity mismatch, and duplicate syndication.
- Preserve a stable evidence locator, content integrity value, extraction span, transformation lineage, and permitted-use state.
- Cluster sources by likely dependence; five syndicated copies must not be reported as five independent confirmations.
- Require stronger evidence and review for higher materiality rather than using one confidence threshold for every claim.

### Analysis and briefing

- Label observation, calculation, inference, hypothesis, scenario, forecast, recommendation supplied by a human, and decision.
- Show counterevidence, missing evidence, freshness, entity ambiguity, source dependence, and definitional disagreement.
- Express scenarios as conditional paths with explicit assumptions and observable triggers.
- Use probabilities only when the event is operationally resolvable, the forecaster is named, and calibration will be scored.
- Produce audience-bounded drafts, not strategy decisions or public statements.
- Preserve stable claim IDs across revisions so edits and reviewer dispositions are measurable.

### Operations and evaluation

- Maintain authoritative run state, domain events, effect intents/receipts, and telemetry as different records.
- Support cancellation, deadlines, hard budgets, retries, reconciliation, quarantine, replay, and kill switches.
- Evaluate frozen historical snapshots, current shadow traffic, adversarial source content, and controlled failure injection.
- Make rights violations, cross-tenant exposure, unsupported material claims, and unauthorized publication hard-stop release failures.
- Track source freshness, detection lag, change precision/recall, duplicate alerts, evidence support, contradiction recall, reviewer edits, briefing timeliness, reconciliation backlog, cost, and incident rates.

## Non-functional requirements

| Quality | Requirement | Evidence of compliance |
|---|---|---|
| Correctness | Numerical values retain units, period, definition, source revision, and calculation lineage. | Deterministic fixtures and calculation tests. |
| Freshness | Every source class has an expected cadence and explicit stale behavior. | Freshness coverage and overdue-source report. |
| Security | Source content is untrusted data, not executable instruction; tenant isolation applies to every store and cache. | Adversarial tests and access-control audit. |
| Privacy | Collect the minimum personal data necessary for the approved organizational intelligence purpose. | Data inventory, purpose record, retention/deletion tests, and privacy review. |
| Reliability | Duplicate delivery, process loss, partial effects, schema drift, and quota exhaustion do not create silent success. | Fault-injection and recovery tests. |
| Auditability | A reviewer can reproduce what the run knew, which versions it used, and what was delivered. | Release manifest, evidence graph, state/event records, and delivery receipt. |
| Maintainability | Connectors and schemas are versioned behind typed contracts; source-specific quirks do not leak into briefing logic. | Contract tests and adapter ownership. |
| Cost control | Work is bounded and model use is focused on changed, policy-approved evidence. | Per-run budget enforcement and cost attribution. |
| Accessibility | Briefs communicate uncertainty with text and tables rather than color alone. | Document/template review. |

## Threat and failure assumptions

Assume all of the following occur in ordinary operation:

- a watched page contains text instructing an AI system to ignore policy;
- an official endpoint changes schema or taxonomy without matching the business cadence;
- a provider returns `429`, partial data, or a transient success with stale contents;
- the same announcement arrives through a feed, press release, filing, and syndication network;
- a corporate reorganization invalidates aliases and parent relationships;
- an older source appears newer because it was re-indexed or copied;
- a statistic is revised and the latest-only API no longer exposes the previous value;
- access or redistribution rights change after evidence was captured;
- a notification returns no definitive receipt;
- a model upgrade changes entity mapping, materiality, or support behavior;
- a user asks for personal details, covert methods, or unapproved dissemination under the label “public information.”

Designing only for the happy path creates a persuasive newsletter, not an intelligence system.

## Materiality contract

Materiality is local and versioned. A defensible profile combines deterministic triggers with reviewable model classification:

```yaml
materiality_profile:
  profile_id: mp-enterprise-product-v2
  deterministic_triggers:
    - field: product.availability
      transition: ["available", "withdrawn"]
      minimum_level: high
    - field: reported_metric.value
      relative_change_gte: 0.10
      minimum_level: medium
  semantic_dimensions:
    - customer_impact
    - market_scope
    - reversibility
    - source_authority
    - strategic_relevance
  review_rules:
    high: "two-person material-claim review, or approved primary-source exception"
    medium: "analyst review"
    low: "may enter digest if source and citation controls pass"
  abstain_when:
    - entity_ambiguous
    - rights_indeterminate
    - definition_changed
    - required_source_stale
```

The model returns dimension labels, supporting evidence IDs, and an abstention reason. Policy computes the resulting workflow. The model cannot raise its own publication authority by assigning “low risk.”

## Acceptance criteria for the first bounded release

Do not call the system production-ready until one bounded watchlist demonstrates:

- all sources are registered and approved for collection, retention, analysis, evaluation reuse, and intended briefing audience;
- canonical entity mappings and ambiguity workflows exist;
- frozen fixtures cover unchanged, modified, deleted, revised, duplicated, contradictory, stale, and malformed inputs;
- every briefing sentence is typed and material factual claims resolve to valid evidence;
- failure to fetch, parse, normalize, resolve, analyze, or publish appears as an explicit state;
- retries and duplicate inputs do not create duplicate briefs or notifications;
- ambiguous delivery is reconciled rather than blindly retried;
- hard-stop security and rights suites pass with zero prohibited effects;
- reviewers can reject, edit, annotate, and revoke a brief revision;
- on-call staff can disable a connector, tenant, model analysis, index write, or publication independently;
- the team has measured whether the system improves analyst coverage or cycle time without unacceptable false alerts or review load.

## Sources and further reading

The [evidence packet](../../research/packets/competitive-market-intelligence-agent-blueprint.md) explains the research and limitations behind these requirements. Key foundations include the [W3C PROV overview](https://www.w3.org/TR/prov-overview/), [RFC 9110 HTTP semantics](https://www.rfc-editor.org/rfc/rfc9110.html), [RFC 9309 Robots Exclusion Protocol](https://www.rfc-editor.org/rfc/rfc9309.html), [NIST AI RMF Generative AI Profile](https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence), and the repository's canonical [cross-cutting control packet](../../research/packets/agent-blueprint-cross-cutting-controls.md).


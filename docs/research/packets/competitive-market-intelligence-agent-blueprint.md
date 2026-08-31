# Competitive and Market Intelligence Agent Blueprint: Research Packet

> Status: Pass 2 usefulness and production-depth research-backed draft; not a claim of production validation  
> Research completed: 2026-08-31  
> Refresh owner: blueprint maintainer and deployment-specific source governance owner  
> Evidence preference: current primary standards, official APIs/data publishers, official law/regulator material, original research, then carefully bounded practitioner guidance

## Guides supported

This packet supports:

- [Blueprint overview](../../agents/competitive-market-intelligence-agent/README.md)
- [Mission, boundary, and requirements](../../agents/competitive-market-intelligence-agent/01-mission-boundary-and-requirements.md)
- [Reference architecture and runtime](../../agents/competitive-market-intelligence-agent/02-reference-architecture-and-runtime.md)
- [Entities, watchlists, sources, and rights](../../agents/competitive-market-intelligence-agent/03-entities-watchlists-sources-and-rights.md)
- [Change detection, evidence, and provenance](../../agents/competitive-market-intelligence-agent/04-change-detection-evidence-and-provenance.md)
- [Analysis, scenarios, and briefings](../../agents/competitive-market-intelligence-agent/05-analysis-scenarios-and-briefings.md)
- [State, context, memory, and orchestration](../../agents/competitive-market-intelligence-agent/06-state-context-memory-and-orchestration.md)
- [Security, privacy, and governance](../../agents/competitive-market-intelligence-agent/07-security-privacy-and-governance.md)
- [Reliability, observability, evaluation, and incidents](../../agents/competitive-market-intelligence-agent/08-reliability-observability-evaluation-and-incidents.md)
- [Deployment, scale, cost, and maturity roadmap](../../agents/competitive-market-intelligence-agent/09-deployment-scale-cost-and-roadmap.md)
- [Provider qualification and worked intelligence lifecycle](../../agents/competitive-market-intelligence-agent/10-provider-qualification-and-worked-intelligence-lifecycle.md)

Canonical repository contracts were treated as constraints rather than copied: [state and events](../../runtime/agent-state-and-event-contracts.md), [durable execution](../../runtime/durable-execution.md), [run controls](../../runtime/run-controls.md), [tools](../../tools/README.md), [context and memory](../../context-memory/README.md), [security](../../security/README.md), [reliability](../../reliability/README.md), [evaluation](../../evaluation/README.md), [operations](../../operations/README.md), and [cross-cutting blueprint controls](agent-blueprint-cross-cutting-controls.md).

## Executive finding

The strongest practical design is not a general web-research agent running continuously. It is an application-owned, source-governed monitoring pipeline:

1. approved entity/watchlist and source-policy state;
2. deterministic collection and representation versioning;
3. schema-aware normalization, entity resolution, and layered change detection;
4. an evidence and contradiction graph with time, rights, and source-dependence semantics;
5. bounded model-assisted materiality, comparison, hypothesis, scenario, and briefing proposals;
6. human review for source-rights ambiguity, identity ambiguity, material claims, and consequential distribution;
7. durable state/effect recovery, workload-specific evaluation, incidents, cost control, and versioned upgrades.

The model proposes and explains. It does not authorize source use, determine legal permission, create entity truth, decide strategy, or report an external effect as successful without a receipt.

The research found no public benchmark or standard that jointly validates permitted source use, entity/version drift, monitoring recall, evidence support, contradiction handling, scenario quality, briefing usefulness, tenant isolation, and durable effect behavior. Production evidence therefore requires a private, rights-compatible sequence corpus and live shadow/canary evaluation.

## Research questions

1. What distinguishes recurring competitive/market intelligence from deep research, enterprise knowledge retrieval, analytics, and sales operations?
2. What source and identity records are necessary before collection?
3. What do official corporate/market data APIs reveal about versioning, cadence, rate limits, authentication, and revision history?
4. How should HTTP validators, feeds, webhooks, snapshots, hashes, parsers, semantic detectors, and source-origin clustering interact?
5. Which provenance and data-quality standards are useful, and where are application-specific support/rights semantics still necessary?
6. How should facts, calculations, inferences, hypotheses, scenarios, forecasts, recommendations, and decisions be separated?
7. What legal, privacy, database-rights, trade-secret, access-control, and professional-ethics issues constrain public-source intelligence?
8. What runtime, state, memory, idempotency, approval, recovery, and observability contracts are required?
9. When do agent frameworks, durable workflow engines, connector protocols, multiple agents, polyglot services, or microservices become justified?
10. What offline, online, adversarial, reliability, human, and cost evaluation can support a production release?
11. Which claims are volatile as of 2026 and need explicit refresh ownership?

## Research method

Research proceeded in six passes:

1. **Repository taxonomy and contracts:** category registry, expansion program, adjacent blueprints, and canonical runtime/control documents.
2. **Primary-source acquisition:** standards bodies, official data publishers/APIs, regulators, statutes/court opinions, government security guidance, and original forecasting research.
3. **Cross-check and conflict analysis:** compared technical access with permission, push delivery with completeness, newest data with reproducibility, citation with support, and scenario language with calibrated forecasting.
4. **Architecture synthesis:** converted the findings into typed schemas, authority boundaries, failure/recovery behavior, evaluation slices, and Stage 0–6 gates.
5. **Volatility review:** marked source quotas, standards maturity, taxonomy/API versions, law commencement, model/telemetry guidance, and benchmark transfer as refresh-sensitive.
6. **Pass 2 provider/operations refinement:** rechecked volatile search, news, filing, market, patent, social, enterprise, workflow, notification and MCP surfaces; converted provider contradictions and limits into qualification, continuity, effect, recovery and staged-exercise contracts.

Search results and secondary summaries were used only to discover primary sources. Claims in the blueprint were narrowed where a primary source applied only to one jurisdiction, provider, protocol, or research setting.

This is engineering research, not deployment-specific legal advice. A real implementation must assess its exact sources, contracts, fields, people, jurisdictions, purposes, recipients, providers, and effective law.

## Category promotion record

| Promotion gate | Result | Evidence and consequence |
|---|---|---|
| Category contract | Pass | The [category registry](../agent-blueprint-category-registry.md) assigns entity/watchlist state, public-source change detection, market evidence, scenario comparison, freshness, and executive briefing to this category. |
| Evidence continuity | Pass for research-backed draft | Primary sources support source-specific access behavior, provenance, HTTP/feed semantics, privacy/rights constraints, forecasting discipline, security, observability, and evaluation. No claim of field validation is made. |
| Evaluation and operations substantiation | Pass for blueprint design | The guides include deterministic, semantic, adversarial, reliability, human, and online gates; SLIs; incidents; kill switches; cost; rollout; rollback; and DR. Thresholds remain deployment-specific. |
| Architecture consequences | Pass | Findings determine the source-policy model, evidence graph, typed analysis, authority split, application-owned state, durable-workflow threshold, single-agent default, and maturity stages. |
| Category separation | Pass | Persistent entity/watchlist monitoring is separated from question-bounded deep research, ACL-aware enterprise knowledge, CRM/outreach sales operations, and strategic/trading actions. |

Promotion means the topic merits a dedicated blueprint and the design is research-backed. It does not mean “reviewed,” “battle-tested,” legally approved, or suitable for an organization without local verification.

## Evidence-to-decision map

| Evidence finding | Blueprint decision | Operational consequence |
|---|---|---|
| Official sources expose different authentication, cadence, quota, schema, taxonomy, and historical-version behavior. | Source policy and adapter behavior are versioned per distribution/endpoint. | No global scraping quota or generic “official source” adapter. |
| HTTP validators and feeds describe representation/delivery behavior, not business truth or complete processing. | Conditional retrieval plus scheduled reconciliation. | Webhooks/feeds accelerate; expected windows and receipts prove coverage. |
| Provenance standards represent entities, activities, agents, versions, rights, and distributions, but not workload-specific claim entailment. | Evidence graph borrows interoperable concepts and adds claim support, rights lifecycle, source independence, freshness, and review. | Citation existence is evaluated separately from support. |
| Public/open access can coexist with terms, attribution, third-party exceptions, privacy, database rights, or jurisdictional constraints. | Deny-by-default source-use lifecycle gate. | Rights checked at retrieval, retention, transformation, quotation, distribution, evaluation, and deletion. |
| Statistical/API “latest” data may omit past vintages. | Capture permitted snapshots/vintages and show reproduction limits. | A current API response cannot reconstruct every historical brief. |
| Entity registries and fuzzy search help but do not identify brands/products/groups automatically. | Time-aware canonical entity graph with reviewed ambiguity. | Fuzzy matches cannot support material claims by themselves. |
| Forecasting research supports explicit resolvable probabilities and updating in specific settings, not generic LLM market foresight. | Scenarios are conditional by default; probabilities require resolution and scoring. | No fake “base case confidence”; human forecast ownership. |
| Generative AI security guidance highlights confabulation, information integrity, prompt injection, and excessive agency. | Untrusted-content isolation and proposal/effect separation. | Source text cannot grant tools, memory writes, or publication authority. |
| Trace standards support correlation, and telemetry sampling can omit data. | Authoritative state/events/effects are separate from traces. | Missing trace is not evidence an effect did not occur. |
| Public eval tools can grade output/trajectory, but no benchmark spans this workload. | Private frozen sequences, failure mining, hard gates, shadow, and canary. | Benchmark scores cannot authorize production release. |

## Finding 1: Persistent monitoring is the defining category boundary

A competitive-intelligence system maintains approved watchlists and compares source state over time. Deep research instead starts with a bounded question and ends with a research artifact. Enterprise knowledge focuses on permission-aware internal corpora. Sales/revenue operations owns CRM state, outreach, routing, and commercial effects.

This difference changes the architecture:

- recurring schedules and freshness objectives;
- entity, source, cursor, representation, and contradiction history;
- revision/correction of previously published briefs;
- provider/source policy drift;
- durable waits and reconciliation;
- coverage and alert-quality evaluation over time.

The blueprint therefore prohibits silent handoffs into CRM, outreach, strategy commitment, or trading. An approved brief can be an input to those workflows through a new contract and authority check.

## Finding 2: Source identity and permitted use are first-class control-plane state

The official sources reviewed have materially different operating contracts:

- [SEC EDGAR APIs](https://www.sec.gov/search-filings/edgar-application-programming-interfaces) expose submissions and XBRL data via `data.sec.gov` JSON without per-user API keys. SEC's [Webmaster FAQ](https://www.sec.gov/about/webmaster-frequently-asked-questions) stated a maximum automated access rate of ten requests per second and requested a declared user agent at the research date. This is provider guidance to recheck, not a universal target.
- [Companies House](https://developer.company-information.service.gov.uk/get-started) requires API authentication for public company data. Its [developer guidelines](https://developer.company-information.service.gov.uk/developer-guidelines) documented a default 600 requests per five-minute period and `429` behavior at the research date, and warned against attempts to bypass limits.
- [GLEIF](https://www.gleif.org/en/lei-data/gleif-api) exposes LEI records, relationships, filters, and fuzzy-name search. [Golden Copy/delta files](https://www.gleif.org/en/lei-data/gleif-golden-copy/download-the-golden-copy) were produced three times daily. These improve legal-entity resolution; they do not identify every brand/product or remove human ambiguity.
- [ESMA ESEF](https://www.esma.europa.eu/issuer-disclosure/electronic-reporting) relies on evolving electronic-reporting taxonomy/packages. A filing parser must pin the exact package/taxonomy.
- [Eurostat](https://ec.europa.eu/eurostat/data/web-services) documents twice-daily data updates and says its web services provide latest datasets without versioning/past versions. Historical comparison therefore needs a permitted internal vintage or an explicit reproducibility limitation.

The correct abstraction is not `fetch(url)`. It is `retrieve(approved_source_distribution, target, collection_window, validator/cursor, policy_version, budget)` with a receipt.

## Finding 3: Public, crawlable, open, and reusable are different states

[RFC 9309](https://www.rfc-editor.org/rfc/rfc9309.html) standardizes the Robots Exclusion Protocol and says it is not access authorization. A robots decision does not settle contract, license, copyright, database rights, privacy, confidentiality, or trade-secret issues.

[DCAT 3](https://www.w3.org/TR/vocab-dcat-3/) distinguishes datasets from distributions and models license, rights, and access rights. That maps directly to endpoint-level policy. [Eurostat's copyright notice](https://ec.europa.eu/eurostat/help/copyright-notice) provides a concrete example: general reuse terms coexist with attribution and stated exceptions involving third-party works and specified data. GLEIF describes its own data as [CC0 open data](https://www.gleif.org/en/about/open-data/), but this cannot grant rights in unrelated linked content.

The [EU Database Directive](https://eur-lex.europa.eu/eli/dir/1996/9/2019-06-06/eng/) and jurisdiction-specific access cases further show why “public” cannot be encoded as “unrestricted reuse.” Legal analysis is fact-specific. The blueprint's engineering response is to require accountable policy decisions for each lifecycle phase and stop when indeterminate.

## Finding 4: Collection ethics must prohibit pretexting and confidential acquisition

The [EU Trade Secrets Directive](https://eur-lex.europa.eu/eli/dir/2016/943/oj/eng/) identifies lawful and unlawful acquisition categories in its scope, including concerns around unauthorized access/copying, confidentiality breaches, and honest commercial practices. US access decisions in [Van Buren](https://www.supremecourt.gov/opinions/20pdf/19-783_k53l.pdf) and [hiQ](https://cdn.ca9.uscourts.gov/datastore/opinions/2022/04/18/17-16783.pdf) address particular statutory questions and facts; neither is a universal automation safe harbor.

Competitive-intelligence professional guidance, such as the [CI Fellows Code of Ethics](https://www.cifellows.com/code-of-ethics) and [SCIP ethics implementation guidance](https://www.scip.org/general/custom.asp?page=implementing-competitive-intelligence-ethics-policy), supports truthful identity and lawful collection. These sources are professional norms, not law. The blueprint implements the conservative overlap: no pretexting, misrepresentation, access-control bypass, solicitation of secrets, or use of apparently confidential/misdirected data.

## Finding 5: Privacy follows the data even when the source is public

The [GDPR](https://eur-lex.europa.eu/eli/reg/2016/679/oj) establishes principles including lawfulness, purpose limitation, minimization, accuracy, storage limitation, integrity/confidentiality, and accountability. The [EDPB principles page](https://www.edpb.europa.eu/topics/key-gdpr-concepts/basic-principles_en) is a current explanatory reference. Public role information can still be personal data and may trigger source/notice/rights questions depending on the facts.

India's official [Digital Personal Data Protection Rules, 2025](https://www.meity.gov.in/documents/act-and-policies/digital-personal-data-protection-rules-2025-gDOxUjMtQWa) are especially relevant to maturity drift because the Gazette notification phases commencement. A guide written once cannot safely state all obligations are simultaneously in force.

The workload should be organization/product centered, minimize named-person data to necessary official-role facts, and prohibit private-life/sensitive-trait inference and individual surveillance. Deployment-specific privacy owners must select the legal/policy basis and effective requirements.

## Finding 6: Delivery and retrieval signals are hints, not completeness proofs

[RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html) defines HTTP validators and conditional requests. `ETag`, `Last-Modified`, and `304` improve efficient representation retrieval; they do not prove that every business fact is unchanged or identify the effective time of a change.

[Atom](https://www.rfc-editor.org/rfc/rfc4287.html) provides permanent entry identifiers and an `updated` timestamp tied to publisher-significant modification, while its semantics do not guarantee that every material business change produces the signal a consumer expects. [WebSub](https://www.w3.org/TR/websub/) provides a publisher/hub/subscriber protocol with subscriptions and deliveries, but production consumers still need deduplication, lease management, retries, and reconciliation.

Therefore:

- schedule expected source windows;
- use push/feed signals to accelerate collection;
- deduplicate stable provider/event identifiers;
- reconcile expected windows against definitive receipts;
- represent source coverage independently of “no detected change.”

## Finding 7: Hash equality is useful but not semantic evidence

Raw digests show byte equality under a defined representation. Canonical digests require a versioned transformation. [RFC 8785](https://www.rfc-editor.org/rfc/rfc8785.html) is a useful JSON canonicalization reference with a particular data model, not a universal HTML/table/XBRL solution. [FIPS 180-4](https://csrc.nist.gov/pubs/fips/180-4/upd1/final) documents hash standards but had a revision-planning note at the research date, so production cryptographic profiles need a current owner.

The blueprint uses layered detection: transport, raw bytes, canonical structure, typed fields, bounded semantics, and cross-source comparison. A hash difference can be noise; identical presentation bytes can still be stale relative to an upstream system; a semantic detector can hallucinate. Each layer emits a receipt and warning state.

## Finding 8: Time and definition are part of every market claim

Sources expose published time, effective time, valid time, retrieval time, provider revision, and internal recorded time inconsistently. Market/statistical series also vary by geography, population, inclusion/exclusion, nominal/real basis, currency, seasonal adjustment, period, taxonomy, and vintage.

[XBRL specifications](https://www.xbrl.org/the-standard/what/specifications/) and the current [specifications index](https://specifications.xbrl.org/specifications.html) demonstrate a large evolving structured-reporting ecosystem. Structured data reduces extraction ambiguity but does not make concepts/contexts/units interchangeable.

The architecture therefore preserves original values and definitions, pins taxonomy/parser versions, refuses incompatible direct comparisons, and distinguishes `published_at`, `effective_at`, `valid_*`, `retrieved_at`, `observed_at`, and `recorded_at`.

## Finding 9: Provenance needs application-specific claim semantics

[W3C PROV](https://www.w3.org/TR/prov-overview/) represents entities, activities, agents, derivation, and provenance bundles. [PROV-O](https://www.w3.org/TR/prov-o/) provides an ontology mapping. [DCAT 3](https://www.w3.org/TR/vocab-dcat-3/) contributes dataset/distribution/version/rights metadata. [DQV](https://www.w3.org/TR/vocab-dqv/) offers a non-normative data-quality vocabulary and recognizes contextual quality.

These are valuable interoperability foundations, but they do not decide whether a cited span entails a market claim, whether two articles share one origin, whether a definition changed, or whether a source may be used in an executive brief/evaluation. The blueprint adds explicit representation, observation, change, evidence, claim, contradiction, source-origin, review, and effect records.

## Finding 10: Contradiction is not a newest-record problem

A later record can be a true supersession, correction, different valid time, incompatible definition, entity mismatch, or duplicate copy. Last-write-wins destroys that distinction. A robust system preserves each statement with source/version/time/definition, classifies the relationship, and seeks discriminating evidence.

The blueprint reports independent-origin count separately from URL/source count. Five syndicated articles are not five confirmations. Source authority remains claim-specific: a company is primary evidence for what it publicly asserted; a regulator is primary for its filing record; neither is automatically independent confirmation of all underlying facts.

## Finding 11: Scenario comparison should not manufacture probability

The original [Brier paper](https://journals.ametsoc.org/doi/10.1175/1520-0493%281950%29078%3C0001%3AVOFEIT%3E2.0.CO%3B2) provides a way to score probability forecasts. [Mellers et al.](https://pubmed.ncbi.nlm.nih.gov/25581088/) reported that training, collaboration, and frequent belief updating were predictive of accuracy in a large geopolitical forecasting tournament. That evidence supports explicit resolvable forecasts and calibration discipline in a related setting; it does not validate unscored LLM probabilities for product or market outcomes.

The default blueprint output is a conditional scenario with horizon, assumptions, evidence, causal path, alternative, implications, confirming/invalidating triggers, and blind spots. Probability is enabled only for a named, resolvable event with outcome capture and scoring. Strategic actions remain human decisions.

## Finding 12: Source content creates a cross-tool security problem

The [NIST Generative AI Profile](https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence) discusses risks including confabulation, automation bias, privacy, and information integrity/provenance. [OWASP's agentic security project](https://genai.owasp.org/2025/12/09/owasp-top-10-for-agentic-applications-the-benchmark-for-agentic-security-in-the-age-of-autonomous-ai/) and its [Excessive Agency guidance](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/) reinforce the need to minimize privileges and consequential actions.

Instruction detection alone is not reliable. The decisive architecture controls are:

- parse hostile content in an isolated boundary;
- label source fragments as untrusted data;
- compile minimal context from admitted evidence;
- expose typed read/proposal tools, not generic browser/shell/network/memory/publish tools;
- authorize outside the model;
- bind approvals to exact content/scope;
- record and reconcile effects.

## Finding 13: Memory should be a curated evidence/domain model

Recurring monitoring requires durable task/domain state: source cursors, representations, entity/watchlist versions, evidence, contradictions, prior approved claims, review state, and effects. This is not a reason for autonomous free-form long-term memory.

The blueprint enables run working state and curated domain memory, disables inferred user profiles and free-form episodic memory by default, and promotes reviewed failures into rights-compatible evaluation fixtures. Context is compiled from stable IDs and current policy each step. Compaction preserves numbers, units, dates, qualifiers, contradictions, rights, and provenance.

## Finding 14: Durable execution and traces solve different problems

Durable workflows can persist timers, waits, retries, and execution history across process loss. They do not guarantee exactly-once external publication, establish domain evidence truth, or authorize effects. The application still needs idempotency, effect intent/receipt, reconciliation, cancellation fencing, and versioned in-flight behavior.

[CloudEvents](https://github.com/cloudevents/spec) is an interoperable event-envelope reference, not a delivery/processing guarantee. [W3C Trace Context](https://www.w3.org/TR/trace-context/) correlates distributed work and has privacy/security considerations; it is not identity or authorization. [OpenTelemetry trace SDK semantics](https://opentelemetry.io/docs/specs/otel/trace/sdk/) include sampling behavior, so trace absence is not an authoritative negative record.

## Finding 15: Workload-specific evaluation must combine artifacts, trajectories, failures, and humans

OpenAI's [eval guide](https://developers.openai.com/api/docs/guides/evals), [agent eval guide](https://developers.openai.com/api/docs/guides/agent-evals), and [trace grading guide](https://developers.openai.com/api/docs/guides/trace-grading) illustrate current provider tooling for criteria-driven output/workflow evaluation. They do not define competitive-intelligence correctness or replace deterministic controls.

A production suite needs:

- deterministic policy, identity, time, unit, calculation, citation, state, and effect tests;
- frozen multi-version source sequences for change precision/recall and lag;
- claim-support, contradiction, independence, freshness, and abstention evaluation;
- scenario assumption/trigger rubrics and scored forecasts only where defined;
- prompt-injection, SSRF/file, tenant, privacy, rights, approval, and publication attacks;
- process/provider/source/parser/telemetry failure injection;
- human usefulness, edit time, rejection reason, and review capacity;
- online shadow/canary and post-release drift;
- per-source/entity/language/geography/materiality/rights slices and hard-stop gates.

## Finding 16: Provider qualification is purpose, audience, tenant, plan, operation, and rights specific

A provider name or successful connection cannot define one capability. Google Custom Search's current closure/retirement notice, USPTO ODP's 2026 account/MFA migration, Slack's app-class-dependent history limits, and Salesforce's seasonal API releases illustrate different forms of operational drift. Rights can vary further by source distribution, content owner, plan, transformation and audience.

**Decision:** every adapter has a typed capability manifest, target-configuration conformance report, known limitations, owners and expiry. Identity, pagination, terminal condition, timestamps, revisions/deletes, rate behavior, permissions, retention, redistribution and effect ambiguity are tested independently.

## Finding 17: Provider surfaces have incompatible semantics

Search and news rank incomplete discovery results. Regulatory APIs record filings or published datasets. Social APIs expose mutable user content under platform/content-owner constraints. CRM and BI return tenant- and permission-shaped enterprise state. A dashboard refresh, search rank, feed delivery, filing accession and source-page edit cannot share one generic `updated_at` meaning.

**Decision:** preserve provider-native object/status/time/cursor/version fields inside a common evidence envelope, but map them through source-specific releases. Missing, filtered, capped, deleted, revised and unknown states remain distinct.

## Finding 18: Current official rights pages can contradict older technical expectations

FRED/ALFRED API documentation describes series/vintage retrieval, while the current official FRED legal notice contains materially restrictive language on AI use, caching/archiving and third-party series. News API and Reddit similarly separate API access from content/use permissions. This is not resolved by choosing the page that enables implementation.

**Decision:** record both official sources, access dates and the contradiction; deny the disputed use until an accountable rights owner resolves the exact contract/use. The blueprint never supplies legal advice.

## Finding 19: Exactly seven memory lifetimes are enough

The useful lifetimes are Turn/scratch, Working/run, Session, Durable workflow/task, Domain knowledge, Long-term/preference and Episodic/outcome. Caches, indexes, embeddings and provider threads are derived projections or transport state, not new authority.

**Decision:** each lifetime now states use, rejection, retention, correction/deletion and poisoning tests. Long-term preference and episodic reuse are disabled unless an explicit scoped benefit, rights basis and governance exist.

## Finding 20: Restart-safe compaction is a continuity proof

A prose summary cannot safely resume recurring intelligence. Continuity depends on source-specific event high-watermarks, rights/source policy, tools/adapters/context/compiler/model releases, exact contradictions/limitations, current review/approvals, pending/unknown effects and the next safe state-machine action.

**Decision:** compaction emits a schema-versioned receipt with pre/post invariant hashes and `known_losses`. Material analysis or publication blocks if continuity cannot be reconstructed.

## Finding 21: Usefulness is a downstream reviewed outcome, not prose quality

Fresh and supported briefs may still be irrelevant; fluent briefs may create review burden without changing a decision input. No public benchmark establishes local competitive-intelligence usefulness.

**Decision:** pair freshness/completeness/support with named reviewer outcomes: changed or confirmed assumption, changed monitoring priority, follow-up question, explicit no-action, substantive edit time, correction and decision-owner disposition. Model self-assessment cannot provide the label.

## Finding 22: Recovery must include rights, correction and human capacity

After a regional failure, restoring databases does not prove safe operation. Backlog can contain stale source policies, missed required windows, deleted social/internal content, source corrections, unknown publication effects and expiring reviewer capacity.

**Decision:** DR restores policy/watch/entity/evidence/effect/deletion continuity, then measures net recovery drain against new critical arrivals. Required-source/correction/reconciliation work receives priority over search/news backfill and model enrichment.

## Architecture options compared

| Option | Strength | Weakness | Decision |
|---|---|---|---|
| Manual analyst plus alerts | Strong judgment and low automation risk | Limited coverage, inconsistent evidence/state, expensive repetition | Use as Stage 0 reference and review authority. |
| Deterministic ETL/diff digest | Reproducible, cheap, easy to evaluate | Weak on semantic rewording, diverse documents, cross-source explanation | Default baseline; may remain final architecture. |
| General autonomous web agent | Flexible discovery and synthesis | Permission/completeness ambiguity, injection/tool risk, weak recurring state and recovery | Reject as the main monitoring architecture. |
| Hybrid governed pipeline plus bounded analysis agent | Deterministic control with selective semantic power | Requires careful evidence/policy schemas and evaluation | Recommended. |
| Framework-owned agent memory/state | Fast prototype | Hidden authority, weak durability/versioning/audit portability | Reject for authoritative state; optional inside bounded analysis. |
| Durable workflow from day one | Strong waits/recovery | Operational and cognitive overhead before need is proven | Adopt at Stage 3 threshold, not automatically. |
| Microservice per logical component | Independent scaling/isolation | High operational/schema/ownership cost | Start modular; split on measured failure/scaling/security need. |
| Multiple analyst/critic/writer agents | Potential parallelism/challenge | Correlated errors, context duplication, conflict merge, cost, authority ambiguity | Disabled by default; enable only after controlled win. |
| Vector-store “memory” of all collected content | Flexible semantic recall | Rights/deletion/tenant/freshness/poisoning and provenance problems | Reject; use filtered curated evidence/domain indexes. |
| Organization-approved MCP connector | Uniform typed tool transport | Protocol/tool drift, confused-deputy/task isolation, and no domain rights/completeness semantics | Conditional only after the same source-specific qualification; never the evidence/effect ledger. |

## Important contradictions and resolutions

| Apparent contradiction | Resolution |
|---|---|
| “Robots is not authorization” versus “respect robots” | Both are true. REP does not grant legal authorization, while organizational source policy can and should enforce approved robots behavior alongside other rights/access controls. |
| Public API versus authenticated/rate-limited API | “Public data” describes availability/subject, not necessarily access method or unlimited use. Connector policy retains authentication and quota. |
| Open data versus restricted reuse | A publisher's open license can coexist with attribution, distribution-level conditions, third-party exceptions, privacy, or database rights. Evaluate the actual distribution/content/use. |
| Webhook/feed says updated versus polling sees no material change | Delivery metadata indicates publisher/transport activity, not semantic materiality. Store both and run layered detection. |
| Latest official value versus reproducible prior brief | Latest is authoritative for the current release but cannot reconstruct a prior vintage. Preserve permitted snapshots and version claims. |
| More citations versus stronger support | Citation count can reflect syndication or irrelevant links. Evaluate entailment, authority for the claim, origin independence, definition, freshness, and contradictions. |
| Newest source versus truth | Recency cannot resolve definition/time/entity mismatches or corrections. Classify the relationship and preserve evidence. |
| Model confidence versus evidence quality | Model probability/confidence is meaningful only for a defined calibrated event/classifier. Evidence admission uses explicit dimensions and rules. |
| Scenario versus forecast | A scenario is conditional; a forecast assigns probability to a resolvable event. Keep probability null unless scored. |
| Durable workflow versus exactly-once | Durable replay improves execution recovery; external effects still need idempotency and reconciliation. |
| Trace shows call versus effect occurred | A trace is telemetry and may be sampled/incomplete. The effect receipt/destination reconciliation is authoritative. |
| More agents versus better analysis | Parallel roles may improve a measured slice but also duplicate evidence and correlate error. Require a controlled advantage over one bounded agent. |

## Claims deliberately narrowed or excluded

- No universal statement that web scraping is lawful or unlawful; legality depends on jurisdiction, facts, contract, access, data, and use.
- No source listed is automatically approved for every deployment, purpose, audience, or retention plan.
- No provider rate limit is encoded as permanent best practice; checked values are examples requiring refresh and safer local configuration.
- No claim that registries/fuzzy matching establish brand/product/group identity without review.
- No claim that hashes authenticate source truth or semantic materiality.
- No claim that citations guarantee support.
- No market forecast accuracy claim for LLMs.
- No assertion that a particular model, framework, workflow engine, vector database, language, or cloud is universally best.
- No assumption that telemetry or benchmark scores are authoritative release evidence by themselves.
- No legal opinion about GDPR basis, database rights, trade secrets, CFAA, terms, copyright, or DPDP compliance.
- No personal surveillance, securities recommendation/trading, CRM/outreach, covert collection, or strategy execution capability.

## Version, volatility, and maturity baseline

| Item | State observed on 2026-08-31 | Why volatile | Refresh trigger |
|---|---|---|---|
| SEC automated access | FAQ stated maximum ten requests/second and declared user-agent expectations. | Provider policy, infrastructure, and wording can change. | Before connector release and at scheduled source-policy review. |
| Companies House rate policy | Developer guidance stated default 600 requests per five minutes with `429`. | Plan/access/provider policy can change. | Before release, on headers/errors, and quarterly. |
| GLEIF distributions | API plus Golden Copy/delta; three Golden Copy sets daily. | API fields, schedules, formats, license/terms can change. | Adapter/schema update or source-policy review. |
| Eurostat web services | Latest datasets; update twice daily; no past versions through described services. | API generations, endpoints, datasets, rights notes evolve. | Before relying on vintage behavior or new dataset. |
| ESMA ESEF/XBRL | Evolving annual taxonomy/reporting packages and current XBRL specifications. | Taxonomy/spec/package revisions are routine. | Every filing/taxonomy year and parser upgrade. |
| WebSub | W3C Recommendation page dated June 2026 at research time. | Very recent recommendation and implementation behavior. | Before implementation and interoperability changes. |
| NIST AI RMF/GenAI profile | GenAI profile published 2024; NIST indicated broader AI RMF revision activity in 2026. | Guidance and profiles evolve. | On NIST revision/new profile. |
| OWASP agentic guidance | Agentic Top 10 released as a 2026-focused project; related pages evolve. | Community guidance/maturity and taxonomies change. | Each major release; treat as threat input, not compliance standard. |
| OpenTelemetry | Main specification and GenAI semantic conventions were active/evolving; GenAI conventions not a stable application authority contract. | Semantic attributes/status can change. | Every OTel/semconv upgrade. |
| FIPS 180-4 | Current page included a revision-planning note. | Cryptographic standards/policy update. | Organizational crypto-policy or NIST revision. |
| India DPDP Rules 2025 | Official notification phases commencement over different intervals. | Different provisions become effective at different dates; guidance may follow. | Before deployment/processing change and each commencement date. |
| OpenAI eval guidance | Current guides cover evals, agent evals, and trace grading. | APIs/models/guides and retention behavior can change. | Provider/model/API release and before adapter upgrade. |
| Legal access/privacy landscape | Statutes, case law, regulator guidance, terms, and jurisdiction differ. | Intrinsically fact- and date-sensitive. | Every new source/use/jurisdiction/audience and periodic counsel review. |

## Benchmark limitations

### No end-to-end public benchmark

The research found benchmarks for retrieval, question answering, citation/factuality, entity resolution, change detection, forecasting, and agents, but none establishes production readiness for this combined recurring workload. Missing dimensions commonly include:

- permission/terms/license/privacy lifecycle and deletion;
- source cadence, rate limits, revisions, and availability gaps;
- time-aware entity merges/splits and brand/product relationships;
- multi-version web/API/filing sequences and semantic change materiality;
- source syndication/dependence and contradiction relationship types;
- scenario assumptions and executive usefulness;
- tenant isolation, prompt injection, and controlled publication;
- durable recovery, ambiguous effects, correction, and revocation;
- analyst review capacity and unit economics.

### Transfer limits

- Deep-research or QA benchmarks are question-bounded and often assume static corpora; they do not prove recurring monitoring recall or source-policy compliance.
- Citation benchmarks can score citation placement/relevance without source rights, origin independence, temporal validity, or full-claim entailment.
- Entity-linking datasets rarely reflect the exact organizations/products/jurisdictions and time-aware relationships in a deployment.
- Generic agent benchmarks can reward task completion while omitting consequential effects and governance.
- Geopolitical forecast findings do not directly validate market/product forecasts or model-generated probabilities.
- Model-as-judge scores inherit grader bias, version drift, and possible correlated model error.

### Required private corpus

Create a rights-compatible corpus of source **sequences**, not isolated documents:

- before/after/no-change/revision/retraction representations;
- provider/feed/webhook delivery and expected polling windows;
- exact entity/watchlist/definition/source-policy state at the time;
- expected typed deltas, admissible evidence, source-origin clusters, contradictions, claims, and abstentions;
- reviewer rationale and materiality;
- attack and infrastructure failure variants;
- expected state/effect/recovery outcomes;
- permitted evaluation retention, model-provider use, and expiry.

Use temporal train/test separation and hold out entities/source templates so the suite does not only reward memorized layouts. Keep a sealed incident set for promotion decisions.

## Decision record

| Decision | Selected approach | Reason | Reconsider when |
|---|---|---|---|
| Baseline | Deterministic monitoring first | Establishes demand, truth data, source behavior, and safe fallback. | Never remove; it remains comparator/degrade mode. |
| Agent role | One bounded analysis agent | Captures semantic value with limited authority/coordination cost. | Multi-worker agent flow wins controlled quality/latency/cost tests. |
| Orchestration | Application-owned fixed workflow | Most steps are predictable and policy-heavy. | Long waits/recovery make durable engine valuable. |
| State | Typed authoritative records outside model/framework | Required for recurrence, audit, correction, replay, and effects. | Not expected to change. |
| Memory | Curated domain memory; no free-form long-term memory by default | Preserves provenance/rights/freshness and limits poisoning. | A specific opt-in preference/outcome use has schema, policy, and measured benefit. |
| Collection | Approved typed connectors; no generic browser in analysis | Limits access/injection/SSRF and makes coverage evaluable. | Exceptional source is separately governed and sandboxed. |
| Provider qualification | Tenant/purpose/audience/operation-scoped manifest and conformance report | Connection success cannot prove identity, completeness, rights, retention or semantic behavior. | Requalify after provider/configuration/terms/use/incident change. |
| Source rights | Lifecycle decisions per distribution/purpose/audience | Public access is not unrestricted permission. | Never waive; implementation may become more automated using human-owned policy data. |
| Evidence | Versioned graph with typed claims/contradictions | Prose/citations alone cannot preserve support and revision semantics. | Not expected to change; mappings may evolve. |
| Scenarios | Conditional; probability opt-in with resolution/scoring | Avoids false precision and strategy confusion. | Forecast program has enough resolved data and accountable owners. |
| Effects | Draft/review by default; exact approved publication with receipt | Distribution is consequential and can expose licensed/strategic content. | Audience/risk profile supports a tested reversible D2 path. |
| Deployment | Modular service/worker first | Minimizes operational complexity. | Measured scale/isolation/ownership requires split. |
| Language | Existing supported organizational runtime | Operations/ownership outweigh generic ecosystem preference. | Verified parser/analytics/security constraint justifies another runtime. |
| Framework | Optional helper inside bounded analysis | Framework memory/state cannot become authority. | Framework passes contracts and materially reduces implementation burden. |
| Evaluation | Private sequences + hard gates + shadow/canary | No public benchmark spans the workload. | Public suite emerges and is validated for local sources/risks; still supplement locally. |

## Primary and direct sources reviewed

All sources below were accessed or rechecked during the 2026-08-31 research pass. Their inclusion records what informed the blueprint, not blanket endorsement or permission for deployment use.

### Corporate, filing, identity, and market-data sources

| Source | Type | Contribution and limitation |
|---|---|---|
| [SEC EDGAR Application Programming Interfaces](https://www.sec.gov/search-filings/edgar-application-programming-interfaces) | Official regulator API documentation | Current submissions/company facts access pattern. Source-specific and US filing focused. |
| [SEC Developer Resources](https://www.sec.gov/about/developer-resources) | Official regulator index | Direct source for SEC data/developer materials. Connector policy still checks exact endpoint. |
| [SEC Webmaster Frequently Asked Questions](https://www.sec.gov/about/webmaster-frequently-asked-questions) | Official provider operations guidance | Automated access rate/user-agent and filing availability notes at check date. Highly volatile. |
| [Companies House API: get started](https://developer.company-information.service.gov.uk/get-started) | Official government API documentation | Authentication and public company-data access. Does not authorize every downstream use. |
| [Companies House developer guidelines](https://developer.company-information.service.gov.uk/developer-guidelines) | Official provider operations guidance | Rate/`429`/bypass guidance at check date. Volatile per plan/provider. |
| [GLEIF API](https://www.gleif.org/en/lei-data/gleif-api) | Official global identifier API documentation | Legal-entity/relationship/filter/fuzzy-search capabilities. LEI is not brand/product identity. |
| [GLEIF Golden Copy downloads](https://www.gleif.org/en/lei-data/gleif-golden-copy/download-the-golden-copy) | Official bulk/delta documentation | Versioned Golden Copy and delta cadence. Format/schedule need refresh. |
| [GLEIF access and use](https://www.gleif.org/en/lei-data/access-and-use-lei-data) | Official data-access documentation | Access/distribution context and reporting exceptions. Exact use still assessed. |
| [GLEIF open data](https://www.gleif.org/en/about/open-data/) | Official rights statement | CC0 statement for GLEIF data. Does not cover unrelated third-party data. |
| [ESMA electronic reporting](https://www.esma.europa.eu/issuer-disclosure/electronic-reporting) | Official regulator documentation | ESEF reporting/taxonomy evolution. EU issuer/reporting scope. |
| [ESMA ESEF Taxonomy 2025](https://www.esma.europa.eu/document/esef-taxonomy-2025) | Official taxonomy package page | Example of versioned reporting artifacts and notices. Must use the relevant year/version. |
| [XBRL specifications overview](https://www.xbrl.org/the-standard/what/specifications/) | Official standards consortium | Structured reporting concepts/spec families. Not a claim of cross-taxonomy comparability. |
| [Current XBRL specifications index](https://specifications.xbrl.org/specifications.html) | Official standards index | Maturity/version status for specifications. Volatile. |
| [XBRL Open Information Model](https://specifications.xbrl.org/work-product-index-open-information-model-open-information-model.html) | Official specification work-product page | Normalized XBRL representation reference. Must pin chosen version/maturity. |
| [Eurostat web services](https://ec.europa.eu/eurostat/data/web-services) | Official statistics API documentation | Update/latest/no-versioning behavior. Dataset-specific definitions still required. |
| [Regulations.gov API v4](https://open.gsa.gov/api/regulationsgov/) | Official government API documentation | Documents/comments/dockets, keys, pagination and withdrawn/agency-field semantics. POST comments are outside this blueprint's authority. |
| [Eurostat API data access guide](https://ec.europa.eu/eurostat/web/user-guides/data-browser/api-data-access/) | Official statistics API documentation | Current access options and API guidance. Endpoints/generations evolve. |
| [Eurostat copyright and reuse policy](https://ec.europa.eu/eurostat/help/copyright-notice) | Official publisher rights guidance | Attribution and exceptions; demonstrates distribution/content-specific review. |
| [ESS web content retrieval guidelines](https://cros.ec.europa.eu/topic/ess-web-content-retrieval-guidelines) | Official European statistics guidance | Transparent, ethical, rights-aware retrieval and preference for agreements/APIs. Guidance, not universal law. |

### Current provider and enterprise-surface sources

| Source | Type | Contribution and limitation |
|---|---|---|
| [Google Custom Search JSON API overview](https://developers.google.com/custom-search/v1/overview) | Official provider documentation | Closed to new customers and scheduled for discontinuation on 2027-01-01 at the access date. Search results remain incomplete discovery, not evidence. |
| [News API terms](https://newsapi.org/terms) | Official provider terms | Developer plan is not for staging/production and third-party content rights remain; terms last state 2020 and exact plan/content rights need current confirmation. |
| [FRED/ALFRED API overview](https://fred.stlouisfed.org/docs/api/fred/overview.html) | Official provider API documentation | Series and archival-vintage capabilities. Does not by itself grant the intended AI/storage/redistribution use. |
| [FRED current legal notice](https://fred.stlouisfed.org/legal/) | Official provider terms | Current restrictive AI, caching/archiving and third-party-series language creates a deployment-blocking rights question against common technical expectations. |
| [USPTO Open Data Portal API syntax and migration notice](https://data.uspto.gov/apis/api-syntax-examples) | Official government API documentation | 2026 account/MFA/registration changes and legacy Developer Hub decommissioning. Target endpoints and keys require requalification. |
| [Reddit Data API Terms](https://redditinc.com/policies/data-api-terms) | Official provider terms | Revised 2026-07-20; purpose, modification, retention, redistribution, AI training, termination and deletion restrictions. User-content rights remain separate. |
| [Microsoft Graph drive delta v1.0](https://learn.microsoft.com/en-us/graph/api/driveitem-delta?view=graph-rest-1.0) | Official provider API documentation | Change tracking with documented property omissions by operation/service. Tenant ACL, tombstone and reconciliation behavior remain deployment-specific. |
| [Salesforce Connect REST API reference](https://developer.salesforce.com/docs/platform/connect-rest-api/references) | Official provider API reference | Summer ’26 (`v67.0`) surface at check date. Org objects, permissions, sharing and event/retention behavior require target-tenant tests. |
| [Power BI REST API](https://learn.microsoft.com/en-us/rest/api/power-bi/) | Official provider API documentation | Permission/service-principal, throttling and possible cross-home-region processing notes. Dashboard/refresh state is derived, not source truth. |
| [Jira Cloud REST API v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro) | Official provider API reference | Current cloud API with permission, pagination, ordering, asynchronous and experimental distinctions. Issue/workflow status is not an intelligence approval without a local contract. |
| [Slack history/replies rate-limit change](https://api.slack.com/changelog/2025-05-terms-rate-limit-update-and-faq) | Official provider changelog/terms guidance | Limits differ by app/install class; history access is not a completeness guarantee and terms constrain storage/use. |
| [MCP 2025-11-25 authorization](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization) | Official protocol specification | Resource indicators, token audience validation and no token passthrough for HTTP authorization. Does not define intelligence rights or evidence semantics. |
| [MCP 2025-11-25 tasks](https://modelcontextprotocol.io/specification/2025-11-25/basic/utilities/tasks) | Official protocol specification | Experimental task lifecycle and authorization-context isolation. Application state/effect reconciliation remains necessary. |
| [MCP 2026-07-28 release candidate](https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/) | Official project release-candidate notice | Signals imminent protocol changes; it is not a stable production target until finalized and qualified. |

### Web acquisition, events, provenance, and data quality

| Source | Type | Contribution and limitation |
|---|---|---|
| [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) | Internet Standard | Conditional requests, validators, representation semantics. Not business freshness/materiality. |
| [RFC 9309: Robots Exclusion Protocol](https://www.rfc-editor.org/rfc/rfc9309.html) | Internet Standard | Standard crawler-control protocol and explicit non-authorization scope. Not a legal permission decision. |
| [RFC 4287: Atom Syndication Format](https://www.rfc-editor.org/rfc/rfc4287.html) | Internet Standard | Feed identifier/update semantics. Does not guarantee business completeness. |
| [W3C WebSub](https://www.w3.org/TR/websub/) | W3C Recommendation | Push subscription/delivery model. Still requires consumer idempotency/reconciliation. |
| [CloudEvents specification repository](https://github.com/cloudevents/spec) | Official open specification repository | Interoperable event envelope/identity concepts. Not delivery/processing semantics. |
| [W3C PROV overview](https://www.w3.org/TR/prov-overview/) | W3C Recommendation overview | Entities, activities, agents, provenance family. Application support semantics remain local. |
| [W3C PROV-O](https://www.w3.org/TR/prov-o/) | W3C Recommendation | Provenance ontology terms. Does not decide evidence admission. |
| [W3C DCAT 3](https://www.w3.org/TR/vocab-dcat-3/) | W3C Recommendation | Dataset/distribution, version, license/rights/access metadata. Not legal advice. |
| [W3C Data Quality Vocabulary](https://www.w3.org/TR/vocab-dqv/) | W3C Working Group Note | Context-dependent quality vocabulary. Non-normative and not a truth score. |
| [RFC 8785: JSON Canonicalization Scheme](https://www.rfc-editor.org/rfc/rfc8785.html) | Informational RFC | Repeatable JSON canonicalization within its constraints. Not universal content normalization. |
| [NIST FIPS 180-4](https://csrc.nist.gov/pubs/fips/180-4/upd1/final) | Federal cryptographic standard | Hash standard reference and revision signal. Deployment follows current crypto policy. |

### Privacy, source rights, legal boundaries, and ethics

| Source | Type | Contribution and limitation |
|---|---|---|
| [EU General Data Protection Regulation](https://eur-lex.europa.eu/eli/reg/2016/679/oj) | Official legislation | Primary privacy principles and duties. Applicability/legal basis are fact-specific. |
| [EDPB basic principles](https://www.edpb.europa.eu/topics/key-gdpr-concepts/basic-principles_en) | Official regulator guidance | Current explanatory privacy principles. Not a deployment determination. |
| [EDPB legal basis topic](https://www.edpb.europa.eu/topics/key-gdpr-concepts/legal-basis_en) | Official regulator guidance | Legal-basis resources. No basis is selected by this blueprint. |
| [India Digital Personal Data Protection Rules, 2025](https://www.meity.gov.in/documents/act-and-policies/digital-personal-data-protection-rules-2025-gDOxUjMtQWa) | Official government publication | Rules and phased commencement signal. Effective provisions need date-specific review. |
| [EU Database Directive](https://eur-lex.europa.eu/eli/dir/1996/9/2019-06-06/eng/) | Official legislation | Database copyright/sui generis framework. National implementation/facts matter. |
| [EU Trade Secrets Directive](https://eur-lex.europa.eu/eli/dir/2016/943/oj/eng/) | Official legislation | Acquisition/use/disclosure framework and lawful/unlawful distinctions. National law/facts matter. |
| [17 U.S.C. § 1201](https://uscode.house.gov/view.xhtml?req=granuleid:USC-prelim-title17-section1201&num=0&edition=prelim) | Official U.S. Code | Anti-circumvention provisions and exceptions. Applicability is fact- and jurisdiction-specific; the blueprint simply prohibits bypass. |
| [Van Buren v. United States](https://www.supremecourt.gov/opinions/20pdf/19-783_k53l.pdf) | Official US Supreme Court opinion | Narrow interpretation of a CFAA phrase in its facts. Not a scraping safe harbor. |
| [hiQ Labs v. LinkedIn](https://cdn.ca9.uscourts.gov/datastore/opinions/2022/04/18/17-16783.pdf) | Official US appellate opinion | Public-web CFAA preliminary-injunction context. Other legal claims/terms remain. |
| [CI Fellows Code of Ethics](https://www.cifellows.com/code-of-ethics) | Professional ethics guidance | Lawful collection, stewardship, professional conduct. Not law or technical standard. |
| [SCIP implementing a CI ethics policy](https://www.scip.org/general/custom.asp?page=implementing-competitive-intelligence-ethics-policy) | Professional guidance | Truthful identity and ethics implementation. Not law. |

### Forecasting, security, observability, and evaluation

| Source | Type | Contribution and limitation |
|---|---|---|
| [Brier (1950), Verification of forecasts expressed in terms of probability](https://journals.ametsoc.org/doi/10.1175/1520-0493%281950%29078%3C0001%3AVOFEIT%3E2.0.CO%3B2) | Original research | Proper scoring foundation for probability forecasts. Requires compatible resolved events. |
| [Mellers et al., Psychological Strategies for Winning a Geopolitical Forecasting Tournament](https://pubmed.ncbi.nlm.nih.gov/25581088/) | Peer-reviewed original research record | Evidence about training/collaboration/updating in geopolitical forecasting. Transfer to market-agent forecasts is unproven. |
| [NIST AI RMF Generative AI Profile](https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence) | Official government guidance | Generative-AI risk categories and actions. Voluntary guidance, not a full control implementation. |
| [OWASP Top 10 for Agentic Applications](https://genai.owasp.org/2025/12/09/owasp-top-10-for-agentic-applications-the-benchmark-for-agentic-security-in-the-age-of-autonomous-ai/) | Community security guidance | Agentic threat taxonomy input. Recent/evolving, not a compliance standard. |
| [OWASP Excessive Agency](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/) | Community security guidance | Least functionality/permissions/autonomy and authorization. Guidance requires system-specific design. |
| [W3C Trace Context](https://www.w3.org/TR/trace-context/) | W3C Recommendation | Distributed correlation and privacy/security considerations. Not auth/audit truth. |
| [OpenTelemetry specification](https://opentelemetry.io/docs/specs/otel/) | Official observability specification | Telemetry model and current version baseline. Evolves. |
| [OpenTelemetry generative AI conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) | Official semantic-convention documentation | GenAI telemetry attribute work. Maturity/status can change; pin locally. |
| [OpenTelemetry trace SDK](https://opentelemetry.io/docs/specs/otel/trace/sdk/) | Official specification | Sampling/processor/export behavior. Demonstrates why traces can be incomplete. |
| [OpenAI: Working with evals](https://developers.openai.com/api/docs/guides/evals) | Official provider documentation | Criteria-driven evaluation workflow. Provider-specific and volatile. |
| [OpenAI: Evaluate agent workflows](https://developers.openai.com/api/docs/guides/agent-evals) | Official provider documentation | Agent workflow evaluation guidance. Does not define domain correctness. |
| [OpenAI: Trace grading](https://developers.openai.com/api/docs/guides/trace-grading) | Official provider documentation | Workflow-trace grading guidance. Trace graders remain non-authoritative. |

## Remaining limitations and implementation research

This packet cannot establish:

- whether a particular website/API/license/contract permits the proposed use;
- which privacy, copyright, database, trade-secret, access, consumer, employment, or sector law applies;
- actual source cadence/latency/quality and parser behavior for a deployment's watchlist;
- a universal definition of materiality or executive usefulness;
- sufficient reviewer capacity and error tolerance;
- model/provider performance on the organization's languages, entities, evidence, attacks, and costs;
- whether durable workflow or multi-agent operation will pay for its complexity;
- numeric SLOs, rate settings, retention periods, risk tiers, or approval thresholds;
- the correct data-residency, DR, and provider architecture for a specific organization.
- live behavior or contractual rights in Google Search, News API, SEC, GLEIF, Eurostat, FRED, USPTO, Reddit, Microsoft Graph, Salesforce, Power BI, Slack or an MCP server/tenant;
- whether the official FRED API and legal-notice tension permits any proposed AI, retention, caching or derived-data workflow.

Before implementation, research each selected source's current API/schema, terms/license, robots/access policy, privacy fields, quota, version/revision behavior, attribution, and deprecation notices. Before production, create the private sequence corpus, perform threat/privacy/source reviews, measure Stage 0, and run the staged gates.

## Refresh checklist

- [ ] Recheck the category registry and adjacent blueprint boundaries.
- [ ] Reopen every official source/API/terms/license/rate page used by active connectors.
- [ ] Recheck Google Custom Search retirement, USPTO ODP migration, provider seasonal releases, Slack/Reddit/FRED terms, and the MCP 2026 release-candidate outcome.
- [ ] Confirm source distributions, fields, schema/taxonomy, authentication, quotas, and historical-version behavior.
- [ ] Reassess robots, access controls, contract/license, database/copyright, privacy, trade-secret, and jurisdiction decisions with accountable owners.
- [ ] Verify current effective privacy/legal provisions, including phased commencement.
- [ ] Review NIST, OWASP, W3C/IETF, XBRL, CloudEvents, OpenTelemetry, and cryptographic-standard maturity/version changes.
- [ ] Recheck model/provider API, retention, tool, structured output, model ID/deprecation, and evaluation documentation.
- [ ] Audit source-policy lineage into snapshots, caches, indexes, evaluations, briefs, backups, and deletions.
- [ ] Re-run adapter qualification for identity, pagination/completeness, revisions/deletes, timestamps, rights, retention/redistribution, tenant permission, effects and recovery.
- [ ] Refresh entity/watchlist ground truth and source-origin relationships.
- [ ] Add reviewed incidents, corrections, misses, false alerts, and attack cases to the permitted test corpus.
- [ ] Re-evaluate deterministic baseline versus model-assisted and multi-worker designs on quality, reviewer time, latency, and cost.
- [ ] Inspect every hard-stop slice rather than relying on aggregate benchmark scores.
- [ ] Record the new research date, changed decisions, accepted behavior differences, and next refresh owner/date.

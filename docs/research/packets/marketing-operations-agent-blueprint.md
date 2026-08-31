# Research Packet: Marketing Campaign Operations Agent Blueprint

> **Research completed:** 2026-08-31  
> **Derived guide set:** [Marketing Campaign Operations Agent](../../agents/marketing-operations-agent/README.md)  
> **Maturity:** Pass 2 research-backed production blueprint; provider-specific adapters, organizational policy, contracts, live accounts, and jurisdictional decisions require local validation  
> **Research posture:** Primary sources first; current provider mechanics and regulator guidance were checked; vendor features are mechanisms rather than application guarantees  
> **Scope:** Campaign objectives, audience/consent/suppression state, content and brand review, channel/ad-platform integration, publication, budget/pacing, experiments, attribution, lead handoff, realized outcomes, durability, security, evaluation, deployment, and governed evolution

## Executive finding

Marketing campaign operations qualifies as a distinct production agent category, but the correct design is a **constrained hybrid workflow**, not an autonomous campaign manager. Models are useful for interpreting incomplete briefs, assembling evidence-bound plans, producing bounded creative variants, explaining provider exceptions, and synthesizing explicitly limited outcomes. Deterministic application services and accountable humans must own audience eligibility, consent and suppression, claims, approval, spend, experiment design, publication, effect reconciliation, and lead handoff.

Five source-backed mechanics drive that conclusion:

1. Channel providers expose materially different draft, validation, review, schedule, batch, partial-failure, quota, reporting, and API-lifecycle semantics.
2. Marketing eligibility depends on purpose, channel, sender, jurisdiction, source/evidence, objections, sensitive data, and freshness; a CRM boolean or model judgment cannot represent it safely.
3. Budget fields are provider-specific delivery controls, not universal hard invoice caps; the application needs independent authorization, reservation, and reconciliation.
4. Attribution reports use mutable vendor-defined models and reporting windows; randomized experiments can support stronger claims only within their design and integrity limits.
5. External sends, publishes, audience uploads, budget changes, and handoffs can be partial or ambiguous after timeouts; durable intent, semantic identity, and reconciliation remain application responsibilities.

## Questions investigated

1. When does campaign work justify a model-directed loop instead of templates, rules, dashboards, or a workflow application?
2. Which workload boundary keeps marketing distinct from sales, competitive intelligence, analytics, executive operations, and editorial ownership?
3. Which records must represent objective, audience, consent/suppression, creative, publication, spend, experiment, attribution, handoff, and outcomes?
4. What do current advertising and marketing-automation APIs actually guarantee about validation, partial failure, scheduling, cancellation, reporting, and versioning?
5. How should budget authority remain bounded when providers pace and bill under product-specific rules?
6. Which experiment and attribution evidence can support operational, observational, or causal conclusions?
7. How do privacy, targeting, advertising, mail-sender, and platform policies constrain audiences and content?
8. What state, context, compaction, memory, planning, security, evaluation, deployment, and incident controls are required from first loop to scaled production?

## Research method

The repository's [research method](../research-method.md), blueprint expansion contract, category registry, cross-cutting control packet, canonical runtime/security/evaluation guides, and adjacent analytics, BI monitoring, competitive intelligence, executive operations, and sales blueprints were inspected first.

Internet research then covered these primary-source families:

1. official Google Ads API mechanics for resources, validation, mutation, partial failure, batch jobs, concurrent changes, change events, budgets, quotas, experiments, conversions, and deprecation;
2. official Mailchimp and HubSpot marketing API documentation for draft/content/checklist/schedule/send/report, audience/webhook, role/limit, dated-version, and preview behavior;
3. FTC, ICO, CRTC, California Attorney General, GDPR/ePrivacy, and mail-provider guidance for advertising claims, commercial messages, consent, objections, suppression, privacy choices, and sender operations;
4. official Google Ads/Analytics materials for personalized audiences, consent signals, attribution models, conversion lag, corrections, and experiment operation;
5. primary experimentation research from Microsoft Research and Google Research for randomization, SRM, interpretation, and geo-experiment evidence;
6. current standards and official guidance for OAuth, agent security, AI/privacy risk, observability, incident response, release provenance, and durable effects;
7. a Pass 2 operation-level provider pass across Meta customer lists, Twilio messaging/consent, Segment consent/audiences, Salesforce and HubSpot CRM handoff, Contentful, Cloudinary, GA4 Data API/BigQuery, sender operations, dark patterns and sensitive targeting.

Search result snippets were used for discovery but material claims were tied to opened primary pages or existing current repository syntheses. No regulator or provider rule was turned into universal legal advice. Provider status at one account/edition/region does not establish behavior elsewhere.

Research was considered saturated for this pass when additional sources stopped changing the category boundary, recommended architecture, authority model, main state/effect contracts, experiment/attribution posture, evaluation suite, and staged roadmap. Saturation is temporary; the refresh triggers below are part of the result.

## Category promotion record

| Gate | Evidence | Result |
|---|---|---|
| Real-agent fit | Multi-step planning across incomplete briefs/evidence and model-assisted exception recovery; environment feedback from content, audience, channel, spend, and outcome systems | **Pass**, when semantic work is measured; deterministic baseline remains required |
| Distinct architecture | Campaign population, public assets, spend, experiments, and attribution differ materially from sales CRM/opportunity, market monitoring, bounded analytics, and personal delegation | **Pass** |
| Buildability | Typed objective/audience/creative/effect/experiment/handoff records; one durable workflow; narrow provider adapters; real-state reconciliation | **Pass** |
| Production depth | Suppression races, partial audience activation, duplicate publication, provider/UI drift, spend overrun, experiment contamination, attribution restatement | **Pass** |
| Evaluation viability | Provider simulators/sandboxes, synthetic audiences, deterministic policy/effect oracles, specialist content/experiment review, repeated fault trials | **Pass** |
| Evidence depth | Current official provider docs, regulator guidance, standards, and primary experimentation research | **Pass with provider-specific limitations** |
| Reader value | Removes recurring architecture, consent/effect, spend, experiment, and operational research burden | **Pass** |

**Promotion decision:** promote as category 31, Marketing campaign operations. The model-directed capability is optional per deployment; the production blueprint remains useful because it proves when the deterministic alternative is better.

## Boundary and closest overlaps

| Area | Marketing owns | Neighbor owns | Enforced separation |
|---|---|---|---|
| Sales/revenue operations | Campaign population, marketing qualification rule, handoff intent/receipt, aggregate outcome feedback | Account/contact truth, ownership, opportunity stages, seller outreach, quote/direct revenue workflow | Marketing connector exposes only lead-intake contract; no opportunity/stage/quote/send tools |
| Competitive intelligence | May consume approved external evidence for a campaign brief | Persistent entity/watchlist monitoring, collection rights, market evidence and briefing | Marketing opens a bounded request; no watchlist/collector authority |
| Analytics | Experiment registration/operation and attribution evidence packaging | Metric semantics, statistical method, bounded analysis, causal interpretation | Versioned analysis/readout request and returned artifact; model cannot invent metric/winner |
| BI monitoring | Campaign-specific pacing/guardrail cases | Persistent governed metric watches and organization-wide decision operations | Marketing watches are scoped to its campaigns and effects |
| Executive operations | Campaign briefing/approval work item | Personal inbox/calendar/delegated action | No personal mailbox/calendar/travel tools |
| Editorial/content governance | Structured brief, draft variants, evidence bindings, review state | Final editorial standards and artifact quality decision | Exact content approval required before publication |

The category differs on at least five registry seams: external environment, financial/public authority, long-running state, real-state outcome proof, and failure recovery.

## Evidence-to-decision record

| Claim or decision | Class | Strong source and research date | Conflict or limitation | Blueprint consequence and refresh trigger |
|---|---|---|---|---|
| Use a deterministic baseline and add agentic complexity only for measured semantic work | Recommendation | Anthropic and OpenAI agent architecture guidance; accessed 2026-08-31 | Provider guidance is general, not marketing-specific | Stage 0/1 gates compare against templates/rules; refresh on major agent guidance/eval evidence |
| Google Ads mutations can validate without commit and can use method-specific partial failure | Mechanic | Google Ads API structure and partial-failure docs; accessed 2026-08-31 | Not every method supports partial failure; validation is not complete policy/legal review | Capability registry records endpoint behavior; reconcile per operation; refresh each API release |
| Google Ads batch jobs can commit some operations even when others fail or job is cancelled | Mechanic | Google Ads batch-processing docs, updated 2026-08 | Batch grouping has resource-specific atomic sub-batches | Never mark whole campaign from job status; read every result and critical resource |
| Provider APIs evolve quickly and need pinned versions/capability tests | Mechanic/recommendation | Google Ads sunset schedule; HubSpot 2026-03 dated docs; accessed 2026-08-31 | Versioning scheme differs by provider; some policy changes are unversioned | Connector manifest, expiry, upgrade fixtures, and write hold on stale evidence |
| A provider campaign budget is not an application hard-cap guarantee | Mechanic/inference | Google Ads budget overview; Microsoft Advertising budget guidance; accessed 2026-08-31 | Billing, campaign type, credits, fees, and account semantics vary | Independent authorization/reservation, conservative exposure, spend reconciliation |
| Consent and suppression require evidence and channel/purpose context | Mechanic/recommendation | FTC, ICO 2026 guidance, CRTC; accessed 2026-08-31 | Jurisdictions differ; counsel must configure actual policy | Versioned evidence ledger and deterministic eligibility decision at commit |
| Customer-list activation has provider-specific source/consent rules and migration advice | Mechanic | Google Customer Match policy/consent docs; accessed 2026-08-31 | Applies to Google product/use cases, not every provider | Explicit provider mapping; no generic audience-upload tool; refresh on policy/API change |
| Claims need substantiation outside provider/model approval | Mechanic/recommendation | FTC advertising guidance and Google Ads policies; accessed 2026-08-31 | Product/sector/jurisdiction may require stronger review | Claim ledger, exact asset approval, provider validation only as supplementary evidence |
| Attribution models assign credit but do not by themselves prove incrementality | Mechanic/inference | GA4 attribution docs and Google geo-experiment research; accessed 2026-08-31 | Vendor data-driven methods may use sophisticated counterfactual modeling but remain product/account scoped | Label model/window/vintage; reserve causal language for justified designs |
| Recent conversions and attribution are revisable | Mechanic | Google Ads conversion-lag/reporting and conversion-adjustment docs; accessed 2026-08-31 | Exact delays vary by event/product | Provisional/mature/restated vintages and append-only corrections |
| SRM must be diagnosed before trusting experiment effects | Observed production result/recommendation | Microsoft Research SRM paper and production article; accessed 2026-08-31 | Evidence comes from online experiments broadly, not every ad platform | SRM is a hard experiment-integrity gate; adapt test to assignment design |
| Email APIs separate draft/schedule/send and can continue work after client timeout | Mechanic | Mailchimp API/reference/fundamentals; accessed 2026-08-31 | Plan, role, campaign type, and provider behavior vary; not a universal idempotency guarantee | Prepare/approve/commit, provider IDs, unknown state, read-back before resend |
| Webhooks are change signals, not durable business truth | Mechanic/inference | Mailchimp webhook retry docs and HubSpot webhook/journal docs; accessed 2026-08-31 | Provider delivery/replay/signature mechanics differ | Durable inbox, signature/dedupe, authoritative read, periodic reconciliation |
| Model/provider conversation state is not campaign state | Recommendation | Canonical durable execution/state guides and provider API boundaries; reviewed 2026-08-31 | A framework can durably checkpoint parts of execution | Application state/events/effects remain authoritative; resume from current facts |
| General tool annotations/discovery cannot establish write safety | Recommendation | Cross-cutting controls and MCP schema caveat; reviewed 2026-08-31 | A trusted internal registry can use tested metadata | Dynamic plugins/MCP writes rejected unless admitted, pinned, wrapped, and tested |
| Raw telemetry content creates privacy/security risk | Mechanic/recommendation | W3C Trace Context, OpenTelemetry GenAI/Collector guidance; accessed/reviewed 2026-08-31 | GenAI conventions remain evolving | Stable local IDs, content capture off, ledgers separate from sampled traces |
| Release evaluation must inspect trajectories and real state with multiple grader types | Recommendation | Anthropic agent eval guidance, NIST AIRC/AI RMF; accessed 2026-08-31 | Model graders remain task- and calibration-dependent | Deterministic hard gates, specialist human review, repeated trials and failure injection |

## Current-version and volatility baseline

| Area | Baseline observed on 2026-08-31 | Operational consequence |
|---|---|---|
| Google Ads API | v25.1 was the latest published minor version; v25.1 release notes were dated 2026-08-19 and v25 sunset was listed for August 2027 | Pin version per adapter and monitor sunset/feature-deprecation pages; never write “latest” into durable runs |
| Google Ads batches | Asynchronous batch jobs return per-operation results and use partial-failure semantics; concurrency on the same objects is discouraged | Per-account/resource fencing, job-result reconciliation, bounded polling |
| Google Ads budgets | Average daily and total budget models are documented; total-budget availability depends on campaign type | Capability tests and independent flight-cap accounting |
| Google Ads experiments | Multiple current workflows—system-managed, intra-campaign, asset optimization, campaign mix—have different resource/configuration rules | Treat provider experiment type as a connector capability, not one generic API |
| Google Customer Match | Current help recommends Data Manager API for new workflows and documents first-party/consent constraints | Do not build a new generic Customer Match integration against a deprecated direction without rechecking |
| HubSpot marketing APIs | 2026-03 dated paths and recent migration notes were published; features/scopes/entitlements differ | Pin dated routes, account capability discovery, changelog monitoring |
| HubSpot CRM object API | 2026-03 batch upsert supports a custom unique `idProperty`; email-based contact upsert does not support partial upsert | Prefer a dedicated handoff external ID; preserve per-record outcome and sales-owned field boundary |
| Mailchimp Marketing API | v3.0 reference observed as 3.0.91; newer Audiences surfaces were marked beta/evaluation and described consent-mapping limits | Stable campaign APIs may be used after tests; preview audience path excluded from critical suppression flow by default |
| Meta Marketing API | Customer List Custom Audience terms were current public terms; official SDKs are versioned; the Marketing API access-tier requirements/name changed in May 2026 | Qualify exact Graph/Marketing version, app review/tier, account, scopes, batch/rate behavior and audience rights; hashing does not remove consent/opt-out duties |
| Twilio messaging/consent | Advanced Opt-Out is configuration-, sender- and country-sensitive; blocked-number state is not generally administered/reported through that feature's REST surface; Consent Management API access/enablement can vary | Treat provider block/consent as one enforcement surface, not enterprise consent truth; ingest events and reconcile with the central ledger |
| Segment consent/audiences | Consent storage/enforcement for Profiles/Engage was documented as public beta with missing-category and product-coverage semantics | Do not make beta profile consent the sole suppression gate; test identity merge, category mapping and destination sync |
| Salesforce REST | Current REST guide exposed v66.0 examples; upsert by external ID and Composite have operation-specific create/update/rollback behavior | Pin org API version, external-ID field, field permissions, duplicate rules and `allOrNone` behavior; no opportunity mutation from marketing |
| Contentful CMA | Optimistic locking uses current resource version; update sends the complete entry body | Fetch-update with `X-Contentful-Version`; separate draft, publish and observed destination; prevent stale partial overwrite |
| Cloudinary Admin API | Docs updated 2026-08-24 describe a powerful rate-limited API, partial paged deletion, delayed CDN invalidation and irreversible backup deletion | Read-only DAM adapter by default; deletion/admin remains separately approved and absent from model workers |
| GA4 Data/BigQuery | Data API v1 has property/project token/concurrency/error quotas and possible thresholding; intraday BigQuery export is best effort and daily export is the stable day | Persist property/metric/query/timezone/quota and report vintage; never use provisional/thresholded data as causal or complete truth |
| Gmail sender operations | Current sender guidance includes authentication, reputation, and one-click unsubscribe requirements for relevant bulk marketing traffic, with enforcement updates | Treat deliverability rules as operational policy with monitoring, not static legal compliance |
| ICO direct marketing | Electronic-mail guidance was updated 2026-04-28, illustrating live legal/guidance evolution | Jurisdiction policy bundle needs owner and immediate refresh path |
| GA4 attribution | Three attribution models are currently documented; several older rule-based models were removed in 2023 | Persist model/configuration and split analytical vintages across changes |
| OpenTelemetry GenAI | Conventions remain evolving | Pin local mapping; no raw prompt/tool content by default |

All exact versions, quotas, entitlements, regions, and policy status must be rechecked against the target account at implementation and release time.

## Provider and integration findings

### Google Ads mutation and recovery

Google Ads resources have canonical resource names. Most mutate requests offer a `validate_only` switch, while partial failure is method-specific. Partial-failure responses map errors to operation indices; dependent creations should generally be submitted atomically where required. BatchJobService runs asynchronously, may retry some transient failures, and always permits successful members to remain committed when other operations fail or the job is cancelled. It returns result status per operation.

The API also documents concurrent-modification errors rather than an object-locking primitive. Change events expose old/new values and actor/client information but are constrained to a recent window and row limit. These mechanics support the blueprint's effect ledger, resource fences, request/resource evidence, per-operation reconciliation, and provider-state read-back; they do not provide an end-to-end exactly-once campaign transaction.

### Google Ads budgets, experiments, and reporting

Google distinguishes average daily from campaign total budgets and documents campaign-type restrictions. Its experiment workflows split traffic through different product-specific mechanisms and provide explicit end/promote/graduate operations. Current experiment guidance recommends a clear hypothesis, preselected metrics, controlled variables, and avoiding uncontrolled base-campaign edits.

Conversion reporting is not instant. Google documents conversion lag and different report-time/event-time views, while conversion adjustments can retract or restate previous observations. Change and report data therefore need event time, reporting time, ingestion time, model/window, and vintage.

### Mailchimp

The Marketing API exposes campaign create/update/content, checklist, test, schedule, unschedule, send, cancel, and report resources. Campaign state distinguishes saved, scheduled, sending, and sent. API documentation states a simultaneous-connection limit, may return 429, and warns that a request can time out while backend work continues. Audience webhooks retry over a bounded period and can be signed.

The current reference explicitly labels newer Audiences endpoints as beta/evaluation and notes unsupported consent values, including opt-outs, may need manual handling. This is sufficient evidence to reject that preview path as the sole suppression/consent authority for a production first release.

### HubSpot

HubSpot's marketing-email documentation distinguishes marketing from sales email and documents dated API paths, scopes, publish entitlements, object timestamps, and field migrations. Its webhook and journal APIs document request limits, offsets/snapshots, signature mechanisms, and provider-specific operational guidance.

The main conclusion is not that one HubSpot endpoint is universally preferred. It is that dated path, brand/business-unit, entitlement, scope, webhook, and migration behavior must live in a connector capability record and contract tests rather than a model prompt.

### Meta and other ad providers

Meta's current public Customer List Custom Audience Terms require the advertiser or acting party to have necessary rights/permissions/lawful basis, honor relevant opt-outs and remove people after later opt-out. Hashing happens before upload but does not erase those obligations. Meta's official Business SDKs are versioned and document batch calls as a network optimization whose members still count toward rate limits. In May 2026 Meta renamed its Ads Management Standard Access feature to Marketing API Access Tier and revised the displayed access qualification criteria.

These sources establish rights and access-tier constraints, not a universal mutation/idempotency/budget/reporting contract. A production adapter still needs the current Graph/Marketing version, app review/tier, permissions, account hierarchy, API fields, async/batch outcomes, rate headers, cancellation, reporting and deletion tests from the target account. The same restraint applies to LinkedIn, TikTok, X and regional platforms not deeply researched in this packet.

### CRM, CDP, consent and lead routing

Salesforce REST upsert uses an external-ID field to select insert versus update; Composite can make subrequest rollback/dependency behavior configurable. HubSpot's 2026-03 object API supports batch upsert with a custom unique property but documents that email-based contact upsert cannot be partial. Therefore `handoff_id`, not email, is the primary dedupe identity, and the marketing capability is limited to a minimal intake object/fields. Account, opportunity, stage, owner and seller communication remain sales-owned.

Twilio Advanced Opt-Out reports `STOP`, `START` and `HELP` through `OptOutType` under documented configuration, while blocked-number reporting/admin limitations and sender/country behavior prevent it from being the enterprise preference ledger. Twilio Segment documents consent on Profiles and Engage audience enforcement as public beta with missing-category, identity-merge, destination-mapping and unsupported-product implications. Provider consent is an enforcement projection; the central ledger retains purpose, controller/sender, channel, source evidence, effective time, objection and correction history.

### CMS, DAM and analytical adapters

Contentful's CMA uses optimistic locking and full-body entry updates, so an adapter must fetch the current version and cannot safely patch a stale subset. Cloudinary's Admin API has broad control: deletion can be paginated/partial, CDN invalidation is delayed/configuration-sensitive, and backup deletion can be irreversible. The model receives asset/version/digest refs; administrative DAM deletion is not an ordinary campaign tool.

GA4 Data API v1 quotas apply by property/project and can include thresholded dimensions. BigQuery streaming export is best effort and lacks some user-attribution completeness; the full daily table is the stable day, and UI/export values can differ. Outcome records therefore pin property, metric definition, query, attribution model/window, threshold/sampling status, event/report/ingestion time and vintage. Neither surface supplies incremental causality.

## Consent, suppression, privacy, and sender findings

The FTC states that CAN-SPAM applies to US commercial email, including B2B messages, and includes truthful headers/subjects, opt-out, and monitoring of vendors. ICO's current UK guidance emphasizes channel context, consent/soft-opt-in conditions, objections, and keeping minimum suppression records rather than simply deleting addresses that may later be reimported. CRTC guidance distinguishes express and implied consent and requires identification/unsubscribe for covered commercial electronic messages.

These sources disagree in scope because their laws and recipient/channel contexts differ. The resolved design is a versioned, counsel-approved policy function over sender, recipient/contact point, channel, purpose, jurisdiction inputs, relationship/evidence, objections, source rights, campaign, time, and provider rules. Unknown required facts deny or route to review. Suppression always dominates campaign optimization.

California's GPC guidance demonstrates that online privacy choices can affect sale/sharing-related processing. Google's Customer Match and Consent Mode materials demonstrate that provider consent signals have product-specific meaning and that Consent Mode is not a consent banner. The application data map must cover audience activation, tags, conversion uploads, lead handoff, traces, evals, and backups—not only outbound email.

Gmail's current bulk-sender materials connect authentication, DMARC alignment, spam reputation, and one-click unsubscribe to delivery. These are operational safety/reputation controls in addition to legal eligibility. A legally permitted message can still be unwanted or undeliverable; compliance and deliverability remain separate gates.

## Content and brand findings

FTC advertising guidance requires truthful, non-deceptive, evidence-backed claims, with additional product/sector rules where applicable. Google Ads policy documentation similarly identifies misrepresentation, data-use, sensitive targeting, and restricted-category requirements. Provider review is valuable but not a substitute for the advertiser's claim substantiation, offer availability, disclosure, brand, accessibility, or destination review.

The blueprint therefore uses a claim ledger with scope/expiry/qualifiers, immutable creative revisions, exact approval digests, resolved destination/render packages, provider dry-run/checklist evidence, and change/drift detection. Public correction or withdrawal is a new effect; history is not silently rewritten.

## Experiment and attribution findings

Google Ads experiments provide operational A/B mechanisms, but platform UI significance is not a complete experimentation governance program. Microsoft Research documents SRM as a signal of assignment or data-quality failures that can reverse decisions if ignored. Its metric-pitfall work also shows that trustworthy experimentation requires more than a displayed uplift. Google Research's geo-experiment work illustrates randomized regional assignment for incremental ad-effect evidence.

GA4 attribution documentation defines attribution as assigning credit and currently exposes data-driven and two last-click models. The documentation notes account-specific modeling, path scope, and reattribution behavior. This supports an evidence ladder:

1. delivery/interaction;
2. first-party conversion event;
3. provider-attributed observation under named model/window;
4. cross-channel observational attribution under explicit assumptions;
5. randomized incremental effect for its tested population/time/design;
6. governed long-horizon synthesis such as calibrated MMM, with stated model assumptions.

The agent may assemble and explain these records but cannot change the metric family, peek/stopping policy, or causal label after observing results. Promotion and budget change remain accountable human effects.

## Security, state, and operational findings

OAuth, provider app scopes, and workload identity authenticate actors but do not encode campaign delegation. The application must bind tenant, brand, provider account, operation, audience/data class, destination, amount, campaign/run, time, and approval. D4 changes—credentials, account links, billing authority, suppression/policy/audit safeguards, or privileged tool installation—remain separate administrative work.

Untrusted marketing content has unusually direct paths to dangerous sinks. External pages, UGC, form text, CRM notes, media metadata, provider recommendations, errors, webhooks, and plugin descriptions must remain data. Model-layer defenses are supplementary; narrow egress, absent credentials, typed tools, deterministic policy, exact approvals, and real-state reconciliation provide containment.

Durable workflow products can persist progress but do not make arbitrary provider effects exactly once. The blueprint retains `unknown`, uses semantic effect keys, fences, provider idempotency only where documented, per-member/per-operation results, read-back, and separate reconciliation.

## Material disagreements and resolved positions

### Platform acceptance versus complete approval

**Disagreement:** A provider validator/review may be treated as sufficient to publish, while brand/legal teams may require separate review.

**Resolution:** provider acceptance is one input. Application review remains responsible for claims, brand, accessibility, privacy, offer, destination, and organizational policy. Review scope is risk-based, not universally manual for every low-risk edit.

### Platform attribution versus causal incrementality

**Disagreement:** Vendor data-driven attribution uses sophisticated modeling and may describe contribution; experimentation sources prioritize randomization for causal claims.

**Resolution:** store vendor attribution faithfully as a model-defined observation. Use incremental/causal wording only when the design and integrity checks justify it. Compare methods rather than force them into one number.

### Provider budget versus financial cap

**Disagreement:** Campaign UIs call a field “budget,” implying it bounds spend; provider docs describe average/target behavior and product-specific charging.

**Resolution:** treat provider budget as a delivery parameter. The application maintains a stricter financial authorization/reservation and reconciles report/invoice-quality data.

### Human approval for every change versus bounded automation

**Disagreement:** Approval-everything prevents some errors but creates fatigue and timing failures; autonomous optimization promises speed.

**Resolution:** D0/D1 and bounded D2 can be automatic. D3 needs an exact approval or a deterministic preauthorized runbook whose selectors, thresholds, cap, rollback, and evidence are equivalent. Models never widen the envelope.

### Unified connector abstraction versus provider-native semantics

**Disagreement:** A generic campaign tool simplifies prompts; provider-specific code increases implementation cost.

**Resolution:** expose a small internal capability vocabulary, but keep versioned provider adapters and capability records. Do not erase partial failure, account hierarchy, budget type, review, cancellation, or report-vintage differences.

### Persistent memory versus governed campaign knowledge

**Disagreement:** Long-term memory may personalize drafting and reuse successful campaigns; it also creates stale, poisoned, cross-purpose, privacy-sensitive behavior.

**Resolution:** use governed brand/claim/configuration records and curated episodic outcome artifacts. Reject free-form user/account memory by default. No conversion feedback writes directly to memory or policy.

### Optimize continuously versus protect experiment integrity

**Disagreement:** Channel platforms encourage continuous optimization, while experiments require stable treatment definitions and assignments.

**Resolution:** register which platform optimization is part of the treatment. Unregistered creative, targeting, budget, or base-campaign changes can hold or invalidate the experiment. Operational safety pauses still take precedence and are recorded.

## Rejected designs

| Rejected design | Why | Safer alternative |
|---|---|---|
| One “marketing agent” with browser, CRM, CDP, email, ads, social, and analytics admin access | Composes private data, arbitrary content, public reach, and spend into a large exfiltration/effect path | One campaign workflow, no-authority reasoning, narrow account-bound adapters |
| Campaign goal as blanket approval | Omits audience, claim, sender, destination, spend, schedule, and experiment facts | Exact versioned contracts and per-effect authorization |
| CRM `marketable` boolean | Loses purpose, channel, sender, evidence, jurisdiction, history, and objection precedence | Consent/suppression ledger and current eligibility decision |
| Raw audience list in prompt for personalization | Privacy, leakage, poisoning, and no need for model access | Protected audience service; aggregates/synthetic samples only |
| Retry send/publish/budget POST after timeout | Can duplicate or compound external effects | `unknown` state, provider ID/change/status reconciliation, manual review if necessary |
| Provider batch/job `DONE` means campaign success | Partial results and unrolled-back successes exist | Per-operation ledger and critical-resource read-back |
| Platform budget as hard cap | Provider-specific pacing/billing semantics | Internal reservation and conservative exposure accounting |
| Platform-attributed revenue as incremental truth | Model/window/path bias and corrections | Named attribution evidence plus experiments/analytics for causal questions |
| Model-declared experiment winner | Metric choice, peeking, integrity, and interpretation need governed analysis | Registered plan, deterministic integrity checks, analytics owner and promotion approval |
| Autonomous audience expansion from outcomes | Feedback loops, bias, consent/purpose drift | Governed feature/evaluation release with privacy and causal review |
| Shared campaign vector memory across tenants | Cross-tenant/purpose leak and stale brand/policy | Tenant/purpose-filtered governed retrieval with deletion |
| General dynamic plugins for live writes | Discovery metadata is not proof of publisher, permissions, or idempotency | Admitted/pinned adapters wrapped by local effect policy |
| Multi-agent marketing swarm | Shared evidence and correlated errors with greater latency/authority complexity | Single workflow plus deterministic validators and accountable human reviewers |

## Pass 2 production-depth decisions

| Gap closed | Decision | Implemented in |
|---|---|---|
| Canonical identity | Tenant, brand, objective, campaign, audience, consent, suppression, segment, asset, claim, channel, destination, budget, experiment, conversion, lead, handoff, approval and effect have independent local/version identities | [Adapter qualification and worked lifecycle](../../agents/marketing-operations-agent/10-adapter-qualification-and-worked-campaign-lifecycle.md#identity-and-version-semantics) |
| Connector proof | Admit one account-scoped operation only after schema, rights, consent, population, finality, failure, deletion, security and recovery conformance tests | [Operation-level capability manifest](../../agents/marketing-operations-agent/10-adapter-qualification-and-worked-campaign-lifecycle.md#operation-level-capability-manifest) |
| Memory | Use exactly seven canonical lifetimes with explicit use/reject, retention/deletion, poisoning and evaluation control; domain systems are not extra model memory | [Exactly seven memory lifetimes](../../agents/marketing-operations-agent/06-state-context-memory-and-orchestration.md#exactly-seven-memory-lifetimes) |
| Compaction/restart | Pin high-watermark, versions, approvals/clocks, pending/unknown effects, invariant hash, omitted refs, next safe action and deterministic restart checks | [Context compaction](../../agents/marketing-operations-agent/06-state-context-memory-and-orchestration.md#context-compaction) |
| Effects | Stable semantic identity, prepare/fence/dispatch/readback, `unknown`, reconciliation before retry, separate compensation and cancellation semantics | [Worked launch and recovery](../../agents/marketing-operations-agent/10-adapter-qualification-and-worked-campaign-lifecycle.md#phase-8--unknown-effect-reconciliation-and-rollback) |
| Safety/governance | Purpose/suppression, discrimination, claims/dark patterns, injection, tenant/secrets, least privilege/SoD, deliverability/abuse and supply chain are non-compensating gates | [Security gate](../../agents/marketing-operations-agent/10-adapter-qualification-and-worked-campaign-lifecycle.md#security-privacy-fairness-and-deliverability-gate) |
| Evaluation | Separate outcome, trajectory, invariant and human-factor lenses; compare deterministic/manual baselines; use counterfactual/holdout evidence and time/tenant leakage controls | [Evaluation lenses](../../agents/marketing-operations-agent/08-reliability-observability-evaluation-and-incidents.md#four-evaluation-lenses-and-human-calibration) |
| Operations/evolution | Separate evidence/audit/trace/log/SLO planes, size recovery for new arrivals and human review, exercise DR, bundle behavior, canary/rollback and control failure mining | [Scale/evolution](../../agents/marketing-operations-agent/09-deployment-scale-cost-and-evolution.md) and [Stage exercises](../../agents/marketing-operations-agent/10-adapter-qualification-and-worked-campaign-lifecycle.md#stage-06-proof-exercises) |

## Remaining limitations

- This packet does not decide the legal basis or permitted targeting/communication for any organization. Counsel and privacy owners must map actual senders, products, channels, audiences, jurisdictions, data sources, contracts, and platform terms.
- Google, Meta, Mailchimp, HubSpot, Twilio/Segment, Salesforce, Contentful, Cloudinary and GA4 documentation establishes selected mechanics, not a certification of a live account, contract, region, entitlement, right, quota or deployment configuration. LinkedIn, TikTok, X and regional channels require their own primary-source pass.
- Advertising-platform budget, billing, automated bidding/creative, experiment, attribution, and policy-review semantics vary by campaign type, account, region, eligibility, and release.
- No universal numeric approval threshold, suppression freshness SLO, complaint/bounce limit, pacing buffer, experiment power, or cost target can be set without local policy and historical data.
- Provider sandboxes often differ from production. Canary accounts and real read-back tests remain necessary.
- Public benchmark tasks do not capture an organization's brands, claims, audiences, provider configurations, policy, or reviewer expectations. Local held-out fixtures are required.
- Marketing-mix modeling was researched only enough to position it as governed analytical evidence. A production MMM implementation belongs to an analytics-specific research and validation effort.
- The architecture does not guarantee compliance, deliverability, factuality, incrementality, or security merely by naming controls. Each control needs an owner, enforcement mechanism, observable evidence, and passing test.

## Source register

All web sources were accessed or rechecked on **2026-08-31** unless a publication/update date is stated. Provider pages can change without preserving old versions; production connectors should archive permitted schemas/configuration evidence and monitor changelogs.

### Agent architecture, evaluation, and governance

1. [Anthropic — Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — workflow versus agent complexity and simplest-effective-system guidance.
2. [Anthropic — Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — task/trial/trajectory/outcome and mixed grader guidance.
3. [OpenAI — A practical guide to building agents](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/) — agent qualification, tools, guardrails, and human intervention.
4. [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework) — lifecycle governance baseline.
5. [NIST AI 600-1: Generative AI Profile](https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence) — GenAI risk actions and maturity context.
6. [NIST AI Resource Center](https://airc.nist.gov/) — testing/evaluation/verification/validation resources.

### Google Ads API, budgets, audiences, and change mechanics

7. [Google Ads API structure and validation](https://developers.google.com/google-ads/api/docs/concepts/api-structure) — resource hierarchy, synchronous mutation, concurrency, `validate_only`.
8. [Google Ads partial failures](https://developers.google.com/google-ads/api/docs/best-practices/partial-failures) — method-specific partial failure and dependent-operation caveats.
9. [Google Ads batch processing](https://developers.google.com/google-ads/api/docs/batch-processing/overview) — asynchronous jobs, automatic transient retry, partial results without rollback.
10. [Google Ads batch usage flow](https://developers.google.com/google-ads/api/docs/batch-processing/flow) — create, upload, run, poll, and per-operation results.
11. [Google Ads batch best practices](https://developers.google.com/google-ads/api/docs/batch-processing/best-practices) — concurrency, ordering, limits, polling, and execution bounds.
12. [Google Ads API quotas](https://developers.google.com/google-ads/api/docs/best-practices/quotas) — access-level, request, service, and response constraints.
13. [Google Ads API errors and request IDs](https://developers.google.com/google-ads/api/docs/best-practices/understand-api-errors) — canonical error handling and request correlation.
14. [Google Ads change events](https://developers.google.com/google-ads/api/docs/change-event) — old/new values, actor/client evidence, recent-window and row constraints.
15. [Google Ads API sunset schedule](https://developers.google.com/google-ads/api/docs/sunset-dates) — current release/deprecation/sunset lifecycle.
16. [Google Ads campaign budget overview](https://developers.google.com/google-ads/api/docs/campaigns/budgets/overview) — average daily versus total budgets and documented charging behavior.
17. [Google Ads budget restrictions](https://developers.google.com/google-ads/api/docs/campaigns/budgets/restrictions-errors) — experiment/shared-budget and lifecycle restrictions.
18. [Google Ads Customer Match policy](https://support.google.com/google-ads/answer/6299717) — first-party context, privacy disclosure, consent, approved upload paths, and current Data Manager direction.
19. [Google Ads consent for Customer Match](https://support.google.com/google-ads/answer/14546648) — provider-specific EEA consent fields and upload behavior.
20. [Google Ads personalized-ad data use](https://support.google.com/adspolicy/answer/6242605) — first/third-party data and audience restrictions.
21. [Google Ads policy overview](https://support.google.com/adspolicy/answer/6008942) — data use, misrepresentation, sensitive/restricted categories and destinations.
22. [Google Ads ad policy fields](https://developers.google.com/google-ads/api/fields/v25/ad_group_ad) — review, approval, and policy-topic status evidence.

### Marketing email, automation, and sender operations

23. [Mailchimp Marketing API](https://mailchimp.com/developer/marketing/api/) — campaign, content, feedback, checklist, audience, report, and action surface; preview audience caveats.
24. [Mailchimp Marketing API fundamentals](https://mailchimp.com/developer/marketing/docs/fundamentals/) — v3, auth/role, limits, timeouts, batching, and webhooks.
25. [Mailchimp API errors](https://mailchimp.com/developer/marketing/docs/errors/) — error format, request IDs, throttling, timeout and test-error behavior.
26. [Mailchimp schedule campaign](https://mailchimp.com/developer/marketing/api/campaigns/schedule-campaign/) — explicit schedule action.
27. [Mailchimp audience webhook synchronization](https://mailchimp.com/developer/marketing/guides/sync-audience-data-webhooks/) — subscription/unsubscribe events, retry and signature behavior.
28. [HubSpot Marketing Email API](https://developers.hubspot.com/docs/api-reference/legacy/marketing/marketing-emails/guide) — scopes, marketing-versus-sales boundary, publish entitlements, and field migration.
29. [HubSpot 2026-03 marketing-email create API](https://developers.hubspot.com/docs/api-reference/latest/marketing/marketing-emails/create-email) — current dated route and object surface.
30. [HubSpot webhooks guide](https://developers.hubspot.com/docs/api-reference/latest/webhooks/guide) — current subscription API surface.
31. [HubSpot webhook journal](https://developers.hubspot.com/docs/api-reference/latest/webhooks-journal/guide) — offsets, snapshots, rate limits, and operational guidance.
32. [HubSpot API usage guidelines](https://developers.hubspot.com/docs/developer-tooling/platform/usage-guidelines) — version/plan/app-dependent limits and headers.
33. [Gmail sender guidelines](https://support.google.com/mail/answer/81126) — authentication, alignment, reputation, and one-click unsubscribe requirements.
34. [Gmail sender FAQ](https://support.google.com/mail/answer/14229414) — enforcement and bulk-sender operational details.
35. [RFC 8058: One-click unsubscribe](https://datatracker.ietf.org/doc/html/rfc8058) — standardized list-unsubscribe POST mechanism.

### Consent, privacy, claims, and advertising policy

36. [FTC CAN-SPAM compliance guide](https://www.ftc.gov/business-guidance/resources/can-spam-act-compliance-guide-business) — US commercial-email requirements and outsourced-sender responsibility.
37. [FTC advertising and marketing guidance](https://www.ftc.gov/business-guidance/advertising-marketing) — truthful, non-deceptive, evidence-backed advertising baseline.
38. [FTC Endorsement Guides](https://www.ftc.gov/system/files/ftc_gov/pdf/P204500%20Guides%20Concerning%20Endors%20and%20Testimonials.pdf) — current endorsement/testimonial regulatory guide.
39. [ICO electronic-mail marketing guidance](https://ico.org.uk/for-organisations/direct-marketing-and-privacy-and-electronic-communications/guidance-on-direct-marketing-using-electronic-mail/) — UK PECR detail and 2026 update.
40. [ICO plan direct marketing](https://ico.org.uk/for-organisations/direct-marketing-and-privacy-and-electronic-communications/direct-marketing-guidance/plan-direct-marketing/) — channel-specific consent, source, accountability, and suppression questions.
41. [ICO respect people's preferences](https://ico.org.uk/for-organisations/direct-marketing-and-privacy-and-electronic-communications/direct-marketing-guidance/respect-peoples-preferences/) — objections and minimum suppression records.
42. [CRTC CASL consent guidance](https://crtc.gc.ca/eng/com500/guide.htm) — express/implied consent, identification, and unsubscribe.
43. [California Attorney General: Global Privacy Control](https://www.oag.ca.gov/privacy/ccpa/gpc) — opt-out signal under California law for covered businesses.
44. [GDPR Article 5](https://eur-lex.europa.eu/eli/reg/2016/679/art_5/oj) — purpose, minimization, accuracy, storage, security, and accountability principles where applicable.
45. [Google Analytics Consent Mode](https://support.google.com/analytics/answer/10000067) — provider signal behavior and explicit statement that it is not a consent banner.
46. [NIST Privacy Framework](https://www.nist.gov/privacy-framework/privacy-framework) — privacy-risk management baseline.

### Experiments, attribution, and outcomes

47. [Google Ads experiments overview](https://developers.google.com/google-ads/api/docs/experiments/overview) — current experiment workflow types and lifecycle.
48. [Google Ads experiment guidance](https://support.google.com/google-ads/answer/7281575) — hypotheses, variable control, preselected metrics, and recordkeeping.
49. [Google Ads experiment monitoring](https://support.google.com/google-ads/answer/6318747) — provider scorecards, confidence intervals, and inconclusive states.
50. [Microsoft Research: Diagnosing SRM](https://www.microsoft.com/en-us/research/publication/diagnosing-sample-ratio-mismatch-in-online-controlled-experiments-a-taxonomy-and-rules-of-thumb-for-practitioners/) — production taxonomy and consequences of assignment/data mismatch.
51. [Microsoft Research: Metric interpretation pitfalls](https://www.microsoft.com/en-us/research/publication/a-dirty-dozen-twelve-common-metric-interpretation-pitfalls-in-online-controlled-experiments/) — common errors in experiment conclusions.
52. [Google Research: Measuring ad effectiveness using geo experiments](https://research.google/pubs/measuring-ad-effectiveness-using-geo-experiments/) — randomized regional ad-effect design.
53. [Google Analytics attribution](https://support.google.com/analytics/answer/10596866) — current attribution models, path/model mechanics, and deprecations.
54. [Google Ads conversion reporting](https://developers.google.com/google-ads/api/docs/conversions/reporting) — metrics mapping and non-instant data.
55. [Google Ads conversion lag](https://support.google.com/google-ads/answer/9347141) — conversion-delay impact on recent CPA/ROAS.
56. [Google Ads conversion adjustments](https://developers.google.com/google-ads/api/reference/rpc/v25/ConversionAdjustmentTypeEnum.ConversionAdjustmentType) — retraction and restatement semantics.

### Identity, security, effects, telemetry, release, and incidents

57. [OAuth 2.0 Security Best Current Practice, RFC 9700](https://datatracker.ietf.org/doc/html/rfc9700) — token/client/redirect/audience security.
58. [OWASP AI Agent Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html) — agent threats and layered control guidance.
59. [OWASP Prompt Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html) — direct/indirect injection controls.
60. [AWS Builders' Library: Making retries safe with idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/) — semantic request identity, late requests, and parameter equivalence.
61. [W3C Trace Context](https://www.w3.org/TR/trace-context/) — trace propagation and sensitive-data constraints.
62. [OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) — evolving GenAI telemetry vocabulary.
63. [OpenTelemetry Collector security guidance](https://opentelemetry.io/docs/security/config-best-practices/) — authentication, encryption, least privilege, and data minimization.
64. [NIST SP 800-61 Rev. 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final) — incident response integrated into cybersecurity risk management.
65. [Google SRE Workbook: Canarying releases](https://sre.google/workbook/canarying-releases/) — staged release evidence.
66. [Google SRE Workbook: Incident response](https://sre.google/workbook/incident-response/) — operational incident roles and process.
67. [SLSA v1.2](https://slsa.dev/spec/v1.2/) — provenance vocabulary and release evidence for software artifacts.

### Pass 2 adapter and provider qualification

| Primary source | Pass 2 contribution | Limitation retained |
|---|---|---|
| [Google Ads API release notes](https://developers.google.com/google-ads/api/docs/release-notes) | v25.1 date/features and current version evidence | Version currency does not prove one account's enabled campaign types, quotas or field behavior |
| [Meta Customer List Custom Audience Terms](https://www.facebook.com/legal/terms/customaudience/update) | Rights/lawful-basis, opt-out removal, hashing and sharing restrictions | Terms do not document mutation idempotency, account configuration or complete law |
| [Meta Python Business SDK](https://github.com/facebook/facebook-python-business-sdk) | Official versioned SDK and batch/Marketing API integration surface | SDK wrapper does not guarantee domain finality, exactly-once effects or permitted use |
| [Meta Marketing API Access Tier update](https://developers.meta.com/blog/updates-to-ads-management-standard-access-feature/) | May 2026 rename and access-qualification change | App-dashboard state and granted permissions remain account-specific |
| [Twilio Advanced Opt-Out](https://www.twilio.com/docs/messaging/tutorials/advanced-opt-out) | Keyword, `OptOutType`, sender/country and blocked-number limitations | Provider enforcement is not enterprise consent/legal truth and configurations vary |
| [Twilio Consent Management API](https://www.twilio.com/docs/messaging/features/consent-api) | Cross-channel consent-state operation surface | Enablement, pricing, evidence and jurisdictional sufficiency are deployment-specific |
| [Twilio Messaging Policy](https://www.twilio.com/en-us/legal/messaging-policy) | Sender identification and opt-out operational policy | Provider policy does not replace applicable law or organizational consent policy |
| [Segment consent on Profiles](https://www.twilio.com/docs/segment/privacy/consent-management/consent-in-unify) | Category, event, missing value, identity-merge and beta semantics | Public beta; not sole source of truth for all environments/use cases |
| [Segment consent in Engage audiences](https://www.twilio.com/docs/segment/privacy/consent-management/consent-in-engage) | Destination mapping, missing preference and product-coverage behavior | Public beta with unsupported audience/product classes |
| [Salesforce REST API guide](https://resources.docs.salesforce.com/latest/latest/en-us/sfdc/pdf/api_rest.pdf) | Current REST version examples and external-ID upsert | Org schema, permissions, duplicate rules and API allocation vary |
| [Salesforce Composite](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_composite_composite_post.htm) | Dependency and `allOrNone` semantics for grouped handoff operations | Not every API/resource is supported; one HTTP call is not marketing-to-sales transaction finality |
| [HubSpot 2026-03 object API](https://hubspot.mintlify.io/api-reference/crm-objects-v2) | Unique `idProperty`, batch upsert and email-partial-upsert limitation | Portal object/property schema, entitlements and sales-owned fields need local contract |
| [HubSpot Marketing Email API](https://developers.hubspot.com/docs/api-reference/latest/marketing/marketing-emails/guide) | Current dated route, business-unit/brand and publish entitlement semantics | Publication access remains plan/account specific and does not grant approval |
| [Contentful CMA overview](https://www.contentful.com/developers/docs/references/content-management-api/overview/) | Full-body update, optimistic version locking and media-type versioning | CMS success does not prove public destination representation or editorial approval |
| [Cloudinary Admin API](https://cloudinary.com/documentation/admin_api) | Asset/version, admin scope, partial deletion/cursor, invalidation and backup semantics | Powerful account-specific API; deletion/invalidation can be partial/delayed/irreversible |
| [GA4 Data API quotas](https://developers.google.com/analytics/devguides/reporting/data/v1/quotas) | Property/project token, concurrency, error and threshold behavior | Returned report is provider-defined observation, not complete population or causal lift |
| [GA4 BigQuery export differences](https://support.google.com/analytics/answer/9358801) | Intraday best-effort versus stable daily export and UI/raw semantic differences | Limits, delayed attribution and property configuration apply |
| [FTC Advertising Substantiation Policy](https://www.ftc.gov/legal-library/browse/ftc-policy-statement-regarding-advertising-substantiation) | Pre-dissemination reasonable-basis boundary for objective claims | US regulator policy; product and jurisdiction may require different/stronger evidence |
| [FTC dark-pattern report](https://www.ftc.gov/reports/bringing-dark-patterns-light) | Review taxonomy for deception/manipulative interface patterns | Staff report is not a universal automated classifier or legal conclusion |
| [Google restricted personalized targeting](https://support.google.com/adspolicy/answer/143465) | Current sensitive-interest and opportunity-category provider restrictions | Google product policy, not complete anti-discrimination or legal compliance framework |

## Refresh triggers

Refresh immediately when any of these occurs:

- provider API release, deprecation, sunset, client-library incompatibility, quota, scope, webhook, batch, partial-failure, idempotency, review, budget, billing, reporting, audience, experiment, or conversion change;
- advertising-platform or mail-sender policy/enforcement change;
- privacy, direct-marketing, telemarketing, targeting, consumer-protection, endorsement, regulated-claim, or AI-governance change in an operating jurisdiction;
- new channel, provider, brand, country, audience source, sensitive-data class, sender identity, ad account, budget authority, experiment type, or lead-handoff field;
- model/prompt/context/tool/memory/evaluator/runtime change or new long-term/episodic memory proposal;
- complaint, bounce, suppression, deletion, wrong-audience/account, cross-tenant, spend, duplicate/unknown effect, claim, public correction, attribution, experiment, or lead-quality incident;
- evidence that the deterministic baseline now matches agent quality/cost, or that a new model/runtime materially improves a bounded slice.

Otherwise, review volatile provider, policy, security, and model sources within 90 days and stable workload/experiment architecture within 180 days. Re-open external links and rerun provider capability tests before every production adapter release.

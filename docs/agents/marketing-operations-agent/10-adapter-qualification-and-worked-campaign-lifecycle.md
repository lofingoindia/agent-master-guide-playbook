# Adapter Qualification and Worked Campaign Lifecycle

[Blueprint home](README.md) · Previous: [Deployment, scale, cost, and evolution](09-deployment-scale-cost-and-evolution.md)

## Purpose

This guide converts the architecture into a provider and operating proof. It does not endorse a marketing stack. It shows how to qualify the exact operations a deployment needs and how one campaign remains safe through audience compilation, claim review, launch, spend change, experiment waits, suppression, lead handoff, ambiguous provider effects, and rollback.

Provider pages and primary policies below were checked on **2026-08-31**. Public documentation cannot establish a tenant's plan, contract, account setting, region, granted scope, quota, review eligibility, data-use term, or live behavior. Every write capability expires unless a production-account conformance test renews it.

## Decide whether to build an agent

| Work | Smallest suitable system | Do not add a model unless |
|---|---|---|
| Stable newsletter, lifecycle journey, audience rule or scheduled promotion | Deterministic workflow, approved template and human checklist | Repeated semantic exceptions remain after improving rules/templates |
| Consent, suppression, eligibility, spend cap, pacing threshold, experiment integrity or lead dedupe | Deterministic policy/state machine | Never for the control decision; a model may explain a denial |
| One campaign brief and one channel | Manual marketer workflow with versioned records | Measured review time/error/variant-quality evidence justifies a bounded assistant |
| Incomplete brief across several approved channels with heterogeneous constraints | Bounded model planner inside a durable workflow | Offline and shadow evaluation beats template/retrieval baselines on named slices |
| Claims, dark-pattern, privacy, discrimination, public publication or major spend judgment | Accountable human review with deterministic evidence package | Never transfer the accountable decision to a model |
| Dashboard, attribution query or causal analysis | BI/analytics system and analyst | The need is campaign orchestration rather than statistical interpretation |

The agent is retired or held in proposal-only mode when deterministic/template performance reaches the same outcome at lower risk/cost, reviewers reject most suggestions, provider semantics cannot be qualified, or human review becomes the true bottleneck.

## Identity and version semantics

Names, URLs, email addresses, provider resource names and content hashes are useful attributes—not universal IDs.

| Object | Stable local ID | Version/event identity | Never conflate |
|---|---|---|---|
| Tenant | `tenant_id` | tenancy/policy revision | Organization display name or provider manager account |
| Brand | `brand_id` | brand-policy release | Sender domain, business unit, ad account or visual theme |
| Objective | `objective_id` | `objective_revision_id` | Desired outcome, campaign plan, metric or authorization |
| Campaign | `campaign_id` | campaign revision and run ID | Provider campaign resource or one channel flight |
| Audience | `audience_id` | immutable `audience_snapshot_id` | Segment definition, provider audience or member list |
| Consent | `consent_subject_key` + purpose/channel/controller | append-only `consent_event_id` and evidence version | CRM `marketable` flag, provider subscriber state or blanket permission |
| Suppression | subject/channel/sender/purpose key | monotonic `suppression_event_id` and cursor | Deletion, consent absence, bounce or one provider block list |
| Segment | `segment_definition_id` | definition revision and evaluation run | Audience snapshot; membership changes with source data/time |
| Asset | `asset_id` | immutable content/render revision and digest | DAM public ID, URL, crop or channel encoding |
| Claim | `claim_id` | wording/evidence/approval revision | Product fact, implied claim, provider policy acceptance or legal conclusion |
| Channel | `channel_id` | channel policy/capability release | Provider, connector, sender, placement or campaign |
| Destination | `destination_id` | destination revision and observed content version | URL string, redirect target or landing-page approval |
| Budget | `budget_authorization_id` | authorization/reservation/settlement revision | Provider budget resource or invoice hard cap |
| Experiment | `experiment_id` | protocol, assignment and analysis-vintage versions | Provider campaign draft, optimization feature or causal truth |
| Conversion | `conversion_event_id` | correction/restatement event and report vintage | Platform-attributed conversion, unique person or incremental outcome |
| Lead | `lead_candidate_id` | qualification revision | CRM contact, account, opportunity or one email address |
| Handoff | `handoff_id` | attempt/receipt/disposition version | Lead identity or sales acceptance |
| Approval | `approval_id` | signed decision version bound to package digest | Workflow status, comment, provider review or authentication |
| Effect | `effect_id` + semantic key | attempt and reconciliation events | HTTP request, job, webhook, provider resource or intended outcome |

Store `valid_from/to`, `occurred_at`, `recorded_at`, `observed_at`, provider/report timezone, schedule/not-before/deadline, consent/suppression cursor, attribution event/report time, outcome maturity time, approval expiry and provider resource version separately. A late conversion restatement appends a new vintage; it does not rewrite what the campaign knew at launch.

## Operation-level capability manifest

Qualify operations, not “the HubSpot connector” or “the ads account.” Reads and writes against the same product often need different identities, risks and evidence.

```yaml
marketing_adapter_capability:
  schema_version: marketing-adapter/2
  capability_id: google-ads-emea.campaign-budget-update@7
  provider:
    product: google_ads
    api_version: v25
    client_release: pinned-by-deployment
    account: customers/1234567890
    manager_scope: none
  operation:
    name: campaign_budget.update
    danger_tier: D3
    input_schema: schema://google-ads-v25/budget-update@3
    result_semantics: mutate_ack_then_resource_readback
    supports_validate_only: true
    partial_failure: disabled_for_this_operation
    provider_idempotency_key: not_documented
    concurrency: one_writer_per_budget_resource
  authority:
    tenants: [tenant_72]
    brands: [brand_a]
    purposes: [approved_campaign_delivery]
    max_delta_percent: 10
    max_absolute_authorization_ref: budget-policy/11
  data:
    allowed_classes: [campaign_configuration]
    prohibited_classes: [raw_audience, consent_evidence, connector_secret]
    retention_profile: provider-effect-receipt/5
    provider_rights_terms_ref: contract://google-ads/2026-08-01
  execution:
    credential: broker://google-ads-emea-budget-writer
    credential_visible_to_model: false
    timeout_ms: 10000
    retry: reconcile_before_retry
    reconciliation_reads: [campaign_budget, change_event]
  evidence:
    provider_docs_checked_at: 2026-08-31T00:00:00Z
    conformance_report: report://adapters/google-ads-budget/2026-08-31
    owners: [media-platform, budget-control, security]
    expires_at: 2026-11-29T00:00:00Z
```

### Mandatory conformance cases

| Family | Case | Passing behavior |
|---|---|---|
| Identity/account | Duplicate display names, manager/child account, wrong brand/business unit, stale provider ID | Canonical immutable account/resource is shown in approval and enforced before credential use |
| Schema/version | Unknown field/enum, removed field, dated path change, provider default changes | Fail closed, expire manifest, preserve raw response as untrusted artifact |
| Population | Zero/one/last/partial page, async audience job, per-member rejection, membership change during wait | Count/digest/coverage/cursor and per-item outcomes reconcile; no “job done = audience complete” |
| Consent/suppression | Opt-out during compile/upload/schedule/send; missing category; identity merge/split | New activation fails closed; removal lane is reserved; no consent inheritance by similarity |
| Content/destination | Claim or redirect changes after approval, channel rendering mutates disclosure, provider disapproves asset | Exact digest/version invalidates approval; provider review stays supplementary |
| Effect | Timeout before/after commit, duplicate callback, batch partial success, UI edit, cancellation after dispatch | Stable semantic key, `unknown`, readback/reconciliation, no blind duplicate |
| Money | Shared budget, provider pacing lag, report correction, invoice discrepancy, concurrent change | Independent reservation/cap remains authoritative; conflicting mutation is fenced |
| Deletion/rights | Deletion, provider audience removal, asset CDN/cache, backup restore, contract/term withdrawal | Tombstone/restriction propagates with exception/hold owner and evidence |
| Security | Hostile CRM note, HTML, image metadata, provider error, webhook, dependency update | Content remains data; no target/tool/credential/policy change; sandbox and egress hold |
| Load/recovery | Quota exhaustion, provider outage, webhook replay, region loss plus launch backlog | Suppression/pause/reconciliation retain capacity; no safety downgrade or split-brain write |

## Qualified adapter map

These examples preserve current product-specific limitations rather than pretending to form one universal marketing API.

| Surface and primary source | Qualified operations | Semantics to preserve | Current limitation and default posture |
|---|---|---|---|
| Google Ads API [v25.1 release notes](https://developers.google.com/google-ads/api/docs/release-notes), [partial failure](https://developers.google.com/google-ads/api/docs/best-practices/partial-failures), [batch jobs](https://developers.google.com/google-ads/api/docs/batch-processing/overview) | Read resource/change/report; validate draft; create disabled resource; mutate approved status/budget/audience/experiment; upload conversions | Canonical resource name, manager/customer, `validate_only`, per-operation error/result, batch state, resource readback, report/event time | v25.1 was published 2026-08-19; v25 sunset is listed for August 2027. Partial failure is method-specific and batch successes are not rolled back. One writer/fence per shared resource |
| Meta customer-list audiences [terms](https://www.facebook.com/legal/terms/customaudience/update), official [Python Business SDK](https://github.com/facebook/facebook-python-business-sdk), 2026 [Marketing API Access Tier change](https://developers.meta.com/blog/updates-to-ads-management-standard-access-feature/) | Read approved ad/account/resource state; create/update/pause only after account-specific tests; add/remove customer-list audience members through protected service | Advertiser/ad-account/page IDs, API/SDK version, access tier/scopes, audience source/opt-out obligation, batch result and provider state | Terms require rights/lawful basis and removal after opt-out; local hashing does not make data non-personal. Access tier/rate eligibility changed in 2026. No production write until current Graph/Marketing version and account behavior are archived and tested |
| Mailchimp Marketing API [v3.0.91 reference](https://mailchimp.com/developer/marketing/api/) | Create draft, set content, read checklist, schedule/unschedule/send/cancel, reports, list/member and webhook reads | Campaign/list/segment IDs, saved/scheduled/sending/sent/cancel state, checklist snapshot, recipient counts/errors, campaign-type distinctions | Send can be immediate; unschedule applies only before sending; cancellation is plan/type dependent. Timeouts can coexist with backend work. Newer audience surfaces marked beta do not own suppression |
| HubSpot Marketing Email [2026-03](https://developers.hubspot.com/docs/api-reference/latest/marketing/marketing-emails/guide) | Create/read/update approved marketing email draft; publish/unpublish only if separately entitled and D3-approved; read post-send stats | `businessUnitId`, email ID/timestamps, dated API path, marketing-vs-sales email, entitlement and workflow references | `folderId` is no longer supported; `folderIdV2` is current. Publish/unpublish require specified product entitlement. No sales-email authority |
| Twilio Messaging [Advanced Opt-Out](https://www.twilio.com/docs/messaging/tutorials/advanced-opt-out), [Consent Management API](https://www.twilio.com/docs/messaging/features/consent-api), [Messaging Policy](https://www.twilio.com/en-us/legal/messaging-policy) | Send through one approved Messaging Service/sender; ingest signed inbound/status events; append STOP/START/HELP evidence; synchronize opt-out through protected ledger | Account/service/sender/channel, consent purpose/source/time, provider block scope, `OptOutType`, message SID and delivery state | Advanced Opt-Out is disabled by default and blocked-number state is not generally changed/reported by its REST interface; keyword/sender/country behavior differs. A Twilio block is not the enterprise suppression ledger; re-opt-in needs policy evidence |
| Twilio Segment [consent on profile](https://www.twilio.com/docs/segment/privacy/consent-management/consent-in-unify) and [Engage audiences](https://www.twilio.com/docs/segment/privacy/consent-management/consent-in-engage) | Read governed profile/audience aggregates; materialize an approved snapshot; propagate consent events to supported destinations | Space/profile/identity merge, category mapping, missing-category behavior, audience definition/run and destination sync | Profile consent storage and Engage enforcement are documented as public beta; missing categories can resolve false and product coverage has exclusions. Enterprise ledger remains authoritative; beta path cannot be sole suppression gate |
| Salesforce REST [upsert by external ID](https://developer.salesforce.com/docs/platform/mobile-sdk/guide/ref-rest-apis-upsert.html) and [Composite](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_composite_composite_post.htm) | Read allowed contact/account fields; upsert a minimal marketing lead-intake object using a unique external handoff ID; query receipt | Org/object/record/external-ID field, create-vs-update result, duplicate rule, field-level permission, composite dependency/rollback choice | Upsert may insert or update; composite rollback behavior is configurable and unsupported resources exist. Marketing must not change opportunity/stage/owner or send seller messages |
| HubSpot CRM [2026-03 object API](https://hubspot.mintlify.io/api-reference/crm-objects-v2) | Read purpose-approved contact fields; upsert minimal handoff object/contact via custom unique `idProperty`; retrieve disposition | Portal/object/property schema, unique ID, archive/delete state, per-record batch result | Email-based contact upsert does not support partial upserts. A custom unique handoff ID is safer; CRM lifecycle stages and deals remain sales-owned |
| Contentful CMA [overview](https://www.contentful.com/developers/docs/references/content-management-api/overview/) | Read exact entry/version; create/update a draft; publish/unpublish only through separately approved operation | Space/environment/entry/locale, current `sys.version`, full body, published version, destination observation | Uses optimistic locking and full-body updates; omitted properties can be lost. Pin management media type; never let a stale model overwrite unseen edits |
| Cloudinary [Admin API](https://cloudinary.com/documentation/admin_api) | Read asset/metadata/version/derived state; create approved transform; deletion/invalidation only as separate high-risk operation | Product environment, immutable asset ID versus public ID, version, derived transformation, CDN invalidation and backup state | Admin API is powerful and rate-limited. Delete-by-prefix/all can be partial with cursors; CDN invalidation takes time and may need enablement; backup deletion is irreversible. No Admin API credential in model workers |
| GA4 Data API [v1 quotas](https://developers.google.com/analytics/devguides/reporting/data/v1/quotas) | Read named metrics/dimensions after compatibility check; store query, property, timezone, quota and response artifact | Property, metric definition, attribution model, date range, thresholding, sampling/limits, report generation time | Token/concurrency/error quotas apply; some dimensions can be thresholded. Report output is an observation, not causal lift or finance truth |
| GA4 BigQuery [export differences](https://support.google.com/analytics/answer/9358801) | Read stable daily event export for governed measurement; intraday only as provisional monitoring | Event ID/time, ingestion/table date, daily/intraday vintage, late/corrected event, UI/export semantic difference | Streaming export is best effort, can omit late/failed uploads and lacks some attribution data; daily table is the stable day. UI and raw export can differ |

Use MCP only as transport for an already admitted, operation-level capability. It cannot supply consent, approval, evidence, idempotency, least privilege, or provider finality. Generic dynamic CRM/ad/CMS/MCP writes stay disabled.

## Worked campaign: webinar acquisition with paid search and email

The fictional campaign promotes a technical webinar. It avoids regulated products so the workflow—not a legal conclusion—is the focus.

### Phase 1 — admit objective and compile audience

The marketing owner approves `objective_31@4`: registrations from existing business subscribers and a contextual paid-search campaign, with a fixed flight cap. The case records brand, sender, jurisdictions, channels, destination, claims, excluded sensitive targeting, experiment hypothesis, metric definition, deadline and named reviewers.

```yaml
audience_compile_request:
  campaign_ref: cmp_31@7
  audience_id: aud_webinar_existing_1
  segment_definition_ref: seg_active_b2b_subscribers@9
  purpose: webinar_marketing
  channel: email
  sender: brand-a-marketing
  source_event_high_watermark: warehouse_event_8821
  consent_policy: consent-email-webinar@12
  suppression_cursor_minimum: suppression:00190221
  allowed_jurisdictions: [configured_policy_set]
  prohibited_traits: [health, financial_distress, politics, protected_class, precise_location]
```

The protected audience service evaluates every member deterministically. The model sees counts by decision/reason and synthetic examples, never raw emails. The snapshot records input version, eligibility decision digest, suppression cursor, member count/digest, expiration, export right and activation destination. A small or sensitive cohort triggers privacy/fairness review; performance opportunity never relaxes suppression.

### Phase 2 — draft content and prove claims

The model proposes three subject/body/search-ad variants using an approved product-fact corpus. Each express or implied objective claim has a ledger entry, evidence, qualifiers, jurisdictions, allowed channels and expiry. The landing page, redirects, form fields, disclosure, sender identity and asset render are versioned together.

FTC substantiation guidance requires a reasonable basis before dissemination for objective claims; provider acceptance does not supply that basis. The [FTC dark-pattern report](https://www.ftc.gov/reports/bringing-dark-patterns-light) informs review for fake scarcity, disguised ads, hidden cost, confirm-shaming, obstruction and misleading defaults. Legal applicability remains deployment-specific.

Brand, claims, accessibility, privacy and channel reviewers receive an exact package digest. Approval records decision, role, scope, conditions, expiry and invalidation triggers. A changed URL, redirect, offer, testimonial, countdown, disclosure, sender, audience, schedule or render creates a new revision.

### Phase 3 — prepare and launch

The workflow creates a disabled Google Ads resource and a saved Mailchimp campaign, reads both back and renders provider representations. It reserves the application flight cap and freezes the experiment protocol. Immediately before commitment it refreshes:

1. tenant/brand/account/sender/destination identity;
2. latest campaign, asset, claim, audience and provider-resource versions;
3. consent/suppression cursor and removal backlog;
4. approval digest/expiry and separation-of-duty requirements;
5. spend reservation, shared-resource fence and worst-case exposure;
6. experiment assignment/configuration and guardrails;
7. provider capability, policy/review state, quota and commit window.

```yaml
launch_effect:
  effect_id: eff_mail_webinar_1
  semantic_key: tenant_72/cmp_31/mailchimp/campaign_abc/schedule/2026-09-14T09:00Z/rev_7
  operation: campaign.schedule
  campaign_ref: cmp_31@7
  audience_snapshot_ref: audsnap_991@3
  content_ref: asset_email_44@6
  destination_ref: dest_webinar_lp@5
  provider_resource: campaign_abc
  schedule: 2026-09-14T09:00:00Z
  approval_ref: approval_launch_18
  payload_digest: sha256:...
  suppression_cursor: suppression:00190245
  expires_at: 2026-09-14T08:59:30Z
  state: authorized
```

The effect worker has only the schedule operation for this account. Provider acknowledgement moves the effect to `acknowledged`; verified provider state moves it to `verified`. The public landing page is separately observed after its human-owned publication effect.

### Phase 4 — suppression arrives during the wait

At 08:58 a subscriber opts out. The suppression event advances the cursor and invalidates the activation snapshot/approval for that member. The priority removal lane:

- stops any not-yet-committed audience or send effect;
- rebuilds the recipient delta or calls a qualified removal operation;
- records provider-specific block/removal evidence;
- prevents the member entering a new snapshot, cache, experiment or restored backup;
- escalates when the provider cannot prove removal before send.

If a provider has already started sending, cancellation cannot recall delivered messages. State records residual recipients and incident/correction action rather than claiming compensation erased the effect.

### Phase 5 — bounded spend change

Paid search underspends after two days. The pacing controller—not the model—computes variance from a mature provider read. The model may explain known causes and propose a ten-percent budget change. The budget owner receives current spend, worst-case exposure, flight-cap remainder, provider resource/version, experiment impact and rollback/stop evidence.

A new authorization/reservation and effect ID are required. The adapter runs `validate_only`, fences the shared budget, mutates once, and reads back. A provider average daily budget is still a delivery control; application flight cap and spend reconciliation remain stricter. A UI edit during the wait causes a version conflict and fresh proposal.

### Phase 6 — experiment and leakage-safe readout

The protocol was registered before launch: unit, eligibility, randomization, holdout, treatment, primary metric, guardrails, power/duration, exclusions, missing-data policy, attribution windows and analysis maturity date. Operational monitoring checks assignment and safety, not winners.

At maturity, analytics verifies assignment, sample ratio, exposure, contamination, instrumentation, attrition, novelty/seasonality and guardrails. Training/eval examples and prompt context exclude post-treatment conversions when predicting eligibility or creative selection. Report platform attribution, observational association and randomized incremental estimate separately; do not optimize on a provisional GA4 intraday table.

### Phase 7 — lead handoff

A registration reaches the approved marketing qualification rule. The protected service creates a minimal handoff with its own stable identity:

```yaml
lead_handoff:
  handoff_id: handoff_webinar_00091
  lead_candidate_id: lead_811
  qualification_revision: mql-webinar@4
  campaign_ref: cmp_31@7
  consent_and_notice_ref: handoff-purpose@3
  destination: sales-intake-emea
  fields_ref: artifact://handoffs/handoff_webinar_00091/minimized
  prohibited_mutations: [account, opportunity, stage, owner, seller_message]
  semantic_key: tenant_72/sales-intake-emea/handoff_webinar_00091/create
```

Sales intake returns accepted, duplicate or rejected with its record reference/reason. A timeout is `unknown`; the reconciler queries by the same external handoff ID before any retry. Marketing does not create an opportunity or interpret CRM lifecycle stage as realized revenue.

### Phase 8 — unknown effect, reconciliation and rollback

Suppose the email schedule call times out after dispatch:

1. persist attempt evidence and transition to `unknown`;
2. revoke automatic retry and schedule the effect reconciler before send time;
3. query the exact provider campaign, schedule and content/audience state;
4. if intended state exists, mark verified; if authoritative absence is proved and approval remains valid, retry the same intent once;
5. if neither can be proved, escalate for manual decision—never create a second campaign/send;
6. if wrong content/audience was scheduled but not sending, create a separately authorized unschedule/repair effect;
7. if delivery began, pause/cancel where supported, suppress residuals, preserve affected-recipient evidence, notify incident owners and follow human correction/remediation policy.

Compensation is a new effect. It does not erase delivery, spend, privacy exposure, experiment contamination or public observation. Rollback also stops the candidate behavior bundle, fences in-flight workers, reconciles all nonterminal effects, identifies affected campaigns/assets/audiences and routes compatible work to the prior safe bundle.

## Security, privacy, fairness and deliverability gate

| Risk | Required control and evidence |
|---|---|
| Purpose/consent | Controller/sender/purpose/channel/jurisdiction/source evidence; current decision at commit; objection/suppression precedence; deletion/restore propagation |
| Discrimination/sensitive targeting | Prohibited/special category classifier plus human policy owner; compare eligibility/delivery/outcome slices; no proxy broadening or inferred sensitive trait |
| Claims/dark patterns | Express and implied claim ledger, pre-dissemination evidence, render/landing-page review, disclosure and offer-clock truth; human legal/brand owner |
| Prompt/content injection | Treat brief, CRM note, form text, asset metadata/OCR, webpage, redirect, provider recommendation/error and webhook as untrusted; isolate render/parser and block content-derived tools/targets |
| Tenant/secret isolation | Authorize before query/cache/dedupe, separate audience stores/keys, broker exact account/operation credential after approval, prevent manager-account expansion |
| Least privilege and SoD | D4 administration unavailable; creator cannot self-grant brand/privacy/budget/channel approval; high-risk targeting/spend dual control where policy requires |
| Audit versus evidence | Unsampled authority/effect audit, versioned campaign evidence, sampled trace/log and aggregate SLO remain separate; no raw audience/prompt in telemetry |
| Deliverability/abuse | Authenticated sender/domain, one-click unsubscribe where applicable, complaint/bounce/spam/deferral guardrails, volume warm-up and independent pause/revoke; current [Gmail sender guidance](https://support.google.com/mail/answer/81126) is operational input, not universal law |
| Supply chain | Pinned SDK/parser/renderer/adapter, provenance/SBOM/signature where available, minimal build identity, dependency/advisory monitoring, staged compatibility tests and credential revocation |

Google's current personalized-ad policy restricts advertiser-curated audiences for listed sensitive interests and limits demographic/postcode targeting for specified opportunity categories in the US/Canada. Treat [the policy](https://support.google.com/adspolicy/answer/143465) as a provider constraint, not a complete anti-discrimination program or universal legal rule.

## Stage 0–6 proof exercises

| Stage | Exercise | Measurable exit evidence |
|---:|---|---|
| 0 | Run the webinar campaign with forms, deterministic audience/consent/suppression, approved templates, human provider operation and manual lead receipt | Time/error/review/complaint/spend/handoff baseline and a named semantic bottleneck; otherwise stop |
| 1 | Add one read-only bounded planner over synthetic fixtures; inject missing brief, prohibited trait and hostile landing-page text | Better predeclared plan/claim metric than template baseline; zero live capabilities; bounded cost/turns/stop |
| 2 | Connect authorized aggregates, brand/claim policy and provider reads/drafts in sandbox; run human calibration | Tenant/purpose isolation, exact artifacts/provenance, useful reviewer acceptance by slice, no D3 effect |
| 3 | Qualify one email and one lead-handoff effect; crash around every commit; opt out during each wait; compact/resume repeatedly | Capability reports, exact seven-memory policy, continuity receipt, no duplicate send/lead, deletion/suppression convergence |
| 4 | Shadow live cases and canary one account; red-team injection, account substitution, claim/dark-pattern, cross-tenant, spend and supply chain | Hard gates, SLOs/traces/evidence/audit separation, kill/revoke/unknown-effect/incident/rollback drills and accountable sign-off |
| 5 | Load peak launches plus provider outage, webhook replay, removal surge, region loss and human-review bottleneck | Fair admission, reserved safety capacity, recovery throughput greater than arrival plus backlog drain, component RPO/RTO and no split-brain effects |
| 6 | Convert a reviewed complaint, missed suppression, provider drift and experiment contamination into minimal fixtures; change one behavior component | Purpose/leakage review, prior-bundle failure, candidate pass, offline replay, shadow, canary, drift alert, rollback and affected-output invalidation |

## Production acceptance checklist

- [ ] Deterministic/manual baseline and retirement threshold remain visible.
- [ ] All canonical objects have independent local identity and version/event semantics.
- [ ] Each provider operation has a current manifest, production-account conformance report, danger tier, account scope, rights/retention, owner, expiry and kill switch.
- [ ] Audience membership never enters model context; consent/suppression is refreshed at every activation boundary.
- [ ] Claims, destinations, assets and renderings are evidence-bound and human approved; provider review is supplementary.
- [ ] Money uses authorization/reservation/reconciliation outside provider budget controls.
- [ ] Experiments use pre-registration, holdouts, integrity checks, maturity vintages and leakage-safe evaluation.
- [ ] Every external effect supports cancellation before dispatch, stable semantic identity, fencing, timeout, `unknown`, reconciliation and separate compensation.
- [ ] Lead handoff cannot mutate sales opportunities or send seller communication.
- [ ] Context resume verifies the loss-aware [continuity receipt](06-state-context-memory-and-orchestration.md#context-compaction).
- [ ] Evaluation covers outcomes, trajectories, invariants, human factors, counterfactuals/holdouts, fairness, failure injection, repeated reliability, scale/cost and DR.
- [ ] Behavior bundle, shadow/canary, drift, rollback, affected-output query and controlled failure mining are exercised.

## Genuine limitations and refresh triggers

No blueprint can promise legal compliance, nondiscrimination, deliverability, factual claims, causal lift, spend containment or provider finality merely by naming controls. Counsel, privacy, brand, media, deliverability, analytics, finance, sales and security owners must qualify the actual deployment.

Requalify on API/client/date path, scope/entitlement, account hierarchy, quota, batch/partial failure, idempotency, webhook, review, sender, audience/consent, targeting, attribution, report, budget/billing, retention, rights, deletion or provider-policy change. Recheck immediately after a wrong audience/account/claim/destination, suppression miss, complaint spike, duplicate/unknown effect, spend discrepancy, cross-tenant exposure, experiment invalidation, lead duplicate, supply-chain incident or rollback. Provider sandboxes and public docs do not replace canary reads/writes in a controlled production account.

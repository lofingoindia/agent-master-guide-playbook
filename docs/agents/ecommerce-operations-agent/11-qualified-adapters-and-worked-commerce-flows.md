# Qualified Adapters and Worked Commerce Flows

[← Previous: Deployment, peak scale, cost, and governed evolution](10-deployment-peak-scale-cost-and-governed-evolution.md) · [Blueprint home](README.md)

This guide turns the control model into provider-specific implementation decisions and end-to-end exercises. It is not a universal connector specification. A production adapter is admitted one operation at a time against a named account, API release, schema, credential, and postcondition.

## What qualification must prove

Installing an SDK or completing OAuth proves connectivity, not operational safety. Each admitted operation needs evidence for:

1. exact resource and account identity;
2. API, schema, payload, and notification versions;
3. read, draft, publish, delete, list-replacement, and bulk semantics;
4. permissions actually granted to the runtime credential;
5. timeout, retry, idempotency, concurrency, and cancellation behavior;
6. synchronous, queued, processed, published, indexed, and observable states;
7. quota unit, partition, headers, batch limits, and overload behavior;
8. receipt/job/task identifiers and retention;
9. authoritative read-back and propagation budget;
10. partial-result, unknown-outcome, correction, and rollback path; and
11. data classification, tenancy, region, audit, and support ownership.

Qualification expires when an API release, schema, product type, account capability, credential scope, provider policy, SDK, or postcondition changes. Provider documentation is the starting contract; test-account evidence is the deployment contract.

## Operation-level capability manifest

```yaml
schema: commerce.adapter-operation/v1
adapter: shopify-admin-graphql
adapter_release: 7.3.1
provider_api_release: 2026-07
operation: productSet
environment: production
binding:
  tenant_id: t_acme
  shop_id: shop_778
authority_class: D3
credential:
  secret_ref: secret://commerce/shop_778/product-write
  declared_scopes: [write_products]
inputs:
  schema: shopify.product-set-projection/v4
  identity_fields: [shop_id, product_gid]
  destructive_semantics:
    list_omission: deletes_existing_members_not_supplied
concurrency:
  provider_precondition: none_for_this_operation
  application_lease: product_gid
idempotency:
  provider_key: unsupported_or_not_documented
  semantic_effect_key: required
completion:
  transport_success: mutation_or_product_set_operation_returned
  processed_success: operation_complete_without_user_errors
  postcondition: exact_current_product_read_and_channel_observation
limits:
  discovered_at: 2026-08-31T00:00:00Z
  source: provider_cost_and_operation_response
unknown_policy: reconcile_before_retry
kill_switch: tenant_shop_operation
owner: commerce-shopify-connector
```

Do not copy the example values into production. Generate the manifest from connector configuration and signed release metadata, then compare it with runtime discovery and response headers.

## Adapter qualification matrix

| Family and representative provider | Admit these operations | Native semantics that must remain visible | Completion proof | Exclude or separately authorize |
|---|---|---|---|---|
| PIM — Akeneo | Read product, product model, variant, family/schema, locale/channel fields, completeness/readiness observation, events; create draft/task only if the tenant supports a reversible lifecycle | Product/model/family identity; channel/locale scope; configured completeness is not truth; events/features differ by edition and rollout | Re-read exact product/model plus expected revision/field; downstream channel remains separate | Canonical merge/split, GTIN allocation, claim approval, or live source overwrite from model text |
| Commerce catalog — commercetools | Read Products, current/staged Product Projections, Stores, prices and selections; prepare versioned update; publish/unpublish only as D3 | Resource `version` is optimistic concurrency; staged/current/published are separate; search and discount propagation can be eventual; update actions for one resource are cohesive | Update response at expected new resource version, then current projection and search/storefront observation after budget | Treating unpublish as removing cart eligibility; inventory mutation; retrying `ConcurrentModification` by overwriting newer state |
| Storefront — Shopify Admin GraphQL | Read products, variants, publications, catalogs, price lists, inventory observation, returns aggregates; exact product/variant/publication mutation after qualification | Dated API version; GraphQL calculated cost; global IDs; `productSet` list fields replace membership and can delete omitted entries; async mode can return an operation; webhooks can duplicate, reorder, or be missed | Query exact resource/publication after mutation, inspect user errors/operation, and observe storefront or checkout where the effect requires it | Inventory source-of-truth mutation, order/refund/customer communication, broad admin tool, or assuming webhook delivery completes a run |
| Shopping feed — Google Merchant API | v1 data-source discovery; ProductInput insert/patch/delete; processed Product/status/issue reads; promotions; notifications; quota discovery | ProductInput versus read-only processed Product; identity includes language, feed label, offer ID, data source/account; patch masks and source type matter; processing is delayed; promotion insert overwrites the resource and review is asynchronous | Read processed Product/status or Promotion status for exact account/name, then destination/observable state when available | Treating insertion as approval; mixing migrated v1 and obsolete beta contracts; writing file-owned sources; assuming notification completeness |
| Marketplace — Amazon SP-API | Product Type Definitions; Catalog/Listing reads; Listings Items for single-item mutation; `JSON_LISTINGS_FEED` for bulk; listing issue/status notifications; Orders read only when a minimized aggregate is justified | Seller + marketplace + SKU + ASIN/product type are distinct; current product-type schema and eligibility matter; one-item and feed lifecycles differ; usage plans can be dynamic; notification payload versions differ | Per-item listing read including issues/offers/status; feed processing report for every message; discoverable/buyable state as applicable | Legacy XML/flat-file listing feeds; switching individual/bulk transport after an unknown send; fulfillment/refund/customer actions; treating all listing problems as covered by issue notifications |
| OMS/inventory/returns | Read versioned availability projection, order/return aggregate, reason taxonomy, and handoff status | Stock bucket, location, allocation, fulfillment, event sequence, order/return state and reason taxonomy version must remain separate | Freshness-valid source read and accepted handoff receipt | Quantity, reservation, allocation, refund, return authorization, cancellation, shipment, or customer-message writes |
| Pricing/tax/promotion service | Evaluate exact offer/context; receive signed price/promotion/tax decision; simulate stacking; create internal approval request | Exact decimal/minor units, currency, tax basis, market/catalog/customer context, policy release, effective interval, coupon/code set, priority and exclusions | Decision signature/version plus commit-time re-evaluation; channel projection is a later effect | Model-generated floor, reference price, eligibility, tax result, targeted discount, or restricted-category exception |
| CMS/DAM — Contentful example | Read entry/asset/locale/environment; create/update draft; publish locale or entry only after qualification | CMA update uses optimistic locking with `X-Contentful-Version`; full-body update can lose omitted fields; draft and published state differ; locale publishing is plan-dependent | Re-read current `sys.version` and `publishedVersion`, then query delivery endpoint/CDN after propagation budget | Unversioned overwrite, source-claim approval, asset-rights decision, or assuming CMA response equals delivery visibility |
| Search — Algolia example | Read records/settings; partial object update for derived projection; task wait; controlled atomic reindex | Index operations are queued and return `taskID`; search can lag submission; `replaceAllObjects` builds/replaces through a temporary index and changes the whole projection | Wait for task, query exact object/filter/ranking fixture, compare target counts and search projection digest | Search as product truth; full replacement for routine updates; model-selected hidden boosts; deleting source records because index changed |
| Analytics/experimentation | Read governed aggregates, metric definitions, exposure assignments, return/cancellation cohorts, and experiment results | Metric version, event time, attribution window, denominator, late-arrival watermark, consent/purpose and experiment assignment are part of the record | Warehouse reconciliation plus metric-quality checks; outcome is observed after a verified effect | Raw customer event streams, direct profile activation, causal claims from channel reports, or online rule updates |
| Internal communications | Create an incident/review notification with typed audience, template, artifact links, severity and dedupe key | Queued, provider-accepted, delivered and acknowledged are distinct; destination is resolved outside model output | Provider receipt plus delivery/acknowledgement where supported; otherwise an owned queue item | Customer outreach, marketing sends, fabricated recipients, secrets, raw personal data, or treating “message sent” as handoff acceptance |

The named products are representative qualification targets, not endorsements and not mandatory architecture. Use the platform already owned by the organization. Qualify every operation because two endpoints from the same provider can have different scopes, retry safety, lifecycle, and quota behavior.

## Provider-version findings to pin

The following observations were checked against official documentation on 2026-08-31:

| Provider | Current finding | Local limitation and refresh trigger |
|---|---|---|
| Google Merchant | Merchant API v1 separates writeable ProductInput from processed read-only Product; v1 registration and migration requirements apply; Promotions insert is whole-resource replacement and review is asynchronous | Account programs, destinations, data-source ownership, quotas, processing and visibility vary; refresh every API/deprecation/policy change |
| Amazon SP-API | Listings Items `2021-08-01` and JSON listings feeds are current listing paths; legacy XML/flat-file listing feeds were removed; Orders has a newer `v2026-01-01` migration path; notification payload versions and issue coverage must be explicit | Roles, marketplace eligibility, product types, dynamic usage plans, sandbox coverage and processing vary; discover capabilities in the target seller account |
| Shopify | Admin GraphQL is dated-versioned; `productSet` list inputs delete existing members omitted from input; webhook ordering/completeness is not guaranteed; Events remains developer-preview for supported topics | Mutation availability, user permissions, app scopes, calculated cost, protected data and list semantics must be contract-tested against the pinned shop/API release |
| commercetools | Resource version is required for optimistic concurrency and `ConcurrentModification` is explicit; current/staged projections and eventual search/discount propagation are separate | Project product-catalog model, Store projection, scopes and beta features differ; do not assume a single read path |
| Contentful | CMA requires current version for optimistic locking; whole-body updates can remove omitted fields; draft, publish, locale and delivery state differ | Plans, environment, locale publishing, residency, rate limits and content model are tenant-specific |
| Algolia | Index writes are asynchronous and return tasks; task completion and a query fixture should both be checked; full replace is a distinct high-blast-radius operation | SDK signatures, ACLs, record limits, replicas, billing and task latency vary; pin client and index configuration |

## Qualification ladder

For every operation, store evidence for each gate:

| Gate | Test | Required artifact |
|---|---|---|
| Q0 — documentation | Current official reference, release/deprecation notes, scope/role, schema and stated limits reviewed | Dated source record and contradiction/limitation note |
| Q1 — static contract | Request/response/error schemas, destructive fields, identity and lifecycle mapped | Versioned adapter contract and fixtures |
| Q2 — simulator | Timeout-before-send, timeout-after-send, throttle, partial result, duplicate/out-of-order event and schema drift exercised | Fault-injection report with expected states |
| Q3 — provider test account | Exact read/write/read-back path tested with representative product type and permissions | Redacted requests, receipts, job/task IDs and postconditions |
| Q4 — shadow | Production reads and desired diffs compared without write credentials | Drift/coverage/latency/quota report |
| Q5 — live canary | One narrow approved effect under manual watch and independent read-back | Proposal, approval, effect ledger, observable result, rollback drill |
| Q6 — sustained admission | Load, peak, recovery, audit, privacy, support and refresh owners pass | Signed operation admission with expiry and kill switch |

An adapter can be admitted for a read while its write remains rejected. Qualification belongs to the tuple `(provider, operation, API release, account class, product type, region, credential, adapter release)`, not the provider logo.

## Worked flow 1 — catalog and content remediation

**Scenario:** a marketplace suppresses a shoe variant because size system and material claims are incomplete.

1. Resolve product, variant, SKU, seller, account, marketplace, listing and current product-type schema exactly.
2. Read the current PIM revision, claim registry, DAM assets, provider input, processed issues and last-known-good projection.
3. Run deterministic required-field, enumeration, claim-evidence, locale and image-role checks.
4. Let the model explain interacting issues and draft bounded field candidates. It cites source fields and marks missing evidence; it cannot invent a material composition or certification.
5. Product/content owners adopt acceptable text into a new PIM or CMS draft revision.
6. Deterministically transform the approved revision into a provider-native immutable proposal. Show adds, changes, clears and deletions.
7. Re-read source and product-type schema. Obtain exact approval for the target variant/listing.
8. Commit through the qualified single-item adapter, persist receipt or `unknown`, then reconcile provider issues and discoverable state.
9. If the suppression remains, keep the original source revision and effect evidence; create a new correction proposal rather than silently mutating the first attempt.

**Success:** exact processed attributes and no blocking issue, with an observable listing when the provider exposes it. A polished draft alone is not success.

## Worked flow 2 — exact price and promotion across channels

**Scenario:** pricing has approved a seven-day INR sale for one offer on a storefront, Google promotion, and marketplace listing.

```mermaid
sequenceDiagram
    participant PS as Pricing/promotion service
    participant C as Commerce coordinator
    participant A as Approval service
    participant S as Storefront adapter
    participant G as Google adapter
    participant M as Marketplace adapter
    participant R as Reconciler

    PS->>C: signed exact decision + interval + targets
    C->>C: simulate stacking, tax basis, clocks, channel mappings
    C->>A: immutable three-channel proposal digest
    A-->>C: exact bounded approval
    C->>S: effect S
    C->>G: effect G
    C->>M: effect M
    S-->>C: accepted
    G-->>C: promotion in review
    M--xC: timeout after send
    C->>R: reconcile all item effects independently
    R-->>C: S verified; G pending; M unknown
```

Rules:

- create one child effect per channel and one parent objective; never call the parent `complete` while a child is pending or unknown;
- the exact amount, currency, tax basis, eligible offers, coupon/benefit, stacking rule and interval come from signed services and policy, not model output;
- do not “roll back all” automatically if one channel lags—the storefront may already have presented the sale to customers;
- freeze any late child whose approval/effective window expires; escalate with observed cross-channel inconsistency;
- if the marketplace request timed out after send, read listing/offer state before resubmission; and
- legal/pricing owners decide whether to withdraw, extend, correct, or communicate an inconsistency.

## Worked flow 3 — flash-sale availability contradiction

**Scenario:** the storefront says available, a marketplace quantity event says 12, but OMS evidence has expired and checkout failures rise.

1. Enter the wrong-availability incident lane; stop promotion/publication work that depends on the stale evidence.
2. Preserve each observation with item, location/allocation, fulfillment context, event sequence, source time and validity window.
3. Ask the OMS/inventory owner for a fresh authoritative availability projection. Do not average conflicting quantities.
4. Protect inventory, reconciliation and emergency-correction queues from enrichment traffic.
5. If a pre-authorized policy says stale availability plus checkout failure requires withdrawal, execute the narrow deterministic runbook; otherwise request the named commerce/inventory decision.
6. Never call a channel quantity mutation merely to make channels agree.
7. Reconcile all affected offers and watch recovery load: restored providers can release delayed events and scheduled scans simultaneously.

**Load gate:** with the enrichment queue saturated and one provider throttling, the correction lane must meet its target queue age, hot merchants must not starve others, and no retry storm may exceed the account governor.

## Worked flow 4 — marketplace publication with partial processing

**Scenario:** a 500-item approved assortment is published through a bulk feed; 470 verify, 20 reject, and 10 have no item result.

| State | Count | Next action |
|---|---:|---|
| Verified | 470 | Close child effects; retain postcondition evidence |
| Definitive reject | 20 | Classify issue; correct source/schema only through new proposals |
| Unknown or missing result | 10 | Query feed/item status and exact listings; suppress equivalent resubmission |

The feed job is a transport parent. Persist all 500 child effect keys before submission, compare approved/submitted/reported/read-back target digests, and treat a missing processing-result document as unknown—not failure and not success. A human may override the desired publication plan, but the override is a versioned decision that supersedes named children; it does not erase attempts.

## Worked flow 5 — return signal to governed hypothesis

**Scenario:** size-related returns rise for one product family.

1. Receive only a purpose-approved aggregate with reason-taxonomy version, market, variant denominators, window, late-arrival watermark and suppression threshold.
2. Check selection bias, low denominator, campaign, stock, price, season and product-change confounders.
3. Compare the actual size-system attributes, guide, imagery and variant labels against approved source content.
4. Produce a hypothesis and counter-evidence, not a causal claim or customer-level action.
5. Have the product/merchandising owner adopt a candidate change and analytics owner define the experiment or monitoring plan.
6. Evaluate conversion, returns, accessibility, support contacts and margin observations separately. Never auto-promote the “winner” into a standing rule.

This flow stays out of customer support, physical returns and refunds. Any need for a case-level remedy becomes a minimized typed handoff to the owning workflow.

## Worked flow 6 — unknown effect and human override

**Scenario:** a live price mutation times out, then an operator changes the price directly in the provider console.

1. Mark the adapter attempt `unknown_after_send`; retain request hash, start/end time, credential subject and transport evidence.
2. Acquire the target reconciliation lease and read exact current price, currency, context, interval and provider update actor/version if available.
3. Compare observed state with the timed-out proposal and any newer authorized intent.
4. If the proposal is present, verify it. If the manual value is current and authorized, mark the first effect `superseded`; do not overwrite it.
5. If authority for the manual value is unknown, open a wrong-price incident and freeze automated correction. A persuasive model explanation is not authority evidence.
6. A reviewer records one of: accept manual state, restore last-known-good, apply a newly approved value, withdraw the offer, or keep unresolved pending investigation.
7. Execute the chosen response as a new correction effect and verify customer-visible state where feasible.

## Multi-channel compensation matrix

| Effect | Safe cancellation before send | After accepted or unknown | Typical correction |
|---|---|---|---|
| Content draft | Cancel draft/task | Archive or supersede draft | New reviewed source revision |
| Live content | Cancel proposal | Reconcile exact field and publication | Publish last-known-good or new approved content |
| Publication | Cancel scheduled attempt | State may already be visible | New approved withdraw/publish effect |
| Price | Invalidate uncommitted approval | Customer/legal exposure may exist | Pricing/legal chooses correction, withdrawal, or no action |
| Promotion | Cancel before provider submission/effective time if supported | Provider review or redemption may already exist | Suspend/end/correct under policy; never rewrite history |
| Search projection | Drop queued derived update if no task submitted | Task may execute; wait/query index | New derived update or controlled reindex |

Compensation is business-specific. “Reverse every successful child when one fails” is unsafe because publication, discount display, checkout and customer reliance can make a simple inverse wrong.

## Consumer, seller, product, and brand safety gates

Before any live offer or recommendation effect, deterministic policy must establish:

- responsible seller/trader identity and channel-account binding;
- product safety, traceability, recall/withdrawal and required economic-operator fields for the applicable market;
- restricted/prohibited goods eligibility, age/location controls, licensing and platform policy;
- substantiated claims, endorsements/reviews integrity, brand authorization, trademark/copyright and asset license;
- all material price, fee, unit-price, reference-price, promotion and availability disclosures;
- accessibility requirements and human checks that automated catalog validation cannot prove;
- whether personalization or segmentation is permitted, explainable and free of prohibited/sensitive-trait use;
- privacy purpose, minimization, consent where required, retention and deletion;
- separation of duties for seller onboarding, price/promotion authority, policy exception and bulk publication; and
- fraud boundary: suspicious seller, review, coupon or order behavior is handed to Fraud/AML or marketplace trust, not self-investigated through broader customer data.

For EU consumer marketplaces, the Digital Services Act includes trader-traceability duties and the General Product Safety Regulation adds marketplace/product-safety obligations. WCAG 2.2 is a current W3C Recommendation, but conformance and legal applicability require accessibility specialists and jurisdiction-specific review. FTC advertising, review and deceptive-pricing materials likewise establish design risk, not a universal compliance rule. This playbook is not legal advice.

## Fault-injection and acceptance suite

| Fault | Expected invariant |
|---|---|
| Shopify list member omitted from `productSet` fixture | Preview identifies deletion; target-set approval changes; no hidden commit |
| commercetools version changes after approval | `ConcurrentModification` or commit precheck stops overwrite; proposal invalidates |
| Contentful entry changes between read and update | Version conflict is surfaced; connector re-reads and never submits a stale full body blindly |
| Google ProductInput insert succeeds but processed Product is absent | Child remains submitted/pending; no approval or visibility claim |
| Google promotion insert omits a prior field | Whole-resource replacement test catches loss before live call |
| Amazon feed reports parent done but omits item rows | Missing children remain unknown and reconcile by listing read |
| Amazon issue notification payload version changes | Event is quarantined; scheduled reconciliation preserves current-state correctness |
| Algolia task is delayed behind a queue | Search projection remains pending; source product does not change |
| OMS evidence expires one millisecond before price/promo commit | Dependent action stops or obtains a new evidence version |
| Provider applies request then connection drops | Same semantic effect is not blindly resent |
| Human changes live state during reconciliation | Current actor/version and supersession are evaluated; no automatic overwrite |
| Restricted product contains “ignore policy” in description/image OCR | Input remains data; no tool, target, policy or approval changes |
| Return aggregate cell is below privacy threshold | Case is rejected before model context construction |
| Flash-sale backlog plus regional failover | correction/reconciliation reserve survives and recovery traffic is admission-controlled |

## Hands-on progression

| Exercise | Build | Measurable pass evidence |
|---|---|---|
| E0 — no model | Exact identity map, source/channel diff, manual correction and reconciliation | 100% critical identity conflicts stop; every manual effect has receipt/read-back |
| E1 — read-only diagnosis | Pinned schemas, D0/D1 tools, evidence-cited suppression report | Beats deterministic/operator baseline on reviewed semantic cases; zero write discovery |
| E2 — provider test flow | Immutable proposal, approval, D2 draft/test operation for two adapters | Destructive semantics, expiry, version conflict, async and partial tests pass |
| E3 — crash-safe saga | Durable three-channel objective, clocks, effects, unknown and handoff | Worker/queue restart loses no child; restart receipt invariant comparison passes |
| E4 — one live effect | One narrow content or publication canary, manual watch and read-back | Exact intended/observed state; rollback/correction drill completes; no unresolved critical unknown |
| E5 — peak rehearsal | Hot tenant, throttle, event storm, provider outage and recovery surge | Defined queue-age/fairness/reconciliation targets met; correction reserve preserved |
| E6 — governed change | Update one model/prompt/adapter/policy artifact as a full behavior bundle | Shadow/canary, drift detection, signed release, compatible rollback and controlled failure case added |

Record the threshold values before each exercise. Do not invent universal SLOs or canary sizes; derive them from business risk, provider behavior and the Stage 0 baseline.

## Production review checklist

- [ ] Every connector is qualified per operation and pinned to an API/schema/account/credential tuple.
- [ ] Reads, drafts, live effects, bulk effects and administrative changes use separate capabilities.
- [ ] Provider-native identity, concurrency, replacement, async, quota and lifecycle semantics remain visible.
- [ ] Every external child effect has a semantic key, ledger state, receipt/unknown classification and read-back.
- [ ] Cross-channel parents report partial truth and never collapse pending/unknown children into success.
- [ ] PIM, OMS, pricing, tax, support, payments, fulfillment and fraud owners retain their authority.
- [ ] Restricted goods, seller, claims, price, brand, accessibility, privacy and marketplace gates are jurisdiction-reviewed.
- [ ] Customer communication and customer-level data remain outside this agent's default tools.
- [ ] Provider test accounts, fault injection, shadow, canary, correction and recovery-load drills have current evidence.
- [ ] Operation admission has an owner, expiry, refresh triggers, kill switch and last-known-compatible rollback.

## Primary sources

- [Google Merchant API overview](https://developers.google.com/merchant/api/reference)
- [Google Merchant: add and manage products](https://developers.google.com/merchant/api/guides/products/add-manage)
- [Google Merchant: processed products and issues](https://developers.google.com/merchant/api/guides/products/list-products-data-issues)
- [Google Merchant Promotions sub-API](https://developers.google.com/merchant/api/guides/promotions/overview)
- [Amazon: manage product listings with SP-API](https://developer-docs.amazon.com/sp-api/docs/manage-product-listings-guide)
- [Amazon Feeds API best practices](https://developer-docs.amazon.com/sp-api/lang-US/docs/feeds-api-best-practices)
- [Amazon Orders API](https://developer-docs.amazon.com/sp-api/docs/orders-api)
- [Amazon notification type values](https://developer-docs.amazon.com/sp-api/docs/notification-type-values)
- [Shopify `productSet`](https://shopify.dev/docs/api/admin-graphql/latest/mutations/productSet)
- [Shopify webhooks](https://shopify.dev/docs/apps/build/webhooks)
- [commercetools general concepts](https://docs.commercetools.com/api/general-concepts)
- [commercetools Product Projections](https://docs.commercetools.com/api/projects/productProjections)
- [Akeneo product concepts](https://api-prd.akeneo.com/concepts/products.html)
- [Contentful Content Management API overview](https://www.contentful.com/developers/docs/references/content-management-api/overview/)
- [Algolia asynchronous index operations](https://www.algolia.com/doc/guides/sending-and-managing-data/send-and-update-your-data/in-depth/index-operations-are-asynchronous)
- [GS1 GTIN Management Standard](https://www.gs1.org/1/gtinrules/en/)
- [FTC advertising and marketing guidance](https://www.ftc.gov/business-guidance/advertising-marketing)
- [FTC endorsements, influencers, and reviews](https://www.ftc.gov/business-guidance/advertising-marketing/endorsements-influencers-reviews)
- [EU Digital Services Act](https://eur-lex.europa.eu/eli/reg/2022/2065/oj/eng)
- [EU General Product Safety Regulation](https://eur-lex.europa.eu/eli/reg/2023/988/oj/eng)
- [W3C WCAG 2.2](https://www.w3.org/TR/WCAG22/)

[← Previous: Deployment, peak scale, cost, and governed evolution](10-deployment-peak-scale-cost-and-governed-evolution.md) · [Blueprint home](README.md)

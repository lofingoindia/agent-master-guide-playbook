# Research Packet: Sales and Revenue Operations Agent Blueprint

> **Research completed:** 2026-08-31  
> **Derived guide set:** [Sales and Revenue Operations Agent](../../agents/sales-revenue-operations-agent/README.md)  
> **Research posture:** Primary sources first; provider behavior and regulatory guidance checked for current status; operational conclusions synthesized rather than copied  
> **Scope:** Account/contact identity, evidence and enrichment, CRM integration, pipeline and routing, forecast assistance, quote preparation, electronic outreach, calls/transcripts, meetings, governed warehouse projections, durable execution, security, privacy, observability, evaluation, deployment, and cost

## Executive finding

The technically sound production design is a **constrained hybrid workflow**, not an autonomous outbound agent. A deterministic control plane owns identity linkage, tenant and territory authorization, consent/suppression, routing rules, commercial configuration, approvals, effect commits, and durable state. Models are useful for evidence synthesis, semantic extraction, ambiguity detection, drafting, and bounded next-step proposals.

This conclusion follows from four independent realities:

1. CRM systems expose provider-specific revisions, object relationships, assignment semantics, limits, and change-feed behavior that cannot be safely abstracted into an unconstrained “CRM tool.”
2. Identity and enrichment are uncertain and time-dependent; a domain or email does not prove a person/legal-entity relationship.
3. Outreach eligibility depends on sender, recipient, location, channel, purpose, relationship, evidence, objections, and changing jurisdiction/provider rules.
4. External APIs cannot generally provide end-to-end exactly-once effects; durable intent, idempotency, and reconciliation are application responsibilities.

## Research method

Research began by mapping neighboring production blueprints and shared control-plane, runtime, tool, reliability, security, and evaluation guides in this repository. Internet research then covered these question groups:

- What object, association, concurrency, assignment, forecasting, quote, API-version, limit, and change-feed semantics do major CRM platforms document?
- Which identity-resolution methods and authoritative company sources support defensible account/contact linkage?
- What do current primary regulatory and regulator-published materials require or emphasize for commercial email, telemarketing, privacy, objections, and suppression?
- What do Gmail, Yahoo, calendar, mail, and quote providers require for safe delivery and idempotent/reconcilable effects?
- How do current CRM conversation-intelligence, call/transcript, Salesforce authentication, and warehouse access surfaces change consent, retention, visibility, version, and adapter-qualification requirements?
- Which durable workflow and HTTP semantics govern crashes, retries, and ambiguous external outcomes?
- Which current standards and official guidance cover OAuth, prompt injection, privacy, AI risk, trace context, telemetry, and evaluation?

Source priority was:

1. laws, regulations, RFCs, standards, regulators, and official government data sources;
2. official provider API documentation, help, limits, and changelogs;
3. official repositories and maintained reference implementations;
4. framework documentation for implementation semantics.

Search snippets were not treated as sufficient evidence for important claims. Dated or preview material was either labeled as volatile or excluded from normative recommendations. No legal source was converted into a universal global rule; local counsel remains responsible for the policy bundle.

## Evidence-to-decision map

| Design question | Strong evidence | Production decision |
|---|---|---|
| Can a model directly update CRM records? | Salesforce and Dataverse document conditional/resource-specific concurrency; provider schemas and limits differ | Use narrow adapters, field allowlists, expected revisions, previous values, and conflict outcomes |
| Are webhooks the source of truth? | Salesforce CDC retention/replay constraints; HubSpot retry behavior | Treat events as invalidation hints; persist/deduplicate; authoritative read and periodic reconciliation |
| Can domain/email similarity auto-merge accounts/people? | Record-linkage theory and registry identifier semantics | Use exact verified IDs first, probabilistic candidates second, a review band, and reversible links before merges |
| Can the model decide outreach legality? | FTC, FCC, GDPR/ePrivacy, ICO, CRTC, and California guidance vary by context and change over time | Deterministic, versioned, counsel-approved communication policy; model cannot return `allow` |
| Is an approval a general go-ahead? | Mutable CRM, consent, policy, content, and pricing state | Bind approval to a canonical digest and expiry; revalidate immediately before commit |
| May a timed follow-up send automatically? | Opt-outs, replies, ownership, and opportunity state can change during waits | Timer means re-evaluate; refresh every mutable precondition and invalidate stale approval |
| Can a failed POST be retried? | RFC 9110 and provider idempotency semantics | An ambiguous result enters reconciliation; retry only with the same operation identity after proving absence |
| Is a message ID enough to deduplicate sends? | Mail APIs separate draft/send but do not promise universal application deduplication | Send immutable provider draft once, persist receipt, reconcile sent state before any retry |
| Should the model set price or accept a quote? | Stripe/Dynamics model catalog and quote state as governed commercial data | Model creates typed draft requests; CPQ/catalog and named approvers own price, terms, and finalization |
| Can CRM stage probability be called the forecast? | CRM providers expose configured stages/categories; predictive models use historical outcomes | Separate stage probability, calibrated model probability, seller category, and manager override |
| Does model/provider conversation state replace workflow state? | Model APIs and durable runtime documentation have different persistence/replay guarantees | Keep authoritative case/effect state in application storage or durable workflow runtime |
| Can prompt defenses be primarily instructional? | OWASP and NIST emphasize injection/tool-abuse risk and layered controls | Isolate untrusted research, keep credentials outside the model, enforce typed tools and server-side policy |
| Is a CRM-associated transcript authoritative opportunity truth? | HubSpot and Dynamics document provider-specific association, media, licensing, processing, consent, storage and retention behavior | Keep time-coded transcript evidence separate; validate speaker, identity, consent and business update before CRM mutation |
| Can the model query a revenue warehouse directly? | BigQuery and Snowflake expose fine-grained controls with documented limits; query/write APIs have distinct job and retry semantics | Use named parameterized point-in-time projections, query receipts and read-only credentials; prohibit arbitrary SQL and reverse CRM mutation |
| Does a working OAuth integration prove least privilege? | Provider app type, scope, delegated principal and source-system row/field security all affect effective visibility | Qualify permissions with positive/negative tenant, record and field tests; never rely on model filtering after broad retrieval |

## Current-version baseline and volatility

| Area | Baseline observed on 2026-08-31 | Refresh trigger |
|---|---|---|
| HubSpot CRM API | 2026-03 dated API is documented as the latest track for new integrations; dated versioning and EOL windows matter | New dated version, deprecation, limit, association, or pipeline change |
| Salesforce | REST/Bulk/Composite and Pub/Sub behavior is org/edition/resource specific; CDC event retention documented as 72 hours | Seasonal release, org limit/configuration change, CDC/API notice |
| Salesforce client application | New connected-app creation is restricted as of Spring ’26; Salesforce recommends external client apps for new integrations | Authentication/app-policy release, migration guidance or org policy change |
| Microsoft Dataverse/Dynamics 365 | Alternate keys and optimistic concurrency are documented; sales scoring/licensing/configuration are tenant-dependent | Release-wave or API/feature/license change |
| Call/transcript systems | HubSpot documents a 2026-03 calling/transcript surface; Dynamics documents feature-specific consent, storage, retention, license and security differences | API track, media format, license, storage, consent, retention or security-control change |
| Warehouse projections | BigQuery and Snowflake access controls and API limitations are service/edition/configuration dependent | IAM/policy, regional, SQL API, streaming/CDC or feature-status change |
| Google Calendar | Event-creation guide observed updated 2026-08-27; client event IDs support duplicate prevention | Calendar API behavior or scope change |
| Gmail sender rules | Bulk and general sender requirements remain enforced; thresholds and rollout status are operational configuration | Google Postmaster/sender guideline update |
| Yahoo sender rules | Bulk-sender practices include authentication, reputation, and easy unsubscribe | Yahoo sender guidance change |
| NIST Privacy Framework | 1.0 remains the final baseline; 1.1 is an active project/draft, not a final replacement | Publication of final 1.1 |
| NIST AI RMF | AI RMF 1.0 remains published and is being revised; GenAI Profile 600-1 is published | Revised AI RMF/profile publication |
| OpenTelemetry GenAI | Semantic conventions and attributes are evolving, with movement into a dedicated repository | Stable release/schema migration |
| US telemarketing/TCPA | FCC consent-revocation implementation and waivers changed in 2024–2026 | FCC/court/state/counsel update |
| ICO direct marketing | Guidance pages show 2026 updates and some data-broker guidance notes policy changes/review | UK law or ICO guidance update |
| OpenAI model/API | Model names, snapshots, supported features, and guidance are volatile | Model/API deprecation, snapshot or feature change |

Version-sensitive values should live in connector/policy configuration and capability tests, not prose constants or model prompts.

## CRM and commercial-system findings

### Salesforce

- Conditional request behavior exists but ETag/`If-Match` support is not universal across every resource. Adapter capabilities must be endpoint-specific.
- Composite requests can coordinate dependent subrequests and rollback behavior, but partial/batch results still need per-record accounting.
- External-ID upsert is useful for stable integration identities; nonunique external-ID matches are not a safe automatic choice.
- CDC/platform events have a finite 72-hour retention window; replay IDs are opaque and should not be treated as permanent sequence numbers.
- API limits vary by org and may change; Bulk API 2.0 is appropriate to evaluate for larger record sets rather than synchronous loops.
- Assignment rules are ordered and stop at a match; local routing logic must preserve configured ordering and a catch-all.
- Forecast categories and reports represent organization-defined sales semantics, not an objective model probability.

### HubSpot

- The CRM object model is association-rich; association direction and type are part of meaning.
- Pipeline stage IDs/configuration are provider data. For deals, probability configuration is part of the pipeline definition and cannot be inferred from the label.
- Dated API versioning introduced a concrete compatibility lifecycle; connectors need pinned paths and upgrade tests.
- API limits are account/app/endpoint dependent; HubSpot recommends batching, caching, and webhooks, and returns 429 for limits.
- Webhook failures can be retried multiple times over an extended window; consumers need quick durable acknowledgment and deduplication.
- An announced 2026-09 validation change illustrates why changelog monitoring belongs in operations.

### Microsoft Dataverse and Dynamics 365

- Alternate keys can identify records without GUIDs but must exist in table metadata and have documented character/operation caveats.
- `If-Match` and change tracking provide mechanisms for concurrency and synchronization; the adapter must propagate conflicts rather than overwrite.
- Assignment uses segments, sequences, ordered rules, capacity, availability, and assignment strategies; processing is not necessarily immediate.
- Opportunity scoring is trained from historical outcomes and has data/configuration prerequisites. It is assistance, not universal truth.
- Quote price calculations rely on product catalogs, price lists, units, quantities, discounts, and configuration. Custom pricing is explicit system logic.

### Stripe as a quote/idempotency reference

- Quote lifecycle distinguishes draft, open/finalized, accepted, and canceled states.
- Price history is preserved through new Price objects rather than mutating unit amounts.
- Idempotency keys on POST return the stored outcome for the same request, including some failures, and reject parameter mismatch; provider retention windows still require application reconciliation.

## Identity and evidence findings

Fellegi–Sunter record linkage formalizes match/nonmatch evidence rather than treating a similarity score as truth. Production systems also need candidate blocking, locally calibrated thresholds, a clerical-review band, and asymmetric costs: a false merge is often much worse than a missed link.

Authoritative identifiers are scoped:

- GLEIF's LEI data supports legal-entity identity and ownership relationships for participating entities.
- SEC EDGAR APIs provide real-time submissions/XBRL and nightly bulk data for US public-company filings, subject to access policy.
- Companies House exposes live UK company information and a published default rate limit.
- CRM external/alternate keys support integration identity only when the target configuration establishes their uniqueness.

No source proves every desired relationship. A registry ID can establish a legal entity but not that an email address is a current employee, that a group domain maps to one subsidiary, or that a contact has consented to marketing.

Evidence therefore requires source, observation/effective time, method, rights, sensitivity, and expiry. Summaries must retain claim-level citations. Licensed enrichment is not system-of-record authority and does not itself establish permission to contact.

## Communication, privacy, and deliverability findings

### United States

The FTC states that CAN-SPAM covers commercial email, including business-to-business messages. It requires truthful routing/header and subject information, identification and postal details as applicable, a working opt-out path, and honoring opt-outs within the specified period. Outsourcing does not transfer all responsibility.

The FTC's Telemarketing Sales Rule and FCC TCPA materials introduce separate calling/texting, consent, revocation, Do-Not-Call, technology, timing, and recordkeeping concerns. Recent FCC orders and waivers make this a high-volatility policy area. The blueprint consequently avoids embedding a universal dial/text rule and requires counsel-approved jurisdiction/channel logic.

California's CCPA materials include rights to know, delete, correct, and opt out of sale/sharing, plus recognition of Global Privacy Control. Activation systems and enrichment stores must connect to the rights process.

### EU and United Kingdom

GDPR principles relevant to the blueprint include minimization, accuracy, storage limitation, transparency for indirectly sourced personal data, and the right to object to direct marketing. Article 21 requires direct-marketing processing to stop after objection. The ePrivacy framework and national implementation determine channel-specific electronic-marketing rules; a simplified “legitimate interest always permits B2B email” rule is unsafe.

ICO guidance distinguishes corporate subscribers from sole traders and some partnerships for UK electronic marketing, while UK GDPR can still apply whenever named business-contact data is personal data. Legitimate interests require a documented purpose, necessity, and balancing assessment. Data-broker users must conduct due diligence rather than rely on supplier assurances.

### Canada

CRTC CASL guidance says commercial electronic messages generally require consent, sender identification, and an unsubscribe mechanism. Implied-consent conditions and time limits require evidence, and the sender needs to be able to prove the basis used.

### Mail providers

Gmail sender guidelines require baseline authentication and operational practices for all senders and stronger SPF/DKIM/DMARC alignment, one-click unsubscribe, and reputation controls for bulk traffic to personal Gmail. Google documents a 0.3% spam-rate ceiling and recommends staying lower. Yahoo publishes comparable bulk-sender practices. RFC 8058 defines the one-click mechanism.

Compliance and deliverability are distinct: meeting legal conditions does not ensure provider acceptance or good recipient experience. The system needs both policy gates and technical reputation/rate controls.

## Mail and calendar effect findings

Gmail's API separates draft creation/update from send. This enables review of immutable provider drafts but does not prove exactly-once delivery. Google Calendar lets clients choose event IDs, explicitly noting duplicate-prevention use. Microsoft Graph's event resource includes a client-provided `transactionId` for reducing redundant POSTs after retries, and delta queries support incremental calendar changes.

The resulting adapter pattern is prepare → approve exact digest → commit once → store provider IDs → reconcile ambiguous outcomes. A message header ID is correlation evidence, not a universal provider deduplication contract.

## Conversation evidence, warehouse, and integration-qualification findings

### Call and transcript surfaces

HubSpot's dated 2026-03 calling-extension documentation treats calls as CRM activity objects that must be associated with CRM records for timeline/transcript use. Its recording integration requires an authenticated recording URL and documents media/channel requirements. The separate transcript endpoint accepts time-bounded utterances and speaker data. These mechanics do not prove the association, speaker identity, or business meaning; they strengthen the case for preserving provider IDs, association evidence, utterance offsets, and uncertainty.

Dynamics 365 conversation-intelligence documentation makes feature boundaries operationally important. Microsoft documents license/security-role dependencies, storage and retention choices, participant consent/privacy responsibilities, feature-specific processing delays and media limitations, and security-control differences from the main CRM. Those facts reject a design that treats “Dynamics data” as one uniform retention or encryption boundary.

The synthesized decision is to separate four records: recording notice/consent evidence, immutable media/transcript artifact, model-derived claim/action candidate, and accepted CRM update. Outreach eligibility is not recording consent. A transcript association is not identity truth. Speaker attribution, transcription, translation, sentiment, and summarization are measurements with failure modes, not authoritative opportunity fields.

### Warehouse projections and forecast vintages

BigQuery documents authorized views plus row- and column-level controls; the documentation also describes regional/authorization constraints and interactions or limitations across features. Snowflake documents row-access policy semantics and limitations, including differences among tables, views, streams and materialized views. Fine-grained warehouse controls are useful defense in depth but do not remove the need for tenant/purpose assertions and negative access tests in the application.

The Snowflake SQL API documents request IDs and a retry flag for ambiguous/retried statement execution, as well as concurrency/rate behavior. This is provider assistance rather than end-to-end exactly-once application behavior. State-changing SQL remains an effect with intent, scope, receipt, and reconciliation.

For model-facing sales work, named and parameterized authorized views are safer than arbitrary SQL. Query receipts should preserve named-query/schema version, principal, purpose, policy, tenant/territory, as-of cutoff, source watermarks, provider job/statement ID, rows/bytes, and result digest. Historical forecast evaluation must distinguish event, ingestion, correction, and label-availability times so later outcomes cannot leak into earlier vintages.

### Provider admission rather than connector trust

Salesforce now recommends external client apps for new integrations and restricts new connected-app creation from Spring ’26. This is a concrete example of authentication architecture becoming stale independently from object APIs. Connector qualification therefore includes app type, OAuth flow, scopes, source-system user/record/field permissions, tenant automation, endpoint capability, schema/version, limits, effect semantics, revocation, retention and negative access tests.

Qualification expires. An integration is admitted per capability—not per vendor logo—after contract, permission, failure, ambiguity, quota, privacy and operational tests. An expired or weakened capability fails closed for new high-impact work while preserving historical receipts and reconciliation.

## Durable execution and HTTP findings

RFC 9110 defines idempotent HTTP methods and warns clients not to automatically retry non-idempotent requests unless they know repetition is safe. `If-Match` protects against lost updates when the resource supplies appropriate validators. RFC 9457 provides a standard structure for machine-readable problem details.

Temporal resumes workflows after failures. DBOS documents durable workflows and warns that steps with external effects can run again, so those effects need idempotency. Restate offers service, per-key virtual-object, and workflow semantics. None makes an arbitrary external provider transaction exactly once by itself.

The blueprint therefore uses an effect ledger, stable operation identity, conditional writes where supported, provider idempotency where documented, reconciliation on ambiguity, and manual review when absence cannot be proven.

## Model/runtime and evaluation findings

The OpenAI Responses API documents typed function tools, structured outputs, conversation/previous-response state, background processing, storage options, and tool-call limits. Current model guidance recommends evaluation, clear tool definitions, and snapshot/feature-aware migration. These capabilities can simplify an implementation but do not move policy or business authority into the provider.

Model/provider state is non-authoritative. The application records its own case state, evidence, decisions, approvals, effects, model/API version, and evaluation trace.

NIST AIRC frames testing, evaluation, verification, and validation as part of operationalizing AI risk management. The blueprint combines deterministic invariants, connector contract tests, historical scenario evaluation, repeated stochastic runs, adversarial injection, failure injection, shadowing, canaries, production SLOs, and sampled human audit. Model-graded quality is supplementary; exact policy/effect invariants use deterministic assertions.

The unit of behavioral release is the evaluated bundle, not only the model: prompts, inference settings, context compiler, compaction schema, retrieval policy, output schemas, tool catalog, adapter-capability manifest, and routing/forecast artifacts can each change behavior or authority. Production corrections and incidents enter a privacy-reviewed curation path, are verified against authoritative systems, and become eval fixtures, deterministic assertions, fault injections, or runbooks. They do not automatically become cross-run memory or training data.

## Security findings

RFC 9700 is the current OAuth 2.0 Security Best Current Practice and strengthens redirect, PKCE, issuer, refresh-token, TLS, and client-authentication guidance. The application still needs provider-specific least privilege, short-lived credentials, tenant mapping, secret isolation, and revocation.

OWASP agent and prompt-injection guidance supports layered controls: untrusted-content isolation, least privilege, typed tools, human approval, monitoring, and adversarial testing. No delimiter or prompt can make web/email content trustworthy.

NIST AI RMF and its Generative AI Profile support govern/map/measure/manage practices and attention to content risks, privacy, security, and monitoring. NIST Privacy Framework provides a complementary privacy-risk structure; 1.1 should not be treated as final until NIST publishes it.

OpenTelemetry's GenAI conventions are evolving and may carry sensitive prompts/outputs. A stable local trace schema and opt-in content capture are necessary.

## Rejected designs

| Rejected pattern | Why rejected | Safer alternative |
|---|---|---|
| One broad “sales agent” with CRM, browser, mailbox, and CPQ tools | Prompt injection and model error can span every trust boundary | Isolated research plus typed effect adapters and server policy |
| Domain/email exact match as identity truth | Shared domains, job changes, aliases, subsidiaries | Canonical graph, authoritative keys, probabilistic candidates, review band |
| Consent/marketability boolean in CRM | Loses purpose, channel, evidence, jurisdiction, withdrawal, and history | Consent/suppression ledger plus policy decision record |
| Approval of a campaign or editable draft | Recipient/content/state can change after review | Approval bound to canonical digest, source revisions, and expiry |
| Retry every 5xx/timeout | Can duplicate messages, invites, quotes, or records | Unknown outcome plus authoritative reconciliation |
| Treat stage probability or LLM confidence as forecast | Miscalibration and configured semantics | Leakage-safe calibrated model, baselines, seller/manager judgment |
| Long-running loop stored in chat history | Loses timers, conflicts, audit, cancellation | Durable state machine/workflow with refreshed context projections |
| Shared vector store/cache without prefilter | Cross-tenant and purpose leakage | Tenant/territory enforcement before retrieval and cache access |
| Fully autonomous outbound from day one | Couples legal, identity, reputation, and irreversible risks | Read-only → drafts → internal writes → supervised effects → narrow automation |
| CRM-associated transcript as stage/forecast truth | Association, speaker, transcription, consent and timing can be wrong | Restricted utterance evidence → cited candidate → owning validation → typed CRM proposal |
| Arbitrary model-generated warehouse SQL | Scope bypass, cost blowout, nondeterministic metrics and point-in-time leakage | Named parameterized authorized projection plus query receipt and budget |
| Vendor-level connector approval | One vendor exposes many permissions, endpoints, versions and failure semantics | Capability-level declaration, expiry, contract/fault tests and requalification |
| Automatic memory from successful or corrected traces | Success may be confounded; traces contain personal/tenant data and bad instructions | Verified, minimized, curated eval/runbook artifacts with deletion lineage |

## Source register

All sources below were consulted or cross-checked for this packet. Dates in provider documentation should be rechecked during implementation.

### CRM, pipeline, routing, forecasting, and quotes

1. [Salesforce REST API Developer Guide](https://resources.docs.salesforce.com/latest/latest/en-us/sfdc/pdf/api_rest.pdf) — object APIs, conditional requests, errors.
2. [Salesforce sObject rows by external ID](https://www.postman.com/salesforce-developers/salesforce-developers/request/lxoduhc/sobject-rows-by-external-id) — external-ID lookup/upsert behavior.
3. [Salesforce Composite requests](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_composite_composite_post.htm) — dependent requests and rollback controls.
4. [Salesforce event durability](https://developer.salesforce.com/docs/platform/pub-sub-api/guide/event-message-durability.html) — 72-hour retention and replay-ID constraints.
5. [Salesforce API limits and monitoring](https://developer.salesforce.com/blogs/2024/11/api-limits-and-monitoring-your-api-usage) — quota visibility and operational guidance.
6. [Salesforce limits quick reference](https://resources.docs.salesforce.com/256/10/en-us/sfdc/pdf/salesforce_app_limits_cheatsheet.pdf) — current platform limits and Bulk API guidance.
7. [Salesforce assignment rules](https://help.salesforce.com/s/articleView?id=sf.creating_assignment_rules.htm&language=en_US&type=5) — ordered first-match semantics.
8. [Salesforce territory assignment](https://help.salesforce.com/s/articleView?id=sales.tm2_assign_accounts_to_territories.htm&language=en_US&type=5) — planning/activation behavior.
9. [Salesforce Pipeline Inspection](https://help.salesforce.com/s/articleView?id=sales.pipeline_inspection.htm&language=en_US) — pipeline changes and consolidated inspection.
10. [Salesforce forecast report semantics](https://help.salesforce.com/s/articleView?id=forecasts3_reports_crt_understand.htm&language=en_US&type=0) — forecast categories and organizational configuration.
11. [HubSpot API overview and versioning](https://developers.hubspot.com/docs/reference/api/overview?Tag=Reports) — dated API lifecycle.
12. [HubSpot CRM object model](https://developers.hubspot.com/docs/api-reference/latest/crm/understanding-the-crm) — objects, properties, and associations.
13. [HubSpot associations guide](https://developers.hubspot.com/docs/api-reference/latest/crm/associations/associate-records/guide) — association direction/type behavior.
14. [HubSpot pipelines guide](https://developers.hubspot.com/docs/api-reference/latest/crm/pipelines/guide) — pipeline and stage administration.
15. [HubSpot create pipeline stage](https://developers.hubspot.com/docs/api-reference/latest/crm/pipelines/stages/create-pipeline-stage) — deal-stage probability configuration.
16. [HubSpot API usage guidance](https://developers.hubspot.com/docs/developer-tooling/platform/usage-guidelines) — rate limits, batching, caching, and 429 handling.
17. [HubSpot error handling](https://developers.hubspot.com/docs/api-reference/error-handling) — webhook and workflow retry behavior.
18. [HubSpot webhook guide](https://developers.hubspot.com/docs/api-reference/latest/webhooks/guide) — subscription model.
19. [HubSpot pipeline validation change](https://developers.hubspot.com/changelog/pipeline-stage-validation-true) — example forward compatibility change.
20. [Microsoft Dataverse alternate keys](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/use-alternate-key-reference-record) — alternate-key references and constraints.
21. [Microsoft Dataverse HTTP/concurrency guidance](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/compose-http-requests-handle-errors) — `If-Match`, change tracking, errors.
22. [Dynamics 365 work assignment overview](https://learn.microsoft.com/en-us/dynamics365/sales/work-assignment-intro) — segments, sequences, and rules.
23. [Dynamics 365 assignment rules](https://learn.microsoft.com/en-us/dynamics365/sales/wa-create-and-activate-assignment-rule) — first-match, capacity, and availability behavior.
24. [Dynamics 365 predictive opportunity scoring](https://learn.microsoft.com/en-us/dynamics365/sales/configure-predictive-opportunity-scoring) — historical-data and model configuration.
25. [Dynamics 365 quote lifecycle](https://learn.microsoft.com/en-us/dynamics365/sales/create-edit-quote-sales) — draft/active/closed quote handling.
26. [Dynamics 365 price calculation](https://learn.microsoft.com/en-us/dynamics365/sales/price-calculation-opportunity-quote-order-invoice-records) — catalog-derived price logic.
27. [Dynamics 365 price lists](https://learn.microsoft.com/en-us/dynamics365/sales/create-price-lists-price-list-items-define-pricing-products) — product/price-list configuration.
28. [Stripe Quotes](https://docs.stripe.com/quotes) — quote lifecycle.
29. [Stripe products and prices](https://docs.stripe.com/products-prices/how-products-and-prices-work?locale=en-GB) — immutable price-history approach.
30. [Stripe idempotent requests](https://docs.stripe.com/api/idempotent_requests) — idempotency-key semantics.

### Identity, company data, and evidence

31. [U.S. Census Bureau: Fellegi–Sunter record linkage](https://www.census.gov/library/working-papers/1991/adrm/rr91-09.html) — probabilistic linkage foundation.
32. [U.S. Census Bureau record-linkage overview](https://www.census.gov/content/dam/Census/library/working-papers/2006/adrm/rrs2006-02.pdf) — practical statistical linkage context.
33. [GLEIF API](https://www.gleif.org/en/lei-data/gleif-api) — LEI search and legal-entity data.
34. [GLEIF access and use of LEI data](https://www.gleif.org/en/lei-data/access-and-use-lei-data) — ownership and identifier coverage.
35. [SEC EDGAR APIs](https://www.sec.gov/search-filings/edgar-application-programming-interfaces) — submissions, XBRL, and bulk data.
36. [Companies House developer API](https://developer.company-information.service.gov.uk/) — UK public-company data.
37. [Companies House developer guidelines](https://developer.company-information.service.gov.uk/developer-guidelines) — access and rate limits.
38. [Splink](https://github.com/moj-analytical-services/splink) — maintained probabilistic-linkage implementation reference.

### Outreach, privacy, and deliverability

39. [FTC CAN-SPAM compliance guide](https://www.ftc.gov/business-guidance/resources/can-spam-act-compliance-guide-business) — US commercial-email requirements.
40. [FTC Telemarketing Sales Rule guide](https://www.ftc.gov/business-guidance/resources/complying-telemarketing-sales-rule) — telemarketing/DNC requirements.
41. [FCC 2024 TCPA revocation order](https://docs.fcc.gov/public/attachments/FCC-24-24A1_Rcd.pdf) — consent-revocation requirements.
42. [FCC 2026 waiver order](https://docs.fcc.gov/public/attachments/DA-26-12A1.pdf) — evidence of current rule volatility.
43. [EUR-Lex GDPR](https://eur-lex.europa.eu/eli/reg/2016/679/oj/eng/) — privacy principles, indirect collection, objection.
44. [EUR-Lex consolidated ePrivacy text](https://eur-lex.europa.eu/legal-content/EN/TXT/PDF/?uri=CELEX%3A02009L0136-20201221) — electronic-communication marketing framework.
45. [EDPB Guidelines 1/2024 on Article 6(1)(f)](https://www.edpb.europa.eu/public-consultations/guidelines-12024-on-processing-of-personal-data-based-on-article-61f-gdpr_en) — legitimate-interest analysis; consultation-version status noted.
46. [ICO electronic-mail marketing guidance](https://ico.org.uk/for-organisations/direct-marketing-and-privacy-and-electronic-communications/guidance-on-direct-marketing-using-electronic-mail/) — UK PECR electronic-mail rules.
47. [ICO B2B marketing guidance](https://ico.org.uk/for-organisations/direct-marketing-and-privacy-and-electronic-communications/business-to-business-marketing/) — corporate/individual subscriber distinctions and UK GDPR.
48. [ICO lead generation and purchased data](https://ico.org.uk/for-organisations/direct-marketing-and-privacy-and-electronic-communications/direct-marketing-guidance/collect-information-and-generate-leads/) — due diligence and suppression.
49. [ICO data-broker user guidance](https://ico.org.uk/for-organisations/direct-marketing-and-privacy-and-electronic-communications/organisations-using-marketing-services-of-data-brokers/) — buyer responsibilities.
50. [CRTC CASL guide](https://crtc.gc.ca/eng/com500/guide.htm) — consent, identification, and unsubscribe.
51. [CRTC CASL FAQ](https://www.crtc.gc.ca/eng/com500/faq500.htm) — scope and consent evidence details.
52. [California CCPA](https://oag.ca.gov/privacy/ccpa) — consumer rights and Global Privacy Control.
53. [Gmail sender guidelines](https://support.google.com/mail/answer/81126?hl=en) — authentication, unsubscribe, and spam-rate requirements.
54. [Gmail sender-guideline FAQ](https://support.google.com/mail/answer/14229414?hl=en) — enforcement and operational detail.
55. [Yahoo sender best practices](https://senders.yahooinc.com/best-practices/) — bulk-sender expectations.
56. [RFC 8058](https://datatracker.ietf.org/doc/html/rfc8058) — one-click list unsubscribe.

### Email, calendar, durable effects, and standards

57. [Gmail API reference](https://developers.google.com/workspace/gmail/api/reference/rest) — draft/message/send separation.
58. [Gmail authorization scopes](https://developers.google.com/workspace/gmail/api/auth/scopes) — least-privilege inputs.
59. [Google Calendar event creation](https://developers.google.com/workspace/calendar/api/guides/create-events) — client event IDs and notifications.
60. [Google Calendar Events resource](https://developers.google.com/workspace/calendar/api/v3/reference/events) — event and conferencing fields.
61. [Microsoft Graph create event](https://learn.microsoft.com/en-us/graph/api/user-post-events?view=graph-rest-1.0) — event creation and `transactionId`.
62. [Microsoft Graph event resource](https://learn.microsoft.com/en-us/graph/api/resources/event?view=graph-rest-1.0) — event identity and fields.
63. [Microsoft Graph calendar delta](https://learn.microsoft.com/en-us/graph/api/event-delta?view=graph-rest-1.0) — incremental synchronization.
64. [RFC 9110](https://datatracker.ietf.org/doc/html/rfc9110) — HTTP idempotency and conditional semantics.
65. [RFC 9457](https://datatracker.ietf.org/doc/html/rfc9457) — machine-readable API problem details.
66. [Temporal documentation](https://docs.temporal.io/) — durable workflow recovery.
67. [DBOS workflow tutorial](https://docs.dbos.dev/typescript/tutorials/workflow-tutorial) — durable workflow semantics.
68. [DBOS external-step tutorial](https://docs.dbos.dev/golang/tutorials/step-tutorial) — at-least-once external-step warning.
69. [Restate services](https://docs.restate.dev/foundations/services) — keyed concurrency and durable workflow semantics.
70. [CloudEvents specification](https://github.com/cloudevents/spec) — event envelope/versioning reference.
71. [W3C Trace Context](https://www.w3.org/TR/trace-context/) — trace propagation.

### Models, security, privacy, observability, and evaluation

72. [OpenAI Responses API](https://developers.openai.com/api/reference/cli/resources/responses/methods/create) — structured outputs, tools, background and state options.
73. [OpenAI model guidance](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-5.5) — evaluation, snapshots, prompting, and tool guidance.
74. [OpenAI Evals API](https://developers.openai.com/api/reference/java/resources/evals/methods/create) — evaluation structures.
75. [RFC 9700](https://datatracker.ietf.org/doc/html/rfc9700) — OAuth 2.0 security best current practice.
76. [RFC 9449](https://datatracker.ietf.org/doc/html/rfc9449) — proof-of-possession for OAuth access tokens.
77. [OWASP AI Agent Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html) — agent threats and layered controls.
78. [OWASP Prompt Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html) — direct/indirect injection defenses.
79. [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework) — AI risk governance baseline and revision status.
80. [NIST Generative AI Profile](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf) — generative-AI risk actions.
81. [NIST AI Resource Center](https://airc.nist.gov/) — testing, evaluation, verification, and validation resources.
82. [NIST Privacy Framework](https://www.nist.gov/privacy-framework) — privacy risk management.
83. [NIST Privacy Framework 1.1 project](https://www.nist.gov/privacy-framework/new-projects/privacy-framework-version-11) — draft/development status.
84. [OpenTelemetry semantic conventions](https://opentelemetry.io/docs/specs/semconv/) — telemetry schema baseline.
85. [OpenTelemetry GenAI attribute registry](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/) — GenAI fields and sensitivity considerations.
86. [OpenTelemetry GenAI semantic conventions](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/README.md) — evolving dedicated specification.
87. [Google SRE: service-level objectives](https://sre.google/sre-book/service-level-objectives/) — SLO/error-budget operational model.

### Pass 2 integration, conversation, and warehouse sources

88. [Salesforce external client apps and connected apps](https://developer.salesforce.com/docs/platform/mobile-sdk/guide/connected-apps.html) — Spring ’26 connected-app restriction and external-client-app direction.
89. [Salesforce Spring ’26 developer guide](https://developer.salesforce.com/blogs/2026/01/developers-guide-to-the-spring-26-release) — current platform authentication and API release context.
90. [HubSpot call recordings and transcripts](https://developers.hubspot.com/docs/api-reference/latest/crm/extensions/calling-extensions/recordings-and-transcriptions) — dated call object, association, authenticated media, and transcription requirements.
91. [HubSpot create transcript endpoint](https://developers.hubspot.com/docs/api-reference/latest/crm/extensions/transcriptions/create-transcript) — utterance, speaker, and time-offset contract.
92. [Dynamics 365 conversation-intelligence retention and privacy](https://learn.microsoft.com/en-us/dynamics365/sales/data-retention-deletion-policy-sales-app) — storage, retention, consent/privacy, role, and use limitations.
93. [Dynamics 365 conversation-intelligence FAQ](https://learn.microsoft.com/en-us/dynamics365/sales/faq-conversation-intelligence) — licensing, processing, media, storage, and feature constraints.
94. [Dynamics 365 Sales privacy and security FAQ](https://learn.microsoft.com/en-us/dynamics365/sales/sales-privacy-faqs) — data location and conversation-intelligence security-control differences.
95. [Dataverse conversation transcript entity](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/reference/conversationtranscript?view=dataverse-latest) — provider entity surface.
96. [BigQuery authorized views](https://cloud.google.com/bigquery/docs/authorized-views) — constrained data sharing and documented limitations.
97. [BigQuery row-level security](https://cloud.google.com/bigquery/docs/row-level-security-intro) — row-filtering access-control surface.
98. [Snowflake row-access policies](https://docs.snowflake.com/en/user-guide/security-row-intro) — policy evaluation, supported objects, interactions, limitations, and audit.
99. [Snowflake SQL API request submission](https://docs.snowflake.com/en/developer-guide/sql-api/submitting-requests) — asynchronous execution, request IDs, retries, ambiguity, concurrency, and rate handling.

## Remaining limitations and implementation research

- This packet does not determine a legal basis or permitted communication for a particular organization. Counsel must map actual senders, recipient locations/types, products, channels, data sources, and contracts into the policy bundle.
- CRM behavior depends on edition, tenant configuration, permissions, custom objects/automation, installed packages, and API version. Run capability discovery and sandbox contract tests against each target tenant.
- No enrichment provider was selected. Vendor rights, geographic coverage, accuracy, refresh, deletion, subprocessors, and contract terms require a separate procurement review.
- No call/meeting provider or recording jurisdiction policy was selected. Participant locations, notice/consent, media topology, transcription quality, employee monitoring, storage, retention, deletion and provider security controls require tenant-specific validation.
- BigQuery and Snowflake are representative warehouse evidence, not a mandate. The target platform's IAM, row/column policies, point-in-time model, query identity, cost, retention and write/reconciliation semantics need independent qualification.
- Forecast thresholds cannot be specified without local point-in-time data, baselines, decision cost, and cohort analysis.
- Model/provider selection requires task-specific quality, latency, cost, privacy, retention, region, and failure evaluation. The blueprint intentionally does not prescribe one model.
- Deliverability thresholds, mail-provider enforcement, CRM API limits, FCC/FTC/ICO guidance, state/national rules, and API versions must be monitored continuously.
- The architecture describes controls and contracts, not a vendor certification. Each control needs an owner, test, operational signal, and evidence of correct deployment.

## Refresh checklist

Refresh this packet when any of the following occurs:

- a new CRM dated API/release wave, endpoint deprecation, association/schema change, or quota policy;
- a new model snapshot or tool/structured-output behavior;
- privacy, electronic-marketing, telemarketing, consent-revocation, or AI-governance change in an operating jurisdiction;
- Gmail/Yahoo or another major mailbox-provider sender-rule change;
- a new enrichment/identity source or contract;
- a new call, recording, transcript, CPQ, warehouse, forecasting, or reverse-ETL capability;
- an incident involving identity, consent, tenancy, approval, duplicate effects, or model injection;
- expansion to a new channel, country, product, price/quote authority, or autonomous effect.

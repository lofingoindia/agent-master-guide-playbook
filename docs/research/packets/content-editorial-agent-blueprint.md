# Content Production and Editorial Operations Agent Research Packet

> **Research cut-off:** 2026-08-31
>
> **Packet status:** Primary-source synthesis for category #49
>
> **Target blueprint:** [Content Production and Editorial Operations Agent](../../agents/content-editorial-agent/README.md)
>
> **Scope warning:** This packet is engineering research, not legal advice, a rights opinion, an accessibility conformance claim, or provider certification.

## Research question and category boundary

The research question was: **what evidence, state, authority, and effect contracts are required for a content-production assistant to help move an approved assignment from brief through research, draft, review, release, correction, and archive without silently becoming the source of truth, rights clearancer, editor of record, or publisher?**

The category owns:

- assignment truth and audience/channel/brand/style contracts;
- research-to-claim handoff, source locators, support status, and expiry;
- content identity, structured fields, versions, redlines, review findings, and approval bindings;
- media and asset references, licenses, consent/model releases, attribution, transformations, and accessibility metadata;
- editorial, legal, brand, accessibility, SEO, and product-data review routing;
- schedule proposals, deterministic publication execution, receipts, reconciliation, cancellation, correction, withdrawal, retraction, and archive history.

The category does **not** own:

- campaign populations, consent segmentation, experimentation, spend, or outcome attribution, which belong to marketing operations;
- protected-source handling, public-interest verification, or investigative fairness judgments, which belong to journalism;
- multilingual release parity and translation quality, which belong to localization;
- OCR, layout extraction, and document structure recovery, which belong to document intelligence;
- final legal claims, rights clearance, brand exceptions, accessibility conformance declarations, or publication authority, which remain with accountable people and policy.

## Method

Research prioritized normative specifications, regulators, official product documentation, official repositories, and maintainers. Product pages were used only to establish adapter behavior or version state, not to validate vendor performance claims. Current pages were checked near the cut-off; dated standards retain their publication dates.

The synthesis deliberately separates:

1. what a source normatively states;
2. what a provider currently implements;
3. what the blueprint infers as a safe cross-provider contract; and
4. what remains an organization-, jurisdiction-, tenant-, or channel-specific policy choice.

## High-confidence findings and engineering consequences

### 1. The assignment is the first durable authority object

A chat request is not sufficient authority to create or release content. The system needs an immutable `assignment_version` with purpose, accountable owner, intended audience, approved channels, claims allowed and prohibited, jurisdiction flags, brand/style policy versions, due dates, disclosure policy, and explicit publication authority. Every later draft and approval binds to its digest.

This is an engineering inference from the provider version/concurrency controls, legal variation, and provenance standards below. No CMS API turns an informal prompt into organizational authority.

### 2. Structured content must remain structured through generation and review

Contentful distinguishes management and delivery APIs and requires a current version on update. Sanity stores queryable JSON documents and distinguishes published, drafts, versions, and release perspectives. Akeneo product values are attribute-, locale-, and channel-scoped. These are not interchangeable rich-text blobs.

The assistant should draft a typed content object against a versioned schema, not emit HTML as the only artifact. Rendering to web, email, social, or help-center markup is a deterministic projection that can be validated independently.

Primary basis: [Contentful CMA overview](https://www.contentful.com/developers/docs/references/content-management-api/overview/), [Sanity Content Lake](https://www.sanity.io/docs/content-lake), [Sanity perspectives](https://www.sanity.io/docs/content-lake/perspectives), [Akeneo API reference](https://api.akeneo.com/api-reference-index.html), and [JSON Schema Draft 2020-12](https://json-schema.org/draft/2020-12).

### 3. Draft, published, release, and provider version are separate coordinates

Sanity's API default perspective changed to `published` for API version `2025-02-19`; Contentful uses resource versions and environments; WordPress exposes drafts, future posts, autosaves, and revisions. A portable adapter cannot collapse these into one integer called `version`.

The blueprint uses:

- `content_revision_id` for the internal immutable revision;
- `provider_object_id` for the remote object;
- `provider_version_token` for optimistic concurrency;
- `publication_generation` for one externally visible release lineage;
- `release_id` for a coordinated provider or internal release; and
- `render_digest` for the exact channel representation approved.

Primary basis: [Contentful environments](https://www.contentful.com/developers/docs/references/content-management-api/environments/), [Sanity perspectives](https://www.sanity.io/docs/content-lake/perspectives), and [WordPress post revisions](https://developer.wordpress.org/rest-api/reference/post-revisions/).

### 4. Optimistic concurrency is a safety boundary, not an inconvenience

Contentful requires `X-Contentful-Version` for updates. HTTP `If-Match` exists to prevent lost updates. JSON Patch can describe typed edits and combine with a precondition. The assistant must not fetch, deliberate, and overwrite a newer human edit.

All write adapters need compare-and-set semantics or an emulated preflight check. A mismatch produces `conflict_requires_rebase`; it never triggers a blind retry.

Primary basis: [RFC 9110 conditional requests](https://www.rfc-editor.org/rfc/rfc9110.html#section-13), [RFC 6902 JSON Patch](https://www.rfc-editor.org/rfc/rfc6902.html), and the [Contentful CMA](https://www.contentful.com/developers/docs/references/content-management-api/overview/).

### 5. A source record and a claim record are different objects

W3C PROV distinguishes entities, activities, and agents. Web Annotation supplies text quote and position selectors, with explicit brittleness and copyright cautions. Therefore the system stores an acquired source representation and separately stores narrow claims with support/contradiction edges, locators, extraction method, reviewer state, and freshness.

A URL in a bibliography is not enough. A claim must identify the representation used, the exact supporting location, what the source says, what the draft says, and whether the transformation preserved meaning.

Primary basis: [W3C PROV overview](https://www.w3.org/TR/prov-overview/), [PROV-O](https://www.w3.org/TR/prov-o/), and [Web Annotation selectors](https://www.w3.org/TR/selectors-states/).

### 6. Provenance does not establish factual truth or rights clearance

C2PA 2.3 supports cryptographically verifiable provenance assertions and a defined trust model. It does not turn an assertion into a truth verdict. IPTC metadata can carry creator, rights, and AI-related fields, but embedded metadata can be wrong, missing, stripped, or stale. ODRL expresses permissions, prohibitions, constraints, and duties; it does not prove that the policy issuer owns the rights.

The blueprint reports provenance validation and rights evidence separately. `c2pa_valid`, `license_declared`, or `copyright_notice_present` can never set `rights_cleared=true`.

Primary basis: [C2PA specifications 2.3](https://spec.c2pa.org/specifications/specifications/2.3/index.html), [C2PA Content Credentials specification](https://spec.c2pa.org/specifications/specifications/2.3/specs/C2PA_Specification.html), [IPTC Photo Metadata 2025.1](https://www.iptc.org/std/photometadata/specification/IPTC-PhotoMetadata-2025.1.html), [ODRL 2.2](https://www.w3.org/TR/odrl-model/), and [IPTC RightsML 2.0](https://iptc.org/std/RightsML/2.0/RightsML_2.0-specification.html).

### 7. Rights are use-, channel-, territory-, time-, and transformation-specific

Creative Commons licenses illustrate why a license label alone is insufficient: attribution, indication of changes, noncommercial, share-alike, and no-derivatives terms differ. ODRL explicitly models constraints and duties. Asset approval must bind to the proposed use, transformation, territory, audience, channel, and time interval.

The assistant may reject obviously incompatible uses and assemble a rights packet. An accountable human or approved policy service decides clearance.

Primary basis: [CC BY 4.0 deed and legal-code link](https://creativecommons.org/licenses/by/4.0/), [Creative Commons license overview](https://creativecommons.org/share-your-work/use-remix/cc-licenses/), and [ODRL 2.2](https://www.w3.org/TR/odrl-model/).

### 8. AI-assisted authorship and disclosure are jurisdiction-sensitive

The U.S. Copyright Office's January 2025 Part 2 report says wholly AI-generated material is not copyrightable in the United States and that prompts alone do not provide sufficient human control, while human selection, arrangement, and modification may be protectable. The EU AI Act's Article 50 transparency obligations apply from 2 August 2026, with scope and exceptions that depend on the content and review context.

The system must preserve human contributions, model/provider/build, input-policy version, material transformations, and disclosure decisions. It must not produce a universal `copyright_owner` or `ai_label_required` answer.

Primary basis: [U.S. Copyright Office AI initiative](https://www.copyright.gov/ai/), [Copyright and AI Part 2](https://www.copyright.gov/ai/Copyright-and-Artificial-Intelligence-Part-2-Copyrightability-Report.pdf), [EU AI Act text](https://eur-lex.europa.eu/eli/reg/2024/1689/oj), and the [European Commission Article 50 FAQ](https://digital-strategy.ec.europa.eu/en/faqs/transparency-obligations-under-article-50-ai-act).

### 9. Facts, expression, trademark, privacy, publicity, and contract rights cannot be collapsed

U.S. Copyright Office guidance says facts, ideas, systems, names, titles, and short phrases are not protected by copyright in the same way as original expression, while other rights may still apply. A content operation still needs separate checks for copyrighted expression, trademarks, privacy/personal data, publicity/personality rights, confidential information, contractual restrictions, and platform terms.

Primary basis: [U.S. Copyright Office Circular 33](https://www.copyright.gov/circs/circ33.pdf), [copyright FAQ](https://www.copyright.gov/help/faq/faq-protect.html), and [Circular 22 on status research](https://www.copyright.gov/circs/circ22.pdf).

### 10. Accessibility automation is necessary but cannot prove conformance

WCAG 2.2 is a W3C Recommendation republished 12 December 2024 and approved as ISO/IEC 40500:2025. WCAG says testable criteria require both automated testing and human evaluation, and even AAA does not address every user need. U.S. federal, U.S. state/local, and EU legal baselines differ and evolve; for example, the 2026 U.S. Title II interim rule moved some compliance dates while retaining WCAG 2.1 AA as its technical standard.

The assistant may generate alt-text candidates, headings, labels, transcripts, captions, and reading-level signals. It may not declare conformance. Human testing with assistive technology and the complete rendered process remains required under policy.

Primary basis: [WCAG 2.2](https://www.w3.org/TR/wcag/), [WCAG documents and ACT links](https://www.w3.org/WAI/standards-guidelines/wcag/docs/), [U.S. DOJ small-entity guide updated for the 2026 interim rule](https://www.ada.gov/resources/small-entity-compliance-guide/), [Section 508 applicability](https://www.section508.gov/develop/applicability-conformance/), and the [European Accessibility Act overview](https://commission.europa.eu/strategy-and-policy/policies/justice-and-fundamental-rights/disability/european-accessibility-act-eaa_en).

### 11. Search metadata is a hypothesis sent to a third-party system

Sitemaps, canonical links, robots directives, and structured data influence crawling and presentation but do not guarantee indexing, canonical selection, or rich results. The system should validate consistency between visible content and structured data, use absolute canonical URLs, record submission receipts, and observe the external result later.

SEO checks must never rewrite a factual claim simply to satisfy a score.

Primary basis: [Google sitemap guidance](https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap), [canonical URL guidance](https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls), [robots meta specification](https://developers.google.com/search/docs/crawling-indexing/robots-meta-tag), and [Schema.org](https://schema.org/).

### 12. Scheduling is an external effect with provider-specific limits

Contentful scheduled actions have status, version checks, per-minute/entity limits, time-zone fields, cancellation, and failure states. Sanity deprecated its older Scheduling API in favor of Scheduled Drafts and Content Releases; large releases can publish in batches rather than atomically. Mailchimp separates schedule, unschedule, send, and cancel actions.

The portable contract therefore promises an intended release window and a reconciliation process, not global atomic publication. All schedules bind to an approved release digest and expire on material change.

Primary basis: [Contentful Scheduled Actions](https://www.contentful.com/developers/docs/references/content-management-api/scheduled-actions/), [Sanity Content Releases](https://www.sanity.io/docs/studio/content-releases), [Sanity deprecated Scheduling API](https://www.sanity.io/docs/http-reference/scheduling), and [Mailchimp schedule API](https://mailchimp.com/developer/marketing/api/campaigns/schedule-campaign/).

### 13. Publication needs intent, approval, effect, and observation records

HTTP method semantics do not make an arbitrary CMS publish endpoint exactly-once. Some providers expose idempotency keys; others rely on version preconditions or remote object identifiers. A network timeout after a publish request leaves an unknown outcome.

The system must persist a publication intent before calling a provider, include a stable operation key when supported, capture provider request IDs, and reconcile by reading the target. It must never blindly repeat an indeterminate publish.

Primary basis: [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html), [Stripe's concrete idempotency semantics](https://docs.stripe.com/api/idempotent_requests), and provider APIs in this packet. Stripe is an example of explicit provider semantics, not a claim that CMS APIs behave identically.

### 14. Corrections and retractions are new versions and effects, not overwrites

IPTC NewsML-G2 2.35 contains update/correction and publishing-status semantics. A content system must retain what was released, what was known, who approved it, which claims/assets were affected, the corrective wording, and whether each downstream destination acknowledged the change.

`corrected` is not terminal until required projections and caches reconcile. A tombstone may coexist with an internally retained record under legal/records policy.

Primary basis: [IPTC NewsML-G2 2.35 Guidelines](https://www.iptc.org/std/NewsML-G2/guidelines/).

### 15. DAM identity, public delivery identity, and bytes are different

Cloudinary documents immutable `asset_id`, mutable public IDs, delivery URL versions, derived assets, backups, and metadata-overwrite settings. Adobe AEM distinguishes asset metadata APIs, deprecated binary-upload paths, direct binary upload, and delivery APIs. The system must preserve a stable asset reference, exact source bytes/digest, approved rendition, transformation recipe, and delivery locator independently.

Primary basis: [Cloudinary Admin API](https://cloudinary.com/documentation/admin_api), [Cloudinary backups/version management](https://cloudinary.com/documentation/backups_and_version_management), [Cloudinary transformations](https://cloudinary.com/documentation/image_transformations), [AEM Assets HTTP API](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/admin/mac-api-assets), and [AEM delivery APIs](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/dynamicmedia/dynamic-media-open-apis/deliver-assets-apis).

### 16. Product descriptions must not become the product system of record

Akeneo recommends UUID product endpoints because business identifiers such as SKUs can change, supports channel/locale-scoped values, drafts/proposals, asset families, and workflows, and documents concrete concurrency/rate limits. Product content may be drafted from approved product facts, but price, safety, ingredients, compatibility, dimensions, warranty, and regulatory fields remain authoritative PIM/ERP/policy data.

The assistant can propose narrative fields and flag inconsistency. It cannot invent or overwrite governed product facts.

Primary basis: [Akeneo REST API reference](https://api.akeneo.com/api-reference-index.html), [API good practices](https://api.akeneo.com/documentation/good-practices.html), and [pagination guidance](https://api.akeneo.com/documentation/pagination.html).

### 17. Social and email adapters are high-drift effect surfaces

LinkedIn's Marketing APIs require dated version headers, publish monthly versions, support them for a limited window, restrict permissions, and expose published/draft/processing states. Mailchimp and SendGrid distinguish create/schedule/send/cancel operations and webhook/event surfaces. Marketing still owns recipients, consent, send strategy, and outcomes.

The content system produces an approved channel payload and may hand it to an authorized executor. Qualification must test edit/delete/correction capabilities, version sunsets, media processing, audience defaults, rate limits, receipts, and regional data handling.

Primary basis: [LinkedIn Marketing API versioning](https://learn.microsoft.com/en-us/linkedin/marketing/versioning?view=li-lms-2026-05), [LinkedIn Posts API](https://learn.microsoft.com/en-us/linkedin/marketing/community-management/shares/posts-api?view=li-lms-2026-05), [Mailchimp Marketing API](https://mailchimp.com/developer/marketing/api/), [SendGrid Mail Send API](https://www.twilio.com/docs/sendgrid/api-reference/mail-send), and [SendGrid signed webhooks](https://www.twilio.com/docs/sendgrid/api-reference/webhooks/get-signed-event-webhooks-public-key).

### 18. Retrieved content remains untrusted instructions

Prompt injection can arrive through web pages, source documents, CMS fields, alt text, metadata, comments, assets, and tool output. Retrieval and fine-tuning do not eliminate the risk. Content-derived text must remain typed as data; only the control plane grants tool authority.

Primary basis: [OWASP LLM01:2025 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/), [NIST AI 600-1](https://doi.org/10.6028/NIST.AI.600-1), and the repository's [prompt-injection guide](../../security/prompt-injection-and-untrusted-data.md).

### 19. Telemetry must be useful without becoming a content leak

OpenTelemetry semantic conventions 1.44.0 provide stable conventions in several areas, while generative-AI conventions have moved and contain development-status attributes. Prompt and completion bodies can include unpublished strategy, licensed material, personal data, secrets, and embargoed content.

Default traces should store identifiers, digests, counts, timing, policy results, model/provider/build, and redacted error classes—not raw prompts or content. Sampling, access, region, retention, and deletion must follow content sensitivity.

Primary basis: [OpenTelemetry semantic conventions 1.44.0](https://opentelemetry.io/docs/specs/semconv/) and [GenAI attribute registry](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/).

### 20. Durable workflow and model reasoning have different responsibilities

Multi-day waits, review deadlines, scheduled releases, retries, cancellations, and reconciliation require durable state. Model reasoning may propose the next bounded action but must not be the workflow database. External effects belong in idempotent or reconcilable activities with explicit timeouts and retry policies.

Primary basis: [Temporal failure detection guidance](https://docs.temporal.io/encyclopedia/detecting-activity-failures), [CloudEvents specification](https://github.com/cloudevents/spec/blob/main/cloudevents/spec.md), and repository runtime/reliability guides.

### 21. Seven memory lifetimes need separate admission and deletion rules

The research found no provider feature that safely replaces a memory architecture. The blueprint requires all seven lifetimes:

| Lifetime | Legitimate content use | Admission rule | Typical deletion/correction behavior |
|---|---|---|---|
| Turn/scratch | Parse one request, compare one revision | Never authoritative; discard by default | End of turn |
| Working/run | Current plan, retrieved excerpts, tool results | Run-scoped, provenance-labeled, size-bounded | End/abort or short diagnostic TTL |
| Session | User navigation and non-authoritative continuity | No rights/claim elevation | Session expiry; user deletion |
| Durable workflow/task | Assignment, revision, reviews, approvals, effects | Typed event/state transition only | Retention schedule; append corrections |
| Domain knowledge | Approved style rules, taxonomies, templates, policies | Curated, versioned, signed/reviewed | Supersede and invalidate dependents |
| Long-term/preference | Approved user/team preferences | Consent, tenant scope, non-sensitive default | Correct/delete on request or role change |
| Episodic/outcome | Released outcome, incidents, evaluation examples | De-identified/minimized; no silent policy | Time-bounded; correction and poisoning review |

Each memory item needs provenance, rights/use basis, tenant, sensitivity, expiry, correction pointer, deletion state, and poisoning review. Model-written summaries never overwrite authoritative records.

### 22. Compaction must be typed and loss-aware

A continuation summary that omits a stale approval, unresolved rights question, publish uncertainty, or cancellation request can cause a real-world error. The blueprint requires a continuity receipt naming preserved identities, assignment/policy digests, current revision, unresolved claims/rights, approvals and invalidations, pending/unknown effects, budgets, and omissions.

Compaction is rejected if any required field cannot fit; the run checkpoint points to durable state instead of compressing authority into prose.

### 23. Evaluation needs claim-, rights-, style-, and render-level oracles

Generic “good writing” scores are insufficient. The system needs:

- atomic claim precision/recall against a reviewed claim ledger;
- citation-entailment, source-version, and locator correctness;
- invented quote/number/name/product-fact rate;
- rights-policy decision agreement and unsafe false-clearance rate;
- brand/style rule precision and false-positive burden;
- schema and render validity;
- automated accessibility checks plus human/assistive-technology review;
- approval/effect/reconciliation trajectory correctness;
- correction propagation completeness;
- cost, latency, and reviewer-time impact.

NIST AI 600-1 emphasizes lifecycle measurement and risk management. WCAG explicitly combines testing with human evaluation. Local representative corpora and adversarial cases are mandatory because provider benchmark scores do not establish workflow fitness.

### 24. Behavior change is a governed release artifact

Prompts, system instructions, examples, tools, schemas, model/provider IDs, retrieval rules, safety filters, compaction templates, policy packs, and renderers form one behavior bundle. A model-only rollback is incomplete if the schema or policy changed.

Release gates require offline replay, shadow traffic, human-scored canaries, tenant/region constraints, kill switches, and bundle-level rollback. Controlled failure mining may propose new tests; it cannot silently train on unpublished content or change production behavior.

### 25. The smallest safe agent is usually one bounded loop

Most content operations should begin with templates, rules, search, and deterministic workflow. Add a model only where research selection, synthesis, or revision judgment materially beats the baseline. The recommended loop is:

`compile assignment → retrieve approved evidence → propose claims/outline or patch → validate → request review or stop`

It has no direct publication credential. A deterministic executor performs an approved effect after rechecking the bound revision and policy digests. Multi-agent editorial rooms are not the default; they duplicate context, rights exposure, and merge ambiguity without creating independent evidence.

## Source and version register

| Area | Primary source | Version/date observed | Engineering use | Important limitation |
|---|---|---:|---|---|
| Accessibility | [WCAG 2.2](https://www.w3.org/TR/wcag/) | Recommendation republished 2024-12-12; ISO/IEC 40500:2025 | Content/render acceptance baseline | Does not cover every user need; legal adoption differs |
| Accessibility tests | [W3C ACT/WCAG documents](https://www.w3.org/WAI/standards-guidelines/wcag/docs/) | Checked 2026-08-31 | Separate machine rules from human evaluation | Passing automated rules is not conformance |
| U.S. Title II | [DOJ small-entity guide](https://www.ada.gov/resources/small-entity-compliance-guide/) | Updated after 2026-04-20 interim rule | Jurisdiction policy example | Applies to covered public entities; dates changed in 2026 |
| U.S. federal | [Section 508 applicability](https://www.section508.gov/develop/applicability-conformance/) | Checked 2026-08-31 | Federal electronic-content policy example | Uses incorporated WCAG 2.0 A/AA with exceptions |
| EU accessibility | [European Accessibility Act](https://commission.europa.eu/strategy-and-policy/policies/justice-and-fundamental-rights/disability/european-accessibility-act-eaa_en) | Applied from 2025-06 | EU policy routing | National transposition and scope require legal review |
| AI transparency | [EU AI Act Article 50 FAQ](https://digital-strategy.ec.europa.eu/en/faqs/transparency-obligations-under-article-50-ai-act) | Applies 2026-08-02 | Disclosure policy inputs | Exceptions and roles are fact-specific |
| AI copyright | [USCO AI Part 2](https://www.copyright.gov/ai/Copyright-and-Artificial-Intelligence-Part-2-Copyrightability-Report.pdf) | 2025-01-29 | Preserve human contribution | U.S. position; not a global ownership rule |
| Copyright scope | [USCO Circular 33](https://www.copyright.gov/circs/circ33.pdf) | Current page checked 2026-08-31 | Separate facts/ideas from expression | Other rights can still apply |
| License | [Creative Commons 4.0](https://creativecommons.org/Version4/) | 4.0 | Attribution/change/share restrictions | Rights outside the license can remain |
| Rights policy | [ODRL 2.2](https://www.w3.org/TR/odrl-model/) | W3C Recommendation 2018-02-15 | Machine-readable permissions/constraints/duties | Expression is not ownership proof or legal evaluation |
| Media rights | [IPTC RightsML 2.0](https://iptc.org/std/RightsML/2.0/RightsML_2.0-specification.html) | 2.0 | Media rights adapter vocabulary | Provider support varies |
| Photo metadata | [IPTC Photo Metadata](https://iptc.org/standards/photo-metadata/iptc-standard/) | 2025.1 | Creator, rights, AI metadata capture | Embedded fields are untrusted evidence |
| Content provenance | [C2PA specifications](https://spec.c2pa.org/specifications/specifications/2.3/index.html) | 2.3, December 2025 | Preserve/validate signed provenance assertions | Not a truth or rights verdict |
| Data provenance | [W3C PROV](https://www.w3.org/TR/prov-overview/) | 2013 Recommendation family | Entity/activity/agent lineage | Requires domain profile |
| Claim locators | [Web Annotation selectors](https://www.w3.org/TR/selectors-states/) | 2017 Note based on Recommendation | Quote/position/fragment locators | Positions are brittle; copying text can implicate rights |
| Structured schemas | [JSON Schema](https://json-schema.org/draft/2020-12) | Draft 2020-12, published 2022-06-16 | Validate content/contracts | Implementations and format assertion behavior vary |
| Redlines | [JSON Patch](https://www.rfc-editor.org/rfc/rfc6902.html) | RFC 6902 | Typed patch/redline representation | Rich-text semantic diffs require domain logic |
| HTTP concurrency | [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html) | STD 97, June 2022 | ETag/If-Match/preconditions | Provider APIs may expose different tokens |
| CMS | [Contentful CMA](https://www.contentful.com/developers/docs/references/content-management-api/overview/) | Management media type v1; checked 2026-08-31 | Versioned entry/asset adapter | API limits/features/regions vary by plan |
| CMS scheduling | [Contentful Scheduled Actions](https://www.contentful.com/developers/docs/references/content-management-api/scheduled-actions/) | Checked 2026-08-31 | Schedule/status/cancel semantics | Limits and non-atomic references |
| Structured CMS | [Sanity Content Lake](https://www.sanity.io/docs/content-lake) | Updated 2026-04-15 | JSON structured content | Query/perspective semantics require pinned API date |
| CMS releases | [Sanity Content Releases](https://www.sanity.io/docs/studio/content-releases) | Studio >=3.77; page updated 2026-08-25 | Coordinated releases and previews | Paid feature; large releases can be batched |
| CMS legacy drift | [Sanity Scheduling API](https://www.sanity.io/docs/http-reference/scheduling) | Deprecated; checked 2026-08-31 | Negative qualification test | Migrate to Scheduled Drafts/Releases |
| CMS | [WordPress REST API](https://developer.wordpress.org/rest-api/reference/) | Core REST API; auth page updated 2025-06-04 | Posts, media, revisions | Plugins/custom fields alter behavior |
| DAM | [Cloudinary Admin API](https://cloudinary.com/documentation/admin_api) | Updated 2026-08-24 | Asset IDs, metadata, derivatives | Rate-limited; plan and region differences |
| DAM | [AEM Assets HTTP API](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/admin/mac-api-assets) | Updated 2026-07-07 | Assets metadata/renditions/comments | Some upload/content-fragment paths deprecated |
| PIM | [Akeneo API reference](https://api.akeneo.com/api-reference-index.html) | REST v1; checked 2026-08-31 | Products, drafts, assets, workflows | Edition/version/connection limits vary |
| SEO | [Google Search Central](https://developers.google.com/search/docs/fundamentals/how-search-works) | Checked 2026-08-31 | Render/crawl/index observation | Indexing and canonical selection are not guaranteed |
| Structured web data | [Schema.org](https://schema.org/) | Checked 2026-08-31 | Article/Product/HowTo vocabulary candidates | Search-engine support is a separate contract |
| Correction exchange | [IPTC NewsML-G2 Guidelines](https://www.iptc.org/std/NewsML-G2/guidelines/) | 2.35 | Update/correction/status patterns | News-centric; adapt, do not blindly impose |
| Social | [LinkedIn API versioning](https://learn.microsoft.com/en-us/linkedin/marketing/versioning?view=li-lms-2026-05) | 202608 current at cut-off; monthly versions | Sunset/version qualification | Access approval and scope restrictions |
| Email campaign | [Mailchimp Marketing API](https://mailchimp.com/developer/marketing/api/) | v3.0.91 observed | Schedule/unschedule/send/cancel distinction | Marketing owns recipients and consent |
| Email delivery | [SendGrid Mail Send](https://www.twilio.com/docs/sendgrid/api-reference/mail-send) | Checked 2026-08-31 | Transactional handoff qualification | Regional endpoints and 202/async outcome require observation |
| Privacy | [GDPR Article 5](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A32016R0679) | Regulation (EU) 2016/679 | Purpose/minimization/accuracy/storage/security | Applicability and lawful basis need counsel/privacy owner |
| Privacy engineering | [NIST Privacy Framework](https://www.nist.gov/privacy-framework/privacy-framework) | 1.0 final; 1.1 still draft at cut-off | Data processing/lineage/manageability model | Voluntary, not law |
| Security controls | [NIST SP 800-53 Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final) | Controls dataset 5.1; checked 2026-08-31 | Least privilege, audit, incident controls | Must be tailored |
| GenAI risk | [NIST AI 600-1](https://doi.org/10.6028/NIST.AI.600-1) | July 2024; page updated 2026-04-08 | Evaluation and risk framing | Cross-sector voluntary profile |
| Prompt injection | [OWASP LLM01:2025](https://genai.owasp.org/llmrisk/llm01-prompt-injection/) | 2025 edition | Untrusted-content threat cases | Community guidance, not a guarantee |
| Telemetry | [OpenTelemetry semantic conventions](https://opentelemetry.io/docs/specs/semconv/) | 1.44.0 | Trace/log/metric naming | GenAI conventions partly development/moved |
| Durable runtime | [Temporal failure detection](https://docs.temporal.io/encyclopedia/detecting-activity-failures) | Checked 2026-08-31 | Timeouts, heartbeats, retries | Product-specific implementation |
| Event envelope | [CloudEvents](https://github.com/cloudevents/spec/blob/main/cloudevents/spec.md) | 1.0.x repository | Event ID/source/type/time envelope | Does not define editorial domain semantics |

## Resolved conflicts and required configuration points

| Question | Sources/signals in tension | Blueprint resolution |
|---|---|---|
| Which WCAG version is “the rule”? | W3C recommends WCAG 2.2; U.S. Title II and Section 508 incorporate older versions; EU/national law differs | Keep a jurisdiction- and property-specific accessibility policy pack. Test to the strictest approved engineering baseline, but make legal claims only against the applicable incorporated rule. |
| Does human review remove an AI disclosure duty? | EU Article 50 contains content- and review-specific rules/exceptions; organizations may adopt broader voluntary disclosure | Store generation and review facts; execute a dated jurisdiction policy; escalate ambiguous cases. Never infer universal exemption. |
| Does valid C2PA or IPTC prove rights? | Provenance/metadata standards encode assertions; legal authority can be absent or false | Record validation and issuer; require separate license/consent/clearance decision bound to the use. |
| Can a CMS release be called atomic? | Providers group content, but large or referenced releases can validate, queue, or publish in batches | Define a release window and reconciliation set. Use `partially_visible` and compensating correction/withdrawal rather than claiming global atomicity. |
| Can approved content be reused on another channel? | Licenses, brand rules, accessibility needs, and channel constraints differ | Approval binds to render digest plus channel/use tuple. Re-rendering invalidates affected reviews. |
| Is a URL enough for a citation? | Web content changes; locators can be brittle; copying quoted text may have rights implications | Preserve representation digest/capture time plus locator and limited excerpt where permitted. Revalidate before high-risk publication. |
| Should model memory remember editor preferences? | Personalization helps; stale or poisoned preferences can override current policy | Admit only explicit, low-risk preferences; tenant-scope, version, expire, correct/delete, and subordinate them to current assignment/policy. |
| Should publication be retried after timeout? | Some providers support keys or status lookup; others do not | Move to `effect_unknown`, read/reconcile by stable identifiers, and require operator action if observation is inconclusive. Never blind retry. |
| Should a correction overwrite the original? | Public UX, SEO, platform capabilities, records duties, and legal holds differ | Preserve immutable internal lineage; apply the approved public correction/withdrawal pattern per destination; track acknowledgements. |
| Does accessibility automation pass the item? | Automated rules catch only a subset; WCAG requires full-page/process reasoning and human evaluation | Automation blocks known defects; accountable reviewers own final acceptance and conformance statements. |
| Is a product copy field editable by the content agent? | PIM may expose PATCH for both narrative and governed facts | Allowlist narrative fields and require source-field locks for regulated/product facts. Conflicts route to product-data owners. |

## Adapter qualification questions derived from research

Every CMS, DAM, PIM, storage, search/SEO, accessibility, rights, model, social, and email adapter must answer with executable evidence:

1. What stable object identity survives rename, move, draft, copy, and release?
2. What version/precondition prevents a lost update?
3. Which read view contains published, draft, release, archived, or deleted content?
4. Does a successful response mean accepted, queued, rendered, visible, indexed, or delivered?
5. Is there a client operation key, provider request ID, or lookup that can reconcile an unknown outcome?
6. What are timeout, retry, rate, batch, payload, media, and scheduled-action limits?
7. Can a schedule be canceled, and until which state?
8. Can content be edited, withdrawn, corrected, tombstoned, or restored after release?
9. Do references/assets publish atomically, eventually, or independently?
10. How are webhook authenticity, replay, order, duplication, and schema evolution handled?
11. Which region stores content, logs, webhook buffers, backups, and derived media?
12. Which scopes can create drafts versus publish/delete, and can those credentials be separated?
13. Which metadata survives overwrite/transformation/export?
14. Which provider/API versions and features are deprecated, preview, plan-gated, or scheduled for sunset?
15. How does the adapter expose exports and deletions for retention, data-subject, or tenant-offboarding requests?

## Queries and research paths used

Research was performed through multiple primary-source search paths rather than one summary query:

- current WCAG, ACT, U.S. Title II/Section 508, and EU accessibility baselines;
- C2PA 2.3, IPTC Photo Metadata 2025.1, RightsML, NewsML-G2 2.35, W3C PROV, ODRL, Web Annotation, and Creative Commons 4.0;
- U.S. Copyright Office AI reports and circulars, EU AI Act Article 50 and its August 2026 application guidance;
- Contentful management/delivery/environments/scheduled actions, Sanity Content Lake/perspectives/releases/deprecations, WordPress REST/revisions/authentication;
- Cloudinary asset identity/version/metadata/backup semantics, Adobe AEM asset/deprecation/delivery semantics, Akeneo product/asset/workflow/rate semantics;
- Google crawl/index/canonical/sitemap/robots guidance and Schema.org;
- LinkedIn API versioning/posts, Mailchimp campaign effects, SendGrid send/webhook/region behavior;
- HTTP conditional/idempotent semantics, JSON Patch, JSON Schema, CloudEvents, Temporal failure detection;
- NIST AI 600-1, NIST Privacy Framework, NIST SP 800-53, OWASP prompt injection, and OpenTelemetry conventions.

## Genuine limitations and refresh triggers

- This research does not resolve organization-specific brand ownership, editorial standards, collective-bargaining rules, records schedules, regulated-product duties, or publication delegation.
- Copyright, defamation, privacy, publicity/personality, consumer-protection, accessibility, election, health, financial, product-safety, and AI-disclosure law vary by jurisdiction and facts. Counsel/policy owners must create dated policy packs.
- CMS/DAM/PIM/social/email features are plan-, tenant-, region-, plugin-, and version-dependent. The packet records representative semantics, not certification of a deployment.
- Provider documentation can change without a standards process. Refresh on deprecation notices, API-version sunsets, region changes, or integration incidents.
- C2PA, IPTC, ODRL, and other metadata can improve evidence portability but do not make upstream assertions true.
- Automated accessibility, style, SEO, factuality, and similarity tools have false positives and false negatives. Human review remains essential.
- No universal benchmark proves that a model is safe for a brand, audience, language, jurisdiction, or content type. Build local evaluation sets.
- OpenTelemetry GenAI conventions were still evolving at the cut-off; pin a schema and treat raw-content telemetry as opt-in sensitive data.
- EU AI Act guidance and the 2026 U.S. Title II interim changes are especially time-sensitive. Recheck before making a compliance claim.

Refresh this packet at least every six months, and immediately when any of the following changes: applicable law, WCAG adoption, C2PA/IPTC versions, CMS/DAM/PIM API versions, social/email version sunsets, model/provider data handling, organizational publishing authority, or a material incident.

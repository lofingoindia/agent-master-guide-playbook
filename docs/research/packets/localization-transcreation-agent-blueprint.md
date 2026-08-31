# Localization and Transcreation Operations Agent Research Packet

**Research date:** 2026-08-31  
**Status:** Evidence packet for the production playbook  
**Scope:** Software strings, help content, and marketing transcreation across repository, CMS, TMS, machine-translation, LLM, terminology, translation-memory, human-review, accessibility, and release systems

## Executive synthesis

A production localization agent is not a text-in/text-out translator. It is a controlled operations system that preserves source identity, message syntax, terminology, rights, approvals, and release lineage while coordinating machines and qualified people. The translation candidate is only one intermediate artifact.

The safest practical architecture is a deterministic workflow with a narrow generative step:

1. freeze and identify an exact source release;
2. parse the native format and protect code-like content;
3. compile locale, terminology, style, product, legal, visual, and historical context;
4. retrieve only eligible translation-memory units;
5. generate or import a candidate through a versioned provider adapter;
6. validate syntax, placeholders, terms, locale rules, and provenance deterministically;
7. allow at most one bounded repair pass for machine-repairable defects;
8. route meaning, culture, risk, and ambiguity to qualified human review;
9. stage an exact artifact and approve its digest;
10. commit through an effect ledger, then read back and reconcile the external system;
11. publish only when release-specific locale gates pass; and
12. monitor corrections, parity, incidents, cost, and drift by language, locale, domain, and risk tier.

This packet rejects five tempting shortcuts:

- A BCP 47 language tag is not a complete market, legal, product, or channel policy.
- A high translation-memory match is evidence, not truth.
- A passing automated metric is not a release approval.
- A successful HTTP response is not proof that a repository, CMS, or TMS contains the intended artifact.
- A generated summary is not authoritative workflow state.

## Research method and confidence

The research prioritized current primary material: standards bodies, official platform documentation, official API contracts, maintained repositories, regulations, and peer-reviewed papers. Secondary commentary was excluded where a primary source was available. Important design claims were cross-checked across multiple source types—for example, XLIFF and ITS for data representation, ICU and platform documentation for runtime behavior, ISO abstracts and MQM for review quality, and actual TMS/CMS API documentation for effects.

Evidence labels used below:

| Label | Meaning |
|---|---|
| **Normative/public** | An openly accessible standard, specification, regulation, or normative platform contract was reviewed. |
| **Official/operational** | Official product or project documentation describes behavior, but the behavior may vary by plan, region, or version. |
| **Empirical** | A peer-reviewed benchmark or paper supports the observation; it is not a universal guarantee. |
| **Qualified** | Only an abstract, preview, draft, or status page was accessible; the playbook must not claim full conformance from it. |

### Important evidence limits

- ISO full texts were not available in the public corpus. The guide uses official abstracts, scopes, catalogue status, and public previews only. An implementation must obtain the licensed text and a qualified auditor before claiming ISO conformity.
- Several provider capabilities are plan-, region-, language-pair-, and date-dependent. Every adapter needs a runtime capability probe or pinned capability record; this packet does not freeze vendor marketing claims into permanent architecture.
- MQM is maintained by an industry group and its W3C Community Group work is not a W3C Recommendation. It is useful as a configurable error taxonomy, not a certification.
- Legacy LISA OSCAR specifications remain useful interchange references, but age and preservation status must be explicit.
- Public documentation cannot establish the data-processing agreement, retention configuration, subprocessors, or transfer mechanism for a particular tenant. Those are deployment inputs.

## Category boundary

The agent owns localization operations from an approved source release to a reconciled localized artifact. It does not own the business intent of the source content.

| In scope | Owned elsewhere | Required interface |
|---|---|---|
| Extracting, segmenting, protecting, routing, translating, transcreating, reviewing, staging, and reconciling localized assets | Editorial truth, campaign strategy, product claims, legal advice, regulatory approval, product behavior, and the decision to launch | An immutable source release plus accountable owner, purpose, audience, risk, rights, and target-locale declarations |
| Termbase and translation-memory policy, provenance, eligibility, and correction workflows | The authoritative facts behind product names, claims, warnings, and regulated statements | Versioned facts/claims and owners who can approve changes |
| Locale-specific linguistic and cultural adaptation | Market strategy and acceptance of business or legal consequences | Named market, marketing, legal, accessibility, and linguistic approvers by risk tier |
| Deterministic artifact validation and release evidence | Application correctness outside localization, general CMS administration, and source-repository ownership | Tested format adapters, scoped credentials, and commit/publish APIs |
| Localization parity and correction monitoring | Global incident command unless delegated | Release policy, on-call ownership, and rollback/correction authority |

The simplest non-agent solution is preferable when there is one source file, one locale, a stable qualified translator, and no meaningful cross-system coordination. A CAT/TMS workflow plus human review may be safer and cheaper. Agentic behavior earns its place only when evidence compilation, conditional routing, cross-system effects, reconciliation, and continuity materially reduce operational error.

## Standards and version ledger

| Area | Current evidence at research date | Design consequence |
|---|---|---|
| XLIFF | OASIS XLIFF 2.1 is the latest approved OASIS Standard (2018). ISO 21720:2024 specifies XLIFF 2.0, not 2.1. | Declare the exact version and supported modules. Never label an XLIFF 2.1 feature as ISO 21720 conformance. Preserve unsupported extensions losslessly or reject the file. |
| ITS | W3C Internationalization Tag Set 2.0 is a Recommendation (2013). | Carry translate/no-translate, locale filter, terminology, preserve-space, provenance, and MT-confidence metadata where formats support it. |
| Terminology | ISO 30042:2019 defines TBX; its official status says it is to be revised. | Store concept-oriented terms with language/locale, status, definition, domain, owner, provenance, and lifecycle; record the TBX dialect/version on interchange. |
| Project process | ISO 11669:2024 describes translation-project specifications and management. | Require a project specification: purpose, audience, risks, resources, workflow, acceptance criteria, and responsibilities. |
| Translation services | ISO 17100:2015 plus Amd 1:2017; official status says revision is pending. Raw MT plus post-editing is outside its scope. | Do not claim ISO 17100 merely because a human touched machine output. Define competence and review roles explicitly. |
| Post-editing | ISO 18587:2017 covers full human post-editing of MT output and post-editor competence; revision is pending. | Make post-editing a declared workflow with its own instructions and acceptance criteria, not an invisible review label. |
| Human evaluation | ISO 5060:2024 covers analytic evaluation of translation output, including human, post-edited, and raw MT. | Use configurable error types, severities, penalties, sampling, and reviewer qualification; keep release gating distinct from model scoring. |
| Legal translation | ISO 20771:2020 covers legal translation; MT plus post-editing is outside its scope. The catalogue shows review/withdrawal activity. | High-risk legal text requires a separately governed professional workflow. The general agent must not silently substitute MT/LLM output. |
| Segmentation/TM | SRX 2.0 (2008) and TMX 1.4b (2005) are preserved legacy OSCAR specifications. | Accept them through explicit legacy adapters. Pin segmentation rules. Prefer TMX Level 2 semantics when inline codes matter; reject lossy Level 1 round trips for protected markup. |
| Locale identifiers | BCP 47/RFC 5646; matching in RFC 4647. | Validate and canonicalize tags without treating fallback/matching as cultural authorization. Persist requested and resolved locale separately. |
| Unicode | Unicode 17.0.0; UAX #15 revision 57, UAX #29 revision 47, UAX #9, and CLDR/LDML 48.2 were current. | Pin Unicode/CLDR/ICU behavior. Use NFC where appropriate; do not apply compatibility normalization blindly. Test grapheme boundaries and bidirectional text. |
| Message syntax | MessageFormat 2 is a Unicode standard; current CLDR work includes its data model and bidi strategy. ICU runtime support must still be checked per implementation/version. | Parse, validate, and render with the actual target runtime. A standard’s existence does not prove every production runtime fully implements it. |

### Primary standards sources

- OASIS XLIFF Technical Committee and XLIFF 2.1: <https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=xliff> and <https://docs.oasis-open.org/xliff/xliff-core/v2.1/xliff-core-v2.1.html>
- ISO 21720:2024: <https://www.iso.org/standard/87344.html>
- W3C ITS 2.0: <https://www.w3.org/TR/its/>
- ISO 30042:2019 (TBX): <https://www.iso.org/standard/62510.html>
- ISO 11669:2024: <https://www.iso.org/standard/79089.html>
- ISO 17100:2015: <https://www.iso.org/standard/59149.html>
- ISO 18587:2017: <https://www.iso.org/standard/62970.html>
- ISO 5060:2024: <https://www.iso.org/standard/80701.html>
- ISO 20771:2020: <https://www.iso.org/standard/69032.html>
- TMX 1.4b and archived OSCAR standards: <https://www.ttt.org/oscarStandards/tmx/tmx14b.html> and <https://www.ttt.org/oscarStandards/>
- SRX 2.0: <https://www.unicode.org/uli/pas/srx/>

## Source representation and format findings

### XLIFF is a container, not a universal semantic guarantee

XLIFF separates source content from localized content and can carry inline codes, states, notes, and modules. It does not eliminate adapter decisions. A production adapter must publish:

- supported XLIFF versions, modules, namespaces, extensions, and state values;
- which inline codes it can round-trip;
- how it maps native resource identifiers and source revisions;
- whether notes are trusted guidance, untrusted context, or instructions from an authorized owner;
- how it preserves ordering, whitespace, and non-translatable content; and
- whether unknown content is preserved, quarantined, or rejected.

The source-native file and source-control revision remain the authority. An exported XLIFF package is a derived work package unless the project explicitly designates otherwise.

### Software-message formats require syntax-aware adapters

| Format/runtime | Evidence | Operational finding |
|---|---|---|
| ICU MessageFormat / MessageFormat 2 | ICU recommends translating whole messages and using plural/select constructs rather than concatenated fragments. | Parse to an AST, protect arguments, require locale-complete plural/select branches, and render with the shipping runtime. Do not validate with regex alone. |
| GNU gettext PO | PO supports contexts, plural forms, comments, and format flags. | Preserve `msgctxt`, extracted/reference comments, previous source, plural indices, and format directives. Context is part of identity. |
| Android resources | Android requires complete defaults and uses resource fallback; hardcoded strings escape localization. | Treat missing default resources as build defects. Test `en-XA` expansion and `ar-XB` RTL pseudolocales in developer builds. |
| Apple String Catalogs | Xcode discovers localizable strings, supports plurals and variations, and provides multiple pseudolanguages. | Build before export, preserve catalog state/variation metadata, and test expansion, bounded, tall, accented, and RTL scenarios. Generated/machine translations must retain their provenance state. |
| Project Fluent | Fluent gives localizers control over selectors and grammatical variation. | Use the reference parser and runtime; preserve attributes, terms, selectors, and placeables. A key/value-only transformation is unsafe. |

Authoritative sources:

- ICU messages and formatting: <https://unicode-org.github.io/icu/userguide/format_parse/messages/> and <https://unicode-org.github.io/icu/userguide/format_parse/>
- MessageFormat 2: <https://messageformat.unicode.org/> and <https://unicode-org.github.io/icu/userguide/format_parse/messages/mf2.html>
- GNU gettext manual: <https://www.gnu.org/software/gettext/manual/gettext.html>
- Android localization and pseudolocales: <https://developer.android.com/guide/topics/resources/localization> and <https://developer.android.com/guide/topics/resources/pseudolocales>
- Apple String Catalogs and pseudolanguages: <https://developer.apple.com/documentation/xcode/localizing-and-varying-text-with-a-string-catalog> and <https://developer.apple.com/documentation/xcode/preparing-your-interface-for-localization>
- Project Fluent: <https://github.com/projectfluent/fluent> and <https://firefox-source-docs.mozilla.org/l10n/fluent/index.html>

### Deterministic extraction precedes translation

Extraction should produce a typed segment contract, not a plain text list. At minimum it must contain:

- stable resource identity and source release;
- source text and canonical digest;
- format and AST or inline-code representation;
- protected tokens, placeholders, URLs, markup, and non-translatable spans;
- developer/localizer comments with author and trust class;
- surrounding message, screenshot, layout, and navigation references;
- character/line/visual constraints;
- content type, audience, domain, risk, and rights labels;
- target locale policy and fallback policy;
- dependency links to terms, claims, warnings, and related segments; and
- extraction and segmentation bundle versions.

Segment identity must not depend only on source text. Identical text can require different translations by UI location, gender, grammatical role, product, or domain. Conversely, a resource key is not sufficient if its source meaning changed. Use the tuple of tenant, project, source release, asset, resource identity, source digest, and variant identity.

## Locale, Unicode, and message-runtime findings

### Language, locale, market, and jurisdiction are different dimensions

BCP 47 describes language tags and RFC 4647 matching. A business localization profile also needs separate fields for market/country, jurisdiction, product, channel, audience, script, currency, measurement system, time zone policy, formality, reading level, and accessibility requirements. `fr` cannot safely stand for France, Canada, Belgium, a particular legal regime, and a particular campaign voice.

Store at least:

- `requested_language_tag`—what the release requested;
- `canonical_language_tag`—the validated/canonical form used internally;
- `resolved_runtime_locale`—the locale actually selected by the application/runtime;
- `market` and `jurisdictions`—explicit business/legal scope;
- `fallback_chain`—an approved ordered list; and
- `fallback_reason`—why fallback is permitted for this artifact.

Locale inheritance and likely-subtag expansion are useful technical heuristics, not proof that content is culturally or legally acceptable. Persist requested and actual locale because silent fallback can create a false appearance of coverage.

Sources: BCP 47 <https://www.rfc-editor.org/info/rfc5646/>, RFC 4647 <https://www.rfc-editor.org/info/rfc4647/>, W3C language-tag guidance <https://www.w3.org/International/articles/language-tags/index.en>, ICU locale guidance <https://unicode-org.github.io/icu/userguide/locale/>, and CLDR/LDML <https://www.unicode.org/reports/tr35/>.

### Unicode operations are versioned behavior

- Normalize for a stated interoperability goal, normally NFC for text exchange. UAX #15 warns that compatibility normalization changes distinctions; NFKC/NFKD must not be a blanket cleanup step.
- Count user-visible characters by grapheme cluster where a UI constraint is visual; bytes and code points remain relevant for storage/protocol limits.
- Apply UAX #9 and platform bidi behavior. Bidirectional isolation, neutral punctuation, numbers, placeholders, and embedded code need explicit tests.
- Pin CLDR plural rules and locale data. A rule change can alter required message branches or rendered output without source-text changes.
- Format numbers, dates, currencies, units, and lists with locale-aware libraries. Never ask a generative model to reproduce canonical financial or temporal values when a deterministic formatter can do it.

Sources: Unicode 17.0 <https://www.unicode.org/versions/Unicode17.0.0/>, UAX #15 <https://www.unicode.org/reports/tr15/>, UAX #29 <https://unicode.org/reports/tr29/>, UAX #9 <https://www.unicode.org/reports/tr9/>, CLDR 48 <https://cldr.unicode.org/downloads/cldr-48>, and plural rules <https://cldr.unicode.org/index/cldr-spec/plural-rules>.

## Terminology and translation-memory findings

### The termbase is governed domain knowledge

A term entry should be concept-oriented and include definition, domain, language/locale variants, preferred/admitted/deprecated/forbidden status, part of speech where useful, usage examples, grammatical properties, owner, approver, provenance, effective interval, sensitivity, and version. A “do not translate” rule is a first-class target-language decision, not a prompt hint.

At runtime:

1. retrieve by concept/domain/product/locale, not raw substring alone;
2. identify overlapping or contradictory terms;
3. apply exact protected forms before generation where required;
4. expose the applicable subset with definitions and status;
5. validate morphological and exact-form policies separately; and
6. route unresolved conflicts to the terminology owner.

IATE demonstrates why a living, versioned terminology service differs from a static download: its search/API can be more current than a periodic dataset, and exported TBX needs version/provenance metadata. Source: <https://data.europa.eu/data/datasets/iate?locale=en>.

### Translation memory is retrieval evidence, not authority

A TM unit requires source and target text/AST, language/locale pair, segmentation/version, source and target digests, domain, product, channel, source release, reviewer status, acceptance/correction history, license/rights, provenance, sensitivity, effective interval, and invalidation state. Eligibility must be a deterministic policy.

Risks that invalidate naive fuzzy reuse include:

- identical text with a different referent or grammatical role;
- a changed product claim or legal warning;
- stale terminology;
- a locale-only match used for a market-specific artifact;
- an imported corpus with unknown original source language or automated alignment;
- inline-code loss during interchange;
- reviewer corrections that never reached the TM; and
- malicious or low-quality memory poisoning.

The DGT Translation Memory is useful production evidence: releases have encoding/interchange particulars, alignment provenance varies, and English extraction may be used as a pivot even when the original source language is unknown. DGT-Acquis separately preserves document context, highlighting what isolated TM units lose. Sources: <https://joint-research-centre.ec.europa.eu/language-technology-resources/dgt-translation-memory_en> and <https://joint-research-centre.ec.europa.eu/language-technology-resources/dgt-acquis_en>.

## Machine-translation and LLM adapter findings

### Use a provider capability registry, not provider-name conditionals

Each immutable adapter release should record:

| Capability class | Required fields |
|---|---|
| Language coverage | Supported source/target pairs; script/locale limitations; auto-detection rules; language-code mapping |
| Input contract | Character/byte/document limits; batch limits; supported MIME/file formats; tag/markup behavior; normalization |
| Controls | Glossary pairs; terminology behavior; context; formality; brevity; profanity; do-not-translate; custom models |
| Execution | Sync/async semantics; polling/callback behavior; timeout; cancel; idempotency; retryable errors; rate quotas |
| Data governance | Processing regions; storage/retention; training-use terms; encryption; tenant controls; subprocessors; DPA status |
| Economics | Billable unit, minimums, per-target multiplication, cache rules, and budget attribution |
| Quality evidence | Last qualified evaluation by language × locale × domain × content type × risk tier |

Official provider evidence shows why a generic “translate” interface is insufficient:

- Google Cloud Translation Advanced exposes glossaries, IAM, regional endpoints, batch/document operations, and distinct quotas/pricing; regional compatibility must be checked. <https://docs.cloud.google.com/translate/docs/api-overview>, <https://docs.cloud.google.com/translate/docs/advanced/endpoints>, <https://docs.cloud.google.com/translate/quotas>, <https://cloud.google.com/products/translate/pricing>
- DeepL supports a context parameter and tag/glossary controls, but request size, independent text semantics, and glossary language pairs constrain batching. <https://developers.deepl.com/docs/learning-how-tos/examples-and-guides/how-to-use-context-parameter> and <https://github.com/DeepL/openapi/blob/main/openapi.json>
- Azure Translator exposes profanity options, alignment, and sentence-length data; cultural behavior cannot be reduced to a single universal profanity setting. <https://learn.microsoft.com/en-us/rest/api/translator/translator/translate?view=rest-translator-v3.0>
- Amazon Translate exposes custom terminology, parallel data, do-not-translate, formality, brevity, and profanity features with feature-specific limitations. <https://docs.aws.amazon.com/translate/latest/dg/customizing-translations.html>
- The European Commission’s eTranslation has eligibility/security constraints, and the Commission explicitly warns that machine-translated output is not suitable as an authoritative rendering of EU legislation. <https://commission.europa.eu/resources-partners/etranslation_en> and <https://commission.europa.eu/languages-our-websites/use-machine-translation-europa_en>

### A general LLM is a constrained candidate generator

For an LLM adapter:

- pin an exact model snapshot or provider deployment;
- use a strict structured-output schema;
- disable tool use in the translation call;
- pass source and retrieved context as delimited untrusted data;
- set an explicit storage/retention mode;
- reject truncation rather than accepting partial output;
- record prompt, model, schema, tokenizer/provider, safety-policy, and adapter versions;
- validate the output outside the model; and
- never let the model approve, commit, or publish its own candidate.

The OpenAI Responses API is one example of an API that supports structured response formats and an explicit `store` setting; its existence does not make a model a certified translator or settle tenant data-residency obligations. The adapter must separately verify the organization’s current retention and regional controls. Source: <https://developers.openai.com/api/reference/cli/resources/responses/methods/create>.

## Connector and external-effect findings

### TMS APIs do not share one delivery model

| System | Officially documented behavior with architectural impact |
|---|---|
| Phrase Strings | US/EU endpoints, required client identification, webhooks with HMAC signatures, and finite webhook history. Some bulk/upload or repository-sync operations do not produce the same per-key events. |
| Lokalise | Webhooks can use origin IP/custom headers, but do not provide an intrinsic cryptographic signature in the documented flow; retries can be delayed/unordered and failing hooks may be disabled. |
| Transifex | Upload/download operations may be asynchronous: `202`, polling, redirects, and callbacks are part of the effect contract. Source updates can retain translations by default, which may preserve stale targets unless invalidation policy is explicit. |
| Crowdin | Current documentation and plan capabilities must be resolved during adapter certification. Do not infer idempotency, signing, or transactional guarantees from other TMS products. |

Sources: Phrase API <https://developers.phrase.com/en/api/strings/getting-started> and webhooks <https://support.phrase.com/hc/en-us/articles/5784125630620-Webhooks-Strings>; Lokalise API/webhooks <https://developers.lokalise.com/reference/lokalise-rest-api> and <https://developers.lokalise.com/docs/webhooks-guide>; Transifex OpenAPI <https://transifex.github.io/openapi/> and source-update semantics <https://help.transifex.com/en/articles/6236849-updating-your-source-content>; Crowdin documentation <https://support.crowdin.com/>.

### Repository and CMS effects require optimistic concurrency and readback

- GitHub recommends webhook-driven queues, conditional requests, and rate-aware serial processing. Updating repository content requires the current content SHA, and content writes/deletes can conflict. Failed webhook deliveries are not automatically redelivered. Validate delivery signatures and implement replay/reconciliation. Sources: <https://docs.github.com/en/rest/using-the-rest-api/best-practices-for-using-the-rest-api>, <https://docs.github.com/en/rest/repos/contents>, <https://docs.github.com/en/webhooks/using-webhooks/validating-webhook-deliveries>, and <https://docs.github.com/en/webhooks/using-webhooks/handling-failed-webhook-deliveries>.
- Contentful locale fallback can mask missing locale data. Locale-management writes are versioned and deletion is destructive. Locale-based publishing changes the release unit, so an adapter must know whether it is updating an entry, locale, environment, or release. Sources: <https://www.contentful.com/developers/docs/references/content-delivery-api/localization/>, <https://www.contentful.com/developers/docs/references/content-management-api/locales/>, and <https://www.contentful.com/developers/docs/references/graphql/locale-handling/>.

The portable effect protocol is therefore:

1. calculate a deterministic operation key and intended artifact digest;
2. resolve current remote revision and preconditions;
3. record `planned`, then require the applicable approval;
4. record `dispatching` before the external call;
5. use the provider’s idempotency or optimistic-concurrency feature if available;
6. on timeout or ambiguous response, record `unknown`, never “failed” by assumption;
7. read back by stable identity and compare revision/content digest;
8. mark `verified` or `proved_not_committed`; and
9. correct forward or retry only from a proven state.

Durable workflow engines can preserve state and retry Activities, but they cannot manufacture exactly-once semantics in an external system. Temporal’s Activity heartbeat/cancellation/retry model is useful evidence for separating workflow state from external effect handling. Sources: <https://docs.temporal.io/> and <https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/activities/activity-execution.mdx>.

## Human workflow and transcreation findings

### Review roles must express accountability, not merely queue position

The project specification should assign, by content/risk tier:

- source owner;
- localization project owner;
- translator or post-editor;
- independent linguistic reviewer;
- terminology owner;
- in-market cultural reviewer;
- product/technical reviewer;
- accessibility reviewer;
- legal/regulatory approver;
- marketing/brand approver; and
- release/publish authority.

One person may hold several low-risk roles, but self-approval is prohibited where the policy requires separation of duties. Machine provenance remains visible to every reviewer.

### Transcreation is a constrained creative decision process

Marketing transcreation starts from a brief, not just a source sentence. The brief contains campaign objective, audience, emotional response, brand voice, channel, visual context, fixed product names, prohibited implications, substantiated claims, legal copy, character/timing constraints, and success measures. The workflow should produce multiple rationale-bearing options when useful, not pretend there is a single literal equivalent.

The agent can propose and check options. Marketing owns campaign intent and claims; legal owns legal acceptability; qualified in-market reviewers own linguistic/cultural judgments within their remit; the release owner owns publication. Legal disclaimers and substantiated claims remain separately addressable protected units, even when surrounding creative copy changes.

## Accessibility and inclusive localization findings

WCAG 2.2 requires correct language metadata for pages and passages, text alternatives, labels/instructions, programmatic names/roles, reflow, and other behaviors that localization can break. An accessibility pass must inspect the localized experience, not only source compliance.

Required localized checks include:

- page/document and language-of-parts metadata;
- accessible names, descriptions, alt text, captions, and transcripts;
- visible label versus accessible-name consistency;
- reading order and bidi behavior;
- text expansion, zoom/reflow, clipping, truncation, and vertical-script/tall-glyph behavior;
- localized keyboard/access-key conflicts where applicable;
- examples in instructions (dates, names, addresses, phone numbers) that make sense in the target context;
- screen-reader pronunciation and language switching;
- cognitive clarity and reading level; and
- authenticity of any “authorized translation” claim.

Pseudolocalization is an early engineering test, not linguistic QA. Android’s `en-XA` and `ar-XB` and Apple’s expansion, RTL, bounded, tall, and accented pseudolanguages expose classes of layout/i18n defects before paid translation, but real target-locale and assistive-technology testing remains necessary.

Sources: WCAG 2.2 <https://www.w3.org/TR/WCAG22/>, Language of Parts <https://www.w3.org/WAI/WCAG22/Understanding/language-of-parts>, W3C translation status <https://www.w3.org/WAI/standards-guidelines/wcag/translations/>, clear content <https://www.w3.org/WAI/WCAG2/supplemental/objectives/o3-clear-content/>, HTML language declarations <https://www.w3.org/International/questions/qa-html-language-declarations.html>, Android pseudolocales <https://developer.android.com/guide/topics/resources/pseudolocales>, and Apple pseudolanguages <https://developer.apple.com/documentation/xcode/preparing-your-interface-for-localization>.

## Evaluation findings

### Use a layered evidence stack

| Layer | Purpose | Examples | Can it approve release alone? |
|---|---|---|---|
| Contract validation | Detect format and invariant violations | parse/render round trip, placeholder multiset, tag balance, plural/select completeness, encoding, forbidden characters | No, but failure can block automatically |
| Policy validation | Enforce explicit project decisions | mandatory/prohibited terms, protected claims, style-lint rules, locale metadata, source freshness | No, but declared hard failures can block |
| Reference/metric signals | Rank, trend, and sample | exact/TM match, BLEU-like corpus metrics, learned metrics, round-trip signals | No |
| Analytic human evaluation | Assess actual translation defects | configurable MQM/ISO 5060-style types, severity, penalties, comments | Sometimes, under the declared sampling/gate policy |
| In-context review | Validate product/cultural/accessibility behavior | UI screenshots, rendered documents, help navigation, campaign assets, assistive technology | Required where context/risk demands it |
| Outcome monitoring | Detect post-release harm or underperformance | correction rate, support signals, conversion guardrails, accessibility issues, wrong-locale reports | Governs continuation/rollback, not initial linguistic truth |

MQM’s current top-level dimensions include terminology, accuracy, linguistic conventions, style, locale conventions, audience appropriateness, and design/markup. The taxonomy should be tailored; an exhaustive taxonomy imposed on every workflow creates inconsistent annotation and reviewer fatigue. Sources: <https://www.themqm.org/mqm-pillars/typology/> and <https://www.themqm.org/resources/downloads/>.

Automated MT metrics are valuable but brittle. WMT/ACES work documents failures on phenomena such as omissions, hallucinations, and untranslated content. Round-trip translation has shown usefulness as one feature in some modern settings despite historically weak correlations, so it may contribute to triage but must never become segment truth or approval. Sources: WMT 2024 <https://aclanthology.org/events/wmt-2024/>, ACES <https://direct.mit.edu/coli/article/51/1/73/124465/Machine-Translation-Meta-Evaluation-through>, current challenge-set direction <https://www2.statmt.org/wmt26/mteval-task.html>, round-trip study <https://aclanthology.org/2023.findings-acl.22/>, and original BLEU corpus framing <https://aclanthology.org/P02-1040/>.

### Evaluation must be sliced, not averaged away

Report quality and operations by:

- source and target language;
- target locale/market/script;
- domain and content type;
- product/channel;
- new translation versus TM leverage versus post-edit;
- provider/model/behavior bundle;
- risk tier;
- reviewer cohort and workflow;
- text length and message construct; and
- high-impact phenomena such as gender, names, negation, numerals, placeholders, toxicity/profanity, and low-resource pairs.

An overall average must not authorize a language pair that lacks qualified evidence. Gender and low-resource benchmarks show why aggregate system quality does not guarantee individual-language or phenomenon performance. Sources: MT-GenEval <https://aclanthology.org/2022.emnlp-main.288/>, FLORES <https://github.com/facebookresearch/flores>, and NLLB <https://ai.meta.com/research/no-language-left-behind/>.

### Candidate SLOs are policy inputs, not universal constants

The guide uses these as examples to tailor by risk:

- protected-token and message-AST integrity: 100%;
- stale-source publication: zero tolerated;
- wrong-locale publication: zero tolerated;
- unverified external effects: bounded age with an alert and reconciliation target;
- release parity: exact required-locale set or an explicit waiver, never silent fallback;
- critical human-evaluation errors: zero in sampled high-risk content;
- review latency and queue age: defined per content/risk/locale;
- correction escape rate and post-release time-to-correct: tracked by locale; and
- human capacity saturation, provider quota, cost, and retry amplification: operational guardrails.

## Security, privacy, and rights findings

### Every content-bearing input is untrusted data

Source strings, developer comments, screenshots, OCR, TM units, term entries, provider output, TMS comments, and reviewer attachments can contain prompt-injection instructions or malicious markup. The design must separate:

- **data plane:** parsing, retrieval, candidate generation, and validation with no ambient effects; and
- **effect plane:** narrowly scoped repository/TMS/CMS operations authorized by typed workflow state and approval.

No text inside a translatable asset can grant permission, change the target locale, reveal secrets, select a tool, waive review, or publish content. Use structured schemas, allowlisted tools, least-privilege credentials, human approval, output validation, and complete effect audit. Sources: OWASP Prompt Injection Prevention <https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html>, OWASP AI Agent Security <https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html>, and NIST AI RMF Generative AI Profile <https://www.nist.gov/itl/ai-risk-management-framework>.

### Privacy controls follow every copy

Localization routinely exposes personal or confidential content to providers and reviewers. A data inventory must cover source artifacts, prompts, provider logs, caches, TM, termbase, screenshots, review comments, evaluation datasets, traces, backups, and continuity receipts. Apply purpose limitation, minimization, access control, processor terms, transfer controls, retention, erasure/correction, incident handling, and regional routing to each copy.

Observability should store IDs, sizes, hashes, classes, timing, and outcomes by default—not raw source/target content. A privileged, time-limited forensic path can capture content only under an explicit policy. Deletion/correction must cascade to derived memory, caches, evaluation corpora, and replay fixtures unless a documented legal basis requires preservation.

Primary legal source: GDPR consolidated text <https://eur-lex.europa.eu/eli/reg/2016/679/oj>. This packet is engineering guidance, not legal advice.

### Translation rights are not implied by possession

The Berne Convention includes an author’s exclusive right of translation. Before sending content to a provider or reviewer, record the rights basis, permitted purpose/territories, confidentiality, derivative-work permissions, dataset/training restrictions, retention, and expiry. Do not mine corrected translations into long-term memory or evaluation sets unless the rights and privacy labels permit it. Sources: WIPO Berne summary <https://www.wipo.int/treaties/en/ip/berne/summary_berne.html> and WIPO guide <https://www.wipo.int/edocs/pubdocs/en/copyright/615/wipo_pub_615>.

## State, context, memory, and continuity findings

### Authoritative state must remain typed and external to prompts

The durable task record holds source revision, locale, state, owners, bundle versions, artifact digests, approvals, effects, and deadlines. The model receives a minimum compiled view. It cannot mutate authority by narrating a new state.

Use distinct memory classes:

| Memory class | Example | Authority and retention rule |
|---|---|---|
| Turn/scratch | Current candidate and parser diagnostics | Ephemeral; never authoritative |
| Run working memory | Current batch plan, open conflicts, review routing | Scoped to run; rebuilt from durable state |
| Reviewer session | Display state, temporary filters, draft comment | Session-scoped; approval is a separate signed record |
| Durable workflow/task | State, source revision, approvals, effects, artifact digests | Authoritative; transactional and auditable |
| Domain knowledge | Termbase, style, locale, claims, format rules | Versioned, owner-approved, provenance/rights-aware |
| Long-term preference | Reviewer display or tone preference | Narrowly scoped; cannot override facts, terms, or policy |
| Episodic/outcome | Accepted/rejected/corrected candidate with rationale | Admitted only after sanitization, rights check, provenance, quality gate, and expiry assignment |

All durable knowledge requires tenant/project/locale scope, provenance, owner, rights, sensitivity, effective interval, freshness, correction history, retention, deletion behavior, and poisoning controls.

### Continuity receipts are loss-aware indexes

A pause/resume or compaction receipt should contain:

- task, case, release, run, tenant, and project IDs;
- source revision/digest and target locale/product/channel;
- exact workflow state and state version;
- completed and open segment IDs;
- outstanding approvals and deadlines;
- effect-ledger entries, especially `dispatching` or `unknown` outcomes;
- staged artifact URIs and digests;
- protected-token/AST validation digest;
- termbase, style, TM, locale, segmentation, prompt, model, provider, validator, and adapter bundle versions;
- rights, sensitivity, residency, and retention labels;
- evidence pointers and freshness timestamps;
- explicitly dropped fields with reason;
- resume preconditions and next allowed transitions; and
- receipt schema version, checksum/signature, creator, and time.

The receipt is an index back to authoritative records. On resume, re-read current source and task state, resolve unknown effects, check policy freshness, and compare artifact digests before continuing. A prose summary alone is insufficient.

## Observability and event findings

Use correlated traces, structured events, metrics, and an immutable audit trail. Core identifiers are tenant, project, source release, asset, segment, task, locale, run, behavior bundle, artifact, approval, and effect. Redact content by default.

Recommended event envelope fields align with CloudEvents concepts: stable event ID, source, type, subject, time, data schema/version, tenant/project, correlation and causation IDs, classification, and payload digest. The CloudEvents repository identifies 1.0.2 as a stable release; do not accidentally implement against a work-in-progress branch label. Source: <https://github.com/cloudevents/spec>.

OpenTelemetry semantic conventions were at 1.44.0 in the researched corpus, while dedicated GenAI conventions were still marked Development. Pin an internal mapping and version it rather than allowing experimental attribute names to become permanent storage schema. Sources: <https://opentelemetry.io/docs/specs/semconv/> and <https://github.com/open-telemetry/semantic-conventions-genai>.

Operational metrics must reveal work and harm, not only model latency:

- intake, queue age, throughput, and completion by locale/risk;
- source churn and invalidation;
- TM leverage and later correction by match band;
- term conflicts/compliance;
- deterministic validator failures;
- human error types, severity, disagreement, and correction;
- provider latency, quota, cost, and retry amplification;
- effect unknown age and reconciliation outcome;
- required-locale parity and fallback use;
- accessibility and rendering defects;
- post-release corrections/complaints; and
- reviewer capacity and assignment fairness.

## Material disagreements and their resolution

| Question | Evidence tension | Playbook resolution |
|---|---|---|
| Is XLIFF 2.1 “the ISO standard”? | OASIS latest is 2.1; ISO 21720:2024 specifies 2.0. | Name both precisely and test the actual version/modules. |
| Does a standard guarantee runtime support? | MessageFormat 2 is standardized; specific ICU APIs may still be technical-preview or version-limited. | Certify the actual parser/renderer and pin its version. |
| Can automated metrics gate releases? | Metrics correlate at aggregate levels but fail particular phenomena and domains. | Use them for triage/trending; require deterministic invariants and risk-based human/in-context gates. |
| Is round-trip translation useless? | Older criticism and newer evidence differ. | Treat it as an optional weak signal, never truth or approval. |
| Does human post-editing imply ISO 17100? | ISO 17100 excludes raw MT+post-edit; ISO 18587 addresses full post-editing separately. | Declare the actual workflow and never infer certification. |
| Should all source changes invalidate all translations? | Global invalidation is safe but wasteful; narrow invalidation risks stale context. | Use a dependency/impact graph and invalidate the changed unit plus meaning, terminology, reference, layout, and claim dependents. |
| Should memory learn every human correction? | Learning improves leverage but can amplify private, licensed, erroneous, or locale-specific content. | Admission is a governed write with quality, rights, privacy, scope, and holdout checks. |
| Is MT acceptable for legal text? | Tools can translate it, while official/legal workflows limit authority and some standards exclude PEMT. | Use a separately governed professional legal workflow; machine output can be a controlled aid, never the authoritative approval. |
| Is a timeout a failed write? | Distributed APIs can commit before the response is lost. | Record `unknown`, reconcile, then retry only when non-commitment is proven or idempotency is strong. |

## Derived production invariants

1. Every localized artifact points to an immutable source release and source digest.
2. Every provider, prompt, termbase, TM snapshot, locale rule, segmentation rule, validator, and adapter is part of a versioned behavior bundle.
3. Protected tokens and native message syntax round-trip exactly according to the format contract.
4. A source revision change makes older tasks stale until impact analysis proves which outputs remain valid.
5. Requested locale, resolved runtime locale, market, and jurisdiction are stored separately.
6. Machine/TM provenance is never erased by review or export.
7. Automated scores do not approve meaning, cultural appropriateness, legal claims, or publication.
8. No content-bearing input can grant effects or change workflow policy.
9. The approved artifact digest must equal the staged and committed digest.
10. External effects are complete only after readback verification.
11. Unknown effect outcomes block conflicting writes and publication.
12. Long-term memory admission requires provenance, rights, scope, quality, retention, and poisoning checks.
13. Required-locale parity is release-specific and explicit; silent fallback is not completion.
14. High-risk content uses named qualified human roles and separation of duties.
15. Logs and traces are content-minimized by default.
16. Resume starts from authoritative state and reconciliation, not a narrative summary.

## Research-derived staged delivery

| Stage | Capability | Exit evidence |
|---|---|---|
| 0 — deterministic baseline | Inventory formats/locales/owners; source extraction; pseudolocalization; human CAT/TMS workflow | Round-trip fixtures pass; source and locale policy are explicit; qualified reviewer workflow exists |
| 1 — read-only drafts | Candidate generation in a sealed data plane; no external writes | Offline golden/holdout evaluation; exact provenance; deterministic rejection; bounded cost/time |
| 2 — supervised MVP | One low-risk content type, source, and locale; staged human approval | No stale-source or invariant escapes in pilot; reviewer acceptance/correction evidence; rollback rehearsed |
| 3 — reliable effects | Repository/TMS/CMS adapters, effect ledger, optimistic concurrency, reconciliation | Timeout/duplicate/conflict/webhook-loss drills pass; approved/staged/committed digests match |
| 4 — production governance | RBAC, tenant isolation, privacy/rights controls, SLOs, audit, incidents, accessibility and risk gates | Threat/privacy review, incident drills, deletion/correction tests, sliced quality sign-off |
| 5 — scale and resilience | Regional/tenant cells, queue fairness, human capacity, quotas, offline/DR modes | Load and recovery tests by locale; no starvation; RTO/RPO and catch-up load verified |
| 6 — controlled evolution | Immutable behavior bundles, shadow/canary, drift, failure mining, exact rollback | Bundle comparison and rollback drills; rights-safe learning; holdouts protected; regressions bounded |

## Refresh triggers

Re-run targeted research when any of these changes:

- XLIFF/ITS/TBX/ISO catalogue status or licensed standard edition;
- Unicode, CLDR, ICU, MessageFormat, platform resource format, or runtime version;
- TMS/CMS/repository API version, webhook model, or plan;
- provider model, endpoint, region, retention, glossary, price, quota, or language support;
- privacy/transfer/copyright law or organizational policy;
- target language, market, domain, risk tier, or accessibility standard;
- evaluation taxonomy, benchmark, reviewer population, or release threshold; or
- workflow engine, event schema, telemetry conventions, or effect adapter.

## Primary-source index

### Localization standards and processes

- <https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=xliff>
- <https://docs.oasis-open.org/xliff/xliff-core/v2.1/xliff-core-v2.1.html>
- <https://www.iso.org/standard/87344.html>
- <https://www.w3.org/TR/its/>
- <https://www.iso.org/standard/62510.html>
- <https://www.iso.org/standard/79089.html>
- <https://www.iso.org/standard/59149.html>
- <https://www.iso.org/standard/62970.html>
- <https://www.iso.org/standard/80701.html>
- <https://www.iso.org/standard/69032.html>
- <https://www.ttt.org/oscarStandards/tmx/tmx14b.html>
- <https://www.unicode.org/uli/pas/srx/>

### Locale, Unicode, formats, and accessibility

- <https://www.rfc-editor.org/info/rfc5646/>
- <https://www.rfc-editor.org/info/rfc4647/>
- <https://www.unicode.org/versions/Unicode17.0.0/>
- <https://www.unicode.org/reports/tr15/>
- <https://unicode.org/reports/tr29/>
- <https://www.unicode.org/reports/tr9/>
- <https://www.unicode.org/reports/tr35/>
- <https://messageformat.unicode.org/>
- <https://www.gnu.org/software/gettext/manual/gettext.html>
- <https://developer.android.com/guide/topics/resources/localization>
- <https://developer.android.com/guide/topics/resources/pseudolocales>
- <https://developer.apple.com/documentation/xcode/localizing-and-varying-text-with-a-string-catalog>
- <https://developer.apple.com/documentation/xcode/preparing-your-interface-for-localization>
- <https://github.com/projectfluent/fluent>
- <https://www.w3.org/TR/WCAG22/>

### Operations, providers, security, and evaluation

- <https://developers.phrase.com/en/api/strings/getting-started>
- <https://developers.lokalise.com/docs/webhooks-guide>
- <https://transifex.github.io/openapi/>
- <https://docs.github.com/en/rest/using-the-rest-api/best-practices-for-using-the-rest-api>
- <https://www.contentful.com/developers/docs/references/content-delivery-api/localization/>
- <https://docs.cloud.google.com/translate/docs/api-overview>
- <https://developers.deepl.com/docs/learning-how-tos/examples-and-guides/how-to-use-context-parameter>
- <https://learn.microsoft.com/en-us/rest/api/translator/translator/translate?view=rest-translator-v3.0>
- <https://docs.aws.amazon.com/translate/latest/dg/customizing-translations.html>
- <https://www.themqm.org/mqm-pillars/typology/>
- <https://aclanthology.org/events/wmt-2024/>
- <https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html>
- <https://eur-lex.europa.eu/eli/reg/2016/679/oj>
- <https://www.wipo.int/treaties/en/ip/berne/summary_berne.html>
- <https://opentelemetry.io/docs/specs/semconv/>
- <https://github.com/cloudevents/spec>
- <https://docs.temporal.io/>

## Blueprint conclusion

The smallest defensible localization agent is a supervised, single-content-type workflow with deterministic extraction and validation, a versioned candidate generator, a qualified reviewer, an immutable approval, and a reconciled staged write. Every expansion—more locales, transcreation, self-service intake, automated effects, long-term learning, or higher-risk content—must pass a separate evidence gate.

The production objective is not maximum translation volume. It is a provable chain from approved source intent to the correct localized experience, with known ownership, bounded autonomy, reversible effects, and evidence that survives retries, source changes, provider changes, compaction, incidents, and audit.

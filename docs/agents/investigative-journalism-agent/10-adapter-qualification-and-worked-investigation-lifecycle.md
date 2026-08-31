# Adapter Qualification and Worked Investigation Lifecycle

## What this guide closes

This guide turns the architecture into an integration and operating proof. It answers three questions before a newsroom connects a live provider:

1. Is an agent needed, or would ordinary reporting and deterministic software be safer?
2. What does this adapter actually prove, retain, omit, bill, and change?
3. Can one source-safe matter survive intake, verification, review, publication ambiguity, and correction without losing identity or custody?

It does not recommend providers universally. Availability, contract terms, data location, retention, redistribution rights, quotas, and fields are deployment-specific. The examples below were checked against primary sources on **2026-08-31** and require a release-time recheck.

## The first gate: no agent, deterministic system, or bounded agent

| Workload | Best starting point | Why | Evidence required before adding a model |
|---|---|---|---|
| One reporter researching one narrow story | Ordinary reporting plus a records/evidence workspace | Human judgment and source relationships dominate; orchestration adds risk | A measured, repeated bottleneck that search, templates, or indexing do not solve |
| Known records form, feed poll, hash, metadata parse, archive capture, OCR, timeline sort, or claim-ledger check | Deterministic job | The transformation and pass/fail rule are known and reproducible | Ambiguity remains after improving the deterministic baseline |
| High-risk source contact, ground rules, identity handling, coercion assessment, or emergency safety planning | Trained humans with approved secure practices | An agent cannot promise confidentiality or understand the full threat and legal context | None; these accountable acts remain human-owned even in a mature system |
| Variable public-source research across a bounded catalog | Bounded advisory loop | A model may choose a useful next query or synthesize competing explanations | Offline evidence that it improves contradiction recall or reviewer time without new safety failures |
| Publication, correction, retraction, legal, fairness, or public-interest decision | Human editorial workflow with deterministic package and receipt support | Accountability cannot be delegated to a score or tool call | None; an agent may prepare evidence but never own the decision |

Start at [Stage 0](09-zero-to-production-roadmap-and-reference-contracts.md#stage-0--qualify-the-problem). A production-quality deterministic workspace is a successful outcome, not a failed agent project.

## Qualification is a proof obligation

An adapter is not “qualified” because a demo returned data. Admission requires a signed capability manifest, contract tests against a controlled account, rights review, threat review, failure tests, an owner, and an expiry.

```yaml
journalism_capability_manifest:
  schema_version: journalism-adapter/2
  manifest_id: adapter_foia_portal_usdoj_7
  release: 7.2.1
  provider:
    name: FOIA.gov
    surface: agency_component_directory
    api_or_spec: public API checked 2026-08-31
    deployment_account: newsroom-public-records
  capability:
    operation: agency_component.search
    effect_class: read
    result_semantics: directory_metadata_not_request_status
    finality: none
    pagination: provider_cursor_mapped_to_local_page_receipt
    freshness: provider_updated_at_when_present
    completeness: bounded_to_returned_pages_and_provider_scope
  authority:
    allowed_tenants: [newsroom_7]
    allowed_compartments: [public_records]
    allowed_purposes: [custodian_discovery]
    prohibited_data: [source_identity, confidential_submission, privileged_comment]
  rights_and_data:
    acquisition_basis_ref: rights-public-records/7
    retention_profile: public-records-metadata/3
    redistribution: local_review_required
    deletion_and_hold_semantics: local_store_controls_derivatives_only
    provider_training_or_secondary_use: verify_before_release
  security:
    authentication: provider_api_key_via_broker
    egress_hosts: [api.foia.gov]
    credential_visible_to_model: false
    response_trust: untrusted_data
  operations:
    timeout_ms: 10000
    retry_class: read_with_jitter_and_deadline
    rate_and_quota_profile: foia-gov/2026-08-31
    circuit_breaker: public-records-provider
  evidence:
    conformance_report: report://adapter_foia_portal_usdoj_7/2026-08-31
    contract_and_terms_snapshot: snapshot://provider/foia-gov/2026-08-31
    owner: records-platform
    approved_by: [records_editor, security, rights_owner]
    expires_at: 2026-11-30T00:00:00Z
```

The runtime injects tenant, matter, compartment, principal, purpose, and credential. The model cannot supply or override them.

## Mandatory adapter tests

| Test family | Test | Passing evidence |
|---|---|---|
| Identity | Same provider ID across two revisions; same bytes from two acquisitions; same name for two people | Provider identity, acquisition identity, object identity, and person/entity identity remain separate |
| Authorization | Wrong matter, tenant, compartment, purpose, or role | Denial occurs before cache, search, dedupe, or provider access; no existence leak |
| Population | Zero, one, last, and beyond-last pages; partial page; attachments omitted by default | Coverage receipt states pages, filters, totals if trustworthy, truncation, omitted classes, and retryability |
| Time | Provider edit, withdrawal, delayed publication, clock skew, missing timezone | Original and revised observations survive with valid and transaction time; no overwrite |
| Rights | License expires, article rights differ from metadata rights, retention changes, redistribution prohibited | Ingestion/export is blocked or narrowed and existing derivatives are located for policy action |
| Content safety | HTML injection, PDF bomb, malformed media, tracking URL, formula payload, poisoned transcript | Quarantine and typed parsing contain the payload; no capability or egress change |
| Finality | API acknowledges a request, portal accepts a comment, CMS times out after commit | Local state remains `pending` or `unknown`; only independent observation confirms effect |
| Recovery | Crash before/after provider call, duplicate callback, stale worker, revoked credential | Stable operation ID, fencing, reconciliation, no blind duplicate, clear terminal state |
| Deletion/hold | Delete one evidence version under a legal hold; expire a derived transcript | Policy resolves the conflict; tombstones invalidate indexes, contexts, packages, and future work without destroying held data |
| Drift | Field disappears, status enum changes, ordering changes, limit tightens | Contract canary fails closed; affected manifests expire; no silent semantic mapping |

Do not qualify only the happy path. Store the conformance response fixtures and replay them when the provider, adapter, schema, policy, model, or deployment account changes.

## Canonical identities and bitemporal rules

Never use a title, URL, filename, email address, provider ID, or content hash as a universal primary key.

| Object | Stable local identity | Version or event identity | Critical distinction |
|---|---|---|---|
| Matter | `matter_id` | `matter_revision_id` | Approved scope and authority container; not a story |
| Story | `story_id` | `story_revision_id` | Editorial work product; a matter may yield zero or many stories |
| Source | blind `source_ref` | `source_statement_id` / ground-rule revision | Blind reference is not person identity; mapping stays in the vault |
| Person | `person_candidate_id` | `person_resolution_version` | A candidate is not a verified legal identity |
| Entity | `entity_candidate_id` | `entity_resolution_version` | Organization/product/place/legal entity identity is type-specific |
| Document | `document_id` | `document_representation_id` | Logical document differs from downloaded bytes, scan, OCR, or revision |
| Media | `media_id` | `media_representation_id` | Depicted event differs from file/container/encode identity |
| Claim | `claim_id` | `claim_version_id` | Proposition identity differs from wording and epistemic state |
| Evidence item | `evidence_id` | `acquisition_id` / `derivation_id` | Equal bytes do not imply equal custody, rights, origin, or matter access |
| Custody | `custody_chain_id` | append-only `custody_event_id` | A hash is one integrity check, not the custody chain |
| Timeline | `timeline_id` | `timeline_version_id` | A derived ordering may change without rewriting events |
| Event | `event_id` | `event_assertion_version` | Occurrence identity differs from a source's assertion about it |
| Verification | `verification_id` | `verification_attempt_id` | A check has method, input, result, limitation, reviewer, and expiry |
| Request | `request_id` | `request_revision_id` / `request_effect_id` | Draft, sent request, acknowledgement, production, and appeal are distinct |
| Brief | `brief_id` | `brief_revision_id` | Investigation question and constraints, not a claim package |
| Draft | `draft_id` | `draft_revision_id` | Text is not approved or published merely because it exists in a CMS |
| Publication | `publication_id` | `publication_revision_id` / `publication_effect_id` | External channel state, not editorial approval |
| Correction | `correction_id` | `correction_revision_id` / propagation effect IDs | Decision, wording, channel propagation, and public observation are separate |

### Record both clocks

Every mutable assertion uses:

- **valid time:** when the claim, event, policy, role, company status, webpage state, or source statement was true or alleged to be true;
- **transaction time:** when the newsroom acquired, observed, entered, corrected, or superseded it.

Keep `source_published_at`, `provider_updated_at`, `valid_from/to`, `acquired_at`, `observed_at`, `recorded_at`, and `superseded_at` distinct. Preserve source clock, timezone, precision, and uncertainty. A later archive capture cannot prove a page existed earlier; a later filing may describe an earlier event; an API's `updated_at` is not acquisition time.

```yaml
event_assertion:
  event_id: evt_meeting_17
  assertion_version: 3
  asserted_valid_time:
    earliest: 2026-04-17T18:00:00-04:00
    latest: 2026-04-17T20:30:00-04:00
    precision: bounded_interval
    source_timezone: America/New_York
  transaction_time:
    acquired_at: 2026-08-29T09:42:18Z
    recorded_at: 2026-08-29T09:43:04Z
  asserted_by: ev_access_log_7
  conflicts_with: [evt_meeting_17_assertion_2]
  status: unresolved
```

## Deterministic services and bounded model work

| Task | Deterministic owner | Model may assist | Human decision |
|---|---|---|---|
| Hash/fixity and byte inventory | Cryptographic library and custody service | None | Investigate a mismatch |
| Metadata/container parse | Sandboxed, pinned parser | Explain fields with limitations | Decide evidentiary weight |
| Search and API traversal | Adapter owns query encoding, pagination, coverage, receipt | Propose bounded queries | Decide when search is sufficient |
| Claim ledger validation | Schema/graph validator checks locators, state, cycles, origin paths | Propose narrow claims and edges | Confirm material claims |
| Timeline normalization | Time parser retains source clocks, precision, alternatives | Propose candidate links/conflicts | Resolve consequential ambiguity |
| Entity reconciliation | Candidate generator and reversible graph | Suggest candidates and discriminators | Approve material merge/split |
| C2PA validation | Conforming validator returns assertion/signature/trust results | Summarize without a truth verdict | Assess alongside custody and context |
| OCR/ASR/translation | Pinned transform creates aligned derivative and coverage | Identify passages needing review | Approve consequential quotes/numbers |
| Corroboration | Origin/dependency graph exposes shared roots | Propose independence questions | Decide whether newsroom evidence bar is met |
| Draft/package | Template builder renders exact evidence/contradictions/limitations | Draft synthesis and questions | Reporter/editor/standards/counsel approve or reject |
| Publish/correct | CMS gateway prepares intent and reconciles target | No credentials or send control | Accountable newsroom role initiates and confirms |

A model answer is never an acquisition receipt, custody event, source promise, verification result, approval, or publication observation.

## Representative provider qualification map

The rows describe useful semantics and known limits, not endorsements.

| Surface and current primary source | What a qualified adapter may do | Meaning that must be preserved | Rights, retention, and finality boundary |
|---|---|---|---|
| Search: [Google Custom Search JSON API](https://developers.google.com/custom-search/v1/overview) | Discovery over an explicitly configured engine for an already-qualified customer | Result/snippet/ranking are provider observations, not captured evidence | Closed to new customers; official page schedules discontinuation for **2027-01-01**. Do not make it a new strategic dependency |
| News aggregation: [News API terms](https://newsapi.org/terms) | Discover candidate articles and retain provider IDs/queries/coverage | Metadata and links do not grant article rights or prove source independence | Developer plan is not production; third-party content rights remain with publishers. Qualify a commercial contract and storage/redistribution separately |
| Web capture: [WACZ 1.1.1](https://specs.webrecorder.net/wacz/1.1.1/), [WARC](https://www.loc.gov/preservation/digital/formats/fdd/fdd000236.shtml), [Memento RFC 7089](https://datatracker.ietf.org/doc/html/rfc7089) | Store capture bytes, request/response metadata, indexes, screenshot, replay record, and capture receipt | Container validity, capture time, origin time, replayability, and completeness are separate | Dynamic/authenticated/streamed dependencies may be absent; an archive holding is not a complete historical record |
| US federal directory: [FOIA.gov developer resources](https://www.foia.gov/developer/) | Search agency/component metadata and ingest published annual-report resources | Directory/form metadata is not legal advice, submission, acknowledgement, production, or appeal status | API key required; agency request POST behavior is a separate spec and remains human-controlled |
| US rulemaking: [Regulations.gov API v4](https://open.gsa.gov/api/regulationsgov/) | Read documents, comments, dockets, and explicit attachments under bounded pagination | Object classes, withdrawn state, configurable fields, attachment coverage, and page limits remain visible | A submitted comment may await agency approval; disable comment POST for this agent. API success is not publication or complete docket coverage |
| Company/regulatory: [SEC EDGAR APIs](https://www.sec.gov/search-filings/edgar-application-programming-interfaces) | Ingest submissions and XBRL facts with accession, form, CIK, taxonomy, unit, period, and context | Filing identity differs from company/entity resolution and from a reporter's claim | No authentication for documented endpoints, but SEC access policy/rates and filing amendments still apply; a filed assertion is not independently true |
| Legal-entity relationships: [GLEIF API](https://www.gleif.org/en/lei-data/gleif-api/) | Resolve LEI records and relationship data as candidate evidence | LEI, registration status, relationship record, and local entity candidate are distinct | Coverage is the LEI system's scope, not a complete beneficial-ownership or corporate-history graph |
| Geospatial: [OSMF Nominatim policy](https://operations.osmfoundation.org/policies/nominatim/) and [OSM copyright](https://www.openstreetmap.org/copyright) | Low-volume candidate geocoding only under an approved profile; retain query/result/version and manual verification | A geocode is a candidate location, not proof that an event occurred there | Public service caps heavy use at 1 request/second, forbids systematic queries/autocomplete, requires identification/attribution, says not to submit personal/confidential data, and can change policy without notice; production bulk use needs another qualified provider or self-hosting |
| Provenance/authenticity: [C2PA 2.3](https://spec.c2pa.org/specifications/specifications/2.3/specs/C2PA_Specification.html) | Validate manifest structure, bindings, signatures, assertions, trust, and validation status using a pinned validator/trust policy | Valid provenance association does not establish truthful content or correct context; absence is not evidence of fakery | Version 2.3 is dated December 2025; validator, trust list, supported formats, and deployment policy must be pinned and regression tested |
| Metadata/media forensics: [ffprobe](https://ffmpeg.org/ffprobe.html), IPTC/EXIF-capable pinned tooling | Offline parse, frames/waveforms, structural checks, metadata normalization, and reproducible derivatives | Parse success and embedded fields are signals, not authorship, location, date, or authenticity guarantees | Use isolated workers; preserve raw output/version/config. Specialist review is required for consequential findings |
| Transcription: [Google Cloud Speech-to-Text data logging](https://cloud.google.com/speech-to-text/docs/data-logging) | Only an approved project, region/model, retention and sensitivity route; aligned transcript remains a derivative | Transcript, speaker hypothesis, confidence and reviewed quote are separate | Official docs say customer audio/transcripts are not logged by default, but logging is opt-in; logged data is not deleted with the project and has separate terms. The deployment must prove the option is off and contract/data-routing is acceptable |
| Translation: [Cloud Translation API overview](https://cloud.google.com/translate/docs/api-overview) | Approved language/model/region with sentence or segment alignment and a human-review flag | Translation never replaces the original; modality, named entities, numbers and culturally loaded wording need review | Official overview says customer data/translations are not used to improve models; still qualify contract, logs, support access, region, glossary and deletion for the deployed edition |
| Secure tips: [SecureDrop documentation](https://docs.securedrop.org/en/latest/) and [current releases](https://securedrop.org/news/) | **No agent adapter.** A trained human may export a sanitized, approved derivative plus blind reference and custody receipt | SecureDrop source session/identity/contact stay in the source-protection environment | At research date: SecureDrop 2.16.1, Workstation 1.8.0 on Qubes 4.3, Inbox 1.6.0. Tor use can be observable and submitted files can contain identifying metadata; no product guarantees anonymity against every adversary |
| Evidence storage: [Amazon S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html) | Store versioned evidence under a matter policy, retention/hold decision, digest and custody ledger | Object version, retention mode/date, legal hold, delete marker, encryption key and custody event are distinct | Requires Versioning; governance and compliance modes differ; lock applies per object version and is not a full custody system. Lost encryption keys can make retained evidence unreadable |
| Newsroom/CMS: [WordPress Posts REST API](https://developer.wordpress.org/rest-api/reference/posts/) | Read approved metadata or create/update a draft in a restricted staging target after human approval | `POST /wp/v2/posts` acceptance, CMS status, public URL observation and editorial approval are separate | Model has no credential. Production publication/correction requires exact target/revision, human initiation, receipt and independent reconciliation |
| Workflow: [Jira Cloud REST API v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro/) | Create/read a bounded review task/package reference with local role mapping | Issue status/comment is not an approval unless the newsroom contract defines and signs that mapping | Qualify auth, permissions, pagination, timestamps, ordering, field schemas, retention and tenant/app access; workflow drift invalidates approval mapping |
| Notification: [Slack platform rate-limit changes](https://api.slack.com/changelog/2025-05-terms-rate-limit-update-and-faq) | Send a non-sensitive notification for an already-committed local event with stable effect ID | Delivery/API success is not editor acknowledgement, approval, or source contact | Rate limits and distribution terms differ by app type. Never place source identity, raw evidence, legal advice, or story-sensitive text in an ordinary notification |

### Bounded MCP rule

MCP is justified only as a transport boundary for an already-qualified read or package-reference capability. Under the [2025-11-25 authorization specification](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization), clients must use resource-bound authorization and must not pass through unrelated tokens. Task support in that specification line is experimental. MCP does not add evidence, custody, rights, approval, or effect semantics; those stay in the domain contract.

Never expose secure tips, the source identity vault, arbitrary browser control, public-record submission, fairness contact, legal advice, CMS publication, or correction authority through a generic MCP server.

## Worked lifecycle: anonymous procurement records tip

This fictional scenario is deliberately ordinary: a source alleges that a city technology contract was tailored for one vendor. It demonstrates the workflow, not the truth of the allegation.

### Phase 1 — admit the matter and protect the source

1. A trained journalist retrieves a submission inside the newsroom's supported SecureDrop workflow. The acting agent has no account, key, session, network path, or source identifier.
2. The journalist and source-protection owner perform threat/OPSEC assessment, establish ground rules through a human process, inventory files, and decide whether any export is safe.
3. An isolated worker calculates digests, inventories formats and metadata, checks for active content/malware, and creates safe representations. It does not automatically strip the only useful metadata or contact the source.
4. The human custodian creates `source_ref: src_blind_12`; only the vault maps it to contact/identity. An approved derivative and custody receipt enter `matter_204`.
5. Matter admission records jurisdiction, deadline, public-interest hypothesis, prohibited methods, retention/hold policy, source disclosure class, reviewer route, and stop/escalation conditions.

**Gate:** if source safety, authority, rights, or necessary compartmentation is unclear, the matter enters `safety_hold`; public research may continue only if it cannot increase risk.

### Phase 2 — preserve and separate what the files establish

```yaml
evidence_acquisition:
  evidence_id: ev_bid_sheet_1
  acquisition_id: acq_secure_export_44
  matter_id: matter_204
  source_ref: src_blind_12
  source_disclosure_class: reporter_only
  acquired_at: 2026-08-18T14:03:10Z
  representation:
    raw_vault_object_ref: vault-object://opaque/771
    approved_case_derivative_ref: evidence://matter_204/ev_bid_sheet_1/rep_2
    digest: sha256:...
  custody_head: custody_event_901
  rights_profile: confidential-submission/5
  limitations:
    - original_creation_time_unverified
    - source_identity_not_evidence_of_document_authorship
```

The system creates separate narrow claims:

- `clm_1`: the source alleged bid criteria were shared early;
- `clm_2`: the submitted spreadsheet contains a criterion matching the later RFX;
- `clm_3`: the spreadsheet existed before the RFX date;
- `clm_4`: a particular official authored or shared it;
- `clm_5`: the procurement was unlawfully tailored.

The spreadsheet may support `clm_2`; it does not by itself establish `clm_3–5`. Hash equality only shows byte equality between acquired representations.

### Phase 3 — deterministic public acquisition and bounded research

The controller assigns a few bounded jobs:

1. Capture the city RFX, award notice, council agenda, minutes, and referenced attachments. Every fetch records URL, request/response, bytes, provider/capture time, coverage and rights.
2. Locate agency contacts and public-record procedure from official sources. The system drafts, but a records specialist checks jurisdiction, scope, requester identity, fees and wording and initiates any request.
3. Search SEC filings and GLEIF only if the vendor/entity question needs them. Preserve accession/CIK/LEI and effective versions; do not merge by name alone.
4. Geocode a disclosed public meeting address only if relevant. Do not send confidential addresses or systematic batches to public Nominatim.
5. OCR records productions with page/region alignment and validate material dates, amounts, names and negation against the image. A no-result over unprocessed or truncated pages is not absence.

The model may propose a next query when it names the unresolved claim, expected discriminator, approved surface, result bound and stop condition. It cannot broaden the matter or contact anyone.

### Phase 4 — reconcile entities, time, origin and contradiction

The case ledger shows two vendors with similar names. The deterministic entity service proposes candidates; a reporter reviews incorporation identifiers, address history, LEI/filing context, and the procurement record before merging.

The timeline retains:

- spreadsheet embedded timestamp, with a `modifiable_metadata` limitation;
- source statement about receipt time;
- file acquisition time;
- RFX publication time;
- archive capture time;
- public-record email sent and received times;
- council meeting asserted local interval and official minutes approval date.

A records production reveals that a generic criterion circulated earlier to several vendors. That evidence contradicts the strongest interpretation and remains prominent in every context and package. The system traces whether three news articles all copied one municipal statement; one origin cluster counts as one path, not three corroborations.

### Phase 5 — build the claim and review package

The deterministic builder freezes `package_digest` containing:

- approved matter question, scope, prohibited methods and policy version;
- every material claim with allegation/observation/fact/inference/editorial state;
- exact support, contradiction, limitation, locator and origin/dependency path;
- source disclosure tier and mosaic/re-identification review result;
- entity candidates and merge decisions;
- timeline with clock/precision alternatives;
- records-request, fairness-contact and no-response receipts;
- authenticity/forensics results without binary overclaim;
- unresolved gaps and what additional evidence could discriminate them;
- draft wording tied to claim versions;
- package, tool, model, parser, context-compiler and adapter versions.

The reporter owns factual confirmation and source relationship; the editor owns story framing/readiness; standards/security review attribution and harm; counsel provides legal advice to the newsroom. The system records their decision, role, package digest, conditions and expiry. It never interprets silence or a Jira status as approval.

### Phase 6 — fairness contact and publication are human effects

The system may draft narrow questions for the vendor and city. A human checks identity, channel, source-protection leakage, deadline, wording and legal/editorial policy, then sends through the newsroom system. A late response changes a material claim, so the package digest changes and prior approvals expire.

```yaml
publication_effect:
  schema_version: journalism-effect/2
  effect_id: pub_story_88_rev_6
  operation: publish_story_revision
  tenant_id: newsroom_7
  story_id: story_88
  draft_revision_id: draft_88_6
  package_id: pkg_204_11
  package_digest: sha256:...
  requested_external_revision: city-contracts-2026-08-31-r1
  target: cms-production
  preconditions:
    approvals: [reporter_91, editor_32, standards_18]
    approvals_current: true
    source_protection_hold: false
  initiated_by: human_user_17
  idempotency_key: newsroom_7/story_88/city-contracts-2026-08-31-r1/publish
  state: prepared
```

The human initiates the effect through the CMS gateway. If the CMS call times out, state becomes `unknown`; the controller queries the exact story/external revision and, when appropriate, observes the public representation. It does not blindly repeat `POST /wp/v2/posts`.

### Phase 7 — correction and propagation

After publication, the city supplies a contemporaneous email showing the criterion was proposed by an independent consultant. The newsroom opens a correction matter version:

1. preserve the request/new email as evidence with custody and rights;
2. link it to affected claim and draft/publication versions;
3. reconstruct what was known and approved at each transaction time;
4. invalidate affected conclusions/packages without rewriting history;
5. obtain human factual/editorial/legal decisions on update, correction, withdrawal, or retraction;
6. create a new correction/publication effect per channel;
7. reconcile CMS, syndication, feeds, social posts, archive notes and partner acknowledgements separately;
8. retain unresolved downstream propagation as visible debt.

```yaml
correction_record:
  correction_id: corr_88_1
  correction_revision_id: corr_88_1_r2
  story_id: story_88
  affected_publication_revisions: [city-contracts-2026-08-31-r1]
  affected_claim_versions: [clm_5_v4]
  new_evidence: [ev_city_email_9]
  human_decision_ref: editorial_decision_711
  wording_digest: sha256:...
  channel_effects:
    cms: confirmed
    rss: confirmed
    syndication_partner_a: pending
    social_post: unknown
  opened_at: 2026-09-02T14:01:00Z
  valid_from: 2026-09-02T16:30:00Z
```

The correction becomes a curated evaluation candidate only after sensitivity, source safety, rights and root-cause review. It is not automatically converted into model memory.

## Unknown-effect reconciliation algorithm

Use the same protocol for records requests, fairness messages, review exports, publication and corrections:

1. commit `effect_intended` with stable semantic identity, exact payload digest, preconditions and approval;
2. initiate once with provider idempotency support when available;
3. record attempt and provider response separately from domain outcome;
4. on timeout or ambiguous response, transition to `unknown`, revoke automatic retry and schedule reconciliation;
5. query an independent target/source of truth using the expected external identity and content revision;
6. mark `confirmed`, `not_applied`, `failed`, or `manual_resolution_required` with evidence;
7. retry only `not_applied` with still-current approval and deadline; never infer success from a workflow comment or notification.

Cancellation stops future work but does not erase an effect that may already have happened. Leases fence stale workers; the reconciler is idempotent; correction is a new version/effect rather than destructive rollback.

## Runbooks that must work before live use

| Trigger | Immediate containment | Recovery evidence |
|---|---|---|
| Possible source identity or rare descriptor leaked | Stop model/vendor/export paths, place matter on safety hold, restrict logs/artifacts, invoke human source-protection plan | Access/export lineage, affected derivatives, revocation, safe communication decision, reviewer release |
| Malicious submission or parser escape | Isolate quarantine/worker, revoke its credentials, preserve sample safely, enumerate descendants | Clean rebuild, pinned-parser replay, artifact and network logs, dependent-package invalidation |
| Provider rights/terms revoked | Disable manifest and exports, identify stored objects/derivatives and holds, involve rights owner | Contract snapshot, inventory, delete/restrict decision, tombstone propagation, replacement qualification |
| Provider field/status drift | Open circuit, freeze affected conclusions, replay canaries against captured fixtures | Schema diff, coverage/finality assessment, adapter release and requalification |
| Unknown records/contact/publication effect | No blind resend; query sent-system/portal/CMS/external revision | Intent, attempt, target observation, human resolution and corrected state |
| Source coercion or immediate danger | Stop automated contact and disclosure; escalate to trained newsroom/security leadership | Human-owned safety decision; minimal protected incident record |
| Defamation/privacy/public-interest dispute | Freeze publication-adjacent work; preserve claim/evidence versions; route through newsroom review/counsel | Exact package and wording, evidence/contradictions, fairness record, accountable decision |
| Dependency or model compromise | Kill switch, pin deterministic-only bundle, revoke token, quarantine new outputs | Software bill/release bundle, affected-run query, replay, rollback and safe canary |
| Region loss or ransomware | Isolate, restore clean control/evidence projections within matter-specific RPO/RTO; keep source vault in its stronger plan | Restore digest/fixity, event high-watermarks, tombstones, pending/unknown effects, custody continuity |
| Recovery backlog threatens deadlines | Admit source-safety/correction lanes first, cap new research, restore deterministic capture/ledger before model work | Queue age by class, review capacity, recovery throughput and explicit missed-deadline decisions |

Never automatically alert a confidential source during an incident; the channel, timing, device, and recipient may be unsafe.

## Stage 0–6 lab and exit evidence

| Stage | Reader exercise | Required exit evidence |
|---:|---|---|
| 0 | Run one public-record matter using only deterministic capture, fixity, OCR, claim ledger, timeline and review checklist; measure the manual path | Baseline quality/time/cost and a specific adaptive bottleneck; otherwise stop here |
| 1 | Add one public-only bounded query decision over frozen fixtures; inject no-result, partial pagination and hostile content | Zero forbidden effects, declared coverage, bounded stop, improvement over Stage 0 on predeclared metric |
| 2 | Complete the fictional matter through reporter package, including a contradiction and namesake collision | Immutable evidence/derivatives, locators, bitemporal ledger, human-accepted usefulness without confidential input |
| 3 | Qualify two read adapters and one draft-package adapter; crash before/after each state/effect boundary and compact repeatedly | Conformance reports, stable identities, exact seven-memory policy, continuity receipts, idempotent recovery and no duplicate effects |
| 4 | Shadow live public matters; red-team source leakage, injection, tenant isolation, legal/rights denial and publication ambiguity | Threat review, SLOs, separate audit/trace/evidence, on-call runbooks, hard eval gates, tested rollback |
| 5 | Load/soak public fetch, OCR/media and review queues; lose a region/store projection and recover while new work arrives | Fair admission, deadline/source-safety/correction reserves, recovery-load capacity, RPO/RTO evidence, DR custody/tombstone/effect reconciliation |
| 6 | Turn a reviewed correction and provider drift incident into minimal fixtures; change one behavior component | Versioned behavior bundle, replay, shadow, canary, drift alarms, affected-output query, rollback and human-approved failure mining |

## Release and refresh checklist

- [ ] The no-agent/deterministic baseline is documented and still compared in evaluation.
- [ ] Each live operation has a current manifest, conformance report, rights owner, security owner, expiry and tested kill switch.
- [ ] Search/news/API results are discovery until captured and qualified as evidence.
- [ ] Provider pagination, coverage, time, update/withdrawal, retention, redistribution and finality semantics are explicit.
- [ ] SecureDrop, source identity, human contact, editorial/legal decisions and CMS credentials remain outside the acting model.
- [ ] Documents/media run in hostile-content quarantine; all derivatives retain raw lineage and limitations.
- [ ] Matter, story, source, person, entity, document, media, claim, evidence, custody, timeline, event, verification, request, brief, draft, publication and correction identities are not conflated.
- [ ] Both valid and transaction time survive update, correction, compaction, replay and export.
- [ ] The [seven memory lifetimes and continuity receipt](05-research-loop-tools-context-and-memory.md#exactly-seven-memory-lifetimes) pass deletion and poisoning tests.
- [ ] Every external effect can be cancelled before initiation, fenced, timed out, reconciled after ambiguity, corrected and audited.
- [ ] Release exercises cover failure injection, human calibration, queue/recovery load, DR, incident response, drift, rollback and correction mining.

Requalify immediately on provider API/terms/rights/retention/finality changes; deployment-account or region changes; new content classes; adapter/auth/schema updates; SecureDrop release/threat-model changes; C2PA validator/trust changes; model/parser/transform changes; a leakage, coverage, duplicate-effect, missed-correction or provider-compromise incident. Live calls were not executed for this blueprint, and no public documentation can establish a newsroom's private contract, account settings, legal rights, threat environment, or editorial policy.

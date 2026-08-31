# Security, Privacy, Tenancy, Brand, Rights, and Accessibility for Content Editorial Agents

## Threat model

Content systems combine high-value unpublished information, third-party material, broad business integrations, and public-effect credentials. Assume both malicious inputs and ordinary mistakes.

| Threat actor/path | Goal or accident | Example impact |
|---|---|---|
| Malicious source/document/page | Prompt-inject or exfiltrate | Retrieved text asks model to reveal brief or call publisher |
| Compromised editor account | Publish harmful content or access another tenant | Draft-to-publish escalation |
| Overprivileged service/model tool | Accidental broad effect | Deletes asset or publishes to all sites |
| Insider or contractor | Copy unpublished/licensed/personal content | Confidentiality and rights incident |
| Poisoned knowledge/style entry | Persist behavior manipulation | Forbidden claims become “brand rule” |
| Provider/supply-chain change | Drift, leakage, outage, malicious package | Different retention or tool behavior |
| Cross-tenant bug | Retrieve/index/cache wrong data | Disclosure and brand contamination |
| Telemetry/export path | Retain raw sensitive content | Unpublished work appears in logs |
| Concurrency/retry bug | Duplicate or stale release | Wrong version visible publicly |

## Control and content separation

Use an explicit trust taxonomy:

| Class | Examples | How handled |
|---|---|---|
| Trusted control | authenticated assignment fields after gateway; signed policy projection; controller capability | Can constrain behavior/authorize finite action |
| Authoritative domain data | approved PIM field, current policy record | Can support scoped claims; still untrusted as instruction |
| Untrusted content | web, files, CMS text, comments, metadata, tool output, model output | Parse/display/reason over as data only |
| Secret | tokens, keys, signing material | Never enters model context; executor fetches just in time |
| Sensitive identity/consent | people, releases, private contact details | Minimized reference; restricted reviewer view |

Do not solve prompt injection with a sentence telling the model to ignore it. Enforce capabilities, allowlists, schemas, egress policy, and approval at the application boundary.

## Prompt-injection defenses

1. Normalize and label external content; preserve its source identity.
2. Authorize sources before retrieval; do not retrieve broadly then filter in the prompt.
3. Put content and control in separate structured fields.
4. Exclude instruction-like metadata fields unless needed for the task.
5. Limit tools to assignment-scoped reads and proposal writes.
6. Validate every tool argument and destination outside the model.
7. Deny arbitrary URL fetch, shell/code execution, credential access, and general messaging by default.
8. Use egress allowlists and network isolation for acquisition/render workers.
9. Detect canary attempts, cross-tenant IDs, secret patterns, and unexpected action language.
10. Test indirect injections in HTML, PDF/OCR, images/alt text, comments, JSON fields, redirects, and tool errors.

Even a “trusted” CMS field can contain pasted hostile content. Authority derives from the field's domain semantics, never its prose.

## Identity, authorization, and service principals

- Authenticate users through the organizational identity provider; require strong authentication for publishers and break-glass roles.
- Authorize every object and action by tenant, role, assignment, risk, destination, and state.
- Recheck role at approval and effect time.
- Use separate service principals for acquisition, draft exchange, render, evaluation, rights read, and publication.
- Keep publication/delete permissions out of interactive/model workers.
- Make human delegation explicit with expiry and reason; never infer it from shared accounts.
- Review privileges periodically and log privileged-function use.

### Capability example

`read source collection X for assignment Y until T` is a safe capability. `Contentful API token` is not.

## Secret handling

- Store secrets in a secret manager, not prompts, environment dumps, code, CMS fields, or telemetry.
- Fetch them just in time inside the adapter executor.
- Scope to tenant/account/environment and operation; prefer short-lived workload identity.
- Rotate automatically and test rotation without downtime.
- Redact headers, query strings, stack traces, provider bodies, and command output.
- Revoke on employee/offboarding, incident, adapter retirement, and project closure.
- Maintain a credential inventory with owner, scopes, region, rotation, and last-used evidence.

If a source asks the model for a token, the application does not possess a model-visible token to reveal.

## Tenant and regional isolation

Enforce tenant context at every layer:

| Layer | Isolation control |
|---|---|
| API | tenant derived from authenticated principal, never caller body alone |
| Database | tenant keys and row policies/schema/database cells as risk requires |
| Object storage | tenant/region buckets or prefixes with policy-bound principals |
| Queue/workflow | tenant in signed job envelope; worker validates before load |
| Search/vector | tenant-specific index/namespace and pre-retrieval filter |
| Cache | tenant included in every key; sensitive responses non-shared |
| Model provider | region/account/project and data-handling policy selected by tenant |
| Telemetry | tenant-aware access and region; bodies excluded |
| Export/backup | encryption, tenant manifest, region, retention, restore test |

High-risk tenants or regions may require separate cells: database, object store, queue, keys, model endpoint, telemetry, and workers. A region label in application metadata is not residency proof.

## Privacy engineering

Map every data action: collect, generate, transform, use, share, log, retain, correct, export, and delete.

### Privacy record

```json
{
  "data_category": "customer_quote_contact",
  "purpose": "verify attribution before publication",
  "subjects": ["customer"],
  "lawful_or_policy_basis_ref": "privacy-decision-2026-114",
  "allowed_processors": ["internal-editorial", "approved-model-provider-eu"],
  "model_projection": "pseudonymous_quote_and_role_only",
  "retention": "delete contact 30 days after release unless dispute/hold",
  "rights_request_route": "privacy-ops",
  "region": "EU",
  "owner": "privacy_editorial_owner"
}
```

Apply purpose limitation, minimization, accuracy, storage limitation, integrity/confidentiality, and accountability where the applicable policy/law requires them. Do not assume all “public” personal data is safe to republish or persist in memory.

### Data-subject and deletion workflow

1. authenticate the request and establish applicable scope;
2. locate source records, claims, drafts, published content, assets, memory, indexes, caches, telemetry, exports, and backups;
3. evaluate legal hold, records, public-interest, contractual, and correction requirements with authorized owners;
4. delete, redact, restrict, or correct each eligible projection;
5. rebuild indexes and invalidate caches;
6. record what was changed, retained, and why;
7. verify provider deletion/retention behavior and close only with evidence.

Deleting a session transcript is not complete deletion.

## Copyright and licensing boundary

Engineering controls should represent, not decide, legal rights.

- distinguish facts/ideas from protected expression but do not infer unrestricted use;
- preserve source and exact excerpts used;
- track license version, licensor, asset/work identity, territory, channel, audience, time, transformations, attribution, share-alike/no-derivatives/noncommercial duties, and contract terms;
- keep plagiarism/similarity scores as triage signals, not infringement findings;
- prevent long source reproduction beyond the task's authorized processing basis;
- record human contribution and AI-generation provenance where policy requires it;
- evaluate AI disclosure with a dated jurisdiction/channel policy;
- invalidate clearance when use, asset, rendition, territory, channel, term, or policy changes.

An ODRL/RightsML expression, IPTC field, C2PA credential, stock receipt, or Creative Commons label is evidence. The accountable rights owner/counsel decides the permitted use.

## Brand governance

Treat brand/style as versioned policy, not a long prompt.

```json
{
  "brand_policy_id": "brand-core@2026.08",
  "effective_from": "2026-08-01T00:00:00Z",
  "markets": ["global"],
  "required_terms": [{"concept": "support_plan", "term": "Care Plan"}],
  "forbidden_claim_patterns": ["guaranteed results", "works everywhere"],
  "voice_profiles": {"help": "direct_calm", "product": "specific_confident"},
  "controlled_blocks": ["legal_footer@11", "safety_warning_battery@6"],
  "exception_role": "brand_director",
  "owner": "brand_ops",
  "digest": "sha256:..."
}
```

Separate:

- deterministic terminology and forbidden-claim rules;
- model-assisted style suggestions;
- human brand judgment;
- controlled language that must never be paraphrased.

Measure rule precision and reviewer overrides. A noisy brand checker trains reviewers to ignore real findings.

## Accessibility governance

WCAG versions, legal incorporation, property scope, and organizational targets vary. Create a dated accessibility policy for each property/content type/jurisdiction.

| Layer | Owner | Evidence |
|---|---|---|
| Structured content | Content schema/team | required alt/caption/heading/table/link fields |
| Renderer/component | Web/product engineering | semantic markup, keyboard/focus, responsive behavior |
| Automated rules | Accessibility tooling | rule IDs, tool/version, DOM/render, failures |
| Human content review | Trained editor/accessibility reviewer | meaning, equivalence, reading order, cognitive clarity |
| Assistive-technology/user testing | Accessibility program | browser/AT matrix, task outcomes, limitations |
| Conformance/legal statement | Authorized owner/counsel | scoped claim and applicable standard/law |

### Alt text contract

```json
{
  "asset_use_id": "ause_01J...",
  "purpose": "instructional",
  "context": "Shows port orientation for step 2",
  "candidate": "The USB-C port is on the left side, beside the status light.",
  "decorative": false,
  "generated_by": "model-activity:act_...",
  "review": {"status": "approved", "reviewer": "idp:..."},
  "rendered_digest": "sha256:..."
}
```

The same image can need different alternative text in different contexts. Object detection labels are not context-equivalent descriptions.

## Legal and jurisdiction policy routing

At minimum, route on:

- publisher/entity and intended territories;
- audience, including children or vulnerable groups;
- content purpose: public-interest information, advertising, product instructions, employment, health, finance, legal, safety;
- AI generation/manipulation and degree/type of human editorial control;
- personal data/privacy/publicity/model-release facts;
- rights/license/contract and asset transformations;
- accessibility property/entity and applicable incorporated standard;
- regulatory product/version and channel.

The result may be `allowed`, `allowed_with_conditions`, `specialist_review`, or `prohibited`. The model cannot turn `unknown` into allowed.

## Content sanitization and render security

- Store structured rich text, not arbitrary unsanitized HTML, where possible.
- Sanitize generated/imported markup using an allowlist outside the model.
- Block scripts, event handlers, dangerous URLs, unexpected embeds, tracking pixels, and active office/PDF content in preview.
- Isolate render workers from publisher credentials and internal networks.
- Fetch remote assets through controlled proxies with size/type/time limits and SSRF protections.
- Validate redirects and canonical domains.
- Apply malware scanning and safe derivative generation before editors preview uploads.
- Use Content Security Policy and safe component libraries on public renderers.

## AI/provider governance

Qualify each model/provider/build for:

- supported regions and subprocessors;
- input/output retention, training, abuse monitoring, and deletion contract;
- encryption and tenant/project isolation;
- model/build pinning and change notice where available;
- structured-output/tool-call behavior and refusal modes;
- context/output/media limits, rate limits, and timeout semantics;
- safety/filter changes and content categories;
- audit/export availability;
- fallback behavior and outage plan.

Provider claims do not replace contractual and local technical verification. Never send source/rights/personal data to a provider merely because it fits in context.

## Security test suite

Include:

- direct/indirect prompt injection in every supported content/media/metadata path;
- cross-tenant IDs, cache keys, search filters, object URLs, and webhook events;
- stale/revoked role and capability replay;
- secret injection and exfiltration canaries;
- arbitrary URL/SSRF, redirect, and oversized decompression cases;
- malicious HTML/SVG/office/PDF and active media metadata;
- poisoned style/knowledge/memory entries;
- forged C2PA/IPTC/license metadata and expired rights;
- telemetry and error-body leakage;
- supply-chain/model/provider version changes;
- concurrent edit and stale approval attempts.

## Incident triggers

Immediately stop effects and preserve evidence when:

- unpublished or cross-tenant content is exposed;
- a model/provider received prohibited content or a secret;
- public content includes a material false, harmful, private, unlicensed, inaccessible, or off-brand element;
- rights/consent/disclosure status was misapplied;
- a publisher credential or service principal is compromised;
- poisoned knowledge or source injection changed a proposal/decision path;
- correction/deletion failed to propagate.

The erroneous-publication runbook is in [deployment and incidents](09-deployment-scaling-cost-incidents-and-evolution.md).

## Security and governance gates

- Zero model-visible publisher/delete credentials.
- Zero cross-tenant retrievals in adversarial and load tests.
- 100% of external content carries an untrusted-instruction label through context compilation.
- 100% of public asset uses have use-scoped rights state and reviewer identity.
- 100% of regulated/legal/AI-disclosure decisions use a current jurisdiction policy or escalate.
- Raw content is absent from default production traces/logs; redaction canaries pass.
- Deletion/correction drills cover authoritative stores and all projections.
- Accessibility releases include automated and required human evidence; no automated conformance claim is emitted.

## Anti-patterns

| Anti-pattern | Failure | Replacement |
|---|---|---|
| Trusted source text treated as trusted instruction | Business system becomes control injection path | Domain authority scoped to claims; prose remains data |
| Broad shared integration token | One compromise crosses tenants and effects | Operation/tenant-scoped principals and capabilities |
| “Public data” bypasses privacy | Republishing and memory create new risk | Purpose and data-action review |
| Similarity score decides infringement | Tool cannot evaluate ownership/exceptions/substantiality | Human rights/legal review with source evidence |
| Brand guide pasted into every prompt | Stale, costly, and ungoverned | Versioned rule projection and controlled blocks |
| Auto-generated alt text accepted | Misses context/equivalence | Candidate plus human review |
| Region field in database | Does not prove provider/backup/telemetry location | Cell-level mapping and verified contracts |

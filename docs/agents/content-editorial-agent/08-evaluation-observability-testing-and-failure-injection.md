# Evaluation, Observability, Testing, and Failure Injection for Content Editorial Agents

## Evaluation unit

Do not score only the final prose. Evaluate the complete trajectory and its typed artifacts:

`assignment → retrieval → claims → plan → revision/patch → findings → approvals → effects → observations → correction`

A polished article with an invented number, expired asset license, stale approval, duplicate publication, or missing correction is a failed run.

## Evaluation corpus

Build a local, versioned corpus stratified by:

- article, product narrative, help, release note, and channel adaptation;
- risk tier, audience, brand, jurisdiction, and accessibility needs;
- short/long and simple/reference-heavy assignments;
- clean, incomplete, contradictory, stale, malicious, and rights-constrained sources;
- common CMS/DAM/PIM/provider states and errors;
- historical near misses and sanitized incidents;
- hard negatives where the correct action is to stop.

Freeze a release gate set. Keep a separate exploration set. Prevent outcome-memory leakage into either; record provenance and rights for every example.

## Layered evaluation stack

| Layer | Oracle | Example failure |
|---|---|---|
| Contract | Schema, IDs, state guards | Revision missing parent/digest |
| Evidence | Reviewed claim/source ledger | Citation does not entail claim |
| Content | Human rubric and deterministic rules | Qualifier removed; wrong audience |
| Rights/governance | Approved policy decisions | Expired web-only image used on social |
| Accessibility | Automated rules plus human/AT tasks | Alt text omits instructional detail |
| Security | Adversarial suite and policy logs | Source prompt changes destination |
| Trajectory | Expected action/stop/approval path | Model continues after contradiction |
| Effect | Sandbox/provider observation | Timeout creates duplicate post |
| Operations | Load/recovery/failover evidence | Correction queue starved by renders |
| Economics | Baseline comparison | Cost rises and reviewer time does not fall |

## Claim evaluation

Annotate material atomic claims in references and outputs.

### Claim metrics

- **Claim precision:** supported output claims / all factual output claims.
- **Required-claim recall:** correctly represented required claims / required claims.
- **Material unsupported rate:** unsupported material claims / material output claims.
- **Citation entailment accuracy:** claim–source edges where located text supports the scoped claim.
- **Locator resolution rate:** locators resolving against the recorded representation.
- **Qualifier preservation:** outputs retaining required scope, uncertainty, denominator, date, and exception.
- **Contradiction handling:** contradictory cases stopped or explicitly presented according to policy.
- **Stale-source escape rate:** released claims whose source violated freshness policy.
- **Invented quote/number/name/product-fact rate:** count per evaluated item and per 1,000 words; target zero in release gate sets.

Model graders may help triage but are not the sole oracle. Calibrate them against expert review, blind provider/model identity, and audit disagreements.

## Rights and provenance evaluation

Create cases varying license, territory, channel, term, transformation, attribution, consent, and incomplete metadata.

| Metric | Critical interpretation |
|---|---|
| Unsafe false-clearance rate | Agent/policy says use is clear when oracle requires reject/review; target zero |
| Unnecessary escalation rate | Operational burden; optimize only after safety |
| Duty extraction accuracy | Attribution/change/share-alike/placement duties captured |
| Use-binding accuracy | Decision matches channel, territory, audience, transformation, and dates |
| Expiry invalidation recall | Every affected approval/release is held |
| Provenance overclaim rate | C2PA/IPTC/metadata presented as truth or ownership |
| AI-disclosure routing accuracy | Correct policy/role/escalation for jurisdiction facts |

Include forged/invalid/absent C2PA, contradictory IPTC fields, ambiguous license labels, expired model releases, and a provider-generated asset with substantial human edits.

## Brand and style evaluation

Separate deterministic rule quality from subjective fit.

- terminology-rule precision/recall;
- forbidden-claim detection recall;
- false-positive findings per item and reviewer dismissals;
- controlled-language exact preservation;
- voice rubric scores from blinded trained reviewers;
- preference leakage across brand/content type/tenant;
- reviewer edit distance and reason codes;
- representation fairness and audience comprehension where applicable.

Do not optimize “sounds like us” until claim accuracy, rights, and accessibility gates pass.

## Accessibility evaluation

Use three levels:

1. **Structured-content checks:** required fields, headings, table headers, captions/transcripts, link purpose, language, asset purpose.
2. **Rendered automated rules:** pinned tool/rules/browser/viewport against representative pages and complete flows.
3. **Human and assistive-technology review:** context-equivalent alternatives, reading/interaction order, cognitive clarity, captions/transcripts, responsive and third-party behavior.

Metrics:

- blocker/critical defect escape rate;
- automated rule precision by rule;
- human alt-text acceptance/edit/reject rates with reason;
- headings/link/caption/label defect rate;
- complete-process task completion in required browser/AT matrix;
- time to remediate and recurrence by component/content source;
- policy coverage by property and jurisdiction.

Passing automation is not a conformance statement.

## Review and approval evaluation

Seed material and non-material changes to test invalidation.

- required-review routing recall;
- wrong-role approval rejection;
- stale/expired/revoked approval rejection;
- approval invalidation precision and recall;
- review UI digest consistency;
- conditional-approval enforcement;
- open-blocker release rate (target zero);
- reviewer active minutes, queue wait, reopen rate, and override reasons.

Track whether human review catches defects, not merely whether a human clicked approve.

## Trajectory evaluation

Score:

- selected only permitted tools/actions;
- respected source scope and budgets;
- asked the smallest useful question;
- stopped on injection, contradiction, missing rights, policy uncertainty, or no progress;
- did not repeat identical retrieval/revision cycles;
- persisted claims/patches/findings in required schemas;
- never attempted a publication capability;
- produced a valid continuity receipt on compaction/interrupt;
- reconciled unknown effects before retry.

Use exact state/policy assertions where possible. Do not rely on reading chain-of-thought; evaluate observable inputs, actions, artifacts, and outcomes.

## External-effect evaluation

Run against provider sandboxes and private test destinations.

| Scenario | Expected invariant |
|---|---|
| Connection lost before commit | eventual not-applied classification or safe retry |
| Connection lost after commit | reconcile applied; no duplicate |
| 429 with `Retry-After` | bounded delayed retry, same semantic operation |
| Stale ETag/version | conflict; no overwrite; rebase required |
| Duplicate/out-of-order webhook | deduplicated and state remains monotonic |
| Provider returns 202 | not marked visible until observed |
| Scheduled content becomes stale | schedule held/canceled and approvals invalidated |
| Cancel races dispatch | truthful `cancel_pending/too_late`, followed by observation |
| Multi-destination release partial | `partial`; no false complete |
| Correction destination unavailable | open propagation gap and retry/escalation |

## Adversarial suite

Include attacks in:

- user brief, source page, PDF/OCR, image metadata/alt text, CMS comment, DAM metadata, PIM field, search snippet, tool error, redirect target;
- encoded/hidden/multilingual instructions and payloads resembling policy JSON;
- attempts to change tenant, destination, role, disclosure, or rights state;
- malicious links/embeds, SSRF, large files, decompression bombs, and active content;
- poisoned brand/style/memory examples;
- forged provider/webhook receipts and stale capabilities;
- requests to reproduce long licensed text or infer private attributes;
- generated content designed to pass keyword checks while violating meaning.

Expected result is often a safe partial artifact plus escalation, not a refusal with no useful evidence.

## Failure injection plan

| Layer | Inject | Verify |
|---|---|---|
| Model | timeout, rate limit, malformed structure, refusal, changed style | bounded retry/fallback; no state corruption |
| Retrieval | stale index, deleted source, permission change, contradictory result | provenance/freshness checks and stop |
| Database | transaction abort, failover, stale version | no duplicate event/effect; safe resume |
| Object store | missing/corrupt rendition, replication lag | digest check; release blocked |
| Queue/workflow | duplicate delivery, lease expiry, worker crash, clock skew | deduplication, fencing, timeout semantics |
| CMS/DAM/PIM | 409/412, 429, partial batch, delayed visibility | conflict/backoff/reconciliation |
| Webhook | invalid signature, replay, out-of-order | reject/dedupe/re-read |
| Render | browser crash, third-party timeout, layout variation | artifact incomplete; no approval |
| Human | reviewer unavailable, role revoked, approval expires | escalation/substitution policy; never auto-approve |
| Region | provider or cell outage | isolation, failover policy, recovery capacity |

## Observability model

### Trace

One trace per workflow decision/effect chain, with linked traces for long waits. Suggested spans:

```text
assignment.admit
context.compile
model.invoke
tool.retrieve
claim.validate
revision.propose
render.build
review.submit
approval.record
effect.dispatch
effect.observe
effect.reconcile
correction.propagate
```

Attributes: tenant pseudonymous ID, assignment/content/revision/release/effect IDs, content type, risk tier, policy/behavior/adapter versions, model provider/build, token/cost counts, status class, retry count, and digests. Do not record raw source, prompt, completion, unpublished prose, license contract, personal data, or secrets by default.

OpenTelemetry GenAI conventions were still evolving at the research cut-off; pin the telemetry schema and own migration.

### Logs

Use structured logs for operational diagnosis:

- timestamp, severity, service/worker, region, trace/span/correlation;
- domain IDs and state transition;
- sanitized error class and provider request ID;
- policy/adapter/behavior versions;
- retry/reconciliation decision;
- no raw content unless a separately authorized diagnostic capture is time-limited and audited.

Audit evidence is a distinct append-only domain record, not whatever happened to be logged.

### Runtime metrics

**Quality:** unsupported claim, citation error, rights escalation/false clearance, style override, accessibility escape, correction rate.

**Workflow:** assignments admitted/rejected, state age, review queue, approval expiry, cancellation latency, no-progress stop.

**Effects:** intents, accepted, applied, partial, unknown, duplicates, reconciliation age, correction gaps.

**Platform:** request/queue/model/render/provider latency, error/rate-limit, queue depth/age, worker saturation, storage/search lag.

**Economics:** tokens, model/media/render/tool/provider cost, active reviewer minutes, cost per approved/released item.

Avoid high-cardinality content text, URLs, user emails, or claim propositions in metric labels.

## Audit record

An audit view should reconstruct:

- authenticated requester/owner/reviewer/publisher roles;
- assignment/policy/schema/behavior versions;
- sources/claims/rights/assets and their decisions;
- every immutable revision/redline/render and digest;
- findings, approvals, invalidations, exceptions, and break-glass use;
- effect intent, capability, provider receipt, public observation, retry/reconciliation;
- corrections/withdrawals/retractions and destination acknowledgements;
- data access, export, retention, hold, correction, and deletion events.

Protect audit records from general editors and model context. Record access to them.

## SLOs and error budgets

Example starting SLOs; calibrate from business impact:

| SLI | Example target |
|---|---:|
| Assignment admission availability | 99.9% monthly |
| Interactive draft-step latency | p95 < 20 s for supported class |
| High-priority correction queue start | 99% < 2 min |
| Scheduled release starts inside window | 99.5%, excluding authorized holds |
| Effect unknown age | 99% reconciled < 10 min; high-risk page < 2 min |
| Duplicate publication effects | 0 |
| Released render differs from approved digest | 0 |
| Cross-tenant exposure | 0 |
| Material unsupported claim in release gate | 0 |
| Correction required destinations acknowledged | 99% within policy deadline; unresolved explicitly escalated |

Do not hide excluded/held releases in availability metrics. Report policy holds, provider failures, and operator waivers separately.

## Release scorecard

A behavior/adapter release passes only when:

- critical contract/security/authority/effect invariants have zero failures;
- claim and locator thresholds meet the content-type gate;
- unsafe rights false-clearance is zero;
- accessibility blocker escape is zero in evaluated scope;
- required-review routing/invalidation tests pass;
- adversarial injection and cross-tenant suites pass;
- fault injection produces expected state/reconciliation;
- quality confidence intervals and regression slices are reported;
- cost/latency/reviewer-time do not exceed defined budgets;
- rollback is exercised, not merely documented.

## Controlled failure mining

Mine production only through a governed pipeline:

1. collect minimized failure metadata and reviewer reason codes;
2. remove or tokenize personal, confidential, licensed, and tenant-sensitive content;
3. obtain rights/policy approval for any retained examples;
4. cluster by control failure, not embarrassing prose;
5. create a synthetic or authorized regression case;
6. add it to exploration, then frozen gate set under review;
7. propose policy/tool/prompt/schema changes in a behavior bundle;
8. shadow, canary, and monitor before promotion.

Never automatically fine-tune or alter prompts from raw production content.

## Anti-patterns

| Anti-pattern | Failure | Replacement |
|---|---|---|
| BLEU/readability as quality | Misses evidence, rights, meaning, effects | Layered domain evaluation |
| Model grades itself | Correlated bias and false confidence | Expert oracles plus calibrated blinded graders |
| Golden prose exact match | Punishes valid variation, misses factual defects | Claim/contract/rubric outcomes |
| Human approved = correct | Click may be uninformed or wrong role | Review evidence and defect-detection measures |
| Trace every prompt | Creates sensitive shadow corpus | IDs/digests/metrics; opt-in controlled diagnostic capture |
| Load test only model calls | Misses render/media/reconciliation bottlenecks | End-to-end queue and recovery load |
| Learn automatically from corrections | Poisons memory and violates rights/privacy | Controlled failure-mining pipeline |

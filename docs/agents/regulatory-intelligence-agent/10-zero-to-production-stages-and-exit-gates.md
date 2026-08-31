# Zero-to-Production Stages and Exit Gates

## How to use this roadmap

Progress by evidence, not calendar. Each stage must preserve a simpler fallback and may be the correct stopping point. A later stage cannot weaken source authenticity, source rights, temporal correctness, professional ownership, tenant isolation, or effect integrity.

At every stage, keep these records separate:

```text
source text ≠ official status ≠ extracted provision
≠ applicability hypothesis ≠ interpretation ≠ owner decision
≠ obligation ≠ control ≠ external effect
```

## Stage map

| Stage | Outcome | Maximum default authority |
|---:|---|---|
| 0 — Qualify | Deterministic subscription/rule database and proof that model assistance is needed | Offline/read-only triage |
| 1 — First bounded loop | Typed candidate extraction over frozen, licensed fixtures | R1 candidate only |
| 2 — Useful MVP | One real jurisdiction/source family and one entity/product advisory workflow | R1 plus R2 review-case effect |
| 3 — Reliable v1 | Durable bitemporal state, corrections, compaction, rights, and reconciled internal handoff | Exact-approved R3 internal handoff |
| 4 — Production readiness | Identity, tenancy, security, release, SLO, tracing, runbooks, and controlled launch | Same human-owned decisions and bounded effects |
| 5 — Scale/resilience | Isolated cells, fair admission, backpressure, source resilience, and DR | No authority increase |
| 6 — Continuous evolution | Correction/failure mining and governed behavior/source/corpus changes | Propose changes; release owners approve |

## Stage 0 — Qualify the problem and deterministic baseline

### Stage 0 objective and non-goals

Prove the selected workload, source coverage, human workflow, temporal complexity, value, and risk. Build a deterministic subscription, rule/filter database, evidence archive, and review queue before adding a model.

Non-goals:

- no conversational assistant;
- no model-generated interpretation, applicability, obligation, or mapping;
- no claim of global or comprehensive regulatory coverage;
- no production GRC/control writes;
- no vector database, agent framework, workflow engine, graph database, or long-term model memory;
- no ingestion of licensed content before rights approval.

### Stage 0 architecture

```text
approved source list/feed/export
→ deterministic acquisition + digest
→ metadata/keyword/classification/scope rules
→ reviewed source/version/date register
→ human triage spreadsheet/database/queue
→ ordinary ticket/manual handoff
```

Use immutable fixtures/object storage and a small relational rule database when possible. A reviewed spreadsheet can be a prototype only if source/version/formula history is controlled and reproducible.

### Stage 0 authority and approvals

Offline/read-only. Source, scope, rights, and rule filters are approved by the regulatory/content-rights owners. Humans make all interpretation, applicability, obligation, and handoff decisions. No model or production writer credential exists.

### Stage 0 inputs and outputs

- **Inputs:** one bounded jurisdiction/source family; source status/licence documentation; historical final/proposed/correction/guidance samples; current human process; entity/product fact owners; selected labels.
- **Outputs:** source catalog and exclusions; rights profiles; normalized IDs/status/date fields; deterministic rule baseline; reviewed change/obligation fixtures; workload/cost/risk report.

### Stage 0 jurisdiction, rule, and obligation state

- Define the jurisdiction/source status vocabulary with responsible legal/regulatory owners.
- Record instruments, versions, renditions, acquisitions, raw date expressions, and corrections separately.
- Store human-labeled provision and obligation candidates only for evaluation; they are not an automated obligation ledger.
- Record what the baseline cannot resolve: cross-references, exceptions, multilingual differences, entity/product predicates, or impact mappings.

### Stage 0 events and effects

Events may be simple append-only acquisition and triage records. External effects are manual and outside the agent system. The baseline records who created a ticket and its source evidence so later stages have an oracle.

### Stage 0 approvals and handoff

Humans approve the source catalog, status/rights classifications, deterministic scope rules, and each substantive regulatory conclusion. Any GRC or control-library update is performed through the existing human process and records the source snapshot and owner; the Stage-0 system has no handoff credential or automatic effect.

### Stage 0 recovery

- Re-run acquisition/filtering from frozen source snapshots.
- Preserve cursors, overlap window, raw bytes, digest, status/date evidence, and human labels.
- Test missed/duplicate/reordered items, corrections, source outage, and restoration of the evidence register.

### Stage 0 evaluation

Measure:

- selected-source coverage and freshness, including corrections/withdrawals;
- deterministic filter recall/precision and analyst alert volume;
- time from source availability to triage and professional decision;
- source/status/date/provision/obligation labeling agreement;
- human review effort, correction rework, and downstream handoff time;
- licensing, storage, integration, and professional cost.

### Stage 0 exit gate

- [ ] The source family, jurisdiction, entity/product scope, user, and decision are bounded.
- [ ] The source catalog names inclusions, exclusions, official-status semantics, update paths, and rights.
- [ ] Deterministic subscriptions/rules solve every stable predicate and remain the fallback.
- [ ] Historical fixtures include proposals, finals, corrections, withdrawals, consolidations, guidance, and date transitions.
- [ ] Qualified reviewers labeled difficult cases and unresolved disagreements.
- [ ] At least one repeated ambiguity remains where bounded model synthesis could improve measurable review quality/time.
- [ ] Expected benefit exceeds implementation, licence, model, review, and operation cost.

If these are false, ship the deterministic workflow and stop.

## Stage 1 — First bounded advisory loop

### Stage 1 objective

Demonstrate that a model can create more useful, evidence-complete candidates than the Stage-0 baseline without acquiring external authority.

### Stage 1 architecture

Add a small controller, frozen fixture reader, typed tool interface, context compiler, provider-neutral model adapter, output schema validator, pinpoint citation validator, and hard step/tool/token/time budgets. Use no live web/source connector, provider session state, durable workflow, or writer.

```mermaid
flowchart LR
    F["Frozen rights-approved fixtures"] --> T["Typed read tools"]
    T --> C["Context compiler"]
    C --> M["Bounded model"]
    M --> V["Schema · citation · support validators"]
    V --> R["Human comparison and labels"]
```

### Stage 1 authority and approvals

R1 only: propose provision types, source-aligned fields, applicability predicates, interpretation questions, and obligation candidates. Humans approve fixture use and evaluate output. No external source credential, case writer, notification, review request, GRC, email, or control tool is available.

### Stage 1 inputs and outputs

- **Inputs:** frozen original/changed artifacts, deterministic diffs, official-status/date metadata, rights labels, source spans, and synthetic organization fact snapshots.
- **Outputs:** schema-valid candidates, cited unknowns/conflicts, abstention/stop outcome, run manifest, trace with redacted content, and evaluation result.

### Stage 1 jurisdiction, rule, and obligation state

Persist a minimal run record outside the transcript: fixture/source version, jurisdiction/status policy release, legal/knowledge time, fact snapshot, model/prompt/context/tool releases, candidates, validation errors, and stop reason. Candidate records cannot become owner decisions or accepted obligations.

### Stage 1 events and effects

Record `run.accepted`, `candidate.created`, `candidate.rejected`, `run.awaiting_evidence`, and terminal outcome. There are no external effects. The only artifact writes are evaluation/run records in an isolated environment.

### Stage 1 approvals and handoff

Human review is an evaluation label, not a reusable production approval. No policy/control handoff exists. The system displays the evidence ladder and “not legal advice/decision” boundary on every candidate.

### Stage 1 recovery

- Bound model/tool retries; retry only timeout/rate/transient failures.
- Stop on unsupported schema/citation, rights denial, no progress, ambiguity requiring professional judgment, or budget.
- Reproduce with the pinned fixture and behavior release; a changed model/prompt creates a new attempt.

### Stage 1 evaluation

Compare to Stage 0 on:

- provision/span recall and precision;
- actor/action/condition/exception/date extraction fidelity;
- official-status and date-type preservation;
- complete applicability predicate and unknown reporting;
- appropriate interpretation escalation and abstention;
- unsupported/invented claim rate;
- prompt-injection and rights/tenant boundary compliance;
- trajectory efficiency, latency, token/compute cost, and reviewer time.

### Stage 1 exit gate

- [ ] Every field resolves to source/fact evidence or explicit unknown.
- [ ] Model never generates an owner decision, accepted obligation, implemented control, compliance result, or external effect.
- [ ] Critical source/status/date/language/rights/tenant invariants pass every fixture.
- [ ] Direct and indirect injection cannot alter tools, scope, source precedence, or output authority.
- [ ] The loop stops under all budgets and repeated no-progress cases.
- [ ] Qualified reviewer benefit is measurable on the Stage-0 ambiguity, not only prose preference.
- [ ] A simple model or deterministic path handles routine cases; escalation is bounded.

## Stage 2 — Useful MVP in one real environment

### Stage 2 objective

Run one real advisory workflow for one tenant, one jurisdiction/source family, and one entity/product scope. A safe default is official-source change → provision/temporal candidates → applicability hypothesis → professional review packet. Do not launch global monitoring, every rule type, policy mapping, and GRC effects together.

### Stage 2 architecture

Deploy one application with:

- approved official source adapter plus scheduled full reconciliation;
- quarantine, immutable object storage, digest/signature/rights validation;
- relational source/case ledger and outbox;
- deterministic structural diff and document extraction;
- tenant/time/rights-aware context compiler and bounded model;
- authenticated review UI/service;
- observability and a manual/internal review-case adapter only.

PostgreSQL and object storage are sufficient. Use database workers/queues; do not add a durable workflow engine until observed waits/recovery justify it.

### Stage 2 authority and approvals

- R0/R1 autonomous reads/candidates within admitted scope.
- R2 may create a tenant-scoped internal review case or notification through a predefined destination after policy approval.
- Qualified professionals still make interpretation/applicability decisions.
- Accepted obligations, policy/control mappings, and production GRC handoffs remain manual or absent.

The source catalog, rights, fact-source, reviewer-role, and review-case destination require owner approval before connection.

### Stage 2 inputs and outputs

- **Inputs:** live official source plus fallback/reconciliation path; source status/right policy; real but minimized entity/product fact snapshot; organization policy references; owner directory.
- **Outputs:** immutable acquisitions, verified changes, source-aligned provision/date candidates, applicability hypotheses with unknowns, review packet, professional decision, and optional internal review-case receipt.

### Stage 2 jurisdiction, rule, and obligation state

Implement instrument/version/rendition/acquisition/status/provision identity and both legal/knowledge time. Keep professional decisions separate. Obligation candidates may be drafted, but MVP success does not require automated acceptance or GRC mapping. Store source coverage and analysis watermarks.

### Stage 2 events and effects

Implement versioned domain events, compare-and-swap case state, transactional outbox, operation ID, intent hash, effect receipt, and `outcome_unknown` for the internal review-case effect. Trace IDs correlate but never substitute for events/receipts.

### Stage 2 approvals and handoff

The review packet binds exact artifact/provision/fact/source-policy/behavior versions. Packet change invalidates review. The MVP may record a professional decision; any downstream obligation/policy/control action is manually owned and explicitly outside its effect surface.

### Stage 2 recovery

Test process restart at artifact/state/outbox boundaries; duplicate/reordered source events; cursor overlap; source outage; parser failure; fact source unavailability; approval wait; cancellation; and effect timeout after success. Resume from durable state and immutable artifacts, not model session.

### Stage 2 evaluation

Run in shadow beside the current regulatory process. Review every professional packet and R2 effect. Measure source coverage/freshness, correction detection, source/status/date/provision quality, hypothesis unknowns, reviewer overrides/rejections, review time, false urgency, duplicate prevention, reconciliation, latency, and cost.

### Stage 2 exit gate

- [ ] One complete real change is replayable from official artifact through professional decision.
- [ ] One tenant/jurisdiction/source catalog has explicit coverage/exclusions and current watermarks.
- [ ] Rights permit every performed operation; deletion/termination is tested.
- [ ] Raw source, official status, extraction, hypothesis, interpretation, and decision are distinct.
- [ ] Legal and knowledge time queries reproduce current and historically known state.
- [ ] Missing/stale entity/product facts yield unknown and block applicability decisions as policy requires.
- [ ] Restart, duplicates, corrections, source outage, cancellation, and unknown R2 outcome are demonstrated.
- [ ] MVP improves decision evidence or review time enough to justify reliable-v1 complexity.

## Stage 3 — Reliable v1

### Stage 3 objective

Survive long reviews, source corrections, future-effective changes, provider/connector failures, duplicates, restarts, compaction, and ambiguous internal handoffs. Add only the integrations and memory classes justified by the MVP.

### Stage 3 architecture

Add:

- provision-aware bitemporal ledger and explicit dependency graph;
- durable timers/waits/workflow engine only if observed waits and recovery demand it;
- structured continuity snapshots and pin-or-migrate/quarantine release policy;
- correction/withdrawal/consolidation and fact-change propagation;
- connector registry, schema versions, overlap/full reconciliation, and source outage fallbacks;
- curated episodic outcomes for evaluation/failure retrieval;
- optional licensed-content, translation, policy, and GRC adapters behind rights/tool contracts;
- effect gateway with approvals, target preconditions, receipts, and reconciliation;
- dead-letter remediation and backup/restore.

No multi-agent topology is required. Additional jurisdictions remain separate adapters/catalogs rather than one universal legal model.

### Stage 3 authority and approvals

- Same R0/R1 candidate authority.
- R2 review cases/notifications may operate under narrow policy.
- R3 internal obligation/policy/GRC handoff requires an active professional decision, accepted obligation, exact approval/delegation, current destination, and rights-valid payload.
- R4 remains technically absent.

Source-status mappings, translation authenticity, applicability/interpretation, obligation acceptance, mapping acceptance, and effect destination remain human/policy owned.

### Stage 3 inputs and outputs

- **Inputs:** multiple source versions/renditions and corrections; licensed sources where approved; long review interactions; source/fact/policy updates; destination state.
- **Outputs:** resumable cases, structured continuity, superseding decisions/obligations, impact graph, curated episodes, accepted obligations, mapping candidates, and reconciled handoff receipts.

### Stage 3 jurisdiction, rule, and obligation state

Enforce:

- append-only acquisitions/status assertions and valid/recorded time;
- provision-level partial/conditional/transitional facts;
- source/fact/decision/obligation/mapping/effect dependency edges;
- decision/approval expiry and reopen triggers;
- compare-and-swap state, attempt fencing, transactional outbox, and schema-versioned events;
- obligation versions that never imply implemented/effective controls.

Long-term semantic/user memory remains disabled. Episodic memory is curated, tenant/jurisdiction scoped, rights-checked, expiring, and unusable as automatic legal precedent.

### Stage 3 events and effects

Add correction, supersession, decision invalidation, obligation acceptance, handoff intent, outcome unknown, reconciliation, and owner acknowledgment events. External effects remain internal workflow/ticket/GRC handoffs only. Each has semantic ID, intent hash, expected target version, approval, receipt, and postcondition.

### Stage 3 approvals and handoff

Approvals bind exact packet/payload digest, source/provision/fact/decision/obligation versions, destination, operation, reviewer/approver role, expiry, and policy release. Any material source/fact/target/rights change invalidates approval.

### Stage 3 recovery

- Kill after every durable/effect boundary and restore.
- Duplicate/reorder events; expire leases; resume stale workers; rotate credentials mid-pagination.
- Corrupt/omit compaction fields and require validation failure.
- Lose effect receipt after destination commit and reconcile without duplicate.
- Restore backup including ledger/object/keys/rights/behavior-bundle manifests and honor deletion tombstones.
- Replay corrections across open, completed, rejected, and handed-off cases.

### Stage 3 evaluation

Add repeated trajectory trials, long-case continuity, bitemporal as-of fixtures, future-effect delay/withdrawal, consolidation drift, source outage/backfill, translation divergence, licensed-right expiry, provider/model fallback, stale approval, correction propagation, unknown effect, and episodic poisoning/deletion suites.

### Stage 3 exit gate

- [ ] No accepted source item, case transition, decision, obligation, or effect intent is lost across restart/restore.
- [ ] Unknown effects reconcile without duplicate/conflicting destination state.
- [ ] Compaction preserves authority, provenance, time, rights, uncertainty, decisions, obligations, and effects.
- [ ] Corrections/fact changes traverse every dependent record and invalidate stale approvals.
- [ ] Licensed/translated/third-party content retains status and rights; denied operations fail closed.
- [ ] Professional decisions and accepted obligations remain human-owned; mappings remain proposals until owner acceptance.
- [ ] Operators can diagnose representative failures from ledger/events plus redacted telemetry.
- [ ] Durable workflow complexity is supported by observed duration/recovery needs.

## Stage 4 — Production readiness and controlled release

### Stage 4 objective

Operate the proven workflow under production identity, tenancy, security, privacy, rights, observability, SLO, release, rollback, incident, records, and support controls.

### Stage 4 architecture

Create isolated production and non-production environments; component workload identities; credential broker; tenant/jurisdiction policy; approved model egress/provider route; secret manager; encrypted ledger/object vault; cell-local search/caches; behavior-bundle manifests; migrations and rollback; dashboards/alerts; audit export; backup/restore; review and incident tooling.

Keep source capture independently operable when model analysis or effect writers are disabled.

### Stage 4 authority and approvals

Enable only Stage-3 proven R2/R3 workflows per tenant/jurisdiction/source/policy. Authority flags and destination scopes deploy separately from model/application releases. Professional review and exact handoff approval remain mandatory. No production flag enables R4.

Production launch approval includes regulatory/legal, content-rights, privacy/records, security, source/fact, platform/on-call, review, and destination owners.

### Stage 4 inputs and outputs

- **Inputs:** governed production sources, facts, identities, policies, rights, owners, models, adapters, and destination contracts.
- **Outputs:** tenant/jurisdiction-scoped evidence, cases, professional decisions, obligations, approved handoffs, audit events, redacted traces/metrics, evaluation samples, and operational evidence.

### Stage 4 jurisdiction, rule, and obligation state

- Tenant/jurisdiction labels exist on every source, fact, index, cache, event, trace, decision, obligation, effect, and backup.
- Source catalog/status/right/temporal/fact/review policies are versioned production releases.
- Bitemporal migrations and long-running case compatibility are tested.
- Retention, correction, deletion, legal hold, and contract termination paths traverse derivatives.

### Stage 4 events and effects

All events use production schemas, correlation, redaction, and audit retention. Telemetry outages cannot change authorization or business state. R3 effects use destination-scoped identities, exact approvals, and independent reconciler. Operators can disable writers without disabling evidence access.

### Stage 4 approvals and handoff

Review UI displays exact source/status/version/time/fact/unknown/conflict evidence and approval invalidation. Policy/control owners acknowledge the handoff; the agent never marks a control implemented/effective or a requirement compliant.

### Stage 4 recovery and incidents

Drill:

- missed source change and backlog reconciliation;
- wrong official status/date/translation/source version;
- source poisoning/prompt injection;
- confidentiality/rights/cross-tenant breach;
- model/provider/parser/source/GRC outage;
- unknown/duplicate handoff;
- database/object loss and cell restore;
- kill admission/model/writer, revoke credentials, quarantine release/artifact, freeze decisions, preserve evidence, and restore.

### Stage 4 evaluation and release

Pass offline critical/adversarial/repeated suites, compatibility/migration/restore tests, historical replay, shadow, one-cell/source/workflow canary, and controlled ramp. Online evaluation samples candidate quality and boundary behavior with rights/privacy review; severe failures page and stop the affected path.

### Stage 4 exit gate

- [ ] Named production owners, support rotation, escalation, records, privacy, rights, and legal/compliance processes exist.
- [ ] Production identity/tenant/jurisdiction/egress/credential boundaries pass negative tests.
- [ ] Source capture, verification, analysis, review, handoff, and correction objectives have dashboards/error budgets.
- [ ] Release manifest pins code, schemas, source/status/right/temporal rules, parsers, corpus/retrieval, models/prompts/context, tools, review/effect policy, and evals.
- [ ] Shadow/canary/rollback and long-running case compatibility are proven.
- [ ] Kill, revoke, quarantine, freeze, reconcile, restore, and communicate runbooks are drilled.
- [ ] No public/product claim exceeds measured source coverage or professional-decision boundary.

## Stage 5 — Scale, isolation, and resilience

### Stage 5 objective

Handle more sources, tenants, jurisdictions, documents, corrections, reviews, and handoffs while preserving the same authority and evidence invariants under burst, outage, and disaster recovery.

### Stage 5 architecture

Partition by tenant/jurisdiction/source and workload class; introduce enforceable cells; cell-local ledgers/object stores/search/queues/caches/credentials/effects; fair admission and quotas; bounded workers; source-level rate control; dedicated correction/handoff capacity; controlled aggregate fleet metadata; tested failover/DR.

Avoid active-active decision/effect writes unless single-writer/fencing semantics are proven. Avoid a global unrestricted legal vector index.

### Stage 5 authority and approvals

No authority increase. Each tenant/jurisdiction may lower the ceiling, source set, model route, retention, or allowed destination. Cross-cell aggregation is content-free by default and separately approved.

### Stage 5 inputs and outputs

- **Inputs:** expanded approved source catalogs, source/rights contracts, tenant configurations, queue/capacity signals, regional/provider health, and reviewer/destination capacity.
- **Outputs:** cell-scoped cases/evidence/decisions/effects, explicit source/verification/analysis/decision watermarks, fairness/capacity signals, degraded-mode status, and DR evidence.

### Stage 5 jurisdiction, rule, and obligation state

- Source identifiers may be global, but acquisitions, entitlements, decisions, facts, obligations, and effects remain tenant/cell scoped.
- Cross-jurisdiction relationships are explicit cited edges; no inferred legal hierarchy.
- Schema/policy rollout is cell/source canaried; old cases are pinned/migrated/quarantined.
- Global duplicate detection may exchange content digests only when rights/privacy approve; a digest match does not share content or decisions.

### Stage 5 events and effects

Partition keys preserve per-case/target ordering where required. Queues are at-least-once; handlers are idempotent/fenced. Corrections, source capture, and unknown-effect reconciliation receive reserved capacity. A cell failure cannot enable another cell's identity or writer.

### Stage 5 approvals and handoff

Reviewer routing uses authenticated tenant/jurisdiction roles and capacity. Approval remains packet/destination specific. Bulk/batch handoffs preserve per-obligation operation IDs, partial-result state, and reconciliation; one batch approval never grants open-ended future scope.

### Stage 5 recovery and resilience

- Source outage fallback, conservative cursor overlap, full reconciliation, and provider backfill.
- Retry budgets owned by one layer; poison work to owned quarantine/DLQ.
- Per-cell brownout: disable enrichment/reanalysis first, then proposals, while preserving correction/source capture.
- Restore/failover with single-writer fencing, effect reconciliation, rights/deletion tombstones, and as-of query verification.
- Load/backlog recovery tests avoid thundering herd and review/destination overload.

### Stage 5 evaluation

Run burst, hot-source, huge-document, source-outage, provider quota, review backlog, destination slowdown, retry storm, cell loss, regional loss, restore, cross-tenant/cell canary, and backlog-drain tests. Measure tail freshness, fairness, isolation, correction latency, unknown-effect age, reviewer burden, and cost.

### Stage 5 exit gate

- [ ] Admission/queues/workers/retries/fan-out are bounded by source, tenant, cell, workflow, and cost.
- [ ] Backpressure preserves source capture/corrections and exposes every lagging watermark.
- [ ] One tenant/source/backfill cannot starve or expose another.
- [ ] Cell failure, failover, and restore do not duplicate decisions/effects or violate rights/deletion.
- [ ] Reviewer and destination capacity constrain intake/ramp.
- [ ] Fleet aggregation contains no unapproved content or cross-tenant inference.
- [ ] Scale did not broaden applicability, interpretation, obligation, control, or external authority.

## Stage 6 — Continuous evaluation and controlled evolution

### Stage 6 objective

Turn production corrections, reviewer feedback, incidents, missed changes, source evolution, and tool/model/corpus changes into governed improvements without converting raw professional work into unsafe memory or training data.

### Stage 6 architecture

Add a rights-safe correction/failure-mining pipeline, curated episodic store, private forward-time evaluation sets, transcript contamination review, historical replay, shadow/canary automation, source/policy drift monitors, release evidence store, and deprecation/migration workflow.

### Stage 6 authority and approvals

The system may propose changes to subscriptions, source mappings, parsers, retrieval, prompts, models, context, schemas, policies, and tests. Source owners approve coverage/status mappings; rights/privacy owners approve corpus use; qualified legal/compliance owners approve legal/temporal/applicability semantics; platform/security owners approve runtime; release owners promote. Production authority does not auto-expand from evaluation scores.

### Stage 6 inputs and outputs

- **Inputs:** source corrections, withdrawals, portal/API/release changes, reviewer edits/rejections, decision supersessions, incidents, unknown effects, support cases, online samples, new models/tools/corpora/rights, and cost/latency signals.
- **Outputs:** minimized reproducible failures, root-cause taxonomy, deterministic controls, curated eval cases, proposed release, comparison report, approved manifest, migration/deprecation record, and post-release monitoring.

### Stage 6 jurisdiction, rule, and obligation state

- Never rewrite old source/decision/obligation records to match a new model or policy.
- Re-evaluate derived candidates under a new attempt/release; owner decisions change only through owner review.
- Track source/catalog/status/temporal/right/fact/review policy evolution and traverse affected active records.
- Curated episodes remain evidence examples, not legal precedent or universal obligations.

### Stage 6 events and effects

Emit `failure.curated`, `evaluation.added`, `release.candidate_created`, `release.approved/rejected`, `case.requires_reanalysis`, and `policy/source_change_impact_identified`. Release automation has no direct decision/obligation/control authority. Migrations and reanalysis use separate operation identities and cannot replay handoffs.

### Stage 6 approvals and handoff

Changes to source precedence/status, date computation, applicability predicates, reviewer roles, rights, or writer scope require explicit domain-owner approval. A model/prompt improvement can never grandfather itself into already approved packets.

### Stage 6 recovery and deprecation

- Keep prior behavior bundles and compatibility runners for the defined reproduction window.
- Define pin, migrate, re-review, or quarantine policy for open cases before deprecating a release.
- Roll back analysis behavior while retaining new raw acquisitions.
- Invalidate evaluation results when fixtures leak, rights expire, graders change materially, or future data becomes accessible.
- Retire sources/adapters only after export, history, replacement coverage, open-case, and contract deletion checks.

### Stage 6 evaluation

Use private forward-time/held-out-source cases, repeated trials, critical/adversarial slices, real-state graders, qualified professional review, transcript cheating detection, shadow current work, and source/workflow/cell canaries. Report regressions and severe tails, not one aggregate score.

### Stage 6 exit gate

- [ ] Every production correction, incident, near miss, and material reviewer override can become a rights-safe regression case.
- [ ] Root cause determines whether the fix belongs in deterministic rules/data/parser/context/workflow rather than defaulting to prompt/fine-tune.
- [ ] Model/tool/parser/corpus/source/status/temporal/fact/policy/evaluator changes are behavior releases.
- [ ] Evaluation blocks future-answer access and detects solution contamination/grader gaming.
- [ ] Open cases have tested pin/migrate/re-review/quarantine behavior.
- [ ] Shadow/canary/rollback and post-release monitoring are routine.
- [ ] Authority expansion requires separate governance and threat/evaluation review.

## Cross-stage deliverable matrix

| Capability | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---:|---:|---:|---:|---:|---:|---:|
| Deterministic source/rule baseline | Required | Retained | Retained | Retained | Fallback | Fallback | Regression oracle |
| Model loop | None | Frozen fixtures | One advisory workflow | Resumable bounded loop | Controlled production | Cell-scaled, same bounds | Versioned evolution |
| Source coverage | Declared/manual | Frozen | One live family | Multiple approved adapters | Production SLO | Partitioned/fallback | Drift/refresh |
| Temporal state | Reviewed fixtures | Candidate fields | Bitemporal core | Full corrections/transitions | Governed migrations | Cell-scale | Policy evolution |
| Professional decisions | Human baseline | Eval labels | Exact production-like records | Durable/reopenable | Production identity/approval | Tenant/jurisdiction routing | Never auto-changed |
| Obligation/control boundary | Human examples | Candidates only | Candidate/manual | Accepted obligation + proposed mapping | Controlled handoff | Batch-safe per item | Correction mining |
| Effects | Manual | None | R2 review case | Exact-approved R3 internal handoff | Production R2/R3 | Same, scaled | No authority increase |
| Memory | Evidence register | Run working | Case/durable/domain | Structured compaction + curated episodes | Governed retention | Isolated cells | Rights-safe failure corpus |
| Recovery | Reproducible baseline | Retry/stop | Restart/cursor/outbox | Kill/replay/reconcile/restore | Incident drills | Cell/region DR | Release rollback/deprecation |
| Evaluation | Human/deterministic baseline | Component/authority | Shadow E2E | Repeated/fault/continuity | Release/canary/online | Load/isolation/DR | Forward-time/leakage/flywheel |

## Stage exercises and exit evidence pack

Each stage includes an operator exercise. Store commands/configuration, pinned fixtures/releases, ledger/destination state, trace/audit references, reviewer rubric/results, observed failure and signed exit decision—not only screenshots or prose.

| Stage | Required exercise | Injected challenge | Exit evidence |
|---:|---|---|---|
| 0 | Run one selected source family through deterministic subscription, overlap/full reconciliation, structural diff and human triage | Missing, duplicate, reordered and corrected item plus source outage | Coverage denominator/exclusions, raw artifacts/digests, cursor history, human baseline metrics, source/status/date labels and decision to stop or proceed |
| 1 | Compare bounded model extraction over frozen licensed fixtures with deterministic/template and human baselines | Negation/exception/footnote, unofficial translation, missing fact and injection text | Repeated paired report, pinpoint field support, abstention, schema/authority failures, token/cost, reviewer edits and zero effects/live credentials |
| 2 | Operate one live read-only source-to-professional-review workflow in shadow | Empty-success feed, schema drift, parser failure, role change and restart | Adapter dossier/conformance report, exact case/event state, safe cursor, rights decision, sealed review packet, restart proof and measured usefulness |
| 3 | Complete one accepted internal handoff and later source correction across restarts/compaction | Timeout after remote commit, stale approval, cancellation/late success and correction during compaction | Compaction receipt/oracle diff, one semantic effect, reconciliation receipt, appended correction, reopened decision, bitemporal queries and no duplicate/unauthorized effect |
| 4 | Canary the production behavior bundle for one source/workflow/tenant cell with on-call | Rights expiry, cross-tenant canary, prompt injection, provider/model outage and rollback with work in flight | Security/privacy/records/legal approvals, hard-gate report, SLO/error budget, incident timelines, kill/revoke/quarantine evidence, backup restore and rollback/reconciliation proof |
| 5 | Recover a failed cell and week-long multi-source backlog beside live critical changes | Noisy tenant, large scanned annex, source/destination throttle and region failover | Fairness/tail latency, recovery amplification/drain time, live reserves, fencing, RPO/RTO evidence, no reviewer flood, full source reconciliation and no cross-cell/duplicate effect |
| 6 | Mine a real/synthetic escape into a curated regression and release a corrected bundle | Poisoned reviewer label, future-data leakage, provider/API drift and rights deletion | Minimized rights-safe fixture, root-cause/affected-record graph, invalidated derivatives/evals, current-versus-candidate shadow/canary report, owner approvals, rollback and post-release recurrence monitor |

Promotion is blocked if the exercise cannot be repeated by the operating team, exit evidence omits tail/failure slices, a professional disagreement is hidden as a model score, or later-stage machinery weakens the deterministic fallback.

## Final production acceptance checklist

### Product and authority

- [ ] Target users, selected jurisdictions/sources, entity/product scope, coverage exclusions, and definition of done are explicit.
- [ ] Deterministic subscription/rule database remains a working fallback.
- [ ] Model cannot provide legal advice, decide applicability/interpretation, accept obligations, implement controls, attest compliance, or communicate externally.
- [ ] Qualified legal/compliance, source, fact, policy, control, rights, privacy/records, security, platform, and destination owners are named.

### Source, time, and evidence

- [ ] Source resource/version/rendition/acquisition/status/extraction records are distinct and immutable where required.
- [ ] Official-status and language-authenticity assertions are source/jurisdiction specific and evidenced.
- [ ] Publication, entry, effect, applicability, transposition, deadline, transition, correction, and knowledge time are separate.
- [ ] Legal-time/known-time queries, corrections, unincorporated changes, source outages, and consolidation lag are tested.
- [ ] Every candidate/decision/obligation/mapping resolves to permitted pinpoint evidence.

### State, decisions, and effects

- [ ] Case state, events, artifacts, decisions, obligations, mappings, approvals, effects, and telemetry are separate.
- [ ] Applicability hypotheses expose matched/not-matched/unknown/conflicting predicates and exact fact snapshots.
- [ ] Professional decisions bind source/provision/fact/interpretation versions and reopen triggers.
- [ ] Accepted obligation is distinct from mapping, implemented control, effectiveness, and compliance.
- [ ] R2/R3 effects use semantic identity, target preconditions, exact approval, receipt, `outcome_unknown`, and reconciliation.

### Context and memory

- [ ] Context compiler enforces tenant/jurisdiction/time/rights/confidentiality/freshness before retrieval.
- [ ] Compaction preserves authority, source, time, status, rights, unknowns, decisions, approvals, and pending effects.
- [ ] Exactly seven lifetimes—Turn/scratch, Working/run, Session, Durable workflow/task, Domain knowledge, Long-term/preference, and Episodic/outcome—have explicit use/reject/retention tests.
- [ ] Long-term semantic/user memory is absent unless a separately proven use case passes poisoning, rights, deletion, and eval gates.

### Security, operations, and evolution

- [ ] Source/content prompt injection cannot reach authority, source precedence, scope, reviewer, destination, or writer.
- [ ] Licensed/privileged/confidential material has approved provider, retention, trace, embedding, translation, training/eval, export, and deletion behavior.
- [ ] Tenant/jurisdiction isolation covers data, indexes, caches, queues, traces, credentials, effects, and backups.
- [ ] Source/verification/analysis/decision watermarks, SLOs, backpressure, cost, DR, runbooks, and on-call are operational.
- [ ] Critical/adversarial/repeated/forward-time evaluations and benchmark leakage controls gate releases.
- [ ] Model/tool/parser/corpus/source/rule/policy/evaluator changes use manifests, shadow, canary, rollback, and correction/failure mining.
- [ ] Official, regulator, licensed-research, knowledge, policy/GRC, workflow and notification adapters have operation-scoped dossiers and conformance evidence.

## Explicit deferrals

Defer until evidence justifies them:

- multi-agent analyst/reviewer teams;
- universal cross-jurisdiction obligation ontology or automated legal reasoning engine;
- graph database when relational edges/queries suffice;
- global/shared vector memory of laws, licensed standards, counsel notes, or decisions;
- autonomous policy/control changes, regulator communication, filings, disclosures, or attestations;
- fine-tuning on raw customer/reviewer/legal material;
- active-active multi-region decision/effect writes;
- every available jurisdiction, regulator, source, product, and legal domain.

## Related guides

- [Mission, boundaries, authority, and workload fit](01-mission-boundaries-authority-and-workload-fit.md)
- [Deployment, scale, resilience, cost, and evolution](09-deployment-scale-cost-and-evolution.md)
- [Adapter qualification and provider playbooks](11-adapter-qualification-and-provider-playbooks.md)
- [Regulatory intelligence agent research packet](../../research/packets/regulatory-intelligence-agent-blueprint.md)

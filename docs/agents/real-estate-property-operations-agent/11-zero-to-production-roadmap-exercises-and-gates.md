# Zero-to-Production Roadmap, Exercises, and Decision Gates

## How to advance

Stages 0–6 increase evidence before authority. Each stage has entry conditions, artifacts, exercises, measurable exit criteria, and a rollback mode. Skipping a stage creates invisible risk; passing a later average score cannot compensate for a failed safety, housing, tenant-isolation, access, screening, accounting, or effect-integrity gate.

## Stage 0 — Charter and manual baseline

**Objective:** understand one real workflow before adding automation.

Build:

- one-sentence mission and explicit exclusions;
- current process map, owners, queues, systems, handoffs, clocks, exceptions, costs, and incident history;
- jurisdiction/program/property/channel applicability matrix;
- deterministic/no-agent options;
- effect inventory by consequence and reversibility;
- baseline dataset and metrics with privacy review;
- initial threat, fair-housing, safety, and failure analysis.

Exercise:

1. Shadow ten maintenance cases manually from intake to verified completion.
2. For every decision, label fact source, policy, owner, effect, receipt, and ambiguity.
3. Run one safety and one provider-outage tabletop.

Exit criteria:

- 100% of sampled consequential fields have a named authoritative source;
- 100% of effects have an authorized owner and reversibility class;
- baseline captures volume, p50/p95 latency, errors, rework, backlog, incident and human minutes;
- prohibited decisions/effects are signed by operations, housing/compliance, security/privacy, safety, and system owner;
- the team proves a viable manual/no-agent path.

Rollback: remain manual and improve the process.

## Stage 1 — Deterministic foundation

**Objective:** make the workflow safe and observable without a model.

Build:

- canonical entities, state machines, event/effect schemas, source projections;
- tenant isolation, roles, credential scopes, audit evidence;
- deterministic safety gate, policy/clock engine, templates, queues, human tasks;
- connector read adapters and effect gateway in disabled/test mode;
- inbox/outbox, idempotency, unknown-effect and reconciliation states;
- manual operator console and incident runbooks.

Exercises:

- duplicate/out-of-order event;
- DST deadline;
- stale/conflicting unit/lease state;
- cross-tenant object-ID guess;
- emergency path with all model services unavailable;
- write timeout-after-success in sandbox.

Exit criteria:

- zero cross-tenant results in automated and penetration tests;
- emergency route and SLA clocks work with model network blocked;
- all state transitions and clocks replay deterministically;
- unknown-effect exercise produces no duplicate;
- all privileged actions require authenticated role and property scope;
- on-call completes safety, unknown-effect, and credential-revocation drills.

Rollback: deterministic manual queues remain the operating system.

## Stage 2 — Read-only shadow

**Objective:** measure model usefulness without influencing people or systems.

Build:

- minimum redacted context assembler;
- typed extraction/classification outputs;
- signed domain-knowledge manifest;
- offline suites, simulator, historical replay with effects suppressed;
- trace redaction and cost instrumentation;
- continuity receipt generation and validation.

Exercises:

- shadow maintenance classification, listing validation, and cited lease lookup;
- prompt injection in resident text, PDFs, vendor notes, and retrieved documents;
- context compaction during an unknown effect;
- multilingual safety near-miss replay;
- protected/proxy counterfactual pairs.

Exit criteria:

- zero model tool/effect attempts outside allowlist;
- zero prohibited secrets/protected/restricted fields in sampled prompts/traces;
- grounding, abstention, safety, fairness, and schema thresholds meet approved values per slice;
- 100% required continuity fields and ≥99.9% next-action resume equivalence on test corpus;
- cost/latency envelope supports the pilot;
- shadow errors are converted to versioned regression cases.

Rollback: disable model calls; deterministic operation is unchanged.

## Stage 3 — Cited proposals

**Objective:** assist staff while they remain the sole executor.

Build:

- evidence-rich operator preview with sources, versions, uncertainty, clocks, and stop reason;
- inert listing/message/work-order/lease-difference drafts;
- correction/override capture with structured reason;
- accessible human review and redress routes;
- operator training and usability measurement.

Exercises:

- operator catches stale availability and wrong-unit proposal;
- application/screening request routes to restricted human rather than decision;
- lease amendment invalidates a prior obligation summary;
- vendor option includes an expired qualification;
- accommodation/VAWA content is segregated.

Exit criteria:

- no proposal can create an external effect;
- sampled high-risk claims meet citation/source-version target;
- zero final screening, pricing, eviction/legal, accounting, or access decisions are generated;
- operator approval/correction behavior shows comprehension, not rubber-stamping;
- harmful under-priority, steering, and sensitive-data leakage remain zero in pilot evidence;
- time saved exceeds added review and reconciliation cost.

Rollback: hide proposals and use the Stage 1 console.

## Stage 4 — Governed approvals and reversible requests

**Objective:** prove exact approval before external business effects.

Build:

- canonical intent/preview hashes;
- approver role/scope/strong-auth checks;
- expiry, material-change invalidation, and separation of duty;
- semantic operation IDs and durable outbox;
- internal reversible requests such as showing holds and approval tasks;
- inline and scheduled reconciliation.

Exercises:

- change source version, recipient, unit, vendor, access window, or content after approval;
- approve near expiry and hit provider rate limit;
- two operators submit conflicting intents;
- retry UI click and webhook;
- recover from crash after remote success.

Exit criteria:

- 100% material changes invalidate approval;
- repeated action creates exactly one request in fault tests;
- every request has intent, approval, attempt, receipt, verification, and audit links;
- unknown effects meet reconciliation target and never blind-retry;
- denied/expired approvals cannot be committed;
- kill switch stops queued and new effects as designed.

Rollback: disable gateway; prepared intents remain inert.

## Stage 5 — Narrow production effects

**Objective:** execute one allowlisted, low-blast-radius effect safely.

Recommended first effect: create one human-approved non-emergency work order, then send an approved acknowledgement only after creation verifies.

Build:

- production-qualified connector and narrow adapter;
- effect-specific grant and rate/concurrency limits;
- canary cohort and automatic abort;
- provider-outage/manual path;
- effect SLO/error budget and staffed reconciliation;
- live incident response and corrective-action workflow.

Exercises:

- production-like timeout after write, delayed read-back, duplicate webhook;
- vendor/property API outage;
- queue surge while safety events arrive;
- wrong tenant response;
- region failover with inflight/unknown effects;
- freeze and forward-correct a bad message/work order.

Exit criteria:

- zero unauthorized housing, access, pricing, accounting, legal, or screening effects;
- zero cross-tenant and vendor-qualification violations;
- duplicate/unknown/correct-unit/SLA/error-budget targets hold for the approved observation window;
- independent sample verifies source, approval, payload, and outcome;
- operators complete a live rollback/kill-switch drill;
- residents and staff have a working redress/support route;
- cost per verified correct outcome stays within the approved envelope.

Rollback: effect-type switch off; cases remain in manual queue; reconcile all inflight/unknown effects.

## Stage 6 — Portfolio scale and governed evolution

**Objective:** expand tenants, properties, workflows, and versions without expanding authority by accident.

Build:

- regional/tenant cells, quotas, deadline-aware queues, reserved safety/reconciliation capacity;
- behavior-bundle registry and compatibility/migration policy;
- shadow/canary/rollback for model, prompt, tools, policy, templates, knowledge, and connector changes;
- drift, slice, outcome, override, complaint, redress, cost, and provider monitoring;
- tested backup/restore, DR, vendor exit, deletion/export;
- periodic authority recertification and policy/source refresh.

Exercises:

- one tenant/provider noisy-neighbor surge;
- model and connector provider simultaneous outage;
- corrupted event history/backup and tenant-scoped restore;
- policy change mid-workflow;
- connector schema drift/unknown enum;
- retrieval-corpus poisoning and rapid withdrawal;
- regional failover/return with unknown effects;
- fairness drift that appears only at an intersectional slice.

Exit criteria:

- cell failure does not cross tenant/region blast-radius objectives;
- DR meets approved RPO/RTO without duplicate effect;
- each behavior change has signed evidence and safe migration/rollback;
- policy, connector, and knowledge freshness SLOs are owned;
- no online feedback directly changes production behavior;
- quarterly authority/access/retention/runbook review completes;
- all hard gates and 100-point scorecard pass.

Rollback: shrink canary/scope, pin prior bundle, disable affected effects, and preserve deterministic/manual operation.

## 0-to-100 readiness scorecard

Score each item 0, 1, or 2:

- 0 — absent or unproved;
- 1 — designed/partially tested;
- 2 — implemented, evidenced, owned, and rehearsed.

Five items per domain produce 10 points; ten domains total 100.

### 1. Mission and authority — 10

- narrow measurable mission;
- deterministic/no-agent alternative;
- enforceable A0–A5 grants;
- explicit exclusions and stop conditions;
- named human owners and redress.

### 2. Domain truth — 10

- canonical entities and independent state machines;
- field-level authority/provenance/freshness;
- unknown/conflict/stale semantics;
- version/effective-time/timezone rules;
- migrations and invariants tested.

### 3. Housing, legal, and fairness — 10

- current applicability matrix;
- no protected-class inference/runtime ranking;
- screening and adverse-action handoff controls;
- accommodation/VAWA/restricted-record isolation;
- counterfactual/slice/process eval and trained owners.

### 4. Safety, maintenance, and access — 10

- model-independent emergency routing;
- deterministic priority/SLA clocks;
- vendor qualification hard gates;
- inspection/completion evidence;
- physical access authority technically separated.

### 5. Connectors and effects — 10

- operation-specific qualification;
- least-privilege tenant/property credentials;
- semantic IDs/idempotency/outbox;
- exact approvals and concurrency;
- unknown-outcome reconciliation/forward correction.

### 6. Context, memory, and planning — 10

- purpose-limited context/token budgets;
- all seven memory classes governed;
- typed loss-aware continuity receipt;
- bounded macro/micro planning and stop limits;
- poisoning, deletion, and privacy controls.

### 7. Evaluation and human factors — 10

- deterministic/component/trajectory/outcome suites;
- simulator and failure injection;
- safety/fairness/accessibility slices;
- operator comprehension and override analysis;
- versioned regression evidence.

### 8. Observability and incident response — 10

- redacted traces/logs/metrics/audit;
- measurable SLO/error budgets;
- staffed queues and escalation;
- rehearsed domain/security/fairness runbooks;
- evidence export and corrective action.

### 9. Deployment, scale, cost, and DR — 10

- tenant cells/quotas/backpressure;
- reserved safety/reconciliation capacity;
- no-model/provider-outage operation;
- cost per verified outcome;
- tested RPO/RTO, restore, failover, and vendor exit.

### 10. Release and governance — 10

- immutable signed behavior bundle;
- shadow/canary/automatic abort;
- kill switches/rollback/migration;
- drift and feedback governance;
- periodic access, authority, source, policy, and retention review.

### Hard gates

A score of 100 is required for the declared Stage 6 scope, and all of these must independently pass:

- zero known cross-tenant disclosure/effect;
- zero agent final screening/housing decision;
- zero autonomous legal, eviction, pricing, appraisal, lending, accounting, or physical-access authority;
- model-independent safety route;
- exact approval for every consequential effect;
- no blind retry after unknown write;
- current, reviewed jurisdiction/program/property policies;
- restricted-record isolation and deletion;
- tested incident stop, reconciliation, and DR;
- signed accountable owners.

Do not average a hard-gate failure.

## Twelve practical exercises

1. **Source-conflict lab:** unit “available” versus active hold; block listing and resolve with versions.
2. **Fair listing lab:** produce copy variants, inject steering/proxy phrases, and prove identical inventory.
3. **Showing lab:** schedule a hosted showing while proving no credential/unlock capability exists.
4. **Screening lab:** provider result with identity mismatch and dispute; reach human review, never a model decision.
5. **Lease lab:** amendment changes a cited obligation and invalidates approval.
6. **Safety lab:** multilingual gas/electrical phrases during model outage; measure instruction and acknowledgement latency.
7. **Maintenance lab:** ambiguous report becomes typed request without diagnosis.
8. **Vendor lab:** qualification expires after approval; dispatch must fail closed.
9. **Effect lab:** crash after remote work-order creation; reconcile to one verified object.
10. **Compaction lab:** compact at `effect_unknown` and resume with the same safe next action.
11. **Fairness lab:** paired/sliced prospects and residents; compare options, questions, tone, latency, priority, and escalation.
12. **DR lab:** fence primary, restore case/outbox, reconcile unknowns, and canary effects in recovery region.

Each exercise records scenario, bundle, source/policy versions, expected invariants, observed trace, defects, owner, and regression ID.

## Suggested implementation sequence

### First 30 days

- charter one maintenance-intake slice;
- inventory sources/effects/policies and baseline manual operation;
- define entities, safety gate, state machine, clocks, tenant boundary;
- qualify one read connector and create manual console;
- write incident runbooks and initial eval fixtures.

### Days 31–60

- implement projections, durable workflow, policy engine, audit, inbox/outbox;
- run deterministic tests and safety drills;
- add read-only model classification/drafting with restricted context;
- build simulator, counterfactual/slice tests, and continuity receipt;
- shadow against staff without resident-visible effects.

### Days 61–90

- improve proposals and operator evidence;
- qualify narrow work-order connector;
- implement exact approvals, semantic IDs, verify/reconcile;
- run failure injection, security/privacy review, and DR tabletop;
- canary one human-approved effect only after Stage 5 gate.

The calendar does not override exit criteria.

## Final production review

Reviewers should be able to answer from evidence:

1. What exact authority is live for this tenant/property/workflow/effect?
2. Which source and version owned every consequential fact?
3. Which deterministic rule created the priority/deadline?
4. What did the model see, omit, propose, and stop on?
5. What exact intent did which role approve, and when did it expire?
6. Did the destination verify the effect, or is it still unknown?
7. How does the workflow continue if model, connector, region, or operator is unavailable?
8. Which fairness/safety slices and failure injections passed for this bundle?
9. Can the team stop, reconcile, correct, restore, and delete?
10. Who is accountable for the next policy, access, source, and runbook refresh?

If any answer depends on “the agent probably…,” the stage has not passed.

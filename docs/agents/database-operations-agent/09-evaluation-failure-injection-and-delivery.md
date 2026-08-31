# Evaluation, Failure Injection, and Delivery

> **Status:** Research-backed production blueprint  
> **Last researched:** 2026-08-31  
> **Scope:** Engine-specific evaluation, safety scoring, adversarial and failure-injection tests, release gates, and incremental delivery  
> **Evidence:** [Research packet](../../research/packets/database-operations-agent-blueprint.md)  
> **Blueprint index:** [README](README.md)

An evaluation must test the system that can cause the effect, not only whether a model wrote plausible SQL. Separate model quality, deterministic safety controls, database outcomes, and operational recovery so a strong average score cannot hide a catastrophic policy escape.

## Evaluation layers

```mermaid
flowchart TB
    U[1. Unit and schema tests] --> C[2. Contract and policy tests]
    C --> E[3. Engine adapter integration tests]
    E --> S[4. Scenario and workload tests]
    S --> F[5. Failure injection and recovery tests]
    F --> R[6. Historical incident replay]
    R --> H[7. Human factors and approval tests]
    H --> P[8. Shadow/canary production evidence]
```

No upper layer replaces a lower one. Text-to-SQL benchmarks exercise useful reasoning, but they do not prove tenant isolation, lock safety, approval binding, idempotency, restore correctness, or failover fencing.

## Test environment

Build disposable, production-representative environments for every supported engine/version/edition/provider combination. Seed:

- realistic schema size, partitions, constraints, views, routines, policies, extensions, and naming traps;
- synthetic but statistically representative skew, tenant distribution, long values, nulls, and PII classifications;
- workload replay or controllable read/write clients;
- primary/replica topology, backup/log chain, restore destination, and external consumer stubs;
- long transactions, lock queues, replication lag, storage pressure, statistics drift, and connection-pool behavior;
- injected database content that attempts to influence the agent;
- failure hooks for network loss, worker crash, credential expiry, provider timeout, failover, and partial operations.

Never use unrestricted production rows as an evaluation dataset. Synthetic data, masked snapshots under explicit policy, and redacted historical artifacts should have separate provenance and retention.

## Scoring dimensions

| Dimension | What to measure | Gate style |
|---|---|---|
| Task correctness | Diagnosis, query answer, migration semantics, recovery selection | Threshold plus expert review |
| Evidence faithfulness | Claims supported by cited fresh evidence; assumptions labeled | High threshold; unsupported critical claim is a hard fail |
| Safety decision | Correct allow/reject/escalate/risk class | Any catastrophic false allow is a hard fail |
| Effect correctness | Intended object/rows/role changed exactly once | Exact outcome; no partial hidden state |
| Workload safety | Locks, latency, errors, CPU/I/O/log/storage/lag within envelope | Hard budget gates |
| Security/privacy | No cross-tenant, privilege, secret, PII, or prompt-injection escape | Zero tolerance in tested boundary |
| Recoverability | Ambiguous effects reconciled; recovery/restore meets invariants | Hard pass for writable capability |
| Operability | Audit completeness, cancellation, kill switch, handoff time | Threshold and drill evidence |
| Efficiency | Database observations, wall time, model/tool cost | Optimize only after safety gates pass |

Report per-scenario and worst-case metrics. Do not average a cross-tenant leak with nine correct queries. Use confidence intervals for sampled model behavior and repeat nondeterministic cases across seeds/model snapshots.

## Baselines, slices, and comparison discipline

Every release report compares the candidate against useful alternatives, not only its previous prompt.

| Baseline | What it establishes |
|---|---|
| No-agent/manual runbook | Whether model assistance improves time, evidence coverage, and operator comprehension without worsening safety |
| Deterministic rules/query templates | Whether the problem needs model reasoning at all and where the model adds value |
| Deployed model/control bundle | Regression and changed-failure evidence for the actual upgrade decision |
| Oracle/offline expert answer | Maximum expected diagnosis/plan quality for curated cases; reviewer disagreement remains visible |
| Unsafe/unbounded implementation | Threat-control tests only: proves policy/adapter gates block attacks; never a production candidate |

Slice before aggregating by operating mode, engine/build/edition/provider, advisory versus actual execution, risk tier, target primary/replica/restore, object/data scale, schema shape, tenant/data classification, topology/lag, workload/connection pressure, migration/recovery primitive, failure point, model language, and known versus novel scenario. Also report hard-failure counts, worst case, p50/p95/p99 where meaningful, abstention/escalation, human override, induced database load, and time to containment/recovery.

Keep a locked holdout set for release comparison, a rotating adversarial set, and a production-shadow set with sanitized provenance. Incidents may create new fixtures, but do not silently rewrite prior labels or tune on the locked holdout. A capability passes only its supported matrix; an untested slice is `UNSUPPORTED`, not inherited success.

## Engine-specific scenario matrix

### PostgreSQL 18

- `EXPLAIN ANALYZE` on a data-changing statement is correctly treated as execution.
- Read-only transaction attempts temporary-object and unsafe-function paths.
- `CREATE INDEX CONCURRENTLY` waits on old transactions, fails, and leaves an invalid index; reconciliation and cleanup behave correctly.
- Two concurrent index builds on one table are serialized/rejected.
- `lock_timeout`, `statement_timeout`, and transaction/idle limits fire in different orders.
- Row-level security tests owner, `BYPASSRLS`, `FORCE ROW LEVEL SECURITY`, restrictive/permissive policies, views, and pooled session context.
- Replication slot/WAL retention approaches disk budget; analysis raises recoverability risk.
- Asynchronous failover exposes a write gap; old primary fencing is unavailable; promotion is blocked/escalated.
- Base/incremental backup dependency is missing; restore preflight fails closed.

### MySQL 8.4 LTS

- `EXPLAIN ANALYZE` is budgeted as a real execution.
- Read-only transaction attempts to change a `TEMPORARY` table.
- A long transaction holds a metadata lock while online DDL queues; the agent detects convoy risk.
- `ALGORITHM=INSTANT`, `INPLACE`, and `COPY` behavior differs; `LOCK=NONE` unsupported case fails rather than downgrades.
- `GET_LOCK()` session loss/failover demonstrates that it is not a durable global mutex.
- Schema/table/column privileges and views are tested without assuming generic RLS.
- Clone-based replication setup loses required binlogs to purge; preflight catches the retention gap.
- Binlog PITR restores to before/after a chosen transaction and validates application state.

### SQL Server 2025

- Estimated and actual execution plans cannot be confused.
- `READ COMMITTED` blocking behavior differs with `READ_COMMITTED_SNAPSHOT` configuration.
- Online index needs brief schema locks; `WAIT_AT_LOW_PRIORITY` paths for `NONE`, `SELF`, and dangerous `BLOCKERS` are policy-tested.
- Resumable index pause/resume/abort survives worker restart.
- Row-level security policy and privileged/side-channel cases are tested; Dynamic Data Masking is not counted as authorization.
- Query Store evidence reset/secondary behavior and stale plan assumptions are surfaced.
- `RESTORE VERIFYONLY` passes while a deeper restored integrity/application check fails; the workflow does not report recoverable.
- Forced Always On failover declares potential data loss and verifies jobs/logins/routing separately.

### Oracle AI Database 26ai

- DDL implicit commit invalidates a generic transactional rollback proposal.
- `EXPLAIN PLAN` differs from actual cursor plan due to binds/environment; the analysis preserves uncertainty.
- `DDL_LOCK_TIMEOUT` expires behind a blocker without unsafe repeated DDL.
- Online redefinition restrictions and VPD/policy interactions cause proposal rejection or a staged alternative.
- RMAN `VALIDATE` passes but an isolated application restore reveals missing external dependencies.
- Data Guard switchover and failover use different data-loss/approval paths; legacy/deprecated transition syntax is not generated.
- Logical-standby/non-replicated services are included in reconciliation.

### Managed services

- Provider API times out after accepting an operation; reconciliation uses request/operation ID.
- Blue/green guardrail rejects switch; no manual bypass occurs.
- Connections drop during switch and clients do/do not reconnect as expected.
- Provider failover completes with replica lag and possible loss; business acknowledgement and reconciliation are enforced.
- Restore cannot access encryption key or cross-region artifact; preflight catches it.
- Quotas, maintenance events, rate limits, and a control-plane outage exercise backoff and human escalation.

## Adversarial evaluation

Test instruction-like content in table/column names, comments, row values, plans, errors, log messages, migration files, and tickets. Examples should attempt to:

- change production target or tenant;
- request a credential, full result, or restricted columns;
- convince the model that an approval already exists;
- alter a timeout or suppress a health gate;
- hide DDL in a routine, comment, encoded identifier, or multi-statement construct;
- claim a backup or replica is verified;
- turn a diagnostic into an actual plan execution;
- induce the agent to log or echo secrets.

The expected result is not merely that the model refuses. The controller must make the attempted escalation impossible even when the model produces the malicious proposal.

## Concurrency and failure injection

```mermaid
sequenceDiagram
    participant W1 as Worker 1
    participant W2 as Worker 2
    participant L as Effect ledger/lease
    participant D as Database
    participant V as Verifier

    W1->>L: begin effect K
    L-->>W1: lease + STARTED
    W1->>D: apply effect tagged K
    D-->>W1: commit succeeds
    Note over W1: crash before receipt persisted
    W2->>L: resume effect K
    L-->>W2: STARTED/unknown; reconcile required
    W2->>D: inspect native/object state for K
    D-->>W2: intended effect already present
    W2->>V: verify definition and health
    V-->>W2: pass
    W2->>L: VERIFIED_SUCCESS; no duplicate execution
```

Inject failures at every boundary before and after an externally visible effect:

- process kill, node loss, workflow replay, duplicate delivery, lost heartbeat;
- database disconnect before send, during send, after commit, during result read;
- deadlock, serialization failure, lock timeout, cancellation race;
- credential expiry/revocation and TLS/certificate failure;
- audit/artifact/approval/policy service outage;
- topology epoch changes between proposal, approval, and commit;
- replica promotion while a session or advisory lock is active;
- storage/log/WAL/binlog pressure and backup deletion race;
- verifier delay, partial evidence, and delayed workload regression.

For every point, assert terminal state, effect count, target correctness, lease/credential cleanup, audit completeness, and human ownership. “The workflow retried” is not a sufficient assertion.

## Benchmark use

Use public benchmarks as components, with their limits explicit:

- [Spider 2.0](https://spider2-sql.github.io/) tests enterprise-style text-to-SQL workflows across complex schemas and dialects; it is useful for query reasoning and tool use, not production effects.
- [BIRD](https://proceedings.neurips.cc/paper_files/paper/2023/file/83fc8fab1710363050bbd1d4b8cc0021-Paper-Datasets_and_Benchmarks.pdf) adds large databases and valid efficiency; it still does not establish operational authority safety.
- [Dr.Spider](https://openreview.net/pdf?id=Wc5bmZZU9cy) evaluates robustness under perturbations and can inspire schema/question adversarial cases.
- [DBA-Bench](https://arxiv.org/abs/2607.22165) is a July 2026 preprint with live PostgreSQL operational scenarios, active workload evaluation, and a safety-aware pass metric. Its reported agent/human gap is directly relevant, but the repository stated artifacts were still being finalized at the research date. Treat it as an emerging external benchmark and reproduce locally before making release claims.

Create an internal benchmark from real schema patterns, rejected changes, near misses, incidents, slow queries, backup gaps, and failovers. Remove or synthesize sensitive data and preserve the original policy/engine version in each fixture.

## Release gates by operating mode

| Stage | Required evidence before production |
|---|---|
| Advisory | High evidence faithfulness; known-unknown behavior; no sensitive-data escape; expert usefulness on historical cases |
| Query assistant | Advisory gates plus zero policy/tenant escapes in suite, resource budgets, cancellation, replica behavior, dialect matrix |
| Migration reviewer | Advisory gates plus lock/rewrite/log/replication forecasts, engine-specific rollback/recovery accuracy, CI artifact stability |
| Supervised read/local effect | Exact typed effect, approval binding, JIT privilege, ambiguous-outcome reconciliation, independent verification, kill drill |
| Production schema/data effect | Representative-scale rehearsal, concurrency/failure suite, recovery drill, live canary gates, on-call ownership |
| Restore/failover | Full application restore or topology drill, fencing, loss decision, routing/consumer reconciliation, incident-command approval |

Every gate has an expiration tied to model/prompt/policy/adapter/engine/provider changes. A new major database version, model snapshot, policy rewrite, dialect parser, or tool schema reopens the relevant evaluation set.

## Delivery roadmap

### Stage 0: deterministic foundations

Define the immutable target/identity registry, all typed request/state/event/plan/tool/effect/receipt schemas, evidence/artifact provenance, seven memory lifetimes, data classifications, policy/approval service, effect ledger, observability, kill path, and disposable engine/provider matrix. No live model access to production.

**Exercise:** restore the control-plane state from backup; replay duplicate/out-of-order events; compact and resume a workflow containing an expired approval and an `UNKNOWN` effect; rotate/revoke a canary credential.

**Exit evidence:** schema/transition tests, event/source high-watermark reconciliation, invariants-hash preservation, tenant/secret negative tests, adapter qualification matrix, DR receipt, named owners, and a documented decision for when the product remains deterministic rather than agentic.

### Stage 1: advisory analyst

Analyze exported/redacted evidence, then allow bounded live catalog/telemetry tools. Establish human feedback, evidence-faithfulness, usefulness, abstention, and unsupported-claim metrics. This stage produces value without database effects.

**Exercise:** replay a lock incident, statistics/cardinality regression, replication-retention warning, malicious database comment, and stale/conflicting evidence across every supported engine.

**Exit evidence:** held-out results beat or complement the deterministic/manual baseline, claims retain provenance and uncertainty, hard privacy/injection gates pass, and no database or credential path exists from the model runtime.

### Stage 2: query assistant and migration reviewer

Enable narrow read previews for low-sensitivity scopes and generate migration-review artifacts for CI. Keep migration execution in the existing human-owned deployment tool. Expand only after engine, plan, tenant, result, cost, connection, and cancellation suites pass.

**Exercise:** run estimated-versus-actual plan traps, side-effecting routine and temporary-object attempts, cross-tenant joins, expensive scans, lock/cancellation races, and expand/contract reviews for PostgreSQL, MySQL, SQL Server, and Oracle.

**Exit evidence:** zero tested authorization/tenant escapes, enforced row/byte/time/connection budgets, verified cleanup from the database side, stable CI artifacts, and engine-specific migration/recovery findings accepted by qualified reviewers.

### Stage 3: supervised low-blast-radius effects

Add one named capability at a time, such as canceling an exact runaway query or creating an approved nonblocking index. Require approval, window, commit-time preflight, JIT credential, durable effect record, independent verification, and kill drill.

**Exercise:** crash before send, after database commit, and before receipt persistence; change the topology epoch after approval; expire credentials; race two workers; force cancellation to return late or ambiguously.

**Exit evidence:** exactly-once semantic outcome under at-least-once delivery, no blind retry, correct `UNKNOWN` containment, approval drift rejection, complete audit, bounded load/locks, verified rollback or forward-recovery, and measured human takeover.

### Stage 4: long-running maintenance and recovery drills

Add durable workflows for batched changes and isolated restores. Validate checkpoints, compaction/resume, provider operations, data handling, capacity admission, and achieved RPO/RTO.

**Exercise:** pause/restart a backfill across batch boundaries; inject lag/storage/connection pressure; remove one backup/log dependency or key; restore at realistic scale with outbound jobs fenced; fail a provider API after acceptance.

**Exit evidence:** reconcilable batch/effect receipts, no invariant loss after restart, verified full backup chain and application restore, measured recovery bottlenecks, cleanup/deletion proof, capacity headroom, and on-call runbook execution within target.

### Stage 5: continuity operations

Only after regular rehearsals, enable supervised switchover and then incident failover assistance. Keep final data-loss and fencing authority with qualified humans and infrastructure controls.

**Exercise:** rehearse zero-loss switchover, lagged failover, unavailable old primary, failed fencing, provider timeout, endpoint/DNS/client reconnect storm, divergent old-primary writes, and downstream CDC/job reconciliation.

**Exit evidence:** explicit loss-window/business exposure decision, fencing proof or recorded incident-authority exception, single-writer and routing proof, new topology epoch, invalidated stale grants, data repair/rejoin ownership, restored backup/PITR protection, and achieved application RTO/RPO.

### Stage 6: controlled evolution across engines and estates

Turn reviewed incidents, unsafe proposals, unknown effects, lock/replication surprises, restore failures, operator corrections, and new engine/provider behavior into candidate evaluation cases. Replay the current and proposed model, prompt, context/compaction, policy, schema, adapter, verifier, engine, provider, migration, secret, workflow, and telemetry bundles; shadow and canary by mode, version, tenant, and effect class; migrate active durable work explicitly; retain the previous complete bundle and manual/operator path.

**Exercise:** upgrade one engine build and one adapter/tool dependency; replay old workflow histories; rotate model/prompt/policy independently; inject rollback mid-canary; delete/disable the candidate while `UNKNOWN` effects still require the old reconciler.

**Exit evidence:** every behavioral release is reproducible and attributable, improves locked holdout and production-shadow evidence without widening authority, preserves tenant/PII/recovery invariants, has named database/on-call owners, supports active-work migration, and can roll back while unknown-effect reconciliation, restore/failover control, and ordinary database operations remain available.

Do not make progression automatic. A useful steady state may remain advisory/reviewer-only.

## Evaluation report template

For each release, record:

- exact model/provider, prompt, controller, policy, schema, adapter, engine, provider, and verifier versions;
- scenario corpus version and sensitive-data provenance;
- repetitions/seeds, scorer method, human rubric and agreement;
- per-mode/per-engine/per-risk outcomes and worst cases;
- every catastrophic or hard-gate failure without aggregation;
- database workload impact and recovery/containment timing;
- comparison with the deployed baseline;
- known gaps, accepted residual risk, owner, expiry, and rollout scope;
- rollback/disable criteria and post-deployment monitoring window.

## Final production-readiness checklist

- [ ] The evaluation environment reproduces supported engine/version/edition/provider behavior.
- [ ] Model correctness and deterministic control effectiveness are scored separately.
- [ ] Safety, tenant, PII, effect count, and recoverability have hard gates.
- [ ] Every operation mode has engine-specific golden and negative scenarios.
- [ ] Adversarial database content cannot escalate authority even when the model follows it.
- [ ] Concurrency and failures are injected before/after every external effect boundary.
- [ ] Historical incidents and near misses are replayed with sanitized provenance.
- [ ] Human approval comprehension and takeover are tested, not assumed.
- [ ] Release evidence expires on relevant model, policy, adapter, engine, or provider change.
- [ ] Production rollout proceeds through shadow, advisory, canary, and narrow capability stages.

## Related guides

- [Operating models, requirements, and risk](01-operating-models-requirements-and-risk.md)
- [Reliability, observability, scaling, and operations](08-reliability-observability-scaling-and-operations.md)
- [Evaluation-driven development](../../evaluation/evaluation-driven-development.md)
- [Trajectory and reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md)
- [Observability and tracing](../../evaluation/observability-and-tracing.md)

## Selected sources

- [Spider 2.0 project](https://spider2-sql.github.io/)
- [Spider 2.0 ICLR 2025 paper](https://proceedings.iclr.cc/paper_files/paper/2025/file/46c10f6c8ea5aa6f267bcdabcb123f97-Paper-Conference.pdf)
- [BIRD benchmark paper](https://proceedings.neurips.cc/paper_files/paper/2023/file/83fc8fab1710363050bbd1d4b8cc0021-Paper-Datasets_and_Benchmarks.pdf)
- [Dr.Spider robustness benchmark](https://openreview.net/pdf?id=Wc5bmZZU9cy)
- [DBA-Bench preprint](https://arxiv.org/abs/2607.22165)
- [DBA-Bench artifact repository](https://github.com/TanJI-C/DBA-Bench)

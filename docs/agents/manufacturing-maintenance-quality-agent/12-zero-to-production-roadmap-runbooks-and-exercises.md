# Zero-to-Production Roadmap, Runbooks, and Exercises

This roadmap advances authority only when measured evidence supports it. Stages 0–6 are cumulative: later stages retain every earlier safety, identity, evidence, effect, and governance control. A team may remain indefinitely at a useful read-only or draft stage.

## Stage 0 — Baseline the manual system

Deliver:

- named mission, site/use-case scope, authority ceiling, accountable owners, RACI, and neighboring-domain handoffs;
- process map from trigger through outcome, including workarounds and night/shift/offline variants;
- hazard analysis, data classification, regulatory/quality applicability, threat model, and prohibited-effects register;
- source/system/vendor/version inventory and record-of-authority map;
- measured case volume, triage time, backlog, error/rework, failure/defect escape, and human review effort;
- no-agent alternatives and why deterministic rule, BI, workflow, or search alone is insufficient;
- evaluation population, acceptance gates, kill conditions, incident roles, and retirement criteria.

Exit only when process, source, safety, quality, security, and operational owners agree on the problem and smallest loop.

## Stage 1 — Establish deterministic foundations

Build:

- site-scoped effective-dated identity and genealogy mappings;
- typed observation/evidence envelopes with time, status, unit, calibration, sequence, and lineage;
- controlled knowledge ingestion with applicability and supersession;
- adapter capability registry and read-only contract tests;
- deterministic freshness, unit, sampling, policy, and constraint services;
- protected evidence/audit storage and baseline telemetry.

Measurable exit gates:

- zero false cross-site/object merge in the representative identity suite;
- 100% required metadata on critical evidence used by eval cases;
- all tag reuse, component replacement, correction, clock, gap, and unit cases resolve or stop correctly;
- withdrawn/inapplicable knowledge is excluded in every test;
- technical proof that agent credentials/routes cannot reach control or safety interfaces.

## Stage 2 — Run a read-only shadow MVP

Enable one coordinator for one site/use case. It can retrieve bounded evidence, produce a structured hypothesis set, cite controlled sources, identify missing/conflicting inputs, and draft an unsubmitted artifact.

Do not enable business writes, schedule changes, holds, external vendor submissions, or cross-site retrieval.

Exit gates:

- no unsafe-boundary or unsupported-authority recommendation admitted by policy;
- subject-matter usefulness lower bound and evidence completeness meet approved targets;
- stop recall meets the risk-approved target without unacceptable false-stop burden;
- false identity join, invented specification/procedure, and hidden unit/calibration failure are zero in the release suite;
- shadow comparisons show value over search/workflow/no-agent baselines;
- users can distinguish fact, inference, conflict, draft, and authoritative state.

## Stage 3 — Add durable human workflow

Add case state, resource locks/fencing, exact approvals, continuity receipts, pause/resume, cancellation, effect-intent ledger without write execution, and accountable handoff queues.

Exit gates:

- crash/replay/duplicate-message tests preserve a single workflow owner;
- compaction and resume preserve all material uncertainty, approvals, locks, and effect state;
- approvals invalidate on target, digest, source-version, evidence, policy, or expiry change;
- shift handoff, rejection, cancellation, timeout, and escalation exercises complete without implied consent;
- audit export reconstructs every evaluated decision and protected fields remain access-controlled.

## Stage 4 — Enable bounded business effects

Start with one M2 operation such as creating a non-released work-order draft. Require semantic operation ID, concurrency guard, exact approval or narrowly pre-authorized policy, fresh preconditions, authoritative read-back, and reconciliation.

Exit gates:

- zero duplicate effects across retries, failovers, timeouts, restores, and adapter rollback;
- 100% confirmed effects match intended target and immutable fields on read-back;
- every unknown outcome is fenced, reconciled, or escalated within the defined objective;
- cancellation and qualified compensation work only in allowed states;
- canary outcomes and human burden meet gates with rollback rehearsed;
- M4 operations remain absent from tools, credentials, routes, and policy.

Add M3 operations one at a time only when their side effects, compensation limits, and human accountability are proven.

## Stage 5 — Scale and survive isolation

Add more sites/workloads only after site-specific qualification. Test edge/plant/region placement, offline profiles, quotas, backpressure, queue expiry, fair scheduling, HA, backup/restore, disaster recovery, and recovery load.

Exit gates:

- a disconnected site never executes central-dependent or expired authority;
- reconnect reconciles and revalidates before dispatch;
- recovery drain stays within source, adapter, operator, and cost capacity;
- one site cannot read, affect, or starve another;
- RPO/RTO and semantic restore/reconciliation objectives are met;
- regional/site failover does not duplicate ownership or external effects.

## Stage 6 — Operate governed evolution

Establish drift monitoring, incident-to-evaluation feedback, controlled datasets, refresh ownership, behavior manifests, staged rollout, rollback compatibility, regulatory/standards watch, and retirement.

Exit gates:

- every material model/prompt/tool/policy/knowledge/adapter/workflow change becomes a traceable behavior release;
- regressions include all prior incidents and high-consequence cases;
- no live outcome enters training/episodic memory without curation, outcome reconciliation, privacy, and bias review;
- shadow/canary/rollback and reduced-authority operation are routine;
- stale standards, vendor versions, procedures, connectors, and models have owners and deadlines;
- the service is retired or reduced when value no longer exceeds risk and operational cost.

## Runbook — Unknown work-order creation

**Trigger:** adapter timeout, connection loss, proxy error, or executor crash after dispatch.

1. Persist `UNKNOWN`; do not report failure or retry.
2. Fence the case/asset operation and stop conflicting order creation.
3. Query the EAM by semantic transaction/external reference.
4. If exactly one match exists, compare site, asset, case, type, job-plan version, and state; record confirmation and read-back.
5. If no match exists, wait the qualified consistency window and query again.
6. Retry with the same operation ID only after authoritative absence, fresh preconditions, valid approval, and remaining attempt budget.
7. If multiple/conflicting matches exist, freeze automation and route the complete reconciliation bundle to the EAM owner and planner.
8. Correct duplicates through controlled vendor procedures; never delete audit evidence.

Success: one intended external record, authoritative identity known, ledger complete, locks released only after reconciliation.

## Runbook — Stale or missing edge evidence

**Trigger:** gap, bad/uncertain status, clock uncertainty, stale data, missing unit/context, or invalid calibration.

1. Mark affected evidence ineligible under the operation-specific policy.
2. Identify every open case/decision that used or awaits it.
3. Stop effects whose preconditions depend on the evidence; do not infer a safe or normal state.
4. Preserve sequence/clock/source diagnostics and notify the edge/OT owner.
5. Use qualified redundant evidence only when policy explicitly permits it and provenance remains separate.
6. On restoration, mark gaps, replay within bounded capacity, and recompute eligibility; do not rewrite prior decisions.

Success: no affected effect proceeds, evidence loss is visible, and recovery does not create false continuity.

## Runbook — Asset or genealogy identity collision

**Trigger:** multiple matches, tag reuse, serial conflict, component replacement mismatch, or cross-site collision.

1. Freeze affected joins, recommendations, effects, and trace-population exclusions.
2. Preserve the mappings and source versions used by prior decisions.
3. Expand containment/trace population conservatively where product risk may exist.
4. Route to the identity/data owner with source records and effective-time evidence.
5. Append corrected mappings with validity; do not overwrite history.
6. Re-evaluate every impacted workflow and product/maintenance record.

Success: historical resolution remains reproducible and no effect used an ambiguous target.

## Runbook — Quality hold or release conflict

**Trigger:** QMS, MES, ERP, warehouse status, or human decision records disagree.

1. Disable agent effects for the affected population.
2. Preserve all source states, versions, timestamps, approvals, and adapter evidence.
3. Treat product as not agent-eligible for release; do not infer the legally controlling status.
4. Notify accountable Quality and Supply Chain owners and follow site containment procedure.
5. Reconcile genealogy and all possible stock/shipments without narrowing unknowns.
6. Resume only after authoritative systems are corrected through controlled processes and policy revalidates.

Success: accountable quality determines disposition/release; all systems and the audit chronology reconcile.

## Runbook — Safety-boundary violation

**Trigger:** attempted control write, unsafe procedure advice, LOTO/permit attestation, interlock bypass, or “safe to operate” conclusion.

1. Activate the site effect kill switch and preserve evidence.
2. Verify through independent network/tool logs whether any prohibited route or effect existed.
3. Escalate to operations, EHS/safety, OT security, and platform owners; use emergency site procedures if physical risk exists.
4. Quarantine the behavior, tool, content, and affected credentials.
5. Inspect related workflows and product/equipment consequences.
6. Correct the architectural/policy/control failure, add deterministic regression tests, and requalify at reduced authority.

Success: physical state is handled by authorized site personnel, scope is known, no hidden path remains, and recurrence gates pass.

## Runbook — Adapter breaking change

**Trigger:** schema fingerprint, capability, permission, endpoint, status mapping, or side effect differs from qualification.

1. Circuit-break the affected operation; reads may continue only if still qualified.
2. Mark inflight ambiguous attempts `UNKNOWN` and reconcile through the last trusted semantics or accountable owner.
3. Pin evidence, vendor version, response samples, and maintenance/deprecation notice.
4. Update the adapter contract and mappings in a nonproduction instance.
5. Run contract, permission, idempotency, concurrency, failure, load, security, and rollback suites.
6. Release by shadow/canary; do not translate unknown vendor fields with model guesses.

Success: exact installed behavior is requalified and all inflight effects remain accounted for.

## Runbook — Site isolation and reconnect

**Trigger:** site-region connectivity loss or regional service failure.

1. Activate the offline profile and display current authority state.
2. Continue bounded evidence capture and approved local human processes.
3. Reject central-dependent effects and expire queues/approvals by policy.
4. Monitor storage, clock, queue age, local policy/knowledge expiry, and operator capacity.
5. On reconnect, authenticate both sides, exchange cursors, and reconcile effects before dispatch.
6. Revalidate identity, evidence, holds, procedures, policy, approval, and windows for every queued item.
7. Drain with throttling and site fairness; expose gaps and expired work.

Success: no stale action replays, no cross-site leakage occurs, and live load remains stable during recovery.

## Runbook — Behavior rollback and recall/CAPA investigation

**Trigger:** safety regression, systematic misclassification, data leak, identity defect, harmful recommendation, or adapter/effect defect.

1. Reduce authority or stop effects; pin affected behavior release.
2. Inventory active workflows and classify every effect state, especially `UNKNOWN`.
3. Roll back only to a state-compatible release; quarantine incompatible workflows.
4. Query release/site/time/use-case/object exposure from audit and effect ledgers.
5. For product impact, conservatively assemble genealogy and distribution evidence for accountable Quality/Regulatory decisions.
6. Preserve model, prompt, tools, policies, knowledge, adapters, inputs, outputs, approvals, and source versions.
7. Conduct causal analysis, corrective action, regression additions, and effectiveness monitoring.

Success: impact population is reconciled, accountable roles make recall/CAPA decisions, and the replacement release passes expanded gates.

## Operational exercises

Run each exercise with observers, expected evidence, pass criteria, and after-action owners:

1. Pump anomaly with a sequence gap, changed operating regime, and expired approval.
2. Dimensional drift with a corrected result, calibration concern, and incomplete genealogy.
3. EAM timeout after acceptance followed by coordinator and region failover.
4. QMS hold confirmed while ERP shows unrestricted stock.
5. Asset tag reuse after component replacement during an open case.
6. Malicious instruction embedded in an OEM PDF and maintenance note.
7. Eight-hour site isolation, local queue saturation, and throttled reconnect.
8. Restore from backup whose ledger predates an externally accepted effect.
9. Model/provider behavior change causing higher unsafe-boundary proposals.
10. Procedure withdrawal during a paused and compacted workflow.
11. Connector credential compromise and cross-site access attempt.
12. Product-impact investigation requiring forward/backward genealogy and distribution reconciliation.

## Final go-live evidence pack

- approved mission, scope, authority, prohibited effects, RACI, and no-agent decision;
- hazard/threat/privacy/regulatory/quality assessments and site architecture;
- source-of-authority, identity, evidence, unit, calibration, and genealogy contracts;
- qualified adapter capabilities and real-system test results;
- workflow/effect/approval/unknown/cancel/compensation schemas and drills;
- memory taxonomy, continuity receipt, retrieval/knowledge controls, and resume tests;
- SLOs, dashboards, alert routing, evaluation results, human-factor review, and canary plan;
- offline/capacity/cost/backpressure/HA/DR/restore/recovery-load evidence;
- security, supply-chain, incident, recall/CAPA, kill-switch, and rollback exercises;
- signed behavior release, owner/on-call map, runbooks, training, refresh calendar, and retirement criteria.

If a required artifact is missing, state the limitation and keep the corresponding authority disabled.

## Source and refresh handoff

The [research packet](../../research/packets/manufacturing-maintenance-quality-agent-blueprint.md) records the primary sources, currency caveats, disagreements, and refresh triggers used for this playbook. Revalidate it against installed vendor versions, local law, site procedures, and current standards before implementation.


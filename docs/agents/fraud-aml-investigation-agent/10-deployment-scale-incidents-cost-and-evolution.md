# Deployment, Scale, Incidents, Cost, and Governed Evolution

> **Purpose:** Operate the bounded investigator under real case volume, deadlines, provider/source failure, behavior change, and regulatory evolution.  
> **Section index:** [Fraud and AML investigation agent](README.md)

## Production topology

Start with the fewest independently operated components that preserve authority and failure boundaries:

~~~mermaid
flowchart TB
    PROD["Alert producers"] --> ADM["Admission / dedupe / priority"]
    ADM --> Q["Durable case queue"]
    Q --> CASE["Case workflow + transactional state"]
    CASE --> READQ["Bounded retrieval jobs"]
    READQ --> ADAPT["Source adapters / evidence store / graph jobs"]
    CASE --> RQ["Reasoning queue"]
    RQ --> WORK["Stateless reasoning workers"]
    WORK --> CASE
    CASE --> HQ["Human review queues"]
    HQ --> CASE
    CASE --> OUT["Approved effect outbox"]
    OUT --> EWORK["Credentialed effect workers"]
    EWORK --> RECON["Destination reconciliation"]
    RECON --> CASE
    CASE --> AUD["Audit/provenance"]
    ADM --> OPS["Metrics / traces / SLOs"]
    ADAPT --> OPS
    WORK --> OPS
    EWORK --> OPS
~~~

One transactional case service and one worker pool may satisfy early production. Add separate queues/pools for data residency, tenants, high-risk deadlines, models, or effects only when isolation, capacity, or compliance evidence requires it. Keep effects separated even at small scale because credential and retry semantics differ fundamentally from reasoning.

## Environment and supply-chain controls

Use isolated development, evaluation, staging, and production identities/data. Production data does not flow into development or ad hoc prompts. Every build uses pinned dependencies, reviewed artifacts, software/model/data manifests, vulnerability and secret scanning, provenance, and promotion rather than rebuild-in-place. Configuration and policy changes follow the same evidence path as code.

Provider model aliases can move. Pin a supported model snapshot or otherwise record the exact served model identifier and continuously detect unexpected changes. Pin tool schemas, context/compaction behavior, provider region/retention settings, source adapters, typology packs, entity/graph jobs, and jurisdiction profiles.

## Behavior release manifest

A deployable behavior is more than a container image:

~~~yaml
release_id: fraud-aml@2026.08.31.1
runtime:
  application_commit: sha256:...
  workflow_version: case-workflow.v7
  dependencies_sbom: artifact:...
reasoning:
  provider: ...
  model_snapshot: ...
  parameters: {...}
  system_and_output_schema: sha256:...
  context_compiler: sha256:...
  compaction_policy: sha256:...
knowledge:
  typology_packs: [...]
  procedure_corpus: [...]
data_and_analytics:
  adapter_schemas: [...]
  entity_resolver: ...
  graph_builder: ...
  feature_jobs: [...]
policy:
  authorization_commit: sha256:...
  jurisdiction_profiles: [...]
  approval_matrix: ...
  privacy_redaction: ...
evaluation:
  suite_versions: [...]
  results: artifact:...
operations:
  allowed_alert_families: [...]
  tenant_and_region_scope: [...]
  budgets: {...}
  slo_policy: ...
  runbooks: [...]
  rollback_target: ...
approvals: [...]
~~~

Store evaluation evidence and approvals with the manifest. A prompt-only hotfix, provider setting change, list-parser update, or typology edit is a behavior release and cannot bypass it.

Treat the manifest as one behavior bundle: inference settings, role prompts, context/compaction, tools and operation
capabilities, provider/API/tenant configuration, schemas/parsers/normalizers, list and source mappings, rules/models/
features/entity/graph jobs, policy/clocks, UI decision framing, evaluators and fallbacks. Produce a semantic diff that
names which alert families, cases, fields, identities, populations, recommendations, effects and records may change.

Rollback selects a previously approved whole bundle for future execution. It never deletes evidence/audit, rewinds a
human decision, changes the historical list/model/rule used, unfiles a report, reverses a restriction, or assumes an
external effect was undone. Pin an in-flight bounded step and explicitly hold, rebuild or migrate active cases when
bundle semantics are incompatible; always recheck current identity, policy, list freshness, approval and destination
state at commit.

## Active-case version policy

| Change | Default active-case behavior | Exception |
|---|---|---|
| Prompt/model/context/compiler | Pin case run generation; new work uses promoted release | Security/quality emergency can force proposal-only or rebuild context |
| Typology/procedure | Preserve reviewed pack; flag material update for reopen/migration | Legal deadline or dangerous error follows approved emergency rule |
| Transaction/KYC source correction | Append revision; invalidate dependent claims when material | None—never overwrite history |
| Sanctions/watchlist update | Preserve decision snapshot and apply current rescreen/escalation rule | Jurisdiction/runbook determines transaction-state urgency |
| Jurisdiction/policy change | Historical decision retains old version; pre-effect commit uses current authority | Legal owner decides migration/reapproval |
| Entity resolver/graph/features | Do not silently recompute reviewed evidence | Targeted re-evaluation/migration with version comparison |
| Effect adapter/schema | Pending intent stays immutable; dispatch compatibility checked | Rebuild payload and reapprove if material |

This resolves the tension between reproducibility and current obligations: keep the old snapshot for explanation, but allow deterministic current policy to block execution or require rescreen/review.

## Progressive delivery

1. **Offline:** deterministic baseline, held-out evaluation, red team, compaction/recovery and load tests.
2. **Shadow:** real eligible traffic, proposal-only, hidden from primary investigators where comparison design permits.
3. **Assisted pilot:** one alert family, jurisdiction, data path, trained queue, strict sampling, no external effects.
4. **Canary:** small tenant/queue share with automated quality/SLO/authority monitoring and on-call ownership.
5. **Incremental expansion:** one dimension at a time—family, rail, jurisdiction, language, tenant, source, memory class, or effect preparation.
6. **Steady state:** continuous sampling, failure mining, drift/capacity monitoring, independent challenge, scheduled refresh.

Later promotion may allow effect **preparation** or entry into an approval workflow; it does not make the model an approver. Do not expand data scope and action authority in the same release.

Shadow workers have no mutating or notification credentials. Early canaries exclude filing dispatch, sanctions or
payment restriction, customer communication, deadline-critical work, confidential cross-entity sharing and other
irreversible paths; those capabilities require separate operation-level evidence and approval. Canary assignment is
deterministic and auditable, keeps a concurrent manual path, and stops automatically on a hard invariant, list/source
freshness failure, severe slice error, confidentiality event, unknown-effect growth or reviewer-capacity breach.

## Capacity model

The bottleneck is usually the slowest protected resource, often qualified human review or a constrained source—not model tokens. Plan each alert family using:

~~~text
arrival_rate
× cases_per_alert_after_dedupe
× mean retrieval fanout and data volume
× reasoning attempts/replans
× human review minutes and rework rate
× escalation/effect/reconciliation probability
× peak and failure headroom
~~~

Maintain separate capacity budgets for:

- admission and deadline calculation;
- transaction/KYC/list/registry/provider reads and their rate limits;
- deterministic graph/entity/feature computation;
- model request/token quotas and latency;
- case database, evidence store, index and audit writes;
- qualified investigators by role, jurisdiction, language, shift, and segregation-of-duties constraint;
- effect workers and destination reconciliation;
- security/privacy/compliance incident and QA capacity.

Load tests need realistic long cases, graph fan-out, large/hostile documents, provider rate limits, source partials, reviewer absence, destination lag, audit slowness, and a sanctions/list update burst—not only average model latency.

## Queueing, backpressure, and degradation

| Pressure | Safe response | Unsafe response |
|---|---|---|
| Alert burst | Dedupe, bounded admission, risk/deadline priority, preserve producer evidence | Drop alerts silently or let model invent priority |
| Human backlog | Reduce new assisted scope, route to trained queues, extend only lawful internal targets, manual/deterministic mode | Auto-approve, hide cases, or lower required evidence |
| Model quota/outage | Pause reasoning; deterministic evidence package/manual workflow; alternate approved release if prevalidated | Send restricted data to an unapproved provider/model |
| Source outage | Mark coverage unavailable, cache only valid versioned snapshots, retry within bounds | Convert failure to zero results or use stale data silently |
| Graph overload | Smaller governed query bounds, precomputed features, queue and surface truncation | Unbounded traversal or opaque sampling |
| Filing destination lag | Continue receipt lookup/reconciliation and escalate deadline risk | Re-submit unknown outcome |
| Audit/control-store failure | Stop governed mutation/effects if evidence cannot be durably recorded | Rely on later reconstruction from logs |
| Cost pressure | Reduce low-value retrieval/context, smaller validated model, batch deterministic work | Suppress contradictions, citations, human review, or safety tests |

Admission control must understand human capacity and legal deadlines. Model capacity without reviewer capacity only creates a faster backlog.

## Cost model and optimization order

Measure cost per **verified case outcome**, not per model call:

~~~text
source/provider fees
+ data movement/storage/index/graph compute
+ model input/output/cached/compaction/retry cost
+ investigator and approver time
+ QA, independent validation, security/privacy and operations
+ reconciliation, correction, incident and customer-remediation cost
~~~

Optimize in this order:

1. remove duplicate/low-value alerts and deterministic work from the loop;
2. precompute reusable versioned facts/features and retrieve narrow views;
3. improve completion/stop rules to avoid repetitive queries;
4. compact typed state and evidence handles, not uncontrolled prose;
5. route easy, low-risk synthesis to a smaller **validated** model and preserve escalation;
6. batch independent deterministic retrieval without weakening per-case authorization;
7. cache immutable/versioned results with tenant/purpose in the key;
8. improve the reviewer UI and evidence ordering to reduce correction time;
9. renegotiate providers or change architecture only after measuring total operating cost.

Include the cost of false negatives, mass false positives, unnecessary information requests/restrictions, confidentiality incidents, and duplicate effects in decisions. Cheap tokens do not make an unsafe workflow economical.

## Resilience and disaster recovery

Define RTO/RPO separately for admission, case state, evidence/provenance, list ingestion, human queues, audit ledger, and effects. Effects require fencing or a single-writer rule across regions. Backups are encrypted, access-controlled, retention-aware, restoration-tested, and included in deletion/legal-hold design.

At least quarterly for production scope, exercise:

- case database restore and projection rebuild;
- evidence/artifact integrity verification;
- queue duplication/reorder and expired leases;
- provider and critical-source outage;
- official-list missed delta/full resynchronization;
- identity/policy/credential revocation;
- region failover with in-flight effects and reconciliation;
- loss of a reviewer queue or jurisdiction specialist;
- rollback/quarantine of behavior release, typology, entity resolver, index, or poisoned source;
- recovery while a filing or sanctions deadline is approaching.

The exact frequency is risk- and jurisdiction-dependent; increase it after material changes or incidents.

### Recovery load and convergence

Steady-state capacity does not predict recovery. Track the backlog that exists after source, region, identity, list,
case, model or destination outage:

~~~text
recovery_work = unprocessed_source_events + unverified_list_deltas + stale_case_projections
              + pending_callbacks + UNKNOWN_effects + sources/graphs/indexes_to_reconcile
              + expired_approvals + due_or_at_risk_clocks + evidence_objects_to_verify
drain_time ≈ recovery_work / safe_net_capacity_after_reserved_critical_capacity
~~~

Restore in a reviewed order: identity/tenant/purpose and policy; official lists and urgent clocks; effect ledger,
single-writer fencing and reconciliation; authoritative source/evidence access; existing case state and human queues;
then new low-priority admission. Never replay a D3 effect or customer communication whose prior outcome is unknown.
Throttle redrive so sources, filing/sanctions destinations and reviewers are not flooded, and preserve tenant/family
fairness. Recovery is complete only after stores converge, lists/coverage and evidence integrity are verified, clocks
and approvals are accounted for, unknown effects are resolved/owned, and backlog drains within the safe operating
envelope—not when compute merely responds to health checks.

## Incident command

~~~mermaid
flowchart TD
    DET["Detect / report"] --> TRI["Classify confidentiality, authority, quality, effect, availability"]
    TRI --> CON["Contain with admission / source / model / tool / effect / export switches"]
    CON --> REC["Reconcile state, evidence, access, pending effects"]
    REC --> IMP["Identify impacted cases, people, jurisdictions, deadlines, releases"]
    IMP --> REM["Legal/compliance/security/operations remediation"]
    REM --> VAL["Independent validation + safe restore"]
    VAL --> LEARN["Failure corpus, controls, evals, runbooks, behavior release"]
~~~

Create category-specific runbooks for:

- cross-tenant or wrong-purpose data exposure;
- SAR/STR confidentiality or regulator-disclosure leakage;
- unauthorized, duplicate, wrong-target, or unknown filing/restriction;
- false-negative cluster or mass false-positive/biased burden;
- erroneous sanctions/entity match or missed official-list update;
- poisoned document, typology, index, entity graph, or provider output;
- model/provider regression and source/adapter incompatibility;
- case backlog and missed/at-risk legal deadline;
- audit/provenance loss and regional failover.

Each runbook names the commander and security, privacy, legal, AML, sanctions, fraud, customer/payment operations, technology, communications, and regulator/law-enforcement liaisons as applicable. It defines evidence preservation, confidentiality of the incident itself, notification decision ownership, lookback/rescreen scope, customer remediation, recovery proof, and re-enable authority.

Use [NIST SP 800-61 Rev. 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final) as general incident-response guidance, then add the workload's legal and effect-specific requirements.

## Change detection and governed evolution

Monitor separately:

| Drift/change | Detection | Response |
|---|---|---|
| Source/schema/coverage | Contract failures, null/volume/distribution and freshness shifts | Quarantine adapter/version; gap affected cases |
| Entity/graph/features | Merge/split challenges, topology/feature distributions, job-version changes | Revalidate; trace dependent claims and cases |
| Alert/typology population | Family volume, trigger contributors, benign mix, evasion indicators | Recalibrate or retire under governance |
| Model/agent behavior | Eval/shadow/post-release quality, trajectory, tool use, refusal, costs | Freeze promotion, rollback, narrow scope |
| Human interaction | Acceptance, edits, returns, disagreement, fatigue and queue age | UI/training/capacity changes; audit automation bias |
| Fairness/burden | Slice errors, requests, holds/restrictions, corrections/duration | Pause affected path; investigate source/policy/system cause |
| Law/regulator/list | Official feeds, legal inventory, filing schemas, AMLA/FIU/authority publications | Legal review, jurisdiction profile and behavior release |
| Threat landscape | Injection/exfiltration attempts, fraud typologies, vendor incidents | Update containment and evals; never paste advisories into control prompt |

The EU AML framework is still operationally evolving: AMLA's direct supervision is scheduled to begin in 2028, and AMLA is publishing regulatory instruments. Teams in scope need owned monitoring and legal interpretation rather than a frozen “EU rules” paragraph.

### Failure mining loop

1. Sample severe errors, near misses, reviewer corrections/dissent, security probes, incidents, unknown effects, deadline breaches, source challenges, and slice regressions.
2. Trace each to source, contract, data quality, entity resolution, typology, context, planning, model, UI, policy, human capacity, adapter, or operational cause.
3. Create the smallest durable fix and a regression/failure-injection case.
4. Re-run affected families, slices, and reliability suites—not only the new example.
5. Shadow/canary under a new behavior manifest.
6. Monitor for displacement: a fix for false positives can create missed risk, unfair burden, cost, or deadline failure elsewhere.

Do not automatically train or retrieve from raw reviewer edits. Curate, minimize, adjudicate, and separate evaluation from training/retrieval to reduce feedback loops and contamination.

Promote a failure into the governed corpus through a typed record:

~~~yaml
failure_candidate:
  candidate_id: fail_01...
  incident_or_case_ref: restricted-ref
  authoritative_outcome_ref: independent-adjudication-88
  minimized_fixture_ref: approved-synthetic-or-deidentified-fixture
  affected_behavior_release: fraud-aml@2026.08.31.1
  affected_versions: {list: ..., rule: ..., model: ..., adapter: ..., policy: ...}
  privacy_confidentiality_and_bias_review: approval-ref
  root_cause_class: list-parser-population-loss
  required_invariant: official-list-population-reconciles
  reviewer_roles: [sanctions-quality, integration-owner]
  train_retrieval_eval_partition: held-out-evaluation
  refresh_or_expiry_at: 2027-02-28
~~~

Do not copy the production case wholesale. Minimize or synthesize it, retain lineage to the independently adjudicated
outcome, test neighboring benign and severe cases, and prevent train/test contamination. Revalidate when list, policy,
source, typology, model, adapter or workflow semantics change; quarantine a fixture whose outcome was corrected.
Failure mining creates reviewed release evidence, never silent online learning.

## Refresh calendar and triggers

| Cadence/trigger | Review |
|---|---|
| Continuous | Source/list freshness, contract health, authority invariants, queues/deadlines, unknown effects, confidentiality/security signals |
| Each behavior release | Full manifest diff, targeted/full eval, privacy/security/model/data impact, runbooks, rollback, independent challenge as required |
| At least every 90 days | Volatile connectors/provider settings, official sources, typologies, access, drift/slices, capacity/cost, failure corpus |
| At least every 180 days | Full workload fit, architecture, memory decisions, jurisdiction inventory, threat/privacy/fairness assessment, DR and deprecation |
| Immediate | Legal/list/schema change; new jurisdiction/source/rail/language/tenant/effect/memory; provider change; severe error or incident |

These are engineering defaults, not regulatory frequencies. The institution must use stricter applicable requirements.

## Production exit checklist

- [ ] Production topology preserves source/evidence, case, reasoning, authority, effect, audit, and telemetry boundaries.
- [ ] One signed behavior manifest pins every material code, model, prompt/context, data, analytics, policy, privacy, evaluation, and operating input.
- [ ] Progressive delivery starts proposal-only and expands one controlled dimension at a time.
- [ ] Shadow/canary restrictions and whole-bundle semantic diff/rollback preserve authority, records and in-flight effects.
- [ ] Capacity includes source, graph, model, storage, audit, qualified human, effect, reconciliation, and incident constraints.
- [ ] Backpressure/degradation protects evidence, authority, confidentiality, deadlines, and review quality.
- [ ] Cost per verified case includes human, provider, data, control, correction, incident, and downstream burden.
- [ ] DR proves restoration, fencing, list recovery, credential revocation, effect reconciliation, and deadline handling.
- [ ] Recovery proves bounded convergence and backlog drain under reserved critical capacity.
- [ ] Category incidents have named owners, kill switches, lookback/remediation, re-enable gates, and exercised runbooks.
- [ ] Drift/failure mining, independent challenge, refresh triggers, rollback, and deprecation run continuously.
- [ ] Failure fixtures are minimized, independently adjudicated, partitioned, expiring and contamination-controlled.

## Sources and related guides

- [Google SRE — Addressing Cascading Failures](https://sre.google/sre-book/addressing-cascading-failures/)
- [NIST SP 800-61 Rev. 3 — Incident Response Recommendations and Considerations](https://csrc.nist.gov/pubs/sp/800/61/r3/final)
- [AMLA — Frequently Asked Questions](https://www.amla.europa.eu/faqs_en)
- [AMLA — Regulatory Instruments](https://www.amla.europa.eu/policy/regulatory-instruments_en)
- [European Commission — EU-level AML/CFT framework](https://finance.ec.europa.eu/financial-crime/anti-money-laundering-and-countering-financing-terrorism-eu-level_en)
- [Deployment, release, and incident response](../../operations/deployment-release-and-incident-response.md)
- [Scaling, capacity, and SLOs](../../operations/scaling-capacity-and-slos.md)

Return to the [category index](README.md) or use the [Stage 0–6 gates](01-workload-fit-authority-and-stages.md) for promotion decisions.

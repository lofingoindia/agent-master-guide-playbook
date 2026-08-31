# Workload Fit, Boundaries, and Autonomy

## 1. Start with the operational outcome

A Data Pipeline Operations Agent should improve **time to safe diagnosis and verified recovery**. It should not be justified by “number of automated actions” or how many systems an LLM can call.

Good target outcomes are measurable:

- reduce time from freshness breach to a supported root-cause hypothesis;
- reduce unsafe or duplicate historical reprocessing;
- detect incompatible changes before publication;
- reduce time to identify affected downstream data products;
- increase the percentage of recoveries with complete receipts and verification;
- lower incident toil without increasing data correctness or security incidents.

If a deterministic monitor, scheduler retry, contract check, or runbook solves the case reliably, use it. The agent belongs where evidence is heterogeneous, diagnosis has genuine ambiguity, and the final operation can still be bounded by code.

## 2. Capability map

| Capability | Deterministic component | Model contribution | Default authority |
|---|---|---|---|
| Detect a missed schedule or stalled offset | SLO monitor | Explain likely impact | Read only |
| Classify a failure | Error rules and incident taxonomy | Synthesize evidence across systems | Read only |
| Assess schema impact | Registry compatibility and contract validator | Explain semantic risk and affected consumers | Proposal |
| Build a backfill plan | Interval and partition planner | Select evidence-backed recovery strategy | Proposal |
| Retry a known transient task | Retry policy and orchestrator adapter | No model needed after classification | Supervised effect |
| Pause a damaging pipeline | Circuit breaker and policy | Summarize justification | Pre-authorized only for narrow conditions |
| Quarantine an output | Quality policy and publication gate | Explain anomalous evidence | Supervised or deterministic |
| Change production SQL, DAG code, or connector config | CI/CD | Draft a patch | Pull request only |
| Delete data or change retention | Governed privacy workflow | Discover evidence and descendants | Human approval; deterministic execution |

Keep model involvement out of a control path when it adds no uncertainty reduction.

## 3. Ownership seams

### Database operations

The pipeline agent may observe source schema metadata, replication lag, transaction identifiers, or warehouse job states. It does not own:

- production DDL;
- index or query-plan changes;
- lock termination;
- engine upgrades;
- backup, restore, replication topology, or failover.

When a source schema change is implicated, the agent produces a compatibility and consumer-impact report, then routes engine or migration work to the Database Operations Agent or database owner.

### Analytics

The pipeline agent verifies that a governed dataset was delivered as contracted. It does not decide whether a business trend is meaningful, perform exploratory statistics, or author an executive narrative. Those are analytic tasks.

### Business intelligence

Pipeline freshness, backlog, duplicate rate, and quality-gate status are operational signals. Persistent business metric monitoring, anomaly interpretation, alert subscriptions, and decision follow-through belong to BI.

### Application ownership

CDC and event pipelines inherit semantics from source applications. An agent cannot infer that a particular update, delete, or event key is business-correct merely because it serialized successfully. Source owners remain accountable for emission semantics.

## 4. Risk classification

Classify risk from concrete dimensions rather than an action-name allowlist.

| Dimension | Lower risk | Higher risk |
|---|---|---|
| Environment | Development | Production |
| Data | Public or synthetic | Restricted, regulated, credentials, free text |
| Scope | One run or partition | Unbounded range, shared stream, many descendants |
| Reversibility | Pause, shadow output | Append to canonical sink, delete, checkpoint reset |
| Evidence | Current and corroborated | Stale, contradictory, incomplete lineage |
| Semantics | Deterministic retry | Schema rewrite, CDC resnapshot, late-data policy change |
| Consumer impact | No downstream publication | Contracted product with critical consumers |
| Identity | Named operator, active incident | Shared service principal or ambiguous requester |
| Cost | Pre-estimated and capped | Historical scan or fan-out without cap |

Policy should compute a risk tier from these fields. The model can supply evidence references but cannot lower the tier.

Example policy:

~~~text
IF environment = production
AND action = retry
AND target_count = 1
AND prior_effect = definitively_absent
AND error_class IN preapproved_transient_errors
AND data_classification <= internal
AND estimated_cost <= pipeline.retry_budget
THEN require on-call confirmation
ELSE require reviewed plan or deny
~~~

## 5. Authority envelopes

An authorization is bound to a plan, not a conversation.

~~~yaml
authorization:
  decision_id: pd_019...
  plan_hash: sha256:...
  subject: oncall@example.com
  tenant_id: retail-eu
  environment: production
  adapter: airflow-prod
  operations: [trigger_dag_run]
  resources: [orders_hourly]
  interval:
    start: 2026-08-28T03:00:00Z
    end: 2026-08-28T04:00:00Z
  max_remote_runs: 1
  max_estimated_cost_usd: 35
  expires_at: 2026-08-31T14:15:00Z
  approvers: [incident-commander@example.com]
~~~

The executor rejects a changed hash, expired approval, wider interval, different adapter, or alternate tenant. Free-text approval such as “go ahead” is not an authorization record.

## 6. Human control points

Require explicit human approval for:

- production backfills above a configured interval, record, cost, or descendant threshold;
- checkpoint resets, CDC snapshots, topic rewinds, or offset changes;
- schema changes classified as breaking or semantically uncertain;
- publication of quarantined data;
- deletes, retention changes, and legal-hold interactions;
- multi-tenant or shared-infrastructure actions;
- actions based on incomplete lineage or conflicting evidence;
- first use of a new adapter operation or upgraded runtime;
- policy override or expansion of an authority envelope.

“Human in the loop” is insufficient if the reviewer receives an opaque summary. Present the exact target, versions, interval, estimated work, expected effect, verification criteria, rollback or forward-recovery plan, and evidence links.

## 7. Bounded autonomy patterns

### Read-only investigator

This is the recommended first release. Tools retrieve normalized metadata, not arbitrary bytes. The output is a cited hypothesis list, uncertainty, blast radius, and next safe checks.

### Proposal generator

The agent creates:

- a backfill or replay plan;
- a contract change proposal;
- a DAG or transformation pull request;
- a quarantine or publication recommendation;
- an incident timeline and evidence bundle.

CI, policy, and humans remain the execution path.

### Supervised operator

Only repetitive, reversible, low-blast-radius effects qualify. Examples include retrying one known transient task or pausing publication after a deterministic quality gate. The effect still uses a stable key, receipt, and independent verifier.

### Emergency circuit breaker

Stopping further harm can justify narrow pre-authorization. The trigger should be deterministic—such as cross-tenant writes or contract-violating output—not a model confidence score. Resume requires a separate decision.

## 8. Alternatives and when they win

| Alternative | Prefer it when | Limitation |
|---|---|---|
| Scheduler retry policy | Failure is known and transient | Cannot synthesize cross-system evidence |
| Static runbook automation | Recovery steps are stable and fully deterministic | Brittle when topology or semantics vary |
| Data observability product | Main need is detection, lineage, and incident routing | May not provide governed recovery execution |
| Platform-specific copilot | Estate is concentrated in one vendor | Cross-system receipts and policy can fragment |
| General tool-using agent | Exploration is in a sandbox | Too much authority and too little semantic control for production |
| Human-only operations | Frequency is low or consequences are extreme | Slower and may produce inconsistent evidence |

The blueprint composes these systems. It does not require replacing them.

## 9. Acceptance criteria for scope

A candidate use case is ready when:

- the owner, target system, tenant, and environment are explicit;
- success is machine-verifiable;
- inputs and outputs have stable identifiers or frontiers;
- versions needed for replay can be recovered;
- an effect can be made idempotent or reconciled;
- a safe stop or escalation exists;
- access can be scoped below “platform administrator”;
- representative failure fixtures can be built;
- a deterministic baseline exists for comparison.

Reject or defer it when the desired result is subjective, the source of truth is unclear, the only tool is arbitrary shell/SQL, or “success” is merely that a remote request returned.

## 10. Selected sources

- [Airflow security model](https://airflow.apache.org/docs/apache-airflow/stable/security/security_model.html)
- [Airflow stable public interface](https://airflow.apache.org/docs/apache-airflow/stable/public-airflow-interface.html)
- [NIST SP 800-53 Rev. 5.1 controls](https://csrc.nist.gov/CSRC/media/Projects/risk-management/800-53%20Downloads/800-53r5/SP_800-53_v5_1-derived-OSCAL.pdf)
- [OWASP AI Agent Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html)
- [Google SRE Workbook: Data Processing Pipelines](https://sre.google/workbook/data-processing/)

# Mission, Boundary, Requirements, and Authority

## Mission

The agent helps accountable model and release owners answer a narrow operational question:

> Is this exact model/prompt release eligible for this exact target, what evidence is missing, how should exposure be controlled, and what should happen if observed behavior degrades?

It closes an evidence-and-control loop across registry, lineage, evaluation, serving, monitoring, and recovery. It does not create its own product objective or treat higher offline scores as permission to change production.

## Real-agent qualification

Use model-directed reasoning only for decisions that cannot be expressed cleanly as fixed predicates, such as:

- selecting the minimum additional evidence needed when a lineage graph is incomplete;
- comparing trade-offs across several non-dominating evaluation dimensions;
- forming bounded hypotheses for a serving degradation from multiple telemetry sources;
- selecting one approved diagnostic or rollout playbook based on current evidence;
- explaining why a candidate is ineligible without weakening the gate.

Keep schema checks, signatures, metric calculations, threshold comparisons, risk classification, approval validity, authorization, traffic weights, and effect reconciliation deterministic.

| Candidate workload | Agent justified? | Better design |
|---|---:|---|
| Promote when a fixed test suite passes | No | CI rule plus registry/deployment controller |
| Copy an approved immutable model between registries | No | Idempotent promotion job |
| Triage a candidate with conflicting quality, latency, lineage, and drift evidence | Yes, bounded | Agent proposes; policy/owner decides |
| Decide whether to retrain and redefine the target from a drift alert | No | Model/data/product governance process |
| Diagnose whether a canary regression is model, feature, serving, or traffic related | Yes, bounded | Read-only agent plus incident handoff |

## Owned and excluded outcomes

### Owned

- release-candidate completeness and eligibility evidence;
- model, prompt, evaluator, serving bundle, and policy version binding;
- source run, code, dataset, feature, and deployment lineage references;
- offline, replay, shadow, canary, and delayed-label evaluation artifacts;
- proposed traffic plan, exposure budget, gates, pause, abort, and recovery evidence;
- registry/deployment reconciliation and model-operations audit history;
- monitoring coverage, baseline freshness, signal quality, and drift alert routing.

### Explicitly excluded

- building or repairing ingestion/transformation pipelines;
- inventing labels, changing population definitions, or interpreting business causality;
- provisioning clusters, networks, GPUs, secrets infrastructure, or serving platforms;
- ordinary application build/release work not specific to model behavior and lineage;
- incident command across services;
- unconstrained hyperparameter search, retraining, fine-tuning, or data acquisition;
- changing intended use, protected-group policy, risk appetite, or compliance classification;
- deleting source evidence or suppressing inconvenient monitoring results.

## Actors and accountability

| Actor | Accountable for | Must not delegate to the agent |
|---|---|---|
| Model owner | Intended use, metric meaning, candidate quality, retraining decision | Acceptance criteria or substantial model-purpose change |
| Data/feature owner | Dataset contracts, labels, feature correctness, lineage repair | Declaring stale/unknown data trustworthy |
| Release manager | Release window, target, exposure and rollback decision | High-impact production commit approval |
| Platform owner | Serving runtime, capacity, isolation, adapter safety | Infrastructure or cluster break-glass |
| Risk/compliance owner | High-risk controls, retention, protected slices, documentation | Waiving a mandatory control for one release |
| SRE/incident commander | Live incident priorities and cross-service mitigation | Incident command or emergency override |
| Agent service owner | Runtime, tools, evals, SLOs, security, cost | Self-approval of agent releases |

Final accountability stays with named humans even when a deterministic policy automatically advances a pre-approved canary step.

## Workload taxonomy

| Workload | Time shape | Main evidence | Safe completion |
|---|---|---|---|
| Candidate intake | Minutes | Manifest, signatures, registry state, lineage | Eligible, rejected, or missing-evidence report |
| Offline qualification | Minutes to hours | Versioned datasets, evaluators, slice metrics, safety tests | Signed evaluation bundle bound to candidate |
| Serving compatibility | Minutes to hours | Load test, inference contract, resource profile, image/model scan | Compatible or ineligible artifact |
| Production rollout | Minutes to days | Traffic, service health, model behavior, business guardrails | Promoted, paused, aborted, rolled back, or indeterminate |
| Drift investigation | Hours to days | Reference/current windows, labels, schema, feature freshness | Classified signal and owner-approved response |
| Provider/model migration | Days to months | lifecycle notice, contract diffs, full eval matrix | Approved replacement or explicit retirement plan |

## Functional requirements

1. Resolve every mutable identifier to an immutable subject before evaluation or approval.
2. Verify the release manifest, signatures/attestations, source lineage, license/policy metadata, and serving compatibility.
3. Run or ingest evaluation tasks without allowing the candidate to modify its own grader, reference data, or acceptance rule.
4. Compare candidate and champion by declared slices, uncertainty, severe failures, cost, and latency—not one aggregate score.
5. Produce a sealed rollout proposal with target revision, exposure budget, gates, observation windows, and recovery target.
6. Pause durably for approvals and delayed evidence.
7. Execute only exact typed effects through scoped identities and reconcile ambiguous outcomes.
8. Track which release actually served each observation without putting high-cardinality raw IDs into unbounded metrics.
9. Distinguish infrastructure health, model behavior, data drift, quality, safety, and business outcomes.
10. Preserve evidence for audit, incident review, rollback, and future regression suites under retention policy.

## Quality requirements

| Property | Requirement |
|---|---|
| Correctness | Eligibility and terminal state derive from authoritative records, never model prose |
| Safety | Forbidden promotion, cross-tenant reads, unsigned artifacts, and stale approvals are hard failures |
| Reliability | Duplicate delivery and crash recovery create at most one semantic traffic/registry effect |
| Explainability | Every proposal cites evidence IDs, gate versions, missing facts, and owner decisions |
| Latency | Interactive intake reports first useful progress; long evaluation/rollout becomes durable asynchronous work |
| Cost | Cost is attributed per candidate, evaluation suite, rollout, tenant, and verified result |
| Privacy | Prediction/label samples and prompts are minimized, classified, redacted, and separately retained |
| Portability | Application state is independent of framework IDs and mutable registry aliases |
| Evolvability | Model, prompt, evaluator, schema, adapter, policy, and telemetry versions are explicit |

## Risk classes

| Risk | Examples | Default treatment |
|---|---|---|
| R0 observe | Read registry metadata, metrics, lineage references | Autonomous, tenant-scoped, rate-limited |
| R1 prepare | Create manifest, evaluation job, shadow plan, ticket | Autonomous proposal; no production traffic |
| R2 non-production effect | Load model in isolated test endpoint, run replay | Pre-authorized capability and resource budget |
| R3 bounded production | Start/advance low-exposure canary, pause traffic | Exact approval; deterministic gates; immediate kill switch |
| R4 high impact | Full promotion, rollback affecting regulated decisions, baseline/policy change | Named multi-role approval and specialist runbook |
| R5 prohibited | Delete lineage, bypass gates, self-approve, use break-glass, unconstrained retrain | Outside the agent capability registry |

Risk rises with irreversible downstream consumption, affected population, missing labels, weak rollback, cross-region data movement, novel model format, new provider, policy change, active incident, or incomplete monitoring.

## Authority matrix

| Capability | Read/write | Reversible? | Identity | Approval | Agent role |
|---|---|---:|---|---|---|
| Registry/lineage query | Read | Yes | Tenant-scoped reader | None for normal scope | Execute |
| Evaluation job creation | Write job/artifact | Usually | Evaluation-runner identity | Policy for sensitive dataset | Prepare/execute within budget |
| Candidate quarantine tag | Write metadata | Yes | Registry metadata writer | Pre-authorized for verified violations | Propose or execute low-risk |
| Prompt/model alias move | Write mutable pointer | Yes but externally visible | Separate release identity | Exact production approval | Never directly; gateway commits |
| Serving revision creation | Write capacity | Reversible with cost | Target-scoped deploy identity | Non-prod policy or prod approval | Propose; gateway executes |
| Traffic weight change | Write live routing | Reversible, exposure already occurred | Rollout identity | Bound envelope | Deterministic controller only |
| Pause/abort | Write routing/control | Yes | Rollout identity | Pre-authorized containment | Agent may request; policy commits |
| High-impact rollback | Write live routing/registry | Partly; past outputs remain | Incident/release identity | Explicit named approval | Propose only |
| Baseline/threshold change | Policy write | Reversible but audit-sensitive | Policy-admin identity | Independent review | Outside runtime agent |
| Retraining trigger | Starts compute/data use | Not a release reversal | Training-system identity | Model/data owner | Recommendation only |

## Admission contract

```yaml
request:
  request_id: req_01K...
  tenant_id: tenant_ref              # trusted gateway value
  principal_id: principal_ref        # trusted gateway value
  operation: qualify_and_propose_release
  candidate_ref: registry://fraud-risk/versions/184
  target_ref: serving://prod-eu/fraud-risk
  declared_intended_use: fraud_triage_v3
  maximum_authority: propose_prod
  budgets:
    deadline_seconds: 7200
    model_calls: 12
    tool_calls: 80
    evaluation_compute_usd: 50
  requested_by: model_owner
```

The gateway resolves `candidate_ref` and `target_ref` under the authenticated tenant. Model-supplied tenant, role, credentials, or risk class are ignored.

## Normal and exceptional journeys

### Normal release

1. Resolve the candidate to digests and ingest its manifest.
2. Verify lineage, source/build attestations, licenses, model format, feature contract, and evaluation inputs.
3. Run deterministic and model/domain evaluations; compare with the current champion and policy floors.
4. Run isolated serving compatibility and load tests.
5. Seal the rollout proposal and wait for the required approval.
6. Revalidate current state, start shadow/canary, and let deterministic gates advance or pause.
7. Verify serving binding, registry pointer, monitoring coverage, and post-release window.

### Missing evidence

The agent identifies the smallest missing set and routes each item to its owner. It never fabricates a dataset version, converts an old evaluation into a current pass, or marks an unverifiable artifact as trusted.

### Cancellation

Cancellation stops new model/tool steps, revokes pending capability grants, requests rollout pause where possible, and reconciles any already-dispatched operation. `cancel_requested` is not terminal until live traffic and registry state are known.

### Drift or degradation

The agent classifies the observation: schema/data quality, feature freshness, covariate shift, prediction shift, delayed-label quality, fairness/safety, serving health, or business outcome. It proposes an approved playbook and preserves uncertainty. Retraining is a human-owned downstream decision.

## Anti-patterns

- A chatbot with registry administrator and cluster credentials.
- “Promote the best metric” without declared slices, uncertainty, or business/safety floors.
- Using a registry alias as the immutable subject of approval.
- Treating every drift statistic as model failure or retraining permission.
- Auto-rollback to a model whose feature/runtime contract is no longer compatible.
- Reusing production predictions as training labels without feedback-loop analysis.
- Letting the same identity build, evaluate, approve, deploy, and verify.
- Hiding missing labels or unscored traffic from the quality denominator.
- One global model-operations agent spanning tenants and regions with ambient credentials.

## Boundary acceptance checklist

- [ ] At least one decision genuinely needs bounded model synthesis; deterministic alternatives were measured first.
- [ ] Each effect class has a human owner, risk, precondition, approval, receipt, and recovery rule.
- [ ] DataOps, infrastructure, DevOps, analytics/product, SRE, risk, and training handoffs are documented.
- [ ] Retraining, threshold changes, high-impact promotion, and high-impact rollback remain human-governed.
- [ ] Completion is an authoritative state/evidence result, not a narrative answer.
- [ ] Tenant, intended-use, target, model, prompt, evaluator, dataset, feature, and policy identity are explicit.

## Related guides

- [Reference architecture, tooling, and integrations](02-reference-architecture-tooling-and-integrations.md)
- [Security, permissions, approvals, reliability, and reconciliation](06-security-permissions-approvals-reliability-and-reconciliation.md)
- [Zero-to-production stages and exit gates](09-zero-to-production-stages-and-exit-gates.md)


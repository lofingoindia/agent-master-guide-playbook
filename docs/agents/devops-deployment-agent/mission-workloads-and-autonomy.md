# Mission, Workloads, and Autonomy

> **Status:** Production design guide  
> **Last researched:** 2026-08-31  
> **Decision:** Define the agent's product boundary, workload classes, and maximum authority before selecting a framework or model.  
> **Evidence:** [Research packet](../../research/packets/devops-deployment-agent-blueprint.md)

## Mission

The agent reduces the cognitive and coordination cost of safe software delivery. It should:

- assemble fresh evidence from repositories, pipelines, registries, environments, tickets, and incidents;
- turn human intent into a typed, reviewable change proposal;
- generate or retrieve deterministic diffs, plans, policies, and rollout options;
- explain risk, uncertainty, missing evidence, and recovery choices;
- create a pull request, change record, or bounded execution request;
- wait durably for external checks, approvals, windows, and controllers;
- supervise a rollout using deterministic gates;
- pause, recommend recovery, or execute a pre-authorized recovery action;
- leave a complete causal evidence trail.

Its primary output is not prose. It is an evidence-bound change state whose human summary is one view.

## Non-goals

The agent should not:

- replace source control, CI, artifact registries, GitOps, IaC state, deployment controllers, policy engines, secrets managers, or incident command;
- accept production target identity, tenant identity, or credentials from free-form user text;
- deploy arbitrary code or infrastructure with a general shell;
- invent a missing artifact, approval, test result, or change-window status;
- approve its own proposal or silently satisfy separation-of-duties rules;
- reinterpret policy denial as a suggestion;
- claim success from command exit text without authoritative status and postconditions;
- blind-retry a possibly committed operation;
- promise rollback safety without compatibility and state evidence;
- optimize for deployment count at the expense of user impact or operator load;
- pursue maximum autonomy as an end in itself.

## Work ownership and handoff rules

The agent coordinates delivery; it does not absorb every neighboring operational role.

| Boundary | This agent may | It must not |
|---|---|---|
| Coding | Request a build, consume a reviewed revision, or propose a desired-state/config PR | Make unreviewed product-code changes or treat its own test run as independent release approval |
| Infrastructure | Consume an approved target and sealed infrastructure plan; prepare/apply an exact application-owned IaC release only when that module is explicitly delegated; report delivery-blocking capacity or drift | Author or repair general cloud, network, IAM, storage, host, or cluster infrastructure through deployment credentials |
| SRE and incident response | Supply release evidence, freeze routine rollout, and execute a narrowly delegated recovery effect | Declare itself incident commander, compete with responder mutations, or optimize rollout over containment |
| MLOps/model operations | Deliver an immutable serving bundle already authorized by model-release policy | Choose a model champion, redefine an evaluation baseline, trigger retraining, or infer delayed model quality from application health |

Cross-boundary work becomes a chain of independently identified effects. Each system retains its own policy, approver, credential, operation ID, verification, and recovery semantics.

## Workload taxonomy

Different tasks need different deadlines, evidence, isolation, and authority.

| Workload | Typical duration | Effects | Default autonomy | Key SLO |
|---|---:|---|---|---|
| Release question | Seconds–minutes | None | A1 observe | Fresh, cited answer latency |
| Failed-pipeline diagnosis | Minutes | None or retry proposal | A1/A2 | Correct failure class and next action |
| Dependency/config update | Minutes–hours | Branch/PR | A2 | Review acceptance and no hidden scope |
| Preview environment | Minutes–hours | Ephemeral non-prod | A3 | Provision success, TTL cleanup, cost cap |
| Standard staging promotion | Minutes–hours | Reversible non-prod | A3 | Same digest, test completion, convergence |
| Standard production release | Minutes–hours | Customer-facing | A4/A5 bounded | Healthy exposure within change/SLO budget |
| Application-coupled IaC promotion | Minutes–days | Potentially destructive | A2; A4 only for explicitly delegated narrow classes | Exact-plan integrity and no surprise changes |
| Data/schema migration | Hours–days | Stateful, often irreversible | A2/A4 with specialized workflow | Compatibility, checkpoint, recovery evidence |
| Incident diagnosis | Minutes | Read-heavy | A1/A2 | Time to useful evidence and mitigation proposal |
| Incident containment | Seconds–minutes | High impact | Dedicated runbook authority | Time to contain, command ownership, audit |
| Fleet maintenance | Hours–days | Broad repeated effects | Policy automation with agent supervision | Bounded concurrency, cohort health, recovery |

Do not queue an interactive incident query behind a fleet rollout, or let a background maintenance run consume the same production-write quota as a rollback.

## Risk classification

Classify the proposed semantic effect, not the tool name. `kubectl`, Terraform, Git, or a cloud SDK can each perform both low- and high-risk operations.

| Risk | Examples | Required posture |
|---|---|---|
| R0 read-only | Get rollout status, fetch logs, resolve digest | Scoped read identity; output and privacy limits |
| R1 reversible metadata | Draft PR, annotate deployment, update non-authoritative ticket field | Idempotent key; bounded scope; audit |
| R2 non-production effect | Create preview namespace, run staging deployment | Environment policy; cleanup TTL; budget; verified target |
| R3 bounded production effect | Promote pre-verified digest via approved rollout template | Exact plan and approval; deterministic gates; recovery target |
| R4 broad or destructive | Delete resources, replace network/IAM, rotate trust, mass rollback | Independent expert review; narrow runbook; often human execution |
| R5 irreversible/stateful | Destructive schema/data migration, key destruction | Specialized procedure, backups/checkpoints, explicit executive/domain authority |

Risk increases with blast radius, irreversibility, target sensitivity, novelty, concurrent change, data movement, credential reach, weak observability, and uncertain recovery. The model may propose a risk class, but deterministic rules establish the minimum.

## Autonomy model

### A0 — explain from supplied context

No external tools. Useful for design review and onboarding, not live operations.

### A1 — observe

Read-only, tenant-scoped adapters retrieve authoritative evidence. The agent can diagnose and recommend, but cannot create external state.

Entry requirements:

- adapter allowlists and result redaction;
- freshness and provenance on every observation;
- rate, byte, token, and query limits;
- test coverage for prompt injection in logs, issues, diffs, and manifests.

### A2 — propose

The agent may create a local patch, a draft branch/PR, a plan, a change-ticket draft, or an unsigned release candidate. It cannot merge, approve, promote, or deploy production.

This is the best default for most organizations because it creates value while preserving existing review and deployment controls.

### A3 — execute bounded non-production changes

The agent may create or update explicitly non-production resources under templates and budgets.

Additional requirements:

- environment registry says the target is non-production;
- automatic cleanup and ownership labels;
- no production credentials or network routes;
- idempotency and reconciliation for provision/deploy/delete;
- tenant quotas and per-run cost ceiling;
- escalation rather than target substitution on capacity failure.

### A4 — initiate exact approved production changes

The agent may commit an operation only when an external policy/approval service authorizes the exact subject, plan, target, and version.

The agent does not decide that a vague approval is “close enough.” Trusted code compares hashes, checks expiry, re-resolves the target, and obtains a short-lived grant.

### A5 — supervise a pre-authorized production envelope

The agent may choose or invoke `advance`, `pause`, `abort`, `traffic_revert`, or another small action set while a rollout is active. Deterministic gates still enforce maximum traffic, SLO floors, sample requirements, and time limits.

A5 is appropriate only for standardized services with strong observability and rehearsed recovery. It is not permission to edit infrastructure, policy, credentials, or schemas during the rollout.

## Grant autonomy per cell

Use an authority matrix rather than a global setting:

| Effect class | Dev | Staging | Production | Incident mode |
|---|---:|---:|---:|---:|
| Read evidence | A1 | A1 | A1 | A1 |
| Create PR | A2 | A2 | A2 | A2 |
| Deploy immutable app artifact | A3 | A3 | A4/A5 | Frozen or incident-delegated |
| Change low-risk configuration | A3 | A3 | A4 | Frozen by default |
| Apply infrastructure create/update | A3 template | A4 | A2/A4 narrow | Incident-delegated only |
| Delete infrastructure | A3 ephemeral only | A2/A4 | A2 | Dedicated runbook |
| Run schema migration | A2/A3 reversible | A4 | A2/A4 specialized | Incident commander decision |
| Change IAM/policy/trust root | A2 | A2 | A2 | Break-glass human |

`A2` in a cell can be the deliberate ceiling. A mature design may permanently keep identity changes proposal-only.

## Admission contract

Before the model sees the task, deterministic admission should resolve:

```yaml
admission:
  principal_id: usr_1042
  tenant_id: ten_acme
  request_id: req_01K...
  workload: production_release
  allowed_environments: [env_prod_in_west]
  allowed_effect_classes: [deploy_existing_release, pause_rollout, abort_rollout]
  autonomy_ceiling: A4
  deadline: 2026-08-31T18:00:00Z
  budgets:
    model_calls: 12
    tool_calls: 80
    wall_time: PT90M
    rollout_exposure_percent: 25
  incident_mode: false
  data_policy: dp_internal_operational
```

The context builder exposes logical IDs and descriptions. The user or model cannot expand this envelope by typing another account, cluster, namespace, or role.

## Novelty and confidence

Do not use model confidence alone. Gate autonomy using observable novelty:

- unseen service, target, tool version, or rollout template;
- first deployment after a schema or platform upgrade;
- plan contains resource types or actions absent from the approved class;
- policy or approval service changed since plan generation;
- telemetry is missing, low-volume, or disagrees across sources;
- concurrent change or active incident affects the same dependency;
- recovery target is unavailable or not recently verified;
- agent/tool behavior falls outside evaluation coverage.

Novelty should lower authority, require more evidence, or route to a human. It should never trigger experimentation in production.

## User and operator experience

A useful proposal view answers, in order:

1. **What changes?** Exact source, artifact, configuration, and target.
2. **Why now?** Requested outcome and linked change/ticket.
3. **What evidence passed?** Tests, provenance, scans, policy, and environment readiness.
4. **What can go wrong?** Blast radius, compatibility, missing signals, and concurrency.
5. **How will exposure progress?** Rollout stages and deterministic gates.
6. **How will we recover?** Tested target and state/data limitations.
7. **What decision is requested?** Approve, reject, edit, or request evidence.

Avoid approval screens that show only a generated narrative. Render a stable machine plan and a concise summary derived from it.

## Failure behavior by workload

| Condition | Read/diagnose | Proposal | Non-prod execute | Production execute |
|---|---|---|---|---|
| Evidence source unavailable | State what is missing | Draft but mark unverified | Pause | Fail closed before commit |
| Model unavailable | Return collected evidence | Preserve run for retry | Do not start new effect | Rollout controller continues its deterministic policy |
| Policy unavailable | Continue safe reads | Preserve proposal | Risk-based pause | Fail closed; break-glass external |
| Approval expires | N/A | Refresh plan | Request new decision | Reject commit |
| Target changes after plan | Refresh evidence | Regenerate proposal | Replan | Reject and require reapproval |
| Deployment response lost | N/A | N/A | Reconcile operation ID | Mark unknown and reconcile; no blind retry |
| Telemetry missing during rollout | Report gap | N/A | Pause or time out | Pause/rollback per versioned missing-data policy |
| Incident declared | Switch to incident evidence | Mark conflict | Freeze by policy | Stop progression and yield to incident command |

## Acceptance criteria for each autonomy increase

- [ ] The exact effect classes and targets are enumerated, not implied.
- [ ] Denied and out-of-scope attempts are tested with adversarial inputs.
- [ ] Every effect has idempotency, cancellation, reconciliation, and postcondition semantics.
- [ ] Approval is external, exact, expiring, and invalidated by material change.
- [ ] Credential issuance is short-lived and target-bound.
- [ ] A kill switch outside the agent path works during queue, model, and policy failures.
- [ ] Historical and synthetic scenarios cover safe success, refusal, ambiguity, and recovery.
- [ ] Shadow results meet thresholds per workload and risk class.
- [ ] Human workload, false blocks, and approval fatigue are measured.
- [ ] Incident command and tenant-isolation drills pass.

## Related guides

- [Reference architecture and build choices](reference-architecture-and-build-choices.md)
- [Plans, policy, approvals, and change control](plans-policy-approvals-and-change-control.md)
- [Context compilation, memory, and behavior evolution](context-compilation-memory-and-behavior-evolution.md)
- [Run controls](../../runtime/run-controls.md)
- [Execution boundaries](../../runtime/execution-boundaries.md)
- [Agent threat model](../../security/agent-threat-model.md)

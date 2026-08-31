# Observability, Evaluation, and Failure Testing

> **Status:** Research-backed production blueprint  
> **Last researched:** 2026-08-31  
> **Scope:** Audit lineage, telemetry, debugging, evaluations, fault injection, and acceptance gates  
> **Research packet:** [Infrastructure Operations Agent research packet](../../research/packets/infrastructure-operations-agent-blueprint.md)

## Production position

The system needs two related but distinct records:

- an **audit record** that proves identities, decisions, credentials, and effects;
- an **engineering trace** that explains latency, model behavior, tool calls, retries, and failures.

Neither should be a raw dump of prompts, secrets, commands, or provider responses. Audit must remain useful when model tracing is disabled, and security-relevant provider events must be exported beyond default retention.

## Correlation model

```mermaid
flowchart LR
    REQ[Request ID] --> RUN[Run/workflow ID]
    RUN --> PLAN[Plan ID + digest]
    PLAN --> DEC[Policy and approval IDs]
    RUN --> OP[Operation ID]
    OP --> ATT[Attempt ID]
    ATT --> LEASE[Credential lease ID]
    ATT --> PROVIDER[Provider request/job ID]
    PROVIDER --> TARGET[Target observation IDs]
    OP --> VERIFY[Verification ID]
    REQ & RUN & PLAN & OP & PROVIDER & VERIFY --> AUDIT[Audit projection]
    REQ & RUN & OP & ATT --> TRACE[Engineering trace]
```

Use stable IDs across queues, workflows, adapters, provider tags/session names, logs, metrics exemplars, artifacts, and operator output.

## Minimum audit events

| Event | Required fields |
|---|---|
| request.accepted | requester, tenant, source, purpose, requested scope |
| evidence.collected | source, resource IDs, observed-at, coverage, artifact refs |
| proposal.generated | model/release, context/evidence IDs, output digest |
| plan.sealed | plan/target/diff digests, tool/policy/inventory versions, risk |
| policy.decided | principal, decision, obligations, policy revision, reason codes |
| approval.decided | approver, role, plan digest, decision, expiry, conditions |
| credential.issued | workload identity, audience, scope digest, TTL, lease ID |
| effect.dispatched | operation/attempt, target generation, tool digest, budgets |
| provider.acknowledged | provider request/job ID and provider identity |
| effect.reconciled | evidence, disposition, ambiguity resolution |
| postcondition.evaluated | check version, observations, pass/fail/unknown |
| rollout.stopped | triggering budget/signal and targets affected |
| recovery.executed | compensate/restore/roll-forward details |
| breakglass.observed | human identity, provider event, incident and review |

Security audit is append only from the application's perspective and exported to a separately administered boundary.

## Trace structure

A trace may contain spans for:

- intent normalization;
- tenant and authorization lookup;
- inventory query and context assembly;
- model invocation;
- schema/semantic validation;
- risk and policy evaluation;
- approval wait;
- credential issuance;
- adapter queue, dispatch, poll, and reconciliation;
- postcondition checks;
- artifact writing and audit export.

### Span attributes

Prefer identifiers and classifications over raw content:

```yaml
infra.operation.id: op_...
infra.plan.digest: sha256:...
infra.tenant.id: t-acme
infra.environment: production
infra.provider: kubernetes
infra.target.type: apps/v1/Deployment
infra.target.count: 1
infra.effect.class: mutate
infra.risk.class: R2
infra.tool.name: kubernetes.restart_workload
infra.tool.version: 2.3.1
infra.workflow.generation: 7
infra.disposition: verified
gen_ai.provider.name: redacted-or-approved-value
gen_ai.request.model: pinned-model-id
```

OpenTelemetry semantic conventions were at 1.44.0 when researched. The GenAI conventions moved to a separate OpenTelemetry repository and their agent/tool attributes remain developmental. They can expose prompts, tool arguments, or responses. Pin an internal telemetry schema, record both convention versions, and explicitly allowlist emitted attributes rather than adopting new fields automatically.

## Logs and artifacts

- Structure logs with IDs, reason codes, and state transitions.
- Redact known schema paths before serialization.
- Do not log tokens, cookies, private keys, kubeconfigs, authorization headers, full Terraform plans, unbounded diff, or PowerShell/SSH environment dumps.
- Store bounded provider/command output as encrypted artifacts with classification, checksum, retention, and access audit.
- Keep operator-safe projections separate from forensic artifacts.
- Mark truncation and redaction so an operator does not interpret an incomplete result as complete.
- Hash sensitive stable values only when correlation is necessary and the HMAC key is separately protected.

## Provider audit correlation

| Provider/boundary | Correlation evidence | Caveat |
|---|---|---|
| AWS | CloudTrail event, STS role session name/tags, request ID, SSM command/automation ID | Event History is 90 days and management-event focused; tunneled SSH/port-forward content is not Session Manager logged |
| Azure | Activity Log, correlation/request ID, managed identity/service principal, Run Command operation | Data-plane logging is service-specific; managed-identity authorization changes may lag token caches |
| Google Cloud | Cloud Audit Logs principal chain, request metadata, operation ID | Data Access logs often require explicit configuration |
| Kubernetes | API audit event, user/service account, verb/resource/UID, audit ID | Audit requires a policy; higher levels can record sensitive bodies and cost volume |
| OpenSSH | CA serial/key ID, principal, gateway session, host audit | Target logs can be tampered with; forwarding/content require explicit controls |
| WinRM/JEA | user, endpoint, transcript/module logging, host event | Target-local evidence is not independent; protect centralized export |

Provider audit proves a control-plane call, not necessarily application health. Join it with independent postconditions.

## Metrics and SLOs

### Safety and correctness

- unauthorized effect attempts denied;
- cross-tenant access attempts;
- stale-plan and target-generation rejections;
- duplicate operation attempts and harmful duplicate effects;
- uncertain outcomes by age;
- verification failures after provider success;
- rollouts stopped by each budget;
- effects missing provider-audit correlation;
- secrets detected in telemetry/artifacts;
- break-glass usage and overdue tests.

### Reliability

- workflow transition and queue latency;
- credential broker availability/issuance latency;
- provider/API success, throttle, timeout, and reconcile latency;
- retry amplification ratio;
- inventory freshness and coverage by authority;
- audit export lag/gap;
- postcondition completion time;
- cancellation acknowledgement and in-flight drain time.

### Model and planning

- structured-output validity/repair rate;
- evidence citation accuracy;
- unsupported inference and abstention rate;
- plan risk-class disagreement with reference;
- target-selection precision/recall;
- unsafe-tool proposal rate;
- prompt-injection success rate;
- tokens, context size, latency, and cost per phase.

### Example objectives

| Objective | Example target | Notes |
|---|---:|---|
| Read request availability | 99.9% | May use explicitly stale cached data |
| New supervised write orchestration | 99.5% | Safety dependencies fail closed |
| Durable pre-dispatch record | 100% | Invariant, not error-budgeted |
| Harmful duplicate effect | 0 | Invariant |
| Provider-audit correlation | 99.9% | Document unavoidable provider gaps |
| Inventory freshness compliance | 99% per authority | Stale results are labeled and writes blocked |
| Uncertain R1/R2 effect reconciled | 99% within class deadline | Escalate the remainder |

### SLI definitions

Define each denominator and exclusion so dashboards cannot hide unsafe work:

```text
pre_dispatch_durability = effects_with_prepared_record_before_first_dispatch
                        / all_first_dispatches

audit_correlation       = verified_effects_with_required_provider_audit_evidence
                        / verified_effects_where_provider_contract_requires_evidence

freshness_compliance    = eligible_inventory_queries_with_all_required_fields_fresh
                        / inventory_queries_used_for_plan_or_commit

reconcile_deadline      = uncertain_effects_reaching_proven_or_escalated_disposition_in_deadline
                        / uncertain_effects_entering_the_class
```

Do not exclude provider outages, restored workflows, partial batches, or missing telemetry unless the published SLI explicitly says so. Safety invariants page immediately; availability SLOs may consume an error budget. Slice all objectives by tenant, cell, provider, adapter/tool version, risk, and operation class so fleet averages do not hide one unsafe boundary.

## Debugging view

An operator should see a timeline, not hidden chain-of-thought:

1. authenticated request and scope;
2. evidence with freshness and coverage;
3. proposed plan and cited evidence;
4. deterministic validation/risk results;
5. policy and approval decisions with reason codes;
6. credential scope;
7. effect attempts and provider IDs;
8. target and service verification;
9. retries, ambiguity, stop signals, and recovery;
10. final disposition and remaining manual actions.

Expose concise model rationale or summary only when useful. Store neither private reasoning nor sensitive raw context as a substitute for evidence.

## Evaluation program

### Dataset layers

| Layer | Cases |
|---|---|
| Contract | Valid/invalid schemas, boundary values, version compatibility |
| Planning | Golden incidents/changes with authoritative evidence and expected plans |
| Security | Direct/indirect injection, tenant confusion, target substitution, secret bait |
| Provider semantics | Throttling, pagination, stale versions, asynchronous operations |
| Reliability | Crashes, duplicates, reordering, timeout after dispatch, policy loss |
| Human factors | Diff comprehension, approval binding, warning salience, cancellation understanding |
| Live shadow | Production reads and recommendations compared with operator decisions |

Split by provider, resource type, risk class, tenant/environment, and failure mode. Hold out recent incidents and adversarial cases to detect overfitting.

### Model evaluation rubric

Score:

- correct target set and no cross-scope targets;
- evidence IDs support every material claim;
- declared uncertainty when coverage is inadequate;
- minimum-effect plan and desired-state owner selection;
- correct provider/tool contract and version;
- risk, blast-radius, maintenance, and recovery completeness;
- no invented APIs, fields, permissions, or outcomes;
- refusal/escalation for prohibited operations;
- injection resistance;
- concise operator communication.

Model quality cannot compensate for missing deterministic enforcement. A poor plan should be rejected safely.

### Hostile corpus

Include compositional cases, not only single attacks:

- prompt injection in logs, tickets, CMDB descriptions, Kubernetes annotations, provider errors, SSH banners, PowerShell objects, and tool descriptions;
- Unicode confusables, bidirectional text, oversized values, deep JSON, duplicate keys, schema bombs, and malicious external `$ref` values;
- cross-tenant operation IDs, artifacts, cache keys, signals, task handles, trace baggage, and selector aliases;
- stale-but-plausible inventory combined with target replacement, clock skew, and permission loss;
- forged provider receipt, reordered/delayed audit event, conflicting observations, and compromised-target “healthy” output;
- approval replay after tool/policy/model/context/plan change and ticket-webhook duplication;
- retry storms across SDK/adapter/workflow layers, response loss after commit, and old-worker return after restore;
- controller conflict, forced/pruned GitOps change, Terraform lock contention, and maintenance-window expiry mid-reboot;
- secret bait in plans, diffs, environment, exceptions, traces, transcripts, and model serialization.

Grade both the proposal and the deterministic system response. A safe rejection with a precise reason is a passing outcome even if the model attempted the prohibited action.

## Failure-injection matrix

| Injection | Expected behavior | Evidence |
|---|---|---|
| Duplicate queue delivery | Same operation is deduplicated/reconciled | Ledger contains attempts, one harmful effect maximum |
| Worker crash before dispatch | New worker safely dispatches | Prepared record and unused idempotency key |
| Worker crash after provider accepts | No blind repeat | Uncertain state, provider/audit query, final reconciliation |
| Provider throttles 50% | Jittered bounded retry; no retry storm | Call budget and retry-amplification metrics |
| Policy service down | No new write credentials | Denial/degraded audit |
| Audit exporter down | Documented buffer or write fail-closed | Gap/queue alert and no silent loss |
| Inventory watcher gap | Relist and partial coverage | Checkpoint gap and repaired snapshot |
| Resource replaced after approval | Plan invalidated | UID/generation mismatch |
| Approval delivered twice | One state transition | Decision nonce/idempotency record |
| Window closes mid-run | No new batch; in-flight reconciled | Window stop event and target states |
| Verification metrics absent | Rollout stops unverified | Stop signal and no success claim |
| GitOps reverts direct repair | Ownership conflict detected | Drift loop/desired owner record |
| Malicious host banner | Treated as data, no new tool/target | Injection trace and denied proposal |
| Secret in command output | Artifact redacted/quarantined | Scanner event and clean trace |
| Tenant ID changed on signal | Signal rejected | Authz/audit event |
| Old worker returns after lease expiry | Fenced from new effect | Generation mismatch |
| Broker revocation cache lag | Adapter budget still limits effects | Revocation timing and kill-switch record |
| Break-glass drill without normal IdP | Human recovery works; agent uninvolved | Provider log, alert, drill review |

## Acceptance gates

### Before advisory production

- inventory coverage/freshness dashboards;
- tenant isolation and read authorization tests;
- prompt/output redaction and injection evaluation;
- provenance for every displayed fact;
- model/version release and rollback procedure.

### Before supervised writes

- immutable plans and exact approval binding;
- brokered short-lived credentials;
- typed effect contracts and provider audit correlation;
- timeout-after-dispatch reconciliation;
- canary/budget/circuit-breaker enforcement;
- independent postcondition verification;
- operator cancellation and uncertainty UX;
- disaster recovery and break-glass drill.

### Before any autonomous remediation

- one named, low-risk remediation class;
- production shadow evidence and low false-positive rate;
- tested idempotency/reconciliation under fault injection;
- proven containment across target/fault-domain/tenant budgets;
- automatic stop on verification/SLO loss;
- named owner, on-call, expiry, kill switch, and review cadence;
- no IAM, perimeter, data-destruction, or agent-control changes.

### Quantitative gate template

Set numbers from the estate's risk tolerance, but publish them before evaluation:

| Gate | Required measurement |
|---|---|
| Scope safety | Zero cross-tenant/cross-environment effects in the full hostile and fault corpus |
| Authority safety | Zero credential issuance or dispatch after deny, expiry, stale generation, or invalid approval |
| Duplicate harm | Zero harmful duplicate effects across repeated delivery/crash/timeout trials |
| Unknown handling | 100% ambiguous dispatches enter `uncertain` and block blind retry |
| Containment | Every injected rollout failure stops within the sealed target/disruption/error budget |
| Evidence quality | Thresholds for supported claims, correct abstention, target precision/recall, and freshness use by risk tier |
| Operability | Kill switch, restore, audit-degradation, broker-compromise, and break-glass drills meet their timed objectives |

Never approve a candidate solely on one aggregate model score. Any safety-invariant regression blocks the complete behavioral release.

## Release comparison

Run the same fixed and recent incident corpus against the current and candidate model, prompt, tool schemas, policies, adapters, and context assembler. Block release on any safety regression even if average task success improves. Canary the control-plane release in a non-production cell, then advisory/shadow production, then supervised low-risk work.

## Failure mining and continuous improvement

```mermaid
flowchart LR
    P[Production run or incident] --> C[Candidate failure case]
    C --> T[Human triage and de-identification]
    T --> K{Failure owner}
    K -->|language/retrieval| E[Eval/context change]
    K -->|contract/effect| A[Adapter/runbook change]
    K -->|authority| O[Policy/approval change]
    K -->|signal| V[Verification/observability change]
    E & A & O & V --> R[Fixed + held-out replay]
    R --> S[Shadow/canary release]
    S --> M[Monitor by version and risk]
```

The candidate record includes the minimal authorized evidence, expected safe behavior, actual trajectory, causal control gap, owner, and severity. Remove secrets and tenant data without erasing the failure mechanism. Keep a fixed regression set and a held-out set; otherwise repeated tuning can overfit visible incidents. No raw production transcript, model self-critique, or operator correction publishes directly to memory, policy, prompts, tools, or runbooks.

## Incident and postmortem use

Preserve a time-ordered bundle of plan, decisions, effect ledger, provider events, observations, and release digests. Postmortems should ask:

- which control should have prevented or contained the event;
- whether the model error was causal or merely visible;
- why the plan, approval view, policy, adapter, or verification allowed it;
- whether retry, timeout, or cancellation semantics were misunderstood;
- how to add the scenario to deterministic tests and evaluation sets.

Avoid fixing every incident by adding prompt text. Prefer contract, policy, adapter, or verification controls when the failure was not inherently linguistic.

## Sources

- [OpenTelemetry semantic conventions](https://opentelemetry.io/docs/specs/semconv/)
- [OpenTelemetry generative-AI attributes](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/)
- [OpenTelemetry GenAI conventions move notice](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
- [AWS CloudTrail concepts](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-concepts.html)
- [AWS CloudTrail Event History](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events.html)
- [Google Cloud audit logging configuration](https://cloud.google.com/logging/docs/audit/configure-data-access)
- [Kubernetes auditing](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)
- [OpenAI Agents SDK tracing](https://openai.github.io/openai-agents-python/tracing/)
- [Google SRE Workbook: Postmortem culture](https://sre.google/workbook/postmortem-culture/)
- [NIST SP 800-61 Rev. 3: Incident response](https://csrc.nist.gov/pubs/sp/800/61/r3/final)

## Related guides

- [State, reliability, recovery, and break-glass](07-state-reliability-recovery-and-break-glass.md)
- [Deployment, scaling, cost, and roadmap](09-deployment-scaling-cost-and-roadmap.md)
- [Observability and tracing](../../evaluation/observability-and-tracing.md)

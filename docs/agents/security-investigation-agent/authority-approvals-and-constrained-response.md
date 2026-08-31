# Authority, Approvals, and Constrained Response

> **Status:** Research-backed production blueprint  
> **Last researched:** 2026-08-31  
> **Scope:** Capability tiers, least privilege, approval binding, case/SOAR writes, containment, remediation, idempotency, and reconciliation.  
> **Section index:** [Security investigation and triage agent](README.md)

Response authority changes the nature of the system. A read-only investigator can expose sensitive data; a response-capable agent can also disrupt services, destroy volatile evidence, lock out responders, alert an attacker, or create duplicate effects. Put response behind a physically and logically separate executor with its own identity, policy, and audit.

## Authority ladder

| Tier | Permitted | Forbidden | Human role |
|---|---|---|---|
| 0 — Offline advisory | Read supplied evidence bundle; draft findings | External queries and all writes | Reviews output |
| 1 — Supervised read-only | Bounded source queries through broker | Case writes, notifications, response | Reviews disposition |
| 2 — Case workflow | Append proposed findings, create assigned tasks, prepare playbook steps | Production containment or remediation | Confirms case truth and task ownership |
| 3 — Approved containment | Execute exact reversible, time-bounded containment after approval | Broad or irreversible remediation | Approves effect and rollback |
| 4 — Pre-authorized narrow containment | Execute a scenario-specific action under deterministic high-confidence policy | Novel targets/actions, destructive effects, self-expanded scope | Owns policy and exception review |
| 5 — Remediation orchestration | Coordinate separately approved recovery tasks | Unsupervised destructive recovery | Incident commander and system owner direct |

Tier 4 is not “general autonomy.” It is automation of a previously approved invariant over a tiny action/target set. Tier 5 should normally be a human-directed workflow, not model discretion.

## Combined privilege is the risk

Review the union of capabilities available in one run:

- what evidence can be read;
- what secrets or personal data can be inferred;
- which external destinations are reachable;
- which case or messaging systems can be written;
- which endpoints, identities, network controls, or cloud resources can be changed;
- what durable memory, prompt, tool metadata, or policy can be modified.

A read tool plus a notification or URL-fetch tool can become an exfiltration path. A case-write tool can spread an injected instruction to another analyst or automation. A “temporary isolate” action can still interrupt a safety-critical service.

## Separate identities and credentials

Use distinct workload identities for:

1. intake;
2. evidence storage;
3. read-only query broker;
4. case writer;
5. policy and approval service;
6. each response actuator or action family;
7. offline malware/forensics workers;
8. evaluation.

Required properties:

- short-lived, audience-bound credentials;
- tenant and region scope;
- resource and action scope;
- no refresh or long-lived token in model-visible context or analysis sandbox;
- explicit end-user or operator identity alongside workload identity;
- credential issuance and use logged independently;
- rapid revocation and break-glass owned outside the agent.

Never forward a user's broad bearer token through the agent to downstream services. Mint a narrower capability after policy evaluation.

## Policy decision at execution time

The policy decision point should evaluate canonical facts immediately before commit:

~~~yaml
authorization_request:
  action_type: endpoint.isolate
  action_schema_version: 4
  action_id: act_91
  idempotency_key: case_781:endpoint.isolate:device_44:plan_3
  actor:
    user_id: analyst_17
    workload_id: response-executor
    tenant_id: tenant_acme
  target:
    resource_id: edr:tenant_acme:device:44
    observed_version: etag-7731
    criticality: high
  parameters:
    duration_seconds: 1800
    allow_management_channel: true
  basis:
    case_id: case_781
    case_version: 22
    evidence_refs: [ev_991, ev_1044]
  approval:
    approval_id: apr_52
    expires_at: 2026-08-31T04:40:00Z
  policy_version: soc-response/19
~~~

The decision must verify:

- authenticated actor and workload;
- tenant, region, resource, and action scope;
- target exists and still matches the approved canonical identifier/version;
- action and parameters exactly match the approval;
- evidence prerequisites and case state are current;
- action is allowed for this asset class and incident phase;
- approval has not expired, been revoked, or already consumed;
- maintenance, preservation, legal hold, and safety constraints;
- idempotency and existing-effect state;
- kill switch, rate limit, and global incident mode.

The model-provided severity or confidence is input to policy only if a deterministic, versioned rule explicitly permits it. Never treat model confidence as authorization.

## Approval is a signed intent record

An approval should show the reviewer:

- exact action and target;
- tenant, environment, owner, and criticality;
- parameters and duration;
- evidence and case version;
- expected impact and possible business disruption;
- evidence-preservation implications;
- rollback or expiry;
- alternatives, including doing nothing;
- who is accountable for observing success;
- when the approval expires.

### Invalidate approval when

- target identifier, version, ownership, or criticality changes;
- action arguments change;
- new evidence materially changes the case;
- policy changes;
- approval expires or approver loses authority;
- the incident enters a different phase;
- the target has already reached an unexpected state.

Do not permit “approve all future actions of this kind” from an investigation prompt. Persistent policy changes need their own governance path.

## Action lifecycle

~~~mermaid
stateDiagram-v2
    [*] --> Proposed
    Proposed --> PolicyDenied: deterministic denial
    Proposed --> AwaitingApproval: approval required
    Proposed --> Authorized: pre-authorized policy
    AwaitingApproval --> Authorized: exact approval valid
    AwaitingApproval --> Expired: timeout or state change
    Authorized --> Submitting: executor claims action
    Submitting --> ObservedSucceeded: target confirms desired state
    Submitting --> ObservedFailed: target confirms failure
    Submitting --> OutcomeUnknown: timeout or ambiguous response
    OutcomeUnknown --> Reconciling
    Reconciling --> ObservedSucceeded
    Reconciling --> ObservedFailed
    ObservedSucceeded --> RollingBack: expiry or approved rollback
    RollingBack --> RolledBack
    RollingBack --> RollbackFailed
~~~

Never map a successful HTTP response directly to “succeeded” if the target's state can be queried. Record submitted, accepted, applied, observed, expired, and rolled back separately.

## Action risk classes

| Class | Examples | Default control |
|---|---|---|
| R0: pure/read | Fetch case or telemetry | Broker authorization and audit |
| R1: reversible workflow write | Append proposed note, create draft task | Idempotency, optimistic concurrency, visible provenance |
| R2: reversible bounded containment | Isolate one endpoint for 30 minutes, revoke one session | Human approval, TTL, management channel, reconciliation |
| R3: high-impact containment | Disable account, block shared infrastructure, quarantine workload | Dual approval or incident-command authority; owner consultation |
| R4: destructive or hard-to-reverse remediation | Delete resource, wipe host, rotate shared root, mass block | Human-directed specialist playbook; model cannot execute |
| R5: offensive or out-of-scope | Exploit, persistence, credential capture, counterattack | Prohibited |

Classify the realized effect, not the friendly tool name.

## Case and SOAR writes

SOAR integration is safer when the agent writes proposals into distinct fields:

| Field | Writer | Meaning |
|---|---|---|
| agent_proposed_disposition | Agent via typed writer | Unconfirmed recommendation |
| agent_claims | Agent via typed writer | Evidence-linked claims |
| analyst_disposition | Analyst | Accountable case decision |
| response_plan | Agent or analyst | Proposed typed steps |
| approved_action | Approval service | Exact authorized effect |
| observed_action_outcome | Response executor | Target-observed result |

Do not let the agent write into “confirmed incident,” “root cause,” “contained,” or “closed” fields unless the case system preserves the proposal status and an authorized human or deterministic policy performs the transition.

### Idempotent write

Use a semantic key based on stable intent:

~~~text
tenant + case + action_type + canonical_target + plan_version
~~~

Store the key, canonical arguments, first result, current status, and expiry. Reject reuse with different arguments. A random key created on each retry does not deduplicate the effect.

Use optimistic concurrency on case version so an old model run cannot overwrite newer analyst work.

## CACAO and OpenC2 boundaries

CACAO 2.0 can represent versioned, shareable security playbooks and supports signatures. OpenC2 defines a vendor-agnostic action/target command and response vocabulary. They improve interoperability but do not provide:

- local user or workload authorization;
- incident-specific evidence sufficiency;
- asset-owner consent;
- safe parameter ranges;
- approval semantics;
- idempotency and exactly-once effects;
- rollback feasibility;
- evidence preservation;
- tenant or regional policy.

Treat imported playbooks as untrusted code/configuration. Verify signature and trust chain, pin version and digest, statically validate actions, map every step to a local allowlisted implementation, and test in a non-production environment.

## Containment design

Containment limits ongoing harm while preserving investigation and service continuity. CISA's playbook explicitly calls for selecting a strategy with evidence preservation, service availability, resource constraints, and duration in mind.

### Endpoint isolation

Preconditions may include:

- exact endpoint and tenant;
- EDR sensor healthy and management channel retained;
- owner and criticality known;
- acquisition or volatile-evidence decision recorded;
- no safety-critical or excluded asset tag;
- TTL and automatic release behavior;
- console or alternative access for responders;
- duplicate isolation state reconciled.

Postconditions:

- target reports isolated state;
- permitted management path remains functional;
- isolation expiry is scheduled and monitored;
- analyst and owner are notified through a governed channel;
- case records actual—not intended—state.

### Identity containment

Revoking a session, disabling an identity, removing a role, and rotating a credential are distinct effects. Consider:

- human versus workload identity;
- shared or emergency account;
- active response access;
- downstream token caches and federation;
- automated job failure and service impact;
- whether disabling removes evidence or attacker visibility;
- re-enable or replacement process.

Prefer a narrowly scoped session/token revocation over disabling an entire identity when it addresses the observed risk.

### Network or indicator block

Validate:

- target type and canonical value;
- shared/CDN/cloud infrastructure risk;
- direction, protocol, port, account, and duration;
- existing rule ordering and conflicts;
- propagation delay and rollback;
- business dependency and false-positive history;
- intelligence freshness and local observation.

A domain or IP reputation match alone is not enough for a permanent enterprise block.

## Remediation design

Remediation changes the compromised state or underlying cause. It can destroy evidence and introduce new outages. Keep it human-directed.

The agent may draft:

- affected resource inventory;
- prerequisite backups and evidence acquisitions;
- ordered recovery steps;
- owners and maintenance windows;
- verification queries;
- rollback and contingency;
- known persistence mechanisms or credentials requiring attention;
- detections to monitor after change.

The model must not execute cleanup commands, wipe or reimage hosts, delete accounts/resources, rotate broad credentials, or mass-deploy detection/blocking content.

### Verify outcome independently

An action is complete only when:

- the target state is observed through an authoritative read;
- the intended threat path is reduced;
- critical service health remains acceptable;
- evidence and case records are intact;
- rollback/expiry is scheduled where relevant;
- residual risk and monitoring are assigned.

## Idempotency and ambiguous outcomes

Network timeouts create three states: definitely not attempted, definitely observed, and unknown. On unknown:

1. Do not issue a new random action.
2. Query the actuator or target by action ID, idempotency key, or desired state.
3. Reconcile case and ledger.
4. Retry only if the same semantic key and canonical arguments are safe.
5. Escalate if the target cannot expose outcome and duplication would be harmful.

Retries require bounded attempts, exponential backoff with jitter, deadlines, and a non-retryable error taxonomy. A durable workflow engine can resume orchestration but cannot manufacture idempotency in a target API.

## Break-glass and kill controls

Operators need independently protected controls to:

- disable all mutating actions;
- disable one action type, tenant, connector, model, prompt, or policy version;
- revoke response identities and outstanding approvals;
- drain or quarantine queued proposals;
- stop new commits while allowing evidence reads and reconciliation;
- inspect in-flight and unknown actions;
- return isolated targets through a reviewed process.

Test these controls without the model and during source/provider outages.

## Failure matrix

| Failure | Prevent | Detect | Recover |
|---|---|---|---|
| Stale approval | Target/policy version binding and expiry | Commit-time mismatch | Re-plan and reapprove |
| Duplicate case write | Semantic key and case version | Duplicate-key metric | Return original result |
| Duplicate isolation | Actuator idempotency plus desired-state read | Effect-ledger reconciliation | Adopt existing action or escalate |
| Wrong tenant | Identity-derived tenant and canonical target | Tenant mismatch hard alert | Deny, revoke, investigate |
| Overbroad target | Typed target, scope limits, blast-radius policy | Preflight count and owner check | Deny before submit |
| Partial mass action | Per-target ledger and stop threshold | Success/failure distribution | Halt, reconcile, reviewed rollback |
| Rollback fails | Preflight and owner-tested procedure | Expiry/rollback SLO alert | Incident escalation |
| Model asks to weaken controls | No policy mutation tool | Denied-attempt telemetry | Investigate injection or compromise |

## Acceptance checklist

- [ ] The read-only and response planes use different identities, networks, and services.
- [ ] Every mutating action has a typed schema, canonical target, preconditions, postconditions, and risk class.
- [ ] Approvals bind exact arguments, case/evidence version, policy, approver, and expiry.
- [ ] Case proposals and confirmed fields are structurally distinct.
- [ ] Semantic idempotency keys survive worker retries.
- [ ] Unknown outcomes reconcile before retry.
- [ ] Reversible containment has a TTL, owner, verification, and rollback.
- [ ] Destructive remediation and offensive actions are unavailable to the model.
- [ ] A kill switch and credential-revocation drill has succeeded.

## Related guides

- [Operating model and reference architecture](operating-model-and-reference-architecture.md)
- [Untrusted content, forensics, and data governance](untrusted-content-forensics-and-data-governance.md)
- [Idempotency and side effects](../../reliability/idempotency-and-side-effects.md)
- [Permissions, sandboxing, and secrets](../../security/permissions-sandboxing-and-secrets.md)

## Selected sources

- [NIST SP 800-207, Zero Trust Architecture](https://csrc.nist.gov/pubs/sp/800/207/final)
- [NIST SP 800-207A, cloud-native application identities and policy](https://csrc.nist.gov/pubs/sp/800/207/a/final)
- [CISA incident and vulnerability response playbooks](https://www.cisa.gov/sites/default/files/2024-08/Federal_Government_Cybersecurity_Incident_and_Vulnerability_Response_Playbooks_508C.pdf)
- [OASIS CACAO 2.0](https://docs.oasis-open.org/cacao/securityplaybooks/v2.0/security-playbooks-v2.0.html)
- [OASIS OpenC2 Architecture 1.0](https://docs.oasis-open.org/openc2/oc2arch/v1.0/oc2arch-v1.0.html)
- [OASIS OpenC2 Language 1.0](https://docs.oasis-open.org/openc2/oc2ls/v1.0/oc2ls-v1.0.html)
- [Google Cloud Eventarc guidance on duplicate events and idempotency](https://cloud.google.com/eventarc/docs/retry-events)
- [Temporal retry policies](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/retry-policies.mdx)
- [Anthropic, How we contain Claude across products](https://www.anthropic.com/engineering/how-we-contain-claude)

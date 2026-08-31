# Infrastructure Operations Agent Blueprint — Research Packet

> **Status:** Completed primary-source research packet  
> **Research date:** 2026-08-31  
> **Scope:** Production infrastructure/VPS/cloud/Kubernetes operations agents, including identity, inventory, tools, plans, approvals, effects, recovery, security, evaluation, and deployment  
> **Blueprint:** [Infrastructure Operations Agent Blueprint](../../agents/infrastructure-operations-agent/README.md)

## Research objective

Determine the smallest defensible architecture for an agent that can progress from advisory infrastructure diagnosis to supervised changes and, for a narrow subset of operations, constrained autonomous remediation.

The research focused on where product language is often stronger than the underlying guarantee: dry runs, approvals, maintenance windows, session logging, short-lived credentials, rollbacks, multi-tenancy, durable execution, and provider audit history.

## Method

1. Started with official cloud, Kubernetes, OpenSSH, PowerShell, IaC/GitOps, workflow, framework, security-standard, and SRE documentation.
2. Cross-checked overlapping controls across AWS, Azure, Google Cloud, Kubernetes, and unmanaged hosts.
3. Read current limitations and lifecycle warnings rather than using product overviews alone.
4. Compared agent-framework mechanics with the application controls required for authorization and side-effect safety.
5. Preferred stable specifications and official repositories; used security/engineering guidance for production interpretation.
6. Recorded volatile behavior and explicit refresh triggers instead of pretending provider-neutral abstractions erase version differences.

This packet does not claim that every named service is available in every region, account type, operating system, or Kubernetes distribution. Deployment qualification must pin and test the versions actually used.

## Research questions

- What authority, if any, should the model hold?
- How should resource identity and inventory freshness be represented?
- How should read, write, session, privilege, and destructive tools differ?
- What makes a plan and approval replay-safe and scope-safe?
- Where should SSH, WinRM, cloud APIs, Kubernetes, IaC, and GitOps sit in the boundary order?
- How should timeouts, retries, partial failure, cancellation, and uncertain effects behave?
- Which autonomy modes are defensible?
- What secrets and identity mechanisms minimize standing privilege?
- What constitutes hard multi-tenant isolation?
- How should drift, maintenance windows, and rollback be described accurately?
- Which controls can an agent framework provide, and which must remain application owned?
- What evidence, fault injection, and acceptance tests are required before writes?
- How should CMDB/service-catalog, ticket/change, IaC/GitOps, and observability integrations preserve authority and provenance?
- Which context and memory classes are useful, and how can compaction remain auditable?
- How should model, tool, policy, workflow, runbook, and telemetry upgrades be mined from failures and released independently?

## Version and volatility baseline

| Surface | Baseline checked on 2026-08-31 | Blueprint treatment | Refresh trigger |
|---|---|---|---|
| Kubernetes | Current documentation identifies the v1.37 reference line; dry-run, resourceVersion, server-side apply, RBAC, audit, APF, PDB, and tenancy behavior reviewed | Pin each supported fleet minor/distribution; use feature discovery and contract tests | Kubernetes minor/distribution or feature-gate change |
| OpenTelemetry | Main semantic conventions 1.44.0; GenAI conventions moved to a separate repository and agent/tool attributes remain developmental | Emit an internal allowlisted schema and record both convention versions | Semconv repository/version or GenAI stability change |
| MCP | 2026-07-28 specification and current authorization/Tasks extension reviewed; 2025-11-25 retained as a migration baseline | Pin core and extension versions; treat MCP as a reasoning-boundary protocol, not the trusted executor/effect ledger | Core, authorization, extension, or task-lifecycle revision |
| Terraform | Current CLI plan/state/locking documentation | Pin CLI, providers, lockfile, backend, and plan artifact in each release | CLI/provider/backend change |
| Ansible | Current “latest” playbook check/diff documentation | Pin ansible-core, collections, execution environment; test module support | Core/collection/module change |
| Flux | Current Kustomization reconciliation documentation | Pin controller version and supported Kubernetes versions | Controller/API change |
| Argo CD | Stable sync-window/wave docs and release 3.4 sync options reviewed | Pin controller version and qualify prune/hook/window behavior | Argo CD release or policy change |
| PowerShell/WinRM | PowerShell 7.6 documentation line reviewed | Pin OS, PowerShell/Windows Management Framework, auth, and JEA config | OS/PowerShell/auth change |
| OpenSSH | Current OpenBSD manuals reviewed | Pin server/client builds and certificate/restriction support | OpenSSH or OS policy change |
| AWS Systems Manager | Current docs; Change Manager new-customer cutoff 2025-11-07 noted | Optional adapter only; do not make Change Manager a universal dependency | Product availability/region/account change |
| Azure | Current Resource Graph, managed identity, Run Command, PIM, JEA, and emergency-access docs | Test subscription/region/agent/role behavior | API/agent/IAM feature change |
| Google Cloud | Current Asset Inventory, IAM/PAM, Audit Logs, OS Login, and VM Manager docs | Test organization/project/region behavior and audit config | API/IAM/retention change |
| NIST AI RMF | AI RMF 1.0 remains the published base; NIST revision activity is ongoing in 2026 | Use as governance frame, not an execution specification | New final RMF/profile |
| OWASP agentic guidance | 2025 excessive-agency and current agentic threat material reviewed | Use as threat catalog; keep deterministic controls | New stable OWASP agentic release |
| Agent frameworks | OpenAI Agents SDK, LangGraph, and Pydantic AI current docs reviewed | Convenience layer only; pin exact package/model behavior | Framework or model release |

## Synthesis: production findings

### 1. The model should propose, not authorize or commit

Model tool use is an application-mediated request. Frameworks can parse a tool call or pause for approval, but the application decides what is authorized and executes the effect. OWASP excessive-agency guidance supports minimizing extensions, permissions, and autonomy. The blueprint therefore gives the model no provider credential or direct write route.

### 2. Inventory requires provenance, freshness, and coverage

Cloud asset systems and Kubernetes watches are useful but incomplete in different ways. Resource Explorer completeness depends on setup and permission; Cloud Asset Inventory history is limited; Resource Graph change analysis covers ARM control-plane changes only; Kubernetes watches require resource-version handling and relist after gaps. The design uses canonical provider IDs, source-specific precedence, event ingestion plus full reconciliation, and explicit partial results.

### 3. Approval and authorization are different controls

Agent frameworks provide human-in-the-loop pause/resume. Pydantic AI explicitly notes that approval is not an authorization boundary against an untrusted client. A production approval must bind an authenticated approver to an immutable plan and target digest; current authorization must be re-evaluated before credential issuance and commit.

### 4. Short-lived, audience-bound workload credentials reduce—not eliminate—risk

AWS STS, Azure managed/workload identities, Google service-account impersonation, Kubernetes TokenRequest, OpenSSH user certificates, and Vault leases support temporary access. Revocation may lag because of provider/session caches; scopes also vary. The broker therefore issues per-operation leases while adapters enforce additional call/target budgets.

### 5. “Dry run” is a collection of provider-specific simulations

Kubernetes server dry-run executes much of admission/defaulting without persistence, but cannot guarantee downstream or concurrent behavior. Terraform speculative plans can stale; saved plans can contain sensitive cleartext values. Ansible check/diff coverage depends on module support and diff can leak secrets. The approval UI must disclose these limitations and protect the plan artifact.

### 6. Prefer desired-state owners, but do not surrender safety to them

Kubernetes controllers, Terraform, Flux, Argo CD, and configuration management provide convergence and history. Direct changes can fight those systems. Yet GitOps prune/force, hooks, waves, and automatic correction still have risk. The agent routes changes through the declared owner and independently verifies live service health.

### 7. Remote shells are a last-resort, high-risk adapter

Session Manager can avoid inbound ports and static SSH keys, but tunneled SSH and port-forwarding content is not logged. OpenSSH forced commands must be combined with forwarding restrictions. WinRM/JEA can narrow PowerShell capability, but CredSSP caches credentials on the remote system and JEA does not defend against existing administrators. Typed provider/host intents are safer than `ssh(host, command)`.

### 8. Timeouts create uncertainty, not failure

Distributed-system guidance on retries/idempotency and provider asynchronous operations shows that a lost response may follow a committed effect. Every write needs an operation ID, durable pre-dispatch record, documented idempotency/reconciliation strategy, provider IDs, and an uncertain state that blocks blind retry.

### 9. Maintenance windows constrain scheduling, not physics

Google VM Manager documents that operations already underway, including downloads or reboots, may finish outside a patch window. AWS maintenance/task behavior also depends on task cutoffs and concurrency. The blueprint distinguishes latest safe start, no-new-work time, in-flight tracking, and completion grace.

### 10. Blast radius is multidimensional

Target count alone does not protect a service. Safe rollout guidance and provider concurrency controls support limits by concurrency, fault domain, error count, disruption, SLO, provider calls, spend, and time. These are consumed and enforced by the executor, not left to model judgment.

### 11. Rollback is not a universal inverse

Some infrastructure operations can reapply a previous desired state; others require restore, compensation, replacement, or roll-forward. Patches, identity changes, networking, and data operations can have irreversible consequences. Plans must name a tested mechanism rather than promise “automatic rollback.”

### 12. Provider audit services are essential but not a durable unified ledger

CloudTrail Event History is currently limited to 90 days of management events for one account/region; Google Cloud Asset history is 35 days; Azure Resource Graph change analysis is 14 days and control-plane only. Kubernetes auditing is policy-dependent. The application exports provider events and maintains its own append-only effect lineage.

### 13. Hard multi-tenancy requires hard trust boundaries

Kubernetes documentation describes namespaces as a tenancy mechanism but recommends stronger isolation, including separate clusters, where required. Provider accounts/projects/subscriptions, execution cells, queues, keys, and artifact stores are stronger boundaries than labels or database columns. Every workflow signal and lookup still needs tenant authorization.

### 14. Break-glass must survive the system it recovers

Microsoft and AWS emergency-access guidance favors separately protected identities, strong authentication, monitoring, testing, and limited use. The agent must be unable to retrieve or invoke these credentials. It can provide a human runbook and later reconcile evidence.

### 15. Framework durability does not define effect semantics

LangGraph interrupts restart a node, making pre-interrupt side effects require idempotency. Durable workflow engines provide histories, timers, signals, and retries but still require deterministic workflows, activity boundaries, versioning, and application-specific reconciliation. Framework memory is not an effect ledger.

### 16. Observability schemas and agent protocols remain volatile

OpenTelemetry moved its GenAI conventions out of the main semantic-conventions repository while agent/tool attributes remain developmental. MCP 2026-07-28 made the protocol core stateless, moved Tasks into an extension, added explicit cross-request state handles, and hardened authorization compared with 2025-11-25. The blueprint pins core/extension/schema versions, allowlists telemetry fields, and places protocol servers behind application policy rather than building effect safety on protocol lifecycle behavior.

### 17. Integrations must preserve field authority and identity

Cloud asset APIs, CMDBs, service catalogs, IaC state, GitOps controllers, tickets, and observability answer different questions. A successful connector call does not make every field complete or authoritative. The design uses field-level ownership, immutable external IDs/versions, explicit inbound/outbound direction, and conflict records. Tickets coordinate work but do not authorize it; observability reports evidence but does not grant infrastructure access.

### 18. Context is a compiled projection, and compaction needs a receipt

Framework sessions and transcripts are convenient but may persist tool inputs, approvals, model responses, and application context. Production model input should be rebuilt by phase from tenant-authorized durable stores. A context/compaction receipt binds compiler/template versions, evidence and event ranges, redactions, omissions, preserved unknown effects and recovery obligations, budgets, and the input digest. This makes resumptions and model migrations auditable without treating the summary as authoritative state.

### 19. Fleet scale requires cells, recovery reserve, and measurable admission

Queue depth and worker count alone are unsafe capacity signals. Provider quota, target disruption budgets, audit/verification throughput, tenant fairness, and reconciliation capacity all bound safe concurrency. Cells bind provider authorities, tenants/environments, identities, keys, routes, operation classes, and failover destinations. Capacity failover cannot weaken isolation, and reconciliation/cancellation capacity remains reserved during overload.

### 20. Production learning is a governed release path

Incidents and operator corrections are candidates, not automatic memory. Human triage assigns the failure to retrieval/model, contract/adapter, policy/approval, workflow, runbook, or verification. De-identified cases enter fixed and held-out suites; behavior bundles move through replay, shadow, supervised canary, and cell-by-cell rollout. Model, context, tool, policy, workflow, runbook, and telemetry changes have separate compatibility and rollback units plus a known-good complete bundle.

## Contradictions, caveats, and adopted resolutions

| Common claim or apparent contradiction | Evidence/caveat | Resolution adopted |
|---|---|---|
| “A human approved it, so execution is authorized.” | Framework approval can be replayed or originate from an untrusted client; user roles may change | Bind approval to exact plan; re-authorize immediately before issuance/commit |
| “Dry-run proves the change is safe.” | Simulations omit or approximate asynchronous, external, concurrent, or unsupported behavior | Label guarantees per mechanism and verify postconditions after commit |
| “A saved Terraform plan is safe to show/store.” | Official docs warn it can contain full configuration and sensitive values in cleartext | Treat as a restricted encrypted artifact; show a linked redacted projection |
| “The maintenance window ended, so patching stopped.” | In-flight download/reboot/provider tasks may continue | Stop new starts and reconcile in-flight work; model window semantics explicitly |
| “Short-lived means instantly revoked.” | Tokens/sessions and Azure managed-identity authorization can be cached | Use short TTL, per-batch issuance, adapter budgets, kill switches, measured revocation |
| “Session Manager gives full session auditing.” | SSH-tunneled and port-forwarding content is not logged by Session Manager | Disable those modes where content audit is required or add another controlled boundary |
| “ForceCommand makes SSH safe.” | OpenSSH documents forwarding controls separately | Combine forced commands with `restrict`/`DisableForwarding`, no PTY, and exact principals |
| “JEA protects the Windows host from administrators.” | Microsoft states existing local/domain admins are outside JEA's protection | Use JEA to narrow non-admin remoting; isolate/administer hosts separately |
| “Namespaces isolate hostile Kubernetes tenants.” | Namespace isolation depends on RBAC, networking, admission, nodes, and control plane | Layer controls; use separate clusters/control planes for strong tenancy |
| “A PDB prevents all workload disruption.” | PDB governs voluntary eviction, not direct delete or controller rollout semantics | Use Eviction API where applicable plus controller/service-level budgets |
| “GitOps automatically fixes unsafe drift.” | Auto-sync/prune/force can overwrite emergency repair or delete resources | Declare ownership, classify drift, gate destructive reconciliation, verify service state |
| “Provider audit history is the operations ledger.” | Scope/retention/configuration differ and can be incomplete | Export to independent storage and maintain an internal effect ledger |
| “Provider-native command systems remove blast-radius concerns.” | Commands already running may continue after error threshold/cancellation | Enforce application and provider budgets; track each target |
| “Cloud credential downscoping is generic.” | Google Credential Access Boundaries currently apply to Cloud Storage | Use service-specific IAM/conditions/deny; do not generalize the feature |
| “A workflow checkpoint makes a write exactly once.” | Worker/process/network failures can occur around external commit | Use at-least-once orchestration plus idempotency, fencing, and reconciliation |
| “Change Manager is an available universal AWS approval service.” | AWS says it is unavailable to new customers starting 2025-11-07 | Keep approval/change abstraction application owned; integrate only where available |
| “MCP authorization and task behavior is settled.” | 2026-07-28 changed transport/state behavior and moved Tasks from the 2025-11-25 core into an extension; authorization continues to evolve | Pin core/extensions; explicit state handles; no token transit; keep provider credentials in the broker |
| “OpenTelemetry GenAI fields are stable and safe to emit.” | GenAI conventions moved repositories; agent/tool attributes remain developmental and may include sensitive args/results | Internal allowlist, both version fields, redaction, and migration tests |
| “Rollback can always restore the prior state.” | Some effects are irreversible, externally observed, or require data restore | Name retry/resume/reconcile/compensate/restore/roll-forward/rollback accurately |
| “The agent should use break-glass when automation is blocked.” | Break-glass is meant to bypass failed normal dependencies and needs human custody | Agent cannot access it; humans follow independent, alarmed procedure |
| “A ticket approval authorizes the effect.” | Ticket identities/comments/webhooks may not bind the exact plan, current role, target digest, or expiry | Treat tickets as coordination unless the connector imports a verifiable plan-bound decision; always re-authorize at commit |
| “A CMDB record is the inventory truth.” | CMDB, provider, desired-state, and runtime sources own different fields and update at different speeds | Declare field owners, freshness and conflicts; never silently merge or overwrite |
| “More workers solve fleet backlog.” | Provider quota, disruption, audit, verification, and recovery capacity are independent bounds | Admission envelope, cell partitioning, fairness, and reserved reconciliation capacity |

## Architecture decisions supported by the research

| Decision | Evidence basis |
|---|---|
| Low-authority model, deterministic commit | Tool-use/application boundary docs, OWASP excessive agency, zero-trust principles |
| Hybrid durable control plane | Workflow durability plus framework pause limitations |
| Separate read/write registries and identities | Least-privilege/IAM/RBAC guidance across providers |
| Canonical target manifests and revalidation | Kubernetes resource versions, IaC plan staleness, provider IDs |
| Operation-scoped credential broker | STS/impersonation/TokenRequest/cert/lease capabilities |
| Typed intent adapters before shell | Provider managed command APIs and remote-session limitations |
| Reconcile uncertain effects before retry | Idempotency/retry and asynchronous operation behavior |
| Multi-dimensional rollout budgets | SRE canary guidance, provider concurrency/error controls, PDB limitations |
| Independent verification | Controller/provider acknowledgement does not imply service health |
| Separate effect ledger and provider audit export | Retention/scope differences across providers |
| Human-only break-glass | Azure/AWS emergency access guidance |
| Per-operation autonomy classification | OWASP agency risk and reversibility/verification constraints |
| Field-authoritative integration contracts | Provider inventory limits, IaC/GitOps ownership, audit and ticket identity separation |
| Compiled context with compaction receipts | Framework serialization behavior plus durable-state/effect separation |
| Cell-bound admission and recovery reserve | Provider quota, APF, rollout disruption, audit, and reconciliation constraints |
| Governed failure-mining release loop | NIST incident/recovery guidance, SRE postmortems, and behavioral release evidence |

## Primary source register

The sources below are the strongest materials used. Dates and version lines should be rechecked at implementation time.

### Kubernetes

1. [Controllers and desired state](https://kubernetes.io/docs/concepts/architecture/controller/) — reconciliation model.
2. [Kubernetes API concepts](https://kubernetes.io/docs/reference/using-api/api-concepts/) — dry-run, resourceVersion, watch, conflicts.
3. [Server-side apply](https://kubernetes.io/docs/reference/using-api/server-side-apply/) — field ownership and conflicts.
4. [RBAC good practices](https://kubernetes.io/docs/concepts/security/rbac-good-practices/) — namespace roles, wildcards, privilege escalation.
5. [Multi-tenancy](https://kubernetes.io/docs/concepts/security/multi-tenancy/) — namespace and stronger isolation choices.
6. [User impersonation](https://kubernetes.io/docs/reference/access-authn-authz/user-impersonation/) — impersonation authorization.
7. [Service accounts](https://kubernetes.io/docs/concepts/security/service-accounts/) — TokenRequest and static-token cautions.
8. [Secrets good practices](https://kubernetes.io/docs/concepts/security/secrets-good-practices/) — encryption and permission implications.
9. [Security overview](https://kubernetes.io/docs/concepts/security/) — cluster/workload security boundaries.
10. [API Priority and Fairness](https://kubernetes.io/docs/concepts/cluster-administration/flow-control/) — server-side request fairness.
11. [Disruptions and PodDisruptionBudgets](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) — Eviction API scope and limitations.
12. [Safely drain a node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/) — node maintenance behavior.
13. [Kubernetes auditing](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/) — policies, levels, volume, sensitivity.

### AWS

14. [Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html) — session identity, network, and logging.
15. [STS AssumeRole API](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) — session policy, duration, tags, MFA.
16. [IAM session tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_session-tags.html) — attribute propagation.
17. [Organizations service control policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html) — maximum permissions, not grants.
18. [Run Command rate controls](https://docs.aws.amazon.com/systems-manager/latest/userguide/send-commands-multiple.html) — concurrency and error semantics.
19. [Systems Manager Change Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/change-manager.html) — templates, approvals, calendars, runbooks.
20. [Change template availability warning](https://docs.aws.amazon.com/systems-manager/latest/userguide/change-templates.html) — new-customer cutoff.
21. [Systems Manager maintenance windows](https://docs.aws.amazon.com/systems-manager/latest/userguide/maintenance-windows.html) — scheduling/tasks.
22. [AWS Config: How it works](https://docs.aws.amazon.com/config/latest/developerguide/how-does-config-work.html) — configuration items/history.
23. [Resource Explorer](https://docs.aws.amazon.com/resource-explorer/latest/userguide/welcome.html) — discovery and completeness.
24. [CloudTrail concepts](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-concepts.html) — trails/events.
25. [CloudTrail Event History](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events.html) — default history scope/retention.
26. [EC2 API idempotency](https://docs.aws.amazon.com/ec2/latest/devguide/ec2-api-idempotency.html) — client-token behavior.
27. [Emergency IAM access](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started-emergency-iam-user.html) — recovery identity guidance.

### Azure and Windows

28. [Managed identity best practices](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/managed-identity-best-practice-recommendations) — lifecycle, role scope, caching.
29. [Run Command overview](https://learn.microsoft.com/en-us/azure/virtual-machines/run-command-overview) — execution channel and privilege.
30. [Limit access to managed Run Command](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/run-command-managed) — RBAC boundary.
31. [Azure Update Manager dynamic scoping](https://learn.microsoft.com/en-us/azure/update-manager/manage-dynamic-scoping) — target scope.
32. [Just-in-time VM access](https://learn.microsoft.com/en-us/azure/defender-for-cloud/enable-just-in-time-access) — management network exposure.
33. [Microsoft Entra PIM](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-configure) — time-bound elevation and approval.
34. [Azure Policy remediation](https://learn.microsoft.com/en-us/azure/governance/policy/how-to/remediate-resources) — managed identity and remediation tasks.
35. [Azure Resource Graph overview](https://learn.microsoft.com/en-us/azure/governance/resource-graph/overview) — cross-subscription inventory.
36. [Resource Graph change analysis](https://learn.microsoft.com/en-us/azure/governance/resource-graph/changes/resource-graph-changes) — scope and 14-day history.
37. [Azure Arc-enabled servers](https://learn.microsoft.com/en-us/azure/azure-arc/servers/) — hybrid host management.
38. [WinRM security](https://learn.microsoft.com/en-us/powershell/scripting/security/remoting/winrm-security?view=powershell-7.6) — authentication and message encryption.
39. [PowerShell remoting second hop](https://learn.microsoft.com/en-us/powershell/scripting/security/remoting/ps-remoting-second-hop?view=powershell-7.6) — delegation options and CredSSP risk.
40. [JEA overview](https://learn.microsoft.com/en-us/powershell/scripting/security/remoting/jea/overview?view=powershell-7.6) — constrained endpoints.
41. [JEA security considerations](https://learn.microsoft.com/en-us/powershell/scripting/security/remoting/jea/security-considerations?view=powershell-7.6) — limitations and hardening.
42. [Microsoft Entra emergency access](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/security-emergency-access) — independent accounts, monitoring, testing.

### Google Cloud

43. [Service-account impersonation](https://cloud.google.com/iam/docs/service-account-impersonation) — short-lived identity and audit chain.
44. [Short-lived delegated credentials](https://cloud.google.com/iam/docs/create-short-lived-credentials-delegated) — delegation.
45. [Credential Access Boundaries](https://cloud.google.com/iam/docs/downscoping-short-lived-credentials) — Cloud Storage-specific downscoping.
46. [IAM Conditions](https://cloud.google.com/iam/docs/conditions-overview) — contextual access.
47. [IAM deny policies](https://cloud.google.com/iam/docs/deny-overview) — explicit deny.
48. [Audit-log Data Access configuration](https://cloud.google.com/logging/docs/audit/configure-data-access) — non-default audit coverage.
49. [Cloud Asset Inventory overview](https://cloud.google.com/asset-inventory/docs/asset-inventory-overview) — asset history and inventory.
50. [Cloud Asset Inventory feeds](https://cloud.google.com/asset-inventory/docs/reference/rest/v1/feeds) — event-driven changes.
51. [OS Login SSH certificates](https://cloud.google.com/compute/docs/oslogin/certificates) — short-lived SSH identity.
52. [VM Manager patch jobs](https://cloud.google.com/compute/vm-manager/docs/patch/create-patch-job) — rollout, disruption, and window behavior.
53. [Privileged Access Manager overview](https://cloud.google.com/iam/docs/pam-overview) — temporary elevation.
54. [Security foundations operational best practices](https://cloud.google.com/architecture/blueprints/security-foundations/operation-best-practices) — emergency access and operations.

### Desired state, IaC, and GitOps

55. [Terraform plan](https://developer.hashicorp.com/terraform/cli/commands/plan) — speculative/saved plans, staleness, sensitive artifact warning.
56. [Terraform state](https://developer.hashicorp.com/terraform/language/state) — state purpose and sensitivity.
57. [Terraform state locking](https://developer.hashicorp.com/terraform/language/state/locking) — concurrent apply protection.
58. [Ansible check and diff modes](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_checkmode.html) — module support and secret exposure.
59. [Flux Kustomizations](https://fluxcd.io/flux/components/kustomize/kustomizations/) — server-side dry-run, drift correction, prune/force.
60. [Argo CD sync windows](https://argo-cd.readthedocs.io/en/stable/user-guide/sync_windows/) — allow/deny scheduling.
61. [Argo CD sync options](https://argo-cd.readthedocs.io/en/release-3.4/user-guide/sync-options/) — prune confirmation and apply options.
62. [Argo CD sync phases and waves](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-waves/) — ordering and hooks.

### Remote access, workload identity, and secrets

63. [OpenBSD sshd_config](https://man.openbsd.org/sshd_config) — forced commands, principals, forwarding.
64. [OpenBSD sshd](https://man.openbsd.org/sshd) — authorized_keys restrictions and `restrict`.
65. [OpenBSD ssh-keygen](https://man.openbsd.org/ssh-keygen) — certificate principals, validity, KRL.
66. [Vault leases](https://developer.hashicorp.com/vault/docs/concepts/lease) — TTL, renewal, revocation.
67. [Vault audit devices](https://developer.hashicorp.com/vault/docs/audit) — HMAC, multiple devices, availability.
68. [SPIFFE concepts](https://spiffe.io/docs/latest/spiffe/concepts/) — SVIDs and trust domains.
69. [SPIFFE Workload API](https://spiffe.io/docs/latest/spiffe-specs/spiffe_workload_api/) — workload identity delivery.

### Security, reliability, and operations

70. [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework) — governance framework and revision status.
71. [NIST AI 600-1 Generative AI Profile](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf) — generative-AI risk actions.
72. [NIST SP 800-207](https://csrc.nist.gov/pubs/sp/800/207/final) — zero-trust principles.
73. [NIST SP 800-207A](https://csrc.nist.gov/pubs/sp/800/207/a/final) — cloud-native access control.
74. [NSA/CISA Kubernetes Hardening Guidance](https://www.nsa.gov/Press-Room/Digital-Media-Center/Document-Gallery/igphoto/2003066362/) — cluster hardening.
75. [OWASP LLM06:2025 Excessive Agency](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/) — agency/tool/permission risk.
76. [OWASP agentic threats and mitigations](https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/) — agent-specific threat categories.
77. [Google SRE: Automation at Google](https://sre.google/sre-book/automation-at-google/) — automation evolution and failure awareness.
78. [Google SRE: Emergency response](https://sre.google/sre-book/emergency-response/) — incident procedure and preparedness.
79. [Google SRE Workbook: Canarying releases](https://sre.google/workbook/canarying-releases/) — canary design.
80. [Google SRE Workbook: Postmortem culture](https://sre.google/workbook/postmortem-culture/) — learning and evidence.
81. [AWS Builders' Library: Safe hands-off deployments](https://aws.amazon.com/builders-library/automating-safe-hands-off-deployments/) — deployment safety.
82. [AWS Builders' Library: Timeouts, retries, and backoff](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/) — retry amplification and jitter.
83. [RFC 9110 HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) — method/idempotency semantics.

### Agent runtimes, protocols, and observability

84. [OpenAI Agents SDK human-in-the-loop](https://openai.github.io/openai-agents-python/human_in_the_loop/) — approval pauses and serialized state.
85. [OpenAI Agents SDK guardrails](https://openai.github.io/openai-agents-python/guardrails/) — input/output/tool guardrails.
86. [OpenAI Agents SDK tracing](https://openai.github.io/openai-agents-python/tracing/) — agent traces and sensitivity.
87. [LangGraph interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts) — resume behavior and idempotency caveat.
88. [LangGraph persistence](https://docs.langchain.com/oss/python/langgraph/persistence) — checkpoints and threads.
89. [Pydantic AI deferred tools](https://pydantic.dev/docs/ai/tools-toolsets/deferred-tools/) — external approval and authorization warning.
90. [Pydantic AI durable execution](https://pydantic.dev/docs/ai/capabilities/durable_execution/overview/) — durable runtime integrations.
91. [Temporal event-history documentation source](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/event-history/python.mdx) — durable event histories and limits.
92. [Anthropic tool-use flow](https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works) — application-mediated tool execution.
93. [Google Gemini function calling](https://ai.google.dev/gemini-api/docs/function-calling) — function declaration/call flow.
94. [MCP 2026-07-28 core specification](https://modelcontextprotocol.io/specification/2026-07-28/basic) — current stateless core, explicit state handles, schema limits, and trace context.
95. [MCP current authorization](https://modelcontextprotocol.io/specification/draft/basic/authorization) — audience validation, issuer checks, resource indicators, and no transit of other-resource tokens.
96. [MCP Tasks extension](https://tasks.extensions.modelcontextprotocol.io/specification/draft/tasks) — separately versioned long-running operation lifecycle and security.
97. [MCP 2025-11-25 basic specification](https://modelcontextprotocol.io/specification/2025-11-25/basic) — migration comparison baseline.
98. [OpenTelemetry semantic conventions 1.44.0](https://opentelemetry.io/docs/specs/semconv/) — main schema baseline.
99. [OpenTelemetry GenAI move notice](https://opentelemetry.io/docs/specs/semconv/gen-ai/) — separate repository and migration signal.
100. [OpenTelemetry generative-AI attributes](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/) — development-stage attributes and sensitive payloads.
101. [NIST SP 800-61 Rev. 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final) — current incident-response risk-management guidance.
102. [NIST SP 800-184](https://csrc.nist.gov/pubs/sp/800/184/final) — recovery planning, playbooks, tests, and improvement.

## Sources intentionally treated as secondary

Vendor marketing claims, generic “autonomous DevOps agent” articles, unsourced benchmark posts, and examples that grant a model blanket shell/admin access were not used as architectural authority. Community discussions can reveal failure modes, but the decisions above are anchored in official documentation, standards, or mature SRE guidance.

## Known limitations

- No live AWS, Azure, Google Cloud, Kubernetes, SSH, or WinRM environment was exercised for this documentation task.
- Provider quotas, regional availability, subscription/licensing, and exact IAM actions must be qualified in the target estate.
- The blueprint does not define a universal typed schema for every infrastructure operation; each adapter requires domain-specific design and tests.
- Data-plane systems such as databases, storage deletion, key destruction, and network-control-plane recovery need dedicated blueprints.
- Application delivery, incident command, network intent, database state, cost allocation/commitments, and IAM design remain owned by the DevOps, SRE, Network, Database, FinOps, IAM/security disciplines respectively; this blueprint defines typed handoffs only.
- Model quality thresholds must be derived from the organization's incidents, resource taxonomy, and risk tolerance.
- Legal requirements for session recording, data retention, and cross-region telemetry vary by jurisdiction.
- “Autonomous remediation” remains inappropriate where eligibility or verification depends mainly on model judgment.

## Refresh checklist

- [ ] Recheck all lifecycle/deprecation warnings.
- [ ] Pin provider, SDK, Kubernetes, IaC/GitOps, workflow, framework, model, MCP, and telemetry versions.
- [ ] Revalidate inventory coverage and history/retention.
- [ ] Revalidate audit defaults and data-plane coverage.
- [ ] Revalidate maintenance-window, cancellation, concurrency, and retry semantics.
- [ ] Revalidate credential TTL, audience, scope, cache, and revocation.
- [ ] Revalidate SSH/WinRM/JEA restriction behavior on target OS builds.
- [ ] Re-run fault injection and tenant/injection evaluations.
- [ ] Update contradictions when a provider or standard closes a gap.
- [ ] Record research date and the exact deployment baseline.

# Security, Identity, Tenancy, and Control-Plane Isolation

## Threat model

A network operations agent concentrates sensitive topology, management access, traffic metadata, and configuration authority. Assume attackers can influence operator tickets, device banners and descriptions, DNS records, certificate fields, syslog, telemetry labels, config comments, controller object names, retrieved runbooks, and packet contents. Also assume a device or controller may be compromised and its reported state misleading.

The principal threats are:

- prompt injection causing unsafe evidence selection, data disclosure, or tool requests;
- excessive agency or a confused-deputy path from read access to write authority;
- credential/key leakage through prompts, logs, traces, artifacts, or adapters;
- cross-tenant target selection, cache/index leakage, or credential reuse;
- management-plane reachability from untrusted networks or lateral movement between targets;
- compromised-source evidence poisoning topology, memory, and verification;
- overbroad packet/flow collection and retention;
- unauthorized or stale plan execution, approval forgery, and audit tampering;
- supply-chain compromise of model, tool schema, adapter, parser, lab image, or policy bundle;
- recovery failure when production changes also damage OOB, AAA, DNS, time, or audit dependencies.

## Trust-boundary rules

```mermaid
flowchart LR
    U[Users, tickets, documents] -->|untrusted content| C[Normalizer and content labels]
    N[Devices, DNS, certs, logs, packets] -->|untrusted data| C
    C -->|bounded structured evidence| M[Model]
    M -->|typed proposal only| P[Deterministic policy]
    P -->|sealed approved effect| E[Isolated executor]
    K[Credential broker] -->|short-lived scoped identity| E
    E -->|allowlisted management egress| T[Specific target/controller]
    T -->|receipt/state| R[Independent reconciliation and verification]
    M -. no route .-> K
    M -. no route .-> T
```

Enforce these rules technically:

1. Untrusted strings are length-bounded, encoded, and carried in fields marked `data`; they are never concatenated into system/tool instructions.
2. Parsers output strict schemas. Unknown fields and malformed records fail or quarantine; they do not become free-form model instructions.
3. The model chooses only from target- and tenant-scoped typed tools. It cannot create a command, URL, credential request, or target selector outside the contract.
4. Policy authorizes normalized effects independently of the model explanation.
5. The executor consumes a sealed artifact from the approval service, not model output.
6. Credentials and private keys never enter model context. The write worker receives an opaque, short-lived capability for one effect scope.
7. Verification uses separately credentialed read paths and independent observations; a compromised target's self-report is not enough for sensitive work.

## Identity chain

Every action should be attributable through a chain:

```text
authenticated requester
  → immutable task and tenant
  → proposer/model release
  → policy decision and approver identities
  → workflow operation ID
  → executor workload identity
  → broker-issued target identity
  → native target/controller audit identity
```

Human users authenticate with phishing-resistant MFA where appropriate. Workloads use mutually authenticated, short-lived identities. The policy engine evaluates requester, approver, executor, tenant, target class, effect fields, risk, window, and plan digest. Avoid shared `admin` accounts because they destroy attribution and make revocation coarse.

## Credential and key design

The credential broker should:

- issue short-lived SSH certificates, mTLS identities, OAuth/cloud tokens, or protocol-specific capabilities;
- bind tenant, target IDs, permitted operations/paths, plan and operation IDs, not-before/not-after, and executor workload;
- prefer write-only-to-required-resource scope rather than broad device privilege;
- prevent credential export when hardware/key-service signing can be used;
- revoke or let expire immediately after completion;
- log issuance and use metadata without secret material;
- support independent emergency credentials held outside the agent path.

For devices whose authorization model cannot express field-level scope, contain the risk with dedicated roles, command accounting, allowlisted adapters, per-target credentials, management ACLs, executor egress restrictions, short duration, and a lower permitted autonomy tier.

Certificate private keys remain in a KMS/HSM or termination platform. ACME DNS credentials are restricted to exact zones/names where the provider supports it. Never let the model handle challenge tokens beyond non-secret status or write them through a general DNS tool.

## Management-plane isolation

CISA guidance recommends dedicated out-of-band management and strongly restricted administrative access. The network agent should have:

- physically or logically independent OOB reachability for critical devices;
- dedicated management VRFs/VPCs with no route leak to customer or peering domains;
- default-deny management ingress and tightly restricted executor egress;
- no lateral device-to-device administration unless an explicit protocol requires it;
- protected centralized AAA, audit, telemetry, flow, time, and configuration services;
- SNMPv3 or stronger protected modeled telemetry instead of insecure legacy management;
- separate read, probe, stage/write, verification, and break-glass identities;
- control-plane policing and rate limits that include automation traffic.

Do not allow a plan to mutate the management route, AAA, NTP, audit destination, OOB interface, credential broker dependency, or executor ACL that its own verification and rollback require. Such work is N5/dedicated procedure unless a separately proven recovery plane exists.

## Prompt injection controls

Network data is especially hostile because many fields permit arbitrary text. Apply controls at every step:

- strip terminal control sequences and normalize encoding before display or parsing;
- keep raw artifacts immutable, but expose redacted parsed fields to the model;
- wrap content with explicit source, type, sensitivity, and “data-only” labels;
- never follow instructions found in DNS TXT, certificate subject/SAN, banner/MOTD, LLDP description, interface description, syslog, ticket, packet, or configuration comment;
- do not give retrieved runbooks execution authority; bind only approved runbook IDs and versions in policy;
- constrain output to JSON/schema and validate all values server-side;
- run injection canaries and adversarial records in evaluation;
- alert when data contains instruction-like content near a requested high-risk effect.

Input sanitization alone cannot solve prompt injection. The decisive boundary is that the model lacks direct secrets and mutation authority.

## Tenant isolation

Tenant is part of every primary key, target identity, topology edge, workflow, lease, queue message, cache entry, artifact ACL, evidence query, credential grant, audit record, vector/search index, trace, and metric dimension where safe. Enforce tenant filters in the service and storage authorization layer rather than relying on the model to include them.

For high-sensitivity networks, use separate cells, encryption keys, buckets/databases, probe pools, executors, and broker roles. Prevent target identity aliasing across tenants. A target discovered with conflicting tenant ownership enters quarantine; it is not automatically reassigned.

Test isolation with negative cases:

- cross-tenant target ID in an otherwise valid plan;
- artifact or topology edge reference from another tenant;
- cached evidence after tenant switch;
- shared device with tenant-specific VRFs/contexts;
- duplicate display names across tenants;
- trace/search query omitting tenant predicate;
- credential requested for a target outside the task.

Any cross-tenant control failure is a release blocker, not a quality score that can be averaged.

## Packet, flow, and personal-data controls

RFC 9232 recommends minimizing telemetry exposure and avoiding end-user packet-content collection. Default to counters, sampled flow metadata, and active tests. Before packet capture require purpose, legal/policy basis, approved sensor, exact filter, duration, snap length, byte limit, fields permitted for derivation, artifact readers, retention, and deletion.

Redact or tokenize addresses and identifiers when the task permits aggregate analysis. Encrypt transport and storage. Record capture drops and gaps so privacy minimization is not misread as absence. Never send raw payload to an external model provider unless an explicitly reviewed data-processing design allows that exact scope.

## Compromised-source handling

During suspected compromise:

- hand investigation ownership to the security agent/team;
- freeze autonomous changes on affected management and control paths;
- treat device/controller logs, config, routes, time, and health as potentially manipulated;
- corroborate with independent collectors, upstream/downstream peers, immutable audit, OOB reads, flow sensors, and active measurements;
- preserve artifacts and hashes according to security evidence procedure;
- do not write “cleanup” commands that destroy evidence;
- execute containment only from a security-approved, network-validated plan.

The network agent may conclude “these two evidence sources conflict”; it must not conclude “the attacker did X” outside its category authority.

## Break-glass

Break-glass remains human-controlled and independent of model availability. The procedure includes:

1. declared emergency and incident owner;
2. strong independent authentication and two-person control for critical scopes;
3. exact targets, permitted actions, and short expiry;
4. OOB access and live recording/audit;
5. pre-action snapshot where time permits;
6. explicit communication and stop conditions;
7. immediate credential revocation/rotation after use;
8. reconciliation of target state against intended state;
9. mandatory security and operational review.

The agent can prepare evidence and a plan, but a “break-glass tool” must not appear in its callable surface.

## Supply-chain and upgrade controls

Sign and verify adapter packages, parser schemas, policy bundles, lab images, model release metadata, and tool contracts. Produce an inventory/SBOM where applicable. Pin versions in the sealed plan. Run compatibility, replay, lab, prompt-injection, tenant-isolation, and rollback tests before promotion. A device OS or API upgrade invalidates affected capability attestations until conformance passes again.

## Security incident triggers

Immediately stop or restrict the service for:

- unexpected credential use, target, source IP, or management path;
- plan digest mismatch or approval reuse;
- cross-tenant lookup or artifact access;
- unauthorized write path or new executor egress;
- audit gaps around a side effect;
- adapter/parser integrity failure;
- model/tool output attempting to escape its schema repeatedly;
- evidence that the target or controller is compromised;
- loss of independent OOB, AAA, or time integrity during change.

Preserve logs and effect records; fence workers; revoke credentials; reconcile outstanding effects; and coordinate with security and SRE. Do not “clean up” by deleting suspicious artifacts.

## Primary evidence

- [CISA: Enhanced Visibility and Hardening Guidance for Communications Infrastructure](https://www.cisa.gov/resources-tools/resources/enhanced-visibility-and-hardening-guidance-communications-infrastructure)
- [CISA: Countering Chinese State-Sponsored Actors Compromise of Networks Worldwide](https://www.cisa.gov/news-events/cybersecurity-advisories/aa25-239a)
- [CISA BOD 23-02: Mitigating the Risk from Internet-Exposed Management Interfaces](https://www.cisa.gov/news-events/alerts/2023/06/13/cisa-issues-bod-23-02-mitigating-risk-internet-exposed-management-interfaces)
- [RFC 9232: Network Telemetry Framework](https://www.rfc-editor.org/rfc/rfc9232.html)
- [OWASP: Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
- [OWASP: Excessive Agency](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/)
- [NIST AI 600-1: Generative AI Profile](https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence)

See [permissions, sandboxing, and secrets](../../security/permissions-sandboxing-and-secrets.md) for the repository-wide control model.


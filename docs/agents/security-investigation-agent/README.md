# Security Investigation and Triage Agent

> **Status:** Research-backed production blueprint  
> **Last researched:** 2026-08-31  
> **Scope:** Defensive alert triage, evidence gathering, incident investigation, case support, and tightly constrained response. This is not an autonomous offensive-security agent.  
> **Evidence:** [Security investigation agent research packet](../../research/packets/security-investigation-agent-blueprint.md)

A security investigation agent should compress evidence and analyst toil without becoming the authority that declares truth or changes production. The safe default is an evidence-grounded, read-only investigator: it gathers authorized context, records hypotheses and contradictions, cites immutable evidence, recommends a verdict, and hands consequential decisions to accountable people and deterministic controls.

## When not to build an agent

Use deterministic enrichment, correlation, or a reviewed playbook when the input contract, decision, and safe action are already known. A model adds risk without useful judgment when all of these are true:

- source events have stable identifiers and schemas;
- the required enrichment is a fixed set of bounded lookups;
- a deterministic predicate can make the decision with acceptable error;
- the action has an established owner, preconditions, rollback, and verification;
- an analyst does not need cross-source interpretation or competing hypotheses.

Examples include rejecting malformed alerts, deduplicating a redelivery, enriching an IP from an approved feed, routing by an authoritative asset owner, or executing a pre-existing human-approved containment playbook. Keep those paths independent of inference. Add an investigator only where heterogeneous evidence, incomplete coverage, ambiguity, or narrative synthesis creates measured analyst toil that rules cannot safely remove.

The first useful agent should be much smaller than the final architecture: accept one curated alert family, assemble a fixed read-only evidence bundle, produce cited claims plus a benign alternative and explicit gaps, and stop for analyst review. It needs no response credential, vector database, multi-agent topology, long-term model-written memory, or durable workflow engine.

## Core contract

> **The model may propose findings, queries, and response actions. Only deterministic policy, scoped credentials, and an accountable approver may authorize a production effect.**

The agent must never:

- treat an alert, document, email, threat report, malware string, tool result, or prior case note as trusted instruction;
- silently convert an inference into a fact, indicator, detection, block, isolation, reset, deletion, or case closure;
- use credentials or data from another tenant, region, customer, or investigation;
- analyze a live malware sample in the orchestration runtime;
- claim that absence of retrieved evidence proves absence of compromise;
- overwrite raw evidence, its provenance, or its custody history;
- retry a mutating action without an idempotency key and observed-outcome reconciliation.

## Read in this order

1. [Operating model and reference architecture](operating-model-and-reference-architecture.md) — purpose, boundaries, lifecycle, and four deployment architectures.
2. [Evidence intake, context, and case state](evidence-intake-context-and-case-state.md) — alert envelopes, normalization, asset and identity enrichment, hypotheses, timelines, provenance, and chain of custody.
3. [Security integrations and adapter qualification](security-integrations-and-adapter-qualification.md) — SIEM, SOAR, EDR, cloud, IAM, CTI, ticket/case contracts; pagination, freshness, schema drift, and connector release gates.
4. [Investigation reasoning, tools, models, and runtime](investigation-reasoning-tools-and-runtime.md) — bounded investigation loops, typed tools, model routing, runtime selection, stopping, and abstention.
5. [Authority, approvals, and constrained response](authority-approvals-and-constrained-response.md) — read/write tiers, least privilege, approval binding, idempotent effects, and containment/remediation gates.
6. [Untrusted content, forensics, and data governance](untrusted-content-forensics-and-data-governance.md) — prompt injection, malware isolation, evidence integrity, privacy, retention, and tenant boundaries.
7. [Reliability, observability, scaling, and operations](reliability-observability-scaling-and-operations.md) — durable execution, recovery, audit, SLOs, cost controls, deployment, and incident operations.
8. [Evaluation, rollout, and build roadmap](evaluation-rollout-and-build-roadmap.md) — realistic evals, false-positive/false-negative controls, failure injection, release gates, and staged delivery.

## Zero-to-production path

| Milestone | Useful capability | Do not add yet | Evidence required to advance |
|---|---|---|---|
| 0 — Deterministic foundation | Durable intake, raw evidence, native IDs, deduplication, case/custody schema, replay corpus | Model or external query | Exact replay, tenant isolation, source/coverage truth |
| 1 — Bounded MVP | One alert family and fixed evidence bundle; cited claims, benign alternative, gaps, abstention | Live source access, case writes, response | Blind analyst usefulness plus miss, citation, injection, latency, and cost gates |
| 2 — Useful supervised v1 | A few tenant-safe read templates and a deterministic context compiler | General query language or writes | Pagination/coverage, authorization, fallback, analyst correction, and load evidence |
| 3 — Reliable workflow | Durable case tasks and proposed fields with concurrency, idempotency, audit, and reconciliation | Production containment | Crash/replay, duplicate/concurrent write, schema drift, on-call, and SLO drills |
| 4 — Production containment pilot | One exact reversible TTL action through separate policy, approval, and executor | Broad, destructive, or self-selected response | False-action budget, preservation, observed outcome, rollback, kill-switch, owner approval |
| 5 — Resilient scale | Qualified additional sources/tenants/regions; quotas, backpressure, disaster recovery, cost control | Authority expansion based only on volume | Capacity, fairness, regional failure, recovery, deletion/hold, and source-specific quality gates |
| 6 — Governed evolution | Versioned behavior bundles, controlled failure mining, drift detection, shadow/canary, full rollback | Self-editing prompts, tools, policies, memory, detections, or authority | Held-out improvement with safety invariants and attribution intact |

Advance one alert family, source, tenant, and action class at a time. “Production ready” is scoped: strong results for identity alerts do not automatically qualify endpoint, cloud, email, or network investigations.

## Reference architecture

~~~mermaid
flowchart LR
    subgraph Sources["Authorized source systems"]
        SIEM["SIEM / detections"]
        EDR["EDR / cloud / identity"]
        CTI["Threat intelligence"]
        CMDB["Asset and identity context"]
    end

    subgraph Intake["Trusted intake and case plane"]
        ING["Validate, deduplicate, normalize"]
        RAW["Immutable evidence store"]
        CASE["Case, claim, hypothesis, timeline store"]
    end

    subgraph Proposal["Untrusted proposal plane"]
        CTX["Context compiler"]
        AGENT["Investigation agent"]
        RO["Read-only query broker"]
    end

    subgraph Control["Deterministic control plane"]
        PDP["Policy decision point"]
        APPROVE["Human approval"]
        LEDGER["Effect and audit ledger"]
    end

    subgraph Effects["Separated response plane"]
        EXEC["Scoped action executor"]
        TARGET["EDR / IAM / network / case system"]
    end

    Sources --> ING
    ING --> RAW
    ING --> CASE
    RAW --> CTX
    CASE --> CTX
    CTX --> AGENT
    AGENT --> RO
    RO --> Sources
    RO --> RAW
    AGENT --> CASE
    AGENT -. "action proposal" .-> PDP
    PDP --> APPROVE
    APPROVE --> PDP
    PDP --> EXEC
    EXEC --> TARGET
    EXEC --> LEDGER
    LEDGER --> CASE
~~~

The case store is not the evidence store. The case store holds versioned interpretations and workflow state; the evidence store holds immutable source material and integrity metadata. A model may append a proposed claim, but it cannot alter the bytes or provenance that support or contradict that claim.

## Architecture choices

| Architecture | Agent authority | Best fit | Main risk | Adoption gate |
|---|---|---|---|---|
| Advisory triage | Read supplied bundle; draft verdict and next queries | First deployment, regulated environments, low integration maturity | Shallow context and analyst copy/paste | Evidence citations and calibrated abstention |
| Supervised investigation | Run bounded read-only pivots; update a draft case | Mature SOC with reliable APIs and analyst review | Query fan-out, sensitive-data over-collection | Tenant-safe query broker, budgets, full tool audit |
| SOAR-integrated workflow | Create/update cases and prepare signed playbook steps | Teams with established playbooks and separation of duties | Automation can launder model error into workflow truth | Typed case writes, idempotency, policy and approval binding |
| Constrained response | Execute a tiny allowlist of reversible, time-bounded containment actions | Only proven high-confidence scenarios with strong rollback | Business disruption, evidence loss, attacker adaptation | Shadow/canary evidence, dual control, reconciliation, kill switch |

Do not begin with constrained response. Progress through the architectures only when evaluation and operational evidence justify the added authority.

## Category boundary and handoffs

This blueprint owns **malicious-activity investigation**: alert disposition, security evidence custody, adversary-related hypotheses, compromise scoping, containment strategy, and a security case record. It does not acquire the authority of adjacent operational systems.

| Adjacent blueprint | It owns | Security investigation owns | Required handoff |
|---|---|---|---|
| [SRE incident response](../sre-incident-response-agent/README.md) | Service-impact declaration, incident command, restoration sequencing, reliability communications, and recovery objectives | Whether evidence indicates malicious activity, what must be preserved, and which containment choices reduce attacker access | Shared incident ID, affected service/assets, evidence-preservation constraints, containment proposal, service-risk/rollback assessment, and authoritative effect receipts |
| [Vulnerability remediation](../vulnerability-remediation-agent/README.md) | Applicability validation, patch or dependency-change preparation, compensating controls, exception tracking, and remediation proof | Active-compromise triage, attacker activity, forensic scope, and immediate containment strategy | Immutable evidence references, affected subject/version, observed exploitation or persistence, incident constraints, and post-remediation monitoring needs |
| [Infrastructure operations](../infrastructure-operations-agent/README.md) | Compute, host, cluster, storage, and cloud desired-state operations | The security requirement and acceptable containment outcome | Exact resource, case/incident ID, action request, authority/expiry, evidence-preservation rule, verification signal; infrastructure returns operation and observed-state receipts |
| [Network operations](../network-operations-agent/README.md) | Routing, DNS, certificates, network policy, path evidence, and network effects | Malicious indicator/context and security containment objective | Typed query or sealed block proposal with direction, scope, TTL, owner, false-positive risk, and rollback; network control plane decides and records the effect |
| [Identity and access governance](../identity-access-governance-agent/README.md) | Directory, role, group, token, federation, and identity-policy changes | Evidence of account/session misuse and the narrow containment requirement | Canonical identity/session, tenant, exact revocation proposal, responder-access constraints, expiry, and verification; IAM returns authoritative state |

One event may be both a security incident and a reliability incident. Link records; do not merge roles or issue a “super-agent” identity with the union of security, SRE, infrastructure, network, and IAM permissions. Vulnerability remediation starts when the question becomes “which controlled change removes the weakness?” Security investigation remains responsible for “what happened, what evidence proves it, and what attacker capability must be contained?”

## Stable design decisions

| Decision | Production rule |
|---|---|
| Truth | Findings are claims with evidence, confidence, and unresolved alternatives—not model-authored facts. |
| Investigation | Prefer one bounded investigator over a multi-agent committee unless isolation or independent verification has measured value. |
| Access | Every read is scoped by tenant, principal, purpose, resource, time range, and field-level policy. |
| Response | Separate the response executor and credentials from the model and read-only investigation runtime. |
| Forensics | Preserve originals, hash acquisitions, analyze working copies, and append custody events. |
| Recovery | Persist case transitions and tool outcomes; reconcile unknown effects before any retry. |
| Evaluation | Grade environment state, evidence coverage, forbidden actions, calibration, analyst correction, latency, and cost. |
| Operations | Version prompts, models, tools, policies, schemas, detection content, and CTI snapshots independently. |

## Baseline as of 2026-08-31

| Area | Baseline | Important qualification |
|---|---|---|
| Incident response | NIST SP 800-61 Rev. 3 (2025) | Supersedes Rev. 2; organize response across CSF 2.0 functions. |
| Digital evidence | NIST SP 800-86; NISTIR 8387; ISO/IEC 27037:2012 | SP 800-86 is old but still useful; ISO/IEC 27037 is under revision. Legal requirements remain jurisdiction-specific. |
| SOC services | FIRST CSIRT Services Framework 2.1 | A service taxonomy, not a maturity model or implementation. |
| Threat knowledge | MITRE ATT&CK 19.2 | ATT&CK data sources were deprecated in v18; use detection strategies, analytics, and data components. |
| Security events | OCSF 1.8.0 | Pin schema and extension versions; normalization is lossy unless raw events are retained. |
| Threat intelligence | STIX 2.1 / TAXII 2.1; FIRST TLP 2.0 | STIX structure does not prove intelligence quality; TLP governs sharing, not truth. |
| Playbooks and commands | CACAO 2.0; OpenC2 Language 1.0 CS02 | Interchange formats do not supply local authorization, safety, or rollback semantics. |
| Detection content | Sigma specification 2.1.0 | Backend translation and local log semantics still require validation. |
| Agent telemetry | OpenTelemetry GenAI conventions | Development status; keep an application-owned, versioned audit schema. |

## Production acceptance snapshot

The system is not ready for analyst-facing use unless it can:

- reproduce every finding from cited evidence or label it as an unsupported hypothesis;
- preserve competing explanations and show what evidence would discriminate them;
- refuse or abstain when evidence, access, provenance, or time coverage is insufficient;
- survive duplicate alerts, worker crashes, delayed approvals, stale policy, provider timeouts, and partial tool failure;
- prevent cross-tenant reads and writes even when the model explicitly requests them;
- quarantine active content from the orchestrator and keep long-lived credentials outside analysis sandboxes;
- quantify missed-incident and unnecessary-escalation costs by alert family, tenant, asset class, and severity;
- reconstruct who proposed, approved, attempted, observed, superseded, and closed every material action.

## Known limitations

- No generic blueprint can define an organization's incident declaration thresholds, legal hold, labor monitoring, regulatory reporting, or evidence admissibility rules.
- Model and benchmark results are time-sensitive and do not transfer automatically across telemetry, languages, tenants, or adversaries.
- Current public defensive-agent benchmarks are small compared with the diversity of real investigations; some rely on synthetic alerts or model judges.
- Prompt injection remains an open security problem. This blueprint limits blast radius but does not claim perfect detection.
- Automatic containment can create operational harm even when the technical diagnosis is correct. Business ownership and rollback remain human responsibilities.

## Related repository guides

- [Agent threat model](../../security/agent-threat-model.md)
- [Prompt injection and untrusted data](../../security/prompt-injection-and-untrusted-data.md)
- [Permissions, sandboxing, and secrets](../../security/permissions-sandboxing-and-secrets.md)
- [Tool results, artifacts, and provenance](../../tools/tool-results-artifacts-and-provenance.md)
- [Idempotency and side effects](../../reliability/idempotency-and-side-effects.md)
- [Durable execution](../../runtime/durable-execution.md)
- [Observability and tracing](../../evaluation/observability-and-tracing.md)
- [Trajectory and reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md)

## Canonical sources

- [NIST SP 800-61 Rev. 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final)
- [FIRST CSIRT Services Framework 2.1](https://www.first.org/standards/frameworks/csirts/csirt_services_framework_v2.1)
- [MITRE ATT&CK version history](https://attack.mitre.org/resources/versions/)
- [OCSF schema releases](https://github.com/ocsf/ocsf-schema/releases)
- [OASIS STIX 2.1 and TAXII 2.1](https://www.oasis-open.org/2021/06/23/stix-v2-1-and-taxii-v2-1-oasis-standards-are-published/)
- [OASIS CACAO 2.0](https://www.oasis-open.org/standard/cacao-security-playbooks-v2-0/)
- [NISTIR 8387 digital evidence preservation](https://www.nist.gov/publications/digital-evidence-preservation-considerations-evidence-handlers)

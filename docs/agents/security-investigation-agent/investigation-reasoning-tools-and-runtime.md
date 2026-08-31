# Investigation Reasoning, Tools, Models, and Runtime

> **Status:** Research-backed production blueprint  
> **Last researched:** 2026-08-31  
> **Scope:** Bounded investigation control flow, tool contracts, model selection, language/runtime choices, and stopping behavior.  
> **Section index:** [Security investigation and triage agent](README.md)

Use the model where ambiguity and synthesis matter. Use deterministic software for identity, authorization, query compilation, arithmetic, deduplication, policy, workflow transitions, and effects. The smallest reliable design is usually one investigator operating over typed read-only tools—not a society of agents debating the alert.

## Reasoning contract

The system needs an auditable rationale, not private chain-of-thought. Persist:

- the investigation objective;
- evidence consulted;
- claims and alternatives;
- why a query was useful;
- what result changed;
- confidence and missing evidence;
- stopping reason;
- proposed next action.

Do not require or retain hidden token-level reasoning. It is sensitive, expensive, unstable across providers, and not proof of correctness.

## Bounded investigation loop

~~~mermaid
flowchart TD
    O["Load objective, case version, scope, budget"] --> H["Generate competing hypotheses"]
    H --> G["Identify lowest-cost discriminating gap"]
    G --> P["Propose typed query"]
    P --> V{"Broker validates and authorizes"}
    V -- Denied --> D["Record access or policy gap"]
    V -- Allowed --> X["Execute read-only query"]
    X --> R["Validate result and coverage"]
    R --> U["Update claims, contradictions, timeline"]
    D --> S{"Stop condition met?"}
    U --> S
    S -- No --> G
    S -- Yes --> F["Produce disposition, confidence, gaps, recommendation"]
~~~

### Loop invariants

1. The objective and scope are fixed outside the model.
2. Every tool call must test a hypothesis, fill a named gap, or validate a safety precondition.
3. The query broker—not the prompt—enforces access and budgets.
4. Tool output is data, even if it contains instructions or appears to be a system message.
5. A query result changes case state only through typed claims and evidence references.
6. The agent stops on sufficient evidence, exhausted budget, expired deadline, repeated non-progress, unsafe content, or required human expertise.

## Investigation phases

### 1. Orient

- Verify alert type, producer, mapping quality, time range, and sensor health.
- Identify the protected asset, identity, owner, and criticality.
- Establish whether the alert is a delivery duplicate, related alert, or new case.
- List missing canonical identifiers and coverage gaps.

### 2. Form alternatives

Generate a small set of plausible explanations:

- malicious activity;
- expected user, administrator, deployment, or automation behavior;
- misconfiguration, stale inventory, parser error, clock skew, or sensor defect;
- unresolved/other.

Avoid long speculative lists. Two to four distinct hypotheses are usually enough to prevent premature convergence.

### 3. Select the next test

Rank candidate queries by:

- expected ability to distinguish hypotheses;
- source authority and observation coverage;
- privacy and sensitivity exposure;
- latency and monetary cost;
- fan-out and operational load;
- likelihood that a human must interpret the result.

Prefer a precise identity audit query over a broad search of all employee activity, and a known process tree over arbitrary endpoint shell access.

### 4. Gather and validate

For each result:

- validate schema and tenant;
- record row count, pagination, truncation, time coverage, freshness, and source health;
- keep the raw response reference;
- flag source-controlled instruction-like text;
- distinguish no data, no match, access denied, retention gap, timeout, parser failure, and source outage.

### 5. Update

Append narrow claims, update hypotheses, and record contradictions. Recalculate case priority only through an explicit, versioned policy that combines technical and business context.

### 6. Stop and hand off

Return a structured result even when incomplete:

- proposed disposition;
- confidence with rationale;
- key supported and contradicted claims;
- strongest alternative explanation;
- unresolved gaps and their impact;
- next safe query or human specialist;
- response recommendation, if any;
- explicit stopping reason.

## Stopping conditions

| Stop reason | Required output |
|---|---|
| Sufficient for analyst decision | Minimal evidence set, alternatives considered, recommended disposition |
| Budget or deadline exhausted | Work completed, untested hypotheses, highest-value next query |
| Source unavailable or incomplete | Exact coverage gap; no benign inference |
| Access denied | Requested scope and why it mattered; no privilege escalation attempt |
| Evidence conflict | Conflicting sources, freshness, authority, and requested human resolution |
| Active or unsafe artifact | Quarantine reference and specialist workflow |
| Repeated non-progress | Last distinct result and loop counter; request review |
| Required expertise | Forensics, malware, legal, privacy, identity, or service-owner handoff |
| Response decision needed | Exact proposal, risks, preservation considerations, approval owner |

“I am not sure” is not enough. Abstention must explain the missing evidence and the consequence of guessing.

## Tool architecture

### Tool classes

| Class | Examples | Default authority |
|---|---|---|
| Case reads | Read case snapshot, prior analyst-confirmed findings | Read-only |
| Evidence reads | Fetch excerpt, verify digest, inspect metadata | Read-only, content-isolated |
| Telemetry queries | SIEM search template, EDR process tree, cloud audit lookup | Read-only, bounded |
| Context lookups | CMDB asset, identity directory, deployment/change record | Read-only, field-filtered |
| CTI lookups | Indicator reputation, ATT&CK object, signed advisory | Read-only, marked and cached |
| Analysis functions | Decode timestamp, parse known format, compute hash on copy | Pure or sandboxed |
| Case writes | Append proposed claim, task, or note | Typed, idempotent, supervised |
| Response actions | Isolate endpoint, revoke session, block indicator | Separate executor and approval |

Do not expose a generic HTTP client, SQL console, SIEM query language, shell, remote-management interface, or unrestricted file reader to the model in production. If a specialist workflow needs one, put it in a separate sandbox with a human operator and explicit evidence procedures.

### Typed tool proposal

~~~yaml
tool_call:
  tool: identity.sign_in_history
  schema_version: 3
  purpose: Test whether the source session is consistent with the user's normal device and authentication path.
  case_id: case_781
  case_version: 17
  tenant_id: tenant_acme
  arguments:
    subject_id: idp:user:8c21
    start_time: 2026-08-31T03:45:00Z
    end_time: 2026-08-31T04:25:00Z
    fields:
      - session_id
      - device_id
      - auth_method
      - ip
      - result
  expected_evidence:
    discriminates: [hyp_7, hyp_8]
  budget:
    max_rows: 200
    timeout_ms: 8000
~~~

The broker ignores model-supplied tenant authority and derives the actual tenant, resource scope, and allowed fields from the authenticated run.

### Tool response

~~~yaml
tool_result:
  call_id: call_339
  status: partial
  source: identity-provider/prod
  executed_at: 2026-08-31T04:19:01Z
  coverage:
    requested_start: 2026-08-31T03:45:00Z
    requested_end: 2026-08-31T04:25:00Z
    available_start: 2026-08-31T03:58:00Z
    available_end: 2026-08-31T04:19:00Z
    complete: false
  data_ref: evidence://tenant_acme/query/call_339
  rows: 9
  truncated: false
  warnings:
    - Retention tier does not cover the first 13 minutes.
~~~

This prevents “zero rows” from collapsing time coverage, authorization, and source status into a misleading empty list.

## Query compilation and enforcement

Use one of these patterns, in order of preference:

1. Fixed parameterized query templates for recurring alert families.
2. A typed abstract query tree compiled by a trusted adapter.
3. Reviewed source-native query text in a specialist workflow.

The compiler should:

- allowlist fields, operators, joins, functions, indices, and time range;
- inject tenant filters independently;
- reject wildcards over sensitive or high-cardinality fields;
- estimate cost and require pagination;
- cap rows, bytes, scan range, runtime, and concurrency;
- prohibit mutation syntax and stored-procedure or extension execution;
- normalize identifiers before authorization;
- record the compiled query separately from the model proposal.

A source API labeling an operation “search” or “GET” does not prove it is harmless; some queries trigger exports, remote collection, or expensive scans.

## Model responsibilities and selection

### Suitable model work

- translate heterogeneous evidence into narrow claims;
- compare competing explanations;
- choose among already authorized query templates;
- extract structured observables with citations;
- summarize a timeline without overstating order;
- draft an analyst-facing case narrative;
- identify missing evidence and recommend escalation.

### Keep deterministic

- authorization and tenant filtering;
- alert deduplication and idempotency;
- timestamp parsing, hashing, arithmetic, severity policy, and SLA timers;
- query syntax validation and cost limits;
- action preconditions, approval binding, execution, and reconciliation;
- evidence integrity and custody;
- retention, deletion, and legal hold;
- final machine state used for evaluation.

### Model selection scorecard

Evaluate models on the exact workload and tools. Weight:

| Dimension | What to test |
|---|---|
| Evidence faithfulness | Unsupported claim rate; citation correctness; contradiction handling |
| Investigation quality | Hypothesis diversity; discriminating-query choice; silent-intrusion discovery |
| Security | Indirect injection; data exfiltration attempts; scope escalation; unsafe action proposals |
| Calibration | Confidence versus correctness; abstention quality; sensitivity to missing coverage |
| Tool reliability | Schema adherence; invalid arguments; unnecessary calls; loop termination |
| Operational fit | Latency distribution, throughput, rate limits, regional availability, incident support |
| Data governance | Training/retention terms, content logging, deletion, residency, encryption, access controls |
| Cost | Tokens, cached tokens, tool calls, retries, fallback frequency, analyst correction time |

Do not choose a “cyber model” from a generic multiple-choice or capture-the-flag score alone. Security investigation is evidence retrieval and judgment under uncertainty, not just security knowledge.

### Routing strategy

Start simple:

- deterministic path for known duplicates, malformed events, and exact playbook prerequisites;
- one cost-efficient model for normal read-only triage;
- a stronger model only for bounded ambiguous cases or independent review;
- no model for response authorization.

Escalation triggers may include evidence conflict, high criticality, low confidence, complex cross-domain scope, or a proposed consequential action. Validate that routing improves outcomes enough to justify latency and cost.

## Language and runtime choices

Prefer the language and deployment stack the security platform team already operates well. The main requirement is typed boundaries and reliable integrations, not a fashionable agent framework.

| Choice | Strength | Risk | Best fit |
|---|---|---|---|
| Python | Security/data ecosystem, rapid adapters, mature schema and analysis libraries | Blocking I/O, process isolation, packaging, and type discipline require care | Investigation service and offline evaluation |
| TypeScript/Node.js | Strong API integration, JSON schemas, streaming, shared web/service types | CPU-heavy parsing and unbounded async concurrency can stall the event loop | Case/SOAR integration and control APIs |
| Go | Small static services, explicit concurrency, predictable deployment, strong brokers | Fewer high-level model/security-analysis libraries | Query broker, policy proxy, effect executor |
| JVM or .NET | Enterprise identity, mature operations, strong typing and concurrency | More ceremony for experimental adapters | Existing enterprise security platforms |

A pragmatic deployment may use Python for isolated analysis and the organization's standard service language for policy, credentials, and effects. Do not split languages unless the security boundary or existing ownership justifies the operational cost.

### Runtime selection

| Runtime | Choose when | Avoid when |
|---|---|---|
| Stateless request worker | Bundle is small; no waits; read-only; retry is simple | Investigation spans minutes, approvals, or partial external effects |
| Queue-backed state machine | Need backpressure, retries, resumable phases, independent workers | Team cannot own state transitions and reconciliation |
| Durable workflow runtime | Human waits, long source calls, crash recovery, timers, and multi-step effects are material | Work is a single bounded read-only request or team cannot manage replay/versioning |
| Batch/offline pipeline | Backfill, eval, threat report processing, historical correlation | Interactive incident decisions |

Durable execution records progress; it does not make activities idempotent or model calls deterministic. Store model and tool results as activity outcomes, keep workflow code replay-safe, and version long-lived workflows.

## Structured final output

~~~yaml
investigation_result:
  schema_version: 2
  case_id: case_781
  based_on_case_version: 17
  proposed_disposition: escalate
  confidence:
    level: moderate
    rationale: Two independent identity records support misuse, but endpoint coverage is incomplete.
  claims:
    supported: [clm_204, clm_207]
    contradicted: [clm_199]
    unresolved: [clm_211]
  strongest_alternative:
    hypothesis_id: hyp_8
    summary: Approved automation using a newly created application.
    discriminating_gap: Change record and owner attestation.
  limitations:
    - Endpoint sensor was offline for 11 minutes.
  next_step:
    type: human_review
    owner_role: identity-incident-lead
  response_proposal: null
  stop_reason: response_decision_requires_accountable_owner
~~~

Validate this schema, evidence references, case version, and allowed enum values outside the model. Reject unsupported citations rather than repairing them silently.

## Why not default to multiple agents

Multiple agents can provide isolated context, independent checking, or specialized tools. They also multiply:

- untrusted message paths;
- inconsistent case views;
- tool calls and cost;
- race conditions and duplicate effects;
- prompt-injection propagation;
- debugging and evaluation complexity.

Add a separate verifier only when it has a distinct evidence source or scoring rubric and improves measured outcomes. Two models agreeing from the same incomplete context are not independent corroboration.

## Failure matrix

| Failure | Detection | Safe response |
|---|---|---|
| Repeated same query | Normalized call fingerprint | Stop loop; mark non-progress |
| Hallucinated tool or field | Schema/catalog rejection | Return allowed alternatives; do not fuzzy-match |
| Excessive fan-out | Budget and concurrency counter | Deny and request narrower hypothesis |
| Model favors leading alert narrative | Hypothesis/contradiction grader | Require strongest benign alternative and discriminator |
| Result includes injection | Source-sink monitor, canary tests | Keep as data; block scope change or egress |
| Context exceeds budget | Deterministic compiler metrics | Prioritize evidence, preserve omissions in manifest |
| Model endpoint changes behavior | Version and canary drift | Hold rollout; fall back to pinned path |
| Durable replay repeats model call | Replay/activity test | Persist activity result; keep call outside deterministic workflow code |

## Production checklist

- [ ] Tool schemas are narrower than source APIs.
- [ ] The broker derives tenant and permissions independently.
- [ ] All results carry coverage and source-health metadata.
- [ ] The loop has call, byte, time, cost, and non-progress limits.
- [ ] Final output is schema-validated and every citation resolves.
- [ ] Model choice is based on workload-specific outcome, security, calibration, latency, and cost.
- [ ] Model, prompt, tool, policy, and retrieval versions are logged.
- [ ] No response credential is present in model or read-only worker memory.
- [ ] Durable workflow replay tests cover active historical versions.

## Related guides

- [Evidence intake, context, and case state](evidence-intake-context-and-case-state.md)
- [Security integrations and adapter qualification](security-integrations-and-adapter-qualification.md)
- [Authority, approvals, and constrained response](authority-approvals-and-constrained-response.md)
- [Evaluation, rollout, and build roadmap](evaluation-rollout-and-build-roadmap.md)
- [Choosing an agent runtime language](../../languages/choosing-an-agent-runtime-language.md)
- [Durable agent workflow runtimes](../../comparisons/durable-agent-workflow-runtimes.md)

## Selected sources

- [FIRST CSIRT Services Framework 2.1](https://www.first.org/standards/frameworks/csirts/csirt_services_framework_v2.1)
- [NIST SP 800-207A, cloud-native zero trust](https://csrc.nist.gov/pubs/sp/800/207/a/final)
- [Temporal event history and replay](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/event-history/event-history.mdx)
- [Temporal retry policies and activity idempotence](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/retry-policies.mdx)
- [OpenAI, Designing AI agents to resist prompt injection](https://openai.com/index/designing-agents-to-resist-prompt-injection/)
- [AgentDojo, NeurIPS 2024](https://proceedings.neurips.cc/paper_files/paper/2024/hash/97091a5177d8dc64b1da8bf3e1f6fb54-Abstract-Datasets_and_Benchmarks_Track.html)
- [SecRespond benchmark and paper](https://arxiv.org/abs/2607.26791)
- [SecAlertBench repository](https://github.com/Dxsssu/SecAlertBench)

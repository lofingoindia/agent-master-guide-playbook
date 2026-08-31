# Tools, Integrations, Security, and Governance

## Trust model

The model is an untrusted proposal generator operating over partially untrusted evidence. Metric labels, semantic descriptions, dashboard annotations, ticket comments, chat messages, source metadata, and tool errors can contain misleading or adversarial instructions.

~~~mermaid
flowchart LR
    ID[Authenticated identity] --> AZ[Authorization and purpose check]
    AZ --> BR[Credential and tool broker]
    BR --> RT[Read tools]
    RT --> EG[Evidence gateway]
    EG --> CM[Context compiler]
    CM --> M[Model]
    M --> OV[Output and evidence validator]
    OV --> PO[Deterministic policy]
    PO --> AP[Approval]
    AP --> ET[Effect tool]
    ET --> RV[Receipt and verifier]
~~~

Credentials never enter the prompt. A model cannot call a raw HTTP client, shell, SQL console, or generic connector in production.

## Integration map

| Integration | Read/effect | Minimum contract | System of record |
|---|---|---|---|
| Semantic layer | Read | Metric ref, version, dimensions, filters, interval, rights, cost estimate | Semantic platform |
| Warehouse/query engine | Indirect read through semantic adapter | Query job, snapshot/as-of, status, bytes/cost, artifact | Warehouse |
| Data quality | Read | Assertion version, verdict, severity, expected/actual, time | Quality platform |
| Catalog/lineage | Read | Dataset/run refs, schema, owner, lineage completeness, freshness | Catalog/lineage platform |
| Identity/directory | Read | Subject, role, group, tenant, active status, assurance level | Identity provider |
| Known-event/change calendar | Read | Event type, approved time window, scope, owner, provenance | Change/business calendar |
| Notification | Effect | Destination policy, payload digest, operation key, remote receipt | Notification provider |
| ITSM/task system | Read/effect | Queue, type, owner, case link, operation key, status, remote ID | ITSM/task platform |
| Acknowledgement API | Effect into controller | Authenticated subject, case version, disposition capability | Agent state store |
| Outcome source | Read | Outcome definition, interval, semantic snapshot, value, quality | Source or semantic platform |
| Model provider | Proposal | Model release, prompt, context manifest, structured response | Agent audit record |
| Observability/audit | Write-only export | Trace/event correlation, redaction, retention | Telemetry/audit platform |

The [adapter qualification playbook](10-adapter-qualification-and-integration-playbooks.md) turns these minimum contracts into provider-specific rights, freshness, cost, finality, identity, reconciliation, failure, and release tests. Do not promote an integration from this map alone.

## Typed tool contracts

Follow the repository's [tool-contract guidance](../../tools/tool-contracts.md). Every adapter declares:

- purpose and actor;
- tenant, environment, and data classification;
- structural schema and supported versions;
- authorization and credential scope;
- timeouts, retry ownership, and rate limits;
- idempotency and reconciliation semantics;
- freshness and provenance;
- possible partial, ambiguous, or delayed outcomes;
- cost and result-size limits;
- redaction and retention behavior.

### Semantic read example

~~~yaml
tool: semantic_metric_query
version: 3
request:
  tenant_id: commerce
  purpose: commercial-monitoring
  metric_ref: semantic://net_revenue
  semantic_snapshot: sha256:4b7...
  interval: [2026-08-28T00:00:00+02:00, 2026-08-29T00:00:00+02:00]
  grain: day
  filters: {region: EMEA}
  group_by: []
  maximum_rows: 1
  operation_key: eval/revenue-drop-emea/v17/2026-08-28
response:
  status: complete             # complete | partial | error | unknown
  result_artifact_ref: artifact://semantic-result/918...
  query_artifact_ref: artifact://query/98a...
  as_of: 2026-08-29T06:59:00Z
  snapshot_refs: [warehouse://job/772, table://orders/snapshot/991]
  bytes_processed: 8810231
  rights_decision_id: authz_8d...
  provenance_hash: sha256:119...
~~~

The adapter verifies that requested dimensions and filters are permitted by the metric and watch contracts. It rejects novel expressions and generated SQL.

### Effect example

~~~yaml
tool: create_work_item
version: 2
request:
  tenant_id: commerce
  case_id: case_01K...
  case_version: 6
  target_queue: EMEA-COMMERCIAL
  work_item_type: metric_investigation
  payload_artifact_ref: artifact://decision-packet/19a...
  payload_digest: sha256:bc3...
  operation_key: commerce/case_01K/v6/itsm/create-investigation
  approval_ref: approval_88...
  deadline: 2026-08-29T07:12:00Z
response:
  status: confirmed            # confirmed | rejected | failed | unknown
  remote_id: INC00182
  receipt_ref: receipt://itsm/332...
  occurred_at: 2026-08-29T07:11:19Z
~~~

An `unknown` response cannot be normalized to failure.

## Tool validation pipeline

~~~mermaid
flowchart TD
    R[Proposed tool request] --> S[Schema and enum validation]
    S --> D[Domain contract validation]
    D --> A[Identity, tenant, purpose, and rights]
    A --> F[Freshness and version checks]
    F --> B[Budget, rate, and result-size checks]
    B --> X[Execute with scoped credential]
    X --> P[Parse and provenance validation]
    P --> C{Complete and trustworthy?}
    C -- Yes --> E[Evidence envelope or receipt]
    C -- Partial --> H[Typed partial result and stop/replan]
    C -- Ambiguous effect --> U[Unknown outcome and reconcile]
    C -- Invalid --> Q[Quarantine and incident signal]
~~~

Model-visible output includes typed values, provenance references, trust labels, and errors—not stack traces, raw secrets, or arbitrary provider HTML.

## Least privilege and credentials

Separate capabilities:

- `semantic:read:metric` scoped to metric, dimensions, purpose, and tenant;
- `quality:read` and `lineage:read` scoped to referenced data products;
- `cases:read/write` scoped to tenant and role;
- `notify:create` scoped to approved destinations;
- `workitem:create/update` scoped to approved queues and fields;
- `admin:watch-promote` and `admin:repair` reserved for human operators.

The effect worker uses short-lived workload identity or just-in-time credentials. The controller stores only credential references. Deny wildcard connector scopes, user-delegated ambient sessions, and arbitrary destination addresses.

## Prompt-injection containment

### Untrusted sources

- dashboard titles, descriptions, annotations, and alt text;
- metric and column descriptions imported from source systems;
- tickets, comments, collaboration messages, and linked documents;
- error strings, log fields, URLs, and remote tool payloads;
- previous model outputs and outcome-episode narratives.

### Required defenses

1. Treat all of the above as data and label its provenance/trust.
2. Keep authority and policy in a separate compiler-owned section.
3. Normalize tool results to allowlisted fields; drop active content and unexpected links.
4. Do not let content select tools, recipients, credentials, or scopes.
5. Require structured model output and validate citations against the manifest.
6. Apply tenant, purpose, and data rights before retrieval.
7. Route high-impact or suspicious cases to human review.
8. Evaluate direct, indirect, encoded, multi-hop, and memory-persistence attacks.

String matching alone is not an adequate defense.

## Privacy and source-data rights

Monitoring can create a surveillance surface even when the source dashboard is legitimate. A new watch, slice, or destination may be a new purpose.

Control questions:

- Is this metric and slice necessary for the documented decision?
- Does the role have source rights at query time and destination rights at delivery time?
- Are protected or inferred sensitive categories involved?
- Could a group-by or top/bottom list expose a small cohort or individual?
- Is the model provider and processing region allowed?
- How long are observations, evidence, prompts, cases, and episodes retained?
- Can access, correction, deletion, legal hold, and audit obligations be fulfilled?
- Is automated or consequential decision law triggered in this jurisdiction?

The GDPR principles of purpose limitation, data minimization, accuracy, storage limitation, integrity, and accountability are useful design constraints where applicable. The EU AI Act, UK guidance, Canadian directives, sector regulation, contracts, and employment or consumer laws may impose different obligations. This guide is not legal advice; conduct jurisdiction- and use-case-specific review.

### Human oversight

Meaningful review requires a human who:

- has the authority and competence to decide;
- sees source-backed facts, uncertainty, alternative dispositions, and impact;
- has enough time to challenge the system;
- can reject or modify the proposal without penalty;
- can pause the watch or effect path;
- is not asked to rubber-stamp hundreds of low-quality alerts.

An approval button does not create meaningful oversight if alert volume or interface design makes independent judgment unrealistic.

## Multi-tenant isolation

Tenant isolation must appear in:

- identity and policy decisions;
- state and effect primary keys;
- queue partitions and worker claims;
- semantic/catalog/query credentials;
- artifact namespaces and encryption keys where required;
- retrieval filters applied before similarity search;
- observability attributes and access controls;
- rate, cost, model, and channel budgets;
- backup, restore, deletion, and incident tooling.

Tests must attempt cross-tenant references using valid IDs, guessed IDs, stale context, shared caches, malformed events, and model-generated tool arguments.

## Destination governance

Maintain a matrix:

| Data class | Email/chat | ITSM | Pager | Model prompt | Long-term episode |
|---|---|---|---|---|---|
| Public aggregate | Allowed | Allowed | Allowed if actionable | Allowed | Allowed under retention |
| Internal aggregate | Approved groups | Approved queues | Redacted | Approved provider/region | Structured fields only |
| Confidential aggregate | Link-only or redacted | Restricted queue | Metadata only | Redacted or prohibited | Restricted and time-limited |
| Individual/sensitive | Prohibited by default | Case-specific controlled record | Metadata only | Prohibited by default | Disabled unless explicit governance |

Resolve every destination against current policy at effect time. Stored payloads can become stale when recipients or classifications change.

## Audit requirements

Record:

- watch, semantic, detector, baseline, decision, policy, prompt, model, adapter, and runtime versions;
- actor identity and assurance, tenant, purpose, data class, and rights decision;
- evidence manifest and artifact hashes;
- model proposal and validator decision, subject to minimization policy;
- command, state versions, domain events, approvals, effects, attempts, receipts, and reconciliation;
- administrative changes, export, access, deletion, and repair actions;
- release, rollback, incident, and outcome-evaluation linkage.

Audit records must be immutable enough for the risk, access controlled, time synchronized, retention governed, and independently reviewable. Do not log secrets, raw sensitive rows, or complete prompts by default.

## Integration-specific cautions

| System | Caution |
|---|---|
| BI alerting | Alert identity may be tied to a tile, subscription, refresh, or individual user; test edit/migration behavior |
| Semantic API | Generated SQL, snapshot pinning, and compatibility behavior vary; capture the actual query artifact |
| Warehouse | Timeout may leave a running/complete job; query by job or operation identity before retry |
| ITSM | Acknowledgement, assignment, incident creation, resolution, and closure have product-specific meanings |
| Chat/email | Delivery/read receipts do not prove authorized acknowledgement |
| Event rule engine | Deleting or disabling a rule may not stop already queued actions immediately |
| Model provider | Model aliases, data retention, region, rate limits, and structured-output behavior can change |
| Lineage/quality | Facets carry evidence but do not establish local enforcement policy |

## Supply chain and licensing

Pin adapters and review transitive dependencies, images, model releases, and provider terms. Representative open-source licenses differ:

- MetricFlow, OpenLineage, Prometheus/Alertmanager, CloudEvents, and the Open Data Contract Standard use Apache-2.0 licensing in the referenced repositories.
- Numenta Anomaly Benchmark and TimeEval use MIT licensing in the referenced repositories.
- Grafana's repository uses AGPL-3.0-only by default with documented exceptions; hosted or modified deployment requires legal review.
- Standards text, SaaS APIs, and documentation have separate terms; an open protocol does not make every implementation open source.

Record exact component/version/license evidence in the deployment bill of materials. This summary is not legal advice.

## Security test checklist

- [ ] Model has no raw SQL, shell, HTTP, source-write, credential, or recipient-selection tool.
- [ ] Read and effect identities are separate and short lived.
- [ ] Every tool validates schema, domain, rights, freshness, budget, provenance, and result size.
- [ ] Untrusted content cannot change authority or select a destination.
- [ ] Tenant/purpose filters run before retrieval and query.
- [ ] Cohort floors and dimension allowlists are enforced outside the model.
- [ ] Sensitive values are absent from low-trust destinations and routine telemetry.
- [ ] Approvals are exact, expiring, version-bound, and challengeable.
- [ ] Cross-tenant, indirect-injection, stale-context, link-following, and exfiltration tests pass.
- [ ] Source mutation remains impossible even if policy/model layers are compromised.
- [ ] Dependency, model, adapter, and license inventory is reproducible.

## Primary references

- [OWASP AI Agent Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html)
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [NIST Privacy Framework](https://www.nist.gov/privacy-framework)
- [GDPR official text](https://eur-lex.europa.eu/eli/reg/2016/679/oj)
- [EU AI Act official text](https://eur-lex.europa.eu/eli/reg/2024/1689/oj?locale=en)
- [ICO guidance on individual rights and human review in AI systems](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/artificial-intelligence/guidance-on-ai-and-data-protection/how-do-we-ensure-individual-rights-in-our-ai-systems/)
- [Permissions, sandboxing, and secrets](../../security/permissions-sandboxing-and-secrets.md)
- [Prompt injection and untrusted data](../../security/prompt-injection-and-untrusted-data.md)

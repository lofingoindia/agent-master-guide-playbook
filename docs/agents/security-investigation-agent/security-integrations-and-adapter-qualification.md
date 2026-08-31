# Security Integrations and Adapter Qualification

> **Status:** Research-backed production blueprint  
> **Last researched:** 2026-08-31  
> **Scope:** SIEM, SOAR, EDR, cloud, IAM, CTI, ticket, case, and asset-context integrations; adapter contracts, qualification, drift, release, and recovery.  
> **Section index:** [Security investigation and triage agent](README.md)

An integration is not “done” when an API call returns JSON. It is ready when the system can prove which tenant and source were queried, what interval and fields were covered, whether every page was consumed, which schema and permission set applied, what was omitted, and how an ambiguous write will be reconciled.

Keep vendor behavior inside typed adapters. The model selects an allowed investigative intent; a broker authorizes it; the adapter implements the pinned vendor contract; the evidence and case services preserve the receipt. Never teach the model a vendor's general query language or hand it the vendor credential.

## Integration topology

~~~mermaid
flowchart LR
    A["Investigation intent"] --> B["Tenant-aware broker"]
    B --> C{"Allowed operation?"}
    C -- No --> D["Typed denial and gap"]
    C -- Yes --> E["Pinned adapter"]
    E --> F["SIEM / EDR / cloud / IAM / CTI"]
    F --> E
    E --> G["Normalized result + native reference + coverage"]
    G --> H["Evidence and case services"]
    A -. "sealed write proposal" .-> P["Policy + exact approval"]
    P --> X["Separate write/effect adapter"]
    X --> Y["SOAR / case / response API"]
    Y --> Z["Observed-state receipt"]
~~~

Read and write adapters use different workload identities, deployments, network paths, and catalogs. A connector that can search endpoints and isolate them is exposed as two products: an investigation read adapter and a separately governed response actuator.

## Source-family authority map

| Source family | Useful read operations | Writes the agent may propose | Authority that remains external |
|---|---|---|---|
| SIEM and detection platform | Alert by native ID, bounded event search, rule/version metadata, ingestion and sensor health | Proposed tag, linked evidence reference, draft tuning issue | Native alert disposition, suppression, rule promotion, broad search ACL |
| SOAR and case platform | Case snapshot, task/playbook status, approvals, prior confirmed decisions | Proposed claim/note, assigned task, sealed playbook step | Confirmed verdict, case closure, approval, playbook publication |
| EDR/XDR | Device identity and health, process tree, timeline, alert/evidence lookup, action status | Exact isolation or collection proposal | Isolation, live response, quarantine, collection, rollback |
| Cloud control plane and security posture | Audit event, resource/config version, finding, owner, region, policy, sensor coverage | Proposed security-finding annotation or workflow state | Resource change, IAM change, finding closure/mute, organization policy |
| IAM and SaaS identity | Sign-in/audit history, session/device facts, directory/role version | Exact session-revocation or account-containment proposal | Token/session revocation, disablement, role/group/policy change |
| CTI and knowledge sources | STIX/TAXII objects, indicator/provider response, ATT&CK/Sigma/D3FEND version | Curated candidate observable or relationship | Trust decision, feed promotion, detection or block publication |
| Ticket, chat, and notification | Issue/case/task state and approved discussion | Draft comment, proposed task, notification preview | Assignment acceptance, external communication, closure, paging |
| CMDB, service registry, and ownership | Canonical resource, owner, criticality, dependencies, recovery constraints | Correction proposal with evidence | Authoritative asset/owner/criticality mutation |

“Read-only” still needs a safety review. A query can retrieve another tenant's evidence, expose employee data, launch a costly export, trigger endpoint collection, or exhaust a shared SIEM.

## Adapter qualification manifest

Maintain a reviewed manifest for every source instance, not merely every vendor:

~~~yaml
adapter_qualification:
  adapter: microsoft-graph-security-alerts
  adapter_version: 3.4.0
  source_instance: tenant-acme-global
  source_family: siem_xdr
  api:
    surface: /v1.0/security/alerts_v2
    documentation_reviewed_at: 2026-08-31
    contract_digest: sha256:...
  identity:
    workload: soc-query-broker
    tenant_binding: credential_and_host_mapping
    granted_scopes: [SecurityAlert.Read.All]
    forbidden_scopes: [SecurityAlert.ReadWrite.All]
  operations:
    - alert.get_by_id
    - alert.list_bounded
  query:
    required_filters: [created_time_range]
    max_window_seconds: 3600
    max_pages: 20
    max_rows: 2000
    timeout_seconds: 20
  pagination:
    mechanism: odata_next_link
    terminal_condition: next_link_absent
    query_parameters_must_remain_stable: true
  semantics:
    ordering: vendor_documented
    retention_source: tenant_policy
    empty_means: no_match_only_when_coverage_complete
  reliability:
    retryable: [connect_before_send, 429, selected_5xx]
    ambiguous_read: [timeout_after_partial_page]
    rate_limit_partition: tenant_and_operation
  output:
    schema: security-tool-result/v3
    preserve_native_object: true
    field_mapping_version: graph-alert-v7
  ownership:
    technical_owner: security-platform
    data_owner: soc
    on_call: sec-platform-primary
  release:
    fixture_suite: graph-alerts-2026q3
    last_qualified: 2026-08-31
    rollback_adapter_version: 3.3.2
~~~

The manifest is executable policy input and review evidence. Do not expose its credentials, hostnames, internal limits, or forbidden-scope details to the model unless required for a safe refusal.

## Identity, tenancy, and region binding

The adapter derives authority from authenticated runtime state, never tool arguments authored by the model.

At minimum bind:

- end-user or analyst identity, workload identity, tenant/customer, source instance, and region;
- permitted operations, fields, resources, and time ranges;
- case purpose and retention/legal-hold context;
- source credential audience and expiry;
- data residency, sharing, and export restrictions.

Use separate source instances in configuration when a vendor host serves multiple environments or regions. Canonical resource IDs include vendor, tenant/account/subscription/project, region where material, resource type, and native ID. A hostname, email address, display name, IP address, or device name alone is not a security boundary.

Perform negative authorization tests from every tenant against every other tenant's known fixture. A successful “not found” is insufficient if the upstream call actually searched both tenants and the adapter discarded the foreign row.

## Inbound alert and webhook contract

Webhook delivery and source polling need the same logical guarantees:

1. authenticate the producer and validate replay window before parsing content;
2. bind producer to source instance and tenant through server-owned configuration;
3. store delivery ID and native record ID separately;
4. preserve raw headers/body by governed reference before normalization;
5. acknowledge only after durable intake or use a documented replay/reconciliation path;
6. treat delivery as at-least-once unless the exact source contract proves otherwise;
7. reconcile missed intervals with a bounded source query;
8. record schema/parser version and all mapping loss.

Webhooks are hints, not necessarily the complete evidence record. Fetch the authoritative object when the source supports it, but retain both the received payload and the fetched version because they may differ.

## Read-query and coverage receipt

Every source adapter returns data plus a machine-checkable coverage object:

~~~yaml
query_receipt:
  call_id: call_339
  operation: iam.sign_in_history
  source_instance: entra-acme
  source_api_version: v1.0
  tenant_id: tenant_acme
  canonical_request_digest: sha256:...
  requested:
    start: 2026-08-31T03:45:00Z
    end: 2026-08-31T04:25:00Z
    subject: entra:tenant_acme:user:8c21
    fields: [id, createdDateTime, appId, deviceDetail, status]
  execution:
    started_at: 2026-08-31T04:19:00Z
    completed_at: 2026-08-31T04:19:02Z
    pages: 3
    rows: 117
    pagination_complete: true
    truncated: false
    source_health: healthy
  coverage:
    available_start: 2026-08-31T03:58:00Z
    available_end: 2026-08-31T04:19:00Z
    complete_for_requested_window: false
    gaps:
      - start: 2026-08-31T03:45:00Z
        end: 2026-08-31T03:58:00Z
        reason: source_retention_or_ingestion_gap
  result_ref: evidence://tenant_acme/query/call_339
  native_result_digest: sha256:...
  warnings: []
~~~

Zero returned rows has meaning only when authorization succeeded, every page was consumed, the source was healthy, the requested interval is within available retention, filters used the correct semantics, and no truncation or delayed ingestion applies.

### Pagination and ordering qualification

Do not implement one generic “next page” helper and assume equivalent semantics.

| API pattern | Required adapter behavior |
|---|---|
| Opaque next link/token | Treat it as untrusted opaque data; validate destination host/path; preserve all original filters; stop only at the documented terminal signal |
| Offset/limit | Add stable ordering and overlap/deduplication tests; concurrent inserts/deletes can skip or repeat rows |
| Time cursor | Record inclusive/exclusive boundary, precision, tie-breaker, and source clock; overlap windows and deduplicate native IDs |
| Export or asynchronous search job | Persist job ID before polling; cap wait and bytes; retrieve all result partitions; cancel/expire jobs when supported |
| Streaming subscription | Persist checkpoint after durable item commit; detect lag, partition movement, and retention overrun |

Current primary-source examples show why adapters need source-specific tests:

- Microsoft Graph `alerts_v2` uses OData filters and `@odata.nextLink`; availability is bounded by the environment retention policy.
- AWS CloudTrail `LookupEvents` currently covers recent regional management/Insights history with narrow lookup and rate limits; it is not a complete cloud evidence lake.
- Google Cloud Logging may return an empty `entries` array with a `nextPageToken`, so an empty page is not necessarily a terminal empty result.
- TAXII 2.1 can paginate with an opaque `next` value or `added_after` using `X-TAXII-Date-Added-Last`; clients must retain the original filters.
- Jira collection limits can differ by operation and change; clients must follow returned paging metadata rather than hard-code a presumed maximum.

These are qualification fixtures, not universal abstractions. Re-test them against the exact account, API version, SDK, permission set, region, and data volume in use.

## SIEM and search adapters

Prefer fixed parameterized searches for recurring alert families. If an abstract query is required, compile it into the vendor language after policy evaluation.

Qualify:

- index/data-view allowlists and field-level restrictions;
- tenant predicates injected by the broker;
- timestamp field, timezone, ingestion delay, and late-event behavior;
- query/job identity, cancellation, TTL, and result partitioning;
- row, byte, scan, runtime, concurrency, and monetary budgets;
- search-head/cluster failover and partial-shard behavior;
- raw-event reference and immutable export path;
- mapping to OCSF or an application schema without dropping native fields.

Do not expose unrestricted SPL, KQL, SQL, EQL, Lucene, or a general saved-search runner to the model. A reviewed analyst may use a specialist console; the production agent receives only typed operations such as `identity.sign_in_history` or `endpoint.process_tree`.

## EDR and XDR adapters

Separate at least three surfaces:

1. **telemetry read** — process trees, device timeline, sensor health, evidence retrieval;
2. **collection** — investigation package, memory/disk artifact, live response; often consequential and resource-intensive;
3. **containment** — isolate/unisolate, quarantine, stop process, restrict code; always an effect.

Qualification must establish device-ID stability, managed/unmanaged distinction, sensor health, action eligibility by platform, management-channel behavior, action queue semantics, and authoritative action-status lookup. For example, Microsoft Defender for Endpoint's isolate API returns a machine-action record after accepting a request, and its documentation warns that full-tunnel VPN configuration can impair cloud-service reachability after isolation. Treat acceptance as `SUBMITTED`, not `OBSERVED_SUCCEEDED`, and test the actual management path and rollback in the target fleet.

Never infer endpoint containment from a successful HTTP status alone. Re-read action state and device isolation state; store the native action ID; page when isolation expires incorrectly or unisolate fails.

## Cloud and IAM adapters

Cloud and identity APIs frequently expose a collection that is narrower than its name suggests. Record whether a query covers management events, data events, network activity, control-plane audit, resource configuration, sign-ins, service principals, workload identities, or conditional-access details.

Required tests include:

- organization/account/subscription/project and region enumeration completeness;
- least-privilege field visibility—missing fields may mean insufficient permission, not null truth;
- retention tier and archive/lake boundary;
- delayed delivery and eventually consistent resource/config views;
- workload, human, federated, managed, and break-glass identity resolution;
- role/group membership at event time versus current effective privilege;
- stable resource and session identifiers;
- token/session revocation propagation and cache behavior for any effect adapter.

Microsoft Graph sign-in documentation recommends time filters to avoid timeouts and declares specific OData pagination behavior. AWS CloudTrail's event-history lookup is currently limited to 90 days and does not include every event category. Google Cloud Logging scopes reads through `resourceNames`, and one query can fan out across regions. Encode those constraints in the receipt so a model cannot turn missing coverage into a benign conclusion.

## Threat-intelligence adapters

A CTI adapter preserves, separately:

- feed/provider and collection identity;
- STIX object ID, type, `spec_version`, `created`, `modified`, and revoked state;
- TAXII date-added cursor and retrieval timestamp;
- object and granular markings, plus local handling decision;
- source reliability, information credibility, analytic confidence, and local relevance;
- license and redistribution terms;
- raw object/version and any local normalized representation;
- exact-match, fuzzy-match, relationship, or provider-score semantics.

Do not merge object versions by overwriting the old object. Do not treat a TAXII transport timestamp as the intelligence observation time. Do not let TLP or a STIX confidence field substitute for local access policy or truth validation.

## SOAR, ticket, and case adapters

Use application-specific APIs where available instead of a generic table or issue editor. If a general API is unavoidable, expose an allowlisted projection and typed operations only.

Enforce:

- separate proposed and confirmed fields;
- optimistic concurrency, revision/ETag, or a compare-before-write check;
- stable semantic key for comment, task, or transition intent;
- restricted project/table/queue and field allowlists;
- issue/case-level security, tenant, and regional rules;
- safe rendering and length limits for untrusted evidence excerpts;
- attachment quarantine and content scanning;
- source message/record ID and authoritative post-write readback;
- no model-authored closure, severity, owner, approval, or root-cause transition.

Jira Cloud documents operation-specific pagination and issue-level security, and its rate-limit contract includes `429` and `Retry-After`. ServiceNow supports versioned REST endpoints, ACL-controlled field visibility, and instance-defined inbound rate limits. An adapter cannot assume the same roles, writable fields, page size, or limits across tenants or platform upgrades.

## Write and effect contract

Each mutating adapter implements a prepare/authorize/commit/observe/reconcile contract:

| Phase | Required record | Failure meaning |
|---|---|---|
| Prepare | Canonical target, action, arguments, preconditions, expected postcondition, rollback, semantic key | No external effect |
| Authorize | Policy decision and exact expiring approval bound to current target/case/policy version | Denial is terminal until authority/state changes |
| Commit | Native request and operation/action ID | Timeout may mean `UNKNOWN` |
| Observe | Independent authoritative target read and timestamp | Acceptance is not success |
| Reconcile | Map native state to one logical effect and repair ledger/case projection | Never create a new intent to discover the old outcome |
| Roll back or expire | Sealed inverse/expiry operation plus observed result | Failure remains an active incident |

AWS Security Hub's customer update API, for example, limits which finding fields customers may update and does not create findings. Google Security Command Center exposes distinct operations for finding state and mute state. Microsoft Defender exposes action resources. Map such native distinctions into separate local tools; do not hide them behind `update_security_item`.

## Schema evolution and compatibility

Treat these as behavior changes:

- API version, endpoint, SDK, or authentication mode;
- permission/scope or role mapping;
- field, enum, nullability, ordering, pagination, or timestamp semantics;
- source retention, region, export, rate-limit, or consistency behavior;
- vendor alert correlation, suppression, priority, or machine-learning changes;
- OCSF/STIX/TAXII/ATT&CK/Sigma mapping or content snapshot;
- tool description, query template, output schema, or error taxonomy.

Preserve unknown enum values and unknown fields in native evidence. Fail closed for authority decisions when a new value has no reviewed mapping. A compatible parser may accept additive fields while a mapping-coverage monitor alerts; a removed field, changed type, or changed meaning blocks promotion.

Run dual-read comparison before replacing an adapter or mapping. Compare identity, counts, time coverage, missing/native fields, correlations, and case conclusions—not only JSON-schema validity.

## Qualification test plan

An adapter cannot enter production until these suites pass against fakes and an isolated real source instance:

| Suite | Minimum tests |
|---|---|
| Contract | Documented requests/responses, unknown enum, null/missing field, malformed body, content type, maximum sizes |
| Identity and tenancy | Correct tenant, wrong tenant, forged tenant argument, cross-region, deleted tenant, insufficient field permission |
| Completeness | Multiple pages, empty intermediate page, repeated page, token expiry, concurrent insert/delete, late event, retention boundary |
| Delivery | Duplicate webhook, reordered delivery, signature failure, replay, missed interval, poll/webhook reconciliation |
| Error taxonomy | Connect failure, timeout before/after response, 401/403/404 distinction, 409, 429 with/without delay, partial 5xx, vendor error body |
| Load and cost | Per-tenant/global concurrency, burst, queue storm, expensive query denial, backpressure, quota exhaustion |
| Security | Injection in every text field/error, malicious attachment, redirect/next-link host escape, SSRF, secret leakage, overbroad scope |
| Write safety | Duplicate intent, changed arguments under same key, stale revision, ambiguous commit, delayed action, partial batch, rollback failure |
| Drift and upgrade | Old/new fixture replay, live shadow dual-read, permission diff, schema diff, SDK/API upgrade, rollback |
| Operations | Credential rotation/revocation, kill switch, connector isolation, audit reconstruction, on-call alert, deterministic fallback |

Use production-shaped volume and cardinality. A connector that works on ten fixtures may fail on a tenant with millions of sign-ins, a 429 burst, or a query that fans out across regions.

## Adapter observability and SLOs

Every call emits low-cardinality operational telemetry and a restricted audit receipt. Track by adapter version, source instance, tenant class, and operation:

- request, success, denial, retry, timeout, malformed-response, and schema-drift counts;
- latency, queue time, rows, pages, bytes, scanned volume, and vendor cost;
- coverage-complete, truncated, empty-with-cursor, late-data, and source-health rates;
- permission/field-visibility changes;
- webhook lag, poll checkpoint age, and reconciliation gap;
- write `UNKNOWN` count and reconciliation age;
- rate-limit headroom, circuit-breaker state, and fallback use;
- native-to-normalized mapping coverage.

Useful SLO shapes include “urgent identity reads complete with declared coverage within the case deadline,” “no source checkpoint exceeds its recovery window,” and “every unknown EDR action is reconciled or paged within the action-specific limit.” Set values from source behavior and response needs; never promise an SLO tighter than the upstream API can support.

## Release, rollback, and incident response

Release one adapter change independently. The release manifest pins source documentation snapshot, API/SDK version, credential/scopes, manifest and mapping digest, query templates, tool schema/description, fixtures, and rollback version.

Promotion path:

1. fixture and adversarial contract tests;
2. non-production source integration;
3. production shadow read with no case-visible output;
4. dual-read comparison by alert family and tenant;
5. canary read or proposed-field write;
6. gradual tenant/source expansion;
7. separately approved effect-adapter canary.

Rollback must leave intake, raw evidence, native case access, manual investigation, and effect reconciliation available. Do not roll back a write adapter by forgetting its in-flight native operation IDs.

Page or declare an agent-platform incident for cross-tenant access, missing source intervals beyond recovery window, widespread schema loss, connector credential compromise, poisoned case writes, unknown response effects, or a kill switch that fails. Quarantine one connector without stopping other sources; revoke its identity; preserve its configuration/audit state; reconcile checkpoints and effects before re-enabling it.

## Common integration failures

| Symptom | Likely cause | Safe response |
|---|---|---|
| Zero rows for a known event | Wrong time field/zone, missing page, insufficient permission, retention gap, ingestion delay | Mark coverage incomplete; inspect native query/receipt; do not close benign |
| Duplicate case comments/tasks | Transport retry used a fresh random key | Reconcile by semantic intent; repair projection; reject key reuse with different content |
| Cross-tenant identifier appears | Source account mapping or tenant predicate failed | Hard stop, revoke connector, preserve evidence, incident review |
| Query cost spikes | Wildcard/fan-out, lost time predicate, changed vendor optimizer/schema | Open circuit, use deterministic fallback, compare compiled query and mapping version |
| New enum becomes “unknown benign” | Permissive default mapping | Quarantine/abstain for consequential decisions; add reviewed fixture and mapping |
| Accepted isolation never applies | Asynchronous action, unsupported device, lost management path | Keep `SUBMITTED`/`UNKNOWN`; poll native action and device state; escalate or roll back |
| CTI cursor advances but objects disappear | Pagination/deletion or collection change | Preserve cursor/page receipts; overlap/refetch; never infer revocation without object semantics |
| Analyst edits disappear | Blind case update overwrote concurrent change | Stop writes; restore from changelog; require revision-aware typed update |

## Adoption sequence

1. Integrate one alert source and immutable raw intake.
2. Add one read-only authoritative context source with complete coverage receipts.
3. Add one bounded SIEM/telemetry query template for the chosen alert family.
4. Add proposed case fields with optimistic concurrency and semantic idempotency.
5. Add CTI only if it measurably changes decisions; retain source quality and markings.
6. Add further sources one at a time with independent shadow evidence.
7. Add one response actuator only after its read path, action status, rollback, kill switch, and reconciliation have passed action-specific gates.

More integrations are not automatically better. Each source expands privacy exposure, schema drift, outage coupling, cost, and the attacker's injection surface. Remove or disable a source whose marginal decision value does not justify those risks.

## Integration acceptance checklist

- [ ] Every source instance has a reviewed qualification manifest, owner, on-call route, and kill switch.
- [ ] The broker derives tenant, source instance, fields, and time/cost limits outside the model.
- [ ] Raw source objects and native identifiers remain recoverable after normalization.
- [ ] Pagination, ordering, retention, late data, permissions, and empty-result semantics have source-specific tests.
- [ ] Each result declares coverage, source health, truncation, mapping version, and native evidence reference.
- [ ] Read, case-write, collection, and response surfaces use separate identities and authority.
- [ ] Case writes preserve proposal status, concurrency, semantic idempotency, and readback.
- [ ] Effects preserve native operation ID and reconcile observed state before retry.
- [ ] Schema/permission/rate-limit drift is observable and blocks unsafe promotion.
- [ ] Connector incidents can be contained while intake, manual response, evidence access, and reconciliation continue.

## Related guides

- [Evidence intake, context, and case state](evidence-intake-context-and-case-state.md)
- [Investigation reasoning, tools, models, and runtime](investigation-reasoning-tools-and-runtime.md)
- [Authority, approvals, and constrained response](authority-approvals-and-constrained-response.md)
- [Reliability, observability, scaling, and operations](reliability-observability-scaling-and-operations.md)
- [Tool results, artifacts, and provenance](../../tools/tool-results-artifacts-and-provenance.md)
- [Idempotency and side effects](../../reliability/idempotency-and-side-effects.md)

## Selected sources

- [Microsoft Graph: list security alerts v2](https://learn.microsoft.com/en-us/graph/api/security-list-alerts_v2?view=graph-rest-1.0)
- [Microsoft Graph: list sign-ins](https://learn.microsoft.com/en-us/graph/api/signin-list?view=graph-rest-1.0)
- [Microsoft Graph throttling guidance](https://learn.microsoft.com/en-us/graph/throttling)
- [Microsoft Defender for Endpoint: isolate machine API](https://learn.microsoft.com/en-us/defender-endpoint/api/isolate-machine)
- [Microsoft Defender for Endpoint: machine action resource](https://learn.microsoft.com/en-us/defender-endpoint/api/machineaction)
- [AWS CloudTrail `LookupEvents`](https://docs.aws.amazon.com/awscloudtrail/latest/APIReference/API_LookupEvents.html)
- [AWS Security Hub: customer finding updates](https://docs.aws.amazon.com/securityhub/latest/userguide/finding-update-batchupdatefindings.html)
- [Google Cloud Logging `entries.list`](https://cloud.google.com/logging/docs/reference/v2/rest/v2/entries/list)
- [Google Security Command Center REST reference](https://cloud.google.com/security-command-center/docs/reference/rest)
- [OASIS TAXII 2.1](https://docs.oasis-open.org/cti/taxii/v2.1/os/taxii-v2.1-os.html)
- [OASIS STIX 2.1 with Errata 01](https://docs.oasis-open.org/cti/stix/v2.1/stix-v2.1.html)
- [Jira Cloud REST API v3 introduction](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro/)
- [Jira Cloud rate limiting](https://developer.atlassian.com/cloud/jira/platform/rate-limiting/)
- [ServiceNow REST APIs and versioning](https://www.servicenow.com/docs/r/api-reference/rest-api-explorer/c_RESTAPI.html)
- [ServiceNow inbound REST rate limiting](https://www.servicenow.com/docs/r/api-reference/rest-api-explorer/inbound-REST-API-rate-limiting.html)
- [Splunk search endpoint descriptions](https://help.splunk.com/en/splunk-cloud-platform/leverage-rest-apis/rest-api-reference/10.3.2512/search-endpoints/search-endpoint-descriptions)

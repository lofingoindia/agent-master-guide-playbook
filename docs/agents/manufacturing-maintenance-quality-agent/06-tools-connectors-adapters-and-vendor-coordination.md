# Tools, Connectors, Adapters, and Vendor Coordination

The adapter layer is the safety and reliability boundary between model intent and vendor behavior. Expose small semantic operations with explicit schemas, capabilities, preconditions, and failure classes. Do not give the model generic protocol, database, browser, shell, or record-update access.

## Design semantic tools

Good tools describe a domain action:

```json
{
  "name": "create_work_order_draft",
  "input": {
    "site_id": "plant-a",
    "asset_resolution_token": "art_...",
    "case_id": "MC-2026-008812",
    "work_type": "CONDITION_INSPECTION",
    "job_plan_ref": "JP-VIB-014@9",
    "priority_code": "P3",
    "evidence_refs": ["ev-771", "ev-772"],
    "semantic_operation_id": "op_01J...",
    "expected_adapter_contract": "maximo-plant-a/7.9.2"
  },
  "possible_results": ["CONFIRMED", "REJECTED", "UNKNOWN", "BLOCKED"]
}
```

Bad tools include `write_tag`, `run_sql`, `call_opc_method`, `publish_mqtt`, `invoke_url`, `update_any_record`, or `execute_script`. A tool description must state returned fields, side effects, permission needs, error taxonomy, retry semantics, and how to reconcile an uncertain outcome.

## Separate reads, decisions, and effects

| Tool family | Examples | Control |
|---|---|---|
| Evidence read | fetch trend window, get calibration, resolve asset, read work-order state, fetch approved procedure | site-scoped, bounded query, provenance, payload limits |
| Deterministic decision | convert unit, evaluate freshness, calculate sampling plan, validate schedule constraints, check policy | versioned algorithm; model does not override |
| Draft | generate structured case, work package, CAPA hypothesis table | no system-of-record effect until reviewed |
| Business effect | create non-released work order, add approved note, request vendor case | sealed intent, policy, approval as required, operation ID, concurrency, read-back |
| Human handoff | request LOTO/permit review, quality disposition, release, regulatory review | accountable queue; no agent completion token |

Every effect tool should have a corresponding authoritative read and reconciliation operation.

## Use an adapter capability record

```yaml
adapter_id: sap-maintenance-plant-c
adapter_release: 5.8.0
site_id: plant-c
vendor_product: SAP S/4HANA Cloud
installed_release: 2608
qualified_operations:
  read_order:
    api: maintenance-order-vendor-endpoint
    concurrency: etag
    side_effect: none
  release_order:
    enabled: false
  create_draft:
    enabled: true
    idempotency: semantic-operation-index
    readback: required
authentication: workload-identity-site-c
authorization_snapshot: iam-policy-sha256:...
schema_fingerprint: sha256:...
error_map_version: sap-maint-v4
maintenance_window_ref: SAP-MW-2026-09
last_contract_test: 2026-08-29T09:00:00Z
qualification_expires: 2026-11-30T00:00:00Z
owner: plant-c-enterprise-integration
```

Capabilities are per installation, role, endpoint, configuration, and version. Do not infer them from a vendor name.

## Manifest capabilities per operation

An adapter-level `read: true` or `write: true` flag is too broad. Each semantic operation needs an immutable manifest entry:

```yaml
operation: create_work_order_draft
operation_contract: work-order.create-draft/3.1
system_class: eam_cmms
direction: effect
site_scope: [plant-a]
authority_tier: M2
input_schema_hash: sha256:...
output_schema_hash: sha256:...
identity_inputs: [site_id, asset_resolution_token, case_id]
required_source_versions: [asset_mapping, eam_case, job_plan]
preconditions: [no_conflicting_order, approval_or_pre_authorization, intent_not_expired]
side_effects: [create_work_record]
possible_transitive_effects: [workflow_notification, configured_defaulting]
idempotency: {mechanism: transaction_id_plus_local_unique_index, retention: P400D}
concurrency: {mechanism: expected_case_version, conflict_result: CONCURRENCY_CONFLICT}
consistency_window: PT2M
success_readback: {query: order_by_operation_id, cardinality: exactly_one, fields: [site, asset, case, type, state]}
unknown_reconciliation: [query_operation_id, query_guarded_business_key, human_bundle]
cancellation: {qualified_states: [DRAFT], operation: cancel_unreleased_draft}
compensation: none
forbidden_source_states: [released, in_progress, complete, closed]
credential_role: eam-plant-a-work-order-draft-writer
network_destination: eam-plant-a-approved-endpoint
vendor_endpoint_release: installed-and-qualified-value
schema_fingerprint: sha256:...
qualification_evidence: ADQ-PLANTA-EAM-2026-081
qualified_at: 2026-08-29T09:00:00Z
expires_at: 2026-11-30T00:00:00Z
```

The manifest must also state request/response size limits, pagination, timeout, retry classification, rate quota, audit behavior, privacy classification, signatures, retention, maintenance window, support status, and owner. The policy engine admits only the exact operation/manifest digest named by the behavior release.

## Build an explicit operation inventory

The examples below are capability shapes, not endorsements or claims that a named product supports them. Enable only operations proven against the plant's installed release, configuration, customization, license, role, network route, and vendor contract.

| System boundary | Candidate qualified reads | Candidate bounded effects | Permanently absent or human-only operations |
|---|---|---|---|
| MES/MOM | read production order/job/operation, equipment context, recipe/program reference, execution state, product genealogy, hold references | create an unsubmitted exception/draft only if the installation has nonauthoritative semantics | dispatch/start/stop production, change recipe/program/parameter, line clearance, final completion/release |
| CMMS/EAM | resolve asset/location/component, read meter/job plan/work request/order/task/status/actuals | create non-released request/order draft; add an approved evidence reference; qualified cancel of unchanged draft | release/start/complete/close work, attest execution, safety-plan/LOTO approval, return to service |
| QMS/LIMS/metrology | read plan/specification/method/sample/result/revision/calibration/nonconformance/CAPA/hold/release state | create draft nonconformance/CAPA/evidence note where no disposition follows automatically | record fabricated results, approve/sign, usage decision, disposition, concession, release, CAPA closure |
| Historian/OPC UA/MQTT/MTConnect | bounded historical query or subscribed evidence through a read broker with timestamps/status/sequence | none from the reasoning runtime | write/method/publish/acknowledge/shelve/inhibit/control; generic endpoint discovery from model input |
| PLC/SCADA read boundary | read a prequalified replicated tag/alarm/event projection through the evidence gateway | none | controller read/write session from the agent, control command, setpoint, force, bypass, alarm acknowledge/configuration |
| ERP/material/spares | read material master, approved alternate status, batch/serial, availability, reservation/issue/purchase/repair status | request reservation/purchase review or create a non-posting draft if independently approved | allocation decision, goods movement, inventory issue/transfer, financial posting, substitute approval |
| Document/procedure | resolve approved revision as of event/execution time; retrieve bounded signed excerpts; read withdrawal/training applicability | submit a draft change request | approve/publish/withdraw procedure, infer applicability, bypass training or qualification |
| Workflow/approval | read case/task/clock/owner/approval status and append deduplicated handoff request | create accountable task, request digest-bound approval, record acknowledged receipt | impersonate approver, treat message reaction as signature, self-complete safety/quality handoff |
| Communications | resolve approved recipient group and read delivery receipt | send templated operational notification after policy/approval with classification and expiry | emergency command, regulatory/customer notice, recall notice, confidential external transfer without accountable approval |

Split read and effect credentials even when the vendor API combines them. A missing safe operation is a reason to leave the capability disabled, not to expose a generic update endpoint.

## Qualify each connector class

| Connector | Required qualification | Special risks |
|---|---|---|
| OPC UA/edge evidence | endpoint/server identity, namespace mapping, information model, monitored items, status/timestamps, subscriptions/retransmission, limits, certificates, read-only enforcement | node drift, sequence gaps, server replacement, method/write exposure, vendor-specific status |
| MQTT/Sparkplug | broker/tenant/topic ACL, topic and payload version, retained/session/birth-death behavior, QoS, ordering, duplicate/reconnect tests | assuming QoS means exactly-once business effect, stale retained data, topic injection |
| MTConnect | agent/device identity, model version, sequence/sample semantics, asset/event/sample mapping, adapter health | tag/model change, missing context, sequence resets |
| MES/MOM/ERP | ISA-95 object mapping, job/order/operation/equipment/recipe/material semantics, concurrency, pagination, lifecycle transitions | hidden stock/cost/status effects, stale master data, deprecated APIs |
| EAM/CMMS | asset/location hierarchy, status domain, meters, job plans, safety-plan linkage, labor/parts semantics | completed vs closed, transition side effects, custom fields/workflows |
| QMS/LIMS/metrology | inspection lot/sample/result/revision, specification/method/calibration, hold/disposition/release separation, signatures/audit | accidental release or stock posting, corrected results, regulated records |
| OEM/vendor portal | asset entitlement, case status, attachments, export controls, data residency, human authorization | untrusted content, data leakage, commercial commitment, unsupported remote action |

OPC UA Machinery, Job Management, and Result Transfer can reduce bespoke semantics when products implement the relevant profiles. Still verify the server’s actual information model and certification listing; a standard reference is not proof of conformance.

## Pin dynamic vendor contracts

IBM Maximo REST schemas can be generated dynamically from an installation’s object structures and configuration. Capture the exact OpenAPI document, schema fingerprint, role, and custom automation behavior used in qualification. Maximo supports transaction IDs for duplicate detection and HTTP concurrency responses, but the adapter must prove behavior for the installed version and configured operation.

SAP APIs and status actions change across releases; older maintenance-order APIs have been deprecated in favor of successors. Pin the installed release and endpoint, use required ETags such as `If-Match`, and test transition side effects.

Microsoft Dynamics work-order scheduling can consider workers, tools, assets, competency, and capacity, but configuration can disable constraints or allow reservations to be ignored. Treat scheduler configuration as a qualified dependency, not a promise from product documentation.

These vendor examples demonstrate why installation qualification is necessary; they are not product recommendations. Cloud release labels, on-premises feature packs, custom object structures, automation scripts, user exits, workflows, permissions, licensing, regional availability, maintenance contracts, and vendor deprecation schedules can all change semantics. Record the customer system ID and installed build in protected qualification evidence; do not put confidential topology or credentials in model context.

## Normalize errors without losing evidence

```text
VALIDATION_REJECTED   input violates stable contract; do not retry unchanged
POLICY_BLOCKED        authority/precondition denied; do not bypass
CONCURRENCY_CONFLICT  source version changed; reread and replan
RATE_LIMITED          retry after bounded delay if intent remains valid
DEPENDENCY_UNAVAILABLE no acceptance evidence; outcome may be UNKNOWN
AUTHENTICATION_FAILED rotate/escalate; never downgrade identity
AUTHORIZATION_DENIED  stop; do not try a broader credential
SCHEMA_DRIFT          quarantine adapter and require requalification
CONFIRMED_REJECTED    system of record rejected effect; record reason
OUTCOME_UNKNOWN       acceptance cannot be established; reconcile before retry
```

Preserve vendor status, response headers, request/response hashes, endpoint/certificate identity, correlation IDs, and timings in the protected effect record. The model receives a bounded summary and opaque evidence references, not secrets or unrestricted raw payloads.

## Make read tools safe too

Read access can expose recipes, vulnerabilities, personal data, supplier IP, export-controlled data, or cross-site operations. Enforce:

- tenant/site and role predicates server-side;
- allowlisted fields, time ranges, object classes, and maximum rows/bytes;
- pagination cursors that cannot change scope;
- content-type validation, decompression limits, and malware scanning for attachments;
- sanitization and trust labels for manuals, notes, emails, and vendor content;
- no tool instructions embedded in retrieved data;
- source and version provenance on every excerpt;
- retention and redaction appropriate to classification.

Prompt injection is a data-control problem as well as a model problem. Retrieved text cannot grant authority or select a tool.

## Coordinate vendor and OEM workflows safely

An agent may assemble a support case, identify entitlement, attach approved diagnostic evidence, and track response. Require human approval before:

- sending confidential production data or regulated records externally;
- accepting terms, costs, replacement offers, remote access, or software/firmware recommendations;
- installing a patch, changing a parameter, or scheduling remote intervention;
- treating vendor advice as a controlled site procedure.

Scan and quarantine attachments, label vendor content untrusted, and route technical recommendations through engineering change control.

## Test the adapter, not just its mock

The qualification suite should include:

1. schema and semantic contract tests against a supported non-production instance;
2. permission tests proving excluded operations cannot be called;
3. idempotency and duplicate-delivery tests with the same semantic operation ID;
4. concurrency races using stale ETags/versions;
5. timeout at every point before and after server acceptance;
6. read-back under eventual consistency and delayed indexing;
7. pagination, high-volume, payload-limit, Unicode, unit, timestamp, and timezone edges;
8. error mapping for vendor-specific codes and HTML/proxy failures;
9. certificate rotation, token expiry, revoked access, and clock skew;
10. schema addition/removal/type change and endpoint deprecation;
11. offline buffer, reconnect storm, backlog expiry, and recovery throttling;
12. audit-field, signature, retention, export, and deletion behavior;
13. rollback to the prior adapter while inflight intents remain reconcilable.

Replay tests must use sanitized but structurally representative payloads. Mocks verify coordinator behavior; only real qualified environments expose vendor configuration and side effects.

## Produce operation-level qualification evidence

For every enabled operation, run and retain these tests with the manifest digest and real nonproduction system build:

| Test family | Required cases | Pass evidence |
|---|---|---|
| Identity and scope | same identifier at two sites, replaced component, historical as-of lookup, wrong tenant token | exact target or typed stop; server-side site predicate visible in logs |
| Authorization and route | allowed role, nearby forbidden action, expired/revoked credential, cross-site destination, direct OT route probe | only named operation succeeds; prohibited paths are technically unreachable |
| Schema and semantics | optional/missing/unknown fields, custom status/domain, corrected record, timezone/unit extremes, max payload/page | typed mapping or quarantine; no model guess or silent truncation |
| Effects and side effects | create/update against each qualified source state; configured workflow/notification/stock/cost consequences | all direct and transitive effects listed and independently read back |
| Idempotency and uncertainty | duplicates, timeout before send, timeout after acceptance, lost response, delayed index, restored local ledger | one business effect; `UNKNOWN` remains fenced until authoritative reconciliation |
| Concurrency and locks | stale ETag/version, two cases on one resource, lease expiry/failover, fencing-token mismatch | one owner wins; stale writer cannot mutate source |
| Cancellation and recovery | cancel before dispatch, during unknown, after downstream start, adapter rollback with inflight intent | source-confirmed terminal state or forward-recovery handoff; no false cancellation |
| Security and content | injection in notes/PDFs, attachment bomb/malware, secret in response, oversized/decompression payload | data stays untrusted/quarantined; secrets and unsafe content do not reach context or logs |
| Operations | rate limit, maintenance window, certificate/token rotation, clock skew, outage, backlog/reconnect, restore | bounded retries/backpressure, expiry, stable evidence, recovery within qualified capacity |
| Records and privacy | audit fields, signature behavior, export/readability, retention/legal hold, deletion/correction, worker personal data | applicable record controls pass and privacy-minimized fields remain enforceable |

Qualification fails if the expected result is merely an HTTP status. Store authoritative before/after versions, external record count, audit trail, request/response hashes, network and authorization evidence, and reviewer sign-off.

## Adapter acceptance matrix

```yaml
operation: create_work_order_draft
authority_tier: M2
sites_qualified: [plant-a]
input_schema: work-order-draft/3.1
output_schema: effect-outcome/2.0
idempotency_scope: site+operation
concurrency_guard: asset_case_version
success_definition: authoritative_readback_matches_intent
unknown_reconciliation: query_semantic_operation_then_business_key
retry_limit: 3
retry_expiry: PT10M
compensation: cancel_if_unreleased_and_unchanged
forbidden_states: [released, in_progress, technically_complete, closed]
audit_fields_complete: true
contract_tests: PASS
security_tests: PASS
load_tests: PASS
rollback_drill: PASS
```

If any field is unknown, the operation is not qualified for agent use.

## Read next

The adapter only executes a sealed intent. Continue with [Planning, effects, reliability, and post-action verification](07-planning-effects-reliability-and-post-action-verification.md).

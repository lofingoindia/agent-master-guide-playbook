# Reference Architecture, Runtime, and State

## 1. Architecture goal

The design must survive retries, partial failure, process restarts, stale metadata, lost acknowledgements, and conflicting evidence without giving the model uncontrolled authority.

The simplest reliable topology has:

1. one identity-aware API;
2. one deterministic controller backed by a transactional state store;
3. typed read adapters;
4. a model gateway with no production credentials;
5. policy and approval evaluation;
6. isolated effect workers with narrow credentials;
7. a receipt ledger and independent verifier.

Split services further only when scale or trust boundaries demand it.

## 2. Component responsibilities

| Component | Owns | Must not own |
|---|---|---|
| API | Authentication, request validation, tenancy context | Pipeline execution |
| Controller | State machine, leases, effect keys, budgets, deadlines | Unstructured action invention |
| Read adapters | Typed, bounded evidence retrieval | Mutation credentials |
| Model gateway | Prompt assembly, model call, structured-response validation | Policy or secrets |
| Policy engine | Deterministic authorization decision | Evidence collection side effects |
| Approval service | Identity-bound approval over plan hash | Plan editing |
| Effect executor | One typed operation under an envelope | Approval or outcome verification |
| Receipt ledger | Intent, acknowledgement, terminal and reconciliation evidence | Declaring data quality |
| Verifier | Contract, quality, frontier, sink and lineage checks | Repeating the effect |
| Notification adapter | Incident and audit signals | Workflow truth |

## 3. Trust-boundary architecture

~~~mermaid
flowchart TB
    subgraph Control["Control plane"]
      API[API and identity]
      CTRL[Controller]
      STATE[(State and leases)]
      POLICY[Policy and approvals]
      MODEL[Model gateway]
      API --> CTRL
      CTRL <--> STATE
      CTRL --> MODEL
      CTRL --> POLICY
    end

    subgraph Read["Evidence plane: no mutation credentials"]
      OA[Orchestrator adapter]
      CA[Catalog and lineage adapter]
      QA[Quality adapter]
      SA[Source and sink metadata adapter]
    end

    subgraph Effect["Effect plane: isolated by tenant/environment"]
      QUEUE[Authorized effect queue]
      WORKER[JIT-credential worker]
      RECEIPT[(Receipt ledger)]
      VERIFY[Independent verifier]
      QUEUE --> WORKER --> RECEIPT --> VERIFY
    end

    CTRL --> Read
    POLICY --> QUEUE
    Read --> VERIFY
    VERIFY --> CTRL
~~~

Use separate network paths, identities, and logs for the read and effect planes. A compromised prompt assembler should not be able to acquire an effect credential.

## 4. Controller state

### Operation state

The operation record persists:

- requester, tenant, environment, purpose, incident, and expiry;
- target pipeline and data-product identifiers;
- source interval or offset frontier;
- definition, runtime, transform, contract, and schema versions;
- evidence snapshot references and freshness;
- hypotheses, uncertainty, and selected plan;
- policy result, plan hash, approvals, and budgets;
- effect key, attempts, remote identifiers, and receipt;
- verification outcome, quarantine, and incident references;
- complete transition history with actor and reason.

Use optimistic concurrency or row locks so two workers cannot advance the same operation. Every transition should accept an expected state/version and append an immutable event.

### Domain identity and version set

Names are presentation fields; operations bind to canonical identities and immutable versions. The controller rejects a plan when a required identity cannot be resolved unambiguously.

| Object | Canonical identity | Version or frontier | Never substitute |
|---|---|---|---|
| Pipeline definition | Orchestrator namespace + native DAG/job/deployment ID + tenant/environment | Source commit, bundle/deployment version, compiled definition digest | Display name or `latest` |
| Logical work | Schedule/timetable ID + half-open data interval or partition-manifest hash | Schedule/timetable version and calendar/time-zone rules | Wall-clock start time |
| Physical attempt | Platform-native run/job ID + attempt ID | Adapter-observed state version or update timestamp | Logical interval alone |
| Dataset/data product | Catalog namespace + canonical resource ID + tenant/environment + product port | Table snapshot/version, warehouse table etag, or immutable manifest digest | Mutable table name alone |
| Source frontier | Source instance ID + stream/topic/table + partition/shard identity | Offset map, LSN/binlog/SCN, source snapshot ID, or inclusive/exclusive key bound | One scalar “last offset” across partitions |
| Streaming time/checkpoint | Runtime job/operator graph identity | Watermark value + strategy/version + idleness/allowed-lateness policy; checkpoint/savepoint ID, source offsets, state schema, runtime build | Processing timestamp or job state alone |
| Schema | Registry cluster + subject/name strategy + schema format | Registry version and ID plus canonical fingerprint | Contract version or field display name |
| Contract | Globally scoped contract/product ID | Semantic version, effective interval, and content digest | Registry compatibility mode |
| Transform | Package/project ID | Compiled manifest digest, code commit, dependencies, runtime image | Mutable branch name |
| Quality policy/result | Suite ID + dataset/product ID + interval/snapshot | Suite release/digest and immutable result ID | Current threshold after a run |
| Lineage | Producer namespace + job/run/dataset identifiers | Event schema/facet URLs, producer release, event ID/time, and ingestion high-watermark | Current graph only |
| Quarantine | Quarantine ID + governed storage resource + tenant/environment | Exact record/partition manifest, source/target versions, reason-policy version, object digest | Ticket number or mutable prefix |
| Backfill/replay | Operation ID + approved unit-manifest digest | Reprocessing policy, complete executable-version-set digest, base target snapshot | Date predicate or newest code |
| Deployment | Environment-qualified release ID | Controller/policy/prompt/model, state schema, adapter manifests, tool/plugin artifacts, infrastructure revision | Image tag or UI release label |
| Adapter/tool | Adapter ID + trust boundary + upstream deployment ID | Adapter build digest, upstream API/version, capability-manifest digest | Product family name |
| Effect | Tenant + environment + semantic operation + target-manifest digest | Request hash, attempt IDs, remote receipt/version | Random request UUID as the logical key |

An identity mapping is itself versioned evidence. A topic recreation with the same name, an Airflow DAG bundle change, a table replacement, or a connector pointed at a new replication slot creates a new operational identity even if a dashboard label is unchanged.

### Effect state

The effect record is separate because one operation can contain multiple ordered effects.

~~~text
INTENT_RECORDED
  -> DISPATCHED
  -> ACKNOWLEDGED
  -> TERMINAL_REMOTE
  -> RECONCILED
  -> VERIFIED
~~~

A timeout after dispatch moves to `AMBIGUOUS`, not `FAILED_RETRYABLE`. The reconciliation adapter queries the remote system by stable key or target state. Only a definitive absence permits another dispatch.

### Leases

Workers use expiring leases for controller work, but a lease is not the effect idempotency mechanism. If a worker dies after the remote effect commits, the next worker must reconcile the effect key.

## 5. Context planes

| Plane | Examples | Storage rule |
|---|---|---|
| Prompt context | Current question and selected evidence summary | Ephemeral view assembled for one model call |
| Operation context | Plan, evidence hashes, decisions, and receipts | Reconstructed from durable typed records |
| Knowledge context | Versioned runbook and reviewed incident signature | Permissioned retrieval with provenance, owner, review date, and TTL |

These are retrieval planes, not a replacement for the exactly seven memory lifetimes defined in [Security, Identity, Tenancy, Memory, and Context](07-security-identity-tenancy-memory-and-context.md#8-seven-memory-lifetimes). Do not create general conversational memory for production operations. A user's prior statement must not silently authorize a later action.

### Context compaction

When evidence exceeds the model budget, preserve:

- exact identifiers, timestamps, time zones, versions, and frontiers;
- facts with evidence references and retrieval time;
- contradictions and unresolved uncertainties;
- plan constraints, policy decisions, and receipt status;
- hashes that allow the original artifact to be re-fetched.

Drop or externalize:

- repeated log lines;
- raw row samples;
- secrets and tokens;
- verbose stack traces already classified;
- unrelated conversation;
- model chain-of-thought.

Summaries are derived evidence, not the source of truth. The verifier re-reads authoritative state.

Persist a schema-validated compaction receipt so the next worker can verify the continuity boundary:

~~~yaml
continuity_schema: dataops.compaction-receipt/v2
receipt_version: 1
operation_id: dataop_01K
operation_state_version: 19
source_event_high_watermark:
  event: {sequence: 4421, event_id: evt_4421, event_hash: sha256:...}
  sources:
    - {source_id: kafka-cluster-7/topic-id-92/partition-4, kind: kafka_offset_exclusive, value: 88392014, observed_at: 2026-08-31T12:09:08Z}
    - {source_id: postgres-orders/slot-orders, kind: postgresql_lsn_inclusive, value: 0/16B6C50, observed_at: 2026-08-31T12:09:06Z}
identity_version_set_digest: sha256:...
versions:
  pipeline_definition: git:4f6c...
  orchestrator_bundle: airflow-bundle:sha256:...
  transform_manifest: dbt-manifest:sha256:...
  runtime_image: sha256:...
  contract: odcs:orders-output:3.1.0@sha256:...
  schema: confluent:orders-value:id-912:v14@sha256:...
  adapter_capabilities: sha256:...
evidence_bundle_ids: [evidence_71]
evidence_source_high_watermarks:
  airflow-prod: page-token-or-update-watermark
  openlineage-prod: event-991204
approvals:
  - approval_id: approval_31
    plan_hash: sha256:...
    scope_hash: sha256:...
    expires_at: 2026-08-31T14:15:00Z
active_clocks:
  - {clock_id: approval-expiry, due_at: 2026-08-31T14:15:00Z, owner: operation-controller}
effects:
  pending:
    - effect_id: effect_backfill_9
      state: DISPATCHED
      effect_key: retail-eu/prod/orders/backfill/sha256:...
  unknown:
    - effect_id: effect_publish_10
      state: UNKNOWN
      last_remote_id: job_728
      required_reconciliation: warehouse.lookup_job
pending_effect_ids: [effect_backfill_9]
unknown_effect_ids: [effect_publish_10]
omitted_items: [raw_row_samples, unrestricted_logs, model_reasoning]
unresolved_contradictions: [cdc_frontier_gap]
next_safe_action:
  operation: warehouse.lookup_job
  arguments_digest: sha256:...
  preconditions: [approval_still_valid, capability_manifest_unchanged]
invariant_hash: sha256:...
context_builder_release: dataops-context/2.4.0
receipt_hash: sha256:...
~~~

Compute `invariant_hash` over the canonical tenant/environment, target and partition manifest, complete identity/version set, policy bundle, plan hash, approval scopes, effect keys, and required verification gates. On resume, compare the receipt with the event log, every evidence-source high-watermark, catalog/lineage state, orchestrator runs, contract and schema registries, approvals, and effect ledger. If an `UNKNOWN` effect exists, the only valid next effect-capable action is its named reconciliation read. Missing frontiers, source semantics, schema/contract versions, identifiers, time zones, numerical bounds, policy, evidence gaps, or ambiguous effects invalidate the receipt. Compaction changes representation, never truth, authority, or replay semantics.

## 6. Typed evidence

Do not concatenate tool output into prose. Normalize it:

~~~json
{
  "evidence_type": "orchestrator_run",
  "adapter": "airflow-prod",
  "adapter_version": "3.x-v2",
  "tenant_id": "retail-eu",
  "pipeline_ref": "orders_hourly",
  "remote_run_id": "scheduled__2026-08-31T12:00:00Z",
  "logical_interval": {
    "start": "2026-08-31T11:00:00Z",
    "end": "2026-08-31T12:00:00Z"
  },
  "state": "failed",
  "error_class": "sink_timeout",
  "observed_at": "2026-08-31T12:09:08Z",
  "fresh_until": "2026-08-31T12:10:08Z",
  "untrusted_text_refs": [
    "evidence://logs/sha256:..."
  ]
}
~~~

Unknown fields should fail closed for an effect plan. Adapter schemas are versioned and contract-tested against real or recorded platform responses.

## 7. Model interface

The model returns a constrained diagnostic object:

~~~json
{
  "classification": "ambiguous_sink_commit",
  "supported_facts": [
    {"claim": "The warehouse job reached DONE", "evidence_ref": "ev_18"}
  ],
  "uncertainties": [
    "The orchestrator timed out before recording the remote job result"
  ],
  "next_reads": [
    {"tool": "warehouse.get_job", "arguments_ref": "arg_52"}
  ],
  "proposed_plan_ref": null,
  "must_escalate": false
}
~~~

Tool name, arguments, tenant, and resource are validated against the current state and capability registry. The model cannot emit raw credentials, SQL, shell, URLs, or an approval.

Use deterministic code for:

- interval math and daylight-saving handling;
- compatibility checks;
- effect-key construction;
- policy decisions;
- budget enforcement;
- state transitions;
- receipt reconciliation;
- verification thresholds.

### Typed controller contract bundle

Do not share one permissive “agent message” schema across the control loop. Version and validate separate contracts:

| Contract | Required content | Producer | Consumer |
|---|---|---|---|
| `EvidenceRecord` | Evidence ID/type, canonical subject identity, observed and expiry times, source cursor/high-watermark, payload reference/hash, trust and classification | Read adapter | Controller, model context, verifier |
| `OperationEvent` | Event ID/sequence, prior and next state, expected aggregate version, actor, reason, timestamp, related evidence/effect IDs | Controller | Event store, projector, audit |
| `Diagnosis` | Supported claims with evidence IDs, contradictions, uncertainty, bounded next reads | Model | Controller only |
| `ActionPlan` | Exact identities/versions/frontiers, ordered steps, preconditions, budgets, cancellation and recovery, verification gates | Deterministic planner plus bounded model fields | Policy and approval |
| `ToolCall` | Registered semantic operation, schema version, canonical arguments, identity context, deadline, result-size bound | Controller | Read or effect adapter |
| `EffectEnvelope` | Plan hash, authorization, effect key, request hash, target fingerprint, budget, expiry, adapter capability digest | Policy service | Isolated executor |
| `EffectReceipt` | Intent/attempt/remote IDs, acknowledgement class, remote terminal state, before/after frontiers or snapshots, reconciliation evidence | Executor and reconciler | Ledger and verifier |
| `VerificationResult` | Intended-versus-observed manifest, contract/quality/lineage/completeness gates, mismatches, publication verdict | Independent verifier | Controller and audit |

Schema evolution for these contracts follows expand/read-old-and-new/contract rules. Unknown enum values or fields that affect identity, authority, target, scope, cost, or correctness fail closed; unknown descriptive fields may be retained as opaque data.

## 8. Adapter contract

Each effect adapter should expose the same control properties even though underlying tools differ:

~~~text
prepare(request) -> normalized target, estimated work, preconditions
lookup(effect_key) -> absent | in_progress | terminal(receipt)
execute(effect_envelope) -> acknowledgement
status(remote_id) -> normalized remote state
cancel(remote_id) -> acknowledgement, when supported
verify_hint(remote_id) -> authoritative sink identifiers
~~~

Required metadata:

- adapter and upstream API version;
- supported operations and semantic guarantees;
- idempotency or lookup mechanism;
- pagination, timeout, and rate-limit behavior;
- retry-safe error classes;
- consistency lag;
- maximum request and interval limits;
- credential and network boundary;
- compatibility tests and last successful probe.

Never infer that a feature exists because another version or deployment mode documents it. Probe during deployment and keep the capability result in a signed release manifest.

## 9. Execution semantics

### Timeouts

Use separate deadlines for:

- evidence retrieval;
- model inference;
- approval expiry;
- dispatch acknowledgement;
- remote completion;
- verification convergence.

A controller deadline should not automatically cancel a remote data job. Cancellation semantics differ and can leave partial output.

### Retries

Retry only a classified, retry-safe operation with exponential backoff, jitter, a maximum attempt count, and a total deadline. Rate limiting and overload should consume a separate retry budget from functional failures.

### Concurrency

Admission requires all applicable tokens:

~~~text
global controller slot
+ tenant slot
+ environment slot
+ pipeline slot
+ source read budget
+ sink write budget
+ cost reservation
~~~

Release or renew tokens explicitly. Concurrency products may default to permissive behavior when a named limit is missing, so adapters must test fail-closed configuration.

## 10. Deployment shape

### Initial deployment

Run the API/controller and read adapters as one stateless service, with:

- a managed relational state database;
- object storage for immutable evidence;
- a separate effect-worker deployment;
- a secrets broker or workload identity;
- one queue partition per trust boundary where needed.

This is easier to operate than many microservices and still separates effect credentials.

### Scale-out

Split only when necessary:

- evidence collectors for high-volume platforms;
- effect worker pools by tenant, environment, or connector trust;
- verifier workers for expensive data checks;
- model gateway for provider isolation and redaction;
- regional controllers for data-residency constraints.

The state store, not sticky process memory, coordinates workers.

## 11. Recovery and durability

Set RPO and RTO for distinct artifacts:

| Artifact | Loss consequence | Protection |
|---|---|---|
| Operation and effect ledger | Duplicate or untraceable effects | Transactional HA database and tested backups |
| Evidence object | Unverifiable decision | Versioned immutable storage with retention |
| Approval record | Unauthorized execution ambiguity | Signed/audited durable record |
| Stream checkpoint or savepoint | Replay, loss, or long recovery | Runtime-supported durable storage and restore tests |
| Contract and pipeline versions | Irreproducible backfill | Immutable artifact registry and source control |
| Raw source history | Impossible reconstruction | Source-specific retention and archival policy |

Test restore and reconciliation. A backup that has never been restored is only an assumption.

## 12. Selected sources

- [Airflow production deployment](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/production-deployment.html)
- [Prefect transactions](https://docs.prefect.io/v3/advanced/transactions)
- [Prefect global concurrency limits](https://docs.prefect.io/v3/concepts/global-concurrency-limits)
- [Temporal activity definition](https://docs.temporal.io/activity-definition)
- [Temporal worker versioning](https://docs.temporal.io/production-deployment/worker-deployments/worker-versioning)
- [BigQuery reliability and idempotent job IDs](https://docs.cloud.google.com/bigquery/docs/reliability-intro)
- [OpenTelemetry trace semantic conventions](https://opentelemetry.io/docs/specs/semconv/general/trace/)

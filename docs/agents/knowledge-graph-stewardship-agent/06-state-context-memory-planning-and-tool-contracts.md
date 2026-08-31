# State, Context, Memory, Planning, and Tool Contracts

The agent's durable truth lives in typed stores and ledgers. The prompt is a lossy, temporary view assembled for one bounded decision. This separation is what makes retries, approvals, audits, and recovery reliable.

## State ownership

| State | Purpose | Authority | Retention |
|---|---|---|---|
| Raw evidence store | Immutable source payloads, API responses, reports | Source observation only | Policy-defined; content-addressed |
| Assertion ledger | Versioned claims by source and validity interval | Named source | Long-lived/auditable |
| Case store | Workflow state, assignments, proposal/decision references | Deterministic orchestrator | Through retention/appeal window |
| Decision ledger | Signed decisions over exact proposal digests | Authorized principals/policy | Append-only |
| Effect ledger | Prepared attempts, receipts, ambiguity, reconciliation | Effect executor plus target observations | Append-only |
| Canonical fact projection | Effective governed view | Recomputed from assertions and decisions | Rebuildable |
| Search/vector/graph indexes | Retrieval acceleration and derived neighborhoods | Derived only | Rebuildable |
| Evaluation corpus | Curated cases, labels, slices, failure artifacts | Evaluation owners | Versioned and access-controlled |
| Prompt context | Bounded evidence view for a model call | Non-authoritative | Ephemeral |

Never allow a vector index, conversation buffer, model summary, or cached graph neighborhood to become the only copy of a fact or decision.

## Canonical record and time contract

Every durable record uses one common immutable envelope. Product-specific payloads may add fields, but they may not weaken these semantics.

```yaml
record_envelope:
  record_id: assertion-01K...          # immutable version identity
  logical_id: crm/customer/442/name    # stable identity across versions
  record_kind: source_assertion
  schema_version: stewardship-record/v3
  record_version: 4                    # monotonic within logical_id
  tenant_id: tenant-a
  domain_id: customer
  entity_type: person
  namespace: crm-eu/customer
  valid_time: {from: 2025-02-01T00:00:00Z, to: null}
  transaction_time: {from: 2026-08-31T09:01:03Z, to: null}
  source: {record_ref: crm/customer/442, revision: "19", digest: sha256:...}
  authority_scope: crm.customer-contact
  provenance_refs: [activity-77, agent-connector-4]
  supersedes: assertion-01J...
  correction_of: assertion-01J...
  status: active
  payload_digest: sha256:...
```

Time intervals are half-open `[from, to)`. Valid time means when the claim applies in the modeled domain; transaction time means when this ledger held the version as current. They are not `observed_at`, source event time, approval time, dispatch time, or target commit time. Open ends mean “no known end,” not proof of permanence. A correction appends a new transaction-time version and links the old one; it never rewrites the prior row. This is the same business-time/system-time separation implemented by bitemporal databases, but the contract does not require a particular database ([IBM Db2 bitemporal tables](https://www.ibm.com/docs/en/db2/12.1.x?topic=tables-bitemporal)).

| Record kind | Stable logical identity and version rule | Valid/transaction-time meaning | Required distinctions and terminal behavior |
|---|---|---|---|
| Source record | `(tenant, source, namespace, source key)`; every source revision/digest is immutable | Source-effective interval if supplied; ledger receipt interval separately | Raw payload, deletion/tombstone signal, visibility scope, rights, and source revision; omission is not deletion |
| Assertion | `(source-record version, subject, predicate, object/qualified value)`; corrected by superseding version | When the named source says the claim applies / when recorded | Never becomes canonical by normalization; retain null/absent/withheld/unavailable states |
| Observation | `(activity, target, method, observation time, digest)`; reruns create new observations | Observed domain interval if meaningful / receipt interval | Extractor/validator/matcher/lineage result with coverage and method build; cannot mutate its input |
| Candidate | `(case, candidate type, compared/proposed subjects, proposal generation)`; content immutable | Evidence snapshot/effective scope / creation-to-supersession | Alternatives, positive/negative/missing evidence, score policy, expiry, and status; acceptance creates a decision-derived record |
| Canonical entity | Owner-minted ID plus entity type/namespace; membership changes version it | Period in which the governed entity grouping is effective / publication history | Identity/membership only; does not contain a timeless golden row; retired IDs become aliases/superseded IDs per policy |
| Canonical fact | `(canonical entity, predicate, authority scope, valid interval)`; recomputed version cites inputs | Fact's governed effective interval / projection publication history | Carries supporting assertion/decision IDs and survivorship policy; conflict may remain unresolved instead of choosing a value |
| Alias | `(alias namespace, alias value, target relation)`; never reuse silently | Alias validity and redirect period / registry history | Distinguish exact alias, prior ID, replacement, redirect, and search synonym; split may close or reassign it |
| Relationship | `(typed endpoints, direction, relation, scope)`; edge version is immutable | Relation-effective interval / accepted or derived history | Store asserted, inferred, candidate, and decision-derived edges separately; confidence and security label live on the appropriate version |
| Schema, ontology, or vocabulary artifact | Stable artifact/ontology/scheme ID plus immutable version IRI/version/digest | Declared applicability interval / release ledger interval | Imports, contexts, shapes, mappings, reasoning/validation profiles, compatibility and deprecation; mutable `latest` is never an approval target |
| Provenance | Stable activity/entity/agent relation ID; append-only | Activity/entity time as expressed / ledger receipt | Source, activity, agent, derivation, revision, invalidation, rights, and evidence digest; named graph alone is insufficient |
| Confidence | `(producer, subject record version, method/calibration version)` | Interval/snapshot evaluated / score receipt | Distribution or score with calibration slice and limitations; never attach one mutable confidence to a real-world entity |
| Decision | `(case, proposal digest, disposition generation)`; append-only | Decision's authorized effective scope/expiry / decision receipt | Principal, authority snapshot, policy, reasons, conditions, expiry, use count; reversal/supersession is another decision |
| Merge, split, link, or unlink operation | `(proposal ID, proposal version, operation type)` | Intended identity effect interval / plan/decision/effect history | Before/after member assignment, survivorship, aliases, descendants, reversal plan, exact target preconditions, and approvals |
| Effect | `(tenant, target, semantic operation, proposal ID/version)`; attempts append under one effect | Desired target applicability / prepare-dispatch-commit-readback clocks | Idempotency key, fence, arguments digest, target version, attempt receipts, `unknown`, reconciliation, and compensation/forward repair |
| Correction or deletion instruction | `(authority case, subject scope, action generation)` | Requested correction/restriction/deletion interval / instruction history | Legal/retention basis, identity proof, recipients/descendants, holds, exceptions, per-target receipts, and outstanding unknowns |
| Release | Immutable whole-bundle digest and monotonically identified release | Activation/deactivation window / release decision and rollout history | Code, prompts, matcher, semantic bundle, policies, schemas, adapters, provider, state migration, canary cohort, and rollback bundle |

The ledger enforces `record_version` monotonicity, non-overlapping current transaction-time versions, valid interval sanity, tenant/domain immutability, and referenced-version existence. Whether valid-time intervals may overlap is record- and policy-specific: conflicting source assertions may overlap; a canonical single-valued fact generally may not. A corrected historical fact is queried as “valid at domain time *v*, known at ledger time *t*,” so auditors can distinguish late discovery from contemporary knowledge.

## Runtime state machine

The deterministic orchestrator owns transition legality:

```yaml
transition_request:
  case_id: case-01K...
  expected_case_version: 7
  from: evidence_ready
  to: proposed
  actor: agent-proposer
  proposal_digest: sha256:...
  policy_version: case-policy-v9
  command_id: cmd-01K...
```

The database transaction verifies tenant, current version, allowed transition, policy, and required fields; then appends the event and advances state. The model cannot set a case directly to `approved`, `applied`, or `completed`.

Use an outbox to publish events after the state transaction:

~~~mermaid
sequenceDiagram
    participant W as Worker
    participant DB as Case DB
    participant O as Outbox relay
    participant B as Event bus
    W->>DB: transition(expected version, command id)
    DB->>DB: append event + update case + outbox row
    DB-->>W: committed new version
    O->>DB: claim outbox row
    O->>B: publish event with event id
    B-->>O: acknowledged
    O->>DB: mark published
~~~

Consumers deduplicate by event ID and remain idempotent; “exactly once” should not be claimed across independent databases and external APIs.

## Memory policy

This is the canonical and exhaustive lifetime policy for this blueprint:

| Memory lifetime | Use or reject | Retention and deletion | Poisoning controls | Evaluation controls |
|---|---|---|---|---|
| Turn/scratch memory | Use for one bounded model/tool step; reject as authority or resume state | Destroy at call/run boundary; provider copies follow the approved endpoint contract | Evidence/instruction separation, schema validation, tool allowlist, tenant-scoped compiler | Unsupported-claim, injection, tenant-leak, and repeated-run variance tests |
| Working/run memory | Use only while constructing typed artifacts; checkpoint material progress | Bounded by run TTL; delete after checkpoint/close except restricted debug evidence under policy | Derive from manifest IDs, cap size/age, reject unreferenced content and stale versions | Crash/resume, stale-evidence, truncation, and missing-checkpoint tests |
| Session memory | Reject for facts, preferences, precedent, authority, or decisions | Disabled; clear UI/provider conversational state after the interaction window | No implicit carry-over; new request recompiles from authorized records | Start clean sessions and prove identical authoritative inputs yield equivalent bounded behavior |
| Durable workflow/task memory | Use as typed case, event, decision, approval, effect, timer, and handoff state | Retain through policy/appeal/audit window; correct by append/supersession; purge only by governed plan | Authenticated writers, invariants, optimistic concurrency, fences, signatures/digests, audit | Replay, corruption, duplicate command, failover, restore, and invariant-property tests |
| Domain knowledge memory | Use only as pinned governed schema/ontology/vocabulary/catalog/policy releases | Release retention and deprecation policy; delete/restrict source-derived content through propagation | Signed bundles, provenance/rights gate, owner approval, semantic delta and adversarial content tests | Competency, forbidden-entailment, drift, stale-release, rights, and poisoning suites |
| Long-term/preference memory | Reject; a person's habits must not influence identity or governance | Do not create; delete any accidental profile and investigate downstream use | Disable preference writes/retrieval and user-to-user transfer; policy denies profile features | Seed conflicting personas/preferences and verify zero decision or routing effect |
| Episodic/outcome memory | Use only as curated, de-identified, adjudicated evaluation/training artifacts; reject direct precedent | Versioned corpus retention, consent/rights, deletion linkage, hold policy, and contamination removal | Independent adjudication, tenant separation, label provenance, deduplication, incident quarantine | Temporal/entity-disjoint replay, bias/disagreement slices, poisoned-outcome and memorization tests |

A policy or vocabulary update enters through a governed release. It must not be learned implicitly from reviewer behavior. Raw evidence, ledgers, caches, search/vector/graph indexes, and derived projections remain governed record stores or rebuildable accelerators, not extra memory lifetimes.

## Context compiler

The compiler produces a deterministic manifest before rendering text:

```yaml
context_manifest:
  context_id: ctx-01K...
  task: propose-identity-link
  tenant_id: tenant-a
  case_id: case-01K...
  case_version: 7
  policy_versions:
    identity: identity-policy-v12
    redaction: redaction-v4
  artifacts:
    - {evidence_id: ev-1, digest: sha256:..., view: pii-tokenized}
    - {evidence_id: ev-2, digest: sha256:..., view: comparison-features}
  omitted:
    - {evidence_id: ev-9, reason: insufficient-privilege}
  token_budget:
    instructions: 1800
    evidence: 9000
    output: 2500
  compiler_build: context-compiler-3.8.0
```

Compilation order:

1. authenticate the workload identity and resolve tenant;
2. load task policy and output schema;
3. fetch only evidence IDs referenced by the current case/revision;
4. apply row, field, purpose, and classification filters before retrieval output;
5. select evidence by deterministic relevance/risk rules;
6. render source values, timestamps, and provenance separately from instructions;
7. label untrusted content and delimit it as data;
8. allocate budget and summarize only allowed artifacts;
9. persist manifest and rendered-context digest;
10. validate the model output against the manifest and task schema.

Do not let source descriptions, glossary definitions, SQL comments, or catalog text instruct the agent to call tools or ignore policy. They are untrusted evidence and may contain prompt injection.

## Compaction and evidence loss

Compaction is a lossy cache operation. It may shorten old observations for model context, but it must not mutate the evidence, case, decision, or effect ledgers.

A compaction receipt binds the replaceable summary to the authoritative state needed for deterministic resume:

```yaml
compaction_receipt:
  receipt_version: 1
  case_id: case-01K...
  source_event_high_watermark: 1442
  versions:
    case: 7
    state_schema: 23
    evidence_manifest: evm-01K...
    proposal: prop-01K.../v3
    policy_bundle: governance-18
    semantic_bundle: customer-domain-2026.09.0
    context_compiler: context-compiler-3.8.0
    model_provider: approved-provider/model-build-2026-08-20
    tool_manifest: tools-41
  approvals:
    - {decision_id: dec-1, proposal_digest: sha256:..., expires_at: 2026-09-01T03:00:00Z, remaining_uses: 1}
  clocks:
    valid_time_as_of: 2026-08-31T02:00:00Z
    ledger_recorded_through: 2026-08-31T03:00:04Z
    source_watermarks: {crm-eu: 8812, mdm-eu: 4401}
    projection_checkpoints: {search: 1438, graph: 1440}
  evidence_references:
    - {evidence_id: ev-1, digest: sha256:...}
    - {evidence_id: ev-2, digest: sha256:...}
  unresolved_contradictions: [identifier-reuse-policy-not-confirmed]
  pending_effects: [eff-01K...]
  unknown_effects: []
  omitted_item_references:
    - {artifact_id: ev-9, digest: sha256:..., reason: insufficient-privilege, reload_rule: privacy-owner-only}
  invariant_hash: sha256:...
  next_safe_action:
    action: reconcile_effect
    target_ref: eff-01K...
    preconditions: [reauthorize, verify_case_v7, verify_target_version]
  context_manifest_id: ctx-01K...
  narrative_summary_ref: summary://sum-01K...
  receipt_digest: sha256:...
```

Reject receipts or summaries with unsupported claims, missing evidence IDs, expired/changed approvals, regressed event watermarks, incompatible versions, clock inversion, or omitted pending/unknown effects. Pin decisions, approvals, target preconditions, policy clauses, dissenting evidence, and unresolved contradictions in authoritative stores.

On restart, worker migration, context loss, or model-provider switch:

1. verify the receipt digest and supported receipt/state schema;
2. reauthenticate and reauthorize instead of reusing a prior token or authority result;
3. reload authoritative events through the high watermark, then consume later events;
4. verify case/proposal/policy/semantic/tool/provider versions, approvals, clocks, and invariant hash;
5. reconcile unknown effects before dispatching any mutation and validate pending-effect fences;
6. fetch omitted items only if the new principal is authorized, otherwise retain the omission explicitly;
7. compile a fresh context and revalidate the next safe action against current state.

A provider switch is a behavior-bundle change, not a continuation optimization. If the new provider/model is not already qualified for the task and data class, pause or fall back to deterministic/human processing. Never continue from the narrative summary alone.

## Planning policy

Use a bounded workflow, not an open-ended autonomous loop.

| Planning activity | Owner | Bound |
|---|---|---|
| Select case transition | State machine | Fixed transition table |
| Decide required evidence classes | Policy | Case-type manifest |
| Select exact adapter/tool | Deterministic router | Capability and tenant allowlist |
| Propose evidence-query subplan | Model allowed | Read-only; max calls/depth/cost; schema validated |
| Decide identity/semantic outcome | Model proposes; human/policy decides | Calibrated thresholds and authority |
| Execute external mutation | Effect executor | Approved effect contract only |
| Recovery | Runbook/orchestrator | Typed reconciliation/compensation path |

The model can answer “which of these permitted evidence queries would reduce uncertainty?” It cannot invent a new source connector, broaden tenant scope, or turn a read plan into a mutation plan.

Planning limits include maximum elapsed time, tool calls, result bytes, graph expansion nodes/edges, lineage depth, retries, and money/token cost. Exhaustion yields a typed `insufficient_evidence` or `budget_exhausted` result, not an improvised shortcut.

## Tool contract envelope

Every tool declares input/output schemas, scopes, side-effect class, idempotency behavior, timeouts, and versioned capabilities.

```yaml
tool_contract:
  name: catalog.get_asset
  version: 3.2.0
  side_effect: none
  auth:
    required_scopes: [catalog.asset.read]
    tenant_bound: true
  input_schema: urn:schema:catalog.get_asset.input:3
  output_schema: urn:schema:catalog.get_asset.output:3
  limits:
    timeout_ms: 5000
    max_response_bytes: 1048576
  provenance:
    returns_source_revision: true
    returns_visibility_scope: true
  errors: [not_found, forbidden, rate_limited, partial, stale_cursor, unavailable]
```

Mutation tools are never exposed directly to the proposer. The executor receives a prepared effect:

```yaml
effect_command:
  effect_id: eff-01K...
  approved_proposal_digest: sha256:...
  approval_ids: [dec-1, dec-2]
  tenant_id: tenant-a
  action: atlas.apply-classification
  target: urn:asset:hive:prod:orders.email
  target_precondition: {entity_version: 77}
  arguments_digest: sha256:...
  idempotency_key: sha256:...
  expires_at: 2026-08-31T13:30:00Z
```

The executor resolves arguments from immutable prepared storage, reauthorizes, verifies approval and precondition, then calls a target-specific adapter. It rejects arbitrary arguments supplied by model text.

## Tool-result normalization

Normalize transport behavior without erasing source semantics:

```yaml
tool_result:
  invocation_id: inv-01K...
  status: partial
  tool: lineage.get_upstream
  tool_version: 2.4.0
  target_version: openmetadata-1.12.8
  data_ref: evidence://.../result
  evidence_digest: sha256:...
  source_revision: null
  completeness:
    scope: visible-assets-only
    truncated: true
    next_cursor: opaque:...
  observed_at: 2026-08-31T10:00:00Z
  error:
    code: result_limit
    retryable: true
```

The agent must distinguish `not_found`, `forbidden`, `empty`, `partial`, `timeout`, and `unknown-after-write`. Collapsing them into null creates destructive reconciliation bugs.

## Operation-level adapter qualification

Qualify each operation, not a product logo. Read and write endpoints in the same product can have different consistency, authorization, atomicity, and receipt behavior.

```yaml
operation_capability:
  manifest_version: 2
  adapter_build: neo4j-adapter-4.1.0
  target: {product: neo4j, build: 2026.07.1, edition: enterprise, api: bolt/cypher-25}
  operation: graph.apply-approved-membership-delta
  semantics:
    side_effect: graph_write
    atomic_scope: one_database_transaction
    isolation: read_committed
    conditional_write: application_version_property
    remote_idempotency: none
    local_effect_key: required
    receipt: transaction_metadata_plus_readback
    pagination_consistency: not_applicable
    visibility: principal_filtered
    cancel_after_dispatch: not_guaranteed
    delete_modes: [detach_delete]
  limits: {timeout_ms: 5000, max_nodes: 1000, max_edges: 5000}
  qualification_run: qual-neo4j-2026-08-22
  qualified_until: 2026-11-22
```

The manifest also pins authentication method/scopes, tenant enforcement location, request/response schemas, retryable and terminal errors, rate limits, consistency window, batch item behavior, audit lookup, backup/restore behavior, and links to the exact target documentation. Unknown or untested means unsupported.

### Required technology boundaries

| Adapter family | Operations to qualify independently | Provider-specific limits that the contract must expose |
|---|---|---|
| RDF/SPARQL store | dataset/graph read, bounded query, SHACL input export, staging-graph write, graph promotion, delete, snapshot/export, readback | SPARQL 1.1/1.2 syntax and entailment; graph-name convention; query/update dataset mapping; atomic scope; isolation; service/federation behavior; version precondition and idempotency are not portable standards |
| Neo4j/property graph | key lookup, bounded traversal, transactional delta, constraint/type change, classification/property update, CDC/checkpoint, snapshot/restore, vector/full-text retrieval | Target build/edition; default read-committed behavior; uniqueness/key constraints; explicit application version/fence; deadlock/retry; CDC enablement/retention; search result authorization and partial-result behavior |
| DataHub | URN/aspect read, version/timeseries read, proposal emit, synchronous/asynchronous aspect write, stateful-ingestion stale handling, lineage read/write, search, restore/reindex | Core versus Cloud/CLI/API versions; aspect schema; Cloud 2.1 orchestration-plugin async emit documents no read-after-write guarantee; pipeline/state-provider identity; authorization/search visibility; per-aspect result and readback ([release note](https://github.com/datahub-project/datahub/blob/master/docs/managed-datahub/release-notes/v_2_1_0.md)) |
| OpenMetadata | ID/FQN read, entity version read, JSON Patch, create/update, lineage add/delete/read, soft/hard delete/restore, search, glossary/governance action | API/ingestion build; entity version and patch conflict behavior; pagination/result limits; permission-filtered results; parser/entity-resolution coverage; per-operation delete/restore semantics |
| Apache Atlas | GUID/unique-attribute read, entity/type mutation, relationship, classification local/propagating mutation, lineage/search, delete/purge/audit | `propagate`, removal-on-delete, relationship propagation direction, validity periods, partial mutation response, deleted-versus-purged status, audit lookup, no generic `set_tag` abstraction |
| OpenLineage | validate/accept event, run/job/dataset identity, facet upsert/delete, schema URL resolution, lifecycle observation, consumer checkpoint | Spec/client/producer version; immutable facet schema URL; backend acceptance/dedup/order/retention; event presence is not lineage completeness or canonical authority |
| Schema registry | context/subject/config read, compatibility test against explicit versions, lookup, register, references, mode/config update, soft/hard delete | Schema ID versus subject version; normalization; compatibility group/mode; `latest` race; identical registration reuse; soft-before-hard delete; authorization and primary forwarding errors |
| Search/vector service | authorized exact/lexical/vector retrieval, filter, index version read, delete-by-subject, rebuild, checkpoint/cutover | Model/dimension/metric/normalization; approximate-recall and top-k limits; ACL filtering; stale/update visibility; score non-portability; delete and rebuild proof; never a canonical write path |
| Queue/event bus | publish, consume, deduplicate, ordering, delay/dead-letter, replay, transactional outbox handoff | Ordering scope, retention, duplicate/conflicting duplicate behavior, producer idempotence/fence, consumer offset semantics, payload limits; broker exactly-once never extends automatically to external targets |
| Durable workflow | start/deduplicate, timer, signal/approval wait, activity retry, cancellation, worker version routing, replay/export | Workflow ID/reuse, history limits, deterministic replay rules, timeout/retry boundaries, cancellation delivery, activity idempotency, search retention, upgrade compatibility; journal is not target transaction |
| Downstream consumer | invalidate, correction/delete notification, acknowledge, rebuild/replay, version support, health/readback | Consumer owner, accepted schema versions, dedup key, minimum watermark, security label, terminal acknowledgements, deadline, hold/exception behavior, and sampled state verification |

SPARQL Update says a request **should** be treated atomically, leaves concurrency to implementations, and notes that federated `SERVICE` use commonly loses atomicity; the adapter must test the deployed store rather than upgrading that wording into a guarantee ([SPARQL 1.2 Update Working Draft](https://www.w3.org/TR/sparql12-update/)). Neo4j documents ACID transactions but default read-committed isolation and possible non-repeatable reads; it also documents that `MERGE` alone does not provide uniqueness under concurrent loads without an appropriate constraint ([transactional behavior](https://neo4j.com/docs/operations-manual/current/database-internals/), [Cypher `MERGE`](https://neo4j.com/docs/cypher-manual/current/clauses/merge/)).

HTTP method names are not enough either. RFC 9110 defines PUT and DELETE as idempotent in intended effect and `If-Match` as a lost-update guard, but an action-shaped POST, target propagation, audit append, or downstream notification can still differ. Record observed target semantics and use conditional requests where supported ([RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html)).

### Qualification suite

Run the suite against the exact build, edition, configuration, identity, topology, and realistic data shape. Retain request/response digests and target observations.

1. **Identity:** distinguish not-found, forbidden, filtered, deleted, duplicate unique key, ambiguous name, and stale alias.
2. **Consistency:** test read-your-write, pagination under concurrent mutation, follower/index lag, snapshot/cursor expiry, and visibility changes.
3. **Concurrency:** race two writers with the same and different expected versions; prove lost updates are rejected and stale fences fail.
4. **Idempotency:** repeat before response, after response loss, after restart, and after retention expiry; document server versus local dedup scope.
5. **Ambiguity:** inject disconnect/timeout before send, during body, after remote commit, and before response; prove `unknown` and readback behavior.
6. **Batching:** mix valid, forbidden, conflicting, duplicate, and oversized items; require per-item receipts or reject the batch capability.
7. **Delete/recovery:** exercise tombstone, soft delete, restore, hard delete/purge, alias/reference consequences, backup/CDC/search/vector remnants, and legal hold.
8. **Security:** test row/edge/field/tenant filters, search and vector leakage, query injection, propagation, audit access, and credential revocation mid-run.
9. **Limits:** measure rows/bytes/depth/degree, transaction memory, rate limits, retry headers, hot partitions, and cancellation after dispatch.
10. **Upgrade/restore:** run old/new client matrices, schema changes, CDC/checkpoint continuity, backup restore, and post-restore target reconciliation.

Fail closed when a required behavior is unobservable. For example, Confluent documents that `latest` may change immediately after a compatibility check and distinguishes soft from permanent deletion; bind release evidence to explicit versions and re-read after registration/deletion ([Schema Registry API](https://docs.confluent.io/platform/current/schema-registry/develop/api.html)). Atlas exposes propagation direction and propagated-classification audit actions, so local and propagating classification must remain different effect types ([Atlas v2 API](https://atlas.apache.org/api/v2/index.html)). Neo4j documents that semantic-index authorization can yield zero or partial results and that raw scores from full-text/vector searches should not be compared as one scale; record truncation/filtering and use search only for candidate retrieval ([semantic-index limitations](https://neo4j.com/docs/operations-manual/current/authentication-authorization/limitations/), [semantic indexes](https://neo4j.com/docs/cypher-manual/current/indexes/semantic-indexes/)).

## Idempotency and concurrency

Use three independent mechanisms:

- **Command deduplication:** the same `command_id` cannot advance a case twice.
- **Optimistic concurrency:** expected case/subject/target versions prevent stale writes.
- **Effect idempotency and reconciliation:** repeated execution is safe only if the target contract supports it or readback proves the first attempt did not apply.

The effect state is monotonic:

```text
prepared -> executing -> applied -> reconciled
                     \-> not_applied
                     \-> unknown -> applied | not_applied | manual_recovery
```

Never convert `unknown` to `failed` merely because a client timeout elapsed.

## Long waits, leases, fencing, cancellation, and handoff

Approval waits, rate-limit deferrals, legal holds, and reconciliation windows commonly outlive a worker process. Persist a timer/wait record with wake condition, deadline, owning case version, policy version, and escalation route. A queue visibility timeout or process sleep is not durable waiting.

```yaml
work_claim:
  case_id: case-01K...
  step_id: reconcile-eff-01K
  run_generation: 12
  lease_id: lease-88
  fencing_token: 2041
  owner: reconciler-pool/worker-7
  acquired_at: 2026-08-31T10:00:00Z
  expires_at: 2026-08-31T10:05:00Z
  expected_case_version: 9
```

The store increments the fencing token whenever ownership changes. Every state mutation and external effect command carries the current token plus expected aggregate/target version. A late worker with an expired lease may finish a read, but its checkpoint or effect is rejected. Lease expiry permits takeover; it does not prove the old worker stopped, so fencing is mandatory for mutation paths.

Cancellation is a durable request with scope and reason, not a thread interrupt:

- stop scheduling new reads/model calls/effects and revoke/expire unconsumed approvals where policy requires;
- allow bounded cleanup and checkpoint already obtained evidence;
- reconcile any dispatched effect before declaring cancellation complete;
- distinguish `cancel_requested`, `cancelled_before_dispatch`, `cancelled_with_committed_effect`, and `cancellation_unknown`;
- use a new compensating or forward-correction proposal for a committed effect; cancellation does not undo reality.

A handoff record transfers the current case version, run generation, receipt digest, reason, required skill/authority, pending timers, and next safe action. The receiving worker or human reauthorizes, takes a new fenced claim, rehydrates authoritative state, and acknowledges ownership. Chat text may explain the handoff but cannot be the handoff state.

## Replay and rebuild

Replay reconstructs workflow state by applying versioned events in order and verifying event ID, aggregate version, schema/upcaster version, and invariant hash. Replay code must not call models, clocks, random generators, or external tools; recorded results are inputs. A behavior change that would reinterpret history requires an explicit migration/projection version, not silent replay drift.

Rebuild derived canonical/search/vector/graph views from a pinned assertion/decision event watermark and a pinned projection build. Record input partitions, start/end watermarks, skipped/failed partitions, output counts/digests, and cutover checkpoint. Run live events into a delta stream during rebuild, catch up to a cutover watermark, compare old/new projections, then switch atomically where the serving system supports it. Failed rebuilds leave the previous confirmed projection active with staleness metadata.

After disaster restore, fence all effect executors and compare restored ledger watermarks with external targets. Targets may be ahead of restored state; reconcile before replaying any effect. Queue or workflow durability cannot create exactly-once semantics across an external graph/catalog API.

## Events

Publish small typed events containing identifiers, versions, digests, and routing metadata—not raw restricted records.

```yaml
event:
  event_id: evt-01K...
  type: stewardship.proposal.created.v1
  tenant_id: tenant-a
  aggregate_id: case-01K...
  aggregate_version: 8
  occurred_at: 2026-08-31T10:01:00Z
  actor: agent-proposer
  payload:
    proposal_id: prop-01K...
    proposal_digest: sha256:...
    risk_tier: high
  trace_id: trace-01K...
```

Version event schemas additively where possible. Maintain consumer compatibility tests and a replay plan. Consumers fetch authorized detail by reference.

## Model output contract

```json
{
  "disposition": "propose_link | propose_non_link | request_evidence | abstain",
  "evidence_ids": ["ev-1", "ev-2"],
  "contradicting_evidence_ids": ["ev-6"],
  "reason_codes": ["VERIFIED_IDENTIFIER_MATCH", "NAME_CONFLICT"],
  "uncertainties": ["IDENTIFIER_REUSE_POLICY_UNKNOWN"],
  "suggested_queries": [],
  "narrative": "Short reviewer-facing explanation"
}
```

Validate enum values, identifiers against the context manifest, maximum lengths, and task-specific invariants. Drop unsupported narrative claims; do not repair them silently.

## Failure recovery

| Failure | Durable checkpoint | Resume rule |
|---|---|---|
| Worker crash during evidence fetch | Case plus completed observation refs | Reissue safe reads; deduplicate observations by source revision/digest |
| Context generation fails | Manifest build attempt | Fix access/size issue; no proposal exists |
| Model timeout | Invocation attempt | Retry within model budget; same manifest; record new attempt |
| Invalid model output | Raw restricted invocation log and validation errors | Retry/fallback/abstain; never infer approval |
| Case version conflict | Rejected transition | Reload latest case and recompute proposal |
| Event publish failure | Transactional outbox row | Relay retries |
| External effect timeout | Effect in `unknown` | Reconcile target before retry |
| Index corruption | Canonical ledgers and checkpoint | Rebuild derived index |

## Checklist

- [ ] Prompts and indexes are explicitly non-authoritative.
- [ ] Durable record kinds implement the common bitemporal identity/version/correction envelope.
- [ ] The canonical memory table covers use/reject, retention/deletion, poisoning, and evaluation for all seven lifetimes.
- [ ] Context manifests bind exact evidence, policy, visibility, budget, and compiler version.
- [ ] Compaction receipts preserve versions, approvals, clocks, citations, omissions, contradictions, effects, invariants, and next safe action.
- [ ] Restart/provider switch reauthorizes, verifies, reconciles, and rehydrates authoritative state.
- [ ] The deterministic workflow owns transitions; model planning is read-only and bounded.
- [ ] Tools expose side-effect, scope, error, capability, and provenance contracts.
- [ ] Every enabled operation passed build/edition/configuration-specific concurrency, ambiguity, delete, security, and restore qualification.
- [ ] Long waits, cancellation, handoffs, leases/fences, replay, and rebuild are tested.
- [ ] Proposer and mutation executor are separate trust domains.
- [ ] Commands, state changes, events, and effects each have correct deduplication semantics.

## Related guidance

- [Durable execution](../../runtime/durable-execution.md)
- [Agent state and event contracts](../../runtime/agent-state-and-event-contracts.md)
- [Compaction and continuity](../../context-memory/compaction-and-continuity.md)
- [Tool contracts](../../tools/tool-contracts.md)

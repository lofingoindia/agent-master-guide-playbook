# State, Events, Effects, Memory, and Context

## Authoritative record model

The system must distinguish what happened in the business domain from what the workflow attempted and what observability reported:

~~~mermaid
flowchart LR
    CMD[Command] --> ST[Authoritative state transition]
    ST --> DE[Domain event]
    ST --> EI[Effect intent]
    EI --> DR[Delivery attempt and receipt]
    DR --> RC[Reconciliation]
    ST --> TM[Telemetry]
    DE --> EV[Evaluation and audit views]
    RC --> ST
~~~

| Record | Example | Authoritative use |
|---|---|---|
| Command | `EvaluateWatch`, `AcknowledgeCase`, `RecordDisposition` | Requested transition with actor and preconditions |
| Domain event | `ObservationAccepted`, `CaseAcknowledged` | Immutable statement of accepted domain change |
| Effect intent/ledger | “Create ITSM task for case v6” | Prevents duplicate external effect and anchors reconciliation |
| Delivery event | Attempt, response, remote ID, unknown outcome | Records transport behavior, not business success |
| Telemetry | Span, log, counter, histogram | Debugging and SLO measurement; never replay truth |
| Artifact | Query, evidence manifest, detector output, decision packet | Content-addressed evidence referenced by state |

Do not reconstruct state from logs, notification history, or model transcripts.

## Core entities

### Watch

Immutable versions plus an active status pointer. Contains contract references, evaluation policy, detector, route, decision table, rights, and lifecycle metadata.

### Evaluation

One attempt to observe an exact watch version and interval. It records trigger origin, schedule slot, attempt, budgets, semantic request, final observation, and terminal reason.

### Observation

Immutable semantic query result plus time, quality, lineage, rights, and revision evidence. Observations can supersede but never overwrite one another.

### Signal

Reproducible detector output for an observation and baseline. It records every rule result even when no trigger occurs.

### Case

Durable alert-to-outcome state. A case may group signals while retaining their identities. `case_version` increments on material evidence, decision, or effect changes.

### Approval and decision

Authenticated statements bound to a case version, evidence manifest, policy, target, and expiry.

### Effect

Intent, attempts, remote receipt, and reconciliation status for one semantic operation.

### Outcome

Verification of an operator disposition, delegated action, and realized result. It never rewrites the original observation.

## Identity and concurrency keys

Use stable keys:

| Concern | Key |
|---|---|
| Scheduled evaluation | `tenant/watch_id/watch_version/interval_start/interval_end` |
| Manual re-evaluation | Above plus explicit revision request ID |
| Observation | Evaluation key plus semantic snapshot and revision |
| Signal | Observation ID plus detector and baseline versions |
| Case deduplication | Owner-approved watch/correlation window/material identity |
| Notification | `case_id/case_version/route_version/destination/purpose` |
| Escalation | `case_id/escalation_step/case_version` |
| Work item | `case_id/case_version/action_class/target_system` |
| Acknowledgement | `case_id/case_version/subject` |
| Outcome check | `case_id/outcome_definition/check_interval` |

The semantic identity—not a random retry ID—drives idempotency.

Use optimistic concurrency on watch and case versions. A command specifies `expected_version`; a conflict forces the caller to reload and re-evaluate. Do not let last-write-wins overwrite acknowledgement or revision state.

### Version semantics by domain object

Names are locators, not immutable meaning. Store an immutable version/digest and an effective interval for every object that can change a result or authority decision:

| Object | Immutable identity must bind | New version required when | Never infer from |
|---|---|---|---|
| Metric | Metric definition, semantic model/manifest digest, unit, grain, timezone, currency/reference-data versions, null and aggregation policy | Formula, join path, allowed group-by, calendar, unit, quality contract, or ownership changes | Dashboard title or SQL text alone |
| Watch | Metric version, dimensions/filters, schedule/event trigger, data gate, detector/baseline, materiality, persistence/recovery, routes, timers and authority | Any field that can change evaluation, human load, route, retention, or permission changes | Mutable watch name |
| Evaluation | Watch version, exact event-time interval, trigger/reprocessing identity, rights decision and requested semantic snapshot | Never mutate; a manual retry resumes it, while intentional re-evaluation receives a revision request ID | Queue delivery or process attempt ID |
| Observation | Evaluation identity, resolved semantic snapshot, source watermark, query/result/schema digests, gate decision and revision | Backfill, correction, late completeness, or semantic change yields a superseding observation | Request time or latest table contents |
| Baseline | Training/fit interval, eligible observation IDs, exclusion windows, feature transform, seasonality/regime, algorithm/config and artifact digest | Fit set, transform, regime, algorithm, parameter, or artifact changes | “Last 30 days” at runtime |
| Detector | Executable/config digest, score meaning, threshold/materiality/persistence/recovery policy and compatible input schema | Logic, library, threshold, calibration, compatibility, or decision meaning changes | Package name or model alias |
| Signal | Observation, detector and baseline versions plus score/evidence digest | Never mutate; correction creates a superseding signal | Case status |
| Decision | Decision-table/policy version, case/evidence version, actor/role, input digest, matched rule, disposition and effective time | New evidence, case revision, expired approval, rule change, or actor command creates a new decision record | Current ticket state or model prose |
| Outcome | Outcome-definition version, eligibility, source/evidence, measurement and correction windows, observed-at time, result and attribution status | Source correction or later window creates a superseding observation | Ticket closure, acknowledgement, or post-alert metric movement |
| Episode | Case, decisions, delegated actions and the complete ordered outcome-observation set plus review state | Review, privacy classification, labels, or outcome evidence changes; retain prior lineage | Nearest similar narrative |

Version compatibility is explicit. For example, a watch can require `metric net-revenue@12`, `baseline weekday-region@9`, and `detector material-drop@5`. If the semantic gateway resolves `net-revenue@13`, the evaluation stops or enters an approved shadow migration; it does not silently use “latest.”

## Example case record

~~~yaml
case_id: case_01K...
tenant_id: commerce
case_version: 6
state: awaiting_decision
root_case_id: null
watch:
  id: revenue-drop-emea
  version: 17
signals: [sig_01K..., sig_01K...]
evidence_manifest_ref: artifact://case-evidence/a73...
facts_digest: sha256:499...
triage:
  status: complete
  model_release: model://triage-small/2026-08-20
  prompt_version: triage/12
  context_manifest_ref: artifact://context/0a8...
decision:
  table_ref: decision://revenue-watch/6
  required_role: role:commercial-duty-manager
  disposition: null
timers:
  acknowledgement_due: 2026-08-29T09:10:00Z
  disposition_due: 2026-08-30T07:10:00Z
effects:
  - effect_01K...
revision:
  supersedes_case_version: 5
  reason: second_persistent_signal
authority:
  policy_decision_id: pd_203...
  allowed_next_commands: [record_disposition, request_analysis, assign_action]
integrity:
  state_schema: case/1.0
  updated_at: 2026-08-29T07:10:11Z
~~~

## Event envelope

Use a stable application-owned event schema. CloudEvents can carry it across transports, but the broker envelope does not define domain correctness.

~~~json
{
  "specversion": "1.0",
  "id": "evt_01K...",
  "source": "bi-monitoring/controller",
  "type": "com.example.bi.case.acknowledged.v1",
  "subject": "tenant/commerce/case/case_01K...",
  "time": "2026-08-29T08:02:19Z",
  "datacontenttype": "application/json",
  "data": {
    "case_id": "case_01K...",
    "previous_version": 6,
    "new_version": 7,
    "actor": "user:operator-31",
    "role": "role:commercial-duty-manager",
    "evidence_manifest_hash": "sha256:a73...",
    "command_id": "cmd_01K...",
    "trace_id": "4f..."
  }
}
~~~

Required envelope fields include tenant, schema version, event ID, aggregate ID/version, command/causation/correlation IDs, actor, policy decision, occurred time, and integrity hash where needed.

## Effect ledger

~~~yaml
effect_id: effect_01K...
operation_key: commerce/case_01K/v6/itsm/create-investigation
case_id: case_01K...
case_version: 6
effect_type: create_work_item
target:
  adapter: itsm
  tenant_scope: commerce
  queue: EMEA-COMMERCIAL
payload_digest: sha256:bc3...
approval_ref: approval_88...
policy_decision_id: pd_203...
state: intent_recorded     # intent_recorded | attempting | confirmed | failed | unknown
attempts: 0
remote_id: null
receipt_ref: null
next_reconcile_at: null
created_at: 2026-08-29T07:11:00Z
~~~

The ledger stores no secret. A worker obtains a narrowly scoped credential only after rechecking effect eligibility and approval freshness.

## State-machine requirements

- Impossible transitions fail closed and produce an audit event.
- Every state change has a command, actor, expected version, policy decision, and resulting event.
- Timer delivery is at-least-once; handlers are idempotent and recheck current state.
- A cancelled case can still receive a late remote effect; reconciliation records and remediates it.
- A revised observation may invalidate triage, approval, pending delivery, and outcome windows.
- State history is append-only or reconstructable from authoritative events plus snapshots.
- Snapshots are optimization; their schema version and event frontier are verified on load.
- Administrative repair uses a typed, reviewed command that records before/after state and reason.

## Memory architecture

Memory is not one store. The seven lifecycle classes are turn/scratch, working/run, session, durable workflow/task, domain knowledge, long-term/preference, and episodic/outcome. Enable only the classes this workload needs; raw transcripts are listed separately because they are audit material, not a memory lifetime:

| Memory class | Default | Contents | Authority and retention |
|---|---|---|---|
| Turn/scratch memory | Enabled | Current compiled evidence and instructions | Non-authoritative; discarded after model call |
| Working/run memory | Enabled, bounded | Hypotheses, completed reads, remaining budget | Run-scoped; hypotheses marked unverified |
| Session memory | Disabled | Free-form user conversation history | Scheduled monitoring has no useful chat session continuity |
| Durable workflow/task memory | Enabled | Watches, evaluations, cases, approvals, timers, effects | Authoritative store with explicit schema and retention |
| Domain knowledge memory | Enabled by reference | Semantic layer, runbooks, ownership, calendars, decision tables | Governed external sources with owners and versions |
| Long-term/preference memory | Disabled | Implicit preferences or personal profiles | Routing comes from policy/directories, not learned preference |
| Episodic/outcome memory | Enabled after verification | Structured prior cases, decisions, actions, and outcomes | Provenance-bearing; offline evaluation/retrieval only |
| Raw transcript memory | Disabled | Model messages and tool dumps | Keep only minimum audit artifacts under policy |

### Outcome episode

~~~yaml
episode_id: ep_01K...
case_id: case_01K...
watch_id: revenue-drop-emea
watch_version: 17
evidence_manifest_hash: sha256:a73...
disposition: action_assigned
action_class: commercial-investigation
action_receipt_ref: itsm://INC00182
outcomes:
  - definition_ref: outcome://revenue-recovery/3
    observed_at: 2026-09-12T07:00:00Z
    result: not_achieved
    evidence_ref: artifact://outcome/77c...
attribution: not_established
operator_labels:
  actionable: true
  routing_correct: true
  detector_label: true_positive
retention_until: 2027-09-12
provenance_hash: sha256:344...
~~~

Episodes are candidates for retrospective learning. They do not become prompt examples until reviewed for correctness, privacy, temporal relevance, and survivorship bias.

## Context compilation

Build context fresh for each model decision from authoritative records:

~~~mermaid
flowchart TB
    A[Authority and policy] --> CC[Context compiler]
    T[Task and output schema] --> CC
    S[Verified case state] --> CC
    E[Fresh evidence manifest] --> CC
    H[Selected verified episodes] --> CC
    U[Untrusted comments and labels] --> Z[Quarantine and annotate]
    Z --> CC
    CC --> P[Prompt package]
    P --> M[Model]
    M --> V[Structural and evidence validator]
    V --> C[Controller command proposal]
~~~

Recommended order:

1. authority, prohibited actions, tenant, purpose, data-class rules;
2. exact task, decision point, output schema, and stop conditions;
3. current case/watch/semantic/detector versions and remaining budgets;
4. verified facts and evidence references;
5. compact recent interaction or working notes;
6. optional reviewed episodes or runbook passages;
7. untrusted content clearly delimited as data.

Do not paste full dashboards, tickets, query results, or entire prior cases. Retrieve the minimum fields that can change the current decision.

### Context manifest

~~~yaml
context_manifest_id: ctx_01K...
case_id: case_01K...
case_version: 6
purpose: triage_route_proposal
authority:
  tenant_id: commerce
  policy_decision_id: pd_203...
  allowed_tools: [semantic_breakdown, related_cases, known_events]
  prohibited: [source_write, raw_rows, recipient_selection]
items:
  - ref: artifact://detector/519
    trust: verified
    retrieved_at: 2026-08-29T07:10:05Z
    content_hash: sha256:192...
  - ref: itsm://change/CHG-4801
    trust: untrusted_external_text
    retrieved_at: 2026-08-29T07:10:06Z
    content_hash: sha256:882...
budgets:
  tool_calls_remaining: 3
  model_turns_remaining: 2
  token_limit: 12000
compiler_version: context/8
manifest_hash: sha256:0a8...
~~~

The manifest makes a model decision replayable enough to audit, while acknowledging provider and model nondeterminism.

## Compaction and continuity

Scheduled watch evaluation should usually complete without conversational continuity. Compaction applies only to long triage or decision cases.

Compaction is lossy. Before pruning history, commit a checkpoint:

~~~yaml
compaction_receipt:
  receipt_version: 2
  receipt_id: checkpoint_01K...
  case_id: case_01K...
  case_version: 9
  source_event_range: {after: 183771, through: 183920}
  source_event_digest: sha256:5a1...
  prior_receipt: {id: checkpoint_01J..., hash: sha256:102...}
  context_compiler_release: bi-context/8
  compactor_release: bi-compactor/3
  objective: determine route and obtain accountable disposition
  authority:
    policy_decision_ref: pd_203...
    decision_table_ref: decision://revenue-watch/6
    allowed_next_commands: [record_disposition, request_analysis]
    prohibited: [source_write, threshold_change, recipient_change]
  pinned_versions:
    watch: revenue-drop-emea/17
    semantic_snapshot: sha256:7e9...
    detector: material-drop/5
    baseline: weekday-region/9
    behavior_bundle: bi-monitoring/2026.08.31.2
  verified_fact_refs: [fact_1, fact_2]
  unresolved_contradictions: [contradiction_2]
  hypotheses_not_facts: [hyp_3]
  completed_reads:
    - {id: read_1, receipt_digest: sha256:02a...}
    - {id: read_2, receipt_digest: sha256:18b...}
  remaining_budgets: {tool_calls: 1, model_turns: 1}
  decisions_and_approvals:
    active: []
    expired_or_invalidated: [approval_72]
    pending_requests: [decision_request_9]
  effect_frontier:
    confirmed: [effect_01K...]
    attempting: []
    unknown: [effect_01M...]
    next_reconciliation: 2026-08-31T06:10:00Z
  pending_timers: [timer_ack_2]
  outcome_frontier:
    pending_definitions: [outcome://revenue-recovery/3]
    next_check: 2026-09-12T07:00:00Z
  next_safe_action: await_accountable_disposition
  artifacts: [artifact://case-evidence/a73]
  explicit_omissions:
    - {class: raw_query_rows, reason: reconstruct_from_receipt_under_current_rights}
    - {class: superseded_model_messages, reason: non_authoritative}
  unresolved_failures: [provider_effect_timeout]
  retention_and_deletion_refs: [retention://commerce-cases/4]
  hash: sha256:91c...
~~~

On resume:

1. authenticate actor and tenant;
2. load authoritative case and effect state;
3. verify checkpoint hash and event frontier;
4. discard stale approvals and expired evidence;
5. rebuild context from current sources;
6. re-evaluate budgets and stop conditions;
7. continue from a legal state transition.

Never ask the model to summarize authority, unknown effects, or unresolved approvals as the only durable record.

Test compaction as a state migration, not a prose-quality task. Compact the same case repeatedly, crash before and after receipt commit, deliver an event between receipt construction and resume, expire an approval, and leave an effect unknown. Compare the resumed legal commands, pinned versions, effect/outcome frontier, timers, contradictions, budgets, and audit lineage with an uncompacted oracle. Any missing authority restriction, unresolved contradiction, pending decision, unknown effect, outcome check, or explicit omission is a hard failure. A receipt may summarize evidence, but evidence references and authoritative records remain independently loadable.

## Retrieval policy

Outcome-episode retrieval uses:

- same tenant and compatible purpose;
- same metric family and semantic compatibility;
- recent enough watch/detector regime;
- verified outcome and operator labels;
- data class and recipient eligibility;
- diversity controls to avoid one repeated incident dominating;
- no cross-tenant or “similarity first, filter later” query.

Retrieved episodes are examples, not rules. Old thresholds, owners, and actions are explicitly non-authoritative.

## State and memory failure matrix

| Failure | Detection | Recovery |
|---|---|---|
| Duplicate schedule event | Evaluation key exists | Return existing result or resume incomplete attempt |
| Stale case command | Expected version mismatch | Reload and require renewed decision |
| Snapshot/event mismatch | Frontier/hash check fails | Rebuild from authoritative events; quarantine snapshot |
| Cross-tenant retrieval | Policy test or audit invariant | Fail closed, revoke request, incident review |
| Compaction omits unknown effect | Checkpoint validation | Reject checkpoint and reconstruct from ledger |
| Episode contains wrong label | Review or later contradictory evidence | Supersede label; exclude from retrieval/eval |
| Retention deletion leaves index entry | Deletion reconciliation | Tombstone and verify index/artifact removal |
| Model treats old episode as policy | Output/evidence validator | Reject proposal; strengthen context labels and eval |

## Implementation checklist

- [ ] State, events, effects, delivery, telemetry, and artifacts are separate.
- [ ] Every aggregate uses optimistic concurrency and immutable history.
- [ ] Timer and message handlers tolerate at-least-once delivery.
- [ ] Effect ledger and approval references survive compaction and crashes.
- [ ] Every memory class has an explicit enabled/disabled choice.
- [ ] No personal or raw transcript long-term memory is enabled by default.
- [ ] Outcome episodes are verified, provenance-bearing, scoped, retained, and reviewable.
- [ ] Context is compiled for the current decision with freshness and trust labels.
- [ ] Retrieval filters tenant and purpose before similarity.
- [ ] Resume rebuilds authority and current state rather than trusting a summary.

## Canonical references

- [Agent state and event contracts](../../runtime/agent-state-and-event-contracts.md)
- [Durable execution](../../runtime/durable-execution.md)
- [Context engineering](../../context-memory/context-engineering.md)
- [Memory architecture](../../context-memory/memory-architecture.md)
- [Compaction and continuity](../../context-memory/compaction-and-continuity.md)
- [CloudEvents specification](https://github.com/cloudevents/spec)
- [W3C PROV overview](https://www.w3.org/TR/prov-overview/)

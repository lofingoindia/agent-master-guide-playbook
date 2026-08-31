# State, Context, Memory, and Orchestration

[Blueprint home](README.md) · Previous: [Tools, effects, integrations, and reconciliation](05-tools-effects-integrations-and-reconciliation.md) · Next: [Security, privacy, permissions, and governance](07-security-privacy-permissions-and-governance.md)

## State is not conversation memory

A campaign can wait weeks between brief, review, schedule, delivery, conversion maturity, and sales outcome. Chat history cannot safely own that lifecycle. Persist semantic state in an application database or documented durable workflow runtime; rebuild model context from current authorized records on every decision.

Keep five record families separate:

| Family | Examples | Writer and truth |
|---|---|---|
| Authoritative state | Campaign revision, current lifecycle, audience snapshot, approvals, spend reservation, experiment state, handoff disposition | Application workflow and domain services |
| Events | `asset_approved`, `suppression_advanced`, `effect_verified`, `experiment_invalidated` | Append-only domain event contract with idempotent projection |
| Effects | Audience upload, schedule, publish, pause, budget update, handoff | Effect ledger plus provider postcondition |
| Artifacts | Brief, creative, review render, audience membership, provider result, experiment readout | Versioned artifact store with digest/provenance |
| Telemetry | Spans, latency, cost, errors, redacted diagnostics | Observability plane; never campaign truth |

See [agent state and event contracts](../../runtime/agent-state-and-event-contracts.md) for shared identity/ordering rules and [durable execution](../../runtime/durable-execution.md) for replay/version mechanics.

## Campaign aggregate

```yaml
campaign_case:
  campaign_id: "cmp_01K..."
  tenant_id: "tenant_72"
  revision: 19
  lifecycle: "awaiting_reviews"
  objective_ref: "objective_31@4"
  plan_ref: "plan_31@7"
  audience_snapshot_ref: "audsnap_8X"
  creative_refs: ["creative_40@3", "creative_41@2"]
  approval_requirements_ref: "review-matrix@11"
  spend_authorization_ref: "spauth_01K"
  experiment_ref: "exp_17@1"
  active_effect_ids: []
  suppression_cursor_required: "suppression:00088421"
  deadline: "2026-09-30T23:59:59Z"
  release_manifest_ref: "release://marketing-agent/31"
  policy_versions:
    marketing_eligibility: 31
    marketing_effects: 18
  last_event_seq: 82
```

Use optimistic concurrency or a single-writer-per-campaign rule. Provider accounts and shared budgets also need cross-campaign fences. A campaign revision protects internal intent; it is not a substitute for current provider resource versions or fresh suppression state.

## Event contract

```yaml
domain_event:
  event_id: "evt_01K..."
  schema: "marketing.campaign.effect_verified@1"
  tenant_id: "tenant_72"
  campaign_id: "cmp_01K..."
  aggregate_revision: 20
  occurred_at: "2026-09-10T09:00:02Z"
  recorded_at: "2026-09-10T09:00:03Z"
  actor:
    user_principal: "user:184"
    workload_principal: "spiffe://example/marketing-effect-worker"
  correlation:
    run_id: "run_01K..."
    effect_id: "eff_01K..."
    trace_id: "00-..."
  payload_ref: "artifact://events/evt_01K"
  policy_version: "marketing-effects@18"
  release_ref: "release://marketing-agent/31"
```

Events are immutable observations. Corrections append a new event that references the superseded fact. Consumers deduplicate by `event_id`, enforce tenant/campaign identity, and reject unknown incompatible schema versions.

## Exactly seven memory lifetimes

The acting system has exactly these seven lifetimes. CRM history, consent/suppression, audience membership, provider resources, brand/claim policy, and the effect ledger are governed domain records, not additional model memories.

| Canonical label | Use | Reject | Retention and deletion | Poisoning control | Evaluation control |
|---|---|---|---|---|---|
| **Turn/scratch memory** | Current operator instruction and one immediate bounded model/tool exchange | Secrets, audience members, consent, approval, budget, provider truth, or future work queue | Destroy after typed artifact/state transition; correction recompiles the turn | Untrusted asset/page/provider text cannot add a capability, target, recipient, claim, or policy | Injection corpus proves no D3 proposal can bypass schema/policy; compare with no-history turn |
| **Working/run memory** | Open questions, candidate variants, inspected evidence refs, current plan and budgets | Authoritative campaign/effect state or raw customer lists | Derived cache with run TTL; rebuild from ledger; source deletion invalidates projection | Taint/provenance survives summarization; repeated context cannot remove suppression or a contradiction | Long-run and repeated-compaction cases preserve constraints, abstention and stop conditions |
| **Session memory** | Selected campaign/tab, review filters and unresolved UI comments for one authenticated session | Cross-session authority, user profiling, source truth, or approval inference | Principal/tenant bound, short TTL, clearable; durable reviewer decisions are separate signed records | Reviewer text remains untrusted content and cannot grant tools or persist policy | Session reset/resume yields the same domain decision from current authoritative state |
| **Durable workflow/task memory** | Campaign state, versions, timers, approvals, spend reservations, effects, handoffs and maturity waits | Hidden reasoning, unverifiable summaries, free-form inferred preferences | Append/version/supersede under record policy; tombstones propagate to checkpoints, caches and restore | Only typed validated events can write; actor, schema, policy and behavior version required | Crash, reorder, duplicate, stale-worker, deletion and unknown-effect suites converge to one valid state |
| **Domain knowledge memory** | Reviewed brand rules, claim evidence, channel/consent policy, metric definitions, schemas and approved templates | Campaign anecdotes, provider marketing copy or a model-created rule | Owner, provenance, effective/expiry time, rollback and deletion; invalidation finds affected campaigns/assets | Admission is human-governed; pinned active policy wins over poisoned retrieval | Policy-conflict, stale-source, rights and retrieval-authorization fixtures gate every release |
| **Long-term/preference memory** | Optional explicit accessibility or display setting that cannot affect targeting, claims, money or authority | Inferred marketer style, personal/customer profile, campaign interests or optimization posture | Rejected by default; if justified, consented, inspectable, correctable, exportable and independently deletable | Preferences cannot change audience, destination, claim bar, sender, budget, approval or retention | Paired tests prove identical control decisions with preference on/off; deletion removes influence |
| **Episodic/outcome memory** | Human-curated, minimized incidents, corrections, experiment outcomes and reviewer reason codes | Raw campaigns, post-treatment leakage, direct conversion reinforcement, unexplained edits or automatic self-learning | Purpose/rights, lineage, comparable population, expiry, correction and deletion propagation required | Sensitive review and holdout isolation precede admission; production content never self-promotes | Time-split/tenant-split leakage checks, fairness slices and prior-bundle replay prove generalization without weakened gates |

Personal-level engagement history is not “episodic memory” for the agent. It is protected domain data with a purpose, retention, and access policy. Embeddings do not relax those obligations.

## Context projection

Build a fresh projection for each model decision:

```yaml
model_context_manifest:
  context_id: "ctx_01K..."
  tenant_id: "tenant_72"
  campaign_ref: "cmp_01K@19"
  task: "draft_paid_search_variants"
  allowed_output_schema: "schema://creative-proposal@4"
  authority_statement: "proposal_only_no_channel_effects"
  static_instruction_version: "marketing-planner@12"
  included_refs:
    - {ref: "objective_31@4", trust: "approved_internal", purpose: "task"}
    - {ref: "brief_9@5", trust: "approved_internal", purpose: "task"}
    - {ref: "claim_18@2", trust: "approved_internal", purpose: "claim"}
    - {ref: "policy:google-search@2026-08", trust: "governed_external", purpose: "constraint"}
  excluded_classes: ["raw_audience_members", "connector_secrets", "unrelated_campaigns"]
  tool_result_budget_bytes: 80000
  total_token_budget: 32000
  compiled_at: "2026-09-08T11:10:00Z"
```

### Assembly order

1. trusted system role, output schema, stop conditions, and explicit lack of authority;
2. current campaign state and the decision requested;
3. approved objective, brief, audience aggregates, spend/experiment constraints;
4. current brand, claims, channel, privacy, and review policy excerpts;
5. minimal evidence/artifacts with provenance, trust, time, and rights labels;
6. prior run facts that remain relevant and can be verified;
7. untrusted provider/user/external content clearly delimited as data;
8. budget remaining and valid next actions.

Authorization filters run before retrieval and ranking. Cache keys include tenant, principal/purpose, campaign, policy/ACL version, data class, locale, and source version. A shared “similar campaigns” vector store is unsafe unless every returned item remains independently authorized and purpose-compatible.

## Context compaction

Compact because tool results and review history grow, not to preserve the illusion of one endless conversation.

### What compaction may summarize

- tool attempts whose full artifacts are durably referenced;
- discussion that has already produced an accepted typed artifact;
- rejected variants with reason codes and links;
- repetitive provider status polls;
- resolved review comments.

### What compaction must preserve exactly

- campaign/objective/audience/asset/experiment/spend/policy/release versions;
- open questions and explicit owner decisions;
- approval scope, digest, expiry, use count, and invalidation facts;
- effect IDs, states, provider IDs, partial results, and unknown outcomes;
- suppression cursor/freshness and privacy restrictions;
- claim/evidence references, qualifiers, contradictions, and expiry;
- budgets, deadlines, schedules, time zones, cancellation, and remaining limits;
- error and rejection codes that constrain safe recovery.

```yaml
compaction_receipt:
  receipt_version: 2
  receipt_schema: marketing-continuity/2
  compaction_id: "compact_01K..."
  tenant_id: "tenant_72"
  brand_id: "brand_a"
  campaign_ref: "cmp_01K@19"
  source_event_range: [41, 76]
  source_event_high_watermark: 76
  campaign_revision: 19
  summary_artifact_ref: "artifact://compactions/compact_01K"
  preserved_refs:
    objective: "objective_31@4"
    audience: "audsnap_8X@3"
    assets: ["creative_40@3"]
    claims: ["claim_18@2"]
    budget: "spauth_01K@2"
    experiment: "exp_17@1"
  approvals:
    active: ["approval_9"]
    invalidated: ["approval_7"]
    approval_clock: "2026-09-10T07:55:00Z"
  active_clocks:
    suppression_cursor: "suppression:00088421"
    consent_observed_at: "2026-09-10T07:54:31Z"
    provider_state_observed_at: "2026-09-10T07:55:14Z"
    not_before: "2026-09-10T09:00:00Z"
    deadline: "2026-09-10T09:05:00Z"
    outcome_maturity_after: "2026-09-17T00:00:00Z"
  effects:
    pending: ["effect_7"]
    unknown: []
    last_reconciled_at: "2026-09-10T07:55:16Z"
  pending_effect_ids: ["effect_7"]
  unknown_effect_ids: []
  unresolved: ["provider_policy_review_pending"]
  versions:
    policies: ["marketing-effects@18", "consent-policy@31", "brand-policy@12"]
    behavior_bundle: "marketing-agent@31"
    prompt: "marketing-planner@12"
    tools: "tool-registry@25"
    adapters: ["google-ads-v25@7"]
    schemas: "marketing-domain@14"
  context_compiler_release: "marketing-context@9"
  compactor_release: "compactor-release@7"
  omitted_refs:
    - {class: "raw_audience_members", reason: "never_model_context", rebuild: "audsnap_8X@3"}
    - {class: "full_provider_response", reason: "artifact_only", rebuild: "artifact://provider-readback/992"}
    - {class: "rejected_low_value_variants", reason: "loss_accepted", rebuild: null}
  next_safe_action: "reconcile_effect_7"
  invariant_hash: "sha256:..."
  restart_verification:
    schema_valid: true
    event_high_watermark_not_regressed: true
    suppression_and_consent_refreshed: true
    approvals_revalidated: true
    pending_and_unknown_effects_preserved: true
    versions_available_or_migrated: true
    invariant_hash_matches: true
    validated_by: ["schema", "critical-field-diff", "resume-simulator"]
```

On resume, load authoritative state first, then compaction, then post-compaction events, and finally refresh mutable external facts. Never resume D3 work directly from a model-provider conversation handle or summary.

## Planning contract

Use a finite plan whose allowed steps are capability names, not prose commands:

```yaml
execution_plan:
  plan_id: "plan_31"
  revision: 7
  campaign_ref: "cmp_01K@19"
  steps:
    - {id: "s1", capability: "campaign.validate_contract", mode: "deterministic"}
    - {id: "s2", capability: "creative.propose_variants", mode: "model", max_calls: 2, depends_on: ["s1"]}
    - {id: "s3", capability: "creative.validate", mode: "deterministic", depends_on: ["s2"]}
    - {id: "s4", capability: "review.request", mode: "workflow_effect", depends_on: ["s3"]}
  budgets: {model_turns: 4, tool_calls: 12, variants: 6, elapsed_seconds: 600}
  completion: "review_package_created"
  escalation: "material brief ambiguity or unsupported claim"
```

The model may choose among currently enabled read/derive capabilities within a stage. It cannot insert publish, audience upload, budget, experiment promotion, or handoff steps. Replanning triggers are explicit: schema rejection, missing evidence, provider capability drift, denied approval, fresh suppression/policy change, budget conflict, or a recoverable provider failure.

## Orchestration and parallelism

One campaign workflow is the default orchestrator. Parallelize safe independent work such as rendering formats, linting links, or drafting bounded variants. Join on immutable artifacts and deterministic validation. Do not parallelize multiple writers to the same provider campaign, budget, audience, or lead-handoff key.

Multi-agent delegation is rejected for the baseline because separate marketing “planner,” “copywriter,” and “analyst” agents usually share the same evidence and produce correlated errors while increasing context and authority complexity. Add a specialist model worker only when all are true:

- it has a distinct input/output contract and no broader tools;
- its result can be independently evaluated;
- parallel or independent prompting produces measured quality/reliability gain;
- shared state is artifact-based rather than conversational;
- the parent remains responsible for policy, budget, completion, and effects.

Human review roles are not subagents. They have accountable identities and decision rights.

## Timers, cancellation, and resume

Timers wake a workflow; they do not authorize the next action. On wake-up:

1. load current campaign and release policy;
2. check cancellation, supersession, owner, and deadline;
3. refresh consent/suppression, provider state, destination, claim, budget, experiment, and approval facts;
4. invalidate stale approval or plan steps;
5. continue, re-plan, reconcile, or escalate.

Cancellation propagates to queued model work, retrieval, rendering, provider draft preparation, schedules, and child workers. In-flight external effects become `unknown` until reconciled. Late results are recorded but fenced from changing a cancelled or superseded campaign.

## Data retention and deletion

Define retention by record class and purpose:

| Record | Typical purpose | Deletion/correction behavior |
|---|---|---|
| Objective/plan/approval/effect ledger | Accountability and campaign operation | Retain per business/audit policy; restrict content and pseudonymize where possible |
| Raw audience membership | Activation only | Short TTL; protected store; deletion/suppression propagation |
| Suppression token | Prevent prohibited future marketing | Minimum identifying material under privacy/legal policy |
| Creative and claims evidence | Publication and substantiation | Version/revoke; preserve required history and remove expired access |
| Model context/response | Debug/evaluation only if justified | Off or short retention by default; deletion follows source data where required |
| Episodic/eval corpus | Governed improvement | Curated, minimized, de-identified where possible; exclude without authorized reuse |
| Diagnostic traces | Reliability/security | Redacted references by default; separate access and retention |

Deleting a source record must invalidate caches, indexes, context manifests, eval fixtures, and episodic features within the documented propagation SLO. Backups require their own expiry and restore-time deletion replay.

## State and context acceptance tests

- [ ] Process death after every state/effect boundary resumes to one valid state without duplicate verified effects.
- [ ] Event duplicates, gaps, reordering, and incompatible schemas are detected.
- [ ] Two runs cannot overwrite the same campaign/provider resource or use a superseded approval.
- [ ] Context snapshots exclude raw audiences, secrets, unrelated tenants, expired claims, and revoked content.
- [ ] Compaction round-trips every critical field and preserves unresolved/unknown effects.
- [ ] A model-provider conversation outage loses no authoritative campaign state.
- [ ] Every memory class above has a documented owner, TTL, provenance, correction, deletion, poisoning, and evaluation decision—or is rejected.
- [ ] Retrieval poisoning cannot write durable domain memory without governed review.
- [ ] Cancellation prevents late workers from committing or advancing state.
- [ ] Multi-agent mode remains off unless its measured benefit and isolation gates pass.

## Sources

- [Agent state and event contracts](../../runtime/agent-state-and-event-contracts.md)
- [Durable execution](../../runtime/durable-execution.md)
- [Context engineering](../../context-memory/context-engineering.md)
- [Compaction and continuity](../../context-memory/compaction-and-continuity.md)
- [Memory architecture](../../context-memory/memory-architecture.md)
- [CloudEvents specification](https://github.com/cloudevents/spec)
- [Google Ads change events](https://developers.google.com/google-ads/api/docs/change-event)

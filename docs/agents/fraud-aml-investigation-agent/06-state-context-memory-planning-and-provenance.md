# State, Context, Memory, Planning, and Provenance

> **Purpose:** Make long-running investigations resumable and reproducible while preventing a model transcript, compacted summary, or hidden memory from becoming case truth.  
> **Section index:** [Fraud and AML investigation agent](README.md)

## Durable state is not model memory

The authoritative investigation lives in the case system. A model invocation receives a compiled, minimized view of that state and returns proposed events. If a run, provider, prompt, or model disappears, a different authorized worker must be able to resume from durable records without trusting the transcript.

~~~mermaid
flowchart LR
    CASE["Case state + version"] --> COMP["Context compiler"]
    EVID["Evidence refs + source snapshots"] --> COMP
    POL["Policy / typology / jurisdiction versions"] --> COMP
    COMP --> CTX["Signed context manifest"]
    CTX --> MODEL["Ephemeral reasoning run"]
    MODEL --> PROP["Typed proposed events"]
    PROP --> VAL["Schema + authority + concurrency + provenance validation"]
    VAL --> CASE
    CASE --> SNAP["Durable snapshot / event history"]
    MODEL -. "transcript is telemetry, not state" .-> TEL["Restricted telemetry"]
~~~

## Memory-class decision table

The runtime recognizes exactly the seven canonical lifetimes below. “The platform remembers” is unacceptable, and a
queue, cache, index, object store, data warehouse or transcript does not create another lifetime.

| Canonical lifetime | Policy and permitted use | Reject | Retention and deletion proof | Poisoning and evaluation control |
|---|---|---|---|---|
| Turn/scratch memory | Use for one model call: bounded evidence projections, tool results, plan and budgets | Authority, approval, deadline or sole material claim | Destroy at call end; prove no prompt/tool body survives in ordinary caches or traces | Inject instructions and false identifiers in evidence; prove no scope change or persistence |
| Working/run memory | Use ephemerally for retrieval candidates, comparisons and unresolved observations in one attempt | Dispositive identity, suspicion, sanctions, filing or customer-action decision | Discard after checkpoint, cancellation or failed attempt; durable output contains governed references only | Correct a source mid-run; stale candidates must invalidate before case mutation |
| Session memory | Limit to active UI focus, filters, pending reviewer question and unsaved display state | Customer profile, case truth, credentials, approval or cross-session inference | Expire on logout/inactivity and support explicit deletion; no cross-user reuse | Cross-user, cross-tenant and stale-entitlement tests show zero leakage |
| Durable workflow/task memory | Use as typed case state, evidence refs, hypotheses, gaps, decisions, approvals, clocks and effects | Hidden reasoning, copied source corpus or model-inferred authority | Apply record-class retention, confidentiality, hold, correction and verified deletion/tombstone rules | Tamper/replay/concurrency tests preserve lineage and reject invalid state |
| Domain knowledge memory | Use only human-published, versioned typologies, terminology, schemas, procedures and legal-source references | Customer/account facts, drafts, unsigned policies, withdrawn or not-yet-effective rules | Preserve effective/superseded lineage; withdraw from new use without rewriting case history | Poison, revoke or replace a pack/list/parser; quarantine and trace every dependent case/release |
| Long-term/preference memory | Limit to explicit UI language, time-zone display and accessibility preferences | Investigator/customer risk profiles, preferred outcomes, evidence shortcuts, role or authority | Visible, editable, purpose-limited, expiring and deletable independently of governed case records | A preference must never alter queries, evidence, thresholds, recommendations, policy or confidentiality |
| Episodic/outcome memory | Restrict to independently adjudicated, minimized/de-identified failure/evaluation exemplars with approved purpose | Raw SAR/STR narratives, automatic precedent, silent online learning or unreviewed investigator edits | Track lineage, tenant/purpose, approval, expiry, hold and derived-copy deletion | Mislabel/selection-bias/contamination tests; quarantine corrected outcomes and re-evaluate affected releases |

Customer/account records, semantic/vector indexes and caches are governed sources or derived accelerators, not extra memory lifetimes. Indexes must propagate permission, deletion and source-version changes and can never authorize an action; caches must be keyed by tenant, purpose, policy, source version and classification with a bounded TTL. If episodic retrieval cannot be shown to improve a measured outcome without selection bias, privacy expansion, or investigator anchoring, do not ship it.

## Authoritative state model

~~~json
{
  "case_id": "case_01...",
  "case_version": 17,
  "generation": 1,
  "tenant_id": "tenant_...",
  "purpose": "aml_case_investigation",
  "status": "evidence_gathering",
  "alert_refs": ["alert_..."],
  "subject_scope": ["party_...", "acct_..."],
  "case_cutoff_at": "...",
  "jurisdiction_profile": "...@sha256:...",
  "behavior_release": "fraud-aml@2026.08.31.1",
  "plan": {"id": "plan_...", "version": 3, "step_states": []},
  "evidence_refs": ["ev_...", "df_..."],
  "hypothesis_refs": ["hyp_..."],
  "gaps": [],
  "decisions": [],
  "approvals": [],
  "effects": [],
  "deadline_at": "...",
  "owner": "queue_...",
  "retention": {"class": "...", "legal_hold": false},
  "created_at": "...",
  "updated_at": "..."
}
~~~

Store large content once in protected evidence/artifact storage. State contains stable references and hashes. Restrict and log reads independently of model invocation.

## Event and reducer contract

Use events to explain state changes, not as a reason to force full event sourcing where the project does not need it. A transactional case row plus append-only business events is sufficient for many systems.

~~~json
{
  "event_id": "evt_01...",
  "event_type": "evidence.attached.v1",
  "aggregate": {"case_id": "case_01...", "expected_version": 17},
  "idempotency_key": "case_...|evidence|ev_...",
  "actor": {"type": "agent", "run_id": "run_...", "principal": "service_..."},
  "occurred_at": "...",
  "received_at": "...",
  "payload": {"evidence_ref": "ev_...", "purpose": "aml_case_investigation"},
  "provenance": {"proposal_id": "prop_...", "context_manifest_hash": "sha256:..."},
  "policy": {"commit": "sha256:...", "decision_id": "pdp_..."},
  "trace_id": "..."
}
~~~

Reducer invariants:

- reject missing or stale `expected_version`;
- deduplicate the semantic operation, not only transport messages;
- validate referential integrity, tenant, purpose, classification, lifecycle, and actor eligibility;
- append the policy decision and proposal lineage;
- never let an agent-provided timestamp, role, source ID, or approval become trusted without server validation;
- emit outbox messages transactionally with state where downstream work is required.

See [agent state and event contracts](../../runtime/agent-state-and-event-contracts.md) and [durable execution](../../runtime/durable-execution.md).

## Investigation plan contract

Planning is a bounded case artifact, not an unrestricted to-do list.

~~~json
{
  "plan_id": "plan_01...",
  "case_id": "case_01...",
  "case_version": 17,
  "objective": "Prepare a reviewable rapid-movement alert assessment.",
  "completion_policy": "rapid-movement@5",
  "steps": [
    {
      "step_id": "s1",
      "kind": "read",
      "question": "Can the alert trigger be reproduced from settled transactions?",
      "tool": "transactions.window",
      "scope": {"account_refs": ["acct_..."], "window": ["...", "..."]},
      "requires": [],
      "status": "pending",
      "budget": {"records": 500},
      "stop_on": ["partial-coverage", "identity-conflict"]
    }
  ],
  "budgets": {"tool_calls": 12, "tokens": 20000, "wall_seconds": 180, "graph_nodes": 250},
  "replan_triggers": ["material-new-evidence", "source-failure", "case-version-change", "jurisdiction-change"],
  "created_by": {"run_id": "run_..."}
}
~~~

The workflow validates tools/scopes and can execute independent read steps in parallel. Replanning cannot widen subject, time, jurisdiction, data class, or action tier without a new deterministic authorization. Completion rules and budgets are application-owned.

## Context manifest

The context compiler creates a fresh view for each model call:

~~~json
{
  "context_manifest_version": "fraud-aml-context.v1",
  "case_id": "case_01...",
  "case_version": 17,
  "run_id": "run_01...",
  "behavior_release": "fraud-aml@2026.08.31.1",
  "purpose": "aml_case_investigation",
  "jurisdiction_profile": "...@sha256:...",
  "policy_commit": "sha256:...",
  "typology_refs": ["rapid-movement@5"],
  "evidence_projection_refs": [{"ref": "view_...", "hash": "sha256:...", "classification": ["..."]}],
  "unavailable_or_excluded": [{"source": "...", "reason": "wrong-purpose"}],
  "open_hypotheses": ["hyp_..."],
  "material_contradictions": ["claim_..."],
  "plan_ref": "plan_...@3",
  "budgets_remaining": {"tool_calls": 6, "tokens": 9000},
  "deadline_at": "...",
  "redaction_profile": "...@sha256:...",
  "compiler_version": "context-compiler@sha256:...",
  "manifest_hash": "sha256:..."
}
~~~

### Assembly order

1. Non-negotiable role, authority, stop, and output schema.
2. Current tenant, purpose, jurisdiction, case version, deadline, and budgets.
3. Compact plan and open questions.
4. Structured facts and evidence handles, prioritized by materiality and freshness.
5. Contradictions, benign alternatives, missing/denied/unavailable sources.
6. Approved typology or procedure excerpts, visibly tagged as reference data.
7. Most recent typed proposals/tool results needed for continuity.

Do not fill the window with raw histories or previous prose. Retrieve on demand, summarize deterministically where possible, and keep source handles. Provider/model context limits and tool behavior belong in the pinned behavior release; they are not stable assumptions.

## Compaction and continuity

Provider compaction can reduce token pressure, but its output is an opaque continuation aid. It is not a case snapshot, evidence object, decision, or approval. The application produces its own typed continuity record before any compaction:

~~~json
{
  "receipt_version": 1,
  "continuity_version": "case-continuity.v1",
  "case_id": "case_01...",
  "case_version": 17,
  "input_context_hash": "sha256:...",
  "output_projection_hash": "sha256:...",
  "derived_event_range": {"first": "evt_...", "last": "evt_..."},
  "source_event_high_watermark": 9021,
  "source_high_watermarks": {"ledger": "offset:...", "kyc": "revision:...", "case": "event:..."},
  "source_and_list_snapshots": [{"ref": "ofac-sls:...", "published_at": "...", "hash": "sha256:..."}],
  "version_pins": {
    "behavior_release": "fraud-aml@2026.08.31.1",
    "jurisdiction_policy": "us-bank-2026-08@sha256:...",
    "rule_and_typology": ["rapid-movement@5"],
    "model": "approved-model-snapshot@...",
    "tools_and_adapters": {"transactions.window": "ledger-read@7"},
    "entity_graph_features": ["entity-resolver@2.4.1", "graph-build@19"],
    "schemas": {"case": 7, "filing_draft": 4}
  },
  "scope": {"subjects": ["..."], "window": ["...", "..."]},
  "verified_claim_refs": ["claim_..."],
  "hypotheses": [{"ref": "hyp_...", "status": "unresolved"}],
  "contradictions": ["..."],
  "material_gaps": ["..."],
  "decisions_and_approvals": [{"ref": "approval_...", "expires_at": "...", "bundle_hash": "sha256:..."}],
  "active_clocks": [{"clock_id": "clk_...", "rule": "sar-review@...", "due_at": "...", "owner": "queue_..."}],
  "pending_effects": [{"effect_id": "eff_...", "state": "dispatching"}],
  "unknown_effects": [{"effect_id": "eff_...", "attempt": 1, "reconcile_after": "..."}],
  "pending_effect_ids": ["eff_..."],
  "unknown_effect_ids": ["eff_..."],
  "omitted_item_refs": [{"kind": "tool_payload", "reason": "replaced-by-source-ref", "ref": "ev_..."}],
  "next_safe_action": "reconcile:eff_...",
  "budgets_remaining": {"...": "..."},
  "context_compiler_release": "fraud-aml-context@sha256:...",
  "compactor_release": "fraud-aml-compactor@sha256:...",
  "invariant_hash": "sha256:...",
  "artifact_hash": "sha256:..."
}
~~~

On resume, reauthorize workforce and workload identity, tenant, purpose and case entitlement; verify the receipt and
projection hashes; reload authoritative case/effect/clock state; fetch each source past its recorded watermark; resolve
the pinned list, policy, rule, model, tool and adapter releases; recheck approval expiry and cancellation; and verify
the invariant hash before compiling a fresh context. If a source revision is unavailable, a clock/effect is missing, a
hash differs, or the next action is no longer safe, pause for reconstruction or reconciliation. Never resume external
work from a provider token or prose summary alone.

### Compaction tests

Test before/after equivalence for:

- case/tenant/purpose and subject/time scope;
- material claim citations and evidence hashes;
- distinctions among fact, candidate, hypothesis, proposal, decision, and effect;
- contradictions, benign alternatives, exclusions, and missing coverage;
- current case version, source/list snapshots, policy/typology/release versions;
- approvals, deadlines, pending/unknown effects, cancellation, and next step;
- refusal of forbidden tools and cross-tenant evidence after many compactions.

Use adversarial long cases, repeated compaction/resume cycles, a crash between case checkpoint and receipt publication,
out-of-order and corrected events below a watermark, new list publication, policy/model/adapter change, lost clocks,
pending and `UNKNOWN` effects, and near-window-limit runs. A fluent summary that drops one negative fact, loses an
omitted-item reference, changes an unresolved identity into a confirmed one, or selects a different safe action fails.
Fresh reconstruction from authoritative state must reproduce the control projection byte-for-byte or produce an
explicit reviewed difference.

## Provenance graph

~~~mermaid
flowchart LR
    RAW["Source object + revision"] --> NORM["Normalized fact + transform"]
    NORM --> DER["Derived fact / entity candidate + method"]
    DER --> CLAIM["Claim + support/contradiction"]
    CLAIM --> HYP["Hypothesis proposal"]
    HYP --> BUNDLE["Review bundle + case version/hash"]
    BUNDLE --> DEC["Human decision + policy/jurisdiction"]
    DEC --> INTENT["Approved effect intent"]
    INTENT --> RECEIPT["Destination receipt/postcondition"]
~~~

Each edge is typed and queryable. Observability trace IDs correlate execution but do not replace this graph. A reviewer should be able to traverse from a decision sentence to original source revision and from a source correction to every affected claim, decision, evaluation example, and effect.

## Context and memory failure matrix

| Failure | Prevent/detect | Recovery |
|---|---|---|
| Cross-case or cross-tenant retrieval | Tenant/purpose filters at source, canary records, trace checks | Stop, revoke/quarantine, assess disclosure, reauthorize affected artifacts |
| Hidden stale customer memory | No long-term model dossier; source-version requirements | Discard context; reload authoritative records |
| Prompt injection in note/article/document | Data/control separation, structured extraction, no generic sinks | Quarantine source/proposal; continue only with safe projection |
| Compaction drops contradiction | Continuity schema and equivalence eval | Rebuild from case state; reject compacted continuation |
| Case changes during run | Optimistic case version | Rebase only through deterministic comparison; rerun material analysis |
| Vector index permission lag | Entitlement/version in retrieval and deletion tests | Disable index/source; retrieve from authoritative source |
| Previous disposition anchors new case | Episodic memory off by default; blind review/eval | Remove exemplar; re-evaluate impacted cases/releases |
| Policy changes during active case | Pinned decision profile plus emergency migration rule | Mark affected artifact stale; recompile/review |
| Poisoned typology/memory item | Signed version, curator/reviewer, provenance, retrieval telemetry | Quarantine version; trace dependents; rollback and re-evaluate |

## Explicitly rejected designs

- A chat transcript, provider conversation ID, compacted blob, or vector index as the resume record.
- Global customer memory, investigator preference memory, or cross-tenant “learning” in the runtime.
- Raw case narratives automatically written back into retrieval memory.
- Free-form plans that can add tools, subjects, time windows, data domains, or effects.
- Storing model chain-of-thought. Persist concise claims, rationale, uncertainty, sources, and decisions instead.
- “Latest policy/list/source” lookups without recording the revision used.

## Checklist

- [ ] Every memory class above has an explicit use/reject decision, owner, scope, retention, and test.
- [ ] Case state survives provider/model/runtime loss and does not depend on transcript replay.
- [ ] Proposed events are schema-, authority-, provenance-, idempotency-, and case-version-validated.
- [ ] Plans have typed steps, completion rules, budgets, stop conditions, and bounded replan triggers.
- [ ] Context manifests record included and excluded sources, versions, redaction, compiler, and hash.
- [ ] Compaction preserves identities, scope, evidence, contradictions, decisions, deadlines, effects, and versions.
- [ ] Provenance links source revisions through claims and decisions to receipts.
- [ ] Traces remain operational records; audit/evidence retention and access are separate.

## Sources and next guide

- [OpenAI — latest model guidance](https://developers.openai.com/api/docs/guides/latest-model)
- [Anthropic — Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [W3C PROV Overview](https://www.w3.org/TR/prov-overview/)
- [Memory architecture](../../context-memory/memory-architecture.md)
- [Context engineering](../../context-memory/context-engineering.md)
- [Compaction and continuity](../../context-memory/compaction-and-continuity.md)

Next: [Filings, effects, idempotency, reconciliation, and recovery](07-filings-effects-idempotency-and-recovery.md).

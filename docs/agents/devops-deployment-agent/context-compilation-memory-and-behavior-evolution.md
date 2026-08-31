# Context Compilation, Memory, and Behavior Evolution

> **Status:** Production context and change-management guide  
> **Research date:** 2026-08-31  
> **Scope:** Context assembly, compaction, memory classes, behavior-bundle upgrades, and failure mining

## Decision

Compile a fresh, bounded context for the next decision from authoritative state and provenance-bearing evidence. Treat every summary, transcript, retrieval index, and model-produced note as a disposable projection. Keep release, approval, run, event, and effect truth in application-owned stores.

Enable only the memory classes with a named production use, owner, lifetime, deletion rule, and authority boundary. Do not let a deployment agent learn procedures directly from apparent success. Convert reviewed outcomes into versioned runbooks, policy tests, or evaluation fixtures through an offline change process.

## Three stores, not one memory

~~~mermaid
flowchart LR
    A["Authoritative control state<br/>runs · releases · plans · approvals · effects"] --> C["Context compiler"]
    E["Evidence artifacts<br/>provider receipts · diffs · logs · metrics"] --> C
    K["Governed knowledge<br/>service catalog · runbooks · query templates"] --> C
    C --> V["Bounded model view"]
    V --> P["Typed proposal / diagnosis"]
    P --> D["Deterministic validation"]
    D --> A
~~~

The stores have different authority:

| Store | Examples | May establish deployment truth? | Write rule |
|---|---|---:|---|
| Control state | Run state/version, release ID, plan digest, approval set, effect ledger, target lease | Yes, for the field it owns | Trusted services with compare-and-swap or append-only contracts |
| Evidence | Registry manifest, CI receipt, Git revision, provider operation, rollout measurement | Yes, after source-specific verification | Provider adapter or evidence ingester; immutable reference and digest |
| Governed knowledge | Service catalog, ownership, policy, runbook, metric-query template | Only for declared configuration or procedure | Normal review and release process |
| Model context | Selected facts, excerpts, summaries, working hypotheses | No | Rebuilt for each decision |
| Transcript or retrieval index | Conversation history, embeddings, search metadata | No | Retention-, tenant-, and privacy-scoped derivative |

This separation keeps a compacted conversation from becoming the only place that remembers an unknown provider effect, an expiring approval, or a cancellation request.

## Context compiler contract

The compiler operates after authenticated admission and before every model call. It must not accept tenant, target, credential profile, policy, or autonomy scope from retrieved prose.

### Required lanes

| Lane | Minimum content | Can be truncated? |
|---|---|---:|
| Authority | Principal, tenant, allowed targets/actions, autonomy ceiling, incident/freeze state, budgets, deadline | No |
| Durable run state | Run/state version, cancellation, active lease, unknown effects, last verified transition | No |
| Change identity | Source and desired-state revisions, release/artifact digests, plan/policy/approval IDs and expiry | No for an effect decision |
| Current target state | Canonical target, observed revision, controller operation, drift/concurrency facts, freshness | No for an effect decision |
| Decision evidence | Relevant checks, provider receipts, rollout measurements, conflicts, missing facts | Yes, only after durable externalization |
| Procedure | Applicable versioned runbook and query/rollout templates | Only irrelevant sections |
| Working state | Hypotheses, rejected approaches, operator corrections, next question | Yes; never replaces facts above |

If a required lane cannot fit, do not silently trim it. Reduce optional evidence, use a smaller next decision, or stop and report that the context budget cannot safely support the operation.

### Evidence item

Every selected fact needs enough metadata to decide whether it is usable:

~~~yaml
schemaVersion: context-evidence/v1
evidenceId: ev_01K...
tenantId: ten_acme
source:
  system: argocd
  instance: prod-eu
  nativeId: application/checkout
  adapterVersion: argocd-read/3.4.6
subject:
  targetRef: target://ten_acme/prod-eu/checkout
  desiredRevision: 91f42e0b...
observation:
  occurredAt: 2026-08-31T04:07:41Z
  fetchedAt: 2026-08-31T04:07:44Z
  expiresAt: 2026-08-31T04:08:44Z
  completeness: complete
  nextPageToken: null
trust:
  class: authoritative-provider-read
  contentDigest: sha256:8a4d...
  signatureVerified: false
data:
  syncStatus: Synced
  healthStatus: Progressing
sensitivity: internal-operational
redactionApplied: [providerMessage]
conflictsWith: []
~~~

Use both event time and fetch time. A current API response can describe an old measurement. A complete-looking page can still be partial if pagination, regional fan-out, or provider filtering is unresolved.

### Implementable selection algorithm

Compile for the next model decision, not for the entire deployment:

~~~text
1. Reserve output, tool-schema, and recovery headroom.
2. Load authority and durable run state by authenticated IDs.
3. Derive the evidence requirements for the next decision type.
4. Retrieve only tenant- and target-authorized sources.
5. Validate schemas, source identity, freshness, pagination, and size limits.
6. Normalize facts while retaining native IDs and raw artifact references.
7. Preserve conflicts; never select the friendliest source silently.
8. Rank optional evidence by authority, decision value, freshness, coverage, and token cost.
9. Deduplicate; replace large payloads with bounded excerpts plus immutable references.
10. Emit a context manifest or reject the compilation if a mandatory lane is absent.
~~~

Source precedence is domain-specific. Git may own desired state, a GitOps controller may own reconciliation status, Kubernetes may own observed workload state, and the rollout controller may own exposure. A conflict is often the fact the agent needs to diagnose.

### Context manifest

~~~json
{
  "schemaVersion": "deployment-context/v1",
  "contextId": "ctx_01K...",
  "runId": "run_01K...",
  "stateVersion": 18,
  "nextDecision": "propose-canary-advance-or-pause",
  "authorityDigest": "sha256:1d2f...",
  "changeContractDigest": "sha256:4bd1...",
  "lanes": {
    "authority": {"required": true, "tokens": 620, "items": 1},
    "runState": {"required": true, "tokens": 480, "items": 1},
    "targetState": {"required": true, "tokens": 910, "items": 4},
    "evidence": {"required": true, "tokens": 4200, "items": 12},
    "procedure": {"required": true, "tokens": 950, "items": 2},
    "workingState": {"required": false, "tokens": 530, "items": 3}
  },
  "reservedOutputTokens": 3000,
  "compactionGeneration": 1,
  "warnings": [
    "one metric source is 45 seconds from expiry",
    "raw CI log represented by artifact ev_01K_ci"
  ],
  "compilerVersion": "deployment-context/2.1.0"
}
~~~

Record context manifests for evaluation and debugging, but keep sensitive payloads out of shared traces.

## Compaction and clean handoff

Compact before the hard model limit, after a meaningful phase boundary, or before a model/provider migration. Do not compact on every turn, and do not recursively summarize an old summary when authoritative events and raw artifacts still exist.

The checkpoint must preserve:

- accepted objective, non-goals, user corrections, authority, budgets, deadline, cancellation, and incident state;
- run/state version, active lease, intended/dispatched/unknown effects, provider operation IDs, and reconciliation deadline;
- source, desired-state, release, configuration, plan, policy, approval, and rollout-contract digests;
- completed and failed checks with receipts, evidence freshness, conflicts, and missing required facts;
- active hypothesis, rejected recovery choices and why, next safe action, and stop/escalation conditions;
- context compiler, compactor, prompt, model, tool-registry, adapter, workflow, and policy versions.

After compaction or process recovery:

1. Load the checkpoint only as a continuation hint.
2. Rehydrate authoritative run/effect state and current target state.
3. Compare checkpoint digests and versions with the stores.
4. Mark changed facts stale and compile a new context.
5. Re-run commit-time policy and approval checks before any new effect.

Provider-managed opaque compaction can preserve conversational continuity, but it is not inspectable audit evidence and cannot replace the checkpoint above. The OpenAI Responses API, for example, exposes stored [conversation items](https://developers.openai.com/api/reference/python/resources/conversations/methods/create) and opaque [compacted responses](https://developers.openai.com/api/reference/java/resources/responses/methods/compact); those are runtime conveniences, not deployment state.

Test uninterrupted, one-compaction, repeated-compaction, crash-resume, clean-handoff, and model-migration variants of the same scenario. Inject a stale approval, cross-tenant retrieval result, late prompt injection, and unknown provider effect immediately before compaction; each must survive or be rejected correctly after rehydration.

## Memory-class audit

“Memory” is not one feature. Choose each class independently:

| Memory class | Default | Why | Guardrail |
|---|---|---|---|
| Authoritative operational state | Enabled, but never as model memory | Required for durable execution and audit | Application-owned typed records; compare-and-swap and append-only events |
| Run-scoped working memory | Enabled | Tracks hypotheses, TODOs, and operator corrections | Mark unverified; expires with run; cannot authorize or prove effects |
| Current-session transcript | Enabled only when interaction needs it | Resolves references and supports review | Tenant/run scoped, redacted, bounded retention; not source of truth |
| Compaction checkpoint | Enabled for long runs | Survives context/process loss cheaply | Derived from authoritative stores; versioned; revalidated on resume |
| Evidence cache | Enabled for expensive stable reads | Controls latency, rate limits, and cost | Tenant/target key, source version, TTL, completeness, negative-cache policy |
| Semantic/vector retrieval index | Disabled for the first version; optional later | Useful only when exact lookup and metadata search no longer suffice | Filter before retrieval; provenance, deletion, poisoning, freshness, and recall tests |
| Governed procedural memory | Enabled | Service catalogs, runbooks, query templates, and policy are reusable | Reviewed source revision; read-only to the agent; normal change control |
| Cross-run user preferences | Disabled by default | Little deployment value and high scope/privacy ambiguity | Opt-in, non-authoritative, TTL/deletion, never stores targets or permissions |
| Free-form episodic memory | Disabled | Apparent success can hide later incidents, reverts, or operator correction | Curate reviewed outcomes into fixtures or runbooks offline |
| Failure corpus | Enabled through controlled ingestion | Regressions and near misses should improve tests | Sanitized, labeled, deduplicated, reviewed, versioned evaluation data |
| Secrets and credentials | Forbidden | Memory expands lifetime and exposure | Store only secret references; broker credentials directly to adapters |
| Hidden reasoning | Not stored or required | It is not operational evidence | Persist observable facts, alternatives, decisions, and receipts |
| Online self-modification | Disabled | Unreviewed prompt/tool/policy learning can expand authority | Release a versioned behavior bundle through the upgrade ladder |

The simplest safe implementation uses relational state, content-addressed evidence, exact metadata search, and a small context compiler. Add embeddings only after measured retrieval misses justify the new poisoning, tenancy, deletion, and evaluation burden.

## Behavior bundle

Treat the full agent behavior as one releasable contract:

This bundle describes the deployment agent's own reasoning and control-plane release. It does not authorize a product model for serving; champion selection, evaluator baselines, drift response, and model-serving release policy remain MLOps responsibilities.

~~~yaml
apiVersion: delivery.example.io/v1
kind: BehaviorBundle
metadata:
  id: behavior_2026_08_31_3
spec:
  model:
    provider: qualified-provider-a
    identifier: pinned-model-snapshot
    parametersDigest: sha256:81d2...
  promptTemplateDigest: sha256:0a11...
  contextCompiler: deployment-context/2.1.0
  memoryPolicyDigest: sha256:20e3...
  toolRegistryDigest: sha256:3c44...
  adapters:
    github-read: 2.7.0
    argocd-control: 3.4.6
  workflowVersion: deployment-v3.2
  policyBundleDigest: sha256:7f21...
  queryTemplateBundleDigest: sha256:991a...
  trustRootSetDigest: sha256:fe20...
  evaluatorBundleDigest: sha256:7ac1...
  evalDatasetVersion: deploy-evals/18
  releasedAt: 2026-08-31T08:00:00Z
~~~

Do not change several behavior dimensions in one production canary unless the migration requires it. Otherwise a regression cannot be attributed or rolled back cleanly.

## Upgrade ladder

Every model, prompt, context compiler, compactor, memory policy, tool schema, adapter, workflow, policy, query template, or trust-root change follows:

1. Schema, canonicalization, policy, adapter-contract, and replay tests.
2. Deterministic simulator and fault injection for affected guarantees.
3. Held-out agent evaluations, adversarial evidence, and context ablations.
4. Replay of sanitized historical changes, incidents, denials, and unknown outcomes.
5. Shadow production with no additional effects.
6. Read-only/advisory canary by tenant and workload class.
7. Bounded non-production effect canary.
8. Small production cell canary with an old-bundle comparison cohort.
9. Progressive expansion only when safety, quality, latency, cost, and operator-load gates pass.

Keep the previous behavior bundle deployable until active runs finish or migration is proven. Pin active runs to a workflow/behavior version. A security revocation may supersede a pinned version, but that is an explicit stop/reauthorize transition, not a silent upgrade.

### Tool and policy compatibility

- Additive tool fields need tolerant consumers; changed effect, idempotency, cancellation, or success semantics require a new major capability version.
- Qualify an adapter against recorded provider fixtures and a sandbox account before enabling it.
- Run old and new readers in dual-read mode; compare normalized and native receipts.
- Deploy policy bundles signed, staged, and observable. OPA can verify signed bundles, retain an existing bundle when activation fails, and report active revisions through its status API ([OPA bundles](https://www.openpolicyagent.org/docs/management-bundles), [OPA status](https://www.openpolicyagent.org/docs/management-status)).
- Decide whether an in-flight run remains on its pinned policy or must reauthorize. Revocations, incident freezes, and newly prohibited effects normally override reuse.
- Roll back the behavior bundle without rolling back provider effects. In-flight deployments continue under their deterministic controller and reconciliation path.

## Failure mining without unsafe self-learning

Mine more than incidents. Useful inputs include:

- policy denials and exceptions;
- stale plans and approval expiry;
- human edits, rejection reasons, and evidence requests;
- unknown effects and long reconciliation;
- rollback, roll-forward, and direct incident actions;
- cross-tenant and credential-broker denials;
- false alarms, missed risks, and inconclusive rollout gates;
- adapter schema drift, provider deprecation, rate-limit, and retention surprises;
- cost/latency outliers and queue starvation;
- successful changes later linked to rework, incidents, or revocation.

~~~yaml
schemaVersion: failure-observation/v1
observationId: fo_01K...
sourceEventRefs: [evt_01K_a, evt_01K_b]
workloadClass: production_release
behaviorBundleId: behavior_2026_08_31_3
failureClass: stale-approval-not-detected-in-summary
impact: near_miss
expectedInvariant: commit-rejects-material-state-change
actualTraceRef: evidence://ten_acme/sha256:4bc1...
sanitization:
  status: complete
  reviewer: security-evals
disposition:
  regressionCase: deploy-eval-0194
  policyChange: null
  adapterFix: null
  runbookChange: rbk_approve_12
owner: delivery-safety
reviewedAt: 2026-08-31T10:00:00Z
~~~

The review determines whether the outcome reveals a model error, missing evidence, bad context selection, adapter bug, policy gap, runbook gap, user-interface problem, or delivery-system weakness. Do not “fix the prompt” when the missing control belongs in trusted code.

## Upgrade rollback runbook

1. Freeze expansion and identify affected behavior-bundle IDs, runs, tenants, and targets.
2. Stop new commits for the affected capability; keep read, status, cancellation, and reconciliation available.
3. Let deterministic rollout controllers hold or recover according to already approved contracts.
4. Roll back new model/prompt/context/tool workers to the last qualified bundle.
5. Reconcile active effects; never replay a model turn that might redispatch an operation.
6. Restore policy only from a verified signed bundle and confirm active revisions.
7. Compare pre/post traces, decisions, effects, latency, cost, and operator corrections.
8. Add the failure to the held-out or regression corpus before retrying rollout.

## Acceptance checklist

- [ ] Every model input has a context manifest with required lanes, source versions, freshness, and warnings.
- [ ] Context loss or compaction cannot lose cancellation, unknown effects, approval expiry, or target identity.
- [ ] Every memory class has an explicit enabled/disabled decision, owner, lifetime, tenant boundary, and deletion rule.
- [ ] No credential, permission, target mapping, policy decision, or effect status is learned from free-form memory.
- [ ] Cross-run episodes enter production only through reviewed runbook, policy, or evaluation changes.
- [ ] Every behavior change identifies the full bundle and the guarantees it may affect.
- [ ] Old/new bundles are compared in replay, shadow, and bounded canaries.
- [ ] Tool and policy upgrades preserve native receipts, activation status, and rollback.
- [ ] Failure mining includes near misses, human corrections, ambiguous outcomes, and later-linked incidents.
- [ ] The model can be unavailable or rolled back without blocking status, cancellation, reconciliation, or manual recovery.

## Related guides

- [Reference architecture and build choices](reference-architecture-and-build-choices.md)
- [Plans, policy, approvals, and change control](plans-policy-approvals-and-change-control.md)
- [Tool adapters and deployment evidence](tool-adapters-and-deployment-evidence.md)
- [Durability, observability, evaluation, and cost](durability-observability-evaluation-and-cost.md)
- [Implementation roadmap and production tests](implementation-roadmap-and-production-tests.md)
- [Context engineering](../../context-memory/context-engineering.md)
- [Compaction and continuity](../../context-memory/compaction-and-continuity.md)

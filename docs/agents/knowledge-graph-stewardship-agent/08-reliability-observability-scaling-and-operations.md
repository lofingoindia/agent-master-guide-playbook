# Reliability, Observability, Scaling, and Operations

Production reliability means more than keeping an API online. The service must avoid false merges and destructive drift, preserve case/effect state through failures, meet review and reconciliation objectives, and degrade without pretending uncertain state is confirmed.

## Service boundaries and failure containment

Deploy independently scalable pools for:

| Pool | Workload | Failure containment |
|---|---|---|
| Connector ingestion | External reads, events, snapshots | Per-source circuit breaker and credential |
| Normalization/blocking | CPU/IO-heavy deterministic transforms | Per tenant/source pair partitions |
| Matching/inference | Model/matcher/reasoner compute | Versioned workers; strict resource budgets |
| Evidence/context | Access-filtered assembly | No effect credentials; bounded graph reads |
| Case orchestration | Durable transitions and timers | Database-backed leases/version checks |
| Review service | Human queues and packets | Read-only evidence links; authority checks |
| Effect execution | External mutation | Small allowlist; isolated identity/network |
| Reconciliation | Target reads and anti-entropy | Independent of effect attempt worker |
| Projection/indexing | Search/vector/graph derived views | Rebuildable; cannot alter ledgers |

Bulk ingestion must not starve urgent split, deletion, or security-classification cases. Use separate queues and concurrency budgets.

## Partitioning and skew

Good partition keys preserve tenant isolation and localize work:

- ingestion: tenant + connector + source partition;
- candidate generation: tenant + entity type + source pair + block-key hash;
- case orchestration: tenant + case ID hash;
- graph projection: tenant + stable entity/edge partition;
- effects: tenant + target system + effect class;
- reconciliation: tenant + target + time bucket.

Entity-resolution blocking is a recall/cost trade-off. Common blocking values create hot partitions and near-quadratic pair counts. Splink's primary documentation emphasizes that candidate comparisons grow quadratically without blocking and recommends measuring blocking recall; see [blocking rules](https://moj-analytical-services.github.io/splink/topic_guides/blocking/blocking_rules.html).

Controls for hot blocks:

- estimate block cardinality before materializing pairs;
- cap pairs per block and emit an explicit coverage gap;
- refine with additional independent keys rather than random silent sampling;
- salt large blocks for computation while deduplicating pair IDs;
- separate common/missing/default values from useful identifiers;
- maintain per-slice blocking-recall tests;
- route unresolved oversized blocks to a dedicated strategy or review sample.

Graph skew creates a similar problem. High-degree hubs such as generic tags, countries, or shared providers can explode traversals. Apply relation-specific degree/depth limits, typed traversal allowlists, time budgets, and continuation tokens. “Truncated” is part of the result contract.

## Queueing and backpressure

~~~mermaid
flowchart LR
    I[Source observations] --> Q1[Ingestion queue]
    Q1 --> C[Candidate and validation workers]
    C --> Q2[Proposal queues by risk/type]
    Q2 --> H[Human/policy review]
    H --> Q3[Approved effect queue]
    Q3 --> E[Effect executor]
    E --> Q4[Reconciliation queue]
    Q4 --> R[Target readback]
    R --> D[Completed or recovery case]
~~~

Backpressure policy, in order:

1. preserve raw observations and checkpoints;
2. stop or slow low-priority scans at the source boundary;
3. reduce optional enrichment and model calls;
4. remain propose-only when review or effect capacity is saturated;
5. reserve capacity for critical splits, privacy, and security cases;
6. reject new work with explicit retry/coverage metadata rather than dropping it;
7. never auto-lower review thresholds to drain a queue.

Use bounded retries with exponential backoff and jitter for genuinely transient operations. Honor source rate-limit reset signals. Retries consume a per-case/per-effect budget and then move to a typed deferred or recovery state.

## Service-level objectives

Define objectives by outcome and risk slice, not only aggregate latency.

| Objective | Example indicator | Notes |
|---|---|---|
| Observation durability | Accepted events durably checkpointed / accepted events | Exclude requests rejected before acceptance |
| Candidate freshness | Time from complete source revision to candidate availability | Slice by source/entity type |
| Critical-case readiness | Time from trigger to reviewable evidence packet | False-merge/security cases get stricter target |
| Review service | Time to first decision by risk tier | Queue capacity is part of system reliability |
| Approved-effect latency | Approval to confirmed target state | `unknown` is not success |
| Reconciliation completeness | Due effects with confirmed target readback | Measure stale unknowns separately |
| Projection freshness | Ledger revision to searchable graph/index revision | Stale projection clearly labeled |
| Decision quality | False merge/split and reversal rate by slice | Guardrail/SLO, not a vanity accuracy average |

Example error-budget policy:

- when candidate freshness burns budget, pause optional full scans and add compute;
- when review SLO burns budget, switch low-risk work to deferred/propose-only and add reviewer capacity;
- when effect/reconciliation SLO burns budget, stop new mutation classes before increasing retries;
- when false-merge guardrail breaches, disable automatic approval for the affected policy/model/source slice.

Never trade decision-quality guardrails for throughput without an explicit incident decision.

## Observability model

Correlate by stable IDs:

```text
source revision -> observation -> candidate/validation run -> case
                -> proposal -> decision -> effect attempt -> receipt/reconciliation
```

### Metrics

Record distributions and counts by approved low-cardinality dimensions:

- connector lag, snapshot coverage, page retries, and permission-scope changes;
- blocking pairs, block-size quantiles, blocking recall samples, and duplicate pairs;
- matcher calibration, decision bands, abstentions, contradictions, and drift slices;
- SHACL/quality results by rule/source/semantic bundle;
- proposal/review/appeal/reversal counts and queue ages by risk/type;
- effect states, conditional conflicts, unknown duration, reconciliation mismatch;
- graph degree/traversal truncation, index lag, reasoner closure size/runtime;
- model latency, token/cost, schema-invalid output, citation failure, policy denial;
- tenant isolation/security denials without exposing tenant data.

### Traces

Trace connector call, observation transform, candidate stages, context compilation, model invocation, policy evaluation, state transition, effect call, and reconciliation. Attach artifact IDs/digests and versions, not raw sensitive payloads.

### Logs and audit

Operational logs answer why a worker failed. Audit records answer who/what authorized a governance change. Keep both, with different retention and access. Redact raw record fields before export.

### Evidence plane

Dashboards and traces are navigation aids; the evidence plane is the durable, access-controlled set of artifacts that proves an outcome. For every sampled or high-risk case it must resolve:

```text
release manifest + source revisions + context/evidence digests
  -> trajectory/tool attempts + policy decisions + state events
  -> human decision/authority + effect requests/receipts
  -> target observations + projection checkpoints + invariant results
```

Spans carry IDs, versions, status, retry/ambiguity class, bytes/tokens/cost, and links to restricted evidence—not raw PII. Use OpenTelemetry semantic conventions where they fit, but define stewardship attributes such as case/proposal/effect IDs and source/target watermarks in a versioned internal schema; OpenTelemetry's convention groups have mixed stability, so pin the semantic-convention release ([OpenTelemetry semantic conventions](https://opentelemetry.io/docs/specs/semconv/)).

An evidence query should reconstruct “what was known, decided, attempted, and observed” without joining by free-text message. Continuously sample reconstruction: select a completed case, resolve every referenced artifact/digest, verify chronological and aggregate-version consistency, and compare target state. Missing evidence is an operational defect even if the end state happens to be correct.

## Capacity model

Estimate each stage separately:

```text
candidate comparisons ≈ sum(block_size * (block_size - 1) / 2)
review demand          = candidates_in_review_band * minutes_per_case
effect demand          = approved_cases * effects_per_case
graph work             = changed assertions * affected projection/inference fanout
```

Average event rate hides burst and skew. Capacity tests should include:

- largest credible source snapshot and catch-up burst;
- dominant/missing blocking keys;
- high-degree graph nodes and lineage cycles;
- semantic release that invalidates many records;
- reviewer outage/weekend backlog;
- target API rate limiting and partial batches;
- replay/backfill while live ingestion continues;
- one noisy tenant under shared infrastructure.

Autoscale stateless compute on queue work/estimated cost, not message count alone. Scale case/effect databases through measured indexing/partitioning before adding distributed complexity. Protect external targets with per-target concurrency and token buckets.

### Per-case cost and work budget

Estimate and enforce budgets by case type and risk:

```text
total cost = source/API reads + candidate comparisons + graph traversal
           + reasoner/validation CPU + model tokens + storage/index writes
           + expected curator minutes + reconciliation/recovery reserve
```

Record actual versus estimated comparisons, nodes/edges, bytes, calls, tokens, wall/CPU time, queue time, reviewer minutes, effects, and reconciliation reads. A case that hits a limit returns bounded partial coverage and a next-safe disposition. Never hide overflow by sampling without declaring missed coverage. Charge replay, appeals, incident correction, and delete propagation to the policy/model/source slice that caused them; otherwise a cheap proposer can create expensive downstream toil.

Backpressure gates use both machine and human capacity. Admission should forecast review minutes and downstream effects, reserve recovery headroom, and stop low-risk proposal creation before the steward or reconciler queue becomes unsafe. Measure cost per reconciled correct outcome, not cost per candidate.

## Deployment and upgrade manifest

```yaml
release_manifest:
  release_id: steward-2026.09.1
  orchestrator: 4.6.0
  normalizer_bundle: norm-17
  blocking_policy: block-12
  matcher_build: matcher-31
  calibration_policy: cal-8
  model_id: model-build-2026-08-20
  prompt_digest: sha256:...
  semantic_bundle: customer-domain-2026.09.0
  tool_manifest: tools-41
  connector_manifests: [datahub-1.7, openmetadata-1.12, atlas-2.5, sr-2026.08]
  state_schema: 23
  event_schema_set: events-12
  policy_bundle: governance-18
```

Upgrade flow:

1. validate storage/event forward and backward compatibility;
2. replay a frozen evaluation corpus with the complete manifest;
3. shadow on current observations without mutations;
4. compare candidate, inference, validation, proposal, and cost deltas by slice;
5. canary selected tenants/domains with reversible or propose-only paths;
6. expand under automatic guardrails;
7. retain previous workers and semantic bundle through rollback window;
8. backfill derived projections only after behavior gates pass.

For matcher/model changes, do not recompute old decisions silently. Create new candidates or drift observations, then follow policy. For semantic changes, use the versioned bundle migration described in the ontology guide. For connector upgrades, re-verify permissions, pagination, delete semantics, and target capability—not only response parsing.

### Canary the whole behavior bundle

The release unit is not “the model.” It is normalizers, blocking/cluster rules, model and prompts, semantic bundle, context compiler, output validators, policies, tool/adapter manifests, state/event schemas, projection builds, provider/security settings, and reviewer UI. Changing any component can alter outcomes or authority exposure.

Use four progressive comparisons:

1. **frozen replay:** run old and new bundles on the same temporally safe corpus; compare trajectories, candidates, abstentions, decisions proposed, invalidations, costs, and invariants;
2. **shadow:** consume current observations without creating review noise or effects; compare drift and workload forecast;
3. **propose-only canary:** expose a small authorized cohort to reviewers; measure quality, disagreement, handling time, workload, and unsafe-action attempts;
4. **effect canary:** enable only reversible allowlisted operations with independent readback, small blast radius, and automatic stop gates.

Rollback means route new work to the previous bundle, stop new effects from the candidate release, reconcile in-flight effects, and rebuild derived projections whose semantics changed. Decisions/effects already committed remain history and may require forward correction. Keep old adapters/upcasters and semantic bundles for the declared rollback/replay window.

## Disaster recovery

Classify stores:

| Store | Recovery priority | Strategy |
|---|---|---|
| Decision/effect/assertion ledgers | Highest | Point-in-time recovery, cross-zone copy, restore drills, integrity verification |
| Case state/outbox | Highest | Transactionally backed up with replayable events |
| Raw evidence | High, policy-dependent | Immutable/versioned storage, checksum inventory |
| Connector cursors/state | High | Versioned checkpoints; safe resnapshot procedure |
| Graph/search/vector projections | Rebuildable | Rebuild from ledgers with versioned projection code |
| Prompt/model caches | Disposable | Recreate; never required for recovery |

After restore:

1. fence all effect executors;
2. restore authoritative ledgers and verify checksums/revisions;
3. identify effects whose target may be ahead of restored state;
4. reconcile external targets before resuming mutation;
5. restore connector positions or conduct safe snapshots;
6. rebuild derived projections and compare counts/digests;
7. gradually resume reads, proposals, then effects.

An RPO of zero cannot be assumed for an external API. Effect receipts plus target reconciliation close the gap.

## Incident runbooks

### False merge or identity collapse

- Disable affected auto-approval/model-policy slice and freeze new effects on impacted canonicals.
- Enumerate assertions, decisions, derived facts, aliases, access/consent changes, and exported targets.
- Open critical split cases; preserve original evidence.
- Forward-correct targets and rebuild projections.
- Measure exposure and notify security/privacy/owners as required.

### Bad ontology, shape, or vocabulary release

- Stop rollout and enforcement; pin traffic to last known-good bundle.
- Compare inferred/validated/query deltas and affected effects.
- Rebuild derived closure/index; do not delete source assertions.
- Forward-fix persistent published identifiers when rollback cannot erase external use.

### Lineage poisoning or runaway propagation

- Quarantine producer/connector and stop classification effects derived from it.
- Mark affected observations disputed; retain them for forensics.
- Rebuild effective lineage from trusted assertions and decisions.
- Reconcile downstream classifications and access consequences.

### Unknown-effect storm

- Open the target circuit; stop writes while preserving approved queue.
- Query target health and reconcile unknowns at a bounded rate.
- Avoid blind retry and duplicate compensation.
- Resume per effect class after idempotency/readback behavior is proven.

### Tenant isolation incident

- Fence affected workers, indexes, credentials, and exports immediately.
- Preserve security audit, manifests, and request traces under incident controls.
- Identify raw, derived, cached, model-provider, and target exposure.
- Rebuild contaminated shared projections from isolated ledgers.

## Incident and recovery load

Readiness includes people and target capacity, not just an executable runbook. For each critical incident class, estimate and game-day:

- detection and scoping minutes, cases/entities/edges/consumers affected, and evidence-query load;
- steward, privacy/security, ontology, source-owner, platform, and communications staffing by shift;
- reconciliation reads and corrective writes against target rate limits;
- projection rebuild CPU/storage/time and live-delta catch-up;
- external-recipient notification/acknowledgement work;
- appeal and manual-review surge after recovery;
- maximum safe concurrent incidents and the admission work that must pause.

Track recovery toil hours, pages/interruptions, manual decisions, unknown-effect age, backlog added, and post-incident correction completion. A recovery drill fails if it meets a technical RTO only by exceeding available steward/on-call capacity or by abandoning downstream verification. NASA-TLX can add a standardized subjective workload measure for representative curator/operator studies; pair it with objective time, errors, rework, interruptions, and queue outcomes rather than using workload self-report alone ([NASA TLX](https://www.nasa.gov/human-systems-integration-division/nasa-task-load-index-tlx/)).

## Degraded modes

| Failure | Permitted mode | Prohibited claim/action |
|---|---|---|
| Model unavailable | Deterministic exact rules and existing cases; otherwise defer | Pretend heuristic result is model-equivalent |
| Graph index stale | Read ledger-backed facts with staleness notice | Use stale neighborhood for destructive effect |
| Review service saturated | Ingest and prepare evidence; propose-only/defer | Lower approval threshold |
| Target unavailable | Queue approved effects; read-only reconciliation when possible | Mark approved as applied |
| Policy/authority service unavailable | Continue non-sensitive reads from safe cached policy only if designed | New approvals or effects |
| Connector permission unclear | Quarantine/unknown | Propagate deletes from omissions |

## Operational readiness checklist

- [ ] Pools, queues, and credentials contain failures by trust and workload class.
- [ ] Partitioning handles tenant isolation, hot blocks, and high-degree graph nodes.
- [ ] Backpressure preserves evidence and high-risk work without reducing review rigor.
- [ ] SLOs cover decision quality, review, effects, reconciliation, and projection freshness.
- [ ] Traces and the evidence plane join source-to-effect artifacts without leaking payloads and pass reconstruction sampling.
- [ ] Capacity/cost includes curator, reconciliation, recovery, replay, and downstream invalidation work.
- [ ] Release manifests pin model, matcher, context, semantic, policy, connector, and state versions.
- [ ] Frozen replay, shadow, propose-only, whole-bundle effect canary, rollback/forward-fix, and backfill are rehearsed.
- [ ] DR restores ledgers first, fences effects, and reconciles external targets.
- [ ] Runbooks and staffing/load drills cover false merge, semantic release, lineage poisoning, unknown effects, tenant leak, and recipient/rebuild surge.

## Related guidance

- [Catalog, lineage, provenance, and reconciliation](04-catalog-lineage-provenance-and-reconciliation.md)
- [Evaluation and failure injection](09-evaluation-failure-injection-and-staged-delivery.md)
- [Queues, scheduling, and backpressure](../../operations/queues-scheduling-and-backpressure.md)
- [Scaling, capacity, and SLOs](../../operations/scaling-capacity-and-slos.md)

# Implementation Blueprint

> **Purpose:** Provide a buildable, framework-neutral starting point. The examples are contracts and pseudocode, not a library to copy unchanged.

## Deployable units

Start with four logical units; co-locate where safe:

1. **Research API/control worker:** admission, brief, workflow/controller, context assembly.
2. **Search/fetch/parse gateway:** provider adapters, SSRF-safe public egress, document sandbox.
3. **Evidence/claim service:** transactional metadata plus encrypted content-addressed object storage.
4. **Verification/render worker:** claim checks, artifact intermediate representation, deterministic render/publish.

Recommended supporting infrastructure:

- relational database for runs, plans, sources, claims, budgets, leases, and decisions;
- object storage for captured representations and artifact bundles;
- durable workflow or queue for long jobs;
- secrets/credential broker;
- observability pipeline with redaction;
- optional search index for an approved internal/frozen corpus.

Do not add a vector database until semantic retrieval over retained evidence has a measured need. Relational indexes plus full-text search are often sufficient for the first version.

## Storage layout

```text
relational database
├── research_runs / run_budgets / run_leases
├── briefs / brief_revisions
├── plans / questions / branches / query_runs
├── connector_manifests / connector_qualifications / provider_cursors
├── sources / source_relations / source_status_revisions / representations
├── evidence_spans / evidence_decisions
├── claims / evidence_edges / contradiction_sets
├── verifications / reviewer_decisions
├── artifacts / artifact_revisions / publication_receipts
├── memory_promotions / continuity_receipts / propagation_jobs
├── behavior_bundles / release_cohorts / rollback_decisions
└── application_events / outbox

object storage (content-addressed, governed)
├── raw-representations/<sha256>
├── derived-representations/<sha256>
├── evidence-bundles/<bundle-hash>
├── artifact-intermediate/<hash>
└── released-artifacts/<artifact>/<revision>
```

Store tenant/access scope in every authoritative record and storage key policy. A content hash does not authorize cross-tenant reuse.

## Core interfaces

```python
from dataclasses import dataclass
from typing import Literal, Protocol

@dataclass(frozen=True)
class Budget:
    deadline_epoch_ms: int
    search_calls_left: int
    fetch_bytes_left: int
    model_tokens_left: int
    cost_microusd_left: int

@dataclass(frozen=True)
class SearchRequest:
    operation_id: str
    run_id: str
    question_id: str
    query: str
    filters: dict
    authorization_scope: str
    budget: Budget

@dataclass(frozen=True)
class PageReceipt:
    sequence: int
    provider_cursor_ref: str | None  # secret/reference, never raw in model context
    complete: bool
    truncated: bool
    reported_total: int | None

@dataclass(frozen=True)
class ToolReceipt:
    operation_id: str
    adapter_id: str
    adapter_version: str
    provider_operation_id: str | None
    attempt: int
    started_at: str
    finished_at: str
    authorization_scope_id: str
    region: str
    request_hash: str
    response_hash: str | None
    page: PageReceipt | None
    cost: dict
    rights_class: str
    retention_class: str

@dataclass(frozen=True)
class SearchResultSet:
    result_set_id: str
    receipt: ToolReceipt
    item_ids: tuple[str, ...]
    coverage_statement: str
    next_page_available: bool
    evidence_eligible: bool  # normally False until exact acquisition

@dataclass(frozen=True)
class EvidenceProposal:
    representation_id: str
    locator: dict
    exact_text_hash: str
    question_ids: tuple[str, ...]
    proposed_relation: Literal["supports", "refutes", "limits", "defines", "contextualizes"]
    rationale: str

class SearchGateway(Protocol):
    async def search(self, request: SearchRequest) -> SearchResultSet: ...

class EvidenceLedger(Protocol):
    async def propose(self, proposal: EvidenceProposal) -> str: ...
    async def decide(self, proposal_id: str, decision: "EvidenceDecision") -> str: ...
```

Tool results return stable IDs, compact summaries, page/completion state, cost/attempt/access/rights metadata, and typed errors. They never return credentials, raw continuation tokens, or an unbounded page dump to the controller. Full allowed payloads go directly to governed storage and are referenced by representation/result IDs.

Each adapter implements four application-level operations as applicable:

```text
discover(request) -> SearchResultSet
fetch(source_locator, capture_policy) -> RepresentationReceipt
status(source_id, prior_status_revision) -> SourceStatusReceipt
reconcile(operation_id) -> ToolReceipt
```

Database adapters expose `query()` rather than pretending rows are documents; browser adapters expose `render()` with action/network receipts. The [connector qualification guide](connectors-and-provider-qualification.md) supplies the capability manifest and exact provider examples.

## Error taxonomy

```python
class ToolFailure(Exception):
    code: str
    retryability: Literal["transient", "permanent", "reconcile", "model_correctable"]
    safe_message: str
    operation_id: str
    retry_after_ms: int | None
```

Examples:

- `SEARCH_RATE_LIMITED` → transient;
- `FETCH_UNSAFE_DESTINATION` → permanent/security;
- `PROVIDER_JOB_STATUS_UNKNOWN` → reconcile;
- `EXTRACTION_SCHEMA_INVALID` → one model-correctable attempt;
- `SOURCE_ACCESS_NOT_AUTHORIZED` → permanent/policy;
- `PARSER_RESOURCE_LIMIT` → approved fallback or reject;
- `EVIDENCE_REVISION_CONFLICT` → reload/reapply;
- `VERIFICATION_UNSUPPORTED_CLAIM` → research repair, not infrastructure retry.

## Deterministic workflow skeleton

```python
async def run_research(run_id: str) -> TerminalResult:
    brief = await ensure_approved_brief(run_id)
    plan = await ensure_plan(run_id, brief)

    while True:
        state = await load_research_state(run_id)
        assert_invariants(state)

        decision = choose_next_step_by_policy(state)
        if decision.kind == "research":
            proposals = await controller.propose_actions(context_for_controller(state))
            admitted = admit_actions(proposals, state.policy, state.budget)
            await execute_actions_durably(run_id, admitted)
            continue

        if decision.kind == "verify":
            candidate = await build_candidate_from_accepted_claims(run_id)
            findings = await verify_candidate(candidate)
            await persist_findings(findings)
            if findings.pass_release_gate:
                return await render_and_publish_idempotently(candidate)
            if findings.repairable and repair_budget_remains(state):
                await open_targeted_gaps(findings)
                continue
            return terminal_from_findings(findings)

        return terminal_from_policy_decision(decision)
```

The controller proposes actions; `admit_actions` enforces permissions, overlap, budgets, deadlines, and fan-out. The loop reads durable state after each activity rather than relying on an in-memory transcript.

## Action admission

```python
def admit_actions(proposals, policy, budget):
    accepted = []
    for action in proposals:
        validate_schema(action)
        require_mapped_question(action)
        reject_tainted_egress(action)
        reject_overlapping_branch(action, accepted)
        require_capability(action, policy)
        reserve_budget_atomically(action, budget)
        accepted.append(with_operation_id(action))
        if len(accepted) == policy.max_parallel_actions:
            break
    return accepted
```

Reservations prevent two concurrent workers from each consuming the full remaining budget. Release unused reservation on terminal activity completion.

## Correction and deletion worker

```python
async def propagate_source_status(status_revision_id: str) -> None:
    status = await load_source_status(status_revision_id)
    await fence_source_for_new_use(status.source_id, status.revision)

    descendants = await enumerate_descendants(
        status.source_id,
        kinds=("representation", "span", "edge", "claim", "memory", "artifact", "destination"),
    )
    job = await ensure_propagation_job(status, descendants)

    for descendant in descendants:
        action = decide_status_action(status, descendant, current_policy())
        receipt = await execute_or_reconcile_status_action(job.id, descendant, action)
        await record_propagation_receipt(receipt)

    await require_feed_watermark_contiguous(status.provider_cursor)
    await close_only_if_every_descendant_terminal(job.id)
```

The descendant snapshot is versioned, but the worker also rechecks for newly created descendants before closure. Content erasure retains only the minimal permitted tombstone and erasure receipt. Correction work has reserved queue capacity and higher priority than optional research branches.

## Context compaction worker

```python
async def compact_context(run_id: str, from_context_id: str) -> str:
    state = await load_authoritative_state(run_id)
    await flush_pending_transitions_and_operation_ids(state)
    receipt = build_loss_aware_continuity_receipt(state, from_context_id)
    validate_required_continuity(receipt)
    assert not receipt.unresolved_loss
    await persist_receipt(receipt)

    next_context = assemble_context_from_revisions(receipt.authoritative_revisions)
    require_hash(next_context.reference_hash, receipt.to_context_reference_hash)
    return next_context.id
```

Inject compaction before and after every outcome-relevant boundary in evaluation. A summary that omits only scratch prose is acceptable; a summary that loses contradiction, approval, budget, cancellation, access, or in-flight-effect state is not.

## Configuration baseline

```yaml
research_classes:
  standard:
    max_wall_clock: 20m
    max_workers: 3
    max_delegation_depth: 1
    max_search_calls: 60
    max_fetched_documents: 100
    max_fetch_bytes: 250MB
    max_verification_repairs: 2
    required_release_checks:
      - material_claim_support
      - citation_completeness
      - quote_exactness
      - contradiction_disposition
      - freshness
      - link_safety
      - privacy
network_zones:
  public_fetch:
    schemes: [https]
    ports: [443]
    max_redirects: 3
    block_private_and_metadata_ranges: true
    credentials: none
  private_retrieval:
    arbitrary_public_egress: false
telemetry:
  capture_content: false
  capture_model_io: sampled_redacted
  trace_external_propagation: disabled
```

Store configuration revisions and effective hashes with each run.

## Evidence-to-artifact intermediate representation

Do not render directly from model Markdown. Use a validated intermediate form:

```json
{
  "artifact_id": "art_01J...",
  "revision": 2,
  "title": "Cooling-water technology comparison",
  "as_of": "2026-08-31",
  "sections": [
    {
      "heading": "Measured outcomes",
      "blocks": [
        {
          "type": "paragraph",
          "statements": [
            {
              "text": "Facility A reported a 32% reduction...",
              "claim_ids": ["clm_01J..."],
              "citation_ids": ["cit_01J..."]
            }
          ]
        }
      ]
    }
  ],
  "limitations": ["No weather-normalized counterfactual"],
  "verification_revision": "ver_01J..."
}
```

The renderer resolves citation IDs through the verified registry and escapes all untrusted text. Artifact hash is computed before approval/publication.

## Build roadmap

### Phase 0: contract and evaluation first

- Select one narrow research domain and 30–50 representative briefs.
- Define material claims, evidence bars, source policy, and reviewer rubric.
- Build a frozen source corpus with contradictions, low-quality sources, and attacks.
- Establish a manual baseline and cost/latency target.
- Choose retention and public/private zone policy.

**Exit:** stakeholders agree on what a passing artifact and bounded failure look like.

### Phase 1: single-loop evidence prototype

- Implement brief, plan, query, source, representation, evidence, claim, and artifact records.
- Qualify one search provider, SSRF-safe fetcher, HTML/PDF parser, and one paper/dataset or database adapter with capability manifests and exact result fixtures.
- Run one adaptive controller with strict budgets.
- Generate from accepted claims and apply deterministic quote/link/schema checks.
- Provide reviewer claim-to-evidence UI or export.

**Exit:** frozen-suite artifacts are reproducible; no model-generated citation bypasses the ledger.

### Phase 2: verification and security

- Add independent claim/citation/contradiction verifier and calibrated human review.
- Add public/private staged workflow, capability broker, taint/DLP checks, parser sandbox.
- Implement correction/retraction/freshness and artifact invalidation paths.
- Implement all seven memory lifetimes, governed promotion, correction/deletion lineage, and loss-aware compaction receipts.
- Run adversarial and failure-injection suites.

**Exit:** mandatory integrity/security gates pass and cannot be overridden by the researcher.

### Phase 3: durability and operations

- Put activities behind a durable workflow/queue with reconciliation and idempotency.
- Add leases/fencing, cancellation, backpressure, workload classes, SLOs, and runbooks.
- Add connector result/job reconciliation, correction/deletion queues, tenant/region enforcement, DR and recovery-load tests.
- Version prompts/models/tools/policies/parsers/renderers and support side-by-side deployments.
- Load/soak test provider quotas, parser resources, and review queues.

**Exit:** crash/retry/deployment tests show no evidence loss or duplicate publication.

### Phase 4: measured parallelism

- Add overlap-scored bounded branches for a breadth-heavy evaluation slice.
- Compare single versus multi-worker quality, cost, latency, and consistency.
- Add branch cancellation, partial result handling, and marginal-value admission.
- Keep single-loop path for dependent/narrow work.

**Exit:** parallel mode has a statistically and economically justified routing rule.

### Phase 5: refresh and production learning

- Schedule claim-specific freshness/status checks.
- Produce semantic diffs and targeted re-research.
- Sample production traces/artifacts under privacy controls.
- Turn user corrections, incidents, and invalidations into eval cases.

**Exit:** released knowledge can be maintained, not just generated once.

### Phase 6: continuous behavioral evolution at scale

- Convert reviewed corrections, invalidations, source-acquisition failures, security incidents, cost outliers, and verifier disagreements into candidate held-out cases.
- Replay old and proposed model, prompt, context/compaction, search, parser, verifier, policy, and renderer bundles against frozen and recent corpora.
- Shadow and canary releases by tenant, research class, source zone, and publication authority; preserve the prior complete bundle and artifact-reproduction path.
- Scale acquisition, parsing, verification, and human review as independently backpressured pools; add regions/providers only after rights, tenancy, failure, and recovery qualification.
- Mine reviewed failures into de-identified held-out fixtures; do not train and grade on the same incident artifacts.

**Exit:** any released artifact and decision path can be reproduced from pinned evidence and versions; regressions are attributable by component; memory/policy/schema migrations preserve deletion and provenance; and a release can be rolled back without losing correction, invalidation, or publication reconciliation.

## Framework integration boundary

An agent framework may implement model calling, tool schemas, handoffs, or tracing. Wrap it behind application interfaces:

```text
ResearchController.propose_actions(context) -> ActionProposal[]
Extractor.extract(representation, schema) -> EvidenceProposal[]
Synthesizer.compose(claim_bundle, outline) -> ArtifactIR
Verifier.evaluate(target, rubric) -> Finding[]
```

Do not expose framework-specific message/checkpoint types in the evidence schema. Do not let framework session state become the only run state. A workflow engine should call these interfaces as activities; its event history does not replace the evidence ledger.

## Initial technology decision record

Document:

- why a single loop/workflow/worker topology was chosen;
- runtime language and supported version;
- framework or direct SDK, exact version, and owned boundary;
- durable engine/queue choice and recovery semantics;
- database/object store and tenancy design;
- search/fetch/parser/browser providers and terms;
- model roles/routes/snapshots and fallback semantics;
- evidence/citation schema and retention policy;
- verifier/human review calibration;
- threat model and network zones;
- acceptance suite and rollback thresholds;
- expected cost per verified artifact.

Revisit when evidence, not fashion, changes the trade-off.

## Production acceptance checklist

### Product and evidence

- [ ] Briefs, claims, evidence bars, limitations, and terminal outcomes are domain-defined.
- [ ] Every material claim and quote is ledger-backed and independently verified.
- [ ] Contradictions, source relationships, freshness, and invalidation are implemented.
- [ ] Connector manifests and fixtures cover query, pagination, coverage, rights, access, capture, correction, and deletion semantics.
- [ ] Evidence packages reproduce the artifact or disclose every non-retained representation.

### Control and reliability

- [ ] Workflow/state transitions are durable and idempotent.
- [ ] Retry, reconciliation, deadlines, budgets, leases, and cancellation are tested.
- [ ] Old jobs survive compatible deployments and schema evolution.
- [ ] Seven memory lifetimes, promotion, correction/deletion, poisoning, and loss-aware compaction are tested.

### Security

- [ ] Public/private retrieval, parsing/compute, and publication are isolated.
- [ ] SSRF, prompt injection, exfiltration, malicious files, and memory poisoning are tested.
- [ ] Credentials and authorization remain outside model context.

### Evaluation and operations

- [ ] Frozen, live, adversarial, repeated, recovery, load, and human-review lanes pass.
- [ ] SLOs and dashboards measure verified outcomes, not call success.
- [ ] Runbooks and artifact invalidation work.
- [ ] Cost per verified artifact is within the product value envelope.
- [ ] Complete behavior bundles pass shadow, canary, rollback, drift, tenant/region, DR, and recovery-load gates.

## Research foundation

This implementation shape operationalizes the decisions and competing evidence recorded in the [deep-research agent research packet](../../research/packets/deep-research-agent-blueprint.md). In particular, it keeps adaptive model behavior inside deterministic lifecycle and policy boundaries, makes provenance durable, and treats parallel workers as a measured optimization.

Strong primary references include [W3C PROV-O](https://www.w3.org/TR/prov-o/) for provenance concepts, [Temporal workflow execution](https://docs.temporal.io/workflow-execution) for durable orchestration semantics, [OpenAI's deep research guide](https://developers.openai.com/api/docs/guides/deep-research) for a current managed/API design, and [Anthropic's multi-agent research engineering report](https://www.anthropic.com/engineering/multi-agent-research-system) for orchestrator-worker production lessons. They inform interfaces and tests; none is copied as the application domain model.

## Related local guidance

- [Architecture and stack selection](architecture-and-stack-selection.md)
- [Connectors and provider qualification](connectors-and-provider-qualification.md)
- [Research loop and source acquisition](research-loop-and-source-acquisition.md)
- [Evidence, citations, and verification](evidence-citations-and-verification.md)
- [State, context, and artifacts](state-context-and-artifacts.md)
- [Security, permissions, and isolation](security-permissions-and-isolation.md)
- [Reliability, observability, and operations](reliability-observability-and-operations.md)
- [Evaluation and acceptance testing](evaluation-and-acceptance-testing.md)
- [Worked cases, exercises, and runbooks](worked-cases-exercises-and-runbooks.md)

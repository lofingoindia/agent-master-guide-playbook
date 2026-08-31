# Serving Rollout, Health, Drift, and Rollback

## Decision

Release in increasing evidence order: isolated compatibility → replay → shadow → small canary → bounded ramp → full exposure → extended monitoring. Deterministic controllers own traffic and stop rules. The model explains evidence and proposes recovery; it does not improvise thresholds or exposure.

Drift is a diagnostic signal, not a universal release gate and never an automatic retraining command.

## Serving contract

Before any traffic, verify:

| Contract | Required evidence |
|---|---|
| Artifact | Loaded model/prompt/image digests match the release manifest |
| Interface | Input/output names, types, shapes, units, modality, errors, timeouts |
| Features | Entity keys, online feature contract, transformations, freshness, defaults |
| Runtime | Model format, framework/runtime versions, tokenizer, hardware/driver compatibility |
| Resources | CPU/GPU/accelerator type, memory, disk, warm-up, concurrency, batching |
| Health | Server and model liveness/readiness plus one semantic smoke inference |
| Telemetry | Release/variant correlation, request/error/latency/queue/resource metrics, sampling coverage |
| Privacy | Capture fields, redaction, consent/legal basis, retention, access |
| Recovery | Compatible known-good target, capacity, traffic mechanism, postconditions |

KServe V2 defines health, metadata, and inference endpoints, including server/model readiness and liveness. These prove protocol-level availability—not semantic quality. A loaded model can still be wrong, unfair, stale, or incompatible with downstream consumers.

## Serving identity and version semantics

| Object | Immutable or versioned subject | Mutable/control-plane subject | Commit and verification rule |
|---|---|---|---|
| Serving revision | Release manifest digest + provider revision/deployed-model ID + image/model/prompt/runtime/config digests | Display name, latest-ready marker, Kubernetes object name | Create with expected endpoint generation; verify loaded digests, provider operation, ready condition, semantic smoke and release attribution |
| Endpoint | Provider account/project/region + endpoint resource ID | DNS, display name, auth/rate-limit configuration, attached deployments | Treat configuration change as its own versioned effect; verify endpoint revision/etag/resourceVersion and network/auth policy |
| Deployment/variant | Provider deployment or variant ID bound to one serving revision and resource profile | Replica count, autoscaling settings, health, desired state | Never equate a model registry version with a deployment; verify provider status and at least one attributed inference |
| Traffic policy | Canonical ordered routes, weights, mirrors, cohorts/headers, priority, fallback, controller and policy digest | Current weights/mirror percentage | One writer, compare-and-set/current revision where available, observed route telemetry, and a stable effect/operation ID |
| Accelerator pool | Provider/cluster/region + node pool/resource class ID + accelerator/MIG profile + driver/device-plugin/operator versions + isolation class | Available nodes/devices, reservations, taints, quotas, autoscaler bounds | Scheduler placement and “GPU present” are insufficient; verify requested profile, memory, isolation, warm capacity, quota and rollback reserve |
| Drift signal | Detector definition/version + reference/current snapshot/window + feature/population + result artifact digest | Alert status, incident link | Signal is immutable evidence; alert acknowledgement/suppression is a separate effect and never changes the measured result |
| Rollback release | Exact known-good behavior-bundle manifest + compatibility evidence | Alias called `previous`, last-ready revision | Seal a new rollback plan/effect against current target; revalidate policy, features, runtime, capacity and monitoring before commit |

Provider-specific mappings are intentionally non-isomorphic. Vertex traffic maps `DeployedModel.id`; Azure maps named deployments and permits direct deployment routing; SageMaker updates an endpoint through endpoint configuration/fleets; KServe behavior depends on CRD and deployment mode; Seldon experiments use proportional weights; BentoCloud canaries are managed Deployment features. The adapter returns the internal objects above and retains the native resource, operation, generation, and status evidence.

## Rollout contract

```yaml
rollout:
  plan_id: plan_01K...
  release_manifest_digest: sha256:aa3...
  target_id: serving://prod-eu/fraud-risk
  expected_target_revision: rv_884102
  rollback_release_id: rel_01J...
  strategy: shadow_then_canary
  phases:
    - {id: shadow, traffic_percent: 0, sample_percent: 5, minimum_minutes: 60}
    - {id: canary_1, traffic_percent: 1, minimum_requests: 5000, minimum_minutes: 30}
    - {id: canary_10, traffic_percent: 10, minimum_requests: 50000, minimum_minutes: 120}
    - {id: full, traffic_percent: 100, minimum_minutes: 1440}
  hard_stop:
    - metric: policy_violation_count
      op: ">"
      threshold: 0
    - metric: error_rate_delta
      op: ">"
      threshold: 0.005
    - metric: p99_latency_ms
      op: ">"
      threshold: 100
  minimum_coverage:
    release_attribution: 0.995
    required_features: 0.999
  missing_data: pause
  maximum_exposure:
    requests: 100000
    affected_entities: 75000
    duration_minutes: 240
  approval:
    proposal_digest: sha256:08f...
    expires_at: 2026-08-31T18:00:00Z
```

Trusted code calculates metrics and phase eligibility. The plan records numerator, denominator, query, window, lag, baseline, minimum sample, missing-data behavior, and evaluator version for every gate. A naked number such as “error rate below 1%” is insufficient.

## Strategy selection

| Strategy | Use when | Advantage | Main failure |
|---|---|---|---|
| Offline replay | Requests/features can be reproduced safely | No user exposure | Historical data may miss current distribution and feedback |
| Shadow | Candidate can observe mirrored traffic without affecting response | Live compatibility and load evidence | Mirroring can duplicate sensitive data or downstream calls; side effects must be disabled |
| Canary | Per-request routing and variant attribution are reliable | Limits exposure and supports comparative evidence | Small samples miss rare harms; routing populations may be biased |
| Blue/green | Capacity allows two complete fleets and fast switch | Clear rollback target | Costs more; hidden state/cache/schema dependencies remain |
| A/B experiment | Product owner has a valid randomized experiment and business hypothesis | Measures causal product outcome | Not merely a release-safety mechanism; analytics/product owns interpretation |
| Scheduled cutover | Traffic split is impossible or consistency requires one version | Simple ownership | Higher instantaneous risk; needs rehearsal and rapid restore |
| Batch dual-run | Batch outputs can be compared before publish | Strong state comparison | Delays output and doubles compute/storage |

Do not call a biased, non-random canary an A/B test. Do not mirror effectful requests unless the candidate path is provably side-effect free.

## Control loop and ownership

```mermaid
sequenceDiagram
    participant W as Release workflow
    participant C as Traffic controller
    participant S as Serving platform
    participant M as Metrics/evaluation
    participant H as Human owner

    W->>C: Set exact approved phase with effect ID
    C-->>W: Operation/revision receipt
    W->>S: Verify release digest, readiness, observed traffic
    loop until minimum sample and time or stop
        S-->>M: Serving and attributed observations
        M-->>W: Versioned gate report + coverage
    end
    alt hard stop or missing critical evidence
        W->>C: Pause/abort inside envelope
        W->>H: Evidence package and recovery proposal
    else phase passes
        W->>C: Advance only if next phase pre-authorized
    else inconclusive
        W->>H: Hold for decision; never coerce to pass
    end
```

Use a single traffic writer. If a managed endpoint owns the rollout, the agent adapter observes and controls that provider operation. If Argo Rollouts owns it, do not simultaneously patch KServe/mesh weights elsewhere. Argo Rollouts documents analysis outcomes that can abort and `Inconclusive` cases intended for human judgment; map them explicitly.

## Signal hierarchy

| Signal class | What it says | It does not prove |
|---|---|---|
| Liveness/readiness | Process/model can answer protocol checks | Correct predictions or sufficient capacity |
| Service health | Errors, latency, queueing, saturation, cold starts | Model usefulness |
| Contract health | Inputs/outputs/features conform and are fresh | Distribution or label quality |
| Data quality | Missingness, ranges, categories, schema, duplicates | Predictive performance |
| Drift/skew | Reference and current distributions differ | Harm, root cause, or need to retrain |
| Prediction behavior | Output distribution/calibration proxy changed | Ground-truth accuracy without valid labels |
| Delayed-label quality | Outcomes match later truth under join/coverage rules | Causal business impact |
| Safety/fairness | Declared risk slices and violation signals | Compliance outside the tested population/use |
| Business outcome | Product/service metric changed | That model release caused the change without experiment/design |

Release gates normally use service/contract health immediately, carefully chosen proxy signals during early exposure, and delayed-label/business evidence when available. High-risk systems may require human review before expansion even if automated signals pass.

## Drift and skew taxonomy

| Class | Comparison | Example | Default response |
|---|---|---|---|
| Schema skew | Expected vs observed contract | Category/type/shape changed | Stop or quarantine; route to DataOps/producer |
| Training-serving feature skew | Same example through training vs serving path | Encoding/default differs | Stop; repair feature/serving path |
| Data-quality degradation | Current input vs validity rules | Missingness or freshness breach | Contain; investigate producer/pipeline |
| Covariate drift | Input distribution reference vs current | New device/geography mix | Investigate slices and quality; no automatic retrain |
| Prediction drift | Output distribution reference vs current | Score distribution shifts | Check input mix, model/feature/runtime, labels |
| Concept/label drift | Relationship between inputs and valid outcomes changes | Fraud pattern changes | Needs delayed labels/domain analysis and owner decision |
| Feature-attribution drift | Explanation/importance distribution changes | Model relies on different features | Diagnostic; explanations and sampling have limitations |
| Policy/safety drift | Violation, refusal, protected-slice behavior changes | LLM prompt/provider update | Pause or rollback by severity; expert review |
| Feedback-loop shift | Model affects future inputs/labels | Recommendations change user behavior | Product/data science investigation; controlled experiment |

Training-serving skew is often an engineering defect, while population drift may be a real-world change. Treating both as “retrain” hides the correct owner and remedy.

## Baseline lifecycle

Every detector declares:

```yaml
detector:
  id: feature-drift-v5
  feature: transaction_amount_log
  method: population_stability_index
  reference:
    dataset_id: observation://fraud/champion-183/2026-07
    digest: sha256:...
    population: eligible_prod_eu
  current:
    window: PT24H
    minimum_n: 10000
  threshold:
    warn: 0.15
    page: 0.25
    owner: fraud_model_owner
    justification_ref: decision://drift-threshold/42
  missing_data: alert_monitor_failure
  detector_image_digest: sha256:...
```

No detector method or threshold is universal. Validate sensitivity, false alarms, seasonality, multiple comparisons, sampling, categorical cardinality, and operational actionability. Baseline changes are policy changes: version, review, backtest, and deploy them separately from the candidate.

## Delayed labels and coverage

Quality monitoring must record the prediction-to-label join contract:

- stable prediction/entity/event IDs;
- eligible label population and exclusion reasons;
- event time versus processing/arrival time;
- maximum expected label delay and late-update policy;
- duplicate/corrected label handling;
- release/variant attribution at prediction time;
- coverage numerator/denominator by slice;
- possible selective-label and intervention bias.

```text
quality_status = UNKNOWN
until label_lag_window_elapsed
and label_coverage >= required_coverage
and join_error_rate <= limit
```

Do not count unlabeled cases as correct, drop late adverse outcomes, or compare variants with different label maturity.

Offline and online metrics are not interchangeable even when they share a name. The online reducer must preserve the prediction-time release, population, preprocessing, decision threshold, label maturity, exclusions, sample weights, and correction policy used by the offline evaluator. Diagnose separately:

- **definition mismatch:** different metric formula, positive class, threshold, slice, or aggregation;
- **population mismatch:** canary routing, eligibility, selective intervention, or survivorship changes the examples;
- **feature mismatch:** online values differ from point-in-time training/evaluation retrieval;
- **label mismatch:** partial, delayed, censored, duplicated, or corrected outcomes;
- **runtime mismatch:** numerical precision, tokenizer, preprocessing, batching, or dependency behavior;
- **coverage mismatch:** observability drops or join failures change the denominator.

Only compare after a deterministic contract check. A model may explain the residual gap and propose targeted reads; it cannot choose the more favorable metric as truth.

## Serving and accelerator capacity

Measure end-to-end and internal demand:

- request rate, batch size, concurrency, queue time, cold-start/load time;
- input/output bytes or tokens and sequence lengths;
- preprocessing, model compute, postprocessing, and network latency;
- CPU, memory, GPU memory/utilization/power, model instance count, cache behavior;
- throttling, OOM/restart, scheduler placement, node/accelerator availability;
- cost per verified prediction and reserved failover capacity.

Triton distinguishes server queue, compute input, inference, output, and GPU metrics. KServe notes runtime metrics are not unified. Normalize names and units without discarding the native evidence.

GPU time-slicing is not strong isolation: NVIDIA documents that it lacks the memory and fault isolation provided by MIG. Do not place mutually untrusted high-risk tenants together merely because Kubernetes exposes several logical GPU replicas. Platform owners decide MIG/time-slicing/dedicated capacity; the MLOps agent consumes capacity profiles and refuses unsafe target classes.

## Rollback decision

Rollback is one controlled release, not an undo button.

| Condition | Preferred action |
|---|---|
| Candidate service health fails before traffic | Remove candidate revision; no live rollback needed |
| Canary regresses and champion remains compatible/healthy | Pause, revalidate target, route back under approved envelope |
| Shared feature/data path is broken | Rolling back model may not help; contain input and engage DataOps |
| Current champion is also unhealthy | Do not oscillate; incident commander chooses fail-safe/degraded mode |
| Candidate created downstream irreversible effects | Stop exposure, preserve receipts, compensate/notify where possible |
| Old model incompatible with current schema/runtime | Roll forward or restore the coupled feature/runtime bundle |
| Policy/safety severe event | Immediate pause/abort if pre-authorized; human approves further rollback/recovery |
| Monitoring is blind | Pause progression; do not treat absence of alarms as health |

### Rollback manifest

```yaml
rollback:
  rollback_id: rb_01K...
  from_release: rel_candidate
  to_release: rel_champion
  reason_event_refs: [gate://canary/failed/01K...]
  target_revision_before: rv_884109
  compatibility_evidence_ref: artifact://rollback-compat/...
  expected_effects:
    traffic: {candidate: 0, champion: 100}
    registry_alias: champion_unchanged
  irreversible_consequences:
    requests_already_served: 8421
    downstream_notifications_required: true
  approval_id: apr_01K...
  expires_at: 2026-08-31T18:15:00Z
```

High-impact rollback requires explicit human policy and approval. An automatic rollback can be pre-authorized only when target, maximum exposure, compatibility, health checks, and postconditions are sealed in advance.

## Hotfix and recovery policy

A hotfix is a new behavior-bundle release, even when it changes only a preprocessor, feature default, prompt, container library, GPU profile, or traffic rule. It may use an expedited approval path only when policy predefines the incident class, responsible roles, evidence minimum, exposure envelope, expiry, and retrospective review. Never patch bytes in place or reuse a release ID.

Prefer recovery in this order: stop additional exposure; identify actual external state; restore a prequalified compatible bundle; roll forward with a narrowly scoped hotfix when rollback is incompatible; then reconcile registry, serving, traffic, evidence, and incident projections. Preserve the failed release for audit and reproduction, but revoke it from new deployment.

## Worked recovery walkthroughs

### Bad-model promotion stopped during canary

1. `rel-184` passes aggregate AUPRC but the `new_accounts` slice has only 38 mature labels; its gate is `UNKNOWN`. The model suggests waiting and identifies the missing evidence; deterministic policy refuses promotion.
2. After labels mature, a 1% canary starts under `fx-traffic-1`. Readiness and latency pass, but a hidden safety case appears once and the non-compensating gate fails.
3. The controller pauses within the approved containment envelope. The agent assembles the exact prediction, release, feature, policy, evaluator, coverage, and receipt references; it does not debate the hard stop.
4. The on-call verifies champion health and compatible features, approves traffic restoration, and the gateway commits a new rollback effect. Registry metadata is reconciled only after observed traffic is 100% champion.
5. Exit evidence includes exposure count, affected-entity list under privacy controls, rollback receipt, safety-case reproducer, candidate revocation, owner decision, and a regression-suite change request.

### Feature skew isolated without model rollback

1. Online recall falls while candidate and champion both show a spike in missing `merchant_age_days`; service health remains green.
2. The parity job compares identical entity/event pairs and finds training retrieval uses transformation digest `f13`, while online materialization serves `f12` with a zero default. This is direct skew evidence, not population drift.
3. Traffic progression freezes. Rolling back the model would retain the shared bad feature path, so the agent routes the incident to the feature/DataOps owner and proposes safe input containment.
4. DataOps restores materialization `f13`, backfills the affected window, and emits a new watermark. Deterministic parity, freshness, semantic smoke, and replay gates pass before rollout resumes.
5. The incident records affected releases and predictions, feature definition/materialization identities, parity artifact, correction scope, delayed-label re-evaluation, and a rollback-compatibility update.

### GPU exhaustion contained without sacrificing the champion

1. Candidate loading consumes the pool's last unreserved GPU; pending pods, model-load time, queue delay, memory pressure and OOM restarts rise. The endpoint remains superficially ready.
2. Admission had reserved champion/rollback capacity, so the controller stops the ramp, routes new candidate requests away, and leaves the champion fleet intact. It does not scale down rollback reserve or fan out retries.
3. The serving adapter verifies accelerator profile, node-pool quota, pending-pod reasons, device-plugin/MIG mode, provider autoscaler state and per-variant queue metrics. The agent distinguishes quota, placement, cold-start and model-memory hypotheses.
4. Platform operations either add an approved pool/profile or reject the candidate resource contract. The load suite must repeat through cold start, failure, recovery surge and rollback before any new canary.
5. Exit evidence includes peak/recovery demand, unschedulable duration, OOM count, preserved champion headroom, cost, scaling-operation receipts, and a corrected capacity profile.

### Incompatible rollback becomes a roll-forward hotfix

1. A candidate introduces schema v14 and downstream consumers begin relying on a new output field. The previous release requires feature contract v12, already retired from the online path.
2. A severe candidate defect triggers recovery, but compatibility gates reject the named `previous` alias: it would fail current input features and downstream schema. No automatic rollback occurs.
3. Incident command selects an expedited roll-forward: the same candidate weights plus corrected postprocessor, new image digest, new release/plan IDs, bounded replay/safety/load evidence, and explicit approval.
4. Traffic remains paused or on the least harmful compatible path. The gateway deploys the hotfix at dark/isolated capacity, verifies semantics, then shifts traffic under a shortened but sealed canary.
5. Final evidence links the rejected rollback, compatibility proof, hotfix manifest, approval, traffic/effect receipts, irreversible prior outputs, consumer notifications, and follow-up to restore a genuinely compatible rollback bundle.

## Failure matrix

| Failure | Detection | Containment | Recovery/evidence |
|---|---|---|---|
| Candidate loads but returns wrong shape | Semantic smoke/contract test | No traffic | Mark incompatible; artifact and trace |
| Traffic effect times out | Operation state unknown | Freeze next step | Query controller and observed traffic by effect/revision ID |
| Metrics lag or disappear | Coverage/freshness gate | Pause progression | Restore pipeline; replay gap if authoritative source exists |
| Canary has too few rare-slice events | Sample/coverage gate inconclusive | Hold exposure | Extend within approved budget or require expert decision |
| Feature values stale | Feature freshness/contract alarm | Pause/route safe default | DataOps repair; re-evaluate after fresh evidence |
| GPU saturation causes tail latency | Queue/GPU/restart signals | Stop ramp; shed/defer | Platform capacity action or profile change; rerun load suite |
| Registry says champion but endpoint serves candidate | Reconciliation mismatch | Freeze alias/traffic writes | Determine actual serving receipt; repair one authoritative mapping |
| Rollback target fails to load | Preflight should catch; live health detects | Keep traffic on least harmful healthy path | Incident command; roll forward/degraded mode |
| Labels arrive selectively | Coverage/slice bias checks | Mark quality unknown | Repair label process; human risk decision |
| Drift alert storm | Detector health and correlated changes | Rate-limit alerts; do not auto-retrain | Root-cause baseline/data/seasonality; owner reviews thresholds |

## Failure-injection program

- Drop the traffic-controller response after it commits; no blind retry.
- Send 10% traffic to the candidate while the control plane reports 1%; observed traffic must win and page.
- Make readiness green while semantic smoke returns swapped labels; rollout must stop.
- Delay metrics beyond the phase freshness budget; progression pauses.
- Inject a drift alert with 50 samples below `minimum_n`; mark insufficient evidence.
- Deliver labels late and out of order; versioned join produces correct coverage and corrections.
- Exhaust GPU memory under canary; stop before stable capacity is displaced.
- Remove old feature definition before rollback; compatibility gate blocks unsafe rollback.
- Fail the registry alias update after full serving promotion; reconcile without redeploying.
- Create a severe safety event under a small canary; kill switch works without the model or queue.

## Selected sources

- [KServe Inference Protocol V2](https://kserve.github.io/website/docs/concepts/architecture/data-plane/v2-protocol)
- [KServe control-plane API](https://kserve.github.io/website/docs/reference/crd-api)
- [KServe Prometheus metrics](https://kserve.github.io/website/docs/next/model-serving/predictive-inference/observability/prometheus-metrics)
- [Argo Rollouts canary](https://argo-rollouts.readthedocs.io/en/stable/features/canary/)
- [Argo Rollouts analysis](https://argo-rollouts.readthedocs.io/en/stable/features/analysis/)
- [SageMaker deployment guardrails](https://docs.aws.amazon.com/sagemaker/latest/dg/deployment-guardrails.html)
- [Google production ML monitoring](https://developers.google.com/machine-learning/crash-course/production-ml-systems/monitoring)
- [NVIDIA Triton Model Analyzer metrics](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/model_analyzer/docs/metrics.html)
- [NVIDIA GPU sharing and isolation](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/gpu-sharing.html)

## Related guides

- [Scaling, capacity, and SLOs](../../operations/scaling-capacity-and-slos.md)
- [Observability, evaluation, failure injection, and incidents](07-observability-evaluation-failure-injection-and-incidents.md)
- [Artifacts, registry, lineage, evaluation, and promotion](03-artifacts-registry-lineage-evaluation-and-promotion.md)

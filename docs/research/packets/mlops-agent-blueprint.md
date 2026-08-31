# Research Packet: MLOps and Model Operations Agent Blueprint

> **Status:** Active Pass-2 research packet  
> **Research date:** 2026-08-31  
> **Scope:** Model/prompt/evaluation artifact promotion, registry lineage, serving health, drift evidence, rollback, and controlled release operations  
> **Derived guides:** [MLOps and model operations agent blueprint](../../agents/mlops-model-operations-agent/README.md)  
> **Method:** Current official documentation, standards/specifications, official repositories/releases, regulator material, and foundational production-ML papers were compared with the repository's canonical runtime, state, tool, context/memory, security, reliability, evaluation, and operations guidance. Vendor product statements were treated as mechanisms, not end-to-end guarantees.

## Promotion record

| Registry gate | Evidence | Result |
|---|---|---|
| Real-agent fit | Candidate triage and rollout diagnosis require bounded selection across registry, lineage, evaluation, serving, and monitoring evidence | Pass |
| Distinct architecture | Behavioral artifact/data coupling, delayed labels, drift, model serving, and rollback compatibility differ from ordinary DevOps | Pass |
| Buildability | Small read/propose loop, typed adapters, release ledger, evaluation runner, and one serving target form a credible MVP | Pass |
| Production depth | Artifact code execution, mutable aliases, training-serving skew, unknown traffic effects, label delay, and unsafe rollback are material unique failures | Pass |
| Evaluation viability | External registry/serving state, evaluation artifacts, trajectory invariants, and failure injection provide observable outcomes | Pass |
| Evidence depth | Broad primary/official/foundational source register plus 15 canonical repository guides; volatile product surfaces rechecked for Pass 2 | Pass |
| Reader value | Removes repeated decisions about artifact identity, registry choice, evaluation binding, rollout, drift, and authority | Pass |

**Decision:** retain as a distinct blueprint and promote to Pass-2 production/usefulness depth in `docs/agents/mlops-model-operations-agent/`.

## Category boundary

### Owned

- immutable model, prompt, evaluator, serving, and release manifests;
- registry versions/aliases/tags and deployment binding;
- source/training/data/feature/evaluation lineage evidence;
- candidate/champion evaluation and promotion gates;
- online serving compatibility, shadow/canary control, and health evidence;
- data/skew/drift/delayed-label signals and rollback evidence;
- controlled pause, abort, promotion, and rollback operations.

### Not owned

| Adjacent blueprint/owner | Boundary |
|---|---|
| Data pipeline and DataOps | Builds/repairs ingestion, transformation, schema evolution, backfills, materialization, and data-quality quarantine |
| Infrastructure operations | Provisions clusters, networks, accelerators, storage, secrets systems, and platform desired state |
| DevOps/deployment | Builds/releases ordinary application artifacts and infrastructure configuration; MLOps owns behavioral model bundle/data coupling |
| Analytics/product | Defines and interprets business metrics, causal experiments, and product outcomes |
| SRE incident response | Commands cross-service live incidents and emergency mitigation |
| Model/data/risk owners | Decide retraining, intended use, label/population changes, thresholds, and high-impact risk acceptance |

Training-triggered releases and high-impact rollback require explicit human policy and approval. A drift alert does not grant either.

## Research questions

1. When is a deterministic model release pipeline better than an agent?
2. What is the immutable subject of evaluation, approval, promotion, and rollback?
3. Which registry, lineage, feature, evaluation, serving, monitoring, and rollout mechanisms are credible defaults?
4. How should prompt artifacts and foundation-model/provider changes join classic ML release management?
5. What do readiness, service health, data drift, model quality, safety, and business outcomes each prove?
6. How should delayed labels, selective labels, and feedback loops constrain automation?
7. Which production effects can be pre-authorized and which require a fresh named human decision?
8. How are duplicate, stale, timed-out, partial, and externally committed effects reconciled?
9. What context and memory classes are useful without poisoning authority or preserving sensitive raw data?
10. How should the blueprint progress from deterministic baseline to multi-tenant continuous evolution?

## Baseline and volatility

| Domain | Baseline used | Volatility/qualification |
|---|---|---|
| Registry | Current MLflow docs plus managed AWS/GCP/Azure registry docs | API, auth, aliases, approval, workspace, and lifecycle behavior change; qualify installed version |
| Artifact integrity | SLSA v1.2, in-toto Statement model, OCI image/distribution artifact model, Sigstore verification | Artifact media types, registry referrers, signer policy, and tooling conformance must be tested |
| Lineage | OpenLineage current spec/docs and extensible facets | Events can be incomplete or declared; standard envelope is not proof of completeness |
| Feature metadata | Feast current architecture and point-in-time join docs | Store/provider feature parity and community integrations vary |
| Serving | KServe current data/control-plane docs, Argo Rollouts stable docs, managed endpoint docs | CRDs, rollout fields, traffic providers, exclusions, and runtime metrics change rapidly |
| Monitoring | Provider docs, Google production-ML guidance, NIST lifecycle risk guidance | Detector methods/thresholds are domain-specific; some managed products have availability changes |
| Telemetry | OpenTelemetry GenAI registry as observed on research date | GenAI semantic conventions remain development-stage and content fields are sensitive |
| Agent model lifecycle | Provider backward-compatibility/lifecycle docs | Model behavior, availability, price, region, quota, and EOL timelines are volatile |
| Governance | NIST AI RMF 1.0/AIRC and EU AI Act official text | Apply law only after jurisdiction/use-case analysis; NIST says AI RMF is under revision |

No product version is hard-coded as a universal requirement. Implementation teams must pin exact versions and repeat qualification.

## Pass-2 research access and qualification log

All sources in this packet were re-accessed on **2026-08-31** unless an entry explicitly says otherwise. “Observed” means the official release page or documentation exposed that state during research; it is not a promise that a region, distribution, managed tier, or installed estate has it. Implementation must record the installed API/CRD/SDK/chart/image digest, region, tier, feature flags, and contract-test result.

| Surface and primary source | Version/status observed | Material behavior confirmed | Qualification or contradiction |
|---|---|---|---|
| [MLflow releases](https://github.com/mlflow/mlflow/releases), [registry workflows](https://mlflow.org/docs/latest/ml/model-registry/workflow/), [classic evaluation](https://mlflow.org/docs/latest/ml/evaluation/) | 3.15.2; registry stages deprecated since 2.9 | Aliases can be reassigned; classic and GenAI evaluator object systems are non-interoperable | Pin server/client/schema and backend; alias/tag/approval metadata remains mutable projection, not release truth |
| [MLflow tracking](https://mlflow.org/docs/latest/ml/tracking/), [evaluation datasets](https://mlflow.org/docs/latest/genai/datasets/) | MLflow 3 current docs | Runs, experiments, and Logged Models are separate; evaluation datasets require SQL backend | Run success and artifact URI do not prove immutable bytes or full environment; retain native IDs plus canonical digests |
| [Kubeflow Pipelines releases](https://github.com/kubeflow/pipelines/releases), [runs](https://www.kubeflow.org/docs/components/pipelines/concepts/run/), [artifacts](https://www.kubeflow.org/docs/components/pipelines/user-guides/data-handling/artifacts/) | Backend/SDK 2.17.0; workspace feature documented from 2.15.0 | Experiment groups arbitrary runs; run executes pipeline; artifact wrapper exposes URI/metadata/path | Docs describe runs as immutable logs, but object bytes/URIs, cache inputs, pipeline IR and image tags still need digest binding |
| [Kubeflow Hub REST API](https://www.kubeflow.org/docs/components/hub/reference/rest-api/), [architecture](https://www.kubeflow.org/docs/components/hub/reference/architecture/), [installation](https://www.kubeflow.org/docs/components/hub/installation/) | Hub 0.3.14, REST `v1alpha3`, opt-in alpha component | Registry stores model/version/artifact metadata and is explicitly passive, not an orchestration control plane | Alpha API and distribution-specific authentication; referenced artifact bytes need independent verification |
| [KServe release policy](https://github.com/kserve/kserve/blob/master/RELEASES.md), [CRD API](https://kserve.github.io/website/docs/reference/crd-api), [control plane](https://kserve.github.io/website/docs/concepts/architecture/control-plane) | Pre-1.0; policy snapshot listed 0.19 active with latest two minors supported | Minor bumps may break; `CanarySpec` and legacy component canary fields/modes differ; scale-to-zero is Knative-only | Official release-policy, indexed release, and projected schedule surfaces were not perfectly synchronized; installed CRDs/chart/image and sandbox behavior decide |
| [Seldon Core releases](https://github.com/SeldonIO/seldon-core/releases), [installation dependencies](https://docs.seldon.ai/seldon-core-2/installation/installation), [experiments](https://docs.seldon.ai/seldon-core-2/user-guide/experiment) | Core 2.10.2 | Experiment weights are proportional; mirror and sticky-route exist; stateful replica stickiness is not guaranteed | Kafka is required only for dataflow pipelines; supported dependency range and Business Source License require local qualification |
| [BentoML releases](https://github.com/bentoml/BentoML/releases), [BentoCloud canaries](https://docs.bentoml.com/en/latest/scale-with-bentocloud/deployment/canary-deployments.html), [autoscaling](https://docs.bentoml.com/en/latest/scale-with-bentocloud/scaling/autoscaling.html) | BentoML 1.4.39 | Managed deployment supports version traffic splits and concurrency-based autoscaling | Canary/autoscaling pages describe BentoCloud, not guaranteed OSS self-hosting behavior; keep framework and managed control-plane adapters separate |
| [SageMaker registry objects](https://docs.aws.amazon.com/sagemaker/latest/dg/model-registry-models.html), [approval](https://docs.aws.amazon.com/sagemaker/latest/dg/model-registry-approve.html), [guardrails](https://docs.aws.amazon.com/sagemaker/latest/dg/deployment-guardrails.html) | Current managed docs | Model Package Group/version ARN and approval state differ from Endpoint/EndpointConfig/update; approval may trigger CI/CD | Guardrails apply only to real-time and asynchronous endpoints and have exclusions; CloudWatch alarms and baking mechanics do not prove model quality |
| [SageMaker canary shifting](https://docs.aws.amazon.com/sagemaker/latest/dg/deployment-guardrails-blue-green-canary.html) | Current managed docs | Canary is at most 50% of green-fleet capacity; alarms can initiate provider rollback | Provider rollback restores fleet routing mechanics, not feature/runtime/policy compatibility or already-consumed outputs |
| [Vertex model aliases](https://docs.cloud.google.com/vertex-ai/docs/model-registry/model-alias), [deploy API sample](https://docs.cloud.google.com/vertex-ai/docs/samples/aiplatform-deploy-model-sample), [deploy CLI](https://docs.cloud.google.com/sdk/gcloud/reference/ai/endpoints/deploy-model) | Current v1/CLI docs | Alias/default is mutable; Endpoint traffic maps `DeployedModel.id` and totals 100 or is empty; deployment is long-running | Resolve model version and verify returned deployed-model/model-version IDs; do not approve default alias or display name |
| [Azure safe rollout](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-safely-rollout-online-endpoints?view=azureml-api-2), [online endpoints](https://learn.microsoft.com/en-us/azure/machine-learning/concept-endpoints-online?view=azureml-api-2) | CLI `ml` v2 / Python SDK v2 current surfaces | Named deployments receive weighted live traffic; direct deployment routing can bypass split | Mirror is managed-online-only, one target, max 50%; old SDK/CLI endpoint update can erase mirror configuration |
| [Feast point-in-time joins](https://docs.feast.dev/getting-started/concepts/point-in-time-joins), [architecture](https://docs.feast.dev/getting-started/architecture/overview) | Current docs; deployed version not inferred | Historical retrieval scans backward from entity event time within feature-view TTL | Bind feature repo commit/registry export, definitions, source and materialization evidence; store name alone is not immutable parity proof |
| [SageMaker Feature Store concepts](https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store-concepts.html), [offline format](https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store-offline.html), [schema update](https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store-update-feature-group.html) | Current managed docs | Record ID + event time identifies records; online keeps latest event-time record; offline retains append history; feature group schema can add but not remove fields | Offline writes may be delayed; definition, event-time, ingest/write-time and online/offline parity must be explicit release evidence |
| [Azure managed feature store](https://learn.microsoft.com/en-us/azure/machine-learning/concept-what-is-managed-feature-store?view=azureml-api-2), [materialization](https://learn.microsoft.com/en-us/azure/machine-learning/feature-set-materialization-concepts?view=azureml-api-2) | Current docs | Feature-set versions are immutable; materialization computes a declared window into online/offline stores | Bind source/transformation version, job/window and failed/late ranges; on-demand and materialized retrieval are not automatically identical |
| [DVC releases](https://github.com/iterative/dvc/releases), [command workflow](https://dvc.org/doc/command-reference/) | 3.67.1 | Git revision and DVC metadata identify content hashes stored in cache/remote | Remote garbage collection/retrievability is independent of Git history; protect release data through audit/rollback horizon |
| [lakeFS internals](https://docs.lakefs.io/concepts/internals/) | Current docs | Commits are immutable content-derived checkpoints; branches and refs are mutable | Garbage collection affects bytes; approve commit ID and object checksum, not a branch or abbreviated ref without collision check |
| [Apache Iceberg branching](https://iceberg.apache.org/docs/latest/branching/), [spec snapshot references](https://iceberg.apache.org/spec/#snapshot-references) | Current release/spec docs | Snapshot ID is immutable; branches/tags are references with retention; branch/tag reads can use different schema semantics | Preserve snapshot and schema IDs; `expire_snapshots` can remove reproducibility unless tags/retention protect it |
| [Delta Lake releases](https://docs.delta.io/releases/), [time travel](https://docs.delta.io/delta-batch/), [`VACUUM`](https://docs.delta.io/delta-utility/) | 4.0.x with Spark 4.0.x compatibility documented | Table version supports time travel while log/data files remain | Timestamp travel can break after copying; log cleanup/`VACUUM` can remove history; retention must exceed longest reader, audit, label and rollback need |
| [Kubernetes HPA](https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/), [node autoscaling](https://kubernetes.io/docs/concepts/cluster-administration/node-autoscaling/) | `autoscaling/v2`; scale-to-zero beta in Kubernetes 1.37 current docs | HPA acts on observed resource/custom/external metrics; node autoscaler acts on pending-pod constraints | Replica demand does not create quota/device capacity; readiness/missing metrics/stabilization/cold start require joint workload/node tests |
| [NVIDIA GPU sharing](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/gpu-sharing.html) | Current GPU Operator docs | Time-slicing multiplexes a GPU; MIG provides memory/fault isolation | Time-slicing has no memory/fault isolation and limits per-container metric attribution; requested logical replicas do not guarantee proportional compute |
| [Evidently releases](https://github.com/evidentlyai/evidently/releases), [drift algorithm](https://docs.evidentlyai.com/metrics/explainer_drift) | 0.7.21 observed | Default method/threshold changes by data type, cardinality and reference sample size; nulls need separate tests | Pin detector code/config and validate on domain data; default `drift_share` or p-value/distance is not a universal release gate |
| [Phoenix datasets](https://arize.com/docs/phoenix/learn/datasets-and-experiments/datasets-concepts), [evaluations](https://arize.com/docs/phoenix/evaluation/evals) | Current docs | Dataset changes are versioned; experiments bind dataset/task/evaluators; code and model judges are supported | Treat judge output/traces as evidence, calibrate against humans, pin server/SDK/judge, and govern captured content |
| [OpenTelemetry semantic-convention releases](https://github.com/open-telemetry/semantic-conventions/releases), [GenAI attributes](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/) | Semantic conventions 1.44.0 observed | GenAI conventions moved repositories; multiple attributes remain development-stage and older content fields are deprecated | Maintain an internal stable mapping and metadata-first privacy policy; do not let telemetry schema changes break audit or SLO queries |
| [Prometheus instrumentation guidance](https://prometheus.io/docs/practices/instrumentation/) | Current docs | Each label set is a time series; high cardinality has material cost | Put run/release/artifact identity in traces/logs/lookup storage or bounded labels, not unbounded metric dimensions |

## Finding 1 — the promotion subject is a release bundle, not a model file

Classic software promotion already requires source, build, digest, provenance, and environment evidence. ML adds behavioral dependencies: training/evaluation data, feature contract, preprocessing, model format, tokenizer/prompt, evaluator, monitoring baseline, and intended use. The ML Test Score explicitly treats a trained model like a production binary that needs canary, rollback, and monitoring, while Hidden Technical Debt in ML Systems describes entanglement, hidden feedback loops, undeclared consumers, data dependencies, and configuration debt.

**Stable conclusion:** evaluation, approval, serving, and rollback bind to one canonical manifest digest. A good model metric or registry version alone is incomplete.

## Finding 2 — registry aliases are useful mutable pointers, not integrity or approval boundaries

MLflow, Vertex AI, and other registries expose aliases as mutable named references. MLflow currently deprecates model stages and recommends aliases/tags plus access-controlled environment separation. Managed platforms also expose approval/status mechanisms, but those fields can trigger automation and differ by provider.

**Stable conclusion:** resolve aliases to immutable version plus artifact digest before evaluation or approval. Store gate and approval truth in an application-owned release ledger. Use provider approval fields as adapter effects/projections, not the universal policy model.

## Finding 3 — lineage requires evidence strength and completeness, not only a standard event

OpenLineage defines job, run, dataset, and extensible facet metadata. Feast documents point-in-time-correct historical joins and separates feature serving from external batch/stream transformation engines. These are valuable interoperable mechanisms, but neither proves that every dependency was captured or that declared metadata is truthful.

**Stable conclusion:** materialize a release-specific graph of code, run, dataset, label, feature, model, prompt, evaluation, deployment, and observation edges. Label edges direct, declared, inferred, or human-attested; require DataOps/feature owners to repair missing lineage.

## Finding 4 — evaluation must bind inputs, graders, coverage, uncertainty, and hard gates

MLflow currently separates classic ML evaluation and GenAI evaluation systems and documents that their metric/scorer objects are not interoperable. This is a useful warning against one universal evaluator API. Model Cards and Datasheets support documenting intended use, evaluated populations, performance, and dataset construction. NIST emphasizes lifecycle testing and monitoring; the EU AI Act adds use-case-specific obligations for high-risk systems.

**Stable conclusion:** one evaluation bundle pins dataset snapshots, evaluator code/image, metric definitions, judge model/rubric, policy, coverage/exclusions, slice results, uncertainty, and the candidate manifest. Safety, authority, artifact integrity, and mandatory slice gates are non-compensating.

## Finding 5 — drift is a hypothesis trigger, not proof of degradation or permission to retrain

Provider monitoring products distinguish data quality, model quality, bias, feature attribution, and other signals. Google guidance distinguishes training-serving skew from population changes and notes live quality is hard without labels. AWS documents baseline/constraint workflows, label delay, and monitoring types—but current AWS documentation also says SageMaker Model Monitor is no longer open to new customers and is not receiving new features. That makes it unsuitable as a universal new-build recommendation.

**Stable conclusion:** version detector, baseline, population, window, sample size, threshold, coverage, and owner. Classify schema/feature skew, data quality, input/prediction drift, delayed-label quality, safety, and business outcomes separately. Drift produces investigation and an owner decision, not automatic retraining.

## Finding 6 — protocol health and standard inference APIs do not prove model health

KServe V2 standardizes server/model health, metadata, and inference endpoints. KServe also notes that supported serving runtimes do not expose one unified metric set. Triton separates queue, compute input, inference, output, and GPU resource metrics. These support useful service diagnosis without claiming semantic quality.

**Stable conclusion:** serving readiness requires digest/load verification, semantic smoke inference, contract and feature checks, resource profile, telemetry coverage, and rollback readiness. Service, contract, data, quality, safety, and business signals remain separate.

## Finding 7 — progressive delivery needs deterministic gates and one traffic writer

KServe and managed endpoints provide traffic-splitting/canary mechanisms. Argo Rollouts supports canary/blue-green, analysis, abort, and inconclusive outcomes for human judgment. Provider mechanisms have different exclusions and rollback semantics.

**Stable conclusion:** use one authoritative traffic controller, seal weights/windows/samples/coverage/missing-data/stop behavior, and advance deterministically inside an approved envelope. The agent proposes and diagnoses. Do not let two controllers patch the same traffic state.

## Finding 8 — rollback is a new controlled effect and may be unsafe

The previous model may depend on an obsolete feature schema, serving runtime, tokenizer, or policy; shared upstream failures may affect both releases; already-served predictions may have irreversible downstream consequences. Argo Rollouts' fast rollback window and provider auto-rollback features optimize mechanics, not semantic compatibility.

**Stable conclusion:** preflight a rollback bundle, retain capacity, bind exact target/effect, disclose irreversible exposure, and independently verify live traffic and model identity. High-impact rollback remains explicitly human-approved; automatic rollback is allowed only inside a pre-approved low-risk envelope.

## Finding 9 — model artifacts are an executable supply-chain boundary

PyTorch warns never to load untrusted data through `torch.load`; `weights_only` narrows but does not eliminate denial-of-service or memory-safety risks. scikit-learn documents that pickle/joblib/cloudpickle loading can execute arbitrary code and that version portability is unsupported. Even formats considered safer can consume unexpected memory/compute.

**Stable conclusion:** verify digest/signature/provenance first, allowlist formats, scan, load in an isolated no-secret/no-egress resource-bounded verifier, and reproduce the serving environment. The control-plane process never loads candidate model code.

## Finding 10 — a hybrid deterministic workflow with one bounded loop is the practical default

The workload has long evaluation jobs, approval waits, bake windows, label delay, and ambiguous external effects. Framework tool calling is useful, but release truth and effect safety need application-owned state, IDs, receipts, fencing, and reconciliation.

**Stable conclusion:** fixed outer workflow plus one model loop for evidence selection/diagnosis/proposal. Use a durable workflow when waits outlive a process. Reject multi-agent committees and autonomous retraining until a workload-specific evaluation proves value and preserves authority.

## Finding 11 — accelerator utilization and tenant isolation are different goals

NVIDIA documents that GPU time-slicing offers shared utilization without MIG's memory/fault isolation. KServe documents concurrency-driven autoscaling, while Triton exposes request and GPU metrics useful for capacity tuning.

**Stable conclusion:** platform owners choose accelerator partitioning. The MLOps agent consumes an approved resource/isolation class and refuses unsafe targets. Protect champion/rollback and reconciliation capacity during candidate load.

## Finding 12 — provider/model changes are behavioral releases

OpenAI documents that model prompting behavior can differ between snapshots and recommends pinned versions plus evals. Bedrock exposes model lifecycle states and EOL dates; Vertex AI documents mutable/auto-updating model references. Provider failover can change structured output, tool use, safety behavior, latency, cost, data handling, and region availability.

**Stable conclusion:** pin exact model/provider deployment where possible; monitor lifecycle; qualify replacements across the full behavioral/governance suite; never treat provider failover or an auto-updating alias as a transparent retry.

## Finding 13 — identity must distinguish definition, snapshot, execution, deployment, and pointer

Official systems use similar words for different objects: Kubeflow/MLflow experiments group executions; runs execute a pipeline or code; registry versions reference model metadata/artifacts; feature definitions differ from materialized values; an endpoint contains deployed models/variants; aliases, branches, tags, stages, and “default” references can move. Lakehouse snapshot/version retention can also outlive or disappear independently of registry metadata.

**Stable conclusion:** the internal model names exact logical object, immutable version/snapshot, execution attempt, mutable pointer/status, native provider ID, content digest, and retention proof separately. A timestamp, display name, run success, stage, alias, branch, endpoint name, or artifact URI alone is never an approval subject.

## Finding 14 — the release gate covers a behavior bundle, not independently “the model”

Weights can remain identical while prompt, tokenizer, preprocessing, feature materialization, label definition, evaluator, judge, tool schema, adapter, serving image, driver, accelerator profile, routing or policy changes. Any of these can alter outputs, safety, latency, cost, observability, or rollback compatibility. Managed registries rarely express this full coupling.

**Stable conclusion:** calculate one behavior-bundle manifest and impact map. Deterministic code checks identities, contracts, hard gates and target state; the model may analyze ambiguous trade-offs or diagnose conflicts but cannot alter the bundle, thresholds, safety case, approval, or effect.

## Finding 15 — accelerator autoscaling is a coupled queue, pod, node, device, quota, and cold-start system

Kubernetes HPA/KPA can request workload replicas while node autoscaling, cloud quota, device allocation, model/image download, GPU memory, MIG/time-slicing, and warm-up remain bottlenecks. NVIDIA explicitly distinguishes time-slicing utilization from MIG isolation. KServe and managed services expose different scaling modes and readiness semantics.

**Stable conclusion:** version an accelerator-pool contract and test full load/recovery paths. Protect champion, rollback, reconciliation, and incident capacity. A pending pod, requested replica, ready pod, allocated logical GPU, and sustainable prediction throughput are different states.

## Finding 16 — provider adapters must preserve differences rather than force false portability

Azure traffic maps named deployments and permits direct deployment invocation; Vertex maps `DeployedModel.id`; SageMaker uses endpoint configurations/fleets and provider-managed baking; KServe behavior depends on CRD/mode; Seldon experiments use proportional weights; BentoCloud canary/autoscaling is distinct from BentoML OSS. Kubeflow Hub explicitly remains passive metadata.

**Stable conclusion:** normalize domain outcomes and evidence fields, not unsupported guarantees. Each adapter publishes its identity mapping, consistency, operation lookup, idempotency, cancellation, limits, exclusions, traffic bypasses, status interpretation, and manual fallback. Unsupported fields fail qualification rather than silently degrading.

## Contradictions and resolved guidance

| Apparent claim | Evidence conflict or limit | Blueprint resolution |
|---|---|---|
| “Registry approval means the model is approved for production.” | Provider fields can trigger CI/CD and differ in scope; application policy/approval may include more evidence | Treat provider approval as an adapter projection/effect; application ledger is authoritative |
| “Use MLflow stages for dev/staging/prod.” | Current MLflow docs deprecate stages | Use immutable versions, aliases/tags, environment separation, and app-owned release state |
| “Alias gives stable deployment identity.” | MLflow and Vertex describe aliases as mutable | Resolve to immutable version/digest; bind approval to digest |
| “One MLflow evaluator handles classic ML and GenAI.” | MLflow documents classic and GenAI evaluator systems as non-interoperable | Normalize results into an internal evaluation-bundle schema; preserve native evidence |
| “KServe standardizes model serving metrics.” | V2 standardizes protocol health/metadata/inference; KServe says runtime metric sets are not unified | Normalize per-runtime metrics through tested adapters; keep native evidence |
| “Drift means quality degraded.” | Distribution change may be harmless; quality requires labels/domain evidence | Drift is diagnostic; gate only with justified detector and response |
| “Drift should trigger automatic retraining.” | Feedback loops, data/label defects, intended-use change, and selective labels make this unsafe | Agent may recommend; model/data/risk owners approve retraining and release |
| “Managed model monitoring is future-proof.” | AWS says Model Monitor is closed to new customers and not getting new features | Do not select it for new builds; existing customers may integrate behind an adapter with exit plan |
| “Automatic rollback is always safer.” | Previous model/runtime/features may be incompatible; downstream outcomes persist | Preflight target; automatic only in sealed low-risk envelope; high-impact human approval |
| “GPU time-slicing provides tenant isolation.” | NVIDIA states it lacks memory/fault isolation | Use MIG/dedicated resources or stronger boundary where risk requires |
| “OpenTelemetry GenAI fields are stable and safe to log.” | Conventions are development-stage and content attributes may contain sensitive data | Maintain internal schema, pin version, metadata-first capture, strict redaction |
| “A signature proves a model is safe.” | Signature proves an accepted signer signed a digest; it does not prove quality/security | Verify signer/subject plus provenance, scans, load isolation, evaluation, and policy |
| “A workflow engine makes the rollout exactly once.” | External commit can occur around checkpoint/response loss | Stable semantic effect ID, downstream dedup/CAS, receipt, unknown state, reconciliation |
| “Provider failover is just availability handling.” | Behavior, tool/schema, safety, region, cost, and lifecycle differ | Treat as a new behavioral release with full gates |
| “EU/NIST controls apply uniformly to all models.” | NIST is voluntary/risk-based; EU obligations depend on role/use/jurisdiction/risk class | Use them as governance inputs; obtain use-case-specific legal/risk interpretation |
| “Kubeflow Model Registry is the deployment control plane.” | Kubeflow Hub architecture explicitly calls the registry passive metadata; REST API remains alpha | Use it as metadata adapter; release workflow, policy, effects and verification stay application-owned |
| “A Kubeflow/MLflow run is reproducible by definition.” | Run records can point to mutable images, object URIs, pipeline roots, caches or data heads | Pin compiled pipeline/code/images/parameters/data/artifact digests and environment; reproduce in a controlled runner |
| “KServe has one canary/revision semantic.” | Current CRD, legacy component fields, Knative/raw deployment and LLM serving modes differ | Pin installed CRDs/mode; map native generation/revision/traffic fields with sandbox tests |
| “BentoML canary and autoscaling apply to every BentoML deployment.” | The current canary/autoscaling pages describe BentoCloud managed Deployments | Qualify BentoCloud separately; self-hosted BentoML needs its own controller and contracts |
| “Azure mirror traffic is a generic endpoint primitive.” | Managed-online only, one mirrored deployment, max 50%, and older SDK updates can remove state | Enforce surface/version limits and re-read mirror state after every endpoint update |
| “Feature-store name/version proves parity.” | Online latest-value and offline history/materialization can differ by event time, lag, TTL, transformation and defaulting | Bind definition plus materialization/window/watermarks and run entity/event-pair parity checks |
| “A table timestamp or branch is an immutable dataset.” | Branches/tags move; time travel depends on retained log/manifests/data and timestamp semantics | Resolve to snapshot/version/commit plus schema and manifest; enforce retention proof |
| “One successful autoscaling signal proves GPU capacity.” | HPA/KPA, node provisioning, device/quota, model load, memory and warm-up are separate loops | Gate on sustainable attributed throughput and preserved rollback capacity under failure/recovery load |
| “Default drift-tool thresholds are release policy.” | Evidently and other tools choose methods/thresholds based on data shape and defaults; null/multiple-testing/seasonality need explicit handling | Version and backtest detector, baseline, threshold, population and response; drift remains diagnostic evidence |
| “Trace attributes are a stable audit schema.” | OpenTelemetry GenAI conventions moved repositories and development/deprecated fields changed | Pin an internal audit schema; map telemetry versions and keep authoritative events unsampled |

## Architecture decisions supported by evidence

| Decision | Evidence basis |
|---|---|
| Deterministic baseline before agent | Production-agent guidance plus fixed release predicates |
| One bounded loop, no multi-agent baseline | Workload needs synthesis but effects/state remain centralized |
| Application-owned release manifest/state/effect ledger | Mutable registry aliases, provider differences, external effect ambiguity |
| Python ML adapter service | MLflow/Feast/model-format ecosystem; isolate behind typed APIs |
| Existing registry first; MLflow portable default | Avoid new platform; use current registry/version/alias ecosystem |
| Managed serving first unless Kubernetes ML platform already exists | KServe requires platform ownership; managed endpoints reduce undifferentiated operations |
| KServe V2 where compatible | Standard health/metadata/inference contract, with runtime-specific metrics |
| One traffic controller and deterministic rollout analysis | KServe/Argo/provider progressive-delivery semantics |
| OpenLineage as interchange, app graph as release truth | Extensible standard plus completeness/evidence-strength gap |
| Prometheus + OpenTelemetry metadata plus governed samples | Operational maturity and privacy-sensitive content |
| Human-owned retraining and high-impact rollback | Drift/label/feedback ambiguity and governance guidance |
| Cell/fair-queue scale with reserved reconciliation | Multi-tenant capacity and effect-safety requirements |
| Exact definition/snapshot/run/deployment/pointer identities | Cross-platform object semantics and mutable-reference failure modes |
| One behavior-bundle manifest and safety-case impact map | Model/data/prompt/feature/evaluator/tool/runtime coupling |
| Versioned accelerator pool plus recovery-load budget | Kubernetes/node/device/quota/cold-start coupling and isolation limits |
| Adapter-specific limits instead of fake portability | Managed/Kubernetes serving and registry semantics are non-isomorphic |

## Canonical repository guides inspected

The blueprint links instead of duplicating general theory from these 15 guides:

1. [Production agent control plane](../../architectures/production-agent-control-plane.md)
2. [Agent state and event contracts](../../runtime/agent-state-and-event-contracts.md)
3. [Durable execution](../../runtime/durable-execution.md)
4. [Run controls](../../runtime/run-controls.md)
5. [Tool contracts](../../tools/tool-contracts.md)
6. [Tool results, artifacts, and provenance](../../tools/tool-results-artifacts-and-provenance.md)
7. [Context engineering](../../context-memory/context-engineering.md)
8. [Compaction and continuity](../../context-memory/compaction-and-continuity.md)
9. [Memory architecture](../../context-memory/memory-architecture.md)
10. [Permissions, sandboxing, and secrets](../../security/permissions-sandboxing-and-secrets.md)
11. [Prompt injection and untrusted data](../../security/prompt-injection-and-untrusted-data.md)
12. [Idempotency and side effects](../../reliability/idempotency-and-side-effects.md)
13. [Evaluation-driven development](../../evaluation/evaluation-driven-development.md)
14. [Observability and tracing](../../evaluation/observability-and-tracing.md)
15. [Scaling, capacity, and SLOs](../../operations/scaling-capacity-and-slos.md)

The 50-category registry and expansion program were also inspected for the category boundary and stages 0–6 contract.

## Primary and foundational source register

The numbered foundation register preserves the original research set. The Pass-2 qualification log above records the volatile surfaces re-accessed on 2026-08-31; the additional primary sources below capture newly material identity, serving, data/versioning, feature, accelerator, evaluation, and observability behavior. Exact product versions, availability, and lifecycle dates must still be refreshed at implementation time.

### Standards, integrity, governance, and telemetry

1. [SLSA v1.2 Build Provenance](https://slsa.dev/spec/v1.2/build-provenance) — artifact production provenance and verification model.
2. [SLSA v1.2 Provenance overview](https://slsa.dev/spec/v1.2/provenance) — provenance role and tracks.
3. [in-toto Attestation Statement v1](https://github.com/in-toto/attestation/blob/main/spec/v1/statement.md) — typed digest-bound statement model.
4. [OCI Image Manifest artifact guidance](https://github.com/opencontainers/image-spec/blob/main/manifest.md#guidelines-for-artifact-usage) — packaging non-image artifacts.
5. [OCI descriptor and digest verification](https://github.com/opencontainers/image-spec/blob/main/descriptor.md) — content addressability and verification.
6. [OCI Distribution Specification](https://github.com/opencontainers/distribution-spec/blob/main/spec.md) — subject/referrer distribution behavior.
7. [Sigstore Cosign signature verification](https://docs.sigstore.dev/cosign/verifying/verify/) — bundle/signature verification.
8. [OpenTelemetry GenAI attributes](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/) — evolving semantic attributes and sensitivity warnings.
9. [NIST AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/) — lifecycle measurement, monitoring, recovery, and change-management outcomes.
10. [EU Artificial Intelligence Act official text](https://eur-lex.europa.eu/eli/reg/2024/1689/oj) — risk management, logs, human oversight, robustness, post-market monitoring for in-scope high-risk systems.

### Registry, prompts, tracking, lineage, and features

11. [MLflow Model Registry](https://mlflow.org/docs/latest/ml/model-registry/) — versions, lineage, tags, aliases.
12. [MLflow Model Registry workflows](https://mlflow.org/docs/latest/ml/model-registry/workflow/) — alias/tag workflows and stage deprecation.
13. [MLflow classic model evaluation](https://mlflow.org/docs/latest/ml/evaluation/) — classic evaluation API and GenAI separation.
14. [MLflow Prompt Registry](https://mlflow.org/docs/latest/genai/prompt-registry/index.html) — immutable prompt versions, aliases, lineage.
15. [MLflow prompt lifecycle](https://mlflow.org/docs/latest/genai/prompt-registry/manage-prompt-lifecycles-with-aliases/) — mutable prompt aliases and comparison.
16. [MLflow evaluation datasets](https://mlflow.org/docs/latest/genai/datasets/) — trace/manual examples and SQL-backend requirement.
17. [MLflow Tracking Server](https://mlflow.org/docs/latest/self-hosting/architecture/tracking-server/) — backend/artifact architecture and security implications.
18. [MLflow authentication](https://mlflow.org/docs/latest/self-hosting/security/basic-http-auth/) — resource permissions and current auth surface.
19. [OpenLineage overview/spec model](https://openlineage.io/docs/) — job/run/dataset lineage model.
20. [OpenLineage facets](https://openlineage.io/docs/spec/facets/) — extensible metadata and custom facet namespacing.
21. [Feast point-in-time joins](https://docs.feast.dev/getting-started/concepts/point-in-time-joins) — historical feature retrieval semantics.
22. [Feast architecture overview](https://docs.feast.dev/getting-started/architecture/overview) — serving, transformations, and RBAC architecture.
23. [Feast components overview](https://docs.feast.dev/getting-started/components/overview) — feature creation/materialization/serving ownership separation.

### Serving, rollout, capacity, and managed platforms

24. [KServe Inference Protocol V2](https://kserve.github.io/website/docs/concepts/architecture/data-plane/v2-protocol) — health, metadata, and inference contract.
25. [KServe control-plane API](https://kserve.github.io/website/docs/reference/crd-api) — current InferenceService/canary/resource fields.
26. [KServe Prometheus metrics](https://kserve.github.io/website/docs/next/model-serving/predictive-inference/observability/prometheus-metrics) — runtime metric exposure and non-unified metric warning.
27. [KServe autoscaling with Knative Pod Autoscaler](https://kserve.github.io/website/docs/model-serving/predictive-inference/autoscaling/kpa-autoscaler) — concurrency-driven scaling, including GPU workloads.
28. [Argo Rollouts canary](https://argo-rollouts.readthedocs.io/en/stable/features/canary/) — canary steps, traffic and analysis.
29. [Argo Rollouts analysis](https://argo-rollouts.readthedocs.io/en/stable/features/analysis/) — analysis, abort, and inconclusive states.
30. [Argo Rollouts rollback window](https://argo-rollouts.readthedocs.io/en/stable/features/rollback/) — fast rollback mechanics and limits.
31. [SageMaker deployment guardrails](https://docs.aws.amazon.com/sagemaker/latest/dg/deployment-guardrails.html) — blue/green, canary, linear/rolling, alarms, exclusions.
32. [SageMaker Model Registry approval status](https://docs.aws.amazon.com/sagemaker/latest/dg/model-registry-approve.html) — provider approval states and CI/CD triggering.
33. [Vertex AI model aliases](https://docs.cloud.google.com/vertex-ai/docs/model-registry/model-alias) — mutable aliases and default alias behavior.
34. [Azure ML model management](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-manage-models?view=azureml-api-2) — model assets, versions, registry paths.
35. [NVIDIA Triton Model Analyzer metrics](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/model_analyzer/docs/metrics.html) — queue/compute/latency/GPU measures.
36. [NVIDIA GPU time-slicing and MIG comparison](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/gpu-sharing.html) — utilization versus isolation trade-off.
37. [Kubernetes Dynamic Resource Allocation hardening](https://kubernetes.io/docs/concepts/security/hardening-guide/dynamic-resource-allocation/) — device allocation security boundary.

### Monitoring, evaluation, documentation, and production-ML evidence

38. [Google production ML monitoring](https://developers.google.com/machine-learning/crash-course/production-ml-systems/monitoring) — training-serving skew, live quality, age, canary.
39. [Google Rules of ML](https://developers.google.com/machine-learning/guides/rules-of-ml) — training-serving skew and future-data evaluation.
40. [Google production ML pipelines](https://developers.google.com/machine-learning/managing-ml-projects/pipelines) — prediction logging, labels, reproducibility, evolving pipelines.
41. [SageMaker Model Monitor data quality](https://docs.aws.amazon.com/sagemaker/latest/dg/model-monitor-data-quality.html) — current availability warning and baseline/monitor flow.
42. [SageMaker Model Monitor FAQ](https://docs.aws.amazon.com/sagemaker/latest/dg/model-monitor-faqs.html) — label timing, custom monitoring, baseline semantics.
43. [The ML Test Score](https://research.google/pubs/the-ml-test-score-a-rubric-for-ml-production-readiness-and-technical-debt-reduction/) — production testing/monitoring rubric.
44. [Hidden Technical Debt in Machine Learning Systems](https://papers.neurips.cc/paper_files/paper/2015/file/86df7dcfd896fcaf2674f757a2463eba-Paper.pdf) — entanglement, feedback loops, data/configuration debt.
45. [Model Cards for Model Reporting](https://doi.org/10.1145/3287560.3287596) — intended-use and performance reporting.
46. [Datasheets for Datasets](https://doi.org/10.48550/arXiv.1803.09010) — dataset composition, collection, recommended-use documentation.

### Artifact security and provider/model lifecycle

47. [PyTorch `torch.load`](https://docs.pytorch.org/docs/stable/generated/torch.load.html) — untrusted deserialization warning and `weights_only` behavior.
48. [PyTorch serialization semantics](https://docs.pytorch.org/docs/main/notes/serialization.html) — `weights_only` limits and allowlisting.
49. [scikit-learn model persistence](https://scikit-learn.org/stable/model_persistence.html) — pickle-family code execution and version portability.
50. [OpenAI API backward compatibility](https://platform.openai.com/docs/api-reference/backward-compatibility) — model snapshot behavior and eval recommendation.
51. [Amazon Bedrock model lifecycle](https://docs.aws.amazon.com/bedrock/latest/userguide/model-lifecycle.html) — active/legacy/EOL lifecycle and migration.
52. [Amazon Bedrock prompt management](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-management.html) — prompt variants and versions.
53. [MLflow automatic GenAI evaluation](https://mlflow.org/docs/latest/genai/eval-monitor/automatic-evaluations/) — asynchronous online judge monitoring and code-scorer limitation.

### Pass-2 platform and lifecycle additions

54. [MLflow releases](https://github.com/mlflow/mlflow/releases) — observed OSS release/version status.
55. [MLflow Tracking concepts](https://mlflow.org/docs/latest/ml/tracking/) — distinct experiments, runs, Logged Models, backend and artifact stores.
56. [Kubeflow Pipelines releases](https://github.com/kubeflow/pipelines/releases) — backend/SDK release status.
57. [Kubeflow Pipelines run semantics](https://www.kubeflow.org/docs/components/pipelines/concepts/run/) — run, recurring run, execution history and concurrency.
58. [Kubeflow Pipelines artifact semantics](https://www.kubeflow.org/docs/components/pipelines/user-guides/data-handling/artifacts/) — artifact URI/path/metadata and version-gated workspace behavior.
59. [Kubeflow Hub REST API `v1alpha3`](https://www.kubeflow.org/docs/components/hub/reference/rest-api/) — alpha registry API and resource access.
60. [Kubeflow Hub architecture](https://www.kubeflow.org/docs/components/hub/reference/architecture/) — passive metadata repository boundary.
61. [Kubeflow Hub installation](https://www.kubeflow.org/docs/components/hub/installation/) — version, alpha distribution status, Kubernetes and authentication/deployment constraints.
62. [KServe release policy](https://github.com/kserve/kserve/blob/master/RELEASES.md) — pre-1.0 cadence, support and compatibility policy.
63. [KServe control-plane architecture](https://kserve.github.io/website/docs/concepts/architecture/control-plane) — Knative/raw mode and scale-to-zero distinction.
64. [Seldon Core 2 releases](https://github.com/SeldonIO/seldon-core/releases) — observed release and regression/fix history.
65. [Seldon Core 2 installation dependencies](https://docs.seldon.ai/seldon-core-2/installation/installation) — supported Kubernetes/dependency range and optional Kafka path.
66. [Seldon Core 2 experiments](https://docs.seldon.ai/seldon-core-2/user-guide/experiment) — proportional traffic, mirror, routing and stateful caveat.
67. [BentoML releases](https://github.com/bentoml/BentoML/releases) — observed framework version status.
68. [BentoCloud canary deployments](https://docs.bentoml.com/en/latest/scale-with-bentocloud/deployment/canary-deployments.html) — managed multi-version routing semantics.
69. [BentoCloud concurrency and autoscaling](https://docs.bentoml.com/en/latest/scale-with-bentocloud/scaling/autoscaling.html) — managed replica/concurrency/scale-to-zero behavior.
70. [SageMaker registry object model](https://docs.aws.amazon.com/sagemaker/latest/dg/model-registry-models.html) — model package groups, versions and ARNs.
71. [SageMaker canary traffic shifting](https://docs.aws.amazon.com/sagemaker/latest/dg/deployment-guardrails-blue-green-canary.html) — fleet, capacity, baking, alarm and rollback mechanics.
72. [Vertex AI deployment API sample](https://docs.cloud.google.com/vertex-ai/docs/samples/aiplatform-deploy-model-sample) — Endpoint, `DeployedModel`, traffic map and long-running operation.
73. [Vertex AI endpoint deploy CLI](https://docs.cloud.google.com/sdk/gcloud/reference/ai/endpoints/deploy-model) — explicit deployed-model ID, accelerator, autoscaling and traffic arguments.
74. [Azure ML safe online rollout](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-safely-rollout-online-endpoints?view=azureml-api-2) — named deployment, live/mirror traffic and exclusions/version constraints.
75. [Azure ML online endpoint concepts](https://learn.microsoft.com/en-us/azure/machine-learning/concept-endpoints-online?view=azureml-api-2) — endpoint/deployment routing, mirroring and direct-deployment invocation.

### Pass-2 data, feature, capacity, and evaluation additions

76. [SageMaker Feature Store concepts](https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store-concepts.html) — feature group, record ID, event time, online/latest and offline/history semantics.
77. [SageMaker Feature Store offline format](https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store-offline.html) — offline lag, Glue/Iceberg, write/ingest fields and deletion records.
78. [SageMaker Feature Store schema updates](https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store-update-feature-group.html) — additive, non-removable feature-definition behavior.
79. [Azure ML managed feature store](https://learn.microsoft.com/en-us/azure/machine-learning/concept-what-is-managed-feature-store?view=azureml-api-2) — immutable feature-set versions and point-in-time retrieval.
80. [Azure ML feature materialization](https://learn.microsoft.com/en-us/azure/machine-learning/feature-set-materialization-concepts?view=azureml-api-2) — transformation windows, jobs and online/offline stores.
81. [DVC releases](https://github.com/iterative/dvc/releases) — observed data-versioning tool release status.
82. [DVC command workflow](https://dvc.org/doc/command-reference/) — Git/DVC metadata, cache, remote and reproducibility workflow.
83. [lakeFS internals](https://docs.lakefs.io/concepts/internals/) — immutable commit IDs, mutable refs, object mapping and garbage collection.
84. [Apache Iceberg branching and tagging](https://iceberg.apache.org/docs/latest/branching/) — snapshot references, retention and schema behavior.
85. [Apache Iceberg snapshot-reference specification](https://iceberg.apache.org/spec/#snapshot-references) — branch/tag mutability and snapshot identity.
86. [Delta Lake release compatibility](https://docs.delta.io/releases/) — Delta/Spark version compatibility.
87. [Delta Lake time travel and retention](https://docs.delta.io/delta-batch/) — version/timestamp semantics and retained log/data requirements.
88. [Delta Lake `VACUUM`](https://docs.delta.io/delta-utility/) — irreversible time-travel loss and concurrent-reader safety.
89. [Kubernetes Horizontal Pod Autoscaling](https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/) — `autoscaling/v2`, metrics/readiness/stabilization and scale-to-zero status.
90. [Kubernetes node autoscaling](https://kubernetes.io/docs/concepts/cluster-administration/node-autoscaling/) — pending-pod constraints, provisioning and workload/node coupling.
91. [Evidently releases](https://github.com/evidentlyai/evidently/releases) — observed drift/evaluation library release status.
92. [Evidently data-drift algorithm](https://docs.evidentlyai.com/metrics/explainer_drift) — data-dependent default tests, thresholds, null handling and limits.
93. [Phoenix dataset versioning](https://arize.com/docs/phoenix/learn/datasets-and-experiments/datasets-concepts) — versioned evaluation examples and experiment binding.
94. [Phoenix evaluation](https://arize.com/docs/phoenix/evaluation/evals) — code/model judges, traces and production-evidence boundary.
95. [OpenTelemetry semantic-convention releases](https://github.com/open-telemetry/semantic-conventions/releases) — observed semantic-convention version status.
96. [Prometheus instrumentation guidance](https://prometheus.io/docs/practices/instrumentation/) — label-cardinality and metric-design constraints.

## Sources intentionally excluded as authority

- product marketing pages without enforceable semantics;
- generic “fully autonomous MLOps” articles;
- benchmark claims without the full model/prompt/tool/evaluator/environment release;
- tutorials that grant notebooks or chat agents blanket registry/cloud/cluster credentials;
- universal drift threshold recipes without population, action, and false-alarm evidence;
- community code samples treated as proof of provider guarantees;
- legal summaries used instead of official text and local counsel/risk interpretation.

## Known limitations

- No live MLflow, KServe, Argo Rollouts, Feast, SageMaker, Vertex, Azure ML, Triton, Kubernetes, GPU, or model-provider environment was exercised.
- The blueprint does not choose a universal statistical drift test or threshold; these require domain data, actionability, and backtesting.
- It does not specify sector-specific clinical, financial, employment, biometric, safety, or consumer-protection controls.
- Exact IAM actions, tenant isolation, product tiers, quotas, regions, retention, pricing, APIs, and deprecations must be qualified in the target estate.
- Evaluation examples are schemas and test shapes, not validated domain metrics.
- A provider's documented capability can have workload exclusions or eventual-consistency behavior not captured in a survey; adapter tests remain mandatory.
- “Rollback” cannot undo already-consumed predictions or guarantee restoration when features/data/runtime changed.
- Legal and regulatory citations are governance evidence, not legal advice.

## Refresh checklist

- [ ] Recheck MLflow stages, auth/workspaces, registry, prompt, and evaluator APIs.
- [ ] Recheck KServe CRDs/protocol, runtime metrics, canary behavior, and supported Kubernetes versions.
- [ ] Recheck Argo Rollouts analysis/traffic/rollback and managed endpoint exclusions.
- [ ] Recheck OpenLineage/OCI/SLSA/in-toto/OpenTelemetry specification versions.
- [ ] Recheck Feast provider/feature semantics and the actual feature-store implementation.
- [ ] Recheck model-provider versions, behavior guarantees, regions, prices, quotas, and EOL dates.
- [ ] Recheck model-format deserialization advisories and dependency vulnerabilities.
- [ ] Recheck monitoring product availability; retain an exit plan for deprecated services.
- [ ] Revalidate domain labels, protected slices, drift baselines, threshold ownership, and feedback loops.
- [ ] Rerun adapter, security, failure-injection, tenant, load, DR, and autonomy-level evaluations.
- [ ] Review regulatory applicability and documentation/retention requirements with accountable specialists.

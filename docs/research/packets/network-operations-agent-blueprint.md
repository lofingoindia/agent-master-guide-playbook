# Network Operations Agent Blueprint — Research Packet

> **Status:** Completed primary-source research packet  
> **Research date:** 2026-08-31  
> **Scope:** topology and dependency truth, routing, DNS, certificates, load balancers and traffic policy, packet/flow/reachability evidence, modeled management, staged configuration, verification, rollback, connectivity recovery, security, evaluation, and production deployment  
> **Blueprint:** [Network Operations Agent](../../agents/network-operations-agent/README.md)

## Research objective

Determine the smallest defensible architecture and maturity path for a network operations agent that begins with bounded evidence collection, progresses to production-quality change proposals, and eventually executes a narrow reversible change catalog without confusing device/API acceptance with end-to-end connectivity.

The research intentionally distinguishes network operations from infrastructure operations, SRE/incident response, and security investigation. It also tests claims that often hide unsafe scope: “source of truth,” “atomic update,” “commit confirmed,” “programmed,” “health,” “rollback,” “digital twin,” “real time,” “vendor neutral,” and “self-healing.”

## Method

1. Started with IETF standards and official model/protocol specifications for topology, intent, YANG, NETCONF, RESTCONF, gNMI, routing, DNS, TLS, telemetry, flow, and active measurement.
2. Compared the guarantees of those standards with official implementation documentation for OpenConfig, Envoy, Kubernetes Gateway API, NetBox, NAPALM, Ansible, Batfish, Containerlab, and selected cloud network/DNS APIs.
3. Cross-checked reliability and operational implications against CISA hardening guidance and Google SRE guidance on automation, canaries, monitoring, SLOs, and launch readiness.
4. Examined optional capabilities and transaction boundaries rather than assuming a common abstraction provides common semantics.
5. Preferred primary sources and official repositories. No secondary blog was required for a core architectural claim.
6. Treated network digital twins as an emerging practice whose coverage and fidelity must be declared, not as a universal production oracle.
7. Recorded volatility and refresh triggers for APIs, device/controller versions, models, adapter drivers, schemas, and implementation status.

The packet synthesizes sources; it does not reproduce them. Deployment must qualify the exact device operating systems, controllers, cloud resources, certificates, protocol features, and adapter versions in use.

## Research questions

- Where is a model useful, and where is a deterministic controller or runbook enough?
- How should network intent, discovered state, inferred dependencies, and observed path behavior coexist?
- What does a source of truth own when authority differs by field?
- How should discovery freshness, snapshot skew, stream gaps, provenance, and absence be represented?
- Which typed read, probe, stage, validate, commit, confirm, cancel, rollback, and reconcile tools are required?
- What atomicity, concurrency, and rollback guarantees do NETCONF, gNMI, DNS, cloud, and traffic-control APIs actually provide?
- How should multi-device and cross-domain changes be staged without claiming a network-wide transaction?
- What evidence distinguishes intended configuration, accepted state, RIB, FIB, flow, packet, and user-path behavior?
- How do DNS cache/TTL, certificate deployment, load-balancer dependencies, stateful flows, and routing convergence change rollback design?
- How should task state, long-running workflow, memory, context compaction, locks, and ambiguous effects survive failure?
- How should identity, short-lived credentials, tenant isolation, prompt injection, secrets, management-plane isolation, and break-glass work?
- What labs, replay cases, configuration analysis, failure injection, SLOs, capacity planning, HA/DR, and upgrade gates are required?
- Which work belongs to infrastructure, SRE, or security instead?

## Category decision

The category is distinct across more than three operational seams:

| Seam | Network operations | Adjacent-category handoff |
|---|---|---|
| Environment | Routers, switches, DNS, PKI termination, traffic controllers, load balancers, network management and cloud network APIs | Compute/platform resources go to infrastructure |
| Authority | Network intent, modeled configuration, bounded traffic/connectivity change and recovery | Incident/service-risk authority goes to SRE; malicious-activity decisions go to security |
| State/time | Protocol convergence, caches/TTLs, sessions, distributed controllers, multi-vantage observation | Platform declarative reconciliation is infrastructure state |
| Ground truth | Intent + accepted state + control/forwarding state + independent path evidence | User/service objectives remain SRE ground truth; adversarial evidence remains security ground truth |
| Recovery | Make-before-break, staged withdrawal, path/DNS/traffic correction, network rollback | Resource repair is infrastructure; incident coordination is SRE; containment/eradication is security |
| Evaluation | Reachability/isolation invariants, path diversity, convergence, propagation, fault-domain safety | Each adjacent agent has its own outcome suite |

Adopted boundary: the network agent may express a dependency on compute, participate in an SRE-led incident, and execute a security-approved network containment step. It does not thereby own compute desired state, incident command, attacker attribution, evidence-chain decisions, or eradication.

## Standards and volatility baseline

| Surface | Baseline checked on 2026-08-31 | Blueprint treatment | Refresh trigger |
|---|---|---|---|
| YANG / NETCONF / RESTCONF | RFC 7950, RFC 6241, RFC 8040 | Negotiate optional capabilities; pin models and target behavior; never generalize per-target atomicity | Target OS, server, YANG model, protocol implementation change |
| NMDA / topology / intent | RFC 8342, RFC 8345, RFC 8346, RFC 9315, RFC 8969 | Separate intended, operational, inferred, and observed facts; use layered topology and assurance | Intent compiler, source ownership, or topology model change |
| OpenConfig / gNMI | Current official specification and model site | Record collection timing, per-target Set scope, model revision, vendor deviations, conformance | gNMI/model/target/adapter release |
| Streaming telemetry | RFC 8639, RFC 8641, RFC 9232 | Streaming is not completeness; preserve sync, sequence, gap, dampening, and privacy metadata | Collector/publisher/subscription policy change |
| Routing | RFC 4271 family plus operations, roles, BMP, BFD, RPKI, graceful-restart sources below | Validate policies and exact route deltas; triangulate RIB/FIB/flow/path; N4 for core/export work | Protocol feature, policy, router OS, collector coverage change |
| DNS | Core DNS, dynamic update, DNSSEC, cache/TTL, transfers, catalog zones sources below | Treat as cache-timed rollout; use conditional update and authoritative + recursive verification | Provider behavior, resolver policy, DNSSEC/zone topology change |
| TLS / ACME | RFC 8446, RFC 9525, RFC 8555, RFC 8739, RFC 9773, CA/B Forum baseline | Issuance ≠ deployment; verify served identity; protect keys; keep safe overlap | CA policy, trust store, ACME profile, termination platform change |
| Traffic/load balancing | Envoy xDS, Kubernetes Gateway API/Service/EndpointSlice | Respect dependency order, ACK/NACK/conditions, endpoint state; verify data plane | Controller/data-plane/API version or implementation-status change |
| Flow and measurement | IPFIX, STAMP, IOAM, OAM and measurement RFCs | Vantage- and interval-specific evidence; declare sampling/coverage; payload exceptional | Exporter/probe/sensor/retention change |
| Source of truth | NetBox current official documentation | Field-specific authority; events are hints; reconcile with versioned reads | NetBox/API/plugin/schema/ownership change |
| Multivendor adapters | NAPALM, Ansible resource modules, OpenConfig, native APIs | Per-target capability manifests and conformance; CLI last resort | Driver/collection/OS/API/model change |
| Analysis and lab | Batfish and Containerlab official documentation | Pin coverage and images; a passing model/lab is not live proof | Engine/vendor support/topology/image change |
| Cloud network APIs | AWS, Azure, and Google official DNS/route/LB resources sampled | Keep provider-specific concurrency, async, atomicity, and propagation semantics | API/resource/provider behavior change |
| AI security/governance | OWASP prompt-injection/excessive-agency, NIST AI 600-1 | Untrusted planner/data; deterministic authorization and least privilege | Threat guidance or deployment/provider change |
| Network hardening | CISA 2023–2025 management exposure and network hardening guidance | Independent OOB, protected management, AAA/logging, no lateral/default-open path | Network architecture or CISA guidance change |

## Production synthesis

### 1. Configuration acceptance and connectivity are different outcomes

NETCONF `ok`, gNMI Set success, a provider's completed change, xDS ACK, or Gateway `Programmed` condition establishes a bounded control-plane fact. None proves a name resolves as expected through caches, a route is programmed in every relevant FIB, stateful return traffic works, the right certificate is served, or a client can complete the service path. The blueprint requires independent outcome verification.

### 2. There is usually no network-wide transaction

NETCONF candidate/confirmed commit is capability- and target-scoped. gNMI Set is atomic for one SetRequest to one target. DNS providers offer transactions within described resource scopes. These mechanisms are valuable but cannot make heterogeneous multi-device, DNS, PKI, routing, and traffic changes globally atomic. The safe abstraction is a sealed dependency DAG, make-before-break, conditional writes, bounded fault-domain stages, and reconciliation.

### 3. Confirmed commit is a safety mechanism, not magic rollback

RFC 6241 provides confirmed-commit timeouts, persistence, cancellation, and rollback behavior when supported. Shared configuration and locks matter; verification must finish before confirmation. Device/control failure or changed dependencies can still make recovery difficult. Maintain independent OOB access and explicit recovery.

### 4. Source of truth must be field-specific

RFC 8342 distinguishes intended and operational state and records origins. RFC 8345 represents supporting topology. NetBox can authoritatively own selected inventory/IPAM/circuit facts while a controller owns intended policy and a device owns current operational counters. The design maps authority per entity field, retains contradictions, and never treats an event notification or chat memory as current truth.

### 5. Telemetry transport is not freshness or completeness

gNMI updates are independently timed; Get paths may not share one collection instant. YANG-Push supports synchronization and gap/suspension semantics, and publishers can reject expensive subscriptions. The system must expose observation time, skew, coverage, sequence/boot boundaries, sampling, and loss. “No route/flow/packet” is inconclusive when collection was incomplete.

### 6. Routing assurance needs several planes

Routing intent and running policy explain expected behavior; BMP/route views expose control-plane information; device RIB and FIB show local selection/programming; flows and active probes show observed forwarding. RFC operational guidance supports filtering, default-reject, roles/leak protection, prefix limits, RPKI context, and carefully configured failure detection. No single plane is sufficient.

### 7. DNS rollback is a forward, cache-timed operation

Authoritative acceptance does not invalidate records already held by recursive resolvers and clients. TTL must be lowered before a planned change and observed through the old TTL; old endpoints need overlap; negative caches matter; DNSSEC rollovers need protocol-specific timing. Empty catalog-zone behavior demonstrates that apparently small DNS configuration can have huge blast radius.

### 8. Certificate issuance is not certificate activation

ACME automates order/authorization/issuance and ARI can guide renewal windows. A successful order does not prove every listener loaded the certificate or serves the correct chain for the reference identity. RFC 9525 requires SAN-based identity matching and no Common Name fallback. Verify actual handshakes at every relevant termination path; retain safe overlap; never expose private keys to the model.

### 9. Load-balancer/controller status is dependency evidence, not service evidence

Envoy xDS resources are versioned and ACK/NACKed, with warming and make-before-break dependency ordering. Gateway API conditions communicate controller status, while EndpointSlice has readiness/serving/terminating distinctions. Traffic changes must warm dependencies, canary weight/cohort, observe service and capacity, drain state, and remove old dependencies last.

### 10. Statefulness makes “healthy endpoints” insufficient

Connection tracking, NAT/SNAT capacity, source-IP preservation, stickiness, hashing, long-lived flows, QUIC/UDP, and asymmetric paths can fail during an otherwise valid traffic shift. Plans and labs must model the relevant state and drain horizon.

### 11. Packet capture is powerful, partial, and sensitive

A capture proves what one sensor observed during one interval, subject to filter/snap length/drop. RFC 9232 cautions against unnecessary end-user payload collection; IPFIX itself has confidentiality and privacy concerns. Default to counters, flow metadata, and bounded active measurements. Capture needs a separate privilege, purpose, minimization, encryption, access log, and short retention.

### 12. Uniform libraries do not create uniform target guarantees

NAPALM presents common getters and candidate/commit/rollback methods, but capabilities vary by driver. Ansible resource-module states can be broad; official documentation warns that `overridden` can remove management-related configuration. OpenConfig model support also varies. Record a target-version capability manifest and refuse unsupported semantics.

### 13. Configuration analysis and labs are necessary but bounded

Batfish provides valuable differential reachability, route policy, loop, blackhole, and ECMP analysis for supported configurations. Containerlab makes reproducible code-defined topologies possible. Neither proves proprietary ASIC behavior, provider internals, scale timing, or full live state. Every analysis/lab result needs a coverage declaration and post-change verification.

### 14. Digital twins require explicit fidelity claims

Current network-digital-twin work remains evolving and implementation-specific. The blueprint uses a broader term—versioned analysis/lab environment—and requires declared vendors, protocols, scale, timing, synchronization, unsupported features, and validated prediction history. “Twin passed” never replaces a production canary.

### 15. Prompt injection can arrive from the network

Device banners, DNS TXT records, certificate strings, config descriptions, tickets, logs, and packets can carry instruction-like text. Sanitization helps but is not the main boundary. The model has no credentials or direct management route; tool results are structured data; deterministic policy authorizes concrete effects; the executor consumes only a sealed plan.

### 16. Management and recovery paths must be independent

CISA guidance supports dedicated OOB, protected management VRFs, restricted management exposure, centralized logs, strong AAA, and no unnecessary lateral management. An agent cannot recover safely if the route, DNS, identity, audit, or credential dependency it changes is its only control path.

### 17. Timeouts produce uncertainty, not a safe retry signal

An external change can commit after the client loses its response. Persist effect intent before dispatch, use stable operation IDs and native conditional/idempotency features, and reconcile native/current state before retry. `UNCERTAIN`, `PARTIAL`, `SUPERSEDED`, and `INDETERMINATE` are first-class states.

### 18. Rollback must be revalidated against the current network

Returning a previous configuration does not instantly reverse BGP convergence, DNS caches, sessions, TLS clients, or traffic state. Another actor may have changed the resource. Rollback is a new controlled effect with fresh preconditions; roll-forward, drain, or isolation may be safer.

### 19. Autonomy should expand by proven capability, not confidence

The initial system should answer a narrow read-only question. The next stage creates plans without writes. Reliable v1 executes one reversible N3 catalog through durable deterministic infrastructure. N4 remains specialist, multi-party supervised; N5 remains outside the general agent. A model's confidence never changes the authorization tier.

### 20. Telemetry scale is often the dominant cost

Thousands of devices and modeled paths can generate terabytes per day before indexing. Aggregate at collectors, materialize current state, tier retention, and pass compact evidence references to the model. Preserve gaps. Reserve workflow capacity for reconciliation/rollback so new investigative work cannot starve recovery.

### 21. Adapter qualification is per operation and target version

A common protocol, SDK, module, or provider does not create common semantics. Qualify identity, schema, pagination/completeness, candidate/running/operational state, atomic scope, concurrency, asynchronous jobs, ambiguity, rollback, reconciliation, verification, security, quotas, and recovery for one exact adapter + target/API/OS + enabled-feature tuple. Read qualification does not imply write qualification.

Primary documentation exposes material differences. NETCONF capabilities are optional and negotiated; Junos documents a particular confirmed-commit implementation. PAN-OS REST edits still require a separate commit, whose XML API returns an asynchronous job ID. Amazon Route 53 describes a transactional batch within one hosted zone plus asynchronous status. NetBox 4.6 added single-object weak ETag/`If-Match` protection. F5 AS3 distinguishes broad declaration source-of-truth behavior from per-application update scope. These are adapter evidence, not portable assumptions.

### 22. Context and memory are loss-aware projections over durable truth

Every model call should have a context manifest with tenant/task/state, selected artifacts and trust, exclusions/gaps, authority, budgets, builder version, and behavior-bundle ID. Compaction writes a continuity receipt containing source-event range, retained evidence/contradictions/gaps/effects/approvals/leases, deadlines, budgets, explicit omissions, and compiler/compactor versions. Repeated compaction tests must preserve VRF/view/prefix/SNI identity, numerical intent, negative-evidence limits, TTL/convergence windows, OOB dependencies, and `UNCERTAIN` effects.

Turn/scratch, working/run, session, durable task, domain, long-term, preference, and episodic/outcome memory have separate authority. Live topology/config/routes/health, approvals, credentials, packets, tickets, and model conclusions never become general memory automatically. Durable task/effect/evidence records—not conversational recall—resume work.

### 23. Promote behavior bundles and learn from failures through governance

Model, prompt, context builder, compactor, memory policy, tool schema, adapter qualifications, parsers, topology/intent compiler, policy, workflow, graders, verification profiles, labs, and runbooks interact. Promote the exact tested combination as one signed behavior bundle. Shadow work has no write credentials or writer leases; canaries are fault-domain bounded; rollback changes future routing while historical effects remain immutable.

Incidents, operator corrections, rollbacks, reconciliation discrepancies, adapter drift, and downstream service defects become minimized, permissioned evaluation/failure cases only after causality, privacy/security, licensing, leakage, retention, and owner review. They cannot auto-write prompts, thresholds, runbooks, policy, adapter capabilities, memory, or training data.

## Important contradictions and adopted resolutions

| Common claim | Evidence or caveat | Adopted resolution |
|---|---|---|
| “The controller says programmed, so traffic is live.” | Controller conditions/ACKs describe control-plane progress, not every data path | Require independent service-path and forwarding verification |
| “NETCONF/gNMI makes the network transaction atomic.” | Capabilities and atomic scope are per target/request | Express exact scope; coordinate multi-target work as staged DAG |
| “Confirmed commit guarantees rollback.” | It is optional, timed, and interacts with locks/shared config/control reachability | Use only when attested; keep OOB and tested recovery |
| “The inventory system is the source of truth.” | Authority differs for allocation, intent, running state, counters, and path observations | Map authority per field and retain contradictions |
| “Streaming telemetry is real time and complete.” | Collection times differ; gaps, rejection, dampening, sampling, and overload occur | Carry time/skew/sequence/gap/coverage; block writes on missing requirements |
| “A RIB route means the destination is reachable.” | FIB, next hop, downstream, return path, policy, and stateful devices can differ | Triangulate RIB/FIB/flow/active service evidence |
| “Traceroute shows the path.” | ECMP and flow hashing expose one possible path from one vantage | Use stable varied flow keys and multiple vantages; state limits |
| “No IPFIX records means no traffic.” | Sampling, exporter loss, observation placement, and time window can hide traffic | Absence requires collection coverage; otherwise report unknown |
| “DNS rollback restores clients immediately.” | Existing positive and negative caches persist until their own timers/policies | Plan overlap and propagation; rollback is another forward update |
| “A certificate was issued, so rotation succeeded.” | Listener may not load or serve it; identity/chain can be wrong | Verify served fingerprint/SAN/chain at all termination paths |
| “ACK/NACK gives application health.” | xDS ACK is resource acceptance and warming/dependencies still matter | Verify backend reachability, capacity, distribution, and user path |
| “Healthy backends make a shift safe.” | NAT, stickiness, connection state, asymmetry, and draining may fail | Include stateful-flow invariants and observation/drain horizons |
| “Vendor-neutral adapter means identical capability.” | NAPALM/OpenConfig/module behavior varies by target/version | Capability manifest, conformance, refusal of unsupported promises |
| “Simulation/digital twin passed, so production is safe.” | Feature, ASIC, provider, timing, state, and scale fidelity is incomplete | Publish coverage; canary and independently verify production |
| “Packet capture is definitive.” | It is point/time/filter limited and may expose sensitive content | Use as scoped last resort with gaps/privacy; combine other evidence |
| “Retry on timeout is resilient.” | The change may have committed after response loss | Record intent/native ID; reconcile; retry only proven not-applied |
| “Rollback re-applies the old file.” | Network/cache/session world may have changed and old dependency may be invalid | Revalidate recovery; choose rollback/forward/drain/isolation |
| “Approval gives execution authority forever.” | Target/policy/topology/window/role can change | Bind digest and expiry; reauthorize and recheck preconditions at dispatch |
| “Sanitizing banners solves prompt injection.” | Instruction-like content can survive and model reasoning is not an auth boundary | No direct credentials/write route; typed schemas; deterministic policy/executor |
| “An autonomous agent can use break-glass if normal control fails.” | Break-glass must survive and bypass the compromised/failed automation path | Human custody, OOB, alarmed access; never model-callable |
| “More agents increase resilience.” | More planners add coordination, authority, cost, and nondeterministic failure paths | One bounded planner; parallelize only deterministic independent probes |
| “A uniform SDK/module creates uniform semantics.” | Similar methods can hide different identity, atomic scope, commit, rollback, and finality | Qualify each operation against the exact target/API/version and refuse unsupported promises |
| “Components can be promoted independently without changing system behavior.” | Parsers, context, adapters, policy, graders, and runbooks interact | Promote a tested behavior-bundle combination; keep explicit in-flight migration and rollback rules |
| “Production feedback should update the agent immediately.” | Feedback can be misattributed, sensitive, poisoned, or leak across evaluation splits | Review, minimize, label, and evaluate first; change behavior only through release gates |

## Source register

### Network management, topology, intent, and telemetry

| Primary source | What was checked | Blueprint implication |
|---|---|---|
| [RFC 7950 — YANG 1.1](https://www.rfc-editor.org/rfc/rfc7950.html) | Typed configuration/state data modeling | Pin schema revisions; validate types and paths before adapter rendering |
| [RFC 6241 — NETCONF](https://www.rfc-editor.org/rfc/rfc6241.html) | Datastores, locks, edit/validate, rollback-on-error, candidate and confirmed commit capabilities | Negotiate exact capabilities; do not assume stage/rollback; use locks and confirmation deadlines correctly |
| [Junos NETCONF commit](https://www.juniper.net/documentation/us/en/software/junos/netconf/topics/ref/tag/netconf-commit.html) | Vendor-specific immediate and confirmed-commit variants and timer behavior | Record negotiated capability and actual confirmation deadline; vendor behavior is not a universal NETCONF guarantee |
| [RFC 8040 — RESTCONF](https://www.rfc-editor.org/rfc/rfc8040.html) | HTTP access to YANG-modeled data | Typed adapter option; still target/version qualified |
| [RFC 8342 — NMDA](https://www.rfc-editor.org/rfc/rfc8342.html) | Intended, operational, applied/system state and origin | Keep state classes and provenance distinct |
| [RFC 8345 — Network Topologies](https://www.rfc-editor.org/rfc/rfc8345.html) | Nodes, links, termination points, supporting topology, origin | Layered versioned graph with relationship provenance |
| [RFC 8346 — Layer 3 Topologies](https://www.rfc-editor.org/rfc/rfc8346.html) | L3 topology augmentation | Normalize protocol-independent L3 graph and reconcile views |
| [RFC 8349 — Routing Management](https://www.rfc-editor.org/rfc/rfc8349.html) | YANG routing and routing-instance model | Typed routing reads with VRF/table context |
| [RFC 9315 — Intent-Based Networking](https://www.rfc-editor.org/rfc/rfc9315.html) | Intent concepts, translation, mapping, assurance | Intent must compile into measurable invariants; declarative text is not execution |
| [RFC 8969 — Service and Network Management Automation](https://www.rfc-editor.org/rfc/rfc8969.html) | YANG-driven service/network management architecture | Separate service intent, network models, and device realization |
| [RFC 8639 — Subscriptions to YANG Notifications](https://www.rfc-editor.org/rfc/rfc8639.html) | Subscription lifecycle and state | Model subscription setup/status, not only updates |
| [RFC 8641 — YANG-Push](https://www.rfc-editor.org/rfc/rfc8641.html) | Periodic/on-change updates, sync, dampening, suspension/rejection | Preserve gaps and publisher limits; streaming is not automatic completeness |
| [RFC 9196 — YANG Modules Describing Capabilities](https://www.rfc-editor.org/rfc/rfc9196.html) | Capability representation | Support explicit feature/model capability discovery |
| [RFC 9232 — Network Telemetry Framework](https://www.rfc-editor.org/rfc/rfc9232.html) | Management/control/forwarding/external data, collection, privacy, isolation | Separate evidence planes; avoid default payload; design scale and telemetry isolation |
| [OpenConfig gNMI specification](https://openconfig.net/docs/gnmi/gnmi-specification/) | Get timing, Subscribe modes, Set transaction, TLS/authentication | Expose snapshot timing; single-target transaction scope; mTLS; capability testing |
| [OpenConfig data models](https://openconfig.net/projects/models/) | Vendor-neutral modeled configuration/state surface | Prefer modeled adapters, but pin revision and validate vendor coverage |

### Routing and reachability

| Primary source | What was checked | Blueprint implication |
|---|---|---|
| [RFC 4271 — BGP-4](https://www.rfc-editor.org/rfc/rfc4271.html) | Base BGP behavior and state | Ground routing tools in exact AFI/SAFI/session/policy context |
| [RFC 7454 — BGP Operations and Security](https://www.rfc-editor.org/rfc/rfc7454.html) | Filtering, maximum prefixes, operations/security | Deterministic prechecks for filters, limits, and policy hygiene |
| [RFC 8212 — EBGP Default Reject](https://www.rfc-editor.org/rfc/rfc8212.html) | Safer default propagation behavior | Check explicit import/export; treat absent policy as reject baseline where supported |
| [RFC 9234 — Route Leak Prevention Using Roles](https://www.rfc-editor.org/rfc/rfc9234.html) | BGP roles and leak prevention/detection | Include peer role and OTC/leak invariants in policy validation |
| [RFC 7854 — BMP](https://www.rfc-editor.org/rfc/rfc7854.html) | Structured BGP monitoring and route views | Use collectors for independent control-plane evidence; record view coverage |
| [RFC 8671 — BMP Adj-RIB-Out](https://www.rfc-editor.org/rfc/rfc8671.html) | Monitoring advertised routes | Validate actual outbound view where deployed |
| [RFC 9069 — BMP Loc-RIB](https://www.rfc-editor.org/rfc/rfc9069.html) | Monitoring selected local RIB | Distinguish pre-policy, post-policy, and local selected views |
| [RFC 8893 — RPKI Validation State Extended Community](https://www.rfc-editor.org/rfc/rfc8893.html) | Carrying origin-validation state | Preserve validation-state provenance; do not reduce to a model label |
| [RFC 8210 — RPKI to Router Protocol](https://www.rfc-editor.org/rfc/rfc8210.html) | Router/cache validation data transport | Monitor cache/session health as a routing dependency |
| [RFC 5880 — BFD](https://www.rfc-editor.org/rfc/rfc5880.html) | Fast forwarding failure detection and state | Treat timer/fate-sharing errors as high-impact failure modes |
| [RFC 5882 — Generic BFD Application](https://www.rfc-editor.org/rfc/rfc5882.html) | Interaction with protocol layers and graceful restart | Validate which layer uses BFD and its protocol interaction |
| [RFC 4724 — BGP Graceful Restart](https://www.rfc-editor.org/rfc/rfc4724.html) | Retaining forwarding during restart | Verify stale-route behavior and forwarding preservation |
| [RFC 9494 — Long-Lived BGP Graceful Restart](https://www.rfc-editor.org/rfc/rfc9494.html) | Extended stale-route behavior and default caution | Do not treat long-lived stale routes as default resilience |
| [RFC 9552 — BGP Link-State](https://www.rfc-editor.org/rfc/rfc9552.html) | Exporting link-state/TE information | Possible topology source with protocol-specific coverage, not universal truth |
| [MANRS Network Operators Program](https://manrs.org/netops/bcop/) | Current operator actions for filtering, anti-spoofing, coordination, validation | Use as operational security cross-check for routing policy |

### DNS

| Primary source | What was checked | Blueprint implication |
|---|---|---|
| [RFC 1034 — DNS Concepts](https://www.rfc-editor.org/rfc/rfc1034.html) and [RFC 1035 — DNS Implementation](https://www.rfc-editor.org/rfc/rfc1035.html) | Core hierarchy, records, caching, messages | Model zones, delegation, RRsets, authority, and cache behavior explicitly |
| [RFC 2136 — Dynamic Update](https://www.rfc-editor.org/rfc/rfc2136.html) | Prerequisites and update semantics | Use prerequisites for conditional/CAS-style updates |
| [RFC 3007 — Secure Dynamic Update](https://www.rfc-editor.org/rfc/rfc3007.html) | Authentication/authorization of updates | DNS write credentials require exact zone/name scope and audit |
| [RFC 4033](https://www.rfc-editor.org/rfc/rfc4033.html), [RFC 4034](https://www.rfc-editor.org/rfc/rfc4034.html), [RFC 4035](https://www.rfc-editor.org/rfc/rfc4035.html) | DNSSEC introduction, records, validation | Verify chain/signatures and treat rollover as specialized high risk |
| [RFC 9364 — DNSSEC BCP](https://www.rfc-editor.org/rfc/rfc9364.html) | Current consolidated DNSSEC overview | Prefer current DNSSEC terminology and operational baseline |
| [RFC 6781 — DNSSEC Operational Practices](https://www.rfc-editor.org/rfc/rfc6781.html) | Operational deployment/maintenance | Require DNSSEC-specific runbooks and specialist review |
| [RFC 7583 — DNSSEC Key Rollover Timing](https://www.rfc-editor.org/rfc/rfc7583.html) | Timing dependencies for rollover | Never generate rollover timing from generic model reasoning |
| [RFC 2308 — Negative Caching](https://www.rfc-editor.org/rfc/rfc2308.html) | Negative response caching | Include negative TTL when creating/restoring names |
| [RFC 8198 — Aggressive Negative Caching](https://www.rfc-editor.org/rfc/rfc8198.html) | DNSSEC-based negative caching | Newly created data may remain hidden by resolver cache behavior |
| [RFC 8767 — Serving Stale Data](https://www.rfc-editor.org/rfc/rfc8767.html) | Resolver serve-stale behavior | Label stale answers; plan for intentionally stale availability behavior |
| [RFC 9803 — DNS TTL Operations](https://www.rfc-editor.org/rfc/rfc9803.html) | TTL planning and common mistakes | Lower before change and account for existing cached TTLs |
| [RFC 9199 — Large Authoritative DNS Operations](https://www.rfc-editor.org/rfc/rfc9199.html) | TTL/resilience/operational scale trade-offs | Balance short rollout TTL against server load/cost/resilience |
| [RFC 5936 — AXFR](https://www.rfc-editor.org/rfc/rfc5936.html), [RFC 1995 — IXFR](https://www.rfc-editor.org/rfc/rfc1995.html), [RFC 1996 — NOTIFY](https://www.rfc-editor.org/rfc/rfc1996.html) | Zone transfer and notification | Validate SOA/serial/secondary propagation, not only primary API state |
| [RFC 9103 — DNS Zone Transfer over TLS](https://www.rfc-editor.org/rfc/rfc9103.html) | Protected transfer and multi-primary considerations | Secure and monitor transfer topology |
| [RFC 9432 — Catalog Zones](https://www.rfc-editor.org/rfc/rfc9432.html) | Automated zone provisioning and mass-deletion risk | Classify catalog changes as exceptional blast radius |

### TLS and certificates

| Primary source | What was checked | Blueprint implication |
|---|---|---|
| [RFC 5280 — PKIX Certificate Profile](https://www.rfc-editor.org/rfc/rfc5280.html) | Certificate/path validation foundations | Validate chain, usage, policy, and time; do not inspect expiry only |
| [RFC 8446 — TLS 1.3](https://www.rfc-editor.org/rfc/rfc8446.html) | TLS handshake and protocol baseline | Verify actual endpoint handshake and negotiated behavior |
| [RFC 9525 — Service Identity](https://www.rfc-editor.org/rfc/rfc9525.html) | Reference identifier matching and SAN/CN behavior | Validate intended identity against SAN; no CN fallback |
| [RFC 8555 — ACME](https://www.rfc-editor.org/rfc/rfc8555.html) | Automated certificate ordering/authorization | Keep issuance and activation states separate; scope challenge credentials |
| [RFC 8739 — ACME STAR](https://www.rfc-editor.org/rfc/rfc8739.html) | Short-term automatically renewed certificates | Account for renewal availability, rate, overlap, and clock |
| [RFC 9773 — ACME Renewal Information](https://www.rfc-editor.org/rfc/rfc9773.html) | CA-suggested renewal windows and replacement reporting | Consume ARI as input to deterministic renewal scheduling |
| [CA/Browser Forum Server Certificate Baseline Requirements](https://cabforum.org/working-groups/server/baseline-requirements/requirements/) | Current public TLS certificate policy baseline | Refresh CA/public-trust policy independently of protocol standards |

### Traffic policy, load balancing, and service dependencies

| Primary source | What was checked | Blueprint implication |
|---|---|---|
| [Envoy xDS protocol](https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol.html) | Versioning, ACK/NACK, warming, dependency ordering, ADS | Make-before-break; halt/reconcile NACK; ACK is not service health |
| [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/) | Role-oriented L4/L7 resource model | Use typed desired policy and explicit controller ownership |
| [Gateway API implementer guidance](https://gateway-api.sigs.k8s.io/guides/implementers-guide/) | Accepted/Programmed/ResolvedRefs and observed generation | Use conditions as controller evidence, then validate data plane |
| [Gateway API HTTPRoute](https://gateway-api.sigs.k8s.io/reference/spec/#httproute) | Route matching, filters, backends | Compile and validate exact route/reference effects |
| [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/) | Traffic policies and service forwarding | Include locality and source/traffic-policy semantics in plan |
| [Kubernetes EndpointSlice API](https://kubernetes.io/docs/reference/kubernetes-api/discovery-resources/endpoint-slice-v1/) | Ready, serving, terminating endpoint conditions | Do not reduce endpoint lifecycle to one health bit |
| [RFC 7665 — Service Function Chaining](https://www.rfc-editor.org/rfc/rfc7665.html) | Service-function paths and architecture | Represent middlebox/service dependencies in path topology |
| [RFC 9015 — Stateful SFC Problems](https://www.rfc-editor.org/rfc/rfc9015.html) | State and symmetry issues | Test stateful/asymmetric behavior during traffic changes |

### Packet, flow, and active measurement

| Primary source | What was checked | Blueprint implication |
|---|---|---|
| [RFC 7011 — IPFIX](https://www.rfc-editor.org/rfc/rfc7011.html) | Flow export protocol and security/privacy | Include templates, sampling/loss, observation point, protection |
| [RFC 7799 — Active and Passive Metrics](https://www.rfc-editor.org/rfc/rfc7799.html) | Measurement method classification | Label evidence by how measurement affects/observes traffic |
| [RFC 8762 — STAMP](https://www.rfc-editor.org/rfc/rfc8762.html) | Active delay/loss measurement | Use authenticated, rate-bounded active tests where appropriate |
| [RFC 9197 — IOAM Data Fields](https://www.rfc-editor.org/rfc/rfc9197.html) | In-situ path telemetry | Optional bounded-domain evidence; not assumed internet-wide |
| [RFC 9378 — IOAM Deployment](https://www.rfc-editor.org/rfc/rfc9378.html) | Deployment boundaries and considerations | Treat as domain-specific and privacy/security governed |
| [RFC 7276 — OAM Tool Overview](https://www.rfc-editor.org/rfc/rfc7276.html) | Ping/traceroute/OAM limitations | Document traceroute path/ECMP caveats and layer selection |
| [Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html/) | Capture/analysis mechanics | Use controlled offline analysis; record capture losses and metadata |
| [Wireshark display filters](https://www.wireshark.org/docs/man-pages/wireshark-filter.html) and [pcap filters](https://www.wireshark.org/docs/man-pages/pcap-filter.html) | Display versus capture filtering | Enforce capture-time minimization separately from later display filters |

### Tools, labs, and sources of truth

| Primary source | What was checked | Blueprint implication |
|---|---|---|
| [NetBox documentation](https://netbox.readthedocs.io/en/stable/) | DCIM/IPAM/circuit/tenancy and API surface | Use for declared fields with explicit ownership, not universal truth; pin the deployed release |
| [NetBox REST API](https://netbox.readthedocs.io/en/stable/integrations/rest-api/) | API version/request correlation and version-specific ETag conditional updates | Pin API/schema and use stable IDs; qualify `If-Match` against the deployed release |
| [NetBox event rules](https://netbox.readthedocs.io/en/stable/features/event-rules/) | Asynchronous event processing and failure | Events trigger refresh; authoritative re-read repairs loss/order |
| [NAPALM base documentation](https://napalm.readthedocs.io/en/latest/base.html) | Getters, candidate, compare, commit, confirm, discard, rollback | Per-driver capability manifest; common method names do not prove semantics |
| [Ansible network resource modules](https://docs.ansible.com/projects/ansible/latest/network/user_guide/network_resource_modules.html) | Resource states and before/after/command rendering | Avoid overbroad replacement/override; protect management paths |
| [Batfish README](https://github.com/batfish/batfish/blob/master/README.md) | Offline config analysis and use cases | Differential predeployment analysis with declared vendor/feature coverage |
| [Batfish symbolic engine](https://github.com/batfish/batfish/blob/master/docs/symbolic_engine/README.md) | Reachability, routes, loops, ECMP, policy questions | Build invariant/differential test suite |
| [Containerlab](https://containerlab.dev/) and [topology definition](https://containerlab.dev/manual/topo-def-file/) | Reproducible code-defined network labs | Pin topology/images/fixtures for adapter and failure tests |
| [Cisco pyATS documentation](https://developer.cisco.com/docs/pyats/) | Testbeds, parsing, testing workflows | Optional testing adapter; qualify vendor/library coverage rather than assuming neutrality |
| [Palo Alto Networks PAN-OS REST structure](https://docs.paloaltonetworks.com/ngfw/api/get-started-with-the-pan-os-rest-api/pan-os-rest-api-request-response-structure) | REST configuration edits and separate activation boundary | Do not treat CRUD success as active firewall policy |
| [Palo Alto Networks PAN-OS commit API](https://docs.paloaltonetworks.com/ngfw/api/pan-os-xml-api-request-types-and-actions/commit) | Queued commit job ID, status, warnings, and completion | Persist and reconcile the commit job; then verify active policy and paths |
| [F5 AS3 per-application declarations](https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/userguide/per-app-declarations.html) | Tenant-wide declaration versus narrower per-application update/source-of-truth behavior | Qualification declares update form and omission/deletion semantics |
| [HashiCorp: Terraform automation](https://developer.hashicorp.com/terraform/tutorials/automation/automate-terraform) | Saved plan/apply binding, provider identity, drift, remote state and locking | Treat plan/provider/state as versioned artifacts; network verification still required |

### Selected cloud/provider semantics

These sources illustrate why provider-specific manifests are required; they do not define a universal cloud adapter.

| Primary source | What was checked | Blueprint implication |
|---|---|---|
| [AWS Route 53 ChangeResourceRecordSets](https://docs.aws.amazon.com/Route53/latest/APIReference/API_ChangeResourceRecordSets.html) | Hosted-zone batch transaction, asynchronous change status, concurrent request behavior | Atomic scope is one batch/zone; poll native status; still verify auth/recursive/service path |
| [Route 53 fine-grained record permissions](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-permissions.html) | Record name/type/action IAM constraints | Scope DNS credential by zone/name/type/action where possible |
| [AWS VPC route-table operations](https://docs.aws.amazon.com/vpc/latest/userguide/WorkWithRouteTables.html) | Route-table association and connection-impact caution | Model association changes as connectivity-impacting effects |
| [Azure DNS zones and records](https://learn.microsoft.com/en-us/azure/dns/dns-zones-records) | ETag conditional update semantics | Use `If-Match`/`If-None-Match`; do not overwrite concurrent changes |
| [Azure Load Balancers REST API](https://learn.microsoft.com/en-us/rest/api/load-balancer/load-balancers/get) | Resource ETag, provisioning and probe-related representation | Pin API version; separate provisioning status from endpoint verification |
| [Google Cloud VPC routes](https://docs.cloud.google.com/vpc/docs/using-routes) | Route change ordering/convergence guidance | Sequence dependent changes and wait for documented propagation before next dependency |
| [Google Cloud DNS changes](https://docs.cloud.google.com/dns/docs/reference/rest/v1/changes) | Atomic collection change and status meaning | Provider `done` is not authoritative/recursive global propagation proof |
| [Google Cloud forwarding rules](https://docs.cloud.google.com/compute/docs/reference/rest/v1/forwardingRules) | Fingerprint/concurrency and resource mutability details | Use native optimistic concurrency and plan replacement for immutable fields |

### Security, reliability, and operations

| Primary source | What was checked | Blueprint implication |
|---|---|---|
| [CISA Enhanced Visibility and Hardening Guidance](https://www.cisa.gov/resources-tools/resources/enhanced-visibility-and-hardening-guidance-communications-infrastructure) | Central config, OOB management, RBAC/MFA, logging, management protections, certificate lifecycle | Isolate management, credentials, logging, and OOB recovery from production path |
| [CISA AA25-239A](https://www.cisa.gov/news-events/cybersecurity-advisories/aa25-239a) | Management VRF/OOB, egress restriction, control-plane protection | No management route leak; allowlist executor egress; default deny |
| [CISA BOD 23-02 alert](https://www.cisa.gov/news-events/alerts/2023/06/13/cisa-issues-bod-23-02-mitigating-risk-internet-exposed-management-interfaces) | Internet-exposed management risk | Keep administrative interfaces off public reachability |
| [OWASP Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/) | Direct/indirect injection and mitigations | Treat all network/ticket text as data; isolate model from credentials/effects |
| [OWASP Excessive Agency](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/) | Extension, permission, and autonomy minimization | Narrow typed tools, least privilege, deterministic authorization |
| [NIST AI 600-1](https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence) | GenAI risk-management profile | Governance and evaluation frame, not network execution spec |
| [Google SRE: Automation at Google](https://sre.google/sre-book/automation-at-google/) | Automation leverage, idempotency, maintenance and evolution | Prefer deterministic automation; continuously test against environmental drift |
| [Google SRE Workbook: Canarying Releases](https://sre.google/workbook/canarying-releases/) | Partial/time-bounded canary and control comparison | Stage changes and compare outcome, adapted to network fault domains |
| [Google SRE: Service Level Objectives](https://sre.google/sre-book/service-level-objectives/) | SLI/SLO/error-budget foundations | Define evidence, reconciliation, authorization, and recovery indicators |
| [Google SRE: Practical Alerting](https://sre.google/sre-book/practical-alerting/) | Actionable, symptom-aware alerting | Alert on user/control symptoms and reconciliation/safety backlog |
| [Google SRE: Launch Coordination Engineering](https://sre.google/sre-book/launch-checklist/) | Production launch review discipline | Require ownership, capacity, security, reliability, and operational readiness |

## Architecture decisions derived from the evidence

| Decision | Evidence basis |
|---|---|
| Untrusted model proposes; deterministic system authorizes and commits | OWASP agency/injection guidance plus heterogeneous protocol limits |
| Versioned layered topology with field-level authority and contradiction retention | RFC 8342/8345/8346, NetBox operational model |
| Evidence facts always include provenance, time, coverage, and gaps | gNMI timing, YANG-Push, telemetry/IPFIX measurement limits |
| Separate typed read, probe, packet, stage, validate, commit, confirm, rollback, reconcile tools | NETCONF/gNMI/API semantics and packet sensitivity |
| Exact adapter/target capability manifest with conformance evidence | Optional NETCONF features, OpenConfig/vendor/driver differences |
| Sealed plan DAG with make-before-break and fault-domain budgets | No multi-system transaction, xDS dependency order, canary practice |
| Durable intent-before-dispatch effect ledger and reconciliation | Asynchronous providers and lost-response ambiguity |
| Independent data-plane/service verification | Difference between controller acceptance and actual path behavior |
| DNS cache timeline and certificate served-identity checks | DNS TTL/negative/stale behavior, ACME vs RFC 9525 endpoint validation |
| Separate read/write identities and OOB recovery plane | CISA management hardening, least privilege, self-dependency risk |
| Packet payload excluded by default from model and telemetry | RFC 9232/IPFIX privacy and injection risk |
| One bounded planner; deterministic parallel probe scheduler | Minimize agency, authority paths, fan-out, and evaluation complexity |
| Lab/analysis coverage declaration and production canary | Batfish/Containerlab scope and emerging twin fidelity |
| Cells with one fenced writer owner per target | Tenant isolation, HA without duplicate mutation |
| Versioned replay/lab/shadow/canary for model, schema, adapter, and target upgrades | Target-dependent semantics and automation bit rot |
| Per-domain adapter qualification dossier and automatic capability demotion | Optional/target-specific protocol features, cloud concurrency models, firewall commit jobs, DDI and load-balancer scope differences |
| Loss-aware context manifest and compaction receipt over durable task/effect/evidence state | Long-running work, stale network state, prompt limits, and ambiguous effects cannot depend on conversational recall |
| Signed whole-system behavior bundle plus governed failure mining | Interacting component versions and potentially poisoned/misattributed production feedback |

## Remaining uncertainties and deployment qualification

The blueprint deliberately leaves several values deployment-specific:

- authoritative owner for each field and relationship;
- freshness/coverage thresholds for each operation;
- supported device/controller/vendor/API/model versions;
- acceptable route/DNS/TLS/traffic canary thresholds and observation windows;
- maintenance windows and fault-domain budgets;
- packet/flow legal basis, redaction, residency, and retention;
- exact RPO/RTO, SLOs, model/provider data policy, and cost budgets;
- which N3 actions, if any, qualify for automatic rollback;
- which analysis/lab constructs accurately cover deployed features.

These are not omissions a model should fill. They are operator, domain-owner, security, legal/privacy, and service-risk decisions validated against the deployment.

## Refresh checklist

Revisit this packet when any of the following occurs:

- target network OS, routing/DNS/LB/controller, cloud API, or certificate platform release;
- OpenConfig/YANG model or gNMI/NETCONF implementation change;
- adapter/driver/parser/resource-module upgrade;
- Kubernetes Gateway API or Envoy xDS implementation change;
- source-of-truth ownership/schema/event pipeline change;
- CA/B Forum requirement, ACME, DNSSEC, RPKI, or routing operational guidance change;
- AI model/provider, tool schema, orchestration, memory, or policy-engine change;
- new tenant/residency/privacy requirement or management architecture;
- topology scale, telemetry rate, retention, or probe budget materially changes;
- a production incident reveals an unmodeled propagation, state, dependency, or recovery behavior;
- network digital-twin standards or implementations mature enough to change assurance claims.

Record the new research date, exact versions tested, changed guarantees, evaluation artifacts, and any capability demotion or promotion.

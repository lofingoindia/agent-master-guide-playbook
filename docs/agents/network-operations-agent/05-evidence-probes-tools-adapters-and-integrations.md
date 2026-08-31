# Evidence, Probes, Tools, Adapters, and Integrations

## Evidence is layered, partial, and vantage-specific

Network diagnosis fails when one convenient signal is promoted into universal truth. Build evidence bundles that state what each source can and cannot prove.

| Evidence layer | Can support | Cannot prove alone | Important metadata |
|---|---|---|---|
| Declared intent | Expected connectivity and policy | That a target accepted or enforced it | Revision, owner, scope |
| Candidate/running config | What was staged or accepted | Correct protocol convergence or forwarding | Datastore, generation, target, read time |
| Protocol/RIB view | Learned/selected control-plane paths | FIB programming or packet delivery | AFI/SAFI, VRF, peer/view, collector coverage |
| FIB/adjacency state | Local programmed next hop | Downstream delivery or return path | Device, table, ASIC/context, timestamp |
| Counters and IPFIX/flow | Traffic seen during a window | Every packet, content, or causes of absence | Sampling, exporter loss, clock, interval, observation point |
| Active probe | Outcome from one vantage with one flow signature | All ECMP paths or all users | Source identity, resolver, protocol, tuple, time, rate |
| Packet capture | Packets visible at one point and interval | Packets elsewhere or intent | Filter, snap length, direction, drops, privacy controls |
| Service synthetic | User-like result from a vantage | Root cause | Name resolution, route, TLS, request, response timings |

Evidence quality improves through independent triangulation, not volume. A routing change might require intended diff, device applied state, RIB/FIB evidence, flow shift, and two-direction active probes. Packet capture is a last-resort discriminator when metadata cannot answer the question.

## Typed tool surface

Expose small, explicit operations. Inputs are bounded; outputs are structured data with provenance and sensitivity labels.

### Read and evidence tools

| Tool | Required bounds | Result highlights |
|---|---|---|
| `read.topology_snapshot` | Tenant, roots, layers, depth, time/skew | Version, nodes/edges, gaps, contradictions |
| `read.target_state` | Stable target ID, model paths, max age | Per-path value, origin, collection time, coverage |
| `read.route_view` | Target/collector, VRF, family, exact prefixes or capped query | RIB/FIB/advertised view, route attributes, missing coverage |
| `read.dns_rrset` | Zone/name/type, authoritative or named resolver vantage | RRset, TTL, SOA serial, DNSSEC validation, server identity |
| `read.tls_endpoint` | Host, port, SNI, trust profile, vantage | Served chain/fingerprint, identity result, protocol, timing |
| `read.traffic_policy` | Controller/resource, exact revision | Listeners/routes/pools/endpoints, health and condition versions |
| `read.flow_summary` | Observation points, tuple/prefix filters, time window, aggregation cap | Counts/rates, sampling, exporter gaps; no payload |
| `probe.reachability` | Approved source, destination, protocol/port, count/rate/deadline | Per-attempt result and path context |
| `probe.path` | Approved source/destination, protocol/flow keys, max TTL/count | Per-hop observations and ECMP limitations |
| `capture.packet_window` | Privileged approval, exact sensor/filter/duration/snaplen/byte cap | Encrypted artifact reference, capture loss, automatic expiry |

Do not expose “run command,” arbitrary XPath, unrestricted DNS recursion, arbitrary URL fetch, or unbounded target selectors to the planner. If a special command is genuinely required, add a reviewed typed operation with a schema and tests.

### Change tools

Keep proposal, staging, mutation, and outcome separate:

- `change.render`: convert normalized effects into adapter-specific candidate artifacts;
- `change.diff`: return semantic before/after and affected dependencies;
- `change.stage`: load candidate without activation when supported;
- `change.validate`: target-native plus platform semantic validation;
- `change.commit`: activate exactly the sealed candidate using expected versions;
- `change.confirm`: make a provisional/confirmed commit permanent after verification;
- `change.cancel`: cancel a pending native operation when supported;
- `change.rollback`: apply the separately sealed recovery plan after fresh prechecks;
- `change.reconcile`: read native/current state and classify an ambiguous or interrupted effect.

A vendor may combine some mechanisms, but the public contract preserves their meanings. An adapter that cannot stage or validate reports that capability as unsupported; it does not simulate success with a direct write.

## Adapter capability manifest

Capabilities are proven for an exact implementation tuple and expire after relevant upgrades.

```yaml
adapter_manifest:
  id: openconfig-gnmi-router
  adapter_version: 3.8.1
  protocol: gnmi
  target_match:
    vendor: example-net
    os: netos
    versions: ["12.4.2", "12.4.3"]
  models:
    openconfig-network-instance: "2025-01-30"
    openconfig-interfaces: "2024-12-05"
  operations:
    read:
      supported: true
      snapshot_atomicity: per_get_response_not_global
    stage:
      supported: false
    validate:
      supported: false
    commit:
      supported: true
      atomic_scope: one_SetRequest_on_one_target
      optimistic_concurrency: adapter_precondition_read
    confirm:
      supported: false
    rollback:
      supported: compensating_change_only
  limits:
    max_paths_per_get: 250
    max_updates_per_set: 50
    rate_per_target_per_minute: 20
  sensitive_paths:
    - "/system/aaa"
    - "/system/ssh-server"
  conformance:
    suite: adapter-gnmi-2026.08
    passed_at: 2026-08-28T04:13:00Z
    artifact: artifact://sha256/031a...
```

The registry should include:

- protocol and authentication mode;
- data-model/schema revisions and field mapping;
- read timing/snapshot semantics;
- candidate, validate, atomicity, confirmed-commit, rollback, cancellation, and optimistic-concurrency semantics;
- asynchronous operation identifiers and status mapping;
- safe request limits and rate limits;
- redacted/sensitive paths;
- known target defects and prohibited combinations;
- conformance evidence and expiry conditions.

The executor refuses a plan that claims semantics absent from the target's manifest.

The manifest is the execution projection of a larger [adapter qualification and vendor-onboarding dossier](11-adapter-qualification-and-vendor-onboarding.md). Before enabling writes, prove identity, pagination/completeness, schema loss, candidate/running/operational state, atomic scope, concurrency, asynchronous jobs, ambiguous outcomes, rollback, direct reconciliation, independent verification, least privilege, quotas, HA and recovery for the exact product/API/OS and enabled feature set. Read qualification and write qualification are separate decisions.

## Protocol semantics that matter

### NETCONF and YANG

NETCONF offers structured configuration with negotiated capabilities. Candidate configuration, validation, rollback-on-error, and confirmed commit are optional capabilities, not assumptions. Confirmed commit can automatically revert if it is not confirmed, but shared configuration requires locks and careful scoping: a rollback must not undo another actor's change. Use YANG models and target-reported capabilities to validate exact paths and types.

### gNMI and OpenConfig

gNMI provides typed Get, Subscribe, and Set over modeled paths. TLS and mutual authentication are part of the specification. A SetRequest is transactional on a single target as specified, not a multi-device transaction. Subscribe updates have independent timing; expose skew and loss. OpenConfig coverage and vendor deviations require target-specific tests.

### RESTCONF and vendor/cloud APIs

RESTCONF maps YANG data to HTTP semantics, but target support varies. Cloud and controller APIs commonly expose ETags/fingerprints, asynchronous operation resources, or per-resource generations. Use conditional requests whenever available; record the native operation ID; distinguish “API accepted,” “provider converged,” and “service verified.”

### CLI fallback

CLI is the least reliable adapter because syntax, output, prompts, privilege mode, pagination, and rollback vary. If unavoidable:

- allowlist command templates and exact parameters;
- pin target OS and parser versions;
- disable arbitrary shell expansion and interactive surprises;
- capture raw output as an artifact but return only parsed, schema-validated data;
- fail on unknown lines or unexpected prompts for write-related checks;
- require stronger lab coverage and a lower fault-domain budget;
- never claim atomicity or rollback unless it is demonstrated for that target.

## Integrations and their roles

| Integration | Appropriate role | Guardrail |
|---|---|---|
| NetBox or equivalent DCIM/IPAM | Declared inventory, addresses, circuits, tenancy, selected intent fields | Authority is field-specific; events trigger re-read; do not overwrite discovered truth silently |
| Git intent repository | Reviewable desired network policy and change history | Bind exact commit; protect branch; compilation still required |
| OpenConfig/gNMI telemetry | Modeled operational and configuration state | Record schema, target support, stream gaps, and skew |
| NETCONF/RESTCONF | Structured target reads and candidate/config operations | Negotiate capabilities; lock/version; avoid assumed rollback |
| NAPALM | Uniform getters and candidate operations where drivers support them | Maintain per-driver capability matrix; common API does not equal common semantics |
| Ansible network resource modules | Deterministic resource rendering and before/after data | Avoid broad `overridden` semantics unless explicitly safe; isolate management config |
| Kubernetes Gateway API | Desired L4/L7 policy and controller conditions | `Accepted`/`Programmed`/`ResolvedRefs` are controller evidence, not path proof |
| Envoy xDS | Typed/versioned traffic resources and ACK/NACK state | Respect dependency order, warming, and NACK; verify service path separately |
| Batfish | Offline config analysis and differential reachability | Declare supported vendors/features; analysis model is not live forwarding truth |
| Containerlab | Repeatable multivendor/emulated topology in CI | Lab fidelity and images are versioned; not production proof |
| Cloud DNS/network APIs | Provider-native resources and operation status | Record per-provider atomicity, concurrency, and propagation semantics |

Avoid choosing an “agent framework” as the architecture. The durable workflow, policy engine, typed tools, effect ledger, adapter conformance, and evidence plane are the product. The reasoning library is replaceable.

## Active measurement design

Use a small set of registered vantage points with stable identity and declared location, tenant, VRF/VPC, resolver, egress, and clock health. Probe selection should maximize information gain across the current hypotheses while respecting:

- per-target and per-protocol rate limits;
- customer/tenant and geographic boundaries;
- maintenance and incident budgets;
- amplification and spoofing protections;
- target allowlists and prohibited ports;
- cancellation when the answer is already discriminated.

STAMP can measure loss and delay; traceroute-like tools expose a possible path; service synthetics exercise DNS, connection, TLS, and protocol behavior. None yields a complete global view. Run reverse-direction or alternate-vantage tests when asymmetry matters.

## Packet and flow handling

IPFIX/flow records can provide valuable aggregate traffic evidence but may contain sensitive endpoints and identifiers. Include exporter identity, template, sampling, loss, time window, and observation point. Apply tenant authorization, minimization, encryption, and retention.

Packet capture requires a purpose-bound approval. Prefer a narrow BPF filter, minimal snap length, short duration, byte cap, and metadata-only analysis. Store the encrypted capture outside model context, expose only necessary derived fields, log every access, and delete on schedule. A device banner, DNS TXT value, HTTP header, certificate string, or packet payload may contain prompt injection; it remains data, never instructions.

## Evidence bundle example

```yaml
evidence_bundle:
  id: evb-9812
  question: "Can fra1 retail clients establish HTTPS to checkout through both edge paths?"
  snapshot: topo-672981
  observations:
    - ref: artifact://sha256/a1...
      kind: intent
      revision: git:80acb72
    - ref: artifact://sha256/a2...
      kind: rib_and_fib
      targets: [router-fra1-a, router-fra1-b]
      max_skew_seconds: 3
    - ref: artifact://sha256/a3...
      kind: active_tls_probe
      vantages: [fra1-retail-a, fra1-retail-b]
      attempts: 10
    - ref: artifact://sha256/a4...
      kind: flow_summary
      sampling: "1:1000"
      exporter_gap: false
  limitations:
    - "No visibility inside carrier AS 64520"
    - "ECMP paths sampled with four stable flow tuples"
  conclusion:
    statement: "Path through edge-b fails after FIB lookup; edge-a succeeds"
    confidence: high
    alternatives_not_excluded: ["intermittent carrier loss outside observation window"]
```

## Primary evidence

- [RFC 6241: NETCONF](https://www.rfc-editor.org/rfc/rfc6241.html)
- [RFC 7950: YANG 1.1](https://www.rfc-editor.org/rfc/rfc7950.html)
- [OpenConfig gNMI specification](https://openconfig.net/docs/gnmi/gnmi-specification/)
- [OpenConfig data models](https://openconfig.net/projects/models/)
- [RFC 9232: Network Telemetry Framework](https://www.rfc-editor.org/rfc/rfc9232.html)
- [RFC 7011: IPFIX Protocol](https://www.rfc-editor.org/rfc/rfc7011.html)
- [RFC 8762: Simple Two-Way Active Measurement Protocol](https://www.rfc-editor.org/rfc/rfc8762.html)
- [RFC 7276: An Overview of Operations, Administration, and Maintenance Tools](https://www.rfc-editor.org/rfc/rfc7276.html)
- [NAPALM base documentation](https://napalm.readthedocs.io/en/latest/base.html)
- [Ansible network resource modules](https://docs.ansible.com/projects/ansible/latest/network/user_guide/network_resource_modules.html)
- [Batfish README](https://github.com/batfish/batfish/blob/master/README.md)
- [Containerlab topology definition](https://containerlab.dev/manual/topo-def-file/)

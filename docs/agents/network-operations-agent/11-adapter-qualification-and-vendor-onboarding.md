# Adapter Qualification and Vendor Onboarding

> **Research baseline:** 2026-08-31  
> **Purpose:** Qualify a concrete network integration before it can provide evidence or execute a change, without pretending that similar API names imply similar state, atomicity, rollback, or verification semantics.

## The adapter is a semantic boundary

A client library, Terraform provider, Ansible module, generic REST connector, or vendor SDK is not automatically a safe network adapter. A production adapter translates one exact target/API/version tuple into the playbook's typed read, stage, validate, commit, confirm, cancel, reconcile, and verify contracts.

Qualification must prove:

1. **identity:** which tenant, account, controller, device, zone, view, VRF, resource, and generation the operation addresses;
2. **authority:** which fields the target owns and which identity can read, propose, stage, commit, or reconcile them;
3. **state semantics:** candidate, desired, accepted, running, operational, converged, and observed service state remain distinct;
4. **effect semantics:** exact atomic scope, preconditions, asynchronous acceptance, finality, cancellation, rollback, and ambiguous-outcome behavior;
5. **evidence semantics:** how raw responses become normalized facts without hiding warnings, gaps, skew, partial coverage, or unsupported fields;
6. **operability:** quotas, locks, deadlines, pagination, event loss, upgrade behavior, credentials, audit, HA, backup, and recovery are known and tested.

The executor refuses a target/version combination without a current qualification record. “It worked on another router,” “the SDK supports it,” or “the API returned success” is not qualification.

## Qualification dossier

Keep one immutable dossier per material adapter/target profile:

```yaml
adapter_qualification:
  qualification_id: aq-router-gnmi-exampleos-12_4_3-v7
  adapter_release: net-adapter/7.2.1+sha.91c2
  target_profile:
    product_family: example-router
    os_or_api_version: 12.4.3
    deployment_role: regional-edge
    management_plane: mgmt-vrf
  transport:
    protocol: gnmi
    protocol_version: 0.10.0
    authentication: workload-mtls
    endpoint_scope: exact-device
  negotiated:
    capabilities_sha256: "..."
    schema_modules:
      openconfig-interfaces: "2024-12-05"
  operations:
    read: {supported: true, snapshot_scope: one_get_response}
    stage: {supported: false}
    validate: {supported: false}
    commit: {supported: true, atomic_scope: one_set_on_one_target}
    confirm: {supported: false}
    cancel: {supported: false}
    reconcile: {supported: true, oracle: direct_read_plus_audit}
    rollback: {kind: newly_validated_compensating_change}
  concurrency:
    generation_or_etag: none
    target_lock: external-fenced-lease
    other_writer_detection: config_change_event_plus_read
  limits:
    get_paths: 250
    set_updates: 50
    writes_per_minute: 5
  evidence:
    conformance_suite: network-adapter-suite/9
    lab_artifact: artifact://sha256/...
    hardware_stage_artifact: artifact://sha256/...
    verified_at: 2026-08-29T12:00:00Z
    expires_at: 2026-11-29T00:00:00Z
  prohibited:
    - /system/aaa
    - /network-instances/*/protocols/bgp/global/config/as
```

The dossier links raw capability negotiation, schema/module sets, endpoint/API documentation, permissions, lab images, test output, known defects, security review, owner, approval, expiry, and refresh triggers. The manifest consumed by execution is a signed/minimized projection of this dossier.

## Common normalized contracts

### Read observation

```yaml
network_observation:
  observation_id: obs_01K
  tenant_id: retail-eu
  target_id: router-fra1-a
  field_path: /network-instances/default/routes/203.0.113.0_24
  plane: fib
  value_ref: artifact://sha256/...
  target_generation: gen-8821
  observed_at: 2026-08-31T10:40:03Z
  valid_for_operation_until: 2026-08-31T10:40:18Z
  coverage:
    afi_safi: ipv4-unicast
    vrf: default
    complete_for_query: true
  provenance:
    adapter_release: net-adapter/7.2.1
    qualification_id: aq-router-gnmi-exampleos-12_4_3-v7
    native_request_id: req-188
    raw_artifact: artifact://sha256/...
  warnings: []
```

“No result” is never normalized to absence without `complete_for_query`, target identity, table/view, time, pagination/cursor completion, and adapter warnings.

### Effect attempt

```yaml
network_effect_attempt:
  operation_id: op-77218
  attempt_id: op-77218/a1
  tenant_id: retail-eu
  plan_digest: sha256:3ad8...
  qualification_id: aq-route53-zone-v4
  target_id: dns-zone-Z123
  effect_type: dns_rrset_replace
  canonical_effect_sha256: "..."
  expected_before:
    rrset_digest: sha256:old...
    zone_generation: provider-specific-or-none
  idempotency_strategy: semantic_operation_plus_native_change_id
  dispatch_started_at: 2026-08-31T10:44:00Z
  native_operation_id: C0792
  transport_outcome: response_lost
  semantic_outcome: UNCERTAIN
```

The adapter returns operational meaning, not merely HTTP status:

| Normalized result | Required proof | Next action |
|---|---|---|
| `REJECTED_BEFORE_EFFECT` | target or adapter proves no mutation began | correct or bounded retry if still valid |
| `ACCEPTED_PENDING` | durable native job/commit/change ID exists | poll/callback plus direct reconciliation; never resubmit |
| `APPLIED_UNVERIFIED` | exact intended config/resource state is observed | run independent control/data/service verification |
| `NOT_APPLIED` | authoritative read/audit proves intended effect absent | retry only with current preconditions and same semantic operation |
| `PARTIAL` | subset or dependent resources changed | freeze scope; create fresh recovery plan |
| `SUPERSEDED` | another writer changed the resource after/beside the effect | preserve both actors; replan, never overwrite blindly |
| `INDETERMINATE` | available oracles cannot establish outcome | human recovery; no blind retry |
| `VERIFIED` | mandatory independent acceptance checks passed | terminalize; confirm provisional commit if applicable |

## Domain onboarding profiles

### Routing, switching, and device configuration

Qualify NETCONF/gNMI/RESTCONF/vendor APIs per exact device family and OS release. Capture negotiated capabilities and schema/module revisions at session start. Test:

- candidate versus running/startup/operational datastores;
- lock scope and conflict behavior with human and controller writers;
- validate, rollback-on-error, confirmed commit, persist tokens, and cancel-commit support;
- target-specific YANG deviations, defaults, replace/delete semantics, list keys, ordered data, and unknown fields;
- per-request atomicity and whether hardware/FIB programming can lag or fail after configuration acceptance;
- session loss before and after commit and direct reconciliation from an independent read identity;
- management-route and AAA changes under separate prohibited/emergency profiles.

RFC 6241 defines optional NETCONF capabilities; targets advertise what they implement. Junos documentation, for example, documents confirmed-commit behavior and a default rollback timer, but the adapter must still negotiate capability, lock the correct candidate state, record the actual deadline, and verify management plus service reachability before confirming. Do not generalize one vendor's behavior to every NETCONF target.

For CLI fallback, qualify exact command templates, privilege transitions, prompts, pagination disablement, parser grammar, locale, output truncation, configuration mode, commit behavior, and target build. Unknown output blocks writes. A CLI parser may be useful for evidence, but a broad `send_commands` surface is not a production tool contract.

### DNS and IPAM/DDI

DNS and IPAM have related objects but different truth and timing:

- IPAM allocation status does not prove address use, route reachability, DHCP activation, or DNS publication;
- an authoritative DNS API change does not prove secondary transfer, recursive cache replacement, DNSSEC validation, or service reachability;
- a provider transaction is scoped to its documented batch/zone/resource, not all dependencies.

Qualify stable object IDs, network and DNS views, tenant/VRF scope, prefix-overlap rules, allocation reservation, concurrent update protection, DHCP/DNS coupling, zone serial behavior, update prerequisites, batch atomicity, asynchronous change IDs, secondary/transfer state, negative/stale caching, and delete/restore behavior.

NetBox's current REST documentation exposes API-version and request-correlation headers and, from NetBox 4.6, weak ETags with `If-Match` for conditional single-object updates. Treat that as a version-specific feature, not a timeless NetBox property. Infoblox WAPI documentation warns that new fields can appear and clients should use the WAPI version whose behavior they expect. Preserve unknown fields and pin the API/profile rather than rejecting or overwriting them accidentally.

Amazon Route 53 documents all-or-none validation/application for one change batch in one hosted zone and an asynchronous change status. `INSYNC` is provider authoritative-server propagation evidence; named authoritative, recursive, DNSSEC, and service-path checks remain separate.

### Firewalls and policy controllers

Firewall adapters need a policy-object graph, not a text diff alone. Qualify:

- candidate versus active/running policy and central-manager versus device-local ownership;
- rule ordering, section boundaries, address/service objects, zones/VRFs, NAT, application/user identity, and implicit rules;
- policy shadowing, overlap, disabled objects, unresolved references, hit counters, session persistence, and HA synchronization;
- preview/validate/commit/push job semantics, job IDs, warnings, partial device-group success, cancellation, and current-state readback;
- semantic rollback after other policy commits and the effect on existing sessions.

PAN-OS official documentation illustrates why CRUD and activation must stay separate: REST operations modify configuration, while activation requires a commit through the XML API or another management interface; the commit is queued and returns a job ID whose status must be queried. The adapter records candidate mutation and commit job as separate effects, drains warnings/details, and then verifies active policy plus the allowed/denied paths. It never reports a successful REST edit as enforced policy.

Security owns malicious-activity conclusions and containment scope. The network adapter may apply only a specifically authorized ACL/policy effect and must preserve the exact rule, position, targets, expiry, and verification evidence.

### Load balancers and traffic controllers

Qualify listeners, addresses, routes, pools/clusters, endpoints, health checks, TLS references, persistence, connection draining, locality, weights, and dependency ordering. Record whether the API is imperative, declarative, or versioned discovery and what an ACK/status actually means.

Envoy xDS ACK/NACK and resource versions prove configuration protocol state, not listener reachability or application health. Kubernetes Gateway `Accepted`, `Programmed`, `ResolvedRefs`, and observed generation are controller evidence; probe the data path. F5 AS3 declarations can have tenant-wide source-of-truth semantics, while newer per-application endpoints narrow update scope. The qualification profile must declare which declaration form is used because omitting an application from a broad authoritative declaration can be materially different from a per-application update.

Test dependency warming, missing references, NACK, endpoint churn, draining, stateful/asymmetric flows, capacity during overlap, rollback while connections exist, and controller failover. A traffic weight change is verified through resource state, endpoint capacity, flow distribution, errors, latency, and service synthetics—not requested weight alone.

### Certificates and termination platforms

Separate certificate order/issuance, key custody, artifact storage, deployment, reload, selection, and served verification. Qualify:

- ACME account/order/authorization/challenge/finalize/certificate/revocation states and retry behavior;
- certificate/key identity, key non-exportability, signer/broker scope, chain and trust profile;
- target binding to listener/SNI/tenant and overlap with the previous valid certificate;
- reload/activation job, per-node propagation, clock, and session/ticket behavior;
- served leaf/chain/fingerprint/SAN/expiry from each relevant ingress and direct node where permitted;
- rollback prerequisites while preserving valid key/certificate lineage.

An issued certificate is an artifact, not an active endpoint. The model never receives a private key or challenge credential. Renewal scheduling is deterministic; an agent may diagnose exceptional inventory, policy, or deployment disagreement.

### Cloud networking

Do not build one generic `update_cloud_network` operation. Qualify provider/account/region/project/subscription resource identities and separate adapters for route tables, attachments, gateways, security/firewall policy, DNS, load balancers/forwarding rules, private endpoints, and network managers.

For each resource, record:

- create/update/replace/delete semantics and immutable fields;
- ETag/fingerprint/generation or absence of optimistic concurrency;
- request idempotency token and retention, if any;
- long-running operation identity, polling/callback, cancellation, and result retention;
- provisioning/configuration status versus documented data-plane convergence;
- eventual-consistency window and independent path verification;
- IAM resource scope, organization policy, region, quotas, and API version.

Azure DNS documents ETag conditional updates; Google forwarding resources expose provider-specific fingerprints/concurrency fields; AWS Route 53 exposes a zone-scoped transactional change batch. These mechanisms are not interchangeable and do not provide a network-wide transaction.

### Telemetry, probes, and configuration systems

Read adapters need the same rigor as writers. For gNMI/YANG-Push, IPFIX, BMP, streaming/controller telemetry, logs, synthetics, packet systems, and DCIM events, qualify subscription mode, sync marker/snapshot boundary, sequence and reboot semantics, timestamps/clock source, sampling, aggregation, exporter/collector loss, schema/template revision, backfill, retention, and tenant labels.

For configuration orchestration such as NAPALM, Ansible network resource modules, Git pipelines, or Terraform:

- pin library/module/provider versions and target driver/profile;
- preserve normalized plan/diff and exact rendered artifact;
- state which methods are native capabilities versus wrapper emulation;
- protect management, AAA, OOB, and unrelated fields from broad replace/override modes;
- use remote state/locking and saved-plan binding where applicable;
- treat target API results as effects requiring network reconciliation and path verification.

HashiCorp's automation guidance requires the same provider plugins for plan and apply, recommends remote state and locking, and warns that drift after a speculative plan can produce changes not reviewed in that plan. A Terraform plan is therefore an input artifact, not a substitute for the network sealed-plan, freshness, fault-domain, effect-ledger, and verification controls.

## Conformance and failure suite

### Mechanical contract tests

- authenticate as each read/write/reconcile role and prove forbidden scopes fail;
- enumerate and hash capabilities/schemas/API versions;
- test empty, singleton, maximum, paginated, partial, unknown-field, warning, and malformed responses;
- prove stable identity through rename, controller failover, device replacement, and resource recreation;
- exercise concurrent human/controller writes, stale ETag/generation, locks, lease expiry, and clock skew;
- crash before dispatch, after acceptance, during polling, after response, and before local receipt persistence;
- duplicate and reorder callbacks/events; expire native operation lookup; lose collector/exporter state;
- verify normalization round-trip or declared loss for every affected field.

### Domain outcome tests

| Adapter domain | Minimum independent acceptance |
|---|---|
| routing/device | exact config diff, adjacency/RIB/FIB, expected and prohibited reachability, management survival |
| DNS/IPAM | allocation/view identity, authoritative data/serial, secondary state, recursive timeline, DNSSEC and service path |
| firewall | active semantic rule/order, HA sync, allowed positive and prohibited negative paths, session behavior |
| load balancer | listener/route/pool/endpoints, health capacity, traffic distribution, TLS and application synthetic |
| certificate | exact binding, served fingerprint/chain/identity at every relevant termination point, old-certificate overlap |
| cloud network | resource generation/status, route/policy propagation, path and isolation tests from named vantages |
| telemetry/config | declared coverage/freshness/gaps, correct diff/plan binding, no unobserved writer or parser loss |

### Qualification exit gates

1. All typed reads preserve source, version, time, coverage, warning, and raw-artifact lineage.
2. Every write documents atomic scope, concurrency, idempotency, asynchronous state, reconciliation oracle, rollback type, and independent verification.
3. The lab and hardware/provider stage cover the exact target version and enabled features.
4. Ambiguous-boundary, duplicate, stale-writer, partial, cancellation, rollback, and recovery tests reach only supported terminal states.
5. Security proves least privilege, management isolation, secret handling, tenant boundaries, and supply-chain integrity.
6. Operations owns quota, SLO, capacity, incident, upgrade, demotion, and expiry runbooks.

Until all gates pass, use the adapter in quarantine-read, read-only, or proposal/lab mode. Qualification for reads does not imply qualification for writes.

## Release, expiry, and demotion

Include the adapter release and qualification ID in every topology fact, plan, attempt, trace, and effect record. Requalify after target OS/API, schema/model, controller, SDK/library/provider, authentication, permission, topology role, enabled feature, HA mode, or management-path change.

Automatically demote affected capabilities when:

- target capability/schema negotiation differs from the dossier;
- an unknown response field changes effect meaning or normalization loses data;
- conformance expires or a known defect/security advisory applies;
- reconciliation, verification, or recovery SLO breaches its policy;
- a production incident contradicts qualified semantics;
- the independent verification or OOB path is unavailable.

Demotion blocks new writes but leaves read-only evidence, reconciliation, rollback/forward-recovery, audit, and incident access available according to the emergency policy. Never disable the only mechanism that can establish what an uncertain prior effect did.

## Sources and volatility notes

- [RFC 6241: NETCONF capabilities, datastores, locks, validation, and confirmed commit](https://www.rfc-editor.org/rfc/rfc6241.html)
- [OpenConfig gNMI specification](https://openconfig.net/docs/gnmi/gnmi-specification/)
- [Junos NETCONF commit variants](https://www.juniper.net/documentation/us/en/software/junos/netconf/topics/ref/tag/netconf-commit.html)
- [NetBox REST API](https://netbox.readthedocs.io/en/stable/integrations/rest-api/)
- [Infoblox WAPI documentation portal](https://csp.infoblox.com/apidoc)
- [Amazon Route 53 `ChangeResourceRecordSets`](https://docs.aws.amazon.com/Route53/latest/APIReference/API_ChangeResourceRecordSets.html)
- [Palo Alto Networks PAN-OS REST request/response structure](https://docs.paloaltonetworks.com/ngfw/api/get-started-with-the-pan-os-rest-api/pan-os-rest-api-request-response-structure)
- [Palo Alto Networks PAN-OS commit API](https://docs.paloaltonetworks.com/ngfw/api/pan-os-xml-api-request-types-and-actions/commit)
- [Envoy xDS protocol](https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol.html)
- [Kubernetes Gateway API specification](https://gateway-api.sigs.k8s.io/reference/spec/)
- [F5 AS3 per-application declarations](https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/userguide/per-app-declarations.html)
- [RFC 8555: ACME](https://www.rfc-editor.org/rfc/rfc8555.html)
- [RFC 9525: Service Identity in TLS](https://www.rfc-editor.org/rfc/rfc9525.html)
- [Azure DNS zones and records](https://learn.microsoft.com/en-us/azure/dns/dns-zones-records)
- [Google Cloud forwarding rules](https://docs.cloud.google.com/compute/docs/reference/rest/v1/forwardingRules)
- [HashiCorp: running Terraform in automation](https://developer.hashicorp.com/terraform/tutorials/automation/automate-terraform)
- [HashiCorp: provider requirements and lock file](https://developer.hashicorp.com/terraform/language/providers/requirements)
- [RFC 9232: Network Telemetry Framework](https://www.rfc-editor.org/rfc/rfc9232.html)

Standards define protocol-level possibilities; vendor and cloud documents above are version-sensitive examples. Re-check them against the deployed product, API, negotiated capabilities, contract, and measured behavior before implementation and at every material upgrade.

# IT Service Desk and Endpoint Support Agent Blueprint — Research Packet

**Status:** evidence packet for the dedicated blueprint  
**Research cut-off:** 2026-08-31  
**Source access date:** 2026-08-31 unless a source row says otherwise  
**Blueprint:** [IT Service Desk and Endpoint Support Agent](../../agents/it-service-desk-agent/README.md)  
**Evidence policy:** prefer current primary sources; label drafts, previews, incomplete telemetry, and benchmark scope explicitly

## Research objective

Determine the smallest production-safe architecture for an agent that can authenticate service-desk intake, bind a requester to the correct managed endpoint, collect bounded diagnostic evidence, propose registered remediation, coordinate consented remote support, maintain case state, and escalate safely.

The research deliberately asks a narrower question than “Can an AI run the help desk?” The blueprint must preserve four boundaries:

1. identity governance remains with IAM systems and their owners;
2. interactive desktop control remains with an authenticated human helper and the approved remote-support product;
3. fleet policy, bulk change, wipe, arbitrary shell, and infrastructure administration remain outside the agent;
4. identity- or endpoint-sensitive effects require exact, expiring approval and an independent deterministic verifier.

## Method

Research proceeded in five passes:

1. **Standards and threat model:** identity proofing and recovery, zero trust, least privilege, separation of duties, remote maintenance, privacy, and incident logging.
2. **System semantics:** ITSM state machines and APIs; endpoint inventory, diagnostics, action, and remote-help APIs; identity-provider recovery capabilities.
3. **Failure semantics:** asynchronous completion, provider retry behavior, stale inventory, duplicate events, rate limits, consent loss, partial audit data, and uncertain outcomes.
4. **Agent evidence:** applicable evaluation guidance and public agent benchmarks, with explicit limits on what those benchmarks establish.
5. **Synthesis:** translate evidence into authority classes, contracts, stage gates, failure tests, and operating runbooks.

Marketing claims, generic “AI help desk” articles, and unsourced maturity claims were excluded. Vendor documentation was treated as evidence for that vendor's interface, not as an independent safety assessment.

## Questions investigated

- What proves the requester, affected account, tenant, and endpoint are the intended subjects?
- Which reads are safe to automate, and which collections are themselves consequential remote actions?
- What can an approval authorize, and what must still be rejected by policy or verification?
- How do ticket, identity, device, remote-help, and orchestration state reconcile after duplicates, timeouts, and partial failures?
- What information can enter prompts, traces, case notes, durable memory, and evaluation datasets?
- Which public benchmarks are relevant, and what important service-desk properties do they not test?
- What evidence is required before each maturity stage can advance?

## Category decision and ownership boundary

The subject warrants a dedicated blueprint because its core risk is neither generic computer use nor generic IAM. It is the binding of a real support case to an authenticated user and an exact managed endpoint while multiple authoritative systems evolve asynchronously.

| Neighboring category | Owns | This blueprint may do | This blueprint must not absorb |
| --- | --- | --- | --- |
| Computer-use agent | UI perception and interaction in a bounded environment | Link to a separately authorized human remote-help session | Observe a user's screen or control pointer and keyboard |
| IAM agent | Entitlement, policy, lifecycle, and authenticator governance | Create a verified recovery handoff and track its status | Decide proofing sufficiency, reset factors, mint recovery credentials, or change entitlement policy |
| Infrastructure operations | Fleet configuration, deployment, privileged administration | Submit a registered per-device remediation proposal | Wipe, bulk change, alter policy, run arbitrary shell, or administer the fleet |
| Security investigation | Threat triage, evidence preservation, containment decisions | Escalate suspected social engineering, compromise, or tampering with evidence references | Conduct an investigation or silently contain an account/device |
| Service desk and endpoint support | Authenticated intake, binding, evidence, proposals, consent coordination, case state, escalation | Own the bounded workflow described above | Infer identity or turn user assistance into unrestricted authority |

The resulting design uses one bounded diagnostic model inside a deterministic case workflow. The model can choose registered read tools, label hypotheses, propose a registered action, and abstain. Application code owns policy, binding, approval, dispatch, reconciliation, closure, and escalation.

## Normative evidence synthesis

### Identity is an explicit binding, not a likelihood

[NIST SP 800-207](https://csrc.nist.gov/pubs/sp/800/207/final) rejects implicit trust based on network location or asset ownership and treats subject and device authorization as distinct decisions. The blueprint therefore binds four identifiers independently: tenant, authenticated requester, affected principal, and managed-device record. Email address, caller ID, IP address, device name, possession of biographical facts, voice resemblance, and behavioral similarity are signals at most; none becomes identity proof.

[NIST SP 800-63A-4](https://pages.nist.gov/800-63-4/sp800-63a.html) provides the proofing and redress foundation, while [SP 800-63B-4](https://pages.nist.gov/800-63-4/sp800-63b.html) defines authenticator and account-recovery requirements. Recovery is not an ordinary service-desk shortcut. It uses the identity provider's approved recovery mechanisms and notification behavior. Knowledge-based authentication and “security questions” are not accepted as authenticator-management proof. The service-desk agent therefore creates a handoff; it does not collect recovery secrets or decide that the user has proved enough.

### Help desks are an active identity attack surface

The [joint government Scattered Spider advisory](https://www.cyber.gov.au/about-us/view-all-content/alerts-and-advisories/scattered-spider) describes threat actors gathering personal information, learning reset processes, and socially engineering help desks to reset passwords and MFA. The [UK NCSC incident guidance](https://www.ncsc.gov.uk/blog-post/incidents-impacting-retailers) specifically calls for review of help-desk reset authentication, particularly for privileged accounts.

This evidence changes the design in four ways:

- a valid ticket does not prove an account-recovery request;
- privileged, suspicious, or recovery-related cases route to stronger independent verification;
- the same actor cannot both assert identity and approve the sensitive effect;
- unusual recovery volume, target privilege, channel changes, and repeated failed proofing are security signals and escalation reasons, not prompts for the agent to improvise.

### Approval does not create authority

[NIST SP 800-53 Rev. 5.1](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final) supports separation of duties, least privilege, and controlled nonlocal maintenance. Its remote-maintenance controls call for authorization, monitoring, strong authentication, maintenance records, and termination of connections. The blueprint's approval object can narrow pre-existing authority; it cannot authorize an out-of-policy action, an ambiguous target, an unsigned runbook, or a D4 operation.

Every D3 effect carries an exact digest over tenant, requester, affected principal, device, action, mode, parameters, expected data access, disruption, runbook version, expiration, permitted uses, and signers. Immediately before dispatch, an independent policy path recomputes the digest and rechecks live bindings, scope, concurrent effects, connector identity, and revocation.

### Remote support is a human-operated maintenance session

[Microsoft Intune Remote Help planning](https://learn.microsoft.com/en-us/intune/remote-help/plan) documents authenticated tenant membership, role-based capabilities, and user consent for remote-help roles such as view, full control, and elevation. It also exposes unattended capabilities for specific deployment patterns. This blueprint selects the safer category rule: user-affine sessions are attended and human-operated. Unattended support is a separate fleet policy for dedicated or kiosk devices and is outside the normal service-desk path.

[Microsoft's Remote Help reporting guidance](https://learn.microsoft.com/en-us/intune/remote-help/troubleshoot) documents session metadata but not screen or keystroke content; some important details, including elevation, are not necessarily represented in the report. Therefore, the blueprint never claims that provider audit metadata is a complete reconstruction. It records application-side proposal, approval, launch, participant, termination, and verification events without recording screen content.

[CISA's remote-access software guidance](https://www.cisa.gov/resources-tools/resources/guide-securing-remote-access-software) reinforces approved-tool inventory and logging. The agent cannot launch arbitrary remote-management software or pass credentials through a ticket.

### Endpoint commands are asynchronous and provider-specific

[Intune device actions](https://learn.microsoft.com/en-us/intune/device-management/actions/) include remote actions with different availability and completion semantics. Some “completed” indications represent service-side acceptance rather than confirmed device-side completion. [Intune diagnostics collection](https://learn.microsoft.com/en-us/intune/device-management/actions/collect-diagnostics) is itself an action that gathers remote data and therefore is D3 in this blueprint, not a passive read.

[Apple device-management command processing](https://developer.apple.com/documentation/devicemanagement/sending-mdm-commands-to-a-device) requires correlation with `CommandUUID` and distinguishes acknowledged, error, and `NotNow` outcomes. [Android Management API commands](https://developers.google.com/android/management/reference/rest/v1/enterprises.devices/issueCommand) return a long-running operation and expose command expiry, management-mode restrictions, and possible user action. These are concrete reasons to keep `accepted`, `running`, `verified`, `failed`, and `unknown` separate.

No provider response alone closes the case. Closure requires a postcondition from an authoritative source or an authenticated user confirmation appropriate to the action. When the result is unknown, reconciliation precedes retry.

Current documentation also constrains what can honestly be implemented. Microsoft's August 2026 Intune diagnostics guidance says collection/download is not available directly through Microsoft Graph, can contain user-identifiable information, and uses Microsoft support storage outside ordinary Intune data-management protections. Conversely, even Graph `syncDevice` currently requires `DeviceManagementManagedDevices.PrivilegedOperations.All`, an admin-consented permission that also reaches high-impact actions. A production design must therefore use a separately protected allowlisting broker or leave the action manual; it must not invent a narrow Graph diagnostics API or mistake one registered route for a narrow provider privilege.

### Device identity requires authoritative identifiers and freshness

The [Microsoft Graph managed-device resource](https://learn.microsoft.com/en-us/graph/api/resources/intune-devices-manageddevice?view=graph-rest-1.0) exposes several distinct identifiers and state fields, including managed-device ID, directory device ID, serial number, user association, ownership, and last-sync information. [ServiceNow CMDB identification and reconciliation](https://www.servicenow.com/docs/r/servicenow-platform/configuration-management-database-cmdb/c_CompsandProcessIDandReconcil.html) and its [identification rules](https://www.servicenow.com/docs/r/servicenow-platform/configuration-management-database-cmdb/c_IdentificationRules.html) exist because raw names and attributes are not reliably unique or current.

The blueprint therefore stores an immutable provider record identifier plus the identifiers used to cross-check it, the authoritative source, observation time, freshness policy, and binding decision. A hostname or serial typed into a ticket never becomes the action target without authoritative resolution.

### Scripts are supply-chain artifacts, not model output

[Intune Remediations](https://learn.microsoft.com/en-us/intune/device-management/tools/deploy-remediations) supports detection and remediation scripts, but its documentation also exposes risks around signature enforcement, execution-policy behavior, output limits, overlapping same-device actions, and handling of personal data or secrets. The safe conclusion is not “let the model write a script.” It is to expose only reviewed, signed, versioned, input-bounded runbooks from an allowlisted catalog; record the artifact digest; serialize effects per device; and verify explicit postconditions.

### Ticket and event systems are not exactly-once logs

[ServiceNow's incident state model](https://www.servicenow.com/docs/r/it-service-management/incident-management/c_IncidentManagementStateModel.html) and [Jira Service Management's request API](https://developer.atlassian.com/cloud/jira/service-desk/rest/api-group-request/) demonstrate that status, assignment, approval, SLA, and transitions belong to the ITSM system. The local runtime should not replace that source of truth.

[Microsoft Graph webhook delivery](https://learn.microsoft.com/en-us/graph/change-notifications-delivery-webhooks) documents acknowledgement deadlines, retry and drop behavior, and subscription renewal. [Atlassian webhooks](https://developer.atlassian.com/cloud/jira/platform/webhooks/) can be retried and include identifiers useful for deduplication. [CloudEvents 1.0.2](https://github.com/cloudevents/spec/tree/ce%40v1.0.2) provides a portable envelope, not exactly-once delivery.

Consequently, the runtime persists intent before dispatch, uses a semantic idempotency key, records provider correlation IDs, deduplicates incoming events, orders by aggregate/version rather than arrival time, and runs scheduled reconciliation. A webhook is a hint to read authoritative state, not proof of truth.

Zendesk provides a third useful counterexample to a universal ITSM adapter. Its agent-facing ticket and end-user-facing request have different visibility, comments can be public or private, safe updates use `safe_update` plus `updated_stamp`, ticket updates have endpoint-specific limits, and Ticket Audits require the global OAuth `read` scope rather than a narrowly named ticket-audit scope. ServiceNow instances add custom tables, ACLs, business rules, states and configured inbound rate rules; Jira Service Management combines request and Jira issue semantics, custom workflows, app-access rules and operation-specific scopes. The only safe normalization is a canonical case contract backed by a live instance mapping, negative permission tests and read-after-write reconciliation.

### Remote-support controls are independent and tenant-specific

Remote-product permissions that sound related are not necessarily coupled. Current ScreenConnect role documentation separates unattended consent bypass from out-of-session commands, tool execution, file transfer, credential handling, elevation and access-installer permissions. Current TeamViewer documentation makes report/event coverage dependent on license, configuration, assigned devices, authentication and logging. Microsoft Remote Help requires specific same-tenant, scope-group, cloud, platform and role combinations, while its provider report has bounded retention and omitted fields.

The resulting qualification rule is empirical: export the effective role, prove excluded capabilities fail, exercise attended consent/mode upgrade/termination, inspect actual provider events, and record retention/export gaps. CISA guidance shows that any legitimate RMM can be abused; a product brand or SSO checkbox is not admission evidence.

### Prompts, traces, and durable memory are privacy boundaries

The [NIST Privacy Framework](https://www.nist.gov/privacy-framework) is used as a risk-management reference, not a compliance certificate. At the research cut-off, version 1.0 is the stable framework exposed by NIST while [version 1.1 remains an initial public draft](https://www.nist.gov/news-events/news/2025/04/nist-privacy-framework-11-initial-public-draft-available-comment). The blueprint consequently versions privacy mappings instead of claiming conformance to a draft.

[W3C Trace Context](https://www.w3.org/TR/trace-context/) supplies interoperable propagation, and [OpenTelemetry's generative-AI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) can help correlate model and tool spans. Relevant generative-AI conventions remain development-status material, and tool arguments/results can contain sensitive data. The blueprint uses stable internal attribute names, allowlisted low-cardinality fields, redaction before export, and evidence references instead of raw ticket bodies, credentials, diagnostic archives, screenshots, or command output.

Durable user “memory” is not needed to resolve a support case. The design retains deterministic case/effect records under explicit schedules. Optional support preferences require purpose, consent, and deletion controls. Procedure knowledge is versioned and reviewed; a model cannot convert a successful one-off action into a new executable runbook.

## Evidence-to-contract decisions

| Evidence | Contract decision | Resulting test |
| --- | --- | --- |
| Zero trust distinguishes subject and device | Tenant, requester, affected principal, and device are separate bindings | Swapping any one identifier invalidates approval |
| Recovery has defined proof and notification paths | Agent creates recovery handoff only | Recovery secret in prompt/ticket fails the run |
| Help-desk social engineering is observed in active campaigns | Recovery and privileged targets require stronger verification and anomaly escalation | Caller knowledge, urgency, or executive claim cannot bypass proof |
| Remote maintenance requires authorization, monitoring, and termination | Remote session is attended, role-bounded, expiring, and explicitly terminated | Disconnect and revocation are verified independently |
| Endpoint APIs are asynchronous | Accepted, device-acknowledged, and verified are distinct states | Lost callback produces `unknown`, never `resolved` |
| Provider reports may omit details | Provider and application audit evidence are both retained | Audit completeness is asserted only field by field |
| Scripts can bypass policy or leak data | Only signed catalog runbooks can execute | Model-generated or hash-mismatched script is rejected |
| Webhooks retry, drop, and expire | Events are deduplicated and authoritative state is reconciled | Duplicate, reordered, and missing events converge safely |
| Rate limits are dynamic | Retry hints, tenant budgets, and protected queues are required | 429 storms cannot starve recovery/security work |
| Device records can be stale or ambiguous | Binding includes provider ID, cross-checks, and freshness | Duplicate hostname/serial cannot select a device silently |

## Contradictions and caveats resolved

| Tempting conclusion | Contrary evidence or caveat | Blueprint resolution |
| --- | --- | --- |
| “An authenticated ticket proves the recovery requester.” | A session may be stolen, delegated, or created before lockout; recovery requires its own approved proof. | Bind ticket identity for case access, then use a separate recovery broker and stronger proof. |
| “A vendor's self-service reset flow is enough for every assurance level.” | Recovery methods and combinations depend on assurance and organizational policy. | The IdP/IAM owner defines recovery policy; the agent never downgrades it. |
| “MDM says completed, so the device changed.” | Some service-side statuses do not prove device-side postconditions. | Track acceptance, device acknowledgement, and verification separately. |
| “User consent permits unattended control later.” | Consent is action-, mode-, target-, and time-specific; unattended support changes the risk model. | User-affine remote help is attended; unattended is a separate D4/fleet policy. |
| “The remote-support report is a complete audit.” | Vendor reports can omit screen content, keystrokes, or elevation details and have bounded retention. | Retain application events and label exactly which provider fields exist. |
| “Remediation scripts are safe because the MDM runs them.” | Unsigned/bypass modes, sensitive output, overlap, and arbitrary code remain risks. | Signed/versioned catalog only; no model-authored shell; serialize per device. |
| “Hostname, serial, or ticket text uniquely identifies a device.” | Inventory can be stale, duplicated, re-enrolled, or cross-tenant. | Resolve an immutable provider record and cross-check source, tenant, owner, and freshness. |
| “A webhook is the final state.” | Retries, duplicates, delays, drops, and subscription expiry occur. | Deduplicate, read authoritative state, and reconcile periodically. |
| “Intune diagnostics can be automated through Graph.” | Current Intune documentation says collection/download is not available directly through Microsoft Graph and identifies additional privacy/storage constraints. | Keep it human/admin-center operated or place a separately supported narrow facade in front; never invent the API. |
| “A Graph scope for one device action is narrow.” | `syncDevice` uses `DeviceManagementManagedDevices.PrivilegedOperations.All`, whose application permission also reaches high-impact actions. | Isolate the broker, enforce endpoint/action allowlists below the model, prove excluded routes fail, or leave the action manual. |
| “Microsoft Entra Account Recovery is generally available.” | Microsoft's release announcement labels Account Recovery Public Preview; tenant profile, external IDV, production mode, user history/claims and TAP policy affect eligibility. | Treat it as an optional preview integration, recheck status, and retain an IAM-owned alternative/redress path. |
| “A remote-support consent setting disables the other dangerous features.” | ScreenConnect documents separate permissions for consent bypass, commands, tools, files, credentials and unattended installers; other vendors have different coupling. | Export and negative-test the effective role/capabilities one by one. |
| “Zendesk audit access is naturally ticket-scoped.” | Ticket Audits currently require the global OAuth `read` scope; ticket/request visibility and public/private comments differ. | Decide whether the scope is acceptable, isolate the reader, and keep comment visibility explicit; otherwise omit audit ingestion. |
| “WorkArena proves autonomous service-desk readiness.” | [WorkArena](https://www.servicenow.com/research/publication/alexandre-drouin-work-icml2024.html) evaluates a set of ServiceNow knowledge-work UI tasks, not identity proof, device targeting, approval integrity, or remote-effect reconciliation. | Use it only for applicable UI/task reasoning; maintain a dedicated safety suite. |
| “ITBench proves endpoint-support readiness.” | [ITBench](https://research.ibm.com/publications/benchmarking-ai-agents-for-it-automation-tasks-with-itbench) focuses on broader IT automation domains and operational telemetry, not this binding-and-consent boundary. | Reuse applicable incident/evidence tasks, not its scope as a maturity claim. |
| “An ITSM benchmark score measures production safety.” | Public ITSM-style task suites vary in realism and rarely test recovery abuse, wrong-device effects, or uncertain completion. A stable primary source for the cited name “ITSMBench” was not established in this research pass. | Do not cite an unverified benchmark or infer readiness from task success alone. |
| “NIST SP 800-46 Rev. 3 is the current final remote-access standard.” | [Rev. 2 is final](https://csrc.nist.gov/pubs/sp/800/46/r2/final); the [Rev. 3 record is preliminary draft activity](https://csrc.nist.gov/pubs/sp/800/46/r3/iprd). | Pin Rev. 2 for normative mapping and track a future final revision. |
| “NIST SP 800-92 Rev. 1 is final log-management guidance.” | [SP 800-92 is final](https://csrc.nist.gov/pubs/sp/800/92/final), while [Rev. 1 remains an initial public draft](https://csrc.nist.gov/pubs/sp/800/92/r1/ipd). | Use the final publication as the labeled baseline and the draft only as a versioned planning input. |
| “NIST Privacy Framework 1.1 is final.” | NIST still labels 1.1 an initial public draft at the cut-off. | Use 1.0 as stable baseline and track 1.1 as draft. |
| “OpenTelemetry GenAI fields are stable and safe to export.” | Convention status evolves; tool inputs and outputs may be sensitive. | Keep an internal versioned schema and export only allowlisted redacted fields. |
| “Vendor REST schemas are stable.” | Jamf distinguishes current, preview, and deprecated endpoints; Atlassian and Microsoft publish changing throttling policy. | Pin tested API/schema versions, run contract tests, and keep connector-specific rate budgets. |
| “Two approvers make any action safe.” | Approval cannot repair a wrong target, invalid binding, revoked authority, unsafe artifact, or prohibited D4 action. | Independent verification is mandatory after approval and immediately before dispatch. |

## Architecture decision record

### ADR-1: One bounded diagnostic model

**Decision:** use one model for classification, evidence-plan selection, hypothesis ranking, response drafting, and registered-action proposals.

**Why:** service-desk risk is dominated by deterministic identity, authority, and effect controls. Multiple free-running agents add coordination and provenance failure modes without solving those controls.

**Rejected:** an intake agent, diagnostic agent, remediation agent, and closure agent sharing conversational memory. This makes authority boundaries harder to audit and does not create independent verification.

### ADR-2: Deterministic control plane owns effects

**Decision:** typed application code owns case transition, policy, approval, dispatch, reconciliation, and closure.

**Why:** these operations need replayable invariants, not probabilistic intent interpretation.

### ADR-3: Provider systems remain authoritative

**Decision:** ITSM owns ticket status, assignment, and SLA; IdP owns principal/authenticator state; MDM owns device state; remote-help system owns session state; local orchestration owns its effect ledger.

**Why:** copying every field into a new “agent database” would create divergent truth. Local records store normalized references, decisions, and correlations needed for workflow safety.

### ADR-4: Evidence and effect are different lanes

**Decision:** passive reads are D1. Fresh diagnostic collection, remote view/control, restart, registered remediation, and recovery-sensitive requests are D3. Ticket annotations and proposals are D2. Wipe, bulk change, policy change, arbitrary code, and autonomous recovery are D4/out of scope.

**Why:** a read that makes a device collect and upload new data is a remote effect even if the provider calls it “diagnostics.”

### ADR-5: Remote-help content stays outside the model

**Decision:** the model may coordinate session prerequisites and summarize human-entered outcomes; it never sees the screen, keystrokes, clipboard, or pointer stream.

**Why:** this preserves the computer-use boundary and sharply reduces secret, privacy, prompt-injection, and unrestricted-control risk.

### ADR-6: Reconciliation before retry

**Decision:** any timeout after possible dispatch becomes `unknown`; the runtime queries provider state and verifies postconditions before retrying.

**Why:** transport idempotency alone cannot prove that a device did not execute an action.

### ADR-7: Learning cannot mutate authority

**Decision:** production outcomes feed offline evaluation and reviewed KB/runbook proposals. They cannot automatically change policies, approval requirements, proofing, tool scope, or executable artifacts.

**Why:** a successful trajectory is not evidence that a broader authority grant is safe.

## Version and volatility baseline

| Surface | Baseline at research cut-off | Volatility treatment |
| --- | --- | --- |
| NIST digital identity | SP 800-63 Revision 4 final suite, July 2025 | Recheck errata and implementation resources quarterly |
| Zero trust | NIST SP 800-207 final | Recheck related implementation guides annually |
| Security controls | NIST SP 800-53 Rev. 5.1 | Pin control mappings by revision |
| Remote access | NIST SP 800-46 Rev. 2 final; Rev. 3 preliminary draft record | Do not silently map to draft |
| Log management | NIST SP 800-92 final; Rev. 1 initial public draft | Separate sampled traces, operational logs, and protected audit; label draft use |
| Privacy | NIST Privacy Framework 1.0 stable; 1.1 initial public draft | Track final release before remapping |
| CloudEvents | 1.0.2 | Pin event schema separately from envelope version |
| W3C Trace Context | Recommendation | Preserve vendor extensions behind internal schema |
| OTel GenAI conventions | development/evolving | Pin collector/schema version and redact before export |
| ServiceNow | Australia documentation observed; live instance release, schemas, ACL/business rules and configured rate limits vary | Pin the target instance mapping; qualification expires on instance upgrade/configuration change |
| Atlassian Cloud | current REST/webhook docs; changing rate policy | Treat limits as runtime signals, not constants |
| Zendesk Support/Guide | current Tickets/Requests/Audits/Articles and rate-limit documentation | Pin subdomain/plan/roles/triggers/custom statuses; safe-update and visibility contract tests |
| Microsoft Graph/Intune | Graph v1.0 plus current August 2026 service documentation; license/cloud/platform dependent | Separate read/effect identities; contract tests, permission-route negative tests, changelog review and rate probes |
| Microsoft Entra Account Recovery | Public Preview; TAP Graph v1.0 is a separate sensitive mechanism | Do not make preview the only recovery path; recheck tenant eligibility/status and isolate the TAP broker |
| Microsoft Remote Help | current documented platform/mode/RBAC matrix; product report retention/fields are bounded | Pin tenant/license/cloud/platform/helper-sharer combination and export application-side evidence |
| Jamf Pro | instance-specific OpenAPI; current/preview/deprecated routes | Production allowlist excludes preview/deprecated routes |
| ScreenConnect/TeamViewer | current vendor role/event/report docs; tenant license/configuration controls effective behavior | Live negative capability and audit/termination tests; no brand-level admission |
| Apple/Android device management | current platform command contracts | Test by OS, enrollment, ownership, and management mode |

## Source register

### Standards, government, security, and privacy

| Source | Evidence used | Strength and limitation |
| --- | --- | --- |
| [NIST SP 800-63-4 suite](https://pages.nist.gov/800-63-4/) | Current identity guidance family and status | Primary final standard suite |
| [NIST SP 800-63A-4](https://pages.nist.gov/800-63-4/sp800-63a.html) | Identity proofing and redress | Primary; organization still chooses assurance policy |
| [NIST SP 800-63B-4](https://pages.nist.gov/800-63-4/sp800-63b.html) | Authenticator lifecycle, recovery, notification, KBA prohibition | Primary; not a vendor implementation guide |
| [NIST SP 800-207](https://csrc.nist.gov/pubs/sp/800/207/final) | No implicit trust; resource and subject decisions | Primary architectural standard |
| [NIST SP 800-53 Rev. 5.1](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final) | Least privilege, separation, nonlocal maintenance, audit | Primary control catalog; tailoring required |
| [NIST SP 800-46 Rev. 2](https://csrc.nist.gov/pubs/sp/800/46/r2/final) | Final remote-access/BYOD security baseline | Primary but dated; resolved against draft status |
| [NIST SP 800-46 Rev. 3 preliminary record](https://csrc.nist.gov/pubs/sp/800/46/r3/iprd) | Draft-status caveat and future refresh trigger | Not used as final normative guidance |
| [NIST SP 800-92](https://csrc.nist.gov/pubs/sp/800/92/final) | Final log-management baseline | Primary but dated; implementation needs current platform controls |
| [NIST SP 800-92 Rev. 1 IPD](https://csrc.nist.gov/pubs/sp/800/92/r1/ipd) | Current planning-guide draft and refresh trigger | Draft; not presented as final guidance |
| [NIST Privacy Framework](https://www.nist.gov/privacy-framework) | Privacy-risk management baseline | Voluntary framework, not certification |
| [NIST Privacy Framework 1.1 IPD notice](https://www.nist.gov/news-events/news/2025/04/nist-privacy-framework-11-initial-public-draft-available-comment) | Draft-status resolution | Primary status source |
| [Joint Scattered Spider advisory](https://www.cyber.gov.au/about-us/view-all-content/alerts-and-advisories/scattered-spider) | Help-desk social engineering and recovery abuse | Multi-government threat advisory |
| [NCSC retail incident guidance](https://www.ncsc.gov.uk/blog-post/incidents-impacting-retailers) | Reset-process review, privileged-account focus | Government operational guidance |
| [CISA remote-access software guide](https://www.cisa.gov/resources-tools/resources/guide-securing-remote-access-software) | Approved remote tools, inventory, logging, hardening | Government guidance; product controls still vary |

### ITSM, state, and event delivery

| Source | Evidence used | Strength and limitation |
| --- | --- | --- |
| [ServiceNow incident state model](https://www.servicenow.com/docs/r/it-service-management/incident-management/c_IncidentManagementStateModel.html) | Ticket lifecycle remains ITSM-owned | Vendor-specific state model |
| [ServiceNow REST APIs](https://www.servicenow.com/docs/r/api-reference/rest-apis/api-rest.html) | Connector surface | Instance configuration can differ |
| [ServiceNow inbound REST rate limiting](https://www.servicenow.com/docs/r/api-reference/rest-api-explorer/inbound-REST-API-rate-limiting.html) | Runtime throttling and error handling | Instance rules are operational inputs |
| [ServiceNow CMDB IRE](https://www.servicenow.com/docs/r/servicenow-platform/configuration-management-database-cmdb/c_CompsandProcessIDandReconcil.html) | Device/CI identification and reconciliation | Does not guarantee source inventory correctness |
| [ServiceNow identification rules](https://www.servicenow.com/docs/r/servicenow-platform/configuration-management-database-cmdb/c_IdentificationRules.html) | Identity matching needs governed rules | Instance-specific configuration |
| [Jira Service Management request API](https://developer.atlassian.com/cloud/jira/service-desk/rest/api-group-request/) | Requests, status, approvals, SLA, transitions | Cloud contract; permissions/configuration vary |
| [Jira Service Management scopes](https://developer.atlassian.com/cloud/jira/service-desk/scopes-for-oauth-2-3LO-and-forge-apps/) | Operation-specific scope baseline | Scopes do not replace Jira permissions, roles or app-access rules |
| [Atlassian webhooks](https://developer.atlassian.com/cloud/jira/platform/webhooks/) | Retry and deduplication behavior | Delivery is not authoritative truth |
| [Atlassian rate limiting](https://developer.atlassian.com/cloud/jira/platform/rate-limiting/) | Dynamic throttling and `Retry-After` handling | Policy evolves; no hard-coded capacity assumption |
| [Zendesk Tickets API](https://developer.zendesk.com/api-reference/ticketing/tickets/tickets/) | Ticket/request distinction, update/audit response and endpoint limits | Plan, roles, triggers, custom fields/status and visibility vary |
| [Zendesk safe ticket updates](https://developer.zendesk.com/documentation/ticketing/managing-tickets/creating-and-updating-tickets/) | `safe_update`, `updated_stamp` and collision handling | Timestamp guard still requires re-read and semantic conflict policy |
| [Zendesk Ticket Audits API](https://developer.zendesk.com/api-reference/ticketing/tickets/ticket_audits/) | Read-only update/event history and required global `read` scope | Scope may be broader than desired; audit is provider history, not full application truth |
| [Zendesk Ticket Comments API](https://developer.zendesk.com/api-reference/ticketing/tickets/ticket_comments/) | Public/private visibility, authorship and comment limits | Trigger ordering and account configuration can add events |
| [Zendesk Articles API](https://developer.zendesk.com/api-reference/help_center/help-center-api/articles/) | Article locale, audience, draft/translation and update fields | Search rank/update time is not applicability or approval |
| [Zendesk API rate limits](https://developer.zendesk.com/api-reference/introduction/rate-limits/) | Account and endpoint-specific headers/429 behavior | Plan/add-on and endpoint limits differ and evolve |
| [Microsoft Graph webhook delivery](https://learn.microsoft.com/en-us/graph/change-notifications-delivery-webhooks) | Ack deadline, retry/drop behavior, renewal | Subscription/resource details vary |
| [CloudEvents 1.0.2](https://github.com/cloudevents/spec/tree/ce%40v1.0.2) | Portable event envelope | Does not provide exactly-once semantics |

### Endpoint and remote support

| Source | Evidence used | Strength and limitation |
| --- | --- | --- |
| [Intune Remote Help planning](https://learn.microsoft.com/en-us/intune/remote-help/plan) | Tenant authentication, roles, consent, supported modes | Licensed/product-specific; configuration-dependent |
| [Intune Remote Help reporting/troubleshooting](https://learn.microsoft.com/en-us/intune/remote-help/troubleshoot) | Audit fields, retention, timeouts, report limitations | Not a full session-content record |
| [Intune device actions](https://learn.microsoft.com/en-us/intune/device-management/actions/) | Action catalog and asynchronous semantics | Availability varies by platform and state |
| [Intune diagnostics collection](https://learn.microsoft.com/en-us/intune/device-management/actions/collect-diagnostics) | Diagnostics as an active remote collection; current no-direct-Graph limitation; identifiable-data/storage/retention constraints | Admin-center/provider behavior varies by diagnostic type/platform; data minimization still belongs to deployer |
| [Intune Graph access and permissions](https://learn.microsoft.com/en-us/intune/developer/configure-graph-api-access) | Read/write/privileged permission distinction and scope changes | Provider permissions can be much broader than the admitted action |
| [Graph `syncDevice` v1.0](https://learn.microsoft.com/en-us/graph/api/intune-devices-manageddevice-syncdevice?view=graph-rest-1.0) | Active license and `PrivilegedOperations.All` requirement | `204`/acceptance does not establish device-side outcome |
| [Intune Remediations](https://learn.microsoft.com/en-us/intune/device-management/tools/deploy-remediations) | Script signature, output, overlap, and privacy considerations | Not authorization to run arbitrary scripts |
| [Microsoft Graph managed device](https://learn.microsoft.com/en-us/graph/api/resources/intune-devices-manageddevice?view=graph-rest-1.0) | Device identifiers, ownership, user, freshness fields | Fields can be absent/stale; cross-check required |
| [Microsoft Graph throttling](https://learn.microsoft.com/en-us/graph/throttling) | Backoff and retry signals | Limits vary by service and workload |
| [Apple MDM commands](https://developer.apple.com/documentation/devicemanagement/commands-and-queries) | Command surface | OS and management-mode constraints apply |
| [Apple MDM command processing](https://developer.apple.com/documentation/devicemanagement/sending-mdm-commands-to-a-device) | `CommandUUID`, acknowledgment, error, `NotNow` | Provider-specific status vocabulary |
| [Apple passcode management](https://developer.apple.com/documentation/devicemanagement/managing-passcodes) | Clear-passcode risk, especially lost devices | Supports explicit exclusion/high-risk handling |
| [Android Management API `issueCommand`](https://developers.google.com/android/management/reference/rest/v1/enterprises.devices/issueCommand) | Long-running operation, expiry, user action, mode restrictions | Product and enrollment-mode specific |
| [Jamf Pro API reference](https://developer.jamf.com/jamf-pro/reference/jamf-pro-api) | Instance OpenAPI, lifecycle distinctions | Preview/deprecated routes excluded from production |
| [Jamf Pro client credentials](https://developer.jamf.com/jamf-pro/docs/client-credentials) | API clients, roles and cumulative privileges | Effective instance privileges must be exported and tested |
| [Jamf Pro privileges/deprecations](https://developer.jamf.com/jamf-pro/docs/privileges-and-deprecations) | Per-route privileges and lifecycle evidence | Instance/version changes can alter the admitted surface |
| [ConnectWise ScreenConnect role permissions](https://docs.connectwise.com/ScreenConnect_Documentation/Get_started/Administration_page/Security_page/Define_user_roles_and_permissions/List_of_role-based_security_permissions) | Independent consent, command, tool, file, credential and access permissions | Live role/session-group configuration decides effective authority |
| [ConnectWise ScreenConnect session events](https://docs.connectwise.com/ScreenConnect_Documentation/Developers/Session_events) | Session/audit event vocabulary | Version and retention/export behavior need instance testing |
| [TeamViewer auditability/event log](https://www.teamviewer.com/en/global/support/knowledge-base/teamviewer-tensor/security/auditability-event-log/) | Session events, participant/authentication coverage and retention | License/configuration/authentication affect coverage; vendor logs are incomplete application audit |

### Identity-provider implementation surfaces

| Source | Evidence used | Strength and limitation |
| --- | --- | --- |
| [Microsoft Entra Temporary Access Pass](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-authentication-temporary-access-pass) | Recovery/bootstrap capability and lifecycle constraints | IAM-owned mechanism; secret never enters agent context |
| [Microsoft Entra account recovery](https://learn.microsoft.com/en-us/entra/identity/authentication/how-to-account-recovery-for-users) | External IDV, profile mode, claim/history/TAP dependencies and failure paths | Public Preview and tenant/provider dependent; feature/status must be rechecked |
| [Microsoft Entra release announcements](https://learn.microsoft.com/en-us/entra/fundamentals/whats-new) | Account Recovery Public Preview status | Monthly status page; recheck before every integration/release decision |
| [Microsoft Graph create TAP](https://learn.microsoft.com/en-us/graph/api/authentication-post-temporaryaccesspassmethods?view=graph-rest-1.0) | Secret-bearing response and TAP lifecycle fields | Tenant-wide sensitive permission belongs only to the recovery broker |
| [Okta credential permissions](https://developer.okta.com/docs/api/openapi/okta-management/guides/permissions) | Scoped management permissions | Tenant policy and licensing vary |
| [Okta user lifecycle/factor reset](https://developer.okta.com/docs/api/openapi/okta-management/management/tags/userlifecycle) | Factor reset is a consequential lifecycle action | Does not delegate recovery policy to service desk |
| [Google Workspace password reset](https://support.google.com/a/answer/33319) | Administrator reset behavior and operational caveats | Admin flow, not an assurance standard |

### Runtime, observability, evaluation, and operations

| Source | Evidence used | Strength and limitation |
| --- | --- | --- |
| [W3C Trace Context](https://www.w3.org/TR/trace-context/) | Cross-service trace correlation | Sensitive baggage still requires governance |
| [OpenTelemetry GenAI conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) | Candidate span/event vocabulary | Evolving status; sensitive content excluded |
| [Google SRE Workbook: SLOs](https://sre.google/workbook/implementing-slos/) | User-centered indicators and error budgets | Must be adapted to safety, not only availability |
| [Google SRE Workbook: incident response](https://sre.google/workbook/incident-response/) | Incident roles and response preparation | General operating guidance |
| [WorkArena publication](https://www.servicenow.com/research/publication/alexandre-drouin-work-icml2024.html) | ServiceNow task-environment evidence | Does not evaluate this blueprint's authority model |
| [AgentLab repository](https://github.com/ServiceNow/AgentLab) | Reproducible web-agent experimentation tooling | Research tooling, not production service-desk proof |
| [ITBench publication](https://research.ibm.com/publications/benchmarking-ai-agents-for-it-automation-tasks-with-itbench) | IT-automation task and telemetry evaluation | Domain coverage differs from endpoint support |

## Guide mapping

| Blueprint guide | Research questions answered |
| --- | --- |
| [Scope, workload fit, and authority](../../agents/it-service-desk-agent/01-scope-workload-fit-and-authority.md) | Category boundary, D0–D4 mapping, workload exclusions |
| [Reference architecture, runtime, and connectors](../../agents/it-service-desk-agent/02-reference-architecture-runtime-and-connectors.md) | System authority, tool contracts, connector semantics |
| [Authenticated intake, identity, device, and recovery](../../agents/it-service-desk-agent/03-authenticated-intake-identity-device-and-recovery.md) | Four-way binding, non-inference, recovery handoff |
| [Diagnostics, evidence, context, memory, and planning](../../agents/it-service-desk-agent/04-diagnostics-evidence-context-memory-and-planning.md) | Evidence provenance, compaction, bounded diagnostic planning |
| [Remote support, remediation, approvals, and effects](../../agents/it-service-desk-agent/05-remote-support-remediation-approvals-and-effects.md) | Consent, exact approval, independent verification, effect lifecycle |
| [Case state, reliability, reconciliation, and escalation](../../agents/it-service-desk-agent/06-case-state-reliability-reconciliation-and-escalation.md) | State ownership, idempotency, unknown outcomes, escalation |
| [Security, privacy, tenancy, and abuse resistance](../../agents/it-service-desk-agent/07-security-privacy-tenancy-and-abuse-resistance.md) | Social engineering, data boundaries, tenant isolation, threat controls |
| [Evaluation, observability, deployment, operations, and roadmap](../../agents/it-service-desk-agent/08-evaluation-observability-deployment-operations-and-roadmap.md) | Stage 0–6 gates, tests, SLOs, scaling, cost, releases, incidents, learning |

## Deliberately excluded claims

- No claim that an LLM can prove identity, detect deception reliably, or replace an approved recovery process.
- No claim that vendor API acceptance equals device execution or case resolution.
- No claim of exactly-once webhook delivery.
- No claim that remote-help metadata recreates what occurred on screen.
- No claim that a benchmark score establishes production authorization safety.
- No claim that increased ticket deflection is a safety or quality improvement.
- No claim that remote scripts are safe merely because they are delivered by MDM.
- No claim of compliance or certification from mapping controls or privacy frameworks.
- No claim that every vendor or platform supports the same action, consent, audit, or cancellation semantics.

## Remaining evidence gaps

1. **Cross-vendor cancellation semantics:** providers differ on whether queued, accepted, or executing effects can be revoked. Each production connector needs empirical contract tests.
2. **Provider audit completeness:** field availability, retention, and export latency require tenant-specific verification.
3. **User confirmation quality:** authenticated confirmation can verify restored service but may not establish every device postcondition. Each runbook needs a machine-verifiable predicate where possible.
4. **BYOD and shared devices:** ownership, privacy, and binding requirements need separate policy profiles before enablement.
5. **Accessibility and alternate channels:** consent and recovery flows require accessibility testing without weakening proof.
6. **Regional data handling:** diagnostic archive contents and residency differ by connector and tenant configuration.
7. **Benchmark coverage:** no public suite found in this pass jointly tests authenticated intake, wrong-device resistance, approval integrity, social-engineering pressure, asynchronous reconciliation, and privacy leakage.
8. **Endpoint state race behavior:** re-enrollment, rename, ownership transfer, and simultaneous admin action need production-like fault laboratories per platform.
9. **Live-tenant truth:** documentation cannot establish enabled license/plan, cloud/region, custom workflow, ACL/business rule, effective OAuth/RBAC scope, audit export/retention, rate rule, or preview availability. ServiceNow, Jira Service Management, Zendesk, Intune/Remote Help, Entra, Jamf and every RMM connector require expiring tenant qualification receipts.
10. **Identity-policy ownership:** NIST defines recovery requirements and vendors expose mechanisms, but the organization must select assurance, proofing provider, privileged-account treatment, notification/redress and human exception policy. Entra Account Recovery is Public Preview at the cut-off and cannot be the sole enterprise recovery route without an approved alternative.
11. **Endpoint-platform gaps:** Intune/Graph, Apple MDM, Android Management and Jamf differ by OS, enrollment/ownership/supervision mode, check-in state and command evidence. Current Intune documentation explicitly lacks a direct Graph path for diagnostic collection, while some seemingly small device actions require a broad privileged-operation scope.
12. **Remote-support effective authority:** consent, command, file, credential, elevation, unattended access, audit and termination controls are independent or configuration-dependent across products. The blueprint cannot claim an admitted ScreenConnect, TeamViewer, Microsoft Remote Help or other RMM path until excluded capabilities fail and disconnect/audit behavior is observed in the target tenant.

These are stage-gate inputs, not permission to fill gaps with model judgment.

## Refresh triggers

Re-run the relevant research and connector tests when any of the following occurs:

- NIST publishes a final SP 800-46 or SP 800-92 revision, Privacy Framework 1.1, or material SP 800-63 errata;
- a vendor changes identity-recovery, remote-help consent, unattended-access, or device-action semantics;
- an API version, OAuth scope, webhook contract, audit field, retention period, or rate-limit policy changes;
- a new endpoint OS/enrollment/ownership mode is enabled;
- a new remote tool, IdP, MDM, ITSM, or CMDB connector is added;
- a security advisory describes a new help-desk or recovery abuse technique;
- an incident exposes a wrong-subject, wrong-device, cross-tenant, duplicate-effect, secret-leak, or incomplete-audit path;
- a new benchmark is proposed as release evidence;
- telemetry schema changes could export prompt, ticket, diagnostic, identity, or remote-session content.

## Research conclusion

The evidence supports a bounded service-desk copilot that can become progressively more capable only while deterministic authority remains outside the model. The production unit is not “a resolved answer.” It is a verified case transition backed by authenticated bindings, provenance-bearing evidence, an authorized and independently verified effect if needed, reconciled provider state, and an auditable closure reason.

The blueprint therefore advances through stages 0–6 by increasing integration and carefully bounded action, not by granting the model broader autonomy. Any stage that cannot prove the right tenant, requester, principal, device, consent, effect, and postcondition must remain read-only or proposal-only.

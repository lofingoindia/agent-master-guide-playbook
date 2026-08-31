# Authenticated Intake, Identity, Device, and Recovery

> **Section index:** [IT Service Desk and Endpoint Support Agent Blueprint](README.md)  
> **Evidence:** [Research packet](../../research/packets/it-service-desk-agent-blueprint.md)

Service-desk safety depends on knowing which authenticated person, account, tenant, and physical or virtual endpoint the case concerns. A plausible match is not enough. Identity and device binding must remain explicit, provenance-bearing, revocable, and allowed to fail.

## Four identities, never one string

Record these separately:

```text
requester principal = (identity issuer, immutable subject)
affected account    = (identity provider tenant, immutable account ID)
device              = (management provider tenant, immutable device/enrollment ID)
operator             = (workforce issuer, immutable subject, support role)
```

Also record the workload identity and connector identity used for every read/effect. Display names, email addresses, phone numbers, hostnames, asset tags, and serial numbers are attributes used to resolve or display—not the final authorization key.

OpenID Connect defines subject identity in the context of an issuer. NIST zero-trust guidance rejects implicit trust based only on network location or asset ownership and separates subject and device authentication/authorization. The application must preserve those distinctions.

## Entity identity and version semantics

Never overload `user_id`, `ticket_id`, or `version`. The canonical type records the provider/tenant namespace, immutable identifier, observed source version, and the purpose for which the observation is fresh enough.

| Entity | Canonical identity | Version/freshness semantics | Unsafe substitution |
|---|---|---|---|
| Requester | Channel tenant + authenticated issuer/subject, or an explicit unverified claimant ID | Revalidate channel/session assurance after waits and before protected disclosure | Affected employee, ticket submitter, email address, or caller ID |
| Principal/account | IdP tenant + immutable directory/account ID; retain issuer/subject when applicable | Directory object version plus authentication/recovery state observed at a recorded time | Requester, display name, UPN/email, or HR person ID |
| Employee/person | HR tenant + immutable person/employment record | Effective-dated employment/assignment relation; termination and leave can revoke access without deleting history | Login account or proof that the claimant controls it |
| Device | Management tenant + provider device/enrollment ID | Enrollment generation, inventory/version token, last successful sync, management and ownership state | Hostname, serial, Entra device ID, or asset tag alone |
| Asset/CI | Asset/CMDB namespace + immutable asset/CI ID | Source precedence, lifecycle state, assignment and reconciliation version; may outlive several enrollments | Current managed-device target or proof of current user possession |
| Ticket | ITSM instance + immutable work-record ID | Provider concurrency token, audit ID, or last-observed fields/time; re-read before and after writes | Local case projection, incident semantics, or authorization |
| Incident | ITSM instance + immutable incident ID and mapped lifecycle | Provider/custom state model and SLA version; restoration semantics remain provider-owned | Every ticket or service request |
| Request/catalog item | ITSM instance + request/customer-request ID plus request-type/version | Request fields, approvals and visible status as of provider version; a provider may expose this as a customer view over another work record | Incident, change, or authenticated requester identity |
| Change | ITSM instance + immutable change ID and approved implementation version/window | Approval, risk, schedule, implementation and rollback state must be re-read; linkage is evidence, not permission | A ticket note, model proposal, or endpoint effect approval |
| Knowledge/article | Knowledge base + logical knowledge ID + immutable article/translation version and locale | Publication state, ACL/audience, owner, applicability, valid/review dates and withdrawal tombstone | Search rank, cached excerpt, draft, or a runbook authorization |
| Session | Session kind + provider/tenant + opaque session ID | Authentication time/expiry/revocation for channel sessions; participant/mode/start/end/disconnect for remote sessions | Principal identity, timeless consent, or proof of actions on screen |
| Effect | Local semantic effect ID over canonical target, capability, versioned intent, parameters and postcondition | Intent version is immutable; dispatch attempts and provider operation IDs are child correlations; terminal state is verified/failed/accountable-unknown | HTTP request ID, provider `2xx`, ticket transition, or model claim |

Adapters may use different words. For example, Zendesk distinguishes the agent-facing ticket from the end-user-facing request, Jira Service Management exposes customer requests backed by Jira issues, and ServiceNow instances can customize incident/request/change workflows. Normalize meaning only after the live tenant mapping is reviewed; never infer equivalence from a display label.

## Channel assurance

| Intake channel | Initial state | Protected reads | Remote/identity effects |
|---|---|---|---|
| Enterprise portal/chat with current SSO | `channel_authenticated` | Only after tenant/purpose authorization | Still requires action-specific approval and verification |
| ITSM ticket created inside authenticated employee portal | `channel_authenticated` if trusted claims are propagated and validated | Case-scoped | Same D3 gate |
| Email | `claimant_unverified` unless strong authenticated channel binding exists | Public/general guidance only | Prohibited |
| Phone | `claimant_unverified`; human operator may initiate approved verification | No protected disclosure before verification | Prohibited until recovery/support policy completes |
| Walk-up/in-person | Organization-defined attended verification | Per local policy | Human operator records method and assurance |
| Monitoring/system-generated ticket | `system_originated`; no human principal inferred | Service/device evidence under system purpose | No user-authorized remote action until owner is resolved |

An authenticated channel proves control of a session at a point in time. It does not prove that the user is entitled to another account, that a device name identifies the intended device, or that an old approval is current.

## Intake contract

```yaml
intake:
  request_id: req_01K...
  tenant_id: tenant_7f2
  received_at: 2026-08-31T12:00:00Z
  channel:
    kind: enterprise_portal
    assurance: authenticated
    session_id_hash: sha256:...
    authenticated_at: 2026-08-31T11:58:41Z
  requester_claim:
    issuer: https://id.example.com/tenant/7f2
    subject: 8c5c0d7e-...
  request:
    summary: "VPN client reports error 809"
    affected_service_hint: corp-vpn
    device_hint: "my laptop"
    attachments: [artifact_quarantine_81]
  trust_labels:
    user_text: untrusted_data
    attachment: quarantined
  dedupe_key: sha256:tenant+principal+service+time-bucket+normalized-symptom
```

The normalized symptom aids routing/deduplication but does not become a diagnosis. Preserve the original message as a controlled artifact when retention policy permits.

## Principal binding

### Accepted sources

- validated identity-provider session claims;
- current directory account queried by immutable issuer/subject or provider-native ID;
- organization HR/person record where policy designates it authoritative for employment relationship;
- approved attended identity-proofing/recovery service output; and
- an authenticated human operator's recorded exception under a separately governed policy.

### Not accepted as proof

- email display/from address, caller ID, SMS number, badge photo sent in chat, voice match, face match by the model, IP/geolocation, department knowledge, manager name, date of birth, employee number stated by the claimant, recent ticket content, or writing style;
- the fact that a claimant can access a possibly compromised mailbox or endpoint; or
- agreement between two model calls using the same untrusted facts.

The 2025 joint Scattered Spider advisory describes attackers gathering PII and learning reset processes before persuading help desks. NIST SP 800-63B-4 prohibits knowledge-based authentication/security questions for authenticator handling and defines specific recovery methods. Do not create a local “five personal questions” substitute.

## Device binding

Device selection is a join across current authoritative records, not a fuzzy hostname lookup.

### Minimum evidence

1. tenant and management provider;
2. immutable device or enrollment ID;
3. inventory version and observed time;
4. management/enrollment state;
5. ownership type and current user assignment, when the platform supports it;
6. corroborating CMDB/asset relationship and its source/freshness;
7. requester confirmation rendered with safe distinguishing attributes; and
8. ambiguity/conflict result.

Microsoft Intune's managed-device resource, for example, distinguishes an immutable managed-device ID, associated user ID, Entra device ID, serial number, ownership, registration state, and last successful sync. ServiceNow CMDB's Identification and Reconciliation Engine separately emphasizes identification rules and authoritative data-source precedence. These product concepts support—not replace—the application's binding contract.

### Binding schema

```json
{
  "binding_id": "bind_01K...",
  "case_id": "case_01K...",
  "tenant_id": "tenant_7f2",
  "principal": {
    "issuer": "https://id.example.com/tenant/7f2",
    "subject": "8c5c0d7e-...",
    "directory_account_id": "acct_91a...",
    "verified_at": "2026-08-31T12:03:00Z",
    "verification_method": "current_sso_plus_directory_lookup"
  },
  "device": {
    "provider": "intune",
    "provider_tenant_id": "tenant_7f2",
    "managed_device_id": "md_57a...",
    "directory_device_id": "dev_14c...",
    "display": {"asset_tag": "LTP-4821", "model": "approved-display-value"},
    "inventory_observed_at": "2026-08-31T11:55:19Z",
    "last_successful_sync_at": "2026-08-31T11:50:04Z",
    "management_state": "managed",
    "ownership": "company"
  },
  "relationship_evidence": [
    {
      "source": "endpoint_management",
      "relationship": "associated_user",
      "source_record_id": "md_57a...",
      "observed_at": "2026-08-31T11:55:19Z"
    },
    {
      "source": "asset_system",
      "relationship": "assigned_to",
      "source_record_id": "asset_3081",
      "observed_at": "2026-08-31T10:00:00Z"
    }
  ],
  "user_confirmation": {
    "confirmed": true,
    "at": "2026-08-31T12:04:08Z",
    "channel_session_ref": "session_hash:..."
  },
  "status": "bound",
  "expires_at": "2026-08-31T12:34:08Z"
}
```

The expiry is short for effect authorization. The case may retain the historical binding, but a D3 commit must refresh current assignment, management state, and policy.

## Ambiguity protocol

| Condition | Result | Allowed next step |
|---|---|---|
| No current device | `unresolved` | Ask user to open approved device picker or escalate |
| Multiple devices | `ambiguous` | Show safe distinguishing fields; user selects; re-query exact ID |
| MDM and asset owner conflict | `conflicting` | No remote action; asset/endpoint owner resolves |
| Inventory stale beyond action policy | `stale` | Refresh; if offline, advisory/user-run steps only |
| Personal/BYOD device | `restricted` | Apply BYOD-specific reads/actions and privacy policy; default no remote control |
| Device reassigned while case waits | `revoked` | Invalidate approvals and proposals; rebind |
| Device record duplicated or serial reused | `conflicting` | Escalate CMDB/MDM data-quality issue; never choose oldest/closest automatically |

## Identity-sensitive requests

Treat these as recovery or security workflows rather than normal troubleshooting:

- password or passkey reset when no valid authenticator remains;
- MFA/authenticator removal, replacement, or transfer;
- Temporary Access Pass or similar bootstrap credential;
- recovery phone/email/contact change;
- session/token revocation after suspected compromise;
- privileged/service/admin account recovery;
- account ownership dispute; and
- repeated, unusual, or high-pressure recovery attempts.

Changing a forgotten password when the subscriber can still authenticate with another bound authenticator may be authenticator binding rather than full account recovery under NIST terminology. The IAM-owned service decides the applicable policy; the agent does not.

## Account-recovery handoff

```mermaid
sequenceDiagram
    participant C as Claimant
    participant S as Service-desk agent/case
    participant R as Recovery service
    participant O as Authorized recovery operator
    participant I as Identity provider
    participant N as Independent notification channel

    C->>S: Cannot authenticate / requests recovery
    S->>S: Mark claimant unverified; disclose no protected details
    S->>R: Create handoff with account hint, case, reason, risk flags
    R->>C: Run configured recovery method outside model/ticket
    R->>O: Evidence-first decision if human review required
    O->>R: Approve/deny exact recovery operation before expiry
    R->>I: Execute through narrow recovery capability
    I-->>R: Provider receipt and resulting authenticator/account state
    R->>N: Send subscriber recovery notification
    R-->>S: Opaque outcome, receipt, notification status, next step
    S->>C: Continue sign-in verification or safe redress
```

### Handoff schema

```yaml
recovery_handoff:
  handoff_id: rh_01K...
  case_id: case_01K...
  tenant_id: tenant_7f2
  affected_account:
    provider: entra
    immutable_account_id: acct_91a...
    resolution_source: authenticated_directory_lookup
  claimant_status: unverified_for_recovery
  reason: lost_all_authenticators
  account_risk_class: workforce_standard
  security_flags:
    suspected_compromise: false
    repeated_attempts: false
    privileged_account: false
  requested_outcome: regain_access_by_org_recovery_policy
  prohibited_data:
    - passwords
    - recovery_codes
    - temporary_access_pass
    - biometric_samples
    - identity_document_images
  service_callback:
    result_fields: [decision, opaque_receipt_id, notification_sent, completed_at, redress_route]
  expires_at: 2026-08-31T13:00:00Z
```

The ticket stores no recovery secret or raw proofing artifact. If the recovery provider needs identity documents/biometrics, its separately approved privacy, retention, fraud, redress, and access controls apply.

## Recovery requirements derived from NIST SP 800-63B-4

NIST's current final Revision 4 recognizes saved recovery codes, issued recovery codes, recovery contacts, and repeated identity proofing, with assurance-dependent combinations. It also requires account-recovery notification to the subscriber or designee. Adopt these principles without claiming NIST conformance unless the entire identity service has been assessed:

- use the identity provider/CSP's configured recovery policy, not agent-generated questions;
- preserve the original identity-proofing/account assurance level;
- require the appropriate independent recovery evidence/combination;
- notify the subscriber through a registered independent channel;
- rate-limit and monitor attempts;
- provide accessible options and redress for failures;
- invalidate/suspend compromised authenticators promptly under IAM policy; and
- keep the agent from seeing, storing, summarizing, or transmitting recovery secrets.

### Vendor-specific examples are mechanisms, not policy

- Microsoft Entra ID Account Recovery is **Public Preview at this guide's 2026-08-31 cut-off**. Current documentation requires a configured external identity-verification provider, scoped recovery profile, production rather than evaluation mode, compatible user/profile claims, prior authentication history in some cases, and TAP eligibility. It is not a generally available promise for every tenant, region, identity, or guest account.
- Microsoft Entra Temporary Access Pass is a separate bootstrap mechanism. It can be time-limited and one-time or multi-use depending on tenant policy. The Graph application permission for creating TAPs is tenant-wide and admin-consented, the response contains the secret, and TAP expiry does not necessarily terminate already established sessions without compatible Conditional Access session controls. A narrow IAM broker—not the agent or general connector—chooses eligibility, creates/delivers the secret through a protected channel, and returns only an opaque receipt.
- Okta exposes narrower credential-lifecycle permissions, factor-reset operations, and system events. Use credential-specific scope and a separate executor rather than broad user administration.
- Google Workspace password reset has distinct privileges and session/cookie consequences. A password reset is not proof that every previous session or downstream token is safe; follow provider and incident policy.

Do not generalize one provider's behavior to another.

## Worked flow: password problem versus account recovery

1. A user says “reset my password.” Intake preserves that phrase as an assertion and resolves the affected account separately from the requester.
2. If the user still has an approved bound authenticator, the IAM service may route them to ordinary password change/SSPR or authenticator binding. The agent provides the current cited path and tracks only an opaque outcome; it never asks for the old/new password or OTP.
3. If all authenticators are lost, the case marks the claimant `unverified_for_recovery` even when an older authenticated ticket exists. The agent creates the typed handoff above; the recovery service chooses proofing/recovery methods and communicates through its protected user interface.
4. A standard workforce account can proceed only under the configured assurance policy. A privileged/service account, suspicious channel change, repeated attempt, inconsistent account/person record, lost/stolen device or risky sign-in diverts to the named IAM/security route with no conversational override.
5. If the IAM service creates a TAP or other bootstrap credential, only its broker and protected delivery channel see the secret. The case receives an opaque receipt, expiry/category, notification status and redress route—not the code.
6. Provider acceptance is not restoration. Verify the intended authenticator state, required subscriber notification and a fresh sign-in appropriate to policy. If the callback is lost after a possible commit, set `recovery_outcome_unknown`, block a second issuance/reset, and have IAM reconcile provider audit/state.
7. Closure distinguishes `self_service_completed`, `recovery_completed`, `recovery_denied_redress`, `security_handoff`, and `accountable_unknown`. A password change alone does not prove old sessions/tokens are revoked or a compromise is contained.

## Notifications and redress

After an identity-sensitive operation:

- notify the subscriber through a pre-registered channel not supplied during the recovery attempt;
- state what changed, when, tenant/account, and how to report fraud without exposing secrets;
- alert security on suspicious or privileged recovery;
- retain decision and provider receipts under policy;
- provide an accessible recovery failure/redress path; and
- prevent the model from suppressing, rewriting, or delaying the notification.

## Failure matrix

| Failure | Detection | Safe state | Recovery/owner |
|---|---|---|---|
| Authenticated session revoked while queued | Token/session introspection or provider denial | `identity_revalidation_required` | Reauthenticate; invalidate approvals |
| Directory has duplicate/mismatched account | Resolver returns conflict | `identity_conflict` | IAM/HR owner; no disclosure/effect |
| Device association changed | Fresh inventory version differs | `binding_revoked` | Rebind and rebuild proposal |
| Recovery service unavailable | Health/timeout | `waiting_recovery_service` | Manual IAM runbook; agent provides no workaround |
| Recovery proof fails | Recovery-service decision | `recovery_denied_or_redress` | Redress or attended proofing; no repeated model questioning |
| Provider accepts reset but callback is lost | Dispatch exists, no result | `recovery_outcome_unknown` | IAM queries provider audit/state; never repeat blindly |
| Temporary credential appears in ticket/model/log | DLP/canary detector | Security incident | Revoke secret, contain connector, investigate exposure |
| Subscriber notification fails | Notification receipt missing | `recovery_notification_pending` | Retry notification through idempotent notifier; operator owns closure |
| Risky sign-in or social-engineering pattern | Identity/security signal | `security_quarantine` | Security investigation handoff |

## Acceptance tests

- [ ] Same display name in two tenants never cross-resolves.
- [ ] Email, phone, voice, department, manager, asset tag, or user facts alone never bind identity.
- [ ] Multiple active devices require explicit selection and exact provider re-query.
- [ ] Reassignment during approval invalidates the proposal.
- [ ] Stale/offline inventory cannot authorize a D3 action.
- [ ] Account recovery uses the independent service and no secret enters the case/model.
- [ ] KBA/security questions are absent from recovery logic.
- [ ] Privileged/repeated/suspicious recovery routes to security/IAM review.
- [ ] Lost-device reports cannot trigger passcode clearing or wipe in this blueprint.
- [ ] Callback loss produces `unknown`, reconciliation, and one eventual recorded outcome.
- [ ] Subscriber notification and redress are observable, not assumed.

## Sources and related guidance

- [NIST SP 800-63 Digital Identity Guidelines Revision 4](https://pages.nist.gov/800-63-4/)
- [NIST SP 800-63A-4 identity proofing](https://pages.nist.gov/800-63-4/sp800-63a.html)
- [NIST SP 800-63B-4 authentication and recovery](https://pages.nist.gov/800-63-4/sp800-63b.html)
- [NIST SP 800-207](https://csrc.nist.gov/pubs/sp/800/207/final)
- [Microsoft Intune managed-device resource](https://learn.microsoft.com/en-us/graph/api/resources/intune-devices-manageddevice?view=graph-rest-1.0)
- [ServiceNow CMDB identification and reconciliation](https://www.servicenow.com/docs/r/servicenow-platform/configuration-management-database-cmdb/c_CompsandProcessIDandReconcil.html)
- [Microsoft Entra Temporary Access Pass](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-authentication-temporary-access-pass)
- [Microsoft Entra account recovery user flow](https://learn.microsoft.com/en-us/entra/identity/authentication/how-to-account-recovery-for-users)
- [Microsoft Entra release announcement: Account Recovery Public Preview](https://learn.microsoft.com/en-us/entra/fundamentals/whats-new)
- [Microsoft Graph TAP creation](https://learn.microsoft.com/en-us/graph/api/authentication-post-temporaryaccesspassmethods?view=graph-rest-1.0)
- [Okta credential permissions](https://developer.okta.com/docs/api/openapi/okta-management/guides/permissions)
- [Joint government Scattered Spider advisory](https://www.cyber.gov.au/about-us/view-all-content/alerts-and-advisories/scattered-spider)
- [Permissions, sandboxing, and secrets](../../security/permissions-sandboxing-and-secrets.md)

# Implementation Roadmap, Acceptance Tests, and Alternatives

> **Last researched:** 2026-08-31  
> **Purpose:** Turn the blueprint into reviewable phases, prove the critical invariants, and decide when a custom loop, framework, hybrid, browser agent, RPA system, or no automation is appropriate.

Build one narrow, low-risk workflow through the full safety and evidence path before making the agent general. Generality multiplies app states, permissions, data exposure, recovery branches, and evaluation cost.

## Reference build target

The first production candidate should support:

- one OS and one versioned application workflow;
- synthetic or low-sensitivity task data;
- one dedicated disposable environment per run;
- hybrid semantic plus screenshot observation;
- a small typed action set with no general shell/code execution;
- read and reversible preparation, with one exact-effect confirmation gate if needed;
- deterministic terminal verification;
- bounded turns/actions/time/cost and stuck detection;
- audit reconstruction, cleanup, kill switch, and incident quarantine;
- one primary model route and no automatic untested fallback.

Do not begin with “operate any app on my laptop.”

## Stage 0–6 roadmap

### Stage 0 — task and risk contract

Deliver:

- supported user story, non-goals, and forbidden actions;
- comparison against no automation, API/workflow, RPA, browser-semantic, and user-assist baselines, with a recorded go/no-go decision;
- step-surface map: API, browser semantic, app API, accessibility, pixels;
- canonical domain identity/version and typed state/event/plan/tool/effect contracts;
- threat model and data classification;
- authoritative initial/target state and validator;
- environment/app/locale/resolution baseline;
- latency, cost, takeover, and severe-failure gates;
- representative clean, edge, and adversarial task set.

Exit criteria:

- product, security, privacy, operations, and application owner agree on authority and evidence;
- every consequential effect has a deterministic/takeover plan;
- GUI control is justified for each GUI step.

### Stage 1 — deterministic environment and executor

Deliver:

- immutable base image and per-run exclusive lease;
- capture, active-window, semantic-state, and geometry RPCs;
- minimum input actions with target/focus checks;
- clipboard/files/network/devices denied by default;
- environment reset/destruction and residue canaries;
- action receipts and local kill switch.
- adapter qualification records for the exact OS/app/session/automation surface, including unsupported cases and rollback.

Exit criteria:

- no model is involved;
- deterministic scripts pass focus, DPI, modal, crash, cancel, and cleanup tests;
- executor cannot escape its allowlisted app/window/action surface.

### Stage 2 — observation and verification

Deliver:

- immutable observation envelope and coordinate transform;
- screenshot redaction before transmission;
- accessibility/DOM/app state normalization with provenance;
- target re-resolution and stale-observation rejection;
- capture/source sequences, locale/input/accessibility/display generations, cross-source skew, and stale-frame detection;
- post-action expected-change and terminal validators;
- offline dataset of observations and targets.

Exit criteria:

- grounding/geometry/redaction thresholds pass across supported themes/locales/resolutions;
- semantic/visual disagreement fails closed for risky targets;
- terminal success cannot be produced from model prose.

### Stage 3 — model loop in read-only/test mode

Deliver:

- provider/model adapter mapped to internal actions;
- context compiler and release manifest;
- run state machine, budgets, stuck detector, and recovery levels 0–4;
- all seven memory lifetimes and a restart-safe compaction receipt with ledger/source high-watermarks, versions, approvals, pending/`UNKNOWN` effects, next safe action, and invariants hash;
- event/artifact correlation and offline audit viewer;
- evaluation harness with synthetic accounts and effects disabled.

Exit criteria:

- repeated task/grounding/latency/cost gates pass;
- no action executes after stale state, cancellation, lease loss, or policy deny;
- prompt-injection suite cannot create a privileged effect because none exists.

### Stage 4 — policy, secrets, and reversible writes

Deliver:

- canonical policy engine and short-lived action capabilities;
- opaque credential/data/file/clipboard brokers;
- reversible write previews and rollback/reconciliation;
- takeover protocol and operator UI;
- egress proxy and download quarantine.

Exit criteria:

- secret canaries never appear in provider payloads, screenshots, clipboard, or logs;
- cross-app/data movement is policy-bound and auditable;
- rollback and unknown-outcome behavior pass failure injection.

### Stage 5 — exact-effect commit

Only if the product truly requires it.

Deliver:

- prepare/digest/approve/revalidate/commit protocol;
- one-use approval and effect IDs;
- authoritative downstream reconciliation;
- approval expiry/deny/edit semantics;
- zero-tolerance catastrophic regression gates.

Exit criteria:

- duplicate, wrong-recipient/account/tenant, stale-approval, post-cancel, and blind-retry tests all pass;
- the user can understand the exact effect preview;
- security/product owner accepts residual risk.

### Stage 6 — canary and operations

Deliver:

- capacity queues/pools and SLO dashboards;
- scoped kill switches and incident runbook;
- read-only/prepare-only canary mode;
- release comparison and rollback;
- automated artifact retention/deletion and VM destruction evidence.
- recovery-load/DR exercises and operator runbooks for stale display, disconnect, crash, unknown effect, injection, residue, rollback, and dependency outage.

Exit criteria:

- canary meets compliant-success, severe-failure, latency, cost, cleanup, and takeover gates;
- on-call can diagnose and stop a run from the audit UI;
- rollback and image drain have been exercised.

## Controlled evolution after Stage 6

Add one app, workflow, locale, resolution, action, data class, or risk tier at a time. Each expansion updates the threat model, environment image, policy, tests, and release gate. Do not infer coverage from a similar application.

Treat every expansion and behavioral change as governed continuous evolution. Convert reviewed UI drift, stale-grounding failures, unknown effects, unsafe proposals, takeovers, incidents, and user corrections into candidate evaluation cases; replay current and proposed environment/model/tool/context/policy bundles; shadow and canary by application and effect class; and keep a pinned full-bundle rollback. No production screenshot, page text, or model conclusion may automatically become a skill, policy, or long-term memory.

For every proposed change, classify whether it affects observation, identity/version semantics, action/effect behavior, authority, data boundary, recovery, evidence, or supported environment. Re-run all affected slices and every safety invariant, publish a compatibility/migration note, canary one dimension at a time, and retain the prior complete release—not an arbitrary mix of old policy with new model/image/adapter.

## Reference configuration

Illustrative application-owned configuration:

```yaml
release: cua-support-draft/0.9.0
environment:
  os_image: windows-11-24h2-cua-2026-08-20
  apps:
    ticketing: 8.14.2
    mail: 6.9.1
  display: {width: 1600, height: 900, scale: 1.0, monitors: 1}
  locale: en-US
  clipboard_bridge: disabled
  host_folders: none
  devices: [virtual_keyboard, virtual_pointer, virtual_display]
  network_policy: support-draft-egress/v3

agent:
  model_route: primary-cua/v5
  tool_contract: cua-actions/v2
  context_compiler: cua-context/v4
  max_model_turns: 24
  max_actions: 40
  max_elapsed_seconds: 600
  max_cost_usd: 2.50
  max_repeated_equivalent_actions: 2
  max_replans: 2

policy:
  version: cua-policy/v7
  allowed_apps: [ticketing, mail]
  allowed_origins: ["https://tickets.example.internal"]
  denied_actions: [install, execute_download, change_permission, delete]
  external_communication:
    mode: prepare_then_exact_confirmation
  credential_mode: brokered_fill_only

evidence:
  event_schema: cua-events/v1
  full_screenshots: incident_only
  action_crops_retention_days: 7
  effect_receipts_retention_days: 365
  redact_before_model: true
```

Configuration is not enforcement by itself. The policy, environment manager, executor, and storage services must reject unsupported or weaker settings.

## Reference control loop

```python
async def run_step(run_id: str) -> None:
    run = await runs.load_for_update(run_id)
    assert run.status == "ready"
    assert run.deadline > clock.now()

    observation = await observer.capture(run.environment_lease)
    context = context_compiler.build(run, observation)
    proposal = await model_adapter.propose(context)

    validated = action_contract.validate(proposal)
    decision = await policy.evaluate(run, observation, validated)
    await events.append(decision.event)

    if decision.kind == "deny":
        await runs.record_rejection(run, decision)
        return
    if decision.kind == "require_confirmation":
        await runs.wait_for_exact_approval(run, decision.effect_digest)
        return

    capability = await capabilities.issue_single_use(decision)
    receipt = await executor.execute(
        lease=run.environment_lease,
        capability=capability,
        observation_id=observation.id,
        action=validated,
    )
    await events.append(receipt.event)

    if receipt.outcome == "unknown":
        await runs.mark_effect_unknown(run, receipt)
        return

    post = await observer.capture(run.environment_lease)
    verdict = await verifier.check(run, validated, receipt, post)
    await runs.apply_verdict(run, verdict)
```

Production code also needs lease generations, cancellation fencing, first-failure batch semantics, redaction failure handling, timeouts, retries for safe reads, and transactional event/state updates. The important point is the order: observe → propose → validate → authorize → execute → record → verify.

## Acceptance test catalog

### Environment and isolation

- clean boot has exact OS/app/browser/font/locale/display versions;
- prior-run file, cookie, clipboard, process, and account canaries are absent;
- guest cannot reach host filesystem, default browser profile, metadata, localhost services, private networks, management ports, or other tenants;
- host↔guest clipboard, drag/drop, shared folders, devices, and printer are denied unless individually required;
- only the controller holding the lease can invoke executor actions;
- destroy/revoke succeeds after success, failure, cancellation, crash, and approval timeout;
- failed cleanup quarantines rather than pools the environment.

### Capture, grounding, and focus

- coordinate calibration passes at supported resolutions, DPI, rotation, and display topology;
- action using an old observation after window move/resize is rejected;
- focus theft one millisecond before dispatch results in zero input;
- minimized, occluded, locked, secure, and disconnected desktop states fail closed;
- small/duplicate/disabled/animated targets trigger zoom/clarification, not risky guess;
- semantic and visual target mismatch stops high-impact action;
- redaction canaries in text, image, terminal, notification, QR code, and accessibility properties do not leave the boundary.

### Policy and effects

- on-screen instructions cannot expand apps, origins, recipients, data, actions, or budget;
- provider safety allow does not override application deny;
- provider confirmation request is handled without executing first;
- approval is invalid after content, destination, account, policy, environment, principal, or source version change;
- denial is terminal for that effect and is not repeatedly re-requested;
- batch stops after first failure and before every confirmation/verification boundary;
- timeout after commit produces `unknown`, freezes related actions, and reconciles;
- repeated callback/replay cannot execute a one-use capability or approval twice;
- cancellation fences late model/executor responses and releases input state.

### Context and recovery

- context pruning retains task, current milestone, unresolved effect, approval, and evidence references;
- repeated click, no-progress, scroll loop, app-switch loop, and completion-claim loop trigger bounded recovery;
- app crash before effect restarts safely; app crash after possible effect reconciles first;
- controller crash increments lease generation and ignores prior late results;
- fresh environment restoration imports only verified state, not compromised workspace/transcript;
- user takeover quiesces the agent and invalidates all observations/approvals on handback.

### Evaluation and operations

- audit replay is read-only and reconstructs sampled runs end to end;
- offline replay cannot contact executor or redeem capabilities;
- eval re-execution uses synthetic effects and new run/effect identities;
- budget limits work even if telemetry/export is unavailable;
- model/provider fallback does not occur unless route is explicitly evaluated;
- queue overload backpressures without weakening isolation or policy;
- kill switch works during model call, action, approval, unknown effect, and cleanup;
- artifact deletion and VM destruction have observable completion and alerting.

## Implementation exercises and exit evidence

Use exercises to build operator and engineering competence; do not count a walkthrough as passing evidence.

### Realistic workflow exercises

| Workflow | Safe implementation shape | Required perturbations |
|---|---|---|
| Support reply across ticketing and native mail | API reads canonical ticket/requester; desktop prepares draft; deterministic readback; exact-effect approval; domain sent-message reconciliation | Wrong account, recipient autocomplete drift, accessibility/pixel disagreement, crash after Send, stale approval, denial, cancel during commit |
| Download a report, inspect it in a native app, export an approved derivative | Browser-semantic download interception to quarantine; file broker imports read-only; native app uses semantic actions; output broker scans/DLP-checks and exports declared hash | Malicious filename/archive, macro/executable disguise, clipboard lure, symlink/junction, unsupported locale/font, app crash during save, artifact-store outage |
| Configure a legacy application but hand over for authentication/consent | Model navigates reversible screens; policy blocks credentials and OS/browser safety prompts; agent quiesces; human authenticates/consents; handback invalidates state | UAC/secure desktop, CAPTCHA, privacy picker, local user moves pointer, RDP reconnect/resolution change, handback without fresh observation |
| Read-only investigation across browser and remote desktop | Separate browser/desktop adapters and origins; fused evidence retains source; no mutation/export tools | Visual and accessibility injection, stale remote frame, VDI popup, wrong tenant, OCR confident false label, remote disconnect/reconnect |

### Stage exit-evidence package

| Stage | Evidence required to advance |
|---|---|
| 0 | Signed task/authority/non-goal contract; baseline comparison; threat/data-flow model; canonical identities/versions; initial eval and severe-failure definitions |
| 1 | Image/adapter manifest and qualification; deterministic executor conformance; focus/permission/session tests; isolation/residue/destruction proof; kill receipt |
| 2 | Observation schema and fixtures; geometry/redaction/stale-frame/source-skew results across declared slices; authoritative verifier results and known blind spots |
| 3 | Versioned context/compiler manifest; seven-lifetime policy; repeated compaction and crash-resume proof; read-only model eval with budgets/stuck/recovery; reconstructable sample audit |
| 4 | Policy decision traces; single-use broker receipts; secret/DLP canary results; reversible rollback/reconciliation; takeover and injection failure tests |
| 5 | Exact preview/readback/digest/approval/commit receipts; duplicate/stale/deny/cancel/crash-after-commit tests; accountable residual-risk acceptance |
| 6 | Canary report by material slice; metrics/traces/logs/audit/SLO review; capacity plus recovery-load results; DR/rollback/kill/incident exercises; deletion and VM-destruction evidence |

Store the package with the release manifest and named owners. An unchecked Markdown box, demo video, benchmark score, or one successful run is not exit evidence.

## Failure-injection campaign

Run before every release that changes model, tool schema, executor, base image, policy, perception, or context.

| Domain | Inject | Pass condition |
|---|---|---|
| Perception | Wrong monitor, DPI change, blank frame, corrupt PNG, stale tree | No mis-targeted input; typed error and bounded recovery |
| UI dynamics | Popup, animation, layout shift, modal, slow save, app not responding | No blind repetition; expected-change verifier governs progress |
| Focus/input | User input, focus theft, held modifier, input API partial failure | One owner; keys/buttons released; receipt accurate |
| Provider | Timeout, rate limit, malformed action, duplicate call ID, late response | Budgets/fencing; safe retries only; no duplicate execution |
| Policy | Engine timeout, version mismatch, conflicting provider signal | Fail closed for actions; decision provenance retained |
| Approval | Delayed, denied, edited, replayed, expired | No commit without fresh matching one-use approval |
| Effects | Response lost before/after commit, eventual consistency, duplicate click | `unknown` then reconcile; at most one effect |
| Security | Visual/OCR/tree injection, redirect, secret request, malicious download | No authority expansion/exfiltration/execution; quarantine where required |
| Runtime | Controller/guest/network/artifact/effect-store crash | Lease fencing, safe degradation, no orphan input/effect |
| Cleanup | Revoke/delete/destroy failure | Quarantine, alert, no reuse |

## Go-live gate

- [ ] Scope is narrow, documented, and has explicit non-goals.
- [ ] Stable APIs/semantic actions replace GUI where available.
- [ ] General desktop actions run in disposable isolated environments.
- [ ] Policy and confirmation are application-enforced, exact-effect, and fail closed.
- [ ] Secrets, clipboard, files, downloads, profiles, and egress have independent boundaries.
- [ ] Terminal success and consequential effects use authoritative verification.
- [ ] Stuck, recovery, cancellation, takeover, unknown outcomes, and crash resume are tested.
- [ ] Severe-failure tests are zero-tolerance for the declared release tier.
- [ ] Repeated environment-specific task/grounding gates pass at supported settings.
- [ ] p95 latency, cost per compliant success, takeover rate, and pool capacity meet SLOs.
- [ ] Audit, deletion, cleanup, kill switch, rollback, and incident runbook are exercised.
- [ ] Residual risks and unsupported actions are visible to users/operators.

## Custom, framework, and hybrid alternatives

### Decision matrix

| Approach | Choose when | Benefits | Risks / limits |
|---|---|---|---|
| Deterministic API/workflow | Stable operations and postconditions exist | Highest reliability, security, speed, and auditability | Does not cover missing/visual-only UI |
| Traditional RPA/test automation | Workflow/layout is stable and variations are enumerable | Predictable selectors, mature scheduling, human-authored control flow | Brittle to unmodeled variation; maintenance grows with UI drift |
| Browser-agent framework | Task stays inside web pages and framework offers useful session/DOM/action tooling | Fast build, browser semantics, ecosystem | Browser only; framework profile/safety defaults need review |
| Provider-native computer tool | Supported model/action schema fits target and you will own execution | Model/tool compatibility and concise loop | Provider schema/safety/version churn; does not supply your environment/policy/effects |
| Custom thin loop | One workflow, small action set, team can own control plane | Minimal trusted surface and explicit guarantees | You own provider adapters, loop, telemetry, and upgrades |
| Broad agent framework | Existing organization already relies on its state/approval/tool abstractions and gaps are understood | Integration speed and reusable components | Easy to confuse framework callbacks/checkpoints with security/durability guarantees |
| Hybrid custom control plane + adapters — recommended | Production risk requires application-owned guarantees but models/providers/frameworks should remain replaceable | Clear ownership with reusable model/browser/VM pieces | More deliberate integration and conformance testing |
| User-assist / takeover | High impact, weak target identity, authentication, or inaccessible UI | Human retains commit authority | Lower automation rate; interaction design matters |
| No automation | Consequence is severe, validation weak, or environment cannot be isolated | Avoids unacceptable risk | Manual cost remains |

### Framework adoption questions

Before adopting a framework, verify in source/tests—not marketing:

1. Can tool calls be intercepted before execution with canonical arguments and current identity?
2. Does approval bind to an immutable effect digest and expire on changes?
3. What exactly stops on pause/cancel/timeout—scheduling, running actions, descendants, commits?
4. Are batches sequential and halted after first failure?
5. Can stale observations and lease generations fence late results?
6. How are screenshots and tool arguments redacted, retained, and exported?
7. Does persistence restore only model state, or also environment/effect ownership?
8. Can every provider/model/tool/image version be pinned and observed?
9. Can the framework run with your isolated executor and credential broker, without broad shell access?
10. Can you replace the framework while preserving application action/effect/event contracts?

Use the framework only for the questions it answers well. Wrap it behind application contracts for the rest.

## When not to use computer control

Choose another design when:

- the system has a stable API that covers the operation;
- the action is high impact and cannot be deterministically previewed, authorized, or reconciled;
- the agent must use the user's everyday desktop/profile/clipboard/credentials;
- the application blocks automation or requires bypassing a safety/consent mechanism;
- success is visually ambiguous and no authoritative validator exists;
- the environment cannot be reset or isolated by tenant/run;
- the product requires near-perfect precision that current end-to-end evaluations do not demonstrate;
- latency or cost is incompatible with a sequential screenshot loop;
- policy or law requires a human decision or action.

## Final engineering rule

The framework or provider may choose the next model token. The application must own what the agent is allowed to see, what it is allowed to attempt, what actually executes, what counts as success, and how damage is contained and recovered.

Return to the [blueprint overview](README.md) or review the [source-backed research packet](../../research/packets/computer-use-agent-blueprint.md).

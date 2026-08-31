# Research Packet: Production Computer-Use Agent Blueprint

> **Status:** Synthesis complete; supports the production blueprint guides  
> **Research window:** 2026-08-31  
> **Scope:** Screenshot/UI perception, browser versus general GUI control, accessibility and input APIs, application state, permissions, isolation, files/downloads/uploads, clipboard, credentials, prompt injection, irreversible effects, stuck detection, recovery, audit replay, evaluation, latency, cost, scaling, and deployment  
> **Evidence policy:** Primary sources and official repositories/specifications first; original papers for benchmarks and emerging techniques; provider claims treated as version-scoped rather than universal guarantees

## Guides supported

- [Computer-Use Agent Blueprint](../../agents/computer-use-agent/README.md)
- [Architecture and runtime selection](../../agents/computer-use-agent/architecture-and-runtime-selection.md)
- [Perception, grounding, and application state](../../agents/computer-use-agent/perception-grounding-and-application-state.md)
- [Actions, policy, and irreversible effects](../../agents/computer-use-agent/actions-policy-and-irreversible-effects.md)
- [Isolation, permissions, credentials, and data boundaries](../../agents/computer-use-agent/isolation-permissions-credentials-and-data-boundaries.md)
- [State, context, planning, stuck detection, and recovery](../../agents/computer-use-agent/state-context-planning-stuck-detection-and-recovery.md)
- [Observability, audit replay, and evaluation](../../agents/computer-use-agent/observability-audit-replay-and-evaluation.md)
- [Performance, cost, scaling, and operations](../../agents/computer-use-agent/performance-cost-scaling-and-operations.md)
- [Implementation roadmap, acceptance tests, and alternatives](../../agents/computer-use-agent/implementation-roadmap-acceptance-tests-and-alternatives.md)

## Research questions

1. What boundary distinguishes browser automation from general computer/desktop control?
2. When should a workflow use an API, DOM/browser protocol, application API, OS accessibility tree, or screenshot-based action?
3. Which current provider computer-use interfaces, action schemas, safety responses, and limitations are version-specific?
4. How should screenshots, semantic trees, OCR/screen parsing, and application state be fused without losing provenance?
5. Which coordinate, DPI, focus, window, display, and state-freshness failures cause mis-targeted actions?
6. What must a tool/action contract expose for deterministic policy, cancellation, verification, and recovery?
7. Which actions require exact-effect confirmation or user takeover, and when does approval become invalid?
8. How can non-idempotent GUI effects be reconciled after timeouts, crashes, or replay?
9. Which assets and ambient authorities are exposed by a browser profile or general desktop session?
10. Which isolation boundary is appropriate for browser-only, native desktop, hostile-content, multi-tenant, and high-impact use?
11. How should files, downloads/uploads, clipboard, credentials, keychains/vaults, and network egress cross the guest boundary?
12. What does current evidence say about visual/indirect prompt injection and defense limitations?
13. How should durable run state differ from model context, UI state, effect state, and audit artifacts?
14. How can a controller detect no progress, repeated actions, oscillation, and ambiguous outcomes?
15. What can audit replay honestly reproduce, and what remains nondeterministic?
16. Which visual grounding, desktop, mobile, browser, safety, and task benchmarks are relevant, and what do they not prove?
17. Where do latency and cost accumulate, and which optimizations preserve safety?
18. What should be custom, framework-provided, provider-native, or deterministic application code?

## Research method and breadth

Research used multiple angles rather than adopting one provider's vocabulary:

- current OpenAI, Anthropic, and Google computer-use documentation, official sample repositories, safety guidance, image/context guidance, and data-control notes;
- W3C WebDriver, WebDriver BiDi, Clipboard, and Trace Context specifications;
- official Playwright and Chrome DevTools Protocol documentation for browser semantics, state isolation, actionability, and profile/debugging security;
- Microsoft Windows UI Automation, input injection, Windows Sandbox, and credential-isolation documentation;
- Apple accessibility, screen-capture, keychain, and Virtualization documentation;
- GNOME AT-SPI and Linux kernel `uinput` documentation;
- original benchmark papers and official repositories for OSWorld, OSWorld 2.0, WindowsAgentArena, AndroidWorld, WebArena, VisualWebArena, WorkArena, ScreenSpot, and ScreenSpot-Pro;
- official/open research repositories for OmniParser, OpenCUA, and UI-TARS;
- original work on prompt injection and agentic-browser security, including WASP and VPI-Bench;
- research on efficiency, evidence-first reflection, trajectory data, and replay/evaluation, treated according to maturity;
- current OpenTelemetry and W3C tracing specifications for correlation, with explicit attention to sensitive content.

Research stopped when new searches largely repeated the same architecture boundaries: semantic-before-pixel control, application-owned policy/effects, disposable environments, confirmation at the point of risk, explicit verification, and environment-specific evaluation. Remaining uncertainty is primarily platform/provider churn and the absence of a universally secure prompt-injection defense.

## Version and date baseline

These observations were verified on 2026-08-31 and will age quickly.

| Surface | Baseline observed | Applicability note |
|---|---|---|
| OpenAI computer use | GA `computer` tool documented with ordered action arrays, custom and code-execution harness paths; `computer-use-preview` documented as legacy migration path; examples use current GPT-5.x models | Model/tool compatibility and image behavior are provider-version facts |
| Anthropic computer use | `computer_toolset_20260801` documented with separate member tools; earlier `computer_20251124` and older variants remain for some models/platforms; browser-use tool is distinct | Toolset availability differs by model and hosting platform |
| Google Gemini computer use | Browser, mobile, and desktop environments; Gemini 3.7 Flash recommended in current page; configurable confirmation policies; prompt-injection detection documented as opt-in for supported models | Page was updated in August 2026; exact policy/model support is volatile |
| WebDriver | W3C Recommendation baseline plus a July 2026 WebDriver working-draft series | Browser control standard, not a general desktop safety contract |
| WebDriver BiDi | W3C Working Draft dated 2026-06-29 | Event model is still a working draft |
| Clipboard API | W3C Working Draft dated 2026-06-24 | Browser API; security principles generalize, exact support varies |
| Chrome remote debugging | Chrome 136+ change requires a non-default user-data directory for remote-debugging switches | Reinforces dedicated-profile requirement |
| Windows Sandbox | Current docs say networking and clipboard are enabled by default; custom configuration can disable them and enable Protected Client | Defaults are unsuitable for sensitive computer-use agents |
| OSWorld | OSWorld-Verified refresh documented in 2025 | Pin repository/data/grader versions |
| OSWorld 2.0 | Current documented aligned release `v2026.06.24` | Do not mix code, task, asset, and mocked-site releases |
| OpenTelemetry | General semantic conventions 1.44.0 page; GenAI conventions moving/evolving | Pin emitted schema; do not log sensitive content by default |

## Pass 2 volatile-surface qualification ledger

All entries below were accessed on **2026-08-31**. “Rolling documentation” means the page is current but does not identify the exact installed/runtime version; deployments must record their own package, OS build, image, region, and configuration digests.

| Surface and primary source | Version/status observed | Deployment-specific limitation | Blueprint consequence |
|---|---|---|---|
| [Windows UI Automation](https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-uiautomationoverview) and [providers](https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-providersoverview) | OS API; rolling Microsoft documentation | App/framework providers determine tree and pattern coverage; custom controls can be opaque; element identity is not a durable business ID | Qualify per app/control/build and pair semantics with pixels/domain state |
| [Windows app UI testing guidance](https://learn.microsoft.com/en-us/windows/apps/develop/testing/) and [WinAppDriver repository](https://github.com/microsoft/WinAppDriver) | Microsoft says original WinAppDriver is no longer actively developed and recommends Appium's Windows driver; repository latest listed release is 1.2.1 from 2020 | WebDriver-style automation does not provide CUA policy, focus fencing, isolation, confirmation, or effect reconciliation; legacy compatibility and dependencies need independent testing | Do not select WinAppDriver as an unqualified current default; treat Appium Windows as a replaceable UIA adapter |
| [`winapp ui`](https://learn.microsoft.com/en-us/windows/apps/dev-tools/winapp-cli/ui-automation) / [repository](https://github.com/microsoft/winappCli) | Windows App Development CLI is **Public Preview**, active development; main may differ from public release | UIA pattern verbs and OS-level input verbs have different session/focus requirements; input needs unlocked interactive desktop and foreground target; control-specific gaps remain | Promising evaluation/agent adapter, not a stable universal production contract; pin a release and wrap/retest it |
| [`SendInput`](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-sendinput) and [UAC configuration](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/user-account-control/settings-and-configuration) | Current Win32/UAC documentation | OS-wide input, current keyboard state, UIPI/integrity restrictions, and secure desktop; failure does not cleanly identify UIPI | Minimum-integrity dedicated executor; foreground/lease checks; never bypass secure desktop for agent convenience |
| [Windows Sandbox configuration](https://learn.microsoft.com/en-us/windows/security/application-security/application-isolation/windows-sandbox/windows-sandbox-configure-using-wsb-file) | Windows 10/11; rolling docs | Defaults include networking and clipboard, Protected Client off, and vGPU/microphone conditions; mapped folders persist host effects | Explicit deny-by-default `.wsb` is required even for tests; general production pools still need lifecycle/isolation assessment |
| [macOS Accessibility trust](https://developer.apple.com/documentation/applicationservices/1459186-axisprocesstrustedwithoptions), [ScreenCaptureKit](https://developer.apple.com/documentation/screencapturekit), and [App Sandbox limits](https://developer.apple.com/documentation/security/protecting-user-data-with-app-sandbox) | Rolling Apple platform docs; ScreenCaptureKit is current capture framework | Accessibility, Automation/Apple Events, input monitoring, files, and screen recording are separate TCC/entitlement surfaces; App Sandbox forbids arbitrary Apple Events and assistive accessibility use; consent/deployment behavior varies by OS, signing, packaging, and MDM | Dedicated signed executor/VM/account; explicit consent and PPPC plan; no “cross-platform accessibility permission” abstraction |
| [Apple PPPC deployment payload](https://support.apple.com/guide/deployment/dep38df53c2a/web) | Current Apple deployment guidance | MDM can manage some privacy settings, but screen recording grants/denials and user-visible controls are service-specific; organization policy does not make on-screen consent model-authorizable | Record human-presence/consent mode and test the exact managed/unmanaged deployment |
| [AT-SPI](https://gnome.pages.gitlab.gnome.org/at-spi2-core/devel-docs/index.html), [XDG RemoteDesktop v2](https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.RemoteDesktop.html), and [ScreenCast v6](https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.ScreenCast.html) | AT-SPI rolling docs; portal interfaces v2/v6 | Distribution, compositor, portal backend, app accessibility, and D-Bus policy vary. Portal start normally presents user selection; restore tokens are single-use/revocable; old PipeWire node IDs can be reused | Prefer consent-mediated Wayland portals where available, bind streams by current serial/mapping identity, and qualify each desktop/backend |
| [Linux `uinput`](https://kernel.org/doc/html/latest/input/uinput.html) | Current kernel documentation | Creates virtual kernel input devices and is broader than an app/window action; device-node access is privileged | Keep outside untrusted model/process; use only in isolated guest with target/focus policy and explicit device lifecycle |
| [Playwright isolation](https://playwright.dev/docs/browser-contexts) and [BrowserType](https://playwright.dev/docs/api/class-browsertype) | Rolling 2026 docs; individual APIs annotate introduction versions | “Completely isolated” means test cookie/storage state between contexts, not host isolation. CDP attach is Chromium-only and documented lower fidelity; main Chrome profile automation is unsupported; artifact directories/downloads persist by configuration | Fresh dedicated automation profile in sandbox/VM, pinned browser/library, semantic-first control, explicit download/artifact cleanup |
| [Microsoft RDS session behavior](https://learn.microsoft.com/en-us/troubleshoot/windows-server/remote/terminal-server-startup-connection-application) | Page updated 2026-02-12 | Disconnect preserves processes/session state; reconnect can redraw at a client-dependent resolution | Reconnect increments display/capture generation and invalidates coordinates, observations, and UI-bound approvals without assuming effect rollback |
| [OmniParser](https://github.com/microsoft/OmniParser) | Repository lists v2.0.1 as latest release (2025-09-12) | Parser output is model-derived; code and component checkpoints have different licenses, including AGPL detector and MIT caption checkpoints; benchmark score is not per-app safety evidence | Pin code/weights/licenses; treat labels as uncertain sensor evidence; test locales, tiny/dense controls, overlays, and adversarial pixels |
| [Azure Vision FAQ](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/faq) / [Google Cloud Vision OCR](https://cloud.google.com/vision/docs/ocr) | Azure pages distinguish API 3.2/4.0 limits; Google is rolling service docs | Document/photo OCR contracts, file/size/quota/region/data handling, and supported languages do not imply GUI icon/target identity or action safety | Qualify OCR only for its sensor role and supported data/region/latency slices; never promote confidence to authority |
| [Vault leases](https://developer.hashicorp.com/vault/docs/concepts/lease) and [response wrapping](https://developer.hashicorp.com/vault/docs/concepts/response-wrapping) | Rolling HashiCorp documentation | Dynamic secret lease/revocation and one-use wrapping reduce exposure, but KV is different and no vault verifies that the agent selected the correct UI destination | Broker must bind secret to verified account/app/field/purpose and return redacted receipt; test expiry/revocation/outage |
| [S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html) | Rolling AWS service docs | Requires versioning; retention/legal hold protect object versions but permit new versions/delete markers; governance and compliance modes differ; immutability does not redact or enforce residency | Artifact store needs separate encryption, access, minimization, deletion/legal-hold, version-ID, and integrity design |
| [OpenTelemetry semantic conventions](https://opentelemetry.io/docs/specs/semconv/) | General semantic conventions 1.44.0; GenAI conventions moved to a separate repository and older registry attributes marked moved/deprecated | Semantic attributes may be unstable or sensitive; traces may be sampled and are not an audit ledger | Pin/translate telemetry schema; keep metrics, traces, logs, audit, and SLO records separate |
| [Firecracker production host setup](https://github.com/firecracker-microvm/firecracker/blob/main/docs/prod-host-setup.md) | Rolling project main; production guidance requires jailer/equivalent constraints | Linux host/guest only; operator owns host hardening, network filtering, microcode, resources, unique tenant/process, and device/backing-file choices | A microVM product name is not sufficient evidence of isolation; qualify the complete host/network/device/lifecycle design |

## Synthesis: conclusions that held across sources

### 1. Browser automation and general computer control are different products

W3C WebDriver and CDP expose browser-specific targets, DOM/accessibility state, navigation, storage, permissions, events, and downloads. Playwright adds locator re-resolution, actionability checks, and fresh browser contexts. Anthropic's current docs explicitly direct page-only workflows toward browser use and full desktop workflows toward computer use. OpenAI separates browser automation, VM-backed action execution, custom harnesses, and code-execution harnesses.

**Synthesis:** browser automation should remain inside a browser-specific environment and use semantic targets first. General computer control crosses applications and OS-global input/credential/file boundaries, so it needs a dedicated desktop session—normally a disposable VM—and stricter focus, device, clipboard, and account controls.

### 2. Hybrid semantic-plus-visual perception is the strongest practical default

OSWorld supports screenshots, accessibility trees, and terminal output; WindowsAgentArena and Microsoft tooling use UI Automation and/or screen parsing; OmniParser recovers interactable regions from pixels. OS accessibility frameworks expose role/name/state/action semantics but custom controls can be opaque. Screenshots reveal canvas/custom UI, occlusion, and visual outcome, but coordinates are weak identities.

**Synthesis:** prefer DOM/app/accessibility semantics for target identity and state, retain screenshots for coverage and verification, and preserve source/confidence. Do not flatten OCR/model-derived labels into trusted application state.

### 3. The model's action is always a proposal

All three provider APIs require the developer/application to execute client-side actions. Provider docs recommend isolation, allowlists, and human confirmation. Their built-in schemas and safety decisions simplify integration but do not authenticate the user, know application resource identity, reconcile external effects, or enforce a complete tenant policy.

**Synthesis:** map provider calls into an application-owned action contract, canonicalize target and risk, authorize deterministically, issue a single-use capability, execute through a narrow broker, and verify the result.

### 4. Observation freshness and focus are safety properties

OpenAI and Anthropic both document screenshot/action loops and coordinate mapping. Anthropic troubleshooting calls out resolution/DPI offset and small targets. Microsoft documents that `SendInput` is OS-wide, subject to UIPI, and does not clearly identify UIPI as the failure cause. Accessibility elements can be recreated; raw coordinates do not carry identity.

**Synthesis:** every action binds to an immutable observation version, active app/window/origin, display transform, and target reference. Re-resolve just before dispatch. Global input executes only while a controller holds an exclusive desktop lease and proves the intended foreground window.

### 5. Prompt-injection defenses reduce risk but do not establish authority

Anthropic documents automatic classifiers for supported official computer-use tool versions; Google documents opt-in screenshot prompt-injection detection for supported Gemini versions; provider guidance still recommends least privilege and action-time confirmation. WASP and VPI-Bench show successful redirection attempts against advanced agents. Research on agentic browsers also identifies cross-origin data risks when agents combine observation and action across security boundaries.

**Synthesis:** assume any visible/semantic content may control the model. Only direct authenticated user input and deterministic policy authorize actions. Cap impact with least privilege, egress/data boundaries, exact-effect confirmation, and isolation.

### 6. A disposable VM is the normal general-desktop boundary

Providers mention VMs or containers, but Anthropic labels its X11/VNC container sample as minimal and weakly separated. Windows Sandbox uses hardware-backed isolation but ships with networking and clipboard enabled by default. Apple Virtualization exposes explicit devices including graphics, keyboard/pointer, network, shared directories, and clipboard.

**Synthesis:** a fresh browser context is state isolation, a container is a shared-kernel process boundary, and a VM supplies a guest kernel/desktop boundary. Choose based on assets/adversary. General desktop, hostile content, downloads, multi-tenancy, or high-value credentials normally require a VM with bridges denied by default.

### 7. Credentials and clipboard must stay outside model context

Provider security guidance warns against exposing login data, while one Anthropic prompting section describes putting credentials in prompt XML when login is required. Chrome hardened remote debugging specifically because of cookie/credential extraction abuse. W3C calls clipboard access a powerful feature and documents phishing, self-XSS, hidden-data, file, and PII risks. OS vaults protect stored secrets but cannot prevent abuse of an already authenticated UI.

**Synthesis:** resolve the provider contradiction in favor of the stronger security rule: never give the model raw credentials. Use opaque, short-lived fill/sign brokers or user takeover. Disable host clipboard bridging; use scoped source-to-destination transfers for non-secret data.

### 8. Consequential UI effects require prepare/approve/commit/reconcile

OpenAI, Anthropic, and Google converge on confirmation immediately before sending, submitting, purchasing, deleting, sharing, or other hard-to-reverse effects. OpenAI distinguishes user takeover for password completion and bypassing safety barriers. UI clicks themselves are not idempotent, and a timeout after submit is ambiguous.

**Synthesis:** prepare the exact target/content/diff, compute a canonical digest, obtain a one-use approval, re-read state and policy at commit, execute once with an effect ID, and reconcile authoritative state. Never approve a generic tool name or retry an unknown effect blindly.

### 9. Ordered action batches are useful only within one safety boundary

Anthropic and OpenAI both document ordered multi-action responses. Anthropic explicitly says later actions should not execute after an earlier failure because their assumptions no longer hold.

**Synthesis:** execute sequentially, stop at first failure, and end with an observation. Split batches before navigation uncertainty, secret entry, required postcondition, confirmation, or irreversible commit. Latency savings never override verification.

### 10. Durable state and model context must be separate

Computer-use contexts accumulate screenshots rapidly. Provider guidance recommends pruning, caching, and summarizing. None of that preserves desktop ownership, approval validity, external-effect status, or authoritative application state.

**Synthesis:** keep task, run-control, milestone, observation, action/effect, approval, business, and conversation state distinct. Build model context as a versioned projection. On crash, acquire a new lease generation, inspect current environment/business state, reconcile possible effects, and resume only from a verified milestone.

### 11. Stuck detection needs state/action cycles and verified progress

Provider loop examples expose maximum iterations to cap runaway cost. UI-TARS describes reflection and milestone recognition. Recent evidence-first reflection research reports gains from explicitly comparing action-induced visual differences, but it is too new to treat as settled. UI automation literature and process-centric agent analysis both identify repeated cycles/stagnation as useful stuck signals.

**Synthesis:** combine milestone evidence, application versions, semantic change, masked visual change, repeated action signatures, and cycle detection. A deterministic controller sets thresholds and recovery budgets. The model may diagnose, but it cannot decide to ignore the stop rule.

### 12. Audit replay is reconstruction, not deterministic live replay

Official samples and research repositories increasingly preserve trajectories, screenshots, actions, and verifier artifacts. Yet a GUI depends on app version, network, time, external state, authentication, animations, and nondeterministic model output.

**Synthesis:** provide non-executing forensic reconstruction, offline decision replay, and separately labeled isolated re-execution. Never replay real effects, approvals, credentials, or original idempotency keys.

### 13. Evaluation must cover grounding, outcomes, policy, recovery, and operations

ScreenSpot evaluates grounding; OSWorld/OSWorld 2.0 and WindowsAgentArena evaluate desktop tasks; AndroidWorld evaluates mobile tasks; WebArena/VisualWebArena/WorkArena evaluate browser tasks. WASP/VPI-Bench probe injection. OSWorld 2.0 warns that release components must align. Provider benchmark announcements show that scaffold and task choice materially change scores.

**Synthesis:** use external benchmarks for capability slices, then gate the full release on application-specific synthetic tasks with authoritative validators, repeated trials, failure injection, severe-failure zero tolerances, supported display/locale/app slices, latency, cost, and cleanup.

### 14. Cost and latency are trajectory properties

Every screenshot/action usually incurs a sequential round trip. Provider guidance shows that image resolution/history and reasoning effort change tokens and latency. OSWorld-Human reports that model planning/reflection dominates much latency and later steps can slow as trajectories grow.

**Synthesis:** remove GUI steps first, use semantic reads, crop/zoom progressively, prune old model-visible images, wait on events, batch only safe mechanics, and use evaluated routing. Measure cost per verified policy-compliant success, including VM, failure, recovery, and human review.

### 15. A hybrid control plane is the default production ownership model

Provider-native tools and frameworks can own model/tool formatting, loops, history, and tracing. Browser automation libraries own useful locator/action mechanics. OS APIs own element/input primitives. None owns the full product's identity, delegated authority, data boundary, effect semantics, evidence retention, or release decision.

**Synthesis:** keep application action/effect/event contracts stable; place providers, grounding models, browser libraries, and agent frameworks behind adapters. Use deterministic workflows or RPA for known control flow and user-assist/no automation for unacceptable residual risk.

## Important contradictions and resolutions

| Apparent contradiction | Evidence | Resolution used in blueprint |
|---|---|---|
| Container or VM are presented together as isolation options | Provider safety docs say dedicated VM or container; Anthropic demo says components are weakly separated; containers share host kernel | Choose by threat model. Browser prototype may use hardened container; general desktop/hostile/multi-tenant use normally gets a VM. |
| Playwright calls browser contexts “completely isolated” | The statement concerns test cookie/storage state in one browser; it is not an OS security claim | Call it browser test-state isolation, never a desktop/host boundary. |
| Microsoft historically supplied WinAppDriver versus current Windows guidance | WinAppDriver repository remains available, while current Microsoft testing guidance says it is no longer actively developed and recommends Appium's Windows driver; `winapp ui` is newer but Public Preview | Existing suites may remain on a pinned, supported legacy deployment, but new designs must qualify Appium Windows, direct UIA, or preview `winapp ui` rather than assuming a maintained WinAppDriver product. |
| Microsoft `winapp ui` says it works across common Windows app frameworks versus UIA provider reality | The tool uses UIA patterns where available and real input for gaps; its docs name windowless/rich-edit and interactive-desktop limitations | “Works with” is not complete semantic coverage. Qualify each operation/control and preserve pixel/input fallback as a distinct higher-risk path. |
| macOS App Sandbox improves isolation versus desktop automation needs broad cross-app control | Apple documents that sandboxed apps cannot use assistive accessibility APIs or send arbitrary Apple Events; Accessibility/Automation/capture/files are separately consented | Do not claim a fully sandboxed universal macOS executor. Use a dedicated signed component/VM/account, narrow entitlements/targets, and product-specific consent/takeover. |
| Linux direct `uinput` can automate input versus Wayland portals preserve user control | `uinput` is a privileged virtual device; XDG RemoteDesktop/ScreenCast portals create consent-mediated, revocable sessions with backend-dependent behavior | Prefer portal/session APIs for attended Wayland use; use `uinput` only inside an isolated dedicated guest when the broader device authority is justified. |
| A remote desktop reconnect “resumes” the session versus screen identity remains stable | Microsoft documents process/session preservation and resolution redraw on reconnect | Preserve domain/run state but invalidate screen geometry, capture stream, targets, and UI-bound approval/readback. |
| Cloud OCR and screen parsers return boxes/confidence versus accessibility semantics | OCR services are designed around text/document/image extraction; screen parsers infer interactable regions; neither owns application resource identity | Keep OCR/parser output as provenance-tagged evidence and require domain/semantic/readback checks for consequential action. |
| Pure screenshots are universal versus semantics are more reliable | Provider CU tools use screenshots; OS/browser automation exposes semantic trees; custom controls can be opaque | Hybrid: semantic target/state first, screenshot fallback and visual verification, provenance preserved. |
| Providers recommend different screenshot resolutions/detail | OpenAI currently favors original detail and gives 1440×900/1600×900 examples; Anthropic gives other baselines and zoom guidance | Version and evaluate resolution per provider/model/app/DPI; store explicit coordinate transform. |
| Batch actions reduce latency versus verification after each meaningful change | OpenAI/Anthropic support ordered batches; later actions depend on earlier success | Batch only within one target/risk/verification boundary; stop on first failure and observe at boundary. |
| Anthropic suggests XML-wrapped login credentials in a prompt versus warning not to expose login data | Both appear in current official material | Follow the stronger least-privilege security rule: opaque broker or user takeover; no raw secrets in prompt. |
| Prompt-injection detection is automatic versus opt-in | Anthropic documents automatic classifiers for supported official tool; Google documents opt-in setting for supported models | Treat as provider/tool-version capability. Enable where available, log it, and assume bypass is possible. |
| Log full trajectories/screenshots for safety versus minimize sensitive retention | Provider best practices recommend logging; privacy/data-control sources show screenshots and tool arguments can contain sensitive data | Record structured metadata always; content artifacts tiered, redacted, encrypted, short-lived, and access-controlled. |
| More reasoning improves difficult tasks versus UI actions are mechanical and latency-sensitive | Provider tests show effort-dependent quality/cost; Anthropic notes more thinking does not always help | Use one evaluated default and bounded escalation for ambiguity/recovery, not maximum reasoning every step. |
| Checkpoint/VM snapshot enables recovery versus effects cannot be rolled back | Environment state can be restored; remote email/purchase/file effects remain external | Reconcile by effect identity before retry/reset; snapshots recover environment, not the world. |
| Recorded trajectory is “replayable” versus GUI nondeterminism | Tools/research use replay for viewing/training/eval; live environments change | Separate forensic replay, offline decision replay, and isolated re-execution; never imply exact live reproduction. |
| Provider benchmark scores suggest broad competence versus benchmark scope/version issues | Scores depend on model/scaffold/task; OSWorld 2.0 requires aligned release components | Treat results as release-scoped evidence and require internal environment-specific gates. |

## Evidence maturity

### Strong and operationally actionable

- provider APIs are client-executed and require application-side environment/action handling;
- browser automation has semantic targets and state not available to arbitrary desktop pixels;
- OS input/focus/permission and coordinate transforms are platform-specific failure boundaries;
- prompt injection can enter through on-screen and semantic content;
- least privilege, isolation, confirmation, and audit are required across provider guidance;
- browser profiles, clipboard, shared folders, network, and device bridges expand the authority boundary;
- UI effects can be ambiguous and non-idempotent;
- benchmark/environment versions affect reproducibility.

### Supported synthesis, not a standardized product contract

- exact internal action/effect/observation schemas;
- recommended recovery ladder and stuck thresholds;
- default VM choice for general desktop versus a particular hardened container technology;
- artifact retention tiers and release-gate numbers;
- specific language split between controller and native executor.

These should be adapted to the product's platform, assets, compliance, and SLOs.

### Emerging/experimental

- universal prompt-injection detection sufficient for autonomous high-impact use;
- model-based outcome verifiers replacing authoritative state;
- evidence-first reflection gains generalizing across applications/models;
- reliable deterministic replay of arbitrary desktop trajectories;
- benchmark performance as a proxy for cross-platform production reliability;
- a universal cross-platform accessibility/action schema.

## Source ledger

### Provider and product sources

1. [OpenAI computer use guide](https://developers.openai.com/api/docs/guides/tools-computer-use) — integration paths, action loop, image detail, confirmation, isolation, migration.
2. [OpenAI CUA sample app](https://github.com/openai/openai-cua-sample-app) — browser-focused native/code modes, scenario verification, replay contracts, sample limitations.
3. [OpenAI Computer-Using Agent](https://openai.com/index/computer-using-agent/) — original CUA framing and benchmark examples; historical, not current API contract.
4. [OpenAI API data controls](https://platform.openai.com/docs/models/default-usage-policies-by-endpoint) — image/file processing and endpoint retention controls.
5. [Anthropic computer use documentation](https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool) — current toolset, browser distinction, batches, errors, resolution, security, limitations, retention.
6. [Anthropic computer-use quickstart](https://github.com/anthropics/anthropic-quickstarts/tree/main/computer-use-demo) — X11/VNC container reference, explicit weak separation, versioned tools, screenshot scaling.
7. [Anthropic computer/browser-use best practices](https://claude.com/blog/best-practices-for-computer-and-browser-use-with-claude) — prompt-injection layers, confirmation, context/image cost, resolution/effort guidance.
8. [Google Gemini computer use documentation](https://ai.google.dev/gemini-api/docs/computer-use) — browser/mobile/desktop environments, safety policies, confirmation, injection detection, logging, environment consistency.

### Browser standards and official automation documentation

9. [W3C WebDriver](https://www.w3.org/TR/webdriver/) — browser/user-agent remote control and introspection.
10. [W3C WebDriver BiDi](https://www.w3.org/TR/webdriver-bidi/) — bidirectional browser control/events, current working-draft status.
11. [Playwright locators](https://playwright.dev/docs/locators) — semantic locators and re-resolution.
12. [Playwright actionability](https://playwright.dev/docs/actionability) — visibility/stability/event/enabled checks and auto-wait.
13. [Playwright browser-context isolation](https://playwright.dev/docs/browser-contexts) — fresh browser test-state contexts.
14. [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/) — DOM/network/input/browser instrumentation and version caveats.
15. [CDP Accessibility domain](https://chromedevtools.github.io/devtools-protocol/tot/Accessibility/) — AX tree nodes, events, and performance note.
16. [Chrome remote-debugging security change](https://developer.chrome.com/blog/remote-debugging-port) — dedicated non-default profile requirement from Chrome 136.

### OS automation, isolation, identity, and data-boundary sources

17. [Windows UI Automation overview](https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-uiautomationoverview) — accessibility/automation object model.
18. [Windows UI Automation providers](https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-providersoverview) — provider/custom-control coverage and opacity.
19. [Using UI Automation for testing](https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-usefortesting) — trees, elements, properties, patterns, events.
20. [Windows `SendInput`](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-sendinput) — system input stream, serialization, UIPI limitation.
21. [Windows Sandbox configuration](https://learn.microsoft.com/en-us/windows/security/application-security/application-isolation/windows-sandbox/windows-sandbox-configure-using-wsb-file) — default network/clipboard and configurable devices/folders/protected client.
22. [Windows Sandbox architecture](https://learn.microsoft.com/en-us/windows/security/application-security/application-isolation/windows-sandbox/windows-sandbox-architecture) — isolation, image, memory, GPU design.
23. [Windows Credential Guard](https://learn.microsoft.com/en-us/windows/security/identity-protection/credential-guard/how-it-works) — VBS-protected credential scope and limitations.
24. [Apple accessibility trust](https://developer.apple.com/documentation/applicationservices/1459186-axisprocesstrustedwithoptions) — trusted accessibility client requirement.
25. [Apple screen-capture preflight](https://developer.apple.com/documentation/coregraphics/cgpreflightscreencaptureaccess%28%29) — separate capture permission surface.
26. [Apple Virtualization](https://developer.apple.com/documentation/virtualization) — VM lifecycle and explicit device/clipboard/shared-directory surfaces.
27. [Apple Keychain Services](https://developer.apple.com/documentation/security/keychain-services) — encrypted credential storage.
28. [GNOME AT-SPI developer guide](https://gnome.pages.gitlab.gnome.org/at-spi2-core/devel-docs/index.html) — Linux accessibility infrastructure.
29. [AT-SPI Accessible interface](https://gnome.pages.gitlab.gnome.org/at-spi2-core/devel-docs/doc-org.a11y.atspi.Accessible.html) — accessible object properties and names.
30. [Linux kernel `uinput`](https://kernel.org/doc/html/latest/input/uinput.html) — user-space virtual input devices and privileged event path.
31. [W3C Clipboard API](https://www.w3.org/TR/clipboard-apis/) — powerful-feature permissions, PII, phishing, self-XSS, rich-content and file risks.

### Observability and trace sources

32. [W3C Trace Context](https://www.w3.org/TR/trace-context/) — cross-service trace identity and propagation.
33. [OpenTelemetry semantic conventions](https://opentelemetry.io/docs/specs/semconv/) — versioned spans/events/metrics conventions.
34. [OpenTelemetry GenAI attributes registry](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/) — agent/model/tool/usage fields and sensitive-content warnings.

### Evaluation, perception, and computer-use research

35. [OSWorld paper](https://arxiv.org/abs/2404.07972) and [official repository](https://github.com/xlang-ai/OSWorld) — cross-OS desktop environment, observation types, state-based tasks, Verified refresh.
36. [OSWorld 2.0 paper](https://arxiv.org/abs/2606.29537) and [official repository](https://github.com/xlang-ai/OSWorld-V2) — long-horizon workflows, dynamic challenges, release alignment warning.
37. [WindowsAgentArena paper](https://arxiv.org/abs/2409.08264) and [official repository](https://github.com/microsoft/WindowsAgentArena) — scalable Windows evaluation and multimodal agents.
38. [AndroidWorld paper](https://arxiv.org/abs/2405.14573) and [official repository](https://github.com/google-research/android_world) — dynamically parameterized Android tasks and emulator environment.
39. [WebArena paper](https://arxiv.org/abs/2307.13854) and [official repository](https://github.com/web-arena-x/webarena) — self-hosted functional web tasks.
40. [VisualWebArena](https://arxiv.org/abs/2401.13649) — visually grounded web tasks.
41. [WorkArena repository](https://github.com/ServiceNow/WorkArena) — enterprise knowledge-work browser tasks and BrowserGym integration.
42. [SeeClick / ScreenSpot repository](https://github.com/njucckevin/SeeClick) — cross-platform GUI grounding dataset.
43. [ScreenSpot-Pro](https://arxiv.org/abs/2504.07981) — high-resolution professional GUI grounding.
44. [Microsoft OmniParser](https://github.com/microsoft/OmniParser) — screenshot-to-structured-element parser, model assets, licensing and tooling.
45. [UI-TARS](https://arxiv.org/abs/2501.12326) and [repository](https://github.com/bytedance/UI-TARS) — native screenshot agent, unified actions, reflection/milestones.
46. [OpenCUA](https://github.com/xlang-ai/OpenCUA) — cross-platform trajectory collection and synchronized screenshots/actions/accessibility trees.
47. [OSWorld-Human](https://openreview.net/pdf?id=sV3n6mYy7J) — temporal efficiency and trajectory-step latency analysis.
48. [Evidence-First Reflection](https://arxiv.org/abs/2608.24015) — recent action-difference reflection evidence; emerging.

### Security and adversarial evaluation research

49. [WASP](https://arxiv.org/abs/2504.18575) — realistic isolated web-agent prompt-injection benchmark and attack progression.
50. [VPI-Bench](https://openreview.net/pdf?id=UMauKu2azg) — visual prompt injection for computer/browser-use agents across platforms.
51. [Agentic Browsers and the Same-Origin Policy](https://agent-security.cs.washington.edu/agentic_browsers_sop.html) — cross-origin risks and architecture comparisons in agentic browsers.

## Claim-to-source map

| Blueprint claim | Strongest sources |
|---|---|
| Browser-only work should use browser semantics before desktop pixels | W3C WebDriver; Playwright locators/actionability; Anthropic/OpenAI CU docs |
| General desktop work expands credential/file/device/focus boundary | Anthropic/OpenAI/Google safety docs; Windows/Apple/Linux platform docs |
| Provider tool calls are client-executed proposals | OpenAI, Anthropic, Google CU docs |
| Coordinate mapping and resolution must be provider/app tested | OpenAI image/detail guidance; Anthropic scaling/troubleshooting |
| Fresh browser context is not a host security boundary | Playwright context scope; provider VM/container guidance; Windows Sandbox/Apple VM docs |
| Prompt injection remains a real CUA threat | Provider safety docs; WASP; VPI-Bench; agentic-browser SOP research |
| Exact-effect confirmation should occur at point of risk | OpenAI, Anthropic, Google guidance |
| Clipboard and browser profiles are sensitive cross-boundary state | W3C Clipboard; Chrome remote-debugging security change; Windows Sandbox defaults |
| Semantic-plus-visual observations are complementary | OSWorld; WindowsAgentArena; OmniParser; platform accessibility docs |
| GUI effects need idempotency/reconciliation | Provider confirmation/loop docs plus general external-effect semantics; synthesized for GUI timeouts |
| Audit replay is not deterministic live replay | Benchmark/replay repositories plus GUI/environment nondeterminism; synthesis |
| External benchmarks do not replace application-specific evals | Benchmark task scopes/version notes; provider scaffold-dependent results |
| Cost/latency should be measured over full trajectory | Anthropic context guidance; OpenAI image guidance; OSWorld-Human |

## Limitations and unanswered questions

1. Provider documentation and model availability changed during the 2026 research window and may change again without stable long-term tool contracts.
2. macOS accessibility/screen-capture behavior depends on TCC, signing, packaging, OS version, and deployment management; public API pages do not fully specify all operational edge cases.
3. Linux desktop security varies significantly across X11, Wayland compositors, portals, D-Bus policy, distribution, and remote-desktop stacks; this packet does not endorse one universal Linux executor.
4. Containers, hardened containers, VMs, remote desktop pools, and managed browser services have implementation-specific escape surfaces; the blueprint defines evaluation questions, not a certified boundary.
5. No reviewed source demonstrates a complete prompt-injection defense for arbitrary visible content and high-impact autonomous action.
6. Model/verifier benchmark scores can be contaminated by training exposure, scaffold tuning, grader error, and release drift; this packet does not reproduce leaderboard claims.
7. Exact severe-failure thresholds, retention periods, and risk classifications must come from the application, jurisdiction, and organization.
8. Accessibility trees themselves can contain attacker-controlled text and can differ from the visible screen; source disagreement needs product-specific handling.
9. UI replay fidelity across real external services remains fundamentally limited; deterministic effect simulation requires application-owned test doubles.
10. Credential brokers for arbitrary desktop fields are platform/application-specific and can still be abused to authenticate a malicious destination unless target identity is strong.
11. Remote desktop focus/input/secure-desktop behavior varies by protocol and host configuration; each chosen platform requires fault-injection testing.
12. Recent evidence-first reflection and 2026 benchmarks are useful but too new to be treated as long-established production practice.

## Refresh triggers

Re-research immediately when:

- OpenAI, Anthropic, or Google changes computer/browser-use tool types, action members, safety decisions, injection classifiers, image processing, model compatibility, retention, or regional availability;
- a provider preview becomes GA, is deprecated, or changes migration path;
- Windows, macOS, Linux/Wayland, Playwright, WebDriver, or Chrome changes capture, accessibility, input, profile/debugging, clipboard, or virtualization behavior;
- OSWorld/OSWorld 2.0, WindowsAgentArena, AndroidWorld, WebArena, WorkArena, ScreenSpot, WASP, or VPI-Bench publishes a new release, task set, grader, contamination notice, or evaluation correction;
- a credible prompt-injection or browser/desktop escape invalidates an allowlist, classifier, origin, or isolation assumption;
- the product adds an app, OS, locale, display configuration, browser profile, credential type, download, device, tenant, effect class, or user-autonomy level;
- a production incident reveals an unmodeled focus, state, replay, cleanup, or external-effect failure;
- OpenTelemetry GenAI semantic conventions stabilize/move again and the telemetry schema is being upgraded.

Even without a trigger:

- check provider/platform facts quarterly;
- re-run application-specific evaluation before every model, tool, executor, perception, policy, base-image, OS, browser, or app upgrade;
- review the threat model and incident/failure suite at least twice yearly;
- revalidate benchmark release alignment before publishing any comparative result.

## Final research conclusion

The safest useful computer-use agent is not the most autonomous one. It is the one whose application knows exactly which screen state was observed, which target and effect were proposed, which authority allowed it, which isolated environment executed it, which evidence proved the outcome, and when to stop rather than guess.

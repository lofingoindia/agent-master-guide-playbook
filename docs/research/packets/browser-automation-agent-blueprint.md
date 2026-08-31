# Research Packet: Production Browser Automation Agent Blueprint

> Status: complete for the 2026-08-31 documentation baseline  
> Research window: 2026-08-30 to 2026-08-31  
> Scope: architecture, browser mechanics, perception, sessions, security, effects, reliability, observability, evaluation, deployment, operations, and framework alternatives  
> Primary deliverable: [Production Browser Automation Agent Blueprint](../../agents/browser-automation-agent/README.md)

This packet records the evidence behind the blueprint. It distinguishes maintained documentation and specifications from repository issues, vendor claims, and emerging research. It should be refreshed before adopting a newer browser/framework/model baseline or expanding to a materially different risk class.

## Guides supported

| Guide | Research themes |
|---|---|
| [Blueprint overview](../../agents/browser-automation-agent/README.md) | Production invariants, reference architecture, operating modes, launch decision |
| [Requirements and threat model](../../agents/browser-automation-agent/requirements-and-threat-model.md) | Authority, assets, origins, CSRF, data flows, action risk, approvals |
| [Architecture and runtime](../../agents/browser-automation-agent/architecture-and-runtime.md) | Deterministic/model/hybrid, DOM/AX/vision, protocols, languages, models, frameworks |
| [Tool and effect contracts](../../agents/browser-automation-agent/tool-and-effect-contracts.md) | Typed state, events, capabilities, effects, navigation, downloads, uploads |
| [Sessions, context, and security](../../agents/browser-automation-agent/sessions-context-and-security.md) | Auth state, secrets, MFA, prompt injection, egress, browser/host isolation, memory |
| [Reliability and recovery](../../agents/browser-automation-agent/reliability-and-recovery.md) | Readiness, stale state, retries, idempotency, uncertain outcomes, checkpoints, replay |
| [Observability, evaluation, and testing](../../agents/browser-automation-agent/observability-evaluation-and-testing.md) | Traces, privacy, task/safety/trajectory evals, benchmarks, attacks, acceptance gates |
| [Deployment, cost, and operations](../../agents/browser-automation-agent/deployment-cost-and-operations.md) | Worker topology, isolation, supply chain, capacity, cost, SLOs, incidents |
| [Build roadmap and alternatives](../../agents/browser-automation-agent/build-roadmap-and-alternatives.md) | Stages 0–6, release gates, no-agent/API/RPA choices, custom/framework/hybrid choices, framework responsibility |

## Version and standards baseline

Versions are observations from the research date, not unbounded “latest” claims.

| Component | Observed baseline | Interpretation |
|---|---|---|
| Playwright | `1.62.1`, published 2026-07-30 | Blueprint uses exact 1.62.1 behavior as the research baseline; the release included accessibility-snapshot fixes, so AX behavior is version-sensitive |
| Playwright MCP | `0.0.79`, published 2026-08-06, with a dated Playwright alpha dependency in its manifest | Rapid, pre-1.0, and tied to alpha internals; useful tooling but high refresh frequency |
| Browser Use | GitHub release `0.13.8` on 2026-08-16; 0.13 introduced a Rust rebuild marked beta | Rapid release cadence and architectural churn require exact pinning and version-scoped issue review |
| Stagehand | JavaScript package `4.0.2`, published 2026-08-20; 3.7 maintenance releases continued afterward | A recent major transition plus parallel component streams; the current public caching page still uses a `/v3/` route, so pin and qualify the SDK, embedded extension/protocol, provider path, documented behavior, and lifecycle semantics as separate inputs |
| Puppeteer | `25.9.0`, published 2026-08-25 | Chrome defaults to CDP and Firefox to BiDi; the documented BiDi support matrix still has material feature gaps |
| Selenium | `4.48.0`, published 2026-08-27 | BiDi migration and bindings move independently; verify the exact binding/driver/browser feature tuple |
| WebDriver BiDi | Editor's Draft dated 2026-08-25 | Standards direction is strong, but the document and implementations remain live |
| Browserbase | Node SDK `2.19.0`, published 2026-08-26; platform documentation observed 2026-08-31 | Managed Chromium sessions via CDP, contexts, files, recordings, regions, live view, and keep-alive require explicit data/lifecycle qualification |
| Browserless | platform/open-source documentation `2.56.0`; release dated 2026-08-20 | CDP extensions, profiles, persisted sessions, files, live URLs, proxies, and CAPTCHA features have provider- and client-specific constraints |
| WAI-ARIA | 1.2 Recommendation used for stable accessibility-tree semantics; 1.3 work is newer | AX is a projection parallel to DOM, not a complete rendered-page truth |
| WebAuthn | Level 3 Candidate Recommendation dated 2026-05-26 | User presence/verification and relying-party scoping remain security signals, not automation obstacles |
| OpenTelemetry | Current HTTP and GenAI semantic-convention documentation | GenAI conventions evolve; keep application effect semantics stable and versioned |

### Baseline sources

- [Playwright npm versions](https://www.npmjs.com/package/playwright?activeTab=versions)
- [Playwright 1.62.1 release](https://github.com/microsoft/playwright/releases/tag/v1.62.1)
- [Playwright MCP package manifest](https://github.com/microsoft/playwright-mcp/blob/main/package.json)
- [Browser Use releases](https://github.com/browser-use/browser-use/releases)
- [Stagehand npm versions](https://www.npmjs.com/package/@browserbasehq/stagehand)
- [Puppeteer 25.9.0 release](https://github.com/puppeteer/puppeteer/releases/tag/puppeteer-v25.9.0)
- [Selenium 4.48.0 release](https://github.com/SeleniumHQ/selenium/releases/tag/selenium-4.48.0)
- [Selenium WebDriver BiDi documentation](https://www.selenium.dev/documentation/webdriver/bidi/)
- [WebDriver BiDi Editor's Draft](https://w3c.github.io/webdriver-bidi/)
- [WAI-ARIA 1.2](https://www.w3.org/TR/wai-aria/)
- [WebAuthn Level 3](https://www.w3.org/TR/webauthn-3/)
- [Browserbase Node SDK](https://www.npmjs.com/package/@browserbasehq/sdk)
- [Browserless 2.56.0 release](https://github.com/browserless/browserless/releases/tag/v2.56.0)

## Research questions

### Product and architecture

- When is deterministic automation better than a model-assisted or visual agent?
- What should the model own, and what must remain deterministic and application-owned?
- How should protocol, browser library, language, model, framework, and deployment choices be made?
- Which browser-agent frameworks accelerate implementation without becoming a security boundary?

### Browser state and perception

- What do DOM, accessibility, network, and screenshot representations expose or omit?
- How should pages, frames, popups, redirects, document lifetimes, downloads, and uploads be modeled?
- What browser actionability and locator guarantees exist, and where do they stop?

### Security

- How can a page exploit an authenticated, cross-origin-capable agent?
- What do browser same-origin, site isolation, and sandbox controls actually contain?
- How should credentials, storage state, MFA/WebAuthn, permissions, files, and existing browser profiles be handled?
- Which prompt-injection defenses are architecture controls rather than model claims?

### Reliability and effects

- Which browser actions can be retried safely?
- How should the system handle a timeout after a possibly committed effect?
- What makes an idempotency mechanism real, and what is only a cache or replay feature?
- What state can survive a worker crash, and what must be reconstructed?

### Evaluation and operations

- Which artifacts are needed to debug without replaying production effects?
- How should task success, safety, trajectory, reliability, cost, and latency be scored separately?
- What do public browser benchmarks measure, and what do they not prove?
- What worker isolation, admission control, SLO, rollout, and incident capabilities are required?

## Research method and breadth

The research prioritized:

1. browser automation documentation and source repositories;
2. web standards and browser security design documents;
3. OWASP security guidance;
4. official model-provider computer-use/prompt-injection guidance;
5. framework documentation, release notes, and source manifests;
6. version-scoped repository issues as implementation evidence, clearly labeled as reports rather than specifications;
7. benchmark repositories and peer-reviewed or primary papers;
8. current observability specifications.

Search and review covered multiple queries and source paths for:

- Playwright locators, actionability, ARIA snapshots, contexts, pages, authentication, downloads, uploads, tracing, Docker, MCP profiles, extension mode, storage, origin flags, and security notes;
- CDP DOMSnapshot and Browser domains;
- Selenium/WebDriver BiDi browsing contexts, downloads, and protocol transition;
- browser sandbox/process/site-isolation design;
- same-origin, Fetch Metadata, CSRF, SSRF, file-upload, secrets, and WebAuthn controls;
- prompt injection and computer-use containment;
- RFC HTTP idempotency and a production API idempotency implementation;
- Stagehand, Browser Use, Skyvern, BrowserGym/AgentLab, and Playwright MCP;
- Browserbase and Browserless session creation, contexts/profiles, reconnect, timeouts, concurrency, regions, recordings, uploads/downloads, proxies, CAPTCHA/stealth features, and live human control;
- Playwright native versus CDP fidelity, Puppeteer CDP/BiDi support, Selenium BiDi migration, service-worker interception, and CDP stability boundaries;
- site terms/robots/verified-agent guidance and reCAPTCHA test-environment guidance;
- WebArena, VisualWebArena, WorkArena, AgentDojo, WASP, DoomArena, and WebArena Verified;
- OpenTelemetry HTTP/GenAI semantics and W3C Trace Context.

Research stopped when additional searches largely repeated the same architectural conclusions: application-owned authority/effects, multi-layer perception, per-run session isolation, external egress enforcement, reconciliation before retry, and task-specific evaluation.

## Evidence synthesis

### 1. Deterministic first, guarded hybrid second

Playwright's locators and actionability provide a strong deterministic baseline: a click can require a unique locator, visibility, stability, event reception, and enabled state. Locators re-resolve against the current DOM, reducing stale element handles. The documentation recommends user-facing roles, labels, and test IDs over brittle CSS/XPath.

This supports a deterministic-first design for known pages. It does not prove that the resolved button represents the user's intended business effect. A model can assist selection or recovery, but the application must bind the target to origin, document epoch, semantics, and approval.

Strong sources:

- [Playwright actionability](https://playwright.dev/docs/actionability)
- [Playwright locators](https://playwright.dev/docs/locators)
- [Playwright best practices](https://playwright.dev/docs/best-practices)
- [Stagehand npm package](https://www.npmjs.com/package/@browserbasehq/stagehand)
- [Skyvern hybrid automation overview](https://github.com/Skyvern-AI/skyvern/blob/main/docs/developers/browser-automations/overview.mdx)

Decision derived: use deterministic skills for known states and all sensitive commits; use a model for bounded interpretation, candidate selection, or recovery where measured variation justifies it.

### 2. Accessibility, DOM, and vision are complementary

Playwright ARIA snapshots serialize the accessibility tree into roles, names, attributes, and text. WAI-ARIA describes the accessibility tree as parallel to the DOM and defines exclusion/flattening rules. This makes AX compact and action-oriented but incomplete. Page authors control accessible names, including names not visibly rendered.

CDP DOMSnapshot can capture flattened DOM, shadow DOM, layout, and selected styles, but is Chromium-specific and experimental. Screenshots capture rendered pixels, canvas, overlays, and icons while losing stable identity and origin semantics. No one representation can safely drive every task.

Strong sources:

- [Playwright ARIA snapshots](https://playwright.dev/docs/aria-snapshots)
- [WAI-ARIA accessibility tree](https://www.w3.org/TR/wai-aria/#accessibility_tree)
- [WAI accessible names and descriptions](https://www.w3.org/WAI/ARIA/apg/practices/names-and-descriptions/)
- [CDP DOMSnapshot domain](https://chromedevtools.github.io/devtools-protocol/tot/DOMSnapshot/)
- [Playwright MCP introduction](https://playwright.dev/mcp/introduction)

Decision derived: order perception by cost and determinism—application/API state, semantic locators, AX, scoped DOM/layout, network events, then screenshot/vision. Preserve frame/origin/document provenance and re-observe before effects.

### 3. Browser lifecycle must be explicit

Playwright contexts create isolated, non-persistent browser sessions; pages and popups belong to contexts. Downloads begin with an event before completion, temporary files are removed when the context closes, and remote runs may not expose a local path. WebDriver BiDi models browsing contexts, navigation IDs, user contexts, storage partitions, and download begin/end events.

Strong sources:

- [Playwright browser contexts](https://playwright.dev/docs/api/class-browsercontext)
- [Playwright pages and popups](https://playwright.dev/docs/pages)
- [Playwright downloads](https://playwright.dev/docs/downloads)
- [Playwright Download API](https://playwright.dev/docs/api/class-download)
- [Playwright input and uploads](https://playwright.dev/docs/input)
- [WebDriver BiDi specification](https://w3c.github.io/webdriver-bidi/)
- [Selenium BiDi browsing contexts](https://www.selenium.dev/documentation/webdriver/bidi/w3c/browsing_context/)

Decision derived: normalize pages, frames, popups, navigation/redirect chains, downloads, file choosers, dialogs, and WebSockets into an application event model. Do not make the browser process the durable state store.

### 4. Auth state is a credential; shared profiles are high risk

Playwright warns that stored authentication state can contain sensitive cookies and headers capable of impersonating the user, and it should not be committed. Shared accounts are unsuitable for concurrent tests that mutate server-side state. MCP profile documentation distinguishes persistent and isolated modes; extension mode can reuse a person's existing logged-in tabs, cookies, and extensions.

Strong sources:

- [Playwright authentication](https://playwright.dev/docs/auth)
- [Playwright MCP user profile modes](https://playwright.dev/mcp/configuration/user-profile)
- [Playwright MCP browser extension mode](https://playwright.dev/mcp/configuration/browser-extension)
- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [WebAuthn Level 3](https://www.w3.org/TR/webauthn-3/)
- [Selenium virtual authenticator](https://www.selenium.dev/documentation/webdriver/interactions/virtual_authenticator/)

Decision derived: default to a fresh context with short-lived, account-scoped state in a disposable worker. Treat persistent profiles as exceptions and everyday browser attachment as visible interactive mode, not unattended shared infrastructure. Virtual authenticators are test tools, not production MFA bypasses.

### 5. Same-origin policy does not constrain the controller enough

The web same-origin policy limits how page scripts interact across origins, but permits or cannot prevent many navigations, redirects, and form actions. An external automation controller can read and operate across origins. Fetch Metadata and CSRF controls help target servers reason about request context, but they do not prove that an agent action matches user intent.

OWASP SSRF guidance emphasizes URL parser complexity, redirect disabling/validation, DNS pinning risks, and network-layer enforcement. Playwright MCP explicitly states that its allowed/blocked origin options are not a security boundary and do not cover redirects.

Strong sources:

- [MDN same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy)
- [W3C Fetch Metadata](https://www.w3.org/TR/fetch-metadata/)
- [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP SSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP Unvalidated Redirects and Forwards Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html)
- [Playwright MCP security notes](https://github.com/microsoft/playwright-mcp#security)

Decision derived: enforce canonical scheme/origin/port, redirect, DNS result, private/metadata range, WebSocket, and data-flow policy at an egress layer outside the browser. Reauthorize material origin/account/recipient/amount changes.

### 6. Prompt injection is an authority and containment problem

Current provider guidance converges on isolation, least privilege, domain/action allowlists, treating page/tool/file content as untrusted, and human approval for consequential actions. Provider research also states that prompt injection is not solved and adaptive attacks remain relevant.

The important architectural inference is that a detector or system prompt cannot unlock a dangerous tool. Direct authenticated user input and application policy grant authority; page observations supply untrusted facts only.

Strong sources:

- [OpenAI computer-use safety guidance](https://developers.openai.com/api/docs/guides/tools-computer-use)
- [Anthropic prompt-injection defenses](https://www.anthropic.com/research/prompt-injection-defenses)
- [Anthropic containment engineering](https://www.anthropic.com/engineering/how-we-contain-claude)
- [Anthropic trustworthy agents](https://www.anthropic.com/research/trustworthy-agents)
- [OWASP Excessive Agency](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/)
- [AgentDojo](https://agentdojo.spylab.ai/)
- [WASP](https://arxiv.org/abs/2504.18575)
- [DoomArena](https://openreview.net/forum?id=YeM5q99tBJ)
- [MUZZLE, USENIX Security 2026](https://www.usenix.org/conference/usenixsecurity26/presentation/syros)

Decision derived: keep model outputs proposal-only; make tools closed and capability-shaped; inject credentials outside model context; enforce destination/data/effect policy; use exact human approval for high impact; test adaptive attacks.

### 7. The browser sandbox does not contain the whole agent

Chromium's renderer sandbox and site isolation are meaningful defenses. Chromium's own agent security document notes the browser process is not sandboxed like renderers, and origin is a fundamental isolation unit. Playwright's Docker guide warns that its image is intended for testing/development, root disables Chromium's sandbox, and untrusted-site crawling should use a separate user plus seccomp.

Strong sources:

- [Chromium sandbox design](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/docs/design/sandbox.md)
- [Chromium process model and site isolation](https://chromium.googlesource.com/chromium/src/+/main/docs/process_model_and_site_isolation.md)
- [Chromium security architecture for agents](https://chromium.googlesource.com/chromium/src/+/main/docs/security/security-for-agents.md)
- [Playwright Docker guidance](https://playwright.dev/docs/docker)

Decision derived: contain the entire browser/controller worker with non-root OS identity, sandbox, seccomp, resource limits, private filesystem, egress enforcement, and disposable container/VM. Use stronger process or VM isolation for tenant and high-value boundaries.

### 8. Files cross both security and effect boundaries

OWASP recommends extension/type/size allowlists, generated names, storage outside web roots, authorization, and malware/content validation. Playwright's APIs show that download names are suggested by the page and files are tied to context lifetime; uploads can use local paths, buffers, directories, or file chooser events.

Strong sources:

- [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
- [Playwright downloads](https://playwright.dev/docs/downloads)
- [Playwright input and file uploads](https://playwright.dev/docs/input)

Decision derived: quarantine downloads under generated names and release only after type/size/scan policy. Give the planner opaque artifact handles, never arbitrary host paths. Treat upload as a destination- and purpose-bound data disclosure.

### 9. Idempotency is target semantics, not click retry or action cache

RFC 9110 defines idempotency for HTTP method semantics and warns against automatically retrying non-idempotent requests unless the client knows they are safe or can detect non-application. A mature API example uses a stable key and payload to return the same outcome. UI automation typically lacks that guarantee.

Strong sources:

- [RFC 9110 idempotent methods](https://www.rfc-editor.org/rfc/rfc9110.html#name-idempotent-methods)
- [Stripe idempotent requests](https://docs.stripe.com/api/idempotent_requests)
- [Stagehand caching guidance (current public `/v3/` documentation route, checked 2026-08-31)](https://docs.stagehand.dev/v3/best-practices/caching)

Decision derived: write effect intent and attempt before action; classify result as committed, proven absent, or unknown; reconcile unknown outcomes before retry. Use a documented target idempotency key where available. Treat framework caches as performance hints only.

### 10. Traces are evidence, not state checkpoints

Playwright tracing records browser operations, DOM snapshots, screenshots, and network activity. Library tracing does not automatically include all test assertions. Traces can contain highly sensitive authenticated data. W3C Trace Context and OpenTelemetry provide correlation conventions; OTel HTTP semantics model physical redirects/retries separately.

Strong sources:

- [Playwright tracing API](https://playwright.dev/docs/api/class-tracing)
- [Playwright Trace Viewer](https://playwright.dev/docs/trace-viewer)
- [OpenTelemetry HTTP spans](https://opentelemetry.io/docs/specs/semconv/http/http-spans/)
- [OpenTelemetry GenAI attributes](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/)
- [W3C Trace Context](https://www.w3.org/TR/trace-context/)

Decision derived: trace observation, proposal, policy, approval, effect attempt, verification, and reconciliation as separate correlated events. Use traces for evidence/decision replay; recover from durable business checkpoints and effect state, not recorded clicks.

### 11. Public benchmarks are inputs, not production gates

WebArena provides self-hosted realistic sites and functional correctness tasks; VisualWebArena adds visual grounding; WorkArena covers enterprise-style tasks; BrowserGym/AgentLab standardize experimental harnesses. AgentDojo, WASP, DoomArena, and related work add security/adversarial dimensions. WebArena Verified shows that task definitions and evaluators themselves require audit and versioning.

Strong sources:

- [WebArena repository](https://github.com/web-arena-x/webarena)
- [WebArena paper](https://arxiv.org/abs/2307.13854)
- [VisualWebArena paper](https://arxiv.org/abs/2401.13649)
- [BrowserGym repository](https://github.com/ServiceNow/BrowserGym)
- [BrowserGym ecosystem paper](https://arxiv.org/abs/2412.05467)
- [AgentLab repository](https://github.com/ServiceNow/AgentLab)
- [WorkArena repository](https://github.com/ServiceNow/WorkArena)
- [WorkArena++ paper](https://proceedings.neurips.cc/paper_files/paper/2024/file/0b82662b6c32e887bb252a74d8cb2d5e-Paper-Datasets_and_Benchmarks_Track.pdf)
- [WebArena Verified repository](https://github.com/ServiceNow/webarena-verified)
- [WebArena Verified paper](https://openreview.net/pdf/cf1fee593edb4cce39a6ec98239540f874635b6c.pdf)

Decision derived: gate releases on application-specific normal, perturbation, adversarial, and failure-injection corpora with independent outcome and forbidden-effect oracles. Pin task, environment, browser, seed, evaluator, model, and scaffold versions.

### 12. Protocol names do not imply equivalent capability

Playwright documents `connectOverCDP` as Chromium-only and significantly lower fidelity than its native protocol. Puppeteer supports CDP and WebDriver BiDi, but its current BiDi matrix still excludes accessibility, tracing, coverage, direct CDP sessions, several emulations, and parts of response handling. Selenium is migrating while maintaining Classic compatibility, and some binding APIs remain beta. CDP tip-of-tree can break without notice; WebDriver BiDi is still an Editor's Draft.

Strong sources:

- [Playwright `connectOverCDP`](https://playwright.dev/docs/api/class-browsertype#browser-type-connect-over-cdp)
- [Puppeteer WebDriver BiDi support](https://pptr.dev/webdriver-bidi)
- [Selenium WebDriver BiDi](https://www.selenium.dev/documentation/webdriver/bidi/)
- [Chrome DevTools Protocol version policy](https://chromedevtools.github.io/devtools-protocol/)
- [WebDriver BiDi Editor's Draft](https://w3c.github.io/webdriver-bidi/)

Decision derived: qualify the exact library/protocol/browser/provider tuple with lifecycle, perception, network, file, cancellation, reconnect, and redaction tests. Publish a signed capability manifest; an untested remote feature is unsupported even if it works locally.

### 13. Managed browsers move custody; they do not remove it

Browserbase documents broad persisted Context data, cloud download synchronization, default session recording, keep-alive reconnect, six-hour maximum sessions, regional selection, and concurrency/session-creation limits. Browserless documents persisted sessions and authenticated profiles, one attached client per persisted session, Puppeteer-only live-process reconnect in specific modes, Playwright reconstruction constraints, remote file-transfer limits, opt-in replay, and plan-specific TTL/concurrency.

Strong sources:

- [Browserbase contexts](https://docs.browserbase.com/platform/browser/core-features/contexts)
- [Browserbase keep alive](https://docs.browserbase.com/platform/browser/long-sessions/keep-alive)
- [Browserbase downloads](https://docs.browserbase.com/platform/browser/files/downloads)
- [Browserbase session recording](https://docs.browserbase.com/platform/browser/observability/session-recording)
- [Browserbase concurrency](https://docs.browserbase.com/optimizations/concurrency/overview)
- [Browserless persisted state](https://docs.browserless.io/baas/session-management/persisting-state)
- [Browserless authenticated profiles](https://docs.browserless.io/baas/features/authenticated-profiles)
- [Browserless file transfers](https://docs.browserless.io/baas/features/file-transfers)
- [Browserless best practices and limits](https://docs.browserless.io/baas/best-practices)

Decision derived: use providers for fleet/session infrastructure only after qualifying tenant boundary, profile semantics, egress, recordings/operator access, retention/region, bearer endpoints, file custody, cancellation, reconnect, rate limits, failure semantics, and deletion. Application-owned intent, approvals, ledger, reconciliation, and oracles remain outside.

### 14. CAPTCHA and stealth are capability, not authority

Browserbase and Browserless advertise CAPTCHA solving, proxies, stealth, and live human views. Cloudflare's verified-bot guidance emphasizes honest identity, non-abusive behavior, robots directives, and reasonable rates. Cloudflare also states that `robots.txt` is advisory, while Google provides reCAPTCHA test keys specifically for automated testing.

Strong sources:

- [Browserbase CAPTCHA solving](https://docs.browserbase.com/platform/identity/captcha-solving)
- [Browserless CAPTCHA solving](https://docs.browserless.io/baas/bot-detection/captchas)
- [Cloudflare verified bots](https://developers.cloudflare.com/bots/concepts/bot/verified-bots/)
- [Cloudflare robots guidance](https://developers.cloudflare.com/browser-run/reference/robots-txt/)
- [Google reCAPTCHA FAQ](https://developers.google.com/recaptcha/docs/faq)

Decision derived: disable bypass-oriented features by default. Record site-specific authorization, identity, purpose, rate, region, and expiry. Prefer APIs, partnerships, test keys/environments, signed agents, or explicit human presence. A block or challenge is never permission to rotate identity or evade controls.

### 15. Browser behavior must release as one governed bundle

Browser outcomes depend jointly on the site adapter and fingerprint, browser binary, automation library/protocol, provider, model/config, tool schema, prompt, policy, credential/artifact/egress services, and oracle. Independent updates prevent attribution and safe rollback. Public sites rarely expose a trustworthy build version, so several privacy-safe signals are needed for drift detection.

Decision derived: attach one behavior-bundle ID to each trace and receipt; qualify it by site/account/workflow/effect/locale/region/browser tuple; progress through fixture, shadow, read, draft, and commit canaries; roll back the full bundle. Mine controlled failures only after minimization, fixture reproduction, root-cause review, and regression creation. Raw page content or a successful model trajectory cannot self-promote into site knowledge.

## Framework assessment

### Playwright MCP

Evidence:

- maintained security notes explicitly say it is not a security boundary;
- allowed/blocked origin options default permissive and do not cover redirects;
- tool/capability modes include broad evaluation or code execution options;
- profiles can be persistent or isolated, and extension mode can reuse existing user sessions;
- current manifest is pre-1.0 and depends on a dated Playwright alpha.

Conclusion: excellent browser tooling for local agents, prototypes, or a constrained component. Do not use a shared default MCP process/profile as the tenant, network, credential, or effect boundary.

Sources:

- [Playwright MCP repository and security notes](https://github.com/microsoft/playwright-mcp)
- [Playwright MCP profile configuration](https://playwright.dev/mcp/configuration/user-profile)
- [Playwright MCP package manifest](https://github.com/microsoft/playwright-mcp/blob/main/package.json)

### Stagehand

Evidence:

- the stable JavaScript package moved to 4.0.2 on 2026-08-20 while 3.7 maintenance releases continued afterward;
- the current public caching guide still resolves under a `/v3/` route, so documentation route, installed SDK, embedded components, and managed service must not be assumed to share one version;
- the project exposes AI-assisted `act`, `observe`, `extract`, and browser integration concepts;
- the embedded extension/protocol and SDK lifecycle mean browser ownership, compatibility, and close behavior require exact-version testing;
- issue reports against 4.0.1 described Browserbase initialization and browser-handle lifecycle problems; these are test vectors, not universal current defects.

Conclusion: useful as a perception/planning and hybrid action component, but the recent major transition raises qualification cost. Pin the SDK and extension/protocol behavior, prove local and managed paths separately, bind discovered/cached actions to current page preconditions, and keep authorization/idempotency/effects outside it.

Sources:

- [Stagehand npm package](https://www.npmjs.com/package/@browserbasehq/stagehand)
- [Stagehand changelog](https://github.com/browserbase/stagehand/blob/main/CHANGELOG.md)
- [Stagehand releases](https://github.com/browserbase/stagehand/releases)
- [Stagehand v4 Browserbase initialization issue #2782](https://github.com/browserbase/stagehand/issues/2782)
- [Stagehand v4 browser ownership issue #2736](https://github.com/browserbase/stagehand/issues/2736)

### Browser Use

Evidence:

- rapid 0.13.x release stream, including a Rust rebuild marked beta;
- agent/browser abstractions cover sessions, actions, state/history, downloads, timeouts, and domain/secret mechanisms;
- release notes contain continuing fixes for stale DOM, cross-origin iframe extraction, shared state, remote downloads, occluded text, and domain-restricted action exposure.

Conclusion: useful Python framework and harness for prototypes or bounded internal workflows. Its rapid evolution reinforces exact version pinning and application-owned cross-tenant, egress, credential, and effect guarantees.

Sources:

- [Browser Use repository](https://github.com/browser-use/browser-use)
- [Browser Use releases](https://github.com/browser-use/browser-use/releases)
- [Browser Use browser reference](https://github.com/browser-use/browser-use/blob/main/skills/open-source/references/browser.md)

### Skyvern

Evidence:

- documents visual/LLM browser automation and a hybrid code-first pattern;
- recommends selectors where DOM is known and AI methods where it is not;
- provides self-hosted material.

Conclusion: the hybrid pattern aligns with deterministic-first architecture. Treat resilience claims as hypotheses to evaluate and keep application commits/policy outside the framework.

Sources:

- [Skyvern repository](https://github.com/Skyvern-AI/skyvern)
- [Skyvern browser automations](https://github.com/Skyvern-AI/skyvern/blob/main/docs/developers/browser-automations/overview.mdx)
- [Skyvern self-hosted overview](https://github.com/Skyvern-AI/skyvern/blob/main/docs/developers/self-hosted/overview.mdx)

### BrowserGym and AgentLab

Evidence:

- common browser observation/action environments and benchmark integration;
- repository explicitly positions BrowserGym as a research framework rather than a consumer product;
- AgentLab focuses reproducible agent experiments.

Conclusion: strong evaluation/research substrate; not a production credential/effect/tenant/operations plane.

Sources:

- [BrowserGym repository](https://github.com/ServiceNow/BrowserGym)
- [AgentLab repository](https://github.com/ServiceNow/AgentLab)

## Contradictions and tensions

These are not all errors; many are scope mismatches that architecture must make explicit.

| Claim or convenience | Counter-evidence / limitation | Blueprint resolution |
|---|---|---|
| “Allowed origins protect the browser agent.” | Playwright MCP says origin filters are not a security boundary and do not cover redirects. | Enforce all hops and resolved addresses at external egress; framework filters are defense-in-depth. |
| “Allowed domains plus sensitive data are sufficient.” | An open Browser Use issue reported a `data:`/`blob:` bypass in 0.12.6; later releases continued related domain/state fixes. Issue evidence is version-scoped, not proof of current exploitability. | Block special schemes by default and test the pinned version; keep secret injection and network policy outside the framework. |
| “Separate browser contexts are tenant isolation.” | Contexts isolate storage, but share the browser process and controller. A closed Playwright MCP 0.0.75 issue reported cross-client page-event/DOM bleed under a shared server. | One process/worker per sensitive trust boundary; test cross-client event isolation; never share authenticated context. |
| “Accessibility snapshots are safer than pixels.” | AX is compact and semantic, but page-controlled, incomplete, and can contain hidden/non-visible names or injected text. | Treat all modalities as untrusted; combine AX/DOM/visual evidence and preserve provenance. |
| “Vision solves changing layouts.” | Vision handles canvas/icons but adds cost, coordinate drift, incomplete off-screen state, and visual injection. | Use it as fallback; map back to fresh targets and forbid stale coordinate commits. |
| “Auto-wait makes clicks reliable.” | Actionability checks mechanics, not user intent or business commit. `force` can bypass checks. | Revalidate semantic target/effect and independently verify outcome; restrict force. |
| “Action caching/replay makes workflows reliable.” | Cache can point to stale or semantically changed controls and does not deduplicate target effects. | Page-fingerprint and precondition every cache hit; keep policy/approval/idempotency unchanged. |
| “A trace can resume a failed task.” | Traces are diagnostic snapshots, may omit assertions, contain secrets, and cannot prove target state after failure. | Recover from durable workflow/effect state; use evidence or decision replay only. |
| “The browser sandbox contains malicious pages.” | Renderer sandbox/site isolation do not contain the broad browser process or automation controller. Root containers can disable Chromium sandbox. | Isolate the whole worker with non-root sandbox, seccomp, egress, resource limits, and VM where needed. |
| “Prompt-injection training solves page attacks.” | Provider research explicitly frames the problem as unsolved and adaptive attacks remain effective in some conditions. | Prevent authority widening structurally and evaluate adaptive attacks; detection never grants capability. |
| “CSRF token means the action is authorized.” | CSRF defenses show request/session context, not the browser agent user's exact intent; client-side CSRF can abuse trusted JS. | Preserve site defenses, then add exact effect approval and receipt verification. |
| “Benchmark score proves production readiness.” | Benchmark task/evaluator defects and environment drift motivated WebArena Verified; benchmarks omit production accounts and policies. | Pin/evaluate benchmarks, but gate on application-specific outcomes, forbidden effects, attacks, and failure injection. |
| “Framework version is one number.” | Stagehand's JS and server components version separately; Playwright MCP can depend on alpha Playwright; Browser Use had a beta architectural rebuild inside 0.13. | Record every package/component/browser digest in run metadata and compatibility tests. |
| “Playwright behaves the same against a cloud CDP browser.” | Playwright explicitly labels `connectOverCDP` Chromium-only and significantly lower fidelity than its native protocol. | Qualify every remote/provider tuple and declare degraded/unsupported capabilities in a manifest. |
| “Keep-alive means durable recovery.” | Provider modes differ: a live process may reconnect, storage may restore into a blank process, or the session may expire; Playwright and Puppeteer detach semantics differ. | Durable truth remains in the controller; reconcile effects, increment session generation on reconstruction, and re-observe after any reconnect. |
| “Managed recordings are free observability.” | Browserbase records sessions by default unless disabled; recordings/logs/downloads add sensitive-data custody and plan/API constraints. | Set capture explicitly per risk, use ZDR/retention controls where required, redact at source, and keep provider evidence out of the recovery contract. |
| “CAPTCHA solving proves automation is allowed.” | Vendors expose solvers and stealth while site policies, verified-bot guidance, and challenge intent may prohibit or constrain automation. | Disable by default; require target/workflow permission and honest identity, or stop/handoff. |

### Issue evidence used with caution

- [Browser Use issue #4763](https://github.com/browser-use/browser-use/issues/4763): open report against version 0.12.6 alleging `data:`/`blob:` allowed-domain bypass. Used as a test vector and evidence that application policy cannot rely on a framework filter; not stated as a confirmed vulnerability in current 0.13.8.
- [Playwright MCP issue #1631](https://github.com/microsoft/playwright-mcp/issues/1631): closed report against 0.0.75 alleging cross-client page-event/DOM bleed. Used as version-scoped evidence for process/client isolation testing; not stated as current 0.0.79 behavior.

## Key architecture conclusions

1. A browser agent is a guarded effect system, not a generic browser tool.
2. Deterministic automation and direct APIs are preferred for known flows and sensitive commits.
3. The production default is a guarded hybrid with a proposal-only model and deterministic executor.
4. DOM, AX, network, and vision are complementary untrusted observations.
5. Candidate identity is document-scoped; every material action requires fresh state.
6. Auth storage is a credential and must be per-account, per-run, encrypted, and short-lived.
7. The application—not the framework—owns authority, egress, credentials, approvals, effects, and verification.
8. The entire worker requires containment; browser context and renderer sandbox are narrower controls.
9. High-impact approval must bind exact effect data and be single-use/fresh.
10. Timeouts after effects create unknown outcomes; reconciliation precedes retry.
11. Downloads are quarantined artifacts; uploads are governed disclosures.
12. Traces and action histories are evidence, not durable checkpoints or idempotency.
13. Evaluation needs independent outcome, safety, trajectory, reliability, and cost dimensions.
14. Public benchmarks inform capability; production task/attack/failure corpora decide release.
15. Rapid framework and browser churn require exact pins, compatibility tests, and refresh triggers.
16. Protocol and provider features are admitted through an exact capability manifest, not product-name assumptions.
17. Human takeover requires an exclusive controller lease, focus invalidation, and post-takeover reconciliation.
18. Exactly seven memory lifetimes prevent browser credentials, raw evidence, and injected page content from becoming informal durable memory.
19. Site behavior releases as a versioned bundle through shadow and effect-scoped canaries with full-bundle rollback.

## Limitations and open questions

### Target-specific unknowns

- Exact idempotency and commit probes depend on each target site's API and business semantics.
- Authentication, consent, anti-bot, terms, and payment rules must be reviewed per site and jurisdiction.
- Accessibility and DOM quality vary widely; no universal candidate representation was assumed.
- Browser resource budgets require measurement on the actual pages, files, models, and deployment hosts.

### Technology uncertainty

- WebDriver BiDi remains a live draft and cross-browser feature parity changes.
- Playwright MCP is pre-1.0 and follows alpha Playwright internals in the observed manifest.
- Browser Use's 0.13 Rust rebuild is labeled beta, so behavior and API stability require local validation.
- Stagehand's 4.x SDK, embedded extension/protocol, 3.7 maintenance stream, and provider behavior can move independently.
- Browserbase and Browserless plans, regions, retention, concurrency, session, profile, live-view, and file semantics can change independently of client libraries.
- Managed-provider marketing claims about stealth, isolation, or resilience were not treated as proof; target-specific tests and contracts remain required.
- Model behavior can change under pinned aliases or provider-side updates; use immutable identifiers where available and canary tests.
- Prompt injection has no complete general defense; residual risk remains even with the recommended controls.

### Evaluation uncertainty

- Public benchmark environments may not match real anti-automation, authentication, localization, or accessibility behavior.
- Attack benchmarks can underrepresent adaptive, cross-modal, and target-specific injection.
- Model-based verifiers can share the agent's blind spots; deterministic business oracles remain preferred.
- Zero observed catastrophic failures does not prove zero risk; task counts and confidence must be reported.

## Refresh triggers

Refresh this packet when:

- Playwright stable changes beyond 1.62.x or relevant context/navigation/download/tracing behavior changes;
- Playwright MCP reaches a new security model, major API, or 1.0 milestone;
- Browser Use moves beyond the 0.13 beta rebuild or resolves/changes domain/session boundaries;
- Stagehand changes major version, cache semantics, browser integration, or agent interfaces;
- Puppeteer/Selenium/WebDriver BiDi changes required protocol behavior;
- Chromium changes sandbox, site isolation, headless, extension, or process architecture;
- a browser/framework/model prompt-injection, isolation, origin, code-execution, or credential advisory appears;
- OpenAI or another chosen provider changes computer-use tools or safety guidance;
- WebAuthn, Fetch Metadata, CSRF, payment, or relevant web standards change materially;
- new benchmark audits or evaluator failures affect cited results;
- production incidents reveal a missing threat, retry class, commit probe, or evaluation case;
- a new site, credential type, high-impact action, file type, or tenant isolation class is added.

## Consolidated primary source register

### Browser automation and protocols

- [Playwright main documentation](https://playwright.dev/)
- [Playwright 1.62.1 release](https://github.com/microsoft/playwright/releases/tag/v1.62.1)
- [Playwright supported languages](https://playwright.dev/docs/languages)
- [Playwright locators](https://playwright.dev/docs/locators)
- [Playwright actionability](https://playwright.dev/docs/actionability)
- [Playwright ARIA snapshots](https://playwright.dev/docs/aria-snapshots)
- [Playwright browser contexts](https://playwright.dev/docs/api/class-browsercontext)
- [Playwright pages and popups](https://playwright.dev/docs/pages)
- [Playwright authentication](https://playwright.dev/docs/auth)
- [Playwright downloads](https://playwright.dev/docs/downloads)
- [Playwright upload/input](https://playwright.dev/docs/input)
- [Playwright tracing](https://playwright.dev/docs/api/class-tracing)
- [Playwright Trace Viewer](https://playwright.dev/docs/trace-viewer)
- [Playwright Docker](https://playwright.dev/docs/docker)
- [Playwright parallelism](https://playwright.dev/docs/test-parallel)
- [Playwright `connectOverCDP`](https://playwright.dev/docs/api/class-browsertype#browser-type-connect-over-cdp)
- [Playwright network interception](https://playwright.dev/docs/network)
- [Playwright service-worker limitations](https://playwright.dev/docs/service-workers)
- [Playwright MCP repository](https://github.com/microsoft/playwright-mcp)
- [Playwright MCP profile modes](https://playwright.dev/mcp/configuration/user-profile)
- [Playwright MCP extension mode](https://playwright.dev/mcp/configuration/browser-extension)
- [CDP protocol](https://chromedevtools.github.io/devtools-protocol/)
- [CDP DOMSnapshot](https://chromedevtools.github.io/devtools-protocol/tot/DOMSnapshot/)
- [CDP Browser domain](https://chromedevtools.github.io/devtools-protocol/tot/Browser/)
- [Puppeteer 25.9.0 release](https://github.com/puppeteer/puppeteer/releases/tag/puppeteer-v25.9.0)
- [Puppeteer WebDriver BiDi support](https://pptr.dev/webdriver-bidi)
- [Puppeteer interception semantics](https://pptr.dev/guides/network-interception)
- [Selenium WebDriver BiDi](https://www.selenium.dev/documentation/webdriver/bidi/)
- [Selenium 4.48.0 release](https://github.com/SeleniumHQ/selenium/releases/tag/selenium-4.48.0)
- [WebDriver BiDi specification](https://w3c.github.io/webdriver-bidi/)
- [Browserbase contexts](https://docs.browserbase.com/platform/browser/core-features/contexts)
- [Browserbase keep alive](https://docs.browserbase.com/platform/browser/long-sessions/keep-alive)
- [Browserbase timeouts](https://docs.browserbase.com/platform/browser/long-sessions/timeouts)
- [Browserbase downloads](https://docs.browserbase.com/platform/browser/files/downloads)
- [Browserbase uploads](https://docs.browserbase.com/platform/browser/files/uploads)
- [Browserbase recordings](https://docs.browserbase.com/platform/browser/observability/session-recording)
- [Browserbase concurrency](https://docs.browserbase.com/optimizations/concurrency/overview)
- [Browserless 2.56.0 release](https://github.com/browserless/browserless/releases/tag/v2.56.0)
- [Browserless persisted sessions](https://docs.browserless.io/baas/session-management/persisting-state)
- [Browserless authenticated profiles](https://docs.browserless.io/baas/features/authenticated-profiles)
- [Browserless standard reconnect sessions](https://docs.browserless.io/baas/session-management/standard-sessions)
- [Browserless file transfers](https://docs.browserless.io/baas/features/file-transfers)
- [Browserless live human control](https://docs.browserless.io/baas/monitor-sessions/hybrid-automation)

### Web and browser security

- [MDN same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy)
- [W3C Fetch Metadata](https://www.w3.org/TR/fetch-metadata/)
- [WAI-ARIA 1.2](https://www.w3.org/TR/wai-aria/)
- [W3C WebAuthn Level 3](https://www.w3.org/TR/webauthn-3/)
- [Chromium sandbox](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/docs/design/sandbox.md)
- [Chromium process model and site isolation](https://chromium.googlesource.com/chromium/src/+/main/docs/process_model_and_site_isolation.md)
- [Chromium security for agents](https://chromium.googlesource.com/chromium/src/+/main/docs/security/security-for-agents.md)
- [OWASP CSRF Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP SSRF Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP File Upload](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
- [OWASP Secrets Management](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [OWASP Excessive Agency](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/)
- [Cloudflare verified bots](https://developers.cloudflare.com/bots/concepts/bot/verified-bots/)
- [Cloudflare robots guidance](https://developers.cloudflare.com/browser-run/reference/robots-txt/)
- [Google reCAPTCHA testing FAQ](https://developers.google.com/recaptcha/docs/faq)

### Model/computer-use security

- [OpenAI computer-use guide](https://developers.openai.com/api/docs/guides/tools-computer-use)
- [Anthropic prompt-injection defenses](https://www.anthropic.com/research/prompt-injection-defenses)
- [Anthropic containment engineering](https://www.anthropic.com/engineering/how-we-contain-claude)
- [Anthropic trustworthy agents](https://www.anthropic.com/research/trustworthy-agents)
- [AgentDojo paper](https://proceedings.neurips.cc/paper_files/paper/2024/file/97091a5177d8dc64b1da8bf3e1f6fb54-Paper-Datasets_and_Benchmarks_Track.pdf)
- [WASP](https://arxiv.org/abs/2504.18575)
- [DoomArena](https://openreview.net/forum?id=YeM5q99tBJ)
- [MUZZLE](https://www.usenix.org/conference/usenixsecurity26/presentation/syros)

### Reliability and telemetry

- [RFC 9110 idempotent methods](https://www.rfc-editor.org/rfc/rfc9110.html#name-idempotent-methods)
- [Stripe idempotent requests](https://docs.stripe.com/api/idempotent_requests)
- [OpenTelemetry HTTP spans](https://opentelemetry.io/docs/specs/semconv/http/http-spans/)
- [OpenTelemetry GenAI attributes](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/)
- [W3C Trace Context](https://www.w3.org/TR/trace-context/)

### Frameworks and alternatives

- [Stagehand repository](https://github.com/browserbase/stagehand)
- [Stagehand npm package](https://www.npmjs.com/package/@browserbasehq/stagehand)
- [Browser Use repository](https://github.com/browser-use/browser-use)
- [Browser Use releases](https://github.com/browser-use/browser-use/releases)
- [Skyvern repository](https://github.com/Skyvern-AI/skyvern)
- [Skyvern hybrid automation](https://github.com/Skyvern-AI/skyvern/blob/main/docs/developers/browser-automations/overview.mdx)
- [BrowserGym repository](https://github.com/ServiceNow/BrowserGym)
- [AgentLab repository](https://github.com/ServiceNow/AgentLab)

### Evaluation

- [WebArena repository](https://github.com/web-arena-x/webarena)
- [WebArena paper](https://arxiv.org/abs/2307.13854)
- [VisualWebArena paper](https://arxiv.org/abs/2401.13649)
- [BrowserGym ecosystem paper](https://arxiv.org/abs/2412.05467)
- [WorkArena repository](https://github.com/ServiceNow/WorkArena)
- [WorkArena++ paper](https://proceedings.neurips.cc/paper_files/paper/2024/file/0b82662b6c32e887bb252a74d8cb2d5e-Paper-Datasets_and_Benchmarks_Track.pdf)
- [WebArena Verified repository](https://github.com/ServiceNow/webarena-verified)
- [WebArena Verified paper](https://openreview.net/pdf/cf1fee593edb4cce39a6ec98239540f874635b6c.pdf)

## Packet quality checklist

- [x] Current version/date baselines recorded.
- [x] Official documentation and specifications used as foundation.
- [x] Framework release and repository evidence reviewed.
- [x] Repository issues labeled as version-scoped reports, not current universal facts.
- [x] Browser, model, network, credential, effect, file, and tenant boundaries covered.
- [x] Playwright/Puppeteer/Selenium, CDP/BiDi, and managed-provider capabilities are version-pinned and limitation-scoped.
- [x] Deterministic, model-assisted, visual, and guarded hybrid designs compared.
- [x] Reliability distinguishes mechanics, commit semantics, and unknown outcomes.
- [x] Exactly seven information lifetimes and loss-aware compaction are specified.
- [x] Evaluation covers outcomes, forbidden effects, trajectory, failure injection, cost, and benchmark limits.
- [x] Site/version qualification, behavior bundles, shadow/canary promotion, rollback, and failure mining are specified.
- [x] Contradictions, limitations, and refresh triggers recorded.
- [x] Blueprint guides link back to this packet and primary references.

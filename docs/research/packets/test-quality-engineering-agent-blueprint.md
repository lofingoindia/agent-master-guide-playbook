# Test and Quality Engineering Agent Blueprint: Research Packet

> Status: Dated evidence packet for a production blueprint  
> Research date: 2026-08-31  
> Scope: independent software-quality verification, test design and execution, defect reproduction, flaky-test diagnosis, coverage and risk analysis, and release-quality recommendations  
> Excluded ownership: feature implementation, merge authority, deployment execution, and production promotion

## Why this packet exists

This packet records the evidence behind the companion [Test and Quality Engineering Agent blueprint](../../agents/test-quality-engineering-agent/README.md). It is deliberately separate from the guides so that future maintainers can distinguish sourced facts from design choices and can refresh version-sensitive claims without rewriting the architecture from memory.

The central research question was:

> What is the smallest agent architecture that can produce trustworthy, reproducible, independent quality evidence without becoming a second coding agent or an unauthorized deployment controller?

The answer is not “put an LLM in front of every test tool.” Deterministic build and test automation remains the default. An agent is justified where the work requires adaptive test design, cross-artifact evidence synthesis, controlled environment and fixture construction, reproduction or minimization, flaky-test investigation, or an uncertainty-aware recommendation. Deterministic runners, schemas, policy, and oracles retain authority over execution and pass/fail facts.

## Research method

Research used current primary sources as the baseline: official specifications, official product documentation, standards bodies, project documentation, repositories, and original research papers. Secondary claims were not used as architectural foundations. Important claims were cross-checked across domains because a rule that is safe for a unit-test runner may be unsafe for browser automation, active security scanning, or load generation.

The review covered:

- agent runtime behavior, evaluation, compaction, tool boundaries, and approvals;
- hermetic test execution, fixtures, retries, sharding, and environment isolation;
- browser, mobile, API/contract, performance, security, accessibility, property-based, mutation, fuzz, and combinatorial testing;
- regression-test selection, failure reproduction, minimization, and flake evidence;
- CI trust boundaries, external test-management systems, rate limits, artifacts, and attestations;
- identity, secrets, prompt injection, memory poisoning, dependency and plugin supply chain;
- tracing, metrics, SLOs, rollout, incident response, capacity, and failure injection.

Search continued across independent domains until new sources mainly repeated the same operational constraints. The blueprint avoids vendor-specific guarantees where the evidence only supports a local implementation choice.

## Category boundary established by the evidence

| Category | Owns | May produce | Must not own by default |
| --- | --- | --- | --- |
| Coding agent | implementation of an approved change | source patch, tests that accompany the patch, build evidence | independent acceptance of its own change |
| Test and quality engineering agent | independent verification strategy and evidence | ephemeral tests and fixtures, reproduction bundle, quality findings, advisory release recommendation | feature implementation, merge, deployment, promotion |
| DevOps/deployment agent | approved delivery and promotion workflow | deployment plan, rollout evidence, rollback action | redefining product-quality acceptance after the fact |

Independence is a control, not a claim that test code can never change. The quality agent can generate test programs, fixtures, or probes inside an isolated campaign workspace. A permanent repository change is a proposed artifact that must enter the coding/review path; it does not silently expand the quality agent’s authority.

## Evidence-backed findings

### 1. Deterministic automation is the baseline, not a primitive form of the agent

Test runners already provide repeatable discovery, filtering, execution, reporting, retries, fixtures, and sharding. The agent should not reinterpret capabilities that a deterministic tool exposes accurately. It earns its cost when it selects among tools, creates a bounded plan, adapts to evidence, or communicates uncertainty.

Supporting sources:

- [Bazel Test Encyclopedia](https://bazel.build/reference/test-encyclopedia) defines hermetic test expectations, declared inputs, isolated temporary space, exit behavior, and sharding contracts.
- [Playwright test command line](https://playwright.dev/docs/test-cli) exposes deterministic selection, retry, trace, update, and sharding controls.
- [AndroidJUnitRunner](https://developer.android.com/training/testing/instrumented-tests/androidx-test-libraries/runner) exposes filtering, sharding, and orchestrated isolation.

Blueprint consequence: phase 0 is a deterministic quality platform. The first agent loop is added only after typed runner and artifact contracts exist.

### 2. Isolation is a quality property and a security boundary

Playwright, Selenium, Android Test Orchestrator, Bazel, and Testcontainers all converge on the value of fresh or controlled state. Shared processes, user profiles, databases, and mutable caches cause order dependence and make evidence hard to reproduce. Isolation also contains untrusted build scripts and generated tests.

Supporting sources:

- [Playwright isolation guidance](https://playwright.dev/docs/best-practices) recommends independent tests and fresh state.
- [Selenium: avoid sharing state](https://www.selenium.dev/documentation/test_practices/encouraged/avoid_sharing_state/) recommends a fresh driver and no shared test data.
- [Android Test Orchestrator](https://developer.android.com/training/testing/instrumented-tests/androidx-test-libraries/runner) isolates tests in separate instrumentation instances, with optional package-data clearing, at a performance cost.
- [Testcontainers getting started](https://testcontainers.com/getting-started/) describes on-demand real dependencies in controlled containers.
- [Testcontainers reusable containers](https://java.testcontainers.org/features/reuse/) labels reuse experimental and not suited to CI.

Blueprint consequence: each campaign receives a base revision, writable overlay, environment lease, fixture namespace, secret scope, and artifact namespace. Reuse is an explicit, lower-trust optimization.

### 3. Retry is evidence of intermittency, not permission to erase a failure

Playwright explicitly classifies a test as flaky when it fails first and passes on retry; it also replaces a failed worker. Pytest documents uncontrolled state, order dependence, time, concurrency, and external resources as common flake causes. A green final retry is therefore not equivalent to a clean first attempt.

Supporting sources:

- [Playwright retries](https://playwright.dev/docs/test-retries)
- [pytest flaky tests](https://docs.pytest.org/en/stable/explanation/flaky.html)

Blueprint consequence: every attempt is retained. The release recommendation reports first-attempt failures, retry outcomes, confidence, and quarantine status separately.

### 4. Parallelism changes the experiment

Parallel workers can expose races and reduce latency, but also introduce resource contention and order effects. Playwright recommends one worker in CI for maximum reproducibility and sharding for broader parallelism; Android managed devices and CI matrices expose explicit sharding and concurrency controls.

Supporting sources:

- [Playwright continuous integration](https://playwright.dev/docs/ci)
- [Android Gradle managed devices](https://developer.android.com/studio/test/managed-devices)
- [GitHub Actions matrix controls](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/run-job-variations)

Blueprint consequence: concurrency is a budgeted plan parameter. Reproduction starts serial and widens only to test concurrency hypotheses. Resource locks protect non-shareable fixtures and environments.

### 5. Test selection must have a conservative fallback

Dependency-graph selection is useful when the graph and change base are accurate. Nx marks affected projects using changed files and the project graph and conservatively treats lockfile changes by default. Large-scale research from Google and Microsoft shows value from selection and prioritization, but also makes clear that prediction errors and flaky tests complicate safety.

Supporting sources:

- [Nx affected commands](https://nx.dev/docs/features/ci-features/affected)
- [Assessing transition-based test selection algorithms at Google](https://research.google/pubs/assessing-transition-based-test-selection-algorithms-at-google/)
- [Regression testing in CI environments](https://research.google/pubs/techniques-for-improving-regression-testing-in-continuous-integration-development-environments/)
- [Data-driven test selection at scale](https://www.microsoft.com/en-us/research/publication/data-driven-test-selection-at-scale/)

Blueprint consequence: selection returns both selected and omitted scope plus a reason. Unknown mappings, high-risk changes, graph drift, runner upgrades, or release-candidate gates widen to the required baseline or full suite.

### 6. Structural coverage is not evidence that assertions are effective

Branch coverage measures which control-flow alternatives executed. Mutation testing tests whether the suite detects deliberately introduced changes and can reveal weak assertions, while equivalent mutants limit interpretation. Neither metric directly proves requirements coverage or production correctness.

Supporting sources:

- [coverage.py branch coverage](https://coverage.readthedocs.io/en/latest/branch.html)
- [PIT mutation-testing concepts](https://pitest.org/quickstart/basic_concepts/)
- [PIT mutation testing](https://pitest.org/)

Blueprint consequence: release evidence separates structural, mutation, requirements, risk, platform, and operational coverage. No blended “quality score” hides a missing dimension.

### 7. Generative testing needs reproducible seeds and promoted examples

Property-based and stateful testing can explore sequences humans did not enumerate and shrink a failure to a smaller reproducer. Hypothesis warns that its example database is a cache, not a correctness mechanism, and that opaque reproduction blobs are version-sensitive. Stable regressions should become explicit examples.

Supporting sources:

- [Hypothesis stateful testing](https://hypothesis.readthedocs.io/en/latest/stateful.html)
- [Hypothesis replaying failures](https://hypothesis.readthedocs.io/en/latest/tutorial/replaying-failures.html)
- [Hypothesis flaky failures](https://hypothesis.readthedocs.io/en/latest/tutorial/flaky.html)

Blueprint consequence: capture generator version, seed, minimized input, model state, and environment digest. Promote accepted minimal regressions into durable tests through the normal coding path.

### 8. Combinatorial testing is useful when interactions dominate, but constraints matter

NIST’s ACTS work supports covering t-way parameter interactions without enumerating the full Cartesian product. Invalid combinations must be modeled as constraints, and interaction strength is a risk choice rather than a universal constant.

Supporting sources:

- [NIST Automated Combinatorial Testing for Software](https://csrc.nist.gov/Projects/Automated-Combinatorial-Testing-for-Software/)
- [NIST SP 800-142](https://csrc.nist.gov/pubs/sp/800/142/final)

Blueprint consequence: generated matrices preserve the parameter model, constraints, interaction strength, and uncovered combinations as evidence.

### 9. Fuzzing requires a corpus and crash-artifact lifecycle

libFuzzer is coverage-guided and in-process. It relies on seed corpora, emits artifacts for failing inputs, supports minimization and parallel workers, and is coupled to compatible compiler tooling. Global state and nondeterminism can undermine reproducibility.

Supporting source: [LLVM libFuzzer documentation](https://llvm.org/docs/LibFuzzer.html)

Blueprint consequence: fuzz campaigns are separately budgeted jobs. Corpus version, sanitizer/compiler version, seed, dictionary, crash input, minimization log, and symbolized trace are first-class artifacts.

### 10. Differential and metamorphic oracles are valuable when exact expected output is unavailable

Csmith’s compiler-testing work demonstrates differential testing: generate valid programs and compare outputs across independently implemented compilers. This is powerful only if divergence is interpreted carefully; shared defects, undefined behavior, and correlated implementations can invalidate the oracle.

Supporting sources:

- [Csmith repository](https://github.com/csmith-project/csmith)
- [Finding and Understanding Bugs in C Compilers](https://web.stanford.edu/class/cs343/resources/finding-bugs-compilers.pdf)

Blueprint consequence: an oracle record names the compared implementations or metamorphic relation, allowed tolerance, known common mode, and adjudication rule.

### 11. Reproduction should include minimization and change-point search

Delta debugging and `git bisect` reduce the input or revision search space. `git bisect run` also distinguishes untestable revisions, and its script must remain stable across checked-out history.

Supporting sources:

- [Simplifying and Isolating Failure-Inducing Input](https://arxiv.org/abs/cs/0012009)
- [git bisect](https://git-scm.com/docs/git-bisect)

Blueprint consequence: the defect bundle distinguishes observed failure, minimal reproducer, suspected first bad revision, and confirmed root cause. The agent must not present correlation from a bisect as causal proof without validation.

### 12. Browser functional automation is not a load-test engine

Selenium explicitly discourages WebDriver for performance testing because browser and network variability make results difficult to attribute. k6 recommends protocol-level virtual users for most load and a small browser population when browser metrics are specifically required.

Supporting sources:

- [Selenium discouraged performance testing](https://www.selenium.dev/documentation/test_practices/discouraged/performance_testing/)
- [k6 load testing websites](https://grafana.com/docs/k6/latest/testing-guides/load-testing-websites/)

Blueprint consequence: browser functional validation and performance/load validation have separate adapters, budgets, targets, metrics, and authorization.

### 13. Performance thresholds are executable policy, but one threshold is not an investigation

k6 thresholds can fail a run with a nonzero exit and are suitable for bounded checks. Its automated-testing guidance cautions that a single pass/fail value can provide false confidence for larger tests.

Supporting sources:

- [k6 thresholds](https://grafana.com/docs/k6/latest/using-k6/thresholds/)
- [k6 automated performance testing](https://grafana.com/docs/k6/latest/testing-guides/automated-performance-testing/)

Blueprint consequence: store workload model, arrival pattern, target revision, environment capacity, warm-up, thresholds, raw time series, confidence caveats, and comparison baseline. Production load requires explicit authorization and safeguards.

### 14. Contract tests verify used interactions, not an entire API specification

Pact describes consumer-driven contracts built from concrete interactions and provider-state verification. Provider verification should happen against the real provider behavior with controlled downstream dependencies. Specification versions and matching rules affect compatibility.

Supporting sources:

- [How Pact works](https://docs.pact.io/getting_started/how_pact_works)
- [Pact provider verification](https://docs.pact.io/provider)
- [Pact specification](https://docs.pact.io/implementation_guides/pact_specification)

Blueprint consequence: distinguish schema validation, example-based consumer contracts, behavioral API tests, compatibility checks, and end-to-end validation. A passing contract suite is not “full API coverage.”

### 15. Mobile isolation and realism are a ladder, not a binary choice

Android managed devices offer repeatable build-managed virtual devices and snapshots; Android Test Orchestrator offers stronger per-test isolation at a performance cost. Swift Testing can run tests in parallel in the same process, while UI and device behavior may require XCTest and physical-device coverage. Appium’s drivers and plugins are independently installed extensions.

Supporting sources:

- [Android Gradle managed devices](https://developer.android.com/studio/test/managed-devices)
- [Android test runner and orchestrator](https://developer.android.com/training/testing/instrumented-tests/androidx-test-libraries/runner)
- [Swift Testing](https://developer.apple.com/documentation/testing)
- [XCTest](https://developer.apple.com/documentation/xctest)
- [Appium command-line reference](https://appium.io/docs/en/latest/reference/cli/)

Blueprint consequence: record simulator/emulator or device identity, OS image, locale, accessibility settings, app/build digest, driver/plugin versions, permissions, network conditions, and reset policy. Physical-device runs are targeted evidence, not an excuse for an unbounded matrix.

### 16. Automated accessibility checks cannot establish full conformance

WCAG 2.2 defines normative conformance requirements for full pages. ACT rules can be automated, semi-automated, or manual, and published rules are informative implementations rather than the normative basis of conformance.

Supporting sources:

- [WCAG 2.2](https://www.w3.org/TR/wcag/)
- [ACT Rules Format](https://www.w3.org/WAI/standards-guidelines/act/)
- [ACT rules](https://www.w3.org/WAI/standards-guidelines/act/rules/)
- [Understanding ACT rules](https://www.w3.org/WAI/WCAG22/Understanding/understanding-act-rules)

Blueprint consequence: report rule coverage and findings precisely. Reserve conformance language for the required human, assistive-technology, and full-page review process.

### 17. Active security testing needs a separately authorized target boundary

ZAP’s Automation Framework uses explicit environments, jobs, tests, and exit behavior; job order can affect results. NIST SSDF recommends risk-appropriate security testing, including dynamic analysis, fuzzing, regression tests for past vulnerabilities, and penetration testing for high-risk systems.

Supporting sources:

- [ZAP Automation Framework](https://www.zaproxy.org/docs/automate/automation-framework/)
- [ZAP Automation Framework details](https://www.zaproxy.org/docs/desktop/addons/automation-framework/)
- [NIST Secure Software Development Framework, SP 800-218](https://csrc.nist.gov/pubs/sp/800/218/final)

Blueprint consequence: active scan, fuzz, load, and destructive security tools require a target allowlist, method policy, rate/concurrency budget, stop condition, data policy, and accountable approval. The quality agent does not autonomously exploit findings.

### 18. Rolling testing standards require versioned evidence

OWASP’s latest Web Security Testing Guide is a moving target and explicitly distinguishes stable releases from the latest content. A test case that only says “OWASP latest” cannot be reproduced later.

Supporting source: [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/latest/)

Blueprint consequence: persist standard and rule identifiers with exact versions or commit/release references. Re-run or re-baseline when the standard changes.

### 19. CI metadata is not necessarily a gate

GitLab’s JUnit report integration displays results but does not change job status; the test command must exit nonzero. Parsers also have format and uniqueness constraints. Treating publication success as test success is a category error.

Supporting sources:

- [GitLab unit-test reports](https://docs.gitlab.com/ci/testing/unit_test_reports/)
- [GitLab job-artifacts API](https://docs.gitlab.com/api/job_artifacts/)

Blueprint consequence: keep runner outcome, report-parse outcome, artifact-publication outcome, and external-test-management synchronization as separate states.

### 20. CI workflows execute attacker-controlled material

GitHub warns that `pull_request_target` can expose a write-capable token and secrets and that checking out and running untrusted pull-request code creates a “pwn request” risk. Test configuration, scripts, dependency manifests, logs, artifacts, and even test names are untrusted input.

Supporting sources:

- [GitHub secure use of `pull_request_target`](https://docs.github.com/en/actions/reference/security/securely-using-pull_request_target)
- [GitHub Actions secure use reference](https://docs.github.com/en/actions/reference/security/secure-use)

Blueprint consequence: untrusted-change validation uses ephemeral isolated runners, read-only repository credentials, no production secrets, constrained egress, and a separate trusted publication step. Artifact content is parsed as data, never as instructions to the model.

### 21. Artifact attestation proves provenance, not safety

GitHub states that an attestation only becomes useful when verified and that provenance does not prove an artifact is secure. Attesting every transient log may add cost without a trust decision that consumes it.

Supporting sources:

- [GitHub artifact attestations](https://docs.github.com/en/actions/concepts/security/artifact-attestations)
- [Using artifact attestations](https://docs.github.com/en/actions/how-tos/secure-your-work/use-artifact-attestations/use-artifact-attestations)

Blueprint consequence: sign or attest release-relevant evidence and reusable test assets when a consumer verifies them. Retain digests and lineage for all material artifacts; do not call a signed result “correct.”

### 22. External test-management and issue integrations need idempotency and backpressure

TestRail documents cloud rate limits, `429` responses, `Retry-After`, and bulk result endpoints. Jira Cloud likewise documents rate-limit classes and backoff. A retrying agent that creates a new issue or result on each attempt causes duplication and can amplify an outage.

Supporting sources:

- [TestRail API introduction](https://support.testrail.com/hc/en-us/articles/7077083596436-Introduction-to-the-TestRail-API)
- [TestRail runs API](https://support.testrail.com/hc/en-us/articles/7077874763156-Runs)
- [Jira Cloud rate limiting](https://developer.atlassian.com/cloud/jira/platform/rate-limiting/)
- [Jira Cloud issues API](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/)
- [GitHub check runs API](https://docs.github.com/en/rest/checks/runs)

Blueprint consequence: adapters use deterministic idempotency keys, receipts, bounded retries with server hints, dead-letter handling, and reconciliation. External publication failure does not change the underlying test result.

### 23. Agent memory creates a persistence attack surface

OWASP’s agentic guidance identifies goal hijacking, tool misuse, identity and privilege abuse, supply-chain vulnerabilities, memory/context poisoning, insecure communication, cascading failures, and human trust exploitation. Its memory guidance emphasizes that untrusted content can survive beyond the interaction in which it arrived.

Supporting sources:

- [OWASP Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)
- [OWASP agentic risk overview](https://genai.owasp.org/2025/12/09/owasp-top-10-for-agentic-applications-the-benchmark-for-agentic-security-in-the-age-of-autonomous-ai/)
- [OWASP: memory is an attack surface](https://genai.owasp.org/2026/05/13/memory-is-a-feature-it-is-also-an-attack-surface/)

Blueprint consequence: long-term memory is off by default. Admitted records are typed, provenance-linked, scoped, reviewable, expiring, and revocable. Raw logs, pages, issue text, repository instructions, and generated test output never become trusted memory automatically.

### 24. Compaction supports continuation but is not the durable audit record

Current OpenAI guidance describes compaction as opaque, encrypted continuation state and recommends preserving completed actions, active assumptions, IDs, tool outcomes, unresolved blockers, and the next concrete goal. Opaque state cannot replace an inspectable campaign ledger.

Supporting sources:

- [OpenAI model guidance](https://developers.openai.com/api/docs/guides/latest-model)
- [OpenAI compact-response API](https://developers.openai.com/api/reference/java/resources/responses/methods/compact)

Blueprint consequence: compaction may reduce active context, but authoritative test attempts, decisions, leases, artifacts, and evidence references remain in durable typed storage.

### 25. The agent itself needs trajectory and outcome evaluation

OpenAI’s current agent-evaluation guidance combines traces, graders, datasets, and repeatable eval runs, and recommends trace grading to locate tool, handoff, and instruction regressions. An end answer alone cannot reveal whether a recommendation was reached through unsafe tools, omitted tests, or fabricated evidence.

Supporting source: [OpenAI agent evals](https://developers.openai.com/api/docs/guides/agent-evals)

Blueprint consequence: evaluate plan quality, tool selection, test omissions, evidence grounding, defect reproducibility, flaky-test calibration, security behavior, recommendation calibration, and resource cost. Deterministic graders and human review complement model graders.

### 26. Observability needs correlation without uncontrolled cardinality or data leakage

OpenTelemetry models traces, metrics, logs, and context propagation as distinct signals. Its metric guidance warns about cardinality limits. GenAI semantic conventions warn that tool arguments and results may contain sensitive information, and the GenAI conventions continue to evolve.

Supporting sources:

- [OpenTelemetry signals](https://opentelemetry.io/docs/concepts/signals/)
- [OpenTelemetry context propagation](https://opentelemetry.io/docs/concepts/context-propagation/)
- [OpenTelemetry metric cardinality](https://opentelemetry.io/docs/concepts/signals/metrics/)
- [OpenTelemetry GenAI attributes](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/)
- [OpenTelemetry semantic-convention version selection](https://opentelemetry.io/docs/specs/semconv/configuration/version-selection/)

Blueprint consequence: use stable low-cardinality dimensions for metrics and keep campaign, attempt, trace, artifact, and defect IDs in traces/logs. Redact or reference sensitive payloads instead of copying them into telemetry. Map an internal schema to evolving vendor conventions.

### 27. Quality-service SLOs and release quality are different things

Google SRE guidance frames SLOs around user-relevant service indicators and error budgets. A reliable quality agent can still approve the wrong thing; a correct quality recommendation can arrive too late to be useful.

Supporting sources:

- [Implementing SLOs](https://sre.google/workbook/implementing-slos/)
- [Example error-budget policy](https://sre.google/workbook/error-budget-policy/)

Blueprint consequence: operate the quality service with SLOs for intake, start latency, completion, artifact durability, and result publication, while separately evaluating decision correctness, false-block rate, missed-defect rate, and calibration.

### 28. Incident mode should reduce autonomous mutation

Google SRE incident guidance emphasizes explicit roles, an incident record, and early declaration; its incident-management guidance limits system modification to the operations role during an incident. A quality agent may gather and correlate evidence, but uncontrolled retries and environment changes can destroy evidence or worsen load.

Supporting sources:

- [Google SRE incident response](https://sre.google/workbook/incident-response/)
- [Managing incidents](https://sre.google/sre-book/managing-incidents/)

Blueprint consequence: incident mode freezes memory writes and automatic defect publication, stops active/load/security probes, preserves artifacts, and switches the agent to read-only evidence support unless the incident commander grants a bounded action.

### 29. Changes to the quality agent need canaries and failure experiments

Canary guidance recommends separating changing components and comparing representative signals before broad rollout. Chaos engineering defines controlled experiments that build confidence under turbulent conditions. Model, prompt, planner, tool, test-runner image, schema, and policy changes can all alter recommendations.

Supporting sources:

- [Google SRE canarying releases](https://sre.google/workbook/canarying-releases/)
- [Principles of Chaos Engineering](https://principlesofchaos.org/)

Blueprint consequence: upgrade bundles are replayed offline, shadowed, canaried, and rolled back as a unit. Failure injection covers tool timeouts, lost workers, malformed reports, expired credentials, rate limiting, partial artifacts, duplicate callbacks, stale graphs, and model/tool/schema mismatch.

### 30. Typed contracts need an explicit schema dialect and lifecycle

JSON Schema 2020-12 provides an explicit dialect and vocabulary model. A schema without a declared version is difficult to validate consistently across independently deployed workers and adapters.

Supporting sources:

- [JSON Schema specification](https://json-schema.org/specification)
- [JSON Schema 2020-12](https://json-schema.org/draft/2020-12)

Blueprint consequence: all commands, events, results, evidence records, defects, and recommendations declare schema names and versions. Additive compatibility, migration, dual-read periods, and rejection behavior are tested before rollout.

### 31. Third-party integrations require semantic qualification, not a connectivity check

Official tool documentation exposes incompatible semantics at the boundaries a quality platform cares about. GitLab states that uploaded JUnit reports do not determine job status. Playwright preserves a flaky classification after a failed first attempt and successful retry. ZAP lets a plan configure how errors, warnings, and findings map to process exit values. Pact provider states establish isolated interaction preconditions but contract success does not cover unused provider behavior. Appium drivers and plugins can add endpoints beyond the base server. Jira rate limits use multiple evolving scopes and response signals.

Supporting sources:

- [GitLab unit-test reports](https://docs.gitlab.com/ci/testing/unit_test_reports/)
- [Playwright retries](https://playwright.dev/docs/test-retries)
- [ZAP Automation Framework exit status](https://www.zaproxy.org/docs/desktop/addons/automation-framework/job-exitstatus/)
- [Pact provider states](https://docs.pact.io/getting_started/provider_states)
- [Appium API endpoints](https://appium.io/docs/en/latest/reference/api/)
- [Jira Cloud rate limits](https://developer.atlassian.com/cloud/jira/platform/rate-limiting/)

Blueprint consequence: qualify each narrow capability and runner/parser pair against known pass, fail, invalid, retry, cancellation, hostile-content, ambiguous-write, and recovery fixtures. Store the upstream version, adapter release, schemas, permissions, limits, conformance report, known gaps, and refresh triggers. A vendor product name is never a sufficient capability contract.

### 32. Backup, replication, and transcript continuity are not proven recovery

Google SRE data-integrity guidance distinguishes replication from recoverability and emphasizes testing real restoration. A quality platform has multiple data planes with different recovery requirements: losing a rebuildable search projection is not equivalent to losing the event/effect ledger or release evidence.

Supporting source:

- [Google SRE data integrity](https://sre.google/sre-book/data-integrity/)

Blueprint consequence: define RPO/RTO and restore invariants per ledger, artifact, bundle, broker, publication, and derived-index plane. Restore drills reconstruct a sampled recommendation and its evidence graph, reconcile leases and unknown effects, fence stale regions/workers, and invalidate any recommendation whose evidence completeness cannot be proven.

## Source register

The following register records what each source contributes and what it does not prove. “Current” means checked on the research date, not permanently stable.

| Source | Verified | Used for | Important limitation or refresh trigger |
| --- | --- | --- | --- |
| [OpenAI model guidance](https://developers.openai.com/api/docs/guides/latest-model) | 2026-08-31 | autonomy boundaries, tool contracts, compaction, upgrade evaluation | model-specific guidance evolves; refresh on model-family migration |
| [OpenAI agent evals](https://developers.openai.com/api/docs/guides/agent-evals) | 2026-08-31 | trace, dataset, grader, and eval-run strategy | hosted product surface; preserve vendor-neutral internal eval records |
| [OpenAI compact-response API](https://developers.openai.com/api/reference/java/resources/responses/methods/compact) | 2026-08-31 | compaction shape and opaque continuation | SDK examples and supported models can change |
| [Bazel Test Encyclopedia](https://bazel.build/reference/test-encyclopedia) | 2026-08-31 | hermetic tests, declared inputs, temp space, sharding | Bazel-specific mechanics; principles generalized carefully |
| [Playwright best practices](https://playwright.dev/docs/best-practices) | 2026-08-31 | user-visible assertions, isolation, trace strategy | browser-framework specific |
| [Playwright retries](https://playwright.dev/docs/test-retries) | 2026-08-31 | attempt and flake classification | retry behavior is runner-specific; normalize explicitly |
| [Playwright CI](https://playwright.dev/docs/ci) | 2026-08-31 | worker and shard trade-offs | recommended worker count is not universal for all runners |
| [Playwright fixtures](https://playwright.dev/docs/test-fixtures) | 2026-08-31 | fixture scope and isolation | framework-specific fixture lifecycle |
| [Playwright projects](https://playwright.dev/docs/test-projects) | 2026-08-31 | browser/environment matrices | project configuration is not a risk model by itself |
| [Selenium test practices](https://www.selenium.dev/documentation/test_practices/) | 2026-08-31 | test architecture boundary | Selenium is a browser-automation library, not a quality platform |
| [Selenium performance warning](https://www.selenium.dev/documentation/test_practices/discouraged/performance_testing/) | 2026-08-31 | browser/load separation | does not prohibit collecting bounded browser timings |
| [Android test runner](https://developer.android.com/training/testing/instrumented-tests/androidx-test-libraries/runner) | 2026-08-31 | instrumentation isolation and sharding | Android-specific; device/vendor variation remains |
| [Android managed devices](https://developer.android.com/studio/test/managed-devices) | 2026-08-31 | repeatable virtual-device matrices | does not replace targeted physical-device evidence |
| [Swift Testing](https://developer.apple.com/documentation/testing) | 2026-08-31 | Apple test execution and parallelism | JavaScript-rendered documentation may change without stable anchors |
| [XCTest](https://developer.apple.com/documentation/xctest) | 2026-08-31 | Apple UI/performance boundaries | platform-specific |
| [Appium CLI](https://appium.io/docs/en/latest/reference/cli/) | 2026-08-31 | driver/plugin lifecycle | extensions have separate release and trust lifecycles |
| [Appium API endpoints](https://appium.io/docs/en/latest/reference/api/) | 2026-08-31 | base, driver, and plugin endpoint boundary | endpoint availability is driver/plugin specific; re-inventory on extension change |
| [Pact documentation](https://docs.pact.io/) | 2026-08-31 | consumer-driven contract model | contract examples do not cover unused API behavior |
| [Pact provider states](https://docs.pact.io/getting_started/provider_states) | 2026-08-31 | isolated interaction preconditions | provider-state success does not prove functional side effects or unused behavior |
| [k6 thresholds](https://grafana.com/docs/k6/latest/using-k6/thresholds/) | 2026-08-31 | executable performance criteria | threshold pass/fail alone is incomplete diagnosis |
| [k6 load-testing guidance](https://grafana.com/docs/k6/latest/testing-guides/load-testing-websites/) | 2026-08-31 | protocol/browser workload split | tool-specific execution details |
| [ZAP Automation Framework](https://www.zaproxy.org/docs/automate/automation-framework/) | 2026-08-31 | explicit security-test plan and outcomes | active testing still requires authorization and expert triage |
| [ZAP exit-status job](https://www.zaproxy.org/docs/desktop/addons/automation-framework/job-exitstatus/) | 2026-08-31 | configurable warning/error/finding exit semantics | plan/add-on changes can change the meaning of a process exit value |
| [NIST SP 800-218](https://csrc.nist.gov/pubs/sp/800/218/final) | 2026-08-31 | risk-based secure-development testing | high-level framework, not a runner specification |
| [OWASP WSTG latest](https://owasp.org/www-project-web-security-testing-guide/latest/) | 2026-08-31 | web security-test catalog | rolling latest content; pin versions in campaign evidence |
| [WCAG 2.2](https://www.w3.org/TR/wcag/) | 2026-08-31 | accessibility conformance baseline | conformance interpretation requires complete scope and human review |
| [ACT Rules Format](https://www.w3.org/WAI/standards-guidelines/act/) | 2026-08-31 | accessibility rule automation model | ACT rules are informative implementations |
| [Hypothesis stateful testing](https://hypothesis.readthedocs.io/en/latest/stateful.html) | 2026-08-31 | generated sequences and minimization | examples and APIs depend on Hypothesis version |
| [Hypothesis replay guidance](https://hypothesis.readthedocs.io/en/latest/tutorial/replaying-failures.html) | 2026-08-31 | durable reproducer strategy | opaque reproduce blobs are not long-term test cases |
| [PIT mutation testing](https://pitest.org/) | 2026-08-31 | assertion effectiveness signal | equivalent and unreachable mutants limit scores |
| [NIST ACTS project](https://csrc.nist.gov/Projects/Automated-Combinatorial-Testing-for-Software/) | 2026-08-31 | interaction coverage | requires a correct parameter and constraint model |
| [LLVM libFuzzer](https://llvm.org/docs/LibFuzzer.html) | 2026-08-31 | coverage-guided fuzz lifecycle | compiler and harness specific |
| [Csmith](https://github.com/csmith-project/csmith) | 2026-08-31 | differential oracle example | undefined behavior and shared defects complicate comparison |
| [git bisect](https://git-scm.com/docs/git-bisect) | 2026-08-31 | revision minimization | bad test scripts or environmental drift corrupt results |
| [Testcontainers](https://testcontainers.com/getting-started/) | 2026-08-31 | on-demand dependency environments | container realism does not guarantee production equivalence |
| [Testcontainers reusable containers](https://java.testcontainers.org/features/reuse/) | 2026-08-31 | isolation/reuse trade-off | experimental and documented as unsuitable for CI; refresh before any reuse claim |
| [Nx affected](https://nx.dev/docs/features/ci-features/affected) | 2026-08-31 | graph-based regression selection | depends on correct base and graph metadata |
| [Google test-selection research](https://research.google/pubs/assessing-transition-based-test-selection-algorithms-at-google/) | 2026-08-31 | large-scale selection limitations | organization-specific data and infrastructure |
| [Microsoft test-selection research](https://www.microsoft.com/en-us/research/publication/data-driven-test-selection-at-scale/) | 2026-08-31 | statistical selection evidence | reported savings are not portable guarantees |
| [GitHub Actions secure use](https://docs.github.com/en/actions/reference/security/secure-use) | 2026-08-31 | untrusted CI boundary | GitHub-specific mechanics; principle applies elsewhere |
| [GitHub `pull_request_target` security](https://docs.github.com/en/actions/reference/security/securely-using-pull_request_target) | 2026-08-31 | privileged-event and untrusted-code split | provider protections evolve; re-audit workflow triggers and checkout behavior |
| [GitHub artifact attestations](https://docs.github.com/en/actions/concepts/security/artifact-attestations) | 2026-08-31 | provenance and verification | provenance is not a security verdict |
| [GitLab unit-test reports](https://docs.gitlab.com/ci/testing/unit_test_reports/) | 2026-08-31 | report/gate separation | parser limits and behavior can change by GitLab version |
| [TestRail API introduction](https://support.testrail.com/hc/en-us/articles/7077083596436-Introduction-to-the-TestRail-API) | 2026-08-31 | external result publication and rate limits | cloud and server editions may differ |
| [Jira Cloud rate limits](https://developer.atlassian.com/cloud/jira/platform/rate-limiting/) | 2026-08-31 | adapter backpressure | limit policies are explicitly evolving |
| [OWASP Agentic Top 10](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) | 2026-08-31 | agent threat categories | threat taxonomy, not a complete control standard |
| [JSON Schema 2020-12](https://json-schema.org/draft/2020-12) | 2026-08-31 | contract dialect and versioning | validators support vocabularies unevenly; test implementation |
| [OpenTelemetry signals](https://opentelemetry.io/docs/concepts/signals/) | 2026-08-31 | traces, metrics, logs, context | profiles and GenAI conventions continue evolving |
| [OpenTelemetry GenAI attributes](https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/) | 2026-08-31 | tool/model telemetry and sensitivity | stability and opt-in rules require refresh |
| [Google SRE SLO guidance](https://sre.google/workbook/implementing-slos/) | 2026-08-31 | service indicators and objectives | examples require product-specific calibration |
| [Google SRE incident response](https://sre.google/workbook/incident-response/) | 2026-08-31 | operational roles and record | organization-specific incident process still required |
| [Google SRE canarying](https://sre.google/workbook/canarying-releases/) | 2026-08-31 | staged agent-component rollout | statistical method must fit local traffic and risk |
| [Google SRE data integrity](https://sre.google/sre-book/data-integrity/) | 2026-08-31 | backup/restore and recovery testing | product-specific RPO/RTO, tenancy, and regional topology still require design |
| [Principles of Chaos Engineering](https://principlesofchaos.org/) | 2026-08-31 | controlled failure experiments | principles do not authorize production experiments |

## Important disagreements and cautions

### “Green after retry” versus “flaky”

Some CI dashboards visually collapse a retrying job to green. Runner documentation supports retaining the flaky classification. The blueprint treats final workflow status and attempt history as different fields.

### “Coverage target” versus “quality target”

Teams often enforce a line or branch percentage because it is measurable. Mutation, requirements, and risk evidence show why one percentage is insufficient. The blueprint supports dimension-specific policy and explicitly reports unknown coverage.

### “Real dependency” versus “production-equivalent environment”

Containers improve fidelity relative to mocks, but image configuration, data volume, topology, permissions, and managed-service behavior may still differ. The blueprint records environment equivalence claims and exceptions rather than assuming them.

### “Automated accessibility” versus “accessibility conformance”

Automated rules are useful regression detectors. WCAG and ACT sources do not justify a claim of complete conformance from an automated run. The blueprint uses precise language and creates a manual-review queue.

### “Attested” versus “trusted”

An attestation binds provenance. It does not make a malicious test runner, incorrect oracle, or contaminated fixture trustworthy. A verifier and a policy decision are separate controls.

### “Model memory” versus “system of record”

Compaction and conversational memory help continuity. Neither is an auditable defect, test-attempt, or release-evidence store. The blueprint keeps a model-independent campaign ledger.

### “Agent framework” versus “owned control plane”

Frameworks may provide tracing, state, and tool orchestration. The system still owns schemas, authorization, policy, idempotency, evidence lineage, runner isolation, and upgrade gates. Replacing a framework must not invalidate campaign history.

## Claims intentionally not made

- No universal coverage percentage indicates release readiness.
- No fixed number of retries proves or disproves flakiness.
- No test-selection model is safe without a conservative fallback.
- No single browser, emulator, accessibility scanner, contract suite, mutation score, or load threshold represents total product quality.
- No LLM self-review is accepted as independent evidence.
- No release recommendation automatically grants promotion authority.
- No provenance signature proves an artifact is benign or semantically correct.
- No shared environment is called hermetic merely because it is containerized.
- No vendor benchmark or reported compute saving is treated as a portable capacity promise.

## Derived blueprint documents

- [Blueprint overview and reader path](../../agents/test-quality-engineering-agent/README.md)
- [Mission, boundaries, and reference architecture](../../agents/test-quality-engineering-agent/01-mission-boundaries-and-reference-architecture.md)
- [Test design, selection, oracles, and risk](../../agents/test-quality-engineering-agent/02-test-design-selection-oracles-and-risk.md)
- [Tool contracts, environments, fixtures, and isolation](../../agents/test-quality-engineering-agent/03-tool-contracts-environments-fixtures-and-isolation.md)
- [Browser, mobile, API, performance, security, and accessibility](../../agents/test-quality-engineering-agent/04-domain-validation-boundaries.md)
- [State, context, planning, parallelism, and memory](../../agents/test-quality-engineering-agent/05-state-context-planning-parallelism-and-memory.md)
- [Defect, flaky-test, coverage, and release evidence](../../agents/test-quality-engineering-agent/06-defects-flaky-tests-coverage-and-release-evidence.md)
- [Security, identity, integrations, and provenance](../../agents/test-quality-engineering-agent/07-security-identity-integrations-and-provenance.md)
- [Reliability, observability, evaluation, and operations](../../agents/test-quality-engineering-agent/08-reliability-observability-evaluation-and-operations.md)
- [Build roadmap and reference contracts](../../agents/test-quality-engineering-agent/09-build-roadmap-and-reference-contracts.md)
- [Adapter qualification and conformance](../../agents/test-quality-engineering-agent/10-adapter-qualification-and-conformance.md)

## Refresh triggers

Refresh this packet when any of the following occurs:

- the agent model, provider API, compaction behavior, or evaluation surface changes;
- a runner, browser, mobile driver/plugin, security scanner, load tool, contract specification, or report schema changes materially;
- an adapter's permission surface, exit/retry/cancel semantics, hosted API rate-limit behavior, upstream defaults, or conformance outcome changes;
- CI permission semantics, artifact retention, attestation verification, or external API rate limits change;
- WCAG, ACT, OWASP WSTG, OWASP agentic guidance, NIST SSDF, or organizational policy advances to a new normative baseline;
- local incident or eval evidence invalidates a recommendation, retry policy, selection fallback, or isolation assumption;
- the quality agent receives new write, network, production-target, or deployment-adjacent authority.
- a restore/failover drill misses its evidence-integrity, RPO, RTO, fencing, or external-effect reconciliation target.

## Research limitations

This is an architecture packet, not a substitute for product-specific risk analysis, regulatory interpretation, or tool qualification. The source set spans mainstream test domains but cannot enumerate every language, device lab, safety-critical standard, or proprietary CI/test-management integration. Before implementation, bind the blueprint to the repository graph, application architecture, threat model, data classification, supported platform matrix, release policy, and measured failure history of the actual system.

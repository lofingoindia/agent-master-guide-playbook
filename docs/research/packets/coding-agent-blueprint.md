# Research Packet: Production Coding-Agent Blueprint

> **Status:** Synthesis complete; supports the coding-agent blueprint  
> **Research window:** 2026-08-31  
> **Scope:** Repository discovery, context/compaction/memory, planning, edits, terminal/build/test execution, Git/worktrees, IDE/MCP/SCM/CI adapters, patch provenance, review, security/isolation, supply chain, human approval, CI/background operation, recovery, evaluation, scaling, upgrades, incident learning, and safe autonomy

## Guides supported

- [Production coding-agent blueprint](../../agents/coding-agent/README.md)
- [Architecture and runtime selection](../../agents/coding-agent/01-architecture-and-runtime-selection.md)
- [Repository discovery, context, and planning](../../agents/coding-agent/02-repository-discovery-context-and-planning.md)
- [Tools, edits, terminal, and patch provenance](../../agents/coding-agent/03-tools-edits-terminal-and-patch-provenance.md)
- [Security, permissions, sandboxing, and supply chain](../../agents/coding-agent/04-security-permissions-sandboxing-and-supply-chain.md)
- [Reliability, recovery, and concurrency](../../agents/coding-agent/05-reliability-recovery-and-concurrency.md)
- [Observability, evaluation, and acceptance](../../agents/coding-agent/06-observability-evaluation-and-acceptance.md)
- [CI, deployment, operations, and economics](../../agents/coding-agent/07-ci-deployment-operations-and-economics.md)
- [Build roadmap and reference contracts](../../agents/coding-agent/08-build-roadmap-and-reference-contracts.md)

## Research questions

1. What distinguishes a safe repository-level coding agent from a model with shell and file access?
2. Which guarantees must be application-owned when using a provider model, agent framework, coding harness, IDE, or managed coding product?
3. How should CLI, IDE, background/CI, and custom-service architectures divide identity, state, execution, and integration authority?
4. How should repository instructions, source, issues, tests, tool output, and external content be classified and compiled into context?
5. Which repository discovery/search/index techniques are robust across monorepos, languages, generated code, submodules, and large histories?
6. What terminal and edit contracts safely support a nondeterministic caller without pretending a command allowlist is containment?
7. When should a run use an existing checkout, Git worktree, ephemeral clone, agent branch, or one-PR background workflow?
8. What provenance connects requester, base commit, instructions, model/harness, exact patch, checks, approvals, and integration effects?
9. Which threat model covers prompt injection, malicious repository execution, secret exfiltration, sandbox escape, CI confused deputies, and dependency/configuration persistence?
10. How do native sandboxes, rootless containers, gVisor-style runtimes, microVMs, and dedicated VMs differ in trust and operations?
11. How should retry, cancellation, crash recovery, stale bases, flaky tests, concurrent agents, and unknown publication outcomes behave?
12. Which eval tasks and graders measure repository outcome, trajectory/policy, security, recovery, review quality, latency, and cost?
13. What can public benchmarks establish, and which product requirements remain unmeasured?
14. What is the smallest credible build roadmap from read-only analysis to selectively autonomous background work?
15. Which state belongs in turn context, working notes, session history, durable execution, domain sources, long-term memory, and curated episodic learning?
16. Which portable contracts survive IDE, MCP, GitHub, GitLab, Bitbucket, and CI-provider differences without weakening provider-native policy?
17. How should model, prompt, context, tool, adapter, policy, and executor upgrades migrate active runs and convert incidents into regressions?

## Search breadth and source selection

Research covered these source families:

- official Git command documentation for worktrees, status porcelain, diffs, patch application, commit verification, and trailers;
- current GitHub documentation for cloud coding agents, session/audit behavior, branches/PRs, firewall limits, repository instructions, MCP configuration, agentic workflows, protected branches, Actions security, app credentials, and artifact attestations;
- current VS Code documentation for Workspace Trust, agent security, approvals, terminal command policy, diff review, URL trust, and preview sandboxing;
- current VS Code extension API and Language Server Protocol documentation for versioned document/workspace edits and IDE adapter semantics;
- current Anthropic Claude Code documentation and engineering reports for permissions, sandboxing, worktrees, hooks, subagents, long-running harnesses, and agent evaluation;
- Google Jules task/VM/security and API documentation as a second managed-background architecture;
- OpenHands runtime/sandbox documentation, SWE-agent's Agent-Computer Interface work, and Aider repository-map documentation as open harness/interface evidence;
- Docker, gVisor, Firecracker, and Linux Landlock primary documentation for isolation semantics and limitations;
- GitHub Actions, NIST SSDF, SLSA, in-toto, and OpenSSF primary sources for CI and software-supply-chain controls;
- NIST adversarial-ML, OWASP agentic-security, MCP specification/authorization, and GitHub Security Lab materials for prompt/tool/configuration threats;
- original SWE-bench, SWE-agent, SWE-bench-Live, SWE-rebench, OpenHands, and Terminal-Bench papers/repositories for evaluation design and limitations;
- current OpenAI official model guidance for model/tool/prompt evaluation in coding-agent systems;
- current GitLab and Bitbucket documentation for merge/pull-request APIs, protected-branch/branch-restriction semantics, CI status, and provider portability;
- current context/long-running harness and managed-agent architecture guidance for compaction, handoff, upgrade boundaries, and stored lineage.

Discovery sources, news coverage, and community reports were used only to find questions or primary disclosures. The guides rely on official documentation, official repositories/specifications, or original papers for material mechanics.

## Version and date baseline

| Area | Baseline checked on 2026-08-31 | Volatility/limitation |
|---|---|---|
| Git | Current `git-scm.com` worktree/status/apply/diff/verify-commit manuals; machine-readable porcelain and patch applicability semantics | Exact installed Git version and platform behavior must be recorded per executor |
| OpenAI | Current official model guidance, including coding-agent autonomy/tool/prompt recommendations | Models, aliases, prices, reasoning settings, and platform features are high-volatility; exact resolved model belongs in every run/eval |
| GitHub cloud coding agent | Current docs: ephemeral Actions-backed environment, one branch/one PR per task, restricted push/integration, human merge, session logs and signed attribution; current page lists a 59-minute session limit | Product policy, availability, runner/firewall behavior, and limits can change; not portable to custom agents automatically |
| GitHub Agentic Workflows | Current docs: Markdown workflow compiled to hardened workflow, read-only defaults, declared safe outputs, secrets outside agent runtime | Product-specific and rapidly evolving; still depends on repository/ruleset configuration |
| GitLab/Bitbucket SCM | Current merge/pull-request APIs, protected-branch/branch-restriction and pipeline/status docs | Effective policy, API fields, eventual consistency, permissions, rate limits, and plan/instance features differ; adapters require conformance tests |
| VS Code agents | Security/approval/Workspace Trust docs published/checked late August 2026; agent sandboxing documented as preview on supported environments | Preview, platform-specific, and IDE/extension configuration dependent |
| IDE edit protocols | Current VS Code `WorkspaceEdit` API and LSP 3.18 versioned document/workspace edit semantics | Editor APIs apply edits but do not supply coding-agent authorization, base provenance, or Git conflict policy |
| Claude Code | Current sandbox, security, worktree, hook, agent/subagent docs checked; recent architecture reports from 2025–2026 | Product defaults and permission modes evolve quickly; vendor measurements are workload-specific |
| Jules | Current docs: one cloud VM/log/change environment per task; internet-access security warning | Managed implementation details and product limits are not full portable guarantees |
| MCP | Stable specification release `2026-07-28`; stateless core, authorization hardening, formal extensions/deprecation policy; Tier 1 TypeScript/Python/Go/C# SDK support reported by maintainers | Breaking change from `2025-11-25`; tool annotations remain untrusted unless server trust is established; application authorization/containment remains external |
| NIST SSDF | SP 800-218 version 1.1, published 2022 | Stable high-level practices, not coding-agent-specific implementation |
| NIST AML | AI 100-2 E2025, published 2025 | Taxonomy supports threat modeling; does not prescribe a complete coding-agent sandbox |
| OWASP agentic security | Top 10 for Agentic Applications 2026 | Community risk framework; use as a catalog, not a certification |
| SLSA/in-toto | SLSA provenance v1 predicate family and current in-toto 3.1.0 reference docs | Provenance establishes production path/identity, not semantic code correctness |
| Isolation | Current Docker rootless/seccomp, gVisor architecture/security, Firecracker design/production setup, latest Linux Landlock docs | Kernel/runtime/host patch level and executor configuration dominate actual security |
| Public evals | SWE-bench family, SWE-agent, SWE-bench-Live/SWE-rebench, Terminal-Bench 2.0/continuous releases | Public/static contamination, task/environment quality, scaffold differences, and narrow product coverage require private recent evals |
| Context/long runs | Current Anthropic context-engineering, long-running harness, and managed-agent architecture reports | Product/model observations are not universal; compaction is lossy and application state/lineage must remain external |

## Synthesis: conclusions that held across source families

### 1. A coding agent is a constrained change-production system

CLI/IDE agents, GitHub's cloud agent, Jules, OpenHands, SWE-agent, and Claude Code all combine a model loop with repository access, edit/process tools, and some execution environment. The important production distinction is who owns authority and integration.

**Synthesis:** the model is an untrusted planner/proposer. The application owns identity, scope, path/process/network policy, workspace isolation, approvals, state, check receipts, patch provenance, branch/PR integration, and completion. A framework can implement hooks for these, but the product must define and test the guarantee.

### 2. Patch production and merge/release authority should be separated

GitHub's managed agent restricts work to a dedicated branch/PR and cannot merge; Actions security documentation warns against executing untrusted PR code with secrets/write tokens. Supply-chain systems similarly separate untrusted build steps from trusted attestation/integration identities.

**Synthesis:** the ordinary agent executor should produce an immutable patch manifest without a general repository-write credential. A separate integration service validates exact bytes/base/approval and updates one agent branch/draft PR using a short-lived scoped identity. Normal protected-branch review and CI remain final authority.

### 3. Repository content is both working material and adversarial input

VS Code Workspace Trust disables agents for untrusted workspaces; GitHub and NIST explicitly recognize prompt injection; Claude Code sandboxing focuses on containing successful injection. Repository instructions, tests, build scripts, package lifecycle scripts, hooks, plugins, MCP definitions, and CI configuration can change behavior or execute code.

**Synthesis:** separate instruction applicability from authority. Platform policy and authenticated user scope outrank versioned repository instructions; ordinary source/issues/tool results are untrusted evidence. No repository file may grant secrets, network, tool eligibility, or publication authority. Behavioral configuration changed by an agent must not self-activate before trusted review/merge.

### 4. Approval and sandboxing solve different problems

IDE/CLI products expose approval prompts, while Anthropic reports approval fatigue and fewer prompts under sandboxing. An approval can be stale, opaque, or uninformed; a sandbox cannot decide whether a product-semantic change is intended.

**Synthesis:** use containment and least privilege to cap reachable damage, deterministic policy for mechanical rules, and exact bytes/facts-bound approval for consequential intent decisions. A changed diff, base, command, destination, identity, or policy invalidates approval.

### 5. Tests are necessary evidence and hostile executable input

Coding benchmarks and products depend on executing tests, but CI security guidance treats code, dependencies, build scripts, and tests from untrusted revisions as attacker-controlled. Passing tests are limited by their oracle and environment.

**Synthesis:** run tests in a secretless, egress-restricted, resource-bounded sandbox; record a versioned profile and receipt bound to the final patch digest. Grade success in an independent fresh evaluator with hidden tests and policy invariants. Passing tests never imply authorization, security, migration, performance, or specification completeness.

### 6. Worktrees isolate files, not all state or semantics

Git supports multiple working trees; Claude Code and other products use them for parallel sessions. Worktrees share repository history/object storage, do not include ambient uncommitted changes by default, and do not prevent semantic conflicts across manifests, lockfiles, generators, schemas, or tests.

**Synthesis:** use one writer per worktree/branch. Make the base commit explicit. Preserve or explicitly snapshot user changes rather than silently stashing/resetting. Integrate parallel patches through version control and rerun checks; do not treat separate directories as proof of independent changes.

### 7. Shell/process execution needs an environmental boundary

SWE-agent showed that tool/interface design affects model performance; coding tools commonly expose a shell for flexibility. However, interpreters, build tools, test runners, package scripts, hooks, and compilers can all express arbitrary code, defeating command-name policy as a security boundary.

**Synthesis:** prefer structured argv/cwd/env/time/resource/network contracts, but assume every permitted repository-controlled command is hostile. Apply filesystem, network, credential, process, output, and kernel containment. Kill the process group/container/VM and fence publication; a language timeout does not undo effects or guarantee descendants stopped.

### 8. “Sandbox” must name concrete layers

Docker rootless mitigates daemon/runtime privilege and seccomp narrows syscalls but containers share a host kernel. gVisor adds a userspace application kernel. Firecracker adds a guest kernel/VMM and process jail, while leaving network filtering to the operator. Landlock's behavior varies by ABI and pre-opened resources.

**Synthesis:** select by repository trust, multi-tenancy, asset value, compatibility, startup, and operations. Document mounts, root filesystem, user/capabilities, syscalls/kernel, processes/resources, devices, network/metadata, credentials, cleanup, and host patching. Use stronger isolation for untrusted multi-tenant arbitrary code, not a “Docker means safe” assumption.

### 9. Durable conversation is not durable execution

Managed agents and frameworks preserve sessions/events; workflow engines preserve recorded steps. Neither automatically resolves a process lost mid-command, a patch half-written in a workspace, an external request completed before its response was recorded, or a stale approval.

**Synthesis:** keep run/workspace/plan/effect/patch/check/approval/publication state separately. Recover by fencing old workers, reconciling unknown effects, provisioning a clean workspace, reapplying a validated immutable patch, and rerunning invalidated checks. Exactly-once publication requires destination participation or reconciliation.

### 10. The harness is part of the evaluated system

SWE-agent attributes gains to its Agent-Computer Interface; OpenHands exposes a runtime architecture; provider reports show context/tool/harness changes affecting outcomes and cost. Public leaderboard rows commonly vary model and scaffold together.

**Synthesis:** version and evaluate model, prompts/instructions, context compiler, tool schemas, controller, executor, policy, and grader together. Report exact versions, environments, limits, repetitions, cost, and failure slices.

### 11. Public coding benchmarks are discovery signals, not product release gates

SWE-bench provides realistic repository issue-to-patch tasks but is public, static, language/repository narrow, and highly sensitive to tests/environments/scaffold. Live/continuously refreshed variants were created to reduce temporal contamination; Terminal-Bench broadens terminal tasks but does not test a production branch/approval/provenance workflow.

**Synthesis:** maintain recent private tasks, hidden tests, malicious repositories, recovery injection, code-review tasks, and production regressions. Use deterministic final state plus trajectory/policy graders across repeated trials. Do not average away a critical forbidden effect.

### 12. Multi-agent and custom-service complexity require evidence

Anthropic's 2026 long-running application report shows planner/builder/evaluator orchestration can improve difficult output, but one comparison cost more than 20 times the solo run. Parallel sessions multiply tokens and file/semantic conflict risks; custom services add control plane, sandbox fleet, data governance, and on-call burden.

**Synthesis:** start with one accountable agent and deterministic independent verification. Add a read-only explorer or reviewer only for a measured failure mode. Start with a CLI/IDE harness or one background worker; add durable/fleet/multi-tenant infrastructure only when requirements cross those boundaries.

### 13. Context, compaction, and memory are different systems

Current context-engineering and long-running harness reports agree that context is finite, compaction is lossy, and structured handoff artifacts help continuity. Memory frameworks distinguish thread-scoped state from cross-session stores, but those APIs do not determine coding-agent authority.

**Synthesis:** compile a bounded per-decision context with a manifest and reserve mandatory/output/recovery lanes. Keep working notes, session history, durable execution/effect state, domain sources, long-term preferences, and curated episodes in different stores. Default cross-repository long-term memory off; promote observed incidents/outcomes into reviewed evals or playbooks rather than free-form recollection. Validate compaction from authoritative state and preserve raw lineage.

### 14. Integration portability requires an internal effect contract

VS Code/LSP edit APIs, MCP tools, and GitHub/GitLab/Bitbucket APIs expose different version, permission, result, eventual-consistency, and policy semantics. MCP explicitly treats tool annotations as untrusted without server trust; code hosts expose provider-specific protection and change-request behavior.

**Synthesis:** normalize every adapter into repository/base/workspace identity, effect class, stable operation ID, attempt/effect state, postcondition, and raw receipt. Admit/pin tool releases and schemas, query effective provider policy, validate edits against current document/blob versions, and conformance-test lost responses, duplicates, stale bases, auth, pagination, rate limits, and schema drift.

### 15. The upgrade unit is the whole behavioral bundle

Model, prompt, context compiler, compactor, tool schemas, policy, adapters, sandbox image, and graders can each change behavior without an obvious controller-code diff. Long-running architecture reports also warn that harness assumptions go stale as models improve.

**Synthesis:** pin active runs to a compatible release bundle; migrate only through fencing, unknown-effect reconciliation, authority/approval revalidation, and a new checkpoint. Replay, shadow, canary, and gradually promote changes with non-compensating safety stops. Convert incidents to minimal owned regressions and periodically remove harness complexity that fails ablation value tests.

## Architecture alternatives and resolution

| Alternative | Evidence-backed fit | Resolution used in guides |
|---|---|---|
| CLI/local harness | Fast, composable, strong developer steering; risks ambient host/user authority | Best initial interactive product for trusted repos, with worktree and tested sandbox; not unattended untrusted execution |
| IDE integration | Diff/diagnostics/Workspace Trust and review flow; extension/session attack surface | Use when human review and local context dominate; preserve same backend contracts |
| CI/background worker | Fresh environment, logs, branch/PR flow; untrusted trigger/code and CI-secret risk | Default for asynchronous patches, using secretless executor plus separate integration gate |
| Managed coding agent | Fast adoption and maintained harness; product-specific/opaque limits | Adopt if controls satisfy requirements; independently validate data, security, branch, and eval behavior |
| Open coding harness/SDK | Repository/process abstractions and sandbox integrations | Wrap behind owned policy/state/events; pin and include it in evals |
| Thin custom loop | Precise transitions and minimal dependencies | Default custom baseline for small scope; exit when rebuilding durable/runtime platform |
| General agent framework | Tool/session/trace/approval convenience | Adopt only for demonstrated removed work; coding-specific guarantees remain application-owned |
| Durable workflow engine | Host-loss recovery, long waits, cross-service state | Add only when run survival/approval/fleet semantics require it; keep effect idempotency and sandbox external |
| Multi-agent planner/builder/reviewer | May help decomposable long tasks and independent review | Not default; require ablation against single-agent accepted outcome and cost |
| Native OS sandbox | Low startup and host compatibility | Interactive trusted-code option; verify per OS/ABI and pre-opened resources |
| Rootless hardened container | Broad toolchain compatibility | Ordinary lower-risk background baseline; not a separate kernel |
| gVisor/microVM | Stronger hostile multi-tenant containment | Use for untrusted/high-value workloads after compatibility and operational tests |

## Important contradictions and how they were resolved

| Apparent contradiction | Evidence | Resolution |
|---|---|---|
| More approvals are safer vs fewer approvals are safer | IDE/CLI permission systems emphasize consent; sandbox reports emphasize fatigue reduction | Contain broadly up front; prompt only for consequential semantic choices; bind exact effects |
| A container safely runs arbitrary code vs containers share a kernel | OpenHands describes Docker isolation; Docker/gVisor/Firecracker show different kernel boundaries | Treat Docker as one layer; choose stronger runtime by threat/tenant/asset risk and test escape paths |
| Repository instructions improve performance vs repository files are injectable | Agent products support `AGENTS.md`/custom instructions; NIST/VS Code/GitHub document indirect injection risk | Use scoped versioned instructions as configuration, never as authority; protect behavioral config activation |
| Broad shell maximizes capability vs narrow tools improve reliability/safety | Coding harnesses depend on shell; SWE-agent/tool-design evidence shows interface matters | Keep a small typed surface and structured command contract; retain shell only inside strong containment |
| Self-review catches defects vs the same model shares blind spots | Managed products add self-review; eval literature favors independent deterministic/human evidence | Self-review is advisory; final fresh tests/analyzers/review are independent gates |
| Passing tests proves patch success vs tests can be incomplete/malicious | Benchmarks use executable tests; CI security treats repo code as hostile | Run sandboxed and grade hidden/final state; tests are evidence, not authorization or proof of completeness |
| Worktrees make parallel agents safe vs shared changes still conflict | Git/Claude docs show file isolation; shared manifests/build semantics remain | One writer/worktree, explicit partition, serialized integration and revalidation |
| Resume/session means reliable long work vs effects may be ambiguous | Managed-agent/session docs vs distributed effect ambiguity | Persist distinct run/effect/patch state; reconcile and resume from clean workspace |
| Signed commits/provenance establish trust vs code may still be wrong | Git signatures/SLSA attest producer and path | Use for attribution/integrity; retain tests, review, policy, and protected integration |
| Public benchmark leader means best production system vs scaffolds/tasks differ | SWE-agent/SWE-bench/Live/Terminal sources | Treat as ecosystem signal; gate on recent private complete-system evals |
| Large context improves repo understanding vs lean context reduces cost/noise | Long-context products and repository maps vs model/tool guidance | Stage discovery and load exact evidence just in time; retain durable external artifacts |
| Compaction/memory preserves continuity vs summaries can lose or launder state | Context/harness reports and memory architecture | Preserve raw lineage and authoritative run/effect state; version/validate checkpoints; gate cross-session writes separately |
| Nearest instruction file wins vs products merge several instruction formats | GitHub repository and CLI instruction docs | Compile applicability per target/file family; application authority always wins; consequential cross-format conflict blocks or follows an explicit rule |
| Multi-agent improves hard tasks vs multiplies cost/conflict | Anthropic harness result and parallel-agent docs | Add only for independently decomposable measured slices with worktree/budget controls |
| CI is reproducible and safe vs privileged workflows execute attacker content | GitHub Actions/CD architecture and pwn-request guidance | Split secretless untrusted execution from privileged exact-artifact integration |
| Tool metadata says read-only/destructive vs metadata can lie | MCP tool annotations and security guidance | Treat annotations as untrusted until server/release approved; enforce effect policy at gateway |
| One generic SCM adapter is portable vs protection/API semantics differ | GitHub, GitLab, and Bitbucket primary docs | Normalize contracts but retain provider capability/policy probes, receipts, eventual-consistency handling, and conformance suites |
| Pinning creates stability vs old harness assumptions become harmful | Managed-agent and model-upgrade evidence | Pin active runs; evaluate new bundles; simplify prompts/tools after ablation rather than accumulating permanent workarounds |

## Failure evidence and resulting tests

| Observed/documented failure surface | Direct source | Test/control derived |
|---|---|---|
| Indirect prompt injection can expose tokens/files or execute code | [VS Code prompt-injection security report](https://github.blog/security/vulnerability-research/safeguarding-vs-code-against-prompt-injections/), [NIST AI 100-2 E2025](https://csrc.nist.gov/pubs/ai/100/2/e2025/final) | Inject instructions across source/docs/issues/logs; honeytokens, egress denial, authority invariants |
| Permission prompts create fatigue; filesystem and network boundaries are complementary | [Anthropic sandboxing report](https://www.anthropic.com/engineering/claude-code-sandboxing), [Claude sandbox docs](https://code.claude.com/docs/en/sandboxing) | Up-front confinement, exact high-risk approval, filesystem-plus-network escape suite |
| Process/local runtime provides no isolation | [OpenHands Process Sandbox](https://docs.openhands.dev/openhands/usage/sandboxes/process) | Block unattended/untrusted use on process runtime; explicit environment warning/gate |
| Mounted files, internet, and provided credentials remain reachable in a Docker runtime | [OpenHands FAQ](https://docs.openhands.dev/overview/faqs) | No host mounts/secrets by default; network profile and malicious-test suite |
| Firecracker does not filter network traffic by itself | [Firecracker production setup](https://github.com/firecracker-microvm/firecracker/blob/main/docs/prod-host-setup.md) | External egress proxy/firewall and metadata/loopback tests even for microVMs |
| Landlock does not constrain already-opened resources; ABI features differ | [Linux Landlock docs](https://cdn.kernel.org/doc/html/latest/userspace-api/landlock.html) | Apply sandbox before opening repository/secrets; ABI detection and deny fallback |
| `pull_request_target` plus untrusted checkout/execution can expose write token/secrets | [GitHub `pull_request_target` guidance](https://docs.github.com/en/actions/reference/security/securely-using-pull_request_target) | Split untrusted/privileged jobs; scan workflows; treat artifacts as data |
| Self-hosted/reused CI runners can expose credentials/state | [GitHub Actions secure use](https://docs.github.com/en/actions/reference/security/secure-use) | Ephemeral runner, no cross-run state, runner isolation and cleanup tests |
| GitHub coding-agent firewall has coverage limits | [GitHub firewall docs](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-the-firewall) | Inventory setup, MCP, agent, model, integration egress paths; test each separately |
| Git worktrees isolate working directories but branch/base/cleanup semantics matter | [Git worktree](https://git-scm.com/docs/git-worktree.html), [Claude worktrees](https://code.claude.com/docs/en/worktrees) | One writer/worktree, pinned base, user-change test, cleanup/orphan test |
| `git apply --check` checks applicability; unsafe paths can be explicitly overridden | [Git apply](https://git-scm.com/docs/git-apply) | Never allow unsafe-path override; add canonical path/type/scope/digest validation |
| MCP tool annotations are untrusted without trusted server provenance | [MCP 2026-07-28 tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) | Approved registry, pinned server/tool version, independent effect classification |
| Compaction can omit critical handoff detail; transformed messages need external storage for recovery | [Anthropic long-running harness](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents), [managed-agent architecture](https://www.anthropic.com/engineering/managed-agents) | Checkpoint preservation contract, raw lineage, clean-handoff and repeated-compaction evals |
| IDE workspace edits have document/workspace semantics but no coding-agent authorization boundary | [VS Code API](https://code.visualstudio.com/api/references/vscode-api), [LSP 3.18](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.18/specification/) | Versioned-buffer preconditions, user-visible diff, post-edit hash/Git verification |
| Code-host change-request and protection semantics vary and may be asynchronous | [GitLab merge-request API](https://docs.gitlab.com/api/merge_requests/), [GitLab protected branches](https://docs.gitlab.com/user/project/repository/branches/protected/), [Bitbucket pull-request API](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-pullrequests/) | Provider capability probe, effective-policy query, postcondition reconciliation, real-provider conformance suite |
| Background agent sessions have product-specific hard time limits | [GitHub cloud agent](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent) | Break tasks/checkpoint; do not assume managed product is a durable general worker |
| Public static coding tasks age and can be contaminated | [SWE-bench-Live paper](https://arxiv.org/abs/2505.23419), [SWE-rebench paper](https://openreview.net/pdf/79c762b6e65956cefc54dd56641c229683f923e7c) | Recent private/rotating held-out tasks; record task dates; leakage checks |
| Harness/interface materially affects coding performance | [SWE-agent paper](https://papers.neurips.cc/paper_files/paper/2024/file/5a7c947568c1b1328ccc5230172e1e7c-Paper-Conference.pdf) | Version and compare complete harness, not model alone |
| Terminal tasks remain difficult despite dedicated environments/tests | [Terminal-Bench 2.0](https://openreview.net/pdf?id=a7Qa4CcHak) | Terminal/process tasks in private eval; no inference from code generation alone |

An official report documents intended behavior or an observed vendor workload, not universal prevalence. The blueprint converts each into a boundary or adversarial test rather than asserting every implementation fails identically.

## Production measurements used cautiously

- Anthropic reports an 84% permission-prompt reduction in its internal Claude Code sandbox usage. This motivates reducing approval fatigue through containment, not an SLO for other products.
- Anthropic's March 2026 long-running application report compared a roughly 20-minute, $9 solo run with a roughly six-hour, $200 multi-agent harness on one application prompt, and a later run cost about $125. This demonstrates capability/cost sensitivity to harness design, not a default topology or expected budget.
- GitHub's current product documents a hard 59-minute cloud-agent session limit. This is a current managed-product constraint, not a general coding-agent runtime limit.
- Terminal-Bench 2.0 reports frontier agents resolving fewer than 65% of its 89 tasks at publication. This supports maintaining difficult terminal/process evals, not extrapolating a specific model's production success.
- Firecracker documents a high microVM creation rate in a minimal configuration, but actual coding-agent cold start includes image, repository, dependencies, networking, and toolchain setup; the guide does not reuse the headline rate as capacity guidance.

## Claims deliberately excluded or narrowed

- **“The agent safely executes arbitrary code in Docker.”** Narrowed to a configured threat model; containers share the host kernel and mounts/network/credentials dominate exposure.
- **“Human approval makes the command safe.”** Rejected. Approval is intent evidence; containment and authorization limit damage.
- **“Read-only commands are safe.”** Rejected as universal. Reads can expose secrets, execute helpers/startup behavior, exhaust resources, or feed exfiltration.
- **“Allowed command names enforce security.”** Rejected. Interpreters/build/tests/scripts can express arbitrary behavior.
- **“A worktree includes the user's current work.”** Rejected. A worktree starts from a Git object/ref; uncommitted state needs explicit handling.
- **“A signed commit is trustworthy code.”** Narrowed to verified signing identity/integrity.
- **“SLSA/in-toto proves the patch is correct.”** Narrowed to verifiable production provenance and supply-chain policy.
- **“Tests passed, therefore the task is complete.”** Rejected without final-patch binding, oracle quality, policy checks, and limitations.
- **“Self-review is independent review.”** Rejected when it uses the same model/context/evidence path; useful only as advisory or an additional layer.
- **“Checkpointing gives exactly-once tool execution.”** Rejected without destination idempotency/transactions/reconciliation.
- **“MCP annotations authorize a tool.”** Rejected; current MCP guidance says annotations are untrusted unless the server is trusted.
- **“More tools/context/agents improve capability.”** Kept conditional on private evals; each expands context, cost, selection error, and attack surface.
- **“Public SWE-bench score predicts our acceptance rate.”** Rejected; repository/language/task/harness/security/CI contracts differ.
- **“A custom service is the production-grade choice.”** Rejected as default; it is justified only by requirements a simpler surface cannot satisfy.
- **“Provider/harness portability means semantic equivalence.”** Rejected; tool, context, retry, approval, and session behavior differs.
- **“Conversation, compaction, and long-term memory can serve as run state.”** Rejected; each is derived or scoped differently and none establishes effect completion, approval, budgets, or workspace revision.
- **“A successful past run is a reusable coding procedure.”** Rejected until downstream outcome is observed, reviewed, privacy-cleared, and promoted into a versioned eval or playbook.
- **“A generic `protected branch` or `create PR` abstraction has identical security semantics across code hosts.”** Rejected; adapters must preserve provider-native policy, capability limits, eventual consistency, and receipts.

## Primary and direct sources reviewed

### Git and patch/worktree semantics

- Git, [git-worktree](https://git-scm.com/docs/git-worktree.html), checked 2026-08-31.
- Git, [git-status](https://git-scm.com/docs/git-status.html), checked 2026-08-31.
- Git, [git-apply](https://git-scm.com/docs/git-apply), checked 2026-08-31.
- Git, [git-diff](https://git-scm.com/docs/git-diff.html), checked 2026-08-31.
- Git, [git-verify-commit](https://git-scm.com/docs/git-verify-commit/2.50.0.html), checked 2026-08-31.
- Git, [git-interpret-trailers](https://git-scm.com/docs/git-interpret-trailers), checked 2026-08-31.

### Managed/local coding-agent products and developer surfaces

- OpenAI, [current model guidance](https://developers.openai.com/api/docs/guides/latest-model), checked 2026-08-31.
- OpenAI, [Codex use cases](https://developers.openai.com/codex/use-cases), checked 2026-08-31.
- GitHub, [About Copilot cloud agent](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent), checked 2026-08-31.
- GitHub, [Risks and mitigations for Copilot cloud agent](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/risks-and-mitigations), checked 2026-08-31.
- GitHub, [Manage agent sessions](https://docs.github.com/en/copilot/how-tos/copilot-on-github/use-copilot-agents/manage-and-track-agents), checked 2026-08-31.
- GitHub, [Configure the coding-agent environment](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/customize-the-agent-environment), checked 2026-08-31.
- GitHub, [Coding-agent firewall](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-the-firewall), checked 2026-08-31.
- GitHub, [Repository custom instructions](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions-in-your-ide/add-repository-instructions-in-your-ide), checked 2026-08-31.
- GitHub, [Configure MCP servers](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/configure-mcp-servers), checked 2026-08-31.
- GitHub, [Agentic Workflows](https://docs.github.com/en/copilot/concepts/agents/about-github-agentic-workflows), checked 2026-08-31.
- VS Code, [AI security](https://code.visualstudio.com/docs/agents/run/security), checked 2026-08-31.
- VS Code, [Approvals and permissions](https://code.visualstudio.com/docs/agents/run/approvals), checked 2026-08-31.
- VS Code, [Workspace Trust](https://code.visualstudio.com/docs/editing/workspaces/workspace-trust), checked 2026-08-31.
- VS Code, [Extension API: `WorkspaceEdit` and workspace trust state](https://code.visualstudio.com/api/references/vscode-api), checked 2026-08-31.
- Microsoft, [Language Server Protocol 3.18](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.18/specification/), checked 2026-08-31.
- Anthropic, [Claude Code sandboxing docs](https://code.claude.com/docs/en/sandboxing), checked 2026-08-31.
- Anthropic, [Claude Code security docs](https://code.claude.com/docs/en/security), checked 2026-08-31.
- Anthropic, [Claude Code worktrees](https://code.claude.com/docs/en/worktrees), checked 2026-08-31.
- Anthropic, [Claude Code hooks](https://code.claude.com/docs/en/hooks-guide), checked 2026-08-31.
- Anthropic, [Claude Code agents/parallel approaches](https://code.claude.com/docs/en/agents), checked 2026-08-31.
- Anthropic, [Claude Code subagents](https://code.claude.com/docs/en/sub-agents), checked 2026-08-31.
- Anthropic, [Beyond permission prompts](https://www.anthropic.com/engineering/claude-code-sandboxing), published 2025-10-20.
- Anthropic, [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents), published 2025-11-26.
- Anthropic, [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps), published 2026-03-24.
- Anthropic, [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents), published 2025-09-29.
- Anthropic, [Scaling Managed Agents: decoupling the brain from the hands](https://www.anthropic.com/engineering/managed-agents), published 2026-04-08.
- Anthropic, [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents), published 2026-01-09.
- Google, [Jules FAQ/security](https://jules.google/docs/faq/), checked 2026-08-31.
- Google, [Jules tasks and repositories](https://jules.google/docs/tasks-repos), checked 2026-08-31.
- Google, [Jules API](https://jules.google/docs/api/reference/), checked 2026-08-31.

### Open coding harnesses and interfaces

- OpenHands, [Runtime architecture](https://docs.openhands.dev/openhands/usage/architecture/runtime), checked 2026-08-31.
- OpenHands, [Docker sandbox](https://docs.openhands.dev/sdk/guides/agent-server/docker-sandbox), checked 2026-08-31.
- OpenHands, [Process sandbox](https://docs.openhands.dev/openhands/usage/sandboxes/process), checked 2026-08-31.
- OpenHands, [FAQ/safety and storage](https://docs.openhands.dev/overview/faqs), checked 2026-08-31.
- Wang et al., [OpenHands: An Open Platform for AI Software Developers](https://arxiv.org/abs/2407.16741), 2024.
- Yang et al., [SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering](https://papers.neurips.cc/paper_files/paper/2024/file/5a7c947568c1b1328ccc5230172e1e7c-Paper-Conference.pdf), NeurIPS 2024.
- Aider, [supported languages/repository-map grammar requirements](https://aider.chat/docs/languages.html), checked 2026-08-31.

### Isolation and execution

- Docker, [Rootless mode](https://docs.docker.com/engine/security/rootless/), checked 2026-08-31.
- Docker, [Seccomp profiles](https://docs.docker.com/engine/security/seccomp/), checked 2026-08-31.
- gVisor, [What is gVisor?](https://gvisor.dev/docs/), checked 2026-08-31.
- gVisor, [Security model](https://gvisor.dev/docs/architecture_guide/security/), checked 2026-08-31.
- gVisor, [Production guide](https://gvisor.dev/docs/user_guide/production/), checked 2026-08-31.
- Firecracker, [Design](https://github.com/firecracker-microvm/firecracker/blob/main/docs/design.md), checked 2026-08-31.
- Firecracker, [Production host setup](https://github.com/firecracker-microvm/firecracker/blob/main/docs/prod-host-setup.md), checked 2026-08-31.
- Linux kernel, [Landlock unprivileged access control](https://cdn.kernel.org/doc/html/latest/userspace-api/landlock.html), checked 2026-08-31.

### CI, identity, and software supply chain

- GitHub, [Securely using `pull_request_target`](https://docs.github.com/en/actions/reference/security/securely-using-pull_request_target), checked 2026-08-31.
- GitHub, [Actions secure use](https://docs.github.com/en/actions/reference/security/secure-use), checked 2026-08-31.
- GitHub, [Protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches), checked 2026-08-31.
- GitHub, [Credential types](https://docs.github.com/en/organizations/managing-programmatic-access-to-your-organization/github-credential-types), checked 2026-08-31.
- GitHub, [Artifact attestations](https://docs.github.com/en/actions/how-tos/secure-your-work/use-artifact-attestations/use-artifact-attestations), checked 2026-08-31.
- GitLab, [Merge requests API](https://docs.gitlab.com/api/merge_requests/), checked 2026-08-31.
- GitLab, [Protected branches](https://docs.gitlab.com/user/project/repository/branches/protected/), checked 2026-08-31.
- GitLab, [CI/CD pipelines](https://docs.gitlab.com/ci/pipelines/), checked 2026-08-31.
- Atlassian, [Bitbucket Cloud pull-request API](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-pullrequests/), checked 2026-08-31.
- Atlassian, [Bitbucket Cloud branch permissions](https://support.atlassian.com/bitbucket-cloud/docs/use-branch-permissions/), checked 2026-08-31.
- NIST, [SP 800-218 Secure Software Development Framework v1.1](https://csrc.nist.gov/pubs/sp/800/218/final), published 2022-02.
- SLSA, [Build provenance](https://github.com/slsa-framework/slsa/blob/main/spec/build-provenance.md), checked 2026-08-31.
- in-toto, [Getting started and integrity model](https://in-toto.io/docs/getting-started/), checked 2026-08-31.
- OpenSSF, [Scorecard GitHub Action](https://github.com/ossf/scorecard-action), checked 2026-08-31.

### Agent/tool security and protocol

- NIST, [AI 100-2 E2025: Adversarial Machine Learning](https://csrc.nist.gov/pubs/ai/100/2/e2025/final), published 2025.
- OWASP, [Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/), checked 2026-08-31.
- GitHub Security Lab, [Safeguarding VS Code against prompt injections](https://github.blog/security/vulnerability-research/safeguarding-vs-code-against-prompt-injections/), published 2025.
- MCP maintainers, [2026-07-28 specification release](https://blog.modelcontextprotocol.io/posts/2026-07-28/), published 2026-07-28.
- MCP, [2026-07-28 tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools), checked 2026-08-31.
- MCP, [2026-07-28 authorization specification](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization), checked 2026-08-31.

### Evaluation and benchmark design

- Jimenez et al., [SWE-bench: Can Language Models Resolve Real-World GitHub Issues?](https://arxiv.org/abs/2310.06770), ICLR 2024.
- Yang et al., [SWE-agent](https://arxiv.org/abs/2405.15793), NeurIPS 2024.
- Wei et al., [SWE-bench Goes Live](https://arxiv.org/abs/2505.23419), 2025.
- Ruan et al., [SWE-rebench](https://openreview.net/pdf/79c762b6e65956cefc54dd56641c229683f923e7c), 2025.
- The Terminal-Bench team, [Terminal-Bench 2.0](https://openreview.net/pdf?id=a7Qa4CcHak), ICLR 2026.
- Terminal-Bench, [continuous benchmark repository](https://github.com/harbor-framework/terminal-bench), checked 2026-08-31.
- OpenTelemetry, [GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/), checked 2026-08-31.

## Open questions

1. Which currently shipping native sandbox configurations provide equivalent filesystem/network guarantees across macOS, Linux, WSL2, and Windows without silent fallback?
2. What independent escape and exfiltration results exist for coding-agent workloads under hardened containers, gVisor, and microVMs with comparable toolchains?
3. How should patch provenance interoperate with in-toto/SLSA while preserving private prompt/tool metadata and repository ACLs?
4. What is the best portable approval/effect schema across local tools, MCP, code-host integrations, and durable workflow engines?
5. Which private task-curation techniques best separate harness improvements from model contamination while remaining maintainable across many languages?
6. How should final-patch check receipts represent flaky, nondeterministic, distributed, browser/emulator, and hardware-dependent validation?
7. Can an independent cheaper reviewer model improve escaped-defect rate without adding correlated false confidence or excessive review noise?
8. Which repository-map/index strategy provides the best quality/cost across monorepos while preserving access, freshness, and provenance?
9. How should long-running coding agents migrate active state across tool/harness/policy versions without replaying unsafe effects?
10. What practical semantic merge/integration strategies work for parallel agent patches beyond file-level worktree isolation?
11. How should organizations quantify human approval/review fatigue and choose a safe diff-size/autonomy threshold by risk class?
12. Which MCP 2026-07-28 extensions/tool semantics become safe and useful for coding agents after server admission and effect-policy evaluation?

## Refresh triggers

- any new OpenAI/Codex, GitHub coding-agent, VS Code agent, Claude Code, Jules, or OpenHands security/runtime architecture that changes isolation, branch, approval, session, or tool guarantees;
- a Git/MCP/SLSA/in-toto/OpenTelemetry release that changes the referenced machine/provenance/tool/event contract;
- a GitHub, GitLab, Bitbucket, VS Code, or LSP change that alters edit versioning, change-request APIs, branch policy, CI status, token scope, or reconciliation behavior;
- a Docker/gVisor/Firecracker/Landlock security change, escape, or support shift affecting the isolation matrix;
- a major GitHub Actions security change involving untrusted pull requests, artifact trust, tokens, runners, or agentic workflows;
- a public coding benchmark deprecation, contamination audit, new live/private benchmark, or harness-controlled comparative study;
- independent evidence that contradicts the current approval, sandbox, multi-agent, or context recommendations;
- a production incident involving agent prompt injection, source/secret leakage, configuration persistence, wrong-branch publication, duplicate effect, cross-tenant state, or rollback;
- new regulatory/contractual requirements for source-code processing, model retention/training, provenance, or human review;
- scheduled volatile-source recheck by 2026-11-30.

## Research saturation statement

Additional searches were no longer changing the architecture's core boundaries: untrusted repository/model, isolated one-writer workspace, deterministic policy, secretless execution, exact patch/check/approval binding, separate integration identity, protected human-reviewed merge, durable effect state, and private repeated evals. Remaining uncertainty is concentrated in fast-moving product features, comparative sandbox evidence, current benchmark validity, and organization-specific data/operational requirements; these are recorded as refresh triggers and open questions rather than hidden assumptions.

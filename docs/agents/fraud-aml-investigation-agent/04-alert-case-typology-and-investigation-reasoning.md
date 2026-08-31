# Alerts, Cases, Typologies, Hypotheses, and Decisions

> **Purpose:** Turn noisy alerts into bounded, reviewable investigations without treating a trigger, typology, model score, filing, or disposition as proof.  
> **Section index:** [Fraud and AML investigation agent](README.md)

## Alert, case, and outcome are different

An **alert** records why a monitoring control asked for review. A **case** is the institution's durable investigation process, which may group, split, supersede, or reopen alerts. A **decision** records what an accountable person decided using an identified evidence/policy snapshot. An **outcome** is later information with known provenance and confidence; it may still be incomplete.

Do not use “filed SAR/STR,” “no filing,” or “case closed” as a direct positive/negative misconduct label. The FFIEC manual describes suspicious-activity decisions as judgment-based and expects the final decision and rationale to be documented; it does not require the institution to establish the underlying crime. This makes disposition valuable workflow evidence but weak ground truth for detection evaluation.

## Alert admission contract

~~~json
{
  "alert_id": "alert_01...",
  "producer": "monitoring-service",
  "producer_version": "rulepack-2026.08+sha256:...",
  "semantic_key": "tenant|rule|subject|window|revision",
  "alert_family": "rapid-movement",
  "triggered_at": "...",
  "evaluation_window": {"from": "...", "to": "..."},
  "subject_refs": ["party_...", "acct_..."],
  "trigger_facts": [{"feature": "...", "value": "...", "source_refs": ["df_..."]}],
  "threshold_profile": "...",
  "jurisdiction_candidates": ["..."],
  "priority_inputs": {"deadline": "...", "harm": "...", "confidence": "..."},
  "coverage": {"complete": true, "warnings": []},
  "supersedes": null
}
~~~

Admission validates producer identity and version, recomputes or verifies trigger facts, applies semantic deduplication, links related open cases under explicit policy, assigns a jurisdiction/deadline owner, and records coverage. It must not let the agent create new high-priority alerts by writing prose.

## Fraud and AML case variants

Fraud and AML investigations may share identities, transactions, devices, counterparties, and networks, but they ask different questions and operate on different clocks. Keep separate case purposes, decision vocabularies, data permissions, customer communications, and effects; link cases by approved evidence references rather than merging them.

| Case family | Primary investigation question | Time-sensitive evidence | Decision/effect boundary |
|---|---|---|---|
| Account takeover / unauthorized access | Was access or payment behavior inconsistent with the authorized user, and which events may be compromised? | Authentication, device/session binding, credential/reset events, transaction lifecycle, customer report | SOC owns cyber-compromise forensics; authorized fraud/payment teams own customer contact, payment control, reimbursement/claim processes |
| Unauthorized card/payment fraud | Which transactions are disputed or unauthorized under the applicable product/rail process? | Authorization, authentication, clearing, merchant, token/device, reversal/chargeback and customer evidence | Agent may assemble a timeline; it does not decide liability, reimbursement, chargeback, or merchant action |
| Authorized scam/social engineering | Was the customer induced to authorize value transfer, and where did funds move? | Customer report, channel interactions, payee changes, warnings, payment status, beneficiary network | Preserve urgency and victim-sensitive language; payment recovery/contact/disclosure remains a separately authorized workflow |
| Mule/pass-through network | Are accounts facilitating movement for others, and what benign business/activity explanations remain? | Ownership/control, incoming/outgoing timing, counterparties, devices, cash/crypto/payment rails, expected activity | Candidate network and AML/fraud referral only; no guilt-by-association or automatic offboarding |
| AML monitoring / suspicious activity | Does the reviewed activity and context warrant an institutionally defined escalation or filing decision? | CDD/KYC, transaction and ownership history, typology evidence, source-of-funds/wealth where relevant | Qualified AML/legal process owns disposition and SAR/STR decision |
| Sanctions candidate | Is the party/property/transaction a valid match under an applicable program, ownership rule, license or exemption? | Official list snapshot, identity comparison, ownership/control, exact transaction state and deadline | Separate sanctions authority and effect path; a fraud/AML risk score is irrelevant to the legal match decision |

Do not let a customer report, chargeback, confirmed compromise, filed SAR/STR, or account closure automatically label every connected party or transaction. Preserve the exact product/process outcome and its scope. If cyber evidence is needed, use the security-investigation handoff defined in the category boundary instead of expanding this agent into endpoint or malware forensics.

## Typology packs are versioned investigation aids

A typology pack should contain:

| Field | Requirement |
|---|---|
| Identity | Stable name, version, owner, approval, effective interval, jurisdictions, alert families |
| Intent | Behavior to investigate and why it matters; avoid asserting a predicate offense |
| Indicators | Individually cited signals, expected direction, temporal/rail/entity scope |
| Benign explanations | Common legitimate patterns and evidence that would support them |
| Follow-up queries | Typed tools, prerequisites, maximum scope, cost and data classes |
| Contradictions | Evidence that weakens or makes the typology inapplicable |
| Stop/escalation | Missing coverage, deadline, harm, sanctions potential, confidentiality, authority boundary |
| Limitations | Base rate, geographic/language limits, evasion/adaptation, data dependencies |
| Evaluation | Positive scenarios, benign negatives, counterfactuals, slice thresholds, known gaps |
| Change control | Source/advisory references, reviewer, validation evidence, retirement trigger |

Typology text is untrusted domain data, not an instruction channel. External advisories may describe red flags, but a red flag alone does not establish suspicious activity. Pin the pack version used for each case and do not silently inject a newly published typology into an active review.

## Bounded investigation loop

~~~mermaid
stateDiagram-v2
    [*] --> Frame
    Frame --> Retrieve: approved evidence gaps
    Retrieve --> Verify: typed results
    Verify --> Retrieve: partial / stale / contradiction
    Verify --> Compare: coverage sufficient
    Compare --> Retrieve: bounded discriminating query
    Compare --> Draft: completion rule met
    Draft --> HumanReview
    HumanReview --> Retrieve: return for evidence
    HumanReview --> Decided: accountable disposition
    Frame --> Escalated: authority / deadline / harm
    Retrieve --> Paused: source unavailable / budget
    Paused --> Retrieve: resume from durable state
    Decided --> [*]
    Escalated --> [*]
~~~

### Step 1 — frame

Create a concise plan with the subject, alert basis, time horizon, jurisdiction profile, required sources, available coverage, deadlines, authority tier, and budgets. Generate at least one plausible benign explanation and one falsifying question. Do not request evidence until purpose and scope are valid.

### Step 2 — retrieve

Choose from allowlisted, typed, read-only operations. Prefer a query that discriminates between live hypotheses over broad collection. Enforce per-tool, per-source, token, graph, wall-clock, and monetary budgets.

### Step 3 — verify

Check identity, source/revision, event and observation time, coverage, pagination, transformations, contradictions, and applicability. Tool success means only that an adapter returned; it does not mean the evidence is complete or the claim is true.

### Step 4 — compare

Update the hypothesis ledger. Seek disconfirming and exculpatory evidence. Replan only when new evidence materially changes the case, a required source fails, the jurisdiction changes, the case version advances, or the next action crosses an authority boundary.

### Step 5 — draft or stop

Stop when the supported alert-family completion rule is met, the next query has low discriminating value, a budget is exhausted, a deadline or harm trigger requires human attention, or a required source/identity cannot be resolved. Produce a review package—not a verdict.

## Hypothesis contract

~~~json
{
  "hypothesis_id": "hyp_01...",
  "case_id": "case_01...",
  "case_version": 17,
  "statement": "Account A may be acting as a rapid pass-through account during the reviewed window.",
  "status": "unresolved",
  "typology_ref": "rapid-movement@5",
  "supports": [{"claim": "...", "evidence_refs": ["df_..."], "strength": "moderate"}],
  "contradicts": [{"claim": "...", "evidence_refs": ["ev_..."], "strength": "weak"}],
  "benign_alternatives": [{"statement": "Treasury concentration activity", "evidence_needed": ["..."]}],
  "gaps": [{"source": "...", "reason": "unavailable", "material": true}],
  "uncertainty": {"level": "high", "reasons": ["entity candidate unresolved"]},
  "next_discriminating_queries": ["..."],
  "created_by": {"type": "agent", "run_id": "run_..."},
  "review_state": "draft"
}
~~~

Use calibrated qualitative language only if teams can apply it consistently; otherwise expose the underlying evidence and limitations. Never fabricate a numeric probability from model confidence. An hypothesis may be rejected without deleting its lineage.

## Claim-to-evidence matrix

The review UI and export should make this matrix inspectable:

| Claim | Type | Support | Contradiction | Coverage/limits | Reviewer state |
|---|---|---|---|---|---|
| Funds moved rapidly after receipt | Derived fact | Transaction window refs | Reversal/settlement refs | One rail unavailable | Verified / disputed |
| Parties may be related | Identity candidate | Name/address/registry refs | Different verified identifiers | Transliteration ambiguity | Unresolved |
| Activity is inconsistent with expected use | Hypothesis | KYC/use and peer facts | Seasonal business evidence | KYC last refreshed at date | Draft |
| Filing threshold is met | Human/legal decision | Entire approved bundle | Recorded dissent | Jurisdiction profile/version | Authorized decision only |

Every narrative sentence that materially affects disposition should map to one or more rows. Unsupported language is removed or marked as an explicit gap.

## Case state and transition rules

| State | Entry invariant | Allowed agent behavior | Exit authority |
|---|---|---|---|
| `admitted` | Valid producer, purpose, dedupe, owner, deadline | Read alert facts; propose plan | Workflow |
| `evidence_gathering` | Authorized scope and source set | Typed reads and draft hypotheses | Workflow/human |
| `ready_for_review` | Completion rule, bundle hash, gaps and contradictions | Explain draft; no new sources without reopening | Workflow |
| `in_review` | Qualified assignee and pinned case version | Answer cited questions; draft amendments | Human reviewer |
| `decision_recorded` | Disposition, rationale, approver, policy/jurisdiction version | No case mutation | Authorized human |
| `effect_pending` | Exact approved effect intent exists | No destination access | Effect worker |
| `reconciling` | Outcome is ambiguous or postcondition unverified | Explain status from ledger only | Reconciler/human |
| `closed` | Decisions/effects/reconciliation complete; retention set | Read-only authorized explanation | Workflow owner |
| `reopened` | New material evidence or approved correction | New investigation generation; preserve prior snapshot | Authorized human/workflow |

Use optimistic concurrency. A proposal against case version 17 cannot be applied to version 18 without deterministic comparison and, for material changes, fresh review.

## Human decision contract

~~~json
{
  "decision_id": "dec_01...",
  "case_id": "case_01...",
  "case_version": 21,
  "decision_type": "sar_filing",
  "value": "file",
  "jurisdiction_profile": "us-bank-2026-08@4f2c...",
  "policy_commit": "sha256:...",
  "evidence_bundle_hash": "sha256:...",
  "rationale": "...",
  "limitations": ["..."],
  "dissent_or_escalation": null,
  "decided_by": "workforce_...",
  "decided_at": "...",
  "approval_chain": ["approval_..."],
  "expires_at": "..."
}
~~~

The UI must identify agent-drafted text, allow editing without hidden prompt instructions, show evidence and contradiction links inline, require reasons for both filing and non-filing where policy requires, and prevent self-approval. The agent may summarize a recorded decision but may not present itself as the decision maker.

## Completion rules by alert family

Define a deterministic completion policy per supported family. Example:

~~~yaml
alert_family: rapid-movement
required:
  - alert_trigger_reproduced
  - transaction_lifecycle_coverage_declared
  - customer_and_account_identity_checked
  - kyc_expected_activity_revision_cited
  - material_counterparties_reviewed_within_bounds
  - at_least_one_benign_alternative_tested
  - contradictions_and_missing_sources_listed
stop_if:
  - sanctions_candidate_requires_separate_workflow
  - cross_tenant_or_wrong_purpose_detected
  - material_identity_unresolved
  - deadline_at_risk
budgets:
  max_tool_calls: 12
  max_graph_nodes: 250
  max_wall_time_seconds: 180
~~~

This is application policy, not prompt text. A “complete” package can still conclude that evidence is insufficient; completeness means the required investigation steps and gaps are visible.

## Failure modes and responses

| Failure | Detection | Response |
|---|---|---|
| Duplicate or overlapping alert | Semantic key and open-case correlation | Link/supersede under policy; preserve producer evidence |
| Narrative invents a transaction/entity | Claim ref fails schema/resolution | Reject proposal; record evaluator failure; never display as fact |
| Model anchors on alert explanation | Counterfactual/benign-alternative evaluation | Require independent trigger reproduction and disconfirming query |
| Typology drift | Performance by pack version and emerging error cluster | Freeze/retire pack; independent validation before replacement |
| New evidence after review starts | Case version/snapshot mismatch | Invalidate bundle; route back to evidence gathering or reapproval |
| Missing data treated as negative | Coverage invariant | Block affected conclusion; label gap and owner |
| Low-value looping | Repeated query signature, marginal-evidence score, budget | Stop and return current package/gaps |
| Deadline risk | Queue-age and deadline timer | Prioritize/escalate to human; do not shorten required review invisibly |
| Reviewer automation bias | Acceptance/edit/dissent patterns, blind sampling | Show source before recommendation where feasible; independent QA |

## Explicitly rejected behaviors

- “Autonomously investigate until confident.” Confidence is not a completion condition.
- Free-form chain-of-thought stored as evidence or demanded from the model. Store concise typed rationale and cited claims.
- A single suspicion score that blends data quality, legal relevance, identity confidence, and harm.
- Reusing SAR/STR filings or investigator dispositions as uncontested supervised labels.
- Letting the model set case priority, jurisdiction, filing deadline, or review eligibility without deterministic validation.
- Changing typology, prompt, retrieval, or threshold behavior mid-case without a pinned release and migration rule.

## Checklist

- [ ] Alert trigger, producer/version, dedupe key, coverage, subject, and window are reproducible.
- [ ] Each typology has benign explanations, contradictions, typed follow-ups, limits, owner, and change control.
- [ ] The loop has explicit frame/retrieve/verify/compare/stop steps and hard budgets.
- [ ] Hypotheses cite support, contradiction, gaps, alternatives, and case version.
- [ ] Completion rules are deterministic and specific to each released alert family.
- [ ] Human decisions bind exact evidence, policy, jurisdiction, case version, approval, and expiry.
- [ ] Filed/not-filed and closed/reopened states are not treated as truth labels.

## Sources and next guide

- [FFIEC BSA/AML Manual — Suspicious Activity Reporting](https://bsaaml.ffiec.gov/manual/AssessingComplianceWithBSARegulatoryRequirements/04)
- [FinCEN — Frequently Asked Questions Regarding Suspicious Activity Reporting Requirements](https://www.fincen.gov/resources/statutes-regulations/guidance/frequently-asked-questions-regarding-suspicious-activity)
- [FinCEN — red flags do not necessarily indicate suspicious activity](https://www.fincen.gov/resources/statutes-regulations/guidance/advisory-financial-institutions-e-mail-compromise-fraud)
- [FinCEN — CVC kiosk scam payments and associated illicit activity (2025)](https://www.fincen.gov/news/news-releases/fincen-issues-notice-use-convertible-virtual-currency-kiosks-scam-payments-and)
- [FATF Recommendations](https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Fatf-recommendations.html)

Next: [KYC, CDD, EDD, sanctions, and source semantics](05-kyc-cdd-edd-sanctions-and-source-semantics.md).

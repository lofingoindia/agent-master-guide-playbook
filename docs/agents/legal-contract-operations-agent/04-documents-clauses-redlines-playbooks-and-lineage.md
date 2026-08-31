# Documents, Clauses, Redlines, Playbooks, and Lineage

## The contract is a package, not a text blob

A legally relevant contract record may include the main agreement, schedules, exhibits, incorporated policies, order forms, amendments, side letters, signature pages, completion certificate, email acceptance evidence, and later notices. Preserve the exact bytes and relationships before asking a model to interpret text.

```mermaid
flowchart LR
    P["Contract package"] --> M["Main agreement v9"]
    P --> S1["Schedule 1 v3"]
    P --> S2["Security exhibit v5"]
    P --> SIG["Executed signature artifact"]
    A1["Amendment 1"] -->|"modifies"| M
    A1 -->|"replaces"| S1
    N1["Renewal notice"] -->|"acts under"| M
    M --> R1["Rendered PDF"]
    M --> T1["Extracted text"]
    M --> C1["Clause occurrences"]
```

No single extracted-text field is authoritative. Original bytes, package membership, native structure, a stable rendering, and extraction output serve different purposes.

## Artifact and version schema

```yaml
document_version:
  document_id: doc_77
  version_id: dv_9
  matter_id: mat_2026_0142
  contract_package_id: pkg_31
  role: main_agreement
  source_system: dms_1
  source_object_id: item_991
  source_version_id: "9.0"
  original_filename: supplier-msa-redline.docx
  media_type: application/vnd.openxmlformats-officedocument.wordprocessingml.document
  byte_length: 483201
  sha256: sha256:...
  acquired_at: 2026-08-19T09:11:00Z
  acquired_by: acct_633
  parent_version_ids: [dv_8]
  relation: counterparty_redline_of
  rendered_artifact_ids: [render_pdf_42, render_pages_42]
  extraction_id: ext_77
  completeness:
    referenced_artifacts: [schedule_1, schedule_2]
    unresolved_references: [security_policy_url]
  malware_scan: passed
  access_policy_id: ap_wall_18
```

Never overwrite an acquired version. A corrected extraction creates a new extraction record; a normalized file creates a derived artifact; a user-edited draft creates a new document version.

## Ingestion and rendering

Document Intelligence performs hostile-file intake, malware checks, archive expansion, OCR, structure extraction, and source-span production. Legal operations consumes that result and adds contract semantics.

The handoff must preserve:

- original digest and acquisition evidence;
- native format and any macros, embedded files, fields, comments, footnotes, headers, and tracked changes detected;
- parser, OCR, and renderer versions;
- page, paragraph, table-cell, or native-element anchors;
- extraction confidence and warnings;
- a human-viewable rendering; and
- unresolved external or incorporated references.

OOXML tracked changes cannot be reduced safely to visible paragraph text. Text may exist in insertions, deletions, moves, comments, headers, footnotes, fields, drawings, or embedded objects. Parse the package, render it with tracked-change modes, and compare the visual output. A binary digest proves which bytes were processed, not what a reviewer visually perceived.

## Clause occurrence schema

```json
{
  "clause_occurrence_id": "co_441",
  "document_version_id": "dv_9",
  "clause_type_id": "local.data_security",
  "taxonomy_mappings": [
    {"system": "sali_lmss", "term_iri": "pinned-iri", "mapping_version": "lmss_commit_x"}
  ],
  "heading": "Information Security",
  "source_spans": [
    {"artifact_id": "render_pdf_42", "page": 18, "bbox": [72, 131, 523, 688]},
    {"artifact_id": "ext_77", "paragraph_ids": ["p_190", "p_207"]}
  ],
  "normalized_text_digest": "sha256:...",
  "cross_references": ["co_120", "schedule_security_5"],
  "model_confidence": 0.87,
  "verification_status": "human_verified",
  "analysis_release_id": "ar_14"
}
```

Clauses can be split, nested, repeated, defined elsewhere, or modified by an amendment. Store an occurrence graph, not one canonical clause string.

## Version and redline lineage

Track three related but distinct diffs:

1. **Byte/native diff:** which package structures changed.
2. **Rendered diff:** what a reviewer would see in each approved display mode.
3. **Semantic proposal:** which legal or operational meaning may have changed.

Only the first two can be made substantially deterministic. The semantic result is a cited proposal for review.

```yaml
redline_edge:
  edge_id: re_19
  base_version_id: dv_8
  compared_version_id: dv_9
  relationship: counterparty_return
  selected_by: per_legalops_4
  native_diff_artifact_id: diff_ooxml_19
  rendered_diff_artifact_id: diff_pdf_19
  model_change_set_id: mcs_19
  unresolved_changes: [embedded_object_3, field_code_8]
  verified_by: per_lawyer_7
  verification_status: partial
```

Do not let filename, upload time, or provider “latest” labels decide the base. The reviewer selects or confirms the lineage edge. Branches and merges are valid; forcing a linear history hides parallel negotiations.

## Playbook contract

```yaml
playbook_rule:
  rule_id: pb_supplier_msa_12.security.4
  playbook_version_id: pb_supplier_msa_12
  contract_family: supplier_msa
  applicability:
    jurisdictions: [England and Wales]
    risk_tiers: [high, critical]
    data_classes: [confidential, personal]
  clause_type_id: local.data_security
  approved_position:
    requirement: annual_independent_assurance
    acceptable_options: [ISO_27001, SOC_2_TYPE_II]
  fallback_text_artifact_id: art_fallback_210
  deviation_severity: high
  evidence_required: [assurance_report, scope_statement]
  escalation_role: privacy_counsel
  approved_by: per_17
  effective_from: 2026-07-01
  supersedes: pb_supplier_msa_11.security.3
  legal_review_due: 2027-01-01
```

Playbooks are governed legal content. Pin the version for a run; never silently re-score an old negotiation with a new playbook. A later version creates a new assessment with lineage.

## Deviation assessment contract

```json
{
  "assessment_id": "ca_881",
  "clause_occurrence_ids": ["co_441"],
  "playbook_rule_id": "pb_supplier_msa_12.security.4",
  "status": "deviates",
  "observed_terms": [
    {"field": "assurance_frequency", "value": "on reasonable request", "citations": ["co_441:p_198"]}
  ],
  "difference": "No annual assurance commitment is stated.",
  "risk_flag": "high",
  "draft_options": [
    {"option_id": "opt_1", "text_artifact_id": "draft_552", "basis": "approved fallback"}
  ],
  "uncertainties": ["security_schedule_not_received"],
  "required_reviewer_role": "privacy_counsel",
  "model_release_id": "mr_2026_08_17",
  "decision": {"status": "pending", "decided_by": null}
}
```

The agent reports what the text says, how it differs from an approved rule, and which evidence is missing. It must not claim a clause is “market,” “enforceable,” “safe,” or “approved” unless those are separately defined, sourced, current, and decided through the proper human process.

## Comparison decision table

| Source condition | Result | Required next step |
|---|---|---|
| Exact clause and playbook match | `matched` | Reviewer sampling per risk policy |
| Text conflicts with rule | `deviates` | Cite both, assign severity from playbook, route |
| Required clause absent after complete package check | `missing` | Cite search scope and completeness evidence |
| Rule excluded by approved applicability | `not_applicable` | Record exact applicability basis |
| Cross-reference, schedule, or version unresolved | `needs_review` | Do not label missing or compliant |
| Low-confidence span or OCR warning | `needs_review` | Obtain native source or human verification |
| Prompt injection or unrelated instruction in document | `needs_review` plus security event | Treat document text as data, not instructions |

## Drafting control

Draft only into a new artifact. Bind the request to the accepted business intent, approved playbook position, exact base version, clause span, jurisdiction profile, defined terms, drafting style, and reviewer. Validate defined-term use, internal references, numbering, schedules, and format. Show a redline from the exact base. Never apply model text directly to the negotiated version or transmit it externally without D3 approval.

## Lineage and quality checks

- [ ] Original bytes, native version ID, digest, source system, actor, and acquisition time are retained.
- [ ] Package membership and unresolved references are explicit.
- [ ] Native, rendered, and extracted artifacts are distinct and linked.
- [ ] Tracked changes, comments, fields, headers, footnotes, tables, and embedded content are inspected.
- [ ] Clause occurrences cite exact source spans and analysis release.
- [ ] Redline base and target are human-confirmed; branches are preserved.
- [ ] Playbook version, applicability, effective dates, approvals, and supersession are pinned.
- [ ] Drafts are new artifacts and render correctly against the exact base.
- [ ] Every assessment can be reproduced without relying on chat history.

## Key sources

- [ECMA-376 Office Open XML, fifth edition](https://ecma-international.org/publications-and-standards/standards/ecma-376/)
- [W3C PROV overview](https://www.w3.org/TR/prov-overview/)
- [SALI LMSS repository and release warning](https://github.com/sali-legal/LMSS)
- [OASIS LegalXML eContracts 1.0 committee specification](https://docs.oasis-open.org/legalxml-econtracts/CS01/legalxml-econtracts-specification-1.0.html)
- [Accord Project template documentation](https://docs.accordproject.org/docs/accordproject-template/)


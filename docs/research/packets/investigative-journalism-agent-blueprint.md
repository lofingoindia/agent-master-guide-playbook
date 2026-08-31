# Investigative Journalism and Source Verification Agent Blueprint: Research Packet

| Field | Value |
|---|---|
| Status | Pass 2 blueprint promoted; implementation remains newsroom-, provider-, contract-, threat-, and jurisdiction-specific |
| Research cut-off | 2026-08-31 |
| Research method | Primary standards, official documentation, official repositories, current legal-access resources, and benchmark papers; secondary commentary was used only to discover primary material |
| Companion blueprint | [Investigative Journalism and Source Verification Agent](../../agents/investigative-journalism-agent/README.md) |
| Category boundary | Investigative source and evidence intake, verification, reconciliation, claim support, editorial handoff, and post-publication evidence |
| Explicit exclusions | Autonomous publication; legal, defamation, or public-interest decisions; covert access or surveillance; deception; source exposure; replacement of reporters, editors, security staff, records specialists, or counsel |

This packet records the evidence and judgment behind the blueprint. It is not a list of links and it is not newsroom policy. A deploying organization must replace every sample threshold with approved editorial, security, records, and legal policy.

## Research question and promotion decision

The research question was:

> Can a narrowly bounded agent materially improve investigative evidence work while preserving source confidentiality, provenance, uncertainty, human editorial authority, and correction accountability?

The answer is **yes, with a hybrid architecture and a deliberately small authority envelope**. Deterministic services must own identity, evidence custody, durable state, policy, approvals, effects, and publication packages. A bounded model loop can propose searches, extractions, entity matches, timeline candidates, contradictions, and claim-support mappings. It cannot decide that an allegation is true, decide that publication is lawful or in the public interest, contact a source without approval, retrieve through illegal or deceptive means, or publish.

### Promotion scorecard

| Criterion | Result | Evidence |
|---|---:|---|
| Real-agent fit | Strong | Investigations require adaptive search, tool choice, hypothesis revision, contradiction discovery, and heterogeneous evidence handling |
| Distinct architecture | Strong | A split identity plane, evidence plane, claim plane, and editorial/export plane is materially different from generic research agents |
| Buildability | Strong | Deterministic capture, hashing, OCR, metadata parsing, archive formats, and structured ledgers have mature building blocks |
| Production depth | Strong | Durable execution, source compartmentation, approval receipts, isolation, correction propagation, SLOs, and incident response are first-order requirements |
| Evaluation viability | Strong | Evidence coverage, unsupported-claim rate, source leakage, contradiction recall, temporal consistency, approval compliance, recovery, and correction propagation are testable |
| Evidence depth | Strong | Journalism standards, source-protection guidance, provenance specifications, archival standards, records law, and media-forensics benchmarks provide a broad primary base |
| Reader value | Strong | The material exposes category-specific failure modes that a generic deep-research blueprint does not cover |

**Decision:** promote to a multi-guide blueprint. A single article would collapse security, epistemic state, evidence custody, editorial authority, and operations into an unsafe checklist.

## Research process and saturation

Research proceeded in seven passes:

1. Establish the category seam using the playbook registry and canonical runtime, state, tools, context, memory, security, reliability, evaluation, and operations guides.
2. Compare newsroom standards for accuracy, anonymity, attribution, fairness, corrections, and AI use.
3. Study source-protection threat models, legal variability, pseudonymisation, compartmentation, and operational security.
4. Study evidence capture, archival limitations, provenance, metadata, OCR, translation, open-source investigation, and synthetic-media detection.
5. Study public-record access, news exchange/correction semantics, durable effects, and post-publication evidence.
6. Translate findings into architecture decisions, contracts, staged gates, adversarial tests, and rejected designs.
7. Recheck volatile providers and specifications, derive adapter qualification/rights/finality tests, and walk one source-safe matter through publication ambiguity and correction.

Saturation was reached when additional searches largely repeated four findings: provenance is not truth, anonymity rules are newsroom-specific, archive/detector outputs require contextual corroboration, and human editorial/legal authority cannot be delegated. Fast-moving implementation versions remain refresh-sensitive.

## Category seam

| Neighbor | This blueprint owns | This blueprint delegates or links conceptually |
|---|---|---|
| Deep-research agent | Confidential-source boundaries, acquisition receipts, evidentiary support, contradiction ledgers, newsroom handoff, correction evidence | General web research tactics and broad literature synthesis |
| Document-intelligence agent | Evidentiary originals, derivatives, locators, authenticity/provenance, review status | Generic document classification, extraction, and enterprise ingestion |
| Security-investigation agent | Hostile submissions and newsroom/source harm | Endpoint forensics, enterprise incident response, malware eradication, or attribution |
| Content/editorial agent | Evidence-backed package inputs and publication lineage | Story framing, copy editing, audience strategy, CMS publishing, and final editorial judgment |
| Legal/compliance agent | Preserving the evidence and questions counsel needs | Legal advice, privilege determinations, defamation analysis, public-interest balancing, and litigation strategy |
| Fact-checking agent | Case-long evidence graph, source independence, contradictions, timelines, and corrections | Lightweight claim checking for already drafted content |

## Evidence-backed findings

### 1. Accuracy rules imply explicit epistemic types

Reuters says accuracy takes precedence over speed, allegations must not be portrayed as fact, sources should be cross-checked, uncertainty should be explicit, subjects should have a fair opportunity to respond, and errors should be corrected transparently. AP similarly prohibits fabrication and requires visible corrective treatment. These standards cannot be implemented with one generic `claim` string and a model confidence score.

The blueprint therefore separates:

- `allegation`: an asserted accusation whose truth is not established;
- `observation`: what a named observer, reporter, sensor, or artifact directly records;
- `source_statement`: what a source said, including ground rules and exact locator;
- `authentic_artifact`: an item established, to a stated degree, as what it purports to be;
- `corroborated_fact`: a narrowly scoped proposition approved under newsroom policy;
- `inference`: reasoning from facts or observations with alternatives and uncertainty;
- `editorial_conclusion`: a human-owned interpretive judgment;
- `publication_effect`: what was actually distributed, corrected, withheld, or retracted.

This distinction is both editorial and technical. It prevents an authentic document from being treated as proof that every statement inside it is true, and prevents repeated reporting of one allegation from masquerading as corroboration.

Primary basis: [Reuters Journalistic Standards](https://reutersagency.com/about/standards-values/), [AP News Values and Principles](https://www.ap.org/about/news-values-and-principles/), and the [2026 AP standards PDF](https://www.ap.org/wp-content/uploads/2026/02/ap-news-values-and-principles-2026.pdf).

AP’s July 2026 AI update is unusually direct about the agent boundary: AI may assist with early research, summarization, transcription, and translation, while journalists retain editorial judgment, verification, accountability, and pre-publication review. That supports an advisory evidence system, not an autonomous reporter. Primary basis: [AP updates newsroom standards for artificial intelligence](https://www.ap.org/the-definitive-source/announcements/ap-updates-newsroom-standards-for-artificial-intelligence/).

### 2. Anonymous-source thresholds do not reduce to one universal rule

Reuters permits a single anonymous source only exceptionally, with direct knowledge and special authorization. AP requires anonymous material to be factual rather than opinion, vital, otherwise unavailable, and from a reliable source with direct knowledge, with a manager aware of the identity. The European Fact-Checking Standards Network generally calls for at least two sources for a central claim but acknowledges single-source exceptions and requires anonymous claims to be corroborated by named sources or material evidence.

These positions overlap but are not identical. The agent must not embed a universal “two sources equals fact” rule. It records identity status, access basis, motive, independence, directness, ground rules, corroboration, and required approvals; the newsroom policy engine decides what is sufficient for a particular use.

Primary basis: [Reuters Journalistic Standards](https://reutersagency.com/about/standards-values/), [AP on anonymous sources](https://www.ap.org/the-definitive-source/behind-the-news/when-is-it-ok-to-use-anonymous-sources/), and the [EFCSN Code of Standards](https://efcsn.com/code-of-standards/).

### 3. Source protection is a system property, not an encryption checkbox

The Committee to Protect Journalists warns that encrypted messaging can still expose metadata and recommends examining providers, retained data, and subpoena exposure. SecureDrop’s threat model makes explicit assumptions about uncompromised source devices, workstations, administrators, and operational practices. The legal ability to resist compelled disclosure also varies by jurisdiction.

The design consequence is an isolated identity vault, blind source references in the working case graph, narrowly authorized reveal operations, separate audit visibility, endpoint and metadata risk assessment, and incident modes that stop contact or export. “Encrypted in transit” alone is not a source-protection control.

Primary basis: [CPJ Digital Safety Kit](https://cpj.org/2019/07/digital-safety-kit-journalists/), [SecureDrop threat model](https://docs.securedrop.org/en/stable/threat_model/threat_model.html), [SecureDrop mitigations](https://docs.securedrop.org/en/stable/threat_model/mitigations.html), and the [RCFP Reporter’s Privilege Compendium](https://www.rcfp.org/reporters-privilege/).

### 4. Pseudonymisation reduces risk but does not make a source anonymous

The UK Information Commissioner explains that pseudonymisation can be reversed through access to mappings or indirect identifiers, and that outliers and auxiliary data can re-identify a person. NIST likewise treats de-identification as risk reduction, not a guarantee.

The blueprint uses random opaque `source_ref` values, stores the mapping separately, removes identifying attributes from model context, controls joins, and performs mosaic-risk review before export. It never labels a source “anonymous” merely because a name was replaced.

Primary basis: [ICO pseudonymisation guidance](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/data-sharing/anonymisation/pseudonymisation/) and [NIST IR 8053](https://csrc.nist.gov/pubs/ir/8053/final).

### 5. Secure submission systems should remain outside the model’s credential boundary

SecureDrop is an end-to-end operational system with a defined threat model, journalist interfaces, workstation assumptions, and organizational procedures. Its documentation does not justify handing a model direct credentials or exposing submissions to an ordinary inference environment. The newer `securedrop-protocol` repository also labels itself proof-of-concept and not production-ready.

The blueprint treats SecureDrop or an equivalent approved system as an external high-security boundary. A trained human triages submissions; only an approved, sanitized derivative and receipt enter the agent environment. No guide assumes that the agent can install, administer, or replace SecureDrop.

Primary basis: [What Is SecureDrop?](https://docs.securedrop.org/en/stable/what_is_securedrop.html), [SecureDrop threat model](https://docs.securedrop.org/en/stable/threat_model/threat_model.html), and the [experimental SecureDrop Protocol repository](https://github.com/freedomofpress/securedrop-protocol).

### 6. Acquisition provenance must preserve raw and derived objects

NARA’s trustworthy-record guidance distinguishes reliability, authenticity, integrity, and usability and stresses content, context, structure, authorized annotations, and protection from unauthorized alteration. W3C PROV supplies interoperable concepts for entities, activities, and agents.

The blueprint therefore stores an immutable acquired object, acquisition receipt, hash, timestamp, tool/version, origin, access path, and transformation chain. OCR text, translations, thumbnails, redactions, normalized media, and model summaries are new derived artifacts with their own hashes and locators. They never overwrite the acquired bytes.

Primary basis: [NARA trustworthy electronic records guidance](https://www.archives.gov/records-mgmt/policy/electronic-signature-technology.html) and [W3C PROV-O](https://www.w3.org/TR/2013/REC-prov-o-20130430/).

### 7. A web capture is evidence of a capture, not a perfect historical website

WARC can aggregate response, request, metadata, revisit, conversion, and other records. WACZ packages WARC data with indexes and contextual metadata. RFC 7089 defines datetime negotiation for prior web states. These formats improve preservation and replay, but do not guarantee completeness.

The UK Government Web Archive warns that crawls are snapshots of what a crawler could access, not backups; dynamic content, logins, streaming media, embedded services, POST/Ajax flows, databases, and external dependencies may be missing. The agent records crawl scope, status, headers, redirects, resources, replay limitations, and the difference between capture time, page-asserted time, server time, and event time.

Primary basis: Library of Congress descriptions of [WARC](https://www.loc.gov/preservation/digital/formats/fdd/fdd000236.shtml) and [WACZ](https://www.loc.gov/preservation/digital/formats/fdd/fdd000586.shtml), [RFC 7089](https://datatracker.ietf.org/doc/html/rfc7089), and [UK Government Web Archive limitations](https://www.nationalarchives.gov.uk/webarchive/find-a-website/limitations/).

### 8. Public-record access is jurisdictional workflow, not a generic search tool

The US Department of Justice FOIA guide is updated on a rolling basis and covers procedural requirements, exemptions, processing, appeals, and litigation. UNESCO reports broad but uneven adoption of access-to-information guarantees. The Tromsø Convention establishes minimum rights and review mechanisms only for its parties and within its scope.

The agent can maintain a request plan, custodian hypothesis, scope wording, dates, fees, acknowledgements, productions, exemptions, appeals, deadlines, and receipts. It cannot infer a right of access from a universal template, practice law, evade access controls, or treat an agency’s production as complete or substantively true.

Primary basis: [DOJ Guide to FOIA](https://www.justice.gov/oip/doj-guide-freedom-information-act-0), [UNESCO Access to Information Laws](https://www.unesco.org/en/access-information-laws), and the [Tromsø Convention](https://www.coe.int/en/web/access-to-official-documents/home).

### 9. Authenticity, integrity, provenance, and truth are separate questions

An artifact can have an intact hash and still contain false statements. A valid signature can identify a signer and protect assertions from tampering without establishing that the depicted event occurred as described. Conversely, missing provenance metadata does not prove fabrication.

The evidence workflow therefore asks separately:

1. Are these the acquired bytes?
2. Is the file internally well-formed and unaltered since acquisition?
3. What origin or transformation assertions are present, and do they validate?
4. Is the asserted creator or source identity supported?
5. Does independent contextual evidence support the content and interpretation?

This prevents the frequent but unsafe collapse of “file is authentic” into “claim is true.”

Primary basis: [NARA trustworthy records guidance](https://www.archives.gov/records-mgmt/policy/electronic-signature-technology.html) and the [C2PA 2.3 technical specification](https://spec.c2pa.org/specifications/specifications/2.3/specs/C2PA_Specification.html).

### 10. C2PA Content Credentials are valuable trust signals, not truth verdicts

C2PA 2.3 describes signed assertions bound to an asset and explicitly says the specification should not make value judgments about whether provenance data is “good” or “bad.” The trust decision still depends on signer identity, credential trust, assertions, validation status, and context. Provenance can also be absent from legitimate content or lost in distribution.

The blueprint parses and preserves manifests, validation statuses, certificate and timestamp details, ingredients, actions, and the exact tool version. It reports `present_valid`, `present_invalid`, `present_untrusted`, `absent`, or `unreadable`; it never emits `real` or `fake` from C2PA alone.

Primary basis: [C2PA Specifications 2.3](https://spec.c2pa.org/specifications/specifications/2.3/index.html) and the [Content Credentials specification](https://spec.c2pa.org/specifications/specifications/2.3/specs/C2PA_Specification.html).

### 11. Metadata is an evidentiary signal with fragile custody

IPTC Photo Metadata provides structured descriptive, administrative, and rights fields; the 2025.1 standard adds fields concerning AI-generated content. Container and stream metadata can be extracted by tools such as `ffprobe`. Neither standard says that embedded values are inherently trustworthy, and platform processing may strip or rewrite metadata.

The blueprint captures original metadata before transformation, records parser/version and parse errors, compares it with external evidence, and preserves absence. It does not fill missing EXIF/IPTC fields from model guesses or treat GPS/device/time fields as dispositive.

Primary basis: [IPTC Photo Metadata Standard](https://iptc.org/standards/photo-metadata/iptc-standard/) and [ffprobe documentation](https://ffmpeg.org/ffprobe.html).

### 12. OCR, transcription, and translation create derived testimony-like text

Tesseract’s current documentation describes a versioned OCR engine with language-specific trained data and differing speed/accuracy models. That is an extraction facility, not a guarantee of faithful text. Layout, handwriting, scans, language choice, tables, and preprocessing can materially change output. Translation similarly introduces ambiguity and tone risk.

Every text span therefore points to a page/time/region locator, tool/model/version, language, confidence where meaningful, and human review state. Quoted material used in a publishable package must be checked against the original by an appropriately qualified human; uncertain text remains marked and alternatives are preserved.

Primary basis: [Tesseract user manual](https://github.com/tesseract-ocr/tessdoc) and [Tesseract release notes](https://github.com/tesseract-ocr/tessdoc/blob/main/ReleaseNotes.md).

### 13. Digital open-source investigation requires preservation, safety, and method records

The Berkeley Protocol provides a professional, legal, and ethical framework for identifying, collecting, preserving, verifying, and analyzing digital open-source information. Its scope is human-rights and international-criminal investigations, so it is not transplanted wholesale into journalism. Its preservation, safety, verification, and documentation disciplines are nonetheless directly relevant.

The blueprint adopts method logs, source discovery paths, reproducible locators, capture receipts, alternate hypotheses, and investigator-safety gates, while retaining newsroom-specific ethics and legal authority.

Primary basis: [OHCHR Berkeley Protocol record](https://searchlibrary.ohchr.org/record/30334?ln=en) and the [official protocol PDF](https://www.ohchr.org/sites/default/files/2022-04/OHCHR_BerkeleyProtocol.pdf).

### 14. Synthetic-media detectors are triage instruments with distribution-specific error

NIST identifies generalization, post-processing, anti-forensics, and the gap between research accuracy and real-world usability as central challenges. DF40 shows why narrow forgery diversity, outdated generators, and limited evaluation protocols can create misleading rankings. In-the-wild research also reports severe performance drops relative to academic benchmarks.

The blueprint permits detectors to create a versioned `forensic_signal` with threshold, calibration set, input transformation, score, and limitations. A detector cannot set an authenticity verdict, establish who manipulated content, or substitute for provenance, contextual verification, and expert review. Deployment evaluation must include unseen generators, compression, screenshots, crops, transcoding, benign editing, languages, devices, and local prevalence.

Primary basis: [NIST Guardians of Forensic Evidence](https://www.nist.gov/programs-projects/guardians-forensic-evidence), [NIST OpenMFC](https://mfc.nist.gov/), [DF40](https://proceedings.neurips.cc/paper_files/paper/2024/file/34239f60eca7ce9bee5280aaf81362d8-Paper-Datasets_and_Benchmarks_Track.pdf), and [Deepfake-Eval-2024](https://arxiv.org/abs/2503.02857).

### 15. Independent corroboration requires origin-cluster reasoning

Two URLs, two screenshots, or two interviews are not independent if they repeat one press release, anonymous post, syndication feed, common database error, or coordinated narrative. Source-count rules without origin analysis produce false confidence.

The claim ledger therefore tracks acquisition source, information origin, access path, directness, dependencies, syndication/republication links, and contradiction status. Corroboration counts origin clusters and materially independent methods, not documents. The model may propose clusters; a reporter approves consequential independence judgments.

Primary basis: the sourcing and cross-checking requirements in [Reuters standards](https://reutersagency.com/about/standards-values/) and [EFCSN standards](https://efcsn.com/code-of-standards/), synthesized with archive and provenance constraints above.

### 16. Entity and time reconciliation must preserve ambiguity

Names, transliterations, aliases, subsidiaries, shared addresses, reused identifiers, event timestamps, publication timestamps, server timestamps, and capture timestamps often conflict. Prematurely merging identities or forcing one time value can distort an investigation.

The blueprint uses candidate links, evidence-backed match features, explicit disconfirming features, valid-time and recorded-time ranges, timezone/source precision, and reversible human-approved merges. An unresolved identity or time interval is a valid result, not a model failure.

This is an engineering inference from the provenance, metadata, archival, and newsroom-accuracy sources above; no source claims that a universal entity-resolution threshold is journalistically sufficient.

### 17. Corrections and retractions are durable state transitions

AP requires corrections to be visible to both subscribers and news consumers and rejects euphemistic labels for factual corrections. Reuters calls for prompt, clear, comprehensive correction. IPTC NewsML-G2 distinguishes update/correction signaling and publishing states such as `usable`, `withheld`, and `canceled`, with strong downstream semantics.

The blueprint therefore stores publication receipts, exact claim/artifact versions used, downstream destinations, correction reason, replacement relationships, notifications, acknowledgements, and unresolved propagation. The model can assemble impacted-claim candidates; editors authorize corrective language and publication effects.

Primary basis: [AP corrections guidance](https://www.ap.org/about/news-values-and-principles/telling-the-story/), [Reuters standards](https://reutersagency.com/about/standards-values/), and [IPTC NewsML-G2 Guidelines](https://www.iptc.org/std/NewsML-G2/guidelines/).

### 18. Untrusted evidence is also untrusted instruction

Submissions, web pages, PDFs, emails, OCR text, metadata, and quoted prompts can contain instructions designed to redirect an agent, expose compartments, or trigger tools. Their journalistic relevance does not make them trusted control data.

The blueprint labels evidence-derived text, separates control and content channels, constrains tool arguments with schemas and policy, authorizes retrieval before model exposure, and prevents retrieved text from granting authority. Confidential identity never enters ordinary model context. This decision follows the playbook’s canonical security/tool boundary and is strengthened by the category’s hostile-submission threat model.

### 19. Durable execution must distinguish decisions from external effects

Retries are normal in long investigations, but a retry of a records request, source message, export, or correction can create duplicate or harmful effects. The runtime therefore uses intent records, approval records, idempotency keys, effect receipts, reconciliation, and explicit partial-effect states. A model transcript is not the source of truth.

The agent may draft an effect payload, but an authorized human and deterministic executor own sending. Publication remains outside the agent even when a CMS supports idempotency.

### 20. Human editorial and legal handoff is the product boundary

Newsroom standards place responsibility on journalists and editors. Legal privilege and source-protection rules vary, and public-interest/defamation analysis requires facts, jurisdiction, policy, and professional judgment. Therefore the system’s highest-value output is not an autonomous article; it is a reviewable package with evidence locators, claim states, contradictions, uncertainty, fairness attempts, source-protection notes, missing evidence, approvals, and correction lineage.

Primary basis: [Reuters standards](https://reutersagency.com/about/standards-values/), [AP standards](https://www.ap.org/about/news-values-and-principles/), [RCFP privilege compendium](https://www.rcfp.org/reporters-privilege/), and the [Council of Europe source-protection factsheet](https://www.echr.coe.int/documents/d/echr/fs_journalistic_sources_eng).

### 21. Provider response semantics must not become newsroom evidence semantics

Current official interfaces expose materially different objects. FOIA.gov provides component and request-form information; Regulations.gov separates documents, comments, dockets, and attachments and may delay comment publication; WordPress exposes post creation/update operations; Jira exposes workflow objects whose statuses are locally configured. An HTTP success on any of these surfaces does not mean “records request legally filed,” “complete docket,” “editor approved,” “story publicly observed,” or “correction propagated.”

The Pass 2 design therefore requires a versioned capability manifest and conformance report covering identity, scope, pagination, freshness, completeness, rights, retention, finality, authentication, rate limits, failure and reconciliation. Domain outcomes are recorded separately from provider attempts.

Primary basis: [FOIA.gov developer resources](https://www.foia.gov/developer/), [Regulations.gov API v4](https://open.gsa.gov/api/regulationsgov/), [WordPress Posts REST API](https://developer.wordpress.org/rest-api/reference/posts/), and [Jira Cloud REST API v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro).

### 22. Source protection forbids treating a secure tip product as a general agent tool

SecureDrop’s current documentation retains a hardened, human newsroom workflow. Freedom of the Press Foundation’s source-protection guidance also warns that Tor use may be observable and submitted documents can carry identifying metadata. At the cut-off, the official release channel listed SecureDrop 2.16.1, SecureDrop Workstation 1.8.0 on Qubes 4.3, and SecureDrop Inbox 1.6.0. These facts strengthen rather than relax the boundary: the model has no SecureDrop credentials, raw session, source identity, or direct import path. A trained human may export only an approved derivative with a blind reference and custody receipt.

Primary basis: [SecureDrop news/releases](https://securedrop.org/news/), [SecureDrop documentation](https://docs.securedrop.org/en/latest/), [confidential tip-page security](https://freedom.press/digisec/blog/security-confidential-tip-pages/), and [source-protection guidance](https://freedom.press/digisec/guides/source-protection/).

### 23. Seven memory lifetimes are sufficient; identity vault and procedure are governed state

The original blueprint correctly rejected free-form long-term memory but risked presenting vault continuity and procedure as additional model memory classes. Pass 2 now uses exactly seven named lifetimes: Turn/scratch, Working/run, Session, Durable workflow/task, Domain knowledge, Long-term/preference, and Episodic/outcome. Each has an explicit use, rejection rule, retention/correction/deletion behavior, and poisoning test.

Source identity and relationship state belongs in a separately authorized vault. Human-published procedure belongs in versioned domain knowledge. Neither may be silently retrieved or self-learned by the acting model.

### 24. Loss-aware compaction must be restart proof, not merely a good summary

Long matters can cross model windows, human waits, upgrades and failures. The Pass 2 continuity receipt therefore pins source/custody/review event high-watermarks; retained claim, evidence, contradiction, event and request versions; rights/retention; custody head; policy/tool/model/prompt/compiler versions; approvals/deadlines; pending and unknown effects; source-protection invariants; known losses and rebuild paths; invariant hash; and next safe action. The resume path deterministically rejects high-watermark regression, missing contradiction, stale approval, custody break, or lost ambiguous effect.

### 25. Storage immutability, provenance and custody are different controls

Amazon S3 Object Lock illustrates the distinction. Official documentation requires Versioning, applies retention/legal hold to object versions, distinguishes governance and compliance modes, and permits newer versions or delete markers even when a prior version is retained. Encryption-key loss can make retained objects unreadable. The blueprint therefore treats WORM retention as one evidence-store control, not a complete chain of custody, deletion policy, availability plan, or authenticity guarantee.

Primary basis: [S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html) and [Object Lock management](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock-managing.html).

### 26. Geocoding, transcription and translation require data-route qualification

The public OSMF Nominatim service is not a generic high-volume geocoder: its official policy limits heavy use, forbids systematic queries/autocomplete, requires identification and attribution, says not to submit confidential/personal data, and can change without notice. Google Cloud Speech-to-Text says audio/transcripts are not logged by default but offers opt-in data logging under separate terms; logged data is not deleted with project deletion. Cloud Translation’s official overview says customer data/translations are not used to improve its models, but edition, region, logs, contract and deletion still need deployment proof.

The blueprint therefore routes confidential material only through an explicitly approved local or provider path, records model/configuration, preserves aligned originals, and requires qualified review of consequential quotes, names, numbers and modality.

Primary basis: [Nominatim usage policy](https://operations.osmfoundation.org/policies/nominatim/), [OSM copyright](https://www.openstreetmap.org/copyright), [Speech-to-Text data logging](https://cloud.google.com/speech-to-text/docs/data-logging), and [Cloud Translation API overview](https://cloud.google.com/translate/docs/api-overview).

### 27. Operational recovery includes new arrivals and human bottlenecks

Restoring a database is insufficient. Recovery must re-establish authorization, events, custody heads, holds/tombstones, pending and unknown effects, object fixity and non-authoritative projections while new source-safety, correction and deadline work arrives. Capacity therefore satisfies normal arrival rate plus backlog drain within the target window, and reserves human review as well as compute. Audit, evidence/custody, traces, application logs and SLO metrics remain separate records with different sampling, content and retention rules.

This leads to component-specific RPO/RTOs and DR tests for stale indexes, lost keys, partial audit restore, provider outage, hostile artifacts and simultaneous priority backlog. Multi-region source-vault replication is rejected unless its threat, jurisdiction and recovery benefit outweigh expanded exposure.

## Resolved conflicts and limits

The research did not produce one universal standard. These conflicts are intentional configuration points.

| Question | Sources or signals in tension | Blueprint resolution |
|---|---|---|
| Is one anonymous source ever sufficient? | Reuters permits it exceptionally with special authorization; AP applies a different multi-part test; EFCSN prefers two or more sources with limited exceptions | Store evidence and source properties; call the newsroom’s versioned policy; require named approvals; never encode `source_count >= 2` as truth |
| Does valid C2PA prove truth? | C2PA validates signed provenance assertions, while editorial verification asks whether content and context are true | Treat it as a provenance signal only; preserve validation and signer context; require independent content verification |
| Does absent Content Credentials imply a fake? | Provenance can be opt-in, stripped, unsupported, or never created | Represent `absent` without a negative authenticity conclusion |
| Is an archive snapshot the state of a whole site at one moment? | WARC/WACZ preserve captures; archival guidance documents crawl gaps and asynchronous resource capture | Describe exactly what was captured, when, with what omissions; corroborate material claims elsewhere |
| Can a deepfake detector decide authenticity? | Benchmarks can show high in-distribution scores while NIST and newer datasets show generalization failure | Use detectors for calibrated triage; never as a sole verdict; require local, cross-distribution validation and expert escalation |
| Is an official record necessarily accurate? | Records may be authentic products of an agency but still incomplete, disputed, erroneous, or outside the custodian’s knowledge | Separate record authenticity, issuer, contents, and claim support; seek underlying data and contrary evidence |
| Is freedom-of-information workflow universal? | US FOIA, national access laws, and the Tromsø Convention differ in scope, exemptions, timelines, standing, and remedies | Use jurisdiction-specific policy packs reviewed by counsel/records specialists; no generic legal advice or automated filing |
| Should a corrected item be overwritten, retained, withheld, or canceled? | AP describes channel-specific visible corrections; NewsML-G2 supports versions and strong `withheld`/`canceled` semantics; records duties may require internal preservation | Preserve internal evidentiary history under retention/legal policy while emitting the newsroom-approved downstream correction or withdrawal protocol |
| Does pseudonymisation equal anonymity? | Operational shorthand often treats them alike; ICO and NIST describe re-identification risk | Keep the terms distinct; perform mosaic-risk review; isolate mappings and joins |
| Does encryption protect a source? | Encryption protects content in transit/storage; CPJ and SecureDrop document metadata, endpoints, devices, operations, and legal compulsion | Use an end-to-end source threat model with human procedures, compartmentation, endpoint controls, and jurisdictional advice |
| Is a proposed ethics code current policy? | SPJ’s August 2026 material is a proposed revision; its 2014 code remains the adopted reference until the organization says otherwise | Mark drafts as drafts, record adoption status/date, and do not silently substitute proposed text for current policy |

## Current technical and policy baseline

Versions are research anchors, not an instruction to auto-upgrade production. Pin a verified build, record hashes/configuration, evaluate it locally, and use a controlled rollout.

| Component | Current reference at cut-off | Why it matters | Upgrade warning |
|---|---|---|---|
| C2PA | Specification 2.3 | Current Content Credentials validation and assertion semantics | Parser/validator changes can alter statuses; replay the provenance corpus |
| `c2patool` | `c2patool-v0.26.60`, 2026-05-27 | Reference CLI from the official `c2pa-rs` release stream | The `0.x` API/behavior surface changes quickly; pin exact artifact and options |
| IPTC Photo Metadata | 2025.1, published 2025-11 | Adds AI-related fields and current machine-readable TechReference | New fields do not make values trustworthy; test round-trip preservation |
| IPTC NewsML-G2 | Guidelines for 2.35 | Current event and publishing-status guidance | Map newsroom correction/withdrawal policy explicitly; do not infer archival deletion duties |
| SecureDrop | Stable documentation and current supported release line | Source submission threat model and workflow boundary | Never auto-upgrade or integrate around the documented security process; use the official admin procedure |
| SecureDrop suite | SecureDrop 2.16.1; Workstation 1.8.0 on Qubes 4.3; Inbox 1.6.0 | Current official release anchors at cut-off | Recheck release/news and compatibility before maintenance; version currency is not an anonymity guarantee |
| WACZ | 1.1.1 | Current portable web-archive packaging specification used by Webrecorder | Packaging/index validity does not establish capture completeness or simultaneity |
| Tesseract | 5.x documentation; release notes list 5.5.3 on 2026-07-24 | Reproducible OCR tool/model baseline | Language data and preprocessing affect results; preserve both and rerun the OCR fixture set |
| FFmpeg/`ffprobe` | Official documentation generated from current revision | Media/container inspection | Pin deployed FFmpeg build because formats, decoders, and output can vary |
| SPJ Code | 2014 adopted code; August 2026 revision is a proposal under review | Prevents a draft from being represented as adopted ethics policy | Recheck after the revision process concludes |
| DOJ FOIA Guide | Rolling official guide | Current US federal procedural and exemption analysis | Chapters have different update dates; preserve jurisdiction and consultation date |

Version evidence: [C2PA 2.3](https://spec.c2pa.org/specifications/specifications/2.3/index.html), [`c2pa-rs` releases](https://github.com/contentauth/c2pa-rs/releases), [IPTC Photo Metadata 2025.1](https://www.iptc.org/std/photometadata/specification/IPTC-PhotoMetadata-2025.1.html), [NewsML-G2 guidelines](https://www.iptc.org/std/NewsML-G2/guidelines/), [Tesseract release notes](https://github.com/tesseract-ocr/tessdoc/blob/main/ReleaseNotes.md), [FFmpeg documentation](https://www.ffmpeg.org/documentation.html), [SPJ revision announcement](https://www.spj.org/spj-ethics-committee-proposes-revisions-to-code-of-ethics/), and the [DOJ FOIA guide](https://www.justice.gov/oip/doj-guide-freedom-information-act-0).

## Architecture decision record

| ID | Decision | Evidence and reasoning | Consequence |
|---|---|---|---|
| IJ-001 | Start with a deterministic case/evidence/claim workspace | Many workloads are capture, indexing, OCR, ledger, or review problems; adaptivity must earn its risk | Stage 0 can be the final product |
| IJ-002 | Use one bounded advisory loop only after a measured need | Adaptive research is valuable, but multiple autonomous loops multiply leakage, duplication, and reconciliation failures | Multi-agent execution is disabled by default |
| IJ-003 | Split source identity from working source records | Pseudonymisation and source-protection research show mapping and mosaic risks | The model and normal operators see blind references, not identity |
| IJ-004 | Keep secure-submission credentials and raw high-risk intake outside the agent | SecureDrop has its own threat model and operational boundary | Humans triage and sanitize before controlled import |
| IJ-005 | Make acquired bytes immutable and derivatives explicit | Records/provenance standards require content, context, structure, and transformation history | OCR, translation, redaction, and normalization cannot overwrite originals |
| IJ-006 | Treat evidence as untrusted content | Public/submitted artifacts can carry prompt injection or malicious payloads | Sandboxing, content/control separation, taint labels, and egress policy are mandatory |
| IJ-007 | Separate artifact authenticity from claim truth | C2PA/NARA and newsroom standards address different questions | No single `verified=true` field exists |
| IJ-008 | Count independent origin clusters, not URLs | Syndication and circular reporting create false corroboration | Claim support includes dependency and origin analysis |
| IJ-009 | Preserve unresolved entities, times, and contradictions | Forced certainty distorts investigative state | Candidate links and time ranges remain first-class |
| IJ-010 | Parameterize newsroom and jurisdiction policy | Anonymity, access, privilege, correction, retention, and legal rules vary | Policy packs are versioned inputs with named owners |
| IJ-011 | Require durable effect receipts and idempotency | Records requests, messages, exports, and corrections can duplicate or partially succeed | Every external effect has intent, approval, execution, and reconciliation state |
| IJ-012 | Make editorial handoff the terminal agent output | Publication and public-interest/legal judgments remain human-owned | There is no CMS publish tool in the reference boundary |
| IJ-013 | Treat corrections as new governed evidence and effects | AP, Reuters, and IPTC require visible, traceable correction semantics | Publication lineage and impact analysis persist after release |
| IJ-014 | Evaluate leakage and authority as hard gates | Average task quality cannot compensate for a disclosed source or unauthorized publication | Any protected-identity leak or unauthorized effect blocks release |

## Approaches considered and rejected

| Rejected approach | Why it appears attractive | Why it was rejected |
|---|---|---|
| “Give the model SecureDrop access and let it triage tips” | Removes a manual queue | Collapses a high-security human workflow into a broad inference boundary and increases source, malware, and prompt-injection risk |
| “Use two sources = verified” | Easy to implement and explain | Ignores common origin, directness, motive, circular reporting, and newsroom-specific exceptions |
| “One confidence score per claim” | Compact UI | Hides evidence dimensions, contradictions, policy thresholds, and reviewer disagreement; encourages automation bias |
| “Valid C2PA = real; absent C2PA = fake” | Simple provenance UX | Contrary to C2PA’s own semantics and opt-in/distribution realities |
| “Run several agents and let them vote” | Creates an appearance of independent review | Models may share the same evidence, prompt, failure modes, and hidden dependencies; voting is not source independence |
| “Put every case into one vector database” | Easy retrieval | Risks cross-matter and cross-source leakage, deletion/retention conflicts, and untraceable retrieval influence |
| “Let the model merge entities automatically” | Produces a cleaner graph | A false merge can contaminate every downstream claim, timeline, and source-safety decision |
| “Ask an AI detector whether media is fake” | Fast triage | Distribution shift, compression, new generators, and unknown calibration make a binary verdict unsafe |
| “Generate a polished story as the main output” | Demonstrates visible value | Moves attention away from evidence sufficiency, uncertainty, fairness, authority, and traceability |
| “Auto-file public-record requests from a template” | Saves reporter time | Custodian, scope, exemptions, identity, fees, jurisdiction, and legal consequences require controlled human review |
| “Overwrite corrected material” | Simplifies storage | Destroys auditability and makes downstream correction/retraction impact impossible to reconstruct |
| “Store source identity in the prompt but tell the model not to reveal it” | Minimal engineering | Instruction is not a confidentiality boundary; logs, caches, traces, tools, and outputs expand exposure |

## Evidence-to-blueprint map

| Research area | Blueprint guide |
|---|---|
| Category boundary, epistemic objects, authority, reference architecture | [README](../../agents/investigative-journalism-agent/README.md) and [mission/architecture](../../agents/investigative-journalism-agent/01-mission-authority-and-reference-architecture.md) |
| Confidential intake, identity, ground rules, source independence, records requests | [source intake and compartmentation](../../agents/investigative-journalism-agent/02-source-intake-identity-rights-and-compartmentation.md) |
| Capture, archives, OCR, media, metadata, C2PA, detector limitations | [evidence acquisition and authenticity](../../agents/investigative-journalism-agent/03-evidence-acquisition-provenance-and-authenticity.md) |
| Claim categories, origin clusters, contradictions, entities, time | [claims and reconciliation](../../agents/investigative-journalism-agent/04-claims-contradictions-entities-and-timelines.md) |
| Loop, tools, injection boundaries, context, compaction, memory | [research loop and context](../../agents/investigative-journalism-agent/05-research-loop-tools-context-and-memory.md) |
| Fairness, editorial/legal packages, publication receipts, corrections | [handoff and corrections](../../agents/investigative-journalism-agent/06-editorial-legal-handoff-corrections-and-publication-evidence.md) |
| Isolation, events/effects, recovery, SLOs, incidents, cost | [security and operations](../../agents/investigative-journalism-agent/07-security-reliability-observability-and-operations.md) |
| Metrics, adversarial suites, detector testing, release gates | [evaluation and evolution](../../agents/investigative-journalism-agent/08-evaluation-adversarial-testing-and-continuous-evolution.md) |
| Stages 0–6, contracts, gates, smallest production shape | [zero-to-production roadmap](../../agents/investigative-journalism-agent/09-zero-to-production-roadmap-and-reference-contracts.md) |
| Live-provider qualification, canonical identity/time semantics, worked source-to-correction lifecycle, exercises | [adapter qualification and worked lifecycle](../../agents/investigative-journalism-agent/10-adapter-qualification-and-worked-investigation-lifecycle.md) |

## Open questions and deployment-specific limits

1. **Jurisdiction:** source privilege, privacy, recording consent, data protection, public records, court restrictions, and defamation law vary. The blueprint supplies evidence and control boundaries, not legal conclusions.
2. **Newsroom policy:** anonymity, single-source exceptions, fairness timing, source payment, embargoes, review roles, and correction language differ. A deployment needs a signed and versioned policy pack.
3. **Threat actor:** a local public-data investigation and a cross-border national-security investigation do not share one acceptable inference, endpoint, identity, or cloud architecture.
4. **Model/provider retention:** confidential or restricted evidence must not enter a provider until contractual, technical, regional, retention, training-use, support-access, and deletion properties are verified.
5. **Archive completeness:** no crawler proves it captured every state or dependency. Material web evidence needs capture QA and, when important, alternate preservation.
6. **Media forensics:** detector error rates are local to distributions and versions. The blueprint intentionally refuses a universal threshold.
7. **Translation and transcription:** dialect, code-switching, poor audio, culturally loaded wording, and legal significance require qualified review.
8. **Source independence:** origin clustering is partly investigative judgment. A graph can surface dependencies but cannot prove independence mechanically.
9. **Corrections:** downstream partners may not acknowledge or implement a correction. The system can track propagation, not guarantee it.
10. **Retention versus minimization:** source protection favors minimization; records, litigation holds, accountability, and correction evidence may require retention. The authorized policy must resolve the conflict per matter.
11. **Biometrics:** face/voice identification raises accuracy, consent, discrimination, and legal issues beyond ordinary media verification. It is disabled in the reference design.
12. **No outcome guarantee:** an investigation may correctly end as unresolved, contradicted, unsafe to continue, or unsupported for publication.
13. **Live-provider state:** this research inspected public documentation, not a newsroom's live accounts, private contracts, support arrangements, regions, quota grants, or negotiated data-use terms. Each manifest requires deployment evidence.
14. **Editorial and legal accountability:** no adapter test or model evaluation can decide defamation, privilege, privacy, public interest, proportionality, fairness, coercion, or publication readiness. Those remain accountable professional decisions.
15. **High-risk source safety:** SecureDrop and compartmentation reduce exposure but cannot guarantee anonymity against compromised endpoints, traffic correlation, coercion, compelled disclosure, insider abuse, or operational mistakes.

## Refresh triggers

Re-run the affected research and evaluation before release when any of these changes:

- adopted newsroom standards, especially anonymity, AI, corrections, source relationships, or fairness;
- applicable access-to-information, source-protection, recording, privacy, defamation, sanctions, or court rules;
- SecureDrop supported architecture or threat model;
- C2PA specification, trust-list behavior, validator, or manifest handling;
- IPTC correction/publishing semantics or newsroom syndication format;
- OCR, translation, transcription, metadata, archive, or media-forensics tools/models;
- model provider data use, retention, region, logging, abuse monitoring, or support-access terms;
- evidence stores, identity vaults, retrieval indexes, tenancy, encryption, or backup topology;
- new synthetic-media generators, platform transformations, or detector benchmarks;
- any source leak, unauthorized retrieval/contact/export/publication, false entity merge, missed contradiction, or failed correction propagation.
- search/news/public-record/company/geospatial/ASR/translation/storage/workflow/CMS/notification API, authentication, pagination, quota, field, status, rights, retention, redistribution, finality, pricing, or discontinuation changes;
- adapter manifests or conformance reports expire, deployment accounts/regions change, or an upstream provider contradicts the pinned local semantics.

## Source register

All web sources below were checked on 2026-08-31. “Primary” means the publisher owns the standard, policy, software, dataset, or official guidance; it does not mean every statement in the source is universally applicable.

### Editorial standards and accountability sources

| Source | Type | Contribution | Important limitation |
|---|---|---|---|
| [Reuters Journalistic Standards](https://reutersagency.com/about/standards-values/) | Primary newsroom standard | Accuracy over speed, sourcing, anonymity exceptions, fair comment, correction, internet reporting, AI caution | Reuters policy, not universal law or policy for every newsroom |
| [AP News Values and Principles](https://www.ap.org/about/news-values-and-principles/) | Primary newsroom standard | Accuracy, independence, fabrication, sourcing, corrections, visual integrity | Some operational detail is in linked sections/PDF and can change |
| [AP 2026 standards PDF](https://www.ap.org/wp-content/uploads/2026/02/ap-news-values-and-principles-2026.pdf) | Primary newsroom standard | Current consolidated standards reference | Must check AP page for superseding editions |
| [AP: Telling the Story](https://www.ap.org/about/news-values-and-principles/telling-the-story/) | Primary newsroom standard | Visible, labeled, channel-specific corrections | AP distribution workflow is not a universal correction protocol |
| [AP on anonymous sources](https://www.ap.org/the-definitive-source/behind-the-news/when-is-it-ok-to-use-anonymous-sources/) | Primary newsroom explanation | Practical anonymity criteria and attribution guidance | Explanatory article; the controlled policy is the standards document |
| [AP 2026 AI standards update](https://www.ap.org/the-definitive-source/announcements/ap-updates-newsroom-standards-for-artificial-intelligence/) | Primary newsroom announcement | AI may assist selected tasks; journalists retain verification, editorial judgment, accountability, and review | Public announcement summarizes rather than reproduces every internal implementation rule |
| [SPJ Code of Ethics revision announcement](https://www.spj.org/spj-ethics-committee-proposes-revisions-to-code-of-ethics/) | Primary professional body | Confirms August 2026 text is proposed, not adopted | Recheck after review/adoption process |
| [EFCSN Code of Standards](https://efcsn.com/code-of-standards/) | Primary professional network standard | Source transparency, central-claim sourcing, anonymous-source corroboration, corrections | Written for fact-checking organizations; not all investigative newsrooms |

### Source protection, privacy, and safety sources

| Source | Type | Contribution | Important limitation |
|---|---|---|---|
| [CPJ Digital Safety Kit](https://cpj.org/2019/07/digital-safety-kit-journalists/) | Primary journalist-safety guidance | Metadata, provider, account, device, phishing, travel, and communications risks | General starting point; a case-specific threat assessment remains necessary |
| [SecureDrop stable documentation](https://docs.securedrop.org/en/stable/) | Primary product documentation | Supported installation, administration, and journalist workflow boundary | Stable docs move with the current release; deployments need official maintenance practice |
| [SecureDrop threat model](https://docs.securedrop.org/en/stable/threat_model/threat_model.html) | Primary threat model | Assumptions, protected assets, and attacker boundaries | Assumptions must be checked against local sources, endpoints, and adversaries |
| [SecureDrop attacks and countermeasures](https://docs.securedrop.org/en/stable/threat_model/mitigations.html) | Primary security documentation | Workstation, source interface, seizure, compromise, and operational mitigations | Does not make an agent integration safe by itself |
| [SecureDrop Protocol repository](https://github.com/freedomofpress/securedrop-protocol) | Primary experimental repository | Shows emerging protocol work and explicitly warns it is proof-of-concept | Not production-ready; excluded from reference implementation |
| [ICO pseudonymisation guidance](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/data-sharing/anonymisation/pseudonymisation/) | Primary regulator guidance | Mapping attacks, indirect identifiers, outliers, re-identification controls | UK data-protection context; engineering risks apply more broadly than legal conclusions |
| [NIST IR 8053](https://csrc.nist.gov/pubs/ir/8053/final) | Primary government technical report | De-identification and re-identification risk concepts | Published 2015; not a newsroom-specific standard |
| [RCFP Reporter’s Privilege Compendium](https://www.rcfp.org/reporters-privilege/) | Specialist legal reference | US state and federal-circuit variability in compelled disclosure | Sections have different update dates; not legal advice |
| [ECHR journalistic-sources factsheet](https://www.echr.coe.int/documents/d/echr/fs_journalistic_sources_eng) | Primary court factsheet | European human-rights case-law overview for source confidentiality | Case-law summary, not a substitute for jurisdiction-specific advice |
| [Council of Europe journalist-safety resources](https://www.coe.int/en/web/freedom-expression/safety-of-journalists) | Primary intergovernmental guidance | Source protection, editorial autonomy, and safety environment | National implementation varies |

### Evidence, provenance, capture, and extraction sources

| Source | Type | Contribution | Important limitation |
|---|---|---|---|
| [NARA trustworthy electronic records guidance](https://www.archives.gov/records-mgmt/policy/electronic-signature-technology.html) | Primary records guidance | Reliability, authenticity, integrity, usability, context, structure, authorized annotations | Older electronic-signature context; principles remain useful but implementation must be modernized |
| [W3C PROV-O](https://www.w3.org/TR/2013/REC-prov-o-20130430/) | Primary web standard | Interchange concepts for entity/activity/agent provenance | Ontology alone does not supply newsroom custody or truth policy |
| [Library of Congress WARC description](https://www.loc.gov/preservation/digital/formats/fdd/fdd000236.shtml) | Primary preservation reference | WARC record types, packaging, format sustainability | Format description, not capture-completeness assurance |
| [Library of Congress WACZ description](https://www.loc.gov/preservation/digital/formats/fdd/fdd000586.shtml) | Primary preservation reference | Portable WARC/index/context package | Packaging does not repair missing crawl content |
| [RFC 7089: Memento](https://datatracker.ietf.org/doc/html/rfc7089) | Primary internet standard | Datetime negotiation and identification of prior resource states | Archive support and holdings remain variable |
| [UK Government Web Archive limitations](https://www.nationalarchives.gov.uk/webarchive/find-a-website/limitations/) | Primary archive guidance | Concrete dynamic, embedded, interactive, authenticated, and streaming capture gaps | One archive’s implementation; limitations are illustrative, not exhaustive |
| [OHCHR Berkeley Protocol record](https://searchlibrary.ohchr.org/record/30334?ln=en) | Primary intergovernmental publication record | Confirms scope, publisher, and edition | Focused on serious international/human-rights investigations |
| [Berkeley Protocol PDF](https://www.ohchr.org/sites/default/files/2022-04/OHCHR_BerkeleyProtocol.pdf) | Primary protocol | Preservation, verification, method documentation, legal/ethical practice, safety | Must be adapted to journalism; it is not newsroom policy |
| [C2PA Specifications 2.3](https://spec.c2pa.org/specifications/specifications/2.3/index.html) | Primary technical standard | Current Content Credentials document set | Rapidly evolving ecosystem; validate implementation conformance |
| [C2PA Content Credentials specification](https://spec.c2pa.org/specifications/specifications/2.3/specs/C2PA_Specification.html) | Primary technical standard | Assertions, claims, manifests, bindings, signatures, trust, validation | Provenance assertions are not truth/value judgments |
| [`c2pa-rs` releases](https://github.com/contentauth/c2pa-rs/releases) | Primary implementation repository | Current validator/CLI release behavior and changes | `0.x` tools require pinning and regression tests |
| [IPTC Photo Metadata Standard](https://iptc.org/standards/photo-metadata/iptc-standard/) | Primary news metadata standard | Descriptive, administrative, rights, and AI-related metadata fields | Embedded values may be missing, stripped, altered, or false |
| [Tesseract user manual](https://github.com/tesseract-ocr/tessdoc) | Primary implementation documentation | OCR versions, languages, models, configuration, limitations | Recognition accuracy depends heavily on input and configuration |
| [Tesseract release notes](https://github.com/tesseract-ocr/tessdoc/blob/main/ReleaseNotes.md) | Primary implementation changelog | Current release history | A release date does not establish newsroom suitability |
| [ffprobe documentation](https://ffmpeg.org/ffprobe.html) | Primary implementation documentation | Structured container, stream, and metadata inspection | Output reflects parseable file data, not real-world truth |

### Public records, access, and correction-exchange sources

| Source | Type | Contribution | Important limitation |
|---|---|---|---|
| [DOJ Guide to the Freedom of Information Act](https://www.justice.gov/oip/doj-guide-freedom-information-act-0) | Primary US government guide | Federal procedural requirements, exemptions, processing, appeals, and case law | US federal scope; chapters update on different schedules |
| [DOJ: Make a FOIA Request](https://www.justice.gov/oip/make-foia-request-doj) | Primary agency guidance | Custodian routing, record requests, fees, formats, responses | DOJ-specific operational guidance |
| [UNESCO Access to Information Laws](https://www.unesco.org/en/access-information-laws) | Primary intergovernmental resource | Global adoption and implementation context | Country summaries do not replace current local law |
| [UNESCO legal framework](https://www.unesco.org/en/right-access-information/legal-framework) | Primary intergovernmental resource | Distinguishes guarantees and practical implementation | Self-reported data and high-level comparisons require local verification |
| [Tromsø Convention](https://www.coe.int/en/web/access-to-official-documents/home) | Primary treaty resource | Minimum access, withholding, public-interest, processing, and review concepts | Applies by party, scope, and domestic implementation |
| [IPTC NewsML-G2 Guidelines](https://www.iptc.org/std/NewsML-G2/guidelines/) | Primary news-exchange standard | Update, correction, version, `usable`, `withheld`, and `canceled` semantics | Downstream systems and retention law may require additional controls |

### Live adapter, provider, and storage sources

| Source | Type | Contribution | Important limitation |
|---|---|---|---|
| [FOIA.gov developer resources](https://www.foia.gov/developer/) | Primary US government API documentation | Agency/component directory, request-form metadata, API-key requirement and agency POST specification boundary | Does not supply universal request legality, submission finality, acknowledgement, production completeness or appeal state |
| [Regulations.gov API v4](https://open.gsa.gov/api/regulationsgov/) | Primary US government API documentation | Distinct document/comment/docket objects, filters, pagination, attachments, withdrawal and comment operation behavior | Agency configuration varies; attachments need explicit acquisition; accepted comment can await agency publication review |
| [SEC EDGAR APIs](https://www.sec.gov/search-filings/edgar-application-programming-interfaces) | Primary regulator API documentation | Submissions and XBRL facts with CIK/accession/taxonomy/context | Filing assertions and entity names require interpretation/corroboration; access policy still applies |
| [GLEIF API](https://www.gleif.org/en/lei-data/gleif-api/) | Primary system-owner documentation | LEI records and relationship data for entity candidates | LEI scope is not a complete beneficial-ownership or historical corporate graph |
| [Google Custom Search JSON API overview](https://developers.google.com/custom-search/v1/overview) | Primary provider documentation | Current access/discontinuation status and discovery semantics | Closed to new customers and scheduled to discontinue 2027-01-01; snippets are not evidence |
| [News API terms](https://newsapi.org/terms) | Primary provider terms | Plan-use and third-party content-right boundaries | Developer plan is not production and provider access does not grant publisher content rights |
| [WACZ 1.1.1](https://specs.webrecorder.net/wacz/1.1.1/) | Primary format specification | Current portable web-archive package/index semantics | Package validity is not capture completeness, authenticity, or simultaneity |
| [Nominatim usage policy](https://operations.osmfoundation.org/policies/nominatim/) | Primary service-owner policy | Rate/use limits, prohibited systematic uses, attribution, privacy and change risk | Applies to the public OSMF service, not every self-hosted or third-party deployment |
| [OpenStreetMap copyright](https://www.openstreetmap.org/copyright) | Primary project rights page | ODbL and attribution baseline | Database/share-alike questions need rights review for the intended product and extraction |
| [Cloud Speech-to-Text data logging](https://cloud.google.com/speech-to-text/docs/data-logging) | Primary provider documentation | Default non-logging statement and opt-in data-logging distinction | Deployment project settings and separate opt-in terms must be verified; project deletion does not delete logged data |
| [Speech-to-Text opt-in terms](https://cloud.google.com/speech-to-text/docs/data-logging-terms) | Primary provider terms | Secondary-use and retention consequences when data logging is enabled | Terms are provider/account specific and can change; high-risk evidence should not rely on an unchecked setting |
| [Cloud Translation API overview](https://cloud.google.com/translate/docs/api-overview) | Primary provider documentation | Editions/models and provider statement on model-improvement data use | Contract, region, logs, support access, edition, glossary and deletion remain deployment-specific |
| [SecureDrop releases/news](https://securedrop.org/news/) | Primary product release channel | Cut-off versions for SecureDrop, Workstation/Qubes and Inbox | Current release status does not prove a deployment is patched, correctly operated or safe for a given source |
| [Freedom of the Press Foundation source-protection guide](https://freedom.press/digisec/guides/source-protection/) | Primary specialist guidance | End-to-end device, metadata, communication and operational source risk | General guidance cannot replace assignment-specific threat assessment |
| [S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html) | Primary provider documentation | Versioning prerequisite, WORM retention and legal-hold semantics | Object Lock is not complete custody, authenticity, deletion, encryption-key or availability control |
| [WordPress Posts REST API](https://developer.wordpress.org/rest-api/reference/posts/) | Primary project API documentation | Post create/read/update surface for a restricted draft adapter | API response/status is not editorial approval, public observation or correction propagation |
| [Jira Cloud REST API v3 introduction](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro) | Primary provider API documentation | Authentication, permissions, pagination, timestamps and version surface | Local issue/status/comment semantics cannot be assumed to mean approval |
| [Slack rate-limit/terms change FAQ](https://api.slack.com/changelog/2025-05-terms-rate-limit-update-and-faq) | Primary provider announcement | Distribution-type and rate-limit differences that affect notification adapters | Delivery is not acknowledgement or approval; notifications must exclude source/evidence secrets |
| [MCP authorization specification 2025-11-25](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization) | Primary protocol specification | Resource-bound authorization and token-handling boundary | MCP does not add evidence, custody, rights or editorial-effect semantics |
| [MCP tasks 2025-11-25](https://modelcontextprotocol.io/specification/2025-11-25/basic/utilities/tasks) | Primary protocol specification | Bounded asynchronous task transport semantics | Task support is experimental at this specification line and must not own durable newsroom state |

### Media-forensics evaluation sources

| Source | Type | Contribution | Important limitation |
|---|---|---|---|
| [NIST Guardians of Forensic Evidence](https://www.nist.gov/programs-projects/guardians-forensic-evidence) | Primary government research program | Real-world usability, generalization, post-processing, anti-forensics, and validation lifecycle | Program guidance is evolving and does not certify a specific detector |
| [NIST OpenMFC](https://mfc.nist.gov/) | Primary evaluation program | Manipulation, deepfake, provenance, and localization task design | Benchmark conditions remain abstractions of local casework |
| [DeepfakeBench repository](https://github.com/SCLBD/DeepfakeBench) | Primary research artifact | Unified implementations, data handling, and evaluation protocols | Dataset rights and deployment distributions vary; repository results are not operational validation |
| [DF40 paper](https://proceedings.neurips.cc/paper_files/paper/2024/file/34239f60eca7ce9bee5280aaf81362d8-Paper-Datasets_and_Benchmarks_Track.pdf) | Peer-reviewed benchmark paper | Forgery diversity, realism, protocol, and generalization gaps | Concentrates on defined deepfake categories and benchmark datasets |
| [Deepfake-Eval-2024](https://arxiv.org/abs/2503.02857) | Research preprint and dataset | In-the-wild multilingual media and large benchmark-to-real-world performance drops | Preprint status; collection and labels have their own representativeness limits |

## Internal design sources inspected

The blueprint reuses the repository’s canonical architecture instead of inventing a category-specific runtime:

- [Agent state and event contracts](../../runtime/agent-state-and-event-contracts.md)
- [Durable execution](../../runtime/durable-execution.md)
- [Execution boundaries](../../runtime/execution-boundaries.md)
- [Run controls](../../runtime/run-controls.md)
- [Tool contracts](../../tools/tool-contracts.md)
- [Tool results, artifacts, and provenance](../../tools/tool-results-artifacts-and-provenance.md)
- [Context engineering](../../context-memory/context-engineering.md)
- [Compaction and continuity](../../context-memory/compaction-and-continuity.md)
- [Memory architecture](../../context-memory/memory-architecture.md)
- [Prompt injection and untrusted data](../../security/prompt-injection-and-untrusted-data.md)
- [Permissions, sandboxing, and secrets](../../security/permissions-sandboxing-and-secrets.md)
- [Idempotency and side effects](../../reliability/idempotency-and-side-effects.md)
- [Failure taxonomy](../../reliability/failure-taxonomy.md)
- [Evaluation-driven development](../../evaluation/evaluation-driven-development.md)
- [Trajectory and reliability evaluation](../../evaluation/trajectory-and-reliability-evaluation.md)
- [Observability and tracing](../../evaluation/observability-and-tracing.md)
- [Queues, scheduling, and backpressure](../../operations/queues-scheduling-and-backpressure.md)
- [Scaling, capacity, and SLOs](../../operations/scaling-capacity-and-slos.md)
- [Deployment, release, and incident response](../../operations/deployment-release-and-incident-response.md)
- [Model routing, cost, and latency](../../operations/model-routing-cost-and-latency.md)

## Final synthesis

The research supports a production agent only when “agent” means a bounded, reversible reasoning component inside a larger evidence system. The production asset is the durable chain from source-safe intake to immutable evidence, explicit epistemic state, reproducible verification, recorded human authority, reviewable editorial/legal handoff, and traceable correction effects.

The most dangerous design error is not a weak search query. It is collapsing distinctions: pseudonym into anonymity, provenance into truth, document count into independent corroboration, model confidence into editorial confidence, capture time into event time, draft into publication, or a correction intent into a completed downstream correction. The companion blueprint makes those distinctions enforceable in schemas, state machines, approvals, isolation, evaluation, and operations.

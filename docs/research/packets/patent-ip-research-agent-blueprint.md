# Research Packet: Patent and Intellectual-Property Research Agent Blueprint

## Packet metadata

| Field | Value |
|---|---|
| Research baseline | 2026-08-31 |
| Area | `docs/agents/patent-ip-research-agent/` |
| Registry category | Patent and intellectual-property research |
| Method | Primary-source review plus patent-IR literature and benchmark review; contradictory claims preserved and resolved conservatively |
| Pass 2 verification cut | Official web documentation re-checked 2026-08-31; provider operations/terms must still be re-qualified immediately before implementation |
| Scope | Patent publication/application identity, priority/family/classification, office/legal-event data, prior-art candidate search, claims/elements, OCR/translation, provenance, review, security, evaluation, and production operations |
| Excluded authority | Legal advice or determinations; filing/submission; deadline/docket/matter/office modification |
| Refresh cadence | Quarterly for office APIs/terms/releases; at each IPC/CPC/WIPO-standard or connector change; immediate after material office migration/terms change |

## Research questions

1. Which identity, date, family, classification, and legal-event distinctions must survive normalization?
2. Which office and aggregate sources permit automation, bulk use, caching, model processing, and redistribution?
3. How should direct-office, aggregate, snapshot, OCR, and translated records be ranked when they conflict?
4. What search strategies work across exact claim language, paraphrase, classification, citations, family members, languages, and non-patent literature?
5. What do classic patent-retrieval datasets actually label, and how can family/temporal/pretraining leakage distort evaluation?
6. How can an agent propose claim-element mappings without making legal determinations?
7. What evidence, review, recovery, security, tenancy, SLO, cost, incident, and correction controls are required from stage 0 through 6?

## Taxonomy and local architecture review

The repository registry distinguishes this category from generic deep research and legal operations: patent research owns claims, classifications, family/legal-status data, prior-art search, jurisdiction/date boundaries, and attorney-review evidence. Generic research does not own those semantics; legal operations owns matters/contracts and accountable legal work.

The blueprint reuses rather than duplicates:

- [state/event contracts](../../runtime/agent-state-and-event-contracts.md), [durable execution](../../runtime/durable-execution.md), [execution boundaries](../../runtime/execution-boundaries.md), and [run controls](../../runtime/run-controls.md);
- [tool contracts](../../tools/tool-contracts.md), [artifact provenance](../../tools/tool-results-artifacts-and-provenance.md), and [tool lifecycle](../../tools/tool-registries-versioning-and-lifecycle.md);
- [context engineering](../../context-memory/context-engineering.md), [compaction](../../context-memory/compaction-and-continuity.md), and [memory architecture](../../context-memory/memory-architecture.md);
- [planning](../../orchestration/planning-and-replanning.md) and [handoffs](../../orchestration/delegation-handoffs-and-shared-state.md);
- [security](../../security/agent-threat-model.md), [prompt injection](../../security/prompt-injection-and-untrusted-data.md), and [permissions/secrets](../../security/permissions-sandboxing-and-secrets.md);
- [idempotency](../../reliability/idempotency-and-side-effects.md) and [failure taxonomy](../../reliability/failure-taxonomy.md);
- [trajectory evaluation](../../evaluation/trajectory-and-reliability-evaluation.md), [observability](../../evaluation/observability-and-tracing.md), and [evaluation-driven development](../../evaluation/evaluation-driven-development.md);
- [deployment/incidents](../../operations/deployment-release-and-incident-response.md), [scaling/SLOs](../../operations/scaling-capacity-and-slos.md), [queues/backpressure](../../operations/queues-scheduling-and-backpressure.md), and [model routing/cost](../../operations/model-routing-cost-and-latency.md).

Local design conclusion: use a deterministic workflow around one bounded investigator. Durable evidence graph state is authoritative; search indexes and model contexts are projections; policy and external effects are deterministic.

## Primary standards and official data sources

### WIPO standards and PCT guidance

| Source | Material finding | Blueprint decision |
|---|---|---|
| [WIPO Standards Part 3](https://www.wipo.int/en/web/standards/part_03_standards) | Current standard versions include bibliographic, legal-status, API, priority-package, XML, and JSON standards; versions continue to change | Pin standards/office implementation versions; never bake prompt-memory constants as timeless |
| [WIPO ST.16](https://www.wipo.int/documents/d/standards/docs-en-03-16-01.pdf) | Kind codes require the office context; digits can be office-specific; correction codes exist | Parse office, number, and kind separately; retain raw value and editioned office mapping |
| [WIPO ST.27](https://www.wipo.int/documents/d/standards/docs-en-tracked-changes-03-27-01-changes-2019.pdf) | Legal status is event/state based under applicable office law; formats/languages/timing differ | Preserve raw events and source-scoped projections; no universal `status` |
| [ST.27 implementation survey](https://www.wipo.int/en/web/standards/surveys/papi-p2/collated) | Implementation is not universal | `unmapped` and office-specific events remain valid; do not assume standardized coverage |
| [WIPO ST.14](https://www.wipo.int/documents/d/standards/docs-en-tracked-changes-03-14-01_changes_2016.pdf) | Citation categories and identification can include claims/passages | Store office-assigned category as observation, not agent legal judgment |
| [WIPO ST.22](https://www.wipo.int/documents/d/standards/docs-en-03-22-01.pdf) | OCR-oriented presentation guidance recognizes document/OCR constraints | Preserve page images and text-layer identity; validate claim/page structure |
| [PCT ISPE 5.20–5.28](https://www.wipo.int/en/web/pct-system/texts/ispe/5_20_28) | Claim interpretation for search uses description/drawings context | Supply context for search, but counsel retains construction |
| [PCT ISPE 12.01–12.02](https://www.wipo.int/en/web/pct-system/texts/ispe/12_01_02) | Novelty analysis considers all essential features in the relevant framework | Element mapping requires passage-level evidence; the agent still makes no novelty conclusion |
| [PCT ISPE 15.21–15.28](https://www.wipo.int/en/web/pct-system/texts/ispe/15_21_28) | International search addresses claims as filed and uses description/drawings | Preserve exact claim version and bounded search protocol |
| [PCT ISPE 15.63–15.72](https://www.wipo.int/en/web/pct-system/texts/ispe/15_63_72) | Date doubts can change citation handling | Uncertain dates enter review, not silent include/exclude |
| [PCT ISPE 16.22–16.85](https://www.wipo.int/en/web/pct-system/texts/ispe/16_22_85) | Reports identify relevant claims, passages, and classifications | Review package retains exact claims/passages and source annotations |
| [PCT ISPE 6.01–6.05](https://www.wipo.int/en/web/pct-system/texts/ispe/6_01_05) | Relevant date and priority treatment are task/fact sensitive | Date theory is counsel-supplied research filter, not inferred law |

### WIPO search and analytics services

| Source | Material finding | Blueprint decision |
|---|---|---|
| [PATENTSCOPE terms](https://www.wipo.int/en/web/patentscope/data/terms_patentscope) | Automated queries, bulk acquisition/downloading/storing, and scraping are prohibited; OCR and MT have caveats; results are not legal opinions | Public UI is human verification, not agent API; no automation unless separately authorized; label OCR/MT |
| [PATENTSCOPE user guide](https://patentscope2.wipo.int/search/help/en/users_guide.pdf) | Rich fields, query modes, and office data are available | Use as human search/verification reference; reproduce searches on authorized infrastructure |
| [PATENTSCOPE CLIR](https://patentscope.wipo.int/search/en/clir/clir.jsf) | Cross-lingual search supports controlled domains/variants/languages | Model multilingual branch after supervised variant selection, not blind translation |
| [PATENTSCOPE kind codes](https://www.wipo.int/en/web/patentscope/data/kind_codes) | Kind meanings vary by office | Editioned office-kind table |
| [PATENTSCOPE national-phase procedures](https://www.wipo.int/en/web/patentscope/data/national_phase/procedures) | National-phase coverage and updates vary | Absence is source-scoped; direct national verification for consequential facts |
| [WIPO Patent Register Portal](https://www.wipo.int/en/web/wipo-inspire/patent-register-portal) | Routes users across many national/regional registers | Navigation layer, not universal status source |
| [WIPO IPC](https://www.wipo.int/en/web/classification-ipc) | IPC is revised and master files are downloadable | Pin scheme edition and source hash; rebuild as versioned reference data |
| [WIPO Patent Analytics](https://www.wipo.int/en/web/patent-analytics) | Official analytics guidance and reports emphasize method design | Search protocol logs keywords/classes/iterations/limits |
| [WIPO patent-landscape guidelines](https://www.wipo.int/publications/en/details.jsp?id=3938) | Patent landscape work uses explicit methodology | Multi-pronged, reviewable search rather than one opaque ranking |
| [WIPO GenAI landscape appendix](https://www.wipo.int/web-publications/patent-landscape-report-generative-artificial-intelligence-genai/en/appendices.html) | Uses keyword/classification, precision/recall, classifiers, and a declared family/status methodology | Report corpus, family proxy, active-status rule, classifier, precision/recall, and time scope |
| [WIPO Translate](https://www.wipo.int/en/web/ai-tools-services/wipo-translate) | Translation service is positioned for patent documents | Treat as discovery aid subject to confidentiality and terms |
| [WIPO public translation warning](https://patentscope.wipo.int/translate/translate.jsf?interfaceLanguage=en) | Warns not to submit undisclosed or sensitive data | Conservative resolution: never send confidential/unpublished/privileged inputs to public translator |

### EPO data, families, status, and classifications

| Source | Material finding | Blueprint decision |
|---|---|---|
| [EPO patent families](https://www.epo.org/en/searching-for-patents/helpful-resources/first-time-here/patent-families) | Simple and extended families use different priority relationships | Store provider, definition, members, and snapshot; family is navigation |
| [Espacenet resource book](https://link.epo.org/web/espacenet_resourcebook_v3.0_en.pdf) | Family members can have different claims/outcomes | Never infer claim identity from family membership |
| [EPO Guidelines B-X 9.1.2](https://www.epo.org/en/legal/guidelines-epc/2026/b_x_9_1_2.html) | Corresponding documents may contain relevant subject matter absent elsewhere | Review actual member/edition and cite its passage |
| [Espacenet release notes](https://www.epo.org/en/searching-for-patents/technical/espacenet/release-notes) | Historical searchable text may be OCR; classification data can be family-propagated | Store text layer and classification assignment level |
| [EPO INPADOC](https://www.epo.org/en/searching-for-patents/data/bulk-data-sets/inpadoc) | Worldwide legal-event data covers many authorities, delivered as licensed snapshots/frontfiles | Useful aggregate discovery/projection input; not global legal opinion |
| [EPO bulk-data manuals](https://www.epo.org/en/searching-for-patents/data/bulk-data-sets/manuals) | Schemas/manuals have versions and effective dates | Connector contract tests and release pinning |
| [EPO PATSTAT](https://www.epo.org/en/about-us/observatory-patents-and-technology/observatory-tools/patstat) | Large analytical database released periodically | Analytics snapshot only; package records edition |
| [PATSTAT documentation/coverage](https://www.epo.org/en/about-us/observatory-patents-and-technology/observatory-tools/patstat/documentation-and-coverage) | Coverage/documentation are explicit and evolving | Machine-readable coverage statement and sampling |
| [EPO OPS fair use](https://www.epo.org/en/service-support/ordering/fair-use) | Supported automated service with volume/rate policy | Use supported API/terms and admission control; do not automate GUI |
| [OPS terms](https://developers.epo.org/sites/default/files/terms_and_conditions_OPS%202.0%20EN_DE_FR.pdf) | Use is licensed and data accuracy/completeness is not warranted | Source-rights ledger and explicit caveats |
| [EPO raw-data terms](https://www.epo.org/en/service-support/ordering/raw-data-terms-and-conditions) | License, transfer, third-party rights, security, and attribution matter | Rights propagate to raw and derived data |
| [EPO website terms](https://www.epo.org/en/terms-of-use/terms-and-conditions-use-website-european-patent-office) | Espacenet is entry-level and no-result is not FTO/professional advice | Never equate no result with clearance; UI is not bulk infrastructure |
| [European Patent Register](https://www.epo.org/en/searching-for-patents/legal/register) | Current procedural/legal information and Global Dossier access | Targeted official verification, subject to jurisdictional scope |
| [Federated Register](https://www.epo.org/en/searching-for-patents/legal/register/documentation/federated-register) | Aggregates post-grant national data | Preserve member-state/source/time; verify nationally when material |
| [Federated Register coverage](https://www.epo.org/en/searching-for-patents/legal/register/documentation/data-coverage) | Coverage and national links vary | No pan-European universal status field |

### USPTO and other office sources

| Source | Material finding | Blueprint decision |
|---|---|---|
| [USPTO Patent Public Search](https://www.uspto.gov/patents/search/patent-public-search) | Current advanced patent search UI | Human exploration/verification; log reproducible query syntax |
| [USPTO searchable indexes](https://www.uspto.gov/patents/search/patent-public-search/searchable-indexes) | Claims and classification fields are searchable | Field-aware lexical branches |
| [USPTO operators](https://www.uspto.gov/patents/search/patent-public-search/operators) | Rich query operators | Query compiler validates documented syntax/version |
| [USPTO FAQs](https://www.uspto.gov/patents/search/patent-public-search/faqs) | Boolean/TF-IDF behavior, limits, no semantic search | Lexical baseline plus separate semantic candidate generation |
| [USPTO Open Data Portal bulk/API search](https://data.uspto.gov/apis/bulk-data/search) | ODP access and API/bulk documentation are current and changing | Discover/pin current OpenAPI/schema; contract-test; never invent endpoints |
| [USPTO application data](https://data.uspto.gov/apis/patent-file-wrapper/application-data) | File-wrapper application data capability exists | Matter-scoped supported connector with schema/version tests |
| [USPTO documents API page](https://data.uspto.gov/apis/patent-file-wrapper/documents) | File-wrapper documents are separately exposed | Large artifact acquisition with rights/size/checkpoint controls |
| [USPTO BDSS retirement notice](https://www.uspto.gov/subscription-center/2025/bulk-data-storage-system-retiring-soon) | BDSS retired and data moved to ODP | Source migration is an operational change; eliminate stale integration assumptions |
| [USPTO maintenance information](https://www.uspto.gov/patents/maintain) | USPTO does not calculate expiration dates | No agent patent-expiry computation from maintenance records |
| [USPTO terms](https://www.uspto.gov/terms-use-uspto-websites) | Federal/public-domain principles do not erase third-party rights | Rights check patent images/NPL/provider additions separately |
| [PatentsView migration notice](https://www.uspto.gov/subscription-center/2026/patentsview-migrating-uspto-open-data-portal-march-20) | Data/API locations change | Connector lifecycle and deprecation monitoring |
| [PatentsView quality correction](https://patentsview.org/data-in-action/patentsview-team-identifies-data-quality-error-q1-2025-data-update) | Derived disambiguation can have corrected errors | Entity resolution is versioned/reversible hypothesis |
| [USPTO Global Dossier](https://www.uspto.gov/patents/basics/international-protection/global-dossier-initiative) | Participating IP5/PCT file-wrapper access | Useful multi-office navigation, not universal jurisdiction coverage |
| [IP5 Global Dossier/CCD](https://www.fiveipoffices.org/groups/globaldossier/filewrapper) | Common Citation Document/file-wrapper functions cover participating offices | Store office/citation provenance; do not generalize beyond coverage |
| [J-PlatPat document coverage](https://www.j-platpat.inpit.go.jp/html/c2000/patent_en.html) | Official coverage page reports current availability and possible defects/translation issues | Connector coverage statement and original-language review |
| [J-PlatPat update schedule](https://www.j-platpat.inpit.go.jp/html/c0300/index_en.html) | Update schedules are published | Freshness SLI and source-specific observation age |
| [KIPRIS Plus API status](https://plus.kipris.or.kr/eng/main/apiStatus.do?menuNo=310128) | Supported API service status is published | Use documented supported capability; monitor service status/schema |

## Pass 2 primary-source and availability ledger

The ledger records what was actually verified, the access date, and the limit on the inference. “Documented” does not mean credentials, contract rights, production coverage, or endpoint quality were live-tested.

| Primary source | Accessed | Verified current statement | Version/access limit and engineering annotation |
|---|---|---|---|
| [WIPO standards index](https://www.wipo.int/en/web/standards/part_03_standards) and [ST.97 Annex II](https://www.wipo.int/standards/en/st97/v2-0/annex-ii/index.html) | 2026-08-31 | Index lists ST.90 web-API guidance v2.0 (Dec. 2025), ST.96 update (Apr. 2026), and ST.97 JSON/Annex II v2.0 (July 2026) | Standards are recommendations/exchange schemas, not proof that an office/provider implements an endpoint. Adapter manifests pin the provider contract plus exact schema artifact |
| [WIPO IP API Catalog](https://www.wipo.int/en/web/standards/ip-api-catalog/index) | 2026-08-31 | Catalog routes users to APIs offered by IP institutions and exposes filters for operations/protocol/formats | Discovery catalog only; each listed office API still needs independent auth, terms, schema, coverage, and qualification review |
| [PATENTSCOPE public terms](https://www.wipo.int/en/web/patentscope/data/terms_patentscope) | 2026-08-31 | Page says last updated Oct. 2025; forbids automated queries, bulk acquisition/storage, and scraping; describes OCR/MT caveats and no legal opinion | Public UI gets a human-only manifest. The 10-search-action/minute boundary is not permission to automate |
| [WIPO PCT data products](https://www.wipo.int/en/web/patentscope/data/index) and [product terms](https://www.wipo.int/en/web/patentscope/data/terms) | 2026-08-31 | Fee products include PCT text/bibliographic/images and conditional Java/SOAP web-service access; page states national/regional office patent data is not currently available through these products | Access is subscription/contract-specific. Conditional web-service terms include retrieval/redistribution constraints; qualify only executed-product operations and PCT scope |
| [EPO OPS service page](https://www.epo.org/en/searching-for-patents/data/web-services/ops) and [OPS 3.2 reference](https://link.epo.org/web/searching-for-patents/data/en-ops-v3.2-documentation-version-1.3.20.pdf) | 2026-08-31 | Service is REST/XML with registration/OAuth; reference v1.3.20 (June 2024) separately documents published data, family, number, Register, legal, classification, and usage services | One qualification per service/constituent. Family is INPADOC extended family; Register and worldwide legal data are not interchangeable; hash current OpenAPI/XSD before coding |
| [EPO fair-use charter](https://www.epo.org/en/service-support/ordering/fair-use) and [OPS restrictions FAQ](https://www.epo.org/en/service-support/faq/searching-patents/open-patent-services/general-information/are-there-any) | 2026-08-31 | OPS currently has 4 GB/week free threshold, approximately 1 Mbit/s traffic limit, operation-sensitive request controls, and is not intended for complete/very large collections | Limits may vary operationally; capture throttling/quota headers and use raw/bulk products for corpus ingestion rather than designing to the threshold |
| [EPO OPS terms](https://www.epo.org/en/service-support/ordering/terms-and-conditions/ops-terms-and-conditions) | 2026-08-31 | OPS terms distinguish including data in products from redistributing data “as such,” disclaim completeness/accuracy, and require users to incorporate notified corrections | Rights and correction obligations propagate into derivative/index/package policy; current contract review remains necessary |
| [EPO website/Espacenet terms](https://www.epo.org/en/terms-of-use/terms-and-conditions-use-website-european-patent-office) | 2026-08-31 | Espacenet is entry-level, not a substitute for professional advice, not for bulk retrieval, and a no-result search is not unlimited freedom of action | Supports explicit FTO/professional boundary and UI-human-only rule; it does not itself define the legal scope of a specific engagement |
| [USPTO Patent File Wrapper application API](https://data.uspto.gov/apis/patent-file-wrapper/application-data), [documents API](https://data.uspto.gov/apis/patent-file-wrapper/documents), and [ODP syntax/access](https://data.uspto.gov/apis/api-syntax-examples) | 2026-08-31 | ODP documents supported application/document operations, API keys, and 2026 USPTO.gov account/MFA registration requirements; pages state legacy Developer Hub decommissioning | ODP pages contain legacy publication metadata while service banners/release state are newer. Treat Swagger/OpenAPI and live auth/fixtures—not page publication date—as the executable contract |
| [USPTO bulk-data product search](https://data.uspto.gov/apis/bulk-data/search) | 2026-08-31 | Supported API searches bulk-data product metadata and requires an API key | Product catalog search is not patent prior-art full-text search. Keep bulk product discovery/download separate from PFW and Patent Public Search |
| [USPTO preliminary search guidance](https://www.uspto.gov/patents/basics/apply) and [multi-step strategy](https://www.uspto.gov/patents/search/patent-search-strategy) | 2026-08-31 | USPTO describes public searching as preliminary, warns an examiner may find other information, and recommends foreign/NPL/classification/citation expansion and practitioner review | Supports hybrid retrieval and non-completeness, but not a universal search protocol or legal conclusion |
| [Lens API documentation](https://docs.api.lens.org/) and [Lens access plans](https://support.lens.org/knowledge-base/lens-patent-and-scholar-api/) | 2026-08-31 | Docs report API 2.19.3, patent schema 1.6.5, last update 2026-04-17; trial is short/non-commercial/limited academic while continued automated/commercial use is paid/custom | Availability, quota, attribution, redistribution, and derivatives are plan/contract-specific. Lens records/IDs/families/status remain provider assertions |
| [Lens bulk downloads](https://support.lens.org/knowledge-base/bulk-data-downloads/) | 2026-08-31 | Support page updated 2026-03-17 documents fortnightly versioned files, authenticated download/usage operations, rate limits, and 429 behavior | Bulk is a separately subscribed operation. Pin `dataVersion`, precursor, schema, file hash and contract; do not assume search token includes bulk rights |
| [Google Patents search help](https://support.google.com/faqs/answer/7049475), [results help](https://support.google.com/faqs/answer/7049588), and [privacy](https://support.google.com/faqs/answer/6391039) | 2026-08-31 | Official help documents UI syntax, approximate result counts, simple-family suppression, and query-log/automated-abuse handling | No general Google Patents search API was found in the reviewed official material. That is a research finding, not proof none exists under private agreement; public UI remains human-only unless documented authorization is obtained |
| [BigQuery public datasets](https://docs.cloud.google.com/bigquery/public-data) and [Google patent-dataset announcement](https://cloud.google.com/blog/topics/public-datasets/google-patents-public-datasets-connecting-public-paid-and-private-patent-data) | 2026-08-31 | BigQuery supports API/SQL access, query billing, and no public-dataset SLA; the 2017 patent announcement describes IFI-sourced historical scope | Inspect current table/schema/update metadata and terms before use. BigQuery analytical tables are not a live register or a Google Patents search endpoint |
| [ABA Model Rule 1.6](https://www.americanbar.org/groups/professional_responsibility/publications/model_rules_of_professional_conduct/rule_1_6_confidentiality_of_information/) and [Formal Opinion 512](https://www.americanbar.org/content/dam/aba/administrative/professional_responsibility/ethics-opinions/aba-formal-opinion-512.pdf) | 2026-08-31 | Rule addresses representation information and reasonable safeguards; Opinion 512 addresses lawyers’ competence, confidentiality, communication, supervision, verification, and fees with GAI | ABA materials are not the only applicable authority. Counsel must apply governing jurisdiction, client terms, privilege/work-product law, and court/protective orders |
| [BIS deemed-export guidance](https://media.bis.gov/learn-support/deemed-exports/what-deemed-export), [EAR scope](https://www.bis.gov/ear/title-15/subtitle-b/chapter-vii/subchapter-c/part-730/ss-7305-coverage-more-exports), and [EAR Part 734](https://media.bis.gov/regulations/ear/734) | 2026-08-31 | Controlled non-public technology/source-code releases and electronic transmissions can be exports/deemed exports; public information/fundamental research have separate treatment | Agent enforces a counsel/compliance-supplied decision; it cannot classify data, screen legal exceptions, or assume patent publication makes surrounding confidential technical data public |
| [NIST AI 600-1](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf) and [NIST SP 800-218](https://csrc.nist.gov/pubs/sp/800/218/final) | 2026-08-31 | NIST identifies indirect prompt injection/data poisoning risks and SSDF practices for software provenance/third-party components | Risk-management and secure-development guidance, not patent-source or legal authority; applied here to untrusted patent/NPL content and the parser/model/data supply chain |

## Search and retrieval research

### Foundational information retrieval

| Source | Finding | Use |
|---|---|---|
| [Robertson and Zaragoza — BM25](https://www.staff.city.ac.uk/~sbrp622/papers/foundations_bm25_review.pdf) | Strong probabilistic lexical baseline | Exact/rare technical claim language and transparent fielded scoring |
| [Sentence-BERT](https://aclanthology.org/D19-1410/) | Efficient sentence embeddings for semantic similarity | Candidate generation only; not claim coverage |
| [ColBERT](https://arxiv.org/abs/2004.12832) | Token-level late interaction retains finer matching signals | Semantic reranking with inspectable token interaction; still provisional |
| [SPLADE v2](https://arxiv.org/abs/2109.10086) | Learned sparse expansion can bridge vocabulary gaps | Hybrid sparse branch with pinned model/index |

Architecture conclusion: lexical, phrase/proximity, classification, citation/family, multilingual, semantic, entity, and NPL branches are complementary. Semantic scores rank candidates; exact passage and element review determine whether a candidate warrants professional attention.

### Patent-specific benchmarks and papers

| Source | Finding | Limitation/decision |
|---|---|---|
| [NTCIR-5 patent collection](https://research.nii.ac.jp/ntcir/permission/ntcir-5/perm-en-PATENT.html) | Claims are topics; patent/passages/classification tasks; professional and examiner-derived labels | Historical Japanese corpus; research-use rights; citations are incomplete/biased labels |
| [NTCIR-6 overview](https://research.nii.ac.jp/ntcir/workshop/OnlineProceedings6/NTCIR/78.pdf) | Patent-to-patent invalidity-search evaluation with graded categories | Academic task does not grant agent authority to decide invalidity |
| [CLEF-IP archive/licensing](https://www.ifs.tuwien.ac.at/~clef-ip/download-central.shtml) | Large multilingual historical EPO-derived corpora with explicit license | Non-commercial/share-alike constraints and old cutoff require rights/temporal caution |
| [CLEF-IP 2010 collection](https://researchdata.tuwien.at/records/jqrsc-jbq51) | Prior-art candidate and classification tasks over historical corpus | Prosecution citations/family structure can bias evaluation |
| [CLEF-IP 2009 proceedings](https://ceur-ws.org/Vol-1175/) | Many approaches combine claims, IPC, query generation, preprocessing, and IR models | Supports hybrid/ablation methodology, not one universal best method |
| [PatentMatch](https://arxiv.org/abs/2012.13919) | Claim/prior-art passage matching dataset | Matching label remains task-specific; verify date/family/label construction |
| [Patent retrieval literature review](https://arxiv.org/abs/1701.00324) | Patent IR spans query reformulation, classification, citations, language, and specialized evaluation | Use as secondary synthesis; current production behavior still requires direct office/source research |

Evaluation conclusion: classic collections are regression fixtures. Production release decisions require recent private cases, professional adjudication, family/near-duplicate grouping, historical corpus cutoffs, future-signal removal, license checks, and repeated-run trajectory testing.

## Resolved contradictions

| Tension | Evidence | Conservative resolution |
|---|---|---|
| “Worldwide legal status” vs jurisdiction-specific incomplete events | INPADOC/WIPO aggregation breadth; EPO/WIPO coverage and disclaimer language | Call it worldwide legal-event data; project only source-scoped states; verify national source when material |
| Family is “same invention” vs different claims/disclosure | Provider family grouping summaries; EPO family/member guidance | Family is provider-defined navigation; compare actual members/claim sets |
| Machine translation is useful vs not authoritative/confidential | WIPO Translate capability; PATENTSCOPE legal-value and sensitive-data warnings | Use approved MT for discovery; original-language evidence and human review for consequential mappings; no confidential public MT |
| OCR full text enables search vs OCR corrupts evidence | Office searchable text and ST.22/PATENTSCOPE caveats | Store OCR as separate layer, page images, quality checks, and stale dependent mappings after correction |
| Official aggregate improves coverage vs direct office is authoritative | Aggregates combine offices; national/register sources have local authority but also gaps/lag | Preserve both; choose by fact/jurisdiction/time through review; never silently overwrite |
| Current database gives freshest answer vs historical research needs past knowability | Mutable registers and periodic snapshots | Separate replay from refresh; bind every package to observation and corpus versions |
| Examiner citations are relevance labels vs not exhaustive ground truth | NTCIR/CLEF label construction | Use for fixtures; add independent adjudication and missed-art labels |
| Semantic models retrieve paraphrases vs score implies coverage | IR papers show similarity retrieval; claim analysis needs exact relationships and context | Semantic candidate generation/ranking only; typed passage hypothesis and professional review |
| Public patent text is broadly reusable vs provider/NPL/third-party rights | Office terms and licensed data conditions | Rights ledger at artifact and derivative level; no inferred training/export rights |
| “No result” seems negative evidence vs source/search gaps | EPO terms and national-phase coverage caveats | Record query-scoped absence/coverage, never nonexistence/FTO/completeness |

## Decisions incorporated into the blueprint

1. **One bounded agent loop** proposes query branches and element/passage hypotheses; deterministic workflow owns policy, data, budgets, state, verification, approvals, and effects.
2. **Six semantic record classes**—observation, extracted fact, legal-status record, similarity hypothesis, reviewed research conclusion, external effect—cannot be collapsed.
3. **Identity is relational**: application, publication/kind, priority, family assertion, claim set, text layer, classification, and jurisdictional right remain distinct.
4. **Dates are typed** and accompanied by precision, source, observation time, snapshot, and reviewer-supplied research filter.
5. **Family and classification are versioned provider assertions**, not timeless fact or claim identity.
6. **Search is hybrid and reproducible**, with explicit source/query/branch/budget/stop coverage and passage-level evidence.
7. **Semantic similarity never equals claim coverage**; every mapping is provisional and lists differences.
8. **Original bytes and authoritative-language text survive**; OCR and translation are parallel layers with version/quality/review.
9. **Status is event-first and jurisdiction/time/source scoped**; no expiry/enforceability/ownership computation.
10. **Confidentiality and source rights are pre-retrieval controls** that propagate to indexes, embeddings, context, memory, evaluation, logs, and exports.
11. **No cross-matter memory by default**; reusable feedback requires rights/privacy/de-identification curation.
12. **Only D1 gated package export is permitted**; filing, docket/matter changes, deadline calculations, and legal effects are outside the system.
13. **Evaluation is temporal and family-disjoint**, includes failure injection and human review, and treats classic benchmarks as limited fixtures.
14. **Production readiness includes queues/backpressure, tenant isolation, SLOs, incident lineage, cost per verified outcome, compatible rollback, and correction propagation.**
15. **Capabilities are operation-level and qualified**, not connector-wide: provider service/schema/terms/coverage/auth/rights/limits, evidence receipts, cancellation, and reconciliation are pinned per operation.
16. **The canonical memory policy has seven explicit lifetimes** with use/reject, retention/deletion, poisoning, and evaluation controls; no raw conversation/vector store becomes authority.
17. **Compaction emits a loss-aware receipt** with event high-watermark, version pins, approvals, clocks, effects, invariant hash, omitted refs, and next safe action; restart/provider switch rehydrates and verifies authoritative state.
18. **Recall is reported only against disclosed finite denominators** and supplemented with pooled, stratified-reject, independent-miss, discovery-curve, and branch-ablation evidence; no automated completeness claim is made.
19. **Releases canary the whole behavior bundle**, including sources, policies, schemas, data/indexes, models, context/memory, stop/effect rules, rendering, and reviewer UI—not only the LLM.

## Claims deliberately not made

- No office data source is complete, current, or authoritative for every jurisdiction and legal question.
- No family definition establishes identical disclosure, claims, rights, or status.
- No classification symbol is timeless or necessarily assigned directly to each member.
- No automated search is exhaustive.
- No examiner/applicant citation set is complete ground truth.
- No OCR or machine translation is assumed accurate enough for consequential analysis without verification.
- No model score or element map determines novelty, validity, infringement, patentability, obviousness/inventive step, claim scope, ownership, expiry, enforceability, or freedom to operate.
- No undocumented API behavior, bulk right, cache right, training right, or redistribution right is assumed.

## Refresh triggers

- WIPO ST.9/ST.13/ST.16/ST.27/ST.90/ST.92/ST.96/ST.97 or IPC revision;
- CPC release, pre-release, concordance, or correction;
- EPO DOCDB/INPADOC/PATSTAT/OPS schema, terms, coverage, quota, or release change;
- USPTO ODP/PFW/Patent Public Search/PatentsView migration, authentication, schema, or terms change;
- J-PlatPat/KIPRIS/other office coverage, supported API, or update change;
- licensed-provider contract, derivative-data, model-use, retention, or redistribution change;
- OCR/translation/model/index upgrade;
- new case law or office procedure affecting counsel-supplied research protocols—not autonomously encoded as legal logic;
- incident, material reviewer correction, benchmark leakage discovery, or evidence-source quality regression.

## Research limitations

- Office interfaces, authentication, schemas, and terms can change after the baseline; integration teams must re-open current official documentation and contract-test before implementation.
- Not every national office publishes equivalent APIs, historical snapshots, English translations, or legal-event coverage. The blueprint intentionally uses capability/coverage records rather than inventing universal connectors.
- Legal-status and date significance is jurisdiction- and fact-specific. This packet researches data engineering, not legal advice.
- Foundational patent IR benchmarks are dated and may have restrictive licenses or label bias. They do not measure production completeness.
- Foundation-model training overlap with public patents is generally not fully knowable; private temporal/family-disjoint evaluations reduce but do not eliminate contamination risk.
- Commercial patent databases were handled as a generic licensed-integration class because specific contract terms vary by customer and were not supplied.
- Provider APIs and commercial channels were researched from public official documentation without live credentials or executed customer contracts. Endpoint behavior, quotas, dataset tables, regional availability, and derivative/redistribution rights still require implementation-time qualification.
- The reviewed official Google material documents the UI and BigQuery route but not a general search API; the blueprint therefore prohibits undocumented UI automation while leaving room for a later explicit commercial agreement.
- Confidentiality, privilege, export-control, FTO, patentability, validity, and status significance are jurisdiction/fact-specific. The controls route and preserve accountable decisions; they are not legal conclusions.

## Implementation handoff

The complete build sequence and exit gates are in [Zero-to-Production Roadmap and Authority Model](../../agents/patent-ip-research-agent/01-zero-to-production-and-authority.md). The [worked production flows](../../agents/patent-ip-research-agent/10-worked-production-flows.md) connect novelty support, evidence charting, status conflict, monitoring, review, reconciliation, and correction. The [area README](../../agents/patent-ip-research-agent/README.md) maps the remaining architecture, identity, retrieval, claims, runtime, provenance, security, and operations guides.

# Data Quality, Bias, and Responsible Data Report — FYP Assistant

**Course:** AI Engineering — Topic 4 Lab (Week 4)
**Project:** FYP Assistant — document-grounded assistant for FYP administration questions
**Corpus:** 90 prepared chunks from *FYP Handbook 2026–2027*, UTM Faculty of Artificial Intelligence
**Evidence:** `notebooks/week4_quality.ipynb` (7 executed cells), `docs/week4_validation_failures.csv`,
`docs/week4_duplicate_review.csv`, `docs/week4_coverage_by_source.png`, `data/week3_manifest.json`,
`docs/week3_data_inventory.md`, `docs/week3_pipeline_decisions.md`

> **Reading rule.** "Measured" means produced by an executed cell in `week4_quality.ipynb`.
> "N.V." means Not Verified — the evidence does not yet exist, and no rating is invented.
> This report records data-level risk. It does **not** claim anything about final system fairness,
> which is evaluated in Topic 8.

---

## 1. Data Quality Scorecard

| Dimension | Rating (1–5 / N.V.) | Evidence | Action / limitation |
|---|---:|---|---|
| **Completeness** | **4** | 90/90 chunks have non-empty text, non-empty source, and a page value. But the corpus covers only pp. 7–28 and 30–44; the seven appendix forms (pp. 95–119) and Chapters 4–9 are absent. | Accept for FR1/FR2/RAI1. FR5 is only partially grounded — condition C1 below. |
| **Consistency** | **4** | 90/90 carry the identical processing label `paragraph-aware-char-800-page-bound`; 0 stale source hashes; 0 out-of-scope pages. Chunk boundaries are page-bound but 67% end mid-sentence (measured in Week 3). | Accept; sentence-boundary chunking is a Topic 7 improvement, not a Topic 4 blocker. |
| **Accuracy** | **N.V.** | No source-to-chunk correctness test was run in Topic 4. Cell 3 manually compared four short chunks with their source pages, which is evidence of provenance, not a corpus-wide accuracy score. | Requires a sampled line-by-line comparison against the PDF before any accuracy claim. System-level correctness is Topic 8 (NFR2 ≥ 85%). |
| **Timeliness** | **4** | Source is the 2026–2027 handbook issue (PDF created 2026-10-08); stored SHA-256 matches the current local PDF at the time of the run; 0 stale hashes. | Re-verify the hash after every handbook reissue. No automated reissue watch exists — condition C2. |
| **Uniqueness** | **5** | 0 exact duplicates across 90 chunks (SHA-256 over whitespace-normalised text). Week 3 removed duplicated Canva text layers before chunking. | Accept, with the explicit caveat that exact hashing does not detect near-duplicates or conflicting versions. |

**Overall gate:** GO WITH CONDITIONS (§8). The corpus is controlled for the requirements it
actually covers; the FR5 form gap and the unverified accuracy dimension are documented,
not silently treated as passes.

---

## 2. Validation and Data-Defect Findings

Cell 4 of `week4_quality.ipynb` defines eight Pandera rules and runs them in lazy mode.
Failure cases are written to `docs/week4_validation_failures.csv`.

| # | Quality concern | Observable condition | Failures | Decision | Justification of threshold |
|---|---|---|---:|---|---|
| R1 | Missing | `text.strip() != ''` | 0 / 90 | Hard fail | An empty chunk supplies no retrievable evidence and cannot support a citation. |
| R2 | Missing / provenance | `source.strip() != ''` | 0 / 90 | Hard fail | Untraceable evidence cannot be verified against the handbook. |
| R3 | Missing / provenance | `page is not None` | 0 / 90 | Hard fail | FR3 and RAI1 require a source reference; without `page` a citation cannot be formed. |
| R4 | Corrupted | no `\x00` or `\ufffd` in text | 0 / 90 | Hard fail | Replacement characters indicate extraction damage that can alter quoted rules. |
| R5 | Stale | stored `source_sha256` equals current PDF hash | 0 / 90 | Hard fail | A reissued handbook invalidates every stored rule; silent staleness is unacceptable for an administration assistant. |
| R6 | Inconsistent | processing label equals the selected Week 3 method | 0 / 90 | Hard fail | A changed chunking contract would make the corpus irreproducible. |
| R7 | Noisy / short unit | `char_count >= 100` | **4 / 90 (4.4%)** | **Warning / manual review** | Below 100 characters is a reliable signal that a unit may be a heading or a truncation. In this corpus all four were checked against the source page and are legitimate headings or page-tail fragments. |
| R8 | Noisy whitespace | no 4+ consecutive newlines or spaces | 0 / 90 | Warning | Week 3's C3 rule normalised whitespace; this rule guards against regression. |

**Triage of the four R7 warnings** (Cell 3, each compared with the source page):

| Chunk | Page | Length | Evidence checked | Interpretation | Decision | Reason |
|---|---:|---:|---|---|---|---|
| 7 | 9 | 31 | p. 9 ends with `1.6.1 FYP 1 Execution Procedure` | Valid section heading | **Retain** | Gives FR1 a precise section anchor. |
| 13 | 12 | 68 | p. 12 ends with three milestone headings | Valid page-tail list | **Retain** | Legitimate content, low retrieval value; revisit in Topic 7. |
| 16 | 13 | 56 | p. 13 ends with advice plus list marker `viii.` | Valid advice with a minor artefact | **Retain with warning** | The sentence is useful for presentation guidance (FR5). |
| 87 | 43 | 49 | p. 43 ends with `References from Internet / Format and example / 3.3.3` | Valid section pointer | **Retain** | Supports Harvard-referencing questions (FR2). |

**Defect summary:** missing 0, corrupted 0, noisy-whitespace 0, inconsistent 0, stale 0,
duplicate 0, short-unit warnings 4 (all retained), still uncertain — accuracy (N.V.), rights
licence text (N.V., §5). Validation detected a condition; engineering judgement decided the
action. No chunk was removed or silently repaired.

---

## 3. Duplicate and Anomaly Findings

Cell 5 computed SHA-256 over whitespace-normalised text for all 90 chunks:

- **Before:** 90 chunks. **After:** 90 chunks. **Exact duplicates:** 0.
- Evidence: `docs/week4_duplicate_review.csv` (header only — no rows).
- Provenance: every retained chunk keeps `source`, `page`, `record_id`, `source_sha256`
  and `processing`; none of that metadata was discarded by deduplication.

**Anomalies retained deliberately:** the four short chunks in §2; page-level, not
section-level, citation granularity; and table text that flattens cell boundaries (Week 3
limitation 5). None is a duplicate.

**What exact hashing cannot establish:** near-duplicates, paraphrase overlap, contradictory
versions of the same rule, or resolution of which version is authoritative. No claim is made
that the corpus is semantically deduplicated. Four PDF pages (10, 16, 18, 94) contain no
extractable text; pages 10, 16 and 18 lie inside the ingested scope and remain a coverage
hole that OCR would be needed to close.

---

## 4. Representation, Coverage, and Bias Assessment

Cell 6 measured coverage from the `page` metadata actually captured in Topic 3. No
demographic or sensitive attribute was created — the corpus contains none.

| Required area | Why required | Expected evidence | Actual evidence | Gap? |
|---|---|---|---|---|
| FR1 — phases, registration, logbook, submission | Core administration questions | Authoritative Chapter 1 rules | **50 chunks**, pp. 7–28 | No gap for prose rules; pp. 10/16/18 unreadable remain a hole |
| FR2 — report structure, page limits, Harvard | Core report questions | Authoritative Chapter 3 rules | **40 chunks**, pp. 30–44 | No major gap; table flattening noted |
| FR5 — forms and submission/evaluation procedures | Form accuracy criterion ≥ 90% | Seven appendix forms, pp. 95–119 | **0 chunks** — appendices not ingested | **Yes — genuine coverage gap** |
| RAI1 — verifiable citations | Every answer needs a source reference | Section-level locator on each chunk | Page number only | **Partial** — citation is page-level, not section-level |

Distribution by source: `FYP_Handbook.pdf` 90 chunks (100%). Distribution by chapter:
Chapter 1 50 (55.6%), Chapter 3 40 (44.4%). See `docs/week4_coverage_by_source.png`.

### Bias risks with evidence

**Risk 1 — Representation bias: form procedures absent (evidence-backed).**
- **Origin:** source selection and the Week 3 page-range decision (pipeline), not the source
  handbook itself — the forms exist in the PDF but were excluded because their layout
  fragments on extraction.
- **Propagation path:** source forms excluded → prepared corpus has no form fields → retrieval
  has nothing to return for "which form do I need?" → generation either declines or invents a
  plausible-but-unsourced answer → a student uses the wrong form or misses a step.
- **Who is affected:** FYP1/FYP2 students asking FR5 questions, especially near submission deadlines.
- **Mitigation:** (a) scope-guard FR5 questions at the interface until forms are ingested;
  (b) add a layout-aware parser for Appendices 1–7 in a later iteration; (c) add a form-question
  subset to the Topic 8 evaluation set so the gap is measured rather than assumed.
- **What remains to test at system level (Topic 8):** form-question accuracy (FR5 threshold
  ≥ 90%) and whether the scope guard fires correctly.

**Risk 2 — Measurement bias: page-level citation treated as sufficient (evidence-backed).**
- **Origin:** the Week 3 chunking decision. Page provenance was made first-class to satisfy
  RAI1, but section numbers were not attached to chunks.
- **Propagation path:** chunk carries `page=25` but no section → retriever returns the chunk →
  the generator cites "p. 25" → the citation is technically present and verifiable at page
  level, but the answer may quote a different rule on that page → RAI1's citation-source match
  (≥ 95%) can be overstated because the unit of verification is coarser than the unit of answer.
- **Who is affected:** any student relying on the cited location to double-check a rule.
- **Mitigation:** (a) attach heading/section labels to each chunk before retrieval (Topic 7);
  (b) re-measure citation-source match at section granularity; (c) mark page-only citations as
  lower-confidence in the response.
- **What remains to test at system level (Topic 8):** citation-source match at section level,
  not just page level.

**Risk 3 — Historical/institutional bias: single-source authority (partly N.V.).**
- The corpus reflects one faculty's 2026–2027 handbook. Its rules are authoritative for UTM FAI
  students, which is the stated scope; answers to students outside that scope could be wrong.
  That is a scope boundary rather than demonstrated unfairness. Whether the handbook itself
  contains systematically biased guidance (for example in assessment criteria) is **N.V.** —
  no content-level fairness audit was performed here.
- **Mitigation:** keep the scope guard (FR4) explicit; do not claim applicability beyond UTM FAI.

**Boundary statement:** unequal chunk counts (50 vs 40) and the absence of appendices are
signals and scope facts. They are **potential representation risk**, not proof of system
unfairness. Fairness performance is evaluated in Topic 8.

---

## 5. Privacy, Consent, Rights/Access, and Data Governance

| Governance question | Evidence from this project | Status | Required action / owner |
|---|---|---|---|
| Source owner, version/date, provenance traceable? | Owner: UTM Faculty of Artificial Intelligence. Version: *Handbook FYP BAI 20262027*, PDF created 2026-10-08. Provenance: SHA-256 `38ad807b…` in `data/week3_manifest.json` and every chunk metadata; verified unchanged in Cell 1. | **OK** | Student re-verifies hash after each handbook reissue. |
| Rights/access known and appropriate? | Course-provided handbook, local use only. The PDF and all full-text corpora are git-ignored; `docs/week3_pipeline_decisions.md` §6 records the decision and approved storage location. The licence text itself was not supplied. | **To verify** | Student to confirm redistribution terms with the course before any public release; keep the corpus local meanwhile. |
| Corpus contains unnecessary personal/sensitive data? | No student records. Editorial staff names and the PDF `/Author` metadata are present. These are not student PII but are personal data in a broad sense. RAI3 concern is student PII transmission to external APIs. | **No student PII** | Strip document metadata before any external sharing; re-check in Topic 7 when an external model is introduced. |
| Personal data collected — basis/consent/disclosure? | The project uses institutional documents only. No student data is collected. Consent does not apply to this corpus; it is not added mechanically. | **N/A** | Revisit if user logs or feedback data are collected later. |
| Update / superseded-version / retention / removal defined? | Version change is detected by SHA-256 mismatch (R5). Full re-run regenerates the corpus; NFR3 target is < 15 min. No automated reissue watch and no retention schedule exist. | **Gap** | Student owns re-verification each semester; define retention and removal before deployment. |

**Governance boundary:** the handbook being available to students does not make it public
domain. Rights are recorded as *to be verified* rather than assumed. The full prepared and
cleaned corpora remain local; only schemas, checksums, counts, samples with truncated text,
and this report are committed. See `docs/week3_pipeline_decisions.md` §6 for the reversal
procedure if the course confirms redistribution is permitted.

---

## 6. Mitigation Plan and Residual Risk

| Issue | Evidence | Action | Owner / timing | Residual risk |
|---|---|---|---|---|
| FR5 form gap (0 appendix chunks) | Cell 6 coverage; Week 3 inventory §4 | Scope-guard FR5 questions; add layout-aware appendix parser; add form questions to Topic 8 test set | Student, before Topic 7 retrieval build | High until fixed — FR5 threshold ≥ 90% cannot be met for form-content questions |
| Accuracy dimension N.V. | No source-to-chunk comparison run | Sample 20 chunks and verify each against its source page; record agreement | Student, Topic 5 | Medium — no corpus-wide accuracy claim may be made until complete |
| Page-only citations | Chunk metadata has `page`, no section | Attach section labels; re-measure RAI1 at section granularity | Student, Topic 7 | Medium — citation-source match may be overstated at page level |
| Four short chunks | 4/90 warnings, all checked against source | Retain; review retrieval value when embeddings are added | Student, Topic 7 | Low — documented valid headings |
| Rights/access unverified | No licence text supplied | Confirm with course; keep corpora local | Student, before any publication | Medium — blocks public release, not local development |
| Stale handbook after reissue | R5 rule + SHA-256 | Re-run pipeline and this notebook on each new issue | Student, each semester | Low if followed; NFR3 < 15 min is met by a full re-run |
| Unreadable pages 10/16/18 | Week 3 inventory §3 | OCR in a later iteration; record as known hole meanwhile | Student, Topic 7+ | Medium — three pages of Chapter 1 are unavailable to retrieval |
| No retention/removal policy | §5 governance row | Define before deployment | Student, before deployment | Low at proposal stage |

---

## 7. Impact on Topic 2 Requirements / Evaluation Criteria

| Requirement / criterion | Threshold | Effect of unresolved data issue |
|---|---|---|
| **FR1** — phases, procedures, milestones | — | Mostly supported by 50 Chapter 1 chunks; pages 10/16/18 remain an evidence hole, so some procedure questions may be unanswerable. |
| **FR2** — report structure, page limits, referencing | — | Supported by 40 Chapter 3 chunks. Table flattening can merge adjacent rows; answers quoting tables must be checked. |
| **FR3 / RAI1** — citation presence and source match | 100% presence; ≥ 95% match | Presence is supported by 100% page-tagged chunks. Match at page level is verifiable; match at section level is **N.V.**, so the ≥ 95% threshold should not be reported as final until section labels exist. |
| **FR5 — form/procedure accuracy** | ≥ 90% correct | **At risk.** With zero appendix chunks, form-content questions cannot be grounded. Procedural questions from Chapter 1 are supported; form-content questions are not. |
| **NFR2 — answer correctness** | ≥ 85% | Cannot be demonstrated in Topic 4. Accuracy is N.V. here; the score belongs to Topic 8 evaluation. |
| **NFR3 — maintainability** | < 15 min update | Supported: SHA-256 detects a reissue and the full pipeline re-runs in seconds. |
| **RAI2 — out-of-scope refusal** | ≥ 95% | Not a data-quality issue; depends on the future system guardrail, but the FR5 gap makes a form-related refusal the honest behaviour for out-of-corpus form questions. |
| **RAI3 — no student PII to external APIs** | 0 transmissions | Corpus contains no student PII (§5). Risk reappears in Topic 7 when an external model is introduced; document metadata should be stripped there. |

---

## 8. Topic 4 Quality Gate Decision

**GO WITH CONDITIONS.**

**Basis.** All eight validation rules relevant to structural quality passed except the
deliberate short-chunk warning, whose four cases were individually inspected against their
source pages and retained with reasons. There are zero missing-text, missing-provenance,
corrupt, stale, inconsistent, noisy-whitespace and duplicate records. Provenance is complete
and the cleaned hand-off was verified byte-for-byte across JSONL and PKL (90/90, identical
text sequence).

**Why not a plain GO.** FR5 has no evidence base for form-content questions (0 appendix
chunks), the accuracy dimension is N.V., citation granularity is coarser than the answer
unit, and redistribution rights remain to be verified. These are material and documented.

**Why not HOLD.** None of these makes downstream use inappropriate for the project's current
stage. The proposal stage (Topic 5, Week 7) requires a justified design, not a deployed
system, and the gap is bounded to a known requirement with an owner and a test plan.

**Conditions attached to the GO**

1. **C1 — FR5 scope guard.** Until Appendices 1–7 are ingested, form-content questions must
   be declined or answered only from Chapter 1 procedure text, with the limitation stated.
   Owner: student, at Topic 7 build.
2. **C2 — Reissue check.** Re-verify the source SHA-256 and re-run both notebooks whenever a
   new handbook issue is released. Owner: student, each semester.
3. **C3 — Rights verification.** Keep full-text corpora local until redistribution terms are
   confirmed in writing. Owner: student, before any publication.
4. **C4 — Accuracy evidence.** Complete a 20-chunk source comparison before claiming any
   corpus accuracy. Owner: student, before Topic 8 evaluation.
5. **C5 — Section-level citation.** Attach section labels before reporting RAI1's ≥ 95%
   citation-source match as final. Owner: student, Topic 7.

A GO decision does not mean the dataset is perfect. It means the remaining limitations and
risks are understood, documented, owned and controlled for the next engineering stage.

---

## Artefacts

| Artefact | Path | Status |
|---|---|---|
| Executed quality notebook (7 code cells) | `notebooks/week4_quality.ipynb` | Committed |
| Validation failure evidence | `docs/week4_validation_failures.csv` | Committed |
| Duplicate review evidence | `docs/week4_duplicate_review.csv` | Committed (header only — 0 rows) |
| Coverage plot | `docs/week4_coverage_by_source.png` | Committed |
| Report | `docs/week4_data_quality_report.md` | This file |
| Reviewed corpus, inspectable | `data/prepared_chunks_clean.jsonl` | Local only — git-ignored (verbatim handbook text) |
| Reviewed corpus, notebook-compatible | `data/chunks_clean.pkl` | Local only — git-ignored |
| Reduced schema sample | `data/prepared_chunks_sample.jsonl` | Committed |

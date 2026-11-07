# Data Inventory — FYP Assistant

**Course:** AI Engineering — Topic 3 Lab (Week 3)
**Project:** FYP Assistant — document-grounded assistant for FYP administration questions
**Source of truth:** *FYP Handbook 2026–2027*, Faculty of Artificial Intelligence, Universiti Teknologi Malaysia
**Pipeline:** `notebooks/week3_pipeline.ipynb` (v `week3-v1`)
**Inventory status:** sources identified, inspected and measured; quality review deferred to Topic 4

---

## 1. Step A1 — From Topic 2 requirements to data needs

Only the requirements that actually consume project data are listed. Each row names the
information the requirement needs, the source that supplies it, and the constraint that
makes that source usable or awkward.

| Requirement ID | Needed information / feature | Candidate source | Why usable / constraint |
|---|---|---|---|
| **FR1** — answer questions on FYP1/FYP2 phases, procedures and milestones | Phase sequence, registration rules, supervisor allocation, logbook meeting rules | `FYP Handbook.pdf`, **pp. 7–28** (Ch. 1) | Authoritative and current (2026–2027 issue, issued by FAI). Constraint: prose in a PDF, no structured export; every rule is embedded in running text, so retrieval must be section-aware. |
| **FR2** — answer questions on report structure, chapter content, page limits and referencing | Report structure per chapter, 50 pp. (FYP1) / 100 pp. (FYP2) limits, Harvard style rules | `FYP Handbook.pdf`, **pp. 30–44** (Ch. 3) | Same authority. Constraint: page limits and reference formats appear in tables and numbered lists, which flatten into ambiguous plain text on extraction. |
| **FR5** — answer questions on forms and submission/evaluation procedures | Form names, form codes, submission and correction steps | `FYP Handbook.pdf`, pp. 95–119 (Appendices 1–7) + pp. 85–93 (Ch. 9) | Forms are fillable-layout pages; extraction yields fragmented labels. **Not ingested in V1** — see §4. |
| **NFR2** — correctness ≥ 85% on 40 curated questions | A stable, citable unit to score answers against | Same PDF, page-level provenance | Requires the page number to survive the pipeline; without it a "correct" answer cannot be attributed to a rule. Drives the page-bound chunking decision. |
| **NFR3** — handbook updates in < 15 min | The handbook is reissued as one replacement PDF | Same PDF, whole file | Re-indexing the single file satisfies the threshold; no incremental parser needed. Constraint: a reissue invalidates every stored SHA-256, so the manifest must be regenerated. |
| **RAI1** — 100% citation presence, ≥ 95% citation-source match | Verifiable chapter/page reference for every answer | Page numbers from the PDF | This is why `page` is a first-class field on every prepared record, not optional metadata. |
| **RAI3** — zero student PII to external model APIs | Confirmation that the corpus carries no student data | Same PDF | The handbook contains staff names in the editorial committee and document metadata (`/Author`), but no student records. Staff names are not student PII but are noted as a low-risk item for Topic 4 review. |

**Deliberately not used:** the Topic 3 dummy dataset pack. The project has a real,
authoritative, in-scope source, so synthetic practice data would add no evidence and
would risk the appearance of demonstrating performance on data the system will never see.

---

## 2. Source inventory

Both rows are **subsets of one file** rather than two separate documents. The lab permits
this ("two representative records/subsets from one dataset where available"), and it is
the honest description: the project has one authoritative source, and splitting it by
chapter is how the requirements divide.

| Source ID | Filename / URL | Type and purpose | Version / date / owner | Access / rights | Inspection evidence |
|---|---|---|---|---|---|
| **S01** | `FYP_Handbook.pdf`, pp. 7–28 | PDF, 22 pp. Chapter 1 — FYP phases, registration, supervisor allocation, logbook rules. Serves FR1, FR5. | *Handbook FYP BAI 20262027*; PDF created 2026-10-08; author metadata "NORMAISHARAH BINTI MAMAT @ MOHD NOR 16056"; issued by UTM FAI. | Course-provided, local use only. Not redistributable; excluded from the public repo via `.gitignore`. | 32,283 chars extracted; 3 pages with no text (10, 16, 18); 12 "logbook" hits. |
| **S02** | `FYP_Handbook.pdf`, pp. 30–44 | PDF, 15 pp. Chapter 3 — report structure, chapter content, page limits, Harvard referencing. Serves FR2. | Same file, same issue. | Same as S01. | 25,430 chars extracted; 0 empty pages; 6 "page limit" hits, 5 "Harvard" hits, 4 "Turnitin/plagiarism" hits. |

**Whole-file properties** (measured, not assumed):

| Property | Measured value |
|---|---|
| Pages | 119 |
| File size | 89,523,826 bytes |
| Extracted characters (all pages) | 187,529 |
| Pages with zero extractable text | 10, 16, 18, 94 |
| Non-empty extracted lines | 4,058 |
| Adjacent duplicate lines | 272 (6.7%) |
| Lines containing repeated text layers | 57 (1.4%) |
| Producer | Canva |
| PDF `/Title` | Handbook FYP BAI 20262027 |

---

## 3. What the inspection actually proved

The point of inspecting before preparing is that these four measurements changed the
Part B design. Each one is a property of *this* file, not a general assumption.

### 3.1 The text is duplicated by the exporter — twice over

The PDF was exported from Canva, which renders each text element twice. Two distinct
patterns appear:

- **Intra-line repetition** — `FINAL YEARFINAL YEARFINAL YEARFINAL YEAR` on the cover,
  `CHAPTER 1: INTRODUCTIONCHAPTER 1: INTRODUCTION` in headings. 57 lines (1.4%).
- **Adjacent identical lines** — 272 lines (6.7%) repeat the line above them.

Uncorrected, every heading is indexed twice and every chunk carries a duplicated copy of
its own text. For a retrieval system this is worse than wasted space: a duplicate chunk
competes with its twin for the same top-k slot, so the retriever returns the same passage
twice and starves a genuinely different passage. Cleaning removed 1.9% of characters
within the V1 scope (8.9% across the whole file, where duplicated headings are denser).

### 3.2 Four pages carry no text at all

Pages 10, 16, 18 and 94 return zero characters. Page 10 sits inside Chapter 1, so S01 has
three unreadable pages. These are almost certainly full-page graphics (process diagrams
and a chapter divider). **They are not silently skipped**: the pipeline reports them, and
they are recorded here because a question whose answer lives only in a diagram on p. 10
cannot be answered by the V1 pipeline and will be scored as a failure unless OCR is added.

### 3.3 Blank lines do not exist, so paragraph detection degenerates

`pypdf` emits one newline per visual line and **no blank line between paragraphs**. This
is the single most consequential finding: the standard "split on blank lines" chunker has
almost nothing to split on. It produced 132 chunks from 56,595 characters, with a minimum
chunk length of 13 characters — fragments, not paragraphs. The measured comparison is in
`docs/week3_pipeline_decisions.md`.

### 3.4 The expected rule content is present and findable

A coverage probe over the whole source confirmed the corpus is not merely readable but
contains the rules the requirements depend on: `logbook` 12 hits, `page limit` 6,
`Harvard` 5, `Turnitin/plagiaris` 4. An empty or purely visual PDF would have passed a
"file loaded" check and failed this one.

---

## 4. Known limitations of this source

These are recorded as scope decisions, not defects to be discovered later.

1. **Chapters 4–9 are not ingested.** They cover report-writing guidance for the
   methodology, design, results and conclusion chapters. They serve no FR in the V1 scope
   because V1 answers *administration* questions (what the rule is, which form, when),
   not *how to write* questions. Deferred to the iteration that widens FR2.
2. **The appendices (pp. 95–119) are not ingested.** They are the fillable forms behind
   FR5, and extraction fragments their labels. FR5 is therefore only partially grounded in
   V1: questions about *procedures* are answerable from Ch. 1, questions about *form
   contents* are not. This must be stated in the evaluation, or FR5's accuracy will be
   measured against evidence the system never had.
3. **Page is the finest citation granularity.** Section numbers appear in the text but are
   not reliably associated with chunks, so a citation reads "p. 14" rather than
   "p. 14, §1.4.2". RAI1's citation-source match is therefore measured at page level.
4. **No OCR.** The four text-free pages stay unreadable.
5. **Tables flatten.** Page limits and reference formats sit in tables; extraction
   interleaves cell text without row boundaries. Answers quoting a table may merge
   adjacent rows.

---

## 5. Access, rights and storage

| Item | Decision |
|---|---|
| Redistribution | The handbook is course-provided and not redistributable. `FYP Handbook.pdf` is listed in `.gitignore` and is never committed. |
| Prepared output | `data/processed/prepared_text.jsonl` and `data/pipeline_v1/documents/prepared_text.jsonl` contain verbatim handbook text, so both are git-ignored. A truncated 5-record sample, the Part B lineage manifest and the Part C `pipeline_manifest.json` are committed instead. See the trade-off in `docs/week3_pipeline_decisions.md` §6. |
| Personal data | No student data is present. Staff names appear in editorial metadata only; not used as retrieval content. |
| Storage | Local copy in the repo working directory; Colab runtime copy under `/content/week3_project/data/raw`. Both are outside the public repository. |

---

## 6. What Topic 4 must add

- An OCR or vision pass for pp. 10, 16, 18, 94 so the unreadable pages stop being silent gaps.
- A quality review of the 90 prepared records — spot-check that each chunk is self-contained
  and that no chunk merges two unrelated rules.
- The appendices, ingested with a layout-aware parser, before FR5 is evaluated.
- A decision on whether staff names in metadata need redaction under the privacy review.

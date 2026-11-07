# Pipeline Decisions — FYP Assistant (Week 3, Part A + Part B)

**Course:** AI Engineering — Topic 3 Lab (Week 3)
**Notebook:** `notebooks/week3_pipeline.ipynb`
**Prepared artefact:** `data/processed/prepared_text.jsonl` (90 records)
**Lineage:** `data/week3_manifest.json`

Every decision below is recorded with the evidence that produced it. Where a decision
deviates from the lab's default, the deviation and its justification are stated explicitly
rather than left implicit.

---

## 1. Pathway selection

**Selected: Pathway A — PDF / text. Pathways B–F are not run.**

| Pathway | Run? | Reason |
|---|---|---|
| A — PDF/TXT | **Yes** | The entire project corpus is document text. |
| B — CSV/Excel | No | No requirement consumes tabular project data. A table of page limits lives *inside* the PDF, so it is handled by A. |
| C — Computer vision | No | No image data in the project scope. |
| D — JSON/API | No | The handbook has no API; there is no authorised endpoint to call. |
| E — Audio | No | No audio data. |
| F — Time-series | No | No dated records or sensor logs. |

The lab permits a single-modality project and requires the choice to be justified by
Topic 2 requirements, not by convenience. Running all six pathways would have produced six
artefacts and no evidence of fitness for purpose.

---

## 2. Cleaning decisions

The extraction produced three artefacts of the PDF exporter, each handled by one explicit
rule. Nothing was removed "because it looked noisy" — each rule targets a measurable defect.

| # | Rule | Targets | Measured effect |
|---|---|---|---|
| C1 | Collapse a line that is the same unit repeated 2–5 times | Canva duplicates each text element: `FINAL YEARFINAL YEARFINAL YEARFINAL YEAR` | 57 lines (1.4%) affected |
| C2 | Drop a line identical to the line immediately above it | Duplicated text layers rendered as separate lines | 272 lines (6.7%) affected |
| C3 | Normalise runs of spaces/tabs to one space | Justified-text spacing and table alignment | Applied throughout |

**Net effect within the V1 scope:** 57,713 raw characters → 56,595 cleaned characters
(−1.9%). Across the whole 119-page file the reduction is 8.9%, because duplicated headings
are denser in the front matter and chapter openers.

**What was deliberately *not* cleaned:** the dotted leader lines in the table of contents
and the running header/footer text. Removing them is tempting, but the TOC is the only
place that maps a section number to a page, which is exactly what a page-level citation
scheme needs later. They stay, and their cost is recorded in §6.

---

## 3. Chunking comparison (Step B3)

Three configurations on identical cleaned input. "Cut mid-sentence" means the chunk does
not end on terminal punctuation, so a reader (or a retrieval model) loses the end of the
thought.

| Config | Method | Chunks | Mean len | Min / Max | Cut mid-sentence | Page provenance |
|---|---|---|---|---|---|---|
| **A** | fixed 500 chars, whole document | 114 | 497 | 167 / 500 | **110 (97%)** | 0% |
| **B** | paragraph-aware 500 chars, blank-line split (lab default) | 132 | 429 | **13** / 500 | **122 (93%)** | 0% |
| **C** | paragraph-aware 800 chars, line-boundary, page-constrained | 90 | 628 | 31 / 800 | **60 (67%)** | **100%** |

### Why B fails here

B is the lab's default and it underperforms A on the metric that matters. `pypdf` emits no
blank lines, so `re.split(r"\n\s*\n", text)` finds almost nothing to split on: most
"paragraphs" are single visual lines, and the chunker emits 132 fragments with a minimum
length of 13 characters. B is not a bad algorithm — it is an algorithm whose input
assumption this PDF does not satisfy. That is precisely why the comparison was run rather
than assumed.

### Why C was selected

| | |
|---|---|
| **Benefit** | Every record carries a `page` number, so an answer can cite *FYP Handbook, p. 14* — the evidence FR3 and RAI1 require. Chunks never straddle a page boundary, so a citation is never half-right. Mean chunk length rises to 628 chars and mid-sentence cuts fall from 97% to 67%. |
| **Accepted limitation** | Fewer, longer chunks than A (90 vs 114), so each retrieval hit covers more text and precision may drop; and a page is the coarsest citable unit, so section-level citation (§1.4.2) is still approximate until heading detection is added. |

67% of chunks still end mid-sentence. That is honest and expected: a page's last chunk
ends wherever the page ends. Fixing it needs sentence-boundary detection, which is a
Topic 4 improvement, not something to fake here.

---

## 4. Stage contracts

Each stage is a contract with a failure response, so a silent bad run is not possible.

| Stage | Contract / evidence | Failure response | Status |
|---|---|---|---|
| Ingest | `FYP_Handbook.pdf` present under `data/raw/`, 89,523,826 bytes, listed by Cell 3 | Stop — `assert files` | **Pass** |
| Inspect | 119 pages, 187,529 chars extracted, 4 text-free pages identified, rule content confirmed present | Stop or flag invalid structure | **Pass** |
| Prepare | Cell 5A1/A2 produce `prepared_text.jsonl`, 90 records, 78,895 bytes | Stop — asserts on empty output and empty chunks | **Pass** |
| Validate | File exists, non-empty, record count matches export, no empty chunk | Stop | **Pass** |
| Record lineage | Cell 6A/6B: source→output pair with SHA-256 on both sides | Stop — `assert lineage_entries`, checksum comparison | **Pass** (1 entry verified) |

### Part C — executed pipeline contracts

Part C re-implements the same preparation inside a registered, re-runnable pipeline
(Cells 7–9) and proves it reproduces the Part B decision.

| Stage | Contract / evidence | Failure response | Result |
|---|---|---|---|
| Ingest | `resolve_source("FYP_Handbook.pdf")` returns an existing file, not a guessed one | `FileNotFoundError` | **Pass** |
| Inspect | `inspect_documents()` reports 119 pages total, 37 in scope, pp. 10/16/18 text-free | `ValueError` on empty text | **Pass** |
| Prepare | `prepare_documents()` writes `data/pipeline_v1/documents/prepared_text.jsonl` | `ValueError` if no chunks | **Pass** — 90 records |
| Validate | `execute_pipeline()` asserts each output exists and is non-empty, then records SHA-256 and byte size | `FileNotFoundError` / `ValueError` | **Pass** — 1 output, 64,666 bytes |
| Record lineage | `pipeline_manifest.json` written with configuration, method, source/output hashes and mapping | — | **Pass** |
| Repeatability | Cell 9 re-executes the pipeline; source fingerprints, output fingerprints, sizes and mappings must match | `AssertionError` per pathway | **PASS: documents** |

**Reproduction check:** the Part C chunk sequence is byte-identical to the Part B output —
90 records, same text order, same page sequence. Part C does not re-decide the
configuration; it proves the decision is executable.

**One deviation from the lab template, stated openly.** Cell 8A's template supports only
`fixed` and `paragraph-aware` and writes `page: None`. This project selected a
page-constrained variant in Part B, so `prepare_documents()` rejects any other method and
records the page on every chunk. Additionally, `portable_path()` in Cell 7 stores
project-relative paths in the manifest instead of absolute ones, so the committed
`pipeline_manifest.json` reads `data/raw/FYP_Handbook.pdf` rather than a machine-specific
absolute path. Both changes are visible in the code and in the manifest's `configuration`
block — nothing about the run is hidden.

---

## 5. Pipeline execution decisions (Part C)

| Decision | Selected | Alternative rejected | Why |
|---|---|---|---|
| **ETL or ELT** | **ETL** — clean and chunk *before* the text reaches any model or index | ELT (load raw pages, transform at query time) | The cleaning rules (C1–C3) are deterministic and cheap, and applying them once means the 90 stored records are already the retrieval units. ELT would re-run deduplication on every query and would leave duplicate text layers inside the index, where they compete for top-k slots. |
| **Batch or streaming** | **Batch** | Streaming / incremental ingestion | The source is a 119-page document reissued as a whole file a few times a year. There is no continuous feed, so streaming would add state and failure modes with no requirement behind it. NFR3 (update < 15 min) is comfortably met by a full re-run: the pipeline completes in seconds. |
| **Orchestration** | Single registered function (`register_and_run`) with explicit stage functions | A workflow engine (Airflow/Prefect) | One source, one pathway, three stages — an orchestrator would be more infrastructure than pipeline. The registered runner already gives the repeatability check something to re-execute. |
| **Output location** | `data/pipeline_v1/documents/` | Overwrite `data/processed/` | The lab keeps Part B and Part C artefacts separate, and so does this project: `data/processed/` holds the Part B preparation, `data/pipeline_v1/documents/` holds the executed pipeline output plus `pipeline_manifest.json`. Keeping both means a regression in either is visible. |
| **Path storage in the manifest** | Project-relative (`data/raw/FYP_Handbook.pdf`) | Absolute paths | An absolute path is machine-specific and would make the committed manifest meaningless outside the machine that produced it. Relative paths make the artefact portable and identical in Colab and locally. |

The diagram for this pipeline is `docs/week3_pipeline_diagram.png`: it expands the DATA
component of the Week 2 V1 architecture into SOURCE → INGEST → INSPECT → PREPARE /
TRANSFORM → VALIDATE OUTPUT → RECORD LINEAGE, annotated with three failure points and the
response to each.

## 6. Data rights trade-off

`prepared_text.jsonl` contains 56,539 characters of verbatim handbook text. The repository
is public and the handbook is course-provided, not redistributable — the same reason
`FYP Handbook.pdf` is already in `.gitignore`.

| Option | Consequence |
|---|---|
| Commit the full prepared file | Satisfies the deliverable list literally; republishes copyrighted university text publicly |
| Commit nothing | Protects the source; leaves no evidence the pipeline produced output |
| **Commit a truncated sample + full manifest (chosen)** | Shows the record schema, the processing label and the checksums; keeps the text local |

Committed: `data/processed/prepared_text_sample.jsonl` (first 5 records, `text` truncated to
300 chars, `record_id`/`page`/`source_sha256` intact), the complete `week3_manifest.json`,
and `data/pipeline_v1/documents/pipeline_manifest.json` (paths, hashes and counts only — no
handbook text). The full prepared files exist locally at
`data/processed/prepared_text.jsonl` and `data/pipeline_v1/documents/prepared_text.jsonl`,
both git-ignored.

### Decision

**The full prepared output stays out of the public repository.** This follows the lab's own
Step C4b provision: *"If restricted data cannot be uploaded, commit non-sensitive evidence
and document the approved storage location."* The handbook is course-provided and not
redistributable, so the prepared text is restricted data by that definition.

### Approved storage location

| Item | Location | Access |
|---|---|---|
| Source | `FYP Handbook.pdf` at the repository root, local only (git-ignored). The notebook reads it from `data/raw/FYP_Handbook.pdf`, so place a copy there before running | Course-provided copy, held by the student |
| Part B output | `data/processed/prepared_text.jsonl` (local only) | Same machine as the source; regenerated by Cells 4–5A2 |
| Part C output | `data/pipeline_v1/documents/prepared_text.jsonl` (local only) | Regenerated by Cell 8A |
| Non-sensitive evidence (committed) | `data/week3_manifest.json`, `data/pipeline_v1/documents/pipeline_manifest.json`, `data/processed/prepared_text_sample.jsonl` | Public repository |

The committed evidence is sufficient to verify the withheld files without holding them:
both manifests carry the source SHA-256, the output SHA-256, the record count and the
processing string, and Cell 9 re-derives the output and compares hashes. Reproducing the
full file takes one run of `notebooks/week3_pipeline.ipynb` with the PDF in `data/raw/`;
that run was already shown to be byte-identical to the Part B output.

**To reverse this decision:** delete the two `prepared_text.jsonl` ignore lines in
`.gitignore`, then `git add -f` the files. Make the repository private first if the
handbook must stay non-public.

---

## 7. Known limitations carried into Topic 4

1. **Three pages inside the V1 scope are unreadable** (pp. 10, 16, 18 — zero extracted
   characters). Any question answered only by a diagram on those pages will fail, and the
   failure is a pipeline gap, not a model error. OCR is the fix.
2. **67% of chunks end mid-sentence.** Sentence-boundary chunking, or overlap, is the next
   improvement.
3. **TOC chunks carry dotted leaders.** They are low-value retrieval targets that still
   occupy index space; ~3 of 90 records are TOC pages.
4. **Page-level, not section-level, citation.** RAI1's citation-source match must be
   measured at page granularity until headings are detected and attached to chunks.
5. **Appendices not ingested**, so FR5 is only partially grounded — see
   `docs/week3_data_inventory.md` §4. Measured: pp. 95–119 yield 513 lines but only 18
   prose-like lines; 98 lines end in a colon (field labels) and 43 are pure dotted-leader
   fill. Mean line length is 23.5 characters against 41.5 in the ingested body, and 8 pages
   carry tripled text layers (`Appendix 1Appendix 1Appendix 1`). These are blank forms, so
   the extraction problem is layout, not OCR: the fix is a layout-aware parser that
   reconstructs field/value pairs.

---

## 8. Traceability to Topic 2

| Objective | Requirement | Data source | Prepared artefact | Evaluation criterion |
|---|---|---|---|---|
| O1 | FR1, FR3 | S01, pp. 7–28 | `prepared_text.jsonl` (records with `page`) | Source Citation and Support |
| O1 | FR2 | S02, pp. 30–44 | `prepared_text.jsonl` | Answer Correctness |
| O1 | FR5 | S01 + appendices (deferred) | `prepared_text.jsonl` (partial) | Form and Procedure Accuracy |
| O2 | RAI1 | S01, S02 | `page` field + `source_sha256` per record | Citation presence / match |
| O3 | NFR2 | S01, S02 | 90 prepared records | Correct response rate ≥ 85% |
| — | NFR3 | whole PDF | `week3_manifest.json` (SHA-256 + `pipeline_version`) | Update time < 15 min |

The `page` field and the per-record `source_sha256` are the two design choices that make
this table enforceable rather than aspirational: without them, an answer cannot be traced
to a rule, and a reissued handbook cannot be detected as stale.

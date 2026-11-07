# FYP Assistant
> An AI assistant that answers Final Year Project (FYP) administration questions — timelines, forms, ethics, and policies — grounded in official FYP documents, for final-year students.

## Problem Statement
Final-year students struggle to get timely, accurate answers about FYP processes, milestones, required forms, ethics clearance, and faculty policies. This information is scattered across the FYP Handbook, faculty pages, and supervisor guidance. FYP Assistant provides instant, document-grounded answers 24/7.

## Evidence of the Problem
- FYP administration information (timelines, forms, ethics, submission rules) is distributed across multiple official documents rather than one searchable source. (Source: course-provided FYP Handbook.)
- A single FYP administration question often requires consulting several separate pages/documents, raising the chance of missed deadlines. (To be strengthened with an institutional source in Week 2.)

## Initial Related Work / Existing Approaches
- Human FYP coordination / supervisor guidance — contextual but limited by staff availability and inconsistent across supervisors. (Key limitation: does not scale.)
- Static faculty FYP FAQ / policy web pages — authoritative but fragmented and hard to navigate. (Key limitation: not question-answering.)
- General-purpose AI assistants (e.g., ChatGPT) — flexible but not grounded in official FYP documents and may hallucinate. (Key limitation: answers may not reflect actual faculty rules; see Part A observation.)

## Engineering Gap / Limitation
Existing approaches are fragmented, depend on staff availability, or may not ground answers consistently in approved FYP documents, risking missed deadlines or incorrect procedure advice.

## Project Objectives
- **O1:** Design a document-grounded AI assistant for FYP administration questions based on official FYP documents.
- **O2:** Develop the assistant with retrieval over FYP documents and appropriate guardrails.
- **O3:** Evaluate answer groundedness, accuracy, and safety.

## Target Users
Final-year undergraduate students navigating FYP processes, milestones, and deadlines (and potentially supervisors / FYP coordinators).

## Data Sources
- FYP Handbook (course-provided PDF) — local copy in `/data`; not publicly committed.
- Faculty FYP Guidelines — https://drive.google.com/file/d/1bNFeGajgc-xN3ET21Ww8GRnMyiS0N8Ms/view
- University thesis writing guidelines — https://drive.google.com/file/d/1L0lUVdK-Fpwnl5XJ7-tKKaf70Jk_j30n/view

## Weekly Progress
| Week | Focus | Deliverable | Status |
|------|-------|-------------|--------|
| W1 | System map + project scope | README + docs | ✅ |
| W2 | Requirements, evaluation criteria, V1 architecture | `docs/week2_requirements.md`, `docs/week2_evaluation_criteria.md`, `docs/week2_architecture_v1.png`, `docs/week2_architecture_decisions.md` | ✅ |
| W3 | Data engineering: inventory, V1 data pipeline, lineage | `docs/week3_data_inventory.md`, `docs/week3_pipeline_decisions.md`, `docs/week3_pipeline_diagram.png`, `notebooks/week3_pipeline.ipynb`, `data/week3_manifest.json`, `data/pipeline_v1/documents/` | ✅ Part A + B + C |

## Week 3 — Data Pipeline

**Pathway:** A (PDF / text) only — the project corpus is the FYP Handbook, and no Topic 2
requirement consumes tabular, image, JSON, audio or time-series data.

**Ingestion scope (V1):** Handbook pp. 7–28 (Ch. 1 — phases, registration, logbook) and
pp. 30–44 (Ch. 3 — report structure, page limits, Harvard). Chapters 4–9 and the appendices
are deferred; see `docs/week3_data_inventory.md` §4.

**Prepared output:** 90 records, mean 628 chars, **100% carrying a page number** so answers
can cite the handbook. Chunking was selected by measurement, not preference — the lab's
default blank-line chunker produced 132 fragments (min 13 chars) because `pypdf` emits no
blank lines; a page-bound chunker cut mid-sentence chunks from 97% to 67%. Full comparison
in `docs/week3_pipeline_decisions.md` §3.

**Executable pipeline:** Cells 7–9 register the preparation as a re-runnable pipeline and
re-execute it to prove repeatability — `PASS: documents`, with a byte-identical chunk
sequence to the Part B output. Diagram: `docs/week3_pipeline_diagram.png`.

**Lineage:** `data/week3_manifest.json` records the source SHA-256, prepared SHA-256 and the
exact processing string (Part B). `data/pipeline_v1/documents/pipeline_manifest.json`
records the pipeline execution — configuration, source/output hashes, mapping and
repeatability evidence (Part C).

> The two `prepared_text.jsonl` files are git-ignored because they contain verbatim
> handbook text; `data/processed/prepared_text_sample.jsonl` holds 5 truncated records as
> schema evidence. See `docs/week3_pipeline_decisions.md` §6.


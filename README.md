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
- Faculty FYP Guidelines webpage — [Add your faculty's public FYP guidelines URL]
- University thesis / ethics submission guidelines — [Add public URL]

## Weekly Progress
| Week | Focus | Deliverable | Status |
|------|-------|-------------|--------|
| W1 | System map + project scope | README + docs | ✅ |

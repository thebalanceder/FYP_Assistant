# Week 1 Project Scoping — FYP Assistant

> Preliminary Week 1 framing. Refined in Topic 2.

## Project Name
FYP Assistant

## Domain
Higher Education / Final Year Project (FYP) administration support

## Problem Statement
Final-year students often cannot get timely, accurate answers about FYP processes, milestones, required forms, ethics/plagiarism rules, and faculty-specific policies. This information is scattered across the FYP Handbook, faculty web pages, and supervisor guidance. An AI assistant grounded in official FYP documents can answer these questions instantly and consistently, 24/7.

## Evidence of the Problem
- **Evidence 1:** Information required for an FYP (timelines, forms, ethics clearance, submission rules) is distributed across multiple official documents rather than one searchable source.
  - Source / citation: Course-provided FYP Handbook (local copy kept in `/data`; not committed to the public repo due to copyright).
- **Evidence 2:** A single FYP administration question often requires consulting several separate pages/documents, increasing the chance of missed deadlines.
  - Source / citation: Common FYP administration practice — to be strengthened with an institutional source in Week 2.

## Initial Related Work (at least 3)
- **Study / solution 1:** Human FYP coordination / supervisor guidance.
  - Source / citation: Course FYP Handbook.
  - Key limitation: Contextual and authoritative, but does not scale and answers vary across supervisors.
- **Study / solution 2:** Static faculty FYP FAQ / policy web pages.
  - Source / citation: [Add your faculty's public FYP guidelines URL here]
  - Key limitation: Authoritative but fragmented across pages and hard to navigate; not question-answering.
- **Study / solution 3:** General-purpose AI assistants (e.g., ChatGPT).
  - Source / citation: Part A observation sheet — ChatGPT showed ambiguity-handling and potential hallucination weaknesses.
  - Key limitation: Flexible but not grounded in official FYP documents; may invent policies.

## Engineering Gap / Limitation
Existing approaches are fragmented, depend on staff availability, or may not ground answers consistently in approved FYP documents — risking missed deadlines or incorrect procedure advice.

## Initial Project Objectives (2–3)
- **O1:** To design a document-grounded AI assistant for FYP administration questions based on official FYP documents.
- **O2:** To develop the assistant with retrieval over the FYP Handbook/policies and appropriate guardrails.
- **O3:** To evaluate the assistant in terms of answer groundedness, accuracy, and safety.

## Target Users
Final-year undergraduate students (and potentially supervisors / FYP coordinators) navigating FYP processes, milestones, and deadlines.

## Data Sources (at least 2)
- **Source 1:** FYP Handbook (course-provided PDF) — local copy in `/data`; not publicly committed.
- **Source 2:** Faculty FYP Guidelines webpage — URL: [Add your faculty's public FYP guidelines URL]
- **Source 3 (if any):** University thesis / ethics / Turnitin submission guidelines — URL: [Add public URL]

## Similar to EduAdvisor
Same pattern as EduAdvisor: a document-grounded Q&A assistant for a university administrative domain.

## Different from EduAdvisor
Focuses on the FYP lifecycle (proposal → ethics → progress → submission → examination) rather than course-prerequisite advising, and works from a smaller, well-bounded document set.

## Responsible AI Risks (at least 2)
- **Risk 1 — Hallucination:** The model invents a deadline or form requirement not in the documents → a student misses a milestone.
- **Risk 2 — Outdated information:** If the FYP Handbook changes and the assistant is not re-indexed, it gives stale rules → wrong procedure followed.
- **Risk 3 — Overtrust:** A student treats guidance as a completed action (e.g., believes submission was done) → missed deadline. (Connects to Part B failure point F6.)

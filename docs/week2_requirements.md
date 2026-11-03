# Requirements Specification — FYP Assistant

**Version:** 1.0
**Course:** AI Engineering — Topic 2 Lab (Week 2)
**Domain:** Higher Education / Final Year Project (FYP) administration
**Source of truth:** *FYP Handbook 2026–2027*, Faculty of Artificial Intelligence, Universiti Teknologi Malaysia (UTM).

---

## Section 0 — Project Framing & Application Scope Checkpoint

| Framing element | Confirmed content |
|---|---|
| **Problem Evidence** | The FYP Handbook 2026–2027 (119 pp.) spreads FYP rules across Chapter 1 (phases, registration, supervisor assignment, logbook, report submission, presentation, correction), Chapter 3 (report structure; page limits of 50 pp. for FYP1 and 100 pp. for FYP2; Harvard referencing), and the appendix forms (proposal consent, title change, report review, no-plagiarism / Turnitin ≤ 20% per chapter, response-to-comment, correction). A student must locate the right rule across many sections. Supporting evidence: the Topic 1 Part A observation that a general chatbot can answer confidently without grounding (ambiguity / hallucination risk). |
| **Existing Approaches** | (1) **Human supervisor / coordinator guidance** — authoritative and contextual, but limited by staff availability. (2) **Static FYP Handbook PDF / faculty pages** — authoritative but long (119 pp.) and fragmented across chapters and forms; manual search is slow. (3) **General-purpose AI assistants (e.g., ChatGPT)** — fast and flexible, but not grounded in the official handbook and may invent deadlines or forms. |
| **Project / Engineering Gap** | Students must either manually search a long, fragmented handbook or rely on an ungrounded chatbot. No lightweight assistant provides **source-cited, scope-controlled answers strictly grounded in the official FYP Handbook**, so students risk missing deadlines, using the wrong form, or following outdated rules. |
| **Objective O1** | To design a document-grounded FYP assistant based on the official FYP Handbook. |
| **Objective O2** | To develop the assistant with retrieval over the handbook and appropriate scope/safety guardrails. |
| **Objective O3** | To evaluate answer groundedness, accuracy, and safety against the handbook. |
| **Intended Users** | FYP1 and FYP2 students of the Bachelor in Artificial Intelligence with Honours (UTM FAI); secondarily supervisors / coordinators for quick reference. |
| **Supported Tasks** | Answering questions about: FYP phases and milestones; registration and supervisor assignment; logbook rules (minimum 7 meetings per semester, at least once every two weeks); report structure, chapter content, page limits, and Harvard referencing; required forms and submission/evaluation procedures; Turnitin / plagiarism limit (≤ 20% per chapter); presentation and correction procedures; project title-change procedure. |
| **Information / Knowledge Domain** | FYP administration and academic regulations as documented in the FYP Handbook 2026–2027. |
| **Operating Context** | A web/chat interface used by students on or off campus, especially in the periods before key milestones (registration, report submission, presentation, correction). |
| **Important Constraints** | Answers must be grounded in and cite the official handbook; the handbook is copyrighted (local use only, not redistributed publicly); the system must not invent rules/forms/deadlines; it must support handbook updates; interactive latency must stay low. |
| **Explicit Out-of-Scope Tasks** | The system must NOT give medical, legal, or mental-health advice; must NOT decide grades, approve extensions, grant waivers, or act as an official authority; must NOT replace supervisor sign-off or guarantee submission/approval; must NOT complete official submissions on the student's behalf. |
| **Evaluation Focus** | Determine whether the assistant gives accurate, source-cited answers to FYP administration questions, refuses out-of-scope queries, and remains grounded in the handbook. |

---

## Section 1 — Functional Requirements

| # | Requirement — The system SHALL | Scope Boundary — SHALL NOT (where relevant) |
|---|---|---|
| **FR1** | Answer questions about FYP1/FYP2 phases, procedures, and milestones using only the official FYP Handbook. | Invent phases, deadlines, or procedures not present in the handbook. |
| **FR2** | Answer questions about report structure, chapter content, page limits, and referencing format as specified in the handbook. | Give formatting advice that contradicts the handbook (e.g., wrong page limits or citation style). |
| **FR3** | Provide a source reference (chapter and/or section) for every grounded answer. | Present unsupported information as if it came from the handbook. |
| **FR4** | Decline out-of-scope queries (grade decisions, extension approval, waivers, medical/legal/mental-health, official sign-off) and redirect the user appropriately. | Decide grades, approve extensions, grant waivers, or give medical/legal/mental-health advice. |
| **FR5** | Answer questions about required forms and submission/evaluation procedures (logbook meetings, Turnitin limit, report submission, presentation, correction) accurately. | Claim to submit forms or complete official actions on the student's behalf. |

---

## Section 2 — Non-Functional Requirements

| # | Quality Attribute | Requirement | Measurement Method | Metric | Threshold |
|---|---|---|---|---|---|
| **NFR1** | Performance | The system shall keep interactive responses within the agreed latency target. | Measure end-to-end latency on a defined workload of 50 FYP queries. | p90 latency | < 3,000 ms |
| **NFR2** | Correctness | Answers shall be factually consistent with the FYP Handbook. | Score a curated set of 40 FYP questions against the handbook using a defined rubric. | Correct response rate (%) | ≥ 85% |
| **NFR3** | Maintainability | Handbook updates shall be incorporated without rebuilding unrelated components. | Run a controlled document-update test (replace one chapter, re-index). | Update time | < 15 min |
| **NFR4** | Cost | Inference cost shall remain within the project budget. | Record model/API usage over the evaluation workload. | Cost per 100 queries | < US$0.50 per 100 queries (illustrative) |
| **NFR5** | Reliability | The system shall remain available during peak pre-deadline periods. | Monitor uptime over a defined period covering key deadlines. | Uptime (%) | ≥ 99% during business hours |

---

## Section 3 — Responsible AI Requirements

| # | Responsible AI Risk | Requirement | Measurement Method | Metric | Threshold |
|---|---|---|---|---|---|
| **RAI1** | Transparency / Grounding (hallucination) | The system shall provide a verifiable source reference (chapter/section) for every grounded answer and indicate when sufficient evidence cannot be found. | Evaluate document-grounded responses for citation presence and whether the cited section supports the answer. | Citation presence rate; citation-source match rate | 100% citation presence; ≥ 95% citation-source match |
| **RAI2** | Safety / Scope | The system shall refuse out-of-scope queries (grades, extensions, waivers, medical/legal/mental-health, official sign-off) and redirect users appropriately. | Run a defined safety/scope test set containing at least 20 out-of-scope queries. | Refusal-and-redirect success rate | ≥ 95% |
| **RAI3** | Privacy | The system shall not transmit student PII to external model APIs. | Run privacy test cases and inspect outgoing model/API payloads. | Number of detected PII transmissions | 0 |
| **RAI4** | Fairness | System performance shall not differ beyond the agreed tolerance across relevant student groups (FYP1 vs FYP2). | Evaluate a stratified test set and compare correctness rates across groups. | Absolute correctness-rate gap between groups | ≤ 10 percentage points |

---

## Reflection (Part A)

**Which Responsible AI requirement will be the most difficult to measure or enforce technically? Why?**

**RAI1 (Transparency / Grounding).** It is relatively easy to force the model to *attach* a citation, but much harder to guarantee that the cited section actually *supports* the claim. Verifying support requires a curated gold set of questions with known correct handbook sections and a matching procedure, and the handbook's tables and form layouts make reliable retrieval difficult. In practice, a citation can be present yet point to a section that does not answer the question — so measuring "citation-source match" is the real engineering challenge, not merely "citation presence".

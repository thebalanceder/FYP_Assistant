# V1 Architecture Decisions & Traceability — FYP Assistant

**Course:** AI Engineering — Topic 2 Lab (Week 2)
**Diagram:** `docs/week2_architecture_v1.png`
**Principle:** Start simple. Add components only when a requirement justifies them.

---

## Step C2 — Architecture Needs from Requirements

| Requirement ID | What the requirement needs | Architecture Response |
|---|---|---|
| **FR1 / FR2 / FR5** | Answers grounded in the official handbook (phases, report rules, forms/procedures). | Add a **Knowledge Source** (indexed FYP Handbook) and a **Retrieval / Knowledge Access** component. |
| **FR3 / RAI1** | Verifiable source references and honest "not found" behaviour. | Add a **Grounding / Citation Control** stage after generation. |
| **FR4 / RAI2** | Refusal of out-of-scope queries and redirect. | Add an explicit **Scope / Safety Control** at the input. |
| **NFR1** | Low interactive latency. | Keep the pipeline lean; retrieve only top-k relevant sections before generation. |
| **NFR3** | Handbook updates without rebuilding everything. | Keep the **Knowledge Source** index separable and independently refreshable. |
| **RAI3** | No student PII sent to external model APIs. | Add a **Privacy / PII filtering** control on the outgoing request. |
| **Part B evaluation** | Ability to measure correctness, citations, refusals, latency. | Add an **Evaluation Point** that logs query, retrieved sources, and response. |

---

## Architecture Decision Records (ADR)

| Decision | Requirement / Constraint | Alternatives Considered | Selected Option | Trade-off Accepted |
|---|---|---|---|---|
| **ADR-1: Include a dedicated Knowledge Source + Retrieval** | FR1/FR2/FR5, RAI1 — answers must be grounded in and cite the official handbook. | Base model only; static FAQ page; dynamic retrieval over the handbook. | Dedicated knowledge source + retrieval component. | More system complexity and indexing effort, but far better grounding and traceability. |
| **ADR-2: Include explicit Scope / Safety Control** | FR4/RAI2 — out-of-scope queries must be refused. | Instruction-only prompt; explicit scope-control component. | Explicit scope-control component at input. | Possible false refusals and extra processing, but stronger, testable safety behaviour. |
| **ADR-3: Separate Grounding / Citation Control stage** | FR3/RAI1 — citations must be present and actually support the answer. | Ask the model to cite inline only; post-generation citation verification. | Post-generation citation verification stage. | Adds latency and a verification step, but raises citation-source match reliability. |

---

## Initial Requirements Traceability Matrix

| Objective | Requirement | Evaluation Criterion | Architecture Element | Later Evidence |
|---|---|---|---|---|
| **O1** | FR3 / RAI1 | Source Citation and Support | Knowledge Source + Grounding / Citation Control | Week 5 citation & support test on 40 questions |
| **O2** | FR4 / RAI2 | Out-of-Scope Refusal | Scope / Safety Control | Safety/scope test set (≥ 20 out-of-scope queries) |
| **O3** | NFR2 | Answer Correctness | Language Model Service + Retrieval | Curated FYP Q&A evaluation (≥ 85% correct) |
| **O3** | NFR1 | Response Latency | Retrieval + Language Model Service | 50-query latency run (p90 < 3 s) |
| **O1** | FR5 | Form and Procedure Accuracy | Knowledge Source (forms/procedures sections) | Procedure test set (≥ 90% correct) |

---

## Final Review Checklist

- **Does each major component support a requirement or evaluation need?** Yes — Knowledge Source/Retrieval (FR1/2/5), Scope Control (FR4/RAI2), Grounding Control (FR3/RAI1), Evaluation Point (Part B).
- **Is any important data source, safety control, or evaluation point missing?** The handbook is the sole knowledge source; scope, privacy, and citation controls are present; an evaluation point is included.
- **Any unjustified technology added?** No — components are labelled by function, not by specific model/vector DB/UI framework (deferred to later topics).
- **Is there a simpler architecture that still satisfies the requirements?** The simple `User → Interface → Model → Response` flow cannot ground answers or refuse out-of-scope queries, so the added components are justified.

---

## Reflection (Part C)

**Which architecture decision involved the most important trade-off? Which requirement drove the decision, and what trade-off did you accept?**

**ADR-1 (Knowledge Source + Retrieval).** It is driven by FR1/FR2/FR5 and RAI1 — without grounding, the assistant is just an ungrounded chatbot and cannot cite the handbook. I accepted significantly more complexity (indexing, retrieval tuning, update handling) in exchange for trustworthy, traceable answers. ADR-2 (scope control) is a close second, trading some false refusals for stronger safety.

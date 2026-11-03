# Evaluation Criteria — FYP Assistant

**Course:** AI Engineering — Topic 2 Lab (Week 2)
**Project:** FYP Assistant (document-grounded FYP administration assistant)
**Source of truth:** *FYP Handbook 2026–2027*, UTM Faculty of Artificial Intelligence.

> Flow: Objective / Requirement → Evaluation Criterion → Measurement Method → Metric → Threshold → User / Business Outcome

| Objective / Requirement ID | Evaluation Criterion | Capability Bucket | Measurement Method | Metric | Threshold | User / Business Outcome |
|---|---|---|---|---|---|---|
| **O3 / NFR2** | Answer Correctness | Domain-specific capability | Evaluate a curated set of 40 supported FYP questions against the handbook using a defined rubric. | Correct response rate | ≥ 85% | Students receive sufficiently reliable answers to routine FYP administration questions. |
| **O1 / FR3 / RAI1** | Source Citation and Support | Instruction-following capability | Inspect grounded responses for citation presence and verify whether the cited section supports the answer. | Citation presence rate; citation-source match rate | 100% presence; ≥ 95% match | Students can verify every answer against the official handbook. |
| **O2 / FR4 / RAI2** | Out-of-Scope Refusal | Instruction-following capability | Test with a defined safety/scope set of ≥ 20 out-of-scope queries (grades, extensions, medical/legal, sign-off). | Refusal-and-redirect success rate | ≥ 95% | Reduces inappropriate advice outside the intended administrative scope. |
| **O3 / NFR1** | Response Latency | Cost and latency | Measure end-to-end response time across a defined 50-query workload. | p90 latency | < 3 s | Students get responsive interaction when checking rules close to a deadline. |
| **O1 / FR5** | Form and Procedure Accuracy | Domain-specific capability | Test with questions on forms and procedures (logbook meetings, Turnitin ≤ 20%, submission, presentation, correction) and compare to the handbook. | Correct procedure-answer rate | ≥ 90% | Students follow the correct procedure and use the correct form, reducing missed deadlines. |

---

## Reflection (Part B)

**Which evaluation criterion is most important for determining whether your AI application will be useful to its intended users? Why?**

**Answer Correctness combined with Source Citation and Support (O3/NFR2 + O1/FR3/RAI1).** The whole value of FYP Assistant is that a student can trust an answer about a deadline, form, or rule without re-reading a 119-page handbook. A fast, well-formatted answer that is wrong — or that cites a section which does not actually support it — is worse than no answer, because it can cause a missed submission or the wrong form. Correctness and verifiable grounding therefore decide usefulness; latency and refusal behaviour are important but secondary.

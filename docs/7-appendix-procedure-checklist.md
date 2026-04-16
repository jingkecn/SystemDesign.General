Appendix: Procedure Checklist
=============================

> [!NOTE]
> Use this checklist at the start of any system design session.

- [ ] Step 1 — Requirements Clarification
  - [ ] Functional requirements identified and bounded (3-5 core use cases)
  - [ ] Non-functional requirements quantified (latency, scale, availability)
  - [ ] Out-of-scope explicitly stated
  - [ ] External dependencies identified

---

- [ ] Step 2 — Capacity Estimation
  - [ ] DAU / concurrent users estimated
  - [ ] Read and write RPS calculated
  - [ ] Storage volume estimated with retention period
  - [ ] Bandwidth estimated (inbound and outbound)
  - [ ] Dominant flow identified

---

- [ ] Step 3 — High-Level Architecture
  - [ ] All major components named and bounded
  - [ ] Data flows drawn with protocol labels
  - [ ] Communication patterns stated (sync/async, push/pull)

---

- [ ] Step 4 — Detailed Component Design
  - [ ] High-risk or high-complexity components chosen for deep dive
  - [ ] Public interfaces defined
  - [ ] Data models sketched (key fields)
  - [ ] Happy path and failure paths described

---

- [ ] Step 5 — Trade-offs and Decisions
  - [ ] Decision record written for each significant choice
  - [ ] At least two alternatives considered per decision
  - [ ] Consequences (not just rationale) stated explicitly

---

- [ ] Step 6 — Failure Modes
  - [ ] At least one failure mode per major component
  - [ ] Impact and mitigation stated for each

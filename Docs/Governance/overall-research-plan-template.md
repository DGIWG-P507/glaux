# [Program/Domain] Overall Research Plan

**Version:** 1.0
**Date:** [Month Day, Year]
**Status:** [Draft | Active | Complete]
**Scope:** [Define exact scope boundary]
**Plan Owner:** [Name/Team]
**Final Report Target:** [Path to final overall report file]

---

## Purpose

This document defines the single controlling plan for the full research effort.

It establishes:

- The complete indexed set of research topics
- The required execution order and dependencies
- Objective completion criteria at topic and program levels
- The final overall report requirement

---

## Research Objective

State the exact overall decision or output this full research effort must support.

- Decision(s) this research must enable:
  - [Decision]
  - [Decision]
- Downstream artifacts this research must enable:
  - [Contribution/Goal Definition]
  - [Implementation Guide]
  - [Roadmap]

---

## Governance Rules (Required)

1. No topic research report without a topic research plan.
2. No topic plan without a matching topic ID in this overall index.
3. Work one topic at a time unless a dependency-based reprioritization is explicitly documented.
4. Each completed topic produces exactly one topic research report.
5. Final overall report is produced only after all indexed topics are complete.
6. Every topic report must include explicit references and evidence.

---

## Operating Model

Research lifecycle:

1. Create this overall research plan.
2. Build and finalize the planning topic index in this document.
3. Create one topic research plan per indexed topic.
4. Execute topic research one topic at a time.
5. Produce one topic research report per completed topic.
6. Produce one final overall research report after all topics are complete.

---

## Planning Index of Topics

Use stable IDs and keep titles outcome-oriented.

| Topic ID | Topic Title | Priority | Dependencies | Output Target | Status |
|---|---|---|---|---|---|
| [ID-001] | [Title] | [High/Med/Low or 1..N] | [None or IDs] | [Report path] | [Not Started] |
| [ID-002] | [Title] | [High/Med/Low or 1..N] | [IDs] | [Report path] | [Not Started] |
| [ID-003] | [Title] | [High/Med/Low or 1..N] | [IDs] | [Report path] | [Not Started] |

Add rows until the indexed set is complete.

---

## Topic Execution Order

Default order:

1. [ID-001]
2. [ID-002]
3. [ID-003]
4. [...]

Order may change only when one of the following is recorded in a change entry:

- A dependency requires reordering
- A documented priority decision is approved

---

## Objective Quality Standards

Each topic report must satisfy all standards below.

### Evidence Standard

- Claims are supported by direct references
- References are specific and reproducible (URL, section/clause, artifact path)
- Assumptions are explicitly marked

### Decision-Usefulness Standard

- Conclusion is explicit (not implied)
- Recommendation is actionable and bounded
- Open issues and their impact are clearly documented

### Completeness Standard

- Topic plan scope is fully addressed
- Core questions are answered
- Unresolved items are explicitly listed with next action

---

## Topic Completion Criteria

A topic is complete only when all are true:

- Matching topic plan exists and is in-scope with this overall index
- Topic research report is complete
- Evidence and references are included
- Conclusion is explicit and decision-usable
- Open issues (if any) are documented
- Topic status in index is updated to `Complete`

---

## Program Completion Criteria

The overall research effort is complete only when all are true:

- All indexed topics are marked `Complete`
- All topic reports exist at their declared output targets
- Findings are sufficient to support downstream planning artifacts
- Final overall report is produced and accepted

---

## Final Overall Report Requirement

After all indexed topics are complete, produce:

- [final-overall-research-report.md]

The final report must:

- Respond directly to this overall plan
- Summarize each topic conclusion
- Provide consolidated cross-topic conclusions
- Identify unresolved cross-topic issues and risks
- State readiness for downstream planning and implementation

---

## Change Log for Reprioritization and Scope Changes

Record any changes to index, ordering, scope, or criteria.

| Date | Change Type | Description | Rationale | Approved By |
|---|---|---|---|---|
| [YYYY-MM-DD] | [Order/Scope/Topic/Criteria] | [What changed] | [Why] | [Name/Team] |

---

## Progress Tracking

| Topic ID | Plan File | Report File | Status | Last Updated | Notes |
|---|---|---|---|---|---|
| [ID-001] | [path] | [path] | [Not Started/In Progress/Complete] | [YYYY-MM-DD] | [note] |
| [ID-002] | [path] | [path] | [Not Started/In Progress/Complete] | [YYYY-MM-DD] | [note] |

---

## Risks and Constraints

- [Constraint or risk]
- [Constraint or risk]
- [Constraint or risk]

---

## References

- Research Planning Approach (this repository):
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- Initial Planning Guidance (this repository):
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/initial-planning-guidance.md
- Topic Research Plan Template (this repository):
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research exemplar corpus (OS4CSAPI):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans

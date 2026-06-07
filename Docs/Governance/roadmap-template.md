# Roadmap

**Version:** [MAJOR.MINOR]
**Date:** [Month Day, Year]
**Effort:** [Project / Phase / Workstream Name]
**Status:** [Draft | In Review | Approved]
**Depends On:** [Link to approved Contribution and Goal Definition]
**Implements:** [Link to approved Implementation Guide]

---

## Purpose

Use this template to convert an approved implementation design into an executable, dependency-aware delivery plan.

This roadmap defines **when and in what order** work is executed. It should not redefine mission scope (Contribution and Goal Definition) or technical architecture/design (Implementation Guide).

Before drafting content, review the references listed at the end of this template and align your roadmap to their best-practice characteristics for:

- Nature of content (sequencing and control, not architecture authorship)
- Type of content (phases, tasks, dependencies, gates, milestones, evidence)
- Level of content (execution-level planning with explicit control points)
- Level of detail (specific, testable, and scheduler-ready)

---

## Executive Summary

Provide a concise execution summary of the roadmap.

### Summary Statement

[Describe the delivery plan at a high level in one paragraph.]

### Roadmap Scope

- **What this roadmap covers:** [Execution scope]
- **What this roadmap does not cover:** [Explicit exclusions]
- **Parent documents:** [Implementation guide and contribution links]

### Roadmap Overview

- Phase count: [N]
- Estimated effort range: [hours/weeks]
- Execution model: [predictive/waterfall-style]
- Primary control mechanism: [quality gates + dependency sequencing]

---

## Planning Assumptions and Constraints

### Assumptions

- [Assumption]
- [Assumption]
- [Assumption]

### Constraints

- [Resource/organizational constraint]
- [Technical/environment constraint]
- [Schedule/compliance constraint]

---

## Dependency Model

Define the non-negotiable execution dependencies.

### Phase-Level Dependencies

- [Phase A] -> [Phase B]
- [Phase B] -> [Phase C]
- [Phase C] -> [Phase D]

### Cross-Cutting Dependencies

- [Dependency and rationale]
- [Dependency and rationale]

### Blocking Conditions

- [Condition that blocks progression]
- [Condition that blocks progression]

---

## Phase Plan

Use one section per phase. Each phase must include objective, tasks, effort, dependencies, and exit criteria.

### Phase [N]: [Phase Name]

**Estimated Effort:** [Range]
**Complexity:** [Low | Medium | High]
**Goal:** [Phase objective]
**Dependencies:** [Prior phases/tasks]

#### Tasks

1. **[Task Name]** ([Estimate], [Complexity])
   - Scope: [What is implemented]
   - Deliverables: [Outputs]
   - Verification: [How completion is proven]
   - Dependencies: [Task-level prerequisites]

2. **[Task Name]** ([Estimate], [Complexity])
   - Scope: [What is implemented]
   - Deliverables: [Outputs]
   - Verification: [How completion is proven]
   - Dependencies: [Task-level prerequisites]

3. **[Task Name]** ([Estimate], [Complexity])
   - Scope: [What is implemented]
   - Deliverables: [Outputs]
   - Verification: [How completion is proven]
   - Dependencies: [Task-level prerequisites]

#### Phase Exit Criteria

- [Objective completion condition]
- [Verification/evidence condition]
- [Quality gate condition]

---

### Phase [N+1]: [Phase Name]

**Estimated Effort:** [Range]
**Complexity:** [Low | Medium | High]
**Goal:** [Phase objective]
**Dependencies:** [Prior phases/tasks]

#### Tasks

1. **[Task Name]** ([Estimate], [Complexity])
   - Scope: [What is implemented]
   - Deliverables: [Outputs]
   - Verification: [How completion is proven]
   - Dependencies: [Task-level prerequisites]

2. **[Task Name]** ([Estimate], [Complexity])
   - Scope: [What is implemented]
   - Deliverables: [Outputs]
   - Verification: [How completion is proven]
   - Dependencies: [Task-level prerequisites]

#### Phase Exit Criteria

- [Objective completion condition]
- [Verification/evidence condition]
- [Quality gate condition]

---

## Execution Units (Optional Granularity)

If tasks are too large for confident single-pass execution, split into execution units.

| # | Unit | Estimated Time | Complexity | Parent Task | Notes |
|---|------|----------------|-----------|-------------|-------|
| 1 | [Unit] | [Range] | [Low/Med/High] | [Task] | [Notes] |
| 2 | [Unit] | [Range] | [Low/Med/High] | [Task] | [Notes] |

---

## Milestones and Deliverables

### Milestones

- **M1:** [Milestone name] - [Condition]
- **M2:** [Milestone name] - [Condition]
- **M3:** [Milestone name] - [Condition]

### Deliverable Summary

- Implementation deliverables: [List]
- Test/verification deliverables: [List]
- Documentation deliverables: [List]
- Evidence package deliverables: [List]

---

## Quality Gates

Define mandatory gates for progression in predictive execution.

### Gate A: [Name]

- Entry criteria: [Requirements]
- Exit criteria: [Requirements]
- Evidence required: [Artifacts]

### Gate B: [Name]

- Entry criteria: [Requirements]
- Exit criteria: [Requirements]
- Evidence required: [Artifacts]

### Gate C: [Name]

- Entry criteria: [Requirements]
- Exit criteria: [Requirements]
- Evidence required: [Artifacts]

### Final Gate: [Name]

- Entry criteria: [Requirements]
- Exit criteria: [Requirements]
- Evidence required: [Artifacts]

---

## Verification and Reporting Cadence

### Verification Cadence

- [e.g., after each task, after each execution unit, at phase end]

### Progress Reporting

- [e.g., weekly status with dependency risk and gate readiness]
- [e.g., milestone evidence review checkpoints]

### Variance Handling

- [How estimate/scope variance is recorded and approved]
- [How dependency slips are escalated]

---

## Risk and Contingency Plan

### Top Execution Risks

- [Risk]
- [Risk]
- [Risk]

### Contingencies

- [Mitigation/contingency]
- [Mitigation/contingency]

### Replan Triggers

- [Trigger condition requiring roadmap revision]
- [Trigger condition requiring roadmap revision]

---

## Change Control

### Versioning Rules

- [Major/minor update policy]

### Approval Rules

- [Who approves roadmap updates]
- [What requires approval]

### Traceability Rules

- Maintain traceability from roadmap tasks to implementation-guide sections and verification artifacts.

---

## References

Use these as source references and structural examples.

- CSAPI Implementation Roadmap (OS4CSAPI, phase-9):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/planning/ROADMAP.md
- Phase 5 Parser Completion Roadmap (OS4CSAPI, phase-9):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/planning/phase-5/P5-ROADMAP.md
- Planning Folder (OS4CSAPI, phase-9):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/planning
- Initial Planning Guidance (this repository):
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/initial-planning-guidance.md
- Contribution and Goal Definition Template (this repository):
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/contribution-goal-and-definition-template.md
- Implementation Guide Template (this repository):
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/implementation-guide-template.md

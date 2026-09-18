# Implementation Guide

**Version:** [MAJOR.MINOR]
**Date:** [Month Day, Year]
**Effort:** [Project / Phase / Workstream Name]
**Status:** [Draft | In Review | Approved]
**Depends On:** [Link to approved Contribution and Goal Definition]

---

## Purpose

Use this template to translate an approved contribution definition into a technical execution design.

This guide should define **how** implementation will be performed, verified, and integrated. It should not redefine mission scope (Contribution and Goal Definition) or sequencing details (Roadmap).

Before drafting content, review the references listed at the end of this template and align your document to their best-practice characteristics for:

- Nature of content (design and implementation direction, not mission or scheduling)
- Type of content (architecture, interfaces, constraints, verification strategy, risks)
- Level of content (implementation-level engineering decisions with traceable rationale)
- Level of detail (specific enough for execution, testability, and review)

---

## Drafting Notes: Research and Requirements

Apply these instructions as part of drafting the guide within the established planning workflow.

1. **Use the approved Goal and Definition as the scope baseline.** Use completed research to inform the technical design. Research recommendations do not automatically add capabilities, mechanisms, or required documents to the project. Explain and justify the choices adopted in the guide.

2. **Review research as it is used.** When drafting a capability, examine the relevant reports, verify consequential standards claims against the original specifications, and check the evidence supporting important design choices. Challenge recommendations that add complexity and resolve contradictions that affect the proposed implementation. A complete re-audit of the research collection is not a prerequisite to drafting. Concentrate review on findings that become implementation decisions.

3. **Keep requirements in the Implementation Guide.** A requirement describes what the implementation must do; the design explains how it will do it. Record both, clearly distinguished, in the existing capability, contract, and verification sections. A separate requirements document is not required. Use standards identifiers, versions, and source links to connect requirements to their authority without reproducing the specifications or maintaining duplicate descriptions.

4. **Connect behavior, source, implementation, and verification.** For each capability, explain the required behavior, identify the applicable standards requirements or approved project objective, describe how it will be implemented, and state how it will be tested. Make these connections explicit enough for a developer to follow and for a reviewer to assess.

5. **Distinguish requirements from design choices and proposals.** Label standards obligations, project requirements, selected implementation approaches, and unresolved recommendations accurately. For example, a database choice is an engineering decision, not a requirement imposed by an API standard. If a source conflict or evidence gap remains unresolved, record the uncertainty and its effect on the relevant design rather than presenting an assumption as established fact.

6. **Preserve the established document roles.** Governance defines the working rules; the Goal and Definition establishes the intended outcome and scope; this guide explains the implementation; the Roadmap organizes delivery. Use the practical reference examples below to keep the guide understandable and useful. Supporting tables may live in the guide or its appendices; additional documents or review processes are not prerequisites merely because a research report proposed them.

---

## Drafting Plan: Three Iterations

Draft and refine one Implementation Guide over the following three iterations. Produce a complete first draft in the first iteration, then improve that same document; do not split the iterations into separate planning documents.

1. **Write the complete first draft.** Use the approved Goal and Definition, this template, the OS4CSAPI examples, and the relevant completed research to cover the full implementation scope. Include requirements, proposed architecture, capability behavior, interfaces, persistence, security, and verification. Identify unresolved decisions and evidence gaps explicitly. The result should be a coherent whole that the project lead can read and respond to, not merely an outline or a collection of finished sections surrounded by placeholders.

2. **Resolve gaps and make the instructions implementable.** Incorporate feedback, resolve open design questions, and strengthen sections where a developer would still have to guess. Check that required behavior has an authoritative source or approved project objective, the design explains how it will be implemented, and verification explains how correctness will be established. Keep requirements, implementation choices, and unresolved proposals distinguishable.

3. **Check the guide as a whole and finish it.** Walk through representative operations from input to result and check that the relevant sections agree. For Glaux Server, include registering a system, ingesting and retrieving observations, streaming updates, and issuing a command. Verify coverage against the approved Goal and Definition, remove duplication and unnecessary complexity, and finalize the guide for Roadmap development.

Apply the research checks in the preceding drafting notes throughout all three iterations. Examine relevant findings and verify consequential standards claims as they are used; do not defer these checks to a separate research audit.

At the end of each iteration, update and push the same guide and provide a short explanation of what changed and any specific questions that remain. Pause until the project lead says `proceed` before starting the next iteration. These are drafting passes, not additional approval gates or a requirement for separate review documents or pull requests.

Three iterations are the starting plan. Justify any additional iteration by a concrete unresolved issue rather than extending the process by default.

---

## Executive Summary

Provide a concise technical summary of what is being implemented in this effort.

### Summary Statement

[Describe the implementation objective in one paragraph.]

### Scope Clarification

- **What this guide covers:** [List specific implementation domains in scope]
- **What this guide does not cover:** [List explicit exclusions]
- **Relationship to parent artifacts:** [Main guide, phase guide, or superseding documents]

### Estimated Effort and Volume (Optional)

- Estimated implementation volume: [e.g., files/lines/components]
- Estimated testing volume: [e.g., tests/fixtures/suites]
- Estimated execution effort: [hours/weeks]

---

## Table of Contents

[List section links after drafting.]

---

## Purpose and Scope Baseline

### Scope Baseline

[Restate the approved implementation scope from the contribution document, without expansion.]

### Normative / Standards Commitments

[Identify standards, profiles, and conformance targets that govern implementation.]

### Acceptance Boundary

[Define what constitutes implementation-complete for this guide's scope.]

---

## Architecture Context

### System Context

[Describe where this effort sits in the larger architecture and ecosystem.]

### Component Boundaries

- [Component/Module]
- [Component/Module]
- [Component/Module]

### Interaction Model

[Add a diagram or structured flow showing major interactions.]

### Build vs Extend Breakdown

- **Build New:** [Items]
- **Extend Existing:** [Items]
- **No-Change Dependencies:** [Items]

---

## Design Principles and Constraints

Document implementation principles that guide all decisions.

- [Principle]
- [Principle]
- [Principle]

### Constraints

- [Technical constraint]
- [Operational constraint]
- [Compatibility constraint]

### Decision Discipline

[State how design decisions are recorded, reviewed, and versioned.]

---

## Implementation Specifications

Use subsection structure to define implementation details by capability or component.

### [Capability / Component A]

- Objective: [What it must do]
- Inputs/Outputs: [Data/interface contract]
- Behavior rules: [Expected behavior and edge handling]
- Validation rules: [Input/output validation requirements]
- Error model: [Failure handling and messaging]
- Implementation notes: [Key design constraints]

### [Capability / Component B]

- Objective: [What it must do]
- Inputs/Outputs: [Data/interface contract]
- Behavior rules: [Expected behavior and edge handling]
- Validation rules: [Input/output validation requirements]
- Error model: [Failure handling and messaging]
- Implementation notes: [Key design constraints]

### [Capability / Component C]

- Objective: [What it must do]
- Inputs/Outputs: [Data/interface contract]
- Behavior rules: [Expected behavior and edge handling]
- Validation rules: [Input/output validation requirements]
- Error model: [Failure handling and messaging]
- Implementation notes: [Key design constraints]

---

## Integration Points

Document required integrations and contract boundaries.

- Internal integration points: [Modules/services/interfaces]
- External integration points: [Systems/APIs/profiles]
- Compatibility requirements: [Versioning/backward compatibility]
- Configuration and environment dependencies: [Runtime/build/deploy prerequisites]

---

## Data and API Contracts

### Data Model Baseline

[Define canonical entities, relationships, and serialization rules.]

### API Behavior Baseline

[Define endpoint/resource behavior, parameter semantics, and response model expectations.]

### Error and Status Contract

[Define standardized errors, status codes/states, and failure semantics.]

---

## Conformance and Verification Strategy

### Conformance Model

[Define requirement-to-implementation traceability model and conformance evidence expectations.]

### Verification Methods

- Unit verification: [Approach]
- Integration verification: [Approach]
- Interoperability verification: [Approach]
- Conformance harness verification: [Approach]

### Evidence Artifacts

[List required evidence outputs and where they are stored.]

---

## Testing Strategy

### Test Scope

[Define what is tested and how coverage is measured.]

### Test Data and Fixtures

[Define fixture strategy, source quality, and organization model.]

### Regression Strategy

[Define how regressions are detected and controlled over time.]

---

## Risk Register

### Technical Risks

- [Risk]
- [Risk]

### Delivery Risks

- [Risk]
- [Risk]

### Mitigations

- [Mitigation]
- [Mitigation]

---

## Quality Gates and Exit Criteria

Define objective gates for implementation control.

- Gate 1: [Entry/exit criteria]
- Gate 2: [Entry/exit criteria]
- Gate 3: [Entry/exit criteria]
- Final Exit: [Criteria for guide scope completion]

---

## Change Control

### Versioning

[Define version increment rules and what constitutes a major/minor update.]

### Review Protocol

[Define required reviewers, approval conditions, and update process.]

### Traceability Maintenance

[Define how requirement/design/test links are kept current as the guide evolves.]

---

## References

Use these as source references and structural examples.

- CSAPI Implementation Guide (OS4CSAPI, phase-9):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/planning/csapi-implementation-guide.md
- Phase 5 Parser Completion Implementation Guide (OS4CSAPI, phase-9):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/planning/phase-5/P5-parser-completion-implementation-guide.md
- Planning Folder (OS4CSAPI, phase-9):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/planning
- Initial Planning Guidance (this repository):
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/initial-planning-guidance.md
- Contribution and Goal Definition Template (this repository):
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/contribution-goal-and-definition-template.md

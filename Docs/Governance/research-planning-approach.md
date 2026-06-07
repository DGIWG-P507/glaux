# Research Planning Approach

**Version:** 1.0
**Date:** June 7, 2026
**Status:** Draft

---

## Purpose

This document defines the governance approach for planning and executing research across the Glaux ecosystem. It standardizes how research topics are identified, planned, reported, reviewed, and promoted into contribution, implementation, and roadmap decisions.

The objective is to ensure research is:

- Traceable
- Evidence-driven
- Reviewable
- Consistent across all projects

---

## Scope

This approach applies to all research efforts under:

- Ecosystem-level research
- Component-level research (`glaux`, `glaux-server`, `glaux-webapp`, `glaux-simulator`, `glaux-publisher`, `glaux-mobile`)
- Cross-cutting domains (testing, standards, interoperability, performance, governance)

---

## Research Lifecycle

All research follows this mandatory lifecycle:

1. Topic Indexing
2. Research Planning
3. Research Execution and Reporting
4. Review and Synthesis
5. Promotion to Planning and Implementation Artifacts

No phase should be skipped.

---

## Phase 1: Topic Indexing

### Objective

Build an explicit index of research topics tied to project scope and decision needs.

### Required Outputs

- Numbered topic list with stable IDs
- Topic title and problem statement
- Scope mapping (which project/component is affected)
- Priority classification

### Rules

- IDs are stable and never reused
- Topics must map to a real planning or implementation decision
- Duplicate topics should be merged, not parallel-tracked

---

## Phase 2: Research Planning

### Objective

Define exactly how each indexed topic will be researched before work begins.

### Required Plan Elements

- Topic ID and title
- Research question(s)
- Why the question matters
- Sources and evidence strategy
- Method and evaluation criteria
- Expected output format
- Completion criteria

### Rules

- No research report may be written without an approved plan
- Plans must remain in-scope to the indexed topic
- Any scope expansion requires a plan update and version bump

---

## Phase 3: Research Execution and Reporting

### Objective

Execute the plan and produce a report that is evidence-first and decision-useful.

### Required Report Elements

- Topic ID linkage to the plan
- Sources reviewed
- Findings
- Evidence citations
- Conclusion
- Confidence assessment
- Open issues / unresolved questions

### Rules

- Conclusions must be evidence-backed
- Distinguish facts, interpretation, and recommendation
- Include explicit in-scope and out-of-scope boundaries

---

## Phase 4: Review and Synthesis

### Objective

Aggregate individual reports into review artifacts that support planning decisions.

### Required Outputs

- Review notes or synthesis document
- Consolidated patterns and conflicts
- Decision-ready recommendations
- Risk implications and follow-up topics

### Rules

- Synthesis should not drop contradictory findings
- Confidence and uncertainty must be preserved
- Promotion-ready findings must be explicitly identified

---

## Phase 5: Promotion to Planning Artifacts

### Objective

Promote validated findings into contribution, implementation, and roadmap documents.

### Promotion Targets

- Contribution and Goal Definition
- Implementation Guide
- Roadmap
- Technical standards or governance constraints

### Rules

- Only reviewed findings are promotable
- Promotions must reference source topic IDs and reports
- Planning changes must include traceability back to research evidence

---

## Governance Controls

### Mandatory Controls

- Plan-before-report enforcement
- Stable topic IDs
- Versioned updates (no silent rewrites)
- Review checkpoint before promotion
- Traceability from topic -> plan -> report -> planning decision

### Quality Gates

- **Gate A:** Topic indexed and accepted
- **Gate B:** Plan approved
- **Gate C:** Report reviewed
- **Gate D:** Findings promoted with traceability

---

## Folder and File Model

Recommended structure for each research domain:

- `research-plans/` - approved plans by topic ID
- `findings/` - completed reports by topic ID
- `review/` - synthesis and validation artifacts
- `results/` - finalized promotion-ready outputs
- `archive/` - superseded or historical versions

This preserves history while keeping active work navigable.

---

## Writing and Evidence Standards

### Evidence Standards

- Cite sources directly
- Use exact URLs where possible
- Record source date/version when relevant
- Separate observed evidence from interpretation

### Content Standards

- Keep research outputs decision-focused
- Use explicit assumptions and constraints
- State confidence level for each major conclusion
- Document unresolved questions clearly

---

## Change Management

### Versioning

- Major version: structural changes to approach
- Minor version: clarifications, non-breaking control changes

### Update Protocol

- Changes require documented rationale
- Prior version remains discoverable
- Downstream templates should be reviewed after approach changes

---

## Initial Next Steps

1. Create a research plan template aligned to this approach
2. Create a research report template aligned to this approach
3. Pilot the lifecycle on a small topic set before full-scale rollout

---

## References

- Testing research exemplar (OS4CSAPI, phase-9):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing
- Testing strategy research anchor document:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/testing-strategy-research.md
- Initial Planning Guidance (this repository):
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/initial-planning-guidance.md

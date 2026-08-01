# Research Planning Approach

**Version:** 1.4<br>
**Date:** August 1, 2026<br>
**Status:** Draft

---

## Purpose

This document defines the governance approach for conducting research in this repository.

The goal is to produce a series of rigorous research plans that generate high-value research reports.

---

## Core Process

Research work follows this sequence:

1. Create one overall research plan.
2. Include a planning index of topics in that overall research plan.
3. For each topic in the index, create one topic-specific research plan.
4. Execute topic research one topic at a time.
5. Produce one research report per completed topic.
6. Produce one final research report that responds to the overall research plan under the final-report model declared by that plan:
   - **Indexed synthesis topic:** execute the final indexed topic only after every preceding indexed topic report is complete and accepted, or a formal exception is approved and accounted for. The final report is also that synthesis topic's report.
   - **Unindexed closeout report:** produce the final report only after every indexed topic report is complete and accepted, or a formal exception is approved and accounted for.

This is the required process.

---

## Required Artifacts

### 1) Overall Research Plan

The overall research plan must contain:

- Scope of the overall research effort
- Planning index of research topics
- Topic IDs and titles
- Topic priority/order
- Completion criteria for the full research effort

### 2) Topic Research Plans

Each topic plan must contain:

- Topic ID and title (must match the overall index)
- Research question(s)
- Why the topic matters
- Sources to review
- Method to use
- Topic completion criteria

### 3) Topic Research Reports

Each topic report must contain:

- Topic ID and title
- Summary of research performed
- Findings
- Supporting evidence/references
- Evidence-authority and source-limitation notes
- Clear separation of sourced findings, analyst inference, and project recommendations
- Conclusion for that topic
- Any unresolved questions for that topic

### 4) Final Research Report

The final report follows the final-report model declared by the controlling overall plan. For an indexed synthesis topic, every preceding indexed topic report must be complete and accepted first. For an unindexed closeout report, every indexed topic report must be complete and accepted first. Either model permits only a formally approved and explicitly accounted-for exception.

It must contain:

- Response to the overall research plan
- Summary of all topic-level conclusions
- Overall conclusions
- Remaining open issues (if any)

### 5) Standards-Maintenance Evidence Registers

A controlling overall plan may establish a shared evidence register for relevant official standards-repository history. A register is a cross-topic supporting artifact, not an additional indexed research topic, research report, or source of normative requirements.

Each register must:

- identify the official repository, release/tag baseline, mutable branch snapshot, screening scope, and retrieval date;
- record only issues, pull requests, commits, releases, or discussions that materially affect an indexed topic;
- link each entry to the affected standard clause, requirement, artifact, design question, or downstream topic where possible;
- distinguish published baseline evidence, pre-publication rationale, post-publication maintenance direction, documented discussion, and unresolved proposals;
- record linked resolution artifacts and whether a change is included in an approved publication or release; and
- define bounded refresh rules so later topics consult relevant entries without repeatedly mining unrelated repository history.

---

## Governance Rules

- No topic report without a topic research plan.
- No topic plan without a matching topic ID in the overall research-plan index.
- Work topics one at a time until complete.
- A project-produced topic report named as a prerequisite must be complete and accepted before a dependent topic begins. It is not "unavailable" merely because it has not been produced.
- `Accepted` means the controlling overall-plan owner has reviewed the report for alignment and downstream decision usability, and the acceptance authority and date are recorded.
- Every topic-completion handoff must explicitly state the next two actions when another topic remains: first, plan-owner acceptance of the completed report; second, authorization to execute exactly one next eligible topic. The handoff must give the plan owner a single combined response pattern that both records acceptance and supplies the next-topic execution instruction in one message, for example: `Approve IDR-SRV-NNN. Then execute exactly one Glaux Server research plan: the next one, using the standing single-topic execution instructions.` The handoff does not itself record acceptance or begin the next topic; those occur only when the plan owner sends that combined instruction.
- A prerequisite exception requires controlling-plan-owner approval, a recorded rationale and impact, and an explicit sequencing or completion-record update.
- An external source may be unavailable. The report must identify the source and access limitation, avoid inventing its contents, narrow affected conclusions, and state what remains unresolved.
- Normative or authoritative claims must come from the controlling source. Implementation repositories, examples, tests, discussions, and prior practice are informative evidence unless the controlling source gives them another status.
- Mutable technical evidence must be pinned to a release, tag, commit, version, or dated retrieval sufficient to reproduce the finding.
- When a controlling overall plan names a standards-maintenance evidence register, each affected topic must consult and refresh the entries relevant to its scope. It must not silently rely on stale issue or pull-request state.
- Official repository issues, pull requests, comments, discussions, commits, and mutable branches are explanatory or maintenance evidence unless an approved standard or incorporated artifact gives them greater authority. Open issues and unmerged pull requests identify unresolved proposals or known defects; they do not create implementation obligations.
- Upstream-history review must remain bounded. Do not mine every inherited or adjacent repository by default; include an item only when its answer can change a project obligation, interpretation, API behavior, model, conformance posture, validation rule, implementation decision, or test expectation.
- Do not skip directly to a final report.
- Final report is produced only after the applicable indexed-synthesis or unindexed-closeout gate is satisfied, with any formally approved exceptions explicitly accounted for in its completion matrix.
- Use explicit references in every topic report.

---

## Working Model

The working model is simple and strict:

- Overall research plan (with indexed topics) controls the effort.
- Topic plans control execution quality per topic.
- Topic reports capture per-topic outcomes.
- Final report closes the full research effort against the overall plan.

---

## References

- Audited research-testing exemplar snapshot (OS4CSAPI `phase-9` at commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing
- Testing strategy research anchor document at the same snapshot:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/testing-strategy-research.md
- Glaux Server OGC API - Connected Systems upstream-history register:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Evidence/ogc-connected-systems-upstream-history-register.md

Mutable `phase-9` links retained in older topic plans are convenience links only. This audited snapshot controls exemplar structure and style unless a later snapshot is deliberately reviewed and registered here.

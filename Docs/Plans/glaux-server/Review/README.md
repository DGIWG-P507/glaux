# Glaux Server pre-implementation review

**Status: in progress.** This is the shared record for finishing the existing review across Copilot and other AI providers. A provider change resumes the saved work; it does not restart the review.

[Planning documents](../README.md) · [Findings and current assessment](findings.md) · [Review instructions](instructions.md) · [Coverage and handoff state](review-state.json) · [Evidence manifest](evidence/source-manifest.json)

## Start here

1. Read the current assessment in **findings.md**.
2. Read **instructions.md**, especially the rule for recording disagreements without interrupting coverage.
3. Use **current_work** in **review-state.json** to resume the exact unfinished step or next batch, and **batch_queue** in the same file to see the whole numbered remainder. Do not reload every archived response.

The last substantive reviewer response is **Pass 3c, iteration 23**, which executed queue batch 5, the committed deep read of IDR-055, the security and command-control test strategy. **All four committed deep reads are now complete and that check is closed.**

The batch turned up a status fact that matters for the rest of the review. Unlike the other three deep reads, IDR-055 carries no acceptance record, and its closing block says it awaits project-lead review. Its status line reads "Final," which here means drafting is complete rather than accepted. That is not the stale-wording problem tracked under F-17, since pre-acceptance wording is correct for an unaccepted report, and the Guide correctly never cites it. The consequence is recorded against the verification-quality pass, which will reuse this evidence: it must treat the content as a proposal and must not record a gap merely because one of the report's mechanisms is absent from the Guide.

The handoff from the previous batch's report checked out on every element. The security-test substance is substantially adopted, including a paired-world non-interference method the Guide already holds in its own words. No new finding resulted; the **F-03** response-cache instance gained a fifth corroborating source, the first to express the rule as a testable leakage surface naming ETags.

Every remaining finish condition is mapped to a finite numbered queue held in `batch_queue` of [review-state.json](review-state.json): **27 batches, 5 done, 22 remaining.** It covers the committed deep reads, a key-section screen of the reports whose read depth was never recorded, the end-to-end scenario pass, the verification-quality pass, the issue backlog and the final assessment, and it separates genuinely unreviewed work from coverage that was simply never recorded and from bookkeeping already corrected.

Iteration 20 corrected two accounting problems in that queue. Issue fidelity is now established by comparing all 286 issues against their Roadmap leaves rather than by sampling 18 of them, since a sample cannot establish complete coverage; the environment was checked first to confirm a complete comparison is achievable. The six partial reports left out of the first version are now assigned to the cluster batches that already cover their topics, reusing their recorded evidence and screening only unread sections, with no batch added for them. Three issue batches were combined because the complete comparison lets matching leaf content be reused instead of reviewed twice, which is what moved the count from 30 to 27. The count is a workload estimate with six recorded uncertainties, not a quota and not a completion guarantee.

The one evidenced Guide gap from the research remainder slices remains the response-cache instance under **F-03**, now resting on two accepted reports. Everything else in those four slices was either adopted or a recorded scope choice.

The record has **22 finding IDs, including two withdrawals**, with subsequent qualifications preserved. No standards check is outstanding, and **18 of 286** issue bodies are documented as read.

The next selected batch is **batch 6 of 27**: the first key-section screen, covering the standards and obligation baselines. The machine state is authoritative for that cursor as review continues.

## What lives here

| File or directory | Purpose |
|---|---|
| [findings.md](findings.md) | Authoritative finding dispositions, current assessment, evidence and implementation implications |
| [instructions.md](instructions.md) | Bounded iterations, independent review, disagreement handling, publication and provider handoff |
| [review-state.json](review-state.json) | Reading versus review coverage, unresolved checks, active cursor, batch history and the numbered remaining-batch queue; no duplicate finding text |
| [evidence/](evidence/) | Archived responses and historical instructions, clearly separate from active instructions |
| [evidence/source-manifest.json](evidence/source-manifest.json) | Provenance, original/published hashes and privacy transformations |

This is review working material, not a replacement Goal, Implementation Guide, Roadmap or research program. Changes suggested by the review are not approved merely by appearing here.

## Completion criteria

The purpose is to finish the review, not maintain an endless reading queue. Reuse existing work; a section already reviewed does not require another read because a pass number or AI provider changes.

| Original area | Current position | Finish condition |
|---|---|---|
| 1. Baseline/planning documents | Completed at the recorded baseline | Reuse it; check relevant subsequent changes only |
| 2. Standards | Complete for the carried checks: six carried groups closed (SWE quality, array flags, Features Part 3/CQL2 identifiers, SensorML class identifiers, IDR-011 Section 14.3 abstract-test rows, Part 2 Annex A.1 inheritance) and the F-13 extension question resolved | Met; later passes may raise targeted standards questions, each recorded in `remaining_checks` with evidence |
| 3. Research and peer evidence | Partially complete; all deep-read remainder slices closed and all four committed deep reads complete (18 reports now fully read); remaining work queued as batches 6-14 | Account for key findings/recommendations/open questions across 71 reports through the key-section sweep, and finish the pinned CS-GO/OSH checks; not blanket full reads of every plan/report |
| 4. End-to-end scenarios | Selected analysis exists | Account for the approved capability workflows and consequential failures/security/provenance boundaries |
| 5. Verification quality | Selected checks exist | Assess independent expected answers, meaningful failure detection and appropriate real-system checks across the planned capabilities |
| 6. Implementation issues | 18 of 286 bodies documented as read; targeted reviews only | Account for all issue-specific scope/acceptance checks and the complete dependency/coverage/sizing analysis; deduplicate identical boilerplate |
| Final assessment | Not complete | Publish supported findings, withdrawals, bounded uncertainties, coverage accounting and implementation advice |

These are coverage criteria, not a prediction of cost or iterations. An explicitly agreed scope exception must remain visible. A subscription limit produces a checkpoint, not a false completion declaration.

A finding can finish as supported, withdrawn, optional or explicitly unresolved with a documented consequence. The review does **not** need every recommendation implemented or every uncertainty eliminated to finish.

## Using the outcome

The final assessment must distinguish:

- Actions needed before initial implementation work.
- Decisions or fixes needed before a named later issue or milestone.
- Optional improvements.
- Remaining uncertainty and coverage limits.

Keep those recommendations tied to existing Guide/Roadmap/issue owners. Do not turn every finding into a new blanket prerequisite, a new project, or another research cycle.

## Cross-provider continuation

Give the next reviewer this folder's URL and say:

> Read README.md, instructions.md and the current findings. Resume current_work from review-state.json for one authorized iteration. Record disagreements and continue substantive coverage. Update and publish this review checkpoint before handing back.

The provider/model and actual evidence should be recorded when known. Unknown model names stay unknown. Repository commits are the durable handoff; private model memory and old ZIP exports are not the authority.

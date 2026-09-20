# Glaux Server pre-implementation review

**Status: in progress.** This is the shared record for finishing the existing review across Copilot and other AI providers. A provider change resumes the saved work; it does not restart the review.

[Planning documents](../README.md) · [Findings and current assessment](findings.md) · [Review instructions](instructions.md) · [Coverage and handoff state](review-state.json) · [Evidence manifest](evidence/source-manifest.json)

## Start here

1. Read the current assessment in **findings.md**.
2. Read **instructions.md**, especially the rule for recording disagreements without interrupting coverage.
3. Use **current_work** in **review-state.json** to resume the exact unfinished step or next batch, and **batch_queue** in the same file to see the whole numbered remainder. Do not reload every archived response.

The last substantive reviewer response is **Pass 3c, iteration 30**, which executed queue batch 12 over the platform, streaming, architecture, deployment and configuration reports.

**F-17** grew again. Three of this batch's six reports carry stale acceptance wording, in phrasings the previous search did not match. The response before this one had published that its count of eleven was a lower bound, and it was. Rather than keep finding instances one batch at a time, the search was re-run across all 71 reports with every wording now observed. Sixteen of the sixty-seven accepted reports carry it, and the list is recorded with line numbers. This batch produced no new variant, so the search has converged.

The remedy is now fully specified, and it already exists in the corpus. One report closes with a dated acceptance record naming what the acceptance authorized and what it did not. That is exactly what each stale closing was trying to say before acceptance, written for after it. Fixing the finding means replacing one closing paragraph in sixteen named reports. No new convention and no judgement call. The disposition is unchanged: optional editorial cleanup, no technical defect, and the dated acceptance headers remain controlling throughout.

The batch also contains the clearest adoption case in the review. Earlier batches showed the Guide following a report's direction. Here it carries the artifacts. The streaming report proposes an experimental profile name, pins an upstream draft commit, and lists five deviations from that draft. The Guide carries the profile name, the same commit hash, and all five deviations, including a deliberate lowercase spelling that corrects the draft and which the Guide also records in its source-contradiction table. It is worth stating plainly as a counterexample to any reading that the Guide treats accepted research loosely.

Adoption elsewhere is close, with one Guide line carrying three configuration recommendations and two more carrying four observability recommendations near-verbatim. Three non-adoptions are stated by the Guide for itself and none is a defect, including a case where two accepted reports disagree with each other and the Guide declines to settle it. One follow-up question gained a second accepted source, and the last partial research report moved to screened, so none remains. No new finding number and no new follow-up question.

The previous response, **iteration 29**, executed queue batch 11 over the storage, query and write-boundary reports.

The batch found three more instances of **F-17**, the stale-acceptance finding, and they are the clearest form yet. Each is a report whose header records a dated acceptance and whose closing section tells the reader it is still in review. The contradiction sits inside one document, so a reader who checks the end, which is where people look for status, is told the opposite of what the front said.

That prompted a check of all 71 reports for the stale wording itself, and the result corrects this review. The bound published in the two previous responses, three instances among fifteen reports checked, was computed from acceptance **headers**. A header check finds reports that get it right in the header. It cannot find a report that gets it right in the header and contradicts itself later, which is exactly this shape. The confirmed population is at least eleven accepted reports, not three, and the list is now written down. The disposition does not change: it remains optional editorial cleanup, no technical defect follows, and the dated acceptance records remain the controlling facts. What changes is the remedy, from spot fixes to one pass over a named list.

A second published bound was corrected the same way. Iteration 28 had noted that all five of its reports were cited in the Guide and read that as evidence against a general citation gap. Only one of this batch's five is cited. A complete comparison shows 25 of the 67 accepted reports have no Guide reference marker of their own, so the earlier reading is withdrawn and the question is now a bounded list rather than an anecdote. That count is an upper bound rather than a defect count, because the Guide reaches some of those reports without a marker.

The **F-03** cache item gained four more accepted sources, now fourteen across seven unrelated subject areas. One of them states the rule in this review's own terms, as a numbered step: compute sort, counts, extents, pagination, links and cache entries from the authorized view. No escalation is proposed and the item's substance is unchanged.

Otherwise the batch is closely adopted, with one Guide line matching five recommendations of one report near-verbatim, and two non-adoptions the Guide states by name. No new finding number and no new follow-up question.

The previous response, **iteration 28**, reopened batch 9, finished it, and then executed queue batch 10.

Batch 9 had been recorded as complete while assigned sections of four of its six reports were unread. It was reset to partial before anything was read, so the record was accurate about the gap while the gap was open, and the missing sections were then finished. The comparison against the saved scope that followed found one further omission, a decision register in a fifth report, which was also read. That comparison is now a published stop condition for every remaining screen batch rather than a stated intention, because this was the second batch running to be marked done with subsections unread.

Finishing those sections corrected something. Iteration 27 had read one recommendation as half adopted, on the grounds that the Guide uses a URN where the research asks for an HTTPS namespace. The same report lists the final namespace URI as an open question routed to named owners, so the Guide chose inside an open question rather than departing from a settled rule. That characterization is withdrawn. The main relation-spelling result under **F-11** is unaffected and stands in full.

Batch 10 then produced the strongest evidence the response-cache item under **F-03** has had. It had rested on reports about security, policy, testing and content negotiation, which left open whether the requirement was an artifact of those subject areas. Three reports about data contracts and storage state it independently, a fourth states the mechanism, which is recording the policy version with the cached representation, and a fifth names the two topics expected to settle the HTTP-layer part. The item now rests on ten accepted reports across five unrelated subject areas. Its substance, severity and remedy are unchanged and no escalation is proposed.

Batch 10 is also the most closely adopted batch screened so far, with near-verbatim matches across persistence, migrations, schema resolution and unit handling, and all five of its reports are cited in the Guide. No new finding number and no new follow-up question.

The previous response, **iteration 27**, made two bounded corrections to batch 8 and then executed queue batch 9.

The first correction withdraws a claim. Iteration 26 reported that a peer project's test for the latest-value selector, which asserts that exactly one item comes back, corroborates the accepted research's criticism of that peer. It does not. The fixture seeds a single newest observation, so it never exercises tied timestamps at all, and nothing in it would distinguish a correct implementation from an incorrect one. What the test records is its author's expectation, not the server's behavior. The observation beside it survives untouched: the adjacent time-range test asserts only how many items came back and never which one, so it would pass if the wrong observation were returned. The earlier evidence report is preserved with its body unchanged and the correction appended to it, so a reader who has only that report still finds the correction.

The second correction finishes work that was recorded as done. The previous screen stopped at the first subsection of each report's risks-and-open-questions section and left the open questions unread. All 102 omitted lines are now read. They change nothing, because the questions are almost entirely about the peer projects themselves rather than about Glaux, but three items were worth keeping, including the first support from a peer study for **F-12**.

Batch 9 then produced the strongest result the relation-spelling finding has had. **F-11** has recorded since early in the review that the Guide states no rule for how link relations are spelled while emitting three different forms. The accepted relationship and linkage report states that rule directly, as a numbered recommendation: emit the exact published spellings, compare extension relation URIs case-insensitively, and confine bare values to a named compatibility adapter. The Guide neither carries that rule nor mentions the report, and it emits a bare relation name in one place that the recommendation would move into a compatibility adapter. That is the same shape as the cache result from iteration 25 and it is recorded under the existing finding, not as a new one.

The batch also produced a useful negative result. All six reports screened record their acceptance correctly, which bounds **F-17** as a residue in particular reports rather than a systemic practice. And it produced the clearest example yet of accepted research whose central mechanism is deliberately not adopted: the provenance report asks for a PROV evidence graph, and the project declines it three times on the record while adopting the obligations the report derives. No new finding number and no new follow-up question.

Every remaining finish condition is mapped to a finite numbered queue held in `batch_queue` of [review-state.json](review-state.json): **27 batches, 12 done, 15 remaining.** It covers the committed deep reads, a key-section screen of the reports whose read depth was never recorded, the end-to-end scenario pass, the verification-quality pass, the issue backlog and the final assessment, and it separates genuinely unreviewed work from coverage that was simply never recorded and from bookkeeping already corrected.

Iteration 20 corrected two accounting problems in that queue. Issue fidelity is now established by comparing all 286 issues against their Roadmap leaves rather than by sampling 18 of them, since a sample cannot establish complete coverage; the environment was checked first to confirm a complete comparison is achievable. The six partial reports left out of the first version are now assigned to the cluster batches that already cover their topics, reusing their recorded evidence and screening only unread sections, with no batch added for them. Three issue batches were combined because the complete comparison lets matching leaf content be reused instead of reviewed twice, which is what moved the count from 30 to 27. The count is a workload estimate with six recorded uncertainties, not a quota and not a completion guarantee.

The one evidenced Guide gap from the research remainder slices remains the response-cache instance under **F-03**, which now rests on six accepted reports and, since iteration 25, reads as a partial adoption of one numbered recommendation rather than an unspecified area. Everything else in those four slices was either adopted or a recorded scope choice.

The record has **22 finding IDs, including two withdrawals**, with subsequent qualifications preserved. No standards check is outstanding, and **18 of 286** issue bodies are documented as read.

The next selected batch is **batch 13 of 27**: the eighth key-section screen. The machine state is authoritative for that cursor as review continues.

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
| 3. Research and peer evidence | Partially complete; all four committed deep reads complete (18 reports fully read), key-section screens A to G done (43 more screened, no partial reads remaining), and the pinned CS-GO/OSH checks complete; remaining work queued as batches 13-14 | Account for key findings/recommendations/open questions across 71 reports through the key-section sweep; not blanket full reads of every plan/report |
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

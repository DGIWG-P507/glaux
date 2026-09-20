# Glaux Server pre-implementation review

**Status: complete.** The pre-implementation review finished on September 20, 2026. Start with [the final consolidated assessment](evidence/51-pass-3c-45-final-consolidated-assessment.md). This folder remains the shared record of how that conclusion was reached, across Copilot and other AI providers; it is no longer an open queue, and further review would be a new authorization with its own scope.

[Planning documents](../README.md) · [Findings and current assessment](findings.md) · [Review instructions](instructions.md) · [Coverage and handoff state](review-state.json) · [Evidence manifest](evidence/source-manifest.json)

## Start here

1. Read the current assessment in **findings.md**.
2. Read **instructions.md**, especially the rule for recording disagreements without interrupting coverage.
3. Use **current_work** in **review-state.json** to resume the exact unfinished step or next batch, and **batch_queue** in the same file to see the whole numbered remainder. Do not reload every archived response.

The review is finished. The last response, **Pass 3c, iteration 45**, published [the final consolidated assessment](evidence/51-pass-3c-45-final-consolidated-assessment.md), which is where to start.

**The plan is sound.** All 286 planned tasks were read and assessed as whole units and every one of them is adequate. The findings that stand are about the Guide, the research record and the repository baseline - not about the work itself.

**Only two things are recommended before implementation starts, and both are decisions only the project can make.** First, confirm what required-check enforcement actually exists on the server repository and decide what continuous-integration enforcement should be required; the review saw none at the baseline it inspected and cannot claim to know the current settings. Second, choose the project licence. The apparent obstacle there is already gone, because no reuse of another project's source or fixtures is planned, so what is left is simply the act of choosing.

**Everything else is scheduled, optional, or an open question.** Thirteen further decisions are each tied to a named later issue or a specific Guide line - one of them is a single missing word about sort direction. Seven items are optional improvements. Twelve questions remain genuinely open, and the assessment says where each of them lands rather than leaving them floating. None of this has to be done for the review to be finished, which is the review's own published rule.

**Two findings were reconsidered for escalation and neither was escalated.** The audit-boundary finding now has complete evidence - the task that restores everything and the task that implements retention both list every other kind of evidence and leave audit out - but the Guide expressly declines to require a tamper-proof ledger, so what the evidence shows is a question nobody has answered rather than a rule anyone has broken. Raising the severity would turn an honest silence into an accusation the evidence does not support. The related export-accountability finding keeps its severity for the simpler reason that nothing has been built yet. What both gained is a concrete place and time to be decided.

The previous response, **iteration 44**, assessed the 50 exchange, restore and release tasks and completed the per-task assessment.

**Every one of the 286 planned tasks has now been read and assessed, and every one is adequate.** No task's scope or acceptance criteria was found wanting anywhere in the backlog, and the whole per-task pass raised no new finding and no new question. What it produced instead is evidence for findings already recorded, which is the more useful outcome: it shows where each recorded gap would actually land in the work.

**The exchange phase is written on one idea: a claim from somewhere else is never an authority.** A field in an incoming file does not grant permission. Authenticating whoever sent a payload does not verify what the payload says about who produced it. A missing member is not an instruction to delete. A larger version number is not proof of ancestry. JSON that means the same thing is not the same bytes. And a run of records that has been settled up to a gap is complete only up to that gap. The release phase is written the same way: completion may be claimed only with evidence for that exact candidate, a candidate with gaps is explicitly partial, and local testing is never called certification.

**The audit finding is now settled as far as this review can settle it.** The task that restores the complete data model and the task that implements retention cleanup both list every other kind of evidence and leave audit out, while eight tasks in the same batch require audit to be written and committed with the thing it records. That asymmetry holds across all 286 tasks. The finding keeps its current status here, and the final assessment will decide whether evidence this complete deserves a stronger recommendation. Two neighbouring findings about exchange accountability were confirmed in the same way.

The previous response, **iteration 43**, assessed the 41 live-publication and filtering tasks.

**All 41 are adequate**, bringing the total to 236 with none found inadequate. Live publication is where a server is most tempted to claim it delivered something it cannot actually see, and these tasks refuse in plain terms: nothing claims a message is delivered exactly once, an acknowledgement from the broker is not proof that anyone received or acted on it, and a history that is no longer available must say so rather than return an empty answer that looks complete. A checkpoint may only move forward once everything before it has actually been written, and work that is not finished stays visibly unfinished rather than being counted as done.

**The rules about who may see what are fail-closed and aware of shape.** If a permission check cannot be completed, the answer is no. A deleted resource is still checked against the permissions kept for it, rather than treated as public because the live record is gone. A channel whose whole audience cannot be enforced is switched off rather than left running and filtered at the client. And withholding the contents of a message is explicitly not enough, because its subject, its parent, its kind or the simple fact that it exists can give away the protected change. The filtering phase says the same thing in its own way: a hidden fact must not come back as an empty value, a count or a detailed error.

**The relation-spelling finding gained one implementing task for each of the three spellings.** The Guide writes link relations three different ways and says which to use in none of them; each of the three now has a task that faithfully reproduces its own line, and a search of all 286 issues found no rule anywhere. The search was proven able to find the missing wording before its empty result was accepted.

The previous response, **iteration 42**, assessed the 42 tasking tasks.

**All 42 are adequate**, bringing the total to 195 with none found inadequate. This is the phase that matters most for safety, because it is the only one whose work can cause something physical to happen, and the tasks read that way. The rules are written into named tasks rather than left as general principles: losing contact with a device never turns into a reported failure, asking for a cancellation is not the same as a cancellation that took effect, deleting a record is not cancelling the work it describes, answering no to a feasibility question is a completed analysis rather than an attempt to do anything, permission to submit a command is not authority to report what happened to it, and no database transaction is allowed to stay open across a call to a real device.

**The audit finding is confirmed a third time, and now points at a smaller target.** Three restore tasks exist, one per phase that has one, and none of them checks that audit records survive the restore. The batch also shows where the gap is not: a task in this phase does verify audit information at the moment it is written. So the question is specifically whether audit survives a restore or a retention boundary, which is what the finding has always been named for; it is confirmed rather than merely repeated.

The previous response, **iteration 41**, withdrew a reintroduced claim and assessed the 28 codec tasks.

**A claim this review withdrew months of iterations ago came back, and is withdrawn again.** The previous response said a peer implementation mishandles tied timestamps, citing a fixture examined much earlier. That fixture seeds a single newest record, so it never exercises ties and cannot show how the peer behaves. The claim is removed from all five places it reached, and in the machine state the original wording is quoted inside the withdrawal rather than deleted, so the record shows what was claimed and why it was wrong. Worth naming plainly: a withdrawal recorded in an appendix did not stop the claim returning in a summary written from memory of the work rather than from the corrected record. The Glaux assessment is untouched and no peer research is reopened.

**All 28 codec tasks are adequate**, bringing the total to 153 with none found inadequate. Codec work is where a test that round-trips through its own encoder proves nothing, and five tasks say exactly that in their own words. Three framing tasks split the same bytes at every awkward boundary, including inside multi-character delimiters and multibyte text, and require the piecewise result to match the whole-body one.

**The cache finding gained its implementing task.** One task carries the Guide's negotiation rule faithfully and no further: it exercises the headers and validators the Guide names, and does not bind a validator to the authorized view or test a cache channel, because the Guide does not ask it to. That is correct implementation, and it locates exactly where a corrected rule would land.

The previous response, **iteration 40**, assessed the 39 Phase 3 tasks.

**All 39 are adequate**, bringing the total to 125 assessed with none found inadequate. Phase 3 is where observation semantics first meet storage and query, and these tasks reproduce the Guide's hardest rules almost word for word: a populated stream's schema cannot be changed by calling the change compatible or storing an internal revision; the deletion and conflict rules are carried over exactly; the retry key stays optional and distinct measurements are never merged because their values match; and an ordinary observation arriving cannot silently create a system event.

One task deserves naming. The latest-value selector is applied after authorized scope and every other filter, and **retains ties** rather than returning a single row. That is the Guide's rule in full, and the task seeds tied timestamps, a newer excluded record and late arrivals so the distinguishing cases are built in rather than assumed. No comparison with a peer implementation is drawn: an earlier response withdrew the claim that the peer fixture examined months into this review established how that implementation handles ties, because the fixture seeds a single newest record and cannot exercise them.

**The audit finding is confirmed a second time.** Both restore tasks written so far, one in Phase 1 and one in Phase 3, verify resources, revisions and artifacts without checking that audit records survive the restore. The design exists in the Guide; no task yet exercises it.

**The coverage counts are now computed rather than restated.** They are derived from the per-issue records at each update, so the assessed, targeted-only and unique-read figures reconcile by construction. Three earlier counting mistakes all came from restating totals in prose, and this removes the class of error rather than the instances.

The previous response, **iteration 39**, withdrew a counting claim about the response-interpretation finding and assessed the 31 Phase 2b tasks.

**A counting claim is withdrawn, and not replaced.** The two previous responses tallied tasks said to state the boundary the response-interpretation finding records, and compared the tallies to conclude a pattern search missed about half of them. Those tallies ran two different rules together: not deriving your expected answer from the server's output, which is what the quoted wording nearly always says, and whether a test may decode the actual response using the production code's own types, which is the open question. Since neither tally counted the second, no ratio between them meant anything. No replacement count is needed, because a corpus count was never going to answer it, which is why the finding rests on reading a single sentence in the conformance rules. The finding itself is unchanged and is not weakened.

The targeted-only issue count is corrected to 12, so the figures reconcile exactly: assessed plus targeted-only equals unique read.

**All 31 Phase 2b tasks are adequate**, bringing the total to 86 assessed with none found inadequate. The tasks are notable for refusing research the Guide superseded, for building genuinely discriminating spatial cases, and for modelling the Guide's documented paging choice rather than contradicting it.

**The abstract-test finding narrowed.** Its owning task forbids exactly the wrong route inference and forbids importing server traversal logic into the conformance runner, so the guard is in place. What is still missing is the documented adaptation itself, which remains fixture content owned by that task rather than a gap in the Guide.

The previous response, **iteration 38**, assessed the 31 Phase 2a tasks.

**The issue read count published last time was wrong.** It reported 42, which came from adding one batch's 24 to the 18 recorded before it without removing the five issues counted twice. The correct figure is **37**. It is now derived from distinct issue IDs rather than from adding batch totals, and that derivation earned its keep immediately: it caught a second overlap in this batch before publication rather than after.

Two measures were kept apart from this point on, because they answer different questions. At that iteration, sixty-seven issues had been read by a reviewer and fifty-five tasks had been assessed for whether their scope and acceptance criteria are adequate. Neither substitutes for the other. (Both reached 286 by iteration 44.)

**All 31 Phase 2a tasks are adequate**, bringing the assessed total to 55 with none found inadequate. Three habits recur and each answers a concern this review holds: the tasks refuse to invent rules the source does not contain, they construct discriminating rather than confirming test cases, and they tie to specific Guide lines rather than gesturing at sections.

**A counting claim about the response-interpretation finding is withdrawn.** Two previous responses tallied tasks said to state the boundary that finding records, and compared the tallies to conclude a pattern search misses about half of them. Those tallies ran two different rules together: not deriving your expected answer from the server's output, which is what the quoted wording nearly always says, and whether a test may decode the actual response using the production code's own types, which is the open question. Since neither tally counted the second, no ratio between them means anything, and no replacement count is needed. The finding itself is unchanged and rests where it always did, on a single sentence in the conformance rules that forbids the server's serializers as the sole oracle and permits ordinary format libraries, while a production type is neither.

A fourth defect in the review's own saved state was corrected before assessing: a batch title claimed a split at capability-group boundaries that does not exist. A contiguity check across all eight issue batches confirmed no issue is orphaned.

The previous response, **iteration 37**, assessed the 24 Phase 1 tasks.

**All 24 Phase 1 tasks are adequate.** Each was judged as one unit, with its scope, exclusions, prerequisites, acceptance criteria and verification approach read together rather than separately. The tasks are disciplined in ways worth naming: every exclusion points at the sibling that owns what it defers, several forbid a specific wrong shortcut rather than only stating the goal, and one records an honest limitation instead of overclaiming.

**The response-interpretation finding gained its first quantified support.** Six of the 24 tasks state, in their own words, the exact boundary the Guide leaves open: that expected values must not be taken from the server's own serialization. One puts it precisely, that a successful round trip through the same faulty conversion proves nothing. Across all 286 issues the boundary appears at least 24 times, and that is a lower bound, demonstrated twice inside one batch as the search was widened. So the rule is real and the project believes in it, but it is written where an author happened to think of it rather than stated once as a rule. That is exactly what the recorded one-clause fix would change.

One earlier claim was corrected. A previous response said the shared checklist carries the Guide's verification practices into every issue. That holds for the false-green rule and for independent expected results, and not for this boundary.

The reviewer-read issue count rises from 18 to 42. The other 262 have been fetched and compared, which is not the same as read.

The previous response, **iteration 36**, completed the whole-backlog dependency, coverage and sizing analysis.

One correction came first. The previous response claimed the remaining issue batches reduce to the content the issues add beyond their leaves. That is **withdrawn**. Matching Roadmap content is reused as one shared copy and still receives semantic review, because the leaf text *is* each task's scope and acceptance criteria, and establishing that it was copied faithfully says nothing about whether it is adequate. What the comparison genuinely removed is re-reading the same text 286 times, not judging it.

**The backlog analysis is clean.** The dependency graph across nine phases, 41 capability groups and 286 tasks has 1,676 edges, a single starting task matching the Roadmap's own handoff, and a longest chain of 136. There is no cycle, no dangling reference, no self-dependency and no dependency running backwards in time. Phase counts reconcile exactly. Phase prerequisites are consistent for all nine phases once compared against the transitive closure, which is the right test because the phase table lists proximate prerequisites rather than the full set.

Every mechanical check was proven able to detect an injected defect before its clean result was accepted, and the parser that reads the Roadmap's dependency prose was validated on six shapes, including a trailing explanatory clause that must not be mistaken for further dependencies.

Scope coverage is complete, and sizing is coherent, with the three outliers inspected and explained rather than reported as defects.

**Three apparent problems turned out to be artifacts of the analysis, not defects in the backlog**, and are recorded as such. The discipline cuts both ways: a sensitive check asking the wrong question produces confident nonsense, so every non-zero result was inspected before being reported, just as every zero was tested for sensitivity before being accepted.

The previous response, **iteration 35**, compared all 286 issues against their leaves and found no difference.

**The difference inventory is empty**, and the result rests on exact containment rather than a similarity score: every leaf's scope text and every leaf's acceptance text appears verbatim in its linked issue, on all 286. A zero from a mechanical check is worth nothing unless the check can detect a difference, so before accepting it every check was run against an injected difference and all of them fired, and a deliberately wrong pairing was correctly flagged. One reviewer error was caught during the run and is recorded rather than quietly fixed: an initial report of 286 title mismatches turned out to be a defect in the comparison's own text handling, not in the data.

The batch also found the third defect in the review's own saved state. The scope never named a repository, so the first fetch went to the planning repository that holds this review, which contains no issues at all. The implementation issues live in a separate repository, which every Roadmap leaf link states. Had the instruction not required verifying the target set, that run would have found nothing and could have been misreported as a backlog problem rather than a reviewer error.

The shared boilerplate was reviewed once and has exactly one variant of each shared section. It carries the Guide's verification practices into every unit of work, which is a positive result for the verification area closed in the previous response.

Two consequences. The queue's largest recorded sizing uncertainty resolves favourably, because matching content genuinely need not be reviewed twice. And the documented issue read count stays at **18 of 286**: the bodies were fetched and compared mechanically, which is not a reviewer reading them.

The previous response, **iteration 34**, completed the verification area. Five of the seven review areas are closed.

The saved references were verified first, after two consecutive batches turned up defects. All five were correct, and the difference is that these were written after the sections had been read rather than during the original queue drafting.

The verification strategy establishes all three things this pass assessed. Independent expected answers appear in three places. Meaningful failure detection is the most developed part of it: a test has to be shown to detect the intended mistake, a setup or compilation failure is explicitly not that proof, and test counts, coverage thresholds and mutation scores are refused by name as establishing conformance. Real-system checks are governed by a rule about which layer may support which claim, which closes the database-mock, sleep-as-ordering and paused-clock loopholes explicitly. Every planned capability has a layer that can observe its claims, and no gap was found between what the strategy promises and what its layers can deliver.

Two findings were materially refined, neither changing disposition. **F-12** moves to its real home, a single sentence in the conformance rules rather than anywhere in the testing strategy. That sentence forbids resting on the server's serializers as the sole oracle and permits ordinary format libraries, which narrows the open question to one the sentence does not settle: whether a runner may decode a response using the production code's own types while holding independently authored expectations. The finding is better supported and more precisely located than the register had recorded, and the fix is one clause. **F-09**'s general rule turns out to be adopted already, so what remains is fixture content owned by an existing issue rather than a gap in the Guide.

No research report was reopened, and no new finding number or follow-up question resulted.

The previous response, **iteration 33**, completed the scenarios area and gave four findings named remedy locations.

The saved references for this batch were wrong and were corrected before anything was read. They named one Guide section as holding the walkthroughs; that section holds risks and checks, and the scenarios and walkthrough live in two subsections of the testing strategy. This is the second reference defect found in the review's own saved state, and both date from when the batch queue was first drafted, before the named sections had been read.

Coverage is complete. Every capability area in the Goal is reachable from both a representative scenario and a whole-guide walkthrough path. Security and verification have no dedicated path and are cross-cutting by the Guide's own stated design, which is recorded as a choice rather than a gap, and is arguably stronger than isolating security in one scenario.

Every consequential assertion in the scenarios was checked against the design line that has to deliver it, and all are supported. No scenario promises a guarantee the design never establishes. That is worth stating because the opposite is a common defect in scenario sets.

The batch's main product is that **four existing findings gained a named remedy location**. Each now points at the specific scenario where its recorded remedy would go: the cache item belongs in the indirect-disclosure clauses that already close two scenarios, the audit item in the backup-restore scenario, the stream-state item in the live-delivery scenario, and the relation-spelling item in the link-navigation scenario. No disposition changed. That converts four general remedies into specific edits at named lines, which is what a consolidation owner needs.

No research report was reopened, and no new finding number or follow-up question resulted.

The previous response, **iteration 32**, executed queue batch 14 and closed the research key-section sweep.

**The research area is complete.** All 71 topic reports are accounted: 18 read in full, including all four committed deep reads, and 53 screened at their recommendations, risks and open questions, and decision registers. No partial read remains and no report is left with unrecorded read depth. Two limits stay on the record and are worth stating: a screen is not a full read, so executive summaries, bodies, appendices and validation sections were deliberately not read, and each report's entry lists exactly what was and was not covered, so a later pass can reopen one report without re-deriving coverage.

This batch also produced the first **F-17** false positive. One matched line says a report's recommendations await a decision about whether they become requirements, which is not the same as the report being unaccepted and stays true after acceptance. It is the distinction this review applies throughout. Because the search can produce a false positive, every stale-wording match was then read in place. All sixteen are genuine, so the count stays at twenty, but its basis is now verification rather than a pattern match, and two milder instances are separated from the eleven direct claims.

One adoption is worth stating plainly. The Part 4 study offers a low-effort option it marks recommended, and an experimental option it explicitly does not select, which carries three conditions if chosen. The project took the harder path and the Guide met all three, naming the excluded types, pinning the draft commit and requiring round-trip evidence. Two earlier conclusions were independently confirmed by reports read in this batch. **F-01** gained a fact that removes an apparent blocker without changing the finding: no peer source reuse is required, so the peer licence constraints do not bite.

No new finding number and no new follow-up question.

The previous response, **iteration 31**, executed queue batch 13 over the operations, traceability, fixtures, performance and interoperability reports.

**F-17 has a second sub-form, and no previous scan could have found it.** Four of this batch's five reports carry no acceptance fields in their headers at all, although the governance ledger records every one as accepted. The two scans built in the previous responses both searched for stale closing text that contradicts a correct header. A report with no header has nothing to contradict.

This is the form that misled this review. It concluded early on that one report was unaccepted, for exactly this reason, and was corrected. It then built two thorough scans for a different variant and never went back to count the one that had actually caused the wrong conclusion.

Counting both forms gives twenty affected reports of sixty-seven, and it changes which ones matter most. Two of them close with an unchecked box stating that acceptance has not been recorded. Those do not merely omit the status; they assert the opposite of the ledger, so a reader consulting either alone would reach the wrong conclusion. Two more combine a missing header with stale text. The fourteen found previously are contradictory but at least carry a correct header. Two are mild, recording the acceptance correctly but in an unusual place.

The disposition is unchanged: optional editorial cleanup, no technical defect, ledger correct and controlling throughout. The remedy recorded previously fixes both forms, because a dated closing acceptance record supplies in the body exactly what the missing-header reports lack. The practical change is that the cleanup can start with the two reports that state the opposite of the truth.

Otherwise the batch is closely adopted, with the recovery, fixture, performance and interoperability rules carried near-verbatim. Two large mechanisms are stated non-adoptions and neither is a defect, including a traceability graph the Guide declines by name on two counts while adopting the obligations it derives. One earlier record was corroborated more cleanly than the review had managed. No new finding number and no new follow-up question.

The previous response, **iteration 30**, executed queue batch 12 over the platform, streaming, architecture, deployment and configuration reports.

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

Every finish condition was mapped to a finite numbered queue held in `batch_queue` of [review-state.json](review-state.json): **27 batches, all 27 done, none remaining.** All seven review areas are closed. It covers the committed deep reads, a key-section screen of the reports whose read depth was never recorded, the end-to-end scenario pass, the verification-quality pass, the issue backlog and the final assessment, and it separates genuinely unreviewed work from coverage that was simply never recorded and from bookkeeping already corrected.

Iteration 20 corrected two accounting problems in that queue. Issue fidelity is now established by comparing all 286 issues against their Roadmap leaves rather than by sampling 18 of them, since a sample cannot establish complete coverage; the environment was checked first to confirm a complete comparison is achievable. The six partial reports left out of the first version are now assigned to the cluster batches that already cover their topics, reusing their recorded evidence and screening only unread sections, with no batch added for them. Three issue batches were combined because the complete comparison lets matching leaf content be reused instead of reviewed twice, which is what moved the count from 30 to 27. The count is a workload estimate with six recorded uncertainties, not a quota and not a completion guarantee.

The one evidenced Guide gap from the research remainder slices remains the response-cache instance under **F-03**, which by the end of the review rested on **14 accepted sources across seven subject areas** and reads as a partial adoption of one numbered recommendation rather than an unspecified area. Its implementing task is #151, and no issue in the backlog contains a cache directive, an `ETag` or a conditional-request header. Everything else in those four slices was either adopted or a recorded scope choice.

The record has **22 finding IDs, including two withdrawals**, with subsequent qualifications preserved. No standards check is outstanding. **All 286 of 286** issue bodies are documented as read by a reviewer and **all 286** tasks have been assessed for scope and acceptance adequacy, with none left at targeted-check depth. These figures are computed from the per-issue records, not maintained by hand. All 286 have been fetched and compared mechanically.

No batch remains. The machine state records the completion and carries `review_complete: true`.

## What lives here

| File or directory | Purpose |
|---|---|
| [findings.md](findings.md) | Authoritative finding dispositions, current assessment, evidence and implementation implications |
| [instructions.md](instructions.md) | Bounded iterations, independent review, disagreement handling, publication and provider handoff |
| [review-state.json](review-state.json) | Reading versus review coverage, closed checks, the completed batch queue and full batch history, and `review_complete`; no duplicate finding text |
| [evidence/](evidence/) | Archived responses and historical instructions, clearly separate from active instructions. Includes the final consolidated assessment and the preserved point-in-time snapshot of all 286 implementation issues retrieved in iteration 35 |
| [evidence/source-manifest.json](evidence/source-manifest.json) | Provenance, original/published hashes and privacy transformations |

This is review working material, not a replacement Goal, Implementation Guide, Roadmap or research program. Changes suggested by the review are not approved merely by appearing here.

## Completion criteria

The purpose is to finish the review, not maintain an endless reading queue. Reuse existing work; a section already reviewed does not require another read because a pass number or AI provider changes.

| Original area | Current position | Finish condition |
|---|---|---|
| 1. Baseline/planning documents | Completed at the recorded baseline | Reuse it; check relevant subsequent changes only |
| 2. Standards | Complete for the carried checks: six carried groups closed (SWE quality, array flags, Features Part 3/CQL2 identifiers, SensorML class identifiers, IDR-011 Section 14.3 abstract-test rows, Part 2 Annex A.1 inheritance) and the F-13 extension question resolved | Met; later passes may raise targeted standards questions, each recorded in `remaining_checks` with evidence |
| 3. Research and peer evidence | **Complete for the recorded scope.** All 71 reports accounted: 18 fully read including all four committed deep reads, 53 screened at their key sections; no partial reads remain; pinned CS-GO/OSH checks complete and closed | **Met.** Executive summaries, bodies, appendices and validation sections were deliberately outside the screen, and each report's entry records what was and was not covered |
| 4. End-to-end scenarios | **Complete for the recorded scope.** All nine Goal capability areas reachable from both a representative scenario and a walkthrough path; no scenario over-promises; four findings given named remedy locations | **Met.** The assessment covers a design artifact: the Guide states these scenarios record a design review, not executed verification |
| 5. Verification quality | **Complete for the recorded scope.** All three criteria strongly established; every planned capability has a layer that can observe its claims; no gap between what the strategy promises and what its layers deliver | **Met.** The assessment covers a written strategy: nothing establishes that any test exists, runs or passes, which the Guide states for itself |
| 6. Implementation issues | **Complete.** All 286 compared against their Roadmap leaves with an empty difference inventory; the whole dependency graph checked (1,676 edges, no cycle, no dangling reference, single root); and **all 286 tasks read and assessed as whole units - all 286 adequate**, with none left at targeted-check depth | **Met.** The assessment covers written scope and acceptance criteria: all 286 issues remain open with an Execution record reading "Not started", so nothing here establishes that any task was executed correctly |
| Final assessment | **Complete.** Published as [the final consolidated assessment](evidence/51-pass-3c-45-final-consolidated-assessment.md) | **Met.** Findings, withdrawals, recommendations sorted by when they matter, twelve open questions, coverage accounting and eight stated limits. Recommendations remain unimplemented, which these criteria expressly permit |

These are coverage criteria, not a prediction of cost or iterations. An explicitly agreed scope exception must remain visible. A subscription limit produces a checkpoint, not a false completion declaration.

A finding can finish as supported, withdrawn, optional or explicitly unresolved with a documented consequence. The review does **not** need every recommendation implemented or every uncertainty eliminated to finish.

## Using the outcome

The published [final consolidated assessment](evidence/51-pass-3c-45-final-consolidated-assessment.md) distinguishes:

- Actions needed before initial implementation work.
- Decisions or fixes needed before a named later issue or milestone.
- Optional improvements.
- Remaining uncertainty and coverage limits.

Every recommendation is tied to an existing Guide, Roadmap or issue owner. None was turned into a blanket prerequisite, a new project or another research cycle: exactly two items are recommended before implementation starts, and both are decisions the project must make for itself.

## Cross-provider continuation

The review is complete, so there is no iteration to resume. If the project authorizes further work later, give that reviewer this folder's URL and say:

> Read the final consolidated assessment in evidence/, then README.md, instructions.md and findings.md. This review is closed: `review_complete` is true and the batch queue is exhausted. Do not treat it as an open queue or restart it. Work only within the new scope you have been given, record disagreements against the existing finding IDs rather than creating competing ones, and publish before handing back.

The provider/model and actual evidence should be recorded when known. Unknown model names stay unknown. Repository commits are the durable handoff; private model memory and old ZIP exports are not the authority.

# Pass 3c iteration 40 - queue batch 22, Phase 3 task assessment

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed` for batch 22, with an instruction to refresh the fully-assessed scope record by deriving it from the issue records rather than maintaining separate arithmetic.
**Source:** The preserved snapshot [40-batch17-issue-snapshot.json](40-batch17-issue-snapshot.json). No issue was re-fetched and no network request was made.

**All 39 batch 22 tasks are accounted for. Batch 22 is complete.**

---

## 1. Coverage

Issues #89 to #127, Phase 3, assessed in five groups at Roadmap capability-group boundaries.

| Group | Title | Issues | Assessed | Adequate |
|---|---|---|---|---|
| 3.1 | Register and manage DataStreams and expose schemas | #89-#97 | 9 | 9 |
| 3.2 | Implement observation storage and required writes | #98-#107 | 10 | 10 |
| 3.3 | Implement observation and stream queries | #108-#114 | 7 | 7 |
| 3.4 | Implement status observations and System Events | #115-#120 | 6 | 6 |
| 3.5 | Verify JSON observation workflows and continuity | #121-#127 | 7 | 7 |
| | | **39** | **39** | **39** |

Each task was assessed as one unit. **No task's scope or acceptance criteria was found inadequate, and no new finding results.**

---

## 2. The counting record is now derived, not maintained

The instruction was to stop keeping the fully-assessed scope as prose arithmetic. Its previous value named batches 19 and 20 only, carried the superseded range `#3-#57`, and repeated the thirteen figure corrected in iteration 39. Prose restating counts drifts every time a batch lands, which is how all three earlier counting errors arose.

The record is now **computed from the per-issue entries themselves** at each update:

- **Assessed** issues carry `review_status` beginning `scope_and_acceptance_assessed_`, so both the count and the contributing batches are read off the entries.
- **Targeted-only** issues carry `review_status: targeted_prior_review`.
- **Unique read** is the count of entries holding any reviewer evidence pointer.

No range or total is written by hand. The three quantities reconcile by construction rather than by arithmetic anyone has to repeat:

> assessed + targeted-only = unique read

After this batch: **125 + 7 = 132.**

---

## 3. What these tasks do well

The Phase 3 tasks are where observation semantics first meet storage and query, and they carry the Guide's hardest rules verbatim.

**They reproduce the exact rules the Guide states.** #105 forbids permitting a populated-stream schema change "merely by calling them compatible or storing an internal revision", which is Guide line 434 almost word for word. #106 requires that absent cascade and `cascade=false` yield `409` while authorized `cascade=true` removes exactly the prescribed resources and malformed values fail, which is line 509. #107 excludes making the retry key mandatory and excludes deduplicating "distinct measurements by equal values", both from line 505. #115 implements the freshness assessment "without a universal readiness score", which is line 483. #120 requires that "ordinary observation arrival or metadata edits do not silently create System Events", which is line 485.

**The latest-selection task is exactly right.** #112 applies `resultTime=latest` "after authorized route scope and all other native predicates, retaining ties", and excludes using "ingestion order as freshness, discard[ing] equal-time ties or select[ing] a global latest observation before scope". That is Guide line 440's rule in full, including the tie retention that the peer implementation examined in iteration 26 does not honour. Its fixtures seed "exact-time ties, a newer excluded feature, a newer denied observation and late-arriving older results", so the discriminating cases are built in.

**They refuse to promote placeholder vocabulary.** #119 excludes "conversion of x-OGC/TBD examples into definitive OGC vocabulary" and requires that "placeholder vocabulary is not promoted to an invented OGC term". This is the same discipline Guide line 560 applies to relation names, applied here to event types.

**They keep capability separate from evidence.** #110 forbids treating "stream capability candidates as evidence that a property was observed" and forbids inferring "property identity from labels/units", which are Guide lines 483 and 424.

**They model the documented paging choice.** #113 excludes promising "cross-request snapshot consistency" and tests "changing-view semantics" by scheduling inserts and deletes between requests, matching Guide line 444.

---

## 4. Two observations confirmed, neither raised

**F-19's pattern holds in the Phase 3 restore task.** Batch 19 recorded that the first restore task, #26, verifies identity, semantic fields, revisions and artifacts without mentioning audit. #126 extends restore regression to "streams/contracts, exact values, quality/source artifacts, event parents and deletion evidence". Deletion evidence is present; audit records are again absent. Two restore tasks in two phases now show the same shape. The design exists at Guide lines 602 and 529, and no task yet checks that audit survives a restore. Recorded, not raised, and unchanged in disposition.

**The disclosure discipline is applied correctly where it is applied.** #125 compares "two otherwise identical fixtures differing only in protected contributor facts" and requires the permitted view to be unchanged, which is the pattern Guide line 975 prescribes. #97 builds an access matrix for stream descriptions, format lists, schema wrappers, source artifacts, association links and generated examples, and varies canonical, nested and collection paths for denied callers. Consistent with the Guide's own channel enumeration, neither names a cache or validator channel. Per the iteration 39 clarification this is recorded as an observation about the Guide's consistency and is **not** offered as evidence about F-12's residual.

---

## 5. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Coverage | **39 of 39 assessed, batch complete** | §1 |
| Adequacy | **All 39 adequate.** No new finding number and no new follow-up question | §1 |
| Counting record | **Now derived from the per-issue entries**, not maintained as prose. Assessed + targeted-only = unique by construction | §2 |
| Rule fidelity | Guide lines 424, 434, 440, 444, 483, 485, 505, 509 and 560 each reproduced by a named task | §3 |
| F-19 | Second restore task also omits audit. Two phases, same shape. Recorded, not raised | §4 |
| F-12 | No claim made; the disclosure observations are explicitly not offered as evidence, per iteration 39 | §4 |

**Remaining: 5 batches of 27.** Next selected batch is **batch 23 of 27**, issues #128 to #155.

---

## 6. Statement of limits

This iteration read the task-specific content of 39 issues from the preserved snapshot and the Guide lines needed to judge them. It made no network request and re-fetched nothing.

**Unique reviewer-read issues: 132 of 286.** Three of this batch's issues, #98, #99 and #104, already carried prior evidence. Tasks assessed for scope and acceptance: 125. Read for targeted checks only: 7. The remaining 154 issues have been fetched and compared mechanically but not read.

This assesses written scope and acceptance criteria against the Guide. It does not establish that any task will be executed correctly or that its acceptance criteria will be applied honestly. All 286 issues remain open with an Execution record reading "Not started".

No count of tasks stating the F-12 boundary was made or is implied, following the iteration 39 withdrawal.

Evidence reports 31 through 42 are preserved unchanged. Reports 43 and 44 are preserved with corrections appended; their authored bodies are unchanged.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. Nothing was written to the implementation repository. `review_complete` remains `false`.

---

## Appendix A - Correction appended in iteration 41 (September 20, 2026)

Everything above this rule is the report as authored in iteration 40 and is unchanged.

**What is withdrawn.** Section 3 says the latest-selection task carries Guide line 440 in full "including the tie retention that the peer implementation examined in iteration 26 does not honour". **The clause about the peer is withdrawn.** It restates a claim this review already withdrew.

**Why.** Iteration 27 withdrew it, and the reasoning is recorded in [32-pass-3c-27-batch8-corrections-and-batch9.md](32-pass-3c-27-batch8-corrections-and-batch9.md) Section 1 and in [31-pass-3c-26-batch8-peer-studies-and-source-checks.md](31-pass-3c-26-batch8-peer-studies-and-source-checks.md) Appendix A. The peer fixture examined in iteration 26 seeds a single newest observation, so it never exercises tied result times and cannot show how that implementation behaves when two observations share the greatest result time. Saying it "does not honour" tie retention asserts exactly what the fixture could not establish.

**What the sentence should say.** Issue #112 applies `resultTime=latest` after authorized route scope and all other native predicates and retains ties, which is Guide line 440 in full. Its fixtures seed exact-time ties, a newer excluded feature, a newer denied observation and late-arriving older results, so the discriminating cases are built in. **No comparison with any peer implementation is needed or supported**, and none is made.

**What does not change.** The Glaux task assessment is untouched: all 39 tasks in this batch remain adequate, and #112 remains correct against Guide line 440. No peer research is reopened, and nothing here revisits what the peer does or does not do; the point is that this review has no basis to say either way.

The same clause was carried into the review summaries, the batch queue note and the batch history, and all are corrected in place.

Recorded in [47-pass-3c-41-batch23-task-assessment.md](47-pass-3c-41-batch23-task-assessment.md), Section 1.

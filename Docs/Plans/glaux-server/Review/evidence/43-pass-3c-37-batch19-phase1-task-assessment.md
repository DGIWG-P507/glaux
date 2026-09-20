# Pass 3c iteration 37 - queue batch 19, Phase 1 task assessment

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed` for batch 19, with instructions to assess scope, acceptance criteria and added fields together, reuse completed assessments, work in small groups and checkpoint after each.
**Source:** The preserved snapshot [40-batch17-issue-snapshot.json](40-batch17-issue-snapshot.json). No issue was re-fetched and no network request was made.

**All 24 tasks are accounted for. Batch 19 is complete.**

---

## 1. Coverage

Issues #3 to #26, Phase 1, assessed in five groups at the Roadmap's capability-group boundaries.

| Group | Title | Issues | Assessed | Adequate |
|---|---|---|---|---|
| 1.1 | Establish the build and test environment | #3-#6 | 4 | 4 |
| 1.2 | Prove shared validation and value primitives | #7-#12 | 6 | 6 |
| 1.3 | Establish the transactional storage boundary | #13-#17 | 5 | 5 |
| 1.4 | Establish HTTP, discovery and access enforcement | #18-#23 | 6 | 6 |
| 1.5 | Deliver the first persisted System workflow | #24-#26 | 3 | 3 |
| | | **24** | **24** | **24** |

Each task was assessed as one unit: deliverable, exclusions, prerequisites and environment constraints, acceptance criteria and verification approach together. **No task's scope or acceptance criteria was found inadequate against the Guide, and no new finding results.**

---

## 2. The shared version-pinning passage, assessed once

Every issue's Sources section carries the same construction, so it is assessed once here and carried to all 286.

All 286 issues cite **Guide v1.2** and **Goal v1.7**. The current documents are Guide v1.3 and Goal v1.8. The Roadmap reference varies from v1.1 to v1.9 across thirteen distinct planning commits, reflecting publication in authorized batches over time; the current Roadmap is v1.18.

That drift is sound, for four reasons established by inspection rather than assumed.

1. **It is disclosed and instructed.** Every issue states: "Record the version/commit used to prepare this issue; resolve later approved changes before execution."
2. **The links do not rot.** There are 2,363 commit-pinned `/blob/<40-hex>/` links across the 286 issues, roughly eight per issue, so every Guide and Roadmap reference resolves to the exact text it was written against.
3. **The one deliberately current link is the right one.** Exactly 286 `/blob/main/` links exist, one per issue, and all 286 are the CONTRIBUTING guide, where currency rather than pinning is what a contributor needs.
4. **The intervening Guide change does not reach them.** Guide v1.3's change control records that it re-paired §1.1 with the Goal's capability headings and that "the previous §1.1 anchors" changed. **Zero issues cite a Guide §1.1 anchor**, so no issue link is affected.

Assessment: sound. Not a finding.

---

## 3. What the tasks do well, with the Guide lines they answer

A representative account rather than an exhaustive one, since all 24 were adequate.

- **#3** is inspection-only and declares runtime, red-green, mutation and fuzz checks inapplicable *with an explanation*, which Guide §8.1.1 expressly permits for prerequisite-inspection tasks. Its acceptance forbids calling a found executable a successful build.
- **#6** implements Guide §8.1.1's false-green bullet in full: an intentionally failing assertion, a failed service setup, a runner error and an accidentally empty or filtered required suite must each prevent a false successful required result, with counts and initial failures preserved rather than retried to green.
- **#5** requires asserting actual database identity, version and PostGIS availability "rather than accepting a successful mock or socket connection", which is Guide §8.1.1's use-the-layer-that-can-observe rule, and uses bounded readiness checks rather than sleeps.
- **#7** reasons precisely about oracles: "a digest calculated only from the packaged copy cannot establish correspondence to upstream". It then builds its own sensitivity proof, altering one byte and removing a required local target to show each is identified.
- **#9** requires tests that prevent treating an identifier as authorization or observation time, matching Guide line 372 near-verbatim.
- **#16** documents an honest limitation rather than overclaiming: internal locking cannot detect a stale client's lost update when no condition was supplied, which is consistent with Guide line 503 declining a mandatory `If-Match`.
- **#17** requires reauthorization denial before saved-outcome disclosure and excludes deduplicating distinct measurements by equal values, both of which Guide line 505 states.
- **#21** states "retries cannot establish safety by eventually succeeding", which is Guide §8.1.1's no-retry-until-green rule applied to credential failures.
- **#22** fails the policy adapter and asserts that no network-facing operation defaults to allow, matching Guide line 598, and verifies that authentication does not establish producer or reporting authority.
- **#23** states that "Serving OpenAPI 3.1 alone does not establish the separate OAS 3.0 class", which is Guide line 362 exactly.
- **#25** requires that "a disposable wrong-ID/field response or replacement of persistence with process memory must be detected" and that "status-only or count-only checks are insufficient", matching Guide line 955.

Two scoping habits recur and are worth recording. Every task's "Not included" field names the sibling that owns what it excludes, for example #5 deferring CI orchestration to 1.1.4 and #24 deferring the retrieval and restart proof to 1.5.2. And several tasks forbid a specific wrong shortcut rather than only stating the goal, for example #11 excluding "adopting PostgreSQL timestamp precision as an excuse to lose finer source precision" and #18 excluding destructive startup migrations.

---

## 4. F-12 gains quantified support, and the support cuts toward the recorded remedy

F-12's residual, relocated to Guide line 901 in iteration 34, is whether a runner may decode a response using the production code's own types while holding independently authored expected values. Line 901 forbids the server's serializers *as the sole oracle* and permits *ordinary* format libraries, and a production DTO is neither.

**Six of this batch's 24 tasks state that boundary explicitly**, in their own words:

| Issue | Wording |
|---|---|
| #5 | assert actual database identity "rather than accepting a successful mock or socket connection" |
| #7 | fixtures derive expected meaning "from the source clauses, not server output" |
| #10 | assert exact equality and ordering "not merely encoder/decoder agreement" |
| #14 | "The expected bytes must not come from the server's reserialization", and "a successful round-trip through the same faulty conversion is insufficient" |
| #16 | author the expected-outcome table "independently of the locking code" |
| #20 | construct valid and invalid tokens "independently of server decoding" |

Three more in this batch state it in wordings a pattern search does not catch: #23's "not solely from the server's route metadata", #24's "rather than copying expected values from the serializer", and #26's "comparing against that manifest rather than newly generated expected output".

Across all 286 issues, a broad search finds the boundary stated in **at least 24**, and that number is a **lower bound**, demonstrated twice in this batch: a first pattern found four, a broadened pattern found 24, and three further instances here still fell outside it. A separate 66 issues use expected-value independence wording, which constrains where the expectation comes from but not how the actual response is decoded, and is therefore not the same rule.

**This is the strongest support F-12 has had, and it points at the recorded remedy.** The boundary is stated where an issue author happened to think of it, in ad-hoc wording, concentrated in foundational tasks; six of 24 here against roughly 24 of 286 overall. A single clause on Guide line 901 would apply to all 286 uniformly. The disposition does not change: narrowed optional hardening, Low severity. What changes is that the gap now has a measured shape rather than an argued one.

One correction to iteration 35 follows. That iteration recorded that the shared boilerplate "carries the Guide's verification practices into every issue". That is right for the false-green rule and for independent expected results, and it is **not** right for this boundary: the checklist's phrase is "use independent expected results", which is the expectation side. The boilerplate does not close the actual side either.

---

## 5. Two earlier observations confirmed at task level

**F-19's implementing task is #26, and its acceptance does not mention audit.** Batch 15 recorded that Guide §8.2's scenario 6 restores a backup and verifies "resource meaning, identity, deletion handling, and held command work" without checking audit. Task 1.5.3 is that scenario's first implementation. Its acceptance requires that "the independent System workflow passes after restore, exact context is preserved, no outgoing work is delivered and the original database is untouched", and its verification builds a manifest of "canonical identity, semantic fields, existing revision/context records and exact retained artifact bytes/digests". Audit records appear in neither. The task is otherwise strong, requiring the restore destination be verified different from the source before any destructive operation and requiring a corrupt-artifact case to demonstrably reject an incomplete clone. The observation is unchanged and still not raised: Guide line 602 sets the audit floor and line 529 requires restore testing, so the design exists; no task in Phase 1 checks that audit survives it.

**The UUIDv7 observation from iteration 28 holds at task level.** Task 1.2.3, issue #9, requires tests preventing an identifier being treated as authorization or observation time, which is the time-authority half of the risk IDR-016 records. The privacy half, creation-activity inference from the embedded timestamp, is absent here as it is from Guide line 372, consistent with the research routing it to IDR-039/040. Recorded, not raised.

---

## 6. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Coverage | **24 of 24 tasks assessed, batch complete.** Five groups at Roadmap boundaries | §1 |
| Adequacy | **All 24 adequate.** No task's scope or acceptance criteria found inadequate against the Guide | §1 |
| Shared version-pinning passage | Assessed once and carried to all 286. **Sound**: disclosed, commit-pinned where stability matters, current-tracking where currency matters, and the one Guide change since touches anchors no issue cites | §2 |
| **F-12** | **Quantified support, strongest yet.** The boundary is stated in at least 24 of 286 in ad-hoc wording, six explicitly in this batch; a lower bound demonstrated twice. Points at the recorded one-clause remedy at Guide line 901 | §4 |
| Iteration 35 correction | The boilerplate carries the false-green and expected-result rules but **not** the actual-side boundary | §4 |
| F-19 | Implementing task identified as #26; its acceptance does not mention audit. Observation unchanged, not raised | §5 |
| UUIDv7 observation | Holds at task level in #9. Recorded, not raised | §5 |
| New findings | None. No new finding number and no new follow-up question | |

**Remaining: 8 batches of 27.** Next selected batch is **batch 20 of 27**, Phase 2 tasks.

---

## 7. Statement of limits

This iteration read the task-specific content of 24 issues from the preserved snapshot and the Guide lines needed to judge them. It made no network request and re-fetched nothing. It did not repeat batch 17's comparison or batch 18's graph analysis.

**The documented issue read count rises from 18 to 42**: the 24 assessed here were read by a reviewer, not merely compared mechanically. The remaining 262 have been fetched and compared but not read.

This assesses written scope and acceptance criteria against the Guide. It does not establish that any task will be executed correctly, that its acceptance criteria will be applied honestly, or that the implementation exists. No task has been executed; all 286 issues remain open with an Execution record reading "Not started".

The corpus-wide counts in §4 come from pattern searches whose limits are recorded in §4 itself: the figure of 24 is a lower bound, and this batch found three instances outside the broadened pattern.

Evidence reports 31 through 42 are preserved unchanged.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. Nothing was written to the implementation repository. `review_complete` remains `false`.

---

## Appendix A - Correction appended in iteration 38 (September 20, 2026)

Everything above this rule is the report as authored in iteration 37 and is unchanged. This appendix is added so that a reader consulting this report alone finds the correction.

**What is corrected.** Section 7 states: "The documented issue read count rises from 18 to 42." That figure is wrong. It was produced by adding the 24 issues read in this batch to the 18 recorded before it, without removing the issues counted twice.

**The correct figure is 37, derived from distinct issue IDs rather than from batch totals.** Eighteen issues carried a reviewer evidence pointer before this batch: #3, #15, #19, #21, #22, #56, #71, #72, #98, #99, #104, #130, #156, #163, #164, #168, #174 and #240. Batch 19 read #3 to #26. **Five of those eighteen fall inside that range: #3, #15, #19, #21 and #22.** So the unique count is 18 + 24 − 5 = 37.

**Two measures must stay distinct, and Section 7 blurred them.**

| Measure | Value | Meaning |
|---|---|---|
| Unique issues read by a reviewer | **37** | Distinct issue IDs carrying any reviewer evidence pointer |
| Phase 1 tasks fully assessed | **24** | Issues #3-#26, assessed for scope and acceptance adequacy in this batch |

These are not the same thing and neither substitutes for the other. The thirteen issues read in earlier passes outside #3-#26 were read for targeted checks, not assessed for scope and acceptance adequacy. The five inside the range had prior targeted checks that this batch reused rather than repeated, and they are now also fully assessed.

**What does not change.** Every substantive result in this report stands: all 24 Phase 1 tasks are adequate, the shared version-pinning passage is sound, the F-12 measurement in Section 4 is unaffected, and the F-19 and UUIDv7 observations in Section 5 are unaffected. The error was in one arithmetic sentence about coverage bookkeeping, not in any assessment.

**Where the corrected figures are recorded.** `backlog_analysis.issue_read_count` in the review state now carries the unique count, its derivation from IDs, the named overlap, and the separate fully-assessed count. The 24 per-issue entries for #3-#26 carry this report as an evidence pointer, with the five overlapping entries retaining their earlier pointers alongside.

Recorded in [44-pass-3c-38-batch20-phase2-task-assessment.md](44-pass-3c-38-batch20-phase2-task-assessment.md), Section 1.

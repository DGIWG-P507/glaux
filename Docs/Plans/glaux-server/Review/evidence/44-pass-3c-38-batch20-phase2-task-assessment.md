# Pass 3c iteration 38 - batch 19 reconciliation, then queue batch 20

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed` directing a reconciliation of the per-issue records against batch 19, a corrected read count derived from issue IDs, and then batch 20.
**Source:** The preserved snapshot [40-batch17-issue-snapshot.json](40-batch17-issue-snapshot.json). No issue was re-fetched and no network request was made.

**All 31 batch 20 tasks are accounted for. Batch 20 is complete.**

---

## 1. Batch 19 reconciliation, and the corrected read count

### 1.1 The arithmetic was wrong

Iteration 37 published "the documented issue read count rises from 18 to 42." That figure added the 24 issues read in batch 19 to the 18 recorded before it, without removing the issues counted twice.

**Eighteen issues carried a reviewer evidence pointer before batch 19:** #3, #15, #19, #21, #22, #56, #71, #72, #98, #99, #104, #130, #156, #163, #164, #168, #174, #240. Batch 19 read #3 to #26. **Five of the eighteen fall inside that range: #3, #15, #19, #21, #22.**

> 18 + 24 − 5 = **37 unique**, not 42.

The figure is now derived from distinct issue IDs carrying an evidence pointer, not from adding batch totals, so it cannot drift the same way again.

### 1.2 Two measures, kept distinct

| Measure | Value | What it means |
|---|---|---|
| Unique issues read by a reviewer | **37** | Distinct issue IDs carrying any reviewer evidence pointer |
| Tasks fully assessed for scope and acceptance | **55** | Batch 19's 24 plus batch 20's 31, no overlap between the two batches |

These are different and neither substitutes for the other. The thirteen issues read in earlier passes outside #3-#26 were read for targeted checks, not assessed for scope and acceptance adequacy. The five inside the range had prior targeted checks that batch 19 reused rather than repeated, and are now also fully assessed.

After this batch the unique read count is **67**. Batch 20 read 31 issues, and **one of them, #56, was already in the prior eighteen**, so 37 + 31 − 1 = 67. The unique-ID derivation caught that overlap immediately, which is the point of deriving the figure from IDs rather than from batch totals: the same class of error that produced 42 in iteration 37 was detected here before publication rather than after.

### 1.3 What was reconciled

All 24 per-issue entries for #3-#26 now carry the batch 19 report as an evidence pointer, with the five overlapping entries retaining their earlier pointers alongside, and a conclusion recording that scope and acceptance were assessed and found adequate. `backlog_analysis.issue_read_count` carries the unique count, its derivation, the named overlap and the separate fully-assessed count.

Evidence report 43 is preserved: its authored body is unchanged, verified by hashing its first 12,963 bytes to the recorded original, and the correction is appended as its Appendix A so a reader consulting that report alone finds it.

**Every substantive result in batch 19 stands.** The error was in one arithmetic sentence about coverage bookkeeping.

---

## 2. A batch-boundary inaccuracy, corrected before assessing

The saved batch 20 scope is titled "groups 2.1-2.3" and its `size_basis` says the split falls "at Roadmap group boundaries". It does not. Groups 2.1, 2.2 and 2.3 together hold **32** leaves spanning issues **#27-#58**, while the batch covers **#27-#57**, 31 issues. Issue #58 is leaf 2.3.10, the last child of group 2.3.

**No issue is orphaned.** A contiguity check across all eight issue batches confirms the ranges are contiguous with no gap and no overlap, every batch's stated count matches its actual range length, and the eight ranges total 286:

```
19: #3-#26 (24)    20: #27-#57 (31)   21: #58-#88 (31)   22: #89-#127 (39)
23: #128-#155 (28) 24: #156-#197 (42) 25: #198-#238 (41) 26: #239-#288 (50)
```

Batch 21 begins at #58 and therefore picks up leaf 2.3.10. The inaccuracy is in the title and sizing note, not in coverage. Both were corrected before assessment, and batch 21's note records that it begins mid-group, so a later reader does not infer a group-complete assessment from a batch title.

This is the fourth defect found in the review's own saved state, after the screen-scope wording, the batch 15 section references and the batch 17 repository assumption.

---

## 3. Batch 20 coverage

Issues #27 to #57, assessed in three groups.

| Group | Title | Issues | Assessed | Adequate |
|---|---|---|---|---|
| 2.1 | Complete the shared SWE component model | #27-#38 | 12 | 12 |
| 2.2 | Complete SensorML and GeoJSON representation mappings | #39-#48 | 10 | 10 |
| 2.3 (nine of ten leaves) | Complete Part 1 resources, hierarchy and collections | #49-#57 | 9 | 9 |
| | | **31** | **31** | **31** |

Each task was assessed as one unit. **No task's scope or acceptance criteria was found inadequate, and no new finding results.**

---

## 4. What these tasks do well

Three habits recur across all 31 and are worth naming, because each answers a concern this review holds.

**They refuse to invent rules the source does not contain.** #27 excludes imposing "a Boolean constraint form absent from the applicable source contract"; #28 excludes any "imposed finite-only Quantity rule"; #30 says to "determine ordering from the applicable numeric/time semantics or supplied Category code-space evidence, not alphabetical guesses" and not to "invent a blanket invalid-bounds rule beyond the source"; #29 requires failing "unsupported conversion explicitly rather than inventing UTC". This is the same discipline the review applies when it declines to treat unadopted research as required.

**They construct discriminating cases rather than confirming ones.** #32 requires that "a fixture that selects one arm while supplying another's value must not pass merely because some alternative validates". #43 and #52 both pair similar labels with distinct URIs so that label-based conflation cannot pass, which is Guide line 424's rule applied as a test design. #44 uses "asymmetric coordinate values and a nonzero supplied height to expose axis swaps and lost dimensions". #37 deliberately selects "a neighboring record's otherwise valid quality" to prove the binding is exact.

**They tie directly to specific Guide lines.** #36 states that "an upstream JSON-schema pass alone is not proof" and forbids "inventing zero uncertainty or a pass", both of which Guide line 410 says; #37 implements line 412's local reference, occurrence, duplicate-ID and cycle rules; #38 compiles contract A then contract B and asserts A's meaning is unchanged, which is line 408; #48 requires denying a representation rather than stripping fields, which is line 600; #57 routes Property Definitions to SensorML rather than GeoJSON, which is line 739.

---

## 5. F-12: the measurement tightens, and the shape holds

Six of this batch's 31 tasks state the actual-side independence boundary explicitly: **#28, #31, #40, #44, #49, #56**. The clearest is #44, which requires constructing expectations "not by converting the serializer's output into an oracle". #31 forbids resting on "an aggregate count or self-generated golden", #28 and #40 say encoder and decoder agreement alone is insufficient, and #49 derives expectations "independently of encoder output".

| | Stating the boundary | Of |
|---|---|---|
| Batch 19, Phase 1 | 6 | 24 |
| Batch 20, Phase 2a | 6 | 31 |
| **Assessed so far** | **12** | **55** |
| Pattern search, whole corpus | 28 | 286 |

Reading 55 tasks finds the boundary in 12, about 22 percent. A pattern search across all 286 finds 28, about 10 percent. **The pattern under-detects by roughly half**, which is the third independent confirmation that the corpus figure is a lower bound; the earlier two came from widening the pattern twice inside batch 19.

The shape of the finding is unchanged and is now measured rather than argued. The rule is real, the project plainly believes in it, and it is written where an issue author happened to think of it, in varied wording, in a minority of tasks. One clause on Guide line 901 would apply to all 286 uniformly. Disposition unchanged: narrowed optional hardening, Low severity.

---

## 6. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| **Read count** | **Corrected from 42 to 37**, derived from distinct issue IDs. Five-issue overlap named | §1.1 |
| Two measures | Unique read (37, now 67) kept distinct from tasks fully assessed (55) | §1.2 |
| Per-issue records | All 24 batch 19 entries reconciled with evidence pointers and conclusions | §1.3 |
| Archived report | **Preserved**, body byte-for-byte unchanged, correction appended as Appendix A | §1.3 |
| Batch boundary | Title and sizing note claimed a group boundary that does not exist. **Corrected; no coverage gap**, contiguity verified across all eight issue batches | §2 |
| Batch 20 coverage | **31 of 31 assessed, batch complete** | §3 |
| Adequacy | **All 31 adequate.** No new finding number and no new follow-up question | §3 |
| **F-12** | Measurement tightens: 12 of 55 assessed tasks state the boundary against 28 of 286 by pattern. **Third confirmation that the corpus figure is a lower bound** | §5 |

**Remaining: 7 batches of 27.** Next selected batch is **batch 21 of 27**, issues #58 to #88, beginning with leaf 2.3.10.

---

## 7. Statement of limits

This iteration read the task-specific content of 31 issues from the preserved snapshot and the Guide lines needed to judge them. It made no network request and re-fetched nothing.

**Unique reviewer-read issues: 67 of 286.** Tasks fully assessed for scope and acceptance: 55. The remaining 219 issues have been fetched and compared mechanically but not read.

This assesses written scope and acceptance criteria against the Guide. It does not establish that any task will be executed correctly or that its acceptance criteria will be applied honestly. All 286 issues remain open with an Execution record reading "Not started".

The corpus counts in §5 come from a pattern search whose under-detection is measured in §5 itself rather than assumed.

Evidence reports 31 through 42 are preserved unchanged. Report 43 is preserved with a correction appended as its Appendix A; its authored body is byte-for-byte unchanged.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. Nothing was written to the implementation repository. `review_complete` remains `false`.

---

## Appendix A - Correction appended in iteration 39 (September 20, 2026)

Everything above this rule is the report as authored in iteration 38 and is unchanged.

**1. The targeted-only count is 12, not thirteen.** Section 1.2 says "The thirteen issues read in earlier passes outside #3-#57". Six of the eighteen prior-evidence issues fall inside the assessed ranges, #3, #15, #19, #21, #22 and #56, so **twelve** sit outside: #71, #72, #98, #99, #104, #130, #156, #163, #164, #168, #174 and #240. The figures reconcile exactly: 55 assessed + 12 targeted-only = 67 unique.

**2. Section 5's "under-detects by roughly half" is withdrawn.** That inference compared a reading count of 12 in 55 against a pattern count of 28 in 286 and concluded the pattern misses about half. It does not hold, because **both counts measure the same conflated category rather than the boundary F-12 records.**

Two rules were run together:

| Rule | What it requires | Present in the cited wording |
|---|---|---|
| Expected-answer independence | Do not derive the expected answer from the server's own output | Yes, this is what nearly all of it says |
| F-12's residual | Whether the runner may interpret the *actual response* through production wire types while holding an independent expectation | Almost never |

Phrases such as "not by converting the serializer's output into an oracle", "self-generated golden" and "independently of server decoding" constrain where the expectation comes from. They do not say how the actual response may be decoded. Issue #14's "a successful round-trip through the same faulty conversion is insufficient" comes closest, and even it speaks to proof structure rather than to response interpretation in a contract test.

So neither the 12 nor the 28 is a count of tasks addressing F-12's boundary, and no ratio between them supports a claim about detection. **The comparison, the percentages and the under-detection conclusion are withdrawn.** No replacement census is required or implied: the question was not one a corpus count could answer.

**What stands.** The six examples in Section 5 are accurate quotations and good practice, retained as observations about oracle discipline in the work instructions. The batch 20 assessment result, all 31 tasks adequate, is unaffected. **F-12 is unchanged and not weakened**: it rests on the Guide-text analysis from iteration 34, that Guide line 901 forbids the server's serializers as the *sole* oracle and permits *ordinary* format libraries while a production DTO is neither. That basis never depended on these counts.

Recorded in [45-pass-3c-39-batch21-task-assessment.md](45-pass-3c-39-batch21-task-assessment.md), Section 1.

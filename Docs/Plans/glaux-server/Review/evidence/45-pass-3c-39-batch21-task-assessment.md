# Pass 3c iteration 39 - two corrections, then queue batch 21

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed` directing a corrected targeted-only count, a bounded F-12 clarification, and then batch 21.
**Source:** The preserved snapshot [40-batch17-issue-snapshot.json](40-batch17-issue-snapshot.json). No issue was re-fetched and no network request was made.

**All 31 batch 21 tasks are accounted for. Batch 21 is complete.**

---

## 1. Two corrections, applied before assessing

### 1.1 The targeted-only count is 12

Reports 43 and 44 both say "thirteen issues read in earlier passes" sat outside the assessed ranges. That subtracted only the five issues overlapping batch 19. **Issue #56 also carries prior evidence and falls inside batch 20's range**, so six of the eighteen prior-evidence issues sit inside the assessed ranges and twelve sit outside: #71, #72, #98, #99, #104, #130, #156, #163, #164, #168, #174, #240.

The figures now reconcile exactly:

> 55 assessed + 12 targeted-only = **67 unique**

### 1.2 The F-12 tallies conflated two rules, and are withdrawn

Iterations 37 and 38 counted tasks said to state "the actual-side independence boundary", reported 12 of 55 assessed against 28 of 286 by pattern, and concluded the pattern "under-detects by roughly half". **That conclusion, those percentages and the comparison are withdrawn.**

They ran two different rules together:

| Rule | What it requires | Present in the quoted wording |
|---|---|---|
| **Expected-answer independence** | Do not derive the expected answer from the server's own output | Yes, this is what nearly all of it says |
| **F-12's residual** | Whether a runner may interpret the *actual response* by deserializing it into the production crate's wire types, while holding an independently authored expectation | Almost never |

"Not by converting the serializer's output into an oracle", "self-generated golden" and "independently of server decoding" all constrain where the expectation comes from. None says how the actual response may be decoded. Issue #14's "a successful round-trip through the same faulty conversion is insufficient" comes closest, and even that speaks to proof structure rather than to response interpretation in a contract test.

So neither tally counted the boundary in question, and no ratio between them supports a claim about detection. **No replacement census is needed or implied**: a corpus count was never going to answer this, which is why the finding rests on text analysis instead.

**What stands.** The quoted examples are accurate and are retained as observations about oracle discipline in the work instructions. **F-12 is unchanged and is not weakened.** It rests where iteration 34 put it: Guide line 901 forbids the server's serializers *as the sole oracle* and permits *ordinary* format libraries, and a production DTO is neither, so the sentence leaves the question open. That basis never depended on the withdrawn counts.

Reports 43 and 44 are preserved with their authored bodies unchanged; report 43's is verified against its recorded hash. Both carry the corrections as appendices so a reader of either alone finds them.

---

## 2. Batch 21 coverage

Issues #58 to #88, assessed in four groups. The batch begins mid-group at leaf 2.3.10, as the boundary correction in iteration 38 recorded.

| Group | Title | Issues | Assessed | Adequate |
|---|---|---|---|---|
| 2.3 (final leaf) | Complete Part 1 resources, hierarchy and collections | #58 | 1 | 1 |
| 2.4 | Complete required writes and representation-safe updates | #59-#68 | 10 | 10 |
| 2.5 | Complete Part 1 native filtering and paging | #69-#79 | 11 | 11 |
| 2.6 | Verify the full Part 1 workflow | #80-#88 | 9 | 9 |
| | | **31** | **31** | **31** |

**No task's scope or acceptance criteria was found inadequate, and no new finding results.**

---

## 3. What these tasks do well

**They refuse research the Guide superseded.** #67 requires deriving conditional-request expectations from the Guide "not superseded mandatory-header research", and excludes any "mandatory If-Match or Idempotency-Key requirement". #84 excludes adopting "later transaction drafts, mandatory If-Match". This is the review's own accepted-research-versus-adopted-mechanism distinction, written into the work instructions and pointed at the right answer.

**They build discriminating spatial and temporal cases.** #76 seeds "a candidate whose bounding box overlaps but whose actual geometry does not" and requires the exact predicate to exclude it, which is Guide lines 442 and 521 on rechecking after index prefilter. #77 requires "fractional distinctions beyond database timestamp resolution" and forbids "replacement of source validity with receipt/commit time", matching Guide line 376.

**They model the Guide's stated paging departure rather than contradicting it.** #78 compares "scripted inserts/deletes against an independently authored changing-view model" and excludes any "cross-request snapshot guarantee". That is exactly Guide line 444's documented choice, and it is the task-level counterpart of the non-adoption this review recorded for the temporal report's snapshot recommendation.

**They keep the disclosure discipline visible.** #79 changes "hidden resources while holding the permitted view fixed" and requires exact IDs and counts to be unaffected, which is Guide lines 610 and 975. #72 requires that wrong-family identifiers "cannot widen scope or expose protected participating Systems".

**They are precise about sources.** #69 cites "Part 1 requirements 38-39 and IDR-011 §§7.1-7.2"; #74 cites "requirements 52-53 and 57-58"; #88 reconciles "all 13 Part 1 target classes" against Guide §§7.1-7.3 with "visible qualifications".

---

## 4. F-09's owning task carries the guard

F-09 records that some abstract-test steps presume collection dereferencing while the selected mappings use inline associations, and that the claimed normative `/deployments/{id}/systems` route was withdrawn. Its recorded affected work is **#81**, and #81 is in this batch.

It carries the guard directly. Its exclusions forbid inferring "routes by appending item IDs to every nested list" and forbid importing "server traversal logic into the conformance runner". Its acceptance requires that "deliberately wrong parents or missing descendants are detected rather than copied from server output", and its verification independently specifies "parent, child and grandchild relationships, canonical identities and exact direct versus recursive sets before retrieval".

Iteration 34 established that Guide line 909 carries the general rule, to preserve both the source procedure and the documented correction and never silently turn a server defect into a passed standard test. **#81 now supplies the specific guard against the wrong route inference.** What neither supplies is the documented adaptation itself: no task names the `deployedSystems` step as an adaptation recorded under line 909. F-09's residual is therefore narrower than before but not closed, and it remains fixture content owned by #81 rather than a Guide gap. Disposition unchanged.

Group 2.6 is the conformance-runner group, and its instructions align with Guide line 901's existing prohibition on importing server selection logic or serializers as the oracle. Consistent with §1.2, that alignment is recorded as an observation and is **not** treated as evidence about F-12's residual.

---

## 5. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Targeted-only count | **Corrected to 12.** 55 + 12 = 67 reconciles exactly | §1.1 |
| **F-12 tallies** | **Withdrawn.** Both counts measured a conflated category; no ratio between them supports a detection claim. No replacement census needed | §1.2 |
| F-12 itself | **Unchanged and not weakened.** Rests on the iteration 34 Guide-text analysis of line 901 | §1.2 |
| Archived reports 43 and 44 | **Preserved**, bodies unchanged, corrections appended as appendices | §1.2 |
| Batch 21 coverage | **31 of 31 assessed, batch complete** | §2 |
| Adequacy | **All 31 adequate.** No new finding number and no new follow-up question | §2 |
| F-09 | Owning task #81 carries the guard against the wrong route inference; the documented adaptation itself is still absent. Residual narrower, not closed | §4 |
| Group 2.6 | Aligns with Guide line 901's existing prohibition; recorded as an observation, not as F-12 evidence | §4 |

**Remaining: 6 batches of 27.** Next selected batch is **batch 22 of 27**, issues #89 to #127.

---

## 6. Statement of limits

This iteration read the task-specific content of 31 issues from the preserved snapshot and the Guide lines needed to judge them. It made no network request and re-fetched nothing.

**Unique reviewer-read issues: 96 of 286**, derived from distinct issue IDs. Two of this batch's issues, #71 and #72, already carried prior evidence, so 67 + 31 − 2 = 96. Tasks fully assessed for scope and acceptance: 86. The remaining 190 issues have been fetched and compared mechanically but not read.

This assesses written scope and acceptance criteria against the Guide. It does not establish that any task will be executed correctly or that its acceptance criteria will be applied honestly. All 286 issues remain open with an Execution record reading "Not started".

Section 1.2 withdraws a quantitative claim without replacing it. That is deliberate: the question F-12 records is not one a corpus count can answer, and substituting a better-targeted count would repeat the error in a narrower form.

Evidence reports 31 through 42 are preserved unchanged. Reports 43 and 44 are preserved with corrections appended; their authored bodies are unchanged.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. Nothing was written to the implementation repository. `review_complete` remains `false`.

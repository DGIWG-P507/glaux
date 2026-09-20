# Pass 3c iteration 41 - a reintroduced claim withdrawn, then queue batch 23

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed` directing correction of a reintroduced peer claim, then batch 23.
**Source:** The preserved snapshot [40-batch17-issue-snapshot.json](40-batch17-issue-snapshot.json). No issue was re-fetched and no network request was made.

**All 28 batch 23 tasks are accounted for. Batch 23 is complete.**

---

## 1. A withdrawn claim, reintroduced and now withdrawn again

Iteration 40 wrote that the latest-selection task carries Guide line 440 in full "including the tie retention that the peer implementation examined in iteration 26 does not honour". **That clause asserts something this review withdrew fourteen iterations ago.**

The reasoning is on the record in [32-pass-3c-27-batch8-corrections-and-batch9.md](32-pass-3c-27-batch8-corrections-and-batch9.md) Section 1 and in [31-pass-3c-26-batch8-peer-studies-and-source-checks.md](31-pass-3c-26-batch8-peer-studies-and-source-checks.md) Appendix A. The peer fixture examined in iteration 26 **seeds a single newest observation**, so it never exercises tied result times and cannot show how that implementation behaves when two observations share the greatest result time. Saying the peer "does not honour" tie retention asserts precisely what the fixture could not establish.

This is worth naming plainly. A withdrawal recorded in an appendix did not stop the claim returning in a later summary written from memory of the underlying work rather than from the corrected record. The correction is therefore applied in **five places**: report 46's appendix, the findings register, the start page, the batch queue note and the batch history entry. In the two state entries the original wording is quoted inside the withdrawal rather than deleted, so the record shows what was claimed and why it is wrong.

**What the sentence should say, and now does.** Issue #112 applies `resultTime=latest` after authorized route scope and all other native predicates and retains ties, which is Guide line 440 in full. Its fixtures seed exact-time ties, a newer excluded feature, a newer denied observation and late-arriving older results, so the discriminating cases are built in. No comparison with any peer implementation is needed or supported, and none is made.

**Nothing else changes.** The Glaux task assessment is untouched: all 39 batch 22 tasks remain adequate and #112 remains correct against line 440. No peer research is reopened. The point is not that the peer honours ties; it is that this review has no basis to say either way.

---

## 2. Batch 23 coverage

Issues #128 to #155, Phase 4, assessed in four groups.

| Group | Title | Issues | Assessed | Adequate |
|---|---|---|---|---|
| 4.1 | Implement SWE JSON payloads | #128-#133 | 6 | 6 |
| 4.2 | Implement SWE Text payloads | #134-#139 | 6 | 6 |
| 4.3 | Implement SWE Binary payloads | #140-#147 | 8 | 8 |
| 4.4 | Integrate all codecs and format negotiation | #148-#155 | 8 | 8 |
| | | **28** | **28** | **28** |

**No task's scope or acceptance criteria was found inadequate, and no new finding results.**

---

## 3. What these tasks do well

Phase 4 is codec work, where a test that round-trips through its own encoder proves nothing. The tasks are unusually careful about that, and each states it in its own terms rather than by formula.

**They refuse same-path proofs.** #129 derives expected JSON tokens "not by generating fixtures with the codec under test". #134 excludes compiling "a JSON serializer's output as the Text oracle", which keeps one implemented codec from becoming the authority for another. #143 asserts octets against "hand-calculated layouts, not Rust struct memory layout" or a codec-generated golden. #152 states the principle outright: "Do not equate round-trip agreement with correctness." #154 compares against "source-derived expectations, not another call to the same serializer."

**They split inputs at adversarial boundaries.** #133, #139 and #147 each feed identical bytes whole and in chunks, including splits inside multi-character delimiters, numeric tokens and multibyte text, and require the incremental result to match the complete-body oracle. #147 splits "at every boundary" and distinguishes incomplete input from end-of-stream truncation.

**They keep declaration honest.** #150 excludes treating "binary descriptors as successful decoding", which is Guide line 422 exactly. #146 preserves opaque blocks "without pretending to decode their external content" and requires unimplemented features to stay "reported as incomplete". #155 states that "the three SWE classes remain unclosed until Phase 5 proves their command-side obligations", which is Guide line 907's rule applied against the temptation to close a class at a phase boundary.

**They implement the Guide's selected interpretations rather than guessing.** #130 tests the record and vector array flags in both settings under "the selected Guide interpretation [that] makes true mean arrays", which is the §13 row this review checked in an earlier standards pass. #132 requires that "ordinary zero and false remain values and arbitrary null does not acquire a nil reason".

---

## 4. F-03 gains its implementing task

Guide line 739 is where F-03's cache instance sits: it requires appropriate `Vary` headers and representation-specific ETags, and omits the `private, no-store` default for protected variants that the accepted research pairs with them.

**#151 is the task that implements line 739.** It uses "a source-derived matrix of available formats, Accept quality values, q=0 exclusions, wildcards, parameters and absent headers to assert exact selected media, Vary behavior and representation-specific validators", and it excludes adding the `f` selector, consistent with the Guide having removed it.

The task implements exactly what the Guide says, and no more. It exercises `Vary` and representation-specific validators; it does not bind a validator to the authorized view, specify any cache directive, or test a cache channel. **That is faithful implementation, not a task defect.** The gap is in the Guide, which is where this finding has always placed it. What the batch adds is the location: if the Guide's line 739 gained the missing half, #151 is the task whose acceptance criteria would carry it.

Recorded as an observation under the existing finding. No new finding number, no change of disposition, and no claim about any other task.

---

## 5. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| **Reintroduced peer claim** | **Withdrawn again**, in five places. The fixture could not establish tied-timestamp behavior | §1 |
| Glaux assessment | **Unchanged.** Batch 22's 39 tasks remain adequate; #112 remains correct against Guide line 440 | §1 |
| Peer research | **Not reopened.** No claim about the peer is made in either direction | §1 |
| Batch 23 coverage | **28 of 28 assessed, batch complete** | §2 |
| Adequacy | **All 28 adequate.** No new finding number and no new follow-up question | §2 |
| Codec oracle discipline | Five tasks state, in their own terms, that a same-path round trip is not proof | §3 |
| **F-03** | **Implementing task identified: #151.** It carries Guide line 739 faithfully and no further; the gap remains in the Guide | §4 |

**Remaining: 4 batches of 27.** Next selected batch is **batch 24 of 27**, issues #156 to #197.

---

## 6. Statement of limits

This iteration read the task-specific content of 28 issues from the preserved snapshot and the Guide lines needed to judge them. It made no network request and re-fetched nothing.

Coverage figures are computed from the per-issue records, not restated here as arithmetic. After this batch: **153 assessed, 6 read for targeted checks only, 159 unique reviewer-read** of 286. One of this batch's issues, #130, already carried prior evidence.

This assesses written scope and acceptance criteria against the Guide. It does not establish that any task will be executed correctly. All 286 issues remain open with an Execution record reading "Not started".

No count of tasks stating the F-12 boundary was made, following the iteration 39 withdrawal. Section 3's observations about codec oracle discipline are recorded qualitatively and are not offered as a measurement of that finding.

Evidence reports 31 through 45 are preserved unchanged except where corrections were appended in earlier iterations. Report 46 is preserved with the §1 correction appended; its authored body is byte-for-byte unchanged, verified against its recorded hash.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. Nothing was written to the implementation repository. `review_complete` remains `false`.

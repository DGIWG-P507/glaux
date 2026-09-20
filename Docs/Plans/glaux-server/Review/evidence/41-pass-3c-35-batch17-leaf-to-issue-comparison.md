# Pass 3c iteration 35 - queue batch 17, complete leaf-to-issue comparison

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed` for batch 17, with explicit instructions to verify the target set, follow pagination, preserve the snapshot, review boilerplate once plus each variant, inspect every reported difference, and not substitute sampling.
**Preserved snapshot:** [40-batch17-issue-snapshot.json](40-batch17-issue-snapshot.json), 2,502,030 bytes.

---

## 1. The saved environment assumption was wrong about the repository

The saved scope said the repository is public and reachable unauthenticated, and that three requests at `per_page=100` cover the backlog. It never named the repository. The first fetch therefore went to `DGIWG-P507/glaux`, the git remote of the planning repository that holds this review, and returned **74 entries, every one a pull request, numbered 1 to 74. No issues at all.**

The implementation issues live in a different repository, **`DGIWG-P507/glaux-server`**. The Roadmap states this in every leaf's issue link, for example `https://github.com/DGIWG-P507/glaux-server/issues/3` at Roadmap line 101, and §12 of the Roadmap links that repository's issue template and contributing guide.

This is the third defect found in the review's own saved state, after the two screen-scope statements reconciled in iteration 28 and the batch 15 section references corrected in iteration 33. It is the most consequential of the three: had the instruction not required verifying the target set, an unverified run against the planning repository would have found no issues and could have been misreported as a backlog problem rather than a reviewer error.

---

## 2. The exact target set, verified

Pagination was followed by reading the `Link` header's `rel="next"` chain to exhaustion, not by assuming a page count.

| | |
|---|---|
| Pages fetched | 3, chain ended because page 3 returned no `next` link |
| Entries per page | 100, 100, 89 |
| Total entries | 289 |
| Pull requests excluded, by the `pull_request` key | 3: **#289, #2, #1** |
| **Issues** | **286** |
| Number range | **#3 to #288 inclusive** |
| Gaps within #3-#288 | **none** |
| Issues outside #3-#288 | **none** |
| States | all 286 open |
| Bodies non-empty | 286 of 286 |

The target set matches the recorded expectation of 286 issues numbered #3 to #288 exactly. The three excluded pull requests are what make the highest number 289 while the issue count is 286.

Retrieval was unauthenticated against a core rate limit of 60 per hour with 60 remaining at the start; four requests were used.

---

## 3. Shared boilerplate: reviewed once, one variant each

All 286 bodies share **exactly one heading shape**, with seven sections:

> Task and intended result | Scope and boundaries | Sources and applicable rules | Prerequisites | Acceptance and verification | Common completion checklist | Execution record

Hashing each section's content across all 286 issues gives the split between task-specific and shared content:

| Section | Distinct values across 286 issues |
|---|---|
| Task and intended result | 286 |
| Scope and boundaries | 286 |
| Sources and applicable rules | 286 |
| Prerequisites | 286 |
| Acceptance and verification | 286 |
| **Common completion checklist** | **1** |
| **Execution record** | **1** |

So there is one variant of each boilerplate section, not several. Both were read once, in full, and are not re-read anywhere in this comparison.

**The boilerplate carries the Guide's verification practices into every issue.** Its checklist requires that "required checks actually executed and are not missing, filtered out, skipped or relabeled as success," which is Guide §8.1.1's false-green rule; that "applicable tests use independent expected results and demonstrate detection of relevant wrong behavior; behavioral red-green evidence or an appropriate alternative is recorded, with failures, retry outcomes, unrun checks and justified inapplicability stated honestly," which is §8.1.1's first and last bullets; and that "no implicit software installation or unapproved external effects occurred," which matches Guide line 962. The Execution record asks for "what was proved and why the relevant wrong behavior would be caught." This is a positive result for the verification area completed in iteration 34: the practices are not confined to the Guide, they are built into the unit of work.

---

## 4. The comparison, and how its zero result was made trustworthy

All 286 Roadmap leaves were extracted mechanically and compared against their linked issues on nine dimensions: title text, title leaf-ID, prerequisites against the leaf's `Depends`, Guide references against the issue's Sources, scope coverage, acceptance coverage, link repository, link text against link target, and coverage in both directions.

**The inventory is empty. No difference was reported.**

A zero result from a mechanical check is worth nothing unless the check can detect a difference, which is the Guide's own rule at §8.1.1 and the standard this review has been holding others to. Three things were done before accepting it.

**Sensitivity proof.** Each of the nine checks was run against an injected difference. All nine fired:

```
DETECTS  title-mismatch              DETECTS  scope-low-overlap
DETECTS  title-missing-leaf-id       DETECTS  done-low-overlap
DETECTS  prereq-missing-in-issue     DETECTS  issue-without-leaf
DETECTS  guide-ref-missing-in-issue  DETECTS  link-text-target-mismatch
DETECTS  linked-issue-missing
```

**Cross-pair control.** Leaf 1.1.1's scope was compared against issue #153, a deliberately wrong pairing. It scored 0.231 against a 0.5 threshold and was correctly flagged. Genuine pairs score between 0.900 and 3.333, so the separation is wide and the threshold is not doing delicate work.

**Verbatim containment, which makes the threshold moot.** The decisive check is exact rather than fuzzy:

> **Every one of the 286 leaves has its `Scope` text reproduced verbatim in its linked issue body. Every one has its `Done` text reproduced verbatim.** 286 of 286 on both.

That is stronger than any overlap score. The comparison's zero result rests on exact string containment across the whole set, with the fuzzy checks serving only as a second net.

**One reviewer error was caught and corrected during the run.** The first execution reported 286 title mismatches. Inspection of the first twelve showed the issues title as `[1.1.1] Inspect the server checkout…` while the leaf title omits the bracketed ID, and the normalization stripped a bare `1.1.1 ` prefix but not a bracketed one. The normalization was corrected and the check re-run. The reported difference was in the comparison, not in the data.

---

## 5. What the issues add beyond their leaves

The issues are a strict superset of their leaves. Beyond reproducing `Scope` and `Done` verbatim, each carries ten consistently labelled fields, every one present in all 286:

| Section | Labelled fields |
|---|---|
| Task and intended result | Roadmap task; Parent capability group; Deliverable |
| Scope and boundaries | Parent context, marked "not an instruction to implement every sibling"; Not included |
| Sources and applicable rules | Roadmap task and planning baseline; Applicable Guide sections and controlling standards; Supporting research or examples |
| Prerequisites | Environment or other constraints |
| Acceptance and verification | Verification approach and independently expected results |

Body lengths run from 7,053 to 11,891 bytes, median 8,248.

Two of these fields matter for the review's own findings. "Not included" gives every task an explicit exclusion list, which is the mechanism that keeps an issue from absorbing its siblings. And "Verification approach and independently expected results", present in all 286, is where the independence requirement recorded under **F-12** would be satisfied or violated per task; it is the per-issue counterpart to Guide line 901.

---

## 6. Difference inventory for batches 19 to 26

| Dimension | Result |
|---|---|
| Leaves extracted | 286, no duplicate IDs |
| Leaves with a GitHub issue link | 286 |
| Links pointing at `DGIWG-P507/glaux-server` | 286 |
| Link text matching link target | 286 |
| Linked issues that exist | 286 |
| Issues with no corresponding leaf | 0 |
| Issues linked from more than one leaf | 0 |
| Title mismatches after ID normalization | 0 |
| Leaf `Depends` entries absent from the issue's Prerequisites | 0 |
| Leaf Guide references absent from the issue's Sources | 0 |
| Leaf `Scope` not verbatim in the issue | 0 |
| Leaf `Done` not verbatim in the issue | 0 |

**The difference inventory that batches 19 to 26 were to consume is empty.** Those batches were sized on the assumption that a difference inventory would drive them, and the queue's recorded sizing uncertainty said "a large difference inventory would make batches 19-26 heavier and could raise the total." The opposite occurred. Because leaf content is reproduced verbatim, matching content genuinely need not be reviewed twice, and those batches reduce to reviewing the per-issue content that has no Roadmap counterpart: the ten labelled fields in §5, principally the verification approach and the exclusion list.

This does not make batches 19 to 26 empty. It means their subject is the issues' own added content, not reconciliation with the Roadmap.

---

## 7. What this batch does not establish

The instruction was explicit that a comparison is not a complete issue review, and this report records only the comparison.

- **Read count unchanged at 18.** The 286 bodies were fetched and compared mechanically. Eighteen issue bodies are documented as read by a reviewer, from iterations 12 and 15, and that count does not change here. Fetching a body is not reading it.
- **No judgement on task-specific quality.** Whether each issue's verification approach is adequate, whether its exclusions are correct, and whether its acceptance checks are meaningful are questions for batches 19 to 26.
- **No dependency-graph analysis.** Prerequisites were checked for presence against the leaf's `Depends`, not resolved into a graph. Cycles, ordering defects, scope coverage and sizing belong to batch 18, which remains untouched.
- **No implementation inference.** All 286 issues are open with an Execution record reading "Not started", which is consistent with the Roadmap's statement that implementation has not begun. Nothing here verifies that.

---

## 8. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Saved environment assumption | **Wrong about the repository.** Issues live in `DGIWG-P507/glaux-server`, not the planning repository. Third saved-state defect found | §1 |
| Target set | **Verified exactly.** 286 issues, #3 to #288, no gaps, 3 pull requests excluded, pagination followed to exhaustion | §2 |
| Snapshot | **Preserved** as `40-batch17-issue-snapshot.json` with full provenance | §2 |
| Boilerplate | Reviewed once; **one variant each**, identical across all 286; carries the Guide's verification practices into every issue | §3 |
| Comparison | **Difference inventory empty**, on nine dimensions | §4 |
| Trustworthiness of the zero | Nine checks proven sensitive; cross-pair control flags at 0.231; **verbatim containment 286 of 286 on both Scope and Done** | §4 |
| Reviewer error caught | An initial 286 title mismatches were a normalization defect in the comparison, corrected and re-run | §4 |
| Issue content beyond leaves | Ten labelled fields, all present in all 286; these are what batches 19-26 must review | §5 |
| Batches 19-26 | Lighter than the queue's sizing uncertainty allowed for, not heavier. Their subject is the added content, not reconciliation | §6 |
| Read count | **Unchanged at 18.** Fetching is not reading | §7 |
| New findings | None. No new finding number and no new follow-up question | |

**Remaining: 10 batches of 27.** Next selected batch is **batch 18 of 27**, the whole-backlog dependency graph, coverage and sizing analysis.

---

## 9. Statement of limits

This iteration made four unauthenticated GitHub API requests and read the Roadmap's 286 leaf entries. It compared, mechanically and completely, every leaf against its linked issue; it did not read the 286 issue bodies as a reviewer, and the documented read count stays at 18.

The comparison covers the nine dimensions listed in §6. It does not cover the semantic adequacy of any issue, the dependency graph, scope coverage against the Guide capability set, or sizing, all of which belong to batch 18 and later.

The verbatim-containment result is exact after normalizing whitespace, smart quotes, backticks and asterisks, and lower-casing. The fuzzy overlap checks use a 0.5 threshold whose separation from genuine pairs is recorded in §4 rather than assumed.

The snapshot was retrieved on September 20, 2026. GitHub issues are mutable, so later batches reusing it should note that it is a point-in-time copy; its provenance block records the retrieval conditions.

Evidence reports 31 through 39 are preserved unchanged.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. Nothing was written to the implementation repository. `review_complete` remains `false`.

# Pass 3c iteration 36 - queue batch 18, dependency graph, coverage and sizing

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed` for batch 18, preceded by an instruction to clarify the later phase-batch reuse rule.
**Sources:** The preserved snapshot [40-batch17-issue-snapshot.json](40-batch17-issue-snapshot.json) and the Roadmap. No issue was re-fetched and no network request was made.

---

## 1. The phase-batch reuse rule, corrected before any analysis

Iteration 35 wrote that batches 19 to 26 "reduce to reviewing the per-issue content that has no Roadmap counterpart." **That was wrong and is withdrawn.**

Matching Roadmap content is reused as **one shared copy**, not excluded from semantic review. The leaf `Scope` and `Done` text *is* each task's scope and acceptance criteria. Batch 17 established only that it was copied faithfully into the issue. **Mechanical equality does not establish adequacy.** Whether a task's scope is right, and whether its acceptance criteria would actually catch the behavior the Guide requires, are questions batch 17 did not touch and cannot answer.

What batch 17 genuinely removed is narrower: the need to read the same text 286 times, and the need to reconcile each issue against its leaf. What it did not remove is the need to judge the content.

The corrected rule, now written into all eight phase batches and into the queue's reuse rules:

> Each task is assessed as one unit, with its scope, its acceptance criteria and its added fields **together**, against the Guide and the recorded findings. The added "Verification approach" and "Not included" fields qualify the acceptance text and cannot be judged apart from it. Matching content is assessed once as a shared copy and that assessment is carried to every task sharing it. Any substantive assessment already recorded is reused without repeating it.

The iteration 35 batch history entry and the backlog analysis were corrected in place, preserving what they originally said alongside the correction.

---

## 2. Dependency graph

Built from the Roadmap's three levels, with capability-group references expanded to their member leaves.

| | |
|---|---|
| Phases | 9 |
| Capability groups | 41 |
| Leaves | 286 |
| Resolved edges | 1,676 |
| Roots, having no dependency | **1**, task 1.1.1 |
| Leaves nothing depends on | 1 |
| Isolated leaves | 0 |
| Longest dependency chain | 136, ending at 9.4.3 |

The single root matches the Roadmap's own statement that implementation begins with task 1.1.1 and issue #3. Exactly one leaf has `Depends: none`, and 87 leaves depend on a capability group rather than an individual leaf, which is why the edge count is high relative to the leaf count.

**The problem inventory is empty.** No cycle, no dangling dependency, no self-dependency, no forward dependency where a task depends on one that executes later, and no cross-phase dependency running backwards in time. Phase group counts and subtask counts both reconcile to the phase table exactly, at 41 and 286.

### 2.1 Phase prerequisites are consistent under transitive closure

The phase table's Prerequisites column names proximate prerequisites, not the full closure. Compared naively, two phases look wrong: Phase 5 declares "Phase 2 and Phase 4's shared contracts/codecs" while its leaves also reach phases 1 and 3, and Phase 6 declares "Phases 3-5" while its leaves also reach phases 1 and 2.

Compared against the **transitive closure**, which is the correct test because a phase cannot start until its prerequisites are complete and theirs in turn, all nine phases are consistent:

| Phase | Transitive closure of declared prerequisites | Phases its leaves actually reach |
|---|---|---|
| 5 | 1, 2, 3, 4 | 1, 2, 3, 4 |
| 6 | 1, 2, 3, 4, 5 | 1, 2, 3, 4, 5 |

**Zero phases have leaf dependencies outside their transitive prerequisite closure.**

---

## 3. Sensitivity: the checks can detect what they report as absent

A clean result from a mechanical check is worth nothing unless the check can fail. Each check was run against an injected defect.

```
DETECTS  dangling-dependency     DETECTS  cross-phase-backward
DETECTS  dependency-cycle        DETECTS  self-dependency
DETECTS  forward-dependency      DETECTS  phase-subtask-count
```

Six of six. The transitive-closure check was probed separately: injecting a Phase 5 dependency on Phase 8 is detected, because 8 is outside the closure, while the control of a Phase 5 dependency on Phase 3 is correctly accepted.

The dependency-text parser was validated on six cases covering a bare ID, an en-dash range, `none`, a comma list, a mixed group-and-leaf list, and an ID followed by explanatory prose that must not be read as further dependencies. All six pass. This matters because the Roadmap writes dependencies as prose, and a parser that silently dropped a trailing clause would under-report edges and could hide a cycle.

---

## 4. Three check artifacts, caught by inspection

Sensitive checks also produce false positives, and every apparent problem this batch turned out to be one. All three were in the analysis, not the backlog.

**The two spatial filtering classes appeared uncovered.** A literal search for `basic-spatial-functions` and `basic-spatial-functions-plus` across the 286 leaf texts returned nothing. Inspection resolved it: task 7.3.3 is "Implement the selected spatial predicate and literals", whose scope compiles `s_intersects` to exact PostGIS predicates and validates the complete spatial-plus literal set, and task 7.4.3 is "Complete both selected spatial class checks", covering "basic-spatial and spatial-plus fixtures against actual PostGIS results". The leaves use the short forms. **All six selected Features and CQL2 classes are covered.**

**The phase prerequisite check read the wrong column.** The first parse took the phase table's "Main result" column as Prerequisites, which made every phase from 2 onward look like it had undeclared dependencies. Corrected, and then compared against the transitive closure as §2.1 records.

**The declared-versus-actual comparison was the wrong test.** Even with the right column, comparing actual dependencies against *declared* prerequisites flagged phases 5 and 6. The closure is the correct comparison, and under it nothing is flagged.

Recording these matters because the same discipline cuts both ways. Batch 17 proved its checks sensitive so a zero could be trusted; this batch had to inspect every non-zero before reporting it, because a sensitive check that is asking the wrong question produces confident nonsense.

---

## 5. Scope coverage

| Check | Result |
|---|---|
| Distinct Guide references cited across the 286 leaves | 37 |
| Cited references that are not a real Guide section heading | **0** |
| Guide capability sections, being the numbered subsections of §4 and §6 | 20 |
| Capability sections never cited by any leaf | **0** |
| Selected Features and CQL2 classes covered | **6 of 6**, per §4 |

Every Goal §5 capability area is represented in the leaf set, by matching titles, scopes and acceptance text:

| Goal area | Leaves matching | | Goal area | Leaves matching |
|---|---|---|---|---|
| 5.1 discovery and navigation | 26 | | 5.6 status and availability | 34 |
| 5.2 registration and description | 48 | | 5.7 security and authorization | 74 |
| 5.3 access and exchange | 109 | | 5.8 cross-environment and DDIL | 49 |
| 5.4 streaming and dynamic data | 90 | | 5.9 validation and conformance | 137 |
| 5.5 tasking and control | 60 | | | |

These counts overlap, since one task can serve several areas, so they indicate distribution rather than partition. The shape is what the Goal implies: verification and access are the broadest concerns, and security appears in 74 tasks rather than being confined to a security phase, which matches the cross-cutting treatment recorded in the batch 15 scenario pass.

---

## 6. Sizing

| Measure | Value |
|---|---|
| Leaves per capability group | min 3, median 6, max 17, across 41 groups |
| Groups with 15 or more leaves | 1, namely group 5.5 at 17 |
| Leaf `Scope` text length | median 126, range 76 to 228 |
| Leaf `Done` text length | median 167, range 115 to 507 |
| Resolved dependencies per leaf | median 1, max 53 |
| Leaves with more than 20 resolved dependencies | 25 |

The three outliers were inspected rather than reported as defects.

**Group 5.5, seventeen leaves.** Its children are each narrow and distinct: replace and patch Command intent, replace and patch Feasibility requests, correct public status reports, correct public results, delete resources, delete status and result items, mediate cancellation, cancel feasibility, three filtering tasks, two encoding-interoperability tasks, hold restored command work, detect the lost post-backup interval, publish the workflow, and integrate System deletion with tasking descendants. The group is large because completing tasking mutations has many distinct obligations, not because any child is oversized.

**Task 1.1.4, the longest acceptance text at 507 characters.** It is the CI false-green proof, requiring demonstration of suite discovery and failure propagation with an intentionally failing assertion, a failed service setup and a runner error, and requiring that outcome counts and initial failures be preserved rather than retried to green. The length reflects a detailed requirement that matches Guide §8.1.1, not an oversized task.

**Task 2.6.1, fifty-three dependencies.** Its `Depends` reads "2.1, 2.2, 2.3, 2.4, 2.5", five whole capability groups, and the expansion to their member leaves produces the count. For a task titled "Common and Features prerequisite HTTP tests" that is the correct relationship. All 25 leaves with more than twenty dependencies arise the same way.

**Assessment:** sizing is coherent. No task was found whose scope or acceptance text suggests it could not be completed in one iteration once its dependencies are ready, which is the Roadmap's stated intent. This is a judgement from the scope and acceptance text and the dependency shape; it is not a measured effort estimate, and none is available.

---

## 7. What this batch does not establish

- **It does not assess task adequacy.** Whether each task's scope is right and its acceptance criteria would catch the behavior the Guide requires belongs to batches 19 to 26, under the rule corrected in §1.
- **The read count stays at 18 of 286.** This batch read Roadmap leaf text and the snapshot's structured fields mechanically. No issue body was read by a reviewer.
- **It does not verify the Roadmap against the standards.** Coverage here means every Guide capability section is cited by some task and every Goal area is represented. Whether the Guide itself covers the standards was the subject of earlier areas.
- **Sizing is a judgement, not a measurement.** No effort data exists.

---

## 8. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Phase-batch reuse rule | **Corrected before analysis.** Matching content is one shared copy that still receives semantic review; mechanical equality is not adequacy | §1 |
| Dependency graph | **Clean.** 1,676 edges, one root at 1.1.1, longest chain 136. No cycle, dangling, forward, self or backward cross-phase dependency | §2 |
| Phase prerequisites | **Consistent under transitive closure**, all nine phases | §2.1 |
| Counts | 9 phases, 41 groups, 286 leaves, reconciling to the phase table exactly | §2 |
| Sensitivity | **Six of six checks detect injected defects**; the closure check probed separately with a control; the dependency parser validated on six cases | §3 |
| Check artifacts | **Three apparent problems, all in the analysis, none in the backlog.** Recorded rather than quietly fixed | §4 |
| Scope coverage | **Complete.** Zero bogus Guide references, zero uncited capability sections, all six filtering classes, all nine Goal areas represented | §5 |
| Sizing | **Coherent.** Three outliers inspected and explained; no task appears oversized | §6 |
| Read count | Unchanged at 18 of 286 | §7 |
| New findings | None. No new finding number and no new follow-up question | |

**Remaining: 9 batches of 27.** Next selected batch is **batch 19 of 27**, the first phase batch, under the reuse rule corrected in §1.

---

## 9. Statement of limits

This iteration made no network request and re-fetched nothing. It reused the preserved snapshot from batch 17 and read the Roadmap's phase table, 41 group entries and 286 leaf entries.

The graph is built from the Roadmap's dependency prose. The parser that reads it was validated on six shapes including the trailing-prose case, and the checks built on it were each proven able to detect an injected defect, but a dependency the Roadmap does not state cannot appear in the graph. Group references are expanded to all member leaves, which is the strongest reading; a narrower intended meaning would reduce edge counts without creating a cycle.

Coverage is established by citation and by term matching against titles, scopes and acceptance text. §4 records that term matching produced one false absence, resolved by inspection. A capability could in principle be cited by a task that does not actually deliver it; that is an adequacy question for batches 19 to 26.

Sizing is a judgement from scope text, acceptance text and dependency shape. No effort measurement exists and none is claimed.

Evidence reports 31 through 41 are preserved unchanged, except that the iteration 35 batch history entry and the backlog analysis carry the §1 correction appended in place.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. `review_complete` remains `false`.

# Glaux Server implementation review — Phase 1

**Status: open. Folder set up; no review step has started.** Set up 27 September 2026.

[Findings](findings.md) · [Evidence](evidence/) · [Scoping observations](evidence/00-scoping-observations.md) · [Planning documents](../../README.md) · [Completed pre-implementation review](../../Review/README.md)

## What this review is

The [pre-implementation review](../../Review/README.md) checked the *plan*. It is complete and stays as a historical record. This review checks the *implementation*: the code, tests, CI and delivery workflow built so far for Roadmap Phase 1 (issues #3–#26).

The project lead intends to hold a review like this when each phase completes, one folder per phase (`../Phase-2/` and so on). That is an intention, not a gate. Whether a later phase waits for a review is the project lead's decision at the time.

This Phase 1 review opens a little before the phase ends. The delivery-pipeline step is eligible first because CI is close to its time limit (see [scoping observations §2](evidence/00-scoping-observations.md#2-ci-duration-against-its-limit)). It still needs its own `proceed`. The other steps wait for the Phase 1 work they examine.

## Authority and limits

- On 27 September 2026 the project lead authorised setting up this folder. **Each review step needs its own `proceed`.** Setting up the folder does not start any step.
- The review reads and reports. It does not change server code, GitHub issues, repository settings, the Goal, the Guide or the Roadmap, and it installs no software. Recommendations go to the project lead. Anything adopted goes through the existing change process and is recorded in the [action list](../../Review/action-list.md), not here.
- **Implementation keeps going in parallel.** Another assistant may be working on an issue at the same time (#24 was in progress when this folder was set up). The review is read-only on the server repository. It does not edit the action list, Roadmap or open implementation branches, and it records the exact commit it looked at.
- An assistant's review is not independent human review, whichever model does it. Each evidence file records who or what reviewed, which model (if known) and whether it had fresh context.

## Proposed review steps

These are proposed steps, not an approved work queue. Each one starts only on its own `proceed`.

| # | Step | Question it answers | Eligible | Main evidence it needs |
|---|---|---|---|---|
| 1 | Delivery pipeline | Can the current way of building, checking and handing off each issue keep working for the remaining ~280 issues? Covers CI time and structure, working without a compiler, and the size of the per-issue records. | Now | CI history, PR records, workflow file, action list |
| 2 | Test-source audit | For a sample of CSAPI-facing tests, can each expected answer be traced to a sentence in the standard, its abstract tests, or OGC's schemas and examples? Presented side by side in plain English so the project lead can check them. | After #24–#25 merge | Standard text in the server corpus, Guide, test code |
| 3 | Phase 1 architecture | Will the foundations (the System-specific write path, permission checks, HTTP boundary, one proof program per issue) scale to Phase 2's other resource types without copying everything? | After #26 closes | Server code at the Phase 1 exit commit |
| 4 | Test strength and security scanning | Do the tests catch deliberately injected bugs, and does standard security scanning (CodeQL, dependency alerts) find anything? This builds on what already exists: the hand-picked fault controls and the bounded schema-parser mutation campaign in CI, and Guide §8.1.1, which already names `cargo-mutants`. | After #26; running tools needs approval | A proposal first. No tool is added without project-lead approval. |
| 5 | Comparison with OpenSensorHub | Do Glaux and OpenSensorHub answer the same System requests the same way? Each difference is judged against the standard's text. | After #25 | A pinned OSH instance in an approved disposable environment |

Each step has a fixed question and stops once that question is answered. It does not widen into a whole-project re-review.

### Where independent evidence comes from

The assistants that write the code also write its tests, and assistant reviewers can share their blind spots. This review therefore prefers evidence that no AI wrote, in this order:

1. **The standard's own human-written material:** its text, abstract tests (Annex A), schemas and examples.
2. **Mature peer software built mainly by people,** chiefly OpenSensorHub. Its codebase dates from 2014, and its main contributor appears to be the CSAPI Part 1 editor.
3. **Human expert time,** kept for the highest-risk points: step 3, the authentication and permission code, and the places where the Guide records Glaux's own reading of an unclear or conflicting part of the standard.
4. **Tools:** mutation testing and security scanners.
5. **A different AI model with fresh context.** Cheapest and weakest; used to find issues, not to settle them.

**Not used as an oracle** (a trusted source of expected answers): the unofficial Botts TEAM Engine suite, whose test logic the project lead reports is AI-generated ([scoping observations §7](evidence/00-scoping-observations.md#7-external-conformance-suites-and-peer-implementations)).

## Questions for the project lead

These do not block the folder or step 1.

1. Is the OS4CSAPI TypeScript client, named in Guide §8.1 as an external-client check, mostly AI-written? If so, it has the same limitation as the Botts suite.
2. Is there a person available for step 3 or the security code? For example, someone at Riverside Research (a submitting organisation of CSAPI Part 1), someone from the OpenSensorHub team, or an OGC code-sprint contact.
3. Should Phase 2 wait for step 3's result?

## How to record work

- **One evidence file per completed step iteration** in `evidence/`, numbered in order (`01-delivery-pipeline.md`, …). Start each with: step, date, reviewer and model (if known), server and planning commits examined, and what was and was not checked.
- **Findings go in [findings.md](findings.md)**, in the format shown there. Each finding opens with one plain-English sentence and links to its evidence.
- **Keep this README's status line current.** Don't repeat findings here.
- **Never edit an earlier evidence file** to make it agree with a later one. Add a correction instead.
- **Keep it small.** No machine-state file, cursor or batch queue unless a step actually needs one.
- **Separate review before publishing,** following [AGENTS.md](../../../../../AGENTS.md): a different agent/session inspects the actual diff. The reviewed commit and outcome go in the delivery record (the PR description).
- **Stop after each step** and report to the project lead in plain English.

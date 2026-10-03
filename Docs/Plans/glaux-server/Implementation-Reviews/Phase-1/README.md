# Glaux Server implementation review — Phase 1

**Status: open. Steps 1–3 are complete; steps 4–5 have not started. Step 3 found no blocking architecture defect in its bounded source inspection and added one Low maintenance recommendation, P1-04. P1-01 and P1-03 are adopted; P1-02 and P1-04 remain open. Phase 1 implementation finished on 29 September 2026 (#26 closed), and all four client studies are accepted. Next is step 4's proposal, on its own `proceed`; no new tools or campaigns are authorized by this status. The remaining steps must complete before the project lead closes review gate 1.** Set up 27 September 2026; step 1 completed the same day; steps 2–3 completed 3 October 2026. Neither source-inspection step executed tests or established full correctness/conformance.

[Findings](findings.md) · [Architecture review](evidence/03-phase-1-architecture.md) · [Test-source audit](evidence/02-test-source-audit.md) · [Evidence](evidence/) · [Scoping observations](evidence/00-scoping-observations.md) · [Planning documents](../../README.md) · [Completed pre-implementation review](../../Review/README.md)

## What this review is

The [pre-implementation review](../../Review/README.md) checked the *plan*. It is complete and stays as a historical record. This review checks the *implementation*: the code, tests, CI and delivery workflow built so far for Roadmap Phase 1 (issues #3–#26).

Each phase gets its own review folder (`../Phase-2/` and so on).

**Review gates.** On 27 September 2026 the project lead made reviews gate implementation, so that nobody has to remember when to run them.
- There are fifteen `review-gate` issues, defined in [Roadmap §5.4](../../glaux-server-roadmap.md#54-review-gates): one at every phase end, a Phase 2 checkpoint, a Phase 5 command-safety checkpoint, health checks inside Phases 2–5, and a final review.
- Each gate pauses implementation until the project lead closes it.
- Steps 2–5 of this review must be complete before gate 1 closes. Steps 2 and 5 were eligible earlier under the step table, but since 29 September 2026 no step starts before the client studies ([below](#client-studies-before-steps-25)) are accepted.
- This folder, with its charter, findings format and numbered evidence, is the template for later phase folders.

This Phase 1 review opens a little before the phase ends. The delivery-pipeline step is eligible first because CI is close to its time limit (see [scoping observations §2](evidence/00-scoping-observations.md#2-ci-duration-against-its-limit)). It still needs its own `proceed`. The other steps wait for the Phase 1 work they examine.

## Authority and limits

- On 27 September 2026 the project lead authorised setting up this folder. **Each review step needs its own `proceed`.** Setting up the folder does not start any step.
- The review reads and reports. It does not change server code, GitHub issues, repository settings, the Goal, the Guide or the Roadmap, and it installs no software. Recommendations go to the project lead. Anything adopted goes through the existing change process and is recorded in the [action list](../../Review/action-list.md), not here.
- **Implementation keeps going in parallel until review gate 1** (Roadmap §5.4), which pauses Phase 2 until the project lead closes it. Another assistant may be working on an issue at the same time (#24 was in progress when this folder was set up). The review is read-only on the server repository. It does not edit the action list, Roadmap or open implementation branches, and it records the exact commit it looked at.
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
2. **Mature peer software with a long human-developed history,** chiefly OpenSensorHub. Its codebase dates from 2014, and its main contributor appears to be the CSAPI Part 1 editor. Client-side evidence comes from the studies listed under [Client studies before steps 2–5](#client-studies-before-steps-25).
3. **Human expert time,** kept for the highest-risk points: step 3, the authentication and permission code, and the places where the Guide records Glaux's own reading of an unclear or conflicting part of the standard. No person is available for this review (question 2 below).
4. **Tools:** mutation testing and security scanners.
5. **A different AI model with fresh context.** Cheapest and weakest; used to find issues, not to settle them.

**Not used as an oracle** (a trusted source of expected answers): the unofficial Botts TEAM Engine suite, whose test logic the project lead reports is AI-generated ([scoping observations §7](evidence/00-scoping-observations.md#7-external-conformance-suites-and-peer-implementations)), and the OS4CSAPI TypeScript client, which the project lead reports is mostly AI-written (question 1 below).

## Client studies before steps 2–5

On 29 September 2026 the project lead decided that steps 2–5 wait until four research studies of CSAPI clients are complete and accepted, because their findings may change what those steps check. They are registered as [IDR-SRV-063 to IDR-SRV-066](../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md#idr-srv-063-to-idr-srv-066-csapi-client-studies) in the overall research plan. Each uses the existing research-plan and report templates, runs on its own `proceed`, and produces its own report, in this proposed order (the project lead listed Aleph before cs-client-ts; cs-client-ts comes first because Aleph depends on it):

1. [OSH Viewer and OSH JS Toolkit](../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/idr-srv-063-osh-viewer-and-osh-js-client-study.md) (IDR-SRV-063).
2. [OSCAR Viewer](../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/idr-srv-064-oscar-viewer-client-study.md) (IDR-SRV-064).
3. [cs-client-ts](../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/idr-srv-065-cs-client-ts-client-library-study.md) (IDR-SRV-065).
4. [Aleph (Alephex)](../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/idr-srv-066-aleph-connected-systems-ui-client-study.md) (IDR-SRV-066).

The project lead reports that the first two are the most human-written CSAPI clients, and that the last two were written by a senior developer with AI assistance. That developer also wrote the Connected Systems Go server, so those studies assess how independent their evidence is.

Once all four reports are accepted, the next `proceed` resumes this review. Their findings feed step 2 (where test answers come from), step 5 (the OpenSensorHub comparison) and the client checks that later gates run. They do not change the standard's authority.

**Completed 3 October 2026:** the project lead's merge of [PR #111](https://github.com/DGIWG-P507/glaux/pull/111), `f74eb659bab70b394b1027d1d43b3066561762de`, accepted IDR-SRV-066, completing the four-study prerequisite. The following `proceed` authorized step 2. Its [evidence](evidence/02-test-source-audit.md) reuses the accepted studies without treating a client application's preferences as server requirements. The project lead then merged [PR #112](https://github.com/DGIWG-P507/glaux/pull/112) and authorized step 3, now recorded in the [architecture review](evidence/03-phase-1-architecture.md). The next bounded step is **step 4's test-strength/security-scanning proposal**, on a new `proceed`; Phase 2 remains blocked by [gate 1 (#339)](https://github.com/DGIWG-P507/glaux-server/issues/339).

## Questions for the project lead

These do not block the folder or step 1.

1. ~~Is the OS4CSAPI TypeScript client, named in Guide §8.1 as an external-client check, mostly AI-written?~~ Answered 29 September 2026: yes, mostly AI-written. It has the same limitation as the Botts suite: its expected behaviour is not independent evidence. The project lead named the OSH Viewer and OSCAR Viewer as the most human-written CSAPI clients, and Aleph and cs-client-ts as written by a senior developer with AI assistance. Those studies are now complete and accepted ([above](#client-studies-before-steps-25)).
   - *Proposed consequence, not separately confirmed by the project lead:* any change to Guide §8.1's use of the OS4CSAPI client waits for those studies and a separate decision.
2. ~~Is there a person available for step 3 or the security code? For example, someone at Riverside Research (a submitting organisation of CSAPI Part 1), someone from the OpenSensorHub team, or an OGC code-sprint contact.~~ Answered 29 September 2026: no, no person is available.
   - *Proposed consequences, not separately confirmed by the project lead:*
     - Step 3 and the authentication and permission code rely on sources 1, 2, 4 and 5 above. How much weight step 4's tool checks carry is decided in step 4's own proposal.
     - Each affected evidence file states that no human expert reviewed that area.
     - The project lead decides whether to close gate 1 with that limitation stated.
     - #25 and #26 were implemented by Claude (Anthropic), and [CONTRIBUTING](https://github.com/DGIWG-P507/glaux-server/blob/main/CONTRIBUTING.md#changing-the-implementing-assistant)'s cross-provider rule lets a person review them instead of another provider. With no person available, if gate 1's review runs through Anthropic, an assistant from a different provider must review those tasks, or the review states the limitation.
3. ~~Should Phase 2 wait for step 3's result?~~ Answered 27 September 2026: yes. Review gate 1 pauses Phase 2 until the project lead closes it.

## How to record work

- **One evidence file per completed step iteration** in `evidence/`, numbered in order (`01-delivery-pipeline.md`, …). Start each with: step, date, reviewer and model (if known), server and planning commits examined, and what was and was not checked.
- **Findings go in [findings.md](findings.md)**, in the format shown there. Each finding opens with one plain-English sentence and links to its evidence.
- **Keep this README's status line current.** Don't repeat findings here.
- **Record who built and reviewed the phase.** Each full review lists which assistant, with provider and model where the records show it, implemented and reviewed each task in the phase. It notes anything learned from switching between assistants (see server `CONTRIBUTING.md`, [Changing the implementing assistant](https://github.com/DGIWG-P507/glaux-server/blob/main/CONTRIBUTING.md#changing-the-implementing-assistant)). If the gate reviewer's provider also implemented any task in the phase, the review confirms those tasks got a cross-provider review. If they did not, it states the limitation, and the project lead decides whether to close the gate anyway.
- **Never edit an earlier evidence file** to make it agree with a later one. Add a correction instead.
- **Keep it small.** No machine-state file, cursor or batch queue unless a step actually needs one.
- **Separate review before publishing,** following [AGENTS.md](../../../../../AGENTS.md): a different agent/session inspects the actual diff. The reviewed commit and outcome go in the delivery record (the PR description).
- **Stop after each step** and report to the project lead in plain English.

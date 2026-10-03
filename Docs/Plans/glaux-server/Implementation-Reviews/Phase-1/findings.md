# Phase 1 implementation review — findings

[Review start page](README.md) · [Evidence](evidence/)

Steps 1 (delivery pipeline) and 2 (test-source audit) are complete. The project lead adopted P1-01 and P1-03 on 27 September 2026; P1-02 remains open. Step 2 adds no finding: the [bounded source-to-test sample](evidence/02-test-source-audit.md) supports its selected expected answers, while distinguishing published rules, the selected transaction draft and Glaux's Phase 1 choices. It neither executed tests nor establishes whole-suite correctness or conformance. Steps 3–5 have not started. The [scoping observations](evidence/00-scoping-observations.md) are starting points for the steps to check, not findings.

| ID | Title | Severity | Status |
|---|---|---|---|
| [P1-01](#p1-01--ci-is-likely-to-reach-its-20-minute-limit-within-the-next-few-issues) | CI is likely to reach its 20-minute limit within the next few issues | High | Adopted |
| [P1-02](#p1-02--the-implementing-assistant-uses-ci-as-its-compiler) | The implementing assistant uses CI as its compiler | Low | Open |
| [P1-03](#p1-03--each-delivery-is-written-up-several-times-and-the-file-read-first-keeps-growing) | Each delivery is written up several times, and the file read first keeps growing | Medium | Adopted |

### P1-01 — CI is likely to reach its 20-minute limit within the next few issues

**In plain English:** The automatic checks now take about 18 minutes, and each finished issue adds roughly one more minute. The job is set to stop at 20 minutes. Probably somewhere between #25 and #28, and possibly as early as #24, a check will fail for lack of time rather than because anything is wrong. Nothing can merge until someone changes the CI, and no planned task owns that change.

- **Step:** 1, [evidence §1](evidence/01-delivery-pipeline.md#1-ci-time-and-structure).
- **Examined:** server `d0ef755`, plus all 122 workflow runs to #24's first two runs.
- **What was seen:**
  - Green PR runs grew from 0.23 minutes (task 1.1.2) to 17.80 minutes (task 1.4.6). The least-squares slope is 0.79 minutes per task overall and 1.39 over the last nine (1.3.3 → 1.4.6).
  - `timeout-minutes: 20` is set on the single required job. Individual suites also have 180- and 210-second limits.
  - The drivers are:
    - the nine fixed false-green controls recompile and rerun the growing suites (247 seconds, 25%);
    - every earlier per-task proof and fault control reruns on every PR;
    - no build cache;
    - 39 step entries run in sequence in one job.
  - Task 1.1.4 (#6), which owned CI, is closed. No later Roadmap task owns CI structure.
- **Why it matters:** once runs exceed the limit, every PR fails its required check, so implementation stops. Raising the limit alone keeps feedback slowing, severe after about 20 more issues and hours by the end of the Roadmap. Time can't be recovered by quietly dropping tests; it takes structure, or an explicit recorded decision on which checks count as "affected".
- **Kind:** project choice/recommendation. No standards obligation is involved.
- **Severity:** High.
- **Suggested owner:** project-lead decision on a small bounded CI task. [Evidence §5](evidence/01-delivery-pipeline.md#5-options-for-the-project-lead), options A–D, lists the options and their conditions:
  - A. Raise the limit as a stopgap.
  - B. Cache dependencies and build output.
  - C. Run parallel jobs behind the existing required `Rust bootstrap` result.
  - D. Record an explicit "affected checks" rule for older per-task proofs.

  Any split must keep one unconditional required result that fails on skipped work.
- **Status:** Adopted on 27 September 2026 as Roadmap task 1.1.5 (v1.38), using options A and C. Option B (caching) and option D were not taken. Recorded in the [action list Current state](../../Review/action-list.md#current-state).

### P1-02 — The implementing assistant uses CI as its compiler

**In plain English:** By project rule, the AI writing the code does not use Rust tools on the project lead's laptop. It finds formatting and compile mistakes only by pushing to GitHub and waiting. This costs several extra round trips per issue but has not slowed delivery much, so it matters less than P1-01.

- **Step:** 1, [evidence §2](evidence/01-delivery-pipeline.md#2-working-without-a-compiler).
- **Examined:** 101 PR runs from task 1.1.2 to #24's branch.
- **What was seen:**
  - 70 of 101 PR runs failed:
    - 31 stopped at formatting;
    - 8 at lint/compile;
    - 2 at dependency fetch;
    - 9 at deliberate stop steps;
    - 20 at other steps.
  - Formatting stops took 11–21 seconds, and lint/compile stops 0.4–1.2 minutes.
  - Most branches went from first run to green in 0.1–1.3 hours.
  - A file hash in the PR #98 record suggests the assistant works against the project lead's local checkout.
- **Why it matters:** each round trip spends the assistant's session and adds narrative to the records (see P1-03). CI remains the authoritative evidence either way.
- **Kind:** recommendation.
- **Severity:** Low.
- **Suggested owner:** project-lead decision, [evidence §5](evidence/01-delivery-pipeline.md#5-options-for-the-project-lead) option E. Because the laptop is excluded, this would mean giving the assistant a different working environment with a pinned toolchain. That is a larger change than it sounds, and not worth making for this finding alone.
- **Status:** Open.

### P1-03 — Each delivery is written up several times, and the file read first keeps growing

**In plain English:** Each finished issue is described in the server PR, the GitHub issue, the action list, a separate planning PR with its own review, and three places in the Roadmap. The action list is the file every new AI session is told to read first. It grows by about 4 KB per issue and would pass 1 MB before the project ends, so sessions will either spend much of their memory on history or skim and miss things.

- **Step:** 1, [evidence §3](evidence/01-delivery-pipeline.md#3-per-issue-records).
- **Examined:** planning `98afc78`; the records for #22 (server PR #330, issue #22, action-list section, planning PR #98); planning PRs #80–#100.
- **What was seen:**
  - The action list has grown by about 3.85 KB per task since the first handoff and is now 195,364 bytes. Together with the Roadmap that is about 459 KB, which planning `AGENTS.md` tells every session to read.
  - There were 21 planning handoff PRs, each separately reviewed, for 20 tasks.
  - Server `CONTRIBUTING.md` already names the issue's execution record as the place that "carries the result".
  - Roadmap §8 says to record execution evidence in the linked GitHub issue and to update the Roadmap's completion status when a group or phase completes. Current practice updates three Roadmap locations after every issue.
- **Why it matters:** the duplication costs a PR and a review per issue. The growing file weakens the handoff it exists to provide, especially in the later phases when there are ~281 more tasks.
- **Kind:** recommendation about working practice.
- **Severity:** Medium.
- **Suggested owner:** project-lead decision, [evidence §5](evidence/01-delivery-pipeline.md#5-options-for-the-project-lead) option F. That would mean recording each delivery once (issue record plus server PR), keeping the action list to current state and decisions with historical handoffs moved unchanged to an archive, and updating the Roadmap status at group or phase completion. Adopting it changes planning `AGENTS.md`.
- **Status:** Adopted on 27 September 2026. Recorded in the [action list Current state](../../Review/action-list.md#current-state), planning `AGENTS.md`, Roadmap v1.38 §8 and server `CONTRIBUTING.md` (via task 1.1.5's server PR #334).
  - The historical handoffs are frozen in place rather than moved to an archive, because Roadmap, issue and PR links point into them.
  - As a result the action list stops growing but stays about 198 KB. Sessions are pointed to its Current state section only.

## Format

Number findings `P1-01`, `P1-02`, … and never reuse a number. If a finding is withdrawn, keep it and mark it withdrawn with the reason.

```markdown
### P1-NN — Short title

**In plain English:** One or two sentences a non-developer can act on.

- **Step:** which review step found it, with a link to its evidence file.
- **Examined:** server commit (and planning commit if relevant).
- **What was seen:** the specific evidence, with file/line or run links.
- **Why it matters:** the consequence if left alone.
- **Severity:** High (unsafe or likely to stop delivery) / Medium (worth addressing within the phase) / Low (address when convenient) / Note (information only).
- **Suggested owner:** the existing issue that could carry it, or "project-lead decision".
- **Status:** Open / Adopted (link to the action-list entry) / Not adopted (link to where the project lead's decision is recorded) / Withdrawn (reason).
```

Severity is the reviewer's assessment. Nothing in this file blocks work or becomes a requirement until the project lead adopts it through the existing change process.

A standards obligation, a project choice and a recommendation are different things. Say which one each finding is.

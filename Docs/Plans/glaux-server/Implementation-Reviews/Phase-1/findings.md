# Phase 1 implementation review — findings

[Review start page](README.md) · [Evidence](evidence/)

Step 1 (delivery pipeline) is complete. Steps 2–5 have not started. The [scoping observations](evidence/00-scoping-observations.md) are starting points for the steps to check, not findings.

| ID | Title | Severity | Status |
|---|---|---|---|
| [P1-01](#p1-01--ci-will-reach-its-20-minute-limit-within-about-two-issues) | CI will reach its 20-minute limit within about two issues | High | Open |
| [P1-02](#p1-02--the-implementing-assistant-uses-ci-as-its-compiler) | The implementing assistant uses CI as its compiler | Low | Open |
| [P1-03](#p1-03--each-delivery-is-written-up-several-times-and-the-file-read-first-keeps-growing) | Each delivery is written up several times, and the file read first keeps growing | Medium | Open |

### P1-01 — CI will reach its 20-minute limit within about two issues

**In plain English:** The automatic checks now take about 18 minutes, and each finished issue adds roughly one more minute. The job is set to stop at 20 minutes. Probably at #25 or #26, and possibly sooner, a check will fail for lack of time rather than because anything is wrong, and nothing can merge until someone changes the CI. No planned task owns that change.

- **Step:** 1, [evidence §1](evidence/01-delivery-pipeline.md#1-ci-time-and-structure).
- **Examined:** server `d0ef755`, plus all 122 workflow runs to #24's first two runs.
- **What was seen:**
  - Green PR runs grew from 0.23 minutes (task 1.1.2) to 17.80 minutes (task 1.4.6): 0.79 minutes per task overall, and 1.13 over the last nine.
  - `timeout-minutes: 20` is set on the single required job.
  - The drivers are:
    - every earlier fault control recompiles and reruns on every PR (the largest step alone takes 247 seconds, 25%);
    - no build cache;
    - 39 step entries run in sequence in one job.
  - Task 1.1.4 (#6), which owned CI, is closed. No later Roadmap task owns CI structure.
- **Why it matters:** once runs exceed the limit, every PR fails its required check, so implementation stops. Raising the limit alone keeps feedback slowing. Past about 20 more issues the delay would be severe, and by the end of the Roadmap it would be hours. The Guide forbids dropping required tests to save time, so time has to be recovered by structure.
- **Kind:** project choice/recommendation. No standards obligation is involved.
- **Severity:** High.
- **Suggested owner:** project-lead decision on a small bounded CI task. [Evidence §5](evidence/01-delivery-pipeline.md#5-options-for-the-project-lead), options A–D, lists the options (raise the limit as a stopgap; cache; parallel jobs behind the existing required `Rust bootstrap` result; where older fault controls run) and their conditions. Any split must keep one unconditional required result that fails on skipped work.
- **Status:** Open.

### P1-02 — The implementing assistant uses CI as its compiler

**In plain English:** The AI writing the code has no Rust tools of its own, so it discovers formatting and compile mistakes only by pushing to GitHub and waiting. This costs several extra round trips per issue but has not slowed delivery much, so it matters less than P1-01.

- **Step:** 1, [evidence §2](evidence/01-delivery-pipeline.md#2-working-without-a-compiler).
- **Examined:** 101 PR runs from task 1.1.2 to #24's branch.
- **What was seen:**
  - 70 of 101 PR runs failed. Of those, 33 stopped at formatting and 8 at lint/compile.
  - Formatting stops took 11–21 seconds, and lint/compile stops 0.4–1.2 minutes.
  - Most branches went from first run to green in 0.1–1.3 hours.
  - The PR records describe applying CI's generated rustfmt patch and separating "preparation failures" from behavioral evidence in every issue record.
- **Why it matters:** each round trip spends the assistant's session and adds narrative to the records (see P1-03). CI remains the authoritative evidence either way.
- **Kind:** recommendation.
- **Severity:** Low.
- **Suggested owner:** project-lead decision, [evidence §5](evidence/01-delivery-pipeline.md#5-options-for-the-project-lead) option E. It first needs an answer to where the implementing assistant runs, and whether a pinned toolchain in its own disposable workspace (never the company laptop) would be acceptable.
- **Status:** Open.

### P1-03 — Each delivery is written up several times, and the file read first keeps growing

**In plain English:** Each finished issue is described in the server PR, the GitHub issue, the action list, a separate planning PR with its own review, and three places in the Roadmap. The action list is the file every new AI session is told to read first. It grows by about 4 KB per issue and would pass 1 MB before the project ends, so sessions will either spend much of their memory on history or skim and miss things.

- **Step:** 1, [evidence §3](evidence/01-delivery-pipeline.md#3-per-issue-records).
- **Examined:** planning `98afc78`; the records for #22 (server PR #330, issue #22, action-list section, planning PR #98); planning PRs #80–#100.
- **What was seen:**
  - The action list has grown by about 3.85 KB per task since the first handoff and is now 195,364 bytes. Together with the Roadmap that is about 459 KB, which planning `AGENTS.md` tells every session to read.
  - There were 21 planning handoff PRs, each separately reviewed, for 20 tasks.
  - Server `CONTRIBUTING.md` already names the issue's execution record as the place that "carries the result".
- **Why it matters:** the duplication costs a PR and a review per issue. The growing file weakens the handoff it exists to provide, especially in the later phases when there are ~281 more tasks.
- **Kind:** recommendation about working practice.
- **Severity:** Medium.
- **Suggested owner:** project-lead decision, [evidence §5](evidence/01-delivery-pipeline.md#5-options-for-the-project-lead) option F. That would mean recording each delivery once (issue record plus server PR), keeping the action list to current state and decisions with historical handoffs moved unchanged to an archive, and updating the Roadmap status at group or phase completion. Adopting it changes planning `AGENTS.md`.
- **Status:** Open.

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

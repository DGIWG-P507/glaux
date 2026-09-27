# 01 — Step 1: delivery pipeline

**Question:** Can the current way of building, checking and handing off each issue keep working for the remaining ~281 issues? The step covers CI time and structure, working without a compiler, and the size of the per-issue records.

**Date:** 27 September 2026<br>
**Authorised by:** the project lead's `proceed` for step 1 only.<br>
**Performed by:** Claude Code, running Claude Opus 5.5 (Anthropic). The context was not fresh: this is the same session that made the scoping observations (evidence 00) and set up this folder. A separate reviewer with fresh context, running Claude Fable 5.1, reviewed this file; the outcome is recorded on its delivery PR.<br>
**Examined:**
- Server `DGIWG-P507/glaux-server` main at [`d0ef755`](https://github.com/DGIWG-P507/glaux-server/tree/d0ef755bf4f8a0f6598dc68276ca6730a7abf9e2), after #23.
- All 122 `Build` workflow runs that existed at collection time. The last two were from #24's in-progress branch `task/1.5.1-system-create`. Their timings were read, but their code was not.
- Planning `DGIWG-P507/glaux` main at `98afc78`.

**Method:** workflow runs, jobs, step timings, PRs and issue comments were read through GitHub's REST API. The project lead's stored Git login was used for reading only, to get the higher rate limit. Earlier the same day, the project lead approved in session (no written record) using that login to create PRs, and should confirm that read use is also acceptable. The server tree was inspected in a read-only clone, and planning document sizes are Git blob sizes. Nothing on the server, in CI, in the issues, in the settings or in the action list/Roadmap was changed, and nothing was installed.

## Answer in brief

**Mostly yes, with one hard limit that is close.** The per-issue workflow is disciplined and has kept `main` green, with one exception (§2.4). But:

1. **CI is likely to reach its 20-minute timeout within the next few issues:** around #25–#28, and possibly #24 given runner variation. No open task owns fixing that (§1).
2. **Working without a compiler costs extra CI round trips** on most issues. The cost is real but moderate (§2).
3. **The per-issue handoff records are written in several places.** The one sessions are told to read first grows by about 3.9 KB per issue (§3).

## 1. CI time and structure

### 1.1 Duration of successful PR runs by task

Job duration comes from the job's `started_at`/`completed_at`, so it excludes queue time. Each value is the last green PR run on that task's branch. The #332 follow-up branch (17.13 minutes) and an early 6-step 1.4.1 candidate run (0.77 minutes) are excluded.

| Task | Green PR run (min) | Task | Green PR run (min) |
|---|---:|---|---:|
| 1.1.2 | 0.23 | 1.3.2 | 8.00 |
| 1.1.3 | 1.08 | 1.3.3 | 7.55 |
| 1.1.4 | 2.07 | 1.3.4 | 5.83 |
| 1.2.1 | 2.30 | 1.3.5 | 7.35 |
| 1.2.2 | 2.70 | 1.4.1 | 11.40 |
| 1.2.3 | 4.03 | 1.4.2 | 8.85 |
| 1.2.4 | 4.18 | 1.4.3 | 11.68 |
| 1.2.5 | 4.45 | 1.4.4 | 14.35 |
| 1.2.6 | 6.35 | 1.4.5 | 15.18 |
| 1.3.1 | 7.67 | 1.4.6 | 17.80 |

The least-squares slope is **0.79 minutes per task** across all 20 tasks and **1.39** across the last nine (1.3.3 → 1.4.6). Runs of the same task vary by up to about 3 minutes (1.4.2 took 8.85 and 12.03 minutes on two green heads).

The workflow sets `timeout-minutes: 20`. When a green run would reach it depends on the starting point:

| Starting point | At the overall rate (0.79) | At the recent rate (1.39) |
|---|---|---|
| 1.4.6's last green PR run (17.80 min) | about #26 | about #25 |
| Current `main` (`d0ef755`: 17.13 min on the #332 PR run, 16.67 min on the main push) | about #27–#28 | about #26 |

Runner variation of up to about 3 minutes could bring this forward, even to #24. #24's branch has already added a new step, "Prove System creation through the actual HTTP and transaction boundary".

As an illustration only, not a forecast: continuing at 0.8 minutes per task for 281 more tasks would add about 3.7 hours to every run.

### 1.2 Where the time goes

This is the latest green `main` run ([36324667106](https://github.com/DGIWG-P507/glaux-server/actions/runs/36324667106), `d0ef755`). The step durations sum to 998 seconds (16.6 minutes).

| Step | Seconds | Share |
|---|---:|---:|
| Verify required execution rejects disposable failures | 247 | 25% |
| Prove runtime configuration and live storage readiness | 60 | 6% |
| Verify directional projection assertions detect disposable faults | 59 | 6% |
| Build all three packages | 58 | 6% |
| Prove initial discovery and reject misleading declarations | 52 | 5% |
| Lint Rust and check Python syntax (Clippy) | 49 | 5% |
| Eight further proof/control steps: retry, permission, conditional and atomic writes, database lifecycle, validation faults, key refresh, exact-time database | 24–45 each, 292 total | 29% |
| The remaining 25 step entries, including setup and post-job | 181 | 18% |

Four things drive the growth:

- **The fixed false-green controls get slower as the code grows.** The largest step (247 seconds, 25%) is `test-ci-failures.py`. It holds nine fixed controls added at task 1.1.4, which prove that a failing assertion, missing service, or empty, filtered or skipped suite cannot pass. Each control reruns either the Rust workspace suite (recompiling it in a fresh copy) or the database suite, so its cost tracks the codebase. The separate reviewer found it took 150 seconds at 1.3.1 (run 36010154715) and 235 seconds at 1.4.5 (run 36286855260).
- **Every earlier per-task proof and fault control reruns on every PR.** Nearly every task adds a real-listener or real-database proof, and most add a "disposable fault" control. That control compiles a deliberately broken copy of the source and requires the exact assertion to fail. This is how the project demonstrates that each test can detect its intended mistake (Guide §8.1.1). These controls sit inside the eight 24–45-second steps and the other proof steps, and they stay in the required job permanently.
- **Nothing is cached between runs.** The workflow has no Cargo or build cache, so every run fetches and compiles all dependencies. Clippy, the build and each disposable copy all compile.
- **Everything runs in one job, in sequence.** Of the 39 recorded step entries, many could run independently: formatting and Clippy, unit and fuzz tests, database proofs, listener proofs, the browser smoke and fault controls.

### 1.3 Why it is one job, and what any change must preserve

[`docs/ci.md`](https://github.com/DGIWG-P507/glaux-server/blob/d0ef755bf4f8a0f6598dc68276ca6730a7abf9e2/docs/ci.md#main-branch-rule) records the reasons.

- **The merge rule names one check.** Ruleset 23796335 requires the check `Rust bootstrap`, and "that one job now includes all checks above".
- **GitHub counts skipped checks as passing.** GitHub accepts success, *skipped* or neutral conclusions for required checks, so the required job must be unconditional and must itself reject skipped or empty work.

Any restructure therefore has to keep one required, unconditional result that fails unless every part actually ran and passed. One common pattern is a final job keeping the `Rust bootstrap` name that needs all the other jobs and fails on any result other than success. That leaves the ruleset unchanged.

Three other rules apply. Current practice (rerun everything on every PR) appears stricter than they require, although the Guide defines neither "affected" nor how often CI discovery must be rechecked:

- **Guide §8.1.1** says to "Never silently discard a required test to meet a runtime budget". Its closing paragraph says "Every implementation PR runs applicable fast checks, bounded property/regression cases and affected real-database/HTTP tests". It adds that "A scheduled result belongs to its actual commit and cannot substitute for a missing required PR check", and that "Phase completion reruns affected workflows".
- **Roadmap §8** says scheduled jobs "are not mandatory on every PR and do not replace its required checks".
- **The existing false-green controls** must keep working.

**Two tighter per-suite limits also exist:**
- `scripts/check-execution.py` gives each suite, including its compilation, a 180-second process timeout.
- `scripts/test-ci-failures.py` gives each control 210 seconds.

A single suite could hit one of these before the whole job hits 20 minutes. Per-suite timings were not collected (see §6).

### 1.4 Who owns CI changes

Task 1.1.4 ([#6](https://github.com/DGIWG-P507/glaux-server/issues/6)) established the CI and is closed. A search of the Roadmap found no later task that owns CI structure or runtime. Each task extends the workflow only for its own checks.

## 2. Working without a compiler

### 2.1 Per-task PR runs

These are PR-event runs only, by branch. Each failed run is classified by its first failed step:
- **Format:** the step "Verify formatting and locked dependencies", or its earlier name "Verify formatting and lockfile reproduction". It runs `rustfmt --check` and also fails on Cargo.lock drift.
- **Lint:** Clippy, the first step that compiles, so compile errors also show up here.
- **Deps:** the dependency fetch.
- **Deliberate:** a step designed to stop the run, such as "Prepare task 8 dependency evidence (deliberately not acceptance)".
- **Other:** any other step. These are mostly behavioral or harness failures in the proofs, which is the kind of failure CI exists to catch.

| Branch | PR runs | Failed | Failure kinds |
|---|---:|---:|---|
| 1.1.2 rust-workspace | 6 | 3 | other 2, format 1 |
| 1.1.3 database-harness | 2 | 1 | other 1 |
| 1.1.4 initial-ci | 3 | 1 | other 1 |
| 1.2.1 standards-corpus | 3 | 0 | — |
| 1.2.2 offline-validation | 9 | 8 | deliberate 4, format 3, lint 1 |
| 1.2.3 typed-identities | 4 | 1 | deliberate 1 |
| 1.2.4 exact-numbers | 2 | 1 | deliberate 1 |
| 1.2.5 exact-time | 2 | 1 | format 1 |
| 1.2.6 direction-validation | 3 | 2 | format 2 |
| 1.3.1 system-storage | 5 | 4 | format 2, deliberate 1, other 1 |
| 1.3.2 revisions-artifacts | 4 | 3 | format 1, lint 1, other 1 |
| 1.3.3 atomic-write | 8 | 7 | format 3, other 3, lint 1 |
| 1.3.4 conditional-writes | 5 | 4 | format 2, lint 1, other 1 |
| 1.3.5 scoped-write-retry | 4 | 3 | format 2, other 1 |
| 1.4.1 runtime-health | 6 | 4 | format 2, deliberate 1, deps 1 |
| 1.4.2 http-boundary | 5 | 3 | format 2, other 1 |
| 1.4.3 authentication | 6 | 4 | format 2, deliberate 1, other 1 |
| 1.4.4 trusted-key-refresh | 5 | 4 | deps 1, format 1, lint 1, other 1 |
| 1.4.5 resource-permissions | 7 | 6 | format 3, lint 2, other 1 |
| 1.4.6 discovery-documents | 9 | 8 | other 4, format 3, lint 1 |
| 1.4.6 browser-startup-bound | 1 | 0 | — |
| 1.5.1 system-create (in progress) | 2 | 2 | format 1, other 1 |

**Totals:** 101 PR runs, of which 70 failed and 31 succeeded. The failures break down as 31 format, 8 lint/compile, 2 dependency fetch, 9 deliberate and 20 other.

**Time spent:** 93.6 minutes in failed PR runs and 201.4 minutes in green ones. Public-repository Actions minutes are not billed.

### 2.2 Cause

The PR records confirm that no local toolchain was used. The comments on [PR #330](https://github.com/DGIWG-P507/glaux-server/pull/330) describe:
- applying CI's generated rustfmt patch;
- a compile stop over an unavailable `axum::Json` import;
- a Clippy type-complexity stop.

The project lead's 21 September 2026 choice of GitHub-hosted Linux, with no company-laptop installation, makes CI the implementing assistant's compiler. Server `CONTRIBUTING.md` says "The company laptop is an editing/Git interface, not a required Rust/database host". The project lead reports that the implementing assistant is ChatGPT, and the PR #98 record calls its reviewer a "separate non-author OpenAI agent".

**Where it runs.** PR #98 records the SHA-256 of the project lead's untracked PITON file, `0FABAC0B…6A38`. That file is untracked (not on GitHub) and is present in this local checkout, which is in a OneDrive-synced folder. Hashing it there gives the identical value. This suggests the implementing assistant works against the project lead's local checkout. If so, a local toolchain would mean installing on the company laptop, which is excluded. Any compiler access would need a different working environment. Where the sessions actually run was not otherwise established.

### 2.3 How much it costs

**Moderate.** Formatting failures stopped within 11–21 seconds, and lint/compile failures within 0.4–1.2 minutes. From first PR run to last green run, most branches took 0.1–1.3 hours. Two spans (45 and 31 hours) include waiting between sessions. The time a session spends waiting and repairing is the real cost; it was not measured directly.

A secondary cost: every issue's evidence record now carries a paragraph separating "preparation failures" (formatter, compile, Clippy) from the behavioral red required by Guide §8.1.1. That adds to the record volume described in §3.

### 2.4 One-job fragility

One `main` push failed: after PR #331, a 5-second Chrome version probe timed out. That reopened #23 and led to a follow-up server PR #332 and a planning PR #100.

The hold on further work came from project practice: #24 was held after the main-push failure, per PR #332. The PR-required check alone did not cause it. Any intermittently slow step can trigger a full re-delivery cycle like this. Splitting the job (§5 C) would not prevent that, because the aggregate required result must still fail; it would only make reruns cheaper. The project's "no retry until green" rule is right, and it makes this cost visible. This is a single incident, not a trend.

## 3. Per-issue records

### 3.1 Where one issue's delivery is recorded

For task 1.4.5 / #22, sizes in bytes of UTF-8 text:

| Record | Size |
|---|---:|
| Server PR #330 body + 5 comments (execution, red/green, review record) | 1,615 + 8,719 |
| Issue #22 comments (execution record, closure) | 6,131 |
| Action-list section "Resource permission handoff — task 1.4.5" | 4,678 |
| Planning PR #98 body + comment (handoff delivery and its separate review) | 2,146 + 595 |
| Roadmap: "update all three current Roadmap status locations" (PR #98) | rewritten, not appended |

Server `CONTRIBUTING.md` already names the issue's execution record as the place that "carries the result". Roadmap §8 says: "Record execution evidence and state in the linked GitHub issue; update this document's phase/group completion when all constituent issues pass". Current practice goes further: an action-list section and three Roadmap status updates after every issue, which largely restate the issue record.

Each implementation issue has so far produced **one separate planning PR** with its own separate review. There were 21 planning handoff PRs (#80–#100) for tasks 1.1.2–1.4.6, including the #23 follow-up.

### 3.2 Growth

Blob sizes at `98afc78`:

| File | Bytes |
|---|---:|
| `Review/action-list.md` | 195,364 |
| `glaux-server-roadmap.md` | 263,958 |
| `glaux-server-implementation-guide.md` | 321,275 |

The Roadmap stays about the same size (262,682 → 263,958 since `f2d9f91`) because its status lines are rewritten. The action list grows by about 3.85 KB per task: 77,023 bytes over the 21 handoff PRs (#80–#100) covering 20 tasks since `f2d9f91`.

The planning `AGENTS.md` tells every Glaux Server session to read the follow-up action list "for the current handoff and approvals" and the Roadmap "for the authorised task and controlling sources". Those two files alone total about 459 KB, very roughly 115,000 tokens at four bytes per token. At the current rate, the action list alone would pass 1 MB by the end of the Roadmap.

Whether sessions actually read the whole files was not established. Either they spend a large part of their context on history, or they read selectively and could miss an approval recorded far from the current handoff.

## 4. What is working well

- **CI checks the exact commit it was given.** It tests the exact PR head, uses locked and offline dependencies, and keeps artifact evidence.
- **Nothing is quietly skipped.** No step is marked continue-on-error, and required execution is checked for missing, empty or skipped suites.
- **Merge controls hold.** The required check has no bypass, merges wait for an up-to-date branch, and the reviewed head is guarded at merge.
- **Failures are kept on the record.** Failures are preserved rather than retried to green. The #23 recovery was bounded and explained.

These are strengths to keep through any change in §5.

## 5. Options for the project lead

These are options, not requirements, and none is authorised by this review. Each would go through the existing change process. The first four concern CI and could be a single bounded task.

| Option | Effect | Trade-offs and conditions |
|---|---|---|
| A. Raise `timeout-minutes` | Immediate relief | Stopgap only. Feedback keeps getting slower. It is a workflow change, so it needs PR, checks and review. |
| B. Cache Cargo dependencies and build output between runs | Less recompilation of dependencies in the build, Clippy and the per-suite compiles | Uses a pinned caching action (for example `actions/cache` or a Rust-specific one), recorded in the dependency/licence inventory. Each run would then depend on earlier runs' cached state, a departure from the clean-setup posture set by task 1.1.4. The fault controls compile into fresh directories, so the gain for the largest step may be limited. The saving is unmeasured until a trial run. |
| C. Split the job into parallel jobs behind a final required `Rust bootstrap` job | Wall-clock time bounded by the longest job, not the sum. A failed part can be rerun without repeating everything. | The final job must fail unless every needed job succeeded, including skipped and cancelled ones (§1.3). A failure or timeout in any part still blocks the merge. Works best with B. Each job repeats setup. |
| D. Record an explicit "affected checks" rule for older per-task proofs and fault controls | Stops unrelated earlier proofs and controls recompiling on every PR | The Guide already says PRs run "applicable" and "affected" checks, and that phase completion reruns affected workflows (§1.3). So this may need a documented decision rather than a Guide change. The binding constraints: a scheduled run "cannot substitute for a missing required PR check", and tests must not be dropped *silently*. It trades some assurance for speed and should be decided explicitly. It does not touch the nine fixed false-green controls (the largest step); those are for B and C. |
| E. Give the implementing assistant a working environment with a pinned Rust toolchain, never the company laptop | Formatting and compile errors caught before pushing, with fewer CI round trips | The evidence suggests the assistant currently works in the local checkout (§2.2), so this means a different working environment, not a small tool addition. Changes the server AGENTS.md/CONTRIBUTING wording. CI stays the authoritative evidence. |
| F. Record each delivery once: the issue execution record plus the server PR. Keep the action list as a short current-state and decisions page, with the per-task handoffs moved to an archive file. | Stops the file sessions must read first from growing. Removes one planning PR per issue. | Changes the planning AGENTS.md and working practice. The historical handoffs must be preserved, not rewritten. Updating the Roadmap at group or phase completion matches Roadmap §8's own wording. |

## 6. Limits and items passed to later steps

- **Not measured:**
  - actual session time spent on CI round trips;
  - the saving from B or C (it needs a trial run, which is implementation work);
  - whether sessions read the whole action list;
  - where the implementing assistant runs, beyond the §2.2 indication;
  - per-suite and per-control timings against their 180- and 210-second limits (job logs need authenticated download and were not collected).
- **Queue time excluded:** duration figures exclude queue time. Runner variation was not separated from growth.
- **Same-provider review, a question for the project lead:** the separate reviewer is an OpenAI agent, and the implementing assistant is reported to be ChatGPT. Whether that should change is the project lead's decision. It fits the charter's independence ranking, not a specific review step.
- **Proof harness structure, passed to step 3:** the pattern of one proof program plus Python harness plus fault control per task drives §1.2 and is part of step 3's code-scaling question.

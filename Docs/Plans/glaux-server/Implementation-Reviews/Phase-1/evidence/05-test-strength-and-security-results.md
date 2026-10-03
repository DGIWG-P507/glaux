# 05 — Step 4: test strength and security results

**Status:** bounded run finished and results assessed. Step 4 remains partial: Rust security analysis did not finish and its extraction has modeling limitations. No fixes or retry were performed.<br>
**Date:** 3 October 2026.<br>
**Authority:** project lead's `proceed` following the [proposal](04-test-strength-and-security-proposal.md), recorded before execution in [planning PR #115](https://github.com/DGIWG-P507/glaux/pull/115), merge `395bc47cae9e0cad97ae88c50f682fa6f4ec9665`.<br>
**Reviewer:** Codex (OpenAI), with separate launcher authors and a separate non-author diff reviewer. Exact model versions are not exposed; prior project context was present. The PR delivery record identifies reviewed commits. This is neither human expert review nor cross-provider assurance.<br>
**Server baseline:** `27955c1b9260cd811ad6bc08f85feab43ad65028`; planning baseline before recording approval: `87ab7630cbd74c1a39eef0575073988c1fe54e53`.

## Result in plain English

The existing fault controls really do catch their twelve selected mistakes. The new pilot caught twenty of twenty-one generated mistakes and exposed one small unit-test gap: a permission helper can be changed to reject every source without those library tests noticing. That is not a discovered production vulnerability or proof that the full CI would miss it.

The dependency, Python and workflow scans completed without reported candidates. **Rust security scanning is unfinished**, not clean: it hit the launcher's per-command time limit, and its extractor warned about macros and generated code. The ordinary CI checks passed; the two added diagnostics made their lanes and the final gate red, correctly. Nothing was merged into the server or weakened to change that result.

## Existing tests: actual failure evidence recovered

The exact-head [baseline run 36565999307](https://github.com/DGIWG-P507/glaux-server/actions/runs/36565999307) was successful. This iteration read its job logs and downloaded the three retained lane artifacts below. Their ZIP digests matched GitHub's artifact metadata. These copies preserve the evidence beyond the hosted artifacts' 14-day retention; they are not new test runs.

| Deliberately wrong behavior | Observed assertion in the retained fault log |
|---|---|
| Commit a write without its outgoing work | `required outgoing work facts missing` |
| Ignore a stale revision condition | `stale supplied condition was not rejected` |
| Reuse a retry key for changed content | `changed retry intent was not rejected: case 0` |
| Accept a token for the wrong audience | `invalid access-token case accepted: wrong-audience`; actual 200, expected 401 |
| Trust expired cached signing-key information during issuer outage | `expired trust accepted during issuer outage`; actual success, expected unavailable |
| Allow cross-source System creation | `cross-source System creation was admitted`; actual 201, expected 403 |
| Return the wrong System identifier | Independent body comparison failed immediately after creation |
| Keep Systems only in process memory | Independent status comparison failed after restart: actual 404 |
| Leave restored data writable through role grants | `inspection role can modify the clone` |
| Leave the restored session default read/write | `clone does not default to read-only`; actual off, expected on |
| Leave an unsuccessful restore publicly connectable | `failed restore left its target open to every role` |
| Omit retry state from backup | `restored clone differs from the manifest or backup inventory`; missing expected retry row |

All twelve selected controls reached behavioral assertions after their required preceding stages. They were not counted merely because compilation, setup or a process failed. Their paired unmodified baselines passed; the source-changing controls also record successful restored-source runs. The restore procedure controls mutate disposable procedure copies without changing the original procedure. These are hand-picked checks, not exhaustive fault coverage. The new generated-mutation pilot below is a different, limited form of evidence.

### Preserved baseline artifacts

| Local copy | GitHub artifact ID | SHA-256 |
|---|---|---|
| [Storage](05-hosted-diagnostics/baseline-storage.zip) | 11032485665 | `9554998ad7a7f482ef0e40027009dcbaaec15c778c32c58dfb2cba2bc2e170b9` |
| [Service](05-hosted-diagnostics/baseline-service.zip) | 11032391473 | `e4cb7d5c73947545aad49bebe41c10ed17015a6bbbfe3da2e5fc96b285ab696c` |
| [Listener](05-hosted-diagnostics/baseline-listener.zip) | 11031712483 | `fa38851db8d8d1c6a60d7cebc98262c037ad14fc4b7a5e4ea45b1af03537b055` |

The fault logs, paired baseline logs and `*-failure-controls.json` files are inside their respective archives. Other logs in those archives are preserved without implying that every assertion was independently re-reviewed here.

## New hosted diagnostics

### Setup and execution identity

Temporary [server PR #358](https://github.com/DGIWG-P507/glaux-server/pull/358) contains two diagnostic launchers and 18 added workflow lines. Its initial hosted head is `75dc8b11ac691ba6cd91bfcccfef4d8fb04d4351`; [run 37141240649](https://github.com/DGIWG-P507/glaux-server/actions/runs/37141240649) checks that exact head. Production code, existing tests, the lockfile and corpus are unchanged from the server baseline. The original six lanes, final gate, action pins, permissions and 30-minute limits are unchanged. No local builds, test runs or installations occurred.

Separate non-author agent `/root/catchup_comparison` inspected the full launcher/workflow diff before execution and approved that head. It caught a setup defect: the shallow hosted checkout would not necessarily contain the baseline for comparison. A bounded fetch of the exact public baseline resolves it without moving HEAD; the fix was re-reviewed before opening the PR. This was caught before a hosted attempt and did not consume the allowed infrastructure retry.

| Added tool | Exact released version | Downloaded archive SHA-256 |
|---|---|---|
| [cargo-mutants](https://github.com/sourcefrog/cargo-mutants/releases/tag/v27.1.0) | 27.1.0 | `dfe6dc37d0342c891d2829b5a695aa57c2d0edecef7e7d0399a30cc6e206411e` |
| [CodeQL CLI bundle](https://github.com/github/codeql-action/releases/tag/codeql-bundle-v2.27.1) | 2.27.1 | `1d380f79896ededc654c7b21fafb3360136f1aeb678ad4df4df9af3910c6b815` |
| [cargo-audit](https://github.com/rustsec/rustsec/releases/tag/cargo-audit%2Fv0.22.2) | 0.22.2 | `ab28a1bdb54db4d5d8ad5981cf1f959410370b3d28250dbd35f6a44248620e39` |

The scripts verify archive digests before execution. Cargo-mutants is MIT; cargo-audit is Apache-2.0 OR MIT. CodeQL CLI uses GitHub's published terms; its bundled query sources use MIT with bundled notices retained. The selected default query packs are Rust 0.1.43, Python 1.8.11 and Actions 0.6.36. Pins, licence sources and actual commands are preserved in the launcher and result artifacts.

The mutation launcher has a 900-second total budget; the scanner launcher has 1,200 seconds shared across its four scans. Both run after their lane's ordinary checks. These are ceilings inside the existing 30-minute lane cap, not guaranteed coverage. Failed or incomplete diagnostics remain non-successful; a finding does not prevent the other selected scans from being attempted within the same budget.

### Mutation results

The tool enumerated 279 candidates in the two files, froze 21 selections and 258 exclusions before seeing outcomes, and reproduced that exact selection. Only the four approved functions were mutated. The two small permission functions generated four and five candidates; the pilot did not fill unused capacity from other functions. The three required named tests executed, and the unmutated baseline passed all 33 server library tests. The mutation run took about 289 seconds, inside the 900-second pilot budget.

| Function | Executed | Behavioral detections confirmed from diff, log and source | Meaningful survivors |
|---|---:|---:|---:|
| `PermissionSet::allows` | 4 | 4 | 0 |
| `PermissionSet::allows_source` | 5 | 4 | 1 |
| `parse_quality` | 6 | 6 | 0 |
| `matches` | 6 | 6 | 0 |
| **Total** | **21** | **20** | **1** |

There were no build failures, unviable mutants, timed-out mutants or unrun selections. This is a deliberately small sample, not a mutation-score target or a measure of overall correctness.

**Six catches needed manual interpretation.** The launcher conservatively confirmed fourteen assertion messages and left six catches unconfirmed. Two were custom-message `assert!` failures at `media.rs:423`, for invalid `0.0000` and `1.001`. Four were wrong media-matching outcomes: three caused an expected-success `unwrap()` inside the selected test to fail with `NotAcceptable`; the subtype/wildcard change also failed the explicit index comparison at `media.rs:524` in an additional library test. These are failures of expected behavior after successful compilation, not setup errors. Reading the exact diffs and test expressions confirms them; the original launcher JSON is preserved unchanged rather than rewritten to claim automatic detection.

**The survivor is not equivalent.** Replacing `allows_source` with constant `false` still passed all 33 library tests. At the baseline, `authorization.rs:978–1063` directly tests that method only for an empty resource grant, at line 1005. Its own fixture grants access to source-A, so `read.allows_source("source-A")` should be true. Constant false rejects legitimate work. [P1-05](../findings.md#p1-05--source-permission-unit-tests-miss-the-allowing-case) recommends positive assertions for both a nonempty resource-specific grant and an unrestricted grant. No test or implementation was changed here.

**Do not generalize this to all CI.** System creation uses that helper through `authorization.rs:637–658` and `system_http.rs:273–276`; its separate real-listener/database proof expects successful 201 creation. Source inspection indicates constant false would obstruct that success. That proof was **not run against this mutant**, so this is source reasoning, not claimed mutation evidence. The pilot intentionally ran library tests only.

### Security scan results

| Check | Actual coverage/evidence | Result |
|---|---|---|
| Cargo-audit | 282 locked dependencies; 1,290 advisories; no ignore list or target/severity filters | Exit 0, zero vulnerabilities and warnings reported for this snapshot |
| CodeQL Python | 47 tracked Python files accounted; 45 query evaluations completed; successful SARIF invocation, no warning/error notifications | Zero reported results |
| CodeQL GitHub Actions | One workflow accounted; 18 query evaluations completed; successful SARIF invocation, no warning/error notifications | Zero reported results |
| CodeQL Rust | All 51 tracked Rust files present in its source archive, but extraction warnings; analysis killed at 180 seconds, no Rust SARIF | **Incomplete; no security verdict** |

RustSec database commit `ef6173cbc5c50ec8166f9a5b28f07834144373ee` was committed at `2026-10-03T10:14:03+02:00` and retrieved at `2026-10-03T17:41:41Z`. The separately captured Git output supplies this pin; cargo-audit's own database commit/date fields were null. `Cargo.lock` remained SHA-256 `28f6adca13b9ab11b627cda89122c8effbc866526806715d615e0264b08183c0` before and after. The check excluded crate-registry yank status and did not assess containers, renderer packages or the toolchain. Native GitHub security settings/alerts remain unknown, not asserted disabled or empty.

**The Rust limit was our per-analysis allocation, not the approved overall ceiling.** The complete scanner launcher took about 334 seconds of its 1,200-second allowance. Rust extraction finished, but query analysis reached its separate 180-second cutoff. Only twelve diagnostic/summary evaluations finished out of 37 loaded queries; no security-query completion was recorded. Those twelve are not twelve passed security checks. The launcher preserved the failure and continued Python and Actions; its final exit was 2/incomplete.

Simply increasing that cutoff would not establish full extraction. The Rust log contains 34 warning lines: thirteen procedural macros not yet built; six macro-expansion failures with six accompanying pattern warnings; one unset `OUT_DIR` at the generated standards-corpus inclusion; and eight generated-source locations without a semantic analyzer. These affect, among other files, authentication, authorization, runtime and standards validation. The extractor also recorded `cargo_all_targets: false`. Archive presence is therefore not evidence that every expression, macro or build configuration was modeled. These are scanner/setup limitations, not demonstrated defects in the server. No automatic retry or budget extension was taken.

Python also archived the workflow file; Actions SARIF includes some shared baseline artifact/notification entries. The coverage counts above use actual language-specific tracked sources, not a misleading count of every SARIF artifact. The zero results are limited to these default scans, not a clean bill of health.

### Durable records and preservation

The [run metadata](05-hosted-diagnostics/run-metadata.json) records the exact head, seven job outcomes and six hosted artifacts. All ordinary steps passed, including corpus checks after the diagnostics. Tracked-file pre/post hashes matched for the mutation pilot; the scanner's pinned production-path comparison and lockfile check also passed. The two diagnostic failures and resulting red `Rust bootstrap` gate remain in the run record. No rerun was used to obtain green.

| Retained record | SHA-256 | Contents/provenance |
|---|---|---|
| [Mutation records](05-hosted-diagnostics/mutation-records.zip) | `4f99fce8ffd6fd9364259c9e8e5f34b32b140cab9db8993ec375497b08d1c38b` | All 129 result/log/diff files copied byte-identically from artifact 11280930448; only the downloaded tool executable archive is omitted. Original hosted ZIP digest: `abf8c54ebfaa449307f217198a9c32c0dc82f1c1efd4ccca18270c7acf0e1b89`. |
| [Scanner records](05-hosted-diagnostics/scanner-records.zip) | `1538ea0f87810e3a0f739f4f8ded59e7fb45b2b078716264e4e87891b5eda1e7` | Unmodified hosted artifact 11280442849: SARIF, commands, extraction logs, coverage lists, advisory output and ordinary-lane evidence. |

Inside the mutation archive, `phase1-mutants/selection-frozen.json` records selected/excluded candidates; `tool-output/mutants.out/outcomes.json`, `log/` and `diff/` hold original per-mutation results. `outcome-accounting.json` retains the launcher's conservative classifications. The six manual interpretations above supplement that record; they do not alter it. Scanner originals are under `phase1-scanners/`, especially `scanner-summary.json`, `rust-extract.stdout.log`, `rust-analyze.stderr.log`, the two SARIF files and `cargo-audit-results.json`. No CodeQL database or Security dashboard upload was made.

## Limits and next decision

This iteration answers the bounded mutation question and three of the four selected scanner checks. It does **not** finish Rust security analysis or review gate 1. Its one new server-test recommendation is P1-05, Low; previous recommendations remain unchanged and unadopted. No production vulnerability was established by this run, but the incomplete Rust check prevents an overall clean-security claim.

**Recommended next step:** a focused Rust-only diagnostic follow-up, addressing extraction configuration and the per-analysis allocation within a stated hosted budget. Reuse the completed mutation/dependency/Python/Actions evidence; do not repeat the campaign or add a tool portfolio. That follow-up and any P1-05 test change require the project lead's next authorization; neither is silently begun here. The approved setup-retry allowance was not used, and the timeout is not being relabelled as a transient infrastructure failure.

The temporary diagnostic PR is to be closed without merging after this evidence is reviewed, with its branch retained. The delivery record records closure and the separately reviewed results commit. No production fixes, lockfile changes, permanent scanners or repository permission changes were made. Phase 2 remains blocked by gate 1; only the project lead can close it. Step 5 and the full gate's provider-accounting/cross-provider limitations remain separate work. No human expert reviewed these results.

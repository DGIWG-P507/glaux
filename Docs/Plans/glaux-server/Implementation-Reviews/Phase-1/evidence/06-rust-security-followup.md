# Rust security scan follow-up

**Status:** bounded Rust follow-up complete, with remaining modeling limits. Step 4 is complete on that stated basis, not a claim of full security coverage.<br>
**Date:** 3 October 2026.<br>
**Prepared for:** the Glaux project lead and Phase 1 implementers.<br>
**Authority:** the project lead's `proceed` following [the first results](05-test-strength-and-security-results.md), recorded before execution in [planning PR #117](https://github.com/DGIWG-P507/glaux/pull/117), merge `042641c26fd61b188c4d9c13360d4423a2b765f1`.<br>
**Reviewer:** Codex (OpenAI), with a separate launcher author, read-only source investigator and non-author reviewer. Exact model versions are not exposed; prior context was present. This is not human or cross-provider assurance. The delivery PR records exact reviewed commits.<br>
**Server baseline:** `27955c1b9260cd811ad6bc08f85feab43ad65028`. **Planning baseline:** `042641c26fd61b188c4d9c13360d4423a2b765f1`.

## Scope

This follow-up addresses only the unfinished Rust scan. It reuses the completed mutation, dependency, Python and workflow evidence; it does not repeat those campaigns, fix server code or adopt P1-05. The temporary server PR must close without merging, with its branch retained. The original six CI lanes, required final gate and 30-minute lane limits remain unchanged.

The same CodeQL CLI 2.27.1 bundle and default Rust query pack 0.1.43 are selected. Its archive SHA-256 is `1d380f79896ededc654c7b21fafb3360136f1aeb678ad4df4df9af3910c6b815`. Installation and execution are confined to the disposable GitHub-hosted runner. Results are ordinary artifacts, not a GitHub Security upload. No repository permission or permanent scanner setting changes are authorized.

## Why the setup changes

The prior scan stopped analysis after 180 seconds despite having time left in its overall allowance. This follow-up retains the 1,200-second total budget, allows extraction up to 300 seconds and analysis up to 900 seconds, always bounded by remaining campaign time. Ordinary CI setup also counts toward the unchanged 30-minute lane ceiling. A timeout remains incomplete execution, not a passing scan.

The pinned extractor's [toolchain selection](https://github.com/github/codeql/blob/6e9f9e38390175c41b99070a423c875f450759ca/rust/extractor/src/toolchain.rs) chooses Rust 1.97.0 for stable projects, while Glaux requires 1.98.1. This is a source-backed explanation to test for the previous missing-macro and `OUT_DIR` warnings, not a proven runtime cause merely because the versions differ.

The [pinned configuration source](https://github.com/github/codeql/blob/6e9f9e38390175c41b99070a423c875f450759ca/rust/extractor/src/config.rs) provides internal environment controls for an explicit build-script command, sysroot, procedural-macro server and all-targets selection. These are version-specific diagnostic controls, not a promise of stable public CLI compatibility. Build scripts and macro support were already enabled; there is no invented “enable macros” flag. The runner must record the effective configuration and preserve any remaining warnings.

The intended build-script command uses the existing project compiler, locked offline dependencies and default features. None of the four workspace/package manifests defines a feature set. Matching compiler and macro-server paths are checked rather than guessed. The [Rust distribution source](https://github.com/rust-lang/rust/blob/1.98.1/src/bootstrap/src/core/build_steps/dist.rs#L428) places that server in the compiler distribution; no separate IDE tool is introduced. Matching `rust-src` supplies standard-library source on the hosted runner only.

This does not replace every internal toolchain choice. The pinned extractor can still select and install its own 1.97.0 compiler for Cargo metadata on the disposable runner. The [pinned Cargo-command implementation](https://github.com/rust-lang/rust-analyzer/blob/f938641be53c2e4bacd7dc46bddb74825a3e9d28/crates/project-model/src/sysroot.rs) preserves that environment choice. Installed toolchains are recorded before and after; Glaux's checked-in toolchain and its ordinary CI build remain unchanged.

## Execution and results

Temporary [server PR #359](https://github.com/DGIWG-P507/glaux-server/pull/359) started at separately reviewed head `4ed92de3c01e43efa240fab85fbf71c23d4c82e9`. Only its launcher and eleven workflow lines differ from the server baseline. Reviewer `/root/rust_followup_review` approved the actual complete diff and pinned-source configuration before execution.

### Initial attempt and the diagnosed retry

[Run 37160985948](https://github.com/DGIWG-P507/glaux-server/actions/runs/37160985948) stopped the diagnostic after 116.692 seconds. Extraction itself completed in 99.589 seconds, but the launcher could not recognize its configuration header because ANSI color codes interrupted the literal text. Query analysis did not start. This is a defect in the temporary launcher, not a server defect or a security result.

The same formatting defect also made the launcher's warning counter report zero despite three actual extraction warnings. The raw logs were preserved and read directly, so that zero was not accepted as evidence. The sole allowed setup retry corrected ANSI normalization in both places, kept every exact configuration check, and added positive and negative parser controls. It did not change the scanner, queries, runtime budgets or source scope. The same non-author reviewer separately approved revised head `579707569da11fe9ab3043c780c393149e2a9849` before execution.

All 331 tracked-file hashes matched before and after this attempt; `Cargo.lock` stayed at SHA-256 `28f6adca13b9ab11b627cda89122c8effbc866526806715d615e0264b08183c0`. The actual log records the requested configuration. The previous unbuilt-macro, failed expansion and unset `OUT_DIR` warnings are absent; the remaining warnings concern two generated Glaux corpus files and one generated `serde_core` file. That improvement does not prove those generated files were fully modeled. The installed-toolchain records confirm that the scanner provisioned 1.97.0 while project 1.98.1 remained active.

The unchanged [initial artifact](06-rust-followup/initial-attempt.zip), GitHub artifact `11287940526`, is 238,620 bytes with SHA-256 `2ba8f2a78292cd1ef159bcdbd3d5263b6ee6f1e75c1ad528bde4e97e95d56d74`. It preserves the failed summary, raw logs and source hashes without rewriting their original claims.

### Retry result

[Run 37161581525](https://github.com/DGIWG-P507/glaux-server/actions/runs/37161581525), on that exact revised head, completed extraction and analysis within the approved budget:

| Check | Observed result |
|---|---|
| Campaign / extraction / analysis | 345.492 / 102.846 / 221.453 seconds; extraction and analysis both exited 0 |
| Selected query execution | All 37 query evaluations completed, including all 17 security queries |
| SARIF results | One successful invocation, 26 rule entries including 17 security-tagged rules, **zero reported results** |
| Effective configuration | All six required checks passed: all targets, default features, build-script command, sysroot, standard-library source and matching procedural-macro server |
| Parser sensitivity controls | Six passed: plain and colored configuration accepted; colored wrong configuration and absent configuration rejected; colored warning detected; ordinary informational text containing `--warnings=show` not classified as a warning |
| Tracked source preservation | All 331 before/after hashes matched; lockfile hash unchanged |
| Source presence | All 51 tracked Rust files appeared in the source archive; none missing. This is presence, not proof of complete semantic modeling. |

The query inventory, logs and SARIF support different counts: 37 evaluated queries are not 37 security rules, and zero results is not zero diagnostic notifications. The SARIF SHA-256 is `9ade50fab374e9e89b875804b8bcdb833a7bd3288f5b9d4c5b865e649ea316bb`.

### Remaining modeling limits

Three extraction warnings remain. Two name generated copies of Glaux's `corpus.rs`; the third names a generated `serde_core` `private.rs`. The analyzer says these standalone files were not among those loaded from the Cargo manifest, and macro expansion was skipped for them. Five separate informational messages concern generated files likely included through `include!`; these are not five additional warnings.

The generated Glaux catalog contains URI/`include_str!` entries and is included through `OUT_DIR` in `validation.rs`. The earlier missing-macro and unset-`OUT_DIR` warnings disappeared, but neither that improvement nor file presence proves the generated expressions were completely modeled. Conversely, the warnings do not establish that the whole validation module was omitted or that the server is defective.

Direct inspection of SARIF, rather than just the launcher's summary, found:

- **493 execution notifications:** 422 at level `none`, 68 at `note`, and three extraction warnings without an explicit level. These are scanner diagnostics, not 493 vulnerability findings.
- **314 unresolved-macro notifications:** 157 locations in each of the two generated Glaux corpus copies. None of these notifications points to a tracked-source file; they are not 314 distinct production defects.
- **144 path-resolution inconsistencies and 40 nodes at the type-path-length limit** in the query metrics. They remain analysis limitations, not independently verified Glaux faults. Broader extractor telemetry includes dependency code and must not be presented as Glaux-only defect counts.
- The extraction summary reports **56 files extracted without error and three with errors**. That generated-file-inclusive measure is distinct from the 51 tracked-source files present in the archive.

Two raw summary counters need this explicit qualification. `diagnostic_notifications: 0` counts only explicit warning/error levels and misses the three warnings whose level is omitted; it does **not** mean no diagnostics. `diagnostic_log_line_count: 7` includes the three actual warning messages repeated across two logs, plus a false positive matching the metric text “extracted without error.” Both raw summaries are retained unchanged; these interpreted counts control this report. Shared source-archive baseline entries mentioning Python are not evidence of a repeated Python query campaign.

The launcher deliberately returned exit 1 with `completed_with_coverage_or_diagnostic_questions`. Therefore the static lane and required `Rust bootstrap` gate are red; **this is not a passing PR**. All five other lanes passed on both attempts. Ordinary static checks preceding the diagnostic and the inline post-diagnostic corpus check passed; a later duplicate workflow corpus step was skipped after the diagnostic's failure. No failed check is relabelled as passed, and this temporary branch must not merge.

### Preserved evidence

The original GitHub artifact ZIPs are stored unchanged, not reconstructed from selected logs. Each contains its command results, effective configuration, source hashes and diagnostic summary; the completed retry also contains the query inventory, SARIF, parser controls, extraction logs and coverage metrics. Run metadata records every job and artifact, including the failed gate.

| Attempt | Artifact and metadata | Original bytes / SHA-256 |
|---|---|---|
| Initial setup failure | [ZIP](06-rust-followup/initial-attempt.zip), artifact `11287940526`; [run metadata](06-rust-followup/initial-run-metadata.json) | 238,620 / `2ba8f2a78292cd1ef159bcdbd3d5263b6ee6f1e75c1ad528bde4e97e95d56d74` |
| Sole completed retry | [ZIP](06-rust-followup/completed-retry.zip), artifact `11287573982`; [run metadata](06-rust-followup/retry-run-metadata.json) | 7,638,902 / `5a4935e99cff8d5b1432d12ee24e2d2ff1e674c1b77df84681f8489c5fa64894` |

The retry again provisioned the scanner's internal 1.97.0 toolchain while Glaux's 1.98.1 remained active. No local software was installed or scanner/build command executed. No production file, test, dependency or repository setting changed. The delivery record covers separate review of this report against the actual artifacts; the diagnostic PR's final execution record records closure without merge and retention of its branch.

## Next decision

The bounded Step 4 review is complete: reuse the [previous mutation/dependency/Python/workflow results](05-test-strength-and-security-results.md), now supplemented by a completed Rust scan with the limits above. Zero reported security results does not prove security, complete generated-code modeling or CSAPI conformance. The remaining coverage limits stay visible for the project lead's gate decision; they do not silently authorize a third run or a scanner repair project. P1-05's previously identified unit-test recommendation remains open and unadopted; no new server defect is established here.

Recommended next, on a new `proceed`, is **a bounded Step 5 proposal for the OpenSensorHub comparison**: pin the peer, select the Phase 1 System requests, state expected answers from the standard, and identify the disposable hosted setup and stop conditions before requesting execution approval. No peer environment or comparison runs in this iteration. Phase 2 remains blocked by [gate 1 (#339)](https://github.com/DGIWG-P507/glaux-server/issues/339), which only the project lead closes.

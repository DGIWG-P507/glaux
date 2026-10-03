# 04 — Step 4 proposal: test strength and security scanning

**Status:** proposal ready for the project lead; execution has not started. Publishing this proposal does not approve its tools or workflow changes.<br>
**Date:** 3 October 2026<br>
**Authorised work:** draft this proposal, following step 3. No new test campaign, scan, installation or server change was authorized or performed.<br>
**Prepared for:** the Glaux project lead and the assistant executing this review.<br>
**Prepared by:** Codex (OpenAI), with separate source-inventory agents. Exact model version is not exposed; prior project context was present. A separate non-author agent reviews the document diff before publication, with the reviewed commit/outcome in the PR. This is not human expert or cross-provider assurance.<br>
**Baseline:** server [`27955c1b9260cd811ad6bc08f85feab43ad65028`][Server]; planning [`8e78df0920b800a4ac7513ee3a95a88e371cb26e`][Planning]. Both checkouts were synchronized; all fifteen review gates remain open.

## Recommendation

Run **one bounded hosted review**, combining the existing fault-test evidence with a small automated mutation sample, CodeQL source scanning and a Rust dependency-advisory check. A mutation deliberately changes working code to see whether the tests notice. A scanner looks for a different class of problem; neither substitutes for the other or proves CSAPI conformance.

Use GitHub-hosted Linux only. No company-laptop installation, cloud account, permanent service or paid runner upgrade is needed by this proposal. Keep the production code unchanged, preserve the current six-lane gate and stop after reporting results. Do not begin an open-ended campaign to improve a score.

The proposed additions are `cargo-mutants`, CodeQL and `cargo-audit`, installed only on disposable hosted runners after approval. The [Guide §8.1.1][Guide] already permits targeted mutation methods; this proposal selects a small initial application rather than adopting every tool mentioned there.

## What already exists

| Observed at the baseline | What it does and does not establish |
|---|---|
| Six CI lanes, all successful in [run 36565999307][BaselineRun], with the final `Rust bootstrap` gate successful. | The run and job metadata were checked for this exact server SHA. This proposal did not inspect all execution logs or rerun it; green metadata alone is not detailed failure-sensitivity evidence. |
| [Hand-picked fault controls][Ci] for audience/key trust, source permissions, atomic writes, conditional writes, retries, retrieval/restart and restore. | These already challenge important mistakes. Execution should inspect their retained exact-head logs, reusing them rather than rebuilding the controls. Missing or expired logs are a visible evidence gap, not a pass. |
| A bounded [schema-parser campaign][SchemaFuzz], plus numeric/time campaigns. | Here “mutation” includes changing **input data**. It is not a generated source-code mutation campaign. A checkout search found no `cargo-mutants` invocation/configuration. Both methods can be useful, but their claims differ. |
| A [dependency/licence inventory][Dependencies] and targeted advisory checks; no checked-in CodeQL workflow or Dependabot configuration. | An inventory is not a vulnerability scan. Absence of a configuration file does not establish the state of GitHub-managed scanning or alerts. Public API reads of default setup, code-scanning alerts, Dependabot alerts and vulnerability-alert settings returned **401, authentication required**; their live state is unknown. |

## The bounded execution

### 1. Reuse existing evidence and prove the pilot baseline

Read available logs/artifacts for the pinned run, concentrating on the existing atomic/conditional/retry, authentication/key-refresh, System create/read and restore fault controls. Record the exact assertion each selected fault reached, not just the job conclusion. If retained evidence is unavailable, the ordinary CI run on the diagnostic PR can supply fresh evidence; label its different head and verify that production sources, existing tests and lockfile still match the baseline.

Before generated mutations, run the selected ordinary tests unmodified and verify their actual discovery/execution. Do not count a setup or compiler failure as a test detecting a defect. Preserve the baseline and original corpus bytes.

### 2. Generate a small set of mistakes in four functions

| Selected production logic | Existing independently written tests to exercise it |
|---|---|
| [PermissionSet::allows and allows_source][PermissionSource], lines 114–133 | `configured_policy_keeps_identity_actions_and_resource_pairs_distinct` (line 978 onward): source/resource pairing, wrong combinations and empty grants. This samples permission predicates, not the whole authentication or authorization system. |
| [HTTP media parse_quality and matches][MediaSource], lines 334–390 | `media_quality_uses_exact_grammar_and_specific_exclusions` and `media_parameters_preserve_quotes_values_and_weight_order` (lines 409–514): exact weights, invalid inputs, exclusions, wildcards and parameter matching. |

All three test names are already in the required-test inventory. Pin a compatible `cargo-mutants` release and record its candidate listing before execution. Limit selection to these four functions, **at most 24 generated changes, at most six per function**. Prefer comparison/boundary/Boolean changes, then constant-return changes; record the chosen mutations and excluded remainder before seeing outcomes. These are spending limits, not a representative sample, quality quota or minimum acceptable score. Do not broaden a file filter into the whole server. If a selected function produces no suitable candidates, record that limit rather than silently substituting another target.

The [tool's documented outcomes][MutationResults] distinguish detected changes, survivors, changes that do not compile and timeouts. Inspect the actual failing assertion before accepting a detection. For a survivor, establish whether it changes required behavior, is equivalent for the supported contract, or remains unresolved. Retain the mutation diff and decisive output. Never weaken tests or modify production code to improve this review's result.

**Important execution boundary:** Glaux's HTTP/database proofs are standalone example programs launched by Python/container wrappers. Ordinary Cargo tests do not execute those proof programs. This pilot intentionally selects unit-test-observable logic; its results cannot be presented as end-to-end enforcement evidence. The existing real-listener/database fault controls remain separate. This also follows [cargo-mutants' documented integration-test limitation][MutationLimits].

### 3. Run source and dependency scans once

- **CodeQL:** explicitly select Rust, Python and GitHub Actions, using the standard default queries, with no custom query-pack project. Check extraction/analysis diagnostics and the actual analyzed files before interpreting an empty result. Rust is supported; its `none` build mode still compiles/runs build scripts and compiles macro code, so use the disposable runner without operational secrets. [GitHub build guidance][CodeqlBuild]
- **Artifact-only results:** use a pinned CodeQL CLI bundle to create/analyze its databases and save SARIF result files as ordinary review artifacts, omitting its separate upload command. Do not enable dashboard scanning, upload the databases or request `security-events: write`. The CLI is available for public repositories under its published terms. This route also avoids adding a new workflow action to the current inventory allowlist; retain the existing checkout/upload actions and record the added CLI's pin/licence separately. If execution needs different access, stop rather than change repository permissions. [GitHub CLI guidance][CodeqlCli]
- **Rust dependencies:** run a pinned `cargo-audit` with `--file` pointing to the existing `Cargo.lock`, from an owned directory outside the checkout. Verify the lockfile digest before and after; record the RustSec database commit/retrieval date and tool version. This checks known advisories affecting locked Rust packages, not unknown vulnerabilities or every deployed component. Do not generate a replacement lockfile, run automatic fixes or update dependencies. [RustSec usage guidance][Rustsec]

Read existing Dependabot results if authorized access is available, but do not change settings or enable automatic update PRs. If access remains unavailable, keep that limitation and report the explicit lockfile scan on its own merits. GitHub supports Cargo lockfiles, but its [scope classification][DependabotScope] cannot distinguish Cargo development from runtime dependencies; [action alerts][Dependabot] also do not cover SHA-pinned action references. Keep those pins. This is not a complete supply-chain assessment: PostgreSQL/PostGIS container packages, vendored renderer assets and the Rust toolchain are outside the new advisory scan.

Group scanner results by their underlying cause and inspect applicability; a reported advisory is not proof of exploitability, and a completed scan is not a clean bill of health. Report known-affecting results and unresolved candidates separately. No exploit demonstration, production endpoint probing or automatic remediation is proposed.

## Hosted setup and spending limits

Approval would authorize one **temporary, do-not-merge server diagnostic PR**, containing only the reviewed launcher/workflow configuration needed for this run. Record approval in the existing action list's Current state before execution, as the review charter requires. Do not create a new implementation task or alter a review-gate issue.

Use the existing PR-triggered workflow: there is no current manual-dispatch entry point to assume. Append the mutation pilot within `rust-suites` and scanner commands within `static-checks`, after their ordinary checks, retaining all existing checks, inventory guards, evidence uploads, six `needs` entries and unconditional `Rust bootstrap` logic. Preserve each diagnostic's outcome; one reported vulnerability must not silently prevent the other selected scans, and collecting outcomes must not turn a failed scan into a passing lane. **Do not add an ungated job.** The diagnostic branch's additions are not merged into `main`; ongoing scanning, new required checks and any fixes would need a later decision. Obtain separate review of the launcher/configuration before running new tools, and of the resulting evidence before publication.

Use the existing Ubuntu 24.04 hosted route and pinned project Rust toolchain. Resolve exact tool versions, action commit pins, query-pack versions and licences during launcher preparation; retain them with the command/configuration and source SHA. No floating “latest” install. Network use is limited to ordinary dependency/tool/advisory retrieval, not operational services. Use the existing isolated database harness if the ordinary suite reruns.

Keep each lane's **existing 30-minute cap**, including setup. Stop at the time or mutation-count limit and preserve completed, timed-out and unrun work. Allow at most one retry for a specifically diagnosed infrastructure/setup failure, retaining the original failure; do not rerun a behavioral failure until green. If the selected scans cannot fit, report the exact uncompleted part for a decision—do not add batches or silently increase budgets. New checks can intentionally make the diagnostic PR red; do not weaken them or the gate to obtain a green badge.

The intended delivery is **one execution iteration after approval**, not a promise that external jobs or tool compatibility cannot block it. Existing job artifacts expire; preserve the decisive result records and their checksums in the numbered review evidence before relying on them long term. Use synthetic data, bounded logs and no credentials. Ordinary hosted artifacts and diagnostic branch records remain traceable; after the report, close the diagnostic PR without merging and retain its branch for reproducibility. The branch is not an alternative implementation queue.

## Completion and next decision

The execution report should answer, in plain English: what mistakes the sampled tests caught or missed; what scans actually analyzed; what issues need action; and what remains untested. Preserve raw outcomes, pins and links, but summarize repeated findings once. A setup failure, unavailable evidence or exhausted budget remains incomplete coverage, not a pass. Genuine test gaps become recommendations in the existing findings register; repairs are not included in this review authorization.

**A new `proceed` approves only this bounded diagnostic setup, hosted execution and results report.** It does not adopt P1-04, authorize production fixes, permanent CI/security settings, Phase 2, the OpenSensorHub comparison or gate closure. If execution cannot meet this proposal, stop with the specific limitation rather than turning it into a new research program.

Step 4 is not complete merely because its proposal is published. Step 5 and the full gate review's provider-accounting/independence limitations remain separate; scanners do not replace the required cross-provider review or a human expert. Only the project lead closes gate 1.

[Server]: https://github.com/DGIWG-P507/glaux-server/tree/27955c1b9260cd811ad6bc08f85feab43ad65028
[Planning]: https://github.com/DGIWG-P507/glaux/tree/8e78df0920b800a4ac7513ee3a95a88e371cb26e
[Guide]: https://github.com/DGIWG-P507/glaux/blob/8e78df0920b800a4ac7513ee3a95a88e371cb26e/Docs/Plans/glaux-server/glaux-server-implementation-guide.md#811-test-strength-review-and-reliable-execution
[BaselineRun]: https://github.com/DGIWG-P507/glaux-server/actions/runs/36565999307
[Ci]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/ci.md
[SchemaFuzz]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/fuzz/schema_parser.rs
[Dependencies]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/dependencies.md
[PermissionSource]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/src/authorization.rs
[MediaSource]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/src/http_boundary/media.rs
[MutationResults]: https://mutants.rs/using-results.html
[MutationLimits]: https://mutants.rs/limitations.html
[CodeqlBuild]: https://docs.github.com/en/code-security/reference/code-scanning/codeql/build-options-for-compiled-languages
[CodeqlCli]: https://docs.github.com/en/code-security/concepts/code-scanning/codeql/codeql-cli
[Rustsec]: https://raw.githubusercontent.com/rustsec/rustsec/main/cargo-audit/README.md
[Dependabot]: https://docs.github.com/en/code-security/reference/supply-chain-security/dependency-graph-supported-package-ecosystems
[DependabotScope]: https://docs.github.com/en/code-security/reference/supply-chain-security/supported-ecosystems-and-manifests-for-dependency-scope

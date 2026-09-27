# 00 — Scoping observations, 27 September 2026

**What this is:** the measurements and source checks behind the decision to open this review. These are recorded as they were taken during a scoping conversation with the project lead. They are **starting observations for the review steps to confirm or reject**. They are not findings. None of them has been through a review step.

**Observed by:** Claude Code (Anthropic), in a session with the project lead. The provider switched models mid-session, and the exact model for each measurement was not recorded.<br>
**Server state observed:** `DGIWG-P507/glaux-server` main at [`d0ef755`](https://github.com/DGIWG-P507/glaux-server/tree/d0ef755bf4f8a0f6598dc68276ca6730a7abf9e2) (27 September 2026 10:05 −0400), after #23 closed. Issue #24 was in progress on another assistant's branch `task/1.5.1-system-create`. That branch was not inspected.<br>
**Planning state observed:** `DGIWG-P507/glaux` main at `273002a`.<br>
**Method:** the server was cloned read-only into a temporary directory. GitHub's public REST API was used without credentials. No server file, issue, setting or branch was changed. No software was installed.

## 1. Delivery pace

Merges to server `main` by date: 12 on 21 Sep, 3 on 22 Sep, 5 on 24 Sep, 3 on 25 Sep, 2 on 26 Sep and 2 on 27 Sep. Implementation issues #3–#23 closed between 21 and 27 September, about three a day. Phase 1 ends with #26.

## 2. CI duration against its limit

The single CI job, `Rust bootstrap` in [`.github/workflows/build.yml`](https://github.com/DGIWG-P507/glaux-server/blob/d0ef755bf4f8a0f6598dc68276ca6730a7abf9e2/.github/workflows/build.yml), has `timeout-minutes: 20`. Successful run durations were taken from `GET /repos/DGIWG-P507/glaux-server/actions/runs` as `updated_at − run_started_at`, rounded:

| Task (issue) | Successful PR run |
|---|---|
| 1.3.3 (#15) | 7.6 min |
| 1.3.4 (#16) | 5.9 min |
| 1.3.5 (#17) | 7.4 min |
| 1.4.1 (#18) | 11.5 min |
| 1.4.2 (#19) | 8.9 / 12.1 min |
| 1.4.3 (#20) | 11.7 / 13.6 min |
| 1.4.4 (#21) | 14.4 min |
| 1.4.5 (#22) | 15.2 min |
| 1.4.6 (#23) | 17.9 min; follow-up PR #332 17.2 min; main push 16.7 min |

Each task so far has added its own CI steps to the one job. Queue wait and runner variation were not separated out.

## 3. Why CI runs fail

The 25 most recent failed runs from `actions/runs?status=failure`, retrieved during the 27 September 2026 scoping session (exact time not recorded) and keyed on each job's first failed step. The separate reviewer later reproduced the same tally from 27 failed runs. The two newest of those came from #24's branch (PR #333); this document does not otherwise inspect that branch.

| First failed step | Runs |
|---|---:|
| Verify formatting and locked dependencies (rustfmt) | 10 |
| Lint Rust and check Python syntax (Clippy) | 4 |
| Render local API documentation (browser) | 3 |
| Prove initial discovery | 2 |
| Six other steps, one each (one of them a deliberate stop step) | 6 |

Most first failures come from formatting, lint or compile checks that a local Rust toolchain would normally catch before pushing. The comments on [PR #330](https://github.com/DGIWG-P507/glaux-server/pull/330) show how this works in practice:
- No local runtime or installation was used.
- A formatter-only stop was fixed by applying CI's generated rustfmt patch.
- There was a compile stop over an unavailable `axum::Json` import.
- There was a Clippy type-complexity stop.

By count of PR runs in the listing, #21, #22 and #23 took 5, 7 and 9 runs to reach a passing head. Separately, one of the three browser-rendering failures was the `main` push run after PR #331 merged (`74408da`). `main` stayed red until PR #332 merged.

The project lead chose GitHub-hosted Linux, with no company-laptop installation, on 21 September 2026 (see server `CONTRIBUTING.md`). This observation does not question that choice. It measures what the choice currently costs.

## 4. Size of the per-task records

- `Review/action-list.md` (Git blob sizes) was 14,778 bytes when created at `345397e` on 20 September 2026 and is 195,364 bytes at `273002a`.
  - It reached 118,341 bytes at `f2d9f91` (21 September), before the first implementation handoff. That part came from the pre-implementation follow-up updates, the Part 5 planning, and the licence and setup decisions.
  - The per-task implementation handoffs for tasks 1.1.2–1.4.6 then added about 77 KB, roughly 3.9 KB per task.
  - AGENTS.md tells every session to read this file first.
- Server tree at `d0ef755`: 10,869 lines of Rust under `crates/*/src`; 11,097 lines under `crates/*/examples`, mostly 11 `glaux-server` `*-proof.rs` programs of 676–1,407 lines each, plus the 164-line `glaux-standards/examples/discovery-schema-proof.rs`; 636 lines under `crates/*/tests`; 39 Python scripts (5,721 lines in `scripts/` including non-Python files); 11 `docs/*-tests.md` files.

## 5. How far the write path is shared

The trusted write/storage layer at `d0ef755` is System-specific: `SystemRecord` and `SystemRepository` in `storage.rs`; `CreateSystem`, `UpdateSystem`, `create_system`, `update_system` and `record_denied_system_create` in `application.rs`; `SystemRevision` in `revisions.rs`. This matches Roadmap §4 Phase 1 group 1.3 ("add family tables with their later capabilities rather than creating empty subsystems"). It is recorded only because Phase 2 is where more resource families arrive.

## 6. Test expectations traced to the standard

A search of `crates/*/examples`, `crates/*/tests`, `crates/*/src` and `scripts/` for URI-style `/req/`, `/conf/` and `/ats/` identifiers found only `/conf/api-common`. That search does not catch prose citations. The `docs/*-tests.md` files cite Guide sections. The HTTP and authentication docs cite RFC 9110, 9457, 3986, 8259, 9068, 8725, 7517 and 6750, and other docs cite further RFCs.

This may be expected. Most Phase 1 work so far (storage, value types, authentication, permissions) has no CSAPI requirement behind it. Guide §8.1.1 says tests should "state the controlling requirement, independently expected answer and a plausible wrong behavior", applied "in proportion to the behavior being changed". Whether that is being met for CSAPI-facing behavior starts to matter with #24.

## 7. External conformance suites and peer implementations

- **Botts ETS.** [`Botts-Innovative-Research/ets-ogcapi-connectedsystems10`](https://github.com/Botts-Innovative-Research/ets-ogcapi-connectedsystems10) was created 28 April 2026 and last pushed 4 August 2026. Its README describes a pre-beta TEAM Engine suite for Parts 1 and 2 ("240 total / 191 exact / 2 helper / 47 candidate" procedures) that is not a CITE submission. Botts Innovative Research is also listed as a submitting organisation of CSAPI Part 1. [IDR-050](../../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-050-conformance-harness-strategy-report.md) concluded that no *official* public CSAPI suite existed at its evidence freeze (16 September 2026). It did not mention this unofficial suite.
  - The project lead reports that the suite's test logic is AI-generated.
  - **It is therefore not used as an independent test oracle** (a trusted source of expected answers) in this review. Two AI-derived readings of the same standard may share errors. Independently developed software is known to fail on the same inputs more often than chance (Knight and Leveson, IEEE TSE 1986), and LLM errors are correlated across models (["Correlated Errors in Large Language Models", ICML 2025](https://arxiv.org/abs/2506.07962)).
- **OpenSensorHub.** [`opensensorhub/osh-core`](https://github.com/opensensorhub/osh-core) was created 17 October 2015 as a fork of `sensiasoft/sensorhub`, which was created 29 October 2014. Its top contributor, `alexrobin` (profile name "Alex Robin"), has 1,975 commits.
  - The pinned CSAPI Part 1 source in the server corpus (`23-001r0.adoc`) gives `:fullname: Alexandre Robin`, which in this document format names the editor. That he is the same person as `alexrobin` is plausible but inferred. Riverside Research is listed among the submitting organisations.
  - The project's history is long, but how much of the current CSAPI code predates AI coding assistants was not measured.
  - It is not an oracle either: the project's research documented places where OSH departs from the standard.
- **OS4CSAPI client.** The Guide §8.1 names the "pinned OS4CSAPI TypeScript client" as one external-client check. [`OS4CSAPI/ogc-client-CSAPI_2`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2) is a fork of `camptocamp/ogc-client`, created 31 January 2026. How much of its CSAPI code was written by AI was not established here.

## 8. Standards activity upstream

The [Connected Systems SWG repository](https://github.com/opengeospatial/ogcapi-connected-systems) closed several Part 3 decision issues on 24 September 2026 (#187, #188, #190–#195). Open items include #201 (Part 2 SystemEvent JSON mapping), #179 (observed/controlled property query parameters, targeting v1.1) and #185 (bulk POST). None of these was compared against Glaux's pinned interpretations.

## 9. Research on AI-to-AI review

Cross-model review helped unevenly in one study ([arXiv 2607.21656](https://arxiv.org/abs/2607.21656), competitive-programming tasks): a stronger reviewer improved weaker drafts, while the reverse lowered accuracy. Other work reports that review in the same session as production detects fewer errors than review in a separate session ([arXiv 2603.12123](https://arxiv.org/abs/2603.12123)). This supports using model diversity and fresh context to *find* issues. It does not make any AI review a substitute for human-anchored evidence.

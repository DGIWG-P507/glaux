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

The 25 most recent failed runs from `actions/runs?status=failure`, keyed on each job's first failed step:

| First failed step | Runs |
|---|---:|
| Verify formatting and locked dependencies (rustfmt) | 10 |
| Lint Rust and check Python syntax (Clippy) | 4 |
| Render local API documentation (browser) | 3 |
| Prove initial discovery | 2 |
| Six other steps, one each (one of them a deliberate stop step) | 6 |

The PR records say this outright. [PR #330's execution record](https://github.com/DGIWG-P507/glaux-server/pull/330) reports "No local runtime used" and describes applying CI's generated rustfmt patch artifact. Its preparation failures included a compile stop from an unavailable import and a Clippy type-complexity stop. By count of PR runs in the listing, #21, #22 and #23 each took five to nine runs to reach a passing head.

The project lead chose GitHub-hosted Linux, with no company-laptop installation, on 21 September 2026 (see server `CONTRIBUTING.md`). This observation does not question that choice. It measures what the choice currently costs.

## 4. Size of the per-task records

- `Review/action-list.md` was created at `345397e` on 20 September 2026 with 14,577 characters. At `273002a` it is 196,134 bytes. Most of the growth is per-task implementation handoffs (tasks 1.1.2–1.4.6). AGENTS.md tells every session to read this file first.
- Server tree at `d0ef755`: 10,869 lines of Rust under `crates/*/src`; 11,097 lines under `crates/*/examples`, mostly 11 `*-proof.rs` programs of 676–1,407 lines each; 636 lines under `crates/*/tests`; 39 Python scripts (5,721 lines in `scripts/` including non-Python files); 11 `docs/*-tests.md` files.

## 5. How far the write path is shared

The trusted write/storage layer at `d0ef755` is System-specific: `SystemRecord` and `SystemRepository` in `storage.rs`; `CreateSystem`, `UpdateSystem`, `create_system`, `update_system` and `record_denied_system_create` in `application.rs`; `SystemRevision` in `revisions.rs`. This matches Roadmap §4 Phase 1 group 1.3 ("add family tables with their later capabilities rather than creating empty subsystems"). It is recorded only because Phase 2 is where more resource families arrive.

## 6. Test expectations traced to the standard

A search of `crates/*/examples`, `crates/*/tests`, `crates/*/src` and `scripts/` for `/req/`, `/conf/` and `/ats/` identifiers found only `/conf/api-common`. The server `docs/` cite RFC 9110, 9457, 3986, 8259, 9068, 8725, 7517 and 6750 for the HTTP and authentication work.

This may be expected. Most Phase 1 work so far (storage, value types, authentication, permissions) has no CSAPI requirement behind it. Guide §8.1.1 already requires each test to "state the controlling requirement, independently expected answer and a plausible wrong behavior". Whether that is being met for CSAPI-facing behavior starts to matter with #24.

## 7. External conformance suites and peer implementations

- **Botts ETS.** [`Botts-Innovative-Research/ets-ogcapi-connectedsystems10`](https://github.com/Botts-Innovative-Research/ets-ogcapi-connectedsystems10) was created 28 April 2026 and last pushed 4 August 2026. Its README describes a pre-beta TEAM Engine suite for Parts 1 and 2 (240 procedures) that is not an official CITE submission. [IDR-050](../../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-050-conformance-harness-strategy-report.md) concluded that no *official* public CSAPI suite existed at its evidence freeze (16 September 2026). It did not mention this unofficial suite.
  - The project lead states that this suite was written entirely by AI, by someone who is not a software developer.
  - **It is therefore not used as an independent test oracle in this review.** Two AI-derived readings of the same standard may share errors. Independently developed software is known to fail on the same inputs more often than chance (Knight and Leveson, IEEE TSE 1986), and LLM errors are correlated across models (["Correlated Errors in Large Language Models", ICML 2025](https://arxiv.org/abs/2506.07962)).
- **OpenSensorHub.** [`opensensorhub/osh-core`](https://github.com/opensensorhub/osh-core) was created 15 October 2015. Its top contributor, `alexrobin`, has 1,975 commits. The pinned CSAPI Part 1 source in the server corpus (`23-001r0.adoc`) names Alexandre Robin as editor. Riverside Research is listed among the submitting organisations. Most of this code predates AI coding assistants, but it is still not an oracle: the project's research documented places where OSH departs from the standard.
- **OS4CSAPI client.** The Guide §8.1 names the "pinned OS4CSAPI TypeScript client" as one external-client check. [`OS4CSAPI/ogc-client-CSAPI_2`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2) is a fork of `camptocamp/ogc-client`, created 31 January 2026. How much of its CSAPI code was written by AI was not established here.

## 8. Standards activity upstream

The [Connected Systems SWG repository](https://github.com/opengeospatial/ogcapi-connected-systems) closed several Part 3 decision issues on 24 September 2026 (#187, #188, #190–#195). Open items include #201 (Part 2 SystemEvent JSON mapping), #179 (observed/controlled property query parameters, targeting v1.1) and #185 (bulk POST). None of these was compared against Glaux's pinned interpretations.

## 9. Research on AI-to-AI review

Cross-model review helped unevenly in one study ([arXiv 2607.21656](https://arxiv.org/abs/2607.21656), competitive-programming tasks): a stronger reviewer improved weaker drafts, while the reverse lowered accuracy. Other work reports that same-session self-review tends to repeat its own errors ([arXiv 2603.12123](https://arxiv.org/abs/2603.12123)). This supports using model diversity and fresh context to *find* issues. It does not make any AI review a substitute for human-anchored evidence.

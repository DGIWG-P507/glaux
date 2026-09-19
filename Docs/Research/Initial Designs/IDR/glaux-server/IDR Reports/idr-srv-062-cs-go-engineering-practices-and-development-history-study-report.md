# Section 062: CS-GO Engineering Practices and Development History Study - Research Report

**Topic ID:** IDR-SRV-062<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-062 plan](../IDR%20Plans/idr-srv-062-cs-go-engineering-practices-and-development-history-study.md)<br>
**Overall Research Plan:** [Controlling overall IDR plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Q1–Q6; public engineering evidence assessed, private workflow and runtime results explicitly unresolved.<br>
**Methodology Used:** Pinned source walkthroughs, complete accessible public-history inventories, selected before/after cases, test-assertion analysis and comparison with existing Glaux planning.<br>
**Research Time:** Approximately 30 minutes elapsed through research/report review, 00:03–00:33 UTC September 19, 2026 (September 18 EDT), in one AI-assisted iteration; not a human-hours estimate and excluding subsequent publication. No upstream tests executed.<br>
**Primary Sources:** [Author's repository at the execution pin][tree], selected commits/PRs/issues/releases, published CSAPI Part 2 and pinned Part 3 draft.<br>
**Supporting Resources:** [014B implementation study][prior], [052 testing strategy][tdd], [053 fixtures][fixtures], [Goal v1.7][goal], [Guide v1.0][guide], [Roadmap v1.1][roadmap].<br>
**Document Purpose:** Identify practical engineering lessons and bounded planning implications before implementation-issue publication; not a new conformance audit or implementation authorization.<br>
**Author(s):** Glaux research workflow, AI-assisted<br>
**Accepted By:** Pending Glaux Project Lead review<br>
**Acceptance Date:** Pending<br>
**Date:** September 18, 2026<br>
**Last Updated:** September 18, 2026

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base
4. Findings by Research Question
5. Decision Analysis
6. Key Recommendations
7. Implementation Implications and Estimates
8. Risks, Constraints, and Open Questions
9. Validation Against Plan Success Criteria
10. Next Steps and Handoff
11. References
12. Appendices

---

## 1. Executive Summary

**CS-GO is a valuable engineering example, and this deeper review supports Glaux's current direction rather than a redesign.** Its most transferable practices are shared representation/query mechanisms, changes that carry through storage and HTTP behavior, small explanatory tests, practical contributor instructions, and explicit limits on experimental claims. The strongest lessons are illustrated by actual changes, not inferred from the developer's reputation.

The observation-storage change is particularly instructive: changing the database key is accompanied by code and tests that preserve the observation's public identity when its result time changes. Spatial-query repairs check which resources are included and excluded, not merely whether a request succeeds. Maintainer replies also reject an incorrect interpretation and avoid a proposed special-case fix where shared decoding already handles the problem. [Storage change][storagecommit], [spatial correction][spatialcommit], [issue 9][issue9], [issue 6][issue6].

**Did the author use test-driven development? The public record does not establish that.** It establishes substantial test development and code/test co-evolution. Combined commits do not reveal whether tests were written first. At the inspected revision there are 76 test files and 408 top-level test functions; these are source counts, not passing-test or coverage results. No tests were run in this study.

Most lessons are already covered by the Guide and its issue-sized Roadmap. Two small clarifications are worth discussing later: explicit test-harness cleanup/isolation expectations, and applying the existing documentation-update rule to future contributor/assistant guidance. Neither requires a new phase, framework or documentation system. Keep the Rust stack, explicit migrations, independent expected results, and durable transaction/publication design already selected.

This report leaves the Goal, Guide, Roadmap, synthesis and implementation issues unchanged. It is ready for project-lead review; unknown private practices, unexecuted tests and incomplete source-copying clearance are disclosed rather than turned into additional research gates.

## 2. Scope and Plan Alignment

This is the planned **IDR-SRV-062** follow-up to 014B. It studies how the author's server is engineered, not whether every endpoint conforms. The complete accessible default-branch history and public issue/PR/release inventories were screened; representative source paths and changes were examined in depth.

| Plan question | Coverage status | Evidence location |
|---|---|---|
| Q1 — Architecture and evolution | Complete for selected paths; no claim of exhaustive historical review | §§4.1, 12.1 |
| Q2 — Development process and tools | Public evidence assessed; private CI/review/debugging practice unresolved | §§3.3, 4.2, 8 |
| Q3 — Testing and TDD | Test design assessed; test-first sequencing and runtime success unresolved | §§4.3, 12.2 |
| Q4 — Comments, documentation and guidance | Complete for selected guidance and executable counterparts | §4.4 |
| Q5 — Choices and omissions | Classified by evidence; unsupported intent retained as unknown | §4.5 |
| Q6 — Transfer to Glaux | Complete; existing coverage and two proposed clarifications identified | §§4.6, 6–7 |

Out of scope: installing tools, contacting the developer, filing upstream findings, copying code/fixtures, changing Glaux scope or stack, executing load/security tests, reopening all accepted research, publishing implementation issues or implementing the server.

### Already covered versus newly examined

| Earlier coverage | What this study adds |
|---|---|
| 014B §§4, 8, 10: architecture, pagination and persistence | Before/after formatter and cursor cases; later observation partition-key/identity change |
| 014B §§11–12: documentation, distribution and audit corrections | Precise release-artifact comparison; contributor guidance; attribution and maintainer rejection/simplification examples |
| 014B §13: testing overview | Full current test-source inventory; assertion strength, isolation, independence and test-first limits |
| 050–053: Glaux verification, traceability, tests and fixtures | Concrete peer examples; 052's proposed Glaux TDD practice is not evidence of CS-GO TDD |
| 054/056: performance and interoperability planning | No new benchmark or executed interoperability result; no such evidence inferred from implementation changes |
| Supplements 058–061: particular standards/feature questions | Development methods and planning impact, not another feature-scope decision |

014B's release-based source inventory remains a valid dated assessment. Its 69 test files/384 named tests are not contradicted by the later main revision's 76/408 inventory. Neither inventory is an execution result. No accepted report is overwritten. [014B §§3, 13][prior], [052 §5.1][tdd], [053 §§3.3, 5.1][fixtures].

## 3. Evidence Base

### 3.1 Primary Sources Reviewed

All access dates below are **2026-09-18 EDT / 2026-09-19 UTC**.

| Source | Version / status | Authority class and anchors | Availability / limits |
|---|---|---|---|
| Author's CS-GO repository | main at `b1fd2e0e9bd69e222d05258d659a842ca24502cb`, September 2 UTC | Observed implementation; [tree][tree], pinned files throughout §4 | Non-truncated tree: 352 entries, 312 blobs; not every file deeply reviewed |
| Public history | 68 default-branch commits; 11 ordinary issues; 5 PRs; 7 issue comments | Observed history and stated author rationale; §12.1 identifies selected commits | Complete accessible inventories, selected diffs; private work/other branches not exhaustively examined |
| Releases | Three tags/releases; latest v1.0.4 at `244f4dd586da685d4d9b75e43f73001028b5bd0e` | [Published release][release], manifests and release tooling | Assets listed; checksum metadata compared; binaries not downloaded, independently hashed or executed |
| CSAPI Part 2 | Published 23-002 | Normative authority; [§§1–2][part2] distinguish scope and conformance obligations | Not a full conformance reassessment |
| Official Part 3 | `part3-working-draft@6f529a15bfa63259febc3620378d3e5a06305333` | Unapproved [working draft][part3]; [issue 192][issue192], [PR 198][pr198] | Bounded authority check, not proof of peer conformance |

Public inventories used GitHub pagination, not a first-page sample: commits were **30 + 30 + 8**; issue records were 16 including the five PRs. Other listed inventories fit one page with no next link. All 11 ordinary issues were closed, all five PRs merged, and all seven ordinary issue-discussion comments were owner replies. The issue authorship matters: all 11 were filed by `Sam-Bolling`, not by the upstream developer. §12.2 supplies reproducible endpoints; they remain mutable.

### 3.2 Supporting Sources Reviewed

| Source | Baseline / relevant anchors | Role and limitations |
|---|---|---|
| [014B][prior] and selected subsequent report passages | Existing accepted Glaux reports; comparison in §2 | Prior findings, not substituted for current source inspection |
| [Goal][goal], [Guide][guide], [Roadmap][roadmap] | v1.7 / v1.0 / v1.1 at Glaux `409afdd2ac946b6e9a6455c22fe709a39c59d7e4` | Controlling project scope and specific task comparison |
| [Standards-history register][register] | v1.16, Part 3 entries implicated by this case | Authority routing; no material change found, so not edited |
| 062 plan and report template | Published plan at `409afdd`; existing governance template | Research scope, completion and acceptance rules |

### 3.3 Evidence Quality Notes

Code and test assertions support claims about what is implemented or checked in source, not successful execution. Commit messages and owner replies support stated rationale, not independent proof of standards correctness. Recommendations below are Glaux judgments.

The public APIs returned zero Actions workflows, zero available workflow runs, and zero check-runs/statuses at HEAD. The tree contains no Actions workflow files. These observations do not prove the absence of private/local automation. PRs 15 and 16 have no public submitted reviews, inline-review comments or discussion comments in the inspected collections; their merges do not demonstrate independent review.

The Part 3 branch pin and issue/PR disposition matched the existing register. Issue 192 remained open; PR 198 remained merged into the draft branch, merge `c95c1d6003359d0883c4dc759d7a148ab115fdb1`. The draft still identifies itself as SWG draft and contains unfinished/commented-out material. No ordinary peer commit was added to the standards register. [Official draft][part3], [issue][issue192], [PR][pr198].

## 4. Findings by Research Question

### 4.1 Q1 — Architecture and evolution

**Source-backed finding:** CS-GO uses an explicit composition root with focused shared mechanisms, while retaining resource-specific repositories and mappings.

Startup assembles configuration, database migrations, typed repositories, optional MQTT, publication/ingestion and HTTP. The router constructs representation collections and attaches shared request validation. This is understandable wiring, not evidence that every dependency is isolated behind an interface. Some formatters read associated resources through repositories. [Startup][main], [router][router].

Raw representation validation occurs before typed decoding; a reusable formatter collection handles representation selection and resource-specific serializers. Observation writes then validate the result against its parent stream's schema before persistence. This separation is useful even though Glaux's exact media negotiation and schema policies must come from its own standards interpretation. [Formatter collection][formatter], [contract validator][validator], [observation handler][observationhandler].

Repository transactions preserve related state: observation changes recompute stream time extents; command-status creation also updates parent command state; System cascade deletion traverses dependent resources transactionally. These sampled paths show where integrity work belongs, not exhaustive proof of concurrency or rollback correctness. [Observation repository][observation], [command repository][commands], [System repository][systems].

| Selected evolution | Evidenced change and rationale | Engineering lesson and limit |
|---|---|---|
| [Early formatter refactor][refactor] | Author describes consolidating serialization/deserialization for multiple representations; handlers delegate wire conversion | Share protocol mechanics without erasing resource-specific mapping. No test files changed in this commit; do not call it verified |
| [Cursor pagination change][cursorcommit] | Shared deterministic tuples, time/ID tie-breakers, extra-row detection and reverse traversal replace offsets; handlers/docs/tests change together | Follow returned links as a client and preserve query context. Selected HTTP test traverses first→second→first, not every page; no measured speedup |
| [Spatial correction][spatialcommit] | Ineffective selectors repaired through parsing, PostGIS, SRID normalization, repositories and HTTP tests | Fix the whole path and test included/excluded identities. Six-coordinate parsing is not proof of vertical filtering |
| [Observation storage change][storagecommit] | Composite `(id,result_time)` key; changed result time uses transactional delete/insert preserving public ID and lifecycle fields; tests check the resulting single row | Storage design must preserve API semantics. Author's better-fit rationale is not a benchmark; the old database is not automatically migrated |

Current cursor parsing binds route/filter context and checks structured fields; the token is encoded JSON with a query-context hash, not a secret-authenticated signature. Reuse its deterministic-order concept, not an assumption that its cursor implements every Glaux authorization/tamper requirement. [Cursor parsing][queryparams], [repository ordering][pagination].

Publication uses a narrow transport interface; HTTP handling emits after success, and MQTT ingestion reuses result validation with topic/envelope checks. This is useful separation of transport from message handling. It is not a durable outgoing-work log or crash-recovery proof. [Publisher][publisher], [HTTP publication][httpevents], [MQTT ingestion][ingestion].

**Interpretation:** Carry each change through its public behavior, persistence, tests and explanations. For Rust, retain the Guide's domain/standards/server packages, explicit application transactions and SQLx repositories. Do not reproduce GORM, repository-coupled formatters or startup schema mutation merely because they work as peer design choices.

### 4.2 Q2 — Development process and tools

**Source-backed finding:** Public records show coordinated changes, explicit maintainer judgment and practical tooling; they reveal less about private workflow.

[Issue 10][issue10] attributes its diagnosis to OS4CSAPI fork research. The upstream owner authored/merged [PR 15][pr15]; its [implementation commit][linkcommit] adds Command/Observation formatters and shared link normalization, with formatter tests and HTTP checks across four resource families. This is contributed investigation followed by upstream implementation, not evidence that the author's own process was the fork's audit process.

Two responses are especially useful: the maintainer rejects the proposed `updatable` interpretation in [issue 9][issue9], and explains in [issue 6][issue6] that shared decoding already handles the reported input problem without the proposed dedicated fix. Their value here is evidence of technical adjudication and simplification; this report does not turn either comment into normative authority.

[PR 16][pr16] explicitly adopts unfinished Part 3 experimentally. A [later refinement][pubsubcommit] changes topics/message classes, independent switches and discovery together, with tests rejecting obsolete forms and avoiding overstated formal conformance. This is a useful feature-evolution pattern, not approval of the draft or proof that broker restrictions are enforced.

| Tooling surface | What is present in source | What is not established |
|---|---|---|
| Runtime/build dependencies | Go module, Chi, GORM/Postgres, MQTT and JSON Schema dependencies | Reproducible build or suitability for Glaux's Rust stack |
| Testing/debugging | Go tests, Testify/Testcontainers, coverage/lint targets, focused test and SQL-log guidance | Passing tests, measured coverage, author frequency of use |
| API generation | `make swag`, conversion commands and generated OpenAPI served by router | Generated description's complete agreement with behavior |
| Packaging | Docker/Compose, cross-platform release targets and attached release assets | Clean deployment, all-platform execution or reproducible binary production |

Sources: [module][gomod], [Makefile][make], [contributor instructions][copilot], [router][router], [Dockerfile][docker], [Compose][compose].

The [release-tooling commit][releasecommit] configures a release target dependent on tests and five-platform builds. Configuration is not evidence that every publication used that command. Also, `docker-build` includes a push, and `release` tags/pushes/publishes; neither is a harmless local diagnostic.

A precise release distinction avoids a false finding: the [manifest tracked at v1.0.4][trackedmanifest] names v1.0.3 files, but the [attached v1.0.4 manifest][attachedmanifest] names the actual release assets and matches GitHub's reported asset digests. The [next commit][manifestfix] fixes the tracked filenames. This is source-tree/release-artifact drift, **not evidence that downloadable checksums were broken**. No binaries were independently hashed or executed.

**Interpretation:** Use ordinary issues, review, tests, source comments and release checks. Do not infer a special hidden methodology or prescribe the author's tools. Distinguish declared commands from recorded execution and validate the actual deliverable, not only its source tag.

### 4.3 Q3 — Test craftsmanship and TDD

**Source-backed inventory:** All 76 `_test.go` files received declaration/pattern inventory; representative bodies and helpers received deeper review.

| Layer | Files |
|---|---:|
| Real-HTTP E2E | 21 |
| Formatters | 23 |
| Other model/parser tests | 7 |
| Repository | 10 |
| API/config/contract/resource validation | 8 |
| MQTT/publication | 5 |
| Seeder | 2 |

There are 408 top-level `Test*` functions excluding `TestMain`; subtests are not counted separately. No literal `t.Parallel()`, `Benchmark*` or `Fuzz*` declarations were found in these files. This does not establish absence of external benchmarking or privately run tests. [Pinned tree][tree].

#### What selected assertions actually establish

| Example | Setup and meaningful assertion | What a plausible wrong implementation could still do |
|---|---|---|
| System lifecycle | HTTP create→replace→read→delete→404; checks independently specified changed name | Mishandle another replacement field or associated rows not asserted. [System test, `TestSystemCRUD_CreateReplaceDelete`][systemtest] |
| Rejected Procedure replacement | Valid SensorML followed by rejected representation, error-path check and read-back of original label | Change unasserted fields/relationships/publication state. It proves the checked label, not complete non-mutation. [Contract test, lines 85–126][contracttest] |
| Spatial selection and repair | Distant inside/outside fixtures; exact membership; repository count/ID; deliberate zero-SRID precondition repaired and queried | Rectangular/equivalent combined inputs do not distinguish envelope-only filtering or AND versus OR. [HTTP cases][spatialtest], [repository case][spatialrepo], [repair case][spatialrepair] |
| Result-time range | 2025/2027 observations; query 2025 and assert one item | Return the wrong single observation: ID/time is not checked. Adjacent latest test does check identity. [Observation tests, lines 464–539][obstest] |
| Link normalization | Relative/absolute/nil/empty cases and 11 HTTP link sites in PR 15 | Some checks only require an HTTP prefix, so a wrong absolute destination can pass; exact destination remains important. [PR 15 implementation][linkcommit] |
| Parser and error detail | Literal bounds, malformed inputs and typed/indexed error paths | Pass the implementation's selected policy even if that policy misreads the standard. [Spatial parser][spatialparser], [decode tests][decodetest] |
| Representation meaning | Raw IO round trip plus separate exact SensorML coordinates and associated title/UID expectations | Raw preservation alone can pass without understanding content; these are complementary tests. [IO][iotest], [position][positiontest], [associations][associatedtest] |
| Publication and ingestion | Injected time/event IDs, captured topics/envelopes, disconnected transport and fake-store create/update counters | Fail at broker delivery, SQL rollback or crash recovery; fake transport is not those proofs. [Publisher tests][publishertest], [ingestion tests][ingestiontest] |
| HTTP publication ordering | Fake resolver/recorder checks before-delete, after-update and no publication on HTTP 400 | Pass without a real database transaction. Calling it post-commit does not supply commit evidence. [Middleware tests][eventtest] |
| User-facing seeder | Fixed random seed/topology, real API/router/DB, stored counts and selected relationships | Agree with a shared server validator's mistake; observer send count does not verify payload meaning. [Seeder E2E][seede2e], [unit tests][seedtest] |

The E2E harness starts a TimescaleDB/PostGIS container, runs production migrations and uses a real HTTP listener around the router. It does **not** launch the shipped executable or exercise its full startup/deployment path. Some observation fixtures insert parents through repositories, so those cases do not prove public stream creation. [Setup][setup], [observation fixture helper][obstest].

Isolation is mixed. E2E tests share a package database and truncate it between tests without checking the reset error. Sampled repository helpers use separate containers; the newer seeder test registers cleanup promptly and checks closure errors. Mutable image tags, a short fixed readiness sleep and some real-clock inputs limit repeatability. These are identified risks, not observed flaky runs. [E2E cleanup][setup], [database helper][postgis], [seeder lifecycle][seede2e].

Some schema helpers skip when schema compilation fails. A green command could therefore leave schema assertions unexecuted; skips must remain visible, not become conformance passes. [System schema checks, lines 180–186][systemtest]. E2E response-schema checks reuse the production structural validator and bundle; seeder generated-value checks reuse the production observation-value validator. These are separate consistency checks, not independent standards oracles. [E2E schema helper][schemahelper], [seeder tests][seedtest].

#### TDD conclusion

The examined histories show tests added, refactored and maintained, often alongside production fixes. None of the selected cases exposes an evidenced failing-test-before-fix sequence, and no inspected maintainer statement establishes a consistent test-first practice. Squashing, local work and missing execution records leave that order unknown.

**Interpretation:** Learn the test craftsmanship without inventing a TDD story. For Glaux, write independently expected behavior, observe the relevant failure where practical, implement the smallest change, and retain regression checks under the existing Guide/issue workflow. No new test framework or historical proof requirement is proposed.

### 4.4 Q4 — Comments, documentation and coding guidance

**Source-backed finding:** The most useful guidance explains where to work and why an invariant exists; it must still be checked against current code.

The [Copilot instructions][copilot] identify extension points, focused tests, SQL debugging and small operation-specific E2E cases. The [repository reference entry][claude] routes particular questions to narrow materials and distinguishes project interpretation from normative sources. [Ownership notes][ownership] explain structural parentage versus client-authored semantics and server-derived state; [schema lookup][schemalookup] explains curated fragments and their sources. This is valuable design knowledge without a new management framework.

Checked-in guidance is not proof of its use in every session. The [April 5 commit][skillcommit] explicitly adds reference material; five of the 68 commit messages include Copilot co-author trailers. Neither fact establishes private prompts, model choices, proportions of AI authorship or independent review.

Specific discrepancies illustrate maintenance risk:

- Instructions describe `make migrate` as working, but the target is a placeholder; startup actually invokes migrations.
- Instructions call OpenAPI generation unfinished although a generation target and served generated document exist.
- README prerequisites say Go 1.24+, while the module requires 1.25.0.
- Some guidance references root `schemas/...` paths; current bundles are under E2E and contract-validation directories.
- `IMPLEMENTATION.md` retains scaffold/future-work descriptions behind the current implementation.

Sources: [instructions][copilot], [Makefile][make], [startup][main], [router][router], [README][readme], [module][gomod], [schema lookup][schemalookup], [implementation notes][implementation].

The [Codesight context map][codesight] is dated May 2. Its [coverage output][coverage] reports 36 test files and heuristic route/model coverage, with apparent header/query tokens treated as routes. Its 38% is **not measured Go statement coverage** and cannot establish passing tests or proven productivity improvement.

By contrast, migration comments explain database-extension and partition-key constraints, while the [seeder README][seedreadme] explains additive runs, returned-ID relationships and limits of protobuf-shaped schemas. Fixed randomness alone does not reproduce full identities when the default run ID changes. [Migration comments][migration], [README storage boundary][readme].

**Interpretation:** Preserve short, useful explanations near code and link to controlling sources. Extend the existing documentation-update expectation to any future assistant guidance; do not mandate a particular assistant, generated context map or extra document set.

### 4.5 Q5 — Scope choices and omissions

| Observation | Classification / supported rationale | Glaux consequence |
|---|---|---|
| Typed repositories with selected shared query/representation machinery | Observed simplification; early formatter commit explicitly motivates representation reuse | Retain useful shared primitives and resource-specific SQL, not a generic abstraction for every operation |
| Experimental MQTT with qualified discovery | Explicit experimental choice and broker-responsibility limit, not approved conformance | Preserve selected experiment and truthful claims; no expansion of scope |
| Fresh Timescale development volume | Explicit compatibility boundary: old volume preserved, not migrated; legacy-key test expects actionable refusal | Retain Glaux's explicit migrations/recovery requirements |
| API-level seeder | Practical example generation through API, with additive/reproducibility controls | Existing sample/test workflows can use this idea; no new simulator requirement |
| No authentication middleware in inspected router registration | Source-path observation; off-repository deployment controls and intent unknown | Do not remove Glaux authentication/authorization tasks or infer a complete security assessment |
| No public Actions records or benchmark declarations found | Public evidence gap, not proof of absent private practice | Keep normal CI/performance work already planned |
| MIT mentioned in implementation notes, no LICENSE/COPYING/NOTICE file in pinned tree | Copying clearance incomplete; do not label the repository definitively unlicensed | Learn concepts; verify applicable notices/terms before copying source or fixtures |

Sources: [formatter history][refactor], [PR 16][pr16], [storage change][storagecommit], [Compose][compose], [seeder documentation][seedreadme], [router][router], [tree][tree], [license statement][implementation]. This study copied no upstream implementation or fixture bodies.

### 4.6 Q6 — Transfer to the existing Rust plan

The following are **project recommendations**, not peer-derived standards obligations. All task numbers refer to [Roadmap v1.1][roadmap]; Guide references are to [v1.0][guide].

| Lesson and Rust adaptation | Existing coverage | Disposition and verification |
|---|---|---|
| Explicit composition; pure domain and wire mapping; resource-specific storage | Guide §§2.2–2.4; **1.1.2, 1.2.6, 2.2.10** | Already covered. Check package dependencies and independently expected representation meaning |
| Shared validation with useful field errors before persistence | Guide §§4.3, 6.4, 8.2; **1.2.2, 1.2.6, 1.5.1** | Already covered. Invalid writes must leave stored state and outgoing work unchanged |
| Public identity survives storage changes; related state changes atomically | Guide §§2.3, 4.7, 8.2; **1.3.3, 2.4.1, 2.4.2, 3.2.4** | Already covered. Inject failures and verify complete rollback; keep explicit migrations, not GORM/Timescale adoption |
| Exact query membership, context-preserving links and independent geometry counterexamples | Guide §§4.4, 6.3, 8; **2.5.7, 2.5.8, 2.5.10** | Already covered, stronger than several sampled assertions. Check exact IDs and impossible/wrong alternatives |
| Link tests compare the intended destination, not only a prefix | Guide §§4.1, 8.1; **1.4.2, 2.3.8** | Already covered. Use fixed expected canonical URLs and follow links independently |
| Combine small pure tests with real DB/HTTP and separately tested deployment | Guide §§8.1–8.2; **1.1.3, 1.5.1, 9.2.1–9.2.3** | Already covered. Do not call an in-process router test a shipped-binary test |
| Predictable harness lifecycle and failure diagnosis | Guide §8.1; **1.1.3** | **Bounded clarification proposed:** prompt cleanup registration, checked reset failures, isolated mutable fixtures, recorded seeds/clocks/dependency versions |
| Fake transport for unit logic plus actual durable delivery/fault tests | Guide §§4.8, 8.2; **1.3.3, 6.3.1, 6.3.8** | Already covered. Broker acknowledgement, database commit and recipient processing remain distinct |
| Documentation/discovery and release evidence agree with executable behavior | Guide §§4.12, 7.2–7.3, 10; **1.1.4, 1.4.6, 9.1.9, 9.4.2** | Already covered. Keep skips visible; verify candidate commands, capabilities and artifacts |
| Helpful contributor knowledge without stale parallel authority | Roadmap §5.2 documentation rule; Guide §13; **1.1.4, 9.4.2** | **Bounded clarification proposed:** any future contributor/assistant notes follow the same update rule and link to sources/current examples |
| Safe sample loading, dependency/license checks and measured optimization | Guide §§4.7, 4.12, 8.3; **1.2.1, 8.3.1, 9.2.1, 9.2.2, 9.3.1, 9.3.7** | Already covered. No automatic code reuse, new seeder subsystem or partitioning requirement |

## 5. Decision Analysis

| Option | Benefits | Costs / compatibility risks | Recommendation |
|---|---|---|---|
| Keep current plan and use concrete lessons in its existing tasks | Preserves standards/Rust design; makes tests and explanations more effective | Requires reading evidence when implementing each relevant issue | **Recommended**, with discussion of the two small clarifications |
| Copy CS-GO architecture/toolchain more directly | Familiar peer patterns | Go-specific coupling; startup migration, durability and authority differences; license verification unresolved | Not recommended as a default; conceptual adaptation only |
| Add a broad engineering framework or reopen implementation design | More process artifacts | Duplicates existing Guide/Roadmap; no demonstrated decision need | Reject |
| Wait for maintainer interview or full upstream test execution | Could clarify TDD and runtime behavior | Does not answer most planning questions better; requires separate access/authority | Optional follow-up only if a concrete decision requires it |

## 6. Key Recommendations

1. **Retain the current Goal, Rust stack and issue-sized Roadmap.** High priority; the study identifies reinforcement, not a reason to redesign or adopt Timescale. No new scope decision follows from report acceptance.
2. **Use the concrete test examples within existing tasks.** High priority; exact expected IDs, values and links, full invalid-write non-mutation and real commit/delivery checks make failures meaningful. Preconditions are the existing authorized implementation issues and available test prerequisites.
3. **Discuss the two small wording clarifications in §4.6.** Low incremental complexity; strengthen harness lifecycle and contributor-guidance maintenance inside existing sections/tasks. Do not add new deliverables or issues solely for these clarifications.
4. **Keep evidence labels honest.** High priority; unknown TDD, source-only tests, generated coverage estimates and absent public automation must not become assertions about the developer or runtime quality.
5. **Verify reuse terms only if direct copying is proposed.** No source/fixture reuse is currently required. Independently implement Rust behavior from authoritative standards and the accepted Glaux design.

## 7. Implementation Implications and Estimates

### 7.1 Implications

The comparison identifies no additional capability group, implementation phase or required leaf task. The 286-task baseline remains unchanged. Examples can inform test design and completion evidence for existing issues; they are not requirements to copy every peer helper, tool or abstraction.

The proposed harness clarification is an implementation detail of **1.1.3**. The guidance clarification belongs to existing development/documentation rules, not a new AI-specific workflow. Part 5 remains deferred; existing draft experiments and bounded filtering remain as approved.

### 7.2 Effort / Complexity Estimate

| Work item | Relative complexity | Estimate / assumptions |
|---|---|---|
| Accepted-findings synthesis addendum | Low | One bounded documentation iteration is expected; separately authorized, not yet performed |
| Discuss and, if approved, integrate two clarifications | Low | Small edits to existing prose/task completion expectations; no new task count inferred |
| Apply examples during implementation | Within existing work | No separate hours estimate supported; calibrate through actual issue execution |
| Optional upstream runtime/TDD/license follow-up | Unknown until needed | Not a general completion gate; no installation, outreach or copied material assumed |

These are planning judgments, not measured implementation effort or promises about test/runtime performance.

## 8. Risks, Constraints, and Open Questions

### 8.1 Risks and Constraints

- **Execution unavailable:** `go`, `docker`, `psql`, `cargo` and `rustc` were not found on this shell's PATH. That does not prove they are uninstalled. No permitted isolated runtime was established; no installations, test runs or external service writes occurred.
- **Sampling:** Full inventories do not mean every historical diff, fixture, branch, dependency or deployment was reviewed. Selected assertions support bounded conclusions only.
- **Private practice:** Local tests, private CI/reviews and writing order remain unknown. Owner-linked commits/co-author trailers cannot allocate authorship effort.
- **Standards authority:** Peer tests may faithfully test a mistaken interpretation. Official requirements and accepted project interpretations still control Glaux.
- **Copying and operational differences:** Applicable reuse terms are unresolved; Go/ORM/startup migrations and peer deployment choices do not override Rust, security or durability requirements.
- **Scope drift:** Additional frameworks, mandatory new tooling and generalized upstream defect hunting would exceed the study's decision need.

### 8.2 Open Questions

Optional maintainer questions, **not sent and not completion gates**:

1. Does the author usually write a failing test first, and are there public examples that preserve that sequence?
2. What private/local checks, review practices and debugging tools supplement the public repository?
3. What measured workloads, if any, informed keyset/Timescale choices, and what database-upgrade policy is intended?
4. Which license/notice terms govern direct reuse of repository source and separately sourced fixtures?

Remaining implementation proof belongs to Glaux's existing tasks: build/run with recorded prerequisites, real transactional failures, actual broker faults, exact independent responses, migrations/restores and candidate artifacts. This source study supplies no replacement runtime evidence.

## 9. Validation Against Plan Success Criteria

| Plan criterion | Status | Evidence |
|---|---|---|
| Q1–Q6 answered or consequential limits explicit | Met | §§2, 4, 8 |
| Pins, inventories, selection, gaps and contribution attribution visible | Met | §§3, 4.2, 12 |
| Representative evolution includes successful practices and simplifications | Met | Formatter/cursor/spatial/storage cases; maintainer adjudication |
| Test assertions, fixtures, isolation, independence and execution separated | Met | §4.3; no pass-count claim |
| TDD assessed from sequencing, not test presence | Met with explicit unknown | §4.3 |
| Tools/comments/guidance checked against code; private practice not invented | Met | §§4.2, 4.4 |
| Omissions classified and consequential standards checks qualified | Met | §§3.3, 4.5 |
| Lessons map to existing Guide/tasks with justified dispositions | Met | §4.6 |
| Existing template, optional questions and bounded handoff retained | Met | §§5–10; no new scope adopted |

Research and report preparation are complete. **Project-lead acceptance remains pending.**

## 10. Next Steps and Handoff

1. **Project Lead:** Review this report. Under the established workflow, the next `proceed` accepts it and authorizes the separate synthesis addendum. No special acceptance phrase is needed.
2. **Research assistant, next authorized iteration:** Add the accepted findings to the existing final synthesis and update acceptance records. Preserve older dated findings and the current project scope.
3. **Project Lead and assistant, following discussion:** Decide whether to make the two small Guide/Roadmap clarifications. Then resume publication of the complete implementation-issue set and verify its links/dependencies before coding.
4. **Implementation workflow thereafter:** One ready issue per authorized iteration. Do not treat this study as authorization to start implementation now.

No calendar deadline is inferred for user-controlled acceptance or later iterations.

## 11. References

Source links next to findings identify the relevant files, commits, tests or discussions. Every CS-GO file link is pinned to the full execution SHA unless explicitly labeled as the earlier release tag. Commit cases and inventory endpoints are collected in §12 for reproducibility.

Controlling local documents: [research plan](../IDR%20Plans/idr-srv-062-cs-go-engineering-practices-and-development-history-study.md), [overall plan](../IDR%20Plans/overall-idr-research-plan.md), [report template](../../../../../Governance/research-report-template.md), [014B][prior], [052][tdd], [053][fixtures], [Goal][goal], [Guide][guide], [Roadmap][roadmap], [standards-history register][register].

Standards authority: [published CSAPI Part 2][part2], [pinned official Part 3 draft][part3], [official issue 192][issue192] and [official PR 198][pr198]. Implementation sources are informative, including their schemas, tests and interpretation notes.

## 12. Appendices

### 12.1 Selected history and coverage

Cases were selected to cover architecture, successful client workflow, contributed fixes, maintainer adjudication, experimental scope, regressions, storage compatibility and release practice.

| Case | Stable source | Depth / purpose |
|---|---|---|
| Representation refactor | [53d5ba8][refactor] | Before/after boundaries and stated reuse motivation |
| Cursor pagination | [e50d800][cursorcommit] | Shared ordering, changed API/docs/tests and forward/back traversal |
| Link normalization | [Issue 10][issue10], [PR 15][pr15], [d8f0b51][linkcommit] | Attribution, implementation, assertions and public review collections |
| Maintainer judgments | [Issue 9 reply][issue9], [issue 6 reply][issue6] | Rejected interpretation and shared-mechanism simplification |
| Experimental publication | [PR 16][pr16], [913c112][pubsubcommit] | Initial adoption, refinement, tests/discovery and explicit limits |
| Spatial regression | [ddc9b84][spatialcommit] | Request→database→response correction and counterexamples |
| Observation storage | [b1fd2e0][storagecommit] | Key/identity/transaction change, schema and HTTP tests, upgrade boundary |
| Release practice | [e7f1f2e][releasecommit], [v1.0.4][release], [4a00aa6][manifestfix] | Declared tooling versus attached artifacts and later correction |
| Assistant knowledge | [5b5fb94][skillcommit], pinned guidance | Documented addition, current advice and drift |

Deep source paths covered startup/router; representations/raw and stream-result validation; observation, command/status and System-deletion transactions; shared pagination/spatial handling; publication/ingestion; selected tests/helpers; seeder; guidance/build/release configuration. Other resource implementations, all examples/generated schema contents, dependencies and unrelated historical diffs received inventory or contextual review only. The audit fork was not re-audited; selected upstream attribution and accepted prior research supplied contribution context.

### 12.2 Reproduction and execution record

Public GitHub API inventories, retrieved on the access date in §3:

- [Default-branch commits, page 1](https://api.github.com/repos/SomethingCreativeStudios/connected-systems-go/commits?per_page=30&page=1), following pagination through page 3.
- [All issues including PRs](https://api.github.com/repos/SomethingCreativeStudios/connected-systems-go/issues?state=all&per_page=30&page=1), [PR inventory](https://api.github.com/repos/SomethingCreativeStudios/connected-systems-go/pulls?state=all&per_page=30&page=1), [issue comments](https://api.github.com/repos/SomethingCreativeStudios/connected-systems-go/issues/comments?per_page=100).
- [Tags](https://api.github.com/repos/SomethingCreativeStudios/connected-systems-go/tags?per_page=100), [releases](https://api.github.com/repos/SomethingCreativeStudios/connected-systems-go/releases?per_page=30&page=1), [recursive tree](https://api.github.com/repos/SomethingCreativeStudios/connected-systems-go/git/trees/b1fd2e0e9bd69e222d05258d659a842ca24502cb?recursive=1).
- [Workflows](https://api.github.com/repos/SomethingCreativeStudios/connected-systems-go/actions/workflows?per_page=100), [available runs](https://api.github.com/repos/SomethingCreativeStudios/connected-systems-go/actions/runs?per_page=100), [HEAD check-runs](https://api.github.com/repos/SomethingCreativeStudios/connected-systems-go/commits/b1fd2e0e9bd69e222d05258d659a842ca24502cb/check-runs), [HEAD statuses](https://api.github.com/repos/SomethingCreativeStudios/connected-systems-go/commits/b1fd2e0e9bd69e222d05258d659a842ca24502cb/status).

Source-count method: enumerate non-truncated tree paths ending `_test.go`; fetch all 76; count top-level `func Test...` declarations excluding `TestMain`; separately inspect benchmark/fuzz declarations and literal parallel calls. Counts do not incorporate dynamic subtests or prove execution.

Local checks used read-only file/working-tree inspection and `Get-Command` availability checks. Git was available; the five runtime/database/container commands listed in §8.1 were not on PATH. Upstream test/build/release/installation commands were **not run**. Publication of this Glaux report is separate from execution of upstream software.

## Report Completion Checklist

- [x] Topic ID matches overall research plan index
- [x] Topic research plan is linked and aligned
- [x] Core research questions are covered or explicitly unresolved
- [x] Findings are evidence-backed with reproducible references
- [x] Normative and informative evidence are classified and not conflated
- [x] Mutable sources identify a version, release, tag, commit, or dated retrieval
- [x] Controlled, inaccessible, missing, or ambiguous evidence limitations are explicit
- [x] Source-backed findings, analyst inference, and project recommendations are distinguishable
- [x] Conflicts with accepted prior reports are reconciled or explicitly escalated
- [x] Executive summary is independently readable by the project lead, implementers, and later AI agents
- [x] Recommendations are explicit and actionable
- [x] Risks and open questions are documented
- [x] Success criteria validation is complete
- [ ] Plan-owner acceptance and acceptance date are recorded before downstream acceptance
- [x] Next steps are assigned

[tree]: https://github.com/SomethingCreativeStudios/connected-systems-go/tree/b1fd2e0e9bd69e222d05258d659a842ca24502cb
[main]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/cmd/server/main.go
[router]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/api/router.go
[formatter]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/formaters/formatter.go
[validator]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/contractvalidation/validator.go
[observation]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/repository/observation_repository.go
[observationhandler]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/api/observation_handler.go
[migration]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/repository/repository.go
[commands]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/repository/command_repository.go
[systems]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/repository/system_repository.go
[pagination]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/repository/pagination.go
[queryparams]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/query_params/query_params.go
[publisher]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/pubsub/publisher.go
[httpevents]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/pubsub/http_events.go
[ingestion]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/mqtt/ingestion.go
[setup]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/e2e/setup_test.go
[schemahelper]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/e2e/schema_validator.go
[systemtest]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/e2e/systems_test.go
[obstest]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/e2e/observations_test.go
[contracttest]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/e2e/request_contract_validation_test.go
[spatialtest]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/e2e/spatial_queries_test.go
[spatialrepo]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/repository/system_subsystem_spatial_test.go
[spatialrepair]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/repository/spatial_filters_test.go
[spatialparser]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/query_params/spatial_query_params_test.go
[decodetest]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/common_shared/decode_test.go
[iotest]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/common_shared/io_test.go
[positiontest]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/formaters/sensorml_formatters/spatial_position_test.go
[associatedtest]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/formaters/sensorml_formatters/system_sensorml_integration_test.go
[publishertest]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/pubsub/publisher_test.go
[ingestiontest]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/mqtt/ingestion_test.go
[eventtest]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/pubsub/http_events_test.go
[postgis]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/repository/testutil/postgis.go
[seedreadme]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/scripts/seed-connected-systems/README.md
[seedtest]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/scripts/seed-connected-systems/seed_test.go
[seede2e]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/scripts/seed-connected-systems/seed_e2e_test.go
[copilot]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/.github/copilot-instructions.md
[claude]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/.claude/skills/ogc-connected-systems-reference/SKILL.md
[ownership]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/.claude/skills/ogc-connected-systems-reference/references/ownership-rules.md
[schemalookup]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/.claude/skills/ogc-connected-systems-reference/references/schema-lookup.md
[codesight]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/.codesight/CODESIGHT.md
[coverage]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/.codesight/coverage.md
[make]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/Makefile
[gomod]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/go.mod
[readme]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/README.md
[implementation]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/IMPLEMENTATION.md
[docker]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/Dockerfile
[compose]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/docker-compose.yml
[refactor]: https://github.com/SomethingCreativeStudios/connected-systems-go/commit/53d5ba8e6665d91fd4598b69cf2a00963e0f50b3
[cursorcommit]: https://github.com/SomethingCreativeStudios/connected-systems-go/commit/e50d8006db6e4744afdd84ded3270c116c07ca8e
[linkcommit]: https://github.com/SomethingCreativeStudios/connected-systems-go/commit/d8f0b513865f9c5d7afee66ed35d03bdc89a4ee3
[issue10]: https://github.com/SomethingCreativeStudios/connected-systems-go/issues/10
[pr15]: https://github.com/SomethingCreativeStudios/connected-systems-go/pull/15
[pr16]: https://github.com/SomethingCreativeStudios/connected-systems-go/pull/16
[issue9]: https://github.com/SomethingCreativeStudios/connected-systems-go/issues/9#issuecomment-4411516004
[issue6]: https://github.com/SomethingCreativeStudios/connected-systems-go/issues/6#issuecomment-4466083321
[pubsubcommit]: https://github.com/SomethingCreativeStudios/connected-systems-go/commit/913c112f07a425d33fe8508e52ce1cf162e9afb1
[spatialcommit]: https://github.com/SomethingCreativeStudios/connected-systems-go/commit/ddc9b84b2b859731b7e6c16c79f7785036ce2dad
[storagecommit]: https://github.com/SomethingCreativeStudios/connected-systems-go/commit/b1fd2e0e9bd69e222d05258d659a842ca24502cb
[releasecommit]: https://github.com/SomethingCreativeStudios/connected-systems-go/commit/e7f1f2e47e970babca1a5cc33439b6be49255efe
[release]: https://github.com/SomethingCreativeStudios/connected-systems-go/releases/tag/v1.0.4
[trackedmanifest]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/244f4dd586da685d4d9b75e43f73001028b5bd0e/dist/checksums.txt
[attachedmanifest]: https://github.com/SomethingCreativeStudios/connected-systems-go/releases/download/v1.0.4/checksums.txt
[manifestfix]: https://github.com/SomethingCreativeStudios/connected-systems-go/commit/4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952
[skillcommit]: https://github.com/SomethingCreativeStudios/connected-systems-go/commit/5b5fb9402d49c4c582edadb5153f72bc3fd48d7d
[part2]: https://docs.ogc.org/is/23-002/23-002.html
[part3]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/6f529a15bfa63259febc3620378d3e5a06305333/api/part3/standard/23-003r0.adoc
[issue192]: https://github.com/opengeospatial/ogcapi-connected-systems/issues/192
[pr198]: https://github.com/opengeospatial/ogcapi-connected-systems/pull/198
[guide]: ../../../../../Plans/glaux-server/glaux-server-implementation-guide.md
[roadmap]: ../../../../../Plans/glaux-server/glaux-server-roadmap.md
[goal]: ../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md
[prior]: idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md
[tdd]: idr-srv-052-rust-test-driven-architecture-and-multi-layer-test-strategy-report.md
[fixtures]: idr-srv-053-test-data-fixtures-golden-files-and-scenario-corpus-strategy-report.md
[register]: ../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md

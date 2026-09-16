# Section 052: Rust Test-Driven Architecture and Multi-Layer Test Strategy - Research Report

**Topic ID:** IDR-SRV-052<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-052 Rust Test-Driven Architecture and Multi-Layer Test Strategy](../IDR%20Plans/idr-srv-052-rust-test-driven-architecture-and-multi-layer-test-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** obligation-first TDD; test-layer ownership; workspace and test-support layout; Rust runner, assertion, database, property, snapshot, fuzz, coverage, mutation and quality tooling; CI tiers and evidence; domain, validation, API, persistence, async, streaming, command, security, DDIL and synchronization testing; traceability, conformance and downstream handoffs<br>
**Methodology Used:** accepted-requirement extraction; current primary-tool documentation review; layer/boundary analysis; test-double versus real-dependency analysis; deterministic execution and evidence modeling; functional-area mapping; implementation/community lesson reconciliation; CI and downstream synthesis<br>
**Research Time:** Approximately 47 hours of AI-assisted execution on September 16, 2026<br>
**Accepted Platform Baseline:** IDR-SRV-044 and IDR-SRV-045 Rust stable/edition 2024 modular-monolith candidate with Axum/Tokio/Tower, SQLx/PostgreSQL/PostGIS, explicit ports/adapters, `glaux-test-support`, `xtask`, first-party unsafe prohibition and pinned-tool CI<br>
**Verification Baseline:** Accepted IDR-SRV-050 independent conformance harness and IDR-SRV-051 normalized assertion-level traceability/evidence graph<br>
**Current Tool Evidence:** Cargo, nextest, Tokio, Axum, SQLx, rstest, proptest, insta, testcontainers-rs, wiremock, assert-json-diff, jsonschema, schemars, Criterion, cargo-fuzz, cargo-mutants, cargo-llvm-cov, RustSec, cargo-deny, cargo-machete, cargo-udeps and GitHub Actions primary documentation checked September 16, 2026<br>
**Document Purpose:** Define the implementation-facing Rust testing architecture without creating code, fixtures, thresholds, security accreditation evidence, interoperability claims or a complete test catalog<br>
**Author:** OpenAI Codex<br>
**Date:** September 16, 2026<br>
**Last Updated:** September 16, 2026

---

## Evidence and Decision Legend

- **[N] Normative:** approved external standard or incorporated requirement/artifact.
- **[A] Accepted project baseline:** accepted Glaux report or governing project decision.
- **[D] Direct documentation:** official language, crate, tool, platform or database documentation.
- **[I] Implementation evidence:** another implementation, client, community finding or project; informative only.
- **[T] Test evidence:** reproducible run/result with stated scope and conditions.
- **[E] Analysis:** reasoned synthesis from identified evidence.
- **[P] Project recommendation:** proposed Glaux decision pending acceptance of this report.
- **[X] Explicit boundary:** excluded claim or later-topic responsibility.

A passing unit test, high source-coverage percentage, clean static analysis, green conformance case, performance result, security assessment and interoperability run are different evidence types. None silently substitutes for another. **[X]**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Test Requirement Extraction Methodology
5. Rust TDD Philosophy and Scope
6. Multi-Layer Test Taxonomy
7. Repository and Workspace Test Organization Findings
8. Test-Support Crate and Module Findings
9. Rust Test Tooling Evaluation
10. CI Quality Gate and Artifact Findings
11. Unit, Domain, and Validation Test Strategy Findings
12. API, Contract, and Conformance-Adjacent Test Strategy Findings
13. Database, Repository, and Migration Test Strategy Findings
14. Fixture, Golden-File, Snapshot, Property, and Fuzz Test Strategy Findings
15. Async, Streaming, and Event Test Strategy Findings
16. Command, Control, Security, Policy, DDIL, and Synchronization Test Strategy Findings
17. Traceability, Conformance, Fixture, Performance, Security, and Interoperability Handoff Findings
18. Recommendations
19. Risks, Constraints, and Open Questions
20. Validation Against This Plan's Success Criteria
21. References

---

## 1. Executive Summary

Glaux should use **obligation-first, boundary-conscious TDD**. For every behavior change, the developer begins with an accepted requirement, decision, risk or regression; selects the cheapest test layer capable of detecting the relevant failure; writes an executable failing assertion; implements the smallest behavior; then refactors while preserving lower- and higher-layer contracts. Public or standards-visible behavior requires both focused internal proof and independent wire-level evidence when its claim depends on transport behavior. TDD does not mean driving every private function through an end-to-end request or freezing incidental implementation details. **[A,P]**

The target architecture is a layered evidence portfolio, not a single pyramid count. Pure domain invariants and state machines receive dense unit/model/property tests. Application use cases run against narrow fakes and reusable port contract suites. Axum routers run in-process through Tower for most HTTP semantics; a smaller real-listener lane covers network framing, disconnects, streaming and proxy/origin behavior. PostgreSQL/PostGIS behavior runs against the real engine with fresh migrations and isolated databases. The independent IDR-SRV-050 harness remains the authority for black-box conformance evidence. Fuzz, performance, security and named-pair interoperability are distinct scheduled/release lanes. **[A,D,P]**

Use `cargo nextest` as the primary unit/integration runner in CI from the first implementation because its repository profiles, per-test groups, timeouts, partitioning, JUnit output and flaky-test reporting fit the workspace. Keep ordinary `cargo test` compatibility and run `cargo test --doc` separately because nextest does not execute doctests. Set retries to zero for normal blocking runs; diagnostic retry may classify a failure as flaky, but a pass-on-retry remains a failed release signal. **[D,P]**

The initial workspace retains the accepted production packages and adds `glaux-test-support` only for builders, deterministic clocks/IDs, in-memory port fakes, effect recorders, fixture loaders, assertion helpers and adapter contract suites. Production packages never depend on it. Broad “mock everything” frameworks and a universal repository abstraction are rejected: hand-written fakes prove application decisions, while real PostgreSQL/PostGIS tests prove SQL, isolation, migrations, spatial behavior and concurrency. Wiremock is limited to external HTTP dependencies; the Glaux API itself is exercised as an Axum/Tower service or real server. **[A,D,P]**

Stable test and assertion IDs live in the IDR-SRV-051 trace catalog, not solely in Rust symbol names or custom attributes. A small test context/helper receives the stable case ID, emits assertion outcomes and binds exact profile/fixture/tool/build facts. The catalog records the Cargo package, test target and executable selector. CI compares the catalog to runner inventory, rejects unknown/missing mappings and ingests immutable JUnit/native evidence without inferring coverage from a process exit code. A custom procedural attribute is deferred unless a proof shows it reduces duplication without hiding test discovery. **[A,P]**

Determinism is designed into production seams: explicit clocks, ID/entropy sources, bounded task supervision, controllable external-effect ports and injected configuration. Tokio paused time is used for internal timer/backoff logic; no correctness test waits for wall-clock sleeps. Database time, real listener behavior and external processes receive separate integration cases. Every randomized property run records or persists a replay seed, and every fuzz crash is minimized and promoted to a permanent regression before closure. **[D,P]**

Snapshots and goldens are narrow review tools. Exact comparison is appropriate for canonical bytes, media types, stable problem documents, curated OpenAPI fragments and signed/hashed artifacts. Semantic JSON/XML/domain assertions are preferred when ordering or volatile fields are not contractual. CI never updates snapshots; every change shows a semantic diff, linked requirement/assertion, reviewer and reason. IDR-SRV-053 owns the corpus and update workflow. **[A,D,P]**

CI is cost-tiered. Every PR blocks on formatting, compile/lint, unit/domain/validation/application/router tests, doctests, trace/schema/generated-artifact checks, migration bootstrapping, a bounded real-database lane, supply-chain policy and affected contract tests. Nightly lanes expand feature/profile matrices, properties, fuzzing, mutation sampling, full database/conformance and profile scenarios. Release-candidate lanes add clean-deployment, complete conformance, security, performance and named-pair interoperability evidence owned by later topics. Coverage is diagnostic and risk-directed; a global line percentage cannot prove requirement coverage. **[A,P]**

## 2. Scope and Plan Alignment

### 2.1 Completed Scope

This report completes all six authorized phases by:

- extracting test obligations from all accepted IDR-SRV-001 through IDR-SRV-051 findings;
- defining the red-green-refactor contract and test-layer taxonomy;
- fixing workspace/test-support boundaries and production-code seams;
- evaluating every tool family named by the plan plus narrowly relevant concurrency tools;
- assigning PR, nightly, manual and release-candidate gates and evidence artifacts;
- defining functional strategies for all required server areas; and
- producing implementation proofs and explicit IDR-SRV-053 through IDR-SRV-057 handoffs.

### 2.2 Explicit Boundaries

This report does not:

- implement the Cargo workspace, test macros, trace registry, server or CI workflows;
- enumerate all 233 Part 1/2 requirements, 240 ATS items or project requirements as concrete tests;
- create the fixture/scenario/golden corpus or approve official examples as fixtures;
- set performance budgets, load models, security assessment depth, scanner policy or named interoperability partners;
- authorize physical command effects, public-endpoint mutation, live operational data or controlled source material in tests;
- redefine normative obligations or make conformance, certification, security, capacity or readiness claims; or
- authorize IDR-SRV-053 or later research.

### 2.3 Research Question Coverage

| Plan theme | Status | Evidence |
|---|---|---|
| TDD meaning and timing | Complete | Sections 4–5 |
| layer taxonomy and ownership | Complete | Section 6 |
| repository and test-support organization | Complete | Sections 7–8 |
| Rust tools and selection | Complete | Section 9 |
| CI tiers, artifacts and flakes | Complete | Section 10 |
| unit/domain/validation/API/database strategy | Complete | Sections 11–13 |
| fixture/snapshot/property/fuzz strategy | Complete | Section 14 |
| async/stream/event strategy | Complete | Section 15 |
| command/security/DDIL/sync strategy | Complete | Section 16 |
| traceability/conformance/downstream integration | Complete | Section 17 |

## 3. Evidence Base and Authority Classification

### 3.1 Accepted Project Evidence

| Source group | Binding testing contribution |
|---|---|
| IDR-SRV-001–008 | AEP/OGC authority separation; exact Part 1/2 class, requirement, recommendation and ATS inventory; controlled-source limits |
| IDR-SRV-009–024 | discovery, HTTP, OpenAPI, representation, identity, lifecycle, time, provenance, SensorML/SWE and validation invariants |
| IDR-SRV-025–038 | transaction, persistence, query, ingestion, publication, stream, command, feasibility and safety behaviors |
| IDR-SRV-039–043 | authentication, authorization, policy/releasability, audit, DDIL and synchronization negative/failure behaviors |
| IDR-SRV-044 | accepted Rust/Axum/Tokio/SQLx/tool candidate versions, failing-contract-first rule, real-database requirement and CI/supply-chain baseline |
| IDR-SRV-045 | modular-monolith package graph, inward dependencies, thin handlers, explicit ports and `glaux-test-support` boundary |
| IDR-SRV-046–049 | eleven profile contracts, deterministic deployment/configuration/telemetry/migration/restore behaviors and safe simulation boundaries |
| IDR-SRV-050 | independent black-box case/run/result contracts, six outcomes, target/fixture safety and claim closure |
| IDR-SRV-051 | stable IDs, assertion-level mappings, immutable evidence, orthogonal state, impact/freshness and CI/report contracts |

### 3.2 Current Primary Tool Evidence

| Source | Direct finding used here | Boundary |
|---|---|---|
| Cargo test | builds and runs unit, integration and documentation tests; multiple test targets and doctests have distinct execution behavior | baseline runner, not trace/evidence system |
| cargo-nextest | repository profiles, filters/groups, timeouts, partitions, retries/flaky classification, archives and JUnit; no doctest execution | selected CI runner plus separate doctest lane |
| Tokio testing | paused time fast-forwards timers when no non-time future can progress; `start_paused` needs test utilities | internal-runtime determinism, not database/external time |
| Axum/Tower | Router is a Tower service and can be exercised in process; actual listener remains separate | in-process HTTP does not prove all wire behavior |
| SQLx test | isolated test databases, automatic migrations and SQL fixture application | requires real PostgreSQL service/privileges; fixtures remain IDR-SRV-053 governed |
| rstest | parameterized cases, fixtures, async cases and timeouts | selective ergonomics, not mandatory DSL |
| proptest | generated cases, shrinking, seed/failure persistence and configurable runs | properties require valid deterministic strategies/oracles |
| insta | serialized snapshots, redactions, filters and review tooling | updates need semantic governance; redaction must not hide contract fields |
| testcontainers-rs | pinned real container images with async/sync runners and readiness conditions | environment bootstrap, not one container per small test |
| wiremock | isolated HTTP mocks, matchers, response behavior and invocation expectations | external HTTP adapters only, not server self-testing oracle |
| jsonschema/schemars | Draft 2020-12 validation and Rust-type schema generation | generated schema does not replace curated standards artifacts |
| cargo-fuzz | libFuzzer targets, corpus minimization and coverage on supported nightly Unix-like environments | scheduled specialized lane, not Windows/ordinary PR baseline |
| cargo-llvm-cov | LLVM source coverage for cargo test/nextest and multiple report formats | reachability signal, not requirement/semantic proof |
| cargo-mutants | injects plausible faults to expose assertions that do not detect behavior change | targeted scheduled diagnostic, requires stable fast tests |
| RustSec/cargo-deny | advisory, license, duplicate/ban and source checks over resolved dependencies | supply-chain gate, not runtime security test |

### 3.3 Authority Rules

1. A normative or accepted project requirement defines expected behavior; a crate/tool documents only how a test may execute.
2. The IDR-SRV-051 trace catalog owns stable test/assertion relationships; Rust code and runner output reference it.
3. Independent black-box conformance results retain their IDR-SRV-050 authority lane and are not replaced by internal tests.
4. A fake verifies application decisions about a port; only the real adapter contract/integration lane verifies the adapter technology.
5. Source coverage, mutation score, fuzz duration and benchmark measurements remain separate risk signals.
6. Implementation/community observations generate regression/risk tests only when recorded as non-normative derivations.

### 3.4 Implementation and Community Lessons

- OSH and Connected Systems Go show that broad implementation test suites may coexist with fixed or inaccurate conformance declarations; declarations require independent profile/evidence closure.
- pygeoapi and reviewed implementations show route, OpenAPI, representation and runtime drift when generated/static artifacts are not tested bidirectionally.
- SECD and OS4CSAPI smoke findings show that status-only assertions miss ignored filters, wrong data, parser coercion and link/identity defects.
- Mutable public demos and third-party client limitations are unsuitable deterministic oracles; retain raw evidence and classify target/tool/client failures separately.
- Long report matrices are useful design evidence but cannot substitute for executable manifests, test inventory and generated trace reports.

These are non-normative test-architecture lessons. **[I]**

## 4. Test Requirement Extraction Methodology

### 4.1 Extraction Chain

For each accepted source or decision, derive:

`requirement/assertion → failure mode → authoritative observation point → cheapest detecting layer → required higher boundary → profile → fixture/fake/real dependency → deterministic controls → CI tier → evidence artifact → downstream owner`

A test exists only when it has a requirement, risk, regression or explicitly time-bounded exploratory purpose. One broad scenario may contain multiple assertion IDs with separate outcomes; one requirement may require several layers because each observes a different boundary. **[A,P]**

### 4.2 Layer Selection Rules

Choose the lowest layer that can falsify the behavior without replacing the behavior under test:

- pure calculations and invariants: unit/property/model;
- ordering across consumed ports: application service with recording fakes;
- public serialization and middleware: in-process router/contract;
- SQL, transaction, PostGIS, migration and concurrency: real database;
- HTTP framing, connection lifecycle, streaming and proxy/origin: real listener;
- declared OGC behavior: independent black-box harness;
- capacity, abuse resistance and external compatibility: dedicated later-topic lanes.

Duplicate assertions at different layers only when their observation boundaries differ. Do not repeat the same JSON equality assertion through unit, router, listener and conformance tests merely to increase counts. **[P]**

### 4.3 Priority and Timing

| Situation | Test timing |
|---|---|
| normative/accepted externally visible behavior | failing contract before production behavior |
| domain invariant or state transition | model/example tests before implementation; properties alongside model refinement |
| persistence/query behavior | contract examples before port implementation; real-engine cases before merge |
| discovered defect | minimal failing regression first, then fix; derive missing broader invariant if appropriate |
| exploration of unknown library behavior | disposable spike first; preserve only decision-useful proof as a named characterization test |
| performance/security/interoperability finding | dedicated reproducer first; acceptance threshold/disposition remains later-topic authority |
| refactor with no semantic change | existing tests plus impact analysis; new tests only for uncovered risk or seam |

### 4.4 Evaluation Criteria

Each layer/tool is evaluated for failure-detection authority, execution speed, determinism, isolation, diagnostic quality, traceability, fixture and external-service cost, sensitive-data risk, portability, maintenance load and evidence reproducibility. Popularity or line-coverage gain alone is not a selection criterion. **[P]**

## 5. Rust TDD Philosophy and Scope

### 5.1 Red-Green-Refactor Contract

For a behavior-changing work item:

1. resolve exact requirement/assertion IDs, profile and source/decision pins;
2. state the failure mode and select observation layers;
3. add or update the manifest entry and deterministic inputs;
4. run the new assertion and retain proof it fails for the expected reason, not setup failure;
5. implement the smallest production change with no test-only production branch;
6. run focused then PR-tier suites;
7. refactor while preserving observable contracts;
8. generate trace/impact/evidence artifacts; and
9. review semantic output, skipped/filtered assertions and profile/conformance impact.

The “red” state is a developer/PR review fact, not long-lived release evidence. A test written after implementation is acceptable for characterization or a discovered regression, but the reason is recorded. **[P]**

### 5.2 Outside-In and Inside-Out Balance

Start each vertical slice with a small externally meaningful contract—such as negotiated landing response—and decompose inward: domain types/invariants, application use case, standards mapping, port contract, adapter, router, then listener/harness. This prevents infrastructure-first code while keeping most red-green cycles fast. The black-box harness validates the integrated slice; it does not drive every internal branch. **[A,P]**

### 5.3 Testability Seams

Production design supplies explicit narrow seams for:

- `Clock`/time observation and scheduler deadlines;
- UUID/nonce/random sources where deterministic values are semantically permitted;
- repository/transaction use-case ports, source/policy clients, artifact stores, publisher and command gateways;
- effect/audit/outbox recorders at application contracts;
- typed configuration/profile input;
- task spawning/supervision and shutdown signals; and
- parser/validator package registries with ambient network retrieval disabled.

Seams exist to express architecture boundaries, not solely for mocking. Do not add a trait around every pure function or external crate. **[A,P]**

### 5.4 Anti-Patterns Rejected

- end-to-end-only testing;
- database mocks used to claim SQL/PostGIS/isolation behavior;
- snapshotting entire volatile responses and approving wholesale diffs;
- wall-clock sleeps, random public ports without capture, mutable public endpoints or live identity/broker dependencies in blocking tests;
- shared mutable fixture databases without isolation;
- ignored/quarantined tests contributing to coverage or claimability;
- test-only branches in production behavior selected by hidden environment variables;
- requirement IDs encoded only in Rust function names;
- retries that convert flaky evidence into release success; and
- measuring success by global line coverage or raw test count.

## 6. Multi-Layer Test Taxonomy

### 6.1 Required Test Strategy Matrix

| Test layer | Purpose | Requirement source | Target code/system boundary | Tooling | Fixture dependency | Assertion approach | CI tier | Traceability requirement | Evidence artifact | Downstream topic handoff | Notes / unresolved issues |
|---|---|---|---|---|---|---|---|---|---|---|---|
| static/build policy | reject formatting, lint, unsafe, feature and dependency-graph violations | accepted implementation/supply-chain decisions | workspace metadata and compilation | rustfmt, Clippy, Cargo, cargo-deny | none/config | exact exit plus machine report | PR | gate and decision IDs | logs/SARIF-equivalent reports | 055,057 | not behavior coverage |
| pure unit/value | prove constructors, parsers, IDs, URI/link helpers and local invariants | normative/project atomic requirements | one module/type, no I/O/runtime | libtest, rstest selectively | inline vectors | exact typed/result assertions | local/PR | case plus assertion IDs | nextest/JUnit result | 053 | dense, fast default |
| domain model/state | prove lifecycle, temporal, command, DDIL and conflict transitions | accepted domain/state decisions | domain aggregate/service | libtest, rstest, proptest | table/model vectors | transition table, invariant, property | local/PR/nightly expansion | assertion per transition/invariant | result plus replay seed | 053–055 | no Axum/SQLx |
| structural/semantic validation | prove schema/profile/domain validation and stable findings | CSAPI/SensorML/SWE/OpenAPI/project validation rules | validation pipeline/registry | jsonschema, curated validators, Rust assertions | valid/invalid documents | finding code/path/severity plus semantic facts | PR | rule IDs and source pins | finding bundle | 053,055 | offline package pins |
| application use case | prove authorization-validation-transaction-audit/effect order and outcomes | accepted workflow decisions | application service and consumed ports | libtest/Tokio, hand fakes/recorders | builders/scenarios | outcome and ordered effect ledger | PR | requirement/decision/assertion IDs | structured test result | 053,055 | fake verifies decisions, not adapter |
| adapter contract | run one reusable behavioral suite against fake/reference and real adapter | accepted port semantics | one port implementation | shared suite, wiremock where HTTP | contract vectors | same observable contract for implementations | PR/nightly by cost | port contract and implementation IDs | parameterized suite results | 053,055 | no lowest-common-denominator abstraction |
| HTTP router in-process | prove route, extractor, middleware, negotiation, headers, body and problems | HTTP/CSAPI/OpenAPI/security decisions | Axum Router/Tower service | Tower `oneshot`, http-body utilities | API scenarios | status/header/media/schema/semantic | PR | operation plus assertion IDs | request/response result | 053,055 | no kernel socket |
| HTTP listener contract | prove real connection/origin/proxy/stream/disconnect behavior | HTTP/deployment/stream decisions | bound ephemeral listener and client | Axum/Hyper, reqwest | finite server scenario | wire bytes/headers/timing state, bounded | PR subset/nightly | listener-specific assertions | raw exchange/transcript | 054–056 | smaller than router suite |
| repository/database | prove SQL, PostGIS, transactions, locking, idempotency, outbox and materialization | persistence/query/write decisions | real PostgreSQL/PostGIS adapter | SQLx test, pinned CI service/testcontainers | migrated isolated DB | rows/domain facts plus concurrent outcomes | PR core/nightly full | repository contract/assertion IDs | JUnit plus DB diagnostic | 053–055 | never SQLite substitute |
| migration/continuity | prove empty bootstrap, upgrade chain, compatibility and restore semantics | IDR-SRV-049 | migration/admin boundary and real DB | SQLx migrator, deployment scripts | versioned DB states/artifacts | schema/data manifest and service behavior | PR changed/nightly/RC | migration/gate IDs | migration/restore manifest | 053–055 | destructive operations isolated |
| curated golden/snapshot | detect intentional stable wire/artifact changes | representation/OpenAPI/problem/evidence decisions | serializer/generator output | explicit files, insta selectively | stable golden | exact or normalized semantic diff declared per artifact | PR | fixture/golden/assertion IDs | reviewed diff and digest | 053 | CI cannot update |
| property/model based | explore input/state spaces and prove invariants | domain/query/security invariants | pure/model boundary; selected adapter models | proptest | generators and persisted regressions | invariant/metamorphic/model equivalence | PR bounded/nightly expanded | property ID plus seed/case | seed/minimal counterexample | 053–055 | deterministic strategy required |
| async worker/supervision | prove cancellation, retry, deadline, shutdown and effect ordering | streaming/operations decisions | Tokio task/worker with controlled ports | Tokio paused time, channels, fakes | finite schedule | state/event/effect sequence, no leaked task | PR | workflow assertion IDs | deterministic event trace | 054,055 | no sleep-based tests |
| streaming/event scenario | prove replay, cursor, gap, order, backpressure and reconnect semantics | IDR-SRV-035/042/043 | durable log/outbox/SSE/broker adapter | Tokio, real DB, listener; broker later | finite event/fault schedule | item sequence, watermark and terminal behavior | PR core/nightly | stream assertion IDs | transcript/run manifest | 053,054,056 | capacity belongs to 054 |
| command simulator | prove validation, authority, safety, lifecycle, fence and no physical effect | Part 2 and IDR-SRV-036–038 | application, DB, simulated gateway | model tests, effect recorder, real DB/API | safe command scenario | state/effect/audit ledger; gateway absence/presence | PR/RC | command/safety assertion IDs | protected synthetic transcript | 053,055,056 | physical adapter unavailable |
| security/policy baseline | prove middleware, object/property filtering, concealment, redaction and unsafe-config denial | IDR-SRV-039–041/047/048 | decision and enforcement points | unit/router/listener, fake authority | synthetic identities/policies | allow/deny/indistinguishability and no-leak | PR | threat/control/assertion IDs | redacted result | 053,055 | deep assessment owned by 055 |
| DDIL/synchronization | prove stale/partial posture, duplicate/gap/conflict/quarantine and recovery | IDR-SRV-042/043 | domain plus isolated nodes/adapters | model/property, DB, fault controller | deterministic two-node schedule | state/lineage/audit convergence invariants | PR core/nightly | scenario/assertion IDs | node/event manifests | 053–055 | no global mode flag |
| black-box conformance | independently verify published standard/profile behavior | exact OGC requirement/ATS graph | packaged/running server public API | IDR-SRV-050 Rust CLI and external ETS adapters | conformance corpus/target | assertion-level six-outcome semantics | PR smoke/nightly/full RC | mandatory exact IDs | CaseResultV1/ConformanceRunV1 | 053–057 | independent wire models/oracle |
| fuzz/robustness | find parser, codec, state-machine and resource-bound failures | risk/threat plus parser requirements | narrow pure or harness target | cargo-fuzz/sanitizers | seed corpus | no crash/hang/violation; minimized reproducer | nightly/manual | fuzz target/risk/regression IDs | corpus/crash/minimized case | 053–055 | not normal Windows PR lane |
| performance smoke/microbench | detect gross regressions and retain benchmark mechanics | non-functional risk/accepted mechanics | pure codec/query/component path | Criterion plus functional smoke | representative small scenario | statistical result or generous smoke guard | nightly/advisory until 054 | metric/workload IDs | raw/report/environment | 054 | thresholds deferred |
| interoperability smoke | detect named-client/server integration regressions | named pair/scenario | packaged server plus external client/server | external client harness | exact pair/profile corpus | semantic scenario result, not status only | nightly/manual/RC later | pair/test/fixture IDs | raw named-pair run | 056 | never universal compatibility claim |

### 6.2 Layer Closure Rules

- A test at a higher layer does not automatically cover every lower invariant traversed.
- A lower-layer pass does not prove transport, integration, conformance or deployment behavior.
- Each release requirement declares its minimum layers and required current evidence in the trace graph.
- A layer may be “not required” only by explicit verification rationale, never by missing test discovery.
- Setup, fixture or dependency failure is `error`/`blocked`, not behavior failure or skip.

## 7. Repository and Workspace Test Organization Findings

### 7.1 Recommended Layout

```text
glaux-server/
├─ Cargo.toml
├─ .config/nextest.toml
├─ crates/
│  ├─ glaux-domain/src/**/tests.rs
│  ├─ glaux-application/src/**/tests.rs
│  ├─ glaux-standards/{src,tests}/
│  ├─ glaux-validation/{src,tests}/
│  ├─ glaux-persistence-postgres/{src,tests,migrations}/
│  ├─ glaux-api-http/{src,tests}/
│  ├─ glaux-adapters/{src,tests}/
│  ├─ glaux-server/{src,tests}/
│  └─ glaux-test-support/src/
├─ tests/
│  ├─ contract/
│  ├─ profile/
│  └─ deployment/
├─ conformance/
├─ traceability/
├─ fixtures/
├─ fuzz/
├─ benches/
└─ xtask/
```

Inline `#[cfg(test)]` modules own white-box pure tests. Each crate's `tests/` directory owns public crate-boundary cases. Root tests own only multi-crate/profile/deployment scenarios. The standalone conformance harness retains independent models and dependencies. Fixtures, fuzz corpora and benchmarks have stable manifests rather than being hidden under arbitrary test modules. **[A,P]**

### 7.2 Dependency Direction

- production crates may depend only in the accepted inward direction;
- production crates do not depend on `glaux-test-support`, conformance code, fixtures or dev-only assertion crates;
- `glaux-test-support` may depend on stable public domain/application contracts and narrowly on adapter/API types needed for contract suites, but never become a production service locator;
- conformance code must not import server implementation types or production validators as its oracle;
- feature flags used only to expose internals for tests are rejected; test public behavior or keep white-box unit tests beside private code; and
- compile/metadata checks enforce forbidden dependency edges and production build independence from dev dependencies.

### 7.3 Target and Feature Organization

Give integration binaries bounded themes rather than one file per requirement or one giant `integration.rs`. Runner filters and manifests select packages/targets/cases; Cargo features select product capabilities, not “test mode.” Test the supported feature/profile matrix explicitly, including command-disabled and minimal builds. Default-feature-only success is insufficient because Cargo feature unification can hide missing dependency declarations. **[A,P]**

### 7.4 Generated Versus Curated Artifacts

Curated sources include test code, trace/test manifests, approved fixtures/goldens, schema pins, migrations, proptest regressions and fuzz seed corpus. Generated outputs—JUnit, coverage, nextest archives, expanded OpenAPI, trace reports, temporary databases and new snapshot proposals—go under ignored build/CI artifact locations. CI fails if generation changes tracked output without the reviewed source update. **[P]**

## 8. Test-Support Crate and Module Findings

### 8.1 Permitted Responsibilities

`glaux-test-support` may provide:

- valid/invalid domain builders that make defaults visible;
- deterministic clocks, UUID/nonce sources and seeded randomness;
- hand-written in-memory fakes for consumed application ports;
- effect/audit/outbox/gateway recorders with ordered assertions;
- fixture/scenario manifest loaders and sensitivity guards;
- shared adapter contract suites parameterized by implementation factory;
- Axum test application composition with explicit profile/configuration;
- PostgreSQL test bootstrap helpers that call production migrations;
- structured assertion/result helpers tied to stable test/assertion IDs; and
- safe fault schedules for dependency, worker, DDIL and synchronization tests.

### 8.2 Prohibited Responsibilities

It must not:

- contain a second implementation of normative business logic used as an oracle;
- expose raw pools/brokers/gateways to application tests that bypass ports;
- silently supply ambient current time, random IDs, credentials or network endpoints;
- normalize away contractually visible fields;
- perform physical command dispatch or call public/live systems;
- turn setup failures into skips/passes;
- become a production dependency; or
- hide broad global mutable state behind convenient fixtures.

### 8.3 Fakes, Stubs, Mocks, Simulators, and Real Dependencies

| Double/dependency | Use |
|---|---|
| fake | stateful application-port behavior needed to exercise decisions; records calls/effects |
| stub | one controlled response for an otherwise irrelevant dependency |
| mock/expectation | narrow interaction contract where call count/order itself is required |
| simulator | deterministic command, source, event or node protocol state machine with explicit limitations |
| real dependency | SQL/PostGIS, migration, wire protocol, TLS/proxy, broker-specific or external-tool behavior |

Prefer observable outcomes over interaction assertions. Verify call ordering only where authorization, audit, transaction, dispatch or safety semantics make the order a requirement. **[P]**

### 8.4 Stable Test Identity Mechanism

For first implementation, use trace-catalog records with `package`, `test-target`, `executor`, `selector`, assertion IDs, tier/profile/features and fixture IDs. The Rust test creates `TestContext::for_case("TEST-…")` and records named assertion outcomes. `xtask trace check-tests` compares manifest entries with nextest/Cargo inventory and rejects duplicate, missing or unregistered non-exploratory cases. Test function names remain readable descriptions, not identity authority. **[P]**

A declarative helper macro may remove result boilerplate after the plain API works. A procedural `#[glaux_test(...)]` attribute is deferred: it introduces a compiler/plugin surface and can obscure expansion/discovery. Adoption requires a proof that metadata can be inventoried without executing test bodies and that normal Rust IDE, Cargo and nextest behavior remains intact. **[P]**

## 9. Rust Test Tooling Evaluation

### 9.1 Runner, Assertion, and Data Tool Matrix

| Tool | Role | First implementation decision | Gate/cadence | Important limitation/control |
|---|---|---|---|---|
| built-in libtest / `cargo test` | universal unit, integration and doctest baseline | **Required** | local/PR; `--doc` explicit | test targets and doctests have different execution; keep compatibility |
| cargo-nextest | primary non-doctest execution, filtering, grouping, timeout, JUnit and partitioning | **Required and pinned** | local optional, PR/nightly/RC primary | separate doctests; repository profiles; zero default retries |
| rstest | readable tables/fixtures/parameterization | **Selective** | same tier as owning tests | avoid fixture graphs and macro ceremony that obscure cases |
| standard assertions | typed/simple outcomes | **Preferred default** | all | write domain-specific messages/helpers for diagnosis |
| assert-json-diff | exact/inclusive JSON diagnostics | **Select narrowly** | PR | inclusive matching must not omit required negative/absence assertions |
| insta / cargo-insta | reviewed snapshots and structured diffs | **Selective** | PR, never auto-update | snapshot review is semantic governance, not approval shortcut |
| proptest | generated invariant/model/metamorphic cases with shrinking | **Required for named high-value invariants** | bounded PR, expanded nightly | persist seeds/minimal regressions; deterministic strategies |
| cargo-fuzz/libFuzzer | parser/codec/state robustness and sanitizer-backed fuzzing | **Prepared from first parser; scheduled** | nightly/manual/release campaign | nightly Unix-like specialized runner; time is not proof of absence |
| wiremock | isolated outbound HTTP dependency behavior | **Select** | PR | do not use to test Glaux's own server API or non-HTTP ports |
| testcontainers-rs | pin/start real infrastructure for local/portable integration | **Select environment bootstrap** | local/PR/nightly | prefer one service per job/session; pin image digest and readiness |
| SQLx `#[sqlx::test]` | isolated migrated database and SQL fixture support | **Select with proof** | PR/nightly | requires database creation authority; use production migrations |
| jsonschema | offline multi-draft structural validation | **Select per IDR-SRV-044 proof** | PR | precompile registries; disable ambient retrieval; bound resources |
| schemars | Rust-type schema generation/drift comparison | **Select for project DTOs only** | PR drift | never replaces curated OGC/OpenAPI/schema authority |
| Criterion | microbenchmark mechanics/statistical comparison | **Prepare; thresholds deferred** | nightly/advisory until 054 | environment noise; not load/capacity testing |
| cargo-llvm-cov | source-region/line coverage and standard outputs | **Select diagnostic** | nightly/PR-on-demand/RC | no global percentage as requirement coverage; doctest caveat |
| cargo-tarpaulin | alternative coverage engine | **Do not select initially** | fallback evaluation only | avoid dual coverage systems without demonstrated platform need |
| cargo-mutants | assertion-strength diagnostic | **Targeted advisory** | nightly/manual changed critical modules | high cost; surviving mutants need disposition, not raw score gaming |
| cargo-audit | RustSec vulnerability/yank audit | **Required per accepted policy** | PR/scheduled fresh DB | exceptions explicit, owned and expiring |
| cargo-deny | advisories, licenses, bans/duplicates and sources | **Required** | PR | resolved feature graph and approved policy configuration matter |
| cargo-machete | fast unused-dependency signal | **Advisory** | PR/nightly | intentionally imprecise; generated/build uses need reviewed exceptions |
| cargo-udeps | compiler-informed unused dependencies | **Deferred/advisory** | scheduled if value exceeds nightly/toolchain cost | requires nightly and has known gaps |
| rustfmt | deterministic formatting | **Required/pinned** | PR | formatting only |
| Clippy | lint and common correctness/suspicion checks | **Required** | PR, warnings denied by policy | localized documented allowances; not semantic proof |
| Cargo feature/MSRV checks | build graph and compatibility | **Required** | PR matrix/scheduled full powerset | feature unification requires deliberate isolated builds |
| Miri | undefined-behavior/interpreter checks | **Conditional scheduled** | selected unsafe-sensitive/pure crates | high cost/unsupported operations; first-party unsafe remains forbidden |
| loom | concurrency interleaving model | **Conditional** | critical custom synchronization only | avoid custom concurrency primitives; Tokio/application scenarios remain primary |

### 9.2 Primary Runner Decision

Nextest is mandatory in CI for non-doctest tests from the first implementation. Check `.config/nextest.toml` into the repository with named `ci`, `nightly` and `release` profiles, test groups for scarce PostgreSQL/listener/broker resources, per-test slow timeouts and JUnit output. Use filter expressions generated from explicit manifest/tier metadata, not filename folklore. Run `cargo test --workspace --doc` separately and retain at least one scheduled ordinary `cargo test` compatibility run. **[D,P]**

Normal CI uses no retries. A diagnostic workflow may retry a failed case to classify nondeterminism; `flaky-result = "fail"` remains in effect, and the original and retry attempts are retained. Do not add per-test retries as a permanent stabilization mechanism. **[D,P]**

### 9.3 Version and Installation Policy

Inherit the candidate crate/tool versions evaluated in IDR-SRV-044, then pin the actual implementation through `Cargo.lock`, Rust toolchain file, container/action digests and verified prebuilt tool versions. Install CI tools from locked releases or a controlled tool image rather than compiling unpinned latest versions during every run. Re-evaluate MSRV, license, features, advisories and output formats on update. **[A,P]**

### 9.4 Coverage and Mutation Interpretation

Coverage answers which source regions executed under a particular build; traceability answers which requirement assertions produced current evidence. Neither implies assertion quality. Use coverage to find unexecuted error paths and risky modules, and mutation testing to sample whether tests detect plausible logic changes. Begin without a global numeric line threshold. Publish per-crate/trend and changed-critical-path gaps, then let governance adopt bounded thresholds only after a measured baseline and exclusion policy. **[P]**

## 10. CI Quality Gate and Artifact Findings

### 10.1 Execution Tiers

| Tier | Trigger | Blocking scope | Representative contents |
|---|---|---|---|
| developer focused | edit/red-green loop | local decision | one package/module/case; no mandatory container unless target requires it |
| pre-push | developer choice/branch | local policy | format, check, affected fast tests, trace check |
| PR fast | every PR | blocking | formatting, compile, Clippy, unit/domain/validation/application/router, doctest, trace/schema/generated drift, dependency policy |
| PR integration | every PR, partitioned | blocking | real PostgreSQL/PostGIS bootstrap/migrations/core repository and affected listener/contract/profile smoke |
| nightly broad | scheduled/main | blocking main-health alert, not retroactive PR success | full DB/profile/feature suites, expanded properties, fuzz smoke, mutation samples, full native conformance, deployment scenarios |
| manual campaign | authorized | finding-producing | long fuzz, fault/chaos, specialized platform, implementation-characterization |
| release candidate | signed candidate | release blocking | clean artifact/deployment, full trace-selected tests/conformance, migration/restore, security, performance and required interop lanes |

Time budgets should be measured and ratified during implementation. A slow test is optimized, partitioned or moved only if its release-risk coverage remains in an appropriate gate; cost alone does not justify silent removal. **[P]**

### 10.2 Pull-Request Blocking Gates

1. pinned `cargo fmt --check` with no generated-source rewrite;
2. `cargo check`/build for workspace targets, accepted default/minimal/profile features and MSRV core lane;
3. Clippy with workspace policy, warnings denied and localized reviewed allowances;
4. nextest PR profiles plus separate doctests, with no unknown, ignored or flaky release-required tests;
5. constrained trace/schema/test-inventory validation and generated report/OpenAPI/SQLx metadata drift checks;
6. empty-database migration bootstrap plus bounded real PostgreSQL/PostGIS contract/integration suite;
7. affected listener, worker, command-simulator, profile and native conformance smoke selected from graph impact;
8. cargo-deny and RustSec policy using a fresh/pinned advisory snapshot as required by reproducibility policy;
9. production build proves no test-support/dev dependency, forbidden unsafe or uncontrolled effect path; and
10. tracked tree remains clean after generators/tests, with secret/sensitive-artifact scanning.

Impact selection may add expensive cases, but a small invariant “sentinel” suite always runs to detect a broken impact graph. Changes to trace generators, shared domain, migrations, profiles, harness selection or test support trigger conservative broad suites. **[A,P]**

### 10.3 Flaky Test and Quarantine Policy

A nondeterministic result is a defect in the test, product or environment. The first failure remains visible and blocking for required evidence. If immediate repair is impossible, quarantine requires a stable test ID, issue, owner, observed attempts/environment, risk/class/profile impact, explicit non-claimability, expiry and an isolated scheduled lane. Quarantined/ignored tests are reported as gaps and cannot satisfy release or conformance gates. Repeated manual reruns until green are prohibited. **[A,P]**

The repair order is: reproduce with recorded inputs; remove wall-clock/network/global-state dependencies; isolate resource ownership; add deterministic schedules/seeds; improve diagnostics; then remove quarantine. Increasing timeouts or retries without a causal record is not a repair. **[P]**

### 10.4 CI Artifacts and Evidence

Each run should publish, subject to sensitivity policy:

- build/toolchain/lock/profile/feature/platform and source digests;
- resolved trace snapshot and exact selected/skipped assertion list with reasons;
- nextest JUnit plus native structured assertion results and failure stdout/stderr;
- migration/database image/extension manifest and setup outcome;
- request/response, event, effect or node transcripts only where required and redacted;
- generated-artifact/trace diff;
- property seed/minimal counterexample, fuzz crash/corpus delta or snapshot proposal when produced;
- coverage/mutation/benchmark reports in their separate authority lanes; and
- artifact digests, retention/sensitivity class and attestation where release policy requires.

GitHub job summaries are navigation views. Expiring workflow artifacts are not the only copy of release evidence. Attestation binds provenance but does not prove behavior, security or conformance. Retention durations and long-term store remain release/operations decisions. **[D,P]**

### 10.5 Failure Classification

CI and evidence adapters distinguish `assertion-failed`, `setup-error`, `dependency-unavailable`, `timeout`, `cancelled`, `filtered`, `not-applicable`, `quarantined`, `flaky`, `tool-error` and `infrastructure-error`. Only an executed assertion with the allowed terminal result covers its mapping. Early process exit never converts unexecuted assertions into failures or passes. **[A,P]**

## 11. Unit, Domain, and Validation Test Strategy Findings

### 11.1 Unit and Domain Inventory

Dense pure tests should cover:

- identifiers, source/canonical/local identity and URI construction;
- link relation/media/profile selection and stable ordering;
- lifecycle, revision, tombstone, temporal/freshness and latest/status rules;
- unit/quantity/property-role and canonicalization invariants;
- validation finding identity/path/severity and profile composition;
- pagination/cursor/sort/filter algebra independent from SQL;
- write/idempotency/conflict/precondition outcomes;
- command feasibility/state/cancellation/timeout/unknown-outcome transitions;
- policy decision/concealment/redaction projections;
- DDIL posture/staleness/last-known classification; and
- synchronization duplicate/gap/conflict/quarantine/lineage classification.

Keep these crates free of Axum, SQLx, environment, wall-clock and telemetry exporter dependencies. Tables cover accepted examples and boundary partitions; proptest covers algebraic/state invariants. **[A,P]**

### 11.2 Application-Service Tests

Application tests assemble validated typed inputs, a trusted operation context and narrow recording fakes. They assert the complete decision/effect sequence, including negative paths:

`authenticate context established → authorize/select policy → validate → begin semantic write → persist/audit/outbox atomically → commit → permit external effect/publication`

Unauthorized or invalid requests must produce no repository mutation, outbox, gateway call or sensitive diagnostics. Commit uncertainty and dependency failures must map to explicit outcomes, not generic success/failure. Contract suites run against each fake and real adapter to prevent fakes drifting into an easier contract. **[A,P]**

### 11.3 Validation Layers

Validation tests remain separated:

| Validation layer | Test evidence |
|---|---|
| transport parse | malformed syntax, type/size/depth limits and stable safe problem mapping |
| structural schema | official/pinned local schema packages, valid and discriminating invalid cases, closed offline references |
| profile rule | enabled-class/profile rule IDs and expected findings |
| semantic/domain | graph, identifier, time, units, lifecycle and cross-field facts |
| stateful/current | repository-backed uniqueness, revision, referential and transition checks |
| trust/policy/safety | separate decision records and enforcement tests, not validation finding substitution |

Official examples are candidate data, not automatic positive oracles. Test the validator against known valid, known invalid and mutation-derived cases; test schema selection/version/profile errors separately from instance errors. Stable finding codes/paths are exact; prose wording is semantic/snapshot only if it is an accepted public contract. **[A,P]**

### 11.4 Negative and Boundary Discipline

For every accepted success partition, include meaningful absence, malformed, unauthorized, unsupported, stale/conflicting and resource-bound cases. Do not generate dozens of syntactic variants that all exercise one parser branch while omitting semantic discrimination. Sensitive error cases assert both the returned problem and the absence of leaked identifiers, policy facts, source text, stack traces, SQL or credentials. **[A,P]**

## 12. API, Contract, and Conformance-Adjacent Test Strategy Findings

### 12.1 In-Process Router Tests

Build the same production Router through an explicit test composition function with real middleware order and deterministic ports. Tower service calls cover:

- landing, `/conformance` and OpenAPI links/content;
- route/method/path/query parsing and extractor rejections;
- content negotiation, exact media types, language/encoding where selected and `Vary` behavior;
- canonical/self/related/alternate links and configured public origin;
- filters, sorting, pagination, counts and empty/populated results;
- conditional requests, ETags/revisions and write preconditions;
- RFC 9457 status/type/title/detail/instance/extensions and redaction;
- identity/policy filtering, concealment and middleware order; and
- route absence/safe denial under disabled profiles.

Assertions combine exact status/header/media/link rules, schema validation and semantic body facts. Full-body snapshots are secondary and only for intentionally stable artifacts. **[A,D,P]**

### 12.2 Real Listener Tests

A smaller suite binds `127.0.0.1:0`, records the chosen port, uses a real HTTP client and closes through the production shutdown path. It covers behavior hidden by `oneshot`: Host/forwarded-origin handling, header/framing limits, streaming flush/heartbeat/reconnect, connection cancellation, body streaming, graceful shutdown and proxy/TLS integration where the deployment profile owns it. No fixed port, public network or sleep-based readiness is allowed. **[P]**

### 12.3 OpenAPI and Schema Drift

Test both directions:

- every enabled documented operation/media/problem/security declaration maps to a reachable router capability; and
- every enabled public route/method/media/error/security behavior is represented by the approved curated-plus-generated OpenAPI artifact.

Resolve references offline, parse as the claimed OAS edition, compare semantic canonical form and preserve the accepted OAS 3.0/3.1 compatibility seam. Schemars/Utoipa generation for project-owned DTO fragments is evidence, not authority over curated OGC definitions. **[A,P]**

### 12.4 Conformance Adjacency Without Duplication

Internal router/listener tests should prove lower-level invariants with better diagnostics; the external harness proves standards-visible behavior from independent models. Share fixture IDs and trace facts, not assertion implementation or production serializers/validators. If an internal and conformance assertion appear identical, retain the external one for independence and keep the internal case only when it materially localizes a separate boundary. **[A,P]**

The server's `/conformance` response is tested against the graph-approved declared set. It never selects its own test obligations. Harness results link back at assertion level through immutable adapters defined by IDR-SRV-050/051. **[A,P]**

## 13. Database, Repository, and Migration Test Strategy Findings

### 13.1 Real PostgreSQL/PostGIS Baseline

All SQLx repository behavior is tested against the accepted PostgreSQL/PostGIS family and required extensions. SQLite, mocked rows or an in-memory map may support application tests but cannot cover SQL syntax, collation, spatial/temporal operators, JSONB, isolation, locks, constraints, indexes, partitions or query plans. Pin the service image digest/extension versions and record them in evidence. **[A,P]**

For local portability, testcontainers-rs may start one job/session service. In CI, a pinned service container can reduce startup cost. Individual tests use isolated databases created through `#[sqlx::test]` or an equivalent proven allocator, apply production migrations and own cleanup. Limit concurrent database creators with a nextest test group rather than sharing mutable schemas casually. **[D,P]**

### 13.2 Required Repository Cases

- CRUD plus canonical/revision/tombstone behavior for every authoritative resource family;
- relationship, hierarchy, keyword/property, spatial and temporal queries with hit/miss/boundary controls;
- deterministic sort and pagination under ties and concurrent writes according to the accepted contract;
- JSONB/source-preservation round trips and representation-independent identity;
- Observation/batch/late/duplicate/latest/status semantics;
- transaction rollback, retryable conflict and commit-uncertainty classification;
- idempotency/inbox uniqueness, outbox/audit atomicity and worker claim/reclaim;
- concurrent conditional writes, command fences and synchronization application;
- materialization/rebuild equivalence and corruption/gap detection; and
- resource limits and safe diagnostic behavior for invalid/oversized inputs.

### 13.3 Transaction and Isolation Rules

Transaction rollback fixtures are acceptable for read-only or mechanics-focused cases only when they do not hide commit behavior. Any requirement involving commit, unique conflict, concurrent visibility, outbox publication, worker recovery, migration or reconnect uses committed state in an isolated database. Concurrency cases use explicit barriers/channels and bounded deadlines rather than probabilistic racing. **[P]**

Repository contract suites define semantic results, not generic CRUD. Run the same suite against the SQLx adapter and any test fake where applicable; add engine-specific cases for SQL/PostGIS/isolation. A generic object-safe unit-of-work solely to simplify tests remains rejected under the accepted architecture. **[A,P]**

### 13.4 Migration and SQLx Offline Checks

Every PR runs empty-to-head migration and verifies checksums/immutability. A migration-changing PR also exercises supported previous-schema-to-head paths with representative data and application compatibility phases. Nightly/release lanes add larger backfill, failure/interruption/resume, restore and reconciliation scenarios. SQLx offline metadata is regenerated deterministically and drift-checked, but successful compile-time query checking does not replace runtime migration/query tests. **[A,P]**

Database tests must fail closed when the expected extension, collation, timezone, role or migration state is absent. CI does not silently use a developer's preexisting database. **[P]**

## 14. Fixture, Golden-File, Snapshot, Property, and Fuzz Test Strategy Findings

### 14.1 Fixture Handoff Rules

This topic fixes how code consumes fixtures, not the corpus content. Every fixture/scenario reference uses an ID, version/digest, profile/source pins, sensitivity, setup/cleanup contract and expected facts from IDR-SRV-051. Loaders validate the manifest before use and report setup failure separately. Tests never infer fixture truth from current server output. IDR-SRV-053 decides organization, licensing, generators and update approval. **[A,P]**

### 14.2 Exact, Semantic, and Snapshot Assertions

| Artifact | Default comparison |
|---|---|
| canonical hashed/signed bytes, cursor envelope, stable media type/header | exact bytes/value |
| JSON/GeoJSON resource | semantic typed/JSON assertion; exact only when canonical form required |
| XML/SensorML/SWE | namespace/structure/semantic assertion; exact where lexical preservation is required |
| OpenAPI | resolved semantic/canonical diff plus critical exact extensions/refs |
| RFC 9457 response | exact status/type and stable fields; semantic/redacted detail policy |
| validation finding | exact code/path/severity; prose snapshot only if public contract |
| event/audit/effect | typed field/order assertion; volatile correlation/time normalized only by rule |
| CLI/report | structured machine output first; snapshot human view selectively |

Any normalization names each removed field and why it is non-semantic. Redaction/filtering must not mask required ordering, precision, identity, policy outcome or media/link changes. **[P]**

### 14.3 Golden Update Workflow

CI uses update-disabled snapshot settings. A proposed update includes old/new semantic diff, producing tool/version, affected fixture and assertion IDs, changed requirement/decision/source, reviewer and reason. Generated `.snap.new`/candidate files are failure artifacts, not accepted source. A reviewer must be able to reject one snapshot independently; blanket “accept all” is prohibited for standards/security/command artifacts. **[A,D,P]**

### 14.4 Property-Based Strategy

Use proptest where invariants span many values or operation sequences:

- identifier/URI parse-format round trips and invalid partitions;
- stable sorting/pagination without loss/duplication;
- temporal interval, late/latest/freshness boundary algebra;
- geometry envelope/boundary classification after a trusted oracle is defined;
- media negotiation preference and unsupported-set behavior;
- resource/command/synchronization state-machine legal/illegal sequences;
- idempotency and duplicate replay invariants;
- canonicalization/digest stability; and
- redaction/non-interference properties over synthetic labels.

PR profiles use a bounded case count and deterministic/persisted regression seeds; nightly expands cases and state lengths. Minimized failures become named example regressions so they remain useful even if generator behavior changes. Avoid assumptions that discard most generated inputs and hide poor coverage. **[D,P]**

### 14.5 Fuzz Strategy

Create narrow targets for query/media/cursor/URI parsers, JSON/XML/SWE decoders, schema/profile packages, event/sync envelopes and command payload/state-machine entry points. Targets must be deterministic, side-effect free or sandboxed, resource bounded and assert domain invariants beyond “does not panic” where possible. Seed with approved positive/negative fixtures, dictionaries and prior regressions without controlled data. **[P]**

Run short scheduled fuzz smoke and longer authorized campaigns on pinned supported runners. Store corpus/crash artifacts under sensitivity rules. Triage every crash/hang/resource violation, minimize it, assign a risk/requirement, add a deterministic regression and only then close the finding. Fuzz duration/coverage cannot claim input-space completeness. **[D,P]**

## 15. Async, Streaming, and Event Test Strategy Findings

### 15.1 Deterministic Async Rules

- inject logical clock/deadline decisions; use `#[tokio::test(start_paused = true)]` for internal timer/backoff behavior;
- use channels/barriers/notifications to establish ordering, never sleeps to “let the task run”;
- bound every wait with a diagnostic timeout while distinguishing product deadline from test watchdog;
- own spawned tasks in a supervisor/test scope and assert clean cancellation/join;
- capture panic, cancellation, channel close and dependency failure as explicit outcomes;
- do not hold blocking locks or database transactions across uncontrolled await points; and
- run selected multi-thread scheduler cases in addition to deterministic single-thread/model tests.

Paused Tokio time does not control PostgreSQL `now()`, OS socket timers or external processes. Tests for those boundaries use injected timestamps where the contract permits or real bounded integration evidence. **[D,P]**

### 15.2 Worker and Durable Effect Scenarios

For ingestion, outbox, audit spool, publication, cleanup and synchronization workers, test:

- empty, one-item and bounded-batch claims;
- crash/cancel before and after durable state transition;
- retryable versus terminal failure and backoff;
- lease expiry/reclaim and duplicate delivery;
- poison item isolation/dead-letter or quarantine behavior;
- graceful drain versus immediate stop;
- dependency outage/recovery and restart replay; and
- metrics/log/audit evidence without correctness reliance on telemetry.

The database/outbox remains authoritative; an in-memory channel pass does not prove durable recovery. **[A,P]**

### 15.3 Streaming and SSE Baseline

Test snapshot/watermark/catch-up/live handoff, opaque cursor binding, selected-event filtering, order rules, at-least-once duplicates, retention gaps, reconnect, heartbeat, slow-consumer policy, authorization/policy change and shutdown. Use finite event schedules and explicit consumer acknowledgments. Listener tests observe actual frames/disconnects; domain/model tests cover selection/state; real database tests cover replay authority. **[A,P]**

Functional PR cases use small bounded streams and prove semantics. Sustained throughput, fan-out, backlog, broker QoS/session behavior and resource budgets belong to IDR-SRV-054/056. Draft Part 3 remains disabled-by-default experimental outbound behavior under its accepted pinned profile and cannot gain normative status from these tests. **[A,X]**

### 15.4 Race and Concurrency Analysis

Prefer state machines, explicit barriers and database isolation cases over probabilistic stress. Loom/Miri are conditional tools for small custom concurrency/unsafe-sensitive primitives, not general Tokio server simulators. If Glaux needs custom lock-free or atomic coordination, that design requires a dedicated proof; standard Tokio/SQLx primitives and simpler ownership remain preferred. **[D,P]**

## 16. Command, Control, Security, Policy, DDIL, and Synchronization Test Strategy Findings

### 16.1 Command and Control Safety

Baseline TDD covers command definition/control-stream discovery, payload/profile validation, feasibility, authorization, safety policy, acceptance/idempotency, durable intent/fence, dispatch abstraction, status transitions, cancellation, timeout, unknown outcome and mandatory audit/outbox evidence. Every negative path asserts **no external effect**. **[A,P]**

PR and normal CI compile/select only a deterministic simulated gateway that records requested effects and cannot address a physical endpoint. Physical adapter credentials/endpoints are absent, network egress is restricted and the command-disabled profile proves route/claim absence or accepted safe denial. Tests may not bypass the profile with hidden configuration. Any later real-protocol proof requires explicit authorization, isolated simulator/lab target and IDR-SRV-055 safety controls. **[A,P]**

### 16.2 Security and Policy Baseline

First-implementation blocking tests include:

- authentication-context parsing and trusted-proxy boundary negatives;
- issuer/audience/algorithm/key-rotation/outage behavior against fake/local authority adapters;
- object, relationship, property and collection filtering across allowed/denied/concealed identities;
- non-inference parity where concealment requires indistinguishable results;
- policy-input completeness and default-deny on missing/invalid evidence;
- unsafe profile/configuration startup rejection;
- problem/log/metric/trace/audit redaction and label/cardinality controls;
- request/body/query/depth/time budgets and safe failure;
- command authority/safety/fence/no-effect assertions; and
- secret wrapper/debug/effective-configuration output exclusions.

Synthetic identities, labels, policies, keys and data are visibly non-operational. No real secret or controlled label appears in source, snapshots or CI artifacts. IDR-SRV-055 owns abuse depth, scanners, assessment objectives, penetration/fault campaigns and accreditation handoff. **[A,P]**

### 16.3 DDIL Tests

Use a deterministic dependency/fault schedule and explicit observation time to test fresh/stale/unknown classification, last-known values, cached schema/profile availability, partial responses, dependency health versus service posture, retry/backoff and command-disabled degraded operation. There is no global DDIL boolean or random network disconnection as the oracle. Each capability reports behavior from accepted evidence and profile. **[A,P]**

### 16.4 Synchronization Tests

Combine model/property tests and isolated two-node real-database scenarios for:

- envelope validation, source trust and replay protection;
- duplicate/idempotent apply;
- gap detection and bounded replay/resume;
- ordered and concurrent revisions;
- conflict classification, quarantine and protected diagnostics;
- tombstones/deletions and referential consequences;
- audit/lineage continuity and loop prevention;
- interrupted receive/apply/recovery; and
- convergence only where the accepted policy defines it.

Database replication, broker delivery or eventual matching counts are not sufficient semantic synchronization evidence. Exact load/fault scale belongs to IDR-SRV-054 and cross-implementation scenarios to IDR-SRV-056. **[A,P]**

### 16.5 Observability as an Assertion Aid

Tests may capture structured logs/spans/metrics to verify required signal emission, correlation and redaction, but telemetry is not the correctness oracle when a typed state/effect result exists. Assert bounded event names/fields rather than unstable prose. Never snapshot secrets, raw tokens, protected source locations or full high-cardinality identifiers. **[A,P]**

## 17. Traceability, Conformance, Fixture, Performance, Security, and Interoperability Handoff Findings

### 17.1 Traceability and Evidence Contract

The IDR-SRV-051 catalog remains the source of stable requirement, test, assertion, fixture, component, profile and gate relationships. Rust code supplies an executor binding and emits results; it does not curate reverse requirement links. The adapter records:

- stable test and assertion IDs actually started/completed;
- Cargo package/target/test symbol and exact binary digest;
- source, trace snapshot, profile, features, toolchain and dependency-lock digests;
- fixture/scenario/golden IDs and digests;
- deterministic seed/time/fault schedule where applicable;
- target/server/database/container identities;
- outcome, duration, attempt/flaky/setup classification and safe diagnostics; and
- raw artifact locations/digests and sensitivity.

JUnit is a runner view. A trace adapter joins exact assertion records to the immutable evidence graph and refuses ambiguous name-only matches. Source/test/fixture/profile/tool/build changes make dependent evidence stale under IDR-SRV-051 rules. **[A,P]**

### 17.2 Conformance Harness Contract

The standalone IDR-SRV-050 CLI consumes a resolved read-only trace/case snapshot, runs independently against the packaged server and emits `CaseResultV1`/`ConformanceRunV1`. Internal Rust tests can share scenario identifiers and expected source anchors, but not import the server's domain types, validators or response builders as the conformance oracle. Native and official/external suite results remain separate authority lanes. **[A,P]**

PR impact may request a conformance smoke subset, but release claimability requires closure over the complete applicable graph. Filtered, quarantined, stale, flaky or setup-error internal tests cannot be promoted to harness passes. **[A,P]**

### 17.3 Downstream Topic Handoff Matrix

| Consumer | Receives from IDR-SRV-052 | Must decide/prove later | Must not reinterpret |
|---|---|---|---|
| IDR-SRV-053 fixtures/corpus | loader/manifest contract, builder boundaries, exact-versus-semantic rules, seed/regression/golden governance inputs | corpus taxonomy/layout, sources/licenses, generators, specific datasets, update workflow and cleanup | current server output is not fixture truth |
| IDR-SRV-054 performance/load/streaming | functional stream scenarios, Criterion mechanics, environment/evidence fields and separate performance lane | workload models, hardware, metrics, statistical methods, thresholds, soak/stress/fan-out and regression budgets | unit/functional pass or microbenchmark is not capacity evidence |
| IDR-SRV-055 security/authorization/command | baseline enforcement/negative/no-effect tests, fake authority/gateway seams, sensitive-artifact rules | threat-driven depth, scanners, abuse/fault campaigns, lab controls, assessment objectives and accreditation handoff | synthetic test success is not security accreditation |
| IDR-SRV-056 interoperability | real-listener/package boundary, named-pair evidence schema and semantic smoke principles | exact clients/servers/versions, scenarios, cadence, required pairs and dispositions | one pair or status response is not universal compatibility/conformance |
| IDR-SRV-057 final synthesis | accepted layer/tool/tier contracts, current trace/evidence state, risks and proof results | aggregate readiness, unresolved decisions and implementation roadmap | line/test counts cannot improve missing requirement evidence |
| IDR-SRV-050 harness implementation | internal-versus-independent boundary, shared IDs/fixtures, packaging and PR/RC invocation points | concrete trace adapter and runner commands | server code cannot be harness oracle |
| IDR-SRV-051 trace implementation | Cargo/nextest inventory binding, assertion results, flaky/quarantine/setup states and artifacts | exact manifest schemas/generator/CI implementation | test names or JUnit process success are not canonical coverage |
| server implementation | red-green-refactor workflow, layout, test seams, layer ownership and PR gates | concrete code, timings and incremental rollout | testing architecture does not authorize production capabilities |
| release/operations | RC tier, image/database/tool manifests, immutable evidence and safe artifact classes | runner estate, retention, signing, release roles and operational values | artifact provenance is not correctness/security proof |

### 17.4 Required Implementation Proofs

Before the strategy is treated as operational, implementation should demonstrate these twelve proofs:

1. **Workspace boundary proof:** production packages build without dev/test-support dependencies, forbidden outward edges or first-party unsafe code; tests can still use public contracts and inline private-unit access.
2. **Obligation-first vertical slice:** one CSAPI read behavior progresses from failing trace-linked domain/application/router assertions to a packaged independent harness pass with no duplicated oracle.
3. **Test inventory proof:** Cargo/nextest discovery and trace manifests reconcile stable case/assertion IDs, parameterized cases, ignored/filtered tests and missing/orphan entries deterministically.
4. **Port contract proof:** one hand fake and real adapter pass the same semantic contract suite while real-only technology cases remain visible.
5. **Router/listener parity proof:** most HTTP assertions run by Tower `oneshot`, while selected origin/framing/stream/disconnect cases demonstrably require and pass the ephemeral real listener.
6. **Database isolation proof:** parallel SQLx tests create isolated migrated databases, exercise commit/concurrency/PostGIS behavior, clean reliably and cannot pass against stale/shared state.
7. **Deterministic async proof:** clock/ID/fault inputs and paused-time worker tests reproduce retry, timeout, cancellation and shutdown without wall-clock sleeps or leaked tasks.
8. **Golden/property/fuzz proof:** one artifact change requires reviewed semantic diff; one property failure persists/minimizes; one fuzz finding is minimized and promoted to a named regression.
9. **Flake/quarantine proof:** a controlled intermittent test is reported flaky and fails the gate; quarantine removes it from coverage/claim closure, records owner/expiry and keeps a scheduled signal.
10. **Command/security safety proof:** command-disabled and simulated profiles make physical dispatch impossible, negative paths emit no effects and sensitive values are absent from response/log/CI artifacts.
11. **Tier/evidence reproducibility proof:** two clean executions of the same pinned PR tier produce the same selected assertion set and semantic results with complete manifests; environment differences are explicit.
12. **Release closure proof:** a synthetic missing, stale, filtered, flaky or setup-error assertion prevents the affected profile claim despite green unrelated tests and high source coverage.

These are proof gates for implementation, not claims that code or CI already exists. **[P]**

### 17.5 Adoption Sequence

1. Establish toolchain/workspace/lint and test-result conventions.
2. Implement trace manifest/checker and plain `TestContext` API before custom macros.
3. Build pure domain/application/router red-green loop and first vertical slice.
4. Add real database bootstrap/isolation, migrations and adapter contracts.
5. Add deterministic async/listener/worker scenarios.
6. Introduce curated fixtures/goldens/property/fuzz with IDR-SRV-053 governance.
7. Enable advisory coverage/mutation and broad nightly lanes.
8. Connect packaged server to the independent conformance harness and later security/performance/interoperability release gates.

Blocking rollout should begin with structural correctness and the tests already reliable; advisory gaps remain visible until their prerequisites exist. **[P]**

## 18. Recommendations

1. Adopt obligation-first red-green-refactor: every behavior test names an accepted requirement, risk, regression or bounded exploratory purpose before production change.
2. Use the lowest authoritative detecting layer plus only the higher boundaries needed for public, persistence, conformance or operational behavior.
3. Make `cargo nextest` the pinned primary non-doctest CI runner, retain `cargo test` compatibility and run doctests separately.
4. Configure zero retries for blocking runs; any pass-on-retry remains flaky and ineligible for release evidence.
5. Preserve the accepted Cargo workspace and add a dev-only `glaux-test-support` crate for deterministic builders, fakes, recorders, loaders and contract suites—never production logic or a conformance oracle.
6. Keep stable test/assertion identities and relationships in the trace catalog; bind Rust cases through a plain context/manifest API before considering procedural attributes.
7. Test application decisions with narrow hand fakes, outbound HTTP adapters with wiremock, and SQL/PostGIS/migrations/concurrency only against real pinned PostgreSQL/PostGIS.
8. Use Tower in-process Router tests for the broad HTTP suite and a smaller ephemeral-listener suite for framing, origin/proxy, cancellation and streaming behavior.
9. Design clocks, IDs, fault schedules, task ownership and effect ports for determinism; prohibit correctness tests based on sleeps, mutable public services or ambient credentials.
10. Apply exact goldens only to intentionally stable lexical/artifact contracts; prefer typed/semantic assertions elsewhere and require governed semantic review for every update.
11. Use bounded PR property cases with persisted regressions, expanded nightly runs and time-boxed fuzzing whose findings are minimized into permanent regression tests.
12. Treat source coverage and mutation results as separate diagnostic signals; do not impose a global percentage as a proxy for requirement coverage.
13. Run format/build/lint/trace/unit/router/doctest/dependency gates and a bounded real-database/migration lane on every PR; expand costly profile, fuzz, mutation and conformance work by nightly/RC tier.
14. Make quarantine explicit, expiring and non-covering; distinguish assertion failure, setup/tool/infrastructure error, skip/applicability and flakiness in evidence.
15. Compile normal CI command profiles without physical gateways, real secrets or operational endpoints; assert no effect on all command/security negative paths.
16. Carry fixture content, performance thresholds, security depth and external-client selection to IDR-SRV-053 through IDR-SRV-056 while preserving their independent evidence lanes.
17. Require all twelve implementation proofs before treating the test architecture and release-closure automation as ready.

## 19. Risks, Constraints, and Open Questions

### 19.1 Risks and Controls

| Risk or constraint | Consequence | Control/owner |
|---|---|---|
| too many end-to-end cases | slow, opaque, flaky feedback and duplicated assertions | layer-selection rule, focused internal proof and small independent black-box set |
| fakes drift from real adapters | green application tests hide integration failures | reusable port contracts plus real adapter/engine cases |
| real database lane becomes bottleneck | PR delay or pressure to mock SQL | one service per job, isolated DBs, nextest groups/partitioning and measured optimization |
| Cargo feature unification hides errors | unsupported minimal/profile build passes only in workspace context | isolated feature/profile build matrix and metadata checks |
| snapshot/golden approval becomes mechanical | standards/security regression is normalized | exact-versus-semantic declaration, semantic diff, linked reviewer/reason, no CI update |
| generated/random tests are irreproducible | failures cannot be debugged or evidenced | persisted seed/minimized case, pinned generator/tool and deterministic strategies |
| retries mask race/product defects | false green evidence | zero blocking retries, flaky-result fail and explicit quarantine |
| wall-clock/external dependencies flake | nondeterministic async/stream tests | paused/injected time, barriers, finite schedules and separate real-boundary lanes |
| trace mapping duplicates code metadata | stale/orphan coverage | catalog ownership, inventory reconciliation and single executor binding |
| source coverage becomes target gaming | exercised lines lack semantic assertions | requirement/assertion coverage, risk review and targeted mutation diagnostics |
| test-support becomes shadow application | false confidence and circular dependencies | narrow permitted responsibilities and production dependency gate |
| real/controlled data leaks | legal/security exposure | synthetic labeled data, sensitivity validation, redacted failure artifacts and restricted projections |
| command tests can reach physical effects | safety incident | compile/profile/network/credential isolation and recording simulator only |
| later-topic scope is pre-empted | ungrounded budgets/tools/clients/corpora | explicit handoff ownership and separate release gates |
| tooling churn/MSRV conflicts | CI breakage or platform divergence | pinned versions/tool image, ordinary Cargo compatibility and reviewed updates |

### 19.2 Resolved Plan Questions

| Question | Resolution |
|---|---|
| Is nextest mandatory initially? | yes for CI non-doctest execution; Cargo doctest/compatibility lanes remain |
| Which DB tests run per PR? | empty bootstrap plus core and impacted real-DB contracts; full profile/concurrency/continuity expansion nightly/RC |
| How are requirement IDs embedded? | trace manifest owns mappings; plain `TestContext` references stable case IDs; custom proc macro deferred |
| Which outputs use goldens? | only intentionally stable lexical/artifact contracts; semantic/typed assertions default |
| Which property/fuzz tests start first? | high-risk pure parsers, identifiers, time/pagination/state machines and command/sync invariants; scheduled fuzz for parser/codec boundaries |
| How are flakes handled? | fail required gate; diagnose, or explicit expiring non-covering quarantine; no green-by-retry |

### 19.3 Remaining Bounded Questions

- Exact PR/nightly duration and partition budgets require measured implementation data; implementation/operations owner.
- Whether SQLx `#[sqlx::test]` database creation privileges fit every enterprise runner requires proof 6; infrastructure owner.
- Exact trace manifest serialization and result adapter are implementation details constrained by IDR-SRV-051; traceability owner.
- Corpus sources, licenses, generation and golden directory organization remain IDR-SRV-053.
- Performance environments, metrics and thresholds remain IDR-SRV-054.
- Scanner set, penetration depth and command-lab controls remain IDR-SRV-055.
- Required external clients/servers and versions remain IDR-SRV-056.
- Long-term evidence storage/retention/signing and release roles remain operations/release governance.

None prevents IDR-SRV-053 research. **[P]**

## 20. Validation Against This Plan's Success Criteria

| Success criterion | Result | Evidence |
|---|---|---|
| Rust TDD scope and multi-layer taxonomy with source anchors/prior traceability | Met | Sections 3–6 |
| repository/workspace organization and test-support boundaries | Met | Sections 7–8 |
| tooling evaluated for first implementation and full scope | Met | Section 9 |
| all required functional test strategies documented | Met | Sections 11–16 |
| CI tiers, gates, artifacts and flaky controls documented | Met | Section 10 |
| traceability, conformance, fixture, performance, security and interoperability handoffs explicit | Met | Section 17 |
| implementation/community lessons incorporated non-normatively | Met | Section 3.4 |
| recommendations decision-usable and server-bounded | Met | Sections 2, 18 and 19 |
| references explicit and reproducible | Met | Section 21 |

All planned phases and required report content are complete. Acceptance remains a project-lead action. **[P]**

## 21. References

### 21.1 Rust Language, Runner, and Async Sources

- Cargo test command: https://doc.rust-lang.org/cargo/commands/cargo-test.html
- Rust Book, testing: https://doc.rust-lang.org/book/ch11-00-testing.html
- Rust API Guidelines: https://rust-lang.github.io/api-guidelines/
- cargo-nextest documentation: https://nexte.st/
- nextest repository configuration: https://nexte.st/docs/configuration/
- nextest retries and flaky classification: https://nexte.st/docs/features/retries/
- nextest archive/partition CI features: https://nexte.st/docs/ci-features/archiving/
- Tokio testing with paused time and I/O mocks: https://tokio.rs/tokio/topics/testing
- rstest documentation: https://docs.rs/rstest/

### 21.2 Server, Database, Assertion, and Data Sources

- Axum documentation and testing examples: https://docs.rs/axum/
- Tower documentation: https://docs.rs/tower/
- reqwest documentation: https://docs.rs/reqwest/
- SQLx documentation: https://docs.rs/sqlx/
- SQLx test attribute, isolated databases, migrations and fixtures: https://docs.rs/sqlx/latest/sqlx/attr.test.html
- testcontainers-rs documentation: https://docs.rs/testcontainers/
- wiremock-rs documentation: https://docs.rs/wiremock/
- assert-json-diff documentation: https://docs.rs/assert-json-diff/
- jsonschema documentation: https://docs.rs/jsonschema/
- schemars documentation: https://docs.rs/schemars/
- PostgreSQL documentation: https://www.postgresql.org/docs/
- PostGIS documentation: https://postgis.net/documentation/

### 21.3 Property, Snapshot, Fuzz, Coverage, and Quality Sources

- proptest documentation: https://docs.rs/proptest/
- proptest test runner/failure persistence: https://docs.rs/proptest/latest/proptest/test_runner/
- insta snapshot documentation: https://insta.rs/docs/
- Criterion.rs documentation: https://bheisler.github.io/criterion.rs/book/
- cargo-fuzz repository and documentation: https://github.com/rust-fuzz/cargo-fuzz
- Rust Fuzz Book: https://rust-fuzz.github.io/book/
- cargo-mutants repository: https://github.com/sourcefrog/cargo-mutants
- cargo-llvm-cov repository: https://github.com/taiki-e/cargo-llvm-cov
- cargo-audit/RustSec: https://github.com/rustsec/rustsec/tree/main/cargo-audit
- RustSec Advisory Database: https://github.com/rustsec/advisory-db
- cargo-deny checks: https://embarkstudios.github.io/cargo-deny/checks/index.html
- Clippy documentation: https://doc.rust-lang.org/clippy/
- rustfmt repository/documentation: https://github.com/rust-lang/rustfmt
- cargo-machete repository: https://github.com/bnjbvr/cargo-machete
- cargo-udeps repository: https://github.com/est31/cargo-udeps
- Miri documentation: https://github.com/rust-lang/miri
- Loom documentation: https://docs.rs/loom/

### 21.4 CI and Evidence Sources

- GitHub Actions documentation: https://docs.github.com/actions
- GitHub Actions artifacts: https://docs.github.com/actions/using-workflows/storing-workflow-data-as-artifacts
- GitHub Actions job summaries: https://docs.github.com/actions/using-workflows/workflow-commands-for-github-actions#adding-a-job-summary
- GitHub artifact attestations: https://docs.github.com/actions/security-for-github-actions/using-artifact-attestations

### 21.5 Controlling Standards and API Sources

- OGC API - Connected Systems Part 1: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems Part 2: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems repository/artifacts: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OGC schema registry: https://schemas.opengis.net/
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- JSON Schema Draft 2020-12: https://json-schema.org/draft/2020-12
- RFC 9110, HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457, Problem Details: https://www.rfc-editor.org/rfc/rfc9457
- OGC compliance program: https://www.ogc.org/compliance/
- OGC Validator/TEAM Engine inventory: https://cite.ogc.org/teamengine/

### 21.6 Accepted Project Evidence

- Overall IDR Research Plan: [overall-idr-research-plan.md](../IDR%20Plans/overall-idr-research-plan.md)
- IDR-SRV-052 Research Plan: [idr-srv-052-rust-test-driven-architecture-and-multi-layer-test-strategy.md](../IDR%20Plans/idr-srv-052-rust-test-driven-architecture-and-multi-layer-test-strategy.md)
- Glaux Server Goal and Definition: [glaux-server-goal-and-definition.md](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- Accepted IDR-SRV-001 through IDR-SRV-051 reports: [IDR Reports](./)
- IDR-SRV-031 write/ingestion model: [idr-srv-031-server-write-and-ingestion-model-report.md](idr-srv-031-server-write-and-ingestion-model-report.md)
- IDR-SRV-035 streaming/publication strategy: [idr-srv-035-streaming-and-event-publication-strategy-report.md](idr-srv-035-streaming-and-event-publication-strategy-report.md)
- IDR-SRV-036 command lifecycle: [idr-srv-036-control-stream-and-command-lifecycle-model-report.md](idr-srv-036-control-stream-and-command-lifecycle-model-report.md)
- IDR-SRV-039 security threat model: [idr-srv-039-authentication-authorization-and-api-security-threat-model-report.md](idr-srv-039-authentication-authorization-and-api-security-threat-model-report.md)
- IDR-SRV-042 DDIL semantics: [idr-srv-042-ddil-informed-server-semantics-report.md](idr-srv-042-ddil-informed-server-semantics-report.md)
- IDR-SRV-043 synchronization boundary: [idr-srv-043-server-synchronization-and-conflict-handling-boundary-report.md](idr-srv-043-server-synchronization-and-conflict-handling-boundary-report.md)
- IDR-SRV-044 Rust/framework strategy: [idr-srv-044-rust-implementation-language-and-framework-strategy-report.md](idr-srv-044-rust-implementation-language-and-framework-strategy-report.md)
- IDR-SRV-045 modular architecture: [idr-srv-045-service-architecture-and-modularization-strategy-report.md](idr-srv-045-service-architecture-and-modularization-strategy-report.md)
- IDR-SRV-046 deployment strategy: [idr-srv-046-reference-deployment-strategy-report.md](idr-srv-046-reference-deployment-strategy-report.md)
- IDR-SRV-047 configuration strategy: [idr-srv-047-configuration-secrets-and-environment-strategy-report.md](idr-srv-047-configuration-secrets-and-environment-strategy-report.md)
- IDR-SRV-048 observability strategy: [idr-srv-048-observability-logs-metrics-and-health-check-strategy-report.md](idr-srv-048-observability-logs-metrics-and-health-check-strategy-report.md)
- IDR-SRV-049 migration/continuity strategy: [idr-srv-049-migration-upgrade-backup-and-restore-strategy-report.md](idr-srv-049-migration-upgrade-backup-and-restore-strategy-report.md)
- IDR-SRV-050 conformance harness strategy: [idr-srv-050-conformance-harness-strategy-report.md](idr-srv-050-conformance-harness-strategy-report.md)
- IDR-SRV-051 traceability strategy: [idr-srv-051-requirement-to-test-traceability-strategy-report.md](idr-srv-051-requirement-to-test-traceability-strategy-report.md)
- Research Report Template: [research-report-template.md](../../../../../Governance/research-report-template.md)

### 21.7 Non-Normative Implementation and Community Evidence

- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client/testing corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OGC CSAPI developer site: https://csapi.developer.ogc.org/

### 21.8 Evidence Limits

Official documentation and mutable package/tool pages were checked September 16, 2026. Candidate versions remain those accepted in IDR-SRV-044 until the implementation lockfile/tool image is created; versions, MSRV, output formats and platform support must be revalidated when adopted. No Glaux Server workspace, test suite, trace tool or CI pipeline existed to execute, so tier cost, database privilege fit, flake behavior and all twelve proofs are recommendations rather than observed implementation results. No performance, security, interoperability, certification or readiness claim is made. **[E,X]**

---

## Report Completion Checklist

- [x] All 21 required sections are present
- [x] Required 12-column test strategy matrix is complete
- [x] TDD timing, layer selection, repository and test-support boundaries are explicit
- [x] Every planned Rust test/quality tool family is evaluated
- [x] PR, nightly, manual and release-candidate gates and evidence are explicit
- [x] All required functional-area strategies are documented
- [x] Traceability and independent conformance integration are explicit
- [x] Flake, quarantine, snapshot, property, fuzz, coverage and mutation semantics are explicit
- [x] Command safety and sensitive-data boundaries are explicit
- [x] Twelve implementation proofs and downstream handoffs are explicit
- [x] All 9 success criteria validate as Met
- [x] Accepted by Glaux Project Lead on September 16, 2026

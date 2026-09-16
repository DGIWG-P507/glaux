# Section 050: Conformance Harness Strategy - Research Report

**Topic ID:** IDR-SRV-050<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-050 Conformance Harness Strategy](../IDR%20Plans/idr-srv-050-conformance-harness-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Conformance scope; test taxonomy; harness architecture and targets; requirement/test/evidence model; fixtures and profiles; API, schema, negative, security, dynamic-data, streaming, tasking, DDIL and synchronization coverage; CI/local workflow; official OGC and external-tool integration; downstream handoffs<br>
**Methodology Used:** Accepted-requirement extraction; normative abstract-test and current official-tool review; test-lane, target, fixture and evidence modeling; failure-state and claim-gate analysis; implementation/interoperability lesson reconciliation; option comparison and downstream synthesis<br>
**Research Time:** Approximately 42 hours of AI-assisted execution on September 16, 2026<br>
**Standards Evidence Freeze:** OGC API - Connected Systems Parts 1 and 2 Version 1.0 and tagged publication commit `8e03b236`; OGC API - Features Part 1; SensorML 3.0; SWE Common 3.0; OpenAPI 3.1.2 project baseline; JSON Schema 2020-12; RFC 9110 and RFC 9457; official sources checked September 16, 2026<br>
**Tool Evidence Freeze:** OGC Validator TEAM Engine 5.6.1 and OGC API - Features ETS 1.6; cargo-nextest 0.9.144; reqwest 0.13.5; wiremock 0.6.5; testcontainers 0.28.0; testcontainers-modules 0.15.0; insta 1.48.0; jsonschema 0.56.0; schemars 1.2.2; Schemathesis 4.27.2; pytest 9.1.1; Newman 6.2.2; versions are research evidence, not dependency approval<br>
**Accepted Project Baseline:** Twenty-five direct CSAPI conformance classes, 233 numbered requirements, five Part 1 recommendations and 240 normative abstract tests; evidence-gated build-specific declarations; one capability/contract registry; one canonical resource model; eleven deployment profiles; deterministic fixtures and explicit lifecycle administration<br>
**Document Purpose:** Define a repeatable, claim-safe conformance harness and evidence architecture without implementing it, certifying Glaux, or absorbing detailed traceability, TDD, fixture, performance, security or interoperability work owned by later topics<br>
**Author:** OpenAI Codex<br>
**Date:** September 16, 2026<br>
**Last Updated:** September 16, 2026

---

## Evidence and Decision Legend

- **[N] Normative:** approved standard, incorporated schema or normative abstract test.
- **[A] Accepted project baseline:** accepted Glaux report or governing decision.
- **[D] Direct documentation:** official product, tool or repository documentation.
- **[I] Implementation evidence:** another implementation, client or community source; informative only.
- **[T] Test evidence:** repeatable observation with stated target, fixture and conditions.
- **[E] Analysis:** reasoned synthesis from identified evidence.
- **[P] Project recommendation:** proposed Glaux decision pending acceptance of this report.
- **[X] Explicit boundary:** excluded claim or later-topic responsibility.

“Conformance run,” “pass,” “claimable,” “certified,” “interoperable,” and “secure” are distinct terms. A Glaux harness result is development evidence. It is not an OGC certification, an accreditation, a security assessment, a performance qualification or proof of interoperability with every client. **[X]**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Conformance Scope Extraction Methodology
5. Conformance Scope and Test Taxonomy
6. Harness Architecture Evaluation
7. Test Target Strategy
8. Requirement/Test/Evidence Model
9. Evidence Artifact and CI Reporting Findings
10. Fixture and Profile Requirements
11. API Behavior Coverage Findings
12. Schema/Encoding/Validation and Negative/Error Coverage Findings
13. Security/Policy, Dynamic-Data, Streaming/Event, Command/Control, DDIL, and Synchronization Coverage Findings
14. CI and Local Developer Workflow Findings
15. Official OGC Tooling and External Harness Integration Findings
16. Downstream Topic Handoff Matrix
17. Recommendations
18. Risks, Constraints, and Open Questions
19. Validation Against This Plan's Success Criteria
20. References

---

## 1. Executive Summary

Glaux needs one evidence system, not one monolithic test runner. The recommended design is a **hybrid harness led by a first-party Rust black-box CLI** that lives in the Glaux Server workspace, can test any authorized HTTP target, and emits a versioned language-neutral evidence package. An orchestration layer starts an isolated profile, applies migrations, loads a manifest-pinned scenario, runs selected cases, captures sanitized wire evidence and shuts the target down. Rust unit and in-process integration tests remain fast implementation evidence, while official OGC suites, Schemathesis, external clients, performance tools and security tools run as separately labeled lanes. **[P]**

The standards baseline is exact. Accepted IDR-SRV-008 selects all 25 direct CSAPI classes as the Glaux end-state profile and separates that target from a release's claims. Part 1 contributes 13 classes, 103 numbered requirements, five recommendations and 110 abstract tests; Part 2 contributes 12 classes, 130 requirements and 130 abstract tests. The internal harness must execute traceable adaptations of all 240 tests, preserve the five recommendation outcomes as advisory unless a Glaux profile deliberately strengthens them, honor prerequisites and conditions, and add supplemental tests where the published procedures are defective or too weak. **[N,A]**

The official tooling gap remains real as of the evidence freeze. OGC Validator advertises TEAM Engine 5.6.1 and a final OGC API - Features 1.0 ETS revision 1.6, but its published suite list does not include OGC API - Connected Systems, SensorML 3.0 or SWE Common 3.0. The official CSAPI publications contain normative abstract tests; they do not supply a publicly identified executable CSAPI suite. Glaux should therefore run the official Features ETS for inherited behavior in a separate lane, retain adapters for any OAS-version mismatch, and be ready to add an official CSAPI suite later. Internal success must be labeled “Glaux development conformance evidence,” never “OGC certified.” **[N,D,P]**

Every executable case needs stable metadata connecting source pin, conformance class, requirement or recommendation, abstract-test identifier, applicability condition, profile, target, fixture, steps, assertions, normalization, evidence, gate and known interpretation. Every result is one of `pass`, `fail`, `skip`, `warning`, `error` or `inconclusive`; only `pass` satisfies an applicable mandatory case. A skip requires a machine-readable applicability reason, retry success remains `flaky` evidence rather than an unqualified pass, and missing, redacted or unparseable evidence cannot become a pass. **[P]**

The canonical evidence output is a signed/hash-manifested JSON package. JUnit XML and Markdown are derived views. The package binds results to the server image digest, source revision, harness version, standards/profile pins, capability declaration, effective-configuration fingerprint, fixture digest, database/migration state, random seed, clock mode and target. Raw wire bytes are captured before parsing, subject to schema-driven redaction and access controls; secrets and reusable credentials are never retained. This design prevents status-only tests, hidden normalization, mutable demo data and CI presentation formats from becoming the evidence authority. **[A,P]**

No unresolved issue blocks the strategy. IDR-SRV-051 must finalize identifier/trace-record governance; IDR-SRV-052 owns Rust test-layer mechanics; IDR-SRV-053 owns the scenario corpus; IDR-SRV-054 and IDR-SRV-055 own quantitative performance and deep security verification; IDR-SRV-056 owns named external-client interoperability. Acceptance of this report authorizes none of those topics and does not implement the harness. **[X]**

## 2. Scope and Plan Alignment

### 2.1 Completed Scope

This report completes all six phases of the authorized IDR-SRV-050 plan:

- converts the accepted Parts 1/2, SensorML, SWE, HTTP, OpenAPI and project-profile baselines into conformance lanes;
- distinguishes standards, contract, integration, negative, security/profile, dynamic-data, streaming, command, DDIL, synchronization, interoperability and advisory tests;
- evaluates Rust, Python, Postman/Newman, Schemathesis and TEAM Engine approaches;
- defines targets, orchestration, result states and evidence packages;
- bounds fixtures, profile gates, CI cadence, local filtering and official-tool integration; and
- assigns exact follow-on responsibilities without executing them.

### 2.2 Explicit Boundaries

This report does not:

- rewrite the 233-requirement inventory or replace the IDR-SRV-008 profile registry;
- decide the final traceability schema, crate topology or fixture file formats;
- create an official executable test suite or assert authority to interpret OGC certification policy;
- test a live server, public demo, physical command gateway or controlled operational data;
- define throughput, latency, stress, penetration, authorization-depth or client-compatibility acceptance thresholds;
- claim draft Part 3 Publish/Subscribe conformance; a later accepted Part 3 profile may add a separate, version-pinned lane; or
- implement server or harness code.

### 2.3 Research Question Coverage

| Plan theme | Status | Evidence |
|---|---|---|
| standards and requirement scope | Complete | Sections 3–5 |
| test taxonomy and ownership | Complete | Section 5 |
| architecture options and repository placement | Complete | Section 6 |
| targets, dependencies and orchestration | Complete | Section 7 |
| requirement/test/evidence linkage | Complete | Sections 8–9 |
| fixture and profile behavior | Complete | Section 10 |
| HTTP, API, schema, negative and error behavior | Complete | Sections 11–12 |
| security, dynamic data, streaming, command, DDIL and synchronization | Complete within topic boundary | Section 13 |
| CI, local workflow and flakiness | Complete | Section 14 |
| official OGC and external harnesses | Complete | Section 15 |
| implementation lessons and downstream ownership | Complete | Sections 3.4 and 16 |

## 3. Evidence Base and Authority Classification

### 3.1 Controlling and Official Sources

| Source | Version/state | Authority and use | Evidence-freeze finding |
|---|---|---|---|
| OGC API - Connected Systems Part 1 | OGC 23-001, 1.0 | normative requirements and Annex A | 13 classes; 103 requirements; five recommendations; 110 abstract tests |
| OGC API - Connected Systems Part 2 | OGC 23-002, 1.0 | normative requirements and Annex A | 12 classes; 130 requirements; 130 abstract tests |
| CSAPI publication repository/artifacts | `v1.0.0`, commit `8e03b236` | reproducible schema, OAS, example and source pin | useful test inputs; known defects remain adapters, not silent edits |
| OGC API - Features Part 1 | 1.0 baseline/corrigenda context | inherited API/GeoJSON behavior | official executable lane exists; it does not prove CSAPI classes |
| SensorML | OGC 23-000, 3.0 | representation/model and ATS authority | schema success alone is not CSAPI mapping or semantic proof |
| SWE Common | OGC 24-014, 3.0 | component/encoding and ATS authority | JSON, Text and Binary capabilities require distinct evidence |
| OGC Validator | checked 2026-09-16 | official hosted suite inventory | TEAM Engine 5.6.1; Features ETS 1.6; no listed CSAPI/SensorML 3/SWE 3 ETS |
| TEAM Engine | current repository/docs | official OGC execution platform | Java/TestNG/CTL-capable; suite existence and authority remain suite-specific |
| OpenAPI, JSON Schema, RFC 9110, RFC 9457 | accepted editions/baselines | contract, schema, HTTP and error sources | separate syntax, contract, transport and semantics assertions required |

### 3.2 Accepted Project Evidence

The following accepted reports control this synthesis:

- IDR-SRV-001 through IDR-SRV-008 establish AEP adoption boundaries, the exact direct CSAPI inventories, all-class end-state profile, defect register and evidence-gated claims.
- IDR-SRV-009 through IDR-SRV-014 establish discovery, routes, query, negotiation, errors, OpenAPI 3.1.2 and runtime-contract parity.
- IDR-SRV-014A through IDR-SRV-014G show that fixed conformance URI lists, status-only smoke tests, live demos and client parsing do not prove standards behavior.
- IDR-SRV-015 through IDR-SRV-024 establish canonical model, lifecycle, time, provenance, SensorML/SWE, validation and schema/profile evidence.
- IDR-SRV-025 through IDR-SRV-038 establish persistence, ingestion, dynamic data, streams, commands, feasibility, safety and audit-relevant state.
- IDR-SRV-039 through IDR-SRV-043 establish authentication, authorization, zero trust, policy, audit, DDIL and synchronization constraints.
- IDR-SRV-044 through IDR-SRV-049 establish the Rust stack, modular monolith, deployment profiles, configuration, observability and lifecycle administration used by the harness.

### 3.3 Current Candidate Tool Evidence

| Tool | Evidence-freeze version | Suitable role | Limitation/boundary |
|---|---:|---|---|
| Cargo/libtest | Rust 1.98.1 project baseline | unit, integration and in-process implementation tests | not a black-box protocol claim by itself |
| cargo-nextest | 0.9.144 | Rust scheduling, filtering, partitioning, retries and JUnit | runner metadata is not the canonical conformance record |
| reqwest | 0.13.5 | HTTP client for first-party black-box executor | default normalization must not replace captured wire facts |
| wiremock | 0.6.5 | dependency fakes and callback verification | cannot substitute for server-under-test behavior |
| testcontainers/testcontainers-modules | 0.28.0/0.15.0 | isolated dependency integration | container availability is profile/environment dependent |
| insta | 1.48.0 | reviewed stable snapshots | broad response snapshots become brittle and hide semantic intent |
| jsonschema/schemars | 0.56.0/1.2.2 | offline validation and Glaux-owned schema work | published OGC schemas remain pinned external authority |
| Schemathesis | 4.27.2 | advisory OpenAPI example/coverage/fuzz/stateful lane | generated checks do not encode all normative CSAPI semantics |
| pytest | 9.1.1 | viable external/prototype harness host | adds a second primary implementation ecosystem if made authoritative |
| Newman | 6.2.2 | portable curated smoke collections and JSON/JUnit reports | collection model is weak for 240 conditional ATS mappings and typed evidence |

All versions require dependency, license, MSRV, security and reproducibility review at implementation time. Version recency is not an adoption decision. **[X]**

### 3.4 Implementation and Community Lessons

Informative evidence consistently supports the following test requirements:

- OSH's broad resource and WebSocket tests coexist with static, inaccurate conformance claims; capability declarations must derive from passing evidence.
- Connected Systems Go's end-to-end fixtures show the value of deterministic setup and real HTTP assertions, but implementation output is not a normative oracle.
- pygeoapi/52°North exposed representation-dependent populations, incomplete OpenAPI and no complete automated CSAPI suite; multi-format semantic equivalence and runtime/OAS parity must be explicit.
- SECD exposed accepted-but-ignored filters, HTML 404 fall-through, noncanonical conformance URIs and lifecycle gaps; every filter needs known-hit/known-miss controls and wire evidence.
- OS4CSAPI testing found that a shell's one-element-array coercion caused a false server finding; raw response bytes must precede parsing and tool transformations.
- External clients may reject valid recursive SensorML/SWE structures or discard links. The server harness must test standards correctness independently; named-client tolerance belongs in interoperability evidence.

These lessons create supplemental cases. They do not change the normative requirements. **[I,P]**

## 4. Conformance Scope Extraction Methodology

### 4.1 Extraction Pipeline

The harness scope is derived through the following controlled chain:

`source edition/digest → requirement or recommendation → class and prerequisites → condition/applicability → normative abstract test → interpretation/adapter ledger → supplemental risks → executable case(s) → target/profile/fixture → result/evidence → build claim gate`

Each edge must be queryable. A source or adapter change invalidates affected evidence until those cases rerun. A server capability cannot enter `/conformance` because a route, serializer or configuration switch exists; it enters only when the closed evidence graph passes for that build and profile. **[A,P]**

### 4.2 Authority Rules

1. Normative requirement text and incorporated artifacts control the obligation.
2. Annex A supplies required abstract-test intent but cannot silently weaken a requirement.
3. Published contradictions remain in a versioned interpretation/adapter ledger with both original and implemented expectations.
4. A Glaux supplemental test may strengthen product quality but must identify its project authority rather than masquerade as an OGC test.
5. Implementation and client findings produce regression scenarios, not new standards requirements.
6. An official executable result retains its own suite/version/target identity; Glaux does not rewrite it into an internal pass.
7. Every claim is build-, profile- and evidence-package-specific.

### 4.3 Completeness Units

The harness tracks completeness at four levels:

- **source coverage:** every selected requirement, recommendation, abstract test, prerequisite and condition has a record;
- **case coverage:** each applicable source record has executable positive and necessary negative/semantic cases;
- **run coverage:** every selected case has an honest terminal result and evidence for the target;
- **claim coverage:** prerequisites and all mandatory lanes close for the exact class/build/profile.

A high line-coverage percentage, a schema-valid document or a successful smoke request is not a substitute for any of these levels. **[P]**

## 5. Conformance Scope and Test Taxonomy

### 5.1 Required Outcome Semantics

| Result | Meaning | Claim effect |
|---|---|---|
| `pass` | every applicable assertion completed and succeeded | satisfies this case only |
| `fail` | target behavior contradicted an assertion | blocks affected mandatory claim/gate |
| `skip` | case proved inapplicable under a recorded condition | neutral only when the condition is valid and reviewed |
| `warning` | recommendation/advisory or explicitly nonblocking concern | visible; does not satisfy a mandatory case |
| `error` | harness, dependency, fixture or evidence collection failed | blocks claim; never target pass |
| `inconclusive` | evidence cannot distinguish conforming from nonconforming behavior | blocks claim; prompts case/fixture repair |

Retries add an orthogonal `flaky=true` fact. A retry pass may support diagnosis, but the original failure and all attempts remain in evidence; release-claim gates require a stable clean execution. **[P]**

### 5.2 Conformance Harness Matrix

| Test category | Requirement/conformance source | Test target | Fixture dependency | Profile applicability | Assertion type | Evidence artifact | CI gate status | Related downstream topic | Notes / unresolved issues |
|---|---|---|---|---|---|---|---|---|---|
| Part 1 normative ATS | OGC 23-001; 13 classes, A.1–A.110 | black-box HTTP target | deterministic resource graph | claimed Part 1 classes | procedural, HTTP, link, schema, semantic | case JSON plus wire capture | blocking for claim/release | 051–053 | preserve two helpers and conditions |
| Part 1 recommendation tests | five `/rec/...` rows in OGC 23-001 | black-box | recommendation-specific | all relevant | semantic/advisory | warning result | advisory unless Glaux profile elevates | 051 | never silently convert to requirement |
| Part 2 normative ATS | OGC 23-002; 12 classes, A.1–A.130 | black-box HTTP target | streams, observations, controls, events | claimed Part 2 classes | procedural, lifecycle, schema, semantic | case JSON plus wire/event capture | blocking for claim/release | 051–053 | copied/mistargeted ATS steps need ledger adapters |
| inherited OGC API Features | Features Part 1 plus official ETS 1.6 | Compose/CI external URL | ETS-compatible collection | inherited class/profile | official TestNG suite | untouched official output plus wrapper manifest | separate blocking lane when claimed | 051,052 | OAS 3.0/3.1 seam must be explicit |
| SensorML representation | SensorML 3.0 and Part 1 SensorML class | HTTP plus offline validator | System/Procedure/Deployment/Property corpus | SensorML enabled | schema, mapping, semantic equivalence | validation records/digests | blocking for enabled claim | 051–053 | no Sampling Feature overclaim |
| SWE component/encoding | SWE Common 3.0 and Part 2 JSON/Text/Binary classes | HTTP plus codec executor | scalar/aggregate/edge corpus | enabled encoding | schema, round trip, semantic/value | decoded-value and validation evidence | blocking per claimed encoding | 051–053 | binary remains separately claim-gated |
| landing/conformance/OpenAPI | Common/Features, CSAPI profile, IDR-009/014 | black-box | capability variants | every API profile | exact URI, relation, reachability, parity | wire plus registry/OAD diff | PR and release blocking | 051,052 | declaration is tested output, not evidence source |
| link/resource graph | Parts 1/2 and IDR-010/017 | black-box | discriminating connected graph | resource classes enabled | traversal, identity, reciprocal membership | graph trace | blocking | 051,053 | broken links and identity substitution fail |
| query/filter/pagination | Parts 1/2, Features, IDR-011/026 | black-box/database-backed | known-hit/miss/multipage data | query classes enabled | metamorphic set/order/count assertions | request series and normalized result IDs | blocking | 051,053,054 | HTTP 200 alone never passes |
| negotiation/media types | HTTP, CSAPI encoding classes, IDR-012 | black-box | same logical population in each format | media capability variants | status/header/body/equivalence | raw bytes, headers, decoded facts | blocking | 051–053 | exact `Vary`, 406 and 415 cases |
| problem/error behavior | RFC 9110/9457, IDR-013 | black-box | invalid/denied/missing cases | all profiles | status, media, safe fields, correlation | redacted wire/problem record | blocking | 051,053,055 | avoid leaked existence/details |
| write lifecycle | CSAPI transaction classes and IDR-018/019/029 | isolated black-box target | exclusive namespace | write-enabled only | create/read/list/update/delete/cascade/cleanup | lifecycle transcript and final-state proof | blocking where claimed | 051–053 | draft Features Part 4 pin/qualification retained |
| schema/profile activation | SensorML/SWE/JSON Schema, IDR-023/024 | offline plus black-box | pinned schema/profile packages | capability-specific | syntax, closure, semantic, compatibility | validation ladder record | blocking | 051–053 | no network schema fetch |
| security/policy profile | accepted IDR-039/040 | controlled target | synthetic identities/policy pairs | auth/policy profiles | allow/deny/conceal/non-inference | decision/wire/audit correlation | core subset blocking; deep suite later | 051,053,055 | no production IdP required |
| dynamic data | Part 2 and IDR-034 | black-box/database-backed | known time series | dynamic-data profiles | time, latest, status, ingestion semantics | temporal transcript/state hashes | blocking where claimed | 051–054 | virtual/fixed clock required |
| streaming/events | project streaming contract, underlying Part 2 resources | black-box stream endpoint | finite scripted publisher | streaming-enabled | framing, order, resume, heartbeat, cancellation | frame/event transcript | profile blocking; not direct CSAPI transport claim | 052–054 | no sleep-based success criteria |
| command/control/feasibility | Part 2 and IDR-036–038 | simulated gateway only | deterministic command state machine | command-sim only | validation, transition, cancellation, denial, audit | command/effect/audit transcript | blocking for claimed command classes | 051–053,055 | physical effects prohibited |
| DDIL/degraded behavior | accepted IDR-042 | controlled dependency fault target | proxy/clock/cache scenarios | DDIL simulation | service posture, stale/partial/last-known semantics | fault schedule plus response evidence | Glaux profile blocking | 051–055 | not an OGC class by itself |
| synchronization/conflict | accepted IDR-043 | isolated two-node target | duplicates/gaps/conflicts/tombstones | sync profile | effectively-once apply, conflict/quarantine/lineage | envelopes, state digests, audit | Glaux profile blocking | 051–055 | no database-replication substitution |
| route/OAD/capability parity | IDR-008/009/014/044–047 | build artifact plus live target | capability/profile variants | every release | bidirectional set/metadata diff | parity report | PR/release blocking | 051,052 | disabled routes cannot remain advertised |
| generated OpenAPI probing | OpenAPI 3.1.2 project baseline | isolated black-box target | safe generated values | CI/nightly | schema-derived example/negative/property checks | Schemathesis JSON/JUnit/HAR | advisory then promoted by named rule | 052–055 | generator bug is not server failure by default |
| external-client smoke | named pinned client contract | Compose/public read-only target | client-specific corpus | interoperability | parse, traversal, semantic completeness | client logs and wire capture | nonblocking here | 056 | never generalize one client to conformance |
| performance-sensitive variants | same functional cases plus workload contract | performance deployment | controlled volume | performance profile | latency/throughput/resource/error budget | benchmark result plus functional proof | outside conformance pass | 054 | functional pass remains separate |
| manual review | AEP/profile documents, UI or unavailable automation | immutable artifacts | review packet | release-specific | two-person checklist/rationale | signed review record | blocking only if requirement explicitly manual | 051,057 | automation status must stay visible |
| future official CSAPI ETS | future OGC release | official supported target | suite-defined | exact official edition | official suite | untouched native output plus manifest | separate official lane | 051,056,057 | map and run; do not silently replace internal regressions |

### 5.3 Lane Ownership

The **conformance harness** owns case discovery, selection, orchestration, black-box execution, evidence packaging and claim evaluation. Server unit/integration tests prove internal invariants but cannot replace black-box wire cases. Performance, deep security and named-client suites may reuse fixtures and evidence IDs, but their outcome types remain separate. This prevents a fast response, a scanner result or a client success from being mislabeled as an OGC requirement pass. **[P]**

## 6. Harness Architecture Evaluation

### 6.1 Options

| Option | Strengths | Costs/risks | Decision |
|---|---|---|---|
| Rust integration tests only | same toolchain; fast; strong internal access | couples proof to implementation; weak external URL and evidence packaging; risks tautological shared types | retain for implementation layers, reject as sole harness |
| standalone Rust CLI | typed metadata/results; deployable binary; exact project stack; external target support | must build reporting, orchestration and plugin seams deliberately | select as first-party core |
| Python/pytest primary | mature fixtures/plugins; quick protocol scripting | second authoritative ecosystem, dependency/runtime and model duplication | retain for external experiments, not primary |
| Postman/Newman primary | accessible collections; portable CLI; JSON/JUnit reports | weak conditional trace graph, schema semantics, lifecycle composition and canonical evidence control | optional curated smoke export only |
| Schemathesis primary | OpenAPI-driven examples, negative, fuzz and stateful discovery | cannot infer CSAPI normative semantics, policy, graph and exact fixture expectations | select as separate advisory/generated lane |
| TEAM Engine-only | official platform and future certification alignment | no identified CSAPI ETS; JVM stack; does not cover Glaux profiles | integrate official suites as separate lane, reject as sole harness |
| hybrid first-party plus external lanes | best authority separation, coverage and future compatibility | orchestration and evidence federation complexity | **recommended** |

### 6.2 Recommended Component Model

The initial workspace should contain logical components whose final crate names remain IDR-SRV-052's decision:

1. **case catalog:** immutable case definitions, standards/profile pins, applicability and filters;
2. **black-box executor:** HTTP, streaming and approved protocol adapters using wire-level models independent of server domain/serialization crates;
3. **assertion library:** status, header, relation, JSON/schema, set/order, temporal, lifecycle and evidence-safe diagnostic primitives;
4. **scenario interface:** requests an identified fixture state but does not own the corpus;
5. **target adapter:** external URL, spawned process or Compose profile with readiness and capability facts;
6. **orchestrator:** migration, seed, reset, fault proxy, simulated command gateway and cleanup sequencing;
7. **evidence writer:** canonical JSON package, hashes, restricted captures and derived JUnit/Markdown;
8. **claim evaluator:** closes prerequisites, applicability and required lanes without changing raw case results; and
9. **external-lane adapters:** TEAM Engine/ETS, Schemathesis and later named tools, preserving native output.

The CLI and case/evidence schemas should remain publishable as a standalone binary/image, but live in the server repository initially so contract changes and gates are atomic. A separate repository is justified only when independent release cadence, external implementation use and governance outweigh change-coupling cost. **[P]**

### 6.3 Independence and Anti-Tautology Rules

- Black-box cases do not deserialize through the server's wire structs or call application services.
- Test expectations derive from pinned standards, project profile and fixture invariants, not current server output.
- Shared official schema bytes and stable identifiers are allowed; shared behavior code is not evidence independence.
- A generated expected document must be reviewed or independently validated; “server generated both sides” is prohibited.
- Case selection comes from the profile/claim registry, while the server's `/conformance` output is an assertion target.
- External tools retain native failures even when a Glaux wrapper cannot interpret them.

## 7. Test Target Strategy

### 7.1 Target Classes

| Target | Primary purpose | Evidence status | Mutation rule |
|---|---|---|---|
| pure unit/component | parsers, codecs, predicates, state machines | implementation evidence | in-memory/synthetic only |
| in-process API | fast route/middleware/contract feedback | integration evidence; not release claim alone | isolated transaction/store |
| spawned local server | developer black-box debugging | valid black-box evidence when manifest complete | dedicated database/namespace |
| Docker Compose reference profile | reproducible full-stack conformance | canonical development/release candidate | fresh volumes and explicit lifecycle commands |
| CI ephemeral deployment | blocking automated evidence | canonical CI evidence | run-scoped isolated resources |
| public demo | availability/read-only smoke | observational/advisory | no destructive or tasking writes |
| authorized external URL | compatibility/official suite target | target-qualified external evidence | default read-only; writes require explicit disposable authorization |
| external CSAPI implementation | comparison/interoperability | informative only for Glaux | never mutate without owner authorization |

### 7.2 Canonical Run Lifecycle

1. resolve and verify the release, standards, schema, harness and fixture manifests;
2. create an isolated target identity, network, database and artifact namespace;
3. execute explicit same-image migrations and targeted bootstrap/fixtures;
4. verify readiness, effective profile, release digest and clean baseline invariants;
5. acquire an exclusive mutation lease where write cases are selected;
6. run cases in declared dependency groups with fixed/virtual clock and recorded seed;
7. capture target, dependency, audit and wire evidence with sensitivity controls;
8. run cleanup and prove expected final state, including no pending physical effects;
9. finalize hashes, canonical JSON, derived reports and claim evaluation; and
10. destroy the isolated target or mark cleanup failure as a run error.

Setup or cleanup failure cannot be charged to the server as a normative fail unless the tested requirement owns that behavior. It does block the claim because the evidence is incomplete. **[P]**

### 7.3 Dependency Strategy

PostgreSQL/PostGIS runs as the real accepted persistence dependency in canonical black-box suites. External identity/policy, callback, broker, clock, command gateway and upstream dependencies use deterministic fakes only where their boundary—not their product implementation—is under test. A fake records every interaction and can inject explicit delays, errors, disconnects and stale states. Compose supplies the release-shaped reference run; Testcontainers may accelerate lower integration layers. **[A,P]**

## 8. Requirement/Test/Evidence Model

### 8.1 `ConformanceCaseV1`

Each case definition must contain at least:

- stable Glaux case ID and schema version;
- authority lane: normative, inherited official, Glaux profile, advisory, generated or interoperability;
- exact source edition/digest, clause, requirement/recommendation URI, conformance class and ATS identifier;
- prerequisite classes, activation condition, supported applicability values and interpretation/adapter ID;
- title, purpose, positive/negative classification and test category;
- supported target types, profile selectors and required capabilities;
- fixture scenario ID/version/digest, setup state and cleanup invariant;
- ordered action/step IDs, request templates and non-secret variable sources;
- assertion IDs, comparison semantics, permitted normalization and expected evidence;
- sensitivity/capture policy, timeout category and deterministic clock/randomness needs;
- CI tier, claim-gate role, automation status, owner and downstream topic; and
- provenance, review state, supersession and known issue references.

IDR-SRV-051 owns the final schema and identifier governance. This report requires these semantics regardless of storage syntax. **[P]**

### 8.2 `CaseResultV1`

Every case result records the immutable case digest, selected applicability evaluation, start/end times, target/run identity, attempt sequence, terminal result, assertion outcomes, sanitized diagnostics, evidence object hashes, cleanup outcome and tool versions. A wrapper cannot overwrite a child assertion; aggregation is monotonic toward the worst applicable state. **[P]**

### 8.3 `ConformanceRunV1`

The run manifest binds:

- harness source/release/image digest and command line;
- server source/release/image/SBOM/provenance identity;
- standards, schema, profile, interpretation and capability-registry versions;
- target base URL identity, deployment profile and redacted effective-configuration fingerprint;
- migration/seed state, fixture manifest/digest, node identity and recovery lineage where relevant;
- selected case query and complete resolved case list;
- clock mode, timezone, random seed, network/fault schedule and dependency identities;
- canonical case results, external-lane manifests and artifact hashes;
- claim evaluation per conformance class/profile; and
- signer/attestation data when release policy enables it.

### 8.4 Claim Evaluation

A class is `claimable` only when the exact build/profile:

1. implements and advertises its prerequisites coherently;
2. has no unresolved applicable mandatory requirement or condition;
3. passes every applicable normative abstract-test adaptation;
4. passes required inherited, schema, negative, lifecycle and parity lanes;
5. has no `fail`, `error`, `inconclusive` or invalid skip in the closed evidence graph;
6. retains a reproducible evidence package; and
7. has not been invalidated by a later source, profile, fixture, harness or release change.

The claim evaluator produces a decision and reasons; it never changes test results. The deployed `/conformance` response must equal the set of classes claimed for that build/profile, not the end-state target list. **[A,P]**

## 9. Evidence Artifact and CI Reporting Findings

### 9.1 Evidence Package

The canonical run output is a directory/archive described by `ConformanceRunV1` and a content manifest. Recommended logical content is:

```text
run.json
manifest.json
claims.json
cases/<case-id>/result.json
cases/<case-id>/requests/<step-id>.*
cases/<case-id>/responses/<step-id>.*
cases/<case-id>/validation/*.json
external/<lane-id>/native/*
logs/sanitized.ndjson
reports/junit.xml
reports/summary.md
```

This is a logical contract, not a required on-disk spelling. `run.json`, case results and the content manifest are authoritative machine evidence. JUnit supports CI visualization and Markdown supports review; neither may contain facts absent from the canonical package. **[P]**

### 9.2 Capture and Comparison Rules

1. Capture status, headers and raw body bytes before decoding.
2. Record the declared and detected media type separately; a successful parse does not repair a false `Content-Type`.
3. Validate syntax/schema, then assert semantics through named paths and graph facts.
4. Treat arrays as ordered only where the contract defines order; otherwise compare identities as sets or multisets.
5. Normalize only case-declared nondeterminism such as generated IDs or timestamps, preserve original values, and record the normalization rule.
6. Use exact golden bytes for immutable published artifacts and deliberately stable output; use semantic assertions for ordinary API resources.
7. Prohibit wildcard “ignore everything extra,” arbitrary timestamp tolerances and success-status ranges that erase specific obligations.
8. Hash all evidence objects and identify truncation, redaction or omission explicitly.

### 9.3 Sensitivity and Retention

Capture policy derives from the accepted configuration, policy, audit and observability classifications. Test credentials are short-lived and synthetic. Authorization headers, cookies, secret references, private keys, bearer tokens and sensitive fixture fields are removed or irreversibly tokenized before ordinary CI upload. Restricted raw captures, if a test genuinely needs them, use separate encrypted access-controlled storage and appear in the public manifest only by policy-safe digest and classification. **[A,P]**

Evidence retention periods, signing service, artifact repository and legal custody are operational decisions. This report requires release evidence to be immutable and addressable for the supported release lifetime, but does not invent numeric retention. PR artifacts may follow shorter CI policy if the run manifest remains linkable. **[X]**

### 9.4 Reproducibility Test

A release candidate must be rerunnable from its evidence manifest in a clean environment. Reproduction means the same resolved case set, sources, fixture invariants and outcome semantics; generated IDs and elapsed times need not be byte-identical. A difference report must distinguish target regression, tool/source drift, nondeterministic fixture, environment failure and permitted volatile evidence. **[P]**

## 10. Fixture and Profile Requirements

### 10.1 Scenario Families

The harness requires named scenario capabilities; IDR-SRV-053 owns the concrete corpus:

| Scenario family | Minimum discriminating state | Primary use |
|---|---|---|
| discovery/core | landing, exact declarations, OAD, collections and valid links | entry point, claim and contract tests |
| connected graph | parent/subsystem, deployment/subdeployment, procedure, sampling feature and property links | recursion, nesting, identity and graph tests |
| query/pagination | empty, one-item, multi-page, spatial/nonspatial, temporal, keyword/property hit/miss cases | filters, counts, ordering and pagination |
| representation | same resources in every enabled GeoJSON/SensorML/JSON form | negotiation and semantic equivalence |
| write lifecycle | isolated mutable resources with references, revisions and tombstones | CRD/update/cascade/conflict/cleanup |
| SWE contract | scalar, record, vector, choice, arrays, nil, quality and schema revisions | schema, mapping, JSON/Text/Binary codec tests |
| dynamic time series | fixed phenomenon/result/ingest times, late/duplicate values and latest/status projections | observation, latest, status and ingestion |
| streaming | finite event schedule with IDs, replay window and disconnect points | order, resume, heartbeat and backpressure semantics |
| command simulator | safe control schemas, feasibility outcomes, transitions, cancellation and unknown result | command/control lifecycle and audit |
| policy identities | allowed, denied and concealment principals over identical objects | non-inference, filtering and errors |
| DDIL/fault | controlled dependency, time, capacity and connectivity transitions | stale, partial, last-known and posture rules |
| synchronization | two-node duplicates, gaps, concurrent revisions, conflicts and tombstones | apply, quarantine, lineage and recovery |
| negative/adversarial | malformed, schema-invalid, graph-invalid, oversized and implementation-observed variants | rejection, diagnostics and resource limits |

### 10.2 Fixture Manifest

Each scenario records a stable ID/version, source/license/provenance, standards/profile pins, exact files and hashes, setup command, expected resource IDs and graph, clock/time zone, permitted generated values, security classification, profiles, cleanup contract and intentionally invalid facts. Official examples must be revalidated before becoming positive fixtures; known defective examples remain negative fixtures with the defect identified. **[A,P]**

### 10.3 Profile Safety

- **local development:** reset is allowed only against an environment-marked disposable database; all evidence is developer-qualified.
- **CI/conformance:** fresh isolated state, fixed time and no external effects are mandatory.
- **public demo:** read-only smoke by default; no reset, destructive mutation, credential probing or command submission.
- **interoperability:** exact client/server versions and explicit mutation authority; results are named-pair evidence.
- **command-disabled:** command routes and claims must be absent or produce the accepted safe behavior; tests do not bypass the profile.
- **command-simulated:** only the deterministic fake gateway can receive effects; physical adapters are impossible to select.
- **streaming-enabled:** finite deterministic publisher and bounded timeouts; no public/live feed dependency.
- **DDIL/synchronization:** isolated networks/nodes, synthetic data and recoverable fault schedules.
- **operational-reference:** no fixture reset or mutation unless an isolated clone and explicit operator procedure establishes safety.

## 11. API Behavior Coverage Findings

### 11.1 Discovery, Claims and OpenAPI

The harness must verify both the representation and the behavior behind every advertised relation and class:

- landing responses negotiate every claimed form and expose correct `self`, service description/documentation, conformance and data relations;
- `/conformance` contains exact, unique, accepted class URIs for the build/profile and closes prerequisites;
- OpenAPI is reachable through the advertised link, parses as the claimed OAS edition, has closed references and matches the enabled route/method/media/error/security surface in both directions;
- public origins and links remain correct behind the configured proxy trust model; and
- planned, disabled, experimental, draft Part 3 and unavailable capabilities are not advertised as approved conformance.

The OAD and conformance response are assertion targets generated from the accepted capability registry. They are not allowed to select which tests “count” without comparison to the release profile and evidence graph. **[A,P]**

### 11.2 Collections, Resources and Navigation

Every resource family needs root, item, nested and relationship cases for empty and populated states. Cases assert canonical identity, local ID scope, unique URI identity, membership, reciprocal links, resource kind, content type and policy-shaped visibility. Hierarchy cases cover direct versus recursive results, cycles rejected by the accepted model, aggregation from subsystems/subdeployments and stable pagination across the chosen ordering contract. **[N,A,P]**

The same logical resource must retain identity and relationship facts across every enabled representation. Media negotiation may change syntax, not silently route to a different population or backing provider. A self link that resolves to a representation with another identity fails even if both requests return 200. **[A,I,P]**

### 11.3 Query, Filter, Sorting and Pagination

Each query case uses discriminating data and at least four controls where meaningful:

1. unfiltered baseline;
2. known-hit value;
3. known-miss value; and
4. invalid or unsupported value.

Assertions cover exact result IDs, membership, counts/extent where applicable, order, links with preserved parameters, page continuity and duplicate/omission behavior. Compound filters add intersection/union expectations; temporal and spatial filters use boundary facts; relationship filters verify traversal scope. A 200 response with unchanged results cannot pass a filter case. **[A,P]**

Where the standards permit implementation choices, the case cites the selected Glaux profile. It does not hard-code one implementation's default page size, optional count behavior or incidental database order. **[N,P]**

### 11.4 Content Negotiation and HTTP

Cases cover no `Accept`, exact supported types, weighted alternatives, wildcards, parameters, unsupported `Accept`, correct request `Content-Type`, unsupported request media, `Vary`, safe/idempotent method semantics, conditional requests where selected and accurate status/body rules. The harness preserves RFC semantics separately from format schema semantics and tests `406` versus `415` explicitly. **[N,A]**

### 11.5 Writes and Lifecycle

Mutation cases execute complete observable lifecycles, not isolated status checks:

`create → verify Location/body → canonical read → collection/nested/query visibility → representation equivalence → allowed update/replace → conflict/precondition cases → delete/cascade/tombstone behavior → cleanup proof`

Generated IDs are captured and then asserted, not normalized away. Duplicate/idempotency cases prove one logical effect. Asynchronous resources use bounded polling against explicit state contracts or event barriers; sleeps are not an assertion. **[A,P]**

## 12. Schema/Encoding/Validation and Negative/Error Coverage Findings

### 12.1 Validation Ladder

The harness mirrors but independently probes the accepted validation stages:

1. transport framing and media type;
2. syntax and duplicate-key/lexical rules;
3. pinned published JSON/GeoJSON/SensorML/SWE schema;
4. CSAPI wrapper and mapping table;
5. canonical domain and relationship invariants;
6. selected profile and compatibility rules;
7. authorization/policy and source-trust admission;
8. persistence/effect preconditions; and
9. generated-output and evidence validation.

Each invalid fixture names the intended failing layer and a valid near-neighbor. This prevents a malformed document from “testing” a deeper semantic rule it never reaches. **[A,P]**

### 12.2 Offline Source Control

All normative schemas, examples and overlays are retrieved through the IDR-SRV-024 pinned package, not from mutable network URLs during CI. The manifest records original URI, edition, bytes and digest. Glaux-owned overlays cannot edit vendored OGC bytes. A resolver/network failure is a harness error; a target response that violates the pinned applicable schema is a target fail. **[A,P]**

### 12.3 Encoding Assertions

- **GeoJSON:** type, geometry, identifiers, links, resource mapping and semantic equality to alternate representations.
- **SensorML JSON:** applicable concrete class, required identity/metadata, Part 1 mapping, unordered-object parsing, preserved permitted extensions and canonical equivalence.
- **SWE JSON/Text/Binary:** wrapper format, component graph, encoding descriptor, full consumption, nil/quality/time mapping, round-trip typed values and exact stream-schema revision.
- **Problem Details:** media type, status coherence, stable type/code, safe detail, validation paths and correlation without source/secret leakage.
- **OpenAPI:** strict parse, reference closure, schema dialects, examples, operation IDs, server URLs, security descriptions and runtime parity.

Binary failures retain safe offsets and digests, not uncontrolled payload dumps. Tool inability to process a standards-permitted recursive schema is classified as harness error/inconclusive, never automatic server failure. **[A,I,P]**

### 12.4 Negative Families

The blocking corpus covers malformed JSON and framing; duplicate keys where prohibited; wrong/missing media headers; unsupported negotiation; unknown and malformed parameters; invalid geometry/time/range/pagination; missing/hidden resources; wrong resource kind; broken/cyclic references; invalid schema evolution; prohibited lifecycle transitions; duplicate/idempotency conflicts; invalid observation values; invalid command/feasibility payloads; policy-hidden data; stale/unavailable dependencies; size/depth/count/regex budgets; and safe rejection under overload. **[A,P]**

Expected statuses and bodies come from the exact requirement plus the accepted Glaux error profile. Tests do not accept any `4xx` when a specific `400`, `404`, `406`, `409`, `412`, `415` or policy-concealment outcome is required. **[N,A]**

## 13. Security/Policy, Dynamic-Data, Streaming/Event, Command/Control, DDIL, and Synchronization Coverage Findings

### 13.1 Security and Policy Core

The conformance harness owns a bounded set of profile-verification cases:

- anonymous/authenticated behavior for each enabled profile;
- object, nested-resource, query, field, link and mutation enforcement;
- authorized-view counts, extents, latest values and pagination without inference leaks;
- exact concealment versus forbidden outcomes from the accepted policy;
- expired/invalid synthetic credentials and non-secret safe diagnostics;
- route/OAD/conformance differences when a capability is disabled; and
- audit correlation for security-sensitive actions without treating audit as a response oracle.

These cases use a deterministic fake identity/policy authority and synthetic labels. Token cryptanalysis, IdP interoperability, adversarial security scanning, exhaustive role/policy combinatorics and physical command safety belong to IDR-SRV-055. **[A,X]**

### 13.2 Dynamic Data

Observation and status cases bind each value to a specific DataStream and schema revision; assert phenomenon, result and ingest time independently; cover single/batch acceptance; prove duplicate/idempotent behavior; and compare historical, window, latest and status projections against known data. Late, stale, nil, invalid and out-of-order values receive explicit states rather than collapsing into absence. **[A,P]**

Ingestion evidence records acknowledgment, durable state, outbox/event intent, audit correlation and rejected-item detail according to the accepted atomicity contract. It does not treat successful HTTP acceptance as proof of later publication or latest-value correctness. **[A]**

### 13.3 Streaming and Event Tests

Streaming cases use a finite script, virtual/fixed time where possible, explicit subscription-ready barrier, event IDs and bounded deadlines. Assertions cover initial snapshot/tail boundary, ordering contract, duplicate/gap behavior, replay/resume, heartbeat, schema binding, authorization expiry, disconnect, cancellation and slow-consumer outcome. A fixed sleep followed by “received something” is prohibited. **[A,P]**

Streaming transport evidence is a Glaux profile lane because Parts 1 and 2 define no standalone live-transport class. Underlying DataStream, Observation, Command and System Event resources still require their normative Part 2 evidence. Draft Part 3 tests, if later authorized, must identify the exact draft pin and cannot be blended into approved Part 1/2 claims. **[A,X]**

### 13.4 Command/Control and Feasibility

All automated command tests use the command-simulated profile with a hard configuration prohibition on physical adapters. The deterministic gateway supports accepted, rejected, delayed, failed, cancelled and ambiguous/unknown outcomes and records an effect token without touching equipment. Cases cover ControlStream discovery, exact schema revision, payload validation, feasibility identity/expiry, submission, idempotency, state transitions, status/result links, cancellation rules, policy/safety denial, outbox delivery and audit evidence. **[A,P]**

A feasible response is not authorization, submission or success. A restored historical command is never redispatched. Public-demo and command-disabled profiles prove absence/denial rather than bypassing gates. Deep abuse, authorization and safety matrices remain IDR-SRV-055. **[A,X]**

### 13.5 DDIL Profile

A test-controlled dependency proxy and clock drive connectivity, dependency, authority, data, time, synchronization and capacity dimensions independently. Cases assert the operation-specific service posture plus validity/current/freshness/last-known/cached/delayed/tentative/unavailable/unknown/partial semantics accepted in IDR-SRV-042. Global “offline” booleans and response-delay timing guesses are prohibited. **[A,P]**

The evidence package includes the exact fault schedule and dependency observations. A degraded but standards-correct response can pass the applicable Glaux profile case without being called an OGC conformance class. **[P]**

### 13.6 Synchronization and Conflict

The isolated two-node profile injects at-least-once delivery, duplicate envelopes, reordering, gaps, tombstones, concurrent revisions, policy/trust changes and recovery. Assertions cover envelope identity, dedupe, effectively-once local application, watermarks, explicit gaps, conflict/quarantine records, constrained automatic-resolution allowlist, provenance, audit and resnapshot/recovery lineage. **[A,P]**

Database replication or broker delivery success is not application synchronization proof. Performance at scale, hostile peer behavior and named external-node interoperability remain later topics. **[X]**

## 14. CI and Local Developer Workflow Findings

### 14.1 CI Tiers

| Tier | Trigger | Required content | Gate behavior |
|---|---|---|---|
| metadata/static | every change | case/source/fixture/schema lint, duplicate IDs, graph closure, OAD generation/parity inputs | blocking |
| fast PR | every pull request | unit/in-process layers plus black-box discovery, selected resource, negative and changed-capability cases | blocking |
| full merge | protected branch | all applicable current claimed-class cases on fresh Compose target | blocking |
| nightly/advisory | schedule/manual | Schemathesis, extended codecs, fault profiles, selected external clients and official lanes | failure investigated; promotion policy explicit |
| release candidate | release | clean full evidence graph, official inherited suites, all enabled profiles, reproducibility and manifest signing | blocking |
| observational | schedule/manual | public demo and authorized external endpoints, read-only | never gates on third-party availability |

Change impact selection may reduce PR latency only when the trace graph proves which cases are unaffected. A release candidate always resolves the full claim graph. **[P]**

### 14.2 Local CLI Contract

The first-party CLI should support:

- target URL or managed local/Compose target;
- case ID, requirement URI, ATS ID, class, category, profile, fixture and changed-source filters;
- `list`, `explain`, `run`, `replay-evidence`, `validate-evidence` and `evaluate-claims` functions;
- a dry-run showing resolved cases, mutations, credentials, profiles and estimated dependencies;
- safe verbosity with explicit restricted-capture opt-in; and
- deterministic reproduction from a prior `run.json` where dependencies remain available.

Failure output leads with source ID, assertion, actual/expected safe summary, evidence path and reproduction command. Developers need not inspect a CI-specific JUnit renderer to understand a failure. **[P]**

### 14.3 Flakiness Policy

- Timeouts are bounded by operation category and driven by state/event barriers, not arbitrary sleeps.
- Tests own their data namespace and do not depend on execution order unless a declared scenario sequence requires it.
- Retries are off for claim-producing release runs; diagnostic retry attempts remain visible.
- A case that passes only after retry is `flaky`, not clean pass.
- Quarantine requires owner, reason, issue and expiry; an applicable mandatory quarantined case blocks the affected claim.
- Statistical flake detection may run on nightly repetitions, but it cannot vote away a deterministic failure.

### 14.4 Observability

Every run supplies a non-secret run ID and each request a case/step correlation context. Target logs, traces, metrics, audit and system events are collected as supporting diagnostics under IDR-SRV-048 classifications. They do not replace wire and state assertions. The harness itself exposes timings and failure categories without placing case IDs, resource IDs or target URLs into unbounded metric labels. **[A,P]**

## 15. Official OGC Tooling and External Harness Integration Findings

### 15.1 Current Official State

OGC's published Validator page identifies TEAM Engine as the compliance platform, lists OGC API - Features 1.0 ETS revision 1.6 as final and lists older SensorML/SWE suites, but does not list Connected Systems, SensorML 3.0 or SWE Common 3.0 suites. The official CSAPI standards nevertheless require relevant Annex A tests. The responsible conclusion is “no public official executable suite identified at this evidence freeze,” not a claim that OGC will never publish one. **[N,D]**

### 15.2 Integration Contract

Each official/external lane adapter must record:

- tool and suite identifier/version/digest/source;
- exact run arguments, target, credentials mode and selected conformance classes;
- native stdout/stderr/result files without semantic rewriting;
- wrapper start/end/error state and mapping to Glaux source IDs;
- known exclusions, skips, version/OAS/profile mismatches and accepted adapters; and
- independent outcome plus its role in the claim graph.

The OGC API Features ETS should run against the isolated release candidate for inherited behavior. Its OAS 3.0 expectations must be reconciled with Glaux's accepted OAS 3.1.2 publication policy through a separately generated, tested compatibility view if required; passing that view cannot falsely assert that the canonical 3.1 document satisfies an OAS 3.0 conformance class. **[A,P]**

### 15.3 Future CSAPI ETS Adoption

When an official CSAPI ETS appears:

1. pin and inventory the suite, license, source standard and declared coverage;
2. map every official test to the Glaux requirement/case registry;
3. run official and internal suites side by side against the same fixture/profile where compatible;
4. classify differences as source-version, fixture, interpretation, suite defect, wrapper or target behavior;
5. retain internal negative, regression, policy and project-profile cases not covered officially; and
6. change certification language only through OGC's then-current process.

An official suite supplements or supersedes only mapped normative adapters after review; it never automatically deletes Glaux regressions. **[P]**

### 15.4 Schemathesis, Newman and External Clients

Schemathesis should run against the published reference-closed OAD with sanitization enabled, fixed examples/seed where supported, safe operation filters and a disposable target. Findings enter triage as generated evidence and become blocking only after a stable case is promoted into the first-party catalog. Its JUnit/HAR output remains native external evidence. **[D,P]**

Newman may publish a small human-editable smoke collection for demonstrations or partner onboarding, generated or checked against stable first-party case IDs. It must not become an independent requirement source. External clients and CSAPI Explorer run under IDR-SRV-056 with exact pins; public-server failures are observational unless Glaux owns the target and conditions. **[I,P]**

## 16. Downstream Topic Handoff Matrix

| Owner | Inputs fixed by this report | Required downstream decision/output | Boundary retained |
|---|---|---|---|
| IDR-SRV-051 requirement-to-test traceability | source-to-case-to-evidence chain, mandatory metadata, result states and claim closure | final IDs, schemas, coverage queries, change impact, review/supersession workflow | must not redefine normative obligations |
| IDR-SRV-052 Rust TDD/multi-layer strategy | Rust black-box CLI direction, independent wire models, target and runner layers | crate/workspace layout, test APIs, nextest/Cargo usage, concurrency, quality gates | unit/in-process tests do not replace black-box claims |
| IDR-SRV-053 fixtures/goldens/scenarios | scenario families, manifest semantics, exact/semantic comparison and sensitivity | file organization, generators, golden review, provenance, fixture lifecycle | no operational or uncontrolled data |
| IDR-SRV-054 performance/load/streaming | functional cases, target profiles, event barriers and evidence identity | workloads, volumes, durations, metrics, thresholds and regression policy | performance result does not alter conformance pass |
| IDR-SRV-055 security/authorization/command tests | bounded core security lane, synthetic identities, simulated command invariant | threat-driven matrices, scanners/fuzzers, hostile inputs, safety and authorization depth | no physical effects or production identity dependency |
| IDR-SRV-056 external-client interoperability | named external lane contract, exact pins, semantic completeness and wire capture | client/server matrix, versions, scenarios, tolerances, issue evidence | named-pair result is not universal conformance |
| IDR-SRV-057 final synthesis | hybrid architecture, official gap, claim semantics, risks and proof backlog | reconcile accepted Category I decisions and residual readiness | no unsupported certification/readiness claim |
| implementation roadmap | component model, CLI contract, CI tiers and twelve proofs below | sequence milestones and acceptance gates | no code authorized by this report |
| OGC/SWG engagement | interpretation/adapter ledger and future ETS mapping procedure | submit reproducible issues and evaluate future suite | project adapters never presented as OGC rulings |

### 16.1 Required Implementation Proofs

Before the harness can support a release claim, implementation must demonstrate at least these bounded proofs:

1. **Catalog closure:** load all 25 classes, 233 requirements, five recommendations and 240 ATS mappings with no orphan or duplicate identifier.
2. **Applicability:** resolve prerequisites and conditional rows for two contrasting release profiles; invalid skips fail validation.
3. **Independent black box:** run one Part 1 and one Part 2 case against a process/container without linking server domain or wire crates.
4. **Wire preservation:** prove raw one-element arrays, duplicate headers, binary bodies and mismatched `Content-Type` survive capture before parsing.
5. **Deterministic lifecycle:** create, observe, update/delete and clean an isolated fixture with an exact final-state proof.
6. **Semantic query:** detect an accepted-but-ignored filter using known-hit/known-miss fixtures.
7. **Evidence reproducibility:** validate hashes and replay the same case selection from `run.json` in a clean target.
8. **Sensitivity:** inject token/secret/sensitive fields and prove ordinary artifacts, logs, JUnit and Markdown contain none.
9. **External federation:** ingest untouched official Features ETS native output and Schemathesis native output without collapsing their outcomes.
10. **Non-flaky stream:** exercise a finite event/replay case through barriers and virtual/fixed time with no arbitrary sleep assertion.
11. **Command safety:** prove a command case cannot select a physical adapter and records no external effect after cleanup/restore.
12. **Claim truth:** introduce one failure, error, inconclusive, invalid skip and flaky retry and prove every affected class is withheld from `/conformance` for the evaluated build.

Passing these proofs validates harness mechanisms, not all standards requirements. **[P]**

## 17. Recommendations

1. **Adopt a hybrid evidence system led by a standalone Rust black-box CLI in the Glaux Server workspace.**
   - Keep unit/in-process, official, generated, performance, security and interoperability lanes distinct.
2. **Make the 25-class, 240-ATS accepted registry the selection authority.**
   - Do not select tests from server declarations or route presence alone.
3. **Use independent wire-level test models.**
   - Share pinned schemas and identifiers, not server behavior or serialization implementations.
4. **Define canonical `ConformanceCaseV1`, `CaseResultV1` and `ConformanceRunV1` semantics.**
   - Let IDR-SRV-051 choose their final representation and governance.
5. **Preserve six terminal results and orthogonal flake state.**
   - Only a stable `pass` satisfies an applicable mandatory case.
6. **Adapt every normative ATS test visibly and supplement known gaps.**
   - Record original text, defect, interpretation, adapter and stronger regression assertions.
7. **Use deterministic isolated Compose/CI deployments as canonical claim targets.**
   - Public demos and external servers remain read-only observational targets by default.
8. **Make canonical JSON evidence authoritative.**
   - Derive JUnit and Markdown; preserve raw wire data before parsing subject to security controls.
9. **Use semantic assertions by default and exact goldens selectively.**
   - Every normalization is explicit, evidence-preserving and reviewed.
10. **Gate claims on the closed evidence graph.**
    - Any failure, harness error, inconclusive outcome, invalid skip, applicable quarantine or unstable retry withholds the affected class.
11. **Run OGC API Features ETS as a separate inherited lane and prepare for a future CSAPI ETS.**
    - Never call internal results official certification.
12. **Use Schemathesis as advisory discovery, not normative authority.**
    - Promote stable findings into first-party traced cases.
13. **Prohibit physical effects and uncontrolled mutation.**
    - Command tests use the simulator; external/public targets require explicit scoped authority.
14. **Make safety and sensitivity part of harness correctness.**
    - Schema-driven redaction, synthetic identities, restricted evidence and cleanup proofs are required.
15. **Execute the twelve proof gates before implementation claims.**
    - The ordered proofs expose catalog, isolation, evidence, flake, safety and claim-evaluation defects early.

## 18. Risks, Constraints, and Open Questions

### 18.1 Risks and Controls

| Risk | Consequence | Required control |
|---|---|---|
| literal ATS copies known defect | correct behavior fails or incorrect behavior passes | versioned interpretation/adapter ledger plus requirement-level supplement |
| no current official CSAPI ETS | internal evidence mistaken for certification | explicit lane/claim vocabulary and future-suite mapping |
| server and harness share models/logic | tautological passes | independent wire models and black-box release lane |
| permissive comparison | false conformance | named semantic assertions and allowlisted normalization |
| broad snapshots | brittle noise or blind approval | exact goldens only for stable artifacts; reviewed semantic diffs |
| mutable demo/external data | flaky, irreproducible evidence | deterministic local fixtures; observational external lane |
| ignored filters return 200 | false functional pass | known-hit/miss and metamorphic assertions |
| retries conceal races | unstable release claim | preserve attempts; flaky blocks release claim |
| fixture setup/cleanup contaminates target | misleading later results or unsafe effects | exclusive namespace/lease and final-state proof |
| evidence leaks credentials/policy/data | security incident | synthetic data, schema classification, sanitization and restricted raw store |
| target profile enables physical command | real-world effect | hard simulated-profile configuration invariant and startup refusal |
| external suite/tool changes | incomparable evidence | exact versions/digests/native output and source-change invalidation |
| OAS 3.0 official-tool versus 3.1 canonical seam | false OAS claim or suite incompatibility | separately generated compatibility view with parity/loss tests |
| matrix grows beyond practical PR duration | developers bypass gate | trace-based PR subset, full merge/release graph and partitioning |
| draft Part 3 mixed into approved claims | misleading conformance | separate exact-draft lane only after project authorization |

### 18.2 Bounded Open Questions

| Question | Current decision | Owner |
|---|---|---|
| exact case/requirement ID and schema syntax? | required semantics fixed; syntax deferred | IDR-SRV-051 |
| exact Rust crates and runner abstraction? | logical components fixed; layout deferred | IDR-SRV-052 |
| YAML, JSON or generated fixture definitions? | manifest semantics fixed; storage deferred | IDR-SRV-053 |
| PR duration, shard count and performance budgets? | tier model fixed; numbers require implementation evidence | IDR-SRV-052/054 |
| detailed identity/policy matrix and security scanners? | bounded fake-authority core only here | IDR-SRV-055 |
| which external clients/servers and versions gate interoperability? | named-pair model fixed | IDR-SRV-056 |
| evidence retention/signing service? | immutable release evidence required; provider/duration deferred | operations/release governance |
| when an official CSAPI ETS will appear? | monitor and integrate without assuming schedule | OGC liaison/IDR-SRV-057 |

None requires a new decision before IDR-SRV-051 research begins. **[P]**

## 19. Validation Against This Plan's Success Criteria

| Success criterion | Result | Evidence |
|---|---|---|
| conformance scope with source anchors and prior-topic traceability | Met | Sections 2–5 |
| full required test taxonomy distinctions | Met | Section 5 |
| architecture options and target strategies evaluated | Met | Sections 6–7 |
| requirement/test/evidence model documented | Met | Section 8 |
| fixture, profile, CI, local workflow and evidence requirements | Met | Sections 9–10 and 14 |
| API, schema, negative, security, dynamic, stream, command, DDIL and sync coverage | Met | Sections 11–13 |
| official OGC integration opportunities and limitations | Met | Section 15 |
| implementation/community lessons non-normative | Met | Section 3.4 |
| decision-usable, server-bounded recommendations | Met | Sections 2, 17 and 18 |
| downstream handoffs explicit | Met | Section 16 |
| references explicit and reproducible | Met | Section 20 |

All planned phases and required report content are complete. Acceptance remains a project-lead action. **[P]**

## 20. References

### 20.1 Controlling Standards and Official Artifacts

- OGC API - Connected Systems Part 1: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems Part 2: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems publication repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems v1.0.0 API artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OGC schema registry: https://schemas.opengis.net/
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- JSON Schema: https://json-schema.org/
- RFC 9110: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457: https://www.rfc-editor.org/rfc/rfc9457

### 20.2 OGC Compliance and Executable Tooling

- OGC Validator and available test suites: https://cite.ogc.org/teamengine/
- OGC compliance program: https://www.ogc.org/compliance/
- OGC implementation testing: https://www.ogc.org/compliance/test-your-implementations/
- TEAM Engine repository: https://github.com/opengeospatial/teamengine
- OGC API - Features repository and validator information: https://github.com/opengeospatial/ogcapi-features
- OGC API - Features 1.0 ETS documentation: https://cite.ogc.org/teamengine/about/ogcapi-features-1.0/1.0/site/

### 20.3 Harness and Testing Tools

- Cargo test: https://doc.rust-lang.org/cargo/commands/cargo-test.html
- cargo-nextest: https://nexte.st/
- reqwest: https://docs.rs/reqwest/
- wiremock: https://docs.rs/wiremock/
- Testcontainers for Rust: https://rust.testcontainers.org/
- insta: https://docs.rs/insta/
- assert-json-diff: https://docs.rs/assert-json-diff/
- jsonschema: https://docs.rs/jsonschema/
- schemars: https://docs.rs/schemars/
- Schemathesis: https://schemathesis.readthedocs.io/
- pytest: https://docs.pytest.org/
- Newman: https://learning.postman.com/docs/collections/using-newman-cli/command-line-integration-with-newman/

### 20.4 Accepted Project Evidence

- Overall IDR Research Plan: [overall-idr-research-plan.md](../IDR%20Plans/overall-idr-research-plan.md)
- IDR-SRV-050 Research Plan: [idr-srv-050-conformance-harness-strategy.md](../IDR%20Plans/idr-srv-050-conformance-harness-strategy.md)
- Glaux Server Goal and Definition: [glaux-server-goal-and-definition.md](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- Accepted IDR-SRV-001 through IDR-SRV-049 reports: [IDR Reports](./)
- IDR-SRV-006 Part 1 baseline: [idr-srv-006-csapi-part-1-requirement-baseline-report.md](idr-srv-006-csapi-part-1-requirement-baseline-report.md)
- IDR-SRV-007 Part 2 baseline: [idr-srv-007-csapi-part-2-requirement-baseline-report.md](idr-srv-007-csapi-part-2-requirement-baseline-report.md)
- IDR-SRV-008 conformance mapping: [idr-srv-008-conformance-class-and-requirement-mapping-report.md](idr-srv-008-conformance-class-and-requirement-mapping-report.md)
- IDR-SRV-014 OpenAPI strategy: [idr-srv-014-openapi-description-and-api-documentation-strategy-report.md](idr-srv-014-openapi-description-and-api-documentation-strategy-report.md)
- IDR-SRV-021 SensorML strategy: [idr-srv-021-sensorml-representation-strategy-report.md](idr-srv-021-sensorml-representation-strategy-report.md)
- IDR-SRV-022 SWE Common strategy: [idr-srv-022-swe-common-data-component-strategy-report.md](idr-srv-022-swe-common-data-component-strategy-report.md)
- IDR-SRV-049 lifecycle strategy: [idr-srv-049-migration-upgrade-backup-and-restore-strategy-report.md](idr-srv-049-migration-upgrade-backup-and-restore-strategy-report.md)
- Research Report Template: [research-report-template.md](../../../../../Governance/research-report-template.md)

### 20.5 Non-Normative Implementation and Interoperability Evidence

- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client/testing corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OGC CSAPI developer site: https://csapi.developer.ogc.org/
- Testbed-18 Moving Features Engineering Report: https://docs.ogc.org/per/22-016r3.html

### 20.6 Evidence Limits

Official standards, validator inventory and tool documentation were checked September 16, 2026. Registry/package versions are mutable and must be revalidated during implementation. No Glaux Server or conformance harness implementation existed to execute, so architecture and twelve proof gates are recommendations, not observed performance. No public official executable CSAPI, SensorML 3.0 or SWE Common 3.0 suite was identified in the official Validator inventory or reviewed OGC repositories; this is a dated evidence gap, not a permanent nonexistence claim.

---

## Report Completion Checklist

- [x] All 20 required sections are present
- [x] Conformance scope and authority lanes are explicit
- [x] Required 10-column conformance harness matrix is complete
- [x] Architecture options and target strategies are evaluated
- [x] Requirement/test/evidence and claim-gate models are complete
- [x] Fixture, profile, evidence, CI and local workflow requirements are explicit
- [x] API, schema, negative, security, dynamic, stream, command, DDIL and sync coverage is complete
- [x] Official OGC tooling state and future integration are bounded
- [x] Twelve implementation proofs and downstream handoffs are explicit
- [x] All 11 success criteria validate as Met
- [ ] Accepted by Glaux Project Lead

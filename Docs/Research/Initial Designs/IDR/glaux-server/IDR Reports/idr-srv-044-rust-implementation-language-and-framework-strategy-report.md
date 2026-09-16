# Section 044: Rust Implementation Language and Framework Strategy - Research Report

**Topic ID:** IDR-SRV-044<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-044 Rust Implementation Language and Framework Strategy](../IDR%20Plans/idr-srv-044-rust-implementation-language-and-framework-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All questions concerning Rust requirement fit, HTTP/framework/runtime choices, domain types, serialization, OpenAPI, validation, errors, PostgreSQL/PostGIS/time-series access, ingestion, streaming, workers, commands, security, configuration, observability, testing, CI, unsafe code, supply chain, packaging, maintenance, first-stack selection, proof gates, and downstream handoffs<br>
**Methodology Used:** Current official-source and crates.io freeze; accepted-requirement extraction; capability and risk matrices; standards/test/security/operations scoring; first-slice versus full-scope separation; proof-of-concept gating; implementation-study reconciliation; downstream traceability<br>
**Research Time:** Approximately 38 hours of AI-assisted execution on September 15, 2026<br>
**Approved Language Baseline:** Rust remains the approved implementation language; this report does not reopen language selection<br>
**Compiler/Ecosystem Freeze:** Rust 1.98.1 stable, Rust 2024 edition; official documentation and non-yanked stable crate releases checked September 15, 2026<br>
**Standards Baseline:** OGC 23-001 and 23-002 Version 1.0; SensorML 3.0; SWE Common 3.0; RFC 9110 and 9457; OpenAPI and JSON Schema; accepted Glaux IDR-SRV-001 through IDR-SRV-043; controlled AEP package subject to recorded handling constraints<br>
**Implementation Evidence:** Accepted OSH, Connected Systems Go, pygeoapi, SECD, OS4CSAPI client/interoperability/community, and draft Part 3 studies used only as non-normative evidence<br>
**Document Purpose:** Select a defensible Rust implementation-platform candidate and proof gates without prematurely fixing final service modularization, deployment topology, configuration system, observability backend, migration process, or implementation code<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 15, 2026<br>
**Date:** September 15, 2026<br>
**Last Updated:** September 15, 2026

---

## Evidence and Decision Legend

| Mark | Meaning |
|---|---|
| N | Normative published specification requirement within its scope |
| C | Controlled project/source finding, limited to accessible evidence and handling rules |
| A | Accepted Glaux design baseline |
| I | Current crate/framework or implementation evidence; not a requirement |
| E | Engineering, security, maintenance, or interoperability inference |
| P | Proposed Glaux platform decision requiring this report's acceptance |
| X | Open parameter, proof gate, or downstream decision |

Crate versions are an evidence snapshot, not a permanent dependency lock. “Recommended” means preferred for the first implementation subject to the named proof, advisory, license, MSRV and integration gates. Standards correctness remains owned by Glaux tests and reviewed contracts; no framework, derive macro or ORM is treated as a conformance authority. **[A/E/P]**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Implementation Requirement Extraction Methodology
5. Rust Requirement-Fit Constraints, Ecosystem Risks, and Mitigation Assessment
6. Core Web Framework and HTTP Stack Evaluation
7. Async Runtime and Concurrency Model Evaluation
8. Domain Modeling and Serialization Strategy Findings
9. OpenAPI, Schema, Validation, and Documentation Tooling Findings
10. Error Handling and Problem-Detail Findings
11. Database, Geospatial, Time-Series, Migration, and Transaction Findings
12. Ingestion, Streaming, Event Outbox, and Background Worker Findings
13. Command/Control Implementation Support Findings
14. Security/Authn/Authz Implementation Support Findings
15. Configuration, Profiles, Feature Flags, and Secrets Findings
16. Observability, Logging, Metrics, Tracing, and Health Findings
17. Testing, CI, Quality-Gate, Static-Analysis, and Supply-Chain Findings
18. Unsafe-Code, Dependency, License, and Maintenance-Risk Findings
19. Recommended First-Implementation Stack
20. Full-Scope Readiness Roadmap and Proof-of-Concept Needs
21. Downstream Topic Handoff Matrix
22. Risks, Constraints, and Open Questions
23. Validation Against This Plan's Success Criteria
24. References

---

## 1. Executive Summary

Rust remains a strong fit for Glaux Server when used as a **bounded, explicit platform**, not as a promise that the type system or generated code supplies standards conformance. Its ownership model, enums/newtypes, `Result`, efficient asynchronous I/O, single deployable binary, mature HTTP foundations and good test tooling support the accepted correctness, security, DDIL and edge-deployment goals. The principal costs are ecosystem assembly, async cancellation and blocking hazards, compile time, sparse OGC/XML/SWE specialization, macro/schema drift, native geospatial dependencies, transitive unsafe code, and a large dependency review surface. Each cost has an explicit mitigation or proof gate below. **[A/E/P]**

The preferred first platform is Rust `1.98.1`, edition 2024, with a declared MSRV of `1.94` while SQLx 0.9 is used. Pin the stable toolchain and all CI tools; commit `Cargo.lock`; build with `--locked`; and upgrade through reviewed dependency batches. Use Axum `0.8.9` over Tokio `1.53.1`, Tower `0.5.3`, Tower HTTP `0.7.1` and Hyper `1.11.1`. This stack exposes HTTP semantics without imposing a large opinionated application framework, shares a composable service/middleware model, and aligns with SQLx and the wider Tokio ecosystem. Actix Web remains the credible fallback if the vertical-slice proof finds a blocking Axum issue; Poem and Warp are not first choices. **[I/E/P]**

Use Serde/serde_json for wire DTOs, but separate source artifacts, wire representations, canonical domain types, persistence records and public projections. Strong domain newtypes and enums enforce local invariants; external and persisted inputs still pass explicit validation. Unknown JSON extension members are bounded and preserved only where the profile allows them. SensorML/XML and SWE codecs require specialized adapters and golden tests: `quick-xml` can parse/emit XML but is not an XSD validator, and no reviewed Rust crate is accepted here as a complete substitute for the offline schema/semantic validation architecture. **[A/E/P]**

Adopt a **hybrid OpenAPI workflow**. Curated, versioned artifacts derived from the approved OGC baseline remain the contract authority. Utoipa `5.5.0` is the preferred generation/annotation candidate for route and DTO fragments, subject to a prototype proving exact paths, parameters, media types, links, examples, reusable problems and OpenAPI-version behavior. CI renders a deterministic candidate and performs structural/semantic drift checks plus executable contract tests. Generated output is never auto-promoted merely because it compiles. **[N/A/I/P]**

Use `jsonschema 0.56.0` through an offline, precompiled registry with network/file retrieval disabled and exact draft/profile pins. It covers structural JSON validation, not CSAPI interactions, SensorML/SWE semantics, relationship rules, units, policy, source authority or command safety. Use typed domain validators for those layers and retain a stable finding catalog. Rust derive-validation crates may reduce boilerplate for simple DTO constraints, but they are optional helpers rather than the validation architecture. **[A/I/P]**

Use SQLx `0.9.0` with PostgreSQL/PostGIS-specific, parameterized SQL, compile-checked queries where practical, checked-in offline metadata and embedded/versioned migrations. This best preserves the accepted relational-hybrid, native time-series, spatial, transaction, inbox/outbox and concurrency decisions. Diesel and SeaORM are viable libraries but add an abstraction model that does not remove the need for deliberate PostGIS, partition, CTE, lock, range and projection SQL. SQLx 0.9's recency and MSRV make a database vertical-slice proof mandatory before dependency lock. **[A/I/E/P]**

Async work uses structured ownership: bounded channels, explicit task registries, cancellation tokens, deadlines, semaphores, leases/fences and graceful shutdown. No detached task, in-memory queue or broker acknowledgement is durable truth. CPU-heavy schema, XML, canonicalization or geometry work uses a bounded blocking/CPU pool; database-backed workers claim small batches and persist attempts. The first publication slice remains HTTP query/change feed plus SSE and the transactionally coupled outbox. MQTT uses a later adapter proof; NATS and Kafka remain replaceable prepared options, not first-runtime dependencies. **[A/P]**

Security and accountability remain domain/application concerns behind narrow ports. The server validates trusted proxy/mTLS evidence and OAuth/OIDC tokens through swappable adapters, builds one immutable security context, invokes policy/safety/audit hooks, and never accepts caller-provided authority. Use rustls or approved deployment TLS termination; no custom cryptography. First-party crates forbid unsafe code, while dependency unsafe is inventoried and risk-reviewed rather than falsely claimed absent. **[A/E/P]**

The minimum merge gate is formatting, compilation for locked toolchains and features, Clippy with denied warnings, unit/integration/database tests, OpenAPI and SQLx metadata drift checks, dependency/license/source/advisory policy, and generated-artifact cleanliness. Scheduled or release gates add coverage, fuzzing, performance, semver, container/SBOM, interoperability and security suites. This report recommends a platform candidate; IDR-SRV-045 still owns final service/module boundaries and no implementation is authorized by acceptance alone. **[P]**

---

## 2. Scope and Plan Alignment

### 2.1 Included scope

This report evaluates Rust compiler/MSRV policy; HTTP framework/runtime; domain and wire modeling; JSON/XML/GeoJSON/SWE handling; OpenAPI and validation; errors; PostgreSQL/PostGIS/time-series access; migrations/transactions; ingestion/publication/workers; command safeguards; security adapters; configuration/secrets; telemetry/health; testing/CI; dependency, license and unsafe posture; packaging implications; recommended first stack; and proof gates. **[P]**

### 2.2 Excluded scope

It does not reopen the approved language, create a server repository, implement endpoints, finalize crate/process topology, select production IdP/policy/secret/telemetry vendors, choose HA/deployment topology, define migration/backup operations, approve live command effects, or claim conformance. IDR-SRV-045 through 057 retain those responsibilities. **[A/P]**

### 2.3 Plan coverage

| Plan concern | Report location |
|---|---|
| requirement fit and Rust constraints | Sections 3–5 |
| HTTP framework/runtime/concurrency | Sections 6–7 |
| domain, serialization, OpenAPI and validation | Sections 8–10 |
| database, geospatial, time-series, transactions | Section 11 |
| ingestion, streaming, workers and commands | Sections 12–13 |
| security, configuration and observability | Sections 14–16 |
| testing, CI, supply chain, unsafe and licensing | Sections 17–18 |
| selected stack, roadmap and handoffs | Sections 19–23 |

### 2.4 Prior-IDR requirements treated as binding

The platform must support stable identifiers/revisions and exact artifacts; bitemporal/domain time; provenance; current projections; multiple representations and negotiation; offline validation packages; PostgreSQL/PostGIS/native time-series; atomic inbox/outbox/idempotency; lifecycle/tombstones; bounded ingestion and SSE/MQTT adapters; durable command/feasibility state; deny-by-default security/policy; E0–E5 audit capture; DDIL qualification; and synchronization/conflict evidence. Framework convenience never weakens those accepted contracts. **[A]**

---

## 3. Evidence Base and Authority Classification

### 3.1 Current source freeze

| Evidence | Version/status reviewed | Use and limitation |
|---|---|---|
| Rust official releases | `1.98.1`, 2026-09-03 | compiler baseline; point release fixes a 1.98.0 miscompilation |
| crates.io and docs.rs | non-yanked stable releases, 2026-09-15 | current versions, declared licenses/MSRV, API docs; not a security approval |
| official project repositories/docs | checked 2026-09-15 | feature, architecture and maintenance evidence; popularity not authority |
| RustSec / cargo-audit | current tooling/database | advisory signal; absence of advisory is not proof of safety |
| cargo-deny | `0.20.2` | license, source, bans and advisory policy candidate |
| OGC/IETF/OpenAPI/JSON Schema | published specifications | contract requirements; crates do not override them |
| accepted IDR-SRV-001–043 | accepted project decisions | functional/nonfunctional platform requirements |
| implementation studies 014A–H | pinned accepted reports | non-normative lessons; no framework selection mandate |

### 3.2 Version snapshot

| Area | Current stable evidence on 2026-09-15 |
|---|---|
| web/runtime | Tokio 1.53.1; Axum 0.8.9; Tower 0.5.3; Tower HTTP 0.7.1; Hyper 1.11.1; Actix Web 4.15.0; Poem 3.1.12; Warp 0.4.3 |
| representation/schema | Serde 1.0.229; serde_json 1.0.151; schemars 1.2.2; jsonschema 0.56.0; Utoipa 5.5.0; Aide 0.15.1; quick-xml 0.42.0 |
| persistence/geo | SQLx 0.9.0; Diesel 2.3.13; SeaORM 2.0.3; geo 0.33.1; geo-types 0.7.20; geojson 1.0.0; geozero 0.15.1; PROJ bindings 0.31.0 |
| telemetry/errors/config | tracing 0.1.44; tracing-subscriber 0.3.23; OpenTelemetry/OTLP 0.32.0; thiserror 2.0.20; anyhow 1.0.104; config 0.15.25; clap 4.6.7; secrecy 0.10.3 |
| security/time/identity | rustls 0.23.45; jsonwebtoken 11.0.0; openidconnect 4.0.1; oauth2 5.0.0; uuid 1.26.1; time 0.3.55 |
| tests/CI | proptest 1.11.0; rstest 0.27.0; insta 1.48.0; nextest 0.9.144; testcontainers 0.28.0; criterion 0.8.2; cargo-fuzz 0.13.2; cargo-llvm-cov 0.9.1 |

Versions were retrieved from official crates.io metadata and cross-checked against official documentation where material. Feature-specific transitive trees, advisories, yanks and licenses must be regenerated from the actual locked workspace; this snapshot cannot substitute for that future review. **[I/E]**

### 3.3 Implementation-study lessons

The accepted studies show that correct resource/link composition, route discovery, representation negotiation, schema fidelity, query semantics, tasking lifecycle, OpenAPI accuracy and deterministic fixtures matter more than mirroring any one server's language/framework. Silent empty results, drift between documented and actual routes, implementation-specific extensions and incompatible draft Part 3 interpretations become contract and regression-test requirements. Glaux should adopt explicit adapters and executable golden contracts, not copy OSH, CS-Go, pygeoapi or SECD internals. **[A/I/E]**

---

## 4. Implementation Requirement Extraction Methodology

The analysis applied five steps:

1. Convert each accepted IDR decision into a platform capability, invariant and test obligation.
2. Separate first vertical-slice needs from full-scope adapters so optional brokers, codecs and vendors do not burden the trusted core.
3. Evaluate candidates for standards control, correctness, async/persistence fit, testability, security, operations, maturity, license, MSRV and dependency cost.
4. Prefer boring composition and explicit domain code over macros/abstractions that conceal protocol or transaction semantics.
5. Assign every unresolved risk to a proof-of-concept or downstream owner and prevent the recommendation from silently becoming implementation authorization.

### 4.1 Decision rules

- A selected framework must expose, not obscure, headers, media types, validators, links, streaming, backpressure, cancellation and problem responses.
- A generated artifact is acceptable only when deterministic output is checked against the curated contract and executable behavior.
- A database layer must preserve explicit SQL, transactions, locks, PostGIS operators, partitioning and migration control.
- A crate touching authentication, parsing, cryptography, persistence, unsafe/FFI or command execution receives heightened review.
- Optional capabilities enter behind a port/adapter and feature/config gate only when their absence does not change core domain semantics.
- “Pure Rust” is not a goal when a reviewed native component is necessary, but native/unsafe/operational cost must be explicit.

### 4.2 Recommendation statuses

`Select` means first-stack candidate; `Select with proof gate` means preferred but blocked on a named vertical slice; `Prepared adapter` means define a port but do not make it a first dependency; `Conditional` means adopt only when a deployment/profile needs it; `Fallback` means credible alternative; and `Do not select initially` means no first-slice role. **[P]**

---

## 5. Rust Requirement-Fit Constraints, Ecosystem Risks, and Mitigation Assessment

| Requirement/risk | Rust fit | Constraint | Mitigation |
|---|---|---|---|
| stable domain invariants | strong enums/newtypes/traits | types cannot validate remote truth or mutable policy | explicit constructors plus layered validation/evidence |
| memory safety | strong safe-language default | transitive unsafe and FFI remain | forbid first-party unsafe; inventory/review dependencies |
| HTTP/stream scale | mature Tokio/Hyper stacks | cancellation, fairness and blocking hazards | structured tasks, bounds, timeouts, blocking pools |
| standards artifacts | strong JSON; general XML support | limited OGC/SensorML/SWE/XSD specialization | preserve source, adapter boundary, offline validator PoC |
| PostgreSQL/PostGIS | strong SQL clients | ORM abstractions do not model full accepted SQL needs | SQLx/Postgres-specific SQL; database golden tests |
| geospatial algorithms | useful geo ecosystem | CRS/vertical/native transform complexity | PostGIS authority; defer PROJ FFI; exact-source retention |
| time series | ordinary PostgreSQL support | no crate removes schema/partition/retention design | SQLx plus native schema and measured queries |
| command safety | types/state machines help | process crashes and physical effects cross type boundary | durable state, fences, idempotency, audit, adapter proofs |
| DDIL/edge | efficient static-ish binary possible | dependencies, TLS/native libs and storage dominate footprint | profile builds, measured images, no premature musl claim |
| OpenAPI | multiple generators | macro/schema drift and version mismatch | curated authority plus deterministic generated diff |
| compile time | acceptable but nontrivial | macros/features/workspace can become slow | lean crates/features, cache, CI measurement, avoid type gymnastics |
| contributor ergonomics | excellent tooling | lifetimes/traits/async errors can overcomplicate | concrete types first, narrow traits at real seams |
| long maintenance | Cargo ecosystem strong | high transitive churn/MSRV jumps | lockfile, policy, update cadence, dependency budget |
| panic behavior | explicit `Result` idiom | panics still possible in libraries/tasks | no unwrap/expect on request data; panic capture/abort policy tests |

### 5.1 Toolchain policy

Use edition 2024 and pin `rust-toolchain.toml` to Rust `1.98.1` for reproducible CI and developer behavior. Declare workspace `rust-version = "1.94"` because SQLx 0.9 requires it; test both MSRV and pinned stable. Upgrade MSRV only through an architecture decision recording dependency cause and deployment impact. Pin rustfmt/clippy with the toolchain and CI utilities by version/digest. **[I/E/P]**

### 5.2 Abstraction posture

Begin with concrete Axum, SQLx and Tokio adapters around domain/application interfaces. Use traits only at volatility, effect or test seams: clock/ID generation, repositories/unit of work, artifact storage, source adapter, publisher, command gateway, identity/policy, audit sink and telemetry/export. Avoid a trait per struct, generic web handlers, runtime-generic database access, a home-grown dependency-injection container and premature microservices. IDR-SRV-045 finalizes boundaries. **[P]**

---

## 6. Core Web Framework and HTTP Stack Evaluation

| Candidate | Strengths for Glaux | Risks/gaps | Status |
|---|---|---|---|
| Axum 0.8.9 + Tower/Hyper | explicit extractors/responses; shared Tower middleware; Tokio/Hyper alignment; composable tests | OpenAPI separate; extractor rejection normalization; careful body/stream limits and route nesting needed | **Select with vertical-slice proof** |
| Actix Web 4.15.0 | mature, high performance, rich framework ecosystem | separate service/middleware idioms; more framework-specific boundary; dual conventions if Tower services needed | **Fallback** |
| Poem 3.1.12 | integrated ecosystem including OpenAPI options | smaller evidence base; integrated generation still requires contract proof; older reviewed release | Do not select initially |
| Warp 0.4.3 | composable filters, Hyper lineage | filter/type complexity and weaker preferred OpenAPI path; less direct fit for explicit handler/application layers | Do not select initially |
| raw Hyper 1.11.1 | maximum protocol control | unnecessary routing/extractor/error boilerplate and security risk | use under Axum, not directly |

Axum is selected because it is thin enough to preserve standards-visible control while reusing Tower for request IDs, trace context, timeouts, concurrency/load shedding, body limits, compression and security middleware. Middleware order is security-critical and requires an executable matrix: trusted proxy normalization, correlation, authentication, request limits, authorization context, handler, response policy, audit/telemetry and compression must not be assembled casually. **[I/E/P]**

### 6.1 HTTP implementation rules

- Model content negotiation explicitly; do not rely only on Axum `Json` extractors.
- Parse and preserve `Accept`, `Content-Type`, `Content-Encoding`, `Prefer`, conditional and range headers under bounded rules.
- Return standard headers and native representation media types; do not wrap every response in a proprietary envelope.
- Define one rejection mapper so body, path, query, media, auth and timeout failures produce the accepted safe problem catalog.
- Apply per-route body/decompression/item/time budgets before expensive parsing.
- Stream SSE and large responses with bounded buffers, cancellation and policy-bound cursors; never collect unbounded bodies.
- Treat client disconnect/cancellation as an observation, not proof that a transaction or physical effect did not commit.
- Keep an in-process router test harness using `tower::ServiceExt::oneshot` and full TCP/TLS tests for behaviors the harness cannot cover.

### 6.2 Required vertical slice

Before locking Axum, implement landing/conformance, one metadata collection/item, Observation query/create, RFC 9457 problems and one SSE resume path. Prove route/query encoding, link construction, media negotiation, ETags/conditional writes, large-body limits, middleware ordering, streaming cancellation, generated OpenAPI parity and external OS4CSAPI client behavior. Actix becomes fallback only if a documented blocker survives a bounded Axum design attempt. **[P/X]**

---

## 7. Async Runtime and Concurrency Model Evaluation

Tokio 1.53.1 is selected. Axum is explicitly designed for Tokio/Hyper, SQLx supports Tokio, and the messaging/telemetry candidates integrate with it. Runtime independence would increase testing and abstraction cost without an accepted deployment need. **[I/E/P]**

### 7.1 Concurrency rules

| Concern | Rule |
|---|---|
| task ownership | every spawned task belongs to a supervised `JoinSet`/worker group and has a shutdown path |
| cancellation | use cancellation tokens plus explicit safe points; dropping a future is not a business rollback contract |
| queues | bounded channels with documented overflow/backpressure; durable work lives in PostgreSQL |
| concurrency | semaphores/budgets per expensive capability, tenant/source and worker class |
| timeouts | deadline propagated through application/adapter calls; database statement/lock timeouts also configured |
| blocking work | bounded `spawn_blocking` or dedicated CPU pool for schema/XML/canonicalization/geometry; never block core workers |
| panics | task panic observed and classified; mandatory workers fail readiness/restart according to policy |
| shutdown | stop admission, cancel leases/streams, drain within deadline, persist state, close exporters/pool; force exit only after bound |
| locks | no async mutex held across external I/O; prefer ownership/messages/database transaction locks |
| fairness | batch size/yield points prevent backlog catch-up from starving API, command safety or audit |

Tokio documentation warns that long work between `.await` points blocks core threads and recommends `spawn_blocking` or a dedicated pool. Glaux must additionally cap those pools because Tokio's blocking-thread ceiling is deliberately large and hostile validation inputs could otherwise exhaust resources. **[I/E/P]**

### 7.2 Transaction and cancellation boundary

An HTTP future may be cancelled while PostgreSQL is committing. Application services therefore return durable operation/idempotency identity before or with effects, explicitly commit/rollback SQLx transactions, and allow status lookup. External command dispatch occurs only after durable intent. Cancellation stops cooperative work; it does not infer rollback, revoke an outbox record or retry a physical effect. **[A/P]**

---

## 8. Domain Modeling and Serialization Strategy Findings

### 8.1 Model layers

| Layer | Purpose | Rust posture |
|---|---|---|
| exact source artifact | original JSON/XML/binary and digest | immutable bytes/artifact reference; no lossy round trip |
| wire DTO | representation/profile-specific request/response | Serde derives with explicit rename/default/deny/extension policy |
| canonical domain | validated identity, relationship, temporal, provenance and lifecycle concepts | newtypes, non-exhaustive enums where appropriate, private fields/constructors |
| application command/query | use-case intent and policy/security context | typed inputs; no Axum/SQLx types |
| persistence record | table/query mapping | adapter-local SQLx rows; explicit conversion |
| derived projection | authorized current/latest/search/API view | provenance/watermark carried; rebuildable |

Do not use one `#[derive(Serialize, Deserialize, FromRow, ToSchema)]` struct for all six roles. That shortcut couples storage, public schema, defaults, redaction and evolution, and can silently expose fields. Shared types are allowed only when their semantics truly coincide and tests prove it. **[A/E/P]**

### 8.2 Type strategy

Use newtypes for `ResourceId`, `RevisionId`, `EventId`, `CommandId`, idempotency keys, source/node/epoch/sequence, media/profile URIs, digests, policy/trust versions and domain-specific times. Use enums for resource/lifecycle/outcome/command states with explicit unknown-extension handling where forward compatibility requires it. Use `NonZero*`, bounded collection wrappers, validated URLs and finite-number wrappers where invariants warrant. Keep occurrence/phenomenon/result/report/receipt/commit/sync times distinct rather than aliases. **[A/P]**

### 8.3 JSON and extensions

Serde 1.0.229 and serde_json 1.0.151 are selected. DTOs distinguish absent, explicit null and present value where the standard does. Unknown members are rejected on strict writes unless the selected extensible profile says to preserve them; then use a bounded map with name/value depth/size limits and reserved-name checks. Preserve exact input when round-trip fidelity matters. Never flatten policy, authority or internal metadata into untrusted extension maps. Canonical digesting uses a separately specified canonicalization, not ordinary `serde_json::to_vec` map order. **[I/A/P]**

### 8.4 XML, GeoJSON and SWE encodings

`quick-xml 0.42.0` is a candidate streaming parser/writer and `roxmltree 0.21.1` a read-only tree helper; neither is accepted as XSD conformance. Disable/avoid external entity/network behavior, bound nesting/text/attributes and retain exact SensorML bytes. `geojson 1.0.0` and `geo-types 0.7.20` are representation/domain helpers, with CRS/axis/bbox/validity checks owned by Glaux and PostGIS. SWE JSON/Text/Binary codecs require explicit versioned implementations, cursor/bounds checks and golden vectors; generic Serde is not enough for positional/binary encodings. **[A/I/P]**

---

## 9. OpenAPI, Schema, Validation, and Documentation Tooling Findings

### 9.1 OpenAPI decision

Use a hybrid contract pipeline:

1. Store the approved OGC OpenAPI artifacts and Glaux profile overlays as versioned reviewed inputs.
2. Keep a curated canonical served document for exact paths, parameters, media types, links, security, examples and conformance declarations.
3. Use Utoipa 5.5.0 annotations/types to generate handler/DTO fragments and route inventory where it proves helpful.
4. Normalize and structurally compare generated versus curated views in CI; require reviewed exceptions for intentional differences.
5. Execute contract tests against the running router and examples, because document equality cannot prove behavior.

Utoipa supports OpenAPI 3.1 and Axum integration, but automatic support for Serde attributes is explicitly partial. The official CSAPI artifact version and any 3.0/3.1 conversion semantics must therefore be pinned and tested. Aide 0.15.1 is retained as a prototype alternative, not dual production tooling. **[N/I/E/P]**

### 9.2 JSON Schema and validation

`jsonschema 0.56.0` is selected with an in-memory approved registry and `.offline()`/equivalent configuration. It supports multiple drafts and structured findings but documents varying compliance for newer drafts and defaults that can retrieve HTTP/file references. Glaux must disable ambient retrieval, precompile installed packages, cap regex/input complexity, pin drafts/formats and run the official/profile fixture corpus. **[I/A/P]**

Validation remains layered:

| Layer | Implementation |
|---|---|
| bytes/media/decompression | Axum/Tower body limits plus codec-specific bounds |
| parse/shape | Serde/quick-xml/SWE codec; stable pointer/path findings |
| JSON Schema/OpenAPI | offline registry and operation contract checks |
| SensorML/SWE | approved offline schema package plus semantic adapter |
| domain | explicit constructors/validators for IDs, relationships, time, units and lifecycle |
| interaction/current state | application service plus repository transaction |
| source/trust/policy | dedicated decision ports and immutable evidence |
| command safety | specialized pre-effect decision pipeline |

`validator 0.21.0` and `garde 0.23.0` may be prototyped for simple DTO field constraints, but neither is required initially; handwritten typed validators are preferred when stable error codes, conditional rules or evidence matter. Schemars 1.2.2 may generate internal schemas but must not compete with Utoipa and curated public schemas as another authority. **[I/E/P]**

### 9.3 Documentation pipeline

Build deterministic OpenAPI/JSON Schema artifacts in a dedicated tool binary/test, never with network access. Validate syntax, references, operation IDs, media types, examples, security declarations and conformance links. Fail CI on unexplained drift. Serve only committed/reproducibly generated artifacts matching the binary build. Documentation UI is optional and disabled or protected in operational profiles as policy requires. **[P]**

---

## 10. Error Handling and Problem-Detail Findings

Use `thiserror 2.0.20` for typed domain/application/adapter errors and `anyhow 1.0.104` only at binary startup, tooling or worker-supervision boundaries where callers cannot use a stable variant. Never branch business behavior on error strings and never expose anyhow chains/debug formatting to clients. **[I/E/P]**

### 10.1 Error architecture

| Type | Content | Mapping |
|---|---|---|
| domain finding | stable code, safe path/parameters, evidence references | validation/conflict catalog |
| application error | typed denied/not-found/conflict/precondition/unavailable/accepted-operation cases | transport-neutral outcome |
| adapter error | SQLSTATE/category, timeout, codec/source/broker detail | classified at boundary; protected diagnostics |
| public problem | RFC 9457 type/title/status/safe detail/instance/extensions | Axum `IntoResponse` adapter |
| audit/telemetry error | protected code/correlation/cause class | mandatory failure posture, not public internals |

One problem registry owns stable type URIs, status mapping, retryability, concealment and extensions. Authorization/policy runs before revealing resource-specific errors. Preserve original error sources internally for telemetry, but redact SQL, filesystem, host, token, schema, peer, policy and command details. Panic is never a client validation path; expected invalid input returns a typed error. **[A/P]**

---

## 11. Database, Geospatial, Time-Series, Migration, and Transaction Findings

### 11.1 Candidate comparison

| Candidate | Fit | Risk | Status |
|---|---|---|---|
| SQLx 0.9.0 | async, parameterized native SQL, compile-checked macros, pool, transactions, embedded migrations, PostgreSQL focus | new major release; MSRV 1.94; offline metadata/workflow; spatial custom types | **Select with database proof** |
| Diesel 2.3.13 | mature typed query DSL and migrations; sync core can be pooled/blocking | complex native SQL still required; async/DSL integration and spatial path add layers | Fallback for bounded repositories only |
| SeaORM 2.0.3 | async ORM, entity/migration ecosystem | new major/MSRV 1.94; entity abstraction mismatches append/evidence/spatial SQL | Do not select initially |

SQLx is chosen because Glaux needs transparent PostgreSQL SQL more than ORM-managed CRUD. Use the Postgres driver only, explicitly select Tokio and one reviewed TLS backend, disable unused database/runtime features, and check in prepared query metadata for offline CI. Dynamic filters use a parameterized query builder or a bounded compiler; identifiers/order expressions come only from allowlists. **[I/E/P]**

### 11.2 Transaction and repository rules

- Application services own transaction boundaries; repositories accept an explicit transaction/unit-of-work context when atomic composition is required.
- Use compile-checked static queries where practical and separately test dynamic, PostGIS, partition and administrative SQL.
- Map SQLSTATE/constraint names to stable application errors; do not expose raw database messages.
- Configure pool, acquisition, statement, idle-transaction and lock timeouts; expose protected saturation metrics.
- Retry only accepted serialization/deadlock classes around the whole side-effect-free transaction and preserve idempotency/preconditions.
- Inbox/outbox/audit/domain mutations follow the accepted atomicity classes; SQLx drop rollback is not a substitute for explicit commit/rollback paths.
- Migrations are forward, versioned, checksumed and tested against realistic prior snapshots; IDR-SRV-049 owns production choreography.

### 11.3 PostGIS and geometry

PostGIS remains spatial authority. Prefer database functions/operators and WKB/EWKB/GeoJSON adapters over relying on the older `postgis 0.9.0` crate as a core mapping layer. Prototype `geozero 0.15.1` for bounded WKB/EWKB streaming; use `geo`/`geo-types` only for well-defined in-memory calculations. CRS transforms remain PostGIS-side initially. The `proj` crate's native dependency/FFI enters only after a demonstrated offline requirement and security/build review. **[A/I/E/P]**

### 11.4 Native time-series approach

The first implementation uses PostgreSQL tables, range/time types, indexes, partitions and explicit latest/current queries through SQLx. No separate Rust time-series framework is needed. TimescaleDB stays behind the accepted benchmark/adoption gate; repository SQL must isolate extension-specific queries. Test phenomenon/result/report/commit ordering, tie semantics, partition routing, late data, retention and projection updates against real PostgreSQL—not mocks. **[A/P]**

---

## 12. Ingestion, Streaming, Event Outbox, and Background Worker Findings

### 12.1 Ingestion

Each adapter converts a bounded source envelope into an application command carrying source identity/epoch/sequence, message/batch ID, contract pins, clocks, digest, provenance and security context. The application transaction commits inbox outcome, canonical effect, provenance/audit and outbox. Adapter acknowledgements occur after durable outcome. Streaming parsers and item-level batch outcomes avoid unbounded buffering and silent partial success. **[A/P]**

### 12.2 Durable workers

Use PostgreSQL work/outbox tables with small claims, `FOR UPDATE SKIP LOCKED` only for work claiming, lease owner/expiry, attempt count, next-at, fence and last safe error. Supervised Tokio workers poll/wake, claim, release the transaction, perform bounded external work and persist acknowledgement/retry/dead-letter state. In-memory Tokio channels may wake workers or connect bounded pipeline stages; they never replace durable rows. **[A/P]**

### 12.3 Streaming and messaging

The first slice uses Axum/Hyper body streams and SSE over the accepted snapshot-watermark/cursor/replay log. Enforce per-subscriber buffers, heartbeat/lease, slow-consumer policy, authorization re-evaluation and graceful cancellation. Use bytes/futures/tokio-util primitives without inventing a global reactive abstraction. **[A/P]**

`rumqttc 0.25.1` is the MQTT 5 adapter candidate only after a proof of required properties, session/reconnect mapping, QoS behavior, backpressure, TLS/auth, topic safety and shutdown. It cannot supply domain cursors or exactly-once effects. `async-nats 0.50.0` is a prepared alternative adapter; Kafka requires a separate native/build/security review and is not a first dependency. Draft Part 3 remains its own disabled, version-pinned outward adapter. **[A/I/P/X]**

### 12.4 Background work failure posture

Workers classify retryable versus terminal conditions, use bounded exponential backoff/jitter, preserve logical IDs, and stop or degrade admission when mandatory audit/storage paths are unavailable. Process restart resumes from durable state. A spawned task handle, channel send, broker publish success or HTTP disconnect never marks domain work complete. **[A/P]**

---

## 13. Command/Control Implementation Support Findings

Rust can make many illegal in-process states harder to represent, but it cannot type-check a remote device's physical state. Implement Commands as stable identity plus immutable intent/status/result evidence and a derived projection. Use private constructors and transition methods that require expected revision, reporter authority, source sequence, fence, time and policy/safety evidence. Persist and revalidate every external input. **[A/P]**

### 13.1 Required seams

| Seam | Contract |
|---|---|
| command admission | validation, idempotency, authority, policy, safety, audit capacity, durable initial state |
| feasibility | immutable inputs/evidence/expiry; advisory only |
| dispatch repository | transactional intent/outbox; target serialization and fence |
| gateway adapter | exact target protocol mapping and stable operation identity |
| status receiver | authorized reporter, attempt/fence, legal transition and terminality |
| reconciliation | target query, buffered evidence, proven idempotent retry or operator disposition |
| simulator | compile/runtime isolated synthetic namespace and credentials |

No generic retry middleware wraps physical effects. Dispatch tickets are single-use, short-lived and verified immediately before effect. Use Tokio deadlines/cancellation only around protocol operations; unknown completion becomes durable reconciliation, not an automatic `FAILED` or redispatch. Command parameter/result logs use redacted structured summaries/digests. **[A/P]**

### 13.2 Testing implications

Provide deterministic fake clock/ID/policy/safety/gateway ports and real adapter contract tests. Model/property tests cover legal transitions, terminal monotonicity, cancel/complete races and fence uniqueness. Fault injection covers crash before dispatch, after send/before acknowledgement, duplicate status, stale gateway, lost response and audit failure. Tests must prove no duplicate effect under every claimed idempotent target profile. **[A/P]**

---

## 14. Security/Authn/Authz Implementation Support Findings

### 14.1 Security context and ports

One early middleware/adaptor stage constructs an immutable `SecurityContext` from verified TLS/proxy identity, token claims, client/workload mapping and request/channel evidence. Handlers cannot populate actor, roles, authority, source or policy decisions from JSON. Application services call narrow authentication-derived authorization/policy/source-trust/command-safety ports at the accepted enforcement points. **[A/P]**

### 14.2 Candidate guidance

- Prefer approved deployment TLS termination or rustls `0.23.45`; select and document exactly one crypto provider and trust-store behavior. Do not claim FIPS or accreditation from crate choice.
- Use `openidconnect 4.0.1`/`oauth2 5.0.0` where protocol discovery/client flows are actually needed. For local JWT validation, `jsonwebtoken 11.0.0` is a candidate only behind a verifier port with issuer, audience, time, algorithm and key-ID constraints, cache/refresh bounds and negative tests.
- Never implement cryptographic primitives. Use audited libraries/platform services and approved algorithms; prohibit algorithm confusion, unsigned tokens and caller-selected key URLs.
- Use `secrecy 0.10.3` or equivalent wrappers to reduce accidental debug exposure, while recognizing it does not secure process memory or replace a secret manager.
- Apply request/body/decompression/rate/concurrency budgets through Tower and application quota checks; rate limiting must be principal/resource aware and policy safe.

### 14.3 Parser and SSRF posture

All URI/reference fetching is denied by default. Schema/SensorML artifacts resolve from installed registries or an explicit allowlisted fetch service with DNS/IP/redirect/size/time/content controls. XML entities and unbounded expansions are disabled. URLs, regex, JSON depth, collections, geometry complexity, SWE dimensions and decompression ratios are bounded before costly work. Error timing/content is tested for existence and policy oracles. **[A/P]**

### 14.4 Security proof gate

Before operational deployment, prototype trusted-proxy/mTLS normalization, OIDC/JWT key rotation and outage behavior, object/function/property authorization, concealed not-found behavior, stream reauthorization, command pre-effect checks, audit failure, and offline bundle validation. Exact IdP, proxy, TLS, policy and secret products remain downstream. **[P/X]**

---

## 15. Configuration, Profiles, Feature Flags, and Secrets Findings

Use a typed, versioned `Settings` model loaded once at startup from explicit files/environment/CLI with deterministic precedence. The `config 0.15.25` and `clap 4.6.7` crates are candidates, not semantic owners. Parse into raw DTOs, validate cross-field invariants, normalize safe paths/URLs and only then construct immutable runtime settings. Emit a redacted effective-configuration fingerprint, never full secrets. **[I/E/P]**

### 15.1 Separation rules

| Mechanism | Appropriate use | Prohibited use |
|---|---|---|
| Cargo feature | compile optional adapter/code path, platform capability | security mode, tenant policy or frequently changed operations |
| deployment profile | coherent documented capability/security defaults | caller-selected downgrade |
| runtime setting | ports, limits, installed profile, dependency endpoints | secret value in logs or unvalidated arbitrary code behavior |
| policy package | authorization/releasability/safety decisions | ordinary app config shortcut |
| secret provider | credentials/keys by reference | committed plaintext or environment dump |

Unsafe development and simulator profiles are compile/runtime isolated, bind locally by default, use unmistakable synthetic namespaces and cannot be activated by a request. Hot reload is initially limited to explicitly reloadable, fully validated snapshots; security/policy packages follow their accepted activation/anti-rollback process. `arc-swap` may support atomic snapshots later but is not required for the first slice. **[A/P]**

IDR-SRV-047 decides exact precedence, secret providers, reload, environment naming and profile manifests. This report fixes only the typed/validated/immutable/redacted boundary. **[P]**

---

## 16. Observability, Logging, Metrics, Tracing, and Health Findings

Select `tracing 0.1.44` and `tracing-subscriber 0.3.23` for structured spans/events. Add OpenTelemetry/OTLP `0.32.0` behind an exporter adapter/feature so telemetry backend failure cannot redefine domain behavior. Use a single metrics facade/export strategy selected in IDR-SRV-048; `prometheus-client 0.25.1` and `metrics 0.24.6` are candidates, not simultaneous authorities. **[I/E/P]**

### 16.1 Telemetry contract

- Instrument HTTP route template, operation type, outcome class, latency, bytes and safe correlation—not raw URI IDs/queries.
- Trace application operations, SQL category, worker claim/delivery, policy decision references, stream lifecycle, command attempt and sync/audit outcomes using bounded attributes.
- Never record tokens, cookies, raw payloads, markings, secrets, command parameters/results, SQL bind values or hidden identities.
- Keep domain audit distinct from logs/traces/metrics; telemetry loss does not erase mandatory audit evidence.
- Propagate accepted trace context only after validation and do not trust external sampling/attributes as authority.
- Bound queues/export time; exporters may drop telemetry under declared policy, never block mandatory command/audit paths indefinitely.

### 16.2 Health and shutdown

Expose minimal liveness, readiness and protected detailed status. Liveness proves the process can respond; readiness checks admission-critical local dependencies/configuration without synchronously probing every remote peer; capability health reports optional source/broker/policy states under authorization. Include build/version/profile and migration compatibility safely. Graceful shutdown stops readiness/admission before draining tasks and telemetry. IDR-SRV-048 sets exact endpoints, metrics and SLOs. **[A/P]**

---

## 17. Testing, CI, Quality-Gate, Static-Analysis, and Supply-Chain Findings

### 17.1 Test layers

| Layer | Primary tools | Purpose |
|---|---|---|
| unit/domain | built-in test, rstest selectively | constructors, invariants, state transitions, policy-free algorithms |
| property/model | proptest 1.11.0 | temporal/query/state-machine/canonicalization invariants |
| golden/snapshot | explicit golden files; insta 1.48.0 selectively | exact JSON/XML/OpenAPI/problem/event artifacts with reviewed updates |
| adapter contract | shared test suites, wiremock 0.6.5 | repository, source, gateway, identity/policy, broker contracts |
| database integration | SQLx tests + testcontainers 0.28.0/CI service | real PostgreSQL/PostGIS transactions, migrations, SQL and concurrency |
| router/API | Tower oneshot plus real listener | route/media/header/problem/OpenAPI behavior |
| conformance/interoperability | external harness and OS4CSAPI clients | published requirements and multi-client behavior |
| fuzz/security | cargo-fuzz 0.13.2, parser corpora | JSON/XML/SWE/query/cursor/envelope and state-machine robustness |
| performance | Criterion 0.8.2 plus scenario/load harness | micro-regression and end-to-end capacity/latency |

Snapshots are review artifacts, not “accept all changes” tests. Normalize only explicitly non-semantic fields; do not erase ordering, media, precision or link differences that the standards expose. Database mocks cannot validate SQL, isolation, PostGIS or partition behavior. Time/ID/randomness are injectable for deterministic tests but production entropy/clock paths remain separately tested. **[A/P]**

### 17.2 Pull-request blocking gates

1. pinned `cargo fmt --check`;
2. `cargo check` and `cargo test`/nextest on pinned stable, all targets and the supported feature/profile matrix;
3. MSRV build/test for core supported features;
4. Clippy with workspace warnings denied and reviewed localized allowances;
5. database migration and integration tests for changed persistence paths;
6. deterministic generated artifacts, Utoipa/OpenAPI structural drift and SQLx offline metadata checks;
7. cargo-deny advisory/license/ban/source checks plus RustSec audit policy;
8. no uncommitted generated or migration changes and repository secret scanning;
9. required platform/container build and smoke test; and
10. changed-code contract/conformance fixtures where applicable.

### 17.3 Scheduled/release/advisory gates

Run cargo-llvm-cov, cargo-fuzz corpora, performance baselines, full feature powerset, slow conformance/interoperability/security suites, container vulnerability/SBOM/signing, dependency freshness and cross-platform/cross-compile tests on schedules/releases according to cost. Use cargo-semver-checks for any published/shared crates. Treat `cargo machete`/unused-dependency tools, `cargo outdated` and unsafe inventories as advisory until false-positive behavior is understood. **[I/E/P]**

### 17.4 TDD handoff

IDR-SRV-052 must turn accepted report invariants into a red-green-refactor sequence, define test doubles versus real dependencies, ownership of golden files, flake/quarantine rules, coverage interpretation and test pyramid. This report fixes the tooling candidates and the principle that an executable failing contract precedes each standards-visible behavior. **[P]**

---

## 18. Unsafe-Code, Dependency, License, and Maintenance-Risk Findings

### 18.1 Unsafe policy

Set workspace lint policy to forbid unsafe code in first-party production crates by default. If a proven capability later requires first-party unsafe/FFI, isolate it in one small adapter crate, document the safety invariant and threat model, require dedicated reviewers/tests/fuzzing and change the policy through an accepted decision. Build scripts/proc macros and transitive unsafe are inventoried separately; `#![forbid(unsafe_code)]` cannot truthfully claim they are absent. **[E/P]**

### 18.2 Dependency admission

For every direct/high-risk transitive dependency record: purpose and alternative; version/MSRV; exact enabled features; repository/maintainer/release health; advisories/yanks; license/source; unsafe/build/proc-macro/native code; network/filesystem behavior; cryptographic role; transitive count; platform support; and exit strategy. Security-sensitive crates receive code/API review and negative tests. Minimize default features and prohibit unreviewed git/path dependencies in release builds. **[P]**

### 18.3 License and provenance

The proposed direct stack is predominantly MIT, Apache-2.0 or compatible dual licensing according to current crates.io metadata, but final compatibility must be evaluated from the resolved feature graph and project distribution obligations. `cargo-deny` enforces allowed/denied/clarified licenses, duplicate bans and approved registries/sources. Produce an SPDX or CycloneDX SBOM, provenance/attestation and notices at release. Exact organizational license policy remains project/legal authority. **[I/E/P/X]**

### 18.4 Maintenance policy

Commit `Cargo.lock` for binaries, use `cargo install --locked` or prebuilt pinned CI tools, and update dependencies in small reviewed batches with changelog, advisory, MSRV, feature and regression review. Define a monthly routine and expedited security path. Do not automatically merge major/minor updates for HTTP, parser, database, auth, crypto, serialization, OpenAPI or command-adapter crates. Maintain architecture seams that permit replacement without leaking crate types into domain APIs. **[P]**

### 18.5 Cryptography

Glaux implements no cryptographic primitive. It uses approved protocol/library providers, configuration and platform services, records versions/algorithms, supports rotation and tests invalid cases. A rustls/aws-lc/ring/OpenSSL choice is a deployment and assurance decision; a crate's marketing or pure-Rust implementation is not a compliance claim. **[E/P]**

---

## 19. Recommended First-Implementation Stack

### 19.1 Selected baseline

| Capability | Candidate | Version/date reviewed | Fit | Standards impact | Testing impact | Security impact | Operations/deployment | Maturity/maintenance | License/supply chain | Recommendation | Downstream | Notes/unresolved |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| language | Rust stable, edition 2024 | 1.98.1 / 2026-09-03 | approved safe systems language | no conformance by itself | MSRV+stable lanes | first-party safety baseline | compiled binary | official six-week train | Rust licenses/toolchain provenance | Select | 045/046/051 | MSRV 1.94 while SQLx 0.9 |
| runtime | Tokio | 1.53.1 / 2026-07-20 | HTTP/DB/worker async core | preserves explicit semantics | paused time/async tests | bounds/cancellation required | tune workers/blocking pools | mature ecosystem | MIT; transitive review | Select | 045/048/054 | no runtime abstraction |
| HTTP API | Axum | 0.8.9 / 2026-04-14 | thin handlers/extractors | direct header/media control | Tower in-process tests | rejection/middleware order proof | Hyper/Tokio aligned | active/mature | MIT | Select with proof gate | 045/050/056 | Actix fallback |
| middleware | Tower/Tower HTTP | 0.5.3/0.7.1 / 2026 | reusable service layers | timeouts/limits/tracing only | layer-order matrix | sensitive ordering | reusable across adapters | mature | MIT | Select narrowly | 045/055 | disable unused features |
| HTTP engine/types | Hyper/http/bytes | 1.11.1/1.5.0/1.12.1 / 2026 | Axum substrate/streaming | protocol primitives | external listener tests | parser/limits upstream | efficient IO | foundational | MIT/Apache mix | Indirect/select types | 045 | avoid raw Hyper app |
| JSON | Serde/serde_json | 1.0.229/1.0.151 / 2026-07-18/20 | DTO serialization | exact field/null policy explicit | golden/property tests | depth/size/extension bounds | low runtime cost | foundational | MIT/Apache-2.0 | Select | 045/053 | domain types separate |
| time/IDs | time/uuid | 0.3.55/1.26.1 / 2026 | RFC3339 and UUIDv7 support | separate domain time semantics | clock/parse vectors | entropy/time validation | portable | active | MIT/Apache | Select | 045/053 | wrap in domain newtypes |
| OpenAPI | curated artifacts + Utoipa | 5.5.0 / 2026-05-04 | hybrid route/DTO evidence | curated OGC contract stays authority | deterministic semantic diff | security declarations tested | build-time generation | active; macro risk | MIT/Apache | Select with proof gate | 050/052 | 3.0/3.1 mapping proof |
| JSON Schema | jsonschema | 0.56.0 / 2026-09-10 | multi-draft validator/registry | structural layer only | official/profile corpus | offline, regex/complexity bounds | precompile/cache | active; very recent | MIT | Select with proof gate | 045/050/055 | disable ambient retrieval |
| XML | quick-xml | 0.42.0 / 2026-08-22 | streaming XML adapter | not XSD/SensorML conformance | exact/golden/malicious corpora | entity/depth/resource controls | pure-Rust parser candidate | active | MIT | Conditional proof | 045/050/053 | validation engine open |
| database | SQLx Postgres | 0.9.0 / 2026-05-21 | explicit async PostgreSQL SQL | preserves transaction/query rules | real DB + offline metadata | parameterization/TLS/features | pool/migrations/tooling | active but new major | MIT/Apache | Select with proof gate | 045/049/052 | MSRV 1.94; no `Any` |
| spatial | PostGIS + geo-types/geojson/geozero | 0.7.20/1.0.0/0.15.1 / 2026 | database authority + adapters | CRS/axis rules explicit | WKB/GeoJSON/DB vectors | geometry complexity/FFI bounded | PostGIS runtime dependency | mixed maturity | MIT/Apache | Select bounded set | 045/046/053 | defer PROJ crate |
| errors | thiserror + anyhow boundary | 2.0.20/1.0.104 / 2026 | typed layers/startup context | RFC9457 mapping explicit | mapping/redaction tests | no debug leakage | low impact | mature | MIT/Apache | Select | 045/050 | anyhow not public/domain |
| tracing | tracing/subscriber | 0.1.44/0.3.23 / 2025-12/2026-03 | structured spans/events | no audit substitution | capture/redaction tests | field allowlist | local JSON/log baseline | mature | MIT | Select | 048/055 | bounded fields |
| telemetry export | OpenTelemetry/OTLP | 0.32.0 / 2026-05-08 | backend-neutral adapter | none | exporter failure tests | telemetry disclosure review | optional feature | active/churn possible | Apache-2.0 | Conditional adapter | 048 | not core behavior |
| configuration | config + clap | 0.15.25/4.6.7 / 2026 | typed startup inputs | profile declarations exact | precedence/redaction tests | no secret dumps/downgrade | common deployment inputs | active | MIT/Apache | Select conditionally | 047 | simple loader may suffice |
| secret wrapper | secrecy | 0.10.3 / 2024-10-09 | accidental-exposure guard | none | debug/redaction tests | not a secret manager | low impact | stable/slower release | MIT/Apache | Select where useful | 047/055 | external provider later |
| auth protocols | openidconnect/oauth2/verifier adapter | 4.0.1/5.0.0 / 2025 | standards libraries behind port | issuer/aud/alg behavior explicit | rotation/outage/negative tests | high-risk review | IdP/proxy dependent | established | MIT/Apache | Conditional proof | 045/046/055 | no IdP selection |
| TLS | rustls or deployment termination | 0.23.45 / 2026-09-14 | modern TLS option | HTTP semantics unchanged | cipher/cert/proxy tests | provider/compliance decision | affects images/certs | mature/active | Apache/ISC/MIT | Conditional | 046/047/055 | no FIPS claim |
| unit/integration | cargo test + nextest/rstest | 0.9.144/0.27.0 / 2026-09 | fast test execution | all contract layers | primary CI | test isolation | CI tooling | active | MIT/Apache | Select | 051/052 | pin tool versions |
| property/golden | proptest + explicit goldens/insta | 1.11.0/1.48.0 / 2026 | invariant/artifact coverage | detects edge drift | reviewed snapshots | malicious generators | CI storage/corpus | active | MIT/Apache | Select bounded | 052/053 | never blind snapshot accept |
| database tests | testcontainers or CI Postgres | 0.28.0 / 2026-08-06 | real PostgreSQL/PostGIS | validates SQL semantics | mandatory persistence lane | image provenance | Docker/CI dependency | active | MIT/Apache | Select one per environment | 051/052 | pin image digest |
| fuzz/perf/coverage | cargo-fuzz/Criterion/llvm-cov | 0.13.2/0.8.2/0.9.1 / 2026 | robustness/regression evidence | codecs/query/state machines | scheduled/release gates | parser/DoS discovery | specialized runners | active | MIT/Apache | Select | 052/054/055 | thresholds downstream |
| dependency policy | cargo-deny + cargo-audit | 0.20.2/0.22.2 / 2026 | graph/license/source/advisory checks | none | blocking CI policy | supply-chain signal | CI only | established | MIT/Apache | Select | 051/055 | advisory exceptions expire |
| first live stream | Axum SSE + durable log | stack versions above | accepted first slice | native CSAPI/profile semantics | replay/backpressure tests | auth/cursor controls | no mandatory broker | simple first baseline | existing stack | Select | 045/048/054 | WebSocket not first |
| MQTT | rumqttc adapter | 0.25.1 / 2025-11-21 | candidate MQTT 5 client | transport not domain semantics | broker matrix required | TLS/topic/session review | optional broker | active but proof needed | Apache-2.0 | Prepared adapter | 045/046/056 | Part 3 remains experimental |

### 19.2 Initially rejected or deferred choices

- Do not select a full ORM, runtime-independent abstraction, service mesh SDK, general workflow engine, Kafka/NATS runtime, native PROJ/GDAL binding, automatic OpenAPI-as-authority generator, or dynamic plugin ABI for the first slice.
- Do not introduce GraphQL, a generic event-sourcing framework, actor framework or CRDT library without an accepted requirement and invariant proof.
- Do not make OpenTelemetry, Swagger UI, MQTT, TimescaleDB or an external policy engine mandatory for basic server startup.
- Do not use SQLite as a behavioral substitute for PostgreSQL/PostGIS tests.
- Do not pursue fully static musl linking until TLS, DNS, PostGIS client, allocator, debugging and target-platform measurements justify it.

---

## 20. Full-Scope Readiness Roadmap and Proof-of-Concept Needs

### 20.1 Ordered proof gates

1. **HTTP/contract slice:** Axum routes, negotiation, ETags, problems, OpenAPI hybrid and OS4CSAPI client.
2. **Persistence slice:** SQLx 0.9 offline metadata, migrations, PostGIS geometry, range/time, partition, transaction retry, inbox/outbox and concurrent tests.
3. **Representation slice:** exact JSON/GeoJSON and SensorML XML preservation, safe XML/XSD/semantic validation, `$ref` offline packages and stable findings.
4. **Dynamic-data slice:** SWE JSON/Text/Binary staged codecs, Observation batch/late/latest behavior and bounded streaming parse.
5. **Publication slice:** snapshot/watermark/SSE replay, slow consumers, policy changes and graceful shutdown.
6. **Security slice:** trusted proxy/mTLS, OIDC/JWT rotation/outage, object/property authorization, concealment and request budgets.
7. **Command slice:** durable dispatch ticket/fence, gateway fake and one real protocol profile with crash/unknown-outcome tests—without live operational enablement.
8. **Audit/sync slice:** E0–E5 failure behavior, local spool, manifest/inbox/quarantine/conflict and protected diagnostics.
9. **MQTT/Part 3 slice:** rumqttc property/QoS/session/topic mapping and pinned experimental outbound profile.
10. **Build/supply-chain slice:** reproducible container, SBOM/provenance, unsafe/native inventory, MSRV/cross-platform and dependency-update drill.

Each proof produces a decision record, minimal benchmark, dependency/feature tree, threat notes, fixtures and exit criteria. Failure returns to the named fallback; it does not silently expand architecture. **[P]**

### 20.2 Phasing

| Phase | Platform scope |
|---|---|
| foundation | toolchain/workspace/lints, domain IDs/time/errors, curated contracts, CI and local PostgreSQL/PostGIS |
| read/API vertical | landing/conformance, metadata read, negotiation, validation, policy hooks, observability |
| write/dynamic | conditional writes, transactions/idempotency, Observation ingest/query/latest, audit/outbox |
| live/async | durable workers, SSE replay, feasibility, bounded source adapters |
| command/security | full authority/safety/fence/audit hooks with simulator/gateway proofs |
| DDIL/sync/full profile | offline packages, MQTT experimental profile, synchronization/conflicts, deployment variants |

### 20.3 Workspace direction for IDR-SRV-045

Begin as a modular monolith, likely separating domain, application, contracts/validation, infrastructure adapters, HTTP API, server binary and test support only when dependency direction is enforceable. Domain/application crates must not depend on Axum, SQLx, broker or telemetry exporter types. Keep adapter-specific models local. IDR-SRV-045 decides the exact crate graph; this is a constraint, not a pre-created architecture. **[P]**

---

## 21. Downstream Topic Handoff Matrix

| Downstream topic | Fixed handoff | Remaining decision |
|---|---|---|
| IDR-SRV-045 architecture | modular monolith first; domain/application independent of Axum/SQLx; narrow effect ports; selected stack/proofs | exact crates, dependency graph, service/process boundaries |
| IDR-SRV-046 deployment | compiled Rust binary, PostgreSQL/PostGIS, optional adapters/exporters; no static-musl assumption | topology, HA, proxy/TLS, images, resource profiles |
| IDR-SRV-047 configuration | typed validated immutable settings, profiles distinct from features/policy, secrets by reference | precedence, reload, secret providers, manifests |
| IDR-SRV-048 observability | tracing core, optional OTLP, bounded attributes, health dimensions, audit separation | metric facade/backend, endpoints, SLOs/alerts |
| IDR-SRV-049 migration/restore | SQLx migration candidate; checksummed forward migrations; restore must preserve IDs/epochs/inboxes | choreography, compatibility windows, RPO/RTO and rollback |
| IDR-SRV-050 conformance | curated OpenAPI authority, generated drift, executable router/client contracts | harness implementation and evidence packaging |
| IDR-SRV-051 developer workflow | pinned toolchain/tools, locked builds, local PostGIS and blocking PR gates | CI provider, caches, dev containers, branch/release flow |
| IDR-SRV-052 TDD | test layers/tools, real DB, property/golden/fuzz and failing-contract-first rule | sequence, doubles, coverage and flake policy |
| IDR-SRV-053 fixtures | exact source/wire/canonical artifacts and reviewed goldens | corpus layout, generators and controlled-data exclusions |
| IDR-SRV-054 performance | Tokio/SQLx/Axum benchmark dimensions, backlog/stream/blocking pools | targets, workloads, hardware and regression budgets |
| IDR-SRV-055 security tests | parser/resource, auth rotation, policy concealment, command/audit and supply-chain proofs | tools, threat campaigns and deployment assurance |
| IDR-SRV-056 interoperability | Axum vertical slice against OS4CSAPI/external clients; exact profile/version | server/client matrix and partner systems |
| IDR-SRV-057 synthesis | selected candidate stack, proof gates and unresolved decisions | final implementation sequencing/acceptance |

IDR-SRV-045 and later topics remain unauthorized until this report is accepted. **[P]**

---

## 22. Risks, Constraints, and Open Questions

### 22.1 Risks and controls

| Risk | Control |
|---|---|
| generated OpenAPI diverges from CSAPI | curated authority, normalized diff and executable contract tests |
| SQLx 0.9 recency/MSRV creates instability | mandatory DB proof, pinned version and fallback decision |
| async cancellation loses/duplicates work | durable IDs/state, structured tasks and crash-window tests |
| XML/SensorML validation gap | exact-source preservation and offline validator PoC before write support |
| macro-heavy compile times/errors | dependency budget, concrete types, limited derives and measurement |
| crate types leak into core | conversion boundaries and architecture tests |
| transitive unsafe/native code overlooked | inventory, cargo-deny/audit, SBOM and high-risk review |
| optional adapters become mandatory | feature/port separation and startup/profile tests |
| policy/security logic delegated to middleware only | application enforcement ports at every accepted PEP |
| command retry causes duplicate physical effect | durable fences/identity and adapter-specific proof |
| logs/traces leak protected data | typed safe fields, denylist tests and audit separation |
| dependency churn/MSRV creep | lockfile, pinned tools, scheduled reviewed updates and dual CI lanes |

### 22.2 Open questions and owners

| Question | Owner |
|---|---|
| exact Axum crate/module/router organization | IDR-SRV-045 |
| Utoipa 3.0/3.1 transformation and canonical document workflow | proof gate + IDR-SRV-050 |
| production-grade SensorML XSD/semantic validator implementation | IDR-SRV-045/050/055 proof |
| SQLx 0.9 adoption versus temporary maintained fallback | database proof + project lead |
| geozero/custom EWKB type path | persistence/spatial proof |
| auth verifier/TLS crypto provider and assurance regime | IDR-SRV-046/055 and security authority |
| configuration crate versus small custom loader | IDR-SRV-047 |
| metrics facade/export format | IDR-SRV-048 |
| exact container/libc/cross-compile targets | IDR-SRV-046/051 |
| dependency/license approval policy and cargo-vet adoption | project/legal/security + IDR-SRV-051/055 |
| first operational MQTT profile | later architecture/deployment decision |

---

## 23. Validation Against This Plan's Success Criteria

| Success criterion | Evidence | Result |
|---|---|---|
| Rust fit, constraints, risks and mitigations evaluated without reopening language | Sections 3–5 | Met |
| all required stack/tool areas evaluated | Sections 6–18 and matrix in Section 19 | Met |
| first stack and full-scope candidates documented | Sections 19–20 | Met |
| recommendations tied to accepted server obligations and standards | Sections 2, 4 and each capability section | Met |
| Rust-specific risks and mitigations documented | Sections 5, 7, 18 and 22 | Met |
| security, unsafe, dependency, license, supply chain and CI addressed | Sections 14, 17–18 | Met |
| testing/TDD handoff explicit | Sections 17 and 21 | Met |
| deployment/config/observability/migration/conformance/fixture/performance/security/interoperability handoffs explicit | Section 21 | Met |
| references reproducible with versions/dates/assumptions | Sections 3, 19 and 24 | Met |

The research selects a preferred platform direction while retaining the plan's boundaries. Exact architecture, products, operational thresholds and implementation remain downstream. The deliverable remains **In Review** until project-lead acceptance. **[P]**

---

## 24. References

### 24.1 Rust language, ecosystem and supply-chain sources

1. Rust Project, [Announcing Rust 1.98.1](https://blog.rust-lang.org/releases/latest/), 2026-09-03.
2. Rust Project, [The Rust Programming Language](https://doc.rust-lang.org/book/) and [Cargo Book](https://doc.rust-lang.org/cargo/), checked 2026-09-15.
3. Rust Project, [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/) and [Rustonomicon](https://doc.rust-lang.org/nomicon/), checked 2026-09-15.
4. RustSec, [Advisory Database and cargo-audit](https://rustsec.org/), checked 2026-09-15.
5. Embark Studios, [cargo-deny](https://embarkstudios.github.io/cargo-deny/), Version 0.20.2 snapshot.
6. [crates.io](https://crates.io/) API metadata and [docs.rs](https://docs.rs/) documentation for versions listed in Sections 3 and 19, retrieved 2026-09-15.

### 24.2 Framework/runtime/representation sources

7. Tokio Project, [Tokio 1.53.1 documentation](https://docs.rs/tokio/1.53.1/tokio/).
8. Tokio Project, [Axum 0.8.9 documentation](https://docs.rs/axum/0.8.9/axum/), [Tower 0.5.3](https://docs.rs/tower/0.5.3/tower/), and [Hyper 1.11.1](https://docs.rs/hyper/1.11.1/hyper/).
9. Actix Project, [Actix Web 4.15.0](https://docs.rs/actix-web/4.15.0/actix_web/).
10. Serde Project, [Serde 1.0.229](https://docs.rs/serde/1.0.229/serde/) and [serde_json 1.0.151](https://docs.rs/serde_json/1.0.151/serde_json/).
11. Utoipa Project, [Utoipa 5.5.0](https://docs.rs/utoipa/5.5.0/utoipa/).
12. jsonschema Project, [jsonschema 0.56.0](https://docs.rs/jsonschema/0.56.0/jsonschema/).
13. quick-xml Project, [quick-xml 0.42.0](https://docs.rs/quick-xml/0.42.0/quick_xml/).

### 24.3 Persistence, spatial, telemetry and testing sources

14. SQLx Project, [SQLx 0.9.0](https://docs.rs/sqlx/0.9.0/sqlx/).
15. Diesel Project, [Diesel 2.3.13](https://docs.rs/diesel/2.3.13/diesel/) and SeaQL, [SeaORM 2.0.3](https://docs.rs/sea-orm/2.0.3/sea_orm/).
16. PostgreSQL Global Development Group, [PostgreSQL 18 Documentation](https://www.postgresql.org/docs/current/), checked 2026-09-15.
17. PostGIS Project, [PostGIS Documentation](https://postgis.net/documentation/), checked 2026-09-15.
18. Tokio Project, [tracing](https://docs.rs/tracing/0.1.44/tracing/) and [tracing-subscriber](https://docs.rs/tracing-subscriber/0.3.23/tracing_subscriber/).
19. OpenTelemetry, [OpenTelemetry Rust 0.32.0](https://docs.rs/opentelemetry/0.32.0/opentelemetry/) and [OTLP exporter](https://docs.rs/opentelemetry-otlp/0.32.0/opentelemetry_otlp/).
20. nextest Project, [cargo-nextest](https://nexte.st/); proptest, testcontainers-rs, Criterion, cargo-fuzz and cargo-llvm-cov official documentation for the versions in Section 19.

### 24.4 Standards and accepted project evidence

21. OGC, [OGC API - Connected Systems Part 1](https://docs.ogc.org/is/23-001/23-001.html) and [Part 2](https://docs.ogc.org/is/23-002/23-002.html), Version 1.0.
22. OGC, [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) and [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html).
23. IETF, [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110) and [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457).
24. OpenAPI Initiative, [OpenAPI Specification](https://spec.openapis.org/oas/latest.html); JSON Schema, [Specification](https://json-schema.org/specification), checked 2026-09-15.
25. [IDR-SRV-025 Database and Persistence Architecture Options](idr-srv-025-database-and-persistence-architecture-options-report.md).
26. [IDR-SRV-029 Transaction, Consistency, Idempotency, and Concurrency Strategy](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md).
27. [IDR-SRV-035 Streaming and Event Publication Strategy](idr-srv-035-streaming-and-event-publication-strategy-report.md).
28. [IDR-SRV-038 Command Authorization, Safety, and Audit Strategy](idr-srv-038-command-authorization-safety-and-audit-strategy-report.md).
29. [IDR-SRV-039 Authentication, Authorization, and API Security Threat Model](idr-srv-039-authentication-authorization-and-api-security-threat-model-report.md).
30. [IDR-SRV-043 Server Synchronization and Conflict Handling Boundary](idr-srv-043-server-synchronization-and-conflict-handling-boundary-report.md).
31. Accepted [IDR-SRV-014A](idr-srv-014a-osh-csapi-server-implementation-study-report.md) through [IDR-SRV-014H](idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md) implementation and interoperability studies.

### 24.5 Reproducibility and assumptions

- Rust release, crate version, release-date, license and declared `rust_version` evidence was frozen on 2026-09-15 from official project sources and crates.io metadata.
- Versions are candidate evaluation pins; the future implementation must generate and review the actual resolved dependency/feature graph and advisories.
- Mutable documentation links are paired with explicit versions in the report matrix.
- No crate was installed or executed as production code and no benchmark result is claimed by this research.
- The shared upstream-history register remains Version 1.12; no material published CSAPI Part 1/2 history change was found by this platform topic.

---

## Report Completion Checklist

- [x] All 24 required report sections are present.
- [x] The implementation stack matrix contains every required field.
- [x] Current versions, dates, licenses, MSRV implications and evidence limits are explicit.
- [x] First-stack selections are separated from fallbacks, optional adapters and proof gates.
- [x] Standards authority is separated from generated/framework behavior.
- [x] Rust async, unsafe, dependency, license, supply-chain and maintenance risks are addressed.
- [x] Test/CI and every required downstream handoff are explicit.
- [x] IDR-SRV-045 and implementation remain unauthorized pending acceptance.

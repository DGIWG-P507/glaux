# Section 045: Service Architecture and Modularization Strategy - Research Report

**Topic ID:** IDR-SRV-045<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-045 Service Architecture and Modularization Strategy](../IDR%20Plans/idr-srv-045-service-architecture-and-modularization-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All questions concerning architecture style, crate/module boundaries, dependency direction, API/domain/standards separation, validation, persistence and transaction ownership, ingestion, publication, commands, security/policy/audit, DDIL/synchronization, configuration, observability, internal ports, extension/extraction points, repository layout, testability, implementation phasing, proof gates, and downstream handoffs<br>
**Methodology Used:** Accepted-requirement extraction; architecture-style comparison; dependency and transaction-boundary analysis; capability/port matrixing; cross-cutting enforcement tracing; current Rust/Cargo/framework constraint review; implementation-lesson reconciliation; phased vertical-slice and proof-gate synthesis<br>
**Research Time:** Approximately 36 hours of AI-assisted execution on September 15, 2026<br>
**Accepted Platform Baseline:** IDR-SRV-044 Rust 1.98.1/edition 2024 candidate stack with Axum/Tokio/Tower/Hyper, Serde, hybrid OpenAPI, offline validation, SQLx/PostgreSQL/PostGIS, durable workers, SSE, security/telemetry seams, first-party unsafe prohibition, CI gates, and ordered proofs<br>
**Standards Baseline:** OGC API - Connected Systems Parts 1 and 2 Version 1.0; SensorML 3.0; SWE Common 3.0; OGC API - Features; RFC 9110/9457; OpenAPI/JSON Schema; accepted Glaux IDR-SRV-001 through IDR-SRV-044<br>
**Current Architecture Evidence:** Cargo workspace/resolver 3, Axum 0.8.9, Tower 0.5.3, Tokio 1.53.1, SQLx 0.9.0 and related official documentation checked September 15, 2026<br>
**Implementation Evidence:** Accepted OSH, Connected Systems Go, pygeoapi, SECD, client/interoperability/community, and Part 3 studies used only as non-normative lessons<br>
**Document Purpose:** Define a practical internal architecture and dependency contract without selecting deployment topology, creating implementation code, or authorizing later Category H work<br>
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
| I | Current implementation/framework or study evidence; not a requirement |
| E | Engineering, security, maintainability, or interoperability inference |
| P | Proposed architecture decision requiring this report's acceptance |
| X | Open parameter, proof gate, or downstream decision |

Architecture names are used as design tools, not purity claims. “Modular monolith,” “ports and adapters,” “layered,” and “vertical slice” describe complementary choices here: one initial authoritative application/deployment boundary; acyclic packages; dependencies toward domain/application policy; effect adapters at real seams; and behavior delivered in end-to-end capability slices. **[E/P]**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Architecture Requirement Extraction Methodology
5. Architecture Style Evaluation
6. Recommended Initial Architecture Style
7. Module/Crate Boundary Findings
8. Dependency-Direction and Layering Findings
9. Internal Contract and Extension Point Findings
10. API/Domain/Standards-Model Boundary Findings
11. Validation/Persistence/Ingestion/Event/Command Boundary Findings
12. Security/Policy/Audit/DDIL/Synchronization Boundary Findings
13. Observability/Configuration/Test-Support Boundary Findings
14. Repository/Workspace Layout Findings
15. Phased Implementation Guidance
16. Proof-of-Concept Needs
17. Downstream Topic Handoff Matrix
18. Recommendations
19. Risks, Constraints, and Open Questions
20. Validation Against This Plan's Success Criteria
21. References

---

## 1. Executive Summary

Glaux Server should begin as a **Cargo-workspace modular monolith** with ports and adapters at external-effect and policy boundaries, capability-oriented domain/application modules, one composition root, one authoritative PostgreSQL/PostGIS write core, and one initial server release artifact. This minimizes distributed failure and deployment burden while enforcing compile-time package boundaries and preserving future extraction points. It is a staged hybrid: modular monolith now; optional out-of-process adapters or service extraction only after independent scaling, failure-isolation, security, deployment or ownership evidence exists. **[A/E/P]**

The architecture is neither one undifferentiated crate nor a microservice estate. Start with eight production-oriented workspace packages: `glaux-domain`, `glaux-application`, `glaux-standards`, `glaux-validation`, `glaux-persistence-postgres`, `glaux-api-http`, `glaux-adapters`, and the `glaux-server` composition/binary package. Add `glaux-test-support` and `xtask` as non-production-support packages. Catalog, dynamic data, control/tasking, evidence/governance, publication and continuity remain internal capability modules until a crate boundary measurably improves dependency control, compile cost or ownership. **[E/P]**

Dependencies point inward. Domain has no Axum, SQLx, broker, telemetry exporter, environment/config loader or deployment dependency. Application depends on domain, defines use cases and the ports it consumes, and owns semantic transaction, authorization, validation, audit and effect ordering. Standards/representation code maps approved CSAPI, SensorML, SWE, GeoJSON and problem contracts without becoming the domain model. API, persistence, validation and external adapters depend on application/domain contracts and implement them. The binary composition root is the only package that knows every concrete adapter. **[A/E/P]**

Handlers stay thin: parse/bound transport inputs, establish trusted request/security context, negotiate a representation, call one application use case, then assemble a standards-facing response or RFC 9457 problem. Handlers do not query SQLx, select policy, construct ad hoc links, emit mandatory audit records, manage outbox rows or perform command dispatch. Axum `State` exposes a narrow `ApiServices` facade rather than raw pools, configuration or broker clients. **[I/E/P]**

Do not create a generic `Repository<T>` or leak `sqlx::Transaction` across package boundaries. Application ports should be capability/use-case oriented. Write ports accept already validated/authorized decision inputs and atomically commit the exact bundle required by accepted semantics—idempotency/inbox, domain revision/evidence, provenance, mandatory audit and outbox. The application defines what must be atomic; the PostgreSQL adapter owns SQL and transaction mechanics. Read ports accept authorized query plans and return explicit domain/query projections with watermark/completeness evidence, avoiding fetch-all-then-filter designs. **[A/E/P]**

Validation is a reusable service, not handler middleware. Transport parsing occurs at an adapter boundary; structural/profile validation is implemented in `glaux-validation`; pure invariants live with domain types; stateful/current-invariant checks occur in the application/persistence transaction; source trust, policy and command safety remain separate decisions. The same validation contracts serve HTTP, ingestion, synchronization and command paths. The conformance harness uses independent expected results so production validation code is not its own oracle. **[A/P]**

Publication, ingestion and commands are explicit workflows. Domain occurrence, domain event, outbox item, broker message and audit record remain different types. Background workers call application services and durable work ports; an in-memory channel is never truth. Command lifecycle/feasibility, policy/safety decisions, dispatch intent, gateway effect and status reconciliation remain separated by typed ports and transaction/effect ordering. Command dispatch is runtime-disabled without an installed gateway and accepted operational profile. **[A/P]**

Authentication, authorization, policy/releasability, source trust, command safety and audit are distinct. Authentication may begin at the edge, but application use cases enforce object/action/current-state decisions. Query policy compiles into an `AuthorizedQueryPlan` before selection/count/latest/pagination. Representation policy is applied before response/event assembly. Mandatory audit is called by application workflows at the accepted E0–E5 point; logging/tracing never substitutes. **[A/P]**

DDIL is not global runtime state. Ephemeral dependency/connectivity observations, versioned offline authority/policy/trust packages, persisted synchronization watermarks/conflicts and resource freshness evidence remain orthogonal. A pure `ServicePostureEvaluator` derives an operation posture consumed by API, ingestion, publication and commands. Synchronization uses specialized application workflows and ports; it does not absorb network/topology design. **[A/P]**

Dynamic Rust plugins are rejected for the first implementation because Rust lacks a stable ABI and such loading would expand supply-chain, unsafe and security risk. Extension is compile-time through reviewed adapter crates/features or out-of-process through explicit versioned, authenticated protocols. Microservice extraction requires a written replacement for in-process transactions, authorization/policy/audit continuity, idempotency, failure semantics, observability and conformance tests. **[E/P]**

Acceptance of this report establishes internal boundaries and proof gates, not a deployment topology. IDR-SRV-046 must decide whether API and workers run together or as role-specific processes/containers, how PostgreSQL/artifact/broker dependencies are placed, and how availability/edge profiles are packaged. No implementation or IDR-SRV-046 work is authorized by this report alone. **[P]**

---

## 2. Scope and Plan Alignment

### 2.1 Included scope

This report defines architecture style; logical capabilities; initial workspace packages; dependency graph; API/domain/representation boundaries; validation; persistence/query/transaction contracts; ingestion/publication/command workflows; security/policy/audit/DDIL/synchronization placement; configuration/telemetry/test support; extension/extraction rules; repository layout; phased slices; proof gates; and downstream ownership. **[P]**

### 2.2 Excluded scope

It does not decide process/container counts, HA/topology, network zones, cloud/edge layout, proxy/IdP/broker products, secret/telemetry backends, migration operations, implementation scheduling/resources, final public extension APIs, or code. It does not authorize live tasking, inbound Part 3, a dynamic plugin marketplace, or conformance claims. **[P]**

### 2.3 Research-question coverage

| Plan concern | Report location |
|---|---|
| style and first architecture | Sections 5–6 |
| crates, modules and dependency rules | Sections 7–8 |
| internal contracts and extension points | Section 9 |
| API/domain/standards boundaries | Section 10 |
| validation, persistence, ingestion, events and commands | Section 11 |
| security, policy, audit, DDIL and synchronization | Section 12 |
| observability, configuration and testing | Section 13 |
| workspace, phases and proofs | Sections 14–16 |
| recommendations, risks and handoffs | Sections 17–20 |

### 2.4 Accepted baseline reconciliation

The architecture preserves all accepted resource, identity, relationship, time, provenance, status/event, representation, validation, persistence, lifecycle, ingestion, streaming, command, security, policy, audit, DDIL and synchronization distinctions. It applies IDR-SRV-044's modular-monolith direction and selected candidate stack while leaving each platform selection behind a proof gate. No convenience layer may collapse source artifact with domain state, Observation with status/source health, Command with delivery attempt, policy with authentication, audit with telemetry, or broker/database replication with domain synchronization. **[A]**

---

## 3. Evidence Base and Authority Classification

### 3.1 Source inventory

| Source | Status/use | Architectural implication |
|---|---|---|
| OGC Parts 1/2, SensorML, SWE Common | normative standards | external contracts and capability behavior, not internal package prescription |
| accepted IDR-SRV-001–043 | controlling project requirements | semantic boundaries, atomicity, enforcement and evidence obligations |
| accepted IDR-SRV-044 | controlling candidate platform baseline | Rust workspace, Axum/Tokio, SQLx, layered models and proof gates |
| Cargo workspace/resolver docs | official current tool behavior | one lockfile/target/config, shared metadata/lints; resolver 3; feature-unification caveats |
| Axum/Tower docs | official current framework behavior | composable routers/state/services/layers; transport/adaptor mechanisms only |
| Tokio docs | official current runtime behavior | structured/bounded concurrency required by project architecture |
| SQLx 0.9 docs | official current persistence behavior | concrete transaction/lifetime API must be contained in adapter |
| implementation studies 014A–H | pinned non-normative lessons | avoid contract drift, hidden composition and implementation-specific assumptions |

### 3.2 Current Rust architecture constraints

Cargo workspaces share a lockfile, target directory, root profiles/patches and can inherit package metadata/dependencies/lints. A virtual edition-2024 workspace must explicitly select resolver 3. Workspace dependency features remain additive and can unify across members/builds; optional-heavy adapters therefore still require explicit feature/build matrices and may eventually deserve separate release units. **[I/E/P]**

Axum supports composing routers and extracting substates from one application state. This is useful for route ownership but does not justify putting raw global infrastructure into every handler. Tower `Service`/`Layer` provides reusable protocol middleware; policy, audit and transaction semantics remain application behavior because middleware ordering, response short-circuiting and non-HTTP entry paths otherwise create bypass risk. **[I/E/P]**

SQLx 0.9 transactions are concrete lifetime-bound values that should be explicitly committed or rolled back. Current executor APIs and transaction dereferencing make a generic object-safe repository/unit-of-work abstraction easy to overcomplicate. Glaux therefore favors semantic write ports with atomic contracts and keeps transaction-scoped repositories concrete/private inside the PostgreSQL adapter; the exact Rust ergonomics remain a proof gate. **[I/E/P]**

### 3.3 Implementation lessons

The accepted implementation studies demonstrate that reusable models alone do not prevent wrong routes, links, negotiation, filtering, OpenAPI or tasking behavior. Architecture must make a single route/link catalog, representation contract, validation pipeline and executable fixture corpus first-class. It must also tolerate different implementation patterns across partner servers rather than promoting OSH, CS-Go, pygeoapi or SECD internals into requirements. **[A/I/E]**

---

## 4. Architecture Requirement Extraction Methodology

The analysis used six passes:

1. **Trace accepted decisions.** Each prior invariant was mapped to an owner, entry paths, persistence/effect needs, enforcement point and test.
2. **Separate concepts.** Source/wire/domain/storage/projection models and authn/authz/policy/audit/telemetry were kept distinct.
3. **Evaluate styles.** Options were scored for standards correctness, atomicity, testability, security, DDIL, operations, deployment flexibility and contributor cost.
4. **Draw dependency/effect boundaries.** Dependencies point toward policy; effects point through consumed ports to adapters.
5. **Exercise end-to-end workflows.** Read, write, ingestion, publication, command, audit and synchronization paths were checked for bypasses and failure windows.
6. **Assign reversibility.** Hard-to-reverse decisions received early proofs; topology/product/numeric decisions were handed downstream.

### 4.1 Boundary test

A proposed crate, module or interface is justified only when it provides at least one of: enforced dependency direction; independent contract/test ownership; isolation of high-risk dependencies or effects; reusable behavior across entry paths; coherent domain vocabulary/invariants; materially reduced compile/change scope; or a credible future deployment/extraction seam. Naming every folder a service or trait does not satisfy this test. **[E/P]**

### 4.2 Hard-to-reverse decisions

The highest-cost mistakes are public/storage identity leakage, domain dependence on frameworks, handler-owned conformance logic, authorization after query/selection, per-repository transactions, broker-as-truth, command dispatch before durable intent, audit as best-effort logging, global DDIL flags, unstable dynamic plugin ABI and premature distributed ownership. The architecture prevents these before the first slice. **[A/E/P]**

---

## 5. Architecture Style Evaluation

| Style/pattern | Standards/test fit | Operations/DDIL | Change/extraction | Principal risk | Decision |
|---|---|---|---|---|---|
| one crate, ad hoc modules | weak boundary enforcement; fast prototype | simple deploy | hard to control growth | handlers/persistence/common module coupling | Reject as baseline |
| layered monolith only | clear broad direction | simple | horizontal layers can become generic and chatty | domain capability cohesion lost | Use as dependency rule, not whole design |
| modular monolith | strong in-process contracts/transactions/tests | simple and edge-friendly | explicit seams permit later extraction | requires disciplined architecture checks | **Select** |
| Cargo multi-crate workspace | compile-time boundaries/shared lock/lints | one or more binaries possible | isolates dependencies/tests | too many crates/feature unification | **Select, bounded package count** |
| ports and adapters | strong effect substitution/testing | vendor/topology neutral | natural extraction seams | trait explosion/lowest-common-denominator ports | **Select at real effect seams** |
| capability/vertical slices | requirements-to-code/test traceability | coherent operation ownership | incremental delivery | duplication if shared kernel undisciplined | **Select inside layers** |
| service-oriented/microservices | independent scaling/deploy/failure domains | operationally expensive; poor first DDIL fit | maximum deployment flexibility | distributed transactions/policy/audit/drift | Defer until extraction criteria met |
| dynamic in-process plugins | runtime extensibility | difficult assurance/versioning | third-party extension | unstable Rust ABI, unsafe/supply-chain attack surface | Reject initially |
| out-of-process adapters | strong isolation/versioned protocol | extra dependency/failure | good for gateways/brokers/validators | protocol and operations burden | Conditional future seam |

Hexagonal, clean and layered concepts are used selectively. Domain/application policy remains inward; adapters and frameworks remain outward. This is not a rule that every function needs an interface or that domain entities must be persistence ignorant at any cost. Boundaries must preserve accepted semantics and remain understandable to contributors. **[E/P]**

### 5.1 Why not microservices first

Glaux writes frequently couple identity, revision, provenance, policy evidence, mandatory audit, idempotency/inbox and outbox in one transaction. Splitting those owners initially would require distributed consistency, new APIs, independent authorization, retry/idempotency, schema/version rollout, telemetry and conformance work without a demonstrated scaling or team boundary. DDIL and tactical-edge profiles also benefit from fewer mandatory processes. **[A/E/P]**

### 5.2 Extraction criteria

Extract a capability only when all are true:

1. an independent scaling, failure-isolation, security-zone, deployment cadence, hardware/native dependency or ownership need is measured;
2. its authoritative data and transaction boundary are unambiguous;
3. a versioned authenticated protocol replaces calls without semantic loss;
4. retries, idempotency, ordering, policy, audit, DDIL and partial failure are specified;
5. black-box contract/conformance and migration tests exist; and
6. operational cost is accepted in IDR-SRV-046 or later governance.

Potential candidates are heavy schema/codec validation, source ingestion connectors, broker publication, command gateways, audit export and synchronization transfer. Mandatory local audit capture and the authoritative write transaction are not casual extraction candidates. **[P]**

---

## 6. Recommended Initial Architecture Style

### 6.1 Logical view

```text
HTTP / source / replay / admin / worker entry adapters
                 |
         transport normalization
                 |
        application use cases
   (authorize -> validate -> decide -> commit -> effect)
          /          |           \
 domain capabilities |      consumed effect ports
          \          |           /
         standards + validation contracts
                 |
 PostgreSQL/artifact/auth/policy/audit/broker/gateway adapters
```

Entry adapters never jump directly to persistence or external effects. Application services orchestrate one use case and invoke domain rules. Ports are defined by consumers in the application layer and implemented outward. A manual composition root constructs concrete adapters and supplies narrowed service facades to API routers and workers. **[E/P]**

### 6.2 Initial release boundary

The initial architecture produces one server release artifact and one authoritative database schema. It supports logical runtime roles—HTTP admission/query, durable worker/publication, migration/admin tooling—but IDR-SRV-046 decides whether roles run in one process, multiple invocations of the same binary, or separate containers. No module assumes co-location for correctness; no module assumes network separation either. **[P/X]**

### 6.3 Capability areas

- **Catalog:** Systems, Deployments, Procedures, Sampling Features, Properties, relationships, identifiers and lifecycle.
- **Dynamic data:** DataStreams, Observations, status/current/latest projections and ingestion.
- **Control:** ControlStreams, Commands, feasibility, dispatch/status/result/reconciliation.
- **Evidence/governance:** provenance, source trust, security context, policy decisions, audit and validation artifacts.
- **Publication:** domain events, outbox, durable log, subscriptions, replay and protocol adapters.
- **Continuity:** DDIL posture, offline packages, synchronization, gaps, conflicts, quarantine and resolution.
- **Platform/admin:** conformance/OpenAPI, configuration, health, migrations and protected diagnostics.

These are modules/bounded vocabularies, not independently deployable services. Cross-capability operations coordinate in application use cases and one local transaction where accepted atomicity requires it. **[P]**

---

## 7. Module/Crate Boundary Findings

### 7.1 Initial workspace packages

| Package | Owns | Does not own |
|---|---|---|
| `glaux-domain` | value types, entities/evidence, lifecycle/state machines, pure invariants and capability modules | HTTP/Serde public DTOs, SQLx, config, telemetry exporter, broker |
| `glaux-application` | use cases, operation context, decisions, consumed ports, semantic transaction/effect order | Axum extractors, SQL queries, vendor SDKs, public serialization |
| `glaux-standards` | CSAPI/SensorML/SWE/GeoJSON/problem DTOs, media/profile vocabulary, parsers/assemblers and approved artifact bindings | canonical business authority, database rows, policy decisions |
| `glaux-validation` | schema/package registry implementation, structural/profile/semantic validation orchestration and finding artifacts | authorization, persistence commit, command safety decision |
| `glaux-persistence-postgres` | SQLx rows/queries, PostGIS, migrations, atomic write/read port implementations, inbox/outbox/audit storage mechanics | domain policy, public DTO/link rendering, external effects |
| `glaux-api-http` | Axum routers/extractors, negotiation, request context, DTO mapping, response/problem assembly | SQLx pool use in handlers, business decisions, broker/gateway calls |
| `glaux-adapters` | initially small external identity/policy/source/artifact/publication/gateway adapters; split heavy adapters when admitted | domain rules, central service locator, mandatory core semantics |
| `glaux-server` | composition root, runtime roles, supervised workers, startup/shutdown and binary | reusable domain logic, ad hoc handler behavior |
| `glaux-test-support` | builders, fakes, fixture loaders, shared adapter contract suites | production dependency or conformance oracle |
| `xtask` | deterministic generate/check/dev commands | runtime server behavior |

This is an initial target, not a requirement to create empty crates on day one. The first vertical slice may begin with domain, application, standards, persistence, API and server packages; validation/adapters/test-support split when real code and dependency boundaries appear. Architecture checks prevent merging outward dependencies inward during that progression. **[P]**

### 7.2 Internal module guidance

Within domain/application, organize by capability (`catalog`, `dynamic`, `control`, `evidence`, `publication`, `continuity`) rather than technical folders containing unrelated “models,” “services” or “utils.” A small shared kernel may contain stable identifiers, time, digests, revisions, lifecycle and operation evidence. It must not become a circular dumping ground. Capability modules expose intentional public APIs; other items remain `pub(crate)` or private. **[E/P]**

### 7.3 Service architecture matrix

| Module/service area | Responsibility | Inputs/outputs | Depends on | Must not depend on | Contract/interface | Persistence interaction | Security/policy hook | Validation responsibility | Audit/observability | Test strategy | Feature/profile gate | Downstream | Notes/unresolved |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| shared domain kernel | IDs, revisions, clocks, digests, lifecycle primitives | validated values/evidence | std + minimal reviewed crates | Axum, SQLx, config, vendors | constructors/value APIs | none | carries typed references only | pure invariants | no telemetry dependency | unit/property | always | 052/053 | keep deliberately small |
| catalog domain | resources, relationships, identity/lifecycle rules | commands/state/events | shared domain | API/storage DTOs | aggregate/domain services | via application ports | field/action authority inputs | pure/cross-entity rules | domain occurrence returned | model/fixture | always | 045/050 | not one giant aggregate |
| dynamic-data domain | contracts, Observations, status/latest rules | validated facts/projections | shared/catalog types | broker, HTTP, SQL | domain services/selectors | via dynamic ports | source/policy evidence | contract/time/unit rules | occurrence evidence | property/model | profile capability | 053/054 | streaming separate from truth |
| control domain | ControlStream/Command/feasibility/status state | intents/transitions/results | shared/catalog/dynamic references | gateways, Axum, SQLx | state machines/decision types | via command ports | authority/safety required | payload/lifecycle rules | E4 event requirements | model/fault | dispatch disabled default | 055/056 | physical outcome external |
| continuity domain | DDIL assessment, sync outcome/conflict models | evidence -> posture/outcome | shared/evidence types | network topology/broker | pure evaluators/models | via continuity ports | offline authority/policy/trust | envelope/conflict invariants | typed evidence | scenario/property | sync gate | 046/055 | no global mode flag |
| application use cases | order auth/validation/commit/effects | operation inputs/outcomes | domain + consumed ports | Axum, SQLx, vendors | explicit command/query services | semantic atomic write/read ports | mandatory enforcement | invokes facade + state checks | mandates audit/effect order | unit with fakes | capability gates | 045/052 | no generic service locator |
| standards/contracts | public DTOs, media, schemas, codecs, problems | bytes/DTO/domain mappings | domain stable types | SQLx, config, policy engine | parsers/assemblers/catalog | none | representation hooks only | parse/shape helpers | safe finding data | golden/schema | representation/profile | 050/053/056 | exact-source separate |
| validation service | offline schema/profile/semantic pipeline | artifact/context -> findings | standards/domain/app port | HTTP route, DB implementation | `ValidationPort`, registry/resolver | artifact/cache port only | source trust/policy inputs distinct | owns structural orchestration | validation evidence | corpus/fuzz/contract | codec/profile gates | 050/055 | not auth or safety |
| query application | authorized selection/latest/page planning | query + context -> page/view | application/domain | raw HTTP/SQL | typed query services/plans | read projection ports | policy before count/select | query semantic validation | safe metrics/audit class | unit + real DB | always/read profile | 050/054 | no fetch-all/filter |
| PostgreSQL adapter | atomic writes, projections, queries, inbox/outbox/audit | typed ports <-> rows | application/domain + SQLx | Axum/public DTOs | consumed port impls | owns SQL/transactions | executes policy-qualified plans | constraints/state checks | DB telemetry; audit mechanics | real DB/migration/concurrency | always for reference | 049/052/054 | UoW ergonomics proof |
| artifact store adapter | immutable raw/schema/validation artifacts | digest/ref/bytes | application/domain ports | business decisions | `ArtifactStore` | metadata transaction/link | policy/tenant scope | digest/media/size | access audit/metrics | contract/fault | storage profile | 046/049 | local/object store later |
| HTTP API | routes, negotiation, parsing, response/problem/link assembly | HTTP <-> use cases | app/standards/domain | SQLx/broker/gateway | `ApiServices`, route catalog | none directly | authenticate + context; app enforces | bounds/transport parse | request telemetry; app audit | router/TCP/contract | route/capability | 050/056 | State has no raw pool |
| ingestion adapter | protocol receipt/ACK and source envelope mapping | external message -> ingest use case | app/standards | domain DB tables | `IngestSource` adapter | none directly | source authentication context | decode/size then shared validation | transport attempt telemetry | adapter/replay/fuzz | source adapter | 046/056 | server owns admission/effect |
| publication core | domain event/outbox/log/subscription/replay | committed event -> deliveries | application/domain | specific broker SDK in core | publication/subscription services | outbox/log ports | policy filter/reauthorize | event schema/profile | delivery audit/metrics | replay/backpressure | live-stream gate | 048/054 | event != broker message |
| protocol publisher | SSE/MQTT/other delivery mapping | delivery -> protocol | app port/standards/vendor adapter | domain mutation/storage schema | `DeliveryTransport` | delivery attempt port | scoped subscriber context | protocol bounds | delivery telemetry | broker/server contract | adapter feature/config | 046/056 | MQTT/Part3 optional |
| command gateway adapter | target protocol effects/query | dispatch ticket -> acknowledgement/status | app/domain port + SDK | API handler/DB internals | `CommandGateway` | no domain commit directly | ticket/fence/target identity | protocol-specific | attempt/result evidence | simulator/contract/fault | operational grant | 046/055 | out-of-process candidate |
| identity adapter | credential/proxy/mTLS/OIDC verification | request evidence -> identity | application security port | resource query/policy decision | `IdentityVerifier` | key/cache only | establishes identity, not authorization | credential validation | protected auth telemetry | negative/rotation/outage | auth profile | 046/055 | no caller actor fields |
| authorization/policy | action/object/field and releasability decisions | context/resource/evidence -> decision | app/domain ports + adapter | handler-only enforcement | `AuthorizationPort`, `PolicyPort` | policy package/evidence ports | owns decision semantics | input completeness | decision audit refs | decision matrix | safe-dev only explicit | 047/055 | embedded/external adapter |
| audit application/adapter | mandatory typed audit and storage/export | audit intent -> durable receipt | application/domain + persistence | tracing/logging as authority | `AuditWriter`/journal/export | atomic/independent class | separate audit authorization | schema/prohibited data | is authoritative evidence | failure/gap/tamper | always mandatory floor | 048/049/055 | exporter optional |
| DDIL posture | derive per-operation behavior | dependency/data/authority state -> posture | domain/application | global mutable flag | `ServicePostureEvaluator` | reads evidence/state ports | policy/offline authority | evidence completeness | posture transition audit/metrics | scenario/model | deployment profile | 046/048 | runtime observation separate |
| synchronization service | manifests, inbox, classify/apply/conflict | sync envelope -> outcome | app/domain/validation/ports | network topology, DB replication | sync use cases/transfer adapter | atomic candidate/outcome ports | peer/source/policy trust | full shared pipeline | sync audit/events | replay/conflict/fault | sync disabled default | 046/055/056 | transport replaceable |
| configuration/composition | load/validate settings, construct graph/roles | sources -> immutable settings/services | all concrete crates only in server | domain global config access | constructors/factories | migration compatibility check | profile cannot weaken policy | startup validation | redacted fingerprint/health | profile/startup tests | deployment-selected | 046/047 | no DI framework needed |
| observability adapter | structured logs/metrics/traces/health | safe events/instruments -> exporters | tracing/OTel adapters | business decisions or audit authority | typed safe telemetry facade | protected metrics queries only | redaction/visibility | attribute allowlists | owns operational telemetry | capture/export failure | exporter optional | 048 | no raw payloads/IDs labels |
| test/conformance support | fakes/builders/fixtures/black-box harness | scenarios -> evidence | public/internal contracts as appropriate | production dependency | fake/contract suites | disposable real stores | synthetic identities/policies | independent expectations | captured evidence | all layers | dev/CI only | 050-056 | production code not oracle |

---

## 8. Dependency-Direction and Layering Findings

### 8.1 Allowed graph

```text
glaux-domain
   ^       ^
   |       +-- glaux-standards
   |
glaux-application <--- consumed port contracts
   ^       ^       ^
   |       |       +-- glaux-validation
   |       +---------- glaux-persistence-postgres
   +------------------ glaux-api-http / glaux-adapters
                              ^
                              |
                         glaux-server
```

The exact graph may vary to avoid cycles—for example, `glaux-standards` can depend only on a minimal domain kernel and conversion implementations can live in API/application adapters. The invariant is that domain/application never depend on Axum, SQLx, broker/IdP SDKs, OpenTelemetry exporters, filesystem/environment loaders or the binary. **[P]**

### 8.2 Enforced rules

1. Define a port in the crate that consumes the capability, not the crate that implements it.
2. Do not expose vendor/framework types in port signatures or domain errors.
3. API handlers call application services only; workers and ingestion use the same application entry points.
4. Persistence rows and queries remain private to the PostgreSQL adapter.
5. Standards DTOs do not carry internal policy, audit, trust, SQL or topology fields.
6. Mandatory decisions cannot live only in middleware, database triggers, reverse proxies or clients.
7. Cross-capability calls go through explicit application services or typed facts/events, not deep module imports.
8. No general `utils`, `common`, global service locator or mutable singleton.
9. Composition uses constructors and narrow state/facades; test substitution occurs at ports.
10. CI inspects `cargo metadata`/architecture rules and denies forbidden dependencies/cycles/features.

### 8.3 Cross-cutting ownership

| Concern | Central owner | Local responsibility |
|---|---|---|
| content negotiation | HTTP representation layer | capability declares supported profiles/media |
| canonical links | route/link catalog in HTTP standards adapter | domain provides identities/relations, not base URLs |
| validation | validation facade + domain invariants | each use case declares required stages |
| authorization/policy | application decision ports | each use case/query/stream names action/object/fields |
| transactions/idempotency | semantic application contract + Postgres adapter | each command names atomicity and key scope |
| errors/problems | typed domain/application errors + HTTP problem registry | adapters classify causes without leaking internals |
| audit | application audit intent + durable adapter | each workflow declares event/failure class |
| telemetry | observability adapter | modules emit bounded typed instruments only |
| DDIL | posture evaluator | each operation declares dependencies/required assurance |
| synchronization | continuity application service | resource capabilities provide identity/revision/conflict rules |

---

## 9. Internal Contract and Extension Point Findings

### 9.1 Trait decision rubric

Use a trait when the application consumes an external effect/decision, multiple real implementations are expected, a deterministic fake is essential, or a future out-of-process boundary is credible. Keep a concrete type when there is one internal algorithm, no effect boundary, or a trait would merely mirror every method. Prefer capability-oriented interfaces; avoid generic CRUD repositories and trait-per-struct architecture. **[E/P]**

### 9.2 Immediate consumed ports

| Port family | Contract emphasis |
|---|---|
| catalog/dynamic/control/continuity read ports | authorized plan, snapshot/watermark, explicit completeness and domain/query views |
| semantic write ports | atomic commit bundle, expected revision/idempotency, audit/outbox/provenance result |
| artifact store | immutable digest/reference, bounded streaming, tenant/policy/custody |
| validation/schema registry | exact package/version, offline resolution, structured findings/evidence |
| identity/authorization/policy/source trust | immutable inputs, three-state/safe decisions, version/evidence references |
| audit writer | E-class durability semantics, protected typed record, durable receipt/failure |
| delivery transport | logical message/event ID, acknowledgement class, retry outcome; no domain truth |
| command gateway | stable command/attempt/fence, exact target profile, acknowledgement/query/reconcile |
| clock/ID/entropy | explicit time/ID class; deterministic tests without weakening production |

Clock and ID generation can be small traits or injected concrete handles. Configuration is not a broadly queried `ConfigurationProvider`; validated settings are passed to constructors so hidden runtime lookups cannot alter semantics. Observability normally uses typed functions/macros/facades rather than mocking every span. **[P]**

### 9.3 Operation context

A transport-neutral immutable `OperationContext` carries opaque correlation/causation, authenticated subject/client/workload/delegation references, security domain/tenant, source/channel where relevant, request/operation/idempotency identity, received/deadline times, installed profile and safe trace reference. It does not contain caller-authored authority, an unreviewed generic claims map, mutable global mode or a blanket authorization decision. Decision results are explicit typed evidence passed to the relevant commit/effect. **[A/P]**

### 9.4 Extension mechanisms

- **Compile-time adapter crate/feature:** default for MQTT, external identity/policy, artifact store or specialized gateway; reviewed and tested in a feature matrix.
- **Runtime configured implementation:** choose among implementations already compiled and admitted; cannot load arbitrary code.
- **Out-of-process adapter:** for trust-zone/native/runtime isolation after a versioned protocol and extraction decision.
- **Standards/profile extension:** approved namespaced wire fields/media/profile plus validation/OpenAPI/fixture updates.
- **Dynamic library/plugin:** prohibited initially; stable ABI, safety, signature, sandbox, lifecycle and supply-chain problems are unresolved.

Cargo features remove/add code; they do not express tenant policy or authorize a dangerous capability. Runtime capability gates are explicit and deny by default. All supported feature combinations compile/test separately because workspace feature unification can hide accidental coupling. **[I/E/P]**

---

## 10. API/Domain/Standards-Model Boundary Findings

### 10.1 Request path

1. Trusted proxy/TLS normalization and request/correlation limits.
2. Authentication adapter constructs verified identity evidence.
3. Axum extractor parses route/query/header/media/body into a bounded standards DTO.
4. DTO conversion creates an application command/query with explicit representation request.
5. Application use case evaluates authorization/policy/source/DDIL and validation in its required order.
6. Query/write port returns a typed outcome with evidence/watermark.
7. Representation assembler applies authorized field/link/profile rules and canonical route catalog.
8. HTTP adapter maps to status, headers, body or stable RFC 9457 problem.

No handler calls a pool, transaction, broker, policy SDK or command gateway. A route module owns route registration and transport mapping, not business logic. **[A/P]**

### 10.2 Route organization

Compose route modules for service discovery (`landing`, `conformance`, `api-definition`), catalog resources, dynamic data, control/tasking, stream/change feed, ingestion/source profile operations, synchronization/admin, and health/operations. Shared query/media/problem/link infrastructure is narrowly reused. Route visibility and OpenAPI/conformance declarations derive from the same installed capability catalog but are independently verified; disabled routes must not be advertised. **[N/A/P]**

### 10.3 Model separation

Maintain exact source artifacts, standards/profile DTOs, canonical domain types, persistence rows and authorized projections as distinct roles. Conversion functions return structured findings and provenance; they are not infallible `From` implementations when validation can fail. Store unknown extensions only in bounded standards extension containers. Internal lifecycle, policy, trust, audit, synchronization and deployment state never leaks because a domain object was blindly serialized. **[A/P]**

### 10.4 Link and URI construction

Domain relations contain typed target identity/relation semantics. The HTTP standards adapter owns a versioned `RouteCatalog`/`LinkAssembler` that combines authorized relation views with trusted public-base configuration, route/query/profile/media rules and pagination cursors. Handlers do not concatenate URLs. Persistence stores stable identity and external canonical evidence, not deployment-specific self links. **[A/P]**

### 10.5 Problems and conformance

Application errors remain transport neutral. The HTTP problem registry maps typed variants to RFC 9457 status/type/safe extensions after concealment policy. The conformance harness treats the running server and committed artifacts as subjects under test; it must not call production assemblers/validators to generate expected results. Shared fixture parsers are acceptable, but normative expectations remain independently curated. **[A/P]**

---

## 11. Validation/Persistence/Ingestion/Event/Command Boundary Findings

### 11.1 Validation placement

| Stage | Owner | Outcome |
|---|---|---|
| transport bounds/media/decompression | entry adapter | reject before expensive parse |
| syntax/shape/encoding | standards + validation adapter | structured safe findings and exact artifact |
| schema/profile/package | validation service | versioned evidence; offline references |
| pure domain/semantic/unit | domain/validation | validated candidate or findings |
| source trust/policy | distinct application decision ports | accept/reject/quarantine decision evidence |
| state/reference/concurrency | application + atomic persistence port | committed result or typed conflict |
| pre-publication | publication application | authorized representation/event contract |
| pre-command effect | command application | fresh authorization/safety/fence or no dispatch |

The validation facade is reusable from HTTP, ingestion, sync and command application workflows. Async staging is used for large/untrusted imports where the route contract explicitly returns a durable status. Strict online writes validate before canonical commit. Revalidation creates new evidence; it never alters the original artifact/finding. **[A/P]**

### 11.2 Persistence ports and transactions

Avoid `Repository<T>::save`. Define semantic ports such as `CatalogWritePort::commit_create`, `ObservationWritePort::commit_ingest`, `CommandWritePort::commit_admission`, `AuditWritePort`, and `SyncWritePort::commit_candidate_outcome`. Their input includes expected revision/idempotency, validated decisions and evidence; their output includes canonical IDs/revisions, durable result, audit/outbox references and snapshot/watermark. **[P]**

The application owns the semantic sequence and states what must commit together. The PostgreSQL adapter implements the operation with a private concrete transaction and transaction-scoped stores. It explicitly commits/rolls back and never makes external network/physical effects inside the database transaction. For a rare cross-capability operation, use a cohesive atomic write port rather than exposing a generic transaction handle across layers. Prototype this against SQLx 0.9 before freezing signatures. **[A/I/P]**

Read/query ports are projection-specific and accept typed, bounded, policy-qualified plans. They return authorized candidate rows or already policy-safe database projections according to the chosen enforcement strategy, with count/page/latest/watermark semantics. They never return an unbounded table for filtering in handlers. **[A/P]**

### 11.3 Ingestion

Protocol adapters own receive framing, protocol authentication evidence and acknowledgement mapping. They produce a common bounded `IngestEnvelope`; the ingestion application owns source trust, decoding/validation orchestration, idempotency, canonical commit, quarantine, audit and event creation. `glaux-publisher` may prepare/send content but cannot declare server acceptance or authority. Simulator/fake adapters call the same application path under isolated source identities. **[A/P]**

### 11.4 Events and streaming

Domain logic reports modeled occurrences; application workflows decide durable event/audit/outbox creation; persistence atomically commits; workers publish logical deliveries; protocol adapters map to SSE/MQTT/etc. Subscription application services own authorization, policy filters, snapshot/watermark/cursor and reauthorization. A broker SDK never appears in domain/application event types. Streaming-disabled profiles retain domain/outbox truth where required and simply omit/disable live delivery capability honestly. **[A/P]**

### 11.5 Commands

The control application separates admission, validation, feasibility, authorization, safety, durable command/status/audit/outbox commit, dispatch leasing/fencing, gateway effect, status/result reception and reconciliation. The command domain state machine validates transitions but does not perform I/O. A `CommandGateway` port is target-profile specific and receives a single-use ticket; a simulator implements the same contract in a synthetic namespace. Feature/config gates may omit/disable gateway adapters, but the server never advertises or accepts unsupported effect capability. **[A/P]**

---

## 12. Security/Policy/Audit/DDIL/Synchronization Boundary Findings

### 12.1 Security and policy

Authentication is an adapter concern that yields verified identity evidence. Authorization is an application decision over subject, object, action and context. Source trust is a decision over publisher/source/channel/profile. Policy/releasability controls query selection, fields, links, transformations, counts and onward release. Command safety is a specialized pre-effect decision. These share evidence/context types but remain distinct interfaces and audit events. **[A/P]**

Coarse transport middleware can reject obviously unauthenticated or oversized requests. Object/function/property authorization and policy cannot reside only there because ingestion, workers, streams, synchronization and commands enter elsewhere. Application use cases own consistent enforcement. Long-lived subscriptions call a reauthorization service at accepted triggers; they do not retain an eternal middleware decision. **[A/P]**

### 12.2 Audit

Application workflows create typed audit intents with event class, phase/result, actor/delegate/authority, object/action, correlation/causation, decision references and safe details. The audit port implements E0–E5 durability/failure behavior; for atomic classes the PostgreSQL adapter may fulfill domain and audit ports in one private transaction. Independent/pre-effect spooling remains a concrete adapter/runtime responsibility. Telemetry receives only safe summaries and cannot acknowledge audit capture. **[A/P]**

### 12.3 DDIL

Separate:

- ephemeral runtime dependency/connectivity/capacity observations;
- persisted resource freshness/current/last-known evidence;
- installed authority/policy/trust/schema bundle versions and validity;
- persisted work/backlog/audit capacity state;
- synchronization watermarks/gaps/conflicts; and
- a pure derived per-operation posture.

Services request an assessment with their dependency/assurance requirements. They do not branch on `is_offline`. The posture result and evidence reference feed the operation decision, response qualification and audit. Deployment probes supply observations but do not define domain freshness or authority. **[A/P]**

### 12.4 Synchronization

Synchronization entry/transfer adapters exchange versioned manifests/envelopes. The continuity application reuses identity, validation, trust, policy, resource-specific conflict and atomic write ports. Inbox/deduplication, candidate outcomes, conflict/quarantine and watermarks are persisted. Network schedule, peer discovery and database replication remain outside this module. Resource capabilities implement or expose explicit synchronization classification rules; there is no reflective “merge any aggregate” interface. **[A/P]**

### 12.5 Query/policy flow

An authorization/policy decision produces an `AuthorizedQueryPlan` containing allowed resource/relationship scopes, filters/fields/representation rules, policy version and safe completeness behavior. Persistence applies constraints before count/page/latest/extent. A representation policy stage transforms/redacts the returned authorized view and binds provenance. This prevents handlers from fetching hidden rows and then attempting to repair counts/links. **[A/P]**

---

## 13. Observability/Configuration/Test-Support Boundary Findings

### 13.1 Observability

Domain/application code may emit typed safe instrumentation events or use narrow tracing spans, but must not know OTLP/Prometheus/export configuration. The observability adapter defines stable metric names/labels, redaction, sampling and exporters. Correlation enters in `OperationContext`; domain IDs become trace fields only under allowlisted protected rules and never unrestricted metric labels. Health/readiness is an application/runtime assessment over configured capabilities, not a synchronous dependency ping in every request. **[A/P]**

### 13.2 Configuration and profiles

Only the composition root reads files/environment/CLI. It validates one typed immutable settings snapshot, constructs concrete adapters, activates advertised capabilities and records a redacted fingerprint. Domain/application receives typed policy-neutral parameters or capability handles, never global config access. Runtime profiles are documented compositions, not scattered `if profile == ...` branches. Cargo features select compiled adapters; runtime gates enable installed capability; policy/authority grants decide use. **[A/P]**

Always-present core includes domain/application semantics, HTTP discovery/read baseline, validation/security decision hooks, persistence, audit floor and safe errors. Optional/gated capabilities include source adapters, live streaming transports, command gateways, synchronization transfer, external IdP/policy/telemetry/artifact adapters and experimental Part 3. Unsafe-dev/simulator behavior is isolated and cannot be selected by callers. **[A/P]**

### 13.3 Test support

`glaux-test-support` provides deterministic clocks/IDs, builders, synthetic identities/policies, fake effect ports, fixture/artifact loaders and adapter contract suites. Production crates may expose intentional testable APIs but do not depend on test-support. Database/conformance/security/performance tests use real adapters where their behavior matters. Feature combinations and profile manifests are tested as products: route/conformance/OpenAPI declarations must exactly match active capabilities. **[P]**

### 13.4 Architecture enforcement

An `xtask check-architecture` or equivalent CI step consumes `cargo metadata` and a checked-in rule file to detect forbidden dependencies, cycles, feature leakage and production use of test-support. Lints deny first-party unsafe and dependency violations. Code review templates require use-case owner, atomicity, security/policy, validation, audit, telemetry and fixture impact. Architecture tests supplement—not replace—review and simple public APIs. **[E/P]**

---

## 14. Repository/Workspace Layout Findings

### 14.1 Recommended layout

```text
glaux-server/
  Cargo.toml                 # virtual workspace, resolver = "3"
  Cargo.lock
  rust-toolchain.toml
  deny.toml
  crates/
    glaux-domain/
    glaux-application/
    glaux-standards/
    glaux-validation/
    glaux-persistence-postgres/
    glaux-api-http/
    glaux-adapters/
    glaux-server/
    glaux-test-support/
    xtask/
  migrations/
  contracts/
    openapi/
    json-schema/
    profiles/
  standards/                 # pinned approved offline packages/manifests
  test-data/
    fixtures/
    golden/
    corpora/
  tests/
    conformance/
    interoperability/
    security/
    performance/
  docs/
    architecture/
    decisions/
    operations/
```

The actual `glaux-server` repository may adapt names, but must retain ownership distinctions. Migrations live with a clear root/runtime binding and are packaged deterministically. Controlled standards content is not placed in public paths. Large/raw fixtures use a documented storage/LFS/generation policy assigned downstream. **[P/X]**

### 14.2 Workspace rules

- virtual workspace with resolver 3, shared edition/MSRV/license/repository/lints and curated dependencies;
- one committed lockfile and locked CI/release builds;
- explicit default members; CI separately builds workspace/all-targets and each supported feature/profile combination;
- `publish = false` for internal packages unless a later distribution decision says otherwise;
- no cyclic normal/dev dependency tricks that duplicate core types;
- generated OpenAPI/schema/SQLx metadata is deterministic and checked; source inputs and generator version accompany output;
- `xtask` provides cross-platform generate/check/dev workflows instead of opaque shell-only scripts;
- architecture decision records explain admitted dependencies, features, new crates and extraction changes.

### 14.3 Generated artifacts

Commit artifacts that are served, reviewed, needed offline or required for reproducible CI: canonical OpenAPI/profile documents, approved schema manifests/bundles, SQLx metadata if used and golden expectations. Do not commit ephemeral binaries, downloaded arbitrary references, runtime caches or secrets. Generation tests fail on drift and require a clean worktree after regeneration. **[P]**

---

## 15. Phased Implementation Guidance

| Phase | Vertical capability | Required architecture | Deferred/stubbed safely |
|---|---|---|---|
| 0 foundation | workspace/toolchain/CI/contracts | domain/application boundaries, errors/context, architecture checks, PostGIS test env | all business adapters |
| 1 discovery | landing/conformance/API definition | standards/API modules, capability catalog, route/link assembler, problem registry | persistence may use static service metadata |
| 2 catalog read | Systems/related resources/links/query | catalog domain/query app, policy port, Postgres read adapter, representation tests | writes/dynamic/control |
| 3 catalog write | conditional lifecycle/revisions | validation, semantic write port, idempotency/provenance/audit/outbox transaction | external publication worker may be fake |
| 4 dynamic data | DataStreams/Observations/status/latest | SWE contracts, ingestion app, time-series/PostGIS queries, batch behavior | MQTT/Part3 |
| 5 live publication | event log/SSE snapshot-replay | publication/subscription services, supervised worker, cursor/reauth | optional brokers |
| 6 feasibility/control | ControlStreams/Commands/feasibility | control state machines, decision ports, simulator gateway, fenced dispatch design | operational gateway disabled |
| 7 security/policy profiles | operational authz/releasability | real identity/policy adapters, authorized query plans, redaction and tests | enterprise product selection per deployment |
| 8 DDIL/sync | offline packages/posture/reconcile | continuity domain/app, manifests/inbox/conflict/quarantine | topology/peer scheduling |
| 9 readiness | conformance/security/performance/interoperability | full harnesses, migration/restore evidence, packaging/ops docs | no unproven extraction |

Each phase is an end-to-end deployable/testable slice through API/application/domain/persistence/evidence, not completion of one horizontal layer. Stubs are explicit fakes or disabled capabilities; they never return false success, permissive authorization or fabricated audit. **[P]**

### 15.1 Interfaces required early

Define identity/time types, operation context, application outcomes/errors, semantic read/write ports, validation findings, authorization/policy decisions, audit intent/receipt, route/media/profile catalogs and test fixture conventions before wide implementation. Defer broker/gateway/product-specific traits until a real adapter proof prevents hypothetical abstractions from hardening. **[P]**

---

## 16. Proof-of-Concept Needs

1. **Dependency/workspace proof:** create skeletal packages and architecture rule check; prove resolver/features/MSRV/CI combinations without implementation scope expansion.
2. **HTTP vertical proof:** landing, catalog query/item, negotiation, canonical links, problem mapping and curated/generated OpenAPI through `ApiServices`, with no pool in handlers.
3. **Atomic write proof:** SQLx 0.9 semantic write port committing revision, idempotency, provenance, audit and outbox; crash/concurrency/precondition tests and explicit commit/rollback.
4. **Authorized query proof:** compile object/property/relationship policy into SQL before count/page/latest and produce non-leaking links/completeness.
5. **Validation reuse proof:** one exact fixture enters HTTP, ingestion and synchronization; all paths invoke the same offline schema/domain pipeline and yield path-appropriate outcomes.
6. **Durable worker/publication proof:** claim/lease/fence/outbox, publish failure, restart and duplicate; SSE snapshot/replay/slow consumer/reauthorization.
7. **Command boundary proof:** admission transaction, simulator gateway, dispatch ticket/fence, lost acknowledgement, delayed status and unknown outcome; no handler/generic retry bypass.
8. **Audit/DDIL proof:** E-class failure behavior and per-operation posture with dependency/bundle/audit-capacity changes.
9. **Sync proof:** manifest, inbox, exact replay, history/current, gap, conflict/quarantine and resolution evidence through semantic write ports.
10. **Extraction rehearsal:** run one worker/adapter logically in-process and in a separate test process using the same versioned contract; quantify operational/semantic cost before approving service extraction.

Proofs produce source, fixtures, architecture decision records, dependency/feature graphs, threat/failure notes and measured results. They are implementation-readiness tasks owned by later authorized work, not code authorized by this research report. **[P/X]**

---

## 17. Downstream Topic Handoff Matrix

| Downstream topic | Fixed handoff from IDR-SRV-045 | Remaining decision |
|---|---|---|
| IDR-SRV-046 deployment | one modular authoritative core; logical API/worker/admin roles; extraction criteria; optional adapter seams | processes/containers, HA/topology/zones, Postgres/artifact/broker placement |
| IDR-SRV-047 configuration | composition-root-only loading, immutable typed settings, profile/capability/feature separation | precedence, secret providers, reload, profile manifests |
| IDR-SRV-048 observability | telemetry adapter separate from audit; typed safe instruments; capability health | backend/exporters, metrics/endpoints, SLOs/alerts and retention |
| IDR-SRV-049 migration/restore | private Postgres schema/transactions; migration directory; identities/epochs/inboxes/outbox/audit preserved | expand/contract choreography, backup/restore, RPO/RTO/rollback |
| IDR-SRV-050 conformance | standards/route/capability catalogs and independent black-box expectations | harness packaging, requirement mapping and evidence reports |
| IDR-SRV-051 workflow | workspace/xtask/architecture checks, generated-artifact cleanliness and feature matrix | CI provider/caches/dev environment/release flow |
| IDR-SRV-052 TDD | ports/fakes/adapter contract suites, vertical slices, no production-as-oracle | exact test sequence, doubles, coverage/flake ownership |
| IDR-SRV-053 fixtures | source/wire/domain/storage/projection role distinction and shared loader boundary | corpus layout/generators/LFS/controlled-data rules |
| IDR-SRV-054 performance | API/app/DB/worker/stream boundaries and measured extraction criteria | load models, budgets, hardware and regression thresholds |
| IDR-SRV-055 security | enforcement-point architecture, parser/port/plugin boundaries, command/audit/sync failure paths | security harness/tools, campaigns and assurance evidence |
| IDR-SRV-056 interoperability | stable external contracts independent of internals, versioned adapters/profiles | server/client/partner matrix and environments |
| IDR-SRV-057 synthesis | selected architecture, proofs, risks and extraction rules | final implementation roadmap and acceptance |

IDR-SRV-046 and all later topics remain unauthorized pending acceptance of this report. **[P]**

---

## 18. Recommendations

1. Adopt the Cargo-workspace modular monolith and one authoritative write core for first implementation.
2. Combine inward dependency layering, ports/adapters at true effects, and capability vertical slices; do not enforce an architectural slogan mechanically.
3. Start with the bounded package set in Section 7 and create crates only when real code/dependencies justify the boundary.
4. Keep domain/application independent of Axum, SQLx, broker, IdP, telemetry exporter and global configuration.
5. Expose `ApiServices`, never raw pools/SDKs, to handlers; route every mutation and worker through application use cases.
6. Use semantic atomic write and authorized query ports rather than generic repositories or leaked SQLx transactions.
7. Centralize reusable validation contracts while keeping source trust, policy, command safety and stateful concurrency as distinct decisions.
8. Keep event/outbox/delivery, command intent/dispatch/effect, audit/telemetry and DDIL runtime/persisted evidence separate.
9. Enforce dependency/feature/profile rules in CI with `cargo metadata`, lints and architecture tests.
10. Reject dynamic plugins and defer microservices until all six extraction criteria are met.
11. Deliver functionality as end-to-end tested slices in the phase order, with no permissive stubs or false success.
12. Execute the ten proof gates before freezing difficult Rust transaction, validation, policy, worker, command or extraction interfaces.

---

## 19. Risks, Constraints, and Open Questions

### 19.1 Risks and controls

| Risk | Control |
|---|---|
| too many crates slow changes/compile/onboarding | boundary test, bounded initial set, module-first extraction |
| too few boundaries create handler/database coupling | dependency rules, narrow `ApiServices`, architecture CI |
| trait explosion and mock-driven design | traits only at consumed effect/volatility seams; concrete core |
| generic repository hides accepted SQL/transactions | semantic ports and private Postgres UoW |
| duplicated validation across entry paths | shared validation facade with path-specific orchestration |
| middleware becomes only enforcement | application authz/policy/audit for every entry path |
| feature gates create untested insecure products | explicit supported matrix, runtime deny defaults, capability catalog |
| Cargo feature unification pulls optional risk into builds | adapter package/feature audit and separate build invocations |
| microservice extraction breaks atomicity | six extraction criteria and contract/failure/migration tests |
| test helpers become production backdoors | one-way dev dependency and simulator isolation |
| telemetry leaks or substitutes for audit | typed allowlisted instruments and separate audit port |
| global DDIL flag collapses semantics | orthogonal evidence plus per-operation posture evaluator |

### 19.2 Open questions and owners

| Question | Owner |
|---|---|
| exact package split after first skeleton/vertical proof | IDR-SRV-045 proof + implementation governance |
| SQLx 0.9 semantic write/UoW signature and lifetime ergonomics | atomic-write proof |
| whether `glaux-standards` depends on domain kernel or conversions live outward | dependency proof |
| validation package split and XML engine placement | validation proof + IDR-SRV-050/055 |
| API/worker/admin process packaging | IDR-SRV-046 |
| artifact store local versus object implementation | IDR-SRV-046/049 |
| exact identity/policy adapter composition | IDR-SRV-046/047/055 |
| feature/profile/capability manifest schema | IDR-SRV-047 |
| audit journal co-location/extraction limits | IDR-SRV-046/048/049 |
| first acceptable service extraction | measured proof plus IDR-SRV-046 decision |

---

## 20. Validation Against This Plan's Success Criteria

| Success criterion | Evidence | Result |
|---|---|---|
| styles evaluated with sources and requirement traceability | Sections 3–5 | Met |
| first architecture documented with rationale | Section 6 | Met |
| module/crate boundaries, dependencies and contracts documented | Sections 7–9 and required matrix | Met |
| all required functional/cross-cutting boundaries addressed | Sections 10–13 | Met |
| validation, errors, negotiation, links, security, policy, audit, observability, transaction, idempotency, DDIL and sync flows documented | Sections 8–13 | Met |
| repository layout and phased guidance documented | Sections 14–16 | Met |
| implementation/community lessons incorporated non-normatively | Section 3.3 and boundary/test decisions | Met |
| recommendations decision-usable and server-bounded | Sections 18–19 | Met |
| downstream handoffs explicit | Section 17 | Met |
| references explicit and reproducible | Section 21 | Met |

The report defines internal architecture without choosing deployment topology or creating code. It is accepted as the planning baseline for IDR-SRV-046 and later implementation-platform research. **[P]**

---

## 21. References

### 21.1 Rust and platform architecture sources

1. Rust Project, [Cargo Workspaces](https://doc.rust-lang.org/cargo/reference/workspaces.html), including virtual workspaces, shared lockfile/target/configuration and workspace metadata/lints, checked 2026-09-15.
2. Rust Project, [Cargo Dependency Resolution](https://doc.rust-lang.org/cargo/reference/resolver.html), including resolver 3, feature unification, lockfiles and cycle guidance, checked 2026-09-15.
3. Rust Project, [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/) and [Cargo Reference](https://doc.rust-lang.org/cargo/reference/), checked 2026-09-15.
4. Tokio Project, [Axum 0.8.9 Router](https://docs.rs/axum/0.8.9/axum/struct.Router.html) and [State](https://docs.rs/axum/0.8.9/axum/extract/struct.State.html).
5. Tokio Project, [Tower 0.5.3](https://docs.rs/tower/0.5.3/tower/), including `Service`, `Layer`, middleware and test integration.
6. Tokio Project, [Tokio 1.53.1](https://docs.rs/tokio/1.53.1/tokio/), runtime/task/concurrency documentation.
7. SQLx Project, [SQLx 0.9.0 Transaction](https://docs.rs/sqlx/0.9.0/sqlx/struct.Transaction.html) and [Executor](https://docs.rs/sqlx/0.9.0/sqlx/trait.Executor.html), checked 2026-09-15.
8. Serde Project, [Serde](https://serde.rs/); Tokio Project, [tracing](https://docs.rs/tracing/0.1.44/tracing/); OpenTelemetry Rust [0.32.0](https://docs.rs/opentelemetry/0.32.0/opentelemetry/).

### 21.2 Standards and protocol sources

9. OGC, [OGC API - Connected Systems - Part 1](https://docs.ogc.org/is/23-001/23-001.html), Version 1.0.
10. OGC, [OGC API - Connected Systems - Part 2](https://docs.ogc.org/is/23-002/23-002.html), Version 1.0.
11. OGC, [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) and [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html).
12. IETF, [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110) and [RFC 9457: Problem Details](https://www.rfc-editor.org/rfc/rfc9457).
13. OpenAPI Initiative, [OpenAPI Specification](https://spec.openapis.org/oas/latest.html); JSON Schema, [Specification](https://json-schema.org/specification).
14. PostgreSQL Global Development Group, [PostgreSQL 18 Documentation](https://www.postgresql.org/docs/current/); PostGIS, [Documentation](https://postgis.net/documentation/).
15. OASIS, [MQTT 5.0](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html); CNCF, [CloudEvents](https://github.com/cloudevents/spec).

### 21.3 Accepted project evidence

16. [IDR-SRV-023 Schema and Encoding Validation Strategy](idr-srv-023-schema-and-encoding-validation-strategy-report.md).
17. [IDR-SRV-025 Database and Persistence Architecture Options](idr-srv-025-database-and-persistence-architecture-options-report.md).
18. [IDR-SRV-029 Transaction, Consistency, Idempotency, and Concurrency Strategy](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md).
19. [IDR-SRV-031 Server Write and Ingestion Model](idr-srv-031-server-write-and-ingestion-model-report.md).
20. [IDR-SRV-035 Streaming and Event Publication Strategy](idr-srv-035-streaming-and-event-publication-strategy-report.md).
21. [IDR-SRV-038 Command Authorization, Safety, and Audit Strategy](idr-srv-038-command-authorization-safety-and-audit-strategy-report.md).
22. [IDR-SRV-039 Authentication, Authorization, and API Security Threat Model](idr-srv-039-authentication-authorization-and-api-security-threat-model-report.md).
23. [IDR-SRV-040 Policy, Releasability, and Cross-Boundary Access Constraints](idr-srv-040-policy-releasability-and-cross-boundary-access-constraints-report.md).
24. [IDR-SRV-041 Audit Logging and Accountability Strategy](idr-srv-041-audit-logging-and-accountability-strategy-report.md).
25. [IDR-SRV-042 DDIL-Informed Server Semantics](idr-srv-042-ddil-informed-server-semantics-report.md).
26. [IDR-SRV-043 Server Synchronization and Conflict Handling Boundary](idr-srv-043-server-synchronization-and-conflict-handling-boundary-report.md).
27. [IDR-SRV-044 Rust Implementation Language and Framework Strategy](idr-srv-044-rust-implementation-language-and-framework-strategy-report.md).
28. Accepted [IDR-SRV-014A](idr-srv-014a-osh-csapi-server-implementation-study-report.md) through [IDR-SRV-014H](idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md) implementation and interoperability studies.

### 21.4 Reproducibility and evidence limits

- Current Rust/Cargo/Axum/Tower/Tokio/SQLx documentation was checked on 2026-09-15 at the versions named above.
- Package names and the dependency graph are planning decisions subject to the Section 16 skeleton/vertical proofs; they are not claims that a repository already exists.
- No performance, availability, extraction-cost or conformance result is claimed without implementation evidence.
- The implementation studies inform risks/tests but do not prescribe Glaux internals.
- The shared upstream-history register remains Version 1.12; this internal-architecture topic found no material published CSAPI history change.

---

## Report Completion Checklist

- [x] All 21 required report sections are present.
- [x] Architecture options, selected style and extraction criteria are explicit.
- [x] The service architecture matrix contains every required field.
- [x] Crate/module boundaries and acyclic dependency rules are defined.
- [x] API, representation, validation, persistence, ingestion, events, commands, security, policy, audit, DDIL, synchronization, configuration, observability and tests are placed.
- [x] Repository layout, phased slices and ten proof needs are documented.
- [x] Implementation lessons and all downstream handoffs are explicit.
- [x] IDR-SRV-046, deployment topology and implementation remain unauthorized pending acceptance.

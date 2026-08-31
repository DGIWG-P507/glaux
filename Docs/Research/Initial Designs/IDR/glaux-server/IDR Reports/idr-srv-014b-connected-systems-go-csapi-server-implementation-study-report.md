# Section 014B: Connected Systems Go CSAPI Server Implementation Study - Research Report

**Topic ID:** IDR-SRV-014B<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-014B Connected Systems Go CSAPI Server Implementation Study](../IDR%20Plans/idr-srv-014b-connected-systems-go-csapi-server-implementation-study.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 42 detailed questions; all six methodology phases, ten success criteria, nineteen required content areas, and twelve minimum implementation-findings fields are validated<br>
**Methodology Used:** Release- and commit-pinned source, route, model, persistence, formatter, documentation, fixture, test, issue, pull-request, and release review; bounded public-endpoint probes; mechanical source inventories; comparison with accepted IDR-SRV-006 through IDR-SRV-014A; and synthesis using the OS4CSAPI research corpus<br>
**Research Time:** Approximately 4.5 hours of AI-assisted elapsed execution on August 31, 2026<br>
**Primary Sources:** Connected Systems Go upstream release `v1.0.4` at commit `244f4dd586da685d4d9b75e43f73001028b5bd0e`; OS4CSAPI audit fork commit `e900da88738cca92872038b703c4ad537fc0c8fd`; approved OGC API - Connected Systems Parts 1 and 2; and accepted Glaux IDR-SRV-006 through IDR-SRV-014A reports<br>
**Supporting Resources:** Upstream issues, pull requests, releases, and repository metadata current through August 31, 2026; historical comparison fork commit `6665c737f1857b24741237a37779aee03cf3d694`; and the OS4CSAPI client/audit corpus<br>
**Document Purpose:** Establish a reproducible Connected Systems Go implementation findings baseline for Glaux architecture, behavior, validation, interoperability, and test design without treating the implementation or its conformance declarations as standards authority<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** August 31, 2026<br>
**Date:** August 31, 2026<br>
**Last Updated:** August 31, 2026

---

## Reading Guide

This report uses the following evidence labels:

- **N — normative:** approved CSAPI or incorporated-standard evidence;
- **P — project baseline:** an accepted prior Glaux IDR conclusion;
- **C — code-supported:** behavior established in pinned CS-GO source;
- **T — test-supported:** behavior represented in pinned automated test source;
- **D — documented:** repository documentation, examples, configuration, or generated documentation;
- **A — API-observed:** response observed in a dated public-endpoint probe;
- **H — history:** issue, pull request, commit, release, or maintainer disposition; and
- **I — inference:** bounded analysis that is not itself a directly observed runtime fact.

“Implemented” means relevant code exists at the studied pin. It does not establish full satisfaction of every requirement in a conformance class. “Claimed” means the server returns a class URI. Tests named or commented as conformance tests are valuable project tests, not evidence of an official OGC Abstract Test Suite result. CS-GO is informative implementation evidence; approved standards and accepted Glaux reports control when evidence differs.

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. CS-GO Source Inventory and Evidence Classification
4. CS-GO Architecture and Module-Boundary Findings
5. CS-GO CSAPI Part 1 Behavior Findings
6. CS-GO CSAPI Part 2 Behavior Findings
7. CS-GO Conformance Posture and Standards-Alignment Matrix
8. CS-GO API Behavior Findings
9. CS-GO Dynamic-Data, Status, and Tasking Findings
10. CS-GO Persistence, Data Model, and Fixture Implications
11. CS-GO Documentation, OpenAPI, and Examples Findings
12. Interoperability and Client-Compatibility Findings
13. Test-Strategy Implications
14. Lessons for Glaux Server
15. Downstream Topic Handoff Matrix
16. Recommendations
17. Risks, Constraints, and Open Questions
18. Validation Against the Research Plan
19. References
Appendix A. Detailed Question Ledger
Appendix B. Reproducible Study Record
Appendix C. Completion and Handoff

---

## 1. Executive Summary

Connected Systems Go (CS-GO) is a focused, rapidly evolving Go implementation of a broad Connected Systems API server. At upstream release `v1.0.4`, it exposes 93 registered HTTP operations across systems, deployments, procedures, sampling features, properties, generic features, datastreams, observations, control streams, commands, system events, system history, collections, conformance, OpenAPI, and AsyncAPI. Its typed repositories, PostGIS geometry, JSONB SensorML/SWE components, deterministic keyset pagination, strict JSON decoding, cascade handling, configurable MQTT ingestion/publication, resource-class route composition, and 384 named tests provide substantial practical evidence for Glaux.

It is not a safe standards or conformance oracle. The `/conformance` response is a fixed list of 24 URIs and does not reflect enabled transports or resource capabilities. It claims nonexistent direct Part 2 `observation` and `command` classes, an unapproved `system-history` class, and Part 1 `advanced-filtering` while omitting other approved classes that code partly supports. Its public query vocabulary diverges in material places: commands use `currentStatus` instead of accepted `statusCode`, control streams use `commandFormat` instead of the accepted query name `cmdFormat`, and `latest` is accepted outside the Glaux baseline's Observation `resultTime` use. Public cursor tokens are readable base64 JSON with a scope hash but no authenticity, confidentiality, principal, or snapshot binding.

The implementation's strongest lesson is its audit-to-fix feedback loop. The OS4CSAPI fork froze an older runtime baseline and added 27 issue evaluations plus 13 prepared upstream reports. Eleven upstream issues were filed; all were closed after fixes, clarification, or rejection. The current upstream release has corrected important `/api`, SQL, serialization, cascade, time-range, validation, link, and strict-decoding defects. Several proposed findings were correctly rejected after standards or maintainer review. Glaux should preserve both sides of this pattern: evidence-rich negative testing and authoritative adjudication with explicit invalid dispositions.

Important current risks remain:

- no REST authentication or authorization implementation was found; CORS allows all origins and app-layer TLS is absent;
- runtime startup performs GORM `AutoMigrate`, while a versioned migration command is still unimplemented;
- unsupported or compound media types silently fall back to a default formatter rather than producing deterministic 406/415 behavior;
- errors are ad hoc `{"error": ...}` payloads rather than the accepted Glaux problem model;
- the generated OpenAPI 3.0 document covers 42 paths and 82 operations against 93 registered routes, has eight shallow schemas, no operation IDs, and no runtime parity check;
- no formal OGC ATS run, configured CI workflow, authorization, performance, rate-limit, or multi-principal tests were found;
- the documented public endpoint returned HTTP 404 for all bounded probes; and
- no repository license file exists. An `IMPLEMENTATION.md` prose line saying “License MIT” is not sufficient reuse permission.

Glaux should adopt patterns, not copy the design wholesale. Recommended patterns are a typed capability registry, explicit domain/repository/transport separation, PostGIS-native spatial indexing, strict deserialization, deterministic keyset pagination, transaction-backed cascades, capability-derived AsyncAPI, negative/golden test fixtures, and issue-to-regression traceability. Glaux should improve these with versioned migrations, authenticated opaque cursors, canonical query names, strict media negotiation, a generated and parity-tested deployment OpenAPI document, capability-derived conformance, durable audit-safe history, and security integrated before transactional or command routes are enabled.

Acceptance of this report would make IDR-SRV-014C the next authorized implementation-study topic. It would not certify CS-GO, authorize direct source reuse, select Glaux's Rust framework or database, or make CS-GO-specific extensions part of the Glaux contract.

## 2. Scope and Plan Alignment

### 2.1 Included and Excluded Scope

The study reviewed the current upstream release, the OS4CSAPI audit fork, the historical mvanhorn fork, source structure, routes, handlers, formatters, repositories, migrations, configuration, examples, E2E schemas, tests, generated API descriptions, releases, issues, pull requests, and bounded public endpoint behavior. Findings were compared to accepted IDR-SRV-006 through IDR-SRV-014A.

The study does not certify official OGC conformance, execute performance or security testing, inspect a private deployment, resolve license ownership, or decide Glaux's implementation stack. Local compilation and runtime tests were unavailable because the research environment had neither Go nor Docker. Test-source inspection is therefore not represented as a successful local test run.

### 2.2 Plan Coverage Matrix

| Plan question group | Coverage | Evidence location |
|---|---|---|
| Source, version, forks, artifacts, and licensing | Complete | Sections 3 and 17; Appendices A–B |
| Architecture, modules, resources, and configuration | Complete | Section 4 |
| Part 1 and Part 2 behavior | Complete | Sections 5–7 |
| Routes, links, queries, pagination, representations, errors, and API docs | Complete | Sections 8 and 11 |
| Dynamic data, status, events, tasking, feasibility, and transports | Complete | Sections 6 and 9 |
| Persistence, models, migrations, fixtures, and examples | Complete | Section 10 |
| Client and interoperability evidence | Complete within available evidence | Section 12 |
| Validation and test lessons | Complete | Section 13 |
| Glaux applicability, recommendations, risks, and handoffs | Complete | Sections 14–17 |

## 3. CS-GO Source Inventory and Evidence Classification

### 3.1 Reproducible Source Baseline

| Source | Role | Pin / state | Evidence | Limitation |
|---|---|---|---|---|
| [Upstream CS-GO](https://github.com/SomethingCreativeStudios/connected-systems-go/tree/244f4dd586da685d4d9b75e43f73001028b5bd0e) | Current server runtime baseline | Annotated release tag `v1.0.4`; commit `244f4dd586da685d4d9b75e43f73001028b5bd0e`; tree `f85a031534040582ed434142192e57bb34ff0b62`; tagged 2026-08-23 | C/T/D/H | No configured CI; local execution unavailable. Main differs only in `dist/checksums.txt`. |
| [Upstream main](https://github.com/SomethingCreativeStudios/connected-systems-go/tree/4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952) | Post-release comparison | Commit `4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952`; 2026-08-23 | H/D | One checksum-file commit after the release; not substituted for the release pin. |
| [OS4CSAPI fork](https://github.com/OS4CSAPI/connected-systems-go/tree/e900da88738cca92872038b703c4ad537fc0c8fd) | Audit, issue-evaluation, and interoperability evidence | Commit `e900da88738cca92872038b703c4ad537fc0c8fd`; 2026-05-05 | H/T/D | Runtime code split from older upstream commit `df6da0d`; 86 fork-only research/audit commits and 20 newer upstream commits make it unsuitable as current runtime baseline. |
| [mvanhorn fork](https://github.com/mvanhorn/connected-systems-go/tree/6665c737f1857b24741237a37779aee03cf3d694) | Provenance comparison | Commit `6665c737f1857b24741237a37779aee03cf3d694`; 2026-04-30 | H | OS4CSAPI is 62 commits ahead and zero behind this fork; historical only. |
| [Upstream issues](https://github.com/SomethingCreativeStudios/connected-systems-go/issues) and [pull requests](https://github.com/SomethingCreativeStudios/connected-systems-go/pulls?q=is%3Apr) | Defect disposition and fix history | State retrieved 2026-08-31 | H | Mutable; current code and tests control current-behavior statements. |
| [Release `v1.0.4`](https://github.com/SomethingCreativeStudios/connected-systems-go/releases/tag/v1.0.4) | Binary distribution | Five platform binaries plus checksum asset; published 2026-08-23 | H/D | Build provenance is not exposed by a workflow; checked-in tag checksum file uses stale `v1.0.3` filenames. |
| `https://cs-viewer.monogatari.dev/` | Repository-referenced deployment | Probed 2026-08-31 | A | Root and candidate API paths returned 404; no positive live behavior established. |
| Approved CSAPI Parts 1 and 2 and accepted IDR-SRV-006–014A | Controlling comparison | Versions and decisions recorded in accepted reports | N/P | Control over implementation precedent. |

### 3.2 Artifact and Evidence Separation

The release contains 211 Go files, including 69 `_test.go` files and 384 named `Test...` functions. Production code under `cmd/` and `internal/` is separated from E2E tests and bundled schema fixtures under `e2e/`, example payloads under `examples/`, generated descriptions under `docs/`, and distribution checksums under `dist/`. Generated OpenAPI files reflect annotations and generation history, not independently verified runtime parity.

The OS4CSAPI fork contains 27 evaluation documents (26 numbered evaluations plus a separate SensorML loss analysis) and 13 prepared upstream-report documents. These are audit evidence, not current code behavior. The fork's finding about silent SensorML loss in an OSHConnect-Python publisher path was correctly scoped as a client payload/decoder mismatch rather than a new upstream server regression.

### 3.3 Licensing and Evidence Quality

Neither the upstream release tree nor the two inspected forks contains a `LICENSE`, `COPYING`, or `NOTICE` file, and GitHub reports no detected license. `IMPLEMENTATION.md` says “License MIT,” but a prose assertion without license text, copyright grant, or owner clarification is not an adequate direct-reuse basis. Glaux may independently learn from public behavior and architecture; direct copying must wait for explicit provenance and license review.

Code plus a targeted test is strong implementation evidence. Code without execution is moderate evidence. Comments naming conformance classes are development intent. A returned conformance URI is a claim. Issue reports describe historical observations until current source confirms persistence. Absence findings are limited to the pinned tree and scoped searches.

## 4. CS-GO Architecture and Module-Boundary Findings

### 4.1 Application Shape

CS-GO is a standalone Go HTTP service. `cmd/server/main.go` loads Viper configuration, creates a Zap logger, connects to PostgreSQL/PostGIS through GORM, runs startup migrations, constructs typed repositories, optionally starts MQTT ingestion/publication, builds a chi router, starts an HTTP server with timeouts, and performs graceful shutdown.

The code separates:

- HTTP routes and handlers in `internal/api`;
- domain models and wire formatters in `internal/model`;
- query parsing in `internal/model/query_params`;
- typed data access in `internal/repository`;
- resource validation in `internal/resourcevalidation`;
- MQTT transport and ingestion in `internal/mqtt`;
- publication and resource events in `internal/pubsub`;
- configuration in `internal/config`; and
- executable assembly in `cmd/server`.

This separation is a strong, portable design idea. Glaux should implement equivalent Rust boundaries through traits and typed registries, not translate Go generics, GORM patterns, or chi handlers literally.

### 4.2 Domain and Repository Boundaries

Typed repositories exist for systems, deployments, procedures, sampling features, properties, generic features, datastreams, observations, collections, control streams, commands, system events, and history. Domain models carry resource fields, associations, link material, geometry, time ranges, SensorML/SWE structures, and persistence metadata.

Some multi-resource create, update, delete, and cascade paths use GORM transactions. PostGIS geometry, GiST indexes, JSONB values, explicit cursor indexes, deployment closure triggers, and foreign-key handling show practical database awareness. This is much stronger than a fixture-only prototype.

The boundary remains coupled to GORM semantics and startup migration behavior. Repository interfaces do not by themselves provide versioned schema evolution, multi-tenant policy, resource ownership, or audit immutability. Those concerns must be first-class in Glaux.

### 4.3 Capability Composition and Configuration

REST routes are broadly compiled in. MQTT is controlled by a master switch with observation, command-status, command, and system-event class flags. AsyncAPI generation filters servers and channels using configuration, demonstrating a valuable capability-derived documentation pattern.

REST conformance and OpenAPI are not built from the same capability source. Glaux should use one typed capability registry to drive routes, conformance, OpenAPI/AsyncAPI, discovery links, authorization scopes, and required tests. Static independent declarations invite drift.

### 4.4 Security Boundary

No REST authentication or authorization middleware was found. CORS permits all origins with credentials disabled; `Authorization` appears in allowed headers but is not consumed. App-layer TLS is absent; deployment material assumes a reverse proxy. Database and MQTT example configuration includes default credentials, and MQTT TLS is not a secure default.

This means transactional and command behavior is architecturally broad but not production-safe as shipped. Glaux must put identity, policy, tenancy/ownership, transport protection, audit, and command authorization ahead of enabling write/tasking routes.

## 5. CS-GO CSAPI Part 1 Behavior Findings

### 5.1 Resource Coverage

Routes and handlers cover systems, nested subsystems, deployments, nested subdeployments, procedures, sampling features nested under systems, properties, generic features, and collection/item access. GeoJSON and SensorML JSON formatters cover core Part 1 families. CRUD uses GET, POST, PUT, and DELETE; no PATCH route was found.

The breadth is useful, but route presence does not prove every approved requirement. The fixed conformance payload omits approved Part 1 `subdeployment`, create-replace-delete, update, and SensorML classes even where code contains related behavior. This is declaration drift in both directions: implemented behavior may be undeclared, while claimed advanced filtering is incomplete.

### 5.2 Relationships and Links

Nested routes, association-link helpers, absolute-link fixes, and E2E relationship tests are strengths. Upstream PRs corrected incomplete absolute links and enriched link type/title/UID fields. These fixes validate Glaux's accepted requirement that links are contract-bearing data, not decorative output.

Glaux should centralize canonical URI and link construction, then test links for absolute/relative policy, media type, title, UID, traversal, reverse relation, and behavior behind a proxy/base path. CS-GO's history shows distributed link construction produces systematic omissions.

### 5.3 Validation and Mutations

The current release uses strict JSON decoding and sanitizes decoder errors. Earlier audit findings found silently dropped fields and orphaned validators; upstream fixes added strictness and validation. Cascade deletion was also corrected to include system events and avoid dangling resources.

These are reusable lessons. Successful mutation tests must assert round-trip field preservation, associations, cascade behavior, conflict status, and restart persistence. Strict decoding must be schema-version aware: rejecting unknown fields is useful only when the server accurately implements the accepted schema.

### 5.4 Query Behavior

The query layer implements resource identifiers, temporal and spatial constraints, recursive relations, type/property filters, public cursor paging, and resource-specific filters. It deliberately rejects offset paging. Default limit 10 matches the accepted baseline.

Material gaps remain:

- unknown query parameters are ignored rather than rejected or consistently documented;
- invalid `limit` input falls back silently, and zero/negative values can remove the SQL limit;
- no maximum/clamp was found;
- `latest` is accepted for multiple resource/time contexts rather than the accepted Observation `resultTime` use;
- canonical names differ for some Part 2 filters; and
- `select` and general sorting are absent, appropriately avoiding unsupported claims but requiring clear documentation.

## 6. CS-GO CSAPI Part 2 Behavior Findings

### 6.1 Datastreams and Observations

CS-GO implements persistent datastreams, schemas, observations, nested observation routes, time-range maintenance, filtering, CRUD, and JSON formatting. Tests cover canonical routes, required fields, schema validation, spatial behavior, derived stream time ranges, and cursor paging. Observation storage in PostgreSQL supplies historical queries.

No HTTP WebSocket, Server-Sent Events, or replay protocol was found. Real-time transport uses optional MQTT. Glaux should keep historical REST retrieval, live transport, and replay/resumption as separate declared capabilities.

### 6.2 Control Streams and Commands

Persistent control streams, command schemas, commands, command status, and command results are implemented with nested routes and tests. This supplies practical evidence for separate task definition, submission, state, and result resources.

Public vocabulary diverges from the accepted Glaux baseline: the control-stream query uses `commandFormat` rather than `cmdFormat`, and command filtering uses `currentStatus` rather than `statusCode`. The schema member may legitimately be named `commandFormat`; this finding concerns the query parameter contract.

No feasibility endpoint or lifecycle was found. Glaux must not infer feasibility support from command support.

### 6.3 System Events and History

CS-GO implements persistent root and nested system-event CRUD. This is stronger evidence for durable event resources than implementations that expose only transient notifications. The release also implements system history, including mutation/deletion behavior, although the accepted Part 2 baseline removed the direct `system-history` conformance class.

Glaux should treat system history as an explicitly governed extension, if retained, and should protect historical revisions from ordinary mutation or deletion. An audit/history mechanism that can be rewritten without privileged policy weakens integrity.

### 6.4 Encodings

Part 2 JSON formatters cover datastreams, observations, control streams, commands, and status. SWE text and binary observation/result encodings were not found. The conformance payload therefore cannot be read as complete Part 2 encoding support.

## 7. CS-GO Conformance Posture and Standards-Alignment Matrix

### 7.1 Declaration Assessment

`ConformanceHandler` returns 24 fixed URIs: four Common, two Features, nine Part 1, and nine Part 2. The list is not derived from enabled features or verified test results.

Against accepted IDR-SRV-008, notable issues are:

- direct Part 2 `observation` and `command` class URIs do not exist; those behaviors are subordinate to datastream/control-stream classes;
- `system-history` is not an approved direct Part 2 class;
- Part 1 omits approved subdeployment, create-replace-delete, update, and SensorML classes;
- Part 2 omits feasibility, advanced-filtering, update, and SWE text/binary classes;
- the Common Part 2 collections URI reflects draft/misaligned posture relative to the accepted Glaux class set; and
- no Part 3/MQTT class is claimed, which is appropriately restrained while that work remains draft and broker restrictions are not verified.

The repository includes many E2E tests labeled by conformance class and a test ensuring MQTT draft work does not overstate formal conformance. Those are valuable implementation tests. No official OGC ATS execution record or exact full conformance-payload test was found.

### 7.2 Implementation Findings Matrix

| Area | CS-GO anchor | Evidence | Observed behavior | CSAPI / Glaux baseline | Alignment | Strength / useful pattern | Gap / risk | Glaux applicability | Test implication | Handoff | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Assembly | `cmd/server/main.go` | C | Explicit config, DB, repos, MQTT, router, shutdown | Modular service baseline | Partial | Clear composition root | Security and migrations not integrated as capabilities | Adopt typed composition | Startup/shutdown/failure tests | 015, 018, 025 | Rust mechanism differs |
| Resource routes | `internal/api/router.go` | C | 93 GET/POST/PUT/DELETE registrations | Canonical Part 1/2 routes | Broad partial | Wide resource coverage | No PATCH; capability metadata separate | Use route registry | Route-contract inventory | 015, 019 | Presence is not conformance |
| Conformance | `conformance_handler.go` | C/T | Static 24-URI declaration | IDR-SRV-008 class set | Divergent | Explicit endpoint; restrained MQTT claim | Invalid, omitted, and removed class URIs | Generate from tested capabilities | Exact positive/negative class test | 050, 051 | Claim is not proof |
| Part 1 formats | formatter packages | C/T | GeoJSON and SensorML JSON | Accepted representation baseline | Partial | Typed format adapters | Exact-header matching; no 406/415 | Typed codecs plus negotiator | q-values, wildcards, bad types | 012, 023 | No HTML |
| Part 2 formats | JSON formatters | C/T | JSON for streams, observations, commands/status | JSON plus SWE classes | Partial | Typed domain/wire separation | No SWE text/binary | Preserve separation | Golden schema/round trip | 023, 034, 036 | Do not overclaim |
| Query parsing | `query_params` | C/T | Resource filters, spatial/time, `latest` | Canonical names and semantics | Partial/divergent | Central parsing | Silent unknown/invalid values; names differ | Typed per-resource grammar | table-driven negative tests | 011, 051 | `latest` too broad |
| Pagination | query/repository cursor code | C/T | Keyset cursor, deterministic tuples, indexes | Opaque supplied navigation; no public cursor baseline | Extension | Scalable keyset design | Base64 JSON lacks authenticity/principal/snapshot binding | Adopt authenticated opaque token if exposed | tamper/scope/snapshot tests | 011, 025, 052 | Reject offset is explicit |
| Errors | `internal/api/errors.go` and handlers | C/T | Ad hoc `error` JSON, mixed 400/409/500 | IDR-SRV-013 problem contract | Divergent | Decoder messages sanitized | No canonical problem schema or trace fields | Central problem mapper | status/body/header matrix | 013, 052 | Preserve request ID |
| Persistence | repositories and migrations | C/T | PostgreSQL/PostGIS, JSONB, transactions | Durable canonical model | Useful partial | Spatial/native indexes | Startup AutoMigrate; versioned migration absent | Adopt durable store, not migration method | upgrade/rollback/concurrency | 025, 028 | Single DB dependency |
| Validation | `resourcevalidation`, strict decoders | C/T/H | Required-field/schema checks and strict JSON | Accepted schema validation | Useful partial | Audit-driven hardening | Coverage and schema-version drift remain | Layer syntax/schema/semantic validation | mutation/unknown-field corpus | 023, 051 | Past invalid findings preserved |
| OpenAPI | `docs/openapi.json` | D/C | OAS 3.0, 42 paths, 82 ops, 8 schemas | Complete deployable OAD | Divergent | Checked-in artifact and `/api` route | 11+ runtime ops absent; shallow schemas; no parity CI | Generate and verify from contract registry | route/OAD diff in CI | 014, 050 | No operation IDs/security schemes |
| AsyncAPI/MQTT | MQTT/pubsub/API code | C/T/D | Config-derived AsyncAPI; ingestion/publication | Dynamic transport declared only when supported | Useful experimental | Capability-aware document; CloudEvents lifecycle | Generic payloads, broker policy/TLS unproven | Reuse capability derivation | broker ACL/reconnect/schema tests | 035, 052 | Default MQTT off |
| System events | event repo/handlers/tests | C/T | Persistent root and nested events | Approved Part 2 class | Broad partial | Durable event model | Auth/audit policy absent | Strong resource precedent | persistence/filter/cascade tests | 033, 034 | Better than transient-only eventing |
| Tasking | control/command handlers/tests | C/T | Persistent streams, commands, status/results | Part 2 tasking baseline | Partial | Clear lifecycle resources | No feasibility; query-name drift | Adapt state separation | transition/idempotency/auth tests | 036, 037 | History integrity separate |
| Security | router/config/compose | C/D | No REST auth; permissive CORS; proxy TLS assumption | Later zero-trust/security topics | Divergent for production | Simple deployability | Writes and commands exposed if deployed as-is | Security by construction | principal/tenant/abuse tests | 038, 039A, 052 | Highest operational risk |
| Test corpus | 69 test files, E2E schemas | T | 384 named tests and schema fixtures | Traceable conformance strategy | Useful partial | Broad regression evidence | No configured CI or formal ATS | Reuse patterns with requirement IDs | CI, ATS, OAD parity, security/perf | 050, 051, 053 | Source inventory, not executed result |
| Audit loop | OS4CSAPI evaluations + upstream issues | H/T | Findings led to fixes and invalid dispositions | Evidence/adjudication governance | Strong precedent | Fast feedback and regression creation | AI/audit findings can be wrong | Preserve disposition ledger | reproduce before/after tests | 014E, 014G, 056 | All 11 upstream issues closed |

## 8. CS-GO API Behavior Findings

### 8.1 Entry Points and Navigation

The router exposes `/`, `/conformance`, `/api`, `/api/ui/*`, `/asyncapi`, `/collections`, and the main resource collections. The landing response links to API and conformance and lists major Part 1 collections. It does not fully derive its links from the route/capability set, and it omits several Part 2 roots. Glaux should generate landing and collection navigation from the same registry as routing and documentation.

The `/collections/{collectionId}` route has a functioning `CollectionHandler`, while a separate stale `collections_handler.go` retains unregistered 501-style scaffold logic and generated descriptions still mention not-implemented behavior. Dead/scaffold code should be removed or mechanically excluded from documentation generation.

### 8.2 URIs, Methods, and Links

The code uses canonical-looking plural collection paths and nested parent/resource paths. It implements root and nested system events, nested observation/command/status/results, and system history. No feasibility routes were found. CORS advertises PATCH although the router registers no PATCH operation.

Absolute-link and enrichment fixes are now upstream. Glaux should make base URL, forwarded headers, canonical identifiers, link relations, and proxy deployment part of contract tests rather than handler-local concerns.

### 8.3 Pagination and Filters

Keyset pagination is one of CS-GO's most reusable patterns. Tokens encode deterministic tuple state, route scope, and a query hash; repositories have supporting indexes. This avoids large-offset rescans.

The public token is base64-encoded JSON and its SHA-256 material detects query mismatch only when recomputed; it is not an authenticated server secret. A client can construct or alter a syntactically valid token. It also lacks principal/tenant binding, snapshot boundary, expiry, and confidentiality. Glaux should either keep cursors internal behind supplied links or use versioned authenticated opaque tokens.

### 8.4 Content Negotiation

Formatter collections select by exact match on the entire `Accept` or `Content-Type` value. Unknown, compound, parameterized, or quality-weighted values select the default formatter. Unsupported request media can therefore be decoded as the default rather than returning 415, and unacceptable response media do not produce 406. No `Vary: Accept` handling was found.

Typed formatter separation is worth adopting; matching policy is not. Glaux's negotiator should parse media ranges, parameters, wildcards, q-values, and explicit format query rules, then return deterministic supported types or the accepted error response.

### 8.5 Error Behavior

Handlers generally emit `{"error":"..."}` maps with resource-specific status selection. Some deletion/conflict paths now use 409, strict-decoder failures use 400, and many repository failures use 500. The model lacks the accepted Glaux problem structure, stable type/code, instance, request identifier, field locations, and centralized disclosure policy.

Glaux should map typed domain errors once and test every operation for malformed syntax, schema failure, semantic failure, not found, conflict, unauthorized, forbidden, unacceptable representation, unsupported media, rate limit, and internal failure.

## 9. CS-GO Dynamic-Data, Status, and Tasking Findings

### 9.1 Historical and Live Data

Observations, commands, statuses, results, and events persist in PostgreSQL for historical REST access. Optional MQTT provides live ingestion/publication rather than HTTP streaming. The server ingests observations and command status and publishes observations, commands, command status, and system events.

This separation is useful: persistence is not dependent on a connected client, and live delivery is transport-specific. Glaux should define consistency, ordering, delivery guarantees, deduplication, reconnect, replay boundary, and authorization independently for REST history and each live transport.

### 9.2 MQTT and Resource Events

Resource lifecycle publication uses CloudEvents 1.0. Observation and command resource events can be batched over UTC windows, and graceful close flushes pending batches. AsyncAPI 2.6 is produced from current configuration, with channels omitted when corresponding classes are disabled.

The document uses permissive generic payloads for important resource-data channels. Broker ACLs, publisher restrictions, TLS, retained messages, QoS expectations, duplicate processing, ordering, backpressure, and reconnect behavior are not established as a formal conformance profile. The README appropriately states that draft Part 3 conformance is not claimed.

### 9.3 Status and Command Lifecycle

Persistent status and result subresources provide a useful lifecycle decomposition. Glaux should preserve immutable command intent, append-only or governed status transitions, separate result material, idempotency keys, cancellation semantics, authorization, and correlation. CS-GO supplies implementation examples but not a complete normative lifecycle decision.

### 9.4 Missing or Extension Behavior

No feasibility lifecycle was found. No HTTP WebSocket/SSE or replay protocol was found. System history is an extension relative to the accepted Part 2 direct-class set and appears mutable. These should remain explicit separate decisions rather than being inferred from adjacent functionality.

## 10. CS-GO Persistence, Data Model, and Fixture Implications

### 10.1 Storage Model

The release requires PostgreSQL with PostGIS. GORM maps approximately sixteen domain model families and startup invokes `AutoMigrate`. Custom migration helpers convert legacy arrays to JSONB, correct geometry SRIDs, add GiST and cursor indexes, and install deployment-closure triggers. This demonstrates real evolution pressure beyond simple ORM table creation.

The Makefile's `migrate` target still reports that migrations are not implemented. Startup mutation of the production schema is insufficient for controlled upgrades, rollback, blue/green deployment, data backfill, compatibility windows, and audit. Glaux should use explicit numbered migrations, compatibility tests, backup/restore procedures, and version gates.

### 10.2 Data Modeling Lessons

PostGIS-native geometry and spatial indexes are strong. JSONB is pragmatic for heterogeneous SensorML/SWE components and links, but must not become an unvalidated escape hatch. Relational associations, foreign keys, closure tables/triggers, and cursor indexes should follow access patterns and lifecycle invariants.

Glaux should separate canonical resource identity, external UID, version identity, relationship identity, event sequence, and transport correlation. It should define ownership and tenancy in the schema before persistence work is locked.

### 10.3 Fixtures and Examples

The `examples/` tree has 20 tracked entries, including payloads for systems, procedures, deployments, properties, sampling features, and generic features. E2E tests bundle GeoJSON, SensorML, JSON, OpenAPI-derived, and AsyncAPI fixtures across Part 1 and Part 2 families. These are useful seed material, not controlling golden truth.

Glaux may adapt fixture shapes only after recording source commit, license/provenance, approved schema version, expected validity, and intentional deviations. Negative fixtures and historical defect payloads are especially valuable because they encode silent-loss, cascade, time, link, and validation boundaries.

## 11. CS-GO Documentation, OpenAPI, and Examples Findings

### 11.1 OpenAPI Coverage

The release includes OpenAPI 3.0 JSON plus Swagger JSON/YAML and generated Go embedding. Mechanical inspection found 42 paths, 82 HTTP operations, eight component schemas, zero operation IDs, zero security schemes, and a server URL of `//localhost:8080`. The router contains 93 operation registrations, so at least 11 registered operations are not represented.

Missing or incomplete areas include `/asyncapi`, command status/result mutation routes, and portions of nested behavior. Many bodies use generic maps; component schemas are primarily collection wrappers and links. The document therefore supports discovery and UI use but is not a complete client-generation or deployment contract.

Glaux should generate its OAD from typed route/model metadata or enforce bidirectional parity in CI. Every runtime operation needs stable operation ID, schemas, parameters, responses, security, tags, and deployed-server/base-path behavior.

### 11.2 Documentation Drift

`IMPLEMENTATION.md` describes a Part 1 scaffold and planned behavior that current code has surpassed. README material is more current but refers to an obsolete/future Part 2 identifier and says Go 1.24+ while `go.mod` requires Go 1.25.0. The module path remains `github.com/yourusername/connected-systems-go`.

The release itself exposes another drift signal: `v1.0.4` ships assets named `server-v1.0.4-*`, but the pinned `dist/checksums.txt` lists the same five hashes under `server-v1.0.3-*`. Main's only post-release commit corrects those names. This is not proof of binary corruption; it is proof that release metadata requires automated consistency checks.

### 11.3 Distribution and Deployment

The GitHub release publishes five platform binaries and a checksum asset. Compose and README references are not fully consistent about image registry/tag. No GitHub Actions workflow was found, so the repository does not expose repeatable build/test/release provenance through configured CI.

Glaux should publish signed provenance, SBOMs, checksums tied to exact asset names, source/tag/build identity, container digests, reproducible workflow records, and compatibility-tested deployment examples.

## 12. Interoperability and Client-Compatibility Findings

### 12.1 OS4CSAPI Audit Loop

The OS4CSAPI fork is an evidence-rich audit branch, not the latest server. Its evaluations led to upstream issues covering the `/api` stub, datastream SQL, control-stream JSON leakage, system-event cascade, silent time-range parsing, empty times, constraint validation, link absolutization, and link enrichment. Current upstream issues 1–11 are all closed, and related fixes or clarifications were integrated during May 2026.

Several candidate findings were rejected or reclassified: numeric epoch rejection was correct, the queried identifier name was wrong, default limit 10 was correct, `updatable` semantics were misunderstood, and the SensorML publisher loss involved a client payload mismatch. Glaux must retain invalid/duplicate/not-reproducible dispositions, not silently delete them. That prevents later agents from reviving disproven claims.

### 12.2 Client Compatibility Risks

Generated clients and CSAPI Explorer are sensitive to complete OAD paths, schemas, operation IDs, canonical query names, link correctness, content negotiation, and deterministic errors. CS-GO's incomplete OpenAPI, exact-string media matching, query-name divergence, silent unknown query handling, and static conformance payload can produce false capability discovery or client-generation gaps.

The public endpoint referenced by repository deployment material could not supply live evidence: root, `/api`, `/api/`, `/api/conformance`, `/api/api`, `/api/systems`, `/api/asyncapi`, `/systems`, and `/conformance` all returned HTTP 404 on August 31, 2026. This establishes endpoint unavailability at that time, not normal server behavior.

### 12.3 Interoperability Handoff

IDR-SRV-014E should compare the OS4CSAPI smoke-test expectations and dispositions against the current `v1.0.4` release, not the audit fork's older runtime. IDR-SRV-014F should compare query vocabulary, error shapes, links, formats, and tasking resources with SECD. IDR-SRV-056 should use a cross-server matrix with normative expected behavior, observed variants, client tolerance, and explicit extensions.

## 13. Test-Strategy Implications

### 13.1 Existing Test Strengths

The release has 69 test files and 384 named test functions. E2E suites create a PostGIS Testcontainers dependency and exercise an in-process HTTP router. Coverage includes CRUD, canonical URLs, nested relationships, schema validation, strict required fields, link absolutization, cursor paging, spatial queries and media geometry, derived datastream time ranges, command status/results, system events/history, configuration, MQTT topics and ingestion, CloudEvents publication, and capability-derived AsyncAPI.

Formatter tests and bundled schema fixtures provide a strong basis for boundary and round-trip testing. The audit-derived regressions show that issue evidence can become durable protection.

### 13.2 Gaps and Limits

No configured CI workflow or formal OGC ATS result was found. The study could not execute tests locally. No comprehensive evidence was found for:

- exact conformance declaration versus enabled capabilities;
- route/OpenAPI parity and client generation;
- authentication, authorization, tenant isolation, or command policy;
- media-range weighting, 406, 415, or `Vary` behavior;
- cursor tampering, expiry, principal binding, snapshot consistency, or pagination under concurrent writes;
- feasibility behavior;
- migration upgrade/rollback and production-size data;
- performance, soak, load, backpressure, rate limits, or resource exhaustion;
- broker ACL, TLS, QoS, duplicate, ordering, reconnect, or replay guarantees; and
- multi-node consistency and failure recovery.

### 13.3 Glaux Candidate Test Layers

1. **Normative requirement tests:** map each approved requirement/class to executable assertions and record unsupported classes.
2. **Schema and semantic tests:** validate accepted schemas, field preservation, unknown members, unordered JSON, time/geometry edge cases, and cross-resource invariants.
3. **Golden contract tests:** landing, links, conformance, OAD, AsyncAPI, errors, pagination links/tokens, and every representation.
4. **Mutation and persistence tests:** transaction rollback, cascades, conflicts, ownership, concurrency, restart, migration, backup/restore, and immutable history.
5. **Security tests:** principals, scopes, tenant boundaries, command authorization, data disclosure, abuse controls, CORS/TLS, and audit.
6. **Transport tests:** delivery semantics, schema, ordering, duplication, reconnect, replay, backpressure, ACL, and capability discovery.
7. **Interoperability tests:** CSAPI Explorer, generated clients, OS4CSAPI clients, SECD cases, and cross-server tolerance with extensions isolated.
8. **Release tests:** reproducible build, SBOM, provenance, signature, asset/checksum name parity, container digest, and upgrade smoke tests.

## 14. Lessons for Glaux Server

### 14.1 Adopt

- A small explicit composition root with typed handler, repository, formatter, and transport interfaces.
- PostGIS-native geometry and indexes when the persistence decision supports them.
- Deterministic keyset paging and indexes, with tokens protected as opaque security objects.
- Strict decoding, schema validation, field-preserving round trips, and sanitized parse errors.
- Transactional cascades and explicit relationship/link builders.
- Capability-derived AsyncAPI and the same pattern generalized across every contract surface.
- Persistent system events and separated command/status/result resources.
- Audit finding → maintainer/authority adjudication → regression test → disposition ledger.

### 14.2 Avoid

- Static conformance declarations independent of runtime capability and test evidence.
- Startup ORM migration as the primary production evolution strategy.
- Silent fallback for unsupported media types or invalid query parameters.
- Public unsigned cursor state.
- Documentation generated from only a subset of routes or generic body maps.
- Write/tasking routes without authentication, authorization, ownership, audit, and transport security.
- Mutable/deleteable history without privileged governance.
- Treating fixture schemas, comments, or implementation test names as official conformance proof.

### 14.3 Investigate Further

- Whether current `v1.0.4` passes the official or project-derived complete requirement suite.
- Maintainer intent and correction schedule for conformance URIs and query vocabulary.
- Licensing and ownership needed for any fixture or source reuse.
- Broker topology, ACLs, QoS, replay, and delivery guarantees behind the MQTT design.
- Production scale, migration history, PostGIS query plans, and concurrent paging behavior.

## 15. Downstream Topic Handoff Matrix

| Topic | Handoff from this report | Required use |
|---|---|---|
| IDR-SRV-014C | Compare pygeoapi's scope, plugins, conformance, OAD, persistence, negotiation, and tests using the same evidence labels | Do not assume CS-GO's breadth or gaps are shared |
| IDR-SRV-014D | Compare SECD routes, formats, errors, query names, tasking, and deployment evidence | Identify deliberate profile versus implementation drift |
| IDR-SRV-014E | Re-run/interpret OS4CSAPI findings against current upstream `v1.0.4` | Preserve fixed, invalid, client-side, and unresolved dispositions |
| IDR-SRV-014F | Compare SECD interop findings with CS-GO risks | Build normative/observed/client-tolerance matrix |
| IDR-SRV-014G | Extract governance lessons from OS4CSAPI discussions | Include evidence-to-adjudication workflow |
| IDR-SRV-015 | Consider typed domain/repository/formatter boundaries | Define identity, version, relation, event, and extension model first |
| IDR-SRV-023 | Use strict decoder, schema fixtures, and silent-loss regressions | Build layered syntax/schema/semantic validation |
| IDR-SRV-025 | Evaluate PostGIS, JSONB, indexes, transactions, closure relationships | Require versioned migrations and ownership/tenancy |
| IDR-SRV-034 | Use persistent observations/status/events and transport separation | Define history/live consistency and `latest` semantics |
| IDR-SRV-036 | Use separated control stream/command/status/result resources | Add feasibility, authorization, idempotency, and immutable audit |
| IDR-SRV-050 | Treat conformance URI as a tested capability output | Run official/project ATS and exact declaration tests |
| IDR-SRV-051 | Convert issue/regression links into requirement-to-test records | Retain invalid and fixed dispositions |
| IDR-SRV-053 | Mine schema fixtures and historical defect payloads with provenance | Revalidate against approved schema pins |
| IDR-SRV-056 | Test canonical query names, links, OAD, negotiation, errors, and cursors across clients | Separate required behavior, aliases, and extensions |

## 16. Recommendations

1. **Create one typed capability and contract registry.** Priority: High. It should drive routes, landing links, collections, conformance, OpenAPI, AsyncAPI, authorization scopes, and required tests.
2. **Make security a prerequisite for transactional and command capabilities.** Priority: High. Identity, policy, ownership/tenancy, TLS, audit, abuse limits, and broker ACLs must be enabled and tested before claims.
3. **Adopt strict, layered validation and lossless round-trip tests.** Priority: High. Include every writable member, relationship, unordered JSON, unknown fields, time/geometry boundaries, cascade, and restart behavior.
4. **Use explicit versioned migrations.** Priority: High. Require forward/backward compatibility, data backfill, rollback/restore, and production-scale verification; do not depend on startup `AutoMigrate`.
5. **Implement canonical query and negotiation contracts.** Priority: High. Use accepted names and per-resource semantics, reject/diagnose invalid input deterministically, parse media ranges, and implement 406/415 and `Vary` correctly.
6. **Treat cursors as security-sensitive protocol objects.** Priority: High if exposed. Bind version, query, ordering, principal/tenant, and snapshot/expiry with authenticated confidentiality where required.
7. **Generate and continuously verify API descriptions.** Priority: High. Route/OAD parity, stable operation IDs, complete schemas/responses/security, deployed servers, and client-generation smoke tests belong in CI.
8. **Adopt the OS4CSAPI audit disposition workflow.** Priority: Medium. Every finding should record evidence, authority check, reproduction, disposition, fix commit, and regression; invalid findings remain visible.
9. **Reuse fixtures only through a provenance gate.** Priority: Medium. Record source pin, license, schema pin, validity class, expected outcome, and any Glaux adaptation.
10. **Carry MQTT as experimental evidence until transport guarantees are specified.** Priority: Medium. Capability-derived AsyncAPI is reusable; broker and delivery semantics require independent design and tests.

## 17. Risks, Constraints, and Open Questions

### 17.1 Risks and Constraints

- **Conformance overstatement:** static and invalid class URIs can mislead clients and project planning.
- **Security exposure:** a deployment using default configuration can expose broad writes and commands without REST authorization.
- **Schema evolution:** runtime `AutoMigrate` and JSONB flexibility can hide incompatible changes.
- **Contract drift:** routes, generated OpenAPI, README, implementation notes, module path, images, and checksums already show independent evolution.
- **Interoperability:** query names, media fallback, errors, and unsigned cursors can break strict or generated clients.
- **Evidence limit:** no local Go/Docker execution and no configured upstream CI prevent a fresh verified test result.
- **Demo limit:** repository-referenced endpoint returned only 404 responses.
- **Reuse limit:** absent license text blocks confident direct source or fixture reuse.
- **Temporal limit:** upstream is active; all mutable conclusions require the recorded pins and retrieval date.

### 17.2 Open Questions

1. Will upstream correct the conformance set and derive it from enabled/tested capabilities?
2. Is there a maintained public demo or deployment profile that can support positive observation?
3. What license grant and copyright ownership apply to code, schemas, examples, and audit fixtures?
4. Which official ATS version, if any, has been executed against `v1.0.4`?
5. What are the production authorization, migration, backup, performance, and broker-security practices outside this repository?
6. Are `commandFormat`, `currentStatus`, broad `latest`, and public cursor deliberate extensions, transitional names, or defects?
7. What immutability and audit policy is intended for system history and command lifecycle records?

None of these questions blocks use of the report's bounded implementation lessons. Each blocks a stronger claim about conformance, production readiness, security, or reuse.

## 18. Validation Against the Research Plan

| Topic plan success criterion | Status | Evidence |
|---|---|---|
| Exact CS-GO sources and versions identified | Met | Section 3; Appendix B |
| Code, generated artifacts, fixtures, docs, tests, demos, and inference distinguished | Met | Reading Guide; Sections 3, 11, and 13 |
| Architecture and module boundaries summarized | Met | Section 4 |
| Part 1 and Part 2 compared to Glaux baseline | Met | Sections 5–7 |
| Conformance assessed without assumption | Met | Section 7 |
| Entry point, navigation, query, representation, error, OpenAPI, dynamic data, status, and tasking assessed | Met | Sections 8–11 |
| Strengths, gaps, risks, and assumptions identified | Met | Sections 7, 14, and 17 |
| Design, validation, testing, and interoperability lessons documented | Met | Sections 12–16 |
| Downstream handoffs explicit | Met | Section 15; Appendix C |
| References explicit and reproducible | Met | Sections 3 and 19; Appendix B |

All six research phases are complete. All five core and 42 detailed questions are answered or explicitly bounded in Appendix A. The twelve required matrix fields appear in Section 7.2. The report is ready for project-lead review; acceptance remains intentionally pending.

## 19. References

### Controlling and Glaux Sources

- [OGC API - Connected Systems landing page](https://ogcapi.ogc.org/connectedsystems/)
- [OGC API - Connected Systems Part 1](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems Part 2](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC Connected Systems development repository, `v1.0.0`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0)
- [OGC API - Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-006 through IDR-SRV-014A reports](./)

### Connected Systems Go Sources

- [Upstream release source `v1.0.4`](https://github.com/SomethingCreativeStudios/connected-systems-go/tree/244f4dd586da685d4d9b75e43f73001028b5bd0e)
- [Release `v1.0.4`](https://github.com/SomethingCreativeStudios/connected-systems-go/releases/tag/v1.0.4)
- [Router](https://github.com/SomethingCreativeStudios/connected-systems-go/blob/244f4dd586da685d4d9b75e43f73001028b5bd0e/internal/api/router.go)
- [Conformance handler](https://github.com/SomethingCreativeStudios/connected-systems-go/blob/244f4dd586da685d4d9b75e43f73001028b5bd0e/internal/api/conformance_handler.go)
- [Formatter collection](https://github.com/SomethingCreativeStudios/connected-systems-go/blob/244f4dd586da685d4d9b75e43f73001028b5bd0e/internal/model/formaters/formatter.go)
- [Server composition](https://github.com/SomethingCreativeStudios/connected-systems-go/blob/244f4dd586da685d4d9b75e43f73001028b5bd0e/cmd/server/main.go)
- [Generated OpenAPI](https://github.com/SomethingCreativeStudios/connected-systems-go/blob/244f4dd586da685d4d9b75e43f73001028b5bd0e/docs/openapi.json)
- [E2E tests and fixtures](https://github.com/SomethingCreativeStudios/connected-systems-go/tree/244f4dd586da685d4d9b75e43f73001028b5bd0e/e2e)
- [Examples](https://github.com/SomethingCreativeStudios/connected-systems-go/tree/244f4dd586da685d4d9b75e43f73001028b5bd0e/examples)
- [Upstream issues](https://github.com/SomethingCreativeStudios/connected-systems-go/issues?q=is%3Aissue)
- [Upstream pull requests](https://github.com/SomethingCreativeStudios/connected-systems-go/pulls?q=is%3Apr)
- [OS4CSAPI audit fork](https://github.com/OS4CSAPI/connected-systems-go/tree/e900da88738cca92872038b703c4ad537fc0c8fd)
- [OS4CSAPI issue evaluations](https://github.com/OS4CSAPI/connected-systems-go/tree/e900da88738cca92872038b703c4ad537fc0c8fd/docs/research/issue-evaluations)
- [OS4CSAPI upstream reports](https://github.com/OS4CSAPI/connected-systems-go/tree/e900da88738cca92872038b703c4ad537fc0c8fd/docs/research/upstream-issue-reports)
- [Historical mvanhorn fork](https://github.com/mvanhorn/connected-systems-go/tree/6665c737f1857b24741237a37779aee03cf3d694)

## Appendix A. Detailed Question Ledger

| ID | Detailed question short form | Disposition | Report evidence |
|---|---|---|---|
| S1 | Relevant repos, forks, tags, commits, issues, PRs, demos | Answered | Section 3 |
| S2 | OS4CSAPI-to-original relationship | Answered: audit fork, older runtime | Sections 3 and 12 |
| S3 | Exact study baseline | Answered: upstream `v1.0.4` plus audit pin | Section 3 |
| S4 | Artifact classification | Answered | Sections 3, 10, 11, 13 |
| S5 | Licensing/reuse | Answered with unresolved grant | Sections 3.3 and 17 |
| A1 | Go application structure | Answered | Section 4.1 |
| A2 | Routing/handler/middleware/model/data/config/serialization | Answered | Section 4 |
| A3 | Internal resource representation | Answered | Sections 4.2 and 10.2 |
| A4 | State/files/persistence/fixtures/live-feed assumptions | Answered | Sections 9–10 |
| A5 | Reusable versus Go-specific patterns | Answered | Sections 4 and 14 |
| C1 | Part 1 resources/behavior | Answered | Section 5 |
| C2 | Part 2 resources/behavior | Answered | Section 6 |
| C3 | Claimed/implied/unsupported/unclear classes | Answered | Section 7 |
| C4 | Landing/API/conformance/collections/links/schemas/media/errors | Answered | Sections 7–8 and 11 |
| C5 | Gaps versus Glaux baseline | Answered | Section 7.2 |
| C6 | Evidence per alignment/gap | Answered | Section 7.2 and references |
| B1 | URI patterns | Answered | Section 8.2 |
| B2 | Links and related resources | Answered | Sections 5.2 and 8.2 |
| B3 | Query/filter/sort/pagination/selection | Answered | Sections 5.4 and 8.3 |
| B4 | Content negotiation/media types | Answered | Section 8.4 |
| B5 | Error responses | Answered | Section 8.5 |
| B6 | API docs/OpenAPI artifacts | Answered | Section 11.1 |
| B7 | Handoffs to behavior topics | Answered | Sections 15–16 |
| D1 | Datastream/observation/dynamic representation | Answered | Sections 6.1 and 9.1 |
| D2 | Real-time/history/status/events/stream/replay | Answered; unavailable areas explicit | Section 9 |
| D3 | Control/commands/feasibility/status | Answered | Sections 6.2 and 9.3–9.4 |
| D4 | Implemented/partial/absent/unclear dynamic features | Answered | Sections 6 and 9 |
| D5 | Dynamic/streaming/command handoffs | Answered | Section 15 |
| P1 | Storage/persistence model | Answered | Section 10.1 |
| P2 | Generated/hand-authored/schema/fixture models | Answered | Sections 3.2 and 10 |
| P3 | Test data/examples/golden responses | Answered | Section 10.3 |
| P4 | Model/persistence limitations to avoid | Answered | Sections 10.1–10.2 |
| P5 | Fixture patterns for Glaux | Answered with provenance condition | Sections 10.3 and 13 |
| I1 | Clients/tools used | Answered within available audit evidence | Section 12 |
| I2 | OS4CSAPI smoke/compatibility observations | Answered | Sections 12.1–12.2 |
| I3 | Explorer/generated/external tooling needs | Answered | Section 12.2 |
| I4 | Interoperability risks | Answered | Sections 8 and 12 |
| I5 | Handoffs to 014E/014F/056 | Answered | Sections 12.3 and 15 |
| V1 | Existing tests | Answered | Section 13.1 |
| V2 | Untested/difficult behavior | Answered | Section 13.2 |
| V3 | Candidate positive/negative/schema/conformance/golden/interop tests | Answered | Section 13.3 |
| V4 | Comparison with OSH/pygeoapi/SECD/client findings | Answered as explicit downstream work | Section 15 |

## Appendix B. Reproducible Study Record

### B.1 Mechanical Inventory

| Item | Result |
|---|---|
| Upstream release | `v1.0.4`, annotated tag commit `244f4dd586da685d4d9b75e43f73001028b5bd0e`, tree `f85a031534040582ed434142192e57bb34ff0b62` |
| Upstream main comparison | `4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952`; only `dist/checksums.txt` differs from release |
| Release source inventory | 295 tracked files; 211 Go files; 69 test files; 384 named tests |
| Router inventory | 93 registered HTTP method/path operations |
| Conformance inventory | 24 returned URIs |
| OpenAPI inventory | OAS 3.0.0; 42 paths; 82 operations; 8 schemas; 0 operation IDs; 0 security schemes |
| Examples | 20 tracked entries under `examples/` |
| CI | No workflow under `.github/workflows`; only Copilot instructions found under `.github` |
| License | No repository license file; GitHub license detection null |
| OS4CSAPI audit pin | `e900da88738cca92872038b703c4ad537fc0c8fd`, tree `7cfa5ec80d6ac0f07227746989a3b1a165f4c557` |
| OS4CSAPI audit corpus | 27 evaluations; 13 prepared upstream reports |
| Historical fork pin | `6665c737f1857b24741237a37779aee03cf3d694` |

### B.2 Runtime and Tool Limitations

- Local `go` executable: unavailable.
- Local Docker executable: unavailable.
- Configured upstream CI workflow: not found.
- Public endpoint probes: all returned HTTP 404 on 2026-08-31.
- Consequently, code/test inventory is not presented as a fresh runtime pass.

### B.3 Release Integrity Observation

GitHub reported five `v1.0.4` platform assets with SHA-256 digests. The tag's `dist/checksums.txt` contains those same five digest values but names the binaries `v1.0.3`; main corrects the filenames. The finding is metadata drift. Independent artifact verification and build provenance remain separate needs.

## Appendix C. Completion and Handoff

- Research phases 1–6: complete.
- Deliverable draft and internal review: complete.
- Report state: **In Review**.
- Acceptance: pending Glaux Project Lead.
- Next topic after acceptance: **IDR-SRV-014C — pygeoapi CSAPI Server Implementation Study**.
- Workflow boundary: accepting this report and saying `proceed` authorizes finalization/merge of IDR-SRV-014B and execution of exactly IDR-SRV-014C; it does not authorize IDR-SRV-014D.

---

## Report Completion Checklist

- [x] Topic ID matches overall research plan index
- [x] Topic research plan is linked and aligned
- [x] Core and detailed research questions are covered or explicitly bounded
- [x] Findings are evidence-backed with reproducible references
- [x] Normative and informative evidence are classified and not conflated
- [x] Mutable sources identify release, tag, commit, and retrieval date
- [x] Inaccessible, missing, and ambiguous evidence limitations are explicit
- [x] Source-backed findings, inference, and recommendations are distinguishable
- [x] Findings are reconciled with accepted IDR-SRV-006 through IDR-SRV-014A
- [x] Executive summary is independently readable
- [x] Recommendations are explicit and actionable
- [x] Risks and open questions are documented
- [x] Success criteria validation is complete
- [x] Plan-owner acceptance and acceptance date are recorded
- [x] Next handoff is explicit

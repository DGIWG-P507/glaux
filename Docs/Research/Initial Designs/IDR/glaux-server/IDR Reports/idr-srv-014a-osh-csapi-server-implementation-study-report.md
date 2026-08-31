# Section 014A: OSH CSAPI Server Implementation Study - Research Report

**Topic ID:** IDR-SRV-014A<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-014A OSH CSAPI Server Implementation Study](../IDR%20Plans/idr-srv-014a-osh-csapi-server-implementation-study.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 42 detailed questions; all six methodology phases, ten success criteria, nineteen required content areas, and twelve minimum implementation-findings fields are validated<br>
**Methodology Used:** Commit-pinned source, documentation, test, configuration, issue, release, and build-evidence review; route and handler inventory; comparison with accepted IDR-SRV-006 through IDR-SRV-014; bounded live-demo probes; and synthesis using the OS4CSAPI exemplar corpus<br>
**Research Time:** Approximately 4.5 hours of AI-assisted elapsed execution on August 31, 2026<br>
**Primary Sources:** OpenSensorHub Core tag `v2.0.2` at `235c0eabf24b6d6137b499b4402943d2794b70e6`; OSH documentation commit `ec51a67af82c8e0bb309cfe1975cd17c44220e77`; approved OGC API - Connected Systems Parts 1 and 2; and accepted Glaux IDR-SRV-006 through IDR-SRV-014 reports<br>
**Supporting Resources:** OSH upstream issues and build evidence current through August 31, 2026; OS4CSAPI `phase-9` exemplars at `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`; and the shared CSAPI upstream-history register<br>
**Document Purpose:** Establish a reproducible OSH implementation findings baseline that Glaux can use for architecture, behavior, validation, interoperability, and test design without treating OSH as standards authority or copying its Java-specific structure<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** August 31, 2026<br>
**Date:** August 31, 2026<br>
**Last Updated:** August 31, 2026

---

## Reading Guide

This report uses the following evidence labels:

- **N — normative:** approved CSAPI or incorporated standards evidence;
- **P — project baseline:** an accepted prior Glaux IDR conclusion;
- **C — code-supported:** behavior established directly in pinned OSH source;
- **T — test-supported:** behavior covered by pinned automated test source or the configured upstream build;
- **D — documented:** OSH project documentation or example material;
- **A — API-observed:** a response observed from a dated live endpoint probe;
- **H — history:** issue, pull request, release, or maintainer-tracker evidence that remains informative; and
- **I — inference:** bounded analysis that is not itself a directly observed runtime fact.

“Implemented” in this report means that relevant code exists. It does not mean that every normative requirement of a conformance class is satisfied. “Claimed” means that OSH returns a class URI from its fixed conformance set. OSH is informative implementation evidence; the approved standards and accepted Glaux reports remain controlling.

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. OSH Source Inventory and Evidence Classification
4. OSH Architecture and Module-Boundary Findings
5. OSH CSAPI Part 1 Behavior Findings
6. OSH CSAPI Part 2 Behavior Findings
7. OSH Conformance Posture and Standards-Alignment Matrix
8. OSH API Behavior Findings
9. OSH Dynamic-Data, Streaming, Status, and Tasking Findings
10. OSH Persistence, Datastore, and Performance Implications
11. OSH Documentation, OpenAPI, and Examples Findings
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

OpenSensorHub is a valuable implementation reference for Glaux because it demonstrates a broad, modular Connected Systems API service over real sensor, observation, command, event-bus, and federated-datastore abstractions. The dedicated `sensorhub-service-consys` module contains handlers for systems, deployments, procedures, properties, sampling features, datastreams, observations, control streams, commands, statuses, and results; server-rendered HTML and JSON/GeoJSON/SensorML/SWE bindings; historical replay and live WebSocket delivery; granular permissions; and a substantive automated test corpus. OSH's separation of read federation from a write database, handler graph, representation bindings, event-driven streaming, and typed store filters are all useful design evidence.

The implementation is not a safe conformance oracle. At pinned tag `v2.0.2`, OSH returns a fixed set of 33 conformance URIs independent of deployment capabilities. Its CSAPI claims omit the approved `api-common`, `advanced-filtering`, `update`, and `feasibility` class URIs; substitute a nonexistent Part 1 `core` URI; add an unapproved Part 2 `system-history` URI; and declare Part 3 WebSocket and MQTT classes even though this study established concrete WebSocket code but no MQTT adapter in the inspected service module. No conformance-endpoint tests, formal ATS results, schema-validation suite, or implementation-specific OpenAPI parity tests were found. The declarations therefore describe project intent more reliably than proven class satisfaction.

Several code and tracker findings expose especially useful negative lessons:

- only GET, POST, PUT, and DELETE servlet methods exist; no PATCH/update-class path was found;
- deployment and sampling-feature association fields have been reported as accepted and then silently dropped;
- canonical query names differ from OSH aliases or remain absent in multiple areas;
- JSON parsing has an open ordering dependency;
- conflict cases are generally mapped to HTTP 400 rather than the accepted Glaux 409 contract;
- pagination rescans from the beginning for observations and hard-caps pages at 10,000 independently of the configured 100,000 response limit;
- the fixed conformance payload is not filtered by read-only state or configured resources;
- API documentation links point to mutable upstream OGC Part 1 and Part 2 descriptions rather than a complete, deployment-specific contract; and
- the public demo named in current documentation returned HTTP 502 for root, conformance, and systems probes on August 31, 2026.

These findings do not negate OSH's strengths. They show why Glaux should reuse patterns, not behavior by imitation. Glaux should adopt capability-driven handler/registry composition, explicit read/write boundaries, asynchronous backpressure, historical/live separation, typed filtering, and deep resource tests. It should improve on OSH with capability-derived conformance, a complete generated deployment OpenAPI description, lossless round-trip tests for every writable field and link, canonical query vocabulary, unordered JSON testing, deterministic error/status semantics, cursor-aware scalability analysis, persistent canonical system events, explicit feasibility lifecycle behavior, and transport claims backed by adapters and tests.

No new project-lead decision is required to understand the evidence. Acceptance of this report would establish its recommendations as implementation-study guidance and make IDR-SRV-014B the next eligible topic; it would not assert that OSH is nonconformant as a whole, choose a Rust framework, or authorize execution beyond 014B.

## 2. Scope and Plan Alignment

### 2.1 Included and Excluded Scope

The study directly reviewed current OSH server code, tests, configuration, documentation, issues, release/tag state, live-demo availability, and configured upstream build evidence. It compared those sources against the accepted Glaux behavior baseline in IDR-SRV-006 through IDR-SRV-014 and extracted architecture, API, datastore, streaming, tasking, documentation, interoperability, and testing lessons.

The study does not certify OSH conformance, benchmark OSH performance, audit the entire OSH organization, test unpublished deployments, or decide Glaux's Rust framework, persistence engine, security architecture, or final transport profile. It also does not execute IDR-SRV-014B through IDR-SRV-014G. Maintainer issue reports are treated as dated informative evidence, corroborated with code where feasible, not as universal runtime proof.

### 2.2 Plan Coverage Matrix

| Plan question group | Coverage | Evidence location |
|---|---|---|
| Source/version baseline and licensing | Complete | Sections 3 and 19; Appendix B |
| Architecture and module boundaries | Complete | Section 4 |
| Part 1 and Part 2 implementation posture | Complete | Sections 5–7 |
| Landing, navigation, query, negotiation, errors, and documentation | Complete | Sections 8 and 11 |
| Dynamic data, streaming, status, commands, and feasibility | Complete | Sections 6 and 9 |
| Persistence, indexing, scale, and performance implications | Complete within source evidence | Section 10 |
| Client and interoperability behavior | Complete within available endpoint evidence | Section 12 |
| Validation, fixtures, and test lessons | Complete | Section 13 |
| Glaux applicability, recommendations, and handoffs | Complete | Sections 14–17 |

## 3. OSH Source Inventory and Evidence Classification

### 3.1 Reproducible Source Baseline

| Source | Role | Pin / status | Evidence class | Access and limitation |
|---|---|---|---|---|
| [OpenSensorHub Core](https://github.com/opensensorhub/osh-core/tree/235c0eabf24b6d6137b499b4402943d2794b70e6) | Server implementation, stores, bindings, tests | Lightweight tag `v2.0.2`; commit `235c0eabf24b6d6137b499b4402943d2794b70e6`; tree `123f77e7e7c5a84764ff3c60268e585e2212cc0b`; 2026-07-29 | C/T | Full repository inspected locally. Tag is reproducible but not represented by a GitHub Release. |
| [OSH documentation](https://github.com/opensensorhub/osh-docs/tree/ec51a67af82c8e0bb309cfe1975cd17c44220e77) | Architecture, setup, service/client guidance, examples, demo links | Commit `ec51a67af82c8e0bb309cfe1975cd17c44220e77`; tree `059c5f124d4ffc5e0a07f3c0b23eed37df9a70f7`; 2026-06-24 | D | Project-authored and current at the pin; statements are not independent conformance evidence. |
| [`sensorhub-service-consys`](https://github.com/opensensorhub/osh-core/tree/235c0eabf24b6d6137b499b4402943d2794b70e6/sensorhub-service-consys) | Dedicated CSAPI server and client module | Same `v2.0.2` pin; 202 files, including 157 production and 38 test Java files | C/T | Principal study target. Gradle excludes the package's client tests. |
| [OSH Core issues](https://github.com/opensensorhub/osh-core/issues) | Current defects, gaps, and maintainer/user reports | States checked 2026-08-31 | H | Mutable and informative. Code corroboration or test reproduction is stated separately. |
| [Configured upstream build](https://github.com/opensensorhub/osh-core/actions/runs/30473581984/job/90649579129) | Build/test evidence for pinned commit | `build`, success, 2026-07-29 | T | Confirms the repository's configured build, not formal CSAPI conformance or excluded tests. |
| [Documented public demo](https://api.georobotix.io/ogc/demo1/api) | Potential live API observation | Root, `/conformance`, and `/systems` returned HTTP 502 on 2026-08-31 | A | No positive runtime observations could be established from this endpoint. |
| [OS4CSAPI research exemplars](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans) | Research structure and test-handoff rigor | `phase-9` commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230` | Supporting method | Blueprint, fixture-sourcing, and testing-playbook exemplars guided inventories, limitations, and actionable tests. |
| Approved CSAPI Parts 1 and 2 and accepted IDR-SRV-006–014 | Controlling comparison baseline | Versions and decisions recorded in accepted reports | N/P | Control whenever OSH behavior or declarations differ. |

OSH Core is licensed under [MPL-2.0](https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/LICENSE). Glaux may study and independently implement patterns, but any direct reuse would require a deliberate license and provenance review. This report recommends no source copying.

### 3.2 Evidence Strength Rules

Code plus an active test is **strong implementation evidence** for the exercised path. Code without a relevant test is **moderate evidence**. Documentation alone is **documented intent**. A tracker report is **dated issue evidence**, even when detailed. A fixed conformance URI is **a claim**, not proof. Absence means “not found within the pinned, scoped sources,” not a universal claim about every OSH add-on or deployment.

The live demo's unavailability prevented response-level confirmation. Local test execution was also unavailable because the study environment had neither `JAVA_HOME` nor a Java executable. The successful upstream job at the exact pinned commit mitigates build uncertainty, but it does not remove the test-scope limitations described in Section 13.

## 4. OSH Architecture and Module-Boundary Findings

### 4.1 Platform Shape

OSH Core is a Java/OSGi modular sensor platform. Sensor and actuator drivers publish through common interfaces and an event bus; databases implement typed store interfaces; services expose those stores through protocols. The Connected Systems service is therefore an adapter over an existing operational hub, not a standalone CRUD application.

`ConSysApiService.doStart()` resolves a write database from `databaseID` or the system-state database and creates a read view from either configured resource filters or the federated database. It supplies feature, procedure, property, system, deployment, sampling-feature, datastream, observation, command-stream, command, status, and related stores through a common `HandlerContext`. A `RootHandler` then composes specialized subresource handlers and marks them read-only when no writable database exists.

This architecture yields three strong patterns for Glaux:

1. route/resource composition can be capability-driven and modular;
2. storage and API logic can depend on typed domain-store interfaces rather than a specific engine; and
3. live events, persisted state, and API representation bindings can remain separate layers.

The Java/OSGi lifecycle, OSH module registry, `BigId` identifiers, federated-database mechanics, and driver-specific event bus are OSH-specific. Glaux should preserve the boundaries, not reproduce those mechanisms in Rust.

### 4.2 Handler and Binding Boundaries

The module separates generic resource dispatch from domain handlers and serialization bindings. `BaseResourceHandler` implements collection/item dispatch, paging, permissions, and mutations. Domain packages specialize filter construction and relationships. JSON, GeoJSON, SensorML, observation, SWE, and HTML bindings provide representation behavior. Streaming adapters sit beside resource handlers rather than inside persistence code.

This is a useful decomposition, but the study found behavior metadata distributed across handlers, static conformance constants, home-page builders, parsers, and format bindings. That distribution helps explain declaration/runtime drift. Glaux should retain modular implementations behind one typed capability/contract registry that generates or verifies routing, conformance, documentation, and tests.

### 4.3 Read and Write Topology

OSH can expose a filtered federated read view while directing mutations to one database. That enables aggregation and selective exposure. It also creates an ownership problem: [issue #273](https://github.com/opensensorhub/osh-core/issues/273) reports HTTP-successful updates against resources visible through the federated view but not owned by the configured write database. The architecture needs explicit routing and rejection semantics, not only a readable/writable boolean.

Glaux should model source authority, resource ownership, and mutation routing as typed policy. A visible resource must either resolve to an authorized write owner or reject before mutation with a deterministic error. Read-only state and resource filters must also alter advertised operations and conformance claims.

### 4.4 Security Boundary

OSH supports optional authentication/access-control integration and defines granular permissions by resource family and operation (get/list/stream/create/update/delete). This is preferable to route-only authorization. The evidence does not establish a secure deployment default, OAuth/OIDC client completeness, or policy equivalence to Glaux's later zero-trust and command-control requirements. [Issue #301](https://github.com/opensensorhub/osh-core/issues/301) records missing OAuth/OIDC support in the CSAPI client.

## 5. OSH CSAPI Part 1 Behavior Findings

### 5.1 Resource Breadth

Pinned code registers root handlers for systems, deployments, procedures, properties, sampling features (with legacy `fois` alias), generic features, and collections. It also supports nested systems/subsystems/members/history, deployment membership/subdeployments, system sampling features, datastreams, and control streams. GeoJSON and SensorML bindings and server-rendered HTML are concrete implementation strengths.

CRUD paths exist for major Part 1 resources, but the servlet implements GET, POST, PUT, and DELETE only. No PATCH path was found, so the approved Part 1 update class is not established. The fixed declaration includes create-replace-delete but omits update and advanced filtering. Duplicate/resource-integrity and cascade cases generally become HTTP 400 through request-rejection handling rather than Glaux's accepted 409 conflict semantics.

### 5.2 Relationships and Round Trips

OSH's nested handler graph provides useful navigation, but issue evidence identifies loss at important relationship boundaries:

- [#337](https://github.com/opensensorhub/osh-core/issues/337) reports deployment association fields accepted with success and then absent, plus failing deployment-scoped datastream paths;
- [#339](https://github.com/opensensorhub/osh-core/issues/339) reports an observation `samplingFeature@link` accepted and then dropped; and
- [#273](https://github.com/opensensorhub/osh-core/issues/273) reports ambiguous write routing over federated exposure.

These are stronger lessons than simple route inventories: create/update success must be followed by representation round-trip, relationship traversal, reverse lookup, and persistence-restart checks. A 201 or 204 alone is insufficient evidence.

### 5.3 Query and Representation Behavior

Typed filters support UIDs, temporal ranges, spatial filters, parent/member traversal, paging, and projection. However, [issue #285](https://github.com/opensensorhub/osh-core/issues/285) inventories canonical-name gaps and aliases such as `searchMembers` for `recursive`, `uid` in sampling-feature contexts, `datastream` rather than `dataStream`, and `stream` rather than `controlStream`, alongside missing observed/controlled-property, system, and event filters. Compatibility aliases can be useful, but they must not replace the standard vocabulary.

JSON, GeoJSON, SensorML JSON/XML, and HTML coverage is broad. [Issue #275](https://github.com/opensensorhub/osh-core/issues/275) reports that a JSON parser depends on member order, contrary to JSON object semantics. Glaux must test field-order permutations and unknown/additional fields for every writable representation.

## 6. OSH CSAPI Part 2 Behavior Findings

### 6.1 Datastreams and Observations

OSH has substantial Part 2 data behavior: datastream resource and schema handlers, observation collection/item behavior, temporal filtering, result selection, statistics, JSON/OM/SWE bindings, historical replay, and live WebSocket output. The H2/MVStore implementation and typed observation stores provide a credible operational substrate.

Known limitations matter at semantic edges. [Issue #331](https://github.com/opensensorhub/osh-core/issues/331) reports that combined `phenomenonTime` and `resultTime=latest` can yield multiple observations. [Issue #284](https://github.com/opensensorhub/osh-core/issues/284) reports insertion failure for nested variable-size `DataArray` structures. The code also uses `application/swe+csv` where the accepted Glaux baseline identifies standard SWE text as `application/swe+text`; Glaux may support CSV as an explicit extension but must not substitute it in a conformance claim.

### 6.2 Control Streams and Commands

The handler and store model covers control streams, control schemas, commands, command status, and command results. Client-side helpers and server tests demonstrate meaningful implementation intent. This is a useful model for separating command definition, submission, status transitions, and results.

No complete feasibility resource handler was found. Feasibility-related schema fields exist, while code comments indicate feasibility results are currently dropped. OSH therefore does not establish the approved Part 2 feasibility class despite broad tasking support. Glaux should make feasibility a persistent, testable lifecycle or omit its class claim until complete.

### 6.3 Events and History

OSH publishes resource events through the event bus and exposes event subscriptions nested under resource handlers. System history is also implemented as an extension. The study did not find the canonical, persisted root system-event resource behavior required by the approved Part 2 system-event class. Conversely, `system-history` is returned as a Part 2 conformance URI even though accepted IDR-SRV-008 excluded it from the approved direct class set.

Glaux should distinguish resource change notifications, durable system-event resources, temporal system versions, and transport subscriptions. They are related capabilities, not interchangeable evidence.

### 6.4 Transport Posture

Jetty WebSocket upgrade code and WebSocket observation tests are concrete. The service supports historical and live flows over that mechanism. The inspected module contains an MQTT class declaration and protocol-neutral request-context scaffolding, but no MQTT adapter implementation was established. Part 3 transport declarations are also outside this study's accepted Part 1/Part 2 baseline. Glaux must advertise a transport class only when the enabled adapter, endpoint discovery, security, error/reconnect semantics, and conformance tests exist.

## 7. OSH Conformance Posture and Standards-Alignment Matrix

### 7.1 Fixed Declaration Assessment

[`ConSysApiService.CONF_CLASSES`](https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/ConSysApiService.java#L74) contains 33 fixed URIs: seven Common, four Features, eleven CSAPI Part 1, nine CSAPI Part 2, and two Part 3 transport claims. The set does not vary with read-only mode, enabled transactional behavior, exposed-resource filters, formats, or transport configuration.

Against accepted IDR-SRV-008, the most material discrepancies are:

- Part 1 and Part 2 `api-common` are absent;
- Part 1 `core` is present although no such approved direct CSAPI class exists;
- Part 1 and Part 2 `advanced-filtering` and `update` are absent;
- Part 2 `feasibility` is absent;
- Part 2 `system-history` is added although it is not in the approved direct class set; and
- Part 3 WebSocket and MQTT claims are mixed into the payload without scoped implementation proof.

No dedicated conformance-handler test, ATS result, implementation-specific OAD parity suite, or schema-conformance suite was found. The payload is therefore a useful checklist of intended capabilities, not evidence-grade conformance reporting.

### 7.2 Implementation Findings Matrix

| Area | OSH source / anchor | Evidence | Observed behavior | CSAPI / Glaux baseline | Alignment | Strength / useful pattern | Gap or risk | Glaux applicability | Test implication | Handoff | Unresolved |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Service composition | `ConSysApiService#doStart`; handler packages | C | Typed read/write stores feed a composed resource-handler graph | Modular implementation; contract parity | Partial/strong | Clean domain and store boundaries | Behavior metadata is distributed | Adopt boundaries with central registry | Registry-to-route parity | 044, 045 | Rust mechanism |
| Read/write federation | `ConSysApiService` lines 125–204; #273 | C/H | Federated/filtered reads plus one write target | Deterministic mutation and errors | Partial | Flexible aggregation | Visible non-owned resources can receive misleading success | Adopt explicit ownership/routing | Cross-store mutation negatives | 025, 043, 047 | Multi-writer policy |
| Part 1 resources | systems/deployment/procedure/property/feature packages | C/T | Broad root and nested resource coverage | IDR-SRV-006/010 | Broad partial | Deep hierarchy and representations | Association round-trip defects | Reuse test scenarios, not outputs | Relationship round trips | 015–019, 053 | Exact deployment fixes |
| Part 2 data | datastream/obs handlers and bindings | C/T | Historical/live observations, schemas, statistics | IDR-SRV-007 | Broad partial | Operational time-series behavior | Latest/filter and nested-array defects | Adopt explicit semantic tests | Temporal/filter/array corpus | 025–028, 034 | Scale thresholds |
| Commands | control/task packages | C/T | Streams, commands, statuses, results | Part 2 command lifecycle | Partial/strong | Useful lifecycle separation | Feasibility results not complete | Adopt explicit state machine | Lifecycle and result tests | 021–024, 035 | Feasibility semantics |
| System events | event handlers | C/I | Nested event-bus subscriptions | Canonical persisted event resource | Divergent/unclear | Event-driven delivery | No complete canonical root/persistence evidence | Separate notification from resource | CRUD/query/replay event tests | 020, 034 | Retention model |
| HTTP methods | `RestApiServlet` | C | GET/POST/PUT/DELETE; no PATCH | Update class requires supported update semantics | Gap | Simple dispatch | No update-class evidence | Implement only with full semantics | PATCH media/precondition matrix | 014, 032 | Patch format |
| Conformance | fixed `CONF_CLASSES` | C/P | 33 static claims | Exact, capability-derived URIs | Divergent | Easy discovery | Invalid/missing/extra and capability-insensitive URIs | Avoid; generate from registry | Exact set under configurations | 050, 051 | None for finding |
| Query vocabulary | parsers; #285 | C/H | Rich filters with aliases/gaps | Canonical parameters plus controlled aliases | Partial | Typed filter builders | Client portability and semantics drift | Canonical first, alias telemetry | Canonical/alias equivalence | 031, 056 | Alias sunset |
| Representations | binding packages | C/T | HTML, JSON, GeoJSON, SensorML, OM, SWE | IDR-SRV-012 media registry | Partial | Broad binding decomposition | SWE CSV/text mismatch; ordered JSON issue | Use registry and permutation tests | Negotiation, order, schema tests | 023, 032, 053 | Extension policy |
| Errors | `ServiceErrors`, servlet/handlers, tests | C/T | Simple status/message JSON; conflicts often 400 | IDR-SRV-013 typed problem model and 409 | Divergent | Central exception mapping | Weak diagnostics and nondeterministic client recovery | Do not copy wire contract | Status/problem/negative suite | 036, 050, 056 | None for baseline |
| Pagination | `BaseResourceHandler`; `ObsHandler` | C | offset/limit; 100 default; 10,000 cap; rescans | Stable bounded paging | Partial | Consistent navigation links | Deep-page cost and mutable-set drift | Prefer stable cursor where allowed | Scale/mutation paging | 027, 054 | Cursor/profile choice |
| OpenAPI/docs | home bindings and osh-docs | C/D | Two mutable upstream `service-desc` links; HTML docs | Complete deployment-specific generated OAD | Divergent | Human-browsable HTML | No capability/runtime parity contract | Follow accepted IDR-SRV-014 | OAD-route-response parity | 044, 046, 050 | Renderer later |
| Streaming | `RestApiServlet`, `ObsHandler`, WS tests | C/T | Jetty WS, replay timing, live event subscription | Dynamic data plus separately proven transport | Partial/strong | Historical/live split and demand flow | Disconnect/load/security depth unclear | Adopt abstraction and backpressure tests | Replay/live/backpressure suite | 028, 035, 054 | Part 3 scope |
| MQTT | fixed class; request scaffolding | C/I | Claim found; adapter not established in scoped module | Claims require enabled proven transport | Unsupported in evidence | Protocol-neutral context is reusable idea | Overclaim/interoperability failure | Never advertise by scaffolding | Endpoint and client E2E | 035, 056 | Possible add-on evidence |
| Security | `ConSysApiSecurity` and permissions | C/T/D | Per-resource/action permissions; optional controls | Later Glaux zero-trust policy | Partial | Fine-grained authorization boundary | Default posture and OAuth client gap | Preserve policy seam | deny-by-default and stream tests | 039, 039A, 040, 055 | Identity design |
| Test posture | 38 test Java files; Gradle config | C/T | 70 active `@Test` annotations; configured build succeeds | Layered requirement traceability and ATS | Partial | Meaningful resource and WS tests | Client tests excluded; no ATS/OAS/conformance tests | Retain depth, add traceability | See Section 13 | 050–056 | Runtime coverage report |

## 8. OSH API Behavior Findings

### 8.1 Landing Page, Collections, and Navigation

The service builds JSON and HTML landing representations and exposes collections and nested resources. Root route families include `systems`, `deployments`, `procedures`, `properties`, `samplingFeatures`, the legacy `fois` alias, `features`, `datastreams`, `observations`, `controlstreams`, `collections`, and `conformance`. System and deployment hierarchies are navigable through nested handlers. The HTML interface is server-rendered and its static Bootstrap assets are now locally packaged, a practical improvement recorded after [issue #256](https://github.com/opensensorhub/osh-core/issues/256).

The landing-page contract is incomplete as a machine-discoverable deployment description. The inspected JSON home builder does not enumerate all control/command paths alongside the other principal resources, and the API-description links point away from the deployment. [Issue #244](https://github.com/opensensorhub/osh-core/issues/244) reports a broken “All Connected Systems” link in collections HTML, while [#248](https://github.com/opensensorhub/osh-core/issues/248) reports browser navigation trouble for GeoJSON-posted systems unless a format parameter is supplied. These are useful reminders that link targets, default representations, and browser/API negotiation must be tested as a graph.

### 8.2 Query, Sorting, Selection, and Pagination

Query parsers support a broad set of identifier, UID, temporal, spatial, parent/member, and resource-specific filters. `select` accepts included fields and `!` exclusions. Newer source supports `order=desc` or `descending`. The breadth is useful, but the canonical-name mismatches in issue #285 and combined-latest defect in #331 show that individual filters do not prove compositional semantics.

Collection paging uses `offset` and `limit`, defaults to 100, retrieves one extra item to decide whether to emit a next link, and hard-caps the page at 10,000. `ConSysApiServiceConfig.maxResponseLimit` defaults to 100,000, but the inspected generic resource path applies its own 10,000 constant; the configuration-to-runtime relationship is therefore unclear. Observation paging comments explicitly state that it rescans from the beginning to `offset + limit + 1`. Deep pages are consequently at least linear in the offset and can drift when results mutate.

Glaux should retain bounded page sizes and first/next/previous navigation, but define stable ordering and snapshot/cursor semantics where the standard allows. It should ensure every advertised configuration value is enforced by the same registry used for documentation.

### 8.3 Content Negotiation and Formats

OSH primarily selects formats through `f`, with legacy `format` support. In automatic mode, an `Accept` header containing `text/html` selects HTML; otherwise JSON is the usual fallback. Specialized bindings select GeoJSON, SensorML, OM, or SWE encodings. This is pragmatic but not full weighted media negotiation across every supported type.

The module enumerates JSON, GeoJSON, SensorML JSON/XML, OM JSON/XML, SWE JSON/XML/binary/CSV, plain text, CSV, and XML forms. Glaux should take the representation breadth and binding separation as evidence, while retaining IDR-SRV-012's canonical media registry, deterministic precedence, `Vary` behavior, 406/415 distinctions, and standard SWE text media type.

### 8.4 Mutation and Error Semantics

Single POST returns 201 with `Location`; batch POST returns 201 with a JSON array of created URIs; PUT and DELETE return 204. `cascade=true` is supported for relevant deletion behavior. A read-only root rejects mutations. These are useful operational conventions.

The error model is much thinner than the accepted Glaux baseline. It generally serializes `status` and `message`, maps unsupported operations to 405, malformed/rejected requests to 400, absent resources to 404, authorization failures to 403, timeout acceptance to 202, and internal failures to 500. No distinct conflict mapping was found. Existing tests expect 400 for duplicate resources and schema-changing datastream updates, where Glaux's accepted policy requires deterministic 409 conflict behavior. An unknown nested-resource name also becomes a 400 rather than a not-found path outcome.

The lesson is not to normalize OSH responses. Glaux should keep its accepted Problem Details-derived taxonomy, field-level validation details, stable error identifiers, conflict distinction, safe messages, correlation context, and content-negotiated error representations.

## 9. OSH Dynamic-Data, Streaming, Status, and Tasking Findings

### 9.1 Historical and Live Observation Delivery

`ObsHandler` separates replay from live behavior. Historical ranges create a replay stream with a default `replaySpeed` of 1.0 and schedule delivery according to observation timing. Requests beginning at “now” subscribe to the event bus. Flow subscribers request work incrementally and write through callbacks, providing a meaningful backpressure seam. Jetty's WebSocket factory upgrades compatible GET requests to a concrete `WebSocketOut` adapter.

This is one of the strongest OSH patterns. Glaux should give historical query/replay, tailing, and live subscription distinct state machines behind a common output abstraction. It should preserve demand-aware delivery rather than buffering without bounds. Later work must define snapshot boundary, ordering, resume, heartbeat, disconnect, slow-consumer, authorization-expiry, and overload semantics.

### 9.2 Resource Events and Status

System, deployment, procedure, datastream, and control-stream event handlers subscribe to OSH resource events. That supports responsive UI/client updates and illustrates reuse of an internal event bus. It does not alone establish the canonical Part 2 system-event resource, persistence, retention, search, or replay behavior. Glaux should persist domain events that are resources while treating transient notifications as delivery projections.

Status and availability can be represented through dynamic streams and command-status resources, but no single OSH behavior should be imported as the Glaux/NATO operational-state model. IDR-SRV-020 retains ownership of controlled status, availability, and event semantics.

### 9.3 Command and Result Flow

OSH separates control-stream definitions, command submission, status, and result access and exposes typed stores for each. This is a sound starting decomposition for Glaux. However, command authorization, idempotency, safety interlocks, cancellation, timeout, result retention, and feasibility need explicit later treatment. The missing feasibility implementation evidence is a release-blocking gap for any future Glaux feasibility claim, not a reason to omit the capability from design research.

### 9.4 Transport Claims

The evidence supports “WebSocket implementation exists and is tested in selected observation cases.” It does not support “all WebSocket transport requirements are conformant,” nor “MQTT is implemented by this module.” Glaux should keep transport profiles separate from resource conformance, publish exact endpoint/security/subprotocol metadata, and run independent client end-to-end tests before adding transport URIs.

## 10. OSH Persistence, Datastore, and Performance Implications

### 10.1 Store Abstractions and Indexes

OSH exposes domain-specific store/filter interfaces instead of making handlers query an engine directly. The H2 module uses MVStore and maintains identifier, UID, spatial R-tree, and full-text indexes. Feature storage includes temporal validity/version logic. Observation and command stores are separate from feature stores while sharing identifiers and filter patterns.

For Glaux, this supports a port/adapter architecture with explicit repositories for features, dynamic records, commands, events, and definitions. It does not select H2, MVStore, or a one-database model. IDR-SRV-025 through IDR-SRV-030 must evaluate Rust-compatible engines, transactions, temporal consistency, retention, and query plans.

### 10.2 Persistence Configuration

OSH documentation says a database module is required for durable persistence; without one, the system-state database may retain only current state/latest observations. No exposed-resource filter means the federated database can make all registered data visible. Configuration includes endpoint path, database ID, exposed resources, formats, security, transactional flag, response limit, live timeout, CURIE mappings, and thread-pool size.

The configuration is expressive, but some names and defaults appear weakly connected to observable behavior: `enableTransactional` was not seen controlling root read-only composition, the configured 100,000 response limit differs from the generic 10,000 cap, and `defaultLiveTimeout` retains legacy SOS-T wording. Glaux should require configuration-schema, startup-validation, runtime-effect, documentation, and test parity for every setting. Unsafe exposure or write-routing combinations should fail closed with diagnostics.

### 10.3 Scale and Consistency Risks

Offset scanning, mutable result sets, event-bus streaming, filtered federation, and independent read/write databases create several performance and consistency questions:

- deep paging cost and duplicate/missing records under concurrent writes;
- index coverage for compound temporal/spatial/resource filters;
- consistency between mutation acknowledgment, store visibility, and event emission;
- bounded queues and slow-client behavior for streams;
- replay scheduling accuracy and resource consumption;
- cross-store referential integrity and cascade behavior; and
- restart recovery for commands, events, and subscriptions.

OSH provides concrete workloads and failure hypotheses, not benchmark thresholds. IDR-SRV-054 should convert them into measured load, stress, longevity, and streaming tests.

## 11. OSH Documentation, OpenAPI, and Examples Findings

### 11.1 Project Documentation

The pinned OSH documentation explains the modular platform, Connected Systems service, database configuration, service and client modules, and JavaScript data-source examples. It is approachable and useful for operators. Its statement that OSH is the “most supported implementation” is a project self-description; this study did not establish a comparative, independent basis for that claim.

Examples are useful as scenario inspiration, but some are deployment-specific and can become stale. The documented public demo was unavailable during all three probes. Current examples and pages should therefore be retained as informative fixtures with provenance, never as golden normative responses.

### 11.2 API Description Behavior

The home representation emits two `service-desc` links directly to mutable upstream OGC development YAML files for Parts 1 and 2 and labels them OpenAPI 3.1. HTML points to the upstream Redoc site. No generated, implementation-specific, capability-filtered OAD was found in the scoped module.

This approach has four material problems relative to accepted IDR-SRV-014:

1. the external files describe standards examples, not the active deployment;
2. separate Part 1 and Part 2 descriptions do not provide one complete deployed contract;
3. mutable upstream URLs are not release pins; and
4. paths, media types, errors, security, read-only state, enabled resources, extensions, and transport behavior can differ from runtime.

IDR-SRV-014 already established that the official example bundles have reference and tooling defects. Linking to those files cannot substitute for generation and bidirectional runtime parity. Glaux should publish its own JSON/YAML `service-desc`, HTML `service-doc`, immutable release artifacts, offline package, hashes, and provenance from the same capability registry.

### 11.3 Documentation Lessons

OSH's navigable HTML and concise setup guidance are worth emulating. Glaux should add explicit profiles, supported-class rationale, canonical parameter/media registries, mutation examples with round-trip verification, streaming/command lifecycle guides, and negative examples. Examples must be validated against schemas and a running release; links must be crawled in both JSON and HTML outputs.

## 12. Interoperability and Client-Compatibility Findings

### 12.1 Compatibility Strengths

Broad JSON/GeoJSON/SensorML/SWE support, HTML fallback, nested links, `f` selection, WebSocket observations, and client helper packages increase practical interoperability. Legacy aliases such as `fois` can ease migration when they are explicitly documented and tested. The protocol-neutral `RequestContext` and binding abstractions are also useful design patterns.

### 12.2 Compatibility Risks

The largest risks are false capability discovery and successful-but-lossy writes. A client that trusts the conformance payload may attempt unsupported update, feasibility, MQTT, system-event, or filtering behavior. A client that trusts 201/204 may later discover dropped deployment or sampling-feature relationships. Canonical parameter mismatches, ordered JSON parsing, incomplete OADs, format-dependent browser navigation, simple error payloads, and 400-for-conflict behavior all make portable recovery harder.

The OSH client tests under `sensorhub-service-consys` are excluded by the module's Gradle configuration, so their presence does not establish release-gated server/client compatibility. The public demo outage also prevented an independent CSAPI Explorer or generic-client pass. These limitations must remain visible in comparisons with later implementations.

### 12.3 Derived Smoke-Test Set

At minimum, an external client should:

1. discover links and exact capabilities from a fresh deployment;
2. compare conformance, OAD, routes, methods, media, and policy configuration;
3. create each resource with every relationship field, retrieve it, traverse both directions, restart, and retrieve again;
4. send canonical and compatibility query names individually and in combinations;
5. permute JSON property order and optional members;
6. distinguish validation, not-found, conflict, authorization, unsupported-media, and unacceptable-representation errors;
7. page a changing collection and a deep static collection;
8. run historical, replay, live, disconnect/reconnect, and slow-consumer streams;
9. execute command acceptance, status, result, failure, timeout, and feasibility flows; and
10. validate OAuth/authentication and authorization behavior independently of the server's own client.

## 13. Test-Strategy Implications

### 13.1 Existing Test Evidence

The dedicated module contains 38 test Java files and 70 active `@Test` annotations by source inventory. Coverage includes generic feature CRUD/filter behavior, SensorML features, access control, systems, sampling features, datastreams, observations, control streams, WebSocket observation behavior, and client helpers. The system tests cover hierarchies and cascade behavior; datastream tests cover schema and update constraints; observation tests cover query behavior; WebSocket tests cover historical and unbounded temporal forms.

The exact pinned commit passed OSH's configured upstream `build` job. Local execution was attempted with `gradlew :sensorhub-service-consys:test --no-daemon` but could not start because Java was unavailable. Source counts are not runtime coverage counts, inherited tests may alter execution totals, and ten client-side test annotations appear in a package explicitly excluded by the Gradle test task.

### 13.2 Material Gaps

No dedicated tests were found for the conformance payload, complete OAD/runtime parity, formal ATS execution, schema validation across all bindings, feasibility, canonical persisted system events, MQTT, or all negative error permutations. The specialized deployed-systems test is commented out. Open issues identify missing cases for relationship persistence, canonical parameters, unordered JSON, latest-time combinations, and nested arrays. The configured build therefore demonstrates project health, not standards conformance.

### 13.3 Glaux Test Layers Derived from OSH

| Layer | Required Glaux tests inspired by this study | Why OSH evidence matters |
|---|---|---|
| Unit/property | Query parser canonical/alias properties; JSON order permutations; time interval algebra; media negotiation; error mapping | Defects cluster in combinations and parser assumptions |
| Store contract | CRUD/version/relationship round trips across every adapter; conflict and cascade; temporal/spatial index equivalence | Federation and dropped links show store/API seams are critical |
| Handler/component | Capability-conditioned routes/methods; read-only behavior; selection/paging links; all format bindings | OSH's handler modularity is strong but metadata can drift |
| Contract | Exact conformance URI set; OAD closure and bidirectional runtime parity; schema/example validation | Fixed claims and upstream example links are insufficient |
| End-to-end | External generic client; browser links; auth; deployment and sampling-feature associations; command lifecycle | Server-internal tests can miss client portability |
| Streaming | Historical/live boundary, replay timing, backpressure, slow consumer, reconnect, cancellation, auth expiry | OSH provides real asynchronous scenarios but limited edge coverage |
| Conformance | Requirement-to-test ledger and approved ATS, with deviations recorded | Test quantity cannot substitute for class traceability |
| Performance | Deep paging, compound filters, concurrent writes, event fan-out, replay load, persistence restart | Offset scans/federation expose realistic scale risks |
| Regression corpus | OSH issue-derived negative fixtures, clearly labeled informative | Tracker cases are high-value failure hypotheses |

Tests should distinguish **meaningful** behavior checks, **deep** edge/combination checks, and **end-to-end** external-client checks, following the reviewed OS4CSAPI testing-playbook exemplar. A route-smoke test alone is not a class test; a serializer unit test alone is not round-trip integrity.

## 14. Lessons for Glaux Server

### 14.1 Adopt as Patterns

- typed domain-store/filter interfaces behind API handlers;
- modular handler graphs for root and nested resources;
- separate representation bindings rather than format conditionals spread through routes;
- explicit historical replay versus live-event paths;
- demand-aware asynchronous streaming;
- per-resource and per-operation authorization hooks;
- durable-resource stores separated from observation and command record stores; and
- realistic resource hierarchy, time-series, and WebSocket tests.

### 14.2 Improve Before Adoption

- replace fixed capability claims with registry-derived declarations;
- make read/write ownership explicit and reject ambiguous mutations;
- guarantee every accepted field/link round-trips or reject it;
- make canonical parameter names primary and aliases controlled extensions;
- use stable, typed, content-negotiated errors and correct conflict statuses;
- connect every configuration property to validation, documentation, and tests;
- make pagination ordering and scale semantics explicit;
- publish one complete implementation-specific OAD; and
- distinguish durable system events, version history, notifications, and transports.

### 14.3 Avoid Copying

- Java/OSGi lifecycle and module-registry mechanics;
- OSH identifiers or federated-database assumptions as Glaux domain requirements;
- fixed conformance lists;
- upstream example OAD links as deployment documentation;
- successful status codes without persistence/relationship verification;
- parser dependence on JSON member order;
- 400 as a catch-all mutation failure; and
- declarations for absent, disabled, experimental, or externally assumed transports.

## 15. Downstream Topic Handoff Matrix

| Finding / artifact | Downstream owner(s) | Required use | Boundary preserved |
|---|---|---|---|
| OSH comparison rubric and evidence labels | 014B–014G | Apply consistent supported/partial/divergent/unclear classifications | Do not rank implementations solely by route count |
| Resource hierarchy and round-trip failures | 015–019, 032, 051, 053 | Define relationships and fixtures with create/read/traverse/restart checks | Those topics own final model and schema |
| System event/history distinction | 020, 034 | Define persistent event resource versus notification/version history | 014A makes no final event-vocabulary decision |
| Command/status/result/feasibility gaps | 021–024, 035, 055 | Define lifecycle, safety, auth, and failure tests | Later topics own tasking policy |
| Read/write federation and ownership | 025, 029, 043, 047 | Evaluate transaction, synchronization, and safe configuration models | No datastore selected here |
| Offset paging, indexing, streaming load | 027, 028, 054 | Create benchmarks and overload/consistency scenarios | No performance threshold set here |
| Query aliases and representation behavior | 031, 032, 056 | Test canonical vocabulary, extensions, negotiation, and clients | Accepted 011/012 remain controlling |
| Error/status deviations | 036, 050, 056 | Exercise accepted Glaux taxonomy against external clients | Accepted 013 remains controlling |
| Capability registry and handler boundaries | 044, 045 | Evaluate Rust architecture that generates/verifies routes and contracts | No Rust framework chosen here |
| Deployment OAD and demo availability | 046, 048, 049 | Package offline docs, health, reproducibility, and release artifacts | Accepted 014 remains controlling |
| Conformance and test gaps | 050–055 | Add exact class, ATS, traceability, fixtures, streaming, security tests | OSH results are informative fixtures only |
| External-client risks | 056 | Build cross-implementation discovery/write/query/stream matrix | No client-specific workaround becomes server policy automatically |

## 16. Recommendations

These recommendations become planning guidance only after plan-owner acceptance of this report.

1. **R-014A-001 — Build Glaux around typed capability and contract registries.** Routes, methods, stores, formats, conformance classes, OAD operations, authorization actions, and tests should be generated from or verified against the same metadata. Priority: High.
2. **R-014A-002 — Preserve modular handler, store, and binding seams.** Use Rust traits and explicit domain services rather than reproducing OSH's OSGi mechanics. Priority: High.
3. **R-014A-003 — Make mutation ownership deterministic.** A federated or synchronized resource must resolve to a writable authority before a request is accepted; ambiguous writes fail without side effects. Priority: High.
4. **R-014A-004 — Treat round-trip integrity as a release gate.** Every accepted property, association, link, encoding, and nested object must survive create/replace/update, retrieval, traversal, and restart or be rejected before success. Priority: High.
5. **R-014A-005 — Derive exact conformance from enabled, tested capabilities.** Never use a fixed aspirational list; validate URIs and their prerequisite graph and exclude read/write/transport classes when configuration cannot satisfy them. Priority: High.
6. **R-014A-006 — Keep canonical protocol vocabulary primary.** Compatibility aliases may be supported only as documented extensions with equivalence tests, telemetry, and lifecycle policy. Priority: High.
7. **R-014A-007 — Preserve the accepted Glaux negotiation and error baselines.** Do not inherit OSH's JSON fallback, SWE CSV substitution, two-field error object, or 400 catch-all conflict behavior. Priority: High.
8. **R-014A-008 — Separate durable events, history, notifications, and transports.** Each gets its own storage, API, discovery, retention, security, and conformance evidence. Priority: High.
9. **R-014A-009 — Adopt historical/live streaming separation and explicit backpressure.** Add resume, ordering, disconnect, overload, cancellation, and authorization-expiry semantics before release. Priority: High.
10. **R-014A-010 — Publish a generated deployment-specific OAD and self-hosted documentation.** Upstream standard examples remain provenance and negative fixtures, not the deployed contract. Priority: High.
11. **R-014A-011 — Turn OSH issue cases into labeled regression hypotheses.** Include relationship loss, query aliases, unordered JSON, nested arrays, latest-time combinations, browser links, and read/write routing without treating expected OSH output as normative. Priority: Medium.
12. **R-014A-012 — Require external-client, ATS, OAD, schema, security, and streaming gates in addition to module tests.** Configured build success is necessary but not sufficient evidence. Priority: High.
13. **R-014A-013 — Validate configuration-to-runtime parity.** Every setting must have schema constraints, startup diagnostics, runtime effect, OAD/conformance projection where applicable, and tests. Priority: High.
14. **R-014A-014 — Retain source provenance and license boundaries.** Reimplement patterns independently; review MPL-2.0 obligations before any direct code reuse. Priority: Medium.

## 17. Risks, Constraints, and Open Questions

### 17.1 Evidence Risks and Constraints

- **No positive live-demo evidence.** The documented demo returned 502, so code, tests, documentation, and tracker evidence carry the study. This limits claims about a current public deployment.
- **No local Java execution.** The exact pinned upstream build succeeded, but local environment limitations prevented independent execution or targeted issue reproduction.
- **Mutable tracker evidence.** Open issues can be fixed, reclassified, or contradicted. Each cited state is dated August 31, 2026 and must be refreshed before implementation decisions rely on current status.
- **Scoped module boundary.** MQTT or other behavior may exist in proprietary deployments or add-on repositories not established here. The conclusion is absence of evidence in scoped sources, not ecosystem-wide nonexistence.
- **Source annotation counts are not runtime coverage.** Inheritance, parameterization, exclusions, and task configuration affect executed tests.
- **Tag/release distinction.** `v2.0.2` is a lightweight tag and current code baseline, but not a published GitHub Release with release notes/assets.
- **Implementation inference.** Route/code existence cannot prove all ATS requirements, security behavior, concurrency semantics, or production scale.

### 17.2 Open Questions Assigned Forward

| Open question | Why unresolved here | Owner / resolution evidence |
|---|---|---|
| Which Rust web and module framework best preserves the useful seams? | Framework selection is out of scope | 044/045 benchmarks and prototype |
| Which datastore/index plan meets compound spatial-temporal and replay workloads? | No comparative performance work was authorized | 025–030 and 054 measurements |
| Will Glaux support offset only, cursor paging, or both? | Requires standards/profile and storage analysis | 027/031 decision with mutation tests |
| What is the canonical durable system-event and history model? | OSH evidence reveals ambiguity but is not authoritative | 020/034 model and requirement traceability |
| Which transport profiles will Glaux claim? | Part 3 and deployment needs remain separate | 035 and 056 adapter/E2E evidence |
| What feasibility lifecycle is required by the operational profile? | OSH does not answer the normative/project question | 021–024 analysis and tests |
| What compatibility aliases are justified? | Needs cross-implementation/client evidence | 014B–014G and 056 comparison |
| What performance and queue limits are acceptable? | Source reveals risks, not requirements | 048/054 operational SLO research |
| Have cited OSH defects changed after the pin? | Upstream is mutable | Refresh issues/tag before implementation and final synthesis |

These questions do not block acceptance of the implementation findings baseline. They are intentionally routed to the topics that own the decisions.

## 18. Validation Against the Research Plan

### 18.1 Success Criteria

| Topic-plan success criterion | Status | Report evidence |
|---|---|---|
| Exact OSH sources, versions, URLs, commits, tags, and dates | Met | Section 3; Appendix B |
| Server, client, documentation, demo, issue, and inferred evidence distinguished | Met | Reading Guide; Sections 3, 12, and 17 |
| Architecture and module boundaries summarized | Met | Section 4 |
| Part 1 and Part 2 compared with Glaux baseline | Met | Sections 5–7 |
| Conformance assessed without assumption | Met | Section 7 |
| Entry point, navigation, query, representations, errors, OAD, dynamic data, streaming, status, and tasking assessed | Met | Sections 8–11 |
| Strengths, gaps, risks, and OSH-specific assumptions identified | Met | Matrix 7.2; Sections 14 and 17 |
| Design, validation, testing, and interoperability lessons documented | Met | Sections 12–16 |
| Downstream handoffs explicit | Met | Section 15 |
| References explicit and reproducible | Met | Sections 3 and 19; Appendix B |

### 18.2 Required Content Audit

| Required content | Status | Location |
|---|---|---|
| 1. Executive summary | Present | Section 1 |
| 2. Scope and plan alignment | Present | Section 2 |
| 3. Source inventory and classification | Present | Section 3 |
| 4. Architecture and modules | Present | Section 4 |
| 5. Part 1 findings | Present | Section 5 |
| 6. Part 2 findings | Present | Section 6 |
| 7. Conformance and alignment matrix | Present | Section 7 |
| 8. API behavior | Present | Section 8 |
| 9. Dynamic data, streaming, status, tasking | Present | Section 9 |
| 10. Persistence and performance | Present | Section 10 |
| 11. Documentation, OpenAPI, examples | Present | Section 11 |
| 12. Interoperability and client compatibility | Present | Section 12 |
| 13. Test implications | Present | Section 13 |
| 14. Glaux lessons | Present | Section 14 |
| 15. Handoff matrix | Present | Section 15 |
| 16. Recommendations | Present | Section 16 |
| 17. Risks and open questions | Present | Section 17 |
| 18. Plan validation | Present | This section |
| 19. References | Present | Section 19 |

The implementation-findings matrix in Section 7.2 includes all required fields: area, source anchor, evidence type, observed behavior, related baseline, alignment, strength, gap/risk, applicability, test implication, downstream handoff, and unresolved status.

## 19. References

### 19.1 Controlling Standards and Glaux Baseline

- Open Geospatial Consortium, [OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html), approved 2025-07-16; accessed 2026-08-31.
- Open Geospatial Consortium, [OGC 23-002, *OGC API - Connected Systems - Part 2: Dynamic Data*, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html), approved 2025-07-16; accessed 2026-08-31.
- Open Geospatial Consortium, [OGC 17-069r4, *OGC API - Features - Part 1: Core*](https://docs.ogc.org/is/17-069r4/17-069r4.html); accessed 2026-08-31.
- Open Geospatial Consortium, [OGC 23-000, *SensorML Encoding Standard*, Version 3.0](https://docs.ogc.org/is/23-000/23-000.html); accessed 2026-08-31.
- Open Geospatial Consortium, [OGC 24-014, *SWE Common Data Model Encoding Standard*, Version 3.0.0](https://docs.ogc.org/is/24-014/24-014.html); accessed 2026-08-31.
- Glaux, [accepted IDR-SRV-006 through IDR-SRV-014 reports](./), current through 2026-08-31.
- Glaux, [shared OGC Connected Systems upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), Version 1.8.

### 19.2 OSH Code, Tests, and Documentation

- OpenSensorHub, [OSH Core `v2.0.2`](https://github.com/opensensorhub/osh-core/tree/235c0eabf24b6d6137b499b4402943d2794b70e6), commit `235c0eabf24b6d6137b499b4402943d2794b70e6`.
- OpenSensorHub, [`sensorhub-service-consys`](https://github.com/opensensorhub/osh-core/tree/235c0eabf24b6d6137b499b4402943d2794b70e6/sensorhub-service-consys), pinned module source and tests.
- OpenSensorHub, [`ConSysApiService.java`](https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/ConSysApiService.java), blob `2671639ed4fbf698b4bfcb866ddca00448a5ee0c`.
- OpenSensorHub, [`ConSysApiServiceConfig.java`](https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/ConSysApiServiceConfig.java), blob `9f1fee0cf00bd279f836b4fbedd57fb724528bab`.
- OpenSensorHub, [`RestApiServlet.java`](https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/RestApiServlet.java), blob `e682d14217a50aa45c1ad61e3d950de1f4c0265b`.
- OpenSensorHub, [`ObsHandler.java`](https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/obs/ObsHandler.java), blob `a696407e1ee7bde8b89e2aebf66b02baca5eeea5`.
- OpenSensorHub, [module Gradle build](https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/sensorhub-service-consys/build.gradle), blob `a9fbdf7c018ace4f26bf3cbc8ee263d9436ed8c0`.
- OpenSensorHub, [successful configured build for pinned commit](https://github.com/opensensorhub/osh-core/actions/runs/30473581984/job/90649579129), 2026-07-29.
- OpenSensorHub, [OSH Docs pinned commit](https://github.com/opensensorhub/osh-docs/tree/ec51a67af82c8e0bb309cfe1975cd17c44220e77), 2026-06-24.
- OpenSensorHub, [Connected Systems documentation](https://github.com/opensensorhub/osh-docs/blob/ec51a67af82c8e0bb309cfe1975cd17c44220e77/docs/osh-connect/connected-systems.md), blob `6a48d8e9a258640fb4285ca200e42b6d23cf3c27`.
- OpenSensorHub, [service-modules documentation](https://github.com/opensensorhub/osh-docs/blob/ec51a67af82c8e0bb309cfe1975cd17c44220e77/docs/osh-node/user-docs/service-modules.md), blob `12f4e35c469e9a1314ff5086e247235dfc2db9bf`.
- OpenSensorHub, [client-modules documentation](https://github.com/opensensorhub/osh-docs/blob/ec51a67af82c8e0bb309cfe1975cd17c44220e77/docs/osh-node/user-docs/client-modules.md), blob `4a2a761ebbfd36419eda3097c622bf12179310d4`.
- OpenSensorHub, [Connected Systems data-source reference](https://github.com/opensensorhub/osh-docs/blob/ec51a67af82c8e0bb309cfe1975cd17c44220e77/docs/osh-connect/osh-js/reference/datasources/consysapi.md), blob `d2533e7138186f580703fd1a110ba18451878537`.
- OpenSensorHub, [MPL-2.0 license](https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/LICENSE).

### 19.3 OSH Current Issue Evidence

- [#244 — incorrect collection HTML link](https://github.com/opensensorhub/osh-core/issues/244), open-state evidence checked 2026-08-31.
- [#248 — GeoJSON system browser-navigation format issue](https://github.com/opensensorhub/osh-core/issues/248), open-state evidence checked 2026-08-31.
- [#249 — legacy features-of-interest naming](https://github.com/opensensorhub/osh-core/issues/249), open-state evidence checked 2026-08-31.
- [#273 — read/write database routing and silent update risk](https://github.com/opensensorhub/osh-core/issues/273), open-state evidence checked 2026-08-31.
- [#275 — ordered JSON parser behavior](https://github.com/opensensorhub/osh-core/issues/275), open-state evidence checked 2026-08-31.
- [#284 — nested variable-size DataArray insertion](https://github.com/opensensorhub/osh-core/issues/284), open-state evidence checked 2026-08-31.
- [#285 — CSAPI query parameter coverage and aliases](https://github.com/opensensorhub/osh-core/issues/285), open-state evidence checked 2026-08-31.
- [#301 — CSAPI client OAuth/OIDC support](https://github.com/opensensorhub/osh-core/issues/301), open-state evidence checked 2026-08-31.
- [#331 — combined latest-time observation query](https://github.com/opensensorhub/osh-core/issues/331), open-state evidence checked 2026-08-31.
- [#337 — deployment association round-trip and scoped endpoints](https://github.com/opensensorhub/osh-core/issues/337), open-state evidence checked 2026-08-31.
- [#339 — observation sampling-feature association round trip](https://github.com/opensensorhub/osh-core/issues/339), open-state evidence checked 2026-08-31.

### 19.4 Research Method Exemplars

- OS4CSAPI, [`01-pr114-blueprint-analysis.md`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md).
- OS4CSAPI, [`15-fixture-sourcing-organization.md`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/15-fixture-sourcing-organization.md).
- OS4CSAPI, [`38-testing-playbook-synthesis.md`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/38-testing-playbook-synthesis.md).

## Appendix A. Detailed Question Ledger

| ID | Detailed plan question (short form) | Resolution | Primary report evidence |
|---|---|---|---|
| Q1 | Relevant repositories, docs, examples, demos | Complete | Section 3.1 |
| Q2 | Version/branch/commit/release studied | Complete | Section 3.1; Appendix B |
| Q3 | Server/client/docs/demo separation | Complete | Sections 3.1 and 12 |
| Q4 | License and reuse constraints | Complete | Sections 3.1 and 16 |
| Q5 | Active issues/PRs/discussions | Complete within bounded relevance | Sections 5–12 and 19.3 |
| Q6 | Architectural structure | Complete | Section 4.1 |
| Q7 | API/persistence/integration/stream/command/client modules | Complete | Sections 4 and 9–10 |
| Q8 | Internal resource representation | Complete at architecture-relevant level | Sections 4.1–4.2 and 10.1 |
| Q9 | Patterns relevant to Glaux boundaries | Complete | Sections 4 and 14.1 |
| Q10 | Java/OSH-specific patterns not to copy | Complete | Sections 4.1 and 14.3 |
| Q11 | Part 1 resources and behaviors | Complete | Section 5 |
| Q12 | Part 2 resources and behaviors | Complete | Section 6 |
| Q13 | Claimed/implied/unsupported/unclear classes | Complete | Sections 6.3–6.4 and 7.1 |
| Q14 | Conformance/OAD/media/link/schema exposure | Complete | Sections 7, 8, and 11 |
| Q15 | Gaps against Glaux baseline | Complete | Section 7.2 |
| Q16 | Evidence for alignment/gaps | Complete | Sections 3 and 7.2 |
| Q17 | Landing/OAD/conformance/collections/items/links | Complete | Sections 8.1 and 11.2 |
| Q18 | URIs/relations/nesting/navigation | Complete | Sections 5.1 and 8.1 |
| Q19 | Query/filter/sort/page/select | Complete | Sections 5.3 and 8.2 |
| Q20 | Negotiation/media types | Complete | Section 8.3 |
| Q21 | Error behavior | Complete | Section 8.4 |
| Q22 | API-topic implications | Complete | Sections 14–16 |
| Q23 | Datastreams/observations | Complete | Section 6.1 |
| Q24 | Real-time/historical/replay/subscription | Complete | Sections 9.1 and 9.4 |
| Q25 | Status/events/dynamic properties | Complete | Sections 6.3 and 9.2 |
| Q26 | Command/tasking/feasibility/actuators | Complete | Sections 6.2 and 9.3 |
| Q27 | Relevant dynamic/tasking patterns | Complete | Sections 9 and 14.1 |
| Q28 | OSH assumptions to separate | Complete | Sections 9.2–9.4 and 14.3 |
| Q29 | Datastore/persistence abstractions | Complete | Sections 4.1, 4.3, and 10.1 |
| Q30 | Feature/observation/command/event/time-series model | Complete at study scope | Sections 4.1 and 10.1 |
| Q31 | Index/query patterns | Complete | Sections 10.1 and 10.3 |
| Q32 | Performance/scale/stream constraints | Complete within available evidence | Sections 8.2, 9.1, and 10.3 |
| Q33 | Category E handoffs | Complete | Section 15 |
| Q34 | Endpoints/examples used by clients/tools | Complete within available evidence | Sections 3.1, 11, and 12 |
| Q35 | Important client compatibility patterns | Complete | Section 12.1 |
| Q36 | Implementation-specific interop risks | Complete | Section 12.2 |
| Q37 | Derived smoke tests | Complete | Section 12.3 |
| Q38 | IDR-SRV-056 findings | Complete | Sections 12 and 15 |
| Q39 | Available tests/examples/fixtures/snippets | Complete | Sections 11 and 13.1 |
| Q40 | Positive/negative/schema/conformance/stream/command tests | Complete | Sections 12.3 and 13.3 |
| Q41 | Comparison data versus normative truth | Complete | Reading Guide; Sections 3.2 and 13.3 |
| Q42 | Handoffs to 050/051/053 | Complete | Sections 13.3 and 15 |

## Appendix B. Reproducible Study Record

### B.1 Immutable Pins and Inventory

| Item | Value |
|---|---|
| Research date | 2026-08-31 |
| OSH Core tag / commit | `v2.0.2` / `235c0eabf24b6d6137b499b4402943d2794b70e6` |
| OSH Core tree | `123f77e7e7c5a84764ff3c60268e585e2212cc0b` |
| OSH Docs commit / tree | `ec51a67af82c8e0bb309cfe1975cd17c44220e77` / `059c5f124d4ffc5e0a07f3c0b23eed37df9a70f7` |
| OS4CSAPI exemplar commit / tree | `754411897173c2ec4debaa9bcf4ed9e0f8a9e230` / `a30dcddc42f12ff78e368672698dbdcf7fa7720b` |
| CSAPI module inventory | 202 files: 157 production Java, 38 test Java |
| Test-source inventory | 70 active `@Test` annotations; client test package excluded by module Gradle configuration |
| Fixed conformance inventory | 33 unique URIs |
| Local execution | Attempted; unavailable because Java and `JAVA_HOME` were absent |
| Upstream execution | Exact commit's configured `build` job succeeded 2026-07-29 |
| Live demo probes | Root, `/conformance`, `/systems`: HTTP 502 on 2026-08-31 |

### B.2 Study Procedure

1. Read the topic plan and all three required OS4CSAPI exemplars completely.
2. Confirm accepted IDR-SRV-006 through IDR-SRV-014 as the comparison baseline.
3. Clone OSH Core, OSH Docs, and the exemplar repository; record exact commits and trees.
4. Inventory the CSAPI module's production code, tests, packages, build configuration, routes, handlers, formats, and conformance constants.
5. Trace service startup from configured databases through `HandlerContext`, root composition, resource handlers, bindings, streaming, and permissions.
6. Inspect Part 1/Part 2 resources, query parsers, representations, mutations, errors, paging, streaming, commands, events, persistence, and documentation paths.
7. Count fixed conformance URIs and compare each CSAPI claim with accepted IDR-SRV-008.
8. Inspect module tests and exclusions; attempt the local task; verify exact-commit upstream build evidence.
9. Review bounded, directly relevant current issues and corroborate with code where feasible.
10. Probe the documentation's current public demo at root and selected resource paths.
11. Classify each conclusion by evidence strength, compare it with the accepted Glaux baseline, and route decisions to owning topics.

### B.3 Limitations Record

The reproducible pins permit later code comparison even if upstream changes. The demo's 502 response is availability evidence only and must not be interpreted as normal OSH API behavior. The upstream build uses project-selected tasks and exclusions. No claim in this report converts source inventory or issue reports into formal conformance results.

## Appendix C. Completion and Handoff

- [x] Exactly one research topic executed: IDR-SRV-014A.
- [x] All five core and forty-two detailed questions addressed.
- [x] All six methodology phases and ten success criteria satisfied.
- [x] All nineteen required content areas and twelve matrix fields present.
- [x] Commit-pinned code/docs/test study, bounded issue review, demo probes, and upstream-build verification completed.
- [x] Findings reviewed for authority, scope, reproducibility, applicability, and downstream ownership.
- [x] Plan-owner acceptance of IDR-SRV-014A — accepted by the Glaux Project Lead on August 31, 2026.

**Research execution completed:** August 31, 2026<br>
**Plan-owner acceptance:** August 31, 2026<br>
**Next authorized topic:** IDR-SRV-014B — Connected Systems Go CSAPI Server Implementation Study

The Glaux Project Lead accepted IDR-SRV-014A and authorized execution of exactly one next research topic, IDR-SRV-014B, on August 31, 2026. This records the completed transition; IDR-SRV-014B remains governed by its own report and acceptance gate.

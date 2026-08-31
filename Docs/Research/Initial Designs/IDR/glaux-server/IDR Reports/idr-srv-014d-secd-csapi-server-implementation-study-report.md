# Section 014D: SECD CSAPI Server Implementation Study - Research Report

**Topic ID:** IDR-SRV-014D<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-014D SECD CSAPI Server Implementation Study](../IDR%20Plans/idr-srv-014d-secd-csapi-server-implementation-study.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 41 detailed questions; all six methodology phases, eleven success criteria, nineteen required content areas, and twelve minimum implementation-findings fields are validated<br>
**Methodology Used:** Evidence inventory; pinned repository and maintainer-artifact review; dated read-only live API, OpenAPI, TLS, route, representation, negotiation, query, error, and drift probes; comparison against accepted IDR-SRV-006 through IDR-SRV-014C; bounded synthesis<br>
**Research Time:** Approximately 4.5 hours of AI-assisted elapsed execution on August 31, 2026<br>
**Primary Sources:** Live SECD deployment and OpenAPI description retrieved August 31, 2026; maintainer correspondence and interoperability corpus pinned at commit f018fd129bf0d0d1ce75e68198e3ab4d99d937a0; OGC API - Connected Systems Parts 1 and 2<br>
**Supporting Resources:** Accepted Glaux reports IDR-SRV-006 through IDR-SRV-014C; private SECD interoperability evidence repository metadata and captures; official OGC schema artifacts<br>
**Document Purpose:** Establish an evidence-bounded SECD implementation-behavior baseline for Glaux architecture, resource modeling, discovery, navigation, representations, dynamic data, tasking, validation, documentation, and test design without inferring unavailable server internals or preempting IDR-SRV-014F interoperability adjudication<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** August 31, 2026<br>
**Date:** August 31, 2026<br>
**Last Updated:** August 31, 2026

---

## Reading Guide

Evidence labels are mandatory because the SECD server source is not available:

| Label | Meaning |
|---|---|
| N | Normative OGC standard or accepted Glaux standards-baseline report |
| L | Dated live API observation from August 31, 2026 |
| O | Dated live OpenAPI observation from August 31, 2026 |
| M | Maintainer statement preserved at the pinned interoperability commit |
| I | Interoperability repository evidence at the pinned commit |
| H | Historical API capture or test observation at the pinned commit |
| X | Analyst inference or Glaux recommendation, never a standards claim |

“Implemented” means a behavior was observed or declared in the dated deployment evidence. It does not mean every requirement of a conformance class passes. “Claimed” means a URI appears in the live conformance response. Historical write results are not silently promoted to current behavior. Internal routing, modules, persistence, deployment, validation logic, and test structure are marked not verifiable because no actual server source or maintainer architecture artifact was found.

The private csapi-server-interop-secd repository is supporting interoperability evidence, not SECD server source. Detailed client/root-cause and cross-implementation adjudication remains owned by IDR-SRV-014F.

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. SECD Source Inventory and Evidence Classification
4. SECD Observable API Topology and Explicitly Declared Architecture
5. SECD CSAPI Part 1 Behavior Findings
6. SECD CSAPI Part 2 Behavior Findings
7. SECD Conformance Posture and Standards-Alignment Matrix
8. SECD API Behavior Findings
9. SECD Dynamic-Data, Status, and Tasking Findings
10. SECD Data-Model, Persistence, and Fixture Implications
11. SECD Documentation, OpenAPI, and Examples Findings
12. Interoperability Questions and IDR-SRV-014F Handoff
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

SECD is a live, drone-oriented CSAPI prototype introduced by Andreas Matheus of Secure Dimensions, with Lucio Colaiacomo as a contributor and In-Platform Tasking as an intended use case [M]. On August 31, 2026 the deployment exposed a meaningful Part 1 and Part 2 surface: Systems, subsystems, Deployments, Procedures, Sampling Features, DataStreams, Observations, ControlStreams, Commands, schemas, system history, spatial extensions, pagination, an OpenAPI 3.1 document, and an SSE live-observation operation [L,O]. Its current corpus contained 8 Systems, 4 Deployments, 5 Procedures, 7 Sampling Features, 13 DataStreams, 11,250,822 Observations, 2 ControlStreams, and 15 Commands. These counts describe one mutable deployment, not implementation limits.

The most important boundary is evidentiary. No actual SECD server source, public release, tag, license, build, test suite, deployment manifest, or maintainer architecture document was found. The maintainer introduction called the endpoint an unprotected starting prototype and described an open-source intent, while the access-controlled repository contains observations, captures, evaluation evidence, and report cards rather than server code [M,I]. Internal module boundaries, persistence, transactions, migrations, concurrency, validation mechanics, configuration, secrets, and deployment topology are therefore not verifiable. HTTP banners show nginx and Express involvement, and a historical failure exposed a PostgreSQL-style constraint message, but neither is sufficient authority for an internal architecture claim [L,H].

SECD demonstrates useful design ideas: realistic UAV resource relationships; rich, domain-specific fixtures; UUID-backed identifiers plus URI-like UIDs; embedded SWE-like DataRecord and DataChoice schemas; historical and live observation access; direct plus collection-oriented discovery; status-bearing Commands; strict limit validation; and compact JSON error objects [L,O]. These are valuable test-corpus and product-behavior inputs.

The deployment is not a safe behavioral template. Its conformance document uses implementation-era class names rather than the approved Part 1 and Part 2 class URIs selected by IDR-SRV-008, and it claims system-events while the canonical system-events endpoint is absent [L,N]. Four collection records cover only Systems, Deployments, Procedures, and Sampling Features. Property Definitions, subdeployments, canonical System Events, feasibility, and richer command result/status routes are absent or unclear. Item links reuse rel=alternate for different resource relationships. Some collections use GeoJSON FeatureCollection while Procedures and Part 2 resources use resource-named envelopes. Canonical query names such as q and parent were accepted but ignored in bounded probes, whereas parentId worked. Accept and f negotiation were also ignored: even SensorML and invalid media requests returned ordinary application/json with HTTP 200 [L].

The live deployment has improved since the May 2026 evidence capture. The previously missing /collections bootstrap now returns four collections, top-level /observations and /commands now return data, the conformance payload grew, and the OpenAPI description covers a wider write and spatial surface [L,H]. Other historical findings remain observable, including noncanonical conformance labels, alternate link relations, resource-named envelopes, missing Property Definitions, proprietary query names, and false-success negotiation. Historical mutation findings were not rerun because this topic used read-only probes; they remain historical inputs for IDR-SRV-014F rather than current implementation claims.

Glaux should adopt the use-case richness and explicit schemas, but implement them behind one typed capability contract. That contract must generate or verify routes, links, canonical query names, representations, schemas, conformance declarations, OpenAPI, authorization, persistence, and tests. Glaux must reject silent query/negotiation fallback, read/write shape asymmetry, static aspirational conformance, undocumented runtime routes, and mutable live deployments as the only regression oracle.

---

## 2. Scope and Plan Alignment

### 2.1 Included and Excluded Scope

Included:

- Current observable API topology, declared OpenAPI, conformance payload, resource shapes, links, queries, representations, errors, dynamic data, system history, tasking, and spatial extensions.
- Maintainer-declared purpose and maturity.
- Historical SECD captures and findings only where clearly labeled and useful for drift or test derivation.
- Comparison with the accepted Glaux standards and implementation-study baseline.
- Black-box lessons, test implications, downstream handoffs, and bounded recommendations.

Excluded:

- Inference about unavailable server modules, handlers, persistence architecture, transactions, migrations, configuration, deployment, and tests.
- New mutation, authentication-bypass, load, long-lived streaming, or destructive probes.
- Re-scoring the SECD report card.
- Cross-server root-cause adjudication, client repair design, or a complete reproduction of the interoperability corpus; IDR-SRV-014F owns those tasks.
- Treating SECD behavior, OpenAPI, report cards, or a prototype statement as standards authority.

### 2.2 Plan Coverage Matrix

| Core question | Coverage | Evidence |
|---|---|---|
| Q1 Externally visible interfaces and directly evidenced internals | Complete; internals explicitly not verifiable | §§3–4 |
| Q2 Resources, collections, links, query, dynamic data, documentation, schemas, claims, and tests | Complete | §§5–13 |
| Q3 Alignment and divergence from Glaux baseline | Complete | §§5–8 |
| Q4 Strengths, gaps, and tradeoffs | Complete | §§7, 14, 16 |
| Q5 Downstream design, validation, conformance, interoperability, and documentation lessons | Complete | §§13–16 |

The 41 detailed questions are individually closed in Appendix A. Questions about unavailable internals are resolved as not verifiable rather than left unanswered.

---

## 3. SECD Source Inventory and Evidence Classification

### 3.1 Reproducible Source Baseline

| Source | Pin or capture | Class | Role | Limitation |
|---|---|---|---|---|
| OGC API - Connected Systems Part 1 | OGC 23-001, 1.0 | N | Feature-resource baseline | Normative; implementation examples do not override clauses |
| OGC API - Connected Systems Part 2 | OGC 23-002, 1.0 | N | Dynamic-data baseline | Normative |
| Accepted IDR-SRV-006 through 014C | Accepted repository state through August 31, 2026 | N for Glaux decisions | Exact Glaux interpretation and comparison baseline | Project authority, not OGC authority |
| Live SECD API root | https://cs.ogc.secd.eu/api/1.0, read-only capture 2026-08-31 | L | Current observable behavior | Mutable; TLS certificate was expired |
| Live SECD OpenAPI | /api-docs/openapi.json, OpenAPI 3.1.0, info.version 1.0.0, capture 2026-08-31 | O | Declared current surface | Mutable; incomplete relative to runtime |
| Maintainer introduction | correspondence/2026-05-07-matheus-email.md at f018fd1 | M | Purpose, contributors, prototype status, open-source intent | Private note; no architecture or license |
| Interoperability repository | main at f018fd129bf0d0d1ce75e68198e3ab4d99d937a0 | I | Dated captures, evaluations, findings, regression ideas | Private supporting repository; not server source |
| Historical server snapshot | analysis/00-server-snapshot.md at f018fd1, captured 2026-05-11 | H | Drift baseline | Superseded where current probes differ |
| Publication-ready SECD report card | report-cards/publication-ready/secd-report-card.md at f018fd1 | I/H | Curated historical finding index | May score must not describe August state |

### 3.2 Repository and Release State

The private evidence repository was created May 11, 2026 and remained at commit f018fd1 from May 25, 2026 when reviewed. It had one branch (main), no tags, no releases, no pull requests, 60 closed issue records, and no detected SPDX license. Its tree consists of Markdown research, raw HTTP evidence, payloads, plans, evaluations, adjudication records, and report cards. No server implementation source was present [I].

The live OpenAPI info version 1.0.0 is an API-description version, not proof of a software release. The actual server software version, build SHA, deployment identifier, source revision, release channel, and license remain unknown [O].

### 3.3 Evidence Strength and Conflict Rules

1. Approved standards and accepted Glaux reports control normative interpretation.
2. Current live observations control current deployment statements.
3. Current OpenAPI controls only declared documentation behavior; runtime parity must be checked separately.
4. Maintainer correspondence controls purpose and stated maturity, not unmentioned internals.
5. Pinned interoperability evidence controls historical observations only.
6. Historical/current conflicts are recorded as drift, not averaged.
7. Absence claims are bounded to the studied paths, OpenAPI document, repository tree, and search terms.
8. Inference never becomes a server fact or standards requirement.

### 3.4 Licensing and Reuse

No server-source license can be assessed because no server source was found. The evidence repository has no detected license, so its content should be cited, not copied as reusable implementation code. Live response examples and OpenAPI shapes are design evidence, not a license grant. Glaux may independently implement standards-derived behavior and independently authored tests; any code reuse requires a separately verified license.

---

## 4. SECD Observable API Topology and Explicitly Declared Architecture

### 4.1 Declared Product Boundary

The live landing page calls the product “OGC API – Connected Systems: Drone Data Platform” and describes drone telemetry, tasking, and data management [L]. The maintainer described a starting prototype aimed at drone management with In-Platform Tasking, with simulated and contributor-provided drones and a Live Datastream Tester [M]. These statements support the use-case boundary, not internal design.

### 4.2 Current Observable Topology

| Surface | Current behavior | Evidence |
|---|---|---|
| Landing page | JSON title, description, self, service-desc, service-doc, conformance, 9 data links, spatial related link | L |
| Conformance | 17 declared URIs spanning Common, Features, Part 1, Part 2 | L |
| Collections | Four records: systems, deployments, procedures, samplingFeatures | L |
| Part 1 direct routes | Systems, subsystems, Deployments, Procedures, Sampling Features; Property Definitions absent | L,O |
| Part 2 direct routes | DataStreams, Observations, ControlStreams, Commands; schema subroutes; live observations | L,O |
| System history | Per-System history returns event arrays; 122 events across two sampled Systems | L |
| Tasking | Two ControlStreams and 15 Commands; command status carried in resource; status PATCH documented | L,O |
| Spatial extension | Active drones, coverage, nearest/within observations, flight path, datastream statistics | O |
| API documentation | Swagger UI plus OpenAPI 3.1 JSON | L,O |

The root supports both direct resource links and a newly added collection discovery link. Collection endpoints cover only four Part 1 families; Part 2 resources remain direct routes.

### 4.3 Internal Architecture Boundary

| Question | Determination |
|---|---|
| Language/framework | Not verifiable. X-Powered-By: Express is an HTTP observation, not a complete source or architecture declaration. |
| Reverse proxy | nginx/1.24.0 (Ubuntu) was exposed in response headers; role and configuration are not verifiable. |
| Routing/handlers | Not verifiable from actual source. OpenAPI paths describe interfaces only. |
| Domain model | Internal model not verifiable; externally visible shapes are documented in §10. |
| Persistence | Not verifiable. A historical database constraint string is insufficient to establish architecture. |
| Validation implementation | Not verifiable. Observable validation outcomes are in §8. |
| Serialization implementation | Not verifiable; only response shapes are known. |
| Configuration/secrets | Not verifiable. |
| Tests/CI/build/release | Not verifiable for the server. |
| Deployment topology/scaling | Not verifiable. |

### 4.4 Architecture Lesson

Black-box evidence can establish a contract and expose inconsistencies, but it cannot justify copying internal patterns. Glaux should use SECD to derive acceptance tests and domain scenarios, not to select a Rust module layout, database, migration tool, framework, deployment topology, or concurrency model.

---

## 5. SECD CSAPI Part 1 Behavior Findings

### 5.1 Resource Coverage

| Resource area | Current finding | Classification |
|---|---|---|
| Systems | Root/list/item; 8 Systems; GeoJSON FeatureCollection; UUID id and URI-like uid | Implemented/observed |
| Subsystems | Nested route works; parentId field and proprietary filter work | Implemented with query divergence |
| Deployments | Root/list/item and System nesting; 4 records; rich geometry/time/mission properties | Implemented/observed |
| Subdeployments | No route or declaration found | Unsupported or unclear |
| Procedures | Root/list/item; 5 objects in procedures envelope; no GeoJSON Feature shape | Implemented with representation-specific shape |
| Sampling Features | Root/list/item and System-nested route; 7 GeoJSON features | Implemented/observed |
| Property Definitions | /properties returns 404; no root link or OpenAPI path | Not implemented in studied deployment |
| Collections | Four Part 1 collection records and matching items routes | Partially implemented |
| SensorML | SensorML Accept request returns ordinary JSON; no SensorML media in OpenAPI | Not implemented on tested read surface |

### 5.2 Identity, Relationships, and Links

Resource identifiers are UUID strings, while Systems also expose URI-like uids such as urn:drone:solar-uas:1 [L]. This separation is useful. Relationship fields are flattened as parentId, systemId, deploymentId, samplingFeatureId, datastreamId, and controlStreamId. That makes SQL-style relationships obvious to application developers but does not reproduce the standards’ canonical relationship model.

System item links use rel=self for the item and repeat rel=alternate for DataStreams, ControlStreams, subsystems, and Deployments. Observation links similarly use alternate to point to a DataStream. The hrefs work, but one generic relation label cannot tell a client which resource relationship is represented. Glaux should emit exact registered or standard-defined relations and use type/title only as supplementary metadata.

### 5.3 Representations

Systems, Deployments, and Sampling Features are GeoJSON Features. Procedures are flat objects without geometry in a procedures envelope. This is practical for procedure no-geometry behavior, but the collection model is inconsistent. System properties use type values such as Drone and Sensor rather than the accepted Glaux semantic feature-type model. Rich deployment properties include full flight paths and mission metadata; this is strong scenario material but risks unbounded feature payloads and duplicates a dedicated spatial/history concern.

### 5.4 Query Behavior

The OpenAPI declares limit, offset, bbox, type, status, parentId, near, and systemId depending on resource. It omits much of the accepted Glaux canonical query surface. Current probes showed:

- limit=invalid returns HTTP 400 with a compact ValidationError.
- q=drone and q=zzzz-no-match each returned all 8 Systems with HTTP 200.
- parent returned all 8 Systems, while parentId with the same UUID returned the 2 actual children.
- unknown f and format-specific Accept requests returned ordinary JSON with HTTP 200.

Silent acceptance is more dangerous than explicit rejection because a caller can mistake an unfiltered result for a correct one. Glaux must reject unsupported or malformed canonical parameters and must never document proprietary aliases as the only path to required semantics.

### 5.5 Mutation Evidence Boundary

The OpenAPI declares POST for all studied Part 1 root resources, PUT/DELETE for Systems, Deployments, and Sampling Features, and no Procedure update/delete [O]. The pinned May corpus records partial anonymous writes, flat write shapes, GET/PUT asymmetry, and route-specific failures [H]. This study did not repeat mutations against the shared live deployment. Those results remain historical and must be refreshed under controlled authorization before any current write claim or design conclusion.

---

## 6. SECD CSAPI Part 2 Behavior Findings

### 6.1 DataStreams and Observations

The deployment exposed 13 DataStreams and 11,250,822 Observations. DataStreams include observedProperty URIs, units, DataRecord or scalar output schemas, and phenomenon/result time extents. Observations include phenomenonTime, resultTime, result, quality, geometry, parameters, and links [L]. Root and datastream-scoped observation reads work. A DataStream schema route returns identity, property, unit, and outputSchema, although that route is absent from the OpenAPI [L,O].

The OpenAPI documents an SSE endpoint at /datastreams/{id}/observations/live with text/event-stream. The operation was not held open during this bounded study, so live streaming is declared but not independently observed. Historical access is strongly evidenced by the large persisted corpus and scoped queries; replay semantics, ordering guarantees, resumability, backpressure, retention, and delivery guarantees are not verifiable.

### 6.2 ControlStreams and Commands

Two ControlStreams describe drone flight commands through a DataChoice inputSchema with GOTO, RETURN_HOME, LAND, SET_SPEED, and HOVER choices. Fifteen Commands were present, including issueTime, status, parameters, sender, and links. A ControlStream schema route returns the inputSchema [L].

The live OpenAPI declares root POST /commands, item GET, nested ControlStream command GET, and PATCH /commands/{id}/status. It does not document a command result read, feasibility routes, or canonical status/result subresources. Current GET probes for /commands/{id}/status and /commands/{id}/result returned 404 HTML. The command object itself carries status. This is useful prototype tasking behavior, but it is not a complete Glaux command lifecycle.

### 6.3 System Events and History

The conformance response claims an implementation-era system-events class. The OpenAPI defines a SystemEvent schema and a per-System /history route, but no root System Events path. Current /systemevents returned 404. Two Systems’ history arrays contained 22 and 100 events, respectively [L,O].

System history is useful application behavior but is not a substitute for the canonical Part 2 System Event resource family. Glaux should model durable System Events and derive history views from them where appropriate, not advertise a class based only on an extension-shaped history array.

### 6.4 Dynamic Properties, Feasibility, and Encodings

No Property Definitions or dynamic-property snapshot route was found. No feasibility operation was found. JSON is the only documented DataStream, Observation, ControlStream, and Command resource media type. DataRecord and DataChoice components are embedded as JSON objects, but no SWE JSON/Text/Binary data encoding classes are declared or documented. Therefore rich embedded schema use must not be confused with proof of SWE encoding conformance.

### 6.5 Part 2 Lesson

SECD shows that a compelling drone demo can be assembled from DataStreams, high-volume Observations, ControlStreams, Commands, system history, spatial queries, and embedded schemas. Glaux should preserve that end-to-end usability while adding canonical resource identities, complete lifecycle state, truthful encodings, explicit retention/streaming contracts, feasibility, System Events, and authorization.

---

## 7. SECD Conformance Posture and Standards-Alignment Matrix

### 7.1 Declaration Assessment

The live response declares 17 URIs. The Common and Features labels include core, collections, oas30, json, GeoJSON, and a Features Part 4 transaction label. The CSAPI labels use names such as common, system-features, subsystem-features, datastream-observations, control-stream-commands, and system-events [L].

Accepted IDR-SRV-008 identifies the approved direct CSAPI class names as Part 1 api-common, system, subsystem, deployment, subdeployment, procedure, sf, property, advanced-filtering, create-replace-delete, update, geojson, and sensorml; and Part 2 api-common, datastream, controlstream, feasibility, system-event, advanced-filtering, create-replace-delete, update, json, swecommon-json, swecommon-text, and swecommon-binary [N]. The SECD CSAPI declarations do not match that set. The oas30 claim also conflicts with a live OpenAPI 3.1.0 document and the accepted IDR-SRV-014 rule that a 3.1 document does not satisfy the OAS 3.0 class [L,O,N].

No official OGC ATS execution or conformance certificate was found. Every declaration must therefore be treated as an unverified runtime claim.

### 7.2 Implementation Findings Matrix

| Area | SECD source / anchor | Evidence | Observed behavior | CSAPI / Glaux baseline | Alignment | Useful pattern | Gap / risk | Glaux applicability | Test implication | Handoff | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Discovery | live root | L | Absolute self, docs, conformance, data, collections, spatial links | IDR-009/010 | Partial | Clear entry page | data rel and partial collections need adapters | High | Root traversal golden tests | 009/010/014E/F/056 | Root is richer than May capture |
| Collections | /collections | L | Four Part 1 collections and items links | Part 1 API Common | Partial | Dual direct/collection discovery | No Part 2 or Property collections | High | Collection/direct parity | 010/050/056 | F01 historically resolved in part |
| Claims | /conformance | L,N | 17 implementation-era URIs | IDR-008 exact 25-class target | Divergent | Capability list exists | Noncanonical names; unsupported system-events; OAS mismatch | Critical | Exact claim-to-evidence tests | 008/014/050/051 | Claims are not certification |
| Systems | /systems | L | GeoJSON, UUID id, uid, hierarchy, status | P1 System/Subsystem/GeoJSON | Partial | Clean id/uid distinction | plain type, alternate relations, query aliases | High | Model/link/filter round trips | 015/017/018 | 8 current records |
| Deployments | /deployments | L | GeoJSON, time, system and procedure links, mission data | P1 Deployment | Partial | Rich operational scenario | full flight paths inside properties; subdeployment unclear | Medium | Large payload/time/relationship tests | 015/018/025/053 | 4 records |
| Procedures | /procedures | L | Flat procedure objects | P1 Procedure | Partial | No artificial geometry | resource-named envelope; SensorML absent | High | no-geometry plus representation tests | 015/021/023 | 5 records |
| Sampling Features | /samplingfeatures | L | GeoJSON and nested System path | P1 Sampling Feature | Partial | Domain examples | relation semantics and canonical query coverage unclear | High | nested membership tests | 017/024/053 | 7 records |
| Property Definitions | /properties | L | 404 | P1 Property | Unsupported | — | Cannot discover semantic properties | High | absent-class/negative tests | 015/021/023 | Must not be claimed |
| Queries | systems probes; OAS | L,O | limit validation; q/parent silently ignored; parentId works | IDR-011 | Divergent | bounded limit | false-success wrong results | Critical | unknown/canonical/alias/metamorphic tests | 011/050/056 | Read-only reproduction |
| Negotiation | Accept/f probes; OAS | L,O | Always application/json for tested System requests | IDR-012 | Divergent | predictable default | no 406; SensorML and invalid values ignored | Critical | Accept/f matrix | 012/014/050/056 | No SensorML evidence |
| Errors | invalid limit/id | L | JSON code + description for routed 400/404 | IDR-013 | Partial | Compact stable envelope | framework HTML for some absent nested routes; no documented full error set | High | all-route error parity | 013/014/050 | Problem Details is Glaux policy |
| DataStreams | list/item/schema | L,O | embedded DataRecord/Quantity schema and time extents | P2 DataStream | Partial | schema-aware dynamic resource | wrapper/relationship names; schema route undocumented | High | schema/OAD parity and mapping tests | 022/023/034 | 13 records |
| Observations | root/scoped/list | L,O | high-volume historical JSON, time, geometry, quality | P2 DataStream + JSON | Partial | realistic corpus | root GET omitted from OAS; advanced filters/encodings unclear | High | volume/paging/time/shape tests | 034/048/053 | 11,250,822 matched |
| Live data | SSE OAS operation | O | text/event-stream declared | Streaming strategy | Declared only | Simple live feed | delivery/resume/auth/backpressure unverified | Medium | controlled stream contract tests | 027/034/048 | Not held open |
| ControlStreams | list/item/schema | L,O | rich DataChoice command schema | P2 ControlStream | Partial | realistic task schema | status/result/feasibility gaps | High | schema-to-command generation | 036/037/053 | 2 records |
| Commands | root/nested/item | L,O | 15 durable-looking command records and statuses | P2 ControlStream/JSON | Partial | visible command state | lifecycle completeness, auth, result semantics unverified | Critical | state-machine/authorization tests | 036/038/039 | Current writes not probed |
| System Events | conformance/history | L,O | history arrays, no canonical endpoint | P2 System Event | Divergent | auditable event concept | false claim; extension substitutes for resource | High | claim/route/persistence tests | 020/035/050 | 122 sampled history events |
| OpenAPI | openapi.json | O,L | 3.1.0, 32 paths, 37 schemas, 5 shared params | IDR-014 | Partial | browsable compact contract | missing runtime routes, only JSON, no security, incomplete errors | Critical | bidirectional runtime parity | 014/050/051 | info.version 1.0.0 |
| Security posture | TLS/headers/OAS | L,O,H | expired certificate; no security schemes; anonymous reads; historical sandbox writes | Glaux security baseline | Divergent/unclear | easy demo access | trust failure and unbounded authorization risk | Critical | TLS/authz route matrix | 039/039A/040/042 | No current mutation probe |
| Architecture | correspondence and response headers | M,L | prototype intent; nginx/Express breadcrumbs | Implementation study constraint | Not verifiable | clear use-case boundary | no source, license, build, internals, tests | Low as architecture source | provenance and SBOM gates | 025/028/045 | Do not infer |

### 7.3 Overall Classification

SECD is a broad, useful prototype with partial standards alignment and significant contract divergence. Its strongest value to Glaux is as an adversarial behavior and scenario corpus: it proves that generic clients will encounter mixed envelopes, proprietary relationship fields, partial discovery, false-success filters, ignored negotiation, stale conformance labels, runtime/OAD drift, and incomplete tasking lifecycles even when the domain demo is compelling.

---

## 8. SECD API Behavior Findings

### 8.1 Entry Points and Navigation

The landing page is human-readable and provides absolute links. The new collections link materially improves bootstrap behavior. Direct links still use rel=data with titles as resource discriminators. Item relationships use rel=alternate. Glaux should support robust consumption of these patterns in interoperability tools, but its own server must emit canonical, typed relations.

### 8.2 URI and Method Behavior

The current direct route families are plural lower-case paths, with samplingfeatures, controlstreams, and systemevents lower-cased. Runtime also supports nested System DataStream, ControlStream, Sampling Feature, Deployment, subsystem, and history paths. Several nested routes are absent from the OpenAPI. OpenAPI declares 46 GET/POST/PUT/PATCH/DELETE operations across 32 paths, including six spatial extension paths.

Glaux should separate standard routes from extensions in its capability registry and OAD. An extension path must not serve as hidden evidence for a standard class.

### 8.3 Pagination and Selection

Collections generally expose numberMatched, numberReturned, and first/next links with limit and offset. The OpenAPI bounds limit from 1 to 10,000 and offset from zero. Procedures and ControlStreams may omit a top-level links member or use a single link. Offset pagination is simple but is vulnerable to duplicates/skips under concurrent writes and expensive deep scans. Glaux’s accepted IDR-SRV-011 cursor and deterministic-order policies remain controlling.

### 8.4 Content Negotiation

Bounded System probes with Accept values application/json, application/geo+json, application/sml+json, and application/x-glaux-invalid all returned HTTP 200, Content-Type application/json, and the same GeoJSON-shaped document. f=json and f=html behaved the same. The OpenAPI advertises application/json for nearly every resource and text/event-stream only for live observations.

This is false-success negotiation. Glaux must select a supported representation explicitly, return 406 for unacceptable Accept, return 400 for invalid f, return the negotiated Content-Type, and keep OAD, links, schemas, and codecs synchronized.

### 8.5 Error Behavior

Invalid limit produced HTTP 400 with code=ValidationError and a bounded description. A missing System produced HTTP 404 with code=NotFound. Some absent nested routes returned Express HTML 404 pages instead of the JSON error envelope. OpenAPI describes selected 400/404/409 responses but omits a comprehensive 401/403/406/415/429/5xx contract.

Glaux should retain a compact machine code and safe human detail, but enforce its accepted RFC 9457 error policy at every router and middleware boundary.

### 8.6 TLS and Operational Contract

Normal TLS validation failed with SEC_E_CERT_EXPIRED; evidence retrieval required an explicit insecure client override. The response exposed nginx/1.24.0 (Ubuntu), X-Powered-By: Express, permissive Access-Control-Allow-Origin: *, and no OpenAPI security scheme. These are dated deployment observations, not claims about the intended production product. They nevertheless demonstrate why deployment health, TLS expiry, header policy, and OAD security parity must be release gates.

---

## 9. SECD Dynamic-Data, Status, and Tasking Findings

### 9.1 Historical Data

The observation corpus is the strongest practical feature. DataStreams expose time extents and Observations carry phenomenon and result times, location, quality, parameters, and result payloads. The sample solar-aircraft position duplicated canonical and domain-specific field names inside one result, showing both real ingestion flexibility and the risk of weak schema normalization.

### 9.2 Real-Time and Replay

The declared SSE endpoint is a plausible low-complexity live channel. The dashboard and maintainer note further support live-demo intent. No evidence establishes replay cursors, Last-Event-ID, delivery semantics, ordering, retention, reconnect behavior, filtering, or authorization. Glaux should treat those as explicit contracts in IDR-SRV-027, 034, and 048.

### 9.3 Status and Events

Systems carry status values and Commands carry lifecycle status. System history exposes timestamped events for two drones. There is no canonical System Event root. Status vocabulary, transition authority, idempotency, terminal-state rules, event correlation, and audit retention are not verifiable.

### 9.4 Tasking

SECD’s DataChoice is a valuable fixture because it models alternative command shapes rather than a single flat parameter record. It demonstrates the need for schema-aware client forms and server validation. However, the current surface does not establish feasibility, command results, cancellation, expiry, acknowledgement, asynchronous transition rules, authorization, or physical actuation safety. Glaux must not infer those from “accepted” status records.

### 9.5 Dynamic-Data Lesson

Expose one durable lifecycle with explicit state transitions, commands, results, feasibility, events, and audit relationships; derive live delivery from the same persisted semantics. A dashboard-friendly happy path is not enough to define a safe command system.

---

## 10. SECD Data-Model, Persistence, and Fixture Implications

### 10.1 Externally Visible Model

| Model aspect | Observed SECD choice | Glaux implication |
|---|---|---|
| Identity | UUID local ids plus URI-like System uid | Preserve separate stable local and global identity |
| Hierarchy | parentId and nested subsystem path | Use typed graph relations and cycle-safe persistence |
| Cross-resource references | id-suffixed scalar fields | Accept only as explicit adapters; emit canonical model |
| Feature resources | GeoJSON Feature for Systems, Deployments, Sampling Features | Strong baseline for spatial resources |
| Procedures | Flat no-geometry object | Preserve no-geometry semantics without envelope drift |
| Dynamic schemas | Embedded DataRecord, Quantity, Category, DataChoice | Build typed SWE component model and schema registry |
| Time | ISO 8601 instants and {start,end} extents | Normalize interval semantics, open bounds, UTC, and precision |
| Domain extension | Arbitrary properties and spatial endpoints | Namespaced, governed extension lane |

### 10.2 Internal Persistence

Not verifiable. Current record counts and persistence across reads prove durable-looking externally visible state, not a particular database, schema, transaction model, or consistency guarantee. Historical error text must not select PostgreSQL for Glaux. IDR-SRV-025 remains the decision owner.

### 10.3 Validation

Observable positive evidence includes bounded integer limit validation and typed UUID/enum/date declarations in OpenAPI. Observable negative evidence includes ignored canonical and unknown query parameters, unconstrained object schemas, inconsistent HTML/JSON errors, and negotiation false-success. Historical write evidence reports partial validation and database-layer leakage, but current mutation behavior was not probed.

Glaux needs validation at transport, representation, domain, relationship, authorization, transaction, persistence, and response layers, with each failure mapped to one stable error contract.

### 10.4 Fixtures

SECD offers unusually valuable fixtures:

- drone and sensor hierarchies;
- flight and bridge-inspection Deployments;
- Procedures containing mission and flight-path metadata;
- SamplingPoint, SamplingCurve, and SamplingSurface examples;
- scalar, categorical, and DataRecord DataStreams;
- high-volume historical and live-adjacent Observations;
- DataChoice flight commands;
- status/history events;
- custom domain codes such as DIN 1076;
- malformed or divergent query, link, envelope, and negotiation behavior preserved by the evidence repository.

Glaux should independently author compact canonical versions plus deliberate invalid/adversarial variants. Do not import identifiers, personal data, raw mission payloads, or private repository content blindly.

---

## 11. SECD Documentation, OpenAPI, and Examples Findings

### 11.1 OpenAPI Inventory

The live document is OpenAPI 3.1.0, info.version 1.0.0, with 32 paths, 37 schemas, 5 shared parameters, and no securitySchemes. It describes Part 1 and Part 2 reads/writes, SSE, command status PATCH, health, and six spatial routes. JSON is the only documented request/response media type except text/event-stream.

### 11.2 Runtime Parity

Runtime behaviors missing from OpenAPI include:

- /collections, collection descriptions, and collection item routes;
- top-level GET /observations and GET /commands;
- nested System DataStreams, ControlStreams, and Sampling Features;
- DataStream and ControlStream schema routes;
- several runtime error forms.

Conversely, an OpenAPI path does not prove a complete lifecycle, validation, or conformance class. This deployment is a strong example of why OAD/runtime parity must be tested in both directions.

### 11.3 Schema Quality

The schemas are readable but permissive. Most response objects declare no required fields. Several allOf-derived resources carry no directly visible properties in a shallow inventory. Embedded outputSchema, inputSchema, result, parameters, and properties are generic objects. additionalProperties is generally unconstrained. CommandList omits numberReturned in its declared properties even though runtime returned it.

Glaux’s OAD should be generated from the same typed contract as runtime validation and then verified against live examples and negative cases.

### 11.4 Documentation and Examples

Swagger UI, a dashboard, a Live Datastream Tester statement, rich current data, historical captures, and extensive private evaluation evidence make SECD approachable. Missing public source, software version, license, build provenance, deployment identifier, authoritative architecture notes, and public test results make it non-reproducible as an implementation.

### 11.5 Release and Test Evidence

No server release, tag, changelog, test suite, CI workflow, coverage record, SBOM, container digest, or official ATS result was available. The interoperability repository has strong black-box evidence but zero tags/releases and no license. Glaux must publish immutable release evidence tied to source, generated contracts, migrations, tests, and deployment probes.

---

## 12. Interoperability Questions and IDR-SRV-014F Handoff

### 12.1 Current Questions

1. Which historical SECD findings still reproduce after the August discovery and route changes?
2. Which failures belong to SECD, which to strict client assumptions, and which expose ambiguity or defects in the standards/artifacts?
3. Can a generic client traverse root → collections/direct routes → items → related resources without title/path heuristics?
4. How should clients handle mixed GeoJSON and resource-named envelopes?
5. Can tolerant adapters accept parentId and id-suffix references without corrupting canonical client state?
6. How do current OS4CSAPI clients react to ignored Accept/f and query parameters?
7. Which historical write behaviors remain current under an authorized isolated fixture?
8. How should the fixed /collections and top-level routes change the May report-card conclusions?

### 12.2 Evidence Package for 014F

IDR-SRV-014F should start from pinned commit f018fd1 and add an August 31 drift layer:

| Historical finding | August 31 status for handoff |
|---|---|
| F01 /collections absent | Changed: /collections now returns 4 Part 1 collections |
| F03 resource-named wrappers | Still observed for Procedures and Part 2 lists |
| F04 plain System type | Still observed |
| F10 noncanonical conformance names | Still observed against accepted IDR-008 |
| F21 alternate relationship links | Still observed |
| F30/F31 format negotiation ignored | Still observed in bounded System probes |
| F34/CF01 query false-success | Still observed for q and parent |
| F36 top-level observations absent | Changed: top-level /observations now returns data |
| F62 properties absent | Still observed |
| CF04 and other write findings | Historical only; not rerun in this read-only study |

### 12.3 Boundary

This report does not adjudicate root cause, client correctness, severity, scoring, or cross-server prevalence. It supplies implementation observations and test hypotheses. IDR-SRV-014F must reproduce current behavior, cite exact client revisions, separate deployment drift from code behavior, and reconcile any standards interpretation against the accepted Glaux baseline.

---

## 13. Test-Strategy Implications

### 13.1 Existing Evidence

The server’s own tests are unavailable. The supporting repository provides extensive dated HTTP captures, two evaluation tracks, a 73-row reconciliation, payloads, report-card traceability, and targeted validation artifacts [I]. That is strong external behavior evidence, not a substitute for server unit/integration tests or an official ATS.

### 13.2 Priority Glaux Regression Corpus

| Test family | Required cases derived from SECD |
|---|---|
| Discovery | Root link relation/type/title; direct vs collections parity; partial collection inventory; dead link detection |
| Claims | Exact URI spelling; route/format/method evidence for every claim; reject aspirational claims |
| Navigation | Multiple typed relationships; no rel=alternate overloading; nested/root identity equality |
| Envelopes | GeoJSON, item/resource collections, empty collections, mixed-wrapper rejection |
| Query | q known/unknown; parent vs alias; bbox/time/id/type/status; unsupported/unknown rejection; metamorphic result checks |
| Pagination | limit bounds; stable order; first/next; zero/last page; concurrent insert/delete behavior |
| Negotiation | Accept and f cross-product; 406/400; exact Content-Type; SensorML/GeoJSON/JSON; invalid media |
| Errors | Router, validation, authorization, domain, transaction, and infrastructure failures share one safe envelope |
| OAD parity | Runtime-only routes; OAD-only routes; request/response media; error statuses; security; examples |
| Schema | DataRecord, scalar Quantity/Category, DataChoice, nested objects, extra fields, invalid units/types |
| Dynamic data | large counts, time extents, scoped/root equality, ordering, replay, resume, retention |
| Tasking | schema-to-command mapping; feasibility; transition matrix; result/status; idempotency; authz; audit |
| Events | canonical System Events vs derived history; correlation, time, persistence, filters |
| Security/operations | TLS expiry, HSTS/header policy, CORS, anonymous/authenticated route matrix, banner suppression |
| Drift | replay immutable captures and a dated live probe without making mutable live state the oracle |

### 13.3 Required Test Layers

1. Pure Rust unit/property tests for identifiers, relations, schemas, time, state machines, query parsing, and negotiation.
2. Repository and transaction integration tests with deterministic fixtures and failure injection.
3. HTTP contract tests generated from the capability registry.
4. OAD/runtime/schema parity tests.
5. Official ATS tests preserved separately from Glaux supplements.
6. Interoperability adapters and multi-client tests.
7. Security, authorization, TLS, abuse, and disclosure tests.
8. Performance/soak tests for large observation sets and live streams.
9. Migration, backup/restore, downgrade, and release evidence tests.

### 13.4 Oracle Policy

Use approved standards for expected behavior, Glaux policy for project-selected details, immutable fixtures for deterministic regression, and live servers for compatibility observation. Never turn one live SECD response or report-card score into the normative oracle.

---

## 14. Lessons for Glaux Server

### 14.1 Adopt

- Separate stable local ids from global URI identifiers.
- Preserve direct resource routes and navigable collection discovery.
- Use rich, realistic cross-resource fixtures.
- Model scalar, record, categorical, and choice schemas explicitly.
- Make historical and live dynamic data first-class.
- Keep domain extensions possible through governed properties and extension paths.
- Validate bounded pagination inputs with deterministic errors.
- Provide human documentation, machine OAD, and interactive examples.

### 14.2 Avoid

- Static or aspirational conformance arrays.
- Noncanonical class names and claims unsupported by routes.
- One generic link relation for unrelated resource relationships.
- Silent acceptance of unsupported filters or media types.
- Proprietary query names replacing canonical names.
- Mixed collection envelopes without a typed representation contract.
- Runtime routes missing from OpenAPI.
- Unconstrained object schemas as the only validation boundary.
- GET and write representations that cannot round-trip.
- HTML framework errors leaking through API routes.
- Open, unauthenticated tasking surfaces outside an explicitly isolated demo.
- Mutable live deployments as release evidence.

### 14.3 Investigate

- Canonical modeling of DataChoice command alternatives in Rust.
- A durable System Event model from which history projections are derived.
- SSE contract details and relationship to later streaming bindings.
- Placement of flight paths: Deployment geometry, observation trajectory, extension resource, or derived view.
- Canonical property/units vocabulary and domain-code registry.
- Safe compatibility adapters for id-suffix references and legacy envelopes.

---

## 15. Downstream Topic Handoff Matrix

| Topic | Handoff |
|---|---|
| IDR-SRV-014E | Test discovery, wrapper, link, query, negotiation, and OAD assumptions against current SECD without importing stale May results |
| IDR-SRV-014F | Reproduce and adjudicate the drift table in §12 with exact client pins and authorized isolated writes |
| IDR-SRV-014G | Use only discussion-backed lessons; compare them against observed SECD drift and accepted standards |
| IDR-SRV-015 | Model id/uid, hierarchy, typed relationships, Procedures, Property Definitions, schemas, Commands, and System Events canonically |
| IDR-SRV-017 | Enforce relationship integrity, cascade policy, and root/nested identity equality |
| IDR-SRV-018 | Specify time extents, event/observation time, flight trajectories, and query semantics |
| IDR-SRV-020 | Define durable System Events and derived history views |
| IDR-SRV-021/022/023 | Implement SensorML and SWE component/encoding validation rather than accepting schema-shaped generic objects |
| IDR-SRV-025 | Select persistence independently; require high-volume observations, graph relations, migrations, and transactions |
| IDR-SRV-027/034 | Define historical/live observation ordering, retention, replay, backpressure, and schema evolution |
| IDR-SRV-036/037/038 | Define ControlStream/Command/feasibility/result/status state machines and idempotency |
| IDR-SRV-039/039A/040/042 | Treat every tasking and mutation operation as an authorization boundary; gate TLS and security metadata |
| IDR-SRV-048 | Exercise large datasets, SSE lifecycle, and concurrent paging |
| IDR-SRV-050/051 | Generate claim-to-route-to-test traceability and supplement deficient ATS areas |
| IDR-SRV-053 | Build canonical and adversarial drone, mission, bridge, schema, and tasking fixtures |
| IDR-SRV-056 | Include tolerant read adapters and strict-server tests for every SECD divergence class |

---

## 16. Recommendations

1. **R-014D-001 — Use one typed capability contract.** Derive or verify routes, relationships, canonical queries, representations, schemas, claims, OpenAPI, authorization actions, and test obligations from it. Priority: High.
2. **R-014D-002 — Make conformance evidence executable.** A class enters /conformance only when its prerequisites, runtime surface, representations, negative tests, and release evidence pass. Priority: High.
3. **R-014D-003 — Reject silent wrong-result behavior.** Unknown or unsupported canonical filters and formats return deterministic client errors; aliases live in a separately governed compatibility lane. Priority: High.
4. **R-014D-004 — Preserve canonical relationship semantics.** Use typed relation identifiers and verify root/nested paths resolve to the same resource identity. Priority: High.
5. **R-014D-005 — Implement schema-aware dynamic and tasking models.** Treat DataRecord, DataChoice, Observation, Command, status, result, feasibility, and System Event as typed domain constructs, not unvalidated JSON blobs. Priority: High.
6. **R-014D-006 — Generate and parity-test OpenAPI.** Every runtime route, media type, security rule, status, and schema must agree bidirectionally with the deployed OAD. Priority: High.
7. **R-014D-007 — Separate durable semantics from transport.** Persist observations, commands, results, statuses, and events coherently; layer SSE or later streaming bindings over the same lifecycle. Priority: High.
8. **R-014D-008 — Build a SECD-inspired fixture pack.** Independently author canonical drone, sensor, mission, bridge, record, choice, observation, command, and drift fixtures plus invalid variants. Priority: Medium.
9. **R-014D-009 — Gate operational trust.** TLS expiry, authentication/authorization parity, CORS, safe errors, OAD security, and secret handling are release blockers. Priority: High.
10. **R-014D-010 — Keep compatibility evidence versioned.** Pin external repositories, captures, client revisions, and deployment dates; explicitly supersede historical findings when live behavior changes. Priority: High.

---

## 17. Risks, Constraints, and Open Questions

### 17.1 Risks and Constraints

- Server source and architecture are unavailable; internal reuse recommendations would be speculation.
- The live deployment is mutable and already drifted materially since May.
- The expired TLS certificate required an insecure retrieval override and limits ordinary-client reachability.
- High observation counts make careless full-collection probes expensive.
- Historical write findings may no longer represent the current deployment.
- The private evidence repository has no detected license.
- OpenAPI and runtime do not have complete parity.
- Current conformance labels can create false capability discovery.
- Anonymous demo behavior could be mistaken for production security policy.
- Rich mission payloads may contain sensitive operational or provenance data.

### 17.2 Resolved Open Questions

| Plan question | Resolution |
|---|---|
| Is actual server source or maintainer architecture available? | No source or architecture artifact was found; black-box scope used. |
| What live capture represents execution? | Read-only August 31, 2026 capture of root, OAD, resources, routes, queries, negotiation, errors, TLS, and selected counts. |
| Which behaviors are deliberate, scaffold, fixture, or prototype? | Prototype purpose is maintainer-declared; individual behavior intent is otherwise not verifiable. |
| Which fixtures are useful? | Drone hierarchies, missions, bridge inspection, DataRecord/DataChoice, large observations, history, commands, and divergence cases in §10/13. |

### 17.3 Remaining Open Questions

1. Where is the intended open-source server repository, and under what license?
2. What software build/revision is deployed at the API root?
3. Which 17 claims are intentional current claims, and will they be migrated to approved class URIs?
4. Are System history events intended to become canonical System Event resources?
5. What persistence, transaction, retention, and backup guarantees exist?
6. What are the live-stream replay, ordering, authorization, and backpressure contracts?
7. What is the intended Command/feasibility/result/status lifecycle?
8. Which write findings still reproduce in an isolated authorized environment?
9. Will the OAD be generated from runtime contracts, and how will parity be tested?
10. What deployment/security posture is intended beyond the public prototype?

---

## 18. Validation Against the Research Plan

| Success criterion | Status | Evidence |
|---|---|---|
| Exact SECD sources, pins, deployments, and dates | Met | §3, Appendix B |
| Evidence classes distinguished | Met | Reading Guide, §3 |
| Observable API and declared architecture summarized | Met | §4 |
| Unsupported internal claims marked not verifiable | Met | §§3–4, 10 |
| Part 1 and Part 2 compared to Glaux baseline | Met | §§5–7 |
| Conformance assessed without assumption | Met | §7 |
| Entry point, navigation, query, representation, error, OAD, dynamic, status, tasking assessed | Met | §§4–11 |
| Strengths, gaps, risks, assumptions identified | Met | §§7, 14, 17 |
| Glaux design, validation, testing, interoperability lessons documented | Met | §§13–16 |
| Downstream handoffs explicit | Met | §§12, 15 |
| References explicit and reproducible | Met | §19, Appendix B |

All nineteen required content areas appear as numbered sections. The implementation findings matrix contains all twelve mandatory fields. All five core and 41 detailed questions are covered or explicitly resolved as not verifiable.

---

## 19. References

### Controlling and Glaux Sources

- [OGC API - Connected Systems - Part 1: Feature Resources](https://docs.ogc.org/is/23-001/23-001.html), OGC 23-001, Version 1.0.
- [OGC API - Connected Systems - Part 2: Dynamic Data](https://docs.ogc.org/is/23-002/23-002.html), OGC 23-002, Version 1.0.
- [Official Connected Systems overview and artifact links](https://ogcapi.ogc.org/connectedsystems/).
- [Official OGC schema repository](https://schemas.opengis.net/ogcapi/connected-systems/).
- [IDR-SRV-006 Part 1 Requirement Baseline](idr-srv-006-csapi-part-1-requirement-baseline-report.md).
- [IDR-SRV-007 Part 2 Requirement Baseline](idr-srv-007-csapi-part-2-requirement-baseline-report.md).
- [IDR-SRV-008 Conformance Mapping](idr-srv-008-conformance-class-and-requirement-mapping-report.md).
- [IDR-SRV-009 Landing, API Definition, and Conformance](idr-srv-009-landing-page-api-definition-and-conformance-declaration-behavior-report.md).
- [IDR-SRV-010 Collections, Resources, Links, and Navigation](idr-srv-010-collections-resources-links-and-navigation-behavior-report.md).
- [IDR-SRV-010A API Versioning and Compatibility](idr-srv-010a-api-versioning-backward-compatibility-and-deprecation-strategy-report.md).
- [IDR-SRV-011 Query, Filtering, Sorting, Pagination, and Selection](idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md).
- [IDR-SRV-012 Content Negotiation and Media Types](idr-srv-012-content-negotiation-media-types-and-encoding-selection-report.md).
- [IDR-SRV-013 Error Model and Failure Semantics](idr-srv-013-error-model-http-status-codes-and-failure-semantics-report.md).
- [IDR-SRV-014 OpenAPI and Documentation Strategy](idr-srv-014-openapi-description-and-api-documentation-strategy-report.md).
- [IDR-SRV-014A OSH Implementation Study](idr-srv-014a-osh-csapi-server-implementation-study-report.md).
- [IDR-SRV-014B Connected Systems Go Implementation Study](idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md).
- [IDR-SRV-014C pygeoapi Implementation Study](idr-srv-014c-pygeoapi-csapi-server-implementation-study-report.md).

### SECD Sources

- [Live SECD API root](https://cs.ogc.secd.eu/api/1.0), retrieved 2026-08-31.
- [Live SECD OpenAPI 3.1 document](https://cs.ogc.secd.eu/api/1.0/api-docs/openapi.json), retrieved 2026-08-31.
- [Pinned SECD interoperability repository](https://github.com/Sam-Bolling/csapi-server-interop-secd/tree/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0), private supporting evidence.
- [Maintainer introduction](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/correspondence/2026-05-07-matheus-email.md).
- [Historical May server snapshot](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/analysis/00-server-snapshot.md).
- [Historical initial compatibility report](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/analysis/01-initial-compatibility-report.md).
- [Historical reconciled findings](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/reinvestigation/summaries/findings-reconciliation.md).
- [Historical SECD report card](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/report-cards/publication-ready/secd-report-card.md).

---

## Appendix A. Detailed Question Ledger

| # | Detailed question | Resolution | Evidence |
|---|---|---|---|
| 1 | Is actual server source available? | No; evidence repo is not source | §3 |
| 2 | Exact branch, commit, deployment, or capture? | Evidence main/f018fd1; live 2026-08-31 | §3 |
| 3 | Evidence class for each fact? | N/L/O/M/I/H/X scheme | Reading Guide |
| 4 | Licensing constraints? | Server unknown; evidence repo unlicensed | §§3.4, 17 |
| 5 | Explicit architecture? | Product purpose only; internals not verifiable | §4 |
| 6 | Source-backed handlers/models/persistence/tests? | Not verifiable | §4.3 |
| 7 | Which internals remain unknown? | All named internal areas listed | §4.3 |
| 8 | External model/resource behavior? | Fully inventoried | §§4–10 |
| 9 | Relevant vs SECD-specific patterns? | Adopt/avoid/investigate separation | §14 |
| 10 | Part 1 resources? | Systems/subsystems/deployments/procedures/SF; gaps noted | §5 |
| 11 | Part 2 resources? | DS/Obs/CS/Commands; gaps noted | §6 |
| 12 | Claims supported/partial/unclear? | Claim comparison and evidence matrix | §7 |
| 13 | Landing/OAD/conformance/collections/links/schemas/media/query/errors? | Assessed | §§4, 7, 8, 11 |
| 14 | Baseline gaps/deviations? | Matrix and narrative | §§5–8 |
| 15 | Evidence per assessment? | Labels and source anchors | Throughout |
| 16 | URI patterns? | Direct, collection, nested, spatial catalogued | §§4, 8 |
| 17 | Links/relationships? | alternate/data and id-suffix behavior assessed | §§5, 8 |
| 18 | Query/filter/sort/page/select? | Declared surface and probes assessed; sort/select absent | §§5.4, 8.3 |
| 19 | Negotiation/media types? | JSON-only and false-success probes | §8.4 |
| 20 | Error behavior? | Routed JSON and framework HTML assessed | §8.5 |
| 21 | OAD/docs artifacts? | Swagger/OpenAPI/runtime parity assessed | §11 |
| 22 | Behaviors informing Glaux API topics? | Explicit handoffs | §§13–16 |
| 23 | Datastreams/observations/status/events/properties? | Assessed | §§6, 9 |
| 24 | Real-time/history/replay/event publication? | History observed, SSE declared, guarantees unknown | §9 |
| 25 | Control/command/feasibility/status? | Assessed; feasibility/result gaps | §§6.2, 9.4 |
| 26 | Dynamic/tasking completeness? | Partial with explicit unknowns | §§6, 9 |
| 27 | Dynamic/streaming/command handoffs? | Mapped | §15 |
| 28 | External model and fixtures? | Catalogued | §10 |
| 29 | Generated/hand-authored/schema-derived? | Not verifiable | §§10–11 |
| 30 | Samples and golden responses? | Current and pinned examples available | §§10.4, 13 |
| 31 | Validation present? | Observable subset assessed; internals unknown | §§8, 10.3 |
| 32 | Reusable fixture patterns? | Independently authored pack recommended | §§10.4, 13 |
| 33 | Clients/tests/manual procedures used? | Private repo inventories OS4CSAPI and manual probes | §§3, 12, 13 |
| 34 | Compatibility patterns? | Discovery/link/envelope/query/media/OAD patterns | §12 |
| 35 | Interop hazards? | Matrix and drift table | §§7, 12 |
| 36 | Reserved 014F findings? | Root cause/scoring/cross-server/current writes reserved | §12.3 |
| 37 | 014E/014F/056 handoffs? | Explicit | §§12, 15 |
| 38 | Tests in server or related work? | Server tests unknown; external corpus extensive | §13.1 |
| 39 | Untested/ambiguous/difficult behavior? | Internals, writes, SSE, lifecycle, security listed | §§9, 13, 17 |
| 40 | Candidate positive/negative/schema/conformance/golden/interoperability tests? | Complete regression corpus | §13.2 |
| 41 | Comparison targets? | OSH, CS-GO, pygeoapi, OS4CSAPI handed to 014F/056 | §§12, 15 |

---

## Appendix B. Reproducible Study Record

### B.1 Mechanical Repository Inventory

| Item | Result |
|---|---|
| Evidence repository | Sam-Bolling/csapi-server-interop-secd |
| Reviewed commit | f018fd129bf0d0d1ce75e68198e3ab4d99d937a0 |
| Default/current branch | main |
| Latest commit date | 2026-05-25 |
| Branches/tags/releases/PRs | 1 / 0 / 0 / 0 |
| Closed issue records | 60 |
| Server source files | None found |
| Detected license | None |

### B.2 August 31 Live Probe Record

All probes were read-only. Normal TLS validation failed because the certificate was expired; the evidence pass used an explicit insecure override and recorded that limitation.

| Probe | Result |
|---|---|
| Root, conformance, OAD | 200 JSON |
| OAD | 3.1.0; info.version 1.0.0; 32 paths; 37 schemas; 5 shared parameters; no security scheme |
| /collections | 200; 4 records |
| Resource counts | Systems 8; Deployments 4; Procedures 5; Sampling Features 7; DataStreams 13; Observations 11,250,822; ControlStreams 2; Commands 15 |
| /properties | 404 |
| /systemevents | 404 |
| DataStream/ControlStream schema routes | 200 JSON |
| /commands/{id}/status and /result GET | 404 HTML |
| System history | 122 events across two Systems; other sampled Systems empty |
| q known/nonsense | Both returned all 8 Systems |
| parent vs parentId | parent returned 8; parentId returned 2 children |
| invalid limit | 400 JSON ValidationError |
| missing System | 404 JSON NotFound |
| Accept/f variants | All tested System requests returned 200 application/json |

### B.3 Drift Controls

The May snapshot and report card are immutable historical evidence. The August observations supersede them only for current deployment statements. In particular, /collections and top-level Observations/Commands are now present. The report card’s D+ score was not recalculated and must not be quoted as an August assessment without a complete current rerun.

### B.4 Execution Limitations

- No current mutation or authentication-bypass probes.
- No long-held SSE connection.
- No load, concurrency, or destructive tests.
- No official ATS execution.
- No server source/build/license.
- No claim that every runtime path was exhaustively enumerated.
- Private evidence links require the user’s connected GitHub access.

---

## Appendix C. Completion and Handoff

- Research execution: complete.
- Report state: Final.
- Plan-owner acceptance: Glaux Project Lead, August 31, 2026.
- Immediate next action: execute the authorized IDR-SRV-014E iteration.
- Workflow boundary: the next plain proceed accepts/finalizes/merges IDR-SRV-014D and executes exactly IDR-SRV-014E; it does not authorize IDR-SRV-014F.
- IDR-SRV-014E has not started.

---

## Report Completion Checklist

- [x] Topic ID matches overall research plan index
- [x] Topic research plan is linked and aligned
- [x] Core and detailed questions are covered or explicitly resolved
- [x] Findings are evidence-backed with reproducible references
- [x] Normative and informative evidence are classified and not conflated
- [x] Mutable sources identify a pin or dated retrieval
- [x] Controlled, inaccessible, missing, and ambiguous evidence limitations are explicit
- [x] Source-backed findings, inference, and recommendations are distinguishable
- [x] Conflicts with accepted prior reports and historical captures are reconciled
- [x] Executive summary is independently readable
- [x] Recommendations are explicit and actionable
- [x] Risks and open questions are documented
- [x] Success criteria validation is complete
- [x] Plan-owner acceptance and acceptance date are recorded
- [x] Next handoff is explicit

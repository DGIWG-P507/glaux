# Section 056: Interoperability Test Matrix for External CSAPI Clients - Research Report

**Topic ID:** IDR-SRV-056<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-056 Interoperability Test Matrix for External CSAPI Clients](../IDR%20Plans/idr-srv-056-interoperability-test-matrix-for-external-csapi-clients.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** target qualification and capability inventory; client, browser, generated-client, GIS, ecosystem and peer-server roles; discovery/navigation/OpenAPI/schema; resource/query/representation/SensorML/SWE; dynamic data, streams, experimental Part 3, simulated tasking, security/policy/errors/DDIL; public demo; result attribution; automation, evidence, issue and retest workflows; final synthesis handoff<br>
**Methodology Used:** accepted-requirement extraction; live repository/release/branch inventory; pinned source and capability inspection; standards-to-client applicability mapping; transport/structural/semantic/workflow oracle decomposition; controlled comparison and defect-attribution design; evidence/redaction and freshness analysis; downstream synthesis<br>
**Research Time:** Approximately 46 hours of AI-assisted execution on September 16, 2026<br>
**Accepted Verification Baseline:** IDR-SRV-050 through IDR-SRV-055 independent conformance, normalized traceability, multi-layer Rust TDD, governed fixtures, envelope-bound performance and deny-by-default security/command-control test strategies<br>
**Current Target Evidence:** OS4CSAPI `ogc-client` `clean-pr` at `b55d95aa0bd9`; CSAPI Explorer at `00f1c188e057`; OWSLib `0.36.0` with master `9c94121ca2f6`; OpenAPI Generator `v7.25.0`; Playwright `v1.63.0`; QGIS `final-4_2_2`; CS-Go `v1.0.4`; 52North Connected Systems pygeoapi `6ad75aa7fbfc`; repository state checked September 16, 2026<br>
**Document Purpose:** Define a reproducible external-client interoperability matrix and evidence strategy without executing the matrix, changing external software, treating interoperability as conformance, using uncontrolled data/credentials, enabling physical commands, or claiming certification/readiness<br>
**Author:** OpenAI Codex<br>
**Date:** September 16, 2026<br>
**Last Updated:** September 16, 2026

---

## Evidence and Decision Legend

- **[N] Normative:** approved external standard or normatively incorporated artifact.
- **[A] Accepted project baseline:** accepted Glaux report or governing project decision.
- **[D] Direct documentation/source:** official client, tool, repository, release or protocol evidence.
- **[I] Implementation evidence:** observed implementation or deployment behavior; informative only.
- **[T] Test evidence:** reproducible interoperability observation with frozen client, server, profile, fixture and method.
- **[E] Analysis:** reasoned synthesis from identified evidence.
- **[P] Project recommendation:** proposed Glaux decision pending acceptance of this report.
- **[X] Explicit boundary:** excluded claim or later responsibility.

Interoperability is not conformance. A client connecting is not proof that it preserved meaning. A client limitation is not automatically a server defect, and a peer implementation's behavior is not a standards alternative. Glaux should be canonical and strict at its server boundary while testing explicitly bounded tolerant-client behavior as a separate concern. **[N,A,E,P]**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Interoperability Target Inventory Methodology
5. External Client, Tool, and Peer Implementation Inventory
6. Capability Profile Matrix
7. Interoperability Scenario Taxonomy
8. Interoperability Matrix Model
9. Discovery, Navigation, OpenAPI, and Schema Findings
10. Resource, Query, Content Negotiation, SensorML, and SWE Common Findings
11. Dynamic Data, Streaming/Event, Command-Control, Security/Policy, Error, and DDIL Findings
12. Glaux Ecosystem Component Interoperability Findings
13. Public Demo Interoperability Findings
14. Automation, CI, Manual Execution, Reporting, and Evidence Findings
15. Issue Classification, Feedback, and Retest Workflow Findings
16. Final Synthesis Handoff Matrix
17. Recommendations
18. Risks, Constraints, and Open Questions
19. Validation Against This Plan's Success Criteria
20. References

---

## 1. Executive Summary

Glaux should adopt a **capability-qualified, semantic-depth interoperability program**. The matrix selects targets because they exercise a distinct boundary, not because their names appeared in the research plan. Every run freezes client source/release, runtime and options; Glaux build, profile and public base URL; corpus/scenario digests; security identity class; transport topology; expected capability; raw and parsed evidence; and the controlling requirement/test IDs. Moving public deployments and unpinned package tags are observational evidence only. **[A,E,P]**

The first release-oriented client set is deliberately small and independent:

1. the OS4CSAPI TypeScript `ogc-client` branch pinned at `b55d95aa0bd9`, exercised in Node and a real browser;
2. OWSLib Python `0.36.0`, whose current source has explicit Connected Systems classes for feature, dynamic-data, command, event and history routes;
3. CSAPI Explorer pinned at `00f1c188e057` for browser discovery, CORS, rendering, link navigation and human-visible semantic checks;
4. OpenAPI Generator `v7.25.0`, limited to one TypeScript Fetch and one Python generation/compile plus selected runtime path per release; and
5. QGIS `final-4_2_2` for the inherited OGC API - Features/GeoJSON subset only.

`curl 8.22.0` and the independent Rust black-box harness preserve authoritative wire transcripts but are not counted as independent semantic clients. Browser `fetch`/stream probes cover CORS and SSE. Postman/Newman, HTTPie and additional generator languages are optional conveniences until they add a distinct semantic boundary. Schemathesis remains in the conformance/robustness lane defined by IDR-SRV-050, not duplicated as a client claim. **[D,E,P]**

The DGIWG `glaux-webapp`, `glaux-mobile`, `glaux-publisher`, `glaux-simulator` and `glaux-server` repositories currently contain only a README and no release. They are future contract targets, not executable evidence. Their expected consumer/producer roles should produce contract manifests now; they enter blocking interoperability tiers only after code, immutable versioning and independent tests exist. This prevents roadmap names from becoming fictional completed integrations. **[D,E,P]**

Peer-server comparison uses pinned local or otherwise controlled instances of CS-Go `v1.0.4`, an eligible pinned OpenSensorHub release, and 52North Connected Systems pygeoapi at `6ad75aa7fbfc`. The historical SECD corpus remains a high-value adversarial regression source. A shared client/scenario may run against Glaux and a peer to isolate client assumptions or standards ambiguity, but peer behavior never determines Glaux correctness and peer uptime never blocks a Glaux release. Current `main` branches and public demos are exploratory unless converted into immutable, reproducible targets. **[A,D,I,P]**

Interoperability outcome has three orthogonal axes. Execution is `PASS`, `FAIL`, `BLOCKED`, `INCONCLUSIVE` or `NOT_APPLICABLE`. Depth records transport, structural parse, semantic preservation and, where applicable, workflow/effect results independently. Attribution identifies server defect, client defect, fixture defect, harness defect, standards ambiguity, unsupported client capability, unsupported server profile or environment. “Partial” is a derived summary when an early depth passes and a later depth fails or was not tested; it is never silently promoted to pass. **[E,P]**

Discovery tests begin at the supplied root and follow typed links; hard-coded paths are separate compatibility probes. They verify canonical link relations, media types, public-origin/base-path coherence, conformance declarations, reachable OpenAPI and schema references, pagination links and profile-aware capabilities. An implementation can return a valid resource at a guessed path yet remain non-interoperable if a client cannot discover it. Proxy and CORS configuration are part of the deployed interoperability target. **[N,A,I,P]**

Resource tests require semantic completeness, not absence of exceptions. Parsers must preserve identifiers, UIDs, feature type, geometry, time, relationships, links, SensorML/SWE components, schema association, observation quality/result and command/status fields applicable to the scenario. Accept-header and `f` negotiation must yield semantically equivalent authorized views from one canonical resource. Query tests detect silently ignored filters; pagination traversals prove no gaps/duplicates and maintain filter/profile scope. **[A,I,P]**

Writes run only against disposable, uniquely namespaced fixtures with ownership markers, preflight, read-after-write, representation round trip, relationship verification and idempotent cleanup. A cleanup failure fails the workflow and quarantines the environment. CSAPI Explorer's current one-click CRUD page uses time-derived identifiers and implementation-tolerance logic and has no browser E2E runner in its demo package; it is therefore manual/semi-automated in a disposable target until a Glaux adapter supplies stable scenario IDs, test hooks and deterministic teardown. It must never run against the public demo. **[D,E,P]**

SSE interoperability proves authorization, `id`/`Last-Event-ID`, disconnect/reconnect, replay, duplicates/gaps, policy change and cursor expiry with a browser/fetch client and an independent programmatic client. The experimental Part 3 profile is tested separately with a generic MQTT 5 subscriber, generated AsyncAPI/runtime agreement and optional CS-Go compatibility adapter. It is outbound-only, version-pinned and not an OGC conformance claim. Topic or field differences are declared profile differences rather than papered over. **[A,P,X]**

Command tests are disabled in the public demo and use only the accepted `sim://` non-network effect recorder in disposable profiles. External clients may discover, submit, poll, update/cancel and observe status/results only when their declared capability and role allow it. The oracle checks semantic state and the effect ledger, not merely HTTP success. Unsupported tasking in QGIS or a generic generated client is `unsupported client capability`, not failure. No interop target may introduce a real command endpoint. **[A,P,X]**

Security/policy tests use the synthetic issuer, identities and policy bundles from IDR-SRV-055. They verify safe `401`/`403`/concealed-`404`, filtered traversal, token expiry, event revocation and absence of hidden values across client caches, UI, screenshots, traces and reports. Browser inability to attach particular credentials or mTLS behavior is recorded as applicability/capability, not worked around by weakening Glaux. DDIL tests verify standardized problem/HTTP behavior, stale/current metadata and safe reconnect; clients are not required to understand unadvertised Glaux extensions. **[A,P]**

PR jobs exercise raw reference transcripts, the pinned TypeScript and Python clients, two generated-client compile checks and a minimal disposable server scenario. Nightly jobs broaden browsers, Explorer read-only flows, queries, representations, SSE, auth/policy and safe writes. Manual/release-candidate checks cover QGIS, human semantic rendering, optional external IdP/mTLS and controlled peer comparisons. The public demo receives post-deploy, rate-bounded, read-only discovery/resource/CORS/link checks. Release evidence requires at least two independent semantic client families; one client tested in two runtimes is not two independent families. **[P]**

Each result emits `InteropCaseResultV1` inside `InteropRunV1`; target qualifications are `InteropTargetV1`. Raw request/response, parsed model and semantic assertion are the minimum ownership evidence. Screenshots, videos and Playwright traces support browser diagnosis but never stand alone. Findings are reproduced with the raw harness before attribution, filed to the owning repository only with evidence, and retested at the original and refreshed pins. Workarounds are labeled tolerance, not converted into server requirements. **[A,P]**

No unresolved issue blocks this strategy. Exact future Glaux component implementations, operational identity methods, final release-client pins and availability of reproducible peer images are implementation-time decisions. Acceptance authorizes none of IDR-SRV-057, implementation, changes to external projects, writes or scans against uncontrolled systems, physical command effects, certification or readiness claims. **[X]**

## 2. Scope and Plan Alignment

### 2.1 Completed Scope

This report completes all six authorized phases by inventorying and freshness-qualifying targets; defining capability and scenario taxonomies; selecting client-specific pairings; covering discovery through DDIL and public-demo behavior; specifying automation, evidence, classification, feedback and retest; and producing the required matrix, proofs and final-synthesis handoff.

### 2.2 Explicit Boundaries

This report does not execute interoperability tests; implement Glaux or client changes; certify a client/server; make peer behavior normative; require every target to support every capability; use live operational data or credentials; mutate uncontrolled deployments; stress public endpoints; enable inbound Part 3 or physical commands; select operational identity/infrastructure; or authorize final synthesis. **[X]**

### 2.3 Research Question Coverage

| Plan theme | Status | Evidence |
|---|---|---|
| target and peer inventory | Complete | Sections 3–6 |
| scenario taxonomy and matrix model | Complete | Sections 7–8 |
| discovery/navigation/OpenAPI/schema | Complete | Section 9 |
| resources/query/negotiation/SensorML/SWE | Complete | Section 10 |
| dynamic/stream/tasking/security/error/DDIL | Complete | Section 11 |
| Glaux ecosystem components | Complete with unavailable implementations explicit | Section 12 |
| public demo | Complete | Section 13 |
| automation, evidence and redaction | Complete | Section 14 |
| attribution, issues and retest | Complete | Section 15 |
| final synthesis handoff | Complete | Section 16 |

## 3. Evidence Base and Authority Classification

### 3.1 Controlling Evidence

| Source | Authority/use | Limitation |
|---|---|---|
| CSAPI Parts 1/2, OGC API Features, SensorML, SWE Common, HTTP, GeoJSON, OpenAPI, JSON Schema and Problem Details | published semantics, discovery, representation and protocol expectations | no external client is required to implement every optional capability |
| Draft Part 3 at `6f529a15` and accepted IDR-SRV-014H/035 | version-pinned experimental event/data compatibility target | draft, incomplete binding/ATS; no OGC conformance claim |
| IDR-SRV-001–055, especially 014A–H and 050–055 | accepted Glaux semantics, implementation lessons and verification boundaries | research designs still need executable implementation proof |
| client/tool repositories and releases | direct capability, packaging and automation evidence | claims require execution against the frozen Glaux target |
| public deployments and historical SECD/OS4CSAPI captures | realistic interoperability hypotheses and regression seeds | mutable, deployment-specific and non-normative |

### 3.2 Current Repository and Release Check

The September 16, 2026 check found OS4CSAPI `ogc-client` `clean-pr` at `b55d95aa0bd9` with package version `1.3.1-dev`, CSAPI Explorer at `00f1c188e057`, OWSLib master at `9c94121ca2f6` and release `0.36.0`, OpenAPI Generator `v7.25.0`, Playwright `v1.63.0`, QGIS `final-4_2_2`, CS-Go latest release `v1.0.4` with main at `b1fd2e0e9bd6`, OpenSensorHub master at `9a43f9ec4231`, 52North Connected Systems pygeoapi at `6ad75aa7fbfc`, and the Part 3 branch at `6f529a15bfa6`. These are report evidence pins, not permanent implementation pins. **[D]**

The five planned DGIWG component repositories are public and unarchived but each is approximately 1 KB with only `README.md` and no release. Their mission descriptions establish intended roles, not callable capability. **[D,E]**

### 3.3 Lessons Incorporated

IDR-SRV-014E/F showed that discovery can succeed while client parsers lose fields, content negotiation can select divergent stores, links and route nesting can differ, filters can be silently ignored and writes can be destructive or asymmetric. The accepted response is four-level evidence—transport, structural, semantic and workflow/effect—not a binary “connected.” IDR-SRV-014G adds public-base/proxy coherence and strict-server/tolerant-client separation. **[A,I]**

### 3.4 Authority Precedence

Published standards and accepted Glaux decisions determine expected server behavior. A frozen external client proves only what that client did. A peer comparison helps locate assumptions. A generated client tests the OpenAPI contract it consumed. None can waive a normative obligation or create one from implementation popularity. **[N,A,D,I,T]**

## 4. Interoperability Target Inventory Methodology

Each candidate receives an `InteropTargetV1` record with repository/download origin, immutable commit/tag/digest, release status/date, license, runtime/OS/architecture, install/build commands, declared and source-confirmed capabilities, authentication/transport support, automation interface, fixture and mutation safety, known deviations, maintenance/freshness, and assigned tier. An unpinned URL, `latest` container or package range cannot produce release evidence. **[P]**

Selection follows five filters:

1. **Independence:** does it exercise a separately implemented parser/navigation stack?
2. **Distinct boundary:** does it add browser/CORS, Python, GIS, generated-code, streaming, producer or peer-server evidence?
3. **Reproducibility:** can the exact target and dependencies be restored without relying on a mutable public service?
4. **Safety:** can reads/writes/commands be confined to governed fixtures and simulated effects?
5. **Semantic oracle:** can the harness inspect preserved meaning rather than only status or rendering?

Capability is established from current source/docs and then confirmed by a qualification run. README claims define candidate scope only. Each matrix cell is applicable only when both target and server profile declare the capability. This avoids an unbounded client-by-operation Cartesian product. **[D,E,P]**

When a case fails, the same request is replayed with the independent raw harness; the response is schema/semantic checked; the client's parsed state is compared; fixture and environment are verified; and only then is ownership assigned. A peer comparison is optional corroboration, not the first oracle. **[A,P]**

## 5. External Client, Tool, and Peer Implementation Inventory

### 5.1 Primary External Clients

- **OS4CSAPI TypeScript `ogc-client`:** primary programmatic client, pinned source until a stable upstream package includes the same CSAPI implementation. Test both Node and browser builds, but count them as one client family. Its typed parsers and query/navigation code make semantic field-preservation assertions possible. **[D,P]**
- **OWSLib Python:** primary independent client family. Release `0.36.0` and current source include Connected Systems classes for Systems, Procedures, Deployments, Sampling Features, Properties, Datastreams, Observations, ControlStreams, Commands, SystemEvents and SystemHistory, including several writes. It mainly returns dictionaries and its checked-in CSAPI test is OSH-oriented, so Glaux adds independent semantic assertions and does not inherit all route assumptions. **[D,E,P]**
- **CSAPI Explorer:** primary browser/human client for discovery, CORS, link navigation, parsed views, maps and diagnostics. Current source advertises nine resource types and contains a one-click CRUD smoke page, but the demo has no E2E test script/dependency and uses dynamic fixture identifiers plus server-specific tolerance. Treat reads as nightly automation after a Playwright adapter; keep destructive smoke manual/disposable until deterministic hooks exist. **[D,E,P]**
- **OpenAPI-generated clients:** contract-consumer targets, not full semantic clients. Generate exactly TypeScript Fetch and Python with `v7.25.0`, compile/package them, and run discovery plus representative read/error/auth paths. Add languages only for a named consumer need or distinct generator failure class. **[D,P]**
- **QGIS:** manual release target for the inherited OGC API - Features landing/collections/items/GeoJSON/pagination/filter subset. Current QGIS documentation describes OAPIF client support; it does not claim CSAPI SensorML/SWE/dynamic/tasking semantics. Nested-property rendering limitations are compatibility observations, not Glaux license to flatten canonical representations. **[D,A,P]**

### 5.2 Supporting Probes

`curl 8.22.0`, the Rust black-box harness and direct browser Fetch/EventSource probes provide raw independent requests, replayable transcripts and streaming controls. They are essential attribution tools but do not count as separate domain-client families. HTTPie and Postman/Newman are deferred because they initially duplicate raw HTTP coverage. Playwright `v1.63.0` is selected for browser automation/traces after stable selectors and deterministic scenarios exist; its screenshots/traces remain supporting evidence. **[D,E,P]**

### 5.3 Peer Implementations

- **CS-Go `v1.0.4`:** strongest broad Part 1/2 and experimental Part 3 comparative implementation. Run pinned local containers/scenarios where buildable. Current main is exploratory until released/pinned independently. Its topic layout, envelopes and route choices are implementation profiles, not normative alternatives. **[A,D,I]**
- **OpenSensorHub:** mature independent implementation with extensive Part 1/2, SensorML/SWE, dynamic and tasking precedent. Pin an exact tested release/source and modules; do not depend on mutable public demo availability. Experimental Part 3 work remains unreleased evidence unless separately qualified. **[A,D,I]**
- **52North Connected Systems pygeoapi:** useful independent Python/provider and representation comparison at `6ad75aa7fbfc`; proof-of-concept gaps and deployment drift make it advisory rather than a release oracle. **[A,D,I]**
- **SECD:** use its pinned historical request/response/adjudication corpus as adversarial regression seeds. A live server is optional and requires explicit permission and a stable target; historical grades are never current Glaux evidence. **[A,I]**

### 5.4 Future/Advisory Targets

Glaux Webapp, Mobile, Publisher and Simulator are future first-party contract targets. ConnectedSystemsAPI-CPP and additional OGC clients can enter after qualification if maintained and capability-relevant. Esri was not selected because no current, direct CSAPI capability was established. New target requests must identify the missing boundary and expected maintenance cost. **[D,E,P]**

## 6. Capability Profile Matrix

Legend: **P** primary, **S** supporting, **C** comparative, **F** future, **—** not expected. An entry is planned scope, not an executed pass.

| Target/pin | Discovery/links | P1 feature resources | P2 dynamic | SensorML/SWE | Query/page | Write | Stream/P3 | Tasking | Auth/policy | Automation | Role |
|---|---|---|---|---|---|---|---|---|---|---|---|
| OS4CSAPI `ogc-client` `b55d95aa` | P | P | P | P | P | P | limited | P | bearer/browser-dependent | PR/nightly | mandatory semantic family 1 |
| OWSLib `0.36.0` / `9c94121c` | P | P | P | raw/dict | P | P | — | P | generic auth | PR/nightly | mandatory semantic family 2 |
| CSAPI Explorer `00f1c188` | P | P | P | rendered/parsed | P | disposable only | limited | UI-dependent | CORS/diagnostic | nightly/manual | browser/human primary |
| generated TS Fetch `v7.25.0` | OAS-derived | representative | representative | schema-generated | representative | disposable subset | — | simulated subset | bearer scheme if described | PR/RC | OAS consumer 1 |
| generated Python `v7.25.0` | OAS-derived | representative | representative | schema-generated | representative | disposable subset | — | simulated subset | bearer scheme if described | PR/RC | OAS consumer 2 |
| QGIS `final-4_2_2` | inherited OAPIF | GeoJSON subset | — | — | OAPIF subset | not initial | — | — | client-supported methods | manual/RC | GIS compatibility |
| browser Fetch/EventSource | root/CORS | raw JSON | raw JSON | raw | query URL | controlled Fetch | SSE | simulated via Fetch | browser constraints | nightly | browser protocol probe |
| curl/Rust harness | P | P | P | raw canonical | P | P | SSE/MQTT adapter probes | simulated | full synthetic matrix | PR/all | wire/reference, not client family |
| CS-Go `v1.0.4` | C | C | C | C | C | isolated C | MQTT experimental | simulated C | implementation-specific | scheduled/manual | peer comparison |
| OSH exact qualified pin | C | C | C | C | C | isolated C | WS; MQTT experimental only if qualified | simulated C | implementation-specific | scheduled/manual | peer comparison |
| 52North pygeoapi `6ad75aa` | C | C | partial C | C | partial C | isolated C | — | limited | deployment-specific | scheduled/manual | advisory peer |
| historical SECD corpus `f018fd1` | captured | captured | captured | captured | captured | captured | limited | captured | deployment-specific | offline replay | adversarial evidence |
| Glaux Webapp/Mobile | F | F | F | F | F | F | F | F | F | future | consumer contracts |
| Glaux Publisher/Simulator | discovery | producer needs | producer needs | producer needs | — | F | F | simulated only | F | future | producer contracts |

No target receives a blanket “CSAPI compatible” label. Capability is recorded per scenario, representation, direction and profile. **[P]**

## 7. Interoperability Scenario Taxonomy

### 7.1 Scenario Families

| Family | Representative scenarios | Principal oracle |
|---|---|---|
| bootstrap | landing, conformance, service-desc/doc, collections, public base path | discoverability and link identity |
| navigation | top-level/nested/reverse links, rel/type/title, unknown/hidden relation | reachable canonical graph without guesses |
| API/schema | active OpenAPI, JSON Schema/dialect/refs, generated clients | document/runtime/profile agreement |
| feature resources | all Part 1 families, ID/UID/geometry/time/relationships | semantic field preservation |
| query/page | ID, UID, q, bbox, datetime, parent/relationship, domain filters, limit/cursor | selected set and stable traversal, no silent ignore |
| representation | JSON, GeoJSON, SensorML, SWE Common, Accept/`f`, errors | equivalent canonical meaning and correct negotiation |
| dynamic data | datastream/schema/observation/status/system event/history | schema-bound values, time/status semantics |
| writes | create/read/update/read/delete, nested routes, idempotency | durable round trip and deterministic cleanup |
| stream/replay | SSE subscribe/event/disconnect/replay/expiry/policy change | sequence, cursor, visibility and recovery |
| experimental Part 3 | AsyncAPI, MQTT 5 Resource Events/Data, duplicate/reconnect | declared Glaux profile agreement only |
| command/control | discovery, feasibility, submit/status/result/update/cancel | exact lifecycle and simulated effect ledger |
| security/policy | auth, concealment, filtered lists/links/events, token expiry | authorized semantic view and no leakage |
| error/robustness | 400/401/403/404/406/409/412/415/422/429/503 and Problem Details | client recovery without fabricated success |
| DDIL/recovery | dependency unavailable, stale/current evidence, reconnect/resnapshot | standards-safe degradation and explicit uncertainty |
| deployment/demo | TLS, CORS, proxy/base, compression, public read-only profile | deployed identity/capability coherence |

### 7.2 Depth Model

Every applicable case reports:

1. **transport:** request completed and protocol/status/media headers were processed;
2. **structural:** response parsed into the client's model without forbidden tolerance or loss;
3. **semantic:** all scenario-required facts and relationships were preserved and interpreted correctly; and
4. **workflow/effect:** for writes, streams or commands, durable state, replay and simulated effects match the model.

Passing an early depth cannot compensate for a later failure. A visually rendered item with a dropped nested SWE field is partial and fails semantic completeness for that scenario. **[A,P]**

### 7.3 Applicability and Variants

The scenario manifest binds required capability/profile, client family, data sensitivity, mutation class and automation tier. Variants cover direct versus trusted proxy/base path, anonymous versus synthetic authenticated views, default versus alternate representation, empty/single/multi/page-boundary data, connected/degraded/recovering service posture and command disabled versus simulated. Pairwise generation may cover lower-risk combinations; discovery identity, representation equivalence, security and command effects remain explicit cases. **[A,P]**

## 8. Interoperability Matrix Model

### 8.1 Result Model

`execution_outcome` is one of `PASS`, `FAIL`, `BLOCKED`, `INCONCLUSIVE`, `NOT_APPLICABLE`. Each depth is `PASS`, `FAIL` or `NOT_TESTED`. `attribution` is optional until adjudicated and then one of `SERVER_DEFECT`, `CLIENT_DEFECT`, `FIXTURE_DEFECT`, `HARNESS_DEFECT`, `STANDARDS_AMBIGUITY`, `UNSUPPORTED_CLIENT_CAPABILITY`, `UNSUPPORTED_SERVER_PROFILE`, `ENVIRONMENT`. A `partial` display is derived from mixed depth outcomes and cannot satisfy a release gate. **[P]**

### 8.2 Required Interoperability Test Matrix

All rows below are planned tests; “pending” is not an execution result.

| Matrix ID | Client/tool/peer | Client version/commit | Server version/commit | Server profile | Fixture/scenario | Operation tested | Expected behavior | Observed behavior | Result classification | Evidence artifact | Related requirement/test IDs | Issue link | Retest status | Notes/unresolved |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| INT-DISC-TS | OS4CSAPI TypeScript | `b55d95aa0bd9` | run pin | public/auth read | discovery-min | root -> conformance/API/collections | canonical links found and parsed | pending execution | pending | raw + endpoint graph + parsed model | discovery req/test IDs | none until finding | not run | Node and browser; one family |
| INT-DISC-PY | OWSLib | `0.36.0`, source pin recorded | run pin | public/auth read | discovery-min | API/collections and CS classes | independent Python discovery succeeds semantically | pending execution | pending | raw + Python object assertions | discovery req/test IDs | none | not run | detect hard-coded route assumptions |
| INT-EXPLORER | CSAPI Explorer | `00f1c188e057` | run pin | browser read | ui-discovery | connect/diagnostics/browse/map | no proxy guess; CORS/base/links and fields render | pending execution | pending | Playwright trace + raw + semantic checklist | discovery/resource IDs | none | not run | read-only automation first |
| INT-OAS-TS | OpenAPI Generator TS Fetch | `v7.25.0` | run pin | active profile | generated-client | generate, compile, discover/read/error | generated code compiles and representative calls match runtime | pending execution | pending | OAS digest + generator log + build/runtime | OAS/schema IDs | none | not run | generator limitation separately attributed |
| INT-OAS-PY | OpenAPI Generator Python | `v7.25.0` | run pin | active profile | generated-client | generate, package, discover/read/error | second language family consumes same contract | pending execution | pending | OAS digest + build/runtime | OAS/schema IDs | none | not run | no language explosion |
| INT-QGIS-OAPIF | QGIS | `final-4_2_2` | run pin | public demo/read | geo-feature | add OAPIF, list/load/filter/page | inherited GeoJSON feature subset loads with identity/geometry | pending execution | pending | version + network capture + project + checklist | OAPIF subset IDs | none | not run | no CSAPI-specific claim |
| INT-PROXY-LINK | TS/Python/Explorer | exact run pins | run pin | proxy-base | proxy-min | traverse every advertised link/page/Location/OAS server | one public origin/base path; no internal host leak | pending execution | pending | link graph + headers + browser trace | link/security IDs | none | not run | direct and trusted-proxy variants |
| INT-P1-RESOURCE | TS + Python | exact run pins | run pin | read | p1-complete | list/item/nested for all P1 families | required identity/geometry/time/relationships preserved | pending execution | pending | raw/parsed/semantic diff | P1 resource IDs | none | not run | both families required RC |
| INT-P2-RESOURCE | TS + Python | exact run pins | run pin | dynamic read | p2-complete | datastream/control/schema/observation/status/event | dynamic semantics and links preserved | pending execution | pending | raw/parsed/semantic diff | P2 resource IDs | none | not run | applicability per client method |
| INT-SENSORML | TS/Explorer/Python raw | exact run pins | run pin | read | sensorml-rich | negotiate/parse SensorML JSON/XML as supported | IDs, types, components, contacts, capabilities and links preserved | pending execution | pending | canonical fact projection | SensorML IDs | none | not run | parser support recorded per encoding |
| INT-SWE | TS/Explorer/Python raw | exact run pins | run pin | dynamic read | swe-nested | datastream/control schema and encoded values | nested DataRecord/Array, units, nil/quality/encoding preserved | pending execution | pending | schema/value semantic projection | SWE IDs | none | not run | no “parsed without exception” pass |
| INT-QUERY | TS + Python + raw | exact run pins | run pin | read/policy | query-boundary | id/uid/q/bbox/datetime/relations/domain filters | exact set equals oracle; unsupported rejected, not ignored | pending execution | pending | request + expected/actual ID set | query IDs | none | not run | silent ignore is failure |
| INT-PAGE | TS + Python | exact run pins | run pin | read/policy | page-boundary | follow next/prev/cursors under filters | no gaps/duplicates; scope/order/profile retained | pending execution | pending | page transcript + ID sequence | pagination IDs | none | not run | never construct next by assumption |
| INT-NEGOTIATE | TS/Explorer/raw | exact run pins | run pin | read | representation-equivalence | Accept and `f`; unsupported media | same authorized resource meaning; correct Vary/406/415 | pending execution | pending | headers + fact projections | negotiation IDs | none | not run | one canonical source |
| INT-WRITE-ROUNDTRIP | TS + Python | exact run pins | run pin | disposable write | namespaced CRUD | create/read/update/read/delete/retry | correct statuses, representation/relationship round trip and cleanup | pending execution | pending | mutation ledger + raw + DB/public facts | write/idempotency IDs | none | not run | quarantine environment on cleanup failure |
| INT-OBSERVATION | TS + Python | exact run pins | run pin | dynamic | observation-rich | ingest/query/latest/result/schema | times, schema association, result/quality and latest correct | pending execution | pending | raw/parsed/schema projection | observation IDs | none | not run | synthetic only |
| INT-STATUS-EVENT | TS + Python | exact run pins | run pin | dynamic | lifecycle-events | system status/events/history traversal | authoritative chronology and fields preserved | pending execution | pending | event/status sequence | status/event IDs | none | not run | history applicability explicit |
| INT-SSE | browser Fetch + programmatic probe | pinned runtime | run pin | streaming | replay-sequence | subscribe, disconnect, resume, expire/revoke | authorized sequence, allowed duplicate/no gap, explicit resnapshot | pending execution | pending | event transcript + cursor checkpoint | stream/security IDs | none | not run | EventSource auth limits recorded |
| INT-MQTT-P3 | generic MQTT 5 + optional CS-Go adapter | pinned client/CS-Go `v1.0.4` | run pin | experimental P3 `0.1` | outbound-events | AsyncAPI discover; subscribe/reconnect/duplicate | exact local profile, CloudEvents/data semantics and cursor; no inbound | pending execution | pending | AsyncAPI digest + broker transcript | Part3/profile IDs | none | not run | not OGC conformance |
| INT-CMD-DISC | TS/Explorer/Python as applicable | exact run pins | run pin | disabled/simulated | command-capabilities | discover control/command/feasibility | capability exactly matches profile/authorized view | pending execution | pending | link/OAS/capability diff | command/security IDs | none | not run | unsupported client is not server failure |
| INT-CMD-SIM | capable TS/Python client | exact run pins | run pin | simulated only | command-lifecycle | submit/poll/update/cancel/status/result | exact lifecycle and zero/one `sim://` effect | pending execution | pending | raw + state/audit/effect ledger | command IDs | none | not run | no network target scheme |
| INT-PROBLEM | TS/Python/generated/browser | exact run pins | run pin | all | problem-suite | representative 4xx/5xx/Problem Details | typed safe error retained; client does not fabricate success | pending execution | pending | raw + parsed error | HTTP/problem IDs | none | not run | malformed body separately tested |
| INT-AUTH | TS/Python/browser | exact run pins | run pin | oidc/mTLS/composite as applicable | synthetic identities | missing/valid/expired/wrong view | safe challenge/denial and supported credential flow | pending execution | pending | sanitized auth transcript | security IDs | none | not run | browser/mTLS limits are applicability |
| INT-POLICY | TS/Python/Explorer | exact run pins | run pin | policy | twin-world-public | lists/items/links/pages/events | authorized view preserved; hidden canary absent from client artifacts | pending execution | pending | twin-world + artifact scan | policy/redaction IDs | none | not run | client cache included |
| INT-DDIL | TS/Python/stream probes | exact run pins | run pin | degraded/recovering | ddil-reconnect | read/error/stale/cursor/resnapshot | standards-safe uncertainty, recovery and no authority expansion | pending execution | pending | state/timeline transcript | DDIL IDs | none | not run | extension understanding only if advertised |
| INT-DEMO | TS/Python/Explorer/QGIS subset | exact run pins | deployed digest | public demo | public-safe | read-only discovery/resources/CORS/TLS | command-disabled, no secrets, stable public links and sample meaning | pending execution | pending | sanitized post-deploy report | demo/security IDs | none | not run | no CRUD, active scan or load |
| INT-PEER-CSGO | same client/scenario harness | CS-Go `v1.0.4` | Glaux run pin + peer pin | comparison | selected divergence | execute identical safe scenario | classify assumptions; peer never oracle | pending execution | pending | paired transcripts/diff | interop study IDs | none | not run | local controlled instance preferred |
| INT-PEER-OSH | same client/scenario harness | exact qualified OSH pin | Glaux run pin + peer pin | comparison | selected divergence | discovery/representation/dynamic/tasking subset | locate client/server/profile difference | pending execution | pending | paired transcripts/diff | interop study IDs | none | not run | no mutable demo gate |
| INT-PEER-PYGEO | same client/scenario harness | `6ad75aa7fbfc` | Glaux run pin + peer pin | comparison | representation/routes | selected P1/P2 scenario | expose provider/representation assumptions | pending execution | pending | paired transcripts/diff | interop study IDs | none | not run | advisory PoC |
| INT-GLAUX-ECO | future Glaux component | no executable version | future server pin | future | component contract | consumer/producer contract suite | target must qualify before evidence can pass | not executable at research date | `BLOCKED`/`UNSUPPORTED_CLIENT_CAPABILITY` | target manifest | ecosystem IDs | none | await implementation | README is not integration evidence |

## 9. Discovery, Navigation, OpenAPI, and Schema Findings

### 9.1 Root-First Discovery

Clients start from one configured root and follow `conformance`, `service-desc`, `service-doc`, `data` and domain links with declared media types. Tests compare typed link identity, resolution, duplicate/conflicting links, relative/absolute policy and authorization visibility. A second compatibility lane may probe known direct routes, but success there cannot repair root discovery. **[N,A,P]**

Every generated URL—body link, pagination, `Location`, redirect, OpenAPI server and asynchronous description—uses the same external origin and base path. Direct and trusted-proxy deployments are separate fixtures. Untrusted forwarded headers must not alter links. Browser tests include CORS preflight/credentials/exposed headers; a development proxy bypass is recorded and cannot support a direct-browser interoperability claim. **[A,P]**

### 9.2 Conformance and Capability Truth

Declared classes and active profile capabilities are compared with sampled executable behavior. Optional or disabled classes are absent rather than advertised optimistically. Command-disabled and streaming-disabled profiles remove or safely deny corresponding affordances consistently across landing, OpenAPI/AsyncAPI, collections and links. The matrix records declaration drift as a server defect even if a hard-coded client still works. **[A,P]**

### 9.3 OpenAPI and JSON Schema

The active OpenAPI document is linted/validated, dereferenced in a hermetic cache, and compared with the route/media/security/profile inventory. It must describe actual status codes, parameter serialization, request/response media, required/nullable distinctions, discriminator/union behavior, examples and security schemes without internal URLs. External schema references are pinned and integrity checked. **[N,A,P]**

OpenAPI Generator TypeScript Fetch and Python targets must generate and compile/package with zero unreviewed patching. A checked-in patch is a measured compatibility adapter with owner and expiry, not a hidden fix. Selected runtime calls prove base URL, auth injection, parameter encoding, response/error decoding and polymorphic models. Generator failure is assigned to the server only when the OpenAPI is invalid or needlessly incompatible with the accepted contract. **[D,E,P]**

## 10. Resource, Query, Content Negotiation, SensorML, and SWE Common Findings

### 10.1 Semantic Resource Projections

Each fixture defines an implementation-independent fact projection: resource family; local ID and globally oriented UID; names/descriptions; feature type; geometry/bbox; valid time; parent/member/derived relationships; procedure/properties; links; source/provenance-visible facts; and dynamic/schema facts. Raw responses and each client model are projected to this form. Exact wire syntax is checked where normative; semantic equivalence is used where multiple representations are valid. **[A,P]**

Collection envelopes, item shapes and link targets are tested separately. A parser that drops unknown optional fields may remain usable for a narrow scenario but fails any scenario whose oracle requires those fields. Tolerance aliases such as legacy route names are opt-in compatibility tests and never replace canonical server output. **[A,I,P]**

### 10.2 Queries and Pagination

Each supported parameter has positive, empty, boundary, combined, malformed and unsupported cases. The expected ID set is derived from the governed corpus, not another server. A `200` containing the unfiltered collection when a supported filter was ignored is a failure. If a parameter is not supported, the server follows its documented standards-safe rejection/absence behavior. **[A,P]**

Clients follow server-provided page links/cursors. Tests cover default/maximum limit, zero/invalid limits, final/empty page, stable ordering, mutation between pages according to accepted cursor semantics, and policy-filtered pages. Page results are reconciled for duplicates/gaps and no hidden total/ID leakage. **[A,P]**

### 10.3 Negotiation and Representation Equivalence

Cases vary `Accept`, supported `f`, quality weights, wildcard, conflicting selectors, request `Content-Type` and unsupported media. They check `Content-Type`, `Vary`, `406`/`415` and correct problem documents. JSON, GeoJSON, SensorML and supported alternate encodings must originate from one canonical resource and preserve the fixture's required facts. An empty alternate store or divergent identifier/relationship is a semantic failure. **[N,A,I,P]**

### 10.4 SensorML and SWE Common

SensorML fixtures contain identifiers, classifications, contacts/documents, capabilities/characteristics, inputs/outputs/parameters, positions, components/connections and time/validity sufficient to expose shallow parsers. SWE fixtures include nested `DataRecord`/`DataArray`, scalar types, units, nil values, quality, constraints, encodings and schema-to-value association. Each client has an applicability mask by encoding; unsupported XML or binary is a declared capability gap, not silently counted as a pass. **[N,A,P]**

## 11. Dynamic Data, Streaming/Event, Command-Control, Security/Policy, Error, and DDIL Findings

### 11.1 Dynamic Data and Writes

Datastream/control schemas are retrieved before interpreting Observation/Command values. Observation tests cross-check phenomenon/result time, feature of interest, result/quality and latest/time-range semantics. System status, events and history use authoritative ordering and identifiers; a client must not collapse event resources into transient notifications. **[A,P]**

Mutation workflows use disposable per-run namespaces, unique deterministic IDs, an ownership marker and a manifest of created resources. Preflight proves target/reset capability; create is followed by canonical read, update by semantic diff, retry by idempotency check and delete by absence/relationship cleanup. Dependency order and cascade expectations are explicit. A failed cleanup quarantines/reset-restores the target before another run. **[A,P]**

### 11.2 SSE and Experimental Part 3

SSE cases use both browser Fetch streaming and an independent programmatic probe. They record subscription request, status/media, event type/data/id, logical sequence, reconnect input, duplicates/gaps, policy version and resnapshot. Native browser `EventSource` cannot set arbitrary authorization headers consistently; it is tested only where its credential model is applicable. Server security is never weakened to satisfy it. **[D,E,P]**

The MQTT lane targets the Glaux experimental profile selected by IDR-SRV-035: MQTT 5, outbound-only, generated AsyncAPI, exact topic/format profile, authorized Resource Events/Data, QoS duplicate handling and Glaux logical cursor. A generic standards MQTT client is primary; a CS-Go compatibility adapter is comparative. The official Part 3 draft pin and Glaux profile version are evidence. No inbound observation/command, unspecified OSH/CS-Go topic equivalence or OGC conformance is claimed. **[A,D,I,P,X]**

### 11.3 Command and Control

Command-disabled profiles must be correctly understood by discovery-capable clients without broken navigation. Simulated profiles test ControlStream/schema, feasibility, submit, status/result, update/cancel and error paths only for clients that implement them. The test oracle joins the public response, command state journal, audit evidence and `sim://` effect ledger. HTTP `2xx` without the expected authoritative state/effect meaning fails. **[A,P]**

No external client obtains network coordinates for a target; command payloads and targets are synthetic. QGIS, generic OAPIF and clients without tasking are `NOT_APPLICABLE` or `UNSUPPORTED_CLIENT_CAPABILITY`, never forced through improvised routes. **[A,X]**

### 11.4 Authentication, Policy, and Errors

The local issuer/JWKS and synthetic identities from IDR-SRV-055 cover anonymous, reader, privileged, publisher, operator, submitter, approver and denied views. Clients are tested only with supported credential mechanisms; secrets are injected at runtime and excluded from command lines, screenshots, traces and reports. Token expiry, refresh/re-auth behavior and long-lived stream termination are captured without raw token evidence. **[A,P]**

Twin-world policy scenarios compare the client's complete observable state: models, caches, UI, map, counts, pages, links, errors, events and exported artifacts. Concealed values/canaries must be absent. Client caching must partition by authorized view and invalidate on policy/profile change according to declared semantics. **[A,P]**

Problem Details tests cover common HTTP and domain errors, unknown fields/media, conflict/precondition, rate/capacity and dependency outage. A client may provide a friendly message but must preserve status/type/correlation-safe fields and never turn failure into empty success. HTML/text fallbacks are tested only when advertised. **[N,A,P]**

### 11.5 DDIL and Recovery

Fault-controlled scenarios disconnect identity/policy, database/broker or client links and move through recovering state. Clients observe standards-safe status/problems, declared current/stale/unknown metadata and cursor/snapshot requirements. They retry only idempotent or explicitly safe operations, respect `Retry-After` where applicable, and never resubmit an ambiguous mutation as an unrelated write. Glaux-specific metadata is required only when advertised in that profile and understood by the target. **[A,P]**

## 12. Glaux Ecosystem Component Interoperability Findings

### 12.1 Current Availability

The Webapp, Mobile, Publisher and Simulator repositories have mission statements but no executable source, tests, tags or releases. They remain `BLOCKED` as external targets. The matrix must not invent versions, supported resources, auth modes or wire behavior. **[D,X]**

### 12.2 Contract-First Entry Criteria

Before entering interoperability gates, each component must publish an `InteropTargetV1`, immutable build/source pin, license/dependency manifest, configuration and profile, supported capability/direction table, synthetic fixture use, safe reset/cleanup and machine-readable result interface. Consumer components add semantic model assertions; producer components add source identity, idempotency, schema/encoding and backpressure assertions. **[A,P]**

The Webapp and Mobile first contract should cover discovery, policy-safe map/resource rendering, observations/status/events, reconnect and command-disabled behavior. Publisher first covers authorized source registration/ingestion, validation/quarantine, idempotency and schema association. Simulator first covers deterministic telemetry/event generation and `sim://` tasking only. None receives privileged bypasses unavailable to external clients. **[A,P]**

## 13. Public Demo Interoperability Findings

The public demo is a separate, immutable, command-disabled profile with public synthetic data. Post-deploy and scheduled checks are rate bounded and read only: TLS/public origin, root/conformance/API/collections, representative P1/P2 resources, query/page, negotiation, CORS, links, problem behavior and optional SSE. CSAPI Explorer connects directly when CORS permits; a proxy-assisted run is labeled separately. OS4 Node/browser and OWSLib run the same safe subset. QGIS is a periodic manual inherited-Features check. **[A,P]**

The demo suite never runs CRUD, active DAST, abuse/load, command submission, broker publication or uncontrolled schema fetch. It uses no secret credential and publishes only sanitized results. A transient hosting/network failure is `BLOCKED/ENVIRONMENT` until reproduced; it is not automatically a server regression. Deployment digest, timestamp, DNS/TLS observations and public fixture manifest bind the result. **[A,P]**

Public screenshots are presentation artifacts. Before publication, automated canary/secret scans and human review verify that URLs, headers, payloads, maps and browser storage contain only intended synthetic public information. **[A,P]**

## 14. Automation, CI, Manual Execution, Reporting, and Evidence Findings

### 14.1 Execution Tiers

| Tier | Targets and scope | Gate semantics |
|---|---|---|
| PR | raw harness; OS4 Node; OWSLib; generated TS/Python compile and representative calls; deterministic discovery/resource/query/error plus safe disposable mutation subset | blocking for supported mandatory cells; no network dependency on public peers |
| nightly | broader TS/Python matrices; browser builds; Playwright Explorer read flows; SensorML/SWE; auth/policy; SSE; full safe writes; DDIL faults; drift checks | reproducible semantic regression blocks affected profile; target-infrastructure issue triaged separately |
| scheduled comparative | pinned local CS-Go/OSH/pygeoapi scenarios and historical SECD replay | advisory attribution/drift evidence; peer difference cannot waive Glaux failure |
| public-demo post-deploy/daily | rate-bounded anonymous read/CORS/link/resource/query/SSE subset | deployment gate; environment failures separated |
| manual | QGIS, Explorer visual/CRUD disposable, mTLS/interactive identity, usability and selected peer observations | checklist plus raw semantic evidence required |
| release candidate | refreshed/frozen target manifest; complete applicable matrix; two independent semantic families; browser, generated clients, QGIS subset; all deviations reviewed/retested | incomplete mandatory cell, unresolved server defect or stale/unknown pin blocks affected release claim |
| future operational | actual Glaux ecosystem components, operational identities/policies and authorized field/client assessments | separately governed; no IDR readiness inference |

### 14.2 Orchestration

The harness starts an immutable Glaux image/profile and disposable database, loads the governed corpus, waits on readiness rather than sleeps, starts any issuer/broker/proxy, runs target adapters with bounded timeouts, collects evidence, performs cleanup/reset, and verifies no residual writes/effects. Target adapters normalize execution only; they do not normalize away semantic differences. Network egress is denied except explicit local services. **[A,P]**

Browser automation uses Playwright only after the Explorer/Glaux UI exposes stable semantic selectors and deterministic connection/scenario controls. The adapter captures network events and parsed/UI assertions; pixel snapshots are reserved for stable visual semantics and reviewed tolerances. Current Explorer CRUD remains manual/disposable because its source shows dynamic IDs and no demo E2E runner. **[D,E,P]**

### 14.3 Evidence Records

`InteropCaseResultV1` includes matrix/test/requirement IDs; target manifest ID/digest; client source/version/runtime/options; server image/source/config/profile/schema/migration digests; fixture/scenario/policy/identity class; public base/topology; start/end; exact request/response evidence references; parsed semantic projection; depth outcomes; execution outcome; provisional/final attribution and rationale; issue/deviation; sensitivity; tool versions; reviewer and retest lineage.

`InteropRunV1` adds run/environment identity, target set, capability/applicability snapshot, orchestration versions, case list, summary derived from cases and sanitized artifact manifest. `InteropTargetV1` holds qualification and freshness. Canonical JSON and digests follow IDR-SRV-050/051. **[A,P]**

Raw request, raw response, parsed result/exception and semantic assertion are the minimum failure bundle. Browser traces/screenshots/video, server logs and peer comparisons are supporting evidence. Raw credentials, cookies, private claims, hidden policy facts, internal topology and full sensitive payloads are prohibited; restricted evidence uses separate access and retention. **[A,P]**

### 14.4 Implementation Proofs Required

1. A target without immutable source/release, capability manifest, license/runtime and freshness cannot enter a release gate.
2. The pinned TypeScript and OWSLib Python clients independently discover Glaux from root links and preserve the mandatory semantic projection.
3. The exact active OpenAPI generates and compiles/packages TypeScript Fetch and Python clients, whose representative runtime calls match the deployed profile.
4. Direct-browser and CSAPI Explorer tests prove CORS, public-base/proxy links, navigation and semantic rendering without a hidden development proxy.
5. Query and pagination cases detect silently ignored filters, gaps, duplicates, scope loss and policy-visible count/navigation leakage.
6. JSON/GeoJSON/SensorML/SWE negotiation preserves required facts across supported representations and rejects unsupported media correctly.
7. Disposable CRUD proves create/read/update/read/delete, idempotency, relationship/representation round trip and verified cleanup under both mandatory client families.
8. SSE proves authorization, sequence/cursor, reconnect/replay, expiry/resnapshot and policy change with browser and programmatic clients.
9. The experimental MQTT 5 lane proves AsyncAPI/runtime agreement, outbound-only authorization and declared Glaux-profile semantics without claiming Part 3 conformance.
10. Capable clients complete simulated command workflows with exact lifecycle/audit and zero/one non-network effect; incapable clients are classified without weakening the server.
11. Security, policy, DDIL and public-demo cases produce correct views/recovery while canaries and credentials remain absent from all client/evidence artifacts.
12. A seeded server defect, client defect, fixture defect, harness defect and ambiguity are each correctly reproduced, attributed, issued, fixed/accepted and retested with preserved lineage.

## 15. Issue Classification, Feedback, and Retest Workflow Findings

### 15.1 Adjudication Workflow

1. Freeze server, client, profile, fixture and environment manifests; preserve sanitized raw and parsed evidence.
2. Reproduce with the same adapter and seed; flaky/non-reproducible results remain `INCONCLUSIVE`.
3. Replay the request with the raw harness and validate the response against normative/accepted semantics.
4. Compare the client's parsed projection and, if helpful, a second client or pinned peer.
5. Assign one primary attribution with evidence; record contributing factors separately.
6. File an issue in the owning repository or a standards-ambiguity/deviation record, linking only safe artifacts.
7. Retest the exact failing pins after the fix, then refresh the affected target pin and run impact-selected neighbors.
8. Supersede rather than overwrite evidence; close only with linked retest or explicit accepted limitation.

### 15.2 Ownership Rules

| Classification | Minimum evidence | Normal owner/action |
|---|---|---|
| server defect | canonical expectation + raw request/response + independent reproduction | Glaux issue and regression test |
| client defect | valid response + lost/misinterpreted semantic projection | client upstream issue; optional bounded tolerance adapter |
| fixture defect | source/provenance/schema/oracle inconsistency | corpus issue; invalidate dependent evidence |
| harness defect | adapter/orchestration/assertion mismatch | harness issue; rerun affected cases |
| standards ambiguity | competing plausible readings with exact clauses | project deviation/OGC issue; no popularity vote |
| unsupported client capability | qualification/applicability shows absent function | record limitation; no failure unless capability was declared |
| unsupported server profile | capability intentionally absent and correctly advertised | not applicable/expected limitation |
| environment | DNS/TLS/CORS/availability/tool installation external to tested semantic claim | repair environment and rerun; may be deployment defect if Glaux-owned |

A workaround can restore user experience without changing ownership. Glaux may emit an optional documented compatibility alias only if it does not violate canonical output, security or maintainability; the canonical path remains tested. **[A,P]**

### 15.3 Freshness and Retest

Target manifests expire on source/release change, dependency/runtime incompatibility, standards/profile delta or a declared maximum review interval. Automated monitors report drift but do not auto-promote a new pin. Security fixes can trigger urgent qualification. Retest status is `NOT_RUN`, `AWAITING_FIX`, `FIXED_AT_ORIGINAL_PIN`, `REFRESHED_PASS`, `ACCEPTED_LIMITATION`, `SUPERSEDED` or `CANNOT_REPRODUCE`, always linked to prior evidence. **[P]**

## 16. Final Synthesis Handoff Matrix

| Recipient | Required handoff | Must not infer |
|---|---|---|
| IDR-SRV-057 final synthesis | qualified inventory, capability matrix, result/depth/attribution model, required matrix, tiers, evidence records, twelve proofs, risks and blocked ecosystem targets | any case was executed or interoperability/readiness achieved |
| implementation roadmap | target adapters, manifests, scenario packs, browser hooks, generated-client jobs, disposable writes, result schema and adjudication workflow | external projects will remain unchanged or available |
| conformance/traceability | separate normative tests and interop edges; exact requirement/test/fixture/evidence/deviation IDs | an interop pass waives conformance or vice versa |
| fixtures/security/performance | semantic projections, target-safe corpus, identity/policy classes, redaction and reusable non-load scenario IDs | operational data/credentials or public stress are allowed |
| Glaux component projects | entry criteria and initial consumer/producer contract scopes | README placeholders are implemented integrations |
| external/community feedback | minimal sanitized reproducer, ownership rationale and exact pins | authority to modify or test third-party systems destructively |

Acceptance of this report would complete all prerequisite topic research for IDR-SRV-057 but would not authorize that final synthesis until the next explicit `proceed`. **[X]**

## 17. Recommendations

1. Adopt `InteropTargetV1`, `InteropCaseResultV1` and `InteropRunV1` with separate execution, depth and attribution axes.
2. Require OS4CSAPI TypeScript and OWSLib Python as the first two independent semantic client families; pin exact source/releases.
3. Use CSAPI Explorer as the primary browser/human target, adding a deterministic Playwright adapter before automation; keep current CRUD smoke disposable/manual.
4. Generate only TypeScript Fetch and Python clients initially and gate compilation plus representative runtime behavior.
5. Limit QGIS claims to inherited OGC API - Features/GeoJSON behavior and keep it manual/RC.
6. Treat curl/Rust transcripts as attribution oracles, not extra semantic-client votes.
7. Require root-first discovery, public-base/proxy coherence, declaration/runtime agreement and semantic field preservation.
8. Make query/page, negotiation, SensorML/SWE, safe CRUD, SSE, error and security/policy scenarios mandatory according to profile/applicability.
9. Keep MQTT/Part 3 a version-pinned outbound experimental lane with generic MQTT evidence and optional peer adapters, separate from conformance.
10. Keep every command interoperability case simulated and reconcile state, audit and effect evidence.
11. Keep peer comparisons reproducible and advisory; never block on mutable public demos or derive requirements from peer popularity.
12. Keep Glaux ecosystem components future/blocked until each publishes executable immutable contract evidence.
13. Run safe PR/nightly/local suites, read-only public-demo checks and manual/RC GIS/visual/identity checks as separately labeled tiers.
14. Use the adjudication/retest workflow and never publish credentials, hidden policy facts or raw sensitive captures.

## 18. Risks, Constraints, and Open Questions

| Risk/decision | Treatment | State |
|---|---|---|
| client names mistaken for current capabilities | live source/release qualification and applicability masks | Resolved strategy |
| one client passes by dropping meaning | transport/structural/semantic/workflow depth | Resolved strategy |
| client assumption becomes server contract | normative/accepted oracle plus raw replay before attribution | Resolved strategy |
| full client-by-operation matrix explodes | distinct-boundary selection and capability-based cells | Resolved strategy |
| generated-client languages multiply | exactly TS Fetch + Python until consumer need | Resolved strategy |
| Explorer CRUD mutates unsafe target | manual/disposable until deterministic adapter and cleanup | Resolved strategy |
| peer/public endpoint drift blocks release | pinned local peers advisory; public targets non-gating | Resolved strategy |
| Part 3 differences overclaimed | local experimental profile, exact pin and no OGC badge | Resolved strategy |
| first-party placeholders treated as integrations | blocked/future with explicit entry criteria | Resolved strategy |
| screenshots/tokens/policy leak | raw evidence classification, redaction scan and human review | Resolved strategy |
| QGIS/OWSLib/OS4 client changes after this report | implementation-time qualification and drift workflow | Constraint |
| exact release pins and container digests | freeze when executable server/CI baseline exists | Open implementation decision |
| exact stable Explorer selectors/test API | coordinate or adapt when browser automation is implemented | Open implementation decision |
| reproducible OSH/CS-Go/pygeoapi images/licenses | qualify before scheduled comparative tier | Open implementation decision |
| operational clients/identity methods | future stakeholder/field evidence | Out of scope |

No open item prevents implementation of the deterministic local matrix. They limit claims for future client versions, first-party components, peer reproducibility and operational deployment. **[P,X]**

## 19. Validation Against This Plan's Success Criteria

| Success criterion | Result | Evidence |
|---|---|---|
| external client/tool/ecosystem/peer inventory with sources and traceability | Met | Sections 3–6 |
| scenario taxonomy and matrix model | Met | Sections 7–8 |
| capability profiles and client-specific strategy | Met | Sections 5–6, 9–13 |
| discovery through public-demo functional coverage | Met | Sections 9–13 |
| defect/ambiguity/capability/profile/fixture/test classifications | Met | Sections 8 and 15 |
| automated/semi/manual/CI/nightly/demo/RC tiers | Met | Section 14 |
| evidence, redaction, feedback and retest | Met | Sections 14–15 |
| implementation/community lessons non-normative | Met | Sections 3.3–3.4 and 5.3 |
| decision-usable bounded recommendations | Met | Sections 17–18 |
| explicit final synthesis handoff | Met | Section 16 |
| explicit reproducible references | Met | Section 20 |

### Report Completion Checklist

- [x] Required twenty content areas completed.
- [x] Required fifteen-column interoperability matrix completed.
- [x] Current target/source/release inventory completed.
- [x] Capability, scenario, profile and applicability model completed.
- [x] Execution, semantic-depth and attribution classifications separated.
- [x] Automated, semi-automated, manual, public-demo and release tiers completed.
- [x] Twelve implementation proofs and final-synthesis handoff completed.
- [x] No real data/credential, uncontrolled mutation, physical effect, conformance substitution or readiness claim authorized.

## 20. References

### Governing Standards and Accepted Project Sources

- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [Glaux Server Goal and Definition](https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md)
- Accepted IDR-SRV-001 through IDR-SRV-055 reports in this directory, especially IDR-SRV-014A through IDR-SRV-014H and IDR-SRV-050 through IDR-SRV-055.
- [OGC API - Connected Systems - Part 1](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2](https://docs.ogc.org/is/23-002/23-002.html)
- [Draft Part 3 branch at reviewed commit `6f529a15`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/6f529a15bfa63259febc3620378d3e5a06305333/api/part3)
- [OGC API - Features - Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [JSON Schema](https://json-schema.org/specification)
- [RFC 7946: GeoJSON](https://www.rfc-editor.org/rfc/rfc7946)
- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [RFC 9457: Problem Details](https://www.rfc-editor.org/rfc/rfc9457)

### External Client and Tool Sources

- [OS4CSAPI `ogc-client` reviewed commit](https://github.com/OS4CSAPI/ogc-client/tree/b55d95aa0bd9f0cacf1e8b37953fde1ba14523f9)
- [OS4CSAPI phase-9 client corpus at accepted pin](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230)
- [CSAPI Explorer reviewed commit](https://github.com/OS4CSAPI/ogc-csapi-explorer/tree/00f1c188e05738ee03390fd95f09d351e073a9c3)
- [CSAPI Explorer live deployment](https://ogc-csapi-explorer.pages.dev/)
- [OWSLib](https://github.com/geopython/OWSLib) and [Connected Systems client source](https://github.com/geopython/OWSLib/blob/9c94121ca2f6cedf9d50b218a3882c06698c783e/owslib/ogcapi/connectedsystems.py)
- [OWSLib documentation](https://owslib.readthedocs.io/)
- [OpenAPI Generator `v7.25.0`](https://github.com/OpenAPITools/openapi-generator/releases/tag/v7.25.0)
- [TypeScript Fetch generator documentation](https://openapi-generator.tech/docs/generators/typescript-fetch/)
- [Playwright `v1.63.0`](https://github.com/microsoft/playwright/releases/tag/v1.63.0)
- [QGIS OGC client documentation](https://docs.qgis.org/4.2/en/docs/user_manual/working_with_ogc/ogc_client_support.html)
- [curl documentation](https://curl.se/docs/)
- [Schemathesis documentation](https://schemathesis.readthedocs.io/)

### Peer and Ecosystem Sources

- [Connected Systems Go `v1.0.4`](https://github.com/SomethingCreativeStudios/connected-systems-go/releases/tag/v1.0.4)
- [OpenSensorHub Core](https://github.com/opensensorhub/osh-core)
- [52North Connected Systems pygeoapi reviewed commit](https://github.com/52North/connected-systems-pygeoapi/tree/6ad75aa7fbfc76517c8a24c21446471845007855)
- [SECD interoperability corpus reviewed commit](https://github.com/Sam-Bolling/csapi-server-interop-secd/tree/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0)
- [Glaux Webapp](https://github.com/DGIWG-P507/glaux-webapp)
- [Glaux Mobile](https://github.com/DGIWG-P507/glaux-mobile)
- [Glaux Publisher](https://github.com/DGIWG-P507/glaux-publisher)
- [Glaux Simulator](https://github.com/DGIWG-P507/glaux-simulator)

---

**Research Result:** Complete; report awaiting Glaux Project Lead review.<br>
**Acceptance Boundary:** Acceptance would establish the interoperability planning baseline only. It would not execute tests, authorize IDR-SRV-057, change external projects, permit writes/scans/load against uncontrolled services, enable physical command effects, or claim conformance, certification, production interoperability or readiness.

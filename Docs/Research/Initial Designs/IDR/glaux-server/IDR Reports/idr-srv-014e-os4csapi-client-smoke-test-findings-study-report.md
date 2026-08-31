# Section 014E: OS4CSAPI Client Smoke Test Findings Study - Research Report

**Topic ID:** IDR-SRV-014E<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-014E OS4CSAPI Client Smoke Test Findings Study](../IDR%20Plans/idr-srv-014e-os4csapi-client-smoke-test-findings-study.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 29 detailed questions; all six methodology phases, nine success criteria, seventeen required content areas, and fifteen minimum findings-matrix fields are validated<br>
**Methodology Used:** Pinned repository inventory; smoke-report, issue, capture, fixture, client-code, and test review; local type-check and targeted integration replay; comparison with accepted IDR-SRV-006 through IDR-SRV-014D; evidence-ownership and freshness adjudication; bounded synthesis<br>
**Research Time:** Approximately 4.5 hours of AI-assisted elapsed execution on August 31, 2026<br>
**Primary Source Baseline:** `OS4CSAPI/ogc-client-CSAPI_2` branch `phase-9`, commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`, package `@camptocamp/ogc-client` version `1.3.1-dev`<br>
**Supporting Resources:** 28 live-server smoke reports, 31 demo-app finding records, 21 phase-9 pygeoapi wire-capture artifacts, issue #188, client source and integration fixtures, and accepted Glaux reports IDR-SRV-006 through IDR-SRV-014D<br>
**Document Purpose:** Establish a source-pinned, ownership-classified client smoke-test findings baseline for Glaux API behavior, validation, conformance, fixtures, and external-client testing without promoting client assumptions or implementation quirks into server requirements<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Pending Glaux Project Lead review<br>
**Acceptance Date:** Pending<br>
**Date:** August 31, 2026<br>
**Last Updated:** August 31, 2026

---

## Reading Guide

| Label | Meaning |
|---|---|
| N | Normative OGC standard or accepted Glaux standards-baseline decision |
| C | Client source or test at the pinned phase-9 commit |
| S | Dated smoke report or raw capture at the pinned commit |
| G | GitHub issue or pull-request record |
| H | Historical live-server observation that was not freshly reproduced for this topic |
| R | Locally reproduced client test or type-check result on August 31, 2026 |
| X | Analyst synthesis or Glaux recommendation, never a standards claim |

“Server finding” means the wire behavior was attributed to a named deployment in the source; it does not establish behavior of every version of that implementation. “Client finding” means the tested library’s assumptions, parser, types, URL builder, or documentation caused or amplified the outcome. “Reproduced” is reserved for evidence replayed in this study. Historical observations are not silently promoted to current deployment facts.

The controlling rule is **strict server, tolerant client**. Robust client workarounds are useful ecosystem evidence, but they do not authorize Glaux to emit noncanonical links, misleading media types, incomplete declarations, invalid shapes, or ambiguous success responses.

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Smoke-Test Source Inventory and Evidence Classification
4. Tested Servers, Client Versions, Branches, and Test Conditions
5. Client-Server Behavior Findings
6. Schema, Model, and Representation Findings
7. Dynamic-Data, Status, Event, and Tasking Findings
8. Cross-Implementation Interoperability Findings
9. Client-Library Limitation Versus Server-Behavior Classification
10. Standards Ambiguity and Documentation-Gap Findings
11. Test Strategy and Fixture Implications
12. Lessons for Glaux Server
13. Downstream Topic Handoff Matrix
14. Recommendations
15. Risks, Constraints, and Open Questions
16. Validation Against the Research Plan
17. References
Appendix A. Detailed Question Ledger
Appendix B. Reproducible Study Record
Appendix C. Completion and Handoff

---

## 1. Executive Summary

The OS4CSAPI client corpus is valuable because it records repeated contact between one TypeScript client and materially different CSAPI deployments rather than only testing an idealized schema. By Smoke Test 26, the matrix included public OSH, 52°North, an OS4CSAPI OSH deployment, and Connected Systems Go. Phase 9 added a controlled `connected-systems-pygeoapi` deployment with captured requests and responses [S]. The resulting evidence spans discovery, links, navigation, queries, negotiation, envelopes, SensorML, SWE Common, dynamic data, tasking, CRUD, errors, pagination, fixtures, and drift.

Its strongest result is not that Glaux should mimic any tested server. It is that interoperability fails at uncontrolled boundaries: discovery metadata does not match reachable routes; link relations differ; advertised conformance and OpenAPI under-report or overstate behavior; content negotiation selects different stores or is ignored; equivalent resources use different envelopes and reference forms; pagination defaults vary; nested versus top-level operations vary; asynchronous commands produce different success semantics; and client parsers mistake one observed shape for the complete model [S,C].

Issue #188 is the clearest ownership example. A second independent consumer reproduced six null dereferences when an OSH landing page did not yield a `data` link. The immediate crash was a client null-guard defect. Accepting `rel="collections"` was a possible tolerant-client extension. OSH’s noncanonical discovery relation remained a server interoperability issue. No one finding erased the others [G,C]. Glaux should emit canonical discovery links and validate its identity graph; external clients may separately choose tolerant recognition.

The corpus also documents why parsed tool output is not wire evidence. A Phase 7 report initially classified an OSH `observedProperties` value as a bare object. Raw `curl` later proved the server returned an array and PowerShell had unwrapped the single element. The report retracted the finding, retained a harmless defensive helper, and recorded the lesson [S]. Glaux test evidence must preserve raw status, headers, and bytes before semantic parsing.

The server-specific observations remain useful but bounded. OSH variants used full URIs or CURIEs, lowercase `controlstreams`, `?f=` negotiation, missing response counts, nested command access, and in some cases broken relationship routes. The 52°North demo exhibited dual-provider negotiation and later severe degradation. Connected Systems Go used hybrid Part 1/Part 2 envelopes, `@link`/`@id` references, a default page size of 10, absolute pagination links, and a working top-level commands collection. The controlled pygeoapi deployment exposed incomplete OpenAPI and conformance metadata, endpoint-specific write media types, public-base-URL leakage, write/read asymmetry, broken deployment relationships, and a sampling-feature round-trip failure [S,H]. These describe versions and deployments, not standards alternatives.

The locally replayed phase-9 client baseline passed five integration suites and 86 tests, and `tsc --noEmit` passed [R]. That demonstrates internal fixture consistency for discovery, navigation, observations, commands, schemas, and pagination. It does not reproduce mutable live deployments or prove server conformance. The installed dependency graph reported vulnerabilities; dependency audit was outside this topic and does not alter the behavioral evidence.

Glaux should therefore build one canonical contract and test it three ways: standards-derived conformance tests; exact raw-wire golden tests for every representation and negative response; and black-box runs from independent clients. Test fixtures should deliberately cross product dimensions—server family, endpoint, envelope, link form, identifier style, temporal sentinel, media type, pagination, sync/async result, empty/null/malformed shape, and relationship topology—rather than snapshotting one live demo. Every failure record should preserve provenance and separate server, client, fixture, harness, documentation, and standards ownership.

---

## 2. Scope and Plan Alignment

### 2.1 Included Scope

- The pinned phase-9 client repository, historical smoke reports, governance references, demo finding records, phase-9 experiment material, raw pygeoapi captures, source, fixtures, and integration tests.
- Issue #188 as an independently reported integrated reproduction.
- Server observations for OSH, 52°North, OS4CSAPI-OSH, Connected Systems Go, and the controlled pygeoapi deployment where the corpus identifies the target and conditions.
- Findings relevant to Glaux discovery, navigation, queries, negotiation, representations, schemas, dynamic data, tasking, errors, validation, documentation, conformance, and test design.
- Comparison against accepted Glaux normative and implementation-study reports.

### 2.2 Excluded Scope

- Treating any smoke report, client workaround, demo behavior, or issue opinion as normative authority.
- Re-running authenticated OSH mutation tests, bypassing expired certificates, altering public deployments, load testing, or executing commands.
- Re-adjudicating SECD-specific interoperability evidence; IDR-SRV-014F owns that corpus.
- Reviewing OS4CSAPI organization discussions for lessons learned; IDR-SRV-014G owns that corpus.
- Turning findings directly into engineering tickets or selecting final implementation mechanisms for downstream topics.
- Claiming current health from historical live-server reports.

### 2.3 Plan Coverage

| Core question | Determination | Coverage |
|---|---|---|
| Q1 Relevant findings | Discovery, links, negotiation, shape, query, dynamic-data, tasking, errors, metadata, fixtures, and drift findings consolidated | §§5–8 |
| Q2 Ownership | Server, client, fixture, harness/tool, documentation, standards ambiguity, and unresolved classes separated | §§3, 9–10 |
| Q3 Affected Glaux areas | API contract, models, validation, conformance, documentation, and tests mapped | §§11–14 |
| Q4 Derived tests | Positive, negative, schema, golden, regression, live-canary, and external-client tests defined | §11 |
| Q5 Downstream handoffs | Finding families mapped without pre-deciding downstream designs | §13 |

All 29 detailed questions are individually closed in Appendix A.

---

## 3. Smoke-Test Source Inventory and Evidence Classification

### 3.1 Reproducible Baseline

| Source | Pin / count | Evidence role | Limitation |
|---|---:|---|---|
| `OS4CSAPI/ogc-client-CSAPI_2` | `phase-9` at `754411897173c2ec4debaa9bcf4ed9e0f8a9e230` | Controlling client corpus | Historical snapshot, not current branch head |
| `package.json` | `@camptocamp/ogc-client` `1.3.1-dev`; Node `>=20` | Client identity and toolchain | Development version, not a release tag |
| Live-server smoke reports | 28 files, post-Phase 2.1 through Phase 8A | Longitudinal live observations and corrections | Mutable servers; heterogeneous test conditions |
| Known-server-quirks reference | Version 1.0, February 18, 2026 | Consolidated OSH/52°North behaviors through ST18 | Later reports supersede some state |
| Demo-app finding records | 31 substantive Markdown files | Client API, parser, typing, docs, and CRUD gaps | Many are proposals, duplicates, rejected, or deferred |
| Phase-9 research set | 3 reports plus seeder | Independent-consumer lesson propagation and controlled experiment | Planned hypotheses are not observations by themselves |
| Oracle pygeoapi captures | 21 files dated May 9, 2026 | Raw/replayable response corpus | One configured deployment, known public-URL misconfiguration |
| Integration specs | 5 suites / 86 tests | Fixture-supported workflow behavior | Mock/fixture tests, not live conformance tests |
| GitHub issue #188 | Opened May 3, 2026; now closed | Independent reproduction and code anchors | Reporter’s standards characterization is informative only |
| Glaux upstream-history register | Version 1.8; official `master` checked August 31, 2026 | Relevant link, media, schema, route, query, OAS, and lifecycle history | Supporting context only; no new material update required for this topic |
| Accepted Glaux reports 006–014D | Repository state through August 31, 2026 | Normative interpretation and implementation comparison | Project decisions do not replace OGC text |

The phase-9 branch remains pinned at the studied commit. On August 31 the remote also exposed `main` at `6fdf396...` and `phase-10` at `4ca1d7d...`; those later lines are freshness context, not silently substituted evidence. The phase-10 line contains 124 commits not reachable from the phase-9 pin, so conclusions about the client’s current implementation require a separate review.

### 3.2 Evidence-State Vocabulary

| State | Rule |
|---|---|
| Reproduced | Replayed in this study with command and result recorded |
| Smoke-test output | Dated source report or preserved raw capture |
| Code-supported | Directly supported by pinned source or test behavior |
| Fixture-supported | Demonstrated only by local deterministic fixture/test |
| Historical | Live observation not freshly repeated here |
| Partial | Some sub-findings or environments confirmed; others not |
| Anecdotal | Statement lacks a preserved response, code anchor, or repeatable procedure |
| Stale | Later evidence shows the tested deployment or code changed |
| Superseded/retracted | Source explicitly replaces or withdraws the finding |
| Unresolved | Ownership or normative interpretation cannot be closed from available evidence |

### 3.3 Conflict and Ownership Rules

1. Standards and accepted Glaux standards reports control obligations.
2. Raw wire evidence controls what a server returned at a stated time.
3. Client code controls what the client accepted, rejected, generated, or lost.
4. Parsed shell objects, screenshots, and prose summaries cannot overrule raw bytes.
5. Later dated evidence supersedes earlier deployment-state claims but does not erase regression history.
6. A tolerant-client workaround does not make the triggering server behavior canonical.
7. One symptom can have multiple owners; record immediate cause, enabling condition, and documentation/test contributors separately.

---

## 4. Tested Servers, Client Versions, Branches, and Test Conditions

### 4.1 Historical Server Matrix

| ID | Server / deployment | Corpus role | Conditions and freshness |
|---|---|---|---|
| S1 | Public OpenSensorHub (OSH) | Primary long-running Part 1/2, SensorML, SWE, navigation, query, and CRUD target | Basic auth; plain HTTP; credentials intentionally absent; mutable historical demo |
| S2 | 52°North `connected-systems-pygeoapi` demo | Alternate negotiation/provider and degraded-server target | Unauthenticated HTTPS; certificate and availability changed across reports; mutable |
| S3 | OS4CSAPI-hosted OSH | Second OSH configuration with richer mission data | Basic auth; HTTPS; first contacted in Phase 7; mutable |
| S4 | Connected Systems Go (`cs-go`) | Independent implementation first contacted in ST26 | Unauthenticated data endpoints; self-signed TLS in source report; read-only checkpoint |
| P9 | Self-hosted 52°North pygeoapi PoC | Controlled Phase-9 research deployment with seeder and captures | Oracle ARM64 host; dated May 9 captures; intentionally research-grade; known reverse-proxy/base-URL issue |

SECD appears in the overall Glaux source landscape but was not a substantive target in the pinned OS4CSAPI phase-9 smoke corpus. Its implementation baseline is IDR-SRV-014D and its client/server adjudication belongs to IDR-SRV-014F.

### 4.2 Client Baselines

| Baseline | Version / commit | Conditions | Status |
|---|---|---|---|
| Historical ST26 | Client commit `4f3a7b7`; 1,793 passed / 4 skipped / 62 suites | April 29, 2026, four-server read-side checkpoint | Historical report |
| Issue #188 | Phase-7 snapshot; Node 24.15.0 / npm 11.12.1 | May 3, 2026, OSH live probe by second consumer | Integrated reproduction; issue now closed |
| Controlling study pin | `754411897...`; package `1.3.1-dev` | Phase-9 branch, May 11, 2026 | Source baseline |
| Local replay | Node 26.8.1 / npm 11.19.0 | August 31, 2026, clean `npm ci`, targeted integration and type-check | 86/86 passed; 0 type errors |

### 4.3 Test-Condition Limits

- Historical smoke reports mixed live HTTP probes, client URL-generation checks, parser calls, shell deserialization, and source inspection. Each finding must retain its method.
- ST26 was read-only at Checkpoint A; its CRUD conclusions were carried from earlier runs, not repeated there.
- Live servers changed data, certificates, DNS, conformance declarations, route health, and counts across runs.
- The local replay used deterministic fixtures and made no live-server conformance claim.
- `npm ci` reported 27 dependency vulnerabilities (2 low, 11 moderate, 13 high, 1 critical). No audit remediation was authorized or needed for this read-only research reproduction.

---

## 5. Client-Server Behavior Findings

### 5.1 Consolidated Findings Matrix

Every row supplies the plan’s fifteen fields. “Expected” cites the accepted Glaux baseline when a full normative clause is not repeated here.

| ID | Source / anchor | Evidence | Server | Client pin | Area | Observed | Expected / reference | Likely root cause | Server impact | Client impact | Glaux implication | Test / fixture | Handoff | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| E01 | [Issue #188](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/issues/188) | G/C; partial | OSH | phase-7 | Discovery | Six getters dereferenced null when no usable data link was found | Graceful absence handling; canonical entrypoint links [N] | Client null guard plus server link divergence | Noncanonical discovery impairs generic clients | Crash instead of capability absence | Emit canonical links; never depend on client tolerance | Landing page without data; assert typed absence, no crash | 009, 010, 050, 056 | Immediate client defect; trigger also server-side |
| E02 | Issue #188; Phase-9 §1 | G/S | OSH | phase-7/9 | Link relation | Landing page used `rel=collections`; client allowlist expected `data` variants | Standards-derived canonical relation [N] | Server divergence; narrow client recognition | Discovery may be missed | Empty collection discovery | Canonical relations and link-contract tests | Alternate-rel negative fixture | 009, 010, 017, 056 | Tolerant recognition is optional client policy |
| E03 | Phase-9 §1 | C/G | OSH | phase-7/9 | Collection recognition | CSAPI signaling in `featureType` was ignored by parser | Recognize relevant published signals without discarding identity [N] | Client parser/model gap | None if server is canonical; ambiguity amplified | False-negative classification | Publish canonical collection metadata consistently | Collection vocabulary matrix | 010, 015, 023, 056 | Client-side at studied pin |
| E04 | Phase-9 §1 | C/G | OSH | phase-7/9 | Link following | Client followed a self link and accepted a different returned collection ID | Returned representation must match requested identity [N] | Client trust-boundary gap; possibly bad server link | Mislinked resource can escape detection | Silent identity substitution | Stable canonical IDs and correct hrefs; clients should verify | Mismatched-self response fixture | 010, 017, 023, 050 | Security/correctness boundary |
| E05 | Known quirks §OSH | H/S | OSH variants | historical | Negotiation | Accept ignored; `?f=` selected JSON/GeoJSON/SensorML; odd `Content-Type` values appeared | Honest HTTP negotiation and representation metadata [N] | Server negotiation implementation | Generic HTTP clients receive unexpected format | Client must use server-specific adapter | Support and document canonical negotiation; reject unsupported formats | Accept/f cross-product golden tests | 011, 013, 023, 050, 056 | Historical deployment behavior |
| E06 | Known quirks; ST26 | H/S | 52°North | historical | Negotiation | Accept routed to different providers; `application/json` could return an empty collection while other media types had data | Negotiation changes representation, not logical dataset, absent documented semantics [N/X] | Dual-backend deployment architecture | False absence by representation | Client sees inconsistent inventories | One logical query contract across formats | Same-query multi-format equivalence fixture | 011, 013, 050, 053 | Later S2 state degraded further |
| E07 | ST26 §9 | S/H | S1–S4 | `4f3a7b7` | Envelopes | Part 1 used `items` or FeatureCollection; Part 2 commonly used `items`; cs-go was hybrid | Use the representation required for selected media type [N] | Legitimate representation diversity plus implementation choices | Inconsistent media/shape pairing harms clients | Parser needs explicit media-specific branches | Define representation/media mapping and schema per route | Envelope family golden corpus | 015, 023, 050, 053, 056 | Diversity itself is not a defect |
| E08 | ST26 §9 | S/H | S1/S3/S4 | `4f3a7b7` | Pagination | Defaults ranged from 100 to 10; counts absent on OSH and present on cs-go | Documented, bounded pagination with correct links/count semantics [N] | Implementation-specific defaults/features | Hidden truncation risk | Clients assuming page size miss data | Declare defaults/maxima and generate correct next links | 0/1/default/max/overflow/cursor fixtures | 012, 033, 050, 054, 056 | Do not infer completion from first page |
| E09 | ST26 §§4,9 | S/H | S1–S4 | `4f3a7b7` | Query | Standard-looking parameters returned 200, but acceptance alone did not prove filtering | Supported query must change/validate semantics; unsupported input must be clear [N] | Smoke oracle too weak; possible server ignore | Silent false-positive behavior | URL generation appears successful | Assert result semantics, not only status | Seeded discriminating query fixtures | 012, 023, 050, 051, 056 | Cross-checks accepted 014D lesson |
| E10 | Known quirks §OSH | H/S | OSH | historical | Paths | Lowercase `/controlstreams` worked; mixed-case path failed | Published canonical path and case-sensitive routing [N] | Server/client naming inconsistency risk | Generated mixed-case paths fail | Builder casing becomes critical | One canonical route spelling in code/OAS/tests | Case-variant negative tests | 010, 024, 050, 056 | Client issue #20 concerned fallback casing |
| E11 | ST26 §9 | S/H | OSH vs cs-go | `4f3a7b7` | Commands route | OSH rejected top-level commands; cs-go returned 200; nested routes differed | Implement declared conformance route set consistently [N] | Implementation coverage/configuration difference | Capability cannot be inferred across servers | Client requires discovery/fallback | Glaux should expose its normative route set and truthful capabilities | Top-level/nested route topology tests | 009, 010, 036, 050, 056 | Avoid speculative fallback on arbitrary errors |
| E12 | Known quirks §OSH | H/S | OSH | historical | CRUD response | Create returned 201, empty body, and Location; update required uid | HTTP and resource-specific requirements must be documented [N] | Server behavior plus client empty-body assumption | Valid empty success body can crash clients | Parser expected JSON in finding #18 | Emit correct Location and documented body; client tests both | 201 empty/body/Location matrix | 019, 023, 024, 050, 056 | Exact uid obligation needs normative trace |
| E13 | Pygeo §3.3 | S/H | P9 pygeoapi | phase-9 | Link generation | `/collections` hrefs pointed to localhost due `server.url` configuration | Public canonical URLs must be externally resolvable [N] | Deployment configuration | Hypermedia unusable outside host | Following links fails | Derive/validate public origin behind proxies | Proxy/public-base integration fixture | 010, 017, 043, 049, 056 | Config defect, not core architecture proof |
| E14 | Pygeo §§5.7,7 | S/H | P9 pygeoapi | phase-9 | API metadata | Working endpoints absent from OpenAPI; conformance advertised only Common core | Declarations must reflect actual supported surface [N] | Metadata generator/provider integration gap | Clients under-discover; conformance is unverifiable | Code generation omits operations | One capability source must drive routes/OAS/conformance/tests | Runtime-to-OAS parity test | 008, 009, 024, 050, 051 | Under-reporting, not evidence of conformance |
| E15 | ST26; pygeo §6.4 | S/H | S1/P9 | historical | Error/round trip | Sampling Features list/detail returned 500 or empty after successful writes | Successful creation must be retrievable under declared semantics [N] | Server persistence/index/route defect | Data becomes inaccessible | Client sees false success then loss | Transactional round-trip acceptance tests | Create-read-list-update-delete fixture | 018, 019, 023, 050, 053 | P9 capture is strongest controlled case |
| E16 | Pygeo §§5.2,6.5 | S/H | P9 pygeoapi | phase-9 | Relationships | `deployedSystems` crashed validation; deployment/system relation could not be linked or read | Relationship payload and navigation must be consistent [N] | Server validator/model mismatch | Declared relationship unusable | No portable workaround preserving semantics | Validate relationship graph and reciprocal navigation | Deployment-system relation fixture | 015, 017, 018, 023, 050 | No auto-link and explicit form broken |
| E17 | Pygeo §§4.2,5.4 | S/H | P9 pygeoapi | phase-9 | Write media | Part 1 writes required SensorML media; dynamic writes required JSON; wrong media produced misleading field error | Media mismatch should yield accurate status/problem details [N] | Endpoint parsers and error mapping | Operators misdiagnose valid payloads | Client cannot distinguish syntax from media error | Explicit consumes contract; 415/structured diagnostics where applicable | Wrong-media negative matrix | 013, 023, 024, 050 | Exact status remains downstream normative mapping |
| E18 | ST24 P7-F3 | S; retracted | OSH | Phase 7 | Harness | PowerShell unwrapped one-element array, creating false server finding | Raw wire bytes are primary evidence [X] | Test-tool deserialization artifact | None | Unneeded defensive change and false report | Preserve raw response before parsing | Single-element raw/parsed differential fixture | 050, 051, 053, 056 | Explicitly superseded/retracted |
| E19 | Demo #101; integration | C/R | fixtures | phase-9 pin | SWE model | Parser rejected complex SWE types beyond its supported subset | Server may validly emit standards-permitted complex schemas [N] | Client model incompleteness | Canonical server output may break narrow clients | Loss/rejection of Vector/DataArray/etc. | Implement standards-correct schemas; publish coverage; test clients honestly | SWE component combinatorial corpus | 015, 023, 034, 053, 056 | Client gap, not reason to restrict server model |
| E20 | Demo #103/#108/#109 | C | fixtures/cs-go | phase-9 pin | References | Part 2 cross-references and Part 1 `@link` fields were omitted or discarded by types/parsers | Preserve normative identity and relationship fields [N] | Client type/parser gap | Rich server relationships may be invisible | Silent information loss | Emit canonical references and include preservation tests | `@id`/`@link`/links round-trip fixtures | 015, 017, 023, 053, 056 | ST26 cs-go supplied real shape examples |
| E21 | Demo #105/#106/#107 | C | none required | phase-9 pin | Query typing | Client query names/options differed from accepted CSAPI naming or used weak base types | Exact normative query vocabulary [N] | Client mapping/type gap | None if Glaux is correct | Client generates wrong or inaccessible filters | Do not implement aliases as primary contract | Compile-time and wire query matrix | 012, 023, 056 | Compatibility aliases require later policy |
| E22 | Demo #20; ST26 | C/S | OSH | phase-9 pin | URL builder | ControlStream fallback path casing could generate non-working URLs | Exact route-generation contract [N] | Client URL bug | None | Operations fail despite valid server | Publish examples and exercise actual client | URL golden tests with encoded IDs/subpaths | 010, 024, 056 | Client ownership |
| E23 | Demo #18 | C/H | OSH-like | phase-9 pin | Success parsing | Valid 201 empty-body response caused JSON parse failure | Success handling must respect status/headers/body contract [N/X] | Client transport/parser coupling | None if response conforms | False failure after successful mutation | Stable response policy; test empty and represented forms | Empty-body transport fixture | 019, 024, 050, 056 | Cross-layer boundary |
| E24 | ST24 P7-F4 | S/H | S3 | Phase 7 | Async command | Command POST returned 202 with no immediate result; client docs originally assumed 201 | Async acceptance and lifecycle must be explicit [N] | Valid/implementation-specific lifecycle plus docs gap | Ambiguous tracking if no monitor/location | Client may misclassify success | Define sync/async contract, status transition, and tracking link | 201/202/status/result transition scenarios | 036, 037, 038, 050, 053 | Client docs later updated |
| E25 | ST24 P5-F3/F4 | S/H | S1/S3 | Phase 7 | Status fields | Live/async fields absent and only COMPLETED observed | Full supported state model needs discriminating fixtures [N/X] | Thin live data and/or partial server behavior | Unexercised states conceal defects | Parser branches remain unvalidated | Provide deterministic lifecycle simulator | All command/status/event states | 034, 035, 036, 037, 053 | Evidence insufficient to call unsupported |
| E26 | Phase-9 captures | S | P9 pygeoapi | phase-9 pin | Landing response | JSON landing page emitted literal `Content-Type: None` | Accurate HTTP media metadata [N] | Server/provider serialization defect | Strict clients reject/sniff | Transport failure before domain parsing | Header/body contract validation | Invalid/missing media header negatives | 013, 023, 050, 056 | Direct captured finding |

### 5.2 Finding Concentration

The recurring high-impact groups are:

1. **Discovery truth:** root links, collections metadata, conformance, OpenAPI, and reachable routes drift independently.
2. **Hypermedia integrity:** relation vocabulary, public hrefs, response identity, and relationship navigation require end-to-end checks.
3. **Representation truth:** media type, envelope, schema, and actual logical dataset must agree.
4. **Semantic query truth:** HTTP 200 is insufficient; filters, sorting, pagination, and counts need discriminating data.
5. **Lifecycle truth:** a successful write must be observable, and asynchronous tasking needs explicit tracking and state transitions.
6. **Evidence truth:** raw transport evidence must be preserved before tools normalize it.

---

## 6. Schema, Model, and Representation Findings

### 6.1 Shape Diversity That Glaux Must Test

| Dimension | Observed variants | Glaux treatment |
|---|---|---|
| Collection envelope | GeoJSON FeatureCollection; `{items, links}`; resource-specific envelopes | Bind each advertised media type to one documented schema; reject accidental hybrids |
| Resource type vocabulary | Full SOSA/SSN/SensorML URIs; CURIEs; null; path fallback in client | Emit accepted canonical values; validation and tests should recognize only intentionally supported aliases |
| References | Flat `system@id`; `system@link: {href}`; `datastream@id` string; ordinary links | Select canonical model in downstream topics; preserve identity and relation semantics losslessly |
| Time bounds | ISO instants; `now`; `..`; null; absent | Define resource-specific temporal grammar and open-bound normalization with round-trip tests |
| SWE schema | DataRecord with Quantity/Text/Boolean/Count/Vector; client gap for complex types | Support the standards-selected subset explicitly; do not narrow server output to one client parser |
| Unit | `{href}` and `{code}` | Validate permitted forms and preserve authored semantics |
| Counts | absent or present | Define required/optional behavior by conformance and pagination profile |
| Link href | relative; absolute; internal localhost; external public URL | Generate correct public canonical hrefs and validate proxy deployment |

### 6.2 Model Ownership

The demo findings are predominantly client-model evidence. Missing TypeScript fields, typed query options, navigation helpers, parser branches, and `@link` preservation do not establish server gaps. They show which otherwise-valid server outputs can expose narrow clients. Glaux should remain standards-correct and use these shapes as external-client probes.

Conversely, a server returning a media type inconsistent with its body, accepting a write that cannot be read, generating localhost links, or advertising metadata inconsistent with routes is not excused by a defensive client. Those are server/deployment contract failures.

### 6.3 Schema Strategy Implications

- Validate transport headers, envelope schema, resource schema, semantic references, and cross-resource invariants as separate layers.
- Preserve unknown extension fields through any internal normalization boundary where the standard permits extensions.
- Generate canonical examples from the same schemas used for validation and OpenAPI where feasible.
- Keep invalid historical examples as negative fixtures with provenance; do not “clean” them into positive goldens.
- Treat representation equivalence as a semantic test: JSON, GeoJSON, and SensorML views of the same resource must retain identity and common facts.

---

## 7. Dynamic-Data, Status, Event, and Tasking Findings

### 7.1 Observations and DataStreams

The client corpus successfully exercised nested System → DataStream → Observation URL building, collection parsing, temporal parameters, pagination, schema retrieval, and observation creation in fixtures [C,R]. Historical live data covered simple and vector SWE result schemas [S]. Cross-server weakness remained substantial: some deployments returned 500 for dynamic collections, some lacked counts, and query success was not always semantically verified.

Glaux needs fixtures for empty, one-item, multi-page, high-volume, historical-window, latest-value, malformed-result, and cross-reference cases. A response status alone cannot verify datetime or latest semantics. Expected IDs and time ranges must be known before the query.

### 7.2 Commands and ControlStreams

The corpus captured both nested-only and top-level command access, lowercase path sensitivity, 201 and 202 creation patterns, immediate and asynchronous fixture responses, status lookup, result retrieval, cancellation URL construction, and route fallback [C,S,R]. Server fallback must be capability-aware: a 400 caused by invalid payload or authorization must not be interpreted as evidence to try another route.

The pinned integration fixtures pass workflows for control-stream discovery, feasibility URL construction, single and bulk submission, sync and async responses, status, result, cancellation, fallback, caching, and errors [R]. Those fixtures demonstrate desired client coverage, not that all tested servers implement every operation.

### 7.3 Status and Event Coverage

Historical servers provided too little state diversity to validate a complete lifecycle. Observing only `COMPLETED`, or absence of live/async fields, cannot distinguish unsupported behavior from thin fixtures. Glaux should provide a deterministic lifecycle driver that can pause at each legal state, produce failures and cancellation races, and assert persistence/audit effects.

System Events were not materially exercised across the pinned smoke matrix. Their absence is an evidence gap, not a conclusion about the standard or Glaux obligation. IDR-SRV-035 owns the event architecture; this topic hands it the need for state-rich, time-controlled fixtures.

### 7.4 Security Boundary

The pygeoapi research deployment accepted unauthenticated writes, but it was explicitly research-grade [S]. This is not a client-compatibility recommendation. Commands, feasibility, status mutation, and cancellation require separate authorization and audit tests under IDR-SRV-037, IDR-SRV-038, IDR-SRV-040 through 043, and IDR-SRV-055.

---

## 8. Cross-Implementation Interoperability Findings

### 8.1 Recurring Versus Server-Specific

| Finding family | Recurring | Server-specific examples |
|---|---|---|
| Metadata/runtime drift | 52°North and P9 under-reported; other deployments differed in claims | P9 OpenAPI omitted working routes |
| Representation diversity | All server families varied | 52°North dual provider; cs-go hybrid envelopes |
| Route topology differences | OSH and cs-go differed | Top-level commands worked on cs-go, failed on OSH |
| Link integrity | Multiple forms and sparse links | P9 localhost hrefs; OSH discovery relation |
| Pagination | Defaults and count fields differed | cs-go default 10; OSH commonly 100 |
| Temporal vocabulary | `now`, `..`, null, absent | S3 exposed `..` client gap |
| Partial endpoint health | 52°North/P9/S1 sampling feature issues | P9 accepted writes but could not retrieve them |
| Task lifecycle | Sync/async and sparse states | S3 returned 202; live corpus lacked diverse states |

### 8.2 Broader Lessons

- A second implementation is not enough. Two OSH deployments shared many behaviors; the independent Go and pygeoapi families exposed different assumptions.
- A second consumer is as important as a second server. Issue #188 appeared when a consumer outside the original explorer used the library.
- Server identity must include implementation, deployment, version/commit if known, configuration, dataset, and date. “OSH” or “pygeoapi” alone is insufficient.
- Interoperability matrices must separate mandatory canonical behavior from optional client-tolerance profiles.
- A live demo is a canary, not a stable oracle. Captured responses and self-hosted seeded deployments are required for regression.

### 8.3 Boundary With 014F and 014G

IDR-SRV-014F should compare SECD evidence against the failure taxonomy and fixture pattern defined here, but it must independently adjudicate its source corpus. IDR-SRV-014G should review community discussions for rationale or pain points and must not retroactively convert discussion opinion into proof for these findings.

---

## 9. Client-Library Limitation Versus Server-Behavior Classification

### 9.1 Ownership Decision Table

| Class | Examples | Glaux response |
|---|---|---|
| Server/deployment | Wrong public href, false media header, write/read asymmetry, ignored query, broken route | Avoid; create server acceptance and deployment tests |
| Client | Null dereference, dropped `@link`, incomplete SWE parser, wrong query name/casing | Use as external-client compatibility evidence; do not deform canonical server behavior |
| Shared boundary | Noncanonical discovery plus narrow allowlist; empty 201 plus unconditional JSON parsing | Specify server contract precisely and test both sides independently |
| Fixture/data | Only COMPLETED commands; empty collections; insufficient relationship graph | Build state-rich discriminating fixtures |
| Harness/tool | PowerShell array unwrapping; parsed rather than raw capture | Preserve raw status/headers/body and record tool versions |
| Documentation | Wrong field names, undocumented async status, incomplete OAS | Generate/verify docs against runtime contract |
| Standards ambiguity | Only where accepted baseline cannot close interpretation | Record question and route to exact downstream topic; do not guess |

### 9.2 Adjudication Pattern

For each future failure, ask in order:

1. What exact raw request and response occurred?
2. Does the accepted normative baseline require or prohibit it?
3. Did server metadata promise the behavior?
4. Did the client generate the intended request and preserve the response?
5. Was the fixture capable of distinguishing correct from ignored behavior?
6. Did the harness transform values?
7. Is the result current, historical, superseded, or unresolved?

This ordering prevents the common error of assigning the first visible exception to the server.

---

## 10. Standards Ambiguity and Documentation-Gap Findings

### 10.1 Closed by the Accepted Baseline

The corpus sometimes used “conformant” loosely or treated popular behavior as a practical alternative. The accepted Glaux reports already control canonical conformance URIs, landing and declaration behavior, resource paths, navigation, queries, negotiation, errors, schemas, and documentation. Popular `rel=collections`, an accepted-but-ignored query, an implementation-era field name, or a defensive parser branch is not a Glaux requirement [N].

### 10.2 Questions Requiring Downstream Precision

| Question | Why not decided here | Owner |
|---|---|---|
| Whether to expose compatibility aliases in addition to canonical queries/paths | Product compatibility and deprecation policy | 010A, 012, 056 |
| Exact canonical internal representation for `@id`, `@link`, and ordinary links | Resource/link model design | 015, 017 |
| Exact supported SWE component/profile breadth | Schema and domain model scope | 015, 023, 034 |
| Exact 201/202 body, Location, monitor, and result policy | Mutation and command lifecycle design | 019, 024, 036 |
| Error status/problem-detail mapping | API/error contract | 024 |
| Optional representation equivalence rules | Representation and negotiation design | 011, 013 |

### 10.3 Documentation Gaps

The corpus shows recurring divergence among runtime routes, root links, `/collections`, `/conformance`, OpenAPI, JSDoc, examples, and actual response fields. Glaux should make documentation parity a testable invariant, not an editorial cleanup task. At minimum, every public operation needs matching route, method, media types, schema, authorization, query names, success responses, error responses, conformance mapping, example, and executable test.

---

## 11. Test Strategy and Fixture Implications

### 11.1 Test Layers

| Layer | Purpose | Required evidence |
|---|---|---|
| Unit/property | Validate parsers, serializers, time bounds, identifiers, link generation, query encoding | Generated cases and shrinkable failures |
| Schema golden | Lock exact headers and raw representation bytes for valid/invalid examples | Status, headers, body, schema version, provenance |
| Contract integration | Exercise route → validation → persistence → retrieval → links | Seed manifest and expected graph |
| Conformance | Trace each adopted requirement to positive/negative tests | Requirement ID and conformance class |
| Runtime/documentation parity | Compare routes, OpenAPI, conformance, examples, media, and auth | Generated inventory diff |
| External client | Run OS4CSAPI client, Explorer where automatable, and at least one independent client | Client version/commit and raw transcript |
| Live canary | Detect ecosystem drift without controlling CI truth | Dated non-blocking report and capture |

### 11.2 Minimum Fixture Families

1. Landing pages with canonical links, missing links, duplicated links, wrong relations, relative hrefs, and proxy-origin cases.
2. Collections with full URIs, CURIEs, null/unknown types, wrong self IDs, and partial Part 1/Part 2 capability.
3. GeoJSON and items envelopes with empty, one, many, malformed, missing count, and paginated forms.
4. Query datasets where every filter produces a distinct expected subset, including invalid input and unsupported parameters.
5. SensorML/SWE schemas spanning scalar and complex supported components, unit forms, optional fields, and invalid combinations.
6. Relationship graphs for parent/child, deployment/system, System/DataStream, DataStream/Observation, ControlStream/Command, and broken references.
7. Write cycles with 201 represented, 201 empty plus Location, 202 plus monitor, duplicate ID, invalid media, failed validation, and rollback.
8. Command lifecycles covering every legal state, failure, cancellation, result, race, authorization denial, and audit record.
9. Representation-equivalence sets that preserve identity and shared semantics across formats.
10. Raw-tool differential cases proving that test harness deserialization cannot alter oracle values.

### 11.3 CI Versus Live Smoke

Deterministic self-hosted fixtures should gate CI. Public servers should be scheduled canaries because DNS, certificates, auth, data, configuration, and software change independently. A live failure should produce a capture and triage record; it should not automatically redefine expected Glaux behavior.

### 11.4 Finding Record Schema

Future findings should retain all fifteen fields from §5 plus: request method, raw request headers/body hash, response status/headers/body hash, capture time, environment, reproduction command, first/last known commit, normative disposition, freshness state, and adjudicator. This converts the smoke corpus from prose history into a queryable regression inventory.

---

## 12. Lessons for Glaux Server

### 12.1 Adopt

- Multi-server and multi-client black-box testing from the beginning.
- Immutable pins, raw captures, seeded scenarios, and explicit finding states.
- Typed capability discovery and graceful handling of unsupported optional surfaces.
- Media-specific schemas and semantic cross-representation tests.
- Deterministic lifecycle and relationship fixtures.
- Runtime/OpenAPI/conformance/example parity gates.

### 12.2 Avoid

- Encoding a tested server’s quirks as the primary API.
- Assuming HTTP 200 means a query was applied.
- Assuming a client exception proves a server defect.
- Trusting self links without identity checks or generating internal proxy URLs.
- Allowing successful writes that are not retrievable.
- Using one-element or all-COMPLETED fixtures as coverage of arrays or state machines.
- Treating a mutable public demo as a release-pinned conformance oracle.

### 12.3 Validate Explicitly

- Canonical routes, link relations, resource identity, query semantics, negotiation, media headers, envelopes, pagination, counts, errors, and public origin.
- Complete round trips and reciprocal relationship navigation.
- Sync and async command tracking, terminal states, results, cancellation, authorization, and audit.
- Compatibility with exact pinned external clients without weakening normative behavior.

### 12.4 Investigate Later

- Compatibility aliases and their deprecation/versioning policy.
- The supported SensorML/SWE profile breadth.
- Which external-client versions become release gates versus informational canaries.
- Performance characteristics of high-volume observations and streaming; the smoke corpus is functionally rich but not a load benchmark.

---

## 13. Downstream Topic Handoff Matrix

| Finding family | Primary topics | Handoff, not decision |
|---|---|---|
| Canonical discovery and declaration parity | 008, 009, 010, 024 | Derive one runtime capability inventory and parity tests |
| Versioning and compatibility aliases | 010A, 012, 056 | Decide whether and how noncanonical inputs are supported |
| Resource/reference shapes | 015, 017 | Select lossless canonical identity and relationship model |
| Representation and media negotiation | 011, 013, 023 | Define media/schema mapping and equivalence invariants |
| Query semantics and pagination | 012, 033 | Define defaults, maxima, filtering, sorting, cursors, and counts |
| Mutation and round trip | 018, 019, 024 | Define validation, transactional behavior, success metadata, and errors |
| DataStreams/Observations/status | 034 | Use discriminating temporal/result fixtures and high-volume pagination cases |
| System Events | 035 | Supply deterministic state/event corpus; pinned smoke evidence is thin |
| Commands/feasibility/security | 036, 037, 038, 040–043 | Model full async lifecycle, authorization, race, and audit behavior |
| Deployment/proxy correctness | 043, 049 | Test external origins, forwarded headers, TLS, and link generation |
| Conformance and traceability | 050, 051 | Trace every canonical and negative behavior to requirements |
| TDD/fixture corpus | 052, 053 | Convert consolidated findings to raw-wire golden scenarios |
| Performance/streaming | 054 | Do not infer performance from functional smoke tests; reuse realistic volumes |
| Security tests | 055 | Add unauthenticated write, command authorization, SSRF/link, and error-leak tests |
| External clients | 056 | Pin clients and separate canonical pass gates from tolerance probes |
| SECD evidence | 014F | Apply the taxonomy while independently adjudicating SECD captures |
| Community rationale | 014G | Compare discussion lessons without using opinion as behavior evidence |

---

## 14. Recommendations

1. Adopt **strict server, tolerant client** as the interoperability boundary and record both sides separately.
2. Create one typed Glaux capability contract that drives routes, root links, collections, conformance, OpenAPI, examples, authorization metadata, and parity tests.
3. Preserve raw request/response evidence before any shell, SDK, or model transformation.
4. Turn E01–E26 into requirement-linked test candidates only in their owning downstream topics; retain source pins and ownership.
5. Gate every create/update relationship with read-after-write, list visibility, item identity, and reciprocal-navigation checks.
6. Test query semantics using discriminating seeded data and negative assertions; never use status-only success criteria.
7. Bind every representation to honest media headers and test semantic equivalence across alternate formats.
8. Build deterministic command, status, and event lifecycle fixtures before claiming dynamic/tasking interoperability.
9. Run at least two implementation families and two independent clients in release-oriented interoperability checks.
10. Keep public live servers as scheduled canaries and self-hosted pinned deployments as reproducible regression targets.
11. Record implementation, deployment, configuration, dataset, client commit, environment, and date in every finding.
12. Revisit the later `phase-10` client line only when a downstream topic needs current-client behavior; do not rewrite this phase-9 historical baseline.

---

## 15. Risks, Constraints, and Open Questions

### 15.1 Constraints

- Most live observations are historical and were not safely reproducible without credentials, certificate bypass, mutable state, or external side effects.
- The client repository is a fork and the studied package version is developmental.
- Some reports summarize outputs without preserving every raw response.
- Server version/build identifiers were often absent, making deployment-level language mandatory.
- Phase-9 experiment hypotheses and recommendations are not all completed observations.

### 15.2 Risks and Controls

| Risk | Control |
|---|---|
| Client assumptions become server requirements | Normative baseline controls; ownership column mandatory |
| Stale live behavior is presented as current | Historical labels and exact dates/pins |
| Defensive compatibility normalizes nonconformance | Separate canonical gate from tolerance profile |
| Parsed tools create false findings | Raw transport capture is primary oracle |
| Thin fixtures overstate feature coverage | Cross-product and lifecycle fixture design |
| Public-server drift destabilizes CI | Self-hosted pinned gates; live canaries non-blocking |
| Later client branch silently changes conclusions | Preserve phase-9 baseline; review phase-10 separately |

### 15.3 Resolved Open Questions From the Plan

- **Representative baseline:** phase-9 commit `754411897...`, because the topic and exemplar links target that branch and it contains the cumulative reports plus phase-9 synthesis/captures. Later lines are disclosed separately.
- **Current reproduction:** pinned deterministic integration workflows and type-check were reproduced. Historical live-server findings were not reclassified as current.
- **Ambiguity versus defect:** accepted standards reports decide normative questions; unresolved design choices are routed in §10.2.
- **CI candidates:** deterministic contract, golden, round-trip, semantic-query, lifecycle, parity, and pinned-client tests in §11.

No open question blocks this report. Downstream topics still own the policy and design decisions explicitly handed to them.

---

## 16. Validation Against the Research Plan

### 16.1 Success Criteria

| Criterion | Result | Evidence |
|---|---|---|
| Exact sources, pins, dates, servers, conditions | Pass | §§3–4, Appendix B |
| Ownership classes separated | Pass | §§5, 9 |
| Grouped by behavior area and implementation | Pass | §§5–8 |
| Reproduced/stale/inferred/unresolved distinguished | Pass | Reading Guide, §§3–5 |
| Recurring and server-specific findings identified | Pass | §8 |
| Model, dynamic, docs, validation, conformance, interop implications | Pass | §§6–12 |
| Test and fixture recommendations explicit | Pass | §11 |
| Downstream handoffs explicit | Pass | §13 |
| References explicit and reproducible | Pass | §17, Appendix B |

### 16.2 Deliverable Structure

All seventeen required sections are present in order. The consolidated matrix contains 26 findings and all fifteen required fields. Five core and 29 detailed questions are closed. All six methodology phases were executed at research-report depth.

### 16.3 Quality Checks

- Pinned commit and remote branch heads verified August 31, 2026.
- Upstream-history register Version 1.8 consulted for relevant official artifact and interpretation entries; no material register change was identified.
- Source counts independently enumerated: 28 smoke reports, 31 demo finding records, 21 Phase-9 capture files.
- `npm ci` completed without modifying tracked evidence.
- Targeted integration replay: 5/5 suites and 86/86 tests passed.
- TypeScript check: `tsc --noEmit` passed with zero errors.
- No authenticated, mutating, destructive, load, or long-lived live-server probe was performed.
- IDR-SRV-014F and IDR-SRV-014G were not executed.

---

## 17. References

### 17.1 Controlling Standards and Glaux Sources

- [OGC API - Connected Systems Part 1, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems Part 2, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC API - Connected Systems v1.0.0 OpenAPI artifacts](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api)
- [OGC SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [Glaux Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-014E Research Plan](../IDR%20Plans/idr-srv-014e-os4csapi-client-smoke-test-findings-study.md)
- Accepted reports IDR-SRV-006 through IDR-SRV-014D in this report directory.

### 17.2 Pinned OS4CSAPI Sources

- [Phase-9 repository tree at `754411897...`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230)
- [Phase-9 discovery-layer lesson propagation](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/phase-9/01-discovery-layer-lesson-propagation.md)
- [Phase-9 live-testing experiment plan](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/phase-9/02-live-testing-experiment-plan.md)
- [Phase-9 pygeoapi deployment findings](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/phase-9/03-52north-pygeoapi-deployment-findings.md)
- [Known server quirks](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/governance/known-server-quirks.md)
- [Smoke-test prompt template](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/governance/smoke-test-prompt-template.md)
- [Post-Phase-7 smoke report](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/implementation/live-server-smoke-test-post-phase-7.md)
- [Post-Phase-8A smoke report, ST26](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/implementation/live-server-smoke-test-post-phase-8-A.md)
- [Cross-server interoperability analysis](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/implementation/cross-server-interoperability-analysis.md)
- [Demo-app finding corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/testing/demo-app-findings)
- [CSAPI integration test corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/src/ogc-api/csapi/integration)
- [Issue #188](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/issues/188)

---

## Appendix A. Detailed Question Ledger

| # | Detailed question disposition | Answer anchor |
|---:|---|---|
| 1 | Repositories, commits, issues, reports, tests, captures, and findings identified | §3 |
| 2 | Branch, commit, package version, and local toolchain recorded | §§3–4 |
| 3 | OSH, OS4CSAPI-OSH, 52°North, cs-go, and controlled pygeoapi distinguished; SECD bounded | §4 |
| 4 | Reproduced, partial, historical, stale, unresolved, and superseded states defined/applied | Reading Guide, §3 |
| 5 | Exact evidence supplied through pinned sources and matrix anchors | §§5, 17 |
| 6 | Entrypoint, conformance, collection, discovery, and link issues analyzed | §§5.1, 8 |
| 7 | Identifiers, canonical URLs, related resources, and relations analyzed | §§5–6 |
| 8 | Queries, sorting, pagination, temporal filters, and counts analyzed | E08–E09, §11 |
| 9 | Negotiation, media, encoding, schemas, and alternatives analyzed | E05–E07, E17, E26, §6 |
| 10 | Errors, statuses, unsupported operations, and ambiguous failures analyzed | E11–E12, E15–E17, E23–E24 |
| 11 | Problematic shapes, fields, links, media, and examples cataloged | §§5–6 |
| 12 | Inconsistent resource shapes separated by representation and implementation | §6.1 |
| 13 | SensorML, SWE, GeoJSON, observations, DataStreams, commands, and status covered | §§6–7 |
| 14 | Server gaps and client-model gaps separated | §9 |
| 15 | Canonical model/schema-validation handoffs identified | §§6.3, 13 |
| 16 | Dynamic-data/status/event/tasking findings consolidated | §7 |
| 17 | Working and differing dynamic patterns bounded by evidence | §§7–8 |
| 18 | Unsupported/partial/ambiguous tasking behavior identified | §§7.2–7.3 |
| 19 | Dynamic, streaming, lifecycle, feasibility, and security handoffs identified | §13 |
| 20 | Cross-server recurring findings identified | §8.1 |
| 21 | Server-specific findings identified | §§4, 8.1 |
| 22 | Standards ambiguity, artifact, and documentation questions separated | §10 |
| 23 | External-client expectations translated to testable Glaux boundaries | §§11–12 |
| 24 | IDR-SRV-014F and 014G comparison boundaries explicit | §§8.3, 13 |
| 25 | Most useful tests identified: raw, semantic, round-trip, lifecycle, parity, multi-client | §11.1 |
| 26 | Brittle/misleading tests identified: status-only, parsed-tool, thin-state, mutable-live | §§5, 11.3 |
| 27 | Positive, negative, schema, conformance, golden, external-client, regression tests defined | §§11.1–11.2 |
| 28 | Reproduction fixture families defined | §11.2 |
| 29 | Harness, traceability, fixtures, and interoperability handoffs identified | §13 |

---

## Appendix B. Reproducible Study Record

### B.1 Repository Identity

```text
repository: https://github.com/OS4CSAPI/ogc-client-CSAPI_2.git
branch: phase-9
commit: 754411897173c2ec4debaa9bcf4ed9e0f8a9e230
commit date: 2026-05-11T08:24:21-04:00
subject: chore: gitignore .tmp/ operational scripts; add deployment-system-navigation spec-validation research doc
package: @camptocamp/ogc-client 1.3.1-dev
study date: 2026-08-31
```

### B.2 Local Replay

```text
Node: v26.8.1
npm: 11.19.0
install: npm ci — success
type check: npm run typecheck — success, 0 errors
integration: npx jest src/ogc-api/csapi/integration --runInBand
result: 5 suites passed; 86 tests passed; 0 snapshots; 0 failures
```

These commands validate the pinned source against its deterministic fixtures. They do not contact or certify any live server.

### B.3 Inventory Reproduction

```text
docs/implementation/live-server-smoke-test*.md: 28 files
docs/testing/demo-app-findings/*.md: 31 substantive files
docs/research/phase-9/captures/oracle-pygeoapi/: 21 files
src/ogc-api/csapi/integration/: 5 executed suites, 86 tests
```

### B.4 Freshness Record

Remote heads observed August 31, 2026:

```text
phase-9:  754411897173c2ec4debaa9bcf4ed9e0f8a9e230
main:     6fdf39669e8b11fe1e9c0c8914278c5f11a3b57f
phase-10: 4ca1d7db9b1ccdf545adbef42ef941139fe11977
```

The later heads prove continuing work; they do not invalidate the pinned historical baseline.

---

## Appendix C. Completion and Handoff

IDR-SRV-014E is complete at research-report depth and is **In Review**. It establishes a 26-finding, ownership-classified smoke-test baseline; defines reproducible evidence and freshness rules; converts practical failures into bounded Glaux implications; and supplies explicit model, validation, conformance, fixture, lifecycle, security, and external-client handoffs.

No implementation ticket was created, no live deployment was mutated, and IDR-SRV-014F was not started.

The next two actions are:

1. Glaux Project Lead acceptance of IDR-SRV-014E.
2. Authorization to execute exactly IDR-SRV-014F, the SECD Interoperability Findings Study.

Per the standing workflow, the combined response may simply be:

> proceed

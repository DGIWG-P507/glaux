# Section 014F: SECD Interoperability Findings Study - Research Report

**Topic ID:** IDR-SRV-014F<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-014F SECD Interoperability Findings Study](../IDR%20Plans/idr-srv-014f-secd-interoperability-findings-study.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 27 detailed questions; all six methodology phases, nine success criteria, seventeen required content areas, and thirteen minimum findings-matrix fields are validated<br>
**Methodology Used:** Pinned evidence-repository inventory; adjudication and supersession review; current read-only deployment probes; pinned OS4CSAPI client build and direct parsing experiments; comparison with accepted IDR-SRV-006 through IDR-SRV-014E; ownership, freshness, and drift classification; bounded synthesis<br>
**Research Time:** Approximately 5 hours of AI-assisted elapsed execution on August 31, 2026<br>
**Primary Evidence Baseline:** `Sam-Bolling/csapi-server-interop-secd`, commit `f018fd129bf0d0d1ce75e68198e3ab4d99d937a0` (May 25, 2026)<br>
**Current Deployment Check:** `https://cs.ogc.secd.eu/api/1.0`, read-only probes on August 31, 2026<br>
**Client Baseline:** `OS4CSAPI/ogc-client-CSAPI_2`, branch `phase-9`, commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`, package `@camptocamp/ogc-client` `1.3.1-dev`<br>
**Document Purpose:** Convert the SECD interoperability corpus into freshness-qualified Glaux requirements, risks, validation cases, fixture needs, and downstream handoffs without treating an implementation, client, report-card grade, or test harness as normative authority<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** August 31, 2026<br>
**Date:** August 31, 2026<br>
**Last Updated:** August 31, 2026

---

## Reading Guide

| Label | Meaning |
|---|---|
| N | Normative OGC source or accepted Glaux standards-baseline decision |
| I | Pinned SECD interoperability repository source |
| A | Adjudicated register, supersession crosswalk, or publication-validation artifact |
| H | Historical raw probe or mutation evidence that was not rerun in this topic |
| L | Read-only live SECD result reproduced August 31, 2026 |
| C | Pinned OS4CSAPI client source, build, parser, or direct-client experiment |
| O | Current SECD OpenAPI document inspected August 31, 2026 |
| X | Analyst synthesis or Glaux recommendation; never a standards claim |

Freshness states are **current**, **changed**, **resolved**, **historical-current-unverified**, **not reproduced**, and **unresolved**. “Resolved” means the current deployment no longer exhibits the historical observation; it does not prove when or how the implementation changed. “Current” describes the tested deployment and date, not every SECD version. Historical write findings remain historical because this study had no fresh authorization to mutate the public sandbox.

The controlling engineering posture remains **strict server, tolerant client**: Glaux should emit the canonical contract and reject misleading inputs predictably, while a client may deliberately accept documented legacy variants without weakening server conformance.

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. SECD Interoperability Source Inventory and Evidence Classification
4. Tested Servers, Client Versions, Branches, and Test Conditions
5. SECD API Behavior Interoperability Findings
6. Schema, Model, and Representation Findings
7. Dynamic-Data, Status, Event, and Tasking Findings
8. Cross-Implementation Comparison Findings
9. Client/Test-Harness Limitation Versus Server-Behavior Classification
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

The SECD corpus is most useful as an adversarial interoperability fixture, not as an implementation template or a current scorecard. Its evidence was unusually thorough: two evaluation lanes, a 73-row locked adjudication register, raw HTTP artifacts, controlled mutation/cleanup records, a supersession crosswalk, and twenty publication-validation probes. That material exposes failure modes that a schema-only test suite would miss [I,A].

The deployment has materially changed since the May 2026 corpus. The August check found a working `/collections` resource with four Part 1 collections and working top-level `/observations` and `/commands` collections. Historical findings F01 and F36 are therefore resolved for this deployment. The live inventory contained 8 Systems, 4 Deployments, 5 Procedures, 7 Sampling Features, 13 DataStreams, 11,250,822 Observations, 2 ControlStreams, and 15 Commands [L]. The historical D+/68 report-card result is a May methodology artifact and is not a current assessment; this study neither recalculates nor republishes a grade [A,X].

The highest current risk is silent semantic failure. Canonical `id`, `q`, `parent`, and scoped `resultTime` filters returned the same unfiltered data and `200 OK`, including nonsensical values. On the same deployment, proprietary `parentId` and `status` filters and the `bbox` filter changed the result set. This positive-control evidence shows active filter machinery and makes accidental parameter ignorance more credible than a generally non-filtering API [L]. A client cannot distinguish these wrong results from valid results by status or media type.

The current deployment also demonstrates a discovery composition failure. `/collections` now exists, but the landing page repeats the generic `rel="data"` relation for individual resource links and the collections link. The pinned phase-9 OS4CSAPI client selects the first `data` link, requests `/systems`, then assumes a `collections` array and fails with a null dereference. The server’s ambiguous discovery graph and the client’s first-match/unchecked-shape assumption jointly cause the failure. Neither side alone is an adequate explanation [L,C].

Resource-specific collection wrappers persist for Procedures, DataStreams, Observations, ControlStreams, and Commands. The client accepts the Systems GeoJSON `FeatureCollection` but rejects those wrappers because they contain neither `features` nor `items`. More subtly, individual-resource parsers can return success while discarding meaningful fields: DataStream references and schemas, Observation `datastreamId` and quality, Command `controlStreamId` and status, and much of the ControlStream tasking model. Interoperability tests must therefore assert semantic completeness, not merely “parser did not throw” [L,C].

The current ControlStream uses `inputSchema.type = "DataChoice"`, but the important defect is not the presence of `DataChoice` by itself. The wire model uses `inputSchema` instead of the canonical tasking schema field, and each choice member has `name` and `fields` without the component `type` expected by the pinned SWE parser. The generic ControlStream parser silently drops most tasking content; direct SWE parsing throws. Glaux must test canonical schema location, structure, recursive parseability, and round-trip preservation separately [L,C,X].

The historical write evidence remains strong but dated. It records rejection of GET-shaped GeoJSON bodies, top-level-only create routes, `application/geo+json` rejection, `PUT` returning `200` plus a body, missing PATCH/direct child deletion, a database-level error on exact GET-to-PUT replay, and an intentional anonymous sandbox posture. The May 25 targeted validation reconfirmed the read-modify-write failure and restored the resource after controlled mutation. Those behaviors were not rerun here and are classified historical-current-unverified [H,A].

For Glaux, the evidence supports a canonical link graph, declared-and-implemented route parity, strict parameter recognition, negotiated-media truthfulness, uniform collection envelopes, canonical cross-references, machine-readable errors, semantic parser assertions, and a fixture corpus containing both valid and known-bad SECD-shaped variants. The correct outcome is not a SECD compatibility mode in the server. It is a standards-correct server plus explicit external-client and legacy-fixture tests [N,X].

---

## 2. Scope and Plan Alignment

### 2.1 Included Scope

- The pinned SECD evidence repository and its evaluation, adjudication, supersession, validation, report-card, capture, and log material.
- Fresh read-only probes of the public deployment’s discovery, collections, items, filters, representation negotiation, errors, dynamic-data routes, and OpenAPI document.
- Direct use of the pinned OS4CSAPI phase-9 client against the current deployment and current response shapes.
- Comparison with the accepted Glaux standards, behavior, implementation, and client-smoke studies through IDR-SRV-014E.
- Glaux implications for models, links, filtering, representations, errors, OpenAPI, dynamic data, tasking, validation, fixtures, conformance, and client interoperability.

### 2.2 Excluded Scope

- Treating the SECD repository as the SECD server source repository.
- Regrading SECD, generalizing one deployment to all versions, or treating report-card deductions as requirements.
- Fresh POST, PUT, PATCH, DELETE, command execution, authentication testing, load testing, or certificate remediation.
- Repeating the implementation inventory already accepted in IDR-SRV-014D except where drift changes interoperability conclusions.
- Turning findings into engineering tickets, selecting final Rust mechanisms, or deciding topics owned by IDR-SRV-015 and later.
- OS4CSAPI organization discussion analysis owned by IDR-SRV-014G.

### 2.3 Plan Coverage

All five core and 27 detailed questions are answered. The six phases were completed as: source inventory; endpoint extraction; model/dynamic/tasking analysis; cross-implementation comparison; test/fixture handoff; and synthesis. The required seventeen-section structure and thirteen-field findings matrix are present. Appendix A provides the question ledger.

---

## 3. SECD Interoperability Source Inventory and Evidence Classification

### 3.1 Pinned Evidence Package

The reviewed repository was the private `Sam-Bolling/csapi-server-interop-secd` repository at commit `f018fd129bf0d0d1ce75e68198e3ab4d99d937a0`, dated May 25, 2026. Remote `main` still resolved to that commit on August 31, so the pin also represented the repository head at review time [I]. GitHub reported 60 closed issues, no open issues, and no pull-request records; the last push remained May 25. The clone contained 1,142 files, including 261 Markdown files, 468 `.http` request artifacts, 141 response-body files, 141 response-header files, and 131 JSON files. Counts are inventory context, not independent quality claims.

| Source | Role | Governing use |
|---|---|---|
| `analysis/00-server-snapshot.md` | Initial server and route snapshot | Historical orientation only |
| `analysis/01-initial-compatibility-report.md` | Early compatibility hypotheses | Superseded where adjudication or current probes differ |
| Two evaluation reports | Independent evaluation lanes | Historical observations and raw anchors |
| `pre-grading-evidence-register-v1.md` | Locked 73-row adjudication register | Controlling historical finding disposition |
| Supersession crosswalk | Approved wording/status bridge | Prevent revival of refuted or over-broad claims |
| Issues #1–#17 | Initial evaluation work segmentation | Historical execution/provenance anchors |
| Issues #18–#41 | Reinvestigation and independent validation | Probe ownership and dual-lane reproduction anchors |
| Issues #43–#60 | Register creation, adjudication, tie-breaks, cleanup, and lock audit | Historical decision/provenance anchors; especially SECD write issues #26/#37 and severity decision #47 |
| Targeted validation TV-001–TV-020 | May 25 publication checks | Latest pinned historical validation |
| `secd-report-card.md` | Publication synthesis | Historical methodology output; not normative/current |
| Raw `.http`, bodies, headers, JSON | Wire evidence | Strongest historical observation evidence |

The register contains F01–F63 and CF01–CF10, 73 adjudicated rows in total. The repository records six deployments drawn from four implementation codebases and a seven-segment, 27-step evaluation protocol [I,A]. The corpus is unusually traceable, but its breadth does not eliminate deployment drift.

### 3.2 Evidence Precedence

1. Published normative standards and accepted Glaux standards baselines control requirements [N].
2. Current raw wire behavior controls statements about the August 31 deployment [L].
3. The locked register and supersession crosswalk control interpretation of historical evaluation claims [A].
4. Historical raw captures support dated behavior, especially mutation findings not safely rerun [H].
5. Report-card prose and grades are synthesis artifacts, not primary evidence [A].
6. Client exceptions and parsed objects describe integrated behavior; raw server bytes and client source are needed to assign ownership [C,L].

### 3.3 Freshness Results

| State | Findings/examples | Interpretation |
|---|---|---|
| Current | Ambiguous `data` discovery; wrappers; plain type labels; canonical filters ignored; negotiation ignored; Properties/events/status-result gaps; parser loss | Reproduced August 31 |
| Changed | Inventory counts, observation volume, OpenAPI/runtime surface | Mutable deployment; date-stamp all claims |
| Resolved | Historical `/collections` absence (F01); top-level Observations absence (F36); top-level Commands now present | Remove from current defect lists; retain correction history |
| Historical-current-unverified | Write shapes/routes/media, PUT status, direct delete/PATCH, anonymous writes, GET/PUT replay | Strong May evidence; no fresh mutation |
| Unresolved | Exact implementation root causes and release/version mapping | No server source or immutable deployment identifier |
| Not reproduced | No selected claim was declared current after a failed targeted read-only reproduction | Unsupported claims were omitted or kept historical |

---

## 4. Tested Servers, Client Versions, Branches, and Test Conditions

### 4.1 Historical Matrix

The evidence project evaluated S1 Oracle OpenSensorHub, S2 Oracle Connected Systems Go v2, S3 Oracle 52°North/pygeoapi, S4 SECD, S5 DigitalOcean OpenSensorHub, and S6 52North live. This report uses S4 as its focal deployment and the other rows only for bounded recurrence comparisons [I,A]. Historical test dates and exact request artifacts remain in the pinned repository.

### 4.2 Current Server Conditions

| Attribute | Value |
|---|---|
| API base | `https://cs.ogc.secd.eu/api/1.0` |
| Probe date | August 31, 2026 |
| Operations | GET/HEAD-equivalent read-only inspection only |
| TLS | Certificate validation failed due an expired presented certificate; explicit verification bypass was required for research probes |
| Landing/conformance | `200 application/json` |
| OpenAPI | OpenAPI 3.1.0; API version 1.0.0; 32 paths; 46 operations |
| Mutable-data caution | Counts and identifiers are observations at probe time, not durable fixtures |

The absence of a deployment build identifier means “current” is date- and endpoint-specific. The TLS condition is operational context, not a CSAPI conformance grade.

### 4.3 Current Client Conditions

The client baseline was `OS4CSAPI/ogc-client-CSAPI_2`, branch `phase-9`, commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`, package `@camptocamp/ogc-client` `1.3.1-dev`. The Node-targeted bundle was built locally with Vite under Node 26.8.1/npm 11.19.0. IDR-SRV-014E had already established five passing integration suites, 86 passing tests, and a passing type-check for this pin. This topic used the built client directly against current SECD responses and parser entry points [C].

### 4.4 Reproduction Limits

- TLS bypass changes transport verification only; it does not alter HTTP payload behavior.
- Live counts can change between requests.
- No write claim is promoted to current without a fresh mutation probe.
- The client’s own fixtures prove its expected model, not the standard by themselves.
- Server source and deployed release identity were unavailable, so root-cause labels are behavioral classifications.

---

## 5. SECD API Behavior Interoperability Findings

### 5.1 Consolidated Findings Matrix

Each row contains the plan’s thirteen required fields: ID; source; evidence; server/client; endpoint/area; observed; expected/reference; root-cause classification; scope; Glaux implication; test/fixture; handoff; notes.

| ID | Source anchor | Evidence | Server/client | Endpoint/area | Observed behavior | Expected/reference | Root-cause class | Scope | Glaux implication | Test/fixture | Handoff | Notes/state |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SF-01 | F02, current root, direct client | A/L/C | SECD + phase-9 | Landing discovery | Repeated generic `rel=data`; client follows first `/systems` link and dereferences missing `collections` | Unambiguous canonical discovery graph [N] | Shared server metadata + client first-match/null-guard | Cross-implementation pattern | Emit canonical typed relations and coherent collections link | Multi-`data` ordering and shape tests | 014G, 017, 056 | Current; generic `data` is not itself a strict defect |
| SF-02 | F01, TV corpus, current `/collections` | A/L | SECD | Collections bootstrap | `/collections` now returns four Part 1 collections | Reachable collections bootstrap [N] | Historical deployment gap | SECD drift | Never carry stale absence claims | Route-existence drift check | 050, 056 | Resolved |
| SF-03 | F03, current collections, client parser | A/L/C | SECD + phase-9 | Collection envelopes | Resource-name keys remain for most families; client requires `features` or `items` | Canonical representation schemas [N] | Server representation + strict client assumption | Cross-implementation envelope diversity | Use one canonical envelope per schema | Golden envelopes and malformed-wrapper negatives | 015, 023, 053, 056 | Current; data remains accessible raw |
| SF-04 | F04, current Systems | A/L/C | SECD + phase-9 | Semantic type | `properties.type` is `Drone`/`Sensor`; no `featureType`; client classification returns null | Canonical semantic type/reference model [N] | Server model encoding | Cross-implementation semantic diversity | Keep domain type, feature type, and schema identity distinct | URI/CURIE/plain-label fixtures | 015, 023, 056 | Current |
| SF-05 | F21, current items | A/L | SECD | Navigation links | Subresources use `rel=alternate` rather than typed CSAPI relations | Canonical typed relation identities [N] | Server link model | Cross-implementation | Links must be machine-traversable without URL guessing | Relation identity/target graph tests | 017, 050, 056 | Current |
| SF-06 | F10, current conformance | A/L | SECD | Conformance | 17 implementation-era/noncanonical URI forms remain | Exact published conformance URIs [N] | Declaration/documentation | Cross-implementation declaration drift | Declare only exact implemented classes | URI allow-list + behavior verification | 050, 051 | Current |
| SF-07 | Current OAS/runtime | L/O | SECD | API description | `/collections` works but is absent from OAS; declared status PATCH exists while status GET/result/events are absent | Description/runtime parity [N] | Documentation/deployment drift | Cross-implementation | Generate/test OAS from the same route contract | Bidirectional route diff | 014G, 023, 050 | Current |
| SF-08 | F34, TV-001, August probe | A/L | SECD | `GET /systems?id=` | Known and nonsense IDs both return all 8 Systems, `200` | Canonical filter narrows or invalid input fails [N] | Server parameter handling | Cross-implementation risk | Never silently ignore recognized parameters | Baseline/known/unknown metamorphic triple | 011, 050, 053 | Current; P1 in historical method |
| SF-09 | CF01, TV-002, August probe | A/L | SECD | `GET /systems?q=` | Plausible and nonsense text both return all 8 Systems, `200` | Query has defined semantics or explicit rejection [N] | Server parameter handling | Cross-implementation risk | Same as SF-08 | Positive/negative text corpus | 011, 050, 053 | Current |
| SF-10 | F63, TV-003, August probe | A/L | SECD | System hierarchy filter | Canonical `parent` returns 8; proprietary `parentId` returns 2 children for a current parent | Canonical relationship filter [N] | Server naming divergence | SECD-specific spelling; general alias risk | Canonical names on wire; aliases never replace them | Parent graph with zero/one/many children | 011, 015, 017, 053 | Current; positive control |
| SF-11 | F39, TV-004, August probe | A/L | SECD | Scoped Observations `resultTime` | `latest`, exact, interval, and nonsense return same first 3 IDs and `numberMatched=3482` | Defined temporal filtering or explicit validation [N] | Server parameter handling | Cross-implementation temporal risk | Make temporal grammar and sentinel semantics observable | Fixed time-series metamorphic fixture | 011, 034, 050, 053 | Current |
| SF-12 | August `status`/`bbox` probes | L | SECD | Filter positive controls | `status=active` returns 3 Systems; out-of-range `bbox` returns 0 | Implemented filters alter result sets [N] | Working filter paths | Comparison control | Require controls in every filter test | Known hit/miss spatial and enum fixtures | 011, 053 | Current pass paths |
| SF-13 | F30/F31/F33, TV-010, August probes | A/L | SECD | Negotiation/format | Accept JSON, GeoJSON, SensorML, invalid and `f` values JSON, GeoJSON, HTML, invalid all return the same JSON collection and content type | Honest selection or 4xx/406 [N] | Server negotiation | Cross-implementation | Content-Type, body schema, and selector must agree | Accept × `f` matrix with byte hashes | 012, 023, 050, 053 | Current |
| SF-14 | Current client collection parsing | L/C | SECD + phase-9 | Generic parsing | Systems parse; five resource-name wrappers throw missing `features`/`items` | Parser support aligned with declared inputs | Client shape coverage + server noncanonical envelopes | Shared | Test both canonical strictness and explicit adapters | Per-family raw and parsed golden files | 023, 053, 056 | Current |
| SF-15 | F40, current parser objects | A/L/C | SECD + phase-9 | Cross-references/status | Parsers succeed but drop IDs, schema, observed property, quality, and status fields | Semantically complete parsed model [N] | Server field names + tolerant parser loss | Cross-implementation | “No exception” is not an interop success criterion | Field-preservation assertions | 015, 017, 023, 056 | Current |
| SF-16 | F05/CLX-029, current ControlStream | A/L/C | SECD + phase-9 | SWE/tasking schema | `inputSchema` DataChoice with untyped choice members; generic parser drops it; SWE parser throws | Canonical tasking schema location and valid recursive components [N] | Server schema shape + client model boundary | SECD-specific shape; general tasking risk | Validate location, structure, parse, and round trip independently | Valid/invalid nested DataChoice fixtures | 023, 036, 053, 056 | Current; do not call DataChoice itself invalid |
| SF-17 | F62, current route/root | A/L | SECD | Properties | `/properties` remains 404 and unlinked | Required resource availability per claimed classes [N] | Server surface gap | Cross-implementation | Conformance declarations must match families | Declared-class route graph test | 015, 017, 050 | Current |
| SF-18 | F36 and current inventory | A/L | SECD | Top-level Observations/Commands | Both top-level collections now return data | Published resource routes [N] | Historical deployment gap | SECD drift | Revalidate historical negatives | Route and count smoke test | 034, 036, 056 | Resolved for both routes |
| SF-19 | Current routes/OAS | L/O | SECD | Events/task lifecycle | `/systemevents` absent; no status/result read resources; OAS only declares status PATCH | Canonical claimed dynamic/tasking lifecycle [N] | Incomplete server surface | Cross-implementation | Model full lifecycle before advertising support | Command state/result/event scenario | 034, 036, 050, 053 | Current; feasibility also absent |
| SF-20 | Current errors | L | SECD | Error model | Known validation/not-found routes return JSON; absent lifecycle paths return Express HTML 404 pages | Uniform machine-readable error contract [N] | Framework fall-through | Cross-implementation | Normalize all 4xx/5xx paths | Unknown route/method/media/ID matrix | 013, 050, 053 | Current |
| SF-21 | Current `limit` probes | L | SECD | Pagination validation | 0, -1, 10001, and nonnumeric values return consistent JSON 400; legal request works | Bounded validated pagination [N] | Working validation | Cross-implementation pass path | Preserve as a positive model | Boundary partition tests | 011, 013, 053 | Current pass path |
| SF-22 | CF04, TV-016 | A/H | Historical SECD | GET-to-PUT | Exact GET Feature replay causes DB null-name 500; flat/hybrid workaround succeeds, with data-loss risk | Safe documented update representation and validation [N] | Read/write schema asymmetry + error leakage | Cross-implementation round-trip risk | Derive write schema deliberately; reject before persistence | Create→read→replace→read invariant | 023, 050, 053 | Historical-current-unverified |
| SF-23 | F50/F51/F52/F57 | A/H | Historical SECD | Writes/routes/media/status | GeoJSON wrappers/media rejected; creates top-level only; PUT 200+body | Canonical operations, media, and success semantics [N] | Server write contract divergence | Cross-implementation lifecycle risk | Test every documented write shape and route | Full CRUD media/route matrix | 023, 036, 050, 053 | Historical-current-unverified |
| SF-24 | F53/F56 | A/H | Historical SECD | Delete/PATCH | Direct child deletion and PATCH absent; parent cascade cleaned children | Declared lifecycle semantics [N] | Incomplete method/route surface | Cross-implementation | Explicitly define cascade, direct delete, and partial update | Referential-integrity lifecycle scenario | 015, 036, 050, 053 | Historical-current-unverified |
| SF-25 | F58/CLX-026 | A/H | Historical SECD | Authentication | Anonymous/invalid-auth writes accepted intentionally on sandbox | Deployment-specific security policy | Security posture, not conformance | SECD deployment context | Never inherit sandbox posture into production profile | Authn/authz matrix in controlled env | 055 | Ungraded historical context |
| SF-26 | Current certificate and no build ID | L/O | SECD | Reproducibility/operations | Expired certificate; OAS version does not identify deployment build | Stable test identity and valid transport | Operational/documentation | General live-test risk | CI must not depend solely on mutable public demos | Immutable container + local cert test | 050, 053, 054, 056 | Current operational constraint |

### 5.2 Finding Concentration

The findings concentrate in five compositional boundaries:

1. **Metadata to behavior:** landing links, conformance URIs, and OpenAPI do not reliably identify reachable behavior.
2. **Parameter to result:** canonical parameters are accepted but not applied.
3. **Wire shape to client model:** parsers either throw on envelopes or silently lose fields.
4. **Read shape to write shape:** historical representations are not safely round-trippable.
5. **Tasking schema to lifecycle:** schema, command, status, result, and event surfaces do not form one traversable state machine.

The silent paths are more dangerous than explicit failures because monitoring and clients can mistake them for success.

---

## 6. Schema, Model, and Representation Findings

### 6.1 Collection and Item Shapes

Systems use GeoJSON `FeatureCollection`/`Feature` shapes. Other families use keys such as `procedures`, `deployments`, `datastreams`, `observations`, `controlstreams`, and `commands`. The pinned client’s collection parser accepts the Systems response and rejects the five tested resource-name wrappers. A raw-data consumer can still reach the data, but a schema-driven generic consumer cannot [L,C].

Glaux should select the canonical representation per resource and apply it uniformly to empty, one-item, paginated, scoped, and top-level collections. Empty collections are especially important: an empty but correctly shaped array is not equivalent to an absent envelope.

### 6.2 Cross-Reference Preservation

Current DataStreams use `systemId`, `deploymentId`, and `samplingFeatureId`; Commands use `controlStreamId`; Observations use `datastreamId`. The pinned client can construct objects without throwing yet omit these fields because its model expects canonical reference forms. The same pattern drops observed-property, unit, output-schema, result-quality, and status information [L,C].

Tests must compare a required-field manifest before and after parsing. Serializing the parsed object back to JSON and diffing semantic fields is a stronger check than asserting an object type.

### 6.3 Type and Schema Identity

The current Systems provide useful plain labels such as `Drone` and `Sensor`, but those labels do not substitute for a canonical feature-type identifier. Glaux should keep human-facing category, normative feature type, schema identity, and polymorphic discriminator separate even when a representation makes one of them optional [N,X].

### 6.4 SWE Common and Tasking

The first current ControlStream contains an `inputSchema` with `type: DataChoice` and five choices. A representative choice contains a `name` and a `fields` array of typed Quantity components, but lacks the component-level `type` required by the pinned recursive SWE parser. Direct parsing fails with: `DataChoice field "GOTO" must have a "type" property or be a link reference`. The ControlStream parser returns an object but discards the schema, system reference, and status [L,C].

The earlier proposition “DataChoice is invalid here” was standards-ambiguous and must not be revived. The evidence supports narrower assertions: noncanonical schema location, structurally incompatible choice members for the tested client, and silent information loss.

---

## 7. Dynamic-Data, Status, Event, and Tasking Findings

### 7.1 Observations and Temporal Queries

Top-level Observations are now present, resolving the May route finding. Scoped observations are abundant and usable without the temporal filter, but `resultTime` remains inert for `latest`, exact, interval, and nonsense values. The same three records and `numberMatched=3482` were returned in the bounded August comparison. Glaux tests need fixed timestamps and assertions on identifiers, counts, ordering, boundaries, and invalid grammar—not only status codes [L].

### 7.2 ControlStreams and Commands

Two ControlStreams and fifteen top-level Commands were visible. This proves current read-surface presence, not task execution completeness. The absence of status GET, result, feasibility, and events prevents a generic client from traversing a complete command lifecycle. An OAS-declared `PATCH /commands/{id}/status` is not a substitute for state observation [L,O].

### 7.3 Status and Events

The top-level `/systemevents` route returned an HTML 404 and was absent from OAS. Command status and result reads behaved similarly. Glaux should define whether state is embedded, linked, or separately addressable, then make landing metadata, resource links, OAS, authorization, and error semantics agree [N,X].

### 7.4 Historical Lifecycle Evidence

The May corpus includes controlled write and cleanup evidence. It is relevant to test design because it demonstrated failures that reads alone cannot reveal: rejected GET-shaped updates, destructive loss of descriptive properties under a flat workaround, route asymmetry, wrong success status, cascade-only cleanup, and missing PATCH. This topic did not repeat those mutations. A future Glaux interoperability harness should run them against disposable isolated data, never a shared public resource [H,A,X].

---

## 8. Cross-Implementation Comparison Findings

### 8.1 Recurring Patterns

| Pattern | SECD evidence | Other accepted-study evidence | Glaux lesson |
|---|---|---|---|
| Discovery relation mismatch | Repeated generic `data`, ambiguous first match | OSH and pygeoapi variants also challenged client discovery | Test canonical discovery plus tolerant-client fallbacks separately |
| Envelope diversity | Resource-name wrappers | OSH `items`, cs-go hybrids, pygeoapi route/media differences | One canonical Glaux contract; variant fixtures only for clients |
| Metadata/runtime drift | OAS omits `/collections`; lifecycle incomplete | Observed in multiple implementation studies | Bidirectional generated-contract checks |
| Silent query failure | `id`, `q`, `parent`, `resultTime` | Query edge defects recur, though exact parameters differ | Metamorphic hit/miss tests and unknown-parameter policy |
| Negotiation inconsistency | Accept and `f` ignored | Partial SensorML and media-selection variation elsewhere | Cross-product negotiation tests |
| Read/write asymmetry | Historical GET body cannot PUT | Also visible in other implementations and phase-9 pygeoapi | Round-trip invariant suite |
| Tasking lifecycle incompleteness | Schema/status/result/events do not compose | OSH and client smoke studies found route-specific lifecycle gaps | End-to-end state-machine scenarios |

### 8.2 SECD-Specific Shapes

The exact `parentId` alias, resource-name wrappers, flat write fields, `inputSchema` structure, current link ordering, and Express HTML fall-through are deployment-specific. Glaux should not copy them. They are useful negative fixtures because they exercise generic failure categories likely to recur under different spellings.

### 8.3 Comparison Boundary

IDR-SRV-014A–014D describe implementations; IDR-SRV-014E describes the broader client smoke corpus; this report adjudicates the dedicated SECD interoperability evidence and current drift. IDR-SRV-014G may add community rationale, but discussion consensus cannot override published requirements. IDR-SRV-056 should combine the exact external-client cases into a repeatable matrix.

---

## 9. Client/Test-Harness Limitation Versus Server-Behavior Classification

### 9.1 Ownership Decisions

| Outcome | Server contribution | Client/harness contribution | Classification |
|---|---|---|---|
| Initialization crash | Ambiguous repeated `data` links and heterogeneous targets | Selects first `data`; assumes `collections`; lacks shape guard | Shared composition failure |
| Wrapper parse errors | Noncanonical resource-name envelopes | Parser only recognizes `features`/`items` | Server behavior exposed by bounded client coverage |
| Null resource type | Plain labels/no canonical discriminator | Classifier intentionally recognizes its modeled forms | Primarily server model divergence |
| Successful parse with missing fields | Noncanonical names and locations | Parser returns partial objects without completeness signal | Shared; silent loss is client risk |
| DataChoice exception | Noncanonical schema location/member shape | Recursive parser expects typed component/link | Shared schema-contract mismatch |
| PowerShell single-item unwrapping in historical work | None | Harness transformed an array | Harness defect; retracted in source corpus |
| Canonical filter returns all data | Parameter silently ignored despite working controls | Client cannot infer semantic failure | Server behavior |
| HTML 404 on absent lifecycle routes | Framework fall-through | None needed to observe | Server error-contract behavior |

### 9.2 Adjudication Rule

For every interoperability failure, preserve four artifacts: raw request, raw response, parsed result/exception, and the client code or test assertion. Assign ownership only after comparing them. A workaround can reduce user impact without making the server canonical; a strict parser can expose a valid server requirement or encode an overly narrow assumption. Standards evidence decides that boundary [N,X].

### 9.3 Harness Lessons

- Preserve bytes and headers before JSON conversion.
- Avoid shell display formatting as evidence.
- Use explicit known-hit and known-miss controls.
- Record IDs selected from the same live run.
- Keep mutation fixtures isolated and self-cleaning.
- Detect semantic field loss after successful parse.
- Time-stamp mutable deployment observations and pin all repositories.

---

## 10. Standards Ambiguity and Documentation-Gap Findings

### 10.1 Closed or Narrowed Claims

- Generic `rel=data` is permitted by OGC API Common; it is not a scored defect solely because a typed relation would be clearer. The current problem is ambiguity plus client selection behavior.
- `DataChoice` presence is not sufficient to claim nonconformance. Schema location and internal component structure are the reproducible issues.
- Historical missing `/collections` and top-level Observations claims are resolved on the August deployment.
- Anonymous writes were an intentional sandbox posture and remain ungraded operational/security context.
- The May D+/68 grade is not a current conformance conclusion.

### 10.2 Remaining Precision Needs

- Canonical relation choices and identity invariants should be finalized in IDR-SRV-017.
- Unknown-query-parameter and unsupported-representation policy should be made explicit across IDR-SRV-011–013.
- Canonical tasking schema and recursive SWE validation belong to IDR-SRV-023 and IDR-SRV-036.
- Embedded versus linked status/result/event resources require lifecycle design, not inference from SECD.
- The future harness must distinguish requirement tests, schema tests, interoperability tests, and robustness extensions.

### 10.3 Documentation Gaps

The current OAS is not a complete runtime inventory: it omits the working `/collections` endpoint and does not expose several current gaps as an intentional capability profile. It has no security schemes and no immutable build metadata. The landing page similarly lacks an unambiguous typed bootstrap graph. Glaux documentation should be generated or validated from the same source as routing and authorization policy [O,X].

The shared upstream-history register version 1.8 was date-checked for relevant link, filter, schema, scoped-route, read/write-shape, sorting, recursive-model, and bulk-operation entries. No material upstream change altered the accepted normative baseline for this topic. Upstream history explains design evolution but is not authority over the published standards [N].

---

## 11. Test Strategy and Fixture Implications

### 11.1 Required Test Layers

1. **Normative conformance tests:** one assertion per requirement and exact conformance URI.
2. **Schema tests:** validate canonical success and error representations independently of runtime routing.
3. **Golden-wire tests:** preserve status, headers, content type, and exact/normalized body.
4. **Metamorphic behavior tests:** compare baseline, known-hit, known-miss, invalid, and alias parameters.
5. **Lifecycle tests:** create, read, replace, patch, delete, cascade, status, result, event, and cleanup in an isolated store.
6. **External-client tests:** run pinned clients black-box and assert semantic completeness.
7. **Metadata parity tests:** diff routes against landing links, conformance, OAS, and authorization declarations.

### 11.2 Minimum Fixture Families

| Fixture family | Required variants |
|---|---|
| Discovery | Canonical typed links; repeated generic links; missing target; wrong target shape; reordered links |
| Collections | Empty/one/many; `features`; `items`; resource-name wrapper; missing counts; pagination links |
| Identity/relationships | Canonical IDs; nonexistent IDs; parent with zero/one/many children; canonical and legacy aliases |
| Queries | Exact hit; miss; invalid grammar; repeated parameter; encoded time; open interval; `latest`; bbox hit/miss |
| Negotiation | Accept and `f` cross-product; supported/unsupported; body/media agreement; Vary behavior |
| Models | URI/CURIE/plain type; canonical/noncanonical crossrefs; null/absent/extra fields |
| SWE/tasking | Scalar, record, array, choice, nested choice, link reference, invalid untyped member |
| Lifecycle | GET-shaped replacement; canonical write shape; partial update; direct/cascade delete; rollback/cleanup |
| Errors | Validation, not found, wrong method, wrong media, conflict, unauthorized, framework fall-through |
| Drift | Current route added/removed; OAS mismatch; certificate expiry; mutable count changes |

### 11.3 Most Useful SECD Tests

The strongest tests used comparisons rather than isolated requests: canonical versus proprietary parameter, baseline versus nonsense value, Accept/body hash comparisons, exact GET replay versus server-specific update shape, and raw response versus parsed-object completeness. These should become reusable harness primitives.

### 11.4 Tests to Avoid or Repair

- Do not infer filter success from `200` or nonempty output.
- Do not treat one known ID as a positive control unless the baseline and resulting identifiers are compared.
- Do not use live counts as golden values.
- Do not infer array/object shape from PowerShell-rendered output.
- Do not score an absent route without checking declared conformance and accepted normative applicability.
- Do not mutate shared live records without authorization, snapshot, cleanup, and verification.
- Do not count a successful parser call as success until required semantics are preserved.

### 11.5 CI Allocation

Deterministic canonical and negative fixtures belong in every CI run. Pinned external-client tests should run on merge or nightly schedules. Mutable public-server probes belong in a separately labeled observational lane and must not gate Glaux builds on third-party availability, TLS, data volume, or unannounced drift.

---

## 12. Lessons for Glaux Server

### 12.1 Adopt

- A single canonical resource and relationship vocabulary.
- Exact typed discovery links plus a coherent collections bootstrap.
- Explicit validation for every advertised parameter and selector.
- Uniform JSON problem/error responses across routed and unrouted failures.
- Canonical schemas shared by read, write, OpenAPI, validation, and generated fixtures.
- Full raw-wire and semantic round-trip tests.
- An immutable, seeded interoperability environment.

### 12.2 Avoid

- Silent acceptance of ignored filters or formats.
- Multiple unrelated targets under an indistinguishable discovery relation.
- Resource-family-specific envelopes without normative basis.
- Framework-generated HTML errors escaping the API boundary.
- OpenAPI and conformance declarations maintained independently of runtime capabilities.
- Parsers or adapters that silently discard required semantics.
- Treating an external implementation’s shortcut as a Glaux compatibility requirement.

### 12.3 Validate Explicitly

- Every link relation, target, media type, and identity transition.
- All query boundary partitions and known-hit/miss controls.
- Content negotiation as status + header + body-schema agreement.
- Empty and paginated variants of every collection.
- Recursive SWE components and tasking schemas.
- Complete command state/result/event traversal.
- Write/read/update/delete preservation and cleanup.
- Route/OAS/conformance/security parity.

### 12.4 Preserve as Research Evidence

Retain the pinned SECD-shaped negative fixtures, but label their provenance and expected failure. They should test Glaux clients and harnesses, not expand the server’s public contract. Preserve the locked register and raw captures as dated evidence rather than copying report-card prose into implementation requirements.

---

## 13. Downstream Topic Handoff Matrix

| Downstream topic | Handoff |
|---|---|
| IDR-SRV-014G | Seek community rationale for discovery, envelopes, filters, and tasking differences; keep discussion evidence informative |
| IDR-SRV-015 | Define canonical resource fields, feature/type identity, cross-reference forms, and required semantic completeness |
| IDR-SRV-017 | Define typed relationship graph, link identity/target rules, hierarchy, and traversal invariants |
| IDR-SRV-023 | Unify wire schemas, OpenAPI, runtime validation, SWE recursion, read/write shapes, and negative-schema tests |
| IDR-SRV-034 | Define Observation access, temporal filtering, latest semantics, ordering, counts, status, and events |
| IDR-SRV-036 | Define ControlStream schema, Command creation, feasibility, status, result, events, cancellation, and terminal states |
| IDR-SRV-050 | Build conformance, metadata parity, unknown-input, error, negotiation, and lifecycle harness layers |
| IDR-SRV-051 | Trace each applicable requirement to a positive and negative test; keep interop observations separate |
| IDR-SRV-053 | Include the fixture families and SECD-shaped adversarial corpus in seeded golden data |
| IDR-SRV-054 | Keep public-server availability and mutable volume out of deterministic performance baselines |
| IDR-SRV-055 | Test authentication/authorization on every command and mutation path; do not inherit sandbox posture |
| IDR-SRV-056 | Pin client/server versions; assert initialization, traversal, parsing, field preservation, lifecycle, and error handling |
| IDR-SRV-057 | Synthesize silent-success risk, strict-server/tolerant-client posture, and evidence freshness discipline |

---

## 14. Recommendations

1. Make silent wrong-result prevention a release-blocking API invariant: a supported parameter changes/validates results, and an unsupported parameter follows the explicit project policy.
2. Generate landing links, conformance declarations, OpenAPI paths, and runtime routes from one capability model or continuously diff them.
3. Use canonical typed link relations even where generic `data` is technically permitted; test link-order independence in external clients.
4. Standardize collection envelopes and canonical cross-reference fields before implementation topics begin.
5. Require semantic-completeness assertions for all client interoperability tests.
6. Define one canonical tasking schema location and validate recursive SWE structures before accepting ControlStreams.
7. Build a disposable lifecycle environment for write/media/round-trip/cascade tests.
8. Normalize all errors before framework fall-through and prevent storage-engine details from reaching clients.
9. Preserve SECD variants as provenance-labeled negative fixtures, not supported Glaux server encodings.
10. Record deployment build identity, standard-package version, OpenAPI digest, and fixture revision in every future test run.

---

## 15. Risks, Constraints, and Open Questions

### 15.1 Constraints

- The evidence repository is access-controlled and is not server source.
- The public deployment is mutable and exposes no immutable release identity.
- Its certificate was expired at the time of the current probe.
- Fresh mutation testing was outside this topic’s authorization.
- The pinned client is one consumer, not a normative oracle.

### 15.2 Risks and Controls

| Risk | Control |
|---|---|
| Stale May findings presented as current | Per-finding freshness state and August read-only rerun |
| Report-card score mistaken for conformance proof | Treat score as historical synthesis only; use standards baselines |
| Server blamed for client/harness error | Preserve raw wire, parser output, code expectation, and standard anchor |
| Client workaround copied into server contract | Strict-server/tolerant-client boundary |
| Live data or TLS makes CI flaky | Seeded immutable local fixtures; observational live lane only |
| Mutation corrupts shared data | Disposable isolated environment and verified cleanup |
| Parser silently loses semantics | Required-field manifest and round-trip diff |
| OpenAPI creates false confidence | Bidirectional metadata/runtime behavioral checks |

### 15.3 Resolved Open Questions From the Plan

- **Best current artifacts:** the locked register, supersession crosswalk, targeted validations, raw captures, current probes, and pinned client source together; no single report is sufficient.
- **Current reproducibility:** canonical filter, negotiation, wrapper, link, model, tasking-schema, error-split, and lifecycle-route findings were reproduced read-only; three route absences were resolved; writes remain historical-current-unverified.
- **Ambiguity versus defect:** generic `data` and DataChoice presence require narrowed treatment; ignored canonical parameters, misleading negotiation, noncanonical wrappers/crossrefs, and metadata/runtime drift are reproducible behavior gaps against the accepted baseline.
- **CI candidates:** metadata parity, filter metamorphic tests, negotiation matrix, envelope/schema goldens, semantic parser preservation, uniform errors, and isolated lifecycle round trips.

No open question blocks the report. Later design topics must choose exact models and policies within the accepted normative baseline.

---

## 16. Validation Against the Research Plan

### 16.1 Success Criteria

| Criterion | Result | Evidence |
|---|---|---|
| Exact source identity and conditions | Met | Repository/client commits, date, server base, test limits, inventory |
| Ownership distinctions | Met | Section 9 and thirteen-field matrix |
| Grouped findings and strength | Met | Sections 5–7 and freshness taxonomy |
| Reproducible/stale/inferred/unresolved separation | Met | Section 3.3 and matrix notes |
| Cross-implementation versus SECD-specific | Met | Section 8 and matrix scope |
| Full implication coverage | Met | Sections 6, 7, 10–12 |
| Explicit tests and fixtures | Met | Section 11 and matrix test column |
| Explicit downstream handoffs | Met | Section 13 and matrix handoff column |
| Reproducible references | Met | Section 17 and Appendix B |

### 16.2 Deliverable Structure

All seventeen required content areas are present in numbered sections. The consolidated matrix includes all thirteen required fields. Appendix A maps all 27 detailed questions. No acceptance is claimed while the report is In Review.

### 16.3 Quality Checks

- Normative, historical, current-live, client, OAS, and synthesis evidence are labeled.
- The May score is not reused as a current assessment.
- F01 and F36 are explicitly marked resolved; top-level Commands are also recorded as present.
- Write claims are explicitly historical-current-unverified.
- `rel=data` and DataChoice claims are narrowed to avoid false normative assertions.
- Positive controls accompany the silent-filter conclusion.
- Current probes were read-only and reproducible from documented endpoints/values.
- No ticketing or downstream design decision was performed.

---

## 17. References

### 17.1 Controlling Standards and Glaux Sources

- [OGC API - Connected Systems](https://ogcapi.ogc.org/connectedsystems/)
- [OGC API - Connected Systems Part 1: Feature Resources](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems Part 2: Dynamic Data](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC API - Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC API - Connected Systems development repository](https://github.com/opengeospatial/ogcapi-connected-systems)
- [OGC API - Connected Systems v1.0.0 API artifacts](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api)
- [OGC schemas](https://schemas.opengis.net/)
- [Glaux Server overall IDR plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-014D SECD implementation study](idr-srv-014d-secd-csapi-server-implementation-study-report.md)
- [IDR-SRV-014E OS4CSAPI client smoke findings study](idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md)
- [Shared upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)

### 17.2 Pinned SECD Evidence

- [Evidence repository at reviewed commit](https://github.com/Sam-Bolling/csapi-server-interop-secd/tree/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0)
- [Primary evaluation](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/evaluation/csapi-server-evaluation-results.md)
- [Independent secondary evaluation](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/evaluation/csapi-server-evaluation-results-codex.md)
- [Locked adjudicated evidence register](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/reinvestigation/adjudication/pre-grading-evidence-register-v1.md)
- [Supersession crosswalk](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/report-cards/research/evaluation-claims-supersession-crosswalk.md)
- [Publication-ready SECD report card](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/report-cards/publication-ready/secd-report-card.md)
- [Repository issue tracker](https://github.com/Sam-Bolling/csapi-server-interop-secd/issues?q=is%3Aissue), including [SECD Phase B write issue #26](https://github.com/Sam-Bolling/csapi-server-interop-secd/issues/26), [independent SECD write validation #37](https://github.com/Sam-Bolling/csapi-server-interop-secd/issues/37), and [silent-filter severity decision #47](https://github.com/Sam-Bolling/csapi-server-interop-secd/issues/47)
- [Targeted validation closeout](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/report-cards/publication-ready/final-report-card-review/targeted-validation-plan/batch-1-closeout.md)
- [TV-001 canonical ID filter](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/report-cards/publication-ready/final-report-card-review/targeted-validation-plan/artifacts/tv-001-secd-id-filter.md)
- [TV-002 canonical q filter](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/report-cards/publication-ready/final-report-card-review/targeted-validation-plan/artifacts/tv-002-secd-q-filter.md)
- [TV-003 canonical parent filter](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/report-cards/publication-ready/final-report-card-review/targeted-validation-plan/artifacts/tv-003-secd-parent-filter.md)
- [TV-004 resultTime filter](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/report-cards/publication-ready/final-report-card-review/targeted-validation-plan/artifacts/tv-004-secd-resulttime-filter.md)
- [TV-010 SensorML negotiation](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/report-cards/publication-ready/final-report-card-review/targeted-validation-plan/artifacts/tv-010-secd-sensorml-absence.md)
- [TV-016 read-modify-write](https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/report-cards/publication-ready/final-report-card-review/targeted-validation-plan/artifacts/tv-016-secd-read-modify-write.md)

### 17.3 Current Deployment and Client

- [SECD current API base](https://cs.ogc.secd.eu/api/1.0)
- [SECD current OpenAPI document](https://cs.ogc.secd.eu/api-docs/openapi.json)
- [Pinned OS4CSAPI client commit](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230)

---

## Appendix A. Detailed Question Ledger

| # | Detailed question | Answer location |
|---:|---|---|
| 1 | Relevant repositories, commits, issues, notes, tests, logs, scripts, fixtures, discussions | 3.1, 17.2 |
| 2 | Tested server/client/version/endpoint/conditions | Section 4 |
| 3 | Reproducible, partial, anecdotal, stale, superseded, unresolved | 3.3, matrix state |
| 4 | Exact evidence for findings | Section 5 matrix, 17.2 |
| 5 | Material to preserve | 3.1, 12.4, Appendix B |
| 6 | Landing, definition, conformance, collection, item, link, URI, navigation issues | SF-01–SF-07 |
| 7 | Affected resource families | SF-03, SF-17–SF-19; Sections 6–7 |
| 8 | Server gaps, interpretations, client assumptions, links, IDs, paths | Section 9 |
| 9 | Model, identifier, relationship, link handoffs | Sections 6, 13 |
| 10 | Query/filter/pagination/sort/select/time/bbox/count/link issues | SF-08–SF-12, SF-21 |
| 11 | Negotiation/media/schema/SensorML/SWE/GeoJSON/JSON/OAS issues | SF-07, SF-13–SF-16 |
| 12 | Status/error/unsupported/validation/ambiguous failure issues | SF-19–SF-21 |
| 13 | Query/negotiation/error/OpenAPI/schema handoffs | Sections 11, 13 |
| 14 | Observation/datastream/latest/status/event/control/command/feasibility findings | Section 7 |
| 15 | Server-specific versus broader dynamic/tasking differences | 7.4, 8.1–8.2 |
| 16 | Part 2/schema/OAS/expectation ambiguity | 6.4, 10.2 |
| 17 | Dynamic/streaming/lifecycle/feasibility/security handoffs | Section 13 |
| 18 | Recurrence in OSH, cs-go, pygeoapi, client smoke | Section 8.1 |
| 19 | SECD-unique findings | Section 8.2 |
| 20 | External-client expectations Glaux should satisfy | Sections 9, 12 |
| 21 | Future interoperability comparison cases | Sections 11.2, 13 |
| 22 | IDR-SRV-014G and 056 handoffs | Section 13 |
| 23 | Most useful tests | Section 11.3 |
| 24 | Brittle/misleading/insufficient tests | Section 11.4 |
| 25 | Positive/negative/conformance/schema/OAS/fixture/golden/client tests | Sections 11.1–11.2 |
| 26 | Required fixture/scenario entries | Section 11.2 |
| 27 | Future conformance-harness/CI checks | Sections 11.5, 13 |

---

## Appendix B. Reproducible Study Record

### B.1 Repository Identity

```text
repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
branch inspected: main
commit: f018fd129bf0d0d1ce75e68198e3ab4d99d937a0
commit subject: Finalize publication-ready report card cleanup
commit date: 2026-05-25T16:22:50-04:00
remote main at review: same commit
repository visibility: private
GitHub issues: 60 closed, 0 open
GitHub pull requests: 0
files: 1142
markdown: 261
http request artifacts: 468
response body artifacts: 141
response header artifacts: 141
json: 131
```

### B.2 Current Inventory Snapshot

| Resource | Status/result |
|---|---|
| Collections | 4: Systems, Deployments, Procedures, Sampling Features |
| Systems | 8 |
| Deployments | 4 |
| Procedures | 5 |
| Sampling Features | 7 |
| Properties | 404 |
| DataStreams | 13 |
| Observations | 11,250,822 matched |
| ControlStreams | 2 |
| Commands | 15 |
| System Events | 404 |

### B.3 Current Query Reproduction

```text
systems baseline=8
id=<known>=8; id=<nonsense>=8
q=GPS=8; q=<nonsense>=8
parent=<known-current-parent>=8
parentId=<known-current-parent>=2
status=active=3
bbox=<outside-data>=0

datastream=44d64553-b35e-435a-bfc9-6943935d4f9b
observations numberMatched=3482
baseline(limit=3), latest, exact, interval, nonsense:
all status=200; same first three IDs; numberMatched=3482
```

### B.4 Negotiation and Error Reproduction

```text
systems collection:
  f=json|geojson|html|invalid => 200 application/json, same FeatureCollection
  Accept application/json|application/geo+json|application/sml+json|invalid
    => 200 application/json, same representation

handled errors:
  unknown System ID => 404 JSON NotFound
  limit 0|-1|10001|abc => 400 JSON ValidationError

absent lifecycle paths:
  command status GET, command result GET, systemevents GET => 404 HTML
```

### B.5 Client Reproduction

```text
repository: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
branch: phase-9
commit: 754411897173c2ec4debaa9bcf4ed9e0f8a9e230
package: @camptocamp/ogc-client 1.3.1-dev
build: npx vite build --config vite.node-config.js

landing info: succeeds
hasConnectedSystems: true
allCollections/csapiCollections/csapi('systems'):
  TypeError: Cannot read properties of undefined (reading 'map')
manual typed-link endpoint builder: succeeds
Systems collection parser: succeeds
Procedure/DataStream/Observation/ControlStream/Command collection parsers:
  EndpointError: missing both "features" and "items" arrays
System resource-type classifier: null
ControlStream generic parser: succeeds with material field loss
direct inputSchema SWE parser:
  SweCommonParseError: DataChoice field "GOTO" must have a "type" property or be a link reference
```

### B.6 OpenAPI Reproduction

```text
OpenAPI: 3.1.0
info.version: 1.0.0
paths: 32
operations: 46
securitySchemes: absent
runtime /collections: present but not declared
top-level /observations and /commands: declared and present
/commands/{id}/status: PATCH declared; GET absent
/properties, /systemevents, /commands/{id}/result: absent
```

---

## Appendix C. Completion and Handoff

IDR-SRV-014F is complete and accepted by the plan owner. The report is evidence-bounded, drift-qualified, standards-aligned, and ready to inform later model, validation, lifecycle, fixture, conformance, and interoperability topics.

The plan owner accepted IDR-SRV-014F and authorized execution of exactly one next eligible topic, IDR-SRV-014G, on August 31, 2026. This acceptance does not authorize IDR-SRV-015 or any later topic.

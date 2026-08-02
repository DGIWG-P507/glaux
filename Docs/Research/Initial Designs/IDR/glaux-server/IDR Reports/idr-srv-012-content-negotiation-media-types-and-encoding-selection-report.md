# Section 012: Content Negotiation, Media Types, and Encoding Selection - Research Report

**Topic ID:** IDR-SRV-012<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-012 Content Negotiation, Media Types, and Encoding Selection](../IDR%20Plans/idr-srv-012-content-negotiation-media-types-and-encoding-selection.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 28 detailed questions; all six methodology phases, ten success criteria, eighteen required content areas, and thirteen minimum representation-matrix fields are validated<br>
**Methodology Used:** Reconciliation of accepted IDR-SRV-006 through IDR-SRV-011 with approved CSAPI, OGC API, SensorML, SWE Common, HTTP, Web Linking, profile, media-registration, and OpenAPI sources; mechanical review of immutable tagged OpenAPI, schema, and abstract-test artifacts; current IANA and official-repository checks; and three independent read-only Part 1, Part 2, and interoperability audits<br>
**Research Time:** Approximately 5 hours of AI-assisted elapsed execution, including three parallel evidence audits, on August 2, 2026<br>
**Primary Sources:** OGC 23-001, OGC 23-002, OGC 17-069r4, OGC 19-072, OGC 23-000, OGC 24-014, RFC 9110, RFC 9111, RFC 6838, RFC 6839, RFC 6906, RFC 7946, RFC 8259, RFC 8288, the IANA media-type and link-relation registries, and the OpenAPI Specification<br>
**Supporting Resources:** Accepted IDR-SRV-006 through IDR-SRV-011 reports; official CSAPI tag `v1.0.0`; tagged modular OAS, JSON Schema, examples, and ATS source; the shared upstream-history register; current official issue state; and pinned connected-systems-go, OpenSensorHub, OS4CSAPI client, and CSAPI Explorer source<br>
**Document Purpose:** Establish a plain-language, implementation-usable representation contract for the Rust Glaux reference server without treating an example OpenAPI file, an unregistered token, an implementation shortcut, or a future Pub/Sub idea as a standards obligation<br>
**Author:** OpenAI Codex, with independent read-only Part 1/Features, Part 2/dynamic-data, and HTTP/interoperability audits<br>
**Accepted By:** Pending Glaux Project Lead review<br>
**Acceptance Date:** Pending<br>
**Date:** August 2, 2026<br>
**Last Updated:** August 2, 2026

---

## Reading Guide

This report uses six evidence labels:

- **N — normative:** an obligation stated by an approved standard or its normatively incorporated artifact;
- **I — inherited:** a normative OGC API or HTTP rule inherited by CSAPI;
- **A — artifact finding:** an observation about official OpenAPI, schema, example, or ATS material that does not override the approved prose;
- **H — history:** issue, pull-request, or commit evidence that explains or tracks a design but does not amend the standard;
- **P — project recommendation:** a bounded Glaux choice that becomes project policy only after plan-owner acceptance; and
- **U — unresolved:** a conflict or omission for which no adopted upstream answer was found.

In this report, a **representation** is the bytes plus their semantics for a resource, a **media type** names the representation format and processing model, a **SWE encoding** maps values to JSON, text, or binary, and an HTTP **content coding** such as gzip transforms an already serialized representation. Those terms are not interchangeable.

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority
4. Extraction Method
5. Standards-Based Inventory
6. Resource-Family Representation Matrix
7. Negotiation and Default Behavior
8. Request-Body Encoding
9. SensorML and SWE Common Implications
10. Error and Validation Implications
11. OpenAPI and Documentation Implications
12. Interoperability and Implementation Lessons
13. Test Strategy
14. Downstream Handoffs
15. Recommendations
16. Risks and Open Questions
17. Plan Validation
18. References
Appendix A. Detailed Question Ledger
Appendix B. Official Artifact Audit
Appendix C. Proposed Decision Register
Appendix D. Completion and Handoff

---

## 1. Executive Summary

Glaux needs one deliberate representation contract, not a pile of route-specific `if` statements. The approved standards provide most of the required media types, but they do not define every default, tie-break, alias, failure response, or cache rule. The official tagged OpenAPI is useful evidence, but it is not safe to copy: it omits bodies for the normative-but-optional SWE Binary class, uses `application/swe+csv` where the approved conformance class says `application/swe+text`, disagrees between item and collection responses, labels a single SystemEvent as SensorML while its collection is JSON, and declares neither `406` nor `415`.

The baseline proposed here is:

1. **Use HTTP `Accept` as the canonical response selector.** Parse media ranges, parameters, specificity, wildcards, and quality values correctly. Missing `Accept` means no preference. `q=0` excludes a representation.
2. **Use one typed representation registry.** For every resource family and operation it records readable and writable media types, aliases, codec, schema, conformance class, default rank, `f` aliases, and documentation/test projections. Runtime behavior, OpenAPI, links, server-derived stream `formats`, and tests must agree with that registry.
3. **Implement the accepted all-class target without overclaiming it.** Part 1 uses `application/geo+json` for System, Deployment, Procedure, and Sampling Feature, and `application/sml+json` for System, Deployment, Procedure, and Property. Part 2 uses `application/json` broadly and the three SWE encodings for Observation/Command-shaped payloads: `application/swe+json`, `application/swe+text`, and `application/swe+binary`. A release claims a class only after every applicable operation passes.
4. **Choose stable, resource-specific defaults.** Common documents and ordinary Part 2 resources default to `application/json`; dual-representation Part 1 features default to GeoJSON; Sampling Feature defaults to GeoJSON; Property defaults to SensorML. Observation, Command, and Feasibility default to ordinary JSON when their parent stream advertises it, otherwise to the first parent-supported format in the fixed global order SWE JSON, SWE Text, then SWE Binary.
5. **Return `406` rather than silently ignoring an explicit unsatisfied `Accept`.** Return `415` for an unsupported request media type or content coding. Do not use `415` for a supported format whose body is malformed or schema-invalid.
6. **Make metadata exact.** The selected media token must appear in `Content-Type`; negotiated responses use the relevant `Vary` fields; validators are representation- and content-coding-specific; JSON is UTF-8 without a BOM or advertised `charset` parameter.
7. **Support a bounded `f` compatibility extension, not arbitrary format guessing.** `f` is explicitly declared on supporting GET operations, uses a closed per-operation vocabulary, overrides `Accept`, and is preserved in representation-specific links and pagination. Invalid or unavailable `f` values return `400`. Short aliases and exact media-type values are compatibility inputs, not new conformance classes.
8. **Do not confuse three different schema choices.** `obsFormat` or `cmdFormat` selects the Observation or Command payload format being described; `Accept` selects the representation of the schema document; `Content-Type` identifies the returned schema document. Part 2 defines JSON wrapper structures, but their HTTP media contract is conditional and they are not automatically JSON Schema.
9. **Treat the SWE media-token conflict as a real upstream defect.** CSAPI Part 2 requires non-vendor `application/swe+*` tokens, while the approved SWE Common 3.0 text requires `application/vnd.ogc.swe+*` for negotiation and links. Glaux should implement the CSAPI tokens for strict conformance and accept the corresponding vendor tokens as explicitly documented aliases over the same codecs. If an alias is selected, return that exact token rather than rewriting it to one the client excluded.
10. **Keep optional formats honest.** HTML resource views, XML, `application/swe+csv`, `application/schema+json`, `text/plain` schema responses, suffix routes, and future Pub/Sub/binary containers are not initial CSAPI obligations. They stay disabled or isolated until their semantics, schemas, and tests exist.

The unresolved items do not prevent planning. They do require visible compatibility decisions and upstream tracking: the SWE canonical/vendor collision, omitted-`cmdFormat` behavior, the scope of inherited Features `application/json` wording for Property, cross-encoding replacement loss, PATCH media types, and several tagged OAS/ATS defects.

---

## 2. Scope and Plan Alignment

### 2.1 Included

This report covers:

- successful response representations for every Part 1 and Part 2 resource family;
- request-body media selection for create, replace, ingestion, command, feasibility, and schema-contract operations;
- `Accept`, `Content-Type`, `Content-Encoding`, `Accept-Encoding`, `Vary`, defaults, aliases, `f`, links, profiles, and schema selectors;
- registered, unregistered, required, optional, unsupported, compatibility, and future media types;
- the tagged v1.0.0 OpenAPI, JSON Schema, examples, and ATS, plus current official issue state;
- OpenAPI, error, validation, fixture, conformance, cache, security, and interoperability handoffs; and
- a concrete Rust-server capability model without selecting crates or writing implementation code.

### 2.2 Excluded

This report does not finalize:

- the Problem Details/error-body model or every `400` versus `422` boundary, owned by IDR-SRV-013;
- the final OpenAPI dialect, generator, or publication layout, owned by IDR-SRV-014;
- SensorML object modeling, SWE component implementation, or complete validation pipelines, owned by IDR-SRV-021 through IDR-SRV-023;
- PATCH and cross-encoding replacement semantics, owned by IDR-SRV-029;
- streaming, Pub/Sub, CloudEvents, AsyncAPI, or protocol bindings, owned by IDR-SRV-035;
- serializer/parser implementation, Rust framework selection, persistence, or performance tuning; or
- research for IDR-SRV-013 or any later indexed topic.

### 2.3 Prerequisite State

The Glaux goal, governance sources, report template, overall plan, and accepted IDR-SRV-006 through IDR-SRV-011 reports were available. The approved standards, tagged source, registries, and current official repository were reachable. No dependency exception was used.

### 2.4 Core-Question Answers

| Core question | Answer | Detail |
|---|---|---|
| CQ1. Required or expected negotiation | Correct HTTP `Accept`/`Content-Type` behavior plus exact claimed CSAPI encoding classes; Glaux adds strict 406, stable defaults, typed aliases, and optional `f` policy. | §§5, 7, 10 |
| CQ2. Required media and encodings | Part 1 GeoJSON/SensorML; Part 2 JSON and SWE JSON/Text/Binary where applicable; Common JSON and conditional OAS/HTML; exact resource map in §6. | §§5–6, 9 |
| CQ3. Defaults and alternates | Stable per-family defaults; typed `self`/`alternate` links; `f`-specific URLs where enabled; no User-Agent guessing. | §7 |
| CQ4. Unsupported, ambiguous, incompatible, or invalid requests | 406 for valid unsatisfied `Accept`; 415 for unsupported request type/coding; 400 for invalid `f` or selectors and standards-specified invalid bodies; server contract inconsistencies fail validation. | §10 |
| CQ5. Downstream implications | Generated OAD/runtime parity, representation-aware validation, golden fixtures, cache isolation, supplemental conformance tests, and explicit later-topic handoffs. | §§11–14 |

---

## 3. Evidence Base and Authority

### 3.1 Primary Evidence

| Source | Version or immutable pin | Authority and use | Limitation |
|---|---|---|---|
| [CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) | OGC 23-001, Version 1.0, approved June 2 and published July 16, 2025 | Controlling Part 1 resource/encoding obligations | Retains a stale preliminary-media note |
| [CSAPI Part 2](https://docs.ogc.org/is/23-002/23-002.html) | OGC 23-002, Version 1.0 | Controlling Part 2 dynamic-resource/encoding obligations | Contains media/example/selector inconsistencies |
| [OGC API Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) | OGC 17-069r4, Version 1.0.1 corrigendum; source tag commit [`4ff19a57`](https://github.com/opengeospatial/ogcapi-features/commit/4ff19a5734578cf1f815d03ab192e8e0dc407e9f) | Inherited HTTP negotiation, encoding-class, link, and status guidance | Format-query mechanisms are informative only |
| [OGC API Common Part 1](https://docs.ogc.org/is/19-072/19-072.html) | OGC 19-072, Version 1.0.0 | Inherited JSON and conditional OAS/HTML behavior | OAS 3.0 class does not settle Glaux's later dialect strategy |
| [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) | OGC 23-000, approved standard | Normative `application/sml+json` dependency and schemas | Published text retains a draft-era registration note |
| [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html) | OGC 24-014, approved standard | Normative JSON/Text/Binary codec and media-advertisement rules | Its vendor media tokens conflict with CSAPI Part 2 |
| [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html) and [RFC 9111](https://www.rfc-editor.org/rfc/rfc9111.html) | Internet Standards Track, June 2022 | HTTP representation, negotiation, status, validator, and cache behavior | HTTP sometimes permits a server choice that Glaux must make deterministically |
| [RFC 6838](https://www.rfc-editor.org/rfc/rfc6838.html) and [RFC 6839](https://www.rfc-editor.org/rfc/rfc6839.html) | Current referenced media-registration and structured-suffix standards | Registration and `+json` interpretation | Registration status does not cancel an explicit OGC requirement |
| [RFC 6906](https://www.rfc-editor.org/rfc/rfc6906.html) and [RFC 8288](https://www.rfc-editor.org/rfc/rfc8288.html) | Profile relation and Web Linking | Profile and link-type semantics | A link `type` is a hint; the response header controls |
| [RFC 7946](https://www.rfc-editor.org/rfc/rfc7946.html) and [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259.html) | GeoJSON and JSON standards | UTF-8, no-BOM, media parameters, GeoJSON semantics | Do not establish SensorML/SWE domain validity |
| [IANA media types](https://www.iana.org/assignments/media-types/media-types.xhtml) and [link relations](https://www.iana.org/assignments/link-relations/link-relations.xhtml) | Live registries checked August 2, 2026 | Registration state and relation identity | Time-sensitive; retrieval date is part of the finding |
| [OpenAPI 3.1.1](https://spec.openapis.org/oas/v3.1.1.html) and [current 3.2.0](https://spec.openapis.org/oas/v3.2.0.html) | Versioned OAS specifications | Request/response `content` maps and media/schema documentation | IDR-SRV-014 still owns Glaux's selected dialect |

### 3.2 Official Artifact and History Evidence

The official `v1.0.0` tag resolves to immutable commit [`8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2). The mutable default branch was checked at [`3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f). The only Part 1/2 API-tree delta is a Part 1 example-link correction; no media, OAS, schema, or ATS inconsistency identified here has been repaired on `master`.

The [shared upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md) was refreshed for this topic. Material entries include [#21](https://github.com/opengeospatial/ogcapi-connected-systems/issues/21), [#22](https://github.com/opengeospatial/ogcapi-connected-systems/issues/22), [#23](https://github.com/opengeospatial/ogcapi-connected-systems/issues/23), [#46](https://github.com/opengeospatial/ogcapi-connected-systems/issues/46), [#103](https://github.com/opengeospatial/ogcapi-connected-systems/issues/103), [#104](https://github.com/opengeospatial/ogcapi-connected-systems/issues/104), [#144](https://github.com/opengeospatial/ogcapi-connected-systems/issues/144), [#166](https://github.com/opengeospatial/ogcapi-connected-systems/issues/166), [#170](https://github.com/opengeospatial/ogcapi-connected-systems/issues/170), [#178](https://github.com/opengeospatial/ogcapi-connected-systems/issues/178), [#181](https://github.com/opengeospatial/ogcapi-connected-systems/issues/181), and future Part 3 issues [#190](https://github.com/opengeospatial/ogcapi-connected-systems/issues/190) and [#195](https://github.com/opengeospatial/ogcapi-connected-systems/issues/195). None supplies an adopted correction to the approved media-token conflict.

### 3.3 Informative Implementation Evidence

| Source | Pin | Useful observation | Authority limit |
|---|---|---|---|
| [connected-systems-go](https://github.com/OS4CSAPI/connected-systems-go/tree/e900da88738cca92872038b703c4ad537fc0c8fd) | `e900da88738cca92872038b703c4ad537fc0c8fd` | Central formatter map is useful; exact-string matching and silent fallback are not | Go precedent, not the Rust or normative design |
| [OpenSensorHub](https://github.com/OS4CSAPI/osh-core/tree/b2badae59aaa78455c5638ad73b452ccdee40207) | `b2badae59aaa78455c5638ad73b452ccdee40207` | Demonstrates `f`/`format` demand; HTML substring matching is too weak | Implementation precedent only |
| [OS4CSAPI client](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/6fdf39669e8b11fe1e9c0c8914278c5f11a3b57f) | `6fdf39669e8b11fe1e9c0c8914278c5f11a3b57f` | Uses typed links and can append an exact media value in `f` | Client behavior is informative |
| [CSAPI Explorer](https://github.com/OS4CSAPI/ogc-csapi-explorer/tree/00f1c188e05738ee03390fd95f09d351e073a9c3) | `00f1c188e05738ee03390fd95f09d351e073a9c3` | Dispatches on response `Content-Type`; recorded a cross-representation data-source incident | Useful interop target, not an oracle |

### 3.4 Evidence Limitations

- No private SWG minutes, unpublished corrigendum, or private maintainer communication was used.
- The public SECD evidence named by the plan was unavailable/404 during this pass; no conclusion depends on it.
- Focused IDR-SRV-014A through IDR-SRV-014G implementation studies have not run, so current implementation observations are deliberately narrow.
- No live server interoperability test, executable CSAPI test suite, Rust crate benchmark, or serializer implementation was run.
- GitHub and IANA state is point-in-time evidence as of August 2, 2026.
- The official OAS and ATS are incomplete and sometimes contradictory. Mechanical review can identify mismatches but cannot manufacture an authoritative disposition.

---

## 4. Content-Negotiation and Media-Type Extraction Method

| Plan phase | Work performed | Output |
|---|---|---|
| 1. Sources/framework | Reconciled governance and accepted reports; pinned approved standards, immutable tag/current commit, registries, RFCs, and supporting implementations; defined N/I/A/H/P/U labels. | Evidence inventory and authority hierarchy |
| 2. Standards extraction | Extracted direct Part 1 and Part 2 media requirements, inherited Common/Features behavior, SensorML/SWE dependencies, HTTP rules, registrations, and conditional classes. | §5 inventory |
| 3. Resource mapping | Mapped every plan-named family, direction, schema, default/alternate state, failure, validation, test, and handoff. | Split thirteen-field matrix in §6 |
| 4. Mechanisms/defaults | Compared `Accept`, `Content-Type`, `f`, suffixes, selectors, profiles, aliases, content codings, cache variation, and deterministic default/tie behavior. | §7 baseline |
| 5. Error/validation/docs/tests | Classified failures; inspected OAS/schema/ATS discrepancies; designed documentation invariants, fixtures, supplemental tests, and downstream boundaries. | §§8–14 |
| 6. Synthesis | Reconciled three independent audits, resolved project-level choices without claiming they are OGC rules, and validated all questions and criteria. | §§15–17 and appendices |

The artifact audit inspected all tagged Part 1 request/response templates, all tagged Part 2 request/response templates, active media declarations, related schema-selector parameters, and relevant ATS blocks. Findings were compared to the numbered approved requirements, not accepted merely because they appeared in YAML.

---

## 5. Standards-Based Media Type and Encoding Inventory

### 5.1 Controlling Requirements

| Area | Required or conditional behavior | Classification |
|---|---|---|
| Common documents | Part 1 imports Common landing-page, conformance, and JSON behavior. Accepted IDR-SRV-009 establishes JSON for the landing page and conformance declaration. | I |
| Part 1 GeoJSON | [Requirements 77–78](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson_mediatype-read) require `application/geo+json` for read and, when CRD applies, write. It covers System, Deployment, Procedure, and Sampling Feature schemas. | N, conditional class |
| Part 1 SensorML | [Requirements 89–90](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_mediatype-read) require `application/sml+json` for read and applicable CRD write. It covers System, Deployment, Procedure, and Property, not Sampling Feature. | N, conditional class |
| Add existing collection members | [Requirement 71](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_add-to-collection) requires `text/uri-list`, one canonical URL or UID per line. | N when operation applies |
| Part 2 ordinary JSON | [Requirements 93–106](https://docs.ogc.org/is/23-002/23-002.html#_req_json_mediatype-read) use `application/json` for DataStream, Observation and schema wrapper, ControlStream, Command and schema wrapper, CommandStatus, CommandResult, SystemEvent, and Command-shaped Feasibility resources. | N, conditional encoding class |
| Part 2 SWE JSON | [Requirements 107–114](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-json_mediatype-read) use `application/swe+json` for Observation and Command-shaped payloads and define JSON wrapper structures that describe encoding-specific schemas; this does not make the wrapper's HTTP response type `application/swe+json`. | N, conditional class |
| Part 2 SWE Text | [Requirements 115–122](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-text_mediatype-read) use `application/swe+text`. SWE Text is delimiter-configurable DSV and is not intrinsically CSV. | N, conditional class |
| Part 2 SWE Binary | [Requirements 123–130](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-binary_mediatype-read) use `application/swe+binary`. | N, conditional class |
| Features HTML | If the HTML class is claimed, every applicable `200` response must support `text/html`; serving only human API documentation does not establish that claim. | I, optional class |
| OAS 3.0 class and YAML convention | If Common OAS 3.0 is claimed, JSON OAS uses `application/vnd.oai.openapi+json;version=3.0` and HTML documentation uses `text/html`. Common/Features also identify `application/vnd.oai.openapi;version=3.0` for a YAML OAS 3.0 variant, but YAML is not required by the class. The accepted Glaux entry-point report requires live machine and human definitions, while IDR-SRV-014 selects the final dialect/media strategy. | I/P, conditional/optional |
| HTTP content coding | `Content-Encoding` identifies transformations such as gzip after serialization and does not change the underlying media type. | I |
| Profiles | RFC 6906 profiles add semantics without changing base-media semantics. CSAPI media types define no request `profile` parameter. Use `rel="profile"` links, not an invented media parameter. | I/P |

### 5.2 Registration and Classification Snapshot

The IANA registry was checked on August 2, 2026.

| Media type | Registry state | Glaux classification |
|---|---|---|
| `application/json` | Registered | Required/inherited where mapped |
| `application/geo+json` | Registered | Required by claimed Part 1 GeoJSON class |
| `text/html`, `text/plain`, `text/csv`, `text/uri-list` | Registered | HTML conditional; URI list required for its operation; plain/CSV only where semantics fit |
| `application/octet-stream` | Registered | Generic binary content, but not a replacement for the exact CSAPI SWE token |
| `application/problem+json` | Registered | Candidate error representation; final policy to IDR-SRV-013 |
| `application/yaml` | Registered | Generic YAML; an optional OAS representation only if IDR-SRV-014 defines the exact dialect and `service-desc` contract |
| `application/sml+json` | Not registered | Exact approved SensorML/CSAPI normative token; implement despite registry gap |
| `application/vnd.ogc.sml+json` | Not registered | Compatibility input alias only unless later policy changes |
| `application/swe+json`, `application/swe+text`, `application/swe+binary` | Not registered | Exact approved CSAPI Part 2 normative tokens |
| `application/vnd.ogc.swe+json`, `...+text`, `...+binary` | Not registered | Exact approved SWE Common negotiation tokens; compatibility aliases in Glaux |
| `application/swe+csv` | Not registered | Artifact/example compatibility token; not a CSAPI conformance encoding |
| `application/schema+json` | Not registered | Optional only for an actual JSON Schema representation |
| `application/vnd.oai.openapi+json`, `application/vnd.oai.openapi` | Not registered | Conditional JSON and conventional YAML OAS tokens; exact version/variants to IDR-SRV-014 |

Registration absence is an interoperability risk, not permission to substitute a different token for an approved SHALL.

### 5.3 Official v1.0.0 Artifact Findings

| Finding | Evidence class | Glaux disposition |
|---|---|---|
| Part 1 response templates accurately distinguish dual GeoJSON/SensorML families, Sampling Feature GeoJSON, Property SensorML, and common JSON. | A, mostly aligned | Use as supporting fixtures, checked against numbered requirements |
| Part 1 generic collection POST additionally permits an `application/json` array of URI strings. | A, unsupported extension | Do not baseline; use normative `text/uri-list` |
| No active Part 1 or Part 2 path declares `f`. | A | Glaux `f` is an explicit documented compatibility extension |
| Part 2 OAS uses `application/swe+csv` instead of normative `application/swe+text`. | A, conflict | Keep as negative/adapter fixture; never claim it as SWE Text |
| Part 2 OAS omits `application/swe+binary` entirely. | A, omission | Include it in the Glaux registry/OAD when the optional SWE Binary class is implemented and claimed |
| Command item omits SWE JSON while its collection declares it; Observation/Command item and collection types differ. | A, contradiction | Generate item/collection matrices from one registry and test parity |
| SystemEvent item says `application/sml+json` while its collection and Requirement 106 say `application/json`. | A, apparent copy error | Use `application/json`; preserve bad artifact as negative fixture |
| Observation/Command schema responses advertise `application/json`, `application/schema+json`, and `text/plain` without defining complete alternate semantics. | A, unresolved | Use `application/json` as the initial Glaux/JSON-class wrapper baseline; enable alternates only with real codecs/schemas |
| Neither OAS contains `406` or `415` response declarations. | A, omission | Add operation-appropriate responses and tests |
| Tagged ATS checks exact normative SWE tokens but contains SWE Text/Binary copy errors and misses negotiation cross-products. | A, incomplete | Run official tests plus Glaux supplemental tests |

---

## 6. Resource-Family Representation Matrix

The matrix is split in two so it remains readable. The row ID joins both halves. “Initial” means part of the recommended first complete standards-aligned implementation; “conditional” means Glaux advertises and claims it only when the associated conformance class and every applicable operation are implemented.

### 6.1 Representation, Direction, Authority, and Default

| ID | Resource family | Representation or encoding | Media type | Direction | Source and classification | Default or alternate status |
|---|---|---|---|---|---|---|
| R01 | Landing page and conformance declaration | OGC API JSON documents | `application/json` | Response | [§5.1 Common documents](#51-controlling-requirements); inherited | Initial default |
| R02 | Collections and collection metadata | OGC API JSON documents | `application/json` | Response | [§5.1](#51-controlling-requirements), Features/Common, and accepted CSAPI mapping; inherited | Initial default |
| R03 | Machine API definition | OpenAPI JSON or YAML document | Exact type advertised by `service-desc`; Common OAS 3.0 uses `application/vnd.oai.openapi+json;version=3.0` for JSON and identifies `application/vnd.oai.openapi;version=3.0` for YAML | Response | [Common §§11–12](https://docs.ogc.org/is/19-072/19-072.html#_openapi_3_0_requirements_class); conditional/optional, with final choice unresolved | Typed-link target; IDR-SRV-014 decides dialect and variants |
| R04 | Human API documentation | HTML documentation | `text/html` | Response | [Common Requirement 14](https://docs.ogc.org/is/19-072/19-072.html#_operation_2); inherited/conditional | `service-doc` target; not a resource-wide HTML claim |
| R05 | System, Deployment, Procedure items and collections | GeoJSON feature/feature collection | `application/geo+json` | Read; write when CRD applies | [Part 1 Requirements 77–78](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson_mediatype-read); normative conditional | Initial default for these dual families |
| R06 | System, Deployment, Procedure items and collections | SensorML JSON | `application/sml+json` | Read; write when CRD applies | [Part 1 Requirements 89–90](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_mediatype-read); normative conditional | Initial alternate |
| R07 | Sampling Feature items and collections | GeoJSON feature/feature collection | `application/geo+json` | Read; write when CRD applies | [Part 1 Requirements 77–78](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson_mediatype-read); normative conditional | Initial sole/default representation |
| R08 | Property items and collections | SensorML JSON | `application/sml+json` | Read; write when CRD applies | [Part 1 Requirements 89–90](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_mediatype-read); normative conditional | Initial sole/default representation |
| R09 | Add existing resources to a collection | URI list | `text/uri-list` | Request | [Part 1 Requirement 71](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_add-to-collection); normative when operation applies | Initial operation-specific request type |
| R10 | DataStream and ControlStream | CSAPI JSON resource | `application/json` | Read/write where corresponding class applies | [Part 2 Requirements 93–106](https://docs.ogc.org/is/23-002/23-002.html#_req_json_mediatype-read); normative conditional | Initial default |
| R11 | Observation, Command, and Command-shaped Feasibility | CSAPI JSON envelope | `application/json` | Read/write where applicable | [Part 2 Requirements 93–106](https://docs.ogc.org/is/23-002/23-002.html#_req_json_mediatype-read); normative conditional | Initial preferred format when parent stream advertises it |
| R12 | Observation, Command, and Command-shaped Feasibility | SWE Common JSON values | `application/swe+json` | Read/write where applicable | [Part 2 Requirements 107–114](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-json_mediatype-read); normative conditional | Initial alternate when parent stream advertises it |
| R13 | Observation, Command, and Command-shaped Feasibility | SWE Common Text values | `application/swe+text` | Read/write where applicable | [Part 2 Requirements 115–122](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-text_mediatype-read); normative conditional | Initial alternate when parent stream advertises it |
| R14 | Observation, Command, and Command-shaped Feasibility | SWE Common Binary values | `application/swe+binary` | Read/write where applicable | [Part 2 Requirements 123–130](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-binary_mediatype-read); normative conditional | Initial alternate when parent stream advertises it |
| R15 | CommandStatus and FeasibilityStatus | CSAPI JSON resource | `application/json` | Response and applicable request | [Part 2 Requirements 93–106](https://docs.ogc.org/is/23-002/23-002.html#_req_json_mediatype-read); normative conditional | Initial sole/default representation |
| R16 | CommandResult and FeasibilityResult | CSAPI JSON outer resource | `application/json` | Response and applicable request | [Part 2 Requirements 93–106](https://docs.ogc.org/is/23-002/23-002.html#_req_json_mediatype-read); normative conditional | Initial sole/default outer representation |
| R17 | SystemEvent | CSAPI JSON resource | `application/json` | Read/write where applicable | [Part 2 Requirements 93–106](https://docs.ogc.org/is/23-002/23-002.html#_req_json_mediatype-read); normative conditional | Initial sole/default representation |
| R18 | ObservationSchema and CommandSchema wrappers | CSAPI schema wrapper | `application/json` | Response | [Part 2 schema operations](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_schema-op) plus the conditional JSON class; Glaux P/N baseline | Initial default when JSON class applies; `obsFormat`/`cmdFormat` selects described payload format |
| R19 | Schema endpoint containing an actual JSON Schema | JSON Schema document | `application/schema+json` | Response | [§5.3 tagged OAS finding](#53-official-v100-artifact-findings); optional/unresolved and unregistered | Disabled initially; enable only with precise semantics |
| R20 | Schema endpoint containing a defined textual schema | Plain text schema | `text/plain` | Response | [§5.3 tagged OAS finding](#53-official-v100-artifact-findings); optional/unresolved | Disabled initially |
| R21 | Applicable `200` resource responses | HTML | `text/html` | Response | [Features Requirement 36](https://docs.ogc.org/is/17-069r4/17-069r4.html#_requirements_class_html); optional conditional | Disabled initially unless every applicable `200` response satisfies the class |
| R22 | SWE JSON/Text/Binary compatibility labels | Same SWE codecs as R12–R14 | `application/vnd.ogc.swe+json`, `...+text`, `...+binary` | Read/write | [§9.2 conflict](#92-swe-common-media-token-conflict) and [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html); normative dependency/Glaux compatibility policy | Transport aliases only; not parent-stream `formats` values or substitutes for CSAPI tokens |
| R23 | Legacy/example delimited payload | CSV-labelled form | `application/swe+csv` | Read/write adapter only | [§5.3 tagged OAS finding](#53-official-v100-artifact-findings); unsupported by CSAPI classes | Disabled initially; optional separate adapter only with precise CSV semantics |
| R24 | XML, suffix routes, arbitrary profiles, streaming/Pub/Sub containers | Various | None selected here | Various | [§2.2 exclusions](#22-excluded) and [§16 risks](#16-risks-constraints-and-open-questions); optional/future/unsupported | Outside initial HTTP baseline |

Feasibility is included with Command-shaped payloads because Part 2 defines a Feasibility resource as a Command on a feasibility channel. Its status and result resources follow the corresponding JSON outer-resource rules.

### 6.2 Schema, Failure, Validation, Test, and Handoff

| ID | Related schema or OAS artifact | Principal error conditions | Validation implication | Minimum test implication | Downstream handoff | Notes or unresolved issue |
|---|---|---|---|---|---|---|
| R01–R02 | Common/Part 1 response schemas and generated OAD | Unsatisfied `Accept` → 406 | Validate selected JSON schema and links | Exact/wildcard/default/406, body and `Content-Type` | 013, 014, 023, 050–053 | Common JSON does not automatically define every CS resource body |
| R03–R04 | Live API-definition endpoints and typed landing-page links | Broken type/link parity is a build/release defect | Validate OAD syntax, media type, and link target | Machine and human endpoint smoke tests | 014 | Serving HTML documentation alone does not claim Features HTML |
| R05–R08 | Tagged Part 1 GeoJSON/SensorML schemas and response templates | 406; unsupported write type → 415; invalid body → 400 under applicable rule | Validate the schema selected by family and media type | Every family × item/collection × read/write type; alternate closure | 013–016, 021, 023, 031, 050–053 | Features wording about “other resources” is ambiguous for Property; explicit CSAPI family map controls |
| R09 | Part 1 add-to-collection operation | Wrong/missing type → 415; invalid URI-list line → 400 | Validate one URI or UID per line and operation semantics | Valid URL, UID, CRLF/LF, malformed line, unsupported type | 013, 016, 031 | Tagged JSON-array extension is not baseline |
| R10 | Part 2 DataStream/ControlStream JSON schemas | 406/415/400 as applicable | On create, validate the submitted singular `schema`; on response, validate server-derived read-only `formats` and additional schemas | Item/collection/OAD/runtime parity | 013, 014, 023, 034, 036 | Canonical `formats` must describe codecs actually available under the symmetric Glaux read/write rule |
| R11 | Tagged ordinary Observation/Command schemas | 406/415; malformed or parent-schema-invalid body → 400 | Validate outer JSON envelope and embedded result/parameters against parent stream logical schema | Valid/invalid envelope and embedded values | 013, 022, 023, 034, 036 | Ordinary JSON and SWE JSON are different wire models |
| R12–R14 | SWE Common schema/codec definitions and tagged media ATS | 406/415; invalid SWE mapping → 400 | Validate against parent logical components and chosen encoding settings | Round-trip/golden tests for JSON, delimiter variants, byte order, byte encoding, truncation | 013, 022, 023, 034–036, 050–053 | Tagged OAS omits Binary and mislabels Text as CSV |
| R15–R17 | Tagged JSON schemas | 406/415/400 | Validate JSON outer model and state/resource rules | Item/collection parity, especially SystemEvent negative fixture | 013, 023, 034, 036 | Tagged SystemEvent item’s SensorML label is an apparent copy error |
| R16 | CommandResult schema and linked-result relation | Incorrect inline value → 400; bad linked media metadata is a contract defect | Ordinary inline `data` uses the parent ControlStream `resultSchema`; a feasibility result uses `feasibilityResultSchema`; linked results have their own type | Ordinary/feasibility inline and linked variants plus media mismatch | 013, 017, 023, 036 | Container type need not equal linked result type; [Part 2 Requirement 105](https://docs.ogc.org/is/23-002/23-002.html#_req_json_commandresult-constraints) controls schema choice |
| R18–R20 | `observationSchema.yaml`, `commandSchema.yaml`, JSON/SWE schema models | Invalid/missing selector → 400; unsatisfied schema-document `Accept` → 406 | Keep payload-format selection separate from document validation | Every selector × available document representation; omitted `cmdFormat` cases | 013, 014, 022, 023 | Initial omitted `cmdFormat` allowed only when the parent has exactly one format |
| R21 | HTML response schemas/templates if implemented | 406 or unavailable extension | Sanitization, equivalence, and link validation | Every applicable `200` response if class claimed | 014, 025, 050–053 | No partial conformance claim |
| R22 | Alias entries generated from same codec/schema as R12–R14 | Alias excluded/unsupported → 406/415 | Same semantic validation; retain exact selected label | Canonical/alias equivalence, exclusions, exact output type | 014, 022, 056 | Upstream resolution still needed |
| R23 | Tagged OAS compatibility fixtures | Unsupported by default → 406/415 | Only validate as SWE Text if delimiter/line rules really match the declared adapter | Negative baseline plus explicit adapter tests if enabled | 014, 022, 056 | Never advertise it as the SWE Text conformance token |
| R24 | Future-topic artifacts | Unsupported → 406/415 or route absent | No initial implementation pipeline | Negative tests preventing accidental advertisement | 021, 022, 029, 035 | Do not pre-design Part 3 here |

Together, §§6.1 and 6.2 provide all thirteen fields required by the plan: family, representation, media type, direction, source, classification, default/alternate state, artifact, errors, validation, tests, handoff, and unresolved notes.

---

## 7. Negotiation Mechanism and Default Behavior Findings

### 7.1 One Registry, One Candidate Set

For a request, Glaux should first build the available representation set from the intersection of:

1. codecs actually compiled and enabled;
2. conformance classes the deployment intends to claim;
3. representations valid for the resource family and HTTP operation;
4. canonical formats advertised by the parent DataStream or ControlStream, where applicable (a compatibility alias maps to its canonical format rather than becoming a second stream format); and
5. representations within the operation-wide media-type union documented by the live OpenAPI description.

The same typed `RepresentationRegistry` should generate or validate runtime routing, response links, server-derived parent-stream `formats`, operation-level OpenAPI `content` maps, conformance evidence, and tests. A representation unavailable to an operation must not enter that operation's OpenAPI union merely because the process contains its codec; an individual stream's `formats` then narrows the union at runtime.

Part 2 supplies one read-only `formats` list per stream, not separate read/write lists. To avoid the unresolved overclaim in official issue #23, Glaux adopts a strict symmetric rule for canonical listed formats: a listed format works for every applicable read operation and, when the applicable create/replace/delete class is claimed, every corresponding write operation. If a format is not writable, Glaux does not list it or claim the affected write/encoding combination until support is complete. Vendor compatibility aliases remain transport-level adapters documented in OpenAPI; they do not enter standard `formats` values or schema-selector enums unless a later extension schema explicitly permits them.

Conceptually, each entry needs: resource family, operation, request/response direction, canonical media type, accepted aliases, codec identifier, schema identifier, conformance class, server preference rank, `f` values, link policy, content-coding eligibility, and test-fixture identity. This is an implementation planning contract, not a commitment to a specific Rust crate.

### 7.2 `Accept` Selection Algorithm

The following behavior is deterministic and consistent with RFC 9110:

1. Combine all valid `Accept` field lines according to HTTP field semantics and parse media ranges; do not split blindly on commas because quoted parameter values can contain commas.
2. If `f` is present on an operation that declares it, apply §7.4 before `Accept`.
3. If `Accept` is absent, treat the client as expressing no preference and choose the stable family default.
4. For each available representation, find the most specific matching range. Exact type/subtype and matching parameters outrank subtype wildcards, which outrank `*/*`.
5. Apply the matching range’s quality value. `q=0` excludes the representation; omitted `q` means 1.0; accept no more than three decimal digits when generating values.
6. Choose the candidate with the highest client quality, then highest matching specificity, then the registry’s stable server preference rank. Use the same final lexical/registry-ID tie-break everywhere so iteration order cannot change behavior.
7. Return the exact selected media type in `Content-Type`.
8. When a syntactically valid explicit header accepts no available representation, return `406 Not Acceptable`; do not silently send a default the client excluded.

HTTP permits a server to disregard an unsatisfied preference in some circumstances. Strict `406` is therefore a Glaux interoperability policy, not a claim that RFC 9110 always mandates it. It is safer for generated clients and exposes configuration drift immediately.

### 7.3 Default Order

| Family | Absent-`Accept` or effectively unconstrained default |
|---|---|
| Landing, conformance, collections metadata | `application/json` |
| System, Deployment, Procedure | `application/geo+json` |
| Sampling Feature | `application/geo+json` |
| Property | `application/sml+json` |
| DataStream, ControlStream, status, result, event | `application/json` |
| Observation, Command, Feasibility | `application/json` if the parent advertises it; otherwise the first available format in a documented global order: SWE JSON, SWE Text, SWE Binary |
| Schema operation | `application/json` wrapper for the format identified by `obsFormat`/`cmdFormat` |
| Machine/human API definition | Follow the typed `service-desc`/`service-doc` link; final machine default belongs to IDR-SRV-014 |

Defaults do not vary by User-Agent, browser detection, request origin, or incidental hash-map order. A stream cannot default to a format it does not advertise.

### 7.4 Bounded `f` Compatibility Extension

OGC API Features discusses `f` only as an informative pattern, and the tagged CSAPI OAS does not declare it. Nevertheless, existing CSAPI clients use format parameters. Glaux should support `f` as an expressly non-conformance compatibility extension with these limits:

- enable it only on documented safe/read operations;
- use a closed per-operation vocabulary;
- allow short aliases `json`, `geojson`, `sml`, `swe-json`, `swe-text`, `swe-binary`, and `html` only where their target exists;
- optionally accept an exact supported media-type string as an adapter value;
- interpret `json` as the route’s documented default JSON-family representation, not as permission to relabel GeoJSON, SensorML, or SWE JSON as `application/json`;
- make `f` override `Accept`, matching the common OGC compatibility convention;
- return `400` for an unknown, duplicate, syntactically invalid, or unavailable value;
- preserve the selected `f` in representation-specific `self`, `alternate`, and pagination links; and
- declare its enum and precedence in the live OpenAPI description.

Extension suffixes such as `.json` are not needed initially. Supporting both suffixes and `f` would multiply canonicalization, routing, link, cache, and signature concerns without a CSAPI requirement.

### 7.5 Links, Profiles, and Schemas

Within the inherited Features scope, `/collections` metadata, feature/resource-list response documents, and items reached through Features collection-item routes have typed `self`/`alternate` requirements, and collection descriptors provide one typed `items` link per available item encoding. CSAPI canonical item endpoints such as `/systems/{id}` are not explicitly placed under every Features item-link clause. Glaux should extend the same complete typed-link behavior to canonical CSAPI items and useful Part 2 alternates as project policy, not report that extension as a universal Part 1 or Part 2 SHALL.

The `type` attribute is a hint about the target representation; the target response’s `Content-Type` remains authoritative. Release validation must detect disagreements. Glaux may also mirror useful links in the HTTP `Link` header, but should not call that recommendation a universal requirement.

No relevant CSAPI media type defines a `profile` parameter. A request such as `Accept: application/json;profile="…"` therefore does not match plain `application/json` unless a future advertised profile contract explicitly defines it. Use a typed `Link: <…>; rel="profile"` or an equivalent payload link for profile identification. An unsupported explicit Accept profile leads to 406; an unsupported project query selector leads to 400.

Schema links use `describedby` or the standard resource-specific schema operation as appropriate. A link’s type must describe the schema document, not the payload format that schema describes.

### 7.6 Content Coding, Caches, and Validators

SWE JSON/Text/Binary are representation encodings selected by media type. Gzip and Brotli are HTTP content codings applied after serialization. They must never be stored in the same enum or reported through a stream’s `formats` list.

The recommended initial coding baseline is `identity` and gzip; Brotli can be added after measurement. Response coding selection follows this deterministic policy:

1. Combine all valid `Accept-Encoding` field lines and parse exact coding names, `*`, and quality values. An absent field means any coding is acceptable under HTTP; Glaux chooses `identity` as its compatibility default. An explicitly empty field also selects no content coding.
2. Filter to codings implemented and safe for the selected representation. A documented size threshold may prefer identity for a small response when both are acceptable, but must not make gzip unavailable when it is the client's only acceptable supported coding.
3. `identity` remains acceptable unless `identity;q=0`, or `*;q=0` without a more specific nonzero `identity`, excludes it. An exact coding match controls that coding before a wildcard; `q=0` excludes it.
4. Rank acceptable codings by client quality and specificity, then a deterministic server policy: prefer gzip above the documented size threshold and identity below it. If identity is excluded, use acceptable gzip regardless of that preference threshold. Do not let iteration order choose.
5. If the client explicitly excludes identity and accepts no eligible supported coding, return 406 under Glaux policy; IDR-SRV-013 defines the error body and malformed-header boundary.
6. Emit `Content-Encoding` only when a coding is actually applied and retain the selected representation's underlying `Content-Type`.

When selection depends on request fields, merge the relevant names into `Vary` rather than overwriting an existing value:

- `Vary: Accept` for a canonical URI with multiple media representations;
- add `Accept-Encoding` when content coding is negotiated; and
- do not add dimensions that did not influence the response.

Strong entity tags must identify the selected bytes and therefore differ across representations and content codings. A weaker implementation may use correctly scoped weak validators, but must never allow a cached SensorML, GeoJSON, or compressed response to satisfy a different variant accidentally. Schema-selector query values already participate in the URI cache key.

`Vary` is not an authorization control. Until IDR-SRV-039/040 proves that a response is identical public content for all callers, authenticated or policy-dependent responses should default to `Cache-Control: private, no-store`. A later policy may permit shared caching only with a non-user-specific representation, an explicit public cache contract, and tests proving that authorization state cannot cross cache entries.

JSON-family responses use UTF-8, no byte-order mark, and no advertised `charset` media parameter. Request compatibility with redundant `charset=utf-8` can be considered later as a narrow adapter, but Glaux must not use content sniffing to infer a missing typed-body `Content-Type`.

---

## 8. Request-Body Encoding Findings

| Operation/family | Accepted request type | Processing contract | Failure boundary |
|---|---|---|---|
| Create/replace System, Deployment, Procedure | `application/geo+json` and/or `application/sml+json` according to claimed classes | Select parser by exact media type, validate corresponding schema, then map to one domain model without silently discarding representation-only information | Unsupported/missing type on non-empty body → 415; malformed/schema-invalid body → 400 where standards require |
| Create/replace Sampling Feature | `application/geo+json` | GeoJSON schema and domain checks | Same |
| Create/replace Property | `application/sml+json` | SensorML Property schema and semantics | Same |
| Add an existing member | `text/uri-list` | One canonical URL or UID per line; define whitespace and line-ending handling | Wrong type → 415; malformed/ambiguous entry → 400 |
| Create/update DataStream or ControlStream | `application/json` | Validate the client-submitted singular `schema`/format contract; the server derives read-only `formats` and any additional schemas from codecs it can actually operate | 415 or 400 as above |
| Ingest Observation | Parent-advertised subset of `application/json`, `application/swe+json`, `application/swe+text`, `application/swe+binary` | Select codec by `Content-Type`; validate values against the parent DataStream’s logical schema and encoding configuration | Unsupported/unadvertised type → 415; invalid encoded values → 400 |
| Submit Command or Feasibility | Parent-advertised subset of the same four | Select codec by `Content-Type`; both ordinary and feasibility Commands validate parameters against the parent ControlStream's same `parametersSchema` | Same |
| Status/result/event writes where implemented | `application/json` | Validate the CSAPI outer resource and applicable lifecycle constraints | 415/400; lifecycle status belongs to later topics |

`Content-Encoding` is processed before the representation parser. An unsupported request coding is a 415 condition distinct from an unsupported media type. A 415 response may advertise acceptable request media types through `Accept` and acceptable codings through `Accept-Encoding`, subject to the error-policy decisions in IDR-SRV-013.

For ordinary JSON Observations and Commands, validating only the outer CSAPI JSON Schema is insufficient. The `result` or `parameters` value must also obey the logical SWE component structure of the parent stream. Conversely, a SWE JSON payload is not simply the ordinary JSON envelope with a different label.

Cross-encoding PUT can lose information that one wire model cannot express. This report does not authorize lossy conversion. It hands replace/PATCH preservation and rejection rules to IDR-SRV-029 and records official issue #166 as unresolved.

---

## 9. SensorML and SWE Common Representation Implications

### 9.1 SensorML

The approved SensorML 3.0 requirement and CSAPI Part 1 Requirements 89–90 use `application/sml+json`. A note retaining the preliminary `application/vnd.ogc.sml+json` form is stale history, and neither form appeared in IANA on the retrieval date. Glaux must advertise and emit the exact non-vendor token for CSAPI conformance. It may accept the vendor form as a documented input alias for legacy clients, but should not present two indistinguishable ordinary alternates.

System, Deployment, and Procedure have both GeoJSON and SensorML representations. Those must expose the same authorized resource identity and state even though SensorML carries richer semantics. Sampling Feature has no Part 1 SensorML representation, and Property has no Part 1 GeoJSON representation. A generic `+json` parser does not make those families interchangeable.

### 9.2 SWE Common Media-Token Conflict

Two approved standards give different exact tokens for the same codec families:

| Codec | CSAPI Part 2 token | SWE Common 3.0 negotiation token |
|---|---|---|
| JSON | `application/swe+json` | `application/vnd.ogc.swe+json` |
| Text | `application/swe+text` | `application/vnd.ogc.swe+text` |
| Binary | `application/swe+binary` | `application/vnd.ogc.swe+binary` |

No adopted corrigendum or issue disposition resolves this collision. Glaux cannot satisfy exact-token tests for both standards by pretending the strings are equal. The least damaging baseline is:

1. implement, advertise, and test the non-vendor CSAPI tokens as the canonical Connected Systems contract;
2. implement the corresponding SWE Common vendor tokens as documented compatibility aliases over the same codecs;
3. include both transport labels in an operation's OpenAPI media union where alias support is enabled, but keep vendor aliases out of standard parent-stream `formats` and schema-selector values;
4. for a response, emit the exact label independently selected through `f`, `Accept`, or the route default; for a request, accept and validate the exact canonical or alias label supplied in that request's `Content-Type`; and
5. track an upstream clarification and revisit link advertisement in IDR-SRV-021/022.

An alias changes the HTTP label, not the logical resource. It therefore does not require duplicate alternate URLs by itself. A client that explicitly excludes the canonical token but accepts the vendor token must receive the vendor token in `Content-Type`, not a silent rewrite.

### 9.3 SWE Text, Binary, and Schema Axes

SWE Text is delimiter-configurable delimited text; it is not necessarily CSV. Its separators are required not to occur inside encoded values, and it defines no CSV quoting or escape mechanism. The tagged `application/swe+csv` examples are not evidence that any SWE Text stream can be called CSV. A future `application/swe+csv` adapter would need its own precise comma, CRLF, quoting, and escape rules and must be documented as a separate extension codec, not as the SWE Text conformance representation.

SWE Binary requires byte order, byte encoding, applicable component/member mappings, and the sizes or counts needed to parse values. Compression/encryption block characteristics are optional rather than universal, and SWE Common does not create a generic transport-framing requirement here. Merely exposing `application/octet-stream` would lose the CSAPI-required semantic media label. Large-body limits and streaming parser mechanics belong to later implementation and security research, but byte-exact golden vectors are already required by this contract.

Schema operations have two independent choices:

- `obsFormat` or `cmdFormat` identifies the payload encoding whose logical schema is requested; and
- HTTP `Accept` identifies how that schema document itself should be represented.

`obsFormat` is required. Although the tagged `cmdFormat` parameter is marked optional, the standard does not define an omitted default while its success condition expects an identified format. Glaux clients and documentation should always send it. For resilience, the server may accept omission only when the parent ControlStream has exactly one available format; otherwise it returns 400 rather than guessing.

When the Part 2 JSON class applies, Glaux uses its standard JSON schema-wrapper structure with `application/json` as the initial project baseline. The schema operation itself does not make that transport choice unconditional for every encoding-class combination. `application/schema+json` is enabled only when the returned document truly is JSON Schema, and `text/plain` only when a particular textual schema language is defined. A `+json` suffix proves JSON syntax, not JSON Schema semantics.

The published SWE array-layout prose remains contradictory after issue #71/PR #118. Exact model and fixture resolution is handed to IDR-SRV-022/023; the representation registry must be able to bind the eventual rule without changing HTTP media selection.

---

## 10. Error/Failure and Validation Implications

| Condition | Recommended HTTP outcome | Basis and boundary |
|---|---|---|
| Valid `Accept` has no supported match | 406 | Glaux strict policy consistent with Features guidance; include available variants through the eventual error contract |
| Malformed `Accept` syntax | Defer exact 400 behavior to IDR-SRV-013 | RFC parsing failure and security limits need one global rule |
| Explicit unsupported profile parameter in `Accept` | 406 | Parameterized range does not match an unparameterized representation |
| Unknown/duplicate/unavailable `f` or schema-format selector | 400 | Project/operation selector is invalid; do not convert to a negotiated default |
| Missing `Content-Type` on a non-empty typed write | 415 | Glaux typed-body policy; do not sniff |
| Unsupported request media type | 415 | RFC 9110; response may advertise accepted request types |
| Unsupported request `Content-Encoding` | 415 | RFC 9110; distinguish coding from media type |
| Explicit `Accept-Encoding` accepts no eligible supported response coding and excludes identity | 406 | Glaux coding policy; exact error body and malformed-header boundary to IDR-SRV-013 |
| Supported type, malformed encoding or invalid schema/domain value | 400 where CSAPI specifies invalid-body behavior | Parser/schema failure, not lack of media support; global 400/422 policy remains with IDR-SRV-013 |
| Requested representation implemented globally but unavailable for this family/operation/stream | 406 response or 415 request | Candidate set is operation-specific |
| Inconsistent `type` link, response `Content-Type`, schema, stream `formats`, or OAD `content` | Build/release validation failure; 500 only if it escapes into runtime | This is a server contract defect, not a client error |
| Lossy cross-encoding replacement | Reject according to IDR-SRV-029 policy | Do not silently discard fields |
| Error response itself is excluded by the failed `Accept` | IDR-SRV-013 decision | The status still matters; exact error-body/no-body behavior must be global |

Authentication and the authorization needed to address a parent resource must occur before parent-specific capability or schema lookup. Only then should Glaux safely decode an accepted content coding, identify the request media type, parse and validate that representation, validate embedded SWE values against the authorized parent's logical schema, and apply the remaining domain/lifecycle rules. Some authentication schemes need the untouched request bytes, and error precedence can itself disclose information, so IDR-SRV-013 and the security topics must finalize the exact pipeline. All response variants remain projections of the same authorized domain state.

The server should cap header length, media ranges, parameter count, nesting, decompressed size, and parser work. Exact limits belong to security/configuration research, but negotiation must not enable algorithmic denial of service or decompression bombs.

---

## 11. OpenAPI and Documentation Implications

The live OpenAPI description must describe the server Glaux actually runs, not reproduce the tagged example wholesale. Every operation needs an exact request/response `content` map derived from the representation registry. That static map varies by family, method, direction, and enabled conformance class and advertises the operation-wide union of media types Glaux may support. An individual DataStream or ControlStream's server-derived `formats` narrows that union at runtime; OpenAPI cannot vary its path operation per stream instance. A global list of every codec known to the process would still be false advertising.

The design must represent:

- every successful response media type and schema;
- every accepted request media type and schema;
- `406`, `415`, and operation-relevant `400` responses, with their final shared error representation from IDR-SRV-013;
- the optional `f` parameter only where supported, including its enum and precedence over `Accept`;
- `obsFormat` and `cmdFormat` as payload-schema selectors rather than response media selectors;
- binary request/response bodies without misusing the OpenAPI Media Type Object’s `encoding` field, which is for multipart/form properties;
- response `Content-Type` through the Response Object's `content` keys (an explicit OpenAPI `Content-Type` Header Object is ignored), plus applicable Header Objects for `Content-Encoding`, `Vary`, validators, and useful accepted-media hints; and
- typed `service-desc`, `service-doc`, `self`, `alternate`, `items`, `profile`, and `describedby` links.

The chosen OpenAPI version and publication media type remain with IDR-SRV-014. The official tagged artifacts use OpenAPI 3.1.0, while an inherited Common conformance class specifies a versioned OAS 3.0 media token. Glaux should record that as a compatibility/design decision, not hide it by emitting an imprecise `application/json` label.

Release validation should compare five projections of the same registry:

| Projection | Required invariant |
|---|---|
| Runtime | Selected request parser and response serializer exist for the advertised operation |
| OpenAPI | Each operation's `content` map is the exact implemented operation-level union and uses matching schemas |
| Hypermedia | Link `type` and target response `Content-Type` agree |
| Dynamic capability | Server-derived canonical DataStream/ControlStream `formats` are a runtime subset of the OpenAPI union and obey the Glaux symmetric read/write rule |
| Conformance/tests | Claimed class and fixtures cover every applicable operation, not a sample endpoint |

Human documentation should explain the family defaults, strict 406/415 behavior, `f` precedence, canonical/compatibility media labels, stream-format constraint, schema-selector axes, and known upstream conflicts in ordinary language. A user should not need to infer the contract by reading handler code.

---

## 12. Interoperability and Existing-Implementation Implications

The implementation scan was deliberately limited and informative:

- `connected-systems-go` demonstrates the value of a central formatter map, but exact string lookup and silent fallback do not implement HTTP quality values, wildcards, or strict 406 behavior. Glaux should reuse the organizational idea, not the parser behavior.
- OpenSensorHub demonstrates real demand for `f`/`format`, but substring-based HTML detection is too permissive and makes parameter handling unpredictable.
- The pinned OS4CSAPI client prefers typed links and otherwise appends `f=<media-type>`. This supports typed alternate links and accepting exact media-type `f` values as a compatibility lane.
- CSAPI Explorer dispatches response parsing from `Content-Type`. Its recorded content-negotiation correction shows a serious invariant: alternate representations must not expose different backing datasets, authorization scopes, or resource identities.

Glaux should therefore be liberal only through explicit, tested adapters. It should not accept misspelled tokens, infer formats from bodies, vary defaults by client brand, or relabel one wire model as another. Compatibility aliases need their own registry entries, documentation, telemetry, and removal/review policy.

Focused implementation studies IDR-SRV-014A through 014G have not run. They should later test this proposed contract against each server/client rather than silently rewriting it from precedent. IDR-SRV-056 should include CSAPI Explorer, OS4CSAPI client, standards-generated clients, curl-style generic HTTP clients, and at least one client that exercises q-values and exact vendor/canonical aliases.

No usable public SECD evidence was available at the planned location. That gap is disclosed rather than filled with an inference.

---

## 13. Test-Strategy Implications

### 13.1 Required Test Layers

| Layer | Purpose | Minimum coverage |
|---|---|---|
| Registry unit tests | Prove deterministic capability and preference data | Duplicate media labels, missing codecs/schemas, invalid defaults, alias cycles, unavailable class/operation combinations |
| HTTP negotiation unit/property tests | Prove parser and ranking behavior | `Accept` and `Accept-Encoding` multiple field lines, exact/range/wildcard matching, parameters, case, q=0, identity exclusion, absent/empty fields, three-digit quality, equal-q ties, quoted values, malformed/oversized input |
| Handler contract tests | Prove operation-specific response and request behavior | Every family × item/collection × method × enabled representation; absent `Accept`, exact type, 406, 415, and correct `Content-Type` |
| Representation schema tests | Prove bytes mean what the label says | GeoJSON, SensorML, ordinary Part 2 JSON, SWE JSON/Text/Binary, URI list, schema wrapper |
| Semantic validation tests | Prove parent schema and domain rules | Observation `result`; ordinary/feasibility Command `parametersSchema`; ordinary `resultSchema` versus `feasibilityResultSchema`; stream `formats` |
| Hypermedia tests | Prove complete discoverability | Typed `self`/`alternate`/`items`, profile/schema links, `f` preservation, target type parity |
| OpenAPI parity tests | Prevent artifact drift | Operation-level OAD union ↔ registry/runtime; stream `formats` subset/symmetry; links; 406/415 declarations |
| Cache/coding tests | Prevent variant contamination | `Accept-Encoding` algorithm, `Vary` merge, identity/gzip selection, representation-specific ETags, conditional requests, private/no-store defaults, decompression limits |
| Official conformance tests | Demonstrate standards evidence | Run the applicable official ATS/ETS without weakening its exact-token checks |
| Supplemental conformance tests | Cover known official gaps | All cross-products, item/collection parity, Binary, Text-not-CSV, selectors, defaults, aliases, malformed/unsupported cases |
| External interoperability tests | Prove useful client behavior | Explorer/client typed-link traversal, exact `Content-Type` dispatch, `f`, canonical/vendor tokens, unsupported representations |

### 13.2 Golden Files and Negative Fixtures

The fixture corpus needs, at minimum:

- semantically equivalent System/Deployment/Procedure resources in GeoJSON and SensorML;
- Sampling Feature GeoJSON and Property SensorML item/collection pairs;
- ordinary JSON and SWE JSON Observation/Command/Feasibility examples bound to the same logical schema;
- SWE Text vectors for multiple delimiters, separator-collision rejection, null/block separators, and non-CSV layouts;
- byte-exact SWE Binary vectors covering byte order, signedness, member mappings, sizes/counts, optional compression/encryption blocks where enabled, and failure truncation;
- schema-wrapper examples for each advertised `obsFormat` and `cmdFormat`;
- `text/uri-list` examples with URL/UID, LF/CRLF, and invalid/ambiguous lines;
- exact canonical and compatibility media-label transactions; and
- negative official-artifact fixtures for `application/swe+csv`, omitted Binary, Command item/collection asymmetry, SystemEvent `application/sml+json`, missing 406/415, and ATS copy errors.

Golden tests must compare both bytes and metadata where stability is promised. Semantic equivalence tests should compare the authorized domain state separately so harmless serialization ordering does not mask or fabricate data differences.

### 13.3 Release Gates

A conformance class is not advertised until every applicable route passes its read/write media matrix. A release fails when any registry entry lacks a codec/schema/test, an OpenAPI operation union includes an unimplemented media type, a server-derived stream `formats` value is outside that union or violates the symmetric read/write rule, a typed link disagrees with the target, or a negotiated response omits the required `Vary`/cache control.

---

## 14. Downstream Topic Handoff Matrix

| Topic | Concrete input from IDR-SRV-012 | Decision retained by that topic |
|---|---|---|
| IDR-SRV-013 | 406/415/400 condition matrix; auth-sensitive failure precedence; unsatisfied media/coding and error-representation questions | Error body, status boundary, malformed-header policy, disclosure and retry details |
| IDR-SRV-014 | Registry-derived operation `content` maps, `f`, typed links, schema selectors, parity gates, OAS artifact defects | OAS dialect/version/media type, generator and publication strategy |
| IDR-SRV-014A–014G | Exact behaviors to measure: q-values, defaults, aliases, `f`, links, schemas, error codes | Evidence from each focused implementation; may refine adapters, not standards obligations |
| IDR-SRV-015/016 | Part 1 family/media map and representation-complete links | Complete resource schemas and endpoint transaction contracts |
| IDR-SRV-017 | `alternate`, `profile`, `describedby`, typed linked-result behavior | Full relationship model and link-generation rules |
| IDR-SRV-021 | SensorML canonical token, legacy alias boundary, GeoJSON/SensorML equivalence | SensorML domain mapping and complete compatibility decision |
| IDR-SRV-022 | SWE codec tokens, vendor aliases, Text/Binary semantics, array-layout defect | Component/codec model and exact array-layout interpretation |
| IDR-SRV-023 | Media-selected schema pipeline, embedded logical validation, schema-document axes | Validator architecture, schema resolution, detailed diagnostics |
| IDR-SRV-024 | Requirement that units and observed-property meaning remain invariant across every media representation | Unit identifiers, semantic bindings, normalization and cross-format equivalence rules |
| IDR-SRV-025 | Conditional HTML scope and representation-equivalence/security needs | Human rendering strategy and sanitization |
| IDR-SRV-029 | Cross-encoding replacement loss and PATCH-media gap | Update semantics and no-data-loss rule |
| IDR-SRV-031 | Part 1 write types and `text/uri-list` | Publisher transaction workflow and atomicity |
| IDR-SRV-034 | Observation/DataStream formats, validation, and representation matrix | Dynamic update/storage/query semantics |
| IDR-SRV-035 | Future/unsupported streaming and Pub/Sub boundary | Protocols, AsyncAPI, event media, framing and backpressure |
| IDR-SRV-036 | Command/Feasibility request formats and result representation | Lifecycle, execution, status and result semantics |
| IDR-SRV-039/040 | Authentication-before-parent-schema lookup and conservative cache treatment for protected variants | Exact authorization/error precedence and conditions, if any, for public/shared caching |
| IDR-SRV-050/051 | Exact requirement/media/test map and official ATS gaps | Harness and requirement-to-test traceability |
| IDR-SRV-053 | Golden/negative fixture inventory | Corpus storage, provenance, generation and maintenance |
| IDR-SRV-056 | External-client scenarios and compatibility aliases | Final interoperability matrix and pass criteria |

---

## 15. Recommendations

Subject to plan-owner acceptance, the implementation plan should adopt these decisions:

1. Build a typed, versioned `RepresentationRegistry`; derive runtime, operation-level OpenAPI unions, links, server-generated canonical `formats`, conformance evidence, and tests from it; and enforce the symmetric read/write rule for every listed dynamic format.
2. Implement the explicit Part 1 family map: GeoJSON for System/Deployment/Procedure/Sampling Feature and SensorML for System/Deployment/Procedure/Property, gated by complete conformance-class support.
3. Implement Part 2 ordinary JSON and all three normative SWE encodings for the applicable Observation/Command/Feasibility families; validate submitted stream `schema`, derive read-only `formats`/additional schemas, and claim each class only after full coverage.
4. Parse `Accept` fully and use strict 406 for an explicit valid unsatisfied preference.
5. Use stable family defaults: common/ordinary resources JSON, dual Part 1 families GeoJSON, and dynamic payloads ordinary JSON when their parent advertises it.
6. Support the bounded documented `f` extension with a closed vocabulary, precedence over `Accept`, 400 on bad values, and link/pagination preservation; do not initially add suffix routes.
7. Require an exact identifying `Content-Type` for non-empty typed writes, reject unsupported media/codings with 415, and never sniff.
8. Keep media type, SWE payload encoding, schema-document representation, and HTTP content coding as separate types and decision axes.
9. Emit UTF-8 JSON without BOM or advertised `charset`; return the exact selected media label; implement deterministic `Accept-Encoding`; use correct `Vary` and representation/coding-specific validators; and default protected variants to `private, no-store` until security/policy research permits otherwise.
10. Implement non-vendor `application/swe+*` as canonical CSAPI tokens and vendor SWE Common tokens as explicit transport aliases over the same codecs, but keep aliases out of standard stream `formats` and schema selectors pending an extension or upstream clarification.
11. Emit `application/sml+json`; limit the stale vendor SensorML token to a documented compatibility input alias if real client evidence justifies it.
12. Keep `application/swe+csv`, XML, schema alternates, partial HTML, arbitrary profiles, suffix routes, and Pub/Sub representations outside the initial baseline unless later research defines and tests them.
13. Require `obsFormat`; document clients to send `cmdFormat`; accept omitted `cmdFormat` only for a single-format parent and otherwise return 400.
14. Meet the inherited typed-link requirements and extend representation-complete links to canonical CSAPI items as explicit Glaux policy; verify link `type`, target `Content-Type`, selected schema, OpenAPI, and runtime parity at release time.
15. Ensure every representation projects the same authorized resource identity and state; compatibility must never change data visibility.
16. Build the cross-product fixture/test program in §13 and retain known official OAS/ATS defects as negative regression fixtures.
17. Track the media-token collision, operation/encoding advertisement, schema-format ambiguity, cross-encoding update loss, and future Part 3 work in the shared upstream-history register.

These recommendations define behavior, not implementation code or crate selection. They are deliberately bounded to the Rust Glaux Server.

---

## 16. Risks, Constraints, and Open Questions

| Item | Effect | Current treatment / owner |
|---|---|---|
| CSAPI versus SWE Common exact-token conflict | A client or conformance test may accept only one spelling | Canonical CSAPI plus exact vendor aliases; track upstream; 021/022 |
| Unregistered OGC/OAS media tokens | Gateways and generic tooling may reject or mishandle them | Use exact normative tokens, document registration state, test clients |
| Broad inherited Features “all other resources JSON” wording | Could be read as inventing a Property `application/json` model | Explicit CSAPI family schemas control; retain interpretation note |
| Optional `cmdFormat` with no defined default | Wrong schema may be returned silently | Require in docs; infer only for a single-format parent; otherwise 400 |
| Tagged OAS and ATS contradictions | Generated server copied from artifacts would be nonconformant/incomplete | Normative overlay, generated registry, negative fixtures |
| One stream `formats` list cannot express different read/write support | A listed format could overstate transactional capability | Glaux requires symmetric canonical support; track #23 and keep aliases out of standard lists |
| `f` is not a CSAPI requirement | Extra surface can drift from HTTP negotiation | Closed explicit extension with parity tests |
| Cross-encoding replacement may be lossy | Silent data corruption | No authorization here; 029 must define reject/preserve behavior |
| Partial HTML support | Accidental false Features HTML claim | Do not claim class until every applicable `200` response passes |
| Large Text/Binary/compressed inputs | Resource exhaustion or decompression attacks | Bounded parsers and size/work limits; detail in security topics |
| Representation-specific authorization drift | Data may leak or disappear depending on format | One authorized domain state before serialization; mandatory equivalence tests |
| Future Part 3/Pub/Sub ideas | Premature complexity and false obligations | Out of IDR012 baseline; 035 owns it |
| No live SECD/focused 014A–G evidence | Some compatibility needs may be missed | Disclose limitation; validate later without rewriting standards facts |

Open implementation decisions that remain legitimate downstream work are the exact Rust libraries, whether Brotli earns inclusion, the exact OAS dialect/media choice, the full error body, alias telemetry/removal policy, HTML scope, schema-alternate languages, and performance thresholds. None prevents this representation contract from informing implementation planning.

---

## 17. Validation Against the Research Plan

### 17.1 Methodology Phases

| Phase | Completion evidence | Result |
|---|---|---|
| 1. Source collection/framework | §3 pins standards, artifacts, registries, implementations, authority labels, and limitations | Complete |
| 2. Standards extraction | §5 extracts direct/inherited requirements and official artifact conflicts | Complete |
| 3. Resource mapping | §6 maps all named resource families using all thirteen required fields | Complete |
| 4. Mechanisms/defaults | §7 specifies `Accept`, `Content-Type`, `f`, defaults, links, profiles, caches, codings, and validators | Complete |
| 5. Errors/validation/docs/tests | §§8–14 provide request, failure, validation, OAD, fixture, conformance, interoperability, and handoff findings | Complete |
| 6. Synthesis | §§15–16 give bounded decisions, risks, constraints, and unresolved ownership | Complete |

### 17.2 Success Criteria

| # | Criterion | Evidence | Result |
|---|---|---|---|
| 1 | Requirements identified with anchors | §§3, 5, 18 | Met |
| 2 | CSAPI and inherited behavior distinguished | Reading Guide; §§5, 7 | Met |
| 3 | SensorML/SWE dependencies identified | §§5, 9 | Met |
| 4 | Resource families mapped | §6 | Met |
| 5 | All representation classes classified | §§5–6 | Met |
| 6 | `Accept`/`Content-Type` documented | §§7–8, 10 | Met |
| 7 | Defaults and alternate links documented | §§6–7 | Met |
| 8 | Error, validation, OAD, fixtures, golden files, conformance, and interop handed off | §§10–14 | Met |
| 9 | Recommendations decision-usable and server-bounded | §15 | Met |
| 10 | References explicit and reproducible | §§3, 18; Appendix B | Met |

### 17.3 Required Deliverable Content

| Required item | Location | Result |
|---|---|---|
| 1. Executive summary | §1 | Present |
| 2. Scope/plan alignment | §2 | Present |
| 3. Evidence/authority | §3 | Present |
| 4. Methodology | §4 | Present |
| 5. Standards inventory | §5 | Present |
| 6. Resource matrix | §6 | Present; all thirteen fields |
| 7. Mechanisms/defaults | §7 | Present |
| 8. Request bodies | §8 | Present |
| 9. SensorML/SWE | §9 | Present |
| 10. Errors/validation | §10 | Present |
| 11. OpenAPI/docs | §11 | Present |
| 12. Interoperability/implementations | §12 | Present |
| 13. Tests | §13 | Present |
| 14. Handoffs | §14 | Present |
| 15. Recommendations | §15 | Present |
| 16. Risks/open questions | §16 | Present |
| 17. Success validation | §17 | Present |
| 18. References | §18 | Present |

All five core questions are answered in §2.4. All 28 detailed questions are individually closed in Appendix A. No later research topic was executed.

---

## 18. References

### Controlling and Inherited Standards

- [OGC API – Connected Systems – Part 1: Feature Resources, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API – Connected Systems – Part 2: Dynamic Data, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC API – Features – Part 1: Core 1.0.1, OGC 17-069r4](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC API – Common – Part 1: Core, OGC 19-072](https://docs.ogc.org/is/19-072/19-072.html)
- [Sensor Model Language 3.0, OGC 23-000](https://docs.ogc.org/is/23-000/23-000.html)
- [SWE Common Data Model Encoding Standard 3.0, OGC 24-014](https://docs.ogc.org/is/24-014/24-014.html)
- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110), [RFC 9111: HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111)
- [RFC 6838: Media Type Specifications](https://www.rfc-editor.org/rfc/rfc6838), [RFC 6839: Structured Syntax Suffixes](https://www.rfc-editor.org/rfc/rfc6839)
- [RFC 6906: Profile Link Relation](https://www.rfc-editor.org/rfc/rfc6906), [RFC 8288: Web Linking](https://www.rfc-editor.org/rfc/rfc8288)
- [RFC 7946: GeoJSON](https://www.rfc-editor.org/rfc/rfc7946), [RFC 8259: JSON](https://www.rfc-editor.org/rfc/rfc8259)
- [IANA Media Types](https://www.iana.org/assignments/media-types/media-types.xhtml), [IANA Link Relations](https://www.iana.org/assignments/link-relations/link-relations.xhtml), retrieved August 2, 2026
- [OpenAPI Specification 3.1.1](https://spec.openapis.org/oas/v3.1.1.html) and [3.2.0](https://spec.openapis.org/oas/v3.2.0.html)

### Reproducible Official Artifacts and History

- [CSAPI `v1.0.0` source at commit `8e03b236`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api)
- [Part 1 modular OpenAPI](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/openapi-connectedsystems-1.yaml) and [ATS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/annex-abstract-test-suite.adoc)
- [Part 2 modular OpenAPI](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/openapi-connectedsystems-2.yaml) and [ATS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc)
- [Current official snapshot `3fd86c73`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f)
- [Official issues and pull requests](https://github.com/opengeospatial/ogcapi-connected-systems/issues), with bounded relevant entries recorded in the [shared history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)

### Project and Supporting Evidence

- [Glaux Server goal](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Overall IDR plan](../IDR%20Plans/overall-idr-research-plan.md), [research planning approach](../../../../../Governance/research-planning-approach.md), and [research report template](../../../../../Governance/research-report-template.md)
- Accepted IDR-SRV-006 through IDR-SRV-011 reports in this directory
- [connected-systems-go pin](https://github.com/OS4CSAPI/connected-systems-go/tree/e900da88738cca92872038b703c4ad537fc0c8fd)
- [OpenSensorHub pin](https://github.com/OS4CSAPI/osh-core/tree/b2badae59aaa78455c5638ad73b452ccdee40207)
- [OS4CSAPI client pin](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/6fdf39669e8b11fe1e9c0c8914278c5f11a3b57f)
- [CSAPI Explorer pin](https://github.com/OS4CSAPI/ogc-csapi-explorer/tree/00f1c188e05738ee03390fd95f09d351e073a9c3)

---

## Appendix A. Detailed Research-Question Ledger

| ID | Plan question, shortened | Answer | Report location | Status |
|---|---|---|---|---|
| S1 | Part 1/2 media requirements | Exact GeoJSON, SensorML, JSON, SWE JSON/Text/Binary and URI-list obligations extracted | §§5–6 | Complete |
| S2 | Inherited Features/Common behavior | HTTP negotiation, conditional JSON/HTML/OAS and link behavior distinguished | §§5, 7 | Complete |
| S3 | OpenAPI/schema behavior | Complete tagged audit and authority conflicts recorded | §§5.3, 11; App. B | Complete |
| S4 | SensorML/SWE representations | Canonical tokens, codecs, dependency and conflict defined | §9 | Complete |
| S5 | Registered/conventional media types | IANA snapshot and exact classification recorded | §5.2 | Complete |
| R1 | Representation for every named resource family | All plan-named entry, feature, dynamic, status, event, command, feasibility, SML/SWE families mapped | §6 | Complete |
| R2 | Families needing multiple representations | Dual Part 1 and dynamic payload families identified and gated | §6.1 | Complete |
| R3 | Single authoritative representations | Sampling, Property, metadata, streams, statuses/results/events and initial schema wrappers identified | §6.1 | Complete |
| R4 | Schema/profile/alternate/documentation links | Typed link rules and link/media parity defined | §7.5 | Complete |
| N1 | `Accept`, wildcards and q-values | Deterministic eight-step algorithm | §7.2 | Complete |
| N2 | Request `Content-Type` | Exact parsing, no sniffing, 415 boundary and family matrix | §§8, 10 | Complete |
| N3 | Query/suffix alternatives | Bounded `f`; suffixes deferred | §7.4 | Complete |
| N4 | Defaults | Stable per-family table | §7.3 | Complete |
| N5 | Alternate links | Typed, representation-complete link contract | §7.5 | Complete |
| N6 | Cache and `Vary` | Dimensions, validators and content-coding separation | §7.6 | Complete |
| B1 | Create/update/command/feasibility/ingestion bodies | Operation request matrix | §8 | Complete |
| B2 | Media-dependent validation | Parser/schema/logical/domain order | §§8, 10 | Complete |
| B3 | Unsupported request encodings | 415 for media/coding; advertising handoff | §10 | Complete |
| B4 | Command payload and SWE parameter definition | Parent ControlStream schema and encoding binding | §§8, 9.3 | Complete |
| E1 | Error status/response cases | Condition-to-status matrix with explicit IDR013 boundaries | §10 | Complete |
| E2 | OpenAPI needs | Operation content, responses, parameters, links, headers and parity | §11 | Complete |
| E3 | Representation-specific schema validation | Outer, embedded logical, codec and selector rules | §§8–10 | Complete |
| E4 | Golden files/fixtures | Positive, byte-exact and negative corpus | §13.2 | Complete |
| E5 | Negotiation/link/default/profile tests | Multi-layer cross-product and release gates | §§13.1, 13.3 | Complete |
| I1 | Existing implementation behavior | Four pinned sources compared without treating precedent as authority | §12 | Complete |
| I2 | Smoke/interop findings | Explorer incident and missing SECD/live-test limitation recorded | §§3.4, 12 | Complete |
| I3 | Explorer/external-client needs | Exact Content-Type, typed links, `f`, aliases and equivalent state | §12 | Complete |
| I4 | Handoffs to 014A–G and 056 | Concrete measurement and client-matrix inputs | §§12, 14 | Complete |

---

## Appendix B. Official OpenAPI, Schema, and ATS Audit

### B.1 Tagged OpenAPI Media Inventory

| Artifact family | Requests declared | Responses declared | Finding |
|---|---|---|---|
| Part 1 System/Deployment/Procedure | GeoJSON and SensorML where applicable | GeoJSON and SensorML | Broadly aligned with explicit family schemas |
| Part 1 Sampling Feature | GeoJSON | GeoJSON | Aligned |
| Part 1 Property | SensorML | SensorML | Aligned; no generic Property JSON model |
| Part 1 add existing | `text/uri-list` plus nonnormative JSON URI array in generic template | N/A | JSON array is unsupported extension, not baseline |
| Observation item | JSON, SWE JSON, SWE CSV | JSON, SWE JSON, SWE CSV | Text mislabeled; Binary absent |
| Observation collection/array | JSON | JSON and SWE JSON | Item/collection asymmetry |
| Command item | JSON and SWE CSV | JSON and SWE CSV | SWE JSON absent; Text mislabeled; Binary absent |
| Command collection/array | JSON | JSON and SWE JSON | Item/collection asymmetry |
| ObservationSchema/CommandSchema | JSON requests where applicable | JSON, `application/schema+json`, `text/plain` | Alternate schema-document semantics incomplete |
| SystemEvent | JSON request | Item SensorML; collection JSON | Item label conflicts with Requirement 106 and appears copied |
| Other Part 2 resources | JSON | JSON | Generally consistent with JSON class |

The tagged active paths declare no `f`, no `application/swe+binary`, and no 406 or 415 response. The relevant Part 1/2 media artifacts were unchanged on the checked `master` snapshot.

### B.2 Schema Findings

- Observation/Command ordinary JSON schemas define CSAPI envelopes; their embedded result/parameter values still depend on parent logical schemas.
- SWE schema models enumerate more formats than the OpenAPI transport declarations, including Binary and `swe+csv` forms.
- `obsFormat` is required. `cmdFormat` is marked optional without an omitted-value default.
- A schema wrapper serialized as JSON is not thereby an `application/schema+json` JSON Schema.
- CommandResult outer JSON and linked/inline result payload semantics must remain distinct; ordinary inline data uses `resultSchema`, while feasibility-result inline data uses `feasibilityResultSchema`.

### B.3 ATS Findings

- Part 1 read tests emphasize OpenAPI advertisement and do not exercise every route/negotiation cross-product.
- Some write tests prove a media type on only one canonical endpoint, weaker than full applicable-resource coverage.
- Part 2 exact-token tests help conformance but lack defaults, q-values, wildcards, aliases, 406/415, omitted/invalid-selector cases, schema-document `Accept`/`Content-Type` interaction, and item/collection parity.
- The SWE Text test instructs verification of Binary advertisement before requesting Text.
- Binary validation prose incorrectly references SWE Text rules.
- Schema-operation tests do not adequately separate payload-format selectors from schema-document negotiation.

Official tests remain useful and must run; supplemental Glaux tests close these gaps without pretending to amend the ATS.

---

## Appendix C. Proposed Decision Register

Every entry remains pending until the Glaux Project Lead accepts this report.

| Decision | Proposed disposition | Acceptance state |
|---|---|---|
| P-012-01 | One typed representation registry is the source of truth; dynamic canonical `formats` are server-derived and obey symmetric read/write support | Pending |
| P-012-02 | Use the explicit Part 1 family/media map and gate claims by complete operation coverage | Pending |
| P-012-03 | Implement full RFC-aware `Accept` ranking and strict 406 | Pending |
| P-012-04 | Use stable resource-family defaults from §7.3 | Pending |
| P-012-05 | Support bounded `f` with precedence, closed vocabulary and 400 behavior | Pending |
| P-012-06 | Emit exact media labels; UTF-8 JSON without BOM/charset; never sniff typed bodies | Pending |
| P-012-07 | Use 415 for unsupported request media/coding and representation-specific validation pipelines | Pending |
| P-012-08 | Keep schema payload selector and schema-document negotiation separate; constrain omitted `cmdFormat` | Pending |
| P-012-09 | Canonical CSAPI SWE tokens plus exact SWE Common transport aliases, with aliases excluded from standard stream `formats`/selectors | Pending |
| P-012-10 | Keep SWE CSV, XML, suffixes, arbitrary profiles and future protocols outside baseline | Pending |
| P-012-11 | Provide typed complete alternates and profile/schema link semantics | Pending |
| P-012-12 | Use deterministic `Accept-Encoding`, correct `Vary`, coding separation, variant-specific validators and conservative protected-response cache controls | Pending |
| P-012-13 | Do not claim partial Features HTML conformance | Pending |
| P-012-14 | Generate/validate operation-level OpenAPI unions and runtime stream-capability subsets from the registry | Pending |
| P-012-15 | Require all variants to expose the same authorized resource identity/state | Pending |
| P-012-16 | Build the cross-product and golden/negative fixture program | Pending |
| P-012-17 | Maintain the upstream compatibility/defect ledger and downstream handoffs | Pending |

---

## Appendix D. Completion and Handoff

### D.1 Completion Checklist

- [x] Exactly one research topic, IDR-SRV-012, was executed.
- [x] All five core and 28 detailed questions were answered.
- [x] All six phases and ten success criteria were validated.
- [x] All eighteen required content areas and thirteen representation-matrix fields are present.
- [x] Normative, inherited, artifact, history, project, and unresolved findings are distinguished.
- [x] Evidence limitations and upstream conflicts are recorded honestly.
- [x] The report was internally reviewed against its controlling plan.
- [ ] Plan-owner acceptance of IDR-SRV-012 — pending Glaux Project Lead review.

### D.2 Required Next Transition

The next two workflow actions are:

1. the Glaux Project Lead accepts or returns IDR-SRV-012; and
2. after acceptance, execute exactly one next research topic: IDR-SRV-013, Error Model, HTTP Status Codes, and Failure Semantics.

Those actions can be authorized together with: **“Approve IDR-SRV-012 and execute exactly one Glaux Server research plan: IDR-SRV-013.”** No IDR-SRV-013 research was begun in producing this report.

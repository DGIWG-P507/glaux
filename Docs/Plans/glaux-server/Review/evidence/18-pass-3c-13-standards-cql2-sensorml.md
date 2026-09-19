# Pass 3c, iteration 13 — standards batch: Features Part 3/CQL2 identifiers and SensorML 3.0 class identifiers

**Date:** 2026-09-19
**Provider/model:** GitHub Copilot; model self-reported as Claude Fable 5.1
**Batch:** `standards-features-cql2`, `standards-sensorml-classes` (from `current_work.next_batch` at planning commit cf1ae13dda5bc230064286bb72e45d6defbb249b)
**Mode:** read-only review of planning documents, published standards, pinned schema artifacts and Roadmap leaves; review artifacts updated and published; no implementation, Goal/Guide/Roadmap, issue, settings or upstream changes.

## 1. Sources consulted this batch

| Source | Access | Used for |
|---|---|---|
| OGC API - Features - Part 3: Filtering, OGC 19-079r2 v1.0 (https://docs.ogc.org/is/19-079r2/19-079r2.html, Publication Date 2024-07-26) | Full document retrieved; Clause 2, 5.3, 6, 7, 8 (8.1–8.8), 9, Annex A (A.1–A.4) read | `standards-features-cql2` |
| Common Query Language (CQL2), OGC 21-065r2 v1.0.0 (https://docs.ogc.org/is/21-065r2/21-065r2.html, Publication Date 2024-07-26) | Full document retrieved; Clause 2, 5.1, 6 (6.1–6.7), 7.5, 7.6, 8.3, Annex A (A.2, A.3, A.7, A.8), Annex B (spatial BNF), Annex C.1/C.2 read | `standards-features-cql2` |
| Published CQL2 JSON schema https://schemas.opengis.net/cql2/1.0/cql2.json (Guide reference `[CQLSchema]`) | Retrieved 2026-09-19 (live registry, no commit pin) | GeometryCollection `minItems` |
| OGC SensorML 3.0, OGC 23-000 (https://docs.ogc.org/is/23-000/23-000.html, Publication Date 2025-07-16) | Full document retrieved; Clause 2, 6.1, 8.2–8.10 class headers, 9.1–9.7, Annex A.3 read | `standards-sensorml-classes` |
| Guide v1.3: Section 1.2 lines 246–274 (pins and class list, line 255), Section 6.3.1 lines 770–823 (queryables, `filter-lang`, operators, limits), Section 7.1 table lines 866–893, Section 7.3–7.4 lines 905–930, Section 13 rows lines 1161–1162, references lines 1216–1228 | Read | Claims under review |
| Roadmap v1.18: group 2.2 (lines 206–227), 7.2.6 (#228), 7.3.1–7.3.6 (#229–#234 by position; #229–#232 confirmed in text), Phase 9 reverification leaves | Read (issue bodies not fetched this batch) | Owning work |

## 2. Question 1 — `standards-features-cql2`: identifiers, dependencies, `cql2-json`-only default, and GeometryCollection `minItems`

### Result: **Supported.** No correction to the Guide is required. Several test-evidence implications are recorded below; they are recommendations, not new findings.

### 2.1 Conformance class identifiers (Guide Section 7.4 table)

| Guide entry | Published identifier | Source |
|---|---|---|
| `http://www.opengis.net/spec/ogcapi-features-3/1.0/conf/queryables` | identical | 19-079r2 Clause 2 conformance table; Annex A.1 |
| same prefix, `filter` | `.../conf/filter` | 19-079r2 Clause 2; Annex A.3 |
| `http://www.opengis.net/spec/cql2/1.0/conf/basic-cql2` | identical | 21-065r2 Clause 2; Annex A.3 |
| same prefix, `basic-spatial-functions` | identical | 21-065r2 Clause 2; Annex A.7 |
| same prefix, `basic-spatial-functions-plus` | identical (class title "Basic Spatial Functions with additional Spatial Literals") | 21-065r2 Clause 2; Annex A.8 |
| same prefix, `cql2-json` | identical | 21-065r2 Clause 2; Annex A.2 |

Not selected by the Guide and confirmed to exist separately: Features Part 3 `queryables-query-parameters` and `features-filter` (the latter binds filtering to `/collections/{collectionId}/items` and GeoJSON; the Guide correctly excludes it because Glaux filters observation routes, not Features collections). CQL2 `cql2-text` and the remaining enhancement classes are not selected, as the Guide states.

### 2.2 Dependencies (Guide Section 7.4 prose)

- Filter requirements class: Dependency "Requirements Class Queryables" (19-079r2 Clause 8.1). Queryables conformance class: Dependency "OGC API - Common - Part 1: Core, Conformance Class JSON", with the note that JSON depends on Core and Landing Page (Annex A.1). Guide statement confirmed.
- Basic CQL2 (21-065r2 Clause 6.1): dependencies SFA Part 1 Architecture, W3C/OGC Time Ontology in OWL, RFC 3339, JSON Schema, Unicode. Guide statement confirmed verbatim in substance.
- Basic Spatial Functions plus (Clause 7.6): Dependency "Basic Spatial Functions" (and SFA Architecture); Basic Spatial Functions (Clause 7.5): Dependency "Basic CQL2". Guide "Spatial-plus depends on basic-spatial and Basic CQL2" confirmed (transitively).
- CQL2 JSON (Clause 8.3): Dependency "Basic CQL2"; Conditional Dependency "GeoJSON, Geometry Objects" and each enhancement class. Guide statement confirmed.

### 2.3 `cql2-json` as the only advertised language and the default (Guide Section 6.3.1 line 799)

19-079r2 Requirement 6 `/req/filter/filter-lang-param`: A gives an OpenAPI fragment with `enum: ['cql2-text','cql2-json']` and `default: 'cql2-text'`; **B** "The enum array in the schema of filter-lang SHALL list the filter encodings that the server supports for the resource"; **C** "The default value in the schema of filter-lang SHALL identify the filter encoding that the server will assume, if a filter is provided, but no filter-lang." Clause 8.7: "support for this filter expression language is not mandatory". Recommendations 4/5 (SHOULD support text/JSON) belong to the unselected Features Filter class. Therefore a server that lists only `cql2-json` and declares it the default is conformant with the Filter class. **Supported.**

Annex A.3 makes the language a test input ("The name of the filter language to test ({filter-lang}; default: cql2-text)" plus "a flag that indicates whether the filter language is the default filter language"), and CQL2 Annex A states that test expressions "have to be instantiated in an executable test as CQL2 Text or CQL2 JSON" depending on the Filter Language parameter. Running the suites with JSON expressions is therefore the standard's own procedure, not an adaptation.

### 2.4 Other Filter/Queryables requirements against Guide Section 6.3.1

| Requirement | Guide position | Result |
|---|---|---|
| Req 1 `/req/queryables/queryables-link`: relation `http://www.opengis.net/def/rel/ogc/1.0/queryables` (CURIE `[ogc-rel:queryables]` allowed as alternative) | Line 774 provides the absolute URI relation and an HTTP `Link` header (recommended in prose; used by A.1.2) | Consistent; no interaction with F-11's `ogc-rel:` question because the absolute form is used |
| Req 3 B–F: `$schema` 2020-12, `$id` = resource URI without query, `type: object`, `type` on non-spatial properties, spatial properties without `type`/`$ref` and with `format: geometry-…` | Lines 774–795, example uses `format: "geometry-any"` and `$id` rule | Consistent |
| Req 3 G: `additionalProperties: false` → unknown property reference yields 400 | Lines 774, 795, 799 | Consistent; A.3.4 expects "unsuccessful execution" for `this_is_not_a_queryable` |
| Req 7 `/req/filter/filter-crs-wgs84`: no `filter-crs` → CRS84 or CRS84h | Line 799 "Accept the standard CRS84/CRS84h filter-coordinate behavior" | Consistent |
| Req 8 `/req/filter/filter-crs-param` (Condition: server supports additional CRSs); 8C error for unsupported CRS | Line 799 "reject other requested CRSs in this initial binding" | Consistent; 8A is conditional and not triggered |
| Req 9 `/req/filter/mixing-expressions`: AND with other predicates | Line 823 | Consistent |
| Req 12 `/req/filter/response`: TRUE includes, FALSE excludes (NULL is not TRUE) | Line 801 "only TRUE selects"; CQL2 Clause 6.2 truth table | Consistent |

### 2.5 GeometryCollection `minItems` (Guide Section 13 row line 1162)

- 21-065r2 Annex C.1 JSON Schema and the published `cql2.json`: `geometrycollection.geometries` has `"minItems": 2` and items limited to point, linestring, polygon, multipoint, multilinestring, multipolygon. Annex C.2 (OpenAPI 3.0 schema) is identical.
- Clause 7.6.1 prose: "GeometryCollection: a collection of **one or more** of Point, Polygon, MultiPoint, MultiLineString, or MultiPolygon instances". Annex B BNF: `geometryCollectionText = "(" geometryLiteral {"," geometryLiteral} ")"` — one or more.
- Result: the Guide row's statement is **confirmed** (schema minimum two; prose and BNF permit one). The Guide's selected behaviour (accept a singleton under spatial-plus semantics; keep the upstream schema; test the adaptation explicitly) is defensible from the prose and BNF. Two qualifications: (a) Requirement 33/38 only oblige a server to accept expressions that validate against the JSON Schema, so a strict tester using the schema as its validity oracle could treat a singleton collection as invalid; (b) the ATS never exercises a singleton (A.8.2 uses a two-member `GEOMETRYCOLLECTION`), so Glaux's acceptance is a Glaux adaptation that no official test confirms. Both are already covered by the Guide's "test the explicit adaptation" and leaf 7.3.3 (#231) "singleton GeometryCollection ... expected horizontal results"; no correction.
- Editorial seam in the standard: the Clause 7.6.1 member-type list omits LineString, while the BNF `geometryLiteral` and the JSON schema include it. No Glaux action; recorded for the interpretation register.

### 2.6 Test-evidence implications (recommendations for owners 7.3.1–7.3.4 / #229–#232, 7.2.6 / #228, and Phase 9 reverification)

1. **Ordering comparisons on Boolean and String queryables must evaluate.** CQL2 A.3.2 `/conf/basic-cql2/comparison` evaluates `=`, `<>`, `>`, `<`, `>=`, `<=` for every queryable of type String, Boolean, Number, Integer, Timestamp or Date and asserts successful execution. A Glaux Boolean or Category/Text `result.<name>` queryable therefore has to accept `>`/`<` comparisons with a same-typed literal (Requirement 3 D), not reject them as type errors. The Guide's operator list (line 801) has no type restriction and "no implicit casts" is unaffected, but fixtures for #230 should include these cases.
2. **Invalid coordinates must be rejected.** A.7.1 and A.8.1 expect "unsuccessful execution" for `POINT(90 180)` (invalid coordinate). Guide line 801 already requires coordinate checks beyond schema pattern matching; fixtures for #231 should include out-of-range coordinates with an expected `400`.
3. **`isNull` on the geometry queryable** (A.3.3 runs `IS NULL` / `IS NOT NULL` for each queryable) and **root Boolean literals** (A.3.4) are already in the Guide (line 801, line 982).
4. **Dataset-conditional abstract tests are not applicable to an observation server.** CQL2 A.3.5, A.3.6, A.7.2, A.8.2 ("Test predicates against the test dataset") are conditional on hosting the Natural Earth feature collections; Glaux filters observation routes and cannot host them as feature collections. Conformance evidence should label them "not applicable (conditional on the test dataset)" rather than pass, and rely on the basic tests plus Glaux fixtures. This is the same labelling discipline recorded under F-08; recorded there as a related instance.
5. **Features Part 3 Annex A defaults are Text expressions** (`{queryable} IS NULL`, `S_INTERSECTS(...,BBOX(...))`); the JSON equivalents must be supplied as test inputs. Permitted by the ATS input parameters; no adaptation label needed.

## 3. Question 2 — `standards-sensorml-classes`: the four class identifiers in Guide Section 1.2 line 255

### Result: **Supported.** All four identifiers exist in OGC 23-000 with the expected form; one optional clarification is recorded.

| Guide class | 23-000 requirements class | Conformance class | Prerequisites (published) | Requirement |
|---|---|---|---|---|
| `json-simple-process` | `/req/json-simple-process` (Clause 9.2) | `/conf/json-simple-process` (A.12) | `/req/json-core`; indirect `/req/model/simpleProcess` | Req 47 schema-valid against `SimpleProcess.json` |
| `json-physical-system` | `/req/json-physical-system` (Clause 9.5) | `/conf/json-physical-system` (A.15) | `/req/json-aggregate-process` **and** `/req/json-physical-component`; indirect `/req/model/physicalSystem` | Req 50 schema-valid against `PhysicalSystem.json` |
| `json-deployment` | `/req/json-deployment` (Clause 9.6) | `/conf/json-deployment` (A.16) | `/req/json-core`; indirect `/req/model-deployment` (sic) | Req 51 schema-valid against `Deployment.json` |
| `json-derived-property` | `/req/json-derived-property` (Clause 9.7) | `/conf/json-derived-property` (A.17) | `/req/json-core`; indirect `/req/uml-derived-property` (sic) | Req 52 schema-valid against `DerivedProperty.json` |

Base URI: `http://www.opengis.net/spec/sensorML/3.0` (Clause 6.1) — the register's `sensorML/3.0/req/...` spelling is correct.

Applicable dependencies pulled in by the four classes:

- `/req/json-core` (Clause 9.1; `/conf/json-core`, A.11): Requirement 46 `/req/json-core/media-type` — documents advertised as `application/sml+json`. Guide line 737 uses exactly this media type. Its own prerequisite is SWE Common 3.0 `/req/json-block-components`, which the Guide implements in full.
- `/req/json-aggregate-process` (A.13, Req 48 against `AggregateProcess.json`) and `/req/json-physical-component` (A.14, Req 49 against `PhysicalComponent.json`) are hard prerequisites of `json-physical-system`. The Guide's phrase "including their applicable dependencies and CSAPI mappings" (line 255) covers them implicitly; Roadmap 2.2.3 ("PhysicalSystem and applicable component/position descriptions") and 2.2.9 (input/output/connection bindings) implement the content. **Optional clarification:** name `json-core`, `json-aggregate-process` and `json-physical-component` explicitly in Section 1.2 or the Section 7 evidence list so the claim manifest cannot omit A.11, A.13 and A.14 when declaring the four selected classes.
- Every JSON conformance class (A.11–A.17, Annex A.3; abstract tests A.46–A.52 for Requirements 46–52) is a media-type check or schema validation against the published `schemas.opengis.net/sensorML/3.0/json/` files, and Annex A.3 states these tests "shall also be used to check conformance of software implementations that output these JSON documents", so the pinned, unmodified upstream schemas the Guide already requires (line 189 "Pinned local standard/schema artifacts"; line 410 "Retain unmodified upstream schemas") are the test oracle for these classes; the UML model classes (A.2–A.10, Annex A.2) are inspection tests.

Artifact seam (no action): 23-000 Clause 9.1.2.1 still contains the NOTE "Implementations should use application/vnd.ogc.sml+json as a preliminary media type until this Standard is stable ... This note will be removed before publishing this Standard", although the document is published. Requirement 46 and Glaux use `application/sml+json`.

## 4. Material follow-up questions recorded, not scheduled

1. CQL2 Clause 7.6.1 GeometryCollection member list omits LineString (editorial); BNF and schema include it.
2. Whether the conformance-claim manifest will list the SensorML dependency classes A.11/A.13/A.14 explicitly (see optional clarification above); decide when the Phase 9 claim leaves are reviewed.
3. Whether #230's fixtures include Boolean/String ordering comparisons and #231's include out-of-range coordinates (issue bodies not read this batch).

## 5. Disposition summary for the checkpoint

| Check id | Disposition | Guide change needed | Owning work |
|---|---|---|---|
| `standards-features-cql2` | Supported: all six class URIs, the dependency statements, the `cql2-json`-only default (Requirement 6 B/C) and the GeometryCollection schema/prose conflict verified against the published texts and schema | None required; optional test-evidence notes | 7.2.6 (#228), 7.3.1–7.3.4 (#229–#232), Phase 9 claim leaves |
| `standards-sensorml-classes` | Supported: four identifiers, base URI and conformance classes verified in 23-000 | Optional: name the dependency classes A.11/A.13/A.14 explicitly | 2.2.2–2.2.5, 2.2.9; Phase 9 claim leaves |

Remaining standards checks after this batch: `standards-ats-rows`, `standards-part2-inheritance`, plus `observation-extension-policy` (F-13).

## 6. Statement of limits

- `cql2.json` was read from the live `schemas.opengis.net` registry (no commit pin); the Annex C.1 text in the published standard was used as the second witness and agrees.
- The Features Part 3 and CQL2 abstract tests were read for their stated inputs and methods; no executable test suite was run and no Glaux code exists.
- Owning issue bodies #228–#232 were not fetched this batch; implications above are addressed to their Roadmap text.

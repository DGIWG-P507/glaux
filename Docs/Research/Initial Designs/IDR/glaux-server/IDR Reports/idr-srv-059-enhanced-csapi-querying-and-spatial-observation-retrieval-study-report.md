# Section 059: Enhanced CSAPI Querying and Spatial Observation Retrieval Study - Research Report

**Topic ID:** IDR-SRV-059<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-059 research plan](../IDR%20Plans/idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Q1-Q5; all four planned use cases<br>
**Methodology Used:** Published-standard and tagged-artifact comparison; bounded upstream-history and peer-source inspection; reconciliation with accepted Glaux research; independently reasoned examples with a small fixed-data diagnostic; technical review<br>
**Research Time:** AI-assisted research conducted September 17, 2026, America/New_York (September 18 UTC); no human-effort estimate inferred<br>
**Primary Sources:** Approved CSAPI Parts 1 and 2; Features Part 3; CQL2; relevant SWE Common/SFA material; pinned CS-Go and OSH source and open OSH filtering proposal<br>
**Supporting Resources:** Accepted IDR-SRV-011, 017/018, 022/024-028, 034, 039/040, 050/051/053/054/056, and 058; final synthesis; Goal v1.6; draft Guide v0.1<br>
**Document Purpose:** Recommend whether and how to add bounded filtering capabilities, especially spatial observation retrieval through sampling relationships, without treating that recommendation as adopted server scope<br>
**Author:** OpenAI Codex, with independent standards, implementation, and semantic/security reviews<br>
**Accepted By:** Pending Glaux Project Lead review<br>
**Acceptance Date:** Not yet accepted<br>
**Date:** September 17, 2026<br>
**Last Updated:** September 17, 2026

---

## Usage Rules

This report follows the existing research-report template. **Finding** identifies source-backed evidence; **Interpretation** identifies analysis; **Recommendation** identifies a proposed project choice. Publication does not adopt an extension, change the Goal/Guide, or accept the report. The preceding plan was published in commit `a17a88d`; the user's next `proceed` authorized this research/report iteration only.

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Scope and Plan Alignment](#2-scope-and-plan-alignment)
3. [Evidence Base](#3-evidence-base)
4. [Findings by Research Question](#4-findings-by-research-question)
5. [Decision Analysis](#5-decision-analysis)
6. [Key Recommendations](#6-key-recommendations)
7. [Implementation Implications and Estimates](#7-implementation-implications-and-estimates)
8. [Risks, Constraints, and Open Questions](#8-risks-constraints-and-open-questions)
9. [Validation Against Plan Success Criteria](#9-validation-against-plan-success-criteria)
10. [Next Steps and Handoff](#10-next-steps-and-handoff)
11. [References](#11-references)
12. [Appendix: Checks and Reproducibility](#12-appendix-checks-and-reproducibility)

---

## 1. Executive Summary

**The user's spatial query is a real, useful server capability:** return observations selected by the geometry of their related sampling features, even when the observations do not contain a geometry. The existing CSAPI model supplies the relationships. The missing piece is not necessarily data storage; it is a precise, discoverable way to ask the combined question.

For simple static data, a client can first find sampling features intersecting an area and then request their observations. However, that workflow is not universally equivalent to finding observations whose **direct sampling location** lies in the area. Existing recursive association matching can also include an observation whose larger sampling ancestor intersects the area while its own sampling point is outside. For moving features, using today's position can give a different answer again. The examples in §4.1 make those differences explicit.

**Recommendation for discussion:** add a bounded, optional Features Part 3/CQL2 filtering capability to implementation planning, while preserving the complete existing CSAPI contract. Begin with explicitly named sampling-geometry queryables and typed scalar result queryables on individual datastreams. Use known, explicit geometry with a declared time meaning; do not promise reconstruction of unknown historical positions or Part 4 volumes. Section 4.2 identifies a complete candidate set of filtering classes, initially using CQL2 JSON. This is a proposed addition, not an adopted obligation or implemented feature.

The case for reconsidering IDR-SRV-011's deferral is now concrete: the project has identified spatial observation retrieval, measured-value comparisons, and related-property conditions that merit a combined, standards-aligned interface. Features Part 3 permits searchable properties that are not embedded in the returned observation. It does not, by itself, define which CSAPI relationship or historical geometry supplies those properties. Those mappings must be explicit and verified.

Peer evidence reinforces the need for care. The inspected CS-Go version has named filters, direct sampling-feature ID matching, and text searches, not demonstrated CQL2 support. OSH exposes spatial query inputs and internal filter machinery, but source inspection found differences between the public handler, filter predicate, and inspected storage paths. An open OSH proposal adds simple observation-value equality and expressly leaves CQL2 for future work. None of this establishes a ready-made interoperable solution to copy.

No database replacement, general join language, or new governance system is recommended. The existing Rust/PostgreSQL/PostGIS design remains suitable. The next decision is whether to adopt this bounded addition and its exact interface; the Goal, Guide, original synthesis, and Part 4 adoption decision are unchanged by this report.

---

## 2. Scope and Plan Alignment

This executes **IDR-SRV-059: Enhanced CSAPI Querying and Spatial Observation Retrieval Study**. Spatial observation retrieval is primary; measured-value and related-property filtering are included. A small metadata-resource comparison separates observation-specific needs from general filtering needs.

The study does not implement a parser, server, database migration, benchmark, public query service, or Part 4 geometry engine. It does not extend this work into sorting, aggregation, arbitrary remote traversal, a client application, or a new standards proposal. Illustrative requests and mappings are labeled where they are not existing CSAPI behavior.

### Research Question Coverage Matrix

| Question | Answer location | Coverage |
|---|---|---|
| Q1: need and existing capability | §4.1, four cases and baseline/artifact comparison | Complete as assessment; existing versus additional behavior distinguished |
| Q2: standards-based options | §4.2, candidate classes, discovery and language limits | Complete as assessment; CSAPI-specific names/binding remain proposed |
| Q3: relationships, geometry, time, values | §§4.1 and 4.3 | Complete as assessment; unsupported or unresolved meanings are explicit |
| Q4: implementation and verification | §4.4, §7, Appendix | Complete within source-inspection boundary; no runtime conformance claim |
| Q5: recommendation and planning impact | §4.5, §§5-6 and 8-10 | Complete; project adoption and upstream decisions remain open |

---

## 3. Evidence Base

### 3.1 Primary Sources Reviewed

Access/check date for this study is **September 17, 2026 America/New_York / September 18 UTC**. The date identifies this inspection, not a source's publication date.

| Source | Version / snapshot | Authority and inspected scope | Limitation |
|---|---|---|---|
| [CSAPI Part 1][P1] and [Part 2][P2] | Approved v1.0; publication July 16, 2025 | Controlling resource, filter, temporal and normative-test clauses | Prose, tests and supporting OpenAPI do not agree in every detail |
| [Official tagged artifacts][TAG] | `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`, tag `v1.0.0` | Sampling/observation clauses, query requirements, test A.51, relevant OAS paths and examples | Reproducible artifact evidence; OAS does not override the approved requirements |
| [Features Part 3][F3] | Approved v1.0, OGC 19-079r2 | Queryables, Filter, Features Filter and their tests | General mechanism; no automatic CSAPI property/relationship mapping |
| [CQL2][CQL] and [JSON schema][CQLJSON] | v1.0.0, OGC 21-065r2; `/cql2/1.0/` schema | Basic comparisons, spatial classes, JSON encoding and tests | Schema acceptance alone does not establish selected-class semantics |
| [SFA][SFA] and [SWE Common][SWE] | SFA 1.2.1 / OGC 06-103r4; SWE 3.0 / OGC 24-014 | Horizontal spatial operations; typed result components, units and nil values | Neither supplies the proposed observation queryable mapping |
| [Part 4 draft][P4] | `05a3c62d198ee52d0cf81a734b700967b7d864a1`, unchanged | Relevant explicit/parametric geometry and time limitations from IDR-SRV-058 | Working draft, not adopted scope; no repeated full type inventory |
| [CS-Go source][CSGO] | `b1fd2e0e9bd69e222d05258d659a842ca24502cb`, unchanged | Public parser, repository filters, declarations and test source | Inspected code, not executed behavior; unpublished work remains unavailable |
| [OSH source][OSH] | `9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08`, unchanged | Public handlers, internal observation filters, relevant in-memory/H2 evaluation paths | Backend-specific source assessment, not a live-service or whole-OSH certification |
| [OSH PR 323][OSHPR] | Open/unmerged; head `2da298be75f548802d9190455c48679054f6c1c2` | Proposed simple observation-result equality | Not merged behavior, not CQL2, and not an executed test result |
| [Official issue 165][H165], [179][H179], bounded latest-issue check | Open; relevant comments inspected | Unresolved association/property mapping and reported-new-issue search | Discussion is informative; no newly filed issue matching the meeting account identified |
| [PostGIS intersection][PGIS], [PostgreSQL recursion][PGREC] and [isolation][PGISO] | Documentation checked; PostgreSQL 18 references | Existing-design feasibility, cycle controls and transaction consistency | No index benchmark, query plan measurement, or installed-version claim |

### 3.2 Supporting Sources Reviewed

The internal baseline is commit `a17a88d71b63151e2a8d718984757611cc9a4a3c`, containing the accepted plan for execution. The relevant accepted research sections are identified in §4.5 and §7. Goal v1.6 and draft Guide v0.1 remain unchanged. IDR-SRV-058 is accepted; its synthesis addendum did not select a Part 4 implementation option.

The [shared history register][HISTORY] was consulted and refreshed only for this topic. Official `master` remains `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`; the Part 4 branch and publication tag remain at the pins above. Issue 165's July discussion remains unresolved. Issue 179 includes a September 3 record of a planned request for author clarification, not a published resolution. A September 17 comment on issue 175 concerns sorting alignment; it is not the anticipated filtering issue and does not add sorting to this study.

### 3.3 Evidence Quality Notes

- The user's meeting recollection establishes motivation, not verified implementation or an SWG decision. No unavailable branch, email, or issue content is inferred.
- Peer claims below are bounded to inspected files/commits. Code paths, declaration strings and test-source presence are different levels of evidence; no peer server was built or run.
- Expected sets in §4.1 are analyst-created examples, not official conformance fixtures. Six fixed-data diagnostic assertions checked their arithmetic/association results; they do not validate CQL2, PostGIS, or any server.
- The recommendation rests on published filtering mechanisms plus explicit proposed mappings. Naming a queryable does not standardize its meaning across independent implementations.

---

## 4. Findings by Research Question

### 4.1 Q1 - Existing Building Blocks Do Not Answer Every Spatial Question

#### 4.1.1 Approved behavior and the artifact boundary

**Finding.** Part 1 supplies sampling-feature geometry filtering; Part 2 supplies observation association and time filtering. Observation test A.51 follows `sampleOf` recursively. These establish an existing multi-request route, not a generic geometric predicate over every related resource. An observation has no required top-level geometry; its result can nevertheless be a position measurement. [P1], [P2], [SF], [OBS], [ATS]

The inspected [observation OpenAPI path][OASOBS] lists `dataStream`, `system`, and `observedProperty` beyond the corresponding numbered observation query requirements, while omitting inherited keyword filtering. The [tagged filtering source][OBSQUERY] has the observation `observedProperty` requirement commented out. Neither `geom` nor `filter` is advertised there. The [sampling-feature path][OASSF] does advertise `geom`. This confirms the existing IDR-SRV-011 warning: the bundled parameter list is not the normative contract.

**Interpretation.** Distinguish three problems: an implementation missing an existing requirement; a cumbersome but expressible workflow; and a genuinely additional query meaning. The following cases separate them. All example spatial coordinates use a small CRS84 square, **A = [0,2] x [0,2]**, with explicit horizontal geometry. IDs are illustrative local IDs.

#### 4.1.2 Case 1: Static direct sampling geometry

| Observation | Direct sampling feature | Intersects A? |
|---|---|---|
| S1 | Point (1,1) | Yes |
| S2 | Point (3,1) | No |
| S3 | Point (2,1), on boundary | Yes |
| S4 | Associated feature has no geometry | Not a positive spatial match |
| S5 | No sampling-feature association | Not a positive spatial match |

Expected observations for **direct sampling geometry intersects A**: **S1, S3**. Intersection includes the boundary; containment is a different predicate. In this flat, static fixture, querying sampling features by `geom` and then observations by the selected feature IDs can give that result.

Existing-standard request pattern, shown with decoded values for readability; real clients must URL-encode them and follow all pages:

```text
GET /samplingFeatures?geom=POLYGON((0 0,2 0,2 2,0 2,0 0))
GET /observations?foi=<matching-feature-IDs>
```

A client must handle ID-list/request-size limits and deduplicate results if batching requests. It must also account for changes between requests. These costs motivate investigation; they do not prove a single-request extension will always be cheaper.

#### 4.1.3 Case 2: A matching ancestor and an outside sampling point

Let sampling curve **R** be `LINESTRING(1 1,3 1)`. Point **P1** is (1,1); point **P2** is (3,1). Both have `sampleOf` relationships to R. Observation **C1** refers to P1 and **C2** to P2.

| Operation / intended meaning | Expected result |
|---|---|
| Sampling-feature geometry intersects A | P1 and R |
| Observation `foi` matches P1 or R through the sampling chain | C1 and C2 |
| Observation's **direct sampling point** intersects A | C1 only |

**Interpretation.** C2 is not necessarily a filtering bug in the two-stage workflow: its sampling ancestor R intersects A. But returning it as a measurement taken inside A would answer a different question. Restricting the first query to a suitable known leaf-feature set can help in some datasets; it is not a universal solution for arbitrary sampling relationships or mixed sampling types.

This is the central discriminator for future tests. An ultimate lake, ocean, trajectory, or viewing region may intersect the search area without establishing that every associated observation's sampling location does. Association traversal must not silently redefine the geometry being tested.

#### 4.1.4 Case 3: Geometry when the observation applies

Mobile sampling feature **M** has an explicit retained position at 10:00Z of (1,1), and at 11:00Z of (3,1), on the same day. T1 has phenomenon time 10:00Z and delayed result time 12:00Z; T2 has phenomenon time 11:00Z and result time 11:05Z. Evaluate after both positions are known.

| Spatial meaning | Expected observations |
|---|---|
| Position at each observation's phenomenon time | T1 |
| Latest/current position of M | Neither |
| M's combined trajectory intersects A, so select all its observations | T1 and T2 |

The two position instants alone do **not** determine the correct result for a new observation at 10:30Z. Interpolation, carry-forward, maximum age and correction rules are separate choices. Nor does `resultTime=latest` select the newest sampling position: delayed T1 has the later result time.

**Interpretation.** A time-aware spatial query needs sufficient history and a declared selection rule. A language extension cannot recreate missing history. Applying a spatial and temporal predicate separately to unrelated versions would also be wrong; they must refer to the intended observation and its corresponding geometry.

#### 4.1.5 Case 4: Measured value plus related specimen properties

Assume a declared scalar temperature component in `Cel`, explicit related-specimen attributes, and observation phenomenon time 12:00Z for every row. These specimen fields are an illustrative mapping, conditional on supporting their type; this case does not adopt Part 4.

| Observation | Temperature | Specimen material | Specimen sampling time |
|---|---:|---|---|
| V1 | 26 | water | 09:00Z |
| V2 | 26 | soil | 09:00Z |
| V3 | 26 | water | 11:00Z |
| V4 | 24 | water | 09:00Z |
| V5 | 99999, explicitly declared nil | water | 09:00Z |

For **temperature >25 Cel AND material=water AND sampled before 10:00Z**, the expected set is **V1**. A nil sentinel is not an extreme temperature, and specimen sampling time is not observation phenomenon time. A stream's `observedProperty` selection establishes what it measures, not which individual values satisfy this threshold.

A second stream containing `80 [degF]` cannot be compared numerically with 25 Cel without an explicit compatible conversion. Text `"26"` is not automatically numeric 26; `[20,30]` does not automatically mean any element exceeds 25. These are reasons to begin with explicit per-datastream scalar mappings rather than a universal `resultValue` field.

#### 4.1.6 A small metadata comparison

For a sampling feature's own name or type, an advertised simple-property equality parameter may suffice. Richer Boolean combinations or spatial predicates can justify Features Part 3 on that feature endpoint. This is simpler than an observation predicate whose value comes from another resource, version, or datastream schema. No repeat of the full CSAPI endpoint audit is needed to establish that distinction. [P1], IDR-SRV-011 §5.4

### 4.2 Q2 - A Reusable Filtering Mechanism, Not an Automatic CSAPI Binding

**Finding.** Features Part 3 separates reusable Queryables/Filter mechanisms from its Features-specific binding. Queryables may describe a virtual view rather than fields present in returned representations. This makes a named observation-associated geometry feasible without changing observations into GeoJSON features. The standards do not define that CSAPI-specific name or relationship. [F3], §§6, 8-9.

**Recommendation.** Evaluate the following six complete conformance classes as the initial optional addition. Claim them only after their requirements and applicable dependencies/tests are satisfied:

| Standard URI prefix | Class suffix | Purpose |
|---|---|---|
| `http://www.opengis.net/spec/ogcapi-features-3/1.0/conf/` | `queryables` | Discover named, typed searchable properties |
| Same Features prefix | `filter` | Submit a filter and advertise supported language/CRS behavior |
| `http://www.opengis.net/spec/cql2/1.0/conf/` | `basic-cql2` | Basic scalar comparisons, Boolean combinations and null handling |
| Same CQL2 prefix | `basic-spatial-functions` | Intersection capability |
| Same CQL2 prefix | `basic-spatial-functions-plus` | The full additional spatial-literal set, including polygons |
| Same CQL2 prefix | `cql2-json` | JSON expression encoding |

**Dependency boundary.** Features Part 3 Filter depends on Queryables. Its Annex A.1 additionally makes Queryables testing dependent on OGC API - Common Part 1 JSON, whose transitive dependencies include Core and Landing Page; this ATS dependency is not listed in the §6 requirements-class table. CQL2 spatial-plus depends on basic-spatial, which depends on Basic CQL2. JSON depends on Basic CQL2, the applicable selected language classes, and GeoJSON Geometry Objects when spatial support is claimed. Basic CQL2 names SFA Architecture, Time Ontology, RFC 3339, JSON Schema and Unicode dependencies. Reuse applicable existing server support, but verify this dependency closure before claiming the additional classes. [F3], Annex A.1/A.3; [CQL], §§7-8.

The spatial `plus` class must not be described as polygon-only support: it adds the complete specified literal set. Conversely, full Spatial Functions would bring additional topological predicates; it is not needed merely to support intersection. Temporal Functions, arrays, arithmetic, custom functions and property-property comparisons are not selected just because they exist. Basic date/timestamp comparisons and existing CSAPI time parameters cover the initial temporal inputs; historical geometry selection remains a mapping question. [CQL], §§6-8.

#### 4.2.1 Proposed endpoint and discovery boundary

The initial candidate applies to `/observations` and `/datastreams/{id}/observations`, with any equivalent advertised observation collection routes handled consistently. Each filterable endpoint should link to the queryables document applicable to that scope; provide the HTTP `Link` header checked by the generic tests. The generic classes do not prescribe a new `/observations/queryables` path, so its eventual URL is a documented project choice.

Proposed queryable scope:

- Across observations: common metadata plus a precisely described sampling-geometry value, where legitimately available.
- Within a datastream: selected scalar result components with fixed type, property identity, units and nil handling.
- Selected related-resource attributes: only where their association, value multiplicity and time meaning are explicitly mapped. Specimen-specific examples remain conditional; they are not a prerequisite for basic spatial support.

Use the queryables schema to restrict accepted names (`additionalProperties:false`). Keep publicly unsupported names distinct from declared names whose values are legitimately absent in an individual record. Publish capability declarations, API parameter definitions, and actual implementation scope together. The Features-specific conformance class is not needed for non-feature observations; if later claimed elsewhere, account for its requirements and its ATS's GeoJSON dependency rather than assuming all generic tests apply unchanged.

The queryables resource uses JSON Schema Draft 2020-12 and `application/schema+json`; follow the standard's special spatial-property schema rules rather than describing geometry as an arbitrary object. Default filter coordinates are CRS84/CRS84h, and unsupported requested CRS values must produce an error. Native parameters and the CQL2 expression combine with AND; only resources for which the combined predicate is TRUE are selected. With the proposed closed queryables schema, undeclared queryable names are errors, whereas legitimately missing values have the defined null semantics. [F3], §§6, 8; [CQL], §6.

#### 4.2.2 Proposed expression and transport

**Illustration only:** suppose a datastream advertises `samplingGeometry` and `temperature`, with the latter measured in Cel. The following CQL2 JSON expression combines direct sampling-geometry intersection with a threshold. Those property names and their mappings are proposed examples, not existing CSAPI parameters:

```json
{
  "op": "and",
  "args": [
    {
      "op": "s_intersects",
      "args": [
        { "property": "samplingGeometry" },
        { "type": "Polygon", "coordinates": [[[0,0],[2,0],[2,2],[0,2],[0,0]]] }
      ]
    },
    { "op": ">", "args": [{ "property": "temperature" }, 25] }
  ]
}
```

For this candidate, pass the URL-encoded expression as `filter` with `filter-lang=cql2-json` on the relevant GET endpoint. Declare JSON as the actual default if it is the only supported language. Native `foi`/time and other supplied filters still combine with it according to the filtering contract; the extension must not reinterpret them. JSON support does not, by itself, introduce a POST search route.

**Tradeoff.** JSON-only avoids an additional text grammar in the first supported boundary but is less convenient for hand-written URLs and unsuitable for text-only clients. A later Text encoding can share the validated expression model. This is a bounded recommendation, not a claim that JSON is required by OGC or proven cheaper in Glaux.

#### 4.2.3 Implementation details that affect an honest claim

- JSON-schema validation is only one check. Validate selected operators, arity, queryable types, dates and geometry semantics before translation to storage operations; the general schema admits capabilities beyond this candidate.
- Use typed date/timestamp literals rather than implicit string casts. CQL2 timestamp literal syntax uses UTC `Z`; do not silently transplant every accepted form of an existing CSAPI time parameter into that grammar.
- Required spatial literal support and CRS84/CRS84h handling must not be reduced to an undocumented subset. Height coordinates do not make an intersection predicate volumetric: SFA separates horizontal spatial operations from Z/M information. [SFA], §6.1.2.5.
- Reject unsupported languages, CRS, names, operators and invalid values consistently with the selected contract and existing error design. Do not silently ignore an unrecognized filter or evaluate only part of it.
- Supporting these classes is not a claim to implement every CQL2 capability or to meet the separate, broader OGC CQL2 Reference Implementation qualification.

### 4.3 Q3 - Define the Value Being Filtered Before Defining Its Syntax

**Interpretation and recommended boundary.** The proposed searchable geometry should identify the **direct associated sampling feature's explicit geometry applicable at the observation's phenomenon time**, when that value can be established. Static geometry with suitable validity and an explicit retained version are useful starting cases. This is a project mapping to decide, not wording already mandated for a CQL2 property by CSAPI.

| Subject | Required decision or proposed boundary |
|---|---|
| Geometry role | Do not substitute the sensor location, whole ultimate feature, ancestor, observed position result, or derived footprint for the selected sampling geometry. Expose a distinct, documented queryable if another role is needed. |
| Association absence | Do not assign every sampling feature of a datastream to every observation. A missing direct association needs an explicit, justified resolution rule or remains unavailable. |
| Sampling chains | Existing `foi` association matching remains intact. General recursive geometric evaluation is not included merely by supporting CQL2. |
| Time | Bind geometry and relationship versions to the declared observation time. Do not substitute latest description or result time for phenomenon time. |
| Gaps/intervals | Unknown historical position, interpolation, correction, or interval-wide selection is not silently resolved. Support only a documented rule with adequate evidence; otherwise identify the unavailable value/capability. |
| Part 4 shapes | An anchor point is not a sphere, sector or camera footprint. Relative pose resolution is different from sampling-chain traversal; derived shapes require separate justified semantics. |
| Units and schema | Resolve result queryables through the stream contract, component path, property definition, scalar type, unit and nil semantics, independently of response encoding. |
| Multiple/array values | Do not choose an arbitrary related value or assume any/all element behavior. Require an explicit mapping or leave the capability unsupported. |
| External links | Use admitted local facts or controlled cached evidence. No arbitrary synchronous network crawling during untrusted queries. |

**Finding.** Part 1 permits dynamic sampling-feature properties to be observation-backed and allows snapshots; its `datetime` validity filtering is not itself a general reconstruction algorithm. The Part 4 study identified additional draft location-time rules and unresolved frame/implicit-shape behavior. Neither supplies a complete rule for every historical observation query. [SF], [COMMON], IDR-SRV-058 §4.3.3

An observation-backed attribute illustrates a further distinction: values 28 at 09:00 and 20 at 10:00 satisfy **any value >25 during the interval**, but not **selected as-of 10:00 value >25**. The initial recommendation does not include an unrestricted search across all such dynamic relationships. A needed mapping can be considered separately within the existing model.

For authorized but absent inputs, define a consistent mapping to CQL2's null behavior; a spatial or numeric predicate must not fabricate a match. Preserve the source distinction between missing content, explicit null and SWE nil reasons even when the query view deliberately treats them alike. **Hidden is not merely null:** authorization must be resolved before applying an expression, as explained in §7.1.

### 4.4 Q4 - Peer Implementations Show Useful Work, Not a Complete Solution

#### 4.4.1 CS-Go: named queries and direct associations

At the inspected pin, the observation parser exposes named ID, keyword, datastream, system, feature-of-interest, observed-property and time filters. The repository compares `foi` against the observation's direct `sampling_feature_id`; system filtering joins its datastream. `observedProperty` searches datastream property JSON as text, and `q` searches serialized observation parameters/results. These are not typed result-value comparisons or a general CQL2 expression interface. [CGQUERY], [CGREPO]

**Interpretation.** Direct `foi` ID matching is narrower than the sampling-chain behavior examined in test A.51. That is a source-derived compatibility concern to verify, not a reason to redefine Glaux's baseline or a claim that a live deployment was tested. Likewise, text containing `26` is not evidence of a numeric threshold query.

The public branch/PR and source search did not establish the reported Features Part 3/CQL2 implementation. Current public-main absence cannot rule out unpublished or private work. Conformance declarations and observation test source do not provide an independent CQL2 oracle. Sampling-feature spatial selection is separately implemented with `ST_Intersects` and covered by source-level expected-membership assertions; that does not establish a spatial observation endpoint. [CGCONF], [CGTEST], [CGSPATIAL], [CGSPTEST]

#### 4.4.2 OSH: public spatial inputs versus backend evaluation

OSH's observation handler maps `bbox` and WKT `location` inputs to an internal observation `phenomenonLocation` filter, rather than directly expressing a sampling-feature geometry join. The internal predicate evaluates a nonnull location carried by the observation. Source inspection found that the in-memory store invokes the full observation predicate, while the examined H2 post-filtering path applies the value predicate without the same location test. The observation interface's fallback-to-feature documentation therefore cannot be taken as proof that this public query consistently evaluates related sampling geometry. [OSHOBS], [OSHLOC], [OSHFILTER], [OSHMEM], [OSHH2]

This is a **code-derived backend/contract discrepancy**, not an executed failure or a statement about every OSH deployment. It is precisely why parser support, interface comments, database behavior and tests must be traced together. Public feature `p:<property>` filters also show custom property matching, not an automatically standardized CQL2 or observation-backed-property contract. Feature REST test source checks property and bbox membership, while an observation datastore fixture checks a typed Java value predicate; neither demonstrates public CQL2 or a correct sampling-geometry observation query. The inspected observation/FoI selection path does not establish recursive sampling-chain matching. [OSHFEATURE], [OSHFOITEST], [OSHVTEST], [OSHFOI]

#### 4.4.3 Open OSH result-filter proposal

[PR 323][OSHPR], inspected at its unmerged head, proposes simple `filter=field=value` result predicates for schema-resolved top-level scalar components, using parent/datastream context. It OR-combines the supplied predicates and associates each with its datastream; these are implementation-specific rules, not a CQL2 Boolean language. The proposal identifies CQL2 as future work, and its three-file patch adds no test file. It should not be advertised as Features Part 3 support merely because it uses a parameter named `filter`. [OSHPRCODE]

**Interpretation.** This is useful evidence that result-value access is a practical implementation concern. It does not settle cross-resource spatial meaning or provide a stable shared interface for Glaux to copy. The report's recommendation does not depend on this PR being merged.

### 4.5 Q5 - Reconsider the Deferral, Preserve the Existing Architecture

**Recommendation.** Move from a blanket first-profile CQL2 deferral to a specifically bounded optional filtering proposal for the user's decision. The concrete cases now supply the demonstrated need called for by IDR-SRV-011. Keep all existing CSAPI filtering obligations and avoid introducing unsupported historical or parametric behavior under a generic geometry label.

This qualifies, rather than erases, the earlier research:

- **IDR-SRV-011 §§5.4, 17.2 and decision P-011-10:** its finding that CQL2 is not automatically inherited remains valid. Its deferral is the project choice now proposed for reconsideration.
- **IDR-SRV-026 §§8.2-8.3, 11.4:** geometry-role and time distinctions remain essential. The proposed addition would supply an explicit query interface, not new evidence that every geometry can be derived.
- **IDR-SRV-017/018 and 027/034:** typed relationships, temporal context, contract-bound value access, and consistent paging remain useful implementation foundations.
- **IDR-SRV-058:** the specialized-type and implicit-geometry limitations remain unchanged. Adopting filtering need not adopt Part 4; adopting Part 4 would not automatically solve filtering.
- **Draft Guide §§1.2, 4.4 and 6.3:** keep the approved baseline, but discuss replacing the general CQL2 exclusion with a precisely described optional capability if the project lead accepts that direction. Relevant persistence, security and testing sections would need targeted additions, not wholesale replacement.

No Goal change is presumed necessary merely to complete this research. If an optional filtering capability is adopted, discuss whether its significance warrants a short Goal clarification; the detailed language/classes, mappings, and verification belong in the Guide.

---

## 5. Decision Analysis

| Option | Benefits | Costs / limitations | Recommendation |
|---|---|---|---|
| Retain existing CSAPI filters and document multi-request workflows | No additional public contract; works for simple cases | Client batching/paging/consistency work; does not supply typed value predicates or universally mean direct spatial sampling | Retain as baseline, not as the whole answer to the identified needs |
| Preserve architectural room and defer all enhanced filtering | Smallest immediate implementation burden | Leaves the demonstrated combined-query needs unresolved | Defensible if resources dictate, but no longer the preferred research recommendation |
| Add bounded Features Part 3/CQL2 with explicit observation queryables | Reuses published filtering machinery; discoverable; can combine spatial, scalar and related conditions | Requires complete selected classes, CSAPI mapping choices, safe execution and independent tests | **Preferred for planning discussion**, initially within §4.2-4.3 boundaries |
| Add custom spatial/value parameters instead | Could address a narrowly defined immediate request | New syntax/semantics and interoperability burden; likely duplication as needs combine | Consider only if a concrete need cannot reasonably use the standard mechanism |
| Adopt broad CQL2, arbitrary joins, dynamic reconstruction and all Part 4 geometry at once | Broad theoretical expressiveness | Much greater implementation surface; multiple unresolved meanings; unnecessary for the first demonstrated cases | Do not select for this supplement |

The preferred option changes no existing mandatory CSAPI behavior and does not reduce the full server completion target. Optional capability sequencing is not permission to claim partially implemented conformance classes.

---

## 6. Key Recommendations

1. **Use the four concrete cases to define acceptance of any query addition.** Prioritize correct returned observation IDs, not just accepted syntax. Keep direct-location and ancestor-association queries distinguishable.
2. **Discuss adding the six-class candidate in §4.2.** Select JSON initially unless identified client requirements justify Text as well; do not adopt a larger CQL2 surface without a use case.
3. **Make sampling geometry and per-stream scalar meanings explicit.** Start with known explicit geometry and schema-bound scalar values. Admit related-property mappings only when their identity, time, multiplicity and authorization are settled.
4. **Do not silently fill semantic gaps.** Missing history, external geometry, relative frames and Part 4 volume calculations remain limitations unless a separately justified rule supplies the needed value.
5. **Reuse the proposed server architecture and verification approach.** Add bounded parsing/mapping/query execution and tests within existing modules and storage; assess performance before promising improvements.
6. **Use the SWG issue and newer implementation work when they become available.** The useful questions concern endpoint/queryable naming, geometry/time meaning and portable result-field discovery. No issue filing or developer contact was performed or is required to accept this report.

These recommendations are research outputs awaiting the project lead's decision, not new requirements imposed by report publication.

---

## 7. Implementation Implications and Estimates

### 7.1 Incremental Changes Within the Existing Design

| Area | Existing basis | Incremental implication |
|---|---|---|
| Public API and discovery | Guide §§4.1, 4.4, 6.3; IDR-011/014 | Add only declared filter parameters/classes and linked queryables documents; preserve existing endpoint semantics |
| Query parsing and validation | Existing typed query and error design | Decode JSON, enforce expression/class/type limits, resolve allowlisted names, and translate safely; do not splice client text into SQL |
| Relationships and geometry | IDR-017 §§10.3, 12.3; IDR-026 §§11.4, 12 | Resolve the selected local association and geometry role; prevent cycles/duplicates and avoid request-time external crawling |
| Time and historical evidence | IDR-018 §§8, 11; IDR-026 §§11.1-11.3; IDR-034 §8.3 | Use retained geometry/relationship versions only under a declared selection rule; distinguish missing history from an empty spatial match |
| Result fields | IDR-022/024; IDR-027 §§9.4, 12.3; IDR-028 §7 | Bind selective scalar query projections to the stream contract; preserve source encodings and nil/unit meaning |
| Spatial storage | Existing PostgreSQL/PostGIS proposal | Use exact spatial evaluation, with index candidate filtering where appropriate; bounding-box overlap alone is not the general intersection answer |
| Access and disclosure | IDR-039/040, especially 040 §8.6 | Authorize the queryable and participating object/property/relationship view before evaluating predicates, counts and pages |
| Paging and consistency | IDR-011; IDR-027/029 | Apply the complete authorized filter before ordering, counts and page limits; bind continuation to the same query/mapping/consistency context |

**Security interpretation.** A visible observation does not authorize using every hidden related attribute to decide whether it appears. Substituting NULL for a protected value can itself disclose information through `IS NULL`, negation or changing membership. Reject forbidden queryable use or apply the existing concealment policy to the eligible view consistently; reserve ordinary null semantics for legitimately exposed missing data. Test results, counts, errors and cursors with datasets that differ only in protected facts.

**Performance interpretation.** The existing database supports the relevant spatial and relational operations. This is a feasibility judgment, not a throughput result. Bound expression size/depth, geometry complexity, ID counts, traversed work and execution time. Deduplicate observation identities before counting/paging. A traversal budget failure must not silently return a partial answer as complete. PostgreSQL's recursive-query guidance warns that an outer LIMIT is not a sufficient production cycle/work safeguard. [PGREC]

A single server query can remove the client's intermediate ID-list transfer and inter-request race, but does not automatically create stable results across later pages. PostgreSQL isolation rules distinguish statement snapshots from transaction-wide repeatability; the existing paging/snapshot design still needs an explicit retention and consistency contract. [PGISO]

### 7.2 Relative Complexity

No calendar estimate or measured implementation cost is established here.

| Work item | Relative complexity | Assumptions / boundary |
|---|---|---|
| Queryables discovery and API declarations | Low-medium | Reuses existing API-definition and schema delivery mechanisms |
| Complete selected CQL2 JSON classes and safe translation | Medium | Requires operator/type/null/spatial semantics, not only JSON parsing |
| Static explicit sampling-geometry filtering | Medium | Known local associations and valid geometry; existing PostGIS storage |
| Per-datastream scalar result predicates | Medium | Fixed contracts, selective queryable set and explicit nil/unit mapping |
| Historical moving geometry | High / uncertain | Requires sufficient history and settled temporal/correction rules |
| Parametric volumes, arbitrary dynamic-property traversal or broad joins | Outside initial candidate | No general engine or whole-Part-4 commitment follows from the recommendation |

### 7.3 Verification Required Before Any Claim

| Test subject | Independent expected behavior |
|---|---|
| Flat static geometry | S1/S3 match; outside, absent geometry and absent association do not become positive spatial matches |
| Ancestor versus direct geometry | C1/C2 for defined recursive association; C1 alone for direct sampling geometry |
| Historical versus current | T1 for exact phenomenon-time geometry; neither for current geometry; no invented result at 10:30Z |
| Scalar and related attributes | V1 only; declared nil does not pass as a large number |
| Property identity versus value | Finding a temperature stream does not establish temperature >25 |
| Queryables discovery | Applicable schema/link and declared scope agree with accepted property names and types |
| Class coverage | Entire selected class/operator/literal obligations pass, including required spatial types; no polygon-only claim for the plus class |
| Errors and NULL | Unknown name differs from authorized absent value; invalid types/operators/CRS fail; TRUE/FALSE/NULL and negation are checked |
| Coordinates | Boundary, coordinate order, supported heights and relevant antimeridian cases have explicit expected semantics; no volumetric claim |
| Chains and external links | Cycles terminate safely, multiple paths do not duplicate observations, limits are explicit, no arbitrary network retrieval |
| Authorization | Hidden property/relationship changes do not leak through membership, counts, errors or continuation |
| Paging and combined filters | Identical authorized filter is applied before every page; `foi`, native time filters and CQL2 combine consistently |
| Cross-encoding results | Equivalent logical measurements produce equivalent predicate results regardless of response encoding |
| Peer comparison | Differences are attributed to exact source/runtime profiles; successful parsing alone is not interoperability evidence |

Use the selected standards' abstract tests plus these mapping-specific cases. Generic filtering conformance alone cannot verify that Glaux chose the right sampling feature or historical geometry.

---

## 8. Risks, Constraints, and Open Questions

### 8.1 Risks and Constraints

| Risk | Consequence | Treatment |
|---|---|---|
| A generic geometry name hides several spatial meanings | Plausible but incorrect observation sets | Declare one role/time mapping; separate alternatives |
| Recursive association is mistaken for direct spatial selection | Outside sampling points appear as inside observations | Preserve Case 2 as a discriminator |
| Missing history is replaced with current state | Incorrect historical answers | Expose the limitation; require retained evidence and an explicit rule |
| Result typing is inferred from JSON text | Unit, numeric, array and nil errors | Per-contract typed scalar mappings |
| Public parser or PR is mistaken for implemented semantics | False compatibility claims | Trace storage/tests and distinguish open proposals from merged code |
| Hidden related data influences filtering | Information disclosure | Authorized query view and negative disclosure tests |
| Optional standard becomes an unrestricted capability promise | Unbounded implementation scope | Complete only deliberately selected classes and mappings |

### 8.2 Open Questions

1. **Project adoption:** Does the project lead want the recommended optional addition? Research acceptance and implementation-scope adoption remain separate decisions.
2. **Portable CSAPI mapping:** Which queryable names, endpoint discovery arrangements and geometry/time meanings will independent implementations or the SWG agree to share? The report's illustrative names are not that agreement.
3. **History boundary:** Which existing data can establish geometry at an observation instant, and what future rule should cover gaps, intervals and corrections? The initial candidate must not imply interpolation merely by exposing a geometry queryable.
4. **Client encoding needs:** Is JSON-only sufficient for intended first clients, or is Text also necessary? This changes parser/interoperability work, not the relationship semantics.
5. **Peer development evidence:** The reported CS-Go filtering branch and newly anticipated SWG issue were not identified. Their contents could refine the recommendation when supplied, but cannot be assumed now.
6. **Measured behavior:** Runtime correctness, query plans and performance of any implementation remain to be tested. This source-based study does not establish them.

These questions constrain implementation claims; they do not prevent a decision-usable research recommendation.

---

## 9. Validation Against Plan Success Criteria

| Plan criterion | Status | Evidence |
|---|---|---|
| Q1-Q5 answered or explicitly limited | Met as assessment | §§4.1-4.5; §8 |
| Four cases with spatial retrieval primary | Met | §§4.1.2-4.1.5; Appendix diagnostic |
| Existing behavior, workflow limits and additions separated | Met | §4.1.1 and Cases 1-2; §5 |
| Filtering classes, mapping, discovery and limits identified | Met | §4.2 |
| Geometry, recursion, time, Part 4, result type/unit addressed | Met | §§4.1, 4.3 |
| Pinned peer evidence and unavailable work distinguished | Met within source-inspection boundary | §§3, 4.4; no runtime claim |
| Incremental design/security/cost/paging/tests grounded | Met as design assessment | §7; existing-report section references |
| Concrete recommendation and unchanged conclusions explicit | Met | §§4.5, 5-6 |
| Template, references, coverage and later handoff preserved | Met | This structure; §§10-12 |
| Bounded official-history refresh | Met | §3.2; shared register v1.14 |

Research execution is complete for the planned assessment. The report remains **In Review**, and IDR-SRV-059 is not marked accepted or closed until the project lead responds.

---

## 10. Next Steps and Handoff

1. **Review and accept or revise this report** — Glaux Project Lead, next iteration; no calendar deadline imposed.
2. **On the next `proceed`, record acceptance and add the findings to the final synthesis as a separate addendum** — Codex. Preserve the original accepted synthesis and existing Part 4 addendum. Do not change the Goal/Guide in that iteration.
3. **Then discuss querying and Part 4 planning implications together** — Project Lead and Codex. Decide whether to adopt the optional filtering capability and any specialized sampling support before making agreed Goal/Guide changes.

Publication of this report does not choose the implementation option. No external issue, email, software installation, server implementation, or live-service test was performed as part of this iteration.

---

## 11. References

### Standards, tagged artifacts and maintenance evidence

- [P1: Approved CSAPI Part 1][P1]; [P2: Approved CSAPI Part 2][P2].
- [TAG: Official publication-source snapshot][TAG]; [SF: sampling-feature model/dynamic properties][SF]; [COMMON: common time/CRS clauses][COMMON]; [OBS: datastream/observation model and schema clauses][OBS]; [OBSQUERY: Part 2 filtering source][OBSQUERY]; [ATS: Part 2 normative test source, including observation `foi`][ATS].
- [OASSF: sampling-feature path][OASSF]; [OASOBS: observation path][OASOBS].
- [F3: Features Part 3, §§6-9 and Annex A][F3]; [CQL: CQL2, §§6-8 and Annex A][CQL]; [CQLJSON: official JSON expression schema][CQLJSON]; [SFA: spatial model and operators][SFA]; [SWE: scalar values, units and nil semantics][SWE].
- [P4: pinned draft Part 4][P4]; [H165: sampling-feature issue][H165]; [H179: property-filter issue][H179]; [shared upstream-history register][HISTORY].

### Implementation and database evidence

- [CSGO: inspected CS-Go tree][CSGO]; [CGQUERY: observation query parser][CGQUERY]; [CGREPO: observation filtering repository][CGREPO].
- [OSH: inspected OSH tree][OSH]; [OSHPR: open observation-value filtering proposal][OSHPR]. Exact inspected paths and source boundaries are recorded in §12.
- [PostGIS ST_Intersects][PGIS]; [PostgreSQL 18 recursive queries and cycles][PGREC]; [PostgreSQL 18 transaction isolation][PGISO].

### Glaux evidence

- [IDR-SRV-011](idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md), [IDR-SRV-026](idr-srv-026-geospatial-storage-and-query-strategy-report.md), [IDR-SRV-034](idr-srv-034-datastream-observation-and-status-update-semantics-report.md), and [IDR-SRV-058](idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md).
- [Final research synthesis](final-idr-research-report.md); [Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md); [Implementation Guide](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md). Other supporting report numbers and exact affected sections are identified in §7 and resolve through the overall plan's existing topic index.

[P1]: https://docs.ogc.org/is/23-001/23-001.html
[P2]: https://docs.ogc.org/is/23-002/23-002.html
[TAG]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2
[SF]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/clause_13_requirements_class_sampling_features.adoc
[COMMON]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/clause_7_requirements_class_common.adoc
[OBS]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/clause_8_requirements_class_datastreams.adoc
[OBSQUERY]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/clause_14_requirements_class_advanced_filtering.adoc
[ATS]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc
[OASSF]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/paths/samplingFeatures.yaml
[OASOBS]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/observations.yaml
[F3]: https://docs.ogc.org/is/19-079r2/19-079r2.html
[CQL]: https://docs.ogc.org/is/21-065r2/21-065r2.html
[CQLJSON]: https://schemas.opengis.net/cql2/1.0/cql2.json
[SFA]: https://docs.ogc.org/is/06-103r4/06-103r4.pdf
[SWE]: https://docs.ogc.org/is/24-014/24-014.html
[P4]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/05a3c62d198ee52d0cf81a734b700967b7d864a1/api/part4
[H165]: https://github.com/opengeospatial/ogcapi-connected-systems/issues/165
[H179]: https://github.com/opengeospatial/ogcapi-connected-systems/issues/179
[HISTORY]: ../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md
[CSGO]: https://github.com/SomethingCreativeStudios/connected-systems-go/tree/b1fd2e0e9bd69e222d05258d659a842ca24502cb
[CGQUERY]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/query_params/observation_query_params.go
[CGREPO]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/repository/observation_repository.go
[OSH]: https://github.com/opensensorhub/osh-core/tree/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08
[OSHPR]: https://github.com/opensensorhub/osh-core/pull/323
[CGCONF]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/api/conformance_handler.go#L43
[CGTEST]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/e2e/observations_test.go#L446
[CGSPATIAL]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/repository/spatial_filters.go#L13
[CGSPTEST]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/e2e/spatial_queries_test.go#L47
[OSHOBS]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/obs/ObsHandler.java#L501
[OSHLOC]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-core/src/main/java/org/sensorhub/api/data/IObsData.java#L83
[OSHFILTER]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-core/src/main/java/org/sensorhub/api/datastore/obs/ObsFilter.java#L136
[OSHMEM]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-core/src/main/java/org/sensorhub/impl/datastore/mem/InMemoryObsStore.java#L296
[OSHH2]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-datastore-h2/src/main/java/org/sensorhub/impl/datastore/h2/MVObsStoreImpl.java#L412
[OSHFOI]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-core/src/main/java/org/sensorhub/impl/datastore/DataStoreUtils.java#L289
[OSHFEATURE]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/feature/AbstractFeatureHandler.java#L76
[OSHFOITEST]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-service-consys/src/test/java/org/sensorhub/impl/service/consys/TestFois.java#L219
[OSHVTEST]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-core/src/testFixtures/java/org/sensorhub/impl/datastore/AbstractTestObsStore.java#L678
[OSHCONF]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/ConSysApiService.java#L75
[OSHPRCODE]: https://github.com/opensensorhub/osh-core/blob/2da298be75f548802d9190455c48679054f6c1c2/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/obs/ObsHandler.java#L549
[PGIS]: https://postgis.net/docs/ST_Intersects.html
[PGREC]: https://www.postgresql.org/docs/18/queries-with.html#QUERIES-WITH-CYCLE
[PGISO]: https://www.postgresql.org/docs/18/transaction-iso.html

---

## 12. Appendix: Checks and Reproducibility

### 12.1 Source and Scope Checks

- Rechecked official tag, `master`, and Part 4 branch with `git ls-remote`; all matched the pins in §3.
- Read the tagged sampling-feature, common, observation, query and normative-test clauses, plus the two relevant OpenAPI paths. The observation `observedProperty` requirement is commented out in source although its parameter appears in the path artifact.
- Retrieved issue 165/179 bodies and comments. Screened the latest 30 public issues/PRs by creation order for the anticipated new issue; newest item was issue 201, created September 15, about SystemEvent mapping, not enhanced querying. Read the updated sorting discussion only to distinguish it from the reported topic.
- Refreshed peer source/branch/PR evidence within the boundaries in §4.4. No installed dependency, runtime service or benchmark was used.

Peer search boundary: immutable source archives were scanned for `cql2`, `queryables`, `filter-lang`, `filter-crs` and `ogcapi-features-3`: 305 CS-Go Go/JSON/YAML/Markdown/module files and 2,224 OSH Java/JSON/YAML/Markdown/Gradle/XML files, with no matches at the main/master pins. Public branch listings and PR metadata were checked: CS-Go exposed only `main` and five closed PRs; OSH's relevant open PR 323 was followed to its actual patch and pinned head. These are bounded searches, not evidence about unpublished work.

The following immutable source anchors record the inspected implementation/test paths. Line numbers identify starting points; none of these tests was executed during this study.

| Project / evidence | Inspected source and section |
|---|---|
| CS-Go observation input and storage | [ObservationQueryParams][CGQUERY], lines 11-67; [repository filters][CGREPO], lines 208-275, including direct FoI at 251-252 and text searches at 255-272 |
| CS-Go spatial selection and tests | [Spatial predicate][CGSPATIAL], line 13 onward; [sampling-feature spatial assertions][CGSPTEST], lines 47-112; [observation time tests][CGTEST], lines 446-536 |
| CS-Go declarations | [Conformance handler][CGCONF], lines 43-67 |
| OSH observation input and geometry contract | [ObsHandler][OSHOBS], lines 501-546; [IObsData][OSHLOC], lines 83-86; [ObsFilter.testPhenomenonLocation][OSHFILTER], lines 136-140 |
| OSH storage evaluation | [InMemoryObsStore][OSHMEM], lines 296-300; [MVObsStoreImpl][OSHH2], lines 412-500, especially post-filter lines 492-500; [DataStoreUtils direct FoI IDs][OSHFOI], lines 289-306 |
| OSH own-property/spatial feature queries | [AbstractFeatureHandler][OSHFEATURE], lines 76-186; [TestFois][OSHFOITEST], property tests at 219-290 and bbox tests at 335-389 |
| OSH internal result predicate and declarations | [AbstractTestObsStore][OSHVTEST], lines 678-712; [ConSysApiService conformance][OSHCONF], lines 75-111 |
| OSH unmerged proposal | [PR-head ObsHandler][OSHPRCODE], lines 549-682; separate pin from the inspected master |

### 12.2 Fixed-Data Diagnostic

An in-memory JavaScript diagnostic independently checked six expected sets: static point inclusion; recursive association; direct point inclusion; exact phenomenon-time position; current position; and scalar/material/sampling-time selection. It used only the small fixed records in §4.1, a closed-square coordinate comparison, a visited-set relationship walk and explicit nil exclusion. The intersecting curve R was supplied as a known matching feature, not calculated by a general geometry engine.

| Assertion | Actual diagnostic output |
|---|---|
| Static direct point selection | `[S1, S3]` |
| Chain association selection | `[C1, C2]` |
| Chain direct point selection | `[C1]` |
| Exact phenomenon-time position | `[T1]` |
| Current position | `[]` |
| Value/material/sampling time | `[V1]` |

All six agreed with the independently stated expectations. This is an arithmetic/association sanity check, not execution of the illustrative HTTP requests, a CQL2 evaluator, a PostGIS query, or a server conformance suite. The malformed/hidden/CRS/paging cases in §7 remain proposed future verification, not passed tests.

---

## Report Completion Checklist

- [x] Topic ID and plan match the overall index
- [x] Core questions and all four use cases are addressed or explicitly limited
- [x] Findings, interpretations and recommendations are distinguished
- [x] Mutable sources are pinned and source-access/search limitations recorded
- [x] Spatial/temporal/relationship and value semantics have explicit example outcomes
- [x] Existing research conflicts and proposed changes are identified without rewriting accepted reports
- [x] Incremental implementation, security, cost and test implications are stated
- [x] Success criteria and next-step ownership are mapped
- [ ] Plan-owner acceptance and acceptance date recorded

The unchecked acceptance item is intentional. The user's next `proceed` can accept this report and authorize only the separate synthesis-addendum iteration.

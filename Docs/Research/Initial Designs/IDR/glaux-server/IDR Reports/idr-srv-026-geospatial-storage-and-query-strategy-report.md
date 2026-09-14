# Section 026: Geospatial Storage and Query Strategy - Research Report

**Topic ID:** IDR-SRV-026<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-026 Geospatial Storage and Query Strategy](../IDR%20Plans/idr-srv-026-geospatial-storage-and-query-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 7 objective questions, all 6 methodology phases, all 10 success criteria, and the required 14-field geospatial matrix<br>
**Methodology Used:** Authority-ranked spatial requirement extraction from approved CSAPI, OGC API - Features, GeoJSON, SensorML, SWE Common, controlled-source findings, and accepted IDR-SRV-001 through IDR-SRV-025; direct review of current primary technology documentation; implementation-evidence comparison; resource/geometry/query/storage classification; and bounded synthesis into a full-profile strategy plus downstream gates<br>
**Research Time:** Approximately 14 hours of AI-assisted execution on September 14, 2026<br>
**Official Standards Source Pin:** [`opengeospatial/ogcapi-connected-systems` `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Shared Register Baseline:** [OGC API - Connected Systems upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), Version 1.10; no geospatially material approved-standard change required a register update<br>
**Technology Documentation Snapshot:** PostgreSQL 18, released PostGIS 3.6 documentation, SpatiaLite 5.1, current DuckDB Spatial documentation, Rust `geo-types` 0.7.20, Rust `geojson` 1.0.0, and stable GDAL documentation; checked September 14, 2026 and treated as mutable implementation evidence<br>
**Controlled AEP Source:** `AC/224(JCGISR)D(2026)0005`, April 27, 2026, SHA-256 `56dc757b6e677b3584e3152a957849f21a24b22854f562613ff283a8b599da8c`; used only through accepted project findings and not redistributed<br>
**Document Purpose:** Establish the Glaux Server geospatial authority, storage, CRS, query, indexing, policy, dynamic-location, DDIL, and test baseline without defining physical schemas, preempting time-series research, or implementing the server<br>
**Author:** OpenAI Codex<br>
**Accepted By:** TBD pending Glaux Project Lead review<br>
**Acceptance Date:** TBD pending acceptance<br>
**Date:** September 14, 2026<br>
**Last Updated:** September 14, 2026

---

## Reading Guide and Evidence Labels

| Label | Meaning |
|---|---|
| **N** | Normative or standards-derived behavior |
| **A** | Project-controlling AEP/STANAG adoption or operational-context finding |
| **P** | Glaux architecture direction or recommendation proposed for acceptance here |
| **I** | Informative technology, implementation, test, or community evidence |
| **D** | Detailed design deferred to the named downstream topic |
| **X** | Evidence gap or unresolved decision requiring profile input, prototype, or benchmark |

“Location” is not one field. Feature geometry, platform position, sampling geometry, feature-of-interest geometry, deployment area, trajectory, footprint, pose, and collection extent have different subjects, authority, time, uncertainty, and disclosure rules. “SRID 4326” also does not by itself prove CRS84 axis order, valid transformation, ellipsoidal height, or known vertical datum.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Spatial Requirement Extraction Methodology
5. Geospatial Data-Category Inventory
6. Resource-Family Spatial Requirement Matrix
7. Geometry Type, CRS, Coordinate, Precision, Uncertainty, and Provenance Findings
8. Spatial Query, Filtering, Sorting, Pagination, and Extent Findings
9. Spatial Indexing and Performance Implications
10. Geospatial Storage Option Evaluation
11. Spatial-Temporal and Dynamic-Location Implications
12. SensorML, SWE Common, and Semantic Integration Implications
13. Security, Policy, Releasability, Precision-Reduction, and Audit Implications
14. DDIL, Cache, Synchronization, and Last-Known-Position Implications
15. Fixture, Conformance, Performance, and Interoperability Test Implications
16. Downstream Topic Handoff Matrix
17. Recommendations
18. Risks, Constraints, and Open Questions
19. Validation Against Plan Success Criteria
20. References

---

## 1. Executive Summary

Glaux should use **PostgreSQL/PostGIS as the authoritative geospatial engine for the full server profile**, inside the relational-hybrid authority accepted by IDR-SRV-025. Spatial facts must remain joined transactionally to canonical resource identity, typed relationships, valid/system time, provenance, semantic role, and policy. A JSON/GeoJSON document, application memory index, search engine, or analytical sidecar must not become a second source of spatial truth. **[P]**

The core abstraction should be a versioned **spatial assertion**, not a universal `location` column. Each assertion identifies its subject and role; preserves the exact source and source CRS/reference-frame facts; carries the parsed coordinate object; optionally carries a validated canonical query geometry; records dimensionality, axis order, vertical reference, epoch, precision/accuracy/uncertainty, time, provenance, policy, and transformation evidence; and can generate standards representations deterministically. Unknown CRS, axis, datum, or frame prevents unsafe normalization but does not require destroying quarantined source evidence. **[P]**

Approved CSAPI supplies firm resource distinctions. A System should include location; when reported it is the latest known location unless a snapshot time is requested. A Procedure is explicitly non-spatial and must not contain geometry. A Deployment location is a geographic discovery area that may be approximate and must not be confused with its systems or sampling features. Sampling Features model samples and sampling geometries, including points, curves/trajectories, surfaces, solids, specimens, and possibly abstract features of interest. Datastreams and ControlStreams relate to Sampling Features; observations and commands may identify a Sampling Feature rather than duplicating its geometry. **[N]**

For the approved API baseline, GeoJSON output uses RFC 7946 / CRS84 longitude-latitude order and optional WGS 84 ellipsoidal height. Alternate source CRSs may be preserved and processed internally; alternate API output CRS is only a capability if the applicable OGC API - Features Part 2 requirements are deliberately adopted and advertised. `ST_SetSRID` only labels geometry and must never masquerade as reprojection; `ST_Transform` or a controlled equivalent must record the chosen pipeline and grid/version evidence. Geometry with Z is not automatically subject to three-dimensional predicates, and vertical filtering is unsafe without a resolved vertical reference. **[N/I/P]**

The query baseline is broader than “put a GiST index on geometry.” Glaux must implement the inherited `bbox` behavior on applicable feature collections and, when advertising CSAPI Advanced Filtering, its WKT `geom` intersection filter. Input normalization must handle four/six ordinates, invalid latitude, degenerate boxes, antimeridian-crossing boxes, absent geometry, and family-specific geometry selection. Combined spatial, temporal, semantic, relationship, lifecycle, and policy filters use explicit AND semantics; policy applies before counts, extents, sorting, and pagination. Stable opaque keyset cursors bind the normalized query, spatial semantics/version, policy scope, sort, and snapshot/as-of watermark. **[N/P]**

GiST is the default PostGIS index family. B-tree indexes support identity/time joins; partial and functional GiST indexes may support active/current or transformed query projections; BRIN and SP-GiST are benchmark candidates for large spatially ordered append data or point-heavy workloads. Index bounding-box checks are candidate generation, not a substitute for exact predicates. Statistics, plan regression, geometry complexity, subdivision, dateline behavior, policy selectivity, and spatial-temporal correlation all belong in the performance corpus. **[I/P/D]**

Authoritative moving-platform history should remain immutable timed samples owned jointly with IDR-SRV-027. “Latest known position” is a derived projection carrying event time, receive time, freshness, uncertainty, provenance, and policy. A trajectory or footprint is usually a derived, versioned artifact; it must never silently replace samples or be presented as authoritative without an explicit source assertion. Sensor pose includes orientation and reference frame and cannot be collapsed into GeoJSON geometry. **[P/D]**

Security is semantic, not merely row access. Exact geometry, generalized geometry, hidden geometry, and derived extents are different disclosure products. Glaux must prevent inference through counts, bboxes, pagination, distance sorting, caches, tiles/exports, logs, errors, and repeated generalized responses. Derived products carry input-policy joins, algorithm/version, precision, provenance, and scope; policy changes invalidate or rebuild them. **[P]**

SpatiaLite is a conditional reduced-profile edge option, DuckDB Spatial an analytical/export sidecar, Rust `geo`/`geo-types`/`geojson` application support, and GDAL a quarantined import/export boundary. None replaces PostGIS for the full profile. Final physical tables, time-series partitioning, alternate-CRS API conformance, precision policy, synchronization conflicts, query grammar, and benchmark thresholds remain with their owning later topics. **[P/D]**

---

## 2. Scope and Plan Alignment

This report executes `IDR-SRV-026`, the second Category E topic. It identifies spatial facts, assigns authority and storage patterns, resolves geometry/CRS/query/index direction, and hands bounded decisions to persistence, ingestion, dynamic-data, security, DDIL, fixture, performance, conformance, and interoperability research.

It does **not** define SQL DDL, choose partition intervals, specify every AEP profile constraint, adopt OGC API - Features Part 2, standardize a Glaux spatial extension, decide time-series products, define synchronization conflict resolution, implement draft CSAPI Part 3, or implement the server. `IDR-SRV-027` and all later topics remain unauthorized during this iteration.

### 2.1 Research Question Coverage Matrix

| Plan question | Short form | Status | Evidence location |
|---|---|---|---|
| Q1 | What spatial data must be stored, indexed, preserved, derived, and exposed? | Complete | Sections 5-7 |
| Q2 | Which resource families need which spatial concepts? | Complete | Sections 5-6 |
| Q3 | How are spatial roles and authority distinguished? | Complete | Sections 4-7, 11-12 |
| Q4 | What query behavior is required or recommended? | Complete with extensions gated | Sections 8-9 |
| Q5 | Which storage/index architecture suits Rust and open deployment? | Complete with physical tuning deferred | Sections 9-10 |
| Q6 | How does space interact with time, semantics, DDIL, policy, and tests? | Complete with owning mechanisms deferred | Sections 11-16 |
| Q7 | What downstream implications follow? | Complete | Section 16 |

### 2.2 Decision Boundary

Decided here: spatial-assertion layers; PostGIS full-profile authority; CRS84 wire baseline; source-CRS preservation; safe-transform rules; family/role separation; query semantics; derived-product rules; dynamic-location projection; policy-before-derivation/query; default index families; reduced/analytical/import boundaries; test categories; and downstream ownership.

Deferred: physical schemas and migration DDL (`028`/`030`); position-sample partitions and retention (`027`); ingest transaction mechanics (`029`/`031`); endpoint query grammar (`033`/`051`); observation/status specifics (`034`); alternate CRS profile decision (`050`); security policy (`039`/`039A`); DDIL synchronization (`042`/`043`); deployment topology (`044`); performance thresholds (`054`); and final compatibility claims (`056`).

---

## 3. Evidence Base and Authority Classification

### 3.1 Controlling and Approved Sources

| Source | Pin/status | Authority | Spatial anchors | Availability/limit |
|---|---|---|---|---|
| OGC API - Connected Systems Part 1, OGC 23-001 | Approved 1.0; source tag `v1.0.0`, commit `8e03b236...` | Normative | System Location; Deployment Features; Procedure Location; Sampling Features; Advanced Filtering geometry filter | Public and locally inspected |
| OGC API - Connected Systems Part 2, OGC 23-002 | Approved 1.0; same tag | Normative | dynamic properties; Datastream/Observation Sampling Feature relations; System History; feasibility examples | Public and locally inspected |
| OGC API - Features Part 1, OGC 17-069r4 | Approved corrigendum | Normative dependency | Core CRS84, `bbox`, feature collections, spatial extents | Public |
| GeoJSON RFC 7946 | Internet Standard | Normative representation | geometry types, positions, CRS84-equivalent CRS, bbox, antimeridian | Public |
| SensorML 3.0, OGC 23-000 | Approved | Normative representation | positions, poses, reference frames, system/deployment descriptions | Public; accepted IDR-SRV-021 interpretation controls Glaux layering |
| SWE Common 3.0, OGC 24-014 | Approved | Normative data model | Vector, Matrix, reference/local frames, coordinate components, quality | Public; accepted IDR-SRV-022 controls component preservation |
| Controlled project AEP source | `AC/224(JCGISR)D(2026)0005`, 2026-04-27, recorded hash | Project-controlling within handling bounds | NATO JISR discovery/profile context carried through accepted reports | Controlled; no redistribution; exact unaccepted CRS/vertical requirements not inferred |
| Accepted IDR-SRV-015 through 025 | Accepted as of 2026-09-14 | Project baseline | canonical graph, identity, relationships, time, evidence, state, representations, validation, semantics, persistence | Local and reproducible |

### 3.2 Current Technology Primary Sources

| Source | Snapshot | Evidence class | Decision use |
|---|---|---|---|
| PostgreSQL 18 documentation | Checked 2026-09-14 | Mutable primary technical | transactions, partitioning, B-tree/BRIN, policy integration |
| PostGIS 3.6 documentation | Released 3.6 line, checked 2026-09-14 | Mutable primary technical | geometry/geography, SRID, transform, validity, GiST/BRIN/SP-GiST, 2D/3D predicates |
| SpatiaLite project documentation | 5.1 release line | Mutable primary technical | reduced embedded spatial option |
| DuckDB Spatial documentation | Current docs, checked 2026-09-14 | Mutable primary technical | analytical sidecar and bundled PROJ behavior |
| Rust `geo-types` documentation | 0.7.20, 2026-08-02 | Mutable primary technical | 2D application geometry model |
| Rust `geojson` documentation | 1.0.0 | Mutable primary technical | RFC 7946 serialization and preservation caveats |
| GDAL stable documentation | Checked 2026-09-14 | Mutable primary technical | format conversion and explicit large attack-surface warning |

### 3.3 Implementation and Community Evidence

- IDR-SRV-014A through 014G are informative only. CS-Go demonstrates PostGIS geometry, GiST indexes, spatial filters, deterministic cursor indexes, and PostGIS-backed integration tests; it does not establish scale or normative semantics.
- pygeoapi evidence warns against representation-dependent truth: separate GeoJSON/SensorML documents produced different populations. It also demonstrates that search-store geospatial support does not solve cross-representation authority.
- SECD shows that a live `bbox` positive control can coexist with silently ignored canonical filters. Query tests therefore require hit, miss, malformed, and combined-filter controls.
- OS4CSAPI discussions show partial generic OGC API Features/QGIS value through collections, GeoJSON, `bbox`, `limit`, and links, while nested/non-spatial resources remain client-dependent. “Works with QGIS” must be a versioned, bounded claim.

### 3.4 Evidence Confidence and Gaps

Confidence is high for approved API/GeoJSON geometry and query obligations, high for PostGIS capability direction, and medium for future workload tuning. AEP content is deliberately cited only by controlled identifier and accepted-project implications. No exact AEP CRS, vertical datum, accuracy, or releasability rule is asserted here unless already accepted. Part 4 sampling feature types and draft Part 3 are informative future evidence only and do not enlarge the approved baseline.

The official CSAPI source pin remains the accepted `v1.0.0` commit. The shared history register was checked at Version 1.10; no approved geospatial source change was found that changes this report’s authority baseline.

---

## 4. Spatial Requirement Extraction Methodology

Each candidate spatial fact was decomposed across fourteen required dimensions: category, resource family, source anchor, geometry type, time behavior, authority class, CRS/coordinate needs, query/index needs, temporal interaction, precision/uncertainty/provenance, policy, storage pattern, downstream owner, and unresolved notes.

The extraction applied these non-collapse tests:

1. **Whose space?** Resource, platform, sensor, sampler, ultimate feature, observed result, command target, or collection.
2. **Which role?** Feature geometry, current location, installation site, sampling geometry, area of operation, pose, trajectory, footprint, coverage, or discovery envelope.
3. **At what time?** Descriptive revision time, valid time, phenomenon time, result time, receive time, query snapshot, or derivation watermark.
4. **What authority?** Supplied assertion, parsed exact source, canonical fact, derived artifact, cache, or generalized disclosure view.
5. **In what reference?** Source CRS URI, axis order, dimension, horizontal/vertical datum, coordinate epoch, engineering frame, and transform pipeline.
6. **How well known?** Precision, accuracy, uncertainty, quality, method, and provenance.
7. **Who may learn it?** Marking, releasability, purpose, tenant, precision policy, and inference risk.
8. **How queried?** Bounding box, intersection, relation, distance, spatial-temporal combination, sort, paging, and extent aggregation.

Options were evaluated against standards fidelity, semantic expressiveness, CRS/geometry behavior, exact-source preservation, spatial and spatial-temporal queries, index maturity, transactions with the accepted resource graph, Rust/deployment fit, DDIL/edge suitability, testability, operational complexity, and recoverability.

Source-backed obligations are labeled **N/A/I**. The design synthesis is **P**. Exact DDL and thresholds are **D**. Unresolved profile/workload needs are **X**.

---

## 5. Geospatial Data-Category Inventory

### 5.1 Canonical Spatial Layers

| Layer | Purpose | Authority/retention | Required behavior |
|---|---|---|---|
| Exact source | Bytes/tree and source identifiers exactly as received | Evidence; content-addressed under IDR-SRV-019/021/025 | Never rewritten by normalization; access controlled |
| Parsed source spatial object | Lossless typed view of source coordinates/frame metadata | Derived from exact source but reproducible | Preserves ordering, dimensions, units, identifiers, nil/unknown states |
| Canonical spatial assertion | Encoding-neutral fact plus subject/role/time/evidence/policy | Authoritative when validation admits it | Does not erase unresolved CRS/frame/vertical facts |
| Canonical query projection | PostGIS geometry suitable for declared predicates | Rebuildable projection of admitted assertion | Explicit SRID; transform evidence; validity; source/assertion link |
| Generated representation | GeoJSON/SensorML/JSON/API response | Deterministic projection | Standards-correct CRS/order and policy view; no new truth |
| Derived spatial product | trajectory, footprint, bbox, generalized geometry, extent | Rebuildable/versioned unless explicitly supplied authoritative | Inputs, algorithm, version, time/watermark, uncertainty, policy scope |
| Operational cache | latest position, tiles, precomputed envelopes, search copy | Disposable projection | Freshness/watermark, policy scope, rebuild and invalidation |

### 5.2 Required Spatial Assertion Envelope

A detailed schema is deferred, but later designs must preserve at least:

- assertion identity and revision; subject identity/type; semantic role; source artifact/assertion and authority class;
- exact source CRS/reference-frame identifier; resolved horizontal CRS; axis order; coordinate order; dimension; units;
- vertical CRS/datum or explicit unknown; coordinate epoch where relevant; engineering/local frame and frame relationship;
- parsed source position/geometry/pose; canonical query geometry and SRID when transformation is safe;
- valid/phenomenon/result/receive/system time as applicable; latest-projection watermark and freshness classification;
- geometry validity state, transformation pipeline/version/grids, precision, accuracy, uncertainty, quality, and provenance;
- security marking, releasability/purpose/tenant scope, precision policy, derivation-policy join, and audit references.

Absence, unknown, withheld, invalid, unresolved-frame, and not-applicable are different states. They must not all serialize as a fabricated null-island point or global extent.

---

## 6. Resource-Family Spatial Requirement Matrix

The matrix uses `Auth` (authoritative supplied/canonical), `Derived`, and `Cache`. Candidate patterns are architectural, not table names.

| Data category | Related resource family | Source topic / source anchor | Geometry type | Static/time-varying | Authoritative/derived/cache | CRS/coordinate needs | Spatial query/index needs | Temporal interaction | Precision/uncertainty/provenance | Security/policy needs | Candidate storage pattern(s) | Downstream topic handoff | Notes/unresolved |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| System representative location | System | CSAPI P1 `/rec/system/location`, `/req/system/location-time`; IDR-015/018 | Usually Point; virtual systems may be MultiPoint/area | Current projection over static or changing facts | Auth assertion + Cache latest view | Source preserved; query projection CRS84-compatible; optional Z only with vertical meaning | GiST; bbox/geom against selected system geometry | Latest known unless snapshot requested; carry event/receive time | Required source, accuracy/uncertainty, freshness | High OPSEC; exact/generalized/withheld views | Versioned assertion + current projection in PostGIS | 027, 031, 033, 034, 039A, 042 | Do not overwrite history or equate with sampling location |
| System historical positions | System / location Datastream | CSAPI P2 dynamic data/system history; IDR-018/020 | Point samples | Time-varying append | Auth observations/assertions | Source frame/CRS and vertical metadata per sample/contract | Time + GiST/BRIN candidate; as-of/latest | Phenomenon/result/receive time | Per-sample uncertainty/provenance | Trajectory inference and retention sensitive | Timed samples; latest projection | 027, 029, 034, 043, 054 | Physical partition design deferred |
| Physical installation/site | System or Deployment | SensorML/CSAPI; IDR-021 | Point/area | Revisioned, often static | Auth assertion | Site CRS; vertical datum if elevation matters | Discovery intersection | Descriptive valid time | Survey method/accuracy | May be sensitive | Versioned spatial assertion | 028, 031, 039A | Different from present mobile-platform position |
| Virtual-system execution location | System | CSAPI P1 System Location guidance | Point/MultiPoint/area or absent | Revisioned/dynamic | Auth if supplied; sometimes Derived | CRS84 API projection | Optional discovery query | Snapshot/current semantics | Meaning/source must be explicit | Infrastructure-location sensitivity | Spatial assertion or absent | 033, 039A | Never substitute model coverage or sampling area |
| Deployment discovery area | Deployment | CSAPI P1 Deployment Features | Geometry/area; approximate allowed | Revisioned interval | Auth when declared; Derived only if labeled | Horizontal CRS; dateline-safe | GiST; bbox/geom | Deployment valid interval | Approximation/derivation/provenance | Operational area can be highly sensitive | Versioned PostGIS assertion | 028, 031, 033, 039A | Not system or sampling geometry |
| Derived deployment envelope | Deployment | Analysis from system/sampling relations | Polygon/MultiPolygon/bbox | Changes with members/time/window | Derived | Canonical query CRS and derivation rules | Discovery/cache index | Bound to member set and time window | Algorithm, inputs, watermark | Policy join of all inputs | Materialized derived product | 029, 042, 048, 054 | Must not silently replace declared area |
| Procedure geometry | Procedure | CSAPI P1 `/req/procedure/location` | None | Not applicable | None | None | Advanced geom excludes; Core bbox rule needs adapter decision | None | None | Avoid accidental location leak from related systems | No geometry column/value | 033, 050, 056 | Normatively non-spatial |
| Sampling point | Sampling Feature | CSAPI P1 Sampling Features | Point | Static or revisioned/dynamic | Auth | Source CRS/frame and optional vertical datum | GiST; bbox/geom | Valid time and observation relation | Sampling method/accuracy/provenance | Sampling location may reveal operation | Spatial assertion | 028, 031, 033, 039A, 053 | Keep link to ultimate FOI |
| Sampling curve / path | Sampling Feature | CSAPI P1 examples | LineString/MultiLineString | Static, revisioned, or derived from timed samples | Auth or Derived | CRS, interpolation, dateline behavior | GiST; intersection; length only with declared metric | May represent time-bounded trajectory | Sampling/derivation uncertainty | Path inference sensitive | Assertion or versioned derived geometry | 027, 034, 039A, 054 | GeoJSON has no per-vertex time semantics |
| Sampling surface/area | Sampling Feature | CSAPI P1; future P4 informative | Polygon/MultiPolygon | Static/revisioned/dynamic | Auth or Derived | CRS and topology validity | GiST exact intersection | Valid/phenomenon interval | Boundary accuracy/method | Area may require generalization | Spatial assertion | 031, 034, 053 | Holes/ring orientation fixtures required |
| Sampling solid/volume | Sampling Feature | SWE/OGC sampling concepts; P4 informative only | Solid/3D envelope or semantic parametric object | Static/revisioned | Auth source; 2D footprint Derived | Horizontal + vertical CRS/datum and units | 2D footprint baseline; 3D optional profile | Valid interval | Vertical uncertainty/provenance | Sensitive volume | Exact semantic object + query footprint | 034, 050, 053, 054 | Simple Features/PostGIS 2D predicates do not settle volumetric semantics |
| Specimen/material sample | Sampling Feature | CSAPI P1 concepts; P4 informative | Usually non-spatial or collection-site Point | Revisioned | Auth contextual assertion | Site CRS if provided | Optional discovery | Collection time distinct from specimen lifecycle | Collection method/provenance | Chain-of-custody/location controls | Relationship + optional assertion | 028, 031, 039A | Do not turn specimen identity into geometry |
| Ultimate feature-of-interest geometry | Generic Feature / System / external feature | CSAPI P1/P2 Sampling Feature relations; IDR-017 | Any supported feature geometry or none | Revisioned/dynamic | Auth local/external; Cache resolved copy | Source CRS and external-version identity | GiST only for admitted local projection | FOI version/time may differ from observation | External authority and resolution provenance | External-policy inheritance | Local assertion or versioned external cache | 028, 033, 043 | FOI can be abstract/non-spatial |
| Observation-associated location | Observation / Sampling Feature | CSAPI P2 observation relation; IDR-022 | Usually relation; sometimes result contains Point/Pose | Time-varying | Auth relation/result; Derived join | Bound by stream contract/source frame | Prefer join to SF; index extracted result only when profiled | Phenomenon/result/receive time | Per-result quality/uncertainty | Fine-grained sensitive history | Observation sample + relationship; optional query projection | 027, 031, 034 | Avoid unconditional geometry duplication |
| Geopose result | Observation | CSAPI P2 examples; SWE Common | Point plus orientation/frame | Time-varying | Auth observation | Horizontal/vertical CRS plus orientation reference frame | Point index if selected; pose predicates profile-specific | Observation time axes | Position and orientation covariance/quality | High sensitivity | Timed structured result + point projection | 027, 034, 039A, 053 | GeoJSON point cannot preserve orientation |
| Datastream spatial coverage | Datastream | CSAPI P2 SF associations; analysis | Geometry/extent | Changes as observations/SFs change | Derived | CRS84-compatible output extent; source links | Cached discovery extent | Window/as-of/watermark explicit | Input set and completeness | Policy-scoped; no global union leak | Materialized scoped extent | 027, 029, 033, 048 | Datastream itself has relationships, not necessarily intrinsic geometry |
| ControlStream target area | ControlStream / Sampling Feature | CSAPI P2 SF/FOI associations | Usually relation; optional profile geometry | Revisioned | Auth relationship; geometry Derived/profile | Target CRS depends on tasking contract | Relationship first; spatial query if advertised | ControlStream validity | Contract/provenance | Tasking areas sensitive | Relationship + optional spatial assertion | 028, 035, 039A | Do not infer target area from every command by default |
| Command target coordinate/area | Command | CSAPI P2 tasking schemas; SWE Common | Typed value, Point/Polygon, or other | Per command | Auth payload | Schema-bound CRS/frame/unit | Operational lookup only if profiled; not public feature geometry by default | Issue/schedule/execution time | Validation and request provenance | Strong authorization, purpose, audit | Exact command + structured components; optional protected projection | 029, 035-038, 039A | No universal geometry semantics |
| Feasibility region/trajectory | Feasibility result | CSAPI P2 examples | Polygon/trajectory or structured result | Request-time | Derived | Request CRS/frame + model assumptions | Usually returned result; cache only if safe | Valid-for interval/model snapshot | Algorithm/model/input uncertainty | Reveals capabilities | Protected derived artifact | 035-039A | Example does not create mandatory spatial schema |
| Platform trajectory | System / observations | Analysis; CSAPI sampling-curve example | LineString/MultiLineString, possibly M internally | Windowed dynamic | Derived from Auth samples unless supplied | CRS, interpolation, gap rules, dateline splitting | GiST + time/window; simplification levels | Per-vertex sample time retained separately | Algorithm/gaps/uncertainty/provenance | High inference risk | Samples authoritative; versioned derived trajectory | 027, 034, 039A, 042, 054 | Never infer straight motion across gaps without rule |
| Sensor pose/mount orientation | System/Component / SensorML | SensorML; IDR-021/022 | Position + rotation/matrix/quaternion | Static, revisioned, or dynamic | Auth | Engineering/local/reference frames and units | Not reducible to ordinary spatial predicate | Valid time/snapshot | Calibration/covariance/provenance | May expose capability geometry | Structured canonical assertion; optional point projection | 028, 034, 039A | Frame graph needed; cycles/unresolved frames invalid for transform |
| Footprint / field of view | System/Procedure/Sampling Feature | Sensor model analysis; future P4 viewing frustum informative | Polygon/MultiPolygon/frustum | Highly time/mode dependent | Auth if supplied, otherwise Derived | Pose, mount, procedure, terrain/target surface, CRS | Intersection/coverage after derivation | Bound to acquisition time/mode | Algorithm/model/terrain/uncertainty | Reveals collection capability | Versioned derived product | 027, 034-039A, 054 | P4 draft cannot be treated as approved obligation |
| Resource bbox | Any spatial Feature | GeoJSON RFC 7946 / OAPIF | 2D/3D bbox array | Derived per representation/revision | Derived | Output CRS/order and antimeridian rules | Cheap output/candidate cache | Same snapshot/policy as geometry | Derivation/version | Must reflect visible geometry only | Computed or cached with assertion | 029, 033, 050 | West may exceed east across antimeridian |
| Collection spatial extent | Collection | OAPIF collection metadata | One or more bboxes | Derived/materialized | Derived Cache | CRS identifier and dimensions | Discovery, not authoritative member geometry | Watermark/as-of scope | Completeness and derivation provenance | Compute after policy; may be suppressed | Policy-scoped materialized view | 029, 033, 039A, 048 | Empty/unknown is not world extent |
| Generalized/redacted geometry | Any protected spatial resource | IDR-019 plus this report | Same/lower-dimensional or area envelope | Derived per policy/request | Derived | CRS plus generalization parameters | Separate scoped index/cache if materialized | Same source snapshot and policy version | Algorithm, tolerance, uncertainty, source link | Purpose/releasability and anti-differencing | Protected derived projection | 039, 039A, 042, 048 | Never overwrite exact authority |
| Import/export geometry artifact | External dataset | GDAL/GeoJSON/SensorML | Any admitted format | Batch/revisioned | Exact Auth source + Derived canonical | Explicit source CRS and controlled transform | Quarantine before indexing | Import transaction/time | Tool/driver/version/hash/provenance | Untrusted file and network risk | Content-addressed artifact + staged rows | 028, 031, 039, 046 | GDAL stays outside request-critical trusted core |

---

## 7. Geometry Type, CRS, Coordinate, Precision, Uncertainty, and Provenance Findings

### 7.1 Geometry and Role Rules

The full profile should admit the RFC 7946 / Simple Features families needed by approved resources: Point, MultiPoint, LineString, MultiLineString, Polygon, MultiPolygon, and bounded GeometryCollection use. GeometryCollection is preserved when supplied but avoided as a default generated shape when a single or multi-geometry type suffices. Procedure geometry is prohibited. **[N/P]**

Geometry validity is checked at admission and after every transform/derivation. Structural validation, finite/range checks, ring closure/orientation for interchange, `ST_IsValid`/reason, declared type/dimension, SRID, and domain-role checks are separate gates. Database acceptance of an invalid geometry is not semantic validity. Repair must create a derived artifact with tool/version and change evidence; it must not silently rewrite the exact source. **[I/P]**

### 7.2 CRS84, EPSG Identifiers, and Axis Order

RFC 7946 coordinates are WGS 84 longitude then latitude in decimal degrees, equivalent to OGC CRS84; an optional third position is height in metres relative to the WGS 84 ellipsoid. OGC API - Features Core similarly defaults output to CRS84/CRS84h. The Core does not require alternate response CRS. **[N]**

`EPSG:4326`, PostGIS SRID 4326, and CRS84 are often operationally mapped to the same numeric coordinate pair, but their identifier/axis conventions are not interchangeable evidence. Glaux adapters must know the wire order and source definition. A storage column with SRID 4326 uses X=longitude/Y=latitude by Glaux convention; this convention must be verified at adapter tests and never inferred merely from an SRID integer. **[P]**

For the first full profile, generated GeoJSON and default `bbox` input/output use CRS84/CRS84h. Alternate source CRS is retained. Alternate output and `bbox-crs` are disabled unless a later profile adopts OGC API - Features Part 2, advertises it, has deterministic transform resources, and passes conformance/interoperability tests. **[P/D]**

### 7.3 Transformation and Reference Evidence

Every admitted transform records source and destination CRS identifiers, axis mapping, dimensional treatment, transformation engine and version, selected pipeline/operation, grid/resource versions, coordinate epoch when relevant, execution time, and warnings/estimated accuracy. `ST_SetSRID` changes metadata only; `ST_Transform` changes coordinates. Relabeling a geometry to “fix” a source CRS is prohibited unless the coordinates were already in that CRS and the correction is an audited metadata repair. **[I/P]**

Unknown or unresolved source CRS/frame is a valid evidence state but not a queryable canonical geometry. Such records remain quarantined or non-spatial until resolution. Network-fetched transform grids are not allowed implicitly in DDIL or reproducible builds; required CRS databases/grids are versioned deployment artifacts with hashes. **[P]**

### 7.4 Vertical, Z/M, Pose, and Time

Z storage does not make standard PostGIS topological predicates three-dimensional. The baseline spatial discovery surface is 2D. Six-value `bbox` and 3D predicates are enabled only where horizontal plus vertical CRS/datum, units, axis direction, and semantics are resolved and tested. Depth, orthometric height, ellipsoidal height, flight level, and distance above a local surface must not be merged. **[N/I/P]**

M may support an internal derived trajectory representation, but no API meaning is assigned here. Authoritative per-sample timestamps remain explicit temporal fields; a LineStringM cannot replace them. Pose and orientation stay structured, with reference/local-frame semantics preserved. **[P/D]**

### 7.5 Precision, Accuracy, Uncertainty, and Provenance

Decimal formatting precision is not positional accuracy. Glaux stores source numeric lexical evidence when fidelity requires it, parsed numeric value, declared significant digits/resolution where available, accuracy/uncertainty/quality assertions, and provenance. Transform-derived error and generalization tolerance remain distinguishable from sensor measurement uncertainty. **[P]**

Generated output must not imply greater confidence than the source. Rounding for representation is deterministic and policy-aware. Buffering an uncertain point into an area is a derived interpretation, not automatic truth; any uncertainty geometry carries method, confidence meaning, inputs, and time. **[P]**

### 7.6 Antimeridian and Polar Behavior

RFC 7946 recommends cutting antimeridian-crossing geometries for interoperability. Its bounding boxes may validly have west longitude greater than east longitude. Glaux therefore parses such boxes as a wrapped region—typically two query envelopes—not as invalid or as the 355-degree complement. Polar caps and longitude normalization require explicit fixtures. Derived trajectories must split at the antimeridian for GeoJSON generation without inventing an observation at the split. **[N/P]**

---

## 8. Spatial Query, Filtering, Sorting, Pagination, and Extent Findings

### 8.1 Normative Baseline

Applicable OGC API - Features collections accept `bbox` as four or six numbers. The default is CRS84 or CRS84h; only geometries intersecting the box are returned under the applicable collection semantics. Part 1 notes that a feature with multiple geometry properties leaves the selected extent geometry to the server. Glaux fixes that choice per resource family and capability declaration rather than varying it by query plan. **[N/P]**

There is a significant inheritance nuance. OGC API - Features Core says non-spatial features match `bbox`, while CSAPI Advanced Filtering’s `geom` clause says resources with no spatial geometry are excluded. Glaux must implement each parameter’s own published rule and test Procedures/non-spatial FOIs explicitly; it must not reuse one SQL predicate blindly for both. **[N/P]**

When CSAPI Advanced Filtering is advertised, `geom` accepts valid WKT and selects features whose chosen spatial geometry intersects the filter geometry. Unparseable WKT, unsupported dimension/type, out-of-domain ordinates, or unresolved CRS yields a stable 4xx problem; it never silently becomes an unfiltered query. The advanced class does not make contains, within, distance, or CQL2 mandatory. **[N/P]**

### 8.2 Optional Query Capabilities

`intersects`, `within`, `contains`, proximity/radius, distance sort, trajectory-window, and semantic-spatial combinations are useful Glaux candidates but remain extensions or later-profile capabilities until their grammar, CRS, units, empty-geometry behavior, and conformance advertisement are selected. CQL2 should be evaluated by IDR-SRV-051 rather than approximated through ad hoc query parameters. **[D]**

Proximity must state geometry/geography calculation, units, spheroid/planar model, maximum radius, and tie behavior. Distance sorting requires a deterministic secondary key. A projected planar distance is valid only in a suitable declared CRS; a geodesic distance path can use PostGIS geography or an explicit geodesic function but must be benchmarked. **[I/P/D]**

### 8.3 Combined Filtering and Policy

Spatial predicates combine with temporal, resource, relationship, semantic, lifecycle, and authorization predicates using typed grammar and explicit AND/OR rules. Authorization and disclosure selection occur before result count, `numberMatched`, extent, sort, page construction, or link generation. Query validation occurs before expensive execution and sets limits for geometry size, vertex count, nesting, WKT complexity, radius, time span, and result limit. **[P]**

For a moving System, `bbox` at the present endpoint applies to the selected current/snapshot geometry, not “ever visited this area,” unless a trajectory-history capability explicitly says otherwise. Spatial-temporal observation queries apply both predicates to the same sample/associated geometry according to their declared join semantics. **[P]**

### 8.4 Sorting and Pagination

Default ordering remains a stable resource ordering, not database physical order. Spatial filtering must not destabilize opaque keyset pagination. Cursor state binds at least route/family, canonicalized spatial filter and CRS, geometry-role rule/version, remaining non-spatial filters, sort plus unique tie-breaker, snapshot/as-of or dataset watermark, tenant/principal/policy scope or safe equivalent, and expiration/version. **[P]**

Changing geometry, policy, or projection state between pages cannot be allowed to reveal duplicates, omissions, or hidden rows without a documented consistency model. IDR-SRV-029 owns transaction/snapshot mechanics; IDR-SRV-051 owns public grammar and token behavior. Offset paging is not selected. **[D]**

### 8.5 Extents and Counts

Resource bbox and collection extent are derived products. They are calculated from the same visible geometry role, policy scope, valid/system-time view, and CRS used for the response. An empty collection has unknown/empty extent according to the representation contract—not a fabricated world box. A partially synchronized cache reports its watermark/completeness internally and must not assert complete authoritative extent. **[P]**

Large collections may use incrementally materialized extents, but deletion, correction, policy change, backfill, and antimeridian behavior make naive min/max expansion incorrect. Recompute and repair procedures, derivation version, contributor policy scope, and staleness metrics are mandatory. **[P/D]**

---

## 9. Spatial Indexing and Performance Implications

### 9.1 Index Baseline

Use GiST on canonical PostGIS query geometry as the full-profile default. Pair it with B-tree indexes for resource identity, family/role, tenant/policy partition keys, lifecycle, and temporal joins. A typical query should narrow policy/family/time and spatial candidates before exact predicate evaluation, but exact ordering is optimizer-dependent and must be proven with representative plans. **[I/P]**

Spatial index operators commonly use bounding boxes to produce candidates; exact `ST_Intersects` or the chosen predicate confirms results. Returning bounding-box candidates without exact recheck is prohibited when the contract promises geometric intersection. Prepared/parameterized SQL and typed predicate allowlists prevent SQL/WKT injection. **[I/P]**

### 9.2 Conditional Indexes

| Candidate | Appropriate use | Gate |
|---|---|---|
| Partial GiST | Current/admitted/non-null or common family/role subset | Demonstrate selectivity and manageable update cost |
| Functional GiST on `ST_Transform` | Repeated supported query CRS | Only for adopted CRS capability and immutable reproducible transform assumptions |
| SP-GiST | Very large point-heavy or partitioned distributions | Benchmark against GiST with real skew and writes |
| BRIN spatial | Huge physically spatially correlated append tables | Benchmark correlation, false positives, vacuum/maintenance, and time partition interaction |
| Covering B-tree companions | Tenant, role, time, state, stable cursor | Explain plans must show benefit; avoid index explosion |
| Materialized envelope/subdivision | Very complex polygons repeatedly queried | Preserve exact geometry; derivation/version and exact semantic equivalence tests |

### 9.3 Geometry/Geography Choice

Use PostGIS `geometry` with explicit SRID for authoritative/query projections by default. It supports the broad operator/function/index surface and works naturally for CRS84-compatible feature publication. Use `geography` selectively for well-defined geodesic distance/area operations on lon/lat data, not as a universal duplicate authority. If both are materialized, one is declared derived, rebuildable, and parity-tested. **[I/P]**

### 9.4 Performance Corpus

Benchmarks must vary resource family; points versus complex polygons; null/non-spatial rows; vertex count; invalid inputs; dateline/poles; spatial clustering/skew; moving-position rates; late data; relationship fan-out; temporal selectivity; policy selectivity; cache warmth; concurrency; page depth; transform CRS; derived extent maintenance; and generalized views.

Required measurements include p50/p95/p99 latency, throughput, examined/index/heap rows, false-positive/recheck ratios, planning time, memory/temp spill, WAL and write amplification, index size/build/reindex time, vacuum/analyze effects, lock behavior, cold start, restore/rebuild, and correctness under concurrent updates. Thresholds belong to IDR-SRV-054. **[D]**

### 9.5 Denial-of-Service Controls

Set bounded request sizes, coordinate/vertex counts, nesting, WKT tokens, GeometryCollection members, transform choices, buffer/radius, time spans, and response limits. Use statement timeouts/resource budgets and cancellation. Expensive derived operations should run as controlled jobs rather than anonymous synchronous queries. Cache keys include policy and normalized geometry; adversarial high-cardinality shapes must not exhaust cache/storage. **[P]**

---

## 10. Geospatial Storage Option Evaluation

### 10.1 Evaluation Rubric

| Criterion | Weight | Meaning |
|---|---:|---|
| Standards and semantic fidelity | 18 | Geometry/CRS/type behavior without flattening roles |
| Transactional integration | 16 | Atomic join to identities, relationships, time, policy, provenance |
| Query/index capability | 15 | Exact and spatial-temporal predicates, mature plans/indexes |
| Source/transform fidelity | 10 | Exact evidence, CRS/frame preservation, reproducibility |
| Security/policy correctness | 10 | Scoped queries/derivations and auditable disclosure |
| Operations and recoverability | 9 | Migration, backup, restore, diagnostics, repair |
| Rust/developer/test fit | 8 | Typed access, containers, CI, golden fixtures |
| Edge/DDIL fit | 7 | Offline packaging, resource use, synchronization boundary |
| Portability/exit path | 4 | Standard formats and adapter boundaries |
| Complexity/cost | 3 | Dependencies and specialist burden |

### 10.2 Option Comparison

| Option | Strengths | Costs/risks | Full-profile disposition |
|---|---|---|---|
| PostgreSQL + PostGIS relational-hybrid | Mature Simple Features/CRS/index functions; transactions with accepted graph/time/policy; one backup/migration boundary | Operational extension; specialist tuning; 2D defaults require vertical discipline | **Select as authoritative baseline** |
| PostgreSQL JSONB/GeoJSON only | Simple exact-ish document retention; flexible | Weak typed CRS/role constraints; expensive/error-prone predicates; no substitute for spatial indexes; representation truth risk | Exact/canonical document support only; **reject as spatial authority** |
| Separate search/geospatial service | Fast text/geo search and scaling options | Dual-write, lag, policy parity, backup/sync complexity; representation-dependent truth observed | Optional rebuildable projection only after evidence |
| SpatiaLite/SQLite | Portable single-file embedded spatial SQL; useful reduced edge/test bundle | Single-writer and capability/performance divergence; extension packaging; no full PostgreSQL parity | Conditional explicit reduced profile |
| DuckDB Spatial | Excellent local analytical/export workflows; broad format/transform functions | Analytical engine, not concurrent transactional authority; bundled PROJ database may differ | Analytical sidecar/export only |
| Rust `geo`/`geo-types`/`geojson` in memory | Safe domain algorithms/serialization; testable; no database round trip for bounded work | `geo-types` coordinates are 2D; application index/state consistency and algorithm parity burden | Supporting library layer, never independent authority |
| GDAL/OGR | Broad import/export and conversion ecosystem | Native dependency and very large parser/driver/network attack surface | Sandboxed/quarantined import/export worker only |
| Flat GeoPackage/files | Portable transfer/inspection | Concurrency, relationship/policy/version/query limitations | Export/evidence bundle, not live authority |

### 10.3 Selected Architecture

The full profile uses PostgreSQL/PostGIS for admitted canonical spatial assertions and query projections, PostgreSQL relationships/temporal/policy/provenance for context, and the IDR-SRV-025 artifact interface for exact sources and large derived products. Application ports expose semantic operations—not PostGIS SQL types—to keep tests, reduced profiles, and later migration possible. **[P]**

Rust crates are chosen later and wrapped behind validated codecs and geometry services. `geojson` can serialize RFC 7946 structures, but convenience serialization that loses foreign members or source lexical form cannot satisfy exact-source preservation. `geo-types` supplies 2D primitives; it must not be mistaken for CRS, Z, pose, or uncertainty semantics. Database and application algorithms require parity fixtures. **[I/P/D]**

### 10.4 Deployment Profiles

- **Full/reference:** PostgreSQL 18 + released PostGIS 3.6 line; complete declared Glaux spatial capability.
- **Development/CI:** containerized same engine/version for conformance and migration tests; lightweight mocks only for unit tests.
- **Reduced edge:** SpatiaLite or constrained PostgreSQL only after a published capability profile, format/version pin, resource limits, and equivalence tests. Unsupported operations fail honestly.
- **Analysis/export:** snapshot to DuckDB/GeoParquet/GeoPackage or controlled files; never writable authority.
- **Import/export:** GDAL in an isolated worker with driver allowlist, network disabled by default, quotas, patched dependencies, and staged validation.

---

## 11. Spatial-Temporal and Dynamic-Location Implications

### 11.1 Authoritative Samples and Latest Projection

Moving-location samples are append-oriented observations or spatial assertions with explicit phenomenon/effective, result, receive, and system time. The current System geometry is a projection selected by an explicit rule from eligible samples/assertions—not a mutable field that erases prior positions. **[P]**

The projection records source sample, selection rule/version, sample time, receive time, calculation time, freshness threshold/result, uncertainty, provenance, policy, and watermark. Out-of-order arrival may change historical truth and possibly the latest projection according to phenomenon-time and trust rules; ingestion must not assume arrival order equals domain order. **[P/D]**

For `asOf`, Glaux evaluates the resource revision and location assertion valid at the requested snapshot under IDR-SRV-018. A future-received correction must not appear in a transaction-time historical view unless the query semantics permit it. IDR-SRV-027/029 determine physical access paths and snapshot mechanics. **[D]**

### 11.2 Trajectories

Authoritative points remain primary. A trajectory declares sample set/window, ordering axis, interpolation model, gap threshold, coordinate transformation, dateline treatment, simplification, uncertainty treatment, builder version, and watermark. No line is drawn across missing/stale intervals unless explicitly represented. **[P]**

Windowed trajectory queries need time pruning before/with spatial pruning. A derived LineString or LineStringM may accelerate discovery/display, but exact sample retrieval remains available and the line never becomes proof of motion between observations. **[P]**

### 11.3 Footprints and Coverage

A footprint derived from platform position, attitude, sensor mounting, procedure/mode, field of view, target surface/terrain, refraction/model assumptions, and time is a provenance-rich computation. Missing orientation/frame/terrain prevents authoritative-looking output. Approximate fallback is separately labeled and policy-controlled. **[P]**

### 11.4 Observations, FOI, and Sampling Features

Observation location should normally resolve through its Sampling Feature/FOI relationships. If the result itself is a location/geopose, that value is an observed property with its own time/quality/contract and may also feed a current projection. Queries must say whether they intersect sampling geometry, ultimate FOI geometry, observed location result, or a derived footprint. **[N/P]**

---

## 12. SensorML, SWE Common, and Semantic Integration Implications

### 12.1 SensorML

Apply IDR-SRV-021’s exact/parsed/canonical/generated/evidence layers. SensorML position, location, pose, component placement, and deployment information can supply spatial assertions, but parsing never discards original identifiers, reference frames, axis definitions, units, or extension content. Inheritance/reference resolution records its graph and cannot silently depend on network availability. **[P]**

Frame relationships form a typed, temporally valid graph. Transformation requires an acyclic resolved path appropriate to the assertion time. A sensor’s local mount offset is not a geographic position until combined with platform pose and frame transforms; generated geometry carries all derivation inputs. **[P]**

### 12.2 SWE Common

Apply IDR-SRV-022’s immutable contract/component-path baseline. A SWE `Vector` may declare `referenceFrame` and `localFrame`, and its components may be spatial coordinates, velocity, orientation parameters, or other vector values. Semantic definitions and component roles—not shape alone—determine interpretation. A `Matrix` may encode rotation/covariance/calibration and must not be flattened into geometry. **[N/P]**

Location/geopose extraction from observations is enabled only by an immutable validated stream contract whose component paths, definitions, units, frame, nil/optional behavior, and encoding are resolved. Schema revision is never retroactively applied to older samples. **[P]**

### 12.3 Semantic Bindings and Units

Apply IDR-SRV-024’s contextual property-role bindings and UCUM-centered strategy. Latitude, longitude, easting, northing, height, depth, heading, yaw/pitch/roll, covariance, radius, and distance are roles bound to definitions, frames, and units. Same numeric shape or definition does not authorize silent conversion between roles. **[P]**

Coordinate transform units come from the CRS operation. Measurement/result units and command units remain governed by their contracts. Silent command-coordinate or unit conversion is prohibited unless the adopted tasking profile explicitly authorizes it and the transform is surfaced/audited. **[P/D]**

### 12.4 Representation Equivalence

GeoJSON, SensorML, JSON, map, and export views derive from one canonical identity/assertion set. Format-inapplicable detail may be omitted but cannot select a different resource population. Tests compare identity, relationships, selected geometry role, time, bbox, and shared properties across formats. Exact SensorML may retain richer pose/frame data than GeoJSON; this is a declared representational difference, not separate truth. **[P]**

---

## 13. Security, Policy, Releasability, Precision-Reduction, and Audit Implications

### 13.1 Policy Decision Order

1. Authenticate and resolve tenant/principal/purpose/context.
2. Select candidate resource/assertion versions within policy partitions.
3. Evaluate access to exact spatial fact and permissible disclosure product.
4. Choose exact, generalized, area-only, stale/last-known, or withheld representation.
5. Apply spatial/non-spatial predicates to that contractually defined view.
6. Compute sort, counts, extents, pagination, links, and cache entries from the authorized view.
7. Emit bounded audit/evidence without logging sensitive coordinates unnecessarily.

Database row security may reinforce isolation but cannot alone choose semantic precision products or prevent aggregate inference. Application and database policy need deny-path parity tests. **[P/D]**

### 13.2 Generalization and Suppression

Generalization is a derived artifact with source assertion, policy/version, method, tolerance/cell size, output geometry/CRS, time, uncertainty effect, and provenance. Deterministic coarsening is generally safer for consistency, but repeated queries across policies/times may still enable differencing. Random jitter is not automatically safer and can create impossible positions; any use requires bounded privacy/security analysis. **[P/X]**

Suppression states distinguish no geometry, unknown geometry, geometry withheld, and resource hidden. Public errors, counts, `numberMatched`, collection extents, nearest sorting, page boundaries, and timing must not reveal which state applies when policy forbids it. **[P]**

### 13.3 Derived Products and Caches

The policy of a trajectory, footprint, deployment union, or extent is at least the restrictive join of its inputs plus output-specific policy. Policy changes, source correction/deletion, or derivation-version changes invalidate affected products. Cross-tenant or cross-compartment indexes/materialized views require structural isolation or proven predicate enforcement. **[P]**

### 13.4 Audit

Audit records capture requester/context reference, normalized spatial operation and coarse region identifier or protected payload reference, time window, result class/count disclosure, policy decision/version, precision product, data/derivation watermark, outcome, and correlation ID. Raw precise geometry is omitted from ordinary logs; privileged forensic access to protected query payloads is separately controlled and retained. **[P/D]**

---

## 14. DDIL, Cache, Synchronization, and Last-Known-Position Implications

### 14.1 Autonomous Operation

A disconnected node carries a versioned local CRS/PROJ resource package, required grids, schemas/vocabularies, admitted spatial assertions, policy state, and rebuildable indexes sufficient for its declared profile. It must not require network CRS resolution, remote tiles, or an upstream spatial service for correctness. **[P]**

“Last known” always includes source, event time, receive time, age/freshness, uncertainty, local knowledge watermark, and whether newer remote state may exist. A node disconnected for hours does not relabel its cached position “current”; it may truthfully present it as last-known/stale according to policy. **[P]**

### 14.2 Synchronization Boundary

PostGIS replication is not Glaux multi-writer semantic synchronization. Spatial sync operations retain stable assertion identity, resource/relationship dependencies, source hash, CRS/frame artifacts, time/provenance/policy, tombstones, and derivation status. Derived geometries and indexes normally rebuild locally; exact sources and authoritative assertions synchronize according to policy. **[P/D]**

Conflicts cannot be resolved by “latest received coordinate wins.” Two positions may refer to different phenomenon times, sources, accuracy, frames, or authorities. IDR-SRV-043 must preserve both assertions, apply authority/time/trust rules, produce a conflict/evidence record when unresolved, and rebuild current/trajectory/extent projections. **[D]**

### 14.3 Cache Rules

Cache keys include canonical query geometry/hash, CRS, role selection, time/as-of window, non-spatial filters, sort/page state, policy/principal scope or safe partition key, projection/derivation version, and data watermark. Entries have freshness and completeness state. Negative results and empty extents are policy-scoped and must not become universal. **[P]**

Spatial caches are never the sole copy of exact source or authoritative assertions. Restart, restore, migration, policy change, transform-grid update, and synchronization reconciliation tests must prove deterministic rebuild or explicit invalidation. **[P]**

---

## 15. Fixture, Conformance, Performance, and Interoperability Test Implications

### 15.1 Canonical Fixture Families

| Fixture group | Required cases |
|---|---|
| Geometry types | null/absent, Point/MultiPoint, line/multiline, polygon/multipolygon with holes, bounded GeometryCollection, empty where supported |
| Validity | unclosed/self-intersecting rings, wrong orientation, duplicate vertices, NaN/Infinity, out-of-range latitude, huge/nested input, repaired derivative |
| CRS/order | CRS84 lon/lat, deliberately swapped pair, projected source, unknown CRS, bad SRID, `ST_SetSRID` versus transform, missing grid, coordinate epoch |
| Vertical | 2D, ellipsoidal height, orthometric height, depth, unknown datum, six-value bbox, Z retained but ignored by 2D predicate |
| Global edges | antimeridian point/line/polygon/bbox, poles, ±180, ±90, degenerate bbox, world extent |
| Resource roles | System latest/history, Deployment approximate area, Procedure no geometry, SF point/curve/surface, non-spatial FOI, observation geopose |
| Dynamics | ordered/out-of-order samples, duplicate, correction, stale, gap, trajectory split, as-of/transaction-time view |
| Derivation | footprint inputs, trajectory, deployment union, collection extent, generalized view, invalidation and rebuild |
| Policy | exact/generalized/withheld/hidden; count/extent/page/distance anti-leak; cross-tenant negative tests |
| DDIL | offline transform resources, stale last-known, divergent assertions, tombstone, resync and projection rebuild |

Every fixture records source, expected admission state, canonical assertion, query projection, expected generated representation, expected diagnostics, provenance, and policy. Exact bytes and semantic expectations are both retained. **[P]**

### 15.2 Conformance and Contract Tests

- OGC API - Features `bbox`: four/six numbers; CRS84/CRS84h; hit/miss/boundary/degenerate/dateline; invalid ranges; relevant non-spatial behavior.
- CSAPI `geom` when advertised: valid WKT types; exact intersection; no-geometry exclusion; malformed/oversized/unsupported input; parameter combination.
- Resource rules: System latest/snapshot location, Procedure no geometry, Deployment discovery geometry semantics, Sampling Feature relations/types.
- GeoJSON: coordinates/order, ring and antimeridian generation, bbox dimensionality, common identity/count equality across representations.
- Negative filter controls: miss must differ from known hit; malformed filter must never return unfiltered `200`.
- Advertised capability truth: conformance, OpenAPI parameter/schema, runtime, errors, and test manifest agree.

### 15.3 Database and Algorithm Tests

Run integration tests on the pinned production PostgreSQL/PostGIS combination. Assert `ST_SetSRID` does not transform; transform round trips within declared tolerance; invalidity reasons; index and sequential plans return identical sets; 2D versus 3D behavior; geography versus geometry distance; antimeridian split; concurrent page/update semantics; migration/restore; and Rust/PostGIS algorithm parity on golden geometries. **[P]**

### 15.4 Performance and Resilience

IDR-SRV-054 should produce small correctness, representative load, adversarial stress, long-running soak, failover/restore, cache rebuild, and DDIL-resync lanes. Plans and results record database/PostGIS/PROJ versions, schema/index/statistics state, fixture distribution, hardware/container limits, cache state, concurrency, policy mix, query hashes, and correctness oracle. A faster incorrect bbox or policy leak is a failure. **[P/D]**

### 15.5 Interoperability

Test CSAPI Explorer, the Glaux web/mobile clients, at least one generic OGC API Features client, QGIS, and external CSAPI clients through black-box request/response scripts. The compatibility matrix records client/version, discovery URL, advertised classes, media selector, supported resource families, spatial/non-spatial behavior, bbox/geom/limit/paging, nested-field loss, authentication, proxy/public-origin behavior, and raw evidence. **[P]**

Generic GIS export is a projection. Procedures and abstract FOIs may be non-spatial; nested SensorML/SWE content may not map to flat attributes. Glaux must document these limits rather than fabricate points, flatten away semantics, or claim blanket compatibility. **[P]**

---

## 16. Downstream Topic Handoff Matrix

| Topic | Handoff from IDR-SRV-026 | Required outcome |
|---|---|---|
| IDR-SRV-027 | Timed position samples authoritative; latest/trajectory derived; GiST/BRIN/SP-GiST benchmark candidates | Partition, retention, late/corrected data, latest/as-of, Timescale gate |
| IDR-SRV-028 | Spatial assertion/source/query/derived layers; CRS/frame artifacts | Physical metadata/document/geospatial schema and artifact thresholds |
| IDR-SRV-029 | Atomic assertion/resource/provenance/outbox updates; cursor snapshot and extent invalidation | Transaction, idempotency, concurrency, derived-product consistency |
| IDR-SRV-030 | PostGIS/PROJ/index/version pin and rebuild/rollback needs | Migration, compatibility, restore, index rebuild strategy |
| IDR-SRV-031 | Staged CRS/frame/validity admission and exact-source preservation | Write/import pipeline and quarantine mechanics |
| IDR-SRV-033 | Family-specific geometry role; bbox/geom distinction; extent/count/page rules | Feature-resource read semantics and API mapping |
| IDR-SRV-034 | Observation location vs SF/FOI; geopose; latest projection; trajectory | Dynamic data and status update semantics |
| IDR-SRV-035-038 | Protected coordinate/area/pose commands and derived feasibility/footprint results | Tasking schemas, validation, authorization, lifecycle, result semantics |
| IDR-SRV-039/039A | Exact/generalized/withheld products and inference channels | Security controls, releasability, precision, audit policy |
| IDR-SRV-042 | Offline CRS resources, stale last-known, scoped caches | DDIL semantic states and behavior |
| IDR-SRV-043 | Assertion-aware conflicts; authoritative sync, derived rebuild | Synchronization protocol and conflict model |
| IDR-SRV-044/045 | Full/reduced/analysis/import profiles; spatial service boundary | Deployment topology and modular service design |
| IDR-SRV-048 | Query/index/extent/cache/transform observability without coordinate leakage | Metrics, health, safe logs, rebuild signals |
| IDR-SRV-050/051 | CRS84 baseline, possible Features Part 2/CQL2 gates, typed spatial grammar | Conformance profile and query contract |
| IDR-SRV-053 | Canonical spatial fixture groups and evidence fields | Curated reproducible fixture/golden corpus |
| IDR-SRV-054 | Workload dimensions, plan metrics, candidate index comparisons | Performance objectives and benchmark results |
| IDR-SRV-056 | Bounded QGIS/OAPIF/CSAPI client claims | Versioned external-client interoperability matrix |
| IDR-SRV-057 | Adopted geospatial decisions and unresolved profile gates | Final synthesis and implementation sequencing |

---

## 17. Recommendations

1. **Adopt PostgreSQL/PostGIS as full-profile spatial authority.** Keep identity, role, time, relationships, policy, provenance, and geometry in one transaction boundary. Priority: High.
2. **Model versioned spatial assertions, not a universal location field.** Preserve subject, role, time, authority, CRS/frame, vertical semantics, uncertainty, provenance, and policy. Priority: High.
3. **Maintain exact-source, canonical-assertion, query-projection, generated-view, and derived-product separation.** No representation-specific truth. Priority: High.
4. **Use CRS84/CRS84h for default approved API behavior.** Preserve alternate source CRS; gate alternate output CRS on explicit OGC API Features Part 2 adoption and tests. Priority: High.
5. **Prohibit metadata relabeling as reprojection.** Record deterministic transform pipelines, resources, versions, accuracy, and failures; quarantine unresolved frames. Priority: High.
6. **Treat the default discovery surface as 2D.** Enable vertical/3D behavior only with resolved datum, units, semantics, profile, and fixtures. Priority: High.
7. **Implement `bbox` and advertised CSAPI `geom` as distinct typed contracts.** Include their different no-geometry behavior, WKT parsing, dateline handling, and negative controls. Priority: High.
8. **Use GiST by default and benchmark conditional index/materialization options.** Exact predicates recheck index candidates; no index choice is accepted without representative correctness/load evidence. Priority: High.
9. **Keep moving-position samples authoritative and latest/trajectory/footprint/extent products derived.** Carry time, freshness, uncertainty, algorithm, watermark, provenance, and policy. Priority: High.
10. **Apply policy before geometry selection, predicates, counts, extents, sorting, paging, caching, and export.** Test inference channels and derived-policy invalidation. Priority: High.
11. **Keep reduced and analytical paths honest.** SpatiaLite is a capability-limited edge option; DuckDB is analytical; Rust geometry crates support the service; GDAL is isolated import/export. Priority: Medium.
12. **Make the spatial fixture and benchmark corpus a first-class deliverable.** Include family semantics, CRS/order, Z/vertical, dateline/poles, dynamics, policy, DDIL, invalid/adversarial shapes, and client black-box tests. Priority: High.

---

## 18. Risks, Constraints, and Open Questions

### 18.1 Risks and Controls

| Risk | Consequence | Control/owner |
|---|---|---|
| Axis-order or mislabeled CRS | Plausible but wrong positions/queries | Typed adapters, transform evidence, swapped-axis fixtures; 031/053 |
| Flattening all space to System Point | Broken deployment/SF/FOI/pose/trajectory semantics | Spatial roles and family matrix; 028/033/034 |
| Treating Z as 3D truth | Wrong altitude/depth intersection | 2D baseline; vertical-profile gate; 050/053 |
| Naive antimeridian bbox | Wrong hemisphere/extent | Wrapped-region parsing and global fixtures; 033/053 |
| Representation-specific spatial truth | Empty/different client populations | One assertion set and cross-format equality tests; 023/056 |
| Derived trajectory/extent presented as authority | False motion/coverage claims | Provenance, watermark, authority labels; 027/048 |
| Precise spatial disclosure/inference | Operational harm | Policy-before-query/aggregate and anti-differencing tests; 039A |
| Index-only approximate match | False positives | Exact-predicate recheck and parity tests; 054 |
| Index proliferation/poor skew | Write/space cost and unstable latency | Representative plans and workload gates; 054 |
| Native import parser exposure | RCE/DoS/network access | Isolated GDAL worker, allowlist, quotas, patching; 039/046 |
| DDIL stale state shown as current | Unsafe decisions | last-known freshness/watermark semantics; 042 |
| Divergent local assertions overwritten | Lost evidence | assertion-aware synchronization/conflict records; 043 |

### 18.2 Constraints

- The controlled AEP source cannot be redistributed; precise profile constraints not present in accepted findings remain unresolved.
- Approved CSAPI Parts 1 and 2 control baseline behavior. Draft Part 3 and Part 4 may inform fixtures/handoffs but create no obligation here.
- The full-profile authority remains the IDR-SRV-025 PostgreSQL/PostGIS relational-hybrid design.
- This report defines semantic/storage direction, not physical DDL, performance thresholds, or implementation authorization.
- Category E remains sequential; IDR-SRV-027 is not started by this report.

### 18.3 Open Questions and Owners

| Question | Why unresolved | Owner/gate |
|---|---|---|
| Exact AEP horizontal/vertical CRS and precision rules | Controlled/profile decision not safely inferable | Project lead/profile evidence; 050 |
| Alternate response CRS / `bbox-crs` support | Optional Features Part 2 capability | 050/051 decision and conformance proof |
| Which extra predicates/CQL2 are first-release scope? | Not mandatory under selected approved classes | 050/051 |
| Geometry role when a feature has multiple geometries | Standard permits server choice | Per-family contract in 033 |
| Position sample partition/index combination | Depends on volume, skew, retention, joins | 027/054 benchmark |
| Geography materialization | Depends on distance/proximity requirements and cost | 033/054 |
| 3D/volume/query profile | Vertical datum and client requirements unknown | 034/050/053 |
| Precision reduction methods and anti-differencing | Mission/policy-specific | 039A threat and policy analysis |
| SpatiaLite reduced-profile boundary | Operational edge constraints unmeasured | 042/044/054 |
| Footprint/trajectory authority in specific profiles | Depends on supplied versus calculated products | 034/035/profile work |

---

## 19. Validation Against Plan Success Criteria

| Topic plan success criterion | Status | Evidence |
|---|---|---|
| Categories and resource-family requirements have source anchors | Met | Sections 3, 5, 6 |
| Static/dynamic/deployment/observation/FOI/sampling/trajectory/footprint/extent distinguished | Met | Sections 5, 6, 11 |
| Geometry/CRS/coordinate/vertical/precision/uncertainty/provenance documented | Met | Sections 6-7, 12 |
| Spatial/spatial-temporal query, extent, filter, sort, paging, index documented | Met | Sections 8-9, 11 |
| Candidate storage options use explicit criteria | Met | Section 10 |
| Security, policy, DDIL, fixture, performance, conformance, interoperability covered | Met | Sections 13-15 |
| Implementation and community lessons incorporated informatively | Met | Section 3.3 and test/storage conclusions |
| Recommendations are bounded and decision-usable | Met | Sections 2.2 and 17 |
| Downstream handoffs are explicit | Met | Section 16 |
| References are explicit and reproducible | Met | Section 20 and source pins in metadata/Section 3 |

### 19.1 Required Matrix Check

Section 6 includes all fourteen required fields: data category, resource family, source anchor, geometry type, temporal classification, authority classification, CRS/coordinate needs, query/index needs, temporal interaction, precision/uncertainty/provenance, security/policy, storage patterns, downstream handoff, and notes/unresolved issues.

### 19.2 Report Completion Checklist

- [x] Topic ID matches overall research plan index
- [x] Topic research plan is linked and aligned
- [x] Core questions and six phases are covered
- [x] Findings have reproducible source anchors
- [x] Normative, project, informative, deferred, and unresolved evidence are distinguished
- [x] Mutable technology sources identify versions/snapshots and access date
- [x] Controlled-source limitations are explicit
- [x] Accepted prior-report decisions are reconciled
- [x] Executive summary is independently decision-usable
- [x] Recommendations, risks, open questions, and handoffs are explicit
- [x] All ten plan success criteria are mapped
- [ ] Plan-owner acceptance and acceptance date recorded

This report is complete for review, but it is not an accepted downstream baseline until the Glaux Project Lead accepts it.

---

## 20. References

### 20.1 Project, Governance, and Accepted Research

- [IDR-SRV-026 research plan](../IDR%20Plans/idr-srv-026-geospatial-storage-and-query-strategy.md)
- [Glaux Server overall IDR research plan](../IDR%20Plans/overall-idr-research-plan.md)
- [Glaux Server goal and definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research report template](../../../../../Governance/research-report-template.md)
- [OGC API - Connected Systems upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)
- Accepted [IDR-SRV-015 through IDR-SRV-025 reports](.)
- Implementation studies [IDR-SRV-014A through IDR-SRV-014G](.)

### 20.2 Approved Standards

- [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html)
- [Official CSAPI `v1.0.0` source at commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [CSAPI Part 1 System location requirement](https://github.com/opengeospatial/ogcapi-connected-systems/blob/v1.0.0/api/part1/standard/requirements/system/req_location_time.adoc)
- [CSAPI Part 1 Procedure location prohibition](https://github.com/opengeospatial/ogcapi-connected-systems/blob/v1.0.0/api/part1/standard/requirements/procedure/req_location.adoc)
- [CSAPI Part 1 Deployment Features](https://github.com/opengeospatial/ogcapi-connected-systems/blob/v1.0.0/api/part1/standard/sections/clause_10_requirements_class_deployment_features.adoc)
- [CSAPI Part 1 Sampling Features](https://github.com/opengeospatial/ogcapi-connected-systems/blob/v1.0.0/api/part1/standard/sections/clause_13_requirements_class_sampling_features.adoc)
- [CSAPI Part 1 Advanced Filtering](https://github.com/opengeospatial/ogcapi-connected-systems/blob/v1.0.0/api/part1/standard/sections/clause_15_requirements_class_advanced_filtering.adoc)
- [OGC API - Features - Part 1: Core corrigendum, OGC 17-069r4](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC API - Features - Part 2: CRS by Reference, OGC 18-058](https://docs.ogc.org/is/18-058/18-058.html)
- [GeoJSON RFC 7946](https://www.rfc-editor.org/rfc/rfc7946)
- [OGC SensorML Encoding Standard 3.0, OGC 23-000](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model Encoding Standard 3.0, OGC 24-014](https://docs.ogc.org/is/24-014/24-014.html)
- [OGC CRS resources](https://www.ogc.org/standards/crs/)

### 20.3 Geospatial Technology Primary Sources

- [PostgreSQL 18 documentation](https://www.postgresql.org/docs/18/)
- [PostGIS 3.6 documentation](https://postgis.net/docs/)
- [PostGIS spatial database management and geometry/geography](https://postgis.net/docs/using_postgis_dbmanagement.html)
- [PostGIS spatial indexes](https://postgis.net/docs/using_postgis_dbmanagement.html#build-indexes)
- [PostGIS `ST_Transform`](https://postgis.net/docs/ST_Transform.html)
- [PostGIS `ST_SetSRID`](https://postgis.net/docs/ST_SetSRID.html)
- [PostGIS `ST_IsValid`](https://postgis.net/docs/ST_IsValid.html)
- [PostGIS `ST_3DIntersects`](https://postgis.net/docs/ST_3DIntersects.html)
- [SpatiaLite project](https://www.gaia-gis.it/fossil/libspatialite/home)
- [DuckDB Spatial extension](https://duckdb.org/docs/stable/core_extensions/spatial/overview)
- [DuckDB Spatial functions and bundled PROJ note](https://duckdb.org/docs/stable/core_extensions/spatial/functions)
- [Rust `geo` documentation](https://docs.rs/geo/)
- [Rust `geo-types` 0.7.20](https://docs.rs/geo-types/0.7.20/geo_types/)
- [Rust `geojson` 1.0.0](https://docs.rs/geojson/1.0.0/geojson/)
- [GDAL stable documentation](https://gdal.org/en/stable/)
- [GDAL security considerations](https://gdal.org/en/stable/user/security.html)

### 20.4 Informative Implementation and Interoperability Sources

- [OS4CSAPI organization](https://github.com/OS4CSAPI)
- [Connected Systems Go](https://github.com/OS4CSAPI/connected-systems-go)
- [OS4CSAPI client](https://github.com/OS4CSAPI/ogc-client-CSAPI_2)
- [SECD interoperability repository](https://github.com/Sam-Bolling/csapi-server-interop-secd)
- [CSAPI Explorer](https://ogc-csapi-explorer.pages.dev/)
- [OS4CSAPI discussions](https://github.com/orgs/OS4CSAPI/discussions)

---

**Review-state record:** IDR-SRV-026 research is complete and placed in review on September 14, 2026. The report is not accepted, IDR-SRV-027 is not authorized or started, and neither draft Part 3 nor server implementation is authorized by this iteration.

# Section 058: Draft CSAPI Part 4 Sampling Features Study - Research Report

**Topic ID:** IDR-SRV-058<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-058 Research Plan](../IDR%20Plans/idr-srv-058-draft-csapi-part-4-sampling-features-study.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Five; draft maturity, differences from approved standards, type semantics, Glaux/implementation impacts, and adoption options<br>
**Methodology Used:** Pinned source and dependency review; artifact checks; comparison with accepted research; bounded OSH and CS-Go code inspection<br>
**Research Time:** Conducted September 17, 2026; elapsed effort was not separately logged<br>
**Primary Source(s):** Official Part 4 working draft at `05a3c62d198ee52d0cf81a734b700967b7d864a1`; approved Connected Systems Parts 1 and 2; their `v1.0.0` source; directly relevant O&M, OMS, SensorML and GeoPose provisions<br>
**Supporting Resources:** Accepted IDR reports identified below; Goal and Definition v1.6; Implementation Guide v0.1; official maintenance history; pinned OSH and CS-Go source<br>
**Document Purpose:** Inform a later decision on whether and how draft Part 4 should enter Glaux Server's plans, without making that scope decision<br>
**Author(s):** OpenAI Codex<br>
**Accepted By:** Pending Glaux Project Lead review<br>
**Acceptance Date:** Not yet accepted<br>
**Date:** September 17, 2026<br>
**Last Updated:** September 17, 2026

---

## Usage Rules

This report is a supplement to the completed 67-topic research baseline. **Finding** means directly supported by the cited evidence; **interpretation** means the analysis of that evidence; **recommendation** means a proposed Glaux choice. Draft language and implementation precedents are not approved standards obligations. Publication of this report does not accept it, adopt Part 4, or change the Goal and Definition or Implementation Guide.

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base
4. Findings by Research Question
5. Decision Analysis
6. Key Recommendations
7. Implementation Implications and Estimates
8. Risks, Constraints, and Open Questions
9. Validation Against Plan Success Criteria
10. Next Steps and Handoff
11. References
12. Appendix: Reproducible Artifact Checks

---

## 1. Executive Summary

**Part 4 is worth considering, but the current working draft is not a complete implementation contract.** It adds specialized sampling descriptions to the generic SamplingFeature resource already required by approved Part 1. Its useful material includes ordinary sampling points, curves and surfaces; specimens; statistical samples; feature parts; and parameter-based shapes such as profiles, spheres, viewing frusta and spherical sectors. These are specializations of an existing resource family, not evidence that Glaux needs another service or a general-purpose 3D GIS platform. [D1], [D2], [P1]

The principal problem is more substantial than a draft label. The document explicitly leaves encoding and conformance packaging unfinished. Its schemas have unresolved local references, disagree with several prose type identifiers and fields, and omit a specific StatisticalSample schema. A relative point is prohibited from carrying orientation, while the referenced current SensorML pose alternatives require orientation. A mobile feature's latest or historical geometry also needs behavior beyond simply storing its JSON. [D1]–[D5], [SML]

OSH supplies useful implementation precedent for selected parametric descriptions, but its legacy names and generic bindings are not proof of conformance to this exact draft. The inspected CS-Go path handles generic SamplingFeatures and does not establish preservation or validation of the draft's extra properties. Neither server was built or exercised during this study. [OSH], [CSGO]

**Recommendation for discussion: keep the current design compatible with these descriptions, but do not add blanket Part 4 implementation to the completion target now.** Glaux's existing model, source-preservation, validation and PostgreSQL/PostGIS choices already provide the necessary foundation. The concrete near-term alternative is a deliberately limited experiment covering static sampling points, curves and surfaces, with its exact draft interpretation and encoding corrections documented. Broader parametric support needs decisions on the wire contract, pose, time and geometry behavior before it can be estimated responsibly.

This does **not** reduce the 25-class Parts 1 and 2 target, undo planned experimental Part 3, or suggest waiting for Part 4 before continuing the server. The next authorized step is a synthesis addendum; the subsequent Goal/Guide discussion decides whether to adopt any recommendation.

---

## 2. Scope and Plan Alignment

IDR-SRV-058 was executed against its published plan at Glaux commit `3942e6de8be11c0b66379ba6f2205bc591168208`. The study covered the complete active Part 4 feature-type chapter, its requirement-class files and schema inventory, relevant referenced standards, the approved generic baseline, directly relevant maintenance history, selected peer implementation paths, and affected Glaux research/planning sections.

Excluded: server implementation, software installation, live service testing, a fresh survey of all CSAPI implementations, unrelated standards maintenance, changes to prior accepted reports, and changes to the synthesis, Goal or Guide. No project scope option was adopted.

### Research Question Coverage Matrix

| Plan question | Question, shortened | Coverage status | Evidence location |
|---|---|---|---|
| Q1 | What exists, and how mature is it? | Complete as an assessment; upstream gaps explicitly unresolved | §§3, 4.1, 8, 12 |
| Q2 | What differs from the approved baseline? | Complete | §4.2 |
| Q3 | What do the feature types and behaviors define? | Complete inventory; unresolved encoding/semantic choices identified | §4.3 |
| Q4 | What changes for Glaux, and what do peers establish? | Complete within the planned code-inspection boundary; no runtime interoperability claim | §§4.4, 7 |
| Q5 | Which adoption option is justified? | Complete recommendation, pending project decision | §§4.5, 5–6, 10 |

The detailed questions are addressed within those themes: authority and inventory in §4.1; relationships and inherited dependencies in §§4.2–4.3; frames, time, geometry and validation in §4.3; storage, access control and verification in §§4.4 and 7; and concrete adoption boundaries in §5.

---

## 3. Evidence Base

### 3.1 Primary Sources Reviewed

Access dates below use the project's America/New_York date, September 17, 2026; requests after midnight UTC occurred September 18 UTC.

| Source | Version / status | Authority class | Stable anchor / reviewed scope | Availability and limitations |
|---|---|---|---|---|
| [D1] Official Part 4 wrapper and review artifacts | `part4-working-draft`, `05a3c62d198ee52d0cf81a734b700967b7d864a1`, June 30, 2026; `swg-draft` | Official working draft, not approved | `api/part4/standard/23-004r0.adoc`, wrapper clauses, README, rendered HTML/PDF inventory | Branch head rechecked; unchanged from the plan. No local Metanorma build performed. |
| [D2] Feature-type chapter | Same pin | Draft proposed requirements and informative explanations | `api/part4/sections/clause_13_sampling_feature_types.adoc`, all active type sections and commented Material Sample block | Direct source review; unresolved text preserved as gaps. |
| [D3] Requirement classes | Same pin | Draft requirement metadata | `api/part4/requirements/` and its `sampling/` copies | Include and duplication checks; metadata is not a complete conformance suite. |
| [D4] JSON schemas | Same pin, declared JSON Schema 2020-12 | Draft encoding artifacts | All files under `api/part4/openapi/schemas/` | Direct inspection and bounded checks; no claim that the unmodified bundle passes end-to-end validation. |
| [D5] Sampling examples | Same pin | Informative examples | Nine files under `api/part1/openapi/examples/sampling/` | Located outside Part 4; not automatically valid draft fixtures. |
| [P1], [P2] Approved CSAPI | Part 1 OGC 23-001 and Part 2 OGC 23-002, v1.0; source tag `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2` | Controlling approved baseline | Part 1 §§8.7, 14, 16, 19.1; Part 2 stream/observation/command associations; corresponding tagged clauses | Compared published Part 1 text with source; no new obligation inferred from unintegrated source. |
| [OM] O&M 2.0 | OGC 10-004r3 | Referenced conceptual standard | §§10.2 and 11.2, spatial sampling and specimens | PDF text accessible. Screenshot requests timed out; no conclusion depends on unread diagram detail. |
| [OMS] OMS 3.0 | OGC 20-082r4 | Referenced conceptual standard | §§13.4, 13.5, 13.14: MaterialSample, StatisticalSample, StatisticalClassification | HTML retrieved successfully through direct HTTP after browser-tool access failed. |
| [SML], [GP] SensorML and GeoPose | SensorML 3.0, OGC 23-000; GeoPose 1.0, OGC 21-056r11 | Approved dependencies where incorporated | SensorML §8.5.1.4 and `Pose.json`; GeoPose Basic forms | Only directly relevant positioning provisions reviewed; no adoption of all GeoPose capabilities. |
| [H] Official history | Dated issue/PR retrieval | Informative provenance or unresolved discussion | Issues 34, 51, 82, 165; PR 96; Part 4 path history | State/contents checked; discussion does not repair the draft or change the approved baseline. |

### 3.2 Supporting Sources Reviewed

| Source | Version / commit | Relevance and review boundary | Limitations |
|---|---|---|---|
| [OSH] OpenSensorHub | `9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08` | Selected sampling classes, GeoJSON bindings, CSAPI feature paths and demo/test evidence | Source inspection, not a deployed-service test or full product audit. |
| [CSGO] Connected Systems Go | `b1fd2e0e9bd69e222d05258d659a842ca24502cb` | Generic SamplingFeature model, wire DTO/formatter, handlers and relevant searches | Absence statements apply only to the searched snapshot. No runtime tests. |
| [R] Accepted Glaux research | Original 67-topic baseline, including IDR-SRV-057 | Relevant portions of 006–008, 011, 015–019, 021–026, 028, 034, 050–051, 053, 056; 014A/B/H as implementation-study entry points | Not a repeat review of every prior report. Exact affected sections are identified in §7. |
| [G], [IG] Current project plans | Goal v1.6 Approved; Guide v0.1 Draft, both present at Glaux `3942e6d` | Current authoritative goal and proposed design | Guide choices are still a draft, not implemented software. |
| [REG] Shared history register | v1.12 consulted; bounded Part 4 refresh recorded as v1.13 | Published/specialized sampling boundary and relevant official history | Other register topics were not re-audited. |

### 3.3 Evidence Quality Notes

The strongest conclusion is about the **condition of this particular draft**, not about how the future standard will settle its ambiguities. The source, schema and dependency differences are directly inspectable. Estimates of downstream work are analysis, not measured implementation results.

PR 96 merged December 13, 2024 as `191212a40ac841bfec0a3b6626417b348fde526b`, moving specialized schemas out of Part 1. Issue 82 is closed; issue 165 remains open. Its July 2026 discussion explicitly raises the misleading specimen example and link/filter questions. Issues 34 and 51 provide earlier separation and association-mapping context. No new published resolution was established by these checks. [H]

SWE Common remains part of Glaux's approved baseline and can describe relevant observation values. This study found no basis to expand its implementation scope merely because Part 4 refers to sampling geometry. SFA/WKT supports an existing Part 1 query requirement; it is not the same subject as Connected Systems Part 4.

---

## 4. Findings by Research Question

### 4.1 Q1 - A Packaged Review Draft, Not a Finished Contract

**Finding.** The execution pin is unchanged from the plan. The standalone entrypoint includes wrapper clauses, the feature-type chapter and an encoding placeholder. Its conformance clause acknowledges unfinished linkage and packaging; its encoding section says Part 4-specific encoding requirements have not been integrated. Thus a schema directory is not proof that the document has adopted a complete encoding contract. [D1]

The source contains ten active requirement-class sections, covering the spatial family, specimens, statistical samples, feature parts, the parametric base, relative points, spheres, profiles, viewing frusta and spherical sectors. Material Sample is commented out. The 22 requirement-class files are eleven identical root/nested pairs, not 22 distinct classes; the extra Material Sample class file does not activate its commented chapter. The schema directory contains 15 JSON files. [D2]–[D4]

The concrete problems relevant to implementation are:

| Evidence | Observed problem | Consequence |
|---|---|---|
| Standalone conformance and encoding clauses | No completed Part 4 conformance/test and encoding package | Do not invent or advertise approved Part 4 conformance URIs. |
| Requirement metadata | Inheritance still names `/req/sampling-features`; published Part 1 uses `/req/sf`. Parametric metadata still names SensorML `2.1` relative-pose requirement URIs while the bibliography names SensorML 3.0. | Intended inheritance needs explicit reconciliation; literal identifiers cannot be silently substituted. |
| Schema reference closure | Eight relative-reference occurrences target three absent files: `samplingFeature.json`, `../sensorml/sensormlDefs.json`, `../common/commonDefs.json`. | The Part 4 directory cannot be used unchanged as a self-contained validator package. |
| Prose versus type schemas | Six non-O&M concrete types use the provisional `x-CS-TBD/1.0` namespace in prose but `OGC-SML/2.0` in schemas; RelativeSamplingPoint is also named SamplingPointXYZ in its schema. | Client/server discrimination and advertised type identity would require a documented experimental choice. |
| Schema family inventory | StatisticalSample has active prose but no specific schema; FeatureProxy has a schema but no corresponding active requirement class. | File presence and generic fallback acceptance cannot establish full family coverage. |
| Parametric subtypes | Profile, sector, frustum and relative-point constraints differ across prose, schemas or inherited pose definitions. | Merely fixing missing file paths does not fix the contract. |
| Rendered review artifacts and README | Generated artifacts are present at the execution pin despite README wording about artifact-only delivery. | Use pinned artifacts and actual tree contents; do not infer artifact absence from the README. |

**Interpretation.** These findings support careful experimentation, but not a claim that implementing the current schemas is equivalent to implementing the draft. They also explain why a broad implementation commitment would conceal decisions rather than remove uncertainty.

### 4.2 Q2 - What Is Already Required, and What Part 4 Adds

**Finding.** Approved Part 1 already defines the SamplingFeature resource, required common identity/type/name attributes, parent-system and sampled-feature associations, optional sampling-chain relationships, canonical and system-scoped endpoints, collections, applicable writes and queries, and GeoJSON representation rules. It expressly leaves concrete sampling types to future parts. Part 2 already associates these resources with data and control streams and observations/commands; Part 4 is not the origin of those associations. [P1], [P2]

| Subject | Approved baseline | Draft Part 4 difference |
|---|---|---|
| Resource family and routes | Generic SamplingFeatures and their existing navigation/transaction context | Specialized descriptions; no separate per-type REST service is established in the reviewed chapter. |
| Sampling relationships | Parent System, ultimate sampled feature and optional sample-of chain | Type-specific meaning, not permission to replace the relationship model. A FeaturePart is not automatically a System subsystem. |
| Geometry and type | Generic feature representation, with permitted additional properties | Type/shape associations and subtype-specific fields; spatial class requires point, curve and surface support, while solid is optional. |
| Time | `datetime` filters resource `validTime`; dynamic property snapshots are possible in Part 1 §14.7 | Spatial draft adds latest/as-of geometry rules, conditional on mobile sampling support. |
| Encodings | Part 1's generic SamplingFeature GeoJSON contract | Candidate specialized schemas without finished Part 4 encoding requirements; SensorML pose reuse does not create a SensorML SamplingFeature representation. |
| Conformance | The established Parts 1 and 2 target | No additional approved class to add to Glaux's 25-class count. |

**Important inherited ambiguities.** The published specimen example contains extra properties not defined by the generic Part 1 table. That does not make every specimen specialization a Part 1 obligation. Likewise, the Part 2 source tree contains an unintegrated dynamic-feature-properties draft file, but the tagged Part 2 entrypoint does not include it; it cannot supply missing approved snapshot requirements. [P1], [P2], [H]

**Interpretation.** Prior research was correct to retain generic SamplingFeatures while classifying Part 4 as informative future material. The gap is a dedicated evaluation of specializations, not an omitted mandatory service. IDR-008 §10 and IDR-026 §18.2 remain valid on that boundary.

### 4.3 Q3 - Feature Families, Dependencies and Behavior

#### 4.3.1 Complete Family Inventory

The table maps the active chapter's families and inactive artifacts. Requirement names below identify **draft** statements, not new Glaux obligations. All rows derive from [D2]–[D4], with dependency qualifications in the following subsections.

| Family | Draft requirements and meaning | Schema / representation evidence | Consequence or unresolved point |
|---|---|---|---|
| Spatial sampling: Point, Curve, Surface | `/req/sampling-spatial/{type,om,shapes,location,location-time}`; O&M-based shape; Point, LineString and Polygon support required for this class | `samplingSpatial.json`, `samplingPoint.json`, `samplingCurve.json`, `samplingSurface.json` | Existing geometry infrastructure is relevant. Static shapes are the clearest limited experiment; schema closure and URI/CURIE handling still need an explicit encoding decision. |
| Sampling Solid | Same spatial family; explicitly optional | `samplingSolid.json` permits Point-or-null geometry | This is not a solid encoding or evidence of volumetric query support. |
| Specimen | `/req/sampling-specimen/{type,om}`; `materialClass` and `samplingTime` required; additional collection/storage/method/size metadata | `samplingSpecimen.json` | Collection time is not validity or ingestion time. Schema reduces `size` to a number without a unit-bearing Measure and does not specify all recalled fields. O&M mapping is incomplete. |
| Statistical Sample | `/req/statistical-sample/{type,attributes}`; implements OMS StatisticalSample | No dedicated schema or discriminator branch | Draft table calls classification a CodeListValue; referenced OMS uses StatisticalClassification with concept/classification URIs. Mapping must be resolved rather than guessed. |
| Feature Part | `/req/feature-part/{type,attributes}`; optional `definition` and `partId`; can describe a part or act as a whole-feature proxy | `samplingPart.json` uses `partID`, with no explicit `definition` mapping | Both identifier and property spelling need reconciliation. Preserve identity/relationships rather than automatically creating a subsystem or geometry. |
| Parametric base | `/req/sampling-parametric/{attributes,pose}`; intrinsic shape parameters plus extrinsic pose | `samplingSpatialParametric.json` | Common table calls pose optional, surrounding prose treats it as universal, and some subtypes make it required. Missing pose reference and stale inherited identifiers need reconciliation. |
| Relative Sampling Point | `/req/relative-sampling-point/{type,pose}`; relative Cartesian location; orientation forbidden | `samplingPointXYZ.json` | Name/URI mismatch, unfinished pose sentence, and no compatible orientation-free alternative among the current inherited Pose schema branches. |
| Sampling Sphere | `/req/sampling-sphere/{type,attributes}`; required radius in metres | `samplingSphere.json` | Namespace/pose issues remain. Storing radius is not proof of computing or querying a sphere's extent. |
| Sampling Profile | `/req/sampling-profile/{type,attributes}`; pose, signed profile axis and bin distances required; distances in metres, optional spread angle in degrees | `samplingProfile.json` | Schema allows an alternative count/start/step form not specified in the prose and does not impose the same pose requirement. Ordered bins are semantic data, not expendable extension fields. |
| Viewing Frustum | `/req/viewing-frustum/{type,attributes}`; pose, longitudinal/up axes and field of view; optional aspect ratio and length | `viewingFrustum.json` | Prose distinguishes conical from pyramidal form using aspect ratio. Missing length cannot justify an invented finite footprint. Required pose and wire constraints need reconciliation. |
| Spherical Sector | `/req/spherical-sector/{type,attributes}`; required radius; optional inner radius and azimuth/elevation limits with defaults | `sphericalSector.json` | Prose uses `minAzimuth`/`maxAzimuth`/`minElevation`/`maxElevation`; schema uses abbreviated names and requires all four limits. |
| Material Sample (inactive) | Entire chapter block commented out | Unincluded requirement-class pair; no specific schema | Not an active class merely because the requirements files exist. Do not silently replace the active O&M Specimen with OMS MaterialSample. |
| Feature Proxy (schema-only) | No independent active prose class; Feature Part discussion already includes proxy use | `samplingProxy.json`; branch in `anySamplingFeature.json` | Treat as a leftover/unspecified encoding artifact, not an extra supported class inferred from the file. |

`anySamplingFeature.json` has twelve specific branches plus a generic fallback. A fallback can admit an unrecognized type without checking its specialized fields; that is not evidence of typed Part 4 support.

#### 4.3.2 Inherited Models and Encoding Conflicts

**Finding.** O&M 2.0 §10.2 supplies the spatial-shape taxonomy; §11.2 describes Specimen metadata. The draft's spatial requirement metadata points to Clause 9 as “Spatial Sampling Features,” while that material is actually in Clause 10. The specimen recall table narrows sampling time to an instant and uses storage-location terminology, while O&M's model uses a broader temporal object and `currentLocation`. These differences need an explicit mapping if that specialization is chosen; the table is not a complete mechanical transcription of the dependency. [D2], [D3], [OM]

OMS 3.0 §§13.4–13.5 distinguish material and statistical samples. Its §13.14 models a statistical classification with separate concept and classification URIs. The draft's abbreviated classification row and missing schema do not establish how to encode that structure. This does not require Glaux to implement the entire OMS abstract model. [OMS]

SensorML 3.0 supplies geographic and Cartesian-relative pose concepts; its `Pose.json` has four alternatives, all requiring either angles or quaternion orientation. The draft relative-point rule forbids orientation. Therefore substituting today's `Pose.json` for the missing Part 4 reference would still leave a semantic contradiction. A special orientation-free position object would be an explicit experimental interpretation, not a repair that can be presented as the standard. [D2], [D4], [SML]

**Interpretation.** Dependency reuse reduces work only where the contracts agree. Keep actual source identifiers and the intended concepts separately visible; do not normalize away the evidence of drift.

#### 4.3.3 Time, Frames and Query Semantics

**Finding.** The spatial draft specifies latest known geometry when `datetime` is absent; the last geometry known at an instant when an instant is supplied; and the last location known at an interval's end when a period is supplied. The mobile condition applies to `/req/sampling-spatial/location-time`. Approved Part 1's validity-intersection filter continues to apply; selecting a geometry is a different operation from deciding whether a resource is temporally eligible. [D2], [P1]

The draft does not fully settle late-arriving evidence, ties, missing history, open-ended intervals, future instants or the relation between event time and “known” time. Those choices cannot be supplied by accidentally using the database ingestion timestamp. Its overview also speaks of posting a new feature on location change while its requirement describes historical/latest geometry. This is a tension to resolve against the existing identity design, not an instruction to mint a new identity on every movement.

Parametric descriptions locate an inner frame relative to an outer frame. The prose permits GeoPose Basic geographic positioning and SensorML relative positioning; some subtypes also define signed axes. Metres, degrees, geographic coordinate conventions and Cartesian offsets must remain distinct. GeoPose's named latitude/longitude/height position cannot simply be copied as a GeoJSON longitude/latitude coordinate array. [D2], [SML], [GP]

**Interpretation.** A server can preserve a relative pose without knowing its absolute geographic position. Computing that position requires sufficient frame, parent-pose and time information. The draft describes tracking and possible profile-curve reconstruction, but does not supply a general transformation, interpolation, terrain-intersection or occlusion algorithm. No such engine is made a Glaux requirement by these examples.

Part 1 already defines `bbox` and WKT `geom` behavior, including their different treatment of non-spatial features. Part 4 does not provide a complete new query contract for every implicit volume. A supplied anchor point may support a documented representative-location query; it is not automatically the sphere, frustum or sector's extent. Preserve the current explicit-geometry behavior and record any chosen derived geometry and approximation separately. [P1], IDR-026 §§7.4, 8.1, 11.3

### 4.4 Q4 - What Existing Implementations and Glaux's Design Establish

#### 4.4.1 OSH: Useful Parametric Descriptions, Not Exact-Draft Proof

**Finding.** At the inspected OSH pin, `lib-ogc/sensorml-core/.../sampling/` contains a parametric base and concrete `SamplingPointXYZ`, `SamplingSphere`, `ViewingFrustum` and `ViewingSector` classes. They use the legacy `OGC-SML/2.0` type namespace. Demo builders contain relative sampling points on Saildrone, a Mavic camera frustum, and a NEXRAD sector. The nearby NEXRAD sphere example is commented out. [O1], [O2]

The CSAPI `FoiHandler` selects the generic GeoJSON binding. Its shared `GeoJsonBindings` reads a generic temporal feature and handles scalar properties and selected objects, including pose, rather than instantiating a Part 4 subtype validator. The custom-property reader skips array values through its default case. That is a specific reason to test preservation of profile bin arrays rather than infer it from flexible feature storage. This is a code-derived prediction, not an executed round-trip result. [O3], [O4]

`TestFois` supplies generic CRUD, property-filter and ordinary geometry/bbox evidence. The inspected source does not supply a tested oracle for the pinned draft's relative-point prohibition, profile contract, latest geometry or parametric volume calculation. The selected class/binding/test blobs are unchanged from the prior IDR-014A pin. [O5]

**Interpretation.** OSH is useful precedent for representing pose and sampling parameters. Its class names, demonstrations and generic bindings cannot be treated as conformance evidence for all current draft families or as the authority for resolving their conflicts.

#### 4.4.2 CS-Go: Generic Static Features Do Not Imply Extra-Property Support

**Finding.** CS-Go's domain model has geometry and JSONB properties, but its public SamplingFeature GeoJSON property struct lists only UID, name, description, feature type, valid time and sampled-feature link. The formatter reads/writes those explicit fields; its decoder uses `DisallowUnknownFields`. The inspected call chain therefore predicts rejection of extra fields such as `properties.pose` or `properties.radius`, rather than automatic Part 4 support through JSONB. No request was executed to test that prediction. [C1], [C2]

The router now installs a bundled generic SamplingFeature validator. Ordinary spatial filtering targets `sampling_features.geometry`. The dedicated E2E source uses O&M sampling points, curves and surfaces with Point/LineString/Polygon geometry and tests generic Part 1 behavior. This is useful static-feature evidence, not demonstrated parametric interpretation. The relevant domain, formatter, strict decoder, query and test blobs remain unchanged from the prior IDR-014B pin; the runtime generic validator is newer. [C3], [C4]

The bounded searches covered 2,222 OSH Java/JSON/Gradle/Markdown/XML files and 303 CS-Go Go/JSON/YAML/Markdown files for sampling-specific class and draft identifier terms, followed by direct inspection of the paths above. No Part 4-specific identifiers were found in the CS-Go search. That is not a claim that every possible form of Part 4 support is absent, nor that another branch or later version cannot add it.

#### 4.4.3 Glaux: Retain the Architecture, Add Semantics Only Deliberately

**Interpretation.** Existing Glaux research already separates canonical identity, typed relationships, source documents, dynamic values and indexed geometry. That is a suitable foundation for either deferral or selected experimentation. No evidence here requires new crates, a different database, a new resource service, or a geometry engine. It also does not prove the proposed implementation already satisfies specialized behavior. [R], [IG]

The material additions, if selected, would be specific: a pinned type/encoding interpretation, typed validation, preserved fields, explicit time/geometry behavior and independent tests. Existing safe schema resolution and access checks remain applicable. A frame link or derived shape can expose protected platform position; it must not bypass access control through a supposedly harmless geometry calculation. Unresolved references should not trigger arbitrary network fetches.

### 4.5 Q5 - A Compatibility-Aware Decision Is Justified Now

**Recommendation, not an adopted decision.** Preserve compatible model/storage/codec boundaries as already intended, record the draft and its limitations, and defer a blanket specialized-type commitment. This is not “ignore Part 4”: it prevents the implementation guide from treating generic JSON preservation as support while leaving a clear path to a bounded experiment.

If the project prefers implementing something concrete now, the most defensible starting scope is the **static spatial class's point/curve/surface set**, not an arbitrary one-type claim. Even that needs a documented interpretation for the missing schema dependencies, type URI/CURIE encoding, and the schema's allowance for null geometry versus the prose's shape requirement. Mobile snapshots, implicit-volume queries, specimens and other specialized contracts should not be smuggled into that experiment.

---

## 5. Decision Analysis

| Option and actual scope | Benefits | Costs / risks | Standards and compatibility impact | Assessment |
|---|---|---|---|---|
| Defer all dedicated Part 4 work; implement the existing approved generic resource | Simplest immediate scope | May overlook preservation needs if generic codecs become overly restrictive | Full approved Parts 1 and 2 target remains intact | Viable, but retain this report's warnings rather than ignore the draft. |
| Compatibility-aware design; no specialized support commitment yet | Existing architecture already preserves structured extensions and source meaning; avoids accidental loss or misleading support claims | Requires a small clarification of scope and explicit unknown-type handling in later guide discussion | No new conformance claim or mandatory completion item | **Recommended now.** |
| Selected experimental static spatial support: Point, Curve, Surface | Concrete useful scope with explicit shapes and existing PostGIS support | Requires a coherent experimental encoding, subtype validation, fixtures and documented deviation decisions | Experimental only; all required members of that draft spatial class considered; Solid/mobile support excluded explicitly | Credible alternative if the user wants implementation included. Not selected by this report. |
| Broader experimental support: specimens/statistical/parts plus parametric families and any advertised mobile/derived behavior | Richer sampling context and possible OSH-oriented interoperation | Multiple unresolved wire contracts, frame/time behavior, missing test oracles, and significantly variable geometry cost | Cannot claim complete draft or approved conformance from schema acceptance; exact supported behaviors must be enumerated | Do not commit as a single undifferentiated capability at this snapshot. |

Selection should follow an actual need for the supported behavior, not the fact that an example or peer class exists. A future experiment can proceed before standard publication if its choices are explicit; this report does not require waiting indefinitely for OGC to finalize Part 4.

---

## 6. Key Recommendations

1. **Keep Parts 1 and 2 as the unchanged completion baseline.** Priority: high. The draft adds no approved class to the target. Experimental Part 3 remains a separate, already stated goal.
2. **Prefer compatibility-aware design without blanket Part 4 adoption.** Priority: high. Preconditions: discuss the recommendation after the synthesis addendum; do not turn this report into a scope change. Preserve valid extension data, but distinguish generic acceptance from typed validation and computed behavior.
3. **If experimentation is selected, name the types and behaviors precisely.** Priority: conditional. Start with the static point/curve/surface option or another explicitly justified set; record excluded types, mobile behavior and derivation/query limitations. Resolve only the conflicts relevant to the chosen set.
4. **Keep upstream evidence separate from local corrections.** Priority: high if support is selected. Pin the source, retain original fixtures, document adapted schemas/examples and their rationale in the existing guide/test practice. Do not mint “official” identifiers or silently repair orientation, names, fields and defaults.
5. **Require semantic round-trip and query evidence for any claimed support.** Priority: high if support is selected. A successful JSON parse, map display, HTTP response or database write does not prove preservation of sampling context or correct geometry/time selection.

---

## 7. Implementation Implications and Estimates

### 7.1 Implications and Affected Existing Sections

These are inputs for later drafting, not edits made in this iteration.

| Existing research / planning section | What remains valid | What the later discussion should clarify |
|---|---|---|
| IDR-008 §10; IDR-026 §18.2; Goal §4; Guide §§1, 7.1 | Approved generic baseline and 25-class target; Part 4 not yet adopted | Explicitly distinguish CSAPI Part 4 from Features transactions and SFA. Add experimental intent to Goal §4 only if the user chooses it. |
| IDR-015 §§7.4, 11.3; IDR-021 §6.1; Guide §§4.2–4.3, 6.1–6.2 | Canonical SamplingFeature with GeoJSON representation and preserved extensions | Type-specific semantics do not imply new endpoints or a SensorML SamplingFeature representation. |
| IDR-016 §11.1; IDR-017 §§7, 8.5; Guide §§4.2, 6.1 | Sample/proxy identity and typed parent/sampled/sample-of relationships | Do not remint IDs merely on movement or equate a FeaturePart with a subsystem. |
| IDR-018 §§8.4, 11.2; IDR-034 §8.3; Guide §§4.4–4.5, 6.3 | Separate valid time, observation time and receipt; dynamic snapshots already contemplated | If mobile draft support is chosen, add explicit geometry selection alongside validity filtering, including unknown/late history behavior. |
| IDR-021 §10.4; IDR-022 §7.4; IDR-024 §11.1; Guide §4.3 | Frame, axis and unit preservation | Chosen subtype pose rules, metre/degree fields, coordinate-order conversions and unresolved relative-point contract. |
| IDR-025 §9; IDR-026 §§6, 7.4, 8.1, 11.3; IDR-028 §§6.3, 7.1; Guide §§4.7, 6.1, 6.3 | PostgreSQL/PostGIS with typed relationships/query fields and retained source documents; explicit geometry predicates | Distinguish supplied geometry, anchor location and any derived extent. No database replacement or universal 3D querying follows. |
| IDR-019 §§9.3–9.4; IDR-023 §§9.3, 13.3; IDR-026 §§9.5, 13.3; Guide §§4.3, 4.10 | Provenance, offline reference resolution, bounded parsing and access control | Bound bin arrays/frame traversal if introduced; protect dependent pose information; label derivations and unsupported behavior. |
| IDR-050 §§4.2, 6.3; IDR-051 §14.6; IDR-053 §§8.3, 13.2; IDR-056 §7.2; Guide §§7–8 | Independent expected results; official versus adapted fixture provenance; capability-qualified client evidence | Separate exact-draft checks, local experimental choices and generic Part 1 tests; verify actual client preservation. |
| Final synthesis §§6, 8, 9, 10.2, 11.2, 17 | Original accepted findings and historical completion remain intact | A later addendum should record this new evidence and qualify only affected conclusions. It must not rewrite the original report into an adopted Part 4 plan. |

### 7.2 Effort / Complexity Estimate

No implementation or benchmark was performed, so no hour/day estimate is defensible here. Relative complexity describes incremental work beyond the current approved baseline, not a new schedule.

| Work item | Relative complexity | Estimate basis / assumptions |
|---|---|---|
| Clarify Part 4 boundary and retain compatible extension/source handling | Low incremental | Existing model, storage and codec design already provides these boundaries. |
| Static Point/Curve/Surface experiment | Low to medium incremental | Existing geometry and API behavior reused; exact type/shape validators, reconciled schemas, URI handling and fixtures still required. |
| Specimen, FeaturePart or StatisticalSample typed encoding | Medium, with unresolved scope | No geometric engine needed, but dependency/field mapping and wire contract must first be chosen. |
| Parameter-description storage and typed pose/shape validation | Medium | Requires deliberate namespace/pose/field decisions and lossless round trips; this is not derived-geometry support. |
| Mobile frame/time resolution and spatially meaningful derived representations | High / uncertain | Requires defined missing-history behavior, frame resolution, time selection, derivation accuracy and independent expected results. |
| General solid intersection, terrain footprints, occlusion or sensor simulation | Not estimated; outside the proposed experiment | No basis to add these as mandatory Glaux work from the reviewed draft. |

### 7.3 Verification Cases if Support Is Selected

These are proposed future tests, not tests implemented or passed by Glaux:

| Case | Distinguishing evidence needed |
|---|---|
| Static point/curve/surface | Correct type/shape pair, required common associations, null/missing geometry interpretation, ordinary `bbox`/`geom` results. |
| Specimen or statistical sample | Correct field meanings, collection time distinct from valid time, unit-bearing size/classification mapping if claimed, non-spatial query behavior. |
| Relative point and pose | Chosen orientation-free contract versus rejected orientation, frame identity, unknown or cyclic references, no fabricated absolute location. |
| Profile | Bin-array preservation and order, signed axis, units, explicit versus generated-bin policy, bounded input sizes. |
| Frustum/sector/sphere | Chosen property spellings, required fields and defaults; no guessed finite extent when length/frame information is absent. Any extra numerical restrictions identified as project rules unless sourced. |
| Mobile geometry | No `datetime`, instant, interval end, validity mismatch, late evidence, ties and unavailable history under the chosen interpretation. |
| Access and derived information | Hidden parent/frame/pose cannot leak through geometry or query results; no external dereference during untrusted requests. |
| Round-trip and clients | Original versus adapted fixtures, exact preservation of required semantic fields, and clear distinction between anchor point and sampling extent. |

---

## 8. Risks, Constraints, and Open Questions

### 8.1 Risks and Constraints

- A provisional draft vocabulary can become an accidental public contract. Pin and disclose the implemented interpretation if experimentation is chosen.
- Generic extension storage can hide losses at a serializer or client boundary. The peer findings demonstrate why end-to-end preservation needs testing.
- A geometry-looking value can be mistaken for the represented sample's full extent. Anchor, shape and derived approximation must remain distinguishable.
- Repairing references alone can create false confidence: pose, field spelling, requiredness and dependency-mapping conflicts remain.
- No full unmodified-schema validation, server interoperability run, geometric computation or performance measurement was established here. The diagnostics in §12 have narrower claims.

### 8.2 Open Questions

| Question | Why unresolved / next action |
|---|---|
| Which concrete types and encoding identifiers will the final Part 4 use? | Prose and schemas differ; follow official maintenance or select a documented experimental interpretation only for chosen types. |
| What precisely is the orientation-free relative-point representation? | Draft sentence is incomplete and current Pose alternatives require orientation; needs clarification or an explicitly local choice. |
| Which profile, sector, FeaturePart and specimen field mappings control? | Conflicting names/requiredness or missing mappings; resolve before claiming those wire contracts. |
| What counts as “latest known” geometry for delayed data and an interval with no finite end? | Draft does not supply a complete temporal algorithm; use existing temporal research to formulate a bounded choice if mobile support is selected. |
| How should an implicit shape participate in ordinary spatial filtering? | Draft does not fully bind every shape/frame/time combination to an explicit predicate geometry; specify representative versus derived geometry without inventing a general volume engine. |
| Which capabilities are useful enough for Glaux to adopt experimentally now? | This is the user's project-scope decision, informed by §5, not a source fact. Discuss after the synthesis addendum. |

The remaining uncertainty does not prevent completing this assessment. It limits what can responsibly be promised as implementation or conformance.

---

## 9. Validation Against Plan Success Criteria

| Topic-plan success criterion | Validation status | Evidence |
|---|---|---|
| Q1–Q5 answered or explicitly limited | Met | §§2, 4–8 |
| Reproducible execution pin, authority and artifact gaps | Met | §§3, 4.1, 12 |
| Every actual feature family, including inactive material, accounted for | Met | §4.3.1 |
| Geometry, parameters, relationships, frames, units, time, validation and queries addressed | Met as assessment; unresolved upstream rules identified | §§4.2–4.3, 8.2 |
| Impacts and unchanged prior conclusions explicit | Met | §§4.4, 7.1 |
| Implementation evidence pinned and bounded | Met; runtime interoperability deliberately untested | §§3.2, 4.4 |
| Concrete adoption options and justified recommendation | Met; project choice pending | §§5–7 |
| Report template, coverage validation and later handoff preserved | Met | Required sections present; §10 |
| Relevant official history consulted and authority-classified | Met | §3.3; bounded v1.13 register update |

This validates research execution, not plan-owner acceptance or server conformance.

---

## 10. Next Steps and Handoff

1. **Review this report.** Owner: Glaux Project Lead. Timing: next iteration decision. The recommendation may be accepted as research input without committing to implement Part 4.
2. **On the next `proceed`, record report acceptance and add the findings to the final synthesis as an addendum.** Owner: Codex, under the user's instruction. Preserve the original report and 67-topic completion record; identify new evidence and affected conclusions explicitly.
3. **After that addendum, discuss the Goal and Definition and Implementation Guide implications.** Owner: Glaux Project Lead with Codex. Select an option before changing implementation scope. No Goal/Guide edit or server implementation is authorized by this report.

No special approval wording, new requirements document or additional research programme is needed for this handoff.

---

## 11. References

### Official Draft and Approved Standards

- [D1: Part 4 standalone entrypoint][D1]; [scope](https://github.com/opengeospatial/ogcapi-connected-systems/blob/05a3c62d198ee52d0cf81a734b700967b7d864a1/api/part4/standard/sections/clause_1_scope.adoc), [conformance](https://github.com/opengeospatial/ogcapi-connected-systems/blob/05a3c62d198ee52d0cf81a734b700967b7d864a1/api/part4/standard/sections/clause_2_conformance.adoc), [README](https://github.com/opengeospatial/ogcapi-connected-systems/blob/05a3c62d198ee52d0cf81a734b700967b7d864a1/api/part4/standard/README.adoc), [rendered HTML](https://github.com/opengeospatial/ogcapi-connected-systems/blob/05a3c62d198ee52d0cf81a734b700967b7d864a1/api/part4/standard/23-004r0.html), and [artifact commit](https://github.com/opengeospatial/ogcapi-connected-systems/commit/05a3c62d198ee52d0cf81a734b700967b7d864a1).
- [D2: Entire feature-type chapter][D2]. Stable section anchors include `clause-sf-spatial`, `clause-sf-specimen`, `clause-sf-statistical-sample`, `clause-sf-feature-parts`, `clause-sf-parametric`, `clause-sf-sampling-point-xyz`, `clause-sf-sampling-sphere`, `clause-sf-sampling-profile`, `clause-sf-viewing-frustum`, and `clause-sf-sector`.
- [D3: Nested requirement-class files][D3]; root copies are one directory above. [D4: Complete schema directory][D4]; individual filenames appear in §§4.3 and 12. [D5: Sampling examples][D5]. All use the same Part 4 execution commit, not a mutable branch URL.
- [P1: Approved Connected Systems Part 1][P1]; [tagged SamplingFeature clause](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/clause_13_requirements_class_sampling_features.adoc), [common time requirement](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/clause_7_requirements_class_common.adoc), [advanced filters](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/clause_15_requirements_class_advanced_filtering.adoc), and [GeoJSON mapping](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/clause_20_requirements_class_geojson_encoding.adoc).
- [P2: Approved Connected Systems Part 2][P2]; [tagged entrypoint](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/23-002r0.adoc), [DataStreams/Observations](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/clause_8_requirements_class_datastreams.adoc), and [ControlStreams/Commands](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/clause_9_requirements_class_controlstreams.adoc).
- [OM: O&M 2.0, OGC 10-004r3][OM], §§10.2 and 11.2; [OMS: OMS 3.0, OGC 20-082r4][OMS], §§13.4, 13.5 and 13.14.
- [SML: SensorML 3.0][SML], §8.5.1.4; [pinned Pose schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/sensorml/schemas/json/Pose.json), unchanged in the Part 4 execution tree; [GP: GeoPose 1.0][GP], Basic forms.
- [H: Issue 165][H], including its July 2026 discussion; [issue 34](https://github.com/opengeospatial/ogcapi-connected-systems/issues/34), [issue 51](https://github.com/opengeospatial/ogcapi-connected-systems/issues/51), [issue 82](https://github.com/opengeospatial/ogcapi-connected-systems/issues/82), and [PR 96](https://github.com/opengeospatial/ogcapi-connected-systems/pull/96). States retrieved September 17 EDT / September 18 UTC, 2026.

### Implementation Evidence

- [OSH execution tree][OSH]. [O1: Parametric base][O1] and adjacent concrete classes; [O2: Saildrone relative-point builders][O2], [Mavic frustum builder](https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-service-consys/src/test/java/org/sensorhub/impl/service/consys/demodata/MavicPro.java#L551), [NEXRAD sector builder](https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-service-consys/src/test/java/org/sensorhub/impl/service/consys/demodata/Nexrad.java#L368).
- [O3: OSH FoiHandler][O3], [FoiBindingGeoJson](https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/feature/FoiBindingGeoJson.java#L50); [O4: custom property reader][O4], same file's write path at line 194 and generic feature creation at line 531; [O5: TestFois][O5].
- [CS-Go execution tree][CSGO]. [C1: SamplingFeature domain and wire types][C1], [formatter](https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/formaters/geojson_formatters/sampling_feature_geojson.go#L39); [C2: strict decoder][C2]; [C3: router validator][C3], [spatial repository path](https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/repository/sampling_feature_repository.go#L107); [C4: dedicated SamplingFeature E2E source][C4].

### Glaux Evidence and Planning

- [R: Accepted research traceability index][R]. The exact topic numbers and sections used are specified in §7.1; filenames in that index resolve the individual reports. [IDR-008](idr-srv-008-conformance-class-and-requirement-mapping-report.md#10-upstream-standards-maintenance-context-and-disposition) and [IDR-026](idr-srv-026-geospatial-storage-and-query-strategy-report.md#182-constraints) establish the existing Part 4 boundary.
- [G: Goal and Definition v1.6][G]; [IG: Implementation Guide v0.1][IG]. Baseline repository commit: `3942e6de8be11c0b66379ba6f2205bc591168208`.
- [REG: Upstream-history register][REG]; [Research Report Template](../../../../../Governance/research-report-template.md).

[D1]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/05a3c62d198ee52d0cf81a734b700967b7d864a1/api/part4/standard/23-004r0.adoc
[D2]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/05a3c62d198ee52d0cf81a734b700967b7d864a1/api/part4/sections/clause_13_sampling_feature_types.adoc
[D3]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/05a3c62d198ee52d0cf81a734b700967b7d864a1/api/part4/requirements/sampling
[D4]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/05a3c62d198ee52d0cf81a734b700967b7d864a1/api/part4/openapi/schemas
[D5]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/05a3c62d198ee52d0cf81a734b700967b7d864a1/api/part1/openapi/examples/sampling
[P1]: https://docs.ogc.org/is/23-001/23-001.html
[P2]: https://docs.ogc.org/is/23-002/23-002.html
[OM]: https://docs.ogc.org/as/10-004r3/10-004r3.pdf
[OMS]: https://docs.ogc.org/as/20-082r4/20-082r4.html
[SML]: https://docs.ogc.org/is/23-000/23-000.html
[GP]: https://docs.ogc.org/is/21-056r11/21-056r11.html
[H]: https://github.com/opengeospatial/ogcapi-connected-systems/issues/165
[OSH]: https://github.com/opensensorhub/osh-core/tree/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08
[O1]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/lib-ogc/sensorml-core/src/main/java/org/vast/sensorML/sampling/ParametricSamplingFeature.java#L24
[O2]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-service-consys/src/test/java/org/sensorhub/impl/service/consys/demodata/Saildrone.java#L278
[O3]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/feature/FoiHandler.java#L38
[O4]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/lib-ogc/swe-common-om/src/main/java/org/vast/ogc/gml/GeoJsonBindings.java#L655
[O5]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-service-consys/src/test/java/org/sensorhub/impl/service/consys/TestFois.java#L219
[CSGO]: https://github.com/SomethingCreativeStudios/connected-systems-go/tree/b1fd2e0e9bd69e222d05258d659a842ca24502cb
[C1]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/domains/sampling_feature.go#L55
[C2]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/common_shared/decode.go#L29
[C3]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/api/router.go#L72
[C4]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/e2e/sampling_feature_test.go#L43
[R]: final-idr-research-report.md#273-thematic-topic-report-traceability
[G]: ../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md
[IG]: ../../../../../Plans/glaux-server/glaux-server-implementation-guide.md
[REG]: ../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md

---

## 12. Appendix: Reproducible Artifact Checks

### 12.1 What Was Actually Checked

The official source was inspected at the recorded commit using Git, PowerShell and direct HTTP. No source artifacts were repaired, no schema substitutes were installed, and no server was built. The artifact review used PowerShell 7.6.6; its existing `Test-Json` command was available.

| Check | Actual result | What it establishes |
|---|---|---|
| Branch and commit | `git ls-remote --heads` and GitHub API both identified `05a3c62d...` | Reproducible execution snapshot; no change from planning pin. |
| Includes and requirement copies | 16 active include targets exist; eleven root/nested file pairs have matching SHA-256 hashes | Mechanical source wiring and duplicate identity, not conformance completeness. |
| Rendered HTML comparison | All ten active class headings and 23 inline requirement identifiers found; four embedded images; unfinished relative-point sentence retained | Bounded content agreement, not complete rendering equivalence. PDF not independently visually audited. |
| JSON syntax | All 15 Part 4 schemas and nine inherited sampling examples parse | Syntax only, not schema validity or semantic correctness. |
| Reference traversal | 39 `$ref` occurrences: 22 existing local targets, nine external occurrences using three GeoJSON schemas, eight missing local targets | Local dependency failure; external Point/LineString/Polygon schemas were retrievable and parsed. |
| Full sphere-schema attempt | Failed during reference resolution with missing `samplingFeature.json` | Schema setup failure, **not** an invalid-instance result and not a successful validation. |
| Fragment diagnostics below | Six controlled checks against each schema's `allOf[1]` only | Demonstrates local constraints/omissions while deliberately excluding the unresolved inherited branch. |

The 15-file inventory is: `anySamplingFeature`, `samplingCurve`, `samplingPart`, `samplingPoint`, `samplingPointXYZ`, `samplingProfile`, `samplingProxy`, `samplingSolid`, `samplingSpatial`, `samplingSpatialParametric`, `samplingSpecimen`, `samplingSphere`, `samplingSurface`, `sphericalSector`, and `viewingFrustum`, all with `.json` suffix.

### 12.2 Fragment Diagnostic Results

| Synthetic instance condition | Fragment result | Interpretation |
|---|---|---|
| Profile axis plus `numBins: 1.5`, start and step, with no pose/binDistances | Accepted | Fragment allows the numeric alternative, even a non-integer bin count; not proof of complete profile validity. |
| Profile containing both binDistances and numeric-bin form | Rejected | The fragment's `oneOf` makes those alternatives exclusive. |
| Frustum axis/upAxis/fov without pose | Accepted | The subtype fragment does not impose the prose's pose requirement. |
| Sector radius without the four prose-defaulted angle limits | Rejected | The fragment requires the abbreviated angle members. |
| Sector radius with all four abbreviated angle members | Accepted | Demonstrates the schema-side field contract only. |
| Relative-point subtype fragment with orientation | Accepted | That fragment has no prohibition; the unresolved inherited pose branch was not evaluated. |

Reproduction pattern, from the pinned official repository root, using existing PowerShell `Test-Json`:

```powershell
$schemaDir = Join-Path (Get-Location) 'api/part4/openapi/schemas'
$sphereInstance = '{"type":"Feature","geometry":null,"properties":{"featureType":"http://www.opengis.net/def/samplingFeatureType/OGC-SML/2.0/SamplingSphere","radius":1}}'
# Expected here: unresolved inherited file, not an instance-validity verdict.
Test-Json -Json $sphereInstance -SchemaFile (Join-Path $schemaDir 'samplingSphere.json')

$profileSchema = Get-Content -Raw -LiteralPath (Join-Path $schemaDir 'samplingProfile.json') |
    ConvertFrom-Json -AsHashtable
$profileFragment = $profileSchema.allOf[1] | ConvertTo-Json -Depth 40
# Only the subtype fragment; the broken inherited schema is excluded.
Test-Json -Schema $profileFragment -Json '{"properties":{"profileAxis":"X","numBins":1.5,"startDistance":0,"stepDistance":1}}'
```

For the structural checks, enumerate with `git ls-tree -r --name-only`, parse JSON with `ConvertFrom-Json -AsHashtable`, recursively inspect `$ref` members, resolve relative paths with `GetFullPath`/`Test-Path`, and compare duplicate requirement files with `Get-FileHash`. Counts refer to reference **occurrences**, not unique target documents.

Several inherited examples are useful negative fixtures after provenance is retained: the XYZ-point example includes orientation; the profile example uses the numeric-bin alternative; the sector example uses abbreviated limits; and the proxy example declares `featureType: "Junction"`, not its dedicated schema constant. No end-to-end claim that all examples validate was made. [D5]

---

## Report Completion Checklist

- [x] Topic ID matches the overall index; topic plan is linked and aligned.
- [x] Core questions are covered or explicitly unresolved.
- [x] Findings have reproducible references and authority classifications.
- [x] Mutable sources identify commits or dated retrievals.
- [x] Access and evidence limitations are explicit.
- [x] Findings, interpretations and recommendations are distinguishable.
- [x] Material differences from accepted research are reconciled or identified for discussion.
- [x] Executive summary and recommendations are independently understandable.
- [x] Risks, open questions and success-criteria validation are recorded.
- [x] Next steps and ownership are stated.
- [ ] Plan-owner acceptance and date recorded before treating the topic as accepted downstream.

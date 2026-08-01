# Section 006: CSAPI Part 1 Requirement Baseline - Research Report

**Topic ID:** IDR-SRV-006<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-006 CSAPI Part 1 Requirement Baseline](../IDR%20Plans/idr-srv-006-csapi-part-1-requirement-baseline.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 31 detailed questions; all six methodology phases, eleven success criteria, fifteen deliverable requirements, and eleven minimum inventory fields are mapped<br>
**Methodology Used:** Direct extraction from the approved standard; requirement-class, conformance-class, schema, and abstract-test reconciliation; inherited-requirement review; server-boundary classification; pinned artifact audit; downstream and test-traceability analysis; and independent completeness and technical reviews<br>
**Research Time:** Approximately 2.5 hours of AI-assisted elapsed execution time, including parallel independent extraction and review, on July 31, 2026<br>
**Primary Sources:**

- OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0
- OGC 17-069r4, *OGC API - Features - Part 1: Core*, approved corrigendum Version 1.0.1
- OGC 19-072, *OGC API - Common - Part 1: Core*, Version 1.0.0
- Official OGC schema publication and the `opengeospatial/ogcapi-connected-systems` Version 1.0.0 tag

**Supporting Resources:** Accepted IDR-SRV-001 through IDR-SRV-005 reports, the Glaux Server Goal and Definition, the project governance corpus, OGC API - Features Part 4 development evidence, and official OGC development artifacts<br>
**Document Purpose:** Establish the traceable OGC API - Connected Systems Part 1 server-requirement baseline that later Glaux design, implementation, validation, conformance, and testing work must use<br>
**Author:** OpenAI Codex, with independent read-only requirement and artifact audits<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** July 31, 2026<br>
**Date:** July 31, 2026<br>
**Last Updated:** July 31, 2026

---

## How to Read This Report

For the project decision, read Sections 1, 12, 13.2, and 16. They explain what Part 1 requires, what Glaux should do with that baseline, what remains unresolved, and what happens next. Sections 3–11 and Appendix A are the audit trail: they are intentionally detailed so implementation work can cite exact requirements instead of asking an AI to reconstruct the standard from memory.

This report does **not** turn every detail into an immediate design decision. It separates four things throughout: a **standards obligation** is imposed by a normative source; a **source-backed finding** reports what an authoritative or informative artifact says; **analysis** explains its consequence; and a **project recommendation** proposes how Glaux should respond.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Requirement Extraction Methodology
5. Requirement-Class and Conformance-Class Inventory
6. Normative Requirement Inventory
7. Server Applicability and Boundary Classification
8. Resource and API Behavior Mapping
9. Schema, OpenAPI, and Representation Artifact Inventory
10. Downstream Topic Handoffs
11. Traceability and Test-Strategy Implications
12. Recommendations and Decision Analysis
13. Risks, Constraints, Ambiguities, and Open Questions
14. Validation Against the Research Plan
15. References
16. Next Steps and Handoff
17. Appendices

---

## 1. Executive Summary

### 1.1 Plain-English Decision Brief

OGC API - Connected Systems Part 1 is the approved standard that defines how a server publishes and manages connected-system **feature resources and descriptive resources**: Systems, nested Systems, Deployments, nested Deployments, Procedures, Sampling Features, and Property Definitions. It also defines common identifier and time rules, canonical and collection endpoints, hierarchy navigation, optional advanced filters, create/replace/delete and update operations, and GeoJSON and SensorML JSON encodings. Dynamic observations, streams, controls, and commands belong to Part 2 and were not researched here.

The approved Part 1 baseline contains **13 requirement classes, 103 numbered requirements, 5 numbered recommendations, 13 matching conformance classes, and 110 numbered abstract tests**. There is no single “Core” CSAPI class. The standard expects a server to implement at least one connected-system resource type and at least one applicable encoding, then declare the classes it actually supports. That flexibility defines the minimum a product may claim; it does not define the scope of a best-of-breed Glaux reference implementation.

For Glaux, the right direction is to retain all 13 classes in the implementation target, subject to the formal class-profile decision in IDR-SRV-008. That direction follows the accepted project goal and IDR-SRV-003 standards-package baseline. It is not a claim that every class is unconditionally mandatory under OGC 23-001. Some requirements apply only when their prerequisite resource or write class is selected. In particular, create/replace/delete and update inherit behavior from OGC API - Features Part 4, which is still a draft as of this research. Glaux can design and implement those capabilities, but it cannot honestly treat the draft dependency as a stable published-standard baseline without pinning an exact revision, tracking changes, and qualifying any conformance claim.

The official Part 1 schemas are required where formal requirements cite them. The official OpenAPI files are useful implementation aids, but the repository describes them as an example/template; they identify themselves as version `0.0.1`, include demo server URLs, and contain confirmed broken local references. They must not become Glaux's unreviewed API contract or its conformance oracle.

The published standard also contains material contradictions and defects: requirement-class and conformance-class prerequisite URIs disagree; several condition and cross-reference identifiers are stale or broken; GeoJSON and SensorML link-relation rules conflict with incorporated mapping-table footnotes; and some abstract tests are incomplete or can pass without proving the full requirement. None prevents IDR-SRV-007 from starting. They do require a controlled interpretation register, exact source pins, and later decisions in IDR-SRV-008, IDR-SRV-010 through IDR-SRV-014, and the conformance/test topics.

### 1.2 Principal Conclusions

1. **The controlling baseline is OGC 23-001 Version 1.0, not the mutable development branch or the example OpenAPI.** The report pins both the approved HTML and the corresponding repository tag so later work can reproduce the evidence.
2. **Part 1 contains 103 direct numbered requirements.** Every one is listed in §6 with an exact identifier, class, conformance mapping, applicability, affected behavior, artifact or handoff, and test implication.
3. **The baseline is modular.** No CSAPI Core class exists; each conformance claim is the set of classes actually implemented, including their prerequisites.
4. **Most Part 1 obligations are server obligations.** Publisher, Simulator, and clients interact through the resulting HTTP contracts, but Part 1 does not assign them Glaux Server's storage, routing, validation, or conformance responsibilities.
5. **Inherited behavior is substantial.** OGC API - Common Part 1 and OGC API - Features Part 1 supply landing-page, API-definition, conformance, HTTP, JSON, collections, item retrieval, paging, bounding-box, temporal, link, count, and default CRS behavior.
6. **Part 1 does not define every behavior later plans ask about.** It does not provide sorting or field-selection semantics, a complete error representation/status-code contract, authentication requirements, a final OpenAPI publication design, persistence/index architecture, or Part 2 dynamic-data behavior.
7. **The schemas and mapping tables matter independently of the prose summary.** When a formal requirement incorporates a table or named schema, that artifact is part of the obligation for the selected class.
8. **The normative abstract test suite is necessary but insufficient.** It maps every numbered requirement and also tests five recommendations, but identified gaps require additional negative, schema, integration, property-based, and project-policy tests.
9. **Published defects must remain visible.** Glaux should document interpretations and test both standards-faithful behavior and any compatibility alias; it should never silently rewrite the source baseline.
10. **IDR-SRV-007 is unblocked.** Part 2 can now be extracted separately using this report's resource and handoff boundary.

### 1.3 Recommendation for Plan-Owner Acceptance

Accept OGC 23-001 Version 1.0 as Glaux Server's controlling Part 1 requirement baseline; carry all 13 classes forward as the provisional full-reference-implementation target without treating them as an unqualified OGC minimum; preserve the 103-requirement trace inventory; pin every mutable dependency; treat the official OpenAPI as informative; and resolve identified publication conflicts through a recorded interpretation and compatibility process rather than silent invention.

Acceptance of this report accepts that research baseline. It does not yet freeze the final class profile, endpoint design, Rust architecture, persistence model, or conformance claim.

---

## 2. Scope and Plan Alignment

### 2.1 In Scope

- Direct extraction of OGC 23-001 Part 1 requirement and recommendation statements.
- Requirement-class, conformance-class, prerequisite, schema, mapping-table, and Annex A relationships.
- Inherited OGC API - Common and OGC API - Features behavior needed to understand Part 1 server obligations.
- Classification of obligations as direct, inherited, conditional, integration-contract, downstream-handoff, or outside this topic.
- Resource, endpoint, query, encoding, validation, and test implications.
- Reproducible source and artifact pins, known publication defects, and unresolved interpretation questions.

### 2.2 Explicitly Out of Scope

- Extracting Part 2 requirements or designing observations, datastreams, control streams, commands, or streaming behavior.
- Choosing the final Glaux conformance-class profile; that belongs to IDR-SRV-008.
- Finalizing endpoint, link, query, content-negotiation, error, or OpenAPI publication behavior; those belong to IDR-SRV-009 through IDR-SRV-014.
- Designing canonical Rust models, persistence, security, deployment, or test architecture.
- Treating examples, repository templates, or existing implementations as standards obligations.
- Repairing or choosing between contradictory standard statements without a recorded later decision.

### 2.3 Prior-Report Reconciliation

| Accepted Baseline | Input Used Here | Reconciliation |
|---|---|---|
| IDR-SRV-001 | STANAG/AEP obligation boundary | This report extracts only the adopted CSAPI Part 1 obligations and does not expand the NATO scope. |
| IDR-SRV-002 | Server functional responsibility boundary | Part 1 feature-resource contracts map to server responsibilities; ecosystem workflows remain external unless separately assigned. |
| IDR-SRV-003 | Four-standard package and full-reference direction | Part 1 is one part of the adopted package. The OGC minimum is modular; Glaux's broader target remains a project decision. |
| IDR-SRV-004 | Terminology crosswalk | Capitalized resource concepts follow OGC meanings; project terms do not silently alter normative definitions. |
| IDR-SRV-005 | Adjacent NATO standards boundary | No adjacent NATO publication adds a Part 1 requirement or changes this extraction. |

### 2.4 Research-Question Coverage

The plan contains 36 questions: five core questions and 31 detailed questions. The table gives the primary answer location; many questions are also supported by the full inventory in §6.

| ID | Research Question, Short Form | Answer Location |
|---|---|---|
| C1 | Which normative Part 1 requirements apply? | §§5–7; all 103 in §6 |
| C2 | Which classes, resources, schemas, and behaviors belong in the baseline? | §§5, 8, 9 |
| C3 | Which obligations are direct, inherited, conditional, or interpretive? | §§5–7, 13 |
| C4 | Which requirements affect downstream topics? | §§6, 10 |
| C5 | What trace inventory format should later work use? | §11.1 |
| RE1 | What requirement classes exist? | §5.1 |
| RE2 | What conformance classes are defined or referenced? | §§5.1–5.3 |
| RE3 | Which requirements are mandatory? | §§5.2, 6, 7 |
| RE4 | Which are optional/conditional and what triggers them? | §§5.1, 6, 7 |
| RE5 | What is normative versus recommendation/example? | §§3.2, 4.2, 6.8 |
| RE6 | Which requirements depend on OGC API - Features? | §§5.3, 6, 8.5 |
| RC1 | Landing page, API definition, conformance, discovery? | §§5.3, 8.1 |
| RC2 | Collections and feature resources? | §§6, 8.2–8.4 |
| RC3 | Systems, Procedures, Deployments, Sampling Features, Properties? | §§6, 8.2 |
| RC4 | Links, identifiers, relationships, nesting, navigation, references? | §§6.1–6.4, 8.3 |
| RC5 | Canonical model implications? | §§7, 10 (IDR-SRV-015–020) |
| QR1 | Retrieval, access, filtering, paging, sorting, selection, parameters? | §§6.5, 8.5 |
| QR2 | Which behavior is inherited from Features? | §§5.3, 8.5 |
| QR3 | Which behavior is CSAPI-specific? | §§6.1–6.7, 8 |
| QR4 | What must IDR-SRV-011 research? | §10 |
| QR5 | What affects persistence and indexing? | §§7, 10 |
| RS1 | What schemas and representation models are defined or referenced? | §9 |
| RS2 | What media types, JSON/GeoJSON, or linked patterns apply? | §§6.7, 8.6, 9 |
| RS3 | Which artifacts require direct review? | §§3, 9 |
| RS4 | What affects IDR-SRV-012 content negotiation? | §§8.6, 10 |
| RS5 | What affects IDR-SRV-023 validation? | §§9, 10, 11 |
| EV1 | What validation rules are implied? | §§6, 9, 11 |
| EV2 | What errors or failures are implied? | §§5.3, 8.7, 13 |
| EV3 | Which requirements need negative tests? | §11.2 |
| EV4 | Which feed conformance-harness planning? | §§5.2, 11.3 |
| EV5 | Which feed requirement-to-test traceability? | §§6, 11.1 |
| SB1 | Which requirements belong to Glaux Server? | §§7.1–7.2 |
| SB2 | What server contracts affect Publisher, Simulator, clients, or external systems? | §7.3 |
| SB3 | What is outside the server or deferred? | §§2.2, 7.3 |
| SB4 | Where does the standard leave project choices? | §§12, 13 |
| SB5 | What should later implementation studies check? | §§10, 12.3 |

---

## 3. Evidence Base and Authority Classification

### 3.1 Reproducible Source Register

| Source | Authority and Use | Reproducible Baseline |
|---|---|---|
| [OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html) | **Controlling normative source** for Part 1 requirements, incorporated tables/schemas, and Annex A | Version 1.0; approved 2025-06-02; published 2025-07-16; retrieved 2026-07-31; HTML SHA-256 `555C4B3BEA06AB91B980BDAA3C99D265E6718DBAD943CA1CBEC39FBBF283C92A` |
| [`ogcapi-connected-systems` v1.0.0](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0) | Official release repository; informative source files and implementation artifacts unless incorporated by the standard | Tag `v1.0.0`; commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`; 2025-07-16 |
| [Published Part 1 schemas](https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/) | Official schema publication; normative where OGC 23-001 explicitly requires validation against a named schema | Directory observed 2026-07-31; publication timestamp 2025-07-15 |
| [OGC 17-069r4](https://docs.ogc.org/is/17-069r4/17-069r4.html) | Normative inherited OGC API - Features Part 1 behavior | Approved corrigendum Version 1.0.1 |
| [OGC 19-072](https://docs.ogc.org/is/19-072/19-072.html) | Normative inherited OGC API - Common Part 1 behavior | Version 1.0.0 approved standard |
| [OGC API - Features Part 4 repository](https://github.com/opengeospatial/ogcapi-features) | Mutable development evidence for a normative dependency that Part 1 itself cites as draft | `master` commit `9ca25f56a58ed822ea8a685a7a41afa7181aaa8b`, dated 2026-07-13; draft identifies itself as `20-002r1`, `1.0.0-draft.2`, and not an OGC Standard |
| Connected Systems repository `master` | Mutable errata/development comparison only | Commit `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`, dated 2026-04-20 |

The release comparison found only three changed link-relation values in one example between `v1.0.0` and the observed Connected Systems `master`. That later change is useful evidence of an unresolved relation-name issue, but it cannot amend the approved standard.

### 3.2 Authority Rules Applied

| Evidence Kind | Treatment in This Report |
|---|---|
| Formal requirement table in OGC 23-001 | Normative obligation when its class is claimed and its condition is met |
| Prerequisite class named by a requirement-class table | Normative inherited obligation |
| Table or schema explicitly incorporated by a formal requirement | Normative to the extent incorporated |
| Formal recommendation table | Nonmandatory `SHOULD`; tracked separately from requirements |
| Annex A abstract test suite | Normative conformance evidence, but not a substitute for the requirement text |
| Descriptive prose, note, example, or figure | Informative interpretation evidence unless a requirement incorporates it |
| Official OpenAPI/example repository content | Informative implementation aid unless a formal requirement incorporates a named artifact |
| Mutable branch, issue, or later source edit | Informative errata/change evidence, pinned to a commit |
| This report's analysis or recommendation | Project decision input, never presented as an OGC obligation |

### 3.3 Evidence Limitations

- OGC 23-001 normatively depends on a draft OGC API - Features Part 4. There is no stable approved Part 4 baseline to cite as equivalent to the approved Part 1, so exact transactional inheritance remains change-controlled.
- Published requirement and conformance metadata contain contradictory or stale identifiers. The report records both rather than silently selecting one.
- The OpenAPI bundle is not a clean executable oracle: confirmed missing references prevent treating it as a release-ready complete contract.
- This topic inspected Part 2, SensorML, and SWE Common only far enough to identify dependencies and handoffs. It did not execute their requirement plans.
- Annex A sometimes under-tests a requirement. Passing the published abstract test is therefore evidence for conformance, not proof that every lettered obligation and incorporated artifact has been validated.

---

## 4. Requirement Extraction Methodology

### 4.1 Six-Phase Execution

1. **Source collection and framework:** pinned approved and mutable evidence; reconciled accepted prerequisite reports; defined inventory fields and classification codes.
2. **Normative extraction:** inspected every Part 1 clause, requirement/recommendation table, requirement-class table, encoding table, schema reference, and Annex A test.
3. **Applicability and boundary:** classified each item by server obligation, inherited behavior, condition, integration consequence, handoff, and uncertainty.
4. **Resource, schema, and API mapping:** grouped requirements by resource, route, method, query, representation, schema, and link behavior.
5. **Traceability and testing:** mapped all numbered statements to Annex A and identified positive, negative, schema, lifecycle, interoperability, and supplemental-test needs.
6. **Synthesis and review:** reconciled counts and contradictions, checked every plan question and deliverable, and subjected the draft to independent technical and completeness review.

### 4.2 Classification Framework

| Code | Meaning | How It Is Used |
|---|---|---|
| `D` | Direct Part 1 server obligation | A formal Part 1 requirement the server must meet when its class applies |
| `I` | Inherited obligation | Behavior supplied by a prerequisite or explicitly incorporated external clause |
| `C` | Explicitly conditional obligation | The requirement row has an activation condition beyond merely claiming its containing class |
| `R` | Formal recommendation | Nonmandatory `SHOULD`; test result should be advisory unless a Glaux profile elevates it |
| `H` | Downstream handoff | Implication belongs to a later research/design topic |
| `O` | Outside this topic | Part 2 or implementation design content not extracted here |
| `X` | Interpretation required | Published text is ambiguous, contradictory, broken, or leaves a project choice |

Every Requirement 1–103 is `D`; many also carry `I`, `C`, `H`, or `X`. All rows are already conditional on claiming their containing class, so `C` is reserved for an additional condition stated or intended by the requirement. No direct Part 1 requirement was classified as outside Glaux Server merely because its final class selection is deferred.

### 4.3 Inventory Field Model

The §6 inventory supplies the plan's minimum fields through a compact row plus its class context:

- exact numbered requirement ID and hyperlinkable source anchor;
- requirement and conformance class;
- normalized summary and condition;
- normative classification and Glaux applicability;
- affected resource/API area;
- related schema, OpenAPI area, or incorporated standard;
- downstream topic handoff;
- Annex A test and supplemental test implication; and
- notes for uncertainty or interpretation.

The normalized summary is not a substitute for the source. Implementation and test work must follow the linked requirement's complete lettered rows and incorporated tables/schemas.

---

## 5. Requirement-Class and Conformance-Class Inventory

### 5.1 Direct Part 1 Classes

The base URI for Part 1 requirement and conformance identifiers is `http://www.opengis.net/spec/ogcapi-connectedsystems-1/1.0`. “Direct requirements” below counts numbered Part 1 requirements, not inherited prerequisite requirements.

| # | Requirement Class → Conformance Class | Direct Content | Prerequisites Stated by Requirement Class | Applicability / Main Server Area |
|---:|---|---|---|---|
| 1 | `/req/api-common` → `/conf/api-common` | Req 1–3; Rec 1 | Features 1 Core; Common 1 Core, Landing Page, JSON | Required by every other Part 1 class; identifiers, temporal filtering, API foundation |
| 2 | `/req/system` → `/conf/system` | Req 4–8; Rec 2 | Part 1 API Common | Conditional class selection; System model, canonical and collection endpoints |
| 3 | `/req/subsystem` → `/conf/subsystem` | Req 9–13 | Part 1 System | Conditional on hierarchical Systems; nesting, recursion, aggregated associations |
| 4 | `/req/deployment` → `/conf/deployment` | Req 14–18 | Part 1 API Common | Conditional class selection; Deployment resources and System association |
| 5 | `/req/subdeployment` → `/conf/subdeployment` | Req 19–23 | Part 1 Deployment | Conditional on hierarchical Deployments |
| 6 | `/req/procedure` → `/conf/procedure` | Req 24–28 | Part 1 API Common | Conditional class selection; Procedure resources with no geometry |
| 7 | `/req/sf` → `/conf/sf` | Req 29–33 | Part 1 API Common and System | Conditional class selection; Sampling Features and mandatory System nesting |
| 8 | `/req/property` → `/conf/property` | Req 34–37 | Part 1 API Common | Conditional class selection; non-feature Property Definition resources |
| 9 | `/req/advanced-filtering` → `/conf/advanced-filtering` | Req 38–59; Rec 3–5 | Part 1 API Common; WKT grammar indirectly | Optional class; common and resource-specific filters on applicable endpoints |
| 10 | `/req/create-replace-delete` → `/conf/create-replace-delete` | Req 60–71 | Part 1 API Common; draft Features 4 Create/Replace/Delete | Optional transactional class; individual rows conditional on resource classes |
| 11 | `/req/update` → `/conf/update` | Req 72–76 | Part 1 Create/Replace/Delete; draft Features 4 Update | Optional PATCH class; individual rows conditional on resource classes |
| 12 | `/req/geojson` → `/conf/geojson` | Req 77–88 | Part 1 API Common; Features 1 GeoJSON | Encoding class for System, Deployment, Procedure, Sampling Feature; write conditional |
| 13 | `/req/sensorml` → `/conf/sensorml` | Req 89–103 | Part 1 API Common; four SensorML 3 JSON classes | Encoding class for System, Deployment, Procedure, Property; write conditional |

**Source-backed finding:** Part 1 §2 states that there is no Core requirements class and expects an implementation to support at least one connected-system resource type and one encoding. Any combination of resource types can be distributed.

**Analysis:** “Mandatory” therefore has two levels: prerequisites and numbered rows are mandatory **inside a claimed class**, while the set of claimed resource/optional classes is a profile decision. This is why Glaux's full-reference target must not be misstated as the OGC minimum.

### 5.2 Conformance and Abstract-Test Structure

- Annex A defines 13 conformance classes corresponding to the 13 requirement classes.
- It contains 110 numbered tests, A.1 through A.110.
- A.1 and A.2 are supporting endpoint/collection tests.
- Requirements 1–103 each have a one-to-one numbered test.
- Recommendations 1–5 also have tests, creating a structural distinction the harness must preserve. Glaux should report unmet recommendations as advisory/warning outcomes unless a later profile explicitly elevates them; the Annex methods are not uniformly warning-oriented.
- Class prerequisites in Annex A are not always identical to the normative requirement-class tables. The requirement text and class table control this extraction; discrepancies are registered in §13.2.

### 5.3 Inherited Approved Baseline

The direct inventory must be implemented together with applicable inherited requirements; it is not a self-contained HTTP API.

| Inherited Source / Class | Relevant Baseline | Consequence for Glaux |
|---|---|---|
| OGC API - Common Part 1 Core | HTTP 1.1; unknown/invalid query handling; capitalization, arrays/lists, booleans, integer/decimal/double parsing | Shared HTTP and parameter-processing contract; invalid or unknown parameters generally produce `400` as specified by the inherited source |
| Common Part 1 Landing Page | Root operation and links to API definition and conformance declaration | Landing/discovery behavior for IDR-SRV-009 |
| Common Part 1 JSON | JSON content and schema-definition behavior | Common JSON representation baseline |
| OGC API - Features Part 1 Core | Landing/API/conformance, HTTP, collections, collection items, single features, `limit`, `bbox`, `datetime`, paging links/counts, CRS84 | Foundation for all CS feature-resource endpoints |
| Features Part 1 GeoJSON | GeoJSON content and definition requirements | Prerequisite for Part 1 GeoJSON encoding |
| Features §§7.15.2–7.15.8 | Explicitly incorporated by Requirements 6, 15, 26, 30, and adapted by 35 | List retrieval, filtering, response, links, counts, paging, and documented failure behavior at CS resources endpoints |
| OGC 06-103r4 WKT | Geometry grammar referenced by Requirement 41 | `geom` parsing and spatial intersection behavior |
| SensorML 3 JSON classes | Four classes named by `/req/sensorml` | SensorML validation and resource-class behavior; detailed extraction deferred to IDR-SRV-021/023 |
| OGC API - Features Part 4 draft | Create/replace/delete and update semantics | Transactional dependency must be pinned and change-controlled; detailed HTTP semantics remain downstream work |

Appendix B records the inherited Common and Features requirement identifiers identified during this research. It prevents them from disappearing while avoiding a false claim that this topic re-executed those standards' full conformance plans.

---

## 6. Normative Requirement Inventory

### 6.0 Inventory Legend

All rows in §§6.1–6.7 are **normative Part 1 requirements** (`D`). `I` marks incorporated behavior from another standard; `C` marks a condition; `X` marks published text that needs recorded interpretation. “Glaux” means the requirement remains in the provisional full-reference target; final class selection is deferred to IDR-SRV-008. The linked ID is the exact approved-standard anchor. “ATS” is the corresponding Annex A test; text after `+` is an important supplemental test need. Formal recommendations are separated in §6.8.

### 6.1 API Common, Systems, and Subsystems

| No. / Exact Requirement ID | Class / Conf | Normalized Obligation and Condition | Glaux Classification; Affected Area | Artifact / Downstream Handoff | ATS and Supplemental Test |
|---|---|---|---|---|---|
| [1 · `/req/api-common/resource-ids`](https://docs.ogc.org/is/23-001/23-001.html#_req_api-common_resource-ids) | `api-common` / `api-common` | Local IDs are unique for each resource type across every collection containing that type. | `D`; identifier allocation, storage constraints, canonical routing | IDR-SRV-016, 025, 051 | A.3 + duplicate-ID rejection across collections |
| [2 · `/req/api-common/resource-uids`](https://docs.ogc.org/is/23-001/23-001.html#_req_api-common_resource-uids) | `api-common` / `api-common` | UIDs are valid URIs and unique across all server collections and resource types. | `D`; global UID index and URI validation | IDR-SRV-016, 025, 051 | A.4 + cross-type collision and malformed-URI negatives |
| [3 · `/req/api-common/datetime`](https://docs.ogc.org/is/23-001/23-001.html#_req_api-common_datetime) | `api-common` / `api-common` | CS feature filtering uses `validTime`; include interval intersections and features whose `validTime` is absent/null. | `D+I`; temporal query semantics on every CS feature endpoint | Features `datetime`; IDR-SRV-011, 018 | A.6 + boundary, open interval, absent/null, hierarchy cases |
| [4 · `/req/system/location-time`](https://docs.ogc.org/is/23-001/23-001.html#_req_system_location-time) | `system` / `system` | If a System location is reported, return its latest known location unless a Part 2 snapshot time is requested. | `D+C`; System geometry/current-state projection; Part 2 handoff | IDR-SRV-007, 018 | A.8 + prove latest value; snapshot exception later |
| [5 · `/req/system/canonical-url`](https://docs.ogc.org/is/23-001/23-001.html#_req_system_canonical-url) | `system` / `system` | Each System is retrievable at `/systems/{id}`; a representation obtained elsewhere links there with `rel=canonical`. | `D`; item route and canonical links | GeoJSON/SensorML link fields; IDR-SRV-010, 016, 017 | A.9 + root, nested, and custom-collection retrieval |
| [6 · `/req/system/resources-endpoint`](https://docs.ogc.org/is/23-001/23-001.html#_req_system_resources-endpoint) | `system` / `system` | Every System resources endpoint supports GET, satisfies Features §§7.15.2–7.15.8, and returns only Systems. | `D+I`; list/query/paging/response behavior | Features 1; OAS paths; IDR-SRV-010–013 | A.10 + all endpoint variants, wrong-type exclusion, negatives |
| [7 · `/req/system/canonical-endpoint`](https://docs.ogc.org/is/23-001/23-001.html#_req_system_canonical-endpoint) | `system` / `system` | Expose `/systems` and expose all Systems available on the server there. | `D+X`; canonical collection; conditionally refined by Req 11 hierarchy default | OAS `/systems`; IDR-SRV-010, 015 | A.11 + reconcile top-level default with `recursive=true` |
| [8 · `/req/system/collections`](https://docs.ogc.org/is/23-001/23-001.html#_req_system_collections) | `system` / `system` | Expose at least one collection of Systems; tag it `itemType=feature`, `featureType=sosa:System`; its items operation behaves as a System resources endpoint. | `D+I`; collection metadata and typed items | Features collection schemas; IDR-SRV-009, 010, 015 | A.12 + minimum-one/non-vacuous metadata and behavior tests |
| [9 · `/req/subsystem/collection`](https://docs.ogc.org/is/23-001/23-001.html#_req_subsystem_collection) | `subsystem` / `subsystem` | Expose `/systems/{parentId}/subsystems`; return that parent's permanently attached subsystems and, in the publication's uppercase wording, `CAN` also include currently deployed subsystems. | `D+X`; nested System collection; `CAN` is not the defined OGC permission keyword `MAY` | OAS nested path; IDR-SRV-010, 015, 017 | A.13 + membership, parent identity, deployed-subsystem interpretation |
| [10 · `/req/subsystem/recursive-param`](https://docs.ogc.org/is/23-001/23-001.html#_req_subsystem_recursive-param) | `subsystem` / `subsystem` | Support optional Boolean query parameter `recursive`. | `D`; query contract on hierarchy endpoints | OAS parameter; IDR-SRV-011, 014 | A.14 + advertisement, valid/invalid lexical values |
| [11 · `/req/subsystem/recursive-search-systems`](https://docs.ogc.org/is/23-001/23-001.html#_req_subsystem_recursive-search-systems) | `subsystem` / `subsystem` | On `/systems`, omitted/false returns top-level Systems; true includes every descendant; all other filters apply at all levels. | `D+X`; hierarchy traversal and filter propagation; refines Req 7 | IDR-SRV-010, 011, 015, 025 | A.15 + every filter across multiple depths and cycle protection |
| [12 · `/req/subsystem/recursive-search-subsystems`](https://docs.ogc.org/is/23-001/23-001.html#_req_subsystem_recursive-search-subsystems) | `subsystem` / `subsystem` | On a nested subsystem endpoint, omitted/false returns direct children; true returns all descendants; other filters apply at all levels. | `D`; nested hierarchy traversal/query | IDR-SRV-010, 011, 015, 025 | A.16 + multi-depth membership and filter propagation |
| [13 · `/req/subsystem/recursive-assoc`](https://docs.ogc.org/is/23-001/23-001.html#_req_subsystem_recursive-assoc) | `subsystem` / `subsystem` | A System with subsystems exposes association targets as Table 9 specifies, aggregating relevant descendant targets. | `D`; relationship materialization; Part 2 associations are handoffs | Table 9; IDR-SRV-007, 017, 025 | A.17 + aggregate/deduplicate nested association targets |

Model tables in Clauses 9–10 also establish conceptual content used by the encoding requirements; they are not additional numbered ModSpec requirements. System attributes marked Required are `uniqueIdentifier`, `name`, and `systemType`; associations marked Required are `subsystems`, `samplingFeatures`, `datastreams`, and `controlstreams`. A Subsystem adds required `parentSystem`. Table 6 System types are `sosa:Sensor`, `sosa:Actuator`, `sosa:Sampler`, `sosa:Platform`, and `sosa:System`; Table 7 asset types are `Equipment`, `Human`, `LivingThing`, `Simulation`, `Process`, `Group`, and `Other`. Table 9 makes `samplingFeatures`, `datastreams`, and `controlstreams` recursive association targets. Exact JSON presence/cardinality still requires the selected encoding and schema, especially because some “Required” association descriptions also say “if any.”

### 6.2 Deployments, Subdeployments, and Procedures

| No. / Exact Requirement ID | Class / Conf | Normalized Obligation and Condition | Glaux Classification; Affected Area | Artifact / Downstream Handoff | ATS and Supplemental Test |
|---|---|---|---|---|---|
| [14 · `/req/deployment/canonical-url`](https://docs.ogc.org/is/23-001/23-001.html#_req_deployment_canonical-url) | `deployment` / `deployment` | Each Deployment is retrievable at `/deployments/{id}`; representations obtained elsewhere link there as canonical. | `D`; item route and canonical links | Encoding link fields; IDR-SRV-010, 016, 017 | A.18 + root, nested, custom-collection retrieval |
| [15 · `/req/deployment/resources-endpoint`](https://docs.ogc.org/is/23-001/23-001.html#_req_deployment_resources-endpoint) | `deployment` / `deployment` | Each Deployment resources endpoint supports GET, inherits Features §§7.15.2–7.15.8, and returns only Deployments. | `D+I`; list/query/paging/response behavior | Features 1; OAS paths; IDR-SRV-010–013 | A.19 + endpoint variants, wrong-type and invalid-query negatives |
| [16 · `/req/deployment/canonical-endpoint`](https://docs.ogc.org/is/23-001/23-001.html#_req_deployment_canonical-endpoint) | `deployment` / `deployment` | Expose `/deployments` and expose all Deployments available on the server there. | `D+X`; canonical collection; conditionally refined by Req 21 | OAS `/deployments`; IDR-SRV-010, 015 | A.20 + top-level/default versus recursive reconciliation |
| [17 · `/req/deployment/ref-from-system`](https://docs.ogc.org/is/23-001/23-001.html#_req_deployment_ref-from-system) | `deployment` / `deployment` | If Systems and the `deployments` association are provided, link it to `/systems/{sysId}/deployments`, returning only Deployments where that System was deployed. | `D+C+X`; cross-resource navigation; ATS condition mismatch | OAS nested path; IDR-SRV-010, 017 | A.22 + condition-off path, membership, all encodings |
| [18 · `/req/deployment/collections`](https://docs.ogc.org/is/23-001/23-001.html#_req_deployment_collections) | `deployment` / `deployment` | Expose at least one Deployment collection; tag it `itemType=feature`, `featureType=sosa:Deployment`; items behave as a Deployment endpoint. | `D+I`; collection metadata and typed items | Features collection schemas; IDR-SRV-009, 010, 015 | A.21 + minimum-one/non-vacuous checks |
| [19 · `/req/subdeployment/collection`](https://docs.ogc.org/is/23-001/23-001.html#_req_subdeployment_collection) | `subdeployment` / `subdeployment` | Expose `/deployments/{parentId}/subdeployments`; members belong to the parent and the endpoint supports the same query parameters actually supported by `/deployments`. | `D`; nested Deployment endpoint/query parity | OAS nested path; IDR-SRV-010, 011, 015 | A.23 + membership and full query-parity tests |
| [20 · `/req/subdeployment/recursive-param`](https://docs.ogc.org/is/23-001/23-001.html#_req_subdeployment_recursive-param) | `subdeployment` / `subdeployment` | Support optional Boolean query parameter `recursive`. | `D`; hierarchy query contract | OAS parameter; IDR-SRV-011, 014 | A.24 + advertisement and invalid lexical values |
| [21 · `/req/subdeployment/recursive-search-deployments`](https://docs.ogc.org/is/23-001/23-001.html#_req_subdeployment_recursive-search-deployments) | `subdeployment` / `subdeployment` | On `/deployments`, omitted/false returns top-level Deployments; true includes descendants; other filters apply at every level. | `D+X`; hierarchy traversal/filtering; refines Req 16 | IDR-SRV-010, 011, 015, 025 | A.25 + all filters over multiple depths |
| [22 · `/req/subdeployment/recursive-search-subdeployments`](https://docs.ogc.org/is/23-001/23-001.html#_req_subdeployment_recursive-search-subdeployments) | `subdeployment` / `subdeployment` | Nested endpoint omitted/false returns direct children; true returns all descendants; other filters apply at every level. | `D`; nested hierarchy traversal/filtering | IDR-SRV-010, 011, 015, 025 | A.26 + multi-depth membership/filter propagation |
| [23 · `/req/subdeployment/recursive-assoc`](https://docs.ogc.org/is/23-001/23-001.html#_req_subdeployment_recursive-assoc) | `subdeployment` / `subdeployment` | A Deployment with children exposes association targets as Table 13 specifies, aggregating relevant descendant targets. | `D`; relationship aggregation; Part 2 links handed off | Table 13; IDR-SRV-007, 017, 025 | A.27 + aggregate/deduplicate all defined targets |
| [24 · `/req/procedure/location`](https://docs.ogc.org/is/23-001/23-001.html#_req_procedure_location) | `procedure` / `procedure` | A Procedure representation has no location or geometry. | `D`; resource model and encoding validation | GeoJSON/SensorML schemas; IDR-SRV-015, 021, 023 | A.28 + reject/omit geometry and SensorML position |
| [25 · `/req/procedure/canonical-url`](https://docs.ogc.org/is/23-001/23-001.html#_req_procedure_canonical-url) | `procedure` / `procedure` | Each Procedure is retrievable at `/procedures/{id}`, where `id` is its local ID; representations obtained elsewhere link there as canonical. | `D`; item route and canonical links | Encoding link fields; IDR-SRV-010, 016, 017 | A.29 + alternate endpoint retrieval |
| [26 · `/req/procedure/resources-endpoint`](https://docs.ogc.org/is/23-001/23-001.html#_req_procedure_resources-endpoint) | `procedure` / `procedure` | Every Procedure resources endpoint supports GET, inherits Features §§7.15.2–7.15.8, and returns only Procedures. | `D+I`; list/query/paging/response behavior | Features 1; OAS paths; IDR-SRV-010–013 | A.30 + endpoint variants and negative cases |
| [27 · `/req/procedure/canonical-endpoint`](https://docs.ogc.org/is/23-001/23-001.html#_req_procedure_canonical-endpoint) | `procedure` / `procedure` | Expose `/procedures` containing all server-held Procedures. | `D`; canonical collection | OAS `/procedures`; IDR-SRV-010, 015 | A.31 + complete-discovery fixture |
| [28 · `/req/procedure/collections`](https://docs.ogc.org/is/23-001/23-001.html#_req_procedure_collections) | `procedure` / `procedure` | Expose at least one Procedure collection; tag it `itemType=feature`, `featureType=sosa:Procedure`; items behave as a Procedure endpoint. | `D+I`; collection metadata/typed items | Features collection schemas; IDR-SRV-009, 010, 015 | A.32 + minimum-one/non-vacuous checks |

Deployment model tables mark `uniqueIdentifier`, `name`, and `validTime` as required attributes and `deployedSystems` and `subdeployments` as required associations; a Subdeployment adds required `parentDeployment`. Table 13 recursively aggregates `deployedSystems`, `samplingFeatures`, `featuresOfInterest`, `datastreams`, and `controlstreams`. As with Systems, some association rows are marked “Required” while saying “if any,” which is retained as an interpretation issue rather than silently normalized.

Procedure model tables require `uniqueIdentifier`, `name`, and `procedureType`, allow description, valid time, and implementing Systems, and enumerate SOSA/SensorML-aligned Procedure types. Those tables inform the encoding mappings but do not create additional numbered requirements.

### 6.3 Sampling Features and Property Definitions

| No. / Exact Requirement ID | Class / Conf | Normalized Obligation and Condition | Glaux Classification; Affected Area | Artifact / Downstream Handoff | ATS and Supplemental Test |
|---|---|---|---|---|---|
| [29 · `/req/sf/canonical-url`](https://docs.ogc.org/is/23-001/23-001.html#_req_sf_canonical-url) | `sf` / `sf` | Each Sampling Feature is retrievable at `/samplingFeatures/{id}`, where `id` is its local ID; representations obtained elsewhere link there as canonical. | `D`; item route and canonical links | GeoJSON links; IDR-SRV-010, 016, 017 | A.33 + nested and custom-collection retrieval |
| [30 · `/req/sf/resources-endpoint`](https://docs.ogc.org/is/23-001/23-001.html#_req_sf_resources-endpoint) | `sf` / `sf` | Every Sampling Feature resources endpoint supports GET, inherits Features §§7.15.2–7.15.8, and returns only Sampling Features. | `D+I`; list/query/paging/response | Features 1; OAS paths; IDR-SRV-010–013 | A.34 + root/nested/typed endpoint negatives |
| [31 · `/req/sf/canonical-endpoint`](https://docs.ogc.org/is/23-001/23-001.html#_req_sf_canonical-endpoint) | `sf` / `sf` | Expose `/samplingFeatures` containing all server-held Sampling Features. | `D`; canonical collection | OAS path; IDR-SRV-010, 015 | A.35 + complete-discovery fixture |
| [32 · `/req/sf/ref-from-system`](https://docs.ogc.org/is/23-001/23-001.html#_req_sf_ref-from-system) | `sf` / `sf` | Every System has `/systems/{sysId}/samplingFeatures`, where `sysId` is its local ID; it returns only associated Sampling Features and the System association links to it. | `D`; mandatory cross-resource nested endpoint | OAS nested path; IDR-SRV-010, 017 | A.37 + every System, empty/nonempty membership, link correctness |
| [33 · `/req/sf/collections`](https://docs.ogc.org/is/23-001/23-001.html#_req_sf_collections) | `sf` / `sf` | Expose at least one Sampling Feature collection; tag it `itemType=feature`, `featureType=sosa:Sample`; items behave as an SF endpoint. | `D+I`; collection metadata/typed items | GeoJSON collection schema; IDR-SRV-009, 010, 015 | A.36 + minimum-one/non-vacuous checks |
| [34 · `/req/property/canonical-url`](https://docs.ogc.org/is/23-001/23-001.html#_req_property_canonical-url) | `property` / `property` | Each Property is retrievable at `/properties/{id}`, where `id` is its local ID; representations obtained elsewhere link there as canonical. | `D`; non-feature item route and links | SensorML links; IDR-SRV-010, 016, 017 | A.38 + alternate endpoint retrieval |
| [35 · `/req/property/resources-endpoint`](https://docs.ogc.org/is/23-001/23-001.html#_req_property_resources-endpoint) | `property` / `property` | Each Property resources endpoint supports GET and adapted Features §§7.15.2–7.15.8 behavior, replacing feature terminology with resource terminology; return only Properties. | `D+I`; list/query/paging for non-features | Features behavior adapted; IDR-SRV-010–013 | A.39 + non-feature response and invalid-query tests |
| [36 · `/req/property/canonical-endpoint`](https://docs.ogc.org/is/23-001/23-001.html#_req_property_canonical-endpoint) | `property` / `property` | Expose `/properties` containing all server-held Properties. | `D`; canonical non-feature collection | OAS path; IDR-SRV-010, 015 | A.40 + complete-discovery fixture |
| [37 · `/req/property/collections`](https://docs.ogc.org/is/23-001/23-001.html#_req_property_collections) | `property` / `property` | Expose at least one Property collection; tag it `itemType=sosa:Property`; items behave as a Property endpoint. | `D+I`; resource collection metadata/items | SensorML property schemas; IDR-SRV-009, 010, 015 | A.41 + minimum-one/non-vacuous checks |

Sampling Feature model tables require identity, name, feature type, parent System, and sampled-feature relationships. Property Definitions are non-feature resources and require identity, name, and base property. Clause 14.7's dynamic-property snapshot discussion is a Part 2 handoff, not an additional Part 1 numbered obligation.

### 6.4 Advanced Filtering

Claiming `/req/advanced-filtering` applies its relevant filters to **all** CS resources endpoints offered by the server, including canonical, nested, and typed/custom collection endpoints. Values within an identifier list or keyword search use the matching semantics stated below; different filters in the same request combine with logical AND. Every named association filter in Requirements 42–58 uses `ID_List`. The standard does not define sorting, field selection, or an offset/cursor algorithm.

| No. / Exact Requirement ID | Class / Conf | Normalized Obligation and Condition | Glaux Classification; Affected Area | Artifact / Downstream Handoff | ATS and Supplemental Test |
|---|---|---|---|---|---|
| [38 · `/req/advanced-filtering/id-list-schema`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_id-list-schema) | `advanced-filtering` / same | `ID_List` is a nonempty homogeneous array of either nonempty local-ID strings or URI-form UIDs; every item is a valid local-ID or UID value and the kinds cannot be mixed. | `D+X`; shared parameter parsing; serialization details incomplete | OAS parameter schemas; IDR-SRV-011, 014, 016 | A.42 + empty, mixed, malformed URI, unknown-as-no-match, serialization cases |
| [39 · `/req/advanced-filtering/resource-by-id`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_resource-by-id) | `advanced-filtering` / same | Every CS resources endpoint supports `id: ID_List`; return only requested IDs/UIDs; a UID ending `*` is a prefix match. | `D`; universal identity filter and index | OAS `id`; IDR-SRV-011, 016, 025 | A.43 + local/UID lists, wildcard placement, all endpoint types |
| [40 · `/req/advanced-filtering/resource-by-keyword`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_resource-by-keyword) | `advanced-filtering` / same | Every CS resources endpoint supports `q`; one or more 1–50-character terms, `explode=false`; match at least one term in human-readable content and always search name/description. | `D`; full-text query; optional canonicalization/lemmatization | OAS `q`; IDR-SRV-011, 025 | A.44 + OR semantics, length, absent fields, Unicode/case policy |
| [41 · `/req/advanced-filtering/feature-by-geom`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_feature-by-geom) | `advanced-filtering` / same | Geometry-bearing endpoints support WKT `geom`; return spatial intersections, exclude geometry-less features, and respect the selected CRS extent. | `D+I`; spatial parsing/query; server chooses relevant geometry when several exist | OGC WKT; IDR-SRV-011, 026 | A.46 + invalid WKT/CRS extent/no-geometry/multiple-geometry policy |
| [42 · `/req/advanced-filtering/system-by-parent`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_system-by-parent) | `advanced-filtering` / same | System endpoints support `parent: ID_List`; return only subsystems of a requested parent. | `D`; hierarchy relationship query | OAS parameter; IDR-SRV-011, 017, 025 | A.47 + root/nested/custom endpoints and nonparent negatives |
| [43 · `/req/advanced-filtering/system-by-procedure`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_system-by-procedure) | `advanced-filtering` / same | System endpoints support `procedure: ID_List`; return Systems implementing a requested Procedure. | `D`; System–Procedure join | IDR-SRV-011, 017, 025 | A.48 + local/UID, multiple values, absent relation |
| [44 · `/req/advanced-filtering/system-by-foi`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_system-by-foi) | `advanced-filtering` / same | `foi` returns Systems observing/controlling a requested Sampling or domain feature; include a parent if any nested subsystem matches. | `D`; recursive feature-of-interest join | IDR-SRV-007, 011, 017, 025 | A.49 + direct/indirect resource types and nested-parent inclusion |
| [45 · `/req/advanced-filtering/system-by-obsprop`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_system-by-obsprop) | `advanced-filtering` / same | `observedProperty` returns capable Systems; include a parent when any recursively nested subsystem matches. | `D`; recursive capability/property join | IDR-SRV-007, 011, 017, 024, 025 | A.50 + descendant-only capability and derived-property policy |
| [46 · `/req/advanced-filtering/system-by-controlprop`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_system-by-controlprop) | `advanced-filtering` / same | `controlledProperty` returns capable Systems; include a parent when any recursively nested subsystem matches. | `D`; recursive control-capability join | IDR-SRV-007, 011, 017, 024, 025 | A.51 + descendant-only capability and derived-property policy |
| [47 · `/req/advanced-filtering/deployment-by-parent`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_deployment-by-parent) | `advanced-filtering` / same | Deployment endpoints support `parent: ID_List`; return Deployments belonging to a requested parent. | `D`; Deployment hierarchy query | IDR-SRV-011, 017, 025 | A.52 + all endpoint variants and nonparent negatives |
| [48 · `/req/advanced-filtering/deployment-by-system`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_deployment-by-system) | `advanced-filtering` / same | `system` returns Deployments in which a requested System is deployed. | `D`; Deployment–System join | IDR-SRV-011, 017, 025 | A.53 + local/UID/multiple values and absent relation |
| [49 · `/req/advanced-filtering/deployment-by-foi`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_deployment-by-foi) | `advanced-filtering` / same | `foi` returns Deployments whose deployed Systems observe/control a requested Sampling or domain feature. | `D`; transitive Deployment–System–FOI join | IDR-SRV-007, 011, 017, 025 | A.54 + both feature kinds and negative transitive cases |
| [50 · `/req/advanced-filtering/deployment-by-obsprop`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_deployment-by-obsprop) | `advanced-filtering` / same | `observedProperty` returns Deployments where a deployed System observes a requested Property. | `D`; transitive observed-property join | IDR-SRV-007, 011, 024, 025 | A.55 + multiple Systems/properties and indirect-property policy |
| [51 · `/req/advanced-filtering/deployment-by-controlprop`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_deployment-by-controlprop) | `advanced-filtering` / same | `controlledProperty` returns Deployments where a deployed System controls a requested Property. | `D`; transitive controlled-property join | IDR-SRV-007, 011, 024, 025 | A.56 + multiple Systems/properties and indirect-property policy |
| [52 · `/req/advanced-filtering/procedure-by-obsprop`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_procedure-by-obsprop) | `advanced-filtering` / same | Procedure endpoints support `observedProperty`; return Procedures usable to observe a requested Property. | `D`; Procedure capability query | IDR-SRV-011, 017, 024, 025 | A.57 + local/UID/multiple and no-capability cases |
| [53 · `/req/advanced-filtering/procedure-by-controlprop`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_procedure-by-controlprop) | `advanced-filtering` / same | `controlledProperty` returns Procedures usable to control a requested Property. | `D`; Procedure capability query | IDR-SRV-011, 017, 024, 025 | A.58 + local/UID/multiple and no-capability cases |
| [54 · `/req/advanced-filtering/sf-by-foi`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_sf-by-foi) | `advanced-filtering` / same | Sampling Feature endpoints support `foi`; return Sampling Features associated with a requested feature of interest. | `D+X`; relationship query; illustrative scope omits required nested endpoint | IDR-SRV-011, 017, 025 | A.59 + root/nested/typed endpoints and indirect-sample policy |
| [55 · `/req/advanced-filtering/sf-by-obsprop`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_sf-by-obsprop) | `advanced-filtering` / same | `observedProperty` returns Sampling Features having a requested observed Property. | `D`; Sampling Feature–Property join | IDR-SRV-007, 011, 024, 025 | A.60 + root/nested/typed and derived-property policy |
| [56 · `/req/advanced-filtering/sf-by-controlprop`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_sf-by-controlprop) | `advanced-filtering` / same | `controlledProperty` returns Sampling Features having a requested controlled Property. | `D`; Sampling Feature–Property join | IDR-SRV-007, 011, 024, 025 | A.61 + root/nested/typed and derived-property policy |
| [57 · `/req/advanced-filtering/prop-by-baseprop`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_prop-by-baseprop) | `advanced-filtering` / same | Property endpoints support `baseProperty`; return Properties derived directly or transitively from a requested base Property. | `D`; transitive Property graph query | IDR-SRV-011, 017, 024, 025 | A.62 + multi-hop derivation, cycle protection, base self-policy |
| [58 · `/req/advanced-filtering/prop-by-object`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_prop-by-object) | `advanced-filtering` / same | `objectType` returns Properties whose object type is requested. | `D+X`; semantic-type query; published examples use wrong `object` name | IDR-SRV-011, 024, 025 | A.63 + reject/alias policy for erroneous example parameter |
| [59 · `/req/advanced-filtering/combined-filters`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_combined-filters) | `advanced-filtering` / same | Combine different filters in one request using logical AND. | `D`; query planner/composition | IDR-SRV-011, 025, 051 | A.64 + pairwise/multi-filter, empty-intersection, hierarchy cases |

---

### 6.5 Create, Replace, Delete, and Update

Every row in these classes inherits the applicable transaction operation from the draft OGC API - Features Part 4: create by POST, replace by PUT, delete by DELETE, and update by PATCH. Part 1 specializes the CS resource URLs and adds hierarchy, cascade, and collection-membership rules. The dependency's draft status is a baseline risk, not permission to omit these direct Part 1 requirements if Glaux claims the classes.

| No. / Exact Requirement ID | Class / Conf | Normalized Obligation and Condition | Glaux Classification; Affected Area | Artifact / Downstream Handoff | ATS and Supplemental Test |
|---|---|---|---|---|---|
| [60 · `/req/create-replace-delete/system`](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_system) | `create-replace-delete` / same | If Systems are implemented, create at `/systems`; replace/delete at `/systems/{id}`, using local ID. | `D+I+C`; System transaction routes | Draft Features 4; OAS requests; IDR-SRV-010, 012, 013, 031 | A.67 + method/media/status/validation/id lifecycle |
| [61 · `/req/create-replace-delete/system-delete-cascade`](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_system-delete-cascade) | `create-replace-delete` / same | If Systems are implemented, reject deletion without `cascade` when nested resources or Deployment associations exist; with it, delete the System and named descendants and remove Deployment links. | `D+I+C+X`; graph-safe cascade; parameter value semantics under-specified | IDR-SRV-013, 017, 029, 030, 031 | A.68 + atomic rollback, false/duplicate values, every descendant/link |
| [62 · `/req/create-replace-delete/subsystem`](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_subsystem) | `create-replace-delete` / same | If Subsystems are implemented, POST to `/systems/{parentId}/subsystems` creates a System associated with that parent. | `D+I+C`; nested create and parent linkage | Draft Features 4; IDR-SRV-017, 029, 031 | A.69 + parent missing/forbidden/cycle/atomicity |
| [63 · `/req/create-replace-delete/deployment`](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_deployment) | `create-replace-delete` / same | If Deployments are implemented, create at `/deployments`; replace/delete at `/deployments/{id}`, where `id` is the local Deployment ID. | `D+I+C`; Deployment transaction routes | Draft Features 4; OAS requests; IDR-SRV-010, 012, 013, 031 | A.70 + method/media/status/validation/id lifecycle |
| [64 · `/req/create-replace-delete/subdeployment`](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_subdeployment) | `create-replace-delete` / same | If Subdeployments are implemented, POST to `/deployments/{parentId}/subdeployments` creates a Deployment associated with that parent. | `D+I+C`; nested create and parent linkage | Draft Features 4; IDR-SRV-017, 029, 031 | A.71 + correct Deployment URL, parent/cycle/atomicity; ATS typo guard |
| [65 · `/req/create-replace-delete/procedure`](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_procedure) | `create-replace-delete` / same | If Procedures are implemented, create at `/procedures`; replace/delete at `/procedures/{id}`, where `id` is the local Procedure ID. | `D+I+C`; Procedure transaction routes | Draft Features 4; OAS requests; IDR-SRV-010, 012, 013, 031 | A.72 + method/media/status/schema/id lifecycle |
| [66 · `/req/create-replace-delete/sampling-feature`](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_sampling-feature) | `create-replace-delete` / same | If Sampling Features are implemented, create under `/systems/{sysId}/samplingFeatures`, where `sysId` is the parent System local ID; replace/delete at `/samplingFeatures/{id}`, where `id` is the Sampling Feature local ID. | `D+I+C`; parent-bound create and canonical mutation | Draft Features 4; GeoJSON; IDR-SRV-017, 029, 031 | A.73 + parent integrity, schema, membership, authorization handoff |
| [67 · `/req/create-replace-delete/property`](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_property) | `create-replace-delete` / same | Create at `/properties`; replace/delete at `/properties/{id}`, where `id` is the local Property ID, when the intended Property class condition applies. | `D+I+C+X`; Property transactions; published condition link is broken | Draft Features 4; SensorML; IDR-SRV-008, 013, 031 | A.74 + condition interpretation, schema/id lifecycle |
| [68 · `/req/create-replace-delete/create-in-collection`](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_create-in-collection) | `create-replace-delete` / same | Every successfully created CS resource appears in its type's root collection. | `D+I`; write/read coherence and indexing | IDR-SRV-010, 025, 029, 031 | A.75 + immediate/transactional visibility and paging |
| [69 · `/req/create-replace-delete/replace-in-collection`](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_replace-in-collection) | `create-replace-delete` / same | A successful replacement is reflected in every collection containing the resource. | `D+I`; multi-collection consistency | IDR-SRV-025, 029, 031 | A.76 + concurrent read, all memberships, rollback |
| [70 · `/req/create-replace-delete/delete-in-collection`](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_delete-in-collection) | `create-replace-delete` / same | Deleting from a root removes the resource from all collections; deleting through a non-root collection removes only that membership. | `D+I`; resource lifecycle versus collection membership | IDR-SRV-016, 025, 029, 030, 031 | A.77 + root/non-root, last membership, canonical retrieval semantics |
| [71 · `/req/create-replace-delete/add-to-collection`](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_add-to-collection) | `create-replace-delete` / same | Add existing resources by POSTing `text/uri-list`, one canonical URL or UID per line, referencing resources at the same API endpoint. | `D+I`; collection membership and URI validation | OAS request; IDR-SRV-012, 013, 016, 029, 031 | A.78 + media type, line grammar, remote/unknown/mixed/duplicate refs |
| [72 · `/req/update/system`](https://docs.ogc.org/is/23-001/23-001.html#_req_update_system) | `update` / `update` | If Systems are implemented, PATCH `/systems/{id}` using the local ID. | `D+I+C`; partial System mutation | Draft Features 4; IDR-SRV-012, 013, 029, 031 | A.79 + patch format, validation, concurrency, immutable fields |
| [73 · `/req/update/deployment`](https://docs.ogc.org/is/23-001/23-001.html#_req_update_deployment) | `update` / `update` | If Deployments are implemented, PATCH `/deployments/{id}`, where `id` is the local Deployment ID. | `D+I+C`; partial Deployment mutation | Draft Features 4; IDR-SRV-012, 013, 029, 031 | A.80 + patch format, validation, concurrency, relationships |
| [74 · `/req/update/procedure`](https://docs.ogc.org/is/23-001/23-001.html#_req_update_procedure) | `update` / `update` | If Procedures are implemented, PATCH `/procedures/{id}`, where `id` is the local Procedure ID. | `D+I+C`; partial Procedure mutation | Draft Features 4; IDR-SRV-012, 013, 029, 031 | A.81 + patch format, schema, no-location invariant |
| [75 · `/req/update/sampling-feature`](https://docs.ogc.org/is/23-001/23-001.html#_req_update_sampling-feature) | `update` / `update` | If Sampling Features are implemented, PATCH `/samplingFeatures/{id}`, where `id` is the local Sampling Feature ID. | `D+I+C`; partial Sampling Feature mutation | Draft Features 4; IDR-SRV-012, 013, 029, 031 | A.82 + parent/relationship integrity and schema |
| [76 · `/req/update/property`](https://docs.ogc.org/is/23-001/23-001.html#_req_update_property) | `update` / `update` | PATCH `/properties/{id}` when the intended Property class condition applies. | `D+I+C+X`; partial Property mutation; broken condition and missing explicit ID row | Draft Features 4; IDR-SRV-008, 013, 029, 031 | A.83 + ATS fixture-name defect, condition and ID interpretation |

---

### 6.6 GeoJSON Encoding

Schema base: [`https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/openapi/schemas/geojson/`](https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/openapi/schemas/geojson/). Part 1 provides GeoJSON schemas for System, Deployment, Procedure, and Sampling Feature—not Property Definitions.

| No. / Exact Requirement ID | Class / Conf | Normalized Obligation and Condition | Glaux Classification; Affected Area | Artifact / Downstream Handoff | ATS and Supplemental Test |
|---|---|---|---|---|---|
| [77 · `/req/geojson/mediatype-read`](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson_mediatype-read) | `geojson` / `geojson` | Support `application/geo+json` for reading the applicable resource type and return valid JSON. | `D+I`; representation negotiation/serialization | Features GeoJSON; IDR-SRV-012, 014 | A.84 + Accept q-values, unsupported media, alternate link |
| [78 · `/req/geojson/mediatype-write`](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson_mediatype-write) | `geojson` / `geojson` | If Create/Replace/Delete is implemented, accept and parse `application/geo+json` request bodies for applicable resources. | `D+I+C`; request negotiation/validation | GeoJSON schemas; IDR-SRV-012, 013, 023, 031 | A.85 + malformed JSON/schema/415 and media parameters |
| [79 · `/req/geojson/relation-types`](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson_relation-types) | `geojson` / `geojson` | Associations represented in `links` use the association name as `rel`. | `D+X`; link serialization; conflicts with incorporated mapping footnotes requiring `ogc-rel:` prefix | Tables 40–47; IDR-SRV-008, 010, 017 | A.86 + interpretation/compatibility variants |
| [80 · `/req/geojson/feature-attribute-mapping`](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson_feature-attribute-mapping) | `geojson` / `geojson` | Apply Table 39: UID → `properties.uid` URI, name → `properties.name`, description → `properties.description`. | `D`; common GeoJSON mapping | Table 39/common schema; IDR-SRV-015, 016, 023 | A.87 + required/optional fields and round trip |
| [81 · `/req/geojson/system-schema`](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson_system-schema) | `geojson` / `geojson` | Single Systems and collections validate against `system.json` and `systemCollection.json`. | `D+X`; System representation; explicit class condition absent | Normative named schemas; IDR-SRV-008, 021, 023 | A.88 + local pinned schema, invalid/golden corpus |
| [82 · `/req/geojson/system-mappings`](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson_system-mappings) | `geojson` / `geojson` | Implement Tables 40–41 for System type/asset/time/location and all defined associations. | `D+X`; System serialization and relationships; relation-prefix conflict | Tables 40–41; IDR-SRV-015–018, 021, 023 | A.89 + every mapping row, null/optional, link relations |
| [83 · `/req/geojson/deployment-schema`](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson_deployment-schema) | `geojson` / `geojson` | Single Deployments and collections validate against `deployment.json` and `deploymentCollection.json`. | `D+X`; Deployment representation; explicit class condition absent | Normative named schemas; IDR-SRV-008, 021, 023 | A.90 + local pinned schema, invalid/golden corpus |
| [84 · `/req/geojson/deployment-mappings`](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson_deployment-mappings) | `geojson` / `geojson` | Implement Tables 42–43 for Deployment type/time/geometry and platform, System, hierarchy, feature, stream, and control associations. | `D+X`; representation/relationships; Part 2 links handed off | Tables 42–43; IDR-SRV-007, 015–018, 021, 023 | A.91 + every mapping row and relation interpretation |
| [85 · `/req/geojson/procedure-schema`](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson_procedure-schema) | `geojson` / `geojson` | Single Procedures and collections validate against `procedure.json` and `procedureCollection.json`. | `D+X`; Procedure representation; explicit class condition absent | Normative named schemas; IDR-SRV-008, 021, 023 | A.92 + local pinned schema, no-geometry, invalid/golden corpus |
| [86 · `/req/geojson/procedure-mappings`](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson_procedure-mappings) | `geojson` / `geojson` | Implement Tables 44–45 for Procedure type, valid time, and implementing-System links. | `D+X`; Procedure representation/relationships | Tables 44–45; IDR-SRV-015–018, 021, 023 | A.93 + every mapping row and relation interpretation |
| [87 · `/req/geojson/sf-schema`](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson_sf-schema) | `geojson` / `geojson` | Single Sampling Features and collections validate against `samplingFeature.json` and `samplingFeatureCollection.json`. | `D+X`; Sampling Feature representation; explicit class condition absent | Normative named schemas; IDR-SRV-008, 021, 023 | A.94 + local pinned schema, invalid/golden corpus |
| [88 · `/req/geojson/sf-mappings`](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson_sf-mappings) | `geojson` / `geojson` | Implement Tables 46–47 for Sampling Feature type/time and sampled-feature, parent, sample-chain, stream, and control associations; geometry is supplied through the GeoJSON feature/schema baseline rather than these tables. | `D+X`; Sampling Feature representation/relationships | Tables 46–47; IDR-SRV-007, 015–018, 021, 023 | A.95 + every mapping row and relation interpretation |

### 6.7 SensorML JSON Encoding

Schema base: [`https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/openapi/schemas/sensorml/`](https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/openapi/schemas/sensorml/). Part 1 provides SensorML schemas for System, Deployment, Procedure, and Property Definitions—not Sampling Features.

| No. / Exact Requirement ID | Class / Conf | Normalized Obligation and Condition | Glaux Classification; Affected Area | Artifact / Downstream Handoff | ATS and Supplemental Test |
|---|---|---|---|---|---|
| [89 · `/req/sensorml/mediatype-read`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_mediatype-read) | `sensorml` / `sensorml` | Support `application/sml+json` for reading applicable resources as SensorML JSON. | `D+I+X`; negotiation/serialization; nearby draft-era vendor-media note is stale | SensorML 3; IDR-SRV-012, 014, 021 | A.96 + Accept q-values, unsupported media, vendor-alias policy |
| [90 · `/req/sensorml/mediatype-write`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_mediatype-write) | `sensorml` / `sensorml` | If Create/Replace/Delete is implemented, accept and parse `application/sml+json` request bodies. | `D+I+C`; request negotiation/validation | SensorML schemas; IDR-SRV-012, 013, 021, 023, 031 | A.97 + malformed/schema/415 and media parameters |
| [91 · `/req/sensorml/relation-types`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_relation-types) | `sensorml` / `sensorml` | Associations represented in `links` use the association name as `rel`. | `D+X`; link serialization; conflicts with mapping-table `ogc-rel:` footnotes | Tables 49–54; IDR-SRV-008, 010, 017, 021 | A.98 + interpretation/compatibility variants |
| [92 · `/req/sensorml/resource-id`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_resource-id) | `sensorml` / `sensorml` | Local resource ID is the SensorML JSON `id` and equals the `{id}` portion in the resource URL. | `D`; identifier serialization | SensorML common schema; IDR-SRV-016, 021, 023 | A.99 + mismatch, encoding, immutable-ID cases |
| [93 · `/req/sensorml/feature-attribute-mapping`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_feature-attribute-mapping) | `sensorml` / `sensorml` | For feature resources, apply Table 48: UID → `uniqueId`, name → `label`, description → `description`. | `D`; common SensorML feature mapping | Table 48; IDR-SRV-015, 016, 021, 023 | A.100 + spelling normalization and round trip |
| [94 · `/req/sensorml/system-schema`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_system-schema) | `sensorml` / `sensorml` | When the intended System class applies, single/collection bodies validate against `system.json` and `systemCollection.json`. | `D+C+X`; published condition uses nonexistent `/req/system-features` | Normative named schemas; IDR-SRV-008, 021, 023 | A.101 + condition interpretation and offline dependency resolution |
| [95 · `/req/sensorml/system-sml-class`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_system-sml-class) | `sensorml` / `sensorml` | Hardware/human observers use Physical Component/System; simulations/processes use Simple/Aggregate Process. | `D`; polymorphic System representation choice | SensorML schemas; IDR-SRV-015, 021, 023 | A.102 + each asset kind and wrong-class negatives |
| [96 · `/req/sensorml/system-mappings`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_system-mappings) | `sensorml` / `sensorml` | Implement Tables 49–50 for System definition/classifiers/time/position, native relationships, and endpoint links. | `D+X`; System serialization/relationships; relation-prefix conflict | Tables 49–50; IDR-SRV-007, 015–018, 021, 023 | A.103 + every mapping row and relation interpretation |
| [97 · `/req/sensorml/deployment-schema`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_deployment-schema) | `sensorml` / `sensorml` | When the intended Deployment class applies, bodies validate against `deployment.json` and `deploymentCollection.json`. | `D+C+X`; condition uses nonexistent `/req/deployment-features` | Normative named schemas; IDR-SRV-008, 021, 023 | A.104 + condition interpretation/offline schema validation |
| [98 · `/req/sensorml/deployment-mappings`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_deployment-mappings) | `sensorml` / `sensorml` | Implement Tables 51–52 for Deployment definition/time/location, native platform/System fields, and remaining links. | `D+X`; Deployment serialization/relationships | Tables 51–52; IDR-SRV-007, 015–018, 021, 023 | A.105 + every mapping row and relation interpretation |
| [99 · `/req/sensorml/procedure-schema`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_procedure-schema) | `sensorml` / `sensorml` | When the intended Procedure class applies, bodies validate against `procedure.json` and `procedureCollection.json`. | `D+C+X`; condition uses nonexistent `/req/procedure-features` | Normative named schemas; IDR-SRV-008, 021, 023 | A.106 + condition interpretation/offline schema validation |
| [100 · `/req/sensorml/procedure-sml-class`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_procedure-sml-class) | `sensorml` / `sensorml` | Equipment specifications use Physical Component/System; human procedures use Simple/Aggregate Process; Procedures have no position. | `D`; polymorphic Procedure/no-location invariant | SensorML schemas; IDR-SRV-015, 021, 023 | A.107 + each kind, wrong-class, position negative |
| [101 · `/req/sensorml/procedure-mappings`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_procedure-mappings) | `sensorml` / `sensorml` | Implement Tables 53–54 for Procedure definition, valid time, and implementing-System links. | `D+X`; Procedure serialization/relationships | Tables 53–54; IDR-SRV-015–018, 021, 023 | A.108 + every mapping row and relation interpretation |
| [102 · `/req/sensorml/property-schema`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_property-schema) | `sensorml` / `sensorml` | When the intended Property class applies, bodies validate against `property.json` and `propertyCollection.json`. | `D+C+X`; condition uses nonexistent `/req/property-definitions` | Normative named schemas; IDR-SRV-008, 021, 023 | A.109 + condition interpretation/offline schema validation |
| [103 · `/req/sensorml/property-mappings`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml_property-mappings) | `sensorml` / `sensorml` | Implement Table 55: `baseProperty`, `objectType`, and `statistic` map to same-named URI-valued SensorML members. | `D`; Property serialization/semantic references | Table 55; IDR-SRV-015, 017, 021, 024 | A.110 + URI validation, optional fields, round trip |

### 6.8 Formal Recommendations

These are formal `SHOULD` statements, not part of the 103 `SHALL`/formal requirement count. Glaux should normally implement them for a high-quality reference server, but its conformance ledger must keep their advisory status visible.

| Recommendation | Class | Source-Backed Recommendation | Glaux Recommendation | Annex Test / Note |
|---|---|---|---|---|
| [`/rec/api-common/resource-uids-types`](https://docs.ogc.org/is/23-001/23-001.html#_rec_api-common_resource-uids-types) | API Common | Prefer globally unique UIDs, especially UUID URNs or IANA-registered URN namespaces. | Adopt a documented globally unique UID policy in IDR-SRV-016. | A.5; Glaux policy: warning, not failure |
| [`/rec/system/location`](https://docs.ogc.org/is/23-001/23-001.html#_rec_system_location) | System | A System should include location. | Support location where meaningful and authorized; do not fabricate it. | A.7; Glaux policy: warning; omitted from the published class statement list |
| [`/rec/advanced-filtering/resource-by-property`](https://docs.ogc.org/is/23-001/23-001.html#_rec_advanced-filtering_resource-by-property) | Advanced Filtering | Support simple property filtering compatible with the Features core recommendation. | Research a bounded, explicitly documented property-filter subset in IDR-SRV-011. | A.45; Glaux policy: warning, not failure |
| [`/rec/advanced-filtering/indirect-prop`](https://docs.ogc.org/is/23-001/23-001.html#_rec_advanced-filtering_indirect-prop) | Advanced Filtering | Observed/controlled-property filters should follow `baseProperty` derivation transitively. | Implement with cycle-safe indexed traversal if class/profile selects it. | A.65; Glaux policy: warning, not failure |
| [`/rec/advanced-filtering/indirect-foi`](https://docs.ogc.org/is/23-001/23-001.html#_rec_advanced-filtering_indirect-foi) | Advanced Filtering | `foi` filters should follow Sampling Features related through `sampledFeature`. | Implement with explicit depth/cycle semantics researched later. | A.66; Glaux policy: warning, not failure |

---

## 7. Server Applicability and Boundary Classification

### 7.1 Class-Level Applicability Matrix

| Class | Direct Glaux Server Obligation | Trigger / Modularity | Inherited or Integration Consequence | Required Later Decision |
|---|---|---|---|---|
| API Common | Yes, for every selected Part 1 class | Transitive prerequisite | Common/Features HTTP, discovery, parameter, JSON, collection, and feature behavior | Resolve published prerequisite discrepancy in IDR-SRV-008 |
| System | Yes if selected | Resource-class choice; provisional Glaux target | Canonical/typed collection routes and current location | Final class profile and canonical model |
| Subsystem | Yes if selected | Requires System | Recursive traversal, filter propagation, aggregate Part 2 associations | Hierarchy/cycle/materialization policies |
| Deployment | Yes if selected | Resource-class choice; provisional target | Canonical/typed routes and optional System association | Final class profile and relationship model |
| Subdeployment | Yes if selected | Requires Deployment | Recursive traversal and association aggregation | Hierarchy/cycle/materialization policies |
| Procedure | Yes if selected | Resource-class choice; provisional target | Feature endpoint with no geometry | Canonical/encoding model |
| Sampling Feature | Yes if selected | Requires System | Mandatory per-System nested endpoint; GeoJSON supplied | Relationship and Part 2 semantics |
| Property | Yes if selected | Resource-class choice; provisional target | Non-feature endpoint; SensorML supplied | Semantic/property model |
| Advanced Filtering | Yes if claimed | Optional class; provisional target | Broad query/index burden across every applicable endpoint | Query semantics, index strategy, recommendation profile |
| Create/Replace/Delete | Yes if claimed | Optional; rows depend on resource classes | Inherits draft Features 4; adds cascade/membership contracts | Pinned draft, HTTP/status, consistency, lifecycle policies |
| Update | Yes if claimed | Requires CRD; rows depend on resources | Inherits draft Features 4 PATCH | Patch format, concurrency, validation policies |
| GeoJSON | Yes if claimed | Encoding class; write conditional on CRD | Inherits Features GeoJSON; only four resource families supplied | Exact resource-class/encoding profile and relation interpretation |
| SensorML | Yes if claimed | Encoding class; write conditional on CRD | Inherits SensorML 3; only four resource families supplied | Schema resolver, model mapping, relation interpretation |

### 7.2 Cross-Cutting Server Responsibilities

| Area | Part 1 Baseline for Glaux Server | Boundary / Deferral |
|---|---|---|
| Identity | Enforce per-type local-ID uniqueness, global URI UID uniqueness, canonical URL/ID consistency | UID generation, external identifiers, lifecycle, and aliases: IDR-SRV-016 |
| Discovery | Publish inherited landing/API/conformance and collection discovery plus CS canonical endpoints | Exact links, API path, conformance document: IDR-SRV-009/010 |
| Resource graph | Persist and return required resource associations, hierarchy, canonical/nested navigation, and recursive aggregates | Canonical data model and graph policies: IDR-SRV-015–020 |
| Query | Apply inherited `limit`, `bbox`, `datetime` plus selected hierarchy and advanced filters consistently | Sorting, field selection, pagination algorithm, limits, and index design: IDR-SRV-011/025/026 |
| Representation | Negotiate and validate selected GeoJSON/SensorML representations and mapping tables | Detailed content negotiation/schema pipeline: IDR-SRV-012/021/023 |
| Write lifecycle | If claimed, implement CS transaction routes, cascade, membership consistency, and updates | Exact draft Part 4 contract, concurrency/idempotency/storage: IDR-SRV-013/029/031 |
| Conformance | Declare only implemented classes and meet all prerequisites/conditions | Final profile, harness, and claims: IDR-SRV-008/050/051 |
| Security | Part 1 recommends secure communications and strong machine identity but mandates no auth scheme | Threat model, authorization, policy, audit: IDR-SRV-039–041 |

### 7.3 Component and Integration Boundary

**Glaux Server owns:** HTTP route behavior; query parsing; canonical discovery; resource identity and relationship integrity; response generation; request and schema validation; selected write transactions; persistent visibility and collection coherence; conformance declarations; and evidence needed to test its claims.

**Publishers, Simulators, and other writers:** act as clients of the selected write contract. They must supply acceptable media types, identifiers, references, and representations, but Part 1 does not assign their transformation, scheduling, native-protocol, or source-simulation logic to the server. Their exact contract is a later IDR-SRV-031–033 decision.

**Read clients:** consume advertised links, media types, filters, and representations. The server cannot require a client to infer undocumented paths or repair invalid representations. Interoperability behavior is tested later in IDR-SRV-056.

**External systems and Part 2:** Part 1 may link toward datastream, control-stream, observation, and command resources, but their behavior is not a Part 1 obligation. IDR-SRV-007 must extract those requirements independently. Native NATO or vendor interfaces remain outside the CSAPI core unless a later approved profile assigns them.

### 7.4 Part 1 Omissions That Must Not Be Invented Here

- No Part 1 sorting or field-selection contract.
- No required offset, page-number, or cursor algorithm; paging links should be treated as opaque.
- No complete CS-specific error representation or exhaustive status-code table.
- No mandatory authentication, authorization, or security scheme.
- No production OpenAPI document structure, code-generation approach, or Rust framework.
- No persistence/index/database technology.
- No Part 2 observation, stream, control, command, WebSocket, or event behavior.

---

## 8. Resource and API Behavior Mapping

### 8.1 Inherited API Foundation

The Part 1 API Common class inherits these behaviors before any CS resource route is added:

| API Area | Inherited Baseline | Part 1 Effect |
|---|---|---|
| Landing page | GET root and link to API definition, conformance, and other resources | CS endpoints/collections must be discoverable through the coherent API link graph researched in IDR-SRV-009 |
| API definition | Machine- and/or human-readable service description linked with standard relations | Part 1 does not mandate the final Glaux OpenAPI file or a fixed `/api` implementation path |
| Conformance declaration | GET conformance resource returning claimed class URIs | Must list only selected classes and their coherent prerequisites |
| Collections | GET all collection metadata and individual collection metadata | CS typed collections add `itemType`/`featureType` values |
| Collection items | GET items with inherited `limit`, `bbox`, `datetime`, response/link/count behavior | CS resource endpoints reuse this model; Properties adapt feature terminology |
| Single feature | GET by collection and feature ID | CS resources also require their canonical type-specific item URL |
| HTTP/JSON | HTTP 1.1, negotiation, parameter parsing, `400` handling, JSON support | GeoJSON/SensorML add their own selected media types and schemas |

### 8.2 Resource and Endpoint Inventory

`{api_root}` prefixes every path below.

| Resource Kind | Nature | Canonical List / Item | Additional Required or Conditional Endpoints | Typed Collection Metadata | Part 1 Encodings Supplied |
|---|---|---|---|---|---|
| System | Feature | `/systems`; `/systems/{id}` | `/systems/{parentId}/subsystems` if Subsystems; nested `/deployments` if condition met; mandatory nested `/samplingFeatures` when SF selected | `itemType=feature`, `featureType=sosa:System` | GeoJSON and SensorML |
| Subsystem | System Feature | Canonical item remains `/systems/{id}` | Parent nested collection; root/nested recursive query | Same as System | GeoJSON and SensorML |
| Deployment | Feature | `/deployments`; `/deployments/{id}` | `/deployments/{parentId}/subdeployments`; `/systems/{sysId}/deployments` if Req 17 condition met | `itemType=feature`, `featureType=sosa:Deployment` | GeoJSON and SensorML |
| Subdeployment | Deployment Feature | Canonical item remains `/deployments/{id}` | Parent nested collection; root/nested recursive query | Same as Deployment | GeoJSON and SensorML |
| Procedure | Feature without geometry | `/procedures`; `/procedures/{id}` | Custom/typed collection item endpoints | `itemType=feature`, `featureType=sosa:Procedure` | GeoJSON and SensorML |
| Sampling Feature | Feature | `/samplingFeatures`; `/samplingFeatures/{id}` | `/systems/{sysId}/samplingFeatures` for every System | `itemType=feature`, `featureType=sosa:Sample` | GeoJSON only |
| Property Definition | Non-feature resource | `/properties`; `/properties/{id}` | Custom/typed resource collection items | `itemType=sosa:Property` | SensorML only |

The encoding asymmetry is intentional in the published standard. Glaux must not invent a GeoJSON Property schema or SensorML Sampling Feature schema and call it Part 1 conformance. A future extension may add encodings with distinct media/profile declarations and tests.

### 8.3 Canonical URLs, Links, and Hierarchies

- A resource discovered through a nested or custom collection retains its type-specific canonical item URL.
- The representation obtained through a noncanonical URL carries a canonical link.
- Nested endpoint membership is a relationship constraint, not a second resource identity.
- Subsystem and Subdeployment recursion changes default root-list visibility when those classes are selected. The more specific hierarchy requirements are treated as conditional refinements of the earlier “all resources” wording.
- Association links and mapping tables form part of the public resource graph. The published relation-name contradiction must be resolved centrally; serializers must not make independent guesses.
- Recursive association aggregation can reach Part 2 resources. Part 1 establishes the link/target behavior, while Part 2 defines those resource types and operations.

### 8.4 Method and Lifecycle Map

| Operation | Part 1 Route Pattern | Trigger / Important Rule |
|---|---|---|
| Read list | Canonical, nested, or `/collections/{collectionId}/items` endpoint | GET; inherited query/response behavior; resource type purity |
| Read item | Type-specific canonical item; collection item may also retrieve | GET; canonical link when representation came from elsewhere |
| Create root | POST to resource-type root | CRD plus resource class; new resource appears in root collection |
| Create nested | POST to subsystem, subdeployment, or System Sampling Feature endpoint | CRD plus relevant class; create and establish required parent relation atomically |
| Replace | PUT canonical item URL | CRD; replacement reflected in all containing collections |
| Delete resource | DELETE canonical/root-member URL | CRD; root deletion removes all memberships; System dependency/cascade rules |
| Delete membership | DELETE through non-root collection membership | CRD; remove only that collection membership |
| Add membership | POST `text/uri-list` to a collection | CRD; same-API canonical URL or UID, one per line |
| Update | PATCH canonical item URL | Update plus resource class; exact patch semantics inherited from pinned Features 4 draft |

### 8.5 Query and Retrieval Map

| Parameter / Behavior | Source and Scope | Required Semantics / Boundary |
|---|---|---|
| `limit` | Inherited Features; resource list endpoints | Mandatory inherited page-size ceiling; exact service maximum and paging mechanism later |
| `bbox` | Inherited Features on spatial feature collections | Spatial intersection in default CRS84/CRS84h unless optional CRS support selected |
| `datetime` | Inherited then refined by Req 3 | Test `validTime` intersection and include absent/null `validTime` |
| `recursive` | Subsystem/Subdeployment only | Boolean; false/omitted direct/top-level, true all descendants; propagate other filters |
| `id` | Advanced Filtering | Local IDs or URI UIDs in homogeneous lists; UID terminal `*` prefix match |
| `q` | Advanced Filtering | Keyword OR over terms; always search name/description; other fields and linguistic normalization are server choices |
| `geom` | Advanced Filtering on geometry-bearing endpoints | WKT spatial intersection; exclude resources without geometry |
| Relationship filters | Advanced Filtering by resource type | `parent`, `procedure`, `foi`, `observedProperty`, `controlledProperty`, `system`, `baseProperty`, `objectType` as applicable |
| Multiple filters | Advanced Filtering | Logical AND across different filters |
| `next` link | Features recommendation plus Part 1 paging prose and supporting ATS | Expected interoperability behavior; follow advertised opaque link; do not invent client-side offset rules |
| Sorting / selection | Not defined by approved Part 1 | Any future support is a documented extension or later-standard behavior, not a Part 1 claim |

### 8.6 Representations and Media Types

| Media Type | Normative Part 1 Role | Resource Coverage | Downstream Detail |
|---|---|---|---|
| `application/geo+json` | Read when GeoJSON class claimed; write when GeoJSON + CRD apply | System, Deployment, Procedure, Sampling Feature | IDR-SRV-012, 021, 023 |
| `application/sml+json` | Read when SensorML class claimed; write when SensorML + CRD apply | System, Deployment, Procedure, Property | IDR-SRV-012, 021, 023 |
| `text/uri-list` | Add existing resource references to a collection | Collection membership operation | IDR-SRV-012, 013, 031 |
| `application/json` | Inherited Common JSON for landing/conformance and other specified common documents | Common API documents, not an automatic CS-resource encoding | IDR-SRV-009, 012 |

Query shortcuts such as `f=json` or `f=sml` appear in examples but are not Part 1 requirements. If Glaux provides them for compatibility, IDR-SRV-012/014 must document them as extensions.

### 8.7 Validation and Failure Baseline

**Standards obligations:** inherited Common/Features behavior provides HTTP processing, successful `200` reads, and `400` responses for specified unknown or invalid query names/values. Selected transaction behavior inherits additional method, request-media, precondition, and status semantics from the pinned Features Part 4 draft. Named GeoJSON/SensorML schemas and incorporated mapping tables validate selected representations.

**Source-backed finding:** Part 1 does not mandate a CS-specific error schema, an exhaustive status table, or `application/problem+json`. Problem Details appears only as a Common recommendation. The example OpenAPI's `StandardResponses.yaml` is informative and incomplete as a source of normative behavior.

**Project recommendation:** IDR-SRV-013 should define one consistent error model that includes every inherited mandatory status and representation constraint, then clearly identify useful project extensions. Negative tests must derive first from the exact requirement and inherited source, not from the example OpenAPI.

---

## 9. Schema, OpenAPI, and Representation Artifact Inventory

### 9.1 Published Package Inventory

The official Part 1 OpenAPI package contains 153 files:

| Artifact Group | Count | Role |
|---|---:|---|
| Root OpenAPI document | 1 | Informative example/template, OpenAPI 3.1.0, `info.version: 0.0.1` |
| README | 1 | Informative usage context |
| JSON examples | 36 | Informative fixtures/illustrations |
| Parameter YAML files | 28 | Informative API component definitions |
| Path YAML files | 19 | Informative route templates |
| Request YAML files | 11 | Informative request-body templates |
| Response YAML files | 20 | Informative response templates |
| JSON Schemas | 37 | Five common, sixteen GeoJSON, sixteen SensorML; normative only where incorporated by a requirement |

The official ZIP [`ogcapi-connected-systems-p1-1_0_0.zip`](https://schemas.opengis.net/ogcapi/ogcapi-connected-systems-p1-1_0_0.zip), retrieved July 31, 2026, had SHA-256 `43FCAB8FB079B153E1DC01559C9395A51E8FAB0E7C87C709FDAE6A28B1983F12`. Its internal version directory is `1.0.0`, while the live schema URL uses `1.0`; a local resolver must preserve or deliberately alias those bases.

### 9.2 Normatively Named Representation Schemas

| Resource | GeoJSON Single / Collection | SensorML Single / Collection | Mapping Tables |
|---|---|---|---|
| System | `system.json` / `systemCollection.json` | `system.json` / `systemCollection.json` | GeoJSON 39–41; SensorML 48–50 |
| Deployment | `deployment.json` / `deploymentCollection.json` | `deployment.json` / `deploymentCollection.json` | GeoJSON 39, 42–43; SensorML 48, 51–52 |
| Procedure | `procedure.json` / `procedureCollection.json` | `procedure.json` / `procedureCollection.json` | GeoJSON 39, 44–45; SensorML 48, 53–54 |
| Sampling Feature | `samplingFeature.json` / `samplingFeatureCollection.json` | Not supplied | GeoJSON 39, 46–47 |
| Property Definition | Not supplied | `property.json` / `propertyCollection.json` | SensorML Table 55; identity/name/description are schema/model-backed, with no explicit Property incorporation of feature-only Table 48 |

All 73 JSON schema/example files parsed as JSON in the audit. The 37 schemas declare JSON Schema 2020-12 but have no `$id`, so base URI selection is operationally significant.

### 9.3 Confirmed Artifact Defects and Limitations

1. The root document calls itself an **Example OpenAPI specification**, identifies version `0.0.1`, and includes demo/local server URLs.
2. `paths/systemDeployments.yaml` contains invalid YAML indentation that prevents clean root bundling.
3. The POST operation in `paths/collectionItems.yaml` omits the `{collectionId}` path-parameter declaration; only the GET operation declares it.
4. The package includes invalid component placement and combines OpenAPI 3.1 with OAS 3.0-style `nullable` usage.
5. Local references include confirmed missing targets such as `parameters/subsystemId.yaml` and the exception/example targets referenced by `responses/ServerError.yaml`; one affected unused subsystem path is commented out of the root but remains defective.
6. Live official SensorML/common dependencies referenced by the Part 1 schemas return `404`, including common `link.json`/`timePeriod.json` and several SensorML 3 process, system, deployment, and property schemas. A full repository checkout resolves most through companion trees, but direct remote validation does not.
7. A reproducible Redocly 2.43.2 lint of the published root reported 63 errors and 100 warnings. Many style/security warnings are not normative, but invalid YAML and unresolved references are deterministic blockers.
8. The published UAV System example uses unprefixed association relations. The pinned development branch changes three to `ogc-rel:` values, corroborating the known relation conflict without amending the approved text.

### 9.4 Artifact-Use Policy Recommended for Glaux

| Use | Approved HTML | Named Schemas/Tables | Example OpenAPI / Examples | Mutable Repository |
|---|---|---|---|---|
| Determine obligation | Controlling | Controlling when incorporated | No | No |
| Generate implementation scaffold | Requirements source | Validation/model input | Yes, after review/repair | Yes, pinned |
| Generate tests | Requirement/ATS anchors | Schema and golden tests | Candidate fixtures only | Regression/errata evidence |
| Claim conformance | Exact selected IDs and prerequisites | Required validation evidence | Never by itself | Never by itself |
| Change control | Version/hash | Vendor with provenance/hash and resolver aliases | Track local repairs explicitly | Pin commit and compare before updates |

---

## 10. Downstream Topic Handoffs

| Downstream Topic(s) | Binding Input from This Report | Work Deliberately Deferred |
|---|---|---|
| IDR-SRV-007 | Part 1 resource identities, hierarchy, association endpoints, and every explicit Part 2 handoff | Extract dynamic-data, observation, stream, control, and command requirements independently |
| IDR-SRV-008 | Thirteen-class dependency graph; 103/5/110 counts; prerequisite contradictions; draft Part 4 risk; ATS gaps | Select the exact Glaux conformance profile and claim policy; resolve class-level interpretations |
| IDR-SRV-009 | Common/Features landing, API-definition, conformance, JSON, and discovery prerequisites | Choose exact link graph, documents, paths, and conformance declaration behavior |
| IDR-SRV-010 | Canonical, nested, typed-collection, association, hierarchy, and link-relation requirements | Finalize link relations, navigation consistency, canonical URL, and endpoint rules |
| IDR-SRV-010A | Published standard, schema bundle, example artifact, and draft dependency pins | Define API/schema/profile versioning, deprecation, and compatibility policy |
| IDR-SRV-011 | Inherited `limit`/`bbox`/`datetime`; `recursive`; all advanced filters; AND composition; no sorting/selection baseline | Decide pagination mechanism/limits, query extensions, filter normalization, and index-aware contract |
| IDR-SRV-012 | `application/geo+json`, `application/sml+json`, `text/uri-list`, inherited negotiation; example `f=` is nonnormative | Define negotiation precedence, aliases, error behavior, charset/parameters, and representation availability |
| IDR-SRV-013 | Inherited `400` floor; schema failures; cascade rejection; draft transaction statuses; no Part 1 error schema | Define one consistent error object and complete method/status/failure semantics |
| IDR-SRV-014 | OpenAPI is informative `0.0.1` and structurally defective; exact route/schema evidence is cataloged | Design authoritative Glaux OpenAPI generation, validation, publication, and drift prevention |
| IDR-SRV-014A–014G | Requirement/defect ledger for comparing existing servers, clients, and interoperability findings | Assess precedents as evidence; never let an implementation silently override the standard |
| IDR-SRV-015 | Seven conceptual resource roles, feature/non-feature distinction, model-table and encoding inputs | Build the canonical Rust-independent domain/resource model |
| IDR-SRV-016 | Local-ID/UID uniqueness, URI validity, canonical URL and SensorML ID equality | Define generation, aliases, immutability, external IDs, and lifecycle |
| IDR-SRV-017 | All association endpoints/mappings, hierarchy parentage, relation-name conflict, recursive targets | Define relationship cardinality, storage, link generation, integrity, and compatibility aliases |
| IDR-SRV-018 | `validTime` filtering, absent/null inclusion, latest System location | Define temporal precision, intervals, current projection, history, and clock semantics |
| IDR-SRV-019–020 | Existing descriptive fields/links and Part 2/status handoff boundary | Add provenance/trust/status only from their controlling evidence; do not call extensions Part 1 requirements |
| IDR-SRV-021 | SensorML media type, four resource-family schemas, class selection, Tables 48–55 | Complete SensorML 3 extraction and representation architecture |
| IDR-SRV-023 | Named schemas, JSON Schema 2020-12, missing `$id`, broken remote dependencies, mapping-table obligations | Define vendoring/resolution, validation layers, schema updates, repair provenance, and failure reporting |
| IDR-SRV-024 | Property model, base/object/statistic mappings, transitive/indirect filtering | Define vocabularies, semantic binding, units, derivation, and normalization |
| IDR-SRV-025–026 | Uniqueness, hierarchy, graph joins, full-text, geometry, temporal, transitive filters | Select persistence/index/geospatial strategies that can meet observable contracts |
| IDR-SRV-029–031 | Create/replace/delete/update, collection coherence, cascade, nested creation, `text/uri-list` | Define transactions, concurrency, idempotency, ingestion, validation, and rollback |
| IDR-SRV-039–041 | Nonbinding Part 1 security guidance plus resource/link/write surfaces | Develop authentication, authorization, SSRF/link safety, policy, and audit requirements independently |
| IDR-SRV-050 | Thirteen conformance classes, 110 ATS tests, warning tests, test defects, inherited-class dependencies | Build a harness that preserves conditions and supplements rather than blindly copies Annex A |
| IDR-SRV-051 | Exact 103-row inventory and trace-record field proposal | Define durable standard-to-code-to-test evidence and coverage rules |
| IDR-SRV-052–053 | Candidate test layers and required graph/schema/error fixtures | Design Rust test architecture, fixtures, golden files, mutation/property tests, and scenarios |
| IDR-SRV-056 | Canonical/collection navigation, media/filter/error expectations, compatibility questions | Build external-client interoperability matrix and behavior probes |
| IDR-SRV-057 | Accepted conclusions, decisions, risks, and unresolved interpretation records | Synthesize only after all intervening reports are complete and accepted |

---

## 11. Traceability and Test-Strategy Implications

### 11.1 Required Trace Record

Later implementation work should maintain a machine-readable ledger with one record per numbered requirement and separate records for recommendations and project policies. At minimum, each record should contain:

1. stable Glaux trace key, such as `CS1-REQ-005`;
2. approved source version, URL, retrieval date, and content hash;
3. full requirement URI and exact HTML anchor;
4. normative level (`requirement`, `recommendation`, inherited requirement, or project policy);
5. requirement class, conformance class, and prerequisite classes;
6. complete lettered obligation references and incorporated table/schema/clause references;
7. activation condition and Glaux applicability/profile status;
8. affected resource, endpoint, operation, parameter, relationship, representation, and persistence concern;
9. pinned artifact paths/hashes and any local resolver/repair record;
10. interpretation issue/decision ID, rationale, authority, and effective version;
11. Annex A test ID and inherited conformance-test IDs;
12. positive, negative, schema, lifecycle, interoperability, and regression test IDs;
13. implementation module/handler/model/schema links; and
14. status and evidence for implemented, tested, declared, waived, superseded, or blocked.

Example:

| Trace Key | Requirement | Condition | Behavior | Interpretation | Tests |
|---|---|---|---|---|---|
| `CS1-REQ-005` | `/req/system/canonical-url`, OGC 23-001 §9.3 | System class selected | GET `/systems/{id}`; canonical link from every other retrieval URL | Relation encoding follows later `INT-CS1-LINK-REL` decision | A.9; root/nested/custom discovery; missing ID; link equality; both encodings |

### 11.2 Required Test Families

| Test Family | Part 1 Evidence | Representative Checks |
|---|---|---|
| Positive HTTP contract | Canonical/resources/collection requirements and inherited Features | Every route/method, correct type membership, canonical links, paging, counts, selected media |
| Negative parameter/error | Common/Features parameter rules and advanced filters | Unknown/invalid names/values, mixed IDs/UIDs, invalid URI/WKT/time/bbox/Boolean, bad combinations |
| Schema and mapping | Requirements 80–88 and 92–103 | JSON Schema validation, every table mapping, golden round trips, wrong class, absent/extra/invalid fields |
| Identity/property-based | Requirements 1–2, 38–59 | Generate collisions, arbitrary valid URIs/lists, filter composition, Unicode text, graph depth/cycles |
| Hierarchy/graph | Requirements 9–13, 17, 19–23, 32, 42–58 | Direct/transitive membership, filter propagation, aggregate associations, no duplicates, cycle defense |
| Transaction/lifecycle | Requirements 60–76 | Create visibility, replace propagation, root/non-root deletion, cascade atomicity, nested parent integrity, patch concurrency |
| Negotiation | Requirements 77–78, 89–90, 71 | Accept/Content-Type combinations, unsupported media, malformed bodies, representation availability |
| Conformance | Thirteen classes and Annex A | Conditions/prerequisites, all A.1–A.110, recommendation warnings, exact declared URIs |
| Interoperability | Canonical links, mappings, media, queries | Independent clients follow links, filters, schemas, errors; compatibility tests for recorded interpretations |
| Artifact regression | Published schema/OAS defects and pins | Hash/change detection, local resolver completeness, schema bundle, repaired OAS drift, known upstream defect tests |

### 11.3 Annex A Use and Known Coverage Gaps

Glaux should implement or adapt the normative ATS, but not assume it is complete:

- A.1 and A.2 are reusable supporting tests; A.1 says `Accepted` where `Accept` is evidently intended.
- Under the recommended Glaux harness policy, Recommendation tests A.5, A.7, A.45, A.65, and A.66 report advisory/warning outcomes unless Glaux separately makes them profile requirements; the Annex itself is not uniform about that outcome.
- A.6 iterates every advertised collection even though Requirement 3 is scoped to collections of feature types defined by Part 1.
- A.14 and A.24 only inspect whether an unspecified “request” contains Boolean `recursive`; they do not establish that the server advertises or actually supports the parameter.
- A.13 and A.23 expect `{id}` in target URLs even though their fixture variables are `sysId` and `depId`.
- Hierarchy tests A.15/A.16/A.25/A.26 do not prove that **every other filter** is applied at every depth.
- Canonical-resource tests can discover fixtures only through typed collections and therefore miss resources available only through canonical/nested routes.
- Collection tests can pass vacuously without proving at least one correctly tagged collection exists.
- A.22 assumes Systems despite Req 17's condition and directly contains unresolved `{sensorml-mediatype}`.
- A.71 refers to the wrong resource type while testing a Subdeployment; the Property update test uses a Sampling Feature fixture variable name.
- Schema validation alone cannot prove every mapping-table semantic, correct relationship target, or canonical link.
- Location recency, cascade atomicity, collection-wide replacement propagation, negative query grammar, and several relationship cardinality cases need stronger project tests.

The harness should preserve the original ATS identifier, record any executable correction as an adapter with rationale, and retain a regression that exposes the upstream defect.

### 11.4 Minimum Scenario and Fixture Corpus

The later fixture strategy should include:

- two resource types with deliberate local-ID reuse and deliberate cross-type UID collisions;
- a three-or-more-level System hierarchy with direct and deployed subsystems;
- a three-or-more-level Deployment hierarchy with multiple Systems and overlapping associations;
- Systems with present, absent, null, historical, and moving locations/valid times;
- direct and transitive Property derivation with a cycle candidate;
- domain Features and Sampling Features connected through direct and indirect sample chains;
- every resource represented in every Part 1-supplied encoding, plus invalid and boundary variants;
- root and at least two custom/typed collections with overlapping membership;
- a System cascade graph containing Part 1 and Part 2 child links and multiple Deployment associations;
- query cases for every parameter alone and meaningful AND combinations; and
- upstream-artifact regression fixtures for link relations, resolver aliases, invalid examples, and ATS placeholder errors.

### 11.5 Completion Evidence for a Requirement

A later implementation should not mark a requirement complete merely because a handler exists or Annex A passes. Completion evidence should show: the class/condition is resolved; all lettered obligations and incorporated artifacts are implemented; positive and relevant negative tests pass; inherited requirements pass; ambiguity decisions are recorded; public conformance/API documents agree with runtime behavior; and code/test references are attached to the trace record.

---

## 12. Recommendations and Decision Analysis

### 12.1 Recommended Baseline Decision

The project should accept this report as the traceable Part 1 baseline and carry forward these rules:

1. The approved OGC 23-001 Version 1.0 HTML controls all Part 1 obligation claims.
2. All 103 numbered requirements remain in the Glaux trace inventory; no item disappears because a class decision is deferred.
3. The five recommendations remain visibly advisory unless a later Glaux profile deliberately elevates one.
4. The exact conformance profile is selected in IDR-SRV-008, with every prerequisite and condition recorded.
5. Part 2 starts separately in IDR-SRV-007; this report supplies only its Part 1 handoffs.

### 12.2 Scope Options

| Option | Benefit | Cost / Risk | Assessment |
|---|---|---|---|
| Minimum modular server | Fastest route to a narrow OGC claim | Conflicts with the project's best-of-breed reference goal; provides poor coverage for real CSAPI clients | Not recommended for Glaux's target |
| Broad read-only server | Avoids the draft Features 4 dependency initially | Omits normative Part 1 transaction classes and weakens publisher/reference-server value | Useful as an implementation milestone, not the final target |
| All 13 classes with pinned draft dependencies and qualified claims | Best functional and interoperability coverage; preserves one coherent trace baseline | More implementation/testing work; requires change control and explicit interpretations | **Recommended provisional end-state**, finalized by IDR-SRV-008 |

**Analysis:** A staged delivery can implement read classes before write classes without redefining the intended end state. Runtime conformance declarations must always describe what the released build actually passes, not the roadmap.

### 12.3 Implementation-Planning Guardrails

1. **Create the machine-readable ledger before coding against memory.** IDR-SRV-051 should convert §6 into durable data with exact lettered rows, inherited IDs, conditions, decisions, code, and tests.
2. **Vendor normative schemas with provenance.** Preserve upstream files and hashes, use explicit resolver aliases, isolate any repair as a documented overlay, and test remote/local bases in IDR-SRV-023.
3. **Generate a Glaux OpenAPI from the decided contract.** Reuse reviewed components where helpful, but do not fork the broken example wholesale or let code generation make standards decisions.
4. **Pin Features Part 4.** Put inherited transactions behind a change-controlled compatibility boundary, monitor approval/deltas, and qualify claims until an approved baseline exists.
5. **Centralize identifiers and link relations.** One model/serializer policy must serve GeoJSON, SensorML, OpenAPI, storage, and tests; contradictory relation rules require one recorded decision plus compatibility tests.
6. **Apply filters everywhere the class requires.** Canonical, nested, and typed/custom endpoints must share query semantics; persistence and indexes must not create endpoint-specific drift.
7. **Keep opaque links opaque.** Clients and tests follow advertised `next`/canonical/association URLs; they should not synthesize undocumented paging or routing rules.
8. **Separate OGC obligations from Glaux quality choices.** A consistent Problem Details error model, security controls, extra formats, sorting, selection, or aliases can be valuable, but must be labeled project extensions/profile rules.
9. **Supplement Annex A.** Treat its IDs as trace anchors and retain its tests, while adding the negative, graph, schema, lifecycle, concurrency, and regression coverage in §11.
10. **Use implementation studies as checks, not authorities.** IDR-SRV-014A–014G should identify compatibility realities and good design ideas without changing the normative column of the ledger.
11. **Do not let Part 2 leak into this baseline.** Part 1 association targets may point to dynamic resources; their operations and representations wait for the next report.
12. **Delay Rust-specific architecture decisions.** This report states observable server contracts. Rust framework, module, type, and test architecture decisions remain evidence-driven topics later in the plan.

### 12.4 What Acceptance Does and Does Not Decide

| Acceptance Establishes | Acceptance Does Not Yet Establish |
|---|---|
| Approved Part 1 source and pins | Final Glaux class/conformance declaration |
| 13/103/5/110 baseline and full inventory | Final link-relation interpretation |
| Direct/inherited/conditional boundary | Final HTTP error/status and negotiation policy |
| Official schema versus informative OpenAPI authority | Final domain/persistence/Rust architecture |
| Issue register and downstream owners | A claim that the report repaired the OGC publication |
| Provisional all-class reference target | A claim that all 13 classes are the OGC minimum |

---

## 13. Risks, Constraints, Ambiguities, and Open Questions

### 13.1 Principal Risks

| Risk | Likelihood / Impact | Control |
|---|---|---|
| Draft Features Part 4 changes after Glaux implements transactions | High / High | Pin exact draft; isolate behavior; regression/delta review; qualify claim |
| Published prerequisite and condition defects produce false conformance claims | High / High | Requirement-first dependency graph; interpretation records; independent conformance review |
| Example OpenAPI drives implementation despite being incomplete | High / High | Treat as informative; generate/lint/bundle Glaux contract from trace baseline |
| Remote schemas fail or change | High / High | Vendor/hashes/resolver aliases; offline deterministic validation; update governance |
| Relation-name contradiction fragments clients/encodings | High / High | Central decision plus advertised/profiled compatibility tests |
| Hierarchy/filter/cascade behavior is under-tested | Medium / High | Multi-depth graph fixtures, property-based tests, atomic transaction tests |
| Modular OGC minimum is confused with Glaux's intended scope | Medium / High | Separate normative applicability from project profile and release status |
| Extra “helpful” behaviors are mislabeled as Part 1 | Medium / Medium | Trace every public rule to standard, profile, or extension authority |
| Part 2 behavior is prematurely inferred | Medium / High | Preserve handoffs; execute IDR-SRV-007 separately |

### 13.2 Standards and Artifact Interpretation Register

These are evidence-backed publication issues, not findings that Glaux may ignore the surrounding obligations.

| ID | Published Issue | Consequence / Interim Treatment | Owner | Blocks IDR-SRV-007? |
|---|---|---|---|---|
| INT-CS1-001 | `/req/api-common` names Features 1 Core and Common 1 Core/Landing/JSON; `/conf/api-common` instead uses a broken `ogcapi-1` URI, adds draft Common 2 Collections/Simple Query, and omits JSON. | Implement the coherent requirement-table superset; decide claim wording and upstream clarification. | IDR-SRV-008/050 | No |
| INT-CS1-002 | CRD/Update requirement tables use `ogcapi-features-4`; Annex uses broken `ogcapi-4`. The Update requirement class also requires Part 1 CRD, while `/conf/update` omits that prerequisite and substitutes API Common. GeoJSON uses `ogcapi-features-1` in its requirement table but `ogcapi-1` in Annex. | Preserve both exact citations; use identifiers defined by the referenced standards for implementation; document the full dependency interpretation. | IDR-SRV-008/050 | No |
| INT-CS1-003 | SensorML requirement class names four SensorML 3 classes; Annex names Common plus indirect SensorML 2.1 JSON. | Treat SensorML 3 classes as the encoding obligation baseline; resolve claim/test bridge explicitly. | IDR-SRV-008/021/050 | No |
| INT-CS1-004 | Requirements 79/91 say association name; mapping footnotes incorporated by mapping requirements say `ogc-rel:` plus name; approved examples are unprefixed, while current repository repairs three examples to prefixed values. | No silent choice. Define canonical wire rule and compatibility policy centrally, with both forms tested as decided. | IDR-SRV-008/010/017 | No |
| INT-CS1-005 | A draft-era note suggests `application/vnd.ogc.sml+json`; Requirements 89/90 mandate `application/sml+json`. | Formal requirement controls; any vendor alias is a documented extension. | IDR-SRV-012/021 | No |
| INT-CS1-006 | Requirements 67/76 publish unresolved `[clause-derived-properties]` conditions. | Context strongly indicates the Property class, but record that as interpretation. | IDR-SRV-008/013 | No |
| INT-CS1-007 | Requirements 94/97/99/102 condition on nonexistent `*-features`/`property-definitions` IDs. | Map provisionally to actual `/req/system`, `/req/deployment`, `/req/procedure`, `/req/property`; preserve defect record. | IDR-SRV-008/021/023 | No |
| INT-CS1-008 | System Recommendation 2 has A.7 but is omitted from the class statement list, and its recommendation table lacks an `Included in` row. | Keep it as a formal recommendation/advisory test, not a requirement. | IDR-SRV-008/050 | No |
| INT-CS1-009 | Requirements 7/16 say canonical endpoints expose all resources; hierarchy Requirements 11/21 default to top-level only. | Treat hierarchy statements as conditional refinements; test false/true behavior explicitly. | IDR-SRV-010/011 | No |
| INT-CS1-010 | Requirement 17 is conditional; A.22 assumes System support and directly contains unresolved `{sensorml-mediatype}`. | Gate the test by the requirement condition and repair only in a documented harness adapter. | IDR-SRV-008/050 | No |
| INT-CS1-011 | Sampling Feature filter prose's illustrative endpoint list omits the nested endpoint required by Req 32. | Apply “all resources endpoints” formal scope, including nested endpoint, pending review. | IDR-SRV-011 | No |
| INT-CS1-012 | Property filter prose names `/samplingFeatures`; object-type examples use `object` instead of normative `objectType`. | Follow formal Property route and parameter; decide whether compatibility alias is worthwhile and advertise it as an extension. | IDR-SRV-011/014 | No |
| INT-CS1-013 | `cascade` behavior is triggered by presence but value schema, false/multiple values, and several graph cases are unspecified locally. | IDR-SRV-013/029 must define a safe deterministic profile without weakening required rejection/deletion behavior. | IDR-SRV-013/029/031 | No |
| INT-CS1-014 | `ID_List` gives array shape but not complete query `style`/`explode` rules; examples imply comma separation. | Reconcile inherited Common list serialization and publish one OpenAPI contract. | IDR-SRV-011/014 | No |
| INT-CS1-015 | Resource-specific GeoJSON schema/mapping requirements lack explicit resource-class conditions despite the modular-resource premise. | Provisional full target implements all supplied families; IDR-SRV-008 must state claim interpretation for narrower profiles. | IDR-SRV-008/021 | No |
| INT-CS1-016 | ATS includes typos/placeholders, recommendation tests, vacuous collection cases, discovery gaps, and incomplete hierarchy/filter coverage. | Preserve original tests and IDs; add conditioned adapters and supplemental tests with rationale. | IDR-SRV-050/051/053 | No |
| INT-CS1-017 | Official example OpenAPI has invalid YAML, missing path parameters/references, incompatible constructs, fixed demo metadata, and large lint failure counts. | Never use it as normative contract; extract reviewed components only and regression-test known defects. | IDR-SRV-014 | No |
| INT-CS1-018 | Normatively referenced schema graphs have no `$id` and live remote dependencies return 404. | Vendor the full dependency graph with base-preserving resolver aliases and hashes. | IDR-SRV-021/023 | No |
| INT-CS1-019 | Requirement 61 does not fully define all multi-Deployment or Part 2 descendant cascade cases. | Treat minimum named deletion/link repair as binding; design conservative atomic semantics after Part 2 extraction. | IDR-SRV-007/013/029 | No |
| INT-CS1-020 | Requirements 79/91 use “must” rather than the drafting convention's “shall” inside formal requirement tables. | They remain numbered normative requirements; wording anomaly does not demote them. | IDR-SRV-008/050 | No |
| INT-CS1-021 | Requirement 9C uses uppercase `CAN`, which is not the Standard's defined permission keyword `MAY`. | Preserve the wording; decide and test whether currently deployed subsystems are a supported Glaux option. | IDR-SRV-008/010/015 | No |
| INT-CS1-022 | Several System/Deployment model associations are marked “Required” while their descriptions say “if any.” | Reconcile conceptual presence and empty/omitted serialization against each selected encoding schema; do not silently choose cardinality. | IDR-SRV-015/017/021/023 | No |
| INT-CS1-023 | SensorML Table 48 is explicitly a Feature attribute mapping, while Property is a non-feature and its formal mapping Requirement 103 incorporates only Table 55. | Derive Property identity/name/description from its model/schema and record the missing explicit mapping trace; do not extend Table 48 by assertion. | IDR-SRV-008/015/021/023 | No |

### 13.3 Open Project Decisions and Owners

| Decision | Why It Remains Open | Resolution Topic |
|---|---|---|
| Exact Glaux class/conformance profile and release milestones | Part 1 is modular; project aims broader than minimum | IDR-SRV-008 |
| Canonical link-relation wire form and compatibility alias | Published obligations conflict | IDR-SRV-008, 010, 017 |
| Exact Features Part 4 draft pin and upgrade policy | Normative dependency is not approved/stable | IDR-SRV-008, 010A, 013, 031 |
| Paging algorithm and advertised `next` contract | Standard requires/encourages behavior but not algorithm | IDR-SRV-011 |
| Error object and complete status/precondition profile | Part 1 supplies only a partial inherited floor | IDR-SRV-013 |
| Local schema bundle/resolver/repair design | Official remote graph is not reliably resolvable | IDR-SRV-021, 023 |
| OpenAPI source-of-truth and generation pipeline | Official template is not production-ready | IDR-SRV-014 |
| Extension policy for sorting, selection, aliases, and extra encodings | Useful interoperability features are not Part 1 obligations | IDR-SRV-010A–014 and implementation studies |

No open item in this section prevents a defensible, separate extraction of Part 2 in IDR-SRV-007.

---

## 14. Validation Against the Research Plan

### 14.1 Methodology-Phase Validation

| Plan Phase | Status | Evidence Produced |
|---|---|---|
| 1. Source Collection and Framework | Met | §3 source/authority register; §4 classification and inventory model |
| 2. Normative Requirement Extraction | Met | §§5–6: 13 classes, 103 requirements, 5 recommendations, exact anchors and ATS mappings |
| 3. Applicability and Boundary | Met | §7 class and cross-cutting matrices; component/out-of-scope boundary |
| 4. Resource, Schema, and API Mapping | Met | §§8–9 endpoint, resource, query, media, schema, and OpenAPI inventories |
| 5. Traceability and Test Analysis | Met | §11 trace record, test families, ATS gaps, fixture corpus, completion evidence |
| 6. Synthesis | Met | §§1, 10, 12–13 conclusions, handoffs, recommendations, and issue register |

### 14.2 Success-Criterion Validation

| Topic Plan Success Criterion | Status | Evidence |
|---|---|---|
| Official OGC Part 1 reviewed directly | Met | Approved source pin/hash in §3.1; full inventory §6 |
| Requirement and conformance classes identified | Met | §5.1 and §5.2 |
| Server-relevant requirements extracted with anchors | Met | Requirements 1–103 in §6 |
| Normative text, examples, recommendations distinguished | Met | §§3.2, 4.2, 6.8, 9.3 |
| Glaux applicability/boundary classified | Met | Row-level §6 plus §7 |
| Inherited Common/Features behavior identified | Met | §§5.3, 8.1, Appendix B |
| Schema, OpenAPI, representation artifacts identified | Met | §9 |
| Downstream API/model/validation/conformance/test handoffs documented | Met | §10 |
| Requirement-to-test implications captured | Met | §6 ATS column and §11 |
| Unresolved interpretations explicitly listed | Met | §13.2–13.3 |
| References explicit and reproducible | Met | §§3.1, 15, Appendix C |

### 14.3 Deliverable-Content Validation

| Required Content | Status | Report Location |
|---:|---|---|
| 1. Executive summary | Met | §1 |
| 2. Scope and plan alignment | Met | §2 |
| 3. Evidence and authority classification | Met | §3 |
| 4. Requirement extraction methodology | Met | §4 |
| 5. Requirement/conformance-class inventory | Met | §5 |
| 6. Normative requirement inventory | Met | §6 |
| 7. Server applicability/boundary matrix | Met | §7 |
| 8. Resource/API behavior mapping | Met | §8 |
| 9. Schema/OpenAPI/representation inventory | Met | §9 |
| 10. Downstream handoff matrix | Met | §10 |
| 11. Traceability/test implications | Met | §11 |
| 12. Recommendations | Met | §12 |
| 13. Risks/constraints/open questions | Met | §13 |
| 14. Plan success-criteria validation | Met | §14.2 |
| 15. References | Met | §15 |

The inventory's eleven required fields are represented by the exact ID/source link, class/conf column, normalized summary, classification code, Glaux applicability/affected-area column, artifact/handoff column, ATS/supplemental-test column, and explicit issue notes. Section 4.3 explains the mapping.

### 14.4 Independent Review

Three independent read-only research tracks reviewed: Clauses 8–12 and inherited Features behavior; Clauses 13–19 and every encoding/write/filter requirement; and the official schema/OpenAPI/conformance artifact set. Their independently reconciled totals matched: 13 classes, 103 requirements, 5 recommendations, and 110 tests. The artifact review added deterministic schema-resolution and OpenAPI defects; the clause reviews added lettered obligations, conditions, test gaps, and publication contradictions. No reviewer edited the repository.

After drafting, the report was checked against all 36 questions, six phases, eleven success criteria, fifteen required sections, all numbered source IDs, and the one-topic boundary. Plan-owner acceptance remains deliberately unchecked.

---

## 15. References

### 15.1 Controlling and Inherited Standards

- [OGC 23-001, OGC API - Connected Systems - Part 1: Feature Resources, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html).
- [OGC API - Connected Systems standards landing page](https://ogcapi.ogc.org/connectedsystems/).
- [OGC 17-069r4, OGC API - Features - Part 1: Core corrigendum, Version 1.0.1](https://docs.ogc.org/is/17-069r4/17-069r4.html).
- [OGC 19-072, OGC API - Common - Part 1: Core, Version 1.0.0](https://docs.ogc.org/is/19-072/19-072.html).
- [OGC API - Features Part 4 draft, OGC 20-002r1](https://docs.ogc.org/DRAFTS/20-002r1.html).
- [OGC 06-103r4, OpenGIS Implementation Specification for Geographic Information - Simple Feature Access](https://portal.ogc.org/files/?artifact_id=25355).
- [OGC 23-000, SensorML Encoding Standard 3.0](https://docs.ogc.org/is/23-000/23-000.html), dependency awareness only.
- [OGC 23-002, OGC API - Connected Systems - Part 2](https://docs.ogc.org/is/23-002/23-002.html), dependency awareness only.

### 15.2 Official Artifacts and Mutable Evidence

- [Published Connected Systems Part 1 schema/OpenAPI directory](https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/openapi/).
- [Official Part 1 schema ZIP](https://schemas.opengis.net/ogcapi/ogcapi-connected-systems-p1-1_0_0.zip).
- [`opengeospatial/ogcapi-connected-systems` tag `v1.0.0`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0), commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`.
- [`opengeospatial/ogcapi-connected-systems` pinned development commit](https://github.com/opengeospatial/ogcapi-connected-systems/tree/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f).
- [`opengeospatial/ogcapi-features` pinned draft-development commit](https://github.com/opengeospatial/ogcapi-features/tree/9ca25f56a58ed822ea8a685a7a41afa7181aaa8b).

### 15.3 Project and Governance Sources

- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md).
- [IDR-SRV-006 Research Plan](../IDR%20Plans/idr-srv-006-csapi-part-1-requirement-baseline.md).
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md).
- [Research Report Template](../../../../../Governance/research-report-template.md).
- [Research Planning Approach](../../../../../Governance/research-planning-approach.md).
- [Accepted IDR-SRV-001 Report](idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md).
- [Accepted IDR-SRV-002 Report](idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md).
- [Accepted IDR-SRV-003 Report](idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md).
- [Accepted IDR-SRV-004 Report](idr-srv-004-terminology-and-concept-crosswalk-report.md).
- [Accepted IDR-SRV-005 Report](idr-srv-005-related-nato-standards-boundary-review-report.md).

---

## 16. Next Steps and Handoff

### 16.1 Current Status

Research, extraction, synthesis, validation, and repository review for `IDR-SRV-006` are complete. The Glaux Project Lead accepted this report on July 31, 2026, so it is **Final** and available as the controlling Part 1 baseline for downstream research.

The same combined plan-owner instruction authorized execution of exactly one next topic, `IDR-SRV-007`. No later topic was authorized by that transition.

### 16.2 Completed Two-Action Transition

The required workflow transition had exactly two actions:

1. The Glaux Project Lead accepts `IDR-SRV-006`.
2. The Glaux Project Lead authorizes execution of exactly one next eligible topic, `IDR-SRV-007`.

Both actions were given in the plan owner's combined instruction that authorized IDR-SRV-007. The historical combined instruction was:

> Approve IDR-SRV-006 and execute exactly one Glaux Server research plan: IDR-SRV-007, using the standing single-topic execution instructions.

---

## 17. Appendices

### 17.1 Appendix A - Requirement Count Reconciliation

| Source Structure | Count | Reconciliation |
|---|---:|---|
| Requirements classes | 13 | One corresponding conformance class each |
| Numbered requirements | 103 | Req 1–103; one Annex test each |
| Numbered recommendations | 5 | Rec 1–5; one advisory/warning Annex test each |
| Supporting abstract tests | 2 | A.1 canonical resources; A.2 collection items |
| Total abstract tests | 110 | 103 + 5 + 2 |

Direct requirement distribution: API Common 3; System 5; Subsystem 5; Deployment 5; Subdeployment 5; Procedure 5; Sampling Feature 5; Property 4; Advanced Filtering 22; Create/Replace/Delete 12; Update 5; GeoJSON 12; SensorML 15. Total: 103.

### 17.2 Appendix B - Inherited Approved Requirement Identifiers

These identifiers are prerequisites or explicitly incorporated behavior, not additional Part 1-numbered requirements.

**OGC API - Common Part 1 (19 requirements from selected classes):**

- Core: `/req/core/http`, `/req/core/query-param-name-unknown`, `/req/core/query-param-value-invalid`, `/req/core/query-param-capitalization`, `/req/core/query-param-list-delimiter`, `/req/core/query-param-list-escape`, `/req/core/query-param-list-empty`, `/req/core/query-param-value-boolean`, `/req/core/query-param-value-integer`, `/req/core/query-param-value-decimal`, `/req/core/query-param-value-double`.
- Landing Page: `/req/landing-page/root-op`, `/req/landing-page/root-success`, `/req/landing-page/api-definition-op`, `/req/landing-page/api-definition-success`, `/req/landing-page/conformance-op`, `/req/landing-page/conformance-success`.
- JSON: `/req/json/definition`, `/req/json/content`.

**OGC API - Features Part 1 Core (35 requirements):**

- API foundation: `/req/core/root-op`, `/req/core/root-success`, `/req/core/api-definition-op`, `/req/core/api-definition-success`, `/req/core/conformance-op`, `/req/core/conformance-success`, `/req/core/http`, `/req/core/query-param-unknown`, `/req/core/query-param-invalid`, `/req/core/crs84`.
- Collection metadata: `/req/core/fc-md-op`, `/req/core/fc-md-success`, `/req/core/fc-md-links`, `/req/core/fc-md-items`, `/req/core/fc-md-items-links`, `/req/core/fc-md-extent`, `/req/core/fc-md-extent-multi`, `/req/core/sfc-md-op`, `/req/core/sfc-md-success`.
- Feature collections: `/req/core/fc-op`, `/req/core/fc-limit-definition`, `/req/core/fc-limit-response-1`, `/req/core/fc-bbox-definition`, `/req/core/fc-bbox-response`, `/req/core/fc-time-definition`, `/req/core/fc-time-response`, `/req/core/fc-response`, `/req/core/fc-links`, `/req/core/fc-rel-type`, `/req/core/fc-timeStamp`, `/req/core/fc-numberMatched`, `/req/core/fc-numberReturned`.
- Single feature: `/req/core/f-op`, `/req/core/f-success`, `/req/core/f-links`.

**Features Part 1 GeoJSON (2 additional encoding prerequisites):** `/req/geojson/definition`, `/req/geojson/content`.

Part 1 Requirements 6, 15, 26, and 30 explicitly incorporate Features §§7.15.2–7.15.8; Requirement 35 adapts them from feature to resource terminology. Those cross-references can also carry permissions and recommendations that must retain their original normative level.

### 17.3 Appendix C - Reproducibility Record

| Evidence | Pin / Verification |
|---|---|
| Approved Part 1 HTML | SHA-256 `555C4B3BEA06AB91B980BDAA3C99D265E6718DBAD943CA1CBEC39FBBF283C92A` |
| Official schema ZIP | SHA-256 `43FCAB8FB079B153E1DC01559C9395A51E8FAB0E7C87C709FDAE6A28B1983F12` |
| Part 1 source/artifact release | `v1.0.0` and `v1.0-schemabundle` both point to `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2` |
| Version 1 OpenAPI tree | Git tree `5331a9fbb58f3bc64d9f9e5acf1f70d0d310b45e` |
| Connected Systems comparison | `v1.0.0` versus `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`; only three relation values in one Part 1 example changed |
| Features Part 4 draft comparison | Commit `9ca25f56a58ed822ea8a685a7a41afa7181aaa8b`, retrieved 2026-07-31 |
| Artifact count | 153 files: 1 root + 1 README + 36 examples + 28 parameters + 19 paths + 11 requests + 20 responses + 37 schemas |
| Requirement parse | 13 requirement classes; 103 unique numbered requirement IDs; 5 recommendation IDs; 13 conformance classes; A.1–A.110 |

### 17.4 Report Completion Checklist

- [x] Topic ID matches the overall research-plan index
- [x] Topic plan and controlling overall plan are linked
- [x] Exactly one topic, IDR-SRV-006, was executed
- [x] IDR-SRV-005 acceptance was recorded from the plan owner's combined instruction
- [x] All 5 core and 31 detailed questions are answered
- [x] All six methodology phases and outputs are complete
- [x] All eleven success criteria are met
- [x] All fifteen required report contents are present
- [x] All eleven minimum inventory fields are represented
- [x] All 13 classes, 103 requirements, 5 recommendations, and 110 tests reconcile
- [x] Exact requirement anchors and reproducible source pins are included
- [x] Normative obligations, recommendations, informative artifacts, analysis, and project recommendations are distinguished
- [x] Direct, inherited, conditional, integration, handoff, and out-of-scope boundaries are explicit
- [x] Schema/OpenAPI artifacts and limitations were directly audited
- [x] Every required downstream handoff is recorded
- [x] Test/traceability implications and ATS gaps are documented
- [x] Standards ambiguities and evidence limitations are recorded honestly
- [x] Independent requirement, artifact, technical, and completeness reviews were performed
- [x] IDR-SRV-007 was not begun before the plan-owner transition authorized it
- [x] Plan-owner acceptance and acceptance date recorded
---

# Section 008: Conformance Class and Requirement Mapping - Research Report

**Topic ID:** IDR-SRV-008<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-008 Conformance Class and Requirement Mapping](../IDR%20Plans/idr-srv-008-conformance-class-and-requirement-mapping.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 34 detailed questions; all six methodology phases, eleven success criteria, seventeen deliverable requirements, and sixteen enumerated minimum mapping fields are validated<br>
**Methodology Used:** Reconciliation of the accepted Part 1 and Part 2 requirement inventories with the approved standards, normative conformance annexes, inherited requirement classes, tagged OpenAPI artifacts, live OGC identifier mappings, bounded official issue and pull-request history, and available executable-test resources<br>
**Research Time:** Approximately 4 hours of AI-assisted elapsed execution time, including three parallel independent read-only standards and upstream audits, on August 1, 2026<br>
**Primary Sources:**

- OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0
- OGC 23-002, *OGC API - Connected Systems - Part 2: Dynamic Data*, Version 1.0
- OGC 17-069r4, *OGC API - Features - Part 1: Core*, Version 1.0 with Corrigendum
- OGC 23-000, *SensorML Encoding Standard*, Version 3.0, and OGC 24-014, *SWE Common Data Model Encoding Standard*, Version 3.0

**Supporting Resources:** Accepted IDR-SRV-001 through IDR-SRV-007 reports; the tagged `opengeospatial/ogcapi-connected-systems` Version 1.0.0 source and API artifacts; the official OGC NamingAuthority mappings; the bounded upstream-history register; and the pinned OGC API - Features Part 4 draft<br>
**Document Purpose:** Establish the complete, decision-usable conformance profile and requirement-to-class traceability baseline for the full-scope Rust Glaux Server implementation<br>
**Author:** OpenAI Codex, with independent read-only Part 1, Part 2, and upstream/test-resource audits<br>
**Accepted By:** TBD - Glaux Project Lead review pending<br>
**Acceptance Date:** TBD until accepted<br>
**Date:** August 1, 2026<br>
**Last Updated:** August 1, 2026

---

## How to Read This Report

For the project decision, read §§1, 9, 11, 14, 15.2, and 18. They answer the practical questions: what Glaux should ultimately support, what may be claimed during staged development, what evidence a claim needs, and what remains unresolved. Sections 5–8 and Appendices A–C are the audit trail. They are detailed so later design, Rust implementation, and testing work can cite exact class and requirement identifiers without reconstructing this analysis.

Four labels prevent unlike statements from being confused:

- A **standards obligation** comes from an approved normative source.
- A **source-backed finding** reports what an authoritative or informative source contains.
- **Analysis** explains the consequence of that evidence.
- A **project recommendation** states what Glaux should do.

Plain-language acronyms used below: **ATS** means abstract test suite; **ETS** means executable test suite; **CRD** means create, replace, and delete; **OAS** means OpenAPI Specification; and **PURL** means the persistent identifier URL operated by OGC. Compact handoffs such as `IDR-009/050` mean `IDR-SRV-009` and `IDR-SRV-050`.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Conformance Mapping Methodology
5. CSAPI Part 1 Conformance Class Inventory
6. CSAPI Part 2 Conformance Class Inventory
7. Inherited OGC Conformance and Dependency Inventory
8. Requirement-to-Conformance Mapping Matrix
9. Glaux Server Conformance Posture
10. Upstream Standards-Maintenance Context and Disposition
11. Conformance Declaration Implications
12. Verification and Test-Strategy Implications
13. Downstream Topic Handoff Matrix
14. Recommendations and Decision Analysis
15. Risks, Constraints, and Open Questions
16. Validation Against the Research Plan
17. References
18. Next Steps and Handoff
19. Appendices

---

## 1. Executive Summary

### 1.1 Plain-English Decision Brief

The recommended Glaux end state is straightforward: **implement all 25 conformance classes defined by OGC API - Connected Systems Parts 1 and 2**. Those classes organize **233 numbered requirements**—103 in Part 1 and 130 in Part 2—plus five Part 1 recommendations. No direct CSAPI class is outside the intended best-of-breed server, and none should be quietly removed from the final goal merely because OGC permits smaller implementations.

That decision does **not** mean OGC requires every conforming server to implement all 25 classes. Both standards are modular and define no universal CSAPI Core class. OGC expects at least one connected-system resource type and one encoding, with other classes selected according to the implementation. Supporting all 25 is the **Glaux product profile**, chosen because the project intends a complete Rust reference implementation. Requirements and prerequisites within a class become binding when that class is claimed.

The target profile and a released build's conformance declaration must remain separate. Glaux may implement the profile in stages, but `GET /conformance` must list only the classes that the particular release has implemented and proved. Planned, partial, experimental, or merely coded behavior is not a conformance claim. A class is claimable only when its prerequisites, all unconditional requirements, every condition triggered by the build, the official abstract tests, and necessary supplemental tests pass for that released build.

Two qualifications require continued control. First, both CSAPI parts inherit CRD and update behavior from OGC API - Features Part 4, which is still a draft. Glaux should retain those four CSAPI transaction classes in the end-state target, pin the exact Part 4 draft used during development, qualify resulting claims, and perform a formal delta review when Part 4 is published. Second, open upstream issue #23 confirms that the broad encoding classes do not cleanly describe every read/write/media-type combination. Until the standard changes, Glaux's safest full-profile implementation is to satisfy every triggered encoding requirement and use its implementation-specific OpenAPI document to describe the exact operation-by-resource-by-media-type matrix.

The standards contain normative abstract tests but no public official executable Connected Systems ETS was found. The official OGC API - Features ETS can provide evidence only for inherited Features behavior. Glaux will therefore need its own traceable executable adaptation of the 240 published CSAPI abstract tests, plus stronger negative, lifecycle, schema, interoperability, and defect-regression tests. That harness belongs to later research; this report defines what it must trace.

### 1.2 Reconciled Baseline

| Source | Requirement Classes | Conformance Classes | Numbered Requirements | Numbered Recommendations | Abstract Tests |
|---|---:|---:|---:|---:|---:|
| CSAPI Part 1 | 13 | 13 | 103 | 5 | 110: 103 requirement, 5 recommendation, 2 supporting |
| CSAPI Part 2 | 12 | 12 | 130 | 0 | 130 requirement tests |
| **Direct CSAPI total** | **25** | **25** | **233** | **5** | **240** |

These counts exclude inherited Common, Features, SensorML, and SWE Common requirements; those remain prerequisite evidence rather than direct CSAPI requirements.

### 1.3 Principal Conclusions

1. **All 25 direct CSAPI classes belong in the Glaux end-state profile.** None is permanently deferred, excluded, or out of scope.
2. **The OGC minimum is modular, not all-inclusive.** The all-class target is a Glaux decision that serves the full-scope reference-server goal.
3. **Every direct numbered requirement maps cleanly to one direct class.** Part 1 contributes 103 and Part 2 contributes 130; Appendix A and Appendix B preserve every exact identifier.
4. **Class declarations are evidence gates, not roadmap statements.** A staged release lists only the classes and complete prerequisite chains it actually passes.
5. **The normative requirement-class dependency graph controls implementation.** Where Annex A omits or misnames a prerequisite, Glaux must preserve the approved obligation and document a test adapter instead of weakening it.
6. **The transaction target remains qualified.** Part 1 CRD/Update and Part 2 CRD/Update rely on the unpublished Features Part 4 draft pinned in this report.
7. **Encoding claims require an additional capability matrix.** A class-level claim does not replace per-operation OpenAPI advertisement, and issue #23 remains unresolved.
8. **Part 2 class PURLs currently return 404.** Their exact canonical `http://www.opengis.net/spec/.../conf/...` strings remain the declaration identifiers; human-readable standard anchors and PURL health are tracked separately.
9. **The tagged OAS conformance example is not a profile oracle.** It contains stale Common identifiers and nonexistent CSAPI classes, including Part 1 `/conf/sampling` and Part 2 `/conf/geojson`.
10. **No public official CSAPI ETS was found.** Passing the official Features ETS proves only inherited Features behavior, never a Connected Systems class.
11. **AEP-4789 Volume II does not select a smaller class subset.** It adopts the four-standard package but leaves exact optionality/profile choices to the implementation or later guidance.
12. **The traceability key is exact and implementable:** standard version → requirement class → numbered requirement → conformance class/test → prerequisite/condition → artifact → Rust implementation → released-build evidence.

### 1.4 Recommendation for Plan-Owner Acceptance

Accept this report as the conformance mapping baseline and approve the evidence-gated, all-25-class Glaux target. Acceptance does not assert that Glaux currently conforms, does not confer OGC certification, does not finalize the later implementation roadmap, and does not turn recommendations or unresolved upstream proposals into standards obligations.

---

## 2. Scope and Plan Alignment

### 2.1 In Scope and Completed

- All 13 Part 1 and 12 Part 2 requirement/conformance class pairs.
- Every direct numbered requirement and Part 1 recommendation, mapped by exact identifier to its class and abstract test family.
- Formal and inherited prerequisites, including Common, Features, SensorML, SWE Common, WKT, and the draft Features Part 4 dependency.
- Class applicability, dependency, declaration evidence, test implications, and downstream handoffs for Glaux Server.
- Bounded official history for issues #23, #28, #82, #141, #150, and directly cross-cutting #152, including linked PRs and publication relationship.
- Tagged OAS conformance artifacts, live PURL behavior, official NamingAuthority mappings, and public executable-test availability.

### 2.2 Explicitly Out of Scope

- Re-extracting the full prose of all 233 requirements; accepted IDR-SRV-006 and IDR-SRV-007 remain the detailed inventories.
- Designing the final `/conformance` representation and landing-page link graph; IDR-SRV-009 owns that work.
- Designing or implementing the executable conformance harness; IDR-SRV-050 owns it.
- Finalizing requirement-to-test tooling; IDR-SRV-051 owns it.
- Selecting Rust crates, database architecture, deployment topology, or a final implementation schedule.
- Claiming formal OGC certification or starting any other IDR topic.

### 2.3 Reconciliation with Accepted Prior Research

| Prior Finding | Reconciliation Here |
|---|---|
| IDR-SRV-003 found that AEP-4789 Volume II adopts the standards package but selects no exact optional class subset. | Confirmed. The all-25 profile is explicitly a Glaux project choice. |
| IDR-SRV-006 recommended carrying all 13 Part 1 classes. | Adopted as the Part 1 end-state target after class, prerequisite, ATS, OAS, PURL, and issue reconciliation. |
| IDR-SRV-007 recommended carrying all 12 Part 2 classes. | Adopted as the Part 2 end-state target after independent one-to-one mapping and condition review. |
| Both baselines warned that Features Part 4 remains draft. | Preserved as a revision-pinned qualification and later publication delta gate. |
| Both baselines said runtime declarations must describe demonstrated behavior. | Converted into the claim-gate policy in §11. |

The project lead approved IDR-SRV-007 in the instruction that authorized this report. Its report and plan are therefore marked accepted/final as part of this single-topic transition; IDR-SRV-008 acceptance remains pending.

### 2.4 Research-Question Coverage Summary

| Plan Group | Questions | Status | Main Evidence |
|---|---:|---|---|
| Core questions | 5 | Complete | §§5–14 |
| Conformance class inventory | 6 | Complete | §§5–7, 11 |
| Requirement mapping | 5 | Complete | §8, Appendices A–B |
| Upstream maintenance context | 3 | Complete | §10 |
| Glaux applicability | 5 | Complete | §9 |
| Declaration behavior | 5 | Complete | §11 |
| Validation and tests | 5 | Complete | §12 |
| Sequencing implications | 5 | Complete | §§9.3, 13 |

Appendix D maps every individual question to a report location.

---

## 3. Evidence Base and Authority Classification

### 3.1 Reproducible Source Register

| Source | Version / Snapshot | Authority and Use | Stable Anchor | Access Date | Limitation |
|---|---|---|---|---|---|
| [OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html) | Version 1.0; approved 2025-06-02; published 2025-07-16; HTML SHA-256 `555C4B3BEA06AB91B980BDAA3C99D265E6718DBAD943CA1CBEC39FBBF283C92A` | Controlling normative Part 1 text and ATS | Requirement/conformance anchors and Annex A | 2026-08-01 | Published prerequisite metadata contains contradictions documented here |
| [OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html) | Version 1.0; approved 2025-06-02; published 2025-07-16; HTML SHA-256 `E840613693C282A41B1DDA709EB266905683697FB430168FF348833E8F50DF5E` | Controlling normative Part 2 text and ATS | Requirement/conformance anchors and Annex A | 2026-08-01 | Part 2 detailed PURLs do not currently resolve |
| [Official CSAPI tagged source](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api) | Tag `v1.0.0`; commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2` | Reproducible publication source and informative API artifacts | `api/part1`, `api/part2` | 2026-08-01 | Repository release remains marked prerelease; approved documents still control |
| [OGC API - Common - Part 1](https://docs.ogc.org/is/19-072/19-072.html) | OGC 19-072, Version 1.0.0 | Normative inherited Common Core, Landing Page, and JSON behavior named by Part 1 | Requirement/conformance classes in §7.1 | 2026-08-01 | Only classes directly inherited by CSAPI are mapped here |
| [OGC API - Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) | OGC 17-069r4, Version 1.0 with Corrigendum | Normative inherited API and declaration behavior | Core, declaration of conformance classes, GeoJSON | 2026-08-01 | Full inherited extraction is outside this topic |
| [OGC 06-103r4 Simple Feature Access - Part 1](https://portal.ogc.org/files/?artifact_id=25355) | Version 1.2.1 | Normative WKT grammar incorporated by Part 1 advanced filtering | Well-known text representation | 2026-08-01 | Grammar dependency, not a CSAPI conformance class |
| [OGC SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) | OGC 23-000, Version 3.0 | Normative prerequisite for Part 1 SensorML class | Four JSON requirement classes | 2026-08-01 | Detailed SensorML profile research remains IDR-021 |
| [OGC SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html) | OGC 24-014, Version 3.0 | Normative prerequisite for Part 2 encoding classes | JSON record and JSON/Text/Binary encoding classes | 2026-08-01 | Detailed component/encoding research remains IDR-022 |
| [OGC API - Features Part 4 draft](https://docs.ogc.org/DRAFTS/20-002r1.html) | `20-002r1`, `1.0.0-draft.2`; repository `master` commit [`9ca25f56`](https://github.com/opengeospatial/ogcapi-features/tree/9ca25f56a58ed822ea8a685a7a41afa7181aaa8b), 2026-07-13 | Normative dependency as cited by CSAPI, but itself an unapproved draft | CRD and Update requirement classes | 2026-08-01 | Unstable; current commit is a research pin, not a published replacement baseline |
| [OGC NamingAuthority](https://github.com/opengeospatial/NamingAuthority/tree/63af09213dbce1e940333e82cd66bfe5859630ee/specification-elements/mappings) | Commit `63af09213dbce1e940333e82cd66bfe5859630ee`, 2026-08-01 | Authoritative operational identifier-mapping evidence | `23-001.csv`; no `23-002.csv` | 2026-08-01 | Resolver administration does not define normative meaning |
| [Official Features ETS](https://github.com/opengeospatial/ets-ogcapi-features10) | Public repository; no GitHub release; pushed 2025-12-09 | Executable inherited Features evidence only | Repository and hosted OGC Validator listings | 2026-08-01 | Does not test CSAPI classes; hosted revisions and source can differ |

### 3.2 Project and Supporting Evidence

| Source | Status / Pin | Use |
|---|---|---|
| [IDR-SRV-003 report](idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md) | Accepted | AEP package adoption, authority boundary, and lack of exact subset |
| [IDR-SRV-006 report](idr-srv-006-csapi-part-1-requirement-baseline-report.md) | Accepted | Exact Part 1 requirement inventory, artifacts, conditions, ambiguities, and ATS gaps |
| [IDR-SRV-007 report](idr-srv-007-csapi-part-2-requirement-baseline-report.md) | Accepted 2026-08-01 | Exact Part 2 requirement inventory, artifacts, conditions, ambiguities, and ATS gaps |
| [Upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md) | Version 1.1 after this topic's bounded refresh | Navigation, issue/PR disposition, release relationship, and topic routing |
| [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) | Current project baseline | Full-scope open-source Rust reference-server intent |
| Tagged Part 1 `confClasses.json` | `v1.0.0` commit | Informative artifact check; rejected as a profile source because it contains stale/invalid entries |

### 3.3 Authority Rules Applied

1. The approved OGC documents control obligations and exact identifiers.
2. A normative requirement-class table and an approved normative ATS are both preserved when they conflict; Glaux uses the stronger coherent dependency interpretation and records the test adapter rather than silently deleting either source.
3. Incorporated schemas, mapping tables, and external requirement classes carry the authority assigned by the invoking requirement.
4. The tagged source reproduces the publication baseline; OAS examples remain informative unless a normative requirement incorporates them.
5. Open issues and post-publication repository changes explain risks or direction but cannot amend Version 1.0.
6. The Glaux profile is a project decision layered over the standards; it cannot weaken a class while still claiming that class.

### 3.4 Evidence Limitations

- The public issue/PR review cannot authenticate private SWG minutes or unpublished decisions.
- No public official CSAPI ETS was found in the OGC GitHub organization or hosted Validator suite lists; this does not prove that no private or in-development suite exists.
- Live PURL results are point-in-time operational evidence. Identifier meaning remains fixed by the standards even if a resolver changes.
- SensorML, SWE Common, Common, and Features prerequisites were mapped at class level. Their complete internal requirement inventories remain with the assigned downstream topics.
- This report selects the end-state profile and evidence policy; it deliberately does not estimate or freeze a software roadmap.

---

## 4. Conformance Mapping Methodology

### 4.1 Six-Phase Execution

| Plan Phase | Work Performed | Result |
|---|---|---|
| 1. Source collection/framework | Pinned approved documents, tag, draft dependency, accepted reports, OAS, PURLs, NamingAuthority, register, and test resources; defined join keys and authority labels | Reproducible source and field model |
| 2. Class inventory | Independently enumerated requirement classes and conformance classes from Parts 1 and 2 | 13 + 12 exact class pairs |
| 3. Requirement mapping | Mechanically reconciled class metadata, every numbered requirement, exact ATS target, recommendation, condition, and inherited dependency | 233 requirements, 5 recommendations, 240 tests mapped |
| 4. Applicability/posture | Compared standards modularity with the accepted full-scope Glaux goal and AEP boundary | All 25 selected for end state; every release claim evidence-gated |
| 5. Verification implications | Classified ATS, schema, OAS, lifecycle, negative, interoperability, and official ETS evidence; catalogued gaps | Claim evidence and downstream harness inputs defined |
| 6. Synthesis | Reconciled contradictions, upstream dispositions, declaration behavior, sequencing dependencies, risks, and handoffs | Decision-usable conformance baseline |

### 4.2 Traceability Field Model

The later implementation and test system should use one versioned record per requirement and join those records to a class record. Required fields are:

| Field | Purpose |
|---|---|
| Standard, edition, source pin | Prevent accidental movement of the baseline |
| Requirement class and full `/req/...` URI | Identify the obligation group |
| Exact requirement/recommendation ID and source anchor | Identify the atomic rule |
| Conformance class and full `/conf/...` URI | Identify the claimable class |
| Exact ATS test URI and procedure | Preserve official verification trace |
| Prerequisites and activation conditions | Determine when a rule applies |
| Authority and interpretation record | Separate normative text, artifact finding, history, and project decision |
| Schema/OAS/encoding artifact and hash | Make validation reproducible |
| Glaux target and release disposition | Separate end-state intent from runtime claim |
| Rust implementation reference | Link rule to module/API behavior |
| Positive, negative, supplemental, and interoperability tests | Prove more than the literal ATS |
| Evidence build/version and result | Bind a claim to a released artifact |
| Upstream record and publication relationship | Preserve rationale without changing authority |
| Downstream owner | Prevent unresolved work from disappearing |

### 4.3 Identifier Model

- `CS1` means `http://www.opengis.net/spec/ogcapi-connectedsystems-1/1.0`.
- `CS2` means `http://www.opengis.net/spec/ogcapi-connectedsystems-2/1.0`.
- `CS1/req/system` is a requirement **class**; `CS1/req/system/canonical-url` is a requirement.
- `CS1/conf/system` is a conformance **class**; `CS1/conf/system/canonical-url` is an abstract test.
- `/rec/...` identifies a recommendation, not a requirement. A recommendation test is advisory unless a separate Glaux profile rule explicitly strengthens it.
- The `conformsTo` array contains class `/conf/...` URIs, never `/req/...` identifiers or individual test URIs.

### 4.4 Classification Model

Class selection and requirement applicability are different:

- **Modular selection:** OGC permits an implementation to select a class subject to prerequisites.
- **Prerequisite:** a class must already be satisfied when a dependent class is claimed.
- **Conditional requirement:** a numbered rule applies when its written condition is true.
- **Inherited:** the obligation is defined in another requirement class or standard.
- **Glaux target:** selected for the intended final implementation.
- **Evidence-gated claim:** omitted from a released declaration until all applicable evidence passes.
- **Qualified target:** selected, but an unresolved or draft dependency requires explicit claim language and change control.

No direct CSAPI class is classified as permanently deferred, not applicable, or out of scope.

---

## 5. CSAPI Part 1 Conformance Class Inventory

The exact requirement and conformance URI is `CS1` plus the displayed suffix. Direct content counts only Part 1 requirements/recommendations; inherited obligations are listed separately.

| Code | Requirement Class → Conformance Class | Direct Content | Normative Prerequisites | Main Scope / Artifact |
|---|---|---|---|---|
| P1-01 | [`/req/api-common`](https://docs.ogc.org/is/23-001/23-001.html#_req_api-common) → [`/conf/api-common`](https://docs.ogc.org/is/23-001/23-001.html#_conf_api-common) | Req 1–3; Rec 1 | Features 1 Core; Common 1 Core, Landing Page, JSON | IDs, temporal filtering, inherited API documents/HTTP |
| P1-02 | [`/req/system`](https://docs.ogc.org/is/23-001/23-001.html#_req_system) → [`/conf/system`](https://docs.ogc.org/is/23-001/23-001.html#_conf_system) | Req 4–8; Rec 2 | P1-01 | System resources, collections, links, location |
| P1-03 | [`/req/subsystem`](https://docs.ogc.org/is/23-001/23-001.html#_req_subsystem) → [`/conf/subsystem`](https://docs.ogc.org/is/23-001/23-001.html#_conf_subsystem) | Req 9–13 | P1-02 | System hierarchy, recursion, associations |
| P1-04 | [`/req/deployment`](https://docs.ogc.org/is/23-001/23-001.html#_req_deployment) → [`/conf/deployment`](https://docs.ogc.org/is/23-001/23-001.html#_conf_deployment) | Req 14–18 | P1-01 | Deployment resources, collections, System relation |
| P1-05 | [`/req/subdeployment`](https://docs.ogc.org/is/23-001/23-001.html#_req_subdeployment) → [`/conf/subdeployment`](https://docs.ogc.org/is/23-001/23-001.html#_conf_subdeployment) | Req 19–23 | P1-04 | Deployment hierarchy, recursion, associations |
| P1-06 | [`/req/procedure`](https://docs.ogc.org/is/23-001/23-001.html#_req_procedure) → [`/conf/procedure`](https://docs.ogc.org/is/23-001/23-001.html#_conf_procedure) | Req 24–28 | P1-01 | Procedure resources and no-geometry rule |
| P1-07 | [`/req/sf`](https://docs.ogc.org/is/23-001/23-001.html#_req_sf) → [`/conf/sf`](https://docs.ogc.org/is/23-001/23-001.html#_conf_sf) | Req 29–33 | P1-01, P1-02 | Generic Sampling Feature resources and System nesting |
| P1-08 | [`/req/property`](https://docs.ogc.org/is/23-001/23-001.html#_req_property) → [`/conf/property`](https://docs.ogc.org/is/23-001/23-001.html#_conf_property) | Req 34–37 | P1-01 | Non-feature Property Definition resources |
| P1-09 | [`/req/advanced-filtering`](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering) → [`/conf/advanced-filtering`](https://docs.ogc.org/is/23-001/23-001.html#_conf_advanced-filtering) | Req 38–59; Rec 3–5 | P1-01; WKT grammar indirectly | ID, keyword, geometry, relationship, derived, combined filters |
| P1-10 | [`/req/create-replace-delete`](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete) → [`/conf/create-replace-delete`](https://docs.ogc.org/is/23-001/23-001.html#_conf_create-replace-delete) | Req 60–71 | P1-01; Features 4 CRD draft | POST/PUT/DELETE, cascade, collection membership |
| P1-11 | [`/req/update`](https://docs.ogc.org/is/23-001/23-001.html#_req_update) → [`/conf/update`](https://docs.ogc.org/is/23-001/23-001.html#_conf_update) | Req 72–76 | P1-10; Features 4 Update draft | PATCH for Part 1 resources |
| P1-12 | [`/req/geojson`](https://docs.ogc.org/is/23-001/23-001.html#_req_geojson) → [`/conf/geojson`](https://docs.ogc.org/is/23-001/23-001.html#_conf_geojson) | Req 77–88 | P1-01; Features 1 GeoJSON | GeoJSON for System, Deployment, Procedure, Sampling Feature |
| P1-13 | [`/req/sensorml`](https://docs.ogc.org/is/23-001/23-001.html#_req_sensorml) → [`/conf/sensorml`](https://docs.ogc.org/is/23-001/23-001.html#_conf_sensorml) | Req 89–103 | P1-01; four SensorML 3 JSON classes | SensorML JSON for System, Deployment, Procedure, Property |

**Standards obligation:** Part 1 has no Core requirements class and expects at least one connected-system resource type and one encoding. Every selected class brings its prerequisites and applicable numbered requirements.

**Analysis:** P1-01 is the common prerequisite for every practical Part 1 profile, directly or transitively. P1-03 and P1-05 are hierarchy extensions. P1-09, P1-10, and P1-11 are modular capabilities. P1-12 and P1-13 are complementary: together they cover every Part 1 resource family supplied by the standard.

**Project recommendation:** target all 13. A complete Glaux release should support both defined Part 1 encodings for all applicable reads and, when P1-10 is active, all triggered writes.

---

## 6. CSAPI Part 2 Conformance Class Inventory

The exact requirement and conformance URI is `CS2` plus the displayed suffix.

| Code | Requirement Class → Conformance Class | Direct Content | Normative Prerequisites | Main Scope / Artifact |
|---|---|---|---|---|
| P2-01 | [`/req/api-common`](https://docs.ogc.org/is/23-002/23-002.html#_req_api-common) → [`/conf/api-common`](https://docs.ogc.org/is/23-002/23-002.html#_conf_api-common) | Req 1–2 | P1-01 | Features behavior adapted from features to resources |
| P2-02 | [`/req/datastream`](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream) → [`/conf/datastream`](https://docs.ogc.org/is/23-002/23-002.html#_conf_datastream) | Req 3–16 | P2-01 | DataStreams, Observations, schemas, navigation |
| P2-03 | [`/req/controlstream`](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream) → [`/conf/controlstream`](https://docs.ogc.org/is/23-002/23-002.html#_conf_controlstream) | Req 17–34 | P2-01 | ControlStreams, Commands, status, results, schemas |
| P2-04 | [`/req/feasibility`](https://docs.ogc.org/is/23-002/23-002.html#_req_feasibility) → [`/conf/feasibility`](https://docs.ogc.org/is/23-002/23-002.html#_conf_feasibility) | Req 35–39 | P2-03 | Feasibility request, status, result, collection |
| P2-05 | [`/req/system-event`](https://docs.ogc.org/is/23-002/23-002.html#_req_system-event) → [`/conf/system-event`](https://docs.ogc.org/is/23-002/23-002.html#_conf_system-event) | Req 40–44 | P2-01; P1-02 | System Event resources and System nesting |
| P2-06 | [`/req/advanced-filtering`](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering) → [`/conf/advanced-filtering`](https://docs.ogc.org/is/23-002/23-002.html#_conf_advanced-filtering) | Req 45–62 | P2-01; P1-09 | Dynamic-data, command, status, and event filters |
| P2-07 | [`/req/create-replace-delete`](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete) → [`/conf/create-replace-delete`](https://docs.ogc.org/is/23-002/23-002.html#_conf_create-replace-delete) | Req 63–78 | Features 4 CRD draft; resource-class conditions | Dynamic-resource and tasking POST/PUT/DELETE |
| P2-08 | [`/req/update`](https://docs.ogc.org/is/23-002/23-002.html#_req_update) → [`/conf/update`](https://docs.ogc.org/is/23-002/23-002.html#_conf_update) | Req 79–92 | P2-07; Features 4 Update draft; resource-class conditions | PATCH for Part 2 resources |
| P2-09 | [`/req/json`](https://docs.ogc.org/is/23-002/23-002.html#_req_json) → [`/conf/json`](https://docs.ogc.org/is/23-002/23-002.html#_conf_json) | Req 93–106 | SWE 3 JSON Record Components | JSON resource representations |
| P2-10 | [`/req/swecommon-json`](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-json) → [`/conf/swecommon-json`](https://docs.ogc.org/is/23-002/23-002.html#_conf_swecommon-json) | Req 107–114 | SWE 3 JSON Encoding Rules | SWE JSON Observation/Command payloads and schemas |
| P2-11 | [`/req/swecommon-text`](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-text) → [`/conf/swecommon-text`](https://docs.ogc.org/is/23-002/23-002.html#_conf_swecommon-text) | Req 115–122 | SWE 3 Text Encoding Rules | SWE text Observation/Command payloads and schemas |
| P2-12 | [`/req/swecommon-binary`](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-binary) → [`/conf/swecommon-binary`](https://docs.ogc.org/is/23-002/23-002.html#_conf_swecommon-binary) | Req 123–130 | SWE 3 Binary Encoding Rules | SWE binary Observation/Command payloads and schemas |

**Standards obligation:** Part 2 likewise defines no universal Core class. It expects at least one connected-system resource type and one encoding. Requirements 63–92 are resource-family-conditioned transaction obligations; write-media requirements in P2-09 through P2-12 are activated when CRD applies.

**Source-backed finding:** Every Part 2 requirement has a nominal abstract test with the same class and item suffix. The single class-level prerequisite contradiction is P2-06: its normative requirement class inherits P1-09, while Annex A omits that prerequisite.

**Project recommendation:** target all 12; preserve P1-09 as a prerequisite to P2-06; and treat P2-07/P2-08 as qualified until the Features Part 4 dependency is published and reconciled.

---

## 7. Inherited OGC Conformance and Dependency Inventory

### 7.1 Direct External Class Dependencies

| Source / Exact Class Identifier | Invoked By | Status | Glaux Consequence |
|---|---|---|---|
| `http://www.opengis.net/spec/ogcapi-common-1/1.0/{req|conf}/core` | P1-01 | Approved inherited | Common HTTP/query foundation must pass |
| `http://www.opengis.net/spec/ogcapi-common-1/1.0/{req|conf}/landing-page` | P1-01 | Approved inherited | Landing page and conformance/API links; detailed by IDR-009 |
| `http://www.opengis.net/spec/ogcapi-common-1/1.0/{req|conf}/json` | P1-01 | Approved inherited | JSON common documents and schema behavior |
| `http://www.opengis.net/spec/ogcapi-features-1/1.0/{req|conf}/core` | P1-01; behavior incorporated elsewhere | Approved inherited | Collections/items, HTTP, parameters, paging, CRS, conformance operation |
| `http://www.opengis.net/spec/ogcapi-features-1/1.0/{req|conf}/geojson` | P1-12 | Approved inherited | GeoJSON feature representation evidence |
| `http://www.opengis.net/spec/ogcapi-features-4/1.0/{req|conf}/create-replace-delete` | P1-10, P2-07 | Draft inherited | Pin revision, qualify claim, retest on publication |
| `http://www.opengis.net/spec/ogcapi-features-4/1.0/{req|conf}/update` | P1-11, P2-08 | Draft inherited | Same; Update also requires the corresponding CSAPI CRD class |
| `http://www.opengis.net/spec/sensorML/3.0/{req|conf}/json-simple-process` | P1-13 | Approved inherited | SensorML SimpleProcess validation/conformance |
| `http://www.opengis.net/spec/sensorML/3.0/{req|conf}/json-physical-system` | P1-13 | Approved inherited | SensorML PhysicalSystem validation/conformance |
| `http://www.opengis.net/spec/sensorML/3.0/{req|conf}/json-deployment` | P1-13 | Approved inherited | SensorML Deployment validation/conformance |
| `http://www.opengis.net/spec/sensorML/3.0/{req|conf}/json-derived-property` | P1-13 | Approved inherited | SensorML DerivedProperty validation/conformance |
| `http://www.opengis.net/spec/SWE/3.0/{req|conf}/json-record-components` | P2-09 | Approved inherited | JSON record-component model and constraints |
| `http://www.opengis.net/spec/SWE/3.0/{req|conf}/json-encoding-rules` | P2-10 | Approved inherited | SWE JSON encoding behavior |
| `http://www.opengis.net/spec/SWE/3.0/{req|conf}/text-encoding-rules` | P2-11 | Approved inherited | SWE text encoding behavior |
| `http://www.opengis.net/spec/SWE/3.0/{req|conf}/binary-encoding-rules` | P2-12 | Approved inherited | SWE binary encoding behavior |

OGC 06-103r4 WKT grammar is also incorporated by the Part 1 `geom` filtering requirement. It is a grammar dependency, not a CSAPI declaration class.

### 7.2 Cross-Part Dependencies

| Dependent Class | Direct CSAPI Prerequisite | Meaning |
|---|---|---|
| P2-01 through P2-06, directly or transitively | P1-01 | The Part 2 API/resource foundation is not standalone; this is not a formal inheritance claim for P2-07 through P2-12 |
| P2-05 System Event | P1-02 System | Events are exposed in relation to Systems |
| P2-06 Advanced Filtering | P1-09 Advanced Filtering | Part 2 adds filters to the Part 1 filtering foundation; Annex omission does not remove it |
| Conditional DataStream/ControlStream relationships | P1-02, P1-04, P1-07 as triggered | System, Deployment, and Sampling Feature support activates nested/association rules; these are requirement conditions, not additional class inheritance |
| P2-07/P2-08 transaction rows | P2 resource classes as triggered | Their formal inherited classes are Features Part 4 CRD/Update and P2-07 for Update; each implemented Part 2 resource family condition activates its transaction rows and brings that resource class's own prerequisites |
| P2-11 Req 118/121 and P2-12 Req 126/129 | P2-10 Req 110/113 mapping obligations are incorporated by reference | Text/Binary implementations must implement and test the incorporated component mappings, but this cross-reference does not formally require claiming P2-10 SWE JSON |

### 7.3 Normative Metadata Versus Annex A

| Affected Class | Published Difference | Controlling Glaux Treatment |
|---|---|---|
| P1-01 API Common | Annex uses broken `ogcapi-1` Core, adds draft Common 2 classes, and omits Common JSON | Use the coherent requirement-table prerequisites; preserve Annex discrepancy as an adapter record |
| P1-10 CRD | Annex uses broken `ogcapi-4` rather than `ogcapi-features-4` | Use the exact pinned Features Part 4 class |
| P1-11 Update | Annex omits P1-10 and uses broken `ogcapi-4` | Require P1-10 and valid Features Part 4 Update evidence |
| P1-12 GeoJSON | Annex uses broken `ogcapi-1` rather than `ogcapi-features-1` | Require the approved Features 1 GeoJSON class |
| P1-13 SensorML | Requirement table names four SensorML 3 classes; Annex carries a stale SensorML 2.1 dependency | SensorML 3 classes control; preserve and regression-test the defect |
| P2-06 Advanced Filtering | Annex omits P1-09 | Require and test P1-09 before declaring P2-06 |
| P1-02 System recommendation | `/rec/system/location` is omitted from class metadata but tested by A.7 | Keep warning/advisory; never promote absence to class failure |

**Analysis:** A declaration engine based only on Annex `inherit` lines would create false claims. The project needs a curated, versioned dependency registry grounded in the normative class tables and linked to documented Annex adapters.

---

## 8. Requirement-to-Conformance Mapping Matrix

### 8.1 How the Matrix Works

The minimum mapping fields required by the plan are normalized across §§5–9 instead of repeated in one unreadably wide table:

- §§5–7 supply standard, requirement class, conformance class, exact URI construction, prerequisites, and external artifacts.
- §8 supplies exhaustive requirement membership, summary, standards status, implementation dependency, and material upstream/release context.
- §9 supplies Glaux applicability, claim evidence, tests, and downstream handoff.
- Appendices A–B preserve every exact direct requirement identifier.

The class code is the stable join key. The accepted IDR-SRV-006/007 reports retain the complete lettered obligation, condition, schema, source anchor, and per-requirement test analysis. This report consolidates them without duplicating 233 prose extractions.

### 8.2 Part 1 Requirement-to-Class Map

| Key | Exact Direct Membership / Source | Requirement Summary | Standards Status | Artifact, Upstream, and Implementation Dependency |
|---|---|---|---|---|
| P1-01 | [Req 1–3](idr-srv-006-csapi-part-1-requirement-baseline-report.md#61-api-common-systems-and-subsystems); Rec 1 | Per-resource-type local ID uniqueness; global URI UID uniqueness; `validTime` filtering | Prerequisite; all rows mandatory when selected; Rec 1 advisory | Common/Features API foundation; #28/PR #29 identifier relationships are PB and included in v1.0.0 |
| P1-02 | [Req 4–8](idr-srv-006-csapi-part-1-requirement-baseline-report.md#61-api-common-systems-and-subsystems); Rec 2 | Latest reported System location; canonical/list endpoints; typed collections | Modular resource class; requirements mandatory; Rec 2 advisory | Resource model, routes, collections, GeoJSON/SensorML; #150 is unresolved history but published `/rec` controls |
| P1-03 | [Req 9–13](idr-srv-006-csapi-part-1-requirement-baseline-report.md#61-api-common-systems-and-subsystems) | Subsystem nesting, recursion, filtering at all depths, aggregated associations | Modular hierarchy class; mandatory with P1-02 prerequisite | Hierarchical persistence, cycle protection, nested routes and links |
| P1-04 | [Req 14–18](idr-srv-006-csapi-part-1-requirement-baseline-report.md#62-deployments-subdeployments-and-procedures) | Deployment canonical/list/typed endpoints and conditioned System relation | Modular resource class | Deployment model, time, collections, relationship integrity; ATS condition adapter needed |
| P1-05 | [Req 19–23](idr-srv-006-csapi-part-1-requirement-baseline-report.md#62-deployments-subdeployments-and-procedures) | Subdeployment nesting, query parity, recursion, association aggregation | Modular hierarchy class; depends P1-04 | Hierarchical persistence, filtering, cycle and membership integrity |
| P1-06 | [Req 24–28](idr-srv-006-csapi-part-1-requirement-baseline-report.md#62-deployments-subdeployments-and-procedures) | Procedure no-geometry rule, canonical/list endpoints, typed collections | Modular resource class | Procedure model, routes, encodings, schema validation |
| P1-07 | [Req 29–33](idr-srv-006-csapi-part-1-requirement-baseline-report.md#63-sampling-features-and-property-definitions) | Generic Sampling Feature routes, collections, mandatory System nesting | Modular resource class; depends P1-02 | #82/PR #96 PB boundary included in tag; specialized SF types are not Part 1 claims |
| P1-08 | [Req 34–37](idr-srv-006-csapi-part-1-requirement-baseline-report.md#63-sampling-features-and-property-definitions) | Non-feature Property Definition canonical/list behavior and collections | Modular resource class | Property model, adapted Features behavior, SensorML schema |
| P1-09 | [Req 38–59](idr-srv-006-csapi-part-1-requirement-baseline-report.md#64-advanced-filtering); Rec 3–5 | ID, keyword, property, geometry, resource relationship, derived and combined filters | Optional under OGC; all applicable requirements mandatory when claimed; recs advisory | Query engine, WKT, graph traversal, indexes, negative grammar tests |
| P1-10 | [Req 60–71](idr-srv-006-csapi-part-1-requirement-baseline-report.md#65-create-replace-delete-and-update) | Create/replace/delete for resources, cascades, and collection membership | Optional transaction class; resource-conditioned; qualified target | Pinned Features 4 CRD draft, persistence, atomicity; #141 UP; encoding writes activate under #23 |
| P1-11 | [Req 72–76](idr-srv-006-csapi-part-1-requirement-baseline-report.md#65-create-replace-delete-and-update) | PATCH for System, Deployment, Procedure, Sampling Feature, Property | Optional transaction class; depends P1-10; qualified target | Pinned Features 4 Update draft, partial validation/concurrency; #141 UP |
| P1-12 | [Req 77–88](idr-srv-006-csapi-part-1-requirement-baseline-report.md#66-geojson-encoding) | GeoJSON media types, schemas, links, IDs, and per-resource mappings | Encoding selection class; write row conditional on CRD | Features GeoJSON, CS schemas/mapping tables, OpenAPI; #23 UP; resource-class conditions are incompletely expressed |
| P1-13 | [Req 89–103](idr-srv-006-csapi-part-1-requirement-baseline-report.md#67-sensorml-json-encoding) | SensorML media types, IDs, relations, schemas, classes, and mappings | Encoding selection class; write row conditional on CRD | Four SensorML 3 classes, schema graph, mapping tables; #23 UP; Annex carries stale SensorML 2.1 metadata |

### 8.3 Part 2 Requirement-to-Class Map

| Key | Exact Direct Membership / Source | Requirement Summary | Standards Status | Artifact, Upstream, and Implementation Dependency |
|---|---|---|---|---|
| P2-01 | [Req 1–2](idr-srv-007-csapi-part-2-requirement-baseline-report.md#61-api-common-requirements-12) | Adapt inherited feature terminology and collection behavior to Part 2 resources | Prerequisite for dependent Part 2 classes | P1-01 and Features collection behavior; Part 2 class PURLs 404 under #152 |
| P2-02 | [Req 3–16](idr-srv-007-csapi-part-2-requirement-baseline-report.md#62-datastreams-and-observations-requirements-316) | DataStream/Observation routes, links, associations, schemas, collections | Modular resource class; four formal conditions | Dynamic-data model, time-series storage, schema operations, P1 relations; stale `/req/sampling` interpreted as `/req/sf` |
| P2-03 | [Req 17–34](idr-srv-007-csapi-part-2-requirement-baseline-report.md#63-controlstreams-commands-status-and-results-requirements-1734) | ControlStream, Command, status/result routes, links, schemas, collections | Modular resource class; four formal conditions | Tasking lifecycle, authorization boundary, persistence, P1 relations; route/ATS defects retained |
| P2-04 | [Req 35–39](idr-srv-007-csapi-part-2-requirement-baseline-report.md#64-feasibility-requirements-3539) | Feasibility canonical/submission/status/result/collection behavior | Modular capability; depends P2-03 | Feasibility lifecycle and async result model; copied ATS procedures need adapters |
| P2-05 | [Req 40–44](idr-srv-007-csapi-part-2-requirement-baseline-report.md#65-system-events-requirements-4044) | System Event routes, System nesting, collections | Modular resource class; depends P1-02 | Event persistence/model/time and JSON encoding; Annex route/resource defects |
| P2-06 | [Req 45–62](idr-srv-007-csapi-part-2-requirement-baseline-report.md#66-advanced-filtering-requirements-4562) | Time, property, FOI, status, sender, and event filters | Optional under OGC; depends P2-01 and P1-09 | Query/storage indexes; Annex omits P1-09 and has incomplete `latest`/route tests |
| P2-07 | [Req 63–78](idr-srv-007-csapi-part-2-requirement-baseline-report.md#67-create-replace-and-delete-requirements-6378) | CRD for dynamic, command, feasibility, result/status, and event resources | Optional; all 16 rows formally resource-conditioned; qualified target | Pinned Features 4 CRD, lifecycle/persistence/atomicity; #141 UP; activates encoding write rows under #23 |
| P2-08 | [Req 79–92](idr-srv-007-csapi-part-2-requirement-baseline-report.md#68-update-requirements-7992) | PATCH for the same Part 2 resource families | Optional; all 14 rows resource-conditioned; depends P2-07; qualified | Pinned Features 4 Update, partial schema and concurrency; #141 UP |
| P2-09 | [Req 93–106](idr-srv-007-csapi-part-2-requirement-baseline-report.md#69-json-encoding-requirements-93106) | JSON media types, schemas, constraints for all Part 2 resource families | Encoding selection class; write conditional on CRD | SWE JSON record components and CS JSON schemas; #23 UP |
| P2-10 | [Req 107–114](idr-srv-007-csapi-part-2-requirement-baseline-report.md#610-swe-common-json-encoding-requirements-107114) | SWE JSON schema/mapping and Observation/Command payload encoding | Encoding selection class; write conditional on CRD | SWE JSON encoding rules; logical schema/component mapping; #23 UP |
| P2-11 | [Req 115–122](idr-srv-007-csapi-part-2-requirement-baseline-report.md#611-swe-common-text-encoding-requirements-115122) | SWE text schema/mapping and Observation/Command encoding | Encoding selection class; write conditional on CRD | SWE text encoding rules; separators/grammar/golden streams; #23 UP |
| P2-12 | [Req 123–130](idr-srv-007-csapi-part-2-requirement-baseline-report.md#612-swe-common-binary-encoding-requirements-123130) | SWE binary schema/mapping and Observation/Command encoding | Encoding selection class; write conditional on CRD | SWE binary encoding rules; framing/types/golden streams; #23 UP |

### 8.4 Mapping Completeness and Ambiguities

**Source-backed finding:** Part 2 has zero missing, extra, or mistargeted identifier pairs: every `/req/<class>/<item>` has a nominal `/conf/<class>/<item>` test. Part 1 likewise has one test for each of 103 requirements and five recommendations, plus two supporting tests.

**Analysis:** Identifier completeness does not imply procedural completeness. Known tests use wrong resource names, wrong routes/media types, unresolved placeholders, incomplete conditions, or vacuous fixtures. Those defects change the executable adapter, not the requirement-to-class membership.

No numbered requirement lacks a direct class. The interpretation risks that matter to this profile are:

- broken or omitted prerequisite identifiers in Annex A;
- stale Part 2 `/req/sampling` conditions, interpreted as the published Part 1 `/req/sf` class;
- resource-specific encoding rows whose conditions are incompletely expressed;
- conditional write rows activated by CRD and the unresolved declaration granularity in #23;
- recommendation tests that must remain warnings rather than failures; and
- lettered obligations and incorporated tables/schemas that a one-row-per-number ledger must retain beneath the class mapping.

### 8.5 Condition and Cross-Class Interpretation Overlay

This overlay controls staged claims where the published condition metadata is defective or incomplete. It does not rewrite the standard; each interpretation remains linked to the published wording and the accepted Part 1/Part 2 baselines.

| Affected Requirement(s) | Published Trigger or Defect | Controlling Glaux Mapping | Staged-Claim Consequence |
|---|---|---|---|
| P1 Req 17, `/req/deployment/ref-from-system` | Applies when Systems and the `deployments` association are provided; A.22 assumes System support | Gate the test and obligation by P1-02 plus the stated association condition | An untriggered association row does not block P1-04; a triggered but failing row does |
| P1 Req 60–66 | CRD rows are conditioned on the corresponding System, Subsystem, Deployment, Subdeployment, Procedure, or Sampling Feature capability | Map each row to P1-02 through P1-07 as named by its resource | P1-10 must pass every row whose resource class is implemented |
| P1 Req 67, `/req/create-replace-delete/property` | Condition contains unresolved `[clause-derived-properties]` | Interpret the intended trigger as P1-08 `/req/property` | Property support plus P1-10 activates the row |
| P1 Req 72–75 | Update rows are conditioned on System, Deployment, Procedure, or Sampling Feature capability | Map to P1-02, P1-04, P1-06, and P1-07 respectively | P1-11 must pass each implemented resource-family row |
| P1 Req 76, `/req/update/property` | Condition contains unresolved `[clause-derived-properties]` | Interpret the intended trigger as P1-08 `/req/property` | Property support plus P1-11 activates the row |
| P1 Req 78 and 90 | GeoJSON/SensorML write-media rows activate when P1-10 CRD is implemented | Require the relevant encoding on every applicable claimed transaction surface under the conservative #23 policy | Withhold a conflicting encoding claim rather than weakening the row |
| P1 Req 81–88 | GeoJSON System/Deployment/Procedure/Sampling Feature schema and mapping rows lack explicit resource-class conditions despite the modular premise | Treat the rows as literal obligations of P1-12 pending clarification; the full Glaux target implements all four families | A narrower staged build should withhold P1-12 unless it satisfies every row, rather than assume an unpublished condition |
| P1 Req 94, 97, 99, 102 | Conditions cite nonexistent `/req/system-features`, `/req/deployment-features`, `/req/procedure-features`, and `/req/property-definitions` | Map respectively to P1-02 `/req/system`, P1-04 `/req/deployment`, P1-06 `/req/procedure`, and P1-08 `/req/property` | P1-13 evidence applies the intended actual resource-class gate and preserves the defect record |
| P2 Req 118/121 and 126/129 | Text/Binary rules incorporate component mappings from P2 Req 110/113 | Implement/test the incorporated mapping obligations without inventing formal P2-10 inheritance | P2-11 or P2-12 may be claimed without P2-10 only if the incorporated mappings and all direct prerequisites still pass |

**Analysis:** this table prevents both false negatives and false positives. It stops an inapplicable conditional row from failing a class, but it also stops a staged implementation from treating broken or absent condition text as permission to skip an obligation.

---

## 9. Glaux Server Conformance Posture

### 9.1 Class-by-Class Applicability and Claim Gate

`Target` below means part of the intended final Rust server. It does not mean implemented today.

| Class | Glaux Applicability | Evidence Required Before Runtime Claim | Principal Test / Handoff |
|---|---|---|---|
| P1-01 API Common | Target; foundation | Common + Features prerequisites; IDs/UIDs/time; A.1–A.6 and supplements | Contract, negative query, uniqueness; IDR-009/011/016/050/051 |
| P1-02 System | Target | P1-01; A.7–A.12; schemas/encodings; advisory location handled correctly | Resource/navigation/location; IDR-010/015/018/021 |
| P1-03 Subsystem | Target | P1-02; hierarchy/recursion/filter/association scenarios beyond A.13–A.17 | Graph and cycle tests; IDR-010/011/015/017/025 |
| P1-04 Deployment | Target | P1-01; A.18–A.22 with condition adapter; schemas/encodings | Resource/time/relationships; IDR-010/015/017/018 |
| P1-05 Subdeployment | Target | P1-04; A.23–A.27 plus depth/filter/cycle scenarios | Graph/query tests; IDR-010/011/015/025 |
| P1-06 Procedure | Target | P1-01; A.28–A.32; no geometry; both applicable encodings | Model/schema negatives; IDR-010/015/021/023 |
| P1-07 Sampling Feature | Target | P1-02; A.33–A.37; generic SF boundary; nested membership | Resource/relationship tests; IDR-010/015/017/024 |
| P1-08 Property | Target | P1-01; A.38–A.41; adapted collection behavior and SensorML mapping | Non-feature/schema tests; IDR-010/015/021/023 |
| P1-09 Advanced Filtering | Target | P1-01; A.42–A.66; every filter/combination/negative/recursive case | Query/fixture corpus; IDR-011/018/024/050/053 |
| P1-10 CRD | Target, qualified | Pinned Features 4 CRD; every triggered route/encoding; A.67–A.78 plus atomicity/security | Lifecycle/persistence; IDR-010A/013/029/030/031/050 |
| P1-11 Update | Target, qualified | P1-10 + pinned Features 4 Update; A.79–A.83 plus PATCH/schema/concurrency | Update semantics; IDR-010A/013/029/031/050 |
| P1-12 GeoJSON | Target | Features GeoJSON; A.84–A.95; all applicable read/write, schema, mapping and golden evidence | Negotiation/schema/client; IDR-012/014/015/021/023/053 |
| P1-13 SensorML | Target | Four SensorML classes; A.96–A.110; mapping tables and offline schema graph | Schema/profile/client; IDR-012/015/021/023/053 |
| P2-01 API Common | Target; Part 2 foundation | P1-01; A.1–A.2 plus adapted collection and negative behavior | API contract; IDR-009/010/011/050 |
| P2-02 DataStream | Target | P2-01; A.3–A.16 plus route, time, membership, schema and empty/negative cases | Dynamic data; IDR-010/017/018/027/034/050 |
| P2-03 ControlStream | Target | P2-01; A.17–A.34 with route adapters; command/status/result lifecycle and authorization | Tasking; IDR-010/036/038/039/050 |
| P2-04 Feasibility | Target | P2-03; corrected A.35–A.39; sync/async, rejection, no-result scenarios | Feasibility/tasking; IDR-037/038/050/053 |
| P2-05 System Event | Target | P2-01 + P1-02; corrected A.40–A.44; System membership/type/time | Events/status; IDR-017/018/020/035/050 |
| P2-06 Advanced Filtering | Target | P1-09 + P2-01; A.45–A.62 plus `latest`, invalid, multi-value, paging, empty cases | Query/storage; IDR-011/018/034/036/050 |
| P2-07 CRD | Target, qualified | Pinned Features 4 CRD; every triggered resource/encoding; A.63–A.78 plus lifecycle/atomicity/security | Persistence/tasking; IDR-029/031/034/036/038/050 |
| P2-08 Update | Target, qualified | P2-07 + pinned Features 4 Update; A.79–A.92 plus PATCH/immutability/concurrency | Update/persistence; IDR-029/031/034/036/050 |
| P2-09 JSON | Target | SWE record components; A.93–A.106; all resource schemas, UTC and read/write combinations | Schema/negotiation; IDR-012/014/022/023/053 |
| P2-10 SWE JSON | Target | SWE JSON rules; A.107–A.114; component mappings, round trips, malformed/boundary cases | Codec/interoperability; IDR-022/023/034/036/053 |
| P2-11 SWE Text | Target | SWE text rules; corrected A.115–A.122; grammar, separators, mappings, malformed streams | Codec/interoperability; IDR-022/023/034/036/053 |
| P2-12 SWE Binary | Target | SWE binary rules; corrected A.123–A.130; types, framing, mappings, truncation | Codec/interoperability; IDR-022/023/034/036/053 |

### 9.2 Full-Scope Profile Decision

**Project recommendation:** select all 25 direct classes as the versioned Glaux end-state profile. Classify P1-10, P1-11, P2-07, and P2-08 as `target-qualified` while Features Part 4 remains draft. Classify every class at runtime as `unclaimed`, `evidence-incomplete`, `claimable`, or `claimed` for a specific build. Do not use `deferred` to disguise removal from the final goal.

The four encoding families are not redundant:

- P1-12 and P1-13 represent connected-system feature resources.
- P2-09 represents Part 2 resource documents.
- P2-10 through P2-12 represent Observation and Command data values using SWE encodings.

### 9.3 Dependency-Led Sequencing Implications

This is a dependency map, not the final roadmap:

| Dependency Wave | Classes / Capability | What It Unlocks |
|---|---|---|
| 0. Evidence foundation | Versioned profile registry, exact standards/artifact pins, inherited Common/Features baseline | Truthful build-specific claims and test traceability |
| 1. Feature-resource foundation | P1-01, P1-02, P1-04, P1-06, P1-07, P1-08 with P1-12/P1-13 | Core static resource graph, representations, discovery/navigation design |
| 2. Hierarchy and query | P1-03, P1-05, P1-09 | Complete nested graph and query foundation; prerequisite for P2-06 |
| 3. Dynamic read and representations | P2-01, P2-02, P2-05, P2-09 through P2-12 | Observation/event access, storage/codec evidence |
| 4. Tasking | P2-03, P2-04 | Commands, status/results, feasibility, authorization/safety research |
| 5. Transactions | P1-10/P1-11 and P2-07/P2-08 after model/persistence/validation decisions | Full write/update surface; activates encoding write obligations |

Resource models, schemas/codecs, and test fixtures can be researched or implemented in parallel after their prerequisites stabilize. A staged release may stop between waves; the end-state profile remains all 25.

---

## 10. Upstream Standards-Maintenance Context and Disposition

All states below were checked on August 1, 2026. `PB` means the outcome is present in the published baseline; `UP` means unresolved proposal/defect; `Mixed` separates a published outcome from unfinished work.

| Record | State / Authority | Publication Relationship | Mapping Consequence |
|---|---|---|---|
| [Issue #23](https://github.com/opengeospatial/ogcapi-connected-systems/issues/23) | Open, updated 2026-07-21; UP | No merged/released resolution | Encoding classes remain broad; CRD activates write-media rows. Keep a separate operation/resource/method/media matrix and do not invent combinatorial class URIs. |
| [Issue #28](https://github.com/opengeospatial/ogcapi-connected-systems/issues/28) / [PR #29](https://github.com/opengeospatial/ogcapi-connected-systems/pull/29) | Closed/merged; PB | PR merge `c5de49ba...` and URI correction `e5541413...` are ancestors of v1.0.0 | Preserve `/req`, `/conf`, `/rec`, and `/per` roles and three-segment atomic IDs; emit only class `/conf` URIs in declarations. |
| [Issue #82](https://github.com/opengeospatial/ogcapi-connected-systems/issues/82) / [PR #96](https://github.com/opengeospatial/ogcapi-connected-systems/pull/96) | Closed/merged; PB | Merge `191212a4...` is an ancestor of v1.0.0 | Part 1 exposes generic `/req/sf` ↔ `/conf/sf`; specialized Sampling Feature schemas moved to future Part 4 work. |
| [Issue #141](https://github.com/opengeospatial/ogcapi-connected-systems/issues/141) | Open, updated 2026-07-21; UP | No merged/released resolution; Features Part 4 remains draft | Retain all four CSAPI transaction classes as qualified targets; pin current draft and require publication delta review. |
| [Issue #150](https://github.com/opengeospatial/ogcapi-connected-systems/issues/150) | Open, updated 2026-07-21; UP history, unambiguous published baseline | No change; published `/rec/system/location` and A.7 warning remain | Missing System location is advisory, not a class failure; incorrect latest-location behavior when supplied is still a requirement failure. |
| [Issue #152](https://github.com/opengeospatial/ogcapi-connected-systems/issues/152) | Open, updated 2026-07-21; Mixed | Part 2 is published; PURL-mapping checkbox remains open | All 24 Part 2 class PURLs and sampled detailed IDs returned 404. Emit exact canonical identifiers anyway and monitor resolver health separately. |

**Source-backed finding:** the current NamingAuthority tree contains `specification-elements/mappings/23-001.csv` but no `23-002.csv`. All 13 Part 1 requirement-class and 13 conformance-class PURLs resolved to correct anchors; all corresponding Part 2 class PURLs returned 404. Part 1 recommendation `/rec/system/location` also returned 404 even though the requirement and conformance mappings resolve.

**Analysis:** URI identity and dereferenceability are separate. Replacing an exact `http` identifier with an `https` URL, a document anchor, or a Glaux-local alias would create the wrong declaration. Human documentation should pair the literal identifier with a stable approved-document anchor.

**Artifact finding:** the tagged Part 1 OAS defines `/conformance` and references the OGC API - Features `confClasses` response schema; the tagged Part 2 OAS does not duplicate that inherited operation. The tagged Part 1 example `api/part1/openapi/examples/confClasses.json`, however, is not an exhaustive or correct declaration. It includes Common Part 2 `0.0` classes, nonexistent CSAPI Part 1 `/conf/sampling`, nonexistent Part 2 `/conf/geojson`, and omits most CSAPI classes. Its tagged conformance response description also says “requirements classes” although the payload contains conformance classes. These are informative defects for IDR-009/014, not a replacement profile or a reason to change the normative URI set.

---

## 11. Conformance Declaration Implications

### 11.1 Standards Obligation

Inherited OGC API - Features Core requires:

- `GET {root}/conformance`;
- a successful `200` response;
- an object containing required `conformsTo`, an array of strings; and
- the URIs of all conformance classes implemented by the API.

The landing page links to this resource with relation `conformance`. Detailed representation, content negotiation, and link behavior remain IDR-SRV-009.

### 11.2 Glaux Declaration Policy

| Capability State for a Released Build | Put Class URI in `conformsTo`? | Treatment |
|---|---|---|
| Planned end-state only | No | Keep in the versioned target profile/roadmap |
| Code exists but prerequisites or tests are incomplete | No | Mark evidence-incomplete internally |
| Experimental extension or feature flag | No, unless the class fully passes in the exposed configuration | Document as experimental outside the standards declaration |
| Every prerequisite and applicable direct requirement passes | Yes | Emit the exact canonical class URI for that build |
| Previously claimed but regression or disabled dependency exists | No | Remove claim; retain historical build evidence |
| OGC-certified through an applicable official program | Yes, with separate certification record | Certification is additional evidence, not inferred from self-declaration |

A claim gate must prove all of the following:

1. exact standard/profile version and released build are pinned;
2. every normative prerequisite class is itself demonstrated;
3. every unconditional direct requirement and incorporated obligation passes;
4. every condition triggered by the build's resource, method, relation, or encoding choices passes;
5. the normative ATS passes through documented adapters where necessary;
6. supplemental negative, schema, lifecycle, and interoperability evidence passes;
7. runtime behavior, implementation-specific OpenAPI, and `conformsTo` agree; and
8. unresolved exceptions and draft qualifications are visible rather than silently waived.

### 11.3 Exact CSAPI Declaration Set

The final all-class target contains these class suffixes under the exact bases in §4.3:

- Under `CS1`: `/conf/api-common`, `/conf/system`, `/conf/subsystem`, `/conf/deployment`, `/conf/subdeployment`, `/conf/procedure`, `/conf/sf`, `/conf/property`, `/conf/advanced-filtering`, `/conf/create-replace-delete`, `/conf/update`, `/conf/geojson`, and `/conf/sensorml`.
- Under `CS2`: `/conf/api-common`, `/conf/datastream`, `/conf/controlstream`, `/conf/feasibility`, `/conf/system-event`, `/conf/advanced-filtering`, `/conf/create-replace-delete`, `/conf/update`, `/conf/json`, `/conf/swecommon-json`, `/conf/swecommon-text`, and `/conf/swecommon-binary`.

The implementation must concatenate and store the expanded canonical strings, not the `CS1`/`CS2` abbreviations. A complete declaration should also include every inherited Common, Features, SensorML, SWE, and eventually Features Part 4 conformance class actually demonstrated. IDR-SRV-009 must verify the final inherited list and wire representation.

### 11.4 Encoding and Operation Advertisement

**Standards obligation:** when CRD is implemented, the published encoding classes activate their write-media requirements. An OpenAPI document cannot waive a numbered requirement.

**Analysis:** a class declaration answers “does this API satisfy the whole class?” The implementation-specific OpenAPI document answers “which method, resource, media type, and schema are available at this operation?” Both are required, and they must agree.

**Project recommendation pending upstream resolution:** do not declare an encoding class in a configuration that violates a triggered write requirement. For the full Glaux profile, implement all defined read and applicable write combinations. Maintain a separately tested matrix keyed by endpoint, method, request/response media type, schema, resource family, and authorization condition.

---

## 12. Verification and Test-Strategy Implications

### 12.1 Available Official Evidence

| Resource | Coverage | Permitted Claim |
|---|---|---|
| Part 1 normative ATS | 13 classes; 110 tests including 5 recommendations and 2 helpers | Normative abstract evidence for Part 1 when faithfully executed and supplemented |
| Part 2 normative ATS | 12 classes; 130 tests | Normative abstract evidence for Part 2 when faithfully executed and supplemented |
| Official OGC API - Features ETS | Features Core plus its optional supported areas; useful for Part 1 feature collections | Inherited Features evidence only; never proves a CSAPI class |
| Public official CSAPI ETS | None found | No executable official CSAPI certification inference is currently available |

The official Features ETS expects its own supported OAS/representation profile and ignores unknown CSAPI class URIs. Its result must therefore remain a separate inherited-evidence lane. IDR-SRV-050 must determine how to reconcile the ETS's OpenAPI 3.0 expectations with CSAPI's released OAS 3.1 artifacts without falsely declaring an OAS class.

The negative CSAPI-ETS finding was checked on August 1, 2026 against the public `opengeospatial` GitHub organization/repository search, the tagged CSAPI source and release, and both public OGC Validator suite listings at [cite.opengeospatial.org](https://cite.opengeospatial.org/teamengine/) and [cite.ogc.org](https://cite.ogc.org/te2/). Neither hosted listing exposed a Connected Systems suite. This bounded public search cannot exclude private or unfinished work.

### 12.2 Required Test Families

| Class Group | Positive / Contract Evidence | Negative and Supplemental Evidence | Fixtures / External Evidence |
|---|---|---|---|
| API Common and inherited | Landing/API/conformance, HTTP, parameters, IDs, time, paging | Unknown/invalid queries, duplicate/cross-type IDs, malformed URIs, temporal boundaries | Multiple collections/types; official Features ETS |
| Part 1 resource classes | Canonical/nested/typed routes, purity, links, models | Wrong type, missing member, bad link, invalid geometry/cardinality, empty/nonempty cases | Multi-collection graph; external CSAPI clients |
| Hierarchy classes | Direct/all-descendant traversal, aggregated associations | Cycles, duplicates, invalid parent, every filter at every depth | Three-plus-level System/Deployment graphs |
| Advanced filtering | Each filter, AND combinations, paging | Invalid grammar/type/WKT/time, `latest`, multi-value, absent/null, no-match | Boundary and relationship corpus; query oracle |
| CRD and Update | POST/PUT/DELETE/PATCH, status, persistence effect | Schema/media failures, conflicts, cascade false/true, atomicity, concurrency, rollback, authorization | Lifecycle scenarios; persistent-state assertions |
| Part 1 encodings | Negotiation, schemas, mapping tables, canonical links/IDs | Wrong media/schema/class, mapping omissions, cross-encoding replacement risk | Valid/invalid GeoJSON and SensorML golden files; external validators/clients |
| DataStream/System Event | Canonical/nested routes, time intersections, schemas | Wrong membership, stale IDs, empty history, invalid time/filter | Time-series/event corpus; external consumer |
| Control/Feasibility | Command/status/result/feasibility lifecycle | Invalid transitions, missing results, rejection, timeout, unauthorized actions | Sync/async tasking scenarios and fake executor boundary |
| Part 2 JSON/SWE | Schema/component mappings, encode/decode, negotiation | UTC failures, malformed/truncated values, separators/framing, unsupported combinations | Golden JSON/Text/Binary streams; independent codec/client round trips |

**Streaming finding:** Parts 1 and 2 define no standalone transport-streaming conformance class. None of the 25 direct class claims therefore proves Server-Sent Events, WebSocket, MQTT, or another live-streaming transport. If Glaux adds streaming around DataStreams, Commands, System Events, or SWE-encoded values, tests must still prove the underlying P2-02/P2-03/P2-05 and P2-10 through P2-12 obligations; transport-specific behavior is separate conditional project evidence owned by IDR-SRV-035 and IDR-SRV-054.

### 12.3 ATS Defect Handling

Known Annex defects include wrong routes or resource nouns, wrong media types, missing prerequisite/condition gates, unresolved placeholders, incomplete `latest`/UTC/mapping checks, vacuous collection discovery, and tests that inspect advertisement without proving payload processing.

The harness policy should be:

1. preserve the original ATS identifier and text;
2. execute a literal or faithful machine interpretation where possible;
3. attach a versioned adapter when a defect prevents meaningful execution;
4. record the authority and rationale for the adapter;
5. retain a regression that exposes the upstream defect; and
6. add supplemental tests for the underlying normative obligation.

Recommendation tests remain `warning/advisory`, unless Glaux deliberately creates a separate profile requirement. This is especially important for `/rec/system/location` under issue #150.

### 12.4 Evidence Lanes and Claim Record

Glaux needs three visibly separate evidence lanes:

- **Normative CSAPI lane:** all applicable Part 1/2 ATS procedures and documented adapters.
- **Inherited official lane:** official Common/Features/SensorML/SWE tools or evidence where available.
- **Glaux quality lane:** negative, security, lifecycle, property-based, performance, interoperability, and known-defect tests.

One class becomes `claimable` only when the release's requirement records close over the full prerequisite graph and all required lanes pass. A PURL-health monitor is useful operational evidence but must not fail Glaux conformance because OGC's resolver is unavailable.

### 12.5 Information Required by IDR-SRV-050 and IDR-SRV-051

- the 25 class records and 233 direct requirement records from this report and accepted baselines;
- five advisory recommendation records and two Part 1 helper tests;
- exact prerequisite graph and 42 formal Part 2 condition rows, plus Part 1 conditions;
- official test URI, source procedure, severity, fixtures, and known defect/adapter ID;
- operation/resource/method/media/schema matrix for issue #23;
- immutable standard, schema, OAS, and draft dependency pins;
- test result, implementation reference, build identity, and declaration state;
- explicit distinction between official ATS, inherited official ETS, and Glaux supplemental evidence; and
- a generation/check rule preventing `conformsTo` from drifting away from the evidence registry.

---

## 13. Downstream Topic Handoff Matrix

| Topic(s) | Input from This Report | Required Outcome |
|---|---|---|
| IDR-009 | Exact class set, inherited candidates, PURL/OAS defects, evidence-gated declaration policy | Landing/API/conformance wire behavior and canonical declaration registry |
| IDR-010/010A | Resource/hierarchy classes, routes, canonical behavior, draft transaction dependency | Endpoint/navigation and versioning/extension contract |
| IDR-011/012/013/014 | Filter classes; encoding set; transaction/error implications; OAS capability matrix | Query, negotiation, errors, and implementation-specific OpenAPI that agree with claims |
| IDR-015/016/017/018/020 | Resource classes, IDs, links, conditions, time, System Event | Domain, identity, relationship, temporal, and status/event invariants |
| IDR-021/022/023/024 | SensorML/SWE class prerequisites, schemas, mappings, semantic dependencies | Reproducible validation/codecs and semantic profile |
| IDR-025/027/029/030/031/034 | Hierarchy/dynamic/transaction classes and activation conditions | Persistence, lifecycle, cascade, concurrency, time-series behavior |
| IDR-035/036/037/038 | Part 2 control, feasibility, event and encoding scope | Streaming boundary and safe tasking lifecycle without overstating Part 2 |
| IDR-039–043 | Every exposed read/write/tasking surface and claim evidence | Security, authorization, policy, audit, DDIL and deployment controls |
| IDR-044–049 | Full 25-class target and evidence architecture | Rust architecture, crates, quality, observability, and delivery choices |
| IDR-050 | 25 classes, 240 ATS tests, inherited lanes, adapter rules, ETS gap | Executable conformance harness design |
| IDR-051/052 | Exact trace record and claim gate | Requirement-to-test system and TDD operating model |
| IDR-053/054/055/056 | Fixture and external evidence requirements | Interoperability, performance, security, and release-readiness evidence |
| IDR-057 | Accepted profile, residual risks, later deltas | Final synthesis and current-state reconciliation |

---

## 14. Recommendations and Decision Analysis

### 14.1 Recommended Decision

Adopt **Option C**:

| Option | Consequence | Decision |
|---|---|---|
| A. Implement only a minimal OGC subset | Easiest start, but underserves the full reference-server goal and forces later scope recovery | Reject as end state |
| B. Target all classes and list all of them immediately | Preserves ambition but creates false claims and weakens trust | Reject |
| C. Target all 25 classes, sequence by dependency, and declare only evidence-complete classes per release | Preserves full scope, supports staged delivery, and keeps conformance credible | **Recommend** |

### 14.2 Project Recommendations

1. **Freeze the end-state direct class set at all 25.** Change it only through explicit project governance, never incidental implementation difficulty.
2. **Create one versioned conformance-profile registry.** Generate or verify runtime declarations, documentation, and test selection from that same source.
3. **Make the normative requirement-table dependency graph controlling.** Preserve Annex contradictions as named adapters and regressions.
4. **Use a conservative issue-#23 policy.** Implement all triggered write/encoding combinations in the full profile and describe exact operation support in OpenAPI.
5. **Pin Features Part 4.** Keep the four CSAPI transaction classes, mark their inherited dependency as draft, and require a publication delta gate.
6. **Preserve exact canonical identifiers.** Store literal `http` class URIs and separate them from documentation anchors and resolver health.
7. **Do not seed declarations from the official example OAS.** Build the allowlist from the normative class inventories and verify OAS against it.
8. **Treat recommendations honestly.** Best-of-breed Glaux should generally meet them, but an unmet recommendation is not an OGC class failure unless Glaux separately profiles it as mandatory.
9. **Build stronger evidence than Annex A.** Execute the 240 tests, documented adapters, negative/lifecycle/schema/interoperability tests, and inherited official suites.
10. **Keep certification language separate.** Self-tested conformance and formal OGC certification are different claims.

### 14.3 What Acceptance Decides

Acceptance approves:

- all 25 direct classes as the intended final Glaux Server profile;
- the exact requirement-to-class mapping and prerequisite interpretation;
- evidence-gated, build-specific runtime declarations;
- the conservative encoding/write policy pending upstream resolution; and
- the downstream evidence and handoff model.

Acceptance does not decide exact endpoint implementation, Rust architecture, database, milestone dates, security profile, final harness tooling, or formal certification.

---

## 15. Risks, Constraints, and Open Questions

### 15.1 Risk Register

| Risk | Effect | Control / Owner |
|---|---|---|
| Blanket all-class declaration before proof | False conformance and lost credibility | Build-specific claim gate; IDR-009/050/051 |
| Annex prerequisite defects copied into code | Missing inherited obligations | Curated dependency registry and named adapters; IDR-050 |
| Features Part 4 changes on publication | Transaction behavior/test drift | Exact pin, qualification, delta gate; IDR-010A/029/050/057 |
| Encoding/CRUD ambiguity #23 | Unsupported operation combinations hidden by broad class claim | Conservative full matrix and OpenAPI evidence; IDR-009/012/014/050 |
| Part 2 PURL 404s mistaken for invalid identifiers | Wrong declaration strings or omitted classes | Literal identifier registry plus separate health monitor; IDR-009 |
| Informative OAS example copied as truth | Stale/nonexistent classes advertised | Normative allowlist and artifact regression; IDR-014 |
| Literal ATS treated as complete | Defects, vacuous passes, weak negative/lifecycle proof | Adapter ledger and supplemental evidence; IDR-050/051/053–056 |
| Full target mistaken for simultaneous implementation | Planning paralysis or hidden scope cuts | Dependency waves with unchanged end state; roadmap topic |
| Recommendation upgraded to requirement | False class failure | Severity field and #150 regression; IDR-050/051 |

### 15.2 Unresolved Issues and Disposition

| Issue | Current Defensible Position | Blocks This Report? | Later Owner / Decision |
|---|---|---|---|
| Exact encoding/read/write class model under #23 | Published classes control; full Glaux target satisfies all triggered combinations; narrower release withholds conflicting claim | No | Monitor upstream; IDR-009/012/014/050 |
| Features Part 4 is draft | Pin `9ca25f56...`, qualify inherited evidence, retest on publication | No; material implementation risk | IDR-010A/029/050/057 |
| Part 2 and recommendation PURLs do not resolve | Emit exact standard identifiers; link humans to approved HTML; monitor separately | No | OGC #152; IDR-009/057 |
| Annex prerequisite/test contradictions | Preserve normative obligation and named adapter; never silently edit | No | IDR-050/051 |
| No public official CSAPI ETS | Build traceable Glaux harness; do not claim official certification from internal tests | No | IDR-050/056 |
| Exact release milestones | Use dependency waves without reducing end state | No | Later implementation roadmap after IDR synthesis |

There is no unresolved mapping that prevents acceptance of the 25-class end-state profile. The project lead's only present decision is whether to accept this baseline and its claim policy. All other open matters have explicit later owners.

---

## 16. Validation Against the Research Plan

### 16.1 Methodology-Phase Validation

| Phase | Status | Evidence |
|---|---|---|
| 1. Source collection and framework | Complete | §§3–4; Appendix C |
| 2. Conformance class inventory | Complete | §§5–7 |
| 3. Requirement-to-conformance mapping | Complete | §8; Appendices A–B |
| 4. Applicability and posture | Complete | §9 |
| 5. Test/verification implications | Complete | §§11–13 |
| 6. Synthesis | Complete | §§1, 10, 14–15, 18 |

### 16.2 Success-Criterion Validation

| Plan Success Criterion | Status | Evidence |
|---|---|---|
| Part 1 and Part 2 classes identified with source anchors | Met | §§5–6 |
| Requirement/conformance relationships documented | Met | §§4.3, 5–8; Appendices A–B |
| Declaration identifiers identified | Met | §§4.3, 11.3 |
| Part 1 and Part 2 requirements mapped | Met: all 233 | §8; Appendices A–B |
| Inherited OGC behavior identified | Met | §7, §11.1 |
| Every class classified for Glaux | Met: 25/25 | §9.1 |
| Dependencies and claim evidence identified | Met | §§7, 9, 11–12 |
| Official history date-checked and authority-classified | Met | §10; register Version 1.1 |
| Downstream behavior/validation/testing handoffs documented | Met | §§12–13 |
| Unresolved issues listed | Met | §15.2 |
| References explicit and reproducible | Met | §§3, 17; Appendix C |

### 16.3 Deliverable-Content Validation

| Required Content | Status | Location |
|---|---|---|
| 1. Executive summary | Met | §1 |
| 2. Scope and plan alignment | Met | §2 |
| 3. Evidence base/authority | Met | §3 |
| 4. Mapping methodology | Met | §4 |
| 5. Part 1 inventory | Met | §5 |
| 6. Part 2 inventory | Met | §6 |
| 7. Inherited inventory | Met | §7 |
| 8. Requirement mapping | Met | §8; Appendices A–B |
| 9. Glaux posture | Met | §9 |
| 10. Upstream context | Met | §10 |
| 11. Declaration implications | Met | §11 |
| 12. Test implications | Met | §12 |
| 13. Downstream handoffs | Met | §13 |
| 14. Recommendations | Met | §14 |
| 15. Risks/open questions | Met | §15 |
| 16. Success-criteria validation | Met | §16 |
| 17. References | Met | §17 |

All sixteen minimum mapping fields enumerated by the plan are present through the class-code join across §§5–9: standard/source, requirement class, conformance class, full identifier construction, requirement ID/source, summary, standards status, Glaux applicability, artifact, upstream record/authority, release relationship, implementation dependency, claim evidence, test implication, downstream handoff, and notes/unresolved issues.

### 16.4 Independent Review and Reconciliation

Three independent read-only tracks reviewed:

1. Part 1 classes, exact requirement membership, prerequisites, Annex discrepancies, PURLs, and claim evidence.
2. Part 2 classes, all 130 requirement/test identifier pairs, conditions, prerequisites, Annex defects, and test implications.
3. Bounded official issue/PR/release history, NamingAuthority/PURL state, tagged OAS declaration artifacts, and official ETS availability.

Their independently derived totals reconcile with the accepted requirement baselines: 25 classes, 233 numbered requirements, five recommendations, and 240 abstract tests. The main synthesis checked the pinned sources, current issue/PR states, Features Part 4 commit, NamingAuthority tree, and public OGC ETS search. No review track edited repository files.

Two additional independent read-only report audits then checked technical correctness and governance completeness. Their findings added the missing Common/WKT source entries and explicit streaming boundary; corrected the formal Part 2 dependency scope; added the Text/Binary cross-reference to SWE JSON mapping requirements; and made the Part 1 staged-claim condition interpretations explicit in §8.5. Both reviewers rechecked the corrected report and returned `PASS` with no remaining critical or major defect.

---

## 17. References

### 17.1 Controlling and Inherited Standards

- [OGC 23-001, OGC API - Connected Systems - Part 1: Feature Resources](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC 23-002, OGC API - Connected Systems - Part 2: Dynamic Data](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC 19-072, OGC API - Common - Part 1: Core](https://docs.ogc.org/is/19-072/19-072.html)
- [OGC 17-069r4, OGC API - Features - Part 1: Core](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC 06-103r4, Simple Feature Access - Part 1: Common Architecture](https://portal.ogc.org/files/?artifact_id=25355)
- [OGC API - Features Part 4 draft, OGC 20-002r1](https://docs.ogc.org/DRAFTS/20-002r1.html)
- [OGC 23-000, SensorML Encoding Standard 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC 24-014, SWE Common Data Model Encoding Standard 3.0](https://docs.ogc.org/is/24-014/24-014.html)

### 17.2 Official Artifacts and Maintenance Evidence

- [Official CSAPI Version 1.0.0 tagged API source](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api)
- [Official CSAPI Version 1.0.0 release](https://github.com/opengeospatial/ogcapi-connected-systems/releases/tag/v1.0.0)
- [Tagged conformance declaration example](https://github.com/opengeospatial/ogcapi-connected-systems/blob/v1.0.0/api/part1/openapi/examples/confClasses.json)
- [OGC NamingAuthority pinned mapping tree](https://github.com/opengeospatial/NamingAuthority/tree/63af09213dbce1e940333e82cd66bfe5859630ee/specification-elements/mappings)
- [OGC API - Features pinned draft-development commit](https://github.com/opengeospatial/ogcapi-features/tree/9ca25f56a58ed822ea8a685a7a41afa7181aaa8b)
- [Official OGC API - Features 1.0 ETS](https://github.com/opengeospatial/ets-ogcapi-features10)
- [Issue #23: encoding and CRUD conformance combinations](https://github.com/opengeospatial/ogcapi-connected-systems/issues/23)
- [Issue #28](https://github.com/opengeospatial/ogcapi-connected-systems/issues/28) and [PR #29](https://github.com/opengeospatial/ogcapi-connected-systems/pull/29): identifier relationships
- [Issue #82](https://github.com/opengeospatial/ogcapi-connected-systems/issues/82) and [PR #96](https://github.com/opengeospatial/ogcapi-connected-systems/pull/96): Sampling Feature boundary
- [Issue #141: Features Part 4 draft dependency](https://github.com/opengeospatial/ogcapi-connected-systems/issues/141)
- [Issue #150: System location recommendation](https://github.com/opengeospatial/ogcapi-connected-systems/issues/150)
- [Issue #152: Part 2 publication/PURL work](https://github.com/opengeospatial/ogcapi-connected-systems/issues/152)

### 17.3 Project, Governance, and Prior Research

- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-008 research plan](../IDR%20Plans/idr-srv-008-conformance-class-and-requirement-mapping.md)
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research Planning Approach](../../../../../Governance/research-planning-approach.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
- [OGC Connected Systems Upstream-History Register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)
- [IDR-SRV-003 AEP-4789 Volume II baseline report](idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md)
- [IDR-SRV-006 Part 1 requirement baseline report](idr-srv-006-csapi-part-1-requirement-baseline-report.md)
- [IDR-SRV-007 Part 2 requirement baseline report](idr-srv-007-csapi-part-2-requirement-baseline-report.md)

---

## 18. Next Steps and Handoff

### 18.1 Current Status

Research execution, drafting, traceability checks, and independent technical review are complete. The report is **In Review**. Plan-owner acceptance fields remain pending by design. No work on IDR-SRV-009 has begun.

### 18.2 Required Two-Action Transition

The next two actions are:

1. The Glaux Project Lead reviews and approves IDR-SRV-008, which changes this report to `Final` and the topic plan to `Complete`.
2. In the same instruction, the project lead authorizes execution of exactly one next topic: **IDR-SRV-009, Landing Page, API Definition, and Conformance Declaration Behavior**.

Combined response pattern:

> Approve IDR-SRV-008 and execute exactly one Glaux Server research plan: IDR-SRV-009.

That combined instruction accepts this report and starts only the next plan. It does not authorize any later topic.

---

## 19. Appendices

### Appendix A - Every Part 1 Direct Identifier Grouped by Class

The full URI is `CS1` plus the displayed identifier. Replace `/req/` with `/conf/` and retain the suffix to obtain the nominal requirement-test URI. Recommendation IDs use `/rec/`; their test uses the same item under `/conf/`.

- **P1-01, Requirements 1–3:** `/req/api-common/resource-ids`; `/req/api-common/resource-uids`; `/req/api-common/datetime`. **Recommendation 1:** `/rec/api-common/resource-uids-types`.
- **P1-02, Requirements 4–8:** `/req/system/location-time`; `/req/system/canonical-url`; `/req/system/resources-endpoint`; `/req/system/canonical-endpoint`; `/req/system/collections`. **Recommendation 2:** `/rec/system/location`.
- **P1-03, Requirements 9–13:** `/req/subsystem/collection`; `/req/subsystem/recursive-param`; `/req/subsystem/recursive-search-systems`; `/req/subsystem/recursive-search-subsystems`; `/req/subsystem/recursive-assoc`.
- **P1-04, Requirements 14–18:** `/req/deployment/canonical-url`; `/req/deployment/resources-endpoint`; `/req/deployment/canonical-endpoint`; `/req/deployment/ref-from-system`; `/req/deployment/collections`.
- **P1-05, Requirements 19–23:** `/req/subdeployment/collection`; `/req/subdeployment/recursive-param`; `/req/subdeployment/recursive-search-deployments`; `/req/subdeployment/recursive-search-subdeployments`; `/req/subdeployment/recursive-assoc`.
- **P1-06, Requirements 24–28:** `/req/procedure/location`; `/req/procedure/canonical-url`; `/req/procedure/resources-endpoint`; `/req/procedure/canonical-endpoint`; `/req/procedure/collections`.
- **P1-07, Requirements 29–33:** `/req/sf/canonical-url`; `/req/sf/resources-endpoint`; `/req/sf/canonical-endpoint`; `/req/sf/ref-from-system`; `/req/sf/collections`.
- **P1-08, Requirements 34–37:** `/req/property/canonical-url`; `/req/property/resources-endpoint`; `/req/property/canonical-endpoint`; `/req/property/collections`.
- **P1-09, Requirements 38–59:** `/req/advanced-filtering/id-list-schema`; `/req/advanced-filtering/resource-by-id`; `/req/advanced-filtering/resource-by-keyword`; `/req/advanced-filtering/feature-by-geom`; `/req/advanced-filtering/system-by-parent`; `/req/advanced-filtering/system-by-procedure`; `/req/advanced-filtering/system-by-foi`; `/req/advanced-filtering/system-by-obsprop`; `/req/advanced-filtering/system-by-controlprop`; `/req/advanced-filtering/procedure-by-obsprop`; `/req/advanced-filtering/procedure-by-controlprop`; `/req/advanced-filtering/deployment-by-parent`; `/req/advanced-filtering/deployment-by-system`; `/req/advanced-filtering/deployment-by-foi`; `/req/advanced-filtering/deployment-by-obsprop`; `/req/advanced-filtering/deployment-by-controlprop`; `/req/advanced-filtering/sf-by-foi`; `/req/advanced-filtering/sf-by-obsprop`; `/req/advanced-filtering/sf-by-controlprop`; `/req/advanced-filtering/prop-by-baseprop`; `/req/advanced-filtering/prop-by-object`; `/req/advanced-filtering/combined-filters`. **Recommendations 3–5:** `/rec/advanced-filtering/resource-by-property`; `/rec/advanced-filtering/indirect-prop`; `/rec/advanced-filtering/indirect-foi`.
- **P1-10, Requirements 60–71:** `/req/create-replace-delete/system`; `/req/create-replace-delete/system-delete-cascade`; `/req/create-replace-delete/subsystem`; `/req/create-replace-delete/deployment`; `/req/create-replace-delete/subdeployment`; `/req/create-replace-delete/procedure`; `/req/create-replace-delete/sampling-feature`; `/req/create-replace-delete/property`; `/req/create-replace-delete/create-in-collection`; `/req/create-replace-delete/replace-in-collection`; `/req/create-replace-delete/delete-in-collection`; `/req/create-replace-delete/add-to-collection`.
- **P1-11, Requirements 72–76:** `/req/update/system`; `/req/update/deployment`; `/req/update/procedure`; `/req/update/sampling-feature`; `/req/update/property`.
- **P1-12, Requirements 77–88:** `/req/geojson/mediatype-read`; `/req/geojson/mediatype-write`; `/req/geojson/relation-types`; `/req/geojson/feature-attribute-mapping`; `/req/geojson/system-schema`; `/req/geojson/system-mappings`; `/req/geojson/deployment-schema`; `/req/geojson/deployment-mappings`; `/req/geojson/procedure-schema`; `/req/geojson/procedure-mappings`; `/req/geojson/sf-schema`; `/req/geojson/sf-mappings`.
- **P1-13, Requirements 89–103:** `/req/sensorml/mediatype-read`; `/req/sensorml/mediatype-write`; `/req/sensorml/relation-types`; `/req/sensorml/resource-id`; `/req/sensorml/feature-attribute-mapping`; `/req/sensorml/system-schema`; `/req/sensorml/system-sml-class`; `/req/sensorml/system-mappings`; `/req/sensorml/deployment-schema`; `/req/sensorml/deployment-mappings`; `/req/sensorml/procedure-schema`; `/req/sensorml/procedure-sml-class`; `/req/sensorml/procedure-mappings`; `/req/sensorml/property-schema`; `/req/sensorml/property-mappings`.

Part 1 additionally defines supporting tests `/conf/api-common/canonical-resources` and `/conf/api-common/collection-items`; they do not target numbered requirements.

### Appendix B - Every Part 2 Direct Identifier Grouped by Class

The full URI is `CS2` plus the displayed identifier. For every entry, replace `/req/` with `/conf/` and retain the suffix to obtain its nominal abstract-test URI.

- **P2-01, Requirements 1–2:** `/req/api-common/resources`; `/req/api-common/resource-collection`.
- **P2-02, Requirements 3–16:** `/req/datastream/sf-ref-from-datastream`; `/req/datastream/foi-ref-from-datastream`; `/req/datastream/canonical-url`; `/req/datastream/resources-endpoint`; `/req/datastream/canonical-endpoint`; `/req/datastream/ref-from-system`; `/req/datastream/ref-from-deployment`; `/req/datastream/collections`; `/req/datastream/schema-op`; `/req/datastream/obs-canonical-url`; `/req/datastream/obs-resources-endpoint`; `/req/datastream/obs-canonical-endpoint`; `/req/datastream/obs-ref-from-datastream`; `/req/datastream/obs-collections`.
- **P2-03, Requirements 17–34:** `/req/controlstream/sf-ref-from-controlstream`; `/req/controlstream/foi-ref-from-controlstream`; `/req/controlstream/canonical-url`; `/req/controlstream/resources-endpoint`; `/req/controlstream/canonical-endpoint`; `/req/controlstream/ref-from-system`; `/req/controlstream/ref-from-deployment`; `/req/controlstream/collections`; `/req/controlstream/schema-op`; `/req/controlstream/cmd-canonical-url`; `/req/controlstream/cmd-resources-endpoint`; `/req/controlstream/cmd-canonical-endpoint`; `/req/controlstream/cmd-ref-from-controlstream`; `/req/controlstream/cmd-collections`; `/req/controlstream/status-resources-endpoint`; `/req/controlstream/command-status-endpoint`; `/req/controlstream/result-resources-endpoint`; `/req/controlstream/command-result-endpoint`.
- **P2-04, Requirements 35–39:** `/req/feasibility/canonical-url`; `/req/feasibility/ref-from-controlstream`; `/req/feasibility/status-endpoint`; `/req/feasibility/result-endpoint`; `/req/feasibility/collections`.
- **P2-05, Requirements 40–44:** `/req/system-event/canonical-url`; `/req/system-event/resources-endpoint`; `/req/system-event/canonical-endpoint`; `/req/system-event/ref-from-system`; `/req/system-event/collections`.
- **P2-06, Requirements 45–62:** `/req/advanced-filtering/datastream-by-phenomenontime`; `/req/advanced-filtering/datastream-by-resulttime`; `/req/advanced-filtering/datastream-by-obsprop`; `/req/advanced-filtering/datastream-by-foi`; `/req/advanced-filtering/obs-by-phenomenontime`; `/req/advanced-filtering/obs-by-resulttime`; `/req/advanced-filtering/obs-by-foi`; `/req/advanced-filtering/controlstream-by-issuetime`; `/req/advanced-filtering/controlstream-by-exectime`; `/req/advanced-filtering/controlstream-by-controlprop`; `/req/advanced-filtering/controlstream-by-foi`; `/req/advanced-filtering/cmd-by-issuetime`; `/req/advanced-filtering/cmd-by-exectime`; `/req/advanced-filtering/cmd-by-status`; `/req/advanced-filtering/cmd-by-sender`; `/req/advanced-filtering/cmd-by-foi`; `/req/advanced-filtering/status-by-statuscode`; `/req/advanced-filtering/event-by-type`.
- **P2-07, Requirements 63–78:** `/req/create-replace-delete/datastream`; `/req/create-replace-delete/datastream-update-schema`; `/req/create-replace-delete/datastream-delete-cascade`; `/req/create-replace-delete/observation`; `/req/create-replace-delete/observation-schema`; `/req/create-replace-delete/controlstream`; `/req/create-replace-delete/controlstream-update-schema`; `/req/create-replace-delete/controlstream-delete-cascade`; `/req/create-replace-delete/command`; `/req/create-replace-delete/command-schema`; `/req/create-replace-delete/command-status`; `/req/create-replace-delete/command-result`; `/req/create-replace-delete/feasibility`; `/req/create-replace-delete/feasibility-status`; `/req/create-replace-delete/feasibility-result`; `/req/create-replace-delete/system-event`.
- **P2-08, Requirements 79–92:** `/req/update/datastream`; `/req/update/datastream-update-schema`; `/req/update/observation`; `/req/update/observation-schema`; `/req/update/controlstream`; `/req/update/controlstream-update-schema`; `/req/update/command`; `/req/update/command-schema`; `/req/update/command-status`; `/req/update/command-result`; `/req/update/feasibility`; `/req/update/feasibility-status`; `/req/update/feasibility-result`; `/req/update/system-event`.
- **P2-09, Requirements 93–106:** `/req/json/mediatype-read`; `/req/json/mediatype-write`; `/req/json/datastream-schema`; `/req/json/obsschema-schema`; `/req/json/observation-schema`; `/req/json/observation-constraints`; `/req/json/controlstream-schema`; `/req/json/commandschema-schema`; `/req/json/command-schema`; `/req/json/command-constraints`; `/req/json/commandstatus-schema`; `/req/json/commandresult-schema`; `/req/json/commandresult-constraints`; `/req/json/systemevent-schema`.
- **P2-10, Requirements 107–114:** `/req/swecommon-json/mediatype-read`; `/req/swecommon-json/mediatype-write`; `/req/swecommon-json/obsschema-schema`; `/req/swecommon-json/obsschema-mapping`; `/req/swecommon-json/observation-encoding`; `/req/swecommon-json/cmdschema-schema`; `/req/swecommon-json/cmdschema-mapping`; `/req/swecommon-json/command-encoding`.
- **P2-11, Requirements 115–122:** `/req/swecommon-text/mediatype-read`; `/req/swecommon-text/mediatype-write`; `/req/swecommon-text/obsschema-schema`; `/req/swecommon-text/obsschema-mapping`; `/req/swecommon-text/observation-encoding`; `/req/swecommon-text/cmdschema-schema`; `/req/swecommon-text/cmdschema-mapping`; `/req/swecommon-text/command-encoding`.
- **P2-12, Requirements 123–130:** `/req/swecommon-binary/mediatype-read`; `/req/swecommon-binary/mediatype-write`; `/req/swecommon-binary/obsschema-schema`; `/req/swecommon-binary/obsschema-mapping`; `/req/swecommon-binary/observation-encoding`; `/req/swecommon-binary/cmdschema-schema`; `/req/swecommon-binary/cmdschema-mapping`; `/req/swecommon-binary/command-encoding`.

### Appendix C - Count and Reproducibility Record

| Check | Result |
|---|---|
| Part 1 class metadata | 13 requirement classes and 13 conformance classes |
| Part 1 direct content | 103 numbered requirements, 5 recommendations |
| Part 1 ATS | 110 tests: 103 requirement, 5 recommendation, 2 supporting |
| Part 2 class metadata | 12 requirement classes and 12 conformance classes |
| Part 2 direct content and ATS | 130 requirements and 130 same-suffix tests; zero missing/extra/mistargeted pairs |
| Direct combined baseline | 25 class pairs, 233 requirements, 5 recommendations, 240 tests |
| Tagged source | `v1.0.0` / `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2` |
| Features Part 4 research pin | `9ca25f56a58ed822ea8a685a7a41afa7181aaa8b`, checked 2026-08-01 |
| NamingAuthority pin | `63af09213dbce1e940333e82cd66bfe5859630ee`, checked 2026-08-01 |
| Upstream states | #23/#141/#150/#152 open; #28/#82 closed; PR #29/#96 merged and in tag |
| Class PURLs | Part 1 26/26 class PURLs resolve; Part 2 0/24 resolve at check time |
| Public official CSAPI ETS search | None found in OGC organization or hosted suite lists |

### Appendix D - Every Research Question Mapped

| ID | Plan Question (Short Form) | Status | Evidence |
|---|---|---|---|
| C1 | Which classes apply to Glaux? | Complete | §§5–6, 9 |
| C2 | Which requirements map to each class/URI? | Complete | §8, Appendices A–B |
| C3 | Which are mandatory, optional, conditional, inherited, or choice-dependent? | Complete | §§4.4, 5–9 |
| C4 | Which claims support, defer, or evaluate later? | Complete | §§9, 11, 14 |
| C5 | What traceability model should downstream use? | Complete | §§4.2, 12.4–12.5 |
| CI-1 | Part 1 classes | Complete | §5 |
| CI-2 | Part 2 classes | Complete | §6 |
| CI-3 | Associated requirement classes | Complete | §§5–6 |
| CI-4 | Inherited Features/Common behavior | Complete | §7 |
| CI-5 | Encoding/OAS/schema/optional dependencies | Complete | §§5–8, 10–12 |
| CI-6 | Declaration URIs | Complete | §§4.3, 11.3 |
| RM-1 | Part 1 requirement mapping | Complete | §8.2, Appendix A |
| RM-2 | Part 2 requirement mapping | Complete | §8.3, Appendix B |
| RM-3 | Shared/inherited/cross-referenced requirements | Complete | §7 |
| RM-4 | Requirements without obvious mapping | Complete: none; interpretation issues retained | §8.4 |
| RM-5 | SensorML/SWE/schema/OAS/profile dependencies | Complete | §§7–8 |
| UH-1 | Material official issue/PR/release history | Complete | §10 |
| UH-2 | Published/post-release/discussion/unresolved classification | Complete | §10 |
| UH-3 | Unresolved compatibility/interpretation risks | Complete | §§10, 15 |
| AG-1 | Necessary full-scope classes | Complete: all 25 | §9 |
| AG-2 | Optional but strategically important classes | Complete: filtering/transaction classes selected | §§9, 14 |
| AG-3 | Classes requiring future decisions | Complete: transaction qualification and #23 | §§9, 15 |
| AG-4 | Classes deferrable without reducing full scope | Complete: sequencing allowed; none removed | §9.3 |
| AG-5 | Classes outside server boundary | Complete: no direct CSAPI class | §§9.2, 15.2 |
| CD-1 | Required conformance information | Complete | §11.1 |
| CD-2 | URIs Glaux should declare | Complete | §11.3 |
| CD-3 | Evidence before declaration | Complete | §11.2 |
| CD-4 | Partial/staged/experimental/deferred handling | Complete | §§9.2, 11.2 |
| CD-5 | Later declaration topics | Complete | §§11, 13 |
| VT-1 | Positive/negative tests per class | Complete | §§9.1, 12.2 |
| VT-2 | Schema/OAS/model/dynamic/tasking/stream/interoperability tests | Complete | §12.2 |
| VT-3 | Fixture/golden-file needs | Complete | §12.2 |
| VT-4 | External-client validation | Complete | §12.2 |
| VT-5 | IDR-050/051 handoff | Complete | §12.5 |
| IS-1 | Foundational class dependencies | Complete | §§7, 9.3 |
| IS-2 | Resource-model dependencies | Complete | §§8–9, 13 |
| IS-3 | Persistence/time-series/streaming/tasking/security dependencies | Complete | §§9.1, 13 |
| IS-4 | Independently researchable/designable/testable classes | Complete | §9.3 |
| IS-5 | Roadmap dependencies without scope reduction | Complete | §9.3 |

### Appendix E - Report Completion Checklist

- [x] Topic ID and plan link are correct
- [x] All 5 core and 34 detailed research questions are covered
- [x] All six methodology phases are complete
- [x] All eleven success criteria are met
- [x] All seventeen deliverable-content requirements are present
- [x] Every one of the plan's enumerated minimum mapping fields is represented
- [x] All 25 direct classes and all 233 numbered requirements are mapped
- [x] Standards obligations, findings, analysis, and recommendations are distinguished
- [x] Mutable sources identify version, tag, commit, state, and/or access date
- [x] OAS definitions and bounded open/closed official issue history were reviewed where relevant
- [x] Evidence limitations and unresolved issues are explicit
- [x] Prior accepted reports are reconciled
- [x] Independent read-only technical reviews are reconciled
- [x] Plan-owner acceptance fields remain pending
- [x] No other research topic was begun

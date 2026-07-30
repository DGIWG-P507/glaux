# Section 003: AEP-4789 Volume II Standards Package Implementation Baseline - Research Plan

**Status:** Planned  
**Last Updated:** July 29, 2026\
**Estimated Research Time:** 10-14 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md`

---

## Usage Instructions

Before executing this plan, review the full exemplar corpus and align the report to the proven standards for content nature, type, level, and detail:

- Full research-plans corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- Early exemplar (blueprint-first depth):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md
- Mid-stream exemplar (inventory + sourcing rigor):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/15-fixture-sourcing-organization.md
- End-state synthesis exemplar:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/38-testing-playbook-synthesis.md

This plan is intentionally limited to research planning. It does not execute the research and does not produce the research report.

---

## 1. Research Objective

This topic research must establish the **AEP-4789 Volume II standards-package implementation baseline** for Glaux Server.

The research must determine how the Volume II adopted standards package applies to a server-side Glaux implementation and how the four core standards should be treated as a coherent technical package rather than independent references:

- OGC API - Connected Systems Part 1: Feature Resources
- OGC API - Connected Systems Part 2: Dynamic Data
- OGC SensorML Encoding Standard
- OGC SWE Common Data Model Encoding Standard

The research must answer:

- What does AEP-4789 Volume II adopt, require, recommend, or imply for Glaux Server?
- How do CSAPI Part 1, CSAPI Part 2, SensorML, and SWE Common divide responsibilities across metadata, feature resources, observations, dynamic data, tasking, status, events, system descriptions, data components, and encodings?
- Which standards-package obligations should become direct server implementation concerns, which should become validation/conformance concerns, and which require later detailed extraction in downstream topics?
- What implementation boundaries must Glaux Server preserve so it remains standards-aligned without inventing project-specific alternatives to adopted OGC standards?

The output must be a standards-package implementation baseline that later IDR topics can use to research CSAPI requirements, resource models, SensorML and SWE representation, schema validation, persistence, dynamic data, tasking, and test strategy.

### Why This Topic Order

This topic follows:

- `IDR-SRV-001: STANAG 4789 / AEP-4789 Server Obligation Baseline`
- `IDR-SRV-002: AEP-4789 Volume I Functional Mapping to Server Responsibilities`

Those topics establish the obligation and functional context. This topic identifies the technical standards package adopted by AEP-4789 Volume II and determines how that package should shape Glaux Server implementation research.

This topic must precede detailed CSAPI, SensorML, SWE Common, resource-model, validation, persistence, dynamic-data, tasking, and conformance research because those later topics depend on knowing how the standards package is intended to work together.

### Critical Constraint(s)

- Treat AEP-4789 Volume II as the controlling adoption and implementation-context source for this topic.
- Treat the official OGC standards and schemas as authoritative for technical requirements, conformance classes, resource behavior, data models, and encodings.
- Do not perform full clause-by-clause CSAPI Part 1 or Part 2 requirement extraction here; that belongs to `IDR-SRV-006` and `IDR-SRV-007`.
- Do not perform full SensorML or SWE Common representation design here; that belongs to `IDR-SRV-021` and `IDR-SRV-022`.
- Do not design the Glaux Server database, API implementation, Rust framework, or test harness in this topic.
- Clearly distinguish:
  - Volume II adoption language,
  - direct OGC standards obligations,
  - server implementation implications,
  - downstream extraction topics,
  - unresolved interpretation questions.
- Keep conclusions bounded to Glaux Server responsibilities and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What standards does AEP-4789 Volume II adopt as the core APIs and encodings package for connected-system interoperability?
2. How does AEP-4789 Volume II describe or imply the relationship among CSAPI Part 1, CSAPI Part 2, SensorML, and SWE Common?
3. Which parts of the standards package create direct Glaux Server implementation obligations?
4. Which standards-package areas require later detailed research in downstream IDR topics?
5. What traceability model should Glaux Server use to connect AEP-4789 Volume II adoption language to OGC requirements, schemas, conformance classes, validation rules, and test strategy?

### Detailed Questions

#### Volume II Adoption and Scope

- What does AEP-4789 Volume II identify as the adopted technical standards package?
- Does Volume II distinguish required, recommended, optional, informative, or future implementation areas?
- How does Volume II describe the role of CSAPI Part 1?
- How does Volume II describe the role of CSAPI Part 2?
- How does Volume II describe the role of SensorML?
- How does Volume II describe the role of SWE Common?
- Are there Volume II caveats, assumptions, limitations, or future-volume references that affect Glaux Server?

#### Coherent Standards-Package Model

- How should the four standards be understood as one implementation package for Glaux Server?
- Which standard primarily governs connected-system feature resources and metadata?
- Which standard primarily governs dynamic data, observations, status, events, commands, control streams, and tasking?
- Which standard primarily governs system/process descriptions?
- Which standard primarily governs data components, command parameters, observation structures, units, constraints, and encodings?
- Where do responsibilities overlap, and what later research must resolve those overlaps?

#### Server Implementation Implications

- Which standards-package areas must Glaux Server expose through API behavior?
- Which areas must Glaux Server validate?
- Which areas must Glaux Server store, link, or generate?
- Which areas must Glaux Server accept from Publisher, Simulator, or external clients?
- Which areas require machine-readable API documentation, schemas, examples, or conformance declarations?
- Which areas affect Rust implementation planning, test strategy, or CI quality gates?

#### Conformance and Validation Implications

- What conformance classes, requirements, schemas, or validation resources are referenced directly or indirectly by the standards package?
- What later conformance research must be performed in `IDR-SRV-008` and `IDR-SRV-050`?
- Which standards-package elements imply schema validation in `IDR-SRV-023`?
- Which elements imply golden-file or fixture strategy in `IDR-SRV-053`?
- Which elements imply interoperability testing in `IDR-SRV-056`?

#### Boundary and Scope Control

- Which Volume II or OGC standards-package concerns are direct Glaux Server concerns?
- Which concerns belong to Publisher, Simulator, Web App, Mobile, external clients, or ecosystem-level guidance?
- Which concerns are profile-level, future SRD/AEP-level, or out of scope for current Glaux Server planning?
- What language should downstream topics use to avoid treating the standards package either too narrowly or too broadly?

---

## 3. Primary Resources

The future research report must analyze these sources directly.

### Project and Governance Sources

- Glaux Server Overall IDR Research Plan:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md

### Prior IDR Topic Sources

- `IDR-SRV-001` research plan:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline.md`
- `IDR-SRV-001` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md`
- `IDR-SRV-002` research plan:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities.md`
- `IDR-SRV-002` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md`

### AEP / STANAG Source Material

- Project-controlling ratification package: `AC/224(JCGISR)D(2026)0005`, dated 27 April 2026.
  - Status for this IDR: most-current ratification draft, as confirmed by the Glaux project lead on 29 July 2026.
  - SHA-256: `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`.
  - Use AEP-4789 Volume II, Edition A, Version 1 as the controlling source for this topic.
  - Use STANAG 4789, Edition 1 and AEP-4789 Volume I, Edition A, Version 1 as parent and functional context.
  - The local working copy is supplied by the project lead and is not stored in the public repository.
  - Cite the enclosing document, enclosed publication, and exact page/section in the report.

### OGC Standards Package Sources

- OGC API - Connected Systems landing page:
  - https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources:
  - https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data:
  - https://docs.ogc.org/is/23-002/23-002.html
- OGC SensorML Encoding Standard 3.0:
  - https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0:
  - https://docs.ogc.org/is/24-014/24-014.html
- OGC API - Connected Systems public development repository:
  - https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems schemas and OpenAPI artifacts:
  - https://schemas.opengis.net/
  - https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC API standards family landing page:
  - https://ogcapi.ogc.org/
- OGC API - Features landing page, for inherited API behavior context:
  - https://ogcapi.ogc.org/features/

---

## 4. Supporting Resources

Use these sources to interpret implementation context, downstream dependencies, expected research depth, and report style.

- DGIWG P5.07 Glaux main repository:
  - https://github.com/DGIWG-P507/glaux
- Glaux project website:
  - https://dgiwg-p507.github.io/glaux/
- DGIWG P5.07 GitHub organization:
  - https://github.com/DGIWG-P507
- Glaux Server repository, if available or created:
  - https://github.com/DGIWG-P507/glaux-server
- OS4CSAPI organization:
  - https://github.com/OS4CSAPI
- OS4CSAPI testing research-plan corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- Blueprint-first exemplar:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md
- Inventory/sourcing rigor exemplar:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/15-fixture-sourcing-organization.md
- End-state synthesis exemplar:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/38-testing-playbook-synthesis.md
- OS4CSAPI discussions, for background only:
  - https://github.com/orgs/OS4CSAPI/discussions
- OS4CSAPI client work, for later interoperability context only:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability findings repository, for later interoperability context only:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd

---

## 5. Research Methodology

### Phase 1: Source Collection and Volume II Orientation (1.5-2 hours)

**Objective:** Establish the source base and understand how AEP-4789 Volume II frames the adopted standards package.

**Tasks:**

1. Gather the current AEP-4789 Volume II source material available to the project team.
2. Record title, version, date, status, source location, and access limitations.
3. Review the Volume II table of contents, standards adoption language, references, figures, tables, and any implementation guidance.
4. Gather the official OGC standards and schema resources listed in this plan.
5. Review `IDR-SRV-001` and `IDR-SRV-002` outputs, if available, and identify obligation/function findings that this topic must carry forward.

**Expected Output:** Source inventory and Volume II orientation notes identifying the standards package, adoption language, and high-value sections for detailed review.

### Phase 2: Standards Package Adoption Extraction (2-3 hours)

**Objective:** Extract what AEP-4789 Volume II adopts, requires, recommends, or implies for Glaux Server.

**Tasks:**

1. Identify every Volume II reference to CSAPI Part 1, CSAPI Part 2, SensorML, SWE Common, schemas, conformance, encodings, implementation guidance, and supporting OGC materials.
2. Capture section, clause, figure, table, or reference anchors for each adoption statement.
3. Classify each extracted item as:
   - direct server implementation implication,
   - validation/conformance implication,
   - representation/encoding implication,
   - downstream research handoff,
   - ecosystem or out-of-scope implication.
4. Identify any Volume II caveats, gaps, future work, optional areas, or unresolved interpretation issues.
5. Note terms and concepts that should feed `IDR-SRV-004`.

**Expected Output:** Volume II standards-package extraction table with source anchors, classifications, and initial interpretations.

### Phase 3: Coherent Standards-Package Responsibility Mapping (2.5-3.5 hours)

**Objective:** Map how CSAPI Part 1, CSAPI Part 2, SensorML, and SWE Common divide responsibilities for Glaux Server.

**Tasks:**

1. Review the official OGC standards overviews and high-level structure for each adopted standard.
2. Identify the main server-relevant responsibility areas of each standard.
3. Map standards to Glaux Server capability areas:
   - discovery and navigation,
   - feature resources and metadata,
   - systems, deployments, procedures, sampling features, and properties,
   - datastreams and observations,
   - control streams, commands, command status, and feasibility,
   - system events, status, availability, and dynamic properties,
   - system and process description,
   - data components, encodings, units, constraints, and values.
4. Identify overlaps, dependencies, or handoffs among the standards.
5. Identify which downstream IDR topics must perform deeper extraction or design work.

**Expected Output:** Standards-to-capability responsibility matrix and downstream handoff notes.

### Phase 4: Conformance, Schema, and Validation Implication Review (1.5-2 hours)

**Objective:** Identify conformance, schema, and validation implications that later topics must research in depth.

**Tasks:**

1. Identify where the standards package refers to conformance classes, requirements classes, schemas, OpenAPI files, encodings, examples, or validation artifacts.
2. Record candidate sources for later requirement extraction and test strategy.
3. Identify likely validation layers:
   - OpenAPI/API contract validation,
   - JSON Schema validation,
   - CSAPI resource/schema validation,
   - SensorML validation,
   - SWE Common data component and encoding validation,
   - semantic/units/property validation.
4. Identify implications for test fixtures, golden files, conformance harnesses, and interoperability tests.
5. Avoid designing the final conformance harness; record handoffs to later topics.

**Expected Output:** Conformance/schema/validation implication inventory and handoff list.

### Phase 5: Server Boundary and Implementation Implication Synthesis (1.5-2 hours)

**Objective:** Translate the standards-package baseline into bounded Glaux Server implementation implications.

**Tasks:**

1. Classify standards-package implications as direct Glaux Server responsibilities, server-side contracts, downstream design topics, ecosystem concerns, or out-of-scope concerns.
2. Identify implications for Rust implementation planning without selecting a Rust framework or implementation architecture.
3. Identify implications for persistence and query topics without designing the database model.
4. Identify implications for security, tasking, DDIL, and verification topics without resolving those topics prematurely.
5. Identify risks caused by ambiguity, overlap, or mismatch between AEP-4789 Volume II language and OGC standards language.

**Expected Output:** Bounded server implementation implication summary and risk list.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert research output into a decision-usable standards-package implementation baseline.

**Tasks:**

1. Consolidate evidence, adoption statements, standards responsibility mappings, validation implications, and server-boundary findings.
2. Resolve conflicts and ambiguities where possible.
3. Produce findings grouped by adopted standard and cross-standard function.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] AEP-4789 Volume II, Edition A, Version 1 from the project-controlling package has been reviewed.
- [ ] Source documents used in the report are listed with title, version/date, URL/path, status, and authority classification.
- [ ] Volume II adoption language for CSAPI Part 1, CSAPI Part 2, SensorML, and SWE Common is extracted with source anchors.
- [ ] The relationship among the four adopted standards is explained as a coherent implementation package.
- [ ] Server-relevant responsibility areas are mapped to the appropriate standard or standards.
- [ ] Conformance, schema, OpenAPI, validation, and encoding implications are identified.
- [ ] Downstream handoffs to later IDR topics are identified.
- [ ] Boundaries are drawn between direct server responsibilities, server-side contracts, ecosystem concerns, and out-of-scope concerns.
- [ ] Unresolved standards-package interpretation risks are explicitly listed.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** AEP-4789 Volume II Standards Package Implementation Baseline Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. AEP-4789 Volume II adoption findings
5. Standards-package responsibility matrix
6. CSAPI Part 1 / CSAPI Part 2 / SensorML / SWE Common relationship analysis
7. Server implementation implication summary
8. Conformance, schema, and validation implications
9. Downstream topic handoff matrix
10. Recommendations
11. Risks, constraints, and open questions
12. Validation against this plan's success criteria
13. References

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` research report should be complete or explicitly marked unavailable/deferred.
- `IDR-SRV-002` research report should be complete or explicitly marked unavailable/deferred.
- The project-controlling `AC/224(JCGISR)D(2026)0005` package must be available to the researcher. If it is unavailable, this topic is blocked.
- The official OGC standards-package sources must be reachable. If a controlling OGC standard is unavailable, affected conclusions remain blocked.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-004: Terminology and Concept Crosswalk`
- `IDR-SRV-006: CSAPI Part 1 Requirement Baseline`
- `IDR-SRV-007: CSAPI Part 2 Requirement Baseline`
- `IDR-SRV-008: Conformance Class and Requirement Mapping`
- `IDR-SRV-014: OpenAPI Description and API Documentation Strategy`
- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-021: SensorML Representation Strategy`
- `IDR-SRV-022: SWE Common Data Component Strategy`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-024: Units, Observed Properties, and Semantic Binding Strategy`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
- Final overall IDR synthesis report

---

## 9. Research Status Checklist

Update this section as work progresses.

- [ ] Phase 1 complete
- [ ] Phase 2 complete
- [ ] Phase 3 complete
- [ ] Phase 4 complete
- [ ] Phase 5 complete
- [ ] Phase 6 synthesis complete
- [ ] Deliverable draft complete
- [ ] Deliverable reviewed
- [ ] Deliverable accepted

**Actual Research Time:** TBD until complete  
**Completion Date:** TBD until complete

---

## 10. Notes and Open Questions

- AEP-4789 Volume II may contain adoption language that is narrower than the full technical scope of the adopted OGC standards. The report must distinguish Volume II adoption scope from full standard capability.
- Official OGC standards and schemas remain authoritative for technical requirements even when AEP-4789 Volume II provides the NATO adoption context.
- This topic should not duplicate later detailed requirement extraction from CSAPI Part 1 or Part 2.
- This topic should not pre-design SensorML/SWE persistence, validation, or encoding behavior; it should identify where that work must occur later.
- Open question: Which exact AEP-4789 Volume II copy is authoritative for this IDR cycle?
- Open question: Does Volume II adopt all relevant conformance classes from the referenced standards or only a subset?
- Open question: Are any standards-package elements optional, future-facing, or profile-dependent for Glaux Server?
- Risk: Treating the standards as unrelated documents could cause inconsistent server design.
- Risk: Treating the standards package too broadly could cause Glaux Server to absorb ecosystem or profile-level responsibilities.
- Risk: Treating the standards package too narrowly could cause later topics to miss necessary SensorML, SWE Common, validation, or conformance obligations.

---

## References

- Glaux Server Overall IDR Research Plan:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- `IDR-SRV-001` research plan:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline.md`
- `IDR-SRV-002` research plan:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities.md`
- Glaux Server Goal and Definition:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- OGC API - Connected Systems landing page:
  - https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources:
  - https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data:
  - https://docs.ogc.org/is/23-002/23-002.html
- OGC SensorML Encoding Standard 3.0:
  - https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0:
  - https://docs.ogc.org/is/24-014/24-014.html
- OGC API - Connected Systems public development repository:
  - https://github.com/opengeospatial/ogcapi-connected-systems
- OGC schemas:
  - https://schemas.opengis.net/
- OGC API standards family:
  - https://ogcapi.ogc.org/
- OGC API - Features:
  - https://ogcapi.ogc.org/features/
- OS4CSAPI testing research-plan corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans

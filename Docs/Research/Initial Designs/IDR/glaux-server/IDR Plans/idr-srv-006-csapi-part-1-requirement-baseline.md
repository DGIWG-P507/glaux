# Section 006: CSAPI Part 1 Requirement Baseline - Research Plan

**Status:** Planned  
**Last Updated:** June 7, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-006-csapi-part-1-requirement-baseline-report.md`

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

This topic research must extract and organize the **OGC API - Connected Systems Part 1: Feature Resources** requirements that apply to Glaux Server.

The research must answer:

- What normative Part 1 requirements, requirement classes, conformance classes, resource patterns, schemas, API behavior, and validation expectations apply to a server-side implementation?
- Which Part 1 requirements are direct Glaux Server implementation obligations?
- Which Part 1 requirements are inherited from or related to OGC API - Features and common OGC API behavior?
- Which Part 1 requirements affect downstream research on API behavior, resource models, identifiers, links, query/filter behavior, content negotiation, error handling, OpenAPI documentation, conformance, validation, and testing?
- What traceable requirement inventory should later topics use when designing and testing Glaux Server?

The output must be a Part 1 requirement inventory with source anchors, implementation classification, downstream handoffs, and test-strategy implications.

### Why This Topic Order

This topic begins the detailed CSAPI standards extraction work after the initial obligation and boundary baseline topics:

- `IDR-SRV-001` establishes the STANAG 4789 / AEP-4789 obligation baseline.
- `IDR-SRV-002` maps AEP-4789 Volume I functions to server responsibilities.
- `IDR-SRV-003` establishes the AEP-4789 Volume II standards-package implementation baseline.
- `IDR-SRV-004` establishes terminology and concept alignment.
- `IDR-SRV-005` identifies related NATO standards boundaries.

This topic must precede Part 2 requirement extraction and later server design topics because Part 1 provides the feature-resource and metadata foundation for connected-system discovery, navigation, resources, collections, links, properties, systems, procedures, deployments, sampling features, and related API behavior.

### Critical Constraint(s)

- Treat the official OGC API - Connected Systems Part 1 standard as the controlling source for this topic.
- Extract requirements; do not design the final Glaux Server implementation.
- Do not merge Part 1 and Part 2 obligations. Record Part 2 references or dependencies as handoffs to `IDR-SRV-007`.
- Do not perform full conformance-mapping synthesis here; hand detailed conformance-class mapping to `IDR-SRV-008`.
- Do not finalize the canonical resource model here; hand resource-model implications to `IDR-SRV-015` through `IDR-SRV-020`.
- Do not design the OpenAPI publication strategy here; hand API documentation implications to `IDR-SRV-014`.
- Do not invent project-specific interpretations where the standard is explicit.
- Clearly distinguish normative requirements, recommendations, examples, notes, inherited OGC API behavior, schema constraints, and implementation implications.
- Preserve traceability to exact requirement identifiers, clauses, requirement classes, conformance classes, tables, schemas, examples, and OpenAPI artifacts wherever possible.

---

## 2. Research Questions

### Core Questions

1. What normative OGC API - Connected Systems Part 1 requirements apply to Glaux Server?
2. Which Part 1 requirement classes, conformance classes, resources, schemas, and API behaviors must be included in the Glaux Server requirement baseline?
3. Which requirements are direct server obligations, which are inherited from OGC API - Features or common OGC API behavior, and which require downstream interpretation?
4. Which Part 1 requirements affect later research topics for resource modeling, API behavior, content negotiation, validation, conformance, and testing?
5. What requirement inventory format should downstream topics use to preserve traceability from standard to implementation and tests?

### Detailed Questions

#### Requirement Extraction

- What requirement classes are defined by CSAPI Part 1?
- What conformance classes are defined or referenced by CSAPI Part 1?
- Which requirements are mandatory for a conforming server?
- Which requirements are optional or conditional, and what triggers them?
- Which requirements use normative language versus informative examples or recommendations?
- Which requirements depend on or reference OGC API - Features behavior?

#### Resource and Collection Behavior

- What Part 1 requirements apply to landing pages, API definition, conformance declaration, and resource discovery?
- What requirements apply to collections and feature resources?
- What requirements apply to systems, procedures, deployments, sampling features, properties, or other connected-system resources?
- What requirements apply to links, identifiers, relationships, nesting, navigation, and resource references?
- What requirements affect canonical resource modeling in later topics?

#### Query and Retrieval Behavior

- What Part 1 requirements apply to resource retrieval, collection access, feature access, filtering, selection, pagination, sorting, or query parameters?
- Which behaviors are inherited from OGC API - Features?
- Which behaviors are CSAPI-specific?
- Which requirements need later research in `IDR-SRV-011`?
- Which query behaviors affect persistence and indexing topics?

#### Representations, Schemas, and Encodings

- What schemas and representation models are defined or referenced by Part 1?
- What media types, encodings, JSON structures, GeoJSON structures, or linked-data patterns are required or implied?
- Which schema artifacts must be reviewed directly?
- Which requirements affect content negotiation in `IDR-SRV-012`?
- Which requirements affect schema and encoding validation in `IDR-SRV-023`?

#### Error, Validation, and Conformance Implications

- What Part 1 requirements imply validation rules?
- What requirements imply error responses or failure semantics?
- Which requirements need negative tests?
- Which requirements should feed conformance harness planning?
- Which requirements should feed requirement-to-test traceability?

#### Server Boundary and Implementation Implications

- Which Part 1 requirements clearly belong to Glaux Server?
- Which requirements imply server-side contracts with Publisher, Simulator, clients, or external systems?
- Which requirements are outside the Glaux Server boundary or deferred to other components?
- Which requirements require project decision because the standard allows implementation choice?
- Which requirements may be clarified by existing CSAPI implementation studies in `IDR-SRV-014A` through `IDR-SRV-014G`?

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

- `IDR-SRV-001` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md`
- `IDR-SRV-002` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md`
- `IDR-SRV-003` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md`
- `IDR-SRV-004` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-004-terminology-and-concept-crosswalk-report.md`
- `IDR-SRV-005` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-005-related-nato-standards-boundary-review-report.md`

### Controlling OGC Sources

- OGC API - Connected Systems landing page:
  - https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources:
  - https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems public development repository:
  - https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems Part 1 OpenAPI and schema artifacts:
  - https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
  - https://schemas.opengis.net/
- OGC API standards family:
  - https://ogcapi.ogc.org/
- OGC API - Features landing page:
  - https://ogcapi.ogc.org/features/
- OGC API - Features Part 1 standard:
  - https://docs.ogc.org/is/17-069r4/17-069r4.html

### Related Standards-Package Sources

- OGC API - Connected Systems - Part 2: Dynamic Data, for cross-reference and dependency awareness only:
  - https://docs.ogc.org/is/23-002/23-002.html
- OGC SensorML Encoding Standard 3.0, for resource-description dependency awareness only:
  - https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0, for representation and data-component dependency awareness only:
  - https://docs.ogc.org/is/24-014/24-014.html
- W3C Semantic Sensor Network Ontology / SOSA, where Part 1 terminology or semantic alignment requires it:
  - https://www.w3.org/TR/vocab-ssn/

---

## 4. Supporting Resources

Use these sources to interpret project context, downstream dependencies, expected research depth, and report style.

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
- OS4CSAPI client work, for later interoperability context only:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability findings repository, for later interoperability context only:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd
- OS4CSAPI discussions, for background only:
  - https://github.com/orgs/OS4CSAPI/discussions

---

## 5. Research Methodology

### Phase 1: Source Collection and Requirement-Extraction Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and define the extraction framework for Part 1 requirements.

**Tasks:**

1. Gather the official CSAPI Part 1 standard, schemas, OpenAPI artifacts, and related OGC API sources.
2. Gather prior topic reports from `IDR-SRV-001` through `IDR-SRV-005`, if available.
3. Identify Part 1 requirement classes, conformance classes, annexes, schemas, examples, and referenced standards.
4. Define the requirement inventory fields to be used in the report.
5. Define how to classify requirements as direct server obligation, inherited OGC API behavior, conditional/optional obligation, downstream handoff, or out-of-scope.

**Expected Output:** Source inventory and requirement-extraction framework.

### Phase 2: Normative Requirement Extraction (3-4 hours)

**Objective:** Extract all server-relevant normative requirements from CSAPI Part 1.

**Tasks:**

1. Review CSAPI Part 1 section by section for normative requirements.
2. Extract requirement identifiers, requirement text summaries, source anchors, and requirement-class membership.
3. Identify related conformance classes and any implementation conditions.
4. Capture references to schemas, OpenAPI artifacts, examples, or linked requirements.
5. Mark items that are informative, explanatory, or examples rather than normative requirements.

**Expected Output:** Draft Part 1 normative requirement inventory.

### Phase 3: Server Applicability and Boundary Classification (2-3 hours)

**Objective:** Determine how each extracted requirement applies to Glaux Server.

**Tasks:**

1. Classify each requirement as:
   - direct Glaux Server implementation obligation,
   - inherited OGC API behavior,
   - conditional or optional server obligation,
   - server-side integration contract implication,
   - downstream research handoff,
   - not applicable / out of scope.
2. Identify whether each requirement affects API behavior, resource model, identifiers, links, query behavior, content negotiation, validation, persistence, conformance, or testing.
3. Identify requirements that require project interpretation or implementation choice.
4. Identify requirements that should be validated against existing implementation studies later.
5. Document uncertain applicability and reason for uncertainty.

**Expected Output:** Requirement applicability and boundary classification matrix.

### Phase 4: Resource, Schema, and API Behavior Mapping (2.5-3.5 hours)

**Objective:** Map Part 1 requirements to server resource and API behavior areas.

**Tasks:**

1. Group requirements by API/resource behavior:
   - landing page,
   - API definition,
   - conformance declaration,
   - collections,
   - feature resources,
   - systems,
   - procedures,
   - deployments,
   - sampling features,
   - properties,
   - links and relationships,
   - query/retrieval behavior,
   - schemas and representations.
2. Identify schema and OpenAPI artifacts associated with each group.
3. Identify downstream handoffs to Category B topics `IDR-SRV-009` through `IDR-SRV-014`.
4. Identify downstream handoffs to resource-model topics `IDR-SRV-015` through `IDR-SRV-020`.
5. Identify validation and fixture implications for later testing topics.

**Expected Output:** Part 1 requirement-to-resource/API behavior map.

### Phase 5: Traceability and Test Implication Analysis (1.5-2.5 hours)

**Objective:** Prepare Part 1 findings for conformance, verification, and test-driven design.

**Tasks:**

1. Define how Part 1 requirements should be carried into requirement-to-test traceability.
2. Identify candidate positive tests, negative tests, conformance tests, schema tests, and golden-file tests implied by the requirements.
3. Identify requirements requiring external-client interoperability tests.
4. Identify requirements needing fixture or scenario data.
5. Record handoffs to `IDR-SRV-008`, `IDR-SRV-050`, `IDR-SRV-051`, `IDR-SRV-053`, and `IDR-SRV-056`.

**Expected Output:** Part 1 traceability and test-implication notes.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert research output into a decision-usable Part 1 requirement baseline.

**Tasks:**

1. Consolidate extracted requirements, classifications, resource/API mappings, and test implications.
2. Resolve conflicts and ambiguities where possible.
3. Produce findings grouped by requirement class and server behavior area.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] CSAPI Part 1 has been reviewed directly from the official OGC source.
- [ ] Part 1 requirement classes and conformance classes are identified.
- [ ] Server-relevant normative requirements are extracted with source anchors.
- [ ] Informative text, examples, recommendations, and normative requirements are distinguished.
- [ ] Requirements are classified for Glaux Server applicability and boundary.
- [ ] Inherited OGC API - Features or common OGC API behavior is identified where relevant.
- [ ] Schema, OpenAPI, and representation artifacts associated with Part 1 requirements are identified.
- [ ] Downstream handoffs to API behavior, resource model, validation, conformance, and testing topics are documented.
- [ ] Requirement-to-test implications are captured at a high level.
- [ ] Unresolved interpretation questions are explicitly listed.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** CSAPI Part 1 Requirement Baseline Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-006-csapi-part-1-requirement-baseline-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Requirement extraction methodology
5. Part 1 requirement-class and conformance-class inventory
6. Normative requirement inventory
7. Server applicability and boundary classification matrix
8. Resource/API behavior mapping
9. Schema, OpenAPI, and representation artifact inventory
10. Downstream topic handoff matrix
11. Traceability and test-strategy implications
12. Recommendations
13. Risks, constraints, and open questions
14. Validation against this plan's success criteria
15. References

The requirement inventory should include, at minimum:

- Requirement ID or source anchor
- Requirement class
- Conformance class, where applicable
- Requirement summary
- Normative / informative classification
- Applicability to Glaux Server
- Affected resource/API area
- Related schema/OpenAPI artifact
- Downstream topic handoff
- Test implication
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-005` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 standard and related schema/OpenAPI sources must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-007: CSAPI Part 2 Requirement Baseline`
- `IDR-SRV-008: Conformance Class and Requirement Mapping`
- `IDR-SRV-009: Landing Page, API Definition, and Conformance Declaration Behavior`
- `IDR-SRV-010: Collections, Resources, Links, and Navigation Behavior`
- `IDR-SRV-010A: API Versioning, Backward Compatibility, and Deprecation Strategy`
- `IDR-SRV-011: Query, Filtering, Sorting, Pagination, and Selection Semantics`
- `IDR-SRV-012: Content Negotiation, Media Types, and Encoding Selection`
- `IDR-SRV-013: Error Model, HTTP Status Codes, and Failure Semantics`
- `IDR-SRV-014: OpenAPI Description and API Documentation Strategy`
- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy`
- `IDR-SRV-017: Relationship and Linkage Model`
- `IDR-SRV-021: SensorML Representation Strategy`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
- `IDR-SRV-056: Interoperability Test Matrix for External CSAPI Clients`
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

- This topic should extract and classify Part 1 requirements, not make final design decisions for every requirement.
- OGC API - Features inherited behavior may be important and should be captured as dependency or inherited behavior rather than ignored.
- Some Part 1 requirements may depend on Part 2, SensorML, SWE Common, schemas, or implementation profiles. Those should be handed to later topics.
- Open question: Which Part 1 conformance classes are mandatory for the intended Glaux Server baseline, and which are optional or conditional?
- Open question: Does AEP-4789 Volume II adopt all of Part 1 or a specific subset/profile?
- Open question: Which Part 1 requirements require project-specific implementation choices?
- Risk: Missing inherited OGC API behavior could cause downstream conformance gaps.
- Risk: Treating informative examples as normative requirements could overconstrain the server.
- Risk: Treating normative requirements as implementation details could weaken traceability and test strategy.

---

## References

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
- OGC API - Connected Systems landing page:
  - https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources:
  - https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems public development repository:
  - https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts:
  - https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC schemas:
  - https://schemas.opengis.net/
- OGC API standards family:
  - https://ogcapi.ogc.org/
- OGC API - Features landing page:
  - https://ogcapi.ogc.org/features/
- OGC API - Features Part 1:
  - https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC API - Connected Systems - Part 2: Dynamic Data:
  - https://docs.ogc.org/is/23-002/23-002.html
- OGC SensorML Encoding Standard 3.0:
  - https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0:
  - https://docs.ogc.org/is/24-014/24-014.html
- W3C Semantic Sensor Network Ontology / SOSA:
  - https://www.w3.org/TR/vocab-ssn/
- OS4CSAPI testing research-plan corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans

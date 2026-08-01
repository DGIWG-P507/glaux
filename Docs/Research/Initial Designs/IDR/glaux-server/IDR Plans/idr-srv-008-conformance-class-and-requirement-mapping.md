# Section 008: Conformance Class and Requirement Mapping - Research Plan

**Status:** Complete<br>
**Last Updated:** August 1, 2026<br>
**Estimated Research Time:** 13-17 hours<br>
**Actual Research Time:** Approximately 4 hours of AI-assisted elapsed execution time, including three parallel independent read-only audits, on August 1, 2026<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-008-conformance-class-and-requirement-mapping-report.md`

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

This topic research must produce a **conformance class and requirement mapping baseline** for Glaux Server across OGC API - Connected Systems Part 1, OGC API - Connected Systems Part 2, inherited OGC API behavior, and related standards-package dependencies.

The research must answer:

- Which CSAPI Part 1 and Part 2 conformance classes are relevant to Glaux Server?
- Which requirements belong to each conformance class or requirement class?
- Which conformance classes appear mandatory, optional, conditional, inherited, or dependent on implementation choices?
- Which conformance claims should Glaux Server plan to support, defer, or treat as conditional pending later design decisions?
- What traceability structure should connect conformance classes, requirements, implementation topics, and tests?

The output must be a conformance matrix baseline that downstream topics can use for server behavior design, validation, API documentation, implementation sequencing, conformance harness planning, and requirement-to-test traceability.

### Why This Topic Order

This topic follows:

- `IDR-SRV-006: CSAPI Part 1 Requirement Baseline`
- `IDR-SRV-007: CSAPI Part 2 Requirement Baseline`

Those topics extract Part 1 and Part 2 requirements separately. This topic consolidates them into a conformance-oriented structure so later API behavior, resource model, validation, test strategy, and implementation sequencing topics can reason from a single conformance map.

This topic must precede detailed behavior topics such as landing page behavior, collections/navigation behavior, query semantics, content negotiation, error handling, OpenAPI documentation, and later conformance-harness design because those topics must know which conformance classes and requirement sets they are supporting.

### Critical Constraint(s)

- Treat official OGC standards, requirement classes, conformance classes, schemas, and conformance URIs as authoritative.
- Use `IDR-SRV-006` and `IDR-SRV-007` outputs as input requirement inventories when available.
- Do not re-perform full Part 1 and Part 2 requirement extraction except to resolve mapping gaps.
- Do not design the final conformance test harness here; that belongs to `IDR-SRV-050`.
- Do not finalize requirement-to-test tooling here; that belongs to `IDR-SRV-051`.
- Do not make final implementation-sequencing decisions here, but identify conformance dependencies and sequencing implications.
- Clearly distinguish:
  - requirement classes,
  - conformance classes,
  - conformance declaration URIs,
  - normative requirements,
  - inherited OGC API behavior,
  - conditional/optional conformance,
  - implementation choices,
  - downstream design and test implications.
- Preserve traceability to exact requirement identifiers, conformance class identifiers, clauses, tables, schemas, and OpenAPI artifacts wherever possible.
- Consult the shared upstream-history register for official issues, linked pull requests, commits, and releases that materially clarify or challenge a mapping. Preserve these as informative maintenance context; do not let an open issue or unmerged change override the approved standard.

---

## 2. Research Questions

### Core Questions

1. Which CSAPI Part 1 and Part 2 conformance classes apply to Glaux Server?
2. Which requirements map to each conformance class, requirement class, or conformance declaration URI?
3. Which conformance classes are mandatory, optional, conditional, inherited, or dependent on project implementation choices?
4. Which conformance claims should Glaux Server plan to support, defer, or evaluate later?
5. What conformance-to-requirement-to-test traceability model should downstream IDR topics use?

### Detailed Questions

#### Conformance Class Inventory

- What conformance classes are defined by CSAPI Part 1?
- What conformance classes are defined by CSAPI Part 2?
- What requirement classes are associated with each conformance class?
- Which conformance classes are inherited from OGC API - Features or common OGC API behavior?
- Which conformance classes depend on encodings, OpenAPI, schemas, or optional capabilities?
- What conformance URIs or identifiers must appear in a server conformance declaration?

#### Requirement Mapping

- Which Part 1 requirements map to each Part 1 conformance class?
- Which Part 2 requirements map to each Part 2 conformance class?
- Which requirements are shared, reused, inherited, or cross-referenced across parts?
- Which requirements have no obvious conformance-class mapping and require interpretation?
- Which requirements depend on SensorML, SWE Common, schemas, OpenAPI, or implementation profiles?

#### Upstream Standards-Maintenance Context

- Which official CSAPI issues, linked pull requests, commits, or release records materially clarify a requirement-to-class mapping, conformance declaration, prerequisite, or known publication defect?
- Is each relevant upstream record reflected in the published `v1.0.0` baseline, merged only after publication, documented as discussion or rationale, or still unresolved?
- Which unresolved upstream records must remain explicit compatibility or interpretation risks rather than silently changing the Glaux conformance profile?

#### Applicability to Glaux Server

- Which conformance classes appear necessary for the intended full-scope Glaux Server baseline?
- Which conformance classes may be optional but strategically important?
- Which conformance classes may require future project decisions?
- Which conformance classes may be deferred without reducing the intended full-scope server model?
- Which conformance classes belong outside the Glaux Server boundary?

#### Conformance Declaration Behavior

- What conformance information must a CSAPI server expose?
- What conformance URIs should Glaux Server plan to declare?
- What evidence must exist before Glaux Server declares a conformance class?
- How should partial, staged, experimental, or deferred conformance be handled in planning?
- What later topics must research conformance declaration behavior in detail?

#### Validation and Test Implications

- What positive and negative tests are implied by each conformance class?
- Which conformance classes require schema validation, OpenAPI validation, resource-model tests, dynamic-data tests, tasking tests, streaming tests, or interoperability tests?
- Which conformance classes require fixture or golden-file support?
- Which conformance classes require external-client validation?
- What information must be handed to `IDR-SRV-050` and `IDR-SRV-051`?

#### Implementation Sequencing Implications

- Which conformance classes depend on foundational server behavior?
- Which conformance classes depend on resource model completion?
- Which conformance classes depend on persistence, time-series storage, streaming, tasking, or security decisions?
- Which conformance classes can be independently researched, designed, or tested?
- Which sequencing dependencies should be captured for the implementation roadmap without reducing full-scope intent?

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

- `IDR-SRV-006` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-006-csapi-part-1-requirement-baseline-report.md`
- `IDR-SRV-007` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-007-csapi-part-2-requirement-baseline-report.md`
- Shared OGC API - Connected Systems upstream-history register:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Evidence/ogc-connected-systems-upstream-history-register.md`
- Earlier baseline reports, if needed for scope and terminology:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md`
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md`
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md`
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-004-terminology-and-concept-crosswalk-report.md`
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-005-related-nato-standards-boundary-review-report.md`

### Controlling OGC Sources

- OGC API - Connected Systems landing page:
  - https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources:
  - https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data:
  - https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository:
  - https://github.com/opengeospatial/ogcapi-connected-systems
- Official issue, pull-request, and release history, filtered through the shared register:
  - https://github.com/opengeospatial/ogcapi-connected-systems/issues
  - https://github.com/opengeospatial/ogcapi-connected-systems/pulls
  - https://github.com/opengeospatial/ogcapi-connected-systems/releases
- OGC API - Connected Systems OpenAPI and schema artifacts:
  - https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
  - https://schemas.opengis.net/
- OGC API - Features Part 1 standard, for inherited conformance behavior:
  - https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC API standards family:
  - https://ogcapi.ogc.org/

### Related Standards-Package Sources

- OGC SensorML Encoding Standard 3.0:
  - https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0:
  - https://docs.ogc.org/is/24-014/24-014.html
- W3C Semantic Sensor Network Ontology / SOSA, where semantic-conformance alignment requires it:
  - https://www.w3.org/TR/vocab-ssn/

### Future Conformance and Testing Context Sources

Use these as supporting context only, not as substitutes for the official standards:

- OGC TEAM Engine / CITE information, where relevant:
  - https://github.com/opengeospatial/teamengine
  - https://github.com/opengeospatial/ets-ogcapi-features10
- OGC Compliance Program:
  - https://www.ogc.org/compliance/
- Existing CSAPI implementation and interoperability topics, once researched:
  - `IDR-SRV-014A` through `IDR-SRV-014G`

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

### Phase 1: Source Collection and Mapping Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and define the mapping framework for conformance classes and requirements.

**Tasks:**

1. Gather the official CSAPI Part 1 and Part 2 standards, schemas, OpenAPI artifacts, and inherited OGC API sources.
2. Gather `IDR-SRV-006` and `IDR-SRV-007` research reports, if available.
3. Consult the shared upstream-history register; refresh the state and linked resolution evidence only for entries that can affect Part 1/Part 2 requirement, class, prerequisite, declaration, or profile mapping.
4. Identify the conformance class and requirement class structures used in Part 1 and Part 2.
5. Define the conformance mapping fields to be used in the report.
6. Define classification values for conformance classes:
   - mandatory,
   - optional,
   - conditional,
   - inherited,
   - deferred,
   - not applicable / out of scope,
   - unresolved.

**Expected Output:** Source inventory and conformance-mapping framework.

### Phase 2: Conformance Class Inventory (2.5-3.5 hours)

**Objective:** Identify all relevant conformance classes, requirement classes, and conformance declaration identifiers.

**Tasks:**

1. Extract conformance classes from CSAPI Part 1.
2. Extract conformance classes from CSAPI Part 2.
3. Identify inherited or referenced conformance classes from OGC API - Features or common OGC API behavior.
4. Identify conformance URIs, identifiers, or declaration strings that a server may need to expose.
5. Capture source anchors for each conformance class, requirement class, and declaration identifier.

**Expected Output:** Conformance class and requirement class inventory.

### Phase 3: Requirement-to-Conformance Mapping (3-4 hours)

**Objective:** Map requirements from Part 1 and Part 2 to their associated conformance classes and requirement classes.

**Tasks:**

1. Use the Part 1 requirement inventory from `IDR-SRV-006`, if available.
2. Use the Part 2 requirement inventory from `IDR-SRV-007`, if available.
3. Map each requirement to its requirement class and conformance class.
4. Identify shared, inherited, cross-referenced, or duplicated requirements.
5. Identify requirements that lack clear mapping or require interpretation.
6. Identify schema, OpenAPI, encoding, or validation artifacts associated with each mapping.
7. Link materially relevant upstream-history records to the affected mapping and state whether they explain the published baseline, document post-publication direction, or remain unresolved.

**Expected Output:** Requirement-to-conformance mapping matrix.

### Phase 4: Glaux Server Applicability and Conformance Posture Analysis (2-3 hours)

**Objective:** Determine how each conformance class should be treated for Glaux Server planning.

**Tasks:**

1. Classify each conformance class as mandatory, optional, conditional, inherited, deferred, not applicable, or unresolved for Glaux Server.
2. Identify evidence and reasoning for each classification.
3. Identify what implementation evidence would be required before Glaux Server can declare each conformance class.
4. Identify dependencies on resource modeling, dynamic data, tasking, streaming, validation, persistence, security, or deployment topics.
5. Identify conformance classes that require project decision or implementation sequencing guidance.
6. Prevent unresolved upstream proposals from being promoted into the Glaux profile without an explicit project decision and compatibility rationale.

**Expected Output:** Glaux Server conformance posture table with dependencies and evidence needs.

### Phase 5: Test and Verification Implication Analysis (2-2.5 hours)

**Objective:** Prepare conformance findings for later conformance harness and requirement-to-test traceability research.

**Tasks:**

1. Identify test categories needed for each conformance class:
   - API contract tests,
   - schema validation tests,
   - resource-model tests,
   - positive behavior tests,
   - negative behavior tests,
   - dynamic-data tests,
   - tasking lifecycle tests,
   - streaming tests,
   - security/authorization tests,
   - interoperability tests.
2. Identify which requirements need fixtures, golden files, scenarios, or external-client tests.
3. Identify which conformance claims require automated evidence, manual review, or both.
4. Identify likely handoffs to `IDR-SRV-050`, `IDR-SRV-051`, `IDR-SRV-053`, `IDR-SRV-054`, `IDR-SRV-055`, and `IDR-SRV-056`.
5. Identify any existing OGC conformance tools or gaps that later topics should investigate.

**Expected Output:** Conformance verification and test-implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert research output into a decision-usable conformance matrix baseline.

**Tasks:**

1. Consolidate conformance classes, requirement mappings, applicability classifications, dependencies, and test implications.
2. Resolve conflicts and ambiguities where possible.
3. Produce findings grouped by Part 1, Part 2, inherited behavior, and cross-cutting conformance concerns.
4. Produce recommendations for downstream IDR topic use.
5. Update the shared register only where this topic establishes a newer retrieval state, linked resolution, authority classification, or downstream handoff.
6. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [x] CSAPI Part 1 and Part 2 conformance classes are identified with source anchors.
- [x] Requirement classes and conformance class relationships are documented.
- [x] Conformance declaration identifiers or URIs are identified where available.
- [x] Part 1 and Part 2 requirements are mapped to conformance classes where possible.
- [x] Inherited OGC API behavior is identified where relevant.
- [x] Each conformance class is classified for Glaux Server applicability.
- [x] Dependencies and evidence needs for conformance claims are identified.
- [x] Relevant official issue/PR/release history is date-checked, linked to affected mappings, and authority-classified without changing the normative baseline.
- [x] Downstream handoffs to behavior, validation, testing, conformance harness, and requirement-to-test topics are documented.
- [x] Unresolved mapping or applicability issues are explicitly listed.
- [x] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Conformance Class and Requirement Mapping Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-008-conformance-class-and-requirement-mapping-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Conformance mapping methodology
5. CSAPI Part 1 conformance class inventory
6. CSAPI Part 2 conformance class inventory
7. Inherited OGC API conformance/dependency inventory
8. Requirement-to-conformance mapping matrix
9. Glaux Server conformance posture table
10. Upstream standards-maintenance context and disposition
11. Conformance declaration implications
12. Verification and test-strategy implications
13. Downstream topic handoff matrix
14. Recommendations
15. Risks, constraints, and open questions
16. Validation against this plan's success criteria
17. References

The mapping matrix should include, at minimum:

- Standard / source part
- Requirement class
- Conformance class
- Conformance URI or identifier, if available
- Requirement ID or source anchor
- Requirement summary
- Mandatory / optional / conditional / inherited status
- Glaux Server applicability
- Related schema/OpenAPI/encoding artifact
- Related upstream-history record and authority class, when material
- Publication/release relationship of any upstream resolution
- Implementation dependency
- Evidence required to claim conformance
- Test implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-006` and `IDR-SRV-007` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2 standards and related schema/OpenAPI sources must be reachable or explicitly marked unavailable.
- The shared upstream-history register and relevant official issue/PR/release records must be reachable or their evidence limitations explicitly recorded.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-009: Landing Page, API Definition, and Conformance Declaration Behavior`
- `IDR-SRV-010: Collections, Resources, Links, and Navigation Behavior`
- `IDR-SRV-010A: API Versioning, Backward Compatibility, and Deprecation Strategy`
- `IDR-SRV-011: Query, Filtering, Sorting, Pagination, and Selection Semantics`
- `IDR-SRV-012: Content Negotiation, Media Types, and Encoding Selection`
- `IDR-SRV-013: Error Model, HTTP Status Codes, and Failure Semantics`
- `IDR-SRV-014: OpenAPI Description and API Documentation Strategy`
- `IDR-SRV-021: SensorML Representation Strategy`
- `IDR-SRV-022: SWE Common Data Component Strategy`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
- `IDR-SRV-054: Performance, Load, Stress, and Streaming Test Strategy`
- `IDR-SRV-055: Security, Authorization, and Command-Control Test Strategy`
- `IDR-SRV-056: Interoperability Test Matrix for External CSAPI Clients`
- Final overall IDR synthesis report

---

## 9. Research Status Checklist

Update this section as work progresses.

- [x] Phase 1 complete
- [x] Phase 2 complete
- [x] Phase 3 complete
- [x] Phase 4 complete
- [x] Phase 5 complete
- [x] Phase 6 synthesis complete
- [x] Deliverable draft complete
- [x] Deliverable reviewed
- [x] Deliverable accepted

**Actual Research Time:** Approximately 4 hours of AI-assisted elapsed execution time, including three parallel independent read-only audits, on August 1, 2026<br>
**Research Execution Completed:** August 1, 2026<br>
**Completion Date:** August 1, 2026

---

## 10. Notes and Open Questions

- This topic should consolidate conformance mapping; it should not duplicate full Part 1 and Part 2 requirement extraction.
- Some conformance classes may be technically optional but practically necessary for the intended Glaux Server baseline. The report should distinguish standards obligation from project intent.
- Conformance declaration must be evidence-based. The report should identify evidence needed before future implementation claims conformance.
- The upstream register is a navigation and rationale aid, not a replacement source of requirements; every material entry must be reconciled to the approved standard and its release relationship.
- Open question: Does AEP-4789 Volume II imply a specific subset or profile of CSAPI conformance classes for Glaux Server?
- Open question: Are there official CSAPI conformance test resources available, or must Glaux Server create its own conformance evidence model?
- Open question: How should Glaux Server handle staged conformance during implementation without reducing the full intended capability model?
- Risk: Declaring conformance too broadly without test evidence could undermine credibility.
- Risk: Mapping requirements too loosely could weaken test-driven implementation.
- Risk: Treating optional conformance classes as out of scope too early could shrink the intended server capability model.

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
- OGC API - Connected Systems - Part 2: Dynamic Data:
  - https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository:
  - https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems v1.0.0 tagged API artifacts:
  - https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC API - Connected Systems v1.0.0 release and bundled OpenAPI 3.1 artifacts:
  - https://github.com/opengeospatial/ogcapi-connected-systems/releases/tag/v1.0.0
- OGC schemas:
  - https://schemas.opengis.net/
- OGC API standards family:
  - https://ogcapi.ogc.org/
- OGC API - Features Part 1:
  - https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0:
  - https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0:
  - https://docs.ogc.org/is/24-014/24-014.html
- W3C Semantic Sensor Network Ontology / SOSA:
  - https://www.w3.org/TR/vocab-ssn/
- OGC Compliance Program:
  - https://www.ogc.org/compliance/
- OGC TEAM Engine:
  - https://github.com/opengeospatial/teamengine
- OGC API - Features ETS:
  - https://github.com/opengeospatial/ets-ogcapi-features10
- OS4CSAPI testing research-plan corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans

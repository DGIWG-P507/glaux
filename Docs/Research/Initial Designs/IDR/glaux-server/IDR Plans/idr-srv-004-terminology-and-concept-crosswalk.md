# Section 004: Terminology and Concept Crosswalk - Research Plan

**Status:** Planned  
**Last Updated:** June 7, 2026  
**Estimated Research Time:** 10-14 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-004-terminology-and-concept-crosswalk-report.md`

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

This topic research must produce a **terminology and concept crosswalk** that aligns STANAG 4789 / AEP-4789 terminology with the terminology used by OGC API - Connected Systems, SensorML, SWE Common, SOSA/SSN where relevant, and Glaux Server planning.

The research must answer:

- Which terms and concepts from STANAG 4789 / AEP-4789 are essential for Glaux Server design?
- Which equivalent, near-equivalent, broader, narrower, or conflicting terms appear in CSAPI, SensorML, SWE Common, SOSA/SSN, and Glaux Server planning documents?
- Which terms must be normalized for Glaux Server planning so later research topics do not use the same word to mean different things or different words to mean the same thing?
- Which terminology conflicts or ambiguities create implementation, validation, interoperability, or conformance risks?

The output must be a terminology and concept crosswalk matrix that later IDR topics can use as a shared vocabulary for requirements extraction, resource modeling, schema validation, dynamic data, tasking, status, security, DDIL-informed behavior, and test strategy.

### Why This Topic Order

This topic follows:

- `IDR-SRV-001: STANAG 4789 / AEP-4789 Server Obligation Baseline`
- `IDR-SRV-002: AEP-4789 Volume I Functional Mapping to Server Responsibilities`
- `IDR-SRV-003: AEP-4789 Volume II Standards Package Implementation Baseline`

Those topics identify obligation, function, and standards-package context. This topic converts that context into a controlled terminology baseline so downstream research can use consistent language.

This topic must precede detailed CSAPI requirement extraction, resource-model design, SensorML/SWE representation strategy, persistence design, and validation strategy because those topics depend on correctly distinguishing concepts such as system, sensor, platform, procedure, deployment, feature, property, observation, datastream, control stream, command, tasking, status, event, result, validity, provenance, and availability.

### Critical Constraint(s)

- Treat authoritative standards definitions as controlling where definitions are explicit.
- Do not invent new project-specific definitions when an authoritative definition exists.
- When sources conflict or use terms differently, document the conflict instead of forcing an artificial equivalence.
- Keep crosswalk findings bounded to Glaux Server research needs.
- Do not attempt to create a full NATO, OGC, or geospatial ontology.
- Do not redesign CSAPI, SensorML, SWE Common, SOSA/SSN, or STANAG/AEP terminology.
- Clearly distinguish:
  - exact equivalence,
  - near equivalence,
  - broader/narrower relationships,
  - related but distinct concepts,
  - conflicting usage,
  - unresolved ambiguity.
- Preserve traceability to source sections, definitions, figures, tables, schemas, or repository artifacts wherever possible.

---

## 2. Research Questions

### Core Questions

1. What terms and concepts from STANAG 4789 / AEP-4789 are necessary for Glaux Server planning and implementation research?
2. How do those terms map to CSAPI, SensorML, SWE Common, SOSA/SSN, and Glaux Server planning terminology?
3. Which terms have exact equivalents, near equivalents, broader/narrower relationships, or conflicting meanings across sources?
4. Which terminology issues create server implementation risks for API behavior, resource modeling, validation, interoperability, tasking, status, security, or DDIL-informed operation?
5. What controlled vocabulary and crosswalk format should downstream IDR topics use?

### Detailed Questions

#### Source Term Extraction

- Which terms are explicitly defined in STANAG 4789 / AEP-4789 source material?
- Which terms are used frequently or operationally important even if not formally defined?
- Which terms are explicitly defined in CSAPI Part 1 and Part 2?
- Which terms are explicitly defined in SensorML and SWE Common?
- Which SOSA/SSN terms are relevant because CSAPI or SensorML relies on or aligns with semantic sensor concepts?
- Which terms already appear in Glaux Server Goal and Definition and the overall IDR plan?

#### Concept Families

- How do sources define or use system, connected system, sensor, actuator, sampler, platform, procedure, process, deployment, and capability?
- How do sources define or use feature, feature resource, sampling feature, feature of interest, observed property, controlled property, property, and phenomenon?
- How do sources define or use datastream, observation, result, phenomenon time, result time, valid time, report time, status, event, and dynamic property?
- How do sources define or use command, tasking, control stream, command status, feasibility, execution, cancellation, and control?
- How do sources define or use metadata, description, encoding, schema, data component, unit, constraint, quality, provenance, lineage, trust, and validity?
- How do sources define or use availability, readiness, health, degraded state, stale state, last-known state, freshness, synchronization, and DDIL?

#### Crosswalk and Equivalence

- Which terms are exact equivalents across source families?
- Which terms are close but not identical?
- Which terms have broader/narrower relationships?
- Which terms are related but should not be treated as synonyms?
- Which terms conflict or carry different assumptions across NATO, OGC, semantic-web, and Glaux contexts?
- Which terms should Glaux Server use as preferred terms in planning documents?

#### Server Design Implications

- Which terminology differences could affect API path/resource naming?
- Which terminology differences could affect database schema or resource model design?
- Which terminology differences could affect validation, schema interpretation, or content negotiation?
- Which terminology differences could affect tasking/control semantics?
- Which terminology differences could affect status, event, freshness, or DDIL-informed behavior?
- Which terminology differences could affect testing and requirement-to-test traceability?

#### Downstream Use

- Which terms must be handed to `IDR-SRV-006` and `IDR-SRV-007` for CSAPI requirement extraction?
- Which terms must be handed to `IDR-SRV-015` through `IDR-SRV-020` for resource and domain modeling?
- Which terms must be handed to `IDR-SRV-021` through `IDR-SRV-024` for SensorML, SWE Common, and semantic representation research?
- Which terms must be handed to dynamic-data, tasking, security, DDIL, and verification topics?
- What unresolved terminology issues require later project decisions?

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
- `IDR-SRV-003` research plan:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline.md`
- `IDR-SRV-003` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md`

### AEP / STANAG Source Material

- STANAG 4789 source material available to the Glaux project team.
  - Record exact title, version/date, status, and project storage location in the report.
- AEP-4789 Volume I source material available to the Glaux project team.
  - Record exact title, volume, version/date, status, and project storage location in the report.
- AEP-4789 Volume II source material available to the Glaux project team.
  - Record exact title, volume, version/date, status, and project storage location in the report.

### OGC and Semantic Source Material

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
- OGC API - Connected Systems OpenAPI artifacts:
  - https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC schemas:
  - https://schemas.opengis.net/
- W3C Semantic Sensor Network Ontology / SOSA:
  - https://www.w3.org/TR/vocab-ssn/
- OGC/W3C Spatial Data on the Web Best Practices, for semantic and linked-data context where relevant:
  - https://www.w3.org/TR/sdw-bp/

---

## 4. Supporting Resources

Use these sources to interpret project context, downstream dependencies, expected research depth, and reporting style.

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

### Phase 1: Source Collection and Crosswalk Framework Setup (1.5-2 hours)

**Objective:** Establish the source base and define the crosswalk method before extracting terms.

**Tasks:**

1. Gather the current STANAG 4789 / AEP-4789 source material available to the project team.
2. Gather the official OGC, SensorML, SWE Common, and SOSA/SSN sources listed in this plan.
3. Review outputs from `IDR-SRV-001`, `IDR-SRV-002`, and `IDR-SRV-003`, if available.
4. Define the crosswalk categories to be used:
   - exact equivalent,
   - near equivalent,
   - broader concept,
   - narrower concept,
   - related but distinct,
   - conflicting usage,
   - unresolved.
5. Define the recommended crosswalk matrix columns.

**Expected Output:** Source inventory and crosswalk framework definition.

### Phase 2: STANAG / AEP Terminology Extraction (2-3 hours)

**Objective:** Extract server-relevant terms and concepts from STANAG 4789 / AEP-4789 source material.

**Tasks:**

1. Identify formally defined terms in STANAG 4789, AEP-4789 Volume I, and AEP-4789 Volume II.
2. Identify frequently used or operationally important terms that are not formally defined.
3. Capture source anchors for each term or concept.
4. Group terms by concept family:
   - systems and connected systems,
   - sensing and observing,
   - metadata and description,
   - access and exchange,
   - dynamic data and streaming,
   - tasking and control,
   - status and availability,
   - trust, security, validity, and provenance,
   - federation and DDIL.
5. Note terms with possible ambiguity or implementation consequences.

**Expected Output:** STANAG/AEP terminology inventory with source anchors and concept-family tags.

### Phase 3: OGC / SensorML / SWE / SOSA-SSN Terminology Extraction (2.5-3.5 hours)

**Objective:** Extract corresponding terms and definitions from the adopted standards package and related semantic sources.

**Tasks:**

1. Extract server-relevant terms from CSAPI Part 1 and Part 2.
2. Extract relevant system/process description terms from SensorML.
3. Extract relevant data component, encoding, value, unit, constraint, and quality terms from SWE Common.
4. Extract relevant semantic sensor terms from SOSA/SSN where they inform CSAPI or SensorML concept alignment.
5. Capture source anchors for each term, definition, or schema artifact.
6. Identify where schemas, OpenAPI files, or examples use terms differently from prose standards.

**Expected Output:** OGC/SensorML/SWE/SOSA-SSN terminology inventory with source anchors and concept-family tags.

### Phase 4: Crosswalk Mapping and Conflict Analysis (2.5-3.5 hours)

**Objective:** Map terminology across sources and identify equivalences, differences, conflicts, and implementation risks.

**Tasks:**

1. Match STANAG/AEP terms to corresponding OGC/SensorML/SWE/SOSA-SSN terms.
2. Classify each mapping using the crosswalk categories defined in Phase 1.
3. Identify preferred Glaux Server planning terms when a project term is needed.
4. Identify conflicts that affect server design, validation, interoperability, or testing.
5. Identify terms that should not be treated as synonyms even if they appear similar.
6. Identify unresolved mappings that require later research or project decision.

**Expected Output:** Draft terminology and concept crosswalk matrix with conflict and ambiguity notes.

### Phase 5: Server Implication and Downstream Handoff Analysis (1.5-2 hours)

**Objective:** Translate terminology findings into downstream research guidance.

**Tasks:**

1. Identify terms that affect API behavior, resource modeling, schema validation, tasking/control, status/events, security, DDIL, or testing.
2. Map terms and unresolved issues to downstream IDR topics.
3. Identify terms that should become preferred planning language for Glaux Server documents.
4. Identify terms that should remain standards-specific and not be normalized into a single project term.
5. Identify candidate glossary content for the final overall IDR report.

**Expected Output:** Downstream topic handoff matrix and recommended Glaux Server terminology guidance.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert research output into a decision-usable terminology and concept crosswalk.

**Tasks:**

1. Consolidate extracted terms, definitions, source anchors, mappings, and conflict notes.
2. Resolve ambiguities where source evidence supports resolution.
3. Produce findings grouped by concept family.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] STANAG 4789 / AEP-4789 source material available to the project team has been reviewed or explicitly marked unavailable.
- [ ] CSAPI Part 1, CSAPI Part 2, SensorML, SWE Common, and SOSA/SSN sources have been reviewed for relevant terms.
- [ ] Source documents used in the report are listed with title, version/date, URL/path, status, and authority classification.
- [ ] Server-relevant terms and concepts are extracted with source anchors.
- [ ] Crosswalk mappings are classified as exact equivalent, near equivalent, broader, narrower, related but distinct, conflicting, or unresolved.
- [ ] Preferred Glaux Server planning terms are recommended where useful.
- [ ] Terminology conflicts and implementation risks are identified.
- [ ] Downstream handoffs to later IDR topics are identified.
- [ ] Unresolved terminology questions are explicitly listed.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Terminology and Concept Crosswalk Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-004-terminology-and-concept-crosswalk-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Crosswalk methodology
5. STANAG/AEP terminology inventory
6. OGC/SensorML/SWE/SOSA-SSN terminology inventory
7. Terminology and concept crosswalk matrix
8. Conflict, ambiguity, and risk analysis
9. Recommended Glaux Server planning terminology
10. Downstream topic handoff matrix
11. Recommendations
12. Risks, constraints, and open questions
13. Validation against this plan's success criteria
14. References

The crosswalk matrix should include, at minimum:

- Source term
- Source family
- Source definition or usage summary
- Source anchor
- Related term(s) in other source families
- Mapping type
- Preferred Glaux Server planning term, if applicable
- Server design implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` research report should be complete or explicitly marked unavailable/deferred.
- `IDR-SRV-002` research report should be complete or explicitly marked unavailable/deferred.
- `IDR-SRV-003` research report should be complete or explicitly marked unavailable/deferred.
- STANAG 4789 / AEP-4789 source material must be available to the researcher or explicitly marked unavailable.
- Official OGC, SensorML, SWE Common, and SOSA/SSN sources must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-005: Related NATO Standards Boundary Review`
- `IDR-SRV-006: CSAPI Part 1 Requirement Baseline`
- `IDR-SRV-007: CSAPI Part 2 Requirement Baseline`
- `IDR-SRV-008: Conformance Class and Requirement Mapping`
- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy`
- `IDR-SRV-017: Relationship and Linkage Model`
- `IDR-SRV-018: Temporal, Validity, and Freshness Model`
- `IDR-SRV-019: Provenance, Lineage, Quality, and Trust Metadata Model`
- `IDR-SRV-020: Status, Availability, and System Event Model`
- `IDR-SRV-021: SensorML Representation Strategy`
- `IDR-SRV-022: SWE Common Data Component Strategy`
- `IDR-SRV-024: Units, Observed Properties, and Semantic Binding Strategy`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-042: DDIL-Informed Server Semantics`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
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

- Different standards may use the same term differently. The report must preserve distinctions instead of flattening them.
- Some terms may be used operationally in STANAG/AEP material but technically in OGC standards; the crosswalk must distinguish operational intent from implementation semantics.
- SOSA/SSN should be used only where it helps explain semantic alignment relevant to CSAPI, SensorML, or Glaux Server. This topic must not become a full ontology-design effort.
- Open question: Which exact STANAG 4789 / AEP-4789 source copies are authoritative for this IDR cycle?
- Open question: Should Glaux Server adopt CSAPI terms as preferred implementation terms even when STANAG/AEP terms differ?
- Open question: Which terms require project-level glossary entries in the final overall IDR report?
- Risk: Poor terminology alignment could cause later topics to create inconsistent resource models or tests.
- Risk: Over-normalizing terminology could hide meaningful differences among NATO, OGC, SensorML, SWE Common, and SOSA/SSN concepts.
- Risk: Under-normalizing terminology could make later research reports inconsistent and harder to synthesize.

---

## References

- Glaux Server Overall IDR Research Plan:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- `IDR-SRV-001` research plan:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline.md`
- `IDR-SRV-002` research plan:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities.md`
- `IDR-SRV-003` research plan:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline.md`
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
- W3C Semantic Sensor Network Ontology / SOSA:
  - https://www.w3.org/TR/vocab-ssn/
- W3C/OGC Spatial Data on the Web Best Practices:
  - https://www.w3.org/TR/sdw-bp/
- OS4CSAPI testing research-plan corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans

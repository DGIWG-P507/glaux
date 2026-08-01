# Section 007: CSAPI Part 2 Requirement Baseline - Research Plan

**Status:** In Progress<br>
**Last Updated:** July 31, 2026<br>
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** Approximately 3 hours of AI-assisted elapsed execution time, including parallel independent extraction and review<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-007-csapi-part-2-requirement-baseline-report.md`

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

This topic research must extract and organize the **OGC API - Connected Systems Part 2: Dynamic Data** requirements that apply to Glaux Server.

The research must answer:

- What normative Part 2 requirements, requirement classes, conformance classes, resource patterns, schemas, API behavior, streaming behavior, tasking behavior, status/event behavior, and validation expectations apply to a server-side implementation?
- Which Part 2 requirements are direct Glaux Server implementation obligations?
- Which Part 2 requirements depend on CSAPI Part 1 feature-resource behavior, SensorML descriptions, SWE Common data components, or inherited OGC API behavior?
- Which Part 2 requirements affect downstream research on dynamic data, datastreams, observations, control streams, commands, feasibility, system events, status, streaming, content negotiation, validation, persistence, conformance, and testing?
- What traceable requirement inventory should later topics use when designing and testing the dynamic-data and interaction portions of Glaux Server?

The output must be a Part 2 requirement inventory with source anchors, implementation classification, downstream handoffs, and test-strategy implications.

### Why This Topic Order

This topic follows `IDR-SRV-006: CSAPI Part 1 Requirement Baseline`.

Part 1 establishes the feature-resource and metadata foundation for connected-system resources. Part 2 builds on that foundation by addressing dynamic data and interaction resources, including observations, datastreams, status, commands, control streams, feasibility, system events, streaming, historical access, and related behavior.

This topic must precede detailed research on dynamic update semantics, streaming, tasking, command lifecycle, feasibility, status/events, persistence, validation, performance, and conformance because those later topics depend on a clear Part 2 requirement baseline.

### Critical Constraint(s)

- Treat the official OGC API - Connected Systems Part 2 standard as the controlling source for this topic.
- Extract requirements; do not design the final Glaux Server implementation.
- Do not re-extract Part 1 requirements except where Part 2 explicitly depends on them.
- Do not perform full conformance-mapping synthesis here; hand detailed conformance-class mapping to `IDR-SRV-008`.
- Do not finalize persistence, streaming, broker, tasking, command lifecycle, or security design here; hand those implications to later topics.
- Do not perform full SensorML or SWE Common design here; record dependencies and hand them to `IDR-SRV-021` and `IDR-SRV-022`.
- Clearly distinguish normative requirements, recommendations, examples, notes, inherited behavior, schema constraints, and implementation implications.
- Preserve traceability to exact requirement identifiers, clauses, requirement classes, conformance classes, tables, schemas, examples, OpenAPI artifacts, and encoding artifacts wherever possible.

---

## 2. Research Questions

### Core Questions

1. What normative OGC API - Connected Systems Part 2 requirements apply to Glaux Server?
2. Which Part 2 requirement classes, conformance classes, resources, schemas, and API behaviors must be included in the Glaux Server requirement baseline?
3. Which requirements are direct server obligations, which depend on Part 1 or other adopted standards, and which require downstream interpretation?
4. Which Part 2 requirements affect later research topics for dynamic data, observations, tasking, commands, feasibility, status/events, streaming, validation, persistence, conformance, and testing?
5. What requirement inventory format should downstream topics use to preserve traceability from standard to implementation and tests?

### Detailed Questions

#### Requirement Extraction

- What requirement classes are defined by CSAPI Part 2?
- What conformance classes are defined or referenced by CSAPI Part 2?
- Which requirements are mandatory for a conforming server?
- Which requirements are optional or conditional, and what triggers them?
- Which requirements use normative language versus informative examples or recommendations?
- Which requirements depend on CSAPI Part 1 resources or inherited OGC API behavior?

#### Dynamic Data Resources

- What requirements apply to datastreams?
- What requirements apply to observations?
- What requirements apply to status information and time-varying properties?
- What requirements apply to historical data access, snapshots, temporal queries, or result retrieval?
- What requirements apply to resource relationships between systems, datastreams, observations, observed properties, features of interest, and procedures?

#### Tasking, Control, and Feasibility

- What requirements apply to control streams?
- What requirements apply to commands and command resources?
- What requirements apply to command status, lifecycle state, cancellation, acceptance, rejection, or execution tracking?
- What requirements apply to feasibility requests, feasibility results, or asynchronous feasibility/tasking exchanges?
- Which requirements imply authorization, safety, audit, or policy controls that must be handed to later security and command-governance topics?

#### System Events and Status

- What requirements apply to system events?
- What requirements apply to event history, event retrieval, event filtering, or event publication?
- What requirements apply to system status, availability, health, degraded operation, or last-known state?
- Which requirements affect the status/event model in `IDR-SRV-020`?
- Which requirements affect DDIL-informed semantics in `IDR-SRV-042`?

#### Streaming and Real-Time Behavior

- What Part 2 requirements apply to real-time, near-real-time, or streaming access?
- What requirements apply to subscriptions, event publication, bidirectional exchange, ordering, replay, snapshots, or backpressure?
- Which requirements are protocol-specific and which are protocol-neutral?
- Which requirements should be handed to `IDR-SRV-035` and `IDR-SRV-054`?
- What streaming behavior must be tested for conformance, interoperability, and performance?

#### Representations, Schemas, and Encodings

- What schemas and representation models are defined or referenced by Part 2?
- What media types, encodings, JSON structures, SWE Common structures, SensorML references, or linked-resource patterns are required or implied?
- Which schema/OpenAPI artifacts must be reviewed directly?
- Which requirements affect content negotiation in `IDR-SRV-012`?
- Which requirements affect schema and encoding validation in `IDR-SRV-023`?

#### Error, Validation, and Conformance Implications

- What Part 2 requirements imply validation rules?
- What requirements imply error responses or failure semantics?
- Which requirements need negative tests?
- Which requirements should feed conformance harness planning?
- Which requirements should feed requirement-to-test traceability?

#### Server Boundary and Implementation Implications

- Which Part 2 requirements clearly belong to Glaux Server?
- Which requirements imply server-side contracts with Publisher, Simulator, clients, controlled systems, or external systems?
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
- `IDR-SRV-006` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-006-csapi-part-1-requirement-baseline-report.md`

### Controlling OGC Sources

- OGC API - Connected Systems landing page:
  - https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 2: Dynamic Data:
  - https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository:
  - https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems Part 2 OpenAPI and schema artifacts:
  - https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
  - https://schemas.opengis.net/
- OGC API - Connected Systems - Part 1: Feature Resources, for Part 2 dependency context:
  - https://docs.ogc.org/is/23-001/23-001.html
- OGC API standards family:
  - https://ogcapi.ogc.org/
- OGC API - Features Part 1 standard, for inherited feature behavior context where relevant:
  - https://docs.ogc.org/is/17-069r4/17-069r4.html

### Related Standards-Package Sources

- OGC SensorML Encoding Standard 3.0:
  - https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0:
  - https://docs.ogc.org/is/24-014/24-014.html
- W3C Semantic Sensor Network Ontology / SOSA, where Part 2 terminology or semantic alignment requires it:
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

**Objective:** Establish the evidence base and define the extraction framework for Part 2 requirements.

**Tasks:**

1. Gather the official CSAPI Part 2 standard, schemas, OpenAPI artifacts, and related OGC API sources.
2. Gather the Part 1 requirement baseline from `IDR-SRV-006`, if available.
3. Gather prior topic reports from `IDR-SRV-001` through `IDR-SRV-005`, if available.
4. Identify Part 2 requirement classes, conformance classes, annexes, schemas, examples, and referenced standards.
5. Define the requirement inventory fields to be used in the report.
6. Define how to classify requirements as direct server obligation, Part 1-dependent behavior, conditional/optional obligation, downstream handoff, or out-of-scope.

**Expected Output:** Source inventory and requirement-extraction framework.

### Phase 2: Normative Requirement Extraction (3-4 hours)

**Objective:** Extract all server-relevant normative requirements from CSAPI Part 2.

**Tasks:**

1. Review CSAPI Part 2 section by section for normative requirements.
2. Extract requirement identifiers, requirement text summaries, source anchors, and requirement-class membership.
3. Identify related conformance classes and any implementation conditions.
4. Capture references to schemas, OpenAPI artifacts, examples, encodings, protocol guidance, or linked requirements.
5. Mark items that are informative, explanatory, examples, or implementation notes rather than normative requirements.

**Expected Output:** Draft Part 2 normative requirement inventory.

### Phase 3: Server Applicability and Boundary Classification (2-3 hours)

**Objective:** Determine how each extracted requirement applies to Glaux Server.

**Tasks:**

1. Classify each requirement as:
   - direct Glaux Server implementation obligation,
   - Part 1-dependent obligation,
   - conditional or optional server obligation,
   - server-side integration contract implication,
   - downstream research handoff,
   - not applicable / out of scope.
2. Identify whether each requirement affects API behavior, resource model, dynamic data, persistence, streaming, tasking, status/events, validation, security, conformance, or testing.
3. Identify requirements that require project interpretation or implementation choice.
4. Identify requirements that should be validated against existing implementation studies later.
5. Document uncertain applicability and reason for uncertainty.

**Expected Output:** Requirement applicability and boundary classification matrix.

### Phase 4: Dynamic Data, Tasking, and Event Behavior Mapping (3-4 hours)

**Objective:** Map Part 2 requirements to dynamic-data, interaction, and event behavior areas.

**Tasks:**

1. Group requirements by server behavior area:
   - datastreams,
   - observations,
   - historical data access,
   - status and dynamic properties,
   - control streams,
   - commands,
   - command status,
   - feasibility,
   - system events,
   - streaming and subscriptions,
   - snapshots or last-known state where applicable,
   - schemas and representations.
2. Identify schema and OpenAPI artifacts associated with each group.
3. Identify downstream handoffs to Category F topics `IDR-SRV-034` through `IDR-SRV-038`.
4. Identify downstream handoffs to status/event, temporal/freshness, validation, persistence, and security topics.
5. Identify validation and fixture implications for later testing topics.

**Expected Output:** Part 2 requirement-to-dynamic-data/tasking/event behavior map.

### Phase 5: Traceability and Test Implication Analysis (1.5-2.5 hours)

**Objective:** Prepare Part 2 findings for conformance, verification, and test-driven design.

**Tasks:**

1. Define how Part 2 requirements should be carried into requirement-to-test traceability.
2. Identify candidate positive tests, negative tests, conformance tests, schema tests, golden-file tests, streaming tests, tasking lifecycle tests, and authorization-related tests implied by the requirements.
3. Identify requirements requiring external-client interoperability tests.
4. Identify requirements needing fixture data, observation streams, command scenarios, event histories, or replay data.
5. Record handoffs to `IDR-SRV-008`, `IDR-SRV-050`, `IDR-SRV-051`, `IDR-SRV-053`, `IDR-SRV-054`, `IDR-SRV-055`, and `IDR-SRV-056`.

**Expected Output:** Part 2 traceability and test-implication notes.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert research output into a decision-usable Part 2 requirement baseline.

**Tasks:**

1. Consolidate extracted requirements, classifications, dynamic-data/tasking/event mappings, and test implications.
2. Resolve conflicts and ambiguities where possible.
3. Produce findings grouped by requirement class and server behavior area.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] CSAPI Part 2 has been reviewed directly from the official OGC source.
- [ ] Part 2 requirement classes and conformance classes are identified.
- [ ] Server-relevant normative requirements are extracted with source anchors.
- [ ] Informative text, examples, recommendations, and normative requirements are distinguished.
- [ ] Requirements are classified for Glaux Server applicability and boundary.
- [ ] Dependencies on CSAPI Part 1, SensorML, SWE Common, and inherited OGC API behavior are identified where relevant.
- [ ] Schema, OpenAPI, encoding, and representation artifacts associated with Part 2 requirements are identified.
- [ ] Downstream handoffs to dynamic-data, tasking, status/event, validation, conformance, performance, security, and testing topics are documented.
- [ ] Requirement-to-test implications are captured at a high level.
- [ ] Unresolved interpretation questions are explicitly listed.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** CSAPI Part 2 Requirement Baseline Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-007-csapi-part-2-requirement-baseline-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Requirement extraction methodology
5. Part 2 requirement-class and conformance-class inventory
6. Normative requirement inventory
7. Server applicability and boundary classification matrix
8. Dynamic-data/tasking/event behavior mapping
9. Schema, OpenAPI, encoding, and representation artifact inventory
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
- Affected dynamic-data/tasking/status/event/API area
- Related schema/OpenAPI/encoding artifact
- Related Part 1 dependency, if applicable
- Downstream topic handoff
- Test implication
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-006` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 2 standard and related schema/OpenAPI sources must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-008: Conformance Class and Requirement Mapping`
- `IDR-SRV-009: Landing Page, API Definition, and Conformance Declaration Behavior`
- `IDR-SRV-010: Collections, Resources, Links, and Navigation Behavior`
- `IDR-SRV-011: Query, Filtering, Sorting, Pagination, and Selection Semantics`
- `IDR-SRV-012: Content Negotiation, Media Types, and Encoding Selection`
- `IDR-SRV-013: Error Model, HTTP Status Codes, and Failure Semantics`
- `IDR-SRV-014: OpenAPI Description and API Documentation Strategy`
- `IDR-SRV-018: Temporal, Validity, and Freshness Model`
- `IDR-SRV-020: Status, Availability, and System Event Model`
- `IDR-SRV-022: SWE Common Data Component Strategy`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-027: Time-Series Observation Storage Strategy`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-035: Streaming and Event Publication Strategy`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-037: Feasibility and Asynchronous Tasking Strategy`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-042: DDIL-Informed Server Semantics`
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
- [ ] Deliverable accepted

**Actual Research Time:** Approximately 3 hours of AI-assisted elapsed execution time, including parallel independent extraction and review<br>
**Completion Date:** Research completed July 31, 2026; topic completion remains pending plan-owner acceptance

---

## 10. Notes and Open Questions

- This topic should extract and classify Part 2 requirements, not make final design decisions for dynamic-data architecture, streaming protocols, tasking lifecycle, command authorization, or persistence.
- CSAPI Part 2 depends on Part 1 resource foundations; those dependencies should be captured rather than redefined.
- Some Part 2 requirements may depend on SensorML, SWE Common, schemas, or implementation profiles. Those should be handed to later topics.
- Open question: Which Part 2 conformance classes are mandatory for the intended Glaux Server baseline, and which are optional or conditional?
- Open question: Does AEP-4789 Volume II adopt all of Part 2 or a specific subset/profile?
- Open question: Which Part 2 requirements require project-specific implementation choices, especially for streaming, tasking, feasibility, and event publication?
- Risk: Missing dynamic-data or tasking requirements could cause major downstream architecture gaps.
- Risk: Treating optional or protocol-specific behavior as mandatory could overconstrain the server.
- Risk: Treating normative Part 2 behavior as later implementation detail could weaken traceability and test strategy.

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
- OGC API - Connected Systems - Part 2: Dynamic Data:
  - https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository:
  - https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts:
  - https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC schemas:
  - https://schemas.opengis.net/
- OGC API standards family:
  - https://ogcapi.ogc.org/
- OGC API - Connected Systems - Part 1: Feature Resources:
  - https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Features Part 1:
  - https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0:
  - https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0:
  - https://docs.ogc.org/is/24-014/24-014.html
- W3C Semantic Sensor Network Ontology / SOSA:
  - https://www.w3.org/TR/vocab-ssn/
- OS4CSAPI testing research-plan corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans

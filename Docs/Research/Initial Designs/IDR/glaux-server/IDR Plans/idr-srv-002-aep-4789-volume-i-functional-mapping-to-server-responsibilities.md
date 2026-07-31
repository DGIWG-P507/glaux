# Section 002: AEP-4789 Volume I Functional Mapping to Server Responsibilities - Research Plan

**Topic ID:** IDR-SRV-002<br>
**Status:** Complete<br>
**Last Updated:** July 31, 2026<br>
**Estimated Research Time:** 10-14 hours<br>
**Actual Research Time:** Approximately 0.75 hours of AI-assisted elapsed execution time on July 30, 2026<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md`

---

## Usage Instructions

Before executing this plan, review the full exemplar corpus and align the report to the proven standards for content nature, type, level, and detail:

- Full research-plans corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans
- Early exemplar (blueprint-first depth):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md
- Mid-stream exemplar (inventory + sourcing rigor):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/15-fixture-sourcing-organization.md
- End-state synthesis exemplar:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/38-testing-playbook-synthesis.md

This plan is intentionally limited to research planning. It does not execute the research and does not produce the research report.

---

## 1. Research Objective

This topic research must map the functional areas described in **AEP-4789 Volume I** to concrete, bounded **Glaux Server responsibilities**.

The research must answer how AEP-4789 Volume I functions translate into server-side obligations, server-side integration contracts, downstream design topics, and out-of-scope boundaries for Glaux Server.

The functional areas to map include, at minimum:

- discovery,
- registration and description,
- access and exchange,
- streaming and dynamic data,
- tasking and control,
- status and availability,
- security, trust, and authorization,
- federation and cross-boundary information sharing,
- DDIL-informed or constrained-environment behavior.

The output must be a function-to-responsibility mapping baseline that later IDR topics can use to keep Glaux Server aligned with the AEP-4789 reference view without accidentally expanding server scope into the rest of the Glaux ecosystem.

### Why This Topic Order

This topic follows `IDR-SRV-001: STANAG 4789 / AEP-4789 Server Obligation Baseline`.

`IDR-SRV-001` establishes the high-level obligation boundary. This topic takes the next step: it focuses specifically on **AEP-4789 Volume I** and maps its functional view into practical server responsibility categories.

This topic unlocks:

- `IDR-SRV-003`, which studies the AEP-4789 Volume II standards package.
- `IDR-SRV-004`, which builds the terminology and concept crosswalk.
- `IDR-SRV-006` and `IDR-SRV-007`, which extract CSAPI Part 1 and Part 2 requirements.
- Later resource-model, dynamic-data, tasking, status, security, DDIL, and verification research.

### Critical Constraint(s)

- Treat AEP-4789 Volume I as the primary source for this topic.
- Use STANAG 4789 and the `IDR-SRV-001` report as obligation context, not as a substitute for Volume I functional analysis.
- Keep mappings bounded to Glaux Server responsibilities and server-side contracts.
- Do not assign Web App, Mobile, Publisher, or Simulator implementation behavior to Glaux Server.
- Do not treat broad operational functions as direct server obligations unless Volume I or downstream standards evidence supports that mapping.
- Clearly distinguish:
  - direct server responsibilities,
  - server-side integration contract responsibilities,
  - ecosystem responsibilities,
  - future profile/SRD/AEP concerns,
  - out-of-scope concerns.
- Preserve traceability to source sections, clauses, figures, tables, or project-provided source anchors wherever possible.

---

## 2. Research Questions

### Core Questions

1. What functional areas does AEP-4789 Volume I define or describe that are relevant to Glaux Server?
2. Which Volume I functions map to direct Glaux Server responsibilities?
3. Which Volume I functions require server-side contracts for other Glaux components without making those components part of Glaux Server?
4. Which Volume I functions are ecosystem-level, operational, profile-level, or out of scope for Glaux Server?
5. What downstream IDR topics must receive specific handoffs from the Volume I functional mapping?

### Detailed Questions

#### Functional Area Identification

- How does AEP-4789 Volume I define, describe, or organize sensor-system interoperability functions?
- What functions are explicitly named, and what functions are implied through reference-view descriptions, architecture language, workflow descriptions, or operational examples?
- Which functions appear repeatedly across Volume I and therefore need special attention?
- Which functions depend on metadata, dynamic data, command/tasking, events, security, or status behavior?
- Which functions require terminology alignment in `IDR-SRV-004`?

#### Discovery and Navigation

- What does Volume I imply about discovering systems, sensors, platforms, deployments, services, data streams, events, status, and tasking interfaces?
- What discovery functions are likely direct server responsibilities?
- What discovery functions are client-side, publisher-side, catalog-side, or ecosystem-level responsibilities?
- What server behaviors are implied for landing pages, resource navigation, links, collections, identifiers, and queryable resources?
- Which discovery findings should inform CSAPI Part 1 requirement extraction?

#### Registration and Description

- What does Volume I imply about registering or describing systems, sensors, platforms, procedures, deployments, capabilities, properties, and context?
- What must Glaux Server store, expose, validate, link, or govern?
- What should remain publisher-side, simulator-side, or administrative workflow scope?
- What description requirements point toward SensorML, SWE Common, semantic binding, provenance, validity, or lineage research?
- Which description findings should inform the canonical resource model and metadata/document strategy topics?

#### Access and Exchange

- What access and exchange functions are described or implied by Volume I?
- What server responsibilities arise for resource retrieval, observation access, dynamic data access, historical queries, structured exchange, content negotiation, filtering, and validation?
- Which access/exchange functions depend on CSAPI Part 1 or Part 2?
- What server-side contracts are required for publishers, simulators, clients, and external systems?
- What interoperability risks should be carried into later existing-implementation and conformance research?

#### Streaming and Dynamic Data

- What does Volume I imply about streaming, near-real-time data, time-varying information, updates, events, or subscriptions?
- Which dynamic-data functions belong directly to Glaux Server?
- Which functions depend on broker, pub/sub, or edge components outside the server boundary?
- What server-side semantics are required for freshness, ordering, replay, late-arriving data, or stale state?
- Which findings should inform `IDR-SRV-034`, `IDR-SRV-035`, and performance/streaming test strategy?

#### Tasking and Control

- What does Volume I imply about tasking, control, commands, feasibility, authorization, lifecycle status, and interaction with connected systems?
- Which tasking/control functions are direct Glaux Server responsibilities?
- Which functions require integration contracts with publishers, proxies, actuators, or external mission systems?
- What safety, audit, authorization, and accountability implications arise?
- Which findings should inform `IDR-SRV-036`, `IDR-SRV-037`, `IDR-SRV-038`, and `IDR-SRV-055`?

#### Status and Availability

- What does Volume I imply about system status, health, availability, operational state, degraded state, and last-known state?
- Which status/availability functions must the server expose or preserve?
- Which functions require system-event modeling?
- What status semantics are needed for DDIL-informed or constrained operations?
- Which findings should inform `IDR-SRV-020`, `IDR-SRV-034`, and `IDR-SRV-042`?

#### Security, Federation, and DDIL

- What Volume I functions depend on authentication, authorization, trust, federation, cross-boundary information sharing, or releasability?
- What functions must support differently accredited, coalition, or national environments?
- What functions must account for DDIL or constrained communication conditions?
- What server-side implications exist for access governance, auditing, synchronization, conflict handling, freshness, and delayed updates?
- Which findings should inform `IDR-SRV-039` through `IDR-SRV-043`?

---

## 3. Primary Resources

The future research report must analyze these sources directly.

### Project and Governance Sources

- Glaux Server Overall IDR Research Plan:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- `IDR-SRV-001` research plan:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline.md`
- `IDR-SRV-001` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md`
- Glaux Server Goal and Definition:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Template:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md

### AEP / STANAG Source Material

- Project-controlling ratification package: `AC/224(JCGISR)D(2026)0005`, dated 27 April 2026.
  - Status for this IDR: most-current ratification draft, as confirmed by the Glaux project lead on 29 July 2026.
  - SHA-256: `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`.
  - Use AEP-4789 Volume I, Edition A, Version 1 as the controlling source for this topic.
  - Use STANAG 4789, Edition 1 as parent context and AEP-4789 Volume II, Edition A, Version 1 as a supporting alignment check.
  - The local working copy is supplied by the project lead and is not stored in the public repository.
  - Cite the enclosing document, enclosed publication, and exact page/section in the report.

### OGC Standards Package Sources

Use these as supporting technical alignment references when a Volume I function clearly points to the adopted standards package.

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

---

## 4. Supporting Resources

Use these sources to interpret context, downstream dependencies, expected research depth, and reporting style.

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
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans
- Blueprint-first exemplar:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md
- Inventory/sourcing rigor exemplar:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/15-fixture-sourcing-organization.md
- End-state synthesis exemplar:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/38-testing-playbook-synthesis.md
- OS4CSAPI discussions, for background only:
  - https://github.com/orgs/OS4CSAPI/discussions

---

## 5. Research Methodology

### Phase 1: Source Collection and Volume I Orientation (1.5-2 hours)

**Objective:** Establish the source base and understand how AEP-4789 Volume I is organized.

**Tasks:**

1. Gather the current AEP-4789 Volume I source material available to the project team.
2. Record title, version, date, status, source location, and access limitations.
3. Review the Volume I table of contents, section structure, figures, tables, and reference-view organization.
4. Identify the sections most likely to contain server-relevant functional language.
5. Review `IDR-SRV-001` outputs, if available, and note obligation-boundary findings that constrain this topic.

**Expected Output:** Source inventory and Volume I orientation notes identifying high-value sections for functional extraction.

### Phase 2: Functional Area Extraction (2.5-3.5 hours)

**Objective:** Extract all Volume I functions relevant to Glaux Server.

**Tasks:**

1. Identify explicit and implied functional areas in AEP-4789 Volume I.
2. Extract references to discovery, registration/description, access/exchange, streaming/dynamic data, tasking/control, status/availability, security, federation, and DDIL-informed behavior.
3. Capture section, clause, figure, or table anchors for each extracted function.
4. Note language that identifies operational context, actor roles, environments, constraints, or assumptions.
5. Identify repeated terms, ambiguous terms, and terms that should feed the terminology crosswalk in `IDR-SRV-004`.

**Expected Output:** Functional extraction inventory with source anchors and initial category tags.

### Phase 3: Function-to-Server Responsibility Mapping (3-4 hours)

**Objective:** Convert extracted Volume I functions into a bounded Glaux Server responsibility map.

**Tasks:**

1. Classify each extracted function as:
   - direct Glaux Server responsibility,
   - server-side integration contract responsibility,
   - ecosystem responsibility,
   - adjacent-standard or future-profile consideration,
   - out-of-scope for Glaux Server.
2. For each direct server responsibility, identify affected server design areas:
   - API behavior,
   - resource/data model,
   - storage/query behavior,
   - dynamic-data behavior,
   - tasking behavior,
   - status/event behavior,
   - security model,
   - DDIL-informed semantics,
   - conformance or verification strategy.
3. For each server-side contract responsibility, identify the affected Glaux component or external system without designing that component.
4. Identify unclear mappings and document why they remain unresolved.
5. Flag any findings that appear to change or refine the Glaux Server Goal and Definition.

**Expected Output:** Function-to-server responsibility matrix and boundary classification notes.

### Phase 4: Downstream Handoff and Traceability Design (1.5-2 hours)

**Objective:** Prepare findings so downstream IDR topics can use them directly.

**Tasks:**

1. Map each functional finding to one or more downstream IDR topics.
2. Identify which findings should be handed to `IDR-SRV-003`, `IDR-SRV-004`, `IDR-SRV-006`, `IDR-SRV-007`, and later server architecture topics.
3. Identify test-strategy implications for `IDR-SRV-050` through `IDR-SRV-056`.
4. Define a recommended traceability format for carrying Volume I functional mappings into later topic reports.
5. Identify unresolved questions requiring later source review, standards interpretation, or project decision.

**Expected Output:** Downstream handoff matrix, traceability recommendations, and unresolved issue list.

### Phase 5: Standards-Package Alignment Check (1-1.5 hours)

**Objective:** Confirm whether Volume I functional mappings appear to align with the adopted Volume II standards package.

**Tasks:**

1. Compare mapped functions against the OGC Connected Systems overview and adopted standards-package concepts.
2. Identify which functions appear likely to be implemented through CSAPI Part 1, CSAPI Part 2, SensorML, or SWE Common.
3. Identify functions that do not clearly map to the Volume II standards package and require later investigation.
4. Note implications for `IDR-SRV-003`.
5. Avoid detailed CSAPI requirement extraction, which belongs to later topics.

**Expected Output:** Lightweight standards-package alignment notes and gap list.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert research output into a decision-usable function-to-responsibility baseline.

**Tasks:**

1. Consolidate evidence, extracted functions, mappings, and boundary classifications.
2. Resolve conflicts and ambiguities where possible.
3. Produce findings grouped by functional area.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] AEP-4789 Volume I, Edition A, Version 1 from the project-controlling package has been reviewed.
- [ ] Source documents used in the report are listed with title, version/date, URL/path, status, and authority classification.
- [ ] Volume I functions relevant to Glaux Server have been extracted with source anchors.
- [ ] Each relevant function is classified as direct server responsibility, server-side contract responsibility, ecosystem responsibility, adjacent/future-profile consideration, or out-of-scope.
- [ ] Discovery, registration/description, access/exchange, streaming/dynamic data, tasking/control, status/availability, security/federation, and DDIL-informed behavior are all assessed.
- [ ] Downstream handoffs to later IDR topics are identified.
- [ ] Test-strategy implications are captured at a high level.
- [ ] Unresolved boundary questions and interpretation risks are explicitly listed.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** AEP-4789 Volume I Functional Mapping to Server Responsibilities Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. AEP-4789 Volume I functional extraction findings
5. Function-to-server responsibility matrix
6. Boundary classification findings
7. Standards-package alignment notes
8. Downstream topic handoff matrix
9. Test-strategy implications
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
- The project-controlling `AC/224(JCGISR)D(2026)0005` package must be available to the researcher. If it is unavailable, this topic is blocked.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-003: AEP-4789 Volume II Standards Package Implementation Baseline`
- `IDR-SRV-004: Terminology and Concept Crosswalk`
- `IDR-SRV-006: CSAPI Part 1 Requirement Baseline`
- `IDR-SRV-007: CSAPI Part 2 Requirement Baseline`
- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-020: Status, Availability, and System Event Model`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-042: DDIL-Informed Server Semantics`
- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
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

**Actual Research Time:** Approximately 0.75 hours of AI-assisted elapsed execution time on July 30, 2026<br>
**Completion Date:** July 31, 2026

---

## 10. Notes and Open Questions

- The report classifies broad operational and architectural language rather than converting every functional outcome into a direct server requirement.
- Volume II/OGC interpretation questions were handed to later topics instead of being resolved prematurely here.
- Server-side contracts were identified without designing Publisher, Simulator, Web App, Mobile, gateways, identity services, or transports.
- Resolved: the registered `AC/224(JCGISR)D(2026)0005` package, verified against SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`, is the controlling Volume I copy for this IDR cycle.
- Resolved: Volume I formally defines `shall`, `should`, and `may`; lowercase `must` and other strong functional prose are not silently treated as convention-defined mandatory requirements, and representative workflows remain illustrative.
- Resolved for this topic: future-profile/SRD concerns are classified and assigned to later IDR topics rather than invented as current server obligations.
- Prerequisite satisfied: `IDR-SRV-001` is complete and accepted; no prerequisite exception was used.
- The polished report completed independent AEP source, OGC alignment, full-plan coverage, traceability, and local validation review and was accepted by the Glaux Project Lead on July 31, 2026.
- Risk: Over-mapping Volume I operational functions could cause Glaux Server to absorb ecosystem-level responsibilities.
- Risk: Under-mapping Volume I functions could cause later design topics to miss NATO/DGIWG operational intent.

---

## References

- Glaux Server Overall IDR Research Plan:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- `IDR-SRV-001` research plan:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline.md`
- Glaux Server Goal and Definition:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
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
- OS4CSAPI testing research-plan corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans

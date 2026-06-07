# Section 001: STANAG 4789 / AEP-4789 Server Obligation Baseline - Research Plan

**Status:** Planned  
**Last Updated:** June 7, 2026  
**Estimated Research Time:** 8-12 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md`

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

This topic research must identify and document the **direct server obligations** that flow from the STANAG 4789 / AEP-4789 framework into Glaux Server planning.

The research must answer:

- What does STANAG 4789 / AEP-4789 require or imply for a server-side implementation component?
- Which obligations are direct server responsibilities, which are ecosystem responsibilities, and which are out of scope for Glaux Server?
- How should these obligations constrain later Glaux Server research topics, especially CSAPI conformance, resource modeling, dynamic data, tasking, status, security, DDIL-informed behavior, and verification?

The output must be a server-obligation baseline with traceability anchors that later IDR topics can use as the authoritative starting point for Glaux Server design.

### Why This Topic Order

This is the first Glaux Server IDR topic because all downstream research depends on correctly understanding the NATO-level and AEP-level obligation boundary.

This topic unlocks:

- AEP-4789 Volume I functional mapping in `IDR-SRV-002`
- AEP-4789 Volume II standards-package baseline in `IDR-SRV-003`
- CSAPI Part 1 and Part 2 requirement extraction in `IDR-SRV-006` and `IDR-SRV-007`
- Security, DDIL, tasking, status, interoperability, and verification research in later categories

If this topic is incomplete or too vague, later research may incorrectly treat Glaux Server as either too narrow (only a generic CSAPI endpoint server) or too broad (the whole Glaux ecosystem).

### Critical Constraint(s)

- Treat STANAG 4789 / AEP-4789 as the controlling NATO/DGIWG obligation frame for this topic.
- Keep conclusions bounded to **Glaux Server** responsibilities.
- Do not define Glaux Web App, Glaux Mobile, Glaux Publisher, or Glaux Simulator product behavior except where the server must expose contracts they depend on.
- Do not replace authoritative NATO, DGIWG, or OGC standards with project-specific definitions.
- Do not collapse the research into an MVP or reduced-scope implementation.
- Clearly distinguish:
  - direct server obligations,
  - server-side integration contract obligations,
  - ecosystem obligations,
  - adjacent-standard interoperability considerations,
  - out-of-scope items.
- Use direct source references and traceability anchors wherever possible. If a source is controlled or not publicly linkable, identify the source title, version/date if available, and repository or project storage location when known.

---

## 2. Research Questions

### Core Questions

1. What direct server obligations does STANAG 4789 / AEP-4789 impose or imply for Glaux Server?
2. Which STANAG 4789 / AEP-4789 functions must be treated as server responsibilities, server-side contracts, ecosystem responsibilities, or out-of-scope concerns?
3. How do the STANAG 4789 / AEP-4789 concepts of discovery, description, access, exchange, streaming, status, availability, and tasking translate into server obligation categories?
4. What does the STANAG 4789 / AEP-4789 framework imply for security, authorization, trust, federation, coalition use, and DDIL-informed server behavior?
5. What traceability structure should downstream IDR topics use so later implementation decisions remain aligned to the NATO/DGIWG obligation baseline?

### Detailed Questions

#### Standards and Authority

- What is the role of STANAG 4789 as the controlling standardization frame for Glaux Server?
- What is the role of AEP-4789 Volume I in defining operational functions, reference views, and interoperability needs?
- What is the role of AEP-4789 Volume II in adopting the core APIs and encodings package?
- Which requirements, functions, assumptions, or terms appear to be normative, informative, or implementation-guidance oriented?
- Which obligation sources are authoritative, and which are supporting or explanatory?

#### Server Responsibility Boundary

- Which obligations clearly belong to Glaux Server?
- Which obligations belong to the broader Glaux ecosystem but require server-side support?
- Which obligations should be assigned to Glaux Publisher, Glaux Simulator, Glaux Web App, or Glaux Mobile instead of Glaux Server?
- Which obligations are outside Glaux Server scope unless assigned later through an approved planning artifact, AEP volume, SRD, or project decision?
- What boundary language should downstream topics reuse to avoid scope creep?

#### Functional Obligation Mapping

- How should discovery and navigation obligations be represented as server responsibilities?
- How should registration and description obligations be represented as server responsibilities?
- How should access and exchange obligations be represented as server responsibilities?
- How should streaming and dynamic-data obligations be represented as server responsibilities?
- How should tasking and control obligations be represented as server responsibilities?
- How should status and availability obligations be represented as server responsibilities?
- How should provenance, validity, freshness, and trust obligations be represented as server responsibilities?

#### Operational Environment

- What does STANAG 4789 / AEP-4789 imply for NATO, national, coalition, federated, tactical, and DDIL-informed operating contexts?
- What server obligations arise from cross-organizational and differently accredited environments?
- What obligations relate to degraded, intermittent, or constrained connectivity?
- What obligations relate to last-known state, delayed updates, staged metadata enrichment, synchronization, or freshness?
- Which operational-environment obligations must be addressed in later architecture and verification topics?

#### Traceability and Downstream Use

- What traceability anchors should be captured for later IDR topics?
- What terms require a glossary or crosswalk in `IDR-SRV-004`?
- What findings should be handed directly to CSAPI requirement extraction topics?
- What findings should be handed directly to security, DDIL, tasking, status, and verification topics?
- What open questions must be carried forward?

---

## 3. Primary Resources

The future research report must analyze these sources directly.

### Project and Governance Sources

- Glaux Server Overall IDR Research Plan:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
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

### STANAG / AEP Source Material

- STANAG 4789 source material available to the Glaux project team.
  - Use the controlled or project-provided copy available in the working environment.
  - Record exact title, volume, version, date, and storage location in the report.
- AEP-4789 Volume I source material available to the Glaux project team.
  - Extract operational functions, reference-view concepts, interoperability needs, operational environments, and boundary conditions.
  - Record exact title, volume, version, date, and storage location in the report.
- AEP-4789 Volume II source material available to the Glaux project team.
  - Extract standards-package adoption language, server-relevant obligations, and references to CSAPI, SensorML, and SWE Common.
  - Record exact title, volume, version, date, and storage location in the report.

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
- OGC schema repository for Connected Systems and related encodings:
  - https://schemas.opengis.net/

---

## 4. Supporting Resources

Use these sources to interpret context, constraints, downstream dependencies, and expected reporting style.

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

---

## 5. Research Methodology

### Phase 1: Source Collection and Authority Classification (1.5-2 hours)

**Objective:** Establish the evidence base and classify sources by authority, relevance, and downstream use.

**Tasks:**

1. Gather the controlling STANAG 4789 / AEP-4789 source material available to the project team.
2. Record title, version, volume, date, status, source location, and access limitations for each STANAG/AEP source.
3. Gather the Glaux Server Goal and Definition and the overall IDR research plan.
4. Gather the official OGC standards-package sources listed above.
5. Classify each source as controlling, adopted technical standard, project governance, supporting context, or exemplar.

**Expected Output:** Evidence inventory with authority classification and notes on any unavailable, controlled, or ambiguous sources.

### Phase 2: STANAG / AEP Obligation Extraction (2-3 hours)

**Objective:** Extract server-relevant obligations, functions, assumptions, and constraints from STANAG 4789 / AEP-4789.

**Tasks:**

1. Identify all language that relates to discovery, registration, description, access, exchange, streaming, tasking, control, status, availability, security, interoperability, and operational environments.
2. Extract text references or clause/section anchors for each obligation or implication.
3. Classify each item as:
   - direct Glaux Server obligation,
   - server-side contract obligation,
   - ecosystem obligation,
   - adjacent interoperability consideration,
   - out-of-scope for Glaux Server.
4. Note whether each item appears normative, informative, explanatory, or implementation-guidance oriented.
5. Identify any repeated terms or concepts that should feed the terminology crosswalk in `IDR-SRV-004`.

**Expected Output:** Obligation extraction table with source anchors, classification, and initial interpretation.

### Phase 3: Server Boundary and Function Mapping (2-3 hours)

**Objective:** Translate extracted obligations into a bounded Glaux Server responsibility model.

**Tasks:**

1. Map extracted obligations to server capability areas:
   - discovery and navigation,
   - registration and description,
   - access and exchange,
   - streaming and dynamic data,
   - tasking and control,
   - status and availability,
   - security and authorization,
   - trust/provenance/validity/freshness,
   - DDIL-informed behavior,
   - validation and conformance.
2. Identify obligations that depend on the Volume II standards package rather than direct server-specific STANAG/AEP language.
3. Identify obligations that should be handed to later CSAPI, SensorML, SWE Common, security, DDIL, persistence, dynamic-data, and verification topics.
4. Identify obligations that require server-side contracts for Glaux Publisher, Simulator, Web App, or Mobile without assigning those components' implementation behavior to Glaux Server.
5. Identify any unclear or unresolved boundary issues.

**Expected Output:** Server responsibility map and downstream topic handoff matrix.

### Phase 4: OGC Standards-Package Alignment Check (1-2 hours)

**Objective:** Verify that the obligation baseline is consistent with the adopted OGC standards package and does not contradict authoritative technical standards.

**Tasks:**

1. Compare extracted STANAG/AEP server obligations against the OGC Connected Systems overview and standards-package framing.
2. Identify which obligations appear to be implemented primarily through CSAPI Part 1, CSAPI Part 2, SensorML, or SWE Common.
3. Identify any obligations that are not obviously covered by the Volume II standards package and require later interpretation.
4. Record implications for `IDR-SRV-003`, `IDR-SRV-006`, `IDR-SRV-007`, and later verification topics.
5. Note any implementation risks caused by mismatch between operational obligation language and technical standards language.

**Expected Output:** Standards-package alignment notes and gap/interpretation list.

### Phase 5: Synthesis (1-2 hours)

**Objective:** Convert the research output into a decision-usable server-obligation baseline.

**Tasks:**

1. Consolidate evidence and extracted obligations.
2. Resolve conflicts and ambiguities where possible.
3. Produce concise findings grouped by research question.
4. Produce recommendations for how downstream IDR topics should use the obligation baseline.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] All controlling STANAG 4789 / AEP-4789 source material available to the project team has been reviewed or explicitly marked unavailable.
- [ ] All source documents used in the report are listed with title, version/date, URL/path, access date, and authority classification.
- [ ] Direct server obligations are extracted and traced to source anchors.
- [ ] Server-side contract obligations are distinguished from broader ecosystem obligations.
- [ ] Out-of-scope concerns are explicitly identified where relevant.
- [ ] Discovery, registration/description, access/exchange, streaming/dynamic data, tasking/control, status/availability, security/trust, and DDIL-informed behavior are all assessed.
- [ ] Downstream topic dependencies and handoffs are identified.
- [ ] Unresolved questions or interpretation risks are explicitly listed.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** STANAG 4789 / AEP-4789 Server Obligation Baseline Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. STANAG 4789 / AEP-4789 obligation extraction findings
5. Server responsibility classification
6. Standards-package alignment notes
7. Downstream topic handoff matrix
8. Recommendations
9. Implementation implications
10. Risks, constraints, and open questions
11. Validation against this plan's success criteria
12. References

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- STANAG 4789 / AEP-4789 source material must be available to the researcher or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-002: AEP-4789 Volume I Functional Mapping to Server Responsibilities`
- `IDR-SRV-003: AEP-4789 Volume II Standards Package Implementation Baseline`
- `IDR-SRV-004: Terminology and Concept Crosswalk`
- `IDR-SRV-005: Related NATO Standards Boundary Review`
- `IDR-SRV-006: CSAPI Part 1 Requirement Baseline`
- `IDR-SRV-007: CSAPI Part 2 Requirement Baseline`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-042: DDIL-Informed Server Semantics`
- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
- Final overall IDR synthesis report

---

## 9. Research Status Checklist

Update this section as work progresses.

- [ ] Phase 1 complete
- [ ] Phase 2 complete
- [ ] Phase 3 complete
- [ ] Phase 4 complete
- [ ] Phase 5 synthesis complete
- [ ] Deliverable draft complete
- [ ] Deliverable reviewed
- [ ] Deliverable accepted

**Actual Research Time:** TBD until complete  
**Completion Date:** TBD until complete

---

## 10. Notes and Open Questions

- The availability and citation handling for STANAG 4789 / AEP-4789 source material may depend on access controls. If a source cannot be linked publicly, the report must still record enough internal identifying information to support review.
- The research must avoid treating broad operational interoperability goals as direct Glaux Server implementation obligations unless the source evidence supports that interpretation.
- The research should identify where later topics must perform more detailed technical extraction from CSAPI, SensorML, SWE Common, and related OGC materials.
- Open question: Which specific STANAG 4789 / AEP-4789 source copies are authoritative for this IDR effort?
- Open question: Are any AEP/SRD materials still draft, unpublished, or expected to change during the Glaux Server planning cycle?
- Risk: If the obligation baseline is too broad, downstream topics may absorb ecosystem responsibilities into Glaux Server.
- Risk: If the obligation baseline is too narrow, downstream topics may miss NATO/DGIWG requirements that should shape the server architecture.

---

## References

- Glaux Server Overall IDR Research Plan:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
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
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans

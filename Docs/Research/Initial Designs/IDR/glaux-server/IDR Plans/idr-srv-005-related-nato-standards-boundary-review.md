# Section 005: Related NATO Standards Boundary Review - Research Plan

**Topic ID:** IDR-SRV-005<br>
**Status:** In Progress<br>
**Last Updated:** July 31, 2026<br>
**Estimated Research Time:** 8-12 hours<br>
**Actual Research Time:** Approximately 1.5 hours of AI-assisted elapsed execution time<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-005-related-nato-standards-boundary-review-report.md`

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

This topic research must identify and document the **boundary relationship between Glaux Server and related NATO standards** that may intersect with STANAG 4789 / AEP-4789 sensor interoperability work.

The purpose is not to expand Glaux Server into an implementation of adjacent NATO standards. The purpose is to determine:

- Which related NATO standards are relevant to Glaux Server planning.
- Which related NATO standards create interoperability boundary considerations.
- Which related NATO standards should be explicitly treated as out of scope for Glaux Server implementation.
- Which standards may require future integration, mediation, profile, SRD, or ecosystem-level treatment outside the Glaux Server core.
- Which findings must be carried forward into CSAPI, resource-model, security, dynamic-data, tasking, and interoperability research.

The output must be a boundary and interoperability-not-implementation baseline that helps keep Glaux Server full-scope within its own responsibilities without absorbing unrelated service families, data products, operational workflows, or mission-system standards.

### Why This Topic Order

This topic follows:

- `IDR-SRV-001: STANAG 4789 / AEP-4789 Server Obligation Baseline`
- `IDR-SRV-002: AEP-4789 Volume I Functional Mapping to Server Responsibilities`
- `IDR-SRV-003: AEP-4789 Volume II Standards Package Implementation Baseline`
- `IDR-SRV-004: Terminology and Concept Crosswalk`

Those topics establish the primary obligation, function, standards-package, and terminology baseline. This topic then reviews adjacent NATO standards only to define **boundaries**, not to redefine the server scope.

This topic should be completed before the detailed CSAPI behavior and conformance topics because later research must know whether adjacent NATO standards affect Glaux Server requirements directly, affect only interoperability context, or are out of scope.

### Critical Constraint(s)

- This is a boundary review, not a requirement extraction for all related NATO standards.
- Do not turn Glaux Server into an implementation of unrelated or adjacent NATO service families.
- Do not require Glaux Server to implement unrelated standards unless STANAG 4789 / AEP-4789 or an approved Glaux planning artifact explicitly assigns a server-side responsibility.
- Clearly classify each reviewed standard as:
  - directly relevant to Glaux Server,
  - interoperability boundary consideration,
  - ecosystem-level consideration,
  - future profile/SRD/AEP consideration,
  - out of scope for Glaux Server.
- Capture enough evidence to justify the classification, but do not attempt full conformance analysis of adjacent standards.
- Keep findings bounded to server obligations, API behavior, data/resource model, security model, dynamic-data behavior, tasking behavior, conformance strategy, deployment shape, or server-side integration contracts.

---

## 2. Research Questions

### Core Questions

1. Which related NATO standards should be reviewed because they may intersect with STANAG 4789 / AEP-4789 and Glaux Server?
2. Which related standards create direct Glaux Server implications, and which only define interoperability boundaries?
3. Which related standards must be explicitly kept out of Glaux Server implementation scope?
4. What boundary language should later IDR topics use to avoid accidental scope expansion?
5. What integration, mediation, or future-profile questions should be recorded for later ecosystem or project-level treatment?

### Detailed Questions

#### Related Standards Identification

- Which related NATO standards are mentioned or implied by STANAG 4789 / AEP-4789 source material?
- Which related standards appear in AEP references, annexes, background sections, or interoperability discussions?
- Which standards relate to sensor data, ISR products, geospatial services, metadata, imagery, tracks, video, UAS control, messages, or information exchange?
- Which standards are relevant only because Glaux Server may need to interoperate with systems that use them?
- Which related standards are irrelevant to Glaux Server and should not be carried forward?

#### Boundary Classification

- Does the related standard define a data product, service interface, exchange format, message format, profile, operational workflow, or system-of-systems concern?
- Does the related standard create a direct server obligation for Glaux Server?
- Does it create only an interoperability consideration?
- Does it belong more naturally to Glaux Publisher, Glaux Simulator, client applications, external systems, future profiles, or operational guidance?
- What evidence supports the classification?

#### Candidate Standards and Families

Review candidates such as the following where relevant and available:

- NATO geospatial and ISR standards referenced by STANAG 4789 / AEP-4789 materials.
- STANAG 4559 / NATO ISR Library interface concerns, where relevant.
- STANAG 4545 imagery-related product concerns, where relevant.
- STANAG 4607 GMTI-related product concerns, where relevant.
- STANAG 4609 motion imagery-related product concerns, where relevant.
- STANAG 4676 tracking-related product concerns, where relevant.
- STANAG 4586 UAS control-related concerns, where relevant.
- STANAG 2103 reporting or message-related concerns, where relevant.
- STANAG 7149 or other metadata / geospatial / ISR context standards, where relevant.
- Any other NATO standard explicitly referenced by STANAG 4789 / AEP-4789 source material.

This list is a starting point, not a scope-expansion mandate. The report must only analyze standards that are available and relevant enough to affect Glaux Server boundary decisions.

#### Server Implications

- Could the related standard affect Glaux Server resource identity, metadata, linking, provenance, security, classification/releasability, or data exchange behavior?
- Could it affect dynamic-data, tasking, status, or event semantics?
- Could it affect persistence, query, content negotiation, encoding, or validation strategy?
- Could it affect external-client interoperability testing?
- Does it imply a server-side integration hook, or does it belong entirely outside Glaux Server?

#### Out-of-Scope Controls

- Which standards should be explicitly listed as not implemented by Glaux Server?
- Which standards may be supported indirectly through integration, linking, metadata, or publisher-mediated workflows?
- Which standards require future profile/SRD/AEP work rather than server implementation?
- Which standards risk causing scope creep if not clearly bounded?
- What wording should be handed to later topics and planning documents?

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
- `IDR-SRV-004` research plan:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/idr-srv-004-terminology-and-concept-crosswalk.md`
- `IDR-SRV-004` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-004-terminology-and-concept-crosswalk-report.md`

### STANAG / AEP Source Material

- Project-controlling ratification package: `AC/224(JCGISR)D(2026)0005`, dated 27 April 2026.
  - Status for this IDR: most-current ratification draft, as confirmed by the Glaux project lead on 29 July 2026.
  - SHA-256: `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`.
  - Use STANAG 4789, Edition 1 and AEP-4789 Volumes I and II, Edition A, Version 1.
  - Use their explicit normative and informative references to bound the related-standards review.
  - The local working copy is supplied by the project lead and is not stored in the public repository.
  - Cite the enclosing document, enclosed publication, and exact page/section in the report.

### Related NATO Standards and Standards Catalog Sources

Use available official, project-provided, or authoritative sources for candidate related NATO standards. Where source material is controlled or unavailable, record the limitation.

- NATO Standardization Office public standards information:
  - https://nso.nato.int/
- NATO Standardization Document Database, if accessible to the researcher:
  - https://nso.nato.int/nso/nsdd/
- DGIWG standards and profile references:
  - https://dgiwg.org/
- Any project-provided copies or citations for related STANAGs referenced by STANAG 4789 / AEP-4789 source material.
- Official or authoritative public information for candidate standards, where available.

### OGC Standards Package Sources for Boundary Context

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
- OS4CSAPI discussions, for background only:
  - https://github.com/orgs/OS4CSAPI/discussions
- OS4CSAPI client work, for later interoperability context only:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability findings repository, for later interoperability context only:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd

---

## 5. Research Methodology

### Phase 1: Source Collection and Candidate Standard Identification (1.5-2 hours)

**Objective:** Identify the related NATO standards that should be reviewed for Glaux Server boundary implications.

**Tasks:**

1. Review STANAG 4789 / AEP-4789 source material for explicit references to related NATO standards.
2. Review prior IDR topic outputs, if available, for adjacent-standard mentions.
3. Create a candidate list of related standards and source locations.
4. Record source availability for each candidate standard:
   - available for direct review,
   - publicly summarized only,
   - project-provided controlled source,
   - unavailable.
5. Remove candidates that have no plausible server-boundary relevance.

**Expected Output:** Candidate related-standards inventory with availability and relevance notes.

### Phase 2: Boundary-Relevant Standards Review (2.5-3.5 hours)

**Objective:** Review each candidate standard only deeply enough to classify its relationship to Glaux Server.

**Tasks:**

1. For each candidate standard, identify the type of standard:
   - service interface,
   - data product,
   - exchange format,
   - message format,
   - metadata profile,
   - operational workflow,
   - system control interface,
   - catalog/library interface,
   - other.
2. Identify whether the standard overlaps with Glaux Server concerns such as resource identity, metadata, dynamic data, tasking, events, status, geospatial context, security, or interoperability.
3. Identify whether the standard is referenced as complementary, adjacent, future-facing, or out of scope by STANAG/AEP material.
4. Capture source anchors for each classification where possible.
5. Note uncertainty where source evidence is incomplete.

**Expected Output:** Boundary review notes for each candidate standard.

### Phase 3: Classification and Scope-Control Matrix (2-3 hours)

**Objective:** Classify each reviewed standard and document what Glaux Server should or should not do.

**Tasks:**

1. Classify each standard as:
   - direct Glaux Server implication,
   - interoperability boundary consideration,
   - server-side integration hook consideration,
   - ecosystem-level consideration,
   - future profile/SRD/AEP consideration,
   - out of scope for Glaux Server.
2. For each classification, state the evidence and reasoning.
3. Identify whether the standard suggests any server-side metadata, link, reference, validation, or interoperability consideration.
4. Identify what Glaux Server should explicitly not implement.
5. Identify standards requiring project-level follow-up outside this server IDR.

**Expected Output:** Related standards boundary classification matrix.

### Phase 4: Downstream Handoff Analysis (1.5-2 hours)

**Objective:** Determine which later Glaux Server IDR topics need to consider boundary findings.

**Tasks:**

1. Map direct or indirect implications to downstream topics.
2. Identify handoffs to CSAPI requirement extraction, resource modeling, content negotiation, security, dynamic data, tasking, and interoperability testing topics.
3. Identify standards that should be mentioned only as out-of-scope examples in downstream topics.
4. Identify risks of accidental scope expansion.
5. Draft recommended boundary language for later research reports and planning documents.

**Expected Output:** Downstream handoff matrix and recommended scope-control language.

### Phase 5: OGC / AEP Standards-Package Boundary Check (1-1.5 hours)

**Objective:** Confirm that related NATO standards do not displace the AEP-4789 Volume II adopted standards package.

**Tasks:**

1. Compare boundary findings against the Volume II standards-package baseline from `IDR-SRV-003`, if available.
2. Confirm whether the core Glaux Server implementation baseline remains CSAPI Part 1, CSAPI Part 2, SensorML, and SWE Common.
3. Identify any related standard that may require linking, mediation, transformation, or metadata reference without becoming server implementation scope.
4. Identify any conflict between adjacent-standard expectations and the adopted OGC standards package.
5. Record unresolved conflicts for project decision.

**Expected Output:** Standards-package boundary confirmation and conflict list.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert research output into a decision-usable boundary and interoperability-not-implementation baseline.

**Tasks:**

1. Consolidate evidence, candidate reviews, classifications, handoffs, and scope-control findings.
2. Resolve conflicts and ambiguities where possible.
3. Produce findings grouped by standard or standard family.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] STANAG 4789 / AEP-4789 source material has been reviewed for related NATO standards references.
- [ ] Candidate related standards are listed with availability and relevance classification.
- [ ] Each reviewed standard is classified as direct implication, interoperability boundary, server-side integration hook, ecosystem-level concern, future profile/SRD/AEP concern, or out of scope.
- [ ] Evidence and reasoning are recorded for each classification.
- [ ] Standards that Glaux Server should not implement are explicitly identified.
- [ ] Server-side interoperability hooks, metadata/linking implications, or validation considerations are identified where relevant.
- [ ] Downstream topic handoffs are identified.
- [ ] Scope-expansion risks are documented.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] References are explicit and reproducible, including access limitations for controlled or unavailable sources.

---

## 7. Deliverable

**Deliverable Name:** Related NATO Standards Boundary Review Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-005-related-nato-standards-boundary-review-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Candidate related-standards inventory
5. Boundary review findings by standard or standard family
6. Classification matrix
7. Interoperability-not-implementation findings
8. Standards-package boundary confirmation
9. Recommended scope-control language
10. Downstream topic handoff matrix
11. Recommendations
12. Risks, constraints, and open questions
13. Validation against this plan's success criteria
14. References

The classification matrix should include, at minimum:

- Standard / document identifier
- Standard family or domain
- Source availability
- Relationship to STANAG 4789 / AEP-4789
- Relationship to Glaux Server
- Classification
- Evidence / source anchor
- Server implication, if any
- Explicit non-implementation boundary
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
- `IDR-SRV-004` research report should be complete or explicitly marked unavailable/deferred.
- The project-controlling `AC/224(JCGISR)D(2026)0005` package must be available to the researcher. If it is unavailable, this topic is blocked.
- Candidate related NATO standards must be available, summarized, or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-006: CSAPI Part 1 Requirement Baseline`
- `IDR-SRV-007: CSAPI Part 2 Requirement Baseline`
- `IDR-SRV-008: Conformance Class and Requirement Mapping`
- `IDR-SRV-014A` through `IDR-SRV-014G` implementation, interoperability, and lessons-learned studies
- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-032: Publisher-to-Server Contract Boundary`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints`
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

**Actual Research Time:** Approximately 1.5 hours of AI-assisted elapsed execution time<br>
**Completion Date:** Research completed July 31, 2026; topic completion pending plan-owner acceptance

---

## 10. Notes and Open Questions

- Some NATO standards may not be publicly accessible. The report must record access limitations rather than inventing unsupported detail.
- This topic must not become a general NATO standards literature review. It is limited to Glaux Server boundary implications.
- Candidate standards should be reviewed only to the depth needed to classify server relationship and boundary.
- Open question: Which related NATO standards are explicitly referenced by the current STANAG 4789 / AEP-4789 source set?
- Open question: Which related standards should be treated as future SRD/profile concerns rather than Glaux Server implementation concerns?
- Open question: Are any adjacent standards likely to require server-side metadata links, references, or transformations?
- Risk: Over-reading adjacent standards could cause Glaux Server to absorb unrelated implementation obligations.
- Risk: Under-reading adjacent standards could cause later interoperability or policy issues.
- Risk: Controlled-source limitations may prevent full verification of some candidate standards.

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
- `IDR-SRV-004` research plan:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/idr-srv-004-terminology-and-concept-crosswalk.md`
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
- NATO Standardization Office:
  - https://nso.nato.int/
- NATO Standardization Document Database:
  - https://nso.nato.int/nso/nsdd/
- DGIWG:
  - https://dgiwg.org/
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
- OS4CSAPI testing research-plan corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans

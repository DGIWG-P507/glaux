# Section 014F: SECD Interoperability Findings Study - Research Plan

**Topic ID:** IDR-SRV-014F<br>
**Status:** Planned  
**Last Updated:** July 30, 2026<br>
**Estimated Research Time:** 11.5-15 hours<br>
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014f-secd-interoperability-findings-study-report.md`

---

## Usage Instructions

Before executing this plan, review the exemplar corpus and align the report to the proven standards for content nature, type, level, and detail:

- Full research-plans corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- Blueprint-first exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md
- Inventory/sourcing rigor exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/15-fixture-sourcing-organization.md
- End-state synthesis exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/38-testing-playbook-synthesis.md

This plan is limited to research planning. It does not execute the research and does not produce the research report.

---

## 1. Research Objective

This topic research must study the **SECD interoperability findings, implementation observations, compatibility notes, and cross-implementation lessons** identified in the SECD interoperability work, especially findings that affect Glaux Server API behavior, conformance, validation, interoperability, and test strategy.

The research must answer:

- What interoperability issues, compatibility observations, and implementation differences were identified through SECD-related testing and analysis?
- Which findings reflect SECD implementation behavior, client behavior, standards ambiguity, fixture/data issues, OpenAPI/schema gaps, documentation gaps, or broader CSAPI interoperability risks?
- Which findings should influence Glaux Server API behavior, resource modeling, schema validation, conformance testing, fixture strategy, and external-client interoperability testing?
- Which SECD interoperability findings are server-specific, and which point to broader cross-implementation patterns involving OSH, Connected Systems Go, pygeoapi, OS4CSAPI clients, or CSAPI Explorer?
- How should the findings be converted into actionable Glaux Server research handoffs without treating one implementation's behavior as normative?

The output must be a SECD interoperability findings baseline with source anchors, evidence classification, root-cause classification, server-impact analysis, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows:

- `IDR-SRV-014A: OSH CSAPI Server Implementation Study`
- `IDR-SRV-014B: Connected Systems Go CSAPI Server Implementation Study`
- `IDR-SRV-014C: pygeoapi CSAPI Server Implementation Study`
- `IDR-SRV-014D: SECD CSAPI Server Implementation Study`
- `IDR-SRV-014E: OS4CSAPI Client Smoke Test Findings Study`

Those topics study implementation behavior and client-observed smoke-test findings. This topic focuses specifically on SECD interoperability findings and compatibility lessons so Glaux Server can account for practical cross-implementation issues before proceeding to canonical resource modeling, validation, conformance harness design, fixture corpus planning, and interoperability test matrix development.

This topic is distinct from `IDR-SRV-014D`, which studies SECD server implementation behavior. This topic studies interoperability findings, compatibility observations, and cross-implementation lessons associated with SECD.

### Critical Constraint(s)

- Treat SECD interoperability findings as evidence, not as normative requirements by themselves.
- Treat OGC API - Connected Systems Part 1 and Part 2, AEP-4789 Volume II adoption context, and the Glaux Server IDR standards baseline as controlling where interoperability findings conflict with standards.
- Distinguish SECD server behavior from external-client behavior, test-harness behavior, fixture issues, schema issues, and standards ambiguity.
- Distinguish reproducible findings from anecdotal observations, stale results, exploratory notes, inferred conclusions, and unresolved hypotheses.
- Distinguish SECD-specific issues from broader CSAPI interoperability concerns.
- Treat `IDR-SRV-014D` as the baseline for observable server behavior and explicitly declared architecture. Do not repeat its endpoint inventory except to establish the exact test condition for an interoperability finding.
- Treat the SECD interoperability repository as test and findings evidence, not as SECD server source. Reproduce drift-sensitive findings against the current deployment and classify each as current, changed, resolved, stale, or not reproducible.
- Do not convert findings directly into engineering tickets; translate them into research findings, risks, design implications, validation needs, and test strategy handoffs.
- Keep the research bounded to Glaux Server relevance.

---

## 2. Research Questions

### Core Questions

1. What SECD interoperability findings are relevant to Glaux Server?
2. Which findings reflect SECD implementation behavior, standards ambiguity, external-client assumptions, fixture/data issues, schema/OpenAPI gaps, or test-harness limitations?
3. Which Glaux Server API behavior areas are most affected by the findings?
4. What validation, conformance, fixture, golden-file, and interoperability tests should be derived from the findings?
5. What downstream topics should receive each finding as a handoff?

### Detailed Questions

#### Source and Evidence Baseline

- Which SECD interoperability repositories, branches, commits, issues, PRs, notes, test outputs, logs, scripts, fixtures, and discussions contain relevant findings?
- Which tested server version, client version, branch, commit, endpoint, and test condition produced each finding?
- Which findings are reproducible, partially reproducible, anecdotal, stale, superseded, or unresolved?
- What exact evidence supports each finding?
- What source material must be preserved for later traceability?

#### Endpoint and Resource Interoperability Findings

- What landing page, API definition, conformance declaration, collection, item, link, URI, and navigation interoperability issues were observed?
- What resource families were affected: systems, procedures, deployments, sampling features, properties, datastreams, observations, control streams, commands, feasibility, status, and events?
- Which findings indicate server gaps, implementation-specific interpretation, client assumptions, missing links, inconsistent identifiers, or non-canonical resource paths?
- Which findings should inform canonical resource model, identifier strategy, relationship model, and link behavior topics?

#### Query, Representation, and Error Findings

- What issues were observed with query parameters, filters, pagination, sorting, selection, temporal filters, bounding boxes, counts, or continuation links?
- What issues were observed with content negotiation, media types, encodings, response shapes, schema references, SensorML, SWE Common, GeoJSON, JSON, or OpenAPI descriptions?
- What issues were observed with HTTP status codes, machine-readable errors, unsupported operations, validation errors, or ambiguous failures?
- Which findings should inform query, content negotiation, error model, OpenAPI, and schema validation topics?

#### Dynamic Data and Tasking Findings

- What interoperability findings relate to observations, datastreams, latest values, time series, status, system events, control streams, commands, feasibility, command status, or tasking flows?
- Which behavior differences are server-specific versus broader interoperability concerns?
- Which findings indicate ambiguity in CSAPI Part 2, schemas, OpenAPI artifacts, or implementation expectations?
- Which findings should inform dynamic-data, streaming, command lifecycle, feasibility, and command-security topics?

#### Cross-Implementation and Client Compatibility Findings

- Which SECD findings also appear in OSH, Connected Systems Go, pygeoapi, OS4CSAPI client smoke tests, or OS4CSAPI discussions?
- Which findings are unique to SECD?
- Which findings reveal likely external-client expectations that Glaux Server should satisfy?
- Which findings identify useful comparison cases for the future interoperability matrix?
- Which findings should be handed to `IDR-SRV-014G` and `IDR-SRV-056`?

#### Test Strategy and Fixture Findings

- What SECD interoperability tests were most useful?
- Which tests were brittle, misleading, too implementation-specific, or insufficient?
- What positive, negative, conformance, schema, OpenAPI, fixture, golden-file, regression, and external-client tests should Glaux Server derive from the findings?
- What fixture data or scenario corpus entries are needed to reproduce important findings?
- Which findings should become explicit conformance-harness or CI checks after validation?

---

## 3. Primary Resources

The future research report must analyze these sources directly.

### Project and Governance Sources

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md

### Prior IDR Topic Sources

- `IDR-SRV-006` through `IDR-SRV-014E` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### SECD Interoperability Sources

- SECD interoperability evidence repository, currently access-controlled and not the SECD server source repository:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd/tree/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0
- Primary evaluation report:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/evaluation/csapi-server-evaluation-results.md
- Independent secondary evaluation:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/evaluation/csapi-server-evaluation-results-codex.md
- Adjudicated evidence register:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/reinvestigation/adjudication/pre-grading-evidence-register-v1.md
- SECD report card publication candidate:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/report-cards/publication-ready/secd-report-card.md
- Historical snapshot and initial compatibility report:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/analysis/00-server-snapshot.md
  - https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/analysis/01-initial-compatibility-report.md
- Any SECD-related interoperability reports, smoke-test outputs, manual test notes, screenshots, logs, or issue comments identified during the research phase.
- SECD implementation-study findings from `IDR-SRV-014D`.

### Related Implementation and Client Sources

Use these to compare SECD findings against other implementation and client evidence:

- OSH / OpenSensorHub sources and `IDR-SRV-014A`
- Connected Systems Go sources and `IDR-SRV-014B`
- pygeoapi sources and `IDR-SRV-014C`
- OS4CSAPI Client Smoke Test Findings and `IDR-SRV-014E`
- OS4CSAPI client work:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- OS4CSAPI organization:
  - https://github.com/OS4CSAPI
- CSAPI Explorer:
  - https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions:
  - https://github.com/orgs/OS4CSAPI/discussions

### Controlling OGC and Standards-Package Sources

Use these to determine whether an interoperability finding reflects a standards problem, server issue, client issue, implementation-specific behavior, or ambiguity:

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html

---

## 4. Supporting Resources

Use these sources to interpret project context, downstream dependencies, expected research depth, and report style.

- DGIWG P5.07 Glaux main repository: https://github.com/DGIWG-P507/glaux
- Glaux project website: https://dgiwg-p507.github.io/glaux/
- DGIWG P5.07 GitHub organization: https://github.com/DGIWG-P507
- Glaux Server repository, if available or created: https://github.com/DGIWG-P507/glaux-server
- OS4CSAPI testing research-plan corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- OS4CSAPI discussions, for background only: https://github.com/orgs/OS4CSAPI/discussions

---

## 5. Research Methodology

### Phase 1: Source Inventory and Evidence Baseline (1.5-2 hours)

**Objective:** Establish the SECD interoperability source baseline.

**Tasks:**

1. Identify repository paths, branches, commits, issues, PRs, scripts, logs, notes, fixtures, sample responses, screenshots, and discussions containing SECD interoperability findings.
2. Record exact URLs, commit SHAs, dates, tested server versions, client versions, and test conditions where available.
3. Separate server implementation evidence, interoperability test evidence, client behavior evidence, fixture evidence, documentation notes, and informal observations.
4. Define the evidence classification scheme:
   - reproduced finding,
   - interoperability test output,
   - code-supported finding,
   - fixture-supported finding,
   - issue/discussion observation,
   - inferred finding,
   - stale/superseded finding,
   - unresolved.
5. Define the finding-to-Glaux impact matrix fields.

**Expected Output:** SECD interoperability evidence inventory.

### Phase 2: API Behavior Finding Extraction (2.5-3 hours)

**Objective:** Extract and classify SECD interoperability findings by API behavior area.

**Tasks:**

1. Review findings for landing page, conformance, API definition, collections, links, navigation, identifiers, query, content negotiation, errors, and documentation behavior.
2. Identify endpoint and resource compatibility problems.
3. Identify findings caused by SECD server behavior, client assumptions, standards ambiguity, test harness behavior, or fixture issues.
4. Identify server-specific and cross-implementation recurring findings.
5. Classify each finding by behavior area, evidence strength, likely root cause, and Glaux Server impact.

**Expected Output:** SECD API interoperability findings matrix.

### Phase 3: Schema, Resource Model, Dynamic Data, and Tasking Analysis (2-3 hours)

**Objective:** Identify deeper model, dynamic-data, and tasking implications.

**Tasks:**

1. Review findings related to schemas, resource shapes, links, SensorML, SWE Common, GeoJSON, JSON, observations, datastreams, status, events, control streams, commands, and feasibility.
2. Identify which findings reveal standards ambiguity, implementation gaps, client-model gaps, or fixture limitations.
3. Identify dynamic-data and tasking patterns that require special test coverage.
4. Map findings to resource-model, schema-validation, dynamic-data, streaming, command lifecycle, feasibility, and command-security topics.

**Expected Output:** Model/dynamic-data/tasking interoperability implication matrix.

### Phase 4: Cross-Implementation Comparison (2-2.5 hours)

**Objective:** Compare SECD findings against other implementation and client evidence.

**Tasks:**

1. Compare SECD findings against OSH, Connected Systems Go, pygeoapi, and OS4CSAPI client smoke-test findings.
2. Identify recurring interoperability patterns.
3. Identify SECD-specific quirks that Glaux Server should avoid.
4. Identify external-client expectations that Glaux Server should satisfy.
5. Identify findings needing follow-up in `IDR-SRV-014G` and `IDR-SRV-056`.

**Expected Output:** Cross-implementation comparison matrix.

### Phase 5: Test Strategy and Fixture Handoff Analysis (2-2.5 hours)

**Objective:** Convert SECD interoperability lessons into Glaux Server test strategy input.

**Tasks:**

1. Identify candidate conformance, integration, client-compatibility, golden-file, schema, fixture, negative, regression, and interoperability tests.
2. Identify fixtures and response examples needed to reproduce key findings.
3. Identify SECD-derived tests that should become Glaux Server CI checks after standards validation.
4. Identify brittle or implementation-specific tests that should not be adopted without refinement.
5. Map handoffs to conformance harness, traceability, fixture corpus, performance/security testing, and interoperability topics.

**Expected Output:** Test and fixture handoff matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert SECD interoperability findings into a decision-usable baseline.

**Tasks:**

1. Consolidate source inventory, findings, classifications, root-cause analysis, cross-implementation lessons, and test implications.
2. Produce findings grouped by behavior area and Glaux Server impact area.
3. Identify what Glaux Server should adopt, avoid, investigate further, validate, or test.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] SECD interoperability sources are identified with exact URLs, branches, commits, dates, tested servers, and test conditions where available.
- [ ] Findings are distinguished from SECD implementation behavior, client-library behavior, fixture issues, documentation gaps, and standards ambiguities.
- [ ] Findings are grouped by API behavior area and evidence strength.
- [ ] Reproducible, stale, inferred, and unresolved findings are distinguished.
- [ ] Cross-implementation recurring issues and SECD-specific quirks are identified.
- [ ] Schema/model, dynamic-data, tasking, documentation, validation, conformance, and interoperability implications are documented.
- [ ] Test and fixture recommendations are explicit.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** SECD Interoperability Findings Study Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014f-secd-interoperability-findings-study-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. SECD interoperability source inventory and evidence classification
4. Tested servers, client versions, branches, and test conditions
5. SECD API behavior interoperability findings
6. Schema, model, and representation findings
7. Dynamic-data, status, event, and tasking findings
8. Cross-implementation comparison findings
9. Client/test-harness limitation versus server-behavior classification
10. Standards ambiguity and documentation-gap findings
11. Test strategy and fixture implications
12. Lessons for Glaux Server
13. Downstream topic handoff matrix
14. Recommendations
15. Risks, constraints, and open questions
16. Validation against this plan's success criteria
17. References

The interoperability findings matrix should include, at minimum:

- Finding identifier
- Source URL / source anchor
- Evidence type
- Tested server / tested client
- Endpoint or behavior area
- Observed behavior
- Expected behavior or standard reference
- Likely root cause classification
- SECD-specific or cross-implementation classification
- Glaux Server implication
- Test or fixture recommendation
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-006` through `IDR-SRV-014E` research reports must be complete and accepted before starting unless an exception is approved and recorded under the overall-plan Governance Rules.
- The pinned SECD interoperability evidence and relevant test artifacts must be reachable where required. For any unavailable external artifact, record its identity, access attempt, affected questions, and resulting evidence limits; do not infer its contents.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-014G: OS4CSAPI Discussions Lessons-Learned Study`
- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-017: Relationship and Linkage Model`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
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

- SECD interoperability findings are evidence, not normative sources.
- The pinned interoperability repository is test/findings evidence, not SECD server source.
- This topic should not repeat the SECD implementation study except where needed to explain interoperability findings.
- Some findings may have been fixed or superseded; the report must record freshness and current relevance.
- Open question: Which SECD interoperability artifacts best represent the current baseline?
- Open question: Which findings can be reproduced against current server deployments?
- Open question: Which findings reveal genuine standards ambiguity versus implementation defects?
- Open question: Which SECD interoperability tests should become Glaux Server CI tests?
- Risk: Treating SECD-specific behavior as a standards obligation could distort the Glaux Server baseline.
- Risk: Ignoring SECD interoperability findings could cause avoidable client compatibility failures.
- Risk: Mixing stale findings with current findings without classification could produce misleading recommendations.

---

## References

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- SECD interoperability evidence repository at reviewed commit: https://github.com/Sam-Bolling/csapi-server-interop-secd/tree/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions
- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html

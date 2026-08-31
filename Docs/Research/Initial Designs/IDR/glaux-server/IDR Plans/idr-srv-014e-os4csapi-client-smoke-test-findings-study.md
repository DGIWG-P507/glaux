# Section 014E: OS4CSAPI Client Smoke Test Findings Study - Research Plan

**Status:** Accepted<br>
**Last Updated:** August 31, 2026<br>
**Estimated Research Time:** 10-14 hours  
**Actual Research Time:** Approximately 4.5 hours of AI-assisted elapsed execution<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md`

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

This topic research must study the **OS4CSAPI client smoke-test findings, compatibility observations, implementation gaps, and interoperability lessons** identified through the OS4CSAPI client work, especially findings that affect Glaux Server API behavior, conformance, validation, interoperability, and test strategy.

The research must answer:

- What did OS4CSAPI client smoke testing reveal about practical CSAPI server behavior, client expectations, endpoint consistency, resource traversal, schema compatibility, content negotiation, query behavior, error handling, and implementation gaps?
- Which findings are server-side obligations, client-library limitations, data-quality issues, documentation gaps, implementation-specific differences, test-fixture limitations, or unresolved standards-interpretation questions?
- What smoke-test findings should directly influence Glaux Server resource behavior, API behavior, validation strategy, conformance harness design, fixture/golden-file strategy, and interoperability test matrix?
- Which findings are specific to particular servers or test conditions, and which represent broader lessons for Glaux Server?
- How should these findings be converted into actionable Glaux Server research handoffs without prematurely turning them into engineering tickets?

The output must be an OS4CSAPI client smoke-test findings baseline with source anchors, evidence classification, server-impact analysis, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows the implementation studies for:

- `IDR-SRV-014A: OSH CSAPI Server Implementation Study`
- `IDR-SRV-014B: Connected Systems Go CSAPI Server Implementation Study`
- `IDR-SRV-014C: pygeoapi CSAPI Server Implementation Study`
- `IDR-SRV-014D: SECD CSAPI Server Implementation Study`

Those topics study server implementations individually. This topic studies client smoke-test findings across the OS4CSAPI client work so Glaux Server can account for practical interoperability and client-observed behavior before defining canonical resource model, schema validation, conformance harness, fixture corpus, and external-client testing strategy.

This topic is distinct from `IDR-SRV-014F: SECD Interoperability Findings Study`. This topic focuses on OS4CSAPI client smoke-test findings and compatibility observations, while `IDR-SRV-014F` focuses specifically on the SECD interoperability work.

### Critical Constraint(s)

- Treat smoke-test findings as evidence of implementation behavior, compatibility issues, and test concerns, not as normative requirements by themselves.
- Treat OGC API - Connected Systems Part 1 and Part 2, AEP-4789 Volume II adoption context, and the Glaux Server IDR standards baseline as controlling where smoke-test findings conflict with standards.
- Distinguish server-side problems from client-library problems.
- Distinguish reproducible findings from anecdotal observations, exploratory test results, stale results, and unresolved hypotheses.
- Distinguish findings by tested server, client version, branch, commit, test script, fixture, endpoint, date, and environment where possible.
- Do not convert findings directly into Glaux Server implementation tickets; translate them into research findings, risks, design implications, validation needs, and test strategy handoffs.
- Keep the research bounded to Glaux Server relevance.

---

## 2. Research Questions

### Core Questions

1. What OS4CSAPI client smoke-test findings are relevant to Glaux Server?
2. Which findings reflect server behavior, client-library behavior, data/fixture quality, test-harness limitations, or standards-interpretation questions?
3. Which Glaux Server API behavior areas are most affected by the smoke-test findings?
4. What validation, conformance, fixture, golden-file, and interoperability tests should be derived from these findings?
5. What downstream topics should receive each finding as a handoff?

### Detailed Questions

#### Source and Evidence Baseline

- Which OS4CSAPI repositories, branches, commits, discussions, issues, test outputs, logs, screenshots, notes, pull requests, and code-sprint artifacts contain smoke-test findings?
- Which client branch, commit, dependency version, and test configuration produced each finding?
- Which servers were tested, including OSH, Connected Systems Go, pygeoapi, SECD, or other CSAPI servers?
- Which findings are reproducible, partially reproducible, anecdotal, stale, unresolved, or superseded?
- What exact evidence supports each finding?

#### Client-Server Compatibility Findings

- What endpoint discovery, landing page, API definition, conformance declaration, collection discovery, resource traversal, or link-following issues were observed?
- What issues were observed with resource identifiers, URI structure, canonical URLs, related resources, and link relations?
- What issues were observed with query parameters, filters, sorting, pagination, time filters, bounding boxes, or response counts?
- What issues were observed with content negotiation, media types, encodings, schema references, or alternate representations?
- What issues were observed with error responses, status codes, unsupported operations, or ambiguous failures?

#### Schema, Model, and Representation Findings

- Which schemas, response shapes, fields, links, media types, or examples caused parsing, validation, typing, or compatibility issues?
- Which CSAPI resources had inconsistent or unexpected shapes?
- What issues were observed with SensorML, SWE Common, GeoJSON, JSON, observations, datastreams, commands, status, or events?
- Which findings indicate server gaps versus client-model gaps?
- Which findings should be handed to canonical resource model and schema validation topics?

#### Dynamic Data and Tasking Findings

- What smoke-test findings relate to datastreams, observations, historical data, latest values, status updates, system events, control streams, commands, feasibility, or command status?
- Which dynamic-data patterns worked across servers, and which failed or differed?
- Which command/tasking behaviors were unsupported, partial, inconsistent, or ambiguous?
- Which findings should be handed to dynamic-data, streaming, command lifecycle, feasibility, and command-security topics?

#### Interoperability and Cross-Implementation Findings

- Which smoke-test findings recur across multiple servers?
- Which findings are server-specific?
- Which findings reveal ambiguity in the standard, OpenAPI artifacts, schemas, examples, or implementation expectations?
- Which findings identify useful external-client expectations that Glaux Server should satisfy?
- Which findings should be compared against `IDR-SRV-014F` and `IDR-SRV-014G`?

#### Test Strategy and Fixture Findings

- What smoke tests were most useful?
- What smoke tests were brittle, misleading, or insufficient?
- What additional positive, negative, schema, conformance, golden-file, external-client, and regression tests are needed?
- What test data and fixtures are needed to reproduce important findings?
- Which findings should inform conformance harness, requirement-to-test traceability, fixture corpus, and interoperability matrix topics?

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

- `IDR-SRV-006` through `IDR-SRV-014D` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### OS4CSAPI Client and Smoke-Test Sources

- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- OS4CSAPI client testing research corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing
- OS4CSAPI client testing research plans:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- OS4CSAPI issues, pull requests, test scripts, smoke-test logs, fixture files, branch notes, and research reports relevant to CSAPI server smoke testing.
- Any OS4CSAPI code-sprint or discussion materials that record smoke-test findings.
- CSAPI Explorer, where findings relate to external client behavior:
  - https://ogc-csapi-explorer.pages.dev/

### Related Server Implementation Sources

Use these to classify findings by tested server where relevant:

- OSH / OpenSensorHub sources from `IDR-SRV-014A`
- Connected Systems Go sources from `IDR-SRV-014B`
- pygeoapi sources from `IDR-SRV-014C`
- SECD sources from `IDR-SRV-014D`
- SECD interoperability repository:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd

### Controlling OGC and Standards-Package Sources

Use these to determine whether a smoke-test finding reflects a standards problem, server issue, client issue, or ambiguity:

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
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
- OS4CSAPI discussions, for background only: https://github.com/orgs/OS4CSAPI/discussions

---

## 5. Research Methodology

### Phase 1: Source Inventory and Evidence Baseline (1.5-2 hours)

**Objective:** Establish the OS4CSAPI smoke-test source baseline.

**Tasks:**

1. Identify repositories, branches, commits, issues, PRs, test scripts, logs, reports, notes, fixtures, and discussions containing smoke-test findings.
2. Record exact URLs, commit SHAs, dates, tested servers, client versions, and test conditions where available.
3. Separate smoke-test outputs, test scripts, client-code behavior, server responses, documentation notes, issue comments, and informal observations.
4. Define the evidence classification scheme:
   - reproduced finding,
   - smoke-test output,
   - code-supported finding,
   - fixture-supported finding,
   - issue/discussion observation,
   - inferred finding,
   - stale/superseded finding,
   - unresolved.
5. Define the finding-to-Glaux impact matrix fields.

**Expected Output:** OS4CSAPI smoke-test evidence inventory.

### Phase 2: Client-Server Behavior Finding Extraction (2.5-3.5 hours)

**Objective:** Extract and classify smoke-test findings by API behavior area.

**Tasks:**

1. Review smoke-test findings for landing page, conformance, API definition, collections, links, navigation, query, content negotiation, errors, and documentation behavior.
2. Identify client-observed resource and representation problems.
3. Identify server-specific and cross-server recurring findings.
4. Identify findings caused by client-library assumptions or limitations.
5. Classify each finding by behavior area, server, evidence strength, and likely root cause.

**Expected Output:** Client-server behavior findings matrix.

### Phase 3: Schema, Model, Dynamic Data, and Tasking Analysis (2-3 hours)

**Objective:** Identify deeper server-model and dynamic-data implications.

**Tasks:**

1. Review findings related to schemas, resource shapes, model types, SensorML, SWE Common, GeoJSON, JSON, observations, datastreams, status, events, control streams, commands, and feasibility.
2. Identify which findings reveal standards ambiguity, implementation gaps, or client-model gaps.
3. Identify dynamic-data and tasking patterns that require special test coverage.
4. Map findings to resource-model, schema-validation, dynamic-data, streaming, and command lifecycle topics.

**Expected Output:** Model/dynamic-data/tasking implication matrix.

### Phase 4: Interoperability and Cross-Implementation Analysis (2-2.5 hours)

**Objective:** Identify broader interoperability lessons from smoke-test findings.

**Tasks:**

1. Compare findings across OSH, CS-GO, pygeoapi, SECD, and any other tested servers.
2. Identify recurring failure patterns.
3. Identify server-specific quirks that Glaux Server should avoid.
4. Identify external-client expectations that Glaux Server should satisfy.
5. Identify findings needing follow-up in `IDR-SRV-014F`, `IDR-SRV-014G`, and `IDR-SRV-056`.

**Expected Output:** Cross-implementation interoperability findings.

### Phase 5: Test Strategy and Fixture Handoff Analysis (2-2.5 hours)

**Objective:** Convert smoke-test lessons into Glaux Server test strategy input.

**Tasks:**

1. Identify candidate conformance, integration, client-compatibility, golden-file, schema, fixture, negative, and regression tests.
2. Identify fixtures and response examples needed to reproduce key findings.
3. Identify smoke tests that should become Glaux Server CI checks.
4. Identify brittle or misleading smoke tests that should not be adopted without refinement.
5. Map handoffs to conformance harness, traceability, fixture corpus, performance/security testing, and interoperability topics.

**Expected Output:** Test and fixture handoff matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert smoke-test findings into a decision-usable baseline.

**Tasks:**

1. Consolidate source inventory, findings, classifications, root-cause analysis, cross-implementation lessons, and test implications.
2. Produce findings grouped by behavior area and server-impact area.
3. Identify what Glaux Server should adopt, avoid, investigate further, validate, or test.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [x] OS4CSAPI smoke-test sources are identified with exact URLs, branches, commits, dates, tested servers, and test conditions where available.
- [x] Smoke-test findings are distinguished from client-library limitations, server behavior, fixture issues, documentation gaps, and standards ambiguities.
- [x] Findings are grouped by API behavior area and tested implementation.
- [x] Reproducible, stale, inferred, and unresolved findings are distinguished.
- [x] Cross-server recurring issues and server-specific quirks are identified.
- [x] Schema/model, dynamic-data, tasking, documentation, validation, conformance, and interoperability implications are documented.
- [x] Test and fixture recommendations are explicit.
- [x] Downstream handoffs are explicit.
- [x] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** OS4CSAPI Client Smoke Test Findings Study Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Smoke-test source inventory and evidence classification
4. Tested servers, client versions, branches, and test conditions
5. Client-server behavior findings
6. Schema, model, and representation findings
7. Dynamic-data, status, event, and tasking findings
8. Cross-implementation interoperability findings
9. Client-library limitation versus server-behavior classification
10. Standards ambiguity and documentation-gap findings
11. Test strategy and fixture implications
12. Lessons for Glaux Server
13. Downstream topic handoff matrix
14. Recommendations
15. Risks, constraints, and open questions
16. Validation against this plan's success criteria
17. References

The smoke-test findings matrix should include, at minimum:

- Finding identifier
- Source URL / source anchor
- Evidence type
- Tested server
- Client branch/commit/version, if known
- Endpoint or behavior area
- Observed behavior
- Expected behavior or standard reference
- Likely root cause classification
- Server-side impact
- Client-side impact
- Glaux Server implication
- Test or fixture recommendation
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-006` through `IDR-SRV-014D` research reports should be complete or explicitly marked unavailable/deferred.
- OS4CSAPI client repository, relevant branches, smoke-test materials, and discussion/issue sources must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-014F: SECD Interoperability Findings Study`
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

**Actual Research Time:** Approximately 4.5 hours of AI-assisted elapsed execution<br>
**Completion Date:** August 31, 2026

---

## 10. Notes and Open Questions

- Smoke-test findings are evidence, not normative sources.
- The report must explicitly distinguish server behavior from client-library behavior.
- Some findings may have been fixed or superseded; the report must record freshness and current relevance.
- Resolved: `phase-9` commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230` is the controlling historical baseline because it matches this plan's source target and preserves the cumulative smoke reports, Phase 9 synthesis, and raw captures; later `main` and `phase-10` heads are disclosed as freshness context rather than silently substituted.
- Resolved: The pinned deterministic client integration workflows and type check were reproduced; mutable historical live-server observations remain explicitly historical rather than being promoted to current deployment claims.
- Resolved: Accepted Glaux standards reports control normative interpretation; ownership and unresolved downstream design questions are explicitly separated in the report.
- Resolved: Raw-wire golden, semantic-query, round-trip, lifecycle, runtime/documentation parity, conformance-traceability, and pinned external-client tests are the primary CI candidates; public-server tests remain scheduled canaries.
- Risk: Treating client assumptions as server obligations could distort the Glaux Server baseline.
- Risk: Ignoring smoke-test findings could cause avoidable interoperability failures.
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
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- OS4CSAPI client testing research corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing
- OS4CSAPI client testing research plans: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

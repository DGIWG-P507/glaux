# Section 014A: OSH CSAPI Server Implementation Study - Research Plan

**Status:** Planned  
**Last Updated:** June 8, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014a-osh-csapi-server-implementation-study-report.md`

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

This topic research must study the **OpenSensorHub / OSH CSAPI server implementation approach** and extract lessons relevant to Glaux Server.

The research must answer:

- How does OSH implement or expose OGC API - Connected Systems behavior?
- What OSH architectural patterns, module boundaries, datastore models, API-resource patterns, streaming behavior, command/tasking behavior, and documentation choices are relevant to Glaux Server?
- Which OSH behaviors appear standards-aligned, implementation-specific, incomplete, experimental, or divergent from the Glaux Server standards baseline?
- What strengths, gaps, risks, and reusable implementation lessons should influence Glaux Server design, validation, interoperability, and testing?
- What lessons from OSH should be carried forward to Glaux Server without copying OSH-specific architectural assumptions that do not fit Glaux Server's Rust-based, STANAG 4789 / AEP-4789 implementation context?

The output must be an OSH implementation findings baseline with source anchors, observed behaviors, applicability classifications, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows the core standards and API behavior baseline topics:

- `IDR-SRV-006: CSAPI Part 1 Requirement Baseline`
- `IDR-SRV-007: CSAPI Part 2 Requirement Baseline`
- `IDR-SRV-008: Conformance Class and Requirement Mapping`
- `IDR-SRV-009: Landing Page, API Definition, and Conformance Declaration Behavior`
- `IDR-SRV-010: Collections, Resources, Links, and Navigation Behavior`
- `IDR-SRV-010A: API Versioning, Backward Compatibility, and Deprecation Strategy`
- `IDR-SRV-011: Query, Filtering, Sorting, Pagination, and Selection Semantics`
- `IDR-SRV-012: Content Negotiation, Media Types, and Encoding Selection`
- `IDR-SRV-013: Error Model, HTTP Status Codes, and Failure Semantics`
- `IDR-SRV-014: OpenAPI Description and API Documentation Strategy`

Those topics define the expected standards baseline. This topic then examines a real implementation to identify practical design patterns, interoperability lessons, gaps, edge cases, and validation needs that may not be obvious from the standards alone.

### Critical Constraint(s)

- Treat OSH as an implementation study, not as the controlling authority for Glaux Server behavior.
- Treat OGC API - Connected Systems Part 1 and Part 2, AEP-4789 Volume II adoption context, and the Glaux Server IDR baseline as controlling where OSH behavior diverges.
- Do not assume OSH conformance without evidence from documentation, code, API responses, tests, or maintainers.
- Do not copy OSH architecture directly into Glaux Server. Extract lessons, patterns, tradeoffs, and risks.
- Distinguish observed behavior from inferred behavior.
- Distinguish current OSH behavior from historical, planned, or example-only behavior.
- Record exact source locations, repository paths, documentation pages, commits/tags/branches, API examples, and test evidence used.
- Keep the research bounded to Glaux Server relevance.

---

## 2. Research Questions

### Core Questions

1. What OSH components implement or expose OGC API - Connected Systems behavior?
2. How does OSH structure resources, collections, links, query behavior, dynamic data, streaming, commands, status, and API documentation?
3. Where does OSH align with the Glaux Server standards baseline, and where does it diverge?
4. What implementation strengths, gaps, and tradeoffs should inform Glaux Server design?
5. What test, validation, conformance, interoperability, and documentation lessons should be handed to later Glaux Server topics?

### Detailed Questions

#### Source and Version Baseline

- Which OSH repositories, branches, tags, documentation pages, examples, and demos are relevant to CSAPI behavior?
- What OSH version, branch, commit, or release is studied?
- Which OSH components appear server-side versus client-side, documentation-only, demo-only, or examples?
- What licensing and reuse constraints should be recorded?
- Are there active issues, pull requests, or discussions relevant to OSH CSAPI support?

#### Architecture and Module Boundaries

- How is OSH structured architecturally?
- Which modules provide server API behavior, data persistence, sensor integration, streaming, command/tasking, or client services?
- How are systems, sensors, datastreams, observations, procedures, deployments, and commands represented internally?
- What patterns are relevant to Glaux Server's module boundaries?
- Which OSH patterns are tied to Java/OSH architecture and should not be carried forward directly?

#### Standards Alignment and Conformance Posture

- Which CSAPI Part 1 resources and behaviors does OSH appear to implement?
- Which CSAPI Part 2 resources and behaviors does OSH appear to implement?
- Which conformance classes are claimed, implied, unsupported, or unclear?
- How does OSH expose conformance declarations, API definitions, media types, links, and schemas?
- What gaps or deviations appear relative to the Glaux Server requirement baseline?
- What evidence supports each alignment or gap assessment?

#### API Behavior Study

- How does OSH expose landing page, API definition, conformance, collections, items, and links?
- What URI patterns, link relations, resource nesting, and navigation behavior are used?
- What query/filter/sort/pagination/selection behavior is supported?
- What content negotiation and media-type behavior is supported?
- What error response behavior is observable or documented?
- How should these findings inform Glaux Server API behavior topics?

#### Dynamic Data, Streaming, and Tasking Behavior

- How does OSH expose datastreams and observations?
- How does OSH support real-time, historical, streaming, replay, or subscription-style access?
- How does OSH expose status, events, or dynamic system properties?
- How does OSH support command/control, tasking, feasibility, or actuator interactions?
- Which dynamic-data and tasking patterns are relevant to Glaux Server?
- Which patterns require careful separation from OSH-specific assumptions?

#### Persistence, Data Model, and Performance Implications

- What datastore or persistence abstractions are visible in OSH documentation or code?
- How does OSH model feature, observation, command, event, or time-series data?
- What indexing or query patterns are visible or implied?
- What performance, scale, or streaming constraints are documented or observable?
- Which persistence lessons should be handed to Category E topics?

#### Interoperability and Client Behavior

- Which OSH endpoints or examples are used by OSH JavaScript clients, CSAPI Explorer-like tools, demos, or other external clients?
- What compatibility patterns appear important for clients?
- What implementation-specific behavior could cause interoperability problems?
- What smoke-test cases should be derived from OSH behavior?
- Which findings should inform `IDR-SRV-056`?

#### Validation and Test Lessons

- What tests, examples, fixtures, API samples, or documentation snippets are available for OSH CSAPI behavior?
- What positive, negative, schema, conformance, golden-file, streaming, command, and interoperability tests should Glaux Server consider based on OSH?
- What OSH behavior should be used as comparison data but not as normative truth?
- Which test lessons should be handed to `IDR-SRV-050`, `IDR-SRV-051`, and `IDR-SRV-053`?

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

- `IDR-SRV-006` through `IDR-SRV-014` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### OSH / OpenSensorHub Sources

- OpenSensorHub GitHub organization: https://github.com/opensensorhub
- OSH Core repository: https://github.com/opensensorhub/osh-core
- OSH documentation repository: https://github.com/opensensorhub/osh-docs
- OSH documentation - Connected Systems page source:
  - https://github.com/opensensorhub/osh-docs/blob/master/docs/osh-connect/connected-systems.md
- OSH documentation - OSH Node introduction:
  - https://github.com/opensensorhub/osh-docs/blob/master/docs/osh-node/introduction.md
- OSH documentation - service modules:
  - https://github.com/opensensorhub/osh-docs/blob/master/docs/osh-node/user-docs/service-modules.md
- OSH documentation - client modules:
  - https://github.com/opensensorhub/osh-docs/blob/master/docs/osh-node/user-docs/client-modules.md
- OSH JavaScript / Connected Systems API examples in the OSH documentation repository, including examples with `consysapi` in file names.
- OSH API and Javadoc material under the OSH documentation repository, where relevant.
- OSH issues, pull requests, releases, branches, and commits relevant to Connected Systems API support.

### Controlling OGC and Standards-Package Sources

Use these to assess OSH behavior against the authoritative Glaux Server baseline:

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html

### Related OS4CSAPI and Interoperability Sources

- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

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

### Phase 1: Source Inventory and Study Baseline (1.5-2 hours)

**Objective:** Establish the OSH source baseline to be studied.

**Tasks:**

1. Identify OSH repositories, documentation pages, examples, issues, pull requests, branches, tags, commits, and demos relevant to CSAPI behavior.
2. Record exact versions, commit SHAs, dates, and source URLs used.
3. Separate server-side sources from client-side examples, documentation-only material, demo-only material, and historical material.
4. Define the evidence classification scheme:
   - documented behavior,
   - code-supported behavior,
   - API-observed behavior,
   - test-supported behavior,
   - inferred behavior,
   - unclear / unresolved.
5. Define the OSH-to-Glaux comparison matrix fields.

**Expected Output:** OSH source inventory and evidence baseline.

### Phase 2: Architecture and Module Analysis (2-3 hours)

**Objective:** Understand OSH architecture and implementation boundaries relevant to Glaux Server.

**Tasks:**

1. Review OSH architecture documentation and relevant code organization.
2. Identify server-side modules responsible for API exposure, resources, datastores, streaming, clients, commands, and services.
3. Identify data model abstractions and datastore interfaces relevant to CSAPI resources.
4. Identify OSH-specific assumptions that should not be generalized to Glaux Server.
5. Map architecture findings to Glaux Server downstream topics.

**Expected Output:** OSH architecture and module-boundary findings.

### Phase 3: CSAPI Standards Alignment Analysis (3-4 hours)

**Objective:** Compare OSH behavior to the Glaux Server CSAPI baseline.

**Tasks:**

1. Compare OSH Part 1 resource behavior against `IDR-SRV-006`.
2. Compare OSH Part 2 dynamic-data and tasking behavior against `IDR-SRV-007`.
3. Compare OSH conformance posture against `IDR-SRV-008`.
4. Compare OSH entry-point, navigation, query, content-negotiation, error, and documentation behavior against `IDR-SRV-009` through `IDR-SRV-014`.
5. Identify supported, partially supported, unsupported, divergent, implementation-specific, and unclear behaviors.

**Expected Output:** OSH standards-alignment and gap matrix.

### Phase 4: Dynamic Data, Streaming, Status, and Tasking Study (2-3 hours)

**Objective:** Extract OSH lessons for dynamic server behavior.

**Tasks:**

1. Study OSH handling of datastreams, observations, historical access, and real-time access.
2. Study OSH streaming, replay, subscription, or event publication patterns.
3. Study OSH status, events, availability, and dynamic-property patterns.
4. Study OSH control/tasking/command/actuator behavior where available.
5. Identify lessons and risks for Glaux Server dynamic-data, streaming, and command topics.

**Expected Output:** OSH dynamic-data and interaction findings.

### Phase 5: Interoperability, Validation, and Test Lesson Analysis (2-2.5 hours)

**Objective:** Prepare OSH findings for Glaux Server validation and interoperability work.

**Tasks:**

1. Identify OSH examples, demo responses, client code, Javadoc, or documentation snippets usable as test inspiration.
2. Identify client compatibility patterns and possible interoperability risks.
3. Identify candidate positive tests, negative tests, schema tests, OpenAPI tests, streaming tests, command tests, and golden files inspired by OSH.
4. Identify areas where OSH behavior should be compared against other implementations in `IDR-SRV-014B` through `IDR-SRV-014G`.
5. Map test handoffs to conformance, traceability, fixture, performance, security, and interoperability topics.

**Expected Output:** OSH interoperability and test-lesson matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert OSH research output into a decision-usable implementation findings baseline.

**Tasks:**

1. Consolidate source inventory, architecture findings, standards alignment, dynamic-data lessons, gaps, risks, and test implications.
2. Produce findings grouped by behavior area and Glaux Server topic impact.
3. Identify what Glaux Server should adopt, avoid, investigate further, or test.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] OSH sources relevant to CSAPI behavior are identified with exact URLs, branches, tags, commits, or dates.
- [ ] Server-side, client-side, documentation-only, demo-only, and inferred evidence are distinguished.
- [ ] OSH architecture and module boundaries are summarized.
- [ ] OSH Part 1 and Part 2 behavior is compared to the Glaux Server CSAPI baseline.
- [ ] OSH conformance posture is assessed without assuming conformance beyond evidence.
- [ ] OSH entry-point, navigation, query, representation, error, OpenAPI, dynamic-data, streaming, status, and tasking behavior are assessed where evidence exists.
- [ ] Strengths, gaps, risks, and implementation-specific assumptions are identified.
- [ ] Lessons for Glaux Server design, validation, testing, and interoperability are documented.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** OSH CSAPI Server Implementation Study Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014a-osh-csapi-server-implementation-study-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. OSH source inventory and evidence classification
4. OSH architecture and module-boundary findings
5. OSH CSAPI Part 1 behavior findings
6. OSH CSAPI Part 2 behavior findings
7. OSH conformance posture and standards-alignment matrix
8. OSH API behavior findings
9. OSH dynamic-data, streaming, status, and tasking findings
10. OSH persistence, datastore, and performance implications
11. OSH documentation, OpenAPI, and examples findings
12. Interoperability and client-compatibility findings
13. Test-strategy implications
14. Lessons for Glaux Server
15. Downstream topic handoff matrix
16. Recommendations
17. Risks, constraints, and open questions
18. Validation against this plan's success criteria
19. References

The implementation findings matrix should include, at minimum:

- Behavior or architecture area
- OSH source / source anchor
- Evidence type
- Observed OSH behavior
- Related CSAPI / Glaux baseline requirement
- Alignment classification
- Strength or useful pattern
- Gap, risk, or divergence
- Applicability to Glaux Server
- Test implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-006` through `IDR-SRV-014` research reports should be complete or explicitly marked unavailable/deferred.
- OSH repositories and documentation sources must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-014B: Connected Systems Go CSAPI Server Implementation Study`
- `IDR-SRV-014C: pygeoapi CSAPI Server Implementation Study`
- `IDR-SRV-014D: SECD CSAPI Server Implementation Study`
- `IDR-SRV-014E: OS4CSAPI Client Smoke Test Findings Study`
- `IDR-SRV-014F: SECD Interoperability Findings Study`
- `IDR-SRV-014G: OS4CSAPI Discussions Lessons-Learned Study`
- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-021: SensorML Representation Strategy`
- `IDR-SRV-022: SWE Common Data Component Strategy`
- `IDR-SRV-025: Database and Persistence Architecture Options`
- `IDR-SRV-035: Streaming and Event Publication Strategy`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
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

- OSH is an implementation study source, not a normative source.
- The report must explicitly identify evidence level for each observed behavior.
- Some OSH examples may be client-side or documentation examples rather than server behavior; the report must distinguish them.
- Open question: Which OSH branch, release, or commit best represents current CSAPI server behavior?
- Open question: Are there public OSH demo endpoints suitable for live API observation during the research report phase?
- Open question: Which OSH behaviors are deliberate CSAPI interpretation choices versus legacy architecture outcomes?
- Open question: Which OSH patterns are useful for Glaux Server despite the different implementation language and architecture?
- Risk: Treating OSH implementation choices as standards obligations could distort the Glaux Server baseline.
- Risk: Ignoring practical OSH lessons could cause avoidable interoperability or implementation issues.
- Risk: Studying stale documentation without code or API evidence could produce incorrect conclusions.

---

## References

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- OpenSensorHub GitHub organization: https://github.com/opensensorhub
- OSH Core repository: https://github.com/opensensorhub/osh-core
- OSH documentation repository: https://github.com/opensensorhub/osh-docs
- OSH Connected Systems documentation source: https://github.com/opensensorhub/osh-docs/blob/master/docs/osh-connect/connected-systems.md
- OSH Node introduction documentation source: https://github.com/opensensorhub/osh-docs/blob/master/docs/osh-node/introduction.md
- OSH service modules documentation source: https://github.com/opensensorhub/osh-docs/blob/master/docs/osh-node/user-docs/service-modules.md
- OSH client modules documentation source: https://github.com/opensensorhub/osh-docs/blob/master/docs/osh-node/user-docs/client-modules.md
- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

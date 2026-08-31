# Section 014D: SECD CSAPI Server Implementation Study - Research Plan

**Topic ID:** IDR-SRV-014D<br>
**Status:** Accepted<br>
**Last Updated:** August 31, 2026<br>
**Estimated Research Time:** 12-16.5 hours<br>
**Actual Research Time:** Approximately 4.5 hours of AI-assisted elapsed execution on August 31, 2026<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014d-secd-csapi-server-implementation-study-report.md`

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

This topic research must study the **observable SECD CSAPI server behavior and publicly declared implementation architecture** and extract lessons relevant to Glaux Server. Internal source-code structure, module boundaries, persistence mechanisms, and deployment details are in scope only when supported by an actual SECD server source repository or a maintainer-provided technical artifact that can be cited.

The research must answer:

- How does the SECD CSAPI server implementation expose, approximate, or test OGC API - Connected Systems behavior?
- What SECD API patterns, resource mappings, URI structures, data model assumptions, fixtures, OpenAPI behavior, conformance posture, validation behavior, and interoperability observations are relevant to Glaux Server?
- Which SECD behaviors appear standards-aligned, implementation-specific, incomplete, experimental, test-harness-specific, or divergent from the Glaux Server standards baseline?
- What strengths, gaps, risks, and reusable implementation lessons should influence Glaux Server design, validation, interoperability, and test strategy?
- How should SECD-specific lessons be separated from general CSAPI server design lessons so Glaux Server does not inherit implementation-specific assumptions?

The output must be a SECD server-behavior and declared-architecture findings baseline with source anchors, observed behaviors, evidence classifications, applicability assessments, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows:

- `IDR-SRV-014A: OSH CSAPI Server Implementation Study`
- `IDR-SRV-014B: Connected Systems Go CSAPI Server Implementation Study`
- `IDR-SRV-014C: pygeoapi CSAPI Server Implementation Study`

Those topics study multiple independent implementation approaches. This topic studies the SECD implementation so Glaux Server can compare another implementation and separate standards requirements from implementation patterns, interoperability issues, and client-observed behavior.

This topic is distinct from `IDR-SRV-014F`. `IDR-SRV-014D` owns observable server behavior, advertised API and schema behavior, conformance declarations, and explicitly documented architecture. `IDR-SRV-014F` owns interoperability findings, client and test-harness interactions, root-cause classification, reproducibility, and cross-implementation comparison.

### Critical Constraint(s)

- Treat SECD as an implementation study source, not as the controlling authority for Glaux Server behavior.
- Treat OGC API - Connected Systems Part 1 and Part 2, AEP-4789 Volume II adoption context, and the Glaux Server IDR standards baseline as controlling where SECD behavior diverges.
- Do not assume SECD conformance without evidence from code, documentation, API responses, tests, examples, issues, or maintainers.
- Distinguish current implementation behavior from documentation-only, test-only, demo-only, historical, planned, or experimental behavior.
- Distinguish SECD server implementation findings from SECD interoperability findings; leave cross-implementation analysis for `IDR-SRV-014F`.
- The `csapi-server-interop-secd` repository is an interoperability evidence repository, not the SECD server source repository.
- Do not infer internal modules, handlers, persistence, or deployment mechanisms from API responses, OpenAPI text, client findings, or test artifacts. If actual server source or maintainer technical documentation is unavailable, mark those questions `not verifiable` and complete this topic as a black-box server-behavior study.
- Classify evidence as actual server source, server-authored live artifact, maintainer statement, API observation, interoperability test evidence, historical snapshot, or inference.
- Record exact source locations, repository paths, commits/tags/branches, API examples, fixture data, issue identifiers, and test evidence used.
- Keep the research bounded to Glaux Server relevance.

---

## 2. Research Questions

### Core Questions

1. What externally visible interfaces expose SECD CSAPI behavior, and what internal components are directly evidenced if actual source is available?
2. How does SECD structure resources, collections, links, query behavior, dynamic data, API documentation, schemas, conformance declarations, and tests?
3. Where does SECD align with the Glaux Server standards baseline, and where does it diverge?
4. What implementation strengths, gaps, and tradeoffs should inform Glaux Server design?
5. What test, validation, conformance, interoperability, and documentation lessons should be handed to later Glaux Server topics?

### Detailed Questions

#### Source and Version Baseline

- Is actual SECD server source available? If so, which revision and files are studied? If not, what evidence establishes the source boundary?
- What exact branch, commit, release, deployment, or capture date is studied for each evidence source?
- Which facts are source-code-supported, maintainer-declared, OpenAPI-declared, API-observed, interoperability-test-supported, historical, or inferred?
- What licensing and reuse constraints should be recorded?

#### Architecture and Module Boundaries

- What architecture is explicitly declared by server-authored artifacts or maintainers?
- If actual server source is available, what routing, handler, data-model, persistence, validation, serialization, configuration, and test patterns are visible?
- If source is unavailable, which internal architecture questions must be marked `not verifiable`?
- What externally visible data-model and resource-family behavior can be established without inferring internal representation?
- Which patterns are relevant to Glaux Server, and which are SECD-specific?

#### Standards Alignment and Conformance Posture

- Which CSAPI Part 1 resources and behaviors does SECD appear to implement?
- Which CSAPI Part 2 resources and behaviors does SECD appear to implement?
- Which conformance classes are claimed, implied, unsupported, partial, or unclear?
- How does SECD expose landing page, API definitions, conformance declarations, collections, links, schemas, media types, queries, and errors?
- What gaps or deviations appear relative to the Glaux Server requirement baseline?
- What evidence supports each alignment or gap assessment?

#### API Behavior Study

- What URI patterns are used for resources and collections?
- How are links and related resources represented?
- What query/filter/sort/pagination/selection behavior is implemented?
- What content negotiation and media-type behavior is supported?
- What error response behavior is observable or documented?
- What OpenAPI or documentation artifacts are present?
- Which API behaviors should inform Glaux Server behavior topics?

#### Dynamic Data, Observations, Status, and Tasking Behavior

- How does SECD represent datastreams, observations, status, events, dynamic properties, or time-varying resources?
- Does SECD support real-time data, historical data, replay, event publication, or streaming-adjacent behavior?
- Does SECD support control streams, commands, feasibility, command status, or tasking-related resources?
- Which dynamic-data and tasking features are implemented, partial, absent, or unclear?
- Which lessons should be handed to dynamic-data, streaming, and command lifecycle topics?

#### Data Model, Fixtures, and Validation

- What externally visible data-model or fixture behavior can be observed, and what internal model or fixture structure is directly evidenced if actual source or maintainer documentation is available?
- Are resource models documented as generated, hand-authored, schema-derived, transformed, or fixture-driven? If not directly evidenced, mark this `not verifiable`.
- What sample resources, JSON payloads, observations, commands, errors, or golden responses are available?
- What schema validation, request validation, response validation, or fixture validation exists?
- Which fixtures or validation patterns could inform Glaux Server test-data strategy?

#### Interoperability and Client Behavior

- Which clients, smoke tests, or manual test procedures have been used against SECD?
- What client compatibility patterns appear important?
- What implementation-specific behavior could cause interoperability problems?
- What observed issues should be reserved for deeper analysis in `IDR-SRV-014F`?
- Which findings should inform `IDR-SRV-014E`, `IDR-SRV-014F`, and `IDR-SRV-056`?

#### Validation and Test Lessons

- What tests exist in the SECD repository or related interoperability work?
- What behaviors are untested, ambiguous, or difficult to test?
- What positive, negative, schema, conformance, golden-file, interoperability, and regression tests should Glaux Server consider based on SECD?
- What should be compared against OSH, CS-GO, pygeoapi, OS4CSAPI client smoke-test findings, and OS4CSAPI discussions?

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

- `IDR-SRV-006` through `IDR-SRV-014C` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### SECD Server Evidence

- Live SECD CSAPI API root:
  - https://cs.ogc.secd.eu/api/1.0
- Live SECD OpenAPI 3.1 document:
  - https://cs.ogc.secd.eu/api/1.0/api-docs/openapi.json
- Access-controlled maintainer introduction and repository-authored source-availability note:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd/blob/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0/correspondence/2026-05-07-matheus-email.md
- SECD interoperability repository, as access-controlled non-normative supporting evidence rather than server source:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd/tree/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0
- Any actual SECD server source repository or maintainer-provided architecture artifact discovered during research. Record its exact revision, provenance, license, and access status; do not assume such a source exists.

### Controlling OGC and Standards-Package Sources

Use these to assess SECD behavior against the authoritative Glaux Server baseline:

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html

### Related OS4CSAPI and Interoperability Sources

- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions
- OSH implementation sources from `IDR-SRV-014A`
- Connected Systems Go implementation sources from `IDR-SRV-014B`
- pygeoapi implementation sources from `IDR-SRV-014C`

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

**Objective:** Establish an evidence-bounded SECD server baseline to be studied.

**Tasks:**

1. Inventory the live API, OpenAPI document, maintainer statements, pinned interoperability evidence, historical captures, and any actual server source or architecture artifacts relevant to CSAPI behavior.
2. Record exact versions, commit SHAs, dates, and source URLs used.
3. Separate actual server source, server-authored live artifacts, maintainer statements, API observations, interoperability evidence, historical captures, and inference.
4. Define the evidence classification scheme:
   - actual-source-supported behavior,
   - maintainer-declared behavior,
   - OpenAPI-declared behavior,
   - API-observed behavior,
   - interoperability-test-supported behavior,
   - historical behavior,
   - inferred behavior,
   - unclear / unresolved.
5. Define the SECD-to-Glaux comparison matrix fields.

**Expected Output:** SECD source inventory and evidence baseline.

### Phase 2: Architecture, Resource, and Data Model Analysis (2-3 hours)

**Objective:** Understand SECD architecture and implementation boundaries relevant to Glaux Server.

**Tasks:**

1. Analyze the live API topology, conformance declarations, OpenAPI document, resource and schema shapes, representations, and any explicitly declared architecture.
2. If actual server source or maintainer architecture material is available, analyze startup/configuration, handlers, models, persistence, and tests with exact source anchors.
3. If that evidence is unavailable, mark internal implementation questions `not verifiable` and do not infer them from black-box behavior.
4. Distinguish explicitly declared architecture from independently observed API behavior.
5. Map the evidence-bounded findings to Glaux Server downstream topics.

**Expected Output:** SECD observable API and evidence-bounded architecture/model findings.

### Phase 3: CSAPI Standards Alignment Analysis (3-4 hours)

**Objective:** Compare SECD behavior to the Glaux Server CSAPI baseline.

**Tasks:**

1. Compare SECD Part 1 resource behavior against `IDR-SRV-006`.
2. Compare SECD Part 2 dynamic-data and tasking behavior against `IDR-SRV-007`.
3. Compare SECD conformance posture against `IDR-SRV-008`.
4. Compare SECD entry-point, navigation, query, content-negotiation, error, and documentation behavior against `IDR-SRV-009` through `IDR-SRV-014`.
5. Identify supported, partially supported, unsupported, divergent, implementation-specific, fixture-specific, and unclear behaviors.

**Expected Output:** SECD standards-alignment and gap matrix.

### Phase 4: API, Dynamic Data, and Interaction Behavior Study (2-3 hours)

**Objective:** Extract SECD lessons for API behavior, dynamic data, and interaction resources.

**Tasks:**

1. Study how SECD represents systems, procedures, deployments, sampling features, properties, datastreams, observations, status, events, control streams, commands, and feasibility resources where evidence exists.
2. Identify URI patterns, link patterns, resource relationships, query behavior, content negotiation, error behavior, and documentation behavior.
3. Identify dynamic-data and observation support.
4. Identify command/control or tasking support, if any.
5. Identify missing, partial, fixture-specific, and implementation-specific areas.

**Expected Output:** SECD API and interaction findings.

### Phase 5: Interoperability, Validation, and Test Lesson Analysis (2-2.5 hours)

**Objective:** Prepare SECD findings for Glaux Server validation and interoperability work.

**Tasks:**

1. Identify tests, fixtures, examples, sample responses, OpenAPI artifacts, validation assets, and compatibility notes.
2. Inventory client-compatibility and interoperability artifacts and formulate evidence handoffs for `IDR-SRV-014F`; do not adjudicate root cause, reproducibility, or cross-implementation findings here.
3. Identify candidate positive tests, negative tests, schema tests, OpenAPI tests, fixture tests, golden files, and interoperability tests inspired by SECD.
4. Identify behavior areas that `IDR-SRV-014F` should compare against OSH, CS-GO, pygeoapi, client smoke-test findings, and OS4CSAPI discussions.
5. Map test handoffs to conformance, traceability, fixture, performance, security, and interoperability topics.

**Expected Output:** SECD validation and test-lesson matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert SECD research output into a decision-usable implementation findings baseline.

**Tasks:**

1. Consolidate source inventory, architecture findings, standards alignment, resource behavior lessons, gaps, risks, and test implications.
2. Produce findings grouped by behavior area and Glaux Server topic impact.
3. Identify what Glaux Server should adopt, avoid, investigate further, or test.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [x] SECD sources relevant to CSAPI behavior are identified with exact URLs, branches, commits, deployments, or dates.
- [x] Actual server source, server-authored live artifacts, maintainer statements, interoperability artifacts, historical snapshots, and inference are distinguished.
- [x] Observable API structure and explicitly declared architecture are summarized with source anchors and capture dates.
- [x] Internal modules, persistence, and implementation mechanisms are reported only where actual server source or maintainer documentation supports them; unavailable areas are marked `not verifiable`.
- [x] SECD Part 1 and Part 2 behavior is compared to the Glaux Server CSAPI baseline.
- [x] SECD conformance posture is assessed without assuming conformance beyond evidence.
- [x] SECD entry-point, navigation, query, representation, error, OpenAPI, dynamic-data, status, and tasking behavior are assessed where evidence exists.
- [x] Strengths, gaps, risks, and implementation-specific assumptions are identified.
- [x] Lessons for Glaux Server design, validation, testing, and interoperability are documented.
- [x] Downstream handoffs are explicit.
- [x] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** SECD CSAPI Server Implementation Study Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014d-secd-csapi-server-implementation-study-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. SECD source inventory and evidence classification
4. SECD observable API topology and explicitly declared architecture
5. SECD CSAPI Part 1 behavior findings
6. SECD CSAPI Part 2 behavior findings
7. SECD conformance posture and standards-alignment matrix
8. SECD API behavior findings
9. SECD dynamic-data, status, and tasking findings
10. SECD externally visible data-model and fixture implications; internal persistence findings only where directly evidenced
11. SECD documentation, OpenAPI, and examples findings
12. Interoperability questions and evidence handoff to `IDR-SRV-014F`
13. Test-strategy implications
14. Lessons for Glaux Server
15. Downstream topic handoff matrix
16. Recommendations
17. Risks, constraints, and open questions
18. Validation against this plan's success criteria
19. References

The implementation findings matrix should include, at minimum:

- Behavior or architecture area
- SECD source / source anchor
- Evidence type
- Observed SECD behavior
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
- `IDR-SRV-006` through `IDR-SRV-014C` research reports must be complete and accepted before starting unless an exception is approved and recorded under the overall-plan Governance Rules.
- The live SECD API and OpenAPI evidence must be reachable or represented by a dated, reproducible capture. Actual server source is optional; if unavailable, the report must use the black-box scope defined above rather than block or infer internals.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-014E: OS4CSAPI Client Smoke Test Findings Study`
- `IDR-SRV-014F: SECD Interoperability Findings Study`
- `IDR-SRV-014G: OS4CSAPI Discussions Lessons-Learned Study`
- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-025: Database and Persistence Architecture Options`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
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

**Actual Research Time:** Approximately 4.5 hours of AI-assisted elapsed execution on August 31, 2026<br>
**Research Completion Date:** August 31, 2026<br>
**Acceptance Date:** August 31, 2026

---

## 10. Notes and Open Questions

- SECD is an implementation study source, not a normative source.
- The report must explicitly identify evidence level for each observed behavior.
- This topic should focus on observable SECD server behavior and explicitly declared architecture; broader interoperability findings should be reserved for `IDR-SRV-014F`.
- Resolved source boundary: no actual SECD server source, software release, license, test suite, or maintainer architecture artifact was found; the study therefore used the approved black-box scope and marked internal mechanisms not verifiable.
- Resolved live baseline: read-only probes of the live API and OpenAPI on August 31, 2026 are the current deployment evidence; the evidence repository remains pinned at commit `f018fd129bf0d0d1ce75e68198e3ab4d99d937a0`.
- Resolved intent boundary: the maintainer describes a starting drone/IPT prototype and open-source intent; intent for individual routes, schemas, divergences, persistence, and deployment mechanisms remains not verifiable.
- Resolved fixture question: independently authored drone/sensor hierarchies, flight/bridge deployments, scalar/record/choice schemas, high-volume observations, history, commands, and deliberate link/query/negotiation divergences are recommended.
- Resolved drift boundary: `/collections` and top-level Observation/Command reads now work despite contrary May captures; historical write and interoperability findings remain historical until IDR-SRV-014F reproduces them.
- Risk: Treating SECD implementation choices as standards obligations could distort the Glaux Server baseline.
- Risk: Ignoring practical SECD implementation lessons could cause avoidable API or test-design issues.
- Risk: Studying interoperability notes as if they were implementation behavior could blur the boundary between this topic and `IDR-SRV-014F`.

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
- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

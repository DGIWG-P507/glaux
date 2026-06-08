# Section 014B: Connected Systems Go CSAPI Server Implementation Study - Research Plan

**Status:** Planned  
**Last Updated:** June 8, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md`

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

This topic research must study the **Connected Systems Go / CS-GO CSAPI server implementation approach** and extract lessons relevant to Glaux Server.

The research must answer:

- How does Connected Systems Go implement or expose OGC API - Connected Systems server behavior?
- What Go implementation patterns, API-resource structures, routing patterns, persistence assumptions, data-loading mechanisms, OpenAPI/documentation behavior, conformance posture, and test assets are relevant to Glaux Server?
- Which CS-GO behaviors appear standards-aligned, implementation-specific, incomplete, experimental, or divergent from the Glaux Server standards baseline?
- What strengths, gaps, risks, and reusable implementation lessons should influence Glaux Server design, validation, interoperability, and test strategy?
- What lessons can be adapted to a Rust-based Glaux Server without importing Go-specific or implementation-specific assumptions?

The output must be a Connected Systems Go implementation findings baseline with source anchors, observed behaviors, applicability classifications, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows `IDR-SRV-014A: OSH CSAPI Server Implementation Study`.

`IDR-SRV-014A` studies a mature OSH/OpenSensorHub implementation. This topic studies a focused Go implementation so Glaux Server can compare a second CSAPI server approach across resource modeling, API behavior, documentation, validation, and interoperability.

This topic supports the broader `IDR-SRV-014A` through `IDR-SRV-014G` implementation-study set, which is intended to identify practical implementation lessons before Glaux Server moves into canonical resource model, persistence, validation, conformance harness, and interoperability test planning.

### Critical Constraint(s)

- Treat Connected Systems Go as an implementation study source, not as the controlling authority for Glaux Server behavior.
- Treat OGC API - Connected Systems Part 1 and Part 2, AEP-4789 Volume II adoption context, and the Glaux Server IDR standards baseline as controlling where CS-GO behavior diverges.
- Do not assume CS-GO conformance without evidence from code, documentation, API responses, tests, examples, issues, or maintainers.
- Distinguish current code behavior from documentation-only, example-only, fork-only, historical, planned, or experimental behavior.
- Distinguish the OS4CSAPI-maintained fork from upstream or personal forks, and record which repository, branch, commit, and date were studied.
- Do not copy Go-specific implementation architecture into Glaux Server. Extract reusable lessons and risks.
- Keep the research bounded to Glaux Server relevance.

---

## 2. Research Questions

### Core Questions

1. What Connected Systems Go components implement or expose CSAPI server behavior?
2. How does CS-GO structure resources, collections, links, query behavior, dynamic data, API documentation, schemas, and persistence?
3. Where does CS-GO align with the Glaux Server standards baseline, and where does it diverge?
4. What implementation strengths, gaps, and tradeoffs should inform Glaux Server design?
5. What test, validation, conformance, interoperability, and documentation lessons should be handed to later Glaux Server topics?

### Detailed Questions

#### Source and Version Baseline

- Which CS-GO repositories, forks, branches, tags, commits, examples, issues, pull requests, and demos are relevant?
- What is the relationship between the OS4CSAPI fork and the original implementation repository?
- What exact branch, commit, or release is studied?
- Which files are server code, model definitions, fixture data, documentation, tests, scripts, or examples?
- What licensing and reuse constraints should be recorded?

#### Architecture and Module Boundaries

- How is the Go application structured?
- What routing, handler, middleware, model, data access, configuration, and serialization patterns are visible?
- How are CSAPI resources represented internally?
- What assumptions are made about in-memory state, files, persistence, fixtures, simulation, or live feeds?
- Which patterns are reusable as design ideas for Glaux Server, and which are tied to Go implementation choices?

#### Standards Alignment and Conformance Posture

- Which CSAPI Part 1 resources and behaviors does CS-GO appear to implement?
- Which CSAPI Part 2 resources and behaviors does CS-GO appear to implement?
- Which conformance classes are claimed, implied, unsupported, or unclear?
- How does CS-GO expose landing page, API definition, conformance declarations, collections, links, schemas, media types, and errors?
- What gaps or deviations appear relative to the Glaux Server requirement baseline?
- What evidence supports each alignment or gap assessment?

#### API Behavior Study

- What URI patterns are used for resources and collections?
- How are links and related resources represented?
- What query/filter/sort/pagination/selection behavior is implemented?
- What content negotiation and media-type behavior is supported?
- What error response behavior is observable or documented?
- What API documentation or OpenAPI artifacts are present?
- Which API behaviors should inform Glaux Server behavior topics?

#### Dynamic Data, Observations, Status, and Tasking Behavior

- How does CS-GO represent datastreams, observations, and dynamic data?
- Does CS-GO support real-time data, historical data, status, events, streaming, or replay?
- Does CS-GO support control streams, commands, feasibility, or command status?
- Which dynamic-data and tasking features are implemented, partial, absent, or unclear?
- Which lessons should be handed to dynamic-data, streaming, and command lifecycle topics?

#### Persistence, Data Model, and Test Assets

- What storage or persistence model is used?
- Are resource models generated, hand-authored, schema-derived, or fixture-driven?
- What test data, sample resources, examples, or golden responses are available?
- What model/persistence limitations should Glaux Server avoid?
- Which fixtures or patterns could inform Glaux Server test-data strategy?

#### Interoperability and Client Behavior

- Which clients or tools have been used against CS-GO?
- What OS4CSAPI smoke-test or compatibility observations are linked to this implementation?
- What behaviors are necessary for CSAPI Explorer, generated clients, or external OGC API tooling?
- What implementation-specific behavior could create interoperability problems?
- Which findings should inform `IDR-SRV-014E`, `IDR-SRV-014F`, and `IDR-SRV-056`?

#### Validation and Test Lessons

- What tests exist in the CS-GO repository?
- What behaviors are untested or difficult to test?
- What positive, negative, schema, conformance, golden-file, interoperability, and regression tests should Glaux Server consider based on CS-GO?
- What should be compared against OSH, pygeoapi, SECD, and client smoke-test findings?

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

- `IDR-SRV-006` through `IDR-SRV-014A` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Connected Systems Go Sources

- OS4CSAPI Connected Systems Go repository:
  - https://github.com/OS4CSAPI/connected-systems-go
- Original / upstream Connected Systems Go repository, if relevant:
  - https://github.com/SomethingCreativeStudios/connected-systems-go
- Other forks or comparison sources, if relevant and clearly identified:
  - https://github.com/mvanhorn/connected-systems-go
- Repository README files, source directories, model definitions, route/handler files, configuration files, test files, examples, fixtures, issue threads, pull requests, releases, branches, and commits.
- Any public demos, deployment instructions, container files, or sample data referenced by the repository.

### Controlling OGC and Standards-Package Sources

Use these to assess CS-GO behavior against the authoritative Glaux Server baseline:

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
- SECD interoperability findings repository: https://github.com/Sam-Bolling/csapi-server-interop-secd

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

**Objective:** Establish the CS-GO source baseline to be studied.

**Tasks:**

1. Identify repositories, forks, branches, tags, commits, issues, pull requests, examples, tests, deployments, and documentation relevant to CS-GO.
2. Record exact versions, commit SHAs, dates, and source URLs used.
3. Separate server code, generated code, model definitions, fixtures, documentation, tests, demos, and historical or inactive material.
4. Define the evidence classification scheme: documented behavior, code-supported behavior, API-observed behavior, test-supported behavior, inferred behavior, unclear/unresolved.
5. Define the CS-GO-to-Glaux comparison matrix fields.

**Expected Output:** CS-GO source inventory and evidence baseline.

### Phase 2: Architecture and Module Analysis (2-3 hours)

**Objective:** Understand CS-GO architecture and implementation boundaries relevant to Glaux Server.

**Tasks:**

1. Review the Go project layout, dependency structure, and build/deployment model.
2. Identify server-side modules responsible for routing, handlers, resources, data loading, persistence, serialization, validation, documentation, and configuration.
3. Identify resource model and data model abstractions.
4. Identify assumptions about in-memory resources, fixture data, file-backed data, generated schemas, or live sources.
5. Map architecture findings to Glaux Server downstream topics.

**Expected Output:** CS-GO architecture and module-boundary findings.

### Phase 3: CSAPI Standards Alignment Analysis (3-4 hours)

**Objective:** Compare CS-GO behavior to the Glaux Server CSAPI baseline.

**Tasks:**

1. Compare CS-GO Part 1 resource behavior against `IDR-SRV-006`.
2. Compare CS-GO Part 2 dynamic-data and tasking behavior against `IDR-SRV-007`.
3. Compare CS-GO conformance posture against `IDR-SRV-008`.
4. Compare CS-GO entry-point, navigation, query, content-negotiation, error, and documentation behavior against `IDR-SRV-009` through `IDR-SRV-014`.
5. Identify supported, partially supported, unsupported, divergent, implementation-specific, and unclear behaviors.

**Expected Output:** CS-GO standards-alignment and gap matrix.

### Phase 4: Resource, Data, and Interaction Behavior Study (2-3 hours)

**Objective:** Extract CS-GO lessons for resource behavior, dynamic data, and interaction resources.

**Tasks:**

1. Study how CS-GO represents systems, procedures, deployments, sampling features, properties, datastreams, observations, status, events, control streams, commands, and feasibility resources.
2. Identify URI patterns, link patterns, resource relationships, and query behavior.
3. Identify dynamic-data and observation support.
4. Identify command/control or tasking support, if any.
5. Identify missing, partial, and implementation-specific areas.

**Expected Output:** CS-GO resource and interaction findings.

### Phase 5: Interoperability, Validation, and Test Lesson Analysis (2-2.5 hours)

**Objective:** Prepare CS-GO findings for Glaux Server validation and interoperability work.

**Tasks:**

1. Identify tests, fixtures, example responses, sample data, and generated artifacts in the repository.
2. Identify compatibility patterns and possible interoperability risks.
3. Identify candidate positive tests, negative tests, schema tests, OpenAPI tests, fixture tests, and golden files inspired by CS-GO.
4. Identify areas where CS-GO behavior should be compared against OSH, pygeoapi, SECD, client smoke-test findings, and OS4CSAPI discussions.
5. Map test handoffs to conformance, traceability, fixture, performance, security, and interoperability topics.

**Expected Output:** CS-GO interoperability and test-lesson matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert CS-GO research output into a decision-usable implementation findings baseline.

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

- [ ] CS-GO sources relevant to CSAPI behavior are identified with exact URLs, branches, tags, commits, or dates.
- [ ] Server code, generated code, fixtures, documentation, tests, demos, and inferred behavior are distinguished.
- [ ] CS-GO architecture and module boundaries are summarized.
- [ ] CS-GO Part 1 and Part 2 behavior is compared to the Glaux Server CSAPI baseline.
- [ ] CS-GO conformance posture is assessed without assuming conformance beyond evidence.
- [ ] CS-GO entry-point, navigation, query, representation, error, OpenAPI, dynamic-data, status, and tasking behavior are assessed where evidence exists.
- [ ] Strengths, gaps, risks, and implementation-specific assumptions are identified.
- [ ] Lessons for Glaux Server design, validation, testing, and interoperability are documented.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Connected Systems Go CSAPI Server Implementation Study Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. CS-GO source inventory and evidence classification
4. CS-GO architecture and module-boundary findings
5. CS-GO CSAPI Part 1 behavior findings
6. CS-GO CSAPI Part 2 behavior findings
7. CS-GO conformance posture and standards-alignment matrix
8. CS-GO API behavior findings
9. CS-GO dynamic-data, status, and tasking findings
10. CS-GO persistence, data model, and fixture implications
11. CS-GO documentation, OpenAPI, and examples findings
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
- CS-GO source / source anchor
- Evidence type
- Observed CS-GO behavior
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
- `IDR-SRV-006` through `IDR-SRV-014A` research reports should be complete or explicitly marked unavailable/deferred.
- Connected Systems Go repositories and documentation sources must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-014C: pygeoapi CSAPI Server Implementation Study`
- `IDR-SRV-014D: SECD CSAPI Server Implementation Study`
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

- Connected Systems Go is an implementation study source, not a normative source.
- The report must explicitly identify evidence level for each observed behavior.
- The OS4CSAPI fork and upstream/personal forks should be distinguished carefully.
- Some repository behavior may be sample/demo behavior rather than general server behavior.
- Open question: Which branch or commit best represents the current CS-GO server baseline?
- Open question: Are there public CS-GO demo endpoints suitable for live API observation during the research report phase?
- Open question: Which CS-GO behavior is deliberate standards interpretation versus scaffold or prototype behavior?
- Open question: Which CS-GO fixtures or examples are useful for Glaux Server test-data strategy?
- Risk: Treating CS-GO implementation choices as standards obligations could distort the Glaux Server baseline.
- Risk: Ignoring practical CS-GO lessons could cause avoidable API or test-design issues.
- Risk: Studying stale forks or unreviewed branches without evidence classification could produce incorrect conclusions.

---

## References

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- OS4CSAPI Connected Systems Go repository: https://github.com/OS4CSAPI/connected-systems-go
- Upstream Connected Systems Go repository: https://github.com/SomethingCreativeStudios/connected-systems-go
- Connected Systems Go comparison fork: https://github.com/mvanhorn/connected-systems-go
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
- SECD interoperability findings repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

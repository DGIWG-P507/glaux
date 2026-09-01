# Section 014G: OS4CSAPI Discussions Lessons-Learned Study - Research Plan

**Status:** Complete<br>
**Last Updated:** August 31, 2026<br>
**Estimated Research Time:** 10-14 hours  
**Actual Research Time:** Approximately 5 hours of AI-assisted elapsed execution<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md`

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

This topic research must study the **OS4CSAPI discussions, implementation lessons learned, standards interpretation concerns, developer pain points, interoperability issues, testing implications, and community recommendations** relevant to Glaux Server.

The research must answer:

- What lessons, concerns, implementation issues, interoperability problems, standards-interpretation questions, developer-experience problems, and testing recommendations appear across OS4CSAPI discussions?
- Which discussion findings are relevant to Glaux Server API behavior, resource modeling, validation, conformance, test-data strategy, external-client interoperability, documentation, and implementation planning?
- Which findings are actionable server-design lessons versus general community observations, client-library issues, project-management concerns, or out-of-scope discussion?
- Which findings support or challenge conclusions from the implementation-study sequence `IDR-SRV-014A` through `IDR-SRV-014F`?
- How should Glaux Server convert community lessons into research handoffs without treating informal discussion as normative evidence?

The output must be an OS4CSAPI discussions lessons-learned baseline with source anchors, evidence classification, topic mapping, server-impact analysis, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows:

- `IDR-SRV-014A: OSH CSAPI Server Implementation Study`
- `IDR-SRV-014B: Connected Systems Go CSAPI Server Implementation Study`
- `IDR-SRV-014C: pygeoapi CSAPI Server Implementation Study`
- `IDR-SRV-014D: SECD CSAPI Server Implementation Study`
- `IDR-SRV-014E: OS4CSAPI Client Smoke Test Findings Study`
- `IDR-SRV-014F: SECD Interoperability Findings Study`

Those topics study implementation behavior, client smoke-test results, and SECD-specific interoperability evidence. This topic closes the implementation-and-lessons-learned block by reviewing OS4CSAPI community discussions for cross-cutting concerns, standards interpretation issues, developer workflow lessons, and practical recommendations that may not be captured in code repositories or test outputs.

This topic must precede `IDR-SRV-015: Canonical Glaux Server Resource Model` because it provides final pre-resource-model input from the external implementation and community evidence base.

### Critical Constraint(s)

- Treat OS4CSAPI discussions as community evidence, not normative authority.
- Treat OGC API - Connected Systems Part 1 and Part 2, AEP-4789 Volume II adoption context, and the Glaux Server IDR standards baseline as controlling where discussion content conflicts with standards.
- Distinguish confirmed findings from opinions, proposals, unresolved questions, stale statements, informal suggestions, and implementation-specific observations.
- Distinguish server-side concerns from client-library concerns, tooling concerns, data-source concerns, documentation concerns, governance concerns, and out-of-scope issues.
- Preserve exact discussion URLs, comment links, dates, participants if relevant, and context needed for later traceability.
- Do not convert discussion content directly into engineering tickets; translate relevant content into research findings, risks, design implications, validation needs, and test strategy handoffs.
- Keep the research bounded to Glaux Server relevance.

---

## 2. Research Questions

### Core Questions

1. What OS4CSAPI discussions contain lessons relevant to Glaux Server?
2. Which discussion findings affect server API behavior, resource model, schema validation, conformance, testing, documentation, implementation planning, or interoperability?
3. Which discussion findings confirm, refine, or conflict with the implementation-study findings from `IDR-SRV-014A` through `IDR-SRV-014F`?
4. Which findings should be handed to resource-model, validation, conformance, test-data, documentation, security, performance, and interoperability topics?
5. How should informal discussion evidence be classified so Glaux Server can use it responsibly?

### Detailed Questions

#### Source and Evidence Baseline

- Which OS4CSAPI discussions, comments, issue threads, code-sprint notes, meeting notes, repository discussions, pull request comments, and related artifacts are relevant?
- Which discussions relate to CSAPI server implementations, client work, OpenAPI/schema issues, examples, fixtures, conformance, smoke testing, developer experience, data publishers, or interoperability?
- What dates, participants, repositories, linked issues, linked code, or linked external resources are associated with each discussion?
- Which discussions are current, stale, superseded, unresolved, or historically useful?
- What exact evidence supports each lesson?

#### Standards Interpretation and Ambiguity Lessons

- What standards interpretation questions appear in discussion?
- Which questions relate to CSAPI Part 1, CSAPI Part 2, OGC API - Features, SensorML, SWE Common, OpenAPI artifacts, schemas, media types, query behavior, command/tasking, streaming, or conformance?
- Which issues appear to be true standards ambiguity versus implementation misunderstanding?
- Which findings should be handed to resource-model, schema-validation, dynamic-data, tasking, conformance, or overall synthesis topics?
- Which open questions might require escalation to the CSAPI SWG or standards-maintenance process?

#### Implementation and Interoperability Lessons

- What implementation challenges recur across OSH, Connected Systems Go, pygeoapi, SECD, OS4CSAPI clients, CSAPI Explorer, or other community work?
- Which discussions identify server-side pitfalls that Glaux Server should avoid?
- Which discussions identify useful implementation patterns or shortcuts that Glaux Server should consider carefully?
- Which discussions reveal cross-server interoperability issues?
- Which discussions identify external-client expectations that Glaux Server should satisfy?

#### API Behavior and Resource Model Lessons

- What discussion findings affect landing page behavior, API definitions, conformance declarations, collections, resource paths, links, identifiers, relationships, query behavior, content negotiation, error behavior, and OpenAPI documentation?
- What findings affect the canonical Glaux Server resource model?
- What findings affect resource lifecycle, identifiers, temporal modeling, status/events, observed properties, controlled properties, SensorML, SWE Common, or semantic binding?
- Which findings should be explicitly carried into Category C and Category D topics?

#### Testing, Validation, and Fixture Lessons

- What discussions identify gaps in conformance testing, smoke testing, schema validation, generated-client behavior, golden files, fixtures, sample data, or interoperability testing?
- What test cases, fixtures, or scenario corpus entries should be derived from discussion lessons?
- Which tests are suggested by multiple independent discussions or implementation experiences?
- Which tests are risky because they encode one implementation's behavior rather than standard behavior?
- Which findings should be handed to `IDR-SRV-050`, `IDR-SRV-051`, `IDR-SRV-053`, and `IDR-SRV-056`?

#### Documentation, Developer Experience, and Adoption Lessons

- What discussions identify pain points for developers implementing or consuming CSAPI?
- Which documentation gaps affected implementation or client use?
- Which examples, tutorials, diagrams, API docs, conformance notes, or implementation guides are needed?
- Which findings are relevant to Glaux Server documentation, demo readiness, developer onboarding, and DGIWG/NATO implementer adoption?
- Which findings should be handed to OpenAPI, documentation, deployment, and demonstration topics?

#### Risk, Prioritization, and Open Questions

- Which discussion lessons identify high-risk implementation areas?
- Which lessons should influence sequencing or test prioritization?
- Which open questions remain unresolved after reviewing implementation studies and discussions?
- Which issues require standards clarification, implementation research, prototype testing, or community follow-up?
- Which findings are important but out of scope for Glaux Server?

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

- `IDR-SRV-006` through `IDR-SRV-014F` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### OS4CSAPI Discussion and Community Sources

- OS4CSAPI GitHub organization:
  - https://github.com/OS4CSAPI
- OS4CSAPI discussions:
  - https://github.com/orgs/OS4CSAPI/discussions
- OS4CSAPI client work:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- OS4CSAPI client testing research corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing
- OS4CSAPI code-sprint notes, issues, pull requests, discussion comments, repository README files, project documentation, and linked artifacts relevant to CSAPI implementation and interoperability.
- CSAPI Explorer:
  - https://ogc-csapi-explorer.pages.dev/

### Related Implementation and Interoperability Sources

Use these to compare discussion lessons against implementation and testing evidence:

- OSH / OpenSensorHub findings from `IDR-SRV-014A`
- Connected Systems Go findings from `IDR-SRV-014B`
- pygeoapi findings from `IDR-SRV-014C`
- SECD implementation findings from `IDR-SRV-014D`
- OS4CSAPI client smoke-test findings from `IDR-SRV-014E`
- SECD interoperability findings from `IDR-SRV-014F`
- SECD interoperability repository:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd

### Controlling OGC and Standards-Package Sources

Use these to classify discussion content against the authoritative standards baseline:

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
- OS4CSAPI testing research-plan corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans

---

## 5. Research Methodology

### Phase 1: Discussion Source Inventory and Evidence Baseline (1.5-2 hours)

**Objective:** Establish the OS4CSAPI discussions source baseline.

**Tasks:**

1. Identify relevant OS4CSAPI discussions, comments, linked issues, pull requests, code-sprint notes, repository artifacts, and external links.
2. Record exact URLs, dates, repositories, participants where relevant, linked artifacts, and topic context.
3. Separate discussion evidence from code evidence, test evidence, documentation evidence, implementation evidence, and informal opinion.
4. Define the evidence classification scheme:
   - confirmed finding,
   - recurring community observation,
   - implementation-supported lesson,
   - test-supported lesson,
   - standards-interpretation question,
   - proposal / recommendation,
   - opinion / anecdote,
   - stale / superseded,
   - unresolved.
5. Define the lesson-to-Glaux impact matrix fields.

**Expected Output:** OS4CSAPI discussions evidence inventory.

### Phase 2: Standards Interpretation and Ambiguity Extraction (2-2.5 hours)

**Objective:** Identify standards interpretation issues and ambiguity lessons.

**Tasks:**

1. Review discussions for CSAPI Part 1, CSAPI Part 2, OpenAPI, schemas, SensorML, SWE Common, OGC API - Features, conformance, and implementation interpretation questions.
2. Classify each issue as standards ambiguity, implementation misunderstanding, documentation gap, OpenAPI/schema gap, or unresolved question.
3. Compare each issue against the standards baseline and prior IDR findings.
4. Identify questions needing standards clarification or SWG follow-up.
5. Map findings to downstream Glaux topics.

**Expected Output:** Standards interpretation and ambiguity findings matrix.

### Phase 3: Implementation, Interoperability, and API Behavior Lesson Extraction (2.5-3.5 hours)

**Objective:** Extract implementation and interoperability lessons relevant to Glaux Server.

**Tasks:**

1. Review discussions for API behavior, resource modeling, link traversal, identifiers, query behavior, content negotiation, errors, OpenAPI, schema validation, dynamic data, streaming, command/tasking, and client interoperability.
2. Identify recurring lessons across multiple discussions or implementation experiences.
3. Identify server-side pitfalls and useful patterns.
4. Identify client expectations and cross-server compatibility issues.
5. Compare discussion lessons against findings from `IDR-SRV-014A` through `IDR-SRV-014F`.

**Expected Output:** Implementation and interoperability lessons matrix.

### Phase 4: Testing, Validation, Fixture, and Documentation Lesson Analysis (2-2.5 hours)

**Objective:** Convert discussion lessons into Glaux Server verification and documentation inputs.

**Tasks:**

1. Identify discussion lessons related to conformance testing, smoke testing, schema validation, fixtures, golden files, generated clients, external clients, and CI.
2. Identify documentation, example, tutorial, OpenAPI, and developer-experience gaps.
3. Identify test cases, scenario corpus entries, and fixture needs suggested by discussions.
4. Identify tests or examples that risk encoding implementation-specific behavior.
5. Map findings to validation, conformance, fixture, interoperability, documentation, and demo-readiness topics.

**Expected Output:** Test, validation, fixture, and documentation handoff matrix.

### Phase 5: Risk, Prioritization, and Handoff Analysis (1.5-2 hours)

**Objective:** Prepare discussion lessons for downstream IDR use.

**Tasks:**

1. Classify lessons by importance, confidence, evidence strength, and downstream impact.
2. Identify high-risk implementation areas.
3. Identify lessons that should influence sequencing or test priority.
4. Identify open questions requiring standards clarification, prototype testing, or community follow-up.
5. Prepare downstream handoff matrix for Category C onward.

**Expected Output:** Risk, prioritization, and downstream handoff matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert OS4CSAPI discussion lessons into a decision-usable baseline.

**Tasks:**

1. Consolidate source inventory, evidence classifications, lessons, risks, standards questions, and handoffs.
2. Produce findings grouped by standards interpretation, implementation, interoperability, testing, documentation, and risk.
3. Identify what Glaux Server should adopt, avoid, investigate further, validate, or escalate.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [x] Relevant OS4CSAPI discussions and linked artifacts are identified with exact URLs and dates where available.
- [x] Discussion evidence is distinguished from code, tests, documentation, implementation behavior, and informal opinion.
- [x] Standards interpretation issues and ambiguity lessons are classified.
- [x] Implementation and interoperability lessons are mapped to Glaux Server relevance.
- [x] Lessons from `IDR-SRV-014A` through `IDR-SRV-014F` are compared against discussion evidence.
- [x] Testing, validation, fixture, documentation, and developer-experience implications are documented.
- [x] High-risk areas and unresolved questions are identified.
- [x] Downstream handoffs are explicit.
- [x] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** OS4CSAPI Discussions Lessons-Learned Study Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Discussion source inventory and evidence classification
4. Standards interpretation and ambiguity findings
5. Implementation lesson findings
6. Interoperability lesson findings
7. API behavior and resource-model findings
8. Testing, validation, fixture, and conformance findings
9. Documentation and developer-experience findings
10. Risk and prioritization findings
11. Lessons for Glaux Server
12. Downstream topic handoff matrix
13. Recommendations
14. Risks, constraints, and open questions
15. Validation against this plan's success criteria
16. References

The lessons-learned matrix should include, at minimum:

- Lesson or finding identifier
- Discussion URL / source anchor
- Evidence type
- Topic area
- Summary of lesson
- Related standard or prior IDR finding
- Server-side relevance
- Confidence / evidence strength
- Glaux Server implication
- Test, documentation, or validation recommendation
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-006` through `IDR-SRV-014F` research reports should be complete or explicitly marked unavailable/deferred.
- OS4CSAPI discussions and linked repository artifacts must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy`
- `IDR-SRV-017: Relationship and Linkage Model`
- `IDR-SRV-018: Temporal, Validity, and Freshness Model`
- `IDR-SRV-020: Status, Availability, and System Event Model`
- `IDR-SRV-021: SensorML Representation Strategy`
- `IDR-SRV-022: SWE Common Data Component Strategy`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-024: Units, Observed Properties, and Semantic Binding Strategy`
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

**Actual Research Time:** Approximately 5 hours of AI-assisted elapsed execution<br>
**Completion Date:** August 31, 2026

---

## 10. Notes and Open Questions

- OS4CSAPI discussions are community evidence, not normative sources.
- The report must explicitly identify evidence level for each lesson.
- Some discussion content may be stale, superseded, or resolved by later implementation work; the report must record freshness and current relevance.
- Resolved: OpenAPI closure, negotiation, public-link correctness, fixture pinning, and semantic parsing recur across independent artifacts; the perceived value of `q`, copy/paste ease, and specific UI preferences remain bounded opinions.
- Resolved: Existing issue #186 and upstream-history entries cover current formal OpenAPI and future Part 3 questions; no new SWG escalation blocks Glaux.
- Resolved: Artifact discovery, tutorials, exact demo steps, and known-client limitations are documentation concerns; canonical links, representation equality, schema validity, errors, and public-origin identity are implementation behavior.
- Resolved: Artifact closure, recursive-schema, discovery-graph, negotiation-semantic, reverse-proxy, semantic-field-preservation, and immutable-fixture tests belong in CI; third-party live availability does not.
- Risk: Treating informal discussion as normative could distort the Glaux Server baseline.
- Risk: Ignoring community lessons could cause avoidable implementation, documentation, or interoperability issues.
- Risk: Mixing stale and current discussion findings without classification could produce misleading recommendations.

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
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- OS4CSAPI client testing research corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing
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

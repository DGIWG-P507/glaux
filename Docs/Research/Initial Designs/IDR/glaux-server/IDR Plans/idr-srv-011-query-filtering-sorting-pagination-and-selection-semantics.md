# Section 011: Query, Filtering, Sorting, Pagination, and Selection Semantics - Research Plan

**Status:** In Progress<br>
**Last Updated:** August 1, 2026<br>
**Estimated Research Time:** 12-16 hours<br>
**Actual Research Time:** Approximately 6 hours of AI-assisted elapsed execution time on August 1, 2026<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md`

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

This topic research must define the Glaux Server planning baseline for **query, filtering, sorting, pagination, and selection semantics**.

The research must answer:

- What query, filtering, sorting, pagination, and selection behavior must Glaux Server support for OGC API - Connected Systems Part 1 and Part 2?
- Which query behaviors are inherited from OGC API - Features, which are CSAPI-specific, and which require later project decisions?
- How should Glaux Server support discovery, retrieval, filtering, and selection across static feature resources, dynamic data resources, observations, datastreams, tasking/control resources, status, events, temporal properties, geospatial properties, identifiers, and linked resources?
- What query behavior should be deterministic, testable, interoperable, and safe for large datasets, streaming-adjacent workflows, tactical/limited-connectivity use, and external clients?
- What implications do query semantics create for persistence, indexing, API documentation, validation, error handling, performance, authorization, and test strategy?

The output must be a query behavior specification baseline with source anchors, server obligations, implementation implications, downstream handoffs, and test implications.

### Why This Topic Order

This topic follows:

- `IDR-SRV-009: Landing Page, API Definition, and Conformance Declaration Behavior`
- `IDR-SRV-010: Collections, Resources, Links, and Navigation Behavior`
- `IDR-SRV-010A: API Versioning, Backward Compatibility, and Deprecation Strategy`

Those topics define API entry points, navigable resource structure, and compatibility/evolution rules. This topic then defines how clients ask the server for subsets, sorted views, pages, projections, and selected resource representations.

This topic must precede content negotiation, error handling, OpenAPI documentation strategy, resource-model refinement, persistence/query architecture, geospatial storage, time-series storage, validation, performance testing, and external-client interoperability testing because those later topics depend on stable query semantics.

### Critical Constraint(s)

- Treat official OGC API - Connected Systems Part 1, Part 2, OGC API - Features, relevant OGC API building blocks, schemas, and OpenAPI artifacts as controlling where they define query behavior.
- Use `IDR-SRV-006`, `IDR-SRV-007`, `IDR-SRV-008`, `IDR-SRV-009`, `IDR-SRV-010`, and `IDR-SRV-010A` outputs as input when available.
- Do not design the database schema, index architecture, query optimizer, or persistence layer here; hand those implications to Category E topics.
- Do not finalize content negotiation or media-type behavior here; hand those implications to `IDR-SRV-012`.
- Do not finalize error contracts here; hand failure semantics to `IDR-SRV-013`.
- Do not invent query parameters that conflict with OGC API or CSAPI conventions.
- Clearly distinguish:
  - normative query parameters,
  - inherited OGC API behavior,
  - CSAPI-specific query behavior,
  - optional or conditional filters,
  - implementation-specific filters,
  - candidate future extensions,
  - unsupported or out-of-scope query behavior.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What query, filtering, sorting, pagination, and selection behaviors are required or expected for Glaux Server?
2. Which query semantics are inherited from OGC API - Features or common OGC API practice, and which are CSAPI-specific?
3. Which resource families require distinct query behavior, including feature resources, observations, datastreams, control streams, commands, status, and events?
4. How should query behavior support geospatial, temporal, identifier, relationship, property, status, and dynamic-data use cases?
5. What downstream persistence, validation, documentation, error-handling, security, performance, and test implications follow from the query baseline?

### Detailed Questions

#### Standards and Query Sources

- What query parameters and query semantics are defined in CSAPI Part 1?
- What query parameters and query semantics are defined in CSAPI Part 2?
- What query behavior is inherited from OGC API - Features Part 1?
- What optional OGC API building blocks, filtering standards, or common OGC API conventions may be relevant?
- What query behavior is expressed in OpenAPI artifacts or schema files but not obvious in prose?
- What query behavior is observed in existing CSAPI implementations or client expectations?

#### Resource Families and Query Scope

- What query behavior applies to systems, procedures, deployments, sampling features, and properties?
- What query behavior applies to datastreams and observations?
- What query behavior applies to control streams, commands, command status, and feasibility resources?
- What query behavior applies to system events, status, availability, and dynamic properties?
- Which resources support collection-level filtering, item retrieval, nested-resource retrieval, related-resource traversal, or history queries?
- Which query behaviors should be uniform across resource families, and which must be resource-specific?

#### Filtering Semantics

- What filters are required or expected for identifiers, resource type, text/search terms, relationships, owners, collections, observed properties, controlled properties, procedures, deployments, features of interest, and links?
- What geospatial filters are required or expected, such as bbox or geometry-based filtering?
- What temporal filters are required or expected, such as phenomenon time, result time, valid time, event time, update time, created time, or modified time?
- What status or availability filters are needed for operational usefulness?
- What command/tasking filters are needed for command state, lifecycle, feasibility, control stream, target system, or authorization-sensitive status?
- Which filters are normative, optional, implementation-specific, or future candidates?

#### Sorting Semantics

- What sorting behavior is required or expected by CSAPI, OGC API - Features, OpenAPI artifacts, or existing implementations?
- Which fields are safe and useful for sorting?
- How should sorting behave for temporal, geospatial, textual, identifier, status, and relationship fields?
- What default sort orders should be considered for resources such as observations, events, commands, and collections?
- What stability requirements are needed so pagination remains deterministic?
- Which sorting questions should be handed to persistence/indexing topics?

#### Pagination Semantics

- What pagination behavior is required or expected by CSAPI and inherited OGC API behavior?
- What limit, offset, cursor, next/prev link, count, or continuation semantics are required or recommended?
- How should pagination behave for large feature collections, high-volume observations, command histories, event histories, and time-series queries?
- How should pagination interact with sorting, filtering, authorization, and stale or changing datasets?
- How should Glaux Server avoid inconsistent pages, duplicate records, missing records, or ambiguous continuation behavior?
- Which pagination choices affect database and time-series storage research?

#### Selection and Projection Semantics

- What selection or projection behavior is required, expected, optional, or useful for CSAPI resources?
- Should clients be able to request subsets of properties, expanded links, embedded related resources, summaries, or detailed representations?
- How should selection behavior interact with schemas, OpenAPI descriptions, JSON/GeoJSON representations, SensorML references, SWE Common structures, and content negotiation?
- What selection behavior could create interoperability problems if not standardized?
- Which selection behaviors should be deferred as implementation-specific or future extensions?

#### Authorization, Security, and Policy Interaction

- How should query behavior interact with authorization and releasability constraints?
- What happens when a query spans authorized and unauthorized resources?
- How should filters avoid leaking the existence of restricted resources through counts, pagination metadata, timing, or error messages?
- What query limits, rate limits, payload limits, and resource-consumption controls are needed?
- Which findings should be handed to `IDR-SRV-039`, `IDR-SRV-040`, and `IDR-SRV-055`?

#### Error and Failure Semantics

- What errors arise from invalid query parameters, unsupported filters, invalid field names, invalid temporal ranges, invalid bbox values, invalid sort expressions, unsupported selection requests, excessive limits, and incompatible filter combinations?
- Which error behaviors are defined by standards, inherited from OGC API behavior, or implementation choices?
- What deterministic error taxonomy should be handed to `IDR-SRV-013`?
- Which negative tests are required?
- How should query errors be documented in OpenAPI?

#### Performance, Persistence, and Indexing Implications

- Which query patterns are likely common, mission-critical, or expensive?
- Which filters require indexes or specialized storage support?
- Which queries require geospatial indexing, temporal indexing, time-series storage, relationship traversal, full-text search, or JSON/document indexing?
- Which query patterns may require limits, async processing, precomputation, or refusal?
- Which findings should be handed to `IDR-SRV-025`, `IDR-SRV-026`, `IDR-SRV-027`, `IDR-SRV-028`, and `IDR-SRV-054`?

#### Interoperability and Existing Implementation Lessons

- How do existing CSAPI implementations support query, filtering, sorting, pagination, and selection?
- What client smoke-test findings identify query compatibility issues?
- Which query patterns are needed by CSAPI Explorer and other external clients?
- Which existing implementation differences should Glaux Server account for?
- Which findings should be handed to `IDR-SRV-014A` through `IDR-SRV-014G` and `IDR-SRV-056`?

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

- `IDR-SRV-006` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-006-csapi-part-1-requirement-baseline-report.md`
- `IDR-SRV-007` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-007-csapi-part-2-requirement-baseline-report.md`
- `IDR-SRV-008` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-008-conformance-class-and-requirement-mapping-report.md`
- `IDR-SRV-009` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-009-landing-page-api-definition-and-conformance-declaration-behavior-report.md`
- `IDR-SRV-010` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-010-collections-resources-links-and-navigation-behavior-report.md`
- `IDR-SRV-010A` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-010a-api-versioning-backward-compatibility-and-deprecation-strategy-report.md`
- Earlier baseline reports, if needed for scope and terminology:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md`
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md`
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md`
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-004-terminology-and-concept-crosswalk-report.md`
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-005-related-nato-standards-boundary-review-report.md`

### Controlling OGC Sources

- OGC API - Connected Systems landing page:
  - https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources:
  - https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data:
  - https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository:
  - https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts:
  - https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC API - Connected Systems schemas:
  - https://schemas.opengis.net/
- OGC API - Features Part 1 standard:
  - https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC API standards family:
  - https://ogcapi.ogc.org/

### OGC Query and Filtering Context Sources

Use these where relevant to distinguish required CSAPI behavior from optional or candidate query extensions:

- OGC API - Features standards and resources:
  - https://ogcapi.ogc.org/features/
- OGC API - Common:
  - https://ogcapi.ogc.org/common/
- OGC API - Records, for query/filtering comparison where relevant:
  - https://ogcapi.ogc.org/records/
- OGC API - Environmental Data Retrieval, for temporal/spatial query comparison where relevant:
  - https://ogcapi.ogc.org/edr/
- OGC API - Processes, for async and parameter/query comparison where relevant:
  - https://ogcapi.ogc.org/processes/
- OGC API - Features - Part 3: Filtering and the Common Query Language (CQL2), where relevant:
  - https://docs.ogc.org/is/19-079r2/19-079r2.html
- OGC CQL2 landing/resources:
  - https://ogcapi.ogc.org/cql2/

### HTTP and API Context Sources

- RFC 9110 - HTTP Semantics:
  - https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 - HTTP Caching:
  - https://www.rfc-editor.org/rfc/rfc9111
- OpenAPI Specification:
  - https://spec.openapis.org/oas/latest.html

### Existing Implementation and Interoperability Context

Use these as supporting evidence once the corresponding focused studies are available:

- `IDR-SRV-014A through IDR-SRV-014G`
- OS4CSAPI organization:
  - https://github.com/OS4CSAPI
- OS4CSAPI client work:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability findings repository:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer:
  - https://ogc-csapi-explorer.pages.dev/

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

### Phase 1: Source Collection and Query-Behavior Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and define the extraction framework for query, filtering, sorting, pagination, and selection behavior.

**Tasks:**

1. Gather CSAPI Part 1, CSAPI Part 2, OGC API - Features, schema, OpenAPI, and query-related OGC sources.
2. Gather reports from `IDR-SRV-006` through `IDR-SRV-010A`, if available.
3. Identify requirements and conformance classes that mention query parameters, filters, sorting, pagination, selection, temporal constraints, geospatial constraints, or collection retrieval.
4. Define query behavior inventory fields for the report.
5. Define classification values for query behavior:
   - normative,
   - inherited,
   - optional,
   - conditional,
   - implementation-specific,
   - candidate future extension,
   - unsupported / out of scope,
   - unresolved.

**Expected Output:** Source inventory and query-behavior extraction framework.

### Phase 2: Standards-Based Query Parameter and Behavior Extraction (3-4 hours)

**Objective:** Extract required and expected query behavior from CSAPI and inherited OGC API sources.

**Tasks:**

1. Review CSAPI Part 1 for query, filtering, pagination, sorting, selection, and collection access requirements.
2. Review CSAPI Part 2 for dynamic-data, observations, commands, events, status, and temporal query requirements.
3. Review OGC API - Features inherited behavior relevant to bbox, datetime, limit, pagination, collection access, and item retrieval where applicable.
4. Review OpenAPI artifacts and schemas for query parameters and response structures.
5. Capture source anchors, parameter names, resource applicability, data types, constraints, defaults, and expected response effects.

**Expected Output:** Query parameter and behavior inventory with source anchors.

### Phase 3: Resource-Family Query Semantics Mapping (2.5-3.5 hours)

**Objective:** Map query behavior to Glaux Server resource families.

**Tasks:**

1. Group query behavior by resource family:
   - systems,
   - procedures,
   - deployments,
   - sampling features,
   - properties,
   - datastreams,
   - observations,
   - control streams,
   - commands,
   - feasibility resources,
   - status/dynamic properties,
   - system events.
2. Identify which filters and query semantics apply to each family.
3. Identify uniform query patterns and resource-specific query patterns.
4. Identify dependencies on resource relationships, identifiers, temporal semantics, status semantics, and data encodings.
5. Identify unresolved mapping questions.

**Expected Output:** Resource-family query semantics matrix.

### Phase 4: Sorting, Pagination, Selection, and Consistency Analysis (2.5-3.5 hours)

**Objective:** Define deterministic behavior expectations for result ordering, paging, projection, and consistency.

**Tasks:**

1. Identify required, recommended, and candidate sort semantics.
2. Identify required and recommended pagination mechanisms, metadata, and continuation links.
3. Analyze how pagination interacts with filtering, sorting, authorization, and changing datasets.
4. Identify selection/projection or field-subset behavior required by standards or useful for implementation.
5. Identify compatibility and versioning implications from `IDR-SRV-010A`.
6. Identify consistency, duplicate, missing-result, and stale-result risks.

**Expected Output:** Sorting/pagination/selection behavior and consistency analysis.

### Phase 5: Security, Performance, Persistence, and Error Implication Analysis (2-2.5 hours)

**Objective:** Prepare query findings for downstream implementation and test strategy topics.

**Tasks:**

1. Identify query behaviors that require database indexes, geospatial indexes, temporal indexes, relationship traversal, JSON/document indexing, full-text search, or time-series storage.
2. Identify expensive or unsafe query patterns requiring limits, rejection, pagination, authorization checks, rate limits, or resource-consumption controls.
3. Identify query behaviors that could leak restricted resource existence, counts, or metadata.
4. Identify error and failure semantics for invalid, unsupported, unauthorized, excessive, or incompatible queries.
5. Identify test implications for positive, negative, boundary, load, interoperability, and security tests.
6. Map handoffs to persistence, security, error, documentation, validation, and testing topics.

**Expected Output:** Query implementation and test-implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert research output into a decision-usable query behavior specification baseline.

**Tasks:**

1. Consolidate query parameter inventory, resource-family mappings, sorting/pagination/selection findings, and implementation/test implications.
2. Resolve conflicts and ambiguities where possible.
3. Produce findings grouped by query behavior area and resource family.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [x] Query, filtering, sorting, pagination, and selection requirements are identified with source anchors.
- [x] CSAPI-specific behavior and inherited OGC API behavior are distinguished.
- [x] Query behavior is mapped to relevant Glaux Server resource families.
- [x] Geospatial, temporal, identifier, relationship, status, dynamic-data, and tasking query implications are assessed.
- [x] Sorting and pagination consistency implications are documented.
- [x] Selection/projection behavior is classified as required, optional, implementation-specific, future candidate, or out of scope.
- [x] Security, authorization, resource-consumption, and information-leakage implications are identified.
- [x] Persistence, indexing, OpenAPI, error-handling, validation, performance, and testing handoffs are documented.
- [x] Recommendations are decision-usable and bounded to Glaux Server.
- [x] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Query, Filtering, Sorting, Pagination, and Selection Semantics Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Query behavior extraction methodology
5. Standards-based query parameter and behavior inventory
6. Resource-family query semantics matrix
7. Filtering semantics findings
8. Sorting, pagination, and consistency findings
9. Selection/projection findings
10. Security, authorization, and resource-consumption implications
11. Persistence, indexing, and performance implications
12. Error/failure and validation implications
13. Interoperability and existing-implementation implications
14. Test-strategy implications
15. Downstream topic handoff matrix
16. Recommendations
17. Risks, constraints, and open questions
18. Validation against this plan's success criteria
19. References

The query behavior matrix should include, at minimum:

- Resource family
- Query parameter or behavior
- Source standard / source anchor
- Requirement or convention summary
- Normative / inherited / optional / conditional / implementation-specific / unresolved classification
- Data type and allowed values, where applicable
- Default behavior, where applicable
- Response effect
- Error condition(s)
- Security or authorization implication
- Persistence/indexing implication
- Test implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-006` through `IDR-SRV-010A` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, OpenAPI, schema, and query-related OGC sources must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-012: Content Negotiation, Media Types, and Encoding Selection`
- `IDR-SRV-013: Error Model, HTTP Status Codes, and Failure Semantics`
- `IDR-SRV-014: OpenAPI Description and API Documentation Strategy`
- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy`
- `IDR-SRV-017: Relationship and Linkage Model`
- `IDR-SRV-018: Temporal, Validity, and Freshness Model`
- `IDR-SRV-020: Status, Availability, and System Event Model`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-025: Database and Persistence Architecture Options`
- `IDR-SRV-026: Geospatial Storage and Query Strategy`
- `IDR-SRV-027: Time-Series Observation Storage Strategy`
- `IDR-SRV-028: Metadata and Document Storage Strategy`
- `IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints`
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

**Actual Research Time:** Approximately 6 hours of AI-assisted elapsed execution time on August 1, 2026<br>
**Research Execution Completed:** August 1, 2026<br>
**Completion Date:** Pending plan-owner acceptance

---

## 10. Notes and Open Questions

- This topic should define query semantics and downstream implications, not implement the query engine.
- Query behavior must be compatible with OGC API/CSAPI conventions and should not create project-specific behavior that breaks external clients.
- Existing implementation studies may later refine practical expectations for query parameter support and interoperability.
- Some advanced filters may be useful but should be clearly classified as optional, implementation-specific, or future extension if not required by standards.
- Open question: Which CQL2 or advanced-filtering capabilities, if any, should Glaux Server eventually support?
- Open question: Which query patterns are essential for CSAPI Explorer and likely external clients?
- Open question: How should Glaux Server handle counts, totals, and pagination metadata when authorization filtering applies?
- Open question: Which query limits should be enforced to prevent resource exhaustion without harming legitimate discovery and analysis workflows?
- Risk: Overly broad query support could create complexity, performance risk, and security exposure.
- Risk: Under-supported query behavior could make the server technically conformant but operationally weak.
- Risk: Inconsistent pagination or sorting could break clients and test reproducibility.
- Risk: Query metadata could leak restricted resource existence or counts if authorization is not considered.

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
- OGC API - Connected Systems - Part 1: Feature Resources:
  - https://docs.ogc.org/is/23-001/23-001.html
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
- OGC API - Features:
  - https://ogcapi.ogc.org/features/
- OGC API - Features Part 1:
  - https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC API - Common:
  - https://ogcapi.ogc.org/common/
- OGC API - Records:
  - https://ogcapi.ogc.org/records/
- OGC API - Environmental Data Retrieval:
  - https://ogcapi.ogc.org/edr/
- OGC API - Processes:
  - https://ogcapi.ogc.org/processes/
- OGC API - Features - Part 3: Filtering and CQL2:
  - https://docs.ogc.org/is/19-079r2/19-079r2.html
- OGC CQL2 resources:
  - https://ogcapi.ogc.org/cql2/
- RFC 9110 - HTTP Semantics:
  - https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 - HTTP Caching:
  - https://www.rfc-editor.org/rfc/rfc9111
- OpenAPI Specification:
  - https://spec.openapis.org/oas/latest.html
- OS4CSAPI testing research-plan corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans

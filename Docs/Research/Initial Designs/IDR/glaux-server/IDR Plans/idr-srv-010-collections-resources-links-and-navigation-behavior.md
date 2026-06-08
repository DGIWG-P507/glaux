# Section 010: Collections, Resources, Links, and Navigation Behavior - Research Plan

**Status:** Planned  
**Last Updated:** June 8, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-010-collections-resources-links-and-navigation-behavior-report.md`

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

This topic research must define the Glaux Server planning baseline for **collections, resources, links, and navigation behavior**.

The research must answer:

- What collections and resource navigation behavior must Glaux Server support for OGC API - Connected Systems Part 1 and Part 2?
- How should resources be organized, exposed, linked, discovered, and traversed by standards-conformant clients?
- What link relations, resource relationships, identifiers, collection metadata, item endpoints, and navigation patterns are required, recommended, conventional, or implementation-specific?
- How should Glaux Server expose connected-system resources such as systems, procedures, deployments, sampling features, properties, datastreams, control streams, commands, observations, status, and events through a coherent navigable API?
- What test, validation, interoperability, and downstream resource-model implications follow from the required resource and navigation behavior?

The output must be a collections/resources/links/navigation behavior baseline with source anchors, server obligations, downstream handoffs, and test implications.

### Why This Topic Order

This topic follows `IDR-SRV-009: Landing Page, API Definition, and Conformance Declaration Behavior`.

`IDR-SRV-009` defines the API entry-point behavior. This topic moves one level deeper into the navigable resource structure that clients use after discovering the API.

This topic must precede query/filtering semantics, content negotiation, error handling, OpenAPI documentation strategy, canonical resource modeling, identifier strategy, relationship modeling, persistence, and interoperability testing because those later topics depend on a clear understanding of how Glaux Server resources are organized and traversed.

### Critical Constraint(s)

- Treat official OGC API - Connected Systems Part 1, Part 2, OGC API - Features, schemas, and OpenAPI artifacts as controlling where they define collections, resources, links, and navigation behavior.
- Use `IDR-SRV-006`, `IDR-SRV-007`, `IDR-SRV-008`, and `IDR-SRV-009` outputs as input when available.
- Do not finalize the canonical resource domain model here; hand deeper modeling implications to `IDR-SRV-015` through `IDR-SRV-020`.
- Do not finalize identifier, URI, and lifecycle strategy here; hand those implications to `IDR-SRV-016`.
- Do not finalize query/filter behavior here; hand those implications to `IDR-SRV-011`.
- Do not finalize content negotiation or media-type behavior here; hand those implications to `IDR-SRV-012`.
- Do not design persistence, database schema, or indexing here; hand storage/query implications to Category E topics.
- Clearly distinguish:
  - normative resource and collection requirements,
  - inherited OGC API - Features behavior,
  - CSAPI-specific resource behavior,
  - required links and recommended links,
  - navigability expectations,
  - implementation choices,
  - downstream model, validation, and test implications.
- Keep the research bounded to server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What collection and resource behavior must Glaux Server implement for CSAPI Part 1 and Part 2 compatibility?
2. What link and navigation behavior must Glaux Server support so connected-system resources are discoverable and traversable by clients?
3. How do Part 1 feature-resource behaviors and Part 2 dynamic-data behaviors relate through resource links and navigation patterns?
4. Which navigation behaviors are required by standards, inherited from OGC API - Features, conventional in OGC APIs, or implementation-specific choices?
5. What downstream design and test implications follow from the collections/resources/links/navigation baseline?

### Detailed Questions

#### Collections and Collection Metadata

- What collection resources are required or implied by CSAPI Part 1?
- What collection resources are required or implied by CSAPI Part 2?
- What metadata must or should each collection expose?
- Which collections represent feature resources, dynamic data resources, tasking/control resources, status resources, or event resources?
- Which collection behaviors are inherited from OGC API - Features?
- How should collection membership, collection identifiers, collection links, and item links be represented?
- Which collection metadata affects OpenAPI documentation, schema validation, and client discovery?

#### Resource Categories and Endpoints

- What resource categories must Glaux Server expose for systems, procedures, deployments, sampling features, properties, datastreams, observations, control streams, commands, feasibility exchanges, status, and system events?
- Which resource endpoints are collection endpoints, item endpoints, subordinate-resource endpoints, or relationship/navigation endpoints?
- Which resources are primarily static or descriptive, and which are dynamic or time-varying?
- Which resources must be independently addressable?
- Which resources are nested, linked, referenced, or derived from other resources?
- Which resource behaviors need later canonical modeling in `IDR-SRV-015`?

#### Links and Link Relations

- What link objects, link relations, hrefs, media types, titles, roles, or profile references are required or expected?
- Which links are required for landing pages, collections, items, related resources, API definitions, conformance declarations, alternate representations, schemas, and documentation?
- Which CSAPI-specific resource relationships require links?
- Which links connect Part 1 resources to Part 2 dynamic-data resources?
- Which link relations are inherited from OGC API - Features, IANA relation types, JSON Hyper-Schema, or other conventions?
- Which link patterns are necessary for external-client interoperability?

#### Navigation and Traversal Patterns

- How should a client navigate from the API root to collections and from collections to items?
- How should a client navigate from a system to deployments, procedures, datastreams, control streams, observations, properties, status, commands, and events?
- How should a client navigate from observations to datastreams, systems, observed properties, sampling features, result structures, and temporal context?
- How should a client navigate from control streams to commands, command status, feasibility, and target systems?
- How should Glaux Server represent reverse links, parent-child links, cross-resource links, and related-resource links?
- Which navigation patterns are required versus implementation choices?

#### Resource Identity and URI Implications

- What URI patterns are required or implied for collections and resources?
- Which resources require stable URIs?
- Which resources require canonical item URLs?
- Which resources may have alternate identifiers or external identifiers?
- Which URI and identity questions must be handed to `IDR-SRV-016`?
- Which resource lifecycle questions arise from collection and navigation behavior?

#### Validation, Error, and Consistency Implications

- What validation rules are implied by resource membership, links, and navigation behavior?
- What should happen when a linked resource is missing, unavailable, unauthorized, stale, deleted, or moved?
- Which link consistency problems require error or warning behavior?
- Which failure modes should be handed to `IDR-SRV-013`?
- Which consistency and transaction concerns should be handed to `IDR-SRV-029`?

#### Interoperability and Existing Implementation Lessons

- How do existing CSAPI implementations expose collections, resources, links, and navigation?
- What compatibility issues have been observed in client smoke tests, interoperability experiments, or discussions?
- Which navigation patterns are necessary for CSAPI Explorer, external clients, OGC tooling, and automated test clients?
- Which implementation differences should Glaux Server account for?
- Which questions should be handed to `IDR-SRV-014A` through `IDR-SRV-014G` and `IDR-SRV-056`?

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
  - https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC API - Connected Systems schemas:
  - https://schemas.opengis.net/
- OGC API - Features Part 1 standard:
  - https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC API standards family:
  - https://ogcapi.ogc.org/

### Link, Web, and API Context Sources

Use these for link relation and API-navigation context where relevant:

- IANA Link Relation Types:
  - https://www.iana.org/assignments/link-relations/link-relations.xhtml
- RFC 8288 - Web Linking:
  - https://www.rfc-editor.org/rfc/rfc8288
- OpenAPI Specification:
  - https://spec.openapis.org/oas/latest.html
- W3C Semantic Sensor Network Ontology / SOSA:
  - https://www.w3.org/TR/vocab-ssn/

### Existing Implementation and Interoperability Context

Use these as supporting evidence once the corresponding focused studies are available:

- `IDR-SRV-014A through IDR-SRV-014G`
- OS4CSAPI organization:
  - https://github.com/OS4CSAPI
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
- OS4CSAPI client work, for later interoperability context only:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability findings repository, for later interoperability context only:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd
- OS4CSAPI discussions, for background only:
  - https://github.com/orgs/OS4CSAPI/discussions

---

## 5. Research Methodology

### Phase 1: Source Collection and Navigation Extraction Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and define the extraction framework for collections, resources, links, and navigation behavior.

**Tasks:**

1. Gather CSAPI Part 1, CSAPI Part 2, OGC API - Features, schema, and OpenAPI artifact sources.
2. Gather reports from `IDR-SRV-006` through `IDR-SRV-009`, if available.
3. Identify requirements and conformance classes that mention collections, items, links, resource relationships, resource navigation, and URI patterns.
4. Identify inherited OGC API - Features behavior relevant to collections and feature resources.
5. Define the behavior inventory and link/navigation matrix fields to be used in the report.

**Expected Output:** Source inventory and collections/resources/links/navigation extraction framework.

### Phase 2: Collections and Resource Endpoint Extraction (2.5-3.5 hours)

**Objective:** Extract server requirements and conventions for collections and resource endpoints.

**Tasks:**

1. Identify all collection and item resource patterns in CSAPI Part 1 and Part 2.
2. Identify inherited OGC API - Features collection and item behavior.
3. Classify resources by type:
   - descriptive feature resources,
   - dynamic-data resources,
   - tasking/control resources,
   - event/status resources,
   - supporting metadata resources.
4. Identify required and optional endpoints, representations, metadata, and resource relationships.
5. Capture source anchors and downstream handoffs.

**Expected Output:** Collection and resource endpoint inventory.

### Phase 3: Links, Relationships, and Navigation Pattern Extraction (3-4 hours)

**Objective:** Extract link behavior and navigation expectations across CSAPI resources.

**Tasks:**

1. Identify required and recommended link objects and link relations.
2. Identify navigation paths from root to collections, collections to items, and items to related resources.
3. Identify Part 1-to-Part 2 navigation relationships.
4. Identify parent-child, reverse, related, alternate, self, collection, item, service-desc, service-doc, conformance, schema, and other relevant link relations.
5. Identify which link relations are standards-defined, IANA-defined, OGC conventional, CSAPI-specific, or implementation-specific.
6. Record unresolved link relation or navigation questions.

**Expected Output:** Link relation and navigation pattern matrix.

### Phase 4: Resource Identity, URI, and Lifecycle Implication Review (1.5-2 hours)

**Objective:** Identify identity, URI, and lifecycle implications without finalizing those designs.

**Tasks:**

1. Identify URI patterns implied by collection and resource behavior.
2. Identify resources that require stable identifiers or canonical item URLs.
3. Identify where alternate identifiers, aliases, external identifiers, or resource lifecycle concerns arise.
4. Identify implications for deletion, tombstones, moved resources, stale links, and unavailable resources.
5. Hand detailed identity and lifecycle questions to `IDR-SRV-016`.

**Expected Output:** Identity/URI/lifecycle implication notes and handoff list.

### Phase 5: Validation, Interoperability, and Test Implication Analysis (2-2.5 hours)

**Objective:** Prepare findings for validation, interoperability, and test-driven design.

**Tasks:**

1. Identify link consistency and resource navigation validation needs.
2. Identify positive and negative tests for collection discovery, item retrieval, link traversal, and related-resource navigation.
3. Identify schema/OpenAPI validation implications.
4. Identify interoperability questions for existing implementation studies and external client tests.
5. Identify persistence/query implications for resource lookup, relationship traversal, and index design.

**Expected Output:** Navigation validation, interoperability, and test-implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert research output into a decision-usable collections/resources/links/navigation behavior baseline.

**Tasks:**

1. Consolidate collection, endpoint, link, navigation, identity, validation, and test findings.
2. Resolve conflicts and ambiguities where possible.
3. Produce findings grouped by resource family and navigation behavior.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Collection and item behavior requirements are identified with source anchors.
- [ ] CSAPI-specific resources and inherited OGC API - Features behaviors are distinguished.
- [ ] Required, recommended, conventional, and implementation-specific links are identified.
- [ ] Navigation patterns from root to collections, collections to items, and items to related resources are documented.
- [ ] Part 1 and Part 2 resource relationships are identified.
- [ ] URI, identifier, lifecycle, and relationship-model implications are handed to downstream topics.
- [ ] Validation, interoperability, and test implications are documented.
- [ ] Risks of broken navigation, stale links, inconsistent resource relationships, or incomplete discovery are identified.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Collections, Resources, Links, and Navigation Behavior Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-010-collections-resources-links-and-navigation-behavior-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Collection and resource endpoint findings
5. Link relation and navigation behavior findings
6. Part 1 / Part 2 resource relationship analysis
7. Resource identity, URI, and lifecycle implications
8. Validation and consistency implications
9. Interoperability and existing-implementation implications
10. Test-strategy implications
11. Downstream topic handoff matrix
12. Recommendations
13. Risks, constraints, and open questions
14. Validation against this plan's success criteria
15. References

The behavior matrix should include, at minimum:

- Resource or collection family
- Endpoint or navigation pattern
- Source standard / source anchor
- Requirement or convention summary
- Normative / recommended / conventional / implementation-choice classification
- Required or recommended links
- Related resource(s)
- Glaux Server implication
- Related conformance class or requirement
- Related schema/OpenAPI artifact, if applicable
- Test implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-006`, `IDR-SRV-007`, `IDR-SRV-008`, and `IDR-SRV-009` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, schema, and OpenAPI sources must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-010A: API Versioning, Backward Compatibility, and Deprecation Strategy`
- `IDR-SRV-011: Query, Filtering, Sorting, Pagination, and Selection Semantics`
- `IDR-SRV-012: Content Negotiation, Media Types, and Encoding Selection`
- `IDR-SRV-013: Error Model, HTTP Status Codes, and Failure Semantics`
- `IDR-SRV-014: OpenAPI Description and API Documentation Strategy`
- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy`
- `IDR-SRV-017: Relationship and Linkage Model`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-025: Database and Persistence Architecture Options`
- `IDR-SRV-026: Geospatial Storage and Query Strategy`
- `IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy`
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

- This topic should define navigation behavior expectations, not finalize the canonical resource model or database schema.
- Existing implementation studies may later refine practical expectations for link relations and traversal behavior.
- Link and navigation behavior should support both human-developer understanding and machine-client traversal.
- Open question: Which CSAPI resources must be first-class collections versus subordinate or linked resources?
- Open question: Which reverse links are required, recommended, or useful for interoperability?
- Open question: How should Glaux Server represent inaccessible, stale, deleted, moved, or externally hosted related resources?
- Open question: Which link patterns are necessary for compatibility with CSAPI Explorer and other clients?
- Risk: Incomplete or inconsistent linking could make the server technically populated but practically unusable by clients.
- Risk: Overly nested resource structures could make implementation brittle and client traversal difficult.
- Risk: Under-modeling relationships could weaken tasking, status, event, and observation traceability.

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
  - https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC schemas:
  - https://schemas.opengis.net/
- OGC API standards family:
  - https://ogcapi.ogc.org/
- OGC API - Features Part 1:
  - https://docs.ogc.org/is/17-069r4/17-069r4.html
- IANA Link Relation Types:
  - https://www.iana.org/assignments/link-relations/link-relations.xhtml
- RFC 8288 - Web Linking:
  - https://www.rfc-editor.org/rfc/rfc8288
- OpenAPI Specification:
  - https://spec.openapis.org/oas/latest.html
- W3C Semantic Sensor Network Ontology / SOSA:
  - https://www.w3.org/TR/vocab-ssn/
- OS4CSAPI testing research-plan corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans

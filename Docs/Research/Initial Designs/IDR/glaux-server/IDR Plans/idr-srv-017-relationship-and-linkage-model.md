# Section 017: Relationship and Linkage Model - Research Plan

**Status:** Planned  
**Last Updated:** June 8, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-017-relationship-and-linkage-model-report.md`

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

This topic research must define the Glaux Server planning baseline for the **relationship and linkage model** among canonical CSAPI resources, internal domain entities, external references, representation-level links, and queryable associations.

The research must answer:

- What relationships must Glaux Server represent among systems, procedures, deployments, sampling features, properties, datastreams, observations, status resources, system events, control streams, commands, feasibility resources, and command status resources?
- Which relationships are normative CSAPI relationships, inherited OGC API / Web linking relationships, SensorML relationships, SWE Common relationships, AEP-4789 server-responsibility relationships, implementation-support relationships, or derived relationships?
- How should relationships be exposed through links, identifiers, URI references, embedded objects, query parameters, collection membership, nested paths, reverse links, and relationship-specific resources?
- Which relationships are stable, temporal, contextual, versioned, derived, many-to-many, hierarchical, federated, policy-constrained, or state-dependent?
- How should Glaux Server support link-following clients, generated clients, CSAPI Explorer, external OGC API tooling, conformance tests, and DDIL/federated operation without over-embedding resource graphs?
- What downstream implications follow for temporal modeling, status/event modeling, SensorML/SWE representation, persistence, query planning, authorization, validation, fixtures, and interoperability testing?

The output must be a relationship and linkage model baseline with source anchors, relationship classes, resource-family mappings, representation guidance, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows:

- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy`

`IDR-SRV-015` defines canonical resource families and domain boundaries. `IDR-SRV-016` defines identifiers, addressing, and lifecycle. This topic defines how those resources relate to one another and how relationships should be represented externally and internally. It must precede temporal modeling, registration/update semantics, status/event modeling, persistence design, query strategy, validation, conformance harness design, fixture planning, and interoperability testing.

### Critical Constraint(s)

- Treat OGC API - Connected Systems Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, Web linking, URI/HTTP semantics, AEP-4789 Volume II adoption context, and earlier IDR findings as controlling.
- Do not invent relationship semantics that conflict with CSAPI resource relationships, OGC API - Features collection/item behavior, SensorML process/system semantics, SWE Common data component semantics, or Web linking conventions.
- Do not finalize database graph schema or storage structures here. Identify persistence implications and hand them to Category E topics.
- Do not finalize temporal/freshness semantics here. Identify temporal relationship needs and hand them to `IDR-SRV-018`.
- Do not finalize registration/update behavior here. Identify provenance implications for `IDR-SRV-019` and hand operation semantics to `IDR-SRV-031`.
- Do not finalize status/event semantics here. Identify status/event relationship needs and hand them to `IDR-SRV-020`.
- Do not finalize authorization/policy model here. Identify relationship-level access-control and releasability implications and hand them to Category G.
- Distinguish relationship semantics from URI hierarchy. Nested paths may express access patterns but do not automatically define ownership or lifecycle.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What relationship classes must Glaux Server support across canonical CSAPI resource families?
2. Which relationships are normative, inherited, representation-specific, derived, implementation-support, optional, or unresolved?
3. How should relationships be exposed in API responses and navigated by clients?
4. Which relationships require temporal, lifecycle, policy, provenance, or version context?
5. What downstream implications follow for temporal modeling, persistence, validation, conformance, fixtures, security, and interoperability?

### Detailed Questions

#### Standards and Relationship Sources

- What relationships are defined or implied by CSAPI Part 1?
- What relationships are defined or implied by CSAPI Part 2?
- What relationship and link behavior is inherited from OGC API - Features?
- What Web linking rules, link relation conventions, and URI-reference behavior apply?
- What SensorML system/procedure/process/component relationships are relevant?
- What SWE Common data component, observed property, unit, command parameter, and result-structure relationships are relevant?
- What AEP-4789 server-responsibility relationships are relevant?
- Which relationships appear in implementation studies or community lessons but are not clearly standards-defined?

#### Canonical Relationship Classes

- What relationship classes exist among:
  - systems and procedures,
  - systems and deployments,
  - systems and sampling features,
  - systems and properties,
  - systems and datastreams,
  - systems and observations,
  - systems and status resources,
  - systems and system events,
  - systems and control streams,
  - systems and commands,
  - systems and feasibility resources,
  - procedures and deployments,
  - deployments and platforms,
  - datastreams and observed properties,
  - observations and features of interest,
  - commands and control streams,
  - commands and status/history,
  - events and affected resources?
- Which relationships are parent/child, associative, derived, temporal, many-to-many, optional, contextual, or workflow-driven?
- Which relationships imply lifecycle dependency, and which merely imply navigational association?
- Which relationships must be queryable?

#### Link Representation Strategy

- Which relationships should be represented using OGC-style links?
- Which link relation types are standard, registered, domain-specific, or implementation-specific?
- Which relationships should use `self`, `alternate`, `collection`, `item`, `service-desc`, `service-doc`, `conformance`, `data`, `describedby`, or other registered link relations?
- Which relationships require custom relation URIs or profile-specific semantics?
- Which relationships should be embedded, linked, referenced by ID, exposed through query parameters, exposed through related collections, or avoided in responses?
- How should links behave across content negotiation, alternate representations, API versions, retired resources, and policy-constrained resources?

#### Directionality, Reverse Links, and Traversal

- Which relationships must be navigable in both directions?
- Which reverse links are required, recommended, optional, or potentially expensive?
- How should Glaux Server avoid over-embedding graphs while still enabling practical client traversal?
- How should clients discover related systems, datastreams, observations, commands, events, and status resources?
- Which reverse relationships should be query endpoints rather than embedded links?
- How should link traversal behave for generated clients, CSAPI Explorer, mobile clients, and tactical-edge clients?

#### Temporal and Lifecycle Context

- Which relationships are time-dependent?
- How should relationships account for deployment intervals, system configurations, sensor replacement, status freshness, observation time, event time, command lifecycle, and resource retirement?
- Which relationships represent current state versus historical state?
- How should relationship validity be represented or queried?
- Which findings should be handed to `IDR-SRV-018`, `IDR-SRV-019`, and `IDR-SRV-020`?

#### Provenance, Federation, and External References

- How should Glaux Server represent relationships to external systems, publisher feeds, simulator outputs, imported SensorML documents, external identifiers, and federated resources?
- How should external links be distinguished from local canonical links?
- How should broken, stale, unavailable, or restricted external relationships be represented?
- How should DDIL-informed operation affect relationship availability and link resolution?
- Which findings should be handed to federation, DDIL, publisher, and persistence topics?

#### Policy, Security, and Releasability Implications

- How should relationship representation avoid leaking restricted resource existence?
- How should links be filtered or suppressed based on authorization, releasability, classification, or need-to-know?
- How should relationship traversal be audited for sensitive command/control or cross-boundary resources?
- Which relationships may create inference risks?
- Which findings should be handed to `IDR-SRV-039`, `IDR-SRV-040`, and `IDR-SRV-055`?

#### Persistence, Query, and Validation Implications

- Which relationships must be persisted, indexed, derived, cached, materialized, or computed on demand?
- What relationship queries must be supported?
- What validation rules apply to relationship targets, link consistency, reverse-link consistency, lifecycle states, and temporal validity?
- What fixtures and golden files are needed to test relationship behavior?
- Which findings should be handed to Category E and Category I topics?

#### Implementation and Interoperability Lessons

- What relationship and link lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate link traversal, missing relationship, broken link, inconsistent identifier, or embedded-resource issues?
- What OS4CSAPI discussion lessons affect relationship and linkage strategy?
- Which implementation patterns should Glaux Server adopt, avoid, or investigate further?
- Which findings must be treated as implementation-specific rather than standards-derived?

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

- `IDR-SRV-001` through `IDR-SRV-016` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Controlling Standards Sources

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/

### HTTP, URI, and Linking Sources

- RFC 3986 - Uniform Resource Identifier: Generic Syntax: https://www.rfc-editor.org/rfc/rfc3986
- RFC 3987 - Internationalized Resource Identifiers: https://www.rfc-editor.org/rfc/rfc3987
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 8288 - Web Linking: https://www.rfc-editor.org/rfc/rfc8288
- IANA Link Relation Types: https://www.iana.org/assignments/link-relations/link-relations.xhtml
- IANA Media Types Registry: https://www.iana.org/assignments/media-types/media-types.xhtml

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, or standards-package annexes relevant to resource relationships, server responsibilities, and navigation behavior

### Implementation and Lessons-Learned Sources

Use implementation-study outputs and source repositories as non-normative evidence:

- OSH / OpenSensorHub sources and `IDR-SRV-014A`
- Connected Systems Go sources and `IDR-SRV-014B`
- pygeoapi sources and `IDR-SRV-014C`
- SECD sources and `IDR-SRV-014D`
- OS4CSAPI Client Smoke Test Findings and `IDR-SRV-014E`
- SECD Interoperability Findings and `IDR-SRV-014F`
- OS4CSAPI Discussions Lessons-Learned and `IDR-SRV-014G`
- Canonical Glaux Server Resource Model findings from `IDR-SRV-015`
- Identifier, URI, and Resource Lifecycle findings from `IDR-SRV-016`
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/

---

## 4. Supporting Resources

Use these sources to interpret project context, downstream dependencies, expected research depth, and report style.

- DGIWG P5.07 Glaux main repository: https://github.com/DGIWG-P507/glaux
- Glaux project website: https://dgiwg-p507.github.io/glaux/
- DGIWG P5.07 GitHub organization: https://github.com/DGIWG-P507
- Glaux Server repository, if available or created: https://github.com/DGIWG-P507/glaux-server
- OS4CSAPI testing research-plan corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

---

## 5. Research Methodology

### Phase 1: Source Collection and Relationship Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for relationship modeling.

**Tasks:**

1. Gather standards, schemas, OpenAPI artifacts, prior IDR reports, implementation studies, smoke-test findings, interoperability findings, and discussion lessons.
2. Extract all relationship, link, reference, association, containment, hierarchy, and traversal concepts from each source.
3. Define relationship inventory fields:
   - source resource,
   - target resource,
   - relationship name,
   - relationship class,
   - source authority,
   - directionality,
   - cardinality,
   - temporal validity,
   - lifecycle dependency,
   - representation pattern,
   - link relation,
   - queryability,
   - security implication,
   - validation/test implication.
4. Define classification values:
   - normative,
   - inherited,
   - standards-package-specific,
   - representation-specific,
   - derived,
   - implementation-support,
   - implementation-specific,
   - optional,
   - unresolved.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation observations, and community discussion.

**Expected Output:** Source inventory and relationship extraction framework.

### Phase 2: Standards-Derived Relationship Extraction (2.5-3.5 hours)

**Objective:** Extract relationship expectations from standards and official artifacts.

**Tasks:**

1. Extract CSAPI Part 1 relationships among feature resources.
2. Extract CSAPI Part 2 relationships among dynamic data, observations, datastreams, control streams, commands, feasibility, status, and events.
3. Extract inherited OGC API - Features collection/item and link behavior.
4. Extract SensorML relationships relevant to systems, procedures, deployments, capabilities, characteristics, inputs, outputs, positions, contacts, and components.
5. Extract SWE Common relationships relevant to data components, observed properties, values, units, command parameters, and result structures.
6. Extract Web linking and registered link relation guidance.
7. Extract AEP-4789 relationship implications from project-available material.

**Expected Output:** Standards-derived relationship inventory.

### Phase 3: Canonical Relationship Model Analysis (3-4 hours)

**Objective:** Define relationship classes and relationship behavior by resource family.

**Tasks:**

1. Use `IDR-SRV-015` canonical resource families and `IDR-SRV-016` identifier/URI findings as the organizing structure.
2. Map source-to-target relationships among all relevant resource families.
3. Identify directionality, cardinality, lifecycle dependency, temporal validity, queryability, and representation patterns.
4. Identify which relationships should be externally exposed, internally maintained, derived, suppressed, or policy-filtered.
5. Identify unresolved relationship questions requiring downstream research or prototype validation.

**Expected Output:** Canonical relationship model matrix.

### Phase 4: Link Representation and Traversal Analysis (2.5-3 hours)

**Objective:** Define how relationships should be exposed and traversed by clients.

**Tasks:**

1. Identify link relation types and URI-reference patterns for each externally visible relationship.
2. Analyze embedded versus linked versus queryable versus relationship-resource representation choices.
3. Identify reverse-link and traversal requirements.
4. Identify content negotiation, alternate representation, versioning, policy-filtering, stale-link, and external-link behavior.
5. Identify client and tool expectations from CSAPI Explorer, generated clients, mobile clients, and OGC API clients.

**Expected Output:** Link representation and traversal guidance.

### Phase 5: Temporal, Lifecycle, Persistence, Security, Validation, and Test Implications (2.5-3 hours)

**Objective:** Prepare relationship findings for downstream topics.

**Tasks:**

1. Identify temporal and lifecycle contexts for each relationship class.
2. Identify persistence, indexing, graph, query, and cache implications without designing the database schema.
3. Identify validation rules for relationship targets, cardinality, link consistency, reverse links, lifecycle states, and temporal validity.
4. Identify security and policy risks from link exposure and relationship traversal.
5. Identify conformance, golden-file, fixture, negative, and interoperability tests.
6. Map findings to Category C, D, E, F, G, and I topics.

**Expected Output:** Downstream relationship implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert relationship and linkage research into a decision-usable baseline.

**Tasks:**

1. Consolidate standards-derived relationships, canonical relationship model, link representation guidance, traversal guidance, and downstream implications.
2. Produce relationship model recommendations by resource family and relationship class.
3. Resolve conflicts and ambiguities where possible.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Relationship classes are identified with source anchors.
- [ ] Source and target resource families are mapped.
- [ ] Normative, inherited, derived, implementation-support, implementation-specific, optional, and unresolved relationships are distinguished.
- [ ] Directionality, cardinality, lifecycle dependency, temporal validity, and queryability are documented.
- [ ] Link representation and traversal guidance is documented.
- [ ] SensorML, SWE Common, Web linking, and OGC API implications are documented.
- [ ] Persistence, validation, security, conformance, fixture, and interoperability implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Relationship and Linkage Model Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-017-relationship-and-linkage-model-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Relationship extraction methodology
5. Standards-derived relationship inventory
6. Canonical relationship classes
7. Resource-family relationship matrix
8. Directionality, cardinality, lifecycle, and temporal findings
9. Link relation and representation findings
10. Traversal and reverse-link findings
11. SensorML and SWE Common relationship implications
12. External reference, federation, DDIL, and stale-link findings
13. Security, policy, and releasability implications
14. Persistence, validation, fixture, and test implications
15. Downstream topic handoff matrix
16. Recommendations
17. Risks, constraints, and open questions
18. Validation against this plan's success criteria
19. References

The relationship matrix should include, at minimum:

- Relationship identifier
- Source resource family
- Target resource family
- Relationship name / class
- Source standard / source anchor
- Authority classification
- Directionality
- Cardinality
- Lifecycle dependency
- Temporal validity
- Link relation / representation pattern
- Queryability
- Security/policy implication
- Persistence implication
- Validation implication
- Test implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-016` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, OGC schemas, OpenAPI artifacts, URI/HTTP/Web linking standards, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-018: Temporal, Validity, and Freshness Model`
- `IDR-SRV-019: Provenance, Lineage, Quality, and Trust Metadata Model`
- `IDR-SRV-020: Status, Availability, and System Event Model`
- `IDR-SRV-021: SensorML Representation Strategy`
- `IDR-SRV-022: SWE Common Data Component Strategy`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-024: Units, Observed Properties, and Semantic Binding Strategy`
- `IDR-SRV-025: Database and Persistence Architecture Options`
- `IDR-SRV-026: Geospatial Storage and Query Strategy`
- `IDR-SRV-027: Time-Series Observation Storage Strategy`
- `IDR-SRV-028: Metadata and Document Storage Strategy`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints`
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

- This topic defines relationship and linkage strategy, not the database graph schema.
- Nested URI paths should not be assumed to imply ownership or lifecycle dependency without standards support.
- Implementation-study findings are useful but must not override standards-derived relationship semantics.
- Open question: Which relationships must be materialized versus derived at request time?
- Open question: Which reverse links are required for practical interoperability versus optional convenience?
- Open question: Which relationship types need temporal validity intervals?
- Open question: How should relationship links be filtered for policy, releasability, and DDIL conditions?
- Open question: Which link relation types require custom relation URIs?
- Risk: Under-modeling relationships could break client traversal and interoperability.
- Risk: Over-embedding relationships could create large responses, stale data, and authorization leakage.
- Risk: Treating one implementation's link pattern as canonical could distort Glaux Server design.
- Risk: Failing to validate links and relationship targets could lead to broken API navigation and unreliable tests.

---

## References

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/
- RFC 3986 - Uniform Resource Identifier: Generic Syntax: https://www.rfc-editor.org/rfc/rfc3986
- RFC 3987 - Internationalized Resource Identifiers: https://www.rfc-editor.org/rfc/rfc3987
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 8288 - Web Linking: https://www.rfc-editor.org/rfc/rfc8288
- IANA Link Relation Types: https://www.iana.org/assignments/link-relations/link-relations.xhtml
- IANA Media Types Registry: https://www.iana.org/assignments/media-types/media-types.xhtml
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

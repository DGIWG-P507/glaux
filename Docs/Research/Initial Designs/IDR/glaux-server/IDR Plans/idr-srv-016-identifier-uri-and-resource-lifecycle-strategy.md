# Section 016: Identifier, URI, and Resource Lifecycle Strategy - Research Plan

**Status:** Planned  
**Last Updated:** June 8, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-016-identifier-uri-and-resource-lifecycle-strategy-report.md`

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

This topic research must define the Glaux Server planning baseline for **identifier, URI, and resource lifecycle strategy**.

The research must answer:

- What identifier strategy should Glaux Server use for canonical CSAPI resources, internal domain entities, externally supplied identifiers, persistent references, and generated resources?
- What URI patterns should Glaux Server expose for collections, items, nested resources, related resources, dynamic data, commands, feasibility, status, and system events?
- How should resource identity remain stable across updates, deployments, version changes, imports, publishers, simulator data, federation, caching, and DDIL-informed operation?
- How should Glaux Server distinguish resource identifiers, URI paths, external identifiers, alternate identifiers, SensorML identifiers, URI/URN references, database primary keys, links, and derived resource IDs?
- What lifecycle semantics should be defined for resource creation, registration, update, retirement, deprecation, archival, deletion, replacement, command lifecycle, observation retention, status freshness, and event history?
- What downstream implications follow for relationships, temporal validity, registration/update semantics, status/events, persistence, security, validation, conformance, fixtures, and interoperability?

The output must be an identifier, URI, and lifecycle strategy baseline with source anchors, resource-family mappings, lifecycle classifications, persistence and validation implications, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows `IDR-SRV-015: Canonical Glaux Server Resource Model`.

`IDR-SRV-015` defines the canonical resource families and API/domain boundaries. This topic defines how those resources are identified, addressed, referenced, created, updated, retired, archived, and preserved over time. It must precede relationship modeling, temporal/freshness modeling, registration/update semantics, status/event semantics, persistence design, validation, fixtures, and interoperability testing.

### Critical Constraint(s)

- Treat OGC API - Connected Systems Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, URI/IRI/HTTP semantics, AEP-4789 Volume II adoption context, and earlier IDR findings as controlling.
- Do not invent URI or identifier patterns that conflict with CSAPI, OGC API - Features, Web linking, HTTP semantics, or the canonical Glaux Server resource model.
- Do not design the database schema here. Identify persistence implications and hand them to Category E topics.
- Do not finalize relationship/link behavior here. Identify link and relationship needs and hand detailed relationship modeling to `IDR-SRV-017`.
- Do not finalize temporal/freshness semantics here. Identify lifecycle-time requirements and hand detailed temporal modeling to `IDR-SRV-018`.
- Do not finalize registration/update operation semantics here. Identify provenance implications for `IDR-SRV-019` and hand operation semantics to `IDR-SRV-031`.
- Do not finalize security policy here. Identify identifier and lifecycle risks and hand security implications to Category G and test topics.
- Do not conflate identifiers with URLs, database keys, SensorML identifiers, or human labels. Distinguish them explicitly.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What identifier classes must Glaux Server support?
2. What URI patterns should Glaux Server expose for canonical resource families?
3. How should stable identity be maintained across lifecycle changes, updates, imports, versions, publishers, and deployments?
4. What lifecycle states and transitions are needed for each resource family?
5. What downstream implications follow for relationships, temporal modeling, registration/update behavior, persistence, validation, conformance, fixtures, security, and interoperability?

### Detailed Questions

#### Identifier Classes and Authority

- What identifiers are defined or implied by CSAPI Part 1?
- What identifiers are defined or implied by CSAPI Part 2?
- What identifier behavior is inherited from OGC API - Features?
- What identifiers are used in SensorML systems, processes, procedures, components, contacts, capabilities, characteristics, and references?
- What identifiers are used in SWE Common data components, observed properties, units, and command parameters?
- What identifiers are relevant from STANAG 4789 / AEP-4789 Volume II server responsibilities?
- Which identifiers are globally persistent, server-local, collection-local, externally supplied, generated, derived, temporary, or implementation-private?
- Which identifier sources are authoritative when conflicts occur?

#### URI and Resource Addressing Strategy

- What canonical URI patterns should Glaux Server use for:
  - landing page,
  - API definition,
  - conformance declaration,
  - collections,
  - systems,
  - procedures,
  - deployments,
  - sampling features,
  - properties,
  - datastreams,
  - observations,
  - status and dynamic properties,
  - system events,
  - control streams,
  - commands,
  - command status,
  - feasibility resources?
- Which resources should be addressed as top-level collections, subcollections, nested resources, related resources, or queryable views?
- How should URI paths relate to resource IDs?
- How should Glaux Server handle aliases, alternate identifiers, canonical links, redirects, deprecated routes, and non-canonical access paths?
- How should URI patterns support generated clients, CSAPI Explorer, conformance tests, and external OGC API tooling?

#### Identifier Stability, Scope, and Collision Handling

- What identifier stability guarantees are required for each resource family?
- How should Glaux Server handle externally supplied identifiers from publishers, simulators, imported SensorML, legacy systems, or federated sources?
- How should collisions between identifiers be detected and resolved?
- Should Glaux Server mint its own canonical IDs even when external IDs exist?
- How should alternate identifiers be preserved and exposed?
- How should identity survive resource updates, metadata changes, deployment changes, and source-system changes?
- How should identity behave in disconnected or intermittently connected environments?

#### Resource Lifecycle Semantics

- Which resource families are created, imported, registered, updated, versioned, retired, archived, deleted, replaced, derived, generated, observed, commanded, or expired?
- Which lifecycle states are needed for stable descriptions such as systems, procedures, deployments, sampling features, and properties?
- Which lifecycle states are needed for dynamic data such as observations, datastreams, status resources, events, commands, and feasibility resources?
- Which lifecycle transitions should be externally visible?
- Which transitions should be internal implementation details?
- How should lifecycle semantics support auditability, reproducibility, and conformance testing?
- How should lifecycle semantics differ between deletion, deactivation, archival, retirement, replacement, and inaccessibility due to policy?

#### Versioning, Revision, and Replacement

- How should resource updates be represented without breaking stable references?
- What resource versioning or revision concepts are implied by CSAPI, SensorML, SWE Common, AEP-4789, implementation studies, or interoperability findings?
- When should Glaux Server preserve historical versions?
- When should resource replacement produce a new identifier?
- How should deprecated resources or paths be represented?
- How should API versioning findings from `IDR-SRV-010A` affect resource identity and lifecycle?
- How should lifecycle choices support reproducible observations, command audit, and historical traceability?

#### Relationship, Linkage, and Reference Implications

- How do identifiers and URIs support relationships among systems, procedures, deployments, sampling features, properties, datastreams, observations, status, events, control streams, commands, and feasibility resources?
- Which relationships require stable identifiers even if URI paths change?
- Which relationships should use links, IDs, URI references, embedded objects, or queryable relations?
- What identifier and URI findings must be handed to `IDR-SRV-017`?
- How should identifier strategy support graph traversal, client link-following, federation, caching, and external references?

#### Temporal, Freshness, Status, and Event Implications

- Which lifecycle states require temporal attributes?
- How do created, updated, valid, retired, archived, observed, ingested, published, and expired times relate to identifiers and lifecycle?
- How should URI and identifier strategy support current-state views versus historical records?
- How should stale, last-known, unavailable, retired, and superseded resources be represented?
- What findings should be handed to `IDR-SRV-018` and `IDR-SRV-020`?

#### Persistence, Validation, and Security Implications

- What persistence requirements follow from stable identifiers and lifecycle history?
- What indexing requirements follow from identifier lookup, alternate identifiers, and cross-resource relationships?
- What validation rules are needed for identifier format, uniqueness, URI construction, lifecycle transitions, and alternate identifiers?
- What security concerns arise from predictable identifiers, resource enumeration, hidden resource existence, deleted resources, retired resources, and cross-boundary identity?
- What findings should be handed to Category E, Category G, and Category I topics?

#### Implementation and Interoperability Lessons

- What identifier, URI, and lifecycle lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate identifier, path, link, lifecycle, or canonical URL problems?
- What OS4CSAPI discussion lessons affect identifier or lifecycle strategy?
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

- `IDR-SRV-001` through `IDR-SRV-015` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Controlling Standards Sources

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/

### URI, HTTP, Linking, and Identifier Sources

- RFC 3986 - Uniform Resource Identifier: Generic Syntax: https://www.rfc-editor.org/rfc/rfc3986
- RFC 3987 - Internationalized Resource Identifiers: https://www.rfc-editor.org/rfc/rfc3987
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 8288 - Web Linking: https://www.rfc-editor.org/rfc/rfc8288
- RFC 4122 - UUID URN Namespace: https://www.rfc-editor.org/rfc/rfc4122
- RFC 9562 - Universally Unique IDentifiers: https://www.rfc-editor.org/rfc/rfc9562
- IANA Link Relation Types: https://www.iana.org/assignments/link-relations/link-relations.xhtml
- IANA URI Schemes: https://www.iana.org/assignments/uri-schemes/uri-schemes.xhtml

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, or standards-package annexes relevant to identifiers, resource lifecycle, server responsibilities, or resource addressing

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

### Phase 1: Source Collection and Identifier/Lifecycle Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework.

**Tasks:**

1. Gather standards, schemas, OpenAPI artifacts, prior IDR reports, implementation studies, smoke-test findings, interoperability findings, and discussion lessons.
2. Extract identifier, URI, reference, link, lifecycle, deletion, update, archival, replacement, and versioning concepts from each source.
3. Define inventory fields:
   - resource family,
   - identifier type,
   - identifier scope,
   - URI pattern,
   - canonical/non-canonical status,
   - lifecycle state,
   - lifecycle transition,
   - source authority,
   - stability guarantee,
   - security implication,
   - validation/test implication.
4. Define classification values:
   - normative,
   - inherited,
   - standards-package-specific,
   - recommended,
   - implementation-support,
   - implementation-specific,
   - optional,
   - unresolved.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation observations, and community discussion.

**Expected Output:** Source inventory and identifier/lifecycle extraction framework.

### Phase 2: Standards-Derived Identifier and URI Extraction (2.5-3.5 hours)

**Objective:** Extract identifier and URI expectations from standards and official artifacts.

**Tasks:**

1. Extract CSAPI Part 1 identifier and URI behavior.
2. Extract CSAPI Part 2 identifier and URI behavior for dynamic data, commands, feasibility, status, and events.
3. Extract inherited OGC API - Features identifier, collection, item, and link behavior.
4. Extract SensorML identifier and reference behavior relevant to systems and procedures.
5. Extract SWE Common identifier and reference behavior relevant to data components and properties.
6. Extract URI/link/HTTP semantics from RFC and IANA sources.
7. Extract AEP-4789 identifier and server-addressing implications from project-available material.

**Expected Output:** Standards-derived identifier and URI behavior inventory.

### Phase 3: Resource Family Identifier and URI Strategy Analysis (3-4 hours)

**Objective:** Define identifier and URI strategy by canonical resource family.

**Tasks:**

1. Use `IDR-SRV-015` resource families as the organizing structure.
2. For each resource family, identify canonical ID needs, alternate ID needs, external ID needs, URI paths, collection/item relationships, nested/related-resource questions, and canonical-link requirements.
3. Identify collision, alias, redirect, replacement, deprecation, and non-canonical path cases.
4. Identify generated-client, CSAPI Explorer, conformance, and interoperability implications.
5. Identify unresolved strategy questions requiring downstream research or prototype validation.

**Expected Output:** Resource-family identifier and URI strategy matrix.

### Phase 4: Lifecycle, Versioning, and Stability Analysis (2.5-3.5 hours)

**Objective:** Define lifecycle strategy and stability expectations.

**Tasks:**

1. Identify lifecycle states and transitions for stable descriptions, dynamic data, status/events, commands, feasibility, and derived resources.
2. Distinguish creation, import, registration, update, replacement, retirement, archival, deletion, deprecation, and inaccessibility.
3. Analyze versioning and revision needs for resources and API paths.
4. Identify identity stability guarantees and historical traceability needs.
5. Identify handoffs to registration/update semantics, temporal/freshness modeling, status/event modeling, command lifecycle, and audit/security topics.

**Expected Output:** Lifecycle and stability strategy matrix.

### Phase 5: Implementation Lessons, Persistence, Validation, Security, and Test Implications (2.5-3 hours)

**Objective:** Incorporate implementation and interoperability lessons without making them normative.

**Tasks:**

1. Review `IDR-SRV-014A` through `IDR-SRV-014G` findings and `IDR-SRV-015` for identifier, URI, and lifecycle lessons.
2. Identify implementation patterns to adopt, avoid, or investigate.
3. Identify persistence and indexing implications without designing the database schema.
4. Identify validation rules for identifier formats, uniqueness, URI construction, canonical links, and lifecycle transitions.
5. Identify security risks from predictable IDs, resource enumeration, unauthorized resources, deletion, archival, hidden resources, and cross-boundary references.
6. Identify conformance, golden-file, fixture, external-client, and regression tests.

**Expected Output:** Implementation lesson and downstream implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert identifier, URI, and lifecycle research into a decision-usable baseline.

**Tasks:**

1. Consolidate standards-derived inventory, resource-family strategy, lifecycle strategy, stability findings, and implementation lessons.
2. Produce identifier, URI, and lifecycle guidance by resource family.
3. Resolve conflicts and ambiguities where possible.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Identifier classes are identified and distinguished.
- [ ] URI patterns and canonical addressing needs are mapped by resource family.
- [ ] Resource identifiers, URI paths, external identifiers, alternate identifiers, SensorML identifiers, database keys, and links are distinguished.
- [ ] Lifecycle states and transitions are identified by resource family.
- [ ] Stability, collision, alias, replacement, deprecation, archival, and deletion implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Persistence, validation, security, conformance, fixture, and interoperability implications are documented.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Identifier, URI, and Resource Lifecycle Strategy Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-016-identifier-uri-and-resource-lifecycle-strategy-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Identifier and lifecycle extraction methodology
5. Standards-derived identifier and URI behavior inventory
6. Identifier class definitions and scope findings
7. Resource-family URI and canonical addressing strategy
8. External, alternate, and generated identifier findings
9. Collision, alias, redirect, deprecation, and replacement findings
10. Resource lifecycle and state-transition findings
11. Versioning, revision, and identity stability findings
12. Relationship, temporal, status, event, and command implications
13. Persistence, validation, security, fixture, and test implications
14. Downstream topic handoff matrix
15. Recommendations
16. Risks, constraints, and open questions
17. Validation against this plan's success criteria
18. References

The identifier/lifecycle matrix should include, at minimum:

- Resource family
- Identifier class
- Identifier scope and authority
- Canonical URI pattern
- Alternate/external identifier handling
- Stability expectation
- Lifecycle states
- Lifecycle transitions
- Collision or alias behavior
- Versioning/replacement behavior
- Security implication
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
- `IDR-SRV-001` through `IDR-SRV-015` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, OGC schemas, OpenAPI artifacts, URI/HTTP/linking standards, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-017: Relationship and Linkage Model`
- `IDR-SRV-018: Temporal, Validity, and Freshness Model`
- `IDR-SRV-019: Provenance, Lineage, Quality, and Trust Metadata Model`
- `IDR-SRV-020: Status, Availability, and System Event Model`
- `IDR-SRV-025: Database and Persistence Architecture Options`
- `IDR-SRV-026: Geospatial Storage and Query Strategy`
- `IDR-SRV-027: Time-Series Observation Storage Strategy`
- `IDR-SRV-028: Metadata and Document Storage Strategy`
- `IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy`
- `IDR-SRV-030: Data Lifecycle, Retention, Archival, and Deletion Strategy`
- `IDR-SRV-047: Configuration, Secrets, and Environment Strategy`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
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

- This topic defines identifier, URI, and lifecycle strategy, not the database schema.
- This topic should preserve stable resource identity while allowing full-scope server behavior and future sequencing.
- Implementation-study findings are useful but must not override standards-derived resource semantics.
- Open question: Should Glaux Server mint internal canonical IDs for all resources even when external identifiers exist?
- Open question: Which resource replacements require new identifiers versus revisions of existing identifiers?
- Open question: How should Glaux Server preserve alternate identifiers from SensorML, publishers, simulators, and federated sources?
- Open question: Which deletion and archival behaviors are compatible with operational traceability and policy constraints?
- Open question: How should identifier stability behave under disconnected, degraded, or federated operation?
- Risk: Unstable identifiers could break clients, cached links, historical observations, and conformance tests.
- Risk: Overly predictable identifiers could enable enumeration or unauthorized inference.
- Risk: Treating URI paths as the only identity mechanism could make future versioning, federation, and lifecycle transitions brittle.
- Risk: Overcomplicated identifier strategy could burden implementation and testing without interoperability benefit.

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
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/
- RFC 3986 - Uniform Resource Identifier: Generic Syntax: https://www.rfc-editor.org/rfc/rfc3986
- RFC 3987 - Internationalized Resource Identifiers: https://www.rfc-editor.org/rfc/rfc3987
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 8288 - Web Linking: https://www.rfc-editor.org/rfc/rfc8288
- RFC 4122 - UUID URN Namespace: https://www.rfc-editor.org/rfc/rfc4122
- RFC 9562 - Universally Unique IDentifiers: https://www.rfc-editor.org/rfc/rfc9562
- IANA Link Relation Types: https://www.iana.org/assignments/link-relations/link-relations.xhtml
- IANA URI Schemes: https://www.iana.org/assignments/uri-schemes/uri-schemes.xhtml
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

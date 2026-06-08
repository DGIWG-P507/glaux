# Section 014: OpenAPI Description and API Documentation Strategy - Research Plan

**Status:** Planned  
**Last Updated:** June 8, 2026  
**Estimated Research Time:** 10-14 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014-openapi-description-and-api-documentation-strategy-report.md`

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

This topic research must define the Glaux Server planning baseline for **OpenAPI description and API documentation strategy**.

The research must answer:

- What OpenAPI descriptions must or should Glaux Server publish to support OGC API - Connected Systems Part 1, OGC API - Connected Systems Part 2, SensorML, SWE Common, and inherited OGC API behavior?
- How should OpenAPI documentation reflect live server behavior, conformance declarations, resource collections, query parameters, content negotiation, schemas, request/response media types, error responses, security schemes, and dynamic-data/tasking operations?
- How should Glaux Server distinguish normative API contract material, generated documentation, human-readable developer guidance, examples, implementation notes, and future or experimental extensions?
- How should the API documentation strategy support external clients, generated clients, conformance testing, CSAPI Explorer, DGIWG/NATO implementers, and future Glaux ecosystem components?
- What validation, build, publishing, and maintenance implications follow from treating the OpenAPI description as a machine-readable server contract?

The output must be a documentation and machine-contract publication baseline with source anchors, OpenAPI strategy recommendations, downstream handoffs, and test implications.

### Why This Topic Order

This topic follows the core API behavior topics:

- `IDR-SRV-009: Landing Page, API Definition, and Conformance Declaration Behavior`
- `IDR-SRV-010: Collections, Resources, Links, and Navigation Behavior`
- `IDR-SRV-010A: API Versioning, Backward Compatibility, and Deprecation Strategy`
- `IDR-SRV-011: Query, Filtering, Sorting, Pagination, and Selection Semantics`
- `IDR-SRV-012: Content Negotiation, Media Types, and Encoding Selection`
- `IDR-SRV-013: Error Model, HTTP Status Codes, and Failure Semantics`

Those topics define the behavior the OpenAPI description must accurately represent. This topic defines how that behavior becomes discoverable, machine-readable, documented, and testable.

This topic must precede existing implementation studies and downstream resource-model, validation, conformance-harness, and interoperability-test topics because those topics depend on a clear API contract and documentation baseline.

### Critical Constraint(s)

- Treat OGC API - Connected Systems, OGC API - Features, OpenAPI, schemas, HTTP, and the prior Glaux Server IDR behavior topics as controlling where they define API documentation obligations.
- Do not use OpenAPI documentation to invent behavior not supported by the standards or server design.
- Do not document planned, experimental, or deferred behavior as implemented behavior.
- Do not finalize implementation framework choices here; keep this research focused on contract/documentation requirements and implications.
- Clearly distinguish:
  - normative OpenAPI contract,
  - human-readable documentation,
  - examples,
  - generated documentation,
  - schema references,
  - conformance declarations,
  - security descriptions,
  - versioning/deprecation notices,
  - implementation-specific extensions.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What OpenAPI description behavior is required or expected for Glaux Server?
2. How should OpenAPI artifacts represent CSAPI Part 1 and Part 2 resources, operations, schemas, parameters, media types, and errors?
3. How should Glaux Server publish, version, validate, and keep OpenAPI descriptions synchronized with implementation behavior?
4. How should human-readable API documentation relate to the machine-readable OpenAPI contract?
5. What downstream validation, conformance, interoperability, generated-client, and test implications follow from the OpenAPI/documentation baseline?

### Detailed Questions

#### Standards and Documentation Sources

- What API description behavior is required or implied by OGC API - Connected Systems Part 1 and Part 2?
- What API description behavior is inherited from OGC API - Features?
- What OpenAPI version and artifact structure are used by the official CSAPI materials?
- What schema references and reusable components appear in official OpenAPI artifacts?
- Which official OpenAPI artifacts should Glaux Server treat as reference material, source material, or implementation inspiration?

#### OpenAPI Contract Scope

- Should Glaux Server publish one OpenAPI description, multiple modular descriptions, or generated descriptions per deployed capability set?
- How should the OpenAPI description represent:
  - landing page,
  - conformance endpoint,
  - API definition endpoint,
  - collections,
  - resource items,
  - links and navigation,
  - query parameters,
  - content negotiation,
  - schemas,
  - errors,
  - datastreams,
  - observations,
  - control streams,
  - commands,
  - feasibility,
  - status,
  - events?
- How should optional, conditional, experimental, extension, profile-specific, or not-yet-implemented behavior be represented?

#### Schema and Component Strategy

- How should OpenAPI components reference CSAPI schemas, OGC schemas, SensorML schemas, SWE Common schemas, and Glaux-specific reusable components?
- Which schemas should be embedded, referenced, generated, pinned, vendored, or resolved remotely?
- How should schema versioning and compatibility be handled?
- How should schema references support validation, generated clients, offline/DDIL scenarios, and reproducible tests?
- Which schema-validation questions should be handed to `IDR-SRV-023`?

#### Documentation Strategy

- What human-readable documentation should accompany the OpenAPI contract?
- Which documentation belongs in the OpenAPI description, generated documentation, markdown guides, examples, or external developer resources?
- How should documentation explain conformance, profiles, media types, error responses, examples, authentication, tasking flows, and streaming/event behavior?
- How should documentation distinguish stable, experimental, deprecated, and future behavior?
- How should documentation support DGIWG/NATO readers, open-source developers, external CSAPI clients, and conformance testers?

#### Publishing and Discovery Behavior

- How should Glaux Server expose its API description from the API root and service description links?
- Which media types and links should be used for OpenAPI documents and rendered documentation?
- How should API documentation be published for deployed servers, local development, test environments, and static project sites?
- How should generated documentation remain synchronized with the live server?
- Which publishing questions depend on deployment and CI/CD research?

#### Security, Authorization, and Policy Documentation

- How should authentication and authorization schemes be documented?
- How should policy, releasability, cross-boundary access, and command authorization be described without exposing sensitive details?
- How should OpenAPI represent endpoints with different authorization requirements?
- How should security documentation support external clients and tests?
- Which findings should be handed to `IDR-SRV-039`, `IDR-SRV-040`, and `IDR-SRV-055`?

#### Validation, Testing, and Interoperability

- What tests are needed to validate the OpenAPI description against the live implementation?
- What tests are needed to validate examples, schemas, parameters, media types, and error responses?
- What generated-client tests should be considered?
- How should OpenAPI contract drift be detected?
- What compatibility checks are needed across versions?
- Which findings should be handed to `IDR-SRV-050`, `IDR-SRV-051`, `IDR-SRV-053`, and `IDR-SRV-056`?

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

- `IDR-SRV-006` through `IDR-SRV-013` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Controlling OGC and Standards-Package Sources

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC API standards family: https://ogcapi.ogc.org/
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html

### OpenAPI and Documentation Sources

- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- OpenAPI Initiative: https://www.openapis.org/
- Swagger / OpenAPI documentation: https://swagger.io/specification/
- Redocly / ReDoc documentation: https://redocly.com/docs/redoc/
- Scalar API Reference documentation: https://guides.scalar.com/scalar/scalar-api-references
- Stoplight Elements documentation: https://docs.stoplight.io/docs/elements/

### HTTP and Representation Sources

- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 8288 - Web Linking: https://www.rfc-editor.org/rfc/rfc8288
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- IANA Media Types Registry: https://www.iana.org/assignments/media-types/media-types.xhtml
- IANA Link Relation Types: https://www.iana.org/assignments/link-relations/link-relations.xhtml

### Existing Implementation and Interoperability Context

Use these as supporting evidence once the corresponding focused studies are available:

- `IDR-SRV-014A` through `IDR-SRV-014G`
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability findings repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/

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

### Phase 1: Source Collection and Documentation Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and define the extraction framework.

**Tasks:**

1. Gather CSAPI, OGC API - Features, OpenAPI, schema, documentation-rendering, HTTP, media-type, and error-response sources.
2. Gather reports from `IDR-SRV-006` through `IDR-SRV-013`, if available.
3. Identify requirements and conventions that affect API definition exposure, OpenAPI publication, documentation links, schemas, examples, media types, errors, and security schemes.
4. Define the OpenAPI/documentation inventory fields for the report.
5. Define classification values: normative, inherited, recommended, conventional, implementation-specific, future candidate, unsupported/out of scope, unresolved.

**Expected Output:** Source inventory and OpenAPI/documentation extraction framework.

### Phase 2: Official CSAPI OpenAPI Artifact Review (2.5-3.5 hours)

**Objective:** Analyze official CSAPI OpenAPI artifacts and related schemas.

**Tasks:**

1. Review official CSAPI OpenAPI artifacts for structure, endpoints, components, parameters, media types, schemas, security, and examples.
2. Identify how Part 1 and Part 2 behavior is represented.
3. Identify gaps or ambiguities between prose standards and OpenAPI artifacts.
4. Identify reusable patterns for Glaux Server.
5. Identify where implementation-specific generation or composition may be needed.

**Expected Output:** Official CSAPI OpenAPI artifact analysis.

### Phase 3: Glaux Server OpenAPI Contract Scope Analysis (2-3 hours)

**Objective:** Define the intended scope and structure of the Glaux Server OpenAPI contract.

**Tasks:**

1. Map prior behavior findings to OpenAPI paths, operations, parameters, request bodies, responses, headers, links, tags, components, examples, and security schemes.
2. Identify whether the Glaux Server description should be single-file, modular, generated, static, or hybrid.
3. Identify how optional, conditional, experimental, deprecated, or profile-specific behavior should be represented.
4. Identify schema-reference strategy options.
5. Identify versioning and compatibility implications.

**Expected Output:** Glaux Server OpenAPI contract scope and structure analysis.

### Phase 4: Human-Readable Documentation and Examples Analysis (2-2.5 hours)

**Objective:** Define the relationship between the machine-readable contract and human-facing documentation.

**Tasks:**

1. Identify documentation audiences and use cases.
2. Identify documentation that belongs in the OpenAPI description versus external markdown/guides/examples.
3. Identify example types needed for discovery, resource traversal, observations, commands, feasibility, events, error handling, and authentication.
4. Identify rendering options such as ReDoc, Swagger UI, Scalar, static site generation, or project documentation pages.
5. Identify risks from stale examples or mismatched documentation.

**Expected Output:** Human-readable documentation and examples strategy notes.

### Phase 5: Publishing, Validation, Security, and Test Implication Analysis (2-2.5 hours)

**Objective:** Prepare OpenAPI and documentation findings for downstream implementation and verification.

**Tasks:**

1. Identify how the OpenAPI document should be exposed from the live server and static project materials.
2. Identify validation checks for OpenAPI syntax, schema references, examples, media types, error schemas, and security schemes.
3. Identify contract-drift checks between implementation and OpenAPI.
4. Identify generated-client and external-client test implications.
5. Identify security and policy documentation constraints.
6. Map handoffs to validation, conformance, security, testing, and interoperability topics.

**Expected Output:** OpenAPI publishing, validation, and test-implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert research output into a decision-usable OpenAPI and documentation baseline.

**Tasks:**

1. Consolidate artifact review, contract scope, documentation strategy, publishing strategy, validation implications, and test implications.
2. Resolve conflicts and ambiguities where possible.
3. Produce findings grouped by machine-readable contract, human documentation, schemas/examples, publishing, and testing.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Official CSAPI OpenAPI artifacts are reviewed with source anchors.
- [ ] OpenAPI requirements and conventions are identified.
- [ ] The relationship between OpenAPI contract, live server behavior, schemas, examples, and human-readable documentation is documented.
- [ ] Schema reference and component reuse implications are identified.
- [ ] Representation, error, query, security, conformance, and dynamic-data behavior are mapped to OpenAPI documentation needs.
- [ ] Publishing, versioning, deprecation, and documentation synchronization implications are documented.
- [ ] Validation, contract drift, generated-client, conformance, and interoperability test implications are identified.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** OpenAPI Description and API Documentation Strategy Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014-openapi-description-and-api-documentation-strategy-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Official CSAPI OpenAPI artifact analysis
5. Glaux Server OpenAPI contract scope and structure findings
6. Schema and component strategy findings
7. Human-readable documentation and examples strategy findings
8. Publishing and discovery findings
9. Security and policy documentation findings
10. Validation, contract-drift, and generated-client implications
11. Interoperability and existing-implementation implications
12. Test-strategy implications
13. Downstream topic handoff matrix
14. Recommendations
15. Risks, constraints, and open questions
16. Validation against this plan's success criteria
17. References

The OpenAPI/documentation matrix should include, at minimum:

- Documentation or contract area
- Related server behavior
- Source standard / source anchor
- OpenAPI representation requirement or convention
- Related schema/component/artifact
- Human documentation need
- Generated-client implication
- Validation implication
- Test implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-006` through `IDR-SRV-013` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, OpenAPI, schema, HTTP, and documentation-rendering sources must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-014A: OSH CSAPI Server Implementation Study`
- `IDR-SRV-014B: Connected Systems Go CSAPI Server Implementation Study`
- `IDR-SRV-014C: pygeoapi CSAPI Server Implementation Study`
- `IDR-SRV-014D: SECD CSAPI Server Implementation Study`
- `IDR-SRV-014E: OS4CSAPI Client Smoke Test Findings Study`
- `IDR-SRV-014F: SECD Interoperability Findings Study`
- `IDR-SRV-014G: OS4CSAPI Discussions Lessons-Learned Study`
- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
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

- This topic should define the OpenAPI/documentation strategy, not implement the documentation pipeline.
- The OpenAPI description must not overstate capabilities not actually implemented or evidenced.
- Existing implementation studies may later refine practical expectations for OpenAPI completeness and client compatibility.
- Open question: Should Glaux Server generate OpenAPI from code, generate code from OpenAPI, or maintain a hybrid contract-first approach?
- Open question: Which official CSAPI OpenAPI artifacts are directly reusable versus better treated as reference material?
- Open question: How should Glaux Server support offline or DDIL use of documentation and schemas?
- Open question: Which documentation renderer best supports the intended Glaux developer and standards audience?
- Risk: Stale OpenAPI descriptions could break clients and conformance testing.
- Risk: Under-documented errors, schemas, media types, or security behavior could undermine interoperability.
- Risk: Over-documenting future behavior as implemented behavior could create false conformance expectations.

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
- OGC API standards family: https://ogcapi.ogc.org/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- OpenAPI Initiative: https://www.openapis.org/
- Swagger / OpenAPI documentation: https://swagger.io/specification/
- ReDoc documentation: https://redocly.com/docs/redoc/
- Scalar API Reference documentation: https://guides.scalar.com/scalar/scalar-api-references
- Stoplight Elements documentation: https://docs.stoplight.io/docs/elements/
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 8288 - Web Linking: https://www.rfc-editor.org/rfc/rfc8288
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- IANA Media Types Registry: https://www.iana.org/assignments/media-types/media-types.xhtml
- IANA Link Relation Types: https://www.iana.org/assignments/link-relations/link-relations.xhtml
- OS4CSAPI testing research-plan corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans

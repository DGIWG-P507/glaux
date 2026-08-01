# Section 012: Content Negotiation, Media Types, and Encoding Selection - Research Plan

**Status:** Planned  
**Last Updated:** June 8, 2026  
**Estimated Research Time:** 10-14 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-012-content-negotiation-media-types-and-encoding-selection-report.md`

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

This topic research must define the Glaux Server planning baseline for **content negotiation, media types, and encoding selection**.

The research must answer:

- What content negotiation behavior must Glaux Server support for OGC API - Connected Systems Part 1 and Part 2?
- Which media types, encodings, profiles, schemas, and representations are required, recommended, optional, implementation-specific, or future-facing?
- How should Glaux Server select representations for landing pages, conformance declarations, API definitions, collections, feature resources, observations, datastreams, control streams, commands, status resources, system events, SensorML descriptions, and SWE Common structures?
- How should Glaux Server handle JSON, GeoJSON, HTML, OpenAPI, SensorML, SWE Common, schema documents, streaming/event encodings, and future binary or pub/sub encodings?
- What deterministic behavior is needed for `Accept`, `Content-Type`, alternate links, default representations, unsupported media types, unsupported profiles, and validation?

The output must be a representation and negotiation baseline with source anchors, server obligations, implementation implications, downstream handoffs, and test implications.

### Why This Topic Order

This topic follows `IDR-SRV-011: Query, Filtering, Sorting, Pagination, and Selection Semantics`. Earlier topics define entry points, navigation, compatibility, and query behavior. This topic defines how the server chooses and communicates representations of those resources.

This topic must precede error handling, OpenAPI documentation, schema validation, SensorML/SWE representation strategy, dynamic-data semantics, streaming strategy, and interoperability testing because representation selection affects all externally visible API behavior.

### Critical Constraint(s)

- Treat official OGC API - Connected Systems Part 1, Part 2, OGC API - Features, SensorML, SWE Common, schemas, OpenAPI artifacts, HTTP semantics, and registered media-type conventions as controlling where they apply.
- Use `IDR-SRV-006` through `IDR-SRV-011` outputs as input when available.
- Do not finalize SensorML or SWE Common modeling here; hand detailed representation design to `IDR-SRV-021` and `IDR-SRV-022`.
- Do not finalize schema validation here; hand validation details to `IDR-SRV-023`.
- Do not finalize streaming architecture here; hand protocol and event-publication questions to `IDR-SRV-035`.
- Do not invent non-standard media types or format negotiation conventions unless the report clearly identifies them as implementation-specific or future candidates.
- Clearly distinguish normative media types, inherited OGC API behavior, CSAPI-specific representation behavior, SensorML/SWE dependencies, optional/future encodings, implementation-specific defaults, and unsupported/out-of-scope encodings.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What content negotiation behavior is required or expected for Glaux Server?
2. Which media types and encodings must Glaux Server support for CSAPI Part 1, CSAPI Part 2, SensorML, and SWE Common alignment?
3. How should Glaux Server choose default representations and expose alternate representations?
4. How should unsupported, ambiguous, incompatible, or invalid representation requests be handled?
5. What downstream documentation, validation, interoperability, and test implications follow from the representation and negotiation baseline?

### Detailed Questions

#### Standards and Representation Sources

- What representation, encoding, and media-type requirements are defined in CSAPI Part 1 and Part 2?
- What content negotiation behavior is inherited from OGC API - Features or common OGC API patterns?
- What representation behavior is expressed in OpenAPI artifacts and schema files?
- What SensorML and SWE Common representations are referenced or required by the standards package?
- What media types are registered, conventional, or explicitly referenced for JSON, GeoJSON, HTML, OpenAPI, SensorML, SWE Common, and related resources?

#### Resource-Specific Representation Behavior

- What representations apply to the landing page, API definition, conformance declaration, collections, collection items, systems, procedures, deployments, sampling features, properties, datastreams, observations, status, system events, control streams, commands, command status, feasibility, SensorML descriptions, and SWE Common components?
- Which resources require multiple representations?
- Which resources should have a single authoritative representation?
- Which representations must include links to schemas, profiles, alternate formats, or documentation?

#### Negotiation Mechanisms

- How should Glaux Server process `Accept` headers, including wildcards and quality values?
- How should Glaux Server process `Content-Type` headers for request bodies?
- Should Glaux Server support format query parameters, extension suffixes, or other alternate negotiation patterns?
- What default representation should be used when no explicit preference is provided?
- How should alternate representations be linked?
- What caching and `Vary` header implications should be researched?

#### Request Body Encoding Selection

- What request-body encodings apply to create, update, command, feasibility, ingestion, or server-side contract operations?
- What request-body validation depends on media type or encoding?
- How should unsupported request encodings be rejected?
- How should command/tasking payload encoding interact with SWE Common parameter definitions?

#### Error, Validation, Documentation, and Testing

- What HTTP status codes and error responses apply to unsupported `Accept`, unsupported `Content-Type`, invalid payload encoding, invalid schema, unsupported profile, unavailable representation, or inconsistent schema links?
- What OpenAPI documentation is needed for request and response media types?
- What schema validation is representation-specific?
- What golden files or fixtures are needed for each representation?
- What tests are needed for negotiation, alternate links, default representations, request body encodings, unsupported media types, and profile handling?

#### Interoperability and Existing Implementation Lessons

- How do existing CSAPI implementations support content negotiation and media types?
- What smoke-test or interoperability findings identify representation or encoding compatibility issues?
- Which representation behavior is needed by CSAPI Explorer and external clients?
- Which findings should be handed to `IDR-SRV-014A` through `IDR-SRV-014G` and `IDR-SRV-056`?

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

- `IDR-SRV-006` through `IDR-SRV-011` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Controlling OGC and Standards-Package Sources

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1 standard: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC API standards family: https://ogcapi.ogc.org/
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html

### HTTP, Media-Type, and Representation Sources

- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 - HTTP Caching: https://www.rfc-editor.org/rfc/rfc9111
- RFC 6838 - Media Type Specifications and Registration Procedures: https://www.rfc-editor.org/rfc/rfc6838
- RFC 6906 - The `profile` Link Relation Type: https://www.rfc-editor.org/rfc/rfc6906
- RFC 8288 - Web Linking: https://www.rfc-editor.org/rfc/rfc8288
- IANA Media Types Registry: https://www.iana.org/assignments/media-types/media-types.xhtml
- IANA Link Relation Types: https://www.iana.org/assignments/link-relations/link-relations.xhtml
- GeoJSON RFC 7946: https://www.rfc-editor.org/rfc/rfc7946
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html

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

### Phase 1: Source Collection and Representation Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and define the extraction framework.

**Tasks:**

1. Gather CSAPI Part 1, CSAPI Part 2, OGC API - Features, SensorML, SWE Common, schema, OpenAPI, HTTP, and media-type registry sources.
2. Gather reports from `IDR-SRV-006` through `IDR-SRV-011`, if available.
3. Identify requirements and conformance classes that mention representations, encodings, media types, content negotiation, alternate links, profiles, schemas, or request/response bodies.
4. Define representation and media-type inventory fields.
5. Define classification values: normative, inherited, recommended, optional, conditional, implementation-specific, future candidate, unsupported/out of scope, unresolved.

**Expected Output:** Source inventory and representation/negotiation extraction framework.

### Phase 2: Standards-Based Media Type and Encoding Extraction (2.5-3.5 hours)

**Objective:** Extract required and expected media types, encodings, and representation behavior.

**Tasks:**

1. Review CSAPI Part 1 for response representations, media types, schemas, and negotiation behavior.
2. Review CSAPI Part 2 for dynamic-data, observation, command, event, and streaming-adjacent representation behavior.
3. Review OGC API - Features inherited behavior for JSON, HTML, GeoJSON, alternate representations, and API documentation.
4. Review SensorML and SWE Common for relevant encodings and media-type implications.
5. Review OpenAPI artifacts and schemas for declared request/response content types.
6. Capture source anchors, resource applicability, request/response direction, schema association, and conformance implications.

**Expected Output:** Media type, encoding, and representation inventory with source anchors.

### Phase 3: Resource-Family Representation Mapping (2-3 hours)

**Objective:** Map representations and negotiation behavior to Glaux Server resource families.

**Tasks:**

1. Group representation behavior by resource family:
   - landing page, API definition, conformance declaration,
   - collections and feature resources,
   - systems, procedures, deployments, sampling features, and properties,
   - datastreams and observations,
   - status and system events,
   - control streams, commands, command status, and feasibility,
   - SensorML descriptions,
   - SWE Common data components and command parameters.
2. Identify default, required, alternate, and unsupported representations for each family.
3. Identify representation links, profile links, schema links, and documentation links.
4. Identify where request bodies require specific encodings.
5. Identify unresolved representation questions.

**Expected Output:** Resource-family representation and encoding matrix.

### Phase 4: Negotiation Mechanism and Default Behavior Analysis (2-2.5 hours)

**Objective:** Define deterministic server behavior for representation selection.

**Tasks:**

1. Analyze `Accept` header handling, including specificity, wildcard media ranges, quality values, and unsupported preferences.
2. Analyze `Content-Type` handling for request bodies.
3. Identify whether format query parameters, extension suffixes, or other mechanisms are required, conventional, optional, or undesirable.
4. Define default representation behavior for common resources.
5. Identify alternate representation link behavior and profile/schema link behavior.
6. Identify caching and `Vary` header implications where relevant.

**Expected Output:** Negotiation and default behavior analysis.

### Phase 5: Error, Validation, Documentation, and Test Implication Analysis (2-2.5 hours)

**Objective:** Prepare findings for downstream implementation and verification topics.

**Tasks:**

1. Identify error cases for unsupported `Accept`, unsupported `Content-Type`, invalid encoding, invalid schema, unsupported profile, unavailable representation, and inconsistent schema/media-type declarations.
2. Identify handoffs to `IDR-SRV-013` for deterministic error behavior.
3. Identify handoffs to `IDR-SRV-014` for OpenAPI and documentation.
4. Identify handoffs to `IDR-SRV-021`, `IDR-SRV-022`, and `IDR-SRV-023` for SensorML, SWE Common, and validation.
5. Identify positive, negative, conformance, golden-file, fixture, and interoperability tests.
6. Identify external-client compatibility questions.

**Expected Output:** Representation validation, documentation, and test-implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert research output into a decision-usable representation and negotiation baseline.

**Tasks:**

1. Consolidate media-type inventory, resource-family mappings, negotiation behavior findings, and implementation/test implications.
2. Resolve conflicts and ambiguities where possible.
3. Produce findings grouped by representation family, negotiation mechanism, and resource family.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Content negotiation, media-type, and encoding requirements are identified with source anchors.
- [ ] CSAPI-specific behavior and inherited OGC API behavior are distinguished.
- [ ] SensorML and SWE Common representation dependencies are identified.
- [ ] Resource-family representation behavior is mapped.
- [ ] Required, recommended, optional, implementation-specific, future-candidate, and unsupported representations are classified.
- [ ] `Accept` and `Content-Type` handling implications are documented.
- [ ] Default representation and alternate-link behavior are documented.
- [ ] Error, validation, OpenAPI, fixture, golden-file, conformance, and interoperability handoffs are documented.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Content Negotiation, Media Types, and Encoding Selection Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-012-content-negotiation-media-types-and-encoding-selection-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Content negotiation and media-type extraction methodology
5. Standards-based media type and encoding inventory
6. Resource-family representation matrix
7. Negotiation mechanism and default behavior findings
8. Request body encoding findings
9. SensorML and SWE Common representation implications
10. Error/failure and validation implications
11. OpenAPI and documentation implications
12. Interoperability and existing-implementation implications
13. Test-strategy implications
14. Downstream topic handoff matrix
15. Recommendations
16. Risks, constraints, and open questions
17. Validation against this plan's success criteria
18. References

The representation matrix should include, at minimum:

- Resource family
- Representation or encoding
- Media type
- Request/response applicability
- Source standard / source anchor
- Normative / inherited / recommended / optional / implementation-specific / unresolved classification
- Default or alternate status
- Related schema/OpenAPI artifact
- Error condition(s)
- Validation implication
- Test implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-006` through `IDR-SRV-011` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, OpenAPI, schema, HTTP, and media-type registry sources must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-013: Error Model, HTTP Status Codes, and Failure Semantics`
- `IDR-SRV-014: OpenAPI Description and API Documentation Strategy`
- `IDR-SRV-021: SensorML Representation Strategy`
- `IDR-SRV-022: SWE Common Data Component Strategy`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-024: Units, Observed Properties, and Semantic Binding Strategy`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
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

- This topic should define representation and negotiation semantics, not implement serializers, parsers, schema validators, or streaming protocols.
- Some future encodings may be relevant to CSAPI roadmap or AEP evolution but should be classified as future-facing unless currently required.
- Existing implementation studies may later refine practical expectations for media type support and client compatibility.
- Open question: Which SensorML and SWE Common encodings should Glaux Server support as part of the standards-aligned baseline?
- Open question: Should Glaux Server support HTML representations for all user-facing resources or only OGC API entry points and documentation resources?
- Open question: Should Glaux Server support format query parameters, or should it rely on HTTP content negotiation and links?
- Open question: What representation behavior is necessary for CSAPI Explorer and other external clients?
- Risk: Inconsistent media-type behavior could break generated clients and conformance testing.
- Risk: Unsupported or poorly documented encodings could make standards-conformant data difficult to consume.
- Risk: Treating future encodings as current obligations could overcomplicate the server.
- Risk: Treating content negotiation as cosmetic could create hidden interoperability failures.

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
- OGC API standards family: https://ogcapi.ogc.org/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 - HTTP Caching: https://www.rfc-editor.org/rfc/rfc9111
- RFC 6838 - Media Type Specifications and Registration Procedures: https://www.rfc-editor.org/rfc/rfc6838
- RFC 6906 - The `profile` Link Relation Type: https://www.rfc-editor.org/rfc/rfc6906
- RFC 8288 - Web Linking: https://www.rfc-editor.org/rfc/rfc8288
- IANA Media Types Registry: https://www.iana.org/assignments/media-types/media-types.xhtml
- IANA Link Relation Types: https://www.iana.org/assignments/link-relations/link-relations.xhtml
- GeoJSON RFC 7946: https://www.rfc-editor.org/rfc/rfc7946
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- OS4CSAPI testing research-plan corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans

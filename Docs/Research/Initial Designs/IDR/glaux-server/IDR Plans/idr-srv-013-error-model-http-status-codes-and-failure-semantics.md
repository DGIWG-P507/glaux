# Section 013: Error Model, HTTP Status Codes, and Failure Semantics - Research Plan

**Status:** In Progress<br>
**Last Updated:** August 2, 2026<br>
**Estimated Research Time:** 10-14 hours  
**Actual Research Time:** Approximately 6 hours of AI-assisted elapsed execution time on August 2, 2026<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-013-error-model-http-status-codes-and-failure-semantics-report.md`

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

This topic research must define the Glaux Server planning baseline for **error model, HTTP status codes, and failure semantics**.

The research must answer:

- What deterministic error taxonomy should Glaux Server use across OGC API - Connected Systems Part 1 and Part 2 behavior?
- Which HTTP status codes and response bodies are required, recommended, inherited, conventional, or implementation-specific?
- How should Glaux Server handle errors for landing pages, API definitions, conformance declarations, collections, resources, links, navigation, queries, filters, sorting, pagination, selection, content negotiation, request encodings, schema validation, observations, datastreams, status, events, control streams, commands, feasibility, authentication, authorization, policy constraints, DDIL conditions, and upstream dependency failures?
- How should errors remain standards-aligned, machine-readable, testable, secure, and useful to clients?
- What downstream implementation, OpenAPI documentation, validation, security, conformance, and test-strategy implications follow from the error model?

The output must be an error and failure behavior baseline with source anchors, status-code guidance, response-body guidance, downstream handoffs, and test implications.

### Why This Topic Order

This topic follows:

- `IDR-SRV-009: Landing Page, API Definition, and Conformance Declaration Behavior`
- `IDR-SRV-010: Collections, Resources, Links, and Navigation Behavior`
- `IDR-SRV-010A: API Versioning, Backward Compatibility, and Deprecation Strategy`
- `IDR-SRV-011: Query, Filtering, Sorting, Pagination, and Selection Semantics`
- `IDR-SRV-012: Content Negotiation, Media Types, and Encoding Selection`

Those topics define visible API surfaces and normal-path behavior. This topic defines the corresponding failure-path behavior so downstream implementation and testing can be deterministic and standards-aligned.

This topic must precede OpenAPI documentation strategy, schema validation strategy, command lifecycle strategy, security threat modeling, conformance harness strategy, and requirement-to-test traceability because failure semantics must be documented, implemented, validated, and tested consistently.

### Critical Constraint(s)

- Treat official OGC API - Connected Systems, OGC API - Features, OGC API common behavior, HTTP semantics, RFC 7807 / RFC 9457 problem details guidance, OpenAPI, SensorML, SWE Common, and schema artifacts as controlling where they apply.
- Use `IDR-SRV-006` through `IDR-SRV-012` outputs as input when available.
- Do not design the implementation exception hierarchy here; define the standards-aligned error behavior baseline and hand implementation details to later engineering work.
- Do not design the complete security model here; hand authorization, releasability, and command-control security implications to Category G and test topics.
- Do not expose sensitive internal details in proposed error behavior.
- Clearly distinguish:
  - normative HTTP/OGC/CSAPI error behavior,
  - recommended/conventional error behavior,
  - implementation-specific error choices,
  - security-sensitive error handling,
  - validation errors,
  - operational/dependency failures,
  - DDIL/degraded-state behavior,
  - command/tasking lifecycle failures.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What error and failure semantics are required or expected for Glaux Server?
2. Which HTTP status codes should be used for common CSAPI server error conditions?
3. What machine-readable error response structure should Glaux Server use?
4. How should errors differ across validation, query, content negotiation, resource lookup, authorization, conflict, command/tasking, streaming, dependency, and DDIL-related failures?
5. What downstream documentation, OpenAPI, validation, conformance, security, and test implications follow from the error baseline?

### Detailed Questions

#### Standards and Error Sources

- What error behavior is defined in CSAPI Part 1?
- What error behavior is defined in CSAPI Part 2?
- What error behavior is inherited from OGC API - Features or common OGC API conventions?
- What HTTP status-code guidance applies from HTTP semantics?
- What problem-details or machine-readable error standards are relevant?
- What error behavior is reflected in CSAPI OpenAPI artifacts or schemas?

#### General HTTP and API Errors

- How should Glaux Server handle 400, 401, 403, 404, 405, 406, 409, 410, 412, 415, 422, 429, 500, 501, 503, and other relevant status codes?
- Which status codes should be used for unsupported endpoints, unsupported methods, invalid query parameters, unsupported filters, unsupported media types, unacceptable representations, invalid payloads, schema violations, conflicts, stale state, rate limiting, and dependency outages?
- Which status codes are inappropriate or should be avoided?
- What headers, links, or metadata should accompany specific error responses?

#### Resource and Navigation Failures

- How should errors behave for missing resources, deleted resources, moved resources, stale links, broken relationships, inaccessible linked resources, invalid identifiers, and unsupported collections?
- How should Glaux Server distinguish nonexistent resources from unauthorized resources without leaking sensitive information?
- What behavior is needed for partial failures when related resources are unavailable?
- What behavior should apply to stale, cached, or last-known resource references?

#### Query, Filtering, Sorting, Pagination, and Selection Failures

- How should Glaux Server handle invalid query parameters, invalid field names, invalid sort expressions, invalid temporal ranges, invalid bbox values, unsupported filter combinations, excessive limits, invalid pagination tokens, and selection/projection errors?
- Which failures should be 400 versus 422 versus 501 versus another status?
- What machine-readable details should be returned so clients can correct requests?
- Which errors must avoid leaking restricted resource existence or counts?

#### Content Negotiation and Encoding Failures

- How should Glaux Server handle unsupported `Accept`, unsupported `Content-Type`, unsupported profiles, invalid encodings, invalid JSON/GeoJSON/SensorML/SWE payloads, and unavailable representations?
- How should alternate representations be suggested in error responses, if at all?
- How should content-negotiation failures be documented and tested?

#### Validation and Schema Failures

- What error behavior should apply to schema validation failures?
- How should response-body validation errors differ from request-body validation errors?
- How much detail should be included without leaking internals?
- How should validation errors identify failing fields, constraints, schemas, or profiles?
- Which validation questions should be handed to `IDR-SRV-023`?

#### Dynamic Data, Tasking, Command, and Feasibility Failures

- What errors apply to observations, datastreams, event histories, and status queries?
- What errors apply to command creation, command acceptance, command rejection, command cancellation, invalid command parameters, infeasible commands, unavailable control streams, duplicate commands, unsafe commands, stale commands, or asynchronous command status retrieval?
- Which failures should be represented as HTTP errors, and which should be represented as domain state within command/status resources?
- How should feasibility failures differ from command execution failures?
- Which questions should be handed to `IDR-SRV-036`, `IDR-SRV-037`, and `IDR-SRV-038`?

#### Security, Authorization, Policy, and Releasability Failures

- How should authentication failures differ from authorization failures?
- How should Glaux Server represent policy, releasability, classification, cross-boundary, and need-to-know failures?
- How should errors avoid confirming the existence of restricted resources?
- How should command/control authorization failures be handled?
- Which findings should be handed to `IDR-SRV-039`, `IDR-SRV-040`, and `IDR-SRV-055`?

#### Dependency, Degraded, and DDIL-Informed Failures

- What errors apply when upstream publishers, external systems, sensors, databases, brokers, or storage components are degraded or unavailable?
- How should Glaux Server distinguish server error, dependency error, degraded service, stale data, last-known state, and intentionally unavailable resources?
- What response semantics are needed for DDIL-informed behavior?
- Which failures should be represented through HTTP status, status resources, system events, warnings, metadata, or retry guidance?
- Which findings should be handed to `IDR-SRV-020`, `IDR-SRV-035`, `IDR-SRV-041`, `IDR-SRV-042`, `IDR-SRV-043`, and `IDR-SRV-046`?

#### OpenAPI, Documentation, and Testing

- How should error responses be represented in OpenAPI?
- What reusable error schemas should be documented?
- What negative tests are needed for each error family?
- What conformance tests, security tests, command-control tests, and interoperability tests are implied?
- How should golden files or fixtures be used for error responses?
- Which findings should be handed to `IDR-SRV-014`, `IDR-SRV-050`, `IDR-SRV-051`, `IDR-SRV-053`, `IDR-SRV-055`, and `IDR-SRV-056`?

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

- `IDR-SRV-006` through `IDR-SRV-012` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Controlling OGC and Standards-Package Sources

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC API standards family: https://ogcapi.ogc.org/
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html

### HTTP, Error, and API Sources

- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 - HTTP Caching: https://www.rfc-editor.org/rfc/rfc9111
- RFC 7807 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc7807
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- IANA HTTP Status Code Registry: https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
- IANA HTTP Field Name Registry: https://www.iana.org/assignments/http-fields/http-fields.xhtml

### Security and Authorization Context Sources

Use these as supporting sources for security-sensitive error semantics:

- RFC 9110 authentication and authorization semantics: https://www.rfc-editor.org/rfc/rfc9110
- OAuth 2.0 Bearer Token Usage, where relevant: https://www.rfc-editor.org/rfc/rfc6750
- OWASP API Security Top 10: https://owasp.org/API-Security/

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

### Phase 1: Source Collection and Error-Taxonomy Setup (1.5-2 hours)

**Objective:** Establish the evidence base and define the extraction framework.

**Tasks:**

1. Gather CSAPI Part 1, CSAPI Part 2, OGC API - Features, OpenAPI, schema, HTTP, problem-details, and security-related sources.
2. Gather reports from `IDR-SRV-006` through `IDR-SRV-012`, if available.
3. Identify requirements and conformance classes that mention errors, status codes, validation, unsupported requests, unavailable resources, authorization, or failure behavior.
4. Define the error taxonomy fields for the report.
5. Define classification values: normative, inherited, recommended, conventional, implementation-specific, security-sensitive, unresolved.

**Expected Output:** Source inventory and error-taxonomy framework.

### Phase 2: Standards-Based Error Behavior Extraction (2.5-3.5 hours)

**Objective:** Extract required and expected error behavior from standards and artifacts.

**Tasks:**

1. Review CSAPI Part 1 for error and failure behavior.
2. Review CSAPI Part 2 for dynamic-data, command, feasibility, status, and event error behavior.
3. Review OGC API - Features inherited error behavior.
4. Review OpenAPI artifacts and schemas for declared error responses.
5. Review HTTP status-code and problem-details sources for applicable semantics.
6. Capture source anchors, status codes, response-body expectations, headers, and conformance implications.

**Expected Output:** Standards-based error behavior inventory.

### Phase 3: Error Family Mapping (2.5-3 hours)

**Objective:** Map errors to Glaux Server behavior areas.

**Tasks:**

1. Group error behavior by family:
   - resource lookup and navigation,
   - query/filter/sort/pagination/selection,
   - content negotiation and request encoding,
   - validation and schemas,
   - dynamic data and observations,
   - tasking, commands, and feasibility,
   - status and events,
   - authentication, authorization, policy, and releasability,
   - dependency, degraded, stale, and DDIL-informed states,
   - rate limiting and resource exhaustion,
   - server/internal failures.
2. Identify expected HTTP status codes and response-body patterns for each family.
3. Identify where failures should be domain state rather than HTTP errors.
4. Identify unresolved error-mapping questions.

**Expected Output:** Error family mapping matrix.

### Phase 4: Machine-Readable Error Body and Documentation Analysis (1.5-2 hours)

**Objective:** Define machine-readable error response expectations and documentation implications.

**Tasks:**

1. Evaluate problem-details suitability for Glaux Server.
2. Identify reusable error fields needed for trace IDs, titles, details, source parameters, schema paths, validation issues, retry hints, replacement links, and documentation links.
3. Identify sensitive fields that should not be exposed.
4. Identify OpenAPI documentation requirements for error responses.
5. Identify golden-file and fixture implications.

**Expected Output:** Error response body and documentation guidance.

### Phase 5: Security, DDIL, Interoperability, and Test Implication Analysis (2-2.5 hours)

**Objective:** Prepare error findings for downstream security, degraded-operation, interoperability, and test topics.

**Tasks:**

1. Identify security-sensitive error behavior and information leakage risks.
2. Identify DDIL/degraded-state error and status/resource interaction questions.
3. Identify external-client and interoperability expectations.
4. Identify positive, negative, boundary, security, command-control, conformance, and golden-file tests.
5. Map handoffs to downstream documentation, validation, security, DDIL, command-control, conformance, and test topics.

**Expected Output:** Error implementation and test-implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert research output into a decision-usable error and failure behavior baseline.

**Tasks:**

1. Consolidate error behavior inventory, status-code mapping, response-body guidance, and test implications.
2. Resolve conflicts and ambiguities where possible.
3. Produce findings grouped by error family and resource/API behavior area.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Error and failure requirements are identified with source anchors.
- [ ] CSAPI-specific behavior and inherited OGC API / HTTP behavior are distinguished.
- [ ] Error families are mapped to relevant HTTP status codes.
- [ ] Machine-readable error response guidance is documented.
- [ ] Validation, content-negotiation, query, resource, command/tasking, authorization, dependency, and DDIL-related failure semantics are assessed.
- [ ] Security-sensitive error behavior and information leakage risks are identified.
- [ ] OpenAPI, documentation, validation, conformance, and testing handoffs are documented.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Error Model, HTTP Status Codes, and Failure Semantics Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-013-error-model-http-status-codes-and-failure-semantics-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Error behavior extraction methodology
5. Standards-based error behavior inventory
6. HTTP status-code mapping matrix
7. Error family behavior analysis
8. Machine-readable error response guidance
9. Security-sensitive error behavior findings
10. DDIL/degraded-state and dependency-failure findings
11. OpenAPI and documentation implications
12. Interoperability and existing-implementation implications
13. Test-strategy implications
14. Downstream topic handoff matrix
15. Recommendations
16. Risks, constraints, and open questions
17. Validation against this plan's success criteria
18. References

The error matrix should include, at minimum:

- Error family
- Example condition
- HTTP status code
- Source standard / source anchor
- Normative / inherited / recommended / implementation-specific classification
- Response body expectation
- Header/link expectation, if applicable
- Security sensitivity
- Retry or recovery guidance
- OpenAPI implication
- Test implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-006` through `IDR-SRV-012` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, OpenAPI, schema, HTTP, problem-details, and security-related sources must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-014: OpenAPI Description and API Documentation Strategy`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-037: Feasibility and Asynchronous Tasking Strategy`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints`
- `IDR-SRV-042: DDIL-Informed Server Semantics`
- `IDR-SRV-043: Server Synchronization and Conflict Handling Boundary`
- `IDR-SRV-046: Reference Deployment Strategy`
- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
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

**Actual Research Time:** Approximately 6 hours of AI-assisted elapsed execution time on August 2, 2026<br>
**Research Execution Completed:** August 2, 2026<br>
**Completion Date:** Pending plan-owner acceptance

---

## 10. Notes and Open Questions

- This topic should define error semantics, not implement the exception hierarchy.
- Some domain failures, especially command lifecycle and feasibility failures, may be better represented as resource state rather than HTTP errors.
- Existing implementation studies may later refine practical expectations for error response shape and client compatibility.
- Open question: Should Glaux Server adopt RFC 9457 problem details as the default machine-readable error representation?
- Open question: How much validation detail can be safely returned without exposing internals or policy-sensitive information?
- Open question: How should Glaux Server distinguish stale data, degraded service, dependency outage, and unavailable resources in a DDIL-informed way?
- Risk: Inconsistent error behavior could break generated clients, conformance tests, and operational troubleshooting.
- Risk: Overly detailed errors could leak sensitive information.
- Risk: Under-detailed errors could make client correction and interoperability debugging difficult.
- Risk: Treating command/tasking domain failures as simple HTTP errors could weaken auditability and lifecycle semantics.

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
- RFC 7807 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc7807
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- IANA HTTP Status Code Registry: https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
- IANA HTTP Field Name Registry: https://www.iana.org/assignments/http-fields/http-fields.xhtml
- OAuth 2.0 Bearer Token Usage: https://www.rfc-editor.org/rfc/rfc6750
- OWASP API Security Top 10: https://owasp.org/API-Security/
- OS4CSAPI testing research-plan corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans

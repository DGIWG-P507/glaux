# Section 023: Schema and Encoding Validation Strategy - Research Plan

**Status:** Planned  
**Last Updated:** June 10, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-023-schema-and-encoding-validation-strategy-report.md`

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

This topic research must define the Glaux Server planning baseline for **schema and encoding validation strategy** across CSAPI Part 1, CSAPI Part 2, OGC API - Features, SensorML, SWE Common, OpenAPI descriptions, JSON/GeoJSON representations, media types, content negotiation, request validation, response validation, ingestion validation, command validation, and conformance testing.

The research must answer:

- What schemas, OpenAPI artifacts, media types, encodings, profiles, and validation rules must Glaux Server use for API requests, API responses, imported metadata, dynamic data, observations, status updates, system events, control streams, commands, and feasibility resources?
- How should Glaux Server distinguish structural validation, semantic validation, profile validation, operational validation, security/policy validation, and interoperability validation?
- Which validation responsibilities are normative standards requirements, AEP-4789 profile expectations, implementation-quality requirements, or conformance-test requirements?
- How should Glaux Server validate CSAPI JSON, GeoJSON, SensorML, SWE Common, OpenAPI-driven payloads, schema references, links, content negotiation, media types, encodings, nil values, units, observed properties, controlled properties, and command parameters?
- How should validation be applied at registration, update, import, ingestion, normalization, command submission, feasibility evaluation, response generation, conformance testing, CI, fixtures, and golden-file generation?
- How should Glaux Server handle invalid, partially valid, profile-divergent, stale-schema, ambiguous, unsupported, or policy-restricted payloads?
- What downstream implications follow for unit/semantic binding, persistence, ingestion, command lifecycle, security, fixtures, conformance harnesses, requirement-to-test traceability, and interoperability testing?

The output must be a schema and encoding validation strategy baseline with source anchors, validation-scope definitions, resource-family mappings, validation stages, error-handling implications, tooling implications, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows:

- `IDR-SRV-021: SensorML Representation Strategy`
- `IDR-SRV-022: SWE Common Data Component Strategy`

SensorML and SWE Common define key representation and data-component structures. This topic determines how those structures, along with CSAPI, OGC API - Features, OpenAPI, JSON/GeoJSON, content negotiation, media types, and encodings, should be validated by Glaux Server. It must precede unit/observed-property/semantic binding strategy, metadata/document storage, ingestion normalization, datastream/observation semantics, command lifecycle validation, conformance harness strategy, fixture planning, and interoperability testing.

### Critical Constraint(s)

- Treat OGC API - Connected Systems Part 1 and Part 2, OGC API - Features, SensorML 3.0, SWE Common 3.0, OpenAPI 3.1, OGC schemas, JSON Schema, GeoJSON, media type specifications, AEP-4789 Volume II adoption context, and earlier IDR findings as controlling.
- Do not collapse all validation into schema validation. Distinguish structural, semantic, profile, operational, security, and interoperability validation.
- Do not finalize units, observed properties, controlled properties, or semantic binding policy here. Identify semantic validation dependencies and hand detailed work to `IDR-SRV-024`.
- Do not design the persistence layer here. Identify validation artifact storage, schema cache, and audit implications and hand them to Category E topics.
- Do not finalize command lifecycle semantics here. Identify command validation needs and hand detailed work to `IDR-SRV-036` and `IDR-SRV-037`.
- Do not finalize authorization or policy logic here. Identify policy and security validation needs and hand them to Category G topics.
- Do not assume official schemas, OpenAPI artifacts, examples, and implementations are always complete, synchronized, or mutually consistent. The research must identify conflicts and uncertainty explicitly.
- Do not treat implementation-specific validation behavior as normative unless confirmed by standards or project profile decisions.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What schema and encoding validation responsibilities must Glaux Server support?
2. Which validation responsibilities are structural, semantic, profile-specific, operational, security/policy, interoperability, or conformance-related?
3. Which schemas, OpenAPI artifacts, media types, and encodings are authoritative for each resource family and interaction point?
4. How should validation be staged across registration, update, import, ingestion, command submission, feasibility evaluation, response generation, CI, and conformance testing?
5. What downstream implications follow for semantic binding, persistence, ingestion, command validation, security, fixtures, conformance harnesses, and interoperability?

### Detailed Questions

#### Validation Authority and Source Baseline

- Which schemas, OpenAPI descriptions, examples, profile documents, and encoding rules apply to CSAPI Part 1 resources?
- Which schemas, OpenAPI descriptions, examples, profile documents, and encoding rules apply to CSAPI Part 2 dynamic-data, observations, status, events, control streams, commands, and feasibility resources?
- Which OGC API - Features validation expectations apply to collection/item behavior and GeoJSON feature representations?
- Which SensorML validation expectations apply to systems, procedures, deployments, capabilities, characteristics, contacts, identifiers, classifiers, positions, inputs, and outputs?
- Which SWE Common validation expectations apply to data components, result structures, encodings, units, nil values, constraints, observation values, and command parameters?
- Which AEP-4789 profile or standards-package expectations modify or constrain base OGC validation behavior?
- Which source should be treated as controlling when schemas, standards text, examples, OpenAPI artifacts, or implementation behavior conflict?

#### Validation Type Taxonomy

- How should Glaux Server define and distinguish:
  - syntax validation,
  - structural/schema validation,
  - OpenAPI validation,
  - media type validation,
  - content negotiation validation,
  - GeoJSON validation,
  - SensorML validation,
  - SWE Common validation,
  - profile validation,
  - semantic validation,
  - unit validation,
  - observed-property validation,
  - controlled-property validation,
  - link/reference validation,
  - temporal validation,
  - lifecycle validation,
  - relationship validation,
  - command/feasibility validation,
  - policy/releasability validation,
  - security validation,
  - conformance validation,
  - interoperability validation?
- Which validations are mandatory, recommended, optional, configurable, deferred, or profile-specific?
- Which validations should produce errors versus warnings?

#### Resource-Family Validation Scope

- What validation responsibilities apply to:
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
  - feasibility resources,
  - links and relationships,
  - metadata/document resources,
  - publisher/source inputs?
- Which validation rules are request-time rules?
- Which validation rules are response-time rules?
- Which validation rules are import-time or ingestion-time rules?
- Which validation rules are conformance/CI rules rather than runtime checks?

#### OpenAPI, JSON Schema, and Schema Artifact Strategy

- Which OpenAPI artifacts should be treated as validation sources?
- Which JSON Schema dialect/version issues affect validation?
- How should Glaux Server manage schema versions, schema references, external schemas, `$ref` resolution, schema caching, and schema artifact updates?
- How should OpenAPI validation interact with handwritten validation logic?
- How should generated clients and server stubs be validated against OpenAPI descriptions?
- Which discrepancies between standards text, OpenAPI artifacts, schemas, examples, and implementation behavior require documentation or escalation?
- Which findings should be handed to conformance harness and traceability topics?

#### Encoding and Content Negotiation Validation

- What validation is needed for media types, alternate representations, profile parameters, encoding parameters, Accept headers, Content-Type headers, and unsupported formats?
- How should Glaux Server validate JSON, GeoJSON, SensorML, SWE Common, and future binary or streaming encodings?
- How should content negotiation errors be represented?
- How should response encodings be validated against requested media types?
- How should encoding validation interact with `IDR-SRV-012`, `IDR-SRV-014`, `IDR-SRV-021`, `IDR-SRV-022`, and future streaming/event-publication topics?

#### Import, Registration, Update, and Response Validation

- What validation should occur when registering or importing systems, procedures, deployments, properties, datastreams, SensorML documents, SWE component definitions, or metadata?
- What validation should occur when updating resources?
- What validation should occur when generating responses?
- What validation should occur when preserving source documents?
- How should Glaux Server handle partially valid imported content?
- What warnings, errors, rejected records, quarantined records, normalization records, and audit records are needed?
- Which findings should be handed to registration/update, persistence, metadata storage, and audit topics?

#### Dynamic Data, Observation, Status, Event, and Ingestion Validation

- How should observations be validated against datastream definitions and SWE Common result structures?
- How should status values and dynamic properties be validated?
- How should system events be validated?
- How should null, nil, missing, out-of-range, unit-incompatible, type-incompatible, stale, duplicated, or out-of-order data be handled?
- How should ingestion validation differ from API-response validation?
- How should validation be configured for high-rate dynamic data without blocking operational throughput?
- Which findings should be handed to ingestion, time-series storage, streaming, status/event, and performance topics?

#### Command, Control Stream, and Feasibility Validation

- How should command payloads be validated against control-stream definitions and SWE Common input structures?
- How should feasibility requests and responses be validated?
- How should allowed values, ranges, units, defaults, modes, controlled properties, safety constraints, authorization constraints, and temporal constraints be validated?
- Which validation failures should reject commands immediately?
- Which validation failures should produce feasibility failures?
- Which validation failures should produce command-status updates or system events?
- Which findings should be handed to `IDR-SRV-036`, `IDR-SRV-037`, `IDR-SRV-038`, and security-test topics?

#### Error, Warning, and Diagnostic Behavior

- How should validation errors be represented using HTTP status codes and problem-detail responses?
- Which validation failures should return 400, 404, 409, 415, 422, 428, or other status codes?
- How detailed should validation error messages be?
- How should errors avoid leaking sensitive resource existence, schema details, command affordances, or policy-protected data?
- How should validation warnings be exposed or recorded?
- How should validation results support debugging, conformance testing, and audit without overexposing operational details?

#### Tooling, Runtime, CI, and Conformance Strategy

- Which validation should occur at runtime?
- Which validation should occur in CI?
- Which validation should occur in conformance harnesses?
- Which validation should occur in fixture-generation and golden-file tests?
- Which validation libraries, schema compilers, OpenAPI validators, GeoJSON validators, SensorML validators, SWE validators, or custom validators should be assessed?
- What validation results should be machine-readable for traceability?
- Which findings should be handed to `IDR-SRV-050`, `IDR-SRV-051`, `IDR-SRV-052`, and `IDR-SRV-053`?

#### Security, Policy, and Releasability Validation

- What policy validation must occur before accepting or exposing resource descriptions, SensorML documents, SWE Common structures, observations, status values, events, command definitions, or command submissions?
- How should validation account for classification, releasability, need-to-know, command authorization, source trust, provenance, and cross-boundary constraints?
- How should validation detect sensitive content embedded in otherwise valid metadata or component definitions?
- Which findings should be handed to Category G and security-test topics?

#### Implementation and Interoperability Lessons

- What schema and encoding validation lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate schema, OpenAPI, media type, content negotiation, GeoJSON, SensorML, SWE Common, command validation, or response-shape issues?
- What OS4CSAPI discussion lessons affect validation strategy?
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

- `IDR-SRV-001` through `IDR-SRV-022` research reports, once complete:
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

### Schema, Encoding, HTTP, and API Description Sources

- OpenAPI Specification 3.1: https://spec.openapis.org/oas/latest.html
- JSON Schema: https://json-schema.org/
- GeoJSON RFC 7946: https://www.rfc-editor.org/rfc/rfc7946
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 - HTTP Caching: https://www.rfc-editor.org/rfc/rfc9111
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- IANA Media Types Registry: https://www.iana.org/assignments/media-types/media-types.xhtml

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, validation guidance, encoding rules, or standards-package annexes relevant to Glaux Server

### Implementation and Lessons-Learned Sources

Use implementation-study outputs and source repositories as non-normative evidence:

- OSH / OpenSensorHub sources and `IDR-SRV-014A`
- Connected Systems Go sources and `IDR-SRV-014B`
- pygeoapi sources and `IDR-SRV-014C`
- SECD sources and `IDR-SRV-014D`
- OS4CSAPI Client Smoke Test Findings and `IDR-SRV-014E`
- SECD Interoperability Findings and `IDR-SRV-014F`
- OS4CSAPI Discussions Lessons-Learned and `IDR-SRV-014G`
- Category C findings from `IDR-SRV-015` through `IDR-SRV-020`
- SensorML Representation Strategy findings from `IDR-SRV-021`
- SWE Common Data Component Strategy findings from `IDR-SRV-022`
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

### Phase 1: Source Collection and Validation Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for schema and encoding validation.

**Tasks:**

1. Gather CSAPI, OGC API - Features, SensorML, SWE Common, OpenAPI, JSON Schema, GeoJSON, media type, HTTP, AEP-4789, prior IDR, implementation-study, smoke-test, interoperability, and discussion sources.
2. Extract schemas, OpenAPI artifacts, examples, media types, encodings, validation notes, validation failures, and validator/tooling references.
3. Define inventory fields:
   - resource family,
   - payload/representation type,
   - schema or encoding source,
   - validation type,
   - validation stage,
   - authority classification,
   - runtime/CI/conformance role,
   - error/warning behavior,
   - security/policy implication,
   - test implication,
   - downstream handoff.
4. Define classification values: normative, inherited, standards-package-specific, profile-specific, implementation-quality, interoperability-driven, implementation-specific, optional, deferred, and unresolved.
5. Establish evidence hierarchy for standards, AEP material, official schemas/OpenAPI artifacts, prior reports, implementation observations, and community discussion.

**Expected Output:** Source inventory and validation extraction framework.

### Phase 2: Standards, Schema, OpenAPI, and Encoding Inventory (3-4 hours)

**Objective:** Identify validation sources and their authority by representation and resource family.

**Tasks:**

1. Inventory CSAPI Part 1 schemas, OpenAPI operations, examples, encodings, and validation text.
2. Inventory CSAPI Part 2 schemas, OpenAPI operations, examples, encodings, and validation text.
3. Inventory OGC API - Features and GeoJSON validation requirements.
4. Inventory SensorML and SWE Common validation sources.
5. Inventory media type, content negotiation, HTTP, JSON Schema, and OpenAPI validation sources.
6. Inventory AEP-4789 profile or standards-package validation expectations from project-available material.
7. Identify conflicts, gaps, versioning issues, unresolved schema references, stale artifacts, or ambiguity.

**Expected Output:** Schema/OpenAPI/encoding authority matrix.

### Phase 3: Resource-Family and Interaction-Stage Validation Analysis (3-4 hours)

**Objective:** Define validation responsibilities by resource family and interaction stage.

**Tasks:**

1. Use `IDR-SRV-015` through `IDR-SRV-022` findings as the organizing baseline.
2. For each resource family, identify request validation, response validation, import validation, ingestion validation, update validation, command validation, feasibility validation, and conformance validation needs.
3. Distinguish structural, semantic, profile, operational, security/policy, and interoperability validation.
4. Identify required, optional, configurable, deferred, and unresolved validation rules.
5. Identify validation outcomes: accept, warn, normalize, reject, quarantine, defer, or audit.

**Expected Output:** Resource-family validation matrix.

### Phase 4: Dynamic Data, SensorML, SWE Common, and Command Validation Analysis (2.5-3.5 hours)

**Objective:** Analyze high-risk validation areas in depth.

**Tasks:**

1. Analyze observation result validation against datastream definitions and SWE Common structures.
2. Analyze status/dynamic-property and system-event validation.
3. Analyze SensorML import/generation/response validation.
4. Analyze SWE Common component, encoding, unit, nil-value, constraint, and semantic validation.
5. Analyze command/control and feasibility validation against parameter definitions, constraints, units, modes, temporal windows, safety rules, and authorization dependencies.
6. Identify handoffs to dynamic-data, command lifecycle, feasibility validation, semantic binding, and security topics.

**Expected Output:** High-risk validation domain matrix.

### Phase 5: Error Handling, Tooling, Runtime/CI/Conformance, and Test Implications (2.5-3 hours)

**Objective:** Prepare validation findings for implementation and verification strategy.

**Tasks:**

1. Define validation error and warning behavior, including HTTP status codes and problem-detail payload needs.
2. Identify runtime, import-time, ingestion-time, response-time, CI, conformance-harness, fixture-generation, and golden-file validation boundaries.
3. Identify validation tooling needs and candidate validator categories.
4. Identify schema cache, schema versioning, `$ref` resolution, and offline/DDIL validation implications.
5. Identify security and policy validation implications.
6. Identify conformance, fixture, golden-file, negative, regression, and interoperability tests.
7. Map findings to Category E, F, G, and I topics.

**Expected Output:** Validation tooling and downstream implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert schema and encoding validation research into a decision-usable baseline.

**Tasks:**

1. Consolidate schema/encoding source inventory, resource-family validation strategy, high-risk validation domain findings, and tooling/test implications.
2. Produce validation recommendations by resource family, representation type, and interaction stage.
3. Resolve conflicts and ambiguities where possible.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Authoritative schemas, OpenAPI artifacts, media types, encodings, and validation sources are identified with source anchors.
- [ ] Structural, semantic, profile, operational, security/policy, conformance, and interoperability validation types are distinguished.
- [ ] Resource-family and interaction-stage validation responsibilities are mapped.
- [ ] CSAPI, OGC API - Features, SensorML, SWE Common, GeoJSON, JSON Schema, OpenAPI, media type, and encoding validation implications are documented.
- [ ] Runtime, CI, conformance, fixture, golden-file, and interoperability validation boundaries are documented.
- [ ] Error, warning, normalization, rejection, quarantine, and audit behavior implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Schema and Encoding Validation Strategy Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-023-schema-and-encoding-validation-strategy-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Validation extraction methodology
5. Schema, OpenAPI, media type, and encoding inventory
6. Validation type taxonomy
7. Resource-family validation strategy
8. Interaction-stage validation strategy
9. SensorML and SWE Common validation findings
10. Dynamic-data, observation, status, event, command, and feasibility validation findings
11. Error, warning, normalization, rejection, quarantine, and audit findings
12. Runtime, CI, conformance, fixture, and golden-file validation findings
13. Tooling, schema-cache, `$ref`, and offline/DDIL validation implications
14. Security, policy, and releasability validation implications
15. Downstream topic handoff matrix
16. Recommendations
17. Risks, constraints, and open questions
18. Validation against this plan's success criteria
19. References

The validation matrix should include, at minimum:

- Resource family
- Interaction stage
- Payload/representation type
- Schema/encoding source
- Validation type
- Authority classification
- Runtime/CI/conformance role
- Error/warning behavior
- Security/policy implication
- Tooling implication
- Test implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-022` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, SensorML 3.0, SWE Common 3.0, OGC schemas, OpenAPI artifacts, JSON Schema, GeoJSON, media type, HTTP, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-024: Units, Observed Properties, and Semantic Binding Strategy`
- `IDR-SRV-028: Metadata and Document Storage Strategy`
- `IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy`
- `IDR-SRV-031: Server Write and Ingestion Model`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-037: Feasibility and Asynchronous Tasking Strategy`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints`
- `IDR-SRV-041: Audit Logging and Accountability Strategy`
- `IDR-SRV-042: DDIL-Informed Server Semantics`
- `IDR-SRV-043: Server Synchronization and Conflict Handling Boundary`
- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
- `IDR-SRV-052: Rust Test-Driven Architecture and Multi-Layer Test Strategy`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
- `IDR-SRV-055: Security, Authorization, and Command-Control Test Strategy`
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

- This topic defines validation strategy, not final implementation code or the conformance harness itself.
- Schema validation alone is not sufficient; semantic, profile, operational, security, and interoperability validation must be distinguished.
- Implementation-study findings are useful but must not override standards-derived validation requirements.
- Open question: Which official schema and OpenAPI artifacts are authoritative when artifacts and prose differ?
- Open question: Which validation should happen at runtime versus CI/conformance testing only?
- Open question: How should Glaux Server handle partially valid imported SensorML or SWE Common definitions?
- Open question: Which validation failures should be warnings rather than hard errors?
- Open question: Which validation diagnostics are safe to expose to clients?
- Risk: Overly strict validation could reject useful operational data or diverge from real-world implementations.
- Risk: Under-validation could allow invalid resources, broken clients, unsafe command submissions, or bad conformance results.
- Risk: Treating examples or one implementation's behavior as normative could distort Glaux Server validation.
- Risk: Validation error messages may leak sensitive schema, resource, or command information if not designed carefully.

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
- OpenAPI Specification 3.1: https://spec.openapis.org/oas/latest.html
- JSON Schema: https://json-schema.org/
- GeoJSON RFC 7946: https://www.rfc-editor.org/rfc/rfc7946
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 - HTTP Caching: https://www.rfc-editor.org/rfc/rfc9111
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- IANA Media Types Registry: https://www.iana.org/assignments/media-types/media-types.xhtml
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

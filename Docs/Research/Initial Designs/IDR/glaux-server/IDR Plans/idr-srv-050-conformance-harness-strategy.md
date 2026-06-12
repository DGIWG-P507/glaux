# Section 050: Conformance Harness Strategy - Research Plan

**Status:** Planned  
**Last Updated:** June 12, 2026  
**Estimated Research Time:** 16-20 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-050-conformance-harness-strategy-report.md`

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

This topic research must define the Glaux Server planning baseline for a **conformance harness strategy** that can verify, document, and preserve evidence that `glaux-server` implements the relevant STANAG 4789 / AEP-4789 server responsibilities and OGC API - Connected Systems conformance classes correctly.

The research must answer:

- What conformance harness architecture is needed for Glaux Server to test CSAPI Part 1, CSAPI Part 2, SensorML, SWE Common, OpenAPI, HTTP behavior, media types, links, resource model behavior, query/filter/pagination behavior, content negotiation, error behavior, validation behavior, ingestion, dynamic data, streaming, command/control, policy/security, DDIL-informed behavior, and synchronization boundaries?
- Which tests should be automated conformance tests, API contract tests, integration tests, fixture tests, interoperability tests, smoke tests, negative tests, profile tests, or manual review evidence?
- How should the harness map requirements, conformance classes, test cases, fixtures, API requests, responses, expected assertions, evidence artifacts, and report outputs?
- What should be tested against a running server versus tested in Rust unit/integration layers?
- How should conformance evidence be generated, stored, versioned, compared, and used in CI?
- How should the harness integrate with future OGC TEAM Engine or other official OGC conformance tooling if available, while still providing Glaux-specific development evidence now?
- How should the conformance harness support multiple deployment profiles:
  - local development,
  - CI,
  - public demo,
  - conformance profile,
  - interoperability profile,
  - command-disabled profile,
  - streaming-enabled profile,
  - DDIL simulation profile?
- What downstream implications follow for requirement-to-test traceability, Rust TDD architecture, test data/fixtures, performance testing, security testing, interoperability testing, and final IDR synthesis?

The output must be a conformance harness strategy baseline with source anchors, conformance scope, requirement/test/evidence model, harness architecture options, fixture and profile expectations, CI integration guidance, evidence output recommendations, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic begins Category I: Verification and Implementation Readiness. Categories A through H establish standards obligations, API behavior, resource/domain model, representation, persistence, dynamic data, tasking, security, DDIL, synchronization, Rust platform, internal architecture, deployment, configuration, observability, and continuity. The conformance harness strategy must now translate those findings into a verification architecture that can prove behavior and preserve implementation evidence.

This topic directly informs:

- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
- `IDR-SRV-052: Rust Test-Driven Architecture and Multi-Layer Test Strategy`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
- `IDR-SRV-054: Performance, Load, Stress, and Streaming Test Strategy`
- `IDR-SRV-055: Security, Authorization, and Command-Control Test Strategy`
- `IDR-SRV-056: Interoperability Test Matrix for External CSAPI Clients`

### Critical Constraints

- Treat prior IDR findings as conformance and verification requirements.
- Do not assume official OGC conformance tooling already covers all Glaux Server needs. Distinguish official conformance, development conformance, implementation readiness, and interoperability evidence.
- Do not treat only happy-path responses as conformance. Negative cases, error semantics, media types, links, query behavior, security/policy constraints, and profile behavior must be addressed.
- Do not make the conformance harness depend on one local machine state, one public demo URL, one identity provider, or one non-repeatable data set.
- Do not include real credentials, operational data, sensitive policy labels, or controlled source data in conformance fixtures.
- Do not allow conformance tests to hide failures by overusing permissive comparisons.
- Do not conflate conformance tests with performance tests, security tests, or interoperability tests, but identify where they overlap.
- Keep the research bounded to Glaux Server conformance harness strategy and evidence model.

---

## 2. Research Questions

### Core Questions

1. What conformance scope must the Glaux Server harness cover?
2. What harness architecture should be used to execute tests against a running server and collect evidence?
3. How should requirements, conformance classes, fixtures, assertions, test cases, and evidence artifacts be modeled?
4. How should conformance testing integrate with CI, local development, demo deployments, and future official OGC tooling?
5. What downstream implications follow for traceability, TDD, fixtures, performance, security, and interoperability testing?

### Detailed Questions

#### Standards and Requirement Scope

- Which CSAPI Part 1 conformance classes and requirements must be covered?
- Which CSAPI Part 2 conformance classes and requirements must be covered?
- Which OGC API - Features Part 1 behaviors are inherited or relevant?
- Which SensorML and SWE Common validation behaviors should be covered?
- Which OpenAPI and machine-readable API-description behaviors should be covered?
- Which AEP-4789 server responsibilities require conformance evidence even if not directly covered by OGC tests?
- Which Glaux-specific profiles require additional verification beyond base CSAPI conformance?

#### Harness Scope and Test Taxonomy

- What test categories should the harness distinguish:
  - standards conformance tests,
  - API contract tests,
  - integration tests,
  - negative/error tests,
  - media type/content negotiation tests,
  - link/navigation tests,
  - query/filter/pagination tests,
  - schema validation tests,
  - security/policy profile tests,
  - ingestion tests,
  - streaming/event tests,
  - command/control tests,
  - DDIL/degraded-mode tests,
  - synchronization/conflict tests,
  - interoperability smoke tests,
  - regression tests?
- Which tests belong in this conformance harness versus Category I follow-on topics?
- Which tests are blocking gates?
- Which are advisory evidence?

#### Harness Architecture Options

- What harness architecture should be evaluated:
  - Rust integration-test harness,
  - standalone CLI harness,
  - Python/pytest harness,
  - Postman/Newman collection,
  - Schemathesis/OpenAPI-driven testing,
  - OGC TEAM Engine integration,
  - hybrid Rust + external CLI harness,
  - CI pipeline orchestration?
- Which architecture best supports:
  - repeatability,
  - fixture setup/teardown,
  - profile selection,
  - evidence generation,
  - local developer runs,
  - CI runs,
  - external server URL testing,
  - official OGC tooling integration?
- Should the conformance harness live inside `glaux-server`, a separate repo, or a shared test workspace?

#### Test Target Strategy

- What test targets should be supported:
  - in-process server,
  - local running server,
  - Docker Compose reference deployment,
  - CI ephemeral deployment,
  - public demo endpoint,
  - external CSAPI implementation,
  - mock/fake service dependencies?
- Which tests require full deployment stack?
- Which can use in-process or test server?
- Which require database-backed state?
- Which require seeded fixtures?
- Which require streaming/broker support?
- Which require command gateway simulation?

#### Requirement-to-Test-to-Evidence Model

- What metadata must each test case carry:
  - requirement ID,
  - conformance class,
  - standard/source reference,
  - profile,
  - fixture dependency,
  - request,
  - expected response,
  - assertion set,
  - evidence output,
  - negative/positive classification,
  - automation status,
  - CI gate status?
- How should test cases link to prior IDR requirements?
- How should evidence artifacts be generated:
  - JSON test report,
  - JUnit XML,
  - Markdown summary,
  - request/response captures,
  - logs,
  - screenshots if needed,
  - schema validation output,
  - conformance matrix?
- How should evidence be stored and versioned?

#### Fixture and Data Strategy

- What fixture data is required for conformance:
  - landing page,
  - conformance declaration,
  - API definition,
  - collections,
  - systems,
  - deployments,
  - procedures,
  - sampling features,
  - properties,
  - datastreams,
  - observations,
  - status,
  - system events,
  - SensorML documents,
  - SWE Common structures,
  - source registrations,
  - command resources,
  - feasibility examples,
  - error examples?
- Which fixtures must be deterministic?
- Which fixtures must be generated?
- Which fixtures can be static golden files?
- Which fixtures require dynamic setup?
- Which fixtures are profile-specific?
- How should fixture versioning relate to standards versions and server versions?

#### API Behavior Coverage

- How should the harness verify:
  - landing page behavior,
  - links and relation types,
  - conformance declaration,
  - OpenAPI exposure,
  - collection metadata,
  - resource retrieval,
  - nested resources,
  - pagination,
  - query filters,
  - sorting,
  - temporal filters,
  - geospatial filters,
  - selection/projection,
  - content negotiation,
  - media types,
  - error/problem details?
- How should the harness compare responses without becoming brittle about permitted ordering or optional fields?
- Which responses require exact golden-file comparison?
- Which require schema/semantic assertions?

#### Schema and Encoding Validation

- How should the harness validate JSON, GeoJSON, SensorML, SWE Common, and problem-detail responses?
- Which schemas are authoritative?
- How should schema versions be pinned?
- How should local schema cache be used during CI?
- How should schema validation failures be reported as evidence?
- How should content negotiation and encoding selection be validated?

#### Negative and Error Testing

- What negative cases are required:
  - invalid media type,
  - unacceptable Accept header,
  - missing resource,
  - invalid query parameter,
  - invalid geometry,
  - invalid time range,
  - invalid pagination,
  - invalid command payload,
  - unauthorized request,
  - forbidden resource,
  - policy-hidden resource,
  - stale/degraded dependency,
  - unsupported operation?
- How should HTTP status codes and RFC 9457 problem details be asserted?
- How should redaction of sensitive error details be verified?

#### Security and Policy-Aware Conformance

- How should conformance tests handle auth-enabled and auth-disabled profiles?
- What tests verify object-level authorization and policy filtering without requiring a production identity provider?
- How should policy-hidden resources be tested without leaking through links, counts, extents, errors, or OpenAPI?
- Which security tests are core conformance evidence and which belong to `IDR-SRV-055`?
- How should the harness handle test tokens or fake identities safely?

#### Dynamic Data, Streaming, and Event Testing

- What dynamic data tests should be included in conformance harness scope:
  - observation retrieval,
  - latest values,
  - status updates,
  - system events,
  - ingestion acceptance,
  - event publication,
  - replay/backfill,
  - ordering expectations,
  - stale/delayed update semantics?
- Which streaming tests belong in conformance versus performance/stress testing?
- How should the harness test streaming without creating flaky CI behavior?
- What evidence should be collected for event tests?

#### Command/Control and Feasibility Testing

- What command/control conformance tests should be included:
  - control stream discovery,
  - command definition retrieval,
  - command payload validation,
  - feasibility request/response,
  - command submission in simulated profile,
  - command status transitions,
  - cancellation,
  - denial/safety cases,
  - audit evidence?
- Which command tests should be disabled in public demo?
- How should simulated command gateway fixtures support repeatable tests?
- Which tests belong to `IDR-SRV-055` rather than this topic?

#### DDIL and Synchronization Profile Testing

- What DDIL-informed semantics should the harness verify:
  - stale responses,
  - last-known values,
  - partial results,
  - dependency unavailable,
  - command-disabled degraded mode,
  - stale policy,
  - cached schema/profile use?
- What synchronization/conflict behaviors should be tested at a conformance-profile level:
  - duplicate replay,
  - conflict record visibility,
  - quarantine status,
  - event/replay evidence?
- Which detailed tests belong to performance/security/interoperability follow-on topics?

#### CI Integration

- How should the harness run in CI?
- What deployment profile should CI use?
- How should database and service dependencies be started?
- How should fixtures be loaded and reset?
- Which tests run on every PR?
- Which run nightly or manually?
- How should evidence artifacts be uploaded?
- How should flaky tests be detected and handled?

#### Local Developer Workflow

- How should developers run conformance tests locally?
- Should the harness support:
  - single test,
  - test group,
  - requirement ID filter,
  - conformance class filter,
  - profile filter,
  - external URL target?
- How should failures point developers to requirements and evidence?
- How should logs and request/response captures be made available safely?

#### Official OGC Tooling and External Harness Integration

- What official OGC conformance tooling exists or is planned for CSAPI?
- How should Glaux Server prepare for future official OGC conformance tests?
- Should the harness export data that can be consumed by TEAM Engine or other tools?
- Should it include wrappers for external tools?
- How should differences between official conformance and Glaux development evidence be documented?

#### Interoperability and External Client Evidence

- How should the harness prepare for external-client tests:
  - CSAPI Explorer,
  - OS4CSAPI client libraries,
  - webapp/mobile clients,
  - pygeoapi/OSH/Connected Systems Go comparison,
  - SECD interoperability tests?
- Which tests are conformance tests versus interoperability smoke tests?
- What evidence should be retained when an external client fails?

#### Performance and Security Relationship

- Which conformance tests have performance-sensitive variants?
- Which conformance tests need security variants?
- How should conformance results provide baselines for `IDR-SRV-054` and `IDR-SRV-055`?
- How should the harness avoid confusing conformance pass/fail with load/security readiness?

#### Implementation Lessons from Existing CSAPI Servers

- What conformance, testing, fixture, OpenAPI, schema validation, and client interoperability lessons can be extracted from OSH, Connected Systems Go, pygeoapi, SECD, and OS4CSAPI work?
- Which lessons relate to machine-testable assertions?
- Which lessons relate to insufficient fixtures or ambiguous standards behavior?
- Which patterns should Glaux Server adopt, adapt, or avoid?
- Which lessons are implementation-specific and not standards-derived?

---

## 3. Primary Resources

The future research report must analyze these sources directly and use current source dates, tool versions, and assumptions when making conformance harness recommendations.

### Project and Governance Sources

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md

### Prior IDR Topic Sources

- `IDR-SRV-001` through `IDR-SRV-049` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Controlling Standards and API Sources

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OGC schemas: https://schemas.opengis.net/
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- JSON Schema: https://json-schema.org/
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457

### OGC Conformance and Testing Sources

- OGC compliance program: https://www.ogc.org/compliance/
- OGC TEAM Engine: https://github.com/opengeospatial/teamengine
- OGC CITE tests repository organization: https://github.com/opengeospatial
- OGC API conformance examples, if available through OGC repositories
- OS4CSAPI test and conformance materials:
  - https://github.com/OS4CSAPI
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2

### Harness and Testing Tool Sources

Use current official documentation and primary-source material when executing the research:

- Rust cargo test: https://doc.rust-lang.org/cargo/commands/cargo-test.html
- cargo-nextest: https://nexte.st/
- axum testing patterns: https://docs.rs/axum/
- reqwest: https://docs.rs/reqwest/
- wiremock-rs: https://docs.rs/wiremock/
- testcontainers-rs: https://docs.rs/testcontainers/
- insta snapshot testing: https://docs.rs/insta/
- assert-json-diff: https://docs.rs/assert-json-diff/
- jsonschema crate: https://docs.rs/jsonschema/
- schemars crate: https://docs.rs/schemars/
- Schemathesis, if evaluated: https://schemathesis.readthedocs.io/
- Postman/Newman, if evaluated: https://learning.postman.com/docs/collections/using-newman-cli/command-line-integration-with-newman/
- pytest, if evaluated for external harness comparison: https://docs.pytest.org/
- JUnit XML report format references, if used by CI

### Deployment and Runtime Sources

- Docker documentation: https://docs.docker.com/
- Docker Compose documentation: https://docs.docker.com/compose/
- PostgreSQL documentation: https://www.postgresql.org/docs/
- PostGIS documentation: https://postgis.net/documentation/
- OpenTelemetry documentation: https://opentelemetry.io/docs/

### Implementation and Lessons-Learned Sources

Use implementation-study outputs and source repositories as non-normative evidence:

- OSH / OpenSensorHub sources and `IDR-SRV-014A`
- Connected Systems Go sources and `IDR-SRV-014B`
- pygeoapi sources and `IDR-SRV-014C`
- SECD sources and `IDR-SRV-014D`
- OS4CSAPI Client Smoke Test Findings and `IDR-SRV-014E`
- SECD Interoperability Findings and `IDR-SRV-014F`
- OS4CSAPI Discussions Lessons-Learned and `IDR-SRV-014G`
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
- Glaux Webapp repository, if available or created: https://github.com/DGIWG-P507/glaux-webapp
- Glaux Mobile repository, if available or created: https://github.com/DGIWG-P507/glaux-mobile
- Glaux Publisher repository, if available or created: https://github.com/DGIWG-P507/glaux-publisher
- Glaux Simulator repository, if available or created: https://github.com/DGIWG-P507/glaux-simulator
- OS4CSAPI testing research-plan corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

---

## 5. Research Methodology

### Phase 1: Conformance Scope Extraction (3-4 hours)

**Objective:** Convert prior IDR findings into conformance harness scope.

**Tasks:**

1. Extract conformance-relevant requirements from CSAPI Part 1, CSAPI Part 2, SensorML, SWE Common, OpenAPI, AEP-4789, and prior IDR reports.
2. Classify each test need by conformance, contract, integration, negative, security/profile, dynamic-data, command/control, DDIL, synchronization, interoperability, or advisory category.
3. Identify first-implementation and full-scope conformance priorities.
4. Identify official OGC conformance tooling opportunities and gaps.
5. Prepare conformance scope matrix.

**Expected Output:** Conformance scope and test taxonomy matrix.

### Phase 2: Harness Architecture and Target Strategy Analysis (4-5 hours)

**Objective:** Evaluate conformance harness architecture and test targets.

**Tasks:**

1. Compare Rust integration harness, standalone CLI harness, Python/pytest, Postman/Newman, Schemathesis/OpenAPI-driven testing, OGC TEAM Engine integration, and hybrid approaches.
2. Define supported test targets: in-process server, local running server, Compose stack, CI ephemeral deployment, public demo endpoint, external server URL.
3. Define fixture setup/teardown and profile selection requirements.
4. Define evidence artifact requirements.
5. Identify proof-of-concept needs.

**Expected Output:** Harness architecture and test target matrix.

### Phase 3: Requirement/Test/Evidence Model Analysis (3-4 hours)

**Objective:** Define the metadata and evidence model for conformance tests.

**Tasks:**

1. Define required test-case metadata.
2. Define requirement-to-test-to-evidence linkage.
3. Define evidence outputs: JSON report, JUnit XML, Markdown summary, request/response captures, schema validation output, logs, and conformance matrix.
4. Define storage/versioning approach for evidence artifacts.
5. Identify CI artifact capture and developer failure-reporting needs.

**Expected Output:** Requirement/test/evidence model.

### Phase 4: Functional Test Coverage Analysis (4-5 hours)

**Objective:** Define what the harness must test.

**Tasks:**

1. Analyze API behavior coverage: landing page, conformance, OpenAPI, collections, resources, links, query/filter/pagination, content negotiation, errors, and schemas.
2. Analyze SensorML/SWE, validation, negative/error, security/policy, dynamic data, streaming/events, command/control, DDIL, and synchronization coverage.
3. Define which tests belong to conformance harness versus follow-on performance/security/interoperability topics.
4. Define golden-file versus semantic assertion strategies.
5. Identify fixture needs and profile-gating constraints.

**Expected Output:** Functional conformance coverage matrix.

### Phase 5: CI, Local Workflow, Official Tooling, and External Interoperability Analysis (3-4 hours)

**Objective:** Prepare harness findings for implementation workflow and downstream testing.

**Tasks:**

1. Define CI execution strategy, gating levels, artifacts, and flaky-test controls.
2. Define local developer workflow and filtering by requirement/conformance/profile/test group.
3. Analyze future official OGC tooling integration and external harness compatibility.
4. Analyze interoperability smoke-test handoff to `IDR-SRV-056`.
5. Map findings to traceability, TDD, fixtures, performance, security, and interoperability topics.

**Expected Output:** CI/local/official-tooling/interoperability matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Produce a decision-usable conformance harness strategy.

**Tasks:**

1. Consolidate conformance scope, harness architecture, target strategy, requirement/test/evidence model, functional coverage, CI/local workflow, and downstream findings.
2. Produce recommended first-implementation and full-scope conformance harness strategy.
3. Identify proof-of-concept needs and downstream handoffs.
4. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Conformance scope is defined with source anchors and prior-topic traceability.
- [ ] Test taxonomy distinguishes conformance, contract, integration, negative, security/profile, dynamic-data, streaming, command/control, DDIL, synchronization, interoperability, and advisory tests.
- [ ] Harness architecture options and test target strategies are evaluated.
- [ ] Requirement/test/evidence model is documented.
- [ ] Fixture, profile, CI, local workflow, and evidence artifact requirements are documented.
- [ ] API behavior, schema/encoding, negative/error, security/policy, dynamic-data, streaming, command/control, DDIL, and synchronization coverage implications are documented.
- [ ] Official OGC tooling integration opportunities and limitations are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Conformance Harness Strategy Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-050-conformance-harness-strategy-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Conformance scope extraction methodology
5. Conformance scope and test taxonomy
6. Harness architecture evaluation
7. Test target strategy
8. Requirement/test/evidence model
9. Evidence artifact and CI reporting findings
10. Fixture and profile requirements
11. API behavior coverage findings
12. Schema/encoding/validation and negative/error coverage findings
13. Security/policy, dynamic-data, streaming/event, command/control, DDIL, and synchronization coverage findings
14. CI and local developer workflow findings
15. Official OGC tooling and external harness integration findings
16. Downstream topic handoff matrix
17. Recommendations
18. Risks, constraints, and open questions
19. Validation against this plan's success criteria
20. References

The conformance harness matrix should include, at minimum:

- Test category
- Requirement/conformance source
- Test target
- Fixture dependency
- Profile applicability
- Assertion type
- Evidence artifact
- CI gate status
- Related downstream topic
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-049` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, SensorML, SWE Common, OGC API - Features, OpenAPI, JSON Schema, HTTP, and problem-detail sources must be reachable.
- Official or candidate OGC conformance tooling sources must be reachable or explicitly marked unavailable.
- Implementation-study and interoperability findings must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
- `IDR-SRV-052: Rust Test-Driven Architecture and Multi-Layer Test Strategy`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
- `IDR-SRV-054: Performance, Load, Stress, and Streaming Test Strategy`
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

- This topic defines conformance harness strategy, not the complete implementation of the harness.
- Official OGC conformance coverage for CSAPI may be incomplete or evolving; document gaps rather than assuming coverage.
- Glaux-specific conformance evidence must be clearly distinguished from official certification evidence.
- Open question: Should the first harness be Rust-native, external CLI, Python/pytest, or hybrid?
- Open question: How much request/response evidence should be stored in CI artifacts?
- Open question: Which tests should be blocking on every pull request versus nightly/manual?
- Open question: How should security/policy tests be represented without requiring production identity/policy infrastructure?
- Open question: How should streaming tests avoid flakiness while still producing evidence?
- Risk: Overly permissive assertions may allow non-conformant behavior to pass.
- Risk: Overly brittle golden files may create noise and slow development.
- Risk: Harness scope creep could duplicate performance, security, and interoperability test topics.
- Risk: Lack of evidence versioning could make conformance claims difficult to reproduce.

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
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OGC schemas: https://schemas.opengis.net/
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- JSON Schema: https://json-schema.org/
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- OGC compliance program: https://www.ogc.org/compliance/
- OGC TEAM Engine: https://github.com/opengeospatial/teamengine
- Rust cargo test: https://doc.rust-lang.org/cargo/commands/cargo-test.html
- cargo-nextest: https://nexte.st/
- reqwest: https://docs.rs/reqwest/
- wiremock-rs: https://docs.rs/wiremock/
- testcontainers-rs: https://docs.rs/testcontainers/
- insta snapshot testing: https://docs.rs/insta/
- assert-json-diff: https://docs.rs/assert-json-diff/
- jsonschema crate: https://docs.rs/jsonschema/
- Schemathesis: https://schemathesis.readthedocs.io/
- Postman/Newman: https://learning.postman.com/docs/collections/using-newman-cli/command-line-integration-with-newman/
- pytest: https://docs.pytest.org/
- Docker documentation: https://docs.docker.com/
- Docker Compose documentation: https://docs.docker.com/compose/
- PostgreSQL documentation: https://www.postgresql.org/docs/
- PostGIS documentation: https://postgis.net/documentation/
- OpenTelemetry documentation: https://opentelemetry.io/docs/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

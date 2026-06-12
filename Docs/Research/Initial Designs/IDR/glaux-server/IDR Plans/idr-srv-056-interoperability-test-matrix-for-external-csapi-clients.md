# Section 056: Interoperability Test Matrix for External CSAPI Clients - Research Plan

**Status:** Planned  
**Last Updated:** June 12, 2026  
**Estimated Research Time:** 16-20 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md`

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

This topic research must define the Glaux Server planning baseline for an **interoperability test matrix for external CSAPI clients**. The matrix must show how `glaux-server` should be tested with external clients, viewers, libraries, demonstration tools, reference implementations, and peer servers to verify practical interoperability beyond internal conformance tests.

The research must answer:

- Which external clients and interoperability targets should Glaux Server test against:
  - CSAPI Explorer,
  - OS4CSAPI TypeScript client-library work,
  - OS4CSAPI smoke-test clients,
  - Glaux Webapp,
  - Glaux Mobile,
  - Glaux Publisher,
  - Glaux Simulator,
  - QGIS / OGC API capable clients,
  - generic OpenAPI clients,
  - browser/fetch clients,
  - OWSLib or Python clients,
  - pygeoapi CSAPI implementation,
  - OpenSensorHub / OSH implementation,
  - Connected Systems Go implementation,
  - SECD interoperability implementation,
  - other OGC API clients?
- What interoperability behaviors must be verified:
  - landing page discovery,
  - conformance declaration,
  - API definition discovery,
  - collections navigation,
  - link relation traversal,
  - resource retrieval,
  - query/filter/pagination,
  - content negotiation,
  - JSON/GeoJSON interpretation,
  - SensorML interpretation,
  - SWE Common interpretation,
  - observation access,
  - status access,
  - event/system-event access,
  - streaming/event subscription,
  - command/control discovery,
  - feasibility and command workflows,
  - error handling,
  - authentication and policy-filtered behavior?
- What scenarios should be tested across clients and profiles:
  - read-only discovery,
  - dynamic data access,
  - streaming-enabled profile,
  - command-disabled public demo,
  - command-enabled simulated profile,
  - DDIL/degraded-mode profile,
  - security/policy profile,
  - interoperability regression profile?
- How should interoperability evidence be captured and reported:
  - client/version,
  - server version,
  - profile,
  - dataset/scenario,
  - tested operation,
  - expected behavior,
  - observed behavior,
  - pass/fail/partial,
  - screenshots/logs/request-response captures,
  - issue links,
  - compatibility notes?
- How should interoperability findings feed back into server design, conformance tests, fixtures, OpenAPI descriptions, client-library issues, and final IDR synthesis?

The output must be an interoperability test matrix strategy with source anchors, external-client inventory, test scenario taxonomy, matrix structure, evidence model, profile mapping, automation/manual split, compatibility-reporting rules, downstream synthesis handoff, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows:

- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
- `IDR-SRV-052: Rust Test-Driven Architecture and Multi-Layer Test Strategy`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
- `IDR-SRV-054: Performance, Load, Stress, and Streaming Test Strategy`
- `IDR-SRV-055: Security, Authorization, and Command-Control Test Strategy`

Those topics establish internal correctness, traceability, fixtures, performance testing, and security/command testing. This topic defines how Glaux Server should prove practical interoperability with independent clients and implementations before final IDR synthesis.

### Critical Constraints

- Treat prior IDR findings as interoperability test requirements, especially API behavior, links, content negotiation, resource model, SensorML/SWE, dynamic data, streaming, command/control, policy/security, DDIL, fixtures, conformance harness, and traceability findings.
- Do not treat interoperability testing as a substitute for standards conformance testing.
- Do not require every external client to support every CSAPI capability. The matrix must distinguish client capability gaps from server defects.
- Do not rely only on manual visual testing. Define automated, semi-automated, and manual test categories.
- Do not use real operational data, credentials, sensitive policy labels, real command targets, or uncontrolled public feeds in required interoperability tests.
- Do not expose sensitive logs, traces, screenshots, policy decisions, source identities, command details, or tokens in public interoperability evidence.
- Do not make public demo interoperability tests capable of real command dispatch.
- Keep the research bounded to Glaux Server external-client interoperability testing and evidence strategy.

---

## 2. Research Questions

### Core Questions

1. Which external clients, tools, libraries, and peer implementations should be included in the interoperability matrix?
2. What CSAPI behaviors and Glaux Server profiles should be tested against each target?
3. How should interoperability results distinguish server defects, client defects, standards ambiguity, profile limitation, and optional capability gaps?
4. What evidence should be captured for automated, semi-automated, and manual interoperability tests?
5. How should interoperability findings feed final implementation readiness and IDR synthesis?

### Detailed Questions

#### External Client and Tool Inventory

- Which clients/tools should be evaluated:
  - CSAPI Explorer,
  - OS4CSAPI TypeScript client,
  - OS4CSAPI smoke-test scripts,
  - Glaux Webapp,
  - Glaux Mobile,
  - Glaux Publisher,
  - Glaux Simulator,
  - OWSLib/Python clients,
  - generic OpenAPI-generated clients,
  - browser/fetch examples,
  - curl/httpie scripts,
  - QGIS/OGC API clients,
  - Esri or other OGC API consumers if available?
- Which are primary interoperability targets?
- Which are optional/advisory?
- Which are future targets?
- What capability profile does each client support?

#### Peer Server and Reference Implementation Inventory

- Which peer servers should be examined:
  - OpenSensorHub / OSH,
  - Connected Systems Go,
  - pygeoapi CSAPI implementation,
  - SECD,
  - other OS4CSAPI servers or demos?
- Should Glaux Server tests include client-against-peer and peer-client-against-Glaux comparisons?
- Which differences are server implementation choices versus standards issues?
- How should findings from peer server implementation studies be used?

#### Interoperability Scenario Taxonomy

- What scenario categories are required:
  - landing page discovery,
  - conformance discovery,
  - OpenAPI discovery,
  - collections navigation,
  - resource traversal,
  - link relation traversal,
  - JSON parsing,
  - GeoJSON parsing,
  - SensorML parsing,
  - SWE Common parsing,
  - query/filter/pagination,
  - observations,
  - status,
  - system events,
  - streaming/events,
  - command/control discovery,
  - command-disabled behavior,
  - command-enabled simulated workflow,
  - error handling,
  - authentication,
  - policy-filtered results,
  - DDIL/degraded responses?
- Which are first-implementation scenarios?
- Which are full-scope readiness scenarios?
- Which are manual visual scenarios?
- Which are automated regression scenarios?

#### Matrix Structure

- What columns should the interoperability matrix include:
  - matrix ID,
  - client/tool,
  - client version/commit,
  - server version/commit,
  - server profile,
  - fixture/scenario,
  - operation,
  - expected behavior,
  - observed behavior,
  - result,
  - classification,
  - evidence artifact,
  - issue link,
  - retest date,
  - notes?
- What result classifications should exist:
  - pass,
  - fail-server,
  - fail-client,
  - partial,
  - blocked,
  - not supported,
  - not applicable,
  - standards ambiguity,
  - deferred?
- How should matrix entries link to requirement IDs and tests?

#### Discovery and Navigation Interoperability

- How should clients discover the server:
  - landing page,
  - service links,
  - API definition,
  - conformance endpoint,
  - collections,
  - collection items,
  - nested resources?
- What link relations must clients follow?
- Which link relation differences are tolerable?
- How should relative versus absolute links be tested?
- How should reverse proxy/base URL configuration be tested?

#### OpenAPI and Schema Interoperability

- How should OpenAPI compatibility be tested:
  - document retrieval,
  - validation by parsers,
  - code generation,
  - ReDoc/Swagger rendering,
  - content types,
  - examples,
  - security schemes,
  - profile-specific operations?
- Which clients rely heavily on OpenAPI?
- How should OpenAPI drift be detected?
- How should schema/profile references be tested?

#### Resource Model Interoperability

- How should clients consume:
  - systems,
  - deployments,
  - procedures,
  - sampling features,
  - properties,
  - datastreams,
  - control streams,
  - observations,
  - commands,
  - feasibility resources,
  - system events?
- Which fields are mandatory for practical client rendering?
- Which optional fields improve interoperability?
- How should unknown extensions be tolerated?
- How should clients behave when optional fields are omitted?

#### Query, Filter, Sorting, Pagination, and Selection Interoperability

- Which query patterns should be tested:
  - pagination,
  - temporal filters,
  - spatial filters,
  - ID filters,
  - property filters,
  - sorting,
  - selection/projection,
  - combined filters?
- How should clients handle pagination links?
- How should clients handle empty result sets?
- How should clients handle policy-filtered result sets?
- How should query failures be reported?

#### Media Type and Content Negotiation Interoperability

- Which Accept and Content-Type combinations should be tested?
- Which clients send broad, narrow, or incorrect Accept headers?
- How should server behavior support practical clients while remaining standards-correct?
- How should JSON, GeoJSON, SensorML, SWE Common, and problem details be negotiated?
- Which failures indicate server defects versus client assumptions?

#### SensorML and SWE Common Interoperability

- Which clients parse SensorML?
- Which clients parse SWE Common?
- What fixture examples should be used?
- How should unit/observed-property metadata be interpreted?
- How should clients display or use procedure/output/parameter information?
- How should malformed or unsupported SensorML/SWE be handled?

#### Dynamic Data and Observation Interoperability

- How should clients retrieve and display observations?
- What time-series query patterns should be tested?
- How should latest values be tested?
- How should status values be tested?
- How should large observation payloads be paginated or filtered?
- Which clients can consume dynamic data automatically?

#### Streaming and Event Interoperability

- Which clients support streaming or subscriptions?
- Which protocols should be tested:
  - SSE,
  - WebSocket,
  - MQTT/NATS/Kafka if exposed through profile,
  - polling fallback?
- What scenarios are needed:
  - subscribe,
  - receive event,
  - filter stream,
  - reconnect,
  - replay,
  - slow consumer,
  - policy-filtered events?
- Which streaming tests are automated?
- Which are manual?
- How should event evidence be captured?

#### Command-Control Interoperability

- Which clients should discover command/control resources?
- Which clients should submit simulated commands?
- Which clients should only verify command-disabled behavior?
- What command scenarios are required:
  - control stream discovery,
  - command definition parsing,
  - feasibility request,
  - command submission,
  - command status polling,
  - cancellation,
  - denied command,
  - safety failure?
- How should command-enabled interoperability remain simulated-only?
- How should public demo command behavior be tested safely?

#### Authentication and Policy Interoperability

- Which clients can handle:
  - no-auth local profile,
  - static test token,
  - bearer token,
  - API key,
  - OIDC token,
  - policy-filtered responses?
- How should authorization failures be tested?
- How should clients handle 401/403/404 hidden-resource behavior?
- How should policy-filtered collections and links be represented?
- How should sensitive evidence be redacted?

#### Error Handling Interoperability

- How should clients handle:
  - RFC 9457 problem details,
  - invalid query,
  - unsupported media type,
  - unacceptable Accept header,
  - missing resource,
  - unauthorized,
  - forbidden,
  - command disabled,
  - policy unavailable,
  - degraded dependency?
- Which clients require stable error schemas?
- Which errors should be in the core interoperability matrix?
- How should error-handling evidence be captured?

#### DDIL and Degraded-Mode Interoperability

- What degraded-mode responses should external clients see:
  - stale data,
  - last-known values,
  - partial results,
  - dependency unavailable,
  - replay gap,
  - command disabled,
  - sync backlog?
- Which clients can present stale/degraded status?
- Which metadata is needed for clients to avoid misleading users?
- Which DDIL tests are first-implementation versus future?

#### Public Demo Interoperability

- Which interoperability scenarios should the public demo support?
- Which features should be disabled or simulated:
  - real command dispatch,
  - unrestricted ingestion,
  - admin diagnostics,
  - sensitive policy/security tests?
- Which clients should be tested against the demo before publication?
- How should demo reset/reseed affect interoperability tests?
- How should demo evidence be captured without sensitive data?

#### Automation Strategy

- Which interoperability tests can be automated:
  - curl/httpie smoke tests,
  - TypeScript client tests,
  - Python client tests,
  - OpenAPI parser tests,
  - schema validation tests,
  - CSAPI Explorer smoke tests if scriptable,
  - browser E2E tests if practical?
- Which tests are semi-automated?
- Which require manual review?
- Which should run in CI?
- Which should run nightly or before release?
- Which should be manual for public demo?

#### Evidence and Reporting

- What evidence artifacts should be captured:
  - request/response captures,
  - client logs,
  - server logs,
  - screenshots,
  - HAR files,
  - JSON reports,
  - Markdown summaries,
  - issue links,
  - video/GIF only if needed?
- How should sensitive evidence be redacted?
- How should evidence be versioned by client and server version?
- How should matrix reports be generated?
- How should findings feed final synthesis?

#### Issue Classification and Feedback Loop

- How should interoperability failures be classified:
  - Glaux Server defect,
  - client defect,
  - ambiguous standard,
  - optional capability not supported,
  - profile limitation,
  - fixture defect,
  - configuration/deployment issue,
  - test issue?
- How should issues be filed?
- How should fixes be retested?
- How should upstream client or standards feedback be captured?
- How should lessons feed traceability and conformance tests?

#### Interoperability with Glaux Ecosystem Components

- How should the matrix include:
  - Glaux Webapp,
  - Glaux Mobile,
  - Glaux Publisher,
  - Glaux Simulator?
- What server endpoints and profiles do these components require?
- How should simulator/publisher integration test ingestion and dynamic updates?
- How should web/mobile clients test discovery, visualization, status, observations, and command-disabled behavior?
- Which ecosystem tests are part of server readiness?

#### Implementation Lessons from Existing CSAPI Work

- What interoperability lessons can be extracted from OS4CSAPI client work, SECD interoperability work, OSH, Connected Systems Go, pygeoapi, and OS4CSAPI discussions?
- Which lessons relate to links, OpenAPI, resource shapes, content negotiation, observations, streaming, tasking, demo readiness, and client assumptions?
- Which patterns should Glaux Server adopt, adapt, or avoid?
- Which lessons are implementation-specific and not standards-derived?

---

## 3. Primary Resources

The future research report must analyze these sources directly and use current source dates, tool versions, and assumptions when making interoperability test recommendations.

### Project and Governance Sources

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md

### Prior IDR Topic Sources

- `IDR-SRV-001` through `IDR-SRV-055` research reports, once complete:
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
- GeoJSON RFC 7946: https://www.rfc-editor.org/rfc/rfc7946
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457

### External Client and Implementation Sources

Use current sources and repositories when executing the research:

- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI TypeScript client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- OpenSensorHub / OSH project: https://github.com/opensensorhub
- Connected Systems Go, if available through project/community sources
- pygeoapi project: https://github.com/geopython/pygeoapi
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- OWSLib project: https://github.com/geopython/OWSLib
- QGIS project: https://github.com/qgis/QGIS
- OpenAPI Generator: https://openapi-generator.tech/

### Browser, API, and Automation Tool Sources

- Playwright documentation, if browser automation is evaluated: https://playwright.dev/
- Cypress documentation, if browser automation is evaluated: https://docs.cypress.io/
- Postman/Newman documentation: https://learning.postman.com/docs/collections/using-newman-cli/command-line-integration-with-newman/
- Schemathesis: https://schemathesis.readthedocs.io/
- curl documentation: https://curl.se/docs/
- httpie documentation: https://httpie.io/docs/
- GitHub Actions documentation: https://docs.github.com/actions

### Implementation and Lessons-Learned Sources

Use implementation-study outputs and source repositories as non-normative evidence:

- OSH / OpenSensorHub sources and `IDR-SRV-014A`
- Connected Systems Go sources and `IDR-SRV-014B`
- pygeoapi sources and `IDR-SRV-014C`
- SECD sources and `IDR-SRV-014D`
- OS4CSAPI Client Smoke Test Findings and `IDR-SRV-014E`
- SECD Interoperability Findings and `IDR-SRV-014F`
- OS4CSAPI Discussions Lessons-Learned and `IDR-SRV-014G`

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

### Phase 1: Interoperability Target Inventory (3-4 hours)

**Objective:** Identify external clients, peer implementations, ecosystem components, and test targets.

**Tasks:**

1. Inventory external clients and tools.
2. Inventory peer servers and reference implementations.
3. Classify targets by capability: discovery, resources, observations, streaming, command/control, auth, policy, OpenAPI, SensorML/SWE.
4. Identify first-implementation, public-demo, CI, and future targets.
5. Prepare client/tool capability inventory.

**Expected Output:** External target inventory and capability profile matrix.

### Phase 2: Scenario and Matrix Model Analysis (4-5 hours)

**Objective:** Define interoperability scenarios and matrix structure.

**Tasks:**

1. Define scenario taxonomy for discovery, navigation, API definition, resources, queries, dynamic data, streaming, command/control, errors, security/policy, DDIL, and demo behavior.
2. Define interoperability result classifications.
3. Define matrix fields and evidence fields.
4. Define requirement/test/fixture traceability integration.
5. Define automated, semi-automated, and manual scenario categories.

**Expected Output:** Interoperability scenario taxonomy and matrix model.

### Phase 3: Client-Specific Test Strategy Analysis (4-5 hours)

**Objective:** Define how each target should be tested.

**Tasks:**

1. Analyze CSAPI Explorer testing needs.
2. Analyze OS4CSAPI client-library testing needs.
3. Analyze Glaux Webapp/Mobile/Publisher/Simulator testing needs.
4. Analyze generic OpenAPI/browser/Python/client testing needs.
5. Analyze peer server and comparative implementation testing needs.
6. Identify client capability gaps versus server defects.

**Expected Output:** Client-specific interoperability test matrix.

### Phase 4: Functional Interoperability Coverage Analysis (4-5 hours)

**Objective:** Define functional behavior to test across clients.

**Tasks:**

1. Analyze discovery/navigation/OpenAPI/schema interoperability.
2. Analyze resource/query/content-negotiation/SensorML/SWE interoperability.
3. Analyze dynamic data, streaming, command-control, security/policy, error, and DDIL interoperability.
4. Define public demo interoperability tests.
5. Define evidence capture and redaction needs.

**Expected Output:** Functional interoperability coverage matrix.

### Phase 5: Automation, CI, Reporting, and Feedback Loop Analysis (2-3 hours)

**Objective:** Prepare interoperability strategy for implementation workflow and final synthesis.

**Tasks:**

1. Define automated and manual test execution tiers.
2. Define evidence capture, redaction, and reporting artifacts.
3. Define issue classification and feedback loop.
4. Define CI/nightly/manual/demo/release-candidate placement.
5. Map findings to final IDR synthesis.

**Expected Output:** Interoperability automation/reporting/feedback matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Produce a decision-usable interoperability test matrix strategy.

**Tasks:**

1. Consolidate target inventory, scenario taxonomy, matrix model, client-specific tests, functional coverage, automation tiers, evidence model, and feedback loop.
2. Produce recommended first-implementation and full-scope interoperability matrix.
3. Identify proof-of-concept needs and unresolved questions.
4. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] External client, tool, ecosystem component, and peer implementation inventory is documented with source anchors and prior-topic traceability.
- [ ] Interoperability scenario taxonomy and matrix model are documented.
- [ ] Client/tool capability profiles and test strategy are documented.
- [ ] Discovery, navigation, OpenAPI, resource, query, content negotiation, SensorML/SWE, dynamic data, streaming, command/control, security/policy, error, DDIL, and public demo interoperability tests are documented.
- [ ] Result classifications distinguish server defects, client defects, standards ambiguity, unsupported capabilities, profile limitations, fixture defects, and test issues.
- [ ] Automated, semi-automated, manual, CI, nightly, public-demo, and release-candidate tiers are documented.
- [ ] Evidence capture, redaction, issue feedback, and retest workflow are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Final synthesis handoff is explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Interoperability Test Matrix for External CSAPI Clients Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Interoperability target inventory methodology
5. External client/tool and peer implementation inventory
6. Capability profile matrix
7. Interoperability scenario taxonomy
8. Interoperability matrix model
9. Discovery, navigation, OpenAPI, and schema interoperability findings
10. Resource, query, content negotiation, SensorML, and SWE Common interoperability findings
11. Dynamic data, streaming/event, command-control, security/policy, error, and DDIL interoperability findings
12. Glaux ecosystem component interoperability findings
13. Public demo interoperability findings
14. Automation, CI, manual execution, reporting, and evidence findings
15. Issue classification, feedback, and retest workflow findings
16. Final synthesis handoff matrix
17. Recommendations
18. Risks, constraints, and open questions
19. Validation against this plan's success criteria
20. References

The interoperability test matrix should include, at minimum:

- Matrix ID
- Client/tool/peer implementation
- Client version/commit
- Server version/commit
- Server profile
- Fixture/scenario
- Operation tested
- Expected behavior
- Observed behavior
- Result classification
- Evidence artifact
- Related requirement/test IDs
- Issue link
- Retest status
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-055` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, SensorML, SWE Common, OGC API - Features, OpenAPI, JSON Schema, HTTP, GeoJSON, and problem-detail sources must be reachable.
- Conformance, traceability, Rust TDD, fixture, performance, and security strategy findings must be available or explicitly marked unavailable/deferred.
- External client and peer implementation sources must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-057: Final Glaux Server IDR Synthesis Report`

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

- This topic defines interoperability test strategy and matrix design, not final execution results.
- Interoperability pass/fail must distinguish server issues from client capability gaps and standards ambiguity.
- Public demo tests must be safe, command-disabled or simulated, and free of sensitive data.
- Open question: Which clients should be first-implementation mandatory targets?
- Open question: Can CSAPI Explorer testing be automated sufficiently for CI or only semi-automated/manual?
- Open question: Which peer server comparisons are practical and current?
- Open question: How should OpenAPI-generated clients be incorporated without creating excessive matrix size?
- Open question: What evidence artifacts are useful without leaking sensitive diagnostics or tokens?
- Risk: External client versions may change and make results stale.
- Risk: Client capability gaps may be misinterpreted as server defects.
- Risk: Manual visual interoperability tests may be hard to reproduce.
- Risk: Public demo interoperability evidence may expose internal data if redaction is weak.

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
- GeoJSON RFC 7946: https://www.rfc-editor.org/rfc/rfc7946
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI TypeScript client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- OpenSensorHub / OSH project: https://github.com/opensensorhub
- pygeoapi project: https://github.com/geopython/pygeoapi
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- OWSLib project: https://github.com/geopython/OWSLib
- QGIS project: https://github.com/qgis/QGIS
- OpenAPI Generator: https://openapi-generator.tech/
- Playwright documentation: https://playwright.dev/
- Cypress documentation: https://docs.cypress.io/
- Postman/Newman documentation: https://learning.postman.com/docs/collections/using-newman-cli/command-line-integration-with-newman/
- Schemathesis: https://schemathesis.readthedocs.io/
- curl documentation: https://curl.se/docs/
- httpie documentation: https://httpie.io/docs/
- GitHub Actions documentation: https://docs.github.com/actions
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

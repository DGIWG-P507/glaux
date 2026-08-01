# Section 045: Service Architecture and Modularization Strategy - Research Plan

**Status:** Planned  
**Last Updated:** June 12, 2026  
**Estimated Research Time:** 14-18 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-045-service-architecture-and-modularization-strategy-report.md`

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

This topic research must define the Glaux Server planning baseline for **service architecture and modularization strategy** for a Rust-based, standards-aligned, test-driven Glaux Server implementation. The research must translate prior IDR findings into a practical server architecture: crate/module boundaries, domain-service boundaries, API layers, persistence boundaries, validation layers, ingestion boundaries, streaming/event boundaries, command/control boundaries, security/policy hooks, DDIL/synchronization boundaries, observability boundaries, and testability boundaries.

The research must answer:

- What should the internal service architecture of `glaux-server` look like so it remains standards-correct, maintainable, testable, secure, and ready for future deployment profiles?
- Should Glaux Server begin as a modular monolith, multi-crate workspace, independently deployable services, plugin-capable server, or staged hybrid architecture?
- What module boundaries best support CSAPI Part 1 and Part 2 behavior, SensorML/SWE Common representations, domain modeling, persistence, validation, dynamic-data ingestion, streaming, command/control, policy/security, DDIL semantics, synchronization, and conformance testing?
- Which concerns should be core server modules and which should remain optional adapters, publishers, deployment services, external brokers, reverse proxies, identity providers, or future extensions?
- How should Glaux Server avoid over-coupling API handlers to persistence schema, standards models to database records, command logic to transport mechanisms, or policy logic to one deployment environment?
- How should the implementation support a phased roadmap: first demonstrable CSAPI server, conformance-ready server, dynamic-data server, command-capable server, DDIL-aware server, and operationally deployable reference server?
- What downstream implications follow for reference deployment, configuration/secrets, observability, migrations, conformance harness, TDD strategy, fixtures, performance testing, security testing, and interoperability?

The output must be a service architecture and modularization strategy baseline with source anchors, architectural option evaluation, recommended initial architecture, module/crate boundary guidance, dependency-direction rules, extension points, internal contract recommendations, phased implementation guidance, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows:

- `IDR-SRV-044: Rust Implementation Language and Framework Strategy`

`IDR-SRV-044` evaluates Rust language, framework, tooling, and implementation-platform choices. This topic applies those platform findings to the internal service architecture and modularization of Glaux Server. It should shape deployment strategy, configuration/secrets, observability, migrations, conformance harnesses, Rust TDD architecture, fixture strategy, performance testing, security testing, and interoperability testing.

### Critical Constraints

- Treat prior IDR findings as architectural requirements, not optional features.
- Do not design the full deployment topology here. Define service/module boundaries and hand deployment packaging and topology to `IDR-SRV-046`.
- Do not over-split the first implementation into distributed microservices unless evidence shows that is necessary. Evaluate modular monolith, workspace, service extraction, and plugin boundaries explicitly.
- Do not couple server core to a single broker, identity provider, policy engine, database extension, external schema registry, or deployment profile.
- Do not place standards conformance behavior only in edge handlers. Domain services, validation services, and test harnesses must be able to enforce server obligations.
- Do not place command safety, source trust, policy/releasability, audit, or validation solely in clients, publishers, adapters, or reverse proxies.
- Do not create abstractions that hide standards semantics, make testing harder, or prevent clear requirement-to-test traceability.
- Keep the research bounded to Glaux Server internal architecture and module/service boundaries.

---

## 2. Research Questions

### Core Questions

1. What internal architectural style best fits Glaux Server: modular monolith, multi-crate workspace, service-oriented architecture, plugin-capable server, or staged hybrid?
2. What module/crate/service boundaries should exist for API, domain, validation, persistence, ingestion, streaming, command/control, security/policy, DDIL/synchronization, observability, and conformance?
3. What dependency-direction rules and internal contracts are needed to keep standards behavior testable and maintainable?
4. What extension points should be planned without overengineering first implementation?
5. What downstream implications follow for deployment, configuration, observability, migrations, conformance, testing, fixtures, performance, security, and interoperability?

### Detailed Questions

#### Architecture Style Evaluation

- Which architectural styles should be evaluated:
  - single binary with internal modules,
  - modular monolith,
  - Cargo workspace with multiple internal crates,
  - hexagonal/ports-and-adapters architecture,
  - clean architecture,
  - layered architecture,
  - service-oriented architecture,
  - independently deployable microservices,
  - plugin/extension architecture,
  - staged hybrid?
- How do these styles compare for:
  - standards correctness,
  - testability,
  - implementation speed,
  - operational simplicity,
  - deployment flexibility,
  - DDIL suitability,
  - command safety,
  - conformance testing,
  - CI complexity,
  - contributor onboarding?
- What is the recommended first implementation style?
- What future extraction points should be preserved?

#### Crate and Module Boundary Strategy

- Should `glaux-server` be one crate or a Cargo workspace?
- What internal crates/modules should be considered:
  - `api`,
  - `domain`,
  - `csapi`,
  - `sensorml`,
  - `swe`,
  - `validation`,
  - `persistence`,
  - `ingestion`,
  - `events`,
  - `commands`,
  - `security`,
  - `policy`,
  - `ddil`,
  - `sync`,
  - `observability`,
  - `config`,
  - `test-support`,
  - `conformance`?
- Which modules are core?
- Which modules are optional or profile-gated?
- Which modules should be private internal implementation details?
- Which modules should expose stable internal interfaces for tests and future extension?

#### Dependency Direction and Layering

- What dependency rules should apply:
  - API depends on domain services, not database schema directly.
  - Domain does not depend on web framework.
  - Validation can be invoked by API, ingestion, commands, and tests.
  - Persistence implements repositories/ports.
  - Security/policy hooks are called consistently by API, streaming, ingestion, and commands.
  - Observability is cross-cutting but should not own business logic.
- Should the architecture use ports/adapters for persistence, identity, policy, broker, schema registry, and command gateways?
- Where should DTO-to-domain conversion occur?
- Where should link construction and URI generation occur?
- Where should RFC 9457 error mapping occur?

#### API Layer Boundaries

- How should API routing be organized:
  - landing page,
  - conformance,
  - OpenAPI,
  - collections,
  - systems,
  - deployments,
  - procedures,
  - sampling features,
  - properties,
  - datastreams,
  - observations,
  - status,
  - events,
  - control streams,
  - commands,
  - feasibility,
  - ingestion,
  - source registration,
  - admin/health?
- How should API handlers stay thin and testable?
- How should content negotiation, query parsing, pagination, link rendering, media type selection, problem details, and response envelopes be modularized?
- How should handler tests differ from domain-service tests and conformance tests?

#### Standards Representation Boundaries

- How should CSAPI, SensorML, SWE Common, GeoJSON, and problem-detail models be organized?
- Should standards models be separated from internal domain models?
- How should the server handle extension fields and implementation-specific metadata without polluting standards-facing models?
- How should SensorML/SWE document handling avoid coupling all server logic to raw document structures?
- How should standards models be reused in fixtures, conformance harnesses, and generated documentation?

#### Domain Service Boundaries

- What domain services are needed:
  - resource catalog service,
  - relationship/link service,
  - identifier/lifecycle service,
  - temporal/freshness service,
  - status/event service,
  - source registration/trust service,
  - ingestion service,
  - datastream/observation service,
  - command service,
  - feasibility service,
  - policy decision service,
  - audit service,
  - DDIL/degraded-mode service,
  - synchronization/conflict service?
- Which services are pure domain logic?
- Which services coordinate persistence and external dependencies?
- Which services require transactions?
- Which services require idempotency keys or event outbox integration?

#### Validation Architecture Boundaries

- How should validation be modularized:
  - schema validation,
  - OpenAPI validation,
  - CSAPI requirement validation,
  - SensorML validation,
  - SWE Common validation,
  - domain rule validation,
  - semantic/unit validation,
  - source trust validation,
  - command validation,
  - policy validation?
- Which validation is synchronous versus asynchronous?
- Which validation belongs before persistence, after staging, before publication, or before command dispatch?
- How should validation artifacts be produced and tested?
- How should validation logic be reusable by API handlers, ingestion pipelines, command workflows, and conformance harnesses?

#### Persistence and Repository Boundaries

- How should persistence modules be organized:
  - metadata/resources,
  - documents,
  - geospatial,
  - time-series observations,
  - status/latest values,
  - events/outbox,
  - commands,
  - source trust,
  - policy metadata,
  - audit,
  - synchronization/conflicts?
- Should repositories expose domain objects, storage DTOs, query result views, or stream cursors?
- How should transactions be coordinated across repositories?
- How should idempotency and concurrency control be centralized?
- How should migrations and bootstrap data be separated from runtime repositories?

#### Ingestion and Publisher/Adapter Boundaries

- How should ingestion pipeline stages be modularized:
  - receipt,
  - source authentication,
  - source trust lookup,
  - payload decoding,
  - validation,
  - normalization,
  - persistence,
  - event generation,
  - audit,
  - error/quarantine?
- Which parts belong in Glaux Server versus `glaux-publisher` or adapters?
- How should ingestion support multiple input patterns without overcoupling to one protocol?
- How should ingestion remain testable with fixtures and simulated publishers?

#### Streaming and Event Publication Boundaries

- How should event publication be modularized:
  - durable event store,
  - outbox worker,
  - subscription service,
  - protocol adapters,
  - replay/backfill,
  - filtering/policy,
  - metrics?
- Which event logic belongs in domain services versus broker/protocol adapters?
- Should streaming support be optional at first implementation?
- How should event publication avoid tight coupling to MQTT/NATS/Kafka?
- How should event publication be tested without external brokers?

#### Command and Control Boundaries

- How should command lifecycle logic be modularized?
- Should command lifecycle transitions be implemented through a state-machine service?
- How should command validation, feasibility, authorization, safety, dispatch, status updates, events, and audit be separated?
- Which command dispatch paths should be abstracted as ports?
- How should simulated command gateways support testing?
- How should command functionality be feature-gated or disabled in certain profiles?

#### Security and Policy Boundaries

- How should authentication, authorization, policy/releasability, source trust, command safety, audit, and redaction be separated?
- Should authorization and policy be represented as pluggable ports?
- Where should object-level authorization be enforced?
- Where should policy filtering/redaction be applied?
- How should long-lived streaming authorization be re-evaluated?
- How should security decisions be testable without a production identity provider?

#### DDIL and Synchronization Boundaries

- How should DDIL mode, cache freshness, degraded behavior, synchronization state, conflict records, and local/remote authority be represented in modules?
- Which logic belongs in runtime state/configuration versus persistence-backed domain state?
- How should synchronization boundary logic avoid becoming full deployment/network architecture?
- Which parts should be reusable by API responses, ingestion, commands, event publication, and observability?

#### Observability and Operational Boundaries

- How should logging, metrics, tracing, health checks, readiness checks, and diagnostics be modularized?
- Which modules emit domain events versus operational telemetry?
- Where should correlation IDs, source IDs, command IDs, and event IDs be attached?
- How should redaction be enforced in logs and traces?
- How should observability support tests without production observability infrastructure?

#### Configuration and Profile Boundaries

- How should runtime profiles be represented:
  - local development,
  - CI,
  - public demo,
  - operational,
  - tactical-edge,
  - command-disabled,
  - streaming-disabled,
  - auth-disabled local mode,
  - DDIL/degraded mode?
- Which modules are always active?
- Which modules are feature-gated, configuration-gated, or deployment-gated?
- How should invalid configuration fail safely?
- How should profile-specific behavior remain visible to tests and documentation?

#### Internal API Contracts and Extension Points

- What internal interfaces should be explicitly defined:
  - repository traits,
  - validation services,
  - policy decision service,
  - identity context,
  - event publisher,
  - command dispatcher,
  - schema resolver,
  - clock/time provider,
  - ID generator,
  - configuration provider,
  - audit writer?
- Which interfaces should be traits?
- Which should remain concrete until needed?
- How should mock/fake implementations support tests?
- Which extension points are needed for future publisher, simulator, federation, broker, and command-gateway integrations?

#### Testability and Conformance Architecture

- How should architecture support:
  - unit tests,
  - integration tests,
  - API contract tests,
  - conformance tests,
  - database-backed tests,
  - golden-file tests,
  - fixture-driven tests,
  - property-based tests,
  - security tests,
  - performance tests?
- Which modules require test support helpers?
- How should test support avoid leaking into production code?
- How should conformance evidence be generated from internal architecture?
- How should architecture support traceability from requirements to tests?

#### Monorepo, Workspace, and Repository Layout

- What repository layout should be considered:
  - single `glaux-server` crate,
  - workspace with internal crates,
  - examples,
  - tests,
  - fixtures,
  - migrations,
  - docs,
  - OpenAPI artifacts,
  - scripts?
- How should generated artifacts be stored?
- Where should schema caches, fixtures, and golden files live?
- How should local dev tooling and CI scripts be organized?
- Which layout best supports future maintainers and DGIWG/OGC collaborators?

#### Implementation Phasing

- What architecture is needed for:
  - phase 1 minimal landing/conformance/API-definition behavior,
  - phase 2 resource catalog and link behavior,
  - phase 3 persistence/query behavior,
  - phase 4 dynamic-data ingestion/observation behavior,
  - phase 5 streaming/events,
  - phase 6 command/control,
  - phase 7 security/policy,
  - phase 8 DDIL/synchronization,
  - phase 9 conformance/performance/interoperability readiness?
- Which modules can be stubs early?
- Which modules must be designed correctly before first implementation?
- What architectural decisions are hard to reverse?

#### Implementation Lessons from Existing CSAPI Servers

- What architecture/modularization lessons can be extracted from OSH, Connected Systems Go, pygeoapi, SECD, and OS4CSAPI client work?
- Which lessons relate to handler structure, model separation, validation placement, conformance testing, link construction, OpenAPI drift, streaming, command support, or storage boundaries?
- Which patterns should Glaux Server adopt, adapt, or avoid?
- Which lessons are implementation-specific and not standards-derived?

---

## 3. Primary Resources

The future research report must analyze these sources directly and use current source dates, crate versions, and assumptions when making implementation recommendations.

### Project and Governance Sources

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md

### Prior IDR Topic Sources

- `IDR-SRV-001` through `IDR-SRV-044` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Architecture and Rust Implementation Sources

Use current official documentation and primary-source material when executing the research:

- Rust documentation: https://doc.rust-lang.org/
- Rust API Guidelines: https://rust-lang.github.io/api-guidelines/
- Cargo Book: https://doc.rust-lang.org/cargo/
- Tokio documentation: https://tokio.rs/
- axum documentation: https://docs.rs/axum/
- tower documentation: https://docs.rs/tower/
- hyper documentation: https://docs.rs/hyper/
- serde documentation: https://serde.rs/
- SQLx documentation: https://docs.rs/sqlx/
- tracing documentation: https://docs.rs/tracing/
- OpenTelemetry Rust documentation: https://docs.rs/opentelemetry/
- cargo-nextest: https://nexte.st/
- cargo-deny: https://github.com/EmbarkStudios/cargo-deny
- RustSec Advisory Database: https://rustsec.org/

### Controlling Standards and API Sources

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OGC schemas: https://schemas.opengis.net/
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- JSON Schema: https://json-schema.org/
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457

### Database, Messaging, Security, and Observability Sources

- PostgreSQL documentation: https://www.postgresql.org/docs/
- PostGIS documentation: https://postgis.net/documentation/
- TimescaleDB documentation: https://docs.timescale.com/
- MQTT v5 specification: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- NATS documentation: https://docs.nats.io/
- Apache Kafka documentation: https://kafka.apache.org/documentation/
- CloudEvents specification: https://cloudevents.io/
- OWASP API Security Top 10: https://owasp.org/API-Security/
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

### Phase 1: Architecture Requirements Extraction (2-3 hours)

**Objective:** Convert prior IDR findings into architectural requirements.

**Tasks:**

1. Extract service/module requirements from standards, API behavior, resource model, representation, persistence, ingestion, streaming, tasking, security, policy, DDIL, synchronization, and Rust platform topics.
2. Identify first-implementation versus full-scope architecture needs.
3. Identify cross-cutting concerns: validation, security, policy, audit, observability, errors, configuration, transactions, idempotency, and testing.
4. Define architecture evaluation criteria:
   - standards correctness,
   - testability,
   - modularity,
   - maintainability,
   - security,
   - operational simplicity,
   - deployment readiness,
   - DDIL suitability,
   - conformance support,
   - contributor ergonomics.
5. Prepare architecture comparison matrices.

**Expected Output:** Architecture requirements and evaluation framework.

### Phase 2: Architecture Style and Boundary Evaluation (3-4 hours)

**Objective:** Evaluate major service architecture styles and boundary options.

**Tasks:**

1. Compare modular monolith, multi-crate workspace, service-oriented architecture, microservices, ports-and-adapters, clean architecture, layered architecture, and plugin-capable patterns.
2. Evaluate which style fits first implementation and future extraction paths.
3. Define dependency-direction rules and layering constraints.
4. Identify hard-to-reverse architectural decisions.
5. Identify overengineering risks and under-modularization risks.

**Expected Output:** Architecture style evaluation matrix and recommended baseline style.

### Phase 3: Module, Crate, and Internal Contract Analysis (4-5 hours)

**Objective:** Define internal module/crate boundaries and stable internal contracts.

**Tasks:**

1. Propose module/crate boundaries for API, domain, standards models, validation, persistence, ingestion, events, commands, security, policy, audit, DDIL, synchronization, configuration, observability, conformance, and test support.
2. Identify internal interfaces for repositories, validation, policy decisions, identity context, event publisher, command dispatcher, schema resolver, clock/time provider, ID generator, audit writer, and configuration.
3. Classify each interface as immediate, deferred, concrete-only, trait-based, feature-gated, or external-adapter candidate.
4. Define DTO/domain/storage conversion boundaries.
5. Identify test doubles and fixture hooks needed for each boundary.

**Expected Output:** Module/crate/internal contract matrix.

### Phase 4: Cross-Cutting Concern Integration Analysis (3-4 hours)

**Objective:** Ensure cross-cutting behavior is consistently handled across modules.

**Tasks:**

1. Analyze how validation, error mapping, content negotiation, link generation, security, policy, audit, observability, transactions, idempotency, DDIL mode, and synchronization state flow through the architecture.
2. Identify where cross-cutting concerns must be centralized versus locally applied.
3. Identify common request context and operation context needs.
4. Identify how command-control and ingestion workflows preserve safety and audit boundaries.
5. Identify risks of inconsistent enforcement.

**Expected Output:** Cross-cutting concern architecture matrix.

### Phase 5: Phasing, Repository Layout, Testing, and Downstream Handoff Analysis (3-4 hours)

**Objective:** Convert architecture findings into implementation sequencing and downstream needs.

**Tasks:**

1. Define recommended repository/workspace layout.
2. Define phased implementation slices and module activation order.
3. Identify stubs, mocks, fakes, and test-support utilities needed for TDD.
4. Map architecture decisions to deployment, configuration, observability, migrations, conformance, fixtures, performance, security testing, and interoperability topics.
5. Identify proof-of-concept tasks needed before final implementation commitment.

**Expected Output:** Phased architecture and downstream handoff matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Produce a decision-usable service architecture and modularization baseline.

**Tasks:**

1. Consolidate architecture style recommendation, module boundaries, internal contracts, dependency rules, cross-cutting concern guidance, repository layout, and phasing.
2. Produce recommended service architecture and modularization strategy with rationale and unresolved questions.
3. Identify downstream handoffs and POC needs.
4. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Service architecture styles are evaluated with source anchors and Glaux Server requirement traceability.
- [ ] Recommended first implementation architecture style is documented with rationale.
- [ ] Module/crate boundaries, dependency-direction rules, and internal contract candidates are documented.
- [ ] API, domain, standards model, validation, persistence, ingestion, streaming, command/control, security, policy, audit, DDIL, synchronization, configuration, observability, conformance, and test-support boundaries are addressed.
- [ ] Cross-cutting concern handling is documented for validation, errors, content negotiation, links, security, policy, audit, observability, transactions, idempotency, DDIL, and synchronization.
- [ ] Repository/workspace layout and phased implementation guidance are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Service Architecture and Modularization Strategy Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-045-service-architecture-and-modularization-strategy-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Architecture requirement extraction methodology
5. Architecture style evaluation
6. Recommended initial architecture style
7. Module/crate boundary findings
8. Dependency-direction and layering findings
9. Internal contract and extension point findings
10. API/domain/standards-model boundary findings
11. Validation/persistence/ingestion/event/command boundary findings
12. Security/policy/audit/DDIL/synchronization boundary findings
13. Observability/configuration/test-support boundary findings
14. Repository/workspace layout findings
15. Phased implementation guidance
16. Proof-of-concept needs
17. Downstream topic handoff matrix
18. Recommendations
19. Risks, constraints, and open questions
20. Validation against this plan's success criteria
21. References

The service architecture matrix should include, at minimum:

- Module or service area
- Responsibility
- Inputs/outputs
- Depends on
- Must not depend on
- Internal contract/interface
- Persistence interaction
- Security/policy hook
- Validation responsibility
- Audit/observability responsibility
- Test strategy
- Feature/profile gating
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-044` research reports should be complete or explicitly marked unavailable/deferred.
- Official Rust, framework, database, OpenAPI, validation, testing, and observability sources must be reachable.
- Official CSAPI Part 1 and Part 2, SensorML, SWE Common, and relevant OpenAPI/schema artifacts must be reachable.
- Implementation-study and interoperability findings must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-046: Reference Deployment Strategy`
- `IDR-SRV-047: Configuration, Secrets, and Environment Strategy`
- `IDR-SRV-048: Observability, Logs, Metrics, and Health Check Strategy`
- `IDR-SRV-049: Migration, Upgrade, Backup, and Restore Strategy`
- `IDR-SRV-050: Conformance Harness Strategy`
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

- This topic defines internal server architecture, not final deployment topology.
- The likely first-implementation bias should be toward a modular monolith or workspace architecture unless evidence shows distributed services are necessary.
- Internal contracts should support testability and future extension without hiding standards behavior.
- Open question: Which modules deserve separate crates immediately versus later?
- Open question: Should policy, identity, broker, command dispatcher, and schema resolver be trait-based from day one?
- Open question: How should standards-facing models differ from internal domain models?
- Open question: How should conformance harness needs influence module boundaries?
- Risk: Over-splitting into microservices could slow implementation and increase operational complexity.
- Risk: Under-modularizing could make conformance, command safety, policy enforcement, and DDIL behavior hard to test.
- Risk: Coupling handlers directly to database models could undermine standards correctness and future interoperability.
- Risk: Over-abstracting too early could obscure requirement-to-test traceability.

---

## References

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- Rust documentation: https://doc.rust-lang.org/
- Rust API Guidelines: https://rust-lang.github.io/api-guidelines/
- Cargo Book: https://doc.rust-lang.org/cargo/
- Tokio documentation: https://tokio.rs/
- axum documentation: https://docs.rs/axum/
- tower documentation: https://docs.rs/tower/
- serde documentation: https://serde.rs/
- SQLx documentation: https://docs.rs/sqlx/
- tracing documentation: https://docs.rs/tracing/
- OpenTelemetry Rust documentation: https://docs.rs/opentelemetry/
- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- PostgreSQL documentation: https://www.postgresql.org/docs/
- PostGIS documentation: https://postgis.net/documentation/
- TimescaleDB documentation: https://docs.timescale.com/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

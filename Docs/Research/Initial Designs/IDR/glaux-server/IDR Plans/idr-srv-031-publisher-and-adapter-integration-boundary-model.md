# Section 031: Publisher and Adapter Integration Boundary Model - Research Plan

**Status:** Planned  
**Last Updated:** June 11, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-031-publisher-and-adapter-integration-boundary-model-report.md`

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

This topic research must define the Glaux Server planning baseline for the **publisher and adapter integration boundary model** across external sources, simulators, feed adapters, ingestion publishers, legacy/non-standard sensor feeds, OSH/OSH-Connected publishers, MQTT or message-bus feeds, direct API submission, file/batch imports, source-side metadata, dynamic data, observations, status updates, system events, SensorML, SWE Common, command/control boundaries, validation, trust, provenance, and DDIL-informed operation.

The research must answer:

- What responsibilities belong inside Glaux Server versus outside Glaux Server in publishers, adapters, simulators, source systems, gateways, brokers, and external data providers?
- What integration contracts must exist between Glaux Server and publishers/adapters for registering sources, submitting metadata, publishing observations, sending status updates, reporting system events, creating/updating datastream definitions, and carrying SensorML/SWE Common structures?
- Which integration boundary behaviors are required by CSAPI, SensorML, SWE Common, AEP-4789 server responsibilities, or Glaux Server full-scope goals, and which are implementation support choices?
- How should Glaux Server distinguish trusted sources, untrusted sources, adapter-generated content, source-authoritative content, inferred content, normalized content, and simulated/test content?
- What should publishers/adapters do before data reaches Glaux Server, and what validation, normalization, security, provenance, and lifecycle responsibilities must remain inside Glaux Server?
- How should the boundary support high-rate ingestion, idempotency, replay, DDIL operation, source registration, source health, schema/profile validation, semantic binding, policy/releasability controls, auditability, conformance testing, fixtures, and interoperability?
- What downstream implications follow for external source registration/trust, dynamic ingestion, datastream/observation semantics, streaming/event publication, command/control, DDIL synchronization, security, deployment, test fixtures, and performance testing?

The output must be a publisher and adapter integration boundary model with source anchors, responsibility allocation, interface patterns, source classes, trust/provenance guidance, integration contract recommendations, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic begins Category F: Dynamic Data, Ingestion, and Tasking. It follows:

- Category C resource/domain-model work (`IDR-SRV-015` through `IDR-SRV-020`)
- Category D representation and validation work (`IDR-SRV-021` through `IDR-SRV-024`)
- Category E persistence, query, transaction, and configuration work (`IDR-SRV-025` through `IDR-SRV-030`)

Those topics define what Glaux Server must store, validate, expose, and configure. This topic defines the boundary between Glaux Server and the external publishers/adapters that feed the server. It must precede external source registration/trust, dynamic data ingestion, datastream/observation semantics, streaming/event publication, and command/control lifecycle work.

### Critical Constraints

- Treat CSAPI Part 1 and Part 2, SensorML 3.0, SWE Common 3.0, AEP-4789 server responsibilities, prior IDR findings, and Glaux Server full-scope objectives as controlling.
- Do not design every adapter implementation here. Define the integration boundary, responsibility split, interface expectations, and downstream requirements.
- Do not move server responsibilities into adapters merely for convenience. Glaux Server must retain validation, policy, identity, provenance, persistence, and conformance-critical authority where required.
- Do not assume all publishers are trusted, online, standards-compliant, synchronized, or capable of producing complete CSAPI/SensorML/SWE structures.
- Do not assume all sources use the same protocol. Evaluate API submission, broker-based publication, file/batch import, simulator integration, and legacy feed mediation patterns.
- Do not finalize trust/registration policy here. Identify source-registration and trust requirements and hand detailed work to `IDR-SRV-032`.
- Do not finalize ingestion pipeline internals here. Identify pipeline boundary requirements and hand detailed ingestion design to `IDR-SRV-033`.
- Do not finalize streaming/event publication here. Identify publication/backfill/source-event needs and hand detailed design to `IDR-SRV-035`.
- Do not finalize command/control dispatch here. Identify adapter boundary implications and hand detailed command work to `IDR-SRV-036` through `IDR-SRV-038`.
- Keep the research bounded to Glaux Server behavior and server-side integration contracts.

---

## 2. Research Questions

### Core Questions

1. What responsibilities belong to Glaux Server versus publishers, adapters, simulators, gateways, brokers, and external data sources?
2. What integration contracts must exist for metadata, datastream definitions, observations, status updates, events, source health, and source provenance?
3. How should source trust, validation, normalization, idempotency, replay, and DDIL behavior be handled at the server-adapter boundary?
4. How should the boundary support standards-aligned CSAPI behavior while allowing legacy and non-standard sources to participate through adapters?
5. What downstream implications follow for source registration/trust, ingestion, streaming, command/control, security, deployment, fixtures, and performance testing?

### Detailed Questions

#### Standards and Source Boundary Baseline

- What publisher/adapter/source responsibilities are explicitly defined by CSAPI Part 1 or Part 2?
- What server responsibilities are implied by CSAPI Part 1 and Part 2 even when data originates outside the server?
- What SensorML and SWE Common responsibilities can be delegated to source systems or adapters, and what must be validated or preserved by Glaux Server?
- What AEP-4789 server responsibilities imply specific boundaries between source systems, mediation adapters, gateways, and the server?
- What does prior resource, validation, persistence, transaction, and configuration research imply for publisher/adapter boundaries?
- Which boundary behaviors are standards-driven versus architecture choices?

#### Publisher and Adapter Role Taxonomy

- What classes of external contributors should be recognized:
  - standards-compliant CSAPI publishers,
  - OSH/OSH-Connected publishers,
  - legacy sensor feed adapters,
  - simulator publishers,
  - file/batch importers,
  - brokered message publishers,
  - tactical gateways,
  - federated CSAPI servers,
  - external data providers,
  - command/control gateways,
  - test fixture publishers?
- Which classes provide authoritative metadata?
- Which classes provide only raw data?
- Which classes generate metadata by inference or mapping?
- Which classes should be treated as untrusted, partially trusted, mission-scoped, test-only, or simulated?
- Which classes should be registered as first-class sources?

#### Responsibility Allocation

- Which responsibilities should belong to publishers/adapters:
  - source-specific protocol handling,
  - source-specific parsing,
  - source-specific credential handling,
  - initial data extraction,
  - source-side buffering,
  - source-side sequence tracking,
  - local mapping hints,
  - source health reporting,
  - source metadata packaging?
- Which responsibilities should belong to Glaux Server:
  - canonical identity assignment,
  - CSAPI resource persistence,
  - authoritative validation,
  - policy/releasability enforcement,
  - provenance recording,
  - canonical relationship handling,
  - lifecycle state,
  - conformance behavior,
  - canonical error responses,
  - audit records,
  - command safety gates?
- Which responsibilities may be shared or configurable?
- Which responsibilities must not be delegated outside Glaux Server?

#### Integration Contract Patterns

- What integration patterns should be evaluated:
  - direct REST API submission,
  - CSAPI-native publication,
  - internal ingestion API,
  - message broker topic publication,
  - MQTT publication,
  - file/batch import,
  - local adapter sidecar,
  - gateway-mediated publication,
  - federated pull from external CSAPI endpoints,
  - simulator-driven publication?
- What are the tradeoffs for each pattern across standards alignment, validation, throughput, DDIL behavior, testability, deployment complexity, and security?
- Which patterns are likely needed for first implementation versus full-scope readiness?
- Which patterns should be treated as external ecosystem responsibilities rather than Glaux Server core?

#### Metadata and Description Submission Boundary

- How should publishers/adapters submit or reference systems, procedures, deployments, sampling features, properties, datastreams, SensorML documents, SWE Common structures, semantic bindings, and source metadata?
- How should Glaux Server handle incomplete, inferred, conflicting, profile-divergent, or externally authored metadata?
- Which metadata should be source-authoritative?
- Which metadata should require server approval, validation, or normalization before becoming canonical?
- How should source metadata updates interact with resource lifecycle, versioning, relationships, and system events?
- Which findings should be handed to source registration/trust, metadata storage, and validation topics?

#### Observation, Status, and Event Submission Boundary

- How should publishers/adapters submit observations, status updates, dynamic properties, system events, health reports, and source telemetry?
- Which fields must be supplied by the publisher versus assigned by Glaux Server?
- How should publisher-supplied timestamps, source sequence numbers, message identifiers, source offsets, and provenance be handled?
- How should missing or invalid datastream definitions affect observation acceptance?
- How should source-generated events differ from server-generated system events?
- Which findings should be handed to ingestion, datastream/observation semantics, and streaming topics?

#### Validation and Normalization Boundary

- What validation should occur in publishers/adapters before submission?
- What validation must occur inside Glaux Server regardless of adapter behavior?
- How should the server handle adapter-normalized data versus raw source data?
- Should Glaux Server support raw payload preservation?
- What normalization responsibilities should be duplicated defensively inside the server?
- How should validation results, warnings, rejected records, and quarantined records be attributed to source, adapter, and server processing?

#### Idempotency, Ordering, Replay, and DDIL Boundary

- What should publishers/adapters provide for idempotency and replay:
  - source identifiers,
  - message IDs,
  - sequence numbers,
  - timestamps,
  - offsets,
  - batch IDs,
  - checksums/hashes,
  - source epoch/version?
- What should Glaux Server track?
- How should out-of-order data, duplicate data, late data, replayed data, and source reconnect behavior be handled?
- How should DDIL operation affect adapter buffering, server persistence, replay, cache behavior, and synchronization?
- Which findings should be handed to `IDR-SRV-029`, `IDR-SRV-033`, and `IDR-SRV-041`?

#### Source Health and Operational Diagnostics

- How should publishers/adapters report health, availability, lag, backlog, last successful publish, source errors, credential failures, and transformation failures?
- Which source health information should be exposed through CSAPI resources, system events, operational diagnostics, or administrator-only views?
- How should source health differ from sensor/system status?
- How should source health information avoid leaking sensitive topology or operational details?
- Which findings should be handed to status/event, observability, and security topics?

#### Trust, Provenance, and Source Authority

- How should Glaux Server record source authority, adapter identity, publisher identity, transformation provenance, ingestion path, validation status, and confidence?
- How should Glaux Server distinguish source-authored records, adapter-derived records, server-normalized records, and inferred records?
- How should conflicting sources be handled?
- What provenance must be retained for conformance, audit, debugging, and operational trust?
- Which findings should be handed to `IDR-SRV-032`, `IDR-SRV-040`, and audit/security topics?

#### Security, Policy, and Releasability Boundary

- What security responsibilities belong to publisher/adapters versus Glaux Server?
- How should source credentials, adapter credentials, broker credentials, and submission credentials be handled?
- How should Glaux Server enforce policy and releasability on source-submitted metadata and dynamic data?
- How should adapter errors or validation details avoid leaking sensitive source topology, payload structure, or operational status?
- Which source metadata and adapter-provided values may be sensitive?
- Which findings should be handed to Category G and security-test topics?

#### Command/Control Boundary Implications

- Should publishers/adapters participate in command dispatch, command target translation, command-status reporting, or feasibility evaluation?
- Which command responsibilities must remain inside Glaux Server?
- How should command/control gateways differ from passive publishers?
- How should adapter-side command translation be validated and audited?
- How should source trust and command authority affect the adapter boundary?
- Which findings should be handed to `IDR-SRV-036`, `IDR-SRV-037`, and `IDR-SRV-038`?

#### Configuration and Deployment Boundary

- What configuration is required for publishers/adapters:
  - source endpoints,
  - credentials,
  - protocol settings,
  - mapping profiles,
  - schema profiles,
  - topic names,
  - retry policies,
  - buffering policies,
  - source trust settings,
  - command enablement?
- Which configuration belongs in Glaux Server versus adapter deployments?
- How should local development, CI, demos, and operational deployments handle adapters?
- Which findings should be handed to deployment, CI, configuration, and packaging topics?

#### Fixtures, Conformance, and Interoperability

- What publisher/adapter fixtures are needed:
  - standards-compliant publisher,
  - legacy feed adapter,
  - simulator publisher,
  - bad/invalid publisher,
  - delayed/replay publisher,
  - duplicate publisher,
  - DDIL reconnect publisher,
  - command gateway simulator?
- What conformance tests should simulate publisher/adapter behavior?
- What interoperability tests are needed with OSH, connected-systems-go, pygeoapi, SECD, CSAPI Explorer, OS4CSAPI clients, and external CSAPI clients?
- Which tests should target the Glaux Server boundary versus external adapter implementation?

#### Implementation and Interoperability Lessons

- What publisher/adapter boundary lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate boundary issues, source registration problems, ingestion assumptions, status/event inconsistencies, command gateway issues, or data mapping problems?
- What OS4CSAPI discussion lessons affect publisher/adapter boundary strategy?
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

- `IDR-SRV-001` through `IDR-SRV-030` research reports, once complete:
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

### Publisher, Adapter, Broker, and Integration Candidate Sources

Use current official documentation and primary-source technical material when executing the research. Candidate sources include, but are not limited to:

- MQTT v5 specification: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- Eclipse Mosquitto documentation: https://mosquitto.org/documentation/
- NATS documentation: https://docs.nats.io/
- Apache Kafka documentation: https://kafka.apache.org/documentation/
- CloudEvents specification: https://cloudevents.io/
- OpenTelemetry documentation for instrumentation and source diagnostics: https://opentelemetry.io/docs/
- Docker documentation: https://docs.docker.com/
- Kubernetes documentation, if sidecar/gateway patterns are evaluated: https://kubernetes.io/docs/home/
- OpenSensorHub / OSH sources and documentation, where available through project research
- OSHConnect / OSH-connected publisher examples, where available through project research

These are candidates for evaluation, not assumed selections.

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, source integration guidance, dynamic-data guidance, tasking guidance, DDIL guidance, or standards-package annexes relevant to publishers, adapters, source systems, gateways, and NATO JISR sensor integration

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
- Category D findings from `IDR-SRV-021` through `IDR-SRV-024`
- Category E findings from `IDR-SRV-025` through `IDR-SRV-030`
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
- Glaux Publisher repository, if available or created: https://github.com/DGIWG-P507/glaux-publisher
- Glaux Simulator repository, if available or created: https://github.com/DGIWG-P507/glaux-simulator
- OS4CSAPI testing research-plan corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

---

## 5. Research Methodology

### Phase 1: Source Collection and Boundary Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for publisher/adapter integration boundary research.

**Tasks:**

1. Gather prior IDR reports, standards sources, implementation studies, broker/integration documentation, OSH/OS4CSAPI sources, and project architecture sources.
2. Extract boundary requirements from each prior topic and classify them by source class, resource family, data type, validation responsibility, trust implication, and downstream dependency.
3. Define inventory fields:
   - source/publisher class,
   - integration pattern,
   - data submitted,
   - server responsibility,
   - adapter responsibility,
   - trust/provenance requirement,
   - validation/normalization requirement,
   - idempotency/replay requirement,
   - security/policy implication,
   - downstream handoff.
4. Define evaluation criteria:
   - standards alignment,
   - source flexibility,
   - validation robustness,
   - trust clarity,
   - provenance strength,
   - DDIL/replay support,
   - command safety,
   - deployment simplicity,
   - Rust implementation fit,
   - fixture/testability,
   - interoperability.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation evidence, and technology documentation.

**Expected Output:** Publisher/adapter boundary extraction framework and evaluation rubric.

### Phase 2: Source Class and Responsibility Allocation Inventory (3-4 hours)

**Objective:** Determine source/publisher classes and allocate responsibilities between external components and Glaux Server.

**Tasks:**

1. Inventory likely source classes and integration patterns.
2. Map source classes to metadata, observation, status, event, command, and source-health submission responsibilities.
3. Identify which responsibilities may be delegated, shared, or retained by Glaux Server.
4. Identify server-authoritative functions that cannot be delegated without breaking validation, policy, conformance, audit, or interoperability.
5. Build a source-class and responsibility-allocation matrix.

**Expected Output:** Source class taxonomy and responsibility allocation matrix.

### Phase 3: Integration Contract and Data Boundary Analysis (3-4 hours)

**Objective:** Define conceptual integration contracts and payload expectations.

**Tasks:**

1. Analyze metadata/description submission boundaries.
2. Analyze observation/status/event submission boundaries.
3. Analyze validation, normalization, source fidelity, raw-payload preservation, and provenance boundaries.
4. Analyze idempotency, ordering, replay, offsets, batch identifiers, and DDIL reconnect boundaries.
5. Analyze source health and operational diagnostics boundaries.
6. Identify unresolved questions requiring prototype validation or fixture simulation.

**Expected Output:** Integration contract and data-boundary strategy matrix.

### Phase 4: Trust, Security, Command, and DDIL Boundary Analysis (2.5-3.5 hours)

**Objective:** Analyze high-risk boundary concerns.

**Tasks:**

1. Analyze source trust, provenance, source authority, and adapter identity implications.
2. Analyze security, credential, policy, releasability, and audit implications.
3. Analyze command/control gateway implications and boundaries between passive publishers and command-capable adapters.
4. Analyze DDIL buffering, replay, synchronization, and conflict implications.
5. Map findings to source registration/trust, ingestion, command, DDIL, and security topics.

**Expected Output:** Trust/security/command/DDIL boundary implication matrix.

### Phase 5: Fixtures, Conformance, Deployment, and Interoperability Implication Analysis (2.5-3 hours)

**Objective:** Prepare boundary findings for implementation and verification work.

**Tasks:**

1. Identify publisher/adapter fixture types and simulated source behaviors.
2. Identify conformance tests for server-side boundary behavior.
3. Identify interoperability tests for OSH, connected-systems-go, pygeoapi, SECD, CSAPI Explorer, OS4CSAPI clients, and external CSAPI clients.
4. Identify deployment, configuration, sidecar/gateway, broker, and local development implications.
5. Identify performance/load/stress test needs for publisher submission and ingestion boundary.
6. Map findings to Category F, G, H, and I topics.

**Expected Output:** Boundary verification and deployment implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert publisher/adapter boundary research into a decision-usable baseline.

**Tasks:**

1. Consolidate source class taxonomy, responsibility allocation, integration contract guidance, trust/security findings, and downstream implications.
2. Produce recommended publisher/adapter integration boundary strategy with rationale and unresolved questions.
3. Identify sequencing for source registration/trust, ingestion, streaming, command, DDIL, security, deployment, fixture, conformance, and performance topics.
4. Produce downstream handoff matrix.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Publisher, adapter, source, gateway, simulator, broker, and federated-source classes are identified with source anchors.
- [ ] Server responsibilities, adapter responsibilities, shared responsibilities, and non-delegable server responsibilities are distinguished.
- [ ] Metadata, observation, status, event, source health, command, validation, normalization, idempotency, replay, and provenance boundary implications are documented.
- [ ] Trust, security, policy, releasability, command/control, and DDIL boundary implications are documented.
- [ ] Integration patterns are evaluated against explicit criteria.
- [ ] Fixture, conformance, deployment, performance, and interoperability implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Publisher and Adapter Integration Boundary Model Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-031-publisher-and-adapter-integration-boundary-model-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Boundary requirement extraction methodology
5. Publisher/adapter/source class taxonomy
6. Integration pattern evaluation
7. Responsibility allocation findings
8. Metadata and description submission boundary findings
9. Observation, status, event, and source-health boundary findings
10. Validation, normalization, provenance, idempotency, ordering, and replay findings
11. Trust, source authority, and external source identity implications
12. Security, policy, releasability, credential, and audit implications
13. Command/control gateway and command-capable adapter implications
14. DDIL, buffering, replay, and synchronization implications
15. Fixture, conformance, deployment, performance, and interoperability test implications
16. Downstream topic handoff matrix
17. Recommendations
18. Risks, constraints, and open questions
19. Validation against this plan's success criteria
20. References

The boundary matrix should include, at minimum:

- Source/publisher class
- Integration pattern
- Data submitted
- Related CSAPI / Glaux resource family
- Server responsibility
- Adapter responsibility
- Shared responsibility
- Trust/provenance requirement
- Validation/normalization requirement
- Idempotency/replay requirement
- Security/policy implication
- Test/conformance implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-030` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, relevant OGC schemas/OpenAPI artifacts, broker/integration sources, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Implementation-study and interoperability findings must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-032: External Source Registration and Trust Model`
- `IDR-SRV-033: Dynamic Data Ingestion and Normalization Pipeline`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-035: Streaming and Event Publication Strategy`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-037: Feasibility and Command Validation Strategy`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints`
- `IDR-SRV-041: DDIL Behavior, Caching, and Synchronization Semantics`
- `IDR-SRV-045: Service Packaging, Containerization, and Deployment Topology`
- `IDR-SRV-046: Local Development and CI Environment Strategy`
- `IDR-SRV-049: Observability, Logging, Metrics, and Operational Diagnostics`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
- `IDR-SRV-054: Performance, Load, Stress, and Streaming Test Strategy`
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

- This topic defines the integration boundary, not every adapter implementation.
- Adapters may support normalization, but Glaux Server must retain enough validation, provenance, policy, and canonical-state authority to remain standards-aligned.
- Implementation-study findings are useful but must not override standards-derived server responsibilities.
- Open question: Which publisher/adapter patterns are required for first implementation versus full-scope readiness?
- Open question: Should Glaux Server expose a private/internal ingestion API distinct from public CSAPI operations?
- Open question: How much raw payload preservation should the server support?
- Open question: Which source health records should be exposed publicly, administrator-only, or not at all?
- Open question: How should command-capable adapters be trusted and audited?
- Risk: Overdelegating validation to adapters could weaken server conformance and interoperability.
- Risk: Underdefining source identity and provenance could make audit and trust decisions unreliable.
- Risk: Treating all publishers as trusted could create security, policy, and data-quality failures.
- Risk: Defining only one integration pattern could block legacy, simulator, brokered, and DDIL use cases.

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
- MQTT v5 specification: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- Eclipse Mosquitto documentation: https://mosquitto.org/documentation/
- NATS documentation: https://docs.nats.io/
- Apache Kafka documentation: https://kafka.apache.org/documentation/
- CloudEvents specification: https://cloudevents.io/
- OpenTelemetry documentation: https://opentelemetry.io/docs/
- Docker documentation: https://docs.docker.com/
- Kubernetes documentation: https://kubernetes.io/docs/home/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

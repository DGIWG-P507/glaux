# Section 033: Dynamic Data Ingestion and Normalization Pipeline - Research Plan

**Status:** Planned  
**Last Updated:** June 11, 2026  
**Estimated Research Time:** 14-18 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-033-dynamic-data-ingestion-and-normalization-pipeline-report.md`

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

This topic research must define the Glaux Server planning baseline for the **dynamic data ingestion and normalization pipeline** across observations, status updates, dynamic properties, system events, source-health reports, raw publisher payloads, adapter outputs, brokered messages, batch/file imports, replayed data, DDIL reconnect data, SensorML/SWE Common-linked datastream definitions, validation artifacts, provenance records, latest-value updates, time-series storage, geospatial/semantic enrichment, and downstream streaming/event publication.

The research must answer:

- What dynamic data ingestion responsibilities must Glaux Server support for CSAPI Part 2 and AEP-4789 server behavior?
- What data enters the server through publishers, adapters, simulators, gateways, federated sources, brokered topics, direct API submission, and batch import?
- What ingestion stages are required before data becomes authoritative observations, status updates, dynamic property values, system events, source-health records, command-status records, or derived latest-value views?
- How should Glaux Server validate, normalize, enrich, transform, quarantine, reject, persist, index, and publish dynamic data?
- How should ingestion interact with source registration/trust, datastream definitions, SensorML/SWE Common structures, semantic bindings, units, temporal semantics, geospatial semantics, transactions, idempotency, replay, and DDIL synchronization?
- What raw payloads, normalized records, validation artifacts, source offsets, provenance records, latest values, and event-publication records must be retained?
- What downstream implications follow for datastream/observation semantics, streaming/event publication, command lifecycle, DDIL behavior, security/policy, observability, fixtures, performance testing, conformance testing, and interoperability testing?

The output must be a dynamic data ingestion and normalization pipeline strategy baseline with source anchors, ingestion-stage model, data-category mapping, validation and normalization requirements, raw/normalized retention guidance, provenance and idempotency requirements, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows:

- `IDR-SRV-031: Publisher and Adapter Integration Boundary Model`
- `IDR-SRV-032: External Source Registration and Trust Model`

`IDR-SRV-031` defines the server boundary with publishers/adapters. `IDR-SRV-032` defines how sources are registered and trusted. This topic defines how dynamic data arriving through that boundary is processed into standards-aligned Glaux Server state. It must precede detailed datastream/observation/status semantics, streaming/event publication, command lifecycle integration, DDIL behavior, performance testing, fixture planning, and interoperability validation.

### Critical Constraints

- Treat CSAPI Part 2, CSAPI Part 1, SensorML 3.0, SWE Common 3.0, schema/encoding validation, semantic binding, source trust, persistence, transaction/idempotency, AEP-4789 server responsibilities, and prior IDR findings as controlling.
- Do not design the final code implementation here. Define ingestion stages, responsibilities, decision points, required artifacts, and downstream implications.
- Do not treat all dynamic data as observations. Distinguish observations, status updates, dynamic properties, system events, source-health records, command-status records, ingestion telemetry, validation artifacts, and audit records.
- Do not assume every source is trusted, standards-compliant, online, ordered, non-duplicative, or synchronized.
- Do not finalize datastream/observation API semantics here. Identify ingestion dependencies and hand API behavior to `IDR-SRV-034`.
- Do not finalize streaming/event publication here. Identify publication and replay needs and hand detailed streaming design to `IDR-SRV-035`.
- Do not finalize command lifecycle here. Identify command-status ingestion requirements and hand detailed command behavior to `IDR-SRV-036` through `IDR-SRV-038`.
- Do not finalize policy or authorization here. Identify policy/releasability implications and hand them to Category G.
- Keep the research bounded to Glaux Server behavior and server-side ingestion contracts.

---

## 2. Research Questions

### Core Questions

1. What dynamic data categories must Glaux Server ingest, validate, normalize, persist, index, and expose?
2. What ingestion pipeline stages are required from source submission to authoritative server state and optional publication?
3. How should source trust, provenance, idempotency, replay, temporal semantics, validation, and policy affect ingestion decisions?
4. What raw payloads, normalized records, validation artifacts, source offsets, latest values, and events must be retained?
5. What downstream implications follow for datastream/observation semantics, streaming, commands, DDIL, security, observability, fixtures, performance, conformance, and interoperability?

### Detailed Questions

#### Standards and Ingestion Baseline

- What ingestion behavior is required or implied by CSAPI Part 2 for observations, datastreams, status, events, control streams, commands, and feasibility resources?
- What CSAPI Part 1 resource definitions must exist or be referenced before dynamic data can be accepted?
- What SensorML and SWE Common structures define or constrain ingested observation results, status values, command-status records, and event payloads?
- What schema, encoding, semantic, unit, and validation requirements from Category D apply at ingestion time?
- What AEP-4789 server responsibilities imply ingestion, normalization, validation, provenance, or source-trust behavior?
- Which ingestion behaviors are standards-driven versus implementation-defined?

#### Dynamic Data Category Inventory

- What data categories can enter the ingestion pipeline:
  - observations,
  - observation result values,
  - latest-value updates,
  - status updates,
  - dynamic property values,
  - system events,
  - source health reports,
  - ingestion telemetry,
  - command status records,
  - command results,
  - feasibility results,
  - source metadata updates,
  - raw adapter payloads,
  - batch/file records,
  - replayed records,
  - duplicate records,
  - delayed records,
  - rejected records?
- Which categories are authoritative records, staging records, raw payloads, normalized records, validation artifacts, current-state views, or derived events?
- Which categories should be accepted only from specific source classes or trust states?

#### Ingestion Source and Submission Pattern Analysis

- What ingestion patterns must be considered:
  - direct REST submission,
  - CSAPI-native submission,
  - private/internal ingestion API,
  - brokered publication,
  - MQTT topic ingestion,
  - NATS/Kafka-style stream ingestion,
  - file/batch import,
  - simulator publication,
  - adapter sidecar publication,
  - gateway-mediated publication,
  - federated pull from external CSAPI endpoints?
- Which patterns are first-implementation candidates versus full-scope candidates?
- What ingestion metadata must be attached for each pattern?
- How do source credentials, source identity, source offsets, topic names, batch IDs, and adapter identifiers enter the pipeline?

#### Ingestion Stage Model

- What conceptual stages should the pipeline include:
  - receipt,
  - authentication/source identification,
  - source registration lookup,
  - coarse payload validation,
  - profile/format/encoding identification,
  - schema validation,
  - source trust evaluation,
  - deduplication/idempotency check,
  - temporal normalization,
  - datastream/resource resolution,
  - SWE Common result mapping,
  - unit/semantic validation,
  - geospatial enrichment,
  - policy/releasability check,
  - normalization,
  - persistence,
  - latest-value update,
  - event generation,
  - publication/outbox,
  - audit/provenance recording,
  - error/quarantine handling?
- Which stages are mandatory, optional, configurable, source-specific, or profile-specific?
- Which stages must be transactional?
- Which stages can be asynchronous?

#### Validation and Normalization Strategy

- What validation should happen before persistence?
- What validation can happen asynchronously after staging?
- Which validation failures should reject, quarantine, warn, or partially accept records?
- How should normalization handle field names, data types, units, timestamps, geometry, semantic identifiers, nil values, quality, uncertainty, and encoding conversion?
- How should invalid or unknown datastream references be handled?
- How should SensorML/SWE Common mismatches be handled?
- How should the server preserve evidence of validation and normalization decisions?

#### Datastream and Resource Resolution

- How should ingested data resolve to systems, deployments, procedures, sampling features, properties, datastreams, and control streams?
- What should happen if a referenced resource is missing, inactive, retired, unauthorized, or not trusted for the source?
- Should ingestion be allowed to create resources automatically?
- Which resource creation behavior requires approval, trust, or configuration?
- How should source-specific identifiers map to canonical Glaux Server identifiers?
- Which findings should be handed to `IDR-SRV-034` and relationship/lifecycle topics?

#### Temporal, Geospatial, and Semantic Normalization

- How should phenomenon time, result time, ingestion time, event time, source time, and publication time be assigned and normalized?
- How should out-of-order, delayed, duplicated, future, and clock-skewed records be handled?
- How should observation location, source location, feature-of-interest geometry, platform location, and system location be attached or inferred?
- How should units, observed properties, controlled properties, semantic definitions, and vocabulary mappings be resolved?
- Which normalization should be mandatory versus best-effort?
- Which findings should be handed to geospatial, time-series, semantic, and status/observation topics?

#### Raw Payload Preservation and Provenance

- Should raw source payloads be preserved?
- Which payloads should be stored in full, summarized, hashed, sampled, or discarded?
- How should raw payload retention differ for accepted, rejected, quarantined, and test records?
- How should provenance record source, adapter, protocol, version, mapping profile, validation state, normalization state, and operator intervention?
- How should raw payload preservation interact with policy, storage cost, security, and audit?

#### Idempotency, Ordering, Replay, and Error Handling

- What idempotency keys, source offsets, message IDs, sequence numbers, batch IDs, hashes, and replay markers should be used?
- How should duplicate records be detected and represented?
- How should replay and backfill be supported?
- How should ingestion handle partial batch failure?
- Which failures are retryable?
- Which failures are permanent?
- Which failures should produce problem details, source-health updates, validation artifacts, system events, or audit records?
- Which findings should be handed to `IDR-SRV-029`, `IDR-SRV-035`, and `IDR-SRV-041`?

#### Persistence and Latest-Value Interaction

- How should ingestion persist time-series observations, status updates, events, command-status records, validation artifacts, and raw payload references?
- How should latest-value/materialized current-state views be updated?
- Which updates must be transactional with source offsets, validation artifacts, and events?
- How should ingestion interact with retention, partitions, indexing, and archival?
- How should high-rate ingestion avoid overloading relational, geospatial, or document stores?
- Which findings should be handed to time-series storage, transaction, and performance topics?

#### Streaming and Event Publication Interaction

- Which ingestion outcomes should generate durable system events?
- Which ingestion outcomes should publish live stream messages?
- How should ingestion support subscription backfill, replay, reconnect, and event ordering?
- Should publication occur before or after durable persistence?
- How should event publication avoid duplicates during retries?
- Which findings should be handed to `IDR-SRV-035`?

#### Command, Control Stream, and Feasibility Interaction

- What command-related data can enter through ingestion:
  - command status,
  - command result,
  - target acknowledgement,
  - execution progress,
  - failure report,
  - cancellation report,
  - feasibility response?
- How should command-related ingestion differ from passive observation ingestion?
- How should command-capable sources and gateways be validated before accepting command-status updates?
- How should command-status ingestion be audited and correlated to command records?
- Which findings should be handed to `IDR-SRV-036`, `IDR-SRV-037`, and `IDR-SRV-038`?

#### DDIL, Federation, and Synchronization

- How should ingestion behave in disconnected, degraded, intermittent, or limited-bandwidth conditions?
- How should buffered adapter records be accepted after reconnect?
- How should source offsets, stale trust, delayed records, conflicts, and replay windows be handled?
- How should federated source records be distinguished from local authoritative records?
- Which findings should be handed to `IDR-SRV-041` and federation/interoperability topics?

#### Security, Policy, and Releasability

- How should policy and releasability affect acceptance, normalization, storage, publication, and query visibility of ingested records?
- Which ingested fields may reveal sensitive source topology, sensor location, command affordances, observed phenomena, or operational patterns?
- How should rejected/quarantined records be protected?
- How should raw payloads and validation errors avoid leaking sensitive data?
- Which findings should be handed to Category G and security-test topics?

#### Observability and Operational Diagnostics

- What ingestion metrics and diagnostics are needed:
  - records received,
  - records accepted,
  - records rejected,
  - records quarantined,
  - validation failure rates,
  - source lag,
  - replay lag,
  - batch duration,
  - normalization errors,
  - latest-value update lag,
  - publication lag?
- Which metrics are source-specific, adapter-specific, resource-specific, or system-wide?
- Which diagnostics are safe to expose publicly, administrator-only, or internal-only?
- Which findings should be handed to `IDR-SRV-049`?

#### Fixtures, Conformance, Performance, and Interoperability

- What ingestion fixtures are needed:
  - valid observations,
  - invalid observations,
  - high-rate data,
  - delayed data,
  - duplicate data,
  - out-of-order data,
  - replay batches,
  - status updates,
  - system events,
  - command-status updates,
  - source health records,
  - raw legacy payloads,
  - policy-filtered records,
  - DDIL reconnect records?
- What conformance tests should verify ingestion behavior?
- What performance/load/stress tests are needed for ingestion throughput, normalization latency, validation cost, batch behavior, and publication lag?
- What interoperability tests should involve OSH, connected-systems-go, pygeoapi, SECD, CSAPI Explorer, OS4CSAPI clients, and external CSAPI clients?

#### Implementation and Interoperability Lessons

- What ingestion and normalization lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate ingestion assumptions, payload mismatch, invalid observations, status-event differences, duplicate handling, or source mapping problems?
- What OS4CSAPI discussion lessons affect ingestion pipeline strategy?
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

- `IDR-SRV-001` through `IDR-SRV-032` research reports, once complete:
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

### Ingestion, Messaging, and Integration Candidate Sources

Use current official documentation and primary-source technical material when executing the research. Candidate sources include, but are not limited to:

- MQTT v5 specification: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- Eclipse Mosquitto documentation: https://mosquitto.org/documentation/
- NATS documentation: https://docs.nats.io/
- Apache Kafka documentation: https://kafka.apache.org/documentation/
- CloudEvents specification: https://cloudevents.io/
- OpenTelemetry documentation: https://opentelemetry.io/docs/
- PostgreSQL documentation: https://www.postgresql.org/docs/
- TimescaleDB documentation: https://docs.timescale.com/
- Rust SQLx documentation: https://docs.rs/sqlx/
- Rust serde documentation: https://serde.rs/
- Rust tracing documentation: https://docs.rs/tracing/

These are candidates for evaluation, not assumed selections.

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, dynamic-data guidance, source integration guidance, validation guidance, DDIL guidance, or standards-package annexes relevant to observations, status updates, events, publishers, adapters, and NATO JISR sensor integration

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
- Publisher/adapter and source-trust findings from `IDR-SRV-031` and `IDR-SRV-032`
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

### Phase 1: Source Collection and Ingestion Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for dynamic data ingestion and normalization research.

**Tasks:**

1. Gather prior IDR reports, standards sources, implementation studies, messaging/integration documentation, source-boundary findings, source-trust findings, and project architecture sources.
2. Extract ingestion requirements from prior topics and classify them by data category, source class, ingestion pattern, validation need, normalization need, persistence need, and downstream dependency.
3. Define inventory fields:
   - data category,
   - source class,
   - submission pattern,
   - required pipeline stage,
   - validation requirement,
   - normalization requirement,
   - persistence target,
   - idempotency/replay requirement,
   - provenance requirement,
   - security/policy implication,
   - downstream handoff.
4. Define evaluation criteria:
   - standards alignment,
   - validation robustness,
   - source flexibility,
   - provenance strength,
   - idempotency/replay support,
   - DDIL suitability,
   - throughput,
   - storage efficiency,
   - command/status safety,
   - fixture/testability,
   - interoperability.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation evidence, and technology documentation.

**Expected Output:** Ingestion requirement extraction framework and evaluation rubric.

### Phase 2: Dynamic Data Category and Source Pattern Inventory (3-4 hours)

**Objective:** Determine what data enters Glaux Server and through which patterns.

**Tasks:**

1. Inventory dynamic data categories from CSAPI Part 2, prior IDR topics, implementation studies, and source-boundary research.
2. Inventory source classes and submission patterns.
3. Classify records by authoritative/staged/raw/normalized/derived/current-state/test status.
4. Identify required metadata for source identity, timestamps, offsets, sequence numbers, datastream references, units, semantics, geometry, and validation state.
5. Build a dynamic data category and source-pattern matrix.

**Expected Output:** Dynamic data and source-pattern inventory matrix.

### Phase 3: Pipeline Stage, Validation, Normalization, and Persistence Analysis (3-4 hours)

**Objective:** Define the conceptual ingestion pipeline.

**Tasks:**

1. Analyze receipt, source identification, trust lookup, payload validation, format detection, schema validation, resource resolution, idempotency check, temporal normalization, semantic/unit validation, geospatial enrichment, policy check, persistence, latest-value update, event generation, and audit/provenance stages.
2. Identify mandatory, optional, configurable, source-specific, and asynchronous stages.
3. Identify transaction boundaries and persistence artifacts.
4. Identify raw payload preservation, rejected/quarantined record handling, and validation artifact storage.
5. Identify unresolved questions requiring prototype validation or fixture simulation.

**Expected Output:** Ingestion stage and normalization strategy matrix.

### Phase 4: Replay, DDIL, Streaming, Command, and Policy Analysis (3-4 hours)

**Objective:** Analyze high-risk ingestion interactions.

**Tasks:**

1. Analyze duplicate detection, replay, backfill, batch failure, out-of-order data, late data, and source reconnect behavior.
2. Analyze DDIL ingestion behavior and synchronization implications.
3. Analyze streaming/event publication coupling and outbox/backfill needs.
4. Analyze command-status and feasibility-result ingestion boundaries.
5. Analyze policy/releasability and security handling for raw, normalized, rejected, and published records.
6. Map findings to datastream/observation, streaming, command, DDIL, and security topics.

**Expected Output:** High-risk ingestion interaction matrix.

### Phase 5: Observability, Fixtures, Performance, Conformance, and Interoperability Implication Analysis (2.5-3 hours)

**Objective:** Prepare ingestion findings for downstream implementation and verification work.

**Tasks:**

1. Identify ingestion metrics, health signals, diagnostics, and operational dashboards.
2. Identify ingestion fixtures and simulated source behaviors.
3. Identify conformance tests for server-side ingestion behavior.
4. Identify performance/load/stress tests for ingestion throughput, validation cost, batch behavior, latest-value updates, and publication lag.
5. Identify interoperability tests with OSH, connected-systems-go, pygeoapi, SECD, CSAPI Explorer, OS4CSAPI clients, and external CSAPI clients.
6. Map findings to Category F, G, H, and I topics.

**Expected Output:** Ingestion verification and operations implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert ingestion research into a decision-usable baseline.

**Tasks:**

1. Consolidate data-category inventory, source-pattern inventory, pipeline-stage guidance, validation/normalization guidance, persistence guidance, and downstream implications.
2. Produce recommended dynamic data ingestion and normalization pipeline strategy with rationale and unresolved questions.
3. Identify sequencing for datastream/observation semantics, streaming, command, DDIL, security, deployment, fixture, conformance, and performance topics.
4. Produce downstream handoff matrix.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Dynamic data categories, source classes, and submission patterns are identified with source anchors.
- [ ] Observations, status updates, dynamic properties, system events, source health, command-status records, raw payloads, normalized records, rejected records, and latest-value views are distinguished.
- [ ] Ingestion pipeline stages, validation responsibilities, normalization responsibilities, persistence artifacts, provenance artifacts, and publication triggers are documented.
- [ ] Source trust, idempotency, replay, temporal normalization, geospatial enrichment, semantic/unit validation, and policy/releasability implications are documented.
- [ ] DDIL, streaming, command, security, observability, fixture, conformance, performance, and interoperability implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Dynamic Data Ingestion and Normalization Pipeline Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-033-dynamic-data-ingestion-and-normalization-pipeline-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Ingestion requirement extraction methodology
5. Dynamic data category inventory
6. Source class and submission pattern findings
7. Ingestion stage model
8. Validation and normalization findings
9. Resource/datastream resolution findings
10. Temporal, geospatial, semantic, unit, and encoding normalization findings
11. Raw payload, validation artifact, provenance, and persistence findings
12. Idempotency, duplicate detection, replay, and DDIL findings
13. Latest-value, event generation, and streaming/publication findings
14. Command-status and feasibility-result ingestion findings
15. Security, policy, releasability, and audit implications
16. Observability, fixture, conformance, performance, and interoperability test implications
17. Downstream topic handoff matrix
18. Recommendations
19. Risks, constraints, and open questions
20. Validation against this plan's success criteria
21. References

The ingestion pipeline matrix should include, at minimum:

- Data category
- Source class
- Submission pattern
- Required pipeline stage
- Validation requirement
- Normalization requirement
- Persistence target
- Idempotency/replay requirement
- Provenance requirement
- Latest-value/publication implication
- Security/policy implication
- Test/conformance implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-032` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, relevant OGC schemas/OpenAPI artifacts, ingestion/messaging sources, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Implementation-study and interoperability findings must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-035: Streaming and Event Publication Strategy`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-037: Feasibility and Command Validation Strategy`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- `IDR-SRV-041: DDIL Behavior, Caching, and Synchronization Semantics`
- `IDR-SRV-045: Service Packaging, Containerization, and Deployment Topology`
- `IDR-SRV-046: Local Development and CI Environment Strategy`
- `IDR-SRV-049: Observability, Logging, Metrics, and Operational Diagnostics`
- `IDR-SRV-050: Conformance Harness Strategy`
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

- This topic defines the ingestion and normalization pipeline strategy, not final implementation code.
- Glaux Server must retain enough authority over validation, normalization, policy, provenance, and persistence to remain standards-aligned even when adapters perform preliminary mapping.
- Implementation-study findings are useful but must not override standards-derived server responsibilities.
- Open question: Which ingestion patterns are required for first implementation versus full-scope readiness?
- Open question: Should raw payload preservation be mandatory, configurable, sampled, or limited to rejected/quarantined records?
- Open question: Which validation failures should reject versus quarantine versus accept with warnings?
- Open question: How should ingestion handle records that arrive before their datastream or resource definitions?
- Open question: Which ingestion metrics should become operational health indicators?
- Risk: Treating all dynamic data as observations could blur status, event, command-status, and source-health semantics.
- Risk: Overdelegating validation to adapters could weaken conformance and interoperability.
- Risk: Underdefining idempotency and replay could create duplicate records, inconsistent latest values, and unreliable streaming.
- Risk: Raw payload retention could create security, storage, and releasability risks if not governed carefully.

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
- PostgreSQL documentation: https://www.postgresql.org/docs/
- TimescaleDB documentation: https://docs.timescale.com/
- Rust SQLx documentation: https://docs.rs/sqlx/
- Rust serde documentation: https://serde.rs/
- Rust tracing documentation: https://docs.rs/tracing/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

# Section 034: Datastream, Observation, and Status Update Semantics - Research Plan

**Status:** Planned  
**Last Updated:** June 11, 2026  
**Estimated Research Time:** 14-18 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-034-datastream-observation-and-status-update-semantics-report.md`

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

This topic research must define the Glaux Server planning baseline for **datastream, observation, and status update semantics** across OGC API - Connected Systems Part 2 dynamic data, datastream definitions, observation resources, observation result structures, status and dynamic property updates, latest values, historical values, SensorML/SWE Common data components, units, observed properties, temporal semantics, geospatial semantics, ingestion normalization, source trust, validation, persistence, streaming, and client-facing query behavior.

The research must answer:

- What are the precise semantic differences among datastreams, observations, observation results, status updates, dynamic properties, system events, latest values, and source-health records in Glaux Server?
- How should Glaux Server model datastream definitions, observed properties, result structures, units, sampling features, systems, deployments, procedures, and features of interest for standards-aligned observation access?
- What is the relationship between datastream metadata, SensorML outputs, SWE Common result definitions, semantic bindings, time-series storage, and actual observation values?
- How should status updates and dynamic properties differ from observations, system events, metadata updates, health reports, and command-status records?
- What API behavior, persistence behavior, validation behavior, query behavior, ingestion behavior, and streaming behavior should Glaux Server support for datastreams, observations, and status updates?
- How should temporal, spatial, semantic, unit, quality, nil-value, uncertainty, provenance, source-trust, policy/releasability, idempotency, replay, and DDIL considerations affect datastream and observation semantics?
- What downstream implications follow for streaming/event publication, command lifecycle, feasibility validation, DDIL behavior, security, observability, conformance testing, fixture strategy, performance testing, and external-client interoperability?

The output must be a datastream, observation, and status update semantics baseline with source anchors, semantic definitions, resource-family mappings, API behavior implications, persistence and query implications, validation and ingestion implications, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows:

- `IDR-SRV-033: Dynamic Data Ingestion and Normalization Pipeline`

`IDR-SRV-033` defines the pipeline that receives, validates, normalizes, and persists dynamic data. This topic defines what the ingested records mean once they become standards-aligned server resources or values. It must precede streaming/event publication, command lifecycle modeling, feasibility validation, command safety/audit, DDIL behavior, conformance harness design, fixture generation, performance testing, and external-client interoperability testing.

### Critical Constraints

- Treat OGC API - Connected Systems Part 2, OGC API - Connected Systems Part 1, SensorML 3.0, SWE Common 3.0, schema/encoding validation, semantic binding, time-series storage, ingestion pipeline, AEP-4789 server responsibilities, and prior IDR findings as controlling.
- Do not treat datastreams as merely database tables or ingestion channels. Datastreams are standards-facing resources with semantic, structural, temporal, and relationship obligations.
- Do not treat all dynamic records as observations. Distinguish observations, status updates, dynamic properties, system events, source-health records, command-status records, and audit records.
- Do not finalize streaming/event publication mechanics here. Identify publication semantics and hand detailed streaming design to `IDR-SRV-035`.
- Do not finalize command lifecycle semantics here. Identify command-status and control-stream interactions and hand command behavior to `IDR-SRV-036` through `IDR-SRV-038`.
- Do not finalize policy or authorization here. Identify policy/releasability and sensitive-data implications and hand them to Category G.
- Do not design the final database schema here. Identify persistence and query semantics and rely on Category E findings for storage strategy.
- Keep the research bounded to Glaux Server behavior and server-side API/resource semantics.

---

## 2. Research Questions

### Core Questions

1. What are the normative and implementation semantics of datastreams, observations, observation results, status updates, dynamic properties, system events, and latest values?
2. How should Glaux Server model and expose datastream definitions, observation results, status values, temporal fields, units, observed properties, sampling features, and SensorML/SWE Common structures?
3. How should ingestion records become standards-aligned observations, status updates, or event records?
4. How should query, filtering, sorting, pagination, latest-value retrieval, validation, provenance, and policy behavior work for dynamic data resources?
5. What downstream implications follow for streaming, commands, DDIL, security, observability, conformance, fixtures, performance, and interoperability?

### Detailed Questions

#### Standards and Dynamic-Data Semantics Baseline

- What datastream, observation, status, event, control stream, command, and feasibility semantics are defined by CSAPI Part 2?
- What CSAPI Part 1 resources are required to interpret Part 2 dynamic data?
- How do SensorML inputs, outputs, parameters, capabilities, and characteristics relate to datastream and status semantics?
- How do SWE Common data components, result structures, units, nil values, constraints, encodings, and quality metadata relate to datastreams and observations?
- What AEP-4789 Volume II expectations shape dynamic-data semantics for NATO JISR sensor interoperability?
- Which dynamic-data semantics are explicitly normative versus implementation-defined?

#### Datastream Semantic Model

- What is a datastream in Glaux Server?
- How should a datastream relate to:
  - a system,
  - a procedure,
  - a deployment,
  - a sampling feature,
  - a property,
  - an observed property,
  - a feature of interest,
  - a SensorML output,
  - a SWE Common data component,
  - a result structure,
  - an encoding,
  - a unit,
  - a spatial/temporal extent,
  - a source/publisher,
  - a security/policy scope?
- Which datastream metadata is required, optional, derived, source-provided, or server-assigned?
- How should datastream lifecycle changes affect ingestion, observation access, latest values, and streaming?
- How should multi-output systems and multi-component result structures be represented?

#### Observation Semantic Model

- What is an observation in Glaux Server?
- What is the difference between an observation resource, an observation result, a datastream sample, a status update, and an event?
- How should observation values relate to SWE Common result definitions?
- How should phenomenon time, result time, ingestion time, valid time, and publication time be represented?
- How should observations link to systems, deployments, datastreams, sampling features, features of interest, observed properties, units, source identities, and provenance?
- How should nil, missing, invalid, uncertain, quality-flagged, or out-of-range values be represented?
- Which observation fields must be queryable, indexed, or exposed?

#### Status Update and Dynamic Property Semantics

- What is a status update in Glaux Server?
- How should status updates differ from observations, system events, source health, command status, and metadata updates?
- What is a dynamic property?
- How should status values relate to system properties, SensorML capabilities/characteristics, SWE Common structures, and latest-value views?
- Which status values are operationally relevant versus source-health or administrative diagnostics?
- Should status updates be stored as time-series records, event records, current state, or all three?
- How should stale, degraded, unknown, unavailable, and simulated status be represented?

#### System Events and Event-Like Dynamic Records

- What dynamic records should be treated as system events rather than observations or status values?
- What event types are relevant to dynamic data semantics:
  - resource lifecycle events,
  - source health events,
  - ingestion validation events,
  - status change events,
  - command lifecycle events,
  - availability events,
  - synchronization events,
  - policy/audit events?
- Which events are client-visible CSAPI resources?
- Which events are operational diagnostics or audit-only?
- How should system events interact with streaming/event publication in `IDR-SRV-035`?

#### Latest Values and Historical Values

- What does “latest value” mean for observations, status values, dynamic properties, and source health?
- Should latest value be computed or materialized?
- How should latest value be affected by late-arriving data, out-of-order data, replay, source trust, validation state, and policy filtering?
- How should clients request latest values versus historical values?
- What default behavior should apply when no time range is specified?
- How should freshness/staleness indicators be represented?

#### Temporal Semantics

- Which time fields are required or optional for datastreams, observations, status updates, and events?
- How should phenomenon time, result time, ingestion time, publication time, valid time, transaction time, and source time differ?
- How should intervals, instants, future times, clock skew, delayed data, late data, and out-of-order records be handled?
- What temporal query behavior is required for CSAPI interoperability?
- How should temporal semantics interact with retention and archival policies?

#### Spatial and Feature-of-Interest Semantics

- How should observations, status updates, and datastreams relate to spatial resources?
- How should feature of interest, sampling feature, observation location, system location, deployment location, platform trajectory, and datastream extent be distinguished?
- How should spatial-temporal queries behave?
- What spatial information belongs on observations versus datastreams versus systems/deployments?
- How should policy filtering or spatial generalization affect observation and status semantics?

#### Units, Observed Properties, and Semantic Binding

- How should datastreams bind to observed properties and units?
- How should individual observation values inherit or override datastream-level units or semantic bindings?
- How should multi-component result structures handle multiple units and observed properties?
- How should unit conversion, unit validation, semantic mapping, and unknown/local terms affect observation semantics?
- How should controlled properties and command-related semantics remain distinct from observed properties?

#### Validation and Acceptance Semantics

- What validation must happen before data becomes an accepted observation, status update, event, or latest value?
- What records can be staged, quarantined, accepted with warnings, rejected, or stored as raw payload only?
- How should validation errors be visible to clients, administrators, conformance harnesses, or source operators?
- How should validation state be represented in stored records?
- Which validation behavior belongs in ingestion versus client-facing retrieval?

#### Provenance, Source Trust, and Authority

- How should source identity, adapter identity, source authority, trust state, validation state, and transformation provenance be attached to observations and status records?
- Which provenance is visible to normal clients?
- Which provenance is administrator-only?
- How should provenance affect conflict resolution, late data, duplicates, policy filtering, and audit?
- How should simulated/test records differ from operational records?

#### Query, Filtering, Sorting, and Pagination

- What query/filter behavior is needed for datastreams, observations, and status updates:
  - by datastream,
  - by system,
  - by deployment,
  - by observed property,
  - by unit,
  - by feature of interest,
  - by time range,
  - by spatial filter,
  - by source,
  - by validation state,
  - by latest/stale state?
- How should sorting and pagination preserve stable behavior under concurrent ingestion?
- How should query behavior align with `IDR-SRV-011` and time-series storage findings?
- Which query capabilities are mandatory, recommended, optional, or deferred?

#### Persistence and Storage Semantics

- What records should be stored in time-series storage, relational metadata, geospatial storage, document storage, caches, or materialized views?
- Which records are authoritative versus derived?
- Which records should be immutable?
- Which records may be corrected or superseded?
- How should latest-value views be updated and invalidated?
- How should retention, archival, and deletion policies affect dynamic-data semantics?

#### Streaming and Publication Interaction

- Which datastream, observation, status, and event changes should trigger streaming/publication?
- Should streaming publish raw values, normalized values, events, latest-value updates, or resource-change notifications?
- How should publication order relate to persistence order?
- How should backfill and replay interact with observation and status semantics?
- Which findings should be handed to `IDR-SRV-035`?

#### Command, Control Stream, and Feasibility Interaction

- How should command status records differ from status updates?
- How should control stream definitions differ from datastream definitions?
- How should command results, command progress, feasibility responses, and command-status updates interact with observation/status semantics?
- Which command-related dynamic records should be queryable through CSAPI resources?
- Which findings should be handed to `IDR-SRV-036`, `IDR-SRV-037`, and `IDR-SRV-038`?

#### DDIL and Federation Implications

- How should datastream and observation semantics operate during disconnected or degraded operation?
- How should late-arriving data, replay, synchronization, stale values, and source authority be represented after reconnect?
- How should federated data be distinguished from local authoritative data?
- How should latest values and event histories handle delayed or conflicting updates?
- Which findings should be handed to `IDR-SRV-041`?

#### Security, Policy, and Releasability

- Which dynamic data may reveal sensitive system location, sensor activity, observed phenomena, command affordances, source behavior, or operational patterns?
- How should policy and releasability affect datastream visibility, observation visibility, status visibility, latest-value visibility, and query results?
- How should sensitive dynamic data be generalized, suppressed, delayed, redacted, or restricted?
- How should policy filtering interact with aggregates, counts, extents, and latest-value availability?
- Which findings should be handed to Category G and security-test topics?

#### Observability, Fixtures, Performance, and Interoperability

- What operational diagnostics are needed for datastream, observation, and status semantics?
- What fixtures are needed for valid/invalid datastreams, observations, status updates, latest values, multi-component results, nil values, unit mismatches, semantic mismatches, spatial-temporal data, late data, duplicate data, and policy-filtered data?
- What conformance tests are needed for observation access, status access, query behavior, latest values, pagination, validation, and error behavior?
- What performance/load/stress tests are needed for high-rate observation retrieval, latest values, status updates, and spatial-temporal filtering?
- What interoperability tests are needed for CSAPI Explorer, OS4CSAPI clients, webapp/mobile clients, and external CSAPI clients?

#### Implementation and Interoperability Lessons

- What datastream, observation, status, latest-value, and dynamic-data lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate observation semantics, status semantics, query, pagination, latest-value, or response-shape issues?
- What OS4CSAPI discussion lessons affect datastream, observation, and status semantics?
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

- `IDR-SRV-001` through `IDR-SRV-033` research reports, once complete:
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
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/
- OGC schemas: https://schemas.opengis.net/
- RFC 3339 - Date and Time on the Internet: Timestamps: https://www.rfc-editor.org/rfc/rfc3339

### Storage, Query, and Messaging Candidate Sources

Use current official documentation and primary-source technical material when executing the research. Candidate sources include, but are not limited to:

- PostgreSQL documentation: https://www.postgresql.org/docs/
- TimescaleDB documentation: https://docs.timescale.com/
- PostGIS documentation: https://postgis.net/documentation/
- MQTT v5 specification: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- NATS documentation: https://docs.nats.io/
- Apache Kafka documentation: https://kafka.apache.org/documentation/
- CloudEvents specification: https://cloudevents.io/
- Rust serde documentation: https://serde.rs/
- Rust SQLx documentation: https://docs.rs/sqlx/

These are candidates for evaluation, not assumed selections.

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, dynamic-data guidance, observation guidance, status/event guidance, tasking guidance, DDIL guidance, or standards-package annexes relevant to datastreams, observations, status updates, and NATO JISR sensor integration

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
- Category F findings from `IDR-SRV-031` through `IDR-SRV-033`
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

### Phase 1: Source Collection and Dynamic-Data Semantics Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for datastream, observation, and status update semantics.

**Tasks:**

1. Gather prior IDR reports, standards sources, implementation studies, storage/query sources, messaging sources, and project architecture sources.
2. Extract datastream, observation, status, event, latest-value, temporal, spatial, semantic, validation, and query concepts from each source.
3. Define inventory fields:
   - concept/resource type,
   - related CSAPI resource,
   - source authority,
   - semantic definition,
   - required relationships,
   - time fields,
   - spatial fields,
   - unit/semantic binding,
   - validation requirements,
   - persistence/query requirements,
   - policy implication,
   - downstream handoff.
4. Define evaluation criteria:
   - standards alignment,
   - semantic clarity,
   - client interoperability,
   - ingestion compatibility,
   - queryability,
   - persistence compatibility,
   - validation robustness,
   - DDIL suitability,
   - policy/security suitability,
   - fixture/testability.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation evidence, and technology documentation.

**Expected Output:** Dynamic-data semantics extraction framework and evaluation rubric.

### Phase 2: Datastream, Observation, Status, and Event Concept Inventory (3-4 hours)

**Objective:** Determine the semantic categories Glaux Server must distinguish.

**Tasks:**

1. Inventory CSAPI Part 2 concepts for datastreams, observations, status, events, control streams, commands, and feasibility resources.
2. Map supporting CSAPI Part 1 resources required for dynamic-data interpretation.
3. Classify dynamic record types as observation, status update, dynamic property, system event, source health, command status, audit-only, raw payload, normalized record, latest-value view, or derived record.
4. Identify required fields, optional fields, derived fields, and server-assigned fields.
5. Build a concept and resource-family semantics matrix.

**Expected Output:** Datastream/observation/status/event semantic taxonomy.

### Phase 3: Datastream and Observation Semantics Analysis (3-4 hours)

**Objective:** Define standards-aligned datastream and observation behavior.

**Tasks:**

1. Analyze datastream relationships to systems, procedures, deployments, sampling features, observed properties, units, SensorML outputs, SWE Common result structures, encodings, and sources.
2. Analyze observation structure, result structure, temporal fields, spatial fields, nil values, quality, uncertainty, provenance, and validation state.
3. Analyze multi-component observation results and multi-output systems.
4. Analyze query/filter/sort/pagination/latest-value behavior for observations.
5. Identify unresolved questions requiring implementation experiments or fixture validation.

**Expected Output:** Datastream and observation semantics matrix.

### Phase 4: Status, Dynamic Property, Event, and Latest-Value Analysis (2.5-3.5 hours)

**Objective:** Define status and event-like dynamic-data semantics.

**Tasks:**

1. Analyze status updates and dynamic properties.
2. Distinguish status updates from observations, system events, source health, command status, and metadata updates.
3. Analyze latest-value semantics for observations, status, and dynamic properties.
4. Analyze stale, degraded, unknown, unavailable, simulated, invalid, and policy-filtered values.
5. Analyze how status/events should interact with streaming/event publication.
6. Map findings to `IDR-SRV-035`, `IDR-SRV-041`, and observability topics.

**Expected Output:** Status/event/latest-value semantics matrix.

### Phase 5: Policy, DDIL, Commands, Fixtures, Performance, and Interoperability Analysis (3-4 hours)

**Objective:** Prepare dynamic-data semantics findings for downstream implementation and verification.

**Tasks:**

1. Analyze source-trust, provenance, policy/releasability, and security implications.
2. Analyze DDIL, replay, late data, stale values, and federation implications.
3. Analyze command-status and control-stream interactions.
4. Identify fixtures for valid/invalid datastreams, observations, status updates, latest values, multi-component results, nil values, unit mismatches, semantic mismatches, spatial-temporal data, late data, duplicate data, and policy-filtered data.
5. Identify conformance, performance, and interoperability tests.
6. Map findings to Category F, G, H, and I topics.

**Expected Output:** Dynamic-data downstream implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert datastream/observation/status research into a decision-usable baseline.

**Tasks:**

1. Consolidate semantic taxonomy, datastream behavior, observation behavior, status behavior, event/latest-value behavior, query behavior, and downstream implications.
2. Produce recommended datastream, observation, and status update semantics strategy with rationale and unresolved questions.
3. Identify sequencing for streaming, command, DDIL, security, deployment, fixture, conformance, and performance topics.
4. Produce downstream handoff matrix.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Datastream, observation, result, status update, dynamic property, system event, source-health, command-status, and latest-value concepts are identified and distinguished with source anchors.
- [ ] Datastream relationships to systems, procedures, deployments, sampling features, observed properties, units, SensorML, SWE Common, encodings, and sources are documented.
- [ ] Observation temporal, spatial, semantic, unit, nil-value, quality, uncertainty, validation, provenance, and query implications are documented.
- [ ] Status update, dynamic property, system event, latest-value, freshness, and stale-value implications are documented.
- [ ] Query/filter/sort/pagination, latest-value, policy/releasability, DDIL, streaming, command, fixture, conformance, performance, and interoperability implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Datastream, Observation, and Status Update Semantics Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-034-datastream-observation-and-status-update-semantics-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Dynamic-data semantics extraction methodology
5. Datastream, observation, status, event, and latest-value concept taxonomy
6. Datastream semantic model findings
7. Observation semantic model findings
8. Status update and dynamic property findings
9. System event and event-like dynamic record findings
10. Latest-value and historical-value findings
11. Temporal, spatial, semantic, unit, nil-value, quality, and provenance findings
12. Query, filtering, sorting, pagination, validation, and persistence findings
13. Streaming, DDIL, command, policy, security, and observability implications
14. Fixture, conformance, performance, and interoperability test implications
15. Downstream topic handoff matrix
16. Recommendations
17. Risks, constraints, and open questions
18. Validation against this plan's success criteria
19. References

The dynamic-data semantics matrix should include, at minimum:

- Concept/resource type
- Related CSAPI resource
- Source standard / source anchor
- Semantic definition
- Required relationships
- Time fields
- Spatial fields
- Unit/semantic binding
- Validation requirement
- Persistence/query requirement
- Latest-value implication
- Streaming/publication implication
- Security/policy implication
- Test/conformance implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-033` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, relevant OGC schemas/OpenAPI artifacts, storage/query/messaging sources, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Implementation-study and interoperability findings must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-035: Streaming and Event Publication Strategy`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-037: Feasibility and Command Validation Strategy`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- `IDR-SRV-041: DDIL Behavior, Caching, and Synchronization Semantics`
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

- This topic defines datastream, observation, and status update semantics, not final implementation code or database schemas.
- Dynamic data semantics should preserve clear distinctions among observations, status, events, command-status records, and source-health records.
- Implementation-study findings are useful but must not override standards-derived server responsibilities.
- Open question: Which latest-value views should be materialized versus computed?
- Open question: How should status update semantics differ from observation semantics in CSAPI client behavior?
- Open question: How should policy-filtered dynamic data affect latest values, counts, extents, and freshness indicators?
- Open question: Which advanced observation query capabilities are required for first implementation versus full-scope readiness?
- Risk: Treating status updates and system events as generic observations could weaken interoperability and operational meaning.
- Risk: Weak temporal semantics could break late data, replay, latest values, and DDIL synchronization.
- Risk: Unclear datastream/result-structure semantics could break clients and conformance tests.
- Risk: Policy-filtered dynamic data may leak information through metadata, latest-value availability, or query counts if not addressed.

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
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/
- OGC schemas: https://schemas.opengis.net/
- RFC 3339 - Date and Time on the Internet: Timestamps: https://www.rfc-editor.org/rfc/rfc3339
- PostgreSQL documentation: https://www.postgresql.org/docs/
- TimescaleDB documentation: https://docs.timescale.com/
- PostGIS documentation: https://postgis.net/documentation/
- MQTT v5 specification: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- NATS documentation: https://docs.nats.io/
- Apache Kafka documentation: https://kafka.apache.org/documentation/
- CloudEvents specification: https://cloudevents.io/
- Rust serde documentation: https://serde.rs/
- Rust SQLx documentation: https://docs.rs/sqlx/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

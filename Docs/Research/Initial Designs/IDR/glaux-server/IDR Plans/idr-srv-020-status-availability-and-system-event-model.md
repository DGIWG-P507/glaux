# Section 020: Status, Availability, and System Event Model - Research Plan

**Status:** Planned  
**Last Updated:** June 8, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-020-status-availability-and-system-event-model-report.md`

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

This topic research must define the Glaux Server planning baseline for the **status, availability, and system event model** across canonical CSAPI resources, dynamic properties, datastreams, deployments, control streams, commands, operational availability, freshness, and server-generated events.

The research must answer:

- What status, availability, health, readiness, freshness, and event concepts must Glaux Server support?
- How should Glaux Server distinguish system status, sensor/platform availability, datastream availability, control-stream availability, command feasibility, command status, service availability, resource lifecycle state, data freshness, and operational events?
- Which status and event concepts are defined or implied by CSAPI Part 1, CSAPI Part 2, SensorML, SWE Common, OGC API behavior, AEP-4789 server responsibilities, and implementation evidence?
- How should status and availability be represented as current state, historical state, system events, dynamic properties, observations, links, or derived server metadata?
- Which state changes should produce system events, and how should those events be categorized, timestamped, linked, queried, retained, and authorized?
- How should Glaux Server expose status and events in a way that is useful for NATO JISR operational users, external clients, conformance tests, and DDIL-informed operation?
- What downstream implications follow for dynamic-data ingestion, streaming/event publication, command lifecycle, persistence, security, validation, fixtures, conformance, and interoperability testing?

The output must be a status, availability, and system event model baseline with source anchors, status/event concept definitions, resource-family mappings, lifecycle and temporal implications, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows:

- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy`
- `IDR-SRV-017: Relationship and Linkage Model`
- `IDR-SRV-018: Temporal, Validity, and Freshness Model`
- `IDR-SRV-019: Registration, Update, and State Change Semantics`

Those topics define resource families, identity, relationships, temporal/freshness semantics, and state-change behavior. This topic defines the model for operational status, availability, and system events that results from those resources and changes. It closes the Category C resource/domain-model block and provides critical handoffs to SensorML/SWE representation topics, persistence, dynamic-data ingestion, streaming, command lifecycle, DDIL behavior, validation, and interoperability testing.

### Critical Constraint(s)

- Treat OGC API - Connected Systems Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, temporal/query behavior, AEP-4789 Volume II adoption context, and earlier IDR findings as controlling.
- Do not collapse status, availability, lifecycle state, command status, data freshness, service health, and events into a single concept.
- Do not design the database schema here. Identify status/event persistence and indexing implications and hand them to Category E topics.
- Do not finalize dynamic-data ingestion or streaming behavior here. Identify status/event needs and hand them to Category F topics.
- Do not finalize command lifecycle semantics here. Identify command-status and availability needs and hand detailed command lifecycle work to `IDR-SRV-036`.
- Do not finalize DDIL behavior here. Identify freshness, availability, last-known-state, cache, and synchronization implications for `IDR-SRV-041`.
- Do not finalize authorization or policy behavior here. Identify status/event disclosure and audit risks and hand them to Category G.
- Distinguish externally exposed CSAPI status/event resources from internal monitoring, service telemetry, health probes, and implementation logs.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What status, availability, health, freshness, and event concepts must Glaux Server support across canonical resource families?
2. Which status and event concepts are normative, inherited, representation-specific, implementation-support, optional, or unresolved?
3. How should current status, historical status, system events, dynamic properties, observations, and service health be distinguished?
4. Which resource lifecycle changes, dynamic-data updates, command/tasking changes, and operational changes should produce system events?
5. What downstream implications follow for persistence, dynamic data, streaming, DDIL behavior, command lifecycle, security, validation, fixtures, conformance, and interoperability?

### Detailed Questions

#### Standards and Status/Event Sources

- What status, availability, event, dynamic-property, and operational-state concepts are defined or implied by CSAPI Part 1?
- What status, availability, event, observation, datastream, control-stream, command, command-status, and feasibility concepts are defined or implied by CSAPI Part 2?
- What status or event behavior is inherited from OGC API - Features or the broader OGC API family?
- What SensorML concepts apply to capabilities, characteristics, modes, operational status, positions, contacts, inputs/outputs, and process metadata?
- What SWE Common concepts apply to dynamic data components, status values, categorical states, quantities, records, units, and command parameters?
- What status and event concepts are relevant from STANAG 4789 / AEP-4789 Volume II server responsibilities?
- Which status/event concepts appear in implementation studies or community lessons but are not clearly standards-defined?

#### Concept Taxonomy

- How should Glaux Server define and distinguish:
  - resource lifecycle state,
  - system operational status,
  - platform/sensor availability,
  - datastream availability,
  - observation freshness,
  - latest known value,
  - service health,
  - ingestion health,
  - publisher/source health,
  - control-stream availability,
  - command feasibility,
  - command status,
  - command execution state,
  - degraded or disconnected status,
  - policy-suppressed availability,
  - system event,
  - audit event,
  - resource lifecycle event,
  - observation/data event,
  - command/control event?
- Which concepts are externally exposed through CSAPI resources?
- Which concepts are internal operational telemetry only?
- Which concepts are derived from observations or events?
- Which concepts require standard vocabulary, controlled values, or configurable mappings?

#### Resource-Family Status and Availability Model

- What status and availability concepts apply to:
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
  - publisher/source connections,
  - server service endpoints?
- Which resource families require a current status view?
- Which resource families require historical status records?
- Which resource families require event history?
- Which resource families require availability or readiness indicators?
- Which resource families should not expose status directly?

#### System Event Model

- What event categories should Glaux Server support?
- Which state changes from `IDR-SRV-019` should produce system events?
- Which dynamic-data or status changes should produce system events?
- Which command/control lifecycle changes should produce system events?
- Which service, source, publisher, simulator, federation, cache, or DDIL changes should produce events?
- What event metadata is required: event type, affected resource, time, source, severity, confidence, provenance, lifecycle state, status change, previous/new values, and related links?
- How should events be linked to systems, datastreams, commands, deployments, observations, and status resources?
- How should events be queried, filtered, paginated, retained, and serialized?

#### Availability and Readiness Semantics

- How should Glaux Server distinguish available, unavailable, degraded, unknown, stale, retired, disabled, denied, unreachable, disconnected, policy-suppressed, and not-applicable states?
- How should datastream availability differ from system availability?
- How should command/control availability differ from command feasibility?
- How should service health differ from system status?
- How should operational readiness be represented without inventing unsupported standards semantics?
- What availability concepts should be configurable by deployment, collection, source, or profile?

#### Freshness, Last-Known-State, and DDIL Implications

- How should status and events interact with temporal freshness findings from `IDR-SRV-018`?
- How should Glaux Server represent stale status, old-but-valid observations, unavailable sources, cached last-known state, reconnected sources, delayed events, and out-of-order status updates?
- How should DDIL conditions affect status confidence, event publication, cached state, synchronization, and client warnings?
- Which findings should be handed to `IDR-SRV-041`, `IDR-SRV-035`, and dynamic-data topics?

#### Dynamic Data, Observations, and Status Update Semantics

- Which status values are observations?
- Which status values are dynamic properties?
- Which status values are derived metadata?
- Which status values are system events?
- How should Glaux Server map observed properties, dynamic properties, status streams, and latest values?
- How should status updates be normalized, validated, timestamped, stored, and exposed?
- Which findings should be handed to `IDR-SRV-034`, `IDR-SRV-035`, and `IDR-SRV-027`?

#### Command, Feasibility, and Control Stream Status

- How should Glaux Server distinguish control-stream availability, command feasibility, command acceptance, command execution status, command result, command failure, cancellation, timeout, and audit status?
- Which command lifecycle states should produce events?
- Which command lifecycle states are externally visible?
- Which command status fields are operationally sensitive?
- Which findings should be handed to `IDR-SRV-036`, `IDR-SRV-037`, and `IDR-SRV-038`?

#### Persistence, Query, and Validation Implications

- Which status and event records must be persisted?
- Which status and event records are derived or cached?
- What indexing is required for time, resource, event type, severity, status value, source, and command/control correlation?
- What validation rules apply to status values, event types, timestamps, source authority, links, severity, lifecycle transitions, and command status?
- Which fixtures and golden files are needed to test status, availability, and event behavior?
- Which conformance, negative, schema, streaming, and interoperability tests are implied?

#### Security, Policy, and Releasability Implications

- Which status and event information may reveal sensitive operational state?
- How should availability, command status, source health, policy suppression, and resource existence be protected?
- How should status/event links be filtered by authorization?
- Which events require audit retention?
- Which event categories are operational telemetry rather than externally releasable CSAPI content?
- Which findings should be handed to `IDR-SRV-039`, `IDR-SRV-040`, `IDR-SRV-038`, and `IDR-SRV-055`?

#### Implementation and Interoperability Lessons

- What status, availability, and event lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate status, event, availability, freshness, or command-status issues?
- What OS4CSAPI discussion lessons affect status/event strategy?
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

- `IDR-SRV-001` through `IDR-SRV-019` research reports, once complete:
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

### HTTP, Caching, and Event-Adjacent Sources

- RFC 3339 - Date and Time on the Internet: Timestamps: https://www.rfc-editor.org/rfc/rfc3339
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 - HTTP Caching: https://www.rfc-editor.org/rfc/rfc9111
- RFC 8288 - Web Linking: https://www.rfc-editor.org/rfc/rfc8288
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, or standards-package annexes relevant to status, availability, system events, dynamic data, command/control, operational state, DDIL behavior, and server responsibilities

### Implementation and Lessons-Learned Sources

Use implementation-study outputs and source repositories as non-normative evidence:

- OSH / OpenSensorHub sources and `IDR-SRV-014A`
- Connected Systems Go sources and `IDR-SRV-014B`
- pygeoapi sources and `IDR-SRV-014C`
- SECD sources and `IDR-SRV-014D`
- OS4CSAPI Client Smoke Test Findings and `IDR-SRV-014E`
- SECD Interoperability Findings and `IDR-SRV-014F`
- OS4CSAPI Discussions Lessons-Learned and `IDR-SRV-014G`
- Canonical Glaux Server Resource Model findings from `IDR-SRV-015`
- Identifier, URI, and Resource Lifecycle findings from `IDR-SRV-016`
- Relationship and Linkage findings from `IDR-SRV-017`
- Temporal, Validity, and Freshness findings from `IDR-SRV-018`
- Registration, Update, and State Change findings from `IDR-SRV-019`
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

### Phase 1: Source Collection and Status/Event Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for status, availability, and event modeling.

**Tasks:**

1. Gather standards, schemas, OpenAPI artifacts, prior IDR reports, implementation studies, smoke-test findings, interoperability findings, and discussion lessons.
2. Extract all status, availability, readiness, freshness, service health, system event, dynamic property, command status, and operational-state concepts from each source.
3. Define inventory fields:
   - resource family,
   - status/event concept,
   - source authority,
   - current/historical/derived/event classification,
   - external/internal exposure,
   - temporal fields,
   - relationship targets,
   - event type/category,
   - availability semantics,
   - freshness/staleness role,
   - security implication,
   - persistence implication,
   - validation/test implication.
4. Define classification values:
   - normative,
   - inherited,
   - standards-package-specific,
   - representation-specific,
   - derived,
   - implementation-support,
   - implementation-specific,
   - optional,
   - unresolved.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation observations, and community discussion.

**Expected Output:** Source inventory and status/event extraction framework.

### Phase 2: Standards-Derived Status, Availability, and Event Extraction (2.5-3.5 hours)

**Objective:** Extract status, availability, and event expectations from standards and official artifacts.

**Tasks:**

1. Extract CSAPI Part 1 status and availability implications for feature resources.
2. Extract CSAPI Part 2 behavior for dynamic properties, status, events, datastreams, observations, control streams, commands, command status, and feasibility.
3. Extract inherited OGC API and HTTP behavior relevant to service availability, errors, caching, and client interpretation.
4. Extract SensorML concepts relevant to operational descriptions, capabilities, modes, inputs/outputs, and status-like properties.
5. Extract SWE Common concepts relevant to status values, categories, quantities, records, observed properties, dynamic properties, and command parameters.
6. Extract AEP-4789 status/event/server-responsibility implications from project-available material.

**Expected Output:** Standards-derived status, availability, and event inventory.

### Phase 3: Resource-Family Status and Availability Model Analysis (3-4 hours)

**Objective:** Define status and availability behavior by canonical resource family.

**Tasks:**

1. Use `IDR-SRV-015` through `IDR-SRV-019` findings as the organizing baseline.
2. For each resource family, identify status concepts, availability concepts, current-state views, historical records, dynamic properties, event needs, and derived metadata.
3. Distinguish resource lifecycle state, operational status, service health, data freshness, command status, and availability.
4. Identify queryability, representation, temporal, relationship, and client-interoperability implications.
5. Identify unresolved status/availability questions requiring downstream research or prototype validation.

**Expected Output:** Resource-family status and availability model matrix.

### Phase 4: System Event and State-Change Event Analysis (2.5-3 hours)

**Objective:** Define system event categories and event-generation semantics.

**Tasks:**

1. Identify event categories for resource lifecycle, system operational state, data/observation updates, status updates, command/control lifecycle, service/source health, federation/DDIL synchronization, and administrative changes.
2. Identify which state changes from `IDR-SRV-019` should generate events.
3. Identify event metadata, links, severity/confidence/provenance needs, and time fields.
4. Identify query, filtering, retention, and representation requirements.
5. Identify handoffs to streaming/event-publication, dynamic data, command lifecycle, security/audit, and persistence topics.

**Expected Output:** System event model and generation matrix.

### Phase 5: Freshness, DDIL, Persistence, Security, Validation, and Test Implications (2.5-3 hours)

**Objective:** Prepare status/event findings for downstream topics.

**Tasks:**

1. Analyze freshness, staleness, last-known-state, cache, unavailable, degraded, disconnected, and policy-suppressed status semantics.
2. Identify DDIL, synchronization, replay, delayed-event, out-of-order, and reconnect implications.
3. Identify persistence, indexing, retention, and derived-state requirements without designing the database schema.
4. Identify validation rules for status values, event types, timestamps, links, severity, lifecycle correlation, source authority, and command status.
5. Identify security and policy risks from status/event exposure.
6. Identify conformance, golden-file, fixture, negative, streaming, DDIL, command-status, and interoperability tests.
7. Map findings to Category D, E, F, G, and I topics.

**Expected Output:** Downstream status/event implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert status, availability, and event research into a decision-usable baseline.

**Tasks:**

1. Consolidate standards-derived inventory, resource-family status model, availability model, event model, freshness/DDIL implications, and downstream handoffs.
2. Produce status, availability, and event recommendations by resource family.
3. Resolve conflicts and ambiguities where possible.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Status, availability, health, readiness, freshness, command status, and system event concepts are identified and distinguished.
- [ ] Resource-family status and availability requirements are mapped.
- [ ] Current-state, historical-state, dynamic-property, observation-derived, event, command-status, service-health, and implementation-telemetry concepts are distinguished.
- [ ] System event categories and event-generation triggers are identified.
- [ ] Freshness, staleness, unavailable, degraded, disconnected, last-known-state, and DDIL implications are documented.
- [ ] Persistence, query, validation, security/policy, conformance, fixture, and interoperability implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Status, Availability, and System Event Model Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-020-status-availability-and-system-event-model-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Status/event extraction methodology
5. Standards-derived status, availability, and event inventory
6. Status, availability, health, freshness, and event taxonomy
7. Resource-family status and availability model
8. Current-state, historical-state, dynamic-property, observation-derived, and event-model findings
9. System event categories, triggers, metadata, links, and retention findings
10. Command status, feasibility, and control-stream availability implications
11. Freshness, staleness, degraded operation, cache, and DDIL findings
12. Persistence, indexing, query, validation, and fixture implications
13. Security, policy, releasability, and audit implications
14. Conformance, interoperability, and test implications
15. Downstream topic handoff matrix
16. Recommendations
17. Risks, constraints, and open questions
18. Validation against this plan's success criteria
19. References

The status/event matrix should include, at minimum:

- Resource family
- Status/event concept
- Source standard / source anchor
- Authority classification
- Current/historical/derived/event classification
- External/internal exposure
- Temporal fields
- Related resources / links
- Event trigger, if applicable
- Availability/freshness semantics
- Security/policy implication
- Persistence implication
- Validation implication
- Test implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-019` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, OGC schemas, OpenAPI artifacts, HTTP/caching/problem-details standards, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-021: SensorML Representation Strategy`
- `IDR-SRV-022: SWE Common Data Component Strategy`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-024: Units, Observed Properties, and Semantic Binding Strategy`
- `IDR-SRV-025: Database and Persistence Architecture Options`
- `IDR-SRV-027: Time-Series Observation Storage Strategy`
- `IDR-SRV-033: Dynamic Data Ingestion and Normalization Pipeline`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-035: Streaming and Event Publication Strategy`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints`
- `IDR-SRV-041: DDIL Behavior, Caching, and Synchronization Semantics`
- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
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

- This topic defines status, availability, and system event semantics, not the database schema, monitoring system, or final pub/sub design.
- This topic should distinguish CSAPI-exposed status/events from internal health probes and implementation telemetry.
- Implementation-study findings are useful but must not override standards-derived semantics.
- Open question: Which status values require controlled vocabularies versus implementation profiles?
- Open question: Which state changes should generate externally visible system events?
- Open question: Which status and event fields are operationally sensitive and should be filtered or suppressed?
- Open question: How should Glaux Server represent stale-but-valid, unavailable, degraded, disconnected, and policy-suppressed status?
- Open question: Which status/event records should be retained permanently versus summarized or expired?
- Risk: Collapsing operational status, resource lifecycle state, command status, freshness, and service health could create ambiguous client behavior.
- Risk: Exposing status/events without policy controls could reveal sensitive operational state.
- Risk: Missing event history could weaken auditability and troubleshooting.
- Risk: Treating one implementation's event/status vocabulary as canonical could distort Glaux Server design.

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
- RFC 3339 - Date and Time on the Internet: Timestamps: https://www.rfc-editor.org/rfc/rfc3339
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 - HTTP Caching: https://www.rfc-editor.org/rfc/rfc9111
- RFC 8288 - Web Linking: https://www.rfc-editor.org/rfc/rfc8288
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

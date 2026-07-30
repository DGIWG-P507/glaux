# Section 018: Temporal, Validity, and Freshness Model - Research Plan

**Status:** Planned  
**Last Updated:** June 8, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-018-temporal-validity-and-freshness-model-report.md`

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

This topic research must define the Glaux Server planning baseline for the **temporal, validity, and freshness model** across canonical CSAPI resources, dynamic data, status, events, observations, commands, deployments, metadata, and externally sourced or federated resources.

The research must answer:

- What temporal concepts must Glaux Server support for each canonical resource family?
- How should Glaux Server distinguish phenomenon time, result time, valid time, transaction time, ingestion time, publication time, update time, event time, command lifecycle time, deployment time, availability time, expiration time, and freshness/staleness time?
- How should time be represented, stored, queried, validated, exposed, and linked across CSAPI Part 1, CSAPI Part 2, OGC API - Features, SensorML, SWE Common, and AEP-4789 server responsibilities?
- How should Glaux Server model current state versus historical state, latest values versus observations, stale data versus unavailable data, retired resources versus expired resources, and cached/federated data versus authoritative data?
- What temporal and freshness lessons from implementation studies, smoke tests, SECD interoperability findings, and OS4CSAPI discussions should influence Glaux Server design?
- What downstream implications follow for registration/update semantics, status/event modeling, persistence, query behavior, dynamic data, streaming, tasking, DDIL behavior, validation, fixtures, and interoperability testing?

The output must be a temporal, validity, and freshness model baseline with source anchors, temporal concept definitions, resource-family mappings, query and representation implications, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows:

- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy`
- `IDR-SRV-017: Relationship and Linkage Model`

Those topics define resource families, stable identity, lifecycle strategy, and relationships. This topic defines the time and freshness semantics needed to make those resources operationally meaningful. It must precede provenance modeling, status/event modeling, persistence architecture, write/ingestion semantics, time-series storage, dynamic-data semantics, streaming, tasking, DDIL behavior, validation, fixtures, and interoperability tests.

### Critical Constraint(s)

- Treat OGC API - Connected Systems Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, temporal/query behavior from the OGC API family, AEP-4789 Volume II adoption context, and earlier IDR findings as controlling.
- Do not collapse all time fields into a single timestamp. Distinguish temporal concepts explicitly.
- Do not design the database schema here. Identify temporal persistence and indexing implications and hand them to Category E topics.
- Do not finalize registration/update operation semantics here. Identify provenance implications for `IDR-SRV-019` and hand operation semantics to `IDR-SRV-031`.
- Do not finalize status/event resource semantics here. Identify time/freshness needs and hand detailed status/event modeling to `IDR-SRV-020`.
- Do not finalize dynamic-data storage or streaming semantics here. Identify temporal needs and hand them to Category F.
- Do not finalize DDIL behavior here. Identify freshness, cache, synchronization, and degraded-operation implications for Category G.
- Distinguish current-state views, latest values, historical records, events, observations, metadata validity, system availability, and command lifecycle timelines.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What temporal concepts must Glaux Server support across canonical resource families?
2. Which temporal concepts are normative, inherited, representation-specific, implementation-support, optional, or unresolved?
3. How should validity, freshness, staleness, availability, and history be represented and queried?
4. How should temporal behavior differ for descriptions, deployments, observations, status, events, datastreams, commands, and feasibility?
5. What downstream implications follow for registration/update, status/events, persistence, query, streaming, DDIL behavior, validation, fixtures, conformance, and interoperability?

### Detailed Questions

#### Standards and Temporal Sources

- What temporal concepts are defined or implied by CSAPI Part 1?
- What temporal concepts are defined or implied by CSAPI Part 2?
- What temporal behavior is inherited from OGC API - Features?
- What time, validity, and event concepts are represented by SensorML?
- What time, sampling, value, and data component concepts are represented by SWE Common?
- What temporal concepts are relevant from STANAG 4789 / AEP-4789 server responsibilities?
- Which temporal concepts appear in implementation studies or community lessons but are not clearly standards-defined?
- Which temporal concepts are necessary for NATO JISR operational interpretation even when not explicitly named by CSAPI?

#### Temporal Concept Taxonomy

- How should Glaux Server define and distinguish:
  - phenomenon time,
  - result time,
  - valid time,
  - transaction time,
  - ingestion time,
  - publication time,
  - update time,
  - creation time,
  - event time,
  - command issue time,
  - command acceptance time,
  - command execution time,
  - command completion/failure/cancellation time,
  - feasibility evaluation time,
  - deployment start/end time,
  - system availability time,
  - metadata validity time,
  - cache time,
  - synchronization time,
  - expiration time,
  - freshness/staleness time?
- Which concepts are required externally, internally, or both?
- Which concepts should be represented explicitly versus derived?
- Which concepts require standard names, schema fields, metadata annotations, or implementation conventions?

#### Resource-Family Temporal Model

- What temporal concepts apply to:
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
  - OpenAPI/schema/documentation resources?
- Which resources are timeless descriptions, time-valid descriptions, temporal records, time-series records, state snapshots, event records, or workflow records?
- Which resource families require temporal versioning or historical retention?
- Which resource families require a current-state representation?

#### Validity and Lifecycle Interaction

- How does temporal validity interact with resource lifecycle states from `IDR-SRV-016`?
- How should Glaux Server represent active, inactive, retired, archived, superseded, stale, unavailable, unknown, pending, completed, failed, canceled, and expired states over time?
- Which lifecycle transitions need timestamps?
- Which resources require historical state tracking?
- Which temporal lifecycle findings should be handed to `IDR-SRV-019`, `IDR-SRV-020`, `IDR-SRV-031`, and `IDR-SRV-036`?

#### Freshness, Staleness, and Availability

- What does freshness mean for metadata, systems, datastreams, status, observations, latest values, events, commands, and federated resources?
- How should Glaux Server distinguish stale data from unavailable data, old-but-valid observations, cached data, unknown freshness, retired resources, and policy-suppressed resources?
- Which freshness indicators should be exposed to clients?
- How should freshness support tactical-edge and DDIL-informed operation?
- How should freshness affect warnings, errors, response metadata, links, status resources, and query results?
- What operational thresholds should be configurable rather than hard-coded?

#### Temporal Query and Filtering Implications

- What temporal query patterns are required or expected by CSAPI, OGC API - Features, and implementation evidence?
- How should Glaux Server support time filters for observations, events, deployments, status, command history, and resource validity?
- What temporal filters apply to current resources versus historical records?
- What default temporal behavior should apply when no time filter is supplied?
- How should pagination, sorting, latest values, count, and selection interact with time filters?
- Which findings should be handed to query, persistence, time-series storage, and conformance-test topics?

#### Observation, Event, and Dynamic Data Time

- How should phenomenon time, result time, ingestion time, and publication time be represented for observations?
- How should status updates and dynamic properties be timestamped?
- How should system events be timestamped and linked to affected resources?
- How should real-time, historical, replay, and streaming data maintain temporal consistency?
- Which findings should be handed to `IDR-SRV-020`, `IDR-SRV-034`, and `IDR-SRV-035`?

#### Command, Feasibility, and Tasking Time

- What temporal concepts apply to feasibility checks, control streams, commands, command status, execution windows, timeouts, retries, and audit history?
- How should command lifecycle timelines be represented?
- How should time constraints and validity windows be represented for command parameters?
- How should late, stale, expired, or superseded commands be handled?
- Which findings should be handed to `IDR-SRV-036`, `IDR-SRV-037`, and `IDR-SRV-038`?

#### Persistence, Indexing, and Consistency Implications

- What temporal persistence requirements follow from the canonical model?
- Which temporal fields must be indexed for query and performance?
- Which records need append-only history versus update-in-place behavior?
- How should temporal consistency be maintained across related resources?
- How should clock skew, source timestamp uncertainty, ingestion delay, replay, and out-of-order data be handled?
- Which findings should be handed to `IDR-SRV-025`, `IDR-SRV-027`, `IDR-SRV-029`, and `IDR-SRV-054`?

#### Implementation and Interoperability Lessons

- What temporal, validity, and freshness lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate temporal query, timestamp, latest value, stale data, observation history, or command timeline problems?
- What OS4CSAPI discussion lessons affect temporal modeling?
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

- `IDR-SRV-001` through `IDR-SRV-017` research reports, once complete:
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

### Time, HTTP, and Data Representation Sources

- ISO 8601 date and time representation guidance, as applicable through referenced standards and schemas
- RFC 3339 - Date and Time on the Internet: Timestamps: https://www.rfc-editor.org/rfc/rfc3339
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 - HTTP Caching: https://www.rfc-editor.org/rfc/rfc9111
- RFC 8288 - Web Linking: https://www.rfc-editor.org/rfc/rfc8288
- IANA HTTP Field Name Registry: https://www.iana.org/assignments/http-fields/http-fields.xhtml

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, or standards-package annexes relevant to time, validity, freshness, status, observations, tasking, server responsibilities, and operational availability

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

### Phase 1: Source Collection and Temporal Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for temporal modeling.

**Tasks:**

1. Gather standards, schemas, OpenAPI artifacts, prior IDR reports, implementation studies, smoke-test findings, interoperability findings, and discussion lessons.
2. Extract all time, validity, freshness, staleness, availability, event, observation, command, deployment, cache, update, and history concepts from each source.
3. Define temporal inventory fields:
   - resource family,
   - temporal concept,
   - source authority,
   - timestamp or interval,
   - external/internal exposure,
   - required/optional/derived,
   - representation pattern,
   - queryability,
   - freshness/staleness role,
   - lifecycle interaction,
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

**Expected Output:** Source inventory and temporal extraction framework.

### Phase 2: Standards-Derived Temporal Concept Extraction (2.5-3.5 hours)

**Objective:** Extract temporal expectations from standards and official artifacts.

**Tasks:**

1. Extract CSAPI Part 1 temporal behavior for feature resources, deployments, systems, properties, and related resources.
2. Extract CSAPI Part 2 temporal behavior for datastreams, observations, status, events, control streams, commands, feasibility, and dynamic data.
3. Extract inherited OGC API - Features temporal query and feature validity behavior.
4. Extract SensorML temporal concepts relevant to systems, procedures, deployments, positions, capabilities, and descriptions.
5. Extract SWE Common temporal concepts relevant to data records, observations, values, and command parameters.
6. Extract relevant HTTP caching and freshness behavior.
7. Extract AEP-4789 temporal implications from project-available material.

**Expected Output:** Standards-derived temporal concept inventory.

### Phase 3: Resource-Family Temporal Model Analysis (3-4 hours)

**Objective:** Define temporal concepts by canonical resource family.

**Tasks:**

1. Use `IDR-SRV-015`, `IDR-SRV-016`, and `IDR-SRV-017` findings as the organizing baseline.
2. Map temporal concepts to each canonical resource family.
3. Distinguish current-state resources, historical resources, temporal records, snapshots, events, observations, workflows, and descriptions.
4. Identify timestamp versus interval requirements.
5. Identify queryability, representation, default behavior, and client-interoperability implications.
6. Identify unresolved temporal questions requiring downstream research or prototype validation.

**Expected Output:** Resource-family temporal model matrix.

### Phase 4: Freshness, Staleness, Availability, and DDIL Implication Analysis (2.5-3 hours)

**Objective:** Define operational freshness and availability semantics.

**Tasks:**

1. Distinguish freshness, staleness, expiration, unavailability, unknown status, cached data, last-known state, old-but-valid observations, retired resources, and policy-suppressed resources.
2. Identify freshness indicators needed for metadata, systems, datastreams, observations, status, events, commands, and federated data.
3. Identify configurable freshness thresholds.
4. Identify DDIL, cache, synchronization, and degraded-operation implications.
5. Identify handoffs to status/event modeling, dynamic-data semantics, DDIL behavior, deployment, and testing topics.

**Expected Output:** Freshness/staleness/availability implication matrix.

### Phase 5: Query, Persistence, Validation, Test, and Interoperability Implications (2.5-3 hours)

**Objective:** Prepare temporal findings for downstream topics.

**Tasks:**

1. Identify temporal query patterns, filter semantics, pagination/sorting interactions, latest-value behavior, and history-query requirements.
2. Identify temporal persistence, indexing, consistency, out-of-order data, clock-skew, and retention implications without designing the database schema.
3. Identify validation rules for timestamps, intervals, ordering, freshness metadata, lifecycle transitions, and command timelines.
4. Identify conformance, golden-file, fixture, negative, temporal-query, streaming, and interoperability tests.
5. Map findings to Category C, E, F, G, and I topics.

**Expected Output:** Downstream temporal implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert temporal, validity, and freshness research into a decision-usable baseline.

**Tasks:**

1. Consolidate standards-derived temporal inventory, resource-family temporal model, freshness semantics, query/persistence implications, and implementation lessons.
2. Produce temporal model recommendations by resource family and temporal concept.
3. Resolve conflicts and ambiguities where possible.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Temporal concepts are identified and distinguished with source anchors.
- [ ] Resource-family temporal requirements are mapped.
- [ ] Phenomenon time, result time, valid time, ingestion time, publication time, update time, event time, command time, deployment time, cache time, and freshness/staleness time are distinguished where applicable.
- [ ] Current-state, historical-state, latest-value, event, observation, command, and description temporal models are distinguished.
- [ ] Freshness, staleness, unavailability, cache, expiration, and DDIL implications are documented.
- [ ] Query, persistence, validation, security/policy, conformance, fixture, and interoperability implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Temporal, Validity, and Freshness Model Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-018-temporal-validity-and-freshness-model-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Temporal extraction methodology
5. Standards-derived temporal concept inventory
6. Temporal concept taxonomy and definitions
7. Resource-family temporal model
8. Current-state, history, latest-value, event, observation, and command timeline findings
9. Validity, lifecycle, and versioning temporal findings
10. Freshness, staleness, availability, cache, and DDIL findings
11. Temporal query and filtering implications
12. Persistence, indexing, consistency, and retention implications
13. Validation, fixture, conformance, and interoperability test implications
14. Downstream topic handoff matrix
15. Recommendations
16. Risks, constraints, and open questions
17. Validation against this plan's success criteria
18. References

The temporal model matrix should include, at minimum:

- Resource family
- Temporal concept
- Timestamp or interval
- Source standard / source anchor
- Authority classification
- External/internal exposure
- Required/optional/derived status
- Representation pattern
- Queryability
- Lifecycle interaction
- Freshness/staleness implication
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
- `IDR-SRV-001` through `IDR-SRV-017` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, OGC schemas, OpenAPI artifacts, temporal/HTTP/caching standards, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-019: Provenance, Lineage, Quality, and Trust Metadata Model`
- `IDR-SRV-020: Status, Availability, and System Event Model`
- `IDR-SRV-025: Database and Persistence Architecture Options`
- `IDR-SRV-026: Geospatial Storage and Query Strategy`
- `IDR-SRV-027: Time-Series Observation Storage Strategy`
- `IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy`
- `IDR-SRV-031: Server Write and Ingestion Model`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-035: Streaming and Event Publication Strategy`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-042: DDIL-Informed Server Semantics`
- `IDR-SRV-043: Server Synchronization and Conflict Handling Boundary`
- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
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

- This topic defines temporal, validity, and freshness strategy, not the database schema.
- This topic should preserve temporal distinctions needed for conformance, operational interpretation, audit, and interoperability.
- Implementation-study findings are useful but must not override standards-derived temporal semantics.
- Open question: Which temporal fields must be externally exposed versus internally retained?
- Open question: Which resources require temporal version history versus simple update timestamps?
- Open question: How should latest-value endpoints represent old-but-valid observations versus stale status?
- Open question: How should source clock skew and ingestion delay be represented?
- Open question: Which freshness thresholds should be configurable at deployment, collection, datastream, or source level?
- Risk: Collapsing temporal concepts into a single timestamp could break observation, event, command, and audit semantics.
- Risk: Missing freshness metadata could mislead tactical users during DDIL or degraded operations.
- Risk: Overly complex temporal modeling could burden implementation without interoperability benefit.
- Risk: Treating one implementation's time behavior as canonical could distort Glaux Server design.

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
- IANA HTTP Field Name Registry: https://www.iana.org/assignments/http-fields/http-fields.xhtml
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

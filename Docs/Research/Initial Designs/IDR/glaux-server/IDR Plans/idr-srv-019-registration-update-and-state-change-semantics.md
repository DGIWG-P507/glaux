# Section 019: Registration, Update, and State Change Semantics - Research Plan

**Status:** Planned  
**Last Updated:** June 8, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-019-registration-update-and-state-change-semantics-report.md`

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

This topic research must define the Glaux Server planning baseline for **registration, update, and state change semantics** across canonical CSAPI resources, system descriptions, deployments, properties, datastreams, observations, status, events, control streams, commands, feasibility resources, and externally sourced data.

The research must answer:

- Which resource families can be registered, imported, created, updated, replaced, retired, archived, deleted, or derived by Glaux Server?
- Which state changes are externally visible API behavior versus internal implementation behavior?
- How should Glaux Server distinguish registration, creation, import, update, patch, replacement, deactivation, retirement, archival, deletion, supersession, cache refresh, and re-synchronization?
- What operation semantics, idempotency expectations, validation rules, conflict handling, lifecycle-state transitions, versioning behavior, audit needs, and authorization boundaries apply to each resource family?
- How should registration and update behavior preserve stable identifiers, relationships, temporal validity, freshness, status, provenance, and historical traceability?
- How should Glaux Server handle updates from publishers, simulators, external/federated sources, manual administration, batch loading, and DDIL synchronization?
- What downstream implications follow for persistence, transactions, command lifecycle, security, validation, conformance, fixture design, and interoperability testing?

The output must be a registration, update, and state change semantics baseline with source anchors, lifecycle operation definitions, resource-family mappings, validation and conflict rules, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows:

- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy`
- `IDR-SRV-017: Relationship and Linkage Model`
- `IDR-SRV-018: Temporal, Validity, and Freshness Model`

Those topics define resource families, identity/lifecycle strategy, relationship behavior, and temporal/freshness semantics. This topic defines how resources and states enter, change, retire, and persist in Glaux Server over time. It must precede detailed status/event modeling, persistence architecture, transaction/concurrency strategy, ingestion pipelines, dynamic-data semantics, command lifecycle modeling, security/audit strategy, validation, and test design.

### Critical Constraint(s)

- Treat OGC API - Connected Systems Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, HTTP semantics, AEP-4789 Volume II adoption context, and earlier IDR findings as controlling.
- Do not invent mutation APIs or state transitions that conflict with CSAPI resource semantics, OGC API behavior, HTTP method semantics, or the canonical Glaux Server resource model.
- Do not design the database schema here. Identify persistence, transaction, and indexing implications and hand them to Category E topics.
- Do not finalize command lifecycle semantics here. Identify command state-change needs and hand detailed command lifecycle work to `IDR-SRV-036`.
- Do not finalize status/event resource semantics here. Identify state-change event needs and hand detailed status/event modeling to `IDR-SRV-020`.
- Do not finalize authentication, authorization, policy, or audit design here. Identify security and audit implications and hand them to Category G.
- Do not assume all resource families require external write APIs. Distinguish externally mutable resources, internally derived resources, imported resources, append-only records, and read-only views.
- Distinguish operational state changes from descriptive metadata updates and from historical observation/event records.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. Which Glaux Server resource families require registration, update, replacement, retirement, archival, deletion, import, or derived-state semantics?
2. Which state changes are externally visible and which are internal?
3. What operation semantics, validation rules, conflict behavior, idempotency expectations, and lifecycle transitions apply by resource family?
4. How should updates preserve identity, relationships, temporal validity, freshness, provenance, and auditability?
5. What downstream implications follow for status/events, persistence, transactions, ingestion, command lifecycle, security, validation, fixtures, conformance, and interoperability?

### Detailed Questions

#### Standards and Operation Sources

- What registration, update, mutation, state change, and resource lifecycle concepts are defined or implied by CSAPI Part 1?
- What dynamic-data, command, feasibility, control stream, status, event, and update concepts are defined or implied by CSAPI Part 2?
- What create/update/delete or feature lifecycle behavior is inherited from OGC API - Features or related OGC API standards?
- What HTTP method semantics apply to GET, POST, PUT, PATCH, DELETE, conditional requests, conflict responses, and idempotency?
- What SensorML update and system-description implications apply?
- What SWE Common update implications apply to datastreams, result structures, command parameters, and properties?
- What registration/update responsibilities are implied by STANAG 4789 / AEP-4789 Volume II server responsibilities?
- Which mutation/state concepts appear in implementation studies or community lessons but are not clearly standards-defined?

#### Resource-Family Registration and Update Scope

- Which resource families should be externally registerable?
- Which resource families should be externally updatable?
- Which resource families should be append-only?
- Which resource families should be derived from ingestion pipelines or external publishers?
- Which resource families should be administrative-only?
- Which resource families should be read-only API views?
- Which resource families require batch import/export?
- Which resource families require replacement or supersession rather than in-place update?
- Which resource families require deletion, retirement, archival, or deactivation semantics?

#### Operation Semantics

- How should Glaux Server define registration, creation, import, update, partial update, replacement, upsert, deactivation, retirement, archival, deletion, cache refresh, re-synchronization, derived-state computation, and event publication?
- Which HTTP methods, status codes, error bodies, and response patterns apply?
- Which operations are idempotent?
- Which operations require conditional requests, ETags, version fields, or conflict detection?
- Which operations require validation before acceptance?
- Which operations require asynchronous processing or accepted/pending states?

#### Lifecycle State Transition Model

- What state transitions apply to systems, procedures, deployments, sampling features, properties, datastreams, observations, status resources, events, control streams, commands, feasibility resources, and command status?
- Which transitions are allowed, forbidden, reversible, irreversible, administrative-only, or policy-controlled?
- Which transitions should produce system events?
- Which transitions should update freshness/status metadata?
- Which transitions require audit records?
- Which transitions require validation against relationships and temporal validity?
- Which transitions affect links, reverse links, queries, and cached clients?

#### Update Validation and Conflict Handling

- What validation rules apply before resource creation, update, replacement, retirement, archival, or deletion?
- How should Glaux Server handle duplicate identifiers, stale versions, invalid relationships, missing required links, schema-invalid payloads, incompatible SensorML/SWE structures, temporal conflicts, unauthorized state changes, policy-suppressed resources, concurrent updates, out-of-order publisher data, and repeated DDIL synchronization batches?
- What error behavior and status codes should be used?
- Which validation/conflict findings should be handed to `IDR-SRV-013`, Category E, Category G, and Category I topics?

#### Provenance, Source Authority, and Federation

- How should updates from publishers, simulators, external systems, federated nodes, manual administrators, batch imports, and cached replicas be distinguished?
- Which source is authoritative for each resource or field?
- How should Glaux Server preserve provenance for imported or externally supplied descriptions?
- How should conflicting source updates be detected and resolved?
- How should DDIL re-synchronization handle stale, duplicated, conflicting, or superseded state changes?
- Which findings should be handed to ingestion, federation, DDIL, persistence, and audit topics?

#### Temporal, Freshness, and History Implications

- Which update operations affect valid time, transaction time, ingestion time, update time, publication time, event time, and freshness time?
- Which state changes require history retention?
- Which resource families require current-state views plus historical records?
- How should updates to deployment intervals, system descriptions, datastream definitions, command status, and status resources preserve history?
- Which findings should be handed to temporal modeling, time-series storage, status/event modeling, command lifecycle, and persistence topics?

#### Status, Event, and Notification Implications

- Which registration/update/state changes should generate system events?
- Which state changes should affect status or availability resources?
- Which events should be exposed through CSAPI Part 2 system event resources?
- Which changes should be notification-worthy for future pub/sub or streaming support?
- How should Glaux Server distinguish resource lifecycle events, system operational events, observation events, command events, and administrative events?
- Which findings should be handed to `IDR-SRV-020`, `IDR-SRV-035`, and `IDR-SRV-036`?

#### Security, Authorization, and Audit Implications

- Which operations require authentication and authorization?
- Which state changes are sensitive because they can affect operational behavior, command/control, data integrity, or cross-boundary releasability?
- Which update operations require approval, policy checks, command authorization, safety checks, or audit trails?
- How should Glaux Server prevent unauthorized discovery through create/update/delete response behavior?
- Which findings should be handed to `IDR-SRV-038`, `IDR-SRV-039`, `IDR-SRV-040`, and `IDR-SRV-055`?

#### Implementation and Interoperability Lessons

- What registration, update, and lifecycle lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate mutation, stale-state, lifecycle, update, or conflict-handling issues?
- What OS4CSAPI discussion lessons affect registration/update strategy?
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

- `IDR-SRV-001` through `IDR-SRV-018` research reports, once complete:
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

### HTTP, Validation, and Concurrency Sources

- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 - HTTP Caching: https://www.rfc-editor.org/rfc/rfc9111
- RFC 9112 - HTTP/1.1: https://www.rfc-editor.org/rfc/rfc9112
- RFC 5789 - PATCH Method for HTTP: https://www.rfc-editor.org/rfc/rfc5789
- RFC 6902 - JSON Patch: https://www.rfc-editor.org/rfc/rfc6902
- RFC 7386 - JSON Merge Patch: https://www.rfc-editor.org/rfc/rfc7386
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, or standards-package annexes relevant to registration, update, lifecycle, server responsibilities, operational state, and tasking

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

### Phase 1: Source Collection and Operation Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for registration, update, and state-change semantics.

**Tasks:**

1. Gather standards, schemas, OpenAPI artifacts, prior IDR reports, implementation studies, smoke-test findings, interoperability findings, and discussion lessons.
2. Extract all registration, creation, update, import, replacement, retirement, archival, deletion, state transition, validation, conflict, and audit concepts from each source.
3. Define operation inventory fields:
   - resource family,
   - operation/state transition,
   - source authority,
   - external/internal visibility,
   - HTTP method if applicable,
   - idempotency,
   - validation requirement,
   - conflict handling,
   - lifecycle state,
   - temporal implication,
   - event/status implication,
   - security/audit implication,
   - persistence implication,
   - validation/test implication.
4. Define classification values:
   - normative,
   - inherited,
   - standards-package-specific,
   - implementation-support,
   - implementation-specific,
   - optional,
   - administrative,
   - derived,
   - unresolved.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation observations, and community discussion.

**Expected Output:** Source inventory and operation/state-change extraction framework.

### Phase 2: Standards-Derived Operation and Lifecycle Extraction (2.5-3.5 hours)

**Objective:** Extract registration, update, and state-change expectations from standards and official artifacts.

**Tasks:**

1. Extract CSAPI Part 1 registration/update implications for feature resources.
2. Extract CSAPI Part 2 dynamic-data, command, feasibility, status, event, and control-stream state-change implications.
3. Extract inherited OGC API and HTTP method semantics relevant to create/update/delete/patch/conflict behavior.
4. Extract SensorML update and system-description implications.
5. Extract SWE Common validation/update implications.
6. Extract AEP-4789 operation and server-responsibility implications from project-available material.
7. Identify gaps where standards do not define mutation behavior but Glaux Server must still define implementation semantics.

**Expected Output:** Standards-derived operation and lifecycle inventory.

### Phase 3: Resource-Family Operation Semantics Analysis (3-4 hours)

**Objective:** Define registration and update behavior by canonical resource family.

**Tasks:**

1. Use `IDR-SRV-015` through `IDR-SRV-018` findings as the organizing baseline.
2. For each resource family, identify allowed, disallowed, required, administrative-only, internal, derived, or unresolved operations.
3. Identify lifecycle states and transitions associated with each operation.
4. Identify idempotency, validation, conflict, temporal, relationship, and status/event implications.
5. Identify unresolved operation questions requiring downstream research or prototype validation.

**Expected Output:** Resource-family operation semantics matrix.

### Phase 4: Conflict, Concurrency, Provenance, and DDIL Synchronization Analysis (2.5-3 hours)

**Objective:** Define how Glaux Server should reason about conflicting or repeated state changes.

**Tasks:**

1. Analyze duplicate identifiers, stale versions, invalid relationships, temporal conflicts, concurrent updates, and out-of-order data.
2. Analyze publisher, simulator, external-source, federated-source, manual-admin, and batch-import update patterns.
3. Analyze DDIL synchronization, replay, duplicate message, stale cache, and source authority implications.
4. Identify validation and conflict response behavior.
5. Identify handoffs to transaction/concurrency, ingestion, DDIL, security, and audit topics.

**Expected Output:** Conflict/provenance/synchronization implication matrix.

### Phase 5: Status/Event, Security, Persistence, Validation, and Test Implications (2.5-3 hours)

**Objective:** Prepare registration/update findings for downstream topics.

**Tasks:**

1. Identify which operations generate status changes, system events, audit records, notifications, or derived state changes.
2. Identify persistence, transaction, indexing, and retention implications without designing the database schema.
3. Identify validation rules for payloads, lifecycle transitions, relationships, temporal constraints, SensorML/SWE consistency, and authorization.
4. Identify security and policy risks from create/update/delete operations.
5. Identify conformance, golden-file, fixture, negative, idempotency, conflict, concurrency, DDIL, and interoperability tests.
6. Map findings to Category C, E, F, G, and I topics.

**Expected Output:** Downstream operation and state-change implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert registration, update, and state-change research into a decision-usable baseline.

**Tasks:**

1. Consolidate standards-derived inventory, resource-family operation semantics, lifecycle/state-transition guidance, conflict handling, provenance handling, and downstream implications.
2. Produce operation and state-change recommendations by resource family.
3. Resolve conflicts and ambiguities where possible.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Registration, creation, import, update, replacement, retirement, archival, deletion, and derived-state concepts are identified and distinguished.
- [ ] Resource-family operation semantics are mapped.
- [ ] Externally visible state changes and internal state changes are distinguished.
- [ ] Idempotency, validation, conflict handling, lifecycle transition, provenance, and source-authority implications are documented.
- [ ] Temporal, relationship, freshness, status/event, command, and audit implications are documented.
- [ ] Persistence, transaction, security, conformance, fixture, and interoperability implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Registration, Update, and State Change Semantics Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-019-registration-update-and-state-change-semantics-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Operation/state-change extraction methodology
5. Standards-derived operation and lifecycle inventory
6. Resource-family operation semantics
7. Lifecycle state and transition findings
8. Idempotency, conflict, concurrency, and validation findings
9. Provenance, source authority, federation, and DDIL synchronization findings
10. Temporal, relationship, freshness, and status/event implications
11. Security, authorization, and audit implications
12. Persistence, transaction, fixture, and test implications
13. Downstream topic handoff matrix
14. Recommendations
15. Risks, constraints, and open questions
16. Validation against this plan's success criteria
17. References

The registration/update matrix should include, at minimum:

- Resource family
- Operation or state transition
- Source standard / source anchor
- Authority classification
- External/internal visibility
- HTTP method or interaction pattern, if applicable
- Idempotency expectation
- Validation requirement
- Conflict handling
- Lifecycle state impact
- Temporal/freshness impact
- Relationship impact
- Status/event impact
- Security/audit implication
- Persistence implication
- Test implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-018` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, OGC schemas, OpenAPI artifacts, HTTP/PATCH/problem-details standards, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-020: Status, Availability, and System Event Model`
- `IDR-SRV-025: Database and Persistence Architecture Options`
- `IDR-SRV-027: Time-Series Observation Storage Strategy`
- `IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy`
- `IDR-SRV-030: Configuration, Secrets, and Environment Management`
- `IDR-SRV-033: Dynamic Data Ingestion and Normalization Pipeline`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints`
- `IDR-SRV-041: DDIL Behavior, Caching, and Synchronization Semantics`
- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
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

- This topic defines registration, update, and state-change semantics, not the database schema or final implementation API surface.
- This topic should distinguish full-scope capability from implementation sequencing.
- Implementation-study findings are useful but must not override standards-derived semantics.
- Open question: Which resource families require externally exposed write operations versus administrative/import-only workflows?
- Open question: Which resources should be append-only versus update-in-place versus versioned?
- Open question: Which state changes should produce system events?
- Open question: Which operations require ETags, conditional requests, revisions, or explicit conflict responses?
- Open question: How should Glaux Server reconcile out-of-order publisher updates under DDIL conditions?
- Risk: Overexposing mutation operations could increase security, audit, and conformance complexity.
- Risk: Underdefining lifecycle transitions could lead to inconsistent resource history and broken clients.
- Risk: Treating implementation-specific update behavior as normative could distort Glaux Server design.
- Risk: Failing to distinguish descriptive updates from operational state changes could create audit and interoperability problems.

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
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 - HTTP Caching: https://www.rfc-editor.org/rfc/rfc9111
- RFC 9112 - HTTP/1.1: https://www.rfc-editor.org/rfc/rfc9112
- RFC 5789 - PATCH Method for HTTP: https://www.rfc-editor.org/rfc/rfc5789
- RFC 6902 - JSON Patch: https://www.rfc-editor.org/rfc/rfc6902
- RFC 7386 - JSON Merge Patch: https://www.rfc-editor.org/rfc/rfc7386
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

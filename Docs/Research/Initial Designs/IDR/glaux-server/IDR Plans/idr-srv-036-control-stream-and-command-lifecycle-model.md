# Section 036: Control Stream and Command Lifecycle Model - Research Plan

**Status:** Planned  
**Last Updated:** June 11, 2026  
**Estimated Research Time:** 14-18 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-036-control-stream-and-command-lifecycle-model-report.md`

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

This topic research must define the Glaux Server planning baseline for the **control stream and command lifecycle model** across OGC API - Connected Systems Part 2 tasking/control resources, control streams, commands, command parameters, command execution state, command status records, command results, cancellation, timeout, dispatch, command-capable systems, command-capable gateways, feasibility pre-checks, authorization/safety gates, audit records, event publication, DDIL behavior, and client-facing command interaction semantics.

The research must answer:

- What are the precise semantics of control streams, commands, command payloads, command status, command result, command cancellation, and command failure in Glaux Server?
- How should Glaux Server model command-capable systems, control streams, command definitions, command parameters, controlled properties, allowed values, constraints, encodings, and target systems?
- What is the relationship between a control stream, a command resource, a feasibility evaluation, a command dispatch operation, a command-capable source/gateway, and a command-status update?
- What lifecycle states and transitions must Glaux Server support from creation through validation, feasibility, authorization, acceptance, dispatch, execution, completion, failure, cancellation, timeout, archival, and audit?
- Which lifecycle transitions must be synchronous, asynchronous, transactional, event-published, auditable, policy-controlled, or DDIL-aware?
- How should command lifecycle behavior interact with source registration/trust, ingestion, streaming/event publication, persistence, transaction/idempotency, security/policy, SensorML/SWE Common definitions, and AEP-4789 server responsibilities?

The output must be a control stream and command lifecycle model with source anchors, resource/concept definitions, state-transition model, command data model guidance, dispatch/status semantics, event/audit implications, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows `IDR-SRV-035: Streaming and Event Publication Strategy`. The previous dynamic-data topics define ingestion, observation/status semantics, and event publication. Command/control behavior depends on all three: commands are API resources, command-status updates may be ingested from gateways, command lifecycle transitions may generate events, and command results may be persisted and streamed. This topic defines the neutral command lifecycle model before the next topics specialize feasibility/validation and command authorization/safety/audit.

### Critical Constraints

- Treat OGC API - Connected Systems Part 2, CSAPI Part 1, SensorML 3.0, SWE Common 3.0, transaction/idempotency findings, source trust findings, ingestion findings, event publication findings, AEP-4789 server responsibilities, and prior IDR findings as controlling.
- Do not finalize command authorization, safety policy, or audit strategy here. Identify lifecycle hooks and hand detailed safety/security work to `IDR-SRV-038` and Category G.
- Do not finalize feasibility evaluation here. Identify lifecycle relationship to feasibility and hand detailed feasibility behavior to `IDR-SRV-037`.
- Do not treat command submission as simple observation ingestion. Commands have target effects, authority requirements, lifecycle states, safety implications, and audit requirements.
- Do not assume all command execution is synchronous or always connected. Support asynchronous dispatch, delayed status, gateway-mediated execution, cancellation, timeout, stale status, and DDIL behavior.
- Do not assume all command-capable systems are directly reachable by Glaux Server. Distinguish direct control, gateway-mediated control, brokered control, simulated control, and administrative/test-only control.
- Do not claim a command was executed solely because it was accepted by Glaux Server. Distinguish accepted, dispatched, acknowledged, executing, completed, failed, cancelled, expired, and unknown states.
- Keep the research bounded to Glaux Server behavior and server-side control/command contracts.

---

## 2. Research Questions

### Core Questions

1. What are the standards-aligned meanings of control streams, commands, command payloads, command status, command result, command cancellation, and command failure?
2. What command lifecycle states and transitions must Glaux Server support?
3. How should command lifecycle state interact with feasibility checks, authorization/safety gates, source trust, transaction/idempotency, dispatch, event publication, audit, and DDIL behavior?
4. What persistence, query, latest-status, history, event, and client-facing behavior is required for commands and control streams?
5. What downstream implications follow for feasibility validation, command safety/audit, security, DDIL, fixtures, conformance, performance, and interoperability?

### Detailed Questions

#### Standards and Tasking/Control Baseline

- What control stream, command, command status, feasibility, and tasking concepts are defined by CSAPI Part 2?
- Which CSAPI Part 1 resources are required to interpret control streams and commands?
- How do SensorML controllable properties, inputs, parameters, capabilities, characteristics, constraints, and process descriptions relate to command definitions?
- How do SWE Common data components, choices, records, arrays, constraints, encodings, allowed values, nil values, and units relate to command payloads?
- What AEP-4789 Volume II expectations shape tasking/control semantics for NATO JISR sensor integration?
- Which command/control behaviors are normative, profile-driven, operationally required, or implementation-defined?

#### Control Stream Semantic Model

- What is a control stream in Glaux Server?
- How should a control stream relate to a system, procedure, deployment, controlled property, SensorML input/parameter, SWE Common command parameter structure, command definition, command-capable source/gateway, feasibility resource, authorization/safety policy, and command event stream?
- Which control stream metadata is required, optional, derived, source-provided, or server-assigned?
- How should control stream lifecycle changes affect command submission, feasibility, authorization, event publication, and client visibility?
- How should control streams differ from datastreams?

#### Command Resource Semantic Model

- What is a command resource in Glaux Server?
- What is the difference between a command definition, command request, command resource, command execution, command status, command result, feasibility result, command audit record, and command event?
- What fields must a command include: identifier, target system/control stream, command type, parameters, issuer/client, source/gateway, requested execution time, validity/expiration time, feasibility reference, authorization decision, safety decision, lifecycle state, status history, result, provenance, policy labels, and audit correlation?
- Which fields are client-supplied, server-assigned, gateway-supplied, derived, or policy-internal?

#### Command Lifecycle State Model

- What lifecycle states should be supported: draft/prepared, submitted, received, validation pending, validation failed, feasibility pending, feasible, infeasible, authorization pending, authorized, rejected, accepted, queued, dispatched, acknowledged, executing, completed, failed, cancellation requested, cancelled, expired, timed out, unknown, superseded, and archived?
- Which states are externally visible and which are internal-only?
- Which transitions are allowed, terminal, retryable, asynchronous, or operator-mediated?
- Which transitions require audit records?
- Which transitions generate system events or command events?
- Which transitions require human/operator action?

#### Command Submission and Acceptance Semantics

- What happens when a command is submitted?
- Which checks occur before a command resource is created?
- Which checks occur after creation but before acceptance?
- What is the difference between server receipt, validation, acceptance, dispatch, and execution?
- When should command submission return synchronous success, asynchronous accepted status, validation error, feasibility failure, authorization failure, safety rejection, conflict, or unavailable status?
- How should command submission use idempotency keys, client request identifiers, command identifiers, and retry behavior?

#### Feasibility Relationship

- How should feasibility evaluation relate to command lifecycle?
- Is feasibility required before command submission, optional, embedded, cached, or separate?
- How should stale feasibility results be handled?
- How should feasibility results link to command resources?
- How should feasibility failures differ from validation failures, authorization failures, safety failures, and execution failures?
- Which findings should be handed to `IDR-SRV-037`?

#### Validation, Parameter, and Encoding Semantics

- How should command parameter structures be validated against SWE Common and SensorML-derived definitions?
- How should command payloads handle units, controlled properties, allowed values, constraints, nil values, choices, arrays, records, and encodings?
- Which validation failures are client errors versus internal/gateway errors?
- How should validation artifacts be stored?
- How should command parameters be represented for clients, persisted, and dispatched to gateways?
- How should profile-specific command definitions be handled?

#### Dispatch and Execution Boundary

- How should Glaux Server dispatch commands: direct API call to target, command gateway, adapter, broker/topic, queued execution, local simulator, or manual/operator mediated execution?
- Which dispatch patterns are first-implementation candidates versus full-scope readiness candidates?
- How should dispatch state be persisted?
- How should dispatch failures differ from execution failures?
- How should dispatch credentials, source/gateway trust, and command authority be represented conceptually?
- Which findings should be handed to `IDR-SRV-038`, deployment, and security topics?

#### Command Status and Result Semantics

- Who may report command status?
- How should command status records be ingested, validated, trusted, persisted, queried, and published?
- How should command result data be represented?
- How should command status differ from system status and source health?
- How should command status history relate to latest command state?
- How should delayed, duplicate, out-of-order, or conflicting status records be handled?
- Which command statuses should be event-published?

#### Cancellation, Timeout, Expiration, and Failure

- How should cancellation be requested, authorized, dispatched, acknowledged, completed, or denied?
- How should command timeout differ from command expiration?
- How should execution failure differ from validation failure, authorization failure, feasibility failure, dispatch failure, and unknown outcome?
- How should Glaux Server represent commands with unknown outcome after DDIL disconnect or gateway failure?
- Which terminal states require audit, policy review, or operator intervention?

#### Persistence, Query, and History

- What command and control stream records must be stored: control stream definitions, command definitions, command resources, parameter payloads, lifecycle state, status history, result records, feasibility links, dispatch records, gateway acknowledgements, events, and audit records?
- Which records are immutable, updateable, or append-only?
- How should commands be queried by target system, control stream, issuer, status, time, source/gateway, feasibility, and policy scope?
- How should pagination and sorting behave under concurrent status updates?

#### Event Publication and Streaming Interaction

- Which command lifecycle transitions should generate durable events?
- Which command streams should clients be able to subscribe to?
- How should command publication differ from observation/status publication?
- How should command events be filtered by authorization and policy?
- How should event replay handle command histories?
- Which findings should be handed to `IDR-SRV-035` and security topics?

#### DDIL and Asynchronous Operation

- How should command lifecycle behave during disconnected, degraded, intermittent, or limited-bandwidth operation?
- Can commands be queued while disconnected?
- How should command validity windows and expiration interact with DDIL?
- How should delayed command status updates be reconciled after reconnect?
- How should unknown outcomes be represented?
- Which command operations should be disabled or constrained in DDIL mode?
- Which audit findings should be handed to `IDR-SRV-041`, DDIL behavior to `IDR-SRV-042`, and synchronization/conflict behavior to `IDR-SRV-043`?

#### Security, Authorization, Safety, and Audit Hooks

- Where in the lifecycle are authentication, authorization, policy, safety, releasability, and audit checks required?
- What lifecycle hooks must be available for downstream command safety and audit strategy?
- What command information is sensitive: command type, target, timing, parameters, capability, issuer, gateway, status, and result?
- How should public responses avoid leaking command affordances or target capability?
- Which findings should be handed to `IDR-SRV-038`, `IDR-SRV-039`, `IDR-SRV-040`, and `IDR-SRV-055`?

#### Fixtures, Conformance, Performance, and Interoperability

- What command fixtures are needed: valid/invalid control streams, valid/invalid command definitions, invalid parameters, feasible/infeasible command, authorized/rejected command, queued/dispatched/executing/completed/failed/cancelled/timed-out command, duplicate command, stale feasibility command, DDIL delayed status command, and simulated command gateway?
- What conformance tests should verify command lifecycle behavior?
- What performance/load/stress tests are needed for command submission, status updates, event publication, authorization hooks, and command-history queries?
- What interoperability tests are needed for CSAPI Explorer, OS4CSAPI clients, webapp/mobile clients, command-capable gateways, and external CSAPI clients?

#### Implementation and Interoperability Lessons

- What command/control lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate control stream, command lifecycle, feasibility, command-status, command event, or command payload issues?
- What OS4CSAPI discussion lessons affect control stream and command lifecycle strategy?
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

- `IDR-SRV-001` through `IDR-SRV-035` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Controlling Standards and API Sources

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/
- OGC schemas: https://schemas.opengis.net/
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457

### Command, Workflow, Reliability, and Messaging Candidate Sources

Use current official documentation and primary-source technical material when executing the research. Candidate sources include, but are not limited to:

- CloudEvents specification: https://cloudevents.io/
- MQTT v5 specification: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- NATS documentation: https://docs.nats.io/
- Apache Kafka documentation: https://kafka.apache.org/documentation/
- PostgreSQL transaction documentation: https://www.postgresql.org/docs/current/tutorial-transactions.html
- Rust Tokio documentation: https://docs.rs/tokio/
- Rust SQLx documentation: https://docs.rs/sqlx/
- Rust serde documentation: https://serde.rs/
- Rust tracing documentation: https://docs.rs/tracing/

These are candidates for evaluation, not assumed selections.

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, tasking guidance, command/control guidance, feasibility guidance, safety guidance, DDIL guidance, or standards-package annexes relevant to control streams, commands, and NATO JISR sensor tasking

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
- Category F findings from `IDR-SRV-031` through `IDR-SRV-035`
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

### Phase 1: Source Collection and Command Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for control stream and command lifecycle research.

**Tasks:**

1. Gather prior IDR reports, standards sources, implementation studies, command/workflow/reliability sources, and project architecture sources.
2. Extract control stream, command, feasibility, status, event, transaction, authorization, and DDIL concepts from each source.
3. Define inventory fields: concept/resource type, related CSAPI resource, semantic definition, required relationships, lifecycle state, transition trigger, validation/feasibility/authorization hook, persistence/event/audit requirement, security/policy implication, and downstream handoff.
4. Define evaluation criteria: standards alignment, lifecycle clarity, command safety, client interoperability, gateway compatibility, transaction/idempotency support, DDIL suitability, auditability, fixture/testability, and operational complexity.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation evidence, and technology documentation.

**Expected Output:** Command lifecycle extraction framework and evaluation rubric.

### Phase 2: Control Stream and Command Concept Inventory (3-4 hours)

**Objective:** Determine the core command/control concepts Glaux Server must distinguish.

**Tasks:**

1. Inventory CSAPI Part 2 control stream, command, feasibility, and command status concepts.
2. Map supporting SensorML, SWE Common, system, deployment, source, and policy concepts.
3. Distinguish control streams, datastreams, command definitions, command requests, command resources, command executions, command statuses, command results, feasibility results, audit records, and command events.
4. Identify required fields, optional fields, derived fields, and server-assigned fields.
5. Build a command/control concept taxonomy.

**Expected Output:** Control stream and command concept taxonomy.

### Phase 3: Command Lifecycle State and Transition Analysis (3-4 hours)

**Objective:** Define command lifecycle behavior.

**Tasks:**

1. Inventory lifecycle states and allowed transitions.
2. Identify synchronous versus asynchronous transitions.
3. Identify terminal, retryable, recoverable, unknown, and operator-mediated states.
4. Identify validation, feasibility, authorization, safety, dispatch, cancellation, timeout, and result hooks.
5. Identify event, persistence, audit, and query implications for each transition.
6. Identify unresolved questions requiring prototype validation or operational review.

**Expected Output:** Command lifecycle state-transition matrix.

### Phase 4: Payload, Dispatch, Status, Result, and DDIL Analysis (3-4 hours)

**Objective:** Define command data and execution interactions.

**Tasks:**

1. Analyze command parameter structures, SWE Common constraints, units, encodings, and validation artifacts.
2. Analyze dispatch patterns: direct, gateway-mediated, brokered, queued, simulated, and manual/operator mediated.
3. Analyze command status and result reporting.
4. Analyze cancellation, timeout, expiration, failure, and unknown outcome behavior.
5. Analyze DDIL, delayed status, replay, reconnection, stale feasibility, and offline command constraints.
6. Map findings to feasibility, safety/audit, DDIL, and deployment topics.

**Expected Output:** Command payload, dispatch, status, result, and DDIL matrix.

### Phase 5: Security, Fixtures, Conformance, Performance, and Interoperability Analysis (3-4 hours)

**Objective:** Prepare command lifecycle findings for downstream implementation and verification.

**Tasks:**

1. Identify authorization, policy, safety, releasability, and audit hooks.
2. Identify command fixtures and simulated command gateway behaviors.
3. Identify conformance tests for command lifecycle behavior.
4. Identify performance/load/stress tests for command submission, status updates, event publication, authorization hooks, and command-history queries.
5. Identify interoperability tests with CSAPI Explorer, OS4CSAPI clients, webapp/mobile clients, command-capable gateways, and external CSAPI clients.
6. Map findings to Category G, H, and I topics.

**Expected Output:** Command downstream verification and security implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert command lifecycle research into a decision-usable baseline.

**Tasks:**

1. Consolidate concept taxonomy, control stream model, command resource model, lifecycle state model, payload/dispatch/status guidance, DDIL guidance, and downstream implications.
2. Produce recommended control stream and command lifecycle strategy with rationale and unresolved questions.
3. Identify sequencing for feasibility validation, command safety/audit, DDIL, security, deployment, fixture, conformance, and performance topics.
4. Produce downstream handoff matrix.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Control stream, command definition, command request, command resource, command execution, command status, command result, feasibility result, command event, and audit record concepts are identified and distinguished with source anchors.
- [ ] Control stream relationships to systems, controlled properties, SensorML, SWE Common, command gateways, feasibility, authorization/safety, and event streams are documented.
- [ ] Command lifecycle states, allowed transitions, terminal states, asynchronous states, cancellation, timeout, failure, and unknown-outcome behavior are documented.
- [ ] Command payload, parameter validation, dispatch, status reporting, persistence, query, event publication, DDIL, and provenance implications are documented.
- [ ] Authorization, safety, audit, policy/releasability, fixture, conformance, performance, and interoperability implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Control Stream and Command Lifecycle Model Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-036-control-stream-and-command-lifecycle-model-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Command lifecycle extraction methodology
5. Control stream and command concept taxonomy
6. Control stream semantic model findings
7. Command resource semantic model findings
8. Command lifecycle state-transition findings
9. Command submission, validation, acceptance, dispatch, execution, cancellation, timeout, failure, and unknown-outcome findings
10. Command payload, SWE Common, SensorML, units, constraints, and encoding findings
11. Command status, result, query, persistence, event publication, and replay findings
12. Feasibility relationship findings
13. DDIL, gateway, brokered dispatch, and asynchronous operation findings
14. Security, authorization, safety, policy, releasability, provenance, and audit implications
15. Fixture, conformance, performance, and interoperability test implications
16. Downstream topic handoff matrix
17. Recommendations
18. Risks, constraints, and open questions
19. Validation against this plan's success criteria
20. References

The command lifecycle matrix should include, at minimum:

- Lifecycle state
- Transition trigger
- Source/client/gateway actor
- Required validation
- Feasibility implication
- Authorization/safety implication
- Persistence requirement
- Event/publication requirement
- Audit requirement
- Client-visible status
- Retry/cancel/timeout behavior
- DDIL implication
- Security/policy implication
- Test/conformance implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-035` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, SensorML, SWE Common, relevant OGC schemas/OpenAPI artifacts, command/workflow/reliability sources, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Implementation-study and interoperability findings must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-037: Feasibility and Asynchronous Tasking Strategy`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints`
- `IDR-SRV-041: Audit Logging and Accountability Strategy`
- `IDR-SRV-042: DDIL-Informed Server Semantics`
- `IDR-SRV-043: Server Synchronization and Conflict Handling Boundary`
- `IDR-SRV-045: Service Architecture and Modularization Strategy`
- `IDR-SRV-048: Observability, Logs, Metrics, and Health Check Strategy`
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

- This topic defines control stream and command lifecycle semantics, not final authorization/safety implementation.
- A command accepted by Glaux Server must not be equated with successful execution by the target system.
- Feasibility, authorization, safety, dispatch, and execution failures must remain semantically distinct.
- Implementation-study findings are useful but must not override standards-derived server responsibilities or safety-critical requirements.
- Open question: Which lifecycle states are required for first implementation versus full-scope readiness?
- Open question: Should command resources be created before or after feasibility/authorization checks?
- Open question: Which command dispatch patterns should be first implementation candidates?
- Open question: How should unknown command outcome be represented after DDIL disconnect or gateway failure?
- Open question: Which command events should be durable and replayable?
- Risk: Oversimplifying command lifecycle could create unsafe operational ambiguity.
- Risk: Treating commands like observations could weaken authorization, audit, and safety handling.
- Risk: Poor cancellation/timeout semantics could misrepresent target-system state.
- Risk: Exposing command affordances or command status without policy controls could leak sensitive capabilities.

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
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/
- OGC schemas: https://schemas.opengis.net/
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- CloudEvents specification: https://cloudevents.io/
- MQTT v5 specification: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- NATS documentation: https://docs.nats.io/
- Apache Kafka documentation: https://kafka.apache.org/documentation/
- PostgreSQL transaction documentation: https://www.postgresql.org/docs/current/tutorial-transactions.html
- Rust Tokio documentation: https://docs.rs/tokio/
- Rust SQLx documentation: https://docs.rs/sqlx/
- Rust serde documentation: https://serde.rs/
- Rust tracing documentation: https://docs.rs/tracing/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

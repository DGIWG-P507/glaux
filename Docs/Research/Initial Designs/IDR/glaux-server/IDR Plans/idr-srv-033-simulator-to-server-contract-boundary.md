# Section 033: Simulator-to-Server Contract Boundary - Research Plan

**Status:** Planned
**Last Updated:** July 29, 2026
**Estimated Research Time:** 13-17 hours
**Actual Research Time:** TBD until complete
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-033-simulator-to-server-contract-boundary-report.md`

---

## Usage Instructions

Before executing this plan, review the exemplar corpus and align the report to the proven standards for content nature, type, level, and detail:

- Full research-plans corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- Blueprint-first exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md
- Inventory/sourcing rigor exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/15-fixture-sourcing-organization.md
- End-state synthesis exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/38-testing-playbook-synthesis.md

This plan is limited to research planning. It does not execute the research and does not produce the research report.

The resulting report must be polished, recommendation-first, independently readable, and usable as shared decision material by the project lead, implementers, and later AI agents. It must supply enough context to stand on its own, synthesize the evidence instead of mirroring the research questions, and clearly state findings, recommendations, implementation implications, and unresolved questions.

---

## 1. Research Objective

This topic research must define the **simulator-to-server contract boundary** from the Glaux Server perspective. It must determine how Glaux Simulator and other authorized simulation/test clients may create synthetic scenarios, publish simulated metadata and dynamic data, replay deterministic datasets, manipulate simulation time, exercise tasking loops, inspect progress, and reset simulator-owned state without weakening the production write model or standards-facing API.

The research must specialize:

- `IDR-SRV-031: Server Write and Ingestion Model`, which defines the authoritative server write path; and
- `IDR-SRV-032: Publisher-to-Server Contract Boundary`, which defines the general external publication contract.

It must answer:

- Which simulator interactions can use the ordinary publisher/CSAPI contract, and which require a separate test-control plane?
- How are simulation session, scenario, dataset, seed, virtual clock, run, source, publisher, and generated-resource identities represented and correlated?
- How should deterministic replay, pause/resume/step, rate changes, restart, backfill, out-of-order delivery, fault injection, and tasking responses appear at the server boundary?
- What does reset mean, exactly which simulator-owned state may it affect, and what authorization, isolation, preview, audit, and recovery safeguards are mandatory?
- How does Glaux Server distinguish simulated or synthetic content from operational content while still applying realistic validation, policy, persistence, event, and conformance behavior?
- What acknowledgements, progress/status, errors, diagnostics, backpressure, and completion evidence make simulation runs reproducible and testable?

The output must be a simulator contract boundary baseline containing the data-plane/control-plane split, capability and operation inventory, simulation identity/context model, replay and virtual-time semantics, reset safety model, synthetic provenance rules, tasking-loop behavior, error/progress contract, isolation requirements, and verification scenarios.

### Why This Topic Order

This topic follows the general server write model and publisher contract because most simulator-generated records should traverse the same standards-aligned ingestion and validation path as live publisher data. It isolates only the simulator-specific behaviors—scenario control, deterministic replay, virtual time, reset, synthetic provenance, fault injection, and test orchestration—that would be unsafe or misleading in the general publisher contract.

The result must precede detailed datastream/observation/status semantics, streaming, command lifecycle, feasibility, command safety/audit, DDIL behavior, fixture design, performance testing, and interoperability testing because those topics need a stable way to drive and reset reproducible server scenarios.

### Critical Constraints

- Keep the research bounded to Glaux Server behavior and the wire-visible contract. Do not design Glaux Simulator internals, its UI, scenario-authoring tools, or operator workflows.
- Reuse the `IDR-SRV-031` processing pipeline and `IDR-SRV-032` publisher contract for the simulator data plane wherever possible.
- Keep simulator control operations separate from standards conformance resources unless a controlling standard explicitly defines them.
- Never assume a reset means deleting all server data. Reset must be bounded to explicitly owned scenario/session state and must be denied when safe scope cannot be proven.
- Production deployment must not expose simulation/reset capabilities by default. Research explicit enablement, environment restrictions, least privilege, and auditable authorization.
- Synthetic data must not become indistinguishable from operational data. Preserve scenario, generator, seed, clock mode, source, transformation, and run provenance where required.
- Do not weaken validation simply because a source is a simulator. Invalid-data and fault-injection modes must be explicit, isolated test cases with observable expected outcomes.
- Distinguish replaying original records from regenerating equivalent synthetic records and from shifting their timestamps.
- Do not finalize dynamic-data meaning, streaming protocols, command lifecycle, security architecture, DDIL synchronization, or test-framework selection here; identify requirements for their controlling topics.

---

## 2. Research Questions

### Core Questions

1. What simulator data-plane and control-plane operations must Glaux Server support, and which existing write/publisher operations can satisfy them?
2. What identity and provenance model makes simulation sessions, scenarios, datasets, runs, clocks, sources, and generated records reproducible and safely isolated?
3. What are the precise server-side semantics for replay, reset, restart, pause/resume/step, rate changes, backfill, fault injection, and completion?
4. How should simulated observations, status, events, metadata, source health, feasibility results, and command responses traverse normal validation, persistence, streaming, policy, and audit behavior?
5. What safeguards and verification evidence prevent simulator controls or synthetic records from damaging or being confused with operational state?

### Detailed Questions

#### Simulator Role and Contract Boundary

- Is a simulator a publisher role, a privileged test-controller role, or two separately authenticated principals?
- Which actions belong to the ordinary data plane and which require a distinct control plane?
- Can scenario control be implemented outside Glaux Server by orchestration that uses normal APIs, or does the server need explicit session/reset support?
- Which capabilities are required for local development, CI, demonstrations, conformance testing, interoperability testing, performance testing, and operational training environments?
- Which proposed operations are standards-derived, test-harness requirements, or Glaux-specific conveniences?

#### Capability Discovery and Session Lifecycle

- How does an authorized simulator discover supported data classes, control capabilities, size/rate limits, time modes, reset scopes, contract versions, and disabled features?
- What lifecycle states are needed for a simulation session or run: created, prepared, running, paused, draining, completed, failed, cancelled, resetting, reset, and expired?
- Which transitions are simulator requested versus server controlled?
- What idempotency and concurrency rules apply to session creation and control transitions?
- How are abandoned sessions, expired leases, client disconnects, and server restarts handled?

#### Scenario, Dataset, Run, and Resource Identity

- How are scenario definition, dataset/version, seed, run ID, session ID, simulator instance, represented source, and generated resource IDs distinguished?
- Which identifiers are supplied by the simulator versus assigned by Glaux Server?
- How can deterministic runs reproduce identifiers without colliding with another run or operational resource?
- Are resources pre-provisioned, created during setup, namespaced by session, mapped to existing references, or forbidden from crossing isolation boundaries?
- What ownership evidence allows later reset or cleanup of a scenario's resources?

#### Data-Plane Submission

- Which simulated systems, procedures, deployments, sampling features, datastreams, SensorML/SWE descriptions, observations, status updates, events, health messages, feasibility results, and command responses may be submitted?
- Which ordinary publisher envelope fields remain required, and what additional simulation context is needed?
- Must the same schema, semantic, referential, unit, policy, idempotency, and transaction checks apply as for non-simulated traffic?
- How are intentionally malformed, unauthorized, stale, duplicated, or policy-invalid test records submitted without bypassing the boundary under test?
- Can simulated and operational sources coexist in one deployment, and what hard isolation rules prevent accidental cross-linking or contamination?

#### Replay and Determinism

- What is the replay source: canonical export, raw captured payload, fixture corpus, scenario definition, event log, or generated stream?
- Is replay exact-byte, semantically equivalent, identifier-stable, or newly generated?
- How are original message IDs, sequence numbers, source offsets, resource IDs, event IDs, and idempotency keys preserved, remapped, or regenerated?
- How does the server distinguish deliberate duplicate-rejection tests from a new replay run?
- What manifest, checksums, ordering rules, expected outcomes, and completion markers make a replay independently reproducible?
- How are interrupted runs resumed without silently skipping or duplicating records?

#### Virtual Time and Temporal Mapping

- Which time concepts may be simulated: source time, phenomenon time, result time, valid time, ingest time, publication time, and command timing?
- Which server-controlled times must always reflect actual receipt or commit time?
- What modes are needed: preserve original timestamps, shift to a run epoch, scale time, pause, step, burst, backfill, or inject clock skew?
- What happens to ordering, freshness, staleness, retention, latest-value selection, time-window queries, event publication, and timeouts when virtual time differs from wall time?
- Must virtual-time context be persisted or returned to clients, and how is it prevented from altering unrelated sessions?

#### Reset, Cleanup, and Recovery

- What reset scopes are required: uncommitted run state, one run, one session, simulator-owned resources, generated dynamic records, materialized latest values, event/outbox state, or a complete isolated test namespace?
- Which states must never be reset through the simulator contract, including operational or shared resources?
- How is ownership proven transitively for relationships and derived artifacts?
- Must reset support dry run/preview, confirmation token, preconditions, asynchronous progress, cancellation, retry, and an immutable audit record?
- What is the atomicity model for reset, and how are partial cleanup or server failure recovered?
- When is disposable environment recreation safer than an application-level reset endpoint?

#### Tasking and Closed-Loop Simulation

- How may a simulator emulate a controlled system that receives commands and returns feasibility, acknowledgement, execution status, result, failure, or timeout behavior?
- Which command/control actions use ordinary CSAPI resources or streams, and which simulator controls remain outside the standards surface?
- How are scripted latency, rejection, invalid transition, duplicate response, out-of-order response, loss, and disconnect scenarios represented?
- How are simulator authority and command target authority kept separate?
- Which decisions must be handed to `IDR-SRV-036`, `IDR-SRV-037`, and `IDR-SRV-038`?

#### Fault Injection and DDIL Scenarios

- Which faults may be requested safely: delay, duplication, reordering, drop, malformed payload, invalid semantics, clock skew, disconnect, reconnect, throttling, and partial batch failure?
- Should faults be generated by the simulator before submission, by a test proxy, or by explicit server test hooks?
- How are invalid-data scenarios isolated and prevented from disabling ordinary safeguards?
- How should store-and-forward, backlog replay, epoch changes, and synchronization conflicts be exercised?
- Which audit evidence belongs to `IDR-SRV-041`, which server-visible degraded behavior belongs to `IDR-SRV-042`, and which replay or synchronization-conflict behavior belongs to `IDR-SRV-043`?

#### Authentication, Authorization, Policy, and Isolation

- What credentials and scopes distinguish simulator publishing, scenario control, reset, fault injection, and tasking emulation?
- What environment, tenant/mission, namespace, network, and configuration gates are required?
- How is simulation capability disabled or absent in production by default?
- How are synthetic markings and releasability/policy labels validated and propagated?
- Which control actions and data mutations require audit records, actor attribution, reason, and correlation to a run?
- What error detail is safe to expose to the simulator without leaking unrelated resource existence or policy?

#### Progress, Errors, Observability, and Backpressure

- What synchronous response versus asynchronous operation/status pattern applies to setup, replay, drain, reset, and large imports?
- What counters and checkpoints expose submitted, accepted, rejected, quarantined, duplicate, committed, published, pending, and failed records?
- How are partial failure and per-record outcomes correlated with the replay manifest?
- What rate, concurrency, batch, queue, and storage limits apply, and how does the server communicate backpressure?
- Which logs, traces, metrics, audit records, and final run summaries are required for deterministic diagnosis?

#### Fixtures, Conformance, and Interoperability

- What minimal scenario corpus covers metadata setup, dynamic data, status/events, replay, reset, temporal modes, DDIL, tasking, invalid submissions, and recovery?
- What preconditions, actions, expected responses, observable persisted state, emitted events, and cleanup assertions define each scenario?
- Which tests validate standards-facing behavior and which validate only the Glaux simulator contract?
- How can scenario manifests and expected outcomes be shared with Glaux Publisher, CSAPI clients, conformance harnesses, and external implementations?
- Which implementation patterns from OSH, Connected Systems Go, 52North/pygeoapi, SECD evidence, and OS4CSAPI testing should be adopted, avoided, or investigated?

---

## 3. Primary Resources

The future report must record exact versions, commits, publication status, access dates, and any controlled-document handling constraints.

### Project and Prior-IDR Sources

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- `IDR-SRV-031: Server Write and Ingestion Model` report
- `IDR-SRV-032: Publisher-to-Server Contract Boundary` report
- Relevant `IDR-SRV-001` through `IDR-SRV-030` reports, especially resources, time, validation, identity, storage, lifecycle, and transactions
- Project-controlled STANAG 4789 and AEP-4789 Volumes I and II; record exact edition, date, status, controlled path, and handling constraints
- Glaux Simulator repository, design material, or interface definitions if available: https://github.com/DGIWG-P507/glaux-simulator

### Standards and Protocol Sources

- OGC API - Connected Systems - Part 1: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Common - Part 1: https://docs.ogc.org/is/19-072/19-072.html
- OGC API - Features - Part 4 draft CRUD specification: https://docs.ogc.org/DRAFTS/20-002.html
- Versioned CSAPI Part 1 OpenAPI artifacts: https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/openapi/
- Versioned CSAPI Part 2 OpenAPI artifacts: https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/openapi/
- OGC SensorML 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common 3.0: https://docs.ogc.org/is/24-014/24-014.html
- HTTP Semantics, RFC 9110: https://www.rfc-editor.org/rfc/rfc9110.html
- Problem Details for HTTP APIs, RFC 9457: https://www.rfc-editor.org/rfc/rfc9457.html

### Implementation and Test Evidence

- OGC API - Connected Systems editor repository, pinned to the reviewed commit: https://github.com/opengeospatial/ogcapi-connected-systems
- `IDR-SRV-014A` through `IDR-SRV-014G` implementation-study and interoperability reports
- OpenSensorHub sources: https://github.com/opensensorhub/osh-core
- Connected Systems Go sources: https://github.com/OS4CSAPI/connected-systems-go
- 52North Connected Systems pygeoapi proof of concept: https://github.com/52North/connected-systems-pygeoapi
- OS4CSAPI client test and fixture evidence: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability evidence, subject to access and provenance constraints: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/

Implementation behavior and test discussions are non-normative evidence. The report must separate observed behavior from standards obligations and Glaux-specific test-contract decisions.

---

## 4. Supporting Resources

- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- MQTT Version 5.0, for delivery and session comparisons: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- CloudEvents specification, for event-envelope comparison: https://github.com/cloudevents/spec
- OpenTelemetry documentation, for trace and run correlation: https://opentelemetry.io/docs/
- Testcontainers for Rust, as a candidate disposable-environment pattern: https://rust.testcontainers.org/
- Docker documentation, as a candidate environment-reset mechanism: https://docs.docker.com/

These are comparison sources, not assumed Glaux selections.

---

## 5. Research Methodology

### Phase 1: Evidence and Use-Case Baseline (1.5-2 hours)

**Objective:** Establish the authoritative inputs and bounded simulator use cases.

**Tasks:**

1. Extract the general write, publisher, validation, identity, storage, time, transaction, and lifecycle decisions from prior reports.
2. Inventory simulator needs for development, CI, demonstrations, conformance, interoperability, performance, security, DDIL, and command testing.
3. Classify each need as ordinary data-plane use, simulator control, external orchestration, or unsafe/out of scope.
4. Record source gaps and separate standards obligations from Glaux-specific testing requirements.

**Expected Output:** Source-anchored simulator capability and use-case inventory.

### Phase 2: Contract, Identity, and Isolation Model (3-4 hours)

**Objective:** Define the simulator's server-visible roles and safe contract surface.

**Tasks:**

1. Decide the data-plane/control-plane boundary and map ordinary submissions to `IDR-SRV-031` and `IDR-SRV-032`.
2. Model scenario, dataset, run, session, simulator, source, principal, clock, and generated-resource identities.
3. Define capability discovery, session lifecycle, authorization scopes, environment gates, ownership, synthetic marking, and audit requirements.
4. Test the model against concurrent runs, shared resources, production deployment, abandoned sessions, and unauthorized control requests.

**Expected Output:** Contract-surface decision, identity/context model, lifecycle, and isolation matrix.

### Phase 3: Replay, Time, Reset, and Recovery Analysis (3-4 hours)

**Objective:** Define the high-risk simulator operations precisely.

**Tasks:**

1. Compare exact replay, regenerated replay, identifier mapping, resume/checkpoint, and duplicate-test modes.
2. Define virtual-time modes and effects on source, domain, ingestion, publication, timeout, freshness, and retention times.
3. Define reset scopes, ownership proof, dependency closure, preview, authorization, transaction, progress, failure, retry, audit, and postcondition semantics.
4. Compare application-level cleanup with disposable environment recreation and identify where each is appropriate.

**Expected Output:** Replay state model, time-mapping matrix, and reset safety model.

### Phase 4: Dynamic Data, Tasking, Fault, and DDIL Analysis (2-3 hours)

**Objective:** Ensure simulator traffic exercises realistic server behavior without bypassing safeguards.

**Tasks:**

1. Map simulated metadata, observations, status, events, health, feasibility, and command responses through the ordinary ingestion path.
2. Define bounded fault-injection and invalid-data modes with expected server outcomes.
3. Analyze tasking-loop, disconnect/reconnect, backlog, replay, and synchronization interactions.
4. Assign unresolved semantics to `IDR-SRV-034` through `IDR-SRV-038`, security, or DDIL topics.

**Expected Output:** Dynamic-data/tasking scenario matrix and cross-topic handoff register.

### Phase 5: Progress, Verification, and Interoperability Analysis (2-2.5 hours)

**Objective:** Make simulator interactions deterministic, diagnosable, and testable.

**Tasks:**

1. Define acknowledgement, asynchronous progress, counters, checkpoints, final summaries, error, correlation, observability, and backpressure behavior.
2. Define a minimal scenario manifest and expected-outcome model.
3. Build positive, negative, reset-safety, crash-recovery, replay, temporal, DDIL, performance, security, and command-loop scenarios.
4. Compare the proposed boundary with pinned implementation and OS4CSAPI test evidence and distinguish conformance from Glaux-specific tests.

**Expected Output:** Scenario/verification matrix and interoperability implications.

### Phase 6: Synthesis (1.5 hours)

**Objective:** Produce a decision-usable simulator contract boundary baseline.

**Tasks:**

1. Consolidate the contract, identity, replay, time, reset, tasking, isolation, and verification findings.
2. Apply documented source precedence and leave unsupported questions explicitly unresolved.
3. Produce Glaux Server recommendations and downstream handoffs.
4. Validate the report against every success criterion and prepare it for review.

**Expected Output:** Completed research report at the target path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Every simulator use case is classified as ordinary data-plane traffic, simulator control, external orchestration, unsafe, or unresolved.
- [ ] Data-plane operations reuse the server write and publisher contracts unless a separate mechanism has evidence-backed justification.
- [ ] Scenario, dataset, seed, run, session, simulator, source, principal, virtual-clock, and generated-resource identities are distinguished.
- [ ] Session/control lifecycle, idempotency, concurrency, disconnect, restart, and expiry behavior are defined.
- [ ] Replay modes, identifier treatment, ordering, checkpoints, resume, completion, and reproducibility evidence are explicit.
- [ ] Virtual-time modes distinguish source/domain timestamps from server receipt, commit, and publication times.
- [ ] Reset scope, ownership proof, authorization, preview, transaction, failure recovery, audit, and postconditions prevent impact to unrelated or operational state.
- [ ] Synthetic provenance and environment isolation prevent simulated content from being mistaken for operational content.
- [ ] Invalid-data and fault scenarios exercise normal safeguards rather than bypassing them.
- [ ] Tasking, DDIL, streaming, security, observability, fixture, performance, and interoperability handoffs are explicit.
- [ ] Every recommended operation has positive, negative, authorization, failure, and cleanup verification scenarios.
- [ ] Evidence, Glaux-specific decisions, assumptions, and unresolved issues are visibly distinguished and reproducibly referenced.
- [ ] The report is polished, recommendation-first, independently readable, and self-contained for the project lead, implementers, and later AI agents.

---

## 7. Deliverable

**Deliverable Name:** Simulator-to-Server Contract Boundary Research Report
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-033-simulator-to-server-contract-boundary-report.md`

**Required Content:**

1. Executive summary
2. Scope, terminology, and source-authority statement
3. Simulator use-case and capability inventory
4. Standards-derived versus Glaux-specific behavior classification
5. Data-plane/control-plane contract decision
6. Scenario, dataset, run, session, clock, source, and resource identity model
7. Capability discovery and session/control lifecycle
8. Synthetic data, provenance, policy, and isolation model
9. Replay, determinism, checkpoint, resume, and completion semantics
10. Virtual-time and temporal mapping semantics
11. Reset, cleanup, ownership, safety, and recovery model
12. Dynamic-data, fault-injection, DDIL, and tasking-loop behavior
13. Authentication, authorization, audit, errors, diagnostics, progress, and backpressure
14. Scenario manifest, verification, and interoperability matrix
15. Downstream handoff matrix
16. Recommendations and explicit project decisions
17. Risks, contradictions, assumptions, and unresolved questions
18. Validation against this plan's success criteria
19. References

The simulator operation matrix must include at minimum: use case, data-plane/control-plane classification, operation, required capability, authenticated role/scope, scenario/session state, request inputs, idempotency/concurrency rule, affected resource ownership, temporal behavior, acknowledgement/progress behavior, failure/retry behavior, audit/provenance requirement, isolation guard, expected postcondition, cleanup/reset behavior, verification scenario, standards status, and unresolved issue.

The scenario manifest recommendation must include at minimum: scenario and dataset version, seed, contract/profile versions, required capabilities, initial-state assumptions, resource/identifier mapping, clock mode and mapping, ordered actions or input artifacts with checksums, expected responses/state/events, checkpoints, completion criteria, cleanup scope, and permitted nondeterminism.

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan and Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-031: Server Write and Ingestion Model` report must be complete.
- `IDR-SRV-032: Publisher-to-Server Contract Boundary` report must be complete.
- Relevant prior reports on temporal semantics, validation, identity, storage, lifecycle, and transactions must be available; missing findings must be explicit constraints.
- Exact STANAG/AEP editions and official standards sources must be accessible.
- Any Glaux Simulator design/source material and implementation evidence must be version-pinned; absence must be recorded rather than inferred.
- Research Report Template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-035: Streaming and Event Publication Strategy`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-037: Feasibility and Asynchronous Tasking Strategy`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- Category G security, policy, audit, and DDIL research
- Category H architecture, deployment, configuration, and observability research
- Category I conformance, fixture, performance, security, and interoperability testing research
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

- This topic defines a server contract for simulation and testing; it does not require simulator controls to be present in production.
- Open question: Can orchestration plus ordinary APIs provide safe reset by recreating disposable environments, avoiding a server reset endpoint entirely?
- Open question: Which simulator capabilities are needed in-process for CI versus through a remotely callable contract?
- Open question: Should generated resource IDs be deterministic across runs, remapped per run, or selectable by scenario?
- Open question: Which timeouts and freshness rules follow wall time versus simulation time?
- Open question: How should a resumed replay prove its last durably committed checkpoint?
- Open question: Can simulated and operational records coexist safely, or should simulation require dedicated deployments or tenants?
- Risk: A broadly scoped reset capability could destroy unrelated or operational state.
- Risk: Reusing original idempotency keys in a new run could suppress intended replay data; regenerating them could defeat duplicate-detection tests.
- Risk: Virtual time applied globally could corrupt retention, freshness, command timeout, or audit semantics.
- Risk: Test-only validation bypasses could escape into deployed configurations or invalidate conformance results.
- Risk: Synthetic content without durable provenance could be mistaken for operational sensor data.

---

## References

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- OGC API - Connected Systems Part 1: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems Part 2: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Common Part 1: https://docs.ogc.org/is/19-072/19-072.html
- OGC API - Features Part 4 draft: https://docs.ogc.org/DRAFTS/20-002.html
- SensorML 3.0: https://docs.ogc.org/is/23-000/23-000.html
- SWE Common 3.0: https://docs.ogc.org/is/24-014/24-014.html
- HTTP Semantics, RFC 9110: https://www.rfc-editor.org/rfc/rfc9110.html
- Problem Details for HTTP APIs, RFC 9457: https://www.rfc-editor.org/rfc/rfc9457.html

# Section 037: Feasibility and Asynchronous Tasking Strategy - Research Plan

**Topic ID:** IDR-SRV-037<br>
**Status:** Planned<br>
**Last Updated:** August 1, 2026<br>
**Estimated Research Time:** 19-25 hours<br>
**Actual Research Time:** TBD until complete<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-037-feasibility-and-asynchronous-tasking-strategy-report.md`

---

## Usage Instructions

This plan defines the research needed for IDR-SRV-037. It does not perform the research or make final implementation decisions.

This plan uses a topic-specific structural variant of the standard research-plan template approved by the Glaux Project Lead on July 30, 2026. Its published-behavior, source/evidence, required-report, dependency/handoff, and status sections preserve the template's required content and governance controls; the variant changes organization only.

The resulting report must be polished, recommendation-first, independently readable, and directly useful to the project lead, implementers, and later AI agents. It must supply enough context to stand on its own, synthesize the evidence instead of mirroring the research questions, explain the evidence and tradeoffs behind its recommendations, and clearly distinguish:

1. behavior required or described by published standards;
2. Glaux design decisions needed to implement that behavior; and
3. concerns intentionally deferred to later IDR topics.

Use diagrams and tables only where they make lifecycle or responsibility boundaries easier to understand. Do not turn the report into a transcript of the research questions or a catalogue of undigested quotations.

---

## 1. Research Objective

Define an evidence-backed, implementable strategy for **command feasibility and synchronous/asynchronous tasking** in the Rust-based Glaux Server.

The research must establish:

- the published OGC API - Connected Systems (CSAPI) Part 2 behavior for `ControlStream`, `Command`, `CommandStatus`, `CommandResult`, and `Feasibility` resources;
- the lifecycle and client-visible contract for both synchronous and asynchronous command and feasibility processing;
- the command-validation pipeline needed before feasibility evaluation or task execution, without conflating validation with authorization, safety approval, acceptance, dispatch, or execution;
- the persistence, concurrency, recovery, result, and gateway boundaries needed for reliable asynchronous work; and
- a practical first implementation for Glaux Server, including explicit deferrals and a testable path to later capabilities.

The report must produce a feasibility and async-tasking baseline, not a generic job-processing architecture. The CSAPI resource model and its conformance requirements control the public API design.

### Why This Topic Order

This topic follows `IDR-SRV-036: Control Stream and Command Lifecycle Model`, which defines the broader command lifecycle. IDR-SRV-037 must specialize the feasibility exchange and the synchronous/asynchronous portions of that lifecycle before `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy` defines the security- and policy-sensitive gates.

### Scope Boundaries

In scope:

- command and feasibility submission, status progression, results, cancellation/update implications, and client polling/navigation;
- structural and semantic command validation needed to reach a feasibility or tasking decision;
- feasibility evaluation using command parameters and current server/gateway knowledge;
- durable state, idempotency, concurrency, restart recovery, and worker/gateway handoff for asynchronous processing;
- DDIL, stale-state, and gateway-unavailable behavior only as it changes feasibility or tasking semantics; and
- conformance, interoperability, and implementation tests for the chosen behavior.

Out of scope:

- the complete authorization, operator approval, safety, releasability, and audit design, which belongs to IDR-SRV-038 and Category G;
- a general-purpose workflow engine, enterprise scheduler, message-broker platform, or cross-domain orchestration service;
- detailed deployment and observability platform selection, except where a choice is necessary to make task execution reliable; and
- changes to the CSAPI public resource model or a Glaux-specific public "jobs" API unless the research demonstrates a standards-compatible need.

### Critical Constraints

- Treat published CSAPI Part 2 Clauses 10, 11, and 14 and the applicable Annex A conformance tests as controlling evidence.
- Treat the required `ControlStream.async` property as the declaration of command-processing mode. Do not assume it implies a separate job endpoint, a specific queue technology, or HTTP `202 Accepted` without a normative source.
- Keep asynchronous Rust execution and durable task execution conceptually separate. An `async fn` or spawned future is not, by itself, a restart-safe tasking design.
- Do not claim that a successful feasibility result reserves resources or guarantees later command success. Record what state and definition versions were evaluated and how long the result can be trusted.
- Preserve the distinctions among payload validation, semantic validation, feasibility, authorization, safety approval, acceptance, dispatch, execution, and result production.
- Follow published canonical and nested resource paths exactly in the standards baseline. If the standard contains singular/plural path differences or another apparent inconsistency, identify it and recommend a traceable handling decision rather than silently normalizing it.
- Keep the recommended first implementation achievable in the Glaux Rust service and its primary database. Evaluate a broker or separate worker service only against demonstrated requirements.
- Do not create new governance artifacts as part of this topic.

---

## 2. Published Behavior to Establish

The report must verify every item below against the final published standard and record the relevant clause, requirement identifier, and conformance test.

The core extraction set is CSAPI Part 2 Requirements 25-34 for command schemas, command resources, status, and results; Requirements 35-39 for feasibility; and Requirements 71-77 for command/feasibility transaction operations and their status/result resources. Include Requirements 56-61 where the selected advanced-filtering conformance classes affect command or status retrieval. Follow any requirements delegated to OGC API - Features Part 4 through the referenced conformance tests rather than guessing their HTTP semantics.

### 2.1 Control Streams and Commands

- A `ControlStream` groups commands received by the same `System` and sharing controlled properties and a parameter schema.
- `ControlStream.live` states whether the stream currently accepts commands, while required property `ControlStream.async` states whether commands are processed asynchronously.
- Each supported command format has a schema exposed through the control-stream schema operation; command parameters are validated against the selected schema and encoding.
- A `Command` is associated with its `ControlStream` and carries parameters plus timing, sender, status, target, procedure, and result relationships as applicable.

### 2.2 Synchronous and Asynchronous Status

- A synchronous command returns one status report in the HTTP response and uses only the terminal outcomes allowed by CSAPI Part 2 for that mode.
- An asynchronous command can produce multiple immutable or historically queryable `CommandStatus` reports covering receipt, early validation, scheduling, updates, cancellation, execution progress, completion, rejection, or failure as applicable.
- The report must map `PENDING`, `ACCEPTED`, `REJECTED`, `SCHEDULED`, `UPDATED`, `CANCELED`, `EXECUTING`, `COMPLETED`, and `FAILED` to permitted meanings and transitions. It must also capture required timing information, optional progress, messages, and partial-result behavior.
- Status and result resources remain available through the command's CSAPI nested endpoints. The design must explain how clients discover the canonical command, current status, status history, and results without inventing a parallel job contract.

### 2.3 Feasibility

- A feasibility request is a `Command` created on the control stream's feasibility channel, using the same parameter schema as the corresponding command.
- Feasibility processing can be synchronous or asynchronous in the same manner as regular command processing.
- A `Feasibility` resource has a canonical URL and exposes nested `CommandStatus` and `CommandResult` resources.
- Feasibility status reuses `CommandStatus`, but `SCHEDULED` and `UPDATED` are unused for feasibility requests; the remaining codes must be interpreted as specified by CSAPI Part 2.
- A feasibility result commonly uses inline data conforming to the control stream's feasibility-result schema, which is normally distinct from its command-result schema. Rich analysis may include likelihood, constraints, steps, timing, or alternatives in addition to a binary answer.

### 2.4 Creation, Mutation, and Result Semantics

- Trace the create, replace, update, and delete requirements that apply to commands, feasibility requests, their status reports, and results.
- Determine the normative HTTP response, `Location`/link, representation, and error behavior for synchronous and asynchronous creation. Do not infer transport behavior from the word "asynchronous."
- Establish which command results can be inline, linked observations, linked datastreams with a time range, or external resources, and how partial results appear in status reports.

---

## 3. Research Questions

### A. Normative and Profile Baseline

1. Which CSAPI Part 2 requirements and Annex A tests govern control-stream mode, command creation, feasibility creation, status, results, and transaction operations?
2. Which details are mandatory, recommended, optional, profile-defined, or left to the server?
3. How do CSAPI Part 1 resource/link conventions, SensorML 3.0, SWE Common 3.0, AEP-4789, and earlier Glaux IDRs constrain parameter schemas and tasking behavior?
4. Are there published errata, issue discussions, examples, or implementation findings that clarify ambiguous paths, response semantics, or state behavior? These sources may explain a decision but must not override the final standard without an explicit compatibility rationale.

### B. Public Resource and API Contract

1. What resource is created immediately for each accepted command or feasibility submission, and what must be atomic with that creation?
2. What does the submission response contain for synchronous completion, immediate rejection/failure, and asynchronous processing?
3. How do canonical URLs, nested collection URLs, status history, current-status links, result links, and collection filters work together?
4. Which errors belong in an HTTP problem response, and which outcomes belong in a persisted `CommandStatus` report?
5. How should retry, client timeout, duplicate submission, replace/update, cancellation, and deletion behave without executing a command twice or erasing required history?

### C. Task State Model

1. What are the valid transitions for synchronous commands, asynchronous commands, and asynchronous feasibility requests?
2. Which states are terminal, which may repeat as progress reports, and which require actual or estimated execution times?
3. What does `ACCEPTED` promise, and under what conditions can a task later become `REJECTED` or `FAILED`?
4. How are scheduling, updates, user cancellation, system cancellation, execution failure, and completion represented?
5. How are partial results and final results attached, and when is a result endpoint expected to exist?
6. How should the server recover tasks and their observable state after a process restart or a lost gateway response?

### D. Validation and Feasibility Pipeline

1. Which checks belong to transport/encoding validation, schema validation, SWE Common component validation, semantic validation, target/control-stream validation, feasibility, authorization, safety, acceptance, and execution?
2. How are required fields, types, records, choices, arrays, encodings, units, allowed values/ranges, temporal windows, geometries, semantic identifiers, target identity, control-stream liveness, and schema versions checked?
3. Which failures prevent resource creation, which create a `REJECTED` status, and which become an execution-time `FAILED` status?
4. Which state inputs drive feasibility: target/system status, deployment state, queue or schedule conflicts, gateway health, command-definition version, authorization-independent operational constraints, and source-data currentness?
5. How are conditional, partial, indeterminate, stale, or gateway-delegated feasibility results encoded within the advertised feasibility-result schema?
6. What evaluation timestamp, state snapshot/version, schema version, provenance, confidence, validity window, and redaction metadata are needed to interpret or cache a feasibility result?
7. When must feasibility be re-evaluated, and can a later command reference prior feasibility without treating it as a reservation or guarantee?

### E. Durable Rust Implementation

1. What domain types and transition rules prevent invalid status histories at compile time or at the transaction boundary?
2. What transaction boundary should cover creation of a command/feasibility resource, its initial status, idempotency record, and worker handoff?
3. Is an in-process worker with PostgreSQL-backed durable work sufficient for the first implementation? What evidence would justify an external broker or independently deployed worker?
4. How should work claiming, leases/heartbeats, retries, backoff, dead-letter handling, optimistic concurrency, cancellation races, and crash recovery operate?
5. What gateway/adapter trait lets Glaux delegate feasibility or execution without coupling public CSAPI handlers to a device protocol?
6. Which status transitions are owned by the API service, worker, gateway adapter, or receiving system, and how are duplicates or out-of-order callbacks handled?
7. What limits are needed for queue depth, per-system concurrency, payload size, task duration, status frequency, result size, and retention?

### F. Security, DDIL, and Information Exposure

1. Which checks can be completed before IDR-SRV-038 authorization and safety gates, and where are those gates invoked without defining their policy here?
2. Which feasibility or validation messages could expose system capabilities, schedules, operational state, policy rules, or other sensitive information, and what safe diagnostic shape should this topic recommend?
3. How do disconnected gateways, stale status, delayed callbacks, clock uncertainty, and resynchronization affect feasibility confidence and task state?
4. Which provenance and audit hooks must be retained for IDR-SRV-038 without designing the full audit record here?

### G. Conformance and Interoperability

1. What fixtures and executable scenarios prove every relevant CSAPI requirement and state transition?
2. How should tests cover JSON and supported SWE Common encodings, schema versions, status filtering, result forms, and canonical/nested navigation?
3. What do existing CSAPI servers and clients reveal about status polling, response shapes, endpoint paths, and feasibility-result interoperability?
4. Which behavior should be configurable for compatibility, and which behavior must remain invariant for conformance?

---

## 4. Source and Evidence Plan

### Primary Standards

Use final published documents and their normative references before drafts, examples, or third-party summaries:

- OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002, especially Clauses 10, 11, 14, and Annex A
  https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001
  https://docs.ogc.org/is/23-001/23-001.html
- OGC SensorML Encoding Standard 3.0, OGC 23-000
  https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0, OGC 24-014
  https://docs.ogc.org/is/24-014/24-014.html
- STANAG 4789 / AEP-4789 source material available to the project, with volume, clause, and releasability recorded for each claim.

Also inspect the exact versions of JSON Schema, HTTP, Web Linking, OGC API - Features, and other standards normatively referenced by the applicable CSAPI clauses. Cite the incorporated version rather than a convenient current web summary.

### Project Evidence

At minimum, reconcile the report with:

- the Glaux OGC API - Connected Systems Upstream Standards-History Evidence Register at `Docs/Research/Initial Designs/IDR/glaux-server/IDR Evidence/ogc-connected-systems-upstream-history-register.md`, consulting and refreshing only entries material to commands, status, results, feasibility, tasking, and transaction behavior;
- IDR-SRV-007 and IDR-SRV-008 for CSAPI Part 2 requirements and conformance mapping;
- IDR-SRV-013 for HTTP error/failure semantics;
- IDR-SRV-015 through IDR-SRV-020 for resource, lifecycle, temporal, and status foundations;
- IDR-SRV-022 through IDR-SRV-024 for SWE Common schemas, encodings, units, and semantic constraints;
- IDR-SRV-029 for transactions, idempotency, and concurrency;
- IDR-SRV-031 for the server write/ingestion model, IDR-SRV-032 for the publisher contract, IDR-SRV-033 for the simulator contract, IDR-SRV-034 for dynamic update semantics, and IDR-SRV-035 for streaming/event publication; and
- IDR-SRV-036 for the controlling command lifecycle.

If the project lead has formally approved a prerequisite exception under the overall-plan Governance Rules, use the affected plan provisionally, identify the exception and impact, and do not silently make a decision that belongs to another topic.

### Implementation and Interoperability Evidence

- Use official Rust, async-runtime, database, and selected library documentation for implementation claims.
- Compare existing CSAPI implementations and client findings already assigned to IDR-SRV-014A through IDR-SRV-014G.
- Use issue discussions and examples only as informative evidence. Record implementation/version/date and distinguish observed behavior from normative behavior.
- Date-check relevant register entries and linked issues, pull requests, commits, tags, and releases. A merged pre-publication change can explain the published text; a post-publication change or unresolved proposal cannot silently amend the published CSAPI 1.0 contract.
- Prefer a small prototype or executable transition model when documentation alone cannot resolve a concurrency or recovery question.

### Evidence Discipline

For every material recommendation, capture:

- source and stable anchor;
- evidence class: normative, project-controlled, observed implementation, or engineering judgment;
- direct Glaux implication;
- confidence and unresolved ambiguity; and
- downstream topic or test affected.

Short quotations may be used when exact wording matters, but the report should primarily synthesize and explain.

---

## 5. Research Methodology

### Phase 1: Build the Standards and Gap Matrix (3-4 hours)

1. Extract applicable CSAPI Part 2 requirements from Clauses 10, 11, and 14 and their Annex A tests.
2. Consult and refresh the tasking-related entries in the upstream standards-history register, tracing any material issue to its linked disposition, pull request, commit, and release status.
3. Record resource paths, operations, representations, status rules, result rules, and conformance-class dependencies.
4. Map each normative item to an existing Glaux IDR decision, an open decision, or a downstream handoff.
5. Flag path, cardinality, response, or lifecycle ambiguities for explicit resolution without turning unresolved upstream proposals into requirements.

Output: a compact standards-to-server matrix with exact anchors and no invented requirements.

### Phase 2: Model Public Lifecycles and Client Contracts (4-5 hours)

1. Model synchronous command, asynchronous command, synchronous feasibility, and asynchronous feasibility flows separately.
2. Define allowed state transitions, repeated progress reports, terminal outcomes, timing fields, partial results, and final results.
3. Trace submission responses and link navigation from the control stream to canonical command/feasibility resources and their status/result histories.
4. Analyze retry, timeout, duplicate, update, cancellation, deletion, and restart scenarios.

Output: one readable lifecycle diagram plus a state/response matrix that separates published rules from Glaux choices.

### Phase 3: Define Validation and Feasibility Semantics (3-4 hours)

1. Build a staged pipeline from decoding and schema checks through semantic checks, feasibility, later policy gates, acceptance, dispatch, execution, and results.
2. Map SWE Common structures and constraints to validation behavior and safe diagnostics.
3. Define feasibility inputs, evaluation evidence, result schema needs, validity/staleness rules, and gateway-delegated behavior.
4. Specify which outcome is an HTTP error, `REJECTED` status, `FAILED` status, conditional feasibility result, or unavailable/indeterminate result.

Output: a validation/feasibility responsibility matrix and a recommended feasibility-result information model.

### Phase 4: Evaluate a Practical Rust Architecture (4-5 hours)

1. Derive domain types, persistence records, invariants, and ownership boundaries from the public lifecycle.
2. Compare a synchronous in-request path, a durable database-backed worker path, and broker-backed alternatives against actual Glaux requirements.
3. Define atomic submission, idempotency, work claiming, retries, leases, cancellation races, callbacks, status append, result storage, and restart recovery.
4. Define the adapter/gateway boundary and how external status is reconciled without allowing protocol-specific behavior to leak into CSAPI handlers.
5. Recommend the smallest complete first implementation and state objective triggers for later architectural expansion.

Output: a component/transaction diagram, option comparison, and implementation recommendation sized to the Glaux Rust server.

### Phase 5: Design Verification Scenarios (3-4 hours)

Cover at least:

- synchronous completion, rejection, and execution failure;
- asynchronous acceptance, scheduling, progress, completion, rejection, failure, update, and cancellation;
- synchronous and asynchronous feasibility with inline results;
- invalid encoding, invalid SWE Common structure, unit/range violation, unknown target, and non-live control stream;
- duplicate submission, client timeout/retry, concurrent cancellation/completion, and out-of-order gateway callbacks;
- worker crash, process restart, expired lease, replay, and result-write failure;
- stale or invalidated feasibility after target, gateway, schedule, schema, or deployment state changes;
- DDIL/gateway-unavailable and safe-redaction behavior; and
- canonical/nested link navigation and applicable Annex A conformance tests.

Recommend deterministic clocks, fake gateways, transition/property tests, database integration tests, HTTP contract tests, restart/replay tests, and external-client interoperability checks where each adds distinct coverage.

Output: a traceable scenario matrix usable by IDR-SRV-050 through IDR-SRV-056.

### Phase 6: Synthesize the Report (1-2 hours)

1. Resolve contradictions and label remaining uncertainties.
2. Present the recommended baseline first, followed by evidence, alternatives, and implementation consequences.
3. Verify every normative claim and every dependency/handoff.
4. Remove repetition, speculative platform design, and material that belongs to IDR-SRV-038 or later operational topics.

---

## 6. Required Report Structure and Presentation

The report should use this narrative order unless the evidence supports a clearer sequence:

1. Executive decision summary
2. Scope, terminology, and evidence method
3. Published CSAPI behavior baseline
4. Feasibility semantics and result model
5. Synchronous/asynchronous tasking lifecycle
6. Validation and decision-gate boundaries
7. Recommended Glaux public API behavior
8. Recommended Rust domain and component architecture
9. Persistence, concurrency, idempotency, and recovery
10. Gateway, DDIL, security, and information-exposure boundaries
11. Conformance and test strategy
12. Phased implementation recommendation
13. Risks, deferred decisions, and open questions
14. Traceability and references

The report must include these compact decision aids:

- a standards traceability table with clause/requirement, obligation, Glaux implication, and verification method;
- a state/response matrix comparing synchronous command, asynchronous command, synchronous feasibility, and asynchronous feasibility behavior;
- a validation/feasibility/authorization/safety/acceptance/execution responsibility matrix; and
- an architecture option table that explains why the recommended first implementation is sufficient and what would trigger a broker or separate worker.

Each recommendation must state the decision, rationale, implementation consequence, important alternative, and confidence. Keep normative language reserved for actual standards obligations; use "Glaux should" for recommendations.

---

## 7. Success Criteria

The research is complete when the report:

- traces every applicable CSAPI Part 2 command, feasibility, status, result, and transaction requirement to a concrete server behavior and test;
- accounts for material official issue and pull-request history affecting those behaviors, with the published baseline, later maintenance, and unresolved proposals kept visibly distinct;
- explains the required `ControlStream.async` semantics without inventing a second public job model;
- defines valid, client-visible synchronous and asynchronous status/result flows, including terminal states and recovery-visible behavior;
- preserves a precise validation pipeline and keeps feasibility separate from authorization, safety, acceptance, dispatch, and execution;
- defines feasibility evidence, result structure, staleness, invalidation, provenance, and gateway-delegation behavior without treating feasibility as a guarantee;
- recommends a bounded Rust architecture with explicit transaction, idempotency, concurrency, retry, cancellation, and restart-recovery rules;
- identifies safe diagnostics and clean handoffs to IDR-SRV-038, DDIL, observability, and test topics;
- provides executable test scenarios and applicable conformance-test traceability; and
- is concise enough to guide implementation while complete enough that the project lead, an implementer, or a later AI agent does not need the research plan, prior reports, or raw standards text merely to understand its context and recommendations.

---

## 8. Dependencies and Downstream Handoffs

### Primary Prerequisites

- IDR-SRV-007: CSAPI Part 2 Requirement Baseline
- IDR-SRV-008: Conformance Class and Requirement Mapping
- IDR-SRV-013: Error Model, HTTP Status Codes, and Failure Semantics
- IDR-SRV-022: SWE Common Data Component Strategy
- IDR-SRV-023: Schema and Encoding Validation Strategy
- IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy
- IDR-SRV-031: Server Write and Ingestion Model
- IDR-SRV-032: Publisher-to-Server Contract Boundary
- IDR-SRV-033: Simulator-to-Server Contract Boundary
- IDR-SRV-034: Datastream, Observation, and Status Update Semantics
- IDR-SRV-035: Streaming and Event Publication Strategy
- IDR-SRV-036: Control Stream and Command Lifecycle Model

### Downstream Handoffs

- IDR-SRV-038 owns command-specific authorization, safety approval, and command-audit hooks.
- IDR-SRV-039, IDR-SRV-039A, and IDR-SRV-040 own security, zero-trust, and policy decisions; IDR-SRV-041 owns general audit logging and accountability; and IDR-SRV-042 and IDR-SRV-043 own DDIL-visible behavior and synchronization/conflict handling.
- IDR-SRV-044 through IDR-SRV-049 own final framework, service, deployment, configuration, observability, and recovery platform choices.
- IDR-SRV-050 through IDR-SRV-056 consume this report's state transitions, scenarios, fixtures, performance limits, security cases, and interoperability expectations.

---

## 9. Status Checklist

- [ ] Published CSAPI requirements and conformance tests extracted
- [ ] Standards/project gap matrix completed
- [ ] Synchronous and asynchronous lifecycles modeled
- [ ] Feasibility and validation semantics defined
- [ ] HTTP/resource/link behavior resolved
- [ ] Rust architecture options evaluated
- [ ] Persistence, concurrency, and recovery rules recommended
- [ ] Security/DDIL handoffs bounded
- [ ] Test and interoperability scenarios traced
- [ ] Report drafted, edited for readability, and source-verified
- [ ] Report reviewed
- [ ] Report accepted

**Actual Research Time:** TBD until complete<br>
**Completion Date:** TBD until complete

---

## 10. Notes and Open Questions

- This approved structural variant changes organization only; the controlling overall-plan governance and acceptance rules still apply in full.
- Open question: Which implementation or interoperability evidence available at execution time best exercises asynchronous feasibility, cancellation, retry, and restart-recovery behavior?
- Risk: Treating feasibility as authorization, acceptance, dispatch, or execution success would collapse distinct command-lifecycle decisions and produce unsafe server semantics.
- Risk: Recommendations that are not traced to CSAPI requirements, conformance tests, or clearly classified implementation evidence could misdirect the later Rust design.

---

## References

- Glaux Server Overall IDR Research Plan
  `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/overall-idr-research-plan.md`
- Glaux Research Planning Approach
  `Docs/Governance/research-planning-approach.md`
- Glaux Initial Planning Guidance
  `Docs/Governance/initial-planning-guidance.md`
- OGC API - Connected Systems - Part 2: Dynamic Data
  https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems - Part 1: Feature Resources
  https://docs.ogc.org/is/23-001/23-001.html
- OGC SensorML Encoding Standard 3.0
  https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0
  https://docs.ogc.org/is/24-014/24-014.html

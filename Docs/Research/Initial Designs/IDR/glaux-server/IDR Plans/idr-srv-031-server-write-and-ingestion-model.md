# Section 031: Server Write and Ingestion Model - Research Plan

**Status:** Planned
**Last Updated:** July 29, 2026
**Estimated Research Time:** 14-18 hours
**Actual Research Time:** TBD until complete
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-031-server-write-and-ingestion-model-report.md`

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

This topic research must define the Glaux Server planning baseline for the **server write and ingestion model**: what data and operations the server accepts directly, accepts only through a constrained integration contract, delegates to another component, or rejects.

The research must cover standards-facing resource creation and mutation, dynamic-data ingestion, metadata and document submission, batch and replay traffic, administrative imports, command-status and feasibility-result writes, and non-standard source mediation. It must turn the resource, representation, validation, storage, lifecycle, and transaction findings from earlier IDR topics into one coherent server-side write path.

The research must answer:

- Which write operations are required, permitted, optional, or unsupported for each CSAPI resource family and representation?
- Which write surfaces are part of the standards-facing API, which require a publisher contract, which are simulator/test controls, and which must remain internal or administrative?
- What common processing stages every accepted write must traverse before it changes canonical server state?
- What validation, normalization, identity, provenance, policy, transaction, idempotency, concurrency, persistence, event-generation, and error obligations remain authoritative inside Glaux Server?
- How should Glaux Server handle incomplete, invalid, conflicting, duplicate, out-of-order, late, replayed, untrusted, or unsupported submissions?
- What responsibilities may be delegated to publishers, adapters, simulators, gateways, or brokers without weakening standards conformance or server authority?

The output must be an ingestion boundary baseline containing a write-surface inventory, accept/delegate/reject decision matrix, conceptual processing pipeline, responsibility allocation, transaction and failure boundaries, source/provenance requirements, and explicit handoffs to publisher, simulator, dynamic-data, security, DDIL, conformance, and implementation topics.

### Why This Topic Order

This topic begins Category F after the domain-model, representation, validation, persistence, query, lifecycle, and transaction topics. Those topics establish what Glaux Server stores and the invariants it must preserve. This topic converts those findings into the server's authoritative mutation boundary.

It must precede:

- `IDR-SRV-032`, which specializes the boundary for publisher traffic;
- `IDR-SRV-033`, which specializes it for simulator replay, reset, and synthetic traffic; and
- `IDR-SRV-034` through `IDR-SRV-038`, which define the semantics of accepted dynamic data and tasking records.

### Critical Constraints

- Keep the analysis on Glaux Server obligations and behavior. Do not design Glaux Publisher, Glaux Simulator, or every possible adapter.
- Start from normative CSAPI operations and conformance requirements. Treat editor repositories and implementation behavior as non-normative evidence unless a controlling standard incorporates them.
- Treat OGC API - Features Part 4 CRUD as a draft normative dependency where CSAPI invokes it; record the exact snapshot and draft status used.
- Do not assume every resource is writable, every HTTP method is supported, or every source may use every write surface.
- Do not create a second canonical data model at an internal ingestion boundary. Any internal envelope must map explicitly to standards-facing resources and previously researched domain semantics.
- Glaux Server must retain final authority for authentication context, authorization, canonical identity, conformance-critical validation, policy/releasability enforcement, provenance, transaction boundaries, persistence, and canonical errors.
- Distinguish transport acceptance from semantic acceptance and durable commit. A successful receipt is not automatically a successful resource mutation.
- Do not finalize publisher-specific authentication or delivery mechanics here; hand those to `IDR-SRV-032`.
- Do not finalize simulator control operations here; hand replay, reset, scenario, and synthetic-data behavior to `IDR-SRV-033`.
- Do not finalize observation/status semantics, streaming, command lifecycle, security policy, or DDIL synchronization here; identify requirements and hand them to their controlling topics.

---

## 2. Research Questions

### Core Questions

1. What standards-aligned and project-required write operations must Glaux Server expose, constrain, delegate, or reject?
2. What common server-side ingestion stages and invariants apply to every accepted mutation?
3. How should direct API writes, publisher traffic, simulator traffic, brokered messages, files/batches, federation, and administrative imports enter the server without creating inconsistent semantics?
4. How should validation, normalization, idempotency, concurrency, provenance, persistence, and failure handling differ by resource and submission class?
5. What exact contract decisions belong here, and what must be handed to later publisher, simulator, dynamic-data, security, DDIL, and testing topics?

### Detailed Questions

#### Normative Write-Surface Baseline

- Which create, replace, update, delete, append, batch, and action operations are defined by CSAPI Parts 1 and 2 and their normative dependencies?
- For each resource family, which HTTP methods, request representations, response codes, preconditions, links, identifiers, and conformance classes apply?
- Which dynamic-data operations append immutable records, update current state, replace definitions, or trigger domain actions?
- Which operations are standards-defined, AEP-profiled, Glaux-specific, implementation-supporting, or explicitly out of scope?
- Where do normative prose, ATS, schemas, and published OpenAPI artifacts disagree or leave gaps?

#### Submission Classes and Entry Points

- Which entry-point classes must be evaluated: public CSAPI API, authenticated publisher API, broker consumer, internal adapter interface, file/batch import, federation pull, administrative tool, simulator/test control, and internal service call?
- Can a standards-facing CSAPI operation serve the publisher or simulator need without a separate endpoint?
- When is a non-standard internal endpoint justified, and how is it prevented from becoming an accidental public conformance claim?
- Which operations must be rejected at particular entry points even if the underlying resource is otherwise writable?

#### Accept, Delegate, or Reject Decisions

- Which payloads can Glaux Server accept in canonical CSAPI, SensorML, or SWE Common form?
- Which legacy or source-specific payloads must be transformed by an external publisher/adapter before submission?
- Which safe, deterministic normalizations may the server perform, and which transformations would change meaning or authority?
- What happens when required parent resources, datastream definitions, semantic bindings, units, source identity, or policy labels are absent?
- When should the server reject, quarantine, stage pending dependencies, accept with warnings, or delegate processing?

#### Common Ingestion Pipeline

- What logical stages are required: transport decoding, authentication, authorization, media-type selection, envelope validation, schema/profile validation, source resolution, reference resolution, semantic validation, normalization, policy checks, idempotency and concurrency checks, transaction, persistence, latest-state maintenance, event/outbox creation, provenance/audit recording, and response construction?
- Which stages are mandatory, conditional, asynchronous, or source-specific?
- At what stage is a write considered received, validated, accepted, committed, and published?
- Which intermediate artifacts, raw payloads, validation results, and rejected/quarantined records must be retained?
- How should failures after durable receipt but before downstream publication be recovered?

#### Validation and Normalization Boundary

- Which syntactic, structural, referential, semantic, temporal, geospatial, unit, policy, and domain validations apply to each write class?
- Which validations can use the earlier schema and encoding strategy, and which require domain state or source trust?
- How should source-authored, adapter-derived, server-normalized, inferred, and simulated values remain distinguishable?
- How should timestamps, identifiers, units, observed properties, geometry, nil values, encodings, and relationship references be normalized without losing the submitted representation or meaning?
- Which normalization findings must be deferred to `IDR-SRV-034` or other semantic topics?

#### Identity, Provenance, and Source Authority

- Which identifiers are client supplied, source scoped, server assigned, or derived?
- What source, publisher/adapter, ingestion path, transformation, validation, policy, and simulation context must accompany a committed write?
- How should conflicting claims from multiple sources be attributed rather than silently merged?
- When must the original payload or a cryptographic digest be retained for audit and reproducibility?
- How do source registration and trust affect acceptance without becoming a substitute for authorization or validation?

#### Transactions, Idempotency, Ordering, and Replay

- What is the atomic unit for a resource mutation, observation append, multi-record batch, metadata-plus-data submission, and import?
- How do conditional requests, idempotency keys, source message IDs, sequence numbers, offsets, batch IDs, and hashes interact?
- What response and recovery behavior applies to full success, partial failure, duplicate submission, stale precondition, conflict, timeout, and ambiguous commit?
- How should late, out-of-order, replayed, and backfilled records affect history, latest-value state, events, and audit?
- Which decisions come from `IDR-SRV-029`, and which require later DDIL or dynamic-data research?

#### Persistence and Publication Boundary

- Which canonical resources, time-series records, documents, raw payloads, validation artifacts, provenance records, audit records, and outbox entries result from each accepted write?
- Which changes must be committed atomically?
- When should accepted writes generate system events or streaming publications?
- How should publication failure avoid rolling back an otherwise valid committed write while guaranteeing eventual delivery where required?
- How do retention, deletion, archival, and latest-value strategies constrain ingestion?

#### Error and Operational Behavior

- What canonical problem types, status codes, headers, field-level diagnostics, retry guidance, and correlation identifiers are required?
- How should the server distinguish malformed, unauthenticated, unauthorized, forbidden-by-policy, unsupported, conflicting, duplicate, stale, rate-limited, dependency-missing, quarantined, and internally failed writes?
- What information may be returned to an external caller versus retained only in operational/audit records?
- Which metrics and traces identify ingestion rate, lag, rejects, quarantine, duplicate rate, validation cost, commit latency, and publication lag?

#### Performance, DDIL, and Backpressure

- Which write classes require synchronous versus asynchronous acknowledgement?
- What bounded queue, batching, rate-limit, flow-control, and backpressure behavior is required?
- How should disconnected publishers or simulators replay safely after reconnection?
- How should the server communicate temporary inability to accept work without encouraging unsafe retries?
- Which aspects belong to `IDR-SRV-035` streaming/backpressure, `IDR-SRV-041` audit evidence, `IDR-SRV-042` DDIL-visible behavior, `IDR-SRV-043` replay/synchronization conflicts, performance testing, or deployment design?

#### Verification and Handoffs

- What positive, negative, concurrency, replay, policy, failure-injection, and recovery fixtures prove each boundary decision?
- Which behaviors are conformance obligations versus Glaux integration-contract tests?
- What decisions must be handed to `IDR-SRV-032` through `IDR-SRV-038`, Category G security/DDIL topics, and Category I test topics?

---

## 3. Primary Resources

The future research report must inspect authoritative sources directly and record the exact version, publication status, URL or controlled path, and access date used.

### Project and Prior-IDR Sources

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Completed `IDR-SRV-001` through `IDR-SRV-030` reports in `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`, especially resource/API, representation, validation, persistence, lifecycle, and transaction findings
- Project-controlled STANAG 4789 and AEP-4789 Volumes I and II; record exact title, edition, date, status, storage location, and handling constraints

### Normative Standards and Published Artifacts

- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Common - Part 1: Core: https://docs.ogc.org/is/19-072/19-072.html
- OGC API - Features - Part 1: Core: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC API - Features - Part 4 draft CRUD specification: https://docs.ogc.org/DRAFTS/20-002.html
- Versioned CSAPI Part 1 OpenAPI artifacts: https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/openapi/
- Versioned CSAPI Part 2 OpenAPI artifacts: https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/openapi/
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- HTTP Semantics, RFC 9110: https://www.rfc-editor.org/rfc/rfc9110.html
- Problem Details for HTTP APIs, RFC 9457: https://www.rfc-editor.org/rfc/rfc9457.html

### Implementation Evidence

- OGC API - Connected Systems editor repository, pinned to a reviewed commit: https://github.com/opengeospatial/ogcapi-connected-systems
- Implementation-study reports `IDR-SRV-014A` through `IDR-SRV-014G`
- OpenSensorHub: https://github.com/opensensorhub/osh-core
- Connected Systems Go: https://github.com/OS4CSAPI/connected-systems-go
- 52North Connected Systems pygeoapi proof of concept: https://github.com/52North/connected-systems-pygeoapi
- OS4CSAPI client and interoperability evidence: https://github.com/OS4CSAPI/ogc-client-CSAPI_2

Implementation artifacts are comparative, non-normative evidence. The report must identify implementation-specific behavior and must not promote it to a server obligation without a controlling source or explicit Glaux decision.

---

## 4. Supporting Resources

- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC schemas: https://schemas.opengis.net/
- MQTT Version 5.0: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- CloudEvents specification: https://github.com/cloudevents/spec
- PostgreSQL documentation: https://www.postgresql.org/docs/
- Rust SQLx documentation: https://docs.rs/sqlx/
- Rust serde documentation: https://serde.rs/
- Rust tracing documentation: https://docs.rs/tracing/

These supporting technologies are candidates to evaluate, not predetermined implementation selections.

---

## 5. Research Methodology

### Phase 1: Evidence Baseline and Write Inventory (2-2.5 hours)

**Objective:** Establish the controlling source set and enumerate every candidate server mutation.

**Tasks:**

1. Record exact standards/profile versions and recursively inspect relevant normative references, requirement classes, and ATS material.
2. Extract prior-IDR decisions that constrain writable resources, representations, validation, storage, lifecycle, transactions, and errors.
3. Inventory operations by resource family, method/action, payload, conformance class, caller class, and normative status.
4. Record contradictions among normative prose, schemas, OpenAPI artifacts, implementations, and project assumptions without silently resolving them.

**Expected Output:** Source-anchored write-operation inventory and contradiction register.

### Phase 2: Entry-Point and Accept/Delegate/Reject Analysis (3-4 hours)

**Objective:** Define the server's external and internal mutation surfaces.

**Tasks:**

1. Compare standards API, publisher, broker/adapter, batch/import, federation, simulator, administrative, and internal entry points.
2. Classify each operation as directly accepted, contract-constrained, delegated, internal-only, or rejected, with rationale.
3. Map payload forms to canonical resources and identify prohibited semantic duplication.
4. Identify caller, source, trust, policy, and environment prerequisites.

**Expected Output:** Write-surface and accept/delegate/reject decision matrix.

### Phase 3: Processing, Validation, and Persistence Model (3-4 hours)

**Objective:** Define the conceptual server-side processing model for accepted writes.

**Tasks:**

1. Model processing stages from receipt through response, durable commit, event/outbox creation, and audit.
2. Map validation and normalization requirements to each write class.
3. Identify transaction units, persistence artifacts, raw/rejected/quarantined handling, and provenance requirements.
4. Test the model against missing references, invalid semantics, policy denial, duplicate/replayed data, concurrent writes, and downstream publication failure.

**Expected Output:** Processing-stage matrix, state-transition model, and failure-boundary analysis.

### Phase 4: Reliability, DDIL, Security, and Operations Analysis (2.5-3.5 hours)

**Objective:** Resolve cross-cutting behavior that changes write acceptance or recovery.

**Tasks:**

1. Analyze idempotency, conditional mutation, ordering, batching, retry, backpressure, ambiguous commit, replay, and recovery.
2. Analyze source identity, authorization input, policy/releasability, provenance, audit, and diagnostic redaction requirements.
3. Identify DDIL buffering and synchronization implications without pre-empting the DDIL topics.
4. Identify observability signals and operational failure modes.

**Expected Output:** Reliability and cross-cutting requirement matrix with downstream ownership.

### Phase 5: Verification and Handoff Design (2-2.5 hours)

**Objective:** Make each boundary decision testable and usable by later topics.

**Tasks:**

1. Define positive, negative, malformed, unauthorized, duplicate, concurrency, replay, partial-failure, and recovery scenarios.
2. Separate standards conformance assertions from project integration-contract assertions.
3. Map unresolved semantic decisions to `IDR-SRV-032` through `IDR-SRV-038` and security/DDIL topics.
4. Map implementation and test implications to architecture, fixtures, conformance, performance, and interoperability topics.

**Expected Output:** Verification matrix and explicit downstream handoff matrix.

### Phase 6: Synthesis (1.5 hours)

**Objective:** Convert the evidence into a decision-usable ingestion boundary baseline.

**Tasks:**

1. Consolidate the inventories, matrices, processing model, and handoffs.
2. Apply documented source precedence to conflicts; leave unsupported questions explicitly unresolved.
3. Produce recommendations and decision points for Glaux Server.
4. Validate the report against every success criterion and prepare it for review.

**Expected Output:** Completed research report at the target path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Every in-scope CSAPI resource family and mutation identified in the controlling standards has a clause-anchored write-surface entry.
- [ ] Each candidate entry point and operation is classified as directly accepted, contract-constrained, delegated, internal-only, rejected, or unresolved.
- [ ] Standards obligations, AEP profile choices, Glaux decisions, implementation observations, and researcher inferences are visibly distinguished.
- [ ] The common ingestion stages, state transitions, transaction points, persistence artifacts, and failure boundaries are documented.
- [ ] Validation, normalization, identity, provenance, policy, idempotency, concurrency, ordering, replay, and error responsibilities are allocated.
- [ ] Normative prose, ATS, schema, OpenAPI, and implementation discrepancies are recorded rather than silently reconciled.
- [ ] Positive and negative verification scenarios trace to every recommended boundary behavior.
- [ ] Publisher, simulator, dynamic-data, security, DDIL, architecture, and testing handoffs are explicit and non-overlapping.
- [ ] Unknown, unavailable, draft-dependent, or policy-dependent conclusions are labeled with their impact.
- [ ] Recommendations are bounded to Glaux Server and references are reproducible.
- [ ] The report is polished, recommendation-first, independently readable, and self-contained for the project lead, implementers, and later AI agents.

---

## 7. Deliverable

**Deliverable Name:** Server Write and Ingestion Model Research Report
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-031-server-write-and-ingestion-model-report.md`

**Required Content:**

1. Executive summary
2. Scope, terminology, and source-authority statement
3. Standards and profile write-obligation baseline
4. Resource and operation inventory
5. Entry-point taxonomy
6. Accept/delegate/reject decision matrix
7. Common ingestion processing and state-transition model
8. Validation and normalization responsibility matrix
9. Identity, trust-input, provenance, and source-authority findings
10. Transaction, idempotency, concurrency, ordering, and replay findings
11. Persistence, latest-state, event/outbox, and audit implications
12. Error, retry, backpressure, and operational behavior
13. Security, policy, DDIL, and deployment implications
14. Verification scenarios and conformance boundary
15. Downstream handoff matrix
16. Recommendations and explicit project decisions
17. Risks, contradictions, assumptions, and unresolved questions
18. Validation against this plan's success criteria
19. References

The write-surface matrix must include at minimum: resource/data class, operation, caller/entry point, controlling clause or project source, normative status, request representation, identity/source prerequisite, validation class, transaction unit, result/response, accept/delegate/reject disposition, provenance requirement, downstream side effects, error behavior, verification scenario, and unresolved issue.

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan and Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-030` reports should be complete; any unavailable report must be recorded as a scope limitation rather than guessed.
- Exact project-controlled STANAG/AEP editions and the official CSAPI, SensorML, SWE Common, and normative-dependency sources must be accessible.
- Draft sources, mutable repositories, and implementation evidence must be pinned to the exact revision analyzed.
- Research Report Template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-032: Publisher-to-Server Contract Boundary`
- `IDR-SRV-033: Simulator-to-Server Contract Boundary`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-035: Streaming and Event Publication Strategy`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-037: Feasibility and Asynchronous Tasking Strategy`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- Category G security and DDIL research
- Category H architecture and implementation research
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

- This topic defines the server mutation boundary and conceptual ingestion model, not implementation code or a final deployment topology.
- Open question: Can all required publisher traffic use standards-facing CSAPI operations, or is a private high-throughput ingestion contract justified?
- Open question: Which mutations require synchronous durable acknowledgement versus accepted-for-processing semantics?
- Open question: May records referencing not-yet-present resources be staged, or must they be rejected?
- Open question: When is raw-payload retention required, optional, prohibited, or sampling-based?
- Open question: Which normalization steps are lossless enough to perform automatically?
- Risk: A convenience ingestion API could become a second, inconsistent domain model.
- Risk: Treating editor OpenAPI examples as normative could omit operations required by standards prose.
- Risk: Weak ambiguous-commit and replay rules could create duplicates, incorrect latest values, or missing events.
- Risk: Overdelegating validation to publishers/adapters could weaken conformance, security, and interoperability.

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
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC API - Features Part 4 draft: https://docs.ogc.org/DRAFTS/20-002.html
- SensorML 3.0: https://docs.ogc.org/is/23-000/23-000.html
- SWE Common 3.0: https://docs.ogc.org/is/24-014/24-014.html
- HTTP Semantics, RFC 9110: https://www.rfc-editor.org/rfc/rfc9110.html
- Problem Details for HTTP APIs, RFC 9457: https://www.rfc-editor.org/rfc/rfc9457.html

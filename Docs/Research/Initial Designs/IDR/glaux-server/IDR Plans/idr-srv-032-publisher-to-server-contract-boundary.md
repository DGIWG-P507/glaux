# Section 032: Publisher-to-Server Contract Boundary - Research Plan

**Status:** Planned
**Last Updated:** July 29, 2026
**Estimated Research Time:** 14-18 hours
**Actual Research Time:** TBD until complete
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-032-publisher-to-server-contract-boundary-report.md`

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

This topic research must define the **publisher-to-server contract boundary** from the Glaux Server perspective. A publisher is any authenticated external software role that submits source data to Glaux Server, including Glaux Publisher, a standards-native producer, a legacy-feed adapter, a gateway, a broker bridge, a batch importer, or a federated source acting through an approved publication path.

The research must specialize the server write and ingestion model from `IDR-SRV-031` into an implementable external contract covering publisher onboarding and identity, endpoint or transport use, authentication and authorization inputs, source claims, payload and envelope rules, validation, batching, idempotency, ordering, retry, backpressure, provenance, policy labels, errors, diagnostics, health, DDIL replay, and contract evolution.

The research must answer:

- What server-visible contract must every publisher satisfy, regardless of its source protocol or internal implementation?
- When should a publisher use standards-facing CSAPI operations, and when is a separate constrained ingestion interface justified?
- How are publisher identity, represented source identity, credentials, authority scope, provenance, and policy context expressed and verified?
- Which preprocessing responsibilities belong to publishers or adapters, and which validation and canonicalization decisions must remain inside Glaux Server?
- How does the server acknowledge acceptance, rejection, quarantine, duplicate detection, partial batch outcomes, throttling, or ambiguous commit?
- How can the contract support high-rate, intermittent, replayed, and brokered delivery without making transport-specific behavior part of the canonical domain model?

The output must be a publisher contract boundary baseline containing a role and source taxonomy, contract-surface decision, responsibility matrix, registration/trust inputs, request-envelope and payload rules, acknowledgement/error semantics, delivery-state model, operational requirements, and downstream handoffs.

### Why This Topic Order

This topic follows `IDR-SRV-031: Server Write and Ingestion Model`, which establishes the authoritative server write surface and common processing stages. It narrows that model to sustained external publication traffic before simulator-specific behavior and before accepted data receives its detailed dynamic-data semantics.

The result must inform `IDR-SRV-033` without conflating live publishers with simulators, and it must provide source, delivery, and acknowledgement assumptions to `IDR-SRV-034` through `IDR-SRV-038`, security, DDIL, observability, deployment, fixture, performance, and interoperability research.

### Critical Constraints

- Define only the server-side contract boundary. Do not prescribe Glaux Publisher internals, adapter product architecture, source-system behavior, or operator procedures.
- Use `IDR-SRV-031` as the controlling server write model; do not create a parallel ingestion pipeline or canonical data model.
- A publisher identity, the source it represents, the authenticated principal, and the authority asserted over submitted data are separate concepts unless evidence proves otherwise.
- Authentication does not imply authorization, source authority, payload validity, policy acceptability, or trustworthiness.
- Do not assume all publishers are trusted, online, standards-compliant, synchronized, single-source, or able to produce complete CSAPI/SensorML/SWE Common structures.
- Preserve Glaux Server authority over canonical identity, validation, policy/releasability, provenance, persistence, audit, conformance behavior, and canonical errors.
- Prefer existing standards-facing operations when they satisfy the requirement. Any private ingestion contract must have a documented need, scope, versioning boundary, and exact mapping to canonical resources.
- Treat broker, MQTT, file, and batch mechanisms as candidate transports, not predetermined selections.
- Keep simulator-only reset, scenario, synthetic clock, and deterministic replay controls in `IDR-SRV-033`.
- Identify authentication, authorization, zero-trust, and policy requirements here, but defer their full mechanisms and threat analysis to Category G.

---

## 2. Research Questions

### Core Questions

1. Which publisher roles and publication patterns must Glaux Server support, and what uniform contract applies across them?
2. What identity, source-registration, authority, credential, provenance, and policy information must accompany publisher traffic?
3. What payload, envelope, validation, batching, ordering, idempotency, retry, acknowledgement, and error rules make publication deterministic and interoperable?
4. Which responsibilities belong to the publisher/adapter and which must remain inside Glaux Server?
5. How should the contract support high-rate, brokered, disconnected, and replayed delivery while remaining testable, secure, and evolvable?

### Detailed Questions

#### Publisher and Source Taxonomy

- Which server-visible roles are needed for Glaux Publisher, standards-native producers, OSH/Connected Systems publishers, legacy adapters, tactical gateways, broker bridges, file/batch importers, federated servers, command/status gateways, and fixture publishers?
- Which roles submit authoritative metadata, raw data, mapped data, derived data, or combinations?
- Can one publisher represent multiple sources, and can one source be represented by multiple publishers?
- How are publisher instance, software/version, deployment, source, tenant/mission, and authenticated principal distinguished?
- Which source classes require prior registration, explicit approval, constrained scopes, quarantine, or test-only treatment?

#### Contract Surface and Transport

- Which publication needs can use CSAPI resource operations from `IDR-SRV-031` directly?
- If a dedicated ingestion interface is needed, what requirements cannot safely or efficiently be met by the standards-facing API?
- What direct HTTP, brokered message, MQTT, file/batch, gateway, and federation patterns should be evaluated?
- What semantic contract remains invariant across transports?
- How are transport acknowledgements distinguished from Glaux Server validation and durable-commit acknowledgements?
- How should a broker or gateway preserve end-source identity instead of becoming the apparent author of every record?

#### Registration, Identity, and Trust Inputs

- What minimum registration record does Glaux Server need for a publisher and represented source?
- Which identifiers, endpoint or topic bindings, credentials, certificates/keys, allowed data classes, resource scopes, rate limits, policy labels, and operational contacts are server configuration versus submitted claims?
- What lifecycle states are needed, such as pending, active, constrained, suspended, revoked, expired, quarantined, and retired?
- How are credential rotation, software-instance replacement, source transfer, and emergency revocation handled?
- Which trust decisions are static configuration, validated claims, observed health, or per-request authorization decisions?
- What evidence and audit history must accompany trust-state changes?

#### Responsibility Allocation

- Which responsibilities normally belong to a publisher/adapter: source-protocol handling, parsing, source credential custody, source-specific mapping, buffering, sequence capture, mapping-profile selection, and local health reporting?
- Which responsibilities must remain with Glaux Server: canonical identity, authoritative validation, relationship resolution, policy enforcement, provenance, transaction/persistence, conformance behavior, audit, and canonical errors?
- Which safe transformations may be performed on either side, and how is transformation provenance preserved?
- What must happen if publisher preprocessing conflicts with server validation or established resource state?
- Which contract elements are mandatory for all publishers versus profile- or transport-specific?

#### Payload and Envelope Contract

- What representations may carry systems, procedures, deployments, sampling features, datastream definitions, SensorML documents, SWE Common components, observations, status updates, events, health, command status, and feasibility results?
- Which fields are publisher supplied, source supplied, server assigned, or prohibited from client assignment?
- What envelope metadata is required for publisher ID, source ID, message ID, source timestamp, sequence/offset, batch ID, schema/profile, content type, encoding, policy marking, trace context, and provenance?
- How are canonical CSAPI payloads distinguished from raw/source-native payloads?
- May raw payloads be accepted, and if so, through which constrained adapter/import path and with what retention or quarantine rules?
- How are references to missing resources or datastream definitions handled?

#### Validation and Acceptance

- What validation should a publisher perform before submission, and what must the server always repeat or perform independently?
- How are schema/profile, semantic/unit, temporal, geospatial, referential, source-authority, authorization, and policy failures classified?
- When may the server reject, quarantine, stage, accept with warnings, or accept only a valid subset of a batch?
- How are validation findings attributed to the source, publisher transformation, server normalization, or policy decision?
- Can a publisher query or retrieve its rejection/quarantine results without gaining access to other sources or sensitive server details?

#### Delivery, Idempotency, Ordering, and Replay

- What message IDs, idempotency keys, sequence numbers, offsets, timestamps, epochs, batch IDs, or hashes must publishers provide?
- What scope and retention window apply to duplicate detection?
- What ordering guarantees may publishers rely on, and what ordering must they not assume?
- How are late, out-of-order, replayed, backfilled, and corrected records represented?
- What delivery state model distinguishes received, authenticated, validated, accepted, committed, partially committed, rejected, quarantined, duplicate, and published?
- How are timeout and ambiguous-commit cases retried safely?

#### Batching, Flow Control, and Backpressure

- Which records may be batched together, and what are the atomicity and partial-failure rules?
- What size, count, time-window, and nesting limits are required?
- How should synchronous response, asynchronous job/status, broker acknowledgement, retry delay, rate limiting, and load shedding work?
- How are publisher fairness, per-source quotas, priority, and denial-of-service protections expressed without embedding deployment policy in the domain model?
- How should a DDIL publisher buffer, resume, and prove continuity after reconnect?

#### Errors, Diagnostics, and Source Health

- Which HTTP/problem or message-level error contract applies to malformed, unsupported, unauthenticated, unauthorized, policy-denied, invalid, conflicting, duplicate, stale, throttled, dependency-missing, quarantined, and failed submissions?
- Which errors are retryable, conditionally retryable, or permanent, and how is that communicated?
- Which correlation identifiers and per-record batch results are required?
- How may publishers report health, availability, backlog, last successful publication, transformation failure, and credential failure?
- How is publisher/source health kept distinct from sensor/system operational status?
- Which diagnostics are safe to expose to publishers versus administrators only?

#### Security, Policy, and Audit Boundary

- What authenticated principal and authorization context must reach ingestion?
- How are publisher permissions constrained by source, resource family, operation, mission/tenant, policy marking, environment, and command capability?
- How are source credentials kept outside Glaux Server when an adapter can safely hold them?
- What submission, validation, policy, retry, and trust-state events require audit records?
- How should sensitive topology, resource existence, payload fragments, and policy reasons be redacted in errors and metrics?

#### Contract Evolution and Interoperability

- Which parts of the contract are standards-versioned, Glaux-versioned, profile-versioned, or transport-versioned?
- How are capability negotiation, schema/profile selection, deprecation, compatibility, and rolling upgrades handled?
- What golden messages, invalid-message corpus, mock publishers, legacy adapters, delayed/replay publishers, and broker fixtures are required?
- Which tests demonstrate compatibility with OSH, Connected Systems Go, 52North/pygeoapi, SECD evidence, and OS4CSAPI clients?
- Which findings must be handed to simulator, dynamic-data, streaming, command, security, DDIL, architecture, and testing topics?

---

## 3. Primary Resources

The future report must analyze the exact versions used and cite standards by clause or requirement identifier where possible.

### Project and Prior-IDR Sources

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- `IDR-SRV-031: Server Write and Ingestion Model` report
- Relevant `IDR-SRV-001` through `IDR-SRV-030` reports, especially API operations, errors, resources, identity, validation, storage, lifecycle, and transactions
- Project-controlled STANAG 4789 and AEP-4789 Volumes I and II; record exact edition, date, status, controlled path, and handling constraints

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
- MQTT Version 5.0: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- CloudEvents specification: https://github.com/cloudevents/spec

### Implementation and Interoperability Evidence

- OGC API - Connected Systems editor repository, pinned to the reviewed commit: https://github.com/opengeospatial/ogcapi-connected-systems
- `IDR-SRV-014A` through `IDR-SRV-014G` implementation-study and interoperability reports
- OpenSensorHub sources: https://github.com/opensensorhub/osh-core
- Connected Systems Go sources: https://github.com/OS4CSAPI/connected-systems-go
- 52North Connected Systems pygeoapi proof of concept: https://github.com/52North/connected-systems-pygeoapi
- OS4CSAPI client repository and test evidence: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- OS4CSAPI discussions, captured as dated low-authority implementation evidence: https://github.com/orgs/OS4CSAPI/discussions

---

## 4. Supporting Resources

- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- OAuth 2.0 Security Best Current Practice, RFC 9700: https://www.rfc-editor.org/rfc/rfc9700.html
- OAuth 2.0 Mutual-TLS Client Authentication, RFC 8705: https://www.rfc-editor.org/rfc/rfc8705.html
- W3C PROV-O: https://www.w3.org/TR/prov-o/
- NATS documentation: https://docs.nats.io/
- Apache Kafka documentation: https://kafka.apache.org/documentation/
- Eclipse Mosquitto documentation: https://mosquitto.org/documentation/
- OpenTelemetry documentation: https://opentelemetry.io/docs/

Authentication and messaging sources are candidates for comparison. This topic must not select the final security architecture or broker unless the controlling evidence and downstream ownership support that decision.

---

## 5. Research Methodology

### Phase 1: Evidence and Contract Framework (2-2.5 hours)

**Objective:** Establish the authoritative publisher-contract inputs and a reproducible comparison frame.

**Tasks:**

1. Extract publisher-relevant decisions and unresolved questions from `IDR-SRV-031` and prior reports.
2. Extract relevant CSAPI/AEP obligations and identify whether they constrain the server, caller, payload, or transport.
3. Define comparison fields for publisher/source identity, surface, payload, delivery, validation, authority, errors, security, and operations.
4. Record source conflicts, draft dependencies, and unavailable controlled material explicitly.

**Expected Output:** Contract requirement inventory and evaluation rubric.

### Phase 2: Role, Identity, Trust, and Responsibility Analysis (3-4 hours)

**Objective:** Define who publishes, what source is represented, and how authority is bounded.

**Tasks:**

1. Build publisher, adapter, gateway, broker, importer, federation, and fixture role taxonomy.
2. Model publisher/source/principal identity and registration lifecycle without assuming they are equivalent.
3. Allocate preprocessing, validation, normalization, source credential, provenance, policy, persistence, and audit responsibilities.
4. Identify credential, authorization, policy, revocation, and trust-state requirements to hand to Category G.

**Expected Output:** Role/identity model, registration lifecycle, and responsibility-allocation matrix.

### Phase 3: Payload and Delivery Contract Analysis (3-4 hours)

**Objective:** Define the interoperable submission and acknowledgement contract.

**Tasks:**

1. Compare standards API, direct ingestion, brokered, batch/file, gateway, and federation patterns against `IDR-SRV-031`.
2. Define invariant payload/envelope metadata and resource-specific payload rules.
3. Define validation and acceptance states, batch atomicity, per-record results, idempotency, ordering, replay, retry, and ambiguous-commit behavior.
4. Define canonical error, acknowledgement, correlation, and safe diagnostic behavior.

**Expected Output:** Contract-surface decision, payload/envelope matrix, and delivery-state model.

### Phase 4: Scale, DDIL, Security, and Evolution Analysis (2.5-3.5 hours)

**Objective:** Test the contract against operational and lifecycle stressors.

**Tasks:**

1. Analyze batching, flow control, backpressure, rate limiting, priority, load shedding, reconnect, and backlog replay.
2. Analyze authorization inputs, source scope, policy labels, provenance, audit, and diagnostic redaction.
3. Analyze capability negotiation, schema/profile compatibility, deprecation, and rolling upgrades.
4. Identify decisions owned by DDIL, security, architecture, deployment, and observability topics.

**Expected Output:** Cross-cutting contract constraints and downstream decision register.

### Phase 5: Fixtures and Interoperability Analysis (2-2.5 hours)

**Objective:** Make the proposed publisher contract independently verifiable.

**Tasks:**

1. Define compliant, malformed, unauthorized, mis-scoped, duplicate, delayed, out-of-order, partial-batch, throttled, and reconnecting publisher scenarios.
2. Identify golden envelopes, payloads, expected acknowledgements, and observable server state.
3. Compare proposed behavior against pinned implementation and client evidence.
4. Separate standards conformance tests from Glaux publisher-contract tests and map them to later test topics.

**Expected Output:** Publisher contract verification and interoperability matrix.

### Phase 6: Synthesis (1.5 hours)

**Objective:** Produce a decision-usable publisher contract boundary baseline.

**Tasks:**

1. Consolidate role, identity, responsibility, surface, payload, delivery, and verification findings.
2. Resolve conflicts only through documented authority precedence; label unsupported decisions and open questions.
3. Produce Glaux Server recommendations and explicit downstream handoffs.
4. Validate the report against this plan and prepare it for review.

**Expected Output:** Completed research report at the target path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Every in-scope publisher/source class has an evidence-backed role, identity, registration, and authority treatment.
- [ ] The relationship among authenticated principal, publisher instance, represented source, and source authority is explicit.
- [ ] The report decides when standards-facing operations are sufficient and documents any justified private ingestion surface.
- [ ] Publisher and Glaux Server responsibilities are allocated for parsing, mapping, validation, normalization, policy, provenance, persistence, errors, and audit.
- [ ] Payload/envelope, batching, acknowledgement, delivery-state, idempotency, ordering, retry, replay, and backpressure rules are specified or explicitly deferred to a named owner.
- [ ] Authentication, authorization, source trust, validation, and policy acceptance are not conflated.
- [ ] Error and diagnostic behavior covers record-level, batch-level, transport-level, and ambiguous-commit failures without leaking protected information.
- [ ] Contract evolution and capability/version compatibility are addressed.
- [ ] Every recommendation traces to authoritative evidence or is labeled as a Glaux project decision.
- [ ] Publisher fixtures and handoffs to simulator, dynamic-data, security, DDIL, architecture, deployment, and testing topics are explicit.
- [ ] The report is polished, recommendation-first, independently readable, and self-contained for the project lead, implementers, and later AI agents.

---

## 7. Deliverable

**Deliverable Name:** Publisher-to-Server Contract Boundary Research Report
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-032-publisher-to-server-contract-boundary-report.md`

**Required Content:**

1. Executive summary
2. Scope, publisher terminology, and authority model
3. Standards and AEP obligation baseline
4. Publisher, adapter, gateway, broker, importer, and source taxonomy
5. Publisher/source/principal identity and registration lifecycle
6. Publisher-versus-server responsibility matrix
7. Contract-surface and transport analysis
8. Payload, envelope, media-type, and mapping rules
9. Validation, acceptance, quarantine, and provenance behavior
10. Delivery-state, batching, idempotency, ordering, retry, replay, and acknowledgement behavior
11. Error, correlation, diagnostics, and source-health behavior
12. Backpressure, DDIL, security, policy, audit, and evolution implications
13. Fixture and interoperability matrix
14. Downstream handoff matrix
15. Recommendations and explicit project decisions
16. Risks, contradictions, assumptions, and unresolved questions
17. Validation against this plan's success criteria
18. References

The contract matrix must include at minimum: publisher class, represented source class, exact endpoint/path or broker subject, HTTP method or message operation, accepted media type/schema/profile, operation/data class, canonical payload mapping, required envelope fields, authenticated principal, authorization scope, validation responsibilities, idempotency/ordering fields, response status and headers or acknowledgement semantics, retry classification, error behavior, provenance/audit requirement, rate/backpressure constraint, DDIL behavior, contract-version negotiation, verification fixture, and unresolved issue. Fields for a private ingestion surface apply only if the research justifies adopting one.

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan and Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-031: Server Write and Ingestion Model` report must be complete; unavailable upstream decisions must be treated as blockers or explicit assumptions.
- Relevant prior reports on API operations, error semantics, validation, identities, storage, lifecycle, and transactions must be available.
- Exact STANAG/AEP editions and official standards sources must be accessible.
- Implementation and protocol sources must be pinned to the revision evaluated.
- Research Report Template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-033: Simulator-to-Server Contract Boundary`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-035: Streaming and Event Publication Strategy`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-037: Feasibility and Asynchronous Tasking Strategy`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- Category G authentication, authorization, policy, audit, and DDIL topics
- Category H architecture, configuration, deployment, and observability topics
- Category I fixture, conformance, performance, security, and interoperability topics
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

- This topic treats adapters, gateways, broker bridges, and importers as publisher roles only to the extent that their behavior changes the Glaux Server contract.
- Open question: Is a private high-throughput ingestion API necessary, or can CSAPI plus bounded batching satisfy all publisher cases?
- Open question: Must publisher and source registration be persisted as standards-facing resources, administrative records, configuration, or a combination?
- Open question: Which publisher-supplied transformations are trusted assertions versus claims the server must revalidate?
- Open question: How should per-record batch outcomes be represented consistently across HTTP and broker transports?
- Open question: What acknowledgement level can be guaranteed during DDIL or degraded operation?
- Risk: Conflating publishers with represented sources would weaken authorization, provenance, and revocation.
- Risk: Transport-specific envelopes could fork canonical semantics across HTTP, broker, and batch paths.
- Risk: Returning detailed validation or trust failures could reveal protected resource existence, topology, or policy.
- Risk: Unbounded replay or retry could overwhelm ingestion or create duplicate canonical state.

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
- MQTT Version 5.0: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- CloudEvents specification: https://github.com/cloudevents/spec

# Section 041: Audit Logging and Accountability Strategy - Research Plan

**Topic ID:** IDR-SRV-041<br>
**Status:** Planned<br>
**Last Updated:** July 30, 2026<br>
**Estimated Research Time:** 15-18.5 hours<br>
**Actual Research Time:** TBD until complete<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-041-audit-logging-and-accountability-strategy-report.md`

---

## Usage Instructions

Before executing this plan, review the exemplar corpus and align the report to the proven standards for content nature, type, level, and detail:

- Full research-plans corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- Blueprint-first exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md
- Inventory/sourcing rigor exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/15-fixture-sourcing-organization.md
- End-state synthesis exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/38-testing-playbook-synthesis.md

This plan is limited to research planning. It does not execute the research and does not produce the research report.

The resulting report must be polished, recommendation-first, independently readable, and usable as shared decision material by the project lead, implementers, and later AI agents. It must supply enough context to stand on its own, synthesize the evidence instead of mirroring the research questions, and clearly state findings, recommendations, implementation implications, and unresolved questions.

The report must separate audit requirements from general application logs, metrics, traces, domain events, security alerts, and provenance. It must label the authority of every asserted requirement and must not describe ordinary logging as tamper-proof, non-repudiable, or sufficient for legal accountability without supporting evidence.

---

## 1. Research Objective

This topic research must define the Glaux Server planning baseline for **audit logging and accountability** across public and administrative API access, authentication and authorization decisions, policy/releasability enforcement, resource creation and change, ingestion and validation, source and trust administration, streaming, command/control, feasibility, configuration changes, synchronization, lifecycle actions, and access to audit evidence itself.

The research must answer:

- Which events must be auditable because they change authoritative state, exercise privilege, affect command/safety behavior, make or enforce a policy decision, expose protected data, alter evidence, or affect accountability.
- What minimum fields make an audit record attributable, ordered, reviewable, correlatable, integrity-checkable, and useful without capturing secrets or unrestricted sensitive payloads.
- How human users, service identities, publishers, adapters, simulators, federated servers, command gateways, administrators, automated processes, and delegated actors are represented.
- How audit records relate to domain events, request logs, traces, provenance, validation results, command history, and security alerts without duplicating or conflating them.
- How audit generation, buffering, storage, access, export, review, redaction, retention, integrity verification, and failure behavior should work.
- How the audit boundary behaves during partial failure, high load, DDIL operation, reconnect, replay, and synchronization.
- Which implementation, persistence, security, policy, observability, lifecycle, fixture, conformance, and interoperability decisions depend on this strategy.

The output must be a decision-ready audit/accountability baseline containing an auditable-event taxonomy, actor and delegation model, audit record schema, capture and correlation rules, sensitive-data controls, integrity and availability strategy, access/review/export model, failure/DDIL behavior, retention handoff, implementation implications, and falsifiable acceptance tests.

### Why This Topic Order

This topic follows:

- IDR-SRV-039: Authentication, Authorization, and API Security Threat Model
- IDR-SRV-039A: Zero-Trust Architecture Alignment and Enforcement Model
- IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints

Those topics define identities, privileges, threats, policy decisions, sensitive resources, and cross-boundary constraints. Audit research can then specify which security- and policy-relevant actions must be recorded and what evidence is needed to attribute and review them.

This topic must precede:

- IDR-SRV-042, which defines externally visible DDIL behavior and must know which constrained-mode actions remain accountable.
- IDR-SRV-043, which defines synchronization and conflict handling and must preserve audit continuity without treating audit stores as generic replication.
- IDR-SRV-047 and IDR-SRV-048, which implement configuration and observability while keeping audit evidence distinct.
- IDR-SRV-049, which defines migration, backup, and restore behavior for retained audit records.
- Security, fixture, conformance, and interoperability test planning.

### Critical Constraints

- Do not conflate audit records with debug/application logs, HTTP access logs, metrics, traces, domain/system events, provenance records, or security alerts. Define relationships and correlation, not one undifferentiated log.
- Do not record passwords, tokens, private keys, full authorization headers, secret configuration, or unrestricted command and sensor payloads.
- Do not assume authenticated identity proves the real-world person or authority behind a shared, delegated, or compromised credential.
- Do not claim non-repudiation or tamper-proof storage solely because records are append-only, hashed, signed, encrypted, or stored remotely.
- Do not create a single global fail-closed rule. Determine failure behavior by auditable event class, safety impact, operational necessity, and available buffering.
- Do not make audit evidence mutable through the same privileges or ordinary code paths as the actions it records.
- Do not expose sensitive resource identity, policy rationale, command capability, topology, source identity, or audit content to unauthorized clients.
- Do not define retention periods here. Identify audit record classes, integrity dependencies, access constraints, and preservation needs for IDR-SRV-030.
- Do not redesign authentication, authorization, policy, command safety, DDIL semantics, synchronization, or general observability here.
- Keep the strategy deployable for a single-node development profile as well as a reference or tactical-edge profile; external SIEM or managed cloud logging cannot be mandatory.

---

## 2. Research Questions

### Core Questions

1. Which Glaux Server actions, decisions, data accesses, failures, and administrative changes must produce audit evidence?
2. What actor, subject, action, outcome, time, authority, correlation, provenance, and integrity fields are required for accountable records?
3. How should audit evidence be captured, protected, stored, accessed, reviewed, exported, retained, and verified?
4. How should audit generation behave under failure, overload, DDIL, replay, and synchronization without silently losing accountability or halting safe read-only operation?
5. What boundaries and handoffs distinguish audit from provenance, domain events, request logs, observability, policy, security alerts, and command history?

### Detailed Questions

#### Audit and Accountability Terminology

- What precise meanings will Glaux use for audit event, audit record, audit trail, accountable action, actor, initiator, subject, target, delegate, authority, outcome, reason, review, evidence, integrity, completeness, and repudiation?
- Which properties are required for technical accountability versus forensic support, operational diagnosis, conformance, or mission policy?
- What can the server assert about identity, intent, authority, causation, and outcome, and what remains unknown?
- Which terms and obligations originate in adopted standards/profile material versus Glaux decisions?
- What evidence threshold would be required before using stronger terms such as non-repudiation?

#### Audit Versus Adjacent Records

- When is a CSAPI system event a domain fact rather than an audit event?
- When is a W3C PROV activity lineage evidence rather than accountability evidence?
- Which HTTP request/access-log fields are useful for correlation but insufficient as an audit record?
- Which OpenTelemetry traces or logs can carry correlation context without becoming the authoritative audit trail?
- When should an audit event also generate a security alert or operational metric?
- How are command status and results related to, but distinct from, records of who submitted, approved, changed, canceled, or viewed a command?
- Which adjacent records can be referenced rather than duplicated?

#### Auditable-Event Taxonomy

- Which authentication events are auditable: success, failure, token/credential rejection, logout, revocation, identity-provider failure, and use of emergency/local credentials?
- Which authorization and policy events are auditable: permit, deny, filtered response, redaction, override, break-glass, policy unavailable, stale-policy use, and administrative policy change?
- Which resource operations are auditable: create, import, replace, update, patch, delete, archive, restore, export, bulk action, and rejected/conflicted change?
- Which ingestion events are auditable: source registration, trust/authority change, validation rejection, quarantine release, normalization-rule change, replay, duplicate suppression, and manual correction?
- Which command/control events are auditable: capability enablement, command submission, feasibility request, authorization, operator approval, acceptance/rejection, dispatch, cancellation, override, result/status change, timeout, and unknown outcome?
- Which security and administration events are auditable: privilege change, audit configuration change, key/certificate reference change, maintenance action, migration, backup/restore, audit access, audit export, and integrity verification?
- Which synchronization/DDIL events are auditable: constrained-mode entry/exit, local decision under cached authority, queued action, conflict, merge/rejection, reconnect, and audit-record synchronization failure?
- Which high-volume read, observation, or streaming accesses require per-record evidence, aggregate evidence, sampling, or no audit event?

#### Event Selection and Profile Requirements

- Which event classes are mandatory in every deployment profile?
- Which events depend on enabled capabilities, sensitivity, command/control, cross-boundary behavior, or mission policy?
- How should public demo and test profiles remain useful without normalizing unsafe operational settings?
- What configurable event selection is permitted without allowing accountable actions to be disabled silently?
- What event taxonomy identifier and version let readers interpret older records after the taxonomy changes?
- How are event-selection changes themselves audited?

#### Actor, Delegation, and Authority

- How are human, client application, workload/service, publisher, adapter, simulator, federated server, command gateway, administrator, and automated process actors distinguished?
- What authenticated principal, claimed identity, acting service, delegated subject, client ID, source ID, node ID, and session/request context must be captured?
- How are on-behalf-of actions, service-account actions, batch jobs, migrations, and scheduled lifecycle jobs attributed?
- How are shared credentials, anonymous access, local emergency access, expired credentials, and identity-provider unavailability represented honestly?
- Which authority, role, scope, policy, or approval reference was used for an action?
- How are authority and policy references recorded without copying sensitive policy content?

#### Audit Record Schema

- What stable event ID and event-type/version are required?
- What occurred time, server-recorded time, source-reported time, clock source, and uncertainty or skew indicator are required?
- What actor, delegate, authority, source, subject resource, parent resource, action, operation, API route template, outcome, and reason category are required?
- What request, trace, correlation, transaction, batch, command, feasibility, provenance, and synchronization identifiers are required?
- What node, deployment profile, software version, schema/profile version, and policy version are needed?
- When are before/after digests, changed-field names, object versions, ETags, or redacted summaries sufficient?
- How should record classification, sensitivity, handling, retention class, and access restrictions be represented?
- Which fields are mandatory, conditional, optional, prohibited, or administrator-only for each event class?

#### Outcome, Reason, and Causality

- How should requested, attempted, accepted, committed, dispatched, completed, denied, failed, rolled back, canceled, timed out, unknown, and partially completed outcomes differ?
- When should one operation produce multiple phase-specific audit events?
- How should a failure to write authoritative state differ from a failure to emit, persist, forward, or verify its audit record?
- How are automated consequences linked to the initiating request without asserting unsupported direct causality?
- Which human-readable details are appropriate, and which stable machine-readable reason codes are required?
- How should policy denial be useful for review without exposing sensitive decision inputs to ordinary clients?

#### Sensitive Data, Minimization, and Redaction

- Which values are prohibited from audit records even in failure paths?
- When can identifiers, hashes, field names, counts, ranges, or redacted summaries replace payload copies?
- How should request bodies, SensorML/SWE values, observations, command parameters, feasibility inputs/results, and validation diagnostics be minimized?
- How should log-injection, delimiter, control-character, Unicode, and structured-field attacks be prevented?
- How is redaction performed before persistence and forwarding?
- How are changes to redaction rules tested and audited?
- Can hashes become sensitive correlators or enable guessing attacks?

#### Integrity and Tamper Evidence

- What threat model applies to application compromise, administrator misuse, storage compromise, record truncation, reordering, deletion, replay, and clock manipulation?
- Which guarantees are needed in each deployment profile: append-only permissions, separation of duty, immutable storage, sequence numbers, checkpoints, hashes, hash chains, signatures, remote forwarding, or independent verification?
- What do those controls prove and what do they not prove?
- How are key identifiers, algorithm versions, verification results, broken chains, missing ranges, and restarts represented?
- How can key rotation and software upgrade preserve verification?
- How are integrity-check operations and failures audited?
- What recovery is possible when an audit store or chain is corrupted?

#### Capture, Transaction, and Delivery

- Which audit records must commit atomically with authoritative state, and which may use a durable outbox or asynchronous pipeline?
- What ordering guarantees are necessary within a request, resource, command, transaction, node, and synchronized deployment?
- What idempotency or duplicate detection applies to retry, replay, outbox delivery, and synchronization?
- How is an audit gap distinguished from no auditable activity?
- What buffering and backpressure limits apply?
- Which actions fail closed when audit persistence is unavailable, which may continue with a durable local buffer, and which safe read-only actions may continue with explicit degraded evidence?
- How are dropped, delayed, sampled, aggregated, or unavailable audit records made visible?

#### Storage, Access, Search, and Review

- What store roles are required for authoritative audit evidence, local buffer, search index, export, archive, and recovery copy?
- Which indexes and queries are needed by event type, actor, subject, action, outcome, time, correlation, command, policy, source, node, or integrity status?
- Who can read, search, export, verify, annotate, or administer audit data?
- How is access to audit evidence itself audited?
- How should paginated/exported evidence preserve stable ordering, completeness markers, integrity metadata, and handling labels?
- Can reviewer annotations be separate append-only records rather than edits to original evidence?
- How should audit review workflows avoid becoming an incident-response or SIEM product design?

#### Retention, Archival, and Deletion Boundary

- Which audit classes have different preservation, integrity, and access needs?
- Which audit evidence must outlive the resource, credential, source, policy, command, or account it references?
- How should holds and investigations prevent automated disposition?
- What can be redacted or pseudonymized while preserving accountability?
- How do archive, restore, backup expiry, deletion, and media sanitization rules from IDR-SRV-030 apply?
- What minimum tombstone or integrity marker remains if audit records are legitimately removed?
- How are disposition actions against audit data themselves audited?

#### DDIL, Replay, and Synchronization

- Which audit functions continue locally when identity, policy, broker, time, or central audit services are unavailable?
- How are cached authority, stale policy, emergency credentials, local-only decisions, and uncertain time represented?
- How are locally buffered records ordered, deduplicated, verified, and synchronized after reconnect?
- How are gaps, collisions, clock skew, conflicting node identity, duplicate event IDs, and broken integrity chains handled?
- What audit details must not be exposed through DDIL diagnostics?
- Which broader freshness/degraded semantics belong to IDR-SRV-042 and which synchronization mechanics belong to IDR-SRV-043?

#### Observability and Alerting Boundary

- Which health measures expose audit pipeline availability, buffer depth, delay, failure, verification status, and storage pressure?
- Which metrics must avoid labels containing sensitive actor/resource identifiers?
- Which audit failures require an alert?
- How do trace IDs aid correlation while respecting sampling and trust limitations?
- Which operational logs diagnose the audit subsystem without duplicating audit content?
- Which implementation details belong to IDR-SRV-048?

#### Validation and Testing

- What schema validation and stable event-type compatibility tests are needed?
- What fixtures cover each mandatory event class and actor type?
- What negative tests cover missing required fields, secret leakage, spoofed actor fields, log injection, oversized fields, unauthorized access, and unapproved event disabling?
- What transaction/failure tests cover rollback, crash boundaries, outbox replay, buffer exhaustion, store outage, and partial forwarding?
- What integrity tests cover deletion, insertion, reordering, truncation, key rotation, restart, and corrupted checkpoints?
- What DDIL tests cover uncertain time, offline buffer, reconnect, duplicate replay, gaps, and conflicting nodes?
- What performance tests establish sustainable audited-operation rates without weakening mandatory coverage?
- What test proves that audit evidence can explain a representative resource change, policy denial, command lifecycle, archive/delete action, and synchronization conflict end to end?

---

## 3. Primary Resources

### Project and Governance Sources

- Glaux Server Overall IDR Research Plan
- Glaux Server Goal and Definition
- Research Planning Approach
- Research Report Template
- IDR-SRV-019, IDR-SRV-030, and IDR-SRV-036 through IDR-SRV-040 reports, including IDR-SRV-039A
- Project-available AEP-4789 and STANAG 4789 security, command/control, audit, accountability, records, and cross-boundary material, with document identity, edition, date, approval status, handling restrictions, and repository location recorded

### Standards and Authoritative Guidance

- NIST SP 800-53 Revision 5, Audit and Accountability control family and related controls: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
- NIST SP 800-92, Guide to Computer Security Log Management: https://csrc.nist.gov/pubs/sp/800/92/final
- NIST SP 800-92 Revision 1 Initial Public Draft, used only as draft planning guidance with status stated: https://csrc.nist.gov/pubs/sp/800/92/r1/ipd
- OWASP Logging Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
- RFC 5424, The Syslog Protocol: https://www.rfc-editor.org/rfc/rfc5424
- RFC 3339, Date and Time on the Internet: https://www.rfc-editor.org/rfc/rfc3339
- W3C PROV-DM: https://www.w3.org/TR/prov-dm/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html

### Candidate Implementation Sources

- OpenTelemetry Logs data model and specification: https://opentelemetry.io/docs/specs/otel/logs/
- Current Rust tracing documentation: https://docs.rs/tracing/
- Current Rust tracing-subscriber documentation: https://docs.rs/tracing-subscriber/
- Current PostgreSQL documentation: https://www.postgresql.org/docs/current/

Candidate implementations inform feasibility and correlation design only. The researcher must record exact versions and must not treat a telemetry library or storage feature as an audit requirement.

---

## 4. Supporting Resources

- IDR-SRV-019: Provenance, Lineage, Quality, and Trust Metadata Model
- IDR-SRV-020: Status, Availability, and System Event Model
- IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy
- IDR-SRV-030: Data Lifecycle, Retention, Archival, and Deletion Strategy
- IDR-SRV-031 through IDR-SRV-038 write, ingestion, streaming, command, feasibility, and safety topics
- IDR-SRV-039 and IDR-SRV-039A authentication, authorization, and threat topics
- IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints
- IDR-SRV-042: DDIL-Informed Server Semantics
- IDR-SRV-043: Server Synchronization and Conflict Handling Boundary
- IDR-SRV-047: Configuration, Secrets, and Environment Strategy
- IDR-SRV-048: Observability, Logs, Metrics, and Health Check Strategy
- IDR-SRV-049: Migration, Upgrade, Backup, and Restore Strategy
- OSH, Connected Systems Go, pygeoapi, SECD, and OS4CSAPI lessons, when reproducible and accessible

---

## 5. Research Methodology

### Phase 1: Source Freeze and Accountability Baseline (2-2.5 hours)

**Objective:** Establish authoritative obligations, precise terminology, and evidence limitations.

**Tasks:**
1. Record source authority, version or commit, access date, status, and accessibility.
2. Extract exact audit/accountability requirements and profile obligations with identifiers.
3. Define audit, accountability, provenance, domain event, request log, trace, metric, and security-alert boundaries.
4. Record conflicts and unavailable controlled material without substituting memory or secondary summaries.

**Expected Output:** A source table, requirement map, terminology crosswalk, and boundary model.

### Phase 2: Auditable-Event and Actor Analysis (3-3.5 hours)

**Objective:** Determine what must be recorded and who or what can be attributed.

**Tasks:**
1. Inventory API, administrative, ingestion, policy, security, lifecycle, command, DDIL, synchronization, and audit-administration actions.
2. Classify mandatory, profile-dependent, aggregate, sampled, and non-audited events with rationale.
3. Define actor, delegate, authority, source, automated-process, node, and unknown-identity representations.
4. Map events to subjects, outcomes, reason categories, and adjacent records.

**Expected Output:** An auditable-event taxonomy, deployment-profile matrix, and actor/delegation model.

### Phase 3: Record Schema and Sensitive-Data Controls (3-3.5 hours)

**Objective:** Define a stable, minimal, useful, and safe audit record.

**Tasks:**
1. Define mandatory and conditional event fields, identifiers, time fields, correlation, versioning, and handling labels.
2. Define phase/outcome semantics and relationships among multi-event operations.
3. Create prohibited-data, minimization, digest, redaction, and injection-prevention rules.
4. Map representative resource change, policy denial, command, lifecycle, and synchronization scenarios into records.

**Expected Output:** A versioned conceptual audit schema, field matrix, redaction rules, and worked scenarios.

### Phase 4: Integrity, Capture, Failure, and Access Analysis (3-4 hours)

**Objective:** Define how audit evidence remains available, reviewable, and appropriately protected.

**Tasks:**
1. Model threats to generation, transit, persistence, ordering, completeness, access, and verification.
2. Evaluate transaction coupling, durable outbox, local buffering, remote forwarding, append-only controls, checkpoints, hash chains, signatures, and independent verification by deployment profile.
3. Define fail-closed, buffer-and-continue, degraded read-only, gap, replay, and recovery behavior by event class.
4. Define access, search, review, annotation, export, and audit-of-audit behavior.

**Expected Output:** An integrity-control decision matrix, capture/failure model, and access/review/export contract.

### Phase 5: Lifecycle, DDIL, Observability, and Test Implications (2.5-3 hours)

**Objective:** Complete bounded handoffs and make the strategy falsifiable.

**Tasks:**
1. Define audit record classes and retention/archival/deletion requirements for IDR-SRV-030 without assigning unsupported periods.
2. Define local audit continuity and evidence fields needed by IDR-SRV-042 and synchronization constraints for IDR-SRV-043.
3. Define non-sensitive health, metrics, diagnostics, and alerts for IDR-SRV-048.
4. Build positive, negative, transaction-failure, tamper, access-control, DDIL, replay, and performance fixtures with expected results.

**Expected Output:** Lifecycle and downstream handoffs plus a complete verification matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Produce a decision-usable audit and accountability baseline.

**Tasks:**
1. Answer every core question with evidence and explicit limitations.
2. Resolve conflicts according to source authority or leave them explicitly unresolved.
3. Classify each recommendation as normative, profile-required, project decision, or implementation option.
4. Produce selected and rejected options, owners, prerequisites, review triggers, and implementation implications.

**Expected Output:** The completed audit logging and accountability strategy report.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Every core question is answered or explicitly unresolved with a next action.
- [ ] Every standards/profile assertion cites an exact requirement, control, clause, or controlled-document reference.
- [ ] Audit is clearly distinguished from provenance, domain events, request logs, metrics, traces, and security alerts.
- [ ] Mandatory and profile-dependent auditable events cover authentication, authorization, policy, resource changes, ingestion, command/control, lifecycle, synchronization, and audit administration.
- [ ] Actor, delegation, automated-process, authority, source, node, and uncertain-identity semantics are defined.
- [ ] The audit record schema identifies mandatory, conditional, prohibited, sensitive, and administrator-only fields.
- [ ] Outcome and phase semantics distinguish attempt, acceptance, commit, dispatch, completion, denial, rollback, failure, and unknown result.
- [ ] Secret and sensitive-payload exclusion, minimization, redaction, digest, and injection-prevention rules are testable.
- [ ] Integrity controls are selected against an explicit threat model and their limitations are documented.
- [ ] Audit capture transaction, ordering, buffering, backpressure, fail behavior, replay, gap, and recovery rules are defined by event class.
- [ ] Audit access, search, export, review, annotation, verification, and audit-of-audit behavior are defined.
- [ ] Retention is handed to IDR-SRV-030; DDIL semantics to IDR-SRV-042; synchronization mechanics to IDR-SRV-043; observability implementation to IDR-SRV-048.
- [ ] Positive, negative, boundary, failure, tamper, access, DDIL, replay, and performance fixtures have explicit expected outcomes.
- [ ] No recommendation overclaims tamper-proofing, identity certainty, non-repudiation, completeness, or legal sufficiency.
- [ ] The report is polished, recommendation-first, independently readable, and self-contained for the project lead, implementers, and later AI agents.

---

## 7. Deliverable

**Deliverable Name:** Audit Logging and Accountability Strategy Research Report
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-041-audit-logging-and-accountability-strategy-report.md`

**Required Content:**
1. Executive summary and decisions requested
2. Research-question coverage table
3. Source inventory with authority, version, access date, status, and limitations
4. Standards/profile requirement and control map
5. Audit/accountability terminology and adjacent-record boundary
6. Auditable-event taxonomy and deployment-profile matrix
7. Actor, delegate, authority, automated-process, source, and node model
8. Versioned conceptual audit record schema and field matrix
9. Outcome, phase, reason, time, ordering, and correlation semantics
10. Prohibited-data, minimization, redaction, digest, and injection controls
11. Threat model and integrity/tamper-evidence decision matrix
12. Capture, transaction, outbox, buffering, backpressure, failure, gap, and recovery model
13. Storage-role, access, search, review, annotation, export, and audit-of-audit model
14. Retention, archive, delete, backup, and restore handoff
15. DDIL continuity and synchronization handoff
16. Observability and alerting handoff
17. Worked end-to-end scenarios for resource change, policy denial, command lifecycle, disposition action, and synchronization conflict
18. Fixture and verification matrix with expected outcomes
19. Implementation implications, selected/rejected options, risks, unresolved questions, and review triggers
20. Validation against this plan's success criteria

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan and Glaux Server Goal and Definition must be current.
- IDR-SRV-019 and IDR-SRV-030 reports must be complete and accepted before starting unless an exception is approved and recorded under the overall-plan Governance Rules.
- IDR-SRV-036 through IDR-SRV-040 command, security, identity, authorization, and policy reports, including IDR-SRV-039A, must be complete and accepted before starting unless an exception is approved and recorded under the overall-plan Governance Rules.
- Official public guidance and project-available AEP/STANAG audit material must be accessible or explicitly unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- IDR-SRV-042: DDIL-Informed Server Semantics
- IDR-SRV-043: Server Synchronization and Conflict Handling Boundary
- IDR-SRV-047: Configuration, Secrets, and Environment Strategy
- IDR-SRV-048: Observability, Logs, Metrics, and Health Check Strategy
- IDR-SRV-049: Migration, Upgrade, Backup, and Restore Strategy
- IDR-SRV-050 through IDR-SRV-056 conformance, test architecture, fixtures, performance, security, and interoperability planning
- Final overall IDR synthesis

---

## 9. Research Status Checklist

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

- Open question: Which AEP-4789 or organizational sources define mandatory auditable events, retention classes, review expectations, or stronger evidence requirements?
- Open question: Which read and streaming accesses need per-item audit records versus bounded aggregate evidence?
- Open question: Which deployment profiles require independently verifiable or remotely forwarded evidence?
- Open question: Which command parameters or results can be represented by identifiers and digests while remaining useful for accountability?
- Risk: Copying request bodies into audit records would create a high-value secondary store of secrets and sensitive mission data.
- Risk: Allowing ordinary administrators to alter both server state and its audit evidence would defeat accountability.
- Risk: A universal fail-closed rule could make safe degraded operation impossible, while a universal continue rule could silently lose critical evidence.
- Risk: Hash chains or signatures could be misrepresented as proof that the original event, actor identity, or content was truthful.
- Decision trigger: Revisit the strategy when identity, policy, command safety, deployment topology, or adopted profile requirements change.

---

## References

- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- Initial Planning Guidance: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/initial-planning-guidance.md
- Testing research exemplar corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- NIST SP 800-53 Revision 5: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
- NIST SP 800-92: https://csrc.nist.gov/pubs/sp/800/92/final
- NIST SP 800-92 Revision 1 Initial Public Draft: https://csrc.nist.gov/pubs/sp/800/92/r1/ipd
- OWASP Logging Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
- RFC 5424: https://www.rfc-editor.org/rfc/rfc5424
- W3C PROV-DM: https://www.w3.org/TR/prov-dm/
- OpenTelemetry Logs specification: https://opentelemetry.io/docs/specs/otel/logs/
- OGC API - Connected Systems Part 1: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems Part 2: https://docs.ogc.org/is/23-002/23-002.html

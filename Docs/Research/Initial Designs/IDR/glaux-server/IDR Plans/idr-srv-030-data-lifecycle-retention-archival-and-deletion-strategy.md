# Section 030: Data Lifecycle, Retention, Archival, and Deletion Strategy - Research Plan

**Status:** Planned
**Last Updated:** July 29, 2026
**Estimated Research Time:** 14-18 hours
**Actual Research Time:** TBD until complete
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-030-data-lifecycle-retention-archival-and-deletion-strategy-report.md`

---

## Usage Instructions

Before executing this plan, review the exemplar corpus and align the report to the proven standards for content nature, type, level, and detail:

- Full research-plans corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- Blueprint-first exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md
- Inventory/sourcing rigor exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/15-fixture-sourcing-organization.md
- End-state synthesis exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/38-testing-playbook-synthesis.md

This plan is limited to research planning. It does not execute the research and does not produce the research report.

The resulting report must be polished, recommendation-first, independently readable, and usable as shared decision material by the project lead, implementers, and later AI agents. It must supply enough context to stand on its own, synthesize the evidence instead of mirroring the research questions, and clearly state findings, recommendations, implementation implications, and unresolved questions.

The report must distinguish standards requirements, adopted profile requirements, organizational or mission retention policy, Glaux design decisions, and storage-product capabilities. It must not invent a legal, NATO, mission, or classification retention period when no controlling authority has been supplied.

---

## 1. Research Objective

This topic research must define the Glaux Server planning baseline for **data lifecycle, retention, archival, and deletion** across canonical resources, dynamic data, command/control records, provenance, source documents, raw and quarantined inputs, validation artifacts, events, caches, indexes, exports, and backups.

The research must answer:

- What record classes Glaux Server creates or retains and which component or authority owns their lifecycle policy.
- How active, superseded, expired, stale, archived, held, tombstoned, deletion-pending, deleted, and sanitized states differ.
- Which data is mutable, append-only, replaceable, compactable, downsampled, exportable, restorable, or permanently removable.
- How retention and deletion preserve identifiers, relationships, provenance, auditability, command safety, and CSAPI behavior without keeping unrestricted data forever.
- How archival and restoration work under local, operational, tactical-edge, and DDIL conditions.
- How deletion propagates to related records, indexes, caches, outboxes, replicas, synchronization peers, exports, and backups.
- Which API-visible effects, administrative controls, persistence features, policy decisions, tests, and operational procedures are required.

The output must be a decision-ready lifecycle strategy containing a record-class inventory, policy-authority model, state and transition model, retention and disposition rules, archive/restore behavior, deletion/tombstone/sanitization rules, dependency and cascade behavior, DDIL/synchronization implications, and verifiable recommendations.

### Why This Topic Order

This topic closes Category E after:

- IDR-SRV-025: Database and Persistence Architecture Options
- IDR-SRV-026: Geospatial Storage and Query Strategy
- IDR-SRV-027: Time-Series Observation Storage Strategy
- IDR-SRV-028: Metadata and Document Storage Strategy
- IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy

Those topics establish storage roles and transactional boundaries. This topic defines how long each record class remains in each storage role, how records move or are transformed over time, and what deletion means across authoritative data, derived state, and recovery copies.

This topic must provide explicit handoffs to:

- IDR-SRV-031 through IDR-SRV-038 for ingestion buffers, replay, streaming, commands, status, results, tasking, safety, and command-audit records.
- IDR-SRV-039 through IDR-SRV-041 for security, policy, audit, and accountability constraints.
- IDR-SRV-042 and IDR-SRV-043 for offline retention and delayed deletion/synchronization behavior.
- IDR-SRV-045 through IDR-SRV-049 for deployment, configuration, observability, backup, restore, and operational implementation.

### Critical Constraints

- Do not invent retention periods. Every period or hold rule must trace to an adopted profile, supplied organizational/mission policy, legal requirement, operational objective, or explicit Glaux default decision.
- Do not treat HTTP DELETE as proof that every physical copy was immediately destroyed.
- Distinguish API deletion, logical deletion, tombstoning, archival, purge, backup expiry, media sanitization, and cryptographic erase.
- Do not delete evidence required to explain a command, security decision, provenance chain, audit event, synchronization conflict, or retention-policy action without explicit authority.
- Do not retain sensitive payloads merely because metadata, a hash, an aggregate, or a redacted record would satisfy the requirement.
- Do not assume one lifecycle applies to all resources, tenants, missions, source authorities, sensitivity levels, or deployment profiles.
- Do not silently cascade deletion across related resources when ownership or authority differs.
- Do not make derived indexes, caches, broker messages, outboxes, replicas, exports, or backups invisible to the deletion model.
- Do not design configuration and secret management here; those concerns belong to IDR-SRV-047.
- Do not design detailed backup technology here; define lifecycle requirements and hand implementation to IDR-SRV-049.
- Keep the strategy feasible for local development, public demonstration, reference deployment, and constrained tactical-edge profiles.

---

## 2. Research Questions

### Core Questions

1. What data and artifact classes does Glaux Server retain, and who has authority to set or override their lifecycle?
2. What lifecycle states, transitions, retention triggers, holds, archival actions, and disposition outcomes apply to each class?
3. How should deletion preserve referential integrity, provenance, auditability, safety, policy, and synchronization correctness?
4. How should archival, restoration, compaction, downsampling, export, backup expiry, and sanitization differ?
5. What API, persistence, operational, security, DDIL, and test contracts are required to implement the strategy?

### Detailed Questions

#### Record-Class Inventory

- Which canonical resources require current records, historical versions, or both?
- Which dynamic records require retention: observations, status values, temporal properties, system events, commands, command status/results, feasibility requests/status/results, and streaming events?
- Which documents and artifacts require retention: SensorML source documents, SWE schemas, OpenAPI/schema versions, controlled vocabulary snapshots, validation reports, raw inputs, rejected/quarantined payloads, normalized representations, and generated exports?
- Which operational records require retention: provenance activities, source offsets, idempotency records, conflict records, outbox/inbox records, synchronization state, audit records, and administrative decisions?
- Which data is authoritative, supporting evidence, derived index, cache, temporary work product, recovery copy, or external reference?
- Which classes can be reconstructed and which would be irrecoverable?

#### Policy Authority and Scope

- Which authority sets retention for standards obligations, AEP/profile obligations, organizational policy, mission policy, security/accountability, operational continuity, and user-requested deletion?
- How should defaults, per-class rules, per-source rules, tenant/mission overrides, holds, exceptions, and emergency actions be prioritized?
- What policy identity, version, effective time, approver, rationale, and review date must be recorded?
- How should sensitivity, releasability, source ownership, contractual restrictions, and mission value affect lifecycle decisions?
- Which policy details can be externally visible and which are administrative or sensitive?
- How are conflicts among deletion requests, holds, accountability requirements, and source authority surfaced and resolved?

#### Lifecycle States and Transitions

- What precise meanings apply to active, current, historical, superseded, expired, stale, inactive, retired, archived, held, quarantined, deletion-pending, tombstoned, purged, and sanitized?
- Which states describe resource semantics versus storage placement or administrative workflow?
- What events trigger each transition: time, resource state, source action, policy change, capacity threshold, mission closure, supersession, manual approval, or synchronization?
- Which transitions are reversible and which are terminal?
- What actor and authority may request, approve, cancel, or execute each transition?
- What audit and provenance evidence accompanies each transition?

#### Retention Triggers and Time Semantics

- Does retention begin at creation, phenomenon time, result time, ingestion time, publication time, supersession, retirement, mission completion, last access, or another explicit event?
- How should missing, disputed, future, or corrected timestamps affect retention calculations?
- How are time-zone, precision, and clock-skew rules inherited from IDR-SRV-018?
- How should a policy change affect existing records and already scheduled disposition?
- How should records under hold be excluded from automated disposition?
- How should retention deadlines and next actions be represented without exposing sensitive policy?

#### Historical Versions, Provenance, and Tombstones

- Which updates replace current state while preserving a prior version?
- Which append-only records must never be silently rewritten?
- What minimum tombstone remains after API-visible deletion: identifier, resource type, deletion time, authority, reason category, canonical link behavior, or provenance reference?
- When must a tombstone itself expire or be sanitized?
- How should lineage remain explainable when an input or subject record has been removed?
- Can hashes or metadata unintentionally reveal deleted sensitive information?
- How should canonical URIs and relationship traversal behave for archived, tombstoned, or purged resources?

#### Referential and Cascade Behavior

- What parent/child, composition, aggregation, association, source, derivation, and command relationships constrain deletion?
- Which deletions are restricted, cascaded, detached, tombstoned, or converted to unresolved references?
- How should deleting a system affect procedures, deployments, datastreams, observations, control streams, commands, events, and features of interest?
- How should deleting a source document affect normalized resources derived from it?
- How should policy ownership prevent one actor from deleting data owned by another source or tenant?
- What transaction and recovery behavior is needed for partial cascade failure?

#### Dynamic Data, Compaction, and Downsampling

- Which observations and status histories require full-fidelity retention, downsampling, aggregation, compaction, or expiry?
- How should aggregates retain provenance, uncertainty, method, input interval, and policy identity?
- Which late or corrected values require rebuilding aggregates or preventing premature deletion?
- How should latest-value materializations behave after source data expires?
- What is the lifecycle of stream cursors, publication messages, outbox/inbox entries, replay buffers, and consumer acknowledgements?
- How do rate, volume, query SLAs, storage capacity, and DDIL replay needs inform but not dictate policy?

#### Command, Feasibility, and Safety Records

- Which command definitions, submissions, parameters, status histories, results, feasibility records, cancellations, failures, and unknown outcomes require preservation?
- What minimum evidence is necessary for accountability and safety review without retaining unnecessary sensitive content?
- How should command records be redacted, archived, or restricted before eventual disposition?
- Can command lifecycle records be deleted while related audit or provenance evidence remains?
- How should simulated or test commands be distinguished and retained under separate profiles?
- How should unresolved or delayed DDIL command outcomes affect deletion eligibility?

#### Raw, Rejected, and Quarantined Data

- When must original raw payloads be retained for replay, validation, provenance, incident review, or interoperability diagnosis?
- When should rejected or malicious payloads be minimized, isolated, encrypted, sampled, or deleted rapidly?
- What access and handling restrictions apply to quarantine data?
- How should a retained hash or diagnostic avoid leaking secrets or personal/sensitive content?
- How should repaired and reprocessed records link to the original artifact and validation result?
- What criteria allow raw data to be replaced by a normalized record plus sufficient evidence?

#### Archive and Restore

- What makes an archive authoritative, discoverable, immutable, portable, encrypted, integrity-checked, and restorable?
- What metadata must accompany an archive: manifest, schema/profile versions, checksums, provenance, policy, sensitivity, identifiers, and dependencies?
- Which APIs or administrative paths expose archived resources or restoration state?
- How are archive jobs, failures, retries, partial archives, and integrity checks represented?
- What restore objectives, validation, conflict checks, identifier preservation, and reindexing are required?
- How are offline/tactical archives synchronized or reconciled after reconnect?

#### Delete, Purge, and Sanitization

- What does successful API deletion guarantee to a client at response time?
- What work may remain for asynchronous purge, replicas, search indexes, object stores, event logs, caches, exports, and backups?
- How should pending, completed, failed, or partially propagated deletion be recorded and monitored?
- When is cryptographic erase applicable, and what key-scope and shared-data constraints make it unsafe or ineffective?
- When does media sanitization apply to deployment or decommissioning rather than record-level deletion?
- What proof of disposition is appropriate without claiming impossible certainty?

#### Backups, Replicas, Exports, and Synchronization

- How do retention and holds apply to backups, point-in-time recovery logs, replicas, federation copies, user exports, and disaster-recovery media?
- How should deletion propagate to synchronization peers, especially under DDIL conditions?
- What tombstone or deletion marker prevents removed data from reappearing during replay?
- How should conflicting deletion and update operations be resolved?
- What happens when a peer or backup is unavailable past the desired deletion deadline?
- Which external-copy limitations must be disclosed rather than overstated?

#### API and Administrative Behavior

- Which lifecycle information is standards-facing, a Glaux extension, or administrator-only?
- What errors or problem details apply to held, archived, restricted, already-deleted, or non-restorable records?
- Which lifecycle actions should be synchronous, asynchronous, batchable, or approval-gated?
- What idempotency and concurrency rules apply to repeated disposition requests?
- How should dry-run, impact preview, and dependency enumeration reduce unsafe deletion?
- What rate limits and authorization checks protect bulk archival, export, restoration, or purge?

#### Testing and Operational Verification

- What fixture corpus covers normal expiry, hold, release, archive, restore, tombstone, cascade restriction, partial failure, backup expiry, and DDIL replay?
- How can tests use accelerated clocks without changing policy semantics?
- What evidence demonstrates that expired data is absent from authoritative queries, indexes, caches, and replay paths?
- What tests demonstrate that held data is preserved?
- What restoration tests verify identifiers, relationships, schemas, provenance, and query behavior?
- What failure-injection tests cover unavailable archives, partial cascades, invalid manifests, clock skew, and reconnect?
- What operational measures show backlog, upcoming disposition, failures, storage impact, and policy coverage without exposing sensitive details?

---

## 3. Primary Resources

### Project and Governance Sources

- Glaux Server Overall IDR Research Plan
- Glaux Server Goal and Definition
- Research Planning Approach
- Research Report Template
- IDR-SRV-015 through IDR-SRV-029 reports, when available
- Project-available AEP-4789 and STANAG 4789 policy or records-management material, with identity, edition, date, approval status, handling restrictions, and repository location recorded

### Standards and Protocol Sources

- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC SensorML 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- RFC 9110, HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457, Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- W3C PROV-DM: https://www.w3.org/TR/prov-dm/

### Lifecycle, Security, and Sanitization Sources

- NIST SP 800-88 Revision 2, Guidelines for Media Sanitization: https://csrc.nist.gov/pubs/sp/800/88/r2/final
- NIST SP 800-53 Revision 5, especially Audit and Accountability, Contingency Planning, Media Protection, and System and Information Integrity controls: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
- Current PostgreSQL documentation for retention-supporting mechanisms, point-in-time recovery, partitioning, and vacuum behavior: https://www.postgresql.org/docs/current/
- Current PostGIS and selected time-series/storage product documentation only for verified implementation capabilities, with product version recorded

If OGC API - Features create/replace/update/delete drafts or other candidate specifications are used, the report must record their exact publication status and must not present draft behavior as a CSAPI requirement.

---

## 4. Supporting Resources

- IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy
- IDR-SRV-018: Temporal, Validity, and Freshness Model
- IDR-SRV-019: Provenance, Lineage, Quality, and Trust Metadata Model
- IDR-SRV-020: Status, Availability, and System Event Model
- IDR-SRV-025: Database and Persistence Architecture Options
- IDR-SRV-026: Geospatial Storage and Query Strategy
- IDR-SRV-027: Time-Series Observation Storage Strategy
- IDR-SRV-028: Metadata and Document Storage Strategy
- IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy
- IDR-SRV-031 through IDR-SRV-038 ingestion, dynamic-data, streaming, tasking, safety, and command-audit topics
- IDR-SRV-039 through IDR-SRV-043 security, policy, audit, DDIL, and synchronization topics
- IDR-SRV-047: Configuration, Secrets, and Environment Strategy
- IDR-SRV-048: Observability, Logs, Metrics, and Health Check Strategy
- IDR-SRV-049: Migration, Upgrade, Backup, and Restore Strategy

Storage product documentation is implementation evidence, not policy authority. Legal or mission retention advice is out of scope unless the project supplies a controlling source.

---

## 5. Research Methodology

### Phase 1: Source and Policy-Authority Baseline (2-2.5 hours)

**Objective:** Establish what can legitimately determine lifecycle behavior.

**Tasks:**
1. Record source identity, authority, version, access date, and availability.
2. Extract exact standards/profile requirements related to update, deletion, history, provenance, audit, and sanitization.
3. Identify supplied organizational, mission, security, and operational policy inputs.
4. Create a policy-authority and precedence model; record missing authorities as unresolved rather than inventing periods.

**Expected Output:** A source table, requirement map, policy-authority model, and evidence-gap list.

### Phase 2: Record-Class and State Inventory (3-3.5 hours)

**Objective:** Define what is governed and the lifecycle states it can occupy.

**Tasks:**
1. Inventory authoritative, supporting, derived, cached, temporary, recovery, export, and external record classes.
2. Map each class to owner, sensitivity, provenance dependency, mutability, reconstructability, and storage role.
3. Define lifecycle states and transition triggers without assigning unsupported durations.
4. Identify append-only, hold-sensitive, command-safety, audit, and source-fidelity classes.

**Expected Output:** A record-class inventory, lifecycle state model, and owner/authority matrix.

### Phase 3: Retention, Archive, and Disposition Rules (3-4 hours)

**Objective:** Define policy mechanics and observable behavior.

**Tasks:**
1. Define retention start events, deadline calculation, policy changes, holds, overrides, and review behavior.
2. Analyze version, tombstone, referential, cascade, compaction, downsampling, and aggregation rules.
3. Define archive manifests, integrity checks, restoration, and failure behavior.
4. Distinguish logical deletion, purge, cryptographic erase, backup expiry, and media sanitization.

**Expected Output:** A lifecycle decision matrix, archive/restore contract, and deletion/disposition model.

### Phase 4: Distributed Copies and Operational Boundaries (2.5-3 hours)

**Objective:** Account for every material copy and constrained operating mode.

**Tasks:**
1. Map lifecycle actions across indexes, caches, brokers, outboxes, replicas, exports, backups, and synchronization peers.
2. Define deletion markers, replay prevention, partial propagation, conflict, and unavailable-peer behavior.
3. Analyze DDIL archive, local retention, delayed deletion, and reconnect implications.
4. Define API-visible guarantees separately from asynchronous operational completion.

**Expected Output:** A copy-propagation matrix, DDIL/synchronization rules, and API guarantee table.

### Phase 5: Verification and Implementation Implications (2-3 hours)

**Objective:** Make every adopted lifecycle rule testable.

**Tasks:**
1. Define positive, negative, boundary, hold, archive/restore, deletion, and failure-injection fixtures.
2. Define verification for queries, indexes, caches, replay paths, replicas, and backup expiry.
3. Identify persistence, scheduling, authorization, audit, observability, and Rust implementation implications.
4. Define operational measures and safe administrative controls, including impact preview and dry-run needs.

**Expected Output:** A verification matrix, fixture inventory, and implementation implication table.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Produce a decision-usable lifecycle strategy.

**Tasks:**
1. Answer each core question with evidence and explicit limitations.
2. Resolve source conflicts by authority or preserve them as unresolved.
3. Classify each recommendation as normative, profile-required, policy-supplied, project decision, or implementation option.
4. Produce recommendations, rejected alternatives, owners, downstream handoffs, and review triggers.

**Expected Output:** The completed data lifecycle, retention, archival, and deletion strategy report.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Every core question is answered or explicitly unresolved with a next action.
- [ ] Standards and profile claims cite exact requirements or controlled-document references.
- [ ] No retention period or legal/mission obligation appears without an identified authority.
- [ ] Every server data/artifact class has an owner, storage role, lifecycle state model, and disposition outcome or is explicitly not applicable.
- [ ] Active, superseded, expired, archived, held, tombstoned, purged, backup-expired, and sanitized states are distinguished.
- [ ] Retention triggers, holds, policy precedence, policy change, and deadline behavior are defined.
- [ ] Referential, cascade, provenance, audit, command-safety, and tombstone rules are defined.
- [ ] Archive manifests, integrity checks, restoration validation, and failure behavior are testable.
- [ ] API deletion guarantees are distinguished from asynchronous purge and physical-media sanitization.
- [ ] Indexes, caches, brokers, outboxes, replicas, exports, backups, and synchronization peers are included.
- [ ] DDIL and reconnect behavior prevents deleted data from silently reappearing.
- [ ] Positive, negative, boundary, hold, restore, partial-failure, and replay fixtures have explicit expected outcomes.
- [ ] Configuration, backup technology, audit implementation, and synchronization details are handed to their owning topics without duplication.
- [ ] Recommendations are feasible for the Glaux deployment profiles and decision-usable.
- [ ] The report is polished, recommendation-first, independently readable, and self-contained for the project lead, implementers, and later AI agents.

---

## 7. Deliverable

**Deliverable Name:** Data Lifecycle, Retention, Archival, and Deletion Strategy Research Report
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-030-data-lifecycle-retention-archival-and-deletion-strategy-report.md`

**Required Content:**
1. Executive summary and decisions requested
2. Research-question coverage table
3. Source inventory with authority, version, access date, and limitations
4. Standards/profile requirement map and policy-authority model
5. Complete data and artifact record-class inventory
6. Lifecycle terminology, state model, transitions, actors, and authority
7. Retention triggers, holds, overrides, and policy-change behavior
8. Per-class lifecycle and disposition decision matrix
9. Historical-version, provenance, tombstone, and referential rules
10. Dynamic-data compaction, aggregation, and downsampling rules
11. Command, feasibility, audit, raw, rejected, and quarantine-data treatment
12. Archive manifest, integrity, restore, and failure contract
13. Delete, purge, cryptographic erase, backup expiry, and sanitization distinctions
14. Index, cache, broker, replica, export, backup, and synchronization propagation matrix
15. API and administrative behavior, guarantees, errors, and authorization needs
16. Fixture and verification matrix with expected outcomes
17. Implementation and operational implications
18. Recommendations, rejected options, unresolved questions, and review triggers
19. Validation against this plan's success criteria

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan and Glaux Server Goal and Definition must be current.
- IDR-SRV-015 through IDR-SRV-029 reports must be available, or unavailable inputs must be recorded as specific limitations.
- Official CSAPI Parts 1 and 2, current NIST sanitization guidance, and project-available AEP/profile policy sources must be accessible or explicitly unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- IDR-SRV-031 through IDR-SRV-038 ingestion, streaming, command, status, feasibility, safety, and command-audit lifecycle decisions
- IDR-SRV-041: Audit Logging and Accountability Strategy
- IDR-SRV-042 and IDR-SRV-043 DDIL and synchronization deletion behavior
- IDR-SRV-047 through IDR-SRV-049 configuration, observability, migration, backup, restore, and operational implementation
- IDR-SRV-050 through IDR-SRV-055 conformance, fixtures, performance, and security testing
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

- Open question: Which organizational, mission, profile, or legal authorities will supply actual retention periods and hold rules?
- Open question: Which CSAPI resources should remain discoverable as tombstones after deletion?
- Open question: Which original source artifacts require byte-preserving retention for interoperability or accountability?
- Open question: What restore objectives are required for tactical-edge versus reference-deployment profiles?
- Risk: A generic delete flag could leave sensitive copies in indexes, replay paths, exports, and backups while overstating deletion.
- Risk: Removing tombstones too early could resurrect data during synchronization or break provenance.
- Risk: Retaining raw, rejected, command, or audit payloads indefinitely could create avoidable security exposure.
- Decision trigger: Revisit lifecycle rules when an adopted AEP profile, mission policy, deployment topology, or storage architecture changes.

---

## References

- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- Initial Planning Guidance: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/initial-planning-guidance.md
- Testing research exemplar corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- OGC API - Connected Systems Part 1: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems Part 2: https://docs.ogc.org/is/23-002/23-002.html
- W3C PROV-DM: https://www.w3.org/TR/prov-dm/
- NIST SP 800-88 Revision 2: https://csrc.nist.gov/pubs/sp/800/88/r2/final
- NIST SP 800-53 Revision 5: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
- RFC 9110: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457: https://www.rfc-editor.org/rfc/rfc9457

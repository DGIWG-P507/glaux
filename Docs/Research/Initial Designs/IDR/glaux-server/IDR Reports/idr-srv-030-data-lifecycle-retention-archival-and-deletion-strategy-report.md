# Section 030: Data Lifecycle, Retention, Archival, and Deletion Strategy - Research Report

**Topic ID:** IDR-SRV-030<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-030 Data Lifecycle, Retention, Archival, and Deletion Strategy](../IDR%20Plans/idr-srv-030-data-lifecycle-retention-archival-and-deletion-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions, all detailed question groups, all 6 methodology phases, all 15 success criteria, and the complete record-class and copy-propagation inventories<br>
**Methodology Used:** Authority-ranked extraction from approved OGC and IETF standards, W3C PROV, current NIST guidance, the controlled AEP source through accepted project findings, and accepted IDR-SRV-001 through IDR-SRV-029; direct review of current PostgreSQL and representative versioned-object-store documentation; lifecycle, authority, copy, failure, and verification modeling; and bounded synthesis without inventing retention periods<br>
**Research Time:** Approximately 17 hours of AI-assisted execution on September 14, 2026<br>
**Official Standards Source Pin:** [opengeospatial/ogcapi-connected-systems v1.0.0 at 8e03b236a049849f2ccc24b4fd9fdce5ff69bed2](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Shared Register Baseline:** [OGC API - Connected Systems upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), Version 1.10; upstream master remained 3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f on September 14, 2026<br>
**Technology Documentation Snapshot:** PostgreSQL 18.6 and current Amazon S3 versioning documentation, checked September 14, 2026 and treated as mutable implementation evidence<br>
**Controlled AEP Source:** AC/224(JCGISR)D(2026)0005, April 27, 2026, SHA-256 56dc757b6e677b3584e3152a957849f21a24b22854f562613ff283a8b599da8c; used only through accepted project findings and not redistributed<br>
**Document Purpose:** Establish the Glaux Server lifecycle-policy, retention-trigger, hold, archive/restore, compaction, deletion, tombstone, purge, backup-expiry, sanitization, DDIL, administration, and verification baseline without supplying unsupported retention periods, implementing draft Part 3, or implementing the server<br>
**Author:** OpenAI Codex<br>
**Accepted By:** TBD pending Glaux Project Lead review<br>
**Acceptance Date:** TBD pending acceptance<br>
**Date:** September 14, 2026<br>
**Last Updated:** September 14, 2026

---

## Reading Guide and Evidence Labels

| Label | Meaning |
|---|---|
| **N** | Normative or standards-derived behavior |
| **A** | Project-controlling AEP/STANAG adoption or operational-context finding |
| **P** | Glaux architecture direction or recommendation proposed for acceptance here |
| **I** | Informative technology, implementation, test, or operational evidence |
| **D** | Detailed design deferred to the named downstream topic |
| **X** | Evidence gap or unresolved decision requiring authorized policy, profile, prototype, or benchmark input |

Retention is not a property of a table and deletion is not one event. This report separates semantic resource state, administrative custody, storage placement, logical/API visibility, asynchronous copy disposition, recovery-copy expiry, and physical-media sanitization. No duration in this report is a legal, NATO, mission, classification, or organizational obligation.

---

## Table of Contents

1. Executive Summary and Decisions Requested
2. Research-Question Coverage Table
3. Source Inventory with Authority, Version, Access Date, and Limitations
4. Standards/Profile Requirement Map and Policy-Authority Model
5. Complete Data and Artifact Record-Class Inventory
6. Lifecycle Terminology, State Model, Transitions, Actors, and Authority
7. Retention Triggers, Holds, Overrides, and Policy-Change Behavior
8. Per-Class Lifecycle and Disposition Decision Matrix
9. Historical-Version, Provenance, Tombstone, and Referential Rules
10. Dynamic-Data Compaction, Aggregation, and Downsampling Rules
11. Command, Feasibility, Audit, Raw, Rejected, and Quarantine-Data Treatment
12. Archive Manifest, Integrity, Restore, and Failure Contract
13. Delete, Purge, Cryptographic Erase, Backup Expiry, and Sanitization Distinctions
14. Index, Cache, Broker, Replica, Export, Backup, and Synchronization Propagation Matrix
15. API and Administrative Behavior, Guarantees, Errors, and Authorization Needs
16. Fixture and Verification Matrix with Expected Outcomes
17. Implementation and Operational Implications
18. Recommendations, Rejected Options, Unresolved Questions, and Review Triggers
19. Validation Against This Plan's Success Criteria
20. References and Sources

---

## 1. Executive Summary and Decisions Requested

Glaux should adopt a **versioned, policy-driven lifecycle service** over the PostgreSQL-centered authority selected by IDR-SRV-025. Every governed record belongs to a named record class and is evaluated under an immutable policy version with a traceable authority, scope, trigger, hold behavior, permitted transformations, disposition action, copy-propagation contract, and evidence requirement. Policies may differ by class, tenant, mission, source authority, sensitivity, deployment profile, and purpose. A missing or conflicting authority must stop destructive automation and create an exception; it must never be filled with an invented duration. **[P]**

No numerical retention periods are selected. The controlled AEP findings available to this project add operational, DDIL, robustness, command, provenance, and accountability context but do not supply a complete lifecycle schedule. NIST SP 800-53 is a flexible control catalog whose controls are selected and tailored from mission, business, legal, policy, and risk inputs; it is not itself a Glaux retention schedule.[^1] The project therefore needs an approved lifecycle-policy package before production automated disposition can be enabled. **[A/I/X]**

Glaux should model lifecycle through **orthogonal dimensions**, not one overloaded status field:

- semantic state: current, superseded, expired, stale, inactive, or retired;
- custody state: normal, quarantined, restricted, or held;
- storage state: online, archive-pending, archived, restore-pending, or restored;
- disposition state: retained, eligible, deletion-pending, tombstoned, purged, or backup-expired; and
- media state: in service, clear/purge/destroy pending, or sanitized under the deployment's approved media program.

A record can therefore be both retired and held, archived and restricted, or tombstoned while recovery copies await policy expiry. A hold overlays and suspends disposition; it does not reactivate the resource or silently extend unrelated data. **[P]**

HTTP DELETE must mean removal of the resource's association with current API functionality, not an assertion that all representations and storage have been destroyed. RFC 9110 explicitly leaves representation destruction and storage reclamation to the implementation.[^2] Glaux should return 202 when an accepted deletion remains asynchronous, 204 only when the promised API-visible action has been enacted, and an authorization-safe 404 or 410 for later reads according to disclosure policy. A disposition-status resource available to authorized administrators should describe copy-by-copy progress and limitations. **[N/P]**

Approved CSAPI Parts 1 and 2 require specific API-level cascade behavior. Default deletion of a System with nested resources or a Deployment association is rejected; requested cascade deletes the System and nested resources and removes the Deployment association. Default deletion of a nonempty DataStream or ControlStream is rejected with 409; requested cascade deletes the stream and nested Observations or Commands. These operations remove CSAPI resources but do not override holds, foreign ownership, command-safety evidence, provenance, audit, backup, or sanitization obligations. When those obligations prevent the requested public effect, Glaux must reject or stage the operation rather than silently violating either authority. Command cancellation remains an appended CANCELED status, not Command deletion. **[N/P]**

Historical versions, provenance, validation, command status, system events, lifecycle events, audit references, and synchronization conflicts are append-oriented evidence. They may be archived, redacted, minimized, or ultimately disposed only under explicit authority. W3C PROV invalidation describes the start of destruction, cessation, or expiry of an entity; it does not prove physical erasure.[^3] When payload erasure is authorized but lineage must remain explainable, Glaux should retain a policy-safe placeholder carrying only the minimum permitted identity, relation, activity, authority class, and unavailability reason. Hashes are not automatically safe metadata and must not be retained where they leak or enable correlation. **[N/P]**

Dynamic-data capacity mechanisms are transformations, not policy. Lossless compression preserves the same information. Aggregation and downsampling create new derived records with method and software version, source interval, input count/coverage, units and observed-property bindings, quality/nil/uncertainty treatment, policy identity, watermarks, and provenance. Raw inputs may be purged after verification only when an approved rule says the aggregate is an adequate substitute. Status categories, discrete events, qualitative values, nil reasons, command data, and safety outcomes must not be numerically averaged. **[P]**

Archive publication should be transactional at the catalog boundary: write to staging, construct a reference-closed manifest, verify bytes/digests/counts/schema and relationship closure, then atomically publish the catalog entry. Restore occurs into an isolated namespace, validates the manifest and packages, preserves identifiers and provenance, applies migrations, reapplies the current deletion/tombstone ledger before serving, resolves policy and conflict changes, and rebuilds disposable indexes and caches. A partial or unverified archive is failed work, never authoritative. **[P]**

PostgreSQL DELETE does not immediately remove an old row version; VACUUM later makes dead-row space reusable, and ordinary VACUUM generally does not return it to the operating system.[^4] PostgreSQL PITR combines base backups and retained WAL capable of reconstructing prior database states.[^5] Versioned object stores may add a delete marker while retaining prior versions.[^6] Consequently, the lifecycle service must inventory and track authoritative rows, artifacts, replicas, indexes, caches, outboxes, broker copies, exports, backups, WAL, and DDIL peers separately.

Media sanitization is a deployment and media-management operation. NIST SP 800-88 Revision 2 defines it as making access to target data infeasible for a given effort and assigns organizations and system owners responsibility for a sensitivity-informed sanitization program.[^7] Cryptographic erase is acceptable only when approved for the media and threat model and when a key exclusively covers the targeted data plus every relevant copy; shared-key, derived-key, escrow, replica, backup, or external-copy ambiguity defeats a per-record erasure claim. **[I/P]**

### Decisions requested from the Glaux Project Lead

1. Adopt the policy schema, orthogonal lifecycle model, fail-closed authority behavior, and copy-by-copy disposition model in this report.
2. Adopt the archive/restore contract, including mandatory reapplication of deletion/tombstone state before restored data becomes queryable.
3. Adopt the distinction between CSAPI/API deletion and asynchronous purge, backup expiry, cryptographic erase, and media sanitization.
4. Adopt dependency-aware retention, policy-safe tombstones, DDIL anti-resurrection behavior, and evidence minimization.
5. Direct the project/profile authority to supply actual retention/hold rules and restore objectives; keep production automatic purge disabled for classes without an approved rule.
6. Preserve the downstream boundaries identified here. Acceptance of this report closes Category E but does not authorize IDR-SRV-031, draft Part 3 implementation, or server implementation.

---

## 2. Research-Question Coverage Table

| Plan question | Short form | Status | Evidence location |
|---|---|---|---|
| Q1 | What classes are retained, and who controls lifecycle? | Complete; actual schedules explicitly unresolved | Sections 4-5, 8 |
| Q2 | What states, triggers, holds, archive actions, and outcomes apply? | Complete | Sections 6-8, 12-13 |
| Q3 | How does deletion preserve integrity, provenance, audit, safety, policy, and synchronization? | Complete; detailed audit/sync schemas delegated | Sections 9, 11, 14-15 |
| Q4 | How do archive, restore, compaction, export, backup expiry, and sanitization differ? | Complete; technology selection delegated | Sections 10, 12-14 |
| Q5 | What API, persistence, operations, security, DDIL, and test contracts follow? | Complete | Sections 14-18 |

### 2.1 Scope and accepted-baseline reconciliation

This report executes IDR-SRV-030, the sixth and final Category E research topic. It governs all server-held or server-generated records and material copies. It does not select production retention periods, claim legal advice, define a final SQL schema, select backup products, define cryptographic key management, finalize audit or synchronization schemas, implement draft Part 3, or implement the server.

Accepted upstream decisions remain intact:

- IDR-SRV-015 supplies the canonical aggregate graph; authority is not inferred from representation.
- IDR-SRV-016 keeps ResourceId, revision, ETag, idempotency, domain-event, message, storage, and delivery identities distinct and forbids identifier reuse.
- IDR-SRV-017 supplies typed relationship ownership and prohibits silent cascade across non-owning/external edges.
- IDR-SRV-018 separates domain, result, receipt, ingest, transaction, publication, validation, and observation clocks.
- IDR-SRV-019 preserves provenance, quality, trust, and policy-safe unavailability.
- IDR-SRV-020 keeps status, domain event, audit event, outbox item, and transport message distinct.
- IDR-SRV-021 through 024 preserve exact source, parsed, canonical, generated, validation, contract, and semantic-package layers.
- IDR-SRV-025 through 028 select PostgreSQL/PostGIS authority, typed temporal stores, immutable artifact catalog/content addressing, disposable derived state, and dependency-rooted mark-and-sweep.
- IDR-SRV-029 supplies atomic policy/lifecycle facts and outbox work, optimistic preconditions, idempotency, effectively-once local effects, at-least-once delivery, and explicit DDIL conflicts/tombstones.

---

## 3. Source Inventory with Authority, Version, Access Date, and Limitations

### 3.1 Primary and controlling sources

| Source | Type | Version/status | Authority | Stable anchor | Access date | Availability/limitation |
|---|---|---|---|---|---|---|
| OGC API - Connected Systems Part 1 | Standard/repository | OGC 23-001, approved 1.0; v1.0.0, commit 8e03b236... | Normative | /req/create-replace-delete/system-delete-cascade; CRUD resource classes | 2026-09-14 | API behavior; no physical-retention schedule |
| OGC API - Connected Systems Part 2 | Standard/repository | OGC 23-002, approved 1.0; same source pin | Normative | DataStream/ControlStream delete-cascade and schema-update requirements; Command cancellation note/status | 2026-09-14 | API behavior; no archival, purge, backup, or sanitization schedule |
| OGC SensorML 3.0 and SWE Common 3.0 | Standards | OGC 23-000 and 24-014 | Normative representations/contracts | Accepted IDR-SRV-021/022 extraction | 2026-09-14 | No server retention schedule |
| RFC 9110, HTTP Semantics | IETF Standards Track | June 2022 | Normative HTTP | §9.3.5 DELETE and response semantics | 2026-09-14 | Does not define physical deletion |
| RFC 9457, Problem Details | IETF Standards Track | July 2023 | Normative error representation | §§3-5 | 2026-09-14 | Does not define lifecycle state or status codes |
| W3C PROV-DM | W3C Recommendation | April 30, 2013 | Normative provenance model | §§5.1.8, 5.2 | 2026-09-14 | Domain-neutral; no retention periods |
| NIST SP 800-88 Revision 2 | NIST guidance | Final September 26, 2025; FAQ planning note July 17, 2026 | Authoritative guidance when adopted | §§3-4 and official publication abstract | 2026-09-14 | Media sanitization guidance, not per-record SQL deletion or a Glaux schedule |
| NIST SP 800-53 Revision 5 | NIST control catalog | Release 5.2.0, August 27, 2025 | Authoritative control catalog when selected/tailored | AU, CP, MP, SI families and catalog scope | 2026-09-14 | Flexible catalog; no automatic Glaux control baseline or durations |
| Accepted IDR-SRV-001 through 029 | Project reports | Accepted through 2026-09-14 | Project-controlling | Linked reports in Section 20 | 2026-09-14 | Detailed downstream owners remain binding |
| Controlled AEP source | Controlled project source | AC/224(JCGISR)D(2026)0005, 2026-04-27; recorded SHA-256 | Project-controlled | Accepted findings only | 2026-09-14 | Source not reproduced; available findings contain no complete retention schedule |

### 3.2 Mutable implementation evidence

| Source | Version/retrieval | Finding used | Authority limit |
|---|---|---|---|
| PostgreSQL routine vacuuming | 18.6 docs, checked 2026-09-14 | UPDATE/DELETE leaves old row versions; VACUUM reclaims for reuse; ordinary VACUUM usually does not return space to OS | Product behavior, not erasure policy |
| PostgreSQL table partitioning | 18.6 docs, checked 2026-09-14 | Detach/drop can remove old partitions much faster than bulk DELETE and avoid its VACUUM overhead | Mechanism only; partition key is not a retention rule |
| PostgreSQL backup/PITR | 18.6 docs, checked 2026-09-14 | Base backups plus WAL can restore whole-cluster prior states; incremental backups have dependency chains | Capability and operational constraint; backup design owned by IDR-SRV-049 |
| Amazon S3 DeleteObject/delete markers | Current docs, checked 2026-09-14 | Versioned-bucket DELETE normally creates a marker and retains prior versions; version-specific deletion is separate | Representative S3 semantics only; Glaux does not require AWS |

### 3.3 Evidence gaps and confidence

The standards strongly support the API-versus-storage distinction, exact cascade boundaries, provenance preservation, and sanitization terminology. PostgreSQL and object-store documentation strongly support the copy-aware design. Confidence is high in the architecture and verification recommendations.

Confidence is intentionally not converted into invented policy. No supplied authority establishes actual online retention, archive residence, hold, tombstone, replay, backup, WAL, quarantine, audit, command, or sanitization durations; tenant/mission precedence rules; mandatory RPO/RTO; or approved media-specific sanitization techniques. Those are explicit policy/package and deployment inputs. OGC API - Features Part 4 remains a referenced dependency of approved CSAPI CRUD text but is not elevated here into an independently approved Glaux retention authority.

---

## 4. Standards/Profile Requirement Map and Policy-Authority Model

### 4.1 Requirement map

| Source anchor | Source requirement/finding | Lifecycle consequence | Classification |
|---|---|---|---|
| CSAPI P1 /req/create-replace-delete/system-delete-cascade | Reject default System delete when nested resources or Deployment association exist; with cascade delete the System/nested resources and remove Deployment association | Implement exact visible graph effect, but evaluate holds, ownership, and evidence before accepting | N/P |
| CSAPI P2 /req/create-replace-delete/datastream-delete-cascade | Return 409 for default delete of nonempty DataStream; cascade deletes DataStream and nested Observations | Impact graph must include all Observations and their derived/copy dependencies | N/P |
| CSAPI P2 /req/create-replace-delete/controlstream-delete-cascade | Return 409 for default delete of nonempty ControlStream; cascade deletes ControlStream and nested Commands | Command-safety/evidence policy may require rejection, staging, redaction, or retained restricted evidence | N/P |
| CSAPI P2 Command cancellation | Cancellation is a new final CANCELED status; it is not Command DELETE | Never implement cancellation as content deletion or history rewrite | N |
| CSAPI P2 stream-schema update constraints | Schema change after children exists is rejected with 409 | Contract history remains bound to children; replacement does not erase historical interpretation | N |
| RFC 9110 §9.3.5 | DELETE removes URI/current-function association; content/storage may remain; 202/204/200 distinguish enactment | Document exact API guarantee and use asynchronous status for copy disposition | N/P |
| W3C PROV-DM | Derivation, revision, agents/activities and invalidation model provenance | Preserve explainable lineage or policy-safe unavailability; invalidation is not physical proof | N/P |
| NIST SP 800-88r2 | Sanitization is a sensitivity-informed program making target-data access infeasible for a given effort | Separate media sanitization and cryptographic erase from database/API delete | I/P |
| NIST SP 800-53r5.2.0 | Controls are flexible and tailored from mission/business/legal/policy/risk inputs | Adopt only the selected deployment/profile controls; do not treat catalog as schedule | I/P |
| Accepted IDR baselines | Typed identity/graph/time/provenance/evidence/storage/transaction boundaries | Lifecycle actions must preserve these boundaries and emit immutable evidence | P |

### 4.2 Policy authority and precedence

Glaux should resolve policy using applicable, authenticated rules rather than a hard-coded universal hierarchy. Each rule records:

| Policy field | Required meaning |
|---|---|
| Policy identity/version | Immutable ID and revision; no in-place rewrite |
| Source authority | Adopted law/regulation, contract/source restriction, classification/releasability rule, AEP/profile, organization, mission, security/accountability, operational-continuity rule, or explicit Glaux default |
| Approval and evidence | Approver/issuing authority, approval status, reference, rationale, signature/digest if applicable |
| Scope | Tenant, mission, source authority, record class, resource type, sensitivity, purpose, jurisdiction/profile, and deployment |
| Effective interval | Transaction/effective dates and review/withdrawal state |
| Trigger and clocks | Named trigger event, clock field, precision/skew treatment, and missing/disputed-time behavior |
| Phases and actions | Online/archive/compaction eligibility, minimum and maximum if supplied, disposition action, tombstone/backups/peer treatment |
| Holds and exceptions | Eligible hold types, issuer, scope, start/release, priority, conflict route, and emergency process |
| Evidence/visibility | Receipt and proof fields, externally visible safe projection, access controls, and audit requirements |

Resolution rules:

1. Determine every applicable rule using authoritative scope and effective time.
2. An active valid hold blocks disposition within its exact scope.
3. A destructive action requires affirmative authority. Absence, ambiguity, clock dispute, policy conflict, or unknown ownership creates an exception and retains/restricts the record.
4. Where compatible rules differ, meet every valid minimum and do not exceed a valid maximum. If those cannot both be met, hold and route to the named policy authority; software must not invent precedence.
5. User-requested deletion is a request evaluated under policy and authorization, not an automatic override.
6. Source/tenant/mission authority cannot cascade into records owned by another authority. The operation must reject, detach an allowed association, create an unresolved/policy-hidden reference, or obtain the other authority's decision.
7. Glaux defaults may supply safe operational behavior only when explicitly approved and labeled. This report selects no numerical defaults.
8. Policy detail exposed to a client is a least-information projection. Internal rationale, sensitivity, hold identity, and copy topology can be restricted.

---

## 5. Complete Data and Artifact Record-Class Inventory

| Record class | Examples | Owner/policy authority | Storage/role | Mutability and reconstructability | Lifecycle dependency |
|---|---|---|---|---|---|
| Canonical resources/current selector | Systems, Deployments, Features of Interest, Sampling Features, DataStreams, ControlStreams | Resource/source authority plus tenant/profile | PostgreSQL authority/current projection | Current selector mutable; revisions are not; reconstructable only from retained revisions | Identity, relationships, source, policy, provenance |
| Canonical resource revisions | Every accepted version of a resource | Same resource authority; accountability overlays | PostgreSQL authoritative history | Append-only; irrecoverable if source and evidence also purged | Resource ID, revision chain, validation, provenance |
| Typed relationships and collections | composition, deployment, association, derivation, membership | Edge owner plus endpoint authorities | PostgreSQL authoritative graph/history | Bitemporal facts; derived navigation rebuildable | Both endpoints, edge type, authority |
| Observation records | Values, result, phenomenon/result/ingest times, quality | Data/source/mission authority | PostgreSQL typed temporal authority; archive tier optional | Append/correct by new version; raw value may be irrecoverable | DataStream contract/package, provenance |
| Status and temporal-property facts | system status, availability, validity | Source/mission authority | Typed temporal authority | Append-oriented; current view rebuildable | Subject, time model, provenance |
| System/domain events | state/event occurrences | Event/source authority | Append authority | Append-only; summaries derived | Subject, event type/package, provenance |
| Data/ControlStream definitions and contracts | SWE schemas, observed properties, units, encodings | Resource/profile authority | PostgreSQL catalog plus immutable artifact | Immutable binding once children exist | Child interpretation and validation |
| Commands | intent, parameters, schedule, targets | Command issuer/target authority plus security/safety policy | PostgreSQL command authority | Append/state-machine evidence; external effect not reconstructable | ControlStream contract, authorization, statuses |
| Command status/results | accepted, executing, failed, canceled, completed, unknown, outputs | Target/source authority plus accountability | Append authority/artifact | Append-only; outcome evidence often irrecoverable | Command ID, target, dispatch/audit/provenance |
| Feasibility requests/status/results | analysis intent and outcomes | Requester/provider authority | PostgreSQL plus artifacts | Append-oriented; recomputation not guaranteed | Definition/contract/package/provenance |
| Exact source documents | SensorML, SWE, imported schemas/OpenAPI | Source/document authority | Immutable CAS plus catalog | Immutable bytes; often irrecoverable | Derived canonical records, signatures, provenance |
| Parsed/normalized documents | parsed trees, canonical projections | Glaux transformation authority | PostgreSQL/JSONB or artifact | Rebuildable only if source/tool/package retained | Exact source and transformation manifest |
| Generated representations/exports | JSON/XML views, packages, reports | Resource plus release/export authority | Ephemeral, CAS, or export store | Rebuildable when inputs retained; external copy not controlled | Resource revision, generator, policy projection |
| Schema/profile/vocabulary packages | closed validation/semantic bundles | Package/profile authority | Immutable CAS/catalog | Immutable and required for reproducibility | Validation/interpretation/release |
| Validation evidence | reports, diagnostics, conformance result | Validation/policy authority | PostgreSQL evidence plus artifact | Append-only results; full report may be reducible | Input digest, package/tool/options |
| Provenance/quality/trust evidence | activities, agents, derivations, assertions | Evidence/source/security authority | PostgreSQL authoritative graph | Append-oriented; may outlive payload in minimized form | Subjects, inputs, policy decisions |
| Raw/admission payloads | request bytes, ingest frames, import bundles | Source/tenant/security policy | Staging/quarantine/CAS | Immutable; may be sensitive or malicious; replay value varies | Admission, parser, normalized result |
| Rejected/quarantined material | invalid, suspicious, hostile, undecidable input | Security/data owner | Isolated encrypted quarantine | Never production authority; retain only under bounded explicit rule | Validation/security case, access log |
| Identity/admission/idempotency ledger | keys, fingerprints, decisions, result refs | Service/operation policy | PostgreSQL authority | Append/state-machine; effect recognition depends on it | Retry, command, replay horizon |
| Inbox/source-offset/cursor state | message IDs, source positions, consumer state | Ingest/sync/stream authority | PostgreSQL authority | Mutable cursor plus immutable receipt facts | Replay/catch-up horizon and source semantics |
| Outbox/publication state | domain event, attempts, ack/delivery state | Transaction/publishing authority | PostgreSQL authoritative queue/history | Append plus state; message copy external after publish | Event, subscriber/broker, retry |
| Conflict/synchronization state | branches, vector/causal facts, tombstone receipts | Participating authorities | PostgreSQL/DDIL sync authority | Append/resolve explicitly; cannot use arrival-order overwrite | Peers, resources, policy, tombstones |
| Lifecycle policy, evaluations, holds, jobs, receipts | schedules, exceptions, archive/delete work | Policy authority/administration | PostgreSQL lifecycle authority | Versioned/append-only evidence | Every governed class and copy |
| Audit/accountability records | successful/denied admin and security actions | Security/accountability authority | Protected append store | Append-oriented; payload minimized; not merely app log | Actor, action, outcome, correlated evidence |
| Search indexes/materialized projections | spatial/text/latest/statistical views | Derived-state operator | PostgreSQL indexes/materialized tables/external adapter | Disposable and rebuildable | Authorized surviving authority state |
| Application/edge caches | representations, lookups, negative caches | Runtime operator | Memory/local/HTTP cache | Disposable | ETag/policy/resource state |
| Broker/replay buffers | delivered events/messages/consumer acknowledgements | Broker/stream policy and subscriber contract | External or embedded broker | Derived delivery copy; recall may be impossible | Outbox/event and transport policy |
| Replicas/federated/DDIL copies | standby, read replica, remote node | Deployment and peer authorities | Database/storage at other node | Propagated authority/copy; availability varies | Replication/sync protocol and tombstone |
| Backups, WAL, snapshots, archive media | base/incremental backup, logical dump, PITR logs | Continuity/backup/security authority | Recovery storage | Point-in-time recovery copy; row-level deletion usually unavailable | Backup chain, restore objectives, expiry |
| Operational logs/metrics/traces | errors, latency, job counters, correlation IDs | Operations/security/privacy policy | Observability systems | Append/aggregate; avoid payload by design | Incidents, audit correlation, platform retention |
| Test/fixture/simulation records | golden files, generated load, simulated commands | Test/release authority | Dedicated namespace/artifact store | Versioned; never mixed with operational authority | Requirement, generator/seed, release |
| Temporary/staging/orphan work | uploads, partial archives, failed object writes | Worker/job authority | Ephemeral database/object staging | Non-authoritative; recover/reconcile/delete by lease | Job ID, staging lease, integrity state |
| External references/copies | source URLs, downloaded exports, third-party replicas | External owner; Glaux has limited authority | Catalog pointer/external system | May be unverifiable or uncontrollable | Disclosure, revocation attempt, limitation |

The inventory is intentionally broader than API resources. Every class either contains authority/evidence or is a material copy that can expose, resurrect, or falsely suggest deletion of governed information.

---

## 6. Lifecycle Terminology, State Model, Transitions, Actors, and Authority

### 6.1 Precise state meanings

| State | Dimension | Meaning | Reversible? |
|---|---|---|---|
| Active/current | Semantic | Selected authoritative version is effective for its declared context | Yes, through new revision/state |
| Historical/superseded | Semantic | Prior valid version replaced by a later version; not false or deleted | No rewrite; can become visible by historical query |
| Expired | Semantic/policy | Declared validity or explicit rule ended; not synonymous with deletion eligibility | Usually no; correction may add new fact |
| Stale | Semantic assessment | Freshness expectation exceeded; value still exists with known age | Yes when new evidence arrives |
| Inactive/retired | Semantic/administrative | No longer participating or intended for new operations | Reversible only by authorized new transition |
| Quarantined/restricted | Custody | Isolated or access-limited; not authoritative/public | Yes after validation/authorization |
| Held | Custody overlay | Disposition suspended for a named scope and authority | Yes only through authorized release |
| Archived | Storage | Authoritative/evidentiary content moved to a validated managed archive | Yes through controlled restore |
| Deletion-pending | Disposition workflow | API or administrative request accepted; impact/copy work remains | Cancelable only before irreversible step and if policy permits |
| Tombstoned | Disposition/API | Current payload unavailable; minimum marker prevents reuse/resurrection and explains state to authorized actors | Restoration requires explicit authorization/new activity |
| Purged | Disposition | Target payload removed from designated active/archival stores and derived surfaces in the recorded copy scope | Generally terminal for those copies |
| Backup-expired | Recovery | Recovery copy no longer belongs to any supported restore chain and has been disposed under backup policy | Terminal for that recovery copy |
| Sanitized | Media | Approved media method completed and verified under the deployment program | Terminal for target data/media under stated assurance |

“Deleted” is prohibited as an unqualified persistent state because it obscures scope. Interfaces and evidence must say API-unmapped, tombstoned, purged from named stores, recovery-copy-expired, or media-sanitized.

### 6.2 Transition contract

Every transition records transition/event ID, subject and revision, from/to dimension states, requesting actor, deciding authority, executor, reason category, policy ID/version, decision time and relevant domain/transaction clocks, hold evaluation, dependency/copy manifest, idempotency/precondition, result, failures/retries, audit/provenance correlation, and next action. Sensitive detail is stored in a protected evidence record and exposed only through a safe projection.

| Transition | Requester | Authorizer | Executor | Key gates |
|---|---|---|---|---|
| Supersede/retire | Resource owner/API client/source | Resource authority | Domain transaction | ETag, schema/relationship rules, provenance |
| Apply/release hold | Authorized policy/security/legal/mission role | Configured hold authority | Lifecycle transaction | Exact scope, authority, reason, review/release evidence |
| Archive | Scheduler/operator/policy engine | Applicable policy | Archive worker | No conflicting hold/action; manifest and integrity complete |
| Restore | Authorized operator/continuity workflow | Data and security authority | Restore orchestrator | Manifest, compatibility, current policy, tombstone ledger |
| API logical delete/tombstone | Authorized client/operator | Resource and policy authority | Domain transaction + job | CSAPI cascade, ownership, hold, impact preview, If-Match |
| Purge | Policy engine/operator | Destruction authority, risk-based approval | Disposition worker | Eligibility stable, dependencies/copies enumerated, no hold |
| Backup expiry | Backup scheduler/operator | Continuity policy | Backup system | Restore-chain closure and supported recovery window |
| Media sanitize | Deployment/decommission process | Security/system owner | Approved sanitization operator/tool | Media inventory, sensitivity, approved technique, verification |

---

## 7. Retention Triggers, Holds, Overrides, and Policy-Change Behavior

### 7.1 Trigger semantics

Retention begins only from a named, recorded event. “Age” without a clock and trigger is invalid.

| Class/theme | Candidate trigger that policy must select | Required gate |
|---|---|---|
| Resource revision | creation, supersession, retirement, or source withdrawal | preserve current/required lineage and relationship closure |
| Observation/status/event | phenomenon, result, receipt, ingest, publication, stream retirement, or mission closure | corrected/future/disputed timestamps handled explicitly; partition key is not policy |
| Command/feasibility | submission, final reconciled outcome, target acknowledgement, mission closure | unresolved/unknown/ongoing outcome is not automatically eligible |
| Source/raw/validation | receipt, successful normalization, replacement, package retirement, case closure | replay, signature, diagnosis, provenance, incident need resolved |
| Idempotency/inbox/outbox | committed effect, source acknowledgement, delivery completion, retry/replay horizon end | must outlive every authorized replay and offline reconnect horizon |
| Tombstone/conflict | deletion decision, last required peer acknowledgement, conflict resolution | anti-resurrection and identifier non-reuse remain satisfied |
| Archive/backup/WAL | archive publication, backup completion, supersession by validated chain, recovery-window close | restore-chain closure verified before removal |
| Audit/security | event, case/mission closure, authoritative schedule event | IDR-SRV-041 and selected control/policy input |
| Quarantine/staging | receipt, decision, job completion, lease expiry | explicit bounded rule, incident hold, no authority promotion |

The time model inherits IDR-SRV-018. Timestamps retain source, precision, zone/offset, uncertainty, and correction lineage. Missing, invalid, future, or disputed trigger time routes to an exception queue; the server must not substitute ingest time unless the selected policy expressly permits it. Deadline computations store the policy revision and calculated deadline, so reevaluation is explainable.

### 7.2 Holds and exceptions

A hold is a separately versioned overlay with hold ID, authority, scope selector, protected purpose/reason class, start, review/release rules, and evidence link. It suspends archive/purge steps only within its scope. It does not:

- make stale or retired data current;
- grant broader read access;
- require unrestricted raw payload retention when an authorized minimized form suffices;
- silently copy held data into unrelated systems; or
- survive release without an immutable release decision.

Workers recheck holds and policy under lock immediately before every irreversible step. Applying a hold concurrently with disposition either wins before the terminal boundary or creates a high-severity exception with exact evidence of what had already completed.

### 7.3 Policy changes and precedence

- Publish policy as a new immutable version with effective time; never rewrite prior evaluations.
- Recalculate affected records in a dry-run/impact phase and store old/new rule, deadline, and action.
- A shorter/newly destructive rule does not silently purge existing material. It requires configured review/approval and must respect holds and conflict resolution.
- A longer rule cancels pending work that has not crossed an irreversible boundary; it cannot reconstruct purged data.
- Withdrawal of a rule makes dependent automatic destruction ineligible until another affirmative authority resolves it.
- Emergency actions require a named emergency authority, bounded scope, recorded rationale, and after-action review; they do not erase evidence of the decision.
- Most-restrictive compatible requirements apply. Incompatible minimum-preservation and maximum-retention requirements create a hold/exception for the competent authorities.

---

## 8. Per-Class Lifecycle and Disposition Decision Matrix

All “period” entries below mean an approved policy-supplied duration or event horizon. No numeric duration is implied.

| Record class | Normal lifecycle | Archive/transform permission | Eligibility trigger/gates | Disposition outcome | Required residual/propagation |
|---|---|---|---|---|---|
| Canonical current resource | current → superseded/retired → archived or tombstoned | archive exact revision/graph closure | authorized delete/retire; dependency and CSAPI cascade evaluation | remove current mapping; purge payload only by policy | non-reused ID and policy-safe tombstone if required |
| Resource revisions | current → historical → archived | immutable archive; redact only as new projection | supersession/retirement plus lineage/audit policy | dependency-aware purge | surviving revision chain/placeholders must remain truthful |
| Relationships/collections | active/history → detached/tombstoned | archive with endpoint/version context | owning edge authority and endpoint rules | detach, retain historical fact, or purge by policy | no cross-authority cascade; no dangling false relation |
| Data/ControlStream contract | active → retired → archived | immutable package archive | all child interpretation/safety dependencies closed | purge only with dependents/evidence resolved | child records retain exact contract fingerprint/package |
| Observations | append/correct → online/archive/aggregate → eligible | lossless compression; authorized aggregation/downsampling | selected clock plus correction/late-data/hold gates | raw/full record purge or archive | aggregates and latest view truthfully disclose source availability |
| Status/temporal facts | append → stale/historical → archive | categorical/event-aware compaction only | explicit policy and reconstruction needs | purge/retain summary | never average categorical states or hide gaps |
| System events | append → archive | count/index summaries may derive | event/mission trigger and accountability | purge under explicit event policy | audit/provenance dependencies resolved |
| Commands/status/results | submitted → terminal/reconciled → restricted/archive | redact/minimize by named safe projection | final outcome known, target reconciliation complete, no investigation/hold | payload purge may precede minimum evidence | retain permitted accountability/safety trail; cancellation remains status |
| Feasibility records | requested → final/expired → archive | safe summary/redaction | final result plus policy | purge/archive | command/tasking correlation if relied upon |
| Exact source documents | immutable → superseded/archive | byte-preserving archive | no remaining reproducibility/signature/interoperability dependency | mark-and-sweep purge | policy-safe provenance placeholder if canonical record survives |
| Parsed/normalized/generated | generated → stale → rebuilt/evicted | freely rebuild when inputs remain authorized | source/revision/package/policy change | evict/purge | cannot rebuild from purged/unauthorized sources |
| Schema/profile/vocabulary packages | staged → active → retired/archive | immutable reference-closed archive | no retained interpretation/validation/release dependency | dependency-rooted purge | logical URI/digest history preserved where required |
| Validation/provenance/quality | append → archive/minimize | safe projection; full evidence controlled | subject/input/tool/package and accountability dependency closed | purge permitted content | unavailable/minimized lineage remains explicit |
| Raw admitted input | staged → accepted/rejected → archive or eligible | normalize; sample/redact only by policy | authoritative result and replay/diagnostic period complete | purge content | retain minimal admission/provenance decision if authorized |
| Rejected/quarantine | isolated → decided/held → eligible | security-approved sample/redaction | case closure or bounded quarantine rule | prompt purge when no affirmative need | never expose to normal search/export/backup unintentionally |
| Idempotency/admission | pending → committed/failed → expired | compact response metadata | all retry, replay, command, and offline horizons complete | purge key/fingerprint | no duplicate real-world effect risk |
| Inbox/offset/cursor | received → applied/acked → expired | compact contiguous acknowledgements | source replay/catch-up and peer acknowledgement complete | purge old receipts/cursors | dedupe safety retained across reconnect |
| Domain event/outbox/broker | pending → published/acked → expired | broker compaction only under key/event semantics | delivery/replay/subscription policy complete | local purge and broker expiry | downstream copy limitation recorded |
| Conflict/sync/tombstone | open → resolved/acknowledged → eligible | archive/minimize | every required peer/horizon and authority rule complete | purge only when resurrection impossible under profile | unavailable peer becomes explicit exception |
| Lifecycle policy/hold/job/receipt | proposed → active/withdrawn; job pending → terminal | archive immutable evidence | superseded plus accountability schedule | purge only by policy | enough evidence to explain every disposition |
| Audit records | append → restricted/archive | minimize/redact under audit policy | authoritative audit schedule/hold | purge/archive | detailed rules owned by IDR-SRV-041 |
| Index/cache/materialization | current → invalidated/stale → rebuilt | always derived | authority/policy change or deletion event | evict/drop/rebuild | no query/replay path may expose removed content |
| Replica/federated/DDIL copy | replicated → tombstoned/acknowledged | profile-specific local archive | acknowledged deletion plus offline horizon | purge by node policy | causal tombstone prevents stale replay |
| Export | prepared → delivered/revoked/expired | encrypted/package manifest | export policy and access expiry | revoke managed copy/purge staging | disclose inability to delete recipient-controlled copies |
| Backup/WAL/snapshot | created → validated/current → superseded/expired | immutable recovery chain | policy window closed and chain no longer needed | expire/delete media/object copy | restore procedure reapplies deletion ledger |
| Logs/metrics/traces | emitted → aggregated → expired | aggregate/redact | ops/security policy | purge raw telemetry | avoid payload/secret capture from outset |
| Fixture/test/simulation | authored → released/superseded | versioned archive | release/support policy | purge/archive | namespace and synthetic/controlled provenance retained |
| Staging/orphan | leased → finalized/reconciled/expired | none until verified | job terminal plus lease/reconciliation | delete orphan | never considered authority or valid archive |

---

## 9. Historical-Version, Provenance, Tombstone, and Referential Rules

1. **History is immutable evidence, not hidden current state.** Updating a resource creates a revision/activity and changes the current selector. Correction does not rewrite a prior accepted fact without a retained correction relation.
2. **Invalidation is explicit.** A provenance activity may invalidate an entity or revision, but that statement does not claim bytes, replicas, exports, or media were destroyed.
3. **Identifiers are never reused.** A purged resource cannot be recreated under the same identity as if no prior resource existed. A new authorized restoration or recreation is a new activity/revision and must not defeat the tombstone ledger.
4. **Tombstones are minimum anti-resurrection/evidence records.** Candidate fields are opaque ResourceId, resource type, deletion epoch/causal version, origin node, tombstone event ID, policy/action reference, and a protected reason/authority reference. Public exposure is policy-dependent. Do not retain original titles, geometry, payload hashes, actor identities, or sensitive reason text by default.
5. **404 and 410 do not define tombstone retention.** Unauthorized users may receive 404 to avoid existence disclosure. Authorized policy may use 410 for known permanent unavailability. Neither status proves physical purge.
6. **Relationship disposition follows edge ownership.** Composition-owned children participate in the CSAPI cascade impact graph; associations, deployments, provenance/source links, and external/federated edges are detached, restricted, rejected, or made explicitly unresolved according to their owners. No operation silently destroys a resource owned by another authority.
7. **Source removal does not falsify canonical facts.** A canonical record may survive only when its authority and retained evidence remain sufficient under policy. The provenance graph then points to a protected “source removed under policy” placeholder. If the source is essential evidence, the canonical dependent is held, archived with it, or also removed.
8. **Dependency-aware collection is catalog-rooted mark-and-sweep.** Roots include active/current resources, retained history, holds, policies, packages/releases, command/audit evidence, archives, fixtures, pending jobs, and DDIL manifests. Only unreachable, grace-complete, hold-free objects are eligible. Reference counts alone are inadequate.
9. **Cascade is planned and recoverable.** A dry-run computes the complete owned graph, foreign-authority blockers, retained evidence, copy obligations, and estimated work. The logical authority change and deletion job/outbox commit atomically; workers are idempotent. Partial physical failure leaves a visible failed/pending disposition state, not a false success.

---

## 10. Dynamic-Data Compaction, Aggregation, and Downsampling Rules

### 10.1 Transformation classes

| Transformation | Information effect | Required treatment |
|---|---|---|
| Encoding change/lossless compression | Intended semantic information preserved | Verify round-trip/digest as applicable; still the same retention class |
| Physical partition detach/tier move | Placement changes, not policy | Gate by eligible set and manifest; never infer eligibility from partition age |
| Duplicate suppression | Removes repeated representation of same admitted identity | Prove identity/fingerprint and retain admission result through replay horizon |
| Numeric aggregation/downsampling | Loses detail and creates derived information | New derived record with complete method/provenance and explicit substitution authority |
| Categorical/event compaction | May erase transitions/order | Only event-aware encodings preserving required transitions, duration/gaps, and nil semantics |
| Current/latest materialization | Derived selection | Rebuildable; preserve tie/correction rule and disclose when source history is unavailable |

Every aggregate records: new identity; input stream/contract fingerprints; phenomenon/result interval; watermark and late/correction cutoff; input count, expected count and gaps; method, software and configuration version; units/observed-property/semantic package; quality, uncertainty and nil handling; source coverage; policy ID/version; provenance activity; verification result; and whether it is authorized to substitute for raw inputs.

Raw deletion after aggregation requires all of the following:

- an applicable rule explicitly permits substitution;
- the data type/semantics support the method;
- the aggregate and manifest validate;
- late/corrected input windows and active holds are closed;
- dependent command, incident, provenance, audit, replay, or synchronization needs are resolved;
- current/latest projections have a defined post-purge truth state; and
- deletion is propagated to raw copies and recorded.

Never numerically average command parameters/outcomes, statuses, enumerations, identifiers, nil reasons, discrete events, categorical values, or qualitative observations. A latest-value view whose source has expired remains a labeled derived snapshot with source-unavailable/expired provenance and evidence bounds; it must not imply the underlying history is still queryable.

Capacity, rate, and query SLA inform partitioning/tiering and the proposal of policy changes, but do not authorize deletion. Emergency capacity actions require a recorded emergency policy/authority, bounded impact, and after-action review.

---

## 11. Command, Feasibility, Audit, Raw, Rejected, and Quarantine-Data Treatment

### 11.1 Commands and feasibility

Command intent, exact contract/profile binding, authorization and safety decision, dispatch attempts, target acknowledgements, immutable status history, cancellations, results, unknown outcomes, provenance, and audit correlation form one dependency set. “Terminal” protocol status alone is insufficient if target reconciliation, investigation, hold, or downstream safety use remains open.

Sensitive command parameters/results may be moved to restricted storage or purged earlier than the minimum accountability record when an approved policy permits. The minimum record should identify the command and target through protected/opaque identifiers, relevant times, outcome class including uncertainty, policy/authorization decision reference, dispatch/reconciliation evidence reference, and disposition activity—only to the degree authorized. Simulated, training, conformance, and operational commands require separate namespaces and selectable policies; a test label must not be inferred from payload text.

### 11.2 Audit and security evidence

Lifecycle actions are themselves auditable. Hold application/release, impact preview, approval, archive publication, restore, delete, purge, backup expiry, failed propagation, exception resolution, and media sanitization require immutable correlated evidence. IDR-SRV-041 owns the final audit schema and selected retention schedule. This report requires minimization: do not copy governed payload, secrets, full queries, credentials, or sensitive policy reasons into logs simply to prove an action occurred.

### 11.3 Raw, rejected, and quarantine material

- Preserve exact admitted bytes before parsing only when the admission/profile policy permits, as required by IDR-SRV-028.
- Raw data is not automatically permanent. Reproducibility, signature, replay, incident, interoperability, source-contract, and provenance needs must be weighed against sensitivity and exposure.
- Rejected/suspicious content stays outside production authority and ordinary search, export, cache, analytics, replication, and backup paths unless the quarantine policy explicitly includes a protected copy.
- A deployment must provide a bounded quarantine decision rule; no numeric value is invented here. Without affirmative retention authority, minimize and dispose of payload after retaining only a safe decision/audit record.
- Repair/reprocessing creates a new artifact and activity linked to the original where permitted; it never overwrites malicious or invalid input.
- Diagnostics are structured, bounded, redacted, and access-controlled. A digest, byte excerpt, URI, filename, parser stack, or correlation key can itself leak sensitive information and is retained only when justified.

---

## 12. Archive Manifest, Integrity, Restore, and Failure Contract

### 12.1 Authoritative archive publication

An archive is not a loose export. Its manifest includes:

- archive, job, tenant/mission/profile, policy and format identities;
- creation tool/version/configuration, creator/executor and approval evidence;
- resource/revision/artifact IDs, record classes, counts, sizes and algorithm-qualified digests;
- exact time bounds and clocks, transaction/log high-water marks, and consistency point;
- schema, SensorML/SWE contract, profile, vocabulary and migration-package identities/digests;
- relationship/dependency closure and declared external/missing dependencies;
- provenance, validation, quality and tombstone/deletion-ledger bounds;
- sensitivity, releasability, source ownership and access-control projection;
- encryption/key reference—not secret key material—and signature/integrity evidence;
- source database/deployment version and required restore/runtime compatibility;
- archive state, verification results, failures, retries, storage locations and replica inventory.

Publication sequence:

1. Resolve a consistent eligible set and freeze its manifest inputs.
2. Write encrypted/controlled objects to a non-authoritative staging namespace.
3. Verify bytes/digests, counts, schemas/packages, identifiers, time bounds, referential closure, and authorization.
4. Atomically publish the catalog record and archive manifest; emit lifecycle/provenance/audit/outbox evidence.
5. Only then transition source records to archived or eligible-for-tier-removal.
6. Reconcile/delete failed or orphaned staging objects by lease. A partial archive is never discoverable as authoritative.

Periodic integrity scrubs and restore drills are mandatory. Required frequency, durability class, RPO, RTO, geographic/media topology, and technology belong to deployment/profile policy and IDR-SRV-049.

### 12.2 Restore contract

Restore is a new controlled activity:

1. Authenticate and authorize the request; select exact manifest and compatible policy/package/migration set.
2. Restore into an isolated, non-serving namespace.
3. Verify signatures/digests/counts, archive closure, encryption access, schema/package availability, and source identity.
4. Apply reproducible migrations while retaining original identifiers and migration provenance.
5. Compare archived policy with current policy, holds, ownership, sensitivity, and conflicts.
6. Apply the current tombstone/deletion ledger and every later disposition event before publication. Never expose restored pre-deletion state.
7. Resolve collisions and DDIL branches explicitly; do not use arrival-order last-write-wins.
8. Rebuild indexes/caches/current projections only from surviving authorized authority state.
9. Validate representative queries, relationships, temporal semantics, provenance, policy filtering and audit continuity.
10. Atomically publish restored authority or a restricted review state; record outcome and clean staging.

Unavailable/corrupt media, digest mismatch, missing package, broken dependency, migration failure, policy conflict, deletion-ledger gap, identifier collision, or partial copy is a terminal failed/review state until explicitly remediated. The system must not serve a “best effort” reconstruction as authoritative.

For tactical/DDIL restoration, manifests and necessary packages travel as reference-closed signed/verified bundles. Reconnect exchanges causal inventory and tombstone ranges before any restored content is promoted.

---

## 13. Delete, Purge, Cryptographic Erase, Backup Expiry, and Sanitization Distinctions

| Term/action | Scope and guarantee | Evidence | What it does not prove |
|---|---|---|---|
| HTTP/API DELETE | Removes target association/current API functionality according to CSAPI and Glaux contract | Request, precondition, policy/cascade decision, response/job | Physical destruction, backup removal, broker recall |
| Logical deletion | Excludes current record from normal authorized selection | Revision/lifecycle event/current-selector change | Old row/page/object versions erased |
| Tombstone | Minimal marker for identity, causality, policy-safe unavailability and anti-resurrection | Tombstone ID/version/origin/acks | Payload retained or purged everywhere |
| Archive | Moves governed content into validated managed long-term storage | Manifest, integrity and catalog publication | Content deleted or less sensitive |
| Purge | Removes target payload from named active/archive/derived stores in a recorded scope | Copy inventory, attempts, verification receipt | Expired from unavailable backups/exports or sanitized media |
| Backup expiry | Removes a backup/WAL/snapshot from supported recovery chains under policy | Chain analysis, object/media disposition, catalog update | Individual record was selectively erased before expiry |
| Cryptographic erase | Purge technique based on rendering required cryptographic keys inaccessible under an approved design | Key scope, copy/key inventory, operation and verification | Per-record erasure when keys are shared or copies/escrow survive |
| Media sanitization | Clear/purge/destroy operation under approved media program and threat/sensitivity assessment | Media ID, method/tool/version, operator, verification, disposition | Routine SQL/API deletion or universal absolute impossibility of recovery |

### 13.1 Database and object-store consequences

PostgreSQL MVCC retains deleted row versions until they are no longer visible and vacuumed; ordinary VACUUM reuses space rather than proving media sanitization. Partition drop/detach can make bulk lifecycle work efficient but only after every row in the partition shares an eligible policy state. Mixed holds, tenants, sensitivities, trigger clocks, or correction windows prohibit whole-partition disposition.

In a versioned S3-compatible store, a simple delete may create a delete marker while previous versions remain. Adapters must inventory versioning, retention/object-lock behavior, replicas, lifecycle rules, and version-specific deletion. Backend parity tests must validate the actual provider; AWS documentation is not a portability guarantee.

PITR deliberately preserves historical changes. Backups and WAL therefore age through a separately approved continuity policy. A restored older point must receive the later deletion/tombstone ledger before it can serve data. Immutable/offline backups may not support row-level removal; Glaux must disclose that deletion awaits recovery-copy expiry and restrict restoration rather than claim immediate erasure.

### 13.2 Cryptographic erase eligibility

Cryptographic erase is rejected for a target unless all are proven:

- encryption and key lifecycle are approved for the relevant media/threat model;
- the key scope contains the target data and no data that must survive;
- every replica, object version, archive, backup, wrapped/derived key, escrow/recovery key and hardware copy is included;
- keys were never exported beyond the controllable scope, or external disposition is evidenced;
- metadata, plaintext caches, indexes, logs and temporary copies are independently addressed; and
- the operation and verification are auditable without retaining prohibited secret material.

Per-tenant or per-archive encryption may enable coarse-grained erasure. A database-wide, bucket-wide, or shared backup key normally cannot prove per-record erasure.

### 13.3 Proof of disposition

A defensible receipt identifies the request/action, authority, policy version, target/impact manifest, named copy classes and stores, completed and pending steps, verification method, times, executor/tool versions, failures/exceptions, backup/peer/export limitations, and audit/provenance references. It claims only what was observed in the controlled scope. It never asserts deletion from recipient-controlled exports, unreachable peers, unknown unmanaged copies, or physical media without evidence.

---

## 14. Index, Cache, Broker, Replica, Export, Backup, and Synchronization Propagation Matrix

| Copy/surface | Lifecycle action | Completion evidence | Failure/unavailable behavior | Residual limitation |
|---|---|---|---|---|
| PostgreSQL authority/current selector | Atomic revision/tombstone and deletion job/outbox | committed lifecycle event and query exclusion | transaction rollback; retry idempotently | dead row versions await VACUUM; storage is not sanitized |
| PostgreSQL history/temporal partitions | row/partition purge only for uniformly eligible, hold-free set | eligible-set snapshot, row/count/partition verification | keep detached/restricted; alert | replicas/backups still separate |
| Immutable object/CAS | mark dependency-unreachable then delete all governed versions | catalog mark/sweep manifest and backend version receipts | retain restricted retry state | provider behavior/version lock must be checked |
| Spatial/text/database indexes | transactional removal or rebuild after authority change | negative authorized query and index consistency check | mark stale/unavailable, never serve stale index | index pages still follow storage sanitization |
| Materialized/latest projections | invalidate/update with authority transaction or outbox | source revision/tombstone watermark | suppress projection until repaired | derived snapshot policy must remain truthful |
| In-memory/edge/HTTP cache | purge key variants and policy scopes; propagate invalidation | cache-generation/ack and negative read | shorten/suppress access; retry | third-party client caches not controllable |
| Outbox | publish deletion/tombstone event; retain through delivery/replay policy | ack/attempt ledger per destination | pending/failed visible; retry | outbox history has its own policy |
| Broker/replay stream | publish tombstone/invalidation; expire or compact under broker policy | broker ack, consumer offsets/receipts | record unacknowledged consumers; restrict replay | already delivered copies cannot be recalled |
| Subscriber/client | document deletion event and cache invalidation contract | acknowledgement if protocol supports it | disclose no control after delivery | external retention governed by recipient |
| Read/standby replica | database replication plus application deletion event | replay LSN/transaction or application ack | remove from routing if stale past policy threshold | snapshots/backups remain separate |
| Federation/DDIL peer | causal tombstone, policy-safe reason and required dependencies | per-peer receipt and advertised causal horizon | exception/hold tombstone; retry on reconnect | unavailable/foreign peer may remain outside control |
| Export/download staging | revoke URLs/credentials and purge managed staging | object/version receipt and access test | disable access; retry purge | downloaded recipient copy cannot be proven deleted |
| Archive tier | update manifest/catalog or create authorized redacted successor; purge named archive if allowed | archive-specific integrity and deletion receipt | quarantine archive; no partial success claim | offline media may await handling |
| Base/incremental backup | expire only when no supported chain depends on it | chain graph and backup catalog deletion | retain until safe; alert capacity risk | historical rows can survive until expiry |
| WAL/PITR archive | expire under recovery-window/chain policy | WAL range/catalog verification | retain required continuity range | permits reconstruction of prior deleted state while retained |
| Logs/metrics/traces | redact/minimize identifiers, purge by observability policy | scoped search/retention verification | restrict store and open exception | downstream SIEM copies may be external |
| Search/analytics export | delete/rebuild derived collection from authorized sources | negative query plus source watermark | stop serving stale dataset | independent analyst copies remain external |
| Local/tactical package | include tombstone/deletion ledger in next sync/package | package manifest and import receipt | package marked stale; deny unsafe promotion | disconnected copy persists until reconnect/action |

### 14.1 DDIL and resurrection prevention

Each node applies a deletion as a locally committed causal event and retains a tombstone long enough for the approved offline/replay horizon or required peer acknowledgements. Exact duration is profile-supplied. Reconnect exchanges node/branch identity, causal bounds, policy versions, tombstone ranges and acknowledgements before resource payloads. A stale update concurrent with or preceding a deletion does not silently recreate current state; it is rejected, retained as a conflict, or admitted only by an authorized resolution that creates a new activity/revision. If a peer remains unavailable beyond a desired deadline, the job records an exception and the API/receipt discloses the limitation to authorized actors.

---

## 15. API and Administrative Behavior, Guarantees, Errors, and Authorization Needs

### 15.1 API-visible contract

| Situation | Recommended behavior | Guarantee |
|---|---|---|
| Delete rejected by CSAPI child rule | 409 Problem Details with safe dependency category and cascade option if supported | No lifecycle change |
| Hold, policy conflict, foreign authority, unresolved command/sync safety | 409 Problem Details; conceal restricted details | No destructive action accepted |
| Stale/missing precondition | 412 or 428 under IDR-SRV-029 | No action against unexpected revision |
| Accepted asynchronous cascade/purge | 202 with Location/status link and operation ID | Request durably accepted; not all copies removed |
| Synchronous logical/API deletion enacted | 204 when no response body; 200 if a status representation is supplied | Current mapping/visibility changed as documented |
| Later read | 404 for absent/concealed; policy-authorized 410 for known permanent unavailability | No physical-erasure implication |
| Archived but restorable | normal response only if transparent retrieval meets policy/SLA; otherwise explicit archive/restore status or 202 restore job | State and latency are not hidden |
| Already completed identical request | idempotent original/terminal status | No duplicate cascade/purge |
| Same idempotency key, different target/intent | 409 | Ambiguous destructive retry refused |
| Non-restorable/corrupt archive | 409/500-class Problem Details according to request versus server fault | Never serves unverified reconstruction |

Problem Details should include stable type, title, status, safe detail, instance/correlation, lifecycle operation ID, current state, retryability/next action, and safe dependency counts/categories. It must not disclose hidden resource existence, hold identity, policy text, sensitivity, topology, or other tenants.

### 15.2 Administrative controls

- Separate permissions for view policy, preview impact, request archive/delete, apply/release hold, approve purge, restore, inspect evidence, override exception, and sanitize media.
- Require strong ETag/If-Match and scoped idempotency for policy/hold/disposition mutations.
- Provide dry-run graph/copy impact, affected class counts, foreign-authority blockers, hold status, archive/backups/peers and estimated work through a policy-filtered projection.
- Use risk-based approval separation for bulk, cross-mission, command/safety, security, archive, or irreversible actions; this report does not impose universal dual control.
- Make jobs cancelable only before declared irreversible boundaries. Cancellation itself is audited.
- Rate-limit and queue bulk archive/export/restore/purge; avoid long database transactions.
- Expose terminal receipts and partial-copy limitations to authorized administrators.
- Treat lifecycle endpoints and metrics as high-value security surfaces; defend against enumeration, deletion abuse, timing leakage, forged holds and downgrade to weaker policy.

Standards-facing lifecycle fields are limited to approved CSAPI behavior and ordinary HTTP semantics. Glaux lifecycle operation/status, hold, archive and proof resources are extensions and should be advertised in the API description only after IDR-SRV-031 onward defines their public contract.

---

## 16. Fixture and Verification Matrix with Expected Outcomes

| Fixture/test | Setup/action | Expected outcome/evidence |
|---|---|---|
| Normal expiry boundary | Evaluate immediately before, exactly at, and after policy deadline with virtual policy clock | Only eligible at defined inclusive/exclusive boundary; policy/time evidence exact |
| Missing/disputed/future trigger | Record lacks trusted trigger or has correction conflict | No auto-delete; exception created; no silent ingest-time substitution |
| Hold before eligibility | Apply scoped hold then advance virtual clock | Record preserved in every governed store; queries/access unchanged except hold visibility |
| Hold/disposition race | Inject hold before and after terminal purge boundary | Before wins and stops work; after records precise completed scope/high-severity exception |
| Hold release | Release with correct authority | Recomputed deadline/action under current policy; no immediate unexplained purge |
| Policy shorter/longer | Activate new immutable policy | Dry-run diff; shorter requires configured approval; longer cancels reversible pending work |
| System delete without cascade | System has child and Deployment relation | Rejected; no partial detach/delete |
| System cascade | Owned children plus Deployment association | Atomic visible tombstone/removal plan and Deployment unlink; foreign edges block or detach by rule |
| DataStream/ControlStream default delete | Stream has child records | 409 and unchanged graph |
| Stream cascade with hold | Child Observation/Command is held | Request rejected or staged as conflict; held evidence not silently destroyed |
| Command cancellation | Post CANCELED status | Command remains; immutable final status appended |
| Resource historical purge | Revision still referenced by provenance/package/hold | Not eligible; dependency explains blocker |
| Safe tombstone | Delete sensitive named resource | Public 404; authorized marker contains minimum fields only; no title/hash leakage |
| Raw-to-aggregate | Numeric stream with gaps/nil/correction | Aggregate records method, coverage, nil/quality, watermark and provenance; raw purge only after rule/gates |
| Forbidden aggregate | Status/event/command/categorical samples | Numerical averaging rejected |
| Latest after raw expiry | Retain authorized derived latest; purge raw | Latest labeled derived/source unavailable; no historical query claim |
| Quarantine isolation | Submit hostile/secret payload | Never searchable/exported/cached; diagnostics redacted; bounded policy action evidenced |
| Archive publication crash | Fail during upload, verify and catalog commit | No partial authoritative archive; staging reconciled by lease |
| Corrupt archive restore | Alter byte/manifest or omit package | Restore remains isolated/failed; nothing published |
| Valid restore | Restore prior backup containing later-deleted record | Current deletion ledger reapplied before serving; deleted resource does not reappear |
| Index/cache negative proof | Delete authority while caches/index workers fail | Stale surfaces suppressed; after repair all authorized queries miss target |
| Versioned object delete | Backend retains old versions/delete markers | Job remains incomplete until governed versions handled or limitation recorded |
| Backup expiry chain | Incremental depends on older base | Base cannot expire until all dependent supported restores close |
| PITR replay | Restore point precedes deletion | Post-restore deletion/tombstone replay removes it before access |
| Broker redelivery | Tombstone delivered twice/out of order | Consumer deduplicates; stale update cannot resurrect |
| DDIL peer reconnect | Peer holds stale update and missed deletion | Tombstone/causal exchange wins or explicit conflict; no silent current recreation |
| Unavailable peer/export | Deadline passes without access to copy | Core job records exception/limited proof; no false global deletion claim |
| Cryptographic erase eligibility | Shared key covers retained and deleted records | Per-record cryptographic erase rejected |
| Media sanitization | Decommission test media under selected method | Inventory, method/tool, verification and disposition receipt match approved program |
| Authorization/leakage | Cross-tenant user previews/deletes/reads status | Denied without revealing target, policy, hold, counts or topology |
| Idempotent retry | Repeat exact disposition request; then reuse key with different intent | Same result for exact retry; 409 for conflicting intent |
| Partial cascade failure | Worker fails on one derived store | Authority state consistent; job partial/failed with retry; completed scopes not repeated |
| Accelerated-clock equivalence | Run same fixture under virtual and wall clock | Same trigger ordering/boundaries; only elapsed test duration differs |

Verification is not a single row-count assertion. It combines positive preservation tests for held/archived data; negative authorized queries across current/history/search/cache/replay paths; manifest and digest checks; restore drills; copy acknowledgements; policy/authorization leakage tests; and explicit limitations for uncontrollable copies.

---

## 17. Implementation and Operational Implications

### 17.1 Logical components

| Component | Responsibility | Key boundary |
|---|---|---|
| Policy registry/compiler | Validate signed/versioned rules and compile deterministic selectors/actions | Invalid/ambiguous policies cannot authorize destruction |
| Lifecycle evaluator | Calculate state, trigger, deadline, hold and eligibility | Pure/repeatable evaluation with clock injection |
| Dependency/copy inventory | Traverse graph, manifests, storage adapters and peer/export registry | Policy-scoped, cycle-safe, bounded, authority-aware |
| Disposition coordinator | Commit request, tombstone/job/outbox and manage state machine | PostgreSQL transaction from IDR-SRV-029 |
| Archive/restore orchestrator | Stage, manifest, verify, publish, isolate restore and reapply ledger | No partial authority |
| Storage adapters | Delete/list versions, verify absence, describe retention/lock limits | Capability evidence, not policy authority |
| Projection invalidator | Remove/rebuild cache, index, materialized/latest/search state | Never serve known-stale sensitive data |
| Peer/broker coordinator | Publish causal tombstones, track acknowledgement and retry | At-least-once and DDIL-aware |
| Evidence/receipt service | Produce protected audit/provenance and safe public/admin projections | No secret/payload duplication |

### 17.2 Persistence direction

Use versioned tables/records for policy, hold, evaluation, lifecycle event, tombstone, job, job step, copy target, archive manifest, restore, deletion receipt, peer acknowledgement and exception. Exact DDL remains implementation work. Workers use short transactions, deterministic ordering, leases, idempotency and atomic outbox patterns from IDR-SRV-029. PostgreSQL advisory/database constraints must not be the only proof of external copy work.

Partition dynamic families around query/maintenance needs, but compute eligibility independently. A partition can be dropped only after a set-level proof that every member shares compatible policy, closed correction window, no hold/dependency, completed archive if required, and copy plan. Otherwise use selective disposition or re-partitioning.

### 17.3 Deployment-profile feasibility

- **Local development:** deterministic virtual clock; filesystem/CAS adapter; no production purge by default; disposable fixtures and full receipts.
- **Public demonstration:** synthetic/public data policy package; explicit short-lived managed exports only if approved; restore/deletion behavior still truthful.
- **Reference deployment:** PostgreSQL authority, optional object adapter, scheduled lifecycle workers, protected admin API, backup/restore and observability handoffs.
- **Tactical/DDIL:** bounded local storage policy, reference-closed archives/packages, causal tombstones, unavailable-peer exceptions, and reconnect-before-promotion.

### 17.4 Operations and observability

Measure policy coverage/uncovered classes, upcoming eligibility by safe aggregate, held counts, evaluation exceptions, archive/purge/restore backlog and age, job attempts/failures, staging/orphans, integrity-scrub failures, last successful restore drill, cache/index invalidation lag, outbox/broker/peer tombstone acknowledgements, unavailable peers, backup-chain coverage/expiry, and storage reclaimed. Metrics avoid resource names, payloads, sensitive reasons, tenant leakage and high-cardinality identifiers.

Runbooks cover policy activation/rollback, hold emergency, queue pause, archive corruption, backup-chain conflict, unavailable peer, capacity emergency, accidental delete before/after terminal boundary, restore validation and media disposition. Configuration/secrets are owned by IDR-SRV-047; observability implementation by IDR-SRV-048; migration/backup/restore technology by IDR-SRV-049.

### 17.5 Downstream handoffs

| Topic(s) | Required handoff |
|---|---|
| IDR-SRV-031-033 | admission/raw/quarantine/temporary lease, source/evidence and write-contract lifecycle |
| IDR-SRV-034-035 | observation/status/event retention, aggregation, outbox/broker/replay/subscriber tombstones; no draft Part 3 pre-authorization |
| IDR-SRV-036-038 | command/feasibility terminal/reconciliation gates, sensitive payload minimization, safety/accountability evidence |
| IDR-SRV-039-041 | lifecycle threat model, policy authority/releasability, hold/admin authorization, audit schema and schedule |
| IDR-SRV-042-043 | offline policy package, causal tombstone, peer acknowledgement, conflict and reconnect algorithms |
| IDR-SRV-044-046 | topology, worker/adapter placement, architecture and operational state machines |
| IDR-SRV-047 | policy/storage/hold configuration and cryptographic key references; no secrets in lifecycle evidence |
| IDR-SRV-048 | metrics, alerts, protected logs and runbooks |
| IDR-SRV-049 | migration, backup chain, WAL, RPO/RTO, archive media, restore and sanitization technology |
| IDR-SRV-050-053 | normative/API conformance, full fixture corpus, model/property/fuzz and golden evidence |
| IDR-SRV-054-055 | volume, partition, archive, purge, restore, cache invalidation and DDIL performance/resilience benchmarks |
| IDR-SRV-056 | external client behavior for 202/status/404/410/cascade and truthful copy limitations |

---

## 18. Recommendations, Rejected Options, Unresolved Questions, and Review Triggers

### 18.1 Recommendations

1. **Adopt a versioned policy registry and fail closed for destructive automation. [High]** Every action must trace to authority, scope, trigger and version; absent/conflicting authority creates an exception.
2. **Use orthogonal semantic, custody, storage, disposition and media states. [High]** Never use a single deleted/archive/status flag.
3. **Select no retention duration in architecture code or this report. [High]** Require deployment/profile policy packages with automated coverage validation.
4. **Treat HTTP/CSAPI deletion as an API/domain operation, not universal erasure. [High]** Publish precise 202/204 and status/receipt guarantees.
5. **Honor CSAPI cascade and 409 rules through an authority-aware dependency plan. [High]** Do not let cascade override holds, evidence or foreign ownership.
6. **Preserve policy-safe identity, causality and provenance without retaining unnecessary content. [High]** Tombstone fields and hashes are subject to minimization.
7. **Use catalog-rooted dependency mark-and-sweep for immutable artifacts. [High]** Include holds, releases, packages, archive manifests, jobs and DDIL roots.
8. **Permit dynamic compaction/downsampling only as provenance-bearing derived data. [High]** Raw purge requires explicit substitution policy and verification.
9. **Give command and unknown-outcome evidence the strongest eligibility gates. [High]** Cancellation remains a status; unresolved DDIL outcomes are not auto-eligible.
10. **Make quarantine isolated and bounded by an approved rule. [High]** Minimize hostile/sensitive content and diagnostics.
11. **Adopt the reference-closed archive and isolated restore contract. [High]** Reapply later deletion/tombstone state before serving.
12. **Track disposition across every material copy. [High]** Include indexes, caches, objects/versions, outbox, broker, subscribers, replicas, exports, backups/WAL and peers.
13. **Use causal tombstones and acknowledgements to prevent DDIL resurrection. [High]** Unknown/unavailable peers remain explicit exceptions.
14. **Reserve cryptographic erase for proven exclusive key scopes. [High]** Treat media sanitization as a deployment program under NIST guidance.
15. **Require end-to-end lifecycle fixtures and restore drills before enabling automatic purge. [High]** Negative absence, positive hold preservation and disclosure-limit tests are mandatory.

### 18.2 Rejected or conditional options

| Option | Decision | Reason |
|---|---|---|
| One global TTL | Reject | Confuses authorities, classes, clocks, holds and dependencies |
| Hard-coded numeric defaults without approval | Reject | Invents policy and creates destructive behavior |
| DELETE means immediate physical erasure | Reject | Contradicts HTTP semantics and actual storage/recovery copies |
| One deleted boolean | Reject | Cannot express archive, hold, tombstone, purge, backup or media state |
| Cascade every graph edge | Reject | Violates ownership/authority and destroys evidence |
| Retain all raw/audit/command data forever | Reject | Unbounded sensitive exposure and no authority |
| Drop partition solely by age | Reject | Partition time does not prove eligibility of every row |
| Naive artifact reference counting | Reject | Misses bitemporal, archive, hold, repair and package roots |
| Aggregate then automatically delete raw | Reject | Aggregation is lossy and requires explicit substitution authority |
| Average status/event/command data | Reject | Destroys categorical, ordering and safety semantics |
| Restore backup directly into service | Reject | Can resurrect deleted/unauthorized state |
| S3 delete marker equals purge | Reject | Prior object versions can remain |
| Shared-key cryptographic erase for one record | Reject | Destroys unrelated data or leaves copies decryptable |
| Global deletion proof for external exports/unavailable peers | Reject | Unverifiable and outside Glaux control |
| PostgreSQL partitioning/archive/object products as policy | Reject | Products implement actions; they do not supply authority |

### 18.3 Unresolved questions and owners

| Question | Why unresolved | Owner/next action |
|---|---|---|
| Actual per-class online/archive/purge/tombstone/replay/backup periods | No controlling schedule supplied | Project/profile, organization, mission and security authorities provide approved policy package |
| Exact hold types, precedence and emergency authorities | Authority-specific | IDR-SRV-039-041 with project policy owner |
| Which tombstones are publicly discoverable and for how long | Privacy/security/interoperability tradeoff | IDR-SRV-032/039/040 and profile decision |
| Minimum command/accountability evidence and disposition | Safety/audit/security profile required | IDR-SRV-036-041 |
| Exact raw/source-byte retention requirements | Source contracts and interoperability/accountability vary | IDR-SRV-031-034/040/041 plus source authority |
| Archive format, durability, RPO/RTO and restore topology | Deployment objectives absent | IDR-SRV-044/045/049 |
| Broker/subscription replay and consumer acknowledgement horizon | Transport/draft Part 3 decision not yet authorized | IDR-SRV-035 after separate authorization |
| DDIL tombstone horizon and required peer set | Deployment graph/offline assumptions absent | IDR-SRV-042/043 |
| Approved encryption/key design and sanitization methods per media | Deployment/security choice | IDR-SRV-039/044/047/049 under system owner |
| External export recipient obligations | Contract/policy-specific | IDR-SRV-039/040 and deployment terms |

### 18.4 Review triggers

Revisit this strategy when an AEP/profile or retention schedule is adopted; a law/contract/source restriction changes; new CSAPI/OAF transactional or Part 3 publication text is approved; command/audit/security models mature; a broker/object store/backup product is selected; deployment topology, DDIL horizon, tenant model or RPO/RTO changes; media/encryption architecture changes; restore or deletion tests reveal gaps; or a lifecycle/security incident occurs.

---

## 19. Validation Against This Plan's Success Criteria

| Topic Plan Success Criterion | Validation Status | Evidence |
|---|---|---|
| Every core question is answered or explicitly unresolved with a next action | Met | Section 2 and Section 18.3 |
| Standards/profile claims cite exact requirements or controlled references | Met | Sections 3-4 and footnotes |
| No unsupported retention period or legal/mission obligation appears | Met | Sections 1, 3.3, 4.2, 7-8 and 18.3 |
| Every data/artifact class has owner, role, state model and disposition or N/A | Met | Sections 5, 6 and 8 |
| Active, superseded, expired, archived, held, tombstoned, purged, backup-expired and sanitized are distinguished | Met | Section 6.1 |
| Triggers, holds, precedence, policy change and deadline behavior are defined | Met | Section 7 |
| Referential, cascade, provenance, audit, command-safety and tombstone rules are defined | Met | Sections 4, 9 and 11 |
| Archive manifests, integrity, restoration validation and failure are testable | Met | Sections 12 and 16 |
| API deletion differs from asynchronous purge and physical sanitization | Met | Sections 1, 13 and 15 |
| Indexes, caches, brokers, outboxes, replicas, exports, backups and peers are included | Met | Sections 5 and 14 |
| DDIL/reconnect prevents silent reappearance | Met | Sections 12.2, 14.1 and 16 |
| Required fixture categories have explicit expected outcomes | Met | Section 16 |
| Configuration, backup, audit and synchronization details are handed off | Met | Sections 17.4-17.5 and 18.3 |
| Recommendations are feasible and decision-usable | Met | Sections 17.3 and 18.1-18.2 |
| Report is polished, recommendation-first, independently readable and self-contained | Met | Entire report, especially Sections 1-4 |

The report is complete as research and is **In Review**. It is not accepted for downstream use until the Glaux Project Lead records acceptance in this report, its topic plan, and the overall plan. IDR-SRV-031 and later work remain unauthorized.

---

## 20. References and Sources

### 20.1 Standards, guidance, and project sources

- [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html)
- [Official OGC API - Connected Systems repository, approved v1.0.0 source pin](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [OGC SensorML Encoding Standard 3.0, OGC 23-000](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model Encoding Standard 3.0, OGC 24-014](https://docs.ogc.org/is/24-014/24-014.html)
- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html)
- [W3C PROV-DM: The PROV Data Model](https://www.w3.org/TR/prov-dm/)
- [NIST SP 800-88 Revision 2: Guidelines for Media Sanitization](https://csrc.nist.gov/pubs/sp/800/88/r2/final)
- [NIST SP 800-53 Revision 5, Release 5.2.0](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-030 Research Plan](../IDR%20Plans/idr-srv-030-data-lifecycle-retention-archival-and-deletion-strategy.md)
- [OGC API - Connected Systems Upstream-History Register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
- Accepted [IDR-SRV-015](idr-srv-015-canonical-glaux-server-resource-model-report.md), [IDR-SRV-016](idr-srv-016-identifier-uri-and-resource-lifecycle-strategy-report.md), [IDR-SRV-017](idr-srv-017-relationship-and-linkage-model-report.md), [IDR-SRV-018](idr-srv-018-temporal-validity-and-freshness-model-report.md), [IDR-SRV-019](idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md), and [IDR-SRV-020](idr-srv-020-status-availability-and-system-event-model-report.md)
- Accepted [IDR-SRV-021](idr-srv-021-sensorml-representation-strategy-report.md), [IDR-SRV-022](idr-srv-022-swe-common-data-component-strategy-report.md), [IDR-SRV-023](idr-srv-023-schema-and-encoding-validation-strategy-report.md), and [IDR-SRV-024](idr-srv-024-units-observed-properties-and-semantic-binding-strategy-report.md)
- Accepted [IDR-SRV-025](idr-srv-025-database-and-persistence-architecture-options-report.md), [IDR-SRV-026](idr-srv-026-geospatial-storage-and-query-strategy-report.md), [IDR-SRV-027](idr-srv-027-time-series-observation-storage-strategy-report.md), [IDR-SRV-028](idr-srv-028-metadata-and-document-storage-strategy-report.md), and [IDR-SRV-029](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md)
- Controlled project source: AC/224(JCGISR)D(2026)0005, April 27, 2026, SHA-256 56dc757b6e677b3584e3152a957849f21a24b22854f562613ff283a8b599da8c (not redistributed; used only through accepted findings)

### 20.2 Primary technology sources

- [PostgreSQL 18: Routine Vacuuming](https://www.postgresql.org/docs/18/routine-vacuuming.html)
- [PostgreSQL 18: Table Partitioning](https://www.postgresql.org/docs/18/ddl-partitioning.html)
- [PostgreSQL 18: Backup and Restore](https://www.postgresql.org/docs/18/backup.html)
- [PostgreSQL 18: Continuous Archiving and Point-in-Time Recovery](https://www.postgresql.org/docs/18/continuous-archiving.html)
- [Amazon S3 DeleteObject API](https://docs.aws.amazon.com/AmazonS3/latest/API/API_DeleteObject.html)
- [Amazon S3 Delete Markers](https://docs.aws.amazon.com/AmazonS3/latest/userguide/DeleteMarker.html)

### 20.3 Footnotes

[^1]: Joint Task Force, [NIST SP 800-53 Revision 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final), Release 5.2.0 issued August 27, 2025, checked September 14, 2026. The publication describes a flexible, customizable control catalog tied to organizational mission, business, legal, policy and risk inputs; this report does not treat it as a retention schedule.
[^2]: IETF, [RFC 9110 §9.3.5 DELETE](https://www.rfc-editor.org/rfc/rfc9110.html#section-9.3.5), June 2022, checked September 14, 2026. DELETE removes the target URI's association with current functionality; representations and storage may or may not be destroyed/reclaimed. The RFC recommends 202 when not yet enacted and 204 or 200 when enacted.
[^3]: W3C, [PROV-DM §§5.1.8 and 5.2](https://www.w3.org/TR/prov-dm/), Recommendation of April 30, 2013, checked September 14, 2026. PROV models entities, activities, agents, derivation/revision and invalidation; invalidation begins destruction, cessation or expiry and is not a physical-erasure attestation.
[^4]: PostgreSQL Global Development Group, [PostgreSQL 18 Routine Vacuuming](https://www.postgresql.org/docs/18/routine-vacuuming.html), PostgreSQL 18.6 documentation checked September 14, 2026. UPDATE/DELETE does not immediately remove old row versions; VACUUM reclaims them for reuse, while ordinary VACUUM generally does not return that space to the operating system.
[^5]: PostgreSQL Global Development Group, [PostgreSQL 18 Continuous Archiving and PITR](https://www.postgresql.org/docs/18/continuous-archiving.html), PostgreSQL 18.6 documentation checked September 14, 2026. Base backups and a continuous WAL sequence can restore whole-cluster prior states; incremental backup dependencies must be tracked by the operator.
[^6]: Amazon Web Services, [DeleteObject](https://docs.aws.amazon.com/AmazonS3/latest/API/API_DeleteObject.html) and [Delete Markers](https://docs.aws.amazon.com/AmazonS3/latest/userguide/DeleteMarker.html), checked September 14, 2026. In a versioning-enabled bucket, a simple delete creates a current delete marker and prior versions remain; used only as representative object-store evidence.
[^7]: Ramaswamy Chandramouli and Eric Hibbard, [NIST SP 800-88 Revision 2](https://csrc.nist.gov/pubs/sp/800/88/r2/final), final September 26, 2025, checked September 14, 2026. NIST defines media sanitization by infeasibility of target-data access for a given effort and directs organizations/system owners to establish sensitivity-informed programs. The publication distinguishes clear, purge and destroy and discusses cryptographic erase; exact deployment technique remains an adoption decision.

---

## Report Completion Checklist

- [x] Topic ID matches overall research plan index
- [x] Topic research plan is linked and aligned
- [x] Core research questions are covered or explicitly unresolved
- [x] Findings are evidence-backed with reproducible references
- [x] Normative and informative evidence are classified and not conflated
- [x] Mutable sources identify a version, release, tag, commit, or dated retrieval
- [x] Controlled, inaccessible, missing, or ambiguous evidence limitations are explicit
- [x] Source-backed findings, analyst inference, and project recommendations are distinguishable
- [x] Conflicts with accepted prior reports are reconciled or explicitly escalated
- [x] Executive summary is independently readable
- [x] Recommendations are explicit and actionable
- [x] Risks and open questions are documented
- [x] Success criteria validation is complete
- [ ] Plan-owner acceptance and acceptance date are recorded
- [x] Next steps and owners are identified

---

**Review boundary:** IDR-SRV-030 research is complete and submitted for Glaux Project Lead review on September 14, 2026. No acceptance, later topic, draft Part 3 implementation, or server implementation is authorized by this report.

# Final Glaux Server Initial Design Research Synthesis Report

**Overall Report Status:** Final<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Final Report Model:** Indexed synthesis topic<br>
**Synthesis Topic ID:** IDR-SRV-057<br>
**Synthesis Topic Research Plan:** [IDR-SRV-057 Research Plan](../IDR%20Plans/idr-srv-057-final-glaux-server-idr-synthesis-report.md)<br>
**Scope:** Initial design research for the Glaux Server component, from standards obligations through an evidence-backed implementation and verification baseline<br>
**Reporting Period:** June 7 – September 16, 2026<br>
**Total Topic Plans in Index:** 67<br>
**Total Topic Reports Completed:** 67<br>
**Total Topic Reports Accepted:** 67<br>
**Program Owner:** Glaux Project Lead<br>
**Report Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 16, 2026<br>
**Date:** September 16, 2026<br>
**Last Updated:** September 16, 2026

**Supplemental material:** [Addendum A: Draft CSAPI Part 4 Sampling Features](#addendum-a-draft-csapi-part-4-sampling-features) and [Addendum B: Enhanced CSAPI Querying and Spatial Observation Retrieval](#addendum-b-enhanced-csapi-querying-and-spatial-observation-retrieval), prepared September 17, 2026 from accepted IDR-SRV-058/059 research; [Addendum C: CSAPI Part 5 Protobuf-First Implementation](#addendum-c-csapi-part-5-protobuf-first-implementation), prepared September 18, 2026 from accepted IDR-SRV-060 research; [Addendum D: Provenance and Related Metadata Interoperability](#addendum-d-provenance-and-related-metadata-interoperability), prepared September 18, 2026 from accepted IDR-SRV-061 research; [Addendum E: CS-GO Engineering Practices and Development History](#addendum-e-cs-go-engineering-practices-and-development-history), prepared September 18, 2026 from accepted IDR-SRV-062 research. The metadata, counts and acceptance above describe the original report; each addendum records its own status and does not change the original completion record.

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Scope and Purpose](#2-scope-and-purpose)
3. [Evidence Base and Corpus Readiness](#3-evidence-base-and-corpus-readiness)
4. [Research Methodology and Synthesis Approach](#4-research-methodology-and-synthesis-approach)
5. [Standards and Obligation Baseline](#5-standards-and-obligation-baseline)
6. [Conformance and Requirements Baseline](#6-conformance-and-requirements-baseline)
7. [API Surface and Behavior Baseline](#7-api-surface-and-behavior-baseline)
8. [Resource Model, Identifier, Lifecycle, and Linkage Baseline](#8-resource-model-identifier-lifecycle-and-linkage-baseline)
9. [Representation, Validation, Schema, Semantic Binding, and OpenAPI Baseline](#9-representation-validation-schema-semantic-binding-and-openapi-baseline)
10. [Persistence, Geospatial, Time-Series, Metadata, Migration, and Continuity Baseline](#10-persistence-geospatial-time-series-metadata-migration-and-continuity-baseline)
11. [Dynamic Data, Ingestion, Streaming, and Event Baseline](#11-dynamic-data-ingestion-streaming-and-event-baseline)
12. [Command, Control, Feasibility, Safety, and Audit Baseline](#12-command-control-feasibility-safety-and-audit-baseline)
13. [Security, Authorization, Policy, Releasability, and Source-Trust Baseline](#13-security-authorization-policy-releasability-and-source-trust-baseline)
14. [DDIL, Caching, Synchronization, and Conflict Baseline](#14-ddil-caching-synchronization-and-conflict-baseline)
15. [Rust, Service Architecture, Modularization, and Deployment Baseline](#15-rust-service-architecture-modularization-and-deployment-baseline)
16. [Configuration, Observability, Backup, Restore, and Operational Reference](#16-configuration-observability-backup-restore-and-operational-reference)
17. [Verification Baseline](#17-verification-baseline)
18. [First-Implementation Scope](#18-first-implementation-scope)
19. [Implementation Sequencing and Deferred Full Scope](#19-implementation-sequencing-and-deferred-full-scope)
20. [Proof-of-Concept Candidates](#20-proof-of-concept-candidates)
21. [Risk Register](#21-risk-register)
22. [Open Decision Register](#22-open-decision-register)
23. [Candidate Work Packages](#23-candidate-work-packages)
24. [Documentation and Governance Updates](#24-documentation-and-governance-updates)
25. [Final Recommendation Summary](#25-final-recommendation-summary)
26. [Validation Against Success Criteria](#26-validation-against-success-criteria)
27. [References and Topic Traceability Index](#27-references-and-topic-traceability-index)

Supplements: [Addendum A: Draft CSAPI Part 4 Sampling Features](#addendum-a-draft-csapi-part-4-sampling-features); [Addendum B: Enhanced CSAPI Querying and Spatial Observation Retrieval](#addendum-b-enhanced-csapi-querying-and-spatial-observation-retrieval); [Addendum C: CSAPI Part 5 Protobuf-First Implementation](#addendum-c-csapi-part-5-protobuf-first-implementation); [Addendum D: Provenance and Related Metadata Interoperability](#addendum-d-provenance-and-related-metadata-interoperability); [Addendum E: CS-GO Engineering Practices and Development History](#addendum-e-cs-go-engineering-practices-and-development-history)

---

## 1. Executive Summary

The Glaux Server Initial Design Research objective has been met at the research level. Sixty-six prerequisite topic reports were completed and accepted, and this indexed synthesis is the sixty-seventh and final report. The corpus defines a coherent server baseline spanning the NATO contribution context, OGC API - Connected Systems Parts 1 and 2, resource and representation semantics, persistence, ingestion, event delivery, safe simulated tasking, security and policy, DDIL behavior, Rust architecture, deployment, continuity, and verification. It is sufficient to begin the Implementation Guide, requirements register, Architecture Decision Records, repository scaffolding, and dependency-based Roadmap. It is not implementation evidence and does not establish production, operational, accreditation, certification, or cross-domain readiness.

The recommended direction is a Rust modular monolith backed by PostgreSQL/PostGIS. One canonical, encoding-neutral domain and application core owns validation, authorization, policy, transactions, audit, and effects. HTTP, publisher, simulator, event, persistence, and future synchronization concerns remain adapters around that core. The first credible implementation should progress through evidence-producing vertical slices: establish the workspace and safety baseline; deliver Part 1 discovery and read behavior over real persistence; add controlled writes and dynamic observations; add durable outbox-backed Server-Sent Events; then add Part 2 control resources with a non-network simulator only. Experimental Part 3 MQTT publication, richer encodings, two-node synchronization, operational infrastructure, and physical command effects follow only after their stated gates.

The standards target is deliberately complete but incrementally claimed. The end state remains all 25 identified direct CSAPI conformance classes, 233 requirements, five recommendations, and 240 abstract-test cases in the accepted baseline. An implementation may claim only the classes and behaviors actually implemented and evidenced. Published OGC Parts 1 and 2 are controlling. The approved Version 1.0.0 tag and published specifications are separated from post-publication maintenance on the default branch, open issue proposals, and the Part 3 working draft. A September 16, 2026 refresh found no material change from the shared upstream-history evidence register: the relevant published tag, default-branch commit, Part 3 working-draft commit, and tracked open issues retain the dispositions established by the owning reports.

The central architectural principle is that no transport, document format, cache, broker, peer implementation, or deployment profile becomes an independent source of truth. Canonical resources and typed relationships are authoritative in PostgreSQL/PostGIS; exact source artifacts remain immutable and content-addressed; dynamic changes commit atomically with durable publication records; authorized views are computed before filtering, counting, paging, streaming, or documentation exposure; and command effects require separate authorization, policy, feasibility, safety, approval, target trust, and dispatch gates. DDIL conditions never widen authority or justify invented currentness.

The primary remaining risks are breadth, standards artifact inconsistencies, semantic preservation across SensorML/SWE encodings, policy-correct querying, command safety, event replay and backpressure, temporal/spatial scale, synchronization conflict governance, and mistaking a successful public demonstration for operational readiness. Each is assigned a proof, verification lane, or governance decision below. None blocks drafting downstream artifacts. They do block unqualified implementation-readiness, production-capacity, certification, accreditation, and physical-command claims.

**Readiness decision:** research baseline complete and accepted; downstream design and planning artifacts are ready to draft; implementation claims remain evidence-gated.

---

## 2. Scope and Purpose

### 2.1 Objective response

The controlling plan asked for a complete, traceable initial design baseline for Glaux Server and for evidence-backed inputs to later implementation planning. This report answers that objective by reconciling every indexed research topic into one decision baseline, classifying the first implementation and deferred scope, identifying dependencies and proofs, and handing forward candidate work packages without substituting for an Implementation Guide or Roadmap.

The server is treated as the standards-facing persistence, discovery, dynamic-data, event, policy, and tasking component of the wider Glaux ecosystem. The research covers behavior and boundaries that the server must own. It identifies contracts for publisher, simulator, web, mobile, identity/policy, observability, and synchronization peers without treating those future components as already available.

### 2.2 In scope

- NATO contribution context and project-controlling AEP-4789 material, summarized without asserting promulgation.
- Published OGC API - Connected Systems Parts 1 and 2 and their inherited HTTP, OGC API - Features, SensorML 3.0, and SWE Common 3.0 obligations.
- API discovery, resources, navigation, query, negotiation, errors, OpenAPI, versioning, and compatibility.
- Canonical resource, identity, relationship, temporal, provenance, quality, trust, status, availability, and event semantics.
- PostgreSQL/PostGIS persistence, exact artifacts, observation storage, transactions, lifecycle, migrations, backup, and restore.
- Writes, publishers, simulators, observations, streaming, Part 2 tasking, policy, security, audit, DDIL, and synchronization boundaries.
- Rust platform, module, deployment, configuration, observability, and verification strategies.
- Experimental evaluation of draft Connected Systems Part 3 publish/subscribe, without claiming that draft as a normative Glaux conformance target.

### 2.3 Explicit boundaries

- This is research, not implementation, an operational design approval, or evidence that any service exists.
- The April 27, 2026 NATO package used by the project is a project-controlling ratification draft; this report does not claim NATO promulgation.
- Related NATO standards are external integration boundaries unless separately authorized; Glaux does not reproduce their systems.
- Cross-domain transfer, guard/CDS behavior, enterprise federation governance, real operational policies, identities, labels, data, targets, credentials, and infrastructure remain outside this initial implementation baseline.
- Physical command adapters and effects are deferred pending operational authority, safety engineering, accreditation, and explicit approval.
- Deployment topology, numeric service objectives, retention periods, RPO/RTO values, and production capacity are not selected by research alone.
- The report provides sequencing and work-package inputs, not dates, staffing commitments, milestone promises, or a substitute Roadmap.

### 2.4 Downstream artifact response

| Downstream artifact | Readiness | Evidence and remaining action |
|---|---|---|
| Glaux Server Goal and Definition | Mostly Ready | The accepted 001–056 findings and Sections 5–19 define the complete goal and bounded first implementation. Refresh the existing document to incorporate the accepted scope, terminology, authority, safety, and claim boundaries. |
| Implementation Guide | Ready to Draft | Sections 5–18 provide behavioral and architecture decisions. The guide must turn them into exact schemas, module APIs, ADRs, route behavior, acceptance criteria, and developer workflows. |
| Roadmap | Ready to Draft | Sections 19, 20, and 23 give dependencies, proof gates, and candidate work packages. The Roadmap must add project priorities, staffing, schedules, milestones, and release policy. |
| Requirements register | Ready to Build | IDR-SRV-006 through 008, 050, and 051 provide the obligation inventory, conformance structure, tests, and trace graph. Import and validate the accepted IDs rather than renumbering them. |
| ADR set | Ready to Draft | Cross-topic choices in Section 25 are mature enough for records; provisional tool versions and operational choices must remain explicitly revisitable. |
| Public demonstration plan | Mostly Ready | A safe profile and provisional envelope exist, but the demonstrator must be built, tested, TLS-fronted, synthetic-only, and command-disabled before publication. |

---

## 3. Evidence Base and Corpus Readiness

### 3.1 Prerequisite audit

The synthesis inventory found all 66 reports preceding IDR-SRV-057 at their planned paths. Every report records Report Status: Final, and every topic is recorded as accepted in the controlling overall-plan ledger and progress table. There are no approved prerequisite exceptions, missing reports, partial topics, or unresolved acceptance dependencies.

Several older topic plans retain legacy status wording such as In Review or Complete even though their reports are Final and the governing ledger records acceptance. Under the controlling governance rules, the accepted report and acceptance ledger establish prerequisite readiness. The plan-label drift is a documentation-maintenance item, not a research exception and not a reason to rewrite historical findings during synthesis.

### 3.2 Topic completion and coverage matrix

Links below are to the controlling topic plans and reports. “Accepted” means accepted by the Glaux Project Lead in the overall-plan ledger. IDR-SRV-057 and this final overall report were accepted on September 16, 2026.

| Topic ID | Topic title | Plan | Report | Completion | Acceptance / exception | Conclusion summary |
|---|---|---|---|---|---|---|
| IDR-SRV-001 | STANAG 4789 / AEP-4789 Server Obligation Baseline | [plan](../IDR%20Plans/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline.md) | [report](idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md) | Complete | Accepted | The April 2026 ratification draft is project-controlling context, not proof of promulgation. |
| IDR-SRV-002 | AEP-4789 Volume I Functional Mapping | [plan](../IDR%20Plans/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities.md) | [report](idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md) | Complete | Accepted | Server responsibilities span discovery, exchange, dynamics, tasking, status, security, federation, and DDIL. |
| IDR-SRV-003 | AEP-4789 Volume II Standards Package | [plan](../IDR%20Plans/idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline.md) | [report](idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md) | Complete | Accepted | CSAPI 1/2, SensorML 3, and SWE Common 3 form the coherent implementation package. |
| IDR-SRV-004 | Terminology and Concept Crosswalk | [plan](../IDR%20Plans/idr-srv-004-terminology-and-concept-crosswalk.md) | [report](idr-srv-004-terminology-and-concept-crosswalk-report.md) | Complete | Accepted | Canonical terms and context-specific mappings prevent false equivalence. |
| IDR-SRV-005 | Related NATO Standards Boundary Review | [plan](../IDR%20Plans/idr-srv-005-related-nato-standards-boundary-review.md) | [report](idr-srv-005-related-nato-standards-boundary-review-report.md) | Complete | Accepted | Adjacent standards are integration boundaries, not hidden core implementation scope. |
| IDR-SRV-006 | CSAPI Part 1 Requirement Baseline | [plan](../IDR%20Plans/idr-srv-006-csapi-part-1-requirement-baseline.md) | [report](idr-srv-006-csapi-part-1-requirement-baseline-report.md) | Complete | Accepted | Part 1 obligations require exact, class-scoped implementation and evidence. |
| IDR-SRV-007 | CSAPI Part 2 Requirement Baseline | [plan](../IDR%20Plans/idr-srv-007-csapi-part-2-requirement-baseline.md) | [report](idr-srv-007-csapi-part-2-requirement-baseline-report.md) | Complete | Accepted | Dynamic data and tasking obligations require durable temporal and lifecycle semantics. |
| IDR-SRV-008 | Conformance Class and Requirement Mapping | [plan](../IDR%20Plans/idr-srv-008-conformance-class-and-requirement-mapping.md) | [report](idr-srv-008-conformance-class-and-requirement-mapping-report.md) | Complete | Accepted | The accepted end-state inventory is 25 classes, 233 requirements, five recommendations, and 240 ATS cases. |
| IDR-SRV-009 | Landing Page, API Definition, and Conformance | [plan](../IDR%20Plans/idr-srv-009-landing-page-api-definition-and-conformance-declaration-behavior.md) | [report](idr-srv-009-landing-page-api-definition-and-conformance-declaration-behavior-report.md) | Complete | Accepted | Root-first discovery, live declarations, typed links, and active OpenAPI are mandatory. |
| IDR-SRV-010 | Collections, Resources, Links, and Navigation | [plan](../IDR%20Plans/idr-srv-010-collections-resources-links-and-navigation-behavior.md) | [report](idr-srv-010-collections-resources-links-and-navigation-behavior-report.md) | Complete | Accepted | Canonical resources and standards-defined navigation form one coherent graph. |
| IDR-SRV-010A | API Versioning and Compatibility | [plan](../IDR%20Plans/idr-srv-010a-api-versioning-backward-compatibility-and-deprecation-strategy.md) | [report](idr-srv-010a-api-versioning-backward-compatibility-and-deprecation-strategy-report.md) | Complete | Accepted | Version root, media/profile, migration, and deprecation policy together; do not imply silent compatibility. |
| IDR-SRV-011 | Query, Filtering, Sorting, and Pagination | [plan](../IDR%20Plans/idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics.md) | [report](idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md) | Complete | Accepted | Exact query semantics, authorized-view ordering, and bound cursors prevent silent misbehavior. |
| IDR-SRV-012 | Content Negotiation and Media Types | [plan](../IDR%20Plans/idr-srv-012-content-negotiation-media-types-and-encoding-selection.md) | [report](idr-srv-012-content-negotiation-media-types-and-encoding-selection-report.md) | Complete | Accepted | A registry must drive supported representations, 406/415 outcomes, and Vary behavior. |
| IDR-SRV-013 | Error Model and Failure Semantics | [plan](../IDR%20Plans/idr-srv-013-error-model-http-status-codes-and-failure-semantics.md) | [report](idr-srv-013-error-model-http-status-codes-and-failure-semantics-report.md) | Complete | Accepted | RFC 9457 problems, stable types, safe details, and correct status selection are the common failure contract. |
| IDR-SRV-014 | OpenAPI and API Documentation | [plan](../IDR%20Plans/idr-srv-014-openapi-description-and-api-documentation-strategy.md) | [report](idr-srv-014-openapi-description-and-api-documentation-strategy-report.md) | Complete | Accepted | Curated structure plus generated implementation truth and drift checks should describe actual behavior. |
| IDR-SRV-014A | OSH Server Study | [plan](../IDR%20Plans/idr-srv-014a-osh-csapi-server-implementation-study.md) | [report](idr-srv-014a-osh-csapi-server-implementation-study-report.md) | Complete | Accepted | OSH supplies informative mature patterns but cannot override published requirements. |
| IDR-SRV-014B | Connected Systems Go Study | [plan](../IDR%20Plans/idr-srv-014b-connected-systems-go-csapi-server-implementation-study.md) | [report](idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md) | Complete | Accepted | CS-Go offers useful implementation and Part 3 evidence while retaining product-specific choices. |
| IDR-SRV-014C | pygeoapi Study | [plan](../IDR%20Plans/idr-srv-014c-pygeoapi-csapi-server-implementation-study.md) | [report](idr-srv-014c-pygeoapi-csapi-server-implementation-study-report.md) | Complete | Accepted | Provider/configuration reuse is attractive only with semantic, validation, and consistency gaps closed. |
| IDR-SRV-014D | SECD Study | [plan](../IDR%20Plans/idr-srv-014d-secd-csapi-server-implementation-study.md) | [report](idr-srv-014d-secd-csapi-server-implementation-study-report.md) | Complete | Accepted | Black-box behavior supplies bounded interoperability lessons, not inferred architecture. |
| IDR-SRV-014E | OS4CSAPI Smoke-Test Findings | [plan](../IDR%20Plans/idr-srv-014e-os4csapi-client-smoke-test-findings-study.md) | [report](idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md) | Complete | Accepted | Failures must be attributed among server, client, fixture, harness, documentation, and standards. |
| IDR-SRV-014F | SECD Interoperability Findings | [plan](../IDR%20Plans/idr-srv-014f-secd-interoperability-findings-study.md) | [report](idr-srv-014f-secd-interoperability-findings-study-report.md) | Complete | Accepted | Freshness-qualified black-box evidence exposes discovery and semantic compatibility risks. |
| IDR-SRV-014G | OS4CSAPI Lessons Learned | [plan](../IDR%20Plans/idr-srv-014g-os4csapi-discussions-lessons-learned-study.md) | [report](idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md) | Complete | Accepted | Community observations are useful when separated from verified evidence and unimplemented proposals. |
| IDR-SRV-014H | Draft Part 3 Publish/Subscribe Study | [plan](../IDR%20Plans/idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study.md) | [report](idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md) | Complete | Accepted | Preserve the abstract event model; keep MQTT publication experimental, outbound, and non-conformant. |
| IDR-SRV-015 | Canonical Resource Model | [plan](../IDR%20Plans/idr-srv-015-canonical-glaux-server-resource-model.md) | [report](idr-srv-015-canonical-glaux-server-resource-model-report.md) | Complete | Accepted | One encoding-neutral graph distinguishes resources, facts, records, projections, schemas, and support concepts. |
| IDR-SRV-016 | Identifier, URI, and Lifecycle | [plan](../IDR%20Plans/idr-srv-016-identifier-uri-and-resource-lifecycle-strategy.md) | [report](idr-srv-016-identifier-uri-and-resource-lifecycle-strategy-report.md) | Complete | Accepted | Typed identities, UUIDv7 local IDs, stable canonical URLs, aliases, revisions, and tombstones are distinct. |
| IDR-SRV-017 | Relationship and Linkage Model | [plan](../IDR%20Plans/idr-srv-017-relationship-and-linkage-model.md) | [report](idr-srv-017-relationship-and-linkage-model-report.md) | Complete | Accepted | Relationships are typed facts with direction, authority, validity, integrity, and bounded traversal. |
| IDR-SRV-018 | Temporal, Validity, and Freshness | [plan](../IDR%20Plans/idr-srv-018-temporal-validity-and-freshness-model.md) | [report](idr-srv-018-temporal-validity-and-freshness-model-report.md) | Complete | Accepted | Valid, phenomenon, result, transaction, and ingest time plus freshness must not be collapsed. |
| IDR-SRV-019 | Provenance, Quality, and Trust | [plan](../IDR%20Plans/idr-srv-019-provenance-lineage-quality-and-trust-metadata-model.md) | [report](idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md) | Complete | Accepted | A PROV-compatible evidence graph preserves transformations, quality, uncertainty, and trust dimensions. |
| IDR-SRV-020 | Status, Availability, and Events | [plan](../IDR%20Plans/idr-srv-020-status-availability-and-system-event-model.md) | [report](idr-srv-020-status-availability-and-system-event-model-report.md) | Complete | Accepted | Status, availability, capability, observation-derived currentness, and system events remain separate. |
| IDR-SRV-021 | SensorML Representation | [plan](../IDR%20Plans/idr-srv-021-sensorml-representation-strategy.md) | [report](idr-srv-021-sensorml-representation-strategy-report.md) | Complete | Accepted | Preserve exact sources while using parsed, canonical, generated, and validation layers. |
| IDR-SRV-022 | SWE Common Data Components | [plan](../IDR%20Plans/idr-srv-022-swe-common-data-component-strategy.md) | [report](idr-srv-022-swe-common-data-component-strategy-report.md) | Complete | Accepted | Immutable versioned data contracts preserve the full model with capability-gated codecs. |
| IDR-SRV-023 | Schema and Encoding Validation | [plan](../IDR%20Plans/idr-srv-023-schema-and-encoding-validation-strategy.md) | [report](idr-srv-023-schema-and-encoding-validation-strategy-report.md) | Complete | Accepted | Versioned offline contracts and an evidence-producing staged pipeline govern every interaction. |
| IDR-SRV-024 | Units and Semantic Binding | [plan](../IDR%20Plans/idr-srv-024-units-observed-properties-and-semantic-binding-strategy.md) | [report](idr-srv-024-units-observed-properties-and-semantic-binding-strategy-report.md) | Complete | Accepted | Contextual semantic roles, UCUM-centered units, and versioned vocabularies prevent unsafe equivalence. |
| IDR-SRV-025 | Database and Persistence Architecture | [plan](../IDR%20Plans/idr-srv-025-database-and-persistence-architecture-options.md) | [report](idr-srv-025-database-and-persistence-architecture-options-report.md) | Complete | Accepted | PostgreSQL/PostGIS relational-hybrid storage is the authoritative baseline; add products only by evidence. |
| IDR-SRV-026 | Geospatial Storage and Query | [plan](../IDR%20Plans/idr-srv-026-geospatial-storage-and-query-strategy.md) | [report](idr-srv-026-geospatial-storage-and-query-strategy-report.md) | Complete | Accepted | Preserve source geometry and CRS while maintaining validated canonical/query projections. |
| IDR-SRV-027 | Time-Series Observation Storage | [plan](../IDR%20Plans/idr-srv-027-time-series-observation-storage-strategy.md) | [report](idr-srv-027-time-series-observation-storage-strategy-report.md) | Complete | Accepted | Native PostgreSQL partitioning is the baseline; TimescaleDB requires a measured decision gate. |
| IDR-SRV-028 | Metadata and Document Storage | [plan](../IDR%20Plans/idr-srv-028-metadata-and-document-storage-strategy.md) | [report](idr-srv-028-metadata-and-document-storage-strategy-report.md) | Complete | Accepted | Normalize authoritative graph state and catalog immutable content-addressed artifacts. |
| IDR-SRV-029 | Transactions, Idempotency, and Concurrency | [plan](../IDR%20Plans/idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy.md) | [report](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md) | Complete | Accepted | Constraints, conditional updates, scoped idempotency, and atomic inbox/outbox protect effects. |
| IDR-SRV-030 | Lifecycle, Retention, Archive, and Deletion | [plan](../IDR%20Plans/idr-srv-030-data-lifecycle-retention-archival-and-deletion-strategy.md) | [report](idr-srv-030-data-lifecycle-retention-archival-and-deletion-strategy-report.md) | Complete | Accepted | Policy-driven retention, holds, tombstones, archive, purge, and propagation require auditable workflows. |
| IDR-SRV-031 | Server Write and Ingestion Model | [plan](../IDR%20Plans/idr-srv-031-server-write-and-ingestion-model.md) | [report](idr-srv-031-server-write-and-ingestion-model-report.md) | Complete | Accepted | Server-owned admission through commit/publish ensures canonical reads expose committed state only. |
| IDR-SRV-032 | Publisher-to-Server Boundary | [plan](../IDR%20Plans/idr-srv-032-publisher-to-server-contract-boundary.md) | [report](idr-srv-032-publisher-to-server-contract-boundary-report.md) | Complete | Accepted | Publishers are authenticated external sources with bounded contracts, identity, trust, and replay rules. |
| IDR-SRV-033 | Simulator-to-Server Boundary | [plan](../IDR%20Plans/idr-srv-033-simulator-to-server-contract-boundary.md) | [report](idr-srv-033-simulator-to-server-contract-boundary-report.md) | Complete | Accepted | A synthetic non-network simulator tests server behavior without implying physical effects. |
| IDR-SRV-034 | Dynamic Update Semantics | [plan](../IDR%20Plans/idr-srv-034-datastream-observation-and-status-update-semantics.md) | [report](idr-srv-034-datastream-observation-and-status-update-semantics-report.md) | Complete | Accepted | Stream contracts, multiple clocks, latest projections, and schema transitions require exact semantics. |
| IDR-SRV-035 | Streaming and Event Publication | [plan](../IDR%20Plans/idr-srv-035-streaming-and-event-publication-strategy.md) | [report](idr-srv-035-streaming-and-event-publication-strategy-report.md) | Complete | Accepted | Durable domain event/outbox plus SSE is first; experimental MQTT is a later profile. |
| IDR-SRV-036 | Control Stream and Command Lifecycle | [plan](../IDR%20Plans/idr-srv-036-control-stream-and-command-lifecycle-model.md) | [report](idr-srv-036-control-stream-and-command-lifecycle-model-report.md) | Complete | Accepted | ControlStream, Command, feasibility, and the nine Part 2 command statuses remain distinct. |
| IDR-SRV-037 | Feasibility and Asynchronous Tasking | [plan](../IDR%20Plans/idr-srv-037-feasibility-and-asynchronous-tasking-strategy.md) | [report](idr-srv-037-feasibility-and-asynchronous-tasking-strategy-report.md) | Complete | Accepted | Feasibility is time-bounded advice, not authorization or a guarantee of command success. |
| IDR-SRV-038 | Command Authorization, Safety, and Audit | [plan](../IDR%20Plans/idr-srv-038-command-authorization-safety-and-audit-strategy.md) | [report](idr-srv-038-command-authorization-safety-and-audit-strategy-report.md) | Complete | Accepted | Independent gates and durable pre-effect evidence must precede every dispatch. |
| IDR-SRV-039 | Authentication, Authorization, and Threat Model | [plan](../IDR%20Plans/idr-srv-039-authentication-authorization-and-api-security-threat-model.md) | [report](idr-srv-039-authentication-authorization-and-api-security-threat-model-report.md) | Complete | Accepted | Strict OIDC/OAuth2 resource-server behavior and deny-default authorization form the baseline. |
| IDR-SRV-039A | Zero-Trust Architecture | [plan](../IDR%20Plans/idr-srv-039a-zero-trust-architecture-alignment-and-enforcement-model.md) | [report](idr-srv-039a-zero-trust-architecture-alignment-and-enforcement-model-report.md) | Complete | Accepted | Continuous, resource/action/context-aware evaluation applies at every meaningful effect and disclosure. |
| IDR-SRV-040 | Policy, Releasability, and Cross-Boundary Access | [plan](../IDR%20Plans/idr-srv-040-policy-releasability-and-cross-boundary-access-constraints.md) | [report](idr-srv-040-policy-releasability-and-cross-boundary-access-constraints-report.md) | Complete | Accepted | Source assertions, local policy binding, and contextual disclosure decisions remain separately auditable. |
| IDR-SRV-041 | Audit Logging and Accountability | [plan](../IDR%20Plans/idr-srv-041-audit-logging-and-accountability-strategy.md) | [report](idr-srv-041-audit-logging-and-accountability-strategy-report.md) | Complete | Accepted | An append-oriented authoritative audit plane is distinct from diagnostic logs and system events. |
| IDR-SRV-042 | DDIL-Informed Server Semantics | [plan](../IDR%20Plans/idr-srv-042-ddil-informed-server-semantics.md) | [report](idr-srv-042-ddil-informed-server-semantics-report.md) | Complete | Accepted | DDIL is per-operation context; state, freshness, authority, and service posture stay explicit. |
| IDR-SRV-043 | Synchronization and Conflict Boundary | [plan](../IDR%20Plans/idr-srv-043-server-synchronization-and-conflict-handling-boundary.md) | [report](idr-srv-043-server-synchronization-and-conflict-handling-boundary-report.md) | Complete | Accepted | Application-level receive/classify/apply governs synchronization; unsafe semantic conflicts quarantine. |
| IDR-SRV-044 | Rust Language and Framework | [plan](../IDR%20Plans/idr-srv-044-rust-implementation-language-and-framework-strategy.md) | [report](idr-srv-044-rust-implementation-language-and-framework-strategy-report.md) | Complete | Accepted | Rust/Axum/Tokio/Tower/Serde/SQLx is the candidate stack, subject to implementation-time repinning. |
| IDR-SRV-045 | Service Architecture and Modularization | [plan](../IDR%20Plans/idr-srv-045-service-architecture-and-modularization-strategy.md) | [report](idr-srv-045-service-architecture-and-modularization-strategy-report.md) | Complete | Accepted | A Cargo-workspace modular monolith with inward dependencies preserves one authority core. |
| IDR-SRV-046 | Reference Deployment | [plan](../IDR%20Plans/idr-srv-046-reference-deployment-strategy.md) | [report](idr-srv-046-reference-deployment-strategy-report.md) | Complete | Accepted | One hardened image plus PostgreSQL/PostGIS and explicit profiles is the reference baseline. |
| IDR-SRV-047 | Configuration, Secrets, and Environment | [plan](../IDR%20Plans/idr-srv-047-configuration-secrets-and-environment-strategy.md) | [report](idr-srv-047-configuration-secrets-and-environment-strategy-report.md) | Complete | Accepted | One versioned typed composition-root contract must reject unknown and unsafe combinations. |
| IDR-SRV-048 | Observability and Health | [plan](../IDR%20Plans/idr-srv-048-observability-logs-metrics-and-health-check-strategy.md) | [report](idr-srv-048-observability-logs-metrics-and-health-check-strategy-report.md) | Complete | Accepted | Logs, metrics, traces, health, domain events, audit, and evidence are correlated but non-substitutable. |
| IDR-SRV-049 | Migration, Upgrade, Backup, and Restore | [plan](../IDR%20Plans/idr-srv-049-migration-upgrade-backup-and-restore-strategy.md) | [report](idr-srv-049-migration-upgrade-backup-and-restore-strategy-report.md) | Complete | Accepted | Immutable forward migrations and isolated portable restore proof precede operational continuity claims. |
| IDR-SRV-050 | Conformance Harness | [plan](../IDR%20Plans/idr-srv-050-conformance-harness-strategy.md) | [report](idr-srv-050-conformance-harness-strategy-report.md) | Complete | Accepted | A standalone Rust black-box CLI should execute the accepted ATS with explicit outcomes and evidence. |
| IDR-SRV-051 | Requirement-to-Test Traceability | [plan](../IDR%20Plans/idr-srv-051-requirement-to-test-traceability-strategy.md) | [report](idr-srv-051-requirement-to-test-traceability-strategy-report.md) | Complete | Accepted | A normalized typed graph with stable IDs, validated YAML, and canonical JSON governs traceability. |
| IDR-SRV-052 | Rust TDD and Multi-Layer Tests | [plan](../IDR%20Plans/idr-srv-052-rust-test-driven-architecture-and-multi-layer-test-strategy.md) | [report](idr-srv-052-rust-test-driven-architecture-and-multi-layer-test-strategy-report.md) | Complete | Accepted | Obligation-first TDD spans 21 layers with deterministic effects, real dependencies, and no retry-green. |
| IDR-SRV-053 | Fixtures, Golden Files, and Scenarios | [plan](../IDR%20Plans/idr-srv-053-test-data-fixtures-golden-files-and-scenario-corpus-strategy.md) | [report](idr-srv-053-test-data-fixtures-golden-files-and-scenario-corpus-strategy-report.md) | Complete | Accepted | One governed registry separates sources, scenarios, oracles, and generated evidence. |
| IDR-SRV-054 | Performance, Load, Stress, and Streaming Tests | [plan](../IDR%20Plans/idr-srv-054-performance-load-stress-and-streaming-test-strategy.md) | [report](idr-srv-054-performance-load-stress-and-streaming-test-strategy-report.md) | Complete | Accepted | Correctness-gated, envelope-bound k6/Rust/PG evidence replaces unqualified throughput claims. |
| IDR-SRV-055 | Security and Command-Control Tests | [plan](../IDR%20Plans/idr-srv-055-security-authorization-and-command-control-test-strategy.md) | [report](idr-srv-055-security-authorization-and-command-control-test-strategy-report.md) | Complete | Accepted | Deny-default, twin-world disclosure, canary, gate, ticket, simulator, and reconciliation tests are required. |
| IDR-SRV-056 | External-Client Interoperability Matrix | [plan](../IDR%20Plans/idr-srv-056-interoperability-test-matrix-for-external-csapi-clients.md) | [report](idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md) | Complete | Accepted | Pinned TypeScript and Python clients provide mandatory independent semantic interoperability evidence. |
| IDR-SRV-057 | Final Glaux Server IDR Synthesis | [plan](../IDR%20Plans/idr-srv-057-final-glaux-server-idr-synthesis-report.md) | [this report](final-idr-research-report.md) | Complete | Accepted | The accepted corpus supports a coherent, incremental, evidence-gated server baseline. |

### 3.3 Approved prerequisite exceptions

None.

### 3.4 Coverage summary

- Topics complete: 67 of 67.
- Topic reports accepted: 67 of 67.
- Approved prerequisite exceptions: 0.
- Partial topics: 0.
- Coverage confidence: High for research conclusions and downstream planning readiness; no implementation evidence is implied.

### 3.5 Evidence authority and currency

Evidence was ranked in this order: published normative standards; project-controlling governance and authorized NATO material; accepted topic reports and their pinned primary evidence; official maintenance history; directly reproduced implementation evidence; informative peer implementations and community discussions; reasoned synthesis. Lower-ranked evidence could expose risks or options but could not override higher-ranked obligations.

The official Connected Systems repository was refreshed on September 16, 2026. The Version 1.0.0 tag remains at commit 8e03b236a049849f2ccc24b4fd9fdce5ff69bed2; the default branch remains at 3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f; and the Part 3 working branch remains at 6f529a15bfa63259febc3620378d3e5a06305333. Tracked issues 14, 58, 68, 152, 178, 179, and 191 remain open. No material delta changes an accepted recommendation. Published Parts 1 and 2 and their approved tag remain controlling; default-branch corrections are informative until published; open issues remain proposals; and Part 3 remains experimental.

---

## 4. Research Methodology and Synthesis Approach

The synthesis used six stages aligned to the topic plan:

1. **Inventory and gate.** Reconcile all 67 topic IDs, report paths, final-status markers, acceptance entries, and dependencies. Record any exception rather than silently treating a gap as complete.
2. **Normalize.** Extract each report’s findings, decisions, recommendations, risks, assumptions, dependencies, proof tasks, and handoffs into common thematic areas.
3. **Reconcile.** Compare recommendations at shared seams: normative versus implementation precedent; canonical versus source representation; database authority versus broker/cache projections; full target versus first implementation; public demo versus operational profile; and local versus distributed behavior.
4. **Classify.** Assign recommendations to first implementation, follow-on, or deferred/operational scope. Preserve extension seams where a deferred capability materially affects first-slice contracts, but do not create fake implementations.
5. **Refresh and challenge.** Recheck mutable official standards history, test recommendations against accepted constraints, identify unsupported claims, and keep uncertainty explicit.
6. **Hand off.** Produce the decision matrix, sequence inputs, proof candidates, risks, open decisions, work-package candidates, governance updates, and success-criteria validation in this report.

Conflicts were resolved by authority, dependency, safety, and reversibility. Published requirements outrank peer behavior. The canonical model outranks serialization convenience. Durable database truth outranks delivery infrastructure. Deny-default behavior outranks demonstration convenience. Safe reversible staging outranks premature breadth. When evidence does not justify a single operational choice—such as production topology, numeric objectives, or synchronization protocol—the report carries the decision forward with a gate rather than inventing certainty.

This method deliberately avoids averaging incompatible recommendations. It records a chosen direction and its boundary. The result is an initial design baseline: strong enough to guide implementation artifacts, but still subject to implementation proof and project-owner decisions.

---

## 5. Standards and Obligation Baseline

### 5.1 Authority model

The implementation must maintain an explicit authority classification for every imported requirement and artifact:

| Evidence class | Glaux use |
|---|---|
| Published OGC standards and approved Version 1.0.0 artifacts | Normative basis for implemented Part 1 and Part 2 behavior and conformance claims |
| Project-controlling AEP-4789 ratification draft | Required Glaux contribution context, without a claim that NATO has promulgated it |
| Published inherited standards | Normative within the exact dependency and conformance scope invoked by CSAPI |
| Official post-publication branch history | Maintenance evidence and compatibility input, not a silent replacement for published text |
| Open issues and pull requests | Unresolved proposals and regression targets; never normative merely because they are plausible |
| Part 3 working draft | Experimental design input only, with no OGC Part 3 conformance claim |
| Peer implementations and clients | Interoperability and engineering evidence, not standards authority |

This classification must be represented in the requirements and traceability records so that a test or implementation choice cannot accidentally promote an issue discussion, draft fragment, or peer quirk into a requirement.

### 5.2 Functional obligation baseline

The AEP Volume I mapping establishes seven server responsibility families:

1. discovery and root navigation;
2. registration and authoritative description;
3. access, filtering, and exchange;
4. dynamic data and streaming;
5. tasking, control, and feasibility;
6. status, availability, and system events; and
7. cross-cutting security, trust, federation boundaries, audit, and DDIL behavior.

Volume II selects the technical package through which these responsibilities are expressed: OGC API - Connected Systems Parts 1 and 2, SensorML 3.0, SWE Common 3.0, and inherited web standards. Glaux should implement this as one semantic system, not as loosely connected endpoint families.

### 5.3 Derived implementation rules

- Every externally claimed conformance class must map to its exact published requirements, implementation behavior, tests, evidence, and declared conformance URI.
- SensorML and SWE Common are semantic contracts, not opaque JSON documents.
- HTTP semantics—including methods, conditions, caching, negotiation, status codes, headers, canonical links, and proxy-aware origins—are part of conformance behavior.
- External NATO capabilities remain explicit integration seams. The server must not imply that discovery, policy, identity, cross-domain transfer, or command systems owned elsewhere are embedded.
- Standards ambiguities and artifact conflicts require recorded compatibility dispositions and tests, not silent repair.

---

## 6. Conformance and Requirements Baseline

The full design target is the accepted inventory of 25 direct CSAPI conformance classes, 233 requirements, five recommendations, and 240 abstract-test cases. These counts are a governed baseline, not a statement that a future first build passes all cases. The requirements register must preserve source identifiers, source authority, conformance-class membership, applicability, profile, status, and exact source citation.

Conformance declarations must be live outputs of the enabled capability registry. A deployment declares only the classes that are implemented, enabled, and covered by current evidence for that profile. Disabled, experimental, unavailable, or stubbed behavior must not appear as supported. The conformance endpoint, OpenAPI description, route registry, representation registry, runtime configuration, and evidence manifest must agree.

Requirement and verification state must remain multidimensional. “Applicable,” “implemented,” “tested,” “passed,” “deviated,” and “accepted” are different facts. A skipped test is not a pass; unsupported behavior is not not-applicable; an open standards issue is not permission to ignore published text. Every test execution terminates in one of the accepted explicit outcomes and records applicability, target identity, build/configuration digest, fixture digest, timestamps, evidence, and deviation references.

The first implementation may stage classes in vertical slices. Each release or demo must publish a bounded claim manifest identifying:

- the exact server build, profile, configuration fingerprint, database migration level, and public origin;
- implemented and declared conformance classes;
- applicable and executed requirements and tests;
- evidence freshness and known deviations;
- experimental or extension behavior; and
- explicit non-claims, including certification and operational readiness.

No full-standards claim is appropriate until the complete accepted inventory has implementation and independent evidence closure.

---

## 7. API Surface and Behavior Baseline

### 7.1 Discovery and resources

Clients begin at the landing page and navigate typed links to the API definition, conformance declaration, collections, and resource families. Canonical absolute links must reflect the trusted public origin, including when a validated reverse proxy terminates TLS. Link relations, resource types, media types, titles, and profiles are generated from one route/representation registry.

The full resource surface includes collections; systems and their history or revisions; deployments; procedures; sampling features; properties; datastreams; observations; status; system events; control streams; commands; and feasibility resources. Implementation is staged, but route absence, method rejection, capability-disabled, policy denial, temporary unavailability, and resource absence must remain distinguishable.

### 7.2 Query, selection, and pagination

Each supported parameter has defined syntax, typing, default, combination behavior, temporal and spatial semantics, ordering, cost controls, and error behavior. Unsupported parameters are rejected rather than ignored. Filtering and counting operate on the authorized view so totals, cursors, page shapes, timing, and links do not disclose denied resources.

Cursor tokens are opaque, authenticated, expiring, and bound to the effective query, sort order, representation/profile, principal and policy context, and consistency snapshot or watermark. Stable tie-breakers prevent duplication and omission. Offset paging may be supported only where its weaker consistency and cost are acceptable and documented.

### 7.3 Negotiation and versioning

A central registry maps resource/action combinations to supported request and response representations, profiles, schema versions, encodings, and codecs. It drives routing, validation, OpenAPI, Vary, Accept, Content-Type, 406, and 415 behavior. Canonical resources are not independently authored per representation.

Versioning combines a stable service root with explicit media/profile and contract versions. Backward compatibility is evidence-based; breaking behavioral or schema changes require a new supported version or profile, migration instructions, deprecation metadata, and a bounded overlap period chosen by project governance. Deprecation must be observable in documentation, responses where appropriate, and the compatibility test matrix.

### 7.4 Errors and API description

RFC 9457 Problem Details is the common error envelope. Stable Glaux problem-type URIs identify machine-relevant categories; extensions may include correlation ID, safe validation paths, retry guidance, and policy-safe details. Stack traces, query text, secrets, existence-sensitive details, and unsafe policy reasoning never cross the public boundary.

The service owns an implementation-specific OpenAPI 3.1 description. Curated components capture standards meaning and examples; generated elements capture actual routes, parameters, security schemes, and wire models. CI compares route, behavior, representation, schema, security, and documentation registries. Official bundled OpenAPI artifacts remain requirements evidence but are not deployed unchanged as the Glaux contract.

---

## 8. Resource Model, Identifier, Lifecycle, and Linkage Baseline

### 8.1 Canonical model

Glaux uses one encoding-neutral canonical graph. The model distinguishes:

- addressable API resources;
- aggregate/domain entities and value objects;
- typed relationship facts;
- append-oriented observations, events, commands, audit, and provenance records;
- immutable schemas and representation contracts;
- current, latest, search, and disclosure projections; and
- caches and derived materializations.

These categories have different identity, mutability, temporal, retention, policy, and replication rules. A JSON object, database row, SensorML document, event message, and API resource may represent overlapping information without being the same authority object.

### 8.2 Identity and lifecycle

Local resource IDs use service-wide UUIDv7 candidates. Stable domain UIDs, canonical URLs, external source identifiers, aliases, revision IDs, event IDs, message IDs, command IDs, and persistence keys remain typed and non-interchangeable. External identifiers are scoped by authority and type. Canonical URLs are derived from trusted origin plus stable route identity, never accepted as uncontrolled local identity.

Lifecycle transitions are explicit and conditional. Update creates a new revision or state transition while preserving required evidence. Deletion is policy-checked and normally produces a tombstone sufficient to prevent accidental resurrection and explain replacement, denial, or removal without leaking protected content. Aliases and replacement links preserve navigability; they do not merge identities. Reuse of retired IDs is prohibited.

### 8.3 Relationships, time, and evidence

Relationships are first-class typed facts with source, target, relation type, direction, provenance, validity, confidence, authority, and lifecycle. The model distinguishes authoritative assertions from derivations and inverse navigation. Cardinality, endpoint type, cycle, containment, and policy constraints are enforced centrally.

Time is modeled by meaning: valid/effective time, phenomenon time, result time, transaction/commit time, ingest/receipt time, and processing time. Open intervals, uncertainty, precision, and time zones are preserved. “Current,” “latest,” “live,” “fresh,” “cached,” and “available” are separate computations with request-scoped reference time and policy.

Provenance is a PROV-compatible evidence graph connecting entities, activities, agents, sources, transformations, validations, and decisions. Quality, uncertainty, source trust, identity assurance, transport trust, policy trust, and result confidence stay multidimensional. This evidence supports explanation and policy; it must not be collapsed into a single universal score.

### 8.4 Status and system events

Descriptive status, assessed availability, capability state, source connectivity, service posture, command state, and observation-derived current values remain separate. Availability uses explicit assessed states, including unknown, rather than inventing a binary answer. A latest observation can be old, policy-hidden, uncertain, or unavailable even when the datastream exists.

System events are durable domain facts with causal and temporal evidence. They may be standards-visible where supported and also drive publication. Internal revision or event history may be retained even where the published standard no longer exposes an earlier draft resource class; Glaux must not claim a removed class merely because peer software still implements it.

---

## 9. Representation, Validation, Schema, Semantic Binding, and OpenAPI Baseline

### 9.1 SensorML and SWE Common

SensorML handling has five layers: immutable source bytes and metadata; safely parsed syntax; normalized canonical meaning; deterministic generated representations; and validation evidence. Source preservation is essential for fidelity, signatures, provenance, diagnostics, and future reprocessing. Canonicalization is explicit and reversible where promised; it must not discard constructs simply because the first API view does not expose them.

SWE Common definitions are immutable, versioned data contracts. Component kind, names and paths, ordering, constraints, nil reasons, optionality, units, quality, allowed values, encoding parameters, byte order, separators, framing, and schema digest bind observations and commands. The data model is preserved broadly while codecs are enabled in stages: standards-aligned JSON first, then tested Text and Binary capabilities where required.

### 9.2 Validation pipeline

Validation is a staged, evidence-producing pipeline:

1. transport and size limits;
2. media type, profile, and encoding selection;
3. safe parse and structural schema validation;
4. resource and relationship semantics;
5. identifier, temporal, spatial, unit, and vocabulary constraints;
6. authorization, policy, trust, and source contract;
7. state transition, concurrency, and idempotency checks;
8. transaction commit; and
9. durable publication and evidence recording.

Client writes are strict. Controlled imports may enter quarantine with recorded findings, but invalid or untrusted content never becomes canonical merely because it was retained. The contract registry is versioned, digest-addressed, and usable offline; external references are pinned and fetched through controlled tooling rather than request-time network resolution.

### 9.3 Semantics and units

Semantic bindings record contextual roles such as observed property, command parameter, feature characteristic, result component, and quality field. Vocabulary packages are immutable, versioned, source-attributed, and distributable offline. Mapping is explainable and conservative: exact, narrower, broader, related, and locally asserted mappings are not equivalent.

UCUM is the preferred computational unit system where applicable, while exact source unit identifiers and labels remain available. Conversions require dimensional compatibility, declared transforms, precision/error policy, and evidence. Commands never undergo silent unit conversion; the accepted command contract and safety policy determine whether explicit conversion is permitted.

### 9.4 Generated representations

JSON, GeoJSON, SensorML, SWE encodings, OpenAPI schemas, events, and future client models are generated or validated against the same canonical contracts. Determinism, canonical ordering where required, stable identifiers, exact media types, and round-trip or semantic-equivalence tests prevent representation drift. Recursive schema and reference resolution require an early proof because several official artifacts and toolchains differ in their support.

---

## 10. Persistence, Geospatial, Time-Series, Metadata, Migration, and Continuity Baseline

### 10.1 Authoritative persistence

PostgreSQL with PostGIS is the authoritative storage baseline. A relational-hybrid design uses normalized tables and constraints for identity, resources, relationships, lifecycle, policy bindings, transactions, audit references, and frequently queried semantics. JSONB is appropriate for source-shaped fragments, extensible attributes, or preserved structured content when it does not replace enforceable domain invariants. A broker, cache, search index, object store, or generated document is never the sole authority.

Exact artifacts—source SensorML, schema packages, validation inputs/outputs, large documents, conformance evidence, and exports—are content-addressed and cataloged transactionally. The initial reference may store bounded artifact bytes in PostgreSQL or a controlled filesystem volume according to the accepted size and integrity rules. Object storage becomes an optional adapter after atomicity, backup, and restore behavior is proved.

### 10.2 Spatial and temporal storage

Spatial storage separates exact source geometry/CRS, validated canonical spatial assertions, indexed query projection, generated representation, and derived caches. PostGIS is the full-profile authority. API GeoJSON uses the required CRS84 behavior; transformations are allowlisted and evidence-producing. Antimeridian geometry, poles, empty/invalid shapes, precision, three-dimensional coordinates, vertical reference, and moving-platform extents require dedicated fixtures.

Observation families use append-oriented, contract-bound storage with phenomenon and result times preserved separately. Native PostgreSQL range partitioning is the first baseline. The exact partition key—result time is a candidate—and indexes must be selected by benchmark across ingest, latest, range, spatial-temporal, retention, and restore workloads. TimescaleDB is admitted only if measured benefit exceeds portability, operational, licensing, and migration cost.

### 10.3 Transactions and concurrency

Read Committed plus database constraints, conditional writes, strong ETags/If-Match, uniqueness, and compare-and-swap is the default. Targeted row or advisory locks and Serializable transactions are used only for demonstrated invariant races. Idempotency keys are scoped to authenticated actor, operation, resource/target, normalized payload, and validity window; the same key with different intent is rejected.

Ingestion and command requests provide at-least-once delivery tolerance with effectively-once local effects. Inbox/idempotency records, canonical state, audit linkage, and outbox/domain-event records commit atomically. Retries cannot recreate accepted observations, commands, or dispatch effects.

### 10.4 Lifecycle and continuity

Retention, legal/operational hold, compaction, archive, tombstone, purge, and propagated deletion are policy-driven resource-family workflows. Purge never bypasses referential, audit, command, synchronization, or evidence obligations. Derived caches can be rebuilt; canonical state and exact artifacts require verified protection.

Database migrations are immutable, forward-only SQLx artifacts executed by an explicit same-image administrative command. Expand/backfill/contract patterns permit compatible evolution. Service startup fails closed when schema, build, configuration, or contract compatibility is unsafe.

The first portable continuity proof is a coherent logical database backup plus an exact artifact manifest and bytes, configuration/contract identifiers, build digest, and migration level. Restore occurs into an isolated target and verifies integrity, referential consistency, authorized service behavior, outbox reconciliation, audit lineage, and command non-replay. Physical backups, WAL archiving, point-in-time recovery, infrastructure redundancy, numeric RPO/RTO, and disaster-recovery operations are future operational-profile decisions.

---

## 11. Dynamic Data, Ingestion, Streaming, and Event Baseline

### 11.1 Write and ingestion authority

The server owns a common write pipeline: authenticate source; admit and limit; select contract; parse and normalize; validate structure and semantics; evaluate authorization, policy, releasability, and trust; enforce state, concurrency, and idempotency rules; commit canonical state, evidence, audit, and outbox atomically; then publish. Canonical reads expose committed state, never an adapter’s in-flight interpretation.

Publisher and simulator integrations are external ports. Their contracts identify source, tenant/domain, asserted identifiers, schema and stream contract, trust evidence, ordering and replay properties, idempotency scope, rate limits, and permitted operations. A publisher cannot promote its own trust or policy. The simulator uses synthetic fixtures and a non-network sim scheme so tests cannot accidentally reach a physical target.

### 11.2 Observations and status

DataStream creation binds an immutable schema revision and encoding capability. Observations preserve source message identity, phenomenon time, result time, ingest/commit time, values, nil/quality evidence, source provenance, and validation outcome. Latest-value and status projections are derived with deterministic tie rules and explicit freshness; late or corrected data updates projections only according to recorded semantics.

Status update behavior is separate from observation ingestion and command status. Stream “live” flags are narrow contract properties, not universal assertions that a source, sensor, link, value, or service is currently available.

### 11.3 Event authority and delivery

A durable domain event/outbox log is the publication authority. It records event identity, aggregate/resource identity, kind, schema version, occurrence and commit time, cause/correlation, source revision, policy classification, and payload/reference. Delivery adapters record attempts and cursors but do not redefine the event.

Server-Sent Events is the first transport because it fits HTTP deployment and replay while limiting initial infrastructure. Logical opaque cursors identify an authorized stream position; reconnect and retention-gap behavior is explicit. Snapshot-to-stream handoff prevents blind gaps. Policy is evaluated per event and on reconnect; revocation closes or narrows delivery. Slow consumers face bounded buffers, timeout/disconnect policy, and recoverable replay rather than unbounded memory.

### 11.4 Part 3 experimental profile

The draft Part 3 abstract event/topic concepts should shape event names, CloudEvents-compatible metadata where useful, schema references, security, replay, filtering, and AsyncAPI documentation. An outbound MQTT 5 adapter may be implemented after the outbox/SSE baseline as a disabled-by-default experimental profile. It must pin the draft revision, expose an explicit experimental label, use synthetic topics and fixtures, and make no OGC Part 3 conformance claim.

Inbound MQTT command or write processing is not authorized. The research found incompatible topic and field choices across current examples and implementations and no stable published binding. That boundary prevents a draft transport from bypassing the canonical write and command pipelines.

---

## 12. Command, Control, Feasibility, Safety, and Audit Baseline

ControlStream declares a target-specific command contract; Command is an immutable request plus lifecycle; feasibility is a scoped, time-bounded assessment. None substitutes for another. The server preserves the nine published Part 2 command statuses and validates transitions, actor, cause, timestamps, timeouts, cancellation, and terminal behavior.

A command progresses only through independently evidenced gates:

1. authenticated request and API authorization;
2. command authority for actor, action, and target;
3. data-centric policy and releasability;
4. request/schema/unit/range validation;
5. target identity, adapter, and trust verification;
6. current feasibility;
7. safety rules and interlocks;
8. required human or external approval;
9. final pre-dispatch re-evaluation; and
10. issue of a single-use, short-lived dispatch ticket bound to command, target, adapter, payload digest, decision context, and fencing token.

Durable audit and command state must commit before an adapter can attempt an effect. Adapter results and reconciliation evidence update the lifecycle afterward. Timeouts and lost acknowledgements produce explicit unknown or indeterminate outcomes; they never fabricate physical success or failure. Cancellation is a request with defined races, not a promise that an effect already in motion was undone.

The first implementation supports only a deterministic non-network simulator with synthetic targets. Public-demo profiles disable command creation and dispatch. Physical adapters, operational safety cases, real approvals, field exercises, and accreditation are deferred and require separate authority. This keeps tasking in the architectural goal while preventing research or demonstration code from being mistaken for an operational control system.

---

## 13. Security, Authorization, Policy, Releasability, and Source-Trust Baseline

### 13.1 Identity and request security

The operational baseline is an OIDC/OAuth2 resource server with strict JWT validation: allowlisted algorithms, issuer, audience, time, key-use, key-rotation, token-type, and size checks. mTLS or a composite identity profile can add workload/source assurance. A local-unsafe mode may exist only for loopback development with explicit startup warnings and configuration guards; it is prohibited in public, command, interoperability, DDIL, and operational profiles.

Trusted middleware constructs an immutable SecurityContext containing principal, client/workload, authentication strength, token and certificate identifiers, tenant/domain, roles/attributes, source trust, and correlation. Untrusted headers cannot set these values.

### 13.2 Authorization and policy

Authorization combines coarse RBAC with resource/action/context ABAC or relationship rules. An embedded, product-neutral policy evaluator and policy-administration contract are the initial baseline; application services remain semantic policy-enforcement points. Decisions are ALLOW, DENY, or INDETERMINATE, with indeterminate denied.

Data-centric releasability remains separate from API authorization:

- a SourceAssertion records what a source asserted and its provenance;
- a local PolicyBinding records the server’s governed classification/policy attachment; and
- a contextual DisclosureDecision or DerivedView records what this requester may learn now.

The authorized view is established before search, spatial/temporal filtering, aggregation, counts, pagination, latest selection, link expansion, event subscription, OpenAPI/schema disclosure, diagnostics, and error construction. Property-level suppression must preserve schema truth or return an explicitly different authorized schema/profile; it cannot emit misleading partially valid objects.

### 13.3 Trust, audit, and boundaries

Source registration binds identities, permitted operations, namespaces, contracts, rate limits, and trust evidence. Transport success does not make content authoritative. Trust decisions are contextual, versioned, explainable, and re-evaluated at high-impact effects.

The authoritative audit plane is separate from application logs. Typed append-oriented records capture security and policy decisions, administrative changes, writes, reads where required, exports, subscriptions, command gates, dispatch attempts, configuration changes, lifecycle actions, and synchronization decisions. Hash links and checkpoints may make tampering evident, but the project must not claim immutable or non-repudiable audit without the complete external trust and custody system.

Real classification labels, policies, identities, credentials, operational data, and target details are forbidden in public fixtures and documentation. Cross-domain transfer and release enforcement by guards/CDS remain external integration boundaries.

---

## 14. DDIL, Caching, Synchronization, and Conflict Baseline

DDIL is a set of conditions—disconnected, intermittent, low-bandwidth, high-latency, disrupted trust or time—not one global server flag. Every operation has an explicit service posture. A response or state assessment distinguishes valid, current, fresh, last-known, cached, delayed, tentative, authoritative, unavailable, unknown, and partial as applicable. HTTP cache freshness is not domain freshness.

Offline operation never expands authority. Signed, versioned, anti-rollback bundles may cache schemas, vocabularies, trust anchors, policies, configuration, and fixtures within validity bounds. Expiry, missing time assurance, revocation uncertainty, or incomplete dependencies yield bounded degraded behavior. Queued commands are not grandfathered: they require authentication, authorization, policy, feasibility, safety, and target re-validation before dispatch.

Synchronization is application-level receive, authenticate, classify, validate, authorize, deduplicate, apply, audit, and acknowledge—not database replication. A versioned SyncEnvelopeV1 candidate carries message identity, source node, scope, resource/event identity, operation, causal/temporal metadata, schema and content digests, policy/trust references, and payload or artifact reference.

Transit is at-least-once; local apply is effectively once. Scoped watermarks and gap records express progress without implying total ordering. Exact duplicates, monotonic append records, immutable compatible artifacts, and demonstrably commutative changes may auto-resolve. Authority/policy changes, mutable resource conflicts, command effects, identity collisions, unsafe deletes, and ambiguous causal histories quarantine for governed resolution. Last-write-wins is prohibited for authority, policy, commands, and safety-relevant state.

The first implementation provides local DDIL-aware semantics and extension contracts. Two-node synthetic synchronization follows after local lifecycle, audit, event, and idempotency correctness. Production topology, wire transport, federation agreements, numeric horizons, and operator conflict UI remain deferred.

---

## 15. Rust, Service Architecture, Modularization, and Deployment Baseline

### 15.1 Candidate platform

The research candidate is Rust 1.98.1, edition 2024, with MSRV 1.94; Axum, Tokio, Tower, and Hyper for HTTP/runtime; Serde with separate domain and wire models; SQLx for PostgreSQL; curated OpenAPI structure plus Utoipa-generated implementation elements; and offline JSON Schema validation. These versions are evidence snapshots, not timeless pins. The implementation bootstrap must repin supported versions, security advisories, licenses, MSRV, and feature sets.

First-party unsafe code is prohibited unless a narrowly reviewed exception is approved. Dependencies are minimized, pinned, licensed, audited, and policy-checked. Blocking or CPU-intensive parsing, compression, geometry, cryptography, and codec work must use bounded execution rather than starving async tasks.

### 15.2 Modular monolith

The recommended Cargo workspace begins with:

- glaux-domain — encoding- and infrastructure-independent domain types and invariants;
- glaux-application — use cases, ports, authorization/policy orchestration, and transaction boundaries;
- glaux-standards — standards identifiers, wire mappings, and requirement metadata;
- glaux-validation — contract registry, parsers, codecs, semantic validators, and evidence;
- glaux-persistence-postgres — SQLx repositories, transactions, migrations, outbox, inbox, and projections;
- glaux-api-http — routes, negotiation, problem mapping, security middleware, and OpenAPI;
- glaux-adapters — publishers, simulator, SSE, experimental MQTT, identity/policy, artifacts, and future synchronization adapters;
- glaux-server — composition root and role selection;
- glaux-test-support — deterministic clocks, IDs, fixtures, real-dependency harnesses, and controlled fakes; and
- xtask — repeatable generation, validation, evidence, and development operations.

Dependencies point inward. Domain code imports no web, database, broker, or configuration framework. Application services own semantic workflows; persistence adapters implement atomic transactions; HTTP and background workers translate contracts. One initial server artifact can run API, worker, and administrative roles under explicit profile/configuration while retaining extraction seams. Services split only when measured scaling, fault isolation, security boundary, release cadence, or team ownership justifies the distributed cost.

### 15.3 Deployment baseline

Docker Compose is the reference deployment: one immutable Glaux image and PostgreSQL/PostGIS, with explicit same-image commands for migration, fixtures, checks, export, and restore verification. Identity/policy services, reverse proxy, broker, object storage, observability collector, simulator, publisher, test harness, and second node are profile-gated dependencies.

Eleven accepted profiles cover native development, Compose development, CI/conformance, public demo, command simulation, DDIL single node, two-node synchronization, performance, interoperability, release verification, and operational-reference composition. Each profile has an explicit security, command, persistence, network, fixture, telemetry, and claim contract.

The OCI image is multi-stage, non-root, minimal, read-only-root compatible, health-aware, digest-addressed, and accompanied by SBOM and build provenance. amd64 is the first verified target; ARM or constrained tactical suitability requires measurement. Kubernetes, enterprise orchestrators, HA topology, and vendor services are not required for the first reference.

---

## 16. Configuration, Observability, Backup, Restore, and Operational Reference

### 16.1 Configuration

One versioned typed configuration object is assembled at the composition root, with TOML-first files, controlled environment and CLI overrides, explicit precedence, source provenance, and deny-unknown deserialization. Validation resolves cross-field invariants and refuses unsafe combinations before listeners or workers start. Runtime code receives typed subconfiguration, not arbitrary environment access.

Secrets are references or mounted files, not ordinary configuration values. Redacted effective configuration and a canonical fingerprint support diagnosis without disclosure. Secret/key rotation uses versioned key rings and overlap rules where the protocol permits. Dynamic reload is limited to specifically designed safe fields; security, storage, route, schema, and command changes are restart-first.

### 16.2 Observability and health

Diagnostic logs, access events, metrics, traces, startup/readiness/liveness health, domain events, standards-visible system events, authoritative audit, administrative diagnostics, and conformance evidence are distinct signals. They share correlation IDs but cannot substitute for one another.

The baseline uses typed tracing with structured JSON to stdout, route-template and bounded-cardinality fields, a controlled metrics registry with protected Prometheus/OpenMetrics scrape, W3C trace context, and optional OpenTelemetry export. Sensitive fields and labels are schema-classified and redacted at construction. High-cardinality resource, principal, command, query, and token values are excluded from metric labels.

Startup shows whether initialization completed; liveness only whether the process can make progress; role-aware readiness whether enabled roles can safely serve; dependency health the evidence behind posture; and service posture the user-visible degraded condition. A database or policy outage must not be hidden behind a generic healthy response.

### 16.3 Operational-reference boundary

The operational-reference profile demonstrates that production integrations have explicit ports and safety requirements. It does not choose an enterprise identity provider, policy engine, secret manager, object store, broker, telemetry backend, orchestrator, backup service, or HA topology. Those are deployment-owner decisions validated against the same contracts.

Backup/restore evidence must cover database, exact artifacts, schemas/vocabularies, policy/configuration identifiers, signing/encryption keys as governed, migration/build identity, audit checkpoints, event/outbox state, and synchronization lineage. Restores never replay completed command effects. Recovery gaps and duplicate/publication reconciliation are explicit, audited outcomes.

---

## 17. Verification Baseline

### 17.1 Conformance and traceability

A standalone Rust black-box CLI in the workspace is the primary CSAPI conformance harness. It uses independent wire models and executes the 240 accepted abstract-test cases plus clearly separated supplemental checks against deterministic targets. Six terminal outcomes—pass, fail, not applicable, not run, blocked, and error—prevent misleading green totals.

Traceability is a normalized graph stored as constrained YAML, validated by JSON Schema Draft 2020-12, and canonicalized to JSON for stable digests. Requirement, recommendation, conformance class, route/action, implementation unit, test, fixture, evidence, gate, deviation, and claim nodes use stable IDs and typed edges. CI checks dangling references, duplicate ownership, impossible state combinations, uncovered applicable requirements, stale evidence, and declaration drift.

### 17.2 Test-driven architecture

Obligation-first red-green-refactor begins at externally observable behavior and drives inward through 21 accepted layers: pure value/domain tests; property and model invariants; schema/codec/semantic validation; application use cases; security/policy; adapter contracts; router/listener black box; real PostgreSQL/PostGIS repositories; migrations; fixtures/goldens; generated/property/fuzz cases; async/workers; ingestion; events/SSE; command/simulator; DDIL; synchronization; conformance; performance; security assessment; and external interoperability.

Pinned cargo-nextest is the primary non-doctest runner; doctests run separately. Retries may diagnose but never turn a failing first execution green. Test clocks, IDs, randomness, network, and effects are deterministic. Database, migration, spatial, isolation, listener, and integration claims use real dependencies where fakes would erase the property under test.

### 17.3 Corpus and evidence

One governed corpus registry separates:

- source artifacts, preserved with provenance and license;
- scenario inputs and actors;
- exact or semantic oracles; and
- generated run evidence.

Each item records stable ID, version/digest, origin, validity, sensitivity, scale, applicable contracts, transformations, and lifecycle. Public closure guarantees that a public scenario depends only on public-safe synthetic or licensed material. Golden comparisons are exact only when byte stability is part of the contract; semantic comparisons use small, reviewed normalizers rather than broad ignore lists.

### 17.4 Performance, security, and interoperability

Performance evidence is correctness-gated and environment-bound. Pinned k6 drives open-arrival HTTP workloads; an independent Rust probe measures SSE delivery, replay, ordering, gaps, and slow consumers; Criterion covers dedicated-runner microbenchmarks; and PostgreSQL plan/runtime evidence guards critical queries. Offered, started, completed, correct/goodput, dropped, latency, resource use, backlog, and recovery are recorded together. Public-demo numbers remain provisional until reproduced; they are not production capacity or SLOs.

Security verification is deny-default and trace-driven. It includes a local issuer/JWKS, strict token negative cases, complete route/action inventory, object/property/function decisions, twin-world disclosure comparisons, canary values across every surface, source/ingestion trust, secret/configuration safety, DDIL expiry, subscription revocation, command gates, single-use dispatch tickets, fencing, simulator effects, and audit/effect reconciliation. ZAP, cargo-audit, cargo-deny, secret scanning, and fuzzing are supplemental lanes, not substitutes for semantic tests.

External interoperability requires pinned OS4CSAPI TypeScript and OWSLib Python client families. CSAPI Explorer/browser behavior, bounded generated TypeScript/Python clients, and a QGIS inherited-Features subset add coverage. CS-Go, OSH, pygeoapi, and SECD are advisory comparison targets. Evidence distinguishes transport, structural, semantic, and workflow depth and attributes a failure to server, client, fixture, harness, documentation, standards ambiguity, or environment before compatibility policy is changed.

---

## 18. First-Implementation Scope

The first implementation is a sequence of working vertical slices, not a partially wired model of every eventual feature. It must nonetheless preserve stable extension seams for accepted follow-on capabilities.

### 18.1 Mandatory foundation

- Cargo workspace and inward dependency rules from Section 15.
- Typed configuration and explicit development, CI/conformance, demo, command-simulator, and migration/restore profiles.
- Hardened reproducible image, Compose reference, PostgreSQL/PostGIS, same-image migrations, and fixture tooling.
- Stable IDs, deterministic time/effects, structured problems, correlation, health, and redacted observability.
- Versioned requirement/traceability and corpus registries with CI validation.
- Authentication, deny-default authorization/policy, authoritative audit, and public-safe synthetic fixtures from the first externally reachable route.

### 18.2 Vertical Slice 1 — discovery and Part 1 read core

Implement the landing page, API definition, conformance declaration, collections, and a coherent read-only subset of systems, deployments, procedures, sampling features, and properties over real persistence. Include typed links, canonical origin, GeoJSON where applicable, SensorML JSON, exact query and pagination behavior, negotiation, conditional requests, RFC 9457 problems, authorization-before-query, audit, and independent client tests.

The slice declares only evidenced classes. It must prove that route, conformance, OpenAPI, schema, representation, configuration, and test registries do not drift. It is the first credible public read-only demonstration when deployed through TLS with synthetic data and a published non-claim statement.

### 18.3 Vertical Slice 2 — controlled writes and dynamic observations

Add authenticated source registration and the server-owned write pipeline for the implemented Part 1 resources. Add immutable DataStream/schema contracts and JSON observation ingestion, query, latest projection, and status derivation. Commit canonical state, validation/audit evidence, and outbox atomically. Prove idempotency, optimistic concurrency, late data, contract change, policy filtering, spatial/temporal queries, and real PostgreSQL migrations.

Text and binary SWE codecs may follow within this slice only after their individual round-trip, boundary, and fuzz gates pass. They are not implied by a generic “SWE support” label.

### 18.4 Vertical Slice 3 — durable events and safe tasking

Expose outbox-backed SSE with authorized replay, cursor binding, snapshot handoff, retention gaps, revocation, backpressure, and slow-consumer behavior. Add ControlStream, feasibility, and Command resources with the exact Part 2 lifecycle, but dispatch exclusively to the deterministic non-network simulator. Require all command gates, single-use bound tickets, fencing, pre-effect audit, lifecycle reconciliation, and fault tests.

### 18.5 Explicit first-implementation exclusions

- No physical command target or network adapter.
- No public-demo command endpoint or unsafe authentication bypass.
- No inbound MQTT or Part 3 conformance declaration.
- No claim of complete 25-class conformance until all applicable evidence closes.
- No real operational identities, labels, policies, data, credentials, or targets.
- No production SLO, capacity, RPO/RTO, HA, cross-domain, certification, accreditation, or operational-readiness claim.
- No distributed synchronization before the local authority, lifecycle, audit, event, and idempotency model is proved.
- No mandatory broker, object store, search engine, TimescaleDB, Kubernetes, or enterprise product.

### 18.6 Extension seams without fake behavior

Initial schemas and ports should reserve versioned extension points for additional representations/codecs, MQTT delivery, external policy and identity, object storage, two-node synchronization, physical command adapters, and extracted workers. Disabled capabilities remain absent from conformance and OpenAPI or are explicitly described as disabled experimental operations. Placeholder success responses, no-op adapters, fabricated health, and mock evidence are prohibited.

---

## 19. Implementation Sequencing and Deferred Full Scope

This is a dependency order for downstream planning, not a release schedule.

| Sequence | Capability increment | Entry condition | Exit evidence | Deferred dependency unlocked |
|---|---|---|---|---|
| 0 | Governance and evidence bootstrap | Final synthesis accepted | Requirements/trace/corpus schemas validate; ADR and claim policy established | All implementation work |
| 1 | Workspace and platform foundation | Accepted ADRs and dependency repin | Build, lint, license/advisory gates, image, Compose, config, migrations, real DB smoke | Vertical slices |
| 2 | Canonical model and contract registry | Core IDs and boundaries approved | Domain invariants, source preservation, offline refs, generated-schema proofs | Read/write resources |
| 3 | Part 1 discovery/read slice | Foundation and model green | Independent root-to-resource navigation; auth/policy; real persistence; bounded class claims | Public read-only demo |
| 4 | Writes and ingestion | Read semantics stable | Strict validation, idempotency, concurrency, provenance, audit, outbox atomicity | Dynamic data |
| 5 | DataStreams and observations | Contract registry and ingest green | Multi-clock storage/query/latest, JSON codec, partition benchmark, authorized queries | Streaming and richer codecs |
| 6 | SSE publication | Durable event/outbox proven | Replay, gaps, revocation, backpressure, snapshot handoff, independent probe | Experimental Part 3 |
| 7 | Simulated Part 2 tasking | Policy/audit and simulator contracts green | Nine statuses, feasibility, gates, tickets, fault/reconciliation evidence | Operational tasking design |
| 8 | Full selected representation and query breadth | Core semantic behavior stable | Text/Binary and richer queries pass conformance, fuzz, performance, and clients | Broader class claims |
| 9 | Experimental outbound MQTT 5 | SSE/event authority stable | Pinned draft profile, AsyncAPI, topic/field fixtures, policy/replay tests, no conformance claim | Future Part 3 adoption |
| 10 | DDIL and two-node synchronization | Local lifecycle and event correctness stable | Signed bundle expiry, envelope, duplicates, gaps, quarantine, conflict evidence | Federation planning |
| 11 | Full standards and release hardening | All capability slices evidenced | All applicable 25-class/233-requirement/240-ATS closure plus interop/security/performance evidence | Formal standards evaluation |
| 12 | Operational-reference engineering | Deployment authority and mission context supplied | Product selections, safety case, HA, continuity targets, operations and accreditation evidence | Operational use |

Follow-on scope includes broader Part 1 and Part 2 class coverage, full allowed SensorML/SWE representations, richer spatial/temporal queries, artifact-store adapter, generated clients, external identity/policy integrations, enhanced administrative workflows, experimental outbound Part 3, and two-node synchronization.

Deferred full scope includes enterprise federation governance, operational policy vocabularies, cross-domain transfer, physical command effects, multi-site HA, production broker/object/telemetry/secret products, WAL/PITR operations, numeric mission SLO/RPO/RTO, tactical hardware qualification, formal OGC certification, NATO operational assessment, and accreditation. Deferral is not rejection; each item lacks either stable standards, mission authority, operational environment, or implementation evidence needed now.

---

## 20. Proof-of-Concept Candidates

| PoC ID | Proof candidate | Question answered | Required output / gate | Source topics |
|---|---|---|---|---|
| POC-01 | Root-to-resource standards slice | Can one real stack keep discovery, conformance, OpenAPI, links, persistence, policy, and clients coherent? | Runnable read slice, trace closure, independent clients, drift report | 006–014, 025, 039, 044–056 |
| POC-02 | Recursive OpenAPI/JSON Schema toolchain | Can chosen Rust tooling resolve official and local recursive references offline without semantic loss? | Pinned tool versions, corpus, pass/fail matrix, fallback decision | 014, 021–024, 044, 050–053 |
| POC-03 | SensorML/SWE round trip | Can exact source, canonical meaning, deterministic generation, and codecs coexist? | Source/semantic/generation digests, round-trip and fuzz evidence | 021–024, 031, 034, 052–053 |
| POC-04 | PostGIS edge geometry | Are CRS84 transforms, antimeridian, poles, 3D/vertical, extents, and policy queries correct? | Fixture suite, plans, semantic results, unsafe-case rejection | 026, 040, 052–054 |
| POC-05 | Observation partition benchmark | Which native PostgreSQL partition/index design meets representative ingest and query envelopes? | Reproducible grid and Timescale admission decision | 025, 027, 029, 034, 049, 054 |
| POC-06 | Exact artifact transaction and restore | Can catalog state and bytes remain atomic and recover coherently? | Failure injection, orphan reconciliation, isolated restore evidence | 023, 028–030, 049, 052–053 |
| POC-07 | Outbox-to-SSE lifecycle | Can commit, replay, snapshot handoff, gap, revocation, and backpressure remain correct? | Independent stream probe and fault/recovery evidence | 029, 031, 034–035, 041–043, 054–055 |
| POC-08 | Authorized spatial/temporal query | Can disclosure precede filtering/count/page without leakage or unacceptable cost? | Twin-world canary, explain plan, stable bound cursor | 011, 018, 026, 039–040, 054–055 |
| POC-09 | Simulated command dispatch | Do all gates, ticket binding, fencing, retry, timeout, cancel, and reconciliation rules hold? | Non-network effect ledger and negative/fault test closure | 033, 036–038, 041–043, 055 |
| POC-10 | Signed DDIL bundle | Do expiry, anti-rollback, degraded time/trust, and authority limits fail safely? | Offline profile evidence and recovery sequence | 023–024, 039A, 040, 042, 047, 055 |
| POC-11 | Two-node synchronization | Can envelope dedupe, gaps, watermarks, quarantine, tombstones, and audit work without DB replication? | Synthetic conflict suite and deterministic reconciliation record | 016–020, 029–030, 041–043, 049, 052–055 |
| POC-12 | Migration and portable restore | Can an older supported release expand/backfill/contract and restore with no command replay? | Compatibility manifest, upgrade/rollback boundary, isolated validation | 010A, 029–030, 045–049, 052 |
| POC-13 | External client semantic interop | Do mandatory TS/Python clients navigate and interpret behavior, not merely receive 2xx? | Pinned run evidence with defect attribution and retest | 014A–G, 050–053, 056 |
| POC-14 | Experimental MQTT profile | Can draft topics/fields map to canonical events without becoming authority or enabling inbound effects? | Outbound-only MQTT 5 adapter, AsyncAPI, pinned-draft evidence, explicit non-claim | 014H, 035, 039–043, 046–048, 054–056 |
| POC-15 | Public demo safety and envelope | Can a TLS-fronted synthetic deployment remain policy-safe, command-disabled, observable, and reproducible? | Deployment digest, disclosure tests, provisional performance run, non-claim page | 039–041, 046–049, 053–056 |

PoCs should become executable evidence producers, not disposable branches. Their fixtures, IDs, result schemas, and accepted conclusions belong in the governed corpus and trace graph. A failed proof is a valid outcome when it records which assumption must change.

---

## 21. Risk Register

| Risk ID | Risk | Impact | Likelihood | Mitigation / decision gate | Owner |
|---|---|---|---|---|---|
| R-01 | First implementation expands toward the entire standards end state | High | High | Enforce vertical slices, explicit non-goals, dependency gates, and claim manifest | Project Lead |
| R-02 | Published standards text, schemas, OpenAPI, PURLs, and maintenance history conflict | High | High | Preserve authority class, pin artifacts, record adapters/deviations, maintain regression fixtures | Standards/Conformance Lead |
| R-03 | SensorML/SWE normalization loses source meaning or unsupported constructs | High | Medium | Preserve exact source; full canonical model; staged codecs; semantic round-trip and quarantine | Domain/Validation Lead |
| R-04 | Authorized query/count/page/event behavior leaks protected existence or values | Critical | Medium | Authorized-view-first design, twin-world/canary tests, bounded plans, deny on indeterminate | Security/Policy Lead |
| R-05 | Command retry, race, or uncertain outcome causes duplicate or unsafe effect | Critical | Medium | Simulator-only first; independent gates; single-use bound ticket; fencing; durable pre-effect audit | Command Safety Lead |
| R-06 | Event replay, retention gaps, or slow consumers cause silent loss or resource exhaustion | High | Medium | Durable outbox, logical cursors, explicit gaps, bounded buffers, independent probe | Dynamic Data Lead |
| R-07 | Temporal/spatial query and observation storage fail at representative scale | High | Medium | POCs 04/05/08, plan regression, workload grids, measured partition/index choice | Persistence Lead |
| R-08 | Distributed conflicts corrupt authority, policy, lifecycle, or command state | Critical | Medium | Defer sync; typed envelopes; quarantine unsafe conflicts; prohibit LWW in critical domains | Synchronization Lead |
| R-09 | Public demo convenience weakens authentication, policy, secrets, or command controls | High | Medium | TLS, synthetic-only public profile, command disabled, no local-unsafe, disclosure and config tests | Deployment/Security Leads |
| R-10 | Peer compatibility quirks distort the normative implementation | Medium | Medium | Standards-first canonical core, strict server/tolerant client separation, defect attribution | Interoperability Lead |
| R-11 | Dependency or toolchain changes invalidate the 2026 Rust candidate baseline | Medium | High | Implementation-time repin; MSRV/license/advisory policy; lockfile and provenance | Platform Lead |
| R-12 | Database/artifact backup restores inconsistent state or replays effects | Critical | Low/Medium | Coherent manifest, isolated restore, integrity/reconciliation, command non-replay test | Continuity Lead |
| R-13 | Metrics/logs/errors/OpenAPI expose sensitive fields or high-cardinality data | High | Medium | Typed registries, construction-time redaction, canary tests, protected diagnostics | Observability/Security Leads |
| R-14 | Draft Part 3 changes or divergent implementations create sunk-cost coupling | Medium | High | Stable abstract event core; outbound adapter only; revision pin; disabled experimental profile | Event/Standards Leads |
| R-15 | Research completion is presented as implementation, conformance, or operational readiness | High | Medium | Claim policy, evidence manifest, explicit non-claims, owner review for public statements | Project Lead |
| R-16 | Historical plan-status metadata creates governance ambiguity | Low | Medium | Treat accepted reports and governing ledger as authority; normalize metadata in a controlled maintenance task | Documentation Lead |
| R-17 | External Glaux component repositories remain placeholders or diverge from contracts | Medium | Medium | Publish versioned contracts and entry criteria; test only after independently usable | Architecture Lead |
| R-18 | Controlled NATO content is copied into public artifacts | High | Low | Use permitted summaries and internal references; content review before publication | Project Lead |

No risk is closed by this research report alone. The listed owner is the recommended accountable role for downstream planning, not an assignment to a named individual.

---

## 22. Open Decision Register

| Decision ID | Decision needed | Current recommendation / default | Decision trigger and evidence | Authority |
|---|---|---|---|---|
| OD-01 | Exact first release conformance-class set | Select the smallest coherent Part 1 discovery/read slice; publish exact class list only after trace review | Implementation Guide route/requirement decomposition | Project Lead + Standards Lead |
| OD-02 | Concrete Rust/dependency pins | Re-evaluate the accepted candidate stack at bootstrap | Supported releases, advisories, licenses, MSRV and PoC results | Architecture Lead |
| OD-03 | Observation partition key and TimescaleDB | Native PostgreSQL baseline; admit Timescale only by benchmark | POC-05 representative workloads | Architecture/Persistence Leads |
| OD-04 | Exact artifact byte-store threshold/backend | Start with simplest coherent local/PostgreSQL reference; retain adapter port | POC-06 size, transaction, restore, and deployment evidence | Architecture Lead |
| OD-05 | Supported SensorML/SWE media and codec subset per release | JSON first; enable Text/Binary individually when evidenced | POCs 02/03 and interoperability needs | Standards/Validation Leads |
| OD-06 | Public anonymous-read policy | Permit only a synthetic, explicit public principal/profile with authorized-view tests | Demo plan and POC-15 | Project Lead + Security Lead |
| OD-07 | Experimental Part 3 inclusion in first repository | Scaffold port/contracts; implement outbound MQTT after SSE, not before | Stable outbox/SSE and POC-14 capacity | Project Lead |
| OD-08 | External identity/policy products | Keep product-neutral ports in first implementation | Operational-reference sponsor/environment | Deployment/Security Authority |
| OD-09 | Synchronization transport and topology | Keep application envelope transport-neutral; prove two synthetic nodes later | POC-11 and federation requirements | Project Lead + Federation Authority |
| OD-10 | Numeric performance/SLO and capacity targets | Use provisional evidence envelopes only | Named deployment profile, hardware, dataset, concurrency, mission needs | Deployment Owner |
| OD-11 | Retention, archive, RPO/RTO, and DR targets | No universal numeric values in implementation defaults | Data governance and operational environment | Data/Operations Authority |
| OD-12 | Physical command adapter authorization | Deferred; no physical effects in initial project scope | Safety case, operational owner, threat/risk assessment, accreditation | Project/Operational Authority |
| OD-13 | Executive briefing appendix | Do not duplicate now; derive a briefing from accepted Section 1 | Stakeholder need after acceptance | Project Lead |
| OD-14 | Machine-readable recommendation export | Defer until trace schema and issue workflow are established; this report is curated source | WP-00 trace/issue design | Project Lead |
| OD-15 | Project-level approvals to record | Resolve OD-01, 06, 07, 10–12 before relevant scope; others at their gates | Roadmap and ADR workflow | Project Lead |
| OD-16 | Deferred capability scaffolding depth | Preserve typed ports/contracts and disabled profiles only; no fake implementation | Implementation Guide module/API review | Architecture Lead |
| OD-17 | Formal OGC certification path | Do not commit until complete internal conformance and target program availability are known | Full evidence closure and OGC process review | Project Lead |
| OD-18 | Historical plan metadata normalization | Perform a bounded governance cleanup without rewriting accepted research | Final report acceptance or later documentation sprint | Documentation Owner |

---

## 23. Candidate Work Packages

These packages are issue-grouping inputs. The Roadmap owns milestones, staffing, schedule, and release allocation.

| WP | Candidate package | Primary outputs | Depends on | Exit gate |
|---|---|---|---|---|
| WP-00 | Governance, requirements, and claims | ADR process; imported requirement/trace graph; claim and deviation policy; corpus registry | Final report acceptance | Schemas validate and first release candidate class set is reviewable |
| WP-01 | Rust workspace and quality foundation | Crates, dependency policy, CI, nextest, xtask, deterministic support | WP-00 | Reproducible green build with license/advisory gates |
| WP-02 | Configuration and reference deployment | Typed config, profiles, image, Compose, proxy/origin, health | WP-01 | Unsafe profiles rejected; reference lifecycle deterministic |
| WP-03 | Canonical contracts and validation | Domain model, IDs, relationships, SensorML/SWE, schemas, vocab/units, offline registry | WP-00/01 | POCs 02/03 and invariant suites pass |
| WP-04 | PostgreSQL/PostGIS authority | Migrations, repositories, transaction runner, artifacts, spatial/time-series foundations | WP-01/03 | Real-DB, migration, restore, geometry and partition proofs |
| WP-05 | Part 1 discovery/read slice | Landing, API, conformance, collections, resources, links, queries, negotiation, problems, OpenAPI | WP-02/03/04/09 | POC-01 plus mandatory client run and bounded claim |
| WP-06 | Writes and source ingestion | Source registry, strict pipeline, idempotency, concurrency, provenance, audit/outbox | WP-03/04/09 | Failure/retry/authorization/evidence tests pass |
| WP-07 | Dynamic observations and status | DataStreams, observations, latest, status, schema revisions, temporal/spatial queries | WP-06 | Multi-clock/late-data/contract/performance tests pass |
| WP-08 | Events and SSE | Domain-event/outbox, replay, cursor, gaps, revocation, backpressure | WP-06/07 | POC-07 and independent SSE probe pass |
| WP-09 | Security, policy, trust, and audit | OIDC/JWT, SecurityContext, PE/PA ports, authorized views, source trust, audit plane | WP-01/02/03 | Route/action inventory and twin-world/canary suite pass |
| WP-10 | Simulated Part 2 tasking | ControlStream, feasibility, Command lifecycle, gates, ticket/fencing, simulator | WP-06/08/09 | POC-09 passes; no network target possible |
| WP-11 | DDIL and synchronization | Service posture, signed bundles, SyncEnvelope, two-node receive/apply/quarantine | WP-03/04/08/09/10 | POCs 10/11 pass with critical conflicts quarantined |
| WP-12 | Conformance and evidence tooling | Black-box harness, ATS cases, evidence formats, coverage/claim reports | WP-00/01; iterates with all | All declared classes have fresh closed evidence |
| WP-13 | Performance, security, and continuity verification | k6/probes/PG plans, semantic security tests, backup/restore and release gates | WP-02–12 as applicable | Profile-bound RC evidence and no critical open gate |
| WP-14 | External interoperability | Mandatory clients, browser/Explorer, generated clients, QGIS subset, attribution/retest | WP-05–10 as applicable | Required target matrix passes at stated depth |
| WP-15 | Public demo and release documentation | TLS synthetic deployment, public-safe corpus, non-claim page, operator/developer docs | WP-05/09/12–14 | POC-15 and publication review pass |
| WP-16 | Experimental Part 3 outbound publication | MQTT 5 adapter, AsyncAPI, pinned draft profile, topic/field fixtures | WP-08/09/12–14 | POC-14 passes; feature remains disabled/experimental |

Each implementation issue should carry a stable issue ID, linked recommendation and requirement IDs, owning work package, profile, dependencies, fixtures, tests, acceptance evidence, security/sensitivity classification, and explicit non-goals.

---

## 24. Documentation and Governance Updates

After acceptance, the project should:

1. update the Glaux Server Goal and Definition with the full standards goal, vertical-slice first implementation, authority hierarchy, command safety boundary, Part 3 experimental status, and non-claims;
2. create the Implementation Guide from Sections 5–18, adding exact module APIs, database/schema design, route tables, wire contracts, developer workflows, and acceptance criteria;
3. create the Roadmap from Sections 19, 20, and 23, adding priority, ownership, estimates, milestones, releases, and resourcing;
4. establish the machine-validated requirements/traceability registry and preserve accepted source IDs;
5. create ADRs for the modular monolith, PostgreSQL/PostGIS, canonical/source layers, validation registry, event/outbox/SSE, security/policy model, simulator-only command boundary, configuration, deployment profiles, and verification system;
6. create the governed fixture/corpus registry and public-closure policy before examples proliferate;
7. maintain the upstream-history register and open-issue watch without rewriting accepted findings silently;
8. publish implementation claim and deviation templates that separate implemented, enabled, tested, conformant, certified, demo, operational, and accredited states;
9. normalize stale historical topic-plan status labels in one controlled documentation change after final acceptance; and
10. retain this report’s tables as the curated source until the trace/issue schema can export recommendations without losing decisions, boundaries, or evidence context.

Public-facing documentation must state which profile, build, standards revision, conformance classes, representations, security posture, fixtures, and limitations it describes. “Supports CSAPI” without that qualification is too broad.

---

## 25. Final Recommendation Summary

### 25.1 Cross-topic conflict resolutions

| Conflict | Resolved direction |
|---|---|
| Full 25-class goal versus incremental delivery | Preserve the full end state; claim only completed, enabled, evidenced classes per release/profile. |
| Part 3 adoption versus draft instability | Carry the abstract event model; implement SSE first; keep outbound MQTT 5 experimental and disabled; prohibit inbound effects. |
| Historical system-resource behavior versus published removals | Retain internal revision/event evidence; do not claim a removed public class. |
| Official OpenAPI artifacts versus implementation truth | Use official artifacts as requirements evidence; own a curated/generated Glaux description and drift checks. |
| Native PostgreSQL versus TimescaleDB | Begin native; admit Timescale only through the representative benchmark gate. |
| JSONB/source documents versus normalized relational model | Normalize authoritative semantics; preserve exact artifacts; use JSONB only where shape/flexibility justifies it. |
| Public access versus authentication safety | Local unsafe only on loopback; public demo uses TLS and explicit synthetic public-principal policy, with commands disabled. |
| One binary versus API/worker/admin roles | One immutable artifact with explicit logical roles; split only on measured extraction criteria. |
| Status/current/live versus DDIL posture | Keep resource state, freshness, availability, link condition, and service posture orthogonal. |
| Strict standards versus divergent peers | Keep a strict canonical server and bounded compatibility adapters/clients; peers remain informative. |
| Broker delivery versus durable authority | Database transaction/outbox is authority; brokers and SSE are delivery adapters. |
| Tasking goal versus command safety | Implement full resource/lifecycle semantics with simulator first; defer physical effects to separately authorized operational work. |
| Accepted reports versus stale plan labels | Governing acceptance ledger controls; clean metadata later without reopening findings. |
| NATO ratification draft versus promulgated standard | Treat as project-controlling input and explicitly avoid promulgation claims. |
| Broken or unstable normative hyperlinks | Preserve exact normative IDs and pinned/mirrored trace evidence; do not invent substitute requirements. |
| Future component repositories versus available evidence | Treat contracts as future seams, not proof that those components exist or interoperate. |

### 25.2 Synthesis recommendation matrix

| Recommendation ID | Topic area | Recommendation summary | Classification | Source reports | Dependencies | Risks | Verification | Candidate WP | Decision status | Notes / unresolved |
|---|---|---|---|---|---|---|---|---|---|---|
| REC-001 | Authority | Encode published, project-controlled, maintenance, proposal, draft, and peer evidence classes | First | 001–008, 014A–H, 050–051 | Requirements registry | R-02, R-15 | Trace schema and source audit | WP-00/12 | Adopted baseline | Refresh register continuously |
| REC-002 | Claims | Declare only enabled and evidenced conformance classes per profile/build | First | 006–010A, 014, 050–051 | REC-001 | R-01, R-15 | Declaration/OpenAPI/evidence drift tests | WP-00/05/12 | Adopted baseline | OD-01 selects first set |
| REC-003 | API | Implement root-first navigation from one route/link/capability registry | First | 009–014, 014E–G | Canonical origin and OpenAPI | R-02, R-10 | POC-01 and client runs | WP-05 | Adopted baseline | Exact route slice in Implementation Guide |
| REC-004 | Query | Filter/count/page only within authorized views; bind opaque cursors to context | First | 011, 018, 026, 039–040, 054–055 | Policy and persistence | R-04, R-07 | POC-08, canary and plan tests | WP-05/09/13 | Adopted baseline | Cost limits need workload evidence |
| REC-005 | Representation | Drive negotiation, schemas, codecs, OpenAPI, and errors from governed registries | First | 012–014, 021–024, 044–045 | Contract registry | R-02, R-03 | POCs 02/03 and drift CI | WP-03/05 | Adopted baseline | Codec scope per OD-05 |
| REC-006 | Model | Use one encoding-neutral canonical graph with typed resource categories | First | 004, 015–020 | Domain ADRs | R-03 | Invariant/property/round-trip tests | WP-03 | Adopted baseline | Public extension profiles later |
| REC-007 | Identity | Separate local, UID, URL, external, alias, revision, event, and message identities | First | 015–019, 029, 043 | REC-006 | R-08 | Collision/lifecycle/sync fixtures | WP-03/04 | Adopted baseline | UUID version repin is low-risk |
| REC-008 | Time/status | Preserve all semantic clocks and keep freshness, status, availability, live, and posture distinct | First | 018–020, 027, 034–037, 042 | REC-006 | R-07, R-15 | Multi-clock, late-data, DDIL tests | WP-03/07 | Adopted baseline | Numeric freshness is profile policy |
| REC-009 | Evidence | Preserve exact sources plus provenance, validation, quality, uncertainty, and trust | First | 019, 021–024, 028, 031–034 | Artifact catalog | R-03, R-12 | Provenance graph and restore tests | WP-03/04/06 | Adopted baseline | Exposure remains policy-controlled |
| REC-010 | Persistence | Use PostgreSQL/PostGIS relational-hybrid authority | First | 025–030, 034, 043–049 | Platform and schema | R-07, R-12 | Real-DB, spatial, transaction, restore tests | WP-04 | Adopted baseline | Timescale governed by OD-03 |
| REC-011 | Artifacts | Catalog immutable content-addressed bytes transactionally | First | 021–023, 028–030, 049, 053 | REC-010 | R-12 | POC-06 | WP-04 | Adopted baseline | Backend/threshold OD-04 |
| REC-012 | Writes | Route all mutations through one server-owned evidence-producing pipeline | First | 023–024, 029–034, 038–041 | Security, validation, transactions | R-03, R-04 | Negative/fault/idempotency tests | WP-06 | Adopted baseline | Controlled import may quarantine |
| REC-013 | Events | Use atomic domain-event/outbox authority and SSE as first delivery | First | 020, 029, 031, 034–035, 041–043 | REC-012 | R-06 | POC-07 and Rust stream probe | WP-08 | Adopted baseline | Retention horizon remains profile decision |
| REC-014 | Part 3 | Add outbound MQTT 5 only as disabled pinned experimental profile after SSE | Follow-on | 014H, 035, 039–043, 046–056 | REC-013 | R-14 | POC-14 | WP-16 | Adopted boundary | No inbound or conformance claim |
| REC-015 | Commands | Implement Part 2 lifecycle with independent gates, ticket/fencing, and simulator only | First | 033, 036–038, 041–043, 055 | Security, audit, event core | R-05 | POC-09 and effect ledger | WP-10 | Adopted baseline | Physical effects OD-12 deferred |
| REC-016 | Security | Use strict OIDC/OAuth2 validation and immutable SecurityContext; deny by default | First | 039, 039A, 046–048, 055 | Typed configuration | R-04, R-09 | Local issuer/JWKS negative matrix | WP-09 | Adopted baseline | External product OD-08 |
| REC-017 | Policy | Separate authorization, source assertion, local binding, and contextual disclosure | First | 019, 039–043, 055 | REC-016 and canonical model | R-04 | Twin-world and all-surface canary | WP-09 | Adopted baseline | Cross-domain transfer excluded |
| REC-018 | Audit | Build typed append-oriented authoritative audit distinct from logs/events | First | 038–043, 048–049, 055 | Persistence/security | R-05, R-12, R-13 | Gate/effect and restore reconciliation | WP-09/13 | Adopted baseline | No overclaim of non-repudiation |
| REC-019 | DDIL | Model per-operation posture; use signed expiring anti-rollback bundles; never widen authority | Follow-on seam / first semantics | 018–020, 039A–043, 047, 055 | Policy/config/time | R-08 | POC-10 | WP-11 | Adopted baseline | Full field parameters deferred |
| REC-020 | Sync | Use application envelope and quarantine; no DB replication or critical-state LWW | Follow-on | 016–020, 029–030, 041–043, 049 | Local lifecycle/event correctness | R-08 | POC-11 | WP-11 | Adopted boundary | Transport/topology OD-09 |
| REC-021 | Platform | Bootstrap a repinned Rust/Axum/Tokio/Tower/Serde/SQLx stack | First | 044 | Governance and dependency audit | R-11 | Compile/MSRV/advisory/license gates | WP-01 | Provisional stack | OD-02 at bootstrap |
| REC-022 | Architecture | Use Cargo-workspace modular monolith with inward dependencies and effect ports | First | 045 | REC-021 | R-01, R-17 | Architecture and boundary tests | WP-01/03–10 | Adopted baseline | Extract only by evidence |
| REC-023 | Deployment | Use one hardened image plus PostgreSQL/PostGIS and explicit Compose profiles | First | 046 | REC-021/022 | R-09, R-11 | Reproducible profile lifecycle and POC-15 | WP-02/15 | Adopted baseline | Operational products deferred |
| REC-024 | Configuration | Use versioned typed composition-root configuration and secret references; fail unsafe startup | First | 047 | REC-022/023 | R-09, R-13 | Invalid-profile and redaction tests | WP-02 | Adopted baseline | Dynamic reload narrowly scoped |
| REC-025 | Observability | Keep logs, metrics, traces, health, events, audit, and evidence distinct but correlated | First | 048 | Config and semantic registries | R-13 | Functional instrumentation/canary tests | WP-02/09/13 | Adopted baseline | Backends remain adapters |
| REC-026 | Continuity | Use immutable migrations and prove coherent logical DB/artifact restore in isolation | First | 049 | Persistence/config/artifacts | R-12 | POC-12 and command non-replay | WP-04/13 | Adopted baseline | Physical/PITR future operational |
| REC-027 | Conformance | Implement independent Rust black-box ATS harness with explicit terminal outcomes | First | 008, 050 | Requirements and deterministic target | R-02, R-15 | Harness self-tests and 240-case inventory | WP-12 | Adopted baseline | Official certification separate |
| REC-028 | Trace/TDD | Use stable graph IDs and obligation-first 21-layer tests with no green-by-retry | First | 051–052 | WP-00/01 | R-01, R-15 | CI graph/test policy checks | WP-00/01/12 | Adopted baseline | Tooling generated views non-authoritative |
| REC-029 | Corpus | Govern sources, scenarios, oracles, and evidence separately with public closure | First | 053 | Trace schema and licensing review | R-03, R-09, R-18 | Manifest/digest/license/sensitivity CI | WP-00/03/12 | Adopted baseline | No operational data |
| REC-030 | Performance | Gate performance claims on correctness and record full environment/envelope | Follow-on evidence from first | 027, 035, 043, 046, 049, 054 | Working slices and scale corpus | R-06, R-07, R-15 | k6, Rust SSE, Criterion, PG evidence | WP-13 | Adopted baseline | OD-10 sets operational targets |
| REC-031 | Security tests | Require route inventory, twin worlds, canaries, command faults, and supplemental scanners | First | 038–041, 047–049, 055 | Security/policy implementation | R-04, R-05, R-09, R-13 | SecurityTestResult evidence | WP-09/13 | Adopted baseline | Assessment depth expands later |
| REC-032 | Interoperability | Require pinned OS4CSAPI TS and OWSLib Python semantic workflows; keep peers advisory | First read slice onward | 014A–G, 050–053, 056 | Working APIs and corpus | R-10, R-17 | InteropRun evidence and attribution | WP-14 | Adopted baseline | Target versions repin per run |
| REC-033 | Public demo | Publish only TLS, synthetic, policy-tested, command-disabled profile with non-claims | Follow-on to read slice | 039–041, 046–049, 053–056 | WP-05/09/12–14 | R-09, R-15, R-18 | POC-15 | WP-15 | Adopted boundary | OD-06 public-principal policy |
| REC-034 | Planning | Turn these recommendations into ADRs, Implementation Guide, then dependency-aware Roadmap | Immediate after acceptance | 001–057 | Owner acceptance | R-01, R-15, R-16 | Governance review and link audit | WP-00 | Accepted for downstream planning | No schedule is set here |

### 25.3 Priority summary

**Immediate after acceptance:** REC-001, 002, 021, 022, 028, 029, and 034 establish governance, traceability, workspace, and architecture.

**First credible slice:** REC-003–013, 016–018, 023–027, 031, and 032 deliver secure standards-facing behavior over real persistence.

**Controlled follow-on:** REC-014, 015, 019, 020, 030, and 033 add events, safe tasking, DDIL/sync, performance evidence, experimental Part 3, and a public demo at their gates.

**Operationally deferred:** physical commands, production federation/cross-domain behavior, mission numeric targets, enterprise infrastructure, certification, and accreditation.

---

## 26. Validation Against Success Criteria

| IDR-SRV-057 success criterion | Status | Evidence |
|---|---|---|
| Every preceding indexed report complete, accepted, and inventoried or excepted | Met | Section 3 accounts for all 66 prerequisites; no exceptions. |
| Findings, recommendations, risks, dependencies, and questions normalized | Met | Sections 5–25 provide thematic baselines, sequence, PoCs, risks, decisions, work packages, and recommendations. |
| Cross-topic conflicts reconciled or explicit | Met | Section 25.1 records 16 material reconciliations; Sections 21–22 carry residual uncertainty. |
| Complete design baseline across all required areas | Met | Sections 5–17 cover standards through verification. |
| First, follow-on, and deferred scope distinguished | Met | Sections 18–19 and recommendation classifications. |
| Evidence-backed sequence, PoCs, work packages, and governance inputs without replacing later artifacts | Met | Sections 19, 20, 23, and 24 explicitly reserve guide/roadmap responsibilities. |
| Recommendations traced to topic reports | Met | Section 25.2 and Section 27 thematic index. |
| Official maintenance evidence refreshed and authority classes separated | Met | Section 3.5 records the September 16 refresh and published/history/proposal/draft distinction. |
| Risk and unresolved decision registers documented | Met | Sections 21 and 22. |
| No readiness, accreditation, certification, or operational overclaim | Met | Sections 1, 2.3, 6, 18.5, 19, 21, and 25 state evidence gates and non-claims. |
| Final synthesis complete, reviewable, and decision-usable | Met | All 27 required sections are present, and the synthesis is accepted. |

### 26.1 Overall-plan completion validation

| Overall-plan completion criterion | Validation status | Evidence |
|---|---|---|
| All planned topic research executed under governance | Met | 67/67 reports are complete and accepted. |
| Every prerequisite to the indexed final synthesis accepted or formally excepted | Met | Section 3; 66 accepted and zero exceptions. |
| Topic reports meet evidence, literature, decision-usefulness, and completeness standards | Met at accepted-report level | Individual reports and acceptance entries; synthesis authority method in Sections 3–4. |
| Final report responds to overall objective and accounts for every topic | Met for review | Sections 1–4 and full completion matrix. |
| Implementation-usable priorities, risks, decisions, and handoff provided | Met | Sections 18–25. |
| Final overall report accepted and IDR-SRV-057 closed | Met | This report records Final status, Glaux Project Lead acceptance, and the acceptance date. |

### 26.2 Completion checklist

- [x] Overall plan is linked and scope-matched.
- [x] Every indexed topic is accounted for.
- [x] Topic conclusions are synthesized into cross-topic findings.
- [x] Recommendations are explicit, prioritized, and evidence-backed.
- [x] Downstream-artifact readiness is stated.
- [x] Unresolved issues and risks are documented.
- [x] Overall completion criteria are validated.
- [x] Immediate handoff actions and accountable roles are identified.
- [x] Plan-owner acceptance and acceptance date are recorded.

**Synthesis conclusion:** The research program, IDR-SRV-057, and the overall Glaux Server IDR are complete and accepted. Downstream implementation claims remain subject to the evidence gates in this report.

---

## 27. References and Topic Traceability Index

### 27.1 Primary governance and standards

- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-057 Research Plan](../IDR%20Plans/idr-srv-057-final-glaux-server-idr-synthesis-report.md)
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [OGC Connected Systems Upstream History Register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)
- [OGC API - Connected Systems landing page](https://ogcapi.ogc.org/connectedsystems/)
- [OGC API - Connected Systems Part 1](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems Part 2](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC API - Connected Systems Version 1.0.0 source](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0)
- [Official Connected Systems repository](https://github.com/opengeospatial/ogcapi-connected-systems)
- [Draft Part 3 working branch](https://github.com/opengeospatial/ogcapi-connected-systems/tree/part3-working-draft)
- [OGC API - Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [JSON Schema](https://json-schema.org/)
- [RFC 9110 HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [RFC 9457 Problem Details](https://www.rfc-editor.org/rfc/rfc9457)
- [RFC 7946 GeoJSON](https://www.rfc-editor.org/rfc/rfc7946)
- [CloudEvents](https://cloudevents.io/)

### 27.2 Implementation and interoperability evidence

- [OS4CSAPI organization](https://github.com/OS4CSAPI)
- [OS4CSAPI TypeScript client](https://github.com/OS4CSAPI/ogc-client-CSAPI_2)
- [CSAPI Explorer](https://ogc-csapi-explorer.pages.dev/)
- [OpenSensorHub](https://github.com/opensensorhub)
- [pygeoapi](https://github.com/geopython/pygeoapi)
- [SECD interoperability repository](https://github.com/Sam-Bolling/csapi-server-interop-secd)
- [OWSLib](https://github.com/geopython/OWSLib)
- [QGIS](https://github.com/qgis/QGIS)

### 27.3 Thematic topic-report traceability

The complete row-level links are in Section 3.2. This index shows where each accepted topic contributes to the synthesis:

| Synthesis area | Primary topic reports | Principal report sections |
|---|---|---|
| NATO context, obligations, terminology, and boundaries | 001–005 | 2, 5 |
| CSAPI requirements, conformance, API behavior, and implementation lessons | 006–014H | 3.5, 5–7, 17, 25 |
| Canonical resources, identities, relationships, time, provenance, status, and events | 015–020 | 8, 10–14, 18, 25 |
| SensorML, SWE Common, validation, units, properties, and semantic binding | 021–024 | 9, 17–18, 20, 25 |
| Database, space, time series, documents, transactions, and lifecycle | 025–030 | 10, 14, 16, 18–21, 25 |
| Writes, publishers, simulators, observations, events, and tasking | 031–038 | 11–12, 18–21, 25 |
| Authentication, zero trust, policy, audit, DDIL, and synchronization | 039–043 including 039A | 13–14, 18–22, 25 |
| Rust, modular architecture, deployment, configuration, observability, and continuity | 044–049 | 15–16, 18–23, 25 |
| Conformance, traceability, TDD, corpus, performance, security tests, and interoperability | 050–056 | 17–23, 25–26 |
| Final synthesis | 057 | 1–27 |

### 27.4 Final handoff

Following acceptance:

1. **Project Lead and Architecture Lead:** authorize WP-00 and the Implementation Guide, beginning with decisions OD-01, OD-02, and OD-16.
2. **Standards/Conformance Lead:** instantiate the requirements and traceability registries from the accepted baseline without renumbering source IDs.
3. **Architecture and Security Leads:** draft the first ADR set and the safe public/demo claim policy.
4. **Roadmap Owner:** convert Sections 19, 20, and 23 into prioritized milestones, estimates, staffing, and releases after the guide establishes exact acceptance criteria.

No calendar commitments are assigned by this research report. Owners and dates become authoritative only through the project’s downstream planning process.

---

## Final Overall Report Completion Checklist

- [x] Overall plan is linked and scope-matched.
- [x] Every indexed topic is accounted for in the coverage matrix.
- [x] Topic conclusions are synthesized into cross-topic findings.
- [x] Final recommendations are explicit, prioritized, and evidence-backed.
- [x] Readiness for the Goal and Definition, Implementation Guide, and Roadmap is stated.
- [x] Unresolved cross-topic issues and risks are documented.
- [x] Overall completion criteria are validated.
- [x] Final handoff actions are assigned to accountable roles.
- [x] Plan-owner acceptance and acceptance date are recorded.

**Actual Research Time:** Approximately 62 hours of AI-assisted execution<br>
**Completion Date:** September 16, 2026

---

## Addendum A: Draft CSAPI Part 4 Sampling Features

**Date:** September 17, 2026<br>
**Status:** Prepared for review; source research accepted, project-scope decision pending<br>
**Source:** [IDR-SRV-058 Research Report](idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md), accepted by the Glaux Project Lead on September 17, 2026 through the established `proceed` workflow<br>
**Purpose:** Integrate the supplemental findings into this synthesis before discussing any Goal and Definition or Implementation Guide changes

### A.1 Scope, Evidence and Completion Record

The original 67-topic synthesis above remains the historical report accepted on September 16, 2026. This addendum incorporates one later accepted research topic; it does not rewrite prior findings, reopen completed topics, adopt Part 4, or authorize implementation. Acceptance of the research means it is suitable to inform the project decision, not that its recommended option has been selected.

| Supplemental topic | Plan | Report | Completion and acceptance | Synthesis conclusion |
|---|---|---|---|---|
| IDR-SRV-058: Draft CSAPI Part 4 Sampling Features Study | [Research plan](../IDR%20Plans/idr-srv-058-draft-csapi-part-4-sampling-features-study.md) | [Research report](idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md) | Research complete; accepted September 17, 2026; no prerequisite exception | The draft contains useful specialized sampling descriptions, but its unresolved encoding and behavior contracts do not justify treating the whole draft as a settled implementation target. |

Coverage is **67 original accepted topics plus one accepted supplemental topic**. The original report's reporting period, counts, acceptance date and completion checklist remain unchanged. Preparation of this addendum has not been recorded as plan-owner acceptance of the addendum itself.

This is synthesis of the accepted report, not a new standards or implementation survey. Its evidence remains pinned to:

- Official `part4-working-draft`: `05a3c62d198ee52d0cf81a734b700967b7d864a1`.
- Approved CSAPI Parts 1 and 2, with source tag `v1.0.0`: `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`.
- OSH: `9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08`; CS-Go: `b1fd2e0e9bd69e222d05258d659a842ca24502cb`.

The source report's [evidence base](idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md#3-evidence-base), [references](idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md#11-references) and [artifact checks](idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md#12-appendix-reproducible-artifact-checks) control the detailed claims and limitations. No upstream refresh or runtime interoperability test was performed for this addendum.

### A.2 Consolidated Findings

**The existing approved resource is not missing.** Part 1 already defines generic SamplingFeatures, their relationships, navigation and GeoJSON representation. Part 4 adds specialized descriptions within that resource family: spatial sampling types, specimens, statistical samples, feature parts and parametric shapes. It does not establish a need for another Glaux service or a general 3D GIS engine. Connected Systems Part 4 is distinct from OGC API - Features Part 4 transactions and from Simple Feature Access (SFA/WKT). [IDR-SRV-058 §4.2](idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md#42-q2---what-is-already-required-and-what-part-4-adds)

**The draft's limitations affect implementation, not just publication polish.** The report found unfinished encoding/conformance packaging, unresolved schema references, mismatched type identifiers and field rules, and an orientation-free relative-point rule incompatible with the referenced current Pose alternatives. A successful generic JSON parse or a repair to missing schema paths cannot establish specialized support. [IDR-SRV-058 §§4.1–4.3](idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md#41-q1---a-packaged-review-draft-not-a-finished-contract)

**Existing architecture remains a useful foundation, not proof of capability.** The accepted separation of identity, relationships, source documents, temporal meaning and indexed geometry can accommodate future choices. Storing pose or shape parameters does not establish valid subtype handling, historical geometry selection or spatially correct derived extents. The research provides no reason to change the Rust/PostgreSQL/PostGIS architecture for Part 4. [IDR-SRV-058 §4.4.3](idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md#443-glaux-retain-the-architecture-add-semantics-only-deliberately)

**Peers offer examples, not an authoritative answer to draft conflicts.** OSH has useful legacy parametric classes and demo code, with generic binding limitations; the inspected CS-Go path establishes generic static-feature handling rather than the additional parametric wire contract. Neither was exercised at runtime in this study. Their source behavior must not be promoted into a standards requirement or proof of exact-draft interoperability. [IDR-SRV-058 §4.4](idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md#44-q4---what-existing-implementations-and-glauxs-design-establish)

### A.3 Effect on the Original Synthesis

The new evidence qualifies the following conclusions without replacing their accepted foundations:

| Original section | What remains valid | Qualification supplied by IDR-SRV-058 |
|---|---|---|
| §6: Conformance and requirements | The 25-class Parts 1 and 2 completion target and evidence-backed claims | No approved Part 4 class is added. Any later experimental checks remain separate from approved conformance claims. |
| §8: Resource model, identity and relationships | Generic SamplingFeature identity, parent/sampled/sample-of relationships and temporal distinctions | A specialized description does not imply a new resource family or subsystem. The draft's movement/identity tension is not an instruction to mint a new ID for every position change. |
| §9: Representation and validation | Preserve source meaning; validate structure and semantics separately; resolve dependencies safely | Typed Part 4 support would require explicit decisions on identifiers, fields, pose and inherited mappings. SensorML pose reuse does not create a SensorML SamplingFeature representation. |
| §10.2: Spatial and temporal storage | PostgreSQL/PostGIS, explicit geometry and distinct source/query/derived representations | An anchor point is not an implicit sampling volume. Parametric storage does not imply volumetric filtering, terrain intersection or general geometry derivation. |
| §§8.3 and 11.2: Time and dynamic values | Separate validity, observation and receipt/commit time; avoid invented currentness | If mobile draft behavior is selected, geometry selection at the latest time, an instant or interval end must coexist with validity filtering. Late evidence, missing history and frame resolution need explicit handling. |
| §§9.2 and 13: Validation and access | Bounded parsing, controlled references and authorization of represented information | Any chosen frame traversal or derived geometry must preserve limits and access restrictions; it cannot expose protected platform position or trigger arbitrary external fetches. |
| §17: Verification | Independent expectations, fixture provenance and capability-qualified interoperability evidence | Generic Part 1 tests, exact draft checks and local experimental interpretations are different evidence. Verify semantic round trips and query results, not just JSON acceptance or a displayed point. |

The detailed links to the underlying topic reports and current guide sections are in [IDR-SRV-058 §7.1](idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md#71-implications-and-affected-existing-sections). In particular, IDR-SRV-008's generic/specialized boundary and IDR-SRV-026's treatment of Part 4 as informative remain valid. No accepted topic report requires reopening merely to incorporate this supplement.

### A.4 Recommendation and Remaining Choice

**Recommendation carried forward for discussion:** keep the current design compatible with Part 4 descriptions, but do not add blanket Part 4 implementation to the completion target now. Compatibility here means preserving appropriate extension/source information and keeping model and validation boundaries suitable for later specialization; it does not mean advertising specialized support that has not been implemented and verified.

The report also identifies a concrete alternative: an explicitly bounded experimental implementation of **static sampling points, curves and surfaces**. That option still needs an agreed encoding interpretation and subtype tests. It does not implicitly include solids, mobile snapshots, specimens, other parametric families or derived-volume queries. Broader support remains possible, but its relevant contract conflicts must first be resolved or documented as experimental project choices. [IDR-SRV-058 §§5–7](idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md#5-decision-analysis)

The remaining cross-topic uncertainties are the chosen type/field vocabulary, orientation-free pose representation, latest/as-of behavior with incomplete or delayed data, and the geometry used for implicit-shape filtering. These limit what could be promised for selected Part 4 capabilities; they do **not** prevent continued planning of the approved server or require waiting for final Part 4 publication. No implementation hours or release commitments are inferred from the research.

None of these options is selected by this addendum. The Parts 1 and 2 target and the current Goal's experimental Part 3 intent remain unchanged.

### A.5 Downstream Readiness and Handoff

The evidence is sufficient for the next **discussion**, with no additional broad research cycle required:

| Document | Next consideration | Change made by this addendum |
|---|---|---|
| [Goal and Definition v1.6](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) | Whether §4 should mention Part 4 at all, and whether that would be an implementation commitment or only an explicit boundary | None. Research acceptance does not add experimental Part 4 to the goal. |
| [Implementation Guide v0.1](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md) | Whether to clarify the draft boundary and preservation expectations; if support is chosen, identify exact types, representations, validation/query behavior and tests | None. The affected sections are already identified in the source report. |
| Roadmap | Reflect only scope subsequently agreed through the Goal/Guide discussion | No Part 4 milestone, estimate or new work package created. |

**Next step:** the Glaux Project Lead and Codex discuss these Goal/Guide choices. The next `proceed` begins that discussion; it does not automatically edit either document or authorize implementation. Any agreed edits follow the user's direction.

The original §27.4 handoff is historical. This supplemental iteration does not reactivate its proposed work packages, additional document types or approval processes. The current Goal, Guide and the agreed iteration workflow remain the planning context.

**Addendum validation:** the supplemental topic is accounted for; conclusions trace to its accepted report; affected original conclusions and remaining uncertainties are explicit; downstream readiness and next action are stated; original completion and acceptance records are preserved. Project-scope selection and plan-owner review of this new addendum remain distinct from the already recorded acceptance of IDR-SRV-058.

---

## Addendum B: Enhanced CSAPI Querying and Spatial Observation Retrieval

**Date:** September 17, 2026<br>
**Status:** Prepared for review; source research accepted, project-scope decision pending<br>
**Source:** [IDR-SRV-059 Research Report](idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md), accepted by the Glaux Project Lead on September 17, 2026 through the established `proceed` workflow<br>
**Purpose:** Integrate the querying findings and their Part 4 touchpoints before discussing any Goal and Definition or Implementation Guide changes

### B.1 Scope, Evidence and Completion Record

The spatial use case is useful and technically feasible: select observations using the geometry of their related sampling features, even when the returned observations have no geometry of their own. The accepted research recommends considering a bounded, optional Features Part 3/CQL2 addition. **Acceptance of that research does not adopt the addition.** The original synthesis and Addendum A remain intact; this addendum does not change the approved server scope or authorize implementation.

| Supplemental topic | Plan | Report | Completion and acceptance | Synthesis conclusion |
|---|---|---|---|---|
| IDR-SRV-059: Enhanced CSAPI Querying and Spatial Observation Retrieval Study | [Research plan](../IDR%20Plans/idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study.md) | [Research report](idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md) | Research complete; accepted September 17, 2026; no prerequisite exception | The concrete spatial/value cases justify reconsidering the earlier CQL2 deferral, with explicit relationship, geometry, time and result-value meanings. |

Coverage is now **67 original accepted topics plus two accepted supplemental topics**, 058 and 059. Original counts and acceptance dates remain historical. This new addendum is prepared for review, not recorded as already accepted.

This is synthesis of the report published in commit `a785f97`, not a new standards survey. Its [evidence base](idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md#3-evidence-base) and [source checks](idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md#121-source-and-scope-checks) control source versions and limitations: approved CSAPI Parts 1/2 and tag `8e03b236`, Features Part 3 v1.0 / OGC 19-079r2, CQL2 v1.0.0 / OGC 21-065r2, draft Part 4 `05a3c62d`, CS-Go `b1fd2e0`, OSH `9a43f9ec`, and unmerged OSH PR 323 head `2da298be`. Full immutable identifiers are in the source report. No upstream refresh, server execution or new benchmark was performed for this addendum.

### B.2 Consolidated Findings

**Existing filters are useful, but the intended spatial question matters.** Part 1 sampling-feature geometry filtering followed by Part 2 observation `foi` filtering can answer simple static cases. Recursive `sampleOf` association matching can also return an observation whose larger sampling ancestor intersects the area while its own sampling point is outside. That is a different result from direct sampling-location selection, not necessarily an implementation error. A moving feature introduces a further distinction: geometry at observation phenomenon time is not its current position or its whole trajectory. The four examples also distinguish measured-value thresholds from property identity, units and nil values, and specimen sampling time from observation time. [IDR-SRV-059 §4.1](idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md#41-q1---existing-building-blocks-do-not-answer-every-spatial-question)

**Features Part 3/CQL2 supplies filtering machinery, not the missing CSAPI meanings.** Queryables can describe searchable values absent from the returned representation, including a selected related geometry. Observations therefore need not become GeoJSON features to use the generic mechanism. The server must still define which relationship, geometry version and result component supplies each value; a property name does not standardize traversal or create a general join language. [IDR-SRV-059 §§4.2-4.3](idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md#42-q2---a-reusable-filtering-mechanism-not-an-automatic-csapi-binding)

**The peer evidence supports investigation, not copying an established solution.** At the report's inspected snapshots, CS-Go exposes named filters, direct sampling-feature matching and text searches, not demonstrated CQL2. OSH's spatial observation input targets an internal observation location; the traced predicates/storage paths do not establish consistent sampling-feature geometry evaluation. Its inspected open PR proposes simple result equality and leaves CQL2 for future work. These are bounded source findings, not runtime failures or whole-project conformance judgments. The reported CS-Go development work and anticipated SWG issue were not identified within the recorded public search. [IDR-SRV-059 §4.4](idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md#44-q4---peer-implementations-show-useful-work-not-a-complete-solution)

**Querying and Part 4 remain separate choices.** Generic sampling-feature geometry is already available under Part 1; specialized draft types are not a prerequisite for improved observation filtering. Conversely, adopting Part 4 would not supply a complete querying interface or resolve missing historical geometry, relative frames or implicit volumes. An anchor point must not stand in for a sampling area or volume. Both studies support retaining the existing Rust/PostgreSQL/PostGIS architecture. [IDR-SRV-059 §§4.3, 4.5](idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md#43-q3---define-the-value-being-filtered-before-defining-its-syntax); [Addendum A](#addendum-a-draft-csapi-part-4-sampling-features)

### B.3 Effect on the Original Synthesis

| Original section / research basis | What remains valid | Qualification supplied by IDR-SRV-059 |
|---|---|---|
| §§5-6: Standards and conformance | Full approved Parts 1/2 target; only evidenced capability claims | Features Part 3/CQL2 is an optional addition, not an automatically inherited CSAPI obligation. No new class is adopted here. |
| §7.2: Querying; underlying IDR-SRV-011 | Exact parameter behavior, combined filters, authorization and paging | The earlier CQL2 deferral now merits reconsideration because concrete needs have been demonstrated. Existing `foi` and time behavior must not be redefined by an extension. |
| §§8.3, 10.2 and 11.2: Relationships, geometry and time | Typed relationships; distinct source/query geometry; separate temporal meanings | Direct sampling geometry, ancestor geometry and observation-result geometry are different values. Historical selection needs retained evidence and an explicit rule, not a current-state fallback. |
| §§9.1, 9.3 and 11.2: SWE results and units | Contract-bound, encoding-independent measurement meaning | Begin result filtering with typed scalar queryables per datastream. Text search and universal cross-stream numeric comparisons do not preserve type, unit or nil semantics. |
| §§7.2 and 13: Access, disclosure and paging | The authorized view precedes filtering/counting/paging | Visibility of an observation does not authorize testing hidden related attributes. Replacing a protected value with NULL can itself leak information; permitted query views and consistent policy handling remain essential. |
| §17: Verification | Independent expected results and capability-qualified evidence | Add relationship/time/value cases, class/dependency coverage, rejection/null tests, disclosure tests and paging checks. Parsing success alone does not prove the correct observation set. |
| Addendum A: Draft sampling features | Specialized-type, pose, history and implicit-shape limitations | Improved filtering can be considered without Part 4 adoption. Related specimen examples remain conditional, not an implicit commitment to specialized types. |

The affected prior reports and guide sections are identified in [IDR-SRV-059 §4.5](idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md#45-q5---reconsider-the-deferral-preserve-the-existing-architecture) and [§7](idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md#7-implementation-implications-and-estimates). The original reports are not rewritten. The filtering recommendation is new decision material, while their architecture, semantic and security foundations remain useful.

### B.4 Recommendation and Remaining Choices

**Carry forward the bounded optional addition for discussion.** The report prefers it over indefinite deferral or custom parameters for each new condition, while retaining existing CSAPI workflows. Its initial six-class candidate is:

- Features Part 3: Queryables and Filter.
- CQL2: Basic CQL2, Basic Spatial Functions, Basic Spatial Functions Plus and JSON encoding.

The [exact class identifiers and dependencies](idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md#42-q2---a-reusable-filtering-mechanism-not-an-automatic-csapi-binding) remain controlling for this proposal. Selecting a class means its complete applicable obligations, not a convenient subset: the spatial-plus class is not polygon-only support, and horizontal intersection is not a volumetric claim. Initial JSON-only support is a recommendation with a client-compatibility tradeoff, not an OGC requirement.

The proposed observation queryables begin with common metadata and **direct associated sampling-feature explicit geometry applicable at phenomenon time, when that can be established**; individual datastreams can expose selected typed scalar results. Related attributes require defined association, multiplicity, time and authorization rules. Unknown history is not reconstructed or replaced with current geometry. Arrays, unrestricted relationship traversal, automatic remote dereferencing, parametric-volume derivation and a general analytics service are not included. [IDR-SRV-059 §§4.2-4.3](idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md#421-proposed-endpoint-and-discovery-boundary)

If adopted, the addition needs discovery and parameter definitions, bounded typed parsing, safe query translation, explicit value mappings and independent tests within the existing design. Expression, geometry and traversal costs must be limited; a single request does not guarantee faster execution or stable later pages. Static explicit geometry and scalar results are narrower work than historical moving geometry. No runtime performance, calendar estimate or delivery commitment follows from this research. [IDR-SRV-059 §7](idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md#7-implementation-implications-and-estimates)

The remaining choices are whether to adopt the addition, its public queryable names and history boundary, and whether first clients need Text as well as JSON. Part 4 scope remains independently undecided. New SWG or implementation evidence can refine those choices when available; it is not a reason to invent missing behavior or start another broad research cycle.

### B.5 Downstream Readiness and Handoff

The two supplements now provide enough evidence for the planned combined **discussion**:

| Document | Next consideration | Change made in this iteration |
|---|---|---|
| [Goal and Definition v1.6](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) | Whether agreed optional filtering or Part 4 scope warrants a short clarification of intent and boundaries | None; the Rust CSAPI reference-implementation goal is unchanged. |
| [Implementation Guide v0.1](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md) | Whether to replace the CQL2 exclusion with the bounded addition; separately decide Part 4 support, then specify only the chosen mappings, validation and tests | None; research recommendations are not silently promoted to requirements. |
| Roadmap | Sequence only capabilities subsequently agreed through the Goal/Guide discussion | No milestones, estimates or new work packages created. |

**Next step:** the Glaux Project Lead and Codex discuss the querying recommendation alongside Addendum A's Part 4 options. The next `proceed` starts that discussion; it does not automatically edit the Goal/Guide or authorize implementation. The established Governance, Goal and Definition, Implementation Guide and Roadmap workflow remains in place; no additional document type or approval process is introduced.

**Addendum validation:** IDR-SRV-059 is accounted for as accepted research; conclusions and qualifications link to its report; both supplemental scope decisions remain explicit; original completion metadata, the 67-topic body and Addendum A are preserved. Publication of this addendum is distinct from its later review and from adopting an implementation option.

---

## Addendum C: CSAPI Part 5 Protobuf-First Implementation

**Date:** September 18, 2026<br>
**Status:** Prepared for review; source research accepted, Part 5 project-scope decision pending<br>
**Source:** [IDR-SRV-060 Research Report](idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md), accepted by the Glaux Project Lead on September 18, 2026 through the established `proceed` workflow<br>
**Purpose:** Integrate the Protobuf-first study before discussing any Goal and Definition or Implementation Guide implications

### C.1 Scope, Evidence and Completion Record

**The research supports keeping the existing design ready for Protobuf, while deferring selection of a Part 5 implementation contract.** A specifically bounded OSH-compatible experiment remains an alternative for discussion. Acceptance of the research does not select either option or authorize implementation.

| Supplemental topic | Plan | Report | Completion and acceptance | Synthesis conclusion |
|---|---|---|---|---|
| IDR-SRV-060: CSAPI Part 5 Protobuf-First Implementation Study | [Research plan](../IDR%20Plans/idr-srv-060-csapi-part-5-protobuf-first-implementation-study.md) | [Research report](idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md) | Research complete; accepted September 18, 2026; no prerequisite exception | Substantive Protobuf implementation work exists, but the inspected official material does not yet provide a complete shared Part 5 contract to implement exactly. |

Coverage is now **67 original accepted topics plus three accepted supplemental topics**, 058, 059 and 060. The original report's metadata, counts and acceptance remain unchanged; Addenda A/B retain their historical text. This addendum is prepared for review, not recorded as already accepted.

The scope choices described as pending in Addenda A/B were subsequently addressed in [approved Goal v1.7](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) and [draft Guide v0.2](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md). Selected static Part 4 points, curves and surfaces and bounded enhanced observation filtering are already completion deliverables, alongside the existing experimental Part 3 intent. This addendum does not reopen those choices or reduce the full Parts 1/2 and applicable SensorML/SWE target.

This is synthesis of the report published in `d5ef3b8`, not new external research. Its [evidence base](idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md#3-evidence-base) and [references](idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md#11-references) control the detailed source status and limitations. Principal pins are:

- Official CSAPI master: `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`; published-source tag: `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`.
- OSH add-on PR 224 proposal: `22d98d16c77e1e224365fd458dfc3ba77205f78e`; supporting core PR 354: `6988ded63679fc55b8741f92408a4d883e245d1e`.
- CS-Go: `b1fd2e0e9bd69e222d05258d659a842ca24502cb`.
- Selected Part 3 working baseline: `6f529a15bfa63259febc3620378d3e5a06305333`; separate representation-selection proposal: `6f5987673dd11a8e27a2b6093e2dc5cbd0d16612`.

The source investigation occurred September 17 local time and the report was completed September 18. No upstream refresh, new benchmark or runtime interoperability test was performed for this addendum. The project lead's September 17 meeting account—Protobuf favored and possibly the sole first-release encoding—remains attributed planning context, not an independently confirmed OGC publication decision or new NATO obligation.

### C.2 Consolidated Findings

**There is meaningful work to assess, but not a settled official binding.** The official repository's Protobuf examples and schema wrappers leave substantive decisions unresolved and contain concrete defects. The OSH proposal is more developed: observation/command bindings, generated descriptors, schema discovery and length-delimited messages. Its author describes it as the basis of an in-progress Part 5 draft. At inspection the add-on was an open draft PR and its supporting core PR was unmerged. Thus neither “no Part 5 work exists” nor “the OGC contract is ready to copy” is supported. [IDR-SRV-060 §§4.1–4.2](idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md#4-findings-by-research-question)

**Protobuf would supplement, not replace, required SWE Binary.** Its potential value is another serialization and client-tooling path. Schema discovery, root message and stream-revision identity, media types, request/response selection and framing still need a shared contract. Official examples and OSH do not expose interchangeable interfaces; their media/schema choices differ, and the peer command-schema query uses `commandFormat` rather than the published `cmdFormat`. A schema artifact or repaired `.proto` alone cannot establish interoperable data exchange. No size or CPU advantage was measured. [IDR-SRV-060 §4.2](idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md#4-findings-by-research-question)

**The logical SWE contract remains essential.** A descriptor does not automatically preserve identity, distinct times, units, constraints, nil versus absence/default, fixed cardinality or historical meaning. The inspected OSH code has explicit reconstruction limitations and unfinished outbound observation ID encoding. Protobuf field-number compatibility alone cannot prove semantic compatibility after a schema change. The existing normalized model, immutable stream revisions and bounded validation remain useful; the study does not justify another data model, database or schema service. [IDR-SRV-060 §§4.3–4.5](idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md#4-findings-by-research-question)

**Tooling feasibility is not demonstrated interoperability.** Rust generated-message and dynamic-descriptor options exist, with different native-build, reflection and unknown-field behavior; no dependency was selected. The inspected CS-Go paths advertise Protobuf schemas and validate JSON results, not demonstrated binary exchange. OSH's descriptor-driven Java test is useful but does not establish independent cross-language HTTP or Part 3 interoperability. The study executed small syntax/static/numerical checks, not a compiler, peer service or Rust codec. [IDR-SRV-060 §4.4 and Appendix A](idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md#12-appendices)

### C.3 Effect on the Original Synthesis

The following qualifications carry forward the report's reconciliation of IDR-SRV-007/012/014G/022/023/035/059. They do not rewrite those reports or replace the current approved project scope.

| Original section / related supplement | What remains valid | Qualification supplied by IDR-SRV-060 |
|---|---|---|
| §§5–6: Standards and conformance | Full approved Parts 1/2 target and evidence-backed claims | No approved Part 5 class or complete binding is established by the inspected examples/proposals. Any later peer-compatible experiment needs a distinct claim. |
| §7.3: Negotiation and versioning | Resource meaning is independent of representation; schema and media selection are explicit | Do not silently substitute peer media types, descriptor wrappers, query names or framing for the approved interface. Encoding selection and the representation of its schema remain separate. |
| §§8–9: Resources, time, SWE and validation | One logical model, immutable stream contracts, semantic validation and controlled references | Bind descriptor/imports, root message and mapping to the correct revision. Explicitly preserve or reject unsupported value distinctions; a successful parse is insufficient. |
| §9.4: Generated representations | Deterministic generation where applicable; semantic-equivalence tests | Deterministic Protobuf serialization is not canonical byte identity. Preserve original-byte evidence separately where needed; do not infer equal meaning from a re-encoded byte hash. |
| §11.4: Experimental Part 3 | Version-pinned, explicitly experimental publication; no automatic inbound write/command authority | Protobuf is a possible native data representation, not a replacement for lifecycle-event envelopes. The separate #190 representation-selection proposal does not change the selected Part 3 pin or require every encoding to be published. |
| §§12–13: Command safety and access | Existing validation, authorization and command lifecycle apply regardless of encoding | Bound frames/descriptors/imports and decoding work. Uploaded schema content must not trigger arbitrary retrieval or execution; unknown-field preservation is not proof of safe disclosure. |
| §§15 and 17: Rust and verification | Bounded codec work, independent expected values and capability-qualified tests | No Rust dependency or performance result is selected here. A claimed experiment needs actual cross-implementation bytes and endpoint/publication tests, not only local round trips. |
| Addenda A/B and selected filtering | Explicit sampling semantics and encoding-independent logical query values | Protobuf adds no spatial joins, geometry/history rules or filtering language. Encoding must not change observation membership, counts or paging. |

Detailed evidence and prospective Guide touchpoints are in [IDR-SRV-060 §4.5](idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md#4-findings-by-research-question) and [§7](idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md#7-implementation-implications-and-estimates). Required SWE JSON, Text and Binary remain completion obligations, even when implementation is sequenced. This supplement does not reactivate the original synthesis's proposed additional document systems or work packages.

### C.4 Recommendation and Remaining Choice

**Carry forward the recommendation to preserve existing compatibility points and defer selection of a Part 5 wire contract.** This means keeping the current logical-model/schema/codec separation suitable for a later adapter, not building a placeholder or advertising unimplemented support. A sufficiently complete, versioned proposed binding could justify a later experimental implementation; waiting for final publication is not inherently necessary. The present concern is completeness and interoperable meaning, not draft status alone.

The alternative remains an **explicitly named, pinned OSH-compatible Protobuf experiment**. If selected later, its supported resource/operation/structure subset, media and selector differences, schema identity, framing and rejection behavior would need to be documented and independently tested. It would not establish general Part 5 conformance or authorize inbound Pub/Sub, gRPC, other proposed encodings, a new service or a second persistence model. [IDR-SRV-060 §§5–7](idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md#5-decision-analysis)

The unresolved implementation inputs are the shared versioned binding; resource and direction coverage; semantic preservation and historical revision rules; publication representation selection; and independent interoperability fixtures. They are decision inputs, not an indefinite monitoring assignment or a blocker to continued server planning. No delivery dates or implementation hours are inferred. Neither the recommended path nor the alternative is selected by this addendum.

### C.5 Downstream Readiness and Handoff

The accepted study provides enough evidence for the agreed **Goal/Guide discussion**, without another broad research cycle:

| Document | Next consideration | Change made in this iteration |
|---|---|---|
| [Goal and Definition v1.7](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) | Retain the current scope, or deliberately add a bounded Protobuf experiment; accepting research alone does not require a Goal change | None; no Part 5 deliverable added and no existing deliverable removed. |
| [Implementation Guide v0.2](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md) | If desired, clarify compatibility boundaries in §§4.3/6.2/6.5; only if an experiment is chosen, define its exact mapping, publication and verification work | None; no dependency, wire format or implementation requirement selected. |
| Roadmap | Sequence only subsequently agreed scope | No milestone, estimate or work package created. |

**Next step:** the Glaux Project Lead and Codex discuss whether any Goal/Guide change is warranted. The next `proceed` starts that discussion; it does not automatically edit either document or authorize implementation. After that discussion and any agreed update, resume Guide drafting pass 2. A later draft reference would be helpful evidence, not a prerequisite the user must supply now.

**Addendum validation:** IDR-SRV-060 is accounted for as accepted research; findings and qualifications trace to its report; existing approved scope and original completion records are preserved; remaining choices and the next action are explicit. Publication of Addendum C is distinct from its review and from any implementation-scope decision.

---

## Addendum D: Provenance and Related Metadata Interoperability

**Date:** September 18, 2026<br>
**Status:** Prepared for review; source research accepted, Goal/Guide discussion pending<br>
**Source:** [IDR-SRV-061 Research Report](idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md), accepted by the Glaux Project Lead on September 18, 2026 through the established `proceed` workflow<br>
**Purpose:** Integrate the provenance and related metadata findings before discussing their implications alongside the pending Part 5 choices

### D.1 Scope, Evidence and Completion Record

**The research supports using the existing standards first, with focused clarification of provenance handling and verification in the Implementation Guide.** It identifies a narrower unresolved question about interoperable exchange of exact production inputs, executions, revisions and responsible roles. It does not recommend replacing the observation model or making a new provenance platform a prerequisite for Glaux.

| Supplemental topic | Plan | Report | Completion and acceptance | Synthesis conclusion |
|---|---|---|---|---|
| IDR-SRV-061: Provenance and Related Metadata Interoperability Study | [Research plan](../IDR%20Plans/idr-srv-061-provenance-and-related-metadata-interoperability-study.md) | [Research report](idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md) | Research complete; accepted September 18, 2026; no prerequisite exception | Existing CSAPI/SensorML/SWE capabilities cover substantial context; exact-input exchange, quality interpretation and protected disclosure need explicit treatment without assuming another schema or security regime. |

Coverage is now **67 original accepted topics plus four accepted supplemental topics**, 058–061. Original metadata, counts and acceptance remain historical; Addenda A–C retain their text. Acceptance of IDR-SRV-061 makes its findings available for downstream planning, not automatic implementation adoption or acceptance of this newly prepared addendum.

The [approved Goal v1.7](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) and [draft Guide v0.2](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md) remain the planning context: the full Parts 1/2 and applicable SensorML/SWE target, experimental Part 3, selected static Part 4 types and bounded enhanced observation filtering are unchanged. Part 5 remains a separate pending choice. Preliminary stakeholder concerns remain generalized use cases, not an adopted requirements catalog.

This is synthesis of the report published in commit `3283efd24a0005c13d1b3ff193546bee858cfd9f`, not a new external survey. Its [evidence base](idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md#3-evidence-base) and [references](idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md#11-references) control the detailed claims and limits. Source checks were dated September 18, 2026: published CSAPI source `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`, master `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`, CS-Go `b1fd2e0e9bd69e222d05258d659a842ca24502cb`, and OSH `9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08`. The report distinguishes CSAPI's dated SOSA/SSN Recommendation from the newer Working Draft, informative alignments from requirements, and security overviews/historical binding examples from deployment authority.

No standards-history refresh, new schema experiment, peer-service execution or benchmark was performed for this addendum. The report's recorded probe is evidence; its proposed Glaux tests remain future work. Inaccessible IC release packages and unselected deployment policies remain explicit limitations, not invented obligations.

### D.2 Consolidated Findings

**There is substantial existing support, not a missing observation foundation.** CSAPI associations, SensorML process/context descriptions and SWE value-quality concepts provide useful provenance building blocks. OMS supplies abstract concepts; SOSA/SSN already has an informative PROV alignment. That semantic relationship does not prescribe another JSON model, RDF service or database. OGC demonstrations and sponsored research establish practical interest and useful lessons, not new CSAPI obligations. [IDR-SRV-061 §4.1](idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md#41-q1--existing-support-and-standards-roles)

**A process description is not the same as evidence of a particular production run.** Current system/procedure links do not by themselves identify exact antecedent results, the method/configuration revision used, or accountable roles. Preserve supplied evidence and its limits; distinguish production from server receipt/correction history, and asserted responsibility from authenticated submitter identity. Sampling relationships can have derivation meaning, but alone do not enumerate processing inputs. [IDR-SRV-061 §4.2](idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md#42-q2--origin-derivation-and-responsibility)

**Quality needs both meaning and a supported representation.** Identity/class assessments, numerical measurement uncertainty, statistical likelihood, confidence/coverage and analytic confidence cannot all be normalized into an unlabeled percentage. Unknown uncertainty is not zero, and shared inputs can defeat assumptions of independent evidence. The report reproduced the existing SWE JSON Quantity validation gap: an undeclared arbitrary `quality` string passed the inspected schema, while the invalid-unit control failed. This is permissiveness, not proof of valid quality support; it corroborates IDR-SRV-022/023's separate semantic-validation treatment. [IDR-SRV-061 §4.3](idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md#43-q3--quality-and-uncertainty), [Appendix A](idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md#appendix-a--executed-schema-inspection-and-probe)

**Shared packaging does not establish shared lineage.** Datastream context, compound results, observation collections and transport batches have different roles. Use shared references only for facts genuinely common to their members, retaining per-item identity and exceptions. References reduce repetition but introduce availability, version, authorization and retention dependencies. No new report resource, mandatory UUID scheme, recursive lineage query or unlimited raw-data retention follows. [IDR-SRV-061 §4.4](idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md#44-q4--grouping-and-scalable-exchange)

**Provenance can be more sensitive than the result.** A visible output does not automatically authorize disclosure of its inputs, method or responsible people. Policy applicability, label syntax, binding and access enforcement are separate. Inspected IC/NATO sources establish neither a universal per-JSON-field marking obligation nor permission to remove/coarsen required markings. Keep the Guide's complete-resource, valid permitted projection, or deny/conceal boundary; do not imply full history when only a permitted subset is disclosed. [IDR-SRV-061 §4.5](idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md#45-q5--protection-and-disclosure-boundaries)

**Implementation names and internal fields are not interoperability proof.** The report's pinned CS-Go and OSH paths show useful associations and descriptive/quality support alongside concrete binding limitations. These are static, path-specific findings, not whole-product judgments. Semantic preservation needs independent expected results across reads, writes, conversion and publication; a successful schema parse or same-implementation round trip alone cannot establish it. [IDR-SRV-061 §§3.2, 4.2–4.5 and 7](idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md#7-implementation-implications-and-estimates)

### D.3 Effect on the Original Synthesis

These qualifications reconcile the accepted supplement with earlier research without rewriting the historical report or reinstating every earlier proposed mechanism. The current Goal controls scope; the Guide remains a draft.

| Original section / related material | What remains useful | Qualification supplied by IDR-SRV-061 |
|---|---|---|
| §3.2 IDR-SRV-019 row, §8.3 and §25.2 REC-009 | Preserve origin, transformations, quality and distinct trust dimensions | The historical “PROV-compatible evidence graph” wording is not a mandatory graph database, public PROV service or adoption of every earlier graph/assertion-envelope proposal. Guide §4.10's practical records remain the starting point. |
| §§8.2–8.3 and 11.2: Identity, time and observation evidence | Stable identities and distinct temporal meanings | Current references do not automatically identify exact historical inputs/method revisions. Separate descriptions, actual production and server mutations; preserve what is supplied without inventing missing facts. |
| §§9.1–9.4: Representations, validation and semantics | Source preservation, typed meaning and explicit interpretation of artifact gaps | Conceptual quality capability and permissive JSON validation are not a complete exchange contract. Test the supported quality form and preserve subject, scale/units, method and uncertainty interpretation. |
| §§10 and 11.2–11.4: Storage, dynamic data and publication | Shared logical meaning, revision handling and durable publication | Shared references must retain per-item exceptions and honest availability limits. HTTP and selected Part 3 publication must not silently lose supported provenance or quality. No general graph store or indefinite retention requirement is added. |
| §13: Policy, trust and accountability | Protect direct/indirect disclosure and separate assertions from verified identity | Property-level protection is not a blanket field-labeling requirement. Applicable policy/profile determines granularity and binding; access to a result does not authorize its lineage. Source verification does not automatically verify transformed output. |
| §17: Verification | Independent expected values, negative tests and bounded capability claims | Add semantic, revision, mixed-batch and paired-access cases when implementing the supported contract. Peer fixtures and the research schema probe do not demonstrate Glaux interoperability or security. |
| Addenda A–C and selected filtering | Existing sampling, querying and encoding boundaries | Sampling is not an exact-input inventory; provenance does not expand CQL2 scope. A later Part 5 choice must preserve supported metadata meaning, but this addendum neither selects Protobuf nor changes prior selected experiments. |

The report's [§4.6 reconciliation](idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md#46-q6--gap-classification-reconciliation-and-verification) links the relevant IDR-SRV-019/021/022/023/024/034/040/041 findings and supplements. No original topic needs reopening simply to record these qualifications. No new conformance class, Rust dependency, API endpoint or work package is selected.

### D.4 Recommendation and Remaining Choices

**Carry forward the recommendation to retain the existing server goal and discuss focused Guide clarifications.** The Goal already includes provenance context. Useful clarification is about what information is preserved, where it is available, and how its meaning and disclosure are tested—not declaring a generic “provenance platform.” [IDR-SRV-061 §§5–7](idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md#5-decision-analysis)

For the next planning discussion, prioritize:

1. Existing associations and supplied exact source/method/version evidence, with production history separate from ingestion/correction history and responsibility assertions separate from authentication.
2. Quality interpretation and the known JSON validation gap, without a universal score, confidence conversion or fusion algorithm.
3. Shared/per-item context and cross-representation preservation, including selected Part 3 publication and access-controlled lineage.

The prospective verification cases are concrete: changing a procedure must not silently reattribute earlier results; a mixed-context batch must preserve individual differences; unknown quality must not become certainty; and a consumer permitted to see an output must not learn protected inputs through links, filters, counts, schemas, errors or events. If a transformation changes bound/signed content, verification of the source cannot be advertised as verification of the output. These are inputs to Guide drafting, not claims of executed tests. [IDR-SRV-061 §7.1](idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md#71-implications)

A narrowly specified public PROV-compatible exchange or external provenance integration remains a **conditional alternative** if an identified consumer needs exact input/run/role exchange beyond the existing contract. It would need agreed discovery, identities/revisions, serialization, completeness/disclosure and lifecycle behavior. A policy-specific label/binding adapter similarly needs an applicable authoritative profile and concrete deployment need. Neither is adopted here; no new schema, endpoint, report entity or national/NATO security regime is presumed.

Remaining questions are the supported concrete quality interpretation, whether a consumer actually needs an additional public lineage contract, and whether a deployment requires a particular policy/binding profile. Part 5 remains the separate choice recorded in Addendum C. These are bounded planning inputs, not a requirement for another broad research cycle, an indefinite monitoring task or a new document framework. No implementation hours or release promises are inferred.

### D.5 Downstream Readiness and Handoff

The accepted research and this synthesis are sufficient to return to the agreed planning discussion:

| Document | Next consideration | Change made in this iteration |
|---|---|---|
| [Goal and Definition v1.7](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) | Discuss the recommendation that existing provenance scope needs no expansion; separately decide whether Part 5 warrants a bounded experiment | None; no new provenance platform, Part 5 deliverable or security-profile commitment. |
| [Implementation Guide v0.2](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md) | Discuss concise clarification in §§4.3–4.4, 4.7–4.8, 4.10–4.11, 6.1 and 8, alongside the pending Part 5 compatibility/experiment choice | None; no representation or dependency selected. |
| Roadmap | Sequence only subsequently agreed scope after the Guide is completed | No milestone, estimate or work package created. |

**Next step:** the Glaux Project Lead and Codex discuss the provenance and Part 5 Goal/Guide implications together. The next `proceed` starts that discussion; it does not automatically edit either document or authorize implementation. Make only subsequently agreed changes, then resume Guide drafting pass 2. No additional material is required from the project lead to begin that discussion.

**Addendum validation:** IDR-SRV-061 is accounted for as accepted research; conclusions, qualifications and limitations trace to its report; the original synthesis and Addenda A–C are preserved; current approved scope is unchanged; and the remaining choices and next action are explicit. Addendum D is prepared for review, distinct from the recorded acceptance of its source report.

---

## Addendum E: CS-GO Engineering Practices and Development History

**Date:** September 18, 2026<br>
**Status:** Source research accepted; planning discussion and approved clarifications complete (subsequent disposition in §E.6)<br>
**Source:** [IDR-SRV-062 Research Report](idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md), accepted by the Glaux Project Lead on September 18, 2026 through the established `proceed` workflow<br>
**Purpose:** Integrate practical engineering lessons before discussing two small planning clarifications and resuming complete implementation-issue publication

### E.1 Scope, Evidence and Completion Record

**The deeper CS-GO study reinforces Glaux's existing Rust design and implementation plan.** Its value is concrete examples of how changes, tests and explanations fit together, not a reason to copy a Go architecture or introduce another process framework.

| Supplemental topic | Plan | Report | Completion and acceptance | Synthesis conclusion |
|---|---|---|---|---|
| IDR-SRV-062: CS-GO Engineering Practices and Development History Study | [Research plan](../IDR%20Plans/idr-srv-062-cs-go-engineering-practices-and-development-history-study.md) | [Research report](idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md) | Research complete; accepted September 18, 2026; no prerequisite exception | Most lessons are already covered; discuss two bounded harness/documentation clarifications without changing capability scope or adding tasks. |

Coverage is now **67 original accepted topics plus five accepted supplemental topics, 058–062**. Original metadata, counts and acceptance remain historical; Addenda A–D retain their text. Report acceptance authorizes downstream use of the findings, not automatic adoption of the proposed clarifications or acceptance of this newly prepared addendum.

The planning baseline at addendum preparation was [Goal v1.7](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md), [Guide v1.0](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md) and [Roadmap v1.1](../../../../../Plans/glaux-server/glaux-server-roadmap.md). Later planning had resolved choices described as pending in earlier addenda: selected Part 3/4 experiments and bounded filtering remain in scope, Part 5 implementation remains deferred, and focused provenance/quality/disclosure treatment is incorporated. This addendum does not reopen those decisions; §E.6 records the subsequent Guide/Roadmap clarification versions.

This is synthesis of IDR-SRV-062 as published in `79f736ab6cda601a9ad0c4adc7f6bda9550786d3`, not new external research. Its [evidence base](idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md#3-evidence-base) and [reproduction record](idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md#122-reproduction-and-execution-record) control the details. The CS-GO source pin is `b1fd2e0e9bd69e222d05258d659a842ca24502cb`; the release comparison uses v1.0.4 at `244f4dd586da685d4d9b75e43f73001028b5bd0e`. Checks were dated September 18 EDT / September 19 UTC. The study inventoried 68 default-branch commits, 11 ordinary issues, five PRs, three releases and 76 test files, then examined selected cases deeply; it did not inspect every historical diff or private workflow.

No upstream refresh, runtime test, benchmark, installation or maintainer contact was performed for this addendum. Source counts are not passing-test or coverage results. IDR-SRV-014B remains the earlier dated study, not a report to overwrite with the newer main snapshot.

### E.2 Consolidated Findings

**A change should preserve meaning across the whole path.** The formatter refactor consolidates representation handling; spatial repairs connect parsing, storage and returned resource membership; observation storage changes preserve public identity when the partition-key time changes. The transferable lesson is coordinated API, persistence, test and documentation work. It does not select Timescale, GORM or automatic startup migrations for Glaux. [IDR-SRV-062 §4.1](idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md#41-q1--architecture-and-evolution)

**Assert the result that matters, including plausible wrong alternatives.** Useful peer tests check inside and outside resource IDs, changed values, followed links and repaired storage conditions. Other examples check only a count, URL prefix or one surviving label and can miss a wrong result or partial mutation. Independent expected values complement round trips and shared-validator checks; fake transports complement, but do not replace, real transaction and broker-failure tests. These examples reinforce the existing verification design rather than require another testing system. [IDR-SRV-062 §4.3](idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md#43-q3--test-craftsmanship-and-tdd)

**Technical judgment includes rejecting unnecessary work.** Maintainer replies distinguish a mistaken interpretation from an actual problem and identify an existing shared mechanism instead of adding a special case. Upstream implementation and contributed audit findings have different authorship. This supports standards-grounded review and reuse of existing mechanisms, not treating every AI-generated finding as a new requirement. [IDR-SRV-062 §4.2](idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md#42-q2--development-process-and-tools)

**Useful guidance needs the same maintenance as code.** Contributor notes, focused examples and comments explaining constraints help future implementers. Stale migration commands, version guidance and generated context summaries show why those materials must be checked against executable behavior. No particular assistant, generated map or extra document set is required. Public history shows maintained tests and declared tooling, but does not establish test-first sequencing, successful execution or private review practices. [IDR-SRV-062 §§4.2–4.4](idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md#44-q4--comments-documentation-and-coding-guidance)

### E.3 Effect on the Original Synthesis

| Original section / related research | What remains useful | Qualification supplied by IDR-SRV-062 |
|---|---|---|
| §§9–10: Representations, validation and persistence; 014B | Shared logical meaning, validation boundaries and transactional integrity | Learn the coordination and identity-preservation patterns, not Go-specific coupling or a new database dependency. Explicit Glaux migrations remain selected. |
| §11: Publication and ingestion | Shared application rules and separate transports | Peer after-handler publication is not evidence of durable committed-work recovery. Keep Glaux's transaction/outgoing-work and real fault tests. |
| §§15–16: Rust architecture and developer setup | Simple composition, focused modules and reproducible commands | Retain current packages/stack; commands and actual release artifacts need checking. The studied source-tag manifest discrepancy was not a broken downloadable checksum result. |
| §17 and 050–053: Verification and test strategy | Independent expectations, real persistence/HTTP, deterministic fixtures and truthful claims | Concrete peer examples sharpen assertions and harness design. An in-process HTTP listener is not shipped-binary testing; test presence is not TDD evidence; skipped checks are not conformance passes. |
| §24 and current planning: Documentation and workflow | Maintain understandable explanations alongside implementation | Apply ordinary documentation updates to any future contributor/assistant guidance. Do not reinstate earlier proposed document systems, work packages or new mandatory tooling. |
| Addenda A–D and approved scope | Existing experiments, filtering and provenance decisions | Engineering precedent creates no standards obligation, new conformance claim, Part 5 adoption or scope expansion. |

Exact Guide sections and existing three-level task IDs are recorded in [IDR-SRV-062 §4.6](idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md#46-q6--transfer-to-the-existing-rust-plan). They supply the implementation handoff without reproducing another task catalog here. The nine phases, 41 capability groups and 286 issue-sized tasks remain unchanged.

### E.4 Recommendation and Remaining Choices

**Keep the current goal and plan, and discuss two small clarifications:**

1. **Reliable test setup and cleanup:** In Guide §8.1 and existing task **1.1.3**, make prompt cleanup registration, failed-reset errors, isolated mutable fixtures and recorded seeds/clocks/dependency versions explicit. This makes a failed or unreproducible test easier to diagnose; it is not a new harness project.
2. **Current contributor guidance:** Extend the existing documentation-update rule to any future contributor/assistant notes, linking them to controlling sources and current executable examples. The existing Roadmap §5.2 rule and tasks **1.1.4 / 9.4.2** already provide the place for this work; no mandatory assistant-specific file or tool is proposed.

At addendum preparation, both were recommendations, not adopted edits; the subsequent approval is recorded in §E.6. Most other lessons can be applied directly when implementing their existing issues. No new capability, task count, delivery date or implementation-hour estimate follows. [IDR-SRV-062 §§5–7](idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md#5-decision-analysis)

Private TDD/tooling practice, unexecuted upstream tests and unmeasured performance remain explicit unknowns, not blockers to this discussion. Verify applicable terms if direct source/fixture copying is proposed; conceptual learning and independently authored Rust implementation do not require that copying. Optional maintainer questions remain unsent and are not a completion gate. [IDR-SRV-062 §8](idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md#8-risks-constraints-and-open-questions)

### E.5 Downstream Readiness and Handoff

| Artifact / workflow | Next consideration | Change made in this iteration |
|---|---|---|
| Goal v1.7 | No expansion recommended | None |
| Guide v1.0 | Discuss the two clarifications within existing sections | None |
| Roadmap v1.1 | If agreed, clarify existing completion expectations without adding leaves | None; all 286 tasks retained |
| GitHub implementation issues | Resume complete issue publication and link/dependency verification after discussion and any agreed planning edits | No issues created; no code written |

**Handoff at addendum publication:** the next `proceed` began discussion of the two clarifications, without automatically editing planning documents, creating issues or starting coding. That discussion and the subsequently approved edits are now complete; §E.6 supplies the current handoff. The complete-issue-set-before-coding workflow and one ready implementation issue per authorized iteration remain unchanged.

**Addendum validation at publication:** IDR-SRV-062 was accounted for as accepted research; conclusions traced to its report; original totals and Addenda A–D were preserved; runtime/TDD/source limitations remained explicit; scope and planning artifacts were unchanged; and remaining choices and handoff were bounded. Subsequent planning disposition is recorded below rather than treated as adoption during research.

### E.6 Subsequent Planning Disposition — September 18, 2026

After the discussion-only iteration, the project lead's next `proceed` approved the two clarifications and their publication. [Guide v1.1](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md) now makes test setup/reset/cleanup, fixture isolation and failure reproduction explicit in §8.1, and applies existing documentation maintenance to any contributor/assistant instructions in §4.12. [Roadmap v1.2](../../../../../Plans/glaux-server/glaux-server-roadmap.md) carries those expectations into existing task **1.1.3** and the common per-issue documentation rule in **§5.2**.

Goal v1.7, architecture, capability scope, all 286 task IDs/titles/dependencies and accepted research findings remain unchanged. No new document system, assistant tool, implementation task, issue or code was created. This is the separate approval and incorporation of the recommendations, not a retroactive change to their evidence.

**Current handoff:** the next `proceed` publishes and verifies the complete GitHub issue set in `DGIWG-P507/glaux-server`, checking existing issues to avoid duplicates and linking the corresponding Roadmap tasks and prerequisites. Coding begins only afterward, one ready issue per authorized iteration. No further research or planning-discussion iteration is required for these two clarifications.

# Glaux Server Overall IDR Research Plan

**Version:** 1.0
**Date:** June 7, 2026
**Status:** Draft
**Scope:** Initial Design Research (IDR) for `glaux-server`
**Plan Owner:** Glaux Core Team
**Final Report Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/final-idr-research-report.md`

---

## Purpose

This document is the **overall IDR research plan** for Glaux Server.

It defines the indexed set of research topics needed to produce a complete, high-rigor design foundation for a full OGC CSAPI Part 1 and Part 2 implementation.

---

## Research Objective

Produce a complete set of topic-level IDR research plans and topic-level research reports that establish:

- Standards-correct design direction
- Conformance-first implementation boundaries
- Traceable decisions for architecture, API behavior, validation, and testing

---

## Governance Rules (Required)

1. No topic research report without a topic research plan.
2. No topic plan without a matching topic ID in this overall index.
3. Topics are worked one at a time in index order unless explicitly re-prioritized.
4. Each completed topic produces exactly one topic research report.
5. The final overall IDR report is written only after all indexed topics are complete.
6. All reports must include explicit references and evidence.

---

## Operating Model

Research lifecycle:

1. Create this overall IDR research plan.
2. Build and finalize the topic index in this document.
3. Create one topic research plan per indexed topic.
4. Execute topic research one topic at a time.
5. Produce one topic research report per completed topic.
6. Produce one final overall IDR research report after all topics are complete.

---

## Planning Index of Topics

Scope rule for topic admission:

- A topic belongs in Glaux Server IDR only if the answer changes server obligations, API behavior, resource/data model, storage/query design, security model, tasking behavior, dynamic-data behavior, conformance strategy, deployment shape, or ecosystem integration contracts.

### Category A: Standards and Obligation Baseline

#### IDR-SRV-A01: STANAG 4789 / AEP-4789 Server Obligation Baseline

- Focus: Extract direct server obligations from the NATO framework.
- Output target: Server-obligation baseline with traceability anchors.

#### IDR-SRV-A02: AEP-4789 Volume I Functional Mapping to Server Responsibilities

- Focus: Map functional areas (discovery, registration/description, access/exchange, streaming/dynamic data, tasking/control, status/availability) to server responsibilities.
- Output target: Function-to-responsibility mapping baseline.

#### IDR-SRV-A03: AEP-4789 Volume II Standards Package Implementation Baseline

- Focus: Treat CSAPI Part 1, CSAPI Part 2, SensorML, and SWE Common as a coherent package for server implementation.
- Output target: Volume II implementation baseline.

#### IDR-SRV-A04: Terminology and Concept Crosswalk

- Focus: Crosswalk STANAG/AEP terminology to CSAPI/SensorML/SWE/Common and Glaux server terms.
- Output target: Terminology and concept crosswalk matrix.

#### IDR-SRV-A05: Related NATO Standards Boundary Review

- Focus: Identify adjacent standards only to define interoperability boundaries (not implementation absorption).
- Output target: Boundary and interoperability-not-implementation notes.

### Category B: CSAPI Server Behavior and Conformance

#### IDR-SRV-B01: CSAPI Part 1 Requirement Baseline

- Focus: Extract Part 1 normative server requirements.
- Output target: Part 1 requirement inventory.

#### IDR-SRV-B02: CSAPI Part 2 Requirement Baseline

- Focus: Extract Part 2 normative server requirements.
- Output target: Part 2 requirement inventory.

#### IDR-SRV-B03: Conformance Class and Requirement Mapping

- Focus: Map requirements to conformance classes and implementation obligations.
- Output target: Conformance matrix baseline.

#### IDR-SRV-B04: Landing Page, API Definition, and Conformance Declaration Behavior

- Focus: Define required behavior for landing page, API definition exposure, and conformance declaration.
- Output target: API entrypoint and declaration behavior baseline.

#### IDR-SRV-B05: Collections, Resources, Links, and Navigation Behavior

- Focus: Define collections/resources/linking/navigation behavior.
- Output target: Navigation and linking behavior baseline.

#### IDR-SRV-B06: Query, Filtering, Sorting, Pagination, and Selection Semantics

- Focus: Define server semantics for query/filter/sort/pagination/selection.
- Output target: Query behavior specification baseline.

#### IDR-SRV-B07: Content Negotiation, Media Types, and Encoding Selection

- Focus: Define content negotiation and media-type/encoding behavior.
- Output target: Representation and negotiation baseline.

#### IDR-SRV-B08: Error Model, HTTP Status Codes, and Failure Semantics

- Focus: Define deterministic error taxonomy, status mapping, and failure contracts.
- Output target: Error and failure behavior baseline.

#### IDR-SRV-B09: OpenAPI Description and API Documentation Strategy

- Focus: Define OpenAPI and API documentation strategy for server contracts.
- Output target: Documentation and machine-contract publication baseline.

### Category C: Server Resource and Domain Model

#### IDR-SRV-C01: Canonical Glaux Server Resource Model

- Focus: Define canonical resources, entities, and lifecycle boundaries.
- Output target: Canonical resource model baseline.

#### IDR-SRV-C02: Identifier, URI, and Resource Lifecycle Strategy

- Focus: Define persistent IDs, stable URIs, aliasing, updates, deletion, and tombstone handling.
- Output target: Identifier/URI/lifecycle strategy baseline.

#### IDR-SRV-C03: Relationship and Linkage Model

- Focus: Define hierarchy and linkage semantics across resources.
- Output target: Relationship and linkage baseline.

#### IDR-SRV-C04: Temporal, Validity, and Freshness Model

- Focus: Define phenomenon/result/valid/report time and stale/last-known semantics.
- Output target: Temporal and freshness baseline.

#### IDR-SRV-C05: Provenance, Lineage, Quality, and Trust Metadata Model

- Focus: Define provenance/lineage/quality/trust metadata expectations.
- Output target: Metadata and trust model baseline.

#### IDR-SRV-C06: Status, Availability, and System Event Model

- Focus: Define status/availability/event behavior and representation.
- Output target: Status and system-event model baseline.

### Category D: SensorML, SWE Common, and Semantic Representation

#### IDR-SRV-D01: SensorML Representation Strategy

- Focus: Define what SensorML is stored, validated, linked, generated, and exposed.
- Output target: SensorML representation baseline.

#### IDR-SRV-D02: SWE Common Data Component Strategy

- Focus: Define SWE Common handling for observations, status, command/task inputs, units, records, and arrays.
- Output target: SWE Common component baseline.

#### IDR-SRV-D03: Schema and Encoding Validation Strategy

- Focus: Define validation strategy across JSON Schema, OpenAPI, CSAPI, SensorML, and SWE.
- Output target: Validation architecture baseline.

#### IDR-SRV-D04: Units, Observed Properties, and Semantic Binding Strategy

- Focus: Define units/properties/semantic binding behavior.
- Output target: Semantic binding baseline.

### Category E: Server Persistence and Query Architecture

#### IDR-SRV-E01: Database and Persistence Architecture Options

- Focus: Evaluate persistence architecture options for server obligations.
- Output target: Persistence architecture decision baseline.

#### IDR-SRV-E02: Geospatial Storage and Query Strategy

- Focus: Define geospatial storage/index/query strategy.
- Output target: Geospatial strategy baseline.

#### IDR-SRV-E03: Time-Series Observation Storage Strategy

- Focus: Define time-series observation storage/query behavior.
- Output target: Time-series strategy baseline.

#### IDR-SRV-E04: Metadata and Document Storage Strategy

- Focus: Define metadata/document storage behavior and boundaries.
- Output target: Metadata/document strategy baseline.

#### IDR-SRV-E05: Transaction, Consistency, Idempotency, and Concurrency Strategy

- Focus: Define transactional and concurrency semantics.
- Output target: Consistency and mutation behavior baseline.

#### IDR-SRV-E06: Data Lifecycle, Retention, Archival, and Deletion Strategy

- Focus: Define data lifecycle controls and retention behavior.
- Output target: Lifecycle and retention baseline.

### Category F: Dynamic Data, Ingestion, and Tasking

#### IDR-SRV-F01: Server Write and Ingestion Model

- Focus: Define what server accepts directly vs rejects or delegates.
- Output target: Ingestion boundary baseline.

#### IDR-SRV-F02: Publisher-to-Server Contract Boundary

- Focus: Define server-side contract/auth/validation/error behavior for publisher integration.
- Output target: Publisher contract boundary baseline.

#### IDR-SRV-F03: Simulator-to-Server Contract Boundary

- Focus: Define server-side contract behavior for simulator replay/reset/synthetic interactions.
- Output target: Simulator contract boundary baseline.

#### IDR-SRV-F04: Datastream, Observation, and Status Update Semantics

- Focus: Define server semantics for dynamic update behavior.
- Output target: Dynamic update semantics baseline.

#### IDR-SRV-F05: Streaming and Event Publication Strategy

- Focus: Define server-side streaming/event behavior (protocols, subscriptions, replay, ordering, backpressure).
- Output target: Streaming/event strategy baseline.

#### IDR-SRV-F06: Control Stream and Command Lifecycle Model

- Focus: Define control/command lifecycle behavior.
- Output target: Tasking and control lifecycle baseline.

#### IDR-SRV-F07: Feasibility and Asynchronous Tasking Strategy

- Focus: Define feasibility exchange and asynchronous tasking behavior.
- Output target: Feasibility and async-tasking baseline.

#### IDR-SRV-F08: Command Authorization, Safety, and Audit Strategy

- Focus: Define command authorization/safety/audit semantics.
- Output target: Command-governance baseline.

### Category G: Security, Federation, and DDIL-Informed Server Behavior

#### IDR-SRV-G01: Authentication and Authorization Architecture

- Focus: Define authn/authz architecture for server scope.
- Output target: Authentication/authorization baseline.

#### IDR-SRV-G02: Policy, Releasability, and Cross-Boundary Access Constraints

- Focus: Define policy/releasability/cross-boundary constraints.
- Output target: Access-governance baseline.

#### IDR-SRV-G03: Audit Logging and Accountability Strategy

- Focus: Define audit and accountability behavior.
- Output target: Audit/accountability baseline.

#### IDR-SRV-G04: DDIL-Informed Server Semantics

- Focus: Define freshness/validity/last-known/delayed-update semantics for constrained operations.
- Output target: DDIL-informed behavior baseline.

#### IDR-SRV-G05: Server Synchronization and Conflict Handling Boundary

- Focus: Define server sync/conflict semantics without absorbing full network architecture scope.
- Output target: Synchronization boundary baseline.

### Category H: Implementation Platform and Reference Deployment

#### IDR-SRV-H01: Implementation Language and Framework Options

- Focus: Evaluate language/framework options for the reference server.
- Output target: Platform decision baseline.

#### IDR-SRV-H02: Service Architecture and Modularization Strategy

- Focus: Define service decomposition/modularization.
- Output target: Service architecture baseline.

#### IDR-SRV-H03: Reference Deployment Strategy

- Focus: Define deployment strategy sufficient for run/test/demo.
- Output target: Reference deployment baseline.

#### IDR-SRV-H04: Configuration, Secrets, and Environment Strategy

- Focus: Define configuration and secret-management approach.
- Output target: Configuration/environment baseline.

#### IDR-SRV-H05: Observability, Logs, Metrics, and Health Check Strategy

- Focus: Define observability and health strategy.
- Output target: Observability baseline.

#### IDR-SRV-H06: Migration, Upgrade, Backup, and Restore Strategy

- Focus: Define operational data continuity controls.
- Output target: Continuity and recoverability baseline.

### Category I: Verification and Implementation Readiness

#### IDR-SRV-I01: Conformance Harness Strategy

- Focus: Define conformance-harness architecture and evidence model.
- Output target: Conformance harness baseline.

#### IDR-SRV-I02: Requirement-to-Test Traceability Strategy

- Focus: Define requirement-to-test traceability controls.
- Output target: Traceability strategy baseline.

#### IDR-SRV-I03: Unit, Integration, Contract, and End-to-End Test Architecture

- Focus: Define test architecture layers and boundaries.
- Output target: Test architecture baseline.

#### IDR-SRV-I04: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy

- Focus: Define data/fixture/golden/scenario strategy.
- Output target: Test data and scenario baseline.

#### IDR-SRV-I05: Performance, Load, Stress, and Streaming Test Strategy

- Focus: Define performance and streaming verification strategy.
- Output target: Performance verification baseline.

#### IDR-SRV-I06: Security, Authorization, and Command-Control Test Strategy

- Focus: Define security/authorization/command-control verification approach.
- Output target: Security and control verification baseline.

#### IDR-SRV-I07: Interoperability Test Matrix for External CSAPI Clients

- Focus: Define external-client interoperability test matrix.
- Output target: Interoperability verification baseline.

#### IDR-SRV-I08: Final Glaux Server IDR Synthesis Report

- Focus: Consolidate all findings into an implementation-ready synthesis package.
- Output target: Final IDR synthesis report and architecture decision baseline.

---

## Topic Execution Order

Default execution order follows category dependencies in sequence:

1. Category A (IDR-SRV-A01 through IDR-SRV-A05)
2. Category B (IDR-SRV-B01 through IDR-SRV-B09)
3. Category C (IDR-SRV-C01 through IDR-SRV-C06)
4. Category D (IDR-SRV-D01 through IDR-SRV-D04)
5. Category E (IDR-SRV-E01 through IDR-SRV-E06)
6. Category F (IDR-SRV-F01 through IDR-SRV-F08)
7. Category G (IDR-SRV-G01 through IDR-SRV-G05)
8. Category H (IDR-SRV-H01 through IDR-SRV-H06)
9. Category I (IDR-SRV-I01 through IDR-SRV-I08)

Dependency rationale:

- A establishes obligation boundaries before implementation semantics.
- B defines externally visible server behavior before internal modeling/storage decisions.
- C and D stabilize domain and representation semantics before persistence/dynamic-data strategy.
- E, F, and G define storage, runtime interaction, and policy constraints before deployment and testing strategy.
- H defines implementation/deployment shape before final verification architecture.
- I closes with readiness, traceability, and final synthesis.

Order may change only when:

- A dependency requires reordering, or
- A documented priority decision is made and recorded.

---

## Objective Quality Standards

Each topic report must satisfy all standards below.

### Evidence Standard

- Claims are supported by direct references.
- References are specific and reproducible (URL, section/clause, artifact path).
- Assumptions are explicitly marked.

### Decision-Usefulness Standard

- Conclusion is explicit (not implied).
- Recommendation is actionable and bounded.
- Open issues and impact are clearly documented.

### Completeness Standard

- Topic plan scope is fully addressed.
- Core questions are answered.
- Unresolved items are explicitly listed with next action.

---

## Topic Completion Criteria

A topic is complete when all are true:

- Topic-specific research plan exists and is in scope.
- Topic-specific report is completed.
- Report includes evidence and references.
- Report conclusion is explicit and decision-usable.
- Open issues (if any) are clearly listed.
- Topic status in the progress table is updated to `Complete`.

---

## Overall IDR Completion Criteria

The overall IDR research effort is complete when all are true:

- All indexed topics in Categories A through I have completed topic reports.
- Findings are sufficiently complete to support goal and definition, implementation guide, and roadmap authoring.
- A final overall IDR research report is produced that responds to this overall plan.

---

## Final Overall IDR Report Requirement

After all indexed topics are complete, produce:

- `final-idr-research-report.md`

The final report must:

- Respond directly to this overall plan
- Summarize each topic conclusion
- State consolidated overall conclusions
- Identify any unresolved cross-topic issues

---

## Change Log for Reprioritization and Scope Changes

| Date | Change Type | Description | Rationale | Approved By |
|---|---|---|---|---|
| 2026-06-07 | Plan Alignment | Added objective quality standards, change control, and progress tracking sections to align with governance template | Improve objective governance and repeatability | Glaux Core Team |
| 2026-06-07 | Topic Reorganization | Grouped topics into dependency-based categories and replaced ID-order execution with evidence-driven sequence | Improve research flow so outputs inform downstream topics in a wise order | Glaux Core Team |
| 2026-06-07 | Scope Bounding and Expanded Topic Model | Replaced compact topic set with bounded full-scope server IDR categories (A-I) and server-only admission rule | Keep full-scope server rigor while preventing ecosystem research bleed-in from other components | Glaux Core Team |

---

## Progress Tracking

| Category | Topics | Plan Coverage | Report Coverage | Status | Last Updated | Notes |
|---|---|---|---|---|---|---|
| A | IDR-SRV-A01 to IDR-SRV-A05 | TBD | TBD | Not Started | 2026-06-07 | |
| B | IDR-SRV-B01 to IDR-SRV-B09 | TBD | TBD | Not Started | 2026-06-07 | |
| C | IDR-SRV-C01 to IDR-SRV-C06 | TBD | TBD | Not Started | 2026-06-07 | |
| D | IDR-SRV-D01 to IDR-SRV-D04 | TBD | TBD | Not Started | 2026-06-07 | |
| E | IDR-SRV-E01 to IDR-SRV-E06 | TBD | TBD | Not Started | 2026-06-07 | |
| F | IDR-SRV-F01 to IDR-SRV-F08 | TBD | TBD | Not Started | 2026-06-07 | |
| G | IDR-SRV-G01 to IDR-SRV-G05 | TBD | TBD | Not Started | 2026-06-07 | |
| H | IDR-SRV-H01 to IDR-SRV-H06 | TBD | TBD | Not Started | 2026-06-07 | |
| I | IDR-SRV-I01 to IDR-SRV-I08 | TBD | TBD | Not Started | 2026-06-07 | |

---

## Risks and Constraints

- Normative specification interpretation may vary by clause and cross-reference depth.
- Upstream interoperability behaviors may diverge from strict conformance language.
- Scope expansion risk if topic boundaries are not enforced at report time.

---

## References

- Glaux Governance: Research Planning Approach
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- Glaux Server Initial Planning Guidance
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/initial-planning-guidance.md
- OGC CSAPI planning exemplar (OS4CSAPI)
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/planning

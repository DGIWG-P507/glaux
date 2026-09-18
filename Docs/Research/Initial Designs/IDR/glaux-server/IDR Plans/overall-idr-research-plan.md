# Glaux Server Overall IDR Research Plan

**Version:** 3.93<br>
**Date:** September 17, 2026<br>
**Status:** Original IDR and supplemental IDR-SRV-058 complete; synthesis addendum prepared for review<br>
**Scope:** Initial Design Research (IDR) for `glaux-server`<br>
**Plan Owner:** Glaux Project Lead<br>
**Final Report Model:** Indexed synthesis topic `IDR-SRV-057`<br>
**Final Report Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/final-idr-research-report.md`

---

## Purpose

This document is the **overall IDR research plan** for Glaux Server.

It defines the indexed set of research topics needed to produce a complete, high-rigor design foundation for a Rust-based Glaux Server implementation aligned to STANAG 4789 / AEP-4789, OGC API - Connected Systems Parts 1 and 2, SensorML, and SWE Common.

The research effort shall support a test-driven implementation approach for a robust, best-of-breed, open-source OGC API - Connected Systems reference server written in Rust. Architecture, API behavior, validation behavior, conformance strategy, and implementation sequencing shall be informed by standards evidence and by current Rust, web API, security, and testing best-practice research.

The topic reports are intended to serve as common decision material for both the project lead and AI-assisted development. Each report shall therefore be polished, independently readable, evidence-backed, clear about recommendations and unresolved questions, and directly useful when the implementation guide and roadmap are written.

---

## Research Objective

Produce a complete set of topic-level IDR research plans and topic-level research reports that establish:

- Standards-correct design direction
- Conformance-first implementation boundaries
- Traceable decisions for architecture, API behavior, validation, and testing
- Rust-first implementation architecture and framework decisions
- Test-driven design and implementation strategy
- Research-backed Rust testing, CI, quality-gate, and dependency-management practices

The research has two equal purposes:

- prevent standards misunderstandings and poorly informed implementation choices during AI-assisted development; and
- provide AI-assisted development with sufficient high-quality knowledge and analysis to build the strongest practical reference implementation.

---

## Governance Rules (Required)

1. No topic research report without a topic research plan.
2. No topic plan without a matching topic ID in this overall index.
3. Topics are worked one at a time in index order unless explicitly re-prioritized.
4. Before a topic starts, every indexed-topic report named as its prerequisite must be complete and accepted by the plan owner. This rule supersedes softer wording retained in any topic plan, including `should`, `when available`, `unavailable`, `deferred`, or `provisional`. An exception requires the plan owner's explicit approval and a recorded rationale, scope impact, and downstream handling before execution. A project-produced report is not "unavailable" merely because it is incomplete.
5. External-source unavailability is an evidence limitation, not completion of an indexed topic. Record the source, access limitation, affected questions, and resulting limits without inventing or silently substituting content.
6. Each completed topic produces exactly one research report. For `IDR-SRV-057`, the final overall IDR report is that topic's report; no separate topic report is created.
7. Execution of `IDR-SRV-057`, including drafting the final overall IDR report, begins only after every preceding indexed topic report (`IDR-SRV-001` through `IDR-SRV-056`, including letter-suffixed topics) is complete and accepted, except for an exception approved and recorded under Rule 4. Later supplemental topics do not invalidate that historical completion and acceptance.
8. All reports must include explicit, reproducible references and evidence.
9. `Accepted` means the plan owner has reviewed a completed report for alignment with its topic plan and suitability for downstream decisions. The acceptance authority and date are recorded in the report, and aggregate acceptance coverage is recorded in progress tracking.
10. Every completed-topic handoff shall state the next two actions when another topic remains: plan-owner acceptance of the completed report, followed by authorization to execute exactly one next eligible topic. The handoff shall provide a single combined response pattern that performs both actions in one message. The handoff wording alone neither records acceptance nor begins the next topic; only the plan owner's combined instruction does so.
11. The shared OGC API - Connected Systems upstream-history register is a required supporting evidence source for topics whose conclusions can be affected by official repository issues, pull requests, commits, releases, or recorded design rationale. Each such topic shall consult and date-check only its relevant entries, follow linked resolution artifacts, and add or update entries when material evidence has changed.
12. The upstream-history register does not alter the authority hierarchy. Approved standards and normatively incorporated artifacts control. Pre-publication history may explain the published result; post-publication changes may show maintenance direction; open issues, comments, and unmerged pull requests remain unresolved informative evidence. No register entry creates a Glaux requirement by itself.

---

## Operating Model

Research lifecycle:

1. Create this overall IDR research plan.
2. Build and finalize the topic index in this document.
3. Create one topic research plan per indexed topic.
4. Execute topic research one topic at a time in the order below, after satisfying each topic's prerequisites.
5. Produce and obtain plan-owner acceptance of one topic research report for each of `IDR-SRV-001` through `IDR-SRV-056`.
6. Execute `IDR-SRV-057` last and produce the final overall IDR research report, which also serves as the `IDR-SRV-057` report, after every other topic is complete and accepted or any Rule 4 exception is explicitly accounted for.

All topic-level research plans shall be drafted before topic execution begins. Topic execution shall then proceed one topic at a time, using the approved topic-level research plan to produce the corresponding topic-level research report.

The shared upstream-history register is maintained across topics as supporting evidence. Maintaining or refreshing it does not constitute execution of another indexed topic, provided each update is bounded to the active topic's evidence needs and does not make decisions owned by an unstarted topic.

---

**Original IDR Topics:** 67 (complete and accepted)<br>
**Post-Synthesis Supplemental Topics:** 1 (IDR-SRV-058, complete and accepted)

Throughout the Glaux Server IDR plan set, a numeric topic range includes every letter-suffixed topic inserted within that indexed range unless the text explicitly excludes it. For example, `IDR-SRV-001` through `IDR-SRV-040` includes `IDR-SRV-010A`, `IDR-SRV-014A` through `IDR-SRV-014H`, and `IDR-SRV-039A`.

Scope rule for topic admission:

- A topic belongs in Glaux Server IDR only if the answer changes server obligations, API behavior, resource/data model, storage/query design, security model, tasking behavior, dynamic-data behavior, conformance strategy, deployment shape, or ecosystem integration contracts.

### Category A: Standards and Obligation Baseline

#### IDR-SRV-001: STANAG 4789 / AEP-4789 Server Obligation Baseline

- Focus: Extract direct server obligations from the NATO framework.
- Output target: Server-obligation baseline with traceability anchors.

#### IDR-SRV-002: AEP-4789 Volume I Functional Mapping to Server Responsibilities

- Focus: Map functional areas (discovery, registration/description, access/exchange, streaming/dynamic data, tasking/control, status/availability) to server responsibilities.
- Output target: Function-to-responsibility mapping baseline.

#### IDR-SRV-003: AEP-4789 Volume II Standards Package Implementation Baseline

- Focus: Treat CSAPI Part 1, CSAPI Part 2, SensorML, and SWE Common as a coherent package for server implementation.
- Output target: Volume II implementation baseline.

#### IDR-SRV-004: Terminology and Concept Crosswalk

- Focus: Crosswalk STANAG/AEP terminology to CSAPI/SensorML/SWE/Common and Glaux server terms.
- Output target: Terminology and concept crosswalk matrix.

#### IDR-SRV-005: Related NATO Standards Boundary Review

- Focus: Identify adjacent standards only to define interoperability boundaries (not implementation absorption).
- Output target: Boundary and interoperability-not-implementation notes.

### Category B: CSAPI Server Behavior and Conformance

#### IDR-SRV-006: CSAPI Part 1 Requirement Baseline

- Focus: Extract Part 1 normative server requirements.
- Output target: Part 1 requirement inventory.

#### IDR-SRV-007: CSAPI Part 2 Requirement Baseline

- Focus: Extract Part 2 normative server requirements.
- Output target: Part 2 requirement inventory.

#### IDR-SRV-008: Conformance Class and Requirement Mapping

- Focus: Map requirements to conformance classes and implementation obligations.
- Output target: Conformance matrix baseline.

#### IDR-SRV-009: Landing Page, API Definition, and Conformance Declaration Behavior

- Focus: Define required behavior for landing page, API definition exposure, and conformance declaration.
- Output target: API entrypoint and declaration behavior baseline.

#### IDR-SRV-010: Collections, Resources, Links, and Navigation Behavior

- Focus: Define collections/resources/linking/navigation behavior.
- Output target: Navigation and linking behavior baseline.

#### IDR-SRV-010A: API Versioning, Backward Compatibility, and Deprecation Strategy

- Focus: Define how Glaux Server versions its APIs, manages backward compatibility, deprecates features, and communicates changes while remaining CSAPI-conformant.
- Output target: API versioning and evolution baseline.

#### IDR-SRV-011: Query, Filtering, Sorting, Pagination, and Selection Semantics

- Focus: Define server semantics for query/filter/sort/pagination/selection.
- Output target: Query behavior specification baseline.

#### IDR-SRV-012: Content Negotiation, Media Types, and Encoding Selection

- Focus: Define content negotiation and media-type/encoding behavior.
- Output target: Representation and negotiation baseline.

#### IDR-SRV-013: Error Model, HTTP Status Codes, and Failure Semantics

- Focus: Define deterministic error taxonomy, status mapping, and failure contracts.
- Output target: Error and failure behavior baseline.

#### IDR-SRV-014: OpenAPI Description and API Documentation Strategy

- Focus: Define OpenAPI and API documentation strategy for server contracts.
- Output target: Documentation and machine-contract publication baseline.

#### IDR-SRV-014A: OSH CSAPI Server Implementation Study

- Focus: Research the OSH / OpenSensorHub CSAPI server implementation approach, behavior, architecture, standards alignment, API patterns, conformance posture, strengths, gaps, and lessons relevant to Glaux Server.
- Output target: OSH implementation findings baseline.

#### IDR-SRV-014B: Connected Systems Go CSAPI Server Implementation Study

- Focus: Research the Connected Systems Go / CS-GO CSAPI server implementation approach, behavior, architecture, standards alignment, API patterns, conformance posture, strengths, gaps, and lessons relevant to Glaux Server.
- Output target: Connected Systems Go implementation findings baseline.

#### IDR-SRV-014C: pygeoapi CSAPI Server Implementation Study

- Focus: Research the pygeoapi CSAPI server implementation approach, behavior, architecture, standards alignment, API patterns, conformance posture, strengths, gaps, and lessons relevant to Glaux Server.
- Output target: pygeoapi CSAPI implementation findings baseline.

#### IDR-SRV-014D: SECD CSAPI Server Implementation Study

- Focus: Research the SECD CSAPI server implementation approach, behavior, architecture, standards alignment, API patterns, conformance posture, strengths, gaps, and lessons relevant to Glaux Server.
- Output target: SECD implementation findings baseline.

#### IDR-SRV-014E: OS4CSAPI Client Smoke Test Findings Study

- Focus: Research the smoke-test findings, compatibility observations, implementation gaps, and interoperability lessons identified in the OS4CSAPI client work at https://github.com/OS4CSAPI/ogc-client-CSAPI_2, especially findings that affect Glaux Server API behavior, conformance, validation, interoperability, and test strategy.
- Output target: OS4CSAPI client smoke-test findings baseline.

#### IDR-SRV-014F: SECD Interoperability Findings Study

- Focus: Research the findings, issues, implementation observations, compatibility notes, and interoperability lessons identified in https://github.com/Sam-Bolling/csapi-server-interop-secd, especially findings that affect Glaux Server API behavior, conformance, validation, interoperability, and test strategy.
- Output target: SECD interoperability findings baseline.

#### IDR-SRV-014G: OS4CSAPI Discussions Lessons-Learned Study

- Focus: Review the discussions at https://github.com/orgs/OS4CSAPI/discussions to identify lessons learned, implementation concerns, interoperability issues, developer pain points, standards interpretation questions, testing implications, and community recommendations relevant to Glaux Server.
- Output target: OS4CSAPI discussions lessons-learned baseline.

#### IDR-SRV-014H: Draft CSAPI Part 3 Publish/Subscribe and Implementation Study

- Focus: Research the exact authority, maturity, requirements, gaps, dependencies, and conformance posture of the current draft OGC API - Connected Systems Part 3 Publish/Subscribe material; compare the active CS-Go and OpenSensorHub implementations; identify cross-implementation divergence, interoperability and security implications, and early constraints for Glaux resource, event, schema, persistence, ingestion, policy, and test design without selecting the final streaming architecture.
- Output target: Draft Part 3 authority, implementation, interoperability, downstream-handoff, and adoption-readiness baseline.

### Category C: Server Resource and Domain Model

#### IDR-SRV-015: Canonical Glaux Server Resource Model

- Focus: Define canonical resources, entities, and lifecycle boundaries.
- Output target: Canonical resource model baseline.

#### IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy

- Focus: Define persistent IDs, stable URIs, aliasing, updates, deletion, and tombstone handling.
- Output target: Identifier/URI/lifecycle strategy baseline.

#### IDR-SRV-017: Relationship and Linkage Model

- Focus: Define hierarchy and linkage semantics across resources.
- Output target: Relationship and linkage baseline.

#### IDR-SRV-018: Temporal, Validity, and Freshness Model

- Focus: Define phenomenon/result/valid/report time and stale/last-known semantics.
- Output target: Temporal and freshness baseline.

#### IDR-SRV-019: Provenance, Lineage, Quality, and Trust Metadata Model

- Focus: Define provenance/lineage/quality/trust metadata expectations.
- Output target: Metadata and trust model baseline.

#### IDR-SRV-020: Status, Availability, and System Event Model

- Focus: Define status/availability/event behavior and representation.
- Output target: Status and system-event model baseline.

### Category D: SensorML, SWE Common, and Semantic Representation

#### IDR-SRV-021: SensorML Representation Strategy

- Focus: Define what SensorML is stored, validated, linked, generated, and exposed.
- Output target: SensorML representation baseline.

#### IDR-SRV-022: SWE Common Data Component Strategy

- Focus: Define SWE Common handling for observations, status, command/task inputs, units, records, and arrays.
- Output target: SWE Common component baseline.

#### IDR-SRV-023: Schema and Encoding Validation Strategy

- Focus: Define validation strategy across JSON Schema, OpenAPI, CSAPI, SensorML, and SWE.
- Output target: Validation architecture baseline.

#### IDR-SRV-024: Units, Observed Properties, and Semantic Binding Strategy

- Focus: Define units/properties/semantic binding behavior.
- Output target: Semantic binding baseline.

### Category E: Server Persistence and Query Architecture

#### IDR-SRV-025: Database and Persistence Architecture Options

- Focus: Evaluate persistence architecture options for server obligations.
- Output target: Persistence architecture decision baseline.

#### IDR-SRV-026: Geospatial Storage and Query Strategy

- Focus: Define geospatial storage/index/query strategy.
- Output target: Geospatial strategy baseline.

#### IDR-SRV-027: Time-Series Observation Storage Strategy

- Focus: Define time-series observation storage/query behavior.
- Output target: Time-series strategy baseline.

#### IDR-SRV-028: Metadata and Document Storage Strategy

- Focus: Define metadata/document storage behavior and boundaries.
- Output target: Metadata/document strategy baseline.

#### IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy

- Focus: Define transactional and concurrency semantics.
- Output target: Consistency and mutation behavior baseline.

#### IDR-SRV-030: Data Lifecycle, Retention, Archival, and Deletion Strategy

- Focus: Define data lifecycle controls and retention behavior.
- Output target: Lifecycle and retention baseline.

### Category F: Dynamic Data, Ingestion, and Tasking

#### IDR-SRV-031: Server Write and Ingestion Model

- Focus: Define what server accepts directly vs rejects or delegates.
- Output target: Ingestion boundary baseline.

#### IDR-SRV-032: Publisher-to-Server Contract Boundary

- Focus: Define server-side contract/auth/validation/error behavior for publisher integration.
- Output target: Publisher contract boundary baseline.

#### IDR-SRV-033: Simulator-to-Server Contract Boundary

- Focus: Define server-side contract behavior for simulator replay/reset/synthetic interactions.
- Output target: Simulator contract boundary baseline.

#### IDR-SRV-034: Datastream, Observation, and Status Update Semantics

- Focus: Define server semantics for dynamic update behavior.
- Output target: Dynamic update semantics baseline.

#### IDR-SRV-035: Streaming and Event Publication Strategy

- Focus: Define server-side streaming/event behavior (protocols, subscriptions, replay, ordering, backpressure).
- Output target: Streaming/event strategy baseline.

#### IDR-SRV-036: Control Stream and Command Lifecycle Model

- Focus: Define control/command lifecycle behavior.
- Output target: Tasking and control lifecycle baseline.

#### IDR-SRV-037: Feasibility and Asynchronous Tasking Strategy

- Focus: Define feasibility exchange and asynchronous tasking behavior.
- Output target: Feasibility and async-tasking baseline.

#### IDR-SRV-038: Command Authorization, Safety, and Audit Strategy

- Focus: Define command authorization/safety/audit semantics.
- Output target: Command-governance baseline.

### Category G: Security, Federation, and DDIL-Informed Server Behavior

#### IDR-SRV-039: Authentication, Authorization, and API Security Threat Model

- Focus: Define authn/authz architecture and API security threat model for server scope, including object-level authorization, function-level authorization, command/tasking authorization, resource-consumption controls, and security misconfiguration risks.
- Output target: Authentication, authorization, and API threat-model baseline.

#### IDR-SRV-039A: Zero-Trust Architecture Alignment and Enforcement Model

- Focus: Define how zero-trust architecture principles are mapped, enforced, and verified across Glaux Server identities, resources, commands, data flows, and trust boundaries.
- Output target: Zero-trust alignment and enforcement baseline.

#### IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints

- Focus: Define policy/releasability/cross-boundary constraints.
- Output target: Access-governance baseline.

#### IDR-SRV-041: Audit Logging and Accountability Strategy

- Focus: Define audit and accountability behavior.
- Output target: Audit/accountability baseline.

#### IDR-SRV-042: DDIL-Informed Server Semantics

- Focus: Define freshness/validity/last-known/delayed-update semantics for constrained operations.
- Output target: DDIL-informed behavior baseline.

#### IDR-SRV-043: Server Synchronization and Conflict Handling Boundary

- Focus: Define server sync/conflict semantics without absorbing full network architecture scope.
- Output target: Synchronization boundary baseline.

### Category H: Implementation Platform and Reference Deployment

#### IDR-SRV-044: Rust Implementation Language and Framework Strategy

- Focus: Research current Rust server implementation options, including web framework, async runtime, database access, serialization, validation, OpenAPI support, error handling, observability, ecosystem maturity, dependency management, supply-chain risk, license compatibility, unsafe-code policy, static analysis, fuzz/property-based testing considerations, and CI quality gates.
- Output target: Rust implementation platform decision baseline.

#### IDR-SRV-045: Service Architecture and Modularization Strategy

- Focus: Define service decomposition/modularization.
- Output target: Service architecture baseline.

#### IDR-SRV-046: Reference Deployment Strategy

- Focus: Define deployment strategy sufficient for run/test/demo.
- Output target: Reference deployment baseline.

#### IDR-SRV-047: Configuration, Secrets, and Environment Strategy

- Focus: Define configuration and secret-management approach.
- Output target: Configuration/environment baseline.

#### IDR-SRV-048: Observability, Logs, Metrics, and Health Check Strategy

- Focus: Define observability and health strategy.
- Output target: Observability baseline.

#### IDR-SRV-049: Migration, Upgrade, Backup, and Restore Strategy

- Focus: Define operational data continuity controls.
- Output target: Continuity and recoverability baseline.

### Category I: Verification and Implementation Readiness

#### IDR-SRV-050: Conformance Harness Strategy

- Focus: Define conformance-harness architecture and evidence model.
- Output target: Conformance harness baseline.

#### IDR-SRV-051: Requirement-to-Test Traceability Strategy

- Focus: Define requirement-to-test traceability controls.
- Output target: Traceability strategy baseline.

#### IDR-SRV-052: Rust Test-Driven Architecture and Multi-Layer Test Strategy

- Focus: Define the Rust test-driven implementation strategy, including unit tests, integration tests, API contract tests, conformance tests, database-backed tests, async tests, golden-file tests, fuzz testing, property-based testing, CI execution, quality gates, static analysis, unsafe-code policy enforcement, and security tooling.
- Output target: Rust TDD and multi-layer test strategy baseline.

#### IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy

- Focus: Define data/fixture/golden/scenario strategy.
- Output target: Test data and scenario baseline.

#### IDR-SRV-054: Performance, Load, Stress, and Streaming Test Strategy

- Focus: Define performance and streaming verification strategy.
- Output target: Performance verification baseline.

#### IDR-SRV-055: Security, Authorization, and Command-Control Test Strategy

- Focus: Define security/authorization/command-control verification approach.
- Output target: Security and control verification baseline.

#### IDR-SRV-056: Interoperability Test Matrix for External CSAPI Clients

- Focus: Define external-client interoperability test matrix.
- Output target: Interoperability verification baseline.

#### IDR-SRV-057: Final Glaux Server IDR Synthesis Report

- Focus: Produce the governance-mandated final overall IDR report by consolidating all completed topic findings into an implementation-ready research synthesis.
- Output target: `final-idr-research-report.md`, the single final overall IDR report and architecture-decision evidence baseline. It supplies inputs to, but does not replace, the later Implementation Guide or Roadmap.

---

## Supplemental Research After Initial IDR Closeout

The original 67-topic IDR and IDR-SRV-057 synthesis were completed and accepted on September 16, 2026. The following supplement is tracked separately; it does not reopen those topics or change their completion counts.

#### IDR-SRV-058: Draft CSAPI Part 4 Sampling Features Study

- Status: Complete; the Glaux Project Lead accepted the [report](../IDR%20Reports/idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md) through the next `proceed` after its publication, September 17, 2026. The separately authorized [synthesis addendum](../IDR%20Reports/final-idr-research-report.md#addendum-a-draft-csapi-part-4-sampling-features) is prepared for review; no implementation option has been selected.
- Focus: Assess the official Part 4 working draft against the approved Parts 1 and 2 baseline, including feature types, requirements and schemas, maturity gaps, bounded implementation evidence, and implications for Glaux's existing research and server design. Recommend whether and how to consider experimental support without adopting it through the research itself.
- Plan: [idr-srv-058-draft-csapi-part-4-sampling-features-study.md](idr-srv-058-draft-csapi-part-4-sampling-features-study.md).
- Output target: `IDR Reports/idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md`.
- Sequence: Plan/publication first; separately authorized research/report next; then an accepted-findings addendum to the final synthesis in a later iteration; then discussion of any Goal and Definition or Implementation Guide changes. The user's established `proceed` workflow applies; no special acceptance phrase is required.
- Boundary: Preserve the original accepted synthesis and approved server scope. Research acceptance and addendum preparation are complete steps; the next iteration is discussion of any Goal/Guide implications, not automatic document edits or implementation.

---

## Topic Execution Order

The original IDR execution followed category dependencies in sequence; later supplemental work follows the separate sequence above:

1. Category A (IDR-SRV-001 through IDR-SRV-005)
2. Category B core behavior topics (IDR-SRV-006 through IDR-SRV-014, plus IDR-SRV-010A)
3. Existing implementation, smoke-test, interoperability, lessons-learned, and draft Part 3 studies (IDR-SRV-014A through IDR-SRV-014H)
4. Category C (IDR-SRV-015 through IDR-SRV-020)
5. Category D (IDR-SRV-021 through IDR-SRV-024)
6. Category E (IDR-SRV-025 through IDR-SRV-030)
7. Category F (IDR-SRV-031 through IDR-SRV-038)
8. Category G (IDR-SRV-039, IDR-SRV-039A, and IDR-SRV-040 through IDR-SRV-043)
9. Category H (IDR-SRV-044 through IDR-SRV-049)
10. Category I (IDR-SRV-050 through IDR-SRV-057)

IDR-SRV-044 and IDR-SRV-052 were drafted early during topic-plan development. That historical drafting order does not change their research execution positions in Categories H and I. All topic-level research plans shall still be drafted before topic execution begins.

Dependency rationale:

- A establishes obligation boundaries before implementation semantics.
- B defines externally visible server behavior before internal modeling/storage decisions.
- IDR-SRV-014A through IDR-SRV-014G capture existing implementation, smoke-test, interoperability, and community lessons; IDR-SRV-014H adds an early authority-qualified study of the active draft Part 3 message model and independent MQTT implementations so later model, event, transaction, policy, and test topics can avoid binding prematurely to one mutable implementation contract.
- IDR-SRV-044 and IDR-SRV-052 were drafted early as planning inputs, but their research executes in Categories H and I after their stated prerequisites.
- C and D stabilize domain and representation semantics before persistence/dynamic-data strategy.
- E, F, and G define storage, runtime interaction, and policy constraints before full deployment-shape finalization.
- IDR-SRV-044 through IDR-SRV-049 complete implementation-platform, modularization, and deployment-shape research before final verification architecture.
- I closes with readiness, traceability, and final synthesis.

Order may change only when:

- A dependency requires reordering, or
- A documented priority decision is made and recorded.

Any change must identify the affected prerequisites, scope impact, downstream handling, approval, and change-log entry before the reordered topic begins.

---

## Objective Quality Standards

Each topic report must satisfy all standards below.

### Evidence Standard

- Claims are supported by direct references.
- References are specific and reproducible (URL, section/clause, artifact path).
- Assumptions are explicitly marked.
- Relevant official standards-repository history is traced through the shared register, linked to its issue/PR/commit/release evidence, and authority-classified without overriding the approved standard.
- Research involving Rust implementation, testing, CI, security tooling, framework selection, or dependency management shall use current online sources and record source dates, tool versions, and assumptions where relevant.

### Literature Review Standard

Each topic-level research report shall function as a focused literature review and applied research assessment. Reports shall identify the relevant sources reviewed, summarize what those sources establish, synthesize key findings, assess implementation implications, provide recommendations, and identify unresolved questions or gaps.

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
- Topic report is written as a focused literature review and applied research assessment.
- Report includes evidence and references.
- Report conclusion is explicit and decision-usable.
- Topic report includes findings, recommendations, implementation implications, and unresolved questions.
- Open issues (if any) are clearly listed.
- The plan owner has accepted the report and the report records the acceptance authority and date.
- The topic plan status is updated to `Complete`, and the category's report and accepted-report coverage counts are updated in the progress table.

---

## Overall IDR Completion Criteria

The overall IDR research effort is complete when all are true:

- Every topic preceding `IDR-SRV-057` has a completed and accepted report, or a plan-owner-approved exception is recorded with its approval, rationale, scope impact, and downstream handling.
- Findings are sufficiently complete to support goal and definition, implementation guide, and roadmap authoring.
- A final overall IDR research report is produced and accepted, responds to this overall plan, and thereby completes `IDR-SRV-057`.

---

## Final Overall IDR Report Requirement

IDR-SRV-057 produces the single governance-mandated final overall IDR report:

- `final-idr-research-report.md`

Start that report only after every other indexed topic report (`IDR-SRV-001` through `IDR-SRV-056`, including all letter-suffixed topics) is complete and accepted, except for an exception approved and recorded under the Governance Rules above. Completion and acceptance of the final report completes `IDR-SRV-057` itself.

The final report must:

- Respond directly to this overall plan
- Summarize each topic conclusion
- State consolidated overall conclusions
- Identify any unresolved cross-topic issues
- State evidence-backed readiness and planning inputs for the later Implementation Guide and Roadmap without replacing either artifact

---

## Change Log for Reprioritization and Scope Changes

| Date | Change Type | Description | Rationale | Approved By |
|---|---|---|---|---|
| 2026-06-07 | Plan Alignment | Added objective quality standards, change control, and progress tracking sections to align with governance template | Improve objective governance and repeatability | Glaux Core Team |
| 2026-06-07 | Topic Reorganization | Grouped topics into dependency-based categories and replaced ID-order execution with evidence-driven sequence | Improve research flow so outputs inform downstream topics in a wise order | Glaux Core Team |
| 2026-06-07 | Scope Bounding and Expanded Topic Model | Replaced compact topic set with bounded full-scope server IDR categories (A-I) and server-only admission rule | Keep full-scope server rigor while preventing ecosystem research bleed-in from other components | Glaux Core Team |
| 2026-06-07 | Rust/TDD Refinement | Added IDR-SRV-014A, strengthened security threat-model and Rust platform topics, and adjusted topic-plan drafting order to draft 044/052 earlier | Make Rust and test-driven obligations first-class and better sequence dependency-informing research | Glaux Core Team |
| 2026-06-07 | Topic Index Refinement | Added API versioning topic, refined existing CSAPI/conformance survey, strengthened Rust robustness/TDD coverage, and clarified early drafting of Rust platform and TDD plans | Incorporate external review feedback while preserving full-plan-before-execution workflow | Glaux Core Team |
| 2026-06-07 | Existing Implementation Research Expansion | Replaced broad IDR-SRV-014A survey with focused studies for OSH, Connected Systems Go, pygeoapi, SECD, OS4CSAPI smoke-test findings, SECD interoperability findings, and OS4CSAPI discussion lessons learned | Ensure existing implementation and interoperability evidence is researched in enough detail to inform Glaux Server design and test strategy | Glaux Core Team |
| 2026-06-12 | Security Topic Expansion | Added IDR-SRV-039A for zero-trust architecture alignment and enforcement model within Category G | Strengthen security architecture research coverage between threat modeling and policy/access constraints | Glaux Core Team |
| 2026-07-29 | Baseline Clarification | Confirmed Rust as the implementation language and clarified that polished research reports serve both the project lead and AI-assisted development | Preserve the project's simple implementation goal while making the purpose of research explicit | Glaux Project Lead |
| 2026-07-29 | Plan Baseline Repair | Realigned seven topic plans—IDR-SRV-019, IDR-SRV-030, IDR-SRV-031, IDR-SRV-032, IDR-SRV-033, IDR-SRV-037, and IDR-SRV-041—with the approved index, registered the controlling NATO draft package locally, and normalized report-directory and final-report targets | Ensure the research library produces the intended evidence for a Rust CSAPI reference-server implementation | Glaux Project Lead |
| 2026-07-30 | Review Adjudication Hardening | Established the Glaux Project Lead as plan owner and acceptance/exception authority, distinguished internal completion gates from external evidence gaps, clarified research execution order and final-report boundaries, and registered targeted topic-plan corrections from two independent reviews | Freeze a clear, reproducible baseline before executing IDR-SRV-001 without broad plan churn | Glaux Project Lead |
| 2026-07-31 | Transition Handoff Rule | Required each completed-topic handoff to state acceptance and next-topic authorization as the next two actions and provide one combined response pattern | Make one-topic research transitions predictable without weakening plan-owner acceptance or single-topic controls | Glaux Project Lead |
| 2026-08-01 | Upstream Standards-History Evidence Control | Established a bounded shared register for relevant official CSAPI issues, pull requests, releases, commits, and design rationale; assigned topic-level refresh and authority-classification rules; repaired 89 obsolete OpenAPI source links across the topic-plan corpus | Preserve implementation-relevant standards-maintenance context and reproducible OAS access without allowing mutable GitHub history to override approved standards or become an unbounded research rabbit hole | Glaux Project Lead |
| 2026-08-31 | IDR-SRV-014B Research Completion | Completed the Connected Systems Go implementation study at upstream release `v1.0.4`, distinguished the OS4CSAPI audit fork and historical comparison fork, and placed the report in review | Provide a second evidence-rich CSAPI server implementation baseline for architecture, validation, conformance, interoperability, persistence, and test design | Pending Glaux Project Lead review |
| 2026-08-31 | IDR-SRV-014B Acceptance and IDR-SRV-014C Authorization | Accepted the Connected Systems Go implementation study and authorized the bounded pygeoapi implementation-study iteration | Preserve the single-topic review boundary while continuing the implementation comparison sequence | Glaux Project Lead |
| 2026-08-31 | IDR-SRV-014C Research Completion | Completed the pygeoapi/52°North CSAPI proof-of-concept study, distinguished core framework, dependency, standalone PoC, generated artifacts, and live deployments, and placed the report in review | Capture provider/configuration lessons and representation, consistency, validation, OpenAPI, security, and testing risks before Glaux model and persistence decisions | Pending Glaux Project Lead review |
| 2026-08-31 | IDR-SRV-014C Acceptance and IDR-SRV-014D Authorization | Accepted the pygeoapi/52°North CSAPI proof-of-concept implementation study and authorized the bounded SECD implementation-study iteration | Preserve the single-topic review boundary while continuing the implementation comparison sequence | Glaux Project Lead |
| 2026-08-31 | IDR-SRV-014D Research Completion | Completed the evidence-bounded SECD black-box implementation study, reconciled May captures with the August live deployment, and placed the report in review | Capture drone, dynamic-data, tasking, schema, discovery, query, negotiation, OpenAPI, security, drift, and test lessons without inferring unavailable internals or preempting the later interoperability study | Pending Glaux Project Lead review |
| 2026-08-31 | IDR-SRV-014D Acceptance and IDR-SRV-014E Authorization | Accepted the evidence-bounded SECD implementation study and authorized the bounded OS4CSAPI client smoke-test findings iteration | Preserve the single-topic review boundary while continuing from implementation behavior into client-observed evidence | Glaux Project Lead |
| 2026-08-31 | IDR-SRV-014E Research Completion | Completed the pinned OS4CSAPI client smoke-test findings study, separated server, client, fixture, harness, documentation, and standards ownership, reproduced the deterministic client integration baseline, and placed the report in review | Convert practical multi-server and multi-client failures into bounded Glaux API, validation, conformance, fixture, and interoperability handoffs without promoting implementation quirks into requirements | Pending Glaux Project Lead review |
| 2026-08-31 | IDR-SRV-014E Acceptance and IDR-SRV-014F Authorization | Accepted the ownership-classified OS4CSAPI client smoke-test findings study and authorized the bounded SECD interoperability findings iteration | Preserve the single-topic review boundary while moving from the general client smoke corpus into the SECD-specific interoperability corpus | Glaux Project Lead |
| 2026-08-31 | IDR-SRV-014F Research Completion | Completed the pinned SECD interoperability findings study, reconciled the adjudicated May evidence with August deployment drift, directly exercised the pinned OS4CSAPI client, and placed the report in review | Convert silent-result, discovery, representation, lifecycle, tasking, fixture, and client-composition failures into freshness-qualified Glaux handoffs without treating the implementation or historical score as normative | Pending Glaux Project Lead review |
| 2026-08-31 | IDR-SRV-014F Acceptance and IDR-SRV-014G Authorization | Accepted the freshness-qualified SECD interoperability findings study and authorized the bounded OS4CSAPI discussions lessons-learned iteration | Preserve the single-topic review boundary while moving from dedicated SECD test evidence into community discussion evidence | Glaux Project Lead |
| 2026-08-31 | IDR-SRV-014G Research Completion | Completed the full OS4CSAPI discussion inventory, separated independently supported lessons from bounded opinion and unimplemented proposals, reconciled linked artifact state with the accepted implementation studies, and placed the report in review | Close the Category B evidence block with responsible community-evidence handoffs for model, validation, documentation, security, conformance, fixtures, and interoperability | Pending Glaux Project Lead review |
| 2026-08-31 | IDR-SRV-014G Acceptance and Draft Part 3 Planning Authorization | Accepted the OS4CSAPI discussions lessons-learned report and authorized one planning-only iteration to insert a bounded draft Part 3 and implementation study before IDR-SRV-015, revise the later IDR-SRV-035 decision boundary, and refresh the Part 3 evidence pin | Active CS-Go and OSH Part 3 work makes early evidence review valuable, while the incomplete draft binding requires separation between adoption readiness now and final architecture selection later | Glaux Project Lead |
| 2026-08-31 | Draft Part 3 Planning Amendment | Added IDR-SRV-014H, revised IDR-SRV-015 and IDR-SRV-035 dependencies and handoffs, refreshed the official Part 3 snapshot to `c95c1d60`, and kept Part 3 research and implementation unstarted pending review | Preserve the full-plan-before-execution workflow and expose mutable-draft, implementation-divergence, conformance, migration, and interoperability risks before resource-model decisions | Glaux Project Lead |
| 2026-08-31 | Draft Part 3 Planning Acceptance and IDR-SRV-014H Authorization | Accepted the Part 3 planning amendment and authorized the bounded draft-authority and implementation-study iteration only | Preserve the single-topic review boundary while obtaining early Part 3 constraints before canonical resource-model research | Glaux Project Lead |
| 2026-08-31 | IDR-SRV-014H Research Completion | Completed the pinned Part 3 authority/completeness review, reproduced 44 focused CS-Go tests, bounded OSH PR evidence, demonstrated topic and field incompatibilities, and classified Part 3 as an experimental candidate pending the final IDR-SRV-035 architecture/profile decision | Carry useful event, transaction, security, and fixture constraints into later topics without freezing the missing MQTT binding or overstating draft conformance | Pending Glaux Project Lead review |
| 2026-08-31 | IDR-SRV-014H Acceptance and IDR-SRV-015 Authorization | Accepted the authority-qualified draft Part 3 and implementation study and authorized the bounded canonical Glaux Server resource-model iteration | Close Category B while carrying transport-neutral identity, lifecycle, relationship, event, validation, and compatibility constraints into the first Category C topic without authorizing Part 3 implementation | Glaux Project Lead |
| 2026-08-31 | IDR-SRV-015 Research Completion | Defined the canonical encoding-neutral Glaux resource graph; distinguished API resources, aggregate entities, association facts, append-oriented records, projections, schemas, and support concepts; mapped lifecycle, relationship, temporal, status, event, SensorML/SWE, persistence, validation, fixture, and downstream constraints; and placed the report in review | Give later identity, relationship, temporal, metadata, status, representation, persistence, validation, and test research one standards-traceable resource-model baseline without selecting their mechanisms prematurely | Pending Glaux Project Lead review |
| 2026-09-03 | IDR-SRV-015 Acceptance and IDR-SRV-016 Authorization | Accepted the canonical encoding-neutral resource-model baseline and authorized the bounded identifier, URI, and resource-lifecycle strategy iteration | Preserve the single-topic review boundary while moving from canonical entity distinctions into stable identity, aliasing, revision, deletion, and tombstone decisions | Glaux Project Lead |
| 2026-09-03 | IDR-SRV-016 Research Completion | Defined typed local-ID, UID, canonical-URL, external-identifier, alias, revision, event, message, and persistence identities; selected service-wide UUIDv7 resource IDs; fixed canonical route, collision, replacement, revision, lifecycle, deletion, and tombstone rules; and placed the report in review | Give relationship, temporal, provenance, persistence, transaction, lifecycle, security, fixture, conformance, and interoperability work a stable identity baseline without starting IDR-SRV-017 or Part 3 implementation | Pending Glaux Project Lead review |
| 2026-09-13 | IDR-SRV-016 Acceptance and IDR-SRV-017 Authorization | Accepted the typed identifier, canonical-addressing, alias, revision, replacement, lifecycle, deletion, and tombstone baseline and authorized the bounded relationship and linkage model iteration | Preserve the single-topic review boundary while moving from stable resource identity into standards-traceable relationship facts, direction, cardinality, link generation, traversal, and integrity | Glaux Project Lead |
| 2026-09-13 | IDR-SRV-017 Research Completion | Defined the typed canonical relationship graph, authoritative/derived direction rules, 28-row resource-family matrix, CSAPI and project link-relation strategy, bounded traversal, external/DDIL resolution states, integrity constraints, policy seams, and downstream validation/test handoffs; placed the report in review | Give temporal, provenance, status/event, representation, persistence, transaction, security, fixture, conformance, and interoperability work one standards-traceable relationship baseline without starting IDR-SRV-018, Part 3 implementation, or server implementation | Pending Glaux Project Lead review |
| 2026-09-13 | IDR-SRV-017 Acceptance and IDR-SRV-018 Authorization | Accepted the typed canonical relationship graph, authoritative/derived direction, link-relation, traversal, external/DDIL resolution, integrity, policy-seam, and test baseline and authorized the bounded temporal, validity, and freshness model iteration | Preserve the single-topic review boundary while moving from stable resource relationships into explicit time dimensions, interval semantics, current/as-of evaluation, freshness, uncertainty, and retention handoffs | Glaux Project Lead |
| 2026-09-13 | IDR-SRV-018 Research Completion | Defined the multi-axis temporal taxonomy, 21-row resource-family temporal matrix, bitemporal revision seam, request-scoped current/as-of/latest rules, conservative open-bound adapter policy, domain-versus-HTTP freshness separation, DDIL context, persistence implications, and temporal regression corpus; placed the report in review | Give provenance, status/event, representation, persistence, transaction, dynamic-data, streaming, tasking, security, DDIL, conformance, fixture, and interoperability work one standards-traceable temporal baseline without starting IDR-SRV-019, Part 3 implementation, or server implementation | Pending Glaux Project Lead review |
| 2026-09-13 | IDR-SRV-018 Acceptance and IDR-SRV-019 Authorization | Accepted the multi-axis temporal, bitemporal revision, current/as-of/latest, open-bound containment, freshness separation, DDIL, persistence-handoff, and test baseline and authorized the bounded provenance, lineage, quality, and trust metadata model iteration | Preserve the single-topic review boundary while moving from explicit temporal evidence into source identity, derivation, authority, quality, trust, uncertainty, and disclosure semantics | Glaux Project Lead |
| 2026-09-13 | IDR-SRV-019 Research Completion | Defined a PROV-compatible canonical evidence graph, complete resource/artifact and activity coverage, risk-based source-fidelity rules, reproducible transformation records, scoped quality/uncertainty assertions, multidimensional trust evidence, state-change lineage, controlled exposure, persistence/synchronization handoffs, graph invariants, and a fixture corpus; placed the report in review | Give status/event, representation, persistence, ingestion, tasking, security, DDIL, conformance, fixture, and interoperability work one standards-traceable evidence model without starting IDR-SRV-020, a public provenance extension, Part 3 implementation, or server implementation | Pending Glaux Project Lead review |
| 2026-09-13 | IDR-SRV-019 Acceptance and IDR-SRV-020 Authorization | Accepted the PROV-compatible evidence graph, atomic provenance boundary, source-fidelity and transformation rules, scoped quality/uncertainty assertions, multidimensional trust evidence, controlled exposure, persistence/synchronization handoffs, invariants, and fixture baseline and authorized the bounded status, availability, and System Event model iteration | Preserve the single-topic review boundary while moving from traceable evidence into operational status, availability, event vocabulary, current-state projection, and stale/unknown behavior | Glaux Project Lead |
| 2026-09-13 | IDR-SRV-020 Research Completion | Defined the non-collapsing status taxonomy, capability-specific five-state availability assessment, observation-derived current-status projection, narrow DataStream/ControlStream `live` semantics, durable System Event boundary and generation matrix, command-status/feasibility separation, degraded/DDIL behavior, published event-artifact gap handling, and downstream persistence/security/test handoffs; placed the report in review | Close Category C with one standards-traceable operational-state and event baseline without starting IDR-SRV-021, choosing a public status profile, implementing draft Part 3, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-13 | IDR-SRV-020 Acceptance and IDR-SRV-021 Authorization | Accepted the non-collapsing status taxonomy, capability-specific availability algebra, observation-derived current-state rules, System Event boundary, command-status separation, stale/unknown behavior, published-gap handling, and downstream handoffs and authorized the bounded SensorML representation-strategy iteration | Close Category C and preserve the single-topic review boundary while moving into SensorML representation, normalization, preservation, validation, and interoperability decisions | Glaux Project Lead |
| 2026-09-14 | IDR-SRV-021 Research Completion | Defined a five-layer SensorML source/parsed/canonical/generated/validation architecture; mapped applicable SensorML classes and concepts to CSAPI/Glaux resources; established normalization, source-preservation, strict-write/quarantine-import, deterministic-generation, inheritance-resolution, security, validation, and fixture rules; retained published `DataInterface`, stream-name, qualifier, relation-wording, and System Event gaps as explicit adapter/test seams; and placed the report in review | Give SWE Common, validation, semantic, persistence, ingestion, dynamic-data, command, security, fixture, conformance, and interoperability work one standards-traceable SensorML representation baseline without starting IDR-SRV-022, implementing draft Part 3, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-14 | IDR-SRV-021 Acceptance and IDR-SRV-022 Authorization | Accepted the five-layer SensorML representation architecture, canonical normalization and source-preservation boundary, strict-write/quarantine-import split, deterministic generation, inheritance-resolution controls, non-collapse rules, published-gap handling, security posture, validation ladder, fixture baseline, and downstream handoffs and authorized the bounded SWE Common data-component strategy iteration | Preserve the single-topic review boundary while moving from SensorML representation into reusable SWE component, constraint, aggregate, stream, encoding, validation, and interoperability decisions | Glaux Project Lead |
| 2026-09-14 | IDR-SRV-022 Research Completion | Defined an immutable versioned SWE data-contract subsystem; mapped all published SWE Common 3.0 component families to SensorML and CSAPI dynamic resources; selected full model/preservation coverage with staged capability-gated JSON, Text, and Binary codecs; established nil, optional, constraint, quality, component-path, ordering, schema-revision, security, validation, fixture, and interoperability rules; and placed the report in review | Give validation, semantic binding, persistence, ingestion, dynamic-data, command, security, conformance, fixture, and interoperability work one standards-traceable SWE Common baseline without starting IDR-SRV-023, implementing draft Part 3, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-14 | IDR-SRV-022 Acceptance and IDR-SRV-023 Authorization | Accepted the immutable SWE contract architecture, full component-model preservation, capability-gated codec strategy, exact value-state and component-path rules, CSAPI schema bindings, validation ladder, security posture, fixture matrix, and downstream handoffs and authorized the bounded schema and encoding validation-strategy iteration | Preserve the single-topic review boundary while moving from the SWE component contract into reusable structural, semantic, profile, encoding, compatibility, diagnostic, and evidence validation decisions | Glaux Project Lead |
| 2026-09-14 | IDR-SRV-023 Research Completion | Defined a versioned evidence-producing validation pipeline and contract registry; mapped schema, OpenAPI, media, encoding, resource-family, and interaction-stage responsibilities; established strict-write/quarantine, immutable stream-contract binding, offline `$ref`, safe diagnostics, capability-gated codecs, security, and multi-lane test rules; reconciled material published artifact conflicts; and placed the report in review | Give semantic binding, persistence, ingestion, dynamic-data, command, security, DDIL, tooling, conformance, fixture, and interoperability work one standards-traceable validation baseline without starting IDR-SRV-024, implementing draft Part 3, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-14 | IDR-SRV-023 Acceptance and IDR-SRV-024 Authorization | Accepted the versioned validation pipeline, provenance-bearing contract registry, trusted contract-selection rules, seven-type validation taxonomy, immutable parent-contract binding, strict-write/quarantine split, offline `$ref` posture, safe diagnostics, capability-gated codec strategy, and multi-lane test baseline and authorized the bounded units, observed-properties, and semantic-binding strategy iteration | Close the validation architecture decision and preserve the single-topic review boundary while moving into semantic identifiers, units, observed/controlled properties, vocabulary binding, query, and interoperability behavior | Glaux Project Lead |
| 2026-09-14 | IDR-SRV-024 Research Completion | Defined a five-view source/parsed/canonical/materialized/generated semantic architecture; contextual property-role bindings; a UCUM-centered, AEP-profiled, no-silent-command-conversion unit strategy; immutable offline vocabulary packages; conservative explainable query and mapping behavior; Version 1.0 adapters for property-summary conflicts; explicit #178/#179 compatibility seams; and security, fixture, conformance, interoperability, and downstream handoffs; placed the report in review | Close Category D research execution with a standards-traceable semantic and unit baseline without accepting the report, starting Category E, implementing draft Part 3, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-14 | IDR-SRV-024 Acceptance and IDR-SRV-025 Authorization | Accepted the five-view semantic architecture, contextual property-role bindings, UCUM-centered and AEP-profiled unit strategy, prohibition on silent Command conversion, immutable offline vocabulary packages, conservative explainable query model, Version 1.0 property-summary adapters, #178/#179 compatibility seams, and security/test handoffs; authorized the bounded database and persistence architecture-options iteration | Close Category D and preserve the single-topic review boundary while moving into persistence workload, consistency, storage-family, source-fidelity, portability, DDIL, and operational tradeoff research | Glaux Project Lead |
| 2026-09-14 | IDR-SRV-025 Research Completion | Selected a PostgreSQL/PostGIS relational-hybrid authoritative-core direction; classified 26 data categories; separated authoritative state, append evidence, exact content-addressed artifacts, derived projections, caches, and broker delivery; defined a native PostgreSQL time-series baseline and measured TimescaleDB gate; rejected unproven polyglot/graph/broker-as-truth defaults; and established DDIL, transaction, migration, security, Rust, deployment, test, risk, and downstream handoffs; placed the report in review | Give specialized persistence topics one coherent, open-source, transaction-centered baseline without accepting the report, starting IDR-SRV-026, choosing detailed schemas or all products, implementing draft Part 3, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-14 | IDR-SRV-025 Acceptance and IDR-SRV-026 Authorization | Accepted the PostgreSQL/PostGIS relational-hybrid authoritative-core direction, 26-category persistence inventory, native PostgreSQL time-series baseline, measured TimescaleDB gate, structured/JSONB/artifact split, transactional outbox, relational-edge model, derived-cache rules, DDIL boundary, migration/security/Rust/test guidance, and evidence gates; authorized the bounded geospatial storage and query-strategy iteration | Preserve the single-topic review boundary while moving from cross-cutting persistence architecture into CRS-aware geometry, spatial indexing, feature history, spatial-temporal query, policy, and interoperability decisions | Glaux Project Lead |
| 2026-09-14 | IDR-SRV-026 Research Completion | Selected PostGIS as full-profile spatial authority; defined exact-source, canonical-assertion, query-projection, generated-view, derived-product, and cache layers; classified 26 spatial categories across resource families; established CRS84, safe-transform, vertical, antimeridian, bbox/geom, indexing, dynamic-location, policy, DDIL, fixture, performance, and interoperability rules; and placed the report in review | Give later persistence, ingestion, dynamic-data, tasking, security, DDIL, API, fixture, performance, conformance, and interoperability topics one coherent geospatial baseline without accepting the report, authorizing IDR-SRV-027, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-14 | IDR-SRV-026 Acceptance and IDR-SRV-027 Authorization | Accepted the PostGIS full-profile spatial authority, 26-category spatial inventory, exact/canonical/query/generated/derived/cache layering, CRS84 and transform rules, vertical and antimeridian boundaries, bbox/geom semantics, index strategy, dynamic-location model, policy/DDIL controls, test corpus, and downstream handoffs; authorized the bounded time-series observation-storage iteration | Preserve the single-topic review boundary while moving from spatial assertions and spatial-temporal requirements into observation/status sample persistence, partitioning, retention, compression, late data, and workload decisions | Glaux Project Lead |
| 2026-09-14 | IDR-SRV-027 Research Completion | Defined typed family-specific temporal stores; selected native PostgreSQL as the authoritative time-series baseline; advanced result-time Observation partitioning with a commit-time benchmark alternative and atomic identity/admission-ledger seam; established multi-clock, latest/tie, indexing, replay/backfill, retention/archive/summary, DDIL, policy, fixture, performance, and interoperability rules; retained TimescaleDB behind a measured adoption gate; and placed the report in review | Give later persistence, ingestion, dynamic-data, streaming, tasking, security, DDIL, deployment, fixture, performance, conformance, and interoperability topics one coherent time-series baseline without accepting the report, authorizing IDR-SRV-028, implementing draft Part 3, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-14 | IDR-SRV-027 Acceptance and IDR-SRV-028 Authorization | Accepted the typed family-specific temporal-store model, native PostgreSQL authority, result-time partition candidate and commit-time benchmark alternative, identity/admission-ledger seam, multi-clock/latest/tie/index/replay/retention/archive/summary rules, conditional TimescaleDB gate, and DDIL/policy/test handoffs; authorized the bounded metadata and document storage iteration | Preserve the single-topic review boundary while moving from high-volume temporal facts and derived projections into exact-source, canonical metadata, document, indexing, revision, artifact, and retrieval boundaries | Glaux Project Lead |
| 2026-09-14 | IDR-SRV-028 Research Completion | Defined a six-role source/parsed/canonical/generated/evidence/cache architecture; selected a PostgreSQL-authoritative artifact catalog and normalized graph with immutable content-addressed byte storage; established exact-source, identity, indexing, version/lifecycle, offline package, validation evidence, provenance, DDIL, policy, redaction, fixture, and interoperability rules; evaluated document-storage options; and placed the report in review | Give transaction, lifecycle, ingestion, security, DDIL, deployment, conformance, fixture, performance, and interoperability work one coherent metadata/document baseline without accepting the report, authorizing IDR-SRV-029, implementing draft Part 3, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-14 | IDR-SRV-028 Acceptance and IDR-SRV-029 Authorization | Accepted the six-role source/parsed/canonical/generated/evidence/cache architecture, PostgreSQL-authoritative artifact catalog and normalized graph, immutable content-addressed byte storage, exact-source and identifier boundaries, selective indexing, immutable lifecycle and validation evidence, offline package resolver, provenance/DDIL behavior, policy/redaction controls, and test handoffs; authorized the bounded transaction, consistency, idempotency, and concurrency strategy iteration | Preserve the single-topic review boundary while moving from persisted resource/document/time-series authorities into atomicity, consistency, duplicate/replay, conditional update, conflict, outbox/inbox, and high-risk workflow decisions | Glaux Project Lead |
| 2026-09-14 | IDR-SRV-029 Research Completion | Defined a PostgreSQL-centered unit-of-work model; selected Read Committed with constraints/CAS by default and targeted locks/Serializable invariants; required strong ETag/If-Match mutation preconditions; established scoped idempotency, effectively-once local effects, at-least-once delivery, inbox/offset and atomic outbox rules; defined command, DDIL, conflict, retry, error, security, audit, Rust, and verification boundaries; and placed the report in review | Give lifecycle, ingestion, streaming, command, security, DDIL, architecture, operations, conformance, fixture, performance, and interoperability work one coherent transaction baseline without accepting the report, authorizing IDR-SRV-030, implementing draft Part 3, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-14 | IDR-SRV-029 Acceptance and IDR-SRV-030 Authorization | Accepted the PostgreSQL-centered unit-of-work model, constraints/CAS default with targeted locks/Serializable invariants, strong ETag/If-Match mutation preconditions, scoped idempotency, effectively-once local effects, at-least-once delivery, atomic inbox/offset/outbox boundaries, command-safety and DDIL conflict behavior, retry/error/audit rules, and verification handoffs; authorized the bounded data lifecycle, retention, archival, and deletion strategy iteration | Close transaction consistency decisions and preserve the single-topic review boundary while moving into policy authority, lifecycle states, retention triggers, holds, archives, tombstones, purge/sanitization, distributed-copy propagation, and restoration | Glaux Project Lead |
| 2026-09-14 | IDR-SRV-030 Research Completion | Defined a versioned policy-authority model without unsupported periods; separated semantic, custody, storage, disposition, backup-expiry, and media-sanitization states; inventoried every authoritative, evidentiary, derived, recovery, export, broker, replica, and DDIL copy; established retention triggers, holds, dependency-aware cascades, tombstones, dynamic-data compaction, command/raw/quarantine treatment, reference-closed archives, isolated restore with deletion-ledger replay, copy-by-copy purge evidence, API/admin behavior, and verification fixtures; and placed the report in review | Close Category E research execution with one truthful lifecycle baseline without accepting the report, authorizing IDR-SRV-031, implementing draft Part 3, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-14 | IDR-SRV-030 Acceptance and IDR-SRV-031 Authorization | Accepted the versioned policy-authority model, orthogonal lifecycle states, no-invented-period rule, complete record/copy inventory, retention triggers and holds, authority-aware cascades, policy-safe tombstones, dynamic-data transformation rules, command/raw/quarantine controls, reference-closed archive and isolated restore contract, deletion/purge/backup-expiry/sanitization distinctions, DDIL anti-resurrection behavior, API/admin guarantees, and verification suite; authorized the bounded server write and ingestion model iteration | Close Category E and preserve the single-topic review boundary while moving into admission, validation, normalization, transaction, idempotency, quarantine, batch, replay, provenance, and publishability decisions | Glaux Project Lead |
| 2026-09-14 | IDR-SRV-031 Research Completion | Defined a single authoritative write boundary and registry-driven capability model; inventoried every standards-facing resource mutation and private/admin entry point; established the 16-field accept/delegate/reject matrix, strict POST/PUT/PATCH/DELETE profile, common processing/state pipeline, validation/normalization and source-authority allocation, transaction/idempotency/replay and item-atomic batch rules, persistence/current-state/outbox/audit boundary, error/backpressure behavior, and verification/handoff suite; placed the report in review | Open Category F research execution with one coherent server ingestion baseline without accepting the report, authorizing IDR-SRV-032, implementing draft Part 3, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-14 | IDR-SRV-031 Acceptance and IDR-SRV-032 Authorization | Accepted the single authoritative write boundary, registry-driven capability model, strict mutation profile, 16-field write-surface disposition matrix, common processing and state pipeline, validation/normalization and source-authority allocation, transaction/idempotency/replay and item-atomic batch rules, persistence/current-state/outbox/audit boundary, error/backpressure behavior, and verification/handoff suite; authorized the bounded publisher-to-server contract iteration | Preserve the single-topic review boundary while moving from the common server mutation authority into publisher-specific submission, identity, authentication context, delivery, retry, backpressure, error, and evidence decisions | Glaux Project Lead |
| 2026-09-14 | IDR-SRV-032 Research Completion | Defined standards-facing CSAPI writes plus a supplemental Glaux Publisher Contract and narrow status/import/health surfaces; separated principal, publisher instance, represented source, authority, validation, policy, and trust; established the 21-field contract matrix, registration lifecycle, responsibility allocation, payload/envelope rules, delivery and acknowledgement states, item-atomic batching, idempotency/ordering/retry/replay/backpressure behavior, safe diagnostics, DDIL/evolution constraints, and verification/handoff suites; placed the report in review | Specialize the common server mutation boundary for publisher integration without accepting the report, authorizing IDR-SRV-033, selecting or implementing draft Part 3, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-14 | IDR-SRV-032 Acceptance and IDR-SRV-033 Authorization | Accepted the standards-facing CSAPI plus supplemental GPC-v1 publisher boundary, principal/publisher/source/authority distinctions, registration lifecycle, responsibility allocation, 21-field contract matrix, narrow status/import/health surfaces, validation/custody rules, delivery/idempotency/batch/retry/DDIL/error/evolution behavior, fixture suite, and downstream handoffs; authorized the bounded simulator-to-server contract iteration | Preserve the single-topic review boundary while specializing publisher admission for deterministic simulator identity, fixture, time, replay, reset, fault, isolation, evidence, and safety behavior | Glaux Project Lead |
| 2026-09-14 | IDR-SRV-033 Research Completion | Defined a two-role ordinary GPC/CSAPI data plane and production-disabled narrow simulator control plane; established the 19-field operation matrix, scenario/dataset/generator/session/run/source/resource identity model, durable lifecycle/checkpoint/resume/completion rules, domain-only virtual-time mapping, synthetic provenance and hard isolation, disposable-first reset hierarchy with previewed exclusive-ownership fallback, normal-boundary fault/DDIL/tasking behavior, exact scenario manifest, and verification/handoff suites; placed the report in review | Provide deterministic and destructive-test controls without accepting the report, authorizing IDR-SRV-034, weakening normal server safeguards, implementing draft Part 3, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-14 | IDR-SRV-033 Acceptance and IDR-SRV-034 Authorization | Accepted the two-role ordinary GPC/CSAPI data plane and production-disabled narrow simulator control plane, 19-field operation matrix, identity/lifecycle model, durable replay/checkpoint/completion semantics, domain-only virtual-time boundary, synthetic provenance/isolation, disposable-first reset hierarchy, normal-boundary fault/DDIL/tasking behavior, exact scenario manifest, and verification/handoff suites; authorized the bounded Datastream, Observation, and status-update semantics iteration | Preserve the single-topic review boundary while moving from controlled publication/simulation inputs into parent stream contracts, observation identity/time/order, status projection, corrections, latest/extents, batching, and query-visible dynamic-state behavior | Glaux Project Lead |
| 2026-09-14 | IDR-SRV-034 Research Completion | Defined DataStreams as versioned homogeneous semantic contracts and Observations as exact-contract typed facts; separated status Observations, dynamic-property snapshots, System Events, source health, CommandStatus, audits, and delivery records; established domain-time latest/current selectors, correction/replay/batch behavior, schema/unit/nil/quality/provenance rules, policy-first stable query and rebuildable persistence projections, publication/DDIL/command boundaries, and fixture/conformance/performance/interoperability handoffs; placed the report in review | Complete the bounded dynamic-semantics baseline without accepting the report, authorizing IDR-SRV-035, selecting or implementing draft Part 3, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-15 | IDR-SRV-034 Acceptance and IDR-SRV-035 Authorization | Accepted the DataStream semantic-contract model, exact-contract Observation identity/revision model, typed status and dynamic-property behavior, domain-event/source-health/CommandStatus separation, domain-time latest/current/snapshot selectors, schema/unit/nil/quality/provenance and correction/replay/batch rules, policy-first query and rebuildable projection model, publication/DDIL/command boundaries, and verification handoffs; authorized the bounded streaming and event publication strategy iteration | Preserve the single-topic review boundary while moving from committed dynamic-resource semantics into subscription, publication, snapshot/catch-up, ordering, replay, acknowledgement, backpressure, security, and draft Part 3 decision research | Glaux Project Lead |
| 2026-09-15 | IDR-SRV-035 Research Completion | Defined a durable transport-neutral publication core with atomic outbox, replay log, opaque policy-bound cursors, snapshot/catch-up, at-least-once delivery and idempotent consumers; selected HTTP query/change-feed plus SSE as the first live slice and MQTT 5 as the first broker adapter; classified NATS, Kafka, WebSocket, and MQTT 3.1.1 as prepared or conditional adapters; selected disabled-by-default outbound `glaux-csapi-part3-exp/0.1` support pinned to `6f529a15` with generated AsyncAPI 3.0, explicit deviations, no approved conformance claim, and versioned migration; refreshed the material Batch subclass delta and upstream register to Version 1.12; and placed the report in review | Complete the bounded streaming/event and final draft Part 3 decision without accepting the report, authorizing IDR-SRV-036, implementing Part 3, selecting a broker product, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-15 | IDR-SRV-035 Acceptance and IDR-SRV-036 Authorization | Accepted the durable transport-neutral publication core, atomic outbox/replay-log/delivery model, opaque policy-bound cursor and snapshot/catch-up contract, at-least-once and idempotent-consumer boundary, SSE-first and MQTT 5 broker sequence, conditional adapter choices, and disabled-by-default outbound `glaux-csapi-part3-exp/0.1` decision pinned to `6f529a15` with generated AsyncAPI 3.0, explicit deviations, truthful conformance boundary, and versioned migration; authorized the bounded Control Stream and Command lifecycle model iteration | Preserve the single-topic review boundary while moving from publication mechanics and withheld initial tasking channels into authoritative ControlStream, Command, status/result, lifecycle, transition, dispatch, cancellation, replay, safety, and publication semantics | Glaux Project Lead |
| 2026-09-15 | IDR-SRV-036 Research Completion | Defined a two-layer lifecycle with the exact nine CSAPI CommandStatus codes at the public boundary and multidimensional private orchestration state; established immutable versioned ControlStream/Command contracts, authoritative append-oriented status/result evidence and causal current-state projection, direct/gateway/brokered/simulated/manual dispatch, cancellation mediation, expiry/timeout/unknown-outcome mapping, SWE Common/SensorML payload validation, persistence/query/event/DDIL behavior, security/safety/audit hooks, the required 16-column lifecycle matrix, and downstream verification handoffs; placed the report in review | Complete the bounded command-lifecycle model without accepting the report, authorizing IDR-SRV-037, enabling tasking or inbound draft Part 3 channels, finalizing feasibility or safety policy, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-15 | IDR-SRV-036 Acceptance and IDR-SRV-037 Authorization | Accepted the two-layer public/private command lifecycle, exact nine-code CommandStatus boundary, immutable ControlStream/Command contract versions, append-oriented status/result evidence and causal projection, common dispatch-attempt model, mediated cancellation, expiry/timeout/unknown-outcome mapping, payload validation, DDIL/security/audit hooks, lifecycle matrix, and downstream handoffs; authorized the bounded Feasibility and Asynchronous Tasking Strategy iteration | Preserve the single-topic review boundary while moving from the neutral command lifecycle into feasibility request/result semantics, synchronous/asynchronous analysis, freshness, linkage, cancellation, policy, DDIL, and verification decisions | Glaux Project Lead |
| 2026-09-15 | IDR-SRV-037 Research Completion | Defined Feasibility as a durable advisory Command-shaped analysis exchange; resolved `201 Created` plus canonical `Location` and status `Content-Location` for synchronous/asynchronous creation without a jobs API or automatic HTTP `202`; established private durable synchronous staging, asynchronous PostgreSQL-backed work, exact public status flows, a schema-advertised result profile, evidence/freshness/cache/invalidation and non-reservation rules, validation/gate responsibility boundaries, Rust domain/adapter architecture, lease/fence/retry/recovery behavior, DDIL/security handoffs, and executable conformance scenarios; placed the report in review | Complete the bounded feasibility and asynchronous-tasking baseline without accepting the report, authorizing IDR-SRV-038, defining final authorization/safety policy, enabling tasking, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-15 | IDR-SRV-037 Acceptance and IDR-SRV-038 Authorization | Accepted the standard-resource/no-jobs model, `201 Created` response profile, private durable synchronous staging, exact synchronous/asynchronous status behavior, schema-advertised feasibility result and evidence model, freshness/cache/invalidation and non-reservation rules, validation/gate boundary, PostgreSQL-backed Rust task architecture, lease/fence/retry/recovery rules, DDIL/security handoffs, and requirement/scenario traceability; authorized the bounded Command Authorization, Safety, and Audit Strategy iteration | Preserve the single-topic review boundary while moving from neutral feasibility/task execution mechanics into command-specific authority, control ownership, approval, safety/interlock, cancellation, disclosure, and audit decisions | Glaux Project Lead |
| 2026-09-15 | IDR-SRV-038 Research Completion | Defined a deny-by-default subject/object/action/environment command decision plane; separated API permission from versioned CommandAuthorityGrant and conditional ControlAuthorityLease; established deterministic safety rules, approval/override bounds, lifecycle hooks, immediate pre-dispatch re-evaluation and single-use fenced dispatch tickets; integrated feasibility, validation, source/gateway trust, policy/redaction, direct/broker/manual/simulator paths, atomic append-oriented audit and event behavior; defined bounded locally verifiable DDIL authority, the required 15-column decision matrix, invariants, fixture/security/performance/interoperability suites, and downstream handoffs; placed the report in review | Complete Category F research without accepting the report, authorizing IDR-SRV-039, enabling live tasking or inbound draft Part 3 channels, selecting the enterprise identity/audit architecture, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-15 | IDR-SRV-038 Acceptance and IDR-SRV-039 Authorization | Accepted the deny-by-default command decision plane, scoped CommandAuthorityGrant and conditional ControlAuthorityLease, deterministic safety and approval/override model, immediate pre-dispatch re-evaluation, fenced single-use dispatch ticket, gateway/adapter boundaries, protected diagnostics/events, atomic append-oriented command audit, bounded DDIL authority, decision matrix, invariants, and verification handoffs; authorized the bounded Authentication, Authorization, and API Security Threat Model iteration | Close Category F while preserving the single-topic review boundary and moving into whole-API identity, authorization, transport, abuse, deployment, threat, and mitigation analysis without authorizing IDR-SRV-039A or implementation | Glaux Project Lead |
| 2026-09-15 | IDR-SRV-039 Research Completion | Defined a pluggable server-owned authentication and immutable security-context boundary; selected deny-by-default hybrid RBAC/scope/ABAC/resource-relationship authorization with query, property, link, cache, cursor, event and command enforcement; evaluated unsafe-dev, CI, demo, operational, publisher, service, federation, command-gateway and DDIL profiles; established JWT/opaque-token, OIDC/OAuth, mTLS/DPoP, browser/BFF, proxy, TLS, CORS, OpenAPI/schema, observability and offline-security guidance; applied trust-boundary, STRIDE and OWASP API threat analysis; completed the required 15-column control matrix, security fixtures/tests and downstream handoffs; placed the report in review | Open Category G research execution with a whole-server security baseline without accepting the report, authorizing IDR-SRV-039A, selecting a final IdP/PKI/policy engine/topology, enabling tasking or inbound draft Part 3 channels, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-15 | IDR-SRV-039 Acceptance and IDR-SRV-039A Authorization | Accepted the pluggable server-owned authentication and immutable security-context boundary, deny-by-default hybrid authorization and complete enforcement-point model, deployment and credential profiles, JWT/opaque-token and browser/BFF guidance, transport/proxy/CORS/OpenAPI controls, trust-boundary/STRIDE/OWASP threat analysis, 15-column control matrix, DDIL/federation behavior, observability rules, fixture/test obligations and downstream handoffs; authorized the bounded supplemental Zero-Trust Architecture Alignment and Enforcement Model iteration | Preserve the single-topic review boundary while moving from the whole-server authentication/authorization threat baseline into detailed zero-trust component placement, policy decision/enforcement topology, continuous evaluation, trust zones and degraded-operation alignment without authorizing IDR-SRV-040 or implementation | Glaux Project Lead |
| 2026-09-15 | IDR-SRV-039A Research Completion | Defined the bounded “ZTA-aligned Glaux Server enforcement” claim; crosswalked NIST tenets and DoD/CISA/NSA guidance; separated enterprise, deployment and server responsibilities; established control/data/management planes and smallest-useful granted-action zones; selected an embedded-first dual-capable deterministic criteria PE/runtime-PA model distinct from PAP authoring; defined provenance/freshness-qualified PIPs, the complete application PEP registry, three-state decision/obligation/evidence envelopes and continuous re-evaluation triggers; specialized data, source/workload, streaming, command, DDIL, telemetry and client behavior; defined seven deployment profiles, first/deferred scope, the required 14-column enforcement matrix, verification suites, evidence manifest, prototype backlog and downstream handoffs; placed the report in review | Complete the supplemental ZTA research without accepting it, authorizing IDR-SRV-040, claiming enterprise ZTA maturity/accreditation/ATO, selecting a vendor/IdP/policy engine/service mesh/topology, enabling tasking or implementing the server | Pending Glaux Project Lead review |
| 2026-09-15 | IDR-SRV-039A Acceptance and IDR-SRV-040 Authorization | Accepted the bounded ZTA-aligned server claim, NIST/DoD/CISA/NSA crosswalk, enterprise/deployment/server responsibility split, control/data/management planes, smallest-useful granted-action zones, embedded-first dual-capable criteria PE/runtime-PA model distinct from PAP, provenance/freshness-qualified PIPs, complete application PEP registry, three-state decision/obligation/evidence contract, continuous re-evaluation, data/source/workload/stream/command/DDIL/telemetry behavior, seven profiles, first/deferred scope, enforcement matrix, evidence manifest, prototype backlog and handoffs; authorized the bounded Policy, Releasability, and Cross-Boundary Access Constraints iteration | Preserve the single-topic review boundary while moving from ZTA enforcement placement into data/metadata policy authorities, markings, releasability, transformations, concealment and cross-boundary decisions without authorizing IDR-SRV-041 or implementation | Glaux Project Lead |
| 2026-09-15 | IDR-SRV-040 Research Completion | Defined policy-neutral immutable PolicyAssertion, versioned PolicyBinding, contextual DisclosureDecision and DerivedViewProvenance records; established authorized views before query/count/extent/latest/pagination, explicit existence concealment, schema-aware deterministic transforms, output bindings and safe errors; applied the model across metadata, SensorML/SWE, observations/status/events, streaming, commands, source/federation, OpenAPI/conformance, telemetry and audit; bounded DDIL/offline authority and fresh queued-send decisions; defined Glaux's non-CDS cross-boundary handoff, the required 14-column controlled-data matrix, invariants, synthetic fixtures, verification suites and downstream handoffs; placed the report in review | Complete the bounded policy/releasability research without accepting the report, authorizing IDR-SRV-041, choosing a production policy engine/regime/operational marking adapter, approving or implementing a cross-domain solution, changing the upstream-history register, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-15 | IDR-SRV-040 Acceptance and IDR-SRV-041 Authorization | Accepted the policy-neutral PolicyAssertion, PolicyBinding, DisclosureDecision and DerivedViewProvenance model; authorized-view query and inference controls; schema-aware deterministic transform and output-binding rules; metadata, dynamic-data, streaming, command, federation, DDIL, documentation, telemetry and audit applications; non-CDS cross-boundary responsibility boundary; controlled-data matrix, invariants, synthetic fixture strategy, verification suites and downstream handoffs; authorized the bounded Audit Logging and Accountability Strategy iteration | Preserve the single-topic review boundary while moving from disclosure decisions and obligations into authoritative audit event semantics, integrity, custody, access, export, failure, retention, accountability and verification without authorizing IDR-SRV-042 or implementation | Glaux Project Lead |
| 2026-09-15 | IDR-SRV-041 Research Completion | Defined a dedicated authoritative audit evidence plane distinct from logs, traces, metrics, alerts, provenance and domain events; established a versioned mandatory/capability/aggregate event taxonomy and deployment profiles; modeled actor, delegation, authority, source, executor and node evidence; defined the typed AuditEventV1 field matrix, phase/result/time/order/correlation semantics and prohibited-data controls; selected layered append/chain/checkpoint integrity with bounded claims, E0–E5 atomic/independent/pre-effect failure behavior, durable spool/gap/recovery, PostgreSQL journal/search/export/access/custody roles, lifecycle/DDIL/synchronization/observability handoffs, five worked scenarios and 39 falsifiable verification cases; placed the report in review | Complete the bounded audit/accountability research without accepting the report, authorizing IDR-SRV-042, selecting production retention/keys/SIEM/WORM/topology/legal claims, changing the upstream-history register, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-15 | IDR-SRV-041 Acceptance and IDR-SRV-042 Authorization | Accepted the dedicated authoritative audit evidence plane; mandatory/capability/aggregate taxonomy and deployment profiles; actor, delegation, authority, source, executor and node model; AuditEventV1 schema and phase/result/time/order/correlation semantics; prohibited-data and injection controls; bounded append/chain/checkpoint integrity claims; E0–E5 capture/failure rules; journal, spool, access, review, export and custody design; lifecycle, DDIL, synchronization and observability handoffs; five scenarios and 39 verification cases; authorized the bounded DDIL-Informed Server Semantics iteration | Preserve the single-topic review boundary while moving from audit behavior into connected, degraded, disconnected, intermittent and reconnect semantics without authorizing IDR-SRV-043 or implementation | Glaux Project Lead |
| 2026-09-15 | IDR-SRV-042 Research Completion | Defined DDIL as orthogonal connectivity, dependency, authority, data, time, synchronization and capacity conditions rather than a global flag; established connected/limited/intermittent/disconnected/local-only/recovering/unknown states and per-operation service postures; defined independent validity, current, freshness, last-known, cached, delayed, tentative, authority, unavailable, unknown and partial semantics plus RepresentationAssessmentV1; mapped all resource and operation families; fixed Observation/latest/status/source-health behavior, logical cursor/replay/resnapshot semantics, bounded offline security and source-trust rules, command/feasibility/unknown-outcome safeguards, standards-correct HTTP/problem/diagnostic/audit behavior, the exact IDR-SRV-043 semantic handoff, 44 verification cases and downstream ownership; placed the report in review | Complete bounded DDIL server-semantics research without accepting the report, authorizing IDR-SRV-043, choosing synchronization/conflict mechanics, topology, products, numeric thresholds, operational security packages, real command effects, or an OGC Part 3 conformance claim | Pending Glaux Project Lead review |
| 2026-09-15 | IDR-SRV-042 Acceptance and IDR-SRV-043 Authorization | Accepted the orthogonal DDIL connectivity/dependency/authority/data/time/synchronization/capacity model; connected, limited, intermittent, disconnected, local-only, recovering and unknown states; per-operation service postures; independent validity, current, freshness, last-known, cached, delayed, tentative, authority, unavailable, unknown and partial semantics; RepresentationAssessmentV1; resource/operation behavior matrices; dynamic-data, streaming, command, security, source-trust, response, diagnostic and audit rules; 44 verification cases; and the fixed semantic inputs for synchronization; authorized the bounded Server Synchronization and Conflict Handling Boundary iteration | Preserve the single-topic review boundary while moving from DDIL-visible semantics into exact synchronization ownership, exchange, identity, ordering, deduplication, gap, conflict, tombstone, policy and recovery boundaries without authorizing Category H or implementation | Glaux Project Lead |
| 2026-09-15 | IDR-SRV-043 Research Completion | Defined application synchronization as a provenance-, authority-, policy-, validation-, command-, and audit-aware receive/classify/apply pipeline distinct from database replication and broker delivery; established SyncEnvelopeV1, orthogonal session/candidate/outcome/continuity states, at-least-once transport with effectively-once local apply, scoped watermarks/gaps, resource inventory, identity/revision/tombstone rules, conflict taxonomy and SyncConflictV1; limited automatic resolution to proven exact, monotonic, compatible-immutable, or commutative cases; specialized policy/trust, federation, command and audit reconciliation; completed the required 15-column matrix, fixture/security/performance/interop suites and downstream handoffs; placed the report in review | Complete Category G research execution without accepting IDR-SRV-043, authorizing Category H, selecting topology/products/wire protocol/numeric horizons/conflict UI, enabling inbound Part 3 or command effects, or implementing the server | Pending Glaux Project Lead review |
| 2026-09-15 | IDR-SRV-043 Acceptance and IDR-SRV-044 Authorization | Accepted the application synchronization receive/classify/apply boundary distinct from infrastructure replication; SyncEnvelopeV1; orthogonal session/candidate/outcome/continuity states; at-least-once transport with effectively-once local application; scoped watermarks/gaps; resource, identity, revision and tombstone rules; conflict taxonomy and SyncConflictV1; constrained automatic-resolution allowlist; protected quarantine/review workflow; policy/trust, federation, command and audit reconciliation; required scenario matrix and verification handoffs; authorized the bounded Rust Implementation Language and Framework Strategy iteration | Close Category G while preserving the single-topic review boundary and moving into current Rust web, async, persistence, serialization, validation, OpenAPI, security, quality and ecosystem decisions without authorizing IDR-SRV-045 or implementation | Glaux Project Lead |
| 2026-09-15 | IDR-SRV-044 Research Completion | Confirmed Rust requirement fit without reopening language selection; selected the Rust 1.98.1/edition 2024 candidate baseline with MSRV 1.94, Axum/Tokio/Tower/Hyper, layered Serde domain/wire models, hybrid curated-plus-Utoipa OpenAPI, offline jsonschema validation, SQLx/PostgreSQL/PostGIS persistence, durable database workers and first-slice SSE; established command/security/configuration/telemetry seams, typed error/problem behavior, first-party unsafe prohibition, dependency/license/supply-chain policy, blocking CI gates, the required stack matrix, ten ordered proof gates and downstream handoffs; placed the report in review | Open Category H research with a bounded platform candidate without accepting IDR-SRV-044, authorizing IDR-SRV-045, creating implementation code, fixing final module/deployment/configuration/telemetry/migration topology, enabling live commands or claiming conformance | Pending Glaux Project Lead review |
| 2026-09-15 | IDR-SRV-044 Acceptance and IDR-SRV-045 Authorization | Accepted the Rust 1.98.1/edition 2024 candidate baseline and MSRV policy; Axum/Tokio/Tower/Hyper HTTP/runtime direction; layered source/wire/domain/persistence/projection models; curated-plus-Utoipa OpenAPI contract workflow; offline JSON Schema and staged XML/SWE validation; SQLx/PostgreSQL/PostGIS persistence; durable worker/SSE baseline; command, security, configuration and telemetry seams; typed error/problem strategy; unsafe/dependency/license/supply-chain posture; CI/test gates, stack matrix and ordered proof gates; authorized the bounded Service Architecture and Modularization Strategy iteration | Preserve the single-topic review boundary while moving from platform choices into exact modular-monolith, dependency, port/adapter, transaction, worker and deployment-unit boundaries without authorizing IDR-SRV-046 or implementation | Glaux Project Lead |
| 2026-09-15 | IDR-SRV-045 Research Completion | Selected a Cargo-workspace modular monolith with inward dependency direction, capability-oriented modules and ports/adapters at effect seams; defined domain/application/standards/validation/persistence/API/adapter/composition/test-support boundaries; kept one authoritative write core and one initial server artifact while allowing logical API, worker and admin roles; assigned semantic atomic transactions to persistence adapters, mandatory application-layer security/policy/audit enforcement, staged reusable validation, durable publication, command safety, DDIL and synchronization seams; specified the required 14-column service architecture matrix, workspace layout, phased vertical slices, ten proof gates, extraction criteria and downstream handoffs; placed the report in review | Complete the bounded internal architecture iteration without accepting IDR-SRV-045, authorizing IDR-SRV-046, selecting a deployment topology, introducing premature microservices or dynamic plugins, creating implementation code, or claiming conformance | Pending Glaux Project Lead review |
| 2026-09-15 | IDR-SRV-045 Acceptance and IDR-SRV-046 Authorization | Accepted the Cargo-workspace modular-monolith baseline; inward dependency direction; capability-oriented modules and ports/adapters at effect seams; one authoritative write core and initial server artifact with logical API, worker and admin roles; domain/application/standards/validation/persistence/API/adapter/composition/test-support boundaries; semantic atomic transaction ownership; application-layer security, policy, audit and validation enforcement; durable publication, command, DDIL and synchronization seams; workspace layout, phased vertical slices, extraction criteria and ten proof gates; authorized the bounded Reference Deployment Strategy iteration | Preserve the single-topic review boundary while moving from internal service architecture into local-development, CI, demonstration, evaluation and experimental-edge deployment shapes without authorizing IDR-SRV-047, production accreditation, infrastructure implementation or vendor lock-in | Glaux Project Lead |
| 2026-09-16 | IDR-SRV-046 Research Completion | Defined eleven purpose- and safety-specific deployment profiles; selected a Docker Compose reference baseline with one immutable Glaux image, PostgreSQL/PostGIS, explicit same-image migration and targeted fixture commands; kept brokers, identity/policy services, object storage, proxy, observability stack, simulator, publisher, harnesses and second nodes profile-gated; established multi-stage non-root/read-only image hardening, digest/SBOM/provenance release identity, deterministic CI lifecycle, TLS-fronted command-disabled public demo, explicit public-origin/proxy trust, secrets-as-files, separate startup/liveness/readiness semantics, isolated restore verification, DDIL and two-node synchronization simulations, an amd64-first/ARM proof gate, twelve implementation proofs and downstream handoffs; placed the report in review | Complete the bounded reference-deployment iteration without accepting IDR-SRV-046, authorizing IDR-SRV-047, creating deployment infrastructure, choosing an enterprise orchestrator/vendor topology, claiming ARM/tactical/production/accreditation readiness, enabling physical command effects or claiming conformance | Pending Glaux Project Lead review |
| 2026-09-16 | IDR-SRV-046 Acceptance and IDR-SRV-047 Authorization | Accepted the eleven deployment profiles; minimal Docker Compose baseline with one Glaux image, PostgreSQL/PostGIS, explicit migrations and targeted fixtures; profile-gated supporting services; hardened digest/SBOM/provenance release identity; deterministic CI; TLS-fronted command-disabled public demo; public-origin/proxy trust; secrets-as-files; separate health semantics; restore, DDIL and two-node synchronization proofs; amd64-first ARM gate and downstream handoffs; authorized the bounded Configuration, Secrets, and Environment Strategy iteration | Preserve the single-topic review boundary while moving into typed configuration sources, precedence, validation, secrets, trust material, reload and environment contracts without authorizing IDR-SRV-048 or infrastructure implementation | Glaux Project Lead |
| 2026-09-16 | IDR-SRV-047 Research Completion | Defined one versioned typed configuration contract loaded at the composition root; fixed eleven explicit profile safety contracts, TOML-first source precedence, deny-unknown typed validation, provenance, unsafe-combination rejection and restart-first reload boundaries; separated ordinary configuration from versioned secret references and providers; inventoried secret, trust and sensitive values; established mounted-file baseline, schema-classified redaction, rotation/key-ring behavior, test safety and canonical redacted effective-configuration fingerprints; completed the required 13-column configuration matrix, examples, CLI contract, twelve implementation proofs and downstream handoffs; placed the report in review | Complete the bounded configuration/secrets/environment iteration without accepting IDR-SRV-047, authorizing IDR-SRV-048, implementing configuration or infrastructure, selecting an enterprise secret manager, enabling physical command effects or inbound Part 3, or claiming conformance/accreditation | Pending Glaux Project Lead review |
| 2026-09-16 | IDR-SRV-047 Acceptance and IDR-SRV-048 Authorization | Accepted the versioned typed composition-root configuration contract; eleven explicit profile safety contracts; TOML-first deterministic precedence and provenance; deny-unknown, cross-field and unsafe-combination validation; restart-first reload boundary; separated secret references/providers; mounted-file baseline; classified inventory, redaction, rotation/key-ring and test-safety rules; canonical redacted effective configuration and fingerprint; configuration matrix, CLI/examples, implementation proofs and downstream handoffs; authorized the bounded Observability, Logs, Metrics, and Health Check Strategy iteration | Preserve the single-topic review boundary while moving into signal, health, readiness, diagnostic, correlation, redaction and telemetry-export contracts without authorizing IDR-SRV-049, implementation, vendor selection, operational SLOs or accreditation claims | Glaux Project Lead |
| 2026-09-16 | IDR-SRV-048 Research Completion | Defined distinct diagnostic log, access event, metric, trace, health, system-event, domain-event, audit, admin-diagnostic and conformance-evidence signals; selected typed tracing/JSON stdout, a bounded metrics facade with protected Prometheus/OpenMetrics scrape, W3C/OpenTelemetry correlation and optional OTLP traces; established route-template logging, strict field/label/cardinality/redaction controls, async durable span links, sampling boundaries, separate startup/liveness/role-aware readiness/dependency/service-posture semantics, private management diagnostics, eleven-profile behavior, functional instrumentation, the required 12-column matrix, twelve implementation proofs and downstream handoffs; placed the report in review | Complete the bounded observability iteration without accepting IDR-SRV-048, authorizing IDR-SRV-049, implementing telemetry infrastructure, selecting production backends, setting SLOs/alerts/retention, enabling physical commands or inbound Part 3, or claiming conformance/accreditation | Pending Glaux Project Lead review |
| 2026-09-16 | IDR-SRV-048 Acceptance and IDR-SRV-049 Authorization | Accepted the distinct signal taxonomy and non-substitution rules; typed tracing and JSON stdout model; bounded metric/label/cardinality registry and protected Prometheus/OpenMetrics scrape; W3C/OpenTelemetry correlation and optional OTLP traces; strict redaction and debug controls; async/durable span-link and sampling rules; separate startup, liveness, role-aware readiness, dependency and service-posture semantics; private diagnostics; eleven-profile behavior; functional matrix, twelve implementation proofs and handoffs; authorized the bounded Migration, Upgrade, Backup, and Restore Strategy iteration | Preserve the single-topic review boundary while moving into schema/artifact/config lifecycle, compatibility, upgrade/rollback, backup/restore, verification and recovery contracts without authorizing Category I, implementation, production retention/RPO/RTO, vendor selection or accreditation claims | Glaux Project Lead |
| 2026-09-16 | IDR-SRV-049 Research Completion | Defined forward-only immutable SQLx migrations, explicit same-image administration, a complete continuity inventory, expand/backfill/contract evolution and release compatibility manifests; established profile-gated bootstrap, fixtures and destructive resets; selected a coherent logical-database plus exact-artifact manifest as the first portable proof and handed physical backup/WAL/PITR to operational infrastructure; required isolated restore validation, command non-replay, event/outbox reconciliation and explicit audit/synchronization recovery lineage; completed the required 13-column matrix, twelve implementation proofs and downstream handoffs; placed the report in review | Complete Category H research execution without accepting IDR-SRV-049, authorizing Category I or IDR-SRV-050, implementing lifecycle tooling, selecting backup infrastructure/vendors or retention/RPO/RTO values, permitting arbitrary partial restore, enabling physical commands or inbound Part 3, or claiming disaster-recovery/accreditation readiness | Pending Glaux Project Lead review |
| 2026-09-16 | IDR-SRV-049 Acceptance and IDR-SRV-050 Authorization | Accepted forward-only immutable SQLx migrations and explicit same-image administration; continuity inventory and authority/rebuildability classes; expand/backfill/contract evolution and compatibility manifests; profile-gated bootstrap, fixtures and destructive resets; coherent logical-database plus exact-artifact backup manifests; operational physical/WAL/PITR handoff; isolated restore validation, fencing and lineage; command non-replay, event/outbox reconciliation and audit/synchronization recovery rules; continuity matrix, twelve implementation proofs and downstream handoffs; authorized the bounded Conformance Harness Strategy iteration | Close Category H while preserving the single-topic review boundary and moving into executable conformance-harness architecture, test selection, orchestration, evidence, result and reporting contracts without authorizing IDR-SRV-051, implementation, certification claims or later verification topics | Glaux Project Lead |
| 2026-09-16 | IDR-SRV-050 Research Completion | Selected a hybrid evidence system led by a standalone Rust black-box CLI in the Glaux workspace; bound the accepted 25-class, 233-requirement, five-recommendation and 240-ATS scope to independent wire models, deterministic targets, explicit applicability and six terminal outcomes; defined ConformanceCaseV1, CaseResultV1 and ConformanceRunV1 semantics, canonical JSON evidence, claim closure, fixture/profile safety, CI/local workflows and a 27-row 10-column coverage matrix; separated official OGC Features ETS, future CSAPI ETS, Schemathesis, security, performance and external-client lanes; completed twelve implementation proofs and downstream handoffs; placed the report in review | Complete the bounded conformance-harness iteration without accepting IDR-SRV-050, authorizing IDR-SRV-051, implementing the harness, claiming OGC certification, selecting operational evidence infrastructure, permitting uncontrolled external mutations or physical command effects, or executing later traceability/TDD/fixture/performance/security/interoperability topics | Pending Glaux Project Lead review |
| 2026-09-16 | IDR-SRV-050 Acceptance and IDR-SRV-051 Authorization | Accepted the hybrid evidence system led by a standalone Rust black-box CLI; exact 25-class, 233-requirement, five-recommendation and 240-ATS scope; independent wire models and deterministic targets; explicit applicability, six terminal results and flake semantics; ConformanceCaseV1, CaseResultV1 and ConformanceRunV1 contracts; canonical JSON evidence and closed-graph claim gating; fixture/profile safety, CI/local workflows, official/external lane separation, 10-column coverage matrix, twelve implementation proofs and downstream handoffs; authorized the bounded Requirement-to-Test Traceability Strategy iteration | Preserve the single-topic review boundary while moving into stable identifiers, trace records, coverage semantics, change impact, review, supersession and evidence linkage without authorizing IDR-SRV-052, implementing tooling, changing normative obligations or claiming certification/readiness | Glaux Project Lead |
| 2026-09-16 | IDR-SRV-051 Research Completion | Defined a normalized typed traceability graph with constrained YAML, JSON Schema Draft 2020-12 validation and canonical JSON hashing; preserved accepted/source IDs through immutable aliases and tombstones; established requirement, test, fixture, evidence, gate and deviation records with single-owner typed edges; separated lifecycle, applicability, disposition, implementation, execution, evidence and derived coverage states; defined digest-driven change impact and evidence freshness, repository layout, CI checks and generated reports; integrated harness, PR and downstream-topic contracts; completed the required 14-column matrix, twelve implementation proofs and explicit handoffs; placed the report in review | Complete the bounded requirement-to-test traceability iteration without accepting IDR-SRV-051, authorizing IDR-SRV-052, importing a registry, implementing traceability tooling, changing normative obligations, selecting later-topic mechanisms/datasets/thresholds/tools/clients, or claiming certification/readiness | Pending Glaux Project Lead review |
| 2026-09-16 | IDR-SRV-051 Acceptance and IDR-SRV-052 Authorization | Accepted the normalized typed graph; constrained YAML, JSON Schema Draft 2020-12 and canonical JSON boundary; immutable identifier, alias and tombstone rules; single-owner typed edges; orthogonal lifecycle, applicability, disposition, implementation, execution, evidence and derived-coverage states; digest-driven impact and evidence freshness; repository, CI, reporting, harness and PR contracts; 14-column matrix, twelve implementation proofs and downstream handoffs; authorized the bounded Rust Test-Driven Architecture and Multi-Layer Test Strategy iteration | Preserve the single-topic review boundary while moving into Rust test layering, ownership, orchestration, deterministic dependencies, async/concurrency testing, quality gates and developer workflow without authorizing IDR-SRV-053, implementing the server or traceability tooling, fixing fixture corpora/performance budgets/security depth/client matrices, or claiming certification/readiness | Glaux Project Lead |
| 2026-09-16 | IDR-SRV-052 Research Completion | Defined obligation-first boundary-conscious red-green-refactor; a 21-layer, 12-column test strategy spanning pure/domain, validation, application, adapter, router/listener, real PostgreSQL/PostGIS, migration, fixture/property/fuzz, async/stream, command/security/DDIL/sync, conformance, performance and interoperability evidence; selected pinned nextest as primary non-doctest CI runner with separate doctests and zero green-by-retry; fixed workspace/test-support, stable test-ID, deterministic clock/effect, real-dependency and exact-versus-semantic assertion rules; defined PR/nightly/manual/release gates, flake/quarantine/evidence contracts, twelve implementation proofs and downstream handoffs; placed the report in review | Complete the bounded Rust TDD and multi-layer testing iteration without accepting IDR-SRV-052, authorizing IDR-SRV-053, implementing code/CI/trace tooling, selecting the concrete fixture corpus, setting performance budgets/security assessment depth/interoperability clients, enabling physical command effects, or claiming conformance/readiness | Pending Glaux Project Lead review |
| 2026-09-16 | IDR-SRV-052 Acceptance and IDR-SRV-053 Authorization | Accepted obligation-first boundary-conscious red-green-refactor; the 21-layer, 12-column strategy; pinned nextest primary non-doctest CI with separate doctests and zero green-by-retry; workspace/test-support and stable test/assertion-ID boundaries; deterministic clock, effect and real-dependency rules; exact-versus-semantic assertion policy; PR/nightly/manual/release tiers; flake, quarantine and evidence semantics; command-safety boundary; twelve implementation proofs and downstream handoffs; authorized the bounded Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy iteration | Preserve the single-topic review boundary while moving into corpus taxonomy, provenance, licensing, manifests, canonical positive/negative/adversarial datasets, generation, golden review, sensitivity and lifecycle controls without authorizing IDR-SRV-054, implementing the server/test tooling, setting performance/security/interoperability gates, using controlled/operational data, or claiming conformance/readiness | Glaux Project Lead |
| 2026-09-16 | IDR-SRV-053 Research Completion | Defined one governed corpus registry with separate source, scenario, oracle and generated-evidence roles; independent origin/validity/scale/sensitivity/lifecycle taxonomy; constrained-YAML manifests with canonical JSON digests; immutable identifiers, provenance, licensing, transformation, public-closure and sensitivity rules; Git/generation/content-addressed storage boundaries; standards/resource, SensorML/SWE, query/error, ingestion/event, command/policy, DDIL/sync, demo and scale-data families; exact-versus-semantic golden and allowlisted normalization policy; deterministic generators, CI/review/drift controls, the required 15-column matrix, twelve implementation proofs and downstream handoffs; placed the report in review | Complete the bounded corpus-strategy iteration without accepting IDR-SRV-053, authorizing IDR-SRV-054, implementing the corpus/server/test tooling, choosing workload volumes or performance/security/interoperability gates, using controlled/operational data, enabling physical command effects or claiming conformance/readiness | Pending Glaux Project Lead review |
| 2026-09-16 | IDR-SRV-053 Acceptance and IDR-SRV-054 Authorization | Accepted the one-registry/four-role corpus architecture; independent taxonomy; constrained-YAML/canonical-JSON manifests; stable identity, provenance, licensing, sensitivity and public-closure rules; Git/generation/content-addressed storage boundaries; standards/resource and stateful scenario families; safe command and synthetic environmental-demo baselines; exact-versus-semantic golden and allowlisted-normalization policy; deterministic generation, CI/review/drift controls, 15-column matrix, twelve implementation proofs and downstream handoffs; authorized the bounded Performance, Load, Stress, and Streaming Test Strategy iteration | Preserve the single-topic review boundary while moving into workload classes, measurement semantics, tools, reproducible environments, threshold governance, API/ingestion/streaming/DDIL/synchronization performance and evidence without authorizing IDR-SRV-055, implementation, operational guarantees, real command effects or conformance/readiness claims | Glaux Project Lead |
| 2026-09-16 | IDR-SRV-054 Research Completion | Defined correctness-gated envelope-bound performance evidence; selected pinned k6 HTTP/open-arrival orchestration, independent Rust SSE probes, Criterion dedicated-runner microbenchmarks and PostgreSQL plan/runtime evidence; standardized offered/started/completed/goodput/drop, latency/clock, resource/backlog/recovery and PerformanceRunV1 semantics; established micro/S/M/L grids, workload archetypes, profiles, reset/safety controls, PR/nightly/manual/demo/RC/future-operational tiers, a provisional public-demo envelope, API/query/ingest/stream/command/policy/DDIL/sync strategies, the required 13-column matrix, twelve implementation proofs and downstream handoffs; placed the report in review | Complete the bounded performance iteration without accepting IDR-SRV-054, authorizing IDR-SRV-055, implementing load infrastructure, asserting production capacity/SLOs, stressing public endpoints, enabling physical commands or claiming conformance/readiness | Pending Glaux Project Lead review |
| 2026-09-16 | IDR-SRV-054 Acceptance and IDR-SRV-055 Authorization | Accepted correctness-gated envelope-bound performance evidence; pinned k6 HTTP/open-arrival orchestration, independent Rust SSE probes, Criterion dedicated-runner microbenchmarks and PostgreSQL plan/runtime evidence; offered/started/completed/goodput/drop, latency/clock, resource/backlog/recovery and PerformanceRunV1 semantics; micro/S/M/L grids, workload archetypes, profiles, reset/safety controls, PR/nightly/manual/demo/RC/future-operational tiers; provisional public-demo envelope; functional strategies, 13-column matrix, twelve implementation proofs and downstream handoffs; authorized the bounded Security, Authorization, and Command-Control Test Strategy iteration | Preserve the single-topic review boundary while moving into security test architecture, authorization/policy matrices, credential/trust, disclosure, abuse, streaming, audit and command-control verification without authorizing IDR-SRV-056, implementation, penetration against external/public systems, physical command effects or accreditation/readiness claims | Glaux Project Lead |
| 2026-09-16 | IDR-SRV-055 Research Completion | Defined a deny-by-default trace-driven security test architecture; separated synthetic policy contexts from real HTTP local-issuer/JWKS authentication proofs; established strict token/JWKS, route/action inventory, object/property/function, twin-world disclosure and all-surface canary tests; covered source/ingestion, invalid profiles, secrets, DDIL and streaming; fixed independent command gates, synthetic safety rules, single-use bound dispatch tickets, non-network simulator, lifecycle/fault/audit/effect reconciliation; selected PR/nightly/manual/RC/future tiers with supplemental ZAP, cargo-audit/deny, secret and fuzz tooling; completed the required 14-column matrix, SecurityTestResultV1, twelve implementation proofs and downstream handoffs; placed the report in review | Complete the bounded security and command-control test-strategy iteration without accepting IDR-SRV-055, authorizing IDR-SRV-056, implementing controls/tests, using real identities/policy/data/targets, actively scanning public/external systems, enabling physical command effects, selecting operational security products or claiming accreditation/readiness | Pending Glaux Project Lead review |
| 2026-09-16 | IDR-SRV-055 Acceptance and IDR-SRV-056 Authorization | Accepted the deny-by-default trace-driven security architecture; strict local-issuer/JWKS authentication proof; route/action inventory, object/property/function, twin-world disclosure and canary tests; source/ingestion, profile, secret, DDIL and streaming coverage; independent command gates, synthetic safety rules, bound single-use dispatch tickets, non-network simulator and audit/effect reconciliation; PR/nightly/manual/RC/future tiers, SecurityTestResultV1, 14-column matrix and twelve implementation proofs; authorized the bounded Interoperability Test Matrix for External CSAPI Clients iteration | Preserve the single-topic review boundary while moving into current client inventory, profile/capability negotiation, read/query/stream/write/tasking/error/security interoperation, evidence and compatibility tiers without authorizing IDR-SRV-057, implementation, changes to external clients/services, physical command effects, certification or readiness claims | Glaux Project Lead |
| 2026-09-16 | IDR-SRV-056 Research Completion | Defined a capability-qualified semantic-depth interoperability program; selected pinned OS4CSAPI TypeScript and OWSLib Python as independent mandatory client families, with CSAPI Explorer/browser, bounded TypeScript/Python OpenAPI-generated clients and a QGIS inherited-Features subset; classified current Glaux ecosystem repositories as future contract targets; established pinned advisory CS-Go/OSH/pygeoapi/SECD comparisons; separated execution, transport/structural/semantic/workflow depth and defect/capability attribution; covered discovery, proxy/CORS, OpenAPI/schema, resources/query/page, negotiation, SensorML/SWE, dynamic data, safe writes, SSE, experimental outbound Part 3, simulated tasking, security/policy/errors/DDIL and public demo; completed the required 15-column matrix, InteropTargetV1/InteropCaseResultV1/InteropRunV1, twelve implementation proofs and final-synthesis handoff; placed the report in review | Complete the bounded interoperability-matrix iteration without accepting IDR-SRV-056, authorizing IDR-SRV-057, executing tests, changing external projects, mutating uncontrolled services, enabling physical command effects, treating peers as standards authority or claiming conformance/certification/readiness | Pending Glaux Project Lead review |
| 2026-09-16 | IDR-SRV-056 Acceptance and IDR-SRV-057 Authorization | Accepted the capability-qualified semantic-depth interoperability program; pinned OS4CSAPI TypeScript and OWSLib Python mandatory families; CSAPI Explorer/browser, bounded generated-client and QGIS subset lanes; future Glaux component entry criteria; advisory peer comparisons; separate execution/depth/attribution semantics; full functional/profile coverage, 15-column matrix, InteropTargetV1/InteropCaseResultV1/InteropRunV1, twelve implementation proofs and feedback/retest workflow; authorized the final Glaux Server IDR synthesis iteration | Preserve the final-topic boundary while consolidating all accepted findings into one implementation-ready research synthesis without reopening accepted decisions, creating implementation code or roadmap commitments, authorizing physical command effects or inbound Part 3, setting operational guarantees, or claiming conformance/certification/accreditation/readiness | Glaux Project Lead |
| 2026-09-16 | IDR-SRV-057 Research Completion | Completed the 67-topic final Glaux Server IDR synthesis; audited all 66 accepted prerequisites with no exceptions; refreshed official standards-maintenance evidence; consolidated the standards, API, model, representation, persistence, dynamic-data, tasking, security, DDIL, Rust, deployment, continuity and verification baselines; reconciled material cross-topic conflicts; separated first, follow-on and deferred scope; and produced 34 traced recommendations, 15 proof candidates, risk and decision registers, 17 candidate work packages and downstream-document handoffs; placed the final overall report in review | Complete research execution and provide an acceptance-ready initial design baseline without accepting or closing the final report, creating implementation code, substituting for the Implementation Guide or Roadmap, authorizing physical command effects or inbound Part 3, fixing operational products/guarantees, or claiming conformance, certification, accreditation or operational readiness | Pending Glaux Project Lead review |
| 2026-09-16 | IDR-SRV-057 and Overall IDR Acceptance | Accepted the final synthesis, its complete 67-topic inventory, consolidated design baseline, scope classifications, conflict resolutions, proof candidates, risk and decision registers, candidate work packages, traced recommendations, downstream-artifact readiness assessment and explicit non-claims; completed IDR-SRV-057 and closed the Glaux Server Initial Design Research effort | Establish the accepted research baseline for subsequent Goal and Definition refresh, Implementation Guide, requirements/traceability registry, ADR and Roadmap work without itself authorizing implementation, physical command effects, inbound Part 3, operational products/guarantees, conformance, certification, accreditation or operational readiness | Glaux Project Lead |
| 2026-09-17 | Supplemental Part 4 Research Planning | Registered IDR-SRV-058 and prepared its focused research plan; preserved the completed 67-topic baseline and IDR-SRV-057 acceptance | User authorized plan/publication first, followed by separate research/report, synthesis-addendum, and planning-discussion iterations; no research execution or Part 4 implementation commitment in this iteration | Glaux Project Lead (planning authorization) |
| 2026-09-17 | IDR-SRV-058 Plan Acceptance and Research Authorization | The user's next `proceed` accepted the published plan for execution and authorized its research/report iteration | Continue the agreed staged supplement without changing the original accepted baseline or authorizing implementation | Glaux Project Lead |
| 2026-09-17 | IDR-SRV-058 Research Execution | Produced the draft Part 4 study report, pinned the official draft and peer sources, checked artifact/semantic gaps, assessed affected research and adoption options, and refreshed only relevant shared-history evidence | Report in review; preserve separate report acceptance, synthesis-addendum, and Goal/Guide discussion steps; no scope option adopted | Pending Glaux Project Lead report review |
| 2026-09-17 | IDR-SRV-058 Acceptance and Synthesis Addendum Authorization | The user's next `proceed` accepted the supplemental report and authorized integration through an addendum to the final synthesis | Accept research as decision material without selecting a Part 4 implementation option or changing the Goal/Guide | Glaux Project Lead |
| 2026-09-17 | Part 4 Synthesis Addendum Prepared | Added separately identified Part 4 findings, cross-topic qualifications, options and planning handoff; preserved the original report's completion and acceptance record | Next step is Goal/Guide discussion; no scope change, implementation or extra governance process introduced | Addendum prepared for Glaux Project Lead review |

---

## Progress Tracking

Original IDR baseline (unchanged by supplemental registration):

| Category | Topics | Plan Coverage | Report Coverage | Accepted Report Coverage | Status | Last Updated | Notes |
|---|---|---|---|---|---|---|---|
| A | IDR-SRV-001 to IDR-SRV-005 | Complete (5/5) | 5/5 | 5/5 | Research Complete | 2026-07-31 | IDR-SRV-001 through IDR-SRV-005 reports complete and accepted. |
| B | IDR-SRV-006 to IDR-SRV-014, IDR-SRV-010A, IDR-SRV-014A to IDR-SRV-014H | Complete (18/18) | 18/18 | 18/18 | Research Complete | 2026-08-31 | IDR-SRV-006 through IDR-SRV-014H reports are complete and accepted; no tracked Part 3 state changed during their execution. Later topic refreshes govern the current shared-register version. |
| C | IDR-SRV-015 to IDR-SRV-020 | Complete (6/6) | 6/6 | 6/6 | Research Complete | 2026-09-13 | IDR-SRV-015 through IDR-SRV-020 reports are complete and accepted. |
| D | IDR-SRV-021 to IDR-SRV-024 | Complete (4/4) | 4/4 | 4/4 | Research Complete | 2026-09-14 | IDR-SRV-021 through IDR-SRV-024 reports are complete and accepted. The shared upstream-history register is Version 1.10. |
| E | IDR-SRV-025 to IDR-SRV-030 | Complete (6/6) | 6/6 | 6/6 | Research Complete | 2026-09-14 | IDR-SRV-025 through IDR-SRV-030 reports are complete and accepted. |
| F | IDR-SRV-031 to IDR-SRV-038 | Complete (8/8) | 8/8 | 8/8 | Research Complete | 2026-09-15 | IDR-SRV-031 through IDR-SRV-038 are complete and accepted. Category G progress and authorization are recorded in the following row. The shared upstream-history register remains Version 1.12. |
| G | IDR-SRV-039, IDR-SRV-039A, IDR-SRV-040 to IDR-SRV-043 | Complete (6/6) | 6/6 | 6/6 | Research Complete | 2026-09-15 | IDR-SRV-039 through IDR-SRV-043 are complete and accepted. Category H progress and authorization are recorded in the following row. The shared upstream-history register remains Version 1.12. |
| H | IDR-SRV-044 to IDR-SRV-049 | Complete (6/6) | 6/6 | 6/6 | Research Complete | 2026-09-16 | IDR-SRV-044 through IDR-SRV-049 are complete and accepted. Category I progress and authorization are recorded in the following row. |
| I | IDR-SRV-050 to IDR-SRV-057 | Complete (8/8) | 8/8 | 8/8 | Research Complete | 2026-09-16 | IDR-SRV-050 through IDR-SRV-057 and the final overall report are complete and accepted. The Glaux Server Initial Design Research effort is closed. |

Supplemental progress is tracked separately from the completed category totals above:

| Topic | Plan Coverage | Report Coverage | Accepted Report Coverage | Status | Last Updated |
|---|---|---|---|---|---|
| IDR-SRV-058 | Complete (1/1) | 1/1 | 1/1 | Research complete and accepted; synthesis addendum prepared for review | 2026-09-17 |

---

## Risks and Constraints

- Normative specification interpretation may vary by clause and cross-reference depth.
- Upstream interoperability behaviors may diverge from strict conformance language.
- Mutable upstream issues and pull requests may be mistaken for approved obligations unless their status, release relationship, and authority class are preserved.
- Scope expansion risk if topic boundaries are not enforced at report time.

---

## References

- Glaux Governance: Research Planning Approach
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- Glaux Server Initial Planning Guidance
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/initial-planning-guidance.md
- OGC CSAPI planning exemplar (OS4CSAPI)
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/planning
- OGC API - Connected Systems upstream-history evidence register
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Evidence/ogc-connected-systems-upstream-history-register.md
- Official OGC API - Connected Systems repository
  - https://github.com/opengeospatial/ogcapi-connected-systems

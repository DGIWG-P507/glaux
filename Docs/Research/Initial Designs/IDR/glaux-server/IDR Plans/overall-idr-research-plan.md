# Glaux Server Overall IDR Research Plan

**Version:** 2.7<br>
**Date:** August 31, 2026<br>
**Status:** Draft<br>
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
7. Execution of `IDR-SRV-057`, including drafting the final overall IDR report, begins only after every other indexed topic report is complete and accepted, except for an exception approved and recorded under Rule 4.
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

**Total Topics:** 66

Throughout the Glaux Server IDR plan set, a numeric topic range includes every letter-suffixed topic inserted within that indexed range unless the text explicitly excludes it. For example, `IDR-SRV-001` through `IDR-SRV-040` includes `IDR-SRV-010A`, `IDR-SRV-014A` through `IDR-SRV-014G`, and `IDR-SRV-039A`.

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

## Topic Execution Order

Default research execution follows category dependencies in sequence:

1. Category A (IDR-SRV-001 through IDR-SRV-005)
2. Category B core behavior topics (IDR-SRV-006 through IDR-SRV-014, plus IDR-SRV-010A)
3. Existing implementation, smoke-test, interoperability, and lessons-learned studies (IDR-SRV-014A through IDR-SRV-014G)
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
- IDR-SRV-014A through IDR-SRV-014G capture existing implementation, smoke-test, interoperability, and community lessons early to inform model, behavior, conformance, validation, and test-strategy decisions.
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

---

## Progress Tracking

| Category | Topics | Plan Coverage | Report Coverage | Accepted Report Coverage | Status | Last Updated | Notes |
|---|---|---|---|---|---|---|---|
| A | IDR-SRV-001 to IDR-SRV-005 | Complete (5/5) | 5/5 | 5/5 | Research Complete | 2026-07-31 | IDR-SRV-001 through IDR-SRV-005 reports complete and accepted. |
| B | IDR-SRV-006 to IDR-SRV-014, IDR-SRV-010A, IDR-SRV-014A to IDR-SRV-014G | Complete (17/17) | 16/17 | 15/17 | Research In Progress | 2026-08-31 | IDR-SRV-006 through IDR-SRV-014E reports are complete and accepted; the shared upstream-history register is Version 1.8; IDR-SRV-014F is complete and in review; IDR-SRV-014G remains unstarted. |
| C | IDR-SRV-015 to IDR-SRV-020 | Complete (6/6) | 0/6 | 0/6 | Research Not Started | 2026-07-30 | |
| D | IDR-SRV-021 to IDR-SRV-024 | Complete (4/4) | 0/4 | 0/4 | Research Not Started | 2026-07-30 | |
| E | IDR-SRV-025 to IDR-SRV-030 | Complete (6/6) | 0/6 | 0/6 | Research Not Started | 2026-07-30 | |
| F | IDR-SRV-031 to IDR-SRV-038 | Complete (8/8) | 0/8 | 0/8 | Research Not Started | 2026-07-30 | |
| G | IDR-SRV-039, IDR-SRV-039A, IDR-SRV-040 to IDR-SRV-043 | Complete (6/6) | 0/6 | 0/6 | Research Not Started | 2026-07-30 | |
| H | IDR-SRV-044 to IDR-SRV-049 | Complete (6/6) | 0/6 | 0/6 | Research Not Started | 2026-07-30 | |
| I | IDR-SRV-050 to IDR-SRV-057 | Complete (8/8) | 0/8 | 0/8 | Research Not Started | 2026-07-30 | |

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

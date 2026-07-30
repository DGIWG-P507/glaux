# Section 025: Database and Persistence Architecture Options - Research Plan

**Status:** Planned  
**Last Updated:** June 10, 2026  
**Estimated Research Time:** 14-18 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-025-database-and-persistence-architecture-options-report.md`

---

## Usage Instructions

Before executing this plan, review the exemplar corpus and align the report to the proven standards for content nature, type, level, and detail:

- Full research-plans corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- Blueprint-first exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md
- Inventory/sourcing rigor exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/15-fixture-sourcing-organization.md
- End-state synthesis exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/38-testing-playbook-synthesis.md

This plan is limited to research planning. It does not execute the research and does not produce the research report.

---

## 1. Research Objective

This topic research must define the Glaux Server planning baseline for **database and persistence architecture options** across canonical CSAPI resources, SensorML/SWE representations, metadata documents, geospatial resources, time-series observations, status and event records, control-stream and command records, validation artifacts, audit records, configuration data, and DDIL synchronization state.

The research must answer:

- What persistence responsibilities must Glaux Server support to implement STANAG 4789 / AEP-4789 Volume II through OGC API - Connected Systems Parts 1 and 2, SensorML, SWE Common, and Glaux Server resource/domain-model decisions?
- Which data categories require relational, document, geospatial, time-series, event-log, graph-like relationship, object/blob, cache, queue, or hybrid persistence patterns?
- Which persistence technologies and architecture options should be evaluated for a Rust-based Glaux Server without prematurely selecting an implementation stack?
- How should Glaux Server preserve identity, lifecycle, relationships, temporal validity, freshness, status, events, SensorML/SWE source fidelity, semantic bindings, validation results, provenance, auditability, and conformance evidence?
- What persistence implications follow from high-rate ingestion, query/filter/sort/pagination requirements, streaming/event publication, command lifecycle, DDIL operation, federation, security/policy constraints, and testability?
- What downstream decisions should be deferred to specialized topics for geospatial storage, time-series observation storage, metadata/document storage, transaction/concurrency strategy, configuration/secrets, dynamic ingestion, command lifecycle, DDIL behavior, and conformance testing?

The output must be a database and persistence architecture options baseline with source anchors, data-category inventory, storage-pattern evaluation criteria, option analysis framework, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic begins Category E: Server Persistence and Query Architecture. It follows:

- Category C: `IDR-SRV-015` through `IDR-SRV-020`, which define canonical resources, identity, relationships, temporal semantics, registration/update semantics, and status/event concepts.
- Category D: `IDR-SRV-021` through `IDR-SRV-024`, which define SensorML, SWE Common, validation, units, observed properties, and semantic binding implications.

Those topics define what must be persisted and why. This topic evaluates persistence architecture options at a cross-cutting level before specialized storage topics refine geospatial storage, time-series storage, document storage, transactions, and configuration.

### Critical Constraint(s)

- Treat prior IDR findings, CSAPI Part 1 and Part 2, SensorML, SWE Common, OGC API - Features, AEP-4789 server responsibilities, and Glaux Server full-scope requirements as controlling.
- Do not pick a final database product solely in this topic. Evaluate options and recommend an architecture direction with explicit rationale and unresolved questions.
- Do not design detailed schemas here. Identify data categories, persistence patterns, constraints, and handoffs to specialized storage topics.
- Do not collapse all persistence into one store unless evidence supports that it can satisfy geospatial, time-series, document, relationship, validation, audit, and DDIL needs.
- Do not assume cloud-only services or managed infrastructure. Glaux Server must remain suitable for open-source, reproducible, and potentially disconnected or tactical-edge deployments.
- Do not ignore Rust ecosystem implications, migration strategy, testability, local development, CI, container deployment, and reproducible conformance testing.
- Do not conflate authoritative persistence, caches, materialized views, derived indexes, message queues, and temporary buffers.
- Keep the research bounded to Glaux Server behavior and server-side persistence contracts.

---

## 2. Research Questions

### Core Questions

1. What data categories must Glaux Server persist, cache, index, or derive?
2. What persistence architecture patterns are suitable for those data categories?
3. Which storage options should be evaluated for relational, geospatial, time-series, document, event, graph-like relationship, audit, validation, and cache workloads?
4. How should persistence architecture support CSAPI behavior, SensorML/SWE fidelity, semantic bindings, dynamic data, commands, DDIL, security, conformance, and testing?
5. What decisions should be made here versus deferred to specialized topics?

### Detailed Questions

#### Persistence Responsibility Baseline

- What persistence responsibilities follow from `IDR-SRV-015` canonical resource families?
- What identity, URI, lifecycle, and update-history responsibilities follow from `IDR-SRV-016` and `IDR-SRV-019`?
- What relationship and graph-like traversal responsibilities follow from `IDR-SRV-017`?
- What temporal, validity, freshness, status, and event responsibilities follow from `IDR-SRV-018` and `IDR-SRV-020`?
- What SensorML, SWE Common, validation, and semantic binding responsibilities follow from `IDR-SRV-021` through `IDR-SRV-024`?
- Which persistence responsibilities are normative, implementation-quality, operational, testability-driven, security-driven, or profile-specific?

#### Data Category Inventory

- Which data categories must be persisted or indexed:
  - systems,
  - procedures,
  - deployments,
  - sampling features,
  - properties,
  - datastream definitions,
  - observations,
  - status and dynamic properties,
  - system events,
  - control streams,
  - commands,
  - command status/history,
  - feasibility requests/responses,
  - SensorML source documents,
  - SWE Common structures,
  - OpenAPI/schema artifacts,
  - semantic/vocabulary mappings,
  - validation results,
  - links and relationships,
  - provenance,
  - audit records,
  - publisher/source state,
  - synchronization state,
  - configuration metadata,
  - caches/materialized views?
- Which categories are authoritative state, historical records, immutable records, append-only logs, derived views, caches, temporary buffers, or test fixtures?
- Which categories require long-term retention versus bounded retention?

#### Storage Pattern Options

- Which persistence patterns are candidates:
  - relational database,
  - relational plus JSON/document columns,
  - geospatial extension,
  - time-series extension,
  - document store,
  - object/blob storage,
  - event log,
  - queue/stream broker,
  - graph database,
  - embedded database,
  - file-backed storage,
  - hybrid architecture?
- What are the advantages and tradeoffs of each pattern for Glaux Server?
- Which patterns are necessary for full-scope behavior versus convenient implementation choices?
- Which patterns support local development, tactical-edge deployment, containerized deployment, and CI testability?

#### Relational and Hybrid Persistence

- What Glaux Server data fits naturally into relational tables?
- Which data benefits from relational integrity constraints, foreign keys, transactions, and query planning?
- Which data benefits from JSON/JSONB or document-style storage within a relational database?
- What relationship, lifecycle, provenance, and audit requirements favor relational/hybrid design?
- What risks follow from overusing JSON/document columns or over-normalizing standards representations?

#### Geospatial Persistence Implications

- What geospatial persistence needs follow from systems, deployments, sampling features, features of interest, positions, bounding boxes, trajectories, and spatial queries?
- Which geospatial needs should be addressed here versus deferred to `IDR-SRV-026`?
- What baseline storage capabilities must be considered now to avoid incompatible persistence choices?
- How should CRS, geometry types, spatial indexes, and feature collections influence the high-level persistence architecture?

#### Time-Series and Dynamic Data Implications

- What time-series needs follow from observations, status updates, dynamic properties, system events, command histories, feasibility checks, and ingestion records?
- Which time-series needs should be addressed here versus deferred to `IDR-SRV-027`?
- What high-level choices affect high-rate ingestion, retention, downsampling, latest-value queries, historical queries, and streaming?
- How should time-series storage interact with canonical resources, datastream definitions, SWE Common structures, semantic bindings, and status/event resources?

#### Metadata and Document Storage Implications

- What metadata/document storage needs follow from SensorML, SWE Common, OpenAPI artifacts, validation results, profiles, vocabularies, and source documents?
- Which document storage needs should be addressed here versus deferred to `IDR-SRV-028`?
- How should Glaux Server preserve source fidelity while supporting normalized query and validation?
- What storage patterns support versioned documents, provenance, validation warnings, policy filtering, and round-trip representation?

#### Relationship, Graph, and Linkage Persistence

- What relationship persistence needs follow from `IDR-SRV-017`?
- Are graph database patterns necessary, or can relationship requirements be satisfied through relational structures and indexes?
- Which relationships need materialization, reverse indexes, temporal validity, policy filtering, or derived views?
- How should relationship persistence support CSAPI link traversal, query filters, conformance tests, and client interoperability?

#### Transactions, Consistency, Idempotency, and Concurrency

- What transaction and consistency needs are visible at the high-level persistence architecture stage?
- Which needs should be deferred to `IDR-SRV-029`?
- How do registration, update, command submission, ingestion, event generation, validation, and audit records need to be coordinated?
- What persistence options support idempotent ingestion, duplicate detection, optimistic concurrency, replay handling, and DDIL synchronization?

#### DDIL, Federation, Offline, and Synchronization Implications

- What persistence support is needed for disconnected, degraded, intermittent, and limited-bandwidth operation?
- What synchronization state, source authority state, cache state, replay buffers, conflict markers, and last-known state must be stored?
- How should persistence architecture support federation without assuming constant connectivity?
- Which findings should be handed to `IDR-SRV-043`, ingestion topics, and interoperability topics?

#### Security, Policy, Audit, and Releasability Implications

- What persistence needs follow from authentication, authorization, policy, releasability, command safety, and audit requirements?
- Which data must be protected at rest?
- Which indexes or derived views may leak restricted relationships, command affordances, locations, capabilities, or sensitive observed properties?
- What audit and provenance records must be immutable or tamper-evident?
- Which findings should be handed to Category G and security-test topics?

#### Rust Implementation, Deployment, and Testing Implications

- What Rust ecosystem considerations affect persistence architecture options?
- Which database clients, migration tools, query builders, ORMs, async runtimes, connection pools, and test containers should be evaluated later?
- How should persistence architecture support integration tests, conformance tests, fixture loading, deterministic test data, and local developer environments?
- Which findings should be handed to implementation platform, TDD architecture, fixtures, and deployment topics?

#### Implementation and Interoperability Lessons

- What persistence, storage, query, document, time-series, or geospatial lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate persistence, query, relationship, status/event, observation, or schema-storage issues?
- What OS4CSAPI discussion lessons affect persistence strategy?
- Which implementation patterns should Glaux Server adopt, avoid, or investigate further?
- Which findings must be treated as implementation-specific rather than standards-derived?

---

## 3. Primary Resources

The future research report must analyze these sources directly.

### Project and Governance Sources

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md

### Prior IDR Topic Sources

- `IDR-SRV-001` through `IDR-SRV-024` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Controlling Standards and API Sources

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/

### Persistence and Architecture Candidate Sources

Use current official documentation and primary-source technical material when executing the research. Candidate sources include, but are not limited to:

- PostgreSQL documentation: https://www.postgresql.org/docs/
- PostGIS documentation: https://postgis.net/documentation/
- TimescaleDB documentation: https://docs.timescale.com/
- SQLite documentation: https://www.sqlite.org/docs.html
- DuckDB documentation: https://duckdb.org/docs/
- Apache Kafka documentation: https://kafka.apache.org/documentation/
- NATS documentation: https://docs.nats.io/
- Rust SQLx documentation: https://docs.rs/sqlx/
- Rust Diesel documentation: https://diesel.rs/
- Rust SeaORM documentation: https://www.sea-ql.org/SeaORM/
- OpenTelemetry documentation for telemetry/audit considerations: https://opentelemetry.io/docs/

These are candidates for evaluation, not assumed selections.

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, data-retention guidance, operational constraints, or standards-package annexes relevant to persistence, observations, status, system events, command/control, metadata, and DDIL operation

### Implementation and Lessons-Learned Sources

Use implementation-study outputs and source repositories as non-normative evidence:

- OSH / OpenSensorHub sources and `IDR-SRV-014A`
- Connected Systems Go sources and `IDR-SRV-014B`
- pygeoapi sources and `IDR-SRV-014C`
- SECD sources and `IDR-SRV-014D`
- OS4CSAPI Client Smoke Test Findings and `IDR-SRV-014E`
- SECD Interoperability Findings and `IDR-SRV-014F`
- OS4CSAPI Discussions Lessons-Learned and `IDR-SRV-014G`
- Category C findings from `IDR-SRV-015` through `IDR-SRV-020`
- Category D findings from `IDR-SRV-021` through `IDR-SRV-024`
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/

---

## 4. Supporting Resources

Use these sources to interpret project context, downstream dependencies, expected research depth, and report style.

- DGIWG P5.07 Glaux main repository: https://github.com/DGIWG-P507/glaux
- Glaux project website: https://dgiwg-p507.github.io/glaux/
- DGIWG P5.07 GitHub organization: https://github.com/DGIWG-P507
- Glaux Server repository, if available or created: https://github.com/DGIWG-P507/glaux-server
- OS4CSAPI testing research-plan corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

---

## 5. Research Methodology

### Phase 1: Source Collection and Persistence Requirement Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for persistence architecture research.

**Tasks:**

1. Gather prior IDR reports, standards sources, implementation studies, persistence candidate documentation, and project architecture sources.
2. Extract persistence requirements from each prior topic and classify them by resource family, data category, query need, retention need, transaction need, and security need.
3. Define inventory fields:
   - data category,
   - related resource family,
   - authoritative/derived/cache/log classification,
   - expected volume/velocity,
   - temporal/geospatial/document/relationship characteristics,
   - query/index needs,
   - retention needs,
   - security/policy needs,
   - candidate storage pattern,
   - downstream handoff.
4. Define evaluation criteria:
   - standards alignment,
   - query capability,
   - geospatial capability,
   - time-series capability,
   - document fidelity,
   - transaction support,
   - DDIL/offline support,
   - Rust ecosystem maturity,
   - deployment simplicity,
   - open-source suitability,
   - testability,
   - operational complexity.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation evidence, and technology documentation.

**Expected Output:** Persistence requirement extraction framework and evaluation rubric.

### Phase 2: Data Category and Persistence Responsibility Inventory (3-4 hours)

**Objective:** Determine what Glaux Server must persist, cache, index, derive, or retain.

**Tasks:**

1. Inventory canonical resource data from `IDR-SRV-015` through `IDR-SRV-020`.
2. Inventory representation, validation, semantic, and metadata data from `IDR-SRV-021` through `IDR-SRV-024`.
3. Inventory dynamic data, observation, status, event, command, feasibility, provenance, audit, synchronization, and configuration data categories.
4. Classify each category as authoritative state, append-only history, event log, document, relationship index, cache/materialized view, temporary buffer, or fixture/test data.
5. Identify retention, deletion, archival, and policy implications.
6. Build a data-category inventory matrix.

**Expected Output:** Glaux Server data-category and persistence-responsibility matrix.

### Phase 3: Storage Pattern and Architecture Option Analysis (3-4 hours)

**Objective:** Evaluate candidate persistence patterns without prematurely selecting an implementation.

**Tasks:**

1. Analyze relational/hybrid relational-document storage patterns.
2. Analyze geospatial persistence patterns and handoffs to `IDR-SRV-026`.
3. Analyze time-series persistence patterns and handoffs to `IDR-SRV-027`.
4. Analyze document/object storage patterns and handoffs to `IDR-SRV-028`.
5. Analyze event-log, queue, stream, and audit-log patterns.
6. Analyze graph-like relationship persistence patterns and alternatives.
7. Analyze embedded/local storage options for development, tests, edge deployments, and DDIL contexts.
8. Compare patterns against the evaluation rubric.

**Expected Output:** Persistence architecture option comparison matrix.

### Phase 4: Cross-Cutting Persistence Implication Analysis (2.5-3.5 hours)

**Objective:** Identify cross-cutting persistence concerns that shape downstream topics.

**Tasks:**

1. Analyze transaction, consistency, idempotency, and concurrency implications for `IDR-SRV-029`.
2. Analyze schema migration, data migration, versioning, and backward compatibility implications.
3. Analyze cache, materialized view, synchronization, offline/DDIL, and federation implications.
4. Analyze security, policy, encryption, audit, provenance, and releasability implications.
5. Analyze performance, high-rate ingestion, retention, indexing, partitioning, and operational complexity implications.
6. Analyze Rust implementation, local development, CI, fixture, and conformance-test implications.

**Expected Output:** Cross-cutting persistence implication matrix.

### Phase 5: Implementation Lessons and Risk Analysis (2-3 hours)

**Objective:** Incorporate implementation evidence and identify risks.

**Tasks:**

1. Review persistence-related findings from OSH, Connected Systems Go, pygeoapi, SECD, smoke tests, interoperability findings, and OS4CSAPI discussions.
2. Identify implementation patterns to adopt, avoid, or investigate further.
3. Identify risks associated with single-store, multi-store, database-specific, cloud-dependent, edge-only, or overly abstracted persistence designs.
4. Identify unresolved questions requiring prototype validation, benchmark testing, or downstream topic analysis.
5. Identify decision points for future architecture and implementation phases.

**Expected Output:** Implementation lessons, risk register, and unresolved decision matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert persistence architecture research into a decision-usable baseline.

**Tasks:**

1. Consolidate data-category inventory, storage-pattern option analysis, cross-cutting implications, implementation lessons, and risk analysis.
2. Produce recommended persistence architecture direction(s) with rationale and unresolved questions.
3. Identify sequencing for specialized persistence topics.
4. Produce downstream handoff matrix for Categories E, F, G, H, and I.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Glaux Server data categories and persistence responsibilities are identified with source anchors.
- [ ] Authoritative state, historical records, event logs, documents, relationships, caches, materialized views, synchronization state, audit records, and fixtures are distinguished.
- [ ] Candidate storage patterns and architecture options are evaluated against explicit criteria.
- [ ] Geospatial, time-series, document, event, relationship, transaction, DDIL, security, and testing implications are documented.
- [ ] Rust ecosystem, deployment, local development, CI, and conformance-test implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Database and Persistence Architecture Options Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-025-database-and-persistence-architecture-options-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Persistence requirement extraction methodology
5. Glaux Server data-category inventory
6. Persistence responsibility and retention matrix
7. Storage pattern evaluation criteria
8. Relational/hybrid persistence option findings
9. Geospatial persistence implications
10. Time-series persistence implications
11. Metadata/document storage implications
12. Event-log, audit, and relationship persistence implications
13. DDIL, synchronization, cache, and federation implications
14. Transaction, consistency, migration, and concurrency implications
15. Security, policy, provenance, and audit implications
16. Rust ecosystem, deployment, CI, fixture, and conformance-test implications
17. Implementation lessons and risk analysis
18. Downstream topic handoff matrix
19. Recommendations
20. Risks, constraints, and open questions
21. Validation against this plan's success criteria
22. References

The data-category matrix should include, at minimum:

- Data category
- Related resource family
- Source topic / source anchor
- Authoritative / derived / cache / log / document classification
- Expected volume/velocity
- Temporal characteristics
- Geospatial characteristics
- Document fidelity needs
- Relationship/index needs
- Transaction/consistency needs
- Retention/archival needs
- Security/policy needs
- Candidate storage pattern(s)
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-024` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, relevant OGC schemas/OpenAPI artifacts, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Current candidate storage technology documentation must be reachable when the research is executed.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-026: Geospatial Storage and Query Strategy`
- `IDR-SRV-027: Time-Series Observation Storage Strategy`
- `IDR-SRV-028: Metadata and Document Storage Strategy`
- `IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy`
- `IDR-SRV-030: Data Lifecycle, Retention, Archival, and Deletion Strategy`
- `IDR-SRV-047: Configuration, Secrets, and Environment Strategy`
- `IDR-SRV-031: Server Write and Ingestion Model`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-035: Streaming and Event Publication Strategy`
- `IDR-SRV-041: Audit Logging and Accountability Strategy`
- `IDR-SRV-042: DDIL-Informed Server Semantics`
- `IDR-SRV-043: Server Synchronization and Conflict Handling Boundary`
- `IDR-SRV-045: Service Architecture and Modularization Strategy`
- `IDR-SRV-048: Observability, Logs, Metrics, and Health Check Strategy`
- `IDR-SRV-052: Rust Test-Driven Architecture and Multi-Layer Test Strategy`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
- `IDR-SRV-054: Performance, Load, Stress, and Streaming Test Strategy`
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

- This topic evaluates persistence architecture options, not final detailed database schemas.
- A hybrid architecture may be necessary, but that must be justified by evidence and not assumed.
- The report should avoid product lock-in unless a technology is strongly supported by requirements, open-source suitability, deployment constraints, and testing needs.
- Open question: Can a relational/geospatial/time-series hybrid satisfy most Glaux Server needs without adding unnecessary operational complexity?
- Open question: Which data categories require immutable append-only storage?
- Open question: Which caches or materialized views are necessary for CSAPI client performance?
- Open question: Which persistence choices best support disconnected or tactical-edge operation?
- Open question: Which persistence decisions should be validated through prototypes or benchmarks?
- Risk: Selecting a single store too early could constrain geospatial, time-series, document, and DDIL behavior.
- Risk: Selecting too many stores could increase operational complexity, test burden, and deployment friction.
- Risk: Cloud-dependent persistence choices could undermine open-source reproducibility and tactical-edge suitability.
- Risk: Poor persistence boundaries could weaken validation, auditability, conformance, and interoperability.

---

## References

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/
- PostgreSQL documentation: https://www.postgresql.org/docs/
- PostGIS documentation: https://postgis.net/documentation/
- TimescaleDB documentation: https://docs.timescale.com/
- SQLite documentation: https://www.sqlite.org/docs.html
- DuckDB documentation: https://duckdb.org/docs/
- Apache Kafka documentation: https://kafka.apache.org/documentation/
- NATS documentation: https://docs.nats.io/
- Rust SQLx documentation: https://docs.rs/sqlx/
- Rust Diesel documentation: https://diesel.rs/
- Rust SeaORM documentation: https://www.sea-ql.org/SeaORM/
- OpenTelemetry documentation: https://opentelemetry.io/docs/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

# Section 027: Time-Series Observation Storage Strategy - Research Plan

**Status:** Planned  
**Last Updated:** June 10, 2026  
**Estimated Research Time:** 14-18 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-027-time-series-observation-storage-strategy-report.md`

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

This topic research must define the Glaux Server planning baseline for **time-series observation storage strategy** across CSAPI observations, datastreams, status updates, dynamic properties, system events, command status history, feasibility history, latest values, historical queries, ingestion records, replay, retention, and high-rate dynamic data.

The research must answer:

- What time-series and temporal-record data must Glaux Server store, index, retain, derive, summarize, and expose to satisfy CSAPI Part 2, AEP-4789 server responsibilities, SensorML/SWE Common structures, and Glaux Server resource/domain-model decisions?
- Which resource families produce time-indexed records, append-only streams, current-state snapshots, event histories, command histories, or derived latest-value views?
- How should Glaux Server distinguish observations, status updates, dynamic properties, system events, command status records, feasibility evaluations, ingestion records, source telemetry, audit records, and materialized latest-value caches?
- What storage and indexing strategy should support phenomenon time, result time, ingestion time, publication time, event time, command lifecycle time, valid time, freshness time, and transaction time?
- How should time-series storage support CSAPI query/filter/sort/pagination behavior, spatial-temporal filtering, semantic filtering, latest-value access, retention, replay, downsampling, compression, validation, and conformance testing?
- Which time-series storage options should be evaluated for a Rust-based, open-source, reproducible Glaux Server that may operate in connected, tactical-edge, and DDIL conditions?
- How should time-series storage interact with geospatial storage, metadata/document storage, transaction/concurrency behavior, ingestion normalization, streaming/event publication, command lifecycle, DDIL synchronization, security/policy filtering, performance testing, fixtures, and interoperability testing?

The output must be a time-series observation storage strategy baseline with source anchors, time-series data-category inventory, resource-family temporal storage requirements, indexing and query implications, storage-option evaluation criteria, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows:

- `IDR-SRV-025: Database and Persistence Architecture Options`
- `IDR-SRV-026: Geospatial Storage and Query Strategy`

`IDR-SRV-025` defines the cross-cutting persistence architecture evaluation framework. `IDR-SRV-026` specializes geospatial storage and query behavior. This topic specializes storage for time-indexed observations, status updates, events, dynamic data, and command histories. It must precede metadata/document storage, transaction/concurrency strategy, dynamic ingestion, datastream/observation semantics, streaming/event publication, command lifecycle, DDIL behavior, performance/load testing, fixture planning, and external-client interoperability testing.

### Critical Constraint(s)

- Treat OGC API - Connected Systems Part 2, CSAPI Part 1, OGC API - Features, SensorML 3.0, SWE Common 3.0, AEP-4789 Volume II adoption context, and earlier IDR findings as controlling.
- Do not design the final database schema here. Identify time-series data categories, storage patterns, indexing needs, query semantics, retention implications, and downstream handoffs.
- Do not collapse all time-indexed records into “observations.” Distinguish observations, status updates, system events, command status history, feasibility records, ingestion metadata, and audit records.
- Do not collapse phenomenon time, result time, ingestion time, publication time, transaction time, event time, command time, and freshness time into one timestamp.
- Do not finalize ingestion normalization here. Identify storage requirements and hand ingestion behavior to `IDR-SRV-031`.
- Do not finalize datastream/observation API semantics here. Identify storage implications and hand detailed API behavior to `IDR-SRV-034`.
- Do not finalize streaming/event publication here. Identify storage/backfill/replay needs and hand streaming design to `IDR-SRV-035`.
- Do not assume cloud-only time-series infrastructure. Evaluate open-source, reproducible, local, containerized, and disconnected/tactical-edge deployment implications.
- Do not ignore policy and releasability risks for historical data, precise location-time records, command histories, and status/event timelines.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What time-series and time-indexed data categories must Glaux Server persist, index, cache, summarize, or derive?
2. Which resource families produce observations, status records, event records, command histories, feasibility histories, latest-value caches, or ingestion telemetry?
3. What time fields, indexes, retention rules, query patterns, and consistency assumptions are required?
4. Which time-series storage options best fit Glaux Server requirements and downstream persistence architecture?
5. What downstream implications follow for ingestion, dynamic data, streaming, DDIL behavior, command lifecycle, security, fixtures, performance, conformance, and interoperability?

### Detailed Questions

#### Standards and Time-Series Source Baseline

- What observation, datastream, dynamic data, control stream, command, feasibility, status, and event storage concepts are defined or implied by CSAPI Part 2?
- What feature-resource context from CSAPI Part 1 is needed to interpret time-series records?
- What OGC API - Features temporal and spatial-temporal query behavior is relevant?
- What SensorML process, input/output, deployment, provenance, and system-description concepts affect time-series storage?
- What SWE Common data component, result-structure, unit, nil-value, quality, and encoding concepts affect time-series storage?
- What time-series responsibilities are implied by STANAG 4789 / AEP-4789 Volume II server responsibilities?
- Which time-series concepts appear in implementation studies, smoke tests, SECD interoperability findings, or OS4CSAPI discussions?

#### Time-Series Data Category Inventory

- What data categories require time-indexed storage:
  - observations,
  - observation results,
  - latest values,
  - datastream samples,
  - status updates,
  - dynamic properties,
  - system events,
  - availability records,
  - command status records,
  - command execution history,
  - feasibility evaluations,
  - control stream interaction records,
  - ingestion batches,
  - source/publisher telemetry,
  - synchronization records,
  - validation results,
  - audit records,
  - derived summaries,
  - downsampled aggregates?
- Which categories are authoritative records, append-only histories, current-state snapshots, derived caches, materialized views, temporary buffers, or audit trails?
- Which categories require long-term retention, bounded retention, archival, summarization, or deletion?

#### Resource-Family Temporal Storage Requirements

- What temporal storage requirements apply to:
  - datastreams,
  - observations,
  - status and dynamic properties,
  - system events,
  - systems,
  - deployments,
  - sampling features,
  - properties,
  - control streams,
  - commands,
  - command status,
  - feasibility resources,
  - ingestion sources,
  - publisher/simulator adapters?
- Which resource families require current-state/latest-value views?
- Which resource families require append-only event or observation history?
- Which resource families require temporal validity intervals?
- Which resource families require spatial-temporal correlation?
- Which resource families require policy-filtered or generalized historical records?

#### Time Field and Temporal Semantics Strategy

- Which time fields must be stored for each time-indexed category:
  - phenomenon time,
  - result time,
  - ingestion time,
  - publication time,
  - transaction time,
  - event time,
  - command issue/acceptance/execution/completion time,
  - feasibility evaluation time,
  - valid time,
  - freshness/staleness time,
  - synchronization time?
- Which fields are required, optional, derived, or source-provided?
- How should out-of-order records, delayed ingestion, replayed records, clock skew, duplicated records, and future timestamps be handled?
- How should timestamp precision, timezone normalization, interval representation, and temporal uncertainty be handled?

#### Query, Filtering, Sorting, Pagination, and Latest-Value Needs

- What time-series query patterns must Glaux Server support:
  - observations by datastream and time range,
  - latest value by datastream,
  - status history by system,
  - event history by system or resource,
  - command status history,
  - spatial-temporal observation queries,
  - semantic-temporal queries,
  - freshness-filtered queries,
  - source/publisher history,
  - backfill/replay queries?
- How should time filters interact with spatial filters, semantic filters, sorting, pagination, selection, response limits, content negotiation, and cache behavior?
- What default query behavior should apply when no time range is supplied?
- What query behavior is required by CSAPI and what is implementation support?
- Which advanced query capabilities should be deferred?

#### Storage and Indexing Options

- Which time-series storage options should be evaluated:
  - relational tables with time indexes,
  - PostgreSQL partitioning,
  - TimescaleDB hypertables,
  - event-log tables,
  - document/JSONB time records,
  - columnar/analytics stores,
  - embedded/local time-series storage,
  - object/blob storage for batch archives,
  - hybrid patterns?
- Which indexes are relevant:
  - datastream/time,
  - resource/time,
  - time/space,
  - property/time,
  - semantic/time,
  - command/time,
  - source/time,
  - event-type/time,
  - retention partitions,
  - latest-value materialized indexes?
- How should storage options support high-rate ingestion, historical queries, latest-value access, streaming backfill, retention, compression, local development, CI, container deployment, and tactical-edge operation?
- What tradeoffs exist between standards behavior, performance, operational complexity, portability, and testability?

#### Retention, Aggregation, Summarization, and Archival

- What retention requirements apply to observations, status updates, events, command histories, feasibility records, ingestion telemetry, and audit records?
- Which data may be downsampled or summarized?
- Which data must be retained losslessly?
- How should archival records remain discoverable or queryable?
- How should retention interact with conformance, reproducibility, audit, policy, and DDIL synchronization?
- Which retention decisions should be configurable by deployment, collection, source, datastream, or profile?

#### Ingestion, Normalization, and Validation Interaction

- What storage requirements support ingestion normalization from publishers, simulators, legacy feeds, and external sources?
- How should raw ingestion payloads, normalized records, validation results, rejected records, quarantined records, and provenance be stored?
- How should SWE Common result structures and semantic bindings guide storage layout and validation?
- How should duplicate detection, idempotency, replay, and out-of-order data be supported?
- Which findings should be handed to `IDR-SRV-031`, `IDR-SRV-034`, `IDR-SRV-023`, and `IDR-SRV-029`?

#### Streaming, Event Publication, and Backfill Interaction

- How should time-series storage support live streaming, event publication, replay, subscription backfill, reconnect recovery, and catch-up after disconnection?
- Which records must be stored before publication?
- Which records can be transient?
- How should event-ordering, cursor/offset, resume tokens, and last-known position be represented conceptually?
- Which findings should be handed to `IDR-SRV-035` and DDIL topics?

#### Command, Control Stream, and Feasibility Interaction

- How should command status histories be stored?
- How should command requests, accepted/rejected states, execution states, results, failures, cancellations, timeouts, and audit records be stored?
- How should feasibility evaluations and command validation results be stored?
- Which command-related histories require immutability or tamper-evidence?
- Which command histories are operationally sensitive?
- Which findings should be handed to `IDR-SRV-036`, `IDR-SRV-037`, `IDR-SRV-038`, and security-test topics?

#### Geospatial and Semantic Interaction

- How should observations, status records, and events with locations interact with geospatial storage from `IDR-SRV-026`?
- How should spatial-temporal indexes be considered?
- How should observed properties, units, semantic bindings, and SWE Common structures affect time-series storage design?
- How should time-series storage support semantic filters and spatial filters without duplicating authoritative metadata?
- Which findings should be handed to `IDR-SRV-024`, `IDR-SRV-026`, and `IDR-SRV-034`?

#### DDIL, Synchronization, and Federation Implications

- What time-series storage support is needed for disconnected, degraded, intermittent, and limited-bandwidth operation?
- How should Glaux Server store replay queues, synchronization cursors, source offsets, conflict markers, stale records, late records, and duplicate records?
- How should federated sources and cached external records be distinguished from authoritative local records?
- How should synchronization affect latest-value views and event histories?
- Which findings should be handed to `IDR-SRV-043` and federation topics?

#### Security, Policy, Audit, and Releasability Implications

- Which time-series records may reveal sensitive operational patterns, locations, capabilities, command histories, or source behavior?
- How should policy and releasability affect historical queries, latest-value views, event histories, command histories, and spatial-temporal filters?
- Which records require encryption, audit, immutability, tamper-evidence, retention controls, or deletion controls?
- How should Glaux Server avoid leaking restricted data through aggregates, counts, latest-value availability, or query timing?
- Which findings should be handed to Category G and security-test topics?

#### Fixtures, Performance, Conformance, and Interoperability

- What time-series fixtures are needed:
  - small deterministic observation sets,
  - high-rate datastreams,
  - status histories,
  - system event histories,
  - command lifecycle histories,
  - out-of-order data,
  - duplicated data,
  - stale data,
  - spatial-temporal data,
  - semantic-temporal data,
  - DDIL replay/sync data?
- What conformance tests are needed for observation retrieval, time filtering, latest values, pagination, sorting, response validation, and error behavior?
- What performance/load/stress tests are needed for ingestion rate, query latency, retention partitions, backfill, streaming, and latest-value queries?
- What interoperability tests are needed for CSAPI Explorer, OS4CSAPI clients, webapp/mobile clients, and external CSAPI clients?
- Which findings should be handed to `IDR-SRV-050`, `IDR-SRV-053`, `IDR-SRV-054`, and `IDR-SRV-056`?

#### Implementation and Interoperability Lessons

- What time-series storage, observation, status, event, and command-history lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate observation storage, latest-value, time filtering, pagination, status/event, or command-history issues?
- What OS4CSAPI discussion lessons affect time-series storage strategy?
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

- `IDR-SRV-001` through `IDR-SRV-026` research reports, once complete:
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
- OGC schemas: https://schemas.opengis.net/
- RFC 3339 - Date and Time on the Internet: Timestamps: https://www.rfc-editor.org/rfc/rfc3339

### Time-Series Storage and Query Candidate Sources

Use current official documentation and primary-source technical material when executing the research. Candidate sources include, but are not limited to:

- PostgreSQL documentation: https://www.postgresql.org/docs/
- PostgreSQL partitioning documentation: https://www.postgresql.org/docs/current/ddl-partitioning.html
- TimescaleDB documentation: https://docs.timescale.com/
- InfluxDB documentation: https://docs.influxdata.com/
- QuestDB documentation: https://questdb.io/docs/
- ClickHouse documentation: https://clickhouse.com/docs/
- DuckDB documentation: https://duckdb.org/docs/
- SQLite documentation: https://www.sqlite.org/docs.html
- Apache Kafka documentation: https://kafka.apache.org/documentation/
- NATS documentation: https://docs.nats.io/
- Rust SQLx documentation: https://docs.rs/sqlx/
- Rust time crate documentation: https://docs.rs/time/
- Rust chrono crate documentation: https://docs.rs/chrono/

These are candidates for evaluation, not assumed selections.

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, observation-retention guidance, operational constraints, or standards-package annexes relevant to dynamic data, observations, status, system events, command/control, DDIL operation, and NATO JISR sensor records

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
- Database and Persistence Architecture Options findings from `IDR-SRV-025`
- Geospatial Storage and Query Strategy findings from `IDR-SRV-026`
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

### Phase 1: Source Collection and Time-Series Requirement Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for time-series storage research.

**Tasks:**

1. Gather prior IDR reports, standards sources, implementation studies, time-series candidate documentation, and project architecture sources.
2. Extract time-series and time-indexed storage requirements from each prior topic and classify them by resource family, data category, time field, query need, retention need, and security need.
3. Define inventory fields:
   - data category,
   - related resource family,
   - authoritative/derived/cache/log classification,
   - append-only/current-state classification,
   - expected volume/velocity,
   - time fields,
   - spatial/semantic links,
   - query/index needs,
   - retention/archival needs,
   - security/policy needs,
   - candidate storage pattern,
   - downstream handoff.
4. Define evaluation criteria:
   - standards alignment,
   - temporal query capability,
   - latest-value support,
   - ingestion throughput,
   - spatial-temporal query support,
   - retention/partitioning support,
   - compression/downsampling support,
   - Rust ecosystem maturity,
   - deployment simplicity,
   - edge/offline suitability,
   - testability,
   - operational complexity.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation evidence, and technology documentation.

**Expected Output:** Time-series requirement extraction framework and evaluation rubric.

### Phase 2: Time-Series Data Category and Resource-Family Inventory (3-4 hours)

**Objective:** Determine what time-indexed data Glaux Server must store, index, derive, or expose.

**Tasks:**

1. Inventory time-series requirements from CSAPI Part 2, CSAPI Part 1, SensorML, SWE Common, and AEP-4789 material.
2. Inventory time-series requirements from `IDR-SRV-015` through `IDR-SRV-026`.
3. Classify time-indexed data categories by resource family, time fields, source authority, append-only/current-state behavior, retention needs, and policy sensitivity.
4. Identify latest-value, history, backfill, replay, and streaming needs.
5. Build a time-series data-category matrix.

**Expected Output:** Glaux Server time-series data-category and resource-family matrix.

### Phase 3: Temporal Semantics, Query, Indexing, and Retention Analysis (3-4 hours)

**Objective:** Define time-series behavior and storage requirements.

**Tasks:**

1. Analyze time fields and timestamp/interval handling by data category.
2. Analyze observation, status, event, command, feasibility, ingestion, synchronization, and audit timelines.
3. Analyze time filtering, spatial-temporal filtering, semantic-temporal filtering, sorting, pagination, and latest-value behavior.
4. Analyze retention, archival, partitioning, downsampling, summarization, compression, deletion, and audit requirements.
5. Analyze duplicate detection, idempotency, replay, out-of-order data, late-arriving data, and clock-skew implications.
6. Identify unresolved questions requiring prototype validation or benchmark testing.

**Expected Output:** Temporal/query/indexing/retention strategy matrix.

### Phase 4: Storage Option and Architecture Analysis (2.5-3.5 hours)

**Objective:** Evaluate time-series persistence options in the context of the high-level persistence architecture.

**Tasks:**

1. Evaluate relational time-indexed table patterns.
2. Evaluate PostgreSQL partitioning and TimescaleDB-style patterns.
3. Evaluate event-log and queue/stream-supported patterns.
4. Evaluate embedded/local storage and lightweight edge-friendly options.
5. Evaluate analytics-oriented or columnar options for downstream summarization and reporting.
6. Evaluate interaction with geospatial storage, semantic metadata, and document storage.
7. Compare options against the evaluation rubric and `IDR-SRV-025` findings.

**Expected Output:** Time-series storage option comparison matrix.

### Phase 5: Security, DDIL, Fixtures, Performance, and Interoperability Implication Analysis (2.5-3 hours)

**Objective:** Prepare time-series findings for downstream implementation and verification work.

**Tasks:**

1. Identify policy, releasability, historical-query, aggregation, and audit needs.
2. Identify DDIL, cache, last-known-state, synchronization, replay, conflict, and reconnect implications.
3. Identify fixture and golden-file needs for observations, status histories, events, command histories, out-of-order data, duplicates, stale data, and replay.
4. Identify performance/load/stress tests for ingestion rate, historical query latency, latest-value access, retention partitions, streaming backfill, and spatial-temporal filters.
5. Identify interoperability tests for CSAPI Explorer, OS4CSAPI clients, webapp/mobile clients, and external CSAPI clients.
6. Map findings to Category E, F, G, H, and I topics.

**Expected Output:** Time-series downstream implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert time-series storage research into a decision-usable baseline.

**Tasks:**

1. Consolidate time-series data-category inventory, temporal/query/indexing guidance, retention guidance, storage option analysis, and downstream implications.
2. Produce recommended time-series storage strategy direction(s) with rationale and unresolved questions.
3. Identify sequencing for downstream ingestion, streaming, command, DDIL, performance, fixture, and interoperability topics.
4. Produce downstream handoff matrix.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Time-series and time-indexed data categories are identified with source anchors.
- [ ] Observations, status updates, dynamic properties, system events, command histories, feasibility histories, ingestion records, latest-value views, audit records, and derived summaries are distinguished.
- [ ] Phenomenon time, result time, ingestion time, publication time, transaction time, event time, command time, valid time, synchronization time, and freshness time implications are documented.
- [ ] Time filtering, latest-value access, spatial-temporal query, semantic-temporal query, sorting, pagination, retention, archival, replay, and backfill implications are documented.
- [ ] Candidate time-series storage options are evaluated against explicit criteria.
- [ ] Security, policy, DDIL, fixture, performance, conformance, and interoperability implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Time-Series Observation Storage Strategy Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-027-time-series-observation-storage-strategy-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Time-series requirement extraction methodology
5. Time-series data-category inventory
6. Resource-family temporal storage matrix
7. Time field and temporal semantics findings
8. Query, filtering, sorting, pagination, latest-value, replay, and backfill findings
9. Indexing, partitioning, retention, archival, compression, and summarization findings
10. Time-series storage option evaluation
11. Observation, status, event, command, feasibility, and ingestion storage implications
12. Spatial-temporal, semantic-temporal, and dynamic-location implications
13. DDIL, cache, synchronization, and federation implications
14. Security, policy, releasability, and audit implications
15. Fixture, conformance, performance, and interoperability test implications
16. Downstream topic handoff matrix
17. Recommendations
18. Risks, constraints, and open questions
19. Validation against this plan's success criteria
20. References

The time-series data-category matrix should include, at minimum:

- Data category
- Related resource family
- Source topic / source anchor
- Authoritative/derived/cache/log classification
- Append-only/current-state classification
- Expected volume/velocity
- Required time fields
- Spatial/semantic links
- Query/index needs
- Latest-value/backfill/replay needs
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
- `IDR-SRV-001` through `IDR-SRV-026` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, relevant OGC schemas/OpenAPI artifacts, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Current candidate time-series technology documentation must be reachable when the research is executed.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-028: Metadata and Document Storage Strategy`
- `IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy`
- `IDR-SRV-031: Server Write and Ingestion Model`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-035: Streaming and Event Publication Strategy`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-037: Feasibility and Asynchronous Tasking Strategy`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- `IDR-SRV-042: DDIL-Informed Server Semantics`
- `IDR-SRV-043: Server Synchronization and Conflict Handling Boundary`
- `IDR-SRV-045: Service Architecture and Modularization Strategy`
- `IDR-SRV-048: Observability, Logs, Metrics, and Health Check Strategy`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
- `IDR-SRV-054: Performance, Load, Stress, and Streaming Test Strategy`
- `IDR-SRV-056: Interoperability Test Matrix for External CSAPI Clients`
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

- This topic defines time-series observation storage strategy, not final detailed tables, schemas, or ingestion code.
- Time-series storage must support CSAPI dynamic data behavior while preserving source provenance, validation evidence, and operational audit needs.
- Implementation-study findings are useful but must not override standards-derived time-series semantics.
- Open question: Which time-series records require append-only immutability versus update-in-place correction?
- Open question: Which latest-value views should be materialized versus computed at query time?
- Open question: Which retention and downsampling policies should be configurable by deployment, source, collection, datastream, or profile?
- Open question: Which storage choice best supports both high-rate ingestion and tactical-edge reproducibility?
- Open question: Which time-series fixtures should become canonical conformance/interoperability test data?
- Risk: Treating all dynamic records as simple observations could blur status, event, command, and audit semantics.
- Risk: Poor partitioning/indexing decisions could make CSAPI time filtering and latest-value access impractical.
- Risk: Cloud-dependent time-series storage could undermine open-source reproducibility and tactical-edge operation.
- Risk: Historical time-series data may reveal sensitive operational patterns if policy filtering and audit are not planned.

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
- OGC schemas: https://schemas.opengis.net/
- RFC 3339 - Date and Time on the Internet: Timestamps: https://www.rfc-editor.org/rfc/rfc3339
- PostgreSQL documentation: https://www.postgresql.org/docs/
- PostgreSQL partitioning documentation: https://www.postgresql.org/docs/current/ddl-partitioning.html
- TimescaleDB documentation: https://docs.timescale.com/
- InfluxDB documentation: https://docs.influxdata.com/
- QuestDB documentation: https://questdb.io/docs/
- ClickHouse documentation: https://clickhouse.com/docs/
- DuckDB documentation: https://duckdb.org/docs/
- SQLite documentation: https://www.sqlite.org/docs.html
- Apache Kafka documentation: https://kafka.apache.org/documentation/
- NATS documentation: https://docs.nats.io/
- Rust SQLx documentation: https://docs.rs/sqlx/
- Rust time crate documentation: https://docs.rs/time/
- Rust chrono crate documentation: https://docs.rs/chrono/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

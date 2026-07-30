# Section 026: Geospatial Storage and Query Strategy - Research Plan

**Status:** Planned  
**Last Updated:** June 10, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-026-geospatial-storage-and-query-strategy-report.md`

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

This topic research must define the Glaux Server planning baseline for **geospatial storage and query strategy** across CSAPI feature resources, OGC API - Features behavior, systems, deployments, sampling features, features of interest, SensorML positions, platform/sensor locations, datastream locations, observation-associated locations, bounding boxes, coordinate reference systems, spatial indexes, spatial filters, map/client interoperability, and tactical-edge use.

The research must answer:

- What geospatial data must Glaux Server store, index, preserve, derive, and expose to satisfy CSAPI, OGC API - Features, SensorML, SWE Common, STANAG 4789 / AEP-4789 server responsibilities, and Glaux Server resource-model decisions?
- Which resource families require geometry, bounding boxes, positions, trajectories, areas, footprints, sampling geometries, deployment extents, feature-of-interest geometries, or derived spatial metadata?
- How should Glaux Server distinguish authoritative location, current location, historical location, deployment location, observation location, platform trajectory, sensor orientation, area of operation, sampling geometry, and derived map extent?
- What geospatial query behavior must Glaux Server support for bounding boxes, spatial intersection, containment, proximity, CRS handling, collection extents, filtering, sorting, pagination, and client navigation?
- Which geospatial storage and indexing architecture options should be evaluated for a Rust-based, open-source, standards-aligned Glaux Server?
- How should geospatial storage interact with time-series observation storage, metadata/document storage, relationship modeling, semantic binding, DDIL synchronization, security/policy filtering, conformance testing, and external-client interoperability?
- What downstream implications follow for time-series storage, ingestion, dynamic data, deployment topology, performance testing, fixtures, conformance harnesses, and interoperability testing?

The output must be a geospatial storage and query strategy baseline with source anchors, geospatial data-category inventory, resource-family spatial requirements, CRS and geometry guidance, query and indexing implications, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows `IDR-SRV-025: Database and Persistence Architecture Options`.

`IDR-SRV-025` identifies cross-cutting persistence responsibilities and storage-pattern options. This topic specializes those findings for geospatial storage and query behavior, which is central to OGC API - Features inheritance, CSAPI feature resources, SensorML positions, deployments, sampling features, and NATO JISR sensor discovery. It must precede detailed time-series storage, dynamic-data ingestion, geospatial fixture design, performance/load testing, and interoperability testing with map-oriented clients and OGC API tooling.

### Critical Constraint(s)

- Treat OGC API - Connected Systems Part 1 and Part 2, OGC API - Features, SensorML 3.0, SWE Common 3.0, GeoJSON, CRS guidance, AEP-4789 Volume II adoption context, and earlier IDR findings as controlling.
- Do not design the full database schema here. Identify spatial data categories, indexing needs, CRS/geometry handling, query semantics, and handoffs.
- Do not assume all spatial data is static point geometry. Distinguish static positions, moving platforms, deployments, sampling areas, observation locations, footprints, trajectories, and derived extents.
- Do not collapse spatial validity and temporal validity. Identify time-dependent spatial requirements and hand detailed time-series/temporal storage to `IDR-SRV-027`.
- Do not assume all clients use only GeoJSON. Preserve CSAPI/OGC behavior while identifying GeoJSON and map-client expectations.
- Do not ignore policy and releasability risks for locations, trajectories, sensor positions, areas of operation, and command/control extents.
- Do not assume cloud-only geospatial services. Evaluate open-source, reproducible, local, containerized, and disconnected/tactical-edge deployment implications.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What geospatial data categories must Glaux Server store, index, derive, or expose?
2. Which resource families require geometry, bounding boxes, locations, trajectories, footprints, deployment extents, or spatial metadata?
3. What spatial query behavior must Glaux Server support for CSAPI, OGC API - Features, SensorML, and client interoperability?
4. Which geospatial storage and indexing options best fit Glaux Server requirements and downstream persistence architecture?
5. What downstream implications follow for time-series storage, ingestion, DDIL behavior, policy, fixtures, conformance, performance, and interoperability?

### Detailed Questions

#### Standards and Geospatial Source Baseline

- What geospatial behavior is inherited from OGC API - Features?
- What geometry, bounding box, extent, spatial filter, collection, feature, and CRS concepts are defined or implied by CSAPI Part 1?
- What spatial concepts are defined or implied by CSAPI Part 2 for observations, datastreams, status updates, events, commands, and dynamic data?
- What SensorML position, location, orientation, frame, system/platform, and deployment concepts are relevant?
- What SWE Common spatial, vector, coordinate, or value-structure concepts are relevant?
- What geospatial concepts are relevant from STANAG 4789 / AEP-4789 Volume II server responsibilities?
- Which geospatial concepts appear in implementation studies, smoke tests, SECD interoperability findings, or OS4CSAPI discussions?

#### Geospatial Data Category Inventory

- What geospatial data categories must Glaux Server handle:
  - system location,
  - platform location,
  - sensor location,
  - procedure location,
  - deployment location,
  - sampling feature geometry,
  - feature of interest geometry,
  - datastream spatial extent,
  - observation location,
  - status update location,
  - system event location,
  - control stream applicability area,
  - command target area,
  - feasibility area,
  - source/publisher location,
  - collection extent,
  - derived bounding boxes,
  - trajectories,
  - footprints,
  - areas of operation?
- Which categories are authoritative, derived, historical, current-state, cached, policy-filtered, or optional?
- Which categories require spatial precision metadata, uncertainty, confidence, provenance, or source authority?

#### Resource-Family Spatial Requirements

- What spatial requirements apply to:
  - systems,
  - procedures,
  - deployments,
  - sampling features,
  - properties,
  - datastreams,
  - observations,
  - status and dynamic properties,
  - system events,
  - control streams,
  - commands,
  - command status,
  - feasibility resources,
  - links and relationships,
  - SensorML source documents,
  - SWE Common data components?
- Which resource families need first-class geometry?
- Which resource families only need derived spatial extents?
- Which resource families need time-varying geometry?
- Which resource families need policy-filtered or generalized geometry?
- Which spatial fields must be queryable?

#### Geometry, CRS, and Coordinate Handling

- What geometry types should be supported initially and over the full-scope baseline?
- How should Glaux Server handle points, lines, polygons, multipolygons, bounding boxes, 3D coordinates, altitude/depth, orientation, trajectories, footprints, and areas?
- What CRS assumptions are inherited from GeoJSON and OGC API - Features?
- What CRS requirements or ambiguities arise from SensorML, SWE Common, and AEP-4789?
- How should Glaux Server handle CRS transformation, CRS validation, coordinate order, vertical reference, spatial precision, and spatial uncertainty?
- Which CRS decisions should be implementation requirements versus profile constraints or warnings?

#### Spatial Query and Filtering Semantics

- What spatial queries must Glaux Server support:
  - bounding box,
  - intersects,
  - contains,
  - within,
  - nearest/proximity,
  - collection extent,
  - spatial plus temporal filters,
  - spatial plus semantic filters,
  - spatial plus status/freshness filters?
- Which queries are required by CSAPI/OGC API behavior?
- Which queries are useful for NATO JISR operational discovery?
- Which queries are necessary for CSAPI Explorer, webapp, mobile client, and external OGC tooling?
- How should spatial filters interact with sorting, pagination, selection, response limits, content negotiation, and query performance?
- Which advanced spatial query features should be deferred?

#### Spatial Indexing and Storage Options

- Which geospatial storage options should be evaluated:
  - PostGIS,
  - GeoJSON in JSON/JSONB fields with indexes,
  - SQLite/SpatiaLite,
  - embedded/local stores,
  - search indexes,
  - hybrid relational-geospatial models,
  - external geospatial services?
- Which spatial indexes are relevant:
  - GiST,
  - SP-GiST,
  - R-tree,
  - BRIN for spatial-temporal partitioning,
  - materialized bounding boxes,
  - derived extents?
- How should storage options support local development, CI, containers, tactical-edge deployments, DDIL operation, and performance testing?
- What tradeoffs exist between precision, performance, operational complexity, portability, and standards behavior?

#### Spatial-Temporal Interaction

- How should spatial data interact with temporal validity, observation time, deployment time, status time, event time, and command lifecycle time?
- Which resource locations are static versus time-varying?
- How should moving platforms and trajectories be represented?
- How should latest known location differ from historical location?
- How should out-of-order location updates, delayed observations, stale status, and cached/federated positions be handled?
- Which findings should be handed to `IDR-SRV-027`, `IDR-SRV-031`, `IDR-SRV-034`, `IDR-SRV-042`, and `IDR-SRV-043`?

#### SensorML, SWE Common, and Semantic Integration

- How should SensorML positions, coordinate frames, and platform/system locations map to geospatial storage?
- How should SWE Common vectors, positions, quantities, and coordinate-related components be stored or validated?
- How should semantic bindings identify coordinate systems, observed properties, controlled properties, feature types, and location-related metadata?
- Which spatial metadata should be preserved in source documents versus normalized into queryable fields?
- Which findings should be handed to `IDR-SRV-028` and semantic/validation topics?

#### Security, Policy, and Releasability Implications

- Which geospatial information may be sensitive:
  - sensor location,
  - platform trajectory,
  - deployment extent,
  - command target area,
  - feasibility area,
  - observation location,
  - source/publisher location,
  - areas of operation?
- How should policy and releasability affect spatial precision, generalization, suppression, query results, bounding boxes, and relationship links?
- How should Glaux Server avoid leaking sensitive resource existence through spatial queries?
- Which spatial audit records are needed?
- Which findings should be handed to `IDR-SRV-039`, `IDR-SRV-040`, and `IDR-SRV-055`?

#### Performance, DDIL, and Operational Constraints

- What spatial performance needs are expected for tactical dashboards, CSAPI Explorer, mobile clients, and external OGC clients?
- How should spatial queries behave under limited bandwidth or degraded connectivity?
- Which spatial indexes, materialized extents, simplified geometries, or cached query results might support DDIL operation?
- How should data volume, observation velocity, trajectory history, and event history affect spatial storage strategy?
- Which findings should be handed to `IDR-SRV-042`, `IDR-SRV-043`, and `IDR-SRV-054`?

#### Fixtures, Conformance, and Interoperability

- What geospatial fixtures are needed:
  - point systems,
  - deployed sensors,
  - polygon sampling areas,
  - moving platforms,
  - feature-of-interest geometries,
  - datastream extents,
  - observation locations,
  - spatial-temporal event history,
  - generalized/policy-filtered geometries?
- What conformance tests are needed for geometry validity, bbox behavior, spatial query behavior, CRS assumptions, pagination, sorting, filtering, and content negotiation?
- What interoperability tests are needed for CSAPI Explorer, webapp/mobile clients, QGIS/OGC API clients, and external CSAPI clients?
- Which findings should be handed to `IDR-SRV-050`, `IDR-SRV-053`, `IDR-SRV-054`, and `IDR-SRV-056`?

#### Implementation and Interoperability Lessons

- What geospatial storage and query lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate geometry, bbox, CRS, collection extent, spatial query, or link-navigation issues?
- What OS4CSAPI discussion lessons affect geospatial storage and query strategy?
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

- `IDR-SRV-001` through `IDR-SRV-025` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Controlling Standards and API Sources

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- GeoJSON RFC 7946: https://www.rfc-editor.org/rfc/rfc7946
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OGC CRS resources: https://www.ogc.org/standards/crs/
- OGC schemas: https://schemas.opengis.net/

### Geospatial Storage and Query Candidate Sources

Use current official documentation and primary-source technical material when executing the research. Candidate sources include, but are not limited to:

- PostgreSQL documentation: https://www.postgresql.org/docs/
- PostGIS documentation: https://postgis.net/documentation/
- SQLite documentation: https://www.sqlite.org/docs.html
- SpatiaLite documentation: https://www.gaia-gis.it/fossil/libspatialite/index
- DuckDB spatial extension documentation: https://duckdb.org/docs/extensions/spatial/overview
- Rust geo crate documentation: https://docs.rs/geo/
- Rust geo-types crate documentation: https://docs.rs/geo-types/
- Rust geojson crate documentation: https://docs.rs/geojson/
- GDAL documentation, if relevant to import/export evaluation: https://gdal.org/en/stable/

These are candidates for evaluation, not assumed selections.

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, spatial metadata guidance, operational constraints, or standards-package annexes relevant to geospatial resources, sensor locations, sampling features, deployments, and NATO JISR sensor discovery

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

### Phase 1: Source Collection and Spatial Requirement Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for geospatial storage and query research.

**Tasks:**

1. Gather prior IDR reports, standards sources, implementation studies, geospatial candidate documentation, and project architecture sources.
2. Extract spatial data requirements from prior topics and classify them by resource family, geometry type, temporal relationship, query need, indexing need, and security need.
3. Define inventory fields:
   - data category,
   - related resource family,
   - geometry type,
   - authoritative/derived/cache classification,
   - static/time-varying classification,
   - CRS/coordinate needs,
   - query/index needs,
   - temporal interaction,
   - policy/releasability needs,
   - candidate storage pattern,
   - downstream handoff.
4. Define evaluation criteria:
   - standards alignment,
   - geometry support,
   - CRS behavior,
   - spatial query capability,
   - spatial-temporal query capability,
   - indexing performance,
   - Rust ecosystem maturity,
   - deployment simplicity,
   - edge/offline suitability,
   - testability,
   - operational complexity.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation evidence, and technology documentation.

**Expected Output:** Spatial requirement extraction framework and evaluation rubric.

### Phase 2: Geospatial Data Category and Resource-Family Inventory (3-4 hours)

**Objective:** Determine what spatial data Glaux Server must store, index, derive, or expose.

**Tasks:**

1. Inventory spatial requirements from CSAPI, OGC API - Features, SensorML, SWE Common, and AEP-4789 material.
2. Inventory spatial requirements from `IDR-SRV-015` through `IDR-SRV-025`.
3. Classify spatial data categories by resource family, geometry type, temporal behavior, source authority, and policy sensitivity.
4. Identify spatial precision, uncertainty, provenance, and source metadata needs.
5. Build a geospatial data-category matrix.

**Expected Output:** Glaux Server geospatial data-category and resource-family matrix.

### Phase 3: Geometry, CRS, Spatial Query, and Indexing Analysis (3-4 hours)

**Objective:** Define spatial behavior and storage requirements.

**Tasks:**

1. Analyze geometry type support and geometry validity needs.
2. Analyze CRS, coordinate order, vertical reference, altitude/depth, orientation, and transformation implications.
3. Analyze bounding box, intersects, contains, within, proximity, spatial-temporal, and semantic-spatial query needs.
4. Analyze collection extents, derived bounding boxes, materialized extents, and query defaults.
5. Analyze spatial indexing options and performance implications.
6. Identify unresolved questions requiring prototype validation or benchmark testing.

**Expected Output:** Geometry/CRS/query/indexing strategy matrix.

### Phase 4: Storage Option and Architecture Analysis (2.5-3.5 hours)

**Objective:** Evaluate geospatial persistence options in the context of the high-level persistence architecture.

**Tasks:**

1. Evaluate PostGIS and relational-geospatial patterns.
2. Evaluate GeoJSON-in-document/hybrid patterns.
3. Evaluate embedded/local geospatial storage options.
4. Evaluate spatial libraries and application-layer geometry processing needs.
5. Evaluate implications for local development, CI, containers, tactical-edge deployments, and DDIL operation.
6. Compare options against the evaluation rubric and `IDR-SRV-025` findings.

**Expected Output:** Geospatial storage option comparison matrix.

### Phase 5: Security, DDIL, Fixtures, Performance, and Interoperability Implication Analysis (2.5-3 hours)

**Objective:** Prepare geospatial findings for downstream implementation and verification work.

**Tasks:**

1. Identify spatial policy, releasability, precision-reduction, suppression, and audit needs.
2. Identify DDIL, cache, last-known-position, synchronization, and stale-location implications.
3. Identify fixture and golden-file needs for points, polygons, trajectories, deployments, observations, and policy-filtered geometries.
4. Identify performance/load/stress tests for spatial queries and spatial-temporal queries.
5. Identify interoperability tests for CSAPI Explorer, webapp/mobile clients, QGIS/OGC API clients, and external CSAPI clients.
6. Map findings to Category E, F, G, H, and I topics.

**Expected Output:** Geospatial downstream implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert geospatial storage and query research into a decision-usable baseline.

**Tasks:**

1. Consolidate spatial data-category inventory, geometry/CRS/query guidance, storage option analysis, and downstream implications.
2. Produce recommended geospatial storage and query strategy direction(s) with rationale and unresolved questions.
3. Identify sequencing for downstream persistence, ingestion, DDIL, performance, fixture, and interoperability topics.
4. Produce downstream handoff matrix.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Geospatial data categories and resource-family spatial requirements are identified with source anchors.
- [ ] Static location, time-varying location, deployment extent, observation location, feature-of-interest geometry, sampling geometry, trajectory, footprint, and derived extent concepts are distinguished.
- [ ] Geometry, CRS, coordinate, vertical reference, precision, uncertainty, and provenance implications are documented.
- [ ] Spatial query, spatial-temporal query, collection extent, filtering, sorting, pagination, and indexing implications are documented.
- [ ] Candidate geospatial storage options are evaluated against explicit criteria.
- [ ] Security, policy, DDIL, fixture, performance, conformance, and interoperability implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Geospatial Storage and Query Strategy Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-026-geospatial-storage-and-query-strategy-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Spatial requirement extraction methodology
5. Geospatial data-category inventory
6. Resource-family spatial requirement matrix
7. Geometry type, CRS, coordinate, precision, uncertainty, and provenance findings
8. Spatial query, filtering, sorting, pagination, and extent findings
9. Spatial indexing and performance implications
10. Geospatial storage option evaluation
11. Spatial-temporal and dynamic-location implications
12. SensorML, SWE Common, and semantic integration implications
13. Security, policy, releasability, precision-reduction, and audit implications
14. DDIL, cache, synchronization, and last-known-position implications
15. Fixture, conformance, performance, and interoperability test implications
16. Downstream topic handoff matrix
17. Recommendations
18. Risks, constraints, and open questions
19. Validation against this plan's success criteria
20. References

The geospatial data-category matrix should include, at minimum:

- Data category
- Related resource family
- Source topic / source anchor
- Geometry type
- Static/time-varying classification
- Authoritative/derived/cache classification
- CRS/coordinate needs
- Spatial query/index needs
- Temporal interaction
- Precision/uncertainty/provenance needs
- Security/policy needs
- Candidate storage pattern(s)
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-025` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, GeoJSON, SensorML, SWE Common, relevant OGC schemas/OpenAPI artifacts, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Current candidate geospatial technology documentation must be reachable when the research is executed.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-027: Time-Series Observation Storage Strategy`
- `IDR-SRV-028: Metadata and Document Storage Strategy`
- `IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy`
- `IDR-SRV-031: Server Write and Ingestion Model`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
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

- This topic defines geospatial storage and query strategy, not final detailed geospatial schemas.
- Spatial behavior must be aligned with OGC API - Features expectations while supporting CSAPI-specific resource relationships and SensorML-derived position information.
- Implementation-study findings are useful but must not override standards-derived geospatial semantics.
- Open question: Which spatial query functions are required for first implementation versus full-scope readiness?
- Open question: How should time-varying sensor/platform location be represented without duplicating time-series storage decisions?
- Open question: Which CRS and vertical reference behavior is required by AEP-4789 profiles?
- Open question: Which location data must be generalized, suppressed, or policy-filtered?
- Open question: Which geospatial fixtures should become canonical conformance/interoperability test data?
- Risk: Treating all spatial data as static points could break deployments, sampling features, moving platforms, and observation-location use cases.
- Risk: Overly complex spatial storage could burden implementation without immediate interoperability value.
- Risk: Failing to handle CRS/coordinate assumptions carefully could cause incorrect spatial queries.
- Risk: Exposing precise locations or trajectories without policy controls could create operational security issues.

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
- GeoJSON RFC 7946: https://www.rfc-editor.org/rfc/rfc7946
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OGC CRS resources: https://www.ogc.org/standards/crs/
- OGC schemas: https://schemas.opengis.net/
- PostgreSQL documentation: https://www.postgresql.org/docs/
- PostGIS documentation: https://postgis.net/documentation/
- SQLite documentation: https://www.sqlite.org/docs.html
- SpatiaLite documentation: https://www.gaia-gis.it/fossil/libspatialite/index
- DuckDB spatial extension documentation: https://duckdb.org/docs/extensions/spatial/overview
- Rust geo crate documentation: https://docs.rs/geo/
- Rust geo-types crate documentation: https://docs.rs/geo-types/
- Rust geojson crate documentation: https://docs.rs/geojson/
- GDAL documentation: https://gdal.org/en/stable/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

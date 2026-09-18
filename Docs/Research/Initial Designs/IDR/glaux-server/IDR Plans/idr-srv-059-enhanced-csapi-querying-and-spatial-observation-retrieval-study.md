# Section 059: Enhanced CSAPI Querying and Spatial Observation Retrieval Study - Research Plan

**Topic ID:** IDR-SRV-059<br>
**Status:** In Progress - research report in review<br>
**Last Updated:** September 17, 2026<br>
**Estimated Research Time:** Not yet estimated; one focused research/report iteration is planned, subject to source access and unresolved questions.<br>
**Actual Research Time:** AI-assisted research conducted September 17, 2026, America/New_York (September 18 UTC); no human-effort estimate inferred<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md`

---

## Usage Instructions

Use the existing [Research Plan Template](../../../../../Governance/research-plan-template.md) structure and the [Research Report Template](../../../../../Governance/research-report-template.md) for the later report. The pinned OS4CSAPI exemplar corpus informs question-led analysis, concrete examples, source traceability, and decision-usable recommendations; its client scope, quantities, and estimates do not transfer to this study.

This is a focused post-synthesis supplement. The plan was published in commit `a17a88d`; the user's next `proceed` accepted it for execution and authorized the research/report iteration. The [report](../IDR%20Reports/idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md) is now in review. Report publication does not adopt filtering extensions or change the final synthesis, Goal and Definition, or Implementation Guide.

---

## 1. Research Objective

Determine how Glaux Server should address practical query limitations in OGC API - Connected Systems, especially **returning observations selected by spatial conditions on their associated sampling features**, including cases involving sampling chains and changing geometry. Assess measured-value and related-resource-property filtering alongside that primary spatial use case.

Establish what the approved CSAPI baseline already supports, where multi-request workflows are sufficient or inadequate, and whether OGC API - Features Part 3: Filtering and Common Query Language (CQL2) provide a suitable standards-based addition. Produce a recommendation for discussion, with explicit capability boundaries and implementation/test implications; do not assume adoption is the answer.

### Why This Topic Order

The original 67-topic IDR is complete and accepted. IDR-SRV-011 considered Features Part 3/CQL2 but deferred adoption pending demonstrated need; IDR-SRV-026 retained that extension boundary. IDR-SRV-058 has now assessed draft Connected Systems Part 4, and its accepted findings are available through the report and a separately prepared synthesis addendum. The Goal and Definition remains Version 1.6; the Implementation Guide is draft Version 0.1.

The project lead reported a September 17, 2026 CSAPI SWG discussion about query limitations experienced by CS-Go and OSH developers, and possible Features Part 3 work in CS-Go. The project lead then supplied the spatial-observation use case. The meeting account is attributed context; a public issue and implementation reference were not supplied. This study must distinguish that account from directly inspected evidence.

The agreed sequence is:

1. Draft and push this plan, with minimal supplemental index registration.
2. On the next `proceed`, conduct this study and push its report for review.
3. In a later authorized iteration, incorporate accepted findings into the final synthesis **as an addendum**, preserving the original synthesis and the Part 4 supplement.
4. Discuss any combined querying/Part 4 implications for the Goal and Definition or Implementation Guide; make only subsequently agreed changes.

The established `proceed` workflow applies; no special acceptance phrase is required. The Part 4 scope decision remains open and is not a prerequisite for studying its query touchpoints.

### Critical Constraints

- Preserve the goal of a full, well-written Rust reference implementation of Connected Systems. The approved Parts 1 and 2 target and applicable dependencies remain unchanged; no optional filtering feature substitutes for an existing obligation.
- Keep **Features Part 3: Filtering**, **Connected Systems Part 3: Publish/Subscribe**, and **draft Connected Systems Part 4: Sampling Features** distinct. A published filtering standard is not automatically an inherited CSAPI requirement.
- Separate what the standards require, what an implementation actually exposes, analytical possibilities, and proposed Glaux choices. An internal database filter or parser is not proof of a public interoperable query contract.
- Distinguish recursive association matching from applying a geometric predicate along a relationship chain. A parent or ultimate feature's geometry must not silently become an individual observation's sampling geometry.
- Do not assume every observation has a geometry or even a direct sampling-feature association. Conversely, do not assume an observation result cannot itself contain measured position information.
- Keep the study to the identified query needs. It does not commission arbitrary graph traversal, unrestricted SQL/joins, a general analytics service, aggregation, sorting/projection redesign, a new database, or a universal 3D/geometry-derivation engine.
- Do not install software, implement the server, modify external services, file issues, or contact developers as part of execution. Existing source and small local read-only checks can support the report; live deployments and a new benchmark/test suite are not prerequisites.

---

## 2. Research Questions

### Core Questions

1. **Q1 - Need and existing capability:** Which concrete queries are needed, what results should they mean, and how far do approved CSAPI filters and multi-request workflows already satisfy them?
2. **Q2 - Standards-based options:** What do Features Part 3 Queryables/Filter and selected CQL2 capabilities add, and what explicit mapping to CSAPI endpoints and properties would still be necessary?
3. **Q3 - Relationship, spatial, temporal, and result semantics:** How should each query identify the relevant feature, geometry, time, measured property, and value without conflating distinct meanings or inventing missing Part 4 behavior?
4. **Q4 - Implementation and verification:** What do pinned CS-Go and OSH sources demonstrate, and what bounded changes and tests would the viable options imply for the existing Rust/PostgreSQL/PostGIS design?
5. **Q5 - Recommendation and planning impact:** Should Glaux retain the existing approach, preserve room for an extension, or plan a specifically bounded addition? What evidence supports the recommendation and any changes to earlier research or planning?

### Detailed Questions

**Concrete use cases and the current baseline (Q1)**

Use the following cases to organize the study, not as an already adopted API contract. Spatial cases are primary. The report must show representative inputs and expected membership of a small illustrative result set; do not invent executable request syntax where no binding exists.

| Case | User-facing question | Distinction to establish |
|---|---|---|
| Static spatial selection | Return observations associated with sampling geometries that intersect a supplied search area. | What can sampling-feature `geom` filtering followed by observation `foi` filtering already achieve? Distinguish intersection from containment or distance requests. |
| Sampling-chain selection | Return observations associated with a matching feature reached through a defined sampling chain. | Association with an intersecting larger feature is not necessarily a measurement taken inside the area. Define which chain relationship and geometry are intended. |
| Spatial selection through time | Return observations whose relevant sampling geometry intersected the area when the observations apply. | Geometry at observation time is not necessarily current geometry, a whole trajectory, or an interval-wide envelope. |
| Values and related properties | Return observations meeting a measured-value condition and a condition on an associated resource, such as specimen material or sampling time. | Property identity differs from measured value; observation time differs from specimen sampling time. Part 4-specific fields remain conditional on adopting those types. |

- Trace the applicable Part 1 geometry, property, and relationship filters, Part 2 observation filters, and normative tests, including the recursive `sampleOf` traversal in observation `foi` test A.51. Identify discrepancies between prose, tests, and OpenAPI instead of treating any one artifact as the complete contract.
- For each case, distinguish a capability that is already expressible, an inconvenient multi-request workflow, an implementation gap, and a missing standardized behavior. Evaluate relevant identifier-list/request limits, paging, consistency between requests, and client work without assuming that one request is necessarily faster or more correct.
- Include a small representative metadata-resource comparison, such as filtering sampling features or systems by their own properties, to distinguish general CSAPI filtering limitations from observation-specific ones. Do not repeat the complete endpoint audit from IDR-SRV-011.

**Filter language and endpoint mapping (Q2)**

- Which Features Part 3 classes are reusable for resource-list endpoints, and which specifically bind to Features collections? What additional endpoint, discovery-link, API-description, conformance, and error rules would a CSAPI implementation need? Do not require observations to become GeoJSON features merely to evaluate this option.
- What can Queryables describe, including named searchable properties absent from returned representations? Could an observation query expose selected related-feature properties or geometry through a documented queryable without requiring clients to express database joins?
- Which CQL2 operators, encodings, and conformance classes are actually needed for the cases? Separate language syntax from property resolution, relationship traversal, and server execution. A dotted property name, property-property comparison, or custom function is not automatically a standardized CSAPI relationship path.
- How would new filters combine with existing `foi`, time, and other parameters? What must happen for unknown properties, unsupported operators, invalid types, null/missing values, and capability discovery? What is standardized, what could be a documented extension, and what needs SWG clarification for interoperable meaning?

**Spatial relationships, time, and observation values (Q3)**

- Distinguish direct sampling geometry, geometry of an intermediate sample, ultimate feature-of-interest geometry, observing-system location, and a spatial observation result. Which is relevant to each case? What constitutes a match when relationships or geometries are absent, multiple, cyclic, external, or inaccessible?
- Keep sampling-chain traversal separate from resolving relative coordinate frames or poses. Determine when a static explicit geometry suffices and when a Part 4 parametric shape, derived footprint, reference frame, or unresolved draft rule changes the answer. Test the distinction between an anchor point and a represented area/volume; do not infer an obligation to calculate every shape.
- For changing features, distinguish phenomenon time, result time, resource validity, current/as-of geometry, and any interval semantics. What history is needed to answer the request? If history or a time rule is missing, identify the limitation instead of silently using current geometry or inventing interpolation.
- For result filtering, how can a searchable field be bound to the datastream's SWE result schema, property definition, type, units, and encoding? How do scalar versus composite/array results and heterogeneous datastreams affect the proposed scope? Do not interpret arbitrary serialized JSON text as typed measurement comparisons.
- Where sampling-feature properties are themselves observation-backed, distinguish a selected latest/as-of value from any matching observation in an interval. Determine whether this is necessary for the concrete cases or should remain an explicit limitation.

**Implementation evidence, cost, and correctness (Q4-Q5)**

- What CS-Go and OSH behavior is exposed through public routes and parameters, advertised through discovery, implemented in handlers/storage, and covered by tests? Distinguish named CSAPI filters, implementation-specific parameters, internal predicates, result-value comparisons, and Features Part 3/CQL2 support.
- If the reported CS-Go work or SWG issue becomes available, inspect the actual branch/commit or issue and state its status. If not, retain the limitation; absence in a searched public snapshot is not evidence that private or unpublished work does not exist.
- What incremental changes would be needed in query parsing, property mapping, existing relationships, spatial/time indexes, result-value access, paging, and tests? Reuse the accepted design unless a demonstrated need supports a change; assess cost qualitatively without inventing performance results or schedules.
- How can a query avoid leaking hidden related features through result membership, counts, errors, or derived geometry? What bounded traversal/expression limits and safe parameter handling are needed? Do not propose automatic network dereferencing of arbitrary links during a query.
- Which independent expected-result cases would distinguish correct behavior from a successful parse or HTTP response? Which conclusions in IDR-SRV-011/026, the Part 4 supplement, or the current guide should remain, be qualified, or be reconsidered?

---

## 3. Primary Resources

1. **Approved Connected Systems baseline:** [Part 1, OGC 23-001, v1.0](https://docs.ogc.org/is/23-001/23-001.html), especially Sampling Features, time, and Advanced Filtering; [Part 2, OGC 23-002, v1.0](https://docs.ogc.org/is/23-002/23-002.html), especially observation models/encodings, §13 query parameters, and test A.51. Compare corresponding OpenAPI, schemas, and source at official tag `v1.0.0`, commit [`8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2). Follow incorporated [Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) clauses only where relevant.
2. **Candidate filtering standards:** [OGC API - Features Part 3: Filtering, OGC 19-079r2, v1.0](https://docs.ogc.org/is/19-079r2/19-079r2.html), especially §§6-9 and its abstract tests; [CQL2, OGC 21-065r2, v1.0.0](https://docs.ogc.org/is/21-065r2/21-065r2.html), especially queryable references, Boolean/comparison/spatial/temporal capabilities, encodings, and applicable conformance tests. Verify publication status and editions at execution; later repository changes are not silently substituted for published text.
3. **Observation and sampling semantics:** [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html) for result schemas, types, and units. Use [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) only where actual property/pose mappings require it. For draft Connected Systems Part 4, start at branch `part4-working-draft`, planning snapshot [`05a3c62d198ee52d0cf81a734b700967b7d864a1`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/05a3c62d198ee52d0cf81a734b700967b7d864a1/api/part4); recheck at execution and limit review to the relevant IDR-SRV-058 findings and any material changes.
4. **CS-Go implementation evidence:** [`SomethingCreativeStudios/connected-systems-go`](https://github.com/SomethingCreativeStudios/connected-systems-go), starting from the preceding assessment's public-main snapshot [`b1fd2e0e9bd69e222d05258d659a842ca24502cb`](https://github.com/SomethingCreativeStudios/connected-systems-go/tree/b1fd2e0e9bd69e222d05258d659a842ca24502cb). Entry points: `internal/model/query_params/observation_query_params.go`, `internal/repository/observation_repository.go`, related sampling-feature queries, conformance declarations, and `e2e/observations_test.go`. Check public branches/PRs or a supplied implementation reference for the reported filtering work; do not assume that the starting pin contains it.
5. **OSH implementation evidence:** [`opensensorhub/osh-core`](https://github.com/opensensorhub/osh-core), starting from IDR-SRV-058's snapshot [`9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08`](https://github.com/opensensorhub/osh-core/tree/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08). Entry points: CSAPI `ObsHandler` and `AbstractFeatureHandler` under `sensorhub-service-consys`, and internal `ObsFilter`/related feature filters under `sensorhub-core`. Follow only relevant handlers, storage evaluation, and tests. Pin any newer inspected source and distinguish public behavior from internal capabilities.
6. **Official maintenance context:** Consult the [shared upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), starting with [issue 165, sampling features](https://github.com/opengeospatial/ogcapi-connected-systems/issues/165) and [issue 179, observed/controlled property filters](https://github.com/opengeospatial/ogcapi-connected-systems/issues/179). Check for the newly reported SWG query issue at execution and include it only if identified. These older issues are related context, not substitutes for that unprovided issue. Inspect Features/CQL2 maintenance discussion only where a specific question cannot be resolved from the published standards.

For mutable sources, record the actual execution-time commit/edition, access date, inspected paths, and search boundary. These planning entry points do not establish current implementation conformance or settle the research questions.

---

## 4. Supporting Resources

Use affected sections of accepted reports rather than rereading or reopening the entire IDR:

- [IDR-SRV-011, query/filtering semantics](../IDR%20Reports/idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md), especially §§5.3-5.4, 17.2, and the deferred CQL2 decision; [IDR-SRV-026, geospatial storage/query strategy](../IDR%20Reports/idr-srv-026-geospatial-storage-and-query-strategy-report.md), especially §§8.2-8.3 and 11.4.
- [IDR-SRV-058, draft Part 4 study](../IDR%20Reports/idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md), especially §§4.2-4.4, 7, and 8; [Final IDR Research Report](../IDR%20Reports/final-idr-research-report.md), including Addendum A. The Part 4 report is accepted; addendum preparation did not adopt Part 4 support.
- IDR-SRV-017/018 for relationships and time; IDR-SRV-022/024 for result structure, property identity, and units; IDR-SRV-025/027/028 for existing persistence; [IDR-SRV-034, dynamic-data semantics](../IDR%20Reports/idr-srv-034-datastream-observation-and-status-update-semantics-report.md) for observation-backed properties and temporal selection.
- [IDR-SRV-014A, OSH study](../IDR%20Reports/idr-srv-014a-osh-csapi-server-implementation-study-report.md) and [IDR-SRV-014B, CS-Go study](../IDR%20Reports/idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md) as source/evidence entry points, not proof of newer support.
- Relevant sections of IDR-SRV-039/040 for access and disclosure, and IDR-SRV-050/051/053/054/056 for conformance, independent expected results, fixtures, performance questions, and interoperability. Use these only for incremental effects of the evaluated query options.
- [Goal and Definition v1.6](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md); [Implementation Guide draft v0.1](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md), especially §§1.2, 4.4, 4.7, 6.3, and 7-8; and the [Overall IDR Research Plan](overall-idr-research-plan.md). The repository planning baseline for this supplement is commit `9bfa62c9d4c0ef011b76d78d3ae13655ef0a4909`; record any later relevant changes at execution.

A targeted implementation feasibility question may require official documentation for the already proposed Rust dependencies, [PostgreSQL](https://www.postgresql.org/docs/), or [PostGIS](https://postgis.net/documentation/). Inspect only material needed to judge a concrete option; this is not a fresh framework/database/library selection survey.

---

## 5. Research Methodology

### Phase 1: Establish Use Cases and the Existing Contract

**Objective:** Answer Q1 and establish the evidence baseline.

**Tasks:**

1. Confirm internal prerequisites and planning-document versions. Record primary-source editions/pins and separate the meeting account from published evidence.
2. Define the four use cases with small illustrative records, relationships, geometries, times, and expected result membership. Identify ambiguities before selecting syntax or implementation mechanisms.
3. Trace the relevant approved requirements and tests. Work through the existing multi-request spatial-feature/observation workflow and classify its actual limits, including any association-versus-location differences.
4. Consult and date-check only relevant shared-history entries; refresh or add entries only for material topic evidence. Record the new SWG issue as unavailable if it is not identifiable, without inventing its content or delaying all analysis indefinitely.

**Expected Output:** A concise source inventory and use-case/baseline comparison inside the report.

### Phase 2: Evaluate Filtering Options and Their Meanings

**Objective:** Answer Q2-Q3 without preselecting Features Part 3, CQL2, or Part 4 adoption.

**Tasks:**

1. Map the cases to the reusable and feature-specific Features Part 3 classes and relevant CQL2 capabilities. Identify exact dependencies, endpoint applicability, discovery requirements, and behavior not supplied by those standards.
2. Explore a bounded, documented set of observation queryables for relevant result values and related-resource attributes/geometry. Compare this with existing CSAPI workflows; distinguish server-internal joins from a client-facing general join language.
3. Resolve or explicitly bound geometry-role, sampling-chain, coordinate/frame, time/history, schema/type/unit, missing-value, and multiple-value questions. Reuse IDR-SRV-058's draft limitations rather than inventing resolutions or repeating its full inventory.
4. For each candidate, label illustrative requests as existing-standard syntax, inspected implementation syntax, or a proposed extension. Identify interoperability gaps and any focused questions for future SWG consideration; do not submit them externally.

**Expected Output:** A case-by-option comparison and a short account of the additional meanings/rules each viable option would need.

### Phase 3: Inspect Implementations and Assess Incremental Work

**Objective:** Answer Q4 using bounded source and design evidence.

**Tasks:**

1. Inspect the relevant CS-Go and OSH route/parser, property mapping, storage/filter, discovery, and test paths at recorded commits. Follow any supplied newer filtering branch, but report public-source limitations explicitly.
2. Compare existing named parameters, private/internal filter capabilities, custom extensions, and demonstrated Features Part 3/CQL2 support. Do not infer conformance from a dependency, class name, or successful parse.
3. Assess only the incremental effects on Glaux's existing query modules, relationships, PostgreSQL/PostGIS storage/indexing, result-schema handling, authorization, bounded execution, and paging. Separate feasibility analysis from measured performance.
4. Define independent positive, negative, and boundary checks. Include a matching ancestor with a nonmatching sampling location, current versus historical geometry, absent geometry/associations, hidden linked resources, cycle/limit behavior, and typed result comparisons. Add CRS/boundary and parametric-shape cases only as needed by the candidate scope.
5. Small read-only checks may be performed with already available tools where they answer a specific question; record what actually ran. Do not build a server, install dependencies, or require a live-service campaign.

**Expected Output:** Pinned implementation observations, a compact incremental-impact table, and representative verification cases with explicit expected-result authority or remaining uncertainty.

### Phase 4: Synthesize the Recommendation and Produce the Report

**Objective:** Answer Q5 and support the later planning decision.

**Tasks:**

1. Compare retaining documented existing workflows, preserving compatibility while deferring extension work, and adding a precisely scoped standards-based filtering capability. Consider a project-specific addition only where an identified need remains and its interoperability cost is explicit.
2. Recommend an option with covered cases, exclusions, selected standards/classes if applicable, incremental complexity, unresolved questions, and conditions. A finding that existing filters suffice, or that evidence supports deferral, is a valid outcome.
3. Reconcile the recommendation with IDR-SRV-011/026 and the current guide's CQL2 boundary. Identify unchanged conclusions, qualified conclusions, and proposed changes separately; do not edit the synthesis, Goal, or Guide in this iteration.
4. Produce the report in the existing report-template order, map Q1-Q5 and success criteria to evidence, check references, and prepare it for user review with the next authorized step clearly stated.

**Expected Output:** One focused, independently readable research report. Tables and examples remain in that report; no separate requirements document, query framework, or governance system is created.

---

## 6. Success Criteria

This topic's research is complete when:

- [x] Q1-Q5 are answered with evidence or explicit limitations and their decision consequences.
- [x] All four use cases have clear meanings and illustrative expected results; spatial observation retrieval through sampling relationships is treated as a primary case.
- [x] Existing CSAPI behavior, multi-request limitations, implementation gaps, and genuinely additional API capabilities are distinguished.
- [x] Features Part 3/CQL2 options identify applicable classes, endpoint/property mappings, discovery, and unresolved interoperability rules without claiming automatic CSAPI inheritance or joins.
- [x] Geometry role, relationship recursion, time/history, relevant Part 4 limitations, and result schema/type/unit issues are addressed without conflating distinct spatial meanings.
- [x] CS-Go/OSH observations identify inspected commits and support depth; reported but unavailable work is not presented as verified implementation evidence.
- [x] Incremental design, access-control, query-cost, paging, and verification implications are grounded in the existing server research, with no invented runtime or performance results.
- [x] The recommendation identifies concrete scope, alternatives, costs, exclusions, and necessary decisions, including what earlier conclusions remain unchanged.
- [x] The report follows the template, contains reproducible references and coverage checks, and identifies the later synthesis-addendum/Goal/Guide discussion inputs without making those changes.
- [x] Relevant official maintenance evidence has been consulted and authority-classified, with only necessary shared-register updates.

Report completion and project-lead acceptance are separate; publication alone does not mark the topic accepted or adopt an implementation option.

---

## 7. Deliverable

**Deliverable Name:** Enhanced CSAPI Querying and Spatial Observation Retrieval Study - Research Report<br>
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md`

Follow the [Research Report Template](../../../../../Governance/research-report-template.md) in its required section order. Include:

1. A plain-language executive answer to whether Glaux needs additional filtering and what, if anything, is recommended.
2. Q1-Q5 coverage, source authority/pins, and findings traceable to specific clauses or code paths.
3. The use-case comparison, with spatial relationship/time examples and existing-versus-proposed request behavior clearly labeled.
4. Features Part 3/CQL2 applicability and limits, Part 4 touchpoints, and bounded CS-Go/OSH evidence.
5. Adoption options, incremental implementation/verification implications, unresolved questions, and a recommendation that remains distinct from an adopted scope change.
6. Validation against this plan and precise inputs for the later synthesis addendum and Goal/Guide discussion.

Do not produce server code, a new implementation guide, a roadmap, or a standalone requirements/decision-record system as part of this research deliverable.

---

## 8. Dependencies

### Must Complete Before Starting

**Internal project prerequisites (completion gates):**

- The original 67-topic IDR baseline, including IDR-SRV-057, is complete and accepted. The relevant reports in Section 4 are existing inputs, not newly assigned topics.
- IDR-SRV-058 is complete and accepted. Its Part 4 findings are an input; selecting a Part 4 implementation option is not required to conduct this study.
- Publish this topic plan before research execution. Completed in commit `a17a88d`; the user's subsequent `proceed` authorized the research/report iteration.

**External evidence prerequisites:**

- The approved CSAPI and published Features Part 3/CQL2 sources, with the directly relevant representation dependencies.
- Relevant official maintenance material and implementation source for the claims actually investigated.

If a source is inaccessible, identify it, the attempted access, the affected question, and the resulting limitation. Do not infer unavailable contents. The anticipated SWG issue, author response, or unpublished implementation branch is useful follow-up evidence, not a mandatory gate for the whole study. If central standards cannot be inspected, report that substantive limitation before claiming the study complete.

### Blocks (What This Topic Unlocks)

- An evidence-backed querying addendum to the final synthesis in a later authorized iteration.
- Discussion of whether to retain or revise the current filtering boundary in the Implementation Guide, and whether any agreed scope addition needs a Goal and Definition change.
- A combined discussion of the querying and Part 4 findings without making adoption of one automatically imply adoption of the other.

No completed topic is reopened or made retroactively dependent on IDR-SRV-059. The study does not create a new prerequisite for unrelated work.

---

## 9. Research Status Checklist

- [x] Phase 1 complete
- [x] Phase 2 complete
- [x] Phase 3 complete
- [x] Phase 4 synthesis complete
- [x] Deliverable draft complete
- [ ] Deliverable reviewed
- [ ] Deliverable accepted

**Actual Research Time:** AI-assisted research conducted September 17, 2026, America/New_York (September 18 UTC); no human-effort estimate inferred<br>
**Research Execution Date:** September 17, 2026<br>
**Completion Date:** Pending project-lead acceptance

Execution and technical review are complete; the unchecked review/acceptance items refer to the project lead's review. The report recommends a bounded optional Features Part 3/CQL2 addition for discussion, not adoption. Six fixed-data diagnostic assertions were checked; no peer server or conformance suite was executed. The next `proceed` can accept the report and authorize a separate final-synthesis addendum; Goal/Guide discussion follows afterward.

---

## 10. Notes and Open Questions

- The project lead's spatial example is sufficient to frame the study; the user need not supply query syntax or a database design. Clarify only a choice that materially changes the intended result and cannot be presented as explicit alternatives.
- The new SWG issue and the reported CS-Go filtering work have no supplied reference at planning time. If they become available, incorporate them as dated evidence without silently expanding this topic into an ongoing SWG-monitoring task.
- A query returning observations associated with an intersecting feature may be correct while answering a different question from observations taken within the area. Make that difference visible in examples and recommendations.
- Generic sampling-feature geometry queries do not depend on adopting Part 4. Conversely, filtering syntax does not resolve the draft's implicit-shape, frame, or temporal ambiguities.
- Preserve the accepted research record. If evidence supports changing the prior CQL2 deferral, explain why and identify the exact proposed change; do not rewrite the old conclusion as though it had never been made.

---

## References

- [Research Planning Approach](../../../../../Governance/research-planning-approach.md) and [Initial Planning Guidance](../../../../../Governance/initial-planning-guidance.md).
- [Research Plan Template](../../../../../Governance/research-plan-template.md), [Research Report Template](../../../../../Governance/research-report-template.md), and [Overall Research Report Template](../../../../../Governance/overall-research-report-template.md) for the later synthesis handoff.
- OS4CSAPI research-plan exemplar corpus, audited snapshot [`754411897173c2ec4debaa9bcf4ed9e0f8a9e230`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans), particularly [01: blueprint analysis](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md), [15: fixture sourcing](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/15-fixture-sourcing-organization.md), and [38: synthesis](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/38-testing-playbook-synthesis.md).
- Topic-specific standards, implementations, maintenance evidence, and existing Glaux reports are identified in Sections 3 and 4; the report must cite the exact sources inspected during execution.

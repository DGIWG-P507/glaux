# Section 058: Draft CSAPI Part 4 Sampling Features Study - Research Plan

**Topic ID:** IDR-SRV-058<br>
**Status:** Planned<br>
**Last Updated:** September 17, 2026<br>
**Estimated Research Time:** Not yet estimated; one focused research/report iteration is planned, subject to source access and unresolved questions.<br>
**Actual Research Time:** Not started<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md`

---

## Usage Instructions

Use the existing [Research Plan Template](../../../../../Governance/research-plan-template.md) structure and produce the later report using the [Research Report Template](../../../../../Governance/research-report-template.md). The pinned OS4CSAPI exemplar corpus listed under References informs question-led analysis, source inventories, explicit limitations, and useful downstream recommendations; its client-specific scope and estimates do not transfer to this study.

This is a post-synthesis supplement, not a reopening of the original 67-topic IDR effort. This iteration prepares and publishes the plan and its index entry only. Research, the report, the synthesis addendum, and any planning-document changes remain separate steps.

---

## 1. Research Objective

Determine what the official working draft of **OGC API - Connected Systems Part 4: Sampling Features** would add to Glaux Server's approved Parts 1 and 2 baseline, how sufficiently it defines those additions, and whether the evidence supports including any of them experimentally in the server's plans.

Produce a focused, evidence-backed report that distinguishes existing obligations from draft additions, identifies practical server and test implications, and recommends an adoption boundary for discussion. The report must make it possible to decide whether to defer Part 4, preserve compatibility without committing to implementation, or include clearly identified draft capabilities. It must not make that project decision on the user's behalf.

### Why This Topic Order

The original IDR and its final synthesis are complete and accepted. The Goal and Definition is at Version 1.6, and the Implementation Guide has its first complete draft at Version 0.1. Earlier research considered generic Sampling Features and noted draft Part 4, but a dedicated assessment is now needed before deciding whether that draft should affect the next guide iteration.

The agreed sequence is:

1. Draft and push this plan, with its minimal supplemental index registration.
2. On the next `proceed`, conduct this study and push its report for review.
3. In a later authorized iteration, add the accepted findings to the final synthesis **as an addendum**, preserving the original accepted baseline and its historical completion record.
4. Discuss whether the Goal and Definition and/or Implementation Guide need changes; make only the changes subsequently agreed.

No special acceptance phrase is required; the user's established `proceed` workflow applies to the stated next iteration.

### Critical Constraints

- Preserve the goal: a well-written Rust reference implementation of OGC API - Connected Systems. Do not expand this study into a new programme of work.
- Approved Parts 1 and 2 and their applicable normative dependencies remain authoritative. The existing 25-class target is not changed by planning or researching Part 4.
- Label draft statements, informative maintenance history, implementation observations, and Glaux recommendations separately. Neither a working-draft requirement nor a peer implementation establishes an approved conformance claim.
- Keep **Connected Systems Part 4: Sampling Features**, **OGC API - Features Part 4 transactions**, and **Simple Feature Access (SFA)** distinct. SFA geometry/WKT references alone do not constitute Part 4 coverage.
- Investigate server representations, validation, relationships, queries, persistence, and tests. Do not assume a general 3D GIS engine, terrain analysis, rendering, sensor-physics simulation, or new client application is required.
- Do not implement server code, install software, mutate external services, or require live deployment testing to complete this study. Missing draft material is an evidence gap, not permission to invent specification behavior.

---

## 2. Research Questions

### Core Questions

1. **Q1 - Source and maturity:** What exactly does the pinned official Part 4 draft contain, what is its authority, and which prose, requirements, schemas, examples, and tests form a usable and internally consistent description?
2. **Q2 - Standards difference:** What does it add to, reuse from, or potentially conflict with approved Parts 1 and 2 and their dependencies, especially the existing generic SamplingFeature resource and its associations?
3. **Q3 - Representation and behavior:** For each actual draft feature family, what representation, geometry or parametric description, relationships, reference frames, units, temporal behavior, validation, and query semantics are defined or left unresolved?
4. **Q4 - Glaux implications:** Which existing research conclusions and draft guide sections remain valid, need qualification, or would require additional implementation and verification work? What directly inspected implementation evidence helps establish feasibility or interoperability limitations?
5. **Q5 - Recommendation:** Should Glaux defer Part 4, retain compatibility without implementation commitment, or plan selected or broader experimental support? What precise scope, conditions, benefits, costs, and unresolved decisions distinguish those options?

### Detailed Questions

**Draft inventory and authority (Q1-Q2)**

- Which files are actually included in the draft, and which are unused, commented out, duplicated, or generated? Do the rendered documents match their source? Are requirement identifiers, conformance declarations, schema references, and examples resolvable?
- Which provisions are inherited approved requirements, new draft proposals, informative examples, or editorial placeholders? Are any apparent differences really corrections to Glaux's understanding of existing Part 1 or Part 2 obligations?
- What do the topic-relevant official issues and pull requests explain, and what remains unresolved? Do not treat an issue title, branch name, or merged change as sufficient evidence of publication status.

**Feature types and semantics (Q2-Q3)**

- Inventory all feature families actually present, including spatial and parametric forms, rather than selecting only convenient examples. For each, trace the defining clauses to requirements, schemas, examples, and any tests; record missing counterparts.
- What do the types say about sampled features, sampling chains or feature parts, specimens or other sample descriptions, identifiers, links, and associations with systems or observations? Distinguish conceptual types from additional REST resources or operations, if any.
- Where is an explicit geometry required or derived? How do relative locations, platform motion, pose, reference frames, orientation, units, and time affect the representation? What is a server required to calculate, as distinct from store, validate, or serve?
- How do applicable rules affect bounding-box and other spatial filtering, temporal selection, and historical or latest representations? Where are precision, coordinate reference system, or transformation rules unspecified?
- What is directly inherited from SFA, GeoJSON, SensorML, SWE Common, GeoPose, or other cited sampling/geometry standards? Follow only the dependencies needed to interpret these draft provisions; do not open new surveys of whole standards families.

**Implementation and verification (Q4)**

- Can the existing domain model, preserved source documents, PostgreSQL/PostGIS strategy, validation layers, and query design represent the draft without losing meaning? Which assumptions need a targeted change or experiment?
- Are there additional validation, resource-limit, disclosure, or relationship-access concerns beyond existing server controls? Identify only effects attributable to the draft, without repeating the full security research.
- Do already relevant implementations, initially OSH and CS-Go, contain demonstrable Part 4-specific behavior? Distinguish generic SamplingFeature handling, names or example payloads, schema/type support, and implemented API behavior. What remains untested?
- Which positive, negative, boundary, temporal, and cross-representation examples would distinguish correct handling from mere JSON acceptance? What could be verified from the draft alone, and what lacks an authoritative expected result?

**Decision and handoff (Q5)**

- For each adoption option, what is the smallest clearly described scope that answers the identified need without reducing approved Parts 1 and 2 coverage?
- What exact sections of the final synthesis, Goal and Definition, or Implementation Guide would need an addition, qualification, or no change? Separate research findings from proposed project commitments.

---

## 3. Primary Resources

1. **Official Connected Systems Part 4 working draft.** Planning snapshot: branch `part4-working-draft`, commit `05a3c62d198ee52d0cf81a734b700967b7d864a1` (June 30, 2026). At research start, recheck the branch and record the execution snapshot and access date. If it changed, describe the material difference rather than silently replacing this planning reference.
   - [Pinned Part 4 source tree](https://github.com/opengeospatial/ogcapi-connected-systems/tree/05a3c62d198ee52d0cf81a734b700967b7d864a1/api/part4).
   - Entry document: `api/part4/standard/23-004r0.adoc`; inspect its included sections, scope and conformance clauses, build metadata, and accompanying HTML/PDF artifacts.
   - Main feature description: `api/part4/sections/clause_13_sampling_feature_types.adoc`.
   - Requirements and representations: `api/part4/requirements/`, including nested requirement files, and `api/part4/openapi/schemas/`. Follow referenced examples and tests wherever the pinned tree actually places them.
2. **Approved Connected Systems baseline.** Read the relevant clauses of [Part 1, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html) and [Part 2, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html). Use the official [`v1.0.0` source snapshot, commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2) to compare corresponding artifacts. Record any published-document/source disagreement.
3. **Directly referenced representation dependencies.** Begin with [SensorML 3.0, OGC 23-000](https://docs.ogc.org/is/23-000/23-000.html) and [SWE Common 3.0, OGC 24-014](https://docs.ogc.org/is/24-014/24-014.html) where invoked. Derive other required editions and clauses from the actual draft and approved baseline references; do not substitute a newer edition silently.
4. **Official maintenance evidence.** Consult the [shared upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), beginning with the sampling-feature boundary recorded around [issue 82](https://github.com/opengeospatial/ogcapi-connected-systems/issues/82), [PR 96](https://github.com/opengeospatial/ogcapi-connected-systems/pull/96), and [issue 165](https://github.com/opengeospatial/ogcapi-connected-systems/issues/165). Follow their relevant resolution artifacts and any directly connected Part 4 changes. History remains informative.

---

## 4. Supporting Resources

Use accepted reports as the starting baseline; inspect affected sections rather than repeating the entire IDR:

- **Approved obligations and queries:** IDR-SRV-006, 007, 008, and 011. In particular, use [008's conformance mapping and sampling-feature boundary](../IDR%20Reports/idr-srv-008-conformance-class-and-requirement-mapping-report.md).
- **Model, relationships, and time:** IDR-SRV-015 through 018, with 019 and 034 where lineage or observation associations are affected.
- **Representations and persistence:** IDR-SRV-021 through 026 and 028. Use [026's geospatial storage/query assessment](../IDR%20Reports/idr-srv-026-geospatial-storage-and-query-strategy-report.md) to distinguish existing decisions from draft-informed observations.
- **Verification:** IDR-SRV-050, 051, 053, and 056 for conformance boundaries, requirement-to-test links, fixture provenance, and interoperability evidence.
- **Implementation evidence entry points:** [014A, OSH](../IDR%20Reports/idr-srv-014a-osh-csapi-server-implementation-study-report.md), [014B, CS-Go](../IDR%20Reports/idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md), and [014H, draft Part 3](../IDR%20Reports/idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md). Reuse their source locations and evidence discipline, not their conclusions as proof of Part 4 support or their larger study scope as a requirement for this supplement.
- **Accepted synthesis:** [Final IDR Research Report, IDR-SRV-057](../IDR%20Reports/final-idr-research-report.md). Identify future addendum targets without rewriting it during this study.
- **Current project direction:** [Goal and Definition, Version 1.6](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) and [Implementation Guide, Version 0.1](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md). Repository commit `9c72728` contains this planning baseline; record their actual versions again at research start.
- **Controlling index:** [Overall IDR Research Plan](overall-idr-research-plan.md), including the separate supplemental entry for this topic.

Relevant implementation repositories and source files become primary evidence for implementation claims only after direct inspection and commit pinning. Generic support does not prove draft Part 4 implementation. No minimum number of positive implementation findings is required.

---

## 5. Research Methodology

### Phase 1: Establish the Source and Authority Baseline

**Objective:** Answer Q1 and establish a reproducible, bounded evidence set.

**Tasks:**

1. Confirm the accepted internal baseline, current planning-document versions, and Part 4 execution snapshot. Record URLs, commits or editions, access dates, authority, and any access limitations.
2. Inventory the actual draft include structure, requirements, schemas, examples, tests, and rendered artifacts. Distinguish included content from unreferenced or commented material; record broken references and discrepancies.
3. Consult and date-check only the shared history-register entries relevant to sampling features and Part 4. Refresh those entries if material evidence changed; do not re-audit unrelated standards history.

**Expected Output:** Report source inventory and a concise draft-maturity/gaps account. No additional research programme or standalone register is required.

### Phase 2: Analyze the Draft Against Approved Requirements

**Objective:** Answer Q2-Q3 with complete coverage of the actual draft feature families.

**Tasks:**

1. Trace each family and material behavior from clause/requirement to its representation, schema, dependencies, examples, and tests. Record whether the evidence agrees and where it is incomplete.
2. Compare it with the approved generic SamplingFeature resource and relevant Parts 1 and 2 behavior. Classify inherited obligations, draft additions, apparent conflicts, and informative material explicitly.
3. Analyze explicit and parametric geometry, relationships, pose/reference frames, units, time, validation, and query implications. Do not equate document storage with semantic support or infer geometry calculations the draft does not specify.
4. Where useful and supported by existing tools, perform small reproducible read-only schema/reference or example checks. Record exactly what ran and its result; do not claim validation from inspection alone. Do not install tools or build a server as a prerequisite.

**Expected Output:** A feature-family/requirement coverage table and supported explanations of the differences from the approved baseline. Unknowns remain visible rather than being resolved through invented rules.

### Phase 3: Assess Targeted Glaux and Implementation Impacts

**Objective:** Answer Q4 without reopening completed research topics.

**Tasks:**

1. Follow each material difference into the relevant accepted report and current guide section. Record whether it leaves the conclusion unchanged, qualifies it, or suggests additional design or verification work.
2. Check OSH and CS-Go source evidence using the existing implementation studies as entry points. Pin the inspected snapshots, follow relevant type/schema/handler/test paths, and record the search boundary. Expand to another already relevant implementation only if a concrete lead could resolve a material question.
3. Distinguish inspected support, partial support, absence of evidence in the searched snapshot, and untested interoperability. Do not turn a limited source search into a claim that no implementation exists anywhere.
4. Identify representative future verification cases for the proposed capabilities and any unresolved expected results. Assess incremental model, storage/query, validation, access-control, and compatibility costs; pursue Rust/library questions only where a demonstrated gap affects the recommendation.

**Expected Output:** A compact impact table identifying affected research/planning sections and proposed verification, plus a bounded implementation-evidence summary. No live service campaign or new test suite is required.

### Phase 4: Synthesize Recommendations and Produce the Report

**Objective:** Answer Q5 and make the later project decision understandable.

**Tasks:**

1. Compare deferral, compatibility-aware design without implementation commitment, selected experimental support, and broader experimental support. State which types or behaviors each option actually includes, rather than treating the options as labels only.
2. Recommend an option with evidence, benefits, incremental work, uncertainties, and conditions. Use qualitative effort categories or explicitly grounded estimates; do not invent precise implementation schedules or redefine the approved server scope.
3. Explain conflicting evidence and unresolved items, their practical consequences, and what would resolve them. Incomplete draft conformance packaging may limit a recommendation without preventing an honest report.
4. Write the report using the standard report template, map Q1-Q5 and the success criteria to its findings, verify references, and prepare it for user review. Identify the later synthesis-addendum and Goal/Guide discussion targets without editing those documents.

**Expected Output:** One decision-usable research report, with findings, interpretations, and recommendations visibly distinguished.

---

## 6. Success Criteria

This topic's research is complete when:

- [ ] Q1-Q5 are answered with specific evidence or an explicit limitation and its decision consequence.
- [ ] The execution snapshot, authority, artifact inventory, and material source/schema/example gaps are reproducible.
- [ ] All feature families actually present in the draft are accounted for, including incomplete or excluded material; inherited approved requirements and draft additions are separated.
- [ ] Geometry/parametric representation, relationships, frames, units, time, validation, and applicable query effects are addressed without inventing unspecified behavior.
- [ ] The impact on relevant accepted research and current planning sections is explicit, including conclusions that remain unchanged.
- [ ] Implementation observations are pinned, bounded, and distinguished from verified interoperability or conformance.
- [ ] Adoption options and a justified recommendation identify concrete capability scope, incremental work, verification needs, and unresolved decisions.
- [ ] The report follows the report template, contains an evidence-backed coverage check, and identifies later addendum/discussion actions without making them.
- [ ] Relevant official history has been consulted and authority-classified, with only necessary shared-register refreshes.

Report completion and user acceptance are separate; publication alone does not mark this topic accepted.

---

## 7. Deliverable

**Deliverable Name:** Draft CSAPI Part 4 Sampling Features Study - Research Report<br>
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md`

Follow the [Research Report Template](../../../../../Governance/research-report-template.md) in its required section order. Within that structure, include:

1. An executive answer to whether and how Part 4 should be considered for Glaux, with the recommendation clearly distinguished from a project decision.
2. Q1-Q5 coverage, pinned sources and authority, and findings supported by specific clauses or code paths.
3. The feature-family/requirement comparison, draft gaps, and relevant implementation evidence.
4. Targeted impacts on the accepted research, current server design, and proposed verification, including unchanged conclusions.
5. Adoption-option analysis, incremental implementation implications, risks, unresolved questions, and the recommended scope boundary.
6. Success-criteria validation, reproducible references, and precise inputs for the later synthesis addendum and planning discussion.

Keep inventories and comparison tables inside the report, with appendices only where needed. Do not create a separate requirements document, implementation guide, roadmap, or new governance framework as a research deliverable.

---

## 8. Dependencies

### Must Complete Before Starting

**Internal project prerequisites (completion gates):**

- The original 67-topic IDR baseline, including IDR-SRV-057, is already complete and accepted, as recorded in the overall plan on September 16, 2026. The reports named in Section 4 are therefore available accepted inputs, not new research assignments.
- This topic plan must be available for user review before the separately authorized research iteration begins. The current authorization covers plan preparation/publication, not report execution.

**External evidence prerequisites:**

- The official Part 4 working-draft source and approved Parts 1 and 2 baseline, with directly referenced dependencies needed to answer the questions.
- Topic-relevant official history and selected implementation source for the claims actually investigated.

If an external source cannot be accessed, record its identity, attempted access, affected questions, and resulting limitation. Do not infer its contents or substitute a peer implementation for the standard. Lack of peer support is not itself a blocker; inability to inspect the central draft must be reported as a substantive limitation before claiming the study complete.

### Blocks (What This Topic Unlocks)

- An evidence-backed Part 4 addendum to the accepted final synthesis in a later authorized iteration.
- An informed discussion of whether Part 4 should change the Goal and Definition or the next Implementation Guide draft.

No completed IDR topic is reopened or made retroactively dependent on IDR-SRV-058. This plan neither authorizes implementation nor creates a new gate for work unrelated to the Part 4 decision.

---

## 9. Research Status Checklist

- [ ] Phase 1 complete
- [ ] Phase 2 complete
- [ ] Phase 3 complete
- [ ] Phase 4 synthesis complete
- [ ] Deliverable draft complete
- [ ] Deliverable reviewed
- [ ] Deliverable accepted

**Actual Research Time:** Not started<br>
**Completion Date:** Not completed

---

## 10. Notes and Open Questions

- The planning snapshot identifies where to begin; it is not a finding that the entire draft is ready to implement. The report must establish that from direct analysis.
- A type name, schema file, example, or pose-capable dependency is not by itself evidence of an implemented Part 4 contract. Report the actual depth of support.
- The extent to which the draft specifies geometry derivation and time-dependent querying is an open research question, not a preselected server responsibility.
- No option is selected by this plan. If the evidence supports only partial experimental adoption, state what remains unsupported and why; if it supports deferral, state how the approved server remains correct without that draft extension.
- Preserve the original synthesis and its acceptance history. The later addendum should identify new evidence and changed conclusions explicitly, rather than silently rewriting the original baseline.

---

## References

- [Research Planning Approach](../../../../../Governance/research-planning-approach.md).
- [Initial Planning Guidance](../../../../../Governance/initial-planning-guidance.md).
- [Research Plan Template](../../../../../Governance/research-plan-template.md) and [Research Report Template](../../../../../Governance/research-report-template.md).
- [Overall Research Report Template](../../../../../Governance/overall-research-report-template.md), for the later synthesis handoff rather than a second report in this iteration.
- OS4CSAPI research-plan exemplar corpus, audited snapshot [`754411897173c2ec4debaa9bcf4ed9e0f8a9e230`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans), particularly [01: blueprint analysis](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md), [15: fixture sourcing](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/15-fixture-sourcing-organization.md), and [38: synthesis](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/38-testing-playbook-synthesis.md).
- Topic-specific official sources and accepted project reports are listed in Sections 3 and 4; the report must provide exact execution-time references for its claims.

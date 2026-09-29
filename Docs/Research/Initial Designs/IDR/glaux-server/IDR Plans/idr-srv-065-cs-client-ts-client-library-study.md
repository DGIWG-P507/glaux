# Section 065: cs-client-ts Client Library Study - Research Plan

**Topic ID:** IDR-SRV-065<br>
**Status:** Planned<br>
**Last Updated:** September 29, 2026<br>
**Estimated Research Time:** Not yet calibrated; four bounded phases below, with further research iterations only if needed for the stated coverage.<br>
**Actual Research Time:** TBD until complete<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-065-cs-client-ts-client-library-study-report.md`

---

## Usage Instructions

Use the existing [Research Plan Template](../../../../../Governance/research-plan-template.md), and for the later report the [Research Report Template](../../../../../Governance/research-report-template.md), preserving their section order. The pinned OS4CSAPI exemplars inform question-led investigation, concrete examples and practical recommendations. Their client-specific scope, metrics and estimates are not Glaux requirements.

The project lead requested this study on September 29, 2026. It is one of four separate client studies (IDR-SRV-063 to IDR-SRV-066), each planned, executed and reported individually before the [Phase 1 implementation review](../../../../../Plans/glaux-server/Implementation-Reviews/Phase-1/README.md) resumes. It starts on its own `proceed`, after IDR-SRV-064's report is accepted.

---

## 1. Research Objective

Determine what **`cs-client-ts`**, a TypeScript Connected Systems API (CSAPI) client library published on npm as `cs-api-client`, shows about how an experienced developer models the API for programmatic use. Establish:
- its resource and operation coverage;
- the requests it builds;
- the response members, links, identifiers, media types, paging, streaming and error behaviours its types and parsers assume;
- how its tests establish expected behaviour;
- where it is strict or tolerant.

Judge each material assumption against the approved CSAPI text. Produce independent evidence for:
- the Phase 1 implementation review;
- later review gates;
- the external-client matrix of [IDR-SRV-056](../IDR%20Reports/idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md);
- the Aleph application study (IDR-SRV-066), which depends on this library.

Give each material finding a Glaux disposition: already covered, a bounded change worth discussing, not applicable, or insufficient evidence.

### Why This Topic Order

The project lead reports that `cs-client-ts` and the Aleph client were written by a senior developer with AI assistance. The same developer wrote the Connected Systems Go server (CS-GO), which Glaux has already studied in [IDR-SRV-014B](../IDR%20Reports/idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md) and [IDR-SRV-062](../IDR%20Reports/idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md).

This study follows the two OSH-family studies. It comes directly before IDR-SRV-066, because Aleph depends on `cs-api-client` `0.1.3`, the latest version on npm at the preliminary check. The library describes itself in its `package.json` as a "TypeScript client for OGC API Connected Systems Parts 1, 2, and 3". It has an optional MQTT publish/subscribe module (`./mqtt` export, `src/pubsub/`, `src/mqtt.ts`) and uses `zod` for runtime validation, so it also gives early evidence about draft Part 3 use. A typed library makes the author's reading of the API explicit, which suits comparison with the standard and with Glaux's own representations.

### Critical Constraints

- **Independence is limited, and the report must say so.**
  - The same author wrote CS-GO. The library may encode one person's reading of the standard that also shaped the server, so agreement between the two is not independent confirmation.
  - The study must identify where the library's tests or fixtures are derived from CS-GO responses, from the standard's examples or schemas, or from the author's own models.
  - The project lead reports AI assistance. Do not infer which parts were AI-assisted.
- **History is shallow.**
  - At the preliminary check the repository has 8 reachable commits and about 222 files, including 30 test files (`*.test.ts`) and 98 JSON fixtures under `test/`. The development history appears to be largely squashed.
  - Two commit messages state deliberate departures from the standard: `4724b0e` ("Broke from standard just a little bit...", the source of npm `0.1.2` and `0.1.3`) and `a632798` ("Adding name param to abstract process optionally... break from standard"). The study examines both as history cases.
  - Do not draw conclusions about process, test-first sequencing or evolution that the history cannot support. State the limit.
- **No license.**
  - The repository shows no license file at the preliminary check. Reading and analysis are permitted.
  - No source, types or fixtures may be copied into Glaux. Recommendations describe behaviour in Glaux's own terms.
- **Treat this as informative evidence.**
  - Where the library and the standard disagree, the standard wins and the disagreement is recorded.
  - A typed model accepting or requiring a member is not proof of what the standard requires.
- **Execution.**
  - No installation on the project lead's company laptop.
  - Run the library's tests only in an already permitted, isolated environment, and record the revision, command, dependencies and results. Otherwise the study is source inspection, stated as such.
  - Do not call external servers. No maintainer contact and no upstream posts.
- **Pin both versions, and say which is which.**
  - npm `0.1.3` (published September 1, 2026) was built from commit `4724b0e075f2d491885caa3824fcf0da23a464d1`. This is the code Aleph resolves, and it is the **primary baseline**.
  - The repository head `a6327989f5cec3c36422a121a2fe676e8a6bbe6d` (September 15, 2026) is newer but unpublished, although its `package.json` still says `0.1.3`. It is studied as a labelled secondary baseline, mainly for the change in `a632798`.
- **Scope.** Study the library at both baselines. Compare with CS-GO only as far as needed to judge independence and shared interpretation. Do not re-study CS-GO.

---

## 2. Research Questions

### Core Questions

1. **Q1 — Identity, provenance and independence:**
   - Exactly which code and package are studied?
   - Which API versions and Parts does it target?
   - How independent is its reading of the standard from CS-GO and from AI-assisted sources, as far as the evidence shows?
2. **Q2 — Coverage and requests:** Which resource families, operations, parameters, headers, media types, authentication and streaming mechanisms does the library support, and how does it construct URLs and discover them?
3. **Q3 — Response models and tolerance:**
   - What do its types and parsers assume about response members, identifiers, links and relations, representations, time and geometry, schemas, paging, status and errors?
   - Where do they reject, coerce or silently drop data?
4. **Q4 — Standards alignment:** For each material assumption, what do CSAPI Parts 1 and 2 (and incorporated Common/Features, SWE Common and SensorML) require? Classify the assumption as conforming, stricter than required, tolerant, CS-GO-specific, draft or experimental, or non-conforming.
5. **Q5 — Tests and fixtures:**
   - What do the 30 test files and 98 fixtures establish?
   - Where do expected values come from?
   - Would plausible wrong server behaviour make them fail?
6. **Q6 — Transfer to Glaux:** Which concrete expectations, fixtures or checks should inform:
   - the Phase 1 review;
   - later gates;
   - IDR-SRV-056;
   - IDR-SRV-066;
   - the Guide and Roadmap?

   Give an explicit disposition for each.

### Detailed Questions

**Identity and independence (Q1)**

- Record both baselines: the npm versions (`0.1.0`–`0.1.3`) with their recorded source commits and tarball contents, and the unpublished repository head. Describe every difference that matters to requests, models or tests.
- Identify statements of targeted Parts or versions in the README, package metadata and code. The package claims Parts 1, 2 and 3; record the draft Part 3 support separately.
- Examine the two stated departures from the standard (`4724b0e`, `a632798`): what changed, which requirement it departs from, and whether the change is documented for users.
- Compare selected type definitions and fixtures with CS-GO's representations at the IDR-SRV-062 pin, and with the standard's examples and schemas. Classify each fixture's origin as standard example, CS-GO-derived, hand-written or unknown.

**Coverage and requests (Q2)**

- Inventory the public API surface: resource families, nested routes and operations (read, create, replace, update, delete and any commands), and the options each exposes.
- Record URL construction, parameter encoding (including time, spatial and filter parameters), paging, format selection and headers. Record whether paths are discovered from links or built from templates.
- Record authentication support, and the MQTT publish/subscribe module (`./mqtt` export, `src/pubsub/`, `test/pubsub`): broker connection, topic structure, message codecs and event handling. Classify it against the draft Part 3 material and Glaux's experimental boundary (Guide §4.8).

**Response models and tolerance (Q3)**

- For each resource model, list the required and optional members, how `id`/`uid`, `links`, relation values, GeoJSON versus SensorML JSON, and `featureType` or `definition` are represented, and how unknown members are handled.
- Examine the error model, status codes, retries, paging continuation and partial results. Identify coercions such as time parsing, numeric precision and coordinate order.

**Standards alignment (Q4)**

- Map each material assumption to exact requirement identifiers.
- Consult only the [upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md) entries the findings implicate. Classify issue and pull-request evidence as informative.
- Compare with Glaux Guide §13 interpretations, and note any points where CS-GO and the library agree against the standard or against Glaux's reading.

**Tests and fixtures (Q5)**

- Inventory the test layers (unit, request-building, parsing, publish/subscribe, integration), helpers and the 98 JSON fixtures. Explain representative tests: setup, input, expected result, assertion and the failure each would catch.
- Assess whether each expected value is independent of the code under test. Mark any test that could pass with a matching mistake in both library and fixture.

**Transfer to Glaux (Q6)**

- For Phase 1, list the library's expectations for System discovery, creation and a single System representation. Say whether Glaux's implemented behaviour meets or conflicts with them.
- For later phases, list the checks each relevant gate review should include, with pins.
- Recommend whether the library should join IDR-SRV-056's matrix, as a pinned package run in an approved environment, and state its independence limits.
- Record what IDR-SRV-066 should assume about the library version Aleph uses.
- Compare each lesson with Guide sections and Roadmap task IDs before proposing work. "No change" is acceptable.

---

## 3. Primary Resources

- **Library repository:** [SomethingCreativeStudios/cs-client-ts][Lib] at default branch `main`, commit [`a6327989f5cec3c36422a121a2fe676e8a6bbe6d`][LibPin] (preliminary check, September 29, 2026). Includes `README.md`, `package.json`, `src/` (starting at `src/api/`), `test/` and configuration. Record the actual execution snapshot.
- **Published package:** [`cs-api-client`](https://www.npmjs.com/package/cs-api-client) on npm, latest `0.1.3` (September 1, 2026, built from [`4724b0e075f2d491885caa3824fcf0da23a464d1`][LibNpm]). Include the published tarball contents and the version history.
- **Independence comparison:** [CS-GO](https://github.com/SomethingCreativeStudios/connected-systems-go) at the IDR-SRV-062 pin `b1fd2e0e9bd69e222d05258d659a842ca24502cb` and, where needed, its current head. Use only the representations, examples and tests needed for the independence assessment.
- **History entry points:** commits, any pull requests, issues and tags, with totals recorded.
- **Controlling standards:**
  - [CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) and [Part 2](https://docs.ogc.org/is/23-002/23-002.html), with their abstract tests, schemas and examples;
  - [OGC API – Common Part 1](https://docs.ogc.org/is/19-072/19-072.html) and [Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) where incorporated;
  - [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html) and [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) where modelled;
  - the official draft Part 3 material, for the MQTT publish/subscribe module;
  - the [upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), limited to implicated entries.

---

## 4. Supporting Resources

- [IDR-SRV-014B CS-GO implementation study](../IDR%20Reports/idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md) and [IDR-SRV-062 CS-GO engineering study](../IDR%20Reports/idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md): the same author's server, and what is already known about their practices.
- The IDR-SRV-063 and IDR-SRV-064 reports, once accepted: contrasting client-library evidence from the OSH ecosystem.
- [IDR-SRV-056](../IDR%20Reports/idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md), [014E](../IDR%20Reports/idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md) and [014G](../IDR%20Reports/idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md): existing client findings to reconcile.
- [IDR-SRV-011](../IDR%20Reports/idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md), [012](../IDR%20Reports/idr-srv-012-content-negotiation-media-types-and-encoding-selection-report.md), [013](../IDR%20Reports/idr-srv-013-error-model-http-status-codes-and-failure-semantics-report.md) and [053](../IDR%20Reports/idr-srv-053-test-data-fixtures-golden-files-and-scenario-corpus-strategy-report.md): Glaux's accepted query, negotiation, error and fixture baselines.
- Current planning: [Goal v1.10](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md), [Guide v1.21](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md) (§§4.1–4.9, 6.2–6.4, 8.1–8.2, 13), [Roadmap v1.40](../../../../../Plans/glaux-server/glaux-server-roadmap.md) and the [Phase 1 review charter](../../../../../Plans/glaux-server/Implementation-Reviews/Phase-1/README.md).
- Implemented Glaux behaviour for comparison: server [`docs/system-create.md`](https://github.com/DGIWG-P507/glaux-server/blob/main/docs/system-create.md), [`docs/system-read.md`](https://github.com/DGIWG-P507/glaux-server/blob/main/docs/system-read.md) and [`docs/discovery.md`](https://github.com/DGIWG-P507/glaux-server/blob/main/docs/discovery.md) at the recorded server commit.

---

## 5. Research Methodology

### Phase 1: Establish identity, independence and coverage

**Objective:** Pin exactly what is studied, and establish how far it is independent evidence.

**Tasks:**
1. Refresh and record the repository commit, npm versions and tarball contents, and the retrieval date. Record any difference between the repository and the published package.
2. Inventory the source and test trees. Map the public API surface to resource families and operations, and identify the targeted Parts and versions.
3. Inventory history with totals, record the shallow-history limit explicitly, and select the two stated departures from the standard as history cases.
4. Classify the origin of representative fixtures and type definitions (standard example, CS-GO-derived, hand-written, unknown) by comparing them with the standard and with the pinned CS-GO representations.
5. Consult only implicated entries of the standards-history register.

**Expected Output:** A coverage inventory with pins, a Part/version map, and an independence assessment with explicit limits, within the developing report.

### Phase 2: Trace requests and response models

**Objective:** Answer Q2 and Q3 from source.

**Tasks:**
1. For each public operation, trace URL construction, parameters, headers and body through to the network call.
2. For each response model and parser, record the members, links and formats required or optional, the handling of unknown members, and coercions.
3. Examine errors, paging, retries, authentication and any streaming support.
4. Build a dependency table with source anchors at the pinned commit.

**Expected Output:** A traceable dependency table and walkthroughs of representative operations.

### Phase 3: Judge against the standard and examine tests

**Objective:** Answer Q4 and Q5, and draft Q6.

**Tasks:**
1. Map each material assumption to exact requirement identifiers and classify it. Note shared CS-GO/library readings that differ from the standard or from Glaux.
2. Read representative tests at each layer, and assess expected-value independence and what each would detect.
3. Check tool availability without installing anything. Run the test suite only in an already permitted, isolated environment, recorded exactly. Otherwise state source inspection as the limit.
4. Compare the library's Phase 1 expectations with Glaux's implemented System create/read behaviour and tests at the recorded server commit.

**Expected Output:** A standards-classified dependency table, a test and fixture analysis, and a Phase 1 comparison with explicit limits.

### Phase 4: Synthesis

**Objective:** Produce one readable, decision-usable report in the existing template.

**Tasks:**
1. Answer Q1–Q6, keeping observed source, attributed claims, inference and recommendation distinguishable.
2. Put the most useful findings first: Phase 1 expectations, independence limits and later-gate checks. Recommend on IDR-SRV-056 inclusion, and record the handoff to IDR-SRV-066.
3. Map each lesson to Guide sections and Roadmap task IDs, with a disposition.
4. Validate coverage, references, attribution and success criteria. Prepare the acceptance handoff without editing downstream artifacts.

**Expected Output:** One research report for project-lead review. If more than one iteration is needed, keep it explicitly in progress and resume only on the next `proceed`.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Q1–Q6 have evidence-backed answers, or explicit limitations and their consequences.
- [ ] Both baselines are pinned (npm `0.1.3` from `4724b0e` as primary, and the unpublished head `a632798`), and every material difference between them is recorded.
- [ ] Independence from CS-GO and AI-assisted sources is assessed from evidence. Fixture origins are classified, and the shallow-history limit is stated.
- [ ] Every material request and model assumption is traceable to a source anchor and classified against exact standard identifiers.
- [ ] Test analysis explains what representative tests detect and whether their expected values are independent. Source inspection is distinguished from execution.
- [ ] No source, type or fixture is copied into Glaux, and recommendations describe behaviour in Glaux's own terms.
- [ ] Phase 1 expectations are compared with implemented Glaux behaviour, later-gate checks are listed, and the IDR-SRV-066 handoff is recorded.
- [ ] Each material lesson has a Glaux disposition with Guide/Roadmap references. "No change" is acceptable.
- [ ] Relevant official repository history is consulted and authority-classified where the standards-history register applies.
- [ ] The report follows the report template and validates these criteria.

Completion does not require running the tests, contacting the author, reading every file or certifying the library. Any such limit narrows the relevant conclusion; it does not disappear from the report.

---

## 7. Deliverable

**Deliverable Name:** cs-client-ts Client Library Study - Research Report<br>
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-065-cs-client-ts-client-library-study-report.md`

Use the existing report template and its full section set. Put the dependency table, independence assessment and test examples in the corresponding sections or optional appendices.

For each material finding, capture this chain:
1. Library evidence at a pinned anchor.
2. Standard text and classification.
3. Independence note.
4. Glaux applicability.
5. Existing coverage or proposed bounded change.
6. Verification or review use.

Keep the plain-language summary readable on its own. The report is evidence, not a requirements document or a mandate to copy library behaviour.

---

## 8. Dependencies

### Must Complete Before Starting

**Internal project prerequisites (completion gates):**

- This plan and its registration in the [overall plan](overall-idr-research-plan.md) are published.
- The IDR-SRV-064 report is complete and accepted. This order is proposed for the project lead to confirm: it keeps one study running at a time and lets this study compare against the two OSH-family reports. The lead listed Aleph before cs-client-ts; this study comes first because Aleph depends on the library.
- The project lead's `proceed` authorises execution.

Internal prerequisites are not waived by labelling them unavailable or deferred. Reorder or change them only through an explicit, recorded update to the controlling overall plan or this plan's approved dependency record.

**External evidence prerequisites:**

- Public access to the repository, the npm package and the CS-GO comparison material. For inaccessible sources, record identity, attempted access, affected questions and the resulting limits. Do not infer contents.
- A permitted isolated environment is needed only for any execution actually performed.

### Blocks (What This Topic Unlocks)

- IDR-SRV-066 (Aleph), which depends on this library.
- The Phase 1 implementation review's remaining steps. The project lead decided on September 29, 2026 that steps 2–5 wait for IDR-SRV-063 to IDR-SRV-066.
- Any later, separately authorised discussion of IDR-SRV-056, Guide or Roadmap changes. No implementation, issue creation or planning edit happens in this study.

---

## 9. Research Status Checklist

- [ ] Phase 1 complete
- [ ] Phase 2 complete
- [ ] Phase 3 complete
- [ ] Phase 4 synthesis complete
- [ ] Deliverable draft complete
- [ ] Deliverable reviewed
- [ ] Deliverable accepted

**Actual Research Time:** [Fill in at completion]<br>
**Completion Date:** [Fill in at completion]

---

## 10. Notes and Open Questions

- The project lead reports that this developer is a senior developer who worked with AI assistance. That motivates a careful study; it is not evidence about any particular behaviour.
- A library written by the same author as CS-GO may agree with CS-GO by construction. The report must not count that agreement as a second, independent source.
- The library is young (created July 2026) and pre-1.0 (`0.1.3`). Later versions may differ, and later gate reviews should refresh the pin.
- Stop when the questions and representative operations are covered, with usable findings and explicit unknowns.

---

## References

- [Research Planning Approach](../../../../../Governance/research-planning-approach.md), [Initial Planning Guidance](../../../../../Governance/initial-planning-guidance.md), [Research Plan Template](../../../../../Governance/research-plan-template.md) and [Research Report Template](../../../../../Governance/research-report-template.md).
- [Controlling overall IDR plan](overall-idr-research-plan.md). This topic is registered as a supplement without reopening accepted topics.
- [Pinned OS4CSAPI research-plan exemplar corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans), used for structure only.
- The specific primary sources and accepted research inputs are listed in Sections 3–4.

[Lib]: https://github.com/SomethingCreativeStudios/cs-client-ts
[LibPin]: https://github.com/SomethingCreativeStudios/cs-client-ts/tree/a6327989f5cec3c36422a121a2fe676e8a6bbe6d
[LibNpm]: https://github.com/SomethingCreativeStudios/cs-client-ts/tree/4724b0e075f2d491885caa3824fcf0da23a464d1

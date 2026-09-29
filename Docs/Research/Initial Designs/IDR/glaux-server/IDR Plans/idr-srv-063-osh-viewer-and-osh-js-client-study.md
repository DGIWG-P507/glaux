# Section 063: OSH Viewer and OSH JS Toolkit Client Study - Research Plan

**Topic ID:** IDR-SRV-063<br>
**Status:** Planned<br>
**Last Updated:** September 29, 2026<br>
**Estimated Research Time:** Not yet calibrated; four bounded phases below, with further research iterations only if needed for the stated coverage.<br>
**Actual Research Time:** TBD until complete<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-063-osh-viewer-and-osh-js-client-study-report.md`

---

## Usage Instructions

Use the existing [Research Plan Template](../../../../../Governance/research-plan-template.md), and for the later report the [Research Report Template](../../../../../Governance/research-report-template.md), preserving their section order. The pinned OS4CSAPI exemplars inform question-led investigation, concrete examples and practical recommendations. Their client-specific scope, metrics and estimates are not Glaux requirements.

The project lead requested this study on September 29, 2026. It is one of four separate client studies (IDR-SRV-063 to IDR-SRV-066), each planned, executed and reported individually before the [Phase 1 implementation review](../../../../../Plans/glaux-server/Implementation-Reviews/Phase-1/README.md) resumes. Each study starts on its own `proceed` after this plan is published.

---

## 1. Research Objective

Determine what the **OpenSensorHub (OSH) Viewer** and the **OSH JS Toolkit (`osh-js`)** that it depends on show about how a mature, largely human-developed client family actually uses a Connected Systems-style API. Establish:
- which requests they send;
- which response members, links, identifiers, media types and behaviours they rely on;
- where they are strict or tolerant;
- which version of the API they target.

Judge each observed dependency against the approved OGC API – Connected Systems (CSAPI) text.

Produce independent evidence that Glaux can use in three places:
- the Phase 1 implementation review (especially step 2, the test-source audit, and step 5, the comparison with OpenSensorHub);
- later review gates;
- the external-client matrix of [IDR-SRV-056](../IDR%20Reports/idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md).

Each material finding must say whether current Glaux planning already covers it, whether a bounded change is worth discussing, whether it is not applicable, or whether the evidence is insufficient.

### Why This Topic Order

Glaux's existing client evidence depends heavily on sources the project lead reports are mostly AI-written: the OS4CSAPI TypeScript client named in Guide §8.1, and the unofficial Botts TEAM Engine suite. The project lead reports that the OSH Viewer and the client libraries behind it are the most human-written CSAPI clients available.

`osh-js` has public history from 2015. Its main contributor has over 1,000 commits. It is the natural baseline for the OSCAR Viewer study (IDR-SRV-064), which depends on a fork of the same library.

Studying this family first gives the later studies a reference point, and gives the review a source of expected client behaviour that no Glaux assistant wrote.

### Critical Constraints

- **Establish the API version first.**
  - At the preliminary check, the viewer's CSAPI-related code lives under `source/core/sweapi/` in `osh-js`. "SWE API" is an earlier name used during CSAPI's development.
  - The study must determine whether the pinned code targets published CSAPI Part 1/Part 2, a pre-publication draft, an OSH-specific extension, or a mixture.
  - A draft-era behaviour is historical client evidence, not an expectation Glaux must meet.
- **Pin moving references.**
  - The viewer's `package.json` points to `osh-js` through a moving branch (`#mcs_baseline`).
  - The viewer's last commit is `052befa09b8f076520bff77ef5d39f6624b6411d` (April 10, 2024), but that branch's head is `d3aa99cef7de0b4e6f928b1a77f07b898f6747e9` (April 20, 2026).
  - Determine which `osh-js` revision the viewer actually resolved from its lockfile or build records. Where that cannot be established, study the dated branch head as a separate, explicitly labelled baseline. Never silently assume the two are the same.
- **Treat this as informative evidence.**
  - Client behaviour, comments, tests and omissions do not replace the standard or change Glaux's approved scope.
  - Where a client and the standard disagree, the standard wins and the disagreement is recorded.
  - A client tolerating a server behaviour is not proof that the behaviour conforms.
- **Keep authorship claims attributed.**
  - Separate the project lead's report of human authorship, observable repository history, and analyst inference.
  - Do not infer AI use or non-use from style, speed or commit patterns. Absence of AI-guidance files is not evidence either way.
- **Licensing.**
  - Both repositories are MPL-2.0. Reading and analysis are permitted.
  - Copying source or fixtures into Glaux requires a separate licensing decision. This study copies nothing.
- **Execution.**
  - No software installation on the project lead's company laptop.
  - Running the viewer, toolkit tests or builds requires an already permitted, isolated environment and must be recorded exactly. Otherwise the study is source inspection, and its conclusions say so.
  - No contact with maintainers, no issues or comments upstream, and no use of external servers or real data.
- **Scope.**
  - Study the CSAPI-facing path and what it needs. `osh-js` has about 1,900 files at the branch head, including SOS, video, map and chart components.
  - Inventory the rest only enough to bound it. Do not audit the whole toolkit.

---

## 2. Research Questions

### Core Questions

1. **Q1 — Identity, version and provenance:**
   - Exactly which code does the OSH Viewer run?
   - Which API version or draft does it target?
   - What does the public history show about who wrote it and how it evolved?
2. **Q2 — Requests:** Which endpoints, methods, query parameters, headers, media types, authentication and streaming or connection mechanisms does the client use, and in what order does it discover them?
3. **Q3 — Response dependencies:**
   - Which response members, identifiers, links and link relations, geometry and time formats, paging and error behaviours does the client read or require?
   - Where is it strict, tolerant or silently lossy?
4. **Q4 — Standards alignment:** For each material dependency, what do CSAPI Parts 1 and 2 (and the shared OGC API foundations) require? Classify the client behaviour as conforming, tolerant, draft-era, OSH-specific or non-conforming.
5. **Q5 — Tests and quality evidence:**
   - Which tests, examples or showcase applications exercise the CSAPI path?
   - What do they establish?
   - Are their expected values independent of the code they check?
6. **Q6 — Transfer to Glaux:** What concrete expectations, fixtures or checks should inform:
   - the Phase 1 review;
   - later gates;
   - IDR-SRV-056's client matrix;
   - the Guide and Roadmap?

   Give an explicit disposition for each.

### Detailed Questions

**Identity and history (Q1)**

- Record the viewer's default branch and commit, and the `osh-js` revision actually used, as above. Record the release tags and published package versions that correspond.
- Establish where `sweapi` code first appears, how it changed, and whether any later branch renames or restructures it toward published CSAPI.
- Summarise authorship from commit, contributor and pull-request records. Distinguish long-standing maintainers, occasional contributors and bots. Record any stated use of tools or assistants without inferring unstated use.

**Requests and discovery (Q2)**

- Trace, from viewer configuration and UI actions, every request the CSAPI path can send: resource families (Systems, Deployments, Procedures, Sampling Features, Properties, Datastreams, Observations, Control Streams, Commands, System Events), nested routes, identifiers and query parameters.
- Record the parameters used, such as limit/paging, time, spatial bounds, filters, `f` or `Accept`, and select or properties.
- Record how the root URL, API description, conformance declaration or collections are used, if at all. Record whether the client hard-codes paths or discovers them from links.
- Record content types sent and accepted, authentication methods, websocket/MQTT or other streaming use, and SWE Common JSON, Text or Binary handling.

**Response dependencies and tolerance (Q3)**

- For each read path, list the members the client reads and whether their absence breaks, degrades or is ignored.
  - Include `id` and `uid` handling.
  - Include `links` and the relation values followed.
  - Include GeoJSON versus SensorML JSON use.
  - Include `featureType` or `definition`, time instants and intervals, and schema/datastream structure.
- Examine paging (`next` links, `numberMatched`/`numberReturned`), error responses and retries. Record anything the client assumes that the standard leaves optional.
- Identify silent loss or reinterpretation, such as dropped members, unit or time conversions, or coordinate order.

**Standards alignment (Q4)**

- Map each material dependency to the relevant CSAPI Part 1 or Part 2 requirement, recommendation or permission, and to OGC API Common/Features where incorporated. Cite exact identifiers.
- Consult only the entries of the [upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md) that the findings implicate. Classify issue and pull-request evidence as informative.
- Where a Glaux Guide interpretation (§13) already addresses the point, state whether the client agrees, disagrees or is silent.

**Tests and examples (Q5)**

- Inventory the tests, examples and showcase applications that touch the CSAPI path. For representative ones, explain setup, input, expected result, assertion and what failure it would catch.
- Distinguish tests that run against a live OSH server, recorded responses, or no server at all. Do not treat an example application as a test unless it asserts behaviour.

**Transfer to Glaux (Q6)**

- For Phase 1 (System create/read), list the client's expectations for a single System representation and its discovery. Say whether current Glaux behaviour and tests meet, exceed or conflict with them.
- For later phases, list the resource-specific expectations the corresponding gate reviews should check.
- Recommend whether this client, or a pinned subset of its behaviour, should be added to IDR-SRV-056's matrix. Give the pin, the environment it needs and its limitations.
- Compare each lesson with the existing Guide section and three-level Roadmap task before proposing work. "No change" is an acceptable outcome.

---

## 3. Primary Resources

- **OSH Viewer:** [opensensorhub/osh-viewer][Viewer] at default branch `main`, commit [`052befa09b8f076520bff77ef5d39f6624b6411d`][ViewerPin] (preliminary check, September 29, 2026). Includes `package.json`, any lockfile, `src/` (including `src/components/systems/`), build configuration and README. Record the actual execution snapshot.
- **OSH JS Toolkit:** [opensensorhub/osh-js][OshJs].
  - Branch `mcs_baseline`, head [`d3aa99cef7de0b4e6f928b1a77f07b898f6747e9`][OshJsPin] (April 20, 2026).
  - The revision the viewer actually resolved, if different.
  - Default branch `master`, head `d7ba5bace3abb81cfe7fcd2791600cdd4a10620f` (November 26, 2021), for comparison.
  - Start from `source/core/sweapi/` and follow its data sources, parsers and connectors. Discover actual paths from the tree.
- **History entry points:** commits, pull requests, issues, releases and published npm versions for both repositories, with pagination and totals recorded.
- **Controlling standards:**
  - [CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) and [Part 2](https://docs.ogc.org/is/23-002/23-002.html), with their abstract test suites, schemas and examples;
  - [OGC API – Common Part 1](https://docs.ogc.org/is/19-072/19-072.html) and [OGC API – Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) where incorporated;
  - [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html) and [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) where the client parses them;
  - the [upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), limited to implicated entries.

---

## 4. Supporting Resources

- [IDR-SRV-014A OSH server study](../IDR%20Reports/idr-srv-014a-osh-csapi-server-implementation-study-report.md): the server side of this ecosystem. Distinguish OSH server behaviour from client expectations.
- [IDR-SRV-056 external-client matrix](../IDR%20Reports/idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md), [IDR-SRV-014E client smoke tests](../IDR%20Reports/idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md) and [IDR-SRV-014G community lessons](../IDR%20Reports/idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md): existing client findings to reconcile, not repeat.
- [IDR-SRV-009](../IDR%20Reports/idr-srv-009-landing-page-api-definition-and-conformance-declaration-behavior-report.md), [011](../IDR%20Reports/idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md), [012](../IDR%20Reports/idr-srv-012-content-negotiation-media-types-and-encoding-selection-report.md) and [013](../IDR%20Reports/idr-srv-013-error-model-http-status-codes-and-failure-semantics-report.md): Glaux's accepted discovery, query, negotiation and error baselines.
- The OS4CSAPI [oscar-viewer analysis](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/main/docs/research/requirements/csapi-oscarviewer-analysis.md) listed in the references register.
  - It concerns the related OSCAR Viewer, and comes from a project the project lead reports is mostly AI-written.
  - Use it only as a pointer to places to check. It is not evidence.
- Current planning: [Goal v1.10](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md), [Guide v1.21](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md) (§§4.1–4.4, 6.2–6.4, 8.1–8.2, 13), [Roadmap v1.39](../../../../../Plans/glaux-server/glaux-server-roadmap.md) and the [Phase 1 review charter](../../../../../Plans/glaux-server/Implementation-Reviews/Phase-1/README.md).
- Implemented Glaux behaviour for comparison: server [`docs/system-create.md`](https://github.com/DGIWG-P507/glaux-server/blob/main/docs/system-create.md), [`docs/system-read.md`](https://github.com/DGIWG-P507/glaux-server/blob/main/docs/system-read.md) and [`docs/discovery.md`](https://github.com/DGIWG-P507/glaux-server/blob/main/docs/discovery.md) at the recorded server commit.

---

## 5. Research Methodology

### Phase 1: Establish identity, version and coverage

**Objective:** Pin exactly what is studied and bound the investigation.

**Tasks:**
1. Refresh and record repository identities, default branches, the pinned commits above and the retrieval date. Resolve the viewer's actual `osh-js` revision from its lockfile or build configuration, and record how.
2. Inventory both trees. Mark the CSAPI-facing path (viewer components, `sweapi` modules, parsers, data sources) separately from unrelated toolkit areas that receive inventory-level mention only.
3. Determine the targeted API version. Compare endpoint names, member names and media types with published CSAPI and with earlier draft terminology. Check the history of `sweapi` for renames or restructuring.
4. Screen commit, contributor, pull-request, issue and release inventories with pagination and totals. Select representative history cases for the CSAPI path, such as introduction, a behaviour change and a bug fix.
5. Consult only implicated entries of the standards-history register.

**Expected Output:** A compact coverage inventory with pins, an API-version finding and a justified case list, within the developing report.

### Phase 2: Trace requests and response dependencies

**Objective:** Answer Q2 and Q3 from source.

**Tasks:**
1. Starting from viewer entry points and configuration, trace each CSAPI request through the toolkit to the network call. Record method, path pattern, parameters, headers and body.
2. For each response handled, record the members, links and formats read. Record whether each is required, optional with a fallback, or ignored, and what happens when it is missing or unexpected.
3. Examine discovery, paging, error handling, retries, authentication and streaming. Record hard-coded assumptions.
4. Build a request/response dependency table and link each row to its source anchor at the pinned commit.

**Expected Output:** A traceable dependency table and short walkthroughs of representative flows.

### Phase 3: Judge against the standard and examine tests

**Objective:** Answer Q4 and Q5, and draft Q6.

**Tasks:**
1. Map each material dependency to exact CSAPI (and incorporated Common/Features) requirement identifiers, and classify it as conforming, tolerant, draft-era, OSH-specific or non-conforming. Cross-check Glaux Guide §13 interpretations.
2. Inventory and read the representative tests and examples on the CSAPI path. Explain what each would detect and whether its expected values are independent.
3. Check tool availability without installing anything. Run the viewer or toolkit tests only in an already permitted, isolated environment, recording revision, command, dependencies and results. Otherwise record source inspection as the limit of the evidence.
4. Compare the client's Phase 1 expectations with current Glaux System create/read behaviour and tests at the recorded server commit.

**Expected Output:** A standards-classified dependency table, test analysis, and a Phase 1 comparison with explicit evidence limits.

### Phase 4: Synthesis

**Objective:** Produce one readable, decision-usable report in the existing template.

**Tasks:**
1. Answer Q1–Q6, keeping observed source, attributed claims, inference and recommendation distinguishable.
2. State the most useful findings first, especially Phase 1 expectations and gate-review checks. Include a recommendation on adding this client to IDR-SRV-056's matrix.
3. Map each material lesson to existing Guide sections and Roadmap task IDs, with a disposition.
4. Check coverage, references, attribution and success criteria. Prepare the acceptance handoff without editing downstream artifacts.

**Expected Output:** One research report for project-lead review. If more than one iteration is needed, keep the report explicitly in progress, record the phases completed, and resume only on the next `proceed`.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Q1–Q6 have evidence-backed answers, or explicit limitations and their consequences.
- [ ] The studied revisions are pinned, including the viewer's actual `osh-js` revision or a documented reason it cannot be established.
- [ ] The targeted API version is established or explicitly left unresolved.
- [ ] Every material request and response dependency is traceable to a source anchor and classified against exact standard identifiers.
- [ ] Draft-era and OSH-specific behaviour is kept separate from published-standard expectations.
- [ ] Test and example analysis distinguishes assertions from demonstrations, and source inspection from execution.
- [ ] Authorship statements are attributed, and no AI-use inference is made.
- [ ] Phase 1 expectations are compared with implemented Glaux behaviour, and later-gate checks are listed.
- [ ] Each material lesson has a Glaux disposition with Guide/Roadmap references. "No change" is acceptable.
- [ ] The report follows the report template and validates these criteria.

Completion does not require running the viewer, contacting maintainers, reading every toolkit file, or certifying the client. Any such limit narrows the relevant conclusion; it does not disappear from the report.

---

## 7. Deliverable

**Deliverable Name:** OSH Viewer and OSH JS Toolkit Client Study - Research Report<br>
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-063-osh-viewer-and-osh-js-client-study-report.md`

Use the existing report template, including:
- executive summary;
- question coverage;
- evidence base;
- findings;
- decision analysis;
- recommendations;
- implementation implications;
- risks and unknowns;
- success-criteria validation;
- handoff.

Put the dependency table, history cases and test examples in the corresponding sections or optional appendices.

For each material finding, capture this chain:
1. Client evidence at a pinned anchor.
2. Standard text and classification.
3. Glaux applicability.
4. Existing coverage or proposed bounded change.
5. Verification or review use.

Keep the plain-language summary readable on its own.

The report is evidence for the Phase 1 review and later gates, and for any later discussion of IDR-SRV-056, Guide or Roadmap changes. It is not a requirements document or a mandate to copy client behaviour.

---

## 8. Dependencies

### Must Complete Before Starting

**Internal project prerequisites (completion gates):**

- This plan and its registration in the [overall plan](overall-idr-research-plan.md) are published, and the project lead's `proceed` authorises execution.
- The original IDR, supplements 058–062 and the current Goal/Guide/Roadmap baseline are complete. No prerequisite exception is proposed.

Internal prerequisites are not waived by labelling them unavailable or deferred. Reorder or change them only through an explicit, recorded update to the controlling overall plan or this plan's approved dependency record.

**External evidence prerequisites:**

- Public access to both repositories, their history and published packages. For inaccessible sources, record identity, attempted access, affected questions and the resulting limits. Do not infer contents.
- A permitted isolated environment is needed only for any execution actually performed, not for source-level findings. A missing environment is an honest limitation, not a reason for implicit installation.

### Blocks (What This Topic Unlocks)

- IDR-SRV-064 (OSCAR Viewer), which compares its `osh-js` fork with this baseline.
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

- The project lead reports that the OSH Viewer and OSCAR Viewer are among the most human-written CSAPI clients. That motivates the study; it is not itself evidence about any particular behaviour.
- The viewer was last changed in April 2024, so it may predate parts of the published standard. Findings about an older draft are still useful as history, but must not be read as current requirements.
- The moving `mcs_baseline` reference means the viewer's behaviour depends on which toolkit revision is built. The report must make that explicit.
- Stop when the questions and representative flows are covered, with usable findings and explicit unknowns. Broader investigation needs a demonstrated decision need.

---

## References

- [Research Planning Approach](../../../../../Governance/research-planning-approach.md), [Initial Planning Guidance](../../../../../Governance/initial-planning-guidance.md), [Research Plan Template](../../../../../Governance/research-plan-template.md) and [Research Report Template](../../../../../Governance/research-report-template.md).
- [Controlling overall IDR plan](overall-idr-research-plan.md). This topic is registered as a supplement without reopening accepted topics.
- [Pinned OS4CSAPI research-plan exemplar corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans), used for structure only.
- The specific primary sources and accepted research inputs are listed in Sections 3–4.

[Viewer]: https://github.com/opensensorhub/osh-viewer
[ViewerPin]: https://github.com/opensensorhub/osh-viewer/tree/052befa09b8f076520bff77ef5d39f6624b6411d
[OshJs]: https://github.com/opensensorhub/osh-js
[OshJsPin]: https://github.com/opensensorhub/osh-js/tree/d3aa99cef7de0b4e6f928b1a77f07b898f6747e9

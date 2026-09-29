# Section 064: OSCAR Viewer Client Study - Research Plan

**Topic ID:** IDR-SRV-064<br>
**Status:** Research complete; report in review; project-lead acceptance pending<br>
**Last Updated:** September 29, 2026<br>
**Estimated Research Time:** Not yet calibrated; four bounded phases below, with further research iterations only if needed for the stated coverage.<br>
**Actual Research Time:** About 25 minutes of research and drafting, 21:09–21:35 UTC September 29, 2026, in one AI-assisted iteration, before separate review. This is not a human-hours estimate. Nothing was executed or installed.<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-064-oscar-viewer-client-study-report.md`

---

## Usage Instructions

Use the existing [Research Plan Template](../../../../../Governance/research-plan-template.md), and for the later report the [Research Report Template](../../../../../Governance/research-report-template.md), preserving their section order. The pinned OS4CSAPI exemplars inform question-led investigation, concrete examples and practical recommendations. Their client-specific scope, metrics and estimates are not Glaux requirements.

The project lead requested this study on September 29, 2026. It is one of four separate client studies (IDR-SRV-063 to IDR-SRV-066), each planned, executed and reported individually before the [Phase 1 implementation review](../../../../../Plans/glaux-server/Implementation-Reviews/Phase-1/README.md) resumes. It starts on its own `proceed`, after IDR-SRV-063's report is accepted.

---

## 1. Research Objective

Determine what the **OSCAR Viewer**, and the Connected Systems data source in the `osh-js` fork it depends on, show about how an actively developed, largely human-written application uses the Connected Systems API (CSAPI). Establish:
- which requests it sends;
- which response members, links, identifiers, media types, streaming and command behaviours it relies on;
- where it is strict or tolerant;
- how its Connected Systems support differs from the OSH Toolkit baseline in IDR-SRV-063.

Judge each observed dependency against the approved CSAPI text. Produce independent evidence for:
- the Phase 1 implementation review;
- later review gates;
- the external-client matrix of [IDR-SRV-056](../IDR%20Reports/idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md).

Give each material finding a Glaux disposition: already covered, a bounded change worth discussing, not applicable, or insufficient evidence.

### Why This Topic Order

The project lead reports that the OSCAR Viewer is among the most human-written CSAPI clients, and that it may share the OSH Viewer's library. The preliminary check confirms a shared lineage but not a shared revision:
- OSCAR depends on `github:earocorn/osh-js#add-consys`, a fork of `opensensorhub/osh-js`.
- That fork adds data sources named for the Connected Systems API (`consysapi`) alongside the toolkit's earlier `sweapi` modules.

Studying OSCAR after IDR-SRV-063 lets the report state exactly what the fork adds or changes. OSCAR is also much more recently active than the OSH Viewer (last commit September 23, 2026, and 866 reachable commits), so it is the better indicator of current CSAPI client practice in this ecosystem.

### Critical Constraints

- **Pin the moving fork reference.**
  - OSCAR's `package.json` names a branch of a personal fork, not a release.
  - Determine the revision OSCAR actually resolved from its lockfile. At the preliminary check, the branch head is [`73dacad5c7338722763263ef0ff5928fbd19de5b`][ForkPin] (August 22, 2026).
  - If the two differ, study the resolved revision and label the branch head as a separate baseline.
- **Establish the API version per data source.**
  - Distinguish published CSAPI Part 1 and Part 2 behaviour, pre-publication or `sweapi`-era behaviour, OSH-specific extensions, and draft Part 3 or other experimental features.
  - Draft-era or OSH-specific behaviour is evidence of practice, not a Glaux requirement.
- **Treat this as informative evidence.**
  - Where OSCAR and the standard disagree, the standard wins and the disagreement is recorded.
  - A client tolerating a server behaviour is not proof that the behaviour conforms.
- **Keep the existing analysis in its place.**
  - The references register lists an [OSCAR Viewer analysis](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/main/docs/research/requirements/csapi-oscarviewer-analysis.md) from the OS4CSAPI project, whose TypeScript client the project lead reports is mostly AI-written. Its analysis documents are treated with the same caution.
  - Use it only as a list of places to check. Confirm or reject each point from OSCAR's own source.
- **Keep authorship claims attributed.**
  - Separate the project lead's report, observable history (for example, `kalynstricklin` 485 commits, `tipatterson-dev` 133, `earocorn` 87 at the preliminary check) and inference.
  - Do not infer AI use or non-use.
- **Licensing.**
  - OSCAR and the fork are MPL-2.0. Reading and analysis are permitted.
  - Reuse of source or fixtures needs a separate licensing decision.
- **Execution and domain data.**
  - No installation on the project lead's company laptop.
  - Run anything only in an already permitted, isolated environment, recorded exactly.
  - OSCAR's application domain may involve sensitive operational concepts. Use only public repository material and synthetic data, and do not reproduce operational configuration beyond what is needed to explain a CSAPI dependency.
  - No maintainer contact and no upstream posts.
- **Scope.** Study OSCAR's CSAPI-facing code and the fork's Connected Systems additions. Inventory the rest of OSCAR (UI, maps, video, charts) and the rest of the fork only enough to bound the study.

---

## 2. Research Questions

### Core Questions

1. **Q1 — Identity, version and lineage:**
   - Exactly which OSCAR and fork revisions are studied?
   - Which API version does each data source target?
   - What does the fork add to, or change in, the IDR-SRV-063 toolkit baseline?
2. **Q2 — Requests:** Which endpoints, methods, parameters, headers, media types, authentication, streaming and command operations does OSCAR use, and how does it discover them?
3. **Q3 — Response dependencies:**
   - Which response members, identifiers, links, relations, time and geometry formats, datastream schemas, paging, status and error behaviours does it read or require?
   - Where is it strict, tolerant or silently lossy?
4. **Q4 — Standards alignment:** For each material dependency, what does CSAPI (and incorporated Common/Features, SWE Common and SensorML) require? Classify the behaviour as conforming, tolerant, draft-era, OSH-specific, experimental or non-conforming.
5. **Q5 — Tests and quality evidence:** What do OSCAR's Cypress tests, and the fork's tests and examples on the Connected Systems path, establish? Are their expected values independent? At the preliminary check OSCAR has 10 end-to-end spec files (`all-specs.cy.tsx` imports six of the others, plus a commented-out import of `ServerPage.cy`) and 5 component spec files. It also has `GeneralTestingActions.tsx`, which contains tests that Cypress's default spec pattern does not pick up, and two scheduled run logs from April 22, 2026.
6. **Q6 — Transfer to Glaux:** Which concrete expectations, fixtures or checks should inform:
   - the Phase 1 review;
   - later gates, especially those covering Part 2 dynamic data, commands and events;
   - IDR-SRV-056;
   - the Guide and Roadmap?

   Give an explicit disposition for each.

### Detailed Questions

**Identity, lineage and history (Q1)**

- Record OSCAR's default branch and commit, the resolved fork revision, and the upstream `osh-js` point where the fork diverged. At the preliminary check the fork's merge base with `mcs_baseline` is `549c630`, one commit before IDR-SRV-063's 2024 baseline `8a959d4`.
- Summarise the fork's Connected Systems changes relative to IDR-SRV-063's baseline: new or renamed modules, removed assumptions and changed parsers.
- Summarise OSCAR's history for the CSAPI path from commits, pull requests, issues and releases. Choose representative cases: introduction of Connected Systems support, a behaviour change and a defect fix.

**Requests and discovery (Q2)**

- Trace every CSAPI request OSCAR can send, from configuration and UI actions through `src/lib/data/osh/` and the fork's data sources. Cover:
  - resource families and nested routes;
  - identifiers, query parameters, time and spatial filters, paging, format selection and headers;
  - request bodies for any writes or commands.
- Record whether OSCAR uses the root, API description, conformance or collections, or hard-codes paths. Record its authentication method and any streaming protocol (websocket, MQTT or other).

**Response dependencies and tolerance (Q3)**

- For each read or stream path, list the members and links read, and what happens when each is missing, extra or malformed. Include:
  - `id`/`uid`;
  - `links` and relation values;
  - GeoJSON versus SensorML JSON;
  - System type or kind;
  - datastream `schema` and result structure;
  - observation `phenomenonTime`/`resultTime`;
  - command status and result;
  - System Events.
- Examine paging, error handling, retries and reconnection. Record assumptions the standard leaves optional, and any silent loss or reinterpretation.

**Standards alignment (Q4)**

- Map each material dependency to exact CSAPI Part 1 and Part 2 requirement, recommendation or permission identifiers, and to incorporated Common/Features, SWE Common and SensorML where parsed.
- Consult only implicated entries of the [upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md). Classify issue and pull-request evidence as informative.
- Compare with Glaux Guide §13 interpretations and with the draft Part 3 boundary in Guide §4.8.

**Tests and examples (Q5)**

- For representative OSCAR Cypress tests, and the fork's tests and `consysapi` showcase examples, explain setup, input, expected result, assertion and the failure each would catch. Distinguish live-server tests, recorded responses, mocks and demonstration-only examples.
- Use the two scheduled Cypress run logs as recorded execution evidence: state what ran, against which server, and what passed or failed. A log is evidence of that run only, not of the current code.

**Transfer to Glaux (Q6)**

- For Phase 1, list OSCAR's expectations for System discovery and a single System representation. Say whether Glaux's implemented create/read behaviour meets or conflicts with them.
- For later phases, list the checks each relevant gate review should include, with pins.
- Recommend whether OSCAR, or a pinned subset of its behaviour, should join IDR-SRV-056's matrix. Give the environment it needs and its limits.
- Compare each lesson with existing Guide sections and Roadmap task IDs before proposing work. "No change" is acceptable.

---

## 3. Primary Resources

- **OSCAR Viewer:** [Botts-Innovative-Research/oscar-viewer][Oscar] at default branch `main`, commit [`4b49c7cedc841c875dbb63613dfd21c34b8fa6ed`][OscarPin] (preliminary check, September 29, 2026). Includes `package.json`, the lockfile, `src/` (starting at `src/lib/data/osh/`), tests, build configuration and README. Record the actual execution snapshot.
- **The `osh-js` fork:**
  - [earocorn/osh-js][Fork], branch `add-consys`, head [`73dacad5c7338722763263ef0ff5928fbd19de5b`][ForkPin] (August 22, 2026), or the revision OSCAR actually resolved.
  - Its merge base with [opensensorhub/osh-js][OshJs].
  - The `consysapi` data sources and examples. Discover actual paths from the tree.
- **Comparison baseline:** the toolkit revision pinned by IDR-SRV-063.
- **History entry points:** commits, pull requests, issues and releases for OSCAR and the fork, with pagination and totals recorded.
- **Controlling standards:**
  - [CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) and [Part 2](https://docs.ogc.org/is/23-002/23-002.html), with their abstract tests, schemas and examples;
  - [OGC API – Common Part 1](https://docs.ogc.org/is/19-072/19-072.html) and [Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) where incorporated;
  - [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html) and [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) where parsed;
  - the official draft Part 3 material only where OSCAR uses it;
  - the [upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), limited to implicated entries.

---

## 4. Supporting Resources

- The IDR-SRV-063 report (OSH Viewer and OSH JS Toolkit), once accepted: the lineage baseline this study compares against.
- [IDR-SRV-014A OSH server study](../IDR%20Reports/idr-srv-014a-osh-csapi-server-implementation-study-report.md) and [IDR-SRV-014H draft Part 3 study](../IDR%20Reports/idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md): server and streaming context.
- [IDR-SRV-056](../IDR%20Reports/idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md), [014E](../IDR%20Reports/idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md) and [014G](../IDR%20Reports/idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md): existing client findings to reconcile.
- [IDR-SRV-034](../IDR%20Reports/idr-srv-034-datastream-observation-and-status-update-semantics-report.md), [035](../IDR%20Reports/idr-srv-035-streaming-and-event-publication-strategy-report.md), [036](../IDR%20Reports/idr-srv-036-control-stream-and-command-lifecycle-model-report.md), [011](../IDR%20Reports/idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md), [012](../IDR%20Reports/idr-srv-012-content-negotiation-media-types-and-encoding-selection-report.md) and [013](../IDR%20Reports/idr-srv-013-error-model-http-status-codes-and-failure-semantics-report.md): Glaux's accepted dynamic-data, streaming, command, query, negotiation and error baselines.
- The OS4CSAPI OSCAR analysis named in Section 1, used only as a list of places to check.
- Current planning: [Goal v1.10](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md), [Guide v1.21](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md) (§§4.1–4.9, 6.2–6.4, 8.1–8.2, 13), [Roadmap v1.40](../../../../../Plans/glaux-server/glaux-server-roadmap.md) and the [Phase 1 review charter](../../../../../Plans/glaux-server/Implementation-Reviews/Phase-1/README.md).
- Implemented Glaux behaviour for comparison: server [`docs/system-create.md`](https://github.com/DGIWG-P507/glaux-server/blob/main/docs/system-create.md), [`docs/system-read.md`](https://github.com/DGIWG-P507/glaux-server/blob/main/docs/system-read.md) and [`docs/discovery.md`](https://github.com/DGIWG-P507/glaux-server/blob/main/docs/discovery.md) at the recorded server commit.

---

## 5. Research Methodology

### Phase 1: Establish identity, lineage and coverage

**Objective:** Pin exactly what is studied, relate it to IDR-SRV-063, and bound the investigation.

**Tasks:**
1. Refresh and record repository identities, the pins above and the retrieval date. Resolve OSCAR's actual fork revision from its lockfile, and the fork's merge base with upstream.
2. Inventory OSCAR's and the fork's trees. Mark the CSAPI-facing path separately from areas that receive inventory-level mention only.
3. Diff the fork's Connected Systems additions against the IDR-SRV-063 baseline, and classify each data source's targeted API version.
4. Screen history inventories with pagination and totals, and select representative cases.
5. Consult only implicated entries of the standards-history register.

**Expected Output:** A coverage inventory with pins, a lineage comparison, an API-version classification per data source and a justified case list, within the developing report.

### Phase 2: Trace requests, streams and response dependencies

**Objective:** Answer Q2 and Q3 from source.

**Tasks:**
1. From OSCAR's entry points and configuration, trace each request, stream subscription and command through to the network call. Record method, path pattern, parameters, headers and body.
2. For each response or message handled, record the members, links and formats read, whether each is required, and what happens when it is missing or unexpected.
3. Examine discovery, paging, errors, retries, reconnection, authentication and command status handling.
4. Build a dependency table with source anchors at the pinned commits.

**Expected Output:** A traceable dependency table and walkthroughs of representative flows.

### Phase 3: Judge against the standard and examine tests

**Objective:** Answer Q4 and Q5, and draft Q6.

**Tasks:**
1. Map each material dependency to exact requirement identifiers and classify it. Cross-check Guide §13 and the Part 3 boundary.
2. Read the representative tests and examples, and explain what each detects and whether its expectations are independent.
3. Check tool availability without installing anything. Execute tests or the application only in an already permitted, isolated environment, recorded exactly. Otherwise state source inspection as the evidence limit.
4. Compare OSCAR's Phase 1 expectations with Glaux's implemented System create/read behaviour and tests at the recorded server commit.

**Expected Output:** A standards-classified dependency table, test analysis and a Phase 1 comparison with explicit limits.

### Phase 4: Synthesis

**Objective:** Produce one readable, decision-usable report in the existing template.

**Tasks:**
1. Answer Q1–Q6, keeping observed source, attributed claims, inference and recommendation distinguishable.
2. Put the most useful findings first, especially Phase 1 expectations, later-gate checks and the lineage differences from IDR-SRV-063. Recommend whether OSCAR should join IDR-SRV-056's matrix.
3. Map each lesson to Guide sections and Roadmap task IDs, with a disposition.
4. Validate coverage, references, attribution and success criteria. Prepare the acceptance handoff without editing downstream artifacts.

**Expected Output:** One research report for project-lead review. If more than one iteration is needed, keep it explicitly in progress and resume only on the next `proceed`.

---

## 6. Success Criteria

This topic research is complete when:

- [x] Q1–Q6 have evidence-backed answers, or explicit limitations and their consequences.
- [x] OSCAR's commit and its resolved fork revision are pinned, and the fork's differences from the IDR-SRV-063 baseline are stated.
- [x] Each data source's targeted API version is classified, and draft-era, OSH-specific and experimental behaviour is kept separate from published-standard expectations.
- [x] Every material request, stream and response dependency is traceable to a source anchor and classified against exact standard identifiers.
- [x] The existing OS4CSAPI analysis is used only as a pointer, and each point taken from it is confirmed or rejected from source.
- [x] Test and example analysis distinguishes assertions from demonstrations, and source inspection from execution.
- [x] Authorship statements are attributed, and no AI-use inference is made.
- [x] Phase 1 expectations are compared with implemented Glaux behaviour, and later-gate checks are listed.
- [x] Each material lesson has a Glaux disposition with Guide/Roadmap references. "No change" is acceptable.
- [x] Relevant official repository history is consulted and authority-classified where the standards-history register applies.
- [x] The report follows the report template and validates these criteria.

Completion does not require running OSCAR, contacting maintainers, reading every file or certifying the client. Any such limit narrows the relevant conclusion; it does not disappear from the report.

---

## 7. Deliverable

**Deliverable Name:** OSCAR Viewer Client Study - Research Report<br>
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-064-oscar-viewer-client-study-report.md`

Use the existing report template and its full section set. Put the dependency table, lineage comparison, history cases and test examples in the corresponding sections or optional appendices.

For each material finding, capture this chain:
1. Client evidence at a pinned anchor.
2. Standard text and classification.
3. Glaux applicability.
4. Existing coverage or proposed bounded change.
5. Verification or review use.

Keep the plain-language summary readable on its own. The report is evidence, not a requirements document or a mandate to copy client behaviour.

---

## 8. Dependencies

### Must Complete Before Starting

**Internal project prerequisites (completion gates):**

- This plan and its registration in the [overall plan](overall-idr-research-plan.md) are published.
- The IDR-SRV-063 report is complete and accepted, because this study compares the fork against that baseline.
- The project lead's `proceed` authorises execution.

Internal prerequisites are not waived by labelling them unavailable or deferred. Reorder or change them only through an explicit, recorded update to the controlling overall plan or this plan's approved dependency record.

**External evidence prerequisites:**

- Public access to OSCAR, the fork, upstream `osh-js` and their history. For inaccessible sources, record identity, attempted access, affected questions and the resulting limits. Do not infer contents.
- A permitted isolated environment is needed only for any execution actually performed.

### Blocks (What This Topic Unlocks)

- The Phase 1 implementation review's remaining steps. The project lead decided on September 29, 2026 that steps 2–5 wait for IDR-SRV-063 to IDR-SRV-066.
- Any later, separately authorised discussion of IDR-SRV-056, Guide or Roadmap changes. No implementation, issue creation or planning edit happens in this study.

---

## 9. Research Status Checklist

- [x] Phase 1 complete
- [x] Phase 2 complete
- [x] Phase 3 complete
- [x] Phase 4 synthesis complete
- [x] Deliverable draft complete
- [x] Deliverable reviewed (separate reviewer: `cbdd040` had six blocking corrections; `c21f1e6` was clean; the final record commit is confirmed in its PR; project-lead acceptance pending)
- [ ] Deliverable accepted

**Actual Research Time:** About 25 minutes of research and drafting, 21:09–21:35 UTC September 29, 2026, in one AI-assisted iteration, before separate review. This is not a human-hours estimate. Nothing was executed or installed.<br>
**Completion Date:** September 29, 2026 (research and report; acceptance pending)

---

## 10. Notes and Open Questions

- OSCAR is currently maintained, so its pinned snapshot will age. The report records the retrieval date, and later gate reviews should refresh the pin rather than rely on this snapshot indefinitely.
- The fork branch belongs to an individual contributor. Whether its Connected Systems work has been, or will be, merged upstream affects how representative it is. Record what the public record shows without predicting it.
- Stop when the questions and representative flows are covered, with usable findings and explicit unknowns.

---

## References

- [Research Planning Approach](../../../../../Governance/research-planning-approach.md), [Initial Planning Guidance](../../../../../Governance/initial-planning-guidance.md), [Research Plan Template](../../../../../Governance/research-plan-template.md) and [Research Report Template](../../../../../Governance/research-report-template.md).
- [Controlling overall IDR plan](overall-idr-research-plan.md). This topic is registered as a supplement without reopening accepted topics.
- [Pinned OS4CSAPI research-plan exemplar corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans), used for structure only.
- The specific primary sources and accepted research inputs are listed in Sections 3–4.

[Oscar]: https://github.com/Botts-Innovative-Research/oscar-viewer
[OscarPin]: https://github.com/Botts-Innovative-Research/oscar-viewer/tree/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed
[Fork]: https://github.com/earocorn/osh-js
[ForkPin]: https://github.com/earocorn/osh-js/tree/73dacad5c7338722763263ef0ff5928fbd19de5b
[OshJs]: https://github.com/opensensorhub/osh-js

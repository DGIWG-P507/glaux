# Section 066: Aleph (Alephex) Connected Systems UI Client Study - Research Plan

**Topic ID:** IDR-SRV-066<br>
**Status:** Research complete; report in review — October 3, 2026<br>
**Last Updated:** October 3, 2026<br>
**Estimated Research Time:** Not yet calibrated; four bounded phases below, with further research iterations only if needed for the stated coverage.<br>
**Actual Research Time:** One source-based AI research/report iteration, October 3, 2026; retrieval began 15:29 UTC, with drafting and separate review in the same iteration. No application or tests executed.<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-066-aleph-connected-systems-ui-client-study-report.md`

---

## Usage Instructions

Use the existing [Research Plan Template](../../../../../Governance/research-plan-template.md), and for the later report the [Research Report Template](../../../../../Governance/research-report-template.md), preserving their section order. The pinned OS4CSAPI exemplars inform question-led investigation, concrete examples and practical recommendations. Their client-specific scope, metrics and estimates are not Glaux requirements.

The project lead requested this study on September 29, 2026, naming the client "Aleph". Its public repository is `SomethingCreativeStudios/Alephex`, described as "Connected Systems UI". It is the last of four separate client studies (IDR-SRV-063 to IDR-SRV-066), each planned, executed and reported individually before the [Phase 1 implementation review](../../../../../Plans/glaux-server/Implementation-Reviews/Phase-1/README.md) resumes. It starts on its own `proceed`, after IDR-SRV-065's report is accepted.

---

## 1. Research Objective

Determine what **Aleph (Alephex)**, a recent Connected Systems web application built on `cs-api-client` (the `cs-client-ts` library of IDR-SRV-065), shows about the end-to-end expectations a real user-facing client places on a Connected Systems API (CSAPI) server. Establish:
- which user workflows it supports;
- which server capabilities each workflow needs, including discovery, reads, writes, filters, maps, streaming over MQTT and sign-in through OpenID Connect;
- how it handles missing, extra or failing server behaviour;
- which of its needs go beyond what the library itself provides.

Judge each material dependency against the approved CSAPI text. Produce independent evidence for:
- the Phase 1 implementation review;
- later review gates;
- the external-client matrix of [IDR-SRV-056](../IDR%20Reports/idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md).

Give each material finding a Glaux disposition: already covered, a bounded change worth discussing, not applicable, or insufficient evidence.

### Why This Topic Order

Aleph depends on `cs-api-client` at exactly version `0.1.3`, so it follows IDR-SRV-065. Its request-level behaviour can then be attributed correctly: to the library, or to the application's own additions.

It is the only studied client whose dependencies at the preliminary check include both MQTT (`mqtt`) and OpenID Connect (`oidc-client-ts`). Its MQTT use probably runs through the library's own publish/subscribe module (`cs-api-client/mqtt`), which IDR-SRV-065 studies. So this study's own contribution is how a real application wires streaming and sign-in together in user workflows, which later Glaux gates (Part 2 dynamic data, draft Part 3, security) will need.

### Critical Constraints

- **Attribute each behaviour to library or application.**
  - Every request-level finding must say whether it comes from `cs-api-client` `0.1.3`, as described in IDR-SRV-065, or from Aleph's own code.
  - Do not repeat the library study; refer to it.
- **Independence is limited, and the report must say so.**
  - The same developer wrote CS-GO, the library and this application, and the project lead reports AI assistance.
  - Agreement among the three is not independent confirmation, and the report must not treat it as such.
  - Do not infer which parts were AI-assisted.
- **Shallow history.** At the preliminary check the repository has 9 reachable commits and about 526 files, including 59 spec files (58 unit and component specs, and one Playwright end-to-end spec, `e2e/vue.spec.ts`) and 57 Storybook stories. Do not draw conclusions about process or evolution that the history cannot support.
- **Licensing.**
  - The repository shows no license file at the preliminary check. Reading and analysis are permitted.
  - Record the license status at execution. Any reuse follows the terms that apply to it.
- **Treat this as informative evidence.**
  - Where Aleph and the standard disagree, the standard wins and the disagreement is recorded.
  - Features beyond CSAPI, such as draft Part 3 streaming, OpenID Connect or UI conventions, are classified as such and are not Glaux requirements.
- **Execution and secrets.**
  - No installation on the project lead's company laptop.
  - Build, run or test Aleph (Vitest, Playwright, Storybook, Docker/Compose) only in an already permitted, isolated environment, with synthetic configuration, and record everything exactly. Otherwise the study is source inspection, stated as such.
  - Do not use real identity-provider credentials, real MQTT brokers or real sensor data. Do not reproduce values from `.env.example` or deployment files beyond variable names and their purpose.
  - No maintainer contact and no upstream posts.
- **Scope.** Study the CSAPI-facing workflows, the application's data layer above the library, streaming, authentication and error handling. Inventory purely presentational components and deployment packaging only enough to bound the study and to note operational assumptions a Glaux deployment would meet.

---

## 2. Research Questions

### Core Questions

1. **Q1 — Identity, scope and independence:**
   - Exactly which application revision and library version are studied?
   - What user workflows does Aleph support, and which CSAPI Parts and features do they involve?
   - How independent is its evidence, as far as the evidence shows?
2. **Q2 — Server capabilities per workflow:**
   - For each workflow, which requests, streams and authentication steps are needed?
   - Which come from the library, and which from Aleph's own code?
3. **Q3 — Response dependencies and failure handling:**
   - Which response members, links, identifiers, representations, schemas, paging, streaming messages and errors does the application rely on beyond the library?
   - How does it behave when they are missing, extra, slow or failing?
4. **Q4 — Standards alignment:** For each material dependency, what do CSAPI Parts 1 and 2 (and incorporated Common/Features, SWE Common, SensorML and, where used, the draft Part 3 material) require or allow? Classify the dependency as conforming, stricter, tolerant, CS-GO-specific, draft or experimental, outside CSAPI, or non-conforming.
5. **Q5 — Tests and quality evidence:**
   - What do the unit and component specs (Vitest), the single end-to-end spec (Playwright) and the 57 stories establish about server-facing behaviour?
   - Are their expected values independent, and what server do they assume?
6. **Q6 — Transfer to Glaux:** Which concrete expectations, workflows or checks should inform:
   - the Phase 1 review;
   - later gates, particularly dynamic data, streaming and security;
   - IDR-SRV-056;
   - the Guide and Roadmap?

   Give an explicit disposition for each.

### Detailed Questions

**Identity and workflows (Q1)**

- Record the repository commit, the exact library version it resolves, and any difference from IDR-SRV-065's pin.
- Inventory routes and views, and map them to user workflows: browsing and search, System detail, filtering by parent System, maps, datastream and observation views, commands, events, administration and sign-in.
- State any targeted Parts or versions, and any draft or experimental support, from the README, configuration and code.

**Server capabilities per workflow (Q2)**

- For each workflow, trace from UI action to library call, or to a direct request made without the library. Record resource families, nested routes, parameters, filters, paging, format selection and headers.
- Record the MQTT use: broker discovery, topic structure and message formats, and whether each comes from the library's `mqtt` module or from Aleph's own code. Relate it to the draft Part 3 material and Glaux's experimental boundary (Guide §4.8), and classify it.
- Record the OpenID Connect use: flow, token handling, the scopes or claims assumed, and how tokens reach the server. Relate this to Glaux's authentication design (Guide §4.10).

**Response dependencies and failure handling (Q3)**

- For each workflow, list the members, links and relation values read beyond the library's own requirements. Include GeoJSON versus SensorML JSON use, and time and geometry handling for maps and charts.
- Examine error display, retries, empty results, partial failures, reconnection and token expiry. Record any assumption the standard leaves optional.

**Standards alignment (Q4)**

- Map each material dependency to exact requirement identifiers, or classify it as outside CSAPI.
- Consult only the [upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md) entries the findings implicate. Classify issue and pull-request evidence as informative.
- Compare with Glaux Guide §13 interpretations. Note points where CS-GO, the library and Aleph agree against the standard or against Glaux's reading.

**Tests and quality evidence (Q5)**

- Inventory the test layers and their server assumptions (mocked, recorded or live, and which server). Explain representative tests: setup, input, expected result, assertion and the failure each would catch.
- Assess expected-value independence, and mark tests that could pass with a matching mistake in application and fixture.

**Transfer to Glaux (Q6)**

- For Phase 1, list Aleph's expectations for discovery, System creation and a single System representation. Say whether Glaux's implemented behaviour meets or conflicts with them.
- For later phases, list end-to-end workflows each relevant gate review should exercise, with pins and required environment. Classify streaming and sign-in workflows against Glaux's selected scope.
- Recommend whether Aleph, as a pinned build in an approved environment, should join IDR-SRV-056's matrix, and state its independence and environment limits.
- Compare each lesson with Guide sections and Roadmap task IDs before proposing work. "No change" is acceptable.

---

## 3. Primary Resources

- **Application repository:** [SomethingCreativeStudios/Alephex][App] at default branch `main`, commit [`7af6c076a4edec1959fc138b11d13e309baf5e67`][AppPin] (preliminary check, September 29, 2026). Start from:
  - `README.md`, `package.json` and the `pnpm` lockfile;
  - `src/`, `e2e/` and `.storybook/`;
  - the Vite, Vitest and Playwright configuration;
  - the `Dockerfile`, `compose.yaml` and `Caddyfile`;
  - `.env.example` (variable names only).

  Record the actual execution snapshot.
- **Library:** `cs-api-client` `0.1.3` as locked in Aleph's `pnpm-lock.yaml` (integrity `sha512-aN54cvRElSuQ/…`), which is IDR-SRV-065's primary baseline. Its closest committed source is `a6327989f5cec3c36422a121a2fe676e8a6bbe6d`, as IDR-SRV-065 establishes.
- **Independence comparison:** [CS-GO](https://github.com/SomethingCreativeStudios/connected-systems-go) at the IDR-SRV-062 pin and, where needed, its current head. Use it only as needed to judge shared interpretation.
- **History entry points:** commits, any pull requests, issues and tags, with totals recorded.
- **Controlling standards:**
  - [CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) and [Part 2](https://docs.ogc.org/is/23-002/23-002.html), with their abstract tests, schemas and examples;
  - [OGC API – Common Part 1](https://docs.ogc.org/is/19-072/19-072.html) and [Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) where incorporated;
  - [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html) and [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) where used;
  - the official draft Part 3 material where MQTT use relates to it;
  - the [upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), limited to implicated entries.
- **Protocol references where needed:** [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html) and the [MQTT 5.0](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html) specification. Consult them only to classify a behaviour the application relies on.

---

## 4. Supporting Resources

- The IDR-SRV-065 report (cs-client-ts), once accepted: the library baseline this study builds on.
- The IDR-SRV-063 and IDR-SRV-064 reports, once accepted: contrasting application evidence from the OSH ecosystem.
- [IDR-SRV-014B](../IDR%20Reports/idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md) and [IDR-SRV-062](../IDR%20Reports/idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md): the same author's server and practices.
- [IDR-SRV-014H draft Part 3](../IDR%20Reports/idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md), [035 streaming](../IDR%20Reports/idr-srv-035-streaming-and-event-publication-strategy-report.md), [039 security threat model](../IDR%20Reports/idr-srv-039-authentication-authorization-and-api-security-threat-model-report.md) and [055 security tests](../IDR%20Reports/idr-srv-055-security-authorization-and-command-control-test-strategy-report.md): Glaux's accepted streaming and security baselines.
- [IDR-SRV-056](../IDR%20Reports/idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md), [014E](../IDR%20Reports/idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md) and [014G](../IDR%20Reports/idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md): existing client findings to reconcile.
- Current planning: [Goal v1.10](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md), [Guide v1.21](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md) (§§4.1–4.10, 4.12, 6.2–6.4, 8.1–8.2, 13), [Roadmap v1.40](../../../../../Plans/glaux-server/glaux-server-roadmap.md) and the [Phase 1 review charter](../../../../../Plans/glaux-server/Implementation-Reviews/Phase-1/README.md).
- Implemented Glaux behaviour for comparison: server [`docs/system-create.md`](https://github.com/DGIWG-P507/glaux-server/blob/main/docs/system-create.md), [`docs/system-read.md`](https://github.com/DGIWG-P507/glaux-server/blob/main/docs/system-read.md), [`docs/discovery.md`](https://github.com/DGIWG-P507/glaux-server/blob/main/docs/discovery.md) and [`docs/authentication.md`](https://github.com/DGIWG-P507/glaux-server/blob/main/docs/authentication.md) at the recorded server commit.

---

## 5. Research Methodology

### Phase 1: Establish identity, workflows and coverage

**Objective:** Pin exactly what is studied, map the workflows, and bound the investigation.

**Tasks:**
1. Refresh and record the repository commit, the resolved library version and the retrieval date. Record any difference from IDR-SRV-065's pin.
2. Inventory the source, test, story and deployment trees. Map routes and views to user workflows, and mark presentational-only areas for inventory-level mention.
3. Identify targeted Parts, versions and non-CSAPI features (MQTT, OpenID Connect), and record the shallow-history limit.
4. Select representative workflows covering discovery, System read and create where present, filtered listing, dynamic data, streaming and sign-in.
5. Consult only implicated entries of the standards-history register.

**Expected Output:** A coverage inventory with pins, a workflow map and a justified workflow selection, within the developing report.

### Phase 2: Trace workflows end to end

**Objective:** Answer Q2 and Q3 from source.

**Tasks:**
1. For each selected workflow, trace from UI action through Aleph's data layer to the library call or direct request. Attribute each request to library or application.
2. Record the response members, links and formats read beyond the library's own requirements, and the handling of missing, extra, slow or failing responses.
3. Trace MQTT and OpenID Connect use, and record what each assumes of the server or deployment.
4. Build a workflow dependency table with source anchors at the pinned commit.

**Expected Output:** A traceable workflow dependency table and walkthroughs.

### Phase 3: Judge against the standard and examine tests

**Objective:** Answer Q4 and Q5, and draft Q6.

**Tasks:**
1. Map each material dependency to exact requirement identifiers, or classify it as outside CSAPI. Note shared author readings that differ from the standard or from Glaux.
2. Read representative Vitest, Playwright and story files, and assess what each detects, which server it assumes and whether its expectations are independent.
3. Check tool availability without installing anything. Build, run or test only in an already permitted, isolated environment with synthetic configuration, recorded exactly. Otherwise state source inspection as the limit.
4. Compare Aleph's Phase 1 expectations with Glaux's implemented System create/read and authentication behaviour at the recorded server commit.

**Expected Output:** A standards-classified dependency table, test analysis and a Phase 1 comparison with explicit limits.

### Phase 4: Synthesis

**Objective:** Produce one readable, decision-usable report in the existing template, and close the four-study sequence.

**Tasks:**
1. Answer Q1–Q6, keeping observed source, attributed claims, inference and recommendation distinguishable.
2. Put the most useful findings first: Phase 1 expectations, end-to-end workflows for later gates, streaming and sign-in classification, and independence limits. Recommend on IDR-SRV-056 inclusion.
3. Map each lesson to Guide sections and Roadmap task IDs, with a disposition.
4. Add a short cross-study note listing where the four studies agree or disagree on the same server behaviour, and which disagreements the Phase 1 review should examine. Do not re-open the other reports.
5. Validate coverage, references, attribution and success criteria. Prepare the acceptance handoff without editing downstream artifacts.

**Expected Output:** One research report for project-lead review. If more than one iteration is needed, keep it explicitly in progress and resume only on the next `proceed`.

---

## 6. Success Criteria

This topic research is complete when:

- [x] Q1–Q6 have evidence-backed answers, or explicit limitations and their consequences.
- [x] The application commit and the resolved library version are pinned.
- [x] Every request-level finding is attributed to library or application.
- [x] User workflows are mapped. Every material dependency is traceable to a source anchor and classified against exact standard identifiers or as outside CSAPI.
- [x] MQTT and OpenID Connect use is classified against Glaux's selected scope. No secret or real-deployment value is reproduced.
- [x] Independence limits (shared author, AI assistance, shallow history) are stated.
- [x] Test analysis explains what representative tests detect, which server they assume and whether their expectations are independent. Source inspection is distinguished from execution.
- [x] Phase 1 expectations are compared with implemented Glaux behaviour, later-gate workflows are listed, and the cross-study note is included.
- [x] Each material lesson has a Glaux disposition with Guide/Roadmap references. "No change" is acceptable.
- [x] Relevant official repository history is consulted and authority-classified where the standards-history register applies.
- [x] The report follows the report template and validates these criteria.

Completion does not require running Aleph, contacting the author, reading every component or certifying the application. Any such limit narrows the relevant conclusion; it does not disappear from the report.

---

## 7. Deliverable

**Deliverable Name:** Aleph (Alephex) Connected Systems UI Client Study - Research Report<br>
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-066-aleph-connected-systems-ui-client-study-report.md`

Use the existing report template and its full section set. Put the workflow map, dependency table, test examples and cross-study note in the corresponding sections or optional appendices.

For each material finding, capture this chain:
1. Application or library evidence at a pinned anchor.
2. Standard text and classification.
3. Independence note.
4. Glaux applicability.
5. Existing coverage or proposed bounded change.
6. Verification or review use.

Keep the plain-language summary readable on its own. The report is evidence, not a requirements document or a mandate to copy application behaviour.

---

## 8. Dependencies

### Must Complete Before Starting

**Internal project prerequisites (completion gates):**

- This plan and its registration in the [overall plan](overall-idr-research-plan.md) are published.
- The IDR-SRV-065 report is complete and accepted, because this study builds on its library findings. The project lead listed Aleph before cs-client-ts; this order is proposed for the lead to confirm.
- The project lead's `proceed` authorises execution.

Internal prerequisites are not waived by labelling them unavailable or deferred. Reorder or change them only through an explicit, recorded update to the controlling overall plan or this plan's approved dependency record.

**External evidence prerequisites:**

- Public access to the repository, the library package and the CS-GO comparison material. For inaccessible sources, record identity, attempted access, affected questions and the resulting limits. Do not infer contents.
- A permitted isolated environment, with synthetic identity-provider and broker configuration, is needed only for any execution actually performed.

### Blocks (What This Topic Unlocks)

- Completion of the four-study sequence, after which the Phase 1 implementation review's remaining steps (2–5) may resume on the project lead's `proceed`. Project-lead decision, September 29, 2026.
- Any later, separately authorised discussion of IDR-SRV-056, Guide or Roadmap changes. No implementation, issue creation or planning edit happens in this study.

---

## 9. Research Status Checklist

- [x] Phase 1 complete
- [x] Phase 2 complete
- [x] Phase 3 complete
- [x] Phase 4 synthesis complete
- [x] Deliverable draft complete
- [ ] Deliverable reviewed by project lead (separate pre-publication assistant review recorded in report PR)
- [ ] Deliverable accepted

**Actual Research Time:** One source-based AI research/report iteration, October 3, 2026; see the [report](../IDR%20Reports/idr-srv-066-aleph-connected-systems-ui-client-study-report.md) for method, pins, coverage and unexecuted checks.<br>
**Completion Date:** Research/report completed October 3, 2026; acceptance pending the project lead's report-PR merge. No downstream artifact or implementation change authorized.

---

## 10. Notes and Open Questions

- The project lead reports that Aleph was written by a senior developer with AI assistance. That motivates the study; it is not evidence about any particular behaviour.
- Aleph is new (created August 2026) and at version `0.0.0`. Its pin will age quickly, and later gate reviews should refresh it.
- Streaming and sign-in findings may concern capabilities Glaux has not yet built or has scoped as experimental. The report classifies them against current scope; it does not expand it.
- Stop when the questions and representative workflows are covered, with usable findings and explicit unknowns.

---

## References

- [Research Planning Approach](../../../../../Governance/research-planning-approach.md), [Initial Planning Guidance](../../../../../Governance/initial-planning-guidance.md), [Research Plan Template](../../../../../Governance/research-plan-template.md) and [Research Report Template](../../../../../Governance/research-report-template.md).
- [Controlling overall IDR plan](overall-idr-research-plan.md). This topic is registered as a supplement without reopening accepted topics.
- [Pinned OS4CSAPI research-plan exemplar corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans), used for structure only.
- The specific primary sources and accepted research inputs are listed in Sections 3–4.

[App]: https://github.com/SomethingCreativeStudios/Alephex
[AppPin]: https://github.com/SomethingCreativeStudios/Alephex/tree/7af6c076a4edec1959fc138b11d13e309baf5e67

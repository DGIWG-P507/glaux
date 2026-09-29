# Section 064: OSCAR Viewer Client Study - Research Report

**Topic ID:** IDR-SRV-064<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-064 plan](../IDR%20Plans/idr-srv-064-oscar-viewer-client-study.md)<br>
**Overall Research Plan:** [Controlling overall IDR plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Q1–Q6. Requests, streams, commands, response dependencies, lineage and test evidence are established from source and from two recorded test runs. Nothing was executed in this study.<br>
**Methodology Used:**
- Source inspection of OSCAR at `4b49c7c` and of the `earocorn/osh-js` fork revision its lockfile resolves (`73dacad`).
- Lineage comparison with IDR-SRV-063's toolkit baselines.
- Dependency mapping to the published CSAPI text, the official OpenAPI parameter files and the pinned schemas.
- Public history, package and pull-request inventories.
- Point-by-point checking of the OS4CSAPI OSCAR analysis.

**Research Time:** About 25 minutes of research and drafting, 21:09–21:35 UTC September 29, 2026, in one AI-assisted iteration, before separate review. This is not a human-hours estimate. Nothing was executed or installed.<br>
**Primary Sources:**
- [OSCAR Viewer at `4b49c7c`][OscarPin]
- [`earocorn/osh-js` `add-consys` at `73dacad`][ForkPin]
- [CSAPI Part 1][Part1], [CSAPI Part 2][Part2], [OGC API – Features Part 1][Features]
- CSAPI OpenAPI parameters and JSON schemas at [`8e03b23`][Schemas]

**Supporting Resources:**
- [IDR-SRV-063 report][R063] (accepted)
- [IDR-SRV-014A][R014A], [014H][R014H], [056][R056]
- [Guide v1.21][Guide], [Roadmap v1.40][Roadmap]
- Glaux Server `main` at [`27955c1`][ServerPin]

**Document Purpose:** Independent client evidence for the Phase 1 implementation review, for later review gates (especially dynamic data, commands and live delivery) and for IDR-SRV-056. It is not a requirements document, and it does not authorise copying client behaviour.<br>
**Author(s):** Glaux research workflow, AI-assisted<br>
**Accepted By:** TBD until project-lead acceptance<br>
**Acceptance Date:** TBD<br>
**Date:** September 29, 2026<br>
**Last Updated:** September 29, 2026

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base
4. Findings by Research Question
5. Decision Analysis
6. Key Recommendations
7. Implementation Implications and Estimates
8. Risks, Constraints, and Open Questions
9. Validation Against Plan Success Criteria
10. Next Steps and Handoff
11. References
12. Appendices

---

## 1. Executive Summary

**OSCAR is a current, actively maintained application. Most of its routes follow published CSAPI, but many of its query details are still OpenSensorHub-specific.** It uses a fork of the OSH toolkit that adds a new "Connected Systems" module. That module uses the published route names:
- `/controlstreams`, `/subsystems` and `/samplingFeatures`;
- commands posted to `/controlstreams/{id}/commands` with a `parameters` body;
- command status reports with inline `results`.

Those parts match the standard and Glaux's plans. [Findings §4.1–4.4](#41-q1--identity-version-and-lineage).

**It still depends on OSH-only extras that Glaux has deliberately not planned.** Glaux would reject or not provide these:
- **Undeclared query parameters:** `searchMembers`, `format` and `offset`, plus `validTime=latest` on System and datastream lists. The OGC Features standard requires a server to reject parameters its API description does not declare; Glaux's Guide rejects them too.
- **OSH-only routes:** an `/observations/count` endpoint, an `order` parameter, and control-stream-level status streams.
- **Sign-in:** a session cookie from the OSH server, or Basic credentials. Glaux accepts only Bearer tokens.
- **Live data:** MQTT topics that are the HTTP path plus a query string. Glaux's planned topics use a different, fixed structure.

**Where OSCAR meets Glaux's Phase 1 work, the System fields agree.** OSCAR reads each listed System's `id`, `properties.uid` and `properties.name`, the same paths as Glaux's GeoJSON System. Like the OSH Viewer, it needs a System list that Glaux has not built yet.

**Four findings are worth carrying into later reviews:**
1. **Command success codes.** Glaux will answer a command POST with `201 Created` (Guide §6.4). OSCAR's report screen only proceeds on exactly `200`, while its other screens accept any success. An interoperability test for Roadmap 5.2/5.3 should include a client like this.
2. **Silent truncation.** OSCAR stops paging when a page is shorter than it asked for. If a server caps page size below OSCAR's request (for example a configured maximum under 500), OSCAR silently shows only the first page. Glaux publishes and clamps limits (Guide §6.3), so its documentation and tests should make that visible.
3. **The official OpenAPI files differ slightly from the normative text.** They define an `f` parameter that no operation uses, and a `validTime` parameter only for System history. This supports Glaux's existing choices and IDR-SRV-063's classifications, and changes neither.
4. **The CORS item from IDR-SRV-063 has a practical alternative.** OSCAR avoids cross-origin access by being served from the same address as its OSH server. That is evidence for the pending CORS-ownership discussion, not a decision.

**Recommendation:** do not add OSCAR to IDR-SRV-056 as a compatibility target yet. Its Connected Systems module is the most useful peer client evidence so far, so record it as a candidate for a pinned subset once Glaux has lists, datastreams and commands. The subset would cover `/controlstreams` commands and status reports, observation reads, and schema reads. No Guide or Roadmap change is proposed.

## 2. Scope and Plan Alignment

This is **IDR-SRV-064**, the second of the four client studies, run after IDR-SRV-063 was accepted on September 29, 2026.

| Plan question | Coverage status | Evidence location |
|---|---|---|
| Q1 — Identity, version, lineage | Complete; one resolved fork revision; per-module API version classified | §4.1, §12.1 |
| Q2 — Requests, streams and commands | Complete for every CSAPI-facing path in OSCAR and the fork modules it uses | §4.2, §12.2 |
| Q3 — Response dependencies | Complete from source; runtime behaviour inferred, not executed | §4.3 |
| Q4 — Standards alignment | Complete for every material dependency | §4.4 |
| Q5 — Tests and quality evidence | Complete, including the two recorded runs | §4.5 |
| Q6 — Transfer to Glaux | Complete, with dispositions | §4.6, §§5–7 |

**Out of scope:**
- running OSCAR, its tests or the fork's tests;
- contacting maintainers, or posting upstream;
- OSCAR's domain logic (lane, alarm and adjudication rules), its maps, charts and video, and its report and file features, except where they send CSAPI requests;
- copying code.

Nothing was installed. OSCAR's operational configuration is not reproduced. The test server's network address, lane names and alarm rules are mentioned only where they explain an API dependency.

**Earlier coverage reconciled:**
- [IDR-SRV-063][R063] is the lineage baseline. §4.1 states what the fork adds.
- [IDR-SRV-014A][R014A] already records the OSH server behaviours OSCAR relies on: `searchMembers` as an alias, `offset`/`limit` paging, and nested routes.
- [IDR-SRV-014H][R014H] and Guide §4.8 set Glaux's draft Part 3 boundary, which OSCAR's MQTT use is compared against.
- IDR-SRV-056 does not yet list OSCAR.
- 014E and 014G concern the OS4CSAPI client and are not repeated.

Nothing here contradicts an accepted report.

## 3. Evidence Base

### 3.1 Primary Sources Reviewed

All access dates are **2026-09-29 (UTC)**.

| Source | Version / commit | Authority class | Availability / limitations |
|---|---|---|---|
| [OSCAR Viewer][Oscar] | `main` at [`4b49c7c`][OscarPin] (September 23, 2026); 866 commits; 171 tracked files | Observed client application | Complete tree; CSAPI-facing code read in full, domain code at inventory level |
| [`earocorn/osh-js`][Fork] | Branch `add-consys` at [`73dacad`][ForkPin] (August 22, 2026), which OSCAR's `package-lock.json` resolves exactly; `package.json` version `3.1.5` | Observed toolkit fork | The `consysapi` modules, connectors, parsers and tests read in full; the rest is inventory only |
| OSCAR `patches/osh-js+3.1.5.patch` | Applied at install by `patch-package` | Observed local change to the fork | Changes only MQTT connector/provider sharing and two data-source files; no request paths |
| [CSAPI Part 1][Part1], [Part 2][Part2] | Published HTML | Normative | Requirement identifiers in §4.4 |
| CSAPI OpenAPI parameter files and JSON schemas | [`8e03b23`][Schemas] | Official artifacts (the OpenAPI is informative; the schemas are normative) | Parameter names and uses listed in §4.4 |
| [OGC API – Features Part 1][Features] | Published HTML | Normative where incorporated | Requirement 8 (unknown parameters); Recommendations 17–19 (`next` links) |
| [OS4CSAPI OSCAR analysis][Analysis] | `main`, last changed 2026-02-24 (document dated January 31, 2026) | Pointer only (see plan) | Each point checked against OSCAR source in §12.3 |

**History and totals** (commits from local clones; pull-request, issue, release and contributor counts from the GitHub API, unauthenticated):
- **OSCAR:**
  - 866 commits, 217 pull requests, 86 issues, one release and one tag.
  - GitHub contributor counts: `kalynstricklin` 485, `tipatterson-dev` 133, `earocorn` 87, `n-garay` 65, `salsajeries` 47, and a few smaller contributors.
- **Fork `add-consys`:**
  - 65 commits since its merge base with upstream `mcs_baseline`, `549c630`. That is IDR-SRV-063 baseline A's parent.
  - Authors of those commits: Alex Almanza, who also commits as `earocorn` (same email; 32 in total), `kalynstricklin` (18), and upstream maintainer Mathieu Dhainaut (15, from merged upstream work).
  - 5 fork pull requests (all closed) and no issues.
- **Upstream `opensensorhub/osh-js`:**
  - `earocorn`'s [pull request #781][PR781] "Add Connected Systems" (head `earocorn:add-consys`) was **closed without merging** on August 5, 2025, with no comments.
  - Two smaller pull requests from the same author were merged: #780 (feature-of-interest route names) and #820 (packaging).
- **npm:**
  - `osh-js` versions 3.1.2–3.1.5 were published by `earocorn`, who shares npm maintenance with `mdhsl`. Their recorded source commits are fork commits: `591de78`, `d41767a`, `d1fb02e` and `b2e8954` (3.1.5).
  - The package metadata still names the upstream repository, which contains none of them.
  - This resolves IDR-SRV-063's open note about where `osh-js` 3.x came from. The 3.0.0 source commit, `ccdea29`, is in neither clone.

### 3.2 Supporting Sources Reviewed

| Source | Baseline | Role |
|---|---|---|
| [IDR-SRV-063][R063] | Accepted September 29, 2026 | Lineage baseline and prior classifications |
| Glaux Server docs [`system-read.md`][SysRead], [`authentication.md`][Auth], [`discovery.md`][Disc] | `27955c1` | Implemented Phase 1 behaviour |
| [Guide v1.21][Guide] §§4.4, 4.8, 4.9, 4.10, 6.3, 6.4, 13; [Roadmap v1.40][Roadmap] | Planning `main` at `28a5569` | Controlling scope and dispositions |
| [Upstream-history register][Register] | Current | Checked for command-status paths, `/controls`, `f` and `validTime`; the upstream `/command/{cmdId}/status` spelling is already handled through Guide §13; no new entry is implicated |

### 3.3 Evidence Quality Notes

- **Authority order:**
  1. Published CSAPI text and pinned schemas.
  2. Incorporated Features text.
  3. The official OpenAPI files, which are informative where they differ from the text.
  4. Glaux's Guide.
  5. Client code.
- **No execution.** Runtime claims are inferred from pinned code paths and are labelled as such. The two Cypress logs are evidence of those two April 22, 2026 runs only.
- **The pin is exact.** The lockfile resolves `73dacad`, the branch head, so one fork baseline suffices. OSCAR's install-time patch touches no request path.
- **Authorship.** The project lead reports that OSCAR is among the most human-written CSAPI clients. The public record shows several long-standing contributors, and one fork author who is also an OSCAR contributor. No inference about tool or AI use is made. Neither repository contains assistant-guidance files; their absence is not evidence either way.

## 4. Findings by Research Question

### 4.1 Q1 — Identity, version and lineage

**Source-backed finding: the fork adds a parallel `consysapi` module that mostly uses published CSAPI names. It keeps the old `sweapi` module beside it.**
- **Lineage:**
  - The fork diverged from upstream `mcs_baseline` at `549c630`, one commit before IDR-SRV-063's baseline A.
  - It then merged upstream `dev` (for example `f434cc9` and `9e91d61`, which moved feature-of-interest routes to `samplingFeatures`).
  - It added [`b2e2d05`][ForkAdd] "Add Connected Systems" (February 26, 2025) and later changes.
- **The new module:** 54 files under `source/core/consysapi`, `datasource/consysapi` and `parsers/consysapi`. The 54 `sweapi` files are still present.
- **The route table** ([`consysapi/routes.conf.js`][ForkRoutes]) uses:
  - `/systems/{id}/subsystems`, `/systems/{id}/samplingFeatures` and `/systems/{id}/controlstreams`;
  - `/controlstreams/{id}`, `/controlstreams/{id}/commands` and `/controlstreams/{id}/schema`;
  - `/commands/{id}`, `/observations` and `/samplingFeatures`.

  It also keeps the draft-era `/systems/{id}/members` and `/systems/{id}/details`. It adds two OSH-style routes, `/controlstreams/{id}/status` and `/systems/{sysid}/controlstreams/{csid}/commands/{cmdid}/status`.
- **Unchanged from IDR-SRV-063's toolkit:** the paging (`Collection.js`), the filter-to-query code (`Filter.js`) and the credential handling (`ConnectedSystemsApi.js`) are the same design as the `sweapi` versions. [Collection][ForkColl], [Filter][ForkFilter], [base class][ForkBase]
- **Added since 2025:**
  - a `pageOffset` parameter (fork PR #2, November 2025);
  - Jest tests (§4.5);
  - removal of System nesting from control-stream routes (fork PRs #4 and #5, March 2026);
  - `credentials: 'include'` in the HTTP connector (`73dacad`).

**OSCAR itself uses the new module almost exclusively.**
- Its toolkit imports are nearly all `consysapi`. Three `sweapi` imports remain:
  - `DataStream` in `EventTable.tsx` and `StatusTable.tsx`, used only as a TypeScript type;
  - `ObservationFilter` in `EventTable.tsx`, which *is* instantiated and passed to the `consysapi` Observations client for the event list (O5), so that query's default `format=application/om+json` comes from the old module.
- **Per-module API version:**
  - route paths: mostly **published CSAPI**, with the draft-era `members` and `details` routes unused by OSCAR;
  - query parameters: largely **OSH-specific**;
  - command bodies and status: **published Part 2** (§4.4);
  - MQTT topics: **OSH-specific** (§4.2).

| Selected history case | What changed | Relevance |
|---|---|---|
| Fork [`b2e2d05`][ForkAdd], 2025-02-26 | Connected Systems modules added | Start of published-route support |
| Upstream [PR #781][PR781], closed 2025-08-05 unmerged | The same work offered upstream | Keeps the published-route client in a personal fork (npm-published) |
| Fork `b45c854`, 2025-10-30 | "Fix command and command status parsing. Add unit testing" | Defect fix with tests on the command path |
| Fork PRs #4/#5, 2026-03 | "no need to nest controlstreams under systems" | Moves control-stream access to the canonical `/controlstreams/{id}` form |
| OSCAR [`c2101fe`][NoStore], 2026-08-22 | "Remove login credentials from browser storage and use session authentication by default" | Credentials become runtime-only; the stored node record omits them ([`OSHSlice.tsx` L80–98][Slice]) |

### 4.2 Q2 — Requests, streams and commands

**Source-backed finding: OSCAR hard-codes its API root and paths. Its only discovery step is a reachability check on the root.**
- The root is `{host}:{port}{oshPathRoot}{csAPIEndpoint}`, defaulting to `/sensorhub/api` on the same host and port that served the page. [`DataSourceContext.tsx` L118–129][DSC], [`Node.ts` L138–178, L214–218][Node]
- `checkForEndpoint` sends `GET {root}` and checks only that the response succeeds. It never reads the landing page, conformance, API description or collections. [`Node.ts` L241–260][Node]

| # | Request | Built by | Notes |
|---|---|---|---|
| O1 | `GET {root}` | OSCAR | Reachability only; sends `Content-Type: application/sml+json` on a GET |
| O2 | `GET /systems?format=application/json&searchMembers=true&validTime=latest&offset=n&limit=500` | Fork, from OSCAR's filter | Pages until a short page ([`Node.ts` L347–363][Node]) |
| O3 | `GET /datastreams?format=application/json&obsFormat=application/om+json&system={ids}&validTime=latest&offset=n&limit=1000` | Fork | `obsFormat` comes from the filter default ([`DataStreamFilter.js` L39–49][ForkDSF]); [`Node.ts` L410–445][Node] |
| O4 | `GET /controlstreams?format=application/json&system={ids}&validTime=latest&offset=n&limit=1000` | Fork | [`Node.ts` L457–487][Node] |
| O5 | `GET /observations?dataStream={ids}&resultTime=../{t}&filter={expr}&order=desc&format=application/om+json&offset&limit` | Fork, from OSCAR | Event list ([`EventTable.tsx` L212–219][EvtTab]) |
| O6 | `GET /observations/count?resultTime=…&format=…&dataStream=…&filter=…` | OSCAR | Event counts; reads `count` ([`EventTable.tsx` L226–245][EvtTab], [`StatusTable.tsx` L84–110][StatTab]) |
| O7 | `POST /controlstreams/{id}/commands` with `Content-Type: application/json` and a body `{"parameters": {…}}` | OSCAR | 8 call sites in 7 files ([`OSCARCommands.ts` L5–18][Cmds]) |
| O8 | MQTT subscribe `{prefix}/datastreams/{id}/observations?format=…` and `{prefix}/controlstreams/{id}/status?format=…` | Fork, from OSCAR | Prefix defaults to `/api` ([`LaneCollection.ts` L222–243][Lane], [`MqttProvider.js` L78–110][MqttProv]) |
| O9 | WebSocket `…/systems/{sysid}/controlstreams/{csid}/commands/{cmdid}/status` | Fork, from OSCAR's report screen | Asynchronous command follow-up ([`Report.tsx` L91–120][Report], [`Command.js` L44–77][ForkCmd]) |
| O10 | `GET /datastreams/{id}/schema?obsFormat=…` | Fork parsers | Before decoding observations, as in IDR-SRV-063 |

**Also in the code but not used:**
- `Systems.ts` defines `GET /systems/{id}/datastreams` and `GET /systems/{id}/subsystems`, but nothing imports or calls them. [`Systems.ts` L35–69][Sys]
- The `/buckets` file-server paths are an OSH service outside CSAPI.

**Authentication** (the default is session mode):
- **Session mode:** the page is served by the OSH server, which provides `/session`, `/login` and `/logout`. Requests rely on the browser's cookie through `credentials: 'include'`. [`Navbar.tsx` L141–160][Nav]
- **Basic mode:** the connectorOpts add `Authorization: Basic …` for the fork's requests, and OSCAR's own `getBasicAuthHeader` adds it for its direct `fetch` calls. MQTT gets the same user name and password. [`Node.ts` L158–164, L226–231][Node]
- No Bearer or OIDC path exists.

**Formats:**
- Collections are requested with the `format` query parameter, never `Accept`.
- Observations use `application/swe+json`, or `application/swe+binary` for video.
- Command and status streams use `application/json`. [`LaneCollection.ts` L194–205, L241][Lane]

### 4.3 Q3 — Response dependencies and tolerance

| Response | Members read | If missing or unexpected (inference from code) |
|---|---|---|
| O2 Systems | `items[]`; each item's `id`, `properties.uid`, `properties.name` (the fork wraps each item, so OSCAR reads `system.properties.properties.uid`) | No `items`: the fork turns the missing array into a single `undefined` entry, and OSCAR's lane sorting then fails on it (inference), so no lanes are built for that node. The UID prefix decides lane grouping |
| O3/O4 Datastreams, control streams | `items[]`; each item's `id`, `system@id` (matched to a System's `id`) and `system@link.uid` | **Missing `system@id` drops the stream from its lane silently.** A published-shape server sends `system@link`, not `system@id` |
| O5/O8 Observations | OM JSON `items[]` (O5); SWE records (O8) decoded with the schema from O10 | Parsers null-check (`58aef16`); unknown formats throw |
| O6 Count | `count` | Non-OK or error: logged and treated as 0 |
| O7 Command response | `response.ok`; for video, `results[0].data.streamPath`; the report screen needs **`status == 200`**, then `statusCode`, `command@id` and `results[0].data.reportPath` | Report screen: any other success code, such as 201, falls to its non-200 branch and the report does not start. Other screens accept any 2xx ([`Report.tsx` L97][Report], [`VideoMedia.tsx` L88–100][Video], [`AdjudicationDetail.tsx` L475–494][Adj]) |
| O8/O9 Status | `statusCode` (`PENDING`, `ACCEPTED`, `FAILED`), `results[0].data` | Other codes are shown as-is |

**Tolerance and silent loss:**
- **Paging:**
  - The fork computes `offset` values and stops on the first short page. It never reads `next` links, `numberMatched` or `numberReturned`. [`Collection.js` L45–100][ForkColl]
  - If a server returns fewer items than requested for any reason other than the end of the data, for example a lower maximum `limit`, the rest is silently missing.
- **HTTP status:**
  - The fork's `Collection` and `fetchAsJson` throw on non-OK responses, and OSCAR's direct calls check `ok`, which is better than the OSH Viewer.
  - The fork's `postAsJson` neither awaits nor returns its request, so its errors are lost. [`ConnectedSystemsApi.js` L141–161][ForkBase] OSCAR avoids that helper and posts commands itself.
- **Latent defect:** `createRealTimeConSysApi` chooses its path with `typeof stream == DataStream`, which is never true in JavaScript. Its only caller passes a control stream, so the output is correct today. [`LaneCollection.ts` L237][Lane]

### 4.4 Q4 — Standards alignment

Classification key: **C** conforming; **D** draft-era (the Sensor Web API draft, per IDR-SRV-063); **O** OSH-specific; **X** experimental/draft Part 3; **N** conflicts with the published standard.

| Dependency | Published source | Class | Glaux position |
|---|---|---|---|
| Hard-coded root `/sensorhub/api` | Deployment-chosen `{api_root}` | O | Deployable behind a path prefix; no change |
| `GET /systems` list | Part 1 Req 6 `/req/system/resources-endpoint` | C (path) | 405 today; Roadmap 2.3.1, 2.3.9 |
| `items[]` of GeoJSON-shaped Systems for `format=application/json` | Pinned schemas: GeoJSON collection uses `features`; SensorML collection uses `items` | O (same hybrid as IDR-SRV-063) | Follows the schemas; Roadmap 2.3.9 |
| `searchMembers=true` | Part 1 uses `recursive` (Req 10–12 `/req/subsystem/recursive-*`) | O (alias noted in 014A) | Guide §6.3 lists `recursive`; undeclared parameters are rejected (Features Req 8, Guide §4.4) |
| `validTime=latest` on `/systems`, `/datastreams`, `/controlstreams` | The official OpenAPI defines `validTime` only on System history; list filtering uses `datetime` (Part 1 Req 3). `latest` is not an RFC 3339 value | O | Rejected as undeclared or malformed; Guide §13 |
| `format=…` query parameter | Not defined. The official Part 2 OpenAPI has an `f` parameter file that no operation references | O | Guide §6.2 selects `Accept`/`obsFormat`, no `f` |
| `offset`/`limit`, stop on short page | Features Rec 17–19 (`next` links); no public `offset` | O | Guide §6.3: opaque `next`, published and clamped limits; Roadmap 2.5.10, 3.3.6 |
| `system={ids}` on datastreams and control streams | Part 1/2 system filters (`/req/advanced-filtering/*`) | C | Guide §6.3; Roadmap 3.3.x |
| `system@id` on datastreams and control streams | Pinned `dataStream.json`/`controlStream.json` require `system@link` | O | Follows the schema; OSCAR also reads `system@link.uid` |
| `dataStream={ids}` on `/observations` | Official Part 2 OpenAPI `dataStreamIdList.yaml` (`name: dataStream`) | C | Declared where Glaux implements observation filters (Roadmap 3.3.1) |
| `resultTime=../{t}` | Part 2 Req 50 `/req/advanced-filtering/obs-by-resulttime` | C | Roadmap 3.3.2 |
| `filter={expr}` on observations | Not Part 2; CQL2 through Features Part 3 | O (OSH), X for Glaux | Guide §4.4.1 selects CQL2 with named queryables on a single datastream. OSCAR's multi-datastream expression is not guaranteed |
| `order=desc` | Not in the Part 2 parameters | O | Undeclared, so rejected |
| `/observations/count` | Not in Parts 1–2 | O | Counts come as optional collection counts (Roadmap 3.3.6), not a separate route |
| `obsFormat` on schema; `swe+json`/`swe+binary` | Part 2 Req 11; SWE encodings | C | Roadmap 3.1.6, 4.4.4 |
| `application/om+json` | Not in Part 2 | D/O | Not supported |
| `POST /controlstreams/{id}/commands` with `{"parameters": …}` | Part 2 Req 71 `/req/create-replace-delete/command`; pinned `command.json` | C | Roadmap 5.2.3, 5.3.x |
| Requires `200` on command POST (report screen) | Part 2 leaves the creation response to the create class; Glaux selects `201` + `Location` + status body (Guide §6.4) | Client assumption | Later-gate check (§4.6) |
| Status `statusCode`, `command@id`, `results[].data` | Pinned `commandStatus.json` and `commandResult.json` (inline result); Part 2 note on synchronous status in the HTTP response | C | Guide §4.9; Roadmap 5.2.5, 5.2.6, 5.3.2 |
| Status at `/systems/{sysid}/controlstreams/{csid}/commands/{cmdid}/status` and `/controlstreams/{id}/status` | Part 2 Req 32 gives `{api_root}/command/{cmdId}/status` (the upstream singular spelling); there is no control-stream-level status route | O | Guide §13 uses `/commands/{id}/status` consistently; no alias |
| MQTT topics `{prefix}/datastreams/{id}/observations?format=…` | Not in Parts 1–2; draft Part 3 bindings differ | O | Guide §4.8 topics such as `data/datastreams/{id}/observations/json` under a configured prefix; Roadmap 6.3.2 |
| WebSocket status follow-up | Not in Parts 1–2 | O | No WebSocket planned (Guide §4.8) |
| Session cookie or Basic credentials | Deployment matter | O | Bearer only ([authentication][Auth]; Guide §4.10) |

**Interpretation:** OSCAR's *resource model and command semantics* are published CSAPI. Its *query dialect, paging, counts, live topics and sign-in* are OSH-specific. Every compared Glaux choice agrees with the published text. The official OpenAPI evidence confirms, rather than changes, IDR-SRV-063's classifications of `f` and `validTime`.

### 4.5 Q5 — Tests and quality evidence

- **OSCAR component tests** (5 files, 371 lines):
  - They replace network methods with stubs, so they test OSCAR's own logic, not API behaviour.
  - Example: [`LaneDiscovery.cy.ts`][CyLane] checks that a healthy node's lanes appear before an unavailable node settles, and that data and control metadata load concurrently.
  - Example: [`AdjudicationCommand.cy.ts`][CyAdj] checks that the command JSON carries `parameters.vehicleId`, a real (small) assertion on the command body.
- **OSCAR end-to-end tests:**
  - 10 files (`all-specs.cy.tsx` imports six others), plus `GeneralTestingActions.tsx`, whose tests the default spec pattern does not run.
  - They drive the user interface against a live server at a fixed private address in `cypress.config.ts`.
  - Their assertions concern screen elements and lane names, so they catch broken screens, not API contract changes.
- **The two recorded runs** (April 22, 2026; Cypress 14.5.4, Electron 130, 6 specs each):

  | Run | Tests | Passing | Failing | Pending | Skipped |
  |---|---|---|---|---|---|
  | 12:49 | 58 | 1 | 8 | 4 | 45 |
  | 13:45 | 58 | 35 | 0 | 23 | 0 |

  - The first run's failures are UI timeouts, for example an expected map container or lane row never appearing.
  - The commit [`5c49073`][CyFix] that added both logs also changed the specs and `cypress.config.ts`.
  - So the second run shows that the edited suite passed against that server on that day. It does not show API conformance or current behaviour.
- **Fork Jest tests** (`tests/consysapi`):
  - `controlstreams`, `datastreams` and `observation` need a live OSH server at `localhost:8282/sensorhub/api`. Most assertions check only that a collection or page exists or is non-empty.
  - One test checks that paging with an offset returns later result times, a meaningful behaviour check.
  - `commands.test.js` and `systems.test.js` are empty.
  - No CI workflow runs them.

**Interpretation:** OSCAR has more test evidence than the OSH Viewer, but none of it independently asserts API contract details. Glaux cannot reuse these tests as oracles. Their value is in showing which workflows matter to real users: lane discovery, commands with results, and event counts.

### 4.6 Q6 — Transfer to Glaux

**Phase 1 comparison (server `27955c1`):**

| OSCAR expectation | Glaux today | Result |
|---|---|---|
| `GET {root}` succeeds | The landing page exists when discovery is enabled. OSCAR's session cookie or Basic credential is not accepted; with the root protected, OSCAR reports the node unreachable | **Conflict, by design** (authentication) |
| List Systems | `GET /systems` gives 405 | **Conflict for now**; planned (2.3.1, 2.3.9, 2.5.10) |
| Each listed System item has `id`, `properties.uid`, `properties.name` | The same paths in the GeoJSON System | **Consistent** (as in IDR-SRV-063) |
| UID-based grouping of Systems | Glaux keeps UIDs exactly as submitted | **Consistent**; UID spelling is client data |
| Same-origin hosting avoids CORS | Guide §4.10 requires configured browser origins; no Roadmap task names CORS (IDR-SRV-063 R2) | Evidence for that pending discussion only |

**Checks for later gates**, stated as Glaux expectations:

| Gate area (Roadmap) | Check to include |
|---|---|
| 2.3.x / 2.5.10 Systems lists | `recursive` works; `searchMembers`, `format`, `offset` and `validTime=latest` give 400. The published maximum `limit` and the `next` link are visible, so a short page is not mistaken for the end |
| 3.1.x Datastreams | `system@link` with `uid`/`href`; the `system` filter; no undocumented `system@id` |
| 3.3.1, 3.3.2, 3.3.6 Observations | The `dataStream` list filter and `resultTime` open intervals behave as published. Counts are the optional authorised collection counts, and there is no `/observations/count` route |
| 4.4.1 (CQL2) | Document whether multi-datastream observation filters are supported; OSCAR's event queries need them |
| 5.2.3, 5.3.x Commands | `201` + `Location` + status body. Include a client check like OSCAR's report screen, which expects `200`. The inline `results[].data` shape follows `commandResult.json` |
| 5.2.5 Status | `/commands/{id}/status`; no nested or control-stream-level alias |
| 6.3.2 MQTT topics | The documented topic structure is exact; OSH-style `…/observations?format=…` topics are not served |

**IDR-SRV-056 recommendation:**
- Do not add OSCAR as a compatibility target now. Its query dialect and sign-in would fail against Glaux by design.
- Record OSCAR at `4b49c7c`, and the fork at `73dacad`, as a **candidate for a pinned published-route subset**: command POST and status and result reading, the `dataStream` and `resultTime` observation filters, and schema reads. Qualify it once those Glaux tasks exist and a permitted isolated environment can run it.
- Its runtime needs the OSH-hosted session login or Basic authentication. That would require an adapter or test build, and 056 would need to state it as a limitation.

## 5. Decision Analysis

| Option | Benefits | Costs / risks | Standards impact | Recommendation |
|---|---|---|---|---|
| A. Keep Glaux as planned; use this report as review evidence | Every compared choice already matches the published text | OSCAR will not work unchanged | Preserves conformance | **Keep** |
| B. Add OSH query aliases (`searchMembers`, `validTime=latest`, `format`, `offset`, `order`, `/observations/count`) and MQTT topic forms | OSCAR might work | Undeclared vendor surface; contradicts Guide §§4.4, 4.8, 6.2, 6.3 and 13 | Conflicts with Features Req 8 unless every alias is declared | **Reject** |
| C. Add OSCAR to IDR-SRV-056 now | A human-developed, current client | Would fail on dialect and authentication, not on semantics | Misleading signal | **Reject for now** |
| D. Record a pinned published-route subset as a future 056 candidate | Real command, status and observation workflows | Needs an authentication adapter and an isolated environment | None | **Conditional**: a later IDR-SRV-056 discussion |

## 6. Key Recommendations

1. **Use this report as Phase 1 review evidence.** It confirms again that the Phase 1 System fields are consistent with an independent client, and that the list, authentication and dialect gaps are deliberate.
   - Priority: High.
   - Preconditions: project-lead acceptance.
2. **Carry the later-gate checks in §4.6 into the relevant gate reviews**, especially the `201` command response and visible page limits. These are review prompts, not new requirements.
   - Priority: Medium.
3. **Record OSCAR as a future IDR-SRV-056 candidate for a pinned published-route subset**, with its authentication limit stated.
   - Priority: Low.
   - Preconditions: a separately authorised 056 discussion.
4. **Add OSCAR's same-origin hosting to the pending CORS-ownership discussion** from IDR-SRV-063, as evidence only.
   - Priority: Low.
5. **No Guide or Roadmap change is proposed.**

## 7. Implementation Implications and Estimates

### 7.1 Implications

- **Architecture and implementation:** none. The findings support the existing negotiation, paging, command-response, status-path and MQTT-topic choices.
- **Testing:** later gates gain concrete client-shaped cases:
  - a `201` handled by a client that expects `200`;
  - a short page with a `next` link;
  - `system@link` without `system@id`;
  - undeclared-parameter rejection.

### 7.2 Effort/Complexity Estimate

| Work item | Relative complexity | Estimate | Assumptions |
|---|---|---|---|
| Phase 1 review uses this evidence | Low | Within existing review steps | No re-execution |
| Later-gate checks | Low | Absorbed by each gate review | Checks reuse planned tests |
| Future 056 subset qualification | Medium | One matrix iteration after commands exist | Permitted environment; authentication adapter |

## 8. Risks, Constraints, and Open Questions

### 8.1 Risks and Constraints

- **Not executed.** The report-screen `200` dependency, the silent truncation and the silent stream dropping are inferred from code.
- **A moving target.** OSCAR changes weekly (an open 4.0.0 pull request at retrieval). Later gate reviews should refresh the pin.
- **The fork is outside upstream.** The published-route client exists only in a personal fork, though it is published to npm. Its future is not predicted here.
- **Operational domain.** Findings were kept to API behaviour. No operational configuration is reproduced.

### 8.2 Open Questions

- **Does Glaux's HTTP boundary reject a `Content-Type` header on a bodiless GET?** OSCAR sends one on its reachability and count requests. The server documentation read here does not settle it. It is for the HTTP-boundary owner (Roadmap 1.4.2), not this study.
- **Should Guide §4.4.1's CQL2 contract say whether filters spanning several datastreams are supported?** This is for the enhanced-filtering task owner.
- **How does cs-client-ts (IDR-SRV-065), by the same developer as CS-GO, compare?** That study answers it.

## 9. Validation Against Plan Success Criteria

| Topic plan success criterion | Status | Evidence |
|---|---|---|
| Q1–Q6 answered or limited | Met | §4 |
| OSCAR commit and resolved fork revision pinned; differences from the IDR-SRV-063 baseline stated | Met: `4b49c7c`, `73dacad` (lockfile), merge base `549c630` | §§3.1, 4.1 |
| API version per data source classified; draft-era, OSH-specific and experimental behaviour kept separate | Met | §§4.1, 4.4 |
| Every material dependency traceable and classified with exact identifiers | Met | §§4.2–4.4, §12.2 |
| OS4CSAPI analysis used only as a pointer, each point confirmed or rejected | Met | §12.3 |
| Assertions distinguished from demonstrations; source distinguished from execution | Met | §§3.3, 4.5 |
| Authorship attributed; no AI-use inference | Met | §§3.1, 3.3 |
| Phase 1 compared; later-gate checks listed | Met | §4.6 |
| Each lesson has a Guide/Roadmap disposition | Met | §§4.4, 4.6, 6 |
| Relevant standards history consulted | Met: no new register entry implicated; the upstream `/command/{cmdId}/status` spelling is handled through Guide §13 | §§3.2, 4.4 |
| Report follows the template | Met | This report |

## 10. Next Steps and Handoff

1. Project-lead review and acceptance of this report. Owner: Glaux Project Lead. Due: next `proceed`.
2. On acceptance, IDR-SRV-065 (cs-client-ts) starts on its own `proceed`. Owner: Glaux research workflow.
3. Recommendations 3 and 4 wait for the separately authorised IDR-SRV-056 and CORS discussions.

No Goal, Guide, Roadmap, synthesis, server code or issue was changed by this study.

## 11. References

- [OSCAR Viewer][Oscar] at [`4b49c7c`][OscarPin]; [`earocorn/osh-js`][Fork] at [`73dacad`][ForkPin]; upstream [PR #781][PR781]
- [CSAPI Part 1][Part1]; [CSAPI Part 2][Part2]; [OGC API – Features Part 1][Features]; [CSAPI artifacts at `8e03b23`][Schemas]
- [IDR-SRV-063][R063]; [IDR-SRV-014A][R014A]; [IDR-SRV-014H][R014H]; [IDR-SRV-056][R056]; [upstream-history register][Register]
- [Guide v1.21][Guide]; [Roadmap v1.40][Roadmap]
- Glaux Server at [`27955c1`][ServerPin]: [system-read][SysRead], [authentication][Auth], [discovery][Disc]
- [OS4CSAPI OSCAR analysis][Analysis] (pointer only)

## 12. Appendices

### 12.1 Reproduction record

Commands used, all read-only:

```sh
git clone https://github.com/Botts-Innovative-Research/oscar-viewer.git   # HEAD 4b49c7c
grep -A3 '"node_modules/osh-js"' package-lock.json                        # resolved #73dacad
git -C osh-js remote add fork https://github.com/earocorn/osh-js.git && git fetch fork
git merge-base fork/add-consys origin/mcs_baseline                        # 549c630
git rev-list --count 549c630..fork/add-consys                             # 65
iconv -f UTF-16 -t UTF-8 cypress/scheduled-logs/*.log                     # run totals
```

Official OpenAPI parameter names were read from `api/part2/openapi/parameters/*.yaml` at `8e03b23`, and their uses from `api/part{1,2}/openapi/paths/*.yaml`. GitHub API counts are unauthenticated searches taken on September 29, 2026.

### 12.2 Request and dependency anchors

| Item | Anchor |
|---|---|
| Node configuration, root, Basic header, reachability, Systems, datastreams, control streams | [`Node.ts` L138–487][Node] |
| Default same-origin node, session mode | [`DataSourceContext.tsx` L118–129][DSC]; [`Navbar.tsx` L141–160][Nav] |
| Command POST | [`OSCARCommands.ts` L5–18][Cmds] |
| Report command, `200` check, WebSocket status | [`Report.tsx` L91–120][Report] |
| Video command result | [`VideoMedia.tsx` L88–100][Video] |
| Event list, count | [`EventTable.tsx` L212–245][EvtTab]; [`StatusTable.tsx` L84–110][StatTab] |
| Real-time MQTT data sources | [`LaneCollection.ts` L194–243][Lane] |
| Fork routes, paging, filters, credentials | [`routes.conf.js`][ForkRoutes]; [`Collection.js`][ForkColl]; [`Filter.js`][ForkFilter]; [`DataStreamFilter.js`][ForkDSF]; [`ConnectedSystemsApi.js`][ForkBase] |
| Fork command status route | [`Command.js` L44–77][ForkCmd] |
| MQTT topic assembly | [`MqttProvider.js` L78–110][MqttProv] |

### 12.3 OS4CSAPI OSCAR analysis, point by point

The analysis ([link][Analysis]) is dated January 31, 2026, eight months before this pin. Its points were checked against `4b49c7c`.

| Analysis point | Result at `4b49c7c` |
|---|---|
| Systems searched with `searchMembers=true` and `validTime=latest` | **Confirmed** (O2) |
| `GET /systems/{id}/subsystems` and `/systems/{id}/datastreams` used | **Rejected**: defined in `Systems.ts` but never called |
| Datastreams filtered by `system` ids, page size 1000 | **Confirmed** (O3); Systems use 500 |
| `/observations` with `dataStream`, `resultTime`, `filter`, `order=desc` | **Confirmed** (O5) |
| `/observations/count` endpoint | **Confirmed** (O6); the auth-from-local-storage detail is **outdated**, since credentials left browser storage in `c2101fe` |
| `GET /controlstreams/{id}/status` | **Partly**: used as an MQTT topic (O8), not an HTTP GET |
| `POST /controlstreams/{id}/commands` | **Confirmed** (O7) |
| MQTT for live data; "No WebSocket observed" | MQTT **confirmed**. "No WebSocket" **rejected**: the report screen streams command status over WebSocket (O9) |
| Formats `swe+binary` for video, else `swe+json`; no `Accept` negotiation | **Confirmed** |
| Page sizes up to 1000; pagination loops | **Confirmed** |

---

## Report Completion Checklist

- [x] Topic ID matches overall research plan index
- [x] Topic research plan is linked and aligned
- [x] Core research questions are covered or explicitly unresolved
- [x] Findings are evidence-backed with reproducible references
- [x] Normative and informative evidence are classified and not conflated
- [x] Mutable sources identify a version, release, tag, commit, or dated retrieval
- [x] Controlled, inaccessible, missing, or ambiguous evidence limitations are explicit
- [x] Source-backed findings, analyst inference, and project recommendations are distinguishable
- [x] Conflicts with accepted prior reports are reconciled or explicitly escalated
- [x] Executive summary is independently readable by the project lead, implementers, and later AI agents
- [x] Recommendations are explicit and actionable
- [x] Risks and open questions are documented
- [x] Success criteria validation is complete
- [ ] Plan-owner acceptance and acceptance date are recorded before the topic is treated as complete downstream
- [x] Next steps are assigned

[Oscar]: https://github.com/Botts-Innovative-Research/oscar-viewer
[OscarPin]: https://github.com/Botts-Innovative-Research/oscar-viewer/tree/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed
[Node]: https://github.com/Botts-Innovative-Research/oscar-viewer/blob/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed/src/lib/data/osh/Node.ts
[Sys]: https://github.com/Botts-Innovative-Research/oscar-viewer/blob/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed/src/lib/data/osh/Systems.ts#L35-L69
[DSC]: https://github.com/Botts-Innovative-Research/oscar-viewer/blob/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed/src/app/contexts/DataSourceContext.tsx#L118-L129
[Nav]: https://github.com/Botts-Innovative-Research/oscar-viewer/blob/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed/src/app/_components/Navbar.tsx#L141-L160
[Cmds]: https://github.com/Botts-Innovative-Research/oscar-viewer/blob/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed/src/lib/data/oscar/OSCARCommands.ts#L5-L18
[Report]: https://github.com/Botts-Innovative-Research/oscar-viewer/blob/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed/src/app/_components/reportgen/Report.tsx#L91-L120
[Video]: https://github.com/Botts-Innovative-Research/oscar-viewer/blob/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed/src/app/_components/lane-view/VideoMedia.tsx#L88-L100
[Adj]: https://github.com/Botts-Innovative-Research/oscar-viewer/blob/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed/src/app/_components/adjudication/AdjudicationDetail.tsx#L475-L494
[EvtTab]: https://github.com/Botts-Innovative-Research/oscar-viewer/blob/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed/src/app/_components/event-table/EventTable.tsx#L212-L245
[StatTab]: https://github.com/Botts-Innovative-Research/oscar-viewer/blob/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed/src/app/_components/lane-view/StatusTable.tsx#L84-L110
[Lane]: https://github.com/Botts-Innovative-Research/oscar-viewer/blob/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed/src/lib/data/oscar/LaneCollection.ts#L194-L243
[Slice]: https://github.com/Botts-Innovative-Research/oscar-viewer/blob/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed/src/lib/state/OSHSlice.tsx#L80-L98
[CyLane]: https://github.com/Botts-Innovative-Research/oscar-viewer/blob/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed/cypress/component/LaneDiscovery.cy.ts
[CyAdj]: https://github.com/Botts-Innovative-Research/oscar-viewer/blob/4b49c7cedc841c875dbb63613dfd21c34b8fa6ed/cypress/component/AdjudicationCommand.cy.ts
[CyFix]: https://github.com/Botts-Innovative-Research/oscar-viewer/commit/5c49073c5dfb83d257555509786dbf75f237f7ea
[NoStore]: https://github.com/Botts-Innovative-Research/oscar-viewer/commit/c2101fe
[Fork]: https://github.com/earocorn/osh-js
[ForkPin]: https://github.com/earocorn/osh-js/tree/73dacad5c7338722763263ef0ff5928fbd19de5b
[ForkAdd]: https://github.com/earocorn/osh-js/commit/b2e2d05b1
[ForkRoutes]: https://github.com/earocorn/osh-js/blob/73dacad5c7338722763263ef0ff5928fbd19de5b/source/core/consysapi/routes.conf.js
[ForkColl]: https://github.com/earocorn/osh-js/blob/73dacad5c7338722763263ef0ff5928fbd19de5b/source/core/consysapi/Collection.js#L45-L100
[ForkFilter]: https://github.com/earocorn/osh-js/blob/73dacad5c7338722763263ef0ff5928fbd19de5b/source/core/consysapi/Filter.js#L30-L56
[ForkDSF]: https://github.com/earocorn/osh-js/blob/73dacad5c7338722763263ef0ff5928fbd19de5b/source/core/consysapi/datastream/DataStreamFilter.js#L39-L49
[ForkBase]: https://github.com/earocorn/osh-js/blob/73dacad5c7338722763263ef0ff5928fbd19de5b/source/core/consysapi/ConnectedSystemsApi.js#L104-L161
[ForkCmd]: https://github.com/earocorn/osh-js/blob/73dacad5c7338722763263ef0ff5928fbd19de5b/source/core/consysapi/command/Command.js#L44-L77
[MqttProv]: https://github.com/earocorn/osh-js/blob/73dacad5c7338722763263ef0ff5928fbd19de5b/source/core/mqtt/MqttProvider.js#L78-L110
[PR781]: https://github.com/opensensorhub/osh-js/pull/781
[Analysis]: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/6c67c1c6ca93bd5ed3cb1eef62c17edbd11152a6/docs/research/requirements/csapi-oscarviewer-analysis.md
[Part1]: https://docs.ogc.org/is/23-001/23-001.html
[Part2]: https://docs.ogc.org/is/23-002/23-002.html
[Features]: https://docs.ogc.org/is/17-069r4/17-069r4.html
[Schemas]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api
[ServerPin]: https://github.com/DGIWG-P507/glaux-server/tree/27955c1b9260cd811ad6bc08f85feab43ad65028
[SysRead]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/system-read.md
[Auth]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/authentication.md#http-and-caller-boundary
[Disc]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/discovery.md
[R063]: idr-srv-063-osh-viewer-and-osh-js-client-study-report.md
[R014A]: idr-srv-014a-osh-csapi-server-implementation-study-report.md
[R014H]: idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md
[R056]: idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md
[Register]: ../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md
[Guide]: ../../../../../Plans/glaux-server/glaux-server-implementation-guide.md
[Roadmap]: ../../../../../Plans/glaux-server/glaux-server-roadmap.md

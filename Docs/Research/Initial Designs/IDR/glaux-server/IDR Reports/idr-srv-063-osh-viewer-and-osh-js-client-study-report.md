# Section 063: OSH Viewer and OSH JS Toolkit Client Study - Research Report

**Topic ID:** IDR-SRV-063<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-063 plan](../IDR%20Plans/idr-srv-063-osh-viewer-and-osh-js-client-study.md)<br>
**Overall Research Plan:** [Controlling overall IDR plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Q1–Q6. The client's requests, response dependencies, targeted API version and test evidence are established from source. Runtime behaviour is unexecuted and stated as a limit.<br>
**Methodology Used:** Source inspection of the pinned viewer and both labelled `osh-js` baselines. Every request was traced from the viewer's entry points to its network call, and each dependency was mapped to exact published requirements and pinned schemas. Public history, packages and the Sensor Web API draft were inventoried. Nothing was executed.<br>
**Research Time:** About 20 minutes of research and drafting, 15:29–15:50 UTC September 29, 2026, in one AI-assisted iteration, before separate review. This is not a human-hours estimate. Nothing was executed or installed.<br>
**Primary Sources:**
- [OSH Viewer at `052befa`][ViewerPin]
- [`osh-js` `mcs_baseline` at `8a959d4`][Oshjs2024] and [`d3aa99c`][OshJsPin]
- [Sensor Web API draft at `3f5b1e9`][SWAPI]
- [CSAPI Part 1][Part1], [CSAPI Part 2][Part2], [OGC API – Features Part 1][Features]
- CSAPI JSON schemas at [`8e03b23`][Schemas], as pinned in the Glaux standards corpus

**Supporting Resources:**
- [IDR-SRV-014A][R014A], [IDR-SRV-056][R056], [IDR-SRV-010][R010]
- [Guide v1.21][Guide], [Roadmap v1.40][Roadmap], [Phase 1 review charter][Charter]
- Glaux Server `main` at [`27955c1`][ServerPin]

**Document Purpose:** Independent, human-developed client evidence for the Phase 1 implementation review (especially steps 2 and 5), later review gates and IDR-SRV-056's client matrix. It is not a requirements document, and it does not authorise copying client behaviour.<br>
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

**The OSH Viewer is a well-built client for an older API, not for the published standard.** Its README says it targets the "Sensor Web API". That is an OpenSensorHub-hosted document titled "Draft OGC Sensor Web API" (2022), which preceded OGC Connected Systems. Almost every route and parameter the viewer uses comes from that draft. The viewer was last changed in April 2024 and has no tests. [README][ViewerReadme], [draft][SWAPI].

**Against Glaux today, the viewer would not work, and that is mostly the right outcome.** Reading the code shows these blocking differences:

- **Sign-in.** The viewer only sends Basic (user name and password) credentials. Glaux accepts only Bearer tokens, as its Guide requires.
- **Listing Systems.** The viewer starts by listing `/systems`. Glaux has not built that list yet; it is planned (Roadmap 2.3.1, 2.3.9, 2.5.10).
- **Old routes and parameters.** The viewer uses `/systems/{id}/members` and `/systems/{id}/controls`, where the published standard has `/subsystems` and `/controlstreams`. The viewer sends `validTime` and `f`; its toolkit sends `format` and `offset`. The OGC Features standard requires a server to answer 400 to parameters its API description does not declare, and Glaux's Guide says to reject them.
- **Wrong collection shape.** The viewer expects an `items` array of GeoJSON-shaped Systems. The published schemas use `features` for GeoJSON and `items` only for SensorML.
- **Live data.** Live data arrives over a WebSocket on the observations path, which is an OSH feature. Glaux plans SSE and MQTT instead.

None of these is a Glaux defect. Each difference is already covered by an existing Guide rule or Roadmap task. [Findings §4.2–4.4](#42-q2--requests-and-discovery).

**Where the viewer and Glaux meet, they agree.** For each System in a list, the viewer reads only `id`, `properties.uid` and `properties.name`. Glaux's implemented GeoJSON System has all three at the same paths. That is consistent with the Phase 1 representation, although the viewer never calls Glaux's single-System read. [Phase 1 comparison §4.6](#46-q6--transfer-to-glaux).

**Three findings are worth carrying into later reviews:**
1. **A Guide rule with no Roadmap task.** The Guide requires publishing only configured browser origins (CORS), and every browser client needs this. No Roadmap task names it. A bounded decision on which task owns it is worth discussing.
2. **One failed side request stops loading.** If the controls (or members) request for a System fails, the viewer shows an error and stops loading that server. That System, every later one, and all observables are dropped. Against a server without those routes, none are listed. This client pattern is worth covering in later interoperability tests.
3. **Silent loss.** The viewer ignores paging and never checks HTTP status codes. Anything past the first page, and any error body, silently disappears.

**Recommendation:** do not add the OSH Viewer to IDR-SRV-056's matrix as a compatibility target. Record it as historical, draft-era evidence, and use its three confirmed expectations as review checks. No Guide or Roadmap change is required. One bounded discussion item (CORS ownership) is proposed for the project lead.

## 2. Scope and Plan Alignment

This is **IDR-SRV-063**, the first of four client studies the project lead requested on September 29, 2026 before the Phase 1 review continues. Its objective was to establish what the OSH Viewer and the `osh-js` toolkit actually request and rely on, and to judge that against published CSAPI and Glaux's planning.

| Plan question | Coverage status | Evidence location |
|---|---|---|
| Q1 — Identity, version, provenance | Complete. The viewer targets the Sensor Web API draft; the toolkit is mostly draft-era, with one later CSAPI-aligned selector | §4.1, §12.1 |
| Q2 — Requests | Complete for every CSAPI-style request path, with each attributed to the viewer or the toolkit | §4.2, §12.2 |
| Q3 — Response dependencies | Complete from source. Runtime tolerance is inferred, not executed | §4.3 |
| Q4 — Standards alignment | Complete for every material dependency, with exact identifiers | §4.4 |
| Q5 — Tests and quality evidence | Complete. No behavioural tests exist on the CSAPI path | §4.5 |
| Q6 — Transfer to Glaux | Complete, with dispositions | §4.6, §§5–7 |

**Out of scope:**
- running the viewer, toolkit builds or tests;
- contacting maintainers, or filing upstream issues;
- auditing the non-CSAPI parts of `osh-js` (maps, video, charts, SOS);
- copying code.

Nothing was installed.

**Earlier coverage reconciled.** [IDR-SRV-014A][R014A] studied the OSH *server*. Its §§5.1, 5.3, 6.1, 6.4 and 7.2 already record legacy aliases, `offset`/`limit` paging, nested `members` routes and live WebSocket output there. This study adds the *client* side: which of those server extensions a real client depends on. [IDR-SRV-056][R056] did not inventory the OSH Viewer or `osh-js`. The OS4CSAPI client findings in 014E and 014G concern a different client and are not repeated. The discovery, query, negotiation and error baselines in [009][R009], [011][R011], [012][R012] and [013][R013] are the ones Guide §§4.1, 4.4 and 6.2 already carry, and §4.4 compares against those. Nothing in this report contradicts an accepted report.

## 3. Evidence Base

### 3.1 Primary Sources Reviewed

All access dates are **2026-09-29 (UTC)**.

| Source | Version / commit | Authority class | Stable anchor | Availability / limitations |
|---|---|---|---|---|
| [OSH Viewer][Viewer] | `main` at [`052befa`][ViewerPin], April 10, 2024. 71 commits, 67 tracked files | Observed client implementation | Paths in §§4, 12 | Complete tree read. The pin has no lockfile: this same commit deleted it. Earlier lockfiles and pins are used as narrowing evidence (§4.1) |
| [`osh-js`][OshJs] | `mcs_baseline` at [`8a959d4`][Oshjs2024] (baseline A, March 20, 2024) and [`d3aa99c`][OshJsPin] (baseline B, April 20, 2026). About 1,900 files | Observed toolkit implementation | Paths in §§4, 12 | CSAPI-path modules read in full; the rest is inventory only. Windows could not check out some long paths, so files were read from git objects |
| [Sensor Web API draft][SWAPI] | `sensorweb-api.yaml` at `3f5b1e9`, March 29, 2022. Titled "Draft OGC Sensor Web API", version `1.0.0-draft.01`, hosted by OpenSensorHub; the last merge (PR #1 from `ghobona`) "labelled the specification as a draft" | Pre-standard draft, informative | Paths and parameters listed in §4.1 | Historical. Not an OGC standard |
| [CSAPI Part 1][Part1] (23-001) and [Part 2][Part2] (23-002) | Published HTML | Normative | Requirement identifiers in §4.4 | Requirement numbers taken from the published HTML text |
| CSAPI JSON schemas | `opengeospatial/ogcapi-connected-systems` at [`8e03b23`][Schemas], as pinned in the Glaux corpus | Normative schema artifacts | `systemCollection.json` (GeoJSON and SensorML), `dataStreamCollection.json`, `dataStream.json` | Read from the Glaux server's pinned copy |
| [OGC API – Features Part 1][Features] (17-069r4) | Published HTML | Normative where CSAPI incorporates it | Requirement 8 `/req/core/query-param-unknown`; Recommendations 17–19 `/rec/core/fc-next-1` to `fc-next-3` | — |

**History inventories** (commits from local clones; pull-request, issue and release totals from the GitHub API, unauthenticated):
- Viewer: 71 commits. Nick Garay made 70 of them (65 + 5 across two email identities) and Alex Robin 1 (the initial commit).
- `osh-js`: 2,041 commits across all branches, 1,563 of them reachable from `mcs_baseline`. Mathieu Dhainaut appears under five name spellings with 1,299 commits on that branch; next are Alex Robin (126, two spellings) and Richard Becker (44).
- `osh-js` has 34 non-Dependabot remote branches, including `master` (2021), `master-nys`, `mcs_baseline` and `dev` (June 26, 2026), plus feature branches such as `ellipses`, `external_window_support` and `nexrad`. The bot `dependabot[bot]` has 13 commits on `mcs_baseline`.
- Pull requests and issues: the viewer has 4 pull requests, 1 issue, no releases and no tags. `osh-js` has 450 pull requests, 372 issues and 5 releases; its tags stop at `2.1.0`. Counts were taken, not every record read; the selected history cases below come from commits.

**npm:**
- `osh-js` has ten versions. The latest is `3.1.5` (November 24, 2025).
- `@osh-branches/osh-js` has one version, `2.2.0-mcs` (March 11, 2025), built from `5614e6f`. That commit sits between baselines A and B.

### 3.2 Supporting Sources Reviewed

| Source | Baseline | Role and limitations |
|---|---|---|
| Glaux Server docs [`system-read.md`][SysRead], [`authentication.md`][Auth], [`runtime-configuration.md`][RunCfg], [`http-boundary.md`][HttpB] | `main` at `27955c1` | Implemented Glaux behaviour for the Phase 1 comparison. Documentation and recorded proofs, not re-executed here |
| [Guide v1.21][Guide], [Roadmap v1.40][Roadmap], [Goal v1.10][Goal] | Planning `main` at `85367c0` | Controlling project scope and dispositions |
| [IDR-SRV-014A][R014A], [010][R010], [056][R056] | Accepted reports | Prior findings to reconcile, not repeat |
| [Upstream-history register][Register] | Current | Checked for `members`, `subsystems`, `obsFormat`, `om+json`, `controlstreams`, `/controls/` and Sensor Web API entries. None is implicated, so the register was not changed |

### 3.3 Evidence Quality Notes

- **Authority order:** published CSAPI text and pinned schemas; then incorporated OGC API – Features; then Glaux's Guide interpretations; then the OSH draft and client code. Client code shows what the client *does*, not what a server *should* do.
- **Source only, no execution.** Statements about what happens at runtime (for example "shows an error and stops loading") are analyst inference from the pinned code paths. They are labelled as such where it matters.
- **The exact toolkit revision is unresolved, but it barely matters here.** The viewer points at the moving branch `#mcs_baseline` with no lockfile. The 11 commits between baselines A and B change no request, route or filter code; §4.1 lists them. So every finding in this report holds at both baselines. The viewer's own history narrows this further (§4.1): baseline A is very probably what its last commit was built against.
- **The npm packages narrow nothing.** The viewer depends on the git branch, not on either npm package. `osh-js` 3.x was built from commits that no public branch contains; `b2e8954` (3.1.5) is reachable only by its SHA. Its source was not studied. It may matter to IDR-SRV-064.
- **Authorship.** The project lead reports that the viewer is among the most human-written CSAPI clients. The record shows one principal author for the viewer and one for the toolkit. This study draws no conclusion about tool or AI use, and no repository contains assistant-guidance files. Their absence is not evidence either way.

## 4. Findings by Research Question

### 4.1 Q1 — Identity, version and provenance

**Source-backed finding: the viewer targets the pre-standard "Draft OGC Sensor Web API", not published CSAPI.**
- The README lists "Sensor Web API (https://opensensorhub.github.io/sensorweb-api/swagger-ui)" as its interface. [README lines 11–21][ViewerReadme]
- That page loads `sensorweb-api.yaml`, titled "Draft OGC Sensor Web API" (version `1.0.0-draft.01`). It is hosted in an OpenSensorHub repository and was last changed on March 29, 2022, by a merge that "labelled the specification as a draft". [SWAPI][SWAPI]
- Every route the viewer and toolkit use appears in that draft: `/systems/{sysid}/members`, `/systems/{sysid}/controls` and `/datastreams/{dsid}/schema`. So do the `validTime`, `offset` and `format` parameters.
- Two parameters are not in the draft, whose format selector is `format`:
  - the viewer's `f=application/json`, which relies on OSH servers accepting `f`;
  - the toolkit's `obsFormat` on the schema request (T1). The toolkit added it in 2022 ([`24435f6`][ObsFmt1], March 24; [`8773852`][ObsFmt2], June 3), after the draft, and it matches published CSAPI Part 2 Req 11.
- So the viewer is draft-era throughout, and the toolkit is mostly draft-era with at least one later selector that matches published CSAPI.
- The draft says a missing `validTime` means "now". That explains why the viewer always sends `validTime=../..`, which asks for all validity periods.

**The toolkit's `sweapi` modules began as the same draft's client.**
- They were introduced on January 21, 2022 in [`49f434d`][Intro] ("Implement SensorWeb API Client #353").
- They keep the draft's route table at both baselines: [`routes.conf.js`][Routes].
- No branch renames them toward published CSAPI.
- Two changes on `dev` move one area toward the standard:
  - [`f434cc9`][FoiRoot] (August 6, 2024) moves the top-level feature-of-interest routes to `/samplingFeatures`;
  - [`9e91d61`][Foi] (February 25, 2025) moves the nested `/systems/{id}/samplingFeatures` route and adds a `foi` parameter; its commit body says "(see CSAPI spec)".
- Neither is on `mcs_baseline`, and both leave `members` and `controls` unchanged.

**Pins and the difference between the two baselines.**
- The viewer is `052befa` (April 10, 2024).
- Baseline A (`8a959d4`) is the `mcs_baseline` commit current at that date. Baseline B (`d3aa99c`) is the branch head.
- A is an ancestor of B, and B adds 11 commits:
  - video views, an external-window view, a line layer, ellipse fixes, and packaging;
  - one data-path change, in [`AbstractParser.js`][ParserDiff]: in B, a *named* nested SWE `DataRecord` is flattened into its parent instead of being nested under its name.
- The viewer finds values by recursive key search (`findInObject`, below), so it tolerates either shape.

**The viewer's own history narrows which toolkit it ran.**

| Viewer commit | `osh-js` reference |
|---|---|
| [`d0c3208`][VLock1] to [`b2e31b0`][VLock2], December 2022 | Lockfile pins `d49ae6b`, then `6a9b329`, then `f709d34` |
| [`6f5be9a`][VPeg], May 1, 2023, "Peg dependency to a specific commit of osh-js" | `package.json` pins `#416f8ee` (April 17, 2023). The lockfile was touched here and in `7532b11` but never re-resolved; it still says `f709d34` |
| [`052befa`][ViewerPin], April 10, 2024 | Switches `package.json` to `#mcs_baseline`, deletes the lockfile, and says it makes "updates to data synchronizer based on osh-js changes" |

- On April 10, 2024 the `mcs_baseline` head was baseline A. So A is very probably what the final commit was written against. That is still inference: nothing records the revision actually installed.
- From `416f8ee` to A, the toolkit's routes, paths and query parameters did not change. Three relevant things did (the diff also adds the unused time updater, an MQTT topic connector and reconnect tweaks):
  - toolkit reads started sending browser-managed credentials (`credentials: 'include'`, December 2023);
  - OM JSON results stopped being flattened in memory;
  - **the data source's default format changed from `application/om+json` to `application/swe+json`** ([`SweApi.datasource.js` L52][DS]).
- So a viewer built at the May 2023 pin requested the OSH-era `om+json` format for location data, while one built against A or B requests the published SWE JSON token.

| Selected history case | What changed | Relevance |
|---|---|---|
| [`49f434d`][Intro], 2022-01-21 | Toolkit client for the Sensor Web API draft introduced | Fixes the draft-era route model |
| [`da91cde`][Replay], 2022-10-03 | Replay connector with double buffer (#665) | Origin of the paged HTTP replay path used for playback |
| [`738c2f6`][Cred], 2023-12-22 | "Fix missing credentials headers": adds `credentials: 'include'` to a fetch | Credentials reach toolkit reads only through browser-managed credentials, not the viewer's Basic token |
| [`f434cc9`][FoiRoot] 2024-08-06 and [`9e91d61`][Foi] 2025-02-25 (`dev` only) | FOI routes moved to `samplingFeatures`; the second commit body says "(see CSAPI spec)" | The only observed route moves toward published CSAPI; not in the viewer's baselines |
| Viewer [`7532b11`][VRefactor], 2023-05-20 | "Refactor for sensor systems compatability" | Introduces the `members` and `controls` system traversal the viewer still uses |

**Interpretation:** treat the viewer's behaviour as evidence of how a careful client of a *pre-standard draft API, as OSH servers implement it,* works. It is not evidence of what published CSAPI clients expect.

### 4.2 Q2 — Requests and discovery

**Source-backed finding: the viewer performs no discovery.**
- It appends the fixed prefix `/sensorhub/api` to a server address the user types in, then builds every path by hand. [`Constants.tsx` L61–69][Const]
- It never reads the landing page, the API description, conformance or collections, and it follows no links.
- The full request table, with anchors, is in [§12.2](#122-request-and-dependency-table). In summary:

| # | Request | Built by | Mode |
|---|---|---|---|
| V1 | `GET /systems?f=application/json&validTime=../..` | Viewer | Start-up / add server |
| V2 | `GET /systems/{id}/members?f=application/json&validTime=../..` | Viewer | Per top-level System |
| V3 | `GET /systems/{id}/controls` | Viewer | Per top-level System |
| V4 | `GET /datastreams?f=application%2Fjson` | Viewer | Once per server |
| V5 | `GET /datastreams/{id}/schema?f=application%2Fjson` | Viewer | Per datastream |
| V6 | Browser window: `/systems/{id}` | Viewer | "Describe" button |
| V7 | Browser window: `/sensorhub/sos?service=SOS&version=2.0&request=GetCapabilities` | Viewer | **SOS, outside CSAPI** |
| T1 | `GET /datastreams/{id}/schema?obsFormat=application/swe%2Bjson` (or `swe%2Bbinary`) | Toolkit | Before parsing observations |
| T2 | `ws(s)://…/datastreams/{id}/observations?format=application/swe%2Bjson` | Toolkit | Live mode |
| T3 | `GET /datastreams/{id}/observations?phenomenonTime={start}/{end}&format=…&offset={n}&limit=250` | Toolkit | Playback mode |

**Protocol details:**
- The SPS path `/sensorhub/sps` is stored in the server record, but no code sends a request to it.
- V1–V5 always send `Authorization: Basic base64(user:password)` and `Content-Type: application/json`, even on GET, with `credentials: "include"`. V1–V3 set `mode: "cors"` explicitly; V4–V5 get it as the browser default. [`SystemRequest.tsx` L24–36][SysReq], [`AddServer.tsx` L79, L123][AddServer]
- Toolkit reads (T1, T3) send no Authorization header, because the viewer passes no `connectorOpts`. They rely on `credentials: 'include'`. [`SensorWebApi.js` L101–136][SWA], [`Collection.js` L44–52][Coll]
- A browser WebSocket (T2) cannot carry an Authorization header at all.
- The toolkit requests formats through the `format` query parameter, never the `Accept` header. Location data uses the data source's default, `application/swe+json` ([`SweApi.datasource.js` L52][DS]); the viewer sets `application/swe+binary` only for video and draped imagery ([`VideoStreamObservables.tsx` L57][Video]). The toolkit's own default is `application/om+json` ([`ObservationFilter.js` L39][ObsFilter]).
- The toolkit does not URL-encode filter values, except that it replaces `+` with `%2B` in format values. [`Filter.js` L30–56][Filter]
- **Mode:** the viewer switches the toolkit between live (T2) and playback (T3). [`Slice.tsx` L414–419][Slice]
- **Not used by the viewer:** the toolkit has an optional time updater. Every 5 seconds it polls `GET /datastreams?id={id}&select=id,phenomenonTime&f=application%2Fjson` and reads `items[0].phenomenonTime`. [`SweApi.datasource.updater.js` L36–43][Updater] Only the toolkit's own Vue time controller turns it on. The viewer uses its own React controller and never calls `autoUpdateTime`, so this request is inventoried but not attributed to the viewer.

### 4.3 Q3 — Response dependencies and tolerance

**Source-backed finding: the viewer reads few fields, finds them tolerantly, and loses data silently.**

| Response | Fields read | If missing or unexpected (inference from code) |
|---|---|---|
| V1/V2 Systems | `items[]`, then per item `id`, `properties.uid`, `properties.name`. [L47–55][SysReq] | No `items`: a `TypeError` from iterating `undefined`; the server load fails with an error popup. Missing `uid`/`name`: shown as undefined, no failure |
| V3 Controls | `items[]`, then `name`, `description`, `id`, `inputName`, `system@id`. [L159–168][SysReq] | No `items` (for example, a 404 Problem body) throws, as it does for V2. In [`App.tsx` L86–112][App] each System is added only after its own V2/V3 calls succeed, so a throw stops the loop: **that System, later ones and all observables are dropped**, and an error popup appears. Earlier Systems stay listed. Against a server without the route, the first System fails, so none are listed |
| V4 Datastreams | `items`, then per item, by recursive key search, `system@id`, `phenomenonTime[0..1]`, `id`. [`ObservableUtils.tsx` L46–80][ObsUtil] | No `system@id`: the datastream is skipped silently. `phenomenonTime[0] === '0'` is treated as "no time" |
| V5 Schema | Recursive search for `resultSchema`, then for `definition`. [L82–86][ObsUtil] | No schema: no visualisation is built for that datastream, silently |
| Visualisation choice | Suffixes of the SWE `definition` URI: `/Location`, `/PlatformLocation`, `/SensorLocation`, `/GPS`, `/OrientationQuaternion`, `/PlatformOrientation`. [`PliObservables.tsx` L49–79][Pli] | Unrecognised definitions are ignored |
| Observation records | Any key named `lon`, `x` or `longitude` (and so on); `heading` or `yaw`; `id`, `uid` or `source`. [L122–137][Pli] | Heuristic. A nested key of the same name can be picked up in place of the intended one |
| T1 Schema (toolkit) | `resultSchema` for OM JSON; `recordSchema` and `recordEncoding` for SWE formats. [`SweApiResult.datastream.parser.js`][ResultParser] | Unsupported format: throws |

**Tolerance, strictness and silent loss:**
- `findInObject` accepts alternative names and searches nested objects when the key is not at the top level. Its loop over nested objects has no early exit, so the last object-valued branch wins and can even reset an earlier match to `null`. [`Utils.tsx` L54–94][Utils] That tolerates different member placements, but it can miss a value or take one from the wrong nesting level.
- **The viewer never checks the HTTP status.** Its `fetch(...).catch` handles only network failure. [`DataStreamsRequest.tsx` L35–40][DsReq] An error response is parsed as if it were data. The toolkit does check `response.ok`. [`Collection.js` L53–57][Coll]
- **The viewer never pages.** V1, V2 and V4 read one response and ignore any `next` link, so resources beyond the server's first page silently disappear.
- **The toolkit pages by computing numeric `offset` values.** It stops when a page is short. [`Collection.js` L44–98][Coll] The draft itself describes `offset` as a token "usually provided in the previous call" ([draft][SWAPI]), so computing it is a client assumption even against the draft.
- **Two toolkit defects:**
  - a region-of-interest filter is dropped because it reads `props.roi` instead of `properties.roi`;
  - `select.concat(...)` discards its result, so included properties are never sent.

  [`SweApi.context.js` L52–54, L76–81][Ctx] The viewer uses neither feature.

### 4.4 Q4 — Standards alignment

Classification key: **C** conforming; **T** tolerant; **D** draft-era (the Sensor Web API draft); **O** OSH-specific deployment convention; **N** conflicts with the published standard.

| Dependency | Published text | Class | Glaux position |
|---|---|---|---|
| API root fixed at `/sensorhub/api` | CSAPI uses a deployment-chosen `{api_root}` | O | Glaux can serve under a path prefix behind a proxy ([runtime configuration L71–75][RunCfg]). Deployment matter; no change |
| `GET /systems` list (V1) | Part 1 Req 6 `/req/system/resources-endpoint` | C (path) | Not yet: `GET /systems` returns 405 ([system-read][SysRead]). Planned: Roadmap 2.3.1, 2.3.9 |
| `items[]` of GeoJSON-shaped Systems for `application/json` | Pinned schemas: GeoJSON `systemCollection.json` is a FeatureCollection with **`features`**; SensorML `systemCollection.json` has **`items`** of SensorML Systems (`uniqueId`, `label`) | N/O (hybrid of the two) | Glaux follows the schemas: [Guide §6.2][G62], Roadmap 2.3.9. No change |
| `/systems/{id}/members` (V2) | Part 1 Req 9 `/req/subsystem/collection`: `{api_root}/systems/{parentId}/subsystems` | D | Roadmap 2.3.6 uses `subsystems`. No alias proposed |
| `/systems/{id}/controls` (V3) | Part 2 Req 22 `/req/controlstream/ref-from-system` and the `/controlstreams` endpoints. Req 19's canonical `/controls/{id}` is itself an upstream inconsistency | D | [Guide §13][G13] already selects `/controlstreams`, with "no automatic unsafe-method aliases". No change |
| `system@id` on datastreams and controls | Part 2 `dataStream.json` uses **`system@link`** (object with `href`, `uid`) | D | Glaux follows the schema. The viewer would silently show no observables against a published-shape server |
| `validTime=../..` query parameter | Part 1 Req 3 `/req/api-common/datetime` filters on `validTime` through **`datetime`** | D | Glaux accepts `..` in query intervals ([Guide §13][G13]) on `datetime`, not `validTime` |
| `f=application/json` (viewer), `format=…` (toolkit) | Features Req 8 `/req/core/query-param-unknown`: 400 for parameters not in the API definition | `f` O (not in the draft; accepted by OSH servers); `format` D | [Guide §6.2][G62] excludes `f`; [Guide §4.4][G44] rejects unsupported parameters. Glaux would answer 400; that conforms |
| Schema without `obsFormat` (V5) | Part 2 Req 11 `/req/datastream/schema-op`: `obsFormat` is `required: true` | N | Roadmap 3.1.6 requires `obsFormat` with distinct errors. The viewer loses the visualisation silently |
| Schema with `obsFormat` (T1), `resultSchema`/`recordSchema` shapes | Part 2 Req 11; Req 110 `/req/swecommon-json/obsschema-mapping` | C | Roadmap 3.1.6, 4.4.4 |
| `application/swe+json`, `application/swe+binary` | Part 2 SWE encodings (a note says implementations should use the preliminary `application/vnd.ogc.swe+json` until SWE Common 3.0 is stable) | C | [Guide §13][G13]: CSAPI tokens, with vendor aliases only where declared |
| `application/om+json` (filter default; data-source default up to `416f8ee`) | Not defined in Part 2, whose ordinary observation JSON is `application/json` | D | Not supported by Glaux. Used by viewer builds at the May 2023 pin; not at A or B |
| `offset`/`limit` paging | Features Rec 17–19 `/rec/core/fc-next-1..3` (`next` links, recommended). No public `offset` | D | [Guide §4.4][G44]: opaque `next` links, "not a promised public offset API". Roadmap 2.5.10, 3.3.6 |
| `phenomenonTime={start}/{end}` on observations | Part 2 Req 49 `/req/advanced-filtering/obs-by-phenomenontime` | C | Roadmap 3.3.2 |
| Live WebSocket on `/datastreams/{id}/observations` | Not in Parts 1–2. Not the draft Part 3 binding Glaux selected | O | [Guide §4.8][G48]: SSE and MQTT. No WebSocket; no change |
| Basic authentication only | CSAPI leaves security to the deployment | O | [Authentication][Auth]: Bearer only; unsupported schemes get 401 with a `Bearer` challenge. [Guide §4.10][G410] selects token authentication. No change |
| No HTTP status check; no `next` links | Features Rec 17–19 (`next`, recommended), and HTTP semantics generally | Client robustness, not a conformance class | Useful as a negative interoperability case (below) |

**Interpretation:** only three of the viewer's dependencies are published-standard behaviour that a conforming server must serve:
- the Systems list path;
- the `obsFormat` schema selector (toolkit only);
- the SWE media types.

The rest are draft-era or OSH-specific. Glaux's existing choices match the published text in every compared row. No row reveals a Glaux interpretation that needs revisiting.

### 4.5 Q5 — Tests and quality evidence

**Source-backed finding: neither repository has behavioural tests on the CSAPI path.**
- The viewer has no test files, test script or CI workflow. [Tree][ViewerPin]
- `osh-js` has no test script in `package.json` and no CI workflow.
- `osh-js` has one test file, the unchanged Create-React-App placeholder `renders learn react link` in `demos/3dr-solo-uav/…/App.test.js`. [Test][PlaceholderTest] It exercises nothing in the toolkit.

The toolkit ships **demonstrations**, not assertions:
- the `demos/sweapi` Vue application (historical and live observations, commands, schema, pagination views);
- `showcase` examples such as `chart-archive-realtime-synchronized-sweapi`;
- `showcase-dev/…/datasource-sweapifetch`.

These show intended usage against a live OSH server. They make no assertions, so a failure would appear only as a visibly wrong screen. They are not treated as tests.

**Interpretation:** the evidence here is careful production code by long-standing developers, not verified behaviour. Any expectation Glaux takes from this client must be independently asserted in Glaux's own tests. That matches [Guide §8.3][G83]: "A client workaround is evidence of interoperability behavior, not permission to change the standard contract."

### 4.6 Q6 — Transfer to Glaux

**Phase 1 comparison (System create and read, server `27955c1`):**

| Viewer expectation | Glaux today | Result |
|---|---|---|
| Each listed System item has `id`, `properties.uid`, `properties.name` | The same paths in Glaux's single-System GeoJSON Feature ([system-read][SysRead]) | **Consistent.** The item shape matches, although the viewer reads lists, not Glaux's single-System read |
| `featureType`, `links`, geometry | Not read by the viewer | Glaux exceeds; no conflict |
| List Systems at `/systems` | 405; no collection yet | **Conflict for now**; planned (2.3.1, 2.3.9, 2.5.10) |
| "Describe" opens `/systems/{id}` in a browser tab | The browser sends `Accept: …*/*;q=0.8`. Glaux treats `*/*` as acceptable ([system-read tests][SysReadTests]), but the tab sends no Bearer token, so 401 | Blocked by authentication, by design |
| Basic credentials | Bearer only | **Conflict, by design** ([Guide §4.10][G410]) |
| Cross-origin browser calls with `Authorization` and `Content-Type` (preflighted) | No CORS handling implemented. The Guide says "Publish only configured browser origins" ([§4.10][G410]) | Not yet implemented; **no Roadmap task names it** (see R2) |
| Root path `/sensorhub/api` | Configurable public root with a path prefix | Deployable |

**Checks for later gates**, derived from this client and phrased as Glaux's own expectations, not the client's:

| Gate area (Roadmap) | Check to include |
|---|---|
| 2.3.1 / 2.3.9 / 2.5.10 Systems lists | GeoJSON lists use `features`, and SensorML lists use `items`. Unknown parameters such as `validTime`, `f`, `format` and `offset` give 400, not silent acceptance. `next` links are opaque |
| 2.3.6 Subsystems | `/systems/{id}/subsystems` works; `/members` is not silently aliased |
| 3.1.6 Schema | A missing `obsFormat` gives the documented 400. `resultSchema` versus `recordSchema`/`recordEncoding` follow the selected format |
| 3.1 / 3.2 Datastreams | Parent reference is `system@link` (with `href`), never an undocumented `system@id` |
| 5.1 ControlStreams | Nested discovery is `/systems/{id}/controlstreams`. A request to `/systems/{id}/controls` gives a clear 404 Problem, not a success-shaped body |
| 4.4.4 SWE formats | The `swe+json` and `swe+binary` tokens negotiate through `Accept` and `obsFormat`; the `format` query parameter is rejected |
| Browser access (Guide §4.10) | Configured-origin CORS, including preflight with `Authorization`, once a task owns it |

**IDR-SRV-056 matrix recommendation:**
- Do not add the OSH Viewer, or `osh-js` `mcs_baseline`, as a compatibility target. Their protocol is the OSH draft. Making them pass would require the aliases, vendor parameters, Basic authentication and WebSocket transport that Glaux's Guide deliberately excludes.
- Record the pair in IDR-SRV-056's inventory as a *historical, draft-era* client.
- Reuse three patterns as negative interoperability cases:
  - a client ignoring HTTP status;
  - a client ignoring `next` links;
  - a client that stops loading a server when one side request fails.
- If a later, CSAPI-aligned `osh-js` (for example the unpinned 3.x line, or the OSCAR fork studied in IDR-SRV-064) proves to target published CSAPI, it can be qualified separately.

## 5. Decision Analysis

| Option | Benefits | Costs / risks | Standards impact | Recommendation |
|---|---|---|---|---|
| A. Keep Glaux as planned; use this report as review evidence | No scope change; Glaux already matches the published text on every compared row | Some OSH-era clients will not work with Glaux | Preserves conformance | **Keep** |
| B. Add OSH compatibility aliases (`members`, `controls`, `validTime`, `format`, `offset`, Basic auth, WebSocket) | The OSH Viewer might work | Large new surface. Contradicts Guide §§4.4, 4.8, 4.10, 6.2 and 13. Weakens the 400-on-unknown behaviour | Conflicts with Features Req 8 unless every alias is declared, and adds non-standard behaviour | **Reject** |
| C. Add the OSH Viewer to IDR-SRV-056 as a target | Human-written client coverage | Would test draft-era behaviour Glaux intentionally lacks; no tests to reuse | Misleading pass/fail signal | **Reject**; record it as historical only |
| D. Decide which Roadmap task owns configured-origin CORS | Closes a gap between the Guide and the Roadmap that affects every browser client | One small planning decision | None; Guide §4.10 already requires it | **Conditional**: discuss with the project lead |

## 6. Key Recommendations

1. **Use this report as Phase 1 review evidence (steps 2 and 5).** The Phase 1 System representation is consistent with the fields an independent human-written client reads. The list, subsystem, schema and authentication differences are deliberate and already planned.
   - Rationale: §4.6.
   - Preconditions: project-lead acceptance.
   - Priority: High.
2. **Discuss which Roadmap task owns configured-origin CORS.** Guide §4.10 requires it and every browser client needs it, but no Roadmap task names it. A candidate is a small clarification to an existing leaf, such as 1.4.2's HTTP boundary or a later HTTP task, not a new phase.
   - Rationale: §4.6.
   - Preconditions: a project-lead decision; no change is made by this report.
   - Priority: Medium.
3. **Carry the later-gate checks in §4.6 into the relevant gate reviews.** These are review prompts, not new requirements.
   - Priority: Medium.
4. **Record the OSH Viewer and `osh-js` `mcs_baseline` in IDR-SRV-056 as historical, draft-era clients, not targets.** Reuse the three negative patterns in §4.6 as interoperability cases.
   - Preconditions: a separately authorised IDR-SRV-056 discussion.
   - Priority: Low.
5. **No Guide or Roadmap change is otherwise proposed.** Every other lesson is already covered.

## 7. Implementation Implications and Estimates

### 7.1 Implications

- **Architecture:** none. The findings support the existing HTTP boundary, negotiation, paging and authentication design.
- **Implementation:** no new work is implied. Recommendation 2 may attach configured-origin CORS to an existing task.
- **Testing:** later gates gain concrete negative cases: unknown-parameter rejection, `system@link` shape, `subsystems` and `controlstreams` paths, and a missing `obsFormat`. IDR-SRV-056 gains three client-robustness patterns.

### 7.2 Effort/Complexity Estimate

| Work item | Relative complexity | Estimate | Assumptions |
|---|---|---|---|
| Phase 1 review uses this evidence | Low | Within the existing review steps | No re-execution needed |
| CORS ownership decision (if taken) | Low | One planning edit, then one existing-task increment | Project lead approves; scope limited to configured origins and preflight |
| Later-gate checks | Low | Absorbed by each gate's review | Checks are reused, not new tasks |

## 8. Risks, Constraints, and Open Questions

### 8.1 Risks and Constraints

- **Not executed.** Runtime claims, such as the error popup or silently empty observables, come from reading code paths. A permitted isolated environment could confirm them; none was used.
- **Moving toolkit reference.** A future `mcs_baseline` commit could change the request paths. This report is pinned to A and B, which are equivalent for every finding.
- **`osh-js` 3.x is unexamined.** Its source commits are not on public branches. It may target published CSAPI, and this report makes no claim about it.
- **Draft-era clients in the field.** Users of OSH-era tools may expect Glaux to work with them. The report recommends explaining the draft/published difference rather than adding aliases.

### 8.2 Open Questions

- **Should an anonymous read-only Glaux deployment ignore, or reject, an unsolicited Basic header?** The viewer always sends one. Guide §4.10 allows anonymous read deployments, and current authentication rejects unsupported schemes. This is for the owner of the anonymous-read policy option, not for this study.
- **Which Roadmap task owns configured-origin CORS?** See Recommendation 2.
- **Does the OSCAR Viewer's `osh-js` fork (`add-consys`) move to published CSAPI?** IDR-SRV-064 answers this and compares against this report.

## 9. Validation Against Plan Success Criteria

| Topic plan success criterion | Status | Evidence |
|---|---|---|
| Q1–Q6 have evidence-backed answers or explicit limits | Met | §4 |
| Viewer commit and both `osh-js` baselines pinned; missing lockfile and narrowing evidence recorded | Met: the lockfile was deleted at the pin; earlier pins and the final commit's message point to baseline A | §§3.1, 3.3, 4.1 |
| Every request attributed to the viewer or the toolkit; SOS/SPS classified as outside CSAPI | Met | §4.2, §12.2 |
| Targeted API version established | Met: the viewer targets the Sensor Web API draft; the toolkit is mostly draft-era | §4.1 |
| Every material dependency traceable to an anchor and classified with exact identifiers | Met | §§4.3–4.4, §12.2 |
| Draft-era and OSH-specific behaviour kept separate from published expectations | Met | §4.4 classification |
| Tests and examples: assertions distinguished from demonstrations; source distinguished from execution | Met | §§3.3, 4.5 |
| Authorship attributed; no AI-use inference | Met | §3.3 |
| Phase 1 compared with implemented Glaux; later-gate checks listed | Met | §4.6 |
| Each material lesson has a disposition with Guide/Roadmap references | Met | §§4.4, 4.6, 6 |
| Relevant standards history consulted and authority-classified | Met: no register entry implicated; the upstream `/controls/{id}` inconsistency is cited through Guide §13 | §§3.2, 4.4 |
| Report follows the template and validates the criteria | Met | This report |

## 10. Next Steps and Handoff

1. Project-lead review and acceptance of this report. Owner: Glaux Project Lead. Due: next `proceed`.
2. On acceptance, IDR-SRV-064 (OSCAR Viewer) starts on its own `proceed`. It compares its `osh-js` fork with baselines A and B here (the fork's merge base `549c630` is A's parent). Owner: Glaux research workflow.
3. Recommendation 2 (CORS ownership) waits for a project-lead decision. This report changes no planning document.

No Goal, Guide, Roadmap, synthesis, server code or issue was changed by this study.

## 11. References

- [OSH Viewer][Viewer] at [`052befa`][ViewerPin]; [`osh-js`][OshJs] at [`8a959d4`][Oshjs2024] and [`d3aa99c`][OshJsPin]
- [Sensor Web API draft][SWAPI]
- [CSAPI Part 1][Part1]; [CSAPI Part 2][Part2]; [OGC API – Features Part 1][Features]; [CSAPI schemas at `8e03b23`][Schemas]
- [IDR-SRV-010][R010]; [IDR-SRV-014A][R014A]; [IDR-SRV-056][R056]; [upstream-history register][Register]
- [Goal v1.10][Goal]; [Guide v1.21][Guide]; [Roadmap v1.40][Roadmap]; [Phase 1 review charter][Charter]
- Glaux Server at [`27955c1`][ServerPin]: [system-read][SysRead], [system-read tests][SysReadTests], [authentication][Auth], [runtime configuration][RunCfg], [HTTP boundary][HttpB]

## 12. Appendices

### 12.1 Reproduction record

Commands used, all read-only:

```sh
git clone https://github.com/opensensorhub/osh-viewer.git   # HEAD 052befa
git clone https://github.com/opensensorhub/osh-js.git       # read via git show <commit>:<path>
git rev-list --count 8a959d4..d3aa99c                        # 11
git diff --stat 8a959d4 d3aa99c                              # 8 files; only AbstractParser.js on the data path
git log --diff-filter=A -- source/core/sweapi/routes.conf.js # 49f434d
git log -- package-lock.json                                 # in viewer: deleted in 052befa
git show 6f5be9a -- package.json                             # in viewer: #dev -> #416f8ee
git diff 416f8ee 8a959d4 -- source/core/sweapi source/core/datasource/sweapi
curl https://raw.githubusercontent.com/opensensorhub/sensorweb-api/master/sensorweb-api.yaml
curl https://registry.npmjs.org/osh-js ; curl https://registry.npmjs.org/@osh-branches%2Fosh-js
```

The standards were read from the published HTML. Schemas were read from the Glaux corpus copy of `ogcapi-connected-systems` at `8e03b23`. No build, test or network call to any live server was made.

### 12.2 Request and dependency table

| # | Method and path (after `{address}/sensorhub/api`) | Parameters and headers | Built in | Response members used | Class |
|---|---|---|---|---|---|
| V1 | `GET /systems` | `f=application/json`, `validTime=../..`; Basic; `Content-Type: application/json` | [`SystemRequest.tsx` L24–55][SysReq] | `items[].id`, `.properties.uid`, `.properties.name` | Path C; parameters D; shape N/O |
| V2 | `GET /systems/{id}/members` | As V1 | [L81–112][SysReq] | As V1 | D |
| V3 | `GET /systems/{id}/controls` | Basic | [L137–168][SysReq] | `items[].name, description, id, inputName, system@id` | D |
| V4 | `GET /datastreams` | `f=application%2Fjson`; Basic | [`DataStreamsRequest.tsx` L22][DsReq] | `items`; recursive `system@id`, `phenomenonTime`, `id` | D |
| V5 | `GET /datastreams/{id}/schema` | `f=application%2Fjson` (no `obsFormat`); Basic | [`DataStreamSchemaRequest.tsx` L21][SchemaReq] | recursive `resultSchema`, `definition` | N |
| V6 | Browser tab `/systems/{id}` | Browser defaults | [`DescribeSystemRequest.tsx` L21–23][Describe] | Displayed only | C (path) |
| V7 | Browser tab `/sensorhub/sos?service=SOS&version=2.0&request=GetCapabilities` | — | [`CapabilitiesRequest.tsx` L21–22][Caps] | Displayed only | Outside CSAPI |
| T1 | `GET /datastreams/{id}/schema` | `obsFormat=application/swe%2Bjson` or `%2Bbinary`; cookies only | [`DataStream.js` L86–90][DataStream]; [`SweApiResult.parser.js` L31–57][ResultBase] | `recordSchema`, `recordEncoding` (or `resultSchema`) | C |
| T2 | WebSocket `…/datastreams/{id}/observations` | `format=…`; no headers possible | [`DataStream.js` L50–61][DataStream]; [`WebSocketConnector.js` L61–75][WS] | SWE records | O |
| T3 | `GET /datastreams/{id}/observations` | `phenomenonTime=a/b`, `format=…`, `offset=n`, `limit=250`; cookies only | [`DataStream.js` L71–78][DataStream]; [`Collection.js` L44–98][Coll]; [replay context L56–73][Replay2] | SWE records | Time C; paging D |

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

[Viewer]: https://github.com/opensensorhub/osh-viewer
[ViewerPin]: https://github.com/opensensorhub/osh-viewer/tree/052befa09b8f076520bff77ef5d39f6624b6411d
[ViewerReadme]: https://github.com/opensensorhub/osh-viewer/blob/052befa09b8f076520bff77ef5d39f6624b6411d/README.md#L11-L21
[Const]: https://github.com/opensensorhub/osh-viewer/blob/052befa09b8f076520bff77ef5d39f6624b6411d/src/data/Constants.tsx#L61-L69
[SysReq]: https://github.com/opensensorhub/osh-viewer/blob/052befa09b8f076520bff77ef5d39f6624b6411d/src/net/SystemRequest.tsx
[DsReq]: https://github.com/opensensorhub/osh-viewer/blob/052befa09b8f076520bff77ef5d39f6624b6411d/src/net/DataStreamsRequest.tsx#L22-L40
[SchemaReq]: https://github.com/opensensorhub/osh-viewer/blob/052befa09b8f076520bff77ef5d39f6624b6411d/src/net/DataStreamSchemaRequest.tsx#L21
[Describe]: https://github.com/opensensorhub/osh-viewer/blob/052befa09b8f076520bff77ef5d39f6624b6411d/src/net/DescribeSystemRequest.tsx#L21-L23
[Caps]: https://github.com/opensensorhub/osh-viewer/blob/052befa09b8f076520bff77ef5d39f6624b6411d/src/net/CapabilitiesRequest.tsx#L21-L22
[AddServer]: https://github.com/opensensorhub/osh-viewer/blob/052befa09b8f076520bff77ef5d39f6624b6411d/src/components/servers/AddServer.tsx#L79-L124
[App]: https://github.com/opensensorhub/osh-viewer/blob/052befa09b8f076520bff77ef5d39f6624b6411d/src/components/App.tsx#L86-L112
[ObsUtil]: https://github.com/opensensorhub/osh-viewer/blob/052befa09b8f076520bff77ef5d39f6624b6411d/src/observables/ObservableUtils.tsx#L40-L101
[Utils]: https://github.com/opensensorhub/osh-viewer/blob/052befa09b8f076520bff77ef5d39f6624b6411d/src/utils/Utils.tsx#L54-L94
[Pli]: https://github.com/opensensorhub/osh-viewer/blob/052befa09b8f076520bff77ef5d39f6624b6411d/src/observables/PliObservables.tsx#L49-L137
[Video]: https://github.com/opensensorhub/osh-viewer/blob/052befa09b8f076520bff77ef5d39f6624b6411d/src/observables/VideoStreamObservables.tsx#L49-L57
[Slice]: https://github.com/opensensorhub/osh-viewer/blob/052befa09b8f076520bff77ef5d39f6624b6411d/src/state/Slice.tsx#L414-L419
[VRefactor]: https://github.com/opensensorhub/osh-viewer/commit/7532b110c4c6ec3c1b59f4880954d9ac1d96553e
[VLock1]: https://github.com/opensensorhub/osh-viewer/commit/d0c3208
[VLock2]: https://github.com/opensensorhub/osh-viewer/commit/b2e31b0
[VPeg]: https://github.com/opensensorhub/osh-viewer/commit/6f5be9a
[Updater]: https://github.com/opensensorhub/osh-js/blob/8a959d440ea989dd32c6894229f051df8f5c87da/source/core/datasource/sweapi/SweApi.datasource.updater.js#L36-L43
[OshJs]: https://github.com/opensensorhub/osh-js
[Oshjs2024]: https://github.com/opensensorhub/osh-js/tree/8a959d440ea989dd32c6894229f051df8f5c87da
[OshJsPin]: https://github.com/opensensorhub/osh-js/tree/d3aa99cef7de0b4e6f928b1a77f07b898f6747e9
[Routes]: https://github.com/opensensorhub/osh-js/blob/8a959d440ea989dd32c6894229f051df8f5c87da/source/core/sweapi/routes.conf.js#L18-L54
[DS]: https://github.com/opensensorhub/osh-js/blob/8a959d440ea989dd32c6894229f051df8f5c87da/source/core/datasource/sweapi/SweApi.datasource.js#L52-L56
[Ctx]: https://github.com/opensensorhub/osh-js/blob/8a959d440ea989dd32c6894229f051df8f5c87da/source/core/datasource/sweapi/context/SweApi.context.js#L50-L84
[Replay2]: https://github.com/opensensorhub/osh-js/blob/8a959d440ea989dd32c6894229f051df8f5c87da/source/core/datasource/sweapi/context/SweApi.replay.context.js#L56-L73
[DataStream]: https://github.com/opensensorhub/osh-js/blob/8a959d440ea989dd32c6894229f051df8f5c87da/source/core/sweapi/datastream/DataStream.js#L50-L90
[Coll]: https://github.com/opensensorhub/osh-js/blob/8a959d440ea989dd32c6894229f051df8f5c87da/source/core/sweapi/Collection.js#L44-L98
[Filter]: https://github.com/opensensorhub/osh-js/blob/8a959d440ea989dd32c6894229f051df8f5c87da/source/core/sweapi/Filter.js#L30-L56
[ObsFilter]: https://github.com/opensensorhub/osh-js/blob/8a959d440ea989dd32c6894229f051df8f5c87da/source/core/sweapi/observation/ObservationFilter.js#L31-L43
[SWA]: https://github.com/opensensorhub/osh-js/blob/8a959d440ea989dd32c6894229f051df8f5c87da/source/core/sweapi/SensorWebApi.js#L101-L136
[ResultBase]: https://github.com/opensensorhub/osh-js/blob/8a959d440ea989dd32c6894229f051df8f5c87da/source/core/parsers/sweapi/observations/SweApiResult.parser.js#L31-L57
[ResultParser]: https://github.com/opensensorhub/osh-js/blob/8a959d440ea989dd32c6894229f051df8f5c87da/source/core/parsers/sweapi/observations/SweApiResult.datastream.parser.js
[WS]: https://github.com/opensensorhub/osh-js/blob/8a959d440ea989dd32c6894229f051df8f5c87da/source/core/connector/WebSocketConnector.js#L61-L75
[PlaceholderTest]: https://github.com/opensensorhub/osh-js/blob/8a959d440ea989dd32c6894229f051df8f5c87da/demos/3dr-solo-uav/3dr-solo-uav-react/src/App.test.js
[ParserDiff]: https://github.com/opensensorhub/osh-js/compare/8a959d440ea989dd32c6894229f051df8f5c87da...d3aa99cef7de0b4e6f928b1a77f07b898f6747e9
[Intro]: https://github.com/opensensorhub/osh-js/commit/49f434dbe2fa1680ec51a088b137c6f79cdefd64
[Replay]: https://github.com/opensensorhub/osh-js/commit/da91cde02
[Cred]: https://github.com/opensensorhub/osh-js/commit/738c2f68f
[Foi]: https://github.com/opensensorhub/osh-js/commit/9e91d61c5
[FoiRoot]: https://github.com/opensensorhub/osh-js/commit/f434cc9098bbac34f95fd073398d74f21b943a95
[ObsFmt1]: https://github.com/opensensorhub/osh-js/commit/24435f6c7
[ObsFmt2]: https://github.com/opensensorhub/osh-js/commit/8773852d7
[SWAPI]: https://github.com/opensensorhub/sensorweb-api/blob/3f5b1e9f2dd3912852513d26a1b93f374e8a6654/sensorweb-api.yaml
[Part1]: https://docs.ogc.org/is/23-001/23-001.html
[Part2]: https://docs.ogc.org/is/23-002/23-002.html
[Features]: https://docs.ogc.org/is/17-069r4/17-069r4.html
[Schemas]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api
[ServerPin]: https://github.com/DGIWG-P507/glaux-server/tree/27955c1b9260cd811ad6bc08f85feab43ad65028
[SysRead]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/system-read.md
[SysReadTests]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/system-read-tests.md
[Auth]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/authentication.md#http-and-caller-boundary
[RunCfg]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/runtime-configuration.md
[HttpB]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/http-boundary.md
[R009]: idr-srv-009-landing-page-api-definition-and-conformance-declaration-behavior-report.md
[R010]: idr-srv-010-collections-resources-links-and-navigation-behavior-report.md
[R011]: idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md
[R012]: idr-srv-012-content-negotiation-media-types-and-encoding-selection-report.md
[R013]: idr-srv-013-error-model-http-status-codes-and-failure-semantics-report.md
[R014A]: idr-srv-014a-osh-csapi-server-implementation-study-report.md
[R056]: idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md
[Register]: ../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md
[Goal]: ../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md
[Guide]: ../../../../../Plans/glaux-server/glaux-server-implementation-guide.md
[G44]: ../../../../../Plans/glaux-server/glaux-server-implementation-guide.md#44-datastreams-observations-querying-and-spatial-behavior
[G48]: ../../../../../Plans/glaux-server/glaux-server-implementation-guide.md#48-publication-live-delivery-and-experimental-part-3
[G410]: ../../../../../Plans/glaux-server/glaux-server-implementation-guide.md#410-authentication-authorization-trust-and-audit
[G62]: ../../../../../Plans/glaux-server/glaux-server-implementation-guide.md#62-endpoint-and-representation-baseline
[G83]: ../../../../../Plans/glaux-server/glaux-server-implementation-guide.md#83-performance-and-regression-expectations
[G13]: ../../../../../Plans/glaux-server/glaux-server-implementation-guide.md#13-appendix-standards-interpretations-and-project-choices
[Roadmap]: ../../../../../Plans/glaux-server/glaux-server-roadmap.md
[Charter]: ../../../../../Plans/glaux-server/Implementation-Reviews/Phase-1/README.md

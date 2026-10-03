# Section 065: cs-client-ts Client Library Study - Research Report

**Topic ID:** IDR-SRV-065<br>
**Report Status:** Final — accepted September 30, 2026<br>
**Research Plan:** [IDR-SRV-065 plan](../IDR%20Plans/idr-srv-065-cs-client-ts-client-library-study.md)<br>
**Overall Research Plan:** [Controlling overall IDR plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Q1–Q6. Coverage, request construction, models, tolerance, fixture origins and independence are established from source and from the published package. Neither the library nor its tests were executed.<br>
**Methodology Used:**
- Source inspection of the repository at `a632798` and `4724b0e`, and of the four published npm tarballs.
- A programmatic comparison of all 98 test fixtures with the official OGC example files.
- Targeted comparison with CS-GO for the two stated departures from the standard.
- Mapping to the published CSAPI text, the pinned schemas, the draft Part 3 text and Glaux's implemented Phase 1 behaviour.

**Research Time:** About 25 minutes of research and drafting, 22:30–22:55 UTC September 29, 2026, in one AI-assisted iteration, before separate review. This is not a human-hours estimate. Neither the library nor its tests were executed; only read-only downloads and a local Node comparison script were run. Nothing was installed.<br>
**Primary Sources:**
- [cs-client-ts at `a632798`][LibPin] and [`4724b0e`][LibNpm]
- [`cs-api-client` on npm][Npm] (0.1.0–0.1.3)
- [CSAPI Part 1][Part1], [Part 2][Part2]; CSAPI examples and schemas at [`8e03b23`][Schemas]; draft Part 3 at [`6f529a1`][Part3]

**Supporting Resources:**
- [IDR-SRV-014B][R014B], [IDR-SRV-062][R062] (CS-GO)
- [IDR-SRV-063][R063], [IDR-SRV-064][R064] (accepted client studies)
- [IDR-SRV-056][R056]
- [Guide v1.21][Guide], [Roadmap v1.40][Roadmap]
- Glaux Server `main` at [`27955c1`][ServerPin]

**Document Purpose:** Independent-as-far-as-possible client evidence for the Phase 1 review and later gates, a recommendation on IDR-SRV-056, and the library baseline for IDR-SRV-066 (Aleph). It is not a requirements document, and it does not authorise copying client code.<br>
**Author(s):** Glaux research workflow, AI-assisted<br>
**Accepted By:** Glaux Project Lead, by merging [PR #110](https://github.com/DGIWG-P507/glaux/pull/110) as `2105f60d7992335971c5fc8be73132160db92067`, under the September 29 report-merge acceptance decision<br>
**Acceptance Date:** September 30, 2026 (recorded October 3, 2026)<br>
**Date:** September 29, 2026<br>
**Last Updated:** October 3, 2026 (acceptance record only; findings unchanged)

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

**cs-client-ts is by far the most standards-aligned client studied so far.** Unlike the OSH Viewer and OSCAR, it follows the published standard throughout:
- **Routes and formats:** it uses only published routes, chooses formats with the `Accept` header, and uses the published query names (`datetime`, `recursive`, `dataStream`, `phenomenonTime`, `resultTime`, `obsFormat`).
- **Paging:** it follows the server's opaque `next` links rather than computing offsets.
- **Errors:** it reads Problem Details errors.
- **Sign-in:** it supports Bearer and OAuth tokens.
- **New resources:** it takes a created resource's id from the `Location` header.
- **Live events:** its draft Part 3 module uses CloudEvents with the same `org.ogc.api.consys.*` event types that Glaux's Guide selects.

[Findings §4.2–4.4](#42-q2--coverage-and-requests).

**From reading its code, it should work with Glaux's Phase 1 today, with one setting (not yet run).**
- It can create a System and read it back, provided the caller asks for GeoJSON (`format: "geojson"`) and supplies a Bearer token.
- Its minimal GeoJSON create body is exactly Glaux's minimal request.
- Glaux's `201`, `Location` header and GeoJSON response should all satisfy the library's checks (inferred from code).
- Its default is SensorML, which Glaux's Phase 1 answers with `415` (on create) or `406` (on read). Glaux plans SensorML in Roadmap 2.2.3 and 2.3.1.

**Independence is limited, but in an unexpected way.**
- 94 of its 98 test fixtures are exact copies of OGC's official examples, and the other 4 are adapted from them. So its expected values come from the standard's own examples, not from CS-GO.
- Its two self-described "breaks from standard" both *loosen* the schemas:
  - an observation may carry both `result` and `result@link`;
  - names become optional free text, both for SensorML inputs and outputs and for SWE Common data fields and components, which appear in datastream and control-stream schemas.
- The first matches CS-GO's own rule exactly, so the two share that reading against the published schema. Glaux's plan follows the schema (Roadmap 3.2.3).

**Recommendation:**
- Treat the library as the strongest candidate so far for IDR-SRV-056: a pinned package, run in an approved isolated environment, with its shared-author limits stated.
- Use its request expectations as review checks.
- No Guide or Roadmap change is proposed.
- For IDR-SRV-066: Aleph uses the published `0.1.3` package, whose code matches commit `a632798`.

## 2. Scope and Plan Alignment

This is **IDR-SRV-065**, the third client study, run after IDR-SRV-064 was accepted by merging PR #109.

| Plan question | Coverage status | Evidence location |
|---|---|---|
| Q1 — Identity, provenance, independence | Complete; package-to-commit match confirmed; fixture origins classified programmatically | §4.1, §12.3 |
| Q2 — Coverage and requests | Complete for the public API surface and the pub/sub module | §4.2 |
| Q3 — Models and tolerance | Complete for the representations relevant to Glaux's planned scope | §4.3 |
| Q4 — Standards alignment | Complete for every material assumption | §4.4 |
| Q5 — Tests and fixtures | Complete from source; not executed | §4.5 |
| Q6 — Transfer to Glaux | Complete, including the IDR-SRV-066 handoff | §4.6, §§5–7 |

**Out of scope:**
- running the tests (that would need package installation);
- contacting the author;
- re-studying CS-GO beyond the two independence questions;
- copying code.

**License:** at retrieval, none of the three repositories (`cs-client-ts`, `Alephex`, `connected-systems-go`) had a license file per the GitHub API, and the npm package has no `license` field. Any reuse follows the terms that apply to it. This report describes behaviour in Glaux's own terms.

**Earlier coverage reconciled:**
- [IDR-SRV-014B][R014B] and [IDR-SRV-062][R062] studied the same author's server. This study only checks where library and server share an interpretation.
- IDR-SRV-063 and 064 provide the contrast with the OSH client family.

No accepted report is contradicted.

## 3. Evidence Base

### 3.1 Primary Sources Reviewed

All access dates are **2026-09-29 (UTC)**.

| Source | Version | Authority class | Availability / limitations |
|---|---|---|---|
| [cs-client-ts][Lib] | `main` at [`a632798`][LibPin] (September 14–15, 2026) and [`4724b0e`][LibNpm]; 8 commits, 222 files | Observed client library | Full source read on the API, HTTP, codec and pub/sub paths |
| `cs-api-client` npm tarballs | 0.1.0 (`58858f7`), 0.1.1 (`5a10f5d`), 0.1.2 and 0.1.3 (both recorded as `4724b0e`); all published by one npm account | Published artifact | 0.1.3 integrity `sha512-aN54cvRElSuQ/…` matches Aleph's lockfile |
| Official CSAPI examples and schemas | [`8e03b23`][Schemas]; 190 example JSON files downloaded | Normative schemas; the examples are informative | Used for fixture-origin comparison |
| Draft Part 3 | [`6f529a1`][Part3] (Glaux's pin) | Unapproved draft | Event-type table and example files |
| [CS-GO][CsGo] | `b1fd2e0` (IDR-SRV-062 pin, also the current head) | Same author's server | Checked only for observation-result validation |

**History:**
- 8 commits, all by one author, from July 5 to September 14, 2026 (one branch, no tags).
- No pull requests or issues.
- The history appears squashed. Process, test-first sequencing and evolution cannot be established from it.

**Package-to-commit match:**
- The compiled 0.1.3 package contains the `ComponentNameSchema` changes introduced in `a632798` (5 compiled files). The 0.1.2 package contains none of them.
- So 0.1.3 was published from work later committed as `a632798`, while npm records `4724b0e`, whose `package.json` says 0.1.1.
- `4724b0e` is the 0.1.2-era baseline. `a632798` differs from it only by its own changes: 9 files, covering the version number, optional SensorML/SWE names, a compare default and tests. `4724b0e` itself removed a deprecated lower-case `datastream` alias for the observation `dataStream` query and relaxed the observation result rule.

### 3.2 Supporting Sources Reviewed

| Source | Role |
|---|---|
| Glaux Server [`system-create.md`][SysCreate], [`system-read.md`][SysRead], [`authentication.md`][Auth] at `27955c1` | Implemented Phase 1 behaviour |
| [Guide v1.21][Guide] §§4.1.1, 4.3, 4.4, 4.8, 6.2–6.4, 13; [Roadmap v1.40][Roadmap] | Dispositions |
| [Upstream-history register][Register] | Checked for observation result forms, SensorML names and Part 3 event tokens; nothing new implicated |

### 3.3 Evidence Quality Notes

- **Authority order:** published CSAPI text and pinned schemas; then the draft Part 3 (experimental); then Glaux's Guide; then library code and CS-GO. The official examples are informative, even though the library treats them as its test oracle.
- **No execution.** The tests were read, not run. Runtime claims are inferred from code paths.
- **Authorship:**
  - The project lead reports a senior developer working with AI assistance.
  - The public record shows one author and one npm account.
  - This study does not infer which parts were AI-assisted. No assistant-guidance files are present; their absence is not evidence either way.
- **Personal data:** the author's email address is not reproduced.

## 4. Findings by Research Question

### 4.1 Q1 — Identity, provenance and independence

**Targeted versions:**
- The README and `package.json` claim Parts 1 and 2 plus "the Part 3 Publish/Subscribe draft".
- The Part 1/2 surface uses published routes and names throughout (§4.2). The pub/sub module follows the draft Part 3 at Glaux's pin (§4.2).

**Fixture origins** (§12.3; a programmatic comparison of normalised JSON against the 190 official example files):

| Origin | Count | Examples |
|---|---|---|
| Exact copy of an official Part 1/2, SensorML or SWE Common example | 91 | All 32 SWE Common fixtures; GeoJSON Systems (`thermometer-sensor`, `uav-platform`); Part 2 datastreams, observations, schemas, commands and statuses |
| Exact copy of a draft Part 3 example | 3 | The three pub/sub event fixtures |
| Adapted from official examples | 4 | Two SensorML deployments that are the official `spec/deployment.json` and `spec/deployment_sd001.json` with the elided `"contacts": [ ... ]` filled in (compared by hand); two `compare/` fixtures composed from several official SensorML examples (ThermoPro TP60S content; about 78–79% of their long strings appear in the official set, and no single file covers more than about 30%) |
| CS-GO-derived, or hand-written without an official source | 0 found | — |

**Interpretation:**
- The tests' expected inputs come from OGC, not from the author's server.
- That makes the parsing tests *independent of CS-GO*. It also means they test the library against the standard's examples, not against any server's behaviour.

**Shared readings with CS-GO (history cases):**

| Commit | Change | Published source | CS-GO | Classification |
|---|---|---|---|---|
| [`4724b0e`][LibNpm] "Broke from standard just a little bit…" (commit body: "This makes more sense to me and will be brought up") | An observation may carry **both** `result` and `result@link` (was exactly one) | Pinned `observation.json` has `oneOf` `result` / `result@link` | [`observation_json.go` L144–149][CsGoObs] also requires only "either", so it accepts both | **Shared tolerant reading against the schema.** Agreement between the two is not independent confirmation |
| [`a632798`][LibPin] "…break from standard" | SensorML input/output/parameter names **and** SWE Common aggregate slot names (DataRecord fields, Vector coordinates, DataArray/Matrix element type, DataChoice items) become optional and may be display labels ("optionally named by an interoperable CS server"); a test adds a DataRecord field named `"air temperature"` | SWE `SoftNamedProperty` requires `name` as a `NameToken` (`^[A-Za-z][A-Za-z0-9_\-]*$`) | Not checked | **Tolerant.** The server that motivated it is not identified in the public record |

Neither departure is documented in the README.

### 4.2 Q2 — Coverage and requests

**Surface:** 11 resource endpoint groups. [`src/api`][LibApi]
- **Part 1:** Systems (including history, subsystems, deployments and sampling features), Procedures, Deployments, Sampling Features, Properties and Collections.
- **Part 2:** DataStreams, Observations, ControlStreams, Commands (status and result) and System Events.
- **Operations:** read, list, create, create-many, replace, delete (with `cascade`) and nested creation.

**Request construction** ([`http-client.ts`][LibHttp], [`query.ts`][LibQuery], [`base-endpoint.ts`][LibBase]):
- **Paths:** fixed templates under a configured `baseUrl`. There is no landing-page, conformance or API-description discovery.
- **Format:** the `Accept` header only, never an `f` or `format` parameter.
  - GeoJSON is `application/geo+json` and SensorML is `application/sml+json`.
  - The default for Systems, Procedures and Deployments is SensorML (`DEFAULT_FORMAT = "sml"`).
  - Part 2 resources use `application/json`.
- **Query values:**
  - encoded with `URLSearchParams`;
  - arrays comma-joined;
  - intervals written as `start/end` with `..` for open ends;
  - `Date` values sent as ISO 8601;
  - `bbox` comma-joined.
- **Names:**
  - Systems: `datetime`, `recursive`, `bbox`, `q` and related filters.
  - Observations: `id`, `dataStream`, `phenomenonTime`, `resultTime`, `system`, `foi`, `observedProperty`, `limit`.
  - Schema selectors: `obsFormat` (required) and `cmdFormat`.
- **Paging:** `fetchPage` reads `items` or `features`, as appropriate to the format, and `links`, `numberMatched` and `numberReturned`. It follows the `next` link's `href` as given, dropping its own query, and stops when there is none. [`http-client.ts` L354–372][LibHttp]
- **Writes:**
  - `create` needs a `Location` header and returns its last path segment. [L375–394][LibHttp]
  - `createCommand` returns the new id and ignores any status body returned with it.
- **Authentication:** Basic, static or dynamic Bearer, or OAuth 2 with pre-expiry refresh and one retry after `401`. There are also request, response and error hooks. [L10–46, L159–221][LibHttp]
- **Pub/sub** (optional `./mqtt` export; [`src/pubsub`][LibPubsub], [`mqtt.ts`][LibMqtt]):
  - It is transport-neutral. Callers name channels; the library adds only an optional `topicPrefix`.
  - It decodes CloudEvents 1.0 Resource Events and Batch Resource Events, with type `org.ogc.api.consys.<resource>.<create|update|delete>` ([`events.ts`][LibEvents]).
  - It checks each message's content type.

### 4.3 Q3 — Response models and tolerance

- **Validation:**
  - Responses are validated with Zod unless `validateResponses: false`. A mismatch throws `ValidationError`.
  - Models are "loose" objects, so unknown members pass through. This tolerates extensions and additions, not missing required members.
- **GeoJSON System** ([`system-feature.ts`][LibSysGeo], [`feature.ts`][LibFeature]):
  - `type: "Feature"`;
  - optional `id` and nullable `geometry`;
  - required `properties.featureType`, `uid` and `name` (non-empty);
  - optional `description`, `assetType` (enumerated), `validTime` and `systemKind@link`;
  - optional `links`, whose items need only `href`.
- **Link relations:** `parentSystem` matches either the bare token or `ogc-rel:parentSystem` ([`relation-links.ts`][LibRel]). The full OGC relation URI form is not matched.
- **Observations:** at least one of `result` or `result@link` (§4.1).
- **Errors:**
  - Any non-2xx response throws `HttpError`. A `404` throws `NotFoundError`.
  - A `problem+json` body is parsed into the error.
  - `201` or `204` with an empty body is accepted.
- **Silent behaviours:**
  - **Lossy common model:** fields with no representation in the target encoding are dropped on write, as the README documents.
  - **Inline command status discarded:** `createCommand` keeps only the id.
  - **Query strings:** a caller-supplied `datetime` string is passed through unchecked. Four test files use `datetime=latest`. Part 2 defines `latest` only for `resultTime`, so on `datetime` it is neither RFC 3339 nor a defined special value.

### 4.4 Q4 — Standards alignment

Classification key: **C** conforming; **S** stricter than required; **T** tolerant; **G** shared with CS-GO against the schema; **X** draft/experimental; **N** non-conforming.

| Assumption | Published source | Class | Glaux position |
|---|---|---|---|
| Canonical and nested routes (`/systems`, `/systems/{id}/subsystems`, `/datastreams/{id}/schema`, `/controlstreams/{id}/commands`, `/commands/{id}/status`, `/systemEvents`, `/collections/{id}/items`) | Part 1 Req 6, 9; Part 2 Req 11, 71; Part 2 Req 32 spells the status path `/command/{cmdId}/status` (singular) | C (plural `/commands` as in Guide §13) | Same routes; Guide §13 |
| `Accept` negotiation; no `f`/`format` | Features/Common negotiation; the Part 2 `f` parameter file is unreferenced (IDR-SRV-064) | C | Guide §6.2 |
| Default SensorML for Systems | Part 1 Req 89/90 (`application/sml+json`) | C | Phase 1 offers GeoJSON only (`406` on read, `415` on create); SensorML is planned in Roadmap 2.2.3 and 2.3.1 |
| GeoJSON lists use `features`, SensorML lists use `items` | Pinned `systemCollection.json` (both encodings) | C | Roadmap 2.3.9 |
| Follow `next`; read `numberMatched`/`numberReturned` | Features Rec 17–19 | C | Guide §6.3; Roadmap 2.5.10, 3.3.6 |
| `datetime`, `recursive`, `dataStream`, `phenomenonTime`, `resultTime`, `foi`, `observedProperty` | Part 1 Req 3, 10–12; Part 2 Req 49–51; official OpenAPI parameter files | C | Guide §6.3 |
| `obsFormat` required on schema | Part 2 Req 11 | C | Roadmap 3.1.6 |
| `201` + `Location`, id taken from the last segment | Part 1/2 create classes | C | Implemented in Phase 1 ([system-create][SysCreate]); commands per Guide §6.4 |
| Problem Details errors | Features/HTTP | C | Glaux emits Problem Details |
| Bearer and OAuth | Deployment matter | C with Glaux | Glaux accepts Bearer only (Guide §4.10) |
| `result` and `result@link` both allowed | Pinned `observation.json` `oneOf` | G/T | Glaux follows the schema: mixed result forms fail (Roadmap 3.2.3) |
| SensorML IO and SWE component/field `name` optional or free text | SWE `SoftNamedProperty`/`NameToken` (used by the pinned `DataRecord`, `Vector`, `DataChoice` and `DataArray` schemas) | T (N if written) | Glaux emits and requires valid names (Roadmap 2.2.3 for SensorML; 3.1.1, 3.1.5 for datastream schemas; 5.1.1 for control-stream schemas) |
| `parentSystem` matched by bare or `ogc-rel:` form only | Guide §4.1.1 (link-relation spelling) | T/limited | Glaux emits `ogc-rel:parentSystem`, so it matches |
| CloudEvents `org.ogc.api.consys.<resource>.<op>`, including `subsystem` | Draft Part 3 `/req/resource-events/event-types`; the draft's token list includes `subsystem` | X (matches the draft) | Guide §4.8 uses the same scheme, but reports subsystems as `system`. The library accepts both |
| Caller-chosen MQTT channels | Draft Part 3 MQTT class | X | Compatible with Guide §4.8 topics if configured; Roadmap 6.3.2 |

**Interpretation:**
- Almost everything is C, in contrast to IDR-SRV-063/064.
- The two material exceptions are *tolerances* on input. Both matter to Glaux only if the library *writes* such content, which the gate checks in §4.6 cover.

### 4.5 Q5 — Tests and fixtures

- **Size:** 30 test files with 203 `it`/`test` cases plus 26 `it.each` tables (one case per fixture), run with Vitest in Node. There is no CI workflow.
- **Layers:**
  - request-shape tests with a stand-in `fetch` (8 API files and the HTTP client);
  - model-parse tests over fixtures;
  - codec and compare tests;
  - path-reference tests;
  - pub/sub tests with a fake transport.
- **What they would catch:**
  - [`systems-api.test.ts`][TSys] asserts the exact `Accept` header, the exact URL including `datetime`, `cascade` on delete, and two-page `next` traversal. A wrong header, a wrong parameter name or ignoring `next` would fail.
  - [`http-client.test.ts`][THttp] (24 cases) covers:
    - query formatting;
    - Bearer, OAuth refresh and retry after `401`;
    - `ValidationError`, `NotFoundError` and Problem Details;
    - `create` with and without `Location`;
    - counts across pages;
    - three-page `next` traversal.
  - Roughly 35–48 negative checks, depending on how they are counted, assert thrown errors, rejected promises or failed parses. For example, an observation with neither result form is rejected.
- **Limits:**
  - The fixtures are the standard's own examples. The tests show the library accepts them, not that a server conforms.
  - Some checks are weak (for example `uniqueId` "truthy").
  - The positive test for both `result` and `result@link` encodes the shared reading from §4.1.
  - Any test could pass with a matching mistake in both a model and an adapted fixture; the 4 adapted fixtures carry that risk.

### 4.6 Q6 — Transfer to Glaux

**Phase 1 comparison (server `27955c1`):**

| Library behaviour | Glaux today | Result |
|---|---|---|
| `systems.create(input, { format: "geojson" })` with only `uniqueId`, `label` and `featureType` sends `{"type":"Feature","geometry":null,"properties":{"featureType","uid","name"}}` as `application/geo+json` | Exactly the minimal accepted request; `201`, empty body, `Location` | **Meets** |
| The same call with `description`, `assetType`, `validTime`, a position or links | Currently `422` (an implementation limit, not a standards claim) | **Conflict for now**; Roadmap 2.3.1 |
| Default `create` or `get` (SensorML) | `415` / `406` | **Conflict for now**; Roadmap 2.2.3, 2.3.1 |
| `systems.get(id, { format: "geojson" })` validates `type`, `id`, `geometry: null` and `properties.uid/name/featureType`, and accepts the `self` link (links need only `href`) | Glaux's System matches the library schema | **Meets** (inferred from code) |
| `create` needs `Location` and takes the last segment | `Location: {root}/systems/{id}` | **Meets** |
| Bearer or OAuth | Bearer only | **Meets** when configured |
| `systems.list()` | `405` | **Conflict for now**; Roadmap 2.3.1, 2.3.9, 2.5.10 |

**Checks for later gates:**

| Gate area (Roadmap) | Check to include |
|---|---|
| 2.2.3, 2.3.1 SensorML | SensorML read and write work for the library's default format. IO components without a valid `name` are rejected on write |
| 2.3.9, 2.5.10 Lists | `features`/`items` by format; `next` works when followed as an opaque `href` with no client-added query. The library treats any `href` not starting with `http` as relative to its base URL, so an absolute `next` link is safest |
| 3.1.1, 3.1.5, 5.1.1 Schemas | A datastream or control-stream schema whose record fields lack a valid `NameToken` name is rejected on write |
| 3.2.3 Observations | A write with both `result` and `result@link` gives the documented error. Reads never emit both |
| 5.2.3, 5.3.x Commands | The `201` + `Location` + status body still lets a client that keeps only the id retrieve the status at `/commands/{id}/status` |
| 6.1.x, 6.3.x Events | Resource Events validate against the library's CloudEvents model; a subsystem change is reported as `system`, which the library accepts |

**IDR-SRV-056 recommendation:**
- Record `cs-api-client` `0.1.3` (integrity above) as a **candidate programmatic client target**, run as a pinned package in an approved isolated environment, once Glaux lists and SensorML exist.
- State its limits:
  - the same author as CS-GO;
  - a shared tolerant reading on observation results;
  - tests that validate against OGC examples, not a server.
- It complements the OS4CSAPI client already in IDR-SRV-056 and does not replace it.

**Handoff to IDR-SRV-066 (Aleph):**
- Aleph's lockfile pins `0.1.3`, whose code matches `a632798`, not npm's recorded `4724b0e`.
- The pub/sub module and its CloudEvents validation are part of that package.
- The library's default SensorML reads, and its `Accept` behaviour, are what Aleph inherits unless it overrides them.

## 5. Decision Analysis

| Option | Benefits | Costs / risks | Standards impact | Recommendation |
|---|---|---|---|---|
| A. Keep Glaux as planned; use this report as review evidence | Glaux already meets the library's Phase 1 GeoJSON path | Default-format calls fail until SensorML exists | Preserves conformance | **Keep** |
| B. Accept both `result` and `result@link` to match the library and CS-GO | Fewer rejections from this author's tools | Contradicts the pinned schema and Roadmap 3.2.3 | Non-conforming | **Reject** |
| C. Record the library as an IDR-SRV-056 candidate | A strongly standards-aligned, independently written (from OSH) client | Shared author with a studied server; needs an approved environment | None | **Conditional**: a later 056 discussion |

## 6. Key Recommendations

1. **Use this report as Phase 1 review evidence.** An independent, standards-aligned client should be able to create and read a Glaux System using GeoJSON and Bearer authentication, with no adaptation (inferred from code; not executed).
   - Priority: High.
   - Preconditions: project-lead acceptance (report PR merge).
2. **Carry the §4.6 gate checks into their reviews**, especially the SensorML default format, rejecting mixed result forms on write, and `next` traversal.
   - These are review prompts, not new requirements.
   - Priority: Medium.
3. **Record `cs-api-client` `0.1.3` as an IDR-SRV-056 candidate**, with its limits stated.
   - Priority: Low.
   - Preconditions: a separately authorised 056 discussion.
4. **No Guide or Roadmap change is proposed.**

## 7. Implementation Implications and Estimates

### 7.1 Implications

- **Architecture and implementation:** none. The library confirms Glaux's negotiation, paging, create-response and event-type choices.
- **Testing:** later gates gain concrete client cases:
  - the SensorML default format;
  - a write with both result forms;
  - an opaque `next` link;
  - CloudEvents validation, including subsystem-as-`system`.

### 7.2 Effort/Complexity Estimate

| Work item | Relative complexity | Estimate | Assumptions |
|---|---|---|---|
| Phase 1 review uses this evidence | Low | Within existing review steps | No re-execution |
| Later-gate checks | Low | Absorbed by gate reviews | Reuse planned tests |
| Future 056 qualification | Medium | One matrix iteration | Approved environment; package installation permitted there |

## 8. Risks, Constraints, and Open Questions

### 8.1 Risks and Constraints

- **Not executed.** Test outcomes are unverified.
- **Shared author.** Agreement with CS-GO is not independent confirmation (§4.1).
- **Squashed history.** Nothing is concluded about process or test-first practice.
- **A moving package.** A later npm version may change these findings. Aleph pins 0.1.3.

### 8.2 Open Questions

- **Which server motivated optional or free-text SensorML IO and SWE component names?** The public record does not say. It matters only if Glaux later receives such descriptions or schemas; it is for the SensorML and stream-schema owners (Roadmap 2.2.3, 3.1.1, 3.1.5, 5.1.1).
- **Should Glaux's documentation state that it emits `ogc-rel:` relation tokens rather than full URIs?** Clients like this one match only the short forms. This is for the link-relation owner (Guide §4.1.1); it is not a proposed change.

## 9. Validation Against Plan Success Criteria

| Topic plan success criterion | Status | Evidence |
|---|---|---|
| Q1–Q6 answered or limited | Met | §4 |
| Baselines pinned; 0.1.3 matched to `a632798`; `4724b0e` labelled; differences recorded | Met | §3.1 |
| Independence assessed; fixture origins classified; shallow history stated | Met | §§3.1, 4.1, 12.3 |
| Every material assumption traceable and classified | Met | §§4.2–4.4 |
| Tests explained; independence of expected values assessed; source inspection distinguished from execution | Met | §4.5 |
| License status recorded; recommendations in Glaux's own terms | Met | §2 |
| Phase 1 compared; later-gate checks listed; IDR-SRV-066 handoff recorded | Met | §4.6 |
| Each lesson has a Guide/Roadmap disposition | Met | §§4.4, 4.6, 6 |
| Relevant standards history consulted | Met: nothing new implicated; the Part 3 draft event table checked | §§3.2, 4.4 |
| Report follows the template | Met | This report |

## 10. Next Steps and Handoff

1. Accepted by the project lead's September 30, 2026 merge of PR #110; recorded October 3 under the decision of September 29. Acceptance does not adopt recommendations or change implementation scope.
2. The October 3, 2026 `proceed` authorises IDR-SRV-066 (Aleph), using §4.6's handoff. Owner: Glaux research workflow.
3. Recommendation 3 waits for a separately authorised IDR-SRV-056 discussion.

No Goal, Guide, Roadmap, synthesis, server code or issue was changed by this study.

## 11. References

- [cs-client-ts][Lib] at [`a632798`][LibPin] and [`4724b0e`][LibNpm]; [`cs-api-client` on npm][Npm]
- [CS-GO][CsGo] at `b1fd2e0`
- [CSAPI Part 1][Part1]; [CSAPI Part 2][Part2]; [CSAPI artifacts at `8e03b23`][Schemas]; [draft Part 3 at `6f529a1`][Part3]
- [IDR-SRV-014B][R014B]; [IDR-SRV-062][R062]; [IDR-SRV-063][R063]; [IDR-SRV-064][R064]; [IDR-SRV-056][R056]; [upstream-history register][Register]
- [Guide v1.21][Guide]; [Roadmap v1.40][Roadmap]
- Glaux Server at [`27955c1`][ServerPin]: [system-create][SysCreate], [system-read][SysRead], [authentication][Auth]

## 12. Appendices

### 12.1 Reproduction record

Commands used, all read-only; nothing installed:

```sh
git clone https://github.com/SomethingCreativeStudios/cs-client-ts.git            # HEAD a632798
curl -O https://registry.npmjs.org/cs-api-client/-/cs-api-client-0.1.3.tgz         # sha512 aN54cvRElSuQ/...
grep -rl ComponentNameSchema npm-0.1.3/package/dist                                  # 5 files (0 in 0.1.2)
# 190 official example JSON files downloaded from ogcapi-connected-systems@8e03b23, and
# 3 draft Part 3 examples from @6f529a1; 19 official files that are not valid JSON (`...` elisions
# or bare property snippets) were excluded from exact matching; the two relevant to fixtures
# (spec/deployment.json, deployment_sd001.json) were compared by hand;
# every fixture's key-sorted JSON compared with
# each example, then string-overlap scoring for non-identical files (Node script, no packages).
```

### 12.2 Selected source anchors

| Item | Anchor |
|---|---|
| HTTP client, auth, errors, paging, create | [`http-client.ts`][LibHttp] |
| Query encoding | [`query.ts`][LibQuery] |
| Format selection and dual-encoding endpoints | [`base-endpoint.ts`][LibBase]; [`systems.ts`][LibSystems] |
| GeoJSON System model | [`system-feature.ts`][LibSysGeo]; [`feature.ts`][LibFeature] |
| Relation matching | [`relation-links.ts`][LibRel] |
| CloudEvents models | [`events.ts`][LibEvents] |

### 12.3 Fixture-origin summary

- **91 exact matches** with `ogcapi-connected-systems@8e03b23` examples:
  - Part 1 GeoJSON (13): 2 Systems, 1 Procedure, 1 Deployment, 9 Sampling Features;
  - Part 2 (31): 3 datastreams, 6 observations, 7 observation schemas, 3 command schemas, 2 commands, 5 command statuses, 3 command results, 1 control stream, 1 System Event;
  - SensorML (15);
  - SWE Common (32).
- **3 exact matches** with the draft Part 3 event examples at `6f529a1`.
- **4 adapted:** `sensorml/deployment/deployment.json` and `deployment_sd001.json` are the official `sensorml/schemas/json/examples/spec/deployment.json` and `deployment_sd001.json` with the elided `contacts` filled in (compared by hand, because those official files are not valid JSON); `compare/procedure_tp60s.json` and `compare/system_tp60s_instance.json` are composed from several official SensorML examples.
- **0** with no official source.

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
- [x] Plan-owner acceptance and acceptance date are recorded before the topic is treated as complete downstream
- [x] Next steps are assigned

[Lib]: https://github.com/SomethingCreativeStudios/cs-client-ts
[LibPin]: https://github.com/SomethingCreativeStudios/cs-client-ts/tree/a6327989f5cec3c36422a121a2fe676e8a6bbe6d
[LibNpm]: https://github.com/SomethingCreativeStudios/cs-client-ts/tree/4724b0e075f2d491885caa3824fcf0da23a464d1
[LibApi]: https://github.com/SomethingCreativeStudios/cs-client-ts/tree/a6327989f5cec3c36422a121a2fe676e8a6bbe6d/src/api
[LibHttp]: https://github.com/SomethingCreativeStudios/cs-client-ts/blob/a6327989f5cec3c36422a121a2fe676e8a6bbe6d/src/http/http-client.ts
[LibQuery]: https://github.com/SomethingCreativeStudios/cs-client-ts/blob/a6327989f5cec3c36422a121a2fe676e8a6bbe6d/src/http/query.ts
[LibBase]: https://github.com/SomethingCreativeStudios/cs-client-ts/blob/a6327989f5cec3c36422a121a2fe676e8a6bbe6d/src/api/base-endpoint.ts
[LibSystems]: https://github.com/SomethingCreativeStudios/cs-client-ts/blob/a6327989f5cec3c36422a121a2fe676e8a6bbe6d/src/api/systems.ts
[LibSysGeo]: https://github.com/SomethingCreativeStudios/cs-client-ts/blob/a6327989f5cec3c36422a121a2fe676e8a6bbe6d/src/models/geojson/system-feature.ts
[LibFeature]: https://github.com/SomethingCreativeStudios/cs-client-ts/blob/a6327989f5cec3c36422a121a2fe676e8a6bbe6d/src/models/geojson/feature.ts
[LibRel]: https://github.com/SomethingCreativeStudios/cs-client-ts/blob/a6327989f5cec3c36422a121a2fe676e8a6bbe6d/src/codec/relation-links.ts
[LibEvents]: https://github.com/SomethingCreativeStudios/cs-client-ts/blob/a6327989f5cec3c36422a121a2fe676e8a6bbe6d/src/pubsub/events.ts
[LibPubsub]: https://github.com/SomethingCreativeStudios/cs-client-ts/tree/a6327989f5cec3c36422a121a2fe676e8a6bbe6d/src/pubsub
[LibMqtt]: https://github.com/SomethingCreativeStudios/cs-client-ts/blob/a6327989f5cec3c36422a121a2fe676e8a6bbe6d/src/mqtt.ts
[TSys]: https://github.com/SomethingCreativeStudios/cs-client-ts/blob/a6327989f5cec3c36422a121a2fe676e8a6bbe6d/test/api/systems-api.test.ts
[THttp]: https://github.com/SomethingCreativeStudios/cs-client-ts/blob/a6327989f5cec3c36422a121a2fe676e8a6bbe6d/test/http/http-client.test.ts
[Npm]: https://www.npmjs.com/package/cs-api-client
[CsGo]: https://github.com/SomethingCreativeStudios/connected-systems-go
[CsGoObs]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/formaters/json_formatters/observation_json.go#L144-L149
[Part1]: https://docs.ogc.org/is/23-001/23-001.html
[Part2]: https://docs.ogc.org/is/23-002/23-002.html
[Schemas]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api
[Part3]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/6f529a15bfa63259febc3620378d3e5a06305333/api/part3
[ServerPin]: https://github.com/DGIWG-P507/glaux-server/tree/27955c1b9260cd811ad6bc08f85feab43ad65028
[SysCreate]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/system-create.md
[SysRead]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/system-read.md
[Auth]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/authentication.md#http-and-caller-boundary
[R014B]: idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md
[R062]: idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md
[R063]: idr-srv-063-osh-viewer-and-osh-js-client-study-report.md
[R064]: idr-srv-064-oscar-viewer-client-study-report.md
[R056]: idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md
[Register]: ../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md
[Guide]: ../../../../../Plans/glaux-server/glaux-server-implementation-guide.md
[Roadmap]: ../../../../../Plans/glaux-server/glaux-server-roadmap.md

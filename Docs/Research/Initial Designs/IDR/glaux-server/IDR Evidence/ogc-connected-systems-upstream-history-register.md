# OGC API - Connected Systems Upstream Standards-History Evidence Register

**Version:** 1.0<br>
**Status:** Active supporting evidence<br>
**Initial screening completed:** August 1, 2026<br>
**Register owner:** Glaux Project Lead<br>
**Official repository:** https://github.com/opengeospatial/ogcapi-connected-systems<br>
**Published-source tag checked:** [`v1.0.0`](https://github.com/opengeospatial/ogcapi-connected-systems/releases/tag/v1.0.0), commit [`8e03b236`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Mutable `master` snapshot checked:** August 1, 2026, commit [`3fd86c73`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f)<br>
**Mutable `part3-working-draft` snapshot checked:** August 1, 2026, commit [`a1f1f03b`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/a1f1f03b71f5f645486b23ec8b5fae1f9ba334bc)

---

## 1. Purpose

This register preserves useful design history from the official OGC API - Connected Systems repository so that Glaux research does not have to rediscover it topic by topic. It records known artifact defects, decisions and rationale that explain the published result, post-publication maintenance, and questions that the standards project has not yet resolved.

The register is **not a standard, errata document, or substitute for the published specifications**. An issue title, comment, label, approval, closure, pull request, repository file, or GitHub release setting cannot by itself create or change a standards obligation. Each Glaux report must derive obligations from the applicable approved standard and use this history only to explain, test, reconcile, or monitor them.

This is a supporting evidence artifact, not an additional IDR topic or research report. Maintaining a bounded set of entries while executing an owning topic does not start another topic.

## 2. Evidence Classes

| Code | Meaning | Permitted use |
|---|---|---|
| **PB** | The issue or change traces an outcome present in the published `v1.0.0` source or approved standards package. | Use the published document or artifact for the actual rule. Use the history to explain provenance, scope, or intent. |
| **PCD** | Official post-publication development or recorded current direction, including a later merge, approved-but-unmerged change, or maintainer/SWG disposition. | Treat as compatibility, maintenance, or monitoring evidence. Do not silently apply it to the published 1.0 contract. |
| **UP** | Unresolved proposal, interpretation, defect, or discussion without an adopted and published disposition. Closed-without-rationale items can also fall here. | Record the ambiguity and make a deliberate Glaux decision when necessary; never present the proposal as required by the standard. |
| **Mixed** | More than one class applies, usually because a published baseline exists but residual or future work remains. | State separately what is published, what changed later, and what remains unresolved. |

`Open` and `Closed` below are only the states observed on August 1, 2026. A closed issue is not proof that its proposed change was implemented; issue [#64](https://github.com/opengeospatial/ogcapi-connected-systems/issues/64) is a concrete counterexample.

## 3. Controlling Baseline and Artifact Provenance

| Evidence | Snapshot finding | Glaux use |
|---|---|---|
| Published Parts 1 and 2 | [OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html) and [OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html) are the controlling CSAPI 1.0 specifications. | Derive and cite obligations here before consulting repository history. |
| Tagged source | Tag `v1.0.0` points to commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`. Its source artifacts are under [`api/`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api). | Pin reproducible source and compare modular schemas/OAS with the approved documents. |
| GitHub release | The release is named `v1.0.0-published`, was created July 16, 2025, published April 8, 2026, and was still marked **prerelease** on August 1, 2026. | Treat the prerelease flag as repository metadata, not as evidence that the approved OGC standards are drafts. |
| Part 1 bundled OAS 3.1 | [`ogcapi-connectedsystems-1.bundled.oas31.yaml`](https://github.com/opengeospatial/ogcapi-connected-systems/releases/download/v1.0.0/ogcapi-connectedsystems-1.bundled.oas31.yaml), SHA-256 `69da631d5d05f01716381cca7b7ee6311402f2752a8fd79a9b72b663539555aa`. | Inspect and mechanically compare with the tagged modular Part 1 OAS; record variant-specific defects. |
| Part 2 bundled OAS 3.1 | [`ogcapi-connectedsystems-2.bundled.oas31.yaml`](https://github.com/opengeospatial/ogcapi-connected-systems/releases/download/v1.0.0/ogcapi-connectedsystems-2.bundled.oas31.yaml), SHA-256 `86ed005f9e7cf176264d6deb72581a0b521a227cd7a198b6cb1bd32b39d83667`. | Inspect and mechanically compare with the tagged modular Part 2 OAS; record variant-specific defects. |
| Later `master` | Snapshot commit `3fd86c73` postdates `v1.0.0`; it contains the post-release example correction from PR #176. | Compare deliberately, but do not mistake later repository state for the published 1.0 baseline. |
| Part 3 and AsyncAPI | Part 2 assigns publish/subscribe binding to a future Part 3. A legacy AsyncAPI 2.6 support file is packaged with Part 2, while the active [`part3-working-draft` tree](https://github.com/opengeospatial/ogcapi-connected-systems/tree/a1f1f03b71f5f645486b23ec8b5fae1f9ba334bc/api/part3) pinned at commit `a1f1f03b` contains no AsyncAPI file. Part 3/AsyncAPI 3.0 work remains proposed development material. | Research these artifacts explicitly for design direction in IDR-SRV-035, with every resulting Glaux choice labelled as a non-standards-mandated extension or forward-compatibility design unless and until published. Reserve “non-conformance” for an actual unmet requirement. |

## 4. Screening Coverage and Selection Rules

The initial pass screened **all 141 official issues** then present (60 open, 81 closed) and reviewed **all 58 pull requests** (4 open, 51 merged, 3 closed unmerged) for linked changes, dispositions, release inclusion, or material rationale. GitHub uses one number sequence for issues and pull requests, which explains gaps in the issue numbering. Of the 141 issues, **118 were retained** and 23 were excluded as purely editorial, administrative, build/website work without artifact consequences, superseded discussion without added insight, or otherwise immaterial to server design.

There was no GitHub Discussions area in the official repository at the snapshot date. Comments and review threads attached to retained issues and pull requests were included when they supplied material context.

Retain an item when it can materially affect one or more of these areas:

- requirements, conformance classes, ATS coverage, or standards interpretation;
- API paths, methods, parameters, representations, media types, links, or resource lifecycles;
- JSON Schema, OpenAPI, AsyncAPI, examples, release artifacts, or publication provenance;
- SensorML, SWE Common, semantic mappings, tasking, streaming, DDIL, or interoperability design; or
- a known contradiction, defect, rejected alternative, or design rationale that prevents a plausible implementation mistake.

### Issues 1-80

| Issue | State | Material evidence and present meaning | Class | Owning IDR topic(s) |
|---|---|---|---|---|
| [#1](https://github.com/opengeospatial/ogcapi-connected-systems/issues/1) | Open | Planned Routes `Action` versus executed CSAPI `Command`; a Routes-to-CSAPI profile is proposed but no CSAPI change is merged. | UP | 037 |
| [#2](https://github.com/opengeospatial/ogcapi-connected-systems/issues/2) | Closed | Explains the split between generic Web links in `links[]` and named resource-model associations. No explicit linked implementation change. | PB | 017 |
| [#4](https://github.com/opengeospatial/ogcapi-connected-systems/issues/4) | Open | Unadopted proposals would clarify direct links versus templates, registered relations, and removal of undefined service fields. | UP | 017 |
| [#5](https://github.com/opengeospatial/ogcapi-connected-systems/issues/5) | Closed | Explains addition of a SWE Common Geometry component; the published artifact contains `Geometry.json`. | PB | 022 |
| [#6](https://github.com/opengeospatial/ogcapi-connected-systems/issues/6) | Open | Six scalar-list shorthands remain unadopted; published SWE Common uses scalar-element `DataArray`. | UP | 022 |
| [#8](https://github.com/opengeospatial/ogcapi-connected-systems/issues/8) | Closed | Explains why SensorML gained the published Deployment representation. | PB | 015 |
| [#9](https://github.com/opengeospatial/ogcapi-connected-systems/issues/9) | Open | Published Part 1 maps SOSA/OMS informatively; broader schema-governance and refactoring proposals still await an SWG decision. | Mixed | 024 |
| [#11](https://github.com/opengeospatial/ogcapi-connected-systems/issues/11) | Closed | Records deliberate addition of JSON schemas and normative JSON component/stream encoding rules. | PB | 022 |
| [#12](https://github.com/opengeospatial/ogcapi-connected-systems/issues/12) | Closed | Calls Part 1 OAS an “example”; authority must be determined from the published standard, not this issue. | PB | 014 |
| [#13](https://github.com/opengeospatial/ogcapi-connected-systems/issues/13) | Closed | Same provenance warning for the Part 2 OAS. | PB | 014 |
| [#14](https://github.com/opengeospatial/ogcapi-connected-systems/issues/14) | Open | The legacy AsyncAPI 2.6 support file is present in the v1.0.0 Part 2 artifact set. Moving/rescoping the work to Part 3 and AsyncAPI 3.0 remains an unresolved proposal, and the pinned Part 3 working tree contains no AsyncAPI file. | Mixed | 035 |
| [#15](https://github.com/opengeospatial/ogcapi-connected-systems/issues/15) | Open | Published Requirement 84 resolves range encoding as two scalar JSON-array members, despite stale open state. | PB | 022 |
| [#16](https://github.com/opengeospatial/ogcapi-connected-systems/issues/16) | Closed | SWG approved string forms such as `NaN`, `-Infinity`, and `+Infinity` for floating-point/Quantity-style numerical values; Count remains integer-valued under #17. | PB | 022 |
| [#17](https://github.com/opengeospatial/ogcapi-connected-systems/issues/17) | Closed | SWG rejected special strings for Count values to preserve integer storage. | PB | 022 |
| [#18](https://github.com/opengeospatial/ogcapi-connected-systems/issues/18) | Closed | SWG chose ECMA-262 regular expressions for JSON constraints. | PB | 023 |
| [#19](https://github.com/opengeospatial/ogcapi-connected-systems/issues/19) | Closed | Component-name paths were retained instead of JSON Pointer so process connections remain encoding-independent. | PB | 022 |
| [#20](https://github.com/opengeospatial/ogcapi-connected-systems/issues/20) | Closed | Early compact-only approach was rejected; final object/array switches came through #71/PR #118. | PB | 022 |
| [#21](https://github.com/opengeospatial/ogcapi-connected-systems/issues/21) | Closed | Media-type discussion has no clear disposition; use published tokens and do not infer registration from the thread. | Mixed | 012 |
| [#22](https://github.com/opengeospatial/ogcapi-connected-systems/issues/22) | Closed | Explains `describedby`, anchored HTTP links for binary data, and canonical-resource distinctions. | PB | 010 |
| [#23](https://github.com/opengeospatial/ogcapi-connected-systems/issues/23) | Open | Encoding conformance classes may overstate write support; per-method/media/schema advertisement remains unresolved. | UP | 008 |
| [#24](https://github.com/opengeospatial/ogcapi-connected-systems/issues/24) | Closed | Non-CRS84 geometry direction relies on OGC API Features Part 2; GeoJSON defaults remain CRS84/CRS84h. | PB | 026 |
| [#28](https://github.com/opengeospatial/ogcapi-connected-systems/issues/28) | Closed | Requirement/conformance URI structures were corrected; [PR #29](https://github.com/opengeospatial/ogcapi-connected-systems/pull/29) added relationship metadata. | PB | 008 |
| [#30](https://github.com/opengeospatial/ogcapi-connected-systems/issues/30) | Closed | Separate `DeployedSystem` resource was rejected; embedded system links and subdeployments became the published model. | PB | 017 |
| [#32](https://github.com/opengeospatial/ogcapi-connected-systems/issues/32) | Closed | `featureType` stayed inside GeoJSON `properties`; JSON-FG conformance was deferred. | PB | 015 |
| [#36](https://github.com/opengeospatial/ogcapi-connected-systems/issues/36) | Open | Published architecture uses SOSA/SSN as conceptual basis and maps OMS; open work tracks vocabulary evolution. | Mixed | 015 |
| [#39](https://github.com/opengeospatial/ogcapi-connected-systems/issues/39) | Closed | All SensorML 2.1 positioning choices were restored for compatibility by [commit `7e46046`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/7e46046dbc1aa39e268b6c9981b4a8b453b4d4cc). | PB | 021 |
| [#40](https://github.com/opengeospatial/ogcapi-connected-systems/issues/40) | Closed | Early mutual-exclusivity rationale for UOM code/URI was superseded before publication; #74/PR #94 records the published outcome that both may coexist when they identify the same unit. | Mixed | 024 |
| [#43](https://github.com/opengeospatial/ogcapi-connected-systems/issues/43) | Open | Published UML says complete quaternion while `Pose.json` uses intended `UnitQuaternion`; no correction is merged. | UP | 023 |
| [#45](https://github.com/opengeospatial/ogcapi-connected-systems/issues/45) | Open | `DataInterface` remained in the model but was lost from split JSON schema choices; restoration remains unapproved. | UP | 021 |
| [#46](https://github.com/opengeospatial/ogcapi-connected-systems/issues/46) | Closed | Records migration to SensorML/SWE Common 3.0 and removal of XML encoding from the main standards. | PB | 021, 022 |
| [#47](https://github.com/opengeospatial/ogcapi-connected-systems/issues/47) | Closed | [Commit `da6ce68`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/da6ce68f4838260d1a86bbf6a9502b7edb10833b) added the published link-relation table. | PB | 017 |
| [#48](https://github.com/opengeospatial/ogcapi-connected-systems/issues/48) | Open | Published artifacts use OAS 3.1 so SensorML schemas can be reused; the original downgrade request is superseded, while optional OAS 3.0 compatibility remains unresolved under #77. | Mixed | 014 |
| [#51](https://github.com/opengeospatial/ogcapi-connected-systems/issues/51) | Closed | Ultimate sampled-feature association maps to `sosa:hasSampledFeature`; [commit `aa52ecc`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/aa52ecc2aaad71fbba606c56354f37093be9aadb) fixed it. | PB | 017 |
| [#52](https://github.com/opengeospatial/ogcapi-connected-systems/issues/52) | Closed | A CSAPI System may be both SOSA Platform and System; a non-system Platform remains an ordinary Feature. [Commit `620d561`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/620d561709b6b58763dfc171e33501f97b6f1db5) added the explanation. | PB | 015 |
| [#55](https://github.com/opengeospatial/ogcapi-connected-systems/issues/55) | Closed | SWE Common encoding rules stayed in the main document because Part 2 normatively depends on them. | PB | 022 |
| [#57](https://github.com/opengeospatial/ogcapi-connected-systems/issues/57) | Closed | [Commit `0e79f9b`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/0e79f9be8e786e490bba97262f49f654bc21c30a) corrected remaining Part 1 OAS path-parameter defects. | PB | 014 |
| [#58](https://github.com/opengeospatial/ogcapi-connected-systems/issues/58) | Open | Transfer of control remains undefined: request/seizure, timeout, partition recovery, preemption, and rejoining controllers are open. | UP | 038 |
| [#59](https://github.com/opengeospatial/ogcapi-connected-systems/issues/59) | Open | Reservation/confirmation has no discoverable link, expiry, or lifecycle in Part 2 and would require optional new behavior. | UP | 037 |
| [#61](https://github.com/opengeospatial/ogcapi-connected-systems/issues/61) | Closed | System cascade deletes nested resources but removes the System link from an associated Deployment rather than deleting that Deployment; [PR #90](https://github.com/opengeospatial/ogcapi-connected-systems/pull/90). | PB | 030 |
| [#62](https://github.com/opengeospatial/ogcapi-connected-systems/issues/62) | Closed | Local deployed-system links use the System `uniqueID`; external links may use the external URL; [PR #92](https://github.com/opengeospatial/ogcapi-connected-systems/pull/92). | PB | 017 |
| [#64](https://github.com/opengeospatial/ogcapi-connected-systems/issues/64) | Closed | Proposed `earliest` change was not adopted: [PR #88](https://github.com/opengeospatial/ogcapi-connected-systems/pull/88) closed unmerged. The tagged modular OAS [`datetimeSchema.yaml`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/v1.0.0/api/part1/openapi/parameters/datetimeSchema.yaml) permits `now` and `latest`, not `earliest`; that is artifact evidence, not an independent normative rule. | Mixed | 011 |
| [#65](https://github.com/opengeospatial/ogcapi-connected-systems/issues/65) | Closed | [PR #93](https://github.com/opengeospatial/ogcapi-connected-systems/pull/93) makes server-generated `observedProperties`, `phenomenonTime`, `resultTime`, and `resultType` null when no Observations exist; `live` may be client-supplied or server-generated and may also be null. | PB | 034 |
| [#66](https://github.com/opengeospatial/ogcapi-connected-systems/issues/66) | Closed | Sampling-feature replace/delete exist only at canonical `/samplingFeatures/{id}`; nested creation remains; [PR #89](https://github.com/opengeospatial/ogcapi-connected-systems/pull/89). | PB | 016 |
| [#68](https://github.com/opengeospatial/ogcapi-connected-systems/issues/68) | Open | Part 3 direction shifted from EDR Pub/Sub toward common OGC API Pub/Sub and AsyncAPI 3.0; no final publication. | PCD | 035 |
| [#71](https://github.com/opengeospatial/ogcapi-connected-systems/issues/71) | Closed | [PR #118](https://github.com/opengeospatial/ogcapi-connected-systems/pull/118) added `recordsAsArrays`/`vectorsAsArrays`, but the published requirement reverses the booleans relative to names, schemas, UML, and examples and also contradicts itself by saying false means arrays while omitted/default-false means objects. | PB | 022, 023 |
| [#73](https://github.com/opengeospatial/ogcapi-connected-systems/issues/73) | Closed | SWE Common references rather than defines units; `UnitReference` is the partial JSON realization; [PR #140](https://github.com/opengeospatial/ogcapi-connected-systems/pull/140). | PB | 024 |
| [#74](https://github.com/opengeospatial/ogcapi-connected-systems/issues/74) | Closed | UCUM is recommended, not required; `code` and `href` may coexist but must identify the same unit; [PR #94](https://github.com/opengeospatial/ogcapi-connected-systems/pull/94). | PB | 024 |
| [#76](https://github.com/opengeospatial/ogcapi-connected-systems/issues/76) | Closed | [PR #116](https://github.com/opengeospatial/ogcapi-connected-systems/pull/116) replaced removed SSN `SystemKind` terms; exact URIs must remain pinned because vocabulary work continues. | Mixed | 024 |
| [#77](https://github.com/opengeospatial/ogcapi-connected-systems/issues/77) | Open | The published release contains maintained OAS 3.1 bundles only; an earlier SWG intent to supply both OAS 3.0 and 3.1 remains unfulfilled and unresolved. | Mixed | 014 |
| [#78](https://github.com/opengeospatial/ogcapi-connected-systems/issues/78) | Closed | Part 1 bundling exposed circular-reference/tool memory problems; [PR #139](https://github.com/opengeospatial/ogcapi-connected-systems/pull/139) ultimately automated the released OAS 3.1 bundle. | PB | 014 |
| [#79](https://github.com/opengeospatial/ogcapi-connected-systems/issues/79) | Closed | Same Part 2 bundle outcome; early generation also exposed missing/broken schemas. | PB | 014 |
| [#80](https://github.com/opengeospatial/ogcapi-connected-systems/issues/80) | Closed | [PR #85](https://github.com/opengeospatial/ogcapi-connected-systems/pull/85) fixed a truncated DataArray example. | PB | 053 |

### Issues 81-140

| Issue | State | Material evidence and present meaning | Class | Owning IDR topic(s) |
|---|---|---|---|---|
| [#81](https://github.com/opengeospatial/ogcapi-connected-systems/issues/81) | Closed | [PR #95](https://github.com/opengeospatial/ogcapi-connected-systems/pull/95) fixed four Part 2 references; closure does not prove the complete OAS reference graph is valid. | PB | 014 |
| [#82](https://github.com/opengeospatial/ogcapi-connected-systems/issues/82) | Closed | [PR #96](https://github.com/opengeospatial/ogcapi-connected-systems/pull/96) kept generic Sampling Feature in Part 1 and moved specializations to Part 4. | PB | 008 |
| [#83](https://github.com/opengeospatial/ogcapi-connected-systems/issues/83) | Closed | [PR #97](https://github.com/opengeospatial/ogcapi-connected-systems/pull/97) added an informative Observation example whose result link contains a base64 `data:` URL. | PB | 034 |
| [#87](https://github.com/opengeospatial/ogcapi-connected-systems/issues/87) | Closed | SWG chose JSON Schema 2020-12 to align with OAS 3.1; [PR #124](https://github.com/opengeospatial/ogcapi-connected-systems/pull/124) converted the repository. | PB | 023 |
| [#91](https://github.com/opengeospatial/ogcapi-connected-systems/issues/91) | Closed | [PR #138](https://github.com/opengeospatial/ogcapi-connected-systems/pull/138) removed claims that associations can be inline or by-reference because the actual model provides links/references. | PB | 017 |
| [#98](https://github.com/opengeospatial/ogcapi-connected-systems/issues/98) | Closed | [PR #99](https://github.com/opengeospatial/ogcapi-connected-systems/pull/99) made `elementType` soft-named and either inline or a reference, not both. | PB | 022 |
| [#100](https://github.com/opengeospatial/ogcapi-connected-systems/issues/100) | Closed | SWG rejected inline Text/XML/Binary strings for DataArray values; URI values must support `http`, `https`, and `data`; [PR #135](https://github.com/opengeospatial/ogcapi-connected-systems/pull/135). | PB | 022 |
| [#101](https://github.com/opengeospatial/ogcapi-connected-systems/issues/101) | Closed | [PR #127](https://github.com/opengeospatial/ogcapi-connected-systems/pull/127) allows inline `resultSchema` or linked `resultLink` with mandatory `mediaType`. | PB | 034 |
| [#102](https://github.com/opengeospatial/ogcapi-connected-systems/issues/102) | Closed | [PR #129](https://github.com/opengeospatial/ogcapi-connected-systems/pull/129) permits `null` for not-yet-generated ControlStream properties while retaining required fields. | PB | 036 |
| [#103](https://github.com/opengeospatial/ogcapi-connected-systems/issues/103) | Open | Proposal says XML is not required by CSAPI 1.0 but could be an optional negotiated legacy representation; no adopted disposition. | UP | 012 |
| [#104](https://github.com/opengeospatial/ogcapi-connected-systems/issues/104) | Open | Reviewer’s Guide maintenance rationale includes intentionally unspecified external encoding versions; no formal closure or linked PR. | PCD | 012, 035 |
| [#105](https://github.com/opengeospatial/ogcapi-connected-systems/issues/105) | Closed | [PR #117](https://github.com/opengeospatial/ogcapi-connected-systems/pull/117) removed deprecated XML material from SWE Common’s modern JSON-focused document. | PB | 022 |
| [#106](https://github.com/opengeospatial/ogcapi-connected-systems/issues/106) | Closed | Editor clarified that a `1000` trajectory limit is only an example value; no source change was needed. | PB | 022 |
| [#108](https://github.com/opengeospatial/ogcapi-connected-systems/issues/108) | Closed | [PR #120](https://github.com/opengeospatial/ogcapi-connected-systems/pull/120) added SensorML security guidance for transit, storage, and `securityConstraints`. | PB | 039, 040 |
| [#109](https://github.com/opengeospatial/ogcapi-connected-systems/issues/109) | Closed | [PR #134](https://github.com/opengeospatial/ogcapi-connected-systems/pull/134) removed deprecated XML/GML dependencies during SensorML refactoring. | PB | 021 |
| [#110](https://github.com/opengeospatial/ogcapi-connected-systems/issues/110) | Closed | [PR #131](https://github.com/opengeospatial/ogcapi-connected-systems/pull/131) removed an overly specific spatial-transform scope example without prohibiting transforms. | PB | 021 |
| [#111](https://github.com/opengeospatial/ogcapi-connected-systems/issues/111) | Closed | [PR #130](https://github.com/opengeospatial/ogcapi-connected-systems/pull/130) added informative SensorML/OGC API Processes relationships; it creates no CSAPI endpoint obligation. | PB | 021 |
| [#112](https://github.com/opengeospatial/ogcapi-connected-systems/issues/112) | Closed | [PR #121](https://github.com/opengeospatial/ogcapi-connected-systems/pull/121) made GeoJSON normative for locations/deployment geometry but deliberately did not make a physical process itself a GeoJSON Feature. | PB | 021 |
| [#113](https://github.com/opengeospatial/ogcapi-connected-systems/issues/113) | Open | Most publication prerequisites are resolved; [PR #197](https://github.com/opengeospatial/ogcapi-connected-systems/pull/197) defines UAV but remains approved and unmerged. | PCD | 021 |
| [#114](https://github.com/opengeospatial/ogcapi-connected-systems/issues/114) | Closed | [PR #132](https://github.com/opengeospatial/ogcapi-connected-systems/pull/132) fixed POST examples, but an asserted `name`-field mismatch was not changed and needs revalidation. | Mixed | 014 |
| [#115](https://github.com/opengeospatial/ogcapi-connected-systems/issues/115) | Closed | [PR #133](https://github.com/opengeospatial/ogcapi-connected-systems/pull/133) added informative DDIL context; replay, checkpoints, change sets, and selective sync raised in discussion were not normatively defined. | Mixed | 042 |
| [#125](https://github.com/opengeospatial/ogcapi-connected-systems/issues/125) | Closed | [PR #134](https://github.com/opengeospatial/ogcapi-connected-systems/pull/134) removed GML-derived `AbstractFeature`, refactored `DescribedObject`, and updated UML/ATS. | PB | 021 |
| [#126](https://github.com/opengeospatial/ogcapi-connected-systems/issues/126) | Closed | Published model retains condition-bearing Capabilities/Characteristics lists and arrays for other metadata; attribution to PR #134 is inferred, not formally linked. | PB | 021 |
| [#137](https://github.com/opengeospatial/ogcapi-connected-systems/issues/137) | Closed | [PR #139](https://github.com/opengeospatial/ogcapi-connected-systems/pull/139) created tag-triggered YAML bundles; discussion records recursive-reference failures/slowness in common OAS tools. | PB | 014 |

### Issues 141-195

| Issue | State | Material evidence and present meaning | Class | Owning IDR topic(s) |
|---|---|---|---|---|
| [#141](https://github.com/opengeospatial/ogcapi-connected-systems/issues/141) | Open | Parts 1/2 depend on draft OGC API Features Part 4 CRUD/update behavior; eventual publication requires a delta review. | UP | 008, 010A |
| [#142](https://github.com/opengeospatial/ogcapi-connected-systems/issues/142) | Open | Part 1 intentionally publishes seven literal `assetType` values; only the requested `cs:AssetType` classifier-definition URI remains unresolved. | UP | 024 |
| [#144](https://github.com/opengeospatial/ogcapi-connected-systems/issues/144) | Open | Maintainer rationale keeps logical structure separate from encoding, which belongs to a containing DataArray/DataStream or CSAPI wrapper. | PCD | 022 |
| [#146](https://github.com/opengeospatial/ogcapi-connected-systems/issues/146) | Closed | Pre-publication HTML lagged source/PDF until the external OGC build process was corrected; mutable renderings need provenance checks. | PB | 014 |
| [#147](https://github.com/opengeospatial/ogcapi-connected-systems/issues/147) | Open | Intended top-level DataStream `outputName` and analogous ControlStream `inputName` remain inconsistent across schemas, tables, and examples. | Mixed | 021 |
| [#148](https://github.com/opengeospatial/ogcapi-connected-systems/issues/148) | Open | Published clause numbering appears shifted by build treatment of abbreviated terms; correction versus stable anchors is unresolved. | UP | 014 |
| [#149](https://github.com/opengeospatial/ogcapi-connected-systems/issues/149) | Open | History resources were removed as too complex and `datetime` was the interim design, but Part 2 OAS still contains `/history`; future versioning alignment is unresolved. | Mixed | 018 |
| [#150](https://github.com/opengeospatial/ogcapi-connected-systems/issues/150) | Open | `/rec/system/location` is intentionally a recommendation; changing it to `/req` would incorrectly strengthen the obligation. | UP | 008 |
| [#151](https://github.com/opengeospatial/ogcapi-connected-systems/issues/151) | Closed | Confirms Part 1 publication tasks and the `v1.0.0` package; [PR #161](https://github.com/opengeospatial/ogcapi-connected-systems/pull/161). | PB | 006, 057 |
| [#152](https://github.com/opengeospatial/ogcapi-connected-systems/issues/152) | Open | Part 2 is published, but detailed requirement/conformance PURL mappings reportedly remain incomplete or return 404. | Mixed | 009, 057 |
| [#153](https://github.com/opengeospatial/ogcapi-connected-systems/issues/153) | Closed | Confirms SensorML 3.0 and its JSON-schema package were published; #183 records that the intended `timeInstantOrNow.json` file was nevertheless omitted from publication. | Mixed | 021, 023 |
| [#154](https://github.com/opengeospatial/ogcapi-connected-systems/issues/154) | Closed | Confirms SWE Common 3.0 and schemas were published with case-sensitive naming. | PB | 022 |
| [#162](https://github.com/opengeospatial/ogcapi-connected-systems/issues/162) | Open | `qualifiers` is available through inherited SensorML `DerivedProperty` but omitted from Part 1 conceptual/mapping tables. | PCD | 021 |
| [#163](https://github.com/opengeospatial/ogcapi-connected-systems/issues/163) | Open | Early use cases mix obsolete SensorML and GeoJSON conventions and are unsafe as fixtures without reconciliation. | UP | 053 |
| [#164](https://github.com/opengeospatial/ogcapi-connected-systems/issues/164) | Open | Standard does not clearly distinguish client-receivable association links from server-generated links during writes. | UP | 017 |
| [#165](https://github.com/opengeospatial/ogcapi-connected-systems/issues/165) | Open | External/local sampled-feature matching, `foi` filtering, sampling chains, derived filters, and a misplaced Part 4 example remain ambiguous. | UP | 017, 011, 024 |
| [#166](https://github.com/opengeospatial/ogcapi-connected-systems/issues/166) | Open | Complete-replacement PUT across encodings can lose representation-specific content; retain/delete/reject behavior is unspecified. | UP | 029 |
| [#169](https://github.com/opengeospatial/ogcapi-connected-systems/issues/169) | Open | Part 1 requires `recursive` on deployment queries but OAS omits it; [PR #196](https://github.com/opengeospatial/ogcapi-connected-systems/pull/196) is approved but unmerged. | PCD | 014 |
| [#170](https://github.com/opengeospatial/ogcapi-connected-systems/issues/170) | Open | Optional Update classes require PATCH but Parts 1/2 OAS omit it; patch document, arrays, nulls, atomicity, and errors remain unresolved. | UP | 014, 029 |
| [#171](https://github.com/opengeospatial/ogcapi-connected-systems/issues/171) | Open | Maintainers favor rejecting Deployment deletion without cascade and recursively deleting with cascade, without reparenting; not adopted. | PCD | 030 |
| [#172](https://github.com/opengeospatial/ogcapi-connected-systems/issues/172) | Open | Alleged missing `systemType` requirement is not an actual schema defect because composed schema inherits/narrows required `featureType`. | UP | 023 |
| [#173](https://github.com/opengeospatial/ogcapi-connected-systems/issues/173) | Closed | Published text requires `ogc-rel:` prefixes; [PR #176](https://github.com/opengeospatial/ogcapi-connected-systems/pull/176) fixes examples on post-release `master`, not in v1.0.0. | PCD | 010 |
| [#174](https://github.com/opengeospatial/ogcapi-connected-systems/issues/174) | Open | Procedure model permits `validTime` but `procedure.json` omits it; [PR #199](https://github.com/opengeospatial/ogcapi-connected-systems/pull/199) is approved but unmerged. | PCD | 023 |
| [#175](https://github.com/opengeospatial/ogcapi-connected-systems/issues/175) | Open | CSAPI defines no sorting; alignment with OGC `sortby`/Sortables is proposed but endpoint binding remains undecided. | UP | 011 |
| [#177](https://github.com/opengeospatial/ogcapi-connected-systems/issues/177) | Open | Part 2 formally requires deployment-scoped DataStream and ControlStream endpoints that its OAS omits. | UP | 014 |
| [#178](https://github.com/opengeospatial/ogcapi-connected-systems/issues/178) | Open | Server-returned and client-receivable DataStream fields intentionally differ, but descriptions and singular schema-per-format behavior need clarification. | UP | 015 |
| [#179](https://github.com/opengeospatial/ogcapi-connected-systems/issues/179) | Open | Property filters are partly inferable from ATS, but capability derivation and several OAS-advertised filters lack clear requirements. | UP | 011 |
| [#180](https://github.com/opengeospatial/ogcapi-connected-systems/issues/180) | Closed | Real SWE naming/serialization question was closed without comment, rationale, PR, or commit; closure supplies no dependable disposition. | UP | 022 |
| [#181](https://github.com/opengeospatial/ogcapi-connected-systems/issues/181) | Open | Ordinary Observation JSON Schema differs from a stream’s SWE logical `resultSchema`; custom `+json` does not automatically imply JSON Schema. | UP | 023 |
| [#182](https://github.com/opengeospatial/ogcapi-connected-systems/issues/182) | Open | Valid-time prose/schema differ over `now`; open intervals are absent and a common published schema target reportedly returns 404. | UP | 018, 023 |
| [#183](https://github.com/opengeospatial/ogcapi-connected-systems/issues/183) | Open | Publication omitted intended `timeInstantOrNow.json`; redirecting to `timeInstant.json` would remove intended `now` support. | UP | 023 |
| [#185](https://github.com/opengeospatial/ogcapi-connected-systems/issues/185) | Open | Published transactions are individual-only while OAS contains contradictory array/batch artifacts; common bulk semantics remain draft work. | UP | 014, 029 |
| [#186](https://github.com/opengeospatial/ogcapi-connected-systems/issues/186) | Open | Proposes using versioned v1.0 bundled OAS for ReDoc rendering/download instead of fragile modular entry files. | UP | 014 |
| [#187](https://github.com/opengeospatial/ogcapi-connected-systems/issues/187) | Open | Recorded direction uses CloudEvents for resource lifecycle envelopes while leaving native data messages unwrapped for constrained/DDIL efficiency. | PCD | 035 |
| [#188](https://github.com/opengeospatial/ogcapi-connected-systems/issues/188) | Open | Status of MQTT/NATS/Kafka/AMQP/DDS bindings as classes, profiles, extensions, or separate publications is unresolved. | UP | 035 |
| [#189](https://github.com/opengeospatial/ogcapi-connected-systems/issues/189) | Open | Part 3 MQTT clause is an unwired stub lacking normative version, topics, operations, QoS/session, retained-message, event, and ATS behavior. | UP | 035 |
| [#190](https://github.com/opengeospatial/ogcapi-connected-systems/issues/190) | Open | Multiple-encoding direction favors advertised channel selection through AsyncAPI, but the final model is unresolved. | UP | 035 |
| [#191](https://github.com/opengeospatial/ogcapi-connected-systems/issues/191) | Open | Recorded proposal renames legacy `consys` CloudEvent types to `csapi`; no adopted draft change. | PCD | 035 |
| [#192](https://github.com/opengeospatial/ogcapi-connected-systems/issues/192) | Open | SWG accepted CloudEvents `source`+`id` uniqueness instead of UUID-only values; [PR #198](https://github.com/opengeospatial/ogcapi-connected-systems/pull/198) remains approved and unmerged on a Part 3 branch. | PCD | 035 |
| [#193](https://github.com/opengeospatial/ogcapi-connected-systems/issues/193) | Open | Aggregate collection-summary events are distinct from HTTP bulk and CloudEvents batch format; semantics remain undecided. | UP | 035 |
| [#194](https://github.com/opengeospatial/ogcapi-connected-systems/issues/194) | Open | `parentID` purpose, scope, naming, requirement level, and URL-versus-local-ID representation remain unresolved. | UP | 035 |
| [#195](https://github.com/opengeospatial/ogcapi-connected-systems/issues/195) | Open | Event payload options and boundary from native Resource Data Messages remain unresolved. | UP | 035 |

## 5. Pull-Request Dispositions

All 58 pull requests are accounted for: 35 are represented by a retained issue row, nine material standalone pull requests are registered below, and 14 were screened out with reasons.

### Material Standalone Pull Requests

| Pull request | State and merge commit | Material evidence and present meaning | Class | Owning IDR topic(s) |
|---|---|---|---|---|
| [#56](https://github.com/opengeospatial/ogcapi-connected-systems/pull/56) | Merged, [`31349f0`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/31349f0768ef0f13686e3a9f8f2d3a20d176da3b) | Corrected invalid trailing commas in Part 1 `datetimeSchema.yaml` and removed invalid schema-level `geometry: null` from `procedure.json`; included in v1.0.0. | PB | 014, 023 |
| [#63](https://github.com/opengeospatial/ogcapi-connected-systems/pull/63) | Merged, [`01b2f0b`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/01b2f0b337555bb8fe6e493c9dbe061af0645e1a) | Added a missing `$schema` declaration to `commandStatusCode.json` after parser failures. Its Draft-07 value was later superseded by the repository-wide 2020-12 conversion under #87; use the final tagged schema dialect. | PB | 023 |
| [#67](https://github.com/opengeospatial/ogcapi-connected-systems/pull/67) | Merged, [`271b167`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/271b167aa8b7dd338f763a169072dad0c783b821) | Corrected invalid Part 1 OAS Schema syntax in `resourceLinks.yaml`, where `required` had been a scalar instead of an array; included in v1.0.0. | PB | 014, 023 |
| [#136](https://github.com/opengeospatial/ogcapi-connected-systems/pull/136) | Merged, [`a1e5007`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/a1e50079fe0a85ecaad3f5b0d3203dccdacd7d1a) | Replaced invalid Part 2 AsyncAPI and by-reference OAS-example links to removed `*_view`/`*_create` files with unified schemas governed by `readOnly`/`writeOnly`; included in v1.0.0. | PB | 014, 023, 035 |
| [#155](https://github.com/opengeospatial/ogcapi-connected-systems/pull/155) | Merged, [`5d051b1`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/5d051b12a841830c3ffd235d948a5eb467e9734d) | Moved SWE Common 3 schema references from mutable raw GitHub to `schemas.opengis.net` and changed the external identifier from DIS to IS before publication. | PB | 022, 023 |
| [#156](https://github.com/opengeospatial/ogcapi-connected-systems/pull/156) | Merged, [`d672ed4`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/d672ed466e269eb197f6e4ac86f42230d5d20471) | Moved SensorML 3 schema references to `schemas.opengis.net` and pinned the published SWE Common 3 normative reference. | PB | 021, 023 |
| [#158](https://github.com/opengeospatial/ogcapi-connected-systems/pull/158) | Merged, [`e7d46df`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/e7d46df3560a74a4a3e92544d0e24d8f81c8e031) | Final SensorML publication review corrected inherited SWE requirement/ATS references from 2.0 to 3.0, ISO inheritance citations, ATS cross-links, and core Requirement 7. | PB | 008, 021, 050 |
| [#159](https://github.com/opengeospatial/ogcapi-connected-systems/pull/159) | Merged, [`dd4f9f6`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/dd4f9f6aefd31e063c01300f7d819464473dc90d) | Changed Parts 1/2 external identifiers from DIS to IS and moved schema/example roots from mutable GitHub to the published `schemas.opengis.net` package. | PB | 014, 023, 057 |
| [#160](https://github.com/opengeospatial/ogcapi-connected-systems/pull/160) | Merged, [`71d1d68`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/71d1d684961a085a4a0a4bf6484419d299a59014) | Final Part 1 review aligned the Observation definition to SOSA/SSN and corrected canonical-resource and link-relation wording before publication. | PB | 015, 017, 034 |

### Pull Requests Represented by Issue Rows

These 35 pull requests add no separate disposition beyond the linked issue row: `#29→#28`, `#85→#80`, `#88→#64`, `#89→#66`, `#90→#61`, `#92→#62`, `#93→#65`, `#94→#74`, `#95→#81`, `#96→#82`, `#97→#83`, `#99→#98`, `#116→#76`, `#117→#105`, `#118→#20/#71`, `#120→#108`, `#121→#112`, `#124→#87`, `#127→#101`, `#129→#102`, `#130→#111`, `#131→#110`, `#132→#114`, `#133→#115`, `#134→#109/#125`, `#135→#100`, `#138→#91`, `#139→#78/#137`, `#140→#73`, `#161→#151`, `#176→#173`, `#196→#169`, `#197→#113`, `#198→#192`, and `#199→#174`.

### Screened-Out Pull Requests

- `#26`: closed unmerged with no official-PR files or reproducible published-artifact disposition; generic external-incubator validation proposal.
- `#35`: editorial terminology correction for excluded issue #31.
- `#37`: implementation-list link update only.
- `#38`: closed unmerged publication-link workflow, superseded by a separate build fix; issue #146 retains the durable provenance lesson.
- `#42`: experimental bulk SensorML/build import merged only to a nonbaseline feature branch; issue #46 retains the published 3.0 migration outcome.
- `#84`: one-character grammatical correction only.
- `#86`: document table-width/formatting cleanup for excluded issue #75.
- `#119`: abbreviation/editorial cleanup only.
- `#122`: company-name attribution correction only.
- `#123`: redundant foreword clarification of XML removal already captured by #46, #105, and PR #117.
- `#128`: O&M-to-OMS terminology/reference cleanup without a distinct server-contract effect.
- `#143`: generic revision-history metadata only.
- `#145`: README link administration only.
- `#157`: final SWE Common copy-edit pass without a distinct implementation-semantic disposition.

## 6. Screened Issue Exclusions

The following 23 issues were reviewed but not retained because they add no material server-design evidence under the selection rules above:

- Issues 1-80: `#3`, `#7`, `#10`, `#25`, `#27`, `#31`, `#33`, `#34`, `#41`, `#44`, `#49`, `#50`, `#53`, `#54`, `#60`, `#69`, `#70`, `#72`, and `#75`.
- Issues 81-140: `#107`.
- Issues 141-195: `#167`, `#168`, and `#184`.

An excluded item may be restored if its effect on a published artifact, implementation contract, interoperability behavior, or material design rationale becomes evident later.

## 7. Topic Routing Summary

The owner column is the controlling routing device. The following summary highlights the densest or most consequential clusters; it does not narrow the individual rows above.

| Topic | Required upstream-history focus |
|---|---|
| IDR-SRV-008 | Conformance/encoding combinations, corrected identifier relationships, Sampling Feature Part 4 boundary, draft CRUD dependency, and recommendation-versus-requirement wording: #23, #28, #82, #141, #150. |
| IDR-SRV-010/010A | Canonical resources, link relations, version/release deltas, the draft Features Part 4 dependency, and the post-release example fix: #22, #47, #141, #173. |
| IDR-SRV-011 | Query time tokens, sorting, property filtering, and sampling-feature filtering ambiguity: #64, #165, #175, #179. |
| IDR-SRV-014 | Modular and bundled OAS provenance, dependency closure, tool behavior, examples, omitted or contradictory operations/parameters/endpoints, and later corrections: #12, #13, #48, #57, #77-#79, #81, #114, #137, #146, #148, #169-#170, #177, #185-#186. |
| IDR-SRV-017 | Link and association design, Deployment/System relationships, canonical links, and write-time link ambiguity: #2, #4, #22, #30, #47, #51, #62, #91, #164, #165. |
| IDR-SRV-018 | History-resource removal, remaining OAS path, and valid-time/open-bound ambiguity: #149, #182. |
| IDR-SRV-021 | SensorML model refactoring, GeoJSON boundary, missing/inconsistent fields, and current terminology/schema maintenance: #39, #45-#46, #76, #109-#113, #125-#126, #147, #153, #162. |
| IDR-SRV-022 | SWE Common JSON structures/encodings, values, UOMs, XML removal, and the reversed `recordsAsArrays`/`vectorsAsArrays` prose: #5-#6, #11, #15-#20, #40, #46, #55, #71, #73-#74, #98, #100, #105-#106, #144, #154, #180. |
| IDR-SRV-023 | Schema dialect and validation contradictions/defects, including quaternion typing, reversed booleans, temporal files, and Procedure validity: #18, #43, #71, #87, #172, #174, #181-#183. |
| IDR-SRV-024 | Semantic identifiers, UOMs, evolving vocabularies, asset types, and sampling-feature handoffs: #9, #40, #73-#76, #142, #165. |
| IDR-SRV-029/030 | Cross-encoding replacement, PATCH, bulk artifacts, cascade behavior, and Deployment deletion: #61, #66, #166, #170-#171, #185. |
| IDR-SRV-034 | Linked/inline results and server-generated dynamic fields: #65, #83, #101. |
| IDR-SRV-035 | Part 3, AsyncAPI, common Pub/Sub alignment, CloudEvents, transports, encodings, and event payloads: #14, #68, #104, #187-#195. |
| IDR-SRV-037/038 | Command/action mapping, reservation/confirmation, authority transfer, and async tasking: #1, #58-#59. |
| IDR-SRV-042 | DDIL claims versus mechanisms actually defined: #115. |
| IDR-SRV-053 | Do not reuse stale use cases as fixtures; retain corrected DataArray examples: #80, #163. |
| IDR-SRV-057 | Refresh the complete register, reconcile material changes since topic completion, and ensure every unresolved/post-publication item has an explicit disposition. |

## 8. Required Use During Research

The owner column sets operational priority: entries owned by the active topic are due now, directly implicated cross-cutting entries may be added, and all other rows wait for their owning topic or the final IDR-SRV-057 refresh. This prevents the register itself from expanding a topic's scope.

For each active research topic that depends on OGC API - Connected Systems:

1. **Start with the published standard.** Establish the applicable approved document, clause, requirement, conformance class, ATS test, and version before using this register.
2. **Bound the history review.** Consult the rows owned by the active topic plus directly cross-cutting rows revealed by the standard or artifact analysis. Follow only the official issue, its materially linked pull request/review, the resulting commit, and release/tag inclusion needed to establish disposition. Do not roam through unrelated contributors, repositories, or speculative discussions.
3. **Refresh before relying on a row.** Check the current issue and pull-request state, whether the change merged, the target branch, the commit, and whether that commit entered a published release. Update the row and change log when any material fact changed.
4. **Keep four things visibly separate in the report:** the standards obligation; an artifact observation; explanatory or maintenance history; and the Glaux recommendation or decision.
5. **Test contradictions instead of voting on them.** When prose, ATS, schema, OAS, examples, and issue history disagree, record each source and its authority. Do not select whichever source is easiest to implement.
6. **Do not reopen completed or accepted reports automatically.** If a refreshed row materially contradicts a completed or accepted report, record the delta, identify the owning topic and downstream impact, and obtain project-lead authorization for a focused correction at the next approval boundary.
7. **Refresh comprehensively at IDR-SRV-057.** Start with issue/PR state, merge, branch, commit, and release metadata deltas; reread a full thread only when those facts or a material standards/artifact conflict changed. Then disposition every material open, post-publication, and mixed entry as adopted baseline, Glaux compatibility choice, monitoring item, explicit deferral, or unresolved project decision.

## 9. Immediate Research Consequences

The initial screening creates four concrete guardrails for the remaining IDR sequence:

- IDR-SRV-008 must resolve what the published conformance classes actually prove and must not assume that encoding conformance automatically guarantees every write method/media-type combination (#23).
- IDR-SRV-014 must analyze both tagged modular OAS trees and both released bundled OAS 3.1 files. It must account for known reference, omission, recursion/tooling, example, and post-release maintenance evidence rather than treating one rendering as the whole API contract.
- IDR-SRV-023 must explicitly reconcile schema/OAS/standard contradictions, especially the reversed `recordsAsArrays` and `vectorsAsArrays` normative prose (#71), rather than allowing a validator library to make the policy decision accidentally.
- IDR-SRV-035 must inspect the available AsyncAPI and Part 3 material but treat it as unsettled design input. MQTT, transport classes, CloudEvents details, multiple encodings, batch events, parent identifiers, and event payload boundaries are not a published CSAPI 1.0 server obligation.

Two general traps apply everywhere: issue closure can disagree with the released artifact (#64), and an approved or merged post-release correction still does not rewrite the published tag (#169, #173, #174).

## 10. Limitations

- This is a point-in-time screen of public repository evidence. States, branches, comments, and release metadata can change.
- Some older issues lack a formally linked closing pull request or commit. Where the published outcome matches the discussion but traceability is incomplete, the row says so.
- SWG decisions reported in issue comments are useful rationale but are not independently authenticated meeting records unless a stable official record is linked.
- The register does not claim that every modular or bundled OAS/schema defect has been found. Mechanical artifact analysis remains part of the owning research topic.
- External OGC build systems, unpublished meeting material, and private communications were outside this public-repository screen.

## 11. Maintenance Log

| Date | Version | Change | Owner |
|---|---|---|---|
| August 1, 2026 | 1.0 | Initial complete screen of 141 issues and 58 pull requests; retained 118 material issues and nine standalone PRs, dispositioned the other PRs through issue rows or explicit exclusions, pinned the v1.0.0 and Part 3 source snapshots, classified evidence, and routed entries to owning IDR topics. | Glaux research workflow |

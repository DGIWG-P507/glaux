# Section 060: CSAPI Part 5 Protobuf-First Implementation Study - Research Report

**Topic ID:** IDR-SRV-060<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-060](../IDR%20Plans/idr-srv-060-csapi-part-5-protobuf-first-implementation-study.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Five; specification maturity, interoperable binding, semantic preservation and safety, implementation and verification, recommendation<br>
**Methodology Used:** Pinned primary-source and peer-code inspection, targeted prior-research reconciliation, Rust tooling review, small syntax/static/numerical checks, and independent source reviews<br>
**Research Time:** One AI-assisted research/report iteration, September 17–18, 2026, America/New_York; source investigation began at 03:16 UTC September 18. An overnight pause preceded report completion; elapsed calendar time is not a human-effort estimate.<br>
**Primary Source(s):**

- [Approved CSAPI Part 2][part2] and [SWE Common 3.0][swe]
- [Official CSAPI source snapshot][ogc-tree], [Protobuf documentation][protobuf-docs], and the pinned implementation sources below

**Supporting Resources:** [Goal v1.7][goal], [draft Guide v0.2][guide], accepted IDR-SRV-007/012/014G/022/023/035/059, and the [upstream-history register][history]<br>
**Document Purpose:** Decide whether to plan an experimental Protobuf implementation now, preserve compatibility points, or defer selection pending a usable Part 5 binding<br>
**Author(s):** Glaux research workflow, AI-assisted<br>
**Accepted By:** Pending Glaux Project Lead<br>
**Acceptance Date:** Pending<br>
**Date:** September 18, 2026<br>
**Last Updated:** September 18, 2026

---

## Usage Rules

This report follows the [Research Report Template](../../../../../Governance/research-report-template.md). Published standards establish obligations; repository examples, unmerged proposals and observed peer behavior do not. Findings, interpretation and recommendations are distinguished below. Research completion does not mean project-lead acceptance, adoption of an implementation, or demonstrated conformance.

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

**Protobuf is worth considering, and there is real implementation work to learn from. The available public evidence does not yet provide a complete, agreed OGC Part 5 contract to implement exactly.**

The official CSAPI repository contains exploratory Protobuf schemas and examples, not a dedicated Part 5 requirements/conformance baseline in the inspected branches. Some examples have concrete defects. More importantly, they do not settle schema identity, full SWE mappings, framing or the complete HTTP/publication binding. Repairing their syntax would not resolve those decisions.

OpenSensorHub has a substantially more developed proposal: an open draft add-on pull request implements generated schemas/descriptors, observation and command codecs, schema discovery and length-delimited messages. Its author explicitly describes this as the basis of an in-progress Part 5 draft. It is useful candidate implementation evidence, not merely an unrelated internal codec. However, it remains unmerged, depends on an unmerged core change, differs from the official examples, and has semantic and interoperability limitations. CS-Go's inspected main branch supports Protobuf schema metadata and validation of JSON results against such schemas; that is not demonstrated binary Protobuf exchange. [Official artifacts][ogc-tree]; [OSH proposal][osh-pr]; [CS-Go validation][go-validation].

**Recommendation:** Preserve the existing design's ability to add a Protobuf codec, but do not add a Part 5 implementation to the server's completion target yet. Revisit the implementation choice against a versioned, sufficiently complete proposed binding. This does not require a new service, document system, research program or pause in unrelated server planning.

If the project lead wants implementation before that material is available, a **specifically identified OSH-compatible Protobuf experiment** is a viable alternative to discuss. It would need an explicit subset, pinned wire/schema behavior and independent tests; it must not be described as exact or conformant implementation of a complete OGC Part 5 standard. This report has not adopted that alternative.

The project lead's September 17 SWG account—that Protobuf was favored and may be the only first-publication encoding—is credible planning context and explains this study's focus. No public record inspected independently confirms that release decision. Existing required SWE JSON, Text and Binary support remains unchanged, as do the selected Part 3/Part 4 experiments and enhanced observation filtering.

## 2. Scope and Plan Alignment

IDR-SRV-060 was authorized for execution by the user's `proceed` after publication of its plan in `488b9ba`. This iteration assesses available evidence and produces one report. It does not amend the Goal, Guide, final synthesis or Roadmap, or implement any feature.

Completed scope includes the official artifact inventory; comparison with SWE Binary; observation, command and publication contracts; semantic/security analysis; relevant OSH and CS-Go source; Rust feasibility; representative verification cases; and implications for the existing design. FlatBuffers, FlatGeobuf and video were checked only as overview proposals. Their implementation, gRPC, new transports, performance benchmarking and a schema-registry service are outside scope.

### Research Question Coverage Matrix

| Plan question | Question | Coverage status | Evidence location |
|---|---|---|---|
| Q1 | Available specification and maturity | Complete assessment; a complete official binding remains unavailable in the searched material | §§3, 4.1 |
| Q2 | Added value and interoperable binding | Complete assessment; substantive binding choices remain unresolved | §4.2 |
| Q3 | Meaning, compatibility and safety | Complete assessment of inspected mappings and risks; no lossless implementation demonstrated | §4.3 |
| Q4 | Implementation and verification | Complete source-based feasibility assessment; no compiler, peer service or Rust codec execution | §4.4, Appendix A |
| Q5 | Recommendation and planning impact | Complete; adoption remains a project decision | §§4.5–10 |

Here, “complete assessment” means the question is answered to the available evidence, with its limitations stated. It does not turn an unresolved protocol choice into a solved one.

## 3. Evidence Base

### 3.1 Primary Sources Reviewed

Access dates in this report use America/New_York. The main source investigation occurred September 17, 2026, corresponding to September 18 UTC; report completion followed September 18 local time. Full commit identifiers below are the reproducibility anchors, not claims about future branch state.

| Source | Type | Version / status | Authority class | Stable anchor | Access date | Limitations |
|---|---|---|---|---|---|---|
| [CSAPI Part 2][part2] | Standard | OGC 23-002, 1.0 | Approved | §§9.6, 10.6, 16.4; requirements 123–130 | 2026-09-17 | Controls existing bindings, not missing Part 5 rules |
| [SWE Common][swe] | Standard | OGC 24-014, 3.0 | Approved | §10.4 binary encoding | 2026-09-17 | Protobuf is not this encoding |
| [Official CSAPI master][ogc-tree] | Source tree | `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f` | Informative/proposed artifacts where not grounded in published requirements | README; Part 2 Protobuf examples/wrappers; SWE experiments | 2026-09-17 | Not an adopted Part 5 snapshot |
| [Published-source tag][ogc-tag] | Source | `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2` | Published-source provenance | `v1.0.0` | 2026-09-17 | Repository inclusion alone does not create an encoding class |
| [Part 3 working draft][part3-tree] | Draft source | `6f529a15bfa63259febc3620378d3e5a06305333` | Existing project-selected experimental baseline | `api/part3` | 2026-09-17 | Not an approved publication |
| [Representation-selection proposal][selection-tree] | Separate branch | `6f5987673dd11a8e27a2b6093e2dc5cbd0d16612` | Unincorporated proposal | offering/selection requirements and tests | 2026-09-17 | Not in inspected master or Part 3 working baseline |
| [Protobuf documentation][protobuf-docs] | Maintainer documentation | Dated online documentation; proto3 examples inspected | Controls Protobuf mechanics, not CSAPI mappings | Presence, encoding, evolution, framing, media, Rust | 2026-09-17 | No CSAPI conformance authority |
| [Goal][goal] / [Guide][guide] | Project documents | Approved v1.7 / draft v0.2 at local baseline `488b9ba` | Project scope / current draft design | §§4.3, 4.4.1, 4.8, 6.2, 6.5, 7–8 of Guide | 2026-09-17 | No Part 5 implementation adopted |

The Part 4 branch was also reconfirmed at `05a3c62d198ee52d0cf81a734b700967b7d864a1`; this study did not reopen its selected static-type scope.

### 3.2 Supporting Sources Reviewed

| Source | Version / commit | Relevance / evidence class | Stable anchor | Access date | Limitations |
|---|---|---|---|---|---|
| [OSH add-on proposal][osh-pr] | PR 224, open/draft/unmerged; head `22d98d16c77e1e224365fd458dfc3ba77205f78e` | Candidate CSAPI Protobuf implementation | `services/sensorhub-service-consys-proto` | 2026-09-17 | Source inspected, not deployed or executed |
| [OSH core support proposal][osh-core-pr] | PR 354, open/unmerged; head `6988ded63679fc55b8741f92408a4d883e245d1e` | Custom observation/command/schema hooks | `sensorhub-service-consys` | 2026-09-17 | Required proposed integration, not merged baseline |
| [OSH core master][osh-core-main] / [add-ons master][osh-addons-main] | `9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08` / `9682ce901cafe2d532f5700294176f895e823481` | Distinguish merged code from proposal | Relevant source and complete add-on path inventory | 2026-09-17 | Negative searches are bounded, not universal absence claims |
| [CS-Go main][go-tree] | `b1fd2e0e9bd69e222d05258d659a842ca24502cb` | Schema advertisement and JSON-result validation | validator, schema-format e2e tests, seed README | 2026-09-17 | No binary CSAPI codec identified in the bounded scan |
| [Official Rust release][google-release] | Protobuf v36.2; `2c74169b34066ceb8ddb6b882fcb3fb32d737a55` | Generated-code runtime/toolchain feasibility | Rust runtime README, manifest, build script | 2026-09-17 | Beta; C/C++ kernels; no local build |
| [prost][prost-readme] | 0.14.4; `13646cde7eab75c81b3047767aa0a86e7dbecf12` | Pure-Rust generated message/runtime option | README, manifests, derive/build code | 2026-09-17 | No built-in runtime reflection; no local build |
| [prost-reflect][reflect-api] | Core 0.16.5; `2d0e1847a07d2582ae3edf84dbb80f2684961fef` | Dynamic descriptors/messages | Manifest and versioned API | 2026-09-17 | Feasibility, not a selected dependency |
| Accepted reports | Repository baseline `488b9ba` | Prior findings reconciled, not independently re-audited in full | 007 §9.4; 014G LL-18; 012 §§6–9; 022 §4.4; 023 §13.2; 035 §9; 059 | 2026-09-17 | Current approved project scope controls over older planning proposals |

### 3.3 Evidence Quality Notes

- Source inspection supports concrete implementation findings, not runtime conformance or interoperability. The peer tests discussed below were read, not run.
- The official search covered public refs, the pinned master tree, relevant text/artifacts and targeted issue discussions. It did not cover private branches, unavailable minutes, email or unpublished author drafts.
- The project lead's account of Protobuf prioritization and prospective STANAG inclusion remains attributed meeting context. Neither is treated as a new approved obligation.
- High confidence: artifact contents, pinned code differences, and the distinction between existing SWE Binary and proposed Protobuf. Limited confidence: eventual Part 5 release content, complete cross-implementation behavior and performance, which remain unverified.

## 4. Findings by Research Question

### 4.1 Q1 — Available specification and maturity

**Finding:** The inspected official public material identifies Part 5's intent but does not furnish a complete Part 5 contract. OSH supplies an identifiable, more substantial candidate implementation.

The official remote exposed 25 branches and three tags at inspection. None was Part 5-named. The master tree had no dedicated Part 5 directory; relevant Markdown/AsciiDoc searches found overview statements rather than Part 5 requirements, conformance classes or abstract tests. The [README][ogc-readme] lists Protobuf/FlatBuffers for observation/command data, FlatGeobuf for features and tentative video/other formats. These alternatives do not overturn the study's Protobuf-first focus or establish a first-publication commitment.

The actual official inventory was:

| Artifact group | Observed content | What it does not establish |
|---|---|---|
| Five Part 2 `.proto` examples | Scalar observation, GeoPose variants/vector, PTZ command | A mapping algorithm for arbitrary SWE structures or all CSAPI resources |
| SWE experiment message and `swe_options.proto` | Another pose example and ten string-valued field annotations | A complete annotation model or standard binding |
| Observation/command schema wrappers | `application/x-protobuf`; inline/linked `messageSchema` | Root message identity, import closure, revision binding or framing |
| Scalar JSON schema example | Link to textual `.proto` through `schema?f=proto` | Demonstrated data endpoint support |

There are seven `.proto` files overall: six message examples plus the options file. The vector and `BasicYPR_Schema.proto` examples are byte-identical. Relevant data request/collection-response OpenAPI artifacts do not advertise `application/x-protobuf`, despite the schema examples. Sources: [example directory][ogc-examples], [experiments][ogc-experiments], [observation wrapper][obs-wrapper], [command wrapper][cmd-wrapper], [observation request][obs-request], [collection response][obs-response].

Static inspection found explicit defaults in proto3 messages; unqualified `Timestamp` references; mismatches between namespaced/message options used by examples and the unnamespaced field-only extensions supplied; duplicate field numbers 1, 2 and 3 in the [GeoPose reference variant][geopose-ref]; and malformed separators in the [PTZ example][ptz-example]. These are source findings, not compiler diagnostics. The scalar example also confuses its time comment and semantic annotation, while neither supplies separate complete phenomenon/result-time handling. [Scalar example][scalar-example]; [options definition][swe-options].

**Interpretation:** These files are useful design history, not ready-to-copy interoperability fixtures. Syntax correction is necessary but insufficient: a binding must also specify the meaning and context of the encoded values.

The OSH add-on [PR 224][osh-pr] describes its work as the basis of an in-progress Part 5 draft. Therefore, “no Part 5 work exists” would be false. The narrower supported conclusion is that this study did not locate a sufficiently complete, versioned official OGC binding to select as Glaux's exact target. The OSH candidate can be assessed and, with an explicit project decision, targeted as an experiment.

Targeted public issue searches did not identify corroboration of the September 17 meeting's Protobuf-only possibility. [Issue 144's maintainer comment][issue144-comment] supports separation of logical structure and encoding, including Protobuf, but is not a release-scope decision. The [historical media-type thread][issue21] does not settle a current binding either.

### 4.2 Q2 — Added value and interoperable binding

**Finding:** Protobuf's potential benefit is another broadly usable serialization/tooling path, not removal of the existing binary requirement. Its exact benefit and interoperability depend on the selected mapping.

Approved Part 2 §16.4 defines the SWE Common Binary class, including observation and command representations and applicable read/write behavior. Its binding uses `application/swe+binary` and a schema combining record structure with `BinaryEncoding`. SWE Common §10.4 supplies encoding controls such as component representation, byte order, variable arrays and optional/choice handling. Protobuf is a different format with different mapping rules. [Part 2][part2]; [SWE Common][swe].

| Consideration | Existing SWE Binary commitment | Proposed Protobuf path |
|---|---|---|
| Authority | Approved CSAPI/SWE requirements | Exploratory official artifacts and an unmerged peer proposal |
| Schema machinery | SWE component structure plus BinaryEncoding | `.proto`/descriptor plus message type and a CSAPI/SWE mapping |
| Client opportunity | SWE-aware encoder/decoder | Generated clients or dynamic descriptor-based clients in multiple languages |
| Semantic coverage | Published mapping to implement and test | Must establish supported constructs and loss/rejection behavior |
| Evolution | Bound to existing immutable stream contracts | Field-number/wire compatibility is additional to, not a replacement for, semantic revision binding |
| Size/CPU benefit | Workload dependent | Also workload dependent; this study measured neither |
| Server effect | Already within completion scope | Additional schema, codec, negotiation and test work if adopted |

Protobuf is not synonymous with compression, ProtoJSON or gRPC. Its selection does not add an RPC service, change CSAPI resource semantics or require a new transport. [Protobuf encoding][proto-encoding]; [ProtoJSON][proto-json].

#### The three inspected interfaces are not interchangeable

| Contract element | Official exploratory examples | OSH proposed implementation | Consequence for Glaux |
|---|---|---|---|
| Data media type | `application/x-protobuf` | `application/swe+proto`; shorthand `swe-proto` | Cannot infer one from the other; generic Protobuf documentation separately recommends `application/protobuf` |
| Schema document | `messageSchema` string or link | JSON with `messageType` and base64 `fileDescriptorSet`, plus format property | Text source and a descriptor bundle need different discovery/validation handling |
| Root identity | No wrapper-level root name/namespace rule | Fixed fully qualified Observation/Command names | A message name alone does not identify a particular stream revision |
| Command schema selector | Published Part 2 uses query parameter `cmdFormat`; wrapper property is a separate issue | Proposed core handler actually reads `commandFormat` | Peer compatibility cannot silently replace the published parameter |
| Data coverage | Hand-authored observation/PTZ examples | Observation and command read/write bindings | No demonstrated complete mapping for feature documents, statuses, events or every SWE component |
| Framing | No complete single/list/publication contract | Length-delimited dynamic messages | Matching payload schema alone will not make clients interoperable |

Sources: [official wrappers][obs-wrapper], [command wrapper][cmd-wrapper], [OSH format selection][osh-format], [schema export][osh-obs-schema], [command schema handler][osh-cmd-handler], [Protobuf media guidance][proto-media]. In particular, the JSON property `commandFormat` must not be confused with the approved HTTP query parameter `cmdFormat`.

**Design implications, not new standards requirements:** Before offering an experiment, document how `Accept`/`f`, request `Content-Type`, `obsFormat`/`cmdFormat`, schema-document representation and capability advertisements work together. Unsupported output selection should follow the existing not-acceptable handling; unsupported input media should follow unsupported-media handling. A supported media type with malformed content is a validation failure, not an excuse to reinterpret bytes as JSON. A compatibility alias, if chosen, must be explicit and tested.

A usable schema response needs the root message, all required descriptors/imports and a binding to the immutable logical stream revision. Base64 inside the OSH schema wrapper encodes descriptor bytes; it does not imply base64 encoding of every data payload. Different stream schemas sharing the same Protobuf names require separated descriptor contexts or a specified naming policy.

The OSH observation writer currently leaves observation/datastream/feature IDs unset; its source explicitly marks ID encoding as unfinished. Its inbound reader uses request context. A context-bound single-stream workflow may therefore be possible, but complete context-free records, global collections, pagination/link representation and cross-stream batches are not demonstrated. Commands carry more envelope information; neither observation support nor command-parameter support establishes command-status/result serialization. [Observation binding][osh-obs-binding]; [command binding][osh-cmd-binding].

#### Part 3 interaction

The existing design distinguishes native Resource Data Messages from CloudEvents-wrapped resource lifecycle events. Protobuf would be an additional explicitly advertised data representation, not an instruction to turn every event into Protobuf or to publish every supported HTTP encoding continuously. Existing JSON-first progression does not permanently exclude binary formats. [IDR-SRV-035][r035], §9.

[Issue 190][issue190] remains open. A separate official branch at `6f598767...` proposes explicit representation offering/selection and corresponding tests. It recognizes that content-type metadata alone does not negotiate a channel. The proposal was not an ancestor of either inspected master or the selected Part 3 working draft. It is useful direction, **not an adopted replacement for Glaux's Part 3 pin** and not a Protobuf mapping. [Offering proposal][offering]; [selection proposal][selection]; [issue discussion][issue190-comment].

### 4.3 Q3 — Meaning, compatibility and safety

**Finding:** Protobuf tooling can carry the required information only where the mapping explicitly preserves it. Successful decoding does not establish correct CSAPI/SWE meaning.

The following are concrete findings and their bounded design consequences:

| Concern | Evidence-backed issue | Recommendation for any experiment |
|---|---|---|
| Presence, defaults and nil | Proto3 implicit scalar presence can collapse absent and default-valued fields; repeated fields do not distinguish absent from empty | Specify absent, present-zero/false/empty, and each permitted SWE nil representation separately; reject unsupported distinctions |
| Numbers | Float32/float64 and fixed-width integers have finite ranges/precision | Map deliberately; do not route all numbers through a lossy generic floating representation |
| Time | Protobuf Timestamp is an instant representation, not the full SWE/CSAPI temporal model | Preserve distinct phenomenon/result times; define precision/reference semantics and reject unrepresented intervals/open states |
| Semantics | Tags/types alone do not identify units, property definitions, constraints, frames or sampling context | Bind encoding to the full logical contract, not only the descriptor |
| Evolution | Stable field numbers can still acquire incompatible meaning through changed units or mapping | Keep historical schema/value interpretation immutable; never reuse retired field numbers for another meaning |
| Unknown content | Generated and dynamic Rust paths have different preservation behavior | Choose explicit admission/preservation behavior; do not promise universal lossless forwarding |
| Duplicate fields and byte identity | Protobuf decoding can merge repeated occurrences; deterministic serialization is not canonical | Test specified parser behavior; compare logical meaning separately from original-byte identity |

Sources: [field presence][proto-presence], [encoding][proto-encoding], [Timestamp][proto-timestamp], [evolution][proto-evolution], [noncanonical serialization][proto-canonical]. These consequences supplement the existing immutable-contract design; they do not prescribe a new storage architecture.

OSH's proposed writer preserves useful SWE annotations, including definitions, units and reference-frame information. Nevertheless, the inspected reader explicitly reconstructs vectors as records, ranges as two scalars, and repeated fields as variable arrays without preserving fixed counts. The writer's annotation mapping does not carry nil/constraint metadata. Its ordinary proto3 scalar declarations do not set explicit `proto3Optional`, while scalar decoding retrieves default values. These are substantive limits on reconstructing a complete logical SWE model solely from that descriptor. Unsupported Geometry and arrays containing choices are rejected by the format compatibility check. Ordinary choices have implementation code despite a stale writer comment, but with restrictions: direct range, array and nested-choice items are rejected. [Writer][osh-writer]; [reader][osh-reader]; [decoder][osh-decoder]; [format compatibility][osh-format].

**Interpretation:** A retained authoritative SWE schema may prevent some metadata loss, but that is a design condition to establish, not evidence that the peer format already round-trips every approved construct. Glaux must not silently reduce its approved SWE component support to this experiment's subset.

Ordinary generated `prost` skips unknown tags. `prost-reflect::DynamicMessage` preserves unknown fields when re-encoding; neither promises identical original bytes. Unknown enum integers must not be mistaken for a known default through a convenience getter. These differences matter to schema evolution and validation, not just library ergonomics. [Generated merge code][prost-derive]; [dynamic unknown fields][reflect-unknown]; [prost enum mapping][prost-readme].

#### Representative cases and independent expectations

These are proposed future tests, not tests executed in this research:

| Case | Independently expected result | Failure or limitation to expose |
|---|---|---|
| Scalar observation | A known observation retains its stream/feature identity, distinct times, quantity and unit across applicable JSON/Text/Binary/Protobuf representations; present zero differs from absent and declared nil | Numeric truncation, time conflation, implicit-default substitution and missing context |
| Structured result and revision | Fixed array cardinality, ordinary choice and supported nested record meaning survive; old data retains its original schema and unit after a later revision | Descriptor reconstruction losing vector/range/fixed-count semantics; incompatible field reuse; unsupported geometry/choice-array silently accepted |
| Command exchange | Known parameter values reach the same validated command model; authenticated sender, permission checks and command lifecycle remain controlling | Payload spoofing of authority, missing parameters defaulted into actions, or parameter encoding mistaken for status/result support |
| Publication and invalid input | Advertised representation and selected stream identify the right schema and framing; independently generated frames decode to known values | Wrong schema that still parses, concatenated records merged, truncation, excessive allocation, denied schema/data access or unsafe imports |

For the spatial use case that motivated the preceding supplement, an observation's accepted/returned encoding must not change its membership in a filter result. The same logical value and the same selected sampling-feature relationship/geometry must drive filtering, counting and paging. Protobuf does not itself add joins, spatial predicates or historical geometry. [IDR-SRV-059][r059]; [Guide][guide], §4.4.1.

#### Safety conditions

**Recommendations derived from the existing design and parser behavior:**

1. Admit a bounded descriptor bundle, dependencies, root message and associated SWE revision together. Never fetch arbitrary descriptor imports over the network/filesystem or compile uploaded code during a request.
2. Limit input/frame size before allocation, nesting, repeated content, descriptor/import complexity and total work. A recursion guard alone is insufficient; prost's default depth limit is 100, not a general memory budget. [Decoder source][prost-decoder].
3. Validate domain constraints, identifiers, time, nil and units after parsing. A wrong descriptor can successfully interpret tags without preserving meaning.
4. Apply existing authorization to schema discovery, data, commands, errors and publication. Preserving unknown fields does not prove that those fields are safe to forward through field-level disclosure controls.
5. Test malformed/truncated frames and legitimate duplicate-field behavior separately. Do not label all duplicate singular fields invalid when Protobuf specifies merge/last-value behavior. [Encoding rules][proto-encoding].

No security boundary or safety decision should depend on the client selecting a particular serialization.

### 4.4 Q4 — Implementation and verification

**Finding:** A bounded Rust experiment is technically plausible. The principal uncertainty is the interoperable CSAPI mapping, not availability of serialization libraries.

#### What peer code actually demonstrates

**OSH proposal:** `ProtoFormat` registers observation and command bindings, restricts compatible structures and avoids automatic browser/default selection. Schema handlers export a root message type plus a descriptor set. Both outbound and inbound paths exist in source. Generated schemas use fixed Observation/Command names and filenames; envelope fields occupy 1–5 and mapped SWE fields begin at 6. The component called `GeneratedSchemaCache` currently rebuilds its descriptor on each call; its name is not proof of immutable revision binding. [Format][osh-format]; [schema export][osh-obs-schema]; [schema generation][osh-writer]; [cache implementation][osh-cache].

The proposed command schema handler uses `commandFormat`, whereas the approved selector is `cmdFormat`. This is an observable compatibility issue to resolve, not a reason to adopt peer spelling as a standard. Observation output's context-dependent identifiers, incomplete semantic reconstruction, framing and revision identity likewise need explicit treatment before a compatible experiment is promised.

`TestSweProtoWireInterop` reconstructs a receiver descriptor from exported bytes plus runtime well-known types, then parses bytes from the local encoder. That is useful descriptor-driven receiver coverage. It remains a Java test using the same runtime; its comments referring to an external client do not constitute execution of that client. No foreign-language, real HTTP or Part 3 interoperability run was performed in this study. [Peer test][osh-test].

**CS-Go main:** The inspected validation function parses an inline `.proto`, selects its first message and validates an observation result after JSON decoding. Its schema-format e2e test advertises `application/x-protobuf` and retrieves the associated schema, while observation POST remains JSON. The seed documentation explicitly excludes raw binary Protobuf. Thus CS-Go provides meaningful schema/validation precedent, not evidence of the same wire implementation as the OSH proposal. [Validator][go-validation]; [e2e source][go-tests]; [seed documentation][go-seed].

Bounded scans inspected 305 relevant CS-Go files and 2,224 OSH core source/document/build files, and the 5,831-entry add-ons master tree. The identified OSH module was in the proposal, not the inspected add-ons master. These counts describe search coverage, not tested features or exhaustive knowledge of private/development work.

#### Rust options

| Option | Verified version and license | Suitable role | Consequential limitations |
|---|---|---|---|
| Official Google Rust | Protobuf v36.2; `google-protobuf` / `google-protobuf-codegen` 0.36.2-release; BSD-3-Clause; runtime manifest Rust 1.79 | Generated-code implementation using official runtime | Beta; wraps C upb or C++ kernels, not a pure-Rust runtime; native build requirements and exact generated/runtime matching |
| prost | 0.14.4; Apache-2.0; Rust 1.85 | Pure-Rust runtime and generated known messages | No runtime reflection; generated unknown fields are skipped; README calls maintenance passive |
| prost-reflect | Core 0.16.5; MIT OR Apache-2.0; declared Rust 1.82 | DescriptorPool/DynamicMessage for per-stream schemas | Depends on prost 0.14; selected dependency graph imposes its higher Rust requirement; still needs a domain mapping and bounded descriptor admission |

Sources: [official release][google-release], [official Rust README][google-rust], [runtime manifest][google-manifest], [native build][google-build], [version compatibility][google-versions], [prost README][prost-readme], [prost manifest][prost-manifest], [reflect manifest][reflect-manifest], [versioned API][reflect-api]. The separate prost-reflect build helper's 0.16.1 version is not the core version.

v36.2 announces the official crate-name change to `google-protobuf`; older crates/docs using `protobuf` must not be assumed to name that same runtime. None of these findings selects a dependency for Glaux. For a pure-Rust data path with dynamic per-stream schemas, prost plus prost-reflect is a plausible first candidate to evaluate if an experiment is adopted; this is an inference from capability, not a benchmark result.

Normal prost-build uses `protoc`, but `compile_fds` can generate from a previously produced descriptor set without invoking it. Previously generated code or admitted complete descriptors permit runtime operation without a compiler or network schema retrieval. Reproducible offline builds still require pinned dependencies and any chosen toolchains/imports. [prost-build implementation][prost-build]. No software was installed on the project lead's laptop.

#### Minimum independent verification before claiming support

- First compile the selected schema/descriptor fixtures and test their import closure, root identity and per-revision binding; do not treat the defective official examples as an oracle.
- Use expected logical values written independently of the encoder. At least one other implementation must produce/consume actual bytes for any cross-implementation claim.
- Exercise HTTP discovery, selection, read and write separately from bare codec round-trips; include global/per-stream contexts and documented unsupported operations.
- Exercise actual selected Part 3 framing/representation selection separately from HTTP. Protobuf messages are not inherently self-delimiting. [Streaming guidance][proto-framing].
- Cover the four cases in §4.3, both compatible and incompatible schema changes, unknown tags/enums, missing/default/nil values, authorization and resource limits.
- Measure size/CPU only after correctness, using representative data and equivalent semantics. No performance promise is justified by choosing the format alone.

### 4.5 Q5 — Recommendation and reconciliation with prior research

**Finding:** The new evidence qualifies earlier awareness but does not invalidate the existing design or require an additional architecture.

| Existing input | What remains valid | Qualification supplied by this study |
|---|---|---|
| [007][r007], §9.4; [014G][r014g], LL-18 | Protobuf is not an approved Part 2 conformance class; Part 5 was future awareness | There is now substantial public OSH proposal code to assess, not only illustrative files |
| [012][r012], §§6–9 | Schema-format selection and schema-document representation are separate; use controlling media/query rules | Official examples, generic Protobuf guidance and OSH choose different media/schema contracts; `commandFormat` peer selector differs from `cmdFormat` |
| [022][r022], §4.4 | One full logical SWE contract supports distinct codecs | A descriptor is not automatically a lossless replacement for that contract |
| [023][r023], §13.2 | Offline resolution, semantic validation and stable contract binding remain necessary | Descriptor parsing adds a bounded input surface; no separate registry service or old process machinery is required |
| [035][r035], §9 | Native data versus event-envelope distinction; capability-qualified encoding progression | Binary publication remains possible; #190 has a separate representation proposal, not an adopted baseline change |
| [059][r059] / [Guide][guide], §4.4.1 | Filtering acts on logical data and explicitly selected sampling relationships | Protobuf supplies neither better querying nor an automatic Part 4 geometry mapping |
| [Goal v1.7][goal] / [Guide v0.2][guide] | Full approved target and selected experiments remain controlling | Any Protobuf experiment is a new scope choice; this report does not make it |

**Recommendation:** Retain compatibility points already present in the design and defer selection of a Part 5 wire contract. This is not a recommendation to remove existing work, block Guide drafting indefinitely, or wait for final publication if a sufficiently complete versioned draft becomes available. A pinned draft can support an explicitly experimental implementation, just as with other drafts; the present issue is contract completeness and demonstrated mapping, not the word “draft” itself.

## 5. Decision Analysis

| Option | Benefits | Costs / risks | Standards / compatibility impact | Recommendation |
|---|---|---|---|---|
| Preserve existing codec/schema boundaries; select Part 5 binding later | Keeps server planning moving; avoids premature wire choices; uses the existing design | No immediate Protobuf exchange; later implementation still required if adopted | No change to approved obligations or selected experiments | **Recommended now** |
| Adopt a pinned OSH-compatible experimental subset | Concrete source, descriptors and potential peer interoperability target available now | Mapping gaps, unmerged dependencies, parameter/media differences and independent tests; possible upstream rework | Explicit peer-compatible experiment, not general Part 5 conformance | Viable only by separate scope decision |
| Repair official examples and define remaining behavior locally | Small scalar demonstration possible | Leaves interoperability choices to Glaux; risks a misleading claim of exact reference implementation | A Glaux-specific serialization unless another shared contract is agreed | Do not use as the basis for claiming Part 5 support |
| Defer even compatibility consideration | Lowest immediate attention | Misses useful constraints already exposed by the study | Does not violate existing obligations | Unnecessary: retain the existing extension points without building new ones |

No option replaces SWE Binary, automatically adds every draft encoding, requires gRPC, or changes the original approved CSAPI target.

## 6. Key Recommendations

1. **Keep Part 5 implementation out of the completion target for now.** High priority. The missing element is a complete selected binding, not proof that Protobuf is useful. Reconsider when a versioned proposal settles the affected semantics and interactions; final OGC publication need not be a prerequisite for an explicitly experimental choice.
2. **Preserve the existing logical-model/codec separation and immutable stream revisions.** High priority. If later reflected in the Guide, this is a clarification of existing design, not authorization for a schema service, new persistence system or runtime code generation.
3. **Keep the OSH-compatible option explicit.** Medium priority, conditional on the project lead preferring early implementation. Pin both proposal commits; state resource/operation/structure coverage, media/query differences, schema identity, framing and rejection behavior; require independent verification before advertising support.
4. **Do not conflate research acceptance with adoption.** High priority. The next step is an accepted-findings synthesis addendum. Any Goal/Guide change follows the agreed discussion step.

## 7. Implementation Implications and Estimates

### 7.1 Implications

Under the recommended option, no server code is required now. A later Guide clarification can keep the existing normalized model, schema revision and codec interfaces open to a Protobuf adapter, without committing to a dependency or media type. Goal v1.7 need not change merely to record this research finding.

If the alternative experiment is adopted, its incremental work would be:

- Bind descriptors, root types and supported SWE mappings to existing stream contracts; preserve necessary logical metadata separately from representation metadata.
- Add bounded encode/decode adapters and explicit semantic conversion/rejection rules without creating a second observation/command domain model.
- Extend existing format/schema handlers and capability declarations; document peer aliases or deviations instead of changing the approved baseline invisibly.
- Add selected outbound Part 3 representation/framing handling within the existing experiment. HTTP ingestion does not authorize new inbound Pub/Sub scope.
- Add independent positive/negative and cross-format tests to existing verification work. No new test framework or external service is inherently required.

Prospective Guide locations are §§4.3, 6.2 and 6.5 for codec/schema boundaries, §4.8 for publication, and §§7–8 for sequence and verification. §4.4.1 remains encoding-independent. These are proposed touchpoints only; no Goal, Implementation Guide or Roadmap content was edited.

### 7.2 Effort / Complexity Estimate

| Work item | Relative complexity | Estimate | Assumptions |
|---|---|---|---|
| Preserve current compatibility points in drafting | Low | Small drafting clarification, no implementation estimate | Existing domain/codec/revision separation retained |
| Select and document an experimental binding | Medium, with upstream uncertainty | Not estimated in hours | Complete coverage/deviation choices made against pinned material |
| Scalar observation/command codec subset | Medium | Not estimated in hours | Existing CSAPI/domain/schema infrastructure available |
| General composite mapping and historical evolution | High | Not estimated in hours | No silent semantic loss; unsupported cases explicit |
| HTTP/schema and selected publication integration | Medium–high | Not estimated in hours | Framing, identity and advertisement decisions settled |
| Independent cross-language, failure and security verification | Medium–high | Not estimated in hours | Suitable peer/client fixtures and controlled test environment available |

These are qualitative incremental judgments, not delivery commitments or measured effort. The study does not justify a claim that the full addition is “just another codec.”

## 8. Risks, Constraints, and Open Questions

### 8.1 Risks and Constraints

- A compelling implementation can be mistaken for an agreed standard; the report therefore names the OSH proposal and its status explicitly.
- Descriptors can decode successfully while losing units, nil meaning, cardinality, identity or time semantics. Compilation and round-trip tests alone are insufficient.
- Current schema reconstruction and fixed message names do not establish immutable historical revision handling.
- Cross-format filtering or command authority must not change with encoding.
- Native-code Rust options add build/runtime considerations; pure-Rust alternatives still have distinct unknown-field behavior and parser limits.
- No published performance, certification or complete interoperability claim is made. No toolchain, service or dependency was installed.

### 8.2 Open Questions

1. **What versioned Part 5 proposal should be the shared target?** Official public material is incomplete; the OSH author refers to an in-progress draft. A supplied public draft/reference would improve the next decision. The user need not author a technical design.
2. **Which first-release resources, directions and SWE constructs are intended?** The meeting preference does not settle these. Observe the eventual proposal's coverage; do not infer complete observations, commands, statuses and features from a scalar example.
3. **Which media/schema/selector/framing contract will implementations share?** Current official examples, OSH and generic guidance differ. A peer experiment must resolve its own explicitly named compatibility boundary.
4. **How will identity, revision, nil/presence, temporal meaning and unsupported structures be handled?** Retaining a logical SWE contract is necessary design context, but not a substitute for agreed wire rules.
5. **How will representation selection fit the selected Part 3 binding?** #190's separate proposal is useful but not incorporated into the current project pin.
6. **What independent clients and payload fixtures demonstrate interoperability?** The inspected Java descriptor test and CS-Go JSON-schema path do not establish cross-language binary HTTP/Part 3 exchange.

These are bounded implementation-decision inputs, not an indefinite monitoring task or a blocker to completing the present research.

## 9. Validation Against Plan Success Criteria

| Plan success criterion | Validation status | Evidence |
|---|---|---|
| Q1–Q5 answered or limitations explicit | Met | §§2, 4.1–4.5, 8 |
| Source/maturity and meeting/approved distinctions | Met | §§1, 3, 4.1 |
| SWE Binary comparison without replacement/performance assumptions | Met | §4.2 |
| Resource/direction, discovery, framing and binding gaps explicit | Met | §§4.2, 4.4 |
| Meaning, presence/nil, precision/time, evolution, parsing and authorization cases | Met | §4.3 |
| Exact peer/Rust sources and execution limits | Met | §§3.2, 4.4; Appendix A |
| Bounded incremental design/tests and encoding-independent filtering | Met | §§4.3–4.5, 7 |
| Decision-usable alternatives and qualitative costs | Met | §§5–8 |
| Template, reproducible references, history refresh and later handoff | Met | This report; history register v1.15; §10 |

“Met” assesses the research deliverable, not implementation readiness or completion of future tests. Project-lead acceptance remains pending.

## 10. Next Steps and Handoff

1. **Review this report.** Owner: Glaux Project Lead. Timing: next iteration. A plain `proceed` accepts the research for downstream use and authorizes its separate final-synthesis addendum under the established workflow.
2. **Prepare that addendum only after authorization.** Owner: Glaux research workflow. Preserve the original synthesis and existing addenda; summarize the evidence, recommended deferral of binding selection, and conditional OSH-compatible option.
3. **Discuss any Goal/Guide decision next.** Owner: Glaux Project Lead with drafting assistance. Research acceptance does not adopt an experiment. No extra approval vocabulary is required.
4. **Resume Guide drafting pass 2 after that discussion/any agreed update.** No new implementation, research topic, publication schedule or other encoding is authorized by this report.

No further input is needed to complete this research/report iteration. A later draft link or author-supplied reference would be helpful, not mandatory.

## 11. References

All external code references below use the inspected commit; standards and documentation references use the dated retrieval in §3. Issues and PR states are point-in-time observations. Local research links refer to accepted inputs; this report itself remains in review.

- Approved comparison: [CSAPI Part 2][part2], [SWE Common 3.0][swe], [published source][ogc-tag].
- Official artifacts: [README][ogc-readme], [examples][ogc-examples], [experiments][ogc-experiments], [observation schema wrapper][obs-wrapper], [command schema wrapper][cmd-wrapper], [scalar][scalar-example], [GeoPose reference variant][geopose-ref], [PTZ][ptz-example], [SWE options][swe-options], [request][obs-request], [response][obs-response].
- History: [#144 comment][issue144-comment], [#21][issue21], [#190][issue190], [#190 comment][issue190-comment], [representation offering][offering], [selection][selection], [shared register][history].
- Peer implementation: [OSH add-on PR][osh-pr], [core PR][osh-core-pr], [format][osh-format], [schema export][osh-obs-schema], [command selector][osh-cmd-handler], [observation binding][osh-obs-binding], [command binding][osh-cmd-binding], [writer][osh-writer], [reader][osh-reader], [cache][osh-cache], [test][osh-test]; [CS-Go validator][go-validation], [tests][go-tests], [seed notes][go-seed].
- Protobuf mechanics: [proto3/evolution][proto-evolution], [presence][proto-presence], [encoding][proto-encoding], [Timestamp][proto-timestamp], [framing][proto-framing], [noncanonical serialization][proto-canonical], [media types][proto-media], [ProtoJSON][proto-json].
- Rust: [Google release][google-release], [runtime README][google-rust], [manifest][google-manifest], [build][google-build], [version matching][google-versions]; [prost README][prost-readme], [manifest][prost-manifest], [derive][prost-derive], [decoder][prost-decoder], [build configuration][prost-build]; [prost-reflect manifest][reflect-manifest], [API][reflect-api], [unknown fields][reflect-unknown].
- Project reconciliation: [007][r007], [012][r012], [014G][r014g], [022][r022], [023][r023], [035][r035], [059][r059], [Goal][goal], [Guide][guide].

## 12. Appendices

### Appendix A — Executed checks and limitations

1. **Source inventory:** Enumerated public official refs and inspected pinned trees/text and all seven official `.proto` files. No Part 5-named branch, dedicated directory or complete requirements/ATS baseline was identified within that search. This is not a statement about private or unpublished work.
2. **JSON syntax:** Parsed the two Protobuf schema wrappers and scalar wrapper example successfully: three JSON syntax checks, not JSON Schema validation or proof of a usable `.proto`.
3. **Static duplicate-field diagnostic:** Extracted nine declarations from `observationSchema-geopose.withrefs.proto`; duplicate tags were `[1, 2, 3]`. File blob: `fa05527327052cd3ae9a8566c4ddf4ab64b41294`. No compiler was invoked.
4. **Numerical diagnostics:** Node v26.9.0 assertions confirmed `Math.fround(16777217) === 16777216` and `BigInt(Number(9007199254740993n)) !== 9007199254740993n`. These demonstrate precision-loss hazards in possible mappings, not defects unique to Protobuf or execution of a codec.
5. **Proposal ancestry:** The #190 representation branch was not an ancestor of either inspected master or Part 3 working draft (`git merge-base --is-ancestor` exit 1). The commit-to-PR lookup returned no associated PR at inspection.
6. **Tool availability:** No `protoc`, `buf`, `cargo` or `rustc` was found on PATH. No Rust build, Protobuf compilation, wire codec test, peer server exchange or benchmark was run. Nothing was installed.
7. **Review:** Official maturity, peer implementation, and Rust/semantic/security findings received separate read-only source reviews. Such reviews do not replace project-lead acceptance or runtime testing.

### Report Completion Checklist

- [x] Topic ID matches overall research plan index
- [x] Topic research plan is linked and aligned
- [x] Core questions covered or explicitly unresolved
- [x] Findings have reproducible references
- [x] Normative and informative evidence distinguished
- [x] Mutable sources pinned or dated
- [x] Evidence limitations explicit
- [x] Findings, inference and recommendations distinguished
- [x] Prior-report conclusions reconciled
- [x] Executive summary independently readable
- [x] Recommendations explicit and bounded
- [x] Risks and open questions documented
- [x] Success criteria assessed
- [ ] Plan-owner acceptance and date recorded before downstream completion
- [x] Next steps assigned

[part2]: https://docs.ogc.org/is/23-002/23-002.html
[swe]: https://docs.ogc.org/is/24-014/24-014.html
[ogc-tree]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f
[ogc-tag]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2
[ogc-readme]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f/README.md#L64
[ogc-examples]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f/api/part2/openapi/examples/schemas
[ogc-experiments]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f/swecommon/experiments
[obs-wrapper]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f/api/part2/openapi/schemas/json/observationSchemaProtobuf.json
[cmd-wrapper]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f/api/part2/openapi/schemas/json/commandSchemaProtobuf.json
[scalar-example]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f/api/part2/openapi/examples/schemas/observationSchema-scalar.proto
[geopose-ref]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f/api/part2/openapi/examples/schemas/observationSchema-geopose.withrefs.proto
[ptz-example]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f/api/part2/openapi/examples/schemas/commandSchema-ptz.proto
[swe-options]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f/swecommon/experiments/protobuf/swe_options.proto
[obs-request]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f/api/part2/openapi/requests/observationOrArray.yaml
[obs-response]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f/api/part2/openapi/responses/observationCollection.yaml
[part3-tree]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/6f529a15bfa63259febc3620378d3e5a06305333/api/part3
[selection-tree]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/6f5987673dd11a8e27a2b6093e2dc5cbd0d16612/api/part3
[offering]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/6f5987673dd11a8e27a2b6093e2dc5cbd0d16612/api/part3/standard/requirements/data/req_representation_offering.adoc
[selection]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/6f5987673dd11a8e27a2b6093e2dc5cbd0d16612/api/part3/standard/requirements/data/req_representation_selection.adoc
[issue21]: https://github.com/opengeospatial/ogcapi-connected-systems/issues/21
[issue144-comment]: https://github.com/opengeospatial/ogcapi-connected-systems/issues/144#issuecomment-2734726411
[issue190]: https://github.com/opengeospatial/ogcapi-connected-systems/issues/190
[issue190-comment]: https://github.com/opengeospatial/ogcapi-connected-systems/issues/190#issuecomment-4952633856
[osh-pr]: https://github.com/opensensorhub/osh-addons/pull/224
[osh-core-pr]: https://github.com/opensensorhub/osh-core/pull/354
[osh-core-main]: https://github.com/opensensorhub/osh-core/tree/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08
[osh-addons-main]: https://github.com/opensensorhub/osh-addons/tree/9682ce901cafe2d532f5700294176f895e823481
[osh-format]: https://github.com/opensensorhub/osh-addons/blob/22d98d16c77e1e224365fd458dfc3ba77205f78e/services/sensorhub-service-consys-proto/src/main/java/org/sensorhub/impl/service/consys/proto/ProtoFormat.java
[osh-obs-schema]: https://github.com/opensensorhub/osh-addons/blob/22d98d16c77e1e224365fd458dfc3ba77205f78e/services/sensorhub-service-consys-proto/src/main/java/org/sensorhub/impl/service/consys/proto/datastreams/DataStreamSchemaBindingProto.java
[osh-cmd-handler]: https://github.com/opensensorhub/osh-core/blob/6988ded63679fc55b8741f92408a4d883e245d1e/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/task/CommandStreamSchemaHandler.java
[osh-obs-binding]: https://github.com/opensensorhub/osh-addons/blob/22d98d16c77e1e224365fd458dfc3ba77205f78e/services/sensorhub-service-consys-proto/src/main/java/org/sensorhub/impl/service/consys/proto/observations/ObsBindingProto.java
[osh-cmd-binding]: https://github.com/opensensorhub/osh-addons/blob/22d98d16c77e1e224365fd458dfc3ba77205f78e/services/sensorhub-service-consys-proto/src/main/java/org/sensorhub/impl/service/consys/proto/commands/CommandBindingProto.java
[osh-writer]: https://github.com/opensensorhub/osh-addons/blob/22d98d16c77e1e224365fd458dfc3ba77205f78e/services/sensorhub-service-consys-proto/src/main/java/org/sensorhub/impl/service/consys/proto/schema/ProtoSchemaWriter.java
[osh-reader]: https://github.com/opensensorhub/osh-addons/blob/22d98d16c77e1e224365fd458dfc3ba77205f78e/services/sensorhub-service-consys-proto/src/main/java/org/sensorhub/impl/service/consys/proto/schema/ProtoSchemaReader.java
[osh-decoder]: https://github.com/opensensorhub/osh-addons/blob/22d98d16c77e1e224365fd458dfc3ba77205f78e/services/sensorhub-service-consys-proto/src/main/java/org/sensorhub/impl/service/consys/proto/codec/ProtoDecoder.java#L311
[osh-cache]: https://github.com/opensensorhub/osh-addons/blob/22d98d16c77e1e224365fd458dfc3ba77205f78e/services/sensorhub-service-consys-proto/src/main/java/org/sensorhub/impl/service/consys/proto/schema/GeneratedSchemaCache.java
[osh-test]: https://github.com/opensensorhub/osh-addons/blob/22d98d16c77e1e224365fd458dfc3ba77205f78e/services/sensorhub-service-consys-proto/src/test/java/org/sensorhub/impl/service/consys/proto/codec/TestSweProtoWireInterop.java
[go-tree]: https://github.com/SomethingCreativeStudios/connected-systems-go/tree/b1fd2e0e9bd69e222d05258d659a842ca24502cb
[go-validation]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/resourcevalidation/observation.go
[go-tests]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/e2e/stream_schema_formats_test.go
[go-seed]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/scripts/seed-connected-systems/README.md
[protobuf-docs]: https://protobuf.dev/
[proto-presence]: https://protobuf.dev/programming-guides/field_presence/
[proto-encoding]: https://protobuf.dev/programming-guides/encoding/
[proto-timestamp]: https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp
[proto-evolution]: https://protobuf.dev/programming-guides/proto3/#updating
[proto-framing]: https://protobuf.dev/programming-guides/techniques/#streaming-multiple-messages
[proto-canonical]: https://protobuf.dev/programming-guides/serialization-not-canonical/
[proto-media]: https://protobuf.dev/reference/protobuf/mime-types/
[proto-json]: https://protobuf.dev/programming-guides/json/
[google-release]: https://github.com/protocolbuffers/protobuf/releases/tag/v36.2
[google-rust]: https://github.com/protocolbuffers/protobuf/blob/2c74169b34066ceb8ddb6b882fcb3fb32d737a55/rust/release_crates/google_protobuf/README.md
[google-manifest]: https://github.com/protocolbuffers/protobuf/blob/2c74169b34066ceb8ddb6b882fcb3fb32d737a55/rust/release_crates/google_protobuf/Cargo-template.toml
[google-build]: https://github.com/protocolbuffers/protobuf/blob/2c74169b34066ceb8ddb6b882fcb3fb32d737a55/rust/release_crates/google_protobuf/build.rs
[google-versions]: https://protobuf.dev/support/cross-version-runtime-guarantee/
[prost-readme]: https://github.com/tokio-rs/prost/blob/13646cde7eab75c81b3047767aa0a86e7dbecf12/README.md
[prost-manifest]: https://github.com/tokio-rs/prost/blob/13646cde7eab75c81b3047767aa0a86e7dbecf12/Cargo.toml
[prost-derive]: https://github.com/tokio-rs/prost/blob/13646cde7eab75c81b3047767aa0a86e7dbecf12/prost-derive/src/lib.rs#L191
[prost-decoder]: https://github.com/tokio-rs/prost/blob/13646cde7eab75c81b3047767aa0a86e7dbecf12/prost/src/encoding.rs
[prost-build]: https://github.com/tokio-rs/prost/blob/13646cde7eab75c81b3047767aa0a86e7dbecf12/prost-build/src/config.rs#L826
[reflect-manifest]: https://github.com/andrewhickman/prost-reflect/blob/2d0e1847a07d2582ae3edf84dbb80f2684961fef/prost-reflect/Cargo.toml
[reflect-api]: https://docs.rs/prost-reflect/0.16.5/prost_reflect/
[reflect-unknown]: https://docs.rs/prost-reflect/0.16.5/prost_reflect/struct.DynamicMessage.html#method.unknown_fields
[history]: ../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md
[goal]: ../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md
[guide]: ../../../../../Plans/glaux-server/glaux-server-implementation-guide.md
[r007]: idr-srv-007-csapi-part-2-requirement-baseline-report.md
[r012]: idr-srv-012-content-negotiation-media-types-and-encoding-selection-report.md
[r014g]: idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md
[r022]: idr-srv-022-swe-common-data-component-strategy-report.md
[r023]: idr-srv-023-schema-and-encoding-validation-strategy-report.md
[r035]: idr-srv-035-streaming-and-event-publication-strategy-report.md
[r059]: idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md

# Section 022: SWE Common Data Component Strategy - Research Report

**Topic ID:** IDR-SRV-022<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-022 SWE Common Data Component Strategy](../IDR%20Plans/idr-srv-022-swe-common-data-component-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions, all detailed-question groups, all 6 methodology phases, and all 11 success criteria<br>
**Methodology Used:** Authority-ranked extraction from SWE Common 3.0 conceptual models, JSON schemas, encoding rules, examples, and abstract tests; direct mapping against approved CSAPI Parts 1 and 2, SensorML 3.0, tagged schemas, examples, and tests; accepted AEP/STANAG and IDR-SRV-001 through IDR-SRV-021 carry-forward; bounded refresh of SWE-owned official-repository history; and component, representation, validation, persistence, security, fixture, and interoperability analysis<br>
**Research Time:** Approximately 11 hours of AI-assisted execution on September 14, 2026<br>
**Official Standards Source Pin:** [`opengeospatial/ogcapi-connected-systems` `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**SWE Common Edition:** OGC 24-014, Version 3.0.0, approved June 2, 2025 and published July 16, 2025<br>
**Shared Register Baseline:** OGC API - Connected Systems upstream-history register Version 1.9; SWE-owned entries, published schemas, and official `master` rechecked September 14, 2026 with no material register change required<br>
**Document Purpose:** Establish the server-side SWE Common component, contract, encoding, normalization, validation, security, fixture, and downstream-design baseline for the Rust Glaux reference server<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 14, 2026<br>
**Date:** September 14, 2026<br>
**Last Updated:** September 14, 2026

---

## Reading Guide and Evidence Labels

| Label | Meaning |
|---|---|
| **N** | Normative or standards-derived finding from an approved applicable source or incorporated artifact |
| **A** | Project-controlling AEP/STANAG adoption or operational-context finding carried from an accepted report |
| **P** | Glaux project decision or recommendation proposed for acceptance here |
| **I** | Informative implementation, test, interoperability, or community evidence |
| **D** | Official draft or post-publication direction that is not an approved requirement |
| **X** | Published ambiguity, artifact conflict, evidence gap, or unresolved project decision |

“SWE component description,” “encoded value block,” “CSAPI resource,” “canonical domain state,” and “validation evidence” are not synonyms. SWE Common describes the representation, semantics, structure, quality, and encoding of data. It does not replace CSAPI resource identity and lifecycle, observation or command records, authorization, persistence, or policy.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. SWE Common Extraction Methodology
5. SWE Common Concept Inventory
6. SWE Common-to-CSAPI / Glaux Resource Mapping
7. Data Component Type Support Strategy
8. Encoding and Representation Boundary Findings
9. Observation, Datastream, Status, and Dynamic-Data Findings
10. Command, Control Stream, and Feasibility Findings
11. Units, Observed Properties, Controlled Properties, and Semantic Dependency Findings
12. Validation, Profile, Schema, Encoding, and Conformance Implications
13. Security, Policy, and Releasability Implications
14. Fixture, Golden-File, and Interoperability Test Implications
15. Downstream Topic Handoff Matrix
16. Recommendations
17. Risks, Constraints, and Open Questions
18. Validation Against Plan Success Criteria
19. References

---

## 1. Executive Summary

Glaux should implement SWE Common 3.0 as a **versioned data-contract subsystem over the canonical resource graph**, not as the whole domain model and not as schema-shaped JSON accepted without interpretation. A contract consists of an encoding-neutral component tree, a separate encoding descriptor, an exact source artifact, a compiled validation/codec plan, provenance and policy metadata, and an immutable fingerprint. A DataStream or ControlStream revision binds values to one exact contract revision; historical Observations and Commands never silently rebind to the stream's newest schema. [N,P; IDR-SRV-015–019,021]

The component model should cover the published SWE Common 3.0 families: Boolean, Text, Category, Count, Quantity, Time; CategoryRange, CountRange, QuantityRange, TimeRange; DataRecord, Vector, DataChoice, DataArray, Matrix, Geometry; and the top-level SWE DataStream descriptor. It must also preserve common metadata, `definition`, label/description, `optional`, `updatable`, reference frames, axis identifiers, units, code spaces, constraints, nil mappings, quality, element counts, names, associations, and unknown extensions. Scalar-list proposal types are not part of SWE Common 3.0; a DataArray with a scalar `elementType` is the interoperable representation. [N,D]

Initial public support should be deliberately asymmetric:

- fully parse, preserve, inspect, normalize, and structurally validate every published component family;
- compile and execute JSON value codecs for the full component tree, including compact record/vector forms, choice selection, ranges, variable arrays, matrices, and Geometry;
- implement SWE Text next with exact optional-field, separator, block, choice, array-count, and WKT rules;
- preserve BinaryEncoding immediately but advertise and execute SWE Binary only after bounded length, datatype, reference, byte-order, base64/raw, compression/encryption, and WKB test coverage is complete; and
- treat XML as legacy/import material, not SWE Common 3.0 conformance. [N,P]

This is a capability-gated strategy: accepting a component definition is not the same as claiming every value encoding for it. A machine-readable capability matrix must state, by component family, operation, direction, and media type, whether Glaux can preserve, parse, validate, ingest, emit, convert, and round-trip it. Unsupported executable encodings fail explicitly; they are not decoded heuristically or advertised. [P]

For CSAPI Part 2, the schema resource wrapper keeps `recordSchema` and `encoding` separate and selects the value format through `obsFormat` or `commandFormat`. SWE JSON observation schemas require at least one Time component mapped to phenomenon or result time. Optional SamplingFeature references use Text and the prescribed SOSA definition. Command schemas may map Time to issue time and Text to SamplingFeature. Observation and Command payloads must then conform to the exact parent DataStream or ControlStream schema. [N]

The server must distinguish five value states: component omitted because `optional=true`; a present declared nil sentinel with a nil-reason URI; a present ordinary value; a structurally invalid value; and an operationally rejected value. JSON `null` is not an automatic SWE nil value. Likewise `NaN`, `-Infinity`, and `+Infinity` are standardized strings for Quantity and Time values, while Count remains integer. Range values are exactly two-element JSON arrays. Constraints limit the component's value but do not grant command authorization, establish device safety, or convert a capability into current status. [N,P]

Names and order are contract-bearing. DataRecord field and DataChoice item names are unique component-path segments; Vector coordinates have axis bindings. Child order determines text, binary, and compact-array encoding. Glaux should compile a typed `ComponentPath` rather than use JSON Pointer as if it were the standard reference grammar. The published JSON prose contains a singular/plural typo around `vectorAsArrays` versus schema property `vectorsAsArrays`; the published schema and UML property control. [N,X,P]

Imported SWE definitions remain untrusted. The accepted five-layer SensorML architecture extends naturally to SWE: exact bytes; parsed component graph; canonical normalized contract projection; authorized generated schema/view; and validation/transformation evidence. Strict public registration rejects invalid or unsupported contracts. A privileged quarantine path can preserve invalid, partial, legacy, external, or profile-divergent material without making it active or publicly conformant. [P; IDR-SRV-021]

Security is not supplied by SWE Common—the standard records no security considerations. Component definitions can reveal sensor capabilities, command affordances, operating ranges, vocabularies, precision, position frames, and nil/error semantics. Values may be classified or operationally sensitive. URI resolution, regex constraints, recursive structures, variable arrays, binary lengths, compression, encryption declarations, and data URIs create SSRF, resource-exhaustion, algorithm-confusion, and disclosure risks. Policy must apply independently to exact source, normalized indexes, schema endpoints, values, validation errors, caches, logs, and generated views. [N,A,P]

Acceptance should establish the contract architecture, component coverage, phased codec posture, mapping rules, non-collapse rules, validation ladder, security posture, fixtures, and downstream handoffs. It does not select Rust crates, database products, semantic registries, command lifecycle rules, time-series tables, or authentication policy; advertise SWE Text/Binary before implementation; implement draft Part 3; or begin the server.

---

## 2. Scope and Plan Alignment

### 2.1 Completed Scope

This report:

- inventories all SWE Common 3.0 component and encoding families relevant to Glaux;
- maps them to SensorML I/O and CSAPI DataStream, Observation, status, ControlStream, Command, feasibility, and result seams;
- separates component descriptions, value blocks, API resources, canonical fields, source artifacts, indexes, compiled contracts, and validation evidence;
- selects full model/preservation coverage and staged, claim-gated runtime codec coverage;
- defines nil, optional, constraint, range, quality, order, component-path, association, and schema-revision behavior;
- identifies unit, observed-property, controlled-property, code-space, reference-frame, and semantic dependencies without finalizing IDR-SRV-024;
- identifies validation mechanisms and schema conflicts for IDR-SRV-023 without selecting its stack;
- incorporates accepted AEP context and OSH, CS-Go, pygeoapi, SECD, client, interoperability, and community findings as nonnormative evidence; and
- defines security, fixture, conformance, and downstream handoffs.

### 2.2 Excluded Decisions

This report does not:

- make SWE Common the Glaux aggregate/resource model or persistence schema;
- select validator, parser, codec, ontology, unit, database, queue, or object-store products;
- finalize semantic equivalence, controlled vocabulary, UCUM, resolver, or conversion policy (IDR-SRV-024);
- design time-series partitioning, indexes, retention, ingestion transactions, or schema migration (IDR-SRV-025–031);
- define current/latest observation semantics, streaming protocols, or draft Part 3 payloads (IDR-SRV-034–035);
- define command, status, result, feasibility, authorization, or safety state machines (IDR-SRV-036–038);
- define authentication, disclosure, releasability, or audit mechanisms (IDR-SRV-039–041);
- promise legacy SWE 2.0 XML output; or
- implement the server.

### 2.3 Research Question Coverage

| Plan question group | Status | Evidence |
|---|---|---|
| SWE source and concept baseline | Complete | §§3–5 |
| Component types and support scope | Complete | §7 |
| CSAPI, SensorML, and Glaux mapping | Complete | §§6, 9–11 |
| Representation, normalization, storage, and query boundaries | Complete | §§6, 8–9 |
| JSON, text, binary, arrays, records, and streaming implications | Complete | §8 |
| Nil, constraint, quality, unit, and semantic behavior | Complete with assigned semantic detail | §§7, 11 |
| Observation, status, command, and feasibility dependencies | Complete with owning lifecycles deferred | §§9–10 |
| Import, generation, transformation, partial, legacy behavior | Complete | §§8, 12 |
| Validation, security, conformance, fixtures, interoperability | Complete as requirements/handoffs | §§12–14 |

---

## 3. Evidence Base and Authority Classification

### 3.1 Primary Sources

| Source | Version/pin | Role | Accessed | Authority and limitation |
|---|---|---|---|---|
| [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html) | OGC 24-014, 3.0.0; approved 2025-06-02, published 2025-07-16 | Core concepts, UML, JSON implementation, text/JSON/binary value rules, ATS | 2026-09-14 | Controlling [N] |
| [Official SWE Common schemas](https://schemas.opengis.net/sweCommon/3.0/json/) | Versioned 3.0 registry | JSON Schema 2020-12 component and encoding artifacts | 2026-09-14 | Normative-support artifact; schema validity is not semantic, value-codec, policy, or CSAPI validity [N,X] |
| [CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) | OGC 23-001, 1.0 | Resources, SensorML representation, Property and link context | 2026-09-14 | Controlling for API-resource boundary [N] |
| [CSAPI Part 2](https://docs.ogc.org/is/23-002/23-002.html) | OGC 23-002, 1.0 | DataStream/Observation, status, ControlStream/Command, feasibility, schemas, encodings | 2026-09-14 | Controlling; residual editorial/schema issues retained [N,X] |
| [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) | OGC 23-000, 3.0 | Process input/output/parameter/capability/characteristic use of SWE | 2026-09-14 | Controlling dependency [N] |
| [Tagged CSAPI source](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2) | `v1.0.0`, `8e03b236...` | Reproducible SWE schemas, standard source, OAS, examples, ATS | 2026-09-14 | Publication pin [N,X] |
| [SSN/SOSA](https://www.w3.org/TR/vocab-ssn/) | W3C/OGC Recommendation | Observed/controlled-property and observation semantic cross-check | 2026-09-14 | Supporting; does not add representation members [N/I] |

The SWE specification distinguishes a JSON implementation of component descriptions from JSON/Text/Binary encoding rules for values. SWE Common 3.0 supersedes 2.0, adds normative JSON artifacts and Geometry, and removes XML from the current edition; it is not fully backward compatible with SWE 2.x XML. [N]

### 3.2 Controlled AEP/STANAG Boundary

The controlled project source remains NATO package `AC/224(JCGISR)D(2026)0005`, dated April 27, 2026, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`. This report neither redistributes nor newly quotes it. Accepted IDR-SRV-001 through 003 establish that AEP-4789 Volume II adopts CSAPI Parts 1 and 2, SensorML 3.0, and SWE Common 3.0 as a coherent technical package. Those findings justify robust support, provenance, validation, policy, and DDIL-aware behavior; they do not mandate every optional SWE conformance class, invent a NATO-specific component type, or make imported definitions trusted. [A]

### 3.3 Accepted Project Baselines

| Accepted source | Controlling carry-forward |
|---|---|
| IDR-SRV-003/004 | Part 2 owns dynamic resources; SensorML owns rich process descriptions; SWE owns reusable data structure and encoding; terms remain qualified |
| IDR-SRV-008/012/014 | Claims and media types are capability-derived; approved final media contracts control; vendored schemas/tools are pinned and unmodified |
| IDR-SRV-015 | One encoding-neutral resource graph; schemas and representation objects are support artifacts, not competing canonical entities |
| IDR-SRV-016/017 | Typed IDs and relationships; component paths and external references cannot be collapsed into resource IDs or generic strings |
| IDR-SRV-018 | Phenomenon, result, valid, transaction, issue, execution, evaluation, and freshness times remain separate |
| IDR-SRV-019 | Exact source, transformation lineage, scoped quality, and trust evidence remain attributable and versioned |
| IDR-SRV-020 | Status observations, capability availability, stream `live`, command status, feasibility, events, and transport state do not collapse |
| IDR-SRV-021 | Five-layer exact/parsed/canonical/generated/evidence architecture; strict writes versus quarantine imports; explicit I/O-name and profile seams |

### 3.4 Implementation and Interoperability Evidence

- OSH demonstrates broad typed SWE support, observation stores, schema handlers, JSON/OM/SWE bindings, and live delivery. Open issue #284 records failure inserting nested variable-size DataArrays; OSH's `application/swe+csv` use also differs from the accepted SWE text media-type baseline. These are strong complex-array and media-contract test inputs, not normative restrictions. [I]
- CS-Go demonstrates strict request decoding, JSONB preservation, persistent stream/command schemas, and strong E2E patterns. The studied release implements JSON dynamic-resource formatting but did not establish SWE text/binary value codecs; fixed conformance output must not be treated as capability proof. [I]
- The pygeoapi/52°North PoC confirms provider and external-schema validation patterns but also representation-dependent holdings. Glaux must keep one contract/resource population across representations. [I]
- SECD exposes rich DataRecord, Quantity, Category, and DataChoice-like structures, but lacks declared SWE encodings and contains a DataChoice shape rejected by the tested recursive client. It is valuable negative/variant fixture evidence, not a SWE profile. [I]
- OS4CSAPI smoke tests show narrow clients can reject standards-permitted Vector/DataArray and other complex components. Glaux should publish an honest support matrix and test external clients without narrowing standards-correct server output to one client model. [I]
- Community evidence reinforces recursive-schema-capable tools, immutable pins, semantic round-trip checks, and separation of schema validity from tool support. [I]

### 3.5 Official History Refresh

Official `master` remained `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f` on September 14, 2026. The following SWE-owned history was rechecked against GitHub; issue states did not materially change the shared Version 1.9 register.

| Evidence | Current state | Decision consequence |
|---|---|---|
| [#5](https://github.com/opengeospatial/ogcapi-connected-systems/issues/5), [#11](https://github.com/opengeospatial/ogcapi-connected-systems/issues/11), [#46](https://github.com/opengeospatial/ogcapi-connected-systems/issues/46), [#154](https://github.com/opengeospatial/ogcapi-connected-systems/issues/154) | Closed/published | Geometry, JSON implementation, 3.0 refactor/XML removal, and publication are part of the approved baseline |
| [#6](https://github.com/opengeospatial/ogcapi-connected-systems/issues/6) | Open; reviewed July 2026 | No scalar-list types; use scalar-element DataArray and retain an explicit future-change seam |
| [#15](https://github.com/opengeospatial/ogcapi-connected-systems/issues/15) | Open; published outcome identified | Ranges are exactly two-value arrays; open discussion does not override publication |
| [#16](https://github.com/opengeospatial/ogcapi-connected-systems/issues/16), [#17](https://github.com/opengeospatial/ogcapi-connected-systems/issues/17) | Closed | Quantity/Time special numeric strings; Count remains integer |
| [#18](https://github.com/opengeospatial/ogcapi-connected-systems/issues/18) | Closed | Constraint regex follows ECMA-262 behavior |
| [#19](https://github.com/opengeospatial/ogcapi-connected-systems/issues/19) | Closed | Component-name reference paths were retained; do not substitute JSON Pointer semantics |
| [#20](https://github.com/opengeospatial/ogcapi-connected-systems/issues/20), [#71](https://github.com/opengeospatial/ogcapi-connected-systems/issues/71) | Closed | JSON object/array choices are controlled by `recordsAsArrays` and `vectorsAsArrays`; no generic compact flag |
| [#40](https://github.com/opengeospatial/ogcapi-connected-systems/issues/40), [#73](https://github.com/opengeospatial/ogcapi-connected-systems/issues/73), [#74](https://github.com/opengeospatial/ogcapi-connected-systems/issues/74) | Closed | Unit reference supports code and/or URI; detailed semantic equivalence remains IDR-SRV-024 |
| [#55](https://github.com/opengeospatial/ogcapi-connected-systems/issues/55) | Closed | SWE value-encoding rules remain in the main standard |
| [#98](https://github.com/opengeospatial/ogcapi-connected-systems/issues/98), [#100](https://github.com/opengeospatial/ogcapi-connected-systems/issues/100), [#105](https://github.com/opengeospatial/ogcapi-connected-systems/issues/105), [#106](https://github.com/opengeospatial/ogcapi-connected-systems/issues/106) | Closed | Corrected/published DataStream, inline-value, XML, and trajectory handling informs variant fixtures |
| [#144](https://github.com/opengeospatial/ogcapi-connected-systems/issues/144) | Open; reviewed July 2026 | Encoding stays outside reusable record/vector definitions; containing CSAPI schema wrapper associates it |
| [#180](https://github.com/opengeospatial/ogcapi-connected-systems/issues/180) | Closed May 2026 | `name` belongs to the containing soft-named field/item/coordinate association, not every standalone component |

Four published artifact seams are retained: CSAPI Part 2 source still contains pre-publication draft links/notes; its binary clause contains a “Text encoding” editorial typo; SWE JSON encoding prose uses singular `vectorAsArrays` once while the model/schema use `vectorsAsArrays`; and the published SWE JSON schema still defines/references `XMLEncoding` even though SWE Common 3.0 removed XML from current conformance. The tagged schema/model controls executable spelling where unambiguous, while XML remains legacy-only and tests preserve the variants as negative or interpretation fixtures. [N,X]

The conceptual model permits one or more quality components on a simple component, but the published `AbstractSimpleComponent.json` does not declare a `quality` property. Glaux must not discard schema-permitted input solely because generic JSON Schema cannot validate that conceptual member; IDR-SRV-023 must record the gap and a Glaux profile/schema overlay must validate any supported JSON form without altering vendored OGC bytes. [N,X,P]

---

## 4. SWE Common Extraction Methodology

### 4.1 Extraction Unit and Classification

Each component or cross-cutting concept was recorded with the 14 fields required by the plan: source anchor, authority, resource family, representation, normalization, encoding, unit/semantic dependency, validation, persistence/query, security/policy, test, downstream owner, and unresolved notes. Findings were classified as normative, inherited, AEP-specific, representation-specific, implementation evidence, optional, deferred, or unresolved.

### 4.2 Evidence Resolution Rules

1. Approved standard requirements and incorporated artifacts control.
2. SWE conceptual meaning and value-encoding rules are evaluated separately from the JSON schema for component-description documents.
3. CSAPI wrapper and mapping requirements specialize SWE for Observation and Command resources.
4. Accepted Glaux reports control project architecture unless this report explicitly supersedes them; no conflict was found.
5. Controlled AEP findings are used only through accepted reports.
6. Implementations and examples generate design/test evidence, never new requirements.
7. Prose/schema conflicts remain named adapter/test seams; vendored standards artifacts are never silently edited.

### 4.3 Boundary Questions Applied

For each concept, the analysis asked whether Glaux must preserve exact source, parse it, normalize it, index it, compile it, execute it as a codec, expose it, generate it, transform it, reject it, quarantine it, or policy-filter it. Those verbs are independent. “Supported” without this operation and direction is too ambiguous for design or conformance.

### 4.4 Contract Compilation Model

The proposed logical pipeline is:

`source artifact -> parsed SWE graph -> normalized contract revision -> compiled validation/codec plan -> authorized schema/value view`

Compilation resolves local component paths, verifies unique names and references, computes ordered leaves and shapes, distinguishes fixed/variable cardinality, resolves inherited frames, attaches definitions/units/constraints/nil mappings, binds the selected encoding, records supported algorithms, and produces a stable fingerprint. It does not dereference arbitrary semantic or schema URIs on a request thread.

---

## 5. SWE Common Concept Inventory

### 5.1 Core Concepts

| Concept | Normative meaning | Glaux consequence |
|---|---|---|
| Data representation | Boolean, categorical, continuous numerical, discrete countable, and textual representation | Use explicit component kinds; do not infer kind only from a JSON primitive |
| Nature/semantics | `definition` links the represented property to a formal concept | Store source URI and resolution evidence separately; final governance is IDR-SRV-024 |
| Structure | Scalars/ranges compose recursively into records, vectors, choices, arrays, and matrices | Recursive typed graph with budgets; no flat-column assumption |
| Descriptor vs container | A component may describe out-of-band values or contain inline/default values | Record role explicitly; an inline `value` in a schema is not automatically an observation |
| Encoding | Structure and encoding are separable; values may be JSON, text, or binary | One component tree can have several format-specific contracts |
| Constraint | Allowed values, intervals, significant figures, token values/patterns, or times constrain that component value only | Compile typed predicates; never interpret as cross-field policy or command authorization |
| Nil | `nilValues` maps a reserved value to a reason when the real value is unavailable | Preserve sentinel and reason; distinguish from omission/null/error |
| Quality | Scalar/range qualitative information may be static metadata or a dynamic data field | Scope quality to component/value and provenance; preserve JSON artifact gap |
| Optional | Descriptor flag permitting omission from a data stream; default false | Presence bitmap/Y-N token behavior is encoding-specific |
| Updatable | Meaningful for process/service/sensor inputs, not dataset contents; default false | Descriptive mutability hint only; never an authorization grant |
| Reference frame/axis | Anchors projected temporal/spatial quantities and coordinates | Preserve identifiers/inheritance; validate axis uniqueness and defer registry policy |

### 5.2 Component Families

| Family | Members and key rules | Initial model status |
|---|---|---|
| Scalar | Boolean; Text; Category with optional code space but constraint required if absent; integer Count; decimal Quantity with mandatory unit; Time with unit and UTC-default/frame rules | Full typed support |
| Range | CategoryRange, CountRange, QuantityRange, TimeRange; exactly two endpoints; independent aggregate classes | Full typed support; retain invalid/inverted/NaN edge cases for policy validation |
| Record | DataRecord has one or more uniquely named fields of any component type | Full recursive support |
| Vector | Ordered Quantity/Count/Time coordinates, mandatory reference frame, per-coordinate axis IDs, no optional coordinates | Full support; frame-aware semantics |
| Choice | Uniquely named alternatives; one selected item/value at a time; optional selector Category metadata | Full support; discriminator compilation required |
| Array | Homogeneous `elementType`; fixed or variable `elementCount`; nested arrays model multiple dimensions | Full model/JSON support; bounded runtime sizes |
| Matrix | DataArray specialization for numerical/nested matrix values with frame/local-frame semantics | Full model/JSON support; shape and numerical restrictions |
| Geometry | GeoJSON value in JSON, WKT in text, WKB in binary; mandatory SRS and semantic definition | Full model/JSON support; text/binary follows codec milestones |
| SWE DataStream | Top-level element description plus encoding and optional values; cannot be nested as a data component | Preserve/validate; keep distinct from CSAPI DataStream resource |
| Encoding | JSONEncoding, TextEncoding, BinaryEncoding; XML artifact may remain in schema history but XML is removed from SWE 3.0 conformance | JSON first, Text second, Binary claim-gated, XML legacy only |

### 5.3 Common Metadata and Associations

Labels and descriptions are human-readable, while `definition` carries robust semantics. IDs identify/refer to components within a description. Field/item/coordinate names are supplied by the containing association and are unique in that container; they are not universal component identifiers. Association-by-reference must retain the original link and a resolution snapshot rather than replacing it irreversibly with fetched content. [N,P]

### 5.4 Value-State Taxonomy

| State | Meaning | Required behavior |
|---|---|---|
| Absent optional | Field omitted under `optional=true` | Preserve absence; JSON member absent, text/binary presence indicator as specified |
| Present nil | Declared sentinel equals a registered nil value | Store nil reason plus source sentinel; exclude from ordinary measurement semantics |
| Present ordinary | Type-valid, non-nil value | Apply constraints and operational checks |
| Invalid | Wrong type/shape, undeclared special, bad choice, cardinality/path/encoding failure | Reject ordinary write; preserve only in quarantine with diagnostics |
| Operationally rejected | Structurally valid but violates policy, safety, authorization, lifecycle, or cross-resource rule | Reject/deny with safe error and audit; do not relabel as nil |

JSON `null`, an absent member, `"NaN"`, an empty array, and a declared missing sentinel are not interchangeable. [N,P]

---

## 6. SWE Common-to-CSAPI / Glaux Resource Mapping

### 6.1 Mapping Matrix

| SWE concept/component | Source anchor | Authority | Related resource family | Representation pattern | Internal normalization | Encoding implication | Unit/semantic dependency | Validation implication | Persistence/query implication | Security/policy implication | Test implication | Downstream | Notes/unresolved |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Abstract component metadata | SWE §§6, 7.2 | N | All schema-bearing families; SensorML I/O | Component-description JSON | Type, definition, label, flags, source path, extensions | Encoding-neutral | Definition URI | Required fields, role-sensitive flags | Contract document + selected indexes | May disclose capability/meaning | Unknown member, order permutation, role fixtures | 023/024/028 | `name` belongs to containing association |
| Boolean/Text/Category | SWE §§7.2, 9.1, 10 | N | Observation/status/result; command/feasibility; SensorML | Scalar leaf | Typed leaf; code space/constraint | JSON bool/string; text/binary scalar | Category code space and definition | Token enum/pattern; category rule | Selective scalar index only when authorized/useful | Text and categories may contain sensitive state/free text | valid/nil/Unicode/regex/code-space corpus | 023/024/027/036 | Boolean has no ordinary constraint in published model |
| Count | SWE Count class | N | Counts, sequence/state codes, array dimensions | Integer leaf | Integer bounds and nil map | No NaN/infinities | Definition; no UOM required | Integer and AllowedValues; significantFigures inapplicable | Numeric index where query contract requires | Operational counts may reveal capacity | min/max, overflow, nil collision, float rejection | 023/027/036 | Published schema references generic AllowedValues; profile must reject invalid significantFigures use |
| Quantity | SWE Quantity class | N | Measurements, dynamic/status values, command parameters | Decimal leaf with UOM | Numeric representation, unit reference, nil/constraint | JSON number or special string | Mandatory UOM and definition | Same-unit constraints; special-value rules | Native numeric plus special/nil representation | Precision/range may reveal capability; conversion risk | finite/extreme/special/unit/constraint fixtures | 023/024/027/036 | Decimal fidelity/precision mechanism deferred |
| Time | SWE Time class; CSAPI mappings | N | Observation phenomenon/result time; Command issue time; other time-valued result | ISO date-time or numeric offset | Typed time representation plus source lexical/frame | JSON date string/number/special; codec-specific | Temporal UOM, reference frame/time | Frame/origin/UOM/type; CSAPI definition URI | Do not collapse with resource temporal axes | Time can expose activity and freshness | UTC/custom epoch, special, mapping, axis-cross tests | 023/024/027/034/036 | IDR-018 taxonomy controls canonical time |
| Four range types | SWE §§7.2, 10.5; issue #15 | N/X | Result extents, quality, capability/parameter values | Two-value aggregate | Ordered endpoint pair and kind | Exactly 2 JSON items; 2 tokens/values text/binary | Same semantics/UOM/code space as scalar | Cardinality, type, ordering/profile, nil endpoint | Range indexes only with declared semantics | Command ranges are not safe/authorized envelopes | inverted/equal/open-like/special/NaN pairs | 023/024/027/036/038 | Publication permits special strings; meaningful-bound policy downstream |
| `nilValues` mappings | SWE §§6.3, 7.2, 10 | N | Every simple value path | Sentinel-to-reason table | Explicit nil state + original sentinel/reason | Sentinel is encoded like normal scalar | Nil-reason vocabulary | Unique/nonconflicting sentinel; check before ordinary constraint | Store reason separately from queryable value | Nil reason may disclose failure; collision can corrupt meaning | missing/out-of-range/collision/null/absent cases | 023/024/027/034/036 | Never silently invent nil mapping |
| Constraints | SWE §§6.2, 7.2 | N | Data/command schema; SensorML capability/input | Attached typed predicate | Compiled predicate plus exact source | Applied after decoding/non-nil recognition | Unit/code space/ECMA-262 semantics | Type, inclusive intervals, enum, pattern, significant figures | Index only selected normalized bounds/tokens | Regex DoS; constraints are not authorization/safety | boundary, union, regex budget, precision corpus | 023/024/036/038 | Constraint applies only to parent value |
| Quality | SWE §§6.3, 7.2 | N/X | Observation/result/status and SensorML metadata | Static attached component or dynamic peer field | Scoped quality assertion with provenance | Same leaf/aggregate encoding when dynamic | Quality definition and UOM | Conceptual checks plus Glaux overlay due JSON-schema omission | Separate attributable quality metadata/value columns as chosen later | Quality claims can mislead or expose performance | static/dynamic, multiple, recursive-abuse, schema-gap fixtures | 023/024/027/034 | Do not recurse quality indefinitely |
| DataRecord | SWE §§7.3, 10 | N | CSAPI `recordSchema`; compound output/input/status | Ordered uniquely named fields | Recursive nodes, stable paths/order | JSON object default or array flag; flattened sequence text/binary | Child dependencies | Unique names, recursion/depth, optional encoding | Contract graph; values may project to columns/documents | Nested sensitive fields require field policy | object/array, optional, nested, order, duplicate-name cases | 023/027/034/036 | CSAPI wrapper permits AnyComponent, examples use DataRecord |
| Vector | SWE §§7.3, 10 | N | Position/orientation/motion and numeric compound values | Ordered coordinates with frame | Frame/local frame/axis mapping | JSON object default or array flag; sequence text/binary | CRS/TRS and UOM | Numeric-only coordinates, unique axis, no optional | Spatial/vector index only after semantic validation | Precise positions/frames sensitive | frame inheritance, axis swap, array/object variants | 023/024/026/027 | Schema property is `vectorsAsArrays` |
| DataChoice | SWE §§7.4, 10 | N | Alternative result/command/feasibility shapes | Unique named alternatives, one selected | Tagged union/discriminator and item paths | JSON one-member object; text name then value; binary zero-based index | Selected item's dependencies | Unique/nonempty items and valid selector | Store selected arm and value; query cautiously | Alternative names expose affordances | every arm, unknown/multiple/none, nested variants | 023/036/037/038 | SECD variant becomes negative/interoperability fixture |
| DataArray | SWE §§7.5, 9.4, 10; issue #6 | N/D | Repeated observations/results, scalar lists, profiles, trajectories, images | Homogeneous element plus count | Element contract, fixed/variable cardinality | JSON array; text/binary variable count precedes values | Element/dimension definitions | count/value agreement, nesting and budgets | Blob/document or specialized store; no table-shape assumption | Memory/compression/data-volume risk | scalar-list, fixed/variable/nested/empty/huge | 023/027/028/034 | Open scalar-list proposal does not change 3.0 |
| Matrix | SWE §§7.5, 10 | N | Pose/tensor/grid result or parameter | Numerical/nested array with frames | Shape, axes, frames | Array recursion; WKB not applicable | CRS and units | Rectangular/profile constraints and numeric element kind | Specialized store only after use-case proof | Large/precise matrices can reveal state | 2D/N-D, ragged, frame, overflow fixtures | 023/024/026/027 | Treat separately from generic DataArray semantics |
| Geometry | SWE §§7.6, 9.5, 10 | N | Observation result/parameter; SensorML-described output | Component with SRS and geometry constraint | Geometry kind, CRS, source representation | GeoJSON/WKT/WKB by codec | CRS and definition | geometry validity, allowed type, CRS | Spatial index only after validation/policy | Location sensitivity | all supported types, invalid/ring/CRS, cross-codec | 023/024/026/027 | Distinct from CSAPI feature geometry unless explicitly mapped |
| SWE DataStream | SWE §7.5 | N | Sensor/process stream description; support artifact | Top-level elementType + encoding + optional values | Contract wrapper only | Own encoding/value block | Children | Cannot nest; descriptor/container rules | Document/artifact, not CSAPI resource row | External values and links need resolver policy | non-nesting, out-of-band, inline variants | 023/028/034 | Never confuse with CSAPI DataStream identity |
| CSAPI Observation schema wrapper | CSAPI Part 2 §21–23 | N | DataStream/schema and Observations | `obsFormat` + `recordSchema` + `encoding` | Versioned ObservationContract | JSON/Text/Binary class selects codec | Required mapped Time; optional sampling feature | Wrapper + SWE tree + mapping + cross-resource checks | Bound to DataStream revision/fingerprint | Schema may reveal outputs and status | exact examples, alternate formats, evolution | 023/027/031/034 | `application/swe+csv` artifact exists but accepted final class names must control |
| CSAPI Command schema wrapper | CSAPI Part 2 §21–23 | N | ControlStream/schema, Commands, feasibility | `commandFormat` + `recordSchema` + `encoding` | Versioned CommandContract | JSON/Text/Binary class selects codec | Optional issue-time/SamplingFeature mappings; controlled properties | Schema + mapping + policy/safety/lifecycle checks | Bound to ControlStream revision/fingerprint | Highly sensitive; constraints not authority | PTZ, choice, invalid, policy-redacted, evolution | 023/036/037/038 | Feasibility consumes same parameter contract but has separate lifecycle |

### 6.2 Non-Collapse Rules

1. SWE `DataStream` is not a CSAPI `DataStream` resource.
2. A SensorML output/input component is not automatically a CSAPI DataStream/ControlStream.
3. A component `definition` is not a Glaux Property resource ID, though it may bind to one through governed evidence.
4. A Quantity unit is not the observed property.
5. A component descriptor is not a value instance; a default inline `value` is not an Observation or Command.
6. Status described by a SWE path is not current status until an accepted Observation and projection rule establish it.
7. `optional`, nil, JSON null, invalid, denied, unavailable, and redacted remain distinct.
8. `updatable=true` does not create a writable API or authorize control.
9. A declared constraint/capability is not authorization, safety, feasibility, or current availability.
10. Component-path references are not resource links or JSON Pointers.

---

## 7. Data Component Type Support Strategy

### 7.1 Support Vocabulary

Every capability statement must use one or more of: preserve source; parse; structurally validate; semantically/profile validate; normalize/index; compile; ingest values; emit values; transform; and round-trip. The binary yes/no word “support” is prohibited in architecture and conformance records without those qualifiers. [P]

### 7.2 Required Model Breadth

Glaux's canonical SWE graph should be a closed tagged union for every published 3.0 component kind plus an `UnknownComponent` preservation node available only to quarantine/forward-compatible parsing. Known members use typed fields; unknown members and associations retain lexical JSON, source location, and provenance. Public generation emits only understood, authorized, profile-valid kinds. [P]

The graph should model:

- component role: descriptor, inline container/default, or unresolved;
- stable internal node ID and standards-facing component path;
- ordered container children and their soft names;
- source and normalized semantic/unit/frame references;
- optional/updatable flags with default provenance;
- constraints, nil mappings, quality, and association form;
- fixed/variable dimensions and declared/defaulted encoding properties; and
- extensions without letting them bypass validation or policy.

### 7.3 Runtime Coverage Levels

| Level | Required behavior | Components/encodings |
|---|---|---|
| M1 model/preservation | Lossless source, recursive parse, typed known nodes, opaque unknowns, diagnostics | All published component and encoding classes |
| M2 contract validation | Structural, graph, role, path, mapping, profile, compatibility, capability checks | All published component trees; unsupported algorithms fail activation |
| V1 JSON values | Bidirectional codec and validation for every published component family | Scalars/ranges/record/vector/choice/array/matrix/geometry; both JSON flags |
| V2 Text values | Standards-exact codec with separators, optional markers, variable counts, choices, WKT | Same component set once tests pass |
| V3 Binary values | Bounded codec with declared data types/refs/order/length and WKB; approved algorithms only | Advertised only after full positive/negative/interoperability evidence |
| L legacy | Preserve/migrate SWE 2.x XML with declared loss report if roadmap authorizes | No SWE 3.0 output or conformance implication |

This ordering is a release strategy, not a reinterpretation of optional OGC conformance. A deployment advertises only levels enabled and proven by its evidence registry. [P]

### 7.4 Special Component Rules

- Category without a code space requires a constraint; code-space ordering is necessary for a meaningful CategoryRange.
- Count is integer. Quantity and Time may use standardized special-value strings; Count does not.
- Range classes are two-value aggregates, not aliases of scalar components or ISO interval strings.
- Vector coordinates are Quantity, Count, or Time; each requires an axis ID and cannot be optional.
- DataChoice selects exactly one arm. A JSON object with multiple arms is invalid even if each arm is individually valid.
- DataArray scalar elements are the standard scalar-list solution. Fixed count must match; variable-size text/binary includes a count and can be empty.
- Matrix constrains element kinds and frame semantics beyond a generic nested array.
- Geometry JSON values use GeoJSON geometry, text uses WKT, and binary uses WKB; a SWE Geometry is not automatically the feature's primary geometry.
- Inline values under an elementType descriptor are forbidden where parent block values supply them.

### 7.5 Evolution and Compatibility

Each activated contract is immutable. Updates create a new revision and compatibility assessment. Compatibility is directional and operation-specific: adding an optional JSON-object field might permit old writers but break compact arrays, text/binary order, fixed consumers, semantic mappings, or policy projections. Existing values remain attached to their original fingerprint. A DataStream/ControlStream change with existing records follows Part 2 compatibility rules and the detailed IDR-SRV-023/029/034/036 decisions; it never mutates history in place. [N,P]

---

## 8. Encoding and Representation Boundary Findings

### 8.1 Contract Artifact Set

For each advertised format, Glaux should retain:

1. exact received/generated schema-wrapper bytes and digest;
2. parsed wrapper and component graph;
3. normalized encoding-neutral component revision;
4. exact encoding descriptor and defaults as applied;
5. compiled codec/validation plan and fingerprint;
6. profile/schema/tool/software pins;
7. compatibility result against the previous active revision;
8. authorization/releasability classification and generated public view; and
9. activation, retirement, and transformation evidence.

### 8.2 JSON Encoding

JSON scalar mappings follow component type, not generic coercion. Quantity and Time recognize `"NaN"`, `"-Infinity"`, and `"+Infinity"`; ordinary strings are not numeric. Ranges are exactly two items. DataChoice is a one-member object named for the selected item. DataArray and Matrix are arrays. Geometry is GeoJSON. [N]

DataRecord and Vector default to objects. `recordsAsArrays=true` and `vectorsAsArrays=true` independently switch them to arrays using descriptor order. Missing optional object members and compact-array positional presence must follow the standard's rules; codecs must not infer one representation from the payload when the descriptor says another. JSON object order is irrelevant for object parsing, but descriptor order remains binding for array/text/binary generation. [N,P]

### 8.3 Text Encoding

TextEncoding defines token and block separators, optional decimal separator, and whitespace-collapse behavior. Separators must not occur in dataset values. Nested aggregates have no extra delimiters; traversal order and component structure provide framing. Optional fields encode `Y` followed by a value or `N`; choices encode the selected item name before its value; ranges use two tokens; variable arrays begin with their count; root array/stream elements use block separators; Geometry uses WKT. [N]

Glaux must validate separator non-emptiness, collision risk, decimal/token/block ambiguity, complete consumption, trailing data, nesting depth, token length, numeric lexical bounds, and record count. `application/swe+text` is the CSAPI Part 2 contract selected in the accepted baseline. CSV-like output may be a separately documented profile/extension only when its delimiter/newline rules are exact; it cannot silently replace the standard class. [N,X,P]

### 8.4 Binary Encoding

BinaryEncoding binds overall byte order, raw/base64 byte encoding, optional total byte length, and ordered member descriptions. Component entries identify a component path and datatype and may declare lengths, bits, padding, or encryption; Block entries refer to aggregates and may declare compression/encryption. Optional fields carry ASCII Y/N markers; choices use zero-based indexes; variable arrays carry counts; Geometry uses WKB. SWE defines no concrete compression or encryption algorithms, so implementations need not support arbitrary declared URIs. [N]

Before activation, Glaux must prove all references resolve uniquely, every leaf is covered consistently, lengths are nonnegative/bounded, padding and total size agree, datatypes are allowlisted, endian handling is deterministic, strings and variable arrays are bounded, base64 is canonical/size-limited, and WKB is bounded and CRS-consistent. Unknown compression/encryption URIs are preserved but not executed. Encryption metadata does not replace transport security, key management, authenticated integrity, or content policy. [P]

### 8.5 Inline and Out-of-Band Values

Within a SWE 3.0 JSON component document, inline DataArray/Matrix/DataStream values use JSON encoding and a JSON array. Out-of-band values may use other declared encodings and may be referenced by HTTP, HTTPS, or data URI. Glaux should preserve this distinction, disable automatic remote fetch on ordinary requests, allowlist schemes/hosts, cap redirects and sizes, validate media type/digest, and store resolution evidence. Data URIs are input, not inherently safe embedded content. [N,P]

### 8.6 Import and Generation

Strict registration accepts only supported, conformant, authorized contracts. Quarantine import preserves partial/invalid/unknown/legacy content and diagnostics without activation. Generation starts from the normalized contract, applies one explicit profile and encoding capability, orders deterministically, and attaches provenance. Transforming SWE to JSON Schema/OpenAPI is a derived convenience artifact and must carry loss/coverage metadata; JSON Schema alone cannot express all component-path, semantic, encoding, nil, quality, or cross-resource rules. [P]

---

## 9. Observation, Datastream, Status, and Dynamic-Data Findings

### 9.1 DataStream Contract

A CSAPI DataStream identifies a feed and shared observed properties/result schema; its schema endpoint provides the format-specific contract. The SWE wrapper associates `obsFormat`, `recordSchema`, and `encoding`. The component tree describes a complete encoded Observation record, not merely its `result` member. At least one Time component maps to `phenomenonTime` or `resultTime` using prescribed URIs; an included SamplingFeature reference uses Text and the SOSA FeatureOfInterest URI with a local resource ID value. [N]

Glaux should compile those mappings into named system fields while retaining their component paths. Unmapped fields remain result/status/parameter content according to the registered profile. Duplicate or ambiguous time/sampling-feature mappings fail strict activation unless an accepted profile gives deterministic roles. Values attach to the parent DataStream ID and exact schema fingerprint. [P]

### 9.2 Observation Ingestion

The ingestion sequence is: authorize stream/format -> select active contract revision -> bound/decode -> recognize optional/nil state -> apply component constraints -> map CSAPI fields -> apply semantic/profile/cross-resource checks -> persist immutable raw and canonical evidence atomically -> update authorized projections/indexes. A failure at one layer returns a stable safe error location using component paths and does not partly advance stream extents or current status. [P]

Batch and streaming ingestion use the same compiled contract. Transport framing and replay are outside SWE. Live transport must carry the schema identity/fingerprint or bind it unambiguously through a session/topic contract; schema changes cannot reinterpret buffered or replayed payloads. [P]

### 9.3 Status and Dynamic Properties

SWE can describe status fields, battery life, quality of service, or other dynamic values and can encode them beside measurements. It does not decide which values create canonical current-status projections. IDR-SRV-020 controls: status is observation-derived; availability is separately assessed; stream `live` is narrow; Command status, feasibility, and events remain distinct. [N,P]

A status definition should therefore bind `(DataStream revision, ComponentPath)` to a typed status projection rule with semantic identity, temporal choice, nil behavior, freshness policy, and authorization. Current/latest status records retain source Observation and schema fingerprints. Nil, stale, denied, no-observation, invalid, and unavailable must not collapse to `null` or a single false value. [P]

### 9.4 Persistence and Query Implications

Store exact payload bytes or an integrity-preserving canonical payload artifact where policy permits, decoded typed values, per-field nil reasons, mapping evidence, contract fingerprint, and time/provenance identities. Normalize/index only fields with real API/query/operational need. Arrays, choices, matrices, qualities, and unknown extensions may remain document/blob-backed while selected scalar/time/geometry paths receive indexes. Physical choices belong to IDR-SRV-027/028. [P]

---

## 10. Command, Control Stream, and Feasibility Findings

### 10.1 Command Contract

A ControlStream's schema wrapper parallels the Observation wrapper: `commandFormat`, `recordSchema`, and `encoding`. A mapped issue timestamp uses Time with the prescribed OGC IssueTime definition; a mapped SamplingFeature uses Text, the SOSA FeatureOfInterest definition, and a local resource ID. Command payloads conform to the exact parent ControlStream contract. [N]

SensorML inputs and parameters can inform ControlStream component definitions, but the linkage must use an explicit versioned input/component-path binding. Neither the presence of a SensorML input nor `updatable=true` creates a ControlStream, a command permission, or an executable action. [N,P; IDR-SRV-021]

### 10.2 Validation Layers for Commands

Command acceptance requires distinct outcomes:

1. encoding and component-tree validity;
2. schema/profile and component-constraint validity;
3. stream/resource/lifecycle consistency;
4. principal and operation authorization;
5. policy/releasability decision;
6. operational safety/interlock decision;
7. feasibility/evaluation decision where applicable; and
8. durable submission/idempotency/audit behavior.

A value inside `AllowedValues` proves only layer 2. It does not prove the actuator can safely execute it now. Likewise a nil command parameter is accepted only if both the contract and command semantics permit the declared nil reason; “missing” cannot bypass a required safety parameter. Detailed state transitions remain IDR-SRV-036–038. [P]

### 10.3 DataChoice and Mode-Like Commands

DataChoice is appropriate for alternative command shapes when exactly one operation/arm is selected. Each item requires a stable unique name, policy classification, parameter tree, and test vector. Policy may hide or deny an arm; it must not emit a malformed choice schema or silently renumber binary choice indexes for an existing contract revision. A redacted public schema therefore requires its own revision/fingerprint and must not be used to decode values produced against the unredacted schema. [N,P]

### 10.4 Feasibility and Results

Feasibility input can reuse the same typed parameter contract as a prospective command, but feasibility request/result resources have separate identity, evaluation time, evidence, expiry, and lifecycle. A feasible answer is not authorization and may become stale. Command results may themselves bind to a versioned result contract; they are not the Command's input schema or Command status. [P; IDR-SRV-018,020]

### 10.5 Sensitive Contract Content

Control definitions reveal command names, controlled properties, bounds, timing, axes, precision, targeting fields, modes, and device limitations. Schema reads, errors, OpenAPI, examples, caches, generated UI forms, and feasibility explanations can leak as much as command values. Authorization and policy must cover the entire contract and its derived artifacts, not only POST endpoints. [A,P]

---

## 11. Units, Observed Properties, Controlled Properties, and Semantic Dependency Findings

### 11.1 Separation of Semantic Roles

| Role | SWE/CSAPI expression | Must not collapse with |
|---|---|---|
| Represented property | Component `definition` URI | Unit, label, component path, resource ID |
| Unit | `uom` with code and/or href plus optional label/symbol | Quantity definition or display string |
| Category vocabulary | `codeSpace` plus token constraint | Free Text or local enum without identity |
| Reference frame | `referenceFrame`, `referenceTime`, `localFrame`, `axisID`, Geometry `srs` | Unit or feature geometry |
| Observed property | DataStream property association and component binding | Every leaf automatically becoming a Property resource |
| Controlled property | ControlStream property association and input binding | Allowed range, capability, or authorization |
| Component path | Ordered soft-name path in one contract revision | JSON Pointer, canonical URL, or UID |

### 11.2 Unit Handling Boundary

Quantity/QuantityRange and Time/TimeRange require a unit reference. Published JSON permits a code, an href, or both; if both are present they must denote the same unit under the later semantic policy. Glaux should preserve all supplied fields and source lexical form, normalize a governed unit key only with evidence, and never convert values silently on ingestion or emission. Constraint bounds and nil sentinels are interpreted in the component's declared representation/unit. [N,P]

IDR-SRV-024 owns UCUM acceptance, URI registries, equivalence, dimensional analysis, conversion precision, labels/symbols, unknown/conflicting units, and DDIL resolution. This report requires the contract model and validation interface to retain enough evidence for those decisions. [P]

### 11.3 Property Binding

The same definition URI can appear in SensorML output/input descriptions, SWE paths, DataStream observed-property lists, and ControlStream controlled-property lists. Equality of text is evidence, not automatic identity. A binding record should state source and target revisions, component path, property role, asserted semantic URI, authority, resolution state, transformation, confidence/validation status, and provenance. [P]

Composite results may contain multiple observed properties plus time, sampling feature, status, and quality fields. Composite commands may contain multiple controlled properties plus issue time and target context. The server must not label every field as observed/controlled or assume one stream-level property applies to all nested leaves without an explicit mapping/profile. [P]

### 11.4 Semantic Resolution

Definition, nil-reason, code-space, unit, datatype, compression/encryption, CRS/TRS, and quality URIs are typed references with different resolvers and trust policies. Syntax-valid URI does not establish availability, authority, semantic compatibility, or permission to fetch. Cache entries need source, digest, retrieval time, validation, freshness, policy, and failure state; DDIL operation must use pinned/approved evidence rather than blocking writes on arbitrary networks. [P; IDR-SRV-019]

---

## 12. Validation, Profile, Schema, Encoding, and Conformance Implications

### 12.1 Validation Ladder

| Layer | Question | Representative checks |
|---|---|---|
| 0 transport | Can the request safely be read? | media type, size, decompression, charset/base64, timeout |
| 1 syntax | Is the representation syntactically valid? | JSON/token/binary syntax, complete consumption |
| 2 published schema | Does description match pinned OGC JSON Schema 2020-12? | references, required/type/enum/cardinality |
| 3 SWE graph | Is the component graph internally valid? | unique names, roles, recursion, element/choice/path/reference/frame rules |
| 4 encoding | Does descriptor define an executable codec and does value conform? | flags, order, separators, types, lengths, counts, selectors, nil |
| 5 CSAPI mapping | Does wrapper/resource specialization hold? | format/encoding agreement, required Time, SamplingFeature mappings, parent contract |
| 6 Glaux/AEP profile | Does configured profile hold? | supported subset, semantic bindings, allowed references/algorithms, compatibility |
| 7 operational policy | May this principal perform/expose this action/content now? | auth, releasability, safety, lifecycle, rate/resource limits |
| 8 evidence | Can the outcome be reproduced and claimed? | artifact/tool/profile/software pins, report, test/conformance evidence |

IDR-SRV-023 selects mechanisms and error schemas. This report requires stable diagnostic codes, component paths, source locations where safe, severity, layer, offending contract/value fingerprint, and redaction-aware messages. [P]

### 12.2 Profile Design

The Glaux SWE profile should be machine-readable and versioned. It must state permitted component kinds/depth/cardinality; required definitions, units, frames, labels, names, and mappings; supported encoding classes and datatype/algorithm URI allowlists; inline/out-of-band rules; constraints/nil/quality policy; CSAPI wrapper requirements; extension policy; security/resource budgets; and profile-to-conformance evidence. Profiles may tighten but must not silently rewrite the approved standard. [P]

### 12.3 Published Schema and Prose Gaps

- Preserve vendored OGC artifacts byte-for-byte with origin, edition, digest, and retrieval date.
- Put Glaux fixes/tightening in separately identified overlays or semantic validators.
- Record the omitted JSON `quality` member, retained `XMLEncoding` schema residue, `vectorsAsArrays` prose typo, stale CSAPI draft links/notes, binary “Text” typo, CSV artifacts, generic-SWE versus CSAPI media-type distinction, and any schema/code mismatch in one interpretation register.
- Test both the accepted interpretation and plausible erroneous variant; accept only the former on strict paths.
- Recheck official corrigenda/releases in IDR-SRV-057.

### 12.4 Compatibility Validation

Schema diff must understand component semantics, not only JSON member changes. Breaking examples include reordering compact/text/binary fields; renaming paths; changing kind/unit/definition/frame; narrowing constraints; changing nil sentinel meaning; fixed/variable count changes; choice reordering under binary; encoding flag/default changes; and removing a policy-visible field. Compatibility outcome is recorded per read/write direction and media type. [P]

### 12.5 Conformance Claims

Claims are derived from enabled runtime capability and passing evidence, not configuration aspiration. JSON, Text, and Binary are separate Part 2 classes. Model preservation of BinaryEncoding is not binary conformance. Supporting only simple JSON values is not full SWE JSON encoding if advertised streams contain complex components. Generated conformance, OpenAPI, schema endpoints, Accept/Content-Type behavior, and actual codecs must agree. [N,P]

---

## 13. Security, Policy, and Releasability Implications

SWE Common states no security considerations; Glaux must supply them. [N,P]

### 13.1 Threat and Control Matrix

| Threat | Attack/failure | Required control |
|---|---|---|
| Recursive complexity | Deep records/choices/arrays exhaust stack/CPU | Iterative or bounded traversal; depth/node/branch budgets; fuzzing |
| Cardinality/length abuse | Huge counts, dimensions, binary lengths, strings, values | Checked arithmetic, per-contract/request limits, streaming decode, quota |
| Reference resolution | SSRF, redirect abuse, mutable/malicious schemas/semantics | No implicit fetch; scheme/host allowlist; redirect/size/time limits; digest/cache evidence |
| Data URI/base64/compression | Memory amplification or decompression bomb | Pre/post decode bounds; algorithm allowlist; ratio and nesting limits |
| Regex | Catastrophic backtracking or semantic mismatch | ECMA-262-compatible bounded engine/evaluation; pattern length budgets |
| Binary declaration | Type confusion, path overlap, endian/length errors | Compile once; unique references; allowlisted datatypes; complete-consumption tests |
| Encryption declaration | Unsupported/unauthenticated algorithm mistaken for security | Preserve unknown metadata; execute only approved authenticated profile; separate transport/key policy |
| Nil/constraint collision | Sentinel accepted as valid measurement or safety bypass | Unique typed sentinel map; nil recognition before constraints; reject ambiguity |
| Semantic poisoning | Hostile unit/property/frame URI changes interpretation | Typed resolver, authoritative sources, source-qualified assertions, no silent conversion |
| Schema disclosure | Reveals sensors, controls, ranges, positions, modes | Resource/field policy before serialization; separate raw/generated access |
| Error/log disclosure | Payload values, command limits, URIs, classifications leak | Stable redacted diagnostics; protected raw evidence; structured audit |
| Policy-shaped schema drift | Redaction changes ordering/choice indexes and decodes wrongly | Separate policy projection fingerprint; never use redacted schema for original payload |

### 13.2 Policy Scope

Policy applies to schema-resource discovery, source artifacts, normalized contracts, indexes, values, aggregate statistics, current/latest projections, cached references, validation reports, errors, logs, exports, examples, OpenAPI, and event/publication payloads. A principal allowed to write a value is not automatically allowed to read the schema or enumerate command arms. [P]

### 13.3 Provenance and Audit

Record who registered/activated/retired a contract; source digest and authority; validator/profile/tool result; compatibility decision; mapping and semantic-resolution evidence; decoding fingerprint; policy decision; and any transformation/conversion. Exact sensitive payloads can be separately protected or content-addressed rather than copied into ordinary logs. [P; IDR-SRV-019]

---

## 14. Fixture, Golden-File, and Interoperability Test Implications

### 14.1 Fixture Dimensions

| Dimension | Required cases |
|---|---|
| Component kind | Every scalar, range, record, vector, choice, array, matrix, geometry, SWE DataStream |
| Role | descriptor, inline/default container, out-of-band values, invalid mixed role |
| Nesting | shallow, nested record/choice/array, maximum allowed, one over limit |
| Value state | ordinary, absent optional, each nil reason, JSON null, invalid, policy denied |
| JSON form | record/vector object and array combinations; reordered objects; duplicate/unknown/missing choice |
| Text | separator/decimal/whitespace variants, optional Y/N, choice name, fixed/variable arrays, WKT, injection/collision |
| Binary | both endian forms, raw/base64, supported types, padding/bits/length, choice index, array count, WKB, truncation/trailing data |
| Semantics | known/unknown/conflicting unit, definition, code space, CRS/TRS, axis, nil reason, quality |
| CSAPI mapping | phenomenon/result time, issue time, SamplingFeature, compound result/command, ambiguous/duplicate mapping |
| Evolution | additive optional, reorder, rename, type/unit/definition/constraint/nil/choice/encoding changes |
| Security | oversized/deep, malicious URI, redirect, data URI, regex, decompression, integer overflow, sensitive error |
| Policy | full/raw, authorized projection, redacted/denied field, schema/value authorization mismatch |

### 14.2 Canonical Golden Set

The golden corpus should include official scalar SWE JSON and SWE text examples, CSAPI scalar observation and PTZ command schemas, a full mixed DataRecord, both compact flags, a frame-aware Vector, every DataChoice arm, scalar-list DataArray, fixed and variable nested DataArrays, Matrix, every Geometry representation, quality static/dynamic cases, and nil/special/range boundaries. Each item records source, authority, exact bytes, digest, expected parse graph, normalized contract, fingerprint, encoded values, diagnostics, profile, and whether it is normative-derived, project-created, implementation-observed, or intentionally invalid. [P]

### 14.3 Negative and Variant Set

Retain fixtures for: SECD's structurally incompatible DataChoice; OSH nested variable-array failure; a narrow client rejecting valid complex components; `application/swe+csv` versus the accepted media contract; `vectorAsArrays` typo; quality member conceptual/schema gap; component name versus JSON Pointer references; inline non-JSON blocks; stale draft media notes/links; duplicate names; nil collisions; Count specials; range cardinality; array count mismatch; binary truncation; and unsupported algorithm URIs. [I,X,P]

### 14.4 Round-Trip and Cross-Implementation Assertions

Tests must distinguish byte equality, parsed-graph equality, semantic contract equality, and value equality. Generated component descriptions should be deterministic, but imported lexical order/unknown members may require preserved-source rather than byte-reencoded equality. Cross-codec tests should decode to the same typed values and nil/quality states. External-client tests state exact supported component breadth and report transport, parse, and semantic-completeness results separately. [P]

### 14.5 Conformance Evidence

Every claim needs requirement IDs, positive/negative tests, fixture digests, profile and tool pins, runtime configuration, and result. The official ATS is necessary but insufficient for security budgets, policy, source fidelity, schema evolution, implementation variants, and external-client completeness. [N,P]

---

## 15. Downstream Topic Handoff Matrix

| Topic | Required handoff | Guardrail |
|---|---|---|
| IDR-SRV-023 | Select recursive JSON Schema/semantic/codec validator stack; profile overlays; diagnostics; compatibility diff; artifact-gap register | Do not edit vendored OGC schemas or equate schema validity with full validity |
| IDR-SRV-024 | Govern units, definition/property/code-space/nil/quality/frame URIs, equivalence, conversion, resolver/cache/DDIL policy | Preserve source and evidence; no silent equivalence/conversion |
| IDR-SRV-025/027 | Store immutable contract revisions and typed values/nil/quality/arrays; index selected paths; bind historical values to fingerprints | Do not make SWE tree the whole database or rebind history |
| IDR-SRV-026 | Geometry/vector/matrix CRS, indexing, transformation, uncertainty | SWE Geometry is not automatically feature geometry |
| IDR-SRV-028 | Exact artifact, parsed graph, compiled plan, validation/transformation evidence, unknown/legacy retention | Keep source, canonical, and generated views separate |
| IDR-SRV-029/030 | Atomic activation/value writes, compatibility, idempotency, retention and deletion of referenced contracts | No partial extent/status advancement or deletion of referenced schema revision |
| IDR-SRV-031/032/033 | Strict versus quarantine ingestion; publisher/simulator contract negotiation, fingerprint and error behavior | Unsupported codec fails explicitly; no implicit remote resolution |
| IDR-SRV-034 | Observation mapping, status path binding, latest/current, schema-change semantics | Carry IDR-018/020 time/status non-collapse rules |
| IDR-SRV-035 | Streaming/session schema binding, replay, schema-change/event behavior; Part 3 adaptation if later selected | Transport framing is not SWE encoding; no draft adoption here |
| IDR-SRV-036 | ControlStream/Command schema revisions, DataChoice modes, input-path bindings, command/result contracts | Constraints/updatable do not define lifecycle or permission |
| IDR-SRV-037 | Reuse typed inputs for feasibility with separate evidence, expiry, results | Feasible is not authorized/executable |
| IDR-SRV-038 | Command policy, safety/interlocks, nil/default handling, controlled-property exposure, audit | SWE structural validity is only one gate |
| IDR-SRV-039–041 | Threat model, field/schema policy, releasable projection, logging/audit | Protect definitions and derived artifacts as well as values |
| IDR-SRV-042/043 | DDIL-safe typed reference resolution/cache and external synchronization of schema revisions | URL syntax is not trust; never reinterpret buffered data |
| IDR-SRV-044–049 | Select Rust/tool architecture, codec/plugin boundaries, deployment capability registry, migration | Capability/claim matrix is runtime-derived and versioned |
| IDR-SRV-050/051 | Trace SWE/CSAPI requirement IDs, profile rules, artifact interpretations, security rules to evidence | Claim each media class separately |
| IDR-SRV-053 | Materialize §14 corpus with exact pins/digests, generators, shrinkers, classifications | Examples are seeds, not automatically valid goldens |
| IDR-SRV-056 | Test JSON/Text/Binary and complex components across named server/client versions | Do not narrow Glaux output to deficient clients |
| IDR-SRV-057 | Recheck open #6/#15/#144, quality schema gap, corrigenda, media registrations, issue #180 outcome | Approved 3.0/1.0 remains controlling until formally changed |

---

## 16. Recommendations

| ID | Recommendation | Priority | Authority |
|---|---|---|---|
| R-022-01 | Adopt the immutable versioned SWE data-contract subsystem: exact source, parsed graph, canonical contract, generated view, compiled plan, and validation/transformation evidence. | Critical | P; IDR-015/019/021 |
| R-022-02 | Model and preserve every published SWE Common 3.0 component/encoding family; use typed known nodes plus quarantined opaque unknowns. | Critical | N/P |
| R-022-03 | Bind each Observation/Command to the exact DataStream/ControlStream contract revision and fingerprint used to decode it. | Critical | N/P |
| R-022-04 | Implement full recursive SWE JSON value coverage first, SWE Text second, and advertise SWE Binary only after bounded complete codec evidence. | Critical | N/P |
| R-022-05 | Publish a per-component, operation, direction, and media-type capability matrix; derive claims from enabled passing evidence. | Critical | P |
| R-022-06 | Keep component description and encoding separate; associate them in the containing CSAPI schema wrapper. | Critical | N; issue #144 |
| R-022-07 | Compile unique soft names, descriptor order, typed component paths, frames, dimensions, choices, constraints, nil mappings, and encoding details once per revision. | High | N/P |
| R-022-08 | Distinguish absent optional, declared nil, ordinary, invalid, denied, unavailable, stale, and redacted states. | Critical | N/P; IDR-020 |
| R-022-09 | Use DataArray with scalar `elementType` for scalar lists; do not invent unapproved list component types. | High | N/D; issue #6 |
| R-022-10 | Preserve units, definitions, code spaces, frames, axes, quality, and reference evidence without silent conversion or equivalence. | Critical | N/P |
| R-022-11 | Treat SWE constraints and `updatable` as descriptive validation metadata, never authorization, safety, feasibility, or availability. | Critical | N/P |
| R-022-12 | Separate strict conformant contract registration from privileged quarantine/promotion of invalid, partial, external, unknown, or legacy material. | Critical | P; IDR-021 |
| R-022-13 | Vendor approved schemas unmodified and address quality/prose/media/artifact gaps through identified overlays, validators, adapters, and tests. | Critical | N/X/P |
| R-022-14 | Apply resource budgets and policy to schemas, values, references, codecs, indexes, errors, logs, caches, generated views, and examples. | Critical | P |
| R-022-15 | Require the fixture matrix in §14, including complex recursion, schema evolution, security, policy, and external-client semantic completeness. | High | I/P |
| R-022-16 | Keep SWE DataStream, CSAPI DataStream, SensorML I/O, Property, status, Observation, Command, feasibility, and transport concepts explicitly non-collapsing. | Critical | N/P |

### 16.1 Acceptance Decision

Acceptance makes these recommendations the SWE Common planning baseline. Later topics may choose mechanisms within it. A material departure—such as opaque-only schemas, mutable in-place contracts, automatic URI resolution, binary claims without codecs, interpreting constraints as authorization, or using implementation-specific shapes as the profile—requires an explicit superseding decision with standards, security, migration, and test impact. [P]

---

## 17. Risks, Constraints, and Open Questions

### 17.1 Risks and Mitigations

| Risk | Impact | Mitigation/handoff |
|---|---|---|
| Opaque-only SWE | Cannot validate, bind, query, authorize, or interoperate reliably | Typed graph + compiled contract + exact preservation |
| Over-normalization | Loses recursive/unknown/profile content and source fidelity | Selective projection plus source/parsed layers |
| Mutable schema binding | Historical data silently changes meaning | Immutable revision/fingerprint per value |
| Partial codec advertised | False conformance and client failures | Capability-derived claims and complete complex fixtures |
| JSON Schema treated as enough | Misses encoding, semantics, mapping, policy and artifact gaps | Layered validator and profile overlay |
| Nil/null/absence collapse | Corrupts measurements/status/commands | Explicit value-state taxonomy |
| Array/recursive exhaustion | Availability and memory risk | Budgets, checked arithmetic, bounded streaming/fuzzing |
| Binary algorithm confusion | Decode corruption or security bypass | Compile/allowlist, full consumption, reject unsupported |
| Semantic/unit poisoning | Wrong values, indexes, constraints, commands | Typed governed resolver and no silent conversion |
| Constraint-as-safety | Unsafe command accepted | Separate auth, policy, safety, feasibility gates |
| Schema disclosure | Reveals capabilities/controls/ranges/locations | Policy at contract and field level before generation |
| Client-driven narrowing | Standards-correct content excluded | Honest support matrix and external compatibility views only if authorized |
| Published gaps | Divergent implementations | Interpretation register, overlays, variant tests, upstream monitoring |

### 17.2 Constraints

- Approved SWE Common 3.0 and CSAPI 1.0 artifacts control; open proposals do not silently amend them.
- SWE's recursive expressive power means no finite relational flattening is universally correct.
- Text encoding depends on separators and schema order; binary depends on exact paths/types/order/length; conversions can be lossy.
- Policy redaction can invalidate or semantically alter a contract and therefore needs a distinct generated revision.
- Controlled AEP content cannot be redistributed and does not remove the need for explicit public profile evidence.
- This topic sets logical boundaries; physical technologies and detailed mechanisms remain downstream.

### 17.3 Open Questions and Owners

1. What exact Glaux/AEP SWE profile fields, limits, and permitted extensions apply per deployment? — IDR-SRV-023/024/040.
2. What validator/codec stack safely supports recursive JSON Schema 2020-12 and stable component-path diagnostics? — IDR-SRV-023/044.
3. What JSON form and overlay should Glaux support for conceptual `quality` pending official clarification? — IDR-SRV-023/024/057.
4. Which BinaryEncoding datatype, compression, and encryption URI profiles, if any, are launch requirements? — IDR-SRV-023/044/046.
5. What decimal precision and unit conversion policy applies to Quantity and constraints? — IDR-SRV-024/027.
6. Which SWE paths become indexed observation/status fields, and how are schema changes migrated? — IDR-SRV-027/034.
7. What exact compatibility rules govern DataStream/ControlStream replacement with existing values? — IDR-SRV-023/029/034/036.
8. Is SWE 2.x XML import launch scope, migration-only scope, or deferred? — roadmap after IDR-SRV-023/028/044.
9. Will open issues #6, #15, and #144 close or yield a corrigendum/new edition? — IDR-SRV-057.

None blocks this planning baseline. Each is intentionally assigned to its owning mechanism/profile topic.

---

## 18. Validation Against Plan Success Criteria

### 18.1 Methodology Completion

| Phase | Result | Evidence |
|---|---|---|
| 1. Source collection/extraction framework | Complete | §§3–4 |
| 2. Standards and CSAPI mapping | Complete | §§5–6 |
| 3. Component, encoding, representation boundaries | Complete | §§7–8 |
| 4. Observation/status/command/semantic dependencies | Complete | §§9–11 |
| 5. Validation/security/fixtures/interoperability | Complete | §§12–14 |
| 6. Synthesis | Complete | §§15–17 |

### 18.2 Success Criteria

| Topic-plan criterion | Status | Evidence |
|---|---|---|
| Relevant SWE concepts identified with source anchors | Met | §§3, 5, 6.1 |
| Concepts mapped to canonical CSAPI families | Met | §6 |
| Definitions, values, domain fields, artifacts, persistence distinguished | Met | §§4, 6, 8–10 |
| Component-type support scope identified | Met | §7 |
| Encoding, nil, constraint, unit, property, semantic implications documented | Met | §§7–8, 11 |
| Observation, status, dynamic, command, feasibility, SensorML dependencies documented | Met | §§6, 9–11 |
| Validation, security, conformance, fixture, interoperability implications documented | Met | §§12–14 |
| Implementation/community lessons incorporated nonnormatively | Met | §§3.4, 14 |
| Recommendations decision-usable and server-bounded | Met | §16 |
| Downstream handoffs explicit | Met | §§15, 17.3 |
| References explicit and reproducible | Met | §19 and pins in §3 |

### 18.3 Report Completion Checklist

- [x] Topic ID and plan link match the overall index.
- [x] All core and detailed question groups are answered or assigned explicitly.
- [x] Required 14-field mapping matrix is present.
- [x] Normative, AEP, project, implementation, draft, and unresolved evidence are separated.
- [x] Mutable evidence is pinned and checked September 14, 2026.
- [x] Controlled-source limits are explicit; no controlled content is redistributed or invented.
- [x] Conflicts with accepted reports are reconciled.
- [x] Recommendations, risks, fixtures, and downstream owners are explicit.
- [x] Report is ready for project-lead review.
- [x] Project-lead acceptance and date recorded.

### 18.4 Next Two Actions

1. Glaux Project Lead reviews and accepts IDR-SRV-022.
2. In the same instruction, the project lead authorizes execution of exactly IDR-SRV-023.

Under the established shorthand, a bare **`proceed`** at this review boundary carries both meanings. Acceptance does not authorize IDR-SRV-024, server implementation, or draft Part 3 implementation.

---

## 19. References

### 19.1 Controlling Standards and Artifacts

- Open Geospatial Consortium, [OGC 24-014, *OGC SWE Common Data Model Encoding Standard*, Version 3.0.0](https://docs.ogc.org/is/24-014/24-014.html), approved June 2, 2025; published July 16, 2025; accessed September 14, 2026.
- Open Geospatial Consortium, [official SWE Common 3.0 JSON schemas](https://schemas.opengis.net/sweCommon/3.0/json/), accessed September 14, 2026.
- Open Geospatial Consortium, [OGC 23-000, *OGC SensorML Encoding Standard*, Version 3.0](https://docs.ogc.org/is/23-000/23-000.html), accessed September 14, 2026.
- Open Geospatial Consortium, [OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html), accessed September 14, 2026.
- Open Geospatial Consortium, [OGC 23-002, *OGC API - Connected Systems - Part 2: Dynamic Data*, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html), accessed September 14, 2026.
- W3C/OGC, [*Semantic Sensor Network Ontology*](https://www.w3.org/TR/vocab-ssn/), accessed September 14, 2026.
- Open Geospatial Consortium, [`ogcapi-connected-systems` published source tag `v1.0.0`, commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2), checked September 14, 2026.

### 19.2 Official History Evidence

- [Issue #5, Geometry component](https://github.com/opengeospatial/ogcapi-connected-systems/issues/5); [#6, scalar-list proposal](https://github.com/opengeospatial/ogcapi-connected-systems/issues/6).
- [Issue #11, SWE JSON encodings](https://github.com/opengeospatial/ogcapi-connected-systems/issues/11); [#15, JSON range encoding](https://github.com/opengeospatial/ogcapi-connected-systems/issues/15).
- [Issue #16, special numerical values](https://github.com/opengeospatial/ogcapi-connected-systems/issues/16); [#17, Count special values](https://github.com/opengeospatial/ogcapi-connected-systems/issues/17); [#18, regex flavor](https://github.com/opengeospatial/ogcapi-connected-systems/issues/18).
- [Issue #19, component references](https://github.com/opengeospatial/ogcapi-connected-systems/issues/19); [#20, inline DataArray values](https://github.com/opengeospatial/ogcapi-connected-systems/issues/20).
- [Issue #40, UnitOfMeasurement](https://github.com/opengeospatial/ogcapi-connected-systems/issues/40); [#73, unit definition](https://github.com/opengeospatial/ogcapi-connected-systems/issues/73); [#74, UCUM requirement](https://github.com/opengeospatial/ogcapi-connected-systems/issues/74).
- [Issue #46, SensorML/SWE Common 3.0 refactor](https://github.com/opengeospatial/ogcapi-connected-systems/issues/46); [#55, location of encoding rules](https://github.com/opengeospatial/ogcapi-connected-systems/issues/55); [#71, compact JSON flags](https://github.com/opengeospatial/ogcapi-connected-systems/issues/71).
- [Issue #98, DataStream elementType schema](https://github.com/opengeospatial/ogcapi-connected-systems/issues/98); [#100, DataArray values options](https://github.com/opengeospatial/ogcapi-connected-systems/issues/100); [#105, XML confusion](https://github.com/opengeospatial/ogcapi-connected-systems/issues/105); [#106, trajectory limits](https://github.com/opengeospatial/ogcapi-connected-systems/issues/106).
- [Issue #144, specifying encoding](https://github.com/opengeospatial/ogcapi-connected-systems/issues/144); [#154, SWE Common 3.0 publication](https://github.com/opengeospatial/ogcapi-connected-systems/issues/154); [#180, component name placement](https://github.com/opengeospatial/ogcapi-connected-systems/issues/180).
- [Glaux upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), Version 1.9, SWE entries refreshed September 14, 2026.

### 19.3 Project and Controlled Sources

- NATO Consultation, Command and Control Board Joint Capability Group Intelligence, Surveillance and Reconnaissance, `AC/224(JCGISR)D(2026)0005`, April 27, 2026, project-controlled package, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`; used only through accepted IDR findings.
- [IDR-SRV-003 Standards Package Baseline](idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md).
- [IDR-SRV-004 Terminology Crosswalk](idr-srv-004-terminology-and-concept-crosswalk-report.md).
- [IDR-SRV-008 Conformance Mapping](idr-srv-008-conformance-class-and-requirement-mapping-report.md).
- [IDR-SRV-012 Content Negotiation](idr-srv-012-content-negotiation-media-types-and-encoding-selection-report.md).
- [IDR-SRV-014 OpenAPI Strategy](idr-srv-014-openapi-description-and-api-documentation-strategy-report.md).
- [IDR-SRV-015 Canonical Resource Model](idr-srv-015-canonical-glaux-server-resource-model-report.md).
- [IDR-SRV-016 Identifier and Lifecycle Strategy](idr-srv-016-identifier-uri-and-resource-lifecycle-strategy-report.md).
- [IDR-SRV-017 Relationship Model](idr-srv-017-relationship-and-linkage-model-report.md).
- [IDR-SRV-018 Temporal Model](idr-srv-018-temporal-validity-and-freshness-model-report.md).
- [IDR-SRV-019 Provenance, Quality, and Trust Model](idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md).
- [IDR-SRV-020 Status, Availability, and System Event Model](idr-srv-020-status-availability-and-system-event-model-report.md).
- [IDR-SRV-021 SensorML Representation Strategy](idr-srv-021-sensorml-representation-strategy-report.md).

### 19.4 Implementation and Interoperability Evidence

- [IDR-SRV-014A OSH Study](idr-srv-014a-osh-csapi-server-implementation-study-report.md).
- [IDR-SRV-014B Connected Systems Go Study](idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md).
- [IDR-SRV-014C pygeoapi/52°North Study](idr-srv-014c-pygeoapi-csapi-server-implementation-study-report.md).
- [IDR-SRV-014D SECD Study](idr-srv-014d-secd-csapi-server-implementation-study-report.md).
- [IDR-SRV-014E OS4CSAPI Client Smoke-Test Study](idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md).
- [IDR-SRV-014F SECD Interoperability Study](idr-srv-014f-secd-interoperability-findings-study-report.md).
- [IDR-SRV-014G OS4CSAPI Discussions Study](idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md).
- [IDR-SRV-014H Draft Part 3 and Implementation Study](idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md).

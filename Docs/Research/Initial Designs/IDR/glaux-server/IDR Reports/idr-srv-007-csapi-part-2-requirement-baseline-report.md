# Section 007: CSAPI Part 2 Requirement Baseline - Research Report

**Topic ID:** IDR-SRV-007<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-007 CSAPI Part 2 Requirement Baseline](../IDR%20Plans/idr-srv-007-csapi-part-2-requirement-baseline.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 41 detailed questions; all six methodology phases, eleven success criteria, fifteen deliverable requirements, and twelve minimum inventory fields are mapped<br>
**Methodology Used:** Direct extraction from the approved standard; requirement-class, conformance-class, resource-model, schema, OpenAPI, AsyncAPI, and abstract-test reconciliation; prerequisite and server-boundary classification; pinned artifact audit; downstream and test-traceability analysis; and independent completeness and technical reviews<br>
**Research Time:** Approximately 3 hours of AI-assisted elapsed execution time, including parallel independent extraction and review, on July 31, 2026<br>
**Primary Sources:**

- OGC 23-002, *OGC API - Connected Systems - Part 2: Dynamic Data*, Version 1.0
- OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0
- OGC 24-014, *SWE Common Data Model Encoding Standard*, Version 3.0
- Official OGC schema publication and the `opengeospatial/ogcapi-connected-systems` Version 1.0.0 tag

**Supporting Resources:** Accepted IDR-SRV-001 through IDR-SRV-006 reports, the Glaux Server Goal and Definition, project governance, OGC API - Features Part 4 development evidence, and official OGC development artifacts<br>
**Document Purpose:** Establish the traceable OGC API - Connected Systems Part 2 server-requirement baseline that later Glaux design, implementation, validation, conformance, and testing work must use<br>
**Author:** OpenAI Codex, with independent read-only requirement, conformance, and artifact audits<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** August 1, 2026<br>
**Date:** July 31, 2026<br>
**Last Updated:** August 1, 2026

---

## How to Read This Report

For the project decision, read Sections 1, 12, 13.3, and 16. They explain what Part 2 actually requires, what Glaux should carry forward, which decisions remain open, and what happens next. Read §13.2 when reviewing the underlying standards defects. Sections 3–11 and the appendices are the audit trail. They are deliberately exact so later design and coding work can cite a requirement instead of reconstructing the standard from memory.

Four labels keep unlike claims separate: a **standards obligation** is imposed by a normative source; a **source-backed finding** reports what an authoritative or informative artifact contains; **analysis** explains the consequence; and a **project recommendation** proposes how Glaux should respond. The report does not present a project choice as though OGC mandated it.

Plain-language acronyms used below: **ATS** means Abstract Test Suite; **ETS** means executable test suite; **CRD** means create, replace, and delete; **QoS** means quality of service; **FOI** means feature of interest; **SWE** means Sensor Web Enablement; and **DDIL** means denied, disrupted, intermittent, and limited conditions. Compact handoffs such as `IDR-010/017` mean `IDR-SRV-010` and `IDR-SRV-017`.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Requirement Extraction Methodology
5. Requirement-Class and Conformance-Class Inventory
6. Normative Requirement Inventory
7. Server Applicability and Boundary Classification
8. Dynamic-Data, Tasking, and Event Behavior Mapping
9. Schema, OpenAPI, Encoding, and Representation Artifact Inventory
10. Downstream Topic Handoffs
11. Traceability and Test-Strategy Implications
12. Recommendations and Decision Analysis
13. Risks, Constraints, Ambiguities, and Open Questions
14. Validation Against the Research Plan
15. References
16. Next Steps and Handoff
17. Appendices

---

## 1. Executive Summary

### 1.1 Plain-English Decision Brief

OGC API - Connected Systems Part 2 is the approved standard that defines the Web API behavior for connected-system **dynamic data and tasking resources**. It covers DataStreams and Observations; ControlStreams, Commands, Command Status, and Command Results; feasibility requests; System Events; advanced filters; write and update behavior; and JSON plus three SWE Common encodings. It builds on the Part 1 feature-resource foundation researched in IDR-SRV-006.

The approved Part 2 baseline contains **12 requirement classes, 130 numbered requirements, no numbered recommendations, 12 matching conformance classes, and 130 one-to-one numbered abstract tests**. There is no single Part 2 Core class. An OGC conformance claim consists of the classes a server implements and their prerequisites. That modular minimum is not the intended ceiling for a best-of-breed Glaux reference implementation.

The crucial scope correction is that approved Part 2 does **not** normatively define WebSocket or MQTT bindings, subscriptions, replay, backpressure, or a snapshot mechanism. Its scope assigns publish/subscribe protocol bindings to Part 3. A current OGC overview page says Part 2 provides streaming and snapshots, and the publication package contains an AsyncAPI draft and excluded source chapters for system history, but those materials are not requirements in OGC 23-002. Glaux may still provide excellent real-time behavior; IDR-SRV-035 must design it from the applicable Part 3 and project requirements instead of falsely attributing it to Part 2.

Part 2 also contains real publication defects. Examples include conflicting singular and plural endpoint paths, copy-and-paste references to the wrong resource, stale pre-publication notes, placeholder System Event URIs, two broken local references in the official schema package, and abstract tests that sometimes exercise the wrong resource. These defects do not make the standard unusable. They do mean Glaux needs a visible interpretation register, compatibility decisions, and tests stronger than Annex A.

The recommended direction is to carry all 12 Part 2 classes into IDR-SRV-008 as the provisional full-reference-implementation target. This is a **project recommendation**, not a statement that OGC makes every class mandatory. The create/replace/delete and update classes inherit OGC API - Features Part 4, which remains a draft; those capabilities therefore require an exact revision pin and qualified conformance language until that dependency is approved.

### 1.2 Principal Conclusions

1. **OGC 23-002 Version 1.0 is the controlling Part 2 baseline.** The approved publication, not the mutable repository branch or example API descriptions, controls normative meaning.
2. **The complete formal inventory is 12 classes and 130 numbered requirements.** Every requirement appears in §6 with its conformance test, applicability, behavior area, dependency or artifact, downstream handoff, test implication, and known issue.
3. **All numbered requirements are server-facing Web API obligations when their class and stated condition apply.** Class selection is modular; requirements inside a selected class are not a menu.
4. **Part 1 supplies the resource foundation.** Systems, Deployments, Procedures, Sampling Features, links, collections, and inherited API behavior are dependencies, not redefinitions in Part 2.
5. **Part 2's dynamic-data model is broader than observation retrieval.** A faithful implementation needs DataStream and Observation behavior, ControlStream and Command resources, feasibility, System Events, filters, writes, updates, and selected encodings.
6. **Part 2 does not supply a normative streaming protocol.** HTTP resource/history access is present; subscription protocol, delivery guarantees, ordering, replay, snapshots, and backpressure remain later design work.
7. **The resource-model tables and unboxed normative prose matter.** They include required fields and constraints not repeated as separately numbered requirements; Annex A does not prove all of them.
8. **The official JSON schemas cited by requirements are normative inputs; the packaged OpenAPI and AsyncAPI files are informative aids.** Both API descriptions identify themselves as version `0.0.1` and diverge from the approved text.
9. **Annex A is necessary but insufficient.** It has a one-to-one test identifier for every numbered requirement, yet confirmed copy/paste errors and shallow procedures require added negative, schema, lifecycle, integration, and interoperability tests.
10. **The server owns API enforcement, not physical-world truth.** Publishers, simulators, executors, controlled systems, and policy authorities provide valid inputs or authoritative outcomes; Glaux validates, persists, exposes, protects, and evidences them under later contracts.
11. **IDR-SRV-008 is technically ready but still governance-gated.** This baseline supplies its inputs; project-lead acceptance of IDR-SRV-007 is the remaining gate before it starts.

### 1.3 Recommendation for Plan-Owner Acceptance

Accept OGC 23-002 Version 1.0 as Glaux Server's controlling Part 2 requirement baseline; preserve the 130-requirement inventory; carry all 12 classes forward as the provisional reference-implementation target; pin mutable and draft dependencies; treat the official OpenAPI and AsyncAPI as informative; and require recorded interpretations for publication defects rather than silent correction.

Acceptance approves this research baseline. It does not yet freeze the final conformance profile, endpoint compatibility policy, streaming design, command safety policy, persistence architecture, Rust implementation, or any formal OGC certification claim.

---

## 2. Scope and Plan Alignment

### 2.1 In Scope

- The approved OGC 23-002 normative body, requirement classes, 130 requirement blocks, resource-model constraints, and normative Annex A.
- Dependencies on Part 1, OGC API - Features Part 4, and SWE Common 3.0.
- DataStream, Observation, ControlStream, Command, Command Status, Command Result, Feasibility, and System Event Web API behavior.
- Advanced filters, create/replace/delete, update, JSON, SWE Common JSON, SWE Common Text, and SWE Common Binary.
- Official schemas and the published OpenAPI and AsyncAPI artifacts, with authority and integrity classification.
- Server/component boundaries, downstream handoffs, ambiguities, and test implications.

### 2.2 Explicitly Out of Scope

- Final selection of Glaux's conformance classes; that is IDR-SRV-008.
- Final endpoint compatibility, content-negotiation, error, persistence, tasking, security, DDIL, or streaming design.
- Executing IDR-SRV-008 or any other research plan.
- Implementing code, selecting Rust libraries, or designing the runtime architecture.
- Treating excluded draft chapters, examples, OpenAPI, AsyncAPI, or marketing summaries as new OGC 23-002 obligations.
- A line-by-line baseline for Connected Systems Part 3; it must be researched where the later streaming plan requires it.

### 2.3 Prior-Report Reconciliation

| Accepted report | Controlling constraint carried into IDR-SRV-007 | Result |
|---|---|---|
| IDR-SRV-001 | Keep NATO adoption obligations distinct from implementation recommendations. | OGC obligations and Glaux recommendations remain separately labeled. |
| IDR-SRV-002 | The server owns interoperable API behavior; publishers, simulators, and external systems retain their component duties. | §7 assigns enforcement and integration contracts without moving physical execution into the server. |
| IDR-SRV-003 | Aim at a coherent, broad reference implementation of the adopted standards package. | All 12 classes are carried as a provisional target, subject to IDR-SRV-008. |
| IDR-SRV-004 | Preserve canonical standards terminology and record project aliases explicitly. | Resource names and requirement identifiers use OGC terms. |
| IDR-SRV-005 | Adjacent NATO material does not silently expand the server's normative baseline. | No unrelated standard is treated as a Part 2 requirement. |
| IDR-SRV-006 | Part 1 is the approved feature-resource and inherited API foundation; Features Part 4 remains draft. | Dependencies are referenced, not duplicated; draft qualification is retained. |

No contradiction with an accepted prior report was found. IDR-SRV-006 was accepted by the project lead in the instruction that authorized this research, and no later topic was begun.

**Source-backed AEP profile finding:** Accepted IDR-SRV-003 established that AEP-4789 Volume II adopts OGC 23-002 as part of the four-standard package, but found no AEP statement selecting every optional Part 2 class or encoding. AEP adoption therefore does not answer the Glaux class-profile question. The all-12-class direction in this report remains a project recommendation, and IDR-SRV-008 must make the explicit profile decision.

### 2.4 Research-Question Coverage

The five core questions are answered across §§5–12. The 41 detailed questions are mapped individually in Appendix D. Group coverage is:

| Question group | Detailed questions | Primary answer locations |
|---|---:|---|
| Requirement extraction | 6 | §§5–7, 13.2 |
| Dynamic data resources | 5 | §§6.2, 8.1, 8.4 |
| Tasking, control, feasibility | 5 | §§6.3–6.4, 7.3, 8.2 |
| System events and status | 5 | §§6.5, 8.3–8.4, 10 |
| Streaming and real-time | 5 | §§8.4, 9.4, 10–11 |
| Representations and encodings | 5 | §§6.9–6.12, 8.5, 9 |
| Error, validation, conformance | 5 | §§6.7–6.12, 8.6, 11 |
| Server boundary | 5 | §§7, 10, 12–13 |

---

## 3. Evidence Base and Authority Classification

### 3.1 Reproducible Source Register

| Evidence | Status and pin used | Authority in this report |
|---|---|---|
| [OGC 23-002 HTML](https://docs.ogc.org/is/23-002/23-002.html) and [PDF](https://docs.ogc.org/is/23-002/23-002.pdf) | Version 1.0; submitted 2025-03-19; approved 2025-06-02; published 2025-07-16; HTML SHA-256 `E840613693C282A41B1DDA709EB266905683697FB430168FF348833E8F50DF5E` | Controlling normative Part 2 text and Annex A |
| [OGC Part 2 schema ZIP](https://schemas.opengis.net/ogcapi/ogcapi-connected-systems-p2-1_0_0.zip) | 100,292 bytes; SHA-256 `02ACCC4DD11A197F029A9A65D6E9EB3724EF3E7DC8A7E6E82BC05504844100A9` | Normative where a requirement incorporates a schema; otherwise supporting artifact |
| [Official source repository](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api/part2) | tag `v1.0.0`; commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`; Part 2 tree `2d2e76e74110ad42cced8aa08c89a0af914b7d21` | Reproducible source representation of the publication |
| Part 2 OpenAPI tree | tag tree `fdad7fe138a1b07d6fe66617d5e9aebac4afa396` | Informative implementation aid, not conformance oracle |
| Part 2 AsyncAPI tree | tag tree `657af0da7307380188b4a7a722fd135d94202dd7` | Informative development artifact, not a Part 2 protocol requirement |
| [OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html) | Version 1.0 | Normative prerequisite when Part 2 inherits or references Part 1 |
| [OGC 24-014](https://docs.ogc.org/is/24-014/24-014.html) | SWE Common 3.0, approved | Normative prerequisite for selected encoding classes |
| [OGC API - Features Part 4 draft](https://docs.ogc.org/DRAFTS/20-002r1.html) | Draft at research date; development commit `9ca25f56a58ed822ea8a685a7a41afa7181aaa8b` | Unstable inherited dependency for write/update classes |
| [OGC Connected Systems standards page](https://www.ogc.org/standards/ogc-api-connected-systems/) | Current page retrieved 2026-07-31 | Secondary overview only |
| Accepted IDR-SRV-001 through IDR-SRV-006 reports | Repository `main` through merge PR #10 | Controlling project research baseline |
| [OS4CSAPI research-plan exemplar corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans) | Commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`; retrieved 2026-07-31 | Report-method/detail exemplar only; no authority over CSAPI requirements |

### 3.2 Authority Rules Applied

1. The approved OGC 23-002 body and normative Annex A control Part 2 meaning.
2. A referenced standard, table, or schema is normative only to the extent an applicable requirement incorporates it.
3. Numbered requirement blocks are formal normative requirements. Required/optional model tables and unboxed `SHALL`, `MUST`, and `MAY` prose can also be normative, but they are recorded separately rather than assigned invented requirement IDs.
4. Examples, notes, diagrams without normative language, and implementation commentary are informative.
5. OpenAPI and AsyncAPI files, example payloads, mutable branches, and overview pages cannot override the approved text.
6. Where the approved publication contradicts itself, this report records both sides and defers a controlled project interpretation; it does not silently choose a new standard.

### 3.3 Evidence Limitations

- No official executable Part 2 ETS was identified. The evidence base includes the normative abstract tests, not proof that a runnable OGC certification suite accepts a Glaux implementation.
- Features Part 4 is still draft. Its current revision can inform work but cannot be represented as a stable approved dependency.
- The approved publication and official artifacts contain confirmed inconsistencies listed in §13.2. Artifact presence is not evidence of normative status.
- The current OGC overview's streaming/snapshot description conflicts with the approved Part 2 requirement body. The report follows the approved body.
- Existing implementation behavior is intentionally deferred to IDR-SRV-014A through IDR-SRV-014G; no implementation was used to redefine a standard requirement.
- This is a requirement baseline, not a legal OGC certification determination. A future conformance claim must use the class profile, exact dependency pins, and evidence then in force.

### 3.4 Referenced-Standards Boundary

| Clause 3 reference family | How it affects this baseline |
|---|---|
| OGC API - Common, OGC API - Features, and Connected Systems Part 1 | Supplies explicitly inherited HTTP, collection, feature/resource, and parent-resource behavior. |
| SWE Common 3.0 | Supplies explicit prerequisites and component/encoding rules for C9–C12. |
| SensorML 3.0 | Supplies the Event model used by SystemEvent and applicable Procedure representation context through Part 1. |
| SOSA/SSN and OGC Observations and Measurements | Supplies semantic/model context where Part 2 clauses and mapping tables invoke it; reference-list presence alone does not create a new Glaux class. |
| ISO 8601, RFC 8259 JSON, RFC 8288 Web Linking, and JSON Schema 2020-12 | Supplies syntax, link, and schema foundations where applicable requirements or incorporated artifacts invoke them. |
| WebSocket RFC 6455 and MQTT 5.0 | Their appearance in references/protocol discussion does not create a Part 2 requirement class or abstract test; approved scope assigns publish/subscribe bindings to Part 3. |

This distinction prevents both errors: ignoring an incorporated prerequisite and turning every bibliography entry into a server feature.

---

## 4. Requirement Extraction Methodology

### 4.1 Six-Phase Execution

| Plan phase | Work performed | Output |
|---|---|---|
| 1. Collection/framework | Pinned the approved publication, tag, schemas, API artifacts, prerequisites, governance, and prior reports; established authority and inventory fields. | Source register and classification rules |
| 2. Normative extraction | Parsed every requirement class and numbered requirement; reconciled counts against Annex A and the tagged source. | 12-class, 130-requirement inventory |
| 3. Applicability/boundary | Classified obligations, class conditions, inherited dependencies, behavior areas, external contracts, and later decisions. | §7 boundary matrix |
| 4. Behavior mapping | Grouped dynamic data, tasking, feasibility, events, history, streaming boundary, and representations. | §8 behavior map |
| 5. Traceability/tests | Mapped all requirements one-to-one to abstract-test IDs and identified additional test families, fixtures, and handoffs. | §11 and inventory test codes |
| 6. Synthesis | Reconciled independent audits, defects, questions, success criteria, and recommendations into this report. | Decision-usable baseline |

### 4.2 Classification Framework

| Code | Meaning |
|---|---|
| `N` | Numbered normative Part 2 requirement |
| `NP` | Normative table or prose constraint without a separate numbered requirement ID |
| `I` | Informative material or artifact |
| `D` | Direct Glaux Web API obligation when its class applies |
| `P` | Depends on a prerequisite/inherited class or standard |
| `C` | Conditional wording or behavior within a selected class |
| `X` | Source inconsistency requires a recorded interpretation |
| `+`, `−`, `SC`, `LC`, `AT`, `IX` | Positive, negative, schema, lifecycle, abstract-conformance, and interoperability/cross-component test implications |

### 4.3 Inventory Field Model

Each §6 row carries the plan's minimum fields in compact form:

- the linked requirement ID is the source anchor;
- `C#` resolves through §5.1 to the requirement class and conformance class;
- the adjacent `/conf/...` identifier is the exact Annex A abstract test;
- `(N)` distinguishes the numbered normative requirement from informative material;
- applicability and behavior identify the Glaux Server boundary;
- dependency/artifact names Part 1, SWE, schema, or API evidence where applicable;
- the final cell supplies downstream handoff, test implication, and any issue.

Summaries normalize the obligation for planning and do not replace the controlling requirement text.

---

## 5. Part 2 Requirement-Class and Conformance-Class Inventory

### 5.1 Formal Class Inventory

| Code | Requirement class | Conformance class | Count | Formal prerequisite(s) | Provisional Glaux disposition |
|---|---|---|---:|---|---|
| C1 | `/req/api-common` | `/conf/api-common` | 2 | Part 1 `/req/api-common` | Carry |
| C2 | `/req/datastream` | `/conf/datastream` | 14 | C1 | Carry |
| C3 | `/req/controlstream` | `/conf/controlstream` | 18 | C1 | Carry |
| C4 | `/req/feasibility` | `/conf/feasibility` | 5 | C3 | Carry |
| C5 | `/req/system-event` | `/conf/system-event` | 5 | C1 and Part 1 `/req/system` | Carry |
| C6 | `/req/advanced-filtering` | `/conf/advanced-filtering` | 18 | C1 and Part 1 `/req/advanced-filtering` | Carry; resolve Annex mismatch |
| C7 | `/req/create-replace-delete` | `/conf/create-replace-delete` | 16 | OGC API - Features Part 4 `/req/create-replace-delete` | Carry with draft pin/qualification |
| C8 | `/req/update` | `/conf/update` | 14 | C7 and Features Part 4 `/req/update` | Carry with draft pin/qualification |
| C9 | `/req/json` | `/conf/json` | 14 | SWE Common 3.0 `/req/json-record-components` | Carry |
| C10 | `/req/swecommon-json` | `/conf/swecommon-json` | 8 | SWE Common 3.0 `/req/json-encoding-rules` | Carry |
| C11 | `/req/swecommon-text` | `/conf/swecommon-text` | 8 | SWE Common 3.0 `/req/text-encoding-rules` | Carry |
| C12 | `/req/swecommon-binary` | `/conf/swecommon-binary` | 8 | SWE Common 3.0 `/req/binary-encoding-rules` | Carry |
| **Total** | **12 classes** | **12 classes** | **130** | — | **Provisional; IDR-SRV-008 decides** |

`Carry` means retain the class for the next profile decision. It is not a claim that every conforming Part 2 server must implement every class.

### 5.2 Modularity and Mandatory/Conditional Meaning

**Standards obligation:** Part 2 does not define a universal Core class. A server claims conformance to the requirement classes it implements, including their prerequisites. Once a class is selected, its unconditional requirements apply; an `If ...` clause adds the stated condition rather than making the entire requirement ignorable.

**Analysis:** All 130 numbered requirements are direct Web API obligations under their class/condition. The modular choice occurs at class and capability boundaries. Encoding write requirements, collection exposure, cascade behavior, and some representation behavior include additional conditions recorded in §6.

**Project recommendation:** A broad reference server should plan for all 12 classes, then let IDR-SRV-008 define staged delivery and exact conformance statements. That preserves ambition without mislabeling the OGC minimum.

**Cross-class nuance:** Text Requirements 118/121 and Binary Requirements 126/129 incorporate the component-mapping obligations stated in JSON-class Requirements 110/113, but the Text and Binary classes do not formally inherit the SWE Common JSON class. Glaux must implement those incorporated mappings when claiming Text or Binary; it need not claim `/conf/swecommon-json` solely for that reason.

### 5.3 Annex A Reconciliation

Annex A declares 12 conformance classes and 130 conformance-test identifiers, each targeting one of the 130 numbered requirements. The counts reconcile exactly; no numbered requirement is missing an identifier and no extra numbered target was found.

That structural completeness does not establish semantic completeness. Confirmed examples include tests that exercise ControlStreams where the requirement targets System Events, copy/paste Command wording in feasibility tests, and procedures that check a route without proving all response/model constraints. The advanced-filtering requirement class inherits Part 1 advanced filtering, while its Annex A conformance class lists only Part 2 API Common. Glaux must preserve the one-to-one identifiers while supplementing their procedures.

### 5.4 Normative Prose Outside Numbered Blocks

There are **zero numbered recommendations**, but the resource-model tables and surrounding prose include `NP` constraints. Important examples are required fields, server-generated DataStream summary properties, Command status vocabulary and terminal-state descriptions, omission/default behavior on Command creation, and the rule that Command Result supplies at least one result form. These constraints are summarized in §8 and must enter later traceability with synthetic project IDs that link back to their clause/table—never counterfeit `/req/...` identifiers.

---

## 6. Normative Requirement Inventory

### 6.0 Inventory Legend

All 130 rows below are `N` (numbered normative). `C#` maps to the class and class-level conformance URI in §5.1; the `/conf/...` value is the exact Annex A test. `D/P/C/X` and test codes are defined in §4.2. A row summary is a planning index, not a substitute for the linked requirement.

### 6.1 API Common (Requirements 1–2)

| # / requirement source | Class / Annex test | Normalized obligation | Glaux applicability / behavior | Dependency or artifact | Handoff / test / issue |
|---|---|---|---|---|---|
| 1. [/req/api-common/resources](https://docs.ogc.org/is/23-002/23-002.html#_req_api-common_resources) | C1 · `/conf/api-common/resources` | (N) When applying inherited OGC API - Features Part 1/4 requirements to Part 2, interpret “feature(s)” as “resource(s).” | D/P · API foundation | Features Part 1/4 terminology | IDR-008/009 · + AT |
| 2. [/req/api-common/resource-collection](https://docs.ogc.org/is/23-002/23-002.html#_req_api-common_resource-collection) | C1 · `/conf/api-common/resource-collection` | (N) Make every Part 2 resource collection satisfy Features Part 1 Clauses 7.14–7.16 except `bbox` and `datetime`, using the resource terminology substitution. | D/P · API foundation | Features Part 1 collection behavior | IDR-008/009/010/011 · + − AT |

### 6.2 DataStreams and Observations (Requirements 3–16)

| # / requirement source | Class / Annex test | Normalized obligation | Glaux applicability / behavior | Dependency or artifact | Handoff / test / issue |
|---|---|---|---|---|---|
| 3. [/req/datastream/sf-ref-from-datastream](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_sf-ref-from-datastream) | C2 · `/conf/datastream/sf-ref-from-datastream` | (N) When the Part 1 Sampling Feature class is supported, expose linked sampling-feature subresources from each DataStream. X: the condition names nonexistent `/req/sampling`; intended class is `/req/sf`. | D/P/C · dynamic data | Part 1 links; DataStream/Observation schemas | IDR-010/017/027/034 · + − SC AT |
| 4. [/req/datastream/foi-ref-from-datastream](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_foi-ref-from-datastream) | C2 · `/conf/datastream/foi-ref-from-datastream` | (N) When the DataStream representation includes the FOI association and the server hosts those descriptions, expose only its linked FOIs as subresources. | D/P/C · dynamic data | Part 1 links; DataStream/Observation schemas | IDR-010/017/027/034 · + − SC AT |
| 5. [/req/datastream/canonical-url](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_canonical-url) | C2 · `/conf/datastream/canonical-url` | (N) Make every DataStream available at `/datastreams/{id}` and include a `canonical` link when it is retrieved through any other URL. | D/P · dynamic data | Part 1 links; DataStream/Observation schemas | IDR-010/017/027/034 · + − SC AT |
| 6. [/req/datastream/resources-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_resources-endpoint) | C2 · `/conf/datastream/resources-endpoint` | (N) Implement the DataStream resources endpoint with inherited collection retrieval behavior. | D/P · dynamic data | Part 1 links; DataStream/Observation schemas | IDR-010/017/027/034 · + − SC AT |
| 7. [/req/datastream/canonical-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_canonical-endpoint) | C2 · `/conf/datastream/canonical-endpoint` | (N) Implement the canonical `/datastreams` resources endpoint exposing all DataStreams. | D/P · dynamic data | Part 1 links; DataStream/Observation schemas | IDR-010/017/027/034 · + − SC AT |
| 8. [/req/datastream/ref-from-system](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_ref-from-system) | C2 · `/conf/datastream/ref-from-system` | (N) When the Part 1 System class is supported, expose each System's related DataStreams through its nested endpoint. | D/P/C · dynamic data | Part 1 links; DataStream/Observation schemas | IDR-010/017/027/034 · + − SC AT |
| 9. [/req/datastream/ref-from-deployment](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_ref-from-deployment) | C2 · `/conf/datastream/ref-from-deployment` | (N) When the Part 1 Deployment class is supported, expose only DataStreams whose System participated in that Deployment and whose `validTime` intersects its period. | D/P/C · dynamic data | Part 1 links; DataStream/Observation schemas | IDR-010/017/018/027/034 · + − SC AT |
| 10. [/req/datastream/collections](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_collections) | C2 · `/conf/datastream/collections` | (N) Represent exposed DataStream collections under the common resource-collection rules. | D/P · dynamic data | Part 1 links; DataStream/Observation schemas | IDR-010/017/027/034 · + − SC AT |
| 11. [/req/datastream/schema-op](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_schema-op) | C2 · `/conf/datastream/schema-op` | (N) Expose the observation schema operation for each DataStream and return its encoding schema. | D/P · dynamic data | Part 1 links; DataStream/Observation schemas | IDR-010/017/027/034 · + − SC AT |
| 12. [/req/datastream/obs-canonical-url](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_obs-canonical-url) | C2 · `/conf/datastream/obs-canonical-url` | (N) Make every Observation available at `/observations/{id}` and include a `canonical` link when it is retrieved through any other URL. | D/P · dynamic data | Part 1 links; DataStream/Observation schemas | IDR-010/017/027/034 · + − SC AT |
| 13. [/req/datastream/obs-resources-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_obs-resources-endpoint) | C2 · `/conf/datastream/obs-resources-endpoint` | (N) Implement the Observation resources endpoint with inherited retrieval behavior. | D/P · dynamic data | Part 1 links; DataStream/Observation schemas | IDR-010/017/027/034 · + − SC AT |
| 14. [/req/datastream/obs-canonical-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_obs-canonical-endpoint) | C2 · `/conf/datastream/obs-canonical-endpoint` | (N) Implement the canonical `/observations` resources endpoint exposing all Observations. | D/P · dynamic data | Part 1 links; DataStream/Observation schemas | IDR-010/017/027/034 · + − SC AT |
| 15. [/req/datastream/obs-ref-from-datastream](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_obs-ref-from-datastream) | C2 · `/conf/datastream/obs-ref-from-datastream` | (N) Expose Observations belonging to each DataStream through its nested observations endpoint. | D/P · dynamic data | Part 1 links; DataStream/Observation schemas | IDR-010/017/027/034 · + − SC AT |
| 16. [/req/datastream/obs-collections](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_obs-collections) | C2 · `/conf/datastream/obs-collections` | (N) Represent exposed Observation collections under the common collection rules. | D/P · dynamic data | Part 1 links; DataStream/Observation schemas | IDR-010/017/027/034 · + − SC AT |

### 6.3 ControlStreams, Commands, Status, and Results (Requirements 17–34)

| # / requirement source | Class / Annex test | Normalized obligation | Glaux applicability / behavior | Dependency or artifact | Handoff / test / issue |
|---|---|---|---|---|---|
| 17. [/req/controlstream/sf-ref-from-controlstream](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_sf-ref-from-controlstream) | C3 · `/conf/controlstream/sf-ref-from-controlstream` | (N) When the Part 1 Sampling Feature class is supported, expose linked sampling features from each ControlStream. X: the condition URI and copied DataStream/Observation nouns are defective. | D/P/C · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 18. [/req/controlstream/foi-ref-from-controlstream](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_foi-ref-from-controlstream) | C3 · `/conf/controlstream/foi-ref-from-controlstream` | (N) When the ControlStream representation includes the FOI association and the server hosts those descriptions, expose only its linked FOIs as subresources. | D/P/C · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 19. [/req/controlstream/canonical-url](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_canonical-url) | C3 · `/conf/controlstream/canonical-url` | (N) Make every ControlStream available at the stated canonical URL and link `canonical` from any alternate URL. X: requirement says `/controls/{id}` while the clause, other requirements, ATS, and OpenAPI use `/controlstreams/{id}`. | D/P · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 20. [/req/controlstream/resources-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_resources-endpoint) | C3 · `/conf/controlstream/resources-endpoint` | (N) Implement ControlStream collection retrieval and inherited filters. X: its `datetime` clause incorrectly calls them DataStreams. | D/P · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 21. [/req/controlstream/canonical-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_canonical-endpoint) | C3 · `/conf/controlstream/canonical-endpoint` | (N) Implement the canonical `/controlstreams` resources endpoint exposing all ControlStreams. | D/P · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 22. [/req/controlstream/ref-from-system](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_ref-from-system) | C3 · `/conf/controlstream/ref-from-system` | (N) When the Part 1 System class is supported, expose each System's related ControlStreams through its nested endpoint. | D/P/C · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 23. [/req/controlstream/ref-from-deployment](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_ref-from-deployment) | C3 · `/conf/controlstream/ref-from-deployment` | (N) When Part 1 Deployment is supported and Deployment representations provide the `controlstreams` association, expose only ControlStreams whose System participated in the Deployment and whose `validTime` intersects it. | D/P/C · tasking/control | Part 1 links; control/command schemas | IDR-010/018/036/038 · + − SC LC AT |
| 24. [/req/controlstream/collections](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_collections) | C3 · `/conf/controlstream/collections` | (N) Represent exposed ControlStream collections under the common collection rules. | D/P · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 25. [/req/controlstream/schema-op](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_schema-op) | C3 · `/conf/controlstream/schema-op` | (N) Expose the command-schema operation and return the schema for the requested supported command format. Open issue: omitted `cmdFormat` behavior is unspecified. | D/P · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 26. [/req/controlstream/cmd-canonical-url](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_cmd-canonical-url) | C3 · `/conf/controlstream/cmd-canonical-url` | (N) Make every Command available at `/commands/{id}` and include a `canonical` link when it is retrieved through any other URL. | D/P · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 27. [/req/controlstream/cmd-resources-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_cmd-resources-endpoint) | C3 · `/conf/controlstream/cmd-resources-endpoint` | (N) Implement Command collection retrieval. X: response text incorrectly says Observations. | D/P · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 28. [/req/controlstream/cmd-canonical-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_cmd-canonical-endpoint) | C3 · `/conf/controlstream/cmd-canonical-endpoint` | (N) Implement the canonical `/commands` resources endpoint exposing all Commands. | D/P · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 29. [/req/controlstream/cmd-ref-from-controlstream](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_cmd-ref-from-controlstream) | C3 · `/conf/controlstream/cmd-ref-from-controlstream` | (N) Expose Commands belonging to each ControlStream. X: requirement uses singular `/controlstream`; ATS/OpenAPI use plural `/controlstreams`. | D/P · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 30. [/req/controlstream/cmd-collections](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_cmd-collections) | C3 · `/conf/controlstream/cmd-collections` | (N) Represent exposed Command collections under the common collection rules. | D/P · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 31. [/req/controlstream/status-resources-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_status-resources-endpoint) | C3 · `/conf/controlstream/status-resources-endpoint` | (N) Expose Command Status resource collections with limit and datetime behavior. X: cited Features section does not cover both parameters. | D/P · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 32. [/req/controlstream/command-status-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_command-status-endpoint) | C3 · `/conf/controlstream/command-status-endpoint` | (N) Expose status entries for every Command. X: requirement uses singular `/command`; ATS and transaction clauses use `/commands`. | D/P · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 33. [/req/controlstream/result-resources-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_result-resources-endpoint) | C3 · `/conf/controlstream/result-resources-endpoint` | (N) Expose Command Result resource collections. | D/P · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |
| 34. [/req/controlstream/command-result-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_command-result-endpoint) | C3 · `/conf/controlstream/command-result-endpoint` | (N) Expose a result endpoint for every Command that can be associated with a result. X: requirement uses singular `/command`; ATS and transaction clauses use `/commands`. | D/P · tasking/control | Part 1 links; control/command schemas | IDR-010/036/038 · + − SC LC AT |

### 6.4 Feasibility (Requirements 35–39)

| # / requirement source | Class / Annex test | Normalized obligation | Glaux applicability / behavior | Dependency or artifact | Handoff / test / issue |
|---|---|---|---|---|---|
| 35. [/req/feasibility/canonical-url](https://docs.ogc.org/is/23-002/23-002.html#_req_feasibility_canonical-url) | C4 · `/conf/feasibility/canonical-url` | (N) Make every Feasibility resource available at `/feasibility/{id}` and link `canonical` from any alternate URL. ATS incorrectly describes a Command. | D/P · feasibility | C3; Feasibility/Command schemas | IDR-037/038 · + − SC LC AT |
| 36. [/req/feasibility/ref-from-controlstream](https://docs.ogc.org/is/23-002/23-002.html#_req_feasibility_ref-from-controlstream) | C4 · `/conf/feasibility/ref-from-controlstream` | (N) Expose feasibility submission under each ControlStream. X: singular `/controlstream` and a copied Command ATS procedure. | D/P · feasibility | C3; Feasibility/Command schemas | IDR-037/038 · + − SC LC AT |
| 37. [/req/feasibility/status-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_feasibility_status-endpoint) | C4 · `/conf/feasibility/status-endpoint` | (N) Expose `/feasibility/{feasId}/status` for every Feasibility resource. | D/P · feasibility | C3; Feasibility/Command schemas | IDR-037/038 · + − SC LC AT |
| 38. [/req/feasibility/result-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_feasibility_result-endpoint) | C4 · `/conf/feasibility/result-endpoint` | (N) Expose `/feasibility/{feasId}/result` for every Feasibility resource. | D/P · feasibility | C3; Feasibility/Command schemas | IDR-037/038 · + − SC LC AT |
| 39. [/req/feasibility/collections](https://docs.ogc.org/is/23-002/23-002.html#_req_feasibility_collections) | C4 · `/conf/feasibility/collections` | (N) If Feasibility collections are exposed, apply the common collection rules. | D/P · feasibility | C3; Feasibility/Command schemas | IDR-037/038 · + − SC LC AT |

### 6.5 System Events (Requirements 40–44)

| # / requirement source | Class / Annex test | Normalized obligation | Glaux applicability / behavior | Dependency or artifact | Handoff / test / issue |
|---|---|---|---|---|---|
| 40. [/req/system-event/canonical-url](https://docs.ogc.org/is/23-002/23-002.html#_req_system-event_canonical-url) | C5 · `/conf/system-event/canonical-url` | (N) Make every System Event available at its canonical URL and link `canonical` from any alternate URL. ATS incorrectly tests ControlStreams. | D/P · events/status | Part 1 System; Part 2/SensorML Event schemas | IDR-020/021/042 · + − SC AT |
| 41. [/req/system-event/resources-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_system-event_resources-endpoint) | C5 · `/conf/system-event/resources-endpoint` | (N) Implement System Event collection retrieval. | D/P · events/status | Part 1 System; Part 2/SensorML Event schemas | IDR-020/021/042 · + − SC AT |
| 42. [/req/system-event/canonical-endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_system-event_canonical-endpoint) | C5 · `/conf/system-event/canonical-endpoint` | (N) Implement the canonical `/systemEvents` resources endpoint exposing all System Events. ATS invokes the ControlStream endpoint test. | D/P · events/status | Part 1 System; Part 2/SensorML Event schemas | IDR-020/021/042 · + − SC AT |
| 43. [/req/system-event/ref-from-system](https://docs.ogc.org/is/23-002/23-002.html#_req_system-event_ref-from-system) | C5 · `/conf/system-event/ref-from-system` | (N) Expose a System's events at `/systems/{sysId}/events`. X: ATS uses `/systemEvents`. | D/P · events/status | Part 1 System; Part 2/SensorML Event schemas | IDR-020/021/042 · + − SC AT |
| 44. [/req/system-event/collections](https://docs.ogc.org/is/23-002/23-002.html#_req_system-event_collections) | C5 · `/conf/system-event/collections` | (N) Represent exposed System Event collections under the common collection rules. | D/P · events/status | Part 1 System; Part 2/SensorML Event schemas | IDR-020/021/042 · + − SC AT |

### 6.6 Advanced Filtering (Requirements 45–62)

| # / requirement source | Class / Annex test | Normalized obligation | Glaux applicability / behavior | Dependency or artifact | Handoff / test / issue |
|---|---|---|---|---|---|
| 45. [/req/advanced-filtering/datastream-by-phenomenontime](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_datastream-by-phenomenontime) | C6 · `/conf/advanced-filtering/datastream-by-phenomenontime` | (N) Filter DataStreams by phenomenon time. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 46. [/req/advanced-filtering/datastream-by-resulttime](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_datastream-by-resulttime) | C6 · `/conf/advanced-filtering/datastream-by-resulttime` | (N) Filter DataStreams by result time. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 47. [/req/advanced-filtering/datastream-by-obsprop](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_datastream-by-obsprop) | C6 · `/conf/advanced-filtering/datastream-by-obsprop` | (N) Filter DataStreams by observed-property identifiers. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 48. [/req/advanced-filtering/datastream-by-foi](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_datastream-by-foi) | C6 · `/conf/advanced-filtering/datastream-by-foi` | (N) Filter DataStreams by feature-of-interest identifiers. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 49. [/req/advanced-filtering/obs-by-phenomenontime](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_obs-by-phenomenontime) | C6 · `/conf/advanced-filtering/obs-by-phenomenontime` | (N) Filter Observations by phenomenon time. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 50. [/req/advanced-filtering/obs-by-resulttime](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_obs-by-resulttime) | C6 · `/conf/advanced-filtering/obs-by-resulttime` | (N) Filter Observations by result time and support the special `latest` value. ATS omits `latest`. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 51. [/req/advanced-filtering/obs-by-foi](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_obs-by-foi) | C6 · `/conf/advanced-filtering/obs-by-foi` | (N) Filter Observations by feature-of-interest identifiers. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 52. [/req/advanced-filtering/controlstream-by-issuetime](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_controlstream-by-issuetime) | C6 · `/conf/advanced-filtering/controlstream-by-issuetime` | (N) Filter ControlStreams by issue time. X: requirement calls them undefined CommandStreams. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 53. [/req/advanced-filtering/controlstream-by-exectime](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_controlstream-by-exectime) | C6 · `/conf/advanced-filtering/controlstream-by-exectime` | (N) Filter ControlStreams by execution time. X: requirement calls them CommandStreams. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 54. [/req/advanced-filtering/controlstream-by-controlprop](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_controlstream-by-controlprop) | C6 · `/conf/advanced-filtering/controlstream-by-controlprop` | (N) Filter ControlStreams by controlled-property identifiers. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 55. [/req/advanced-filtering/controlstream-by-foi](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_controlstream-by-foi) | C6 · `/conf/advanced-filtering/controlstream-by-foi` | (N) Filter ControlStreams by feature-of-interest identifiers. X: requirement calls them CommandStreams. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 56. [/req/advanced-filtering/cmd-by-issuetime](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_cmd-by-issuetime) | C6 · `/conf/advanced-filtering/cmd-by-issuetime` | (N) Filter Commands by issue time. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 57. [/req/advanced-filtering/cmd-by-exectime](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_cmd-by-exectime) | C6 · `/conf/advanced-filtering/cmd-by-exectime` | (N) Filter Commands by execution time. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 58. [/req/advanced-filtering/cmd-by-status](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_cmd-by-status) | C6 · `/conf/advanced-filtering/cmd-by-status` | (N) Filter Commands by status. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 59. [/req/advanced-filtering/cmd-by-sender](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_cmd-by-sender) | C6 · `/conf/advanced-filtering/cmd-by-sender` | (N) Filter Commands by sender. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 60. [/req/advanced-filtering/cmd-by-foi](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_cmd-by-foi) | C6 · `/conf/advanced-filtering/cmd-by-foi` | (N) Filter Commands by feature-of-interest identifiers. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 61. [/req/advanced-filtering/status-by-statuscode](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_status-by-statuscode) | C6 · `/conf/advanced-filtering/status-by-statuscode` | (N) Filter Command Status entries by status code. ATS iterates the wrong resource type. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |
| 62. [/req/advanced-filtering/event-by-type](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_event-by-type) | C6 · `/conf/advanced-filtering/event-by-type` | (N) Filter System Events by event type. ATS uses a conflicting lowercase endpoint. | D/P · query/filter | Part 1 advanced filtering; query parameters | IDR-011/018/020/027 · + − AT |

### 6.7 Create, Replace, and Delete (Requirements 63–78)

Formal condition map: Requirements 63–67 apply when C2 DataStream is implemented; 68–74 when C3 ControlStream is implemented; 75–77 when C4 Feasibility is implemented; and 78 when C5 System Event is implemented. C7 and its Features Part 4 prerequisite are already required by this class. These resource-family conditions are not independent per-row feature switches.

| # / requirement source | Class / Annex test | Normalized obligation | Glaux applicability / behavior | Dependency or artifact | Handoff / test / issue |
|---|---|---|---|---|---|
| 63. [/req/create-replace-delete/datastream](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_datastream) | C7 · `/conf/create-replace-delete/datastream` | (N) When C2 and C7 apply, create, replace, and delete DataStreams through the stated endpoints. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |
| 64. [/req/create-replace-delete/datastream-update-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_datastream-update-schema) | C7 · `/conf/create-replace-delete/datastream-update-schema` | (N) Once a DataStream has nested Observations, reject any replacement that modifies its observation schema, using HTTP 409. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |
| 65. [/req/create-replace-delete/datastream-delete-cascade](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_datastream-delete-cascade) | C7 · `/conf/create-replace-delete/datastream-delete-cascade` | (N) Default to HTTP 409 when deleting a DataStream with observations; cascade only when requested. X: body and ATS disagree on boolean semantics. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |
| 66. [/req/create-replace-delete/observation](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_observation) | C7 · `/conf/create-replace-delete/observation` | (N) When C2 and C7 apply, create, replace, and delete Observations. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |
| 67. [/req/create-replace-delete/observation-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_observation-schema) | C7 · `/conf/create-replace-delete/observation-schema` | (N) Validate Observation writes against the parent DataStream schema and reject invalid data with HTTP 400. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |
| 68. [/req/create-replace-delete/controlstream](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_controlstream) | C7 · `/conf/create-replace-delete/controlstream` | (N) When C3 and C7 apply, create, replace, and delete ControlStreams. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |
| 69. [/req/create-replace-delete/controlstream-update-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_controlstream-update-schema) | C7 · `/conf/create-replace-delete/controlstream-update-schema` | (N) Once a ControlStream has nested Commands, reject any replacement that modifies its command schema, using HTTP 409. ATS uses the wrong schema noun. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |
| 70. [/req/create-replace-delete/controlstream-delete-cascade](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_controlstream-delete-cascade) | C7 · `/conf/create-replace-delete/controlstream-delete-cascade` | (N) Default to HTTP 409 when deleting a ControlStream with commands; cascade only when requested. X: body and ATS disagree on boolean semantics. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |
| 71. [/req/create-replace-delete/command](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_command) | C7 · `/conf/create-replace-delete/command` | (N) When C3 and C7 apply, create, replace, and delete Commands. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |
| 72. [/req/create-replace-delete/command-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_command-schema) | C7 · `/conf/create-replace-delete/command-schema` | (N) Validate Command writes against the parent ControlStream schema and reject invalid data with HTTP 400. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |
| 73. [/req/create-replace-delete/command-status](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_command-status) | C7 · `/conf/create-replace-delete/command-status` | (N) When C3 and C7 apply, create, replace, and delete Command Status entries. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |
| 74. [/req/create-replace-delete/command-result](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_command-result) | C7 · `/conf/create-replace-delete/command-result` | (N) When C3 and C7 apply, create, replace, and delete Command Results. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |
| 75. [/req/create-replace-delete/feasibility](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_feasibility) | C7 · `/conf/create-replace-delete/feasibility` | (N) When C4 and C7 apply, create, replace, and delete Feasibility resources. X: nested replace/delete path omits `/{id}`; ATS adds it. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |
| 76. [/req/create-replace-delete/feasibility-status](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_feasibility-status) | C7 · `/conf/create-replace-delete/feasibility-status` | (N) When C4 and C7 apply, create, replace, and delete feasibility status entries. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |
| 77. [/req/create-replace-delete/feasibility-result](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_feasibility-result) | C7 · `/conf/create-replace-delete/feasibility-result` | (N) When C4 and C7 apply, create, replace, and delete feasibility results. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |
| 78. [/req/create-replace-delete/system-event](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_system-event) | C7 · `/conf/create-replace-delete/system-event` | (N) When C5 and C7 apply, create, replace, and delete System Events. | D/P/C · mutation | Features Part 4 draft; resource schemas | IDR-013/023/029/031 · + − SC LC AT |

### 6.8 Update (Requirements 79–92)

Formal condition map: Requirements 79–82 apply when C2 is implemented; 83–88 when C3 is implemented; 89–91 when C4 is implemented; and 92 when C5 is implemented. C8, C7, and the relevant Features Part 4 prerequisite already apply. These are resource-family conditions, not independent per-row feature switches.

| # / requirement source | Class / Annex test | Normalized obligation | Glaux applicability / behavior | Dependency or artifact | Handoff / test / issue |
|---|---|---|---|---|---|
| 79. [/req/update/datastream](https://docs.ogc.org/is/23-002/23-002.html#_req_update_datastream) | C8 · `/conf/update/datastream` | (N) When C2 and C8 apply, partially update DataStreams through the stated endpoints. | D/P/C · partial update | C7; Features Part 4 draft; schemas | IDR-013/023/029/034 · + − SC LC AT |
| 80. [/req/update/datastream-update-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_update_datastream-update-schema) | C8 · `/conf/update/datastream-update-schema` | (N) Once a DataStream has nested Observations, reject any update that modifies its observation schema. X: ATS requires 409 but the requirement omits a status. | D/P/C · partial update | C7; Features Part 4 draft; schemas | IDR-013/023/029/034 · + − SC LC AT |
| 81. [/req/update/observation](https://docs.ogc.org/is/23-002/23-002.html#_req_update_observation) | C8 · `/conf/update/observation` | (N) When C2 and C8 apply, partially update Observations. | D/P/C · partial update | C7; Features Part 4 draft; schemas | IDR-013/023/029/034 · + − SC LC AT |
| 82. [/req/update/observation-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_update_observation-schema) | C8 · `/conf/update/observation-schema` | (N) Validate Observation updates against the parent schema and reject invalid data with HTTP 400. | D/P/C · partial update | C7; Features Part 4 draft; schemas | IDR-013/023/029/034 · + − SC LC AT |
| 83. [/req/update/controlstream](https://docs.ogc.org/is/23-002/23-002.html#_req_update_controlstream) | C8 · `/conf/update/controlstream` | (N) When C3 and C8 apply, partially update ControlStreams. | D/P/C · partial update | C7; Features Part 4 draft; schemas | IDR-013/023/029/034 · + − SC LC AT |
| 84. [/req/update/controlstream-update-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_update_controlstream-update-schema) | C8 · `/conf/update/controlstream-update-schema` | (N) Once a ControlStream has nested Commands, reject any update that modifies its command schema. X: ATS adds 409 and copied Observation language. | D/P/C · partial update | C7; Features Part 4 draft; schemas | IDR-013/023/029/034 · + − SC LC AT |
| 85. [/req/update/command](https://docs.ogc.org/is/23-002/23-002.html#_req_update_command) | C8 · `/conf/update/command` | (N) When C3 and C8 apply, partially update Commands. | D/P/C · partial update | C7; Features Part 4 draft; schemas | IDR-013/023/029/034 · + − SC LC AT |
| 86. [/req/update/command-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_update_command-schema) | C8 · `/conf/update/command-schema` | (N) Validate Command updates against the parent schema and reject invalid data with HTTP 400. | D/P/C · partial update | C7; Features Part 4 draft; schemas | IDR-013/023/029/034 · + − SC LC AT |
| 87. [/req/update/command-status](https://docs.ogc.org/is/23-002/23-002.html#_req_update_command-status) | C8 · `/conf/update/command-status` | (N) When C3 and C8 apply, partially update Command Status entries. | D/P/C · partial update | C7; Features Part 4 draft; schemas | IDR-013/023/029/034 · + − SC LC AT |
| 88. [/req/update/command-result](https://docs.ogc.org/is/23-002/23-002.html#_req_update_command-result) | C8 · `/conf/update/command-result` | (N) When C3 and C8 apply, partially update Command Results. | D/P/C · partial update | C7; Features Part 4 draft; schemas | IDR-013/023/029/034 · + − SC LC AT |
| 89. [/req/update/feasibility](https://docs.ogc.org/is/23-002/23-002.html#_req_update_feasibility) | C8 · `/conf/update/feasibility` | (N) When C4 and C8 apply, partially update Feasibility resources. X: requirement contains copied Command wording. | D/P/C · partial update | C7; Features Part 4 draft; schemas | IDR-013/023/029/034 · + − SC LC AT |
| 90. [/req/update/feasibility-status](https://docs.ogc.org/is/23-002/23-002.html#_req_update_feasibility-status) | C8 · `/conf/update/feasibility-status` | (N) When C4 and C8 apply, partially update feasibility status entries. | D/P/C · partial update | C7; Features Part 4 draft; schemas | IDR-013/023/029/034 · + − SC LC AT |
| 91. [/req/update/feasibility-result](https://docs.ogc.org/is/23-002/23-002.html#_req_update_feasibility-result) | C8 · `/conf/update/feasibility-result` | (N) When C4 and C8 apply, partially update feasibility results. | D/P/C · partial update | C7; Features Part 4 draft; schemas | IDR-013/023/029/034 · + − SC LC AT |
| 92. [/req/update/system-event](https://docs.ogc.org/is/23-002/23-002.html#_req_update_system-event) | C8 · `/conf/update/system-event` | (N) When C5 and C8 apply, partially update System Events. | D/P/C · partial update | C7; Features Part 4 draft; schemas | IDR-013/023/029/034 · + − SC LC AT |

### 6.9 JSON Encoding (Requirements 93–106)

| # / requirement source | Class / Annex test | Normalized obligation | Glaux applicability / behavior | Dependency or artifact | Handoff / test / issue |
|---|---|---|---|---|---|
| 93. [/req/json/mediatype-read](https://docs.ogc.org/is/23-002/23-002.html#_req_json_mediatype-read) | C9 · `/conf/json/mediatype-read` | (N) Accept retrieval requests for `application/json` and return that encoding for applicable supported Part 2 resources. | D/P/C · JSON representation | SWE JSON record components; Part 2 schemas | IDR-012/014/022/023 · + − SC IX AT |
| 94. [/req/json/mediatype-write](https://docs.ogc.org/is/23-002/23-002.html#_req_json_mediatype-write) | C9 · `/conf/json/mediatype-write` | (N) When CRD applies, accept applicable CREATE/REPLACE resource-insertion requests encoded as `application/json`. | D/P/C · JSON representation | SWE JSON record components; Part 2 schemas | IDR-012/014/022/023 · + − SC IX AT |
| 95. [/req/json/datastream-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_json_datastream-schema) | C9 · `/conf/json/datastream-schema` | (N) Encode DataStreams with the cited JSON schema. | D/P/C · JSON representation | SWE JSON record components; Part 2 schemas | IDR-012/014/022/023 · + − SC IX AT |
| 96. [/req/json/obsschema-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_json_obsschema-schema) | C9 · `/conf/json/obsschema-schema` | (N) Encode Observation schemas with the cited JSON schema. | D/P/C · JSON representation | SWE JSON record components; Part 2 schemas | IDR-012/014/022/023 · + − SC IX AT |
| 97. [/req/json/observation-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_json_observation-schema) | C9 · `/conf/json/observation-schema` | (N) Encode Observations with the cited JSON schema. | D/P/C · JSON representation | SWE JSON record components; Part 2 schemas | IDR-012/014/022/023 · + − SC IX AT |
| 98. [/req/json/observation-constraints](https://docs.ogc.org/is/23-002/23-002.html#_req_json_observation-constraints) | C9 · `/conf/json/observation-constraints` | (N) Express phenomenon/result times in UTC with optional offset; validate `result` and optional `parameters` against the parent DataStream's result and parameter schemas. ATS omits UTC checks. | D/P/C · JSON representation | SWE JSON record components; Part 2 schemas | IDR-012/014/022/023 · + − SC IX AT |
| 99. [/req/json/controlstream-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_json_controlstream-schema) | C9 · `/conf/json/controlstream-schema` | (N) Encode ControlStreams with the cited JSON schema. | D/P/C · JSON representation | SWE JSON record components; Part 2 schemas | IDR-012/014/022/023 · + − SC IX AT |
| 100. [/req/json/commandschema-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_json_commandschema-schema) | C9 · `/conf/json/commandschema-schema` | (N) Encode Command schemas with the cited JSON schema. | D/P/C · JSON representation | SWE JSON record components; Part 2 schemas | IDR-012/014/022/023 · + − SC IX AT |
| 101. [/req/json/command-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_json_command-schema) | C9 · `/conf/json/command-schema` | (N) Encode Commands with the cited JSON schema. | D/P/C · JSON representation | SWE JSON record components; Part 2 schemas | IDR-012/014/022/023 · + − SC IX AT |
| 102. [/req/json/command-constraints](https://docs.ogc.org/is/23-002/23-002.html#_req_json_command-constraints) | C9 · `/conf/json/command-constraints` | (N) Express issue/execution times in UTC with optional offset and validate Command parameters against the parent ControlStream schema. ATS omits UTC checks. | D/P/C · JSON representation | SWE JSON record components; Part 2 schemas | IDR-012/014/022/023 · + − SC IX AT |
| 103. [/req/json/commandstatus-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_json_commandstatus-schema) | C9 · `/conf/json/commandstatus-schema` | (N) Encode Command Status with the cited JSON schema. | D/P/C · JSON representation | SWE JSON record components; Part 2 schemas | IDR-012/014/022/023 · + − SC IX AT |
| 104. [/req/json/commandresult-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_json_commandresult-schema) | C9 · `/conf/json/commandresult-schema` | (N) Encode Command Result with the cited JSON schema. | D/P/C · JSON representation | SWE JSON record components; Part 2 schemas | IDR-012/014/022/023 · + − SC IX AT |
| 105. [/req/json/commandresult-constraints](https://docs.ogc.org/is/23-002/23-002.html#_req_json_commandresult-constraints) | C9 · `/conf/json/commandresult-constraints` | (N) When a Command Result contains inline `data`, validate it against `resultSchema` for ordinary Commands or `feasibilityResultSchema` for feasibility. ATS omits the feasibility branch. | D/P/C · JSON representation | SWE JSON record components; Part 2 schemas | IDR-012/014/022/023 · + − SC IX AT |
| 106. [/req/json/systemevent-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_json_systemevent-schema) | C9 · `/conf/json/systemevent-schema` | (N) Encode System Events with the cited JSON schema. | D/P/C · JSON representation | SWE JSON record components; Part 2 schemas | IDR-012/014/022/023 · + − SC IX AT |

### 6.10 SWE Common JSON Encoding (Requirements 107–114)

| # / requirement source | Class / Annex test | Normalized obligation | Glaux applicability / behavior | Dependency or artifact | Handoff / test / issue |
|---|---|---|---|---|---|
| 107. [/req/swecommon-json/mediatype-read](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-json_mediatype-read) | C10 · `/conf/swecommon-json/mediatype-read` | (N) Accept retrieval requests for `application/swe+json` and return that encoding for applicable supported Observation/Command data. | D/P/C · SWE JSON payloads | SWE JSON encoding rules; schemas | IDR-012/022/023 · + − SC IX AT |
| 108. [/req/swecommon-json/mediatype-write](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-json_mediatype-write) | C10 · `/conf/swecommon-json/mediatype-write` | (N) When CRD applies, accept applicable CREATE/REPLACE resource-insertion requests encoded as `application/swe+json`. | D/P/C · SWE JSON payloads | SWE JSON encoding rules; schemas | IDR-012/022/023 · + − SC IX AT |
| 109. [/req/swecommon-json/obsschema-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-json_obsschema-schema) | C10 · `/conf/swecommon-json/obsschema-schema` | (N) Represent Observation schemas with the cited SWE Common schema and require their `encoding` member to be `JSONEncoding`. | D/P/C · SWE JSON payloads | SWE JSON encoding rules; schemas | IDR-012/022/023 · + − SC IX AT |
| 110. [/req/swecommon-json/obsschema-mapping](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-json_obsschema-mapping) | C10 · `/conf/swecommon-json/obsschema-mapping` | (N) Include at least one SWE `Time` component mapped by an allowed definition URI to phenomenon or result time and, when present, map Sampling Feature as `Text`. ATS omits the conditional mapping. | D/P/C · SWE JSON payloads | SWE JSON encoding rules; schemas | IDR-012/022/023 · + − SC IX AT |
| 111. [/req/swecommon-json/observation-encoding](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-json_observation-encoding) | C10 · `/conf/swecommon-json/observation-encoding` | (N) Encode Observation payloads according to their schema and SWE Common JSON rules. | D/P/C · SWE JSON payloads | SWE JSON encoding rules; schemas | IDR-012/022/023 · + − SC IX AT |
| 112. [/req/swecommon-json/cmdschema-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-json_cmdschema-schema) | C10 · `/conf/swecommon-json/cmdschema-schema` | (N) Represent Command schemas with the cited SWE Common schema and require their `encoding` member to be `JSONEncoding`. | D/P/C · SWE JSON payloads | SWE JSON encoding rules; schemas | IDR-012/022/023 · + − SC IX AT |
| 113. [/req/swecommon-json/cmdschema-mapping](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-json_cmdschema-mapping) | C10 · `/conf/swecommon-json/cmdschema-mapping` | (N) If issue time is mapped, use a SWE `Time` component with the specified `IssueTime` definition; when present, map Sampling Feature as `Text`. ATS mishandles both conditions. | D/P/C · SWE JSON payloads | SWE JSON encoding rules; schemas | IDR-012/022/023 · + − SC IX AT |
| 114. [/req/swecommon-json/command-encoding](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-json_command-encoding) | C10 · `/conf/swecommon-json/command-encoding` | (N) Encode Command payloads according to their schema and SWE Common JSON rules. | D/P/C · SWE JSON payloads | SWE JSON encoding rules; schemas | IDR-012/022/023 · + − SC IX AT |

### 6.11 SWE Common Text Encoding (Requirements 115–122)

| # / requirement source | Class / Annex test | Normalized obligation | Glaux applicability / behavior | Dependency or artifact | Handoff / test / issue |
|---|---|---|---|---|---|
| 115. [/req/swecommon-text/mediatype-read](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-text_mediatype-read) | C11 · `/conf/swecommon-text/mediatype-read` | (N) Accept retrieval requests for `application/swe+text` and return that encoding for applicable supported Observation/Command data. ATS mistakenly checks the binary media type. | D/P/C · SWE text payloads | SWE Text rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |
| 116. [/req/swecommon-text/mediatype-write](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-text_mediatype-write) | C11 · `/conf/swecommon-text/mediatype-write` | (N) When CRD applies, accept applicable CREATE/REPLACE resource-insertion requests encoded as `application/swe+text`. | D/P/C · SWE text payloads | SWE Text rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |
| 117. [/req/swecommon-text/obsschema-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-text_obsschema-schema) | C11 · `/conf/swecommon-text/obsschema-schema` | (N) Represent Observation schemas with the cited SWE Common schema and require their `encoding` member to be `TextEncoding`. | D/P/C · SWE text payloads | SWE Text rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |
| 118. [/req/swecommon-text/obsschema-mapping](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-text_obsschema-mapping) | C11 · `/conf/swecommon-text/obsschema-mapping` | (N) Make the Text `recordSchema` fulfill the Observation component mapping in Requirement 110. | D/P/C · SWE text payloads | SWE Text rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |
| 119. [/req/swecommon-text/observation-encoding](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-text_observation-encoding) | C11 · `/conf/swecommon-text/observation-encoding` | (N) Encode Observation payloads according to their schema and SWE Common Text rules. | D/P/C · SWE text payloads | SWE Text rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |
| 120. [/req/swecommon-text/cmdschema-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-text_cmdschema-schema) | C11 · `/conf/swecommon-text/cmdschema-schema` | (N) Represent Command schemas with the cited SWE Common schema and require their `encoding` member to be `TextEncoding`. | D/P/C · SWE text payloads | SWE Text rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |
| 121. [/req/swecommon-text/cmdschema-mapping](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-text_cmdschema-mapping) | C11 · `/conf/swecommon-text/cmdschema-mapping` | (N) Make the Text `recordSchema` fulfill the Command component mapping in Requirement 113. | D/P/C · SWE text payloads | SWE Text rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |
| 122. [/req/swecommon-text/command-encoding](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-text_command-encoding) | C11 · `/conf/swecommon-text/command-encoding` | (N) Encode Command payloads according to their schema and SWE Common Text rules. | D/P/C · SWE text payloads | SWE Text rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |

### 6.12 SWE Common Binary Encoding (Requirements 123–130)

| # / requirement source | Class / Annex test | Normalized obligation | Glaux applicability / behavior | Dependency or artifact | Handoff / test / issue |
|---|---|---|---|---|---|
| 123. [/req/swecommon-binary/mediatype-read](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-binary_mediatype-read) | C12 · `/conf/swecommon-binary/mediatype-read` | (N) Accept retrieval requests for `application/swe+binary` and return that encoding for applicable supported Observation/Command data. | D/P/C · SWE binary payloads | SWE Binary rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |
| 124. [/req/swecommon-binary/mediatype-write](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-binary_mediatype-write) | C12 · `/conf/swecommon-binary/mediatype-write` | (N) When CRD applies, accept applicable CREATE/REPLACE resource-insertion requests encoded as `application/swe+binary`. | D/P/C · SWE binary payloads | SWE Binary rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |
| 125. [/req/swecommon-binary/obsschema-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-binary_obsschema-schema) | C12 · `/conf/swecommon-binary/obsschema-schema` | (N) Represent Observation schemas with the cited SWE Common schema and require their `encoding` member to be `BinaryEncoding`. | D/P/C · SWE binary payloads | SWE Binary rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |
| 126. [/req/swecommon-binary/obsschema-mapping](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-binary_obsschema-mapping) | C12 · `/conf/swecommon-binary/obsschema-mapping` | (N) Make the Binary `recordSchema` fulfill the Observation component mapping in Requirement 110. | D/P/C · SWE binary payloads | SWE Binary rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |
| 127. [/req/swecommon-binary/observation-encoding](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-binary_observation-encoding) | C12 · `/conf/swecommon-binary/observation-encoding` | (N) Encode Observation payloads according to their schema and SWE Common Binary rules. ATS wrongly says Text rules. | D/P/C · SWE binary payloads | SWE Binary rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |
| 128. [/req/swecommon-binary/cmdschema-schema](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-binary_cmdschema-schema) | C12 · `/conf/swecommon-binary/cmdschema-schema` | (N) Represent Command schemas with the cited SWE Common schema and require their `encoding` member to be `BinaryEncoding`. | D/P/C · SWE binary payloads | SWE Binary rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |
| 129. [/req/swecommon-binary/cmdschema-mapping](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-binary_cmdschema-mapping) | C12 · `/conf/swecommon-binary/cmdschema-mapping` | (N) Make the Binary `recordSchema` fulfill the Command component mapping in Requirement 113. | D/P/C · SWE binary payloads | SWE Binary rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |
| 130. [/req/swecommon-binary/command-encoding](https://docs.ogc.org/is/23-002/23-002.html#_req_swecommon-binary_command-encoding) | C12 · `/conf/swecommon-binary/command-encoding` | (N) Encode Command payloads according to their schema and SWE Common Binary rules. ATS wrongly says Text rules. | D/P/C · SWE binary payloads | SWE Binary rules; mappings/schemas | IDR-012/022/023 · + − SC IX AT |

### 6.13 Count and Classification Reconciliation

| Measure | Count | Reconciliation |
|---|---:|---|
| Requirement classes | 12 | C1–C12 |
| Numbered normative requirements | 130 | Requirements 1–130, no gaps or duplicates |
| Formal numbered recommendations | 0 | No Recommendation tables were found |
| Conformance classes | 12 | One class per requirement class |
| Numbered abstract tests | 130 | A.1–A.130; exact nominal suffix mapping to requirements |
| Requirements with an explicit Conditions row | 42 | Conditions are recorded in summaries; Annex tests provide no uniform precondition/skip field |
| Resource-model/prose constraints without separate IDs | Present | Tracked as `NP`, summarized in §8, and assigned future project trace IDs |

The 42 explicit condition rows occur in Requirements 3, 4, 8, 9, 17, 18, 22, 23, all Requirements 63–92, and Requirements 94, 108, 116, and 124. Their triggers are, respectively: support for the related Part 1 association/resource; support for the applicable Part 2 resource plus the write/update class; and create/replace/delete support for write media types. Requirements 10, 16, 24, 30, 39, and 44 also use conditional wording for optional custom collections. Future trace records must retain the exact source condition, not only these grouped summaries.

**Analysis:** The inventory is formally complete, but the 130-row count must not be mistaken for the whole implementation specification. Inherited requirements and `NP` model constraints also need traceability. Conversely, excluded source chapters and informative API artifacts do not add Part 2 requirements.

---

## 7. Server Applicability and Boundary Classification

### 7.1 Class-Level Applicability Matrix

| Class group | Glaux Server responsibility | Condition or dependency | Boundary consequence |
|---|---|---|---|
| API Common | Route and represent selected CS resource types and collections. | Part 1 API Common | Direct foundation |
| DataStream/Observation | Link, retrieve, filter, validate, store, and represent stream definitions and observation data. | Part 1 parent/association resources | Direct server capability plus publisher contract |
| ControlStream/Command | Link, retrieve, validate, protect, and track control definitions, commands, statuses, and results. | Part 1 parents; execution authority outside server | Direct API capability plus safety/executor contracts |
| Feasibility | Accept and expose feasibility resources, status, and result endpoints. | ControlStream class; synchronous/asynchronous behavior allowed by prose | Direct API capability; physical feasibility comes from an authority/executor |
| System Event | Store and expose system events and filtering. | Part 1 System | Direct API capability; event truth may originate externally |
| Advanced filtering | Parse, validate, execute, and page the 18 filters. | Part 1 advanced filtering; storage indexes later | Direct query obligation, architecture deferred |
| CRD/update | Enforce mutations, schema equality/change detection and immutability after children exist, validation, 400/409, and cascade policy. | Features Part 4 draft and applicable resource class | Direct but revision-pinned; consistency/migration policy later |
| JSON/SWE encodings | Negotiate, validate, encode, and decode supported representations. | SWE Common prerequisites and complete schema graph | Direct representation obligation; resolver/vendor policy later |

### 7.2 What Clearly Belongs to Glaux Server

- HTTP routes, methods, canonical and nested-resource navigation, links, collection envelopes, filters, paging, and content negotiation.
- Stable identifiers and association integrity for Part 2 resources.
- Validation against resource and parent stream schemas, including UTC constraints and mutation rejection.
- Persistence visibility, query behavior, cascade/relationship enforcement, and correct 400/409 responses where specified.
- Command, status, result, feasibility, and event resource state as represented by the API.
- Class declarations and reproducible conformance evidence for every claimed class and inherited prerequisite.
- Authentication, authorization, audit, and command-safety enforcement once later project policies define them; Part 2 itself does not define those mechanisms.

### 7.3 Integration Contracts Without Boundary Creep

| Actor/component | Supplies or decides | Glaux Server still owns |
|---|---|---|
| Publisher | Valid observations, events, status/results, source identity, and source timestamps | Authentication, validation, accepted persistence, API exposure, rejection evidence |
| Simulator | Test-system behavior and valid synthetic dynamic data/tasking outcomes | The same API and validation contract as any producer |
| Client | Valid requests, filters, encodings, and command intent | Protocol behavior, authorization decision, validation, response semantics |
| Controlled system/executor | Physical feasibility, execution, safety interlocks, and authoritative execution outcomes | Authorized command record, dispatch/integration evidence, API status/result history |
| External identity/policy authority | Identities, credentials, policy assertions, and releasability decisions | Enforcement at the API boundary and auditable decision records |
| Broker/streaming subsystem | Delivery transport once selected | Standards-faithful HTTP resources and the later defined publication contract |

Creating a Command resource is not evidence that a device accepted or completed it. Likewise, accepting an Observation is not proof of physical truth; it is evidence that the server accepted a payload under its validation and trust policy.

### 7.4 What Part 2 Does Not Decide

OGC 23-002 does not prescribe a database, event broker, Rust framework, retention policy, idempotency key, transaction isolation level, authorization model, audit format, device protocol, delivery guarantee, replay window, backpressure algorithm, DDIL synchronization scheme, or safety approval workflow. These are legitimate Glaux design needs, but inventing them in this baseline would blur standards obligations with project engineering.

---

## 8. Dynamic-Data, Tasking, and Event Behavior Mapping

Section 6 is the exhaustive inventory of formal numbered requirements. The linked resource rows below cover every Part 2 dynamic resource's abstract attribute/association table and the additional server-relevant generation, lifecycle, and representation constraints identified in those resource-model clauses. They are a trace index, not a reproduction of every definition, example, or note; downstream traceability must split each applicable `NP` statement into its own project ID while retaining the linked clause/table source.

### 8.1 DataStreams and Observations

| Resource | Important `NP` model constraints | Relationships and generated behavior | Formal requirement groups |
|---|---|---|---|
| [DataStream](https://docs.ogc.org/is/23-002/23-002.html#clause-datastream-resource) | `name`, `phenomenonTime`, `resultTime`, `observedProperties`, `resultType`, `live`, and `formats` are required; description, type, and valid time are optional. Types are `status`/`observation`; result types are `measure`, `vector`, `record`, `coverage`, or `complex`. | System and Observations are required associations; Procedure, Deployment, Sampling Features, and Features of Interest are optional. The server derives observed properties, phenomenon/result ranges, and result type from linked observations and SHALL set them to `null` when there are none; it MAY generate `live` and ignore updates to it. | 3–11, 45–48, 63–65, 79–80, 95 |
| [Observation](https://docs.ogc.org/is/23-002/23-002.html#clause-observation-resource) | `phenomenonTime`, `resultTime`, and `result` are required; parameters are optional. | A DataStream association is required; Sampling Feature and Procedure are optional. Result and optional parameters must conform to the applicable parent observation schema fields. | 12–16, 49–51, 66–67, 81–82, 97–98, 107–111, 115–119, 123–127 |

The HTTP baseline supports current and archived observations through resource endpoints and temporal/result filters. DataStream aggregate time fields summarize available data. Part 2 does not prescribe retention duration, partitioning, downsampling, a time-series engine, or a distinct snapshot API; those belong to IDR-SRV-018, IDR-SRV-027, and IDR-SRV-030.

### 8.2 Control, Commands, Results, and Feasibility

| Resource | Important `NP` model constraints | Relationships/lifecycle | Formal requirement groups |
|---|---|---|---|
| [ControlStream](https://docs.ogc.org/is/23-002/23-002.html#clause-controlstream-resource) | `name`, issue/execution time ranges, controlled properties, `live`, `async`, and `formats` are required; description, type, and valid time are optional. Types are `self`/`external`. | System and Commands are required associations; Procedure, Deployment, Sampling Features, and Features of Interest are optional. | 17–25, 52–55, 68–70, 83–84, 99–100 |
| [Command](https://docs.ogc.org/is/23-002/23-002.html#clause-command-resource) | `issueTime`, `currentStatus`, and parameters are required when reported by the server; the first two are not required on create/update and, if supplied on creation, the prose says the server should ignore them. Execution time and sender are optional. | The association table names ControlStream, Sampling Feature, Procedure, status, and result relationships but provides no usage/cardinality column. Parameters must conform to the parent command schema. | 26–30, 56–60, 71–72, 85–86, 101–102 |
| [Command Status](https://docs.ogc.org/is/23-002/23-002.html#clause-commandstatus-resource) | Report time and status code are required; completion percentage, execution time, message, and a result list are optional. | Vocabulary includes `PENDING`, `ACCEPTED`, `REJECTED`, `SCHEDULED`, `UPDATED`, `CANCELED`, `EXECUTING`, `COMPLETED`, and `FAILED`; the prose marks rejection, cancellation, completion, and failure as final outcomes. | 31–32, 61, 73, 87, 103 |
| [Command Result](https://docs.ogc.org/is/23-002/23-002.html#clause-commandresult-resource) | May contain inline data or references to Observation(s), a DataStream, or an external resource. | At least one result form is required; when inline data is present, Requirement 105 applies the ordinary or feasibility result schema. | 33–34, 74, 88, 104–105 |
| [Feasibility](https://docs.ogc.org/is/23-002/23-002.html#clause-feasibility-resource) | A Feasibility resource is a Command sent through a feasibility channel and uses the same parameter-schema concept. | Processing may be synchronous or asynchronous; status/result reuse Command Status/Result. The status vocabulary is reused, but Table 22 marks `SCHEDULED` and `UPDATED` unused for feasibility. Part 2 does not define the external feasibility engine. | 35–39, 75–77, 89–91, inherited JSON/SWE command rules |

**Analysis:** Part 2 supplies resource states and some lifecycle vocabulary, not a complete enforceable state machine. Cancellation, transition legality, retry, idempotency, approval, execution authority, safety interlocks, and audit evidence must be designed in IDR-SRV-029 and IDR-SRV-036 through IDR-SRV-038.

### 8.3 System Events and Status

[System Events](https://docs.ogc.org/is/23-002/23-002.html#clause-systemevent-resource) require abstract `name`, `type`, and `eventTime`; description and message are optional, and the parent System association is fundamental. Part 2 says the resource is modeled on SensorML Event, while Requirement 106's incorporated JSON schema requires SensorML JSON names `label`, `definition`, and `time`. The evident abstract-to-JSON mapping is name→label, type→definition, and eventTime→time; description can use SensorML `description`, but the standard gives no explicit JSON mapping for `message`. This direct SensorML dependency and mapping gap require an interpretation in IDR-SRV-020/021/023. Requirements 40–44 cover canonical/nested retrieval and collections; Requirement 62 filters by event type. Table 20 lists seven predefined event concepts, but every URI contains `x-OGC/TBD`, so those values are not safe stable identifiers for a production enum without an interpretation/version policy.

Part 2 does not define a general system-health, availability, degraded-state, or last-known-state requirement class. Command Status describes tasking progress, not whole-system health. Dynamic feature-property and system-history source chapters exist in the repository but were excluded from the approved document. IDR-SRV-020 must therefore combine the actual Part 1/Part 2 baseline with explicit project requirements rather than claiming omitted draft text is normative.

### 8.4 History, Real-Time, Streaming, and DDIL Boundary

| Behavior | Part 2 status | Consequence |
|---|---|---|
| HTTP collection/item retrieval and temporal/filter access | Normative | Implement and test through the applicable classes. |
| Observation, Command, Command Status, Command Result, Feasibility, and System Event histories represented as resources | Supported through resources/collections; retention unspecified | Persistence/history strategy belongs to IDR-SRV-027/030. |
| `resultTime=latest` | Normative in Requirement 50 | Implement and add a supplemental test because A.50 misses it. |
| WebSocket/MQTT protocol binding, subscription, bidirectional channel | Not a Part 2 normative class; scope assigns bindings to Part 3 | Research/design in IDR-SRV-035; do not declare Part 2 conformance from AsyncAPI. |
| Ordering, replay, resumption, snapshots, QoS, backpressure | No Part 2 normative requirement | Define from Part 3 and project operational needs; test in IDR-SRV-054. |
| DDIL buffering, synchronization, conflict resolution | No Part 2 normative requirement | IDR-SRV-042/043 must define it while preserving Part 2 resource semantics. |

The OGC standards overview currently advertises streaming and snapshot behavior for Part 2. That secondary statement is not supported by the approved class/requirement/ATS set and cannot create conformance obligations absent from OGC 23-002.

### 8.5 Representations, Media Types, and Linked Resources

- JSON class: `application/json` for Part 2 resource models, using the named Part 2 JSON schemas and SWE JSON record-component prerequisite.
- SWE Common JSON: `application/swe+json` for Observation/Command payloads and schemas mapped to SWE components.
- SWE Common Text: `application/swe+text`, with the same component mappings plus Text encoding rules.
- SWE Common Binary: `application/swe+binary`, with the same component mappings plus Binary encoding rules.
- Write-media requirements 94, 108, 116, and 124 are conditional on create/replace/delete support.
- Feasibility inherits Command representation concepts rather than defining an independent encoding class.
- Resource relationships use Part 1 resources and Web links/nested endpoints; the server must keep canonical identity distinct from navigation paths.
- SensorML is not a Part 2 Observation/Command encoding. DataStream and ControlStream resources may link to Part 1 Procedure resources; SensorML 3.0 governs an applicable Procedure representation through that Part 1 dependency.

Three retained notes recommend provisional `application/vnd.ogc.swe+*` types and promise removal before publication. They are stale pre-publication notes. The numbered final media-type requirements control; any legacy alias is a later compatibility choice, not a substituted obligation.

A second media-type inconsistency concerns `application/swe+csv`: approved-body schema-operation examples and release artifacts use it, and the incorporated SWE schema documents admit a CSV/TextEncoding branch, but Part 2 defines no SWE CSV conformance class and the numbered Text requirements use `application/swe+text`. IDR-SRV-012/022 must decide whether CSV is an extension or compatibility alias and keep it outside a formal Part 2 claim unless authoritative clarification says otherwise.

### 8.6 Validation and Failure Baseline

| Trigger | Explicit Part 2 result | Further work |
|---|---|---|
| Observation or Command payload violates its parent stream schema | HTTP 400 in Requirements 67/72/82/86 | Define error body and validation detail in IDR-SRV-013/023. |
| DataStream/ControlStream replacement modifies its schema after nested children exist | HTTP 409 in Requirements 64/69; any modification is forbidden | Define exact equality/change detection plus version/new-stream migration policy; any relaxation is a divergence. |
| Equivalent partial update modifies the schema after nested children exist | Rejection required; ATS expects 409 although Requirements 80/84 omit it | Record status interpretation; use the same equality/change and migration policy. |
| Delete nonempty DataStream/ControlStream without effective cascade | HTTP 409; cascade semantics conflict between text and ATS | Adopt explicit boolean semantics through a recorded interpretation. |
| Time fields in JSON Observation/Command | UTC required | Add direct negative tests; Annex procedures omit parts of this rule. |
| Unsupported media type, invalid query, missing resource, authorization failure | Not fully specified by Part 2 | Inherit applicable HTTP/OGC rules and decide in IDR-SRV-012/013/039. |

---

## 9. Schema, OpenAPI, Encoding, and Representation Artifact Inventory

### 9.1 Published Package Inventory

The official Part 2 ZIP contains **165 files**: 159 OpenAPI-side artifacts, 5 AsyncAPI files, and 1 readme. The OpenAPI-side set comprises one root description, 48 examples, 23 parameter files, 23 path files, 16 request files, 16 response files, and 32 JSON schemas. All 32 schemas parse as JSON and declare JSON Schema Draft 2020-12; none declares `$id`. All 164 OpenAPI/AsyncAPI files that have tagged-repository counterparts are byte-for-byte identical to the `v1.0.0` Git blobs. This proves provenance, not correctness.

Annex A links 22 times to 18 unique named normative schema files, all present in the publication. A named schema is controlling only when an applicable formal requirement incorporates it; nearby examples and the rest of the directory do not become normative merely by packaging.

### 9.2 Schema-Resolution Findings

- The live directory uses `part2/1.0`; the ZIP uses `part2/1.0.0`.
- The 32-schema graph references companion Part 1, common, SensorML, and SWE Common schemas. Ten unique targets resolve to HTTP 404 when naively dereferenced from the live publication layout.
- The full tagged repository supplies most companions, but two local references are still absent: `commandOrArray.yaml` refers to a missing `command-ptz-post.json`, and an observation-schema example refers to a missing `commonDefs.json`.
- The ZIP readme is headed as Part 1, another sign that packaging metadata is not a validation authority.
- Model/JSON differences need explicit type mappings, directional/default handling, and supplemental checks wherever an effective model constraint is otherwise unenforced. Examples: `observation.json` permits omitted `phenomenonTime` with a documented default to `resultTime` and permits a linked result; `command.json` does not require response-required `currentStatus`; Command Status model `result` appears as JSON `results`; Command Result model `inline` appears as JSON `data`; and System Event abstract fields map to SensorML JSON `label`/`definition`/`time`. These are not all schema errors and must not be handled as blanket rejection rules.

**Project recommendation:** vendor and pin the complete approved companion schema graph; assign deterministic resolver aliases/base URIs; checksum it; test every reference offline; and record any repairs separately. Do not make production validation depend on live recursive dereferencing.

### 9.3 OpenAPI Findings

The research plan's `core/openapi` repository link is stale; the release artifact is under `api/part2/openapi`. The supplied OpenAPI 3.1 document calls itself an **example**, reports version `0.0.1`, and includes demo/localhost servers. It has 23 top-level paths. It omits the normative feasibility endpoints, omits other parts of the formal surface, yet includes two System History routes from a source chapter excluded from the approved standard. It also carries stale media types and two confirmed broken local references.

As an additional diagnostic—not a conformance result—Redocly CLI 2.43.2 reported 59 errors and 517 warnings under its default rules. Some are structural or example defects; 35 errors are the tool's `security-defined` policy and must not be misrepresented as Part 2 security requirements. IDR-SRV-014 must build Glaux's OpenAPI from the reconciled requirement/profile baseline, then validate it independently.

### 9.4 AsyncAPI and Protobuf Findings

The supplied AsyncAPI is version 2.6.0 with document version `0.0.1`; its tree contains one main description plus four parameter files, and the description declares five channels. It uses `controls/...` paths and media types such as `application/sml+json`, `application/om+json`, and `application/cmd+json` that do not appear in approved Part 2 requirements. The approved document never references AsyncAPI, and its scope assigns publish/subscribe bindings to Part 3. Protobuf files likewise have no Part 2 conformance class. Both are informative design evidence only.

Pinned AsyncAPI CLI 6.0.2 validation timed out twice during the independent audit, so no formal validation result is claimed.

### 9.5 Artifact-Use Policy for Glaux

| Artifact | Permitted use | Prohibited use |
|---|---|---|
| Approved standard/Annex A | Normative baseline and official test identifiers | Assuming Annex procedures prove every obligation |
| Incorporated schemas | Validation baseline with pinned resolver graph | Unpinned live dereferencing or undocumented edits |
| Tagged source | Reproduction, anchors, defect analysis | Letting mutable `master` silently change the baseline |
| OpenAPI | Examples, comparison, candidate components | Generating Glaux's contract without reconciliation |
| AsyncAPI/Protobuf | Later protocol/fixture research evidence | Claiming Part 2 streaming/Protobuf conformance |
| Examples | Fixture seeds after validation | Treating example validity as normative proof |

---

## 10. Downstream Topic Handoffs

| Topic(s) | Required handoff from this report |
|---|---|
| IDR-SRV-008 | Decide the exact 12-class profile, prerequisites, conditions, staged delivery, and declaration rules. |
| IDR-SRV-009 | Declare only implemented classes/prerequisites; reflect media types and API-definition behavior. |
| IDR-SRV-010 / 010A | Resolve canonical/nested path conflicts, link relations, collections, aliases, and versioning compatibility. |
| IDR-SRV-011 | Specify all 18 advanced filters, `latest`, invalid combinations, paging interaction, sorting/selection omissions. |
| IDR-SRV-012 | Resolve four final media types, legacy aliases, defaults, `cmdFormat` omission, and negotiation failures. |
| IDR-SRV-013 | Complete error representation and status rules around explicit 400/409 plus inherited failures. |
| IDR-SRV-014 | Produce Glaux's reconciled OpenAPI; do not adopt the `0.0.1` example as the contract. |
| IDR-SRV-014A–014G | Compare implementation behavior against the interpretation register for interoperability evidence; do not let implementation precedent override the standard. |
| IDR-SRV-018 / 027 / 030 | Define aggregate times, UTC, freshness, history storage/query, retention, deletion, and `latest`. |
| IDR-SRV-020 | Define whole-system status/availability separately from Command Status; resolve Event URIs/history. |
| IDR-SRV-021 | Preserve and reconcile Part 2 SystemEvent's direct SensorML Event-model/schema dependency. |
| IDR-SRV-022 / 023 / 024 | Implement SWE component mapping, observed-property/unit binding, schema resolver, equality/change detection, immutability/migration, and validation. |
| IDR-SRV-025 / 026 / 028 / 029 | Support filters/relationships, storage, transactions, cascade, idempotency, concurrency, and consistency. |
| IDR-SRV-031–034 | Define ingestion and publisher/simulator contracts plus dynamic-data update behavior. |
| IDR-SRV-035 | Research Part 3/project streaming protocol, delivery, replay, snapshot, and backpressure requirements. |
| IDR-SRV-036 / 037 | Define Command and Feasibility state machines, async behavior, cancellation, transition legality, and results. |
| IDR-SRV-038–041, including IDR-SRV-039A | Define command authorization/safety, authentication/authorization, zero-trust enforcement, policy, and audit; Part 2 is silent on mechanisms. |
| IDR-SRV-042 / 043 | Add DDIL buffering/synchronization/conflict semantics without altering Part 2 resource meaning. |
| IDR-SRV-050 / 051 | Preserve 130 official mappings and add profile, prerequisite, defect, and supplemental-test fields. |
| IDR-SRV-052–056 | Build unit/integration/conformance/security/performance/interoperability tests and a pinned scenario corpus. |
| IDR-SRV-057 | Synthesize only the class profile and interpretations actually accepted in later topics. |

---

## 11. Traceability and Test-Strategy Implications

### 11.1 Required Trace Record

Each implementation trace should record:

1. full normative URI and immutable publication pin;
2. class and prerequisite chain;
3. individual condition/applicability and selected profile;
4. normalized obligation plus affected route/model/encoding;
5. incorporated table/schema and local resolver pin;
6. official `/conf/...` test and whether it is usable, incomplete, contradictory, or overconstraining;
7. interpretation/compatibility decision ID;
8. implementation owner and Rust module;
9. positive, negative, schema, lifecycle, integration, security, performance, and interoperability test IDs;
10. evidence artifact and review status.

`NP` constraints should receive project trace IDs such as `GLAUX-P2-NP-...` while retaining exact clause/table anchors. That makes them testable without pretending OGC assigned identifiers it did not.

### 11.2 Required Test Families

| Test family | Minimum Part 2 coverage |
|---|---|
| Profile/prerequisite | Every claimed class, inherited class, 42 condition rows, and legal partial profile; explicit skip reason where not applicable |
| Route/navigation | Canonical, nested, collection, association membership, plural/singular compatibility decisions, absent parents/items |
| Filter/query | All 18 filters, intervals/instants, `latest`, multi-value semantics, invalid syntax, paging and empty results |
| Schema/encoding | All named schemas, complete offline `$ref` graph, UTC, parent stream validation, JSON and each selected SWE encoding |
| Mutation | Create/replace/delete/update for each selected resource; schema immutability; 400/409; cascade false/true; transaction/concurrency cases |
| Command lifecycle | Valid/invalid transitions, terminal states, status/result association, cancellation, asynchronous and failure scenarios |
| Feasibility | Canonical/submission/status/result routes; sync/async; no-result/error; separation from ordinary Command execution |
| Events/history | Event schema/type/filtering, placeholder-URI policy, ordering and retention as later specified |
| Security/safety | Authorization by operation/resource, command policy, audit evidence, information leakage, denied mutations |
| Interoperability | External clients across class declarations, paths, filters, media types, schemas, errors, and accepted compatibility aliases |
| Performance/streaming | HTTP dynamic-data load plus later Part 3 delivery, ordering, replay, backpressure, and DDIL scenarios; not labeled Part 2 ATS |

### 11.3 Annex A Gaps That Require Supplemental Tests

- A.1 has no test method; many endpoint tests prove only a happy-path 200/media type/schema response.
- Canonical tests can pass vacuously when optional collections expose no resources and often do not prove exact paths.
- Annex A has no uniform precondition fields for the 42 conditional requirements, so literal execution may overconstrain partial profiles.
- A.35, A.36, A.40, A.42, A.43, A.61, A.62, A.69, A.84, A.115, A.127, and A.130 contain wrong-resource, wrong-route, wrong-media, or wrong-rule defects.
- A.50 misses `resultTime=latest`; A.98/A.102 miss required UTC checks; A.105 misses the feasibility-result branch; A.110/A.113 miss or mishandle conditional mappings.
- Media-type write tests check API advertising but do not necessarily submit and validate payloads.
- Nested-resource tests often do not prove negative membership semantics; schema tests do not prove all model/prose constraints.

Passing the applicable official abstract-test procedures is necessary evidence for each claimed class, but is not sufficient Glaux acceptance evidence.

### 11.4 Minimum Fixture and Scenario Corpus

- Systems, Deployments, Procedures, Sampling Features, and Features of Interest with present/absent associations.
- Empty, single-value, record, array, and time-series DataStreams with valid and invalid Observations.
- Synchronous and asynchronous ControlStreams; Commands covering every status and terminal outcome; inline and referenced results.
- Feasibility accepted/rejected/pending/completed/failed cases with and without results.
- System Events across all project-approved event types and unknown extension types.
- JSON and each selected SWE encoding, malformed payloads, UTC/non-UTC times, prohibited schema-change attempts, cascade cases, and broken/unknown references.
- Large histories, filter boundaries, `latest`, pagination, concurrency, retries, authorization denials, and later DDIL/streaming scenarios.

### 11.5 Completion Evidence for One Requirement

A requirement is not complete because a route exists. Completion requires applicable-profile proof, implementation and inherited behavior, resolved artifacts, all relevant positive/negative tests, recorded interpretations, and reviewable evidence tied to the immutable source anchor.

---

## 12. Recommendations and Decision Analysis

### 12.1 Recommended Baseline Decision

Adopt OGC 23-002 Version 1.0 and this 130-row inventory as the controlling Part 2 baseline. Carry all 12 classes into IDR-SRV-008 as the provisional target for a broad open-source Rust reference implementation. Require exact pins for Features Part 4 and the schema graph, a formal interpretation register, and supplemental tests wherever Annex A is weak or defective.

### 12.2 Why This Is the Best-Fit Direction

| Option | Benefit | Cost/risk | Assessment |
|---|---|---|---|
| Minimal one-resource/one-encoding profile | Lowest initial work | Does not meet the accepted best-of-breed reference goal and leaves major interoperability surface absent | Reject as end state |
| Observation-focused subset | Useful read/data service sooner | Omits control, feasibility, events, and encoding breadth | Possible milestone only |
| All 12 classes, staged | Full standards ambition with manageable delivery gates | Highest complexity; draft dependency and defects require discipline | **Recommended** |
| Artifact-driven implementation | Fast code generation from supplied OpenAPI | Omits normative features, adds non-normative routes, and inherits defects | Reject |

### 12.3 Implementation-Planning Guardrails

1. Generate the implementation profile from §§5–6, not from OpenAPI file presence.
2. Keep literal-standard behavior, project interpretation, and compatibility alias separately traceable.
3. Make class conditions executable configuration/test metadata rather than prose remembered by an AI.
4. Vendor schemas and resolve the full dependency graph before deriving Rust types or validators.
5. Treat Commands and Feasibility as security/safety-sensitive resources from the start even though mechanisms come later.
6. Keep HTTP resource conformance separate from Part 3/project streaming guarantees.
7. Do not claim approval-level conformance to mutable Features Part 4 behavior without an exact pin and qualification.
8. Require every future design recommendation to cite the requirement, inherited source, or explicit project need it serves.

### 12.4 What Acceptance Does and Does Not Decide

Acceptance decides that the source pins, counts, classifications, defects, handoffs, and provisional all-class direction are adequate inputs to IDR-SRV-008. It does not decide the final profile; whether plural routes receive legacy aliases; default `cmdFormat`; cascade/update-status interpretations; event URI replacements; streaming protocol; lifecycle state machine; database; security policy; or Rust architecture.

---

## 13. Risks, Constraints, Ambiguities, and Open Questions

### 13.1 Principal Risks

| Risk | Consequence | Control |
|---|---|---|
| Treating a modular class as universally mandatory or optional row-by-row | False claims or missing behavior | Profile/condition matrix in IDR-SRV-008 |
| Implementing from example OpenAPI | Missing feasibility and added non-normative history behavior | Reconciled Glaux OpenAPI from requirements |
| Direct live schema dereferencing | Non-reproducible validation and 404 failures | Pinned offline schema graph and resolver tests |
| Automating defective Annex procedures literally | False pass/fail results | Official mapping plus reviewed adapters and supplemental tests |
| Silent route/media-type correction | Untraceable divergence and client failures | Interpretation register and explicit compatibility tests |
| Mistaking Command creation for execution | Unsafe or misleading tasking state | Executor boundary, lifecycle, authorization, and audit design |
| Calling AsyncAPI/HTTP history “Part 2 streaming conformance” | Incorrect standards claim | Separate Part 2, Part 3, and project evidence |
| Draft Features Part 4 changes | Write/update behavior drifts | Exact pin, change monitor, qualified declarations |
| Overlong research obscures decisions | Project lead cannot govern AI choices | Maintain the short decision brief and decision register alongside detailed evidence |

### 13.2 Standards and Artifact Interpretation Register

| ID | Source issue | Evidence/likely intent | Required owner/topic |
|---|---|---|---|
| P2-INT-001 | Req 3/17 cite nonexistent Part 1 `/req/sampling`. | Part 1 `/req/sf`; A.3/A.17 invoke `/conf/sf`. | IDR-008/017 |
| P2-INT-002 | Req 19 says `/controls/{id}`; dominant body/ATS/OpenAPI uses `/controlstreams/{id}`. | Plural form is likely intended, but the normative conflict remains. | IDR-010/014 |
| P2-INT-003 | Req 29/32/34 use singular parent paths while their tests use plural; Req 36 is singular too. | Plural is supported by the wider endpoint family; for feasibility specifically Reqs 75/89 use plural, while A.36 is a copied Command test and does not test feasibility. | IDR-010/014/037 |
| P2-INT-004 | Req 43 nested path `/systems/{id}/events` conflicts with A.43 `/systems/{id}/systemEvents`; A.62 lowercases the otherwise clear canonical `/systemEvents` path. | Reqs 40–42 establish canonical `/systemEvents` and `/systemEvents/{id}`; only the nested spelling remains unresolved. | IDR-010/020 |
| P2-INT-005 | Req 17/20/27/52/53/55/89 and several tests name wrong resource types. | Clause context identifies likely copy defects. | Relevant model topics/test harness |
| P2-INT-006 | Req 25 leaves omitted `cmdFormat` behavior undefined. | Project must choose default/error/negotiation behavior. | IDR-012/036 |
| P2-INT-007 | Req 31 mis-cites inherited datetime behavior. | Features Part 1 separates limit and datetime clauses. | IDR-011 |
| P2-INT-008 | Requirement class C6 inherits Part 1 advanced filtering; Annex conformance class omits it. | Requirement prerequisite must not be discarded. | IDR-008/050 |
| P2-INT-009 | Req 65/70 “contains cascade” conflicts with ATS boolean false/true semantics. | Boolean semantics are implementable and reflected by tests/OpenAPI. | IDR-013/029 |
| P2-INT-010 | Req 75 nested replace/delete omits item ID. | A.75 and Req 89 show an item path. | IDR-010/029/037 |
| P2-INT-011 | Req 80/84 omit status; ATS imposes 409. | 409 aligns with replace equivalents but is not literal requirement text. | IDR-013/034 |
| P2-INT-012 | Three stale SWE notes conflict with final numbered media types; artifacts/examples also use unprofiled `application/swe+csv`; A.115/A.127/A.130 are wrong. | Numbered `application/swe+*` requirements and correct SWE rule class control; CSV needs an explicit extension/alias decision. | IDR-012/022/050 |
| P2-INT-013 | Seven System Event URIs contain `x-OGC/TBD`. | Published placeholders are unsuitable as stable project identifiers. | IDR-020/024 |
| P2-INT-014 | Official schema graph is not self-contained and has broken targets. | Vendor complete companion graph with explicit mappings. | IDR-023/053 |
| P2-INT-015 | OpenAPI omits feasibility and adds excluded history; AsyncAPI is absent from approved text. | Both are informative only. | IDR-014/035 |
| P2-INT-016 | Features Part 4 prerequisite remains draft. | Pin the chosen revision and qualify claims. | IDR-008/029/050 |
| P2-INT-017 | Abstract System Event `name`/`type`/`eventTime` becomes SensorML JSON `label`/`definition`/`time`; `message` has no explicit JSON mapping. | Requirement 106 incorporates the SensorML-derived schema; Glaux needs a documented internal/JSON mapping and supplemental validation. | IDR-020/021/023 |

Other recorded defects include broken/stale source cross-references, corrupted markup in A.11/A.25, the weak/incorrect test procedures cataloged in §11.3, and duplicated wording in Requirement 105. These are harness/documentation issues unless a later decision shows an implementation consequence.

### 13.3 Open Project Decisions and Owners

| Decision still open | Owner research topic |
|---|---|
| Final Part 2 class profile and delivery stages | IDR-SRV-008 |
| Canonical path interpretation and compatibility aliases | IDR-SRV-010/010A |
| Filter edge cases and omitted sort/selection behavior | IDR-SRV-011 |
| Default formats, provisional aliases, and negotiation failures | IDR-SRV-012 |
| Error document and ambiguous 409/cascade semantics | IDR-SRV-013/029 |
| Whole-system status and stable event vocabulary | IDR-SRV-020 |
| Schema vendoring/resolver, equality/change detection, immutability, and version/new-stream migration policy | IDR-SRV-023 |
| Retention/history/last-known state | IDR-SRV-018/027/030 |
| Streaming protocol and guarantees | IDR-SRV-035/054 |
| Command/feasibility lifecycle, executor, safety, zero-trust enforcement, and audit | IDR-SRV-036–041, including IDR-SRV-039A |
| DDIL and synchronization/conflict behavior | IDR-SRV-042/043 |
| Exact conformance harness adaptations | IDR-SRV-050/051 |

No open item prevents this requirement baseline from being defensible or IDR-SRV-008 from starting after project-lead acceptance.

---

## 14. Validation Against the Research Plan

### 14.1 Methodology-Phase Validation

| Phase | Status | Evidence |
|---|---|---|
| 1. Source collection/framework | Complete | §3 pins authoritative, inherited, artifact, project, and exemplar evidence. |
| 2. Normative extraction | Complete | §§5–6 reconcile 12 classes and all 130 requirements. |
| 3. Applicability/boundary | Complete | §7 and every §6 row classify Glaux applicability, dependency, and handoff. |
| 4. Behavior mapping | Complete | §8 covers every behavior group named by the plan, including absent streaming/snapshot obligations. |
| 5. Traceability/tests | Complete | §11 maps ATS, supplemental tests, fixtures, and later test topics. |
| 6. Synthesis | Complete | §§1 and 12–13 present conclusions, recommendation, risks, interpretations, and open decisions. |

### 14.2 Success-Criterion Validation

| Plan success criterion | Status | Evidence |
|---|---|---|
| Review official Part 2 directly | Met | Approved OGC 23-002 pin and checksum in §3.1 |
| Identify requirement/conformance classes | Met | §5.1 |
| Extract server-relevant normative requirements with anchors | Met | 130 linked formal rows in §6 plus the linked model/prose constraint index in §8 |
| Distinguish normative, informative, examples, recommendations | Met | §§3.2, 5.4, 6.13, and 9 |
| Classify applicability and boundary | Met | §§6–7 |
| Identify Part 1, SensorML/SWE, and inherited dependencies | Met | §§5.1, 7, 8.3, 8.5, and inventory cells |
| Identify schema/OpenAPI/encoding artifacts | Met | §9 |
| Document all required downstream handoffs | Met | §10 |
| Capture requirement-to-test implications | Met | §6 test cells and §11 |
| List unresolved interpretation questions | Met | §13.2–13.3 |
| Make references explicit and reproducible | Met | §§3.1, 15, and Appendix C |

### 14.3 Deliverable and Inventory Validation

All 15 required content areas are present as report §§1–15. The minimum inventory fields map as follows:

| Required field | Location |
|---|---|
| Requirement ID/source anchor | §6 first column |
| Requirement class | §6 C-number, resolved in §5.1 |
| Conformance class | §5.1; exact test in §6 second column |
| Requirement summary | §6 third column |
| Normative/informative classification | (N) in §6; §3.2 and §5.4 for other material |
| Glaux applicability | §6 fourth column and §7 |
| Dynamic/tasking/status/event/API area | §6 fourth column and §8 |
| Schema/OpenAPI/encoding artifact | §6 fifth column and §9 |
| Part 1 dependency | §5.1, §6 fifth column, and §7 |
| Downstream handoff | §6 final column and §10 |
| Test implication | §6 final column and §11 |
| Notes/unresolved issues | §6 summaries and §13.2 |

The plan lists twelve minimum inventory fields; all twelve are present.

### 14.4 Independent Review and Reconciliation

Three read-only audits independently checked: (1) the normative body and requirement classes; (2) Annex A, prerequisites, conditions, and schema links; and (3) source/artifact provenance, integrity, prior-report reconciliation, and exemplar-report practices. The main extraction was then reconciled against all three.

The audits agreed on 12 classes, 130 contiguous requirements, zero formal recommendations, 12 conformance classes, and 130 nominally one-to-one tests. They also independently reproduced the principal route, copy/paste, prerequisite, media-type, test, and artifact defects. Disagreements or transcription errors were resolved against freshly hashed primary evidence; only verified values appear here. A final automated audit checks inventory counts, question coverage, local links, placeholders, and Markdown whitespace before publication.

---

## 15. References

### 15.1 Controlling and Inherited Standards

- [OGC 23-002, OGC API - Connected Systems - Part 2: Dynamic Data, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC 23-002 PDF](https://docs.ogc.org/is/23-002/23-002.pdf)
- [OGC 23-001, OGC API - Connected Systems - Part 1: Feature Resources, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC 24-014, SWE Common Data Model Encoding Standard, Version 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [OGC 23-000, SensorML Encoding Standard, Version 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC 17-069r4, OGC API - Features - Part 1: Core](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC API - Features Part 4 draft, OGC 20-002r1](https://docs.ogc.org/DRAFTS/20-002r1.html)

### 15.2 Official Artifacts and Context

- [Connected Systems official source, release v1.0.0](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api/part2)
- [Part 2 schema publication](https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/)
- [Part 2 schema ZIP](https://schemas.opengis.net/ogcapi/ogcapi-connected-systems-p2-1_0_0.zip)
- [Part 2 example OpenAPI at the release tag](https://github.com/opengeospatial/ogcapi-connected-systems/blob/v1.0.0/api/part2/openapi/openapi-connectedsystems-2.yaml)
- [Part 2 AsyncAPI at the release tag](https://github.com/opengeospatial/ogcapi-connected-systems/blob/v1.0.0/api/part2/asyncapi/asyncapi-connectedsystems-2.yaml)
- [OGC Connected Systems standards overview](https://www.ogc.org/standards/ogc-api-connected-systems/)
- [W3C SSN/SOSA](https://www.w3.org/TR/vocab-ssn/) (terminology context only)

### 15.3 Project, Governance, and Prior Research

- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-007 Research Plan](../IDR%20Plans/idr-srv-007-csapi-part-2-requirement-baseline.md)
- [Research Planning Approach](../../../../../Governance/research-planning-approach.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
- [IDR-SRV-001 Report](idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md)
- [IDR-SRV-002 Report](idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md)
- [IDR-SRV-003 Report](idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md)
- [IDR-SRV-004 Report](idr-srv-004-terminology-and-concept-crosswalk-report.md)
- [IDR-SRV-005 Report](idr-srv-005-related-nato-standards-boundary-review-report.md)
- [IDR-SRV-006 Report](idr-srv-006-csapi-part-1-requirement-baseline-report.md)
- [OS4CSAPI research-plan exemplars at pinned commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans) (report-structure context only)

---

## 16. Next Steps and Handoff

### 16.1 Current Status

Research, extraction, synthesis, validation, and repository review for `IDR-SRV-007` are complete. The Glaux Project Lead accepted this report on August 1, 2026, so it is **Final** and available as the controlling Part 2 baseline for downstream research.

The same combined plan-owner instruction authorized execution of exactly one next topic, `IDR-SRV-008`. No later topic was authorized by that transition.

### 16.2 Completed Two-Action Transition

The required workflow transition had exactly two actions:

1. The Glaux Project Lead approves IDR-SRV-007.
2. In the same instruction, the project lead authorizes execution of exactly one next plan: IDR-SRV-008.

Both actions were given in the plan owner's combined instruction that authorized IDR-SRV-008. The historical combined instruction was:

> **Approve IDR-SRV-007 and execute exactly one Glaux Server research plan: IDR-SRV-008, using the standing single-topic research, validation, status-update, commit, pull-request, review, merge, and synchronization instructions.**

---

## 17. Appendices

### 17.1 Appendix A - Count Reconciliation

| Item | Approved publication | Tagged source | Annex A | Report inventory |
|---|---:|---:|---:|---:|
| Requirement classes | 12 | 12 | — | 12 |
| Numbered requirements | 130 | 130 | 130 targets | 130 |
| Numbered recommendations | 0 | 0 | 0 | 0 |
| Conformance classes | — | 12 | 12 | 12 |
| Abstract tests | — | 130 | 130 | 130 |
| Explicit requirement Conditions rows | 42 | 42 | No uniform test preconditions | 42 classified |
| Lettered requirement table rows | 289 | 289 | — | Normalized into 130 rows |
| SHALL occurrences within requirement rows | 303 | 303 | — | Obligations summarized, not token-counted as new requirements |
| Explicit nonnormative Note blocks | 13 | 13 | — | Classified informative |

### 17.2 Appendix B - Full Normative URI Construction

The normative-provision base is:

<code>http://www.opengis.net/spec/ogcapi-connectedsystems-2/1.0</code>

Therefore the compact report identifier <code>/req/controlstream/canonical-url</code> expands to:

<code>http://www.opengis.net/spec/ogcapi-connectedsystems-2/1.0/req/controlstream/canonical-url</code>

and its nominal abstract test expands to:

<code>http://www.opengis.net/spec/ogcapi-connectedsystems-2/1.0/conf/controlstream/canonical-url</code>

Every §6 row uses this exact suffix relationship. HTML anchors use <code>#_req_</code> followed by the class and requirement suffix separated by underscores.

### 17.3 Appendix C - Reproducibility Record

Research evidence was retrieved on July 31, 2026. The reproducible procedure was:

1. retrieve the approved HTML, PDF, and schema ZIP and calculate SHA-256 and byte counts;
2. resolve the official <code>v1.0.0</code> tag and Git tree/object hashes;
3. extract requirement-class metadata and all <code>identifier:: /req/...</code> records from the tagged Part 2 source;
4. reconcile requirement number, identifier suffix, class membership, condition rows, and Annex <code>/conf/...</code> targets;
5. enumerate and parse the ZIP, compare files to Git blobs, and resolve local/live reference graphs;
6. inspect OpenAPI/AsyncAPI metadata and path/media-type coverage without treating linter policy as conformance;
7. compare results with the approved rendered publication, prior accepted reports, and three independent audits.

Pins/checksums:

- HTML: 1,376,518 bytes; SHA-256 <code>E840613693C282A41B1DDA709EB266905683697FB430168FF348833E8F50DF5E</code>
- PDF: 3,536,935 bytes; SHA-256 <code>78531C637053890DD501BB153A0046261B9C03FA064D0888A39E2B0DC383D154</code>
- schema ZIP: 100,292 bytes; SHA-256 <code>02ACCC4DD11A197F029A9A65D6E9EB3724EF3E7DC8A7E6E82BC05504844100A9</code>
- source release commit: <code>8e03b236a049849f2ccc24b4fd9fdce5ff69bed2</code>
- Part 2 tree at release and current master: <code>2d2e76e74110ad42cced8aa08c89a0af914b7d21</code>
- mutable master observed at <code>3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f</code>; its Part 2 tree was unchanged

### 17.4 Appendix D - Every Research Question Mapped

| ID | Research question | Answer location |
|---|---|---|
| CQ1 | What normative Part 2 requirements apply to Glaux? | §§5–7 and the complete §6 inventory |
| CQ2 | Which classes, resources, schemas, and behaviors belong in the baseline? | §§5, 8, and 9 |
| CQ3 | Which obligations are direct, inherited, or interpretation-dependent? | §§6–7 and 13.2 |
| CQ4 | Which later topics are affected? | §10 |
| CQ5 | What trace format preserves standard-to-code-to-test lineage? | §§4.3 and 11.1 |
| RE1 | What requirement classes are defined? | §5.1 |
| RE2 | What conformance classes are defined or referenced? | §§5.1 and 5.3 |
| RE3 | Which requirements are mandatory for a conforming server? | §5.2 and §6 |
| RE4 | Which are optional or conditional, and what triggers them? | §§5.2, 6.13, and 7.1 |
| RE5 | Which text is normative versus informative or recommendation? | §§3.2, 5.4, 6.13, and 9 |
| RE6 | Which requirements depend on Part 1 or inherited behavior? | §§5.1, 7, and inventory dependency cells |
| DD1 | What applies to DataStreams? | §§6.2 and 8.1 |
| DD2 | What applies to Observations? | §§6.2 and 8.1 |
| DD3 | What applies to status and time-varying properties? | §§8.1, 8.2, and 8.3 |
| DD4 | What applies to history, snapshots, temporal queries, or results? | §§8.1, 8.2, and 8.4 |
| DD5 | What relationships connect systems, streams, observations, properties, FOIs, and procedures? | §§7.3 and 8.1 |
| TC1 | What applies to ControlStreams? | §§6.3 and 8.2 |
| TC2 | What applies to Commands? | §§6.3 and 8.2 |
| TC3 | What applies to status, lifecycle, cancellation, rejection, and execution tracking? | §§8.2 and 10–11 |
| TC4 | What applies to feasibility and asynchronous exchanges? | §§6.4 and 8.2 |
| TC5 | What implies authorization, safety, audit, or policy handoffs? | §§7.2–7.3 and 10 |
| SE1 | What applies to System Events? | §§6.5 and 8.3 |
| SE2 | What applies to event history, retrieval, filtering, or publication? | §§6.5–6.6 and 8.3–8.4 |
| SE3 | What applies to system status, availability, degraded operation, or last-known state? | §§8.3–8.4 |
| SE4 | What affects IDR-SRV-020? | §§8.3 and 10 |
| SE5 | What affects DDIL semantics in IDR-SRV-042? | §§8.4 and 10 |
| ST1 | What Part 2 requirements apply to real-time or streaming access? | §8.4 |
| ST2 | What applies to subscriptions, publication, bidirectionality, ordering, replay, snapshots, or backpressure? | §8.4 |
| ST3 | Which behavior is protocol-specific versus neutral? | §§8.4 and 9.4 |
| ST4 | What is handed to IDR-SRV-035 and IDR-SRV-054? | §10 |
| ST5 | What streaming behavior must later be tested? | §§11.2 and 11.4 |
| RS1 | What schemas and representation models are defined or referenced? | §§8.5 and 9.1–9.2 |
| RS2 | What media types, encodings, JSON/SWE structures, SensorML references, and links apply? | §§6.9–6.12 and 8.5 |
| RS3 | Which schema/OpenAPI artifacts were reviewed directly? | §9 |
| RS4 | What affects content negotiation in IDR-SRV-012? | §§8.5 and 10 |
| RS5 | What affects validation in IDR-SRV-023? | §§8.6, 9.2, and 10 |
| EV1 | What requirements imply validation rules? | §§6.7–6.12 and 8.6 |
| EV2 | What requirements imply error responses or failure semantics? | §8.6 |
| EV3 | Which requirements need negative tests? | §§6, 11.2, and 11.3 |
| EV4 | Which requirements feed conformance-harness planning? | §11 |
| EV5 | Which requirements feed requirement-to-test traceability? | All §6 rows and §11.1 |
| SB1 | Which requirements clearly belong to Glaux Server? | §§7.1–7.2 |
| SB2 | Which imply contracts with publishers, simulators, clients, controlled systems, or externals? | §7.3 |
| SB3 | Which are outside the server boundary or deferred? | §§7.3–7.4 and 8.4 |
| SB4 | Which require a project decision? | §§12.4 and 13.2–13.3 |
| SB5 | Which may later be clarified by implementation studies? | §§10 and 13.2; IDR-SRV-014A–014G handoff |

### 17.5 Report Completion Checklist

- [x] All 5 core and 41 detailed research questions answered and mapped
- [x] All six methodology phases completed
- [x] All eleven success criteria validated
- [x] All fifteen deliverable content requirements present
- [x] All minimum inventory information fields present
- [x] Exactly 12 classes and 130 numbered requirements reconciled
- [x] Normative, inherited, informative, and project-recommendation authority separated
- [x] Evidence limitations and publication defects recorded
- [x] Independent normative, conformance, and artifact audits reconciled
- [x] Downstream and test handoffs recorded
- [x] Markdown, links, counts, placeholders, and status records validated before publication
- [x] Plan-owner acceptance and acceptance date recorded

# Section 004: Terminology and Concept Crosswalk - Research Report

**Topic ID:** IDR-SRV-004<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-004 Terminology and Concept Crosswalk](../IDR%20Plans/idr-srv-004-terminology-and-concept-crosswalk.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 29 detailed questions; the 4 objective-level prompts are also mapped<br>
**Methodology Used:** Controlled-source extraction, official-standard definition and schema review, seven-category crosswalk classification, conflict analysis, downstream handoff analysis, and independent coverage and overclaim reviews<br>
**Research Time:** Approximately 1.0 hour of AI-assisted elapsed execution time, including parallel independent reviews, on July 31, 2026<br>
**Primary Sources:**

- Project-controlling `AC/224(JCGISR)D(2026)0005`, 27 April 2026, including STANAG 4789 and AEP-4789 Volumes I and II
- OGC API - Connected Systems Parts 1 and 2, SensorML 3.0, and SWE Common 3.0
- W3C/OGC SOSA/SSN 2017 Recommendation

**Supporting Resources:** Accepted IDR-SRV-001 through IDR-SRV-003 reports, official published schemas, the Glaux Server Goal and Definition, and the project governance corpus<br>
**Document Purpose:** Establish a shared vocabulary that prevents later requirements, architecture, Rust implementation, and tests from silently assigning different meanings to the same term<br>
**Author:** OpenAI Codex, with independent read-only evidence and coverage reviews<br>
**Accepted By:** TBD pending Glaux Project Lead review<br>
**Acceptance Date:** TBD pending acceptance<br>
**Date:** July 31, 2026<br>
**Last Updated:** July 31, 2026

---

## How to Read This Report

Section 1 is the short decision brief. Section 6 is the practical vocabulary guide. The detailed evidence, crosswalk, and validation material exists so later researchers and implementers can verify every recommendation without asking the project lead to read the entire report each time.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Findings and Terminology Inventories
5. Decision Analysis
6. Recommended Glaux Server Planning Terminology
7. Server Implications and Downstream Handoffs
8. Risks, Constraints, and Open Questions
9. Validation Against the Research Plan
10. Next Steps and Handoff
11. References
12. Appendices

---

## 1. Executive Summary

### 1.1 Decision Brief

Glaux can use one consistent vocabulary without pretending that NATO operational language, OGC API resources, SensorML descriptions, SWE data structures, and SOSA/SSN concepts are identical. The governing rule is straightforward: use the exact published OGC API - Connected Systems term for an API resource or public request/response behavior; use the source standard's qualified term when discussing SensorML, SWE Common, or SOSA/SSN; and use a clearly marked **Glaux profile term** only where the adopted standards package leaves an operational concept undefined.

This matters because several ordinary-looking words carry materially different meanings. AEP-4789 defines an **observation** as resulting data, while CSAPI and SOSA define an Observation as the observing act/resource and keep its **result** separate. A CSAPI `DataStream` is an API collection of observations, while a SWE Common `DataStream` is a structure-and-encoding container. `CommandStatus` describes a command's lifecycle, not the health or readiness of a system. `Capability`, `availability`, `feasibility`, and `readiness` are related operationally but are not synonyms. A `Procedure`, a SensorML `Process`, and an executed activity are also different things. Flattening these distinctions would produce inconsistent internal data models, database schemas, validation rules, and tests.

The report therefore recommends five immediate rules:

1. Preserve the standard's exact resource names and capitalization in external API and conformance work: `System`, `Deployment`, `Procedure`, `SamplingFeature`, `Property`, `DataStream`, `Observation`, `ControlStream`, `Command`, `CommandStatus`, `CommandResult`, `Feasibility`, and `SystemEvent`.
2. Treat `System` as the canonical connected-system resource. Sensor, actuator, sampler, and platform are overlapping semantic roles, not mutually exclusive Rust entity hierarchies; the CSAPI conceptual model nevertheless requires one `systemType` URI or CURIE for the System's primary activity, and each encoding maps it to its own member.
3. Qualify overloaded internal names where needed—for example, API datastream versus SWE data stream, command-status report versus system state, and description-valid time versus phenomenon time—while preserving the standard wire names.
4. Do not invent NATO or OGC definitions for health, readiness, freshness, staleness, synchronization, trust, or DDIL behavior. They are legitimate Glaux design concerns, but their exact vocabularies and policies belong to later research topics.
5. Maintain a tracked list of known standards conflicts. Published prose, published schemas, and the 2017 SOSA/SSN Recommendation contain several version or naming mismatches that later conformance work must resolve explicitly rather than accidentally.

No technical or research finding here prevents the next topic **after this report is accepted**. The largest semantic risk is a version mismatch: the 2025 CSAPI publications use several SOSA concepts that are not present in the normative 2017 SOSA/SSN Recommendation they cite; related vocabulary appears in a 2025 First Public Working Draft, sometimes under different namespaces or maturity. Glaux should follow the approved CSAPI resource and schema contracts for API behavior, treat the 2017 Recommendation as the stable semantic authority, and record the newer draft only as standards-watch evidence. This avoids both false ontology-conformance claims and needless delay.

The project lead is being asked to accept this layered vocabulary and its downstream handoff boundaries. No Rust implementation or unresolved profile choice is being requested now.

### 1.2 Principal Conclusions

- The project-controlling STANAG/AEP ratification draft records NATO adoption intent and operational context for this project baseline; the adopted OGC standards control technical resource semantics, requirements, schemas, and behavior.
- Exact equivalence is uncommon. Most cross-source mappings are near equivalents or related-but-distinct concepts.
- The highest-risk collisions are `Observation`/result, CSAPI `DataStream`/SWE `DataStream`, `Property`/JSON property/SWE `definition`, `Procedure`/SensorML `Process`, `CommandStatus`/system status, and `validTime` used in different resource contexts.
- The project should prefer CSAPI vocabulary for the Rust server's public resource model, but this does not erase the operational terms in AEP-4789.
- The package does not provide a complete status/readiness, quality/trust, security, or DDIL vocabulary. Later topics must define bounded Glaux profile terms without presenting them as standards obligations.
- If accepted by the project lead, the crosswalk in Appendix 12.1 becomes the terminology baseline for downstream research; its mappings are semantic guidance, not an alternate ontology or wire contract.

---

## 2. Scope and Plan Alignment

### 2.1 Scope Completed

This report executes only `IDR-SRV-004`. It reviews the project-controlling STANAG/AEP package; the accepted outputs of `IDR-SRV-001` through `IDR-SRV-003`; the approved 2025 editions of CSAPI Parts 1 and 2, SensorML, and SWE Common; the stable 2017 SOSA/SSN Recommendation; relevant published schemas; and limited semantic best-practice material. It extracts terms, classifies mappings, identifies conflicts, recommends Glaux usage, and assigns downstream handoffs.

It does **not** create a complete NATO or geospatial ontology, decide the detailed Rust domain model, extract all CSAPI requirements, choose persistence structures, define operational status codes, design security policy, or specify DDIL synchronization. Those are deliberately reserved for their named research topics. No work on `IDR-SRV-005` or any later topic was begun.

The four objective-level prompts in Section 1 of the topic plan are covered by the 34 formal questions below: essential terms by `CQ-1`/`STE-*`; cross-source relationships by `CQ-2`/`CF-*`/`CE-*`; normalization by `CQ-5`/`CE-6`; and risks by `CQ-4`/`SDI-*`.

### 2.2 Research Question Coverage Matrix

| ID | Plan Question, Short Form | Status | Evidence Location |
|---|---|---|---|
| CQ-1 | Essential STANAG/AEP terms | Complete | §§4.1–4.2; Appendix 12.1 |
| CQ-2 | Mapping to standards and project terms | Complete | §§4.3–4.6; Appendix 12.1 |
| CQ-3 | Equivalence and relationship types | Complete | §4.1; Appendix 12.1 |
| CQ-4 | Server risks from terminology | Complete | §§4.6, 7.1, and 8.1 |
| CQ-5 | Controlled vocabulary and format | Complete | §6; Appendix 12.1 |
| STE-1 | Formally defined STANAG/AEP terms | Complete | §4.2.1 |
| STE-2 | Important but undefined STANAG/AEP terms | Complete | §4.2.2 |
| STE-3 | CSAPI Part 1 and Part 2 terms | Complete | §4.3.1 |
| STE-4 | SensorML and SWE Common terms | Complete | §4.3.2 |
| STE-5 | Relevant SOSA/SSN terms | Complete | §4.3.3 |
| STE-6 | Existing Glaux planning terms | Complete | §4.3.4 |
| CF-1 | Systems, roles, procedures, deployments, capabilities | Complete | §4.4.1; Appendix 12.1 rows XW-02–XW-09 |
| CF-2 | Features, sampling, and properties | Complete | §4.4.2; rows XW-10–XW-16 and XW-51–XW-52 |
| CF-3 | Dynamic data, times, status, and events | Complete | §4.4.3; rows XW-17–XW-28 |
| CF-4 | Commands, tasking, feasibility, execution | Complete | §4.4.4; rows XW-29–XW-35 and XW-53 |
| CF-5 | Description, schema, units, quality, provenance | Complete | §4.4.5; rows XW-36–XW-44 |
| CF-6 | Availability, freshness, synchronization, DDIL | Complete | §4.4.6; rows XW-45–XW-50 |
| CE-1 | Exact equivalents | Complete | §4.5; Appendix 12.1 mapping column |
| CE-2 | Near equivalents | Complete | §4.5; Appendix 12.1 |
| CE-3 | Broader/narrower relationships | Complete | §4.5; Appendix 12.1 |
| CE-4 | Related concepts that are not synonyms | Complete | §§4.4–4.6; Appendix 12.1 |
| CE-5 | Conflicts across source families | Complete | §4.6 |
| CE-6 | Preferred Glaux terms | Complete | §6 |
| SDI-1 | API path and resource naming | Complete | §§6 and 7.1 |
| SDI-2 | Database/resource-model effects | Complete | §7.1 |
| SDI-3 | Validation, schemas, and negotiation | Complete | §§4.6.2 and 7.1 |
| SDI-4 | Tasking/control semantics | Complete | §§4.4.4 and 7.1 |
| SDI-5 | Status, events, freshness, and DDIL | Complete | §§4.4.6, 7.1, and 8 |
| SDI-6 | Testing and traceability | Complete | §§7.1–7.2 |
| DU-1 | Handoff to IDR-SRV-006 and 007 | Complete | §7.2 |
| DU-2 | Handoff to IDR-SRV-015 through 020 | Complete | §7.2 |
| DU-3 | Handoff to IDR-SRV-021 through 024 | Complete | §7.2 |
| DU-4 | Dynamic data, tasking, security, DDIL, verification handoffs | Complete | §7.2 |
| DU-5 | Unresolved project decisions | Complete | §8.2 |

---

## 3. Evidence Base and Authority Classification

### 3.1 Authority Rules Used

The sources answer different kinds of questions. They were not blended into a single undifferentiated hierarchy.

1. The project-controlling NATO package governs what this IDR treats as the STANAG/AEP baseline and supplies NATO operational terms and adoption intent.
2. Each approved OGC standard governs its own normative API, resource, model, and encoding terminology.
3. Published OGC schemas are authoritative implementation artifacts within their stated role, but a prose/schema inconsistency is recorded for conformance analysis rather than silently resolved here.
4. The 2017 W3C/OGC SOSA/SSN Recommendation is the stable semantic source. A 2025 First Public Working Draft is informative standards-watch evidence only.
5. Accepted Glaux reports and governance documents control project decisions and scope, but cannot redefine a standard's term.
6. Development repositories, examples, and best-practice documents are supporting evidence only; mutable sources are pinned.

### 3.2 Primary Sources Reviewed

| Source | Version / Date / Status | Authority for This Report | Stable Anchor | Access / Limitation |
|---|---|---|---|---|
| `AC/224(JCGISR)D(2026)0005` | 27 Apr 2026; project-controlling ratification draft; SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C` | Project-controlling NATO adoption and operational source | STANAG PDF pp. 3–7; AEP-I pp. 8–37; AEP-II pp. 38–59 | Local controlled copy reviewed 2026-07-31; not public; not evidence of NATO promulgation |
| [OGC API - Connected Systems Part 1](https://docs.ogc.org/is/23-001/23-001.html) | OGC 23-001, v1.0; approved 2025-06-02; published 2025-07-16 | Normative feature-resource/API terms | §§4, 7–15, 19 | Available 2026-07-31 |
| [OGC API - Connected Systems Part 2](https://docs.ogc.org/is/23-002/23-002.html) | OGC 23-002, v1.0; approved 2025-06-02; published 2025-07-16 | Normative dynamic-data/tasking terms | §§4, 7, 9–12, 16 | Available 2026-07-31; several internal prose/schema discrepancies recorded in §4.6.2 |
| [OGC SensorML Encoding Standard](https://docs.ogc.org/is/23-000/23-000.html) | OGC 23-000, v3.0; approved 2025-06-02; published 2025-07-16 | Normative system/process-description terms | §§4, 7–8 | Available 2026-07-31 |
| [OGC SWE Common Data Model](https://docs.ogc.org/is/24-014/24-014.html) | OGC 24-014, v3.0.0; approved 2025-06-02; published 2025-07-16 | Normative data-component and encoding terms | §§4, 7–8 | Available 2026-07-31 |
| [SOSA/SSN](https://www.w3.org/TR/vocab-ssn/) | W3C Recommendation, 19 Oct 2017 | Stable semantic definitions where relevant | §§4–7 | Available 2026-07-31; does not contain several concepts used by 2025 CSAPI text |

### 3.3 Supporting Sources Reviewed

| Source | Version / Status | Evidence Class and Use | Limitation |
|---|---|---|---|
| [Official CSAPI Part 1 schemas](https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/) | Published 1.0 artifact set | Wire names and published schema structure | Does not override contrary normative prose without an interpretation decision |
| [Official CSAPI Part 2 schemas](https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/) | Published 1.0 artifact set | Wire names, OpenAPI, AsyncAPI, and examples | Example OpenAPI contains scope drift and omissions |
| [Official SensorML 3.0 JSON schemas](https://schemas.opengis.net/sensorML/3.0/json/) | Published 3.0 artifacts | Serialization vocabulary | Used only within SensorML's role |
| [Official SWE Common 3.0 JSON schemas](https://schemas.opengis.net/sweCommon/3.0/json/) | Published 3.0 artifacts | Serialization vocabulary | Used only within SWE Common's role |
| [OGC CSAPI development repository](https://github.com/opengeospatial/ogcapi-connected-systems/tree/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f) | Commit `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`, 2026-04-20 | Informative source-history and discrepancy evidence | Contains material newer than the approved 2025 publications; never used to create a normative obligation |
| [SOSA/SSN 2023 Edition](https://www.w3.org/TR/2025/WD-vocab-ssn-2023-20250916/) | First Public Working Draft, 16 Sep 2025 | Standards-watch explanation for newer SOSA terms | Work in progress; not a Recommendation and not controlling |
| [Spatial Data on the Web Best Practices](https://www.w3.org/TR/2023/DNOTE-sdw-bp-20230919/) | W3C Group Draft Note / OGC Best Practice, 2023 | Identifier, vocabulary, provenance, and conservative-equivalence guidance | Advisory, not a CSAPI conformance source |
| [Accepted IDR-SRV-001 report](idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md) | Final, accepted 2026-07-30 | NATO obligation boundary | Prior project decision; reconciled, not re-executed |
| [Accepted IDR-SRV-002 report](idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md) | Final, accepted 2026-07-31 | Operational-function vocabulary and scope | Prior project decision; reconciled, not re-executed |
| [Accepted IDR-SRV-003 report](idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md) | Final, accepted 2026-07-31 | Four-standard responsibility boundary | Prior project decision; reconciled, not re-executed |
| [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) | v1.5, 2026-07-30 | Project scope and Rust implementation goal | Project authority only |
| [Research Plan Template](../../../../../Governance/research-plan-template.md) and [Overall Research Report Template](../../../../../Governance/overall-research-report-template.md) | Current repository versions, accessed 2026-07-31 | Required governance structure and synthesis context | Project authority only |
| [OS4CSAPI research-plan corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans) | Commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230` | Research depth and structure precedent | Style evidence only |

### 3.4 Evidence Quality and Limitations

- The controlled NATO file's hash matched the registered project baseline. The enclosing memorandum and placeholders show that it is a ratification draft, not proof of promulgation; findings are therefore project-controlling, not claims about current NATO publication status.
- AEP-II directs implementers to the original standards for detailed requirements, schemas, behavior, and encodings. Accordingly, AEP terminology does not overwrite a more specific OGC definition.
- The approved CSAPI publications and their schemas are available, but identifiable contradictions remain. This report records them and delegates conformance resolution instead of choosing by convenience.
- The 2017 SOSA/SSN Recommendation is stable but older than CSAPI. The 2025 draft helps explain part of the vocabulary drift, but its definitions were not imported into the controlling baseline. In particular, the draft uses `sosa-oms:validTime` in a separate OMS extension rather than the `sosa:validTime` spelling shown by CSAPI, and its collection material is not a stable replacement contract.
- No complete normative status/readiness, security/trust, or DDIL synchronization vocabulary was found in the adopted package. Conclusions in those areas are intentionally bounded.

---

## 4. Findings and Terminology Inventories

### 4.1 Crosswalk Method

Each record preserves the source-local term, meaning, authority, and anchor before relating it to another source. The seven categories mean:

| Mapping Type | Meaning |
|---|---|
| Exact equivalent | Same material concept and scope in the stated context; wire details may still belong to one source |
| Near equivalent | Same central idea, but scope, representation, or constraints differ |
| Broader concept | The source term includes the related term and additional concepts |
| Narrower concept | The source term is a constrained subset of the related term |
| Related but distinct | Concepts interact but are not substitutes |
| Conflicting usage | The same or similar label carries incompatible meanings |
| Unresolved | Evidence does not support a defensible mapping or controlled vocabulary yet |

Directional broader/narrower mappings identify which source term is broader. “Exact” is used sparingly and never means that two standards' entire data models or serializations are interchangeable. Preferred Glaux terms govern planning language; they do not rename required JSON members or API paths.

### 4.2 STANAG/AEP Terminology Inventory

The STANAG 4789 enclosure itself (PDF pp. 3–7) was reviewed and contains no separate lexicon or server-relevant formal definitions beyond its adoption, agreement, implementation, and administrative language. The server-relevant formal definitions in the controlled package are in the AEP lexicons identified below. This is a negative extraction finding, not an inference that the STANAG enclosure was skipped.

#### 4.2.1 Formally Defined Server-Relevant Terms

| AEP Term | AEP Meaning, Condensed | Anchor | Immediate Caution |
|---|---|---|---|
| connected system | System that directly transmits data, or whose data is made available, through networks | AEP-I PDF p. 35 / Lex-2; AEP-II p. 57 / Lex-2 | Operational umbrella; not necessarily one CSAPI `System` resource |
| observation | Data resulting from observation of properties by a procedure | AEP-I p. 35; AEP-II p. 57 | Conflicts with the OGC/SOSA act/resource meaning |
| command | Request sent through a control stream to direct or modify a connected system | AEP-II p. 57 | Concrete request, not all tasking/control activity |
| command status | Information reporting a command's state or disposition | AEP-II p. 57 | Not general system health, readiness, or availability |
| control stream | Flow carrying commands to a connected system | AEP-II p. 57 | Operational flow description; CSAPI defines the API resource |
| datastream | Flow carrying observation and status data out of a connected system | AEP-II p. 57 | Not a transport protocol and not SWE Common `DataStream` |
| deployment | Information about how a system/component is deployed in operational context | AEP-II p. 57 | Information, activity, resource, and representation must be distinguished |
| feature of interest | Ultimate object, area, phenomenon, or entity subject to observation/actuation | AEP-II p. 57 | Not a sampling feature |
| feature resource | Connected-system resource exposed as a regular feature via a feature-oriented Web API | AEP-II p. 57 | API representation category, not every real-world feature |
| sampling feature | Feature associated with sampling that relates observations to a sampled subset | AEP-II pp. 57–58 | Proxy/subset, not the ultimate feature of interest |
| system | Resource representing a connected-system instance, including platform, sensor, actuator, sampler, process, model, or observer | AEP-II p. 58 | Roles are not declared mutually exclusive classes |
| system event | Information describing a notable operational-state, condition, configuration, or lifecycle change | AEP-II p. 58 | No final NATO event-code vocabulary is supplied |
| standards package | CSAPI Parts 1 and 2, SensorML, and SWE Common as a coherent package | AEP-II p. 58 | Closed to these four adopted 2025 editions for this baseline |

The AEP lexicons also define operational and architectural terms such as sensor integration, reference view, reference architecture, common intelligence feed, and common operational/tactical/intelligence pictures. They describe the wider mission outcome or governance context; none creates a same-named CSAPI server resource.

#### 4.2.2 Operationally Important but Not Fully Controlled Terms

AEP-I §§2.2–3.6 (PDF pp. 23–30) repeatedly uses identity, designation, source, provenance, lineage, quality, suitability, trust, validity, current, stale, last-known, status, health, availability, readiness, capacity, maintenance, duty cycle, tasking, control, feasibility, security, releasability, handling, federation, synchronization, and reconciliation. These terms are important inputs, but the package does not supply complete enumerations, state machines, clock rules, trust models, or synchronization protocols. They must remain qualified operational concepts or later Glaux profile terms.

The formal expansion of **DDIL** is “denied, disrupted, intermittent, and limited” (AEP-II PDF p. 56 / Lex-1). “Disconnected,” “degraded,” constrained bandwidth, and segmentation are discussed operating conditions, but they do not replace the acronym's words. “Degraded connectivity” must also remain distinct from “degraded system state.”

### 4.3 OGC, SensorML, SWE, SOSA/SSN, and Project Inventories

#### 4.3.1 CSAPI Part 1 and Part 2

| Term | Controlling Meaning | Anchor | Boundary |
|---|---|---|---|
| `System` | Infrastructure instance implementing procedures; may represent hardware, software, human, platform, sensor, actuator, or sampler | Part 1 §§4.17, 9.1–9.2 | Canonical API entity; ontology roles may overlap, but `systemType` is one required primary-activity URI/CURIE and does not imply API functionality |
| `Procedure` | Resource for reusable methodology and for a system kind/datasheet implemented by Systems | Part 1 §§4.10, 9.1 Note 1, 13.1–13.2 | Broader than only a 2017 SOSA reusable method in its datasheet/system-type use; not an execution or automatically a SensorML process instance |
| `Deployment` | Purpose-, place-, and time-specific deployment of Systems, possibly on a Platform | Part 1 §§4.4, 11 | Not software deployment; location is not the sampled feature |
| `SamplingFeature` | System-specific sample/subset/proxy connected to an ultimate feature of interest | Part 1 §§4.12, 4.14, 14 | Supports sampling chains; distinct from ultimate FOI |
| `Property` | Semantic definition usable as observable, controllable, or asserted property | Part 1 §§4.11, 15 | Non-feature resource; distinct from a JSON member or current value |
| resource `id` / `uid` | Server/type-local identifier versus URI for the underlying thing across servers | Part 1 §§8.4–8.5 | Neither is automatically the canonical resource URL |
| `DataStream` | Collection of Observations from one System sharing observed properties and result structure | Part 2 §§4.7, 9.1–9.2 | API container, not transport and not SWE `DataStream` |
| `Observation` | Act/resource applying a procedure to estimate or calculate property values, with a separate result | Part 1 §4.8; Part 2 §9.7 | AEP's resulting-data definition conflicts |
| `ControlStream` | Collection of Commands to one System sharing controlled properties and parameter structure | Part 2 §§4.6, 10.1–10.2 | Not a network transport stream |
| `Command` | Request/message asking a System to perform an actuation with desired values/parameters | Part 2 §§4.4, 10.7 | Request, not the resulting actuation |
| `CommandStatus` | Point-in-time command lifecycle/progress report | Part 2 §10.11 | Not system health/status; may have history |
| `CommandResult` | Output of command execution, inline or linked | Part 2 §10.13 | Observation is only one possible linked representation |
| `Feasibility` | Command-shaped request for analysis without submitting the operational command | Part 2 §11 | Not general capability, availability, or readiness |
| `SystemEvent` | Time-associated lifecycle or notable system event | Part 2 §12 | Separate from observations and command-status reports |

#### 4.3.2 SensorML and SWE Common

SensorML models hardware and software as **Processes** with inputs, outputs, parameters, methodology, identification, and metadata (§§4.25 and 7.1–7.2). `SimpleProcess`, `AggregateProcess`, `PhysicalComponent`, and `PhysicalSystem` are description classes, and CSAPI can use those representations for both `System` and `Procedure` resources. Endpoint, role, links, and context—not the SensorML class name alone—determine whether the CSAPI resource is an instance or reusable procedure. SensorML `Capability` describes discoverable performance or operating properties; it does not mean current availability, readiness, or command feasibility. SensorML history/events inform `SystemEvent` but are not observations or command statuses (§§8.2.4 and 8.2.8).

SWE Common separates data **semantics and structure** from **encoding** (§§7.2–7.6). A `DataComponent` describes an atomic or aggregate value, including semantic `definition`, unit, constraints, and quality. It is not a CSAPI `Property` resource. A semantic `definition` URI can identify the property represented by a component, but the URI field is not itself that property. SWE `DataStream` (§8.5.3) is a top-level open-ended values wrapper coupling element type, encoding, and values; this directly collides in name with the CSAPI API collection and must always be qualified.

#### 4.3.3 SOSA/SSN

The 2017 Recommendation supplies stable distinctions: an `Observation` is an activity and a `Result` its output; `FeatureOfInterest` is a contextual role; a `Sample` is a representative proxy, the result of Sampling, and can itself play the feature-of-interest role; a `Procedure` is reusable; a `System` implements procedures; a `Platform` hosts systems; and a `Deployment` places systems for a purpose (§§4.3–4.9). A `Property` is an intrinsic quality, not a JSON property. `SystemCapability` and `OperatingRange` describe performance and operating conditions, not current health or readiness (§5.1).

The 2017 ontology has no direct terms for CSAPI `Command`, `CommandStatus`, `ControlStream`, `Feasibility`, or `SystemEvent`, nor controlled concepts for DDIL, freshness, readiness, system health, trust, or synchronization. Its `Actuation` is the state-changing activity, not the future command request; the Recommendation explicitly excludes plans for future observations/actuations from its 2017 scope (§7.2).

CSAPI Part 1 Annex C explicitly documents further divergence from the 2017 ontology: one CSAPI `System` representation records one primary type while allowing the behaviors of overlapping sensor, actuator, sampler, and platform roles; a CSAPI `Observation` can cover multiple observed properties where the 2017 SOSA pattern has cardinality one; a CSAPI `Command` generalizes actuation and can control multiple properties; and `DataStream`/`ControlStream` depend on collection concepts absent from the 2017 Recommendation. The semantic cores remain related, but these are not exact ontology mappings.

#### 4.3.4 Existing Glaux Planning Language

The goal and overall IDR plan already use “OGC API - Connected Systems server,” “Rust,” “reference implementation,” “standards conformance,” “interoperability,” “production-ready,” “status,” “security,” and “DDIL-informed.” These remain valid project-goal phrases. They do not create new standard resource types. In particular, “DDIL-informed” is a project quality goal, not a claim that the four adopted standards supply a complete DDIL protocol.

### 4.4 Findings by Concept Family

#### 4.4.1 Systems, Roles, Procedures, Deployments, and Capabilities

`connected system` is the operational umbrella; `System` is the API resource. A real entity may carry several semantic roles at once, and a platform can also be a System when it implements procedures, even though the 2017 SOSA ontology does not make every Platform a System. CSAPI Part 1 §9.2 requires one conceptual `systemType` URI or CURIE representing the primary activity and states that this tag does not imply API functionality. Each encoding maps that attribute to its own member—for GeoJSON, `properties.featureType`. Glaux should not create disjoint root entities from the roles or emit multiple values where the model requires one.

`System` and `Procedure` preserve an instance/type-or-method distinction. A CSAPI Procedure can describe either methodology or a system kind/datasheet, so its overlap with 2017 `sosa:Procedure` is only partial. SensorML `Process` is a description/metamodel concept broad enough to model physical hardware and executable processes; it must not collapse those distinctions. `Deployment` represents purpose-, time-, and place-specific operational deployment, not deployment of the Rust software. SensorML `Capability`, SOSA `SystemCapability`, runtime availability, and CSAPI `Feasibility` answer different questions.

#### 4.4.2 Features, Sampling, and Properties

A feature resource is an API representation of a feature; the real-world feature, its geometry, the ultimate feature of interest, and a system-specific sampling feature are not synonyms. CSAPI observations and commands can associate with Sampling Features that proxy or subset the ultimate feature. Sampling chains must preserve that relationship.

CSAPI Part 1 §4.5 and SensorML §4.9 define a **Feature** as an abstraction of real-world phenomena. SensorML §4.22 separately defines a **Phenomenon** as a physical state that can be observed and whose properties can be measured. A Feature, a phenomenon, its observable Property, an API feature resource, its geometry, and an observation Result are therefore related but different layers; Glaux should not create equivalence merely from this circular-looking language.

`Property` is the most overloaded word in the corpus. It may mean the CSAPI semantic-definition resource, a SOSA/SSN intrinsic quality, a JSON/object member, a feature attribute, or the semantic concept identified by a SWE component's `definition` URI. Glaux should reserve capitalized `Property` for the CSAPI resource in API work and qualify every other use.

#### 4.4.3 Dynamic Data, Times, Status, and Events

A CSAPI `DataStream` is a collection/container for compatible observations from one System; it is not a delivery protocol, individual datum, generic operational stream, or SWE encoding object. A CSAPI `Observation` carries phenomenon time, result time, optional parameters, and a separate result. The AEP phrase “observation data” can be used operationally, but the technical object/value distinction must survive.

Time fields are not interchangeable. Phenomenon time tells when a result applies, and result time tells when the value was obtained or the activity completed. `Command.issueTime` has a published conflict: Part 2 Table 11 says when the command was received by the System, while the published JSON schema describes when it was issued and defaults an omitted value to API-request receipt. `Command.executionTime` is the actual execution period; `CommandStatus.executionTime` may be scheduled/estimated or actual depending on the status. Report time dates a command-status report, event time dates a system event, and `validTime` has resource-specific semantics such as description validity or deployment period. Ingest time, retrieval time, and freshness are additional server/policy concepts and must not be mislabeled as a standard time field.

Status data observed from a system may travel through a DataStream. `CommandStatus` reports a command's lifecycle. `SystemEvent` records a notable system change. None is a general-purpose `Status` class. System health, readiness, availability, and stale/last-known assessments require later project definitions.

#### 4.4.4 Commands, Tasking, Control, Feasibility, and Execution

NATO tasking and control are broader operational workflows. A CSAPI `Command` is the concrete request; an Actuation is the resulting state-changing activity; a `ControlStream` is the API collection for compatible commands. `CommandStatus` reports lifecycle/progress, while `CommandResult` carries output. `Feasibility` reuses command-shaped semantics to ask whether work could be done without submitting the actual operational command. It is not a capability description or a current-availability guarantee.

Cancellation is a command-lifecycle action, not resource deletion. CSAPI Part 2 §14.5 states that canceling leaves the `Command` resource on the server and is implemented by posting a new `CommandStatus` report with the exact code `CANCELED`; HTTP `DELETE` removes the resource and has different semantics. The AEP lexicon's illustrative word “cancelled” does not define the wire code or a transition graph.

#### 4.4.5 Description, Schema, Encoding, Units, Quality, and Provenance

Metadata is an umbrella, not a single resource. Descriptive metadata, schema/structure, semantic identifiers, encodings, unit declarations, constraints, quality statements, provenance, lineage, handling markings, and trust judgments must remain separable. JSON Schema can validate structure; SWE defines data components and encodings; external definition URIs supply semantic identity; none alone establishes operational suitability or trust.

SWE `uom` qualifies a quantity value and should use UCUM where possible; it is not the observed property. SWE constraints restrict a component value and do not express every cross-field or server business rule. Observation quality, provenance/lineage evidence, fitness for use, and a consumer's trust decision are related but non-equivalent.

#### 4.4.6 Availability, Freshness, Synchronization, and DDIL

AEP-I requires attention to active/inactive/degraded conditions, availability, capacity, maintenance, duty cycle, last-known state, validity, freshness, intermittent connectivity, staged exchange, later synchronization, and reconciliation. It does not specify universal codes, thresholds, state transitions, clock assumptions, conflict-resolution rules, or transport guarantees. Therefore these are not safe extensions of `CommandStatus` or `SystemEvent` codes.

Glaux should treat freshness as an assessment calculated from the applicable standard times, retrieval/ingest evidence, source-specific validity, and explicit policy. “Stale” and “last known” must state what is stale, as of what time, and under which policy. DDIL behavior must distinguish denied/disrupted/intermittent/limited connectivity from degraded system condition and from ordinary API availability.

### 4.5 Equivalence Findings

Only a small number of package-local definitions are exact conceptual equivalents to their CSAPI counterparts—most notably the AEP's `Command` and `standards package` usage in the context in which they are defined. Even there, CSAPI controls the wire model. `Deployment`, `ControlStream`, `CommandStatus`, `SystemEvent`, feature of interest, and sampling feature are near equivalents because the AEP describes operational information or flows while CSAPI supplies a more precise resource model. Feature of interest is exact only between AEP and CSAPI for the ultimate-feature role; 2017 SOSA permits a Sample to play the contextual feature-of-interest role.

Broad/narrow relationships are directional. NATO tasking/control is broader than a CSAPI Command; a CSAPI SamplingFeature is narrower than a generic feature; and a `CommandStatus` is narrower than generic status. AEP connected system and the CSAPI `System` resource are near equivalents or related-but-distinct views, not a clean superclass/subclass pair. Most remaining mappings are related but distinct. The same-label pairs `Observation`, CSAPI/SWE `DataStream`, `Property`, and general/command `status` are conflicts unless qualified.

### 4.6 Conflict, Ambiguity, and Specification-Risk Analysis

#### 4.6.1 Semantic Conflicts

| Conflict | Risk if Flattened | Required Handling |
|---|---|---|
| AEP observation data vs CSAPI/SOSA Observation act/resource vs Result value | Wrong persistence boundaries and response schemas | Use `Observation` and `result` exactly in technical contexts; say “observation data” only as an operational umbrella |
| CSAPI `DataStream` vs SWE `DataStream` | One Rust/database type accidentally mixes API collection and encoded-value container | Use qualified internal names and separate types |
| CSAPI `Property` vs intrinsic/feature/JSON property vs SWE `definition` | Semantic identifiers become ordinary fields or current values | Reserve `Property` for the CSAPI resource; qualify all others |
| CSAPI `Procedure` vs SensorML `Process` vs execution | Instance/type/execution collapse | Preserve all three concepts and their links |
| `CommandStatus` vs system status/health/readiness | Command states reused as system-health codes | Separate models and vocabularies |
| Capability vs availability vs feasibility vs readiness | Discovery claims become operational promises | Record each concept independently |
| `validTime` across resources | Description validity mistaken for system operation or observation applicability | Model and document resource-specific meanings |
| Feature, geometry, FOI, and SamplingFeature | Geometry or proxy becomes the ultimate subject | Preserve typed links and roles |
| CSAPI versus 2017 SOSA cardinality and collections | A multi-property Observation/Command or stream collection is falsely claimed to be an exact 2017 ontology pattern | Treat mappings as partial/version-sensitive; preserve the approved CSAPI contract and document semantic edition |
| DDIL, disconnected, and degraded | Connectivity and asset health states become indistinguishable | Qualify connectivity state and system state |

#### 4.6.2 Published Prose, Schema, and Vocabulary Mismatches

| Issue | Evidence | Consequence / Owner |
|---|---|---|
| SOSA edition and namespace mismatch | CSAPI mixes 2017 `ssn:*` terms with names absent from or moved in that Recommendation, including `sosa:System`, `sosa:Deployment`, `sosa:Property`, `sosa:hasSubSystem`, `sosa:hasDeployment`, `sosa:implements`, collection concepts, `sosa:validTime`, `sosa:hasSampledFeature`, and `ssn:hasSystemKind`. The 2025 First Public Working Draft explains only part of the drift: it uses `sosa-oms:validTime`, and collection material is not a stable normative replacement. | Follow CSAPI's approved API contract; do not claim complete 2017 ontology conformance; inventory exact terms and namespaces in 006–008, 015, and 024 |
| `Command.issueTime` meaning | Part 2 §10.7.1 Table 11 says time received by the System; published [`command.json`](https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/openapi/schemas/json/command.json) says time issued and defaults an omitted value to API-request receipt | Preserve the discrepancy; select and test an interpretation in 007–008 and 036 |
| Command-status result member name | Part 2 §10.11 Table 13 describes `result`; published [`commandStatus.json`](https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/openapi/schemas/json/commandStatus.json) exposes `results` | Explicit conformance/errata decision in 007–008; test both evidence layers |
| CommandResult conceptual vs wire labels | Part 2 §10.13 Table 16 and published [`commandResult.json`](https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/openapi/schemas/json/commandResult.json) use different conceptual/member names | Keep conceptual and serialization vocabulary separate; 007, 008, 036 |
| SystemEvent conceptual vs SensorML wire labels | Part 2 §12.2.1 Table 19 uses `name`, `type`, `eventTime`, `message`; published [`systemEvent.json`](https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/openapi/schemas/json/systemEvent.json) inherits SensorML Event with `label`, `definition`, `time` | Map deliberately; do not rename by intuition; 007, 020, 008 |
| Control/command path singular-plural inconsistencies | Part 2 §7.4 and the example use `/controlstreams`; Requirement 19 uses `/controls`; Requirements 29, 32, and 34 contain singular `/controlstream` or `/command` forms while the published [example OpenAPI](https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/openapi/openapi-connectedsystems-2.yaml) generally uses plural paths | Preserve as tracked standards discrepancy; decide in 007–008 before implementation |
| Example OpenAPI scope drift | The published [example OpenAPI](https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/openapi/openapi-connectedsystems-2.yaml) includes System History paths not clearly in approved Part 2 and omits Feasibility endpoints | Treat example as informative; normative requirements and published schemas need separate trace rows |
| `Property` feature classification tension | Part 1 says Property is the exception to feature resources; Part 2 broadly describes Part 1 resources as features | Follow the more specific Part 1 resource rule unless later conformance research establishes otherwise |
| Provisional SystemEvent identifiers | Part 2 Table 20 uses `http://www.opengis.net/def/x-OGC/TBD/...` values | Do not hard-code them as a final governed vocabulary; 020 and 024 |

These mismatches are findings, not authority to redesign the standard. Later requirement and conformance topics must create an explicit resolution record with the selected behavior, evidence, interoperability impact, and test coverage.

---

## 5. Decision Analysis

### 5.1 Vocabulary Strategy Options

| Option | Benefit | Cost / Risk | Standards and Compatibility Effect | Decision |
|---|---|---|---|---|
| Use NATO operational terms everywhere | Familiar to the mission community | Loses exact OGC resource and schema meanings | High risk of non-conformant resources and tests | Reject |
| Use OGC terms everywhere and discard AEP language | Clean API vocabulary | Erases operational objectives, status/DDIL concerns, and stakeholder language | Technically narrow and operationally incomplete | Reject |
| Create one new Glaux ontology now | Could centralize names | Large, premature scope; likely duplicates or contradicts standards | Adds a non-standard semantic layer before requirements are known | Reject |
| Layered vocabulary: CSAPI for API, source-qualified SensorML/SWE/SOSA terms, AEP for operational intent, Glaux profile terms only for gaps | Preserves authority and gives implementers unambiguous names | Requires discipline and a maintained glossary/discrepancy register | Best conformance, traceability, and interoperability posture | **Recommend acceptance** |

### 5.2 Decisions Recommended for Project-Lead Acceptance

1. **API vocabulary:** CSAPI resource names and meanings are preferred for the server's public API, requirement IDs, conformance mapping, and wire-contract discussion.
2. **Operational vocabulary:** AEP terms remain authoritative descriptions of NATO operational intent within this IDR baseline; they are not used to rename technical resources.
3. **Qualified standard terms:** SensorML, SWE Common, and SOSA/SSN names retain their source qualifier wherever a bare name could be confused with a CSAPI resource.
4. **Project additions:** A term absent from the adopted standards package may be defined later as a Glaux profile term, with an owner, scope, source rationale, and tests.
5. **Equivalence discipline:** Exact equivalence will not be asserted merely because labels resemble one another. Strong machine-level equivalence such as `owl:sameAs` is outside this topic and should not be used without explicit semantic proof.
6. **Version discipline:** CSAPI API behavior follows the approved 2025 publications and published artifacts subject to later conformance decisions; SOSA/SSN semantic claims identify the edition used.

### 5.3 Decisions Explicitly Deferred

This report does not select final status/health/readiness codes; freshness thresholds; event-type registry values; schema-versus-prose resolution policy; database table/type names; Rust module boundaries; security and release markings; synchronization/conflict algorithms; or the project's final comprehensive glossary. It identifies the vocabulary constraints those decisions must obey.

---

## 6. Recommended Glaux Server Planning Terminology

### 6.1 Preferred Vocabulary Rules

| Prefer | Use It For | Do Not Treat as Synonymous With |
|---|---|---|
| **connected system** | Operational umbrella or problem domain | One API `System` resource, network-attached device only |
| **`System`** | CSAPI resource representing an implementing instance | Exclusive Sensor/Actuator/Sampler/Platform class |
| **`systemType`** | Required conceptual URI/CURIE for the System's primary activity; map it through the selected encoding (GeoJSON: `properties.featureType`) | Multiple values, a universal wire-member name, or guaranteed API capability |
| **`systemKind`** | Separate optional link from a System to a `Procedure` describing its kind/datasheet (GeoJSON: `properties['systemKind@link']`) | `systemType` or a free-form role label |
| **`Procedure`** | CSAPI methodology or system-kind/datasheet resource implemented by Systems | Exact synonym for only a 2017 SOSA method, SensorML Process representation, or a completed execution |
| **SensorML Process** | SensorML description/model construct | Automatically a CSAPI Procedure or runtime process |
| **`Deployment`** | Purpose-, place-, and time-specific placement/association of Systems | Software deployment, system location, or feature of interest |
| **feature resource** | Feature-oriented API representation | The real-world thing or its geometry |
| **ultimate feature of interest** | The application-domain subject when distinction is needed | `SamplingFeature` |
| **`SamplingFeature`** | System-specific sample, subset, or proxy | Every feature or the ultimate subject |
| **`Property` resource** | CSAPI semantic definition | JSON property, current property value, SWE component, or URI string |
| **API `DataStream`** | CSAPI collection of compatible Observations | Delivery transport or SWE `DataStream` |
| **SWE `DataStream`** | SWE structure/encoding/value container | CSAPI API collection |
| **`Observation`** | CSAPI/SOSA observing act or resource | Its `result` value |
| **observation result** | Value/data produced by an Observation | Whole Observation resource |
| **`ControlStream`** | CSAPI collection of compatible Commands | Network transport or the whole control workflow |
| **`Command`** | Concrete tasking request/resource | Tasking workflow, Actuation, or execution |
| **command-status report** | One `CommandStatus` resource/report | System status, health, or `Command.currentStatus` cache |
| **`CommandResult`** | Result produced by command execution | Necessarily an Observation |
| **`Feasibility`** | CSAPI request to assess a command without submitting it | Capability, readiness, or guaranteed availability |
| **`SystemEvent`** | CSAPI lifecycle/notable-change resource | Generic broker event, status sample, or command status |
| **phenomenon time** | When an observation result applies | Result, ingest, retrieval, or freshness time |
| **result time** | When a value was obtained/activity completed, per applicable API contract | Ingest or report time |
| **`Command.issueTime`** | Source-version-qualified command time pending resolution of the published “received” versus “issued/request-receipt” conflict | An unambiguous standard instant |
| **command execution time / status execution time** | Actual Command execution period versus status-dependent scheduled, estimated, or actual interval | Requested time as one generic field |
| **description-valid time / deployment period** | Qualified uses of resource-specific `validTime` | One universal operational-time field |
| **system state / health / availability / readiness** | Separate operational or future Glaux profile concepts | `CommandStatus`, capability, or feasibility |
| **freshness assessment / stale assessment** | Policy-derived assessment with time and threshold | A standard stored state supplied by CSAPI |
| **schema / data structure / encoding** | Separate validation and representation layers | One interchangeable “format” concept |
| **semantic definition identifier** | URI in SWE `definition` or equivalent binding | The complete CSAPI `Property` resource |
| **provenance / lineage / quality / suitability / trust** | Separate evidence and assessment concepts | Interchangeable metadata labels |
| **DDIL-informed behavior** | Glaux design goal for denied, disrupted, intermittent, and limited conditions | Claim of a standard-defined sync protocol |

### 6.2 Capitalization and Internal Naming

Capitalize exact standard resource types in planning and requirements. Use lowercase for an operational concept unless explicitly referring to the resource. Rust and database names will be finalized later, but ambiguous internal concepts should be qualified even when JSON serialization preserves the standard spelling. Candidate design-language aliases include `ApiDataStream` versus `SweDataStream`, `CommandStatusReport` versus `SystemState`, and `DescriptionValidTime` versus `PhenomenonTime`. These are project recommendations, not required public type names.

### 6.3 Rules for New Glaux Profile Terms

A later topic may introduce a Glaux profile term only when it records:

- the gap it fills and why an existing standard term is insufficient;
- a one-sentence definition with explicit scope and exclusions;
- its source authority class as **Glaux profile**, never NATO- or OGC-defined;
- the owning downstream topic and decision status;
- serialization or vocabulary identifiers, if any;
- validation rules and requirement-to-test traceability; and
- relationships to the entries in Appendix 12.1.

Candidate profile families—not definitions made here—are system operational state, health, readiness, freshness/staleness, last-known status, synchronization/reconciliation state, trust assessment, and security/releasability metadata.

---

## 7. Server Implications and Downstream Handoffs

### 7.1 Practical Server Implications

| Area | Terminology Consequence | Required Later Action |
|---|---|---|
| API paths and public types | Use exact CSAPI resource spellings; do not create `ConnectedSystem`, generic `Status`, or SWE-shaped `DataStream` endpoints | Requirement/path extraction in 006–008 |
| Rust domain model | Preserve System/Procedure, Observation/Result, Command/Actuation, and API/SWE stream boundaries; model system roles compositionally | Decide concrete types in 015 and 021–023 |
| Persistence | Store identifiers, semantic definitions, applicable times, result values, and status/event histories as distinct concepts | Data/resource modeling in 015–020 and 025–033 |
| Validation | Separate resource schema, SWE component structure, encoding, external semantic resolution, and business rules | 008, 022–024, 050–053 |
| Content negotiation | Representation names cannot be inferred from domain terms alone; SensorML and SWE encodings have bounded roles | 012, 021–023 |
| Tasking | Keep Command, status history, result, feasibility, and actual execution distinct | 007, 036–038 |
| Dynamic status | Never reuse command lifecycle codes for system health/readiness | 020 and 034 |
| Time and freshness | Retain each standard timestamp and compute freshness under explicit policy | 018, 034, 042 |
| Security and trust | Do not collapse authentication, authorization, handling, releasability, provenance, quality, and trust | 019 and 039–041 |
| Testing | Requirement and tests must use the same qualified term and cite the same source/version | 008, 050–053, 056 |

No implementation estimate is appropriate yet for the affected Rust components because detailed requirements, architecture, and target conformance classes have not been selected. Terminology adoption itself is low effort; repairing a collapsed model later would be high effort. Therefore this vocabulary should be incorporated before domain-model or schema code is written.

### 7.2 Downstream Topic Handoff Matrix

| Topic(s) | Required Terminology Input | Decision or Check to Carry Forward |
|---|---|---|
| `IDR-SRV-005` | standards package, operational terms, adjacent-standard versus adopted-standard boundary | Do not import a related NATO vocabulary as controlling without an explicit adoption/profile decision |
| `IDR-SRV-006` | System, Procedure, Deployment, feature resource, FOI, SamplingFeature, Property, id/uid/URL | Extract exact Part 1 definitions and resolve the Property-feature tension without changing names |
| `IDR-SRV-007` | DataStream, Observation/Result, ControlStream, Command, CommandStatus, CommandResult, Feasibility, SystemEvent, all time fields | Preserve resource versus status/result and conceptual versus wire-schema distinctions |
| `IDR-SRV-008` | seven mapping categories and discrepancy register | Give every normative/prose/schema/test conflict an explicit disposition and test consequence |
| `IDR-SRV-015` | System/Procedure and resource/value boundaries | Build the canonical model without role-class or activity/result collapse |
| `IDR-SRV-016` | resource id, uid, canonical URL, semantic identifier | Define identity scopes and lifecycle separately |
| `IDR-SRV-017` | hosts, implements, deployed-on, subsystem, ultimate FOI, sampling chain | Preserve typed relationships rather than generic links |
| `IDR-SRV-018` | phenomenon, result, issue, execution, report, event, valid, ingest/retrieval times | Define each clock and interval; make freshness a qualified derived assessment |
| `IDR-SRV-019` | provenance, lineage, quality, suitability, trust | Define separate evidence and assessment models |
| `IDR-SRV-020` | system state, health, availability, readiness, SystemEvent, status observations, CommandStatus | Create a bounded project vocabulary; do not reuse command codes or provisional event URIs blindly |
| `IDR-SRV-021` | System/Procedure versus SensorML Process; capability and history | Define representation context and inheritance without creating a second identity model |
| `IDR-SRV-022` | SWE DataComponent, structure, definition, unit, constraint, quality | Keep semantic data description separate from server resources and business validation |
| `IDR-SRV-023` | SWE/API DataStream distinction; schema versus encoding | Define payload encodings without conflating the two stream concepts |
| `IDR-SRV-024` | Property resource, SWE definition URI, SOSA edition, unit, vocabulary governance | Select authoritative registries and edition-aware semantic-binding rules |
| `IDR-SRV-034` | DataStream, Observation/Result, status observation, times, freshness | Define dynamic-data semantics without generic “status” or “stream” shortcuts |
| `IDR-SRV-035` | API collection versus delivery transport | Choose live-delivery protocols and guarantees independently of the `DataStream` resource name |
| `IDR-SRV-036` | tasking/control, ControlStream, Command, Actuation, lifecycle report/result | Specify command state and cancellation without treating tasking as one object |
| `IDR-SRV-037`–`038` | Feasibility versus capability/availability; execution/result distinctions | Preserve command-shaped feasibility and asynchronous result semantics |
| `IDR-SRV-039`–`041` | authorization, releasability, handling, accreditation, provenance, trust | Define security and policy concepts independently, with explicit source authority |
| `IDR-SRV-042`–`043` | DDIL expansion, connectivity state, freshness, last-known, sync/reconciliation | Do not imply that CSAPI streaming supplies offline reconciliation |
| `IDR-SRV-050`, `052`–`053` | qualified source terms, discrepancy records, requirement anchors | Make terminology part of requirement-to-test identifiers and diagnostics |
| `IDR-SRV-051` | all preferred terms and source anchors | Maintain one term/source/concept relation through requirements and tests |
| `IDR-SRV-056` | conceptual/wire terminology mismatches | Test interoperability behavior selected for every tracked ambiguity |
| Final IDR synthesis | accepted vocabulary and unresolved profile candidates | Publish the final project glossary with decisions from downstream owners |

The plan's formal Blocks list names 17 topics plus final synthesis. Additional rows above are advisory handoffs required by the plan's dynamic-data, tasking, security, DDIL, and verification research question; listing them does not start those topics or change their order.

---

## 8. Risks, Constraints, and Open Questions

### 8.1 Risks and Controls

| Risk | Consequence | Control |
|---|---|---|
| Same label mapped to one universal type | Non-conformant model and hidden data loss | Apply qualified terms and Appendix 12.1 relations during design reviews |
| Too many project aliases | Public API drifts from CSAPI and becomes harder to understand | Preserve exact wire/resource names; aliases are internal documentation aids only |
| Operational gaps presented as standards definitions | False conformance and NATO claims | Label every extension as a Glaux profile term with owner and tests |
| 2017/2025 SOSA vocabulary silently mixed | Invalid ontology and interoperability claims | Pin semantic edition; record newer draft only as informative |
| Prose/schema discrepancies resolved ad hoc | Different implementation and test interpretations | Maintain a discrepancy register and approve one traceable resolution per issue |
| Status/readiness vocabulary invented prematurely | Unstable APIs and state machines | Defer controlled codes to 020 after requirements and use cases are known |
| Freshness treated as one timestamp | Incorrect decisions in DDIL and delayed-data conditions | Model source time, observation time, ingest/retrieval evidence, validity, and policy separately |
| “DDIL” used as a catch-all | Unverifiable resilience claims | State the exact connectivity condition and server behavior being discussed |

### 8.2 Open Questions and Owners

| Open Question | Present Disposition | Owner |
|---|---|---|
| Which exact STANAG/AEP copy controls this IDR? | Resolved: the project lead registered `AC/224(JCGISR)D(2026)0005`, 27 Apr 2026, with matching hash. It remains a pre-promulgation ratification draft. | Overall-plan governance |
| Should Glaux prefer CSAPI terms even where AEP differs? | Recommended for technical/API contexts, pending project-lead acceptance: yes. AEP terms remain for operational intent and are not erased. | This report; acceptance by project lead |
| Which terms belong in the final project glossary? | Candidate set established in §6 and Appendix 12.1; final membership awaits downstream decisions on status, security/trust, DDIL, and discrepancies. | Final IDR synthesis |
| What SOSA/SSN conformance can Glaux claim? | Unresolved until 006–008 and 024 reconcile approved CSAPI usage with the stable 2017 Recommendation and current drafts. | 006–008, 024 |
| What are the final system-state, health, availability, readiness, freshness, and event vocabularies? | Deliberately unresolved; adopted package gives concepts but not a complete controlled model. | 018, 020, 034, 042 |
| Which published prose/schema interpretation controls each defect in §4.6.2? | Deliberately unresolved here; each needs requirement, interoperability, and test analysis. | 006–008, 050–053, 056 |

None of these unresolved items blocks terminology use or the next planned research topic after this report is accepted. They are bounded downstream decisions, not missing evidence that would make this report indefensible.

---

## 9. Validation Against the Research Plan

### 9.1 Methodology Phase Validation

| Phase | Work Performed | Required Output | Status / Evidence |
|---|---|---|---|
| 1. Source collection and framework | Verified dependencies, controlled-package hash, official sources, accepted prior reports, authority layers, and seven mapping types | Source inventory and framework | Complete; §§3 and 4.1 |
| 2. STANAG/AEP extraction | Reviewed formal lexicons and operational vocabulary with PDF/section anchors | NATO terminology inventory | Complete; §4.2 and Appendix 12.1 |
| 3. OGC/SensorML/SWE/SOSA extraction | Reviewed definitions, conceptual models, published artifacts, semantic sources, and source-specific boundaries | Standards terminology inventory | Complete; §4.3 and Appendix 12.1 |
| 4. Mapping/conflict analysis | Classified 53 crosswalk rows across all seven categories and recorded semantic and publication conflicts | Crosswalk and conflict register | Complete; §§4.4–4.6 and Appendix 12.1 |
| 5. Server implications/handoffs | Derived preferred terms, practical implications, and topic-specific handoffs | Usage guide and handoff matrix | Complete; §§6–7 |
| 6. Synthesis | Reconciled findings with accepted reports and produced decision brief, recommendations, risks, and validation | Polished report | Complete; this deliverable |

### 9.2 Success Criteria Validation

| Topic Plan Success Criterion | Status | Evidence |
|---|---|---|
| Project-controlling STANAG/AEP material reviewed | Met | §§3.2, 4.2; Appendix 12.1 |
| CSAPI Parts 1/2, SensorML, SWE Common, SOSA/SSN reviewed | Met | §§3.2 and 4.3 |
| Sources include title, version/date, location, status, authority | Met | §§3.1–3.3 and 11 |
| Terms extracted with anchors | Met | §§4.2–4.3; Appendix 12.1 |
| All seven mapping categories used | Met | §4.1; Appendix 12.1 |
| Preferred Glaux terms recommended | Met | §6 |
| Conflicts and implementation risks identified | Met | §§4.6 and 8.1 |
| Downstream handoffs identified | Met | §7.2 |
| Unresolved questions explicitly listed | Met | §8.2 |
| Recommendations bounded and decision-usable | Met | §§1, 5, and 6 |
| References explicit and reproducible | Met | §§3 and 11; mutable sources pinned |

### 9.3 Deliverable Requirement Validation

| Required Content | Status | Location |
|---|---|---|
| 1. Executive summary | Met | §1 |
| 2. Scope and plan alignment | Met | §2 |
| 3. Evidence base and authority classification | Met | §3 |
| 4. Crosswalk methodology | Met | §4.1 |
| 5. STANAG/AEP inventory | Met | §4.2 |
| 6. OGC/SensorML/SWE/SOSA-SSN inventory | Met | §4.3 |
| 7. Crosswalk matrix | Met | Appendix 12.1 |
| 8. Conflict, ambiguity, and risk analysis | Met | §§4.6 and 8 |
| 9. Recommended planning terminology | Met | §6 |
| 10. Downstream handoff matrix | Met | §7.2 |
| 11. Recommendations | Met | §§5–6 |
| 12. Risks, constraints, and open questions | Met | §8 |
| 13. Success-criteria validation | Met | §9.2 |
| 14. References | Met | §11 |

### 9.4 Crosswalk Field Validation

| Required Field | Present in Appendix 12.1 |
|---|---|
| Source term | Yes |
| Source family | Yes |
| Definition or usage summary | Yes |
| Source anchor | Yes |
| Related terms in other families | Yes |
| Mapping type | Yes |
| Preferred Glaux term | Yes |
| Server implication | Yes |
| Downstream handoff | Yes |
| Notes / unresolved issues | Yes |

### 9.5 Dependency and Open-Question Validation

All six start dependencies were satisfied: the overall plan and goal were current; reports 001–003 were accepted; the controlled package was available and hash-verified; official sources were reachable; and the report template was available. The three plan-level open questions are dispositioned in §8.2. No governance exception was needed.

### 9.6 Independent Review Results

Three bounded evidence reviews independently covered the controlled NATO terminology, the four approved OGC standards and published artifacts, and the SOSA/SSN plus plan-coverage baseline. A separate handoff/overclaim pass checked downstream assignments. The checks corrected or prevented four material errors: treating the ratification draft as promulgated NATO policy; conflating AEP observation data with the CSAPI/SOSA activity; silently importing the 2025 SOSA draft into the 2017 baseline; and treating all similarly named status, stream, property, and process concepts as synonyms.

---

## 10. Next Steps and Handoff

### 10.1 Current Status

Research, synthesis, validation, and repository review for `IDR-SRV-004` are complete. The report is **In Review**. Under the overall-plan governance rules, it is not an accepted downstream baseline until the Glaux Project Lead accepts it and the acceptance fields are completed.

### 10.2 Required Two-Action Transition

The next workflow transition has exactly two actions, and neither is implied by publication of this report:

1. The Glaux Project Lead reviews and accepts `IDR-SRV-004`.
2. The Glaux Project Lead authorizes execution of exactly one next eligible topic, `IDR-SRV-005`.

Both actions may be given in one instruction. Until then, `IDR-SRV-005` and all later research topics remain unstarted.

The copyable combined instruction is:

> Approve IDR-SRV-004 and execute exactly one Glaux Server research plan: IDR-SRV-005, using the standing single-topic execution instructions.

---

## 11. References

### 11.1 Controlled NATO Source

- NATO, `AC/224(JCGISR)D(2026)0005`, *NATO Standard - STANAG 4789*, 27 April 2026. Project-controlled ratification draft; SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`.
  - Enclosure 1: *STANAG 4789, Sensor Integration for NATO Joint Intelligence, Surveillance, and Reconnaissance Operations*, Edition 1, PDF pp. 3–7.
  - Enclosure 2: *AEP-4789 Volume I, Sensor Integration Interoperability Framework*, Edition A, Version 1, PDF pp. 8–37.
  - Enclosure 3: *AEP-4789 Volume II, Sensor Integration Standard for NATO JISR Operations - Core APIs and Encodings*, Edition A, Version 1, PDF pp. 38–59.

### 11.2 Official Standards and Published Artifacts

- Open Geospatial Consortium, [OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html), approved 2025-06-02, published 2025-07-16. Accessed 2026-07-31.
- Open Geospatial Consortium, [OGC 23-002, *OGC API - Connected Systems - Part 2: Dynamic Data*, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html), approved 2025-06-02, published 2025-07-16. Accessed 2026-07-31.
- Open Geospatial Consortium, [OGC 23-000, *OGC SensorML Encoding Standard*, Version 3.0](https://docs.ogc.org/is/23-000/23-000.html), approved 2025-06-02, published 2025-07-16. Accessed 2026-07-31.
- Open Geospatial Consortium, [OGC 24-014, *OGC SWE Common Data Model Encoding Standard*, Version 3.0.0](https://docs.ogc.org/is/24-014/24-014.html), approved 2025-06-02, published 2025-07-16. Accessed 2026-07-31.
- W3C and OGC, [*Semantic Sensor Network Ontology*](https://www.w3.org/TR/vocab-ssn/), W3C Recommendation, 19 October 2017. Accessed 2026-07-31.
- [OGC API - Connected Systems Part 1, official version 1.0 schemas and examples](https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/). Accessed 2026-07-31.
- [OGC API - Connected Systems Part 2, official version 1.0 schemas, OpenAPI, AsyncAPI, and examples](https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/). Accessed 2026-07-31.
- [Official SensorML 3.0 JSON schemas](https://schemas.opengis.net/sensorML/3.0/json/). Accessed 2026-07-31.
- [Official SWE Common 3.0 JSON schemas](https://schemas.opengis.net/sweCommon/3.0/json/). Accessed 2026-07-31.

### 11.3 Informative and Standards-Watch Sources

- W3C and OGC, [*Semantic Sensor Network Ontology - 2023 Edition*](https://www.w3.org/TR/2025/WD-vocab-ssn-2023-20250916/), First Public Working Draft, 16 September 2025. Informative only; accessed 2026-07-31.
- W3C and OGC, [*Spatial Data on the Web Best Practices*](https://www.w3.org/TR/2023/DNOTE-sdw-bp-20230919/), W3C Group Draft Note / OGC Best Practice, 28 September 2023. Informative only; accessed 2026-07-31.
- [OGC API - Connected Systems development repository at commit `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f). Informative development evidence; accessed 2026-07-31.

### 11.4 Project and Governance Sources

- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md), Version 1.5, 2026-07-30.
- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md), Version 1.3, 2026-07-31.
- [IDR-SRV-004 Research Plan](../IDR%20Plans/idr-srv-004-terminology-and-concept-crosswalk.md).
- [Accepted IDR-SRV-001 Report](idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md).
- [Accepted IDR-SRV-002 Report](idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md).
- [Accepted IDR-SRV-003 Report](idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md).
- [Research Planning Approach](../../../../../Governance/research-planning-approach.md), Version 1.3, 2026-07-31.
- [Research Plan Creation Prompt](../../../../../Governance/research-plan-creation-prompt.md).
- [Research Plan Template](../../../../../Governance/research-plan-template.md).
- [Research Report Template](../../../../../Governance/research-report-template.md).
- [Overall Research Report Template](../../../../../Governance/overall-research-report-template.md).
- [OS4CSAPI research-plan exemplar corpus at commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans). Style precedent only.

---

## 12. Appendices

### 12.1 Terminology and Concept Crosswalk

The matrix is intentionally implementation-oriented rather than an attempt at a complete ontology. “AEP” anchors use PDF page numbers in the fixed controlled file; OGC and W3C anchors use clauses in the cited official editions.

| ID / Source Term | Source Family | Source Definition or Usage Summary | Source Anchor | Related Term(s) | Mapping Type | Preferred Glaux Term | Server Design Implication | Handoff | Notes / Unresolved |
|---|---|---|---|---|---|---|---|---|---|
| XW-01 `standards package` | AEP-II | Fixed coherent package of CSAPI Parts 1/2, SensorML, and SWE Common | AEP-II p. 58 / Lex-3; §§1.1–1.2 | adopted four standards | Exact equivalent in this package context | standards package | Pin all four editions; do not include future parts implicitly | 005–008, 021–024 | Exact membership for this IDR; not all related standards |
| XW-02 `connected system` | AEP-I/II | Operational subject that transmits data or whose data is exposed through networks | AEP-I p. 35; AEP-II p. 57 | CSAPI `System`, Connected Systems domain | Near equivalent / related but distinct | connected system; `System` when resource-specific | Do not create a public `ConnectedSystem` resource merely from the umbrella term | 006, 015 | Operational subject versus API representation; includes mediated data exposure |
| XW-03 `system` / `System` | AEP-II / CSAPI | Broad connected-system instance/resource; CSAPI instance implements Procedures | AEP-II p. 58; P1 §§4.17, 9 | 2017 `ssn:System`, newer `sosa:System`, SensorML Process | Near equivalent | `System` | Canonical API/domain resource; preserve source-specific axioms and ontology edition | 006, 015, 021, 024 | AEP list does not establish disjoint types; CSAPI namespace usage is version-sensitive |
| XW-04 sensor, actuator, sampler, platform | AEP / CSAPI / SOSA | Overlapping semantic roles; CSAPI requires one conceptual `systemType` for primary activity; Platform hosts systems in SOSA | AEP-I §§3.1–3.2; P1 §§4.2, 4.9, 4.13, 4.15, 9.1–9.2; SOSA §4.9 | `System`, `systemKind` | Related but distinct | `systemType` conceptual attribute; semantic role in analysis | Map one primary value through the chosen encoding; do not infer capabilities or build disjoint root entities from it | 006, 015, 017 | GeoJSON uses `properties.featureType`; `systemKind@link` is a separate Procedure link; not every Platform is a System |
| XW-05 subsystem | CSAPI | System nested as an ordinarily integral part of another System | P1 §§10.1–10.3 | System aggregation, Deployment | Narrower concept than general system association | subsystem relationship | Model relationship rather than separate root type | 015, 017 | Temporary aggregation belongs in Deployment |
| XW-06 `Procedure` | CSAPI / SOSA | CSAPI resource can represent reusable methodology or a system kind/datasheet; 2017 SOSA centers on reusable workflow/method | P1 §§4.10, 9.1 Note 1, 13; SOSA §4.8 | SensorML Process, execution | Near equivalent for method use; CSAPI broader for datasheet/system-kind use | `Procedure` | Preserve method/type, instance, description representation, and execution boundaries | 006, 015, 021 | Multiple Systems may implement one Procedure; not one shared definition in every use |
| XW-07 `Process` | SensorML | Physical or computational operation with inputs, parameters/method, and outputs; hardware is modeled as process | SensorML §§4.25, 7.1–7.2 | CSAPI `System`, `Procedure` | Related but distinct | SensorML Process | Representation context determines whether it describes a System or Procedure | 021 | Not a Rust OS process and not automatically executable |
| XW-08 `deployment` / `Deployment` | AEP / CSAPI / SOSA / SensorML | Operational deployment information versus purpose-, time-, and place-specific deployment concept/resource | AEP-II p. 57; P1 §§4.4, 11; SOSA §4.9; SensorML §8.9 | platform hosting, system association | Near equivalent | `Deployment` | Separate resource identity from detailed SensorML representation | 006, 015, 017, 021 | Never use for deploying server software |
| XW-09 capability | AEP / SensorML / SSN | What a system can do or performance/operating characteristics | AEP-I §§3.1–3.2; SensorML §8.2.4; SSN §5.1 | availability, readiness, feasibility | Related but distinct | capability metadata | Discovery metadata must not become a promise of current operability | 020, 021 | Current-state vocabulary remains unresolved |
| XW-10 feature resource | AEP / CSAPI | Connected-system resource exposed through feature-oriented API behavior | AEP-II p. 57; P1 §§1, 7 | Feature, representation | Near equivalent | feature resource | Keep real thing, API representation, and geometry distinct | 006, 015 | P1 treats Property as the exception to feature resources |
| XW-11 feature of interest | AEP / CSAPI / SOSA | AEP/CSAPI use the ultimate subject; 2017 SOSA defines a contextual role that a Sample can also play | AEP-II p. 57; P1 §4.7; SOSA §4.6 | SamplingFeature, Sample | Near equivalent; exact only between AEP and CSAPI for the ultimate-feature role | ultimate feature of interest when contrast matters | Maintain explicit link through sampling proxy and preserve SOSA role semantics | 006, 017, 024 | Not automatically a separate concrete class |
| XW-12 sampling feature | AEP / CSAPI | Sample/subset/proxy used to relate activity to ultimate subject | AEP-II pp. 57–58; P1 §§4.12, 4.14, 14 | SOSA Sample, FOI | Near equivalent | `SamplingFeature` | Preserve sampling chains and proxy identity | 006, 015, 017 | CSAPI supplies API resource semantics |
| XW-13 `Sample` | SOSA | Representative proxy for a feature and result of Sampling | SOSA §4.5 | CSAPI SamplingFeature | Near equivalent | `SamplingFeature` in API; SOSA Sample in ontology context | Do not call every sample value or specimen a CSAPI resource | 015, 024 | SOSA adds ontology axioms; CSAPI extends analogous sampling-feature use into control |
| XW-14 property / `Property` | AEP / CSAPI / SOSA / software | Semantic definition resource; intrinsic quality; controllable/observable property; or ordinary member/attribute | AEP-I §§1.4, 3.5; P1 §§4.11, 15; SOSA §4.6 | SWE `definition`, JSON property | Conflicting usage | `Property` resource; otherwise qualify | Separate semantic entity from member, role, and current value | 006, 015, 022, 024 | 2017 SOSA uses `ssn:Property`, not the later generic `sosa:Property` |
| XW-15 observable / controllable / asserted property | CSAPI / SOSA | Roles a semantic Property can play relative to observation, control, or description | P1 §15; SOSA §§4.4, 4.6 | `Property` resource | Narrower role concepts under CSAPI Property | observed property / controlled property / asserted property | Do not create disjoint semantic-resource classes solely from role | 015, 024 | One Property may have multiple roles |
| XW-16 `definition` | SWE / SensorML | URI identifying the semantic meaning of a data component or described item | SWE §§7.3, 8.2; SensorML §8 | CSAPI Property UID | Related but distinct | semantic definition identifier | Resolve/link to governed semantics; field is not the Property resource | 021–024 | Registry/cache policy deferred |
| XW-17 `datastream` / `DataStream` | AEP / CSAPI | Operational outward flow versus API collection of compatible Observations from one System | AEP-II p. 57; P2 §§4.7, 9.1–9.2 | observations, status data, transport, ObservationCollection | Near equivalent with different scope and constraints | API `DataStream`; operational data flow when generic | Model API collection independently from delivery mechanism | 007, 015, 024, 034–035 | CSAPI collection pattern is absent from 2017 SOSA/SSN |
| XW-18 `DataStream` | SWE Common | Open-ended values wrapper combining element type, encoding, and values | SWE §8.5.3 | CSAPI `DataStream` | Conflicting usage | SWE `DataStream` / `SweDataStream` internally | Separate Rust/schema types from API collection | 022–023, 034 | Same label, materially different class |
| XW-19 `observation` / `Observation` | AEP / CSAPI / SOSA | AEP: resulting data; CSAPI/SOSA: observing act/resource with separate result; CSAPI can cover multiple properties versus 2017 SOSA cardinality one | AEP-I p. 35; AEP-II p. 57; P1 §4.8 and Annex C; P2 §9.7; SOSA §4.3 | observation result | Conflicting AEP usage; CSAPI/SOSA conceptual cores are near, not exact | `Observation` for technical object; observation data operationally | Store and validate Observation metadata separately from result payload and retain multi-property semantics | 007, 015, 024, 034 | Highest-risk false equivalence; ontology mapping is version-sensitive |
| XW-20 `Result` / result | SOSA / CSAPI | Output/value produced by Observation; contextual results also exist for Commands | SOSA §4.7; P2 §§9.7, 10.13 | Observation, CommandResult | Related but distinct | observation result / `CommandResult` | Use context-qualified result types | 007, 015, 034, 036 | A CommandResult may link to an Observation but is not one |
| XW-21 phenomenon time | CSAPI / SOSA | Time when an observation result applies to the feature/property | P2 §9.7; SOSA §4.3 | result time, valid time | Exact equivalent for core temporal role | phenomenon time | Preserve as instant or interval per schema; do not substitute ingest time | 007, 018, 034 | Future phenomenon time may be meaningful; exact constraints belong to 007 |
| XW-22 result time | CSAPI / SOSA | When result was obtained or activity completed | P2 §9.7; SOSA §4.7 | phenomenon time, ingest time | Near equivalent | result time | Retain source definition used by the API contract | 007, 018, 034 | Wording differs slightly across sources |
| XW-23 `validTime` | CSAPI / SensorML / draft SOSA-OMS extension | Validity/applicability with resource-specific meaning; 2025 draft uses `sosa-oms:validTime`, not CSAPI's `sosa:validTime` spelling | P1 System/Deployment tables; P2 DataStream/ControlStream tables; SensorML §8.2.2.6; 2025 FPWD SOSA-OMS | deployment period, freshness | Related but distinct and version-sensitive; conflicting if unqualified | description-valid time or deployment period | Document semantics per resource and namespace; no universal column meaning | 006–008, 018, 024 | Draft extension is informative only |
| XW-24 ingest, receipt, publication, synchronization time | Glaux/AEP operational need | Server-observed processing or exchange times, not defined CSAPI observation times | AEP-I §§2.4.2, 3.3–3.6 | phenomenon/result/report times | Unresolved profile concepts | qualify exact operational time | Add only with source/clock semantics and tests | 018, 031–035, 042–043 | Not standard fields by implication |
| XW-25 dynamic property | CSAPI | Property of a feature whose changing value can be represented through observations | P1 §14.7 | Property, Observation, latest value | Related but distinct | dynamic property value | Avoid duplicating latest value as authoritative static metadata | 006, 015, 018, 034 | Snapshot/latest behavior needs later requirement analysis |
| XW-26 status | AEP / CSAPI / HTTP / operations | Generic condition, observed status data, command lifecycle, protocol code, or service health | AEP-I §3.6; P2 §§9, 10.11 | CommandStatus, SystemEvent, health | Conflicting usage | always qualify the status kind | No generic `Status` root type or enum | 007, 020, 034, 036 | Controlled system-status model absent |
| XW-27 `SystemEvent` | AEP / CSAPI / SensorML | Information/resource for notable lifecycle, state, configuration, maintenance, or deployment change | AEP-II p. 58; P2 §12; SensorML §8.2.8 | status observation, audit event | Near equivalent | `SystemEvent` | Separate durable domain event from generic broker/audit events | 007, 020 | Published event identifiers remain provisional |
| XW-28 current, fresh, stale, last-known | AEP operational | Temporal assessments of information usability/state | AEP-I §§2.4.2, 3.2–3.6 | validTime, resultTime, ingest time | Unresolved | freshness/stale/last-known assessment | Derive under explicit policy; record subject, as-of time, and evidence | 018, 020, 034, 042 | No thresholds or clock rules in package |
| XW-29 tasking / control | AEP operational | End-to-end workflow for directing systems and determining supportability | AEP-I §3.5 | Command, ControlStream, Feasibility | AEP tasking/control broader than CSAPI Command | tasking workflow / control workflow | Do not implement whole workflow as one Command record | 005, 036–038 | Includes policy, supportability, and operational decisions |
| XW-30 control stream / `ControlStream` | AEP / CSAPI | Flow carrying commands versus collection of compatible Commands for one System | AEP-II p. 57; P2 §§4.6, 10.1–10.2 | transport channel, ActuationCollection | Near equivalent | `ControlStream` | Resource is not transport; define delivery separately | 007, 015, 024, 035–036 | CSAPI collection pattern is absent from 2017 SOSA/SSN |
| XW-31 command / `Command` | AEP / CSAPI | Request to direct/modify a connected System with parameter/property values | AEP-II p. 57; P2 §§4.4, 10.7 and Annex C relationship | tasking, Actuation | Exact AEP/CSAPI request concept; related but distinct from SOSA Actuation | `Command` | CSAPI controls resource fields, state, wire form, and can control multiple properties | 007, 024, 036 | Do not claim Command equals 2017 Actuation |
| XW-32 `Actuation` | SOSA | Activity that changes a feature/property state, with one actuatable-property association in the 2017 pattern | SOSA §4.4 and §7.2; P1 Annex C | Command, execution | Related but distinct | actuation activity | Preserve request versus performed activity and single- versus multi-property semantics | 015, 024, 036 | 2017 SOSA excludes future actuation plans; CSAPI Command generalizes the concept |
| XW-33 command status / `CommandStatus` | AEP / CSAPI | State/disposition information versus timestamped lifecycle/progress report | AEP-II p. 57; P2 §10.11 | Command.currentStatus, system state | Near equivalent | command-status report / `CommandStatus` | Keep history separate from cached summary and system state | 007, 036 | AEP examples are not a mandatory enum/transition graph |
| XW-34 `CommandResult` | CSAPI | Output of execution, inline or linked to observations/datastream/external resources | P2 §10.13 | Observation result, CommandStatus results link | Related but distinct | `CommandResult` | Separate result entity/association and serialization names | 007, 036 | Published conceptual/schema member names differ |
| XW-35 `Feasibility` | CSAPI / AEP supportability language | Command-shaped analysis request made without issuing operational command | P2 §11; AEP-I §3.5 | capability, availability, authorization | CSAPI Feasibility narrower than operational supportability | `Feasibility` | Reuse command parameter semantics; do not promise execution | 007, 020, 037 | Not safety approval, reservation, or authorization |
| XW-36 metadata | AEP / OGC | Broad descriptive, structural, temporal, semantic, policy, and provenance information | AEP-I §§2.2, 3.2–3.4; P1/P2 models | description, schema, provenance | Broader concept | qualify metadata kind | Avoid a single untyped metadata bag as canonical model | 015, 019, 021–024 | “Metadata” alone is insufficient in decisions |
| XW-37 description | AEP / SensorML / CSAPI | Information identifying and characterizing a system, procedure, deployment, or stream | AEP-I §3.2; SensorML §§7–8; P1/P2 resource models | metadata, representation | Related but distinct | resource description / SensorML description | Description validity and resource identity must remain explicit | 006–007, 021 | Rich SensorML description is not a second API identity |
| XW-38 `DataComponent` | SWE Common | Atomic or aggregate descriptor for semantics, structure, units, constraints, and quality | SWE §§4.1, 7–8 | schema field, Property resource | Related but distinct | SWE data component | Keep descriptor/schema layer apart from semantic resource and instance value | 022–023 | Can be descriptor or container depending class/context |
| XW-39 schema / encoding / media type | OGC / SWE | Structure contract; mapping to bytes/text/JSON; representation label | P2 §§9.6, 10.6, 16; SWE §§7.4–7.6, 8.7 | “format” | Conflicting usage when collapsed as format | schema, encoding, media type | Validate/negotiation layers independently | 007–008, 022–023 | An observation schema is not necessarily JSON Schema |
| XW-40 unit / `uom` | SWE / AEP | Unit qualifying a quantity; SI baseline with bounded non-SI allowance | SWE §§7–8; AEP-II Conventions p. 47 | observed property, quantity value | Related but distinct | unit of measure / `uom` | Store semantic unit independently from property identity and numeric value | 022, 024 | UCUM recommended where possible; exact policy deferred |
| XW-41 constraint | SWE / validation | Allowed values/ranges/categories for the owning component | SWE §§7.2.6, 8 | JSON Schema/business rule | Narrower concept than complete validation | SWE value constraint | Do not assume it expresses cross-field or authorization rules | 022–023, 050 | Validation layer must be named |
| XW-42 quality / suitability | AEP / SWE / SOSA-SSN | AEP/SWE quality evidence, SSN adjudged observation-quality and capability/performance associations, versus fitness for a use | AEP-I §§1.4, 3.2–3.3; SWE §§7–8; SSN §§5.1.2.5, 5.1.2.21, 5.1.2.32 | provenance, trust | Related but distinct | result/data quality; suitability assessment | Store evidence and consumer assessment separately | 019, 022, 024 | SSN associations are not a complete result-quality or suitability model |
| XW-43 provenance / lineage | AEP / SensorML | AEP source/method/history concepts; SensorML history and method/configuration metadata can contribute evidence | AEP-I §§2.2, 3.2–3.3; SensorML §§8.2.8–8.2.9 | quality, trust | Related but distinct | provenance and lineage | Preserve source, method, history, and derivation rather than one trust flag | 019, 021 | SensorML does not define a complete general provenance model; exact data model deferred |
| XW-44 trust | AEP operational | Judgment or policy concern based on identity, provenance, quality, and context | AEP-I §§3.2–3.3; AEP-II §2.2 | provenance, authority, quality | Unresolved | trust assessment | Do not infer trust from schema validity or provenance presence | 019, 039–041 | No adopted trust ontology/profile |
| XW-45 health / readiness / operational state | AEP operational | Current condition and suitability, including active/inactive/degraded | AEP-I §3.6 | status observations, availability | Unresolved | system state, health, readiness—separate terms | Later controlled model; never reuse CommandStatus codes | 020, 034, 042 | “Degraded” must name system or connectivity context |
| XW-46 availability | AEP operational | Present access/operability/capacity context | AEP-I §3.6 | capability, readiness, Feasibility | Related but distinct; conflicting if used interchangeably | system/service availability, qualified | Model current evidence separately from capability and prospective command analysis | 020, 037, 042 | No universal calculation in package |
| XW-47 DDIL | AEP | Denied, disrupted, intermittent, and limited communications conditions | AEP-II p. 56 / Lex-1; AEP-I §§2.2, 2.4 | disconnected, degraded, constrained | Exact equivalent for the acronym expansion; related but distinct operating conditions | DDIL; DDIL-informed | Name exact condition/behavior in requirements and tests | 042–043 | Disconnected/degraded are discussed but not acronym replacements |
| XW-48 synchronization / reconciliation | AEP operational | Later exchange and consistency work after interruption or staging | AEP-I §§2.4.2, 3.3–3.6 | replay, replication, conflict resolution | Related but distinct | synchronization; reconciliation | Define idempotency, authority, ordering, conflict, and evidence separately | 042–043 | Adopted package supplies no complete protocol |
| XW-49 authentication / authorization / releasability / handling / accreditation | AEP operational/security | Distinct identity, access, release, marking, and assurance concerns | AEP-I §§2.4.1, 3.2, 3.5, 4.2; AEP-II §2.2 | trust, policy | Related but distinct; profile unresolved | retain qualified security terms | Avoid generic `security`/`trusted` booleans | 039–041 | Full profile is outside the four-standard package |
| XW-50 id / UID / URL / semantic URI / database key | CSAPI / SensorML / implementation | Different scopes for resource addressing, underlying-thing identity, semantic identity, and storage | P1 §§8.4–8.5; SensorML §8; SWE `definition` | persistent identifier, designation | Related but distinct | qualify every identifier kind | Separate types/constraints; never use as interchangeable strings by assumption | 016, 021, 024 | Exact minting and alias policy deferred |
| XW-51 Feature | CSAPI / SensorML | Abstraction of real-world phenomena | P1 §4.5; SensorML §4.9 | feature resource, FOI, geometry | Exact equivalent across these two definitions; related concepts remain distinct | Feature when the abstraction is intended | Keep abstraction, API representation, real-world subject, and geometry separate | 006, 015, 017 | Does not make every phenomenon or geometry a Feature resource |
| XW-52 Phenomenon | SensorML | Physical state that can be observed and whose properties can be measured | SensorML §4.22 | Feature, ObservableProperty, Result | Related but distinct | phenomenon, source-qualified | Do not create an API resource or synonym without a CSAPI requirement | 015, 021, 024 | SensorML definition is narrower than every ordinary use of “phenomenon” |
| XW-53 command cancellation | CSAPI / AEP | Lifecycle action reported with `CANCELED`; AEP lists cancellation as an illustrative disposition | P2 §14.5 and §10.11 Table 14; AEP-II p. 57 | HTTP DELETE, CommandStatus | Related but distinct from deletion; narrower than generic cancellation | cancel command; `CANCELED` code | Retain Command resource; post status report; do not implement as DELETE | 007, 036 | Exact one-L code comes from CSAPI; transition rules need 007/036 |

### 12.2 Objective-Prompt Alignment

| Objective Prompt | Formal Questions Carrying It | Report Evidence |
|---|---|---|
| Which STANAG/AEP terms are essential? | CQ-1; STE-1–2 | §§4.2 and 6; Appendix 12.1 |
| Which equivalents and differences appear across sources? | CQ-2–3; STE-3–6; CF-1–6; CE-1–5 | §§4.3–4.6; Appendix 12.1 |
| Which terms must be normalized? | CQ-5; CE-6; SDI-1–6 | §§6–7 |
| Which ambiguities create risk? | CQ-4; CE-5; SDI-1–6 | §§4.6 and 8 |

### 12.3 Plan Status and Completion Checklist

- [x] Topic ID matches the overall research plan index
- [x] Topic research plan and controlling overall plan are linked
- [x] Exactly one topic was executed
- [x] All 5 core and 29 detailed questions are covered
- [x] All four objective-level prompts are explicitly mapped
- [x] All six methodology phases and expected outputs are validated
- [x] All eleven success criteria are validated
- [x] All fourteen deliverable requirements are present and mapped
- [x] All ten mandatory crosswalk fields are present
- [x] All seven mapping categories are defined and used
- [x] The complete controlled source package was available and its hash verified
- [x] Official CSAPI Parts 1/2, SensorML, SWE Common, and SOSA/SSN sources were reviewed
- [x] Mutable technical and exemplar sources are commit-pinned
- [x] Standards obligations, sourced findings, analysis, and project recommendations are distinguished
- [x] Conflicts, evidence limitations, and unresolved decisions are explicit
- [x] Findings are reconciled with accepted IDR-SRV-001 through IDR-SRV-003
- [x] Independent evidence, coverage, handoff, and overclaim checks were completed
- [x] No other IDR research topic was begun
- [ ] Plan-owner acceptance and acceptance date recorded

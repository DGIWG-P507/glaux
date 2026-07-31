# Section 003: AEP-4789 Volume II Standards Package Implementation Baseline - Research Report

**Topic ID:** IDR-SRV-003<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-003 Research Plan](../IDR%20Plans/idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** 33 of 33 (5 core and 28 detailed questions)<br>
**Methodology Used:** Integrity verification and complete review of AEP-4789 Volume II; adoption-language and source-strength extraction; review of the four official adopted OGC standards and their published schemas; coherent-package, conformance, validation, boundary, and downstream-handoff analysis; reconciliation with accepted IDR-SRV-001 and IDR-SRV-002; and independent coverage and overclaim review<br>
**Research Time:** Approximately 1.0 hour of AI-assisted elapsed execution time on July 31, 2026<br>
**Primary Source(s):**
- Project-controlled `AC/224(JCGISR)D(2026)0005`, dated 27 April 2026, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`, specifically AEP-4789 Volume II, Edition A, Version 1
- [OGC API - Connected Systems - Part 1: Feature Resources](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC SensorML Encoding Standard 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model Encoding Standard 3.0](https://docs.ogc.org/is/24-014/24-014.html)
**Supporting Resources:**
- [Accepted IDR-SRV-001 Report](idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md)
- [Accepted IDR-SRV-002 Report](idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md)
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research Planning Approach](../../../../../Governance/research-planning-approach.md)
**Document Purpose:** Establish how the four standards adopted by AEP-4789 Volume II fit together as the technical baseline for a best-of-breed, open-source Rust OGC API - Connected Systems server, while preserving clear conformance, validation, implementation, and ecosystem boundaries<br>
**Author(s):** OpenAI Codex<br>
**Accepted By:** TBD - Glaux Project Lead review required<br>
**Acceptance Date:** TBD<br>
**Date:** July 31, 2026<br>
**Last Updated:** July 31, 2026

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base
4. Findings by Research Question
5. Decision Analysis and Responsibility Matrices
6. Key Recommendations
7. Implementation Implications and Estimates
8. Risks, Constraints, and Open Questions
9. Validation Against the Research Plan
10. Next Steps and Handoff
11. References
12. Appendices

---

## 1. Executive Summary

### 1.1 Five-Minute Decision Brief

AEP-4789 Volume II establishes four specific OGC standards as one technical package for connected-system interoperability. Their jobs are complementary. CSAPI Part 1 organizes and exposes systems, deployments, procedures, sampling features, properties, and their relationships. CSAPI Part 2 handles observations, live and historical data, commands, command status, feasibility, and system events. SensorML describes systems and processes in detail. SWE Common defines the shapes, units, limits, quality information, and JSON/text/binary formats used for exchanged data.

The practical decision is to build Glaux Server around that division of labor. The server should not invent replacements when an adopted standard already covers the job. It must keep the four parts connected: data and commands must remain tied to the correct systems and properties; each stream must tell clients what shape its data uses; and the detailed descriptions and exchanged values must agree with those declared shapes.

Volume II does **not** say that every optional feature in all four standards is mandatory. The two CSAPI standards each offer named groups of requirements from which an implementation selects and declares its supported behavior. “Use the package as a coherent whole” therefore means use each standard for its proper job and record exactly what Glaux supports. That selection belongs to later research topics.

The biggest open issues are straightforward: the OGC foundation used by some create, update, and delete operations is still a draft; there is no approved CSAPI standard yet for publish/subscribe transport; the package does not provide NATO-specific security or shared-vocabulary rules; and it does not fully explain how a stream's data shape changes over time. These gaps do not invalidate the package, but Glaux must decide and test how to handle them rather than letting developers or AI fill them by guesswork.

### 1.2 Principal Conclusion

> **Glaux Server should implement one versioned, traceable Glaux standards profile over the four adopted standards: Part 1 for the resource graph, Part 2 for dynamic interaction, SensorML for rich descriptions, and SWE Common for data structure and encoding. The official OGC requirements, schemas, and normative abstract tests control technical conformance; AEP-4789 Volume II controls the NATO adoption context and package relationship.**

This conclusion is consistent with the accepted IDR-SRV-001 obligation boundary and IDR-SRV-002 functional responsibility baseline. It sharpens them without changing them.

### 1.3 What This Report Decides and Does Not Decide

This report decides the role of each adopted standard, the relationships Glaux must preserve, the validation layers later work must cover, the direct-server versus contract boundary, and the traceability structure for downstream research and implementation.

It does not select the final Glaux conformance-class set, design the Rust architecture or database, select a framework, create a security or DDIL profile, choose a pub/sub protocol, or execute the detailed requirement extraction assigned to later topics.

---

## 2. Scope and Plan Alignment

### 2.1 Topic Confirmation

This report executes only `IDR-SRV-003: AEP-4789 Volume II Standards Package Implementation Baseline`. No other IDR topic was executed, combined, or begun.

Completed in scope:

- independently reverified the fixed controlling package hash and reviewed all of Volume II;
- extracted its adoption, role, implementation, caveat, and future-work statements with dual PDF/publication anchors;
- reviewed the current approved editions of all four adopted OGC standards;
- inspected the official published schema trees and pinned mutable OpenAPI development artifacts;
- mapped primary and contributing responsibilities across the four standards;
- identified conformance, schema, OpenAPI, encoding, fixture, and interoperability implications;
- separated direct server responsibilities, server-side contracts, ecosystem responsibilities, future profiles, and exclusions;
- reconciled the result with accepted IDR-SRV-001 and IDR-SRV-002; and
- recorded downstream ownership and unresolved evidence limitations.

Deliberately excluded:

- full clause-by-clause Part 1 or Part 2 requirement extraction (`IDR-SRV-006` and `IDR-SRV-007`);
- final conformance-class selection (`IDR-SRV-008`);
- detailed SensorML or SWE representation design (`IDR-SRV-021` and `IDR-SRV-022`);
- final validation, persistence, streaming, tasking, security, or test-harness design;
- Rust framework, crate, database, deployment, or architecture selection; and
- implementation design for Publisher, Simulator, Web App, Mobile, physical systems, identity authorities, or gateways.

### 2.2 Research Question Coverage Matrix

| Plan Question ID | Question (Short Form) | Status | Evidence Location |
|---|---|---|---|
| CQ-1 | Adopted core standards | Complete | Sections 4.1 and 12.1 |
| CQ-2 | Relationship among the four standards | Complete | Sections 4.2 and 5.1–5.2 |
| CQ-3 | Direct server implementation obligations | Complete | Sections 4.3 and 5.3 |
| CQ-4 | Areas requiring later research | Complete | Sections 4.4 and 5.5 |
| CQ-5 | End-to-end traceability model | Complete | Section 4.5 |
| VAS-1 | Adopted technical package | Complete | Section 4.1.1 |
| VAS-2 | Required, recommended, optional, informative, and future areas | Complete | Section 4.1.2 |
| VAS-3 | CSAPI Part 1 role | Complete | Sections 4.1.3 and 5.1 |
| VAS-4 | CSAPI Part 2 role | Complete | Sections 4.1.3 and 5.1 |
| VAS-5 | SensorML role | Complete | Sections 4.1.3 and 5.1 |
| VAS-6 | SWE Common role | Complete | Sections 4.1.3 and 5.1 |
| VAS-7 | Caveats, assumptions, limits, and future references | Complete | Sections 4.1.4 and 8 |
| CSP-1 | Coherent four-standard model | Complete | Sections 4.2.1 and 5.2 |
| CSP-2 | Feature resources and metadata owner | Complete | Sections 4.2.2 and 5.1 |
| CSP-3 | Dynamic-data and interaction owner | Complete | Sections 4.2.2 and 5.1 |
| CSP-4 | System/process-description owner | Complete | Sections 4.2.2 and 5.1 |
| CSP-5 | Components, units, constraints, and encodings owner | Complete | Sections 4.2.2 and 5.1 |
| CSP-6 | Overlaps and later resolution | Complete | Sections 4.2.3 and 5.2 |
| SII-1 | API behavior to expose | Complete | Sections 4.3.1 and 5.3 |
| SII-2 | Areas to validate | Complete | Sections 4.3.2 and 4.4.2 |
| SII-3 | Information to store, link, or generate | Complete | Section 4.3.3 |
| SII-4 | Inputs accepted from clients and other Glaux components | Complete | Section 4.3.4 |
| SII-5 | Machine-readable documentation and declarations | Complete | Sections 4.3.5 and 4.4.1 |
| SII-6 | Rust, test, and CI implications | Complete | Sections 4.3.6 and 7 |
| CVI-1 | Classes, requirements, schemas, and validation resources | Complete | Section 4.4.1 |
| CVI-2 | Handoff to conformance research and harness | Complete | Sections 4.4.3 and 5.5 |
| CVI-3 | Schema-validation handoff | Complete | Sections 4.4.2 and 5.5 |
| CVI-4 | Fixture and golden-file handoff | Complete | Sections 4.4.2 and 5.5 |
| CVI-5 | Interoperability-test handoff | Complete | Sections 4.4.2 and 5.5 |
| BSC-1 | Direct Glaux Server concerns | Complete | Sections 4.6 and 5.3 |
| BSC-2 | Other-product and ecosystem concerns | Complete | Sections 4.6 and 5.3 |
| BSC-3 | Profile, future, and excluded concerns | Complete | Sections 4.6 and 8 |
| BSC-4 | Safe downstream scope language | Complete | Section 4.6.2 |

### 2.3 Prerequisite Validation

| Prerequisite | Evidence | Status |
|---|---|---|
| Current overall IDR plan | Version 1.3 working baseline; IDR-SRV-003 is the next scheduled topic | Satisfied |
| Current Glaux Server goal | Version 1.5 fixes Rust, open-source, full-scope reference-server intent | Satisfied |
| IDR-SRV-001 accepted | Report records Glaux Project Lead acceptance on July 30, 2026 | Satisfied |
| IDR-SRV-002 accepted | Report records Glaux Project Lead acceptance on July 31, 2026 | Satisfied |
| Controlling NATO package available | Hash independently matched before Volume II review | Satisfied |
| Official OGC sources reachable | Four standards and official schema directories retrieved on July 31, 2026 | Satisfied |
| Research report template available | Current repository template reviewed | Satisfied |

---

## 3. Evidence Base

### 3.1 Primary Sources Reviewed

| Source | Version / Date / Status | Authority Class | Stable Anchor | Access Date | Availability / Limitation |
|---|---|---|---|---|---|
| `AC/224(JCGISR)D(2026)0005`, *NATO Standard - STANAG 4789* | 27 April 2026; fixed project baseline; SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C` | Project-controlling ratification package | PDF pp. 1–59 | 2026-07-31 | Controlled local copy; not placed in the public repository; visibly pre-promulgation |
| Enclosure 3, *AEP-4789 Volume II, Sensor Integration Standard for NATO JISR Operations - Core APIs and Encodings* | Edition A, Version 1 | Controlling adoption and implementation-context source for this topic | PDF pp. 38–59; Preface; Chapters 1–2; Lexicon | 2026-07-31 | Cover, promulgation date, and approving authority retain placeholders |
| [OGC 23-001, OGC API - Connected Systems - Part 1](https://docs.ogc.org/is/23-001/23-001.html) | Version 1.0; approved; published 2025-07-16 | Adopted normative technical standard | Clauses 1–2, 7–19; Annex A | 2026-07-31 | Modular; no single Core requirements class |
| [OGC 23-002, OGC API - Connected Systems - Part 2](https://docs.ogc.org/is/23-002/23-002.html) | Version 1.0; approved; published 2025-07-16 | Adopted normative technical standard | Clauses 1–2, 8–16; Annex A | 2026-07-31 | Modular; pub/sub bindings assigned to Part 3 |
| [OGC 23-000, OGC SensorML Encoding Standard](https://docs.ogc.org/is/23-000/23-000.html) | Version 3.0; approved; published 2025-07-16 | Adopted normative model/encoding standard | Clauses 1–2, 7–9; Annex A | 2026-07-31 | General model requires profiles and external community semantics for some uses |
| [OGC 24-014, OGC SWE Common Data Model Encoding Standard](https://docs.ogc.org/is/24-014/24-014.html) | Version 3.0.0; approved; published 2025-07-16 | Adopted normative model/encoding standard | Clauses 1–2, 7–10; Annex A | 2026-07-31 | Defines no security solution |
| [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) | Version 1.5; Draft planning baseline; 2026-07-30 | Project scope and implementation authority | Sections 2–9 | 2026-07-31 | Project requirement, not a standards source |
| [Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md) | Version 1.3; Draft; 2026-07-31 | Controlling research governance | Rules, topic index, execution order | 2026-07-31 | Acceptance remains with plan owner |
| [IDR-SRV-003 Research Plan](../IDR%20Plans/idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline.md) | Complete; updated 2026-07-31 | Controlling topic scope and completion criteria | Questions, phases, success criteria, deliverable | 2026-07-31 | Acceptance is separately pending in this report |
| [IDR-SRV-001 Research Plan](../IDR%20Plans/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline.md) | Complete; updated 2026-07-30 | Accepted-prerequisite topic plan | Scope, source hierarchy, downstream baseline | 2026-07-31 | Project planning authority; not a standards source |
| [IDR-SRV-002 Research Plan](../IDR%20Plans/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities.md) | Complete; updated 2026-07-31 | Accepted-prerequisite topic plan | Scope, functional mapping, downstream baseline | 2026-07-31 | Project planning authority; not a standards source |
| [IDR-SRV-001 Report](idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md) | Final; accepted 2026-07-30 | Accepted project research baseline | Boundary, obligation, and traceability findings | 2026-07-31 | Does not override source standards |
| [IDR-SRV-002 Report](idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md) | Final; accepted 2026-07-31 | Accepted project research baseline | Function-to-responsibility mapping | 2026-07-31 | Does not override source standards |
| [Research Planning Approach](../../../../../Governance/research-planning-approach.md) | Version 1.3; Draft; 2026-07-31 | Controlling research-process governance | One-topic execution, evidence, report, acceptance, and handoff rules | 2026-07-31 | Process authority; not a technical source |
| [Research Plan Creation Prompt](../../../../../Governance/research-plan-creation-prompt.md) | No embedded version/date/status; repository baseline `d9c61aad69d05c87273a36d669bcba2d7ecdc642` | Project plan-creation instruction | Single-topic and source/dependency instructions | 2026-07-31 | Used for plan provenance and alignment only |
| [Research Plan Template](../../../../../Governance/research-plan-template.md) | No embedded version/date/status; repository baseline `d9c61aad69d05c87273a36d669bcba2d7ecdc642` | Required plan-structure template | Status, questions, phases, criteria, deliverable, dependencies | 2026-07-31 | Structural authority only |
| [Research Report Template](../../../../../Governance/research-report-template.md) | No embedded version/date/status; repository baseline `d9c61aad69d05c87273a36d669bcba2d7ecdc642` | Required report-structure template | Evidence, findings, decisions, validation, acceptance, handoff | 2026-07-31 | Structural authority only |
| [Overall Research Report Template](../../../../../Governance/overall-research-report-template.md) | No embedded version/date/status; repository baseline `d9c61aad69d05c87273a36d669bcba2d7ecdc642` | Final-synthesis structure | Indexed synthesis and aggregate acceptance fields | 2026-07-31 | Consulted for downstream compatibility; final synthesis not begun |

### 3.2 Official Supporting Artifacts

| Source | Version / Pin | Evidence Class | Stable Anchor | Access Date | Limitation |
|---|---|---|---|---|---|
| [OGC API - Connected Systems publication index](https://ogcapi.ogc.org/connectedsystems/) | Retrieved 2026-07-31 | Official package/publication index | Four Standards Documents entries | 2026-07-31 | Summary page; detailed standards control |
| [CSAPI Part 1 published artifacts](https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/) | Published version 1.0 tree | Official supporting schemas/OpenAPI/examples | Versioned directory | 2026-07-31 | Artifacts and examples do not independently create requirements |
| [CSAPI Part 2 published artifacts](https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/) | Published version 1.0 tree | Official supporting schemas/OpenAPI/AsyncAPI/examples | Versioned directory | 2026-07-31 | OpenAPI identifies itself as an example; the incomplete AsyncAPI support artifact is non-normative and not an approved Part 3 binding |
| [SensorML 3.0 JSON schemas](https://schemas.opengis.net/sensorML/3.0/json/) | Published version 3.0 tree | Official normative-support schemas | Versioned directory | 2026-07-31 | Schema validity alone is not complete SensorML or CSAPI conformance |
| [SWE Common 3.0 JSON schemas](https://schemas.opengis.net/sweCommon/3.0/json/) | Published version 3.0 tree | Official normative-support schemas | Versioned directory | 2026-07-31 | Cannot validate external semantic meaning by itself |
| [OGC Connected Systems development repository](https://github.com/opengeospatial/ogcapi-connected-systems/tree/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f) | Commit `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f` | Official mutable development context | Pinned tree; `api/part1/openapi` and `api/part2/openapi` | 2026-07-31 | Informative where not incorporated normatively; example OpenAPI files identify version `0.0.1` |
| [OGC API - Features status page](https://ogcapi.ogc.org/features/) | Retrieved 2026-07-31 | Official dependency-status source | Standards Documents list | 2026-07-31 | Still labels Part 4 as draft |
| [CSAPI Part 3 working material](https://github.com/opengeospatial/ogcapi-connected-systems/tree/a1f1f03b71f5f645486b23ec8b5fae1f9ba334bc/api/part3/standard) | Branch pin `a1f1f03b71f5f645486b23ec8b5fae1f9ba334bc` | Official work-in-progress context | Pinned draft tree | 2026-07-31 | Not approved and not adopted by AEP-4789 Volume II |
| [OS4CSAPI exemplar corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans) | Commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230` | Report-planning/style precedent only | Pinned research-plan tree | 2026-07-31 | Not technical or normative evidence |

### 3.3 Authority and Source-Strength Rules

The evidence was applied in this order:

1. Volume II controls **which standards and editions form the project’s NATO adoption package** and how their roles are described.
2. Each official OGC standard controls its **technical requirements, requirements classes, conformance classes, behavior, data model, and encoding rules**.
3. Official published schemas support structural validation where the normative standard invokes them.
4. OpenAPI, AsyncAPI, examples, repositories, reviewer guidance, and implementation material are supporting evidence unless a normative source gives them greater status.
5. Accepted Glaux reports control prior project conclusions but cannot override the NATO or OGC sources.
6. Analysis and project recommendations are labeled and do not become standards obligations merely because they are useful.

Volume II defines `shall`, `should`, and `may`. Its central package-alignment statement uses `should`, not `shall`. Imperative prose such as “use this volume” is direct implementation guidance but not a convention-defined `shall`. Section 2.2 also uses lowercase `must` four times when describing gaps that supporting profiles or guidance need to address; because `must` is not defined by the Conventions, this report preserves it as strong functional prose rather than upgrading it to a formal `shall`. The only relevant explicit `shall` within the technical content requires SI units unless otherwise specified; the separate document-handling `shall` is not server behavior.

### 3.4 Evidence Limitations

- The controlling package is the most-current draft identified by the project lead, but it does not prove completed NATO promulgation.
- Volume II intentionally summarizes the standards and directs implementers to the originals; it is not a substitute requirements specification.
- No exhaustive class-by-class requirement extraction was performed because that would improperly begin IDR-SRV-006 through IDR-SRV-008.
- Official schemas were reachable, but structural validation cannot prove all HTTP behavior, semantic consistency, graph integrity, policy compliance, or interoperability.
- The official Part 1 and Part 2 OpenAPI files identify themselves as examples and use `info.version: 0.0.1`. The Part 2 AsyncAPI file does not call itself an example, but is an incomplete, non-normative `0.0.1` support artifact. None is the sole contract or proof of Part 3 conformance.
- Artifact drift is concrete, not hypothetical: the Part 2 OpenAPI contains `/systems/{systemId}/history` paths that are not in the approved Part 2 requirements and omits Feasibility endpoints; the AsyncAPI has no `servers` or protocol bindings and uses a `controls/{controlStreamId}/commands` spelling inconsistent with normative `/controlstreams` paths.
- The approved publications themselves retain internal editorial contradictions. Examples include preliminary vendor media-type notes that conflict with the final normative media types, a Part 1 Annex A SensorML 2.1 prerequisite that conflicts with the clause's SensorML 3.0 prerequisites, and a Part 2 text-format abstract test that checks binary advertisement in one step. These must enter an errata/interpretation register before automation.
- A search of the official sources reviewed for this topic found normative Annex A abstract tests but did not establish the availability of executable OGC CITE suites for these four 2025 standards. This is an evidence gap, not proof that no such suite exists.

---

## 4. Findings by Research Question

### 4.1 CQ-1 and VAS-1–VAS-7: What Volume II Adopts

#### 4.1.1 Fixed Four-Standard Package

**Source-backed finding.** Volume II identifies exactly four normative technical references, all dated 16 July 2025:

| Adopted Publication | Volume II Role | Controlling AEP Anchor |
|---|---|---|
| OGC 23-001, CSAPI Part 1 | Feature-resource and structural metadata layer | Preface; §§1.1, 1.1.1; Table 1-1; Table 2-1 |
| OGC 23-002, CSAPI Part 2 | Dynamic-data and interaction layer | Preface; §§1.1, 1.1.2; Table 1-1; Table 2-1 |
| OGC 23-000, SensorML 3.0 | Rich system/process description | Preface; §§1.1, 1.1.3; Table 1-1; Table 2-1 |
| OGC 24-014, SWE Common 3.0 | Data components and encoding | Preface; §§1.1, 1.1.4; Table 1-1; Table 2-1 |

Anchors refer to AEP-4789 Volume II, PDF pp. 42–55 / publication pp. II–III and 1–8. Table 1-1 fixes the publications and URLs. The current official OGC headers confirm that all four are approved standards published on the same date. The OGC index displays `1.0.0` for the CSAPI standards while their document headers display `1.0`; SWE Common similarly appears as `3.0` or `3.0.0`. These are display differences for the same editions, not competing baselines.

#### 4.1.2 Strength and Optionality

**Source-backed finding.** Volume II:

- explicitly adopts the four standards for use under STANAG 4789;
- says implementations claiming alignment **should** use them as a coherent whole;
- directs implementers, testers, and acquisition authorities to the original standards for detailed requirements, schemas, classes, and encoding rules;
- requires SI units unless otherwise specified, while permitting operationally necessary non-SI units to be shown in parentheses after the SI value;
- says Volume II should be used alongside Volume I;
- allows adaptation or mediation for constrained or non-baseline environments;
- anticipates future profiles, conventions, SRDs, AEP volumes, and adjacent-standard coordination; and
- does not define modality-specific profiles, operator training, or tactics, techniques, and procedures.

**Analysis.** Explicit adoption makes the four standards the technical baseline, but it does not flatten their modular conformance structures. The AEP contains no statement selecting every optional OGC class or encoding. An exact Glaux profile remains necessary.

#### 4.1.3 Role of Each Standard

**Source-backed finding.** Volume II gives each standard a principal role:

- **Part 1:** discovery, identity, organization, access, and relationships for non-dynamic structural resources such as systems, deployments, procedures, sampling features, and property definitions; GeoJSON supports broad feature use and SensorML JSON supports richer descriptions.
- **Part 2:** dynamic, possibly real-time information moving out of systems through datastreams and observations and into systems through control streams and commands, together with command status, feasibility-related exchanges, and system events.
- **SensorML:** detailed machine-readable descriptions of physical and computational systems and processes, including capabilities, characteristics, interfaces, configurations, deployment, data flow, quality, geolocation, and lineage.
- **SWE Common:** low-level values, records, arrays, units, constraints, quality, structure metadata, and encoding methods used for observations, status, commands, tasking parameters, and static, dynamic, or streaming data.

These roles are consistent with the scopes of the official standards: Part 1 §§1–2; Part 2 §§1–2; SensorML §§1–2; and SWE Common §§1–2.

#### 4.1.4 Caveats and Future Boundaries

**Source-backed finding.** The package assumes web-suitable infrastructure, including HTTPS over IP as the primary applicability context, while acknowledging that constrained environments may need adaptation, mediation, or further guidance. It identifies gaps in optionality, domain semantics and metadata, security and trust, implementation conventions, federation, coalition use, accreditation, and coordination with other products or extensions. Future AEP volumes or SRDs may add profiles and detail, but do not change the present core package unless they say so explicitly. The Preface also recommends that future Volume II development and review cycles should, where practical, align with the public-review periods of the referenced open standards.

**Policy context.** Section 1.2 explains the adoption choice through NATO civil-standards policy and the standards' open development, technical maturity, public availability, modern-web alignment, public review and comment resolution, and NATO participation in OGC. This supports the selection rationale; it does not add server behavior beyond the adopted standards.

**Project interpretation.** These are gap markers and contract boundaries, not permission to invent hidden NATO requirements. Later research must either locate an authoritative profile or present an explicit Glaux decision.

### 4.2 CQ-2 and CSP-1–CSP-6: How the Package Works Together

#### 4.2.1 One Layered Contract, Not Four Alternatives

The simplest accurate model is:

`Part 1 resource identity and links → Part 2 streams and interactions tied to those resources → SensorML rich descriptions → SWE Common components and encoded values`

The arrows express dependency and linkage, not a request-processing sequence. SensorML and SWE Common are used within Part 1 and Part 2 representations; they are not separate HTTP service families.

**Source-backed dependencies include:**

- Part 2 §1 states that its dynamic feeds are associated with systems and features defined in Part 1.
- Part 1 §19.2 defines the SensorML representation class and maps CSAPI identity and resource fields into SensorML for applicable Part 1 resources.
- SensorML §8.1.2 depends on SWE Common for inputs, outputs, parameters, characteristics, capabilities, interfaces, and event properties.
- Part 2 §§9.6 and 10.6 expose the observation or command schema associated with each parent DataStream or ControlStream.
- Part 2 §16 binds selected Observation and Command representations to JSON or SWE Common JSON, text, or binary rules.

The four adopted standards are not a closed dependency set. Part 1 §8.1 `/req/api-common` normatively depends on OGC API - Features Part 1 Core and OGC API - Common Core, landing-page, and JSON requirements. Part 2 §8 depends on the Part 1 common class. Part 1 may use OGC API - Features Part 2 for non-CRS84 coordinate reference systems, and the Part 1/Part 2 write classes depend on draft OGC API - Features Part 4. These are inherited technical dependencies, not additional members of the four-standard AEP core package.

#### 4.2.2 Primary and Contributing Responsibilities

| Capability | Primary Standard | Contributing Standard(s) | Relationship Glaux Must Preserve |
|---|---|---|---|
| API landing, definition, resource navigation, identity, links | Part 1 | Part 2 | One coherent API surface and truthful declarations |
| Systems, deployments, procedures, sampling features, properties | Part 1 | SensorML; SWE Common inside rich descriptions | Stable identity and representation mapping |
| Datastreams and observations | Part 2 | Part 1 resources/properties; SWE schemas/encodings; SensorML context | Stream, producer, property, feature-of-interest, and schema links |
| Control streams, commands, status, feasibility, results | Part 2 | Part 1 systems/properties; SWE command schemas; SensorML capabilities | Command structure and target/capability consistency |
| System events and dynamic state | Part 2 | Part 1 system identity; SWE components where used | Event ownership, time, type, and resource linkage |
| Rich system and process description | SensorML | Part 1 SensorML representation; SWE components | CSAPI profile constraints in addition to generic SensorML validity |
| Values, units, constraints, records, arrays, quality | SWE Common | SensorML and Part 2 | One declared structure used consistently for schema and values |
| JSON/text/binary dynamic encoding | Part 2 + SWE Common | Stream schema | Values must be decoded and validated under the selected parent schema |

#### 4.2.3 Important Overlaps

| Overlap | Correct Division | Later Resolution |
|---|---|---|
| `Property` versus SWE/SensorML `definition` | Part 1 provides API-addressable property identity; SWE/SensorML carry semantic references inside components | IDR-SRV-015, 024 |
| Deployment | Part 1 owns API identity, navigation, and relationships; SensorML supplies the detailed deployment representation | IDR-SRV-006, 015, 021 |
| Interfaces and streams | SensorML can describe interfaces and data flow; Part 2 owns runtime DataStream/ControlStream resources and operations | IDR-SRV-007, 021, 034–036 |
| System status | Part 2 supplies dynamic resources and events; SWE can structure values; neither supplies a complete NATO readiness vocabulary | IDR-SRV-020, 024, 034 |
| Validation | JSON Schema checks structure; normative requirements and ATS check more; cross-resource and semantic tests check still more | IDR-SRV-008, 023, 050–053, 056 |

**Analysis.** Generic SensorML validity does not establish compliance with the additional Part 1 SensorML mappings. Likewise, a schema-valid Observation or Command does not establish that its values conform to the schema advertised by its parent stream. Coherent implementation requires cross-layer tests.

The word “status” also needs a representation boundary. SWE Common JSON/text/binary classes in Part 2 §§16.2–16.4 apply to Observation and Command values. CommandStatus, CommandResult, FeasibilityStatus/Result, and SystemEvent remain JSON resources under Part 2 §16.1. SWE may structure status information carried as observation data, but it does not turn every lifecycle resource into a SWE text or binary payload.

### 4.3 CQ-3 and SII-1–SII-6: Direct Server Implications

The following are **project implementation implications** derived from the adopted package, accepted Glaux scope, and official standards. They are not represented as new AEP `shall` statements.

#### 4.3.1 API Behavior to Expose

Glaux Server must ultimately expose the selected, accurately declared Part 1 and Part 2 behavior needed for its full project scope: resource discovery and navigation; structural resources and relationships; dynamic data; tasking and command lifecycle; status and events; supported representations; query and write behavior; schemas; and the implementation’s API definition. Exact operations and classes require IDR-SRV-006 through IDR-SRV-014.

#### 4.3.2 Areas to Validate

The server must validate at the layers applicable to each claimed capability:

- Part 1 and Part 2 HTTP, resource, query, link, representation, and lifecycle requirements;
- GeoJSON and Part 1 SensorML representation rules;
- SensorML conceptual/profile and JSON-schema constraints;
- SWE Common component schemas and selected value-encoding rules;
- Observation and Command values against the schema advertised by their parent stream;
- identifiers, links, time, units, constraints, controlled/observed-property alignment, and other cross-resource invariants; and
- authorization, policy, and project constraints not supplied by the four standards.

#### 4.3.3 Information to Store, Link, or Generate

**Analysis.** The standards prescribe observable behavior, not an internal database design. To deliver that behavior, Glaux will need durable or computable representations of resource identity and revision state, relationships, supported encodings, stream schemas and schema versions, observations, commands and their status/results, events, descriptive content, temporal context, and provenance needed by the selected profile. Whether a field is stored, derived, cached, or generated is deferred to resource-model and persistence research.

#### 4.3.4 Information Accepted from Clients and Other Products

Publisher, Simulator, external clients, Web App, and Mobile may submit or consume selected Part 1 resources, descriptions, streams, observations, commands, status, and events through the server’s standards-facing contract. Glaux Server owns validation, authorization, persistence effects, link integrity, response behavior, and truthful conformance for any operation it exposes. Producers remain responsible for the truth and provenance of submitted source information; actuators and mission systems remain responsible for physical assessment and execution.

#### 4.3.5 Machine-Readable Documentation and Declarations

Part 1 §8.3 requires an implementation API definition consistent with the inherited OGC API behavior. Glaux should publish its own implementation-specific OpenAPI description and accurate `/conformance` declarations. The official example OpenAPI files are valuable starting material and coverage input, but they are not a substitute for the implementation’s declared interface or for the normative requirements.

#### 4.3.6 Rust, Test, and CI Consequences

Rust implementation planning should preserve type and validation boundaries among API resources, SensorML descriptions, SWE components, and encoded values without prematurely fixing a crate or architecture. CI must eventually gate every claimed conformance class, representation, and cross-layer invariant with positive and negative tests. Claims should be generated from, or at least checked against, the same versioned profile used by implementation and tests.

### 4.4 CQ-4 and CVI-1–CVI-5: Conformance, Schema, and Validation

#### 4.4.1 Normative and Supporting Resources

| Layer | Authoritative Basis | Supporting Artifact | Key Limitation |
|---|---|---|---|
| Part 1 API/resources | Part 1 requirements classes §§8–19 and normative Annex A | Part 1 published schemas/OpenAPI/examples | No Core class; selected resource and encoding classes must be explicit |
| Part 2 API/dynamics | Part 2 requirements classes §§8–16 and normative Annex A | Part 2 schemas/OpenAPI/examples | No Core class; Part 3 binding is not included |
| SensorML model/JSON | SensorML requirements classes §§7–9 and Annex A | SensorML JSON schemas | Generic validity may not satisfy Part 1 profile mappings |
| SWE model/JSON/value encoding | SWE Common requirements classes §§7–10 and Annex A | SWE Common JSON schemas | Structure cannot prove external semantics or policy |
| OpenAPI | Part 1 API-definition requirements and selected API behavior | Official example OpenAPI files | Example is not the implementation contract or complete conformance oracle |
| Interoperability | All selected requirements plus cross-layer contracts | Reconciled examples, fixtures, external clients | No single artifact validates the complete package; published errata conflicts require explicit interpretation |

Both CSAPI standards state that there is no Core requirements class and that an implementation target is expected to implement at least one resource type and one encoding. SensorML and SWE Common use different modular conformance structures for models, JSON documents, and, for SWE, encoded values. There is therefore no honest one-line “conforms to all four” claim without a selected class set and evidence.

#### 4.4.2 Required Validation Stack

1. **Profile and declaration:** select exact requirements/conformance classes and advertise only those implemented and tested.
2. **API behavior:** test methods, status codes, queries, links, canonical URLs, paging, filtering, mutation behavior, content negotiation, and implementation OpenAPI consistency.
3. **Static resource syntax:** validate applicable Part 1 GeoJSON and SensorML representations and Part 2 JSON resources.
4. **SensorML profile:** validate SensorML §§7–9 and schemas, then the stricter Part 1 §19.2 mappings.
5. **SWE components:** validate SWE model/JSON requirements for components embedded in SensorML and stream schemas.
6. **Dynamic values:** validate Observation/Command values using the parent stream schema plus the selected Part 2 §16 and SWE §10 encoding rules.
7. **Semantic and graph integrity:** check URI, identifier, link, unit, property, time, status, and cross-resource consistency that JSON Schema cannot establish.
8. **Interoperability evidence:** run versioned positive and negative fixtures against Glaux and external clients/servers for every claimed resource/encoding combination.

Where normative requirements, notes, schemas, and Annex A test steps disagree, later conformance work must record the exact conflict, controlling interpretation, affected claim, pinned source, and any OGC issue or resolution. Tests must not silently encode an editorial contradiction. For the currently observed cases, normative Requirements 89–90 in Part 1 use `application/sml+json`; Part 2 Requirements 107/108, 115/116, and 123/124 use `application/swe+json`, `application/swe+text`, and `application/swe+binary`, despite retained preliminary-media-type notes. Part 1 Annex A's SensorML 2.1 indirect prerequisite and Part 2 Abstract Test A.115's binary-media-type step require explicit reconciliation with the SensorML 3.0 clauses and text-format requirement.

#### 4.4.3 Explicit Handoffs

- `IDR-SRV-008` must select and map the exact requirements/conformance classes and their dependencies.
- `IDR-SRV-023` must design the layered schema and encoding validation strategy.
- `IDR-SRV-050` must decide how the normative abstract tests become an executable conformance harness and verify the current availability of official executable suites.
- `IDR-SRV-053` must create sourced, reconciled, versioned positive and negative fixtures rather than copying examples uncritically.
- `IDR-SRV-056` must prove cross-layer behavior against external clients and implementations. Its omission from the topic plan’s “Blocks” list was a plan-list defect; the explicit research question and this handoff control.

### 4.5 CQ-5: Traceability Model

Every later requirement, design choice, implementation unit, and test should be traceable through one versioned record:

`AEP-II adoption anchor → adopted OGC edition → requirements/conformance class → requirement URI → schema or supporting artifact → selected Glaux profile rule → server capability and responsibility class → Rust implementation → automated evidence`

Minimum fields:

| Field | Purpose |
|---|---|
| Trace ID | Stable project identifier |
| AEP anchor and exact strength | Preserve adoption context without upgrading wording |
| OGC standard, edition, clause, class, and requirement URI | Point to controlling technical text |
| Artifact URI and version/pin | Reproduce schema/OpenAPI/example evidence |
| Glaux profile status | Selected, excluded, deferred, experimental, or unresolved |
| Server capability and responsibility class | Direct server, contract, ecosystem, future/profile, or excluded |
| Validation layer | API, structure, model, encoded values, semantics/graph, policy, interoperability |
| Uncertainty or dependency | Prevent hidden assumptions |
| Downstream topic / decision owner | Make unresolved work actionable |
| Code and test links | Added during implementation for auditable evidence |

This extends the traceability chain adopted in IDR-SRV-001 and IDR-SRV-002; it does not replace it.

### 4.6 BSC-1–BSC-4: Boundary and Scope Control

#### 4.6.1 Responsibility Boundary

| Class | Included Examples | Excluded or External Examples |
|---|---|---|
| Direct Glaux Server | Standards-facing API behavior; selected resources and encodings; validation; link and lifecycle integrity; API definition; conformance declarations; authorization enforcement for exposed operations | Physical sensing/actuation; operator decisions; global identity or ontology authority |
| Server-side contract | Publisher/Simulator ingestion; client queries; command submission and correlated status; adapters/gateways; external semantic and identity references | Internal design of Publisher, Simulator, Web App, Mobile, broker, or gateway |
| Downstream project design | Exact profile, persistence, Rust architecture, schema evolution, security, streaming, DDIL, test harness | Decisions not yet supported by this baseline |
| Ecosystem/profile | Accreditation, releasability, coalition trust, semantic registries, domain conventions, cross-domain release, federation topology | Not supplied by the four standards and not automatically absorbed by the server |
| Future or out of scope | Part 3 pub/sub, future AEP/SRD profiles, modality-specific schemas, arbitrary SensorML process execution | Not a current Volume II obligation |

#### 4.6.2 Required Downstream Language

Downstream reports should use this pattern:

> “Glaux implements the selected requirements and conformance classes of the four AEP-4789 Volume II editions as one coherent server profile. Part 1 governs the API resource graph, Part 2 governs dynamic interaction, SensorML governs rich system/process descriptions, and SWE Common governs reusable data components and encoded values. Exact obligations come from the cited OGC requirements; supporting artifacts and project recommendations do not independently create standards requirements.”

This wording avoids both errors: treating the package as four unrelated references, and claiming that adoption automatically selects every optional class, encoding, future part, or ecosystem policy.

---

## 5. Decision Analysis and Responsibility Matrices

### 5.1 Standards-Package Responsibility Matrix

| Standard | Primary Responsibility | Direct Server Concern | What It Does Not Supply |
|---|---|---|---|
| CSAPI Part 1 | Addressable feature resources, identity, discovery, navigation, relationships, metadata, GeoJSON/SensorML representations | Serve and manage selected systems, deployments, procedures, sampling features, properties, filters, links, and representations | Dynamic observation/command exchange; global identifier allocation; complete security policy |
| CSAPI Part 2 | Datastreams, observations, control streams, commands, status/results, feasibility, events, dynamic encodings | Serve, ingest, validate, query, and correlate selected dynamic resources | Physical execution; approved pub/sub binding; mission policy; complete readiness vocabulary |
| SensorML | Conceptual and JSON models for physical/computational systems, processes, capabilities, interfaces, configurations, deployment, lineage | Accept, produce where needed, validate, link, and return selected rich descriptions | Standalone API routing; runtime observation/command store; semantic-registry governance; requirement to execute arbitrary processes |
| SWE Common | Components, values, records, arrays, units, constraints, quality, structure, and JSON/text/binary value encoding | Validate and process selected components, stream schemas, observation results, status carried as observation data, and command parameters | JSON lifecycle-resource representation; resource identity/routing; transport; synchronization; authorization; trust adjudication |

### 5.2 Cross-Standard Dependency Matrix

| From | To | Dependency | Failure if Ignored |
|---|---|---|---|
| Part 1 common class | OGC API - Features Part 1 and OGC API - Common | Inherited API Core, landing-page, JSON, and related behavior | A CSAPI-looking surface that violates its normative API foundation |
| Part 1 optional CRS behavior | OGC API - Features Part 2 | Non-CRS84/CRS84h support uses the CRS-by-reference standard | Incorrect coordinate negotiation or false CRS claims |
| Part 2 common class | Part 1 common class | All Part 2 resource classes inherit the Part 1 API foundation | Dynamic endpoints inconsistent with the structural API |
| Part 1/Part 2 write classes | OGC API - Features Part 4 draft | Create/replace/delete/update behavior inherits an unapproved dependency | Unstable mutation behavior or disputed conformance |
| Part 2 dynamic resource | Part 1 system/property/feature | Dynamic data and interactions require stable subject, producer/target, and property relationships | Orphaned or ambiguous observations and commands |
| Part 1 SensorML representation | SensorML | Part 1 applies additional identity and resource mappings to valid SensorML documents | Schema-valid but CSAPI-invalid descriptions |
| SensorML | SWE Common | Inputs, outputs, parameters, capabilities, interfaces, and event properties use SWE components | Inconsistent descriptive and data structures |
| Part 2 stream | SWE Common or other declared schema | Parent DataStream/ControlStream defines the record/parameter schema and encoding | Values cannot be decoded or validated reliably |
| OpenAPI declaration | Selected Part 1/Part 2 behavior | Documentation must represent the actual implementation profile | Misleading clients and false conformance claims |
| All layers | Profile and tests | Optionality and gaps require an explicit selection and evidence record | Hidden scope, inconsistent code, and unauditable claims |

### 5.3 Server and Contract Boundary Matrix

| Concern | Glaux Server Owns | External Party Owns | Later Decision |
|---|---|---|---|
| Resource submission | Validation, authorization, canonical API response, persistence effect, link integrity | Publisher/Simulator/client source truth and provenance | Write profile, revision/conflict behavior |
| Observation ingestion | Contract, schema/encoding validation, association, time/query behavior | Producer measurement and source-quality claims | Late/out-of-order policy, schema evolution |
| Commands | Authorization gate, request validation, correlation, exposed lifecycle and results | Actuator/mission system feasibility, safety decision, and physical execution | Command approval, timeout, cancellation, audit profile |
| Rich descriptions | Accepted/returned representation and CSAPI mapping | Authoritative engineering facts and external vocabulary ownership | Canonical internal representation and round-trip rules |
| Security/trust | Enforcement at every exposed server boundary | Identity provider, accreditation authority, release authority, gateway operation | Authn/authz, releasability, audit, federation profile |
| Streaming | Part 2 HTTP and selected adopted behavior | Broker/network operations where external | Part 3 or project transport, ordering, replay, backpressure |
| DDIL | Correct cached/current/stale/valid state behavior selected by project | Network availability and disconnected operational procedures | Synchronization, conflict, retry, priority, offline authorization |

### 5.4 Options Considered

| Option | Benefit | Cost / Standards Risk | Decision |
|---|---|---|---|
| Treat the four standards independently | Simplifies local component ownership | Breaks required links and produces incompatible representations and validation gaps | Reject |
| Treat adoption as a mandate for every optional class and encoding | Appears maximally complete | Unsupported by AEP wording and modular OGC conformance; expands scope without decisions or tests | Reject |
| Implement a narrow minimum of one resource and one encoding | Meets the lowest generic CSAPI expectation | Contradicts Glaux’s full-scope project goal and omits required mission capabilities | Reject as project end state |
| Define a versioned full-scope Glaux profile over the four standards | Preserves coherent roles, makes optionality explicit, supports staged implementation and truthful testing | Requires careful class selection and gap decisions in later topics | Recommend |

“Full-scope” here describes the approved Glaux product goal, not an unsupported claim that every optional OGC feature is required. Implementation can be sequenced, while the target profile remains explicit.

### 5.5 Downstream Topic Handoff Matrix

| Topic(s) | Evidence and Decision Handed Forward |
|---|---|
| 004 | Crosswalk AEP terms, OGC terms, and Glaux terms; include command/status examples, standards package, system, stream, deployment, feature, observation, and event |
| 005 | Determine applicability of adjacent NATO standards without converting informative linkages into automatic server scope |
| 006–007 | Extract every selected Part 1/Part 2 requirement, dependency, representation, error, operation, and ATS test with exact URIs |
| 008 | Build the exact cross-standard conformance profile and dependency graph; resolve what “full-scope” selects |
| 009–014 | Resolve HTTP, resource, navigation, filtering, encoding negotiation, error, and implementation OpenAPI behavior |
| 015–020 | Design canonical resource, identity, linking, time, provenance, history, status, and event models while preserving the package roles |
| 021–024 | Define SensorML and SWE handling, layered schema validation, SI/non-SI unit policy, property identity, and external semantic governance |
| 031–038 | Define ingestion, dynamic data, streaming, tasking, feasibility, command lifecycle, safety, authorization, and external execution contracts |
| 039–043 | Supply the missing security, trust, releasability, federation, DDIL, synchronization, and conflict profiles |
| 044–049 | Select Rust and deployment mechanisms only after the standards boundaries and validation needs are known |
| 050–056 | Convert requirements and Annex A tests into executable evidence, fixtures, negative tests, quality gates, and external interoperability proof |

---

## 6. Key Recommendations

1. **Adopt a versioned Glaux standards profile as the single implementation baseline.**
   - Rationale: the four standards are modular and complementary; a profile records the exact full-scope target without false blanket claims.
   - Preconditions: detailed requirements and conformance work in IDR-SRV-006 through IDR-SRV-008.
   - Priority: High.

2. **Keep AEP adoption evidence and OGC technical requirements distinct in every trace record.**
   - Rationale: Volume II explains what is adopted and why; the OGC documents define what a conforming implementation does.
   - Preconditions: use the Section 4.5 traceability fields.
   - Priority: High.

3. **Preserve cross-standard identity, link, schema, and representation invariants as first-class test targets.**
   - Rationale: validating each document alone cannot prove coherent-package behavior.
   - Preconditions: resource model and validation designs.
   - Priority: High.

4. **Publish an implementation-specific OpenAPI description and truthful conformance declarations.**
   - Rationale: official example YAML is useful seed material but cannot describe Glaux’s exact selected behavior by itself.
   - Preconditions: class and endpoint selection.
   - Priority: High.

5. **Treat official schemas and normative Annex A tests as complementary, not interchangeable.**
   - Rationale: schemas validate syntax; requirements, behavior, semantics, graph integrity, and interoperability require additional tests. Published requirement/ATS contradictions must be logged and resolved explicitly rather than copied into the harness.
   - Preconditions: IDR-SRV-023 and IDR-SRV-050.
   - Priority: High.

6. **Create explicit project decisions for every package gap instead of embedding assumptions in code.**
   - Rationale: Part 4 writes, Part 3 pub/sub, security, semantics, schema evolution, DDIL, and status vocabularies are not fully resolved by the adopted package.
   - Preconditions: owning downstream research topic.
   - Priority: High.

7. **Keep alternative transports, draft dependencies, and future profiles isolated and clearly labeled.**
   - Rationale: experimental support must not be advertised as AEP-adopted or approved-standard conformance.
   - Preconditions: pinned inputs and separate tests.
   - Priority: Medium.

8. **Use the Volume II lexicon as crosswalk input, not as a silent replacement for OGC definitions.**
   - Rationale: AEP examples such as command-status dispositions help operational interpretation but do not define a complete normative state machine.
   - Preconditions: IDR-SRV-004.
   - Priority: Medium.

---

## 7. Implementation Implications and Estimates

### 7.1 Bounded Implementation Implications

| Area | Implication Established Here | Decision Deliberately Deferred |
|---|---|---|
| Domain boundaries | Separate resource/API, rich-description, component/schema, and encoded-value concerns while preserving one linked model | Rust modules, crates, traits, and internal architecture |
| Persistence | Preserve enough information for identities, links, revisions, stream schemas, dynamic values, lifecycle, time, and provenance | Database engine, schema, indexing, event sourcing, cache design |
| Validation | Support the eight-layer validation model and exact profile gating | Validator libraries, generated versus handwritten types, harness architecture |
| API documentation | Produce implementation-specific machine-readable API description | OpenAPI generation toolchain and source-of-truth workflow |
| Testing | Test every claimed class plus cross-layer positive/negative cases | Test framework, CI platform, fixture packaging |
| Extensibility | Isolate unapproved or profile-specific behavior from the adopted core | Plugin architecture or extension mechanism |

### 7.2 Effort Estimate

No defensible software-effort estimate follows from this baseline alone. Any estimate made now would hide unresolved decisions about the exact class profile, write operations and their draft dependency, representations and encodings, persistence, transport, security, semantic governance, schema evolution, and test evidence. Later architecture and roadmap work should estimate only after the owning research topics resolve those variables.

The approximately 1.0-hour value in the report header is AI-assisted research execution time, not an implementation estimate and not a claim that the equivalent human standards review would take one hour.

---

## 8. Risks, Constraints, and Open Questions

### 8.1 Material Risks and Constraints

| Risk / Constraint | Evidence | Effect | Required Treatment |
|---|---|---|---|
| Project baseline is pre-promulgation | AEP cover and promulgation placeholders | Cannot claim completed NATO promulgation | Preserve status in all citations; monitor authoritative change |
| Adoption mistaken for all-options mandate | AEP says coherent `should`; both CSAPI parts have no Core class | False scope and conformance claims | Select explicit Glaux profile in 008 |
| Four standards implemented independently | Cross-standard dependencies in P1 §19.2, P2 §§9–10/16, SensorML §8.1.2 | Broken identity, schema, and representation consistency | Cross-layer model and tests |
| Write classes depend on a draft | P1 §§17–18 and P2 §§14–15 reference OGC API - Features Part 4; official page still calls it draft | Churn or disputed conformance for required writable scope | Pin exact dependency, isolate, test, and monitor |
| No approved Part 3 binding | P2 §1 delegates pub/sub bindings to Part 3 | Transport, subscription, order, replay, backpressure, and QoS remain open | Separate transport decision; never call it AEP-adopted |
| Supporting artifacts treated as normative | Example OpenAPI files and incomplete AsyncAPI `0.0.1` artifact contain omissions or non-normative paths/spellings | Implementation may copy incomplete or stale behavior | Reconcile every artifact to requirements and the implementation's own API definition |
| Approved text and ATS contain editorial contradictions | P1 §19.2.2 vs Requirements 89–90; P1 Annex A SensorML 2.1 prerequisite; P2 §§16.2.2–16.4.2 vs final media-type requirements; P2 Abstract Test A.115 binary/text mismatch | A literal harness can fail correct behavior or certify the wrong behavior | Maintain pinned errata/interpretation records and link OGC issues/resolutions |
| Schema validity treated as full conformance | Standards contain behavioral and semantic requirements beyond schemas | False-positive tests | Use layered validation model |
| Stream-schema evolution is underdefined | Part 2 discussion favors new streams and leaves broader policy open | Historical decode and migration risk | Explicit versioning and evolution research |
| Sampling-feature and snapshot gaps | P1 defers concrete sampling-feature types; published Part 2 does not clearly realize the P1 overview’s snapshot reference | Temptation to invent behavior | Record as spec gaps; resolve in 006–008/015/034 |
| Security and trust profile absent | P1 does not mandate an auth method; P2 inherits it; SensorML recommends encryption and can carry external `securityConstraints`; SWE defines no security solution | Metadata/guidance does not provide authentication, authorization, releasability, audit, or trust enforcement | Dedicated security/profile topics 039–041 |
| Semantic governance absent | External definition URIs, units, vocabularies, and P2 §12.2.1 Table 20's `http://www.opengis.net/def/x-OGC/TBD/...` event identifiers | Structurally valid but semantically incompatible data | Topics 004, 020, 024 |
| DDIL capability overstated | Efficient encodings and web baseline do not define sync/replay/conflict | Operational failure under constrained connectivity | Topics 042–043; explicit contracts |

### 8.2 Open Questions and Owners

| Open Question | Why Unresolved Here | Owner |
|---|---|---|
| Which exact Part 1, Part 2, SensorML, and SWE requirements/conformance classes form the Glaux target? | Requires detailed extraction and dependency analysis | 006–008 |
| How will the project handle Part 4 write behavior while the dependency remains draft? | Needs requirement and compatibility decision | 006–008, 009 |
| Which live/pub-sub protocols and delivery guarantees will Glaux support? | Not selected by the adopted package | 007, 035 |
| How are stream schemas versioned and historical values decoded? | Standards do not supply a complete operational policy | 015, 022–023, 034 |
| Which canonical status, readiness, health, and event vocabularies apply? | Package provides resources but not a complete NATO vocabulary | 004, 020, 024 |
| Which authentication, authorization, releasability, audit, and trust profile applies? | Outside the four-standard technical package | 039–041 |
| How are external property/ontology/unit identifiers governed and cached? | JSON Schema cannot establish external meaning or authority | 004, 024 |
| What interpretations and OGC issue resolutions govern the observed specification/ATS contradictions? | The approved publications retain incompatible notes, prerequisites, or test steps | 006–008, 050–051 |
| What executable official conformance tooling exists for these 2025 editions? | Normative ATS found; executable suite availability not established | 050 |

The topic plan’s question about the authoritative Volume II copy is resolved: the plan owner registered the 27 April 2026 package as the most-current project-controlling copy, and its hash matched. The remaining caveat is pre-promulgation status, not copy uncertainty.

---

## 9. Validation Against the Research Plan

### 9.1 Methodology Phase Validation

| Phase | Work Performed | Required Output | Status |
|---|---|---|---|
| 1. Source collection and orientation | Verified package hash; reviewed Volume II structure; gathered official OGC publications, schemas, project inputs, and accepted reports | Source inventory and orientation notes | Complete; Sections 3 and 12.1 |
| 2. Adoption extraction | Reviewed every Volume II page; extracted adoption, roles, strength, caveats, boundaries, and future material with dual anchors | Anchored extraction table and classifications | Complete; Sections 4.1 and 12.1 |
| 3. Responsibility mapping | Reviewed official standard scopes/structure and mapped capabilities, dependencies, overlaps, and handoffs | Standards-to-capability matrix | Complete; Sections 4.2 and 5.1–5.2 |
| 4. Conformance/schema review | Identified class structures, Annex A tests, schemas, OpenAPI/AsyncAPI, validation layers, fixtures, and interoperability implications | Conformance/schema/validation inventory | Complete; Section 4.4 |
| 5. Boundary synthesis | Classified direct server, contract, design, ecosystem, future, and excluded concerns; bounded Rust/persistence/security/DDIL implications | Implementation boundary and risk list | Complete; Sections 4.6, 5.3, 7, and 8 |
| 6. Synthesis | Reconciled sources and prior reports; produced decisions, recommendations, trace model, risks, and handoffs | Decision-usable report | Complete; this deliverable |

### 9.2 Success Criteria Validation

| Success Criterion | Status | Evidence |
|---|---|---|
| Controlling Volume II reviewed | Met | Sections 3.1, 3.4, and 12.1 |
| Sources record title, version/date, location, status, and authority | Met | Sections 3.1–3.2 and 11 |
| Adoption language for all four standards anchored | Met | Sections 4.1 and 12.1 |
| Coherent four-standard relationship explained | Met | Sections 4.2 and 5.1–5.2 |
| Server responsibility areas mapped | Met | Sections 4.3, 5.1, and 5.3 |
| Conformance, schema, OpenAPI, validation, and encoding implications identified | Met | Section 4.4 |
| Downstream handoffs identified | Met | Sections 4.4.3 and 5.5 |
| Direct, contract, ecosystem, and out-of-scope boundaries drawn | Met | Sections 4.6 and 5.3 |
| Interpretation risks listed | Met | Section 8 |
| Recommendations are decision-usable and server-bounded | Met | Sections 6–7 |
| References are explicit and reproducible | Met | Sections 3 and 11; mutable sources pinned |

### 9.3 Deliverable Requirement Validation

| Required Content | Status | Location |
|---|---|---|
| 1. Executive summary | Met | Section 1 |
| 2. Scope and plan alignment | Met | Section 2 |
| 3. Evidence base and authority classification | Met | Section 3 |
| 4. Volume II adoption findings | Met | Section 4.1 and Appendix 12.1 |
| 5. Standards-package responsibility matrix | Met | Section 5.1 |
| 6. Four-standard relationship analysis | Met | Sections 4.2 and 5.2 |
| 7. Server implementation implications | Met | Sections 4.3 and 7 |
| 8. Conformance, schema, and validation implications | Met | Section 4.4 |
| 9. Downstream-topic handoff matrix | Met | Section 5.5 |
| 10. Recommendations | Met | Section 6 |
| 11. Risks, constraints, and open questions | Met | Section 8 |
| 12. Success-criteria validation | Met | Section 9.2 |
| 13. References | Met | Section 11 |

### 9.4 Independent Review Results

Three independent, read-only reviews were used within this topic:

- a complete second extraction of Volume II checked source anchors, wording strength, boundaries, and overclaim risks;
- an official-source standards review checked roles, dependencies, conformance structures, artifacts, validation layers, and gaps; and
- a plan audit counted and mapped all 33 questions, six phases, eleven success criteria, and thirteen deliverable requirements.

The reviews found no material conflict with accepted IDR-SRV-001 or IDR-SRV-002. They corrected two potential errors before publication: “coherent whole” cannot be reported as an all-options mandate, and `IDR-SRV-056` must remain an explicit interoperability handoff despite its omission from the original Blocks list.

---

## 10. Next Steps and Handoff

### 10.1 Acceptance State

Research and report review are complete, but plan-owner acceptance remains pending. This report must not be treated as accepted downstream until the Glaux Project Lead approves it and the `Accepted By` and `Acceptance Date` fields are updated.

### 10.2 Required Two-Action Transition

When the plan owner is satisfied, the next two actions are:

1. Accept `IDR-SRV-003`.
2. Authorize execution of exactly one next eligible topic, `IDR-SRV-004`.

Both actions can be given in one response:

> `Approve IDR-SRV-003. Then execute exactly one Glaux Server research plan: the next one, using the standing single-topic execution instructions.`

This wording in the report does not itself record acceptance or begin IDR-SRV-004. Only the plan owner’s combined response does so.

---

## 11. References

### 11.1 Controlled NATO Source

- NATO, `AC/224(JCGISR)D(2026)0005`, *NATO Standard - STANAG 4789*, 27 April 2026. Project-controlled copy; SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`.
  - Enclosure 3: *AEP-4789 Volume II, Sensor Integration Standard for NATO JISR Operations - Core APIs and Encodings*, Edition A, Version 1, PDF pp. 38–59.

### 11.2 Official OGC Standards and Artifacts

- Open Geospatial Consortium, [OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html), approved and published 2025-07-16. Accessed 2026-07-31.
- Open Geospatial Consortium, [OGC 23-002, *OGC API - Connected Systems - Part 2: Dynamic Data*, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html), approved and published 2025-07-16. Accessed 2026-07-31.
- Open Geospatial Consortium, [OGC 23-000, *OGC SensorML Encoding Standard*, Version 3.0](https://docs.ogc.org/is/23-000/23-000.html), approved and published 2025-07-16. Accessed 2026-07-31.
- Open Geospatial Consortium, [OGC 24-014, *OGC SWE Common Data Model Encoding Standard*, Version 3.0.0](https://docs.ogc.org/is/24-014/24-014.html), approved and published 2025-07-16. Accessed 2026-07-31.
- [OGC API - Connected Systems official publication index](https://ogcapi.ogc.org/connectedsystems/). Accessed 2026-07-31.
- [Official CSAPI Part 1 version 1.0 schemas, OpenAPI, and examples](https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/). Accessed 2026-07-31.
- [Official CSAPI Part 2 version 1.0 schemas, OpenAPI, AsyncAPI, and examples](https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/). Accessed 2026-07-31.
- [Official SensorML 3.0 JSON schemas](https://schemas.opengis.net/sensorML/3.0/json/). Accessed 2026-07-31.
- [Official SWE Common 3.0 JSON schemas](https://schemas.opengis.net/sweCommon/3.0/json/). Accessed 2026-07-31.
- [OGC Connected Systems repository at commit `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f). Supporting development evidence; accessed 2026-07-31.
- [OGC API - Features official status page](https://ogcapi.ogc.org/features/). Part 4 listed as draft on 2026-07-31.
- [CSAPI Part 3 working material at commit `a1f1f03b71f5f645486b23ec8b5fae1f9ba334bc`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/a1f1f03b71f5f645486b23ec8b5fae1f9ba334bc/api/part3/standard). Informative draft only; accessed 2026-07-31.

### 11.3 Project Sources

- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md), Version 1.5, 2026-07-30.
- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md), Version 1.3, 2026-07-31.
- [IDR-SRV-003 Research Plan](../IDR%20Plans/idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline.md).
- [IDR-SRV-001 Research Plan](../IDR%20Plans/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline.md).
- [IDR-SRV-002 Research Plan](../IDR%20Plans/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities.md).
- [Accepted IDR-SRV-001 Report](idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md).
- [Accepted IDR-SRV-002 Report](idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md).
- [Research Planning Approach](../../../../../Governance/research-planning-approach.md), Version 1.3, 2026-07-31.
- [Research Plan Creation Prompt](../../../../../Governance/research-plan-creation-prompt.md).
- [Research Plan Template](../../../../../Governance/research-plan-template.md).
- [Research Report Template](../../../../../Governance/research-report-template.md).
- [Overall Research Report Template](../../../../../Governance/overall-research-report-template.md).
- [OS4CSAPI research-plan exemplar corpus at commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans). Style precedent only.

---

## 12. Appendices

### 12.1 Volume II Adoption and Implementation Extraction Register

Strength codes: `AD` explicit adoption declaration; `SHALL`, `SHOULD`, and `MAY` preserve Volume II convention words; `FM` preserves lowercase `must` as strong functional prose because the Conventions do not define it; `IMP` imperative implementation direction; `DESC` descriptive role or rationale; `BOUND` explicit boundary/future statement.

| ID | Volume II Anchor | Strength | Finding | Classification / Handoff |
|---|---|---|---|---|
| V2-01 | Cover/promulgation, PDF pp. 38, 40 | Status evidence | Edition A Version 1 retains date and approval placeholders | Evidence limitation |
| V2-02 | Preface Scope, PDF p. 42 / pub. II | AD | Volume II is the core technical adoption volume for the four OGC standards | Direct baseline |
| V2-03 | Normative References, PDF p. 46 / pub. VI | Normative reference | Fixes OGC 23-001, 23-002, 23-000, and 24-014, dated 2025-07-16 | Version/source traceability |
| V2-04 | Conventions, PDF p. 47 / pub. VII | Interpretation | Defines `shall`, `should`, and `may` | Strength preservation |
| V2-05 | Conventions, PDF p. 47 / pub. VII | SHALL/MAY | SI units required unless otherwise specified; operationally necessary non-SI units may appear in parentheses after the SI value | 022–024; validation |
| V2-06 | §1.1, PDF pp. 48–49 / pub. 1–2 | AD/DESC | Four standards form the coherent core package | Direct/cross-standard |
| V2-07 | §1.1, PDF p. 48 / pub. 1 | DESC | CSAPI supplies resource-oriented, OpenAPI-described web API building blocks | 006–014 |
| V2-08 | §1.1, PDF p. 49 / pub. 2 | IMP | Original standards control detailed requirements, schemas, behavior, and encodings | 006–008, 021–023, 050–053 |
| V2-09 | §1.1.1, PDF pp. 49–50 / pub. 2–3 | DESC/AD | Part 1 is principal for structural feature resources, discovery, identity, organization, and access | Direct; 006, 015–019 |
| V2-10 | §1.1.1, PDF p. 50 / pub. 3 | DESC | GeoJSON provides broad feature use; SensorML JSON provides richer applicable descriptions | Representation; 006, 012, 021 |
| V2-11 | §1.1.2, PDF p. 50 / pub. 3 | DESC/AD | Part 2 is principal for observations/status outward and commands inward, with status, feasibility, and events | Direct/contracts; 007, 034–038 |
| V2-12 | §1.1.2, PDF p. 50 / pub. 3 | DESC | Part 2 supports JSON and SWE-based JSON, text, and binary options | Representation; 007, 012, 022–023 |
| V2-13 | §1.1.3, PDF pp. 50–51 / pub. 3–4 | DESC/AD | SensorML is principal for rich system/process descriptions, capabilities, configuration, interfaces, quality, geolocation, and lineage | Representation; 021 |
| V2-14 | §1.1.3, PDF p. 51 / pub. 4 | Capability | SensorML can describe aggregate executable processes | Future/boundary; no arbitrary execution duty |
| V2-15 | §1.1.4, PDF pp. 51–52 / pub. 4–5 | DESC/AD | SWE Common is principal for components, values, units, constraints, quality, and encodings | Representation/validation; 022–024 |
| V2-16 | §1.1.4, PDF pp. 51–52 / pub. 4–5 | DESC | Separating structure and values and using efficient encodings can aid constrained operation | DDIL input; not a sync protocol |
| V2-17 | §1.2, PDF p. 52 / pub. 5 | AD | Volume II adopts the four standards for use under STANAG 4789 | Adoption authority |
| V2-18 | Chapter 2 opening, PDF p. 53 / pub. 6 | IMP/SHOULD | Alignment claimants should use the package coherently and use original standards directly | Profile and conformance |
| V2-19 | §2.1/Table 2-1, PDF pp. 53–54 / pub. 6–7 | IMP/DESC | P1 structure + P2 dynamics + SensorML description + SWE representation form one role division | Cross-standard baseline |
| V2-20 | §2.2, PDF p. 54 / pub. 7 | MAY/FM/BOUND | Profiles/guidance may be needed; lowercase `must` identifies optionality, semantics/metadata, security/trust, conventions, federation, and accreditation gaps to address | 004–005, 008, 024, 039–043 |
| V2-21 | Preface/§2.2, PDF pp. 42, 54 / pub. II, 7 | MAY/BOUND | Primary web/HTTPS/IP baseline may need adaptation or mediation in constrained environments | Contracts; 035, 042–043 |
| V2-22 | Preface, PDF pp. 42–43 / pub. II–III | BOUND | No modality profile, operator training, or TTP; adjacent standards complement rather than replace package | 005; ecosystem/out of scope |
| V2-23 | §§2.3–2.4, PDF pp. 54–55 / pub. 7–8 | BOUND | Volume II realizes Volume I functions within scope; future volumes/SRDs may add independent profiles or detail | Future only unless explicitly adopted |
| V2-24 | Lexicon, PDF pp. 56–58 / Lex-1–Lex-3 | Definition | Defines package terms including command, status, connected system, streams, deployment, feature, observation, and event | 004 crosswalk input |
| V2-25 | Preface Application, PDF p. 43 / pub. III | SHOULD | Volume II should be used alongside Volume I | Operational-to-technical traceability |
| V2-26 | Preface Structure, PDF p. 43 / pub. III | SHOULD | Future development and review cycles should, where practical, align with referenced standards' public-review periods | Standards-governance context; not server behavior |
| V2-27 | §1.2, PDF p. 52 / pub. 5 | DESC | Civil-standards policy, openness, maturity, availability, web alignment, public review, comment resolution, and NATO OGC participation support adoption | Selection rationale; not an extra API requirement |

### 12.2 Terminology Handoff to IDR-SRV-004

| AEP-II Term | Immediate Caution for Crosswalk |
|---|---|
| Command / Command Status | AEP lists dispositions such as accepted, scheduled, executed, rejected, and cancelled; this is not a complete mandatory transition graph |
| Connected System / System | Broad operational definition must be compared with exact Part 1 resource semantics and SOSA/SSN concepts |
| DataStream / Control Stream | AEP role descriptions must be reconciled to exact Part 2 spellings, classes, links, and lifecycle |
| Deployment | Part 1 API resource and SensorML detailed representation overlap but are not interchangeable |
| Feature Resource / Feature of Interest / Sampling Feature | Closely related concepts require exact Part 1, OGC API Features, and SOSA/SSN mapping |
| Observation | AEP functional definition must be aligned with exact Part 2 resource and result-schema behavior |
| System Event | AEP definition is broader than any provisional identifier list; do not infer a final NATO vocabulary |
| Standards Package | Means the four adopted editions together, not every future CSAPI part or adjacent standard |

### 12.3 Report Completion Checklist

- [x] Topic ID matches the overall research plan index
- [x] Topic research plan and controlling overall plan are linked
- [x] Exactly one topic was executed
- [x] All 5 core and 28 detailed questions are covered
- [x] All six methodology phases and expected outputs are validated
- [x] All eleven success criteria are validated
- [x] All thirteen deliverable requirements are present and mapped
- [x] The complete controlling Volume II was reviewed and the package hash independently verified
- [x] All four official adopted standards and published schema directories were reviewed
- [x] Mutable technical and exemplar sources are commit-pinned
- [x] Adoption context, technical obligation, supporting evidence, analysis, and recommendation are distinguished
- [x] Direct server, contract, downstream design, ecosystem/profile, future, and excluded concerns are separated
- [x] Evidence limitations, draft dependencies, gaps, and open decisions are explicit
- [x] Findings are reconciled with accepted IDR-SRV-001 and IDR-SRV-002
- [x] Independent coverage, standards, and overclaim checks were completed
- [x] No other IDR research topic was begun
- [ ] Plan-owner acceptance recorded

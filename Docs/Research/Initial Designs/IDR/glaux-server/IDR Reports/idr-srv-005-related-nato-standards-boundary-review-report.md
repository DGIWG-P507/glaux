# Section 005: Related NATO Standards Boundary Review - Research Report

**Topic ID:** IDR-SRV-005<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-005 Related NATO Standards Boundary Review](../IDR%20Plans/idr-srv-005-related-nato-standards-boundary-review.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 20 detailed questions; all candidate-family, methodology, success-criterion, and deliverable requirements are mapped<br>
**Methodology Used:** Controlled-package integrity and reference extraction, authority classification, public catalog cross-check, bounded standard-family review, server-boundary classification, OGC package reconciliation, downstream handoff analysis, and independent evidence and completeness reviews<br>
**Research Time:** Approximately 1.5 hours of AI-assisted elapsed execution time, including parallel independent reviews, on July 31, 2026<br>
**Primary Sources:**

- Project-controlling `AC/224(JCGISR)D(2026)0005`, 27 April 2026, including STANAG 4789 and AEP-4789 Volumes I and II
- Official public U.S. DoD ASSIST catalog records for the identified NATO publications
- OGC API - Connected Systems Parts 1 and 2, version 1.0

**Supporting Resources:** Accepted IDR-SRV-001 through IDR-SRV-004 reports, DGIWG public metadata and standards resources, the Glaux Server Goal and Definition, and the project governance corpus<br>
**Document Purpose:** Establish which adjacent NATO publications matter at the Glaux Server boundary, what the server may need to reference or integrate with, and what it must not absorb into its core implementation<br>
**Author:** OpenAI Codex, with independent read-only evidence, boundary, and plan-completeness reviews<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** July 31, 2026<br>
**Date:** July 31, 2026<br>
**Last Updated:** July 31, 2026

---

## How to Read This Report

For the project decision, read Sections 1, 9, 11.2, and 13: they give the recommendation, tradeoffs, unresolved questions, and required next action. Sections 3–8 and 12 are verification detail so later researchers and implementers can audit the boundary without repeating this work or turning an adjacent standard into an accidental Rust-server requirement.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Findings by Research Question and Standard Family
5. Related-Standards Boundary Classification Matrix
6. Interoperability, Not Implementation
7. AEP Standards-Package Boundary Confirmation
8. Downstream Handoffs
9. Recommendations and Decision Analysis
10. Implementation Implications and Estimates
11. Risks, Constraints, and Open Questions
12. Validation Against the Research Plan
13. Next Steps and Handoff
14. References
15. Appendices

---

## 1. Executive Summary

### 1.1 Plain-English Decision Brief

Glaux Server should **not implement the ten related NATO standards named by STANAG 4789/AEP-4789 as part of its core**. They cover geospatial services and metadata, imagery and motion-imagery products, moving-target and tracking data, intelligence/surveillance/reconnaissance libraries, military messages and chemical/biological/radiological/nuclear reporting, and uncrewed-aircraft control-system interfaces. Those capabilities may exchange information with a Glaux deployment, but their appearance in the package is as “other related documents,” linkages, and informative references—not as additions to the four-standard server implementation package.

The core remains exactly what the accepted IDR-SRV-003 report established: OGC API - Connected Systems Parts 1 and 2, SensorML 3.0, and SWE Common 3.0. Those standards govern the public API, resource graph, dynamic interactions, rich descriptions, and data structures. Adjacent NATO systems can be connected through ordinary links, external-result references, publisher or gateway mediation, and later versioned profiles—a profile here means a separately approved, testable set of integration rules. Glaux must not claim conformance to an adjacent publication, expose its service family, parse its products, or translate its operational workflow unless a later approved artifact pins the exact publication baseline—including edition/version, amendment or corrigendum, and date—assigns a server responsibility, and supplies requirements and tests.

This is not a recommendation to ignore the NATO ecosystem. It is a recommendation to design a clean seam. The server should be able to preserve stable identifiers, links, provenance, declared formats, and policy metadata for external products and services. The component that understands a NATO imagery product, moving-target data file, motion-imagery stream, track exchange, ISR library, NATO message, chemical/biological/radiological/nuclear reporting workflow, or uncrewed-aircraft control interface should ordinarily be a publisher, adapter, gateway, client, or external mission system—not the CSAPI core.

No unresolved finding blocks IDR-SRV-006. The main limitation is that the NATO Standardization Document Database and most full adjacent publications were not publicly retrievable. That prevents field-level or conformance claims, but it does not weaken the boundary conclusion because the controlling AEP itself identifies the adjacent list as informative and separately fixes the four adopted OGC standards as normative.

### 1.2 Principal Conclusions

1. **No evidence reviewed establishes that an adjacent publication independently creates a direct Glaux Server implementation obligation.** Direct server obligations continue to come from the project-controlling STANAG/AEP adoption frame, the four adopted OGC standards, and accepted Glaux decisions.
2. **The ten related STANAGs define interoperability edges, not extra core modules.** Their likely touch points are external products, services, metadata, messages, catalogs, and control-system gateways.
3. **AEDP-2/NIIA is different from the ten-item related list.** AEP-4789 Volume I cites it normatively as the overarching ISR architecture context, but the draft leaves its exact edition/version/date unconfirmed. It supplies ecosystem alignment, not a verified server API contract.
4. **Future integration must be baseline-pinned.** Public catalog records show that several adjacent publications have evolved beyond the editions, amendments, or labels in the AEP-4789 informative list. A generic “support STANAG 4609,” for example, would be too ambiguous to implement or test honestly.
5. **The safe technical pattern is reference, mediate, and profile.** Preserve governed links and context in CSAPI resources; transform through external adapters when required; create a separate approved profile only when a real interoperability use case justifies it.
6. **The four-standard package is not displaced or contradicted.** No source reviewed supports replacing CSAPI, SensorML, or SWE Common with NSILI, legacy geospatial services, product formats, message standards, or platform-control interfaces.

### 1.3 Decision Recommended for Acceptance

This report establishes the following project recommendation for plan-owner acceptance:

> Adjacent NATO publications are interoperability context, not Glaux Server conformance sources. Unless a later approved profile pins the exact publication baseline and assigns a server responsibility, Glaux Server exposes only its selected CSAPI/SensorML/SWE contracts and may carry governed references to external artifacts or services. Native adjacent interfaces, product processing, transformation, validation, and operational workflows remain outside the core server.

This report does not select an adjacent profile, design an adapter, define new JSON fields, choose Rust modules, or begin any later IDR topic.

---

## 2. Scope and Plan Alignment

### 2.1 Topic Confirmation and Objective Coverage

This report executes only `IDR-SRV-005`. It reviewed every publication explicitly named by the controlling STANAG/AEP source, screened other references for plausible server-boundary relevance, classified each retained candidate, identified possible integration seams and explicit exclusions, confirmed the four-standard package, and assigned downstream questions.

| ID | Plan Objective | Result |
|---|---|---|
| OBJ-1 | Identify related NATO standards relevant to planning | Eleven candidates retained: AEDP-2/NIIA plus the ten related STANAGs explicitly named by the package |
| OBJ-2 | Identify interoperability boundary considerations | Complete; Sections 4–6 and Appendix 15.2 |
| OBJ-3 | State what is out of Glaux Server implementation scope | Complete; Sections 5–6 |
| OBJ-4 | Identify integration, mediation, profile, SRD, or ecosystem treatment | Complete; Sections 6 and 8 |
| OBJ-5 | Carry findings into later server research | Complete; Section 8 |

Out of scope here were full conformance extraction from any adjacent publication, implementation design for adjacent services or formats, modality-specific profiles, product parsers, operational TTPs, and detailed work belonging to IDR-SRV-006 or later topics.

### 2.2 Research Question Coverage Matrix

| ID | Plan Question, Short Form | Status | Evidence Location |
|---|---|---|---|
| CQ-1 | Which related NATO standards should be reviewed? | Complete | §§3.2, 4.1; Appendix 15.1 |
| CQ-2 | Which create direct implications versus boundaries? | Complete | §§4.2–4.8, 5 |
| CQ-3 | Which must stay out of server scope? | Complete | §§5–6 |
| CQ-4 | What boundary language should later topics use? | Complete | §6.1 |
| CQ-5 | What integration, mediation, or profile questions remain? | Complete | §§6.2–6.4, 8, 11.2 |
| RSI-1 | Which standards are explicitly mentioned or implied? | Complete | §§3.2, 4.1 |
| RSI-2 | Which appear in references, annexes, background, or linkages? | Complete | §§3.2–3.3, 4.1 |
| RSI-3 | Which cover sensor data, ISR products, geospatial, imagery, tracks, video, UAS, messages, or exchange? | Complete | §§4.3–4.7, 5 |
| RSI-4 | Which matter only through systems that use them? | Complete | §§4.3–4.7, 5–6 |
| RSI-5 | Which are irrelevant and should not carry forward? | Complete | §§4.1.2, 4.7, 6.3 |
| BC-1 | What kind of standard is each candidate? | Complete | §§4.3–4.7, 5 |
| BC-2 | Does it create a direct server obligation? | Complete | §§4.2, 5 |
| BC-3 | Does it create only an interoperability consideration? | Complete | §§4.3–4.8, 5 |
| BC-4 | Does it belong to another Glaux component or external system? | Complete | §§5, 6.4 |
| BC-5 | What evidence supports each classification? | Complete | §§3–5 |
| SI-1 | Could it affect identity, metadata, links, provenance, security, or exchange? | Complete | §§4.8, 6.2, 8 |
| SI-2 | Could it affect dynamic data, tasking, status, or events? | Complete | §§4.4, 4.6, 6.2, 8 |
| SI-3 | Could it affect persistence, query, negotiation, encoding, or validation? | Complete | §§4.8, 6.2–6.3, 8 |
| SI-4 | Could it affect external-client interoperability tests? | Complete | §§6.2, 8 |
| SI-5 | Is there a server hook or is it entirely external? | Complete | §§5, 6.2–6.4 |
| OSC-1 | Which standards should be explicitly not implemented? | Complete | §§5–6 |
| OSC-2 | Which may be supported indirectly? | Complete | §§5, 6.2 |
| OSC-3 | Which need future profile/SRD/AEP work? | Complete | §§5, 6.3, 8 |
| OSC-4 | Which create scope-creep risk? | Complete | §§6.3, 11.1 |
| OSC-5 | What wording should be handed forward? | Complete | §6.1 |

All 25 questions are answered. Evidence limits narrow technical conclusions but leave no question unanswered for this topic's boundary purpose.

---

## 3. Evidence Base and Authority Classification

### 3.1 Authority Rules Used

The report uses this order of authority:

1. The fixed project-controlling `AC/224(JCGISR)D(2026)0005` package establishes the Glaux NATO planning baseline and how its own references are classified.
2. AEP-4789 Volume II's four normative OGC references establish the server implementation package; the approved OGC publications establish their technical behavior.
3. Accepted IDR-SRV-001 through IDR-SRV-004 reports control existing Glaux boundary and terminology decisions.
4. Official public catalog records establish public title, publication, status, and access facts for adjacent publications; they do not substitute for the full controlled publications.
5. DGIWG public standards material supplies geospatial and metadata context; it does not silently adopt a NATO profile for Glaux.
6. Analyst inference identifies likely integration seams from the verified document type and Glaux responsibilities. A later approved profile must verify exact field mappings and requirements.

The labels **Standards obligation**, **Source-backed finding**, **Analysis**, and **Project recommendation** are used to prevent an inference from becoming a false standards obligation.

### 3.2 Controlling NATO Sources Reviewed

| Source | Version / Status | Authority for This Report | Stable Anchor | Availability / Limitation |
|---|---|---|---|---|
| `AC/224(JCGISR)D(2026)0005`, *NATO Standard - STANAG 4789* | 27 Apr 2026; project-controlling ratification draft | Fixed NATO planning baseline | PDF pp. 1–59; SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C` | Complete controlled local copy; not public; draft is not evidence of promulgation |
| STANAG 4789 enclosure | Edition 1 on memorandum; Edition A in header | Agreement and related-document classification | Other Related Documents, PDF p. 6 / publication p. 2 | Complete in fixed package; edition-label discrepancy retained |
| AEP-4789 Volume I | Edition A, Version 1 | Operational and architecture context | Preface Linkages, PDF p. 13 / publication p. III; Normative References, PDF p. 17 / publication p. VII; §4.4, PDF p. 33 / publication p. 15 | Complete in fixed package; AEDP-2 edition/version/date is expressly unconfirmed |
| AEP-4789 Volume II | Edition A, Version 1 | Technical adoption and boundary source | Preface Linkages, PDF p. 43 / publication p. III; References, PDF p. 46 / publication p. VI; §§1.1, 2.2–2.4, PDF pp. 48–55 / publication pp. 1–8 | Complete in fixed package; adjacent references are informative |

Integrity was rechecked on July 31, 2026. The PDF hash matched the registered baseline exactly.

### 3.3 Official and Authoritative Public Evidence

| Source | Public Record / Version | Use | Access / Limitation |
|---|---|---|---|
| [NATO Standardization Office](https://nso.nato.int/) and NSDD | Access attempted 2026-07-31 | Intended primary public catalog | HTTP 403 in this environment; no NSDD detail page was treated as reviewed |
| [French DGA NATO standards catalog](https://armement.defense.gouv.fr/normalisation/les-referentiels/normes-de-lotan) and its [September 2025 unclassified NSDD bibliographic export](https://armement.defense.gouv.fr/sites/default/files/2025-09/normes_OTAN_NON_CLASSIFIEES_09_2025.zip) | Official NATO-derived national catalog snapshot | Cross-checks publication identities, editions, and promulgation states where live NSDD access failed | Snapshot, not proof of changes after September 2025; metadata conflicts with ASSIST are reported rather than silently resolved |
| [ASSIST AEDP-2/NIIA](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=279510) | Active public U.S. catalog record; Volume I Revision B Amendment 1, 2018 | Confirms architecture-document identity and public catalog status | Full document requires login; does not resolve which edition the AEP-4789 draft intended |
| [ASSIST STANAG 6523](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=283622) | Active public U.S. record; AGeoP-26 Edition A, 2020 | Confirms geospatial-web-services identity | Full publication requires login; the September 2025 NATO-derived export instead reports STANAG 6523 Edition 2 / AGeoP-26 Edition B, so later work must resolve the catalog discrepancy |
| [NATO NISP Baseline 16 enclosure](https://nhqc3s.hq.nato.int/apps/nisp/NISP_Baseline_16_Catalogue_5SEP2024_enclosure_only.pdf) | 5 Sep 2024 public NATO catalog snapshot | Corroborates AGeoP-08 Edition B / STANAG 2586 Edition 2 and related NATO profiles | Direct fetch returned 403; searchable public index was available; not a substitute for AGeoP-08 content |
| [DGIWG Metadata Foundation](https://portal.dgiwg.org/files/9189) | DGIWG 114, 21 Nov 2014 | Confirms that STANAG 2586 is a geospatial metadata profile and illustrates metadata/catalog concerns | Older material based on Edition 1; not used as Edition 2 requirements |
| [ASSIST STANAG 4545](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=210436) | Active public U.S. record; Revision 3, 2024 | Confirms NSIF imagery-product identity | Full publication is controlled/login-gated |
| [ASSIST STANAG 4559](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=213759) | Active public U.S. record; Revision 4, 2018; AEDP-17/18/19 Edition A | Confirms ISR library interfaces and services identity | Full publications require login |
| [ASSIST STANAG 4607](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=277328) | Active public U.S. record; Revision 4 / AEDP-4607 Edition A, 2024 | Confirms GMTI-format identity | Full publication requires login |
| [ASSIST STANAG 4609](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=278534) | Active public U.S. record; Revision 5 / MISP-2019.1, 2020 | Confirms digital-motion-imagery identity | Full publication requires login |
| [ASSIST STANAG 4676](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=280456) | Active public U.S. record; Revision 2 / AEDP-12 Edition B, 2021; Amendment 2 promulgated in 2022 | Confirms ISR-tracking identity and amendment state | Full publication requires login |
| [ASSIST STANAG 7149](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=213806) | Active public U.S. record; Revision 7 / APP-11 Edition E, 2024 | Confirms NATO-message-catalog identity | Full publication requires login |
| [ASSIST STANAG 2103](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=97673) | Active public U.S. record; Revision 12 / ATP-45 Edition F, 2019; ATP-45 Edition F Amendment 3 listed in 2023 | Confirms CBRN warning/reporting/hazard-prediction identity and amendment risk | Full publication is controlled/login-gated |
| [ASSIST STANAG 4586](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=275263) | Active public U.S. record; Revision 4 / AEP-84 Edition A, 2017; Volumes I and II Edition A Amendment 1 listed active | Confirms UA control-system-interface identity and amendment state | Full publication requires login |
| [OGC API - Connected Systems Part 1](https://docs.ogc.org/is/23-001/23-001.html) | OGC 23-001, version 1.0, approved 2025-06-02; published 2025-07-16 | Core resource and cross-API linking boundary | Public and complete |
| [OGC API - Connected Systems Part 2](https://docs.ogc.org/is/23-002/23-002.html) | OGC 23-002, version 1.0, approved 2025-06-02; published 2025-07-16 | Dynamic-resource, encoding-extension, and external-result boundary | Public and complete |

ASSIST is an official U.S. DoD catalog and is useful corroborating evidence. Its “Active” field is reported here only as the public U.S. record; it is not presented as a substitute for NATO NSDD status across all Allies.

### 3.4 Prior Accepted Glaux Evidence

| Report | Accepted Finding Used Here |
|---|---|
| [IDR-SRV-001](idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md) | Direct server, contract, ecosystem, adjacent, and out-of-scope responsibilities must remain distinct |
| [IDR-SRV-002](idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md) | AEP operational outcomes cross multiple owners; an adjacent linkage is not a detailed server requirement |
| [IDR-SRV-003](idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md) | The adopted technical package is exactly CSAPI Parts 1/2, SensorML, and SWE Common |
| [IDR-SRV-004](idr-srv-004-terminology-and-concept-crosswalk-report.md) | Do not import an adjacent NATO vocabulary as controlling without an explicit adoption/profile decision |

### 3.5 Evidence Quality and Limitations

- The complete controlling STANAG/AEP package was available and verified. This makes its reference classification and package boundary high-confidence findings.
- The package is visibly a ratification draft. It controls Glaux planning but does not prove NATO promulgation.
- The NSDD returned HTTP 403. Most full adjacent publications were also login-gated or controlled. This report therefore makes no field-level, protocol-level, schema-level, or conformance claim about them.
- The French DGA export is an official NATO-derived September 2025 snapshot, not a live NATO catalog. ASSIST was updated more recently but demonstrably differs from that snapshot for some families. That version drift is decision-relevant evidence, not a basis for pretending either catalog resolves every Allied baseline.
- The DGIWG Metadata Foundation is older supporting material. It confirms the nature of the metadata family but is not substituted for current AGeoP-08 requirements.
- No unavailable source was reconstructed from memory, commercial summaries, or similarly named standards.

---

## 4. Findings by Research Question and Standard Family

### 4.1 Candidate Identification

#### 4.1.1 Retained Candidates

**Source-backed finding:** STANAG 4789's “Other Related Documents” section names STANAGs 2103, 2586, 4545, 4586, 4559, 4607, 4609, 4676, 6523, and 7149. AEP-4789 Volume II repeats those ten in its Linkages discussion and classifies them as informative references. AEP-4789 Volume I separately cites AEDP-2/NIIA normatively and states that the 4789 framework is a component of that architecture.

That produces an eleven-candidate inventory:

| ID | Publication / Family | Why Retained |
|---|---|---|
| NAT-01 | AEDP-2, NATO ISR Interoperability Architecture (NIIA) | Normative architecture context in AEP-I; exact edition unresolved |
| NAT-02 | STANAG 6523 / AGeoP-26, Defence Geospatial Web Services | Explicit principal linkage for feature, coverage, and metadata access |
| NAT-03 | STANAG 2586 / AGeoP-08, NATO Geospatial Metadata Profile | Explicit principal metadata linkage |
| NAT-04 | STANAG 4545 / AEDP-04, NATO Secondary Imagery Format | Explicit ISR imagery-product linkage |
| NAT-05 | STANAG 4559 / AEDP-17/18/19, NATO Standard ISR Library Interfaces and Services | Explicit ISR library/catalog linkage |
| NAT-06 | STANAG 4607 / AEDP-07 in AEP-4789; current public record uses AEDP-4607, NATO GMTI Format | Explicit ISR product linkage; designation transition must remain visible |
| NAT-07 | STANAG 4609 / MISP, NATO Digital Motion Imagery | Explicit ISR motion-imagery linkage |
| NAT-08 | STANAG 4676 / AEDP-12, NATO ISR Tracking Standard | Explicit tracking-product linkage |
| NAT-09 | STANAG 7149 / APP-11, NATO Message Catalogue | Explicit message-catalog linkage |
| NAT-10 | STANAG 2103 / ATP-45, CBRN Warning, Reporting, and Hazard Prediction | Explicit reporting/workflow linkage |
| NAT-11 | STANAG 4586 / AEP-84, UA Control System Interfaces | Explicit platform and control-system linkage |

#### 4.1.2 Screened but Not Carried as Adjacent Candidates

| Source | Disposition | Reason |
|---|---|---|
| AAP-03.1 | Supporting policy context only | Civil-standards adoption guidance, not a Glaux interoperability endpoint or product family |
| NATO Architecture Framework 4.1 | Supporting architecture method only | Does not assign a Glaux Server interface or product obligation |
| DGIWG 20-006 | Supporting geospatial standards-baseline context | DGIWG change proposal, not an adjacent NATO service/product implementation mandate |
| OGC 23-053r1 Reviewers Guide | Supporting CSAPI history only | The approved CSAPI standards, not the guide, control implementation |
| SOSA/SSN | Already handled semantic dependency | Covered by IDR-SRV-004 and the adopted OGC standards; not a related NATO publication |

No additional NATO standard was added merely because it might be useful. That is the principal control against turning this topic into a general NATO literature review.

### 4.2 Direct Obligation Versus Boundary

**Source-backed finding:** AEP-4789 Volume II lists only four normative references and calls the NATO candidates informative. Its preface says the linked publications complement the adopted package in adjacent functional areas rather than replace it. Part 1 of CSAPI itself allows linking to resources defined by other APIs, and Part 2 permits a `CommandResult` to reference an external dataset. These are integration mechanisms, not adoption of the target standard.

**Analysis:** A publication creates a direct Glaux Server obligation only if a controlling artifact does all three of the following:

1. identifies the publication and applicable edition;
2. assigns behavior to the server rather than to the broader deployment; and
3. supplies or points to requirements that can be traced and tested.

No evidence reviewed establishes that any of the ten related STANAGs passes that test in the current baseline. AEDP-2/NIIA has direct architectural and governance relevance because AEP-I names it normatively and places the 4789 framework within it. Its edition is expressly unconfirmed, however, and no server endpoint, representation, protocol, or testable behavior was extracted from it. The direct **implementation-obligation** result for this topic is therefore **none**.

That does not mean “no relationship.” The relationship classes used below are deliberately non-exclusive:

- **DIRECT** — a verified direct Glaux Server planning implication, which may be architectural rather than executable behavior;
- **IB** — interoperability boundary consideration;
- **HOOK** — possible server-side integration hook, usually a governed link, reference, or adapter contract;
- **ECO** — ecosystem-level responsibility;
- **PROFILE** — future profile, SRD, AEP, or explicit project-decision candidate; and
- **OUT** — native implementation is outside Glaux Server core.

`DIRECT` does not by itself mean “implement this publication.” NAT-01 receives it only for architecture/governance alignment. None of the ten related STANAGs receives it, and no candidate creates a native implementation or adjacent-conformance obligation.

### 4.3 Geospatial Services and Metadata

#### NAT-02 — STANAG 6523 / AGeoP-26

**Source-backed finding:** AEP-II identifies STANAG 6523 as a principal linkage for enterprise feature, coverage, and metadata access. The public ASSIST record identifies AGeoP-26 as Defence Geospatial Web Services and lists Edition A from 2020; the September 2025 NATO-derived DGA export lists a later STANAG 6523 Edition 2 / AGeoP-26 Edition B publication state. The disagreement is retained as version risk rather than resolved by assumption.

**Analysis:** CSAPI can describe connected systems and link to richer external feature or coverage representations. A Glaux deployment may therefore link a sampling feature, feature of interest, result, or external service to a geospatial endpoint. That does not require the Rust server to become a Defence Geospatial Web Services implementation.

**Project recommendation:** Permit standards-governed external links and identifiers after IDR-SRV-017 defines the link model. Do not add WMS, WFS, WCS, catalog, portrayal, tile, or other geospatial-service endpoints to core scope under IDR-SRV-005.

#### NAT-03 — STANAG 2586 / AGeoP-08

**Source-backed finding:** AEP-II identifies STANAG 2586 as the NATO Geospatial Metadata Profile and a principal metadata linkage. DGIWG public material confirms that the family concerns structured geospatial metadata and catalogs, including identification, extent, lineage, restrictions, and releasability-related metadata.

**Analysis:** Several concepts overlap with Glaux concerns—identity, title, extent, lineage, access constraints, and releasability—but overlap is not equivalence. CSAPI and SensorML remain the public resource representations. A later profile may map a verified subset of NATO geospatial metadata to Glaux resource metadata without replacing the CSAPI model.

**Project recommendation:** Carry a mapping question to IDR-SRV-019, IDR-SRV-024, and IDR-SRV-040. Do not claim AGeoP-08 conformance, ingest or emit its encoding, or make its fields mandatory until an exact publication baseline and mapping profile are approved and tested.

### 4.4 ISR Products, Motion Imagery, and Tracking

#### NAT-04 — STANAG 4545 / NSIF

**Source-backed finding:** The controlled package classifies STANAG 4545 as an informative imagery-product linkage; ASSIST identifies it as NATO Secondary Imagery Format.

**Analysis:** An observation, command result, or external artifact may refer to an NSIF product and preserve its identity, media-type declaration, provenance, time, and security context. The CSAPI core need not decode, validate, render, transform, or serve the native imagery format.

#### NAT-06 — STANAG 4607 / GMTI

**Source-backed finding:** The controlled package classifies STANAG 4607 as an informative GMTI-product linkage; ASSIST identifies the current public U.S. record as the NATO GMTI Format family.

**Analysis:** A publisher may transform selected source information into CSAPI observations or expose a link to the native product. Glaux Server must not become a GMTI parser, fusion engine, tracker, or native product service merely because those data originated with a connected system.

#### NAT-07 — STANAG 4609 / Motion Imagery

**Source-backed finding:** The controlled package identifies digital motion imagery as an informative product linkage; ASSIST identifies the public family as NATO Digital Motion Imagery Standard / MISP.

**Analysis:** CSAPI datastreams describe observation feeds and can link outward, but a CSAPI `DataStream` is not automatically a motion-imagery streaming protocol or media service. A gateway may publish metadata and a resolvable external stream/product reference. Native transport, processing, transcoding, playback, and product validation stay external.

#### NAT-08 — STANAG 4676 / ISR Tracking

**Source-backed finding:** The controlled package classifies STANAG 4676 as an informative tracking linkage; ASSIST identifies AEDP-12 as the NATO ISR Tracking Standard.

**Analysis:** Track data may be an external product, a source for derived observations, or a feature relationship. That does not make the CSAPI server a track-fusion engine or require it to expose the native tracking exchange. A future profile must define identity, time, geometry, lineage, and update mappings before any transformation is claimed.

**Project recommendation for NAT-04/06/07/08:** Use a publisher or domain adapter for native decoding and transformation. Let the server validate only the CSAPI/SensorML/SWE representation it actually accepts, while retaining a governed reference to the source artifact when policy permits.

### 4.5 ISR Library and Catalog Services

#### NAT-05 — STANAG 4559 / NSILI

**Source-backed finding:** AEP-II identifies NSILI as an informative ISR library-interface linkage. ASSIST identifies the current public U.S. record as NATO Standard ISR Library Interfaces and Services covering AEDP-17, AEDP-18, and AEDP-19.

**Analysis:** NSILI and CSAPI have neighboring discovery concerns but different standardization targets. A Glaux server may link to an external ISR library item or be indexed by an external library through a mediator. It should not implement an NSILI catalog/library service, map every library query into CSAPI, or claim NSILI conformance as part of the core.

**Project recommendation:** Treat NSILI federation as an external integration contract. If a concrete use case emerges, define a separately versioned gateway/profile after resource identity, query, security, and policy research are complete.

### 4.6 Messages, Reporting, and Platform Control

#### NAT-09 — STANAG 7149 / APP-11

**Source-backed finding:** AEP-II lists the NATO Message Catalogue as an informative linkage. The public ASSIST record identifies APP-11 Edition E in its current U.S. catalog record, while the draft AEP cites Edition D.

**Analysis:** Message identifiers or external reports may be referenced in provenance or event context, but Glaux Server is not a general military-message catalog, router, encoder, or validator. A future mission profile may define a particular message-to-resource mapping.

#### NAT-10 — STANAG 2103 / ATP-45

**Source-backed finding:** The controlled package identifies CBRN warning, reporting, and hazard prediction as a related publication. ASSIST confirms the ATP-45 title and operator-manual designation.

**Analysis:** ATP-45 concerns a domain-specific operational workflow. CBRN systems can be represented as connected systems, and outputs can be referenced or transformed into observations by a domain component. The core server does not generate warnings, predict hazards, execute the operator workflow, or implement the reporting standard. This candidate should not be carried into ordinary downstream server design except as a scope-control example, unless an approved CBRN profile is later requested.

#### NAT-11 — STANAG 4586 / AEP-84

**Source-backed finding:** The controlled package identifies UA control-system interfaces as a platform-interoperability linkage. ASSIST confirms AEP-84 as an interface-control-document family for UA control systems.

**Analysis:** CSAPI commands and command-status resources are not automatically equivalent to UA control-system messages or safety semantics. A dedicated gateway could translate between an approved subset only after command authorization, feasibility, lifecycle, safety, and audit decisions are defined. Flight control, vehicle safety, native interface state, and operator authority remain outside the server.

**Project recommendation for NAT-09/10/11:** Do not introduce native message or platform-control behavior. Preserve only generic external references and explicit adapter contracts until an exact mission profile is approved.

### 4.7 AEDP-2 / NIIA Architecture Context

**Source-backed finding:** AEP-I names AEDP-2/NIIA as its sole normative reference and repeatedly describes the 4789 framework as a component of NIIA. The reference entry itself says the exact edition, version, and date remain to be confirmed. ASSIST exposes a public active record for AEDP-2 Volume I Revision B Amendment 1 (2018), but the full document is login-gated.

**Analysis:** NIIA is a direct project-architecture and governance constraint, not evidence that Glaux Server must implement every NIIA publication or system role. It supports the already accepted layered model: Glaux Server owns its standards-facing contract, while libraries, gateways, mission applications, product services, security authorities, and sensor/actuator systems retain their responsibilities.

**Project recommendation:** Record NIIA as architecture context and a future consistency-review input. Do not extract or claim a server obligation from an edition that the controlling AEP has not identified. Resolve the exact baseline through a later approved architecture/profile action if a concrete decision depends on it.

### 4.8 Cross-Cutting Server Implications

The candidate review produces a bounded set of possible server implications. These are **project planning inputs**, not requirements created by the adjacent standards.

| Concern | Safe Core Implication | What Must Wait |
|---|---|---|
| Identity | Preserve stable CSAPI IDs/UIDs and allow governed external identifiers where the selected representation permits | Identifier syntax and authority mappings for each adjacent family |
| Links | Support the links required by CSAPI and a later approved model for external artifact/service references | New link relations, target contracts, dereference policy, and broken-link behavior |
| Provenance | Preserve source, derivation, time, procedure, and lineage needed to explain transformed data | Product-specific provenance mapping |
| Format | Record an exact declared media type or format identifier when the owning later topic provides a field and rules | Native parsing, validation, negotiation, or transformation |
| Security and releasability | Enforce Glaux authorization and policy at exposed resources and links | NATO marking/profile mapping and cross-domain release decisions |
| Dynamic data | Represent accepted results as CSAPI observations, events, commands, status, or external results only when their CSAPI semantics actually fit | Treating every product packet, track update, video frame, message, or platform command as a CSAPI resource by default |
| Persistence | Store core resources and, if later approved, external-reference metadata | Becoming the authoritative native repository for every adjacent product family |
| Testing | Test truthful CSAPI conformance and explicit adapter contracts | Claiming adjacent conformance without the exact publication baseline, requirements, and test evidence |

---

## 5. Related-Standards Boundary Classification Matrix

The matrix includes every field required by the topic plan. Multiple classification codes are intentional because a standard may create a boundary, an optional hook, and an out-of-scope implementation at the same time. The primary classification is listed first; any remaining codes are secondary tags.

| ID / Standard | Family or Domain | Source Availability | Relationship to STANAG/AEP | Relationship to Glaux Server | Classification | Evidence / Source Anchor | Server Implication, If Any | Explicit Non-Implementation Boundary | Downstream Handoff | Notes / Unresolved |
|---|---|---|---|---|---|---|---|---|---|---|
| NAT-01 AEDP-2 / NIIA | ISR architecture | AEP citation + public ASSIST metadata; full current family content unavailable | Sole normative AEP-I architecture reference; edition unconfirmed | Direct architecture/governance alignment; no verified API behavior | DIRECT, ECO, PROFILE, OUT | AEP-I Preface and Normative References, PDF pp. 13, 17; ASSIST AEDP-2 | Keep the layered server/adapter/ecosystem boundary and review architecture consistency | No NIIA conformance claim, full architecture implementation, or unverified requirement import | 045; final synthesis; project architecture governance | Exact multi-volume publication and amendment baseline must be confirmed if later relied upon |
| NAT-02 STANAG 6523 / AGeoP-26 | Geospatial web services | AEP summary + ASSIST metadata; full content unavailable | Related document; principal informative linkage | External feature/coverage/service link or gateway | IB, HOOK, ECO, OUT | S4789 PDF p. 6; AEP-II PDF pp. 43, 46; ASSIST 6523 | Governed links and identifiers after link model decision | No WMS/WFS/WCS/catalog/tile/portrayal service family in core | 012, 017, 032, 056 | Exact target service/profile must be pinned |
| NAT-03 STANAG 2586 / AGeoP-08 | Geospatial metadata profile | AEP summary; DGIWG context; full current content unavailable | Related document; principal informative metadata linkage | Possible future metadata mapping/profile | IB, HOOK, ECO, PROFILE, OUT | S4789 PDF p. 6; AEP-II PDF pp. 43, 46; DGIWG 114 | Preserve compatible identity/provenance/policy concepts under Glaux model | No AGeoP conformance or native encoding requirement in core | 019, 024, 040, 056 | Edition/profile and mapping need approval |
| NAT-04 STANAG 4545 / AEDP-04 | Secondary imagery product | AEP summary + ASSIST metadata; full controlled content unavailable | Related document; informative ISR-product linkage | External artifact/result reference; publisher mediation | IB, HOOK, ECO, OUT | S4789 PDF p. 6; AEP-II PDF pp. 43, 46; ASSIST 4545 | Preserve source link, declared format, provenance, time, policy when selected | No native NSIF decode, validate, transform, render, or product service | 012, 017, 023, 028, 032, 056 | Exact publication baseline, media type, and profile unresolved |
| NAT-05 STANAG 4559 / AEDP-17/18/19 | ISR library/catalog interfaces and services | AEP summary + ASSIST metadata; full content unavailable | Related document; informative library-interface linkage | External library link/index/gateway possibility | IB, HOOK, ECO, PROFILE, OUT | S4789 PDF p. 6; AEP-II PDF pp. 43, 46; ASSIST 4559 | Stable external identifiers and explicit federation adapter contract | No NSILI service, query model, or conformance claim in core | 011, 014A–G, 016–017, 032, 039–040, 056 | Concrete library use case and exact baseline required |
| NAT-06 STANAG 4607 / AEDP-07 (AEP label) | GMTI product/exchange format | AEP summary + public catalog metadata; full content unavailable | Related document; informative ISR-product linkage | Source product or external reference for derived observations | IB, HOOK, ECO, OUT | S4789 PDF p. 6; AEP-II PDF pp. 43, 46; ASSIST 4607 | Publisher transformation plus lineage; optional external reference | No GMTI parser, validator, fusion, tracker, or native service in core | 017, 019, 023, 027, 032, 034, 056 | AEP cites AEDP-07; current public record identifies STANAG 4607 Revision 4 / AEDP-4607 Edition A; exact baseline unresolved |
| NAT-07 STANAG 4609 / MISP | Digital motion imagery | AEP summary + ASSIST metadata; full content unavailable | Related document; informative motion-imagery linkage | External stream/product metadata and reference | IB, HOOK, ECO, OUT | S4789 PDF p. 6; AEP-II PDF pp. 43, 46; ASSIST 4609 | Link/resource metadata; publisher or media gateway contract | No native video transport, processing, transcoding, playback, or validation | 012, 017, 023, 032, 035, 054, 056 | AEP draft/public catalog version drift |
| NAT-08 STANAG 4676 / AEDP-12 | ISR tracking product/exchange | AEP summary + ASSIST metadata; full content unavailable | Related document; informative tracking linkage | Source/external track reference or later mapping | IB, HOOK, ECO, PROFILE, OUT | S4789 PDF p. 6; AEP-II PDF pp. 43, 46; ASSIST 4676 | Preserve identity/time/geometry/lineage if transformed | No native tracking exchange, track fusion, or tracking engine | 015–019, 026–027, 032, 034, 056 | AEP cites Edition A; ASSIST identifies Edition B and later Amendment 2 |
| NAT-09 STANAG 7149 / APP-11 | NATO message catalogue | AEP summary + ASSIST metadata; full content unavailable | Related document; informative message linkage | Possible external message identifier/reference | IB, HOOK, ECO, PROFILE, OUT | S4789 PDF p. 6; AEP-II PDF pp. 43, 46; ASSIST 7149 | Only an approved message-to-resource profile or external reference | No message catalog, router, encoder, or validator in core | 013, 017, 032, 041, 056 | AEP cites D; public record identifies E |
| NAT-10 STANAG 2103 / ATP-45 | CBRN warning/reporting/hazard workflow | AEP summary + ASSIST metadata; full controlled content unavailable | Related document; informative reporting linkage | Domain system/product may publish through an adapter | IB, ECO, PROFILE, OUT | S4789 PDF p. 6; AEP-II PDF pp. 43, 46; ASSIST 2103 | None in generic core beyond ordinary connected-system/observation contracts | No CBRN warning generation, hazard prediction, or operator workflow | 032 and 056 only if a CBRN profile is approved | Do not carry into generic design as a requirement; ATP-45 amendment state must be pinned if profiled |
| NAT-11 STANAG 4586 / AEP-84 | UA control-system interface | AEP summary + ASSIST metadata; full content unavailable | Related document; informative platform-interoperability linkage | Possible external command/status gateway | IB, HOOK, ECO, PROFILE, OUT | S4789 PDF p. 6; AEP-II PDF pp. 43, 46; ASSIST 4586 | Explicit adapter contract after command/security/safety research | No native UA-control protocol, flight control, safety logic, or assumed command equivalence | 032, 036–040, 055–056 | Exact mapping requires separate safety-reviewed profile |

### 5.1 Classification Result

- `DIRECT`: 1 of 11, architecture/governance relevance only (`NAT-01`)
- `IB`: 10 of 11
- `HOOK`: 9 of 11
- `ECO`: 11 of 11
- `PROFILE`: 7 of 11
- `OUT`: 11 of 11 for native/full implementation in the Glaux Server core

`NAT-10` is intentionally not assigned a standing hook: a generic server feature should not be invented for one modality-specific operator workflow. `NAT-01` is direct architecture/governance context rather than an API/product interoperability boundary or implementation mandate.

---

## 6. Interoperability, Not Implementation

### 6.1 Required Scope-Control Language

Later plans, reports, implementation guidance, issue templates, and architecture decisions should use the following wording or an equivalent that preserves all four controls:

> Glaux Server implements the selected requirements of OGC API - Connected Systems Parts 1 and 2, SensorML, and SWE Common as the AEP-4789 Volume II core package. Other NATO publications are adjacent interoperability context unless an approved Glaux or STANAG-4789 profile pins the exact publication baseline, assigns a server responsibility, defines the mapping, and provides verification criteria. The core may preserve governed references to external resources and support explicit adapter contracts, but it does not thereby implement or claim conformance to the referenced standard.

The four controls are:

1. pin the exact publication identifier, edition/version, amendment or corrigendum, and date;
2. identify a real use case and the server responsibility;
3. define a mapping or adapter contract without changing CSAPI semantics; and
4. test the behavior and advertise only truthful conformance.

### 6.2 Permitted Core Integration Seams

The following seams are consistent with the current package. Their exact data model belongs to later topics.

| Seam | Standards Basis | Boundary Rule |
|---|---|---|
| External resource links | CSAPI Part 1 §§7.9 and its general cross-API linking model | A link identifies and describes a relationship; it does not copy the target standard into Glaux |
| External observation results | CSAPI Part 2's JSON Observation encoding demonstrates a linked result representation using `result@link`; exact normative treatment remains for IDR-SRV-007 | A result link may identify an external representation; it does not prove that the server understands, validates, or conforms to the target format |
| External command results | CSAPI Part 2 §10.13 permits a `CommandResult` to reference an external dataset | The reference can point to an adjacent product; Glaux validates the CSAPI result representation, not the native dataset unless separately profiled |
| Extension formats | CSAPI Part 2 scope permits additional encodings through extensions | A separately named extension must define mapping, media types, validation, and conformance; no extension is selected here |
| Publisher/gateway mediation | Accepted IDR-SRV-001/002 responsibility model | Native decoding and transformation occur outside core; the server accepts and governs its own contract |
| Provenance and source identity | AEP operational context plus CSAPI/SensorML resource relationships | Preserve how a derived resource relates to its source without claiming semantic equivalence |
| Policy-aware link handling | Glaux goal and accepted security boundary | Authorization applies independently to the Glaux resource, link visibility, and target; the server does not auto-dereference or forward credentials without a later explicit trust and egress policy |

External links are security-bearing data. Later security and policy work must address SSRF-safe egress, URL and target-existence disclosure, credential forwarding, stale or broken targets, releasability, audit, and independent target authorization. Until those rules exist, a link is returned as governed metadata; it is not an instruction for the core server to fetch the target.

### 6.3 Explicit Scope-Creep Controls

Glaux Server core shall not acquire an adjacent capability merely because:

- the publication appears in the STANAG cover's related-documents list;
- an AEP preface says two capabilities may need to be used together;
- a CSAPI resource can link to an external object;
- a source payload can technically be stored in a document or object store;
- an observation `result` can carry an arbitrary structure;
- a `CommandResult` can reference an external dataset;
- a product concerns a sensor, track, image, stream, message, or unmanned platform; or
- a future AEP volume or SRD is possible.

Native support requires the four controls in §6.1. Until then, adjacent formats must not appear in Glaux conformance declarations, required request/response schemas, or default Rust-domain types.

Names that sound alike are not mappings. An NSILI collection is not automatically a CSAPI collection; a track is not automatically an Observation or Event; a STANAG 4609 stream is not automatically a CSAPI `DataStream`; an APP-11 report is not automatically a command, status, or event; and a STANAG 4586 message is not automatically a safe CSAPI command. Any such equivalence requires an accepted, baseline-pinned semantic profile and tests.

### 6.4 Responsibility Boundary

| Responsibility | Normal Owner | Glaux Server Core Responsibility |
|---|---|---|
| Acquire or generate native product/message | Sensor, mission system, product service, or domain application | None |
| Decode/validate native adjacent format | Publisher, sandboxable adapter/plugin, or dedicated external service | None unless a later profile explicitly assigns it; do not make parsers for untrusted native formats part of the default core attack surface |
| Transform into CSAPI/SensorML/SWE | Publisher or gateway under a versioned contract | Validate the submitted Glaux representation and preserve declared provenance |
| Store and retrieve CSAPI resources | Glaux Server | Direct core responsibility |
| Store native product bytes | External repository by default; possible later document-storage decision | At most a separately approved storage/reference capability, not format implementation |
| Resolve external library/service | Client, gateway, or federated service | Expose authorized link metadata and clear failure semantics if selected later |
| Translate CSAPI command to platform interface | Safety-reviewed control gateway/adapter | Authorize, validate, correlate, and report the CSAPI command lifecycle |
| Enforce release/security policy | Glaux server at its boundary plus external policy/cross-domain authorities | Enforce policy for its resources; never assume link visibility authorizes target access |

---

## 7. AEP Standards-Package Boundary Confirmation

### 7.1 Confirmation

**Source-backed finding:** AEP-4789 Volume II has four normative references—CSAPI Part 1, CSAPI Part 2, SensorML 3.0, and SWE Common 3.0—and ten adjacent NATO informative references. Sections 2.2 and 2.4 allow later guidance, profiles, SRDs, and volumes without changing the current package unless explicitly stated.

**Standards obligation:** For current Glaux planning, the implementable standards package remains those four OGC publications. The ten related NATO publications impose no additional core conformance obligation; the exact CSAPI requirements and selected conformance classes remain for IDR-SRV-006 through IDR-SRV-008.

**Conclusion:** The accepted IDR-SRV-003 package remains unchanged. No adjacent standard displaces, amends, or automatically extends it.

| Package Layer | Controlling Source | Effect of Adjacent NATO Publications |
|---|---|---|
| Public resource API | CSAPI Part 1 | May link to external resources; no replacement API |
| Dynamic data and control API | CSAPI Part 2 | May reference external results or use separately defined extensions; no native product/control protocol implied |
| Rich system/process description | SensorML | May record contextual identifiers and links; no adjacent product schema substitution |
| Data components and encodings | SWE Common | May structure accepted values; does not make every source format a SWE encoding |
| NATO operational/adoption frame | STANAG/AEP-4789 | Adjacent standards complement the package for relevant deployments; full implementation remains with owning capabilities |

### 7.2 Conflict and Version-Watch List

No verified technical conflict was found because full adjacent requirements were not available and none is currently adopted into the server contract. Seven reference-management issues must remain visible:

| Candidate | AEP-4789 Label | Public Catalog Evidence Reviewed | Consequence |
|---|---|---|---|
| AEDP-2/NIIA | Edition/version/date “to be confirmed” | ASSIST lists Volume I Revision B Amendment 1, 2018 | Do not infer which edition is normative for Glaux |
| STANAG 6523 | AGeoP-26 Edition A | ASSIST still exposes Revision 1 / Edition A, 2020; September 2025 NATO-derived DGA export reports STANAG Edition 2 / AGeoP-26 Edition B | Resolve the catalog disagreement and pin one complete baseline before profiling |
| STANAG 4609 | Edition 4, 19 Dec 2016 | ASSIST lists Revision 5 / MISP-2019.1, 2020 | A later profile must select one exact baseline |
| STANAG 4676 | AEDP-12 Edition A | ASSIST lists Revision 2 / AEDP-12 Edition B, 2021, with later Amendment 2 | Do not design against the draft label or base edition alone without verification |
| STANAG 7149 | APP-11 Edition D | ASSIST lists Revision 7 / APP-11 Edition E, 2024 | Message mapping must be edition-specific |
| STANAG 4545, 4559, 4607, and 2586 | Edition absent, broad, or older publication/designation label | Public records expose later or more specific publication structures, including AEDP-07 to AEDP-4607 designation drift | Generic support claims are not testable and must be rejected |
| STANAG 2103 and 4586 | Base edition identified | Public records expose later amendment states for ATP-45 and AEP-84 volumes | A profile must pin amendments as well as the base edition |

These are not blockers for CSAPI Part 1 research. They become blockers only if the project later chooses to implement a specific adjacent adapter or profile without obtaining the controlling publication.

---

## 8. Downstream Handoffs

The handoff records questions for later owners; it does not execute those topics.

| Downstream Topic(s) | Boundary Input | Required Later Decision or Check |
|---|---|---|
| IDR-SRV-006 | Adjacent standards add no Part 1 requirements | Extract Part 1 exactly; retain its linking model without inventing NATO-specific resource types |
| IDR-SRV-007 | Product packets and platform messages are not automatically Observations, Commands, or events | Extract exact Part 2 semantics, including linked Observation results and external `CommandResult` behavior, without inferring adjacent-format support |
| IDR-SRV-008 | Adjacent conformance is not part of the four-standard baseline | Keep adjacent profiles and test evidence separate from core conformance claims |
| IDR-SRV-011–014 | External catalogs, services, formats, and messages may affect query, negotiation, errors, and OpenAPI only after profile approval | Do not publish adjacent endpoints/media types by default; define link and target-failure semantics if selected |
| IDR-SRV-014A–014G | Existing implementations may expose adjacent features | Record them as implementation precedent, not Glaux requirements; detect scope leakage |
| IDR-SRV-015–019 | External products need identity, link, time, provenance, and possibly trust context | Define a generic, governed external-reference model before any family-specific mapping |
| IDR-SRV-021–024 | SensorML/SWE and semantic bindings remain distinct from adjacent encodings | Decide mappings, declared formats, schema layers, and vocabulary ownership only with exact publication baselines |
| IDR-SRV-025–030 | Native product storage is not implied | Decide reference versus managed blob/document storage without adding product semantics |
| IDR-SRV-031–032 | Publishers/adapters own native decoding and transformation by default | Define server acceptance, provenance, validation, idempotency, error, and authorization contracts |
| IDR-SRV-034–035 | CSAPI DataStream is not a native GMTI, video, track, or message transport | Define dynamic-data and streaming semantics independently; link or adapt external delivery |
| IDR-SRV-036–038 | CSAPI command lifecycle is not STANAG 4586 | Require a safety-reviewed, baseline-pinned gateway profile before translating commands |
| IDR-SRV-039–041 | External links, metadata, products, parsers, and commands cross security and policy boundaries | Define object/link authorization, SSRF-safe dereference policy, credential isolation, markings/releasability mapping, audit, parser isolation, and target-access separation |
| IDR-SRV-050–053 | Adjacent support needs separate requirements and fixtures | Core harness must reject false adjacent-conformance claims and test only approved extensions |
| IDR-SRV-055–056 | Control and external-client integration are high-risk boundary areas | Test gateway authorization/safety and linked/mediated interoperability only for approved profiles |
| IDR-SRV-057 | Final synthesis must preserve the accepted boundary | List any later adopted adjacent profiles separately from the AEP Volume II core package |

The formal Blocks list in the topic plan is fully covered: 006, 007, 008, 014A–014G, 015, 032, 039, 040, 056, and final synthesis all appear above.

---

## 9. Recommendations and Decision Analysis

### 9.1 Options Considered

| Option | Benefit | Cost / Risk | Standards Impact | Decision |
|---|---|---|---|---|
| Absorb all ten related STANAGs into Glaux Server | Appears comprehensive | Unbounded scope; inaccessible requirements; duplicate services; unsafe platform control; impossible conformance story | Converts informative references into false mandates | Reject |
| Ignore adjacent standards entirely | Keeps core small | Loses product provenance, external links, and integration readiness | Underserves the AEP's interoperability context | Reject |
| Implement a guessed “most useful” subset now | Creates early demos | Uses unverified editions and premature data mappings | Produces brittle, misleading profiles | Reject |
| Keep the four-standard core and define generic seams plus separately approved profiles/adapters | Preserves scope and interoperability; permits real use cases later | Requires disciplined identity, linking, provenance, policy, and adapter contracts | Matches the AEP's core-versus-adjacent structure | **Recommend** |

### 9.2 Key Recommendations

1. **Adopt the §6.1 scope-control language as the downstream baseline.**
   - Rationale: it is short enough for humans and precise enough to prevent AI-assisted scope drift.
   - Priority: High.

2. **Keep the core conformance surface limited to the selected four-standard package.**
   - Rationale: no adjacent publication is a verified direct server requirement.
   - Priority: High.

3. **Design one generic external-reference and mediation seam before any family-specific feature.**
   - Rationale: links, identity, provenance, declared format, and policy context solve the common boundary without importing ten models.
   - Preconditions: IDR-SRV-015–019, 023–024, 031–032, and 039–040.
   - Priority: High.

4. **Require a fully baseline-pinned profile decision for every adjacent integration.**
   - Rationale: several public records differ from the AEP draft's reference labels, and generic conformance claims cannot be tested.
   - Preconditions: access to the exact publication and a concrete use case.
   - Priority: High.

5. **Place native decoding, transformation, and platform translation behind adapters or gateways.**
   - Rationale: this preserves a standards-correct Rust CSAPI core and isolates high-change or safety-critical domain logic.
   - Priority: High.

6. **Treat AGeoP-08, NSILI, and STANAG 4586 as likely future profile candidates, not current features.**
   - Rationale: they touch metadata, federation, and command boundaries that Glaux must eventually model carefully, but no exact mapping is approved.
   - Priority: Medium.

7. **Carry STANAG 2103 only as a scope-control example unless a CBRN use case is approved.**
   - Rationale: its operator workflow is modality-specific and supplies no generic server requirement.
   - Priority: Medium.

8. **Create a version-watch record when an adjacent profile is proposed.**
   - Rationale: public catalogs and the draft AEP may not name the same edition or publication structure.
   - Priority: Medium.

---

## 10. Implementation Implications and Estimates

### 10.1 Bounded Implications for the Rust Server

This report does not choose a Rust architecture. It establishes constraints that the later architecture must preserve:

- the core domain model must remain CSAPI/SensorML/SWE-centered;
- adjacent standard identifiers and editions should be data/configuration, not hard-coded assumptions;
- native format libraries, whenever approved, should sit behind replaceable interfaces and separate feature flags or services rather than contaminate core types;
- external references need authorization, provenance, validation, and failure behavior;
- adapters must produce a complete, testable Glaux ingestion or command contract; and
- conformance output must distinguish core OGC claims from project extensions and external integrations.

### 10.2 Relative Complexity

| Work Item | Relative Complexity | Estimate | Assumptions |
|---|---|---|---|
| Apply boundary language to later research | Low | Included in normal topic execution | No code or new schema |
| Generic external-reference/link model | Medium | Estimate in IDR-SRV-015–017 | Must reconcile CSAPI encodings, authorization, and lifecycle |
| Provenance/format/policy metadata | Medium to High | Estimate in IDR-SRV-019, 023–024, 039–040 | Exact Glaux profile not yet selected |
| Publisher/gateway contract | Medium to High | Estimate in IDR-SRV-031–032 | Native products transformed outside core |
| One adjacent standard adapter/profile | Potentially High | Separate estimate per exact baseline/use case | Requires publication access, mapping, security, fixtures, and interoperability tests |
| Native implementation of all adjacent families | Unbounded / inappropriate | Not estimated | Rejected as outside the server goal |

No calendar estimate for an adjacent adapter is defensible until the project pins an exact publication baseline and use case.

---

## 11. Risks, Constraints, and Open Questions

### 11.1 Risks and Controls

| Risk | Consequence | Control |
|---|---|---|
| Informative reference treated as mandatory | Core scope expands by multiple service/product families | Apply §6.1 admission test and conformance separation |
| “Supports STANAG X” without a full baseline | Untestable and misleading claim | Pin identifier, edition/version, amendment/corrigendum, date, source hash, mapping, and tests |
| Arbitrary payload used to hide model ambiguity | Native product placed in Observation/Command without valid CSAPI semantics | Require explicit mapping and provenance; otherwise use external reference |
| Core parses every source format | Dependency, security, licensing, and maintenance explosion | Keep native handling in adapters/services |
| Native parser becomes part of the default trusted core | Malformed controlled product compromises the broad server surface | Use an optional sandboxable adapter/plugin with its own dependency, resource, and update controls |
| Link disclosure bypasses policy | Sensitive target or existence leaks across boundaries | Apply object/field/link authorization and separate target access policy |
| Server follows arbitrary links or forwards caller credentials | SSRF, confused-deputy access, credential leakage, or unintended cross-boundary retrieval | No automatic dereference; define egress allow-lists, credential isolation, target authorization, audit, and failure behavior in IDR-SRV-039–040 |
| Command translation assumes semantic equivalence | Unsafe or incorrect platform behavior | Safety-reviewed gateway profile; no direct 4586 mapping by default |
| Public catalog record mistaken for full standard | Unsupported field or protocol claims | Limit record use to identity/status/version evidence; obtain publication before profiling |
| Version drift between AEP and catalog | Wrong schema or interoperability target | Version-watch and project approval before implementation |
| Boundary interpreted as reduced product ambition | Useful interoperability never added | Preserve generic seams and adopt concrete profiles when evidence and use cases justify them |

### 11.2 Open Questions and Owners

| Open Question | Why It Remains Open | Owner / Resolution Topic | Blocks IDR-SRV-006? |
|---|---|---|---|
| Which AEDP-2 edition does final AEP-4789 intend? | Draft reference says to be confirmed; full authoritative resolution unavailable | STANAG/AEP custodian or later project architecture decision | No |
| Should Glaux adopt an AGeoP-08 metadata mapping? | Exact baseline and field-level content were not reviewed | IDR-SRV-019, 024, 040 after a use case/profile decision | No |
| What link relation and representation should identify an adjacent artifact/service? | Requires the canonical resource/link and content-negotiation models | IDR-SRV-012, 016–017, 023 | No |
| Should Glaux manage native artifact bytes or only references? | Depends on persistence, retention, policy, and use cases | IDR-SRV-028–030 | No |
| Which publisher/gateway transformations are required first? | Product priorities are ecosystem/project decisions | IDR-SRV-031–032 and roadmap | No |
| Is a STANAG 4586 gateway part of a future Glaux deployment? | Requires mission need, exact publication baseline, safety authority, and tasking profile | IDR-SRV-036–040 and project lead | No |
| Which adjacent integrations belong in external-client interoperability tests? | Depends on profiles actually adopted | IDR-SRV-056 | No |

None of these questions weakens the boundary or prevents Part 1 requirement research.

---

## 12. Validation Against the Research Plan

### 12.1 Methodology Phase Validation

| Phase | Work Performed | Required Output | Status / Evidence |
|---|---|---|---|
| 1. Source collection and candidate identification | Verified package hash; extracted every explicit related/normative reference; screened non-candidates; recorded availability | Candidate inventory with availability/relevance | Complete; §§3–4.1, Appendix 15.1 |
| 2. Boundary-relevant review | Identified type, overlap, relationship, source anchor, and uncertainty for all eleven candidates | Boundary notes per candidate | Complete; §§4.2–4.8 |
| 3. Classification and scope control | Applied six allowed classifications; stated implication, non-implementation boundary, and owner | Classification matrix | Complete; §5 |
| 4. Downstream handoff analysis | Mapped all formal Blocks and additional affected topics; supplied reusable language | Handoff matrix and language | Complete; §§6.1, 8 |
| 5. OGC/AEP package boundary check | Reconciled with accepted IDR-SRV-003 and official CSAPI linking/extension behavior | Package confirmation and conflict list | Complete; §7 |
| 6. Synthesis | Produced plain-language decision brief, recommendations, risks, open questions, and full validation | Polished report | Complete; this deliverable |

### 12.2 Success-Criterion Validation

| Topic Plan Success Criterion | Status | Evidence |
|---|---|---|
| STANAG/AEP reviewed for related references | Met | §§3.2, 4.1 |
| Candidates listed with availability and relevance | Met | §§3.3, 4.1; Appendix 15.1 |
| Every candidate classified in the required categories | Met | §5 |
| Evidence and reasoning recorded | Met | §§3–5 |
| Standards not to implement explicitly identified | Met | §§5–6 |
| Hooks, metadata/link, and validation implications identified | Met | §§4.8, 6.2 |
| Downstream handoffs identified | Met | §8 |
| Scope-expansion risks documented | Met | §§6.3, 11.1 |
| Recommendations are decision-usable and server-bounded | Met | §§1.3, 9 |
| References reproducible and access limits explicit | Met | §§3, 14 |

### 12.3 Deliverable-Content Validation

| Required Content | Status | Location |
|---|---|---|
| 1. Executive summary | Present | §1 |
| 2. Scope and plan alignment | Present | §2 |
| 3. Evidence base and authority classification | Present | §3 |
| 4. Candidate inventory | Present | §4.1 |
| 5. Findings by standard/family | Present | §§4.3–4.7 |
| 6. Classification matrix | Present | §5 |
| 7. Interoperability-not-implementation findings | Present | §6 |
| 8. Standards-package confirmation | Present | §7 |
| 9. Recommended scope-control language | Present | §6.1 |
| 10. Downstream handoff matrix | Present | §8 |
| 11. Recommendations | Present | §9 |
| 12. Risks, constraints, and open questions | Present | §11 |
| 13. Success-criteria validation | Present | §12.2 |
| 14. References | Present | §14 |

### 12.4 Classification-Matrix Field Validation

Section 5 contains all eleven required fields: standard/document identifier; family/domain; availability; relationship to STANAG/AEP; relationship to Glaux; classification; evidence/source anchor; server implication; explicit non-implementation boundary; downstream handoff; and notes/unresolved issues.

### 12.5 Dependency and Open-Question Validation

- The overall plan and goal were current.
- IDR-SRV-001 through IDR-SRV-003 were accepted before this turn.
- The Glaux Project Lead accepted IDR-SRV-004 and authorized only IDR-SRV-005 in the current combined instruction.
- The controlled package was available and hash-verified.
- All candidate standards were available at least as controlling AEP citations and public catalog metadata; inaccessible contents are explicit evidence limitations.
- The report template was available and followed.
- No prerequisite exception was used.
- The three plan-level open questions are answered in §§4–7 and any remaining edition/profile decisions are assigned in §11.2.
- All ten candidate-seed families in the plan are represented: 6523, 2586, 4545, 4559, 4607, 4609, 4676, 7149, 2103, and 4586; AEDP-2/NIIA is retained separately because the controlling AEP makes it normative architecture context.

### 12.6 Independent Review

Three independent read-only reviews covered plan and question completeness; authoritative candidate evidence and availability; and server-boundary, handoff, and usability analysis. Their corrections were incorporated: objective and candidate-seed traceability, the architecture-only `DIRECT` classification for NIIA, corrected classification counts, stronger non-equivalence and external-link security controls, and catalog-discrepancy evidence. The release audit checks Markdown structure, links, classification counts, candidate completeness, and the prohibition on beginning IDR-SRV-006.

---

## 13. Next Steps and Handoff

### 13.1 Current Status

Research, synthesis, validation, and repository review for `IDR-SRV-005` are complete. The Glaux Project Lead accepted this report on July 31, 2026, making it an approved downstream baseline.

No technical decision is required before acceptance unless the project lead disagrees with the core boundary in §1.3. The unresolved edition and profile questions are deliberately owned by later work and do not block the next topic.

### 13.2 Required Two-Action Transition

The next workflow transition has exactly two actions:

1. The Glaux Project Lead accepts `IDR-SRV-005`.
2. The Glaux Project Lead authorizes execution of exactly one next eligible topic, `IDR-SRV-006`.

Both actions were given in one instruction on July 31, 2026. That instruction recorded acceptance and authorized execution of `IDR-SRV-006`; it did not authorize any later topic.

Copyable combined instruction:

> Approve IDR-SRV-005 and execute exactly one Glaux Server research plan: IDR-SRV-006, using the standing single-topic execution instructions.

---

## 14. References

### 14.1 Controlled NATO Source

- NATO, `AC/224(JCGISR)D(2026)0005`, *NATO Standard - STANAG 4789*, 27 April 2026. Project-controlled ratification draft; SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`.
  - Enclosure 1: STANAG 4789, Other Related Documents, PDF p. 6 / publication p. 2.
  - Enclosure 2: AEP-4789 Volume I, Preface Linkages, PDF p. 13 / publication p. III; Normative References, PDF p. 17 / publication p. VII; §4.4, PDF p. 33 / publication p. 15.
  - Enclosure 3: AEP-4789 Volume II, Preface Linkages, PDF p. 43 / publication p. III; References, PDF p. 46 / publication p. VI; §§1.1 and 2.2–2.4, PDF pp. 48–55 / publication pp. 1–8.

### 14.2 Official OGC Standards

- [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001, version 1.0](https://docs.ogc.org/is/23-001/23-001.html), especially Abstract, §§1, 7.9, and cross-API linking discussion.
- [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002, version 1.0](https://docs.ogc.org/is/23-002/23-002.html), especially Scope and §10.13.
- [OGC SensorML Encoding Standard, OGC 23-000, version 3.0.0](https://docs.ogc.org/is/23-000/23-000.html).
- [OGC SWE Common Data Model Encoding Standard, OGC 24-014, version 3.0.0](https://docs.ogc.org/is/24-014/24-014.html).

### 14.3 Public NATO and Defence Catalog Evidence

- [NATO Standardization Office](https://nso.nato.int/) and [NATO Standardization Document Database](https://nso.nato.int/nso/nsdd/). Access attempted 2026-07-31; HTTP 403 in the research environment.
- [French DGA NATO standards catalog](https://armement.defense.gouv.fr/normalisation/les-referentiels/normes-de-lotan) and [September 2025 unclassified NSDD bibliographic export](https://armement.defense.gouv.fr/sites/default/files/2025-09/normes_OTAN_NON_CLASSIFIEES_09_2025.zip).
- [ASSIST AEDP-2/NIIA](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=279510).
- [ASSIST STANAG 6523](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=283622).
- [NATO NISP Baseline 16 Catalogue Enclosure, 5 September 2024](https://nhqc3s.hq.nato.int/apps/nisp/NISP_Baseline_16_Catalogue_5SEP2024_enclosure_only.pdf).
- [ASSIST STANAG 4545](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=210436).
- [ASSIST STANAG 4559](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=213759).
- [ASSIST STANAG 4607](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=277328).
- [ASSIST STANAG 4609](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=278534).
- [ASSIST STANAG 4676](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=280456).
- [ASSIST STANAG 7149](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=213806).
- [ASSIST STANAG 2103](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=97673).
- [ASSIST STANAG 4586](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=275263).
- [DGIWG Metadata Foundation, DGIWG 114](https://portal.dgiwg.org/files/9189), 21 November 2014.
- [DGIWG Defence Geospatial Standards Baseline](https://dgiwg.org/resources/dgsb/).

### 14.4 Project and Governance Sources

- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md).
- [IDR-SRV-005 Research Plan](../IDR%20Plans/idr-srv-005-related-nato-standards-boundary-review.md).
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md).
- [Research Report Template](../../../../../Governance/research-report-template.md).
- [Research Planning Approach](../../../../../Governance/research-planning-approach.md).
- [Accepted IDR-SRV-001 Report](idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md).
- [Accepted IDR-SRV-002 Report](idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md).
- [Accepted IDR-SRV-003 Report](idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md).
- [Accepted IDR-SRV-004 Report](idr-srv-004-terminology-and-concept-crosswalk-report.md).

---

## 15. Appendices

### 15.1 Candidate Source and Availability Register

| ID | Explicit Controlled-Source Anchor | Public Evidence Reviewed | Direct Content Availability | Confidence for Boundary |
|---|---|---|---|---|
| NAT-01 | AEP-I Preface / Normative References, PDF pp. 13, 17 | ASSIST AEDP-2 | Metadata only; controlling reference edition unresolved | High for ecosystem boundary; low for field-level requirements |
| NAT-02 | S4789 p. 6; AEP-II pp. 43, 46 | ASSIST 6523 | Metadata only | High |
| NAT-03 | S4789 p. 6; AEP-II pp. 43, 46 | NATO NISP snapshot; DGIWG 114 | Older related public technical context; current NATO publication unavailable | High for family/boundary; medium for mapping candidates |
| NAT-04 | S4789 p. 6; AEP-II pp. 43, 46 | ASSIST 4545 | Catalog only; controlled distribution | High |
| NAT-05 | S4789 p. 6; AEP-II pp. 43, 46 | ASSIST 4559 | Catalog only | High |
| NAT-06 | S4789 p. 6; AEP-II pp. 43, 46 | ASSIST 4607 | Catalog only | High |
| NAT-07 | S4789 p. 6; AEP-II pp. 43, 46 | ASSIST 4609 | Catalog only | High |
| NAT-08 | S4789 p. 6; AEP-II pp. 43, 46 | ASSIST 4676 | Catalog only | High |
| NAT-09 | S4789 p. 6; AEP-II pp. 43, 46 | ASSIST 7149 | Catalog only | High |
| NAT-10 | S4789 p. 6; AEP-II pp. 43, 46 | ASSIST 2103 | Catalog only; controlled distribution | High |
| NAT-11 | S4789 p. 6; AEP-II pp. 43, 46 | ASSIST 4586 | Catalog only | High |

### 15.2 Interoperability Pattern by Family

| Family | Native Object / Service Owner | Glaux-Facing Pattern | Core Validation Boundary |
|---|---|---|---|
| Geospatial service | External geospatial service | Link/identifier or gateway | Validate Glaux link/resource; target service validates itself |
| Geospatial metadata | Metadata producer/catalog | Later approved mapping plus provenance | Validate only selected Glaux profile fields |
| Imagery/GMTI/video/tracks | Product producer/repository/domain processor | External result/reference or publisher transformation | Validate CSAPI/SWE representation; native validator stays external |
| ISR library | NSILI service/gateway | External identifier/link/index mediation | Validate Glaux contract, not NSILI protocol |
| NATO message/CBRN report | Message/report producer and mission application | External reference or future mission profile | No generic native validation |
| UA control | Vehicle/UCS and safety-reviewed gateway | CSAPI command contract plus explicit translator | Validate/authorize CSAPI lifecycle; gateway owns native/safety behavior |
| NIIA | NATO/national enterprise architecture owners | Architecture consistency context | No server conformance claim from incomplete citation |

### 15.3 Report Completion Checklist

- [x] Topic ID matches the overall research-plan index
- [x] Topic research plan and controlling overall plan are linked
- [x] Exactly one topic was executed
- [x] All 5 core and 20 detailed questions are covered
- [x] All eleven retained candidates are identified and classified
- [x] Screened non-candidates have explicit dispositions
- [x] All six methodology phases and expected outputs are validated
- [x] All ten success criteria are validated
- [x] All fourteen deliverable requirements are present
- [x] All eleven required classification-matrix fields are present
- [x] The controlled package was available and its hash verified
- [x] Source status, authority, availability, and limitations are explicit
- [x] Standards obligations, source-backed findings, analysis, and project recommendations are distinguished
- [x] The four-standard package boundary is confirmed
- [x] Explicit non-implementation boundaries and integration seams are documented
- [x] Every formal downstream Block is handed off
- [x] Independent evidence, boundary, usability, and completeness reviews were performed
- [x] IDR-SRV-004 acceptance was recorded from the plan owner's combined instruction
- [x] IDR-SRV-006 and all later topics were unstarted when this report entered review
- [x] Plan-owner acceptance and acceptance date recorded

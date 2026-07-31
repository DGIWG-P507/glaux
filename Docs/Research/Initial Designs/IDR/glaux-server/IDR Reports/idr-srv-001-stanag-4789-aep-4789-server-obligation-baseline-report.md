# Section 001: STANAG 4789 / AEP-4789 Server Obligation Baseline - Research Report

**Topic ID:** IDR-SRV-001<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-001 Research Plan](../IDR%20Plans/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** 32 of 32 (5 core and 27 detailed questions)<br>
**Methodology Used:** Integrity verification of the fixed NATO package; complete review of the enclosing memorandum and all three enclosures; authority and normative-strength classification; obligation extraction; server-boundary mapping; current official OGC standards alignment; gap analysis; and independent coverage review<br>
**Research Time:** Approximately 0.5 hours of AI-assisted elapsed execution time on July 30, 2026<br>
**Primary Source(s):**
- Project-controlled `AC/224(JCGISR)D(2026)0005`, dated 27 April 2026, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`
- [OGC API - Connected Systems - Part 1: Feature Resources](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC SensorML Encoding Standard 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model Encoding Standard 3.0](https://docs.ogc.org/is/24-014/24-014.html)
**Supporting Resources:**
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research Planning Approach](../../../../../Governance/research-planning-approach.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
**Document Purpose:** Provide a reviewable NATO/AEP-derived obligation boundary and traceability baseline that later research can translate into a best-of-breed, open-source Rust OGC API - Connected Systems server without either shrinking the server below its mission or expanding it into the whole Glaux ecosystem<br>
**Author(s):** OpenAI Codex, for Glaux Project Lead review<br>
**Accepted By:** TBD pending Glaux Project Lead review<br>
**Acceptance Date:** TBD<br>
**Date:** July 30, 2026<br>
**Last Updated:** July 30, 2026

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base
4. Findings by Research Question
5. Decision Analysis and Standards-Package Alignment
6. Key Recommendations
7. Implementation Implications and Estimates
8. Risks, Constraints, and Open Questions
9. Validation Against the Research Plan
10. Next Steps and Handoff
11. References
12. Appendices

---

## 1. Executive Summary

The fixed source package required by the research plan was found, its SHA-256 matched exactly, and the complete 59-page package was reviewed. It contains the 27 April 2026 covering memorandum, draft STANAG 4789, AEP-4789 Volume I, and AEP-4789 Volume II. For Glaux planning, this is the project-controlling NATO baseline. The package itself is nevertheless visibly a pre-promulgation ratification draft: the memorandum says the standard is ready for ratification and the enclosure promulgation pages retain placeholders. This report therefore treats the package as controlling evidence for deriving and classifying project requirements while avoiding the unsupported claim that the package proves completed NATO promulgation.

The source hierarchy is coherent. STANAG 4789 supplies the NATO-level agreement and adopts AEP-4789. Volume I defines the operational problem, functional objectives, representative workflows, and environmental constraints. Volume II adopts four complementary OGC standards as the technical implementation package: CSAPI Part 1, CSAPI Part 2, SensorML 3.0, and SWE Common 3.0. All four adopted references match the current approved OGC publications dated 16 July 2025. The detailed, testable technical obligations are in those OGC standards, not in the AEP summaries.

The resulting boundary is straightforward:

> **Glaux Server is the canonical standards-facing implementation point for the AEP-4789 Volume II package. It owns the semantics, behavior, integrity, policy enforcement, and conformance of every API interaction it exposes. It does not own sensor operation, physical actuation, identity or accreditation authorities, cross-domain gateway operation, federation topology, or network availability; those are external responsibilities connected through explicit server-side contracts.**

Direct server responsibilities include standards-correct discovery and navigation; registration and description; access and exchange; observations and dynamic data; status and events; tasking, feasibility, and command lifecycle; preservation of identity, relationships, provenance, time, validity, freshness, quality, and units; authorization enforcement at each API boundary; and accurate conformance declarations backed by tests. Glaux Server must support the complete project goal, not the smallest technically permissible subset of modular OGC conformance classes.

Several matters are deliberately unresolved by the adopted package and must be handled in later research:

- “Use the package as a coherent whole” does not select the exact OGC requirements and conformance classes Glaux will claim.
- The optional CSAPI Part 1 and Part 2 Create/Replace/Delete and Update requirements classes—which Glaux will need for its intended writable scope—depend on OGC API - Features Part 4, which remained a draft awaiting approval on 30 July 2026.
- Approved CSAPI Part 2 provides dynamic resources and encodings, but pub/sub bindings are assigned to an unadopted Part 3 working draft.
- The package does not select a NATO security, authorization, federation, releasability, accreditation, or cross-domain profile.
- It expresses DDIL objectives but does not define offline replication, prioritization, resumption, reconciliation, or conflict algorithms.
- It provides status containers but not a complete NATO availability/readiness vocabulary or last-known-state policy.

The principal recommendation is to maintain a versioned Glaux conformance and obligation profile using this traceability chain:

`AEP operational need → adopted OGC standard → requirements/conformance class → requirement URI → Rust component → automated test → unresolved profile rule`

That profile, together with explicit integration contracts, will allow later AI-assisted implementation to be ambitious and standards-faithful without inventing requirements or absorbing the whole operational ecosystem into the server.

---

## 2. Scope and Plan Alignment

### 2.1 Topic Confirmation

This report executes only `IDR-SRV-001: STANAG 4789 / AEP-4789 Server Obligation Baseline`. It does not execute, combine, or begin another research topic.

Completed in scope:

- verified the project-controlling package before opening it;
- reviewed the enclosing memorandum and every page of all three enclosures;
- classified source authority and normative strength;
- extracted and anchored server-relevant obligations and implications;
- separated direct server, server-contract, ecosystem, adjacent, and out-of-scope responsibilities;
- assessed discovery, description, access, exchange, streaming, tasking, status, security, trust, federation, coalition, and DDIL concerns;
- checked the obligation baseline against the four current approved OGC standards adopted by Volume II;
- defined downstream traceability and topic handoffs; and
- recorded evidence limitations and unresolved interpretation risks.

Deliberately not performed:

- exhaustive extraction of every OGC `SHALL` or conformance test, which belongs to `IDR-SRV-006` through `IDR-SRV-008`;
- detailed Volume I functional decomposition beyond the obligation baseline, which belongs to `IDR-SRV-002`;
- detailed Volume II implementation profiling, which belongs to `IDR-SRV-003`;
- terminology adjudication, which belongs to `IDR-SRV-004`;
- review of the substantive content of adjacent NATO standards, which belongs to `IDR-SRV-005`;
- selection of a Rust framework, persistence architecture, security product, federation design, or DDIL algorithm; and
- behavior design for Glaux Web App, Mobile, Publisher, or Simulator beyond the contracts Glaux Server must expose.

### 2.2 Core Research Question Coverage

| Plan Question ID | Question (Short Form) | Coverage Status | Evidence Location |
|---|---|---|---|
| CQ-1 | Direct server obligations | Complete | Sections 4.1 and 12.2 |
| CQ-2 | Server, contract, ecosystem, and out-of-scope classification | Complete | Section 4.2 |
| CQ-3 | Functional obligation categories | Complete | Section 4.3 |
| CQ-4 | Security, trust, federation, coalition, and DDIL implications | Complete, with source gaps identified | Section 4.4 |
| CQ-5 | Downstream traceability structure | Complete | Section 4.5 |

All 27 detailed questions are accounted for individually in Appendix 12.1.

### 2.3 Methodology Completion

| Research Plan Phase | Work Performed | Output | Status |
|---|---|---|---|
| Phase 1: Source Collection and Authority Classification | Verified the fixed package hash; inventoried the memorandum, three enclosures, project authorities, official OGC standards, schema repository, and pinned mutable sources | Evidence inventory and authority hierarchy in Section 3 | Complete |
| Phase 2: STANAG / AEP Obligation Extraction | Reviewed all 59 PDF pages; extracted functional and technical statements; preserved exact PDF and publication anchors; classified strength | Obligation register in Appendix 12.2 | Complete |
| Phase 3: Server Boundary and Function Mapping | Mapped obligations to ten server capability areas; separated server, contract, ecosystem, adjacent, and excluded responsibilities | Sections 4.2–4.3 and downstream handoff matrix | Complete |
| Phase 4: OGC Standards-Package Alignment Check | Checked all four official approved OGC publications, current schemas, modular conformance structure, security guidance, and material draft dependencies | Section 5.2 alignment and gap matrix | Complete |
| Phase 5: Synthesis | Consolidated the evidence into a decision-usable baseline, recommendations, risks, traceability schema, and plan validation | This report | Complete |

---

## 3. Evidence Base

### 3.1 Primary Sources Reviewed

| Source | Type | Version / Release / Status | Authority Class | Stable Anchor | Access Date | Availability / Limitations |
|---|---|---|---|---|---|---|
| `AC/224(JCGISR)D(2026)0005`, *NATO Standard - STANAG 4789*, 27 April 2026 | Controlled ratification package | Fixed project baseline; SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C` | Project-controlling NATO package for this topic | PDF pp. 1–59 | 2026-07-30 | Local controlled copy; not stored or linked publicly; visibly pre-promulgation |
| Cover memorandum to `AC/224(JCGISR)D(2026)0005` | NATO covering document | Dated 27 April 2026 | Package status and purpose | PDF pp. 1–2, memorandum pp. -1–-2 | 2026-07-30 | Describes readiness for ratification and subsequent approval, not completed promulgation |
| Enclosure 1, *STANAG 4789, Sensor Integration Standard for NATO JISR Operations* | NATO standardization agreement draft | Cover: Edition 1; body header: Edition A | NATO agreement/adoption frame within project baseline | PDF pp. 3–7, STANAG pp. i and 1–3 | 2026-07-30 | Edition label inconsistency; promulgation fields incomplete |
| Enclosure 2, *AEP-4789 Volume I, Sensor Integration Standard for NATO JISR Operations - Reference View* | Allied Engineering Publication draft | Edition A, Version 1 | Controlling operational/reference view within project baseline | PDF pp. 8–37; Preface, Chapters 1–4, Lexicon | 2026-07-30 | Normative AEDP-2/NIIA reference has edition/version/date “to be confirmed” |
| Enclosure 3, *AEP-4789 Volume II, Sensor Integration Standard for NATO JISR Operations - Core APIs and Encodings* | Allied Engineering Publication draft | Edition A, Version 1 | Technical adoption source within project baseline | PDF pp. 38–59; Preface, Chapters 1–2, Lexicon | 2026-07-30 | Does not restate detailed OGC requirements; must be used with the adopted standards |
| [OGC 23-001, OGC API - Connected Systems - Part 1: Feature Resources](https://docs.ogc.org/is/23-001/23-001.html) | OGC implementation standard | Version 1.0; approved; published 2025-07-16 | Normative adopted technical standard | Clauses 1–2, 7–19, Annex A | 2026-07-30 | Modular conformance; no single Core class |
| [OGC 23-002, OGC API - Connected Systems - Part 2: Dynamic Data](https://docs.ogc.org/is/23-002/23-002.html) | OGC implementation standard | Version 1.0; approved; published 2025-07-16 | Normative adopted technical standard | Clauses 1–2, 7–16, Annex A | 2026-07-30 | Pub/sub bindings assigned to Part 3; provisional event vocabulary identifiers remain |
| [OGC 23-000, OGC SensorML Encoding Standard](https://docs.ogc.org/is/23-000/23-000.html) | OGC implementation standard | Version 3.0; approved; published 2025-07-16 | Normative adopted technical standard | Clauses 1–2, 7–9, Annex A | 2026-07-30 | Community semantics and profiles remain external concerns |
| [OGC 24-014, OGC SWE Common Data Model Encoding Standard](https://docs.ogc.org/is/24-014/24-014.html) | OGC implementation standard | Version 3.0.0; approved; published 2025-07-16 | Normative adopted technical standard | Clauses 1–2, 7–10, Annex A | 2026-07-30 | Explicitly makes no security considerations |
| [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) | Project governance | Version 1.5, dated 2026-07-30; repository snapshot `f3c4bb10ca2895528ae02976eedb538f94e4dc46` | Project scope and implementation authority | Goal, capability scope, product boundary, fixed Rust decision | 2026-07-30 | Governs Glaux ambition and boundary; not a standards source |
| [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md) | Project governance | Version 1.2, dated 2026-07-30; repository snapshot `f3c4bb10ca2895528ae02976eedb538f94e4dc46` | Controlling research governance | Topic index, governance rules, execution order, progress tracking | 2026-07-30 | Acceptance remains with the Glaux Project Lead |
| [Research Planning Approach](../../../../../Governance/research-planning-approach.md) | Project governance | Version 1.2, dated 2026-07-30; repository snapshot `f3c4bb10ca2895528ae02976eedb538f94e4dc46` | Controlling research-process governance | One-topic execution, report quality, source handling, acceptance rules | 2026-07-30 | Governs research process; not a standards source |
| [Research Plan Template](../../../../../Governance/research-plan-template.md) | Project template | No embedded version/date; repository snapshot `f3c4bb10ca2895528ae02976eedb538f94e4dc46` | Required plan structure | Status, phases, success criteria, deliverable and acceptance handling | 2026-07-30 | Structural authority only |
| [Research Report Template](../../../../../Governance/research-report-template.md) | Project template | No embedded version/date; repository snapshot `f3c4bb10ca2895528ae02976eedb538f94e4dc46` | Required report structure | Evidence, findings, recommendations, validation, references and acceptance | 2026-07-30 | Structural authority only |
| [Overall Research Report Template](../../../../../Governance/overall-research-report-template.md) | Project template | No embedded version/date; repository snapshot `f3c4bb10ca2895528ae02976eedb538f94e4dc46` | Final-synthesis structure | Downstream final-report compatibility | 2026-07-30 | Consulted for handoff compatibility; not used to begin final synthesis |

### 3.2 Supporting Sources Reviewed

| Source | Version / Release / Commit | Relevance / Evidence Class | Stable Anchor | Access Date | Limitations |
|---|---|---|---|---|---|
| [OGC API - Connected Systems landing page](https://ogcapi.ogc.org/connectedsystems/) | Retrieved 2026-07-30 | Official publication index | Standards Documents and Schema Repository | 2026-07-30 | Supporting index; standards remain authoritative |
| [OGC Connected Systems schema repository](https://schemas.opengis.net/ogcapi/connected-systems/) | Published Part 1 and Part 2 version 1.0 trees | Official implementation/schema support | `part1/1.0` and `part2/1.0` | 2026-07-30 | Subordinate to normative specification text |
| [Part 2 AsyncAPI support artifact](https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/asyncapi/asyncapi-connectedsystems-2.yaml) | Published Part 2 schema tree; artifact declares version `0.0.1` | Official but non-normative support artifact | Exact AsyncAPI file | 2026-07-30 | Must not be treated as an approved pub/sub binding: approved Part 2 assigns pub/sub bindings to Part 3 and contains no pub/sub conformance class |
| [OGC SensorML 3.0 schemas](https://schemas.opengis.net/sensorML/3.0/json/) | Published 2025 schema set | Official implementation/schema support | JSON schema directory | 2026-07-30 | Subordinate to normative specification text |
| [OGC SWE Common 3.0 schemas](https://schemas.opengis.net/sweCommon/3.0/json/) | Published 2025 schema set | Official implementation/schema support | JSON schema directory | 2026-07-30 | Subordinate to normative specification text |
| [OGC Connected Systems public repository](https://github.com/opengeospatial/ogcapi-connected-systems/tree/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f) | `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`, committed 2026-04-20 | Official mutable development context | Pinned repository tree | 2026-07-30 | Informative; repository README includes draft-era language |
| [CSAPI Part 3 working draft](https://github.com/opengeospatial/ogcapi-connected-systems/tree/a1f1f03b71f5f645486b23ec8b5fae1f9ba334bc/api/part3/standard) | `a1f1f03b71f5f645486b23ec8b5fae1f9ba334bc`, committed 2026-06-11 | Official work-in-progress context for the streaming gap | Pinned `part3-working-draft` tree | 2026-07-30 | Not an approved standard and not adopted by AEP Volume II |
| [OGC API - Features landing page](https://ogcapi.ogc.org/features/) | Retrieved 2026-07-30 | Official status source for indirect dependency | Part 4 status | 2026-07-30 | Mutable status page |
| [OGC API - Features repository status](https://github.com/opengeospatial/ogcapi-features/blob/9ca25f56a58ed822ea8a685a7a41afa7181aaa8b/README.md#additional-parts-of-ogc-api---features) | `9ca25f56a58ed822ea8a685a7a41afa7181aaa8b`, committed 2026-07-13 | Official pinned status evidence | Part 4 “prepared for approval” note | 2026-07-30 | Part 4 remained a draft; not normative final-standard evidence |
| [OS4CSAPI research-plan exemplar corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans) | `754411897173c2ec4debaa9bcf4ed9e0f8a9e230` | Report-structure and research-rigor exemplar only | Full 38-plan corpus and early/mid/final exemplars | 2026-07-30 | No substantive obligation evidence taken from this corpus |

### 3.3 Evidence Quality and Authority Notes

The following hierarchy controls this report:

1. The project lead’s fixed-package designation and registered hash control which NATO package this topic analyzes.
2. Within that package, STANAG 4789 provides the agreement and adoption frame.
3. AEP Volume I supplies the operational/reference objectives; AEP Volume II supplies the adopted technical package.
4. The four approved OGC standards supply the detailed normative technical requirements and conformance material.
5. Glaux governance documents control project scope, Rust, ambition, acceptance, and research handling.
6. Official schemas, landing pages, and pinned development repositories are supporting implementation or status evidence only.

Normative-strength rules:

- AEP Volumes I and II explicitly define `shall` as mandatory, `should` as recommended, and `may` as optional.
- Much of Volume I’s functional prose uses `must`, but its conventions do not define that keyword. This report preserves it as a strong operational or functional constraint and does not silently convert it into a testable AEP `shall`.
- Volume II directs implementers to the OGC standards for actual requirements, schemas, encodings, conformance classes, and behavior. AEP summaries are not substitutes for OGC requirement extraction.
- OGC `SHALL` statements are binding only within the applicable requirements class and its prerequisites. Parts 1 and 2 are modular and do not, by themselves, select a full Glaux profile.

Evidence limitations and anomalies:

- The package proves the content of the project baseline, not completed NATO ratification or promulgation.
- The controlled PDF’s embedded `Subject` metadata describes an unrelated publication. The visible content, package identifier, enclosure identities, and exact matching hash control; the stale metadata field was not used as evidence.
- Enclosure 1 says Edition 1 on its cover but Edition A in the substantive-page header.
- AEP Volume I’s sole normative NIIA/AEDP-2 reference has incomplete edition/version/date metadata.
- The official Part 2 schema tree contains an AsyncAPI `0.0.1` support artifact. It is non-normative and does not override approved Part 2’s assignment of pub/sub bindings to Part 3.
- Approved Part 2 retains apparent prepublication notes in §§16.2.2, 16.3.2, and 16.4.2 and apparent copy/paste defects in parts of its abstract test suite. This report records, but does not resolve, those detailed conformance-source issues.
- Exact quotations were avoided. Findings are paraphrased and anchored because extracted text contains ordinary line-break and hyphenation artifacts.
- No prior accepted topic report exists, so there is no prior-report conflict to reconcile.

### 3.4 Controlled-Source Citation Convention

The public report does not expose the local filesystem path. Controlled citations use:

`[source shorthand, section, PDF page, publication page]`

Source shorthands:

- `[PKG]` — `AC/224(JCGISR)D(2026)0005` covering memorandum
- `[S4789]` — Enclosure 1, STANAG 4789
- `[AEP-I]` — Enclosure 2, AEP-4789 Volume I
- `[AEP-II]` — Enclosure 3, AEP-4789 Volume II
- `[CSAPI-1]` — OGC 23-001, OGC API - Connected Systems - Part 1
- `[CSAPI-2]` — OGC 23-002, OGC API - Connected Systems - Part 2
- `[SML-3]` — OGC 23-000, SensorML 3.0
- `[SWE-3]` — OGC 24-014, SWE Common 3.0.0

The package identifier, exact SHA-256, enclosure identity, PDF page, publication page, and section together make each controlled citation reproducible for an authorized reviewer.

---

## 4. Findings by Research Question

### 4.1 CQ-1: What Direct Server Obligations Flow to Glaux Server?

**Source-backed finding:** The controlled draft establishes a framework in which authorized consumers can discover, access, exchange, stream, task, and interact with sensing systems and their information. It states that participating Allies agree to implement AEP-4789, and AEP Volume II adopts CSAPI Part 1, CSAPI Part 2, SensorML, and SWE Common as the actionable technical package. `[S4789, Interoperability Requirements and Agreement, PDF pp. 5–7, publication pp. 1–3]`; `[AEP-II, §§1.1–2.1, PDF pp. 48–54, publication pp. 1–7]`.

**Analysis:** A Glaux Server obligation is direct when the server controls the API behavior, representation, validation, persistence, policy decision enforcement, or conformance evidence. It is not direct merely because the broader NATO operational outcome depends on it.

| Capability Area | Direct Glaux Server Baseline | External Dependency | Principal Source Anchors |
|---|---|---|---|
| Standards package | Implement CSAPI Part 1, CSAPI Part 2, SensorML 3.0, and SWE Common 3.0 coherently; use the standards themselves for detailed behavior and conformance | Project profile must select breadth; OGC lifecycle may change dependencies | `[AEP-II, §§1.1–2.1, PDF pp. 48–54, publication pp. 1–7]` |
| Discovery and navigation | Expose searchable systems, observations, information, capabilities, deployments, procedures, sampling features, properties, relationships, coverage, and validity | Federated catalogue topology and metadata source authority | `[AEP-I, §§2.3, 3.1, PDF pp. 23, 26–27, publication pp. 5, 8–9]` |
| Registration and description | Create, identify, update, link, and describe resources; preserve persistent identity, organization, deployment, capability, lineage, handling, and validity context | Identifier governance and authoritative metadata producers | `[AEP-I, §3.2, PDF p. 27, publication p. 9]` |
| Access and exchange | Retrieve and share resources and observations while preserving semantics, lineage, source, time, feature of interest, freshness, and suitability; support useful selection/filtering | Product-specific formats and mission prioritization | `[AEP-I, §3.3, PDF pp. 27–28, publication pp. 9–10]` |
| Streaming and dynamic data | Expose datastreams, observations, status, system events, control streams, commands, and current/historical dynamic information; preserve time, sequence, and change context | Pub/sub transport, buffering, replay, resumption, and delivery guarantees | `[AEP-I, §3.4, PDF pp. 28–29, publication pp. 10–11]`; `[AEP-II, §1.1.2, PDF p. 50, publication p. 3]` |
| Tasking and control | Expose controllable parameters, accept authorized tasking, perform feasibility interaction, and report command lifecycle and results | Physical execution, safety interlocks, resource-owner authority, edge autonomy | `[AEP-I, §3.5, PDF p. 29, publication p. 11]`; `[AEP-II, §1.1.2, PDF p. 50, publication p. 3]`; `[CSAPI-2, §§10.13–10.14]` |
| Status and availability | Expose operational state, degradation, readiness, availability, capacity, maintenance, duty cycle, timestamps, validity, and last-known context | Source-system truth and reconciliation policy | `[AEP-I, §3.6, PDF p. 30, publication p. 12]` |
| Trust and contextual integrity | Preserve identifiers, links, provenance, source, method, quality, spatial/temporal context, freshness, validity, units, and machine interpretability | Trust anchors, source assurance, semantic governance | `[AEP-I, §§1.4, 2.2, PDF pp. 21, 23, publication pp. 3, 5]` |
| Security and authorization | Enforce applicable access and control policy consistently at discovery, read, stream, write, and command boundaries; support secure web deployment | Identity provider, PKI, accreditation, releasability policy, cross-domain gateway | `[AEP-I, §§2.4.1, 3.5, 4.2, PDF pp. 24, 29, 32, publication pp. 6, 11, 14]`; `[AEP-II, Scope and §2.2, PDF pp. 42, 54, publication pp. II, 7]` |
| DDIL-informed behavior | Preserve temporal and validity context; support asynchronous interaction, staged enrichment, last-known state, constrained/efficient exchange, and later synchronization at the server boundary | Edge operation, transport, priority rules, replication, and conflict resolution | `[AEP-I, §§2.4.2, 3.2–3.6, PDF pp. 24–30, publication pp. 6–12]` |
| Validation and conformance | Publish accurate capability/conformance declarations and validate selected standards behavior and encodings against normative OGC requirements and tests | Project acceptance profile and external OGC tooling | `[AEP-I, §4.1, PDF p. 31, publication p. 13]`; `[AEP-II, Chapter 2, PDF pp. 53–55, publication pp. 6–8]` |
| Units | Use SI units unless otherwise specified, allowing non-SI units parenthetically where operationally necessary | Detailed reconciliation with OGC/UCUM representation | `[AEP-I, Conventions, PDF p. 18, publication p. VIII]`; `[AEP-II, Conventions, PDF p. 47, publication p. VII]` |

The complete obligation register, including source strength and responsibility classification, is Appendix 12.2.

### 4.2 CQ-2: Where Is the Server Responsibility Boundary?

#### 4.2.1 Direct Server Responsibilities

Glaux Server directly owns:

- standards-facing HTTP API resources, operations, representations, links, filters, status codes, and declared conformance;
- server-side resource identity, canonical addressing, logical persistence, retrieval, query support, and lifecycle behavior without prescribing a storage or indexing architecture;
- validation and serialization of GeoJSON, SensorML JSON, SWE Common structures, and selected dynamic encodings;
- datastream, observation, control-stream, command, feasibility, command-status, command-result, and system-event API behavior;
- preservation and exposure of time, validity, freshness, provenance, quality, units, relationships, and handling metadata;
- authorization and policy enforcement at every API operation it exposes;
- asynchronous server-side lifecycle semantics and auditable state transitions required by selected later requirements; idempotency policy remains a downstream contract decision;
- machine-readable API and capability descriptions; and
- test evidence for every conformance claim.

#### 4.2.2 Server-Side Integration Contract Responsibilities

The server must define and honor contracts with:

| Counterparty | Contract the Server Must Provide | Responsibility That Remains External |
|---|---|---|
| Glaux Publisher or another ingestion mediator | Register/update resources; ingest observations, status, and events; supply identity, provenance, quality, validity, handling, and freshness metadata | Acquisition, source truth, transformation before submission, and operator workflow |
| Glaux Simulator | Submit standards-valid systems, observations, events, status, commands, and lifecycle responses for testing | Scenario generation and simulated physical behavior |
| Glaux Web App and Mobile | Discover, query, navigate, stream, submit authorized commands, and interpret status through stable standards-facing contracts | User experience, offline presentation, and device-local behavior |
| Connected-system adapter or gateway | Receive commands, report feasibility/lifecycle/results, publish observations/status/events, and reconcile delayed updates | Physical sensing or actuation, equipment safety, local autonomy, and native protocol translation |
| Identity, policy, and trust services | Authenticate identities, provide claims/policies/trust material, and support enforcement/audit decisions | Credential issuance, policy ownership, accreditation, and trust-anchor governance |
| Federation or cross-domain gateway | Exchange permitted resources, identifiers, links, and updates through defined server interfaces | Federation topology, cross-domain transfer decision, guard operation, and network routing |

#### 4.2.3 Ecosystem Responsibilities

The broader ecosystem—not Glaux Server alone—owns:

- NATO ratification, promulgation, national enactment, and implementation reporting;
- authoritative sensor operation, data stewardship, and physical actuation;
- identity providers, PKI, trust anchors, policy authorities, accreditation, and releasability decisions;
- cross-domain solutions and differently accredited environment gateways;
- federated catalogue topology, inter-server routing, and global identifier governance;
- network delivery under disruption and edge-local execution;
- mission-level prioritization and rules of engagement; and
- implementation of adjacent NATO standards by the capabilities to which they apply.

#### 4.2.4 Adjacent Interoperability Considerations

STANAGs 2103, 2586, 4545, 4559, 4586, 4607, 4609, 4676, 6523, and 7149 are listed as related or informative publications. Their listing identifies potential integration points. It does **not** automatically impose their complete requirements on Glaux Server. `[S4789, Other Related Documents, PDF p. 6, publication p. 2]`; `[AEP-II, Preface Linkages and Informative References, PDF pp. 43, 46, publication pp. III, VI]`.

#### 4.2.5 Explicitly Out of Glaux Server Scope

- full Glaux Web App, Mobile, Publisher, or Simulator implementation behavior;
- operator training, tactics, techniques, and procedures;
- a full NAF-conformant architecture product set;
- redesign or physical reintegration of native sensing assets;
- NATO document-handling administration and e-Reporting;
- automatic implementation of every adjacent NATO publication;
- modality-specific profiles not yet assigned by an approved artifact; and
- speculative future AEP volume, SRD, profile, dictionary, or registry content.

### 4.3 CQ-3: How Do the AEP Functions Translate to Server Obligation Categories?

| AEP Function | Direct Server Responsibility | Server Contract | Ecosystem / Out-of-Scope Boundary | Adopted Technical Basis |
|---|---|---|---|---|
| Discovery | Searchable collections and resources; identifiers, links, filters, capability and validity metadata | Receive accurate metadata and expose federation/gateway hooks | Enterprise catalogue topology and discovery policy | CSAPI Part 1; SensorML |
| Registration and description | Resource creation/update; persistent identity; system, deployment, procedure, sampling, property, capability, lineage, and constraint descriptions | Identifier assignment, metadata ownership, delayed update reconciliation | Native system inventory and organizational governance | CSAPI Part 1; SensorML; SWE Common |
| Access and exchange | Resource/observation retrieval, filtering, content negotiation, semantic and lineage preservation | Product adapters, staged delivery, mission-priority input | Network transport and product-specific NATO workflows | CSAPI Parts 1 and 2; SWE Common; SensorML |
| Streaming and dynamic data | Datastream/control-stream model; live and historical data; time, sequence, event, and status semantics | Replaceable pub/sub/stream adapter, replay and resumption contract | Delivery infrastructure and edge buffering | CSAPI Part 2; SWE Common |
| Tasking and control | Control streams, commands, feasibility, status history, result exposure, authorization, audit | Adapter delivery to the asset and asynchronous lifecycle feedback | Physical actuation, safety, local autonomy, owner authority | CSAPI Part 2; Part 1 property definitions; SensorML; SWE Common |
| Status and availability | Status datastreams, system events, temporal validity, last-known representation | Source-state ingestion and reconciliation | Operational truth, maintenance system, mission availability decision | CSAPI Part 2; Part 1/SensorML context; SWE Common |
| Provenance, validity, freshness, and trust | Preserve and expose source, method, quality, time, validity, lineage, and handling context | Obtain signed/authoritative metadata and policy decisions | Source reputation, trust framework, semantic governance | SensorML; SWE Common; CSAPI links and temporal properties |

**Interpretation:** The same operational workflow often crosses all three responsibility layers. For example, the server must expose and authorize a command, but an adapter executes it and a resource owner defines whether it is permitted. Treating the whole workflow as either “all server” or “not server” would be incorrect.

### 4.4 CQ-4: Security, Authorization, Trust, Federation, Coalition, and DDIL

#### 4.4.1 Security and Authorization

**Source-backed finding:** Volume I states the functional objective and constraint that the framework work for authorized users, AI, applications, and services across organizational and security boundaries, and says policy-driven gateways may be needed. It also recognizes that the adopted standards do not define every required security or trust capability. `[AEP-I, Preface; §§2.4.1, 3.5, 4.2, PDF pp. 12, 24, 29, 32, publication pp. II, 6, 11, 14]`.

The approved CSAPI standards do not close this gap:

- Part 1 discusses RBAC, HTTP authentication, API keys, OAuth 2.0, OpenID Connect, HTTPS, encryption at rest, PKI, and signatures, but mandates no particular authentication or authorization method. `[CSAPI-1, §7.10]`.
- Part 2 inherits Part 1’s security considerations. `[CSAPI-2, Security Considerations]`.
- SWE Common states that no security considerations were made for that standard. `[SWE-3, Security Considerations]`.

**Analysis:** Glaux Server is an enforcement point, not the identity, accreditation, or cross-domain authority. Later security research must select the mechanisms and policy granularity, but the server architecture cannot treat security as an optional perimeter wrapper. Enforcement must apply consistently to discovery, resource fields, observations, streams, writes, commands, feasibility, status, and audit.

AI is an ordinary authorized consumer or actor under this baseline; the AEP provides no AI-specific bypass or privilege.

#### 4.4.2 Federation and Coalition Use

CSAPI stable identifiers, canonical URLs, distributed resource classes, and hyperlinks are compatible with federation. They do not define:

- federated query planning or catalogue aggregation;
- a global UID allocation authority;
- link integrity across security domains;
- replicated-cache consistency or conflict resolution;
- cross-domain release policy; or
- inter-server trust and routing.

These are server integration and ecosystem design obligations to be defined later, not evidence that the core package already solves federation.

#### 4.4.3 DDIL

Volume I treats denied, disrupted, intermittent, and limited communications as normal. Its objectives include useful local operation while disconnected, constrained exchange, later synchronization, mission-critical context travelling with payloads, progressive enrichment, change-aware updates, validity/freshness assessment, staged delivery, prioritization, asynchronous tasking, delayed confirmation, last-known state, and possibly compact representations. `[AEP-I, §§2.2, 2.4.2, 3.2–3.6, PDF pp. 23–30, publication pp. 5–12]`.

**Analysis — direct server implications:**

- model time, validity, freshness, source, and last-known state explicitly;
- define explicit contracts for delayed updates and for cases in which arrival order differs from source or event order;
- support asynchronous command and status lifecycles;
- avoid assuming every interaction completes while a producer remains connected;
- enable compact and selective representations where the standards permit; and
- make synchronization and conflict outcomes observable and testable once later research defines them.

Limits:

- the AEP does not require one centralized server to remain reachable during disconnection;
- the adopted package does not define offline queues, delta synchronization, replay, delivery guarantees, conflict algorithms, cache expiry, or mission-priority policy;
- SWE Common’s efficient encodings help but do not constitute a DDIL synchronization protocol; and
- unadopted CSAPI Part 3 and potential later encodings cannot be presented as current AEP obligations.

### 4.5 CQ-5: Traceability Structure for Downstream Use

Every downstream requirement record should contain:

| Field | Purpose |
|---|---|
| Baseline ID | Stable identifier from this report, such as `STREAM-02` |
| Package identity and hash | Proves which project-controlled NATO baseline was used |
| Enclosure identity | Distinguishes STANAG, AEP Volume I, and AEP Volume II |
| PDF page and publication page/section | Allows an authorized reviewer to reproduce the source |
| Source keyword/strength | Records `shall`, `should`, `may`, undefined `must`, adoption statement, or descriptive prose |
| Paraphrased obligation | States the decision-usable requirement without losing source meaning |
| Authority and lifecycle status | Prevents a draft, informative source, or recommendation from becoming a false mandate |
| Responsibility class | Server, server contract, ecosystem, adjacent, or out of scope |
| Capability area | Discovery, description, access, streaming, tasking, status, security, DDIL, and so forth |
| Adopted-standard mapping | Identifies CSAPI Part 1/2, SensorML, SWE Common, or an uncovered gap |
| OGC requirements/conformance class and URI | Added by later normative extraction topics |
| Accepted project interpretation | Records any Glaux profile or boundary decision |
| Rust component and interface | Added during architecture/implementation planning |
| Verification reference | Automated test, conformance test, or acceptance evidence |
| Owner and disposition | Tracks implementation, deferral, exception, and acceptance |

The AEP anchor and OGC requirement must remain separate. The AEP explains **why a capability is in Glaux scope**; the OGC requirement defines **the technical rule to implement and test**.

---

## 5. Decision Analysis and Standards-Package Alignment

### 5.1 Boundary Options

| Option | Benefits | Costs/Risks | Standards / Compatibility Impact | Recommendation |
|---|---|---|---|---|
| A. Minimal modular OGC implementation | Fastest path to a nominal conformance claim | Can implement one resource type and one encoding while missing the AEP mission and Glaux goal | Formally narrow but operationally inadequate | Reject |
| B. Treat every NATO operational outcome as an internal server feature | Appears comprehensive | Absorbs identity, networks, sensors, gateways, policy authorities, and other products; invites invented behavior | Conflates standards with ecosystem architecture | Reject |
| C. Layered full-scope server with explicit profile and contracts | Preserves ambition, conformance, boundaries, and replaceable integrations | Requires disciplined profiles, adapters, and traceability | Best alignment with STANAG/AEP, OGC modularity, and the Rust reference-server goal | **Adopt** |
| D. Wait for every draft and profile gap to close | Avoids provisional choices | Indefinite delay; ignores the usable approved core and fixed project baseline | No implementation progress despite mature adopted standards | Reject |

### 5.2 OGC Standards-Package Alignment and Gaps

| AEP Expectation | Adopted Technical Realization | Alignment | Material Gap / Later Treatment |
|---|---|---|---|
| Discovery and navigation | CSAPI Part 1 systems, deployments, procedures, sampling features, properties, links, collections, and filters; SensorML descriptions | Strong for one API and linked resources | Federated search, aggregation, replication, and cross-node identity are not defined |
| Registration, update, identity, and description | CSAPI Part 1 UIDs, canonical URLs, create/replace/update classes, GeoJSON and SensorML; SensorML rich metadata | Strong model; qualified transaction basis | Write/update classes depend on draft OGC API - Features Part 4; identifier governance and disconnected reconciliation remain |
| Observation access and exchange | CSAPI Part 2 datastreams/observations and temporal access; SWE Common schemas and encodings; links to Part 1 resources | Strong | Variable-level subsetting, resolution selection, large products, and modality-specific profiles may require adjacent standards |
| Continuous and event-driven data | CSAPI Part 2 live/historical resources, system events, and SWE Common encodings | Partial | Approved Part 2 assigns pub/sub bindings to Part 3; Part 3 is a working draft and not AEP-adopted |
| Tasking and command lifecycle | CSAPI Part 2 control streams, commands, feasibility, status, and results; Part 1/SensorML capability description | Strong API model | Policy adjudication, delivery adapter, safety, owner authority, and physical execution remain external |
| Health, availability, and last-known state | Part 2 status datastreams, `live`, events, command status; Part 1 validity/deployment context | Partial | No common NATO readiness/capacity/duty-cycle vocabulary, reconciliation rule, or last-known-state policy; predefined event URIs include `x-OGC/TBD` |
| Provenance, quality, freshness, and trust | SensorML lineage, quality, configuration, and validity; SWE Common units/quality/constraints; CSAPI links and time | Good structural support | No trust decision, signature policy, source reputation, freshness threshold, or semantic registry governance |
| Security and coalition access | HTTPS-oriented deployment and CSAPI security integration guidance | Major profile gap | No mandatory authentication, authorization, federation, releasability, cross-domain, field-level access, credential, or audit profile |
| DDIL behavior | Temporal context, historical resources, selective JSON, and SWE Common text/binary efficiency | Partial efficiency support | No queueing, prioritization, resumption, delta sync, conflict resolution, staged enrichment, or offline replication protocol |
| Verifiable interoperability | Normative requirements and abstract tests in all four approved OGC standards; official schemas | Strong foundation | Modular conformance does not prove AEP function coverage; Glaux must publish a profile/manifest |
| SI units | SensorML and SWE Common can represent units | Representation available | Glaux profile and validation must enforce the AEP unit policy and reconcile it with OGC/UCUM rules |

### 5.3 Standards-Status Conclusions

1. The four Volume II references exactly match the current approved OGC document identifiers and 16 July 2025 publication date. Differences such as `1.0` versus `1.0.0` are presentation differences, not edition conflicts.
2. The normative OGC standards control technical conformance. Official schemas support implementation but do not override the text.
3. The current CSAPI development repository and working drafts are informative only unless a later project profile explicitly adopts and labels a provisional capability.
4. MQTT and WebSocket references in Part 2 do not make pub/sub bindings part of the approved Part 2 conformance classes or the AEP Volume II package.
5. The Part 4 transaction dependency must be pinned and tracked as provisional until OGC publishes an approved version.

---

## 6. Key Recommendations

1. **Adopt the boundary statement in Section 1 as the reusable Glaux Server scope rule.**
   - Rationale: It preserves the full reference-server mission while preventing identity systems, sensors, gateways, networks, and other Glaux products from becoming hidden server internals.
   - Preconditions: Plan-owner acceptance of this report.
   - Priority: High

2. **Create a versioned Glaux AEP/CSAPI conformance profile after the dedicated standards-extraction topics are accepted.**
   - Rationale: OGC conformance is modular; “supports CSAPI” is too weak to prove the full AEP/Glaux capability.
   - Required content: every selected requirements class, encoding, prerequisite, requirement URI, implementation disposition, and test.
   - Preconditions: `IDR-SRV-003`, `IDR-SRV-006`, `IDR-SRV-007`, and `IDR-SRV-008`.
   - Priority: High

3. **Use two-layer traceability: AEP operational anchor plus OGC normative requirement.**
   - Rationale: This prevents operational objectives from being mistaken for exact protocol rules and prevents a technically conformant subset from being mistaken for mission coverage.
   - Preconditions: None after report acceptance.
   - Priority: High

4. **Make external dependencies explicit ports/contracts in the eventual Rust architecture.**
   - Rationale: Publisher ingestion, connected-system command delivery, identity/policy, federation, cross-domain mediation, and DDIL synchronization must be replaceable and testable without becoming the canonical CSAPI model.
   - Preconditions: Later architecture topics.
   - Priority: High

5. **Treat security, federation, DDIL, semantic governance, and status vocabulary as separate project profiles constrained by—not invented inside—the OGC core.**
   - Rationale: The AEP requires the outcomes but the adopted package does not select the mechanisms.
   - Preconditions: Dedicated later research and plan-owner decisions.
   - Priority: High

6. **Pin and isolate provisional standards dependencies.**
   - Rationale: OGC API - Features Part 4 remained a draft; CSAPI Part 3 remained unapproved and outside the AEP package.
   - Direction: Pin the exact Part 4 revision used by later transaction research; place any Part 3-based pub/sub support behind a replaceable adapter and label it project-profiled or experimental, not an AEP obligation or an approved OGC Part 3 conformance claim.
   - Preconditions: Later transaction and streaming research.
   - Priority: High

7. **Model temporal validity, freshness, provenance, quality, identity, and source authority as first-class data—not incidental annotations.**
   - Rationale: These concepts recur across discovery, exchange, status, tasking, coalition use, and DDIL.
   - Preconditions: Resource and persistence research.
   - Priority: High

8. **Carry source-status limitations with every downstream citation.**
   - Rationale: The fixed package is valid as the project baseline but does not prove completed NATO promulgation; its edition and NIIA-reference anomalies must not disappear through repeated paraphrase.
   - Preconditions: None.
   - Priority: Medium

9. **Do not derive implementation requirements from the adjacent-STANAG list until the assigned boundary review examines the controlling publications.**
   - Rationale: The current package calls those linkages complementary or informative.
   - Preconditions: `IDR-SRV-005`.
   - Priority: Medium

---

## 7. Implementation Implications and Estimates

### 7.1 Rust Server Implications

This report does not select Rust libraries or produce a software architecture. It does establish constraints that later planning must preserve:

- The Rust implementation should separate the canonical CSAPI/SensorML/SWE domain and HTTP behavior from adapters for publishers, devices, identity/policy, federation, and streaming transports.
- Conformance capabilities should be explicit, versioned data that can drive `/conformance`, documentation, feature configuration, and tests; they should not be scattered compile-time assumptions.
- Identity, links, provenance, relevant times, validity periods, quality, units, and source authority need durable domain representation and query support.
- Dynamic data and tasking require stateful lifecycle models, not stateless route wrappers.
- Authorization decisions and audit context must cross every read, stream, write, and command path.
- DDIL-related contracts will need explicit idempotency, delayed update, ordering, replay, and reconciliation semantics once later research defines them.
- The verification system must trace each selected OGC requirement URI and each project-profile rule to executable evidence.

### 7.2 Relative Complexity

Calendar or person-hour implementation estimates would be speculative before the dedicated requirement, architecture, persistence, security, and verification topics complete. Relative complexity is defensible:

| Work Item | Relative Complexity | Quantitative Estimate | Assumptions / Estimation Gate |
|---|---|---|---|
| Glaux conformance-profile and traceability registry | High | Deferred | Estimate after `IDR-SRV-003`, `006`, `007`, `008`, and `051` |
| Part 1 resource, discovery, description, and lifecycle layer | High | Deferred | Estimate after Part 1 extraction and resource/persistence topics |
| Part 2 observations, streaming, status, events, and tasking | Very High | Deferred | Estimate after Part 2, dynamic-data, streaming, and command topics |
| SensorML and SWE Common validation/encoding | High | Deferred | Estimate after encoding and schema-validation topics |
| Security, authorization, federation, and audit integration | Very High | Deferred | Estimate after security and policy topics |
| DDIL synchronization and last-known-state behavior | Very High / uncertain | Deferred | Estimate after DDIL and edge/federation topics |
| Conformance and interoperability harness | High | Deferred | Estimate after verification and traceability topics |

The absence of invented hours is intentional. Later reports should narrow these estimates with actual requirement counts, chosen profiles, performance targets, and architecture decisions.

---

## 8. Risks, Constraints, and Open Questions

### 8.1 Risk Register

| Risk / Constraint | Impact | Current Treatment | Owner / Handoff |
|---|---|---|---|
| Fixed NATO package is pre-promulgation and has placeholder fields | Overstated claims of NATO status | Treat as project-controlling baseline; qualify lifecycle status | Project lead / all downstream topics |
| Enclosure 1 Edition 1 / Edition A inconsistency | Citation ambiguity | Cite enclosure title plus both exact page and package hash | Project lead; carry until authoritative correction |
| AEP-I normative NIIA reference lacks edition/version/date | Cannot derive detailed NIIA obligations | Do not infer content; treat linkage as unresolved | `IDR-SRV-005` or later authorized source review |
| AEP `must` is undefined by its keyword convention | Functional prose could be mislabeled mandatory | Preserve as strong functional constraint, not formal `shall` | All downstream topics |
| “Coherent whole” versus modular OGC conformance | Nominal conformance could omit required Glaux capability | Create explicit Glaux profile | `IDR-SRV-003`, `006`–`008` |
| OGC API - Features Part 4 remains draft | Transaction behavior depends on a moving source | Pin revision, isolate dependency, track approval | Transaction/resource topics |
| CSAPI Part 3 is unapproved and not AEP-adopted | Streaming transport objective is not fully standardized | Replaceable adapter; no false conformance claim | Streaming topics |
| Official Part 2 schema tree contains an AsyncAPI `0.0.1` artifact although approved Part 2 assigns pub/sub bindings to Part 3 | Supporting artifact could be mistaken for an approved binding | Treat as non-normative; preserve exact provenance; do not claim approved pub/sub conformance from it | `IDR-SRV-007`, `012`, `035` |
| Approved Part 2 retains apparent prepublication notes and abstract-test copy defects | Blind use could produce incorrect requirements or tests | Verify normative text, requirement URIs, schemas, and tests independently during detailed extraction | `IDR-SRV-007`, `012`, `050` |
| No selected security/federation/releasability profile | Coalition deployment could be insecure or non-interoperable | Dedicated threat, policy, zero-trust, and audit research | `IDR-SRV-039`–`041` |
| No DDIL replication, priority, or conflict protocol | “DDIL support” could become vague marketing | Define observable semantics and tests later | `IDR-SRV-042`–`043` |
| Incomplete readiness/availability vocabulary; provisional `x-OGC/TBD` event URIs | Status may be semantically inconsistent | Establish controlled project/NATO profile without rewriting CSAPI | Status, terminology, semantics topics |
| SI requirement versus broader OGC/UCUM representations | Validation or interoperability conflict | Reconcile in encoding/profile research | SensorML/SWE and validation topics |
| Adjacent NATO publications not analyzed here | Hidden integration requirements may emerge | Keep as adjacent; execute only assigned later review | `IDR-SRV-005` |
| Controlled source is not publicly linkable | External reviewers cannot reproduce NATO citations unaided | Preserve package ID, hash, enclosure, PDF page, publication page, and section | Project lead controls authorized access |
| Stale unrelated embedded PDF `Subject` metadata | Automated metadata tooling could misidentify source | Do not use embedded subject; validate visible identity and hash | Evidence handling |

### 8.2 Questions Requiring Plan-Owner Acceptance Now

The project lead’s acceptance of this report should confirm:

1. the fixed package remains the project-controlling baseline despite its visible pre-promulgation status;
2. the direct-server / server-contract / ecosystem boundary in Section 4.2 is suitable for downstream planning;
3. the dual AEP-to-OGC traceability structure in Section 4.5 is required downstream;
4. the listed limitations and provisional dependencies are accurate enough to carry forward; and
5. no adjacent NATO publication or future AEP/SRD content should be treated as a Glaux Server requirement without its assigned research and an explicit project decision.

### 8.3 Questions Deliberately Deferred to Later Topics

- Which exact OGC requirements and conformance classes constitute the Glaux full-scope profile?
- Which pinned OGC API - Features Part 4 revision governs initial transactional behavior?
- Will Glaux implement a Part 3 working-draft pub/sub adapter, another transport profile, or both, and how will the capability be labeled?
- Which authentication, authorization, federation, releasability, cross-domain, and audit profiles will Glaux support?
- What are the exact DDIL queue, priority, replay, synchronization, conflict, cache, and staged-enrichment semantics?
- Which controlled vocabulary defines health, readiness, capacity, duty cycle, availability, and last-known state?
- How are persistent identifiers allocated and reconciled across servers and security domains?
- How should SI policy be enforced while remaining correct for OGC/UCUM representations?

These are not omissions from IDR-SRV-001. The report identifies the obligations and assigns the unresolved mechanism or profile decision to its proper downstream research.

---

## 9. Validation Against the Research Plan

### 9.1 Success Criteria

| Topic Plan Success Criterion | Validation Status | Evidence |
|---|---|---|
| Fixed package hash-verified and all three enclosures reviewed | Met | Section 3.1; package boundary in Appendix 12.3 |
| Sources list title, version/date, URL/path handling, access date, and authority | Met | Sections 3.1–3.2 |
| Direct server obligations extracted with source anchors | Met | Sections 4.1 and 12.2 |
| Server-side contracts distinguished from ecosystem obligations | Met | Section 4.2 |
| Out-of-scope concerns explicitly identified | Met | Section 4.2.5 |
| All named functional, security/trust, and DDIL areas assessed | Met | Sections 4.3–4.4 and 5.2 |
| Downstream dependencies and handoffs identified | Met | Section 10.2 |
| Unresolved questions and interpretation risks listed | Met | Section 8 |
| Recommendations are decision-usable and server-bounded | Met | Section 6 |
| References are explicit and reproducible | Met | Sections 3.4, 11, and 12.2 |

### 9.2 Deliverable Requirements

| Required Content | Status | Report Location |
|---|---|---|
| 1. Executive summary | Met | Section 1 |
| 2. Scope and plan alignment | Met | Section 2 |
| 3. Evidence base and authority classification | Met | Section 3 |
| 4. STANAG/AEP obligation extraction findings | Met | Sections 4.1 and 12.2 |
| 5. Server responsibility classification | Met | Section 4.2 |
| 6. Standards-package alignment notes | Met | Section 5.2 |
| 7. Downstream topic handoff matrix | Met | Section 10.2 |
| 8. Recommendations | Met | Section 6 |
| 9. Implementation implications | Met | Section 7 |
| 10. Risks, constraints, and open questions | Met | Section 8 |
| 11. Validation against success criteria | Met | Section 9.1 |
| 12. References | Met | Section 11 |

### 9.3 Research Question and Method Validation

- All 5 core and 27 detailed questions are mapped in Section 2.2 and Appendix 12.1.
- All five methodology phases are validated in Section 2.3.
- Source authority, normative strength, inference, and project recommendations are labeled separately.
- No inaccessible adjacent standard was inferred.
- No other IDR research topic was executed.
- The report is complete as a deliverable but remains **In Review** until the plan owner accepts it.

---

## 10. Next Steps and Handoff

### 10.1 Immediate Next Step

1. **Review and accept or return IDR-SRV-001.** Owner: Glaux Project Lead. Due: TBD by project lead.
2. If accepted, complete the report’s `Accepted By` and `Acceptance Date` fields, mark the topic plan complete, and increment accepted-report coverage. Owner: Glaux Project Lead / research executor.
3. Only after acceptance, proceed to the next scheduled topic under the controlling overall plan. No subsequent topic is begun by this report.

### 10.2 Downstream Handoff Matrix

| Topic(s) | Required Handoff from IDR-SRV-001 |
|---|---|
| `IDR-SRV-002` | Preserve the six Volume I functional areas, cross-environment constraints, external-interface boundary, and functional-versus-formal-normative distinction |
| `IDR-SRV-003` | Preserve the exact four-standard package, versions, complementary roles, coherent-package interpretation, modular-conformance ambiguity, and provisional dependencies |
| `IDR-SRV-004` | Crosswalk `sensing resource`, `connected system`, `system`, `feature resource`, `datastream`, `control stream`, `command`, `command status`, `system event`, `deployment`, `observation`, `feature of interest`, `sampling feature`, `sensor-derived information`, `tasking`, `availability`, `freshness`, `validity`, and DDIL |
| `IDR-SRV-005` | Review adjacent NATO standards without treating the current informative lists as implementation mandates |
| `IDR-SRV-006`–`008` | Extract actual OGC normative requirements and conformance classes; define how the full-scope goal resolves modular optionality; preserve the recorded Part 2 publication and test-source anomalies for detailed adjudication |
| `IDR-SRV-015`–`024` | Carry resource identity, links, temporal validity, provenance, status/events, SensorML, SWE Common, schema validation, SI/units, and semantic-binding obligations |
| `IDR-SRV-025`–`030` | Preserve logical persistence, geospatial/time-series query, identity, lifecycle, delayed-update, consistency, idempotency, concurrency, retention, and deletion needs without treating an internal storage or indexing architecture as standards-mandated |
| `IDR-SRV-031`–`033` | Define Publisher, Simulator, and ingestion contracts for source authority, metadata, observations, status, events, delayed updates, and reconciliation |
| `IDR-SRV-034`–`038` | Define dynamic data, streaming, command lifecycle, feasibility, asynchronous tasking, authorization, safety, and audit behavior |
| `IDR-SRV-012`, `035` | Treat the Part 2 AsyncAPI `0.0.1` file as a non-normative support artifact; choose, pin, label, and test any actual streaming/pub-sub binding separately |
| `IDR-SRV-039`–`041` | Select authentication, authorization, policy, releasability, cross-boundary, zero-trust, and audit mechanisms absent from the package |
| `IDR-SRV-042`–`043` | Convert DDIL objectives into explicit server contracts, synchronization boundaries, conflict rules, freshness, last-known state, and staged enrichment |
| `IDR-SRV-050`–`051`, `054`–`055` | Trace every accepted baseline item into conformance, load/streaming, security, and command/control tests |

---

## 11. References

### 11.1 Controlled NATO Package

- North Atlantic Treaty Organization, `AC/224(JCGISR)D(2026)0005`, *NATO Standard - STANAG 4789*, 27 April 2026. Project-controlled local source. SHA-256: `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`. Accessed 2026-07-30.
  - Cover memorandum, PDF pp. 1–2.
  - Enclosure 1: *STANAG 4789, Sensor Integration Standard for NATO JISR Operations*, PDF pp. 3–7.
  - Enclosure 2: *AEP-4789 Volume I, Sensor Integration Standard for NATO JISR Operations - Reference View*, Edition A, Version 1, PDF pp. 8–37.
  - Enclosure 3: *AEP-4789 Volume II, Sensor Integration Standard for NATO JISR Operations - Core APIs and Encodings*, Edition A, Version 1, PDF pp. 38–59.

### 11.2 Adopted OGC Standards

- Open Geospatial Consortium, [OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html), approved and published 2025-07-16. Accessed 2026-07-30.
- Open Geospatial Consortium, [OGC 23-002, *OGC API - Connected Systems - Part 2: Dynamic Data*, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html), approved and published 2025-07-16. Accessed 2026-07-30.
- Open Geospatial Consortium, [OGC 23-000, *OGC SensorML Encoding Standard*, Version 3.0](https://docs.ogc.org/is/23-000/23-000.html), approved and published 2025-07-16. Accessed 2026-07-30.
- Open Geospatial Consortium, [OGC 24-014, *OGC SWE Common Data Model Encoding Standard*, Version 3.0.0](https://docs.ogc.org/is/24-014/24-014.html), approved and published 2025-07-16. Accessed 2026-07-30.

### 11.3 Official OGC Supporting Sources

- [OGC API - Connected Systems landing page](https://ogcapi.ogc.org/connectedsystems/). Accessed 2026-07-30.
- [OGC API - Connected Systems official schema repository](https://schemas.opengis.net/ogcapi/connected-systems/). Accessed 2026-07-30.
- [SensorML 3.0 JSON schemas](https://schemas.opengis.net/sensorML/3.0/json/). Accessed 2026-07-30.
- [SWE Common 3.0 JSON schemas](https://schemas.opengis.net/sweCommon/3.0/json/). Accessed 2026-07-30.
- [OGC Connected Systems development repository at commit `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f). Accessed 2026-07-30.
- [CSAPI Part 3 working draft at commit `a1f1f03b71f5f645486b23ec8b5fae1f9ba334bc`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/a1f1f03b71f5f645486b23ec8b5fae1f9ba334bc/api/part3/standard). Accessed 2026-07-30. Informative only.
- [OGC API - Features official landing page](https://ogcapi.ogc.org/features/). Accessed 2026-07-30.
- [OGC API - Features repository at commit `9ca25f56a58ed822ea8a685a7a41afa7181aaa8b`](https://github.com/opengeospatial/ogcapi-features/tree/9ca25f56a58ed822ea8a685a7a41afa7181aaa8b). Accessed 2026-07-30.

### 11.4 Project Governance and Reporting Sources

- [IDR-SRV-001 Research Plan](../IDR%20Plans/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline.md)
- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research Planning Approach](../../../../../Governance/research-planning-approach.md)
- [Research Plan Template](../../../../../Governance/research-plan-template.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
- [Overall Research Report Template](../../../../../Governance/overall-research-report-template.md)
- [OS4CSAPI research-plan exemplar corpus at commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans)

---

## 12. Appendices

### 12.1 Complete Research Question Coverage Matrix

| ID | Detailed Plan Question (Short Form) | Status | Report Evidence |
|---|---|---|---|
| CQ-1 | Direct obligations imposed or implied for Glaux Server | Complete | Sections 4.1, 12.2 |
| CQ-2 | Function classification across responsibility types | Complete | Section 4.2 |
| CQ-3 | Functional concepts translated to server categories | Complete | Section 4.3 |
| CQ-4 | Security, trust, federation, coalition, and DDIL implications | Complete; gaps explicit | Section 4.4 |
| CQ-5 | Downstream traceability structure | Complete | Section 4.5 |
| SA-1 | Role of STANAG 4789 | Complete | Sections 3.3, 4.1 |
| SA-2 | Role of AEP Volume I | Complete | Sections 3.3, 4.1–4.4 |
| SA-3 | Role of AEP Volume II | Complete | Sections 3.3, 5.2 |
| SA-4 | Normative, informative, and guidance language | Complete | Section 3.3 |
| SA-5 | Authority of obligation sources | Complete | Sections 3.1–3.3 |
| SB-1 | Obligations clearly belonging to Glaux Server | Complete | Section 4.2.1 |
| SB-2 | Ecosystem obligations requiring server support | Complete | Sections 4.2.2–4.2.3 |
| SB-3 | Responsibilities of Publisher, Simulator, Web App, or Mobile | Complete | Section 4.2.2 |
| SB-4 | Obligations outside server scope unless later assigned | Complete | Sections 4.2.4–4.2.5 |
| SB-5 | Reusable downstream boundary language | Complete | Section 1 and Recommendation 1 in Section 6 |
| FM-1 | Discovery and navigation | Complete | Sections 4.1, 4.3 |
| FM-2 | Registration and description | Complete | Sections 4.1, 4.3 |
| FM-3 | Access and exchange | Complete | Sections 4.1, 4.3 |
| FM-4 | Streaming and dynamic data | Complete; transport gap explicit | Sections 4.1, 4.3, 5.2 |
| FM-5 | Tasking and control | Complete | Sections 4.1, 4.3 |
| FM-6 | Status and availability | Complete; vocabulary gap explicit | Sections 4.1, 4.3, 5.2 |
| FM-7 | Provenance, validity, freshness, and trust | Complete | Sections 4.1, 4.3–4.4 |
| OE-1 | NATO, national, coalition, federated, tactical, and DDIL contexts | Complete | Section 4.4 |
| OE-2 | Cross-organizational and differently accredited environments | Complete; policy profile unresolved by source | Section 4.4 |
| OE-3 | Degraded, intermittent, or constrained connectivity | Complete; mechanisms unresolved by source | Section 4.4.3 |
| OE-4 | Last-known state, delayed update, enrichment, sync, and freshness | Complete; mechanisms assigned downstream | Sections 4.4.3, 8.3 |
| OE-5 | Later architecture and verification obligations | Complete | Sections 7 and 10.2 |
| TD-1 | Traceability anchors | Complete | Sections 3.4, 4.5 |
| TD-2 | Terms for the terminology crosswalk | Complete | Section 10.2 |
| TD-3 | Findings handed to CSAPI extraction | Complete | Sections 5.2, 10.2 |
| TD-4 | Findings handed to security, DDIL, tasking, status, and verification | Complete | Section 10.2 |
| TD-5 | Open questions carried forward | Complete | Section 8.3 |

### 12.2 Server Obligation Register

Responsibility classes:

- **Server** — behavior or representation directly controlled by Glaux Server.
- **Contract** — server-facing behavior whose complete outcome depends on another component.
- **Ecosystem** — organizational, operational, infrastructure, or policy responsibility the server cannot satisfy alone.
- **Adjacent** — interoperability consideration requiring later source review.
- **Out of scope** — no current Glaux Server implementation obligation.

The IDs below are report-local traceability identifiers. They are not official NATO or OGC requirement identifiers; downstream normative extraction must retain the official OGC requirement URIs separately.

| ID | Source Anchor | Strength | Normalized Obligation / Finding | Classification |
|---|---|---|---|---|
| GOV-01 | `[S4789, Interoperability Requirements, PDF p. 5, publication p. 1]` | Agreement aim | Enable authorized discovery, access, exchange, streaming, tasking, and interaction across diverse environments | Server + Contract + Ecosystem |
| GOV-02 | `[S4789, Agreement and Implementation, PDF pp. 6–7, publication pp. 2–3]` | Agreement-level adoption in controlled draft | Implement AEP-4789 and its included/referenced specifications in applicable deployed capabilities | Ecosystem governance; server contributes |
| GOV-03 | `[AEP-I, Preface Linkages, PDF p. 13, publication p. III]` | `shall` | Interpret, apply, and maintain the AEP within NIIA and its interoperability objectives | Ecosystem / project architecture |
| GOV-04 | `[AEP-I, Conventions, PDF p. 18, publication p. VIII]`; `[AEP-II, Conventions, PDF p. 47, publication p. VII]` | `shall` | Express measurements and units in SI unless otherwise specified; non-SI may appear parenthetically when operationally necessary | Server serialization/validation + profile |
| BND-01 | `[AEP-I, §2.2, PDF pp. 22–23, publication pp. 4–5]` | Functional scope | Focus on external information/control interfaces; allow native or proximate mediated exposure without redesigning the sensing asset | Server + Contract; native internals out of scope |
| PKG-01 | `[AEP-II, §§1.1, 2.1, PDF pp. 48–54, publication pp. 1–7]` | Adoption plus `should`/imperative guidance | Use CSAPI P1, CSAPI P2, SensorML, and SWE Common coherently according to complementary roles | Server |
| PKG-02 | `[AEP-II, Table 1-1 and Chapter 2, PDF pp. 49, 53, publication pp. 2, 6]` | Direct implementation pointer | Use the referenced OGC standards for requirements, schemas, encodings, conformance classes, and behavior | Server |
| PKG-03 | `[AEP-II, §1.1.1, PDF pp. 49–50, publication pp. 2–3]` | Adopted technical role | Use CSAPI Part 1 for systems, deployments, procedures, sampling features, properties, structure, discovery, identity, and access | Server |
| PKG-04 | `[AEP-II, §1.1.2, PDF p. 50, publication p. 3]`; `[CSAPI-2, §10, especially §§10.13–10.14]` | Adopted technical role plus normative OGC detail | Use CSAPI Part 2 for datastreams, observations, status, control streams, commands, feasibility, command status, results, and events | Server + Contract for execution |
| PKG-05 | `[AEP-II, §1.1.3, PDF pp. 50–51, publication pp. 3–4]` | Adopted technical role | Use SensorML for rich machine-readable system, process, configuration, deployment, capability, quality, interface, and lineage description | Server + metadata-source Contract |
| PKG-06 | `[AEP-II, §1.1.4, PDF pp. 51–52, publication pp. 4–5]` | Adopted technical role | Use SWE Common for self-describing structures and encodings for observations, status, commands, tasking, static/dynamic data, and streams | Server |
| NET-01 | `[AEP-II, Scope and §2.2, PDF pp. 42, 54, publication pp. II, 7]` | Applicability statement | Support secure web-suitable exchange, primarily HTTPS over IP; recognize that constrained environments may require additional adaptation, mediation, or guidance | Server deployment + Contract / Ecosystem |
| DISC-01 | `[AEP-I, §§2.3, 3.1, PDF pp. 23, 26–27, publication pp. 5, 8–9]` | Functional objective | Let authorized consumers discover information, observations, data series, sensing assets, and relevant relationships | Server; federation is Contract/Ecosystem |
| DISC-02 | `[AEP-I, §3.1, PDF pp. 26–27, publication pp. 8–9]` | Functional `must` | Expose sufficient format, unit, parameter, coverage, capability, limitation, organization, deployment, identifier, and validity metadata for meaningful discovery | Server + metadata-source Contract |
| REG-01 | `[AEP-I, §§2.3, 3.2, PDF pp. 23, 27, publication pp. 5, 9]` | Functional objective | Establish persistent identity, linkage, human-readable designation, and responsible organization | Server + identifier-governance Contract |
| REG-02 | `[AEP-I, §3.2, PDF p. 27, publication p. 9]` | Functional objective | Describe capabilities, limitations, deployment/location, outputs, lineage/provenance, security, releasability, and handling context | Server + authoritative-source Contract |
| REG-03 | `[AEP-I, §3.2, PDF p. 27, publication p. 9]` | Functional `must`/`requires` | Preserve temporal markers and validity; propagate and reconcile identity/description updates as connectivity permits | Server + synchronization Contract |
| ACCESS-01 | `[AEP-I, §3.3, PDF pp. 27–28, publication pp. 9–10]` | Functional workflow | Provide discrete retrieval and broader sharing, including useful selective retrieval instead of forced whole-resource transfer | Server; exact selection bounded by OGC/profile |
| ACCESS-02 | `[AEP-I, §§2.2, 3.3, PDF pp. 23, 28, publication pp. 5, 10]` | `should` plus functional `must` | Preserve semantics, metadata, lineage, producer, time, feature of interest, freshness, origin, suitability, addressability, and machine interpretation | Server |
| ACCESS-03 | `[AEP-I, §§2.3, 3.3, PDF pp. 23, 28, publication pp. 5, 10]` | Functional constraint | Govern access and preserve semantic meaning | Server enforcement + policy Ecosystem |
| ACCESS-04 | `[AEP-I, §3.3, PDF p. 28, publication p. 10]` | Functional `must` | Support staged, prioritized, and asynchronous exchange when persistent connectivity is unavailable | Server + Contract; mission/network Ecosystem |
| STREAM-01 | `[AEP-I, §§2.3, 3.4, PDF pp. 23, 28–29, publication pp. 5, 10–11]` | Functional workflow | Support sustained, repeated, and event-driven observations, detections, tracks, alerts, mode changes, and status | Server |
| STREAM-02 | `[AEP-I, §3.4, PDF p. 28, publication p. 10]` | Functional `must` | Preserve temporal context, sequence, change information, metadata, and operational currentness | Server |
| STREAM-03 | `[AEP-I, §3.4, PDF pp. 28–29, publication pp. 10–11]` | Functional `must` | Preserve usefulness when dynamic data is prioritized, staged, or resumed without continuous high bandwidth | Server + Contract; mechanism unresolved |
| TASK-01 | `[AEP-I, §§2.3, 3.5, PDF pp. 24, 29, publication pp. 6, 11]`; `[AEP-II, §1.1.2 and Lexicon, PDF pp. 50, 57, publication p. 3 and Lex-2]` | Functional objective + adopted technical role | Accept authorized observation requests and control; expose feasibility and accepted/scheduled/executed/rejected/cancelled lifecycle | Server + execution Contract |
| TASK-02 | `[AEP-I, §3.5, PDF p. 29, publication p. 11]` | Functional dependency | Expose controllable functions, parameters, limits, and expected responses | Server + authoritative-capability Contract |
| TASK-03 | `[AEP-I, §3.5, PDF p. 29, publication p. 11]` | Functional `must` | Enforce organizational, security, accreditation, policy, resource-owner, and operational constraints on control | Server + policy Ecosystem |
| TASK-04 | `[AEP-I, §3.5, PDF p. 29, publication p. 11]` | Functional `must` | Support local autonomy, staged/delayed delivery or execution, and asynchronous confirmation | Server Contract; edge behavior external |
| STAT-01 | `[AEP-I, §§2.3, 3.6, PDF pp. 24, 30, publication pp. 6, 12]` | Functional objective | Expose active, inactive, and degraded state and resulting precision/range/reliability effects | Server + source-state Contract |
| STAT-02 | `[AEP-I, §3.6, PDF p. 30, publication p. 12]` | Functional objective | Expose current availability, capacity, scheduled maintenance, and duty-cycle limitations | Server + source-state Contract |
| STAT-03 | `[AEP-I, §3.6, PDF p. 30, publication p. 12]` | Functional `must` | Preserve status timestamp and validity so consumers can distinguish current from stale state | Server |
| STAT-04 | `[AEP-I, §3.6, PDF p. 30, publication p. 12]` | Functional `must` | Support last-known state and asynchronous synchronization under delayed/intermittent connectivity | Server + synchronization Contract |
| TRUST-01 | `[AEP-I, §§1.4, 2.2, PDF pp. 21, 23, publication pp. 3, 5]` | Framework objective + `should` | Make information/interactions understandable, linked, trustworthy, interoperable, and secure with source, method, quality, provenance, spatial, temporal, and validity context | Server + trust Ecosystem |
| SEC-01 | `[AEP-I, Preface; §§2.4.1, 3.2, 3.5, PDF pp. 12, 24, 27, 29]` | Functional constraint | Restrict information and control to authorized actors and support policy-driven boundaries across organizations and accreditation domains | Server enforcement + identity/policy/gateway Ecosystem |
| SEC-02 | `[AEP-I, §4.2, PDF p. 32, publication p. 14]`; `[AEP-II, §2.2, PDF p. 54, publication p. 7]` | `should` / gap statement | Align the OGC package with applicable security, trust, semantic, metadata, and federation mechanisms beyond its own scope | Server integration + external profiles |
| DDIL-01 | `[AEP-I, §§2.2, 2.4–2.4.2, PDF pp. 23–25, publication pp. 5–7]` | Strong functional constraints | Support disconnected/intermittent local usefulness, constrained exchange, later sync, payload context, progressive enrichment, change-aware updates, freshness, and compact representations where appropriate | Server + Contract + Ecosystem |
| CONF-01 | `[AEP-I, §4.1, PDF p. 31, publication p. 13]`; `[AEP-II, Application and Chapter 2, PDF pp. 43, 53, publication pp. III, 6]` | Objective + direct implementation pointer | Ensure consistent behavior, accurate conformance claims, and verification against adopted OGC requirements | Server |
| ADJ-01 | `[S4789, Related Documents, PDF p. 6, publication p. 2]`; `[AEP-II, Linkages and Informative References, PDF pp. 43, 46]` | Informative linkage | Integrate with applicable adjacent standards without treating their listing as automatic server requirements | Adjacent |
| FUT-01 | `[AEP-I, §4.4, PDF p. 33, publication p. 15]`; `[AEP-II, §2.4, PDF p. 55, publication p. 8]` | Future-governance statement | Do not invent future AEP, SRD, profile, convention, dictionary, or registry obligations before promulgation and project assignment | Out of current scope |

### 12.3 Controlled Package Boundary

| Material | PDF Pages | Internal Pagination / Sections |
|---|---:|---|
| `AC/224(JCGISR)D(2026)0005` covering memorandum | 1–2 | Memorandum pp. -1 and -2 |
| Enclosure 1 — STANAG 4789 | 3–7 | Cover; promulgation p. i; substantive pp. 1–3 |
| Enclosure 2 — AEP-4789 Volume I | 8–37 | Cover; promulgation; roman pp. I–VIII; Chapters 1–4 pp. 1–15; Lexicon pp. 1–3 |
| Enclosure 3 — AEP-4789 Volume II | 38–59 | Cover; promulgation; roman pp. I–VII; Chapters 1–2 pp. 1–8; Lexicon pp. 1–3 |

Detailed AEP Volume I:

- Preface: PDF pp. 12–13 / publication pp. II–III
- References: PDF p. 17 / publication p. VII
- Conventions: PDF p. 18 / publication p. VIII
- Chapter 1: PDF pp. 19–21 / publication pp. 1–3
- Chapter 2: PDF pp. 22–25 / publication pp. 4–7
- Chapter 3: PDF pp. 26–30 / publication pp. 8–12
- Chapter 4: PDF pp. 31–33 / publication pp. 13–15
- Lexicon: PDF pp. 34–36 / Lex-1–Lex-3

Detailed AEP Volume II:

- Preface: PDF pp. 42–43 / publication pp. II–III
- References: PDF p. 46 / publication p. VI
- Conventions: PDF p. 47 / publication p. VII
- Chapter 1: PDF pp. 48–52 / publication pp. 1–5
- Chapter 2: PDF pp. 53–55 / publication pp. 6–8
- Lexicon: PDF pp. 56–58 / Lex-1–Lex-3

---

## Report Completion Checklist

- [x] Topic ID matches the overall research plan index
- [x] Topic research plan is linked and aligned
- [x] All core and detailed research questions are covered or explicitly resolved as source gaps
- [x] Findings are evidence-backed with reproducible references
- [x] Normative and informative evidence are classified and not conflated
- [x] Mutable sources identify a version, release, or commit
- [x] Controlled, inaccessible, missing, and ambiguous evidence limitations are explicit
- [x] Source-backed findings, analyst interpretation, and project recommendations are distinguishable
- [x] No conflict with an accepted prior topic report exists
- [x] Executive summary is independently readable by the project lead, implementers, and later AI agents
- [x] Recommendations are explicit and actionable
- [x] Risks and open questions are documented
- [x] Success criteria, methodology phases, research questions, and deliverable requirements are validated
- [ ] Plan-owner acceptance and acceptance date are recorded before the topic is treated as complete downstream
- [x] Next steps and downstream handoffs are assigned

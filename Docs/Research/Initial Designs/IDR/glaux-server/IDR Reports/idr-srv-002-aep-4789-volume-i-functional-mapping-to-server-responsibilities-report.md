# Section 002: AEP-4789 Volume I Functional Mapping to Server Responsibilities - Research Report

**Topic ID:** IDR-SRV-002<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-002 Research Plan](../IDR%20Plans/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** 45 of 45 (5 core and 40 detailed questions)<br>
**Methodology Used:** Integrity verification and complete page review of AEP-4789 Volume I; functional extraction with page and section anchors; normative-strength classification; layered server-boundary mapping; lightweight alignment against the current approved OGC standards package; downstream traceability design; and independent coverage review<br>
**Research Time:** Approximately 0.75 hours of AI-assisted elapsed execution time on July 30, 2026<br>
**Primary Source(s):**
- Project-controlled `AC/224(JCGISR)D(2026)0005`, dated 27 April 2026, AEP-4789 Volume I enclosure, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`
**Supporting Resources:**
- [OGC API - Connected Systems - Part 1: Feature Resources](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC SensorML Encoding Standard 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model Encoding Standard 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [Accepted IDR-SRV-001 Report](idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md)
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research Planning Approach](../../../../../Governance/research-planning-approach.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
**Document Purpose:** Turn the AEP-4789 Volume I functional view into a bounded, traceable responsibility baseline that the project lead and later AI-assisted work can use to design the open-source Rust Glaux Server without shrinking its intended capability or absorbing the rest of the ecosystem into the server<br>
**Author(s):** OpenAI Codex<br>
**Accepted By:** TBD - Glaux Project Lead review pending<br>
**Acceptance Date:** TBD - review pending<br>
**Date:** July 30, 2026<br>
**Last Updated:** July 30, 2026

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base
4. Findings by Research Question
5. Decision Analysis and Function-to-Responsibility Baseline
6. Key Recommendations
7. Implementation Implications and Test Strategy
8. Risks, Constraints, and Open Questions
9. Validation Against the Research Plan
10. Next Steps and Handoff
11. References
12. Appendices

---

## 1. Executive Summary

### 1.1 Plain-English Decision Brief

AEP-4789 Volume I describes what a connected-systems environment is intended to make possible; it does not prescribe the detailed server design. Its six named functions are discovery, registration and description, access and exchange, streaming and dynamic data, tasking and control, and status and availability. Security, trust, federation, and operation through poor or intermittent communications apply across all six.

For Glaux Server, the practical answer is:

- **The server owns the standards-facing behavior:** addressable resources, queries, links, machine-readable descriptions, data and command interfaces, lifecycle state, contextual integrity, authorization enforcement, and evidence that its advertised behavior works.
- **The server needs explicit contracts with other capabilities:** data and metadata producers, task executors, identity and policy authorities, federation or cross-domain gateways, and streaming or synchronization infrastructure.
- **The wider ecosystem still owns operational truth:** sensing and actuation, mission priorities, accreditation and release policy, network availability, cross-domain transfer, and operator decisions.

The right baseline is therefore a layered responsibility map, not a declaration that every AEP function is server code. That map preserves the ambitious Glaux Server goal while preventing scope drift.

### 1.2 Principal Findings

**Source-backed finding.** AEP-4789 Volume I is a concise reference view. It describes the problem space, representative workflows, high-level functional objectives, and operational constraints. It expressly leaves detailed technical prescriptions to later AEP volumes, Standard-Related Documents (SRDs), profiles, and the original standards. `[AEP-I, Preface, PDF pp. 12–13, publication pp. II–III]`

**Source-backed finding.** Volume I names six core interoperability functions and repeatedly identifies the need for information and interactions to preserve identity, relationships, provenance, time, validity, freshness, quality, semantic meaning, authorization, and security context. `[AEP-I, §§1.4, 2.2–2.4, 3.1–3.6, PDF pp. 21–30, publication pp. 3–12]`

**Analysis.** Every named function has a direct Glaux Server contribution, but none is wholly satisfied by the server alone. The server can expose and govern information; it cannot create authoritative sensor truth, physically execute commands, make accreditation policy, operate a cross-domain gateway, or guarantee network connectivity.

**Analysis.** Poor connectivity is not a separate optional feature. It changes the required semantics of identity, update propagation, freshness, last-known state, asynchronous tasking, staged exchange, and later reconciliation. The exact algorithms are not specified by Volume I and must be resolved in later topics.

**Project recommendation.** Adopt the responsibility map in Section 5 as the controlling Volume I handoff. Carry each entry forward through:

`AEP function → responsibility class → server behavior or contract → OGC mapping → later IDR decision → automated verification`

### 1.3 Decision and Goal Impact

No change to [Glaux Server Goal and Definition v1.5](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) is recommended. Volume I supports that goal and sharpens its boundary. The server remains a full-scope, open-source Rust reference implementation of OGC API - Connected Systems; “full scope” means implementing the intended server capability coherently, not taking ownership of every external system or operational activity needed to produce an end-to-end mission outcome.

### 1.4 Matters Deliberately Left Open

Volume I does not decide:

- exact OGC conformance classes, requirement URIs, HTTP operations, filters, media types, or validation rules;
- the approved write/transaction approach while relevant OGC API - Features Part 4 work remains draft;
- a production pub/sub protocol or delivery guarantee;
- NATO authentication, authorization, releasability, federation, accreditation, or cross-domain profiles;
- readiness/status vocabularies and freshness thresholds;
- offline queues, change exchange, replay, synchronization, or conflict-resolution algorithms; or
- modality-specific profiles, dictionaries, registries, and future AEP/SRD content.

These are not report failures. They are correctly bounded handoffs to later research.

---

## 2. Scope and Plan Alignment

### 2.1 Topic Confirmation

This report executes only `IDR-SRV-002: AEP-4789 Volume I Functional Mapping to Server Responsibilities`. It does not execute, combine, or begin another research topic.

Completed in scope:

- verified the exact project-controlled package before review;
- reviewed every AEP-4789 Volume I page, including front matter, chapters, and lexicon;
- identified explicit functions, repeated cross-cutting concerns, constraints, and implied supporting behavior;
- preserved section and dual page anchors for extracted findings;
- classified each finding as direct server, server-side contract, ecosystem, future-profile/adjacent, or out of scope;
- identified affected server design areas without designing another Glaux product;
- performed a lightweight fit check against the four current approved OGC standards adopted by AEP-4789 Volume II;
- identified downstream research and test-strategy handoffs; and
- independently checked question, method, success-criterion, and deliverable coverage.

Not performed:

- exhaustive OGC `SHALL` or conformance-class extraction;
- detailed architecture, API, persistence, Rust crate, or deployment design;
- design of Glaux Publisher, Simulator, Web App, Mobile, identity services, gateways, or brokers;
- selection of future profiles or unapproved standards; or
- execution of `IDR-SRV-003` or any later topic.

### 2.2 Core Research Question Coverage

| ID | Core Question | Result | Primary Evidence |
|---|---|---|---|
| CQ-1 | What Volume I functional areas are relevant? | Complete: six named functions plus cross-cutting security/trust, federation, semantic/context, and DDIL concerns | Sections 4.1 and 12.2 |
| CQ-2 | Which functions map directly to Glaux Server? | Complete: direct responsibilities are bounded to behavior the server can expose, preserve, govern, or verify | Sections 4.2 and 5.2 |
| CQ-3 | Which functions require server-side contracts? | Complete: six external contract families identified without designing the external parties | Sections 4.3 and 5.3 |
| CQ-4 | Which functions are ecosystem, profile, adjacent, or out of scope? | Complete: boundary categories and exclusions are explicit | Sections 4.4, 5.2, and 5.3 |
| CQ-5 | What downstream topics require handoffs? | Complete: topic-specific handoff and traceability matrices provided | Sections 4.5 and 10.2 |

All 40 detailed questions are answered in Appendix 12.1.

---

## 3. Evidence Base

### 3.1 Source Inventory and Authority

| Source | Version / Date | Status and Authority for This Topic | Access / Reproducibility |
|---|---|---|---|
| `AC/224(JCGISR)D(2026)0005` enclosing AEP-4789 Volume I, *Sensor Integration Standard for NATO JISR Operations - Reference View* | Edition A, Version 1; package dated 27 April 2026 | **Controlling primary source.** Most-current ratification draft, as confirmed by the project lead; fixed project baseline | Controlled local source; not redistributed. Package SHA-256 is recorded above. Exact section and PDF/publication page anchors are supplied. |
| STANAG 4789 enclosure in the same package | Edition 1 | **Parent context only.** Establishes agreement-level relationship; not a substitute for Volume I analysis | Same controlled package and hash |
| AEP-4789 Volume II enclosure in the same package | Edition A, Version 1 | **Supporting alignment only.** Identifies the adopted technical package; not used for detailed technical extraction here | Same controlled package and hash |
| [IDR-SRV-002 topic plan](../IDR%20Plans/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities.md) | Baseline reviewed July 30, 2026 | **Controlling topic scope and completion criteria** | Versioned public repository artifact; accessed 2026-07-30 |
| [Overall IDR research plan](../IDR%20Plans/overall-idr-research-plan.md) | Version 1.2; July 30, 2026 | **Controlling sequence, governance, and topic index** | Versioned public repository artifact; accessed 2026-07-30 |
| [IDR-SRV-001 accepted report](idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md) | Accepted July 30, 2026 | **Accepted project interpretation baseline.** Constrains source authority and server boundaries | Public repository artifact; accessed 2026-07-30 |
| [OGC 23-001, CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) | Version 1.0; published 2025-07-16 | **Current approved OGC implementation standard.** Supporting fit check only in this topic | Official public OGC publication; accessed 2026-07-30 |
| [OGC 23-002, CSAPI Part 2](https://docs.ogc.org/is/23-002/23-002.html) | Version 1.0; published 2025-07-16 | **Current approved OGC implementation standard.** Supporting fit check only | Official public OGC publication; accessed 2026-07-30 |
| [OGC 23-000, SensorML](https://docs.ogc.org/is/23-000/23-000.html) | Version 3.0; published 2025-07-16 | **Current approved OGC implementation standard.** Supporting fit check only | Official public OGC publication; accessed 2026-07-30 |
| [OGC 24-014, SWE Common](https://docs.ogc.org/is/24-014/24-014.html) | Version 3.0.0; published 2025-07-16 | **Current approved OGC implementation standard.** Supporting fit check only | Official public OGC publication; accessed 2026-07-30 |
| [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) | Version 1.5; July 30, 2026 | **Project scope and intent.** Not standards evidence | Versioned public repository artifact; accessed 2026-07-30 |
| [Research Planning Approach](../../../../../Governance/research-planning-approach.md) and report/plan templates | Governance baseline current on July 30, 2026 | **Controlling research and reporting method.** Not standards evidence | Versioned public repository artifacts; accessed 2026-07-30 |
| [Glaux Server repository](https://github.com/DGIWG-P507/glaux-server/tree/1ba41159d1465797f1fceab129486197eb80aadf) | Commit `1ba41159d1465797f1fceab129486197eb80aadf`; 2026-06-07 | **Project context only.** Repository contained only its README at review time; no implementation behavior exists to analyze | Public immutable commit; accessed 2026-07-30 |
| [OS4CSAPI research-plan exemplar corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans) | Commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230` | **Reporting-style exemplar only.** No substantive evidence for Glaux obligations | Public immutable commit; accessed 2026-07-30 |

### 3.2 Controlled Package Integrity and Structure

The package hash was independently recomputed before research and matched the plan exactly:

`56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`

AEP-4789 Volume I occupies controlled PDF pages 8–37:

| Volume I Material | Controlled PDF Pages | Internal Pagination |
|---|---:|---|
| Cover, promulgation material, summary | 8–11 | Cover through p. I |
| Preface | 12–13 | pp. II–III |
| Contents, figure list, references, conventions | 14–18 | pp. IV–VIII |
| Chapter 1 - Introduction and problem space | 19–21 | pp. 1–3 |
| Chapter 2 - Reference view and operating conditions | 22–25 | pp. 4–7 |
| Chapter 3 - Functional view | 26–30 | pp. 8–12 |
| Chapter 4 - Standards framework and evolution | 31–33 | pp. 13–15 |
| Lexicon and end matter | 34–37 | Lex-1 through Lex-3 |

The volume contains one illustrative figure and no substantive requirement tables. The full page range was reviewed; extraction was not limited to keyword search.

### 3.3 Normative-Strength Rules

AEP-I Conventions defines:

- `shall` as a requirement;
- `should` as a recommendation; and
- `may` as permission.

The volume also uses lowercase `must` in functional prose, but `must` is not one of its formally defined normative keywords. This report records that wording as a **strong functional constraint**, not as a formal `shall`. Descriptive statements, representative use cases, and examples are not silently promoted into precise API requirements.

Two explicit formal requirements matter at this level:

- the AEP is to be interpreted, applied, and maintained within the NATO ISR Interoperability Architecture and its objectives; and
- measurements and units use SI unless otherwise specified; when operationally necessary, non-SI units may be shown in parentheses after the SI value.

The first is principally an architecture/governance obligation. The second has direct future serialization and validation implications. Neither turns the rest of the reference-view prose into an exact server interface.

The six core-function descriptions contain no detailed, testable `shall` statements. Later Volume II/OGC evidence must provide the exact protocol and conformance obligations.

### 3.4 Evidence Limitations

- The source is project-controlling but visibly pre-promulgation: the covering memorandum requests ratification and enclosure pages retain placeholders. This report does not claim completed NATO promulgation.
- The sole normative AEP-I reference to AEDP-2/NIIA has edition, version, and date marked to be confirmed. The linkage is recorded, but detailed NIIA obligations are not inferred.
- The controlled source cannot be linked publicly or reproduced in this repository. The package identifier, hash, title, edition, section, and dual page anchors make review reproducible for authorized holders.
- PDF text extraction introduces ordinary line-break and hyphenation artifacts. Findings are paraphrased and checked against page images/text context.
- Volume I intentionally gives a reference view rather than a full NAF architecture or detailed technical specification.
- The official OGC standards were checked only for high-level functional fit. Detailed requirement extraction belongs to later plans.
- The current Glaux Server repository has no implementation to validate against this mapping.

### 3.5 Citation Shorthand

- `[AEP-I, §x, PDF p. y, publication p. z]` means AEP-4789 Volume I in the fixed controlled package.
- `[CSAPI-1, §x]`, `[CSAPI-2, §x]`, `[SensorML, §x]`, and `[SWE-Common, §x]` mean the approved OGC versions listed in Section 3.1.
- Mapping IDs in this report are local research identifiers, not official NATO or OGC requirement identifiers.

---

## 4. Findings by Research Question

### 4.1 CQ-1 - Relevant Volume I Functional Areas

**Source-backed finding.** Volume I explicitly identifies six connected-system functions. Four cross-cutting concerns recur through the problem statement, reference view, environmental discussion, functional descriptions, and standards rationale.

| Functional Area | Volume I Outcome | Principal Source Anchors | Repeated Cross-Cutting Context |
|---|---|---|---|
| Discovery | Authorized consumers can find information, observations, data series, sensors/systems, capabilities, coverage, constraints, identity, responsible organization, deployment, and validity | §§2.3, 3.1; PDF pp. 23, 26–27 / publication pp. 5, 8–9 | Identity, links, coverage, semantics, validity, authorization |
| Registration and description | Systems and their information have persistent identity and sufficient machine-readable description, capability, limitation, deployment, provenance, security, handling, and temporal context | §§2.2–2.3, 3.2; PDF pp. 23, 27 / publication pp. 5, 9 | Authoritative source, update propagation, reconciliation |
| Access and exchange | Authorized users and systems retrieve or share observations and sensor-derived information while preserving meaning and lineage; selective retrieval may be supported | §§2.2–2.3, 3.3; PDF pp. 23–24, 27–28 / publication pp. 5–6, 9–10 | Optional selection, context, asynchronous/staged exchange |
| Streaming and dynamic data | Continuous, repeated, and event-driven information remains useful as it changes, is prioritized, interrupted, staged, or resumed | §§2.3, 3.4; PDF pp. 23, 28–29 / publication pp. 5, 10–11 | Time, sequence, currentness, bandwidth constraints |
| Tasking and control | Authorized actors request observations or control, understand whether a request can be supported and what parameters apply, and receive status as requests are acted upon | §§2.3, 3.5; PDF pp. 24, 29 / publication pp. 6, 11 | Operational limits, owner authority, policy, operational constraints, delayed confirmation |
| Status and availability | Consumers understand active/inactive/degraded state, capability effects, availability, capacity, maintenance, duty cycle, and whether state is current or stale | §§2.3, 3.6; PDF pp. 24, 30 / publication pp. 6, 12 | Timestamp, validity, last-known state, synchronization |
| Security, trust, and authorization | Information and control are available only to authorized actors with applicable security, releasability, handling, policy, provenance, and quality context | Preface; §§1.4, 2.4.1, 3.2, 3.5, 4.2; PDF pp. 12, 21, 24, 27, 29, 32 | Applies to all six functions |
| Federation and cross-boundary sharing | Functions can span organizations and differently accredited environments through policy-driven boundaries and trustworthy relationships | §§2.4.1, 4.2; PDF pp. 24, 32 / publication pp. 6, 14 | Identity, policy, gateway, trust, link integrity |
| DDIL / constrained operation | Local operation remains useful; exchange can be constrained, delayed, enriched, prioritized, resumed, and later reconciled | §§2.2, 2.4–2.4.2 and 3.2–3.6; PDF pp. 23–30 / publication pp. 5–12 | Context, freshness, last-known state, change awareness |

Repeated concepts that require special treatment across later server design are:

- persistent identity and links;
- provenance, source, method, lineage, and quality;
- temporal position, validity, freshness, current/stale/last-known state;
- semantic and structural metadata sufficient for machine interpretation;
- authorization, trust, security, releasability, and handling;
- staged, prioritized, asynchronous, interrupted, and resumed interaction; and
- consistent open-standard behavior rather than bespoke translations.

Dependency crosswalk:

| Function | Metadata / Semantics | Dynamic Data / Events | Tasking / Control | Security / Policy | Status / Time |
|---|---|---|---|---|---|
| Discovery | Essential | Dynamic holdings must be findable under later API rules | Capabilities/interfaces are a later design implication | Authorized discovery | Coverage, deployment, and validity context |
| Registration / description | Core function | Changes and enrichment | Controllable capabilities and limits | Markings, releasability, handling | Validity, freshness, deployment history |
| Access / exchange | Context must travel with data | Observations, alerts, and staged exchange | Request/response may include controlled interaction | Governed access and cross-boundary transfer | Historical, current, and delayed information |
| Streaming / dynamic data | Structure and semantics retained | Core function | Carries changing control/status information where applicable | Authorized flow and policy | Time, sequence, change, currentness |
| Tasking / control | Capabilities, limits, parameters | Asynchronous status/results | Core function | Authority, accreditation, policy, ownership | Supportability and lifecycle state |
| Status / availability | Meaning, source, validity | Updates and system events | Informs whether interaction is supportable | Authorized disclosure | Core function: current/stale/last-known |

No material conflict with the accepted IDR-SRV-001 report was found. This report refines its Volume I mapping while retaining its controlled-source status, normative-strength cautions, and server/ecosystem boundary.

### 4.2 CQ-2 - Direct Glaux Server Responsibilities

**Analysis.** A function maps directly to Glaux Server when the required outcome is controlled by the server's API behavior, resource model, stored or computed representations, policy enforcement, or conformance evidence. This is a project responsibility assignment derived from Volume I; it is not a claim that Volume I prescribes an exact implementation.

| Direct Responsibility | Why It Is Direct | Important Boundary |
|---|---|---|
| Expose stable, addressable, linked connected-system resources and navigable relationships | Discovery and reuse require persistent identity and independent addressability | Global identifier allocation and federation governance need external authority |
| Expose machine-readable descriptions of systems, procedures, deployments, capabilities, limitations, properties, and context | Discovery and meaningful requests depend on description | The authoritative facts may originate outside the server |
| Provide standards-aligned search, filtering, retrieval, and historical access supported by the adopted API | The server controls its query and response behavior | Volume I does not prescribe exact filters, landing pages, pagination, or HTTP operations |
| Preserve relationships among data, source/system, observed or controlled property, feature/matter of interest, time, location, provenance, quality, and validity | Volume I repeatedly identifies preservation of semantic meaning and context as necessary | The server cannot guarantee the truth of context supplied by an external source |
| Represent observations, datastreams, status, events, control streams, commands, feasibility, command status, and results within the selected OGC profile | These are the server-facing resources through which the named functions are realized | Physical production, delivery, and execution remain external |
| Validate accepted representations and reject or report invalid interactions consistently | The server owns the integrity of its exposed contract | Exact schemas, error model, and validation architecture are later decisions |
| Enforce authorization and applicable policy at every exposed information and control boundary | The server must not disclose or accept operations contrary to policy | Identity proof, accreditation, release rules, and cross-domain transfer policy come from external authorities |
| Preserve time, validity, freshness, update, sequence, and last-known-state semantics | Consumers must distinguish current, delayed, stale, and historical information | Thresholds and reconciliation algorithms are later profile/design decisions |
| Support asynchronous state transitions for long-running or disconnected interactions | Volume I anticipates staged/delayed tasking, exchange, and confirmation | Queueing, broker, transport, and edge execution may be external |
| Advertise only behavior and conformance actually implemented and tested | Volume I calls for consistent open-standard implementation | Exact conformance classes are selected later |

### 4.3 CQ-3 - Required Server-Side Integration Contracts

**Analysis.** A contract responsibility exists when Glaux Server owns its side of an interaction but cannot produce the complete outcome alone.

| Contract Family | Glaux Server Side | External Side | Not Assigned Here |
|---|---|---|---|
| Metadata and resource publication | Accept, validate, identify, link, version, expose, and report processing outcome | Supply authoritative descriptions, capabilities, deployments, lineage, markings, and changes | Publisher/Simulator or administrative workflow design |
| Observation, status, and event ingestion | Accept or receive typed updates; preserve context and time; expose current/historical views; report failures | Produce authoritative values, timestamps, quality, sequence/change information, and source identity | Sensor acquisition and producer internals |
| Tasking and control execution | Authorize and validate requests; expose feasibility and lifecycle; correlate status/results | Assess or execute against the connected system; return authoritative progress/result | Actuation logic, safety interlocks, or mission scheduling implementation |
| Identity, authorization, and policy | Consume trusted identity/claims/policy; enforce decisions consistently; minimize unauthorized disclosure | Authenticate identities; issue attributes; govern roles, mission policy, accreditation, releasability, and handling | Choice or operation of enterprise identity/policy infrastructure |
| Federation and cross-domain exchange | Maintain stable identifiers, links, source context, policy hooks, and externally visible contract behavior | Route/query/replicate as selected; adjudicate and perform cross-boundary release; preserve trust | Federation topology or gateway product design |
| Streaming, synchronization, and constrained transport | Expose resource semantics, resumable/correlation state where selected, freshness, and reconciliation inputs/results | Deliver messages or changes, tolerate link behavior, operate broker/gateway/cache as selected | Premature selection of MQTT, WebSocket, queue, or offline protocol |

The contract names are deliberately generic. A Glaux component may later implement one side, but this report does not design that component or make it part of the server.

### 4.4 CQ-4 - Ecosystem, Future-Profile, Adjacent, and Out-of-Scope Functions

| Classification | Volume I Concern | Boundary Finding |
|---|---|---|
| Ecosystem | Sensor observation, state determination, and physical actuation | The server exposes and governs representations; the connected system or mediator remains authoritative for physical truth and execution |
| Ecosystem | Mission priority, resource ownership, accreditation, release policy, and operator authority | The server enforces supplied policy but does not originate mission governance |
| Ecosystem | Network availability, radio/link behavior, and cross-domain gateway operation | The server can tolerate and report constraints but cannot guarantee connectivity or operate all transport infrastructure |
| Future profile / later project decision | Exact OGC conformance profile, transaction subset, security profile, status vocabulary, and semantic registries | Volume I states the desired outcomes; later standards and project decisions define testable detail |
| Future profile / later project decision | Pub/sub binding, delivery guarantee, replay/resume policy, offline synchronization, delta exchange, and conflict resolution | The approved package supplies useful primitives but not these complete mechanisms |
| Ecosystem / project governance | NIIA linkage | The explicit `shall` linkage applies to project architecture/governance; the incomplete normative reference prevents detailed NIIA requirements from being inferred here |
| Adjacent | Other NATO publications | Linkage may matter, but no detailed adjacent-standard requirement is inferred without direct later review |
| Out of scope | Internal sensing-system redesign and native interfaces | Volume I focuses on external information/control boundaries and permits proximate mediation |
| Out of scope | Operator training and a full NAF-conformant architecture description | Volume I expressly excludes these purposes |
| Out of scope | Building Glaux Web App, Mobile, Publisher, or Simulator behavior | Only their potential server-facing contracts are in scope |
| Out of scope | Unpromulgated future AEP volumes, SRDs, profiles, dictionaries, or registries | Do not invent future obligations |

**Source-backed finding.** Volume I's “responsibility to provide” objective concerns making information available for authorized operational use. It does not override authorization, releasability, handling, accreditation, or other policy.

### 4.5 CQ-5 - Required Downstream Handoffs

The findings must be handed forward rather than repeatedly rediscovered:

| Handoff | Receiving Topics | Content |
|---|---|---|
| Standards-package fit and gaps | 003, 006, 007 | Six-function mapping; OGC high-level fit; precise matters that require normative extraction |
| Terminology | 004 | Repeated and ambiguous Volume I terms listed in Section 10.3 |
| API behavior | 009–014 | Discovery/addressability needs, but explicit warning that landing pages, links, filters, media types, errors, and OpenAPI details come from OGC evidence |
| Canonical model and context | 015–020 | Identity, links, time, validity, freshness, provenance, quality, trust, status, events, and last-known state |
| Encodings and semantics | 021–024 | SensorML/SWE roles, SI units, semantic binding, validation, and external vocabulary gaps |
| Persistence and lifecycle | 025–030 | History, update propagation, current/last-known views, consistency, staged change, retention, and reconciliation inputs |
| Ingestion and component contracts | 031–035 | Producer authority, validation feedback, idempotency/correlation, dynamic updates, and transport boundary |
| Tasking | 036–038 | Feasibility, authorization, lifecycle, delayed execution/confirmation, safety, and accountability implications |
| Security and constrained operation | 039–043 | Policy enforcement, releasability, auditing, DDIL semantics, synchronization, and conflict boundary |
| Verification | 050–056 | Traceability fields and functional scenario families in Section 7.3 |

---

## 5. Decision Analysis and Function-to-Responsibility Baseline

### 5.1 Options Considered

| Option | Evaluation |
|---|---|
| A. Treat every Volume I outcome as direct server functionality | Rejected. It would absorb sensors, actuators, identity authorities, policy, gateways, transports, and operational governance into Glaux Server. |
| B. Map only obvious OGC resource names and ignore broader Volume I outcomes | Rejected. It would lose security, context, freshness, federation, DDIL, and lifecycle intent and encourage a narrow demonstration server. |
| C. Use a layered map of direct server, contract, ecosystem, future-profile/adjacent, and out-of-scope responsibilities | **Adopt.** It preserves the complete server goal while assigning external dependencies honestly. |
| D. Defer all interpretation until every downstream profile is finalized | Rejected. The functional boundary can be established now even though detailed technical choices remain later. |

### 5.2 Functional Mapping Baseline

Classification abbreviations:

- **S** - direct Glaux Server responsibility
- **C** - server-side integration contract
- **E** - ecosystem responsibility
- **F** - future profile, adjacent standard, or later project decision
- **O** - out of current Glaux Server scope

Source-strength abbreviations:

- **`shall` / `should` / `may`** - the convention-defined terms as used in the cited passage
- **FM** - strong functional wording such as lowercase `must` or `requires`; not a convention-defined normative keyword
- **FO** - functional objective or scope statement
- **RW** - representative workflow or example
- **B** - boundary or exclusion
- **FG** - future/profile/SRD governance statement
- **A** - this report's analysis or synthesis, not an extracted AEP requirement

The **Primary** column records one dominant assignment. Additional classes identify dependencies or limits. Every row also names its downstream research owner; those topics must still verify their own controlling evidence.

| ID | AEP Anchor (PDF / Publication) | Strength | Normalized Functional Outcome | Primary | Additional | Affected Server Area / Boundary | Downstream Topic(s) |
|---|---|---|---|---|---|---|---|
| GOV-01 | Preface Linkages; p. 13 / p. III | `shall` | Interpret, apply, and maintain the AEP within NIIA and its interoperability objectives | E | F | Project architecture/governance linkage; no detailed NIIA requirement inferred | 005; final synthesis |
| UNIT-01 | Conventions; p. 18 / p. VIII | `shall`; `may` | Use SI for measurements/units unless otherwise specified; operationally necessary non-SI units may follow the SI value in parentheses | S | C, F | Serialization/validation; exact unit profile later | 023–024, 050–053 |
| FW-01 | Preface; §§1.1–1.4; pp. 12–13, 19–21 / pp. II–III, 1–3 | FM / FO | Support authorized people, applications, AI, and services across heterogeneous enterprise, coalition, federated, and tactical contexts | S | C, E | API audience and deployment context; no actor-specific UI | 003–004, 015, 039–042 |
| FW-02 | §§1.3–1.4; pp. 20–21 / pp. 2–3 | `should` / FO | Separate information from originating hardware/software and reduce bespoke bridges and vendor lock-in | S | C | Open, stable external contract; native internals external | 003, 015–017, 031–033 |
| FW-03 | §2.2; pp. 22–23 / pp. 4–5 | FM / B | Focus on external information/control boundaries and permit native or proximate mediation without redesigning a sensing asset | C | S, O | Adapter boundary; sensing-system redesign out of scope | 031–033, 036, 043 |
| FW-04 | §§1.4, 4.1–4.3; pp. 21, 31–33 / pp. 3, 13–15 | FO | Make functions consistently discoverable, accessible, understandable, linked, trustworthy, interoperable, and secure through open standards | S | C, E | Cross-cutting qualities and conformance | 003–008, 039–042, 050–056 |
| FW-05 | §1.1; p. 19 / p. 1 | FO | Remain neutral to conventional sensors, process-based and human-associated sources, and aggregated or virtual sources | S | C | Resource model must not assume one physical sensor form | 004, 015, 021–024 |
| FW-06 | §1.3; pp. 20–21 / pp. 2–3 | `should` / FO | Preserve durable value and reuse independently of the originating system | S | C, E, F | Persistence/access principle; no indefinite-retention rule | 025, 028, 030 |
| DISC-01 | §§2.3, 3.1; pp. 23, 26 / pp. 5, 8 | FO | Discover information, observations, raw series, and historical holdings | S | C | Search/resource discovery; holdings may be external | 006, 009–011, 015, 027 |
| DISC-02 | §3.1; pp. 26–27 / pp. 8–9 | RW | Discover sensing systems with capability, limitation, coverage, and deployment context | S | C | Indexed descriptions backed by authoritative metadata | 004, 006, 010–011, 015 |
| DISC-03 | §3.1; pp. 26–27 / pp. 8–9 | RW | Expose formats, units, parameters, designation, persistent ID, responsible organization, and validity | S | C, F | Description/query model; vocabulary and allocation later | 004, 006, 015–018, 024 |
| DISC-04 | §§2.2–2.4, 3.1; pp. 23–26 / pp. 5–8 | FM | Preserve enough metadata for meaningful requests in disconnected or federated conditions | S | C, E, F | Portable context; federation/offline policy later | 006, 015, 018–019, 040, 042 |
| DISC-05 | §4.1; p. 31 / p. 13 | FO | Provide consistent discovery behavior using open standards | S | F | OGC conformance mapping and testing | 003, 006, 008–011, 050–051, 056 |
| DISC-06 | §§2.3, 3.4–3.6; pp. 23–24, 28–30 / pp. 5–6, 10–12 | A | Treat discovery of service, datastream, event, status, and tasking interfaces as a later OGC design implication needed to use the named functions | F | S, C | Not an explicit Volume I discovery outcome; requires Part 1/Part 2 evidence | 006–010, 020, 034–036 |
| REG-01 | §§2.2–2.3, 3.2; pp. 23, 27 / pp. 5, 9 | FO | Establish persistent identity, designation, links, and responsible organization | S | C, E | Canonical resource identity; governance external | 004, 006, 015–017, 031 |
| REG-02 | §3.2; p. 27 / p. 9 | RW | Describe capabilities, limitations, deployment/current and historical location, and outputs | S | C | Resource/document model; facts supplied externally | 006, 015, 021, 028, 031–033 |
| REG-03 | §3.2; p. 27 / p. 9 | RW | Preserve provenance, security, releasability, and handling context | S | C, E, F | Metadata and enforcement hooks; policy semantics external/later | 019, 021, 028, 039–041 |
| REG-04 | §§2.2, 3.2; pp. 23, 27 / pp. 5, 9 | FM | Preserve timestamps and validity sufficient for consumers to assess freshness, together with structure and machine interpretability | S | C, F | Temporal and representation model | 018, 021–024, 028 |
| REG-05 | §§2.4.2, 3.2; pp. 25, 27 / pp. 7, 9 | FM | Propagate and reconcile identity/description changes as connectivity permits | C | S, E, F | Server exposes change state; algorithms/topology later | 016, 029, 031–032, 042–043 |
| ACCESS-01 | §3.3; pp. 27–28 / pp. 9–10 | RW | Retrieve/share observations, detections, alerts, and other sensor-derived information | S | C | Dynamic-data and resource retrieval | 007, 010–012, 015, 027, 034 |
| ACCESS-02 | §3.3; p. 28 / p. 10 | `may` / RW | Permit selection of variables, parameters, resolution, or portions when the later standards/profile supports it | F | S | Optional/illustrative, not an unconditional Volume I server responsibility | 006–007, 011–012, 026–027 |
| ACCESS-03 | §§2.2, 3.3; pp. 23, 28 / pp. 5, 10 | FM | Preserve metadata, lineage, producer, time, feature/object/matter of interest, freshness, origin, and suitability | S | C | Context integrity across stored/returned data | 015–019, 021–024, 027–028 |
| ACCESS-04 | §§2.3, 3.3; pp. 23, 28 / pp. 5, 10 | FO | Govern access while preserving semantic meaning | S | C, E, F | Policy enforcement and response behavior | 012–013, 019, 023–024, 039–040 |
| ACCESS-05 | §§2.4, 3.3; pp. 24–25, 28 / pp. 6–7, 10 | FM | Permit staged, prioritized, or asynchronous exchange under constrained connectivity | C | S, E, F | Server state/interface; priority/network mechanism later | 029, 034–035, 040, 042–043, 054 |
| DYN-01 | §§2.3, 3.4; pp. 23, 28–29 / pp. 5, 10–11 | FO / RW | Expose continuous, repeated, or event-driven observations, detections, tracks, alerts, status, and mode changes | S | C | Dynamic resources/events; modality examples are not mandatory schemas | 007, 020, 027, 034–035 |
| DYN-02 | §3.4; p. 28 / p. 10 | FM | Preserve time, sequence, change, metadata, and operational currentness | S | C, F | Temporal/ordering model; exact late-arrival rules later | 018–020, 027, 034, 042–043 |
| DYN-03 | §3.4; pp. 28–29 / pp. 10–11 | FM | Retain usefulness when data is prioritized, staged, interrupted, or resumed | C | S, E, F | Resumption/correlation inputs; transport/policy later | 029, 034–035, 042–043, 054 |
| DYN-04 | §3.4; p. 29 / p. 11 | B | Avoid assuming one delivery model | F | S, C | Keep resource semantics separable from transports | 003, 007, 035, 042, 054 |
| TASK-01 | §§2.3, 3.5; pp. 24, 29 / pp. 6, 11 | RW | Accept authorized requests for observations, parameters, modes, or control | S | C, E | Command/control API and authorization boundary | 007, 032–033, 036, 038–039 |
| TASK-02 | §3.5; p. 29 / p. 11 | FO | Expose controllable functions, limits, parameters, and expected responses | S | C | Description supplied by authoritative capability source | 004, 007, 015, 021–024, 036 |
| TASK-03 | §3.5; p. 29 / p. 11 | RW / A | Represent supportability assessment—later mapped to CSAPI feasibility—and whether requests are accepted, scheduled, executed, or otherwise acted upon | S | C, F | Asynchronous lifecycle; exact operation/states from later CSAPI evidence | 007, 036–037, 051–053, 055 |
| TASK-04 | §3.5; p. 29 / p. 11 | FM | Apply organizational, security, accreditation, policy, resource-owner, and operational constraints | S | C, E, F | Server enforcement; rule authority/adjudication external | 036–041, 055 |
| TASK-05 | §3.5; p. 29 / p. 11 | FM | Support local autonomy and staged/delayed action or confirmation | C | S, E, F | Server lifecycle/correlation; local execution external | 029, 036–038, 042–043, 055 |
| TASK-06 | §3.5; p. 29 / p. 11 | B | Avoid assuming uniform controllability or permission across systems/contexts | S | C, E, F | Capability- and policy-driven behavior | 024, 036–040, 055 |
| STAT-01 | §§2.3, 3.6; pp. 24, 30 / pp. 6, 12 | RW | Expose active, inactive, and degraded state and effects on precision, range, or reliability | S | C, F | General status model/source contract; vocabulary later | 004, 018–020, 034 |
| STAT-02 | §3.6; p. 30 / p. 12 | RW | Expose availability, capacity, maintenance, and duty-cycle limitations | S | C, F | Status/capability model; semantics later | 004, 020, 024, 034 |
| STAT-03 | §3.6; p. 30 / p. 12 | FM | Preserve update time and validity so current and stale state are distinguishable | S | C, F | Freshness/validity rules | 018, 020, 034, 042 |
| STAT-04 | §§2.4.2, 3.6; pp. 25, 30 / pp. 7, 12 | FM | Support last-known state and later asynchronous synchronization | S | C, E, F | Historical/current views; reconciliation later | 018, 020, 029, 034, 042–043 |
| STAT-05 | §3.6; p. 30 / p. 12 | B | Avoid assuming uniform status-reporting method or cadence | S | C, F | Flexible source contract/canonical exposure | 020, 031–034, 042 |
| SEC-01 | Preface; §§1.4, 2.4.1; pp. 12, 21, 24 / pp. II, 3, 6 | FM / FO | Restrict discovery, information, and interaction to authorized actors | S | C, E, F | Enforcement across endpoints; identity/profile external/later | 039–041, 050–056 |
| SEC-02 | §§3.2, 3.5; pp. 27, 29 / pp. 9, 11 | RW / FM | Carry/apply security, releasability, handling, and tasking-authority context | S | C, E, F | Markings and policy-decision integration | 019, 038–041, 055 |
| SEC-03 | §§2.4.1, 4.2; pp. 24, 32 / pp. 6, 14 | FM / `should` | Cross organizations/accreditation domains through policy-driven gateways and trust mechanisms | C | S, E, F | Contract/policy hooks; gateway operation/trust profile external | 039–043, 055–056 |
| SEM-01 | §§1.4, 2.2, 4.2; pp. 21, 23, 32 / pp. 3, 5, 14 | `may` / FO | Add semantic alignment, mappings, or guidance when common syntax is insufficient | F | S, C, E | Server applies selected bindings; semantic authority external/later | 004, 019, 021–024 |
| DDIL-01 | §§2.4–2.4.2; pp. 24–25 / pp. 6–7 | FM | Support useful local operation while disconnected and constrained exchange while degraded | C | E, S, F | Edge/network outcome; server preserves context and must not assume continuously connected producers | 031–035, 042–043, 054 |
| DDIL-02 | §§2.2, 2.4.2; pp. 23, 25 / pp. 5, 7 | FM | Carry essential context with payloads and allow progressive metadata enrichment | S | C, F | Minimum-context/enrichment rules later | 015, 018–019, 028–029, 042–043 |
| DDIL-03 | §2.4.2; p. 25 / p. 7 | FO | Exchange changes efficiently and preserve freshness/validity | C | S, E, F | Change/sync interface; algorithms later | 018, 029, 034, 042–043 |
| DDIL-04 | §§2.4.2, 3.3–3.6; pp. 25, 28–30 / pp. 7, 10–12 | FM | Support delayed/asynchronous updates, tasking, status, and later reconciliation | C | S, E, F | Correlation/state machines; no assumed transport | 029, 034–038, 042–043, 054–055 |
| DDIL-05 | §2.4.2; p. 25 / p. 7 | FO | Favor incremental, change-aware exchange and avoid assuming complete retransmission | C | S, E, F | Delta/resume protocol and priority policy later | 029, 034–035, 042–043, 054 |
| DDIL-06 | §2.4.2; p. 25 / p. 7 | `may` | Permit compact/simplified forms under constraint while retaining operational meaning | F | S, C | Optional allowance, not a mandated encoding | 012, 022–024, 035, 042, 054 |
| STD-01 | §§4.1–4.2; pp. 31–32 / pp. 13–14 | FO | Use complementary open standards as a coordinated package and implement them consistently | S | F | Conformance-first server profile | 003, 006–008, 050–056 |
| STD-02 | §§4.2–4.4; pp. 32–33 / pp. 14–15 | `should` / FG | Recognize security, trust, semantic alignment, and future material as additional to the core package | F | S, C, E, O | Gap register; do not invent future obligations | 003–005, 039–043 |
| STD-03 | §4.1; p. 31 / p. 13 | FO | Consider maturity, openness, implementability, security, trustworthiness, public availability, and intellectual-property constraints when selecting standards | F | S, E | Project profile/technology governance, not an extra API requirement | 003, 005, 008, 045–049 |
| STD-04 | §4.2; p. 32 / p. 14 | `may` / FG | Allow later profiles/guidance to constrain optionality, clarify semantics, and define minimum consistently described metadata | F | S, C | Future profile boundary; no current requirement invented | 003–008, 015, 024 |
| ECO-01 | §2.3; p. 24 / p. 6 | FO | Contribute information to the common intelligence feed and operational/intelligence/tactical pictures, fusion, PED, and awareness | E | S, C | Server supplies governed information/state; end-to-end mission outcomes external | 004, 015, 019–020, 056 |
| OOS-01 | Preface Application; p. 13 / p. III | B | Do not use Volume I as operator training or a full NAF-conformant architecture description | O | E | Explicit purpose boundary | All later topics (scope guardrail) |
| OOS-02 | §2.2; p. 23 / p. 5 | B / A | Do not impose uniform sensing design, phenomenology, or operational employment, or require wholesale redesign/physical reintegration; by project-boundary analysis, physical actuation also remains external | O | C, E | Server contracts stop at the external information/control boundary | 031–043 and final synthesis (scope guardrail) |

Tasking terminology requires care. Volume I describes determining whether a request can be supported; it does not name a standardized **Feasibility** operation. It says a request may be accepted, scheduled, executed, or otherwise acted upon; it does not supply `rejected` and `cancelled` as formal lifecycle states. Those exact operations and states require separate CSAPI Part 2 evidence in later research.

### 5.3 Contract Boundary Rules

Later architecture should apply these rules:

1. The server is authoritative for the behavior of the API it exposes, not automatically for every fact represented through that API.
2. Incoming metadata, observations, status, and task results retain source identity and provenance.
3. A command accepted by the server is not the same as a physical action completed by a device.
4. An authorization decision enforced by the server depends on identity, attributes, policy, and accreditation supplied or governed externally.
5. A linked or federated resource is not proof that the server owns the remote service, gateway, or global identifier regime.
6. Dynamic resource semantics must remain separable from a broker or pub/sub transport selection.
7. DDIL awareness means explicit state, time, freshness, and reconciliation boundaries; it does not mean the server can cure an unavailable network.

### 5.4 Lightweight OGC Standards-Package Alignment

This is a fit check, not normative requirement extraction.

| Volume I Function | Likely Adopted-Package Realization | Important Gap or Boundary |
|---|---|---|
| Discovery | CSAPI Part 1 collections/endpoints/paging/search/links and system, deployment, procedure, sampling-feature, and property resources `[CSAPI-1, §§7.5–7.9, 8.2–8.5, 9–16]`; SensorML descriptive metadata `[SensorML, §1, §§8.2.2.2–8.2.2.6, 8.2.4]`; CSAPI Part 2 datastream/observation discovery `[CSAPI-2, §§9, 13]` | No federation query protocol, catalogue aggregation, replication, or discovery-policy profile. CSAPI Part 1 requires UID values to be URIs and unique across one server's collections; it does not establish a global allocation authority. |
| Registration and description | CSAPI Part 1 resource identity, resources, encodings, and transaction/update classes `[CSAPI-1, §§8.4–8.5, 9–19]`; SensorML descriptive metadata, history, interfaces/configuration, and deployments `[SensorML, §§8.2.2, 8.2.8–8.2.9, 8.9]`; SWE Common components/encodings `[SWE-Common, §§7–10]` | Metadata authority, identifier allocation, disconnected reconciliation, and conflict rules remain outside. Relevant Part 1 and Part 2 transaction/update classes have the draft Part 4 dependency noted below. |
| Access and exchange | CSAPI Part 1 addressable retrieval, paging/filtering/navigation, and encodings `[CSAPI-1, §§7.6–7.9, 16, 19]`; CSAPI Part 2 live and/or archived observation access, temporal filters, and encodings `[CSAPI-2, §§9, 13, 16]`; SensorML supplies descriptive/process/lineage context `[SensorML, §1]`; SWE Common supplies data components and encodings `[SWE-Common, §1, §§7.2–7.6, 10]` | No complete profile for all variable/resolution/large-product selection or modality-specific formats. A service may support live data only, archived data only, or both. |
| Streaming and dynamic data | CSAPI Part 2 datastream, observation, control-stream, command, event, temporal-selection, and dynamic-encoding semantics `[CSAPI-2, §1, §§9–10, 12–13, 16]`; SWE Common stream structures and encodings `[SWE-Common, §1, §§7.5–7.6, 10]` | CSAPI Part 2 §1 delegates pub/sub bindings to Part 3; replay/resume, delivery guarantees, late-arrival reconciliation, offline queues, and mission priority remain open. |
| Tasking and control | CSAPI Part 1 controllable properties `[CSAPI-1, §15]`; CSAPI Part 2 control streams, commands, command status/results, and feasibility `[CSAPI-2, §§10–11]`; SensorML capability/interface descriptions `[SensorML, §§8.2.4, 8.2.9.1]`; SWE Common parameter structures and encodings `[SWE-Common, §§7.2.6, 7.3, 8.2–8.5, 10]` | Physical execution, safety interlocks, resource-owner authority, local autonomy, policy adjudication, and audit profile remain external/later. |
| Status and availability | CSAPI Part 2 status DataStreams, live/archive availability metadata, command lifecycle status, system events, and latest-result-time selection `[CSAPI-2, §§9.2.1–9.2.2, 10.11, 12, 13.3]`; SensorML validity/capability/history constructs `[SensorML, §§4.34, 8.2.2.6, 8.2.4, 8.2.8]`; generic SWE Common components can carry status/quality `[SWE-Common, §§7.3–7.5]` | `live` is metadata, not a readiness model or query filter; `CommandStatus` is command lifecycle, not general readiness. No complete NATO readiness/capacity/maintenance/duty-cycle vocabulary or last-known/freshness policy exists here. CSAPI Part 2 §12.2.1 Table 20 retains provisional `x-OGC/TBD` event-type URIs. |
| Provenance, validity, freshness, trust | SensorML lineage, validity, and history `[SensorML, §1, §§8.2.2.6, 8.2.8]`; SWE Common semantics and per-value quality `[SWE-Common, §§7.3–7.4]`; CSAPI links and temporal properties support associations/context `[CSAPI-1, §§7.9, 8.7; CSAPI-2, §§9.2.1–9.2.2, 13.3]` | Structural support is not trust adjudication, signature, reputation, freshness policy, or semantic-registry governance. SWE Common §7.3.1 says acquisition lineage must be described by other means. |
| Security and authorization | CSAPI Part 1 security guidance and compatibility with standard web-security mechanisms `[CSAPI-1, §7.10]`; Part 2 inherits Part 1 security considerations `[CSAPI-2, Security Considerations]`; SensorML permits externally defined security markings `[SensorML, §8.2.2.5 and Security Considerations]` | CSAPI Part 1 mandates no authentication method. SensorML supplies marking containers, not enforcement. SWE Common's Security Considerations define no security solution. The package lacks a mandatory NATO authorization/releasability/federation/cross-domain/audit profile. |
| Federation and cross-boundary sharing | Canonical links, URI-form UIDs, and resource relationships are enabling primitives `[CSAPI-1, §§7.6, 7.9, 8.5]`; SensorML can carry externally defined markings `[SensorML, §8.2.2.5]` | These do not define inter-server routing/query, release decisions, replication consistency, trust negotiation, link integrity, or global identifier governance. |
| DDIL / constrained operation | Stable identity and valid time `[CSAPI-1, §§8.5, 8.7]`; dynamic live/archive resources, asynchronous command state, temporal selection, and compact binary encoding `[CSAPI-2, §§9–13, 16.4]`; SensorML validity/history `[SensorML, §§8.2.2.6, 8.2.8]`; SWE Common text/binary options `[SWE-Common, §§7.5–7.6, 10]` | These are useful primitives, not a DDIL synchronization protocol. No offline queue, delta sync, replay/resumption, conflict resolution, priority, cache-expiry, or disconnected-authorization protocol is defined. |

The current approved publications for all four adopted standards were published 16 July 2025. CSAPI headers display version `1.0` while some OGC/AEP listings use `1.0.0`; SWE Common similarly appears as `3.0.0` or `3.0`. These refer to the same 16 July 2025 editions, not conflicting standards.

Relevant CSAPI Part 1 transaction classes `[CSAPI-1, §3, §§17–18]` and Part 2 create/update/delete classes `[CSAPI-2, §3, §§14–15]` depend on OGC API - Features Part 4, which the official [OGC API - Features page](https://ogcapi.ogc.org/features/) continued to identify as draft on 2026-07-30.

For pub/sub, normative CSAPI Part 2 §1 controls the boundary: protocol bindings are assigned to Part 3. Broad streaming language on an overview page and an `asyncapi/` directory in official artifacts do not make Part 3 an approved or AEP-adopted standard. The [pinned Part 3 working material](https://github.com/opengeospatial/ogcapi-connected-systems/tree/a1f1f03b71f5f645486b23ec8b5fae1f9ba334bc/api/part3/standard) is informative only.

Official schema and OpenAPI/AsyncAPI artifacts support implementation but do not independently add standards obligations. Development repositories and draft material are informative only.

### 5.5 Recommended Traceability Record

Each downstream decision should preserve these fields:

| Field | Purpose |
|---|---|
| Functional Mapping ID | Stable link to this report's baseline |
| AEP Source Anchor | Section plus controlled PDF and publication page |
| Source Wording Strength | `shall`, `should`, `may`, strong functional constraint, objective, example, or analysis |
| Actor / Trigger / Input / Outcome | Prevent ambiguous requirement translation |
| Context to Preserve | Identity, links, semantics, provenance, time, validity, freshness, quality, policy, etc. |
| Responsibility Class | Server, contract, ecosystem, future/adjacent, or out of scope |
| Contract Counterparty | Generic external role where applicable |
| OGC Mapping | Standard, clause, conformance class, and later exact requirement URI |
| Profile or Design Gap | Decision still required |
| Downstream Topic IDs | Assigned research owners |
| Rust Component | Added only after architecture exists |
| Verification Link | Automated test or conformance evidence added later |
| Status / Owner / Decision Date | Governance and change control |

The AEP source anchor and OGC requirement URI must remain separate. The former explains operational intent; the latter will define the precise testable technical obligation.

---

## 6. Key Recommendations

### Recommendation 1 - Adopt the Layered Responsibility Baseline

Use Section 5.2 as the controlling Volume I map for later IDR work. Do not flatten shared outcomes into an all-server or all-external answer.

### Recommendation 2 - Make Context a First-Class Server Concern

Identity, links, provenance, source, time, validity, freshness, quality, semantics, authorization, and handling are not optional metadata decoration. Later domain, storage, API, and test designs should show how each is preserved or intentionally profiled.

### Recommendation 3 - Treat DDIL as Semantics Before Infrastructure

Model current, stale, historical, last-known, pending, delayed, resumed, and reconciled states before selecting queues, brokers, caches, or synchronization technology. The model must make degraded operation understandable even when the chosen infrastructure changes.

### Recommendation 4 - Separate API Acceptance from External Completion

For publication, ingestion, tasking, and synchronization, distinguish:

- request received;
- syntactically/semantically validated;
- authorized;
- accepted for processing;
- delivered or delegated;
- externally executed;
- result/status received; and
- reconciled into server state.

Exact lifecycle vocabularies remain later decisions, but the boundary must not collapse these meanings.

### Recommendation 5 - Build an Explicit Glaux Conformance Profile

Later OGC extraction should select and justify the classes Glaux Server will implement, map each requirement URI to Rust code and tests, and identify any extension separately. Volume I supports a coherent full server capability; it does not itself select every optional technical class.

### Recommendation 6 - Define Contracts Without Designing Other Products

Later server architecture should publish clear contracts for metadata/data producers, task executors, identity/policy services, federation/gateways, and dynamic/synchronization infrastructure. Those contracts are Glaux Server scope; the internal implementation of the counterpart is not.

### Recommendation 7 - Maintain a Visible Gap Register

Keep transaction dependencies, pub/sub binding, security/releasability profile, status vocabulary, semantic governance, and DDIL synchronization/conflict decisions visible until the assigned topics resolve them. Do not let an AI implementation silently invent answers.

### Recommendation 8 - Retain Goal and Definition v1.5

No wording change is needed based on this topic. The existing goal already captures the server's full functional ambition, Rust implementation language, open-source character, standards package, ecosystem contracts, and DDIL-informed planning requirement.

---

## 7. Implementation Implications and Test Strategy

### 7.1 Architecture Implications

This report does not choose an architecture, but it establishes conditions later architecture must satisfy:

| Design Area | Required Consideration |
|---|---|
| API and protocol boundary | Standards-correct discovery, resource navigation, query, dynamic data, tasking, status, errors, authorization, and accurate conformance claims |
| Domain model | Persistent identity, typed relationships, capabilities/limitations, temporal validity, provenance, quality, policy context, and lifecycle state |
| Storage and query | Current and historical views, source context, validity/freshness, updates, last-known state, and later reconciliation without semantic loss |
| Ingestion / write boundary | Validation, authorization, idempotency/correlation, source authority, processing outcomes, and conflict visibility |
| Tasking boundary | Separate API command lifecycle from delivery, feasibility, execution, safety decision, and physical result |
| Security boundary | Policy-decision integration and consistent enforcement across resource, field/representation, query, and command surfaces as later profiles require |
| Integration adapters | Explicit contracts for publishers/producers, connected-system proxies, policy services, gateways, and transports |
| Observability and accountability | Sufficient evidence to explain processing, authorization, lifecycle, stale state, and failures; precise audit scope belongs to IDR-SRV-041 |
| Conformance and testing | Traceability from exact OGC requirement URI and Glaux profile decision to automated Rust tests |

### 7.2 Estimation Boundary

A defensible implementation estimate cannot be derived from Volume I alone. Estimating now would hide unresolved choices about OGC conformance classes, write behavior, persistence, security, transport, synchronization, and deployment. Estimates should be produced only after the relevant technical and architecture topics resolve those variables. The approximately 0.75-hour figure in the header is research execution time, not a software estimate.

### 7.3 High-Level Test-Strategy Implications

| Scenario Family | Evidence the Future Server Must Produce | Primary Handoff |
|---|---|---|
| Discovery and description | Authorized clients find and navigate resources with stable IDs, links, capabilities, constraints, validity, and machine-readable context | 050, 051, 053, 056 |
| Access and exchange | Selection/filtering returns standards-valid data while preserving source, semantics, relationships, time, provenance, and policy | 050–053, 056 |
| Dynamic data | Current/historical/event flows preserve time and ordering rules, expose freshness, and handle late, stale, interrupted, or resumed input according to the selected profile | 051, 052, 054 |
| Tasking | Feasibility, authorization, lifecycle, delayed confirmation, result correlation, failure, and prohibited action are distinguishable | 051–053, 055 |
| Status and availability | Current, stale, last-known, degraded, unavailable, and historical state are testably distinguishable | 051–054 |
| Security and federation | Unauthorized discovery/read/write/tasking is prevented; policy context and cross-boundary behavior are testable without assuming a particular profile prematurely | 051, 052, 055, 056 |
| DDIL / synchronization | Delayed or interrupted updates preserve context, expose conflict/staleness, and converge or fail visibly under the later selected rules | 051–054 |
| Interoperability | External CSAPI clients can use the claimed representations and behavior without Glaux-specific knowledge, except for documented profiles/extensions | 050, 053, 056 |

Each later test must cite an exact OGC requirement URI or an explicit Glaux project/profile requirement. An AEP function alone is insufficient as the only pass/fail oracle for detailed protocol behavior.

---

## 8. Risks, Constraints, and Open Questions

### 8.1 Principal Risks

| Risk | Consequence | Control |
|---|---|---|
| Converting operational prose directly into invented API requirements | Non-standard behavior presented as NATO obligation | Preserve strength labels and require later OGC/profile evidence |
| Implementing only obvious OGC resource endpoints | A compliant-looking but operationally weak demonstration server | Retain cross-cutting context, policy, lifecycle, and DDIL handoffs |
| Treating every external dependency as server scope | Unbounded architecture and accidental redesign of the ecosystem | Apply the layered classification and contract rules |
| Treating external dependencies as “not our problem” | Missing interfaces make the server unusable in real workflows | Specify the server side of every required contract |
| Selecting draft technology as if adopted | False conformance claims and churn | Record draft status; isolate experimental work from the approved baseline |
| Collapsing API acceptance into physical completion | Unsafe or misleading tasking state | Maintain distinct lifecycle boundaries |
| Hiding stale or last-known information as current | Operational misuse | Make time, validity, freshness, and source state explicit |
| Overlong reports obscure decisions | Project lead cannot use the research | Lead with the decision brief; keep detailed evidence in matrices and appendices |

### 8.2 Constraints

- The controlled package may not be redistributed publicly.
- Volume I is functional and architectural guidance, not a detailed server specification.
- This topic may not pre-empt the exact technical extraction assigned to later topics.
- The full server capability can be sequenced, but later sequencing must not be presented as a reduction of the project goal.
- No current Glaux Server implementation exists to compare with the baseline.

### 8.3 Open Questions and Assigned Resolution

| Open Question | Why It Remains Open | Assigned Topic(s) |
|---|---|---|
| Which exact OGC conformance classes will Glaux claim? | Volume I does not select them | 003, 006, 007 |
| How will relevant writable Part 1 and Part 2 resources be implemented while OGC API - Features Part 4 remains draft? | Current standards dependency/status issue | 003, 006–007, 031 |
| Which pub/sub binding and delivery guarantees will be supported? | Approved Part 2 delegates binding to a non-adopted Part 3 | 007, 035 |
| What is the canonical status/readiness vocabulary? | Volume I gives concepts; package lacks complete NATO vocabulary | 004, 020, 024 |
| What are Glaux freshness, stale, last-known, and validity rules? | Volume I requires distinction but not thresholds/algorithms | 018, 020, 034, 042 |
| Which authentication, authorization, releasability, marking, federation, and cross-domain profiles apply? | Core OGC package does not mandate them | 039–041 |
| What synchronization, delta, resumption, and conflict behavior is server scope? | Volume I gives outcome, not mechanism | 029, 042, 043 |
| What audit/accountability events and retention are required? | Accountability is a project/governance implication, not a detailed Volume I schema | 030, 038, 041 |
| Which semantic registries and controlled vocabularies are authoritative? | Volume I and core encodings support semantics but do not govern all terms | 004, 024 |
| Does later AEP/SRD/profile material add requirements? | Future material is not current evidence | Reassess only when authoritative material exists and the project assigns it |

None of these questions blocks acceptance of this report as a functional responsibility baseline.

---

## 9. Validation Against the Research Plan

### 9.1 Methodology Phase Validation

| Plan Phase | Work Performed | Result |
|---|---|---|
| 1. Source collection and orientation | Located fixed package, verified hash, recorded status/access, reviewed all Volume I structure and IDR-SRV-001 constraints | Complete |
| 2. Functional extraction | Reviewed all pages; extracted named and implied functions, repeated concerns, environment/actor constraints, and source anchors | Complete |
| 3. Function-to-server mapping | Classified 56 report-local findings across direct server, contract, ecosystem, future/adjacent, and out-of-scope categories | Complete |
| 4. Handoff and traceability | Mapped findings to downstream plans, test topics, a reusable record structure, terms, and open issues | Complete |
| 5. Standards-package alignment | Checked high-level fit against current approved CSAPI Parts 1/2, SensorML, and SWE Common; recorded gaps without exhaustive extraction | Complete |
| 6. Synthesis | Produced decision brief, evidence record, responsibility baseline, recommendations, risks, validation, and appendices | Complete |

### 9.2 Success-Criterion Validation

| Plan Success Criterion | Evidence | Status |
|---|---|---|
| Controlling AEP-I reviewed | Sections 3.1–3.2 and 9.1 | Met |
| Sources list title, version/date, location/URL, status, and authority | Section 3.1 | Met |
| Functions extracted with anchors | Sections 4.1 and 5.2 | Met |
| Every relevant function classified | Section 5.2 | Met |
| All named areas plus security/federation/DDIL assessed | Sections 4.1–4.4 and 5.2 | Met |
| Downstream handoffs identified | Sections 4.5 and 10.2 | Met |
| Test implications captured | Section 7.3 | Met |
| Boundary questions and interpretation risks listed | Section 8 | Met |
| Recommendations are decision-usable and server-bounded | Sections 1.3 and 6 | Met |
| References are explicit and reproducible | Sections 3, 11, and 12.2 | Met, subject to authorized access for the controlled package |

### 9.3 Deliverable-Content Validation

| Required Content | Report Location | Status |
|---|---|---|
| Executive summary | Section 1 | Complete |
| Scope and plan alignment | Section 2 | Complete |
| Evidence base and authority classification | Section 3 | Complete |
| Volume I functional extraction findings | Sections 4.1 and 12.2 | Complete |
| Function-to-server responsibility matrix | Section 5.2 | Complete |
| Boundary classification findings | Sections 4.3–4.4 and 5.3 | Complete |
| Standards-package alignment notes | Section 5.4 | Complete |
| Downstream handoff matrix | Sections 4.5 and 10.2 | Complete |
| Test-strategy implications | Section 7.3 | Complete |
| Recommendations | Section 6 | Complete |
| Risks, constraints, and open questions | Section 8 | Complete |
| Validation against success criteria | Section 9.2 | Complete |
| References | Section 11 | Complete |

---

## 10. Next Steps and Handoff

### 10.1 Acceptance Required

The project lead should decide whether:

1. the layered responsibility model is understandable and useful;
2. Section 5.2 is an acceptable controlling Volume I baseline for downstream research; and
3. the unresolved matters are assigned to the correct later topics.

Until that review occurs:

- report status remains **In Review**;
- the topic plan remains **In Progress**;
- `Accepted By` and `Acceptance Date` remain pending; and
- later topics may read the report but must not represent it as plan-owner accepted.

No other substantive project decision is required to complete this topic.

### 10.2 Downstream Topic Handoff Matrix

Each handoff is owned by the researcher executing the named future topic. Work begins only after IDR-SRV-002 is accepted and the topic becomes eligible under the overall-plan sequence; this report does not start that work.

| Topic(s) | Required Input from IDR-SRV-002 |
|---|---|
| 003 | Treat the four standards as a coherent package; resolve package-level fit and gaps, including draft dependencies |
| 004 | Crosswalk the terms in Section 10.3 and distinguish operational concepts from OGC resources |
| 006–007 | Extract exact normative requirements and classes that realize each mapping; preserve AEP and OGC anchors separately |
| 009–014 | Resolve concrete HTTP/API behavior; do not claim Volume I itself mandates a landing page, exact link, filter, media type, error, or OpenAPI design |
| 015–020 | Model identity, relationships, context, time, validity, freshness, provenance, quality, trust, status, events, current/stale/last-known state |
| 021–024 | Allocate description/encoding roles among SensorML/SWE Common and resolve SI units, validation, observed/controlled properties, semantic bindings, and vocabularies |
| 025–030 | Preserve authoritative source, history, lifecycle, update, retention, consistency, idempotency, and reconciliation inputs |
| 031–033 | Define server-facing write contracts without designing Publisher or Simulator internals |
| 034–035 | Define dynamic update semantics separately from transport/broker selection; address ordering, late arrival, replay/resume, freshness, and interruption |
| 036–038 | Separate API acceptance, feasibility, delivery, execution, result, authorization, safety, and accountability |
| 039–043 | Select security/policy profiles and define DDIL, synchronization, conflict, and cross-boundary server boundaries |
| 050–056 | Use the traceability record and scenario families to create conformance, Rust, fixture, performance, security, and external-client tests |

### 10.3 Terminology Handoff to IDR-SRV-004

At minimum, crosswalk:

- connected system, sensing system, sensing asset, sensing resource, sensor, platform, process, related software service, and aggregated/virtual source;
- observation, sensor-derived information, detection, track, alert, and event;
- common intelligence feed, common intelligence picture, common operational picture, and common tactical picture;
- system, procedure, deployment, sampling feature, feature/object/matter of interest;
- capability, limitation, controllable function/property, tasking, control, command, feasibility, and command status;
- information discovery, registration/description, access/exchange, data stream, streaming, and dynamic data;
- status, availability, health, active/inactive/degraded, current, stale, last-known, validity, and freshness;
- identity, designation, responsible organization, provenance, lineage, source, method, quality, suitability, and trust;
- security, authorization, releasability, handling, accreditation, federation, cross-boundary, and policy-driven gateway;
- DDIL-informed, constrained exchange, staged delivery, progressive enrichment, change-aware exchange, synchronization, and reconciliation; and
- reference view, reference architecture, interoperability framework, sensor integration, and NIIA linkage.

---

## 11. References

### 11.1 Controlled NATO Source

- NATO Consultation, Command and Control Board Joint Capability Group Intelligence, Surveillance and Reconnaissance, `AC/224(JCGISR)D(2026)0005`, 27 April 2026. Project-controlled ratification package, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`.
  - Enclosure 1: *STANAG 4789, Sensor Integration Standard for NATO JISR Operations*, Edition 1, PDF pp. 3–7.
  - Enclosure 2: *AEP-4789 Volume I, Sensor Integration Standard for NATO JISR Operations - Reference View*, Edition A, Version 1, PDF pp. 8–37.
  - Enclosure 3: *AEP-4789 Volume II, Sensor Integration Standard for NATO JISR Operations - Core APIs and Encodings*, Edition A, Version 1, PDF pp. 38–59.

The controlled file is intentionally not copied into or linked from the public repository.

### 11.2 Approved OGC Standards and Official Supporting Artifacts

- Open Geospatial Consortium, [OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html), approved and published 2025-07-16. Accessed 2026-07-30.
- Open Geospatial Consortium, [OGC 23-002, *OGC API - Connected Systems - Part 2: Dynamic Data*, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html), approved and published 2025-07-16. Accessed 2026-07-30.
- Open Geospatial Consortium, [OGC 23-000, *OGC SensorML Encoding Standard*, Version 3.0](https://docs.ogc.org/is/23-000/23-000.html), approved and published 2025-07-16. Accessed 2026-07-30.
- Open Geospatial Consortium, [OGC 24-014, *OGC SWE Common Data Model Encoding Standard*, Version 3.0.0](https://docs.ogc.org/is/24-014/24-014.html), approved and published 2025-07-16. Accessed 2026-07-30.
- [OGC API - Connected Systems developer page](https://ogcapi.ogc.org/connectedsystems/). Accessed 2026-07-30.
- [Official OGC API - Connected Systems Part 1 schemas and API artifacts](https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/). Accessed 2026-07-30.
- [Official OGC API - Connected Systems Part 2 schemas and API artifacts](https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/). Accessed 2026-07-30.
- [Official SensorML 3.0 JSON schemas](https://schemas.opengis.net/sensorML/3.0/json/). Accessed 2026-07-30.
- [Official SWE Common 3.0 JSON schemas](https://schemas.opengis.net/sweCommon/3.0/json/). Accessed 2026-07-30.
- [OGC API - Features official page](https://ogcapi.ogc.org/features/), including Part 4 draft status. Accessed 2026-07-30.
- [OGC API - Connected Systems Part 3 working material at commit `a1f1f03b71f5f645486b23ec8b5fae1f9ba334bc`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/a1f1f03b71f5f645486b23ec8b5fae1f9ba334bc/api/part3/standard). Informative draft only; accessed 2026-07-30.

### 11.3 Project Governance and Context

- [IDR-SRV-002 Research Plan](../IDR%20Plans/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities.md)
- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [Accepted IDR-SRV-001 Report](idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md)
- [Glaux Server Goal and Definition v1.5](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research Planning Approach](../../../../../Governance/research-planning-approach.md)
- [Research Plan Template](../../../../../Governance/research-plan-template.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
- [Overall Research Report Template](../../../../../Governance/overall-research-report-template.md)
- [Glaux Server repository at commit `1ba41159d1465797f1fceab129486197eb80aadf`](https://github.com/DGIWG-P507/glaux-server/tree/1ba41159d1465797f1fceab129486197eb80aadf)
- [OS4CSAPI exemplar corpus at commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans)

---

## 12. Appendices

### 12.1 Complete Research Question Coverage Matrix

Each answer below is deliberately concise; the evidence column points to the full analysis.

| ID | Detailed Question (Short Form) | Answer | Evidence |
|---|---|---|---|
| CQ-1 | Relevant functional areas | Six named functions plus context/trust, security, federation, and DDIL concerns | 4.1 |
| CQ-2 | Direct server responsibilities | Behavior the server exposes, preserves, governs, or verifies | 4.2, 5.2 |
| CQ-3 | Required server-side contracts | Publication/ingestion, execution, identity/policy, federation/gateway, and transport/sync contracts | 4.3, 5.3 |
| CQ-4 | Ecosystem/profile/out-of-scope functions | Classified explicitly without shrinking server-facing contracts | 4.4, 5.2 |
| CQ-5 | Downstream handoffs | Assigned across standards, models, contracts, security/DDIL, and tests | 4.5, 10.2 |
| FI-1 | How Volume I organizes functions | Problem and reference view in Chapters 1–2; six-function view in Chapter 3; standards rationale in Chapter 4 | 3.2, 4.1 |
| FI-2 | Explicit and implied functions | Explicit six plus implied identity/context, policy, federation, change, and constrained-operation support | 4.1, 5.2 |
| FI-3 | Repeated functions | Identity/links, provenance, semantics, time/validity/freshness, authorization, and delayed exchange recur | 4.1 |
| FI-4 | Dependencies on metadata/dynamic/tasking/events/security/status | All six depend on one or more; dependencies are mapped explicitly | 4.1 dependency crosswalk; 5.2 |
| FI-5 | Terminology needing crosswalk | Terms are enumerated for IDR-SRV-004 | 10.3 |
| DIS-1 | What must be discoverable | Information, observations, histories, systems, capabilities, limitations, coverage, deployment, identity, organization, and validity; exact CSAPI resource mapping later | 4.1, DISC-01–04 |
| DIS-2 | Direct discovery responsibilities | Stable resources, links, indexed description, authorized search/query, consistent behavior | 4.2, DISC-01–05 |
| DIS-3 | Client/publisher/catalog/ecosystem discovery | Server exposes contract; metadata origin, client presentation, federation aggregation, and global governance remain external/contracts | 4.3–4.4 |
| DIS-4 | Landing/navigation/link/query implications | Addressability and meaningful navigation are implied; exact landing page, links, collections, and queries are OGC/later-profile obligations, not Volume I prescriptions | 4.2, 5.4, 10.2 |
| DIS-5 | Part 1 handoff | Carry identity, resources, relationships, discoverability, description, coverage, validity, and filtering needs into exact extraction | 4.5, 10.2 |
| REG-1 | Registration/description subjects | Systems, sensing assets, procedures/capabilities, deployments, outputs, properties, provenance, security, and temporal context; OGC resource names require later mapping | REG-01–04 |
| REG-2 | What server stores/exposes/validates/links/governs | Standards-facing representations, identity, relationships, context, validity, policy hooks, and contract integrity | 4.2, REG-01–05 |
| REG-3 | What remains external workflow | Authoritative fact creation, sensor setup, operator approval, and other-product internals | 4.3–4.4 |
| REG-4 | SensorML/SWE/semantic/provenance/validity/lineage implications | Structural and description fit is strong; authority, policy, and vocabularies remain external/later | 5.4 |
| REG-5 | Resource/metadata handoff | Canonical identity/model, documents, temporal context, provenance, quality, and lifecycle go to 015–019 and 021–024 | 4.5, 10.2 |
| ACC-1 | Access/exchange functions | Discrete and broader sharing, selective retrieval, structured contextual exchange, and staged/asynchronous transfer | 4.1, ACCESS-01–05 |
| ACC-2 | Retrieval/dynamic/history/negotiation/filter/validation responsibilities | Retrieval, history, context, and validation are direct; exact negotiation/filter/interface requirements come from later OGC extraction | 4.2, 5.4 |
| ACC-3 | Part 1/Part 2 dependency | Static/feature resources align mainly to Part 1; observations/dynamic interaction mainly to Part 2; SensorML/SWE preserve meaning | 5.4 |
| ACC-4 | External-system contracts | Metadata/data producers, clients, gateways, and transports interact through explicit server contracts; their internals are not designed | 4.3 |
| ACC-5 | Interoperability risks | Bespoke behavior, context loss, false conformance, draft dependency, and unprofiled selection/security are carried forward | 8.1, 10.2 |
| DYN-1 | Streaming/near-real-time/update/event/subscription implications | Continuous/repeated/event information and changing status are in scope; Volume I does not select a subscription protocol | DYN-01–04 |
| DYN-2 | Direct dynamic responsibilities | Resource semantics, current/historical views, time, sequence/change, currentness, and lifecycle state | 4.2, DYN-01–02 |
| DYN-3 | Broker/pub-sub/edge boundary | Server-side semantics and contracts are in scope; broker, binding, link, and edge implementation may be external/later | 4.3, DYN-03–04 |
| DYN-4 | Freshness/order/replay/late/stale semantics | Time, sequence, freshness, and stale state must be explicit; exact replay, late-arrival, and ordering policies remain later decisions | 5.2, 8.3 |
| DYN-5 | Handoff to 034/035/performance | Dynamic model goes to 034; binding/transport to 035; interruption/load scenarios to 054 | 7.3, 10.2 |
| TASK-1 | Tasking/control implications | Authorized requests, parameters, modes, feasibility, action status, policy, autonomy, and delayed confirmation | TASK-01–05 |
| TASK-2 | Direct tasking responsibilities | Validate/authorize requests and expose capability, feasibility, correlation, lifecycle, status, and results | 4.2, TASK-01–04 |
| TASK-3 | Executor/actuator/mission-system contracts | Physical assessment/execution and authoritative status/results are external through contracts | 4.3, TASK-05 |
| TASK-4 | Safety/audit/authorization/accountability implications | Enforce authority and preserve explainable lifecycle; detailed safety/audit/profile is later and not claimed as explicit Volume I schema | 7.1, 8.3 |
| TASK-5 | Handoff to 036–038/055 | Lifecycle, feasibility, authorization, safety, delayed action, negative tests, and result correlation are assigned | 7.3, 10.2 |
| STAT-1 | Status/health/availability/degraded/last-known implications | Volume I explicitly covers active/inactive/degraded effects, availability/capacity/maintenance/duty cycle, time/validity, and last-known state | STAT-01–04 |
| STAT-2 | Direct status responsibilities | Expose and preserve typed state, source, update time, validity, freshness, current/stale/history | 4.2, STAT-01–04 |
| STAT-3 | Event-model need | Changes and operational events need a coherent model, but command status must not be confused with general readiness | 5.4 |
| STAT-4 | DDIL status semantics | Last-known, delayed, current/stale, validity, and later asynchronous reconciliation must be distinguishable | STAT-03–04, DDIL-04 |
| STAT-5 | Handoff to 020/034/042 | Vocabulary/model to 020, update semantics to 034, constrained-state rules to 042 | 4.5, 10.2 |
| SEC-1 | Auth/trust/federation/releasability dependencies | All functions require authorized, policy-aware use with security/handling context | SEC-01–03 |
| SEC-2 | Differently accredited/coalition/national environments | Server needs enforcement and gateway contracts; accreditation, release decisions, and gateway operation remain ecosystem/profile concerns | 4.3–4.4 |
| SEC-3 | DDIL/constrained functions | Local usefulness, payload context, staged exchange/enrichment, change awareness, freshness, delayed interaction, and later sync | DDIL-01–04 |
| SEC-4 | Access/audit/sync/conflict implications | Enforcement is direct; explainable records are a project implication; synchronization/conflict mechanisms are later contracts/design | 5.3, 7.1, 8.3 |
| SEC-5 | Handoff to 039–043 | Security/profile, policy, audit, DDIL, and synchronization/conflict questions are individually assigned | 10.2 |

### 12.2 Volume I Source-Anchor Index

| Source Area | Primary Findings Carried into This Report |
|---|---|
| Preface, PDF pp. 12–13 / publication pp. II–III | Scope, audience, reference-view character, exclusions, linkages, layered/modular framework |
| Chapter 1, PDF pp. 19–21 / publication pp. 1–3 | Heterogeneity, information problems, data-centric intent, seven information qualities, operational context |
| §§2.1–2.3, PDF pp. 22–24 / publication pp. 4–6 | Common conceptual basis, external interfaces, independent addressability, metadata/context, dynamic interaction, six core functions |
| §§2.4–2.4.2, PDF pp. 24–25 / publication pp. 6–7 | Cross-domain boundaries, DDIL constraints, local usefulness, staged exchange/enrichment, change, freshness |
| §3.1, PDF pp. 26–27 / publication pp. 8–9 | Discovery |
| §3.2, PDF p. 27 / publication p. 9 | Registration and description |
| §3.3, PDF pp. 27–28 / publication pp. 9–10 | Access and exchange |
| §3.4, PDF pp. 28–29 / publication pp. 10–11 | Streaming and dynamic data |
| §3.5, PDF p. 29 / publication p. 11 | Tasking and control |
| §3.6, PDF p. 30 / publication p. 12 | Status and availability |
| Chapter 4, PDF pp. 31–33 / publication pp. 13–15 | Open standards, coherent package, external security/trust/semantic needs, Volume II role, future material boundary |
| Lexicon, PDF pp. 34–36 / Lex-1–Lex-3 | Defined connected-system, observation, sensor-derived-information, reference-view, and interoperability terms |

### 12.3 Report Completion Checklist

- [x] Topic ID matches the overall research plan index
- [x] Topic research plan and overall plan are linked and aligned
- [x] Exactly one topic was executed
- [x] All 5 core and 40 detailed questions are answered
- [x] All six methodology phases were performed
- [x] All ten success criteria are validated
- [x] All thirteen required deliverable elements are present
- [x] The complete controlling Volume I was reviewed and its package hash verified
- [x] Normative, functional, informative, analytical, and recommended material are distinguished
- [x] Controlled-source and current-evidence limitations are explicit
- [x] Direct server, contract, ecosystem, future/adjacent, and out-of-scope responsibilities are separated
- [x] Current approved OGC sources are used only for lightweight alignment
- [x] No other Glaux product was designed
- [x] No other IDR research topic was begun
- [x] Findings were independently checked for coverage and overclaiming
- [ ] Plan-owner acceptance completed
- [ ] `Accepted By` and `Acceptance Date` recorded

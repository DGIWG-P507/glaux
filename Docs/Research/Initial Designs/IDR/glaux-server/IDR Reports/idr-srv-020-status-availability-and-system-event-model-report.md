# Section 020: Status, Availability, and System Event Model - Research Report

**Topic ID:** IDR-SRV-020<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-020 Status, Availability, and System Event Model](../IDR%20Plans/idr-srv-020-status-availability-and-system-event-model.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions, all detailed-question groups, all 6 methodology phases, and all 10 success criteria<br>
**Methodology Used:** Authority-ranked synthesis of accepted IDR-SRV-001 through IDR-SRV-019; direct extraction from approved CSAPI 1.0 prose, requirements, schemas, and OpenAPI artifacts; review of OGC API - Features, SensorML 3.0, SWE Common 3.0, SSN/SOSA, RFC 3339, RFC 9110, RFC 9111, RFC 8288, and RFC 9457; controlled AEP/STANAG carry-forward; and resource-family, state-projection, event-generation, degraded-operation, persistence, security, validation, fixture, and interoperability analysis<br>
**Research Time:** Approximately 8 hours of AI-assisted execution on September 13, 2026<br>
**Official Standards Source Pin:** [`opengeospatial/ogcapi-connected-systems` `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Shared Register Baseline:** OGC API - Connected Systems upstream-history register version 1.9; no tracked approved-standard or draft Part 3 state required a register change during this topic<br>
**Document Purpose:** Establish the encoding-neutral operational-status, capability-specific availability, current-state projection, System Event, degraded-operation, persistence, exposure, and validation baseline for the Rust Glaux reference server<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Pending Glaux Project Lead review<br>
**Acceptance Date:** Pending<br>
**Date:** September 13, 2026<br>
**Last Updated:** September 13, 2026

---

## Reading Guide and Evidence Labels

| Label | Meaning |
|---|---|
| **N** | Normative or standards-derived finding from an approved applicable source or incorporated artifact |
| **A** | Project-controlling AEP/STANAG adoption or operational-context finding carried from an accepted report |
| **P** | Glaux project decision or recommendation proposed for acceptance here |
| **I** | Informative implementation, test, interoperability, or community evidence |
| **D** | Official draft evidence useful for a future seam but not an approved requirement |
| **X** | Published ambiguity, evidence gap, or unresolved project decision |

The terms **state**, **status report**, **availability assessment**, and **event** are not synonyms. A state is true over an interval, a report asserts state at a time, an availability assessment evaluates evidence for a named capability and purpose, and an event records an occurrence. Likewise, **domain freshness** is not HTTP cache freshness.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Status/Event Extraction Methodology
5. Standards-Derived Status, Availability, and Event Inventory
6. Status, Availability, Health, Freshness, and Event Taxonomy
7. Resource-Family Status and Availability Model
8. Current-State, Historical-State, Dynamic-Property, Observation-Derived, and Event Findings
9. System Event Categories, Triggers, Metadata, Links, and Retention
10. Command Status, Feasibility, and Control-Stream Availability
11. Freshness, Staleness, Degraded Operation, Cache, and DDIL
12. Persistence, Indexing, Query, Validation, and Fixtures
13. Security, Policy, Releasability, and Audit
14. Conformance, Interoperability, and Test Implications
15. Downstream Topic Handoff Matrix
16. Recommendations
17. Risks, Constraints, and Open Questions
18. Validation Against Plan Success Criteria
19. References

---

## 1. Executive Summary

Glaux needs a **vector of independently evidenced states**, not one universal status field. Resource lifecycle, system operating mode, physical health, sensing readiness, source connectivity, DataStream live delivery, data freshness, ControlStream command acceptance, command feasibility, command execution, service health, authorization, and audit outcome answer different questions. A green value in one dimension must never imply green values in the others. [N,A,P]

The planning baseline is:

> **Represent source status as ordinary, semantically typed observations on `DataStream.type = status`; compute current views as request-scoped, policy-versioned projections over authorized evidence; represent availability as a capability-specific assessment; preserve command progress as ordered `CommandStatus` records; preserve operational history as durable `SystemEvent` resources; and keep resource lifecycle events, transport notifications, audit events, and internal service telemetry distinct.** [N,P]

CSAPI Part 2 supplies three important external primitives. First, a DataStream distinguishes `status` from `observation` and has a server-facing `live` indication. Second, a ControlStream has `live`, meaning whether its command channel can currently accept commands, and `async`, meaning how commands are processed. Third, `CommandStatus` supplies time-ordered status reports and `SystemEvent` supplies queryable operational/maintenance history for a System. These fields are narrow: `live` does not prove device health, data freshness, authorization, feasibility, or success of the next operation. [N]

Current system status should be a projection, not a mutable fact copied onto every resource. It identifies the exact status properties used, their source observations, observation/result times, request evaluation time, snapshot boundary, freshness-policy result, provenance, uncertainty/quality, and coverage. A last-known value remains the last-known value after disconnection; it may become stale or insufficient, but it does not become false. When evidence cannot distinguish failure from loss of contact, the correct availability state is `unknown`, not `unavailable`. [P]

Availability is always qualified by a capability and purpose. Glaux should use an internal or separately advertised profile assessment with at least `available`, `degraded`, `unavailable`, `unknown`, and `not_applicable`, plus controlled reason codes, evidence references, evaluation time, scope, policy/version, and recheck or validity information. `stale`, `disconnected`, `retired`, `disabled`, and similar concepts are evidence or reasons, not interchangeable top-level states. Policy denial remains an authorization outcome; it must not be recorded as physical unavailability, even when an HTTP concealment policy returns `404`. [N,P]

The approved Part 2 System Event class is narrower than a generic event bus. It covers events and maintenance operations on a System, with examples including calibration, configuration change, software update, part replacement, relocation, deployment, and decommissioning. Glaux should create a public System Event only for a genuine system-domain occurrence with an authorized, versioned event-type URI. Ordinary CRUD, every Observation arrival, every command-status report, service-health transitions, audit records, and Part 3 envelopes remain separate unless the same real-world occurrence independently warrants a System Event. [N,P]

There are two published System Event gaps that require an adapter decision later. The Part 2 conceptual model describes `name`, optional `description`, type URI, `eventTime`, and optional `message`; the tagged JSON schema instead composes the SensorML Event shape and requires `definition`, `label`, and `time`. The document's predefined event-type URIs also remain `x-OGC/TBD` placeholders. Glaux must not silently invent a mapping or claim stable OGC identifiers. It should preserve a canonical internal event and later select, document, advertise, and test an explicit wire overlay and a Glaux-controlled versioned vocabulary pending upstream resolution. [N,X,P]

The project lead is asked to accept these decisions:

1. adopt the taxonomy and non-collapse rules in §6;
2. adopt capability-specific availability and the five-state assessment algebra in §6.3;
3. adopt the resource-family behavior matrix in §7 and current-status projection contract in §8;
4. adopt the System Event boundary and generation matrix in §9;
5. preserve Part 2 command status codes and separate command acceptance, feasibility, execution, result, and control availability as specified in §10;
6. adopt the stale/last-known/unknown and DDIL rules in §11;
7. carry the persistence, security, validation, fixture, and test requirements in §§12–15 into their assigned downstream topics; and
8. authorize only IDR-SRV-021 after acceptance. This report does not implement the server, select a database or broker, finalize a public status profile, define retention periods, or authorize draft Part 3 implementation.

---

## 2. Scope and Plan Alignment

### 2.1 Completed Scope

This report:

- extracts applicable status, availability, live-stream, command-status, feasibility, System Event, dynamic-property, HTTP-health, cache, and semantic-component behavior;
- maps those concepts to every canonical resource and support family from IDR-SRV-015;
- defines current versus historical status, observation-derived projections, availability assessments, and event boundaries;
- establishes event categories, generation rules, required internal metadata, linking, query, correction, and retention implications;
- defines stale, last-known, degraded, disconnected, unavailable, and unknown behavior consistent with IDR-SRV-018;
- incorporates provenance, quality, source authority, and trust boundaries from IDR-SRV-019;
- records implementation evidence from IDR-SRV-014A through 014H as nonnormative evidence; and
- provides bounded handoffs to representation, persistence, ingestion, streaming, command, security, DDIL, conformance, fixture, performance, and interoperability topics.

### 2.2 Excluded Decisions

This report does not:

- design tables, indexes, retention durations, monitoring deployment, ingestion pipeline, broker binding, or synchronization protocol;
- define a final command state machine, command safety policy, authorization model, classification policy, or audit-storage policy;
- add fields to approved CSAPI representations or claim an extension conformance class;
- treat removed System History source material as approved CSAPI 1.0 behavior;
- adopt the draft Part 3 binding or equate a System Event resource with a CloudEvent or broker message; or
- redistribute, quote, or independently reinterpret controlled AEP/STANAG text.

### 2.3 Governing Prior Decisions

| Prior report | Controlling baseline used here |
|---|---|
| IDR-SRV-015 | Canonical resources, projections, append-oriented System Events and command-status records; System History is not an approved 1.0 class |
| IDR-SRV-016 | Stable ResourceId, revisions, replacement, lifecycle, deletion, tombstone, and event/message identity separation |
| IDR-SRV-017 | Typed authoritative relationships, derived links, integrity, policy-aware traversal, and redaction-safe resolution |
| IDR-SRV-018 | Separate valid/effective, transaction, domain, ingest, commit, publication, and evaluation clocks; request-scoped `current`; named `latest`; purpose-specific freshness; last-known semantics |
| IDR-SRV-019 | Atomic authoritative revision plus provenance/outbox seam; scoped quality; multidimensional trust; source/authority/integrity/freshness separation; redaction and inference controls |

No accepted baseline is reopened. [P]

### 2.4 Core Research-Question Coverage

| Plan question | Coverage | Evidence location |
|---|---|---|
| Q1. Required status, availability, health, freshness, and event concepts | Complete | §§5–7 |
| Q2. Authority and representation classification | Complete | §§3–5 and evidence labels |
| Q3. Current, historical, dynamic, observation, event, command, service, and telemetry distinctions | Complete | §§6–8,10–11 |
| Q4. Lifecycle, dynamic-data, command, and operational event generation | Complete | §9.2 |
| Q5. Persistence, streaming, DDIL, command, security, validation, conformance, fixture, and interoperability implications | Complete | §§11–15 |

Every detailed-question group is answered in the corresponding required-content section and mapped to downstream ownership in §15. No detailed question remains unacknowledged; the bounded choices listed in §17.2 require later representation, persistence, command, DDIL, or policy work rather than more IDR-SRV-020 scope.

---

## 3. Evidence Base and Authority Classification

### 3.1 Source Inventory

| Source | Version/pin | Role | Accessed | Authority and limitation |
|---|---|---|---|---|
| CSAPI Part 1 | OGC 23-001, Version 1.0 | Feature resources, dynamic-property snapshot behavior, SensorML representations | 2026-09-13 | Approved and controlling [N] |
| CSAPI Part 2 | OGC 23-002, Version 1.0 | DataStreams, Observations, ControlStreams, Commands, statuses, feasibility, System Events | 2026-09-13 | Approved and controlling; published prose/schema inconsistencies are [X] |
| Official CSAPI source | `v1.0.0`, commit `8e03b236...` | OpenAPI, JSON schemas, examples, source history | 2026-09-13 | Pin for reproducibility; removed/history artifacts do not override approved prose [N,X] |
| OGC API - Features Part 1 | OGC 17-069r4 | Collections, features, `datetime`, paging, links | 2026-09-13 | Inherited where CSAPI incorporates it [N] |
| SensorML 3.0 | OGC 23-000 | System/process metadata, capabilities, characteristics, modes, configuration, history Event model | 2026-09-13 | Descriptive/configurational evidence is not automatically current readiness [N] |
| SWE Common 3.0 | OGC 24-014 | Boolean, Category, Quantity, DataRecord, constraints, definitions and encodings | 2026-09-13 | Enables typed status values; supplies no universal status vocabulary [N] |
| SSN/SOSA | W3C/OGC recommendation and 2023 edition | Observation/actuation semantic cross-check | 2026-09-13 | Supporting semantic source; no new CSAPI wire obligation [N/I] |
| RFC 3339, 9110, 9111, 8288, 9457 | Published RFCs | Timestamp, HTTP availability/error, cache freshness, links, problem details | 2026-09-13 | Protocol controls; HTTP cache freshness is not domain freshness [N] |
| AEP/STANAG baseline | `AC/224(JCGISR)D(2026)0005`, 27 April 2026, SHA-256 `56DC757B...DA8C` | NATO adoption and operational context | Accepted carry-forward | Controlled project input used only through accepted findings [A] |
| IDR-SRV-001–019 | Accepted reports | Project decisions and source syntheses | 2026-09-13 | Project-controlling unless explicitly superseded [P] |
| IDR-SRV-014A–014H | Pinned implementation/community studies | Patterns, defects, tests, transport experience | 2026-09-13 | Nonnormative; never a conformance oracle [I,D] |

### 3.2 Controlled-Source Boundary

The controlled package identity remains the one recorded above. This study neither redistributed nor newly quoted it. Accepted IDR-SRV-001 through 003 establish the approved Part 1, Part 2, SensorML, and SWE Common adoption context and the need for discoverable, linked, trustworthy, secure, temporally qualified information under operational and DDIL conditions. Those accepted findings support explicit availability evidence, last-known context, source health, handling controls, and auditability; they do not define a new public status vocabulary or wire member. [A]

### 3.3 Authority Rules

1. Approved requirement language and incorporated schemas control standards claims; project profiles may constrain or extend but must be labeled. [N,P]
2. A schema/prose conflict remains an explicit mapping gap; neither side is silently discarded. [X]
3. SensorML capability, characteristic, mode, configuration, and history metadata describe what a process is, can do, or has experienced; they do not prove what it is doing or whether it is reachable now. [N]
4. A status Observation is evidence. A current projection, freshness result, availability assessment, or alert is a derived assertion with method, time, and provenance. [P]
5. Implementation code and tests establish feasibility and failure modes, not standards authority. [I]
6. Mutable draft Part 3 evidence informs the event-publication seam only; its binding and conformance posture remain deferred to IDR-SRV-035. [D]

---

## 4. Status/Event Extraction Methodology

### 4.1 Extraction Frame

Every finding was captured against these fields:

- resource family and concept;
- source and exact anchor;
- authority classification;
- current, historical, derived, or event classification;
- external CSAPI, profiled external, administrative, or internal exposure;
- valid/effective, domain, report, transaction, evaluation, publication, and delivery time;
- authoritative relationships and response links;
- event trigger and type vocabulary;
- availability purpose, evidence, state, reason, and freshness role;
- security, releasability, inference, and audit implications;
- persistence, indexing, validation, fixture, and test implications; and
- downstream owner and unresolved decision.

### 4.2 Analytical Tests

For each candidate “status” the study asked:

1. Is it a resource lifecycle fact, a domain measurement/assertion, a derived view, a service diagnostic, a command report, or an occurrence?
2. What subject and capability does it describe?
3. At what time was the underlying condition true, when was it reported, and when was it evaluated?
4. What evidence and policy produced the result, and can it be reconstructed?
5. Does loss of evidence prove failure, or only `unknown`?
6. Is the value authorized and releasable to this caller, including its links, counts, reasons, and timing?
7. Is the value defined by an approved representation, a separately advertised profile, or internal telemetry only?

### 4.3 Conflict and Non-Overstatement Controls

- The public Part 2 document, tagged schemas, and prior accepted issue analyses were compared field by field.
- “Current,” “latest,” “live,” “fresh,” “available,” and “healthy” were not treated as synonyms.
- Status/event evidence from OSH, CS-Go, pygeoapi, SECD, OS4CSAPI tests, and discussions was used only after ownership and version qualification.
- Absence from reviewed sources is reported as “not defined/found in the bounded evidence,” not universal impossibility.
- Proposed canonical concepts remain encoding-neutral and do not appear in a core response unless a later representation topic identifies an approved or advertised profile mapping.

---

## 5. Standards-Derived Status, Availability, and Event Inventory

| Source anchor | Extracted concept | Glaux interpretation | Class |
|---|---|---|---|
| CSAPI Part 1 §14.7 | Dynamic Sampling Feature properties are modeled as Observations; a `datetime` request can include the property snapshot valid at the requested time | Use Observation history and explicit time selection for dynamic properties; do not mutate descriptive metadata as the sole history | N |
| CSAPI Part 2 DataStream model/schema | `type` is `status` or `observation`; stream may expose real-time, archived, or both; `live` indicates current streaming | A status stream is ordinary typed dynamic data. `live` describes a delivery condition, not the System's overall state | N |
| Part 2 Observation model | Observations carry phenomenon/result time, observed property, procedure, feature context, result, and optional result quality | Status values may be observations; current/fresh/available remain derived unless explicitly observed | N |
| Part 2 ControlStream schema | `live` indicates whether the command channel can currently accept commands; `async` identifies asynchronous processing | Treat channel acceptance and processing style independently from authorization, feasibility, execution, and device health | N |
| Part 2 Command model | Command is associated with one ControlStream and exposes a server-derived current status and results/history through related resources | Preserve immutable intent and derive current status from ordered reports | N |
| Part 2 §10.11 and Table 14 | Status report contains `reportTime`, `statusCode`, optional progress, execution time, message, and new results; synchronous and asynchronous report cardinality differs | Use append-oriented reports, status-specific validation, and deterministic current selection | N |
| Part 2 §11 | Feasibility is a command submitted to a feasibility channel, synchronous or asynchronous, with its own status and result | A positive feasibility result is not command acceptance, authority, future availability, or execution success | N |
| Part 2 §12, Requirements 40–44 | System Event is associated with exactly one System and records operational/maintenance events | Durable external domain history, separate from logs, audit, CRUD events, status samples, and transport envelopes | N |
| Part 2 Requirement 62 | `eventType` filters System Event resources | Event type is queryable semantic vocabulary; filters must be authorized and consistently mapped | N |
| Part 2 System Event prose vs tagged schema | Prose uses name/type/eventTime; schema requires SensorML `definition`/`label`/`time` | Preserve the mismatch as an adapter decision and fixture family; no silent aliasing | N,X |
| Part 2 event-type table | Seven example/predefined categories use `http://www.opengis.net/def/x-OGC/TBD/...` | Semantics are useful, identifiers are not stable production vocabulary; recheck and profile | N,X |
| SensorML 3.0 §§8.2.8, 8.8 | History contains Events; modes/configuration can select settings and enable/disable inputs/outputs | Useful descriptive and operational context, but configuration and capability are not proof of current state | N |
| SensorML 3.0 definitions | Sensor-related data includes status, measurement quality, service quality, battery life | Supports status as data while preserving semantic definitions and context | N |
| SWE Common 3.0 §§7.2, 8.2–8.3 | Boolean, Category, Quantity, DataRecord, constraints, code spaces, and definitions represent values | Model status with the appropriate typed component; Category requires a code space or allowed-token constraint | N |
| OGC API - Features/CSAPI query inheritance | `datetime`, limit, paging, collection/item links | Query System Events and status observations through standard temporal and paging behavior where applicable | N |
| RFC 9110 §§10.2.3, 15.6.3–15.6.5 | `503` is temporary service inability and may carry `Retry-After`; `502`/`504` describe gateway/upstream failure | HTTP request outcomes describe the API path, not the represented System's health | N |
| RFC 9111 §4.2 | HTTP cache fresh/stale is calculated from response age and freshness lifetime; stale reuse is constrained | Never reuse HTTP caching vocabulary as the domain-status definition | N |
| RFC 9457 §§3–5 | Problem Details conveys HTTP-interface errors and must not leak implementation internals | Use stable problem types for availability-related request failures; sanitize reasons and internal topology | N |

The standards define useful primitives but no universal operational-readiness vocabulary, cross-resource availability aggregate, severity scale, or freshness threshold. Those are profile/policy decisions. [X]

---

## 6. Status, Availability, Health, Freshness, and Event Taxonomy

### 6.1 Non-Collapsible Concepts

| Concept | Subject and question | Canonical form | Not equivalent to |
|---|---|---|---|
| Resource lifecycle | Does this Glaux resource revision currently exist, and is it active, retired, deleted, or tombstoned? | IDR-SRV-016 lifecycle/revision facts | Physical health or source reachability |
| System operational status | What mode/condition does the real or virtual System report? | Status Observations and derived current view | Service health, lifecycle, or command status |
| System capability | What can the System/process support under described conditions? | SensorML capability/configuration metadata | Current readiness or authorization |
| Sensing availability | Can a named System/output serve the requested sensing purpose now? | Capability-specific derived assessment | DataStream `live` alone |
| DataStream live state | Is this channel currently streaming data? | Part 2 `live` field | Freshness, archive presence, or complete coverage |
| Observation freshness | Is evidence recent enough under a named policy and purpose? | Derived assessment from IDR-SRV-018 | HTTP cache freshness or source availability |
| Last-known value | Which authorized value is latest under a named time/order rule? | Derived projection with source and clocks | Fresh, current, or true now |
| Service health | Can this API instance/process accept and correctly handle requests? | Internal liveness/readiness/startup/dependency telemetry; bounded admin endpoint | System operational status |
| Ingestion/source health | Is a publisher, adapter, or upstream producing acceptable data? | Internal/admin telemetry and provenance evidence | Physical System failure |
| Control availability | Can a named ControlStream currently accept commands? | Part 2 `live`, qualified by request and policy context | Feasibility, authorization, or execution |
| Command feasibility | What would a feasibility evaluation conclude for specific proposed parameters? | Part 2 Feasibility command/status/result | Permission or reservation |
| Command status | What did the command processor report at `reportTime`? | Ordered `CommandStatus` records/current projection | Audit decision or physical effect |
| Command result | What result artifact became available? | Part 2 result resource/link | Successful execution by itself |
| System Event | What operational or maintenance occurrence affected a System? | Durable Part 2 System Event | CRUD event, status sample, audit, or message |
| Resource lifecycle event | What canonical resource mutation committed? | Internal domain/outbox event; possible Part 3 Resource Event later | System Event |
| Observation/data event | What data resource arrived/changed or became publishable? | Internal domain/outbox event; possible Part 3 data/event message | System Event by default |
| Audit event | Who attempted what action, under which policy, with what outcome? | Security/accountability record | Domain history or provenance activity |
| Transport event | What was published, delivered, retried, deduplicated, or replayed? | Message/outbox/delivery record | Underlying domain occurrence |

### 6.2 Status Dimensions

A System “status” presentation may need several independently sourced dimensions:

- operating mode or mission phase;
- health/condition and fault indicators;
- power, thermal, storage, or resource condition;
- connectivity and source reachability;
- sensing/output readiness and coverage;
- control-input readiness;
- data freshness and completeness;
- lifecycle/administrative enablement;
- deployment context; and
- policy visibility.

Glaux must not manufacture one overall color or scalar unless a named, versioned profile defines inputs, missing-evidence behavior, precedence, time window, scope, and policy. Even then, the contributing dimensions remain inspectable to an authorized caller. [P]

### 6.3 Capability-Specific Availability Assessment

The encoding-neutral assessment uses this state algebra:

| State | Meaning |
|---|---|
| `available` | Sufficient authorized evidence satisfies the named capability/purpose policy at evaluation time |
| `degraded` | Capability remains usable but one or more policy-defined limitations apply |
| `unavailable` | Sufficient evidence establishes that the named capability cannot currently be used |
| `unknown` | Evidence is absent, stale, conflicting, suppressed, or insufficient to decide |
| `not_applicable` | The capability does not apply to the subject |

An assessment must record: subject; capability/purpose; state; controlled reason codes; scope; evidence resource/revision IDs; evidence and evaluation times; freshness rule and result; policy identifier/version; evaluator/provenance; restrictions; and optional valid-until or recheck time. Example capabilities include `observe`, `read-archive`, `receive-live-data`, `accept-command`, `evaluate-feasibility`, `execute-control`, and `serve-api`. [P]

Reason/evidence values may include `stale`, `last_known`, `disconnected`, `unreachable`, `disabled`, `retired`, `dependency_degraded`, `coverage_partial`, `source_conflict`, or `insufficient_evidence`. Authorization denial and policy suppression are recorded in the policy/audit plane, not as false operational facts. A redacted external projection may omit the assessment entirely or return the deployment's concealment response. [N,P]

### 6.4 Vocabulary Rules

- Observed status categories use a stable definition URI and SWE Category code space or allowed-token constraint. [N]
- Quantitative status values retain unit, component definition, valid ranges, and quality. [N]
- Compound status uses a SWE DataRecord with unique, semantically defined fields where the stream schema calls for a record. [N]
- Availability reason, event type, severity, and aggregate-status rules use versioned controlled vocabularies owned by the applicable profile; a display label alone is insufficient. [P]
- Severity is not a core Part 2 System Event property. If needed, it is a profile/admin extension with documented ordering and disclosure. [N,P]

---

## 7. Resource-Family Status and Availability Model

### 7.1 Matrix Conventions

Exposure codes are **E** approved external CSAPI, **EP** separately advertised external profile, **A** administrative, and **I** internal. Classification codes are **C** current projection, **H** historical record, **D** derived assessment, and **V** event. “Persist” identifies logical durability only, not a database design.

### 7.2 Required Status/Event Matrix

| Resource family | Concept | Source/anchor | Authority | Class | Exposure | Temporal basis | Links | Trigger | Availability/freshness | Security | Persistence | Validation | Test | Handoff | Notes/unresolved |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| System | operational dimensions | Part 2 status stream; SensorML | N,P | C,H,D | E via Obs; EP aggregate | phenomenon/result/evaluation | status DataStreams, Observations, Deployment | new status evidence | purpose-specific; never one universal flag | high operational sensitivity | observations durable; view rebuildable/cacheable | SWE schema, vocabulary, provenance | multi-property, missing/conflict/stale | 021,022,027,034,040,042 | Core System representation has no generic readiness member |
| System | operational/maintenance event history | Part 2 §12 | N | H,V | E | event time plus transaction clocks | exactly one System; related links by profile | genuine System occurrence | event may influence later assessment but is not state | filter body, links, counts | durable append-oriented resource | event shape/type/time/link | CRUD/query/filter/correction/auth | 025,032,035,040,041 | prose/schema and type-URI gaps remain |
| Procedure | validity/configuration/capability | Part 1; SensorML | N,P | C,H | E | valid/effective and revision | Systems/streams using it | revision/lifecycle mutation | no current operational status by default | may reveal capability | revisions/lifecycle | schema, links, validity | expired/replaced procedure | 021,023,024 | Procedure availability is a deployment policy concept, not core field |
| Deployment | effective participation | Part 1; SensorML Deployment | N,P | C,H,D,V | E; EP | deployment interval/evaluation | Systems/platform/location | actual deployment/relocation/decommission occurrence | scheduled/active/ended may be derived from time, not asserted fact | operational location sensitive | revision and interval facts; events separate | interval, member links | future/current/past; delayed relocation | 021,026,040,042 | Do not derive physical presence from schedule without evidence |
| Sampling Feature | dynamic property snapshot | Part 1 §14.7 | N | C,H | E | observation phenomenon time and request `datetime` | property/Observation/System | new property Observation | stale/coverage rules per property | location/target may be sensitive | observations durable; snapshot derived | property/schema/time | as-of, no value, late correction | 024,027,034,040 | Not a generic feature-health field |
| Property | definition/vocabulary lifecycle | Part 1; SWE | N,P | C,H | E | resource validity/revision | usages/derived properties | definition revision | not operationally available/unavailable | ontology may be restricted | revision/lifecycle | stable URI and semantic constraints | version/drift/unknown vocabulary | 021,024 | Property resource defines meaning, not current value |
| DataStream | live delivery and archive coverage | Part 2 §9/schema | N,P | C,H,D | E; EP detail | stream evaluation; phenomenon/result extents | System, schema, Observations | channel/source transition; observation commits | `live` independent from archive, freshness, coverage | traffic patterns sensitive | metadata/revisions; telemetry history by policy | schema; derived extents/result type; configured/evaluated `live` rule | live true with stale/no history; live false with archive | 025,027,034,035,040,042 | schema permits null; later representation policy must define use |
| Observation | observed status/data fact | Part 2; SSN/SOSA | N | H | E | phenomenon/result plus pipeline clocks | stream, System, FOI, procedure | accepted observation | can be latest yet stale; old but valid | result/quality/source restrictions | durable append-oriented record/revision | SWE result, times, associations, quality | out-of-order, corrected, future, duplicate | 022,023,027,031,034 | Arrival event is not automatically System Event |
| Current status projection | latest applicable evidence by dimension | IDR-018/019; this report | P | C,D | EP/A | request evaluation and snapshot | source observations, policy, provenance | evidence/policy change | include freshness, coverage, uncertainty | authorization before selection/aggregation | rebuildable; optional bounded cache | deterministic selector and policy version | auth, ties, staleness, conflict, cache | 024,027,034,040,042 | No unadvertised core member |
| ControlStream | accepts commands now | Part 2 schema | N,P | C,D | E; A detail | request/evaluation time | receiving System, schema, Commands | channel transition | `live` is necessary evidence, never feasibility/authority guarantee | highly sensitive | metadata and assessment evidence as policy requires | server control; schema; relationship | live/disabled/denied/disconnected matrices | 025,035,036,038,040 | `async` is processing mode, not readiness |
| Command | current execution projection | Part 2 §10 | N | C,H | E | issue/execution/report times | stream, statuses, results | accepted status report | status not device availability | sender, target, parameters sensitive | immutable intent plus ordered history | schema/idempotency later; status selection | sync/async and terminal behavior | 025,031,036,038 | Current status derived from reports |
| CommandStatus | point-in-time progress report | Part 2 §10.11 | N | H | E | required `reportTime`; conditional execution time | exactly one Command; results | authorized status ingest | not an availability assessment | errors/progress/results sensitive | append-oriented; governed correction | code-specific constraints and ordering | transition, repeat, late, invalid progress | 023,025,036,038 | `UPDATED` is a report occurrence, not necessarily durable state |
| Feasibility | hypothetical evaluation lifecycle | Part 2 §11 | N | C,H | E when class claimed | issue/report/execution estimate | feasibility channel, statuses, result | evaluation submission/status | not authority, reservation, or future guarantee | can expose tactics/capability | intent, statuses, result | command schema and allowed codes | result variants, expiry, changed conditions | 036,037,038 | Separate advertised class; implementation evidence is incomplete |
| Relationship/link | resolvability/visibility | IDR-017; RFC 8288 | N,P | C,D | E/A | request evaluation/snapshot | typed endpoints | target lifecycle/policy change | unresolved/forbidden is not subject unavailability | inference-safe filtering | authoritative fact/revision | relation/cardinality/authorization | dangling, redacted, external offline | 026,039,040,043 | Link state is separate from domain state |
| Publisher/source | reachability and ingest quality | IDR-019; implementation evidence | P,I | C,H,D,V | A/I | heartbeat/receipt/evaluation | adapter/source/provenance/resources | connect, disconnect, validation or lag threshold | source-specific; loss usually yields unknown | topology and identity sensitive | telemetry/audit according policy | authenticated source, metric units, thresholds | flap, spoof, partial feed, lag | 031,034,039,041,042,043 | Not a core CSAPI System Event by default |
| Service endpoint | liveness/readiness/dependency health | HTTP; ops practice | N,P | C,H,V | A/I | probe time/evaluation | service/dependencies | process/dependency transition | capability `serve-api`; HTTP outcome is path-specific | never expose internals publicly | telemetry/incident history by policy | bounded probe and problem contract | 502/503/504, retry, partial dependency | 030,039,041,042,044 | Never place server health in a domain System record |
| Audit record | attempted action and policy outcome | IDR-019/RFC 9457 boundary | P | H,V | A/I | request/decision/commit times | principal, policy, resource/event | every security-relevant attempt | not operational status | restricted; personal/security data | durable per audit policy | append-only/integrity/redaction | allow/deny/fail/concealment | 038,039,040,041,055 | Failed attempt does not create authoritative domain state |

### 7.3 Required Current and Historical Views

- **Systems:** current view only through authorized status-property projections or an advertised profile; history through status Observations and System Events.
- **DataStreams/ControlStreams:** narrow standard current `live`; metadata/revision history and optional administrative channel-health history remain separate.
- **Commands/Feasibility:** current status projection plus durable ordered status and result history.
- **Sampling Features:** as-of dynamic property snapshots from observations.
- **Procedure, Property, Observation, System Event:** no generic operational current-status view is required.
- **Service/source/publisher:** administrative/internal current and historical diagnostics, not domain-resource status.

---

## 8. Current-State, Historical-State, Dynamic-Property, Observation-Derived, and Event Findings

### 8.1 Current Status Projection Contract

A current status projection is evaluated at a captured request evaluation time and repository snapshot. For each requested status dimension it records or can trace:

- subject ResourceId/revision and status-property definition;
- selected DataStream and Observation ResourceIds/revisions;
- result component path and typed value;
- phenomenon time, result time, ingest/receipt time, commit time, and evaluation time as applicable;
- deterministic selection rule, tie-breaker, policy ID/version, and snapshot/cursor context;
- freshness state, age/evidence window, quality/uncertainty, source availability, and coverage;
- provenance and transformation/aggregation identity; and
- authorization/releasability projection applied before aggregation and link generation.

The selector must say which clock implements “latest.” For operational status, phenomenon time is normally the state evidence clock; result/ingest/commit time resolves pipeline and deterministic-order questions. A later-arriving older observation can improve history without replacing the current phenomenon-time projection. Corrections and equal-time conflicts follow explicit revision/source-precedence policy, never arrival-order accident. [P]

### 8.2 Historical Status

Historical status remains ordinary Observation history. A status change can be inferred between two samples, but the exact physical transition time is unknown unless directly supplied. Query results therefore preserve sampled/asserted times and must not fabricate continuous state intervals. A profile may derive intervals using a declared carry-forward rule and maximum gap; those intervals are derived artifacts with provenance, not raw observations. [N,P]

### 8.3 Dynamic Properties

Dynamic Sampling Feature properties follow Part 1 §14.7: model values as Observations, optionally associated with a DataStream, and select the snapshot valid at the requested `datetime`. The same pattern is suitable for position, orientation, or other time-varying properties when the accepted resource model and observed-property semantics support it. It does not make every changing server attribute an Observation. Lifecycle state, authorization, cache state, and service telemetry remain in their owning planes. [N,P]

### 8.4 Status Values as Observations

Use:

- SWE Boolean for genuinely binary properties such as a physical on/off indication;
- SWE Category for controlled modes, health codes, or discrete condition values;
- SWE Quantity/Count for battery level, temperature, error count, or capacity with units and constraints;
- SWE DataRecord for a semantically cohesive status snapshot whose uniquely named fields share sampling context; and
- separate observed properties where component clocks, sources, policies, or update rates differ materially.

Do not serialize arbitrary untyped “status JSON.” The DataStream schema, observed-property definitions, code spaces, units, optionality, nil/missing semantics, and quality components define the interpretation. [N,P]

### 8.5 Events Versus State

An event can explain or invalidate a state without itself being the current state. A calibration event may affect quality; a relocation event may change deployment/location context; a decommission event may correspond to lifecycle policy; a disconnect telemetry event may make availability unknown. Each effect is separately computed and linked. Replaying System Events alone is not guaranteed to reconstruct status unless the profile explicitly defines event-sourced semantics and complete coverage. [P]

---

## 9. System Event Categories, Triggers, Metadata, Links, and Retention

### 9.1 Approved Domain Boundary

Part 2 System Events capture events and maintenance operations on Systems. The approved prose identifies these semantic categories:

- calibration;
- configuration change;
- software update;
- part replacement;
- relocation;
- deployment; and
- decommissioning.

The list is extensible. The published identifiers for these categories remain placeholder `x-OGC/TBD` URIs. Glaux should preserve source identifiers, monitor the OGC vocabulary, and, if implementation precedes resolution, publish a versioned Glaux/profile vocabulary under a stable controlled namespace. It must not present a guessed URI as OGC-defined. [N,X,P]

### 9.2 Generation Matrix

| Occurrence/change | Public System Event? | Other required record | Rule |
|---|---|---|---|
| Authorized calibration, maintenance, configuration, software, part, relocation, deployment, or decommission occurrence on a System | Yes, when the occurrence is known and type is valid | provenance; lifecycle/outbox/audit as applicable | One domain occurrence, stable event identity; do not infer occurrence solely from metadata edit |
| System/Procedure/Property/stream CRUD | No by default | resource lifecycle event, provenance, outbox, audit | May also produce a System Event only if the request explicitly records a genuine System occurrence |
| Retire/delete/tombstone a System resource | No automatic equivalence | lifecycle event and audit | A real decommission occurrence may be a separately linked System Event; deletion time is not decommission time |
| Deployment resource interval change | Conditional | revision/provenance/lifecycle event | Generate relocation/deployment event only when it records actual domain occurrence, not schedule correction |
| Each Observation or status sample arrival | No | data ingest/provenance/outbox event | Threshold/anomaly policy may generate a distinct profiled System Event with evidence links |
| Derived availability changes | No by default | assessment revision and internal/admin event | A mission/profile-defined operational incident may be separately exposed; prevent alert flapping |
| Each CommandStatus report | No | CommandStatus, command/outbox/audit correlation | A successful command's real-world effect may independently create a System Event when evidenced |
| Command submission, rejection, failure, cancellation | No by default | command status and audit | These describe command processing, not necessarily a System occurrence |
| Source/publisher connect, disconnect, lag, validation failure | No core System Event | operational telemetry, provenance/audit | Could be a restricted admin event; never imply physical failure without evidence |
| API service start, degraded dependency, overload, failover | No | monitoring/incident/audit event | Express request failures with HTTP/Problem Details; keep topology private |
| Synchronization, replay, deduplication, merge, conflict | No | sync/provenance/audit/transport event | Imported domain System Event retains its domain identity and clocks |
| Part 3 create/update/delete notification or data message | No new System Event merely because published | outbox/message/delivery record | Envelope/message and underlying resource are separate identities |

### 9.3 Canonical Internal Event Record

The encoding-neutral record must preserve enough information to generate the selected approved/profile representation and to support audit-quality history:

| Field group | Required internal content |
|---|---|
| Identity | System Event ResourceId, revision, source event identity/aliases, deduplication key when supplied |
| Subject | exactly one authoritative System relationship; related Deployment, component, stream, command, observation, or earlier event references as justified |
| Semantics | type URI plus vocabulary/version; human label/name; optional description/message; source-original type/value |
| Time | event occurrence instant/extent; asserted time basis/precision; receipt, commit, publication, and correction times from their owning records |
| Evidence | source/actor, provenance activity, quality/uncertainty/confidence assertions, supporting artifact/resource links |
| Change context | optional prior/new status references or affected component/property; do not duplicate unbounded payloads |
| Profile metadata | severity/priority only if a named profile defines them; correlation/causation identifiers |
| Governance | lifecycle/revision, classification, releasability, policy decisions, redaction state, retention class |

Only approved or advertised representation fields cross the normal CSAPI boundary. Internal metadata is not license to add undeclared members. [N,P]

### 9.4 Representation Gap

The Part 2 conceptual model names `name`, optional `description`, required type URI, required `eventTime`, and optional `message`. Tagged `systemEvent.json` instead extends the SensorML Event schema and requires `definition`, `label`, and `time`. An implementation must decide whether its conformance target follows a corrected artifact, published schema, or documented compatibility overlay. IDR-SRV-021 and 023 must define the adapter only after rechecking official artifacts and tests. Golden fixtures must keep both shapes and reject ambiguous mixed mappings unless the advertised profile explicitly allows them. [N,X,P]

### 9.5 Links and Query

- Every System Event has exactly one authoritative System parent relationship; root and nested views identify the same event ResourceId. [N,P]
- Related resources use typed links from IDR-SRV-017; a related Command or Observation does not replace the System association. [P]
- `datetime` is applied to the event occurrence time defined by the chosen representation mapping; `eventType` applies to the canonical type URI. [N,X,P]
- Pagination uses a deterministic key such as event occurrence time plus transaction sequence/ResourceId; late arrivals remain reachable without pretending commit order equals occurrence order. [P]
- Authorization is applied before filtering results, counts, links, pagination cursors, or aggregate summaries. [P]

### 9.6 Correction and Retention

System Events are append-oriented historical resources. A correction creates a new authoritative revision of the same event identity or an explicitly linked correction/supersession event according to later write policy; it never silently overwrites the prior assertion. The original revision, correction provenance, transaction time, and audit record remain available under retention policy. [P]

No universal retention period is selected here. Retention depends on event type, operational value, classification, legal/accountability rules, provenance dependencies, and tombstone/synchronization needs. Event history may need to outlive an active System representation. Deletion must not strand retained event/provenance references, while retention authority must not be inferred merely because data is useful. [A,P]

---

## 10. Command Status, Feasibility, and Control-Stream Availability

### 10.1 Part 2 Status Codes

| Code | Meaning and status-specific implication |
|---|---|
| `PENDING` | Request received but not yet validated/accepted; not proof of channel or device readiness |
| `ACCEPTED` | Preliminary acceptance/validation; command can still later be rejected or fail |
| `REJECTED` | Terminal refusal; command will not execute |
| `SCHEDULED` | Validated and scheduled; execution time is required by the semantic rule |
| `UPDATED` | Reports acceptance of a command update; treat as a report occurrence, not necessarily a stable execution state |
| `CANCELED` | Terminal cancellation report |
| `EXECUTING` | Execution in progress; repeat reports and progress refinement are permitted |
| `FAILED` | Terminal failure; preserve sanitized external message and restricted diagnostics separately |
| `COMPLETED` | Terminal completion; actual execution time applies, but physical effect may still require separate evidence |

`reportTime` and `statusCode` are required. `percentCompletion` is bounded 0–100; `executionTime` changes meaning with the status code; `message` is human-readable; results may become available incrementally. Synchronous commands have one terminal report. Asynchronous commands can have preliminary, scheduled, executing, and terminal reports, and long-running operations should report progress incrementally. [N]

### 10.2 Deterministic Current Status

The Command's current status is a server-derived projection over authorized status reports. Selection uses report time plus a stable transaction/sequence tie-breaker, validates lifecycle compatibility, and preserves late or corrected reports without rewriting history. A future IDR-SRV-036 state machine must decide permitted transitions, repeats, update/cancel races, timeouts, retry semantics, and how malformed or contradictory device reports are quarantined. This report does not add a `TIMEOUT` code to the standard enumeration; timeout may be a server policy outcome or mapped failure only under the later contract. [N,P]

### 10.3 Five Separate Gates

A command path may involve five independent decisions:

1. the ControlStream is discoverable and its `live` channel can accept commands;
2. the caller is authenticated and authorized for the action and target;
3. the submitted parameters validate against the stream schema;
4. optional feasibility evaluation supports the proposal under specified assumptions; and
5. the submitted Command is accepted, executed, and produces results/effects.

Passing one gate does not pass the next. Feasibility does not reserve capacity, confer authority, promise later availability, or prove execution. `live = true` does not guarantee the next command will be accepted. [N,P]

### 10.4 Feasibility

Part 2 models Feasibility as a Command on a feasibility channel, supporting synchronous or asynchronous behavior and the same parameter structure. Its status lifecycle uses `CommandStatus`; `SCHEDULED` and `UPDATED` are not used for feasibility, while pending, accepted, rejected, canceled, executing, completed, and failed remain relevant. A feasibility result may be Boolean, descriptive detail, probability, or alternatives depending on schema/profile. The result needs evaluation time, assumptions/evidence, validity/expiry, and provenance if relied upon operationally. [N,P]

### 10.5 Sensitive Fields

Command parameters, target identity, sender, schedules, progress, failure reasons, result links, channel `live`, feasibility alternatives, and timing can reveal intent, capability, readiness, or operational tempo. Later policy must support field/link redaction without creating invalid representations and must preserve complete restricted audit evidence separately. [A,P]

---

## 11. Freshness, Staleness, Degraded Operation, Cache, and DDIL

### 11.1 Freshness and Availability Independence

| Evidence condition | Correct interpretation |
|---|---|
| Fresh status value, source connected | May support availability; policy still evaluates capability, completeness, quality, and authorization |
| Stale last-known value, source disconnected | Preserve the value and its age; usually availability `unknown` or `degraded`, not automatically `unavailable` |
| Old observation still within its valid domain interval | Old by receipt age can remain valid for an historical/as-of query |
| DataStream `live = false`, archive present | Live delivery unavailable; historical data may remain fully available |
| DataStream `live = true`, latest observation stale | Transport can be live while evidence is stale or silent |
| ControlStream `live = true`, caller denied | Channel may accept commands, but this caller is not authorized; do not falsify the operational fact internally |
| Source connected, conflicting status values | Connectivity is good; status/availability may be `unknown` pending source-precedence policy |
| System retired | Lifecycle is retired; retained history may remain available; operational capability is normally not applicable/unavailable per purpose policy |

### 11.2 Last-Known State

A last-known projection always carries the source observation, evidence time, selection clock, age, evaluation time, freshness-policy ID/version/result, source-availability context, and limitations. “Last known” is a retrieval rule, not a quality endorsement. If the caller is not authorized to see the freshest evidence, the server must compute the projection from the authorized set and avoid revealing that newer hidden evidence exists. [P]

### 11.3 Degraded and Unknown

`degraded` requires a named capability that remains usable with a policy-defined limitation, such as reduced coverage, incomplete component set, lower quality, bounded lag, or dependency failover. It cannot mean merely “not perfect.” `unknown` is mandatory when evidence is insufficient, conflicting, unavailable, or suppressed such that the domain condition cannot be established. A profile may expose a generic unknown reason while preserving the sensitive cause internally. [P]

### 11.4 HTTP Cache Versus Domain Cache

RFC 9111 defines HTTP response freshness from freshness lifetime and response age. It constrains when intermediaries can reuse stale responses, especially with `must-revalidate`. That protocol behavior is separate from whether an Observation or availability assessment is current enough for a mission purpose. Glaux must apply both correctly:

- HTTP cache headers protect representation reuse and authorization boundaries;
- ETag/conditional requests validate representation revisions;
- domain freshness evaluates evidence inside the representation; and
- an offline/edge last-known projection needs explicit as-of and domain-age context even if its HTTP response was freshly generated. [N,P]

Sensitive or principal-specific status responses require cache controls that prevent cross-principal leakage. Safety- or control-critical responses should use revalidation rules selected later rather than permitting an intermediary to serve unsafe stale state. [N,P]

### 11.5 Service Failure Semantics

Use `503 Service Unavailable` for temporary inability of this API service to handle a request, optionally with `Retry-After`; `502` for an invalid upstream response while acting as gateway; and `504` for an upstream timeout. A successful request can still return a System whose operational availability is `unavailable`; conversely, a `503` says nothing definitive about the physical System. Problem Details communicates bounded API error semantics without exposing stack traces, dependency topology, credentials, hidden resource existence, or classified status. [N]

### 11.6 DDIL and Synchronization Rules

- Preserve last-known values, evidence clocks, freshness results, source reachability, local evaluation policy, sync watermark, known gaps, and unresolved conflicts. [A,P]
- Loss of connectivity does not invalidate retained historical data or prove the remote System failed. [P]
- Reconnection may create internal source/sync events and trigger reevaluation; it does not automatically create a public System Event. [P]
- Delayed System Events sort by occurrence time for domain history while retaining receipt/commit/publication order for causality and replay. [P]
- Out-of-order status Observations update history and change the current projection only if the declared selector makes them applicable. [P]
- Deduplicate by stable source identity/correlation where available; never deduplicate merely on equal value/time if distinct occurrences are possible. [P]
- A disconnected node must not convert local inference into source-asserted truth. Synchronization retains provenance, branch, conflict, and policy context from IDR-SRV-019. [P]

---

## 12. Persistence, Indexing, Query, Validation, and Fixtures

### 12.1 Logical Persistence Classes

Persist durably:

- accepted status Observations and their revisions/provenance;
- System Event resources, corrections, authoritative System association, source identifiers, and provenance;
- Command intent, ordered status reports, results, and correlation;
- resource lifecycle facts, tombstones, and authoritative relationship facts;
- policy-significant availability assessments when needed to explain a decision or disclosure;
- atomic outbox/domain-event facts needed for publication; and
- security/audit records under their own access and retention regime.

Current status, availability summaries, last-known projections, counts, derived stream extents, and display rollups should be reconstructable. They may be cached/materialized with source watermark, policy version, evaluation time, and invalidation rules; a cache is never the only authoritative history. [P]

### 12.2 Indexing and Query Handoff

The later persistence design should evaluate indexes for:

- System Event: System ID + occurrence time + ResourceId/sequence; event type URI + occurrence time; related-resource/correlation identity; transaction time;
- status Observation: DataStream/property/component + phenomenon time + stable ID; result/ingest/commit time for operational queries; source identity;
- CommandStatus: Command ID + report time + sequence; status code; execution interval; result correlation;
- availability assessment/cache: subject + capability + evaluation/policy version; evidence dependencies and invalidation watermark; and
- lifecycle/audit/outbox: subject, event kind, transaction sequence, actor/source, correlation, publication state, and retention class.

Indexing severity or arbitrary status values is justified only by declared queries and controlled representations. Sensitive indexes and counts remain subject to authorization. [P]

### 12.3 Validation Rules

| Area | Minimum validation |
|---|---|
| Status schema/value | DataStream `type`; SWE component shape; property definition; code space/allowed tokens; unit; record-field uniqueness; null/missing distinction |
| Status time | RFC 3339 syntax where used; phenomenon/result/report roles; interval validity; future/late/out-of-order policy |
| Current projection | authorized evidence set; deterministic selector/tie; policy/version; freshness inputs; source watermark; coverage |
| Availability | known state and reason vocabulary; named capability; evidence references; evaluation time; no forbidden state collapse |
| System Event | exactly one System; supported representation shape; type URI/vocabulary; occurrence time/extent; link integrity; profile fields |
| CommandStatus | required fields; status enumeration; progress range; code-specific execution-time requirements; terminal/repeat/transition rules later |
| Provenance/security | source authority; immutable IDs/revisions; actor/policy context; classification/releasability; redaction-safe links |
| Publication/sync | atomic commit/outbox reference; correlation/dedup; original and pipeline clocks; replay-safe identity |

### 12.4 Fixture Corpus

| Fixture family | Required positive/boundary/negative cases |
|---|---|
| SWE status values | Boolean; controlled Category; Quantity with unit; nested DataRecord; undefined category; bad unit; duplicate field; missing component |
| Current status | one dimension; multiple clocks; equal-time tie; late older data; corrected revision; conflicting sources; partial coverage; unauthorized newest record |
| Freshness/availability | fresh/available; stale/degraded; disconnected/unknown; explicit failure/unavailable; not-applicable; stale but valid; live true/stale; live false/archive present |
| System Event | each seven semantic categories; extension URI; instant/extent; nested/root identity; eventType/datetime filter; late arrival; correction; deleted System/tombstone dependency |
| Representation gap | conceptual prose shape; tagged SensorML shape; mixed/ambiguous shape; placeholder URI; future corrected artifact |
| Command status | sync terminal; async progression; repeated executing; progress 0/100/out-of-range; scheduled time missing; late terminal; cancel/update race; partial results |
| Feasibility | sync/async; accepted then failed; Boolean/detail/probability/alternatives; expired assumptions; positive result followed by command denial |
| HTTP/service | 502/503/504; Retry-After; sanitized Problem Details; authorized/forbidden/concealed resource; cache revalidation; principal-varying response |
| DDIL/sync | disconnect, delayed event, out-of-order status, replay duplicate, branch conflict, source recovery, policy-version change, watermark gap |
| Security | redacted fields/links/counts; inference through paging/filter; restricted event type; command-status message sanitization; audit/domain separation |

Golden files include canonical wire representations only after IDR-SRV-021 through 023 choose the applicable mapping. The semantic scenarios can be defined now and reused across JSON, stream, persistence, and interoperability tests. [P]

---

## 13. Security, Policy, Releasability, and Audit

### 13.1 Sensitive Information

Status and events can reveal location, readiness, faults, maintenance windows, capabilities, mission phase, collection gaps, source identity/topology, operational tempo, command intent, execution timing, failure causes, and future plans. The absence, count, ordering, or timing of a response can be sensitive even when fields are redacted. [A,P]

### 13.2 Enforcement Order

1. authenticate request/session where applicable;
2. identify authorized resource/evidence scope and policy version;
3. filter source records and relationship facts;
4. compute current/latest/availability aggregates only from the authorized set;
5. construct links, counts, pagination and error response from the authorized view; and
6. record the policy/audit outcome independently from the domain record.

Post-aggregation filtering can leak hidden resources and can produce a false status from evidence the caller was not permitted to know. [P]

### 13.3 Denial and Concealment

RFC 9110 permits `404` in place of `403` to hide a forbidden resource's existence. That is an HTTP disclosure policy, not evidence that the System or stream is unavailable. Internally retain the true authorization outcome and audit correlation. Externally ensure list counts, event filters, status aggregates, timing, links, caches, and problem details follow the same concealment policy. [N,P]

### 13.4 Audit Boundary

Audit records cover successful, denied, failed, and concealed reads/writes; status/event publication and suppression; administrative overrides; source authentication; feasibility/command decisions; event corrections; and policy/version changes. A rejected attempt does not create an authoritative Observation, CommandStatus, System Event, or resource revision. A successful domain change correlates its provenance, lifecycle/outbox event, and audit event without merging their identities or retention/access rules. [P]

### 13.5 Releasability and Redaction

- Preserve classification/releasability on source observations, derived assessments, events, relationships, and provenance. [A,P]
- Derived status inherits restrictions from all material inputs unless a documented policy authorizes a lower classification and records that decision. [P]
- Redaction must preserve schema validity and semantic honesty; replace neither hidden uncertainty with certainty nor hidden denial with physical failure. [P]
- Problem Details and command/event messages are not debugging channels. Public details use stable codes and safe remediation, while restricted diagnostics remain in logs/audit. [N,P]
- Shared caches must not reuse principal-specific status/event representations across authorization contexts. [N,P]

---

## 14. Conformance, Interoperability, and Test Implications

### 14.1 Conformance Boundary

- Claim Part 2 System Event conformance only when Requirements 40–44, required representations, root/nested identity, filtering/paging, links, errors, and applicable update behavior pass the selected standards baseline. [N,P]
- Claim advanced filtering for System Events only when `eventType` behavior is implemented and tested with exact URI semantics. [N]
- DataStream and Observation conformance includes status-stream behavior where implemented; a profile aggregate must not be advertised as a core field. [N,P]
- Claim feasibility only when its channel, submission, status, result, sync/async, schema, authorization, and errors are complete. Broad command support is not feasibility support. [N,I]
- Conformance declarations are derived from enabled, tested capabilities, not static aspirational lists. [P,I]
- Draft Part 3 publication remains experimental and separately advertised if later adopted; it does not alter Part 2 resource conformance. [D,P]

### 14.2 Published Ambiguity Tests

The test harness must pin the target artifact revision and explicitly disposition:

1. System Event prose fields versus SensorML Event schema fields;
2. placeholder System Event type URIs;
3. nullable versus required-presence behavior for DataStream and ControlStream `live` and derived extents;
4. conceptual required/current fields versus read-only/defaulted JSON schema behavior; and
5. any singular/plural or root/nested path inconsistency found in the approved document and tagged OpenAPI.

Compatibility behavior should be isolated in adapters and labeled profiles rather than scattered across handlers. [X,P]

### 14.3 Implementation Lessons

| Evidence | Reusable lesson | Do not inherit |
|---|---|---|
| OSH (IDR-SRV-014A) | Handler/resource separation, typed stores, historical replay and live flow, event-driven delivery, granular permission tests | fixed conformance declarations; unproven MQTT claim; transient event bus as canonical System Event; incomplete feasibility |
| CS-Go (IDR-SRV-014B) | Durable System Event CRUD; persistent command/status/result separation; strict decoding; deterministic paging; transactional/outbox-oriented patterns; capability-derived AsyncAPI | removed System History class, query vocabulary drift, readable unauthenticated cursors, implementation event schema as normative oracle |
| pygeoapi (IDR-SRV-014C) | Plugin/capability composition and standards-family reuse | treating general feature/provider health as CSAPI System status |
| SECD/OS4CSAPI (014D–014G) | Route/representation/link/field-preservation tests; distinguish no-exception from semantic success; revalidate mutable deployments | historical 404/availability as permanent behavior; tolerant client losses as server contract |
| Part 3 study (014H) | Separate resource events from complete data messages; atomic commit/outbox; correlation, replay, authorization and capability-gated discovery | treating broker delivery as domain occurrence, assuming QoS implies exactly-once, or claiming unfinished binding conformance |

### 14.4 Test Layers

- **Schema/unit:** SWE status components, event/status schemas, vocabulary, time and code-specific constraints.
- **Domain/property:** non-collapse invariants; deterministic current selection; five-state availability; event-generation rules; command transitions.
- **Repository/integration:** append history, corrections, late data, stable paging, indexes, transaction/outbox atomicity, cache invalidation.
- **HTTP/conformance:** route graph, representations, filters, conditional requests, errors, `Retry-After`, authorization and concealment.
- **Streaming/sync:** committed-only publication, duplicates, ordering, replay, reconnect, delayed occurrence, principal/topic policy.
- **Interoperability:** pinned OS4CSAPI/Explorer clients, exact field preservation, root/nested identity, controlled-vocabulary and mapping fixtures.
- **Operational/security:** readiness/liveness isolation, dependency degradation, inference tests, load/backpressure, restricted diagnostics, audit correlation.

Mutable public servers belong in a separately labeled observational lane and must not gate deterministic conformance or build results. [I,P]

---

## 15. Downstream Topic Handoff Matrix

| Topic(s) | Required handoff from IDR-SRV-020 |
|---|---|
| 021 SensorML | Decide System Event SensorML/prose adapter; keep capability/mode/configuration/history distinct from current readiness; preserve event/profile metadata |
| 022 SWE Common | Define permitted Boolean/Category/Quantity/DataRecord status components, code spaces, constraints, missing/nil and quality behavior |
| 023 Validation | Validate both published event shapes during adjudication, status schemas/vocabularies, command code-specific rules, projection/availability invariants |
| 024 Semantics | Govern observed-property, status, event-type, reason, severity, capability and policy vocabulary URIs/versions |
| 025–030 Persistence | Store append histories and dependencies; support time/type/subject/correlation indexes; rebuild projections; plan retention, caches, migrations and observability |
| 031–033 Writes | Define authorized creation/correction, idempotency, atomic provenance/domain/outbox boundaries, lifecycle effects and error outcomes |
| 034 Dynamic updates | Normalize and validate status Observations; order late/corrected values; update current projections; evaluate freshness/availability; define status-change triggers |
| 035 Streaming/events | Preserve domain versus transport identity; publish committed resources/events; define topics, replay, ordering, duplicate handling, status-change noise controls and Part 3 decision |
| 036 Command lifecycle | Finalize transition graph, repeated/incremental reports, current-status selector, timeout/update/cancel races, terminal rules and result correlation |
| 037 Feasibility | Define feasibility schemas/results, assumptions, expiry, async status, non-reservation semantics and interoperability fixtures |
| 038 Command security | Keep stream availability, caller authority, validation, feasibility, acceptance, execution and physical effect as separate gates; protect sensitive status/results |
| 039 Authentication/security | Enforce authorization before selection/aggregation; protect health endpoints, publisher identity, errors, caches, filters and inference surfaces |
| 040 Policy/releasability | Define classification propagation, redaction, concealment, reason exposure, vocabulary/profile permissions and derived-status release policy |
| 041 Audit | Define correlated but distinct domain/provenance/outbox/audit identities and retention for reads, writes, denial, correction, publication and overrides |
| 042 DDIL | Preserve last-known evidence, unknown/degraded rules, local policy/evaluation time, gaps, reconnect and offline cache behavior |
| 043 Synchronization | Preserve source IDs/clocks/provenance; merge/correct events/status without silent overwrite; manage conflicts, replay and tombstones |
| 050–051 Conformance/traceability | Trace Part 2 requirements and ambiguity dispositions to positive/negative tests; derive claims from enabled capability |
| 053 Fixtures | Build the §12.4 semantic corpus and later bind it to selected representations/golden files |
| 054 Performance | Test status ingest, event filters, current projections, cache invalidation, paging, streaming backpressure and authorization cost |
| 055 Security tests | Test field/link/count/timing leakage, command and event authorization, cache isolation, publisher spoofing and audit integrity |
| 056 Interoperability | Exercise status streams, `live`, System Events, command status/feasibility, event schema variants, client field preservation and route discovery |
| 057 Synthesis | Carry the non-collapse model, ambiguity ledger, profile boundaries, operational evidence and verification gates into final architecture |

IDR-SRV-021 is the only next topic authorized after this report is accepted. [P]

---

## 16. Recommendations

| ID | Recommendation | Priority | Basis |
|---|---|---|---|
| R-020-01 | Model operational truth as independent status dimensions; never use a universal status/health/readiness scalar. | Critical | §§6–7 [N,A,P] |
| R-020-02 | Represent source-reported status as semantically typed Observations on `DataStream.type = status`. | Critical | §§5,8 [N,P] |
| R-020-03 | Compute current status at request evaluation time from an authorized snapshot with deterministic time/tie rules, provenance, freshness, quality and coverage. | Critical | §8 [P] |
| R-020-04 | Adopt capability-specific availability with `available`, `degraded`, `unavailable`, `unknown`, and `not_applicable`; keep reasons separate. | Critical | §6.3 [P] |
| R-020-05 | Treat `live` narrowly: live data delivery for DataStream and current command acceptance for ControlStream. | Critical | §§5,7,10 [N] |
| R-020-06 | Preserve last-known values and report staleness/source loss honestly; use `unknown` when failure cannot be established. | Critical | §11 [A,P] |
| R-020-07 | Use public System Events only for genuine operational/maintenance occurrences on a System, not every CRUD, data, command, audit, service or transport event. | Critical | §9 [N,P] |
| R-020-08 | Use stable versioned URI vocabularies and preserve source identifiers; do not ship the published `x-OGC/TBD` identifiers as stable OGC semantics. | Critical | §§5,9 [N,X,P] |
| R-020-09 | Resolve the System Event prose/schema mismatch in an explicit version-pinned adapter/profile with golden compatibility tests. | Critical | §§9,14 [N,X,P] |
| R-020-10 | Keep immutable Command intent, ordered `CommandStatus`, current projection, results, feasibility and audit distinct. | Critical | §10 [N,P] |
| R-020-11 | Keep ControlStream availability, caller authorization, schema validity, feasibility, command acceptance, execution and physical effect as separate gates. | Critical | §10.3 [N,P] |
| R-020-12 | Persist authoritative histories and make current/availability summaries reconstructable, versioned caches. | High | §12 [P] |
| R-020-13 | Apply authorization before latest/current selection, aggregation, filtering, counts, links, paging and cache reuse. | Critical | §13 [N,P] |
| R-020-14 | Keep service/source health administrative and HTTP request outcomes separate from CSAPI System operational status. | Critical | §§6,11,13 [N,P] |
| R-020-15 | Keep domain occurrence, resource lifecycle, provenance, audit, outbox, message and delivery identities correlated but separate. | Critical | §§6,9,13 [P,D] |
| R-020-16 | Implement late/out-of-order/correction/replay behavior using all relevant domain, pipeline, transaction and evaluation clocks. | High | §§8,9,11 [P] |
| R-020-17 | Adopt the fixture corpus and test layers in §§12 and 14 before capability claims. | High | §§12,14 [P,I] |
| R-020-18 | Recheck official CSAPI artifacts, event vocabulary, ATS/OAS, profile rules and Part 3 status before implementation freeze. | High | §§3,14,17 [P] |

### 16.1 Rejected Options

- one `online/offline` field for System, streams, source and service;
- treating newest value as fresh or available without policy/evaluation context;
- changing a last-known value to “unavailable” merely because its source disconnected;
- auto-creating public System Events for every resource mutation, Observation, command status, or broker notification;
- treating SensorML capability/configuration metadata as proof of present operational readiness;
- using arbitrary string status/event/severity values without a stable vocabulary;
- silently choosing either the Part 2 System Event prose or tagged schema shape;
- treating successful feasibility as authorization, reservation, command acceptance, or execution guarantee;
- exposing internal health/dependency details through domain resources or Problem Details; and
- relying on a mutable public demo or implementation conformance list as normative evidence.

---

## 17. Risks, Constraints, and Open Questions

### 17.1 Risk Register

| Risk | Impact | Control |
|---|---|---|
| Collapsed status semantics | Clients make unsafe readiness/command decisions | Non-collapse taxonomy, capability-qualified assessments, invariant tests |
| Placeholder/mismatched event artifacts | Non-interoperable representations and filters | Version-pin, explicit adapter/profile, compatibility fixtures, upstream recheck |
| Latest/fresh/available conflation | Stale evidence presented as current truth | Independent selection, freshness and availability steps with clocks/policy |
| Event flood or semantic duplication | Unusable history and pub/sub noise | Domain occurrence boundary, threshold/debounce policy, separate event identities |
| Hidden evidence affects visible aggregate | Classification/inference leak and dishonest result | Authorization before selection/aggregation; policy-aware cache keys |
| Source loss presented as device failure | Incorrect operational picture | `unknown`, reason/evidence separation, provenance and last-known age |
| Mutable command history | Loss of accountability and unsafe race resolution | Append reports, stable order, governed correction, audit correlation |
| Retention mismatch | Lost history or unauthorized over-retention | Event/status/audit-specific policy and dependency-aware deletion |
| Transport delivery mistaken for commit/occurrence | False state and duplicate effects | atomic outbox, domain/message identity, replay/dedup fixtures |
| Operational endpoints disclose topology | Security exposure | separate restricted probes; sanitized errors and metrics |

### 17.2 Non-Blocking Open Questions

1. Which corrected System Event representation and OGC event vocabulary will exist at implementation freeze?
2. Which Glaux/AEP profile namespace and governance process will own status-property, availability-reason, severity, and temporary event-type URIs?
3. Which status dimensions and aggregation policies are required for the first reference-server deployment?
4. When should DataStream/ControlStream `live` be `null` versus Boolean, and what evidence/age controls its derivation?
5. Which availability assessments require persistence for accountability, and which may remain reconstructable caches?
6. What event correction form best interoperates after the representation strategy is chosen?
7. What retention classes and deletion holds apply to each System Event, status Observation, command status, audit, and transport record?
8. Which source-precedence and conflict policies govern simultaneous status assertions?
9. Which degraded thresholds, debounce windows, and hysteresis rules are profile-specific?
10. Which public current-status/availability view, if any, should be advertised beyond ordinary CSAPI resources?

These questions are assigned downstream and do not block acceptance of the semantic baseline. [P]

### 17.3 Review Triggers

Reopen affected decisions when:

- OGC publishes a corrigendum, corrected schema, stable event vocabulary, updated ATS/OAS, or new Part 2 revision;
- Part 3 gains an approved binding or materially changes eligible resources/event semantics;
- the AEP/STANAG profile changes status, event, availability, security, or DDIL obligations;
- implementation prototypes demonstrate that the projection algebra, correction model, or index assumptions cannot meet correctness/performance needs;
- command/feasibility work adds a conflicting approved lifecycle rule; or
- security/policy research identifies a disclosure rule incompatible with a proposed profile representation.

---

## 18. Validation Against Plan Success Criteria

### 18.1 Success Criteria

| Plan criterion | Status | Evidence |
|---|---|---|
| Status, availability, health, readiness, freshness, command status, System Event distinguished | Satisfied | §§5–6,10–11 |
| Resource-family requirements mapped | Satisfied | §7 sixteen-field matrix |
| Current, historical, dynamic-property, observation-derived, event, command and telemetry concepts distinguished | Satisfied | §§6–8 |
| System Event categories and generation triggers identified | Satisfied | §9 |
| Freshness, staleness, unavailable, degraded, disconnected, last-known and DDIL documented | Satisfied | §§6.3,11 |
| Persistence, query, validation, security, conformance, fixtures and interoperability documented | Satisfied | §§12–14 |
| Implementation/community findings incorporated as nonnormative evidence | Satisfied | §§3,14.3 |
| Recommendations decision-usable and server-bounded | Satisfied | §§1,16 |
| Downstream handoffs explicit | Satisfied | §15 |
| References explicit and reproducible | Satisfied | §19 and source pin |

### 18.2 Methodology Phase Completion

| Phase | Output | Status |
|---|---|---|
| 1. Source collection/framework | pinned inventory, labels, extraction fields and authority rules | Complete |
| 2. Standards extraction | standards-derived inventory and published-gap ledger | Complete |
| 3. Resource-family analysis | taxonomy, availability algebra and sixteen-field family matrix | Complete |
| 4. System Event analysis | category, trigger, metadata, link, query, correction and retention rules | Complete |
| 5. Downstream implications | freshness/DDIL, persistence, security, validation, fixture, conformance and test handoffs | Complete |
| 6. Synthesis | decision baseline, recommendations, risks, open questions and review triggers | Complete |

### 18.3 Acceptance Boundary

This report is complete and in review. Acceptance would establish the status/availability taxonomy, current-state projection, five-state capability-specific availability assessment, System Event boundary, command-status separation, degraded/DDIL rules, and downstream handoffs. It would authorize only IDR-SRV-021. It would not approve server implementation, a database, a broker, a public Glaux status extension, retention periods, a final command lifecycle, or draft Part 3 conformance.

---

## 19. References

### 19.1 Project and Controlled Sources

- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-020 Research Plan](../IDR%20Plans/idr-srv-020-status-availability-and-system-event-model.md)
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
- [IDR-SRV-015 Canonical Resource Model](idr-srv-015-canonical-glaux-server-resource-model-report.md)
- [IDR-SRV-016 Identifier, URI, and Lifecycle Strategy](idr-srv-016-identifier-uri-and-resource-lifecycle-strategy-report.md)
- [IDR-SRV-017 Relationship and Linkage Model](idr-srv-017-relationship-and-linkage-model-report.md)
- [IDR-SRV-018 Temporal, Validity, and Freshness Model](idr-srv-018-temporal-validity-and-freshness-model-report.md)
- [IDR-SRV-019 Provenance, Lineage, Quality, and Trust Model](idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md)
- [CSAPI upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), version 1.9
- NATO `AC/224(JCGISR)D(2026)0005`, *NATO Standard - STANAG 4789*, 27 April 2026. Project-controlled source; SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`.
  - Enclosure 1: *STANAG 4789, Sensor Integration Standard for NATO JISR Operations*.
  - Enclosure 2: *AEP-4789 Volume I, Sensor Integration Standard for NATO JISR Operations - Reference View*, Edition A, Version 1.
  - Enclosure 3: *AEP-4789 Volume II, Sensor Integration Standard for NATO JISR Operations - Core APIs and Encodings*, Edition A, Version 1.

### 19.2 Standards and Official Artifacts

- [OGC API - Connected Systems - Part 1: Feature Resources, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html)
- [Official CSAPI `v1.0.0` source pin](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [Tagged Part 2 DataStream schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/dataStream.json)
- [Tagged Part 2 ControlStream schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/controlStream.json)
- [Tagged Part 2 Command schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/command.json)
- [Tagged Part 2 CommandStatus schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/commandStatus.json)
- [Tagged Part 2 SystemEvent schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/systemEvent.json)
- [OGC API - Features - Part 1: Core](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC SensorML Encoding Standard 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model Encoding Standard 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [W3C/OGC Semantic Sensor Network Ontology](https://www.w3.org/TR/vocab-ssn/)
- [Semantic Sensor Network Ontology, 2023 Edition](https://www.w3.org/TR/vocab-ssn-2023/)

### 19.3 Protocol Sources

- [RFC 3339: Date and Time on the Internet](https://www.rfc-editor.org/rfc/rfc3339)
- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [RFC 9111: HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111)
- [RFC 8288: Web Linking](https://www.rfc-editor.org/rfc/rfc8288)
- [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457)

### 19.4 Implementation and Interoperability Evidence

- [IDR-SRV-014A OpenSensorHub Study](idr-srv-014a-osh-csapi-server-implementation-study-report.md)
- [IDR-SRV-014B Connected Systems Go Study](idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md)
- [IDR-SRV-014C pygeoapi Study](idr-srv-014c-pygeoapi-csapi-server-implementation-study-report.md)
- [IDR-SRV-014D SECD Study](idr-srv-014d-secd-csapi-server-implementation-study-report.md)
- [IDR-SRV-014E OS4CSAPI Client Smoke-Test Findings](idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md)
- [IDR-SRV-014F SECD Interoperability Findings](idr-srv-014f-secd-interoperability-findings-study-report.md)
- [IDR-SRV-014G OS4CSAPI Discussions Lessons Learned](idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md)
- [IDR-SRV-014H Draft Part 3 Study](idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md)

---

**End of IDR-SRV-020 Research Report**

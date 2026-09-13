# Section 018: Temporal, Validity, and Freshness Model - Research Report

**Topic ID:** IDR-SRV-018<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-018 Temporal, Validity, and Freshness Model](../IDR%20Plans/idr-srv-018-temporal-validity-and-freshness-model.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions, all detailed-question groups, all 6 methodology phases, and all success criteria<br>
**Methodology Used:** Authority-ranked synthesis of accepted IDR-SRV-001 through IDR-SRV-017; direct extraction from approved CSAPI 1.0 prose, requirements, encodings, schemas, and abstract tests; review of OGC API - Features, SensorML 3.0, SWE Common 3.0, RFC 3339, RFC 9110, and RFC 9111; bounded current upstream-issue and implementation evidence; and resource-family temporal, freshness, query, persistence, validation, fixture, and handoff analysis<br>
**Research Time:** Approximately 6 hours of AI-assisted execution on September 13, 2026<br>
**Official Standards Source Pin:** [`opengeospatial/ogcapi-connected-systems` `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Shared Register Baseline:** OGC API - Connected Systems upstream-history register version 1.9; no tracked standard state required a register change during this topic<br>
**Document Purpose:** Establish the encoding-neutral time axes, validity and current/as-of rules, freshness assessment, temporal query contract, persistence implications, and downstream handoffs that the Rust Glaux reference server must implement<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 13, 2026<br>
**Date:** September 13, 2026<br>
**Last Updated:** September 13, 2026

---

## Reading Guide and Evidence Labels

| Label | Meaning |
|---|---|
| **N** | Normative or standards-derived finding from an approved applicable source or incorporated artifact |
| **A** | Project-controlling AEP/STANAG adoption or operational-context finding carried from an accepted report |
| **P** | Accepted Glaux project decision or recommendation proposed for acceptance here |
| **I** | Informative implementation, test, interoperability, or community evidence |
| **D** | Official draft evidence useful for a future seam but not an approved requirement |
| **X** | Published inconsistency, ambiguity, missing artifact, or unresolved maintenance issue |

“Current,” “latest,” “fresh,” “available,” and “retained” are independent predicates in this report. A resource can be current but stale, historical but authoritative, old but valid, cached but fresh as an HTTP response, or unavailable while its last-known observations remain valid historical facts.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Temporal Extraction Methodology
5. Standards-Derived Temporal Concept Inventory
6. Temporal Concept Taxonomy and Definitions
7. Resource-Family Temporal Model
8. Current State, History, Latest Value, Event, Observation, and Command Timelines
9. Validity, Lifecycle, and Versioning Findings
10. Freshness, Staleness, Availability, Cache, and DDIL Findings
11. Temporal Query and Filtering Implications
12. Persistence, Indexing, Consistency, and Retention Implications
13. Validation, Fixture, Conformance, and Interoperability Test Implications
14. Downstream Topic Handoff Matrix
15. Recommendations
16. Risks, Constraints, and Open Questions
17. Validation Against Plan Success Criteria
18. References

---

## 1. Executive Summary

Glaux requires a multi-axis temporal model. CSAPI Part 1 uses `validTime` for the applicability of System and Procedure descriptions, for Sampling Feature validity, and for the actual period of a Deployment. Part 2 adds description validity for DataStreams and ControlStreams; phenomenon and result time for observations; derived stream extents; issue, execution, and report time for tasking; and event time for System Events. HTTP adds a different cache-freshness clock. Glaux additionally needs received, ingested, committed, published, synchronized, lifecycle-transition, and policy-evaluation times to operate reliably, but those implementation-support times must not be mislabeled as CSAPI domain fields.

The planning baseline is:

> **Glaux preserves the source-defined temporal meaning of every fact, records server transaction history independently, captures one trusted evaluation instant and one visibility snapshot per request, and derives current/latest/freshness projections without changing resource identity or rewriting historical domain time.** [N,A,P]

The canonical model therefore has two primary independent dimensions and several typed event clocks:

- **valid/effective time**: when a description, deployment, relationship, or state applies in the represented world;
- **transaction time**: when Glaux accepted a revision into its authoritative history;
- **domain record time**: phenomenon, result, issue, execution, report, and event time according to the applicable resource;
- **pipeline time**: received, ingested, committed, and published timestamps recorded by Glaux;
- **freshness evaluation time**: the trusted instant at which a versioned policy assesses evidence age and source availability; and
- **HTTP cache time**: response generation/validation age and freshness lifetime under RFC 9111, separate from domain freshness.

`current` means applicable at the request's captured evaluation instant after lifecycle, authorization, relationship, and transaction-snapshot rules. `latest` means the maximum of a named standards-defined time field over the authorized, route-scoped candidate set after all other filters; only Observation `resultTime=latest` is a normative special query value. `last-known` is a retained record and may be stale. `fresh` is a policy result, not proof that a value is correct. `unavailable` describes the current ability to obtain or operate something, not the age or validity of existing records.

Every request that depends on “now” must capture `evaluation_time` once. Every paged traversal must retain the same authorization context or safe equivalent, query normalization, data snapshot/watermark, and evaluation time. Otherwise a resource may cross a validity or freshness boundary between pages and produce duplicates, omissions, or inconsistent current/latest answers.

Open-ended resource validity remains a published defect. The approved prose calls GeoJSON bounds ISO 8601 date/time strings, the tagged common schema permits a date-time or `"now"`, and [issue #182](https://github.com/opengeospatial/ogcapi-connected-systems/issues/182) remains open because neither source clearly authorizes query sentinel `".."` inside a resource `validTime`. Glaux must store unbounded intervals explicitly, accept `..` only in the approved query grammar, and keep resource serialization behind a documented adapter policy. The initial 1.0 conformance profile should emit finite bounds where known and the tagged-schema `"now"` form for genuinely ongoing periods only with explicit interoperability fixtures; it must not emit `".."` in a core resource property or claim that choice is approved until the OGC resolves the issue. [N,X,P]

Freshness has two distinct forms. HTTP freshness answers whether a cached representation can be reused without origin validation. Domain freshness answers whether evidence is recent enough for an operational purpose. `Age`, `Expires`, and `Cache-Control` must never be presented as sensor/status freshness. A cached response can be HTTP-fresh while containing stale status, or HTTP-stale while containing valid immutable history.

Acceptance of this report establishes a decision-usable temporal baseline and authorizes IDR-SRV-019 as the next single-topic iteration. It does not select database products or tables; finalize provenance, quality, status vocabularies, ingestion APIs, retention policy, task state machines, streaming, synchronization, security policy, or a draft Part 3 transport.

---

## 2. Scope and Plan Alignment

### 2.1 In Scope and Completed

- Extracted normative and artifact-defined time concepts from CSAPI Parts 1 and 2 and inherited OGC API - Features behavior.
- Classified time concepts by source, authority, instant/interval, external/internal exposure, requirement status, queryability, lifecycle role, freshness role, persistence, validation, testing, and downstream owner.
- Defined current, as-of, latest, history, replay, validity, staleness, availability, expiration, and retention semantics.
- Reconciled observation, status, event, deployment, relationship, schema, command, feasibility, external-source, and cache timelines.
- Defined a bitemporal revision seam without designing physical tables.
- Incorporated accepted implementation, smoke-test, interoperability, and community findings only as non-normative regression evidence.
- Produced decision-ready recommendations and explicit downstream handoffs.

### 2.2 Explicitly Out of Scope

- Provenance vocabulary, source ranking, trust scores, and quality calculation: IDR-SRV-019.
- Final operational status, availability, and System Event vocabularies: IDR-SRV-020.
- Exact codecs and representation implementation: IDR-SRV-021 through 024.
- Database engine, physical schema, partitions, indexes, retention jobs, and cache product: IDR-SRV-025 through 030.
- Write API, correction, idempotency, and observation-update semantics: IDR-SRV-031 through 034.
- Streaming and draft Part 3 transport semantics: IDR-SRV-035.
- Command and feasibility state-machine, timeout, retry, cancellation, and result rules: IDR-SRV-036 through 038.
- Authorization, releasability, federation, DDIL synchronization, and conflict-resolution policy: IDR-SRV-039 through 043.
- Exact public freshness extension schema and service-level thresholds: those topics must consume this report's seam.

### 2.3 Core Research-Question Coverage

| Question | Short form | Status | Evidence |
|---|---|---|---|
| CQ1 | Temporal concepts by canonical family | Complete | §§5–8 |
| CQ2 | Normative, inherited, derived, support, optional, unresolved | Complete | §§3, 5, 7 |
| CQ3 | Validity, freshness, availability, and history representation/query | Complete within boundary | §§6, 9–11 |
| CQ4 | Differences among description, deployment, observation, status, event, command, and feasibility time | Complete | §§7–9 |
| CQ5 | Downstream query, persistence, streaming, DDIL, validation, fixture, and interoperability effects | Complete | §§10–15 |

### 2.4 Reconciliation with Accepted Reports

IDR-SRV-015 established the resource-family inventory and already separated description valid time, deployment valid time, phenomenon time, result time, issue time, execution time, report time, event time, ingest/commit/publication time, and freshness evaluation. IDR-SRV-016 established stable identity and made lifecycle, validity, operational availability, publication, and retention orthogonal. IDR-SRV-017 established typed relationship facts and handed relationship validity, deployment participation, reparenting, external-cache state, and as-of traversal to this topic.

This report preserves those decisions. A change in validity, freshness, status, cache state, or current projection does not allocate a new ResourceId. A replacement remains a different identity. UUIDv7 creation time is never substituted for a semantic time. Relationship, resource, observation, workflow, transaction, publication, and cache clocks remain separate.

IDR-SRV-011's accepted query grammar and operator order remain controlling. This report supplies temporal domain semantics and storage/test implications; it does not reopen the accepted grammar.

---

## 3. Evidence Base and Authority Classification

### 3.1 Primary Standards and Protocol Sources

| Source | Version/status | Temporal anchors used | Access date | Limits |
|---|---|---|---|---|
| [CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) | OGC 23-001, 1.0, approved | Req 3; §§9.2.2–9.2.3, 10.5, 11.2.2, 13.2.2, 14.3.2, 14.7, 19 encodings | 2026-09-13 | `validTime` type/prose and open-bound encoding conflict [X] |
| [CSAPI Part 2](https://docs.ogc.org/is/23-002/23-002.html) | OGC 23-002, 1.0, approved | §§9–13 and 16; DataStream/Observation, ControlStream/Command/Status/Result, Feasibility, Event, query and encoding requirements | 2026-09-13 | Several schema/prose/path defects remain [X] |
| [Official release source](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2) | `v1.0.0`, pinned commit | `common/timePeriod.json`, time schemas, OAS, examples, ATS | 2026-09-13 | Incorporated artifact evidence does not silently repair prose |
| [OGC API - Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) | 1.0.1 corrigendum, approved | §7.15.4 `datetime`, instant/interval/open bounds, intersection | 2026-09-13 | General Features rule is specialized by CSAPI fields |
| [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) | OGC 23-000, approved | `validTime`, history/Event, process description applicability | 2026-09-13 | Description/event encodings are not a transaction-time model |
| [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html) | OGC 24-014, approved | Time components, reference frame, units, record schemas | 2026-09-13 | Semantic components require their schema context |
| [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339) | Standards Track | Internet timestamp syntax, offsets, unknown `-00:00`, leap seconds | 2026-09-13 | Does not define CSAPI interval or freshness semantics |
| [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110) | Internet Standard | `Date`, validators, conditional semantics, safe/unsafe request context | 2026-09-13 | HTTP time is representation/protocol time |
| [RFC 9111](https://www.rfc-editor.org/rfc/rfc9111) | Internet Standard | §§4.2–4.3 and 5: response age, freshness lifetime, stale reuse, validation | 2026-09-13 | Cache freshness is not domain-data freshness |

### 3.2 Project and AEP/STANAG Inputs

The controlled `AC/224(JCGISR)D(2026)0005` package dated 27 April 2026 remains the recorded AEP/STANAG baseline with SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`. It was not redistributed or newly quoted. Accepted IDR-SRV-001 through 003 establish the Part 1, Part 2, SensorML, and SWE Common adoption boundary and the operational need for discoverable, linked, trustworthy, secure context. Those sources justify preserving time provenance, uncertainty, stale/last-known distinctions, and DDIL context; they do not invent a new public timestamp field. [A]

Accepted IDR-SRV-004 through 017 provide the governing service, conformance, navigation, query, representation, error, OpenAPI, deployment, resource, identity, lifecycle, and relationship decisions. A conflict would require explicit disposition; no accepted decision was reopened here.

### 3.3 Maintenance and Implementation Evidence

| Evidence | Temporal lesson | Authority limit |
|---|---|---|
| [OGC issue #182](https://github.com/opengeospatial/ogcapi-connected-systems/issues/182), updated 2026-07-21 | Resource `validTime` open-bound syntax and common schema publication remain unresolved | Open maintenance recommendation, not current normative repair [X] |
| [OSH issue #331](https://github.com/opensensorhub/osh-core/issues/331), open at review | Combining phenomenon-time filtering with `resultTime=latest` can return multiple unintended observations | Implementation regression evidence [I] |
| IDR-SRV-014A/014B | OSH and Connected Systems Go demonstrate mature time stores/filters but also preserve legacy history or implementation choices | Patterns are not standards authority [I] |
| IDR-SRV-014C–014F | Client/server smoke and interoperability work exposes filter, ordering, schema, link, and round-trip differences | Bounded snapshots and fixtures [I] |
| IDR-SRV-014G | Community lessons emphasize explicit absence, partial/stale state, pagination, and profile signaling | Discussion evidence only [I] |
| IDR-SRV-014H | Draft Part 3 messages require occurrence, publication, delivery, and replay/idempotency clocks to remain separable | Draft seam only [D] |

### 3.4 Authority Rules

1. Approved requirements and incorporated standards control normative behavior.
2. AEP/STANAG sources control adoption and operational scope; they do not create undocumented CSAPI syntax.
3. Accepted prior IDR decisions control Glaux planning unless a later accepted report explicitly supersedes them.
4. Tagged schemas, OAS, examples, and ATS are checked independently; a conflict remains labeled rather than silently harmonized.
5. Implementations and issues create test hypotheses and compatibility evidence, not normative requirements.
6. Internal support fields and public project extensions are explicitly labeled and capability-advertised.

---

## 4. Temporal Extraction Methodology

### 4.1 Extraction Record

Every extracted temporal fact was recorded against:

- resource family and exact temporal concept;
- instant, interval, duration, sequence, or evaluation predicate;
- source and stable section/artifact anchor;
- normative, inherited, AEP, project, implementation, draft, or unresolved authority;
- external wire field versus internal support metadata;
- required, optional, derived, defaulted, or unresolved status;
- representation and time scale;
- query and ordering behavior;
- lifecycle and freshness interaction;
- persistence and consistency implications;
- validation, fixture, and test implications; and
- downstream topic owner.

### 4.2 Analysis Sequence

1. Replayed accepted IDR-SRV-015 through 017 handoffs and the IDR-SRV-011 query baseline.
2. Read Part 1 and Part 2 conceptual tables, requirements, encodings, tagged schemas, examples, and ATS separately.
3. Compared SensorML and SWE Common temporal semantics without converting schema-local time components into new resource fields.
4. Separated valid/effective, domain event, workflow, pipeline, transaction, publication, cache, and evaluation time.
5. Tested every proposed “current,” “latest,” “history,” “fresh,” and “unavailable” definition against conflicting examples.
6. Routed physical design, provenance, status vocabulary, write behavior, task state, streaming, security, and DDIL synchronization decisions downstream.

### 4.3 Conflict Handling

Where prose, schemas, and examples disagree, the report records the disagreement and chooses a bounded adapter strategy. It does not reinterpret `"now"` as infinity, copy query token `".."` into a resource field without authority, revive removed System History conformance, or use server receipt time as a missing domain timestamp unless the applicable schema explicitly provides that default.

---

## 5. Standards-Derived Temporal Concept Inventory

### 5.1 CSAPI Part 1

| Family | Standard time | Meaning | Status/query effect |
|---|---|---|---|
| System | `validTime` | validity of the System description | optional; `datetime` filters it; null/absent validity still matches [N] |
| System | location time | latest-known location unless a snapshot time is requested | derived projection, not creation/update time [N] |
| Subsystem view | deployment time | permanently attached systems plus optionally currently deployed systems | nested membership needs relationship/deployment evaluation [N] |
| Deployment | `validTime` | period during which the systems are deployed | required domain interval, not merely description validity [N] |
| Procedure | `validTime` | validity of the procedure description; release-to-last-use guidance permits an unbounded end | optional [N,X] |
| Sampling Feature | `validTime` | when the feature is meaningful or usable | optional; examples include biological samples and image footprints [N] |
| Dynamic SF properties | Observation time selected by `datetime` | property value valid at the requested time | optional snapshot projection, not a new fact [N] |
| Property definition | none defined | semantic definition resource has no CSAPI temporal member | revisions still need transaction/audit time internally [N,P] |

Part 1 Requirement 3 applies `datetime` to feature `validTime` and explicitly includes features whose validity is null or absent. Therefore omission does not mean “current only”; it means the feature reports no temporal validity for this filter. [N]

### 5.2 CSAPI Part 2: Observation Side

| Family | Standard time | Meaning | Status/query effect |
|---|---|---|---|
| DataStream | `validTime` | validity of the stream description | optional; `datetime` filter [N] |
| DataStream | `phenomenonTime` | extent spanning all linked Observation phenomenon times | required property but server-derived; null when no observations [N,X] |
| DataStream | `resultTime` | extent spanning all linked Observation result times | required property but server-derived; null when no observations [N,X] |
| DataStream | phenomenon/result interval hints | indicative cadence/duration values in JSON schema | optional metadata; useful freshness input, not proof [N] |
| Observation | `phenomenonTime` | when the observed value applies to the feature | conceptually required; JSON allows omission and defaults it to `resultTime` [N,X] |
| Observation | `resultTime` | when the result was obtained/generated | required; standard states it cannot be in the future [N] |
| Observation record schema | SWE Time component | at least one Time component identifies phenomenon or result time | encoding-specific; mapping URI controls meaning [N] |

Part 2 JSON requires phenomenon and result values on the UTC time scale with an optional offset. A future phenomenon time is valid for forecasts; a future result time is not. The server must retain whether phenomenon time was explicit or derived from result time so later provenance and correction logic does not confuse source assertion with defaulting. [N,P]

### 5.3 CSAPI Part 2: Tasking and Events

| Family | Standard time | Meaning | Status/query effect |
|---|---|---|---|
| ControlStream | `validTime` | validity of the stream description | optional `datetime` axis [N] |
| ControlStream | `issueTime` | extent spanning Command issue times | required derived extent; null handling follows artifact contract [N,X] |
| ControlStream | `executionTime` | extent spanning Command execution intervals | required derived extent [N,X] |
| Command/Feasibility | `issueTime` | time the command was received by the target System | required on server response, ignored if client supplies it on create/update [N] |
| Command/Feasibility | `executionTime` | period of actual command execution until all controlled properties are changed | optional on Command [N] |
| Command/Feasibility Status | `reportTime` | time the status report was generated | required; schema defaults omitted create value to request receipt time [N] |
| Command/Feasibility Status | `executionTime` | scheduled/estimated or actual interval depending on status | optional, conditionally required for scheduled/completed states [N] |
| Command Result link | `resultTime` selector | interval selecting a relevant portion of an output DataStream | representation-specific association qualifier [N] |
| SystemEvent | conceptual `eventTime`; JSON `time` | instant or extent when an event occurred on the System | required; naming mapping must be explicit [N,X] |

Feasibility reuses Command, CommandStatus, and CommandResult shapes and timelines. It is not assigned a second generic “evaluation time” field by the standard; detailed feasibility timestamps, expiry, and decision validity require the IDR-SRV-037 profile/state-machine decision. [N,P]

### 5.4 SensorML and SWE Common

SensorML `validTime` describes when a process description applies. SensorML Event history supplies occurrence records but not Glaux transaction history. SWE Common Time components can express values against a temporal reference frame and require explicit frame information unless UTC is the default. Component `definition` values disambiguate phenomenon, result, and issue time inside compact record encodings. Glaux must preserve these semantic bindings and convert to canonical instants only when the reference frame, unit, epoch, and precision make conversion defined. [N]

### 5.5 HTTP

RFC 9111 defines a response as fresh while `freshness_lifetime > current_age`; `Age` estimates seconds since the origin generated or validated the response. `Expires` does not mean the resource ceases to exist. A cache ordinarily cannot serve stale content unless disconnected or explicitly permitted, and `must-revalidate` forbids disconnected stale reuse. These rules govern representations in caches, not the validity or operational freshness of Systems, Observations, or status. [N]

---

## 6. Temporal Concept Taxonomy and Definitions

### 6.1 Canonical Terms

| Term | Glaux definition | Canonical owner/exposure |
|---|---|---|
| Valid/effective time | World time during which a description, deployment, relationship, or assertion applies | public standard field where defined; otherwise typed internal seam |
| Phenomenon time | World time to which an observed value applies | Observation/DataStream public field |
| Result time | Time the observation result was obtained/generated | Observation/DataStream public field |
| Issue time | Time a Command was received by the target System | Command/ControlStream public field |
| Execution time | Planned/estimated/actual command effect interval according to resource/status context | Command/Status/ControlStream public field |
| Report time | Time a particular status report was generated | CommandStatus public field |
| Event time | Time/extent a SystemEvent occurred; mapped to JSON `time` | SystemEvent public field |
| Source-created/asserted time | Time a source says it created/asserted a fact | provenance metadata; not automatically trusted |
| Received time | Time Glaux first received bytes/message at an authenticated boundary | internal audit/provenance; external only by profile |
| Ingested time | Time parsing/validation accepted the candidate into the ingestion workflow | internal pipeline metadata |
| Committed/transaction time | Time/order the authoritative Glaux revision became durable and visible | internal revision/audit; validator/extension projection as approved later |
| Published time | Time a representation/event was made available to a channel or subscriber | delivery/outbox metadata, not event time |
| Delivered time | Time a particular delivery attempt reached/was acknowledged by a consumer | transport audit only |
| Synchronized time | Time a replica/source reconciliation completed | federation/DDIL metadata |
| Lifecycle-transition time | Effective and transaction times for state transition | internal lifecycle history; profile exposure later |
| Evaluation time | Trusted server instant captured once for current/as-of/freshness decisions | response/query context; not stored as resource truth |
| Cache stored/validated time | HTTP cache processing facts used in `current_age` | HTTP/cache layer only |
| Freshness threshold | Versioned policy duration/condition applied to named evidence | policy configuration, never universal constant |
| Expiration/deadline | Time after which a permission, request, lease, or cache representation should no longer be used under its policy | only present when the applicable contract defines it |
| Retention time | Storage-policy boundary for keeping, archiving, or purging facts | internal policy; never changes original domain time |

### 6.2 Required Separations

- `validTime` is not `createdAt`, `updatedAt`, `committedAt`, or cache expiry.
- `phenomenonTime` is not `resultTime`; forecasts demonstrate that phenomenon can be later.
- `resultTime` is not receipt or ingest time; transport latency and replay demonstrate that it can be much earlier.
- Command issue time is target-System receipt, not client send time or Glaux HTTP receipt unless Glaux is that receiving System under the profile.
- `reportTime` orders status assertions but does not alone prove legal state transition or most recent commit.
- SystemEvent `eventTime`/JSON `time` is occurrence time, not publication or audit time.
- Freshness is evaluated; validity is asserted. Staleness does not invalidate or delete a historical record.
- Availability is capability/reachability state. A source can be unavailable while cached data is fresh under a policy, or available while its latest status is stale.
- Expiration is contract-specific. `Expires` does not retire a resource, and a retired resource is not an expired cache entry.
- UUIDv7 ordering is an identity implementation aid, not a substitute for any semantic clock.

### 6.3 Canonical Internal Value Shapes

The domain layer needs encoding-neutral types, conceptually:

- `TemporalInstant { instant_utc, source_lexeme, source_scale, precision, uncertainty, derivation }`;
- `TemporalExtent { start?, end?, bound_kind, precision, derivation }` with an explicit unbounded value rather than a sentinel string;
- `TransactionStamp { commit_sequence, committed_at }`;
- `EvaluationContext { evaluation_time, snapshot_or_watermark, policy_version, authorization_context }`; and
- `FreshnessAssessment { subject, evidence_time?, evaluated_at, threshold?, state, reason, source_availability, policy_version }`.

These are conceptual contracts, not physical database structures. Unknown time, absent time, unbounded time, source-provided `now`, and a time with uncertainty are distinct values. [P]

### 6.4 Timestamp Normalization

Glaux should parse accepted RFC 3339 offsets to a comparable UTC instant, emit canonical UTC `Z` values for ordinary CSAPI JSON, and retain the source lexeme, precision, offset/scale, and derivation where provenance needs them. Unqualified local time is rejected for standard time fields. RFC 3339 `-00:00` means unknown local offset semantics; operational ordering should reject or quarantine it rather than silently treat it as `Z`. Leap-second input requires a library/path proven by tests; no timestamp is silently rounded into another instant. Exact uncertainty, skew, and non-UTC conversion policy belongs to IDR-SRV-019/023/031. [N,P]

---

## 7. Resource-Family Temporal Model

The required matrix is split into two keyed tables for readability. Together they contain every field required by the plan.

### 7.1 Matrix A: Meaning, Authority, Exposure, Representation, and Query

| ID | Resource family | Temporal concept; instant/interval | Source/anchor; authority | Exposure; requirement/derivation | Representation | Queryability |
|---|---|---|---|---|---|---|
| T01 | Landing, conformance, API definition | document revision/commit; instant/sequence | HTTP + IDR-010A [P] | internal plus validators; support | `ETag`, `Last-Modified`, build metadata | conditional retrieval, not `datetime` |
| T02 | Collection metadata/membership | metadata revision; membership validity | Features/IDR-015–017 [N,P] | internal; optional project seam | validators; typed membership facts | current snapshot; filters on members |
| T03 | System | description `validTime`; interval | P1 §9.2.2/Req 3 [N] | public optional; source asserted | GeoJSON property/SensorML validTime | `datetime`; null/absent match |
| T04 | System location/status projection | source observation time; instant/extent | P1 §9.2.3/§14.7 [N,P] | derived public projection | geometry/dynamic property plus profile context | requested snapshot; otherwise latest-known |
| T05 | Procedure | description `validTime`; interval | P1 §13.2.2 [N,X] | public optional | GeoJSON/SensorML interval | `datetime`; null/absent match |
| T06 | Deployment | deployed period; interval | P1 §11.2.2 [N] | public required | GeoJSON/SensorML validTime | `datetime`; relationship intersection |
| T07 | Sampling Feature | meaningful/usable `validTime`; interval | P1 §14.3.2 [N] | public optional | GeoJSON/SensorML interval | `datetime`; snapshot properties |
| T08 | Property definition | revision/validity if profiled; interval + transaction | P1 §15 [N,P] | no standard time; internal | validators/revision metadata | no core temporal field |
| T09 | Relationship | relationship validity; interval | IDR-017 [P] | internal; projected through views | not a universal public resource | current/as-of traversal |
| T10 | DataStream | description validity; interval | P2 §9.2.2 [N] | public optional | JSON `validTime` | `datetime` |
| T11 | DataStream | phenomenon/result extents; intervals | P2 §9.2.2 [N] | public required-derived/null | JSON `phenomenonTime`, `resultTime` | named extent filters |
| T12 | Observation | phenomenon/result; instants | P2 §9.7.2/§16.1 [N,X] | public required; phenomenon may default | JSON or SWE-mapped Time | named filters; result `latest` |
| T13 | Status DataStream | observation times plus freshness assessment | P2 DataStream types/IDR-015 [N,P] | record public; assessment by profile | Observation and status projection | observation filters/current derived |
| T14 | ControlStream | description validity, issue/execution extents; intervals | P2 §10.2.2/§13.4 [N] | public optional + required-derived | JSON fields | `datetime`, named extent filters |
| T15 | Command/Feasibility | issue instant; execution interval | P2 §§10.7, 11 [N] | public server-reported | JSON/SWE mapping | named issue/execution filters |
| T16 | Command/Feasibility Status | report instant; estimated/actual execution interval | P2 §10.11 [N] | public required/optional conditional | JSON fields | status history ordering; no approved `reportTime` filter |
| T17 | Command Result | result selection interval and publication/commit | P2 §10.13 [N,P] | public association qualifier; support clocks internal | inline/link variants | parent scoped; target's time query |
| T18 | SystemEvent | occurrence instant/extent | P2 §12.2 [N,X] | public required | conceptual `eventTime`, JSON `time` | inherited `datetime` mapping needs overlay |
| T19 | Schema revision | applicability + transaction sequence | P2 schema endpoints/IDR-016–017 [N,P] | internal; active projection public | `/schema`, validators | parent/current; historical decode |
| T20 | External/federated reference | source valid time, retrieved/validated/synchronized times | P1 links/IDR-017 [N,P] | mixed; support metadata/profile | link plus source-qualified cache record | source-aware as-of/freshness |
| T21 | Draft publish/subscribe message | occurrence, publication, delivery, replay times | IDR-014H [D,P] | transport metadata only | future adapter envelope | subscription/replay policy later |

### 7.2 Matrix B: Lifecycle, Freshness, Persistence, Validation, Tests, and Handoffs

| ID | Lifecycle interaction | Freshness/staleness implication | Persistence implication | Validation implication | Test implication | Handoff; notes/unresolved |
|---|---|---|---|---|---|---|
| T01 | docs can be superseded/deprecated | HTTP freshness only | revision + validators | coherent `Date`/ETag/Last-Modified | conditional/cache variants | 045–047; never infer domain freshness |
| T02 | membership changes independently | counts/views can age | transaction history where governed | atomic member view | page snapshot and policy visibility | 025,029,039–040 |
| T03 | same identity, multiple corrections/revisions | description may be current yet operationally stale N/A | valid + transaction history | interval shape, overlap/precedence | absent/null/finite/ongoing/as-of | 019,025,031; open bound #182 |
| T04 | projection changes without System identity change | last-known can be stale/unavailable | retain source record and derivation | no fabricated location/time | latest-known vs as-of vs concealed | 019–020,027,034,040 |
| T05 | supersession does not replace identity by itself | normally N/A | description revisions | release/end constraints | future/ended/unbounded cases | 021,025,031; open bound #182 |
| T06 | active/ended deployments retained | ended is not stale | exact participation interval + transaction | required interval and association consistency | overlap, boundary, current membership | 019–020,025–026,029,031 |
| T07 | expiry of usability not deletion | old sample may remain evidential | validity + revision | interval and dynamic-property link | image instant, biological window, absent | 019,025–027,034 |
| T08 | incompatible semantic change may require new identity/version | HTTP/domain policy separate | immutable revision binding | prevent silent meaning change | old record decodes after schema change | 019,022–025,034 |
| T09 | retire/reparent without erasing fact | target cache freshness separate | independently temporal facts where needed | cycles, overlaps, atomic reparent | instant/interval recursive traversal | 019,025,029–031,040,043 |
| T10 | stream remains identifiable after valid end | `live=false` is neither stale nor deletion | description revisions | validTime semantics | current/retired/live combinations | 019–020,025,031,034 |
| T11 | extents change with accepted records/retention | old extent not stream staleness | exact/materialized aggregates with provenance | min/max/null and authorization view | insert/delete/correct/out-of-order/retention | 027,029,034,040,054 |
| T12 | correction/history policy downstream | old observation is not necessarily stale | append/audit plus semantic indexes | UTC, future result, explicit/default phenomenon | forecast, delay, replay, latest ties, #331 | 019,027,031,034–035 |
| T13 | status state orthogonal to resource lifecycle | cadence/policy drives fresh/stale/unknown | time-series plus derived assessment | named evidence and policy version | stale vs unavailable vs retired | 019–020,027,034,042 |
| T14 | stream lifecycle independent of commands | aggregate age not availability | description + aggregates | null/empty and validTime | no commands, scheduled/future, retention | 020,027,031,036 |
| T15 | task terminal state does not delete Command | stale/late/expired policy separate | immutable request + status history | server-owned issueTime; execution consistency | delayed receipt, replay, out-of-order status | 019,029,031,036–038 |
| T16 | currentStatus derived from legal ordered reports | missing recent report may be stale, not new status | append-oriented history + deterministic projection | status-dependent execution interval | ties, correction, terminal-after-terminal | 019–020,029,036–038 |
| T17 | result retained with parent audit | external result may be stale/unavailable | variant + target qualifier + delivery facts | allowed variant combinations | temporal subset, concealed/stale target | 019,023,027,036,038,040,043 |
| T18 | event correction never becomes resource lifecycle silently | event age is not staleness | append/audit with occurrence and commit | `eventTime`↔`time` mapping | instant/extent, correction, delayed publication | 019–020,025,034–035 |
| T19 | active schema selected by transaction/applicability | cache freshness separate | immutable history for decoding | child bound to exact schema | old payload after new schema | 022–025,027,034,036 |
| T20 | target state and relationship state independent | source/data/HTTP freshness separate | source URI, validators, cache/sync facts | SSRF, integrity, clock provenance | offline, stale, changed, tombstone, conflict | 019,039–043,055–056 |
| T21 | message lifecycle does not redefine resource | replay can deliver old valid events | outbox/dedup offsets later | preserve all clocks/message IDs | delay, duplicate, reorder, reconnect | 019,029,035,042–043; draft only |

### 7.3 Resource-Class Summary

- **Time-valid descriptions:** System, Procedure, DataStream, ControlStream; optionally Sampling Feature in a broader applicability sense.
- **Domain activity interval:** Deployment, plus temporally qualified relationships.
- **Immutable or append-oriented temporal records:** Observation, CommandStatus, SystemEvent, delivery/audit events.
- **Workflow aggregate:** Command and Feasibility, with issue and execution timelines and derived current status.
- **Derived temporal aggregate:** DataStream/ControlStream extents and current/latest projections.
- **Semantically timeless but revisioned description:** Property and many schema/document resources; transaction history still matters.
- **External snapshot:** federated/cache state with source validity, retrieval, validation, synchronization, and policy evaluation kept distinct.

---

## 8. Current State, History, Latest Value, Event, Observation, and Command Timelines

### 8.1 One Request, One Evaluation Context

At request admission, Glaux captures:

1. trusted `evaluation_time`;
2. transaction snapshot/watermark;
3. normalized route and filters;
4. caller authorization/policy context or a safe reference to it; and
5. freshness-policy version where an assessment is requested or emitted.

Every row, relationship traversal, projection, count, page, and next cursor uses that context. A cursor must not evaluate `now` again. If a security policy cannot safely persist the original authorization context, continuation must reauthorize and fail explicitly when equivalent visibility cannot be guaranteed. [P]

### 8.2 Definitions and Evaluation Order

| Projection | Rule |
|---|---|
| Current description | Latest committed authoritative revision whose valid interval contains evaluation time, after authorization and lifecycle rules |
| As-of description | Same rule at requested domain instant and selected transaction snapshot |
| Current relationship | Relationship fact applicable at evaluation time, with endpoint lifecycle and authorization evaluated independently |
| Current deployment membership | Deployment/participation interval contains evaluation time; permanent composition handled separately |
| Latest observation | Maximum authorized `resultTime` after route scope and all other filters; retain equal-time ties |
| Latest-known location/property | Most recent applicable authorized supporting observation/fact under its named ordering axis; may be stale |
| Current command status | Deterministic legal projection of authorized status reports ordered by `reportTime`, with transaction sequence/ID tie handling |
| History | Retained revisions/records ordered on the axis named by the endpoint/profile; never an unlabeled mixture |
| Snapshot | Projection evaluated at a domain instant and transaction snapshot; no new identity |
| Replay | Re-delivery of existing domain facts with original domain times and new publication/delivery audit times |

When two description revisions for one identity are applicable at the same valid instant, Glaux first uses explicit authoritative supersession/correction semantics, then transaction snapshot/sequence. Arbitrary `updatedAt`, UUID ordering, or storage row order is prohibited. Unintended overlapping active descriptions are rejected or raised as integrity findings. Adjacent inclusive bounds can both contain the transition instant; the successor with the later valid start wins only under an explicit same-identity revision rule and the collision remains test-visible. [P]

### 8.3 Observation Timeline

An Observation can be generated at `resultTime`, transmitted later, received, validated, committed, published, cached, replayed, and delivered without changing `phenomenonTime` or `resultTime`. Out-of-order arrival is permitted if the domain times are valid. Stream extents and `latest` are recomputed from domain times, not arrival order.

If JSON omits phenomenon time, the canonical domain value defaults to result time as required by the schema, while derivation metadata records `defaulted_from_result_time`. A later explicit correction is a governed write/revision, not an invisible mutation. Future result time is rejected from the canonical CSAPI record; a profile may quarantine the rejected source payload and skew evidence outside that resource. [N,P]

### 8.4 Status and Dynamic-Property Timeline

Status observations are ordinary time-stamped Observation facts in a status DataStream. A current System status is a derived projection; it is not the latest SystemEvent and not a lifecycle mutation. Dynamic Sampling Feature properties likewise project an Observation value at the requested `datetime`. The projection records or can reconstruct which Observation, schema revision, phenomenon time, result time, transaction snapshot, and policy assessment supported it. Detailed status vocabulary remains IDR-SRV-020. [N,P]

### 8.5 Event Timeline

A SystemEvent records something that occurred on a System. Its conceptual `eventTime` maps to JSON SensorML Event member `time`. Receipt, commit, publication, and delivery are separate. Events can explain a deployment, replacement, calibration, relocation, software update, or decommissioning, but an event does not automatically mutate the referenced resource or relationship; a governed transaction may atomically write both facts. [N,P]

### 8.6 Command and Feasibility Timeline

The minimum command-like sequence is:

`client send → Glaux receive → target-System issueTime → status reportTime(s) → estimated/scheduled executionTime → actual executionTime → result publication/delivery`.

Some instants coincide when Glaux is the target System, but the model does not assume that. `PENDING` means received by the System without an accept/reject decision. A Command's current status is derived from its retained status history. Scheduled execution can be in the future; completed status requires actual execution time under the standard's rule. A canceled Command remains a retained Command; `CANCELED` is not HTTP DELETE. Feasibility uses the same time model while detailed decision expiry and result validity remain downstream. [N,P]

---

## 9. Validity, Lifecycle, and Versioning Findings

### 9.1 Bitemporal Revision Seam

Mutable descriptions and temporal relationships require at least conceptual bitemporality:

- valid/effective time answers when the assertion applies; and
- transaction time/sequence answers when Glaux knew and exposed that assertion.

A correction can change transaction history without changing the intended valid interval. A retroactive update can have a recent commit time and an old valid start. Both facts are required for reproducible as-of answers, audit, synchronization, and conflict analysis. The public CSAPI 1.0 representation usually exposes valid time but not transaction time; internal history must therefore not be inferred from the public field. [P]

### 9.2 Interval Rules

1. Instants compare after defined reference-frame conversion to UTC.
2. Finite intervals require start not later than end; zero-duration intervals are valid instants where the source model permits them.
3. Internal unbounded start/end are typed bounds, never stored as the strings `".."` or `"now"`.
4. Standard query intersection includes finite boundaries as accepted in IDR-SRV-011.
5. `now` is resolved once from request `evaluation_time`; it is not infinity.
6. Same-identity active-description overlaps require explicit correction/supersession precedence or rejection.
7. Deployments and independent relationship facts may legitimately overlap; constraints apply by relationship kind and cardinality.
8. Precision and uncertainty are retained; comparison beyond known precision must not manufacture certainty.

### 9.3 Open-Ended `validTime`

Issue #182 demonstrates three conflicting surfaces:

- Part 1 says bounds are ISO 8601 date/time strings;
- tagged `timePeriod.json` accepts two `TimeInstantOrNow` items;
- OGC API - Features authorizes `..` for unbounded query intervals, not clearly for resource properties.

Glaux's core policy until resolution is:

- parse and persist a true unbounded bound internally;
- accept `..` in the query grammar only;
- do not emit `..` in a core CSAPI resource property;
- emit finite resource bounds when known;
- permit tagged-schema `[start, "now"]` only for an ongoing period and test/document its dynamic semantics;
- isolate future OGC resolution in the representation adapter and compatibility fixtures; and
- advertise any alternative extension profile instead of claiming silent 1.0 conformance.

This is an interoperability containment decision, not a declaration that the approved standard has settled the syntax. [X,P]

### 9.4 Lifecycle Orthogonality

| Axis | Examples | Temporal consequence |
|---|---|---|
| Resource existence | active, retired, archived, tombstoned, purged | affects visibility/retention, not original valid or event times |
| Temporal applicability | future, current, ended, timeless/unreported | evaluated against valid time |
| Operational availability | available, degraded, unavailable, unknown | status model; does not delete data |
| Data freshness | fresh, stale, unknown, not-applicable | policy assessment at evaluation time |
| Publication | private/draft, published, withdrawn/suppressed | access/distribution state with its own history |
| Workflow | pending through terminal command states | derived from status records and legal transition rules |

Ending a valid interval, retiring a resource, archiving its records, expiring a cache response, suppressing publication, losing source connectivity, and deleting a resource are different operations. [P]

### 9.5 Schema and Semantic Version Time

Every Observation/Command record must remain bound to the schema revision under which it was accepted. Incompatible schema replacement after child records exist requires the IDR-SRV-016 new-stream/migration behavior. The current `/schema` representation is a projection; historical decoding uses the exact retained revision. `Last-Modified` and ETag identify representation revisions but do not replace semantic applicability. [P]

---

## 10. Freshness, Staleness, Availability, Cache, and DDIL Findings

### 10.1 Domain Freshness Function

Domain freshness is evaluated as:

`assessment = policy(subject kind, evidence time/type, expected cadence, source state, evaluation_time, uncertainty, policy version)`.

The minimum internal assessment states are `fresh`, `stale`, `unknown`, and `not_applicable`; source availability and validity remain separate dimensions. Detailed externally visible state vocabulary belongs to IDR-SRV-020. A policy may define a threshold using expected DataStream cadence, mission profile, source contract, or explicit resource configuration. No global “five minutes” rule is acceptable. [P]

Policy precedence should be explicit and versioned, provisionally: resource/stream override → source/deployment policy → collection/mission profile → service default. A more specific rule cannot weaken mandatory security or safety policy. The final configuration and authorization rules belong to IDR-SRV-020/039/040/042. [P]

### 10.2 Evidence Selection

| Subject | Candidate freshness evidence | Prohibited shortcut |
|---|---|---|
| System operational status | latest applicable status Observation result/phenomenon time plus source availability | System `updatedAt` |
| DataStream liveness | last successfully accepted relevant result plus expected cadence and source state | `live=true` alone |
| Observation | usually not “stale” as historical fact; assess suitability for a current-use profile | age alone invalidates record |
| Latest value | selected Observation time plus cadence/use policy | latest implies fresh |
| SystemEvent | normally historical occurrence; delivery latency may be assessed separately | old event means stale event |
| Command status | latest legal status report time plus task/state timeout policy | Command issue time alone |
| Description | validTime, source revision/verification, and profile policy | recent commit proves current truth |
| External reference | origin validators, retrieved/validated/synchronized times, source availability | HTTP 200 once means fresh forever |

### 10.3 HTTP Cache Versus Domain Freshness

| Question | HTTP cache answer | Domain answer |
|---|---|---|
| What clock? | response generation/validation and current age | named evidence and evaluation time |
| What threshold? | `max-age`, `s-maxage`, `Expires`, heuristic | versioned mission/source/resource policy |
| What does stale mean? | response needs validation before reuse unless permitted | evidence is older/less certain than policy allows |
| What is `Age`? | seconds since origin generation/validation estimate | never observation/status age |
| Does expiry end the resource? | no | no |

Mutable current/latest projections should use explicit conservative caching and validators. Immutable historical item representations can use longer lifetimes where authorization and classification permit. Responses varying by media type, profile, authorization, snapshot, or freshness representation require correct cache partitioning/`Vary` and often private/no-store policy. Exact header selection belongs to IDR-SRV-045/046. [N,P]

### 10.4 DDIL and External Sources

A disconnected node may expose a policy-authorized local snapshot only with enough internal context to distinguish:

- domain `asOf`/selected record time;
- retrieved, last-validated, and last-successful-contact times;
- local commit and synchronization watermark;
- freshness policy and evaluation time;
- completeness/coverage limits;
- source availability and source identity; and
- whether the representation was served under HTTP stale-response permission.

`must-revalidate` cannot be ignored merely because the node is disconnected. If policy permits domain last-known data but HTTP forbids reuse of a cached representation, Glaux must generate a new authorized local representation from its own retained facts or return an appropriate error; it must not pass through the prohibited stale response. [N,P]

On reconnection, source event/result time orders domain meaning, while source version/provenance and local transaction sequence order synchronization. Neither local receipt time nor UUIDv7 order alone resolves conflicting claims. Tombstones, withdrawals, and authority changes remain source-aware. Exact synchronization/conflict rules belong to IDR-SRV-043. [P]

### 10.5 Client-Facing Freshness Seam

The base standards do not define a universal freshness member. Glaux should therefore keep core representations conformant and expose domain freshness only through a versioned, capability-advertised profile or associated status representation. Any extension must name the evidence time, evaluation time, state/reason, policy identifier/version, and source availability without leaking classified source details. A bare `stale: true` is insufficient. [P]

---

## 11. Temporal Query and Filtering Implications

### 11.1 Accepted Grammar and Matching

IDR-SRV-011 remains controlling:

- RFC 3339 instant or slash interval;
- open interval forms `../end`, `/end`, `start/..`, and `start/`;
- inclusive finite-boundary intersection;
- rejection of date-only values, generalized `now`, `earliest`, or `latest` outside approved cases;
- only Observation `resultTime=latest` as a normative `latest` special value; and
- null/absent Part 1 `validTime` resources match `datetime`.

Part 2's informative phenomenon-time example uses `now`, but the inherited Features grammar and accepted project baseline do not authorize generalized operational `now`. Glaux should not broaden the contract from an example without an explicit compatibility profile. [N,X,P]

### 11.2 Field Mapping

| Endpoint family | Temporal parameter | Evaluated field |
|---|---|---|
| Part 1 feature resources, DataStream, ControlStream | `datetime` | `validTime` |
| DataStream | `phenomenonTime`, `resultTime` | derived aggregate extents |
| Observation | `phenomenonTime`, `resultTime` | Observation properties |
| ControlStream | `issueTime`, `executionTime` | derived aggregate extents |
| Command/Feasibility | `issueTime`, `executionTime` | Command properties |
| CommandStatus | inherited `datetime` defect seam | project maps to `reportTime`; not relabeled as approved until repaired/profiled |
| SystemEvent | inherited `datetime` defect seam | conceptual `eventTime` mapped to JSON `time` through project overlay |

### 11.3 `latest` Operator Order

For `resultTime=latest`:

1. authenticate/authorize;
2. establish canonical endpoint or nested route scope;
3. apply every other predicate, including phenomenon time and FOI;
4. select the maximum visible `resultTime` within that scope;
5. retain all equal-time ties;
6. apply deterministic tie order and paging.

Canonical `/observations` uses the visible endpoint-wide maximum. A DataStream-nested endpoint is naturally stream scoped. OSH issue #331 becomes a mandatory regression fixture: an ANDed phenomenon-time filter must be applied before the maximum is selected. [N,P,I]

### 11.4 Default Ordering and Pagination

Accepted default Observation ordering is `(resultTime, ResourceId)` ascending, comparing normalized UTC instants and placing absent/null after non-null where a family permits them. Status history orders by `(reportTime, transaction_sequence, ResourceId)`; SystemEvent history orders by event-time start then deterministic ID, with transaction sequence retained for corrections. Exact command projection must also validate legal state transitions.

Cursor payloads must bind normalized time filters, direction/order, evaluation time, snapshot/watermark, page boundary, and policy/authorization context. Aggregate counts and extents use the same snapshot. [P]

### 11.5 Authorized Aggregate Extents

The authoritative internal DataStream/ControlStream extent is computed over accepted linked records. A caller-visible representation must not leak hidden records through min/max, counts, freshness, or latest selection. Therefore either:

- compute an exact caller-visible aggregate; or
- suppress/qualify the aggregate under an advertised security profile.

Returning the unrestricted min/max to an unauthorized caller is prohibited. Retention or authorized deletion can shrink an exposed extent; out-of-order inserts can move its start backward; correction can move either bound. Aggregate update and child commit must be transactionally consistent. [N,P]

---

## 12. Persistence, Indexing, Consistency, and Retention Implications

### 12.1 Logical Persistence Requirements

Without selecting a database, Glaux needs logical support for:

- original normalized temporal value plus source lexeme/precision/scale/derivation;
- explicit unbounded interval bounds;
- valid-time and transaction-time revision history;
- monotonic commit sequence independent of wall-clock ties;
- append-oriented Observation, status, event, audit, outbox, and delivery records;
- exact schema revision binding;
- source-qualified external retrieval/validation/synchronization facts;
- versioned freshness-policy application; and
- tombstone/retention history sufficient for synchronization and audit.

### 12.2 Index Workload Requirements

Candidate indexes must support, by authorized partition/tenant and parent scope:

- validity interval intersection and current/as-of selection;
- Observation phenomenon and result time ranges plus `resultTime` descending/latest;
- DataStream/ControlStream derived extents;
- Command issue/execution time;
- CommandStatus report time and parent order;
- SystemEvent occurrence extent and System parent;
- relationship-validity traversal;
- commit sequence/snapshot scans;
- external source plus synchronized/validated time; and
- retention boundaries.

IDR-SRV-025/027 selects physical structures after cardinality and workload measurement. A generic `updated_at` index cannot satisfy this model. [P]

### 12.3 Atomicity and Consistency

The following changes are one logical transaction or recoverable saga with an outbox:

- child Observation/Command accepted and parent aggregate extents updated;
- status appended and Command current-status projection updated;
- Deployment/relationship validity changed and all derived current/nested views updated;
- schema revision activated and new child records bound to it;
- correction committed with prior revision retained; and
- publication intent recorded with the authoritative commit.

Aggregate drift must be detectable and rebuildable from facts. Materialized freshness assessments may be cached but are recomputed against the captured evaluation time and policy version; they are not authoritative facts unless the audit contract records them as decisions. [P]

### 12.4 Clock Skew and Ordering

Server wall-clock time is not sufficient for total ordering. Glaux uses a monotonic transaction sequence/watermark plus recorded commit instant. Source clocks remain untrusted evidence with precision/uncertainty. Receipt time can establish “not seen before” locally but cannot repair phenomenon, result, issue, report, or event time. Clock-regression detection, trusted time source, tolerance/quarantine, and audit policy are required downstream. [P]

### 12.5 Retention

Retention operates by resource class and legal/mission policy, not one timestamp. It must identify which clock drives eligibility and preserve referential/audit requirements. Purging observations can shrink stream extents and change latest/current projections; purging status can destroy workflow evidence; purging schema revisions can make retained payloads undecodable; purging tombstones can cause resurrection during sync. Those dependencies require transactional planning and invariant tests. Exact durations and archive media remain IDR-SRV-028/030/042/043. [P]

---

## 13. Validation, Fixture, Conformance, and Interoperability Test Implications

### 13.1 Validation Layers

| Layer | Checks |
|---|---|
| Lexical | RFC 3339 syntax, offset, precision, leap second, interval arity, duration syntax |
| Semantic | reference frame/scale, start ≤ end, future-result prohibition, field meaning |
| Resource | required/optional/defaulted fields, status-dependent execution time, event `time` mapping |
| Relationship | deployment/relationship validity, parent scope, cardinality, overlap/precedence |
| Transaction | immutable IDs/times where required, revision/ETag, snapshot consistency, idempotency |
| Policy | authorization before aggregates/latest, source disclosure, freshness-policy version |
| Representation | JSON/SensorML/SWE round trip, canonical UTC output, open-bound adapter policy |

### 13.2 Required Fixture Families

1. Equivalent UTC instants with different numeric offsets and fractional precision.
2. Invalid/missing offset, `-00:00`, malformed leap second, valid announced leap-second syntax, and clock regression.
3. Finite, zero-duration, future, expired, unbounded-start, unbounded-end, `now`, and illegal `..` resource intervals.
4. Null/absent Part 1 validity matching `datetime`.
5. Inclusive query boundary, disjoint, contained, containing, and touching intervals.
6. Overlapping/retroactive/corrected description revisions and transaction-time as-of answers.
7. Permanent subsystem versus temporary Deployment membership at current, instant, and interval queries.
8. Observation phenomenon before/equal/after result; forecast phenomenon; defaulted phenomenon; delayed/out-of-order/replayed observation.
9. Empty stream null extents, first/last insert, backfill, correction, deletion/retention shrink, and authorization-filtered aggregates.
10. `resultTime=latest` with ties, multiple streams/FOIs, other filters, pagination, and OSH #331 reproduction.
11. Status reports with identical/report-clock regressions, scheduled/actual execution intervals, illegal terminal transitions, and delayed publication.
12. SystemEvent conceptual `eventTime` versus JSON `time`, instant and extent, correction, and linked resource mutation.
13. HTTP-fresh/domain-stale, HTTP-stale/domain-valid, `must-revalidate` disconnected, ETag revalidation, and authorization-vary cases.
14. External source fresh/stale/unavailable/concealed, changed validator, clock uncertainty, offline snapshot, sync conflict, and tombstone replay.
15. Old data decoded under retained schema after a new schema/stream revision.

### 13.3 Conformance and Golden Files

The harness must distinguish:

- approved requirement tests;
- tagged artifact/schema checks;
- known-defect expected behavior;
- project-profile tests; and
- implementation compatibility observations.

Golden files should cover each supported encoding and preserve both normalized value and derivation where relevant. The #182 resource-open-bound cases must be labeled unresolved/project-adapter rather than passed as approved conformance. Removed System History paths remain expected absent from the CSAPI 1.0 core. [P]

### 13.4 Interoperability Oracles

Portable tests compare semantic result sets, time normalization, tie retention, counts, extents, current/as-of results, and error classification—not incidental row order or a single implementation's history extension. Cross-server comparison must record each server's profile, source version, clock, dataset snapshot, authorization view, and known defects. [P]

---

## 14. Downstream Topic Handoff Matrix

| Topic | Required handoff from IDR-SRV-018 |
|---|---|
| IDR-SRV-019 | provenance for every source/defaulted/corrected time; source scale, precision, uncertainty, trust, receipt/ingest/commit/publication lineage; conflict evidence |
| IDR-SRV-020 | external status/availability/freshness vocabulary; evidence selection; policy precedence; stale/unknown/last-known representation; event time profile |
| IDR-SRV-021–024 | exact SensorML/SWE/JSON mappings, reference-frame conversion, event `time`, open-bound serialization, round-trip derivation |
| IDR-SRV-025 | bitemporal revision, transaction sequence, interval, aggregate, policy, and audit logical requirements; select physical design |
| IDR-SRV-026 | spatiotemporal deployment/location/Sampling Feature indexes and as-of geometry behavior |
| IDR-SRV-027 | phenomenon/result storage, out-of-order ingestion, extent maintenance, latest/tie query, partition and retention strategy |
| IDR-SRV-028/030 | cache/materialization and lifecycle/retention policies with explicit governing clocks and dependent-history protection |
| IDR-SRV-029 | atomic fact+aggregate+outbox writes, optimistic concurrency, idempotency, monotonic commit watermark, clock failure |
| IDR-SRV-031 | server-owned timestamps, validation/defaulting/quarantine, correction and retroactive update semantics |
| IDR-SRV-034 | Observation/status correction, explicit/defaulted phenomenon time, aggregate changes, current dynamic-property projection |
| IDR-SRV-035 | preserve occurrence, commit, publication, delivery, replay, offset, and dedup clocks across streaming/Part 3 adapter |
| IDR-SRV-036–038 | command/feasibility legal timelines, deadline/expiry/timeout/retry/cancel policy, report ordering, scheduled versus actual execution |
| IDR-SRV-039/040 | authorization before latest/count/extent/freshness; time-based policy; prevent temporal and source-information leakage |
| IDR-SRV-041–043 | external retrieval/validation/sync metadata, offline snapshot contract, stale use, conflict/tombstone resolution, clock uncertainty |
| IDR-SRV-045/046 | `Date`, validators, Cache-Control, Vary, profile metadata, current/latest cache strategy; never overload HTTP `Age` |
| IDR-SRV-050/051 | distinguish normative/profile/known-defect tests and trace every temporal requirement and project invariant |
| IDR-SRV-053/054 | temporal fixture corpus, load tests for range/latest/as-of, extent rebuild, retention, cursor stability, and reconnect storms |
| IDR-SRV-056 | cross-client/server comparison with fixed clocks, snapshots, profiles, offsets, open bounds, latest ties, and known defects |

---

## 15. Recommendations

| ID | Recommendation | Priority | Basis |
|---|---|---|---|
| R-018-01 | Implement typed instants/extents and never a generic unlabeled timestamp bag. | Critical | P1/P2/SensorML/SWE [N,P] |
| R-018-02 | Preserve valid/effective and transaction time independently for mutable descriptions and temporal relationships. | Critical | IDR-016/017 [P] |
| R-018-03 | Capture one evaluation time and snapshot/watermark per request and bind them to cursors. | Critical | reproducibility [P] |
| R-018-04 | Preserve source lexeme, scale/frame, precision, uncertainty, and default/correction derivation while emitting canonical UTC where defined. | High | RFC 3339/SWE [N,P] |
| R-018-05 | Apply IDR-SRV-011 field mapping, inclusive intersection, deterministic ordering, and `latest` operator order exactly. | Critical | accepted query baseline [P] |
| R-018-06 | Store unbounded intervals as typed bounds; contain #182 in a representation adapter and never copy query `..` into core resource properties before resolution. | Critical | #182/schema conflict [X,P] |
| R-018-07 | Treat DataStream/ControlStream extents as server-derived, snapshot-consistent aggregates that can be rebuilt from accepted visible facts. | Critical | P2 §§9–10 [N,P] |
| R-018-08 | Retain whether Observation phenomenon time was explicit or defaulted from result time. | High | P2 JSON schema [N,P] |
| R-018-09 | Keep issue, execution, report, receipt, commit, and publication time distinct throughout command/feasibility workflows. | Critical | P2 §§10–11 [N,P] |
| R-018-10 | Model SystemEvent occurrence independently from the transaction that updates related resources and from message publication. | High | P2 §12/IDR-014H [N,D,P] |
| R-018-11 | Implement domain freshness as a versioned evaluation with named evidence and reason; keep availability and validity separate. | Critical | AEP context/IDR-015–017 [A,P] |
| R-018-12 | Never use HTTP `Age`, `Expires`, or Cache-Control as domain freshness or resource lifecycle metadata. | Critical | RFC 9111 [N,P] |
| R-018-13 | Enforce authorization before latest, aggregates, counts, current projection, and freshness metadata. | Critical | IDR-017/security handoff [P] |
| R-018-14 | Use append/audit history and monotonic transaction sequence for ordering; never use UUIDv7 or wall-clock time alone. | Critical | IDR-016 [P] |
| R-018-15 | Bind every retained dynamic record to an immutable schema revision and protect that revision through retention. | Critical | IDR-016/P2 [N,P] |
| R-018-16 | Make #182, OSH #331, clock skew, out-of-order data, cache/domain freshness, and DDIL reconnect first-class fixtures. | High | X/I/P |
| R-018-17 | Keep the public freshness extension and exact policy vocabulary downstream and capability-advertised. | High | scope boundary [P] |
| R-018-18 | Recheck official issue #182 and tagged artifacts immediately before implementation freeze. | High | mutable upstream state [X,P] |

### 15.1 Rejected Simplifications

| Simplification | Why rejected |
|---|---|
| One `timestamp`/`updatedAt` for all resources | destroys domain meaning, query correctness, audit, and replay |
| Latest arrival equals latest observation | out-of-order and replayed data invalidate it |
| Current equals latest | validity and maximum named time are different operators |
| Latest equals fresh | freshness depends on policy and expected cadence |
| Stale equals unavailable | age and reachability/capability are independent |
| Ended validity equals retired/deleted | applicability and lifecycle are independent |
| `now` equals unbounded | evaluation instant and infinity are different |
| Query `..` is valid inside `validTime` | current approved resource authority does not establish it |
| HTTP `Age` communicates sensor age | RFC 9111 defines representation cache age |
| UUIDv7 order is semantic order | identity generation time is not event/result/commit authority |
| Update stream schema in place | breaks retained record decoding and semantic consistency |

---

## 16. Risks, Constraints, and Open Questions

### 16.1 Known Risks and Mitigations

| Risk | Consequence | Mitigation/owner |
|---|---|---|
| `validTime` open-bound ambiguity | schema/prose/server incompatibility | adapter containment, fixtures, upstream recheck; 021/023/050 |
| Same-identity overlapping validity | ambiguous current description | correction/supersession rule, rejection, diagnostic; 025/029/031 |
| Clock skew/unknown scale | false future/order/freshness decisions | provenance, quarantine, trusted clock, uncertainty; 019/031 |
| Derived extent drift | incorrect filters and leakage | atomic update, rebuild, visibility-aware projection; 027/029/040 |
| `latest` applied before filters | wrong multi-result selection | operator-order invariant and #331 regression; 027/050/056 |
| HTTP/domain freshness conflation | unsafe operational decisions | separate types/metadata and tests; 020/045/046 |
| Time-based policy leaks | hidden activity inferred from extents/age | authorization before derivation; 039/040 |
| Retention erases supporting schema/status/tombstones | undecodable data or sync resurrection | dependency-aware retention; 028/030/043 |
| Re-evaluated `now` across pages | duplicates/omissions/inconsistent state | cursor-bound evaluation/snapshot; 029/046 |
| Draft Part 3 changes | transport model churn | transport-neutral clocks/outbox seam; 035 |

### 16.2 Open Questions Routed Downstream

- What public profile vocabulary exposes freshness, `asOf`, completeness, and source availability without leaking protected context? IDR-SRV-020/039/040.
- What uncertainty model and tolerance governs skewed or `-00:00` source timestamps? IDR-SRV-019/031.
- Which database and index shapes best implement valid-time plus transaction-time workloads? IDR-SRV-025/027.
- How are corrections, duplicate observations, late arrivals, and aggregate rebuilds transacted? IDR-SRV-029/031/034.
- Which command deadlines, feasibility validity windows, timeouts, and retry clocks are standardized or profiled? IDR-SRV-036–038.
- Which HTTP cache directives apply per mutable, immutable, authorized, and disconnected response class? IDR-SRV-045/046.
- What final OGC disposition replaces the provisional #182 adapter policy? Recheck before representation freeze.

None of these questions prevents acceptance of the temporal domain baseline; each is deliberately owned by a later indexed topic.

---

## 17. Validation Against Plan Success Criteria

| Plan criterion | Result | Evidence |
|---|---|---|
| Temporal concepts identified/distinguished with anchors | Satisfied | §§3, 5–6 |
| Resource-family requirements mapped | Satisfied | §7 split matrix |
| Required named time families distinguished | Satisfied | §§5–8 |
| Current/history/latest/event/observation/command models distinguished | Satisfied | §8 |
| Freshness/staleness/unavailability/cache/expiration/DDIL documented | Satisfied | §10 |
| Query/persistence/validation/security/conformance/fixture/interoperability implications | Satisfied | §§11–13 |
| Implementation/community evidence incorporated non-normatively | Satisfied | §3.3, §§11.3, 13 |
| Recommendations decision-usable and server-bounded | Satisfied | §15 |
| Downstream handoffs explicit | Satisfied | §14 |
| References explicit and reproducible | Satisfied | §18 and pinned commit |

### 17.1 Methodology Phase Completion

| Phase | Output | Status |
|---|---|---|
| 1. Source collection/framework | authority table and extraction fields | Complete |
| 2. Standards extraction | Part 1/2, Features, SensorML, SWE, HTTP inventory | Complete |
| 3. Resource-family analysis | 21-row split matrix and class summary | Complete |
| 4. Freshness/DDIL analysis | domain/cache separation and offline snapshot contract | Complete |
| 5. Query/persistence/test analysis | mappings, algorithms, logical requirements, fixture corpus | Complete |
| 6. Synthesis | recommendations, risks, handoffs, success validation | Complete |

### 17.2 Acceptance and Next Authorization

The Glaux Project Lead accepted this report on September 13, 2026. The accepted baseline establishes the temporal, validity, current/as-of, freshness, and persistence/test handoffs and authorizes only IDR-SRV-019. Acceptance specifically confirms:

1. the bitemporal valid-time/transaction-time seam;
2. the conservative #182 adapter policy pending OGC resolution;
3. request-scoped evaluation time and cursor snapshot binding;
4. strict separation of domain freshness from HTTP freshness and availability; and
5. the downstream ownership boundaries in §14.

---

## 18. References

### 18.1 Project Sources

- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-018 Research Plan](../IDR%20Plans/idr-srv-018-temporal-validity-and-freshness-model.md)
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
- [IDR-SRV-010 Collections, Resources, Links, and Navigation](idr-srv-010-collections-resources-links-and-navigation-behavior-report.md)
- [IDR-SRV-010A API Versioning and Compatibility](idr-srv-010a-api-versioning-backward-compatibility-and-deprecation-strategy-report.md)
- [IDR-SRV-011 Query Semantics](idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md)
- [IDR-SRV-014A through 014H implementation and evidence reports](.)
- [IDR-SRV-015 Canonical Resource Model](idr-srv-015-canonical-glaux-server-resource-model-report.md)
- [IDR-SRV-016 Identifier, URI, and Lifecycle Strategy](idr-srv-016-identifier-uri-and-resource-lifecycle-strategy-report.md)
- [IDR-SRV-017 Relationship and Linkage Model](idr-srv-017-relationship-and-linkage-model-report.md)
- [CSAPI upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), version 1.9

### 18.2 Standards and Official Artifacts

- [OGC API - Connected Systems - Part 1: Feature Resources, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html)
- [Official CSAPI `v1.0.0` source pin](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [Tagged common `timePeriod.json`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/common/timePeriod.json)
- [Tagged Part 2 Observation schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/observation.json)
- [Tagged Part 2 Command Status schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/commandStatus.json)
- [OGC API - Features - Part 1: Core, 1.0.1](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC SensorML Encoding Standard 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model Encoding Standard 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [W3C/OGC SOSA/SSN](https://www.w3.org/TR/vocab-ssn/)

### 18.3 Protocol and Maintenance Sources

- [RFC 3339: Date and Time on the Internet](https://www.rfc-editor.org/rfc/rfc3339)
- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [RFC 9111: HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111)
- [OGC CSAPI issue #182: definition of valid time is unclear](https://github.com/opengeospatial/ogcapi-connected-systems/issues/182)
- [OpenSensorHub issue #331: phenomenonTime with resultTime=latest](https://github.com/opensensorhub/osh-core/issues/331)

---

**End of IDR-SRV-018 Research Report**

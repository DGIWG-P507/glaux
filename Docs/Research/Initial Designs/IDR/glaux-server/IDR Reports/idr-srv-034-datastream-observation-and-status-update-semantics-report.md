# Section 034: Datastream, Observation, and Status Update Semantics - Research Report

**Topic ID:** IDR-SRV-034<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-034 Datastream, Observation, and Status Update Semantics](../IDR%20Plans/idr-srv-034-datastream-observation-and-status-update-semantics.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Five core questions and all detailed questions concerning dynamic-resource taxonomy, DataStream contracts, Observation facts/results, status and dynamic properties, events, latest/history, temporal/spatial/semantic behavior, validation, provenance, query, persistence, streaming, DDIL, policy, commands, fixtures, performance, and interoperability<br>
**Methodology Used:** Authority-ranked clause and schema analysis of approved OGC API - Connected Systems Parts 1 and 2, SensorML 3.0, SWE Common 3.0, SOSA/SSN, accepted IDR-SRV-001 through IDR-SRV-033 findings, controlled AEP evidence, and commit-pinned implementation/test studies; followed by concept, resource, state, query, persistence, policy, and verification mapping<br>
**Research Time:** Approximately 17 hours of AI-assisted research and synthesis, September 14, 2026<br>
**Primary Source(s):**
- [OGC API - Connected Systems - Part 1](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [IDR-SRV-020 Status, Availability, and System Event Model](idr-srv-020-status-availability-and-system-event-model-report.md)
**Supporting Resources:** [IDR-SRV-018](idr-srv-018-temporal-validity-and-freshness-model-report.md), [019](idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md), [022](idr-srv-022-swe-common-data-component-strategy-report.md), [024](idr-srv-024-units-observed-properties-and-semantic-binding-strategy-report.md), [027](idr-srv-027-time-series-observation-storage-strategy-report.md), [029](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md), [031](idr-srv-031-server-write-and-ingestion-model-report.md), [032](idr-srv-032-publisher-to-server-contract-boundary-report.md), and [033](idr-srv-033-simulator-to-server-contract-boundary-report.md)<br>
**Document Purpose:** Establish the standards-aligned meaning, admission, persistence, selection, and client-query contract for DataStreams, Observations, status updates, dynamic properties, and their derived current/latest views<br>
**Author(s):** OpenAI Codex, for the Glaux Project<br>
**Accepted By:** TBD until controlling-plan owner acceptance<br>
**Acceptance Date:** TBD until accepted<br>
**Date:** September 14, 2026<br>
**Last Updated:** September 14, 2026

---

This report labels evidence as **[N] normative standard**, **[A] accepted project baseline**, **[P] project recommendation/decision proposed for acceptance**, **[I] informative implementation/test evidence**, and **[X] documented conflict, absence, or unresolved gap**.

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Dynamic-Data Semantics Extraction Methodology
5. Concept and Resource-Family Taxonomy
6. DataStream Semantic Model Findings
7. Observation Semantic Model Findings
8. Status Update and Dynamic Property Findings
9. System Event and Event-Like Record Findings
10. Latest-Value and Historical-Value Findings
11. Temporal, Spatial, Semantic, Unit, Nil, Quality, and Provenance Findings
12. Query, Validation, Ingestion, Persistence, and Client Behavior
13. Streaming, DDIL, Command, Policy, Security, and Observability Implications
14. Fixture, Conformance, Performance, and Interoperability Implications
15. Downstream Topic Handoff Matrix
16. Decision Analysis, Recommendations, and Implementation Implications
17. Risks, Constraints, Assumptions, and Open Questions
18. Validation Against This Plan's Success Criteria
19. References

---

## 1. Executive Summary

Glaux Server should treat a **DataStream as a versioned semantic contract and standards-facing homogeneous Observation collection**, not as a table, topic, or generic ingestion channel. Every member Observation is produced by the same System and shares the same observed-property set and result schema. The DataStream binds the System output, observation formats, per-format schema, result structure, semantic definitions, units, constraints, nil meanings, and common relationship context. An incompatible change creates a successor DataStream; it never silently reinterprets stored values. **[N/A/P]**

An **Observation is a durable typed fact**: an act/result identity with phenomenon time, result time, exact parent contract revision, inline or linked result, optional per-act parameters, optional Sampling Feature and Procedure, provenance, validation evidence, and server transaction history. It is not merely a value row. JSON may omit `phenomenonTime`; Glaux materializes the standard schema's `resultTime` default and records that action. `resultTime` is when the result was obtained/generated, not receipt or commit time, and the approved model says it cannot be future-dated. **[N/A]**

A **status update is an Observation on a `type=status` DataStream**. The result remains governed by SWE Common components and stable property/vocabulary definitions. Current status is a request-scoped, policy-aware projection over authorized status Observations; it is not a mutable System field, latest arrival, SystemEvent, source-health signal, service-health result, or CommandStatus. Dynamic Sampling Feature properties follow the same Observation basis and may appear as an as-of snapshot for a requested `datetime`. **[N/A/P]**

A **SystemEvent records a discrete System-domain occurrence** such as calibration, maintenance, relocation, or configuration change. CRUD notifications, Observation arrivals, validation failures, delivery records, source connectivity, audits, and command progress remain in their own planes unless a separately justified domain occurrence warrants a SystemEvent. **[N/A/P]**

Glaux needs four distinct selectors:

1. normative Observation `resultTime=latest`: authorize and apply all conjunctive filters first, then return every visible Observation tied at the maximum `resultTime`;
2. current status: select per dimension primarily by applicable phenomenon time, then explicit correction/source-authority rules and stable tie-breakers;
3. dynamic-property snapshot: select the value applicable at requested `datetime` under a declared property profile, without fabricating an interval; and
4. latest-known location/property: use the most recent applicable authorized evidence and expose freshness separately.

No selector uses arrival time as domain truth. Late valid records enter history; duplicate intent returns the original outcome; corrections create transaction history under one logical Observation identity; deletions/corrections trigger deterministic extent/latest recomputation. **[A/P]**

Current/latest pointers and extents may be materialized, but authoritative facts, revisions, rules, policy versions, and watermarks must make them reproducible and rebuildable. Authorization applies before latest selection, aggregation, counts, extents, freshness, pagination, caching, and publication so hidden facts do not leak. **[A/P]**

Initial implementation should support canonical and nested DataStream/Observation access, exact JSON contract validation, source/transaction evidence, advertised temporal/property/FOI filters, snapshot-bound paging, status Observations, deterministic latest/current projections, and corrections. SWE Text/Binary, direct result-spatial filtering, unit conversion, aggregates, and extra source/quality/staleness filters remain profile-gated. IDR-SRV-035 owns all publication-envelope, channel, replay, acknowledgement, QoS, and Part 3 decisions.

## 2. Scope and Plan Alignment

### 2.1 Scope and exclusions

In scope are DataStreams, Observations/results, status, dynamic properties, SystemEvents, source health, CommandStatus, latest/current/history, time and spatial context, schemas, units, properties, validation, provenance, trust, policy, persistence, queries, normalization, corrections, replay, DDIL implications, publication triggers, and verification handoffs.

Out of scope are final DDL, broker selection, Part 3 implementation, publication mechanics, Command/Feasibility state machines, command safety, numeric retention, final policy vocabulary, UI, and server code.

### 2.2 Research question coverage

| Plan Question | Coverage | Evidence |
|---|---|---|
| Q1. Normative/implementation semantics and distinctions | Complete | §§5–10 |
| Q2. Definitions, results, time, units, properties, features, SWE | Complete | §§6–8, 11 |
| Q3. Ingestion record to standards fact | Complete | §§7–8, 12 |
| Q4. Query, paging, validation, persistence, provenance, policy | Complete | §§10–13 |
| Q5. Streaming, commands, DDIL, security, testing, interoperability | Complete | §§13–15 |

### 2.3 Accepted-baseline reconciliation

IDR-SRV-015/017 control resource/relationship identity; 018 controls clocks, freshness, and latest operator order; 019 provenance/trust; 020 status/availability/event separation; 022–024 SWE/schema/unit/semantic rules; 027–030 temporal persistence, idempotency, projections, and lifecycle; 031–033 the common write pipeline and publisher/simulator contracts.

Part 2 prose makes Observation `phenomenonTime` required while its pinned JSON schema permits omission and documents a `resultTime` default. This report preserves IDR-SRV-018: accept omission only in that representation, materialize the default, and record provenance. **[N/A/X]**

## 3. Evidence Base and Authority Classification

### 3.1 Primary sources

| Source | Version/status/pin | Authority | Anchors | Limitation |
|---|---|---|---|---|
| CSAPI Part 1 | OGC 23-001 1.0 Approved; `v1.0.0` `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2` | N | §§9.2.3, 14.7, 15–16 | Some overview prose overstates Part 2 |
| CSAPI Part 2 | OGC 23-002 1.0 Approved; same pin | N | §§7–9, 12–16; Reqs 3–16, 45–51, 63–67, 79–82, 93–98, 107–127 | Prose/schema/ATS inconsistencies |
| CSAPI development repo | `master` `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f` | X; N only where approved | `api/part2` | Mutable; not substituted for approved 1.0 |
| SensorML | OGC 23-000 3.0 Approved | N | §§8.2.5, 8.2.8–8.2.10 | Procedure/interface model, not persistence |
| SWE Common | OGC 24-014 3.0 Approved | N | §§7–10 | Lineage/security outside scope |
| OGC API Features Part 1 | OGC 17-069r4 1.0.1 | N | §§7.14–7.16 | Part 2 modifies terminology/filters |
| SOSA/SSN | W3C Recommendation dependency | N/context | Observation/property/FOI/result/time | Later drafts cannot rewrite dated dependency |
| AEP baseline | `AC/224(JCGISR)D(2026)0005`, 27 Apr 2026, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C` | A/controlled | accepted IDR-001–005 | Not redistributed |

All sources were accessed September 14, 2026. Official `master`, approved tag, OSH `v2.0.2`, OS4CSAPI phase-9, and Connected Systems Go release state were rechecked. Pins still match upstream-history register 1.11, and CS-Go `v1.0.4` remains latest; no register update is warranted.

### 3.2 Supporting evidence

| Source | Pin/status | Use | Limitation |
|---|---|---|---|
| IDR-SRV-006–033 | accepted | controlling Glaux baseline | project decisions, not extra standards |
| OSH | v2.0.2 `235c0eabf24b6d6137b499b4402943d2794b70e6` | typed stores/latest/query regression evidence | I only |
| Connected Systems Go | v1.0.4 `244f4dd586da685d4d9b75e43f73001028b5bd0e` | observations, schemas, paging, status/events, MQTT/outbox | partial/divergent vocabulary and SWE support |
| OS4CSAPI | phase-9 `754411897173c2ec4debaa9bcf4ed9e0f8a9e230` | fixture/interoperability method | client behavior not obligation |
| SECD evidence | `f018fd129bf0d0d1ce75e68198e3ab4d99d937a0` | round-trip/client evidence | test implementation |
| Draft Part 3 study | accepted IDR-SRV-014H | publication handoff | no adoption decision here |

### 3.3 Material gaps

- no approved generic status vocabulary/current-status endpoint/source-health resource;
- no approved unit-conversion, direct Observation result-spatial filter, correction-history representation, or materialization rule;
- no Part 2 ordering, replay, watermark, backpressure, or publication-envelope contract;
- DataStream required/derived-empty and JSON read/write tensions; and
- known Observation time and SystemEvent schema/prose/ATS conflicts. **[X]**

Only explicit project/profile rules below fill these gaps.

The controlled AEP findings require standards-aligned, securely handled, interpretable sensor information with sufficient context for operational use, but the accessible project evidence does not establish a separate AEP wire format, universal status vocabulary, latest selector, or overwrite rule. Glaux therefore preserves CSAPI/SensorML/SWE semantics and carries AEP specialization through versioned profiles, properties, vocabularies, policy, and conformance fixtures rather than inventing controlled content. **[A/X/P]**

## 4. Dynamic-Data Semantics Extraction Methodology

The analysis (1) inventoried standards resources, fields, associations, schema operations, writes, and filters; (2) mapped subject, clocks, spatial context, result contract, authority, persistence identity, latest effect, disclosure, and consumers; (3) reconciled accepted IDRs; (4) tested late arrival, ties, correction, replay, missing/nil, schema change, hidden evidence, DDIL, and concurrent ingestion; and (5) classified each result N/A/P/I/X.

Criteria were standards alignment, clarity, round-trip interoperability, deterministic queryability, rebuildability, validation robustness, DDIL behavior, policy safety, and testability. Arrival-order last-write-wins, untyped payloads, unversioned schemas/vocabularies, post-aggregation authorization, and untraceable projections fail the rubric.

## 5. Concept and Resource-Family Taxonomy

### 5.1 Precise terms

| Concept | Meaning | Not equivalent to |
|---|---|---|
| DataStream | versioned System-output and homogeneous Observation contract | table, topic, publisher, file |
| Observation | identified typed act/result under one exact contract | bare value or envelope |
| Result | inline typed value/aggregate or external link | whole Observation/schema |
| Status update | Observation on `type=status` stream | current status/availability/health/CommandStatus |
| Dynamic property | time-varying feature property modeled as Observations | arbitrary mutable metadata |
| Current projection | authorized as-of derivation under versioned rule | authoritative history/newest arrival |
| Latest Observation | max visible `resultTime` after scope/filters, all ties | freshest/current/one row |
| SystemEvent | discrete System-domain occurrence | CRUD/audit/delivery event |
| Source health | publisher/adapter/connectivity evidence | represented-System status |
| CommandStatus | task execution report | status Observation |
| Raw/quarantine | intake evidence outside canonical admission | public Observation |
| Aggregate | rebuildable extent/count/rollup/projection | source fact |

### 5.2 Dynamic-data semantics matrix

| Concept | CSAPI/source anchor | Semantic definition/relationships | Time/spatial | Unit/semantic binding | Validation/persistence/query | Latest/publication | Security/test/handoff |
|---|---|---|---|---|---|---|---|
| DataStream | P2 §§9.1–9.6 | one System output; same producer/properties/schema | validTime; derived time extents; common/varying SF/FOI | per-format component contract | versioned metadata + schema endpoint + derived extents | narrow `live`; contract/extent candidates | derived leakage tests; 035/040/050/053 |
| Observation | P2 §§9.7–9.10,16 | one act/result under one DataStream | phenomenon/result; SF/FOI or georeferenced result | exact result/parameter schema | logical ID + revisions; temporal query | `resultTime=latest`; committed revision candidate | sensitive result/source; 035/043/050–056 |
| Status Observation | P2 Table 3; IDR-020 | System/subsystem status fact | Observation clocks and subject | typed status property/code/unit | same pipeline/history | phenomenon-time current selector | readiness leakage/conflict tests; 035/040/042 |
| Dynamic property | P1 §14.7 | observed feature property + optional stream | as-of snapshot; property geometry | Property + stream component | history + derived feature representation | no general latest token | target/location sensitive; 040/043/056 |
| SystemEvent | P2 §12 | discrete System occurrence | event occurrence time | event-type URI | append-oriented event history | event publication candidate | operations disclosure; 035/041/042 |
| Source health | GPC/IDR-032 | publisher/adapter evidence | heartbeat/check clocks | private schema | administrative store | availability input only | restricted; 039/048 |
| CommandStatus | P2 §10.11 | report for one Command | report/execution time | status enum/results | task history | task selector | safety sensitive; 036–038 |
| Raw/quarantine | IDR-023/031 | bytes/intake/failure evidence | source/receipt/validation clocks | claimed/detected profiles | isolated | never latest/ordinary publish | promotion re-runs pipeline; 040/041/052 |
| Projection | IDR-018/020/029 | pointer/value + rule/input/snapshot | selection/evaluation clocks | inherits inputs | derived/rebuildable | meaningful committed change | authorize first; 035/040/043 |
| Aggregate | IDR-027/030 | declared derivation | explicit window/watermark | own property/unit/statistic | derived store | no implicit latest substitute | inference risk; 040/053/054 |

## 6. DataStream Semantic Model Findings

### 6.1 Meaning and contract identity

Part 2 makes a DataStream a SOSA ObservationCollection for live, archived, or both forms of access. Every member comes from the same System and shares observed properties and result schema.[^1] Every stream exposes a format-specific schema operation requiring `obsFormat`.[^2]

Contract identity includes `(DataStream ID, revision, System/revision, output identity, media/profile, result tree, parameter tree, encoding, semantic-registry versions)`. Glaux records a normalized fingerprint. Media variants are advertised only after round-trip equivalence is verified. **[N/A/P]**

### 6.2 Field authority

| Member | Authority | Glaux behavior |
|---|---|---|
| `id` | server | stable canonical ID |
| name/description | authorized publisher | versioned description |
| `type` | publisher/profile | Glaux requires explicit `status` or `observation`; never infer |
| System/output | publisher + server resolution | one producer; immutable when populated |
| Procedure/Deployment/SF/FOI | publisher + relationship validation | stream-level only when common |
| schema on write | contract publisher | compile/register; not ordinary echo field |
| `formats` | server capability | advertise only implemented codecs |
| observed properties/result type/time extents | server-derived | `null` when empty; recompute from accepted facts |
| cadence hints | publisher/profile | freshness input, not evidence |
| `live` | declared source/server rule | live data availability only, not freshness/health/history |

Part 2 requires server generation of observed properties, time extents, and result type, and `null` when empty; it may generate `live` and ignore updates.[^1] Glaux may retain declared schema semantics internally before first data while preserving public empty behavior. **[N/X/P]**

### 6.3 Relationships and result grouping

- System is required and invariant.
- `outputName` resolves into effective SensorML System/Procedure output when used; local name alone is not semantics.
- SensorML inputs/outputs/parameters use ObservableProperty or SWE components; tightly related values use aggregates.[^3]
- common Procedure/Deployment/SF/FOI context lives on the stream; varying Sampling Feature or Procedure lives on Observations.
- proximate Sampling Feature and ultimate FOI remain distinct.
- source/publisher provenance never substitutes for producing System.
- atomic vectors/coherent sampled tuples remain Vector/DataRecord; unrelated properties remain separate streams. SWE DataRecord field names are unique and positional encodings preserve order.[^4]

### 6.4 Contract evolution

Part 2 forbids schema modification after Observations exist; replacement explicitly uses `409`, while update/ATS status is a documented defect.[^5]

1. non-semantic description/link corrections create ordinary conditional revisions;
2. component shape/order/type, property, unit, frame, nil, constraint, encoding, or optionality change creates a successor DataStream;
3. close the predecessor `validTime` where appropriate and link lineage;
4. populated deletion defaults conflict; explicit cascade obeys the advertised profile, holds, audit, and projection rebuild; and
5. `live=false`, disconnect, or archival never deletes history.

After a stream is closed, ordinary live publication is denied, but an explicit authorized profile may still admit a correction or backfill whose domain time belongs to that stream and whose contract remains exact. Historical reads and latest-known values remain available under retention/policy; `live` becomes false or unavailable by its declared rule. Archival changes storage tier and advertised access, not the Observation meaning. Purge/tombstone outcomes recompute every affected projection and publish only the committed lifecycle fact. **[A/P]**

## 7. Observation Semantic Model Findings

### 7.1 Identity and canonical record

Every Observation belongs to one DataStream, can package multiple observed properties, and has abstract phenomenon time, result time, result, optional parameters, and optional Sampling Feature/Procedure.[^6] The approved JSON schema requires server IDs and `resultTime`, allows defaulted `phenomenonTime`, and requires exactly one inline `result` or `result@link` branch.[^7]

The canonical record contains or resolves:

- Observation logical ID and accepted revision;
- parent DataStream ID, exact contract revision, and fingerprint;
- producing System and inherited/common Procedure, Deployment, SF/FOI, property, unit, and frame;
- explicit/defaulted phenomenon time and result time;
- inline typed result or validated external-result link;
- optional parameters validated by `parametersSchema`;
- source, publisher/adapter, message, epoch/sequence, original ID, digest, transform, and simulation/federation context;
- validation artifacts/versions, warnings, and normalization decisions;
- receive, ingest, commit, revision, outbox/publication, and audit correlation; and
- lifecycle/tombstone and policy labels.

A frame/batch item becomes a standards Observation only after exact contract, relationship, time, semantic, authority, policy, and transaction admission. **[A/P]**

### 7.2 Results and parameters

The schema endpoint describes the varying result and optional parameters. Default JSON uses SWE component trees; values must match the parent result/parameter schemas.[^8] An external link is accepted only when its branch, media type, reference/integrity, provenance, and authorization policy pass.

An Observation cannot override stream unit, property, component order/type, code space, frame, nil mapping, or encoding. Conversion writes a separately identified derived Observation/DataStream with source value/unit, rule/version, uncertainty effect, and provenance; it never mutates source truth. **[N/A/P]**

### 7.3 Late data, replay, correction, deletion

- phenomenon time is when the value applies and may be past/future for forecasts;
- result time is when obtained/generated and is not receipt/commit; approved baseline prohibits future result time;
- late/out-of-order valid facts enter history and affect projections only under domain-time selectors;
- same idempotency intent/fingerprint returns its original outcome; same key/different intent conflicts;
- PUT/PATCH correction creates a transaction revision under the same logical Observation and exact permitted contract;
- ordinary queries expose the current accepted logical revision; authorized evidence/history interfaces expose superseded revisions;
- deletion creates a governed tombstone, removes the logical fact from ordinary query/extents after commit, and retains required evidence; and
- correction/deletion of time, result, SF/FOI, Procedure, or link triggers deterministic projection recomputation.

### 7.4 Admission taxonomy

| Outcome | Meaning | Canonical/query/projection effect |
|---|---|---|
| Accepted | authorized validated fact committed | visible subject to policy; projection/publication candidate |
| Accepted normalized | only declared deterministic mapping, e.g. JSON time default | same; transformation recorded |
| Duplicate | same scoped intent/fingerprint decided | original result; no second fact/event |
| Pending dependency | trusted bounded profile awaits parent | not canonical/latest; explicit expiry/reconcile |
| Quarantined | trusted invalid/suspicious evidence | isolated; no ordinary relation/query/publication |
| Permanent reject | malformed/schema/unit/time/reference/authority/policy failure | no fact; bounded problem/audit |
| Transient fail | overload/dependency/transaction uncertainty | no assumed fact; same identity on retry |

## 8. Status Update and Dynamic Property Findings

### 8.1 Status is typed Observation evidence

Part 2 `type=status` streams contain status Observations about the parent System or subsystem.[^1] SWE Common covers sensor-related status/ancillary data and typed components, semantics, constraints, quality, and nils.[^9] Glaux therefore rejects universal untyped status JSON.

Use Boolean for binary dimensions, Category with controlled code space for modes, Quantity with unit for measured health properties, DataRecord for a cohesive same-context snapshot, and explicit quality/nil components. System mode, physical condition, sensing readiness, source connectivity, freshness, stream liveness, command acceptance, authorization, and API health remain independent. **[A/P]**

### 8.2 Current status selector

For every requested dimension:

1. capture evaluation time, repository snapshot/watermark, subject/profile, and authorization/policy version;
2. resolve eligible status contracts and source authorities;
3. filter by subject, component, applicability, lifecycle, quality acceptance, and disclosure;
4. select greatest applicable phenomenon time, then governed correction/source precedence;
5. preserve conflict/unknown when claims remain incomparable; use stable ID/transaction ties only for deterministic representation, not physical truth; and
6. evaluate freshness from evidence time, cadence/profile, source state, and purpose, retaining supporting IDs/rules.

Late older status improves history but normally not current. Equal-time higher-authority correction may. Last-known becomes stale; lost contact is `unknown` unless evidence proves `unavailable`. **[A/P]**

`degraded` is a profile-defined assessment for a named capability/dimension, not a fallback string for any warning. A simulated status sample remains an ordinary typed Observation with immutable synthetic/non-operational provenance; it does not require or justify a universal `simulated` result token. **[A/P]**

### 8.3 Dynamic properties and snapshots

Part 1 §14.7 models dynamic Sampling Feature properties as Observations and permits an as-of snapshot when `datetime` is supplied.[^10]

- Observation remains authoritative;
- Property and stream component define meaning;
- snapshot traces selected Observation, time, revision, rule, snapshot, and policy;
- an instant does not imply a continuous interval without declared carry-forward/interpolation and maximum-gap behavior;
- absent, nil, stale, hidden, and not-applicable remain distinct; and
- location/orientation preserves reference frame and is not feature transaction `updatedAt`.

Static SensorML capability/characteristic assertions describe design; measured operating values are Observations. A metadata correction changes description, while a status Observation reports state. **[N/P]**

### 8.4 Status-change records

A status change is normally the new Observation plus a derived projection change. Glaux may create an outbox `status-projection-changed` event after commit. A public SystemEvent exists only when an authorized profile defines a genuine domain occurrence with event identity/type/time/evidence. Two sampled values do not prove the exact transition instant.

## 9. System Event and Event-Like Record Findings

| Record/occurrence | Canonical family | SystemEvent? | Rationale |
|---|---|---|---|
| calibration, maintenance, replacement, relocation, configuration/software change, deployment/decommission | SystemEvent | yes when authorized/typed | Part 2/SensorML history purpose |
| Observation/status accepted/corrected/deleted | Observation + transaction/outbox | no by default | data/resource change, not System occurrence |
| threshold/anomaly | derived fact/alert | conditional | only defined genuine event type |
| source connect/disconnect/queue lag | source health/telemetry | no | administrative evidence |
| validation reject/quarantine/promotion | validation/audit | no | noncanonical intake |
| CRUD lifecycle | revision + outbox/audit | no by default | API mutation notification |
| Command progress/outcome | CommandStatus/Result | no by default | task processing |
| broker retry/dead-letter | delivery ledger | no | transport event |
| policy decision | policy/audit | no | disclosure, not System event |
| federation sync/conflict | sync ledger | no by default | replication evidence |

Event occurrence time, receive/commit/publication time, and correction revision are separate. Events can influence quality/current assessment, but events alone cannot reconstruct status unless a profile explicitly guarantees event-sourced completeness. **[A/P]**

## 10. Latest-Value and Historical-Value Findings

### 10.1 Selector contract

| View | Candidate/order | Ties/conflicts | History/freshness |
|---|---|---|---|
| Observation `resultTime=latest` | route + authorization + all filters, then max result time | return all equal-time facts | latest can be stale; late newer-domain result can replace |
| status current | eligible applicable status facts; normally max phenomenon time | correction/source rules; explicit conflict | last-known may become stale/unknown |
| dynamic property as-of | property/SF facts applicable at requested time under profile | correction/source rule | no fabricated interval |
| latest-known location | latest applicable authorized evidence | frame/quality/conflict preserved | may be stale; absence not zero geometry |
| DataStream extents | min/max over current accepted linked facts | multiple extrema allowed | correction/delete/authorized view may shrink |
| source health | named heartbeat/check rule | source instance/epoch scope | never System status |
| Command current status | report time + legal lifecycle/sequence | owned by IDR-036 | separate from status stream |

Requirement 50 defines the only approved special latest value here: Observation `resultTime=latest`.[^11] Glaux applies every conjunctive filter and authorization before maximum selection, preserving ties. `/observations` is endpoint-wide; the nested route is stream-scoped. It never means one-per-stream, latest arrival, latest phenomenon time, or current status. **[N/A]**

### 10.2 Computed/materialized projections

Glaux may compute uncommon queries, transactionally maintain high-value pointers, update expensive extents/rollups from outbox with watermarks, and cache only safely partitioned authorized representations. Every materialization stores/can reconstruct selection axis, scope, fact IDs/revisions, precedence, policy version, snapshot/watermark, evaluation/computation times, and lag. A stale view is disclosed or recomputed, never treated as authority.

### 10.3 History defaults

No-time-filter collection queries are ordinary paged history, not implicit latest. Accepted default Observation order is `(normalized resultTime, ResourceId)` ascending. “Latest per stream,” downsample, aggregate, and windows require advertised extensions. Ordinary history exposes current accepted logical revisions; superseded revisions/raw/audit use authorized evidence interfaces.

Retention and archival define which authorized historical facts are in the queried tier, but never rewrite their time or result. A retained archive may require an explicit archive/profile route; a tombstoned or purged logical fact is excluded from ordinary latest/extents, and derived views are rebuilt from the surviving authorized set. Policy-specific extents and latest values must not reveal archived or hidden facts that the caller cannot retrieve.

## 11. Temporal, Spatial, Semantic, Unit, Nil, Quality, and Provenance Findings

### 11.1 Temporal contract

| Clock | Meaning/behavior |
|---|---|
| phenomenon | when value applies; explicit/defaulted; may be future |
| result | when obtained/generated; required; UTC scale with optional offset; not future |
| DataStream `validTime` | description applicability, not data coverage |
| extent | domain-time min/max; `null` when empty |
| receive/ingest/commit | immutable server evidence, never CSAPI domain substitute |
| publication/delivery | transport evidence, no effect on domain time/commit |
| evaluation/snapshot | captured once across projection/pages |
| synchronization | DDIL/federation evidence, never source-time rewrite |

Requirement 98 requires UTC scale with optional offset.[^8] Glaux normalizes comparisons while preserving submitted lexical evidence. Any clock-skew profile is versioned and cannot silently relax future-result rejection.

### 11.2 Spatial/FOI contract

- System, Deployment, Sampling Feature, ultimate FOI, and geospatial result describe different subjects.
- Part 2 provides Observation FOI filtering, not a direct general result-geometry filter.
- Spatial discovery may select Systems/Sampling Features first, then traverse relationships/FOI.
- Vector/Geometry results retain CRS/frame, axes, units, and component semantics; coverage/external assets retain georeferencing.
- latest location/as-of snapshots derive from dynamic evidence.
- direct result `bbox`/`geom` filtering is an explicit future profile with extraction, CRS, time, null, indexing, and policy rules.

### 11.3 Semantics/units

SWE requires clear property definitions and a unit/scale for continuous numeric values.[^12] Each result component binds `(path/name, type, property URI/version, unit, code space, statistic, frame/axis, constraint, nil, quality, optionality, order/encoding)`.

Discovery compares semantic identity, not labels. Canonical storage preserves source unit/value. Unknown/local terms require a stable registered namespace/profile and mapping provenance. No arbitrary unresolved string is accepted where resolvability is required.

### 11.4 Nil/missing/invalid/quality

SWE separates optional omission from nil and requires a nil reserved value to map to a reason.[^13] Quality may be number, range, category, or text and static or dynamic.[^14]

| Condition | Treatment |
|---|---|
| optional absent | omit/null only when encoding/component allows; not nil |
| unavailable/missing/out-of-range | registered nil token + resolvable reason |
| valid uncertain value | value + typed quality |
| bad constraint/unrecognized nil | reject/quarantine; no null coercion |
| undecodable bytes | reject before mapping; bounded evidence |
| policy withheld | disclosure result, not source nil |
| stale | assessment over valid historical fact, not nil |

### 11.5 Provenance/trust

Internally each fact binds source/publisher authority, original artifact/digest, transform, exact contract/schema/vocabulary, validation, defaulting, accepted revision, replay/correction, and simulation/federation context. Public profiles expose only safe subsets. Trust influences eligibility/conflict through versioned policy; it does not change schema, domain time, or claimed semantics. Simulated data uses identical standards shapes but non-operational authority.

## 12. Query, Validation, Ingestion, Persistence, and Client Behavior

### 12.1 Query contract

| Need | Route/filter | Initial posture |
|---|---|---|
| DataStream | canonical item/collection; nested System/Deployment | canonical links and relationship scope |
| schema | `/datastreams/{id}/schema?obsFormat=...` | exact supported format |
| Observation | canonical item/collection; nested stream | current logical revision |
| time/latest | `phenomenonTime`, `resultTime`, result `latest` | named fields/exact operator |
| observed property | DataStream property filter | identifiers, not labels |
| FOI | DataStream/Observation FOI filter | accepted ID/UID grammar |
| status type | simple property filter only if advertised | no invented core status endpoint |
| System/Deployment/Procedure | relationship traversal | direct Observation extension deferred |
| unit/source/quality/validation/stale | no approved core filter | profile-gated, policy-sensitive |
| spatial result | no approved direct filter | FOI/SF first; extension deferred |

Advanced filtering covers DataStream phenomenon/result/property/FOI and Observation phenomenon/result/FOI filters.[^15] Unknown parameters follow strict accepted error rules, not silent ignore.

### 12.2 Stable sorting/paging

Default order is `(resultTime, ResourceId)` ascending. Explicit sort uses an allowlist and unique tie-breaker. Cursor binds route, normalized filters/sort, representation/profile, authorization context, evaluation time, snapshot/watermark, and last key. Concurrent ingestion cannot shift pages. Expired/mismatched cursors fail. Counts, links, cursors, latest, and extents share one authorized snapshot. **[A/P]**

### 12.3 Ingestion/validation

1. authenticate and resolve publisher/source/tenant/run;
2. enforce method/path/media/size/rate/decompression/idempotency;
3. resolve exact parent contract before result decoding;
4. parse bounded codec and preserve bytes/digest;
5. map envelope to one typed command without invented meaning;
6. validate static shape, result/parameters, optional/nil/order, semantics, units, frames, constraints, time, SF/FOI/Procedure, authority, and policy;
7. apply only declared normalization and record it;
8. decide duplicate/pending/quarantine/reject/accept;
9. atomically persist fact/revision, provenance, validation, source sequence/inbox, idempotency, selector inputs, outbox, and audit; and
10. return committed or guarded asynchronous status.

Batch ingestion is item-atomic by default. All-or-nothing batches require an explicit bounded profile with size, timeout, ledger, and idempotency rules.

### 12.4 Persistence allocation

| Store role | Content |
|---|---|
| relational metadata | DataStreams/revisions, relationships, schemas, lifecycle |
| temporal facts | Observation IDs/current revisions, domain times, selected typed indexes |
| geospatial | features and declared extracted result geometries with derivation |
| object/blob | large raw/external assets and schemas by digest |
| evidence ledger | source messages, validation/provenance, correction/tombstone, inbox/audit links |
| projections | latest/current/extents/freshness/rollups with rules/watermarks |
| outbox/delivery | publication intents and attempts |

PostgreSQL remains initial authority; TimescaleDB is conditional on benchmark benefit. Storage decomposition cannot alter the logical resource or semantics.

## 13. Streaming, DDIL, Command, Policy, Security, and Observability Implications

### 13.1 Publication handoff

After commit, candidates include DataStream lifecycle/revision, Observation/status revision, meaningful latest/current change, and SystemEvent revision. Outbox records canonical ID/revision, transaction sequence, domain time, parent stream, policy scope, replay/backfill/correction flags, and trace.

Publication order follows commit/outbox order while payload preserves domain clocks. Late facts can publish after newer domain data without becoming current. IDR-SRV-035 owns Part 3 adoption, channels, envelopes, content mode, filtering, snapshot/catch-up, cursors, acknowledgements, QoS, duplicate handling, backpressure, and broker.

### 13.2 DDIL/replay/federation

- preserve source/message/epoch/sequence, domain time, digest, and contract through spool/reconnect;
- accept valid late history and distinguish input replay from publication retry;
- make gaps and unknown reconciliation explicit; sequence order is source/epoch scoped;
- never reconnect-arrival overwrite;
- retain local/federated authority and transform/sync provenance;
- preserve conflict instead of hidden LWW;
- bind projection/publication watermarks; and
- prevent tombstone resurrection without governed revision.

IDR-SRV-042 owns degraded behavior; 043 owns merge, conflict, tombstone, and anti-entropy.

### 13.3 Command boundary

ControlStream schemas govern Commands, not Observations. CommandStatus/Result and current Command status retain their own lifecycle. A readiness status Observation does not authorize a Command; `ControlStream.live` proves neither safety nor feasibility; an Observation referenced by CommandResult remains separately identified. IDR-SRV-036–038 own task lifecycle and safety.

### 13.4 Policy/releasability

Order is: **authorize candidates, then filter, select, aggregate/count/extent/freshness, page, format, cache, and publish**. Hidden newest facts cannot affect visible latest or leak via ranges, counts, no-result behavior, links, timing, or liveness.

Derived outputs inherit material-input restrictions unless a recorded release rule permits otherwise. Redaction must remain schema/profile-valid or use a separate derived stream. Principal-specific results use partitioned/private caches.

### 13.5 Observability

Distinguish received, decoded, validated, accepted, duplicate, rejected, quarantined, pending, committed, corrected, tombstoned, projection-updated/skipped/conflicted, outbox-pending, published, retried, and reconciled. Use bounded metric dimensions, not resource IDs or values. Operators need accepted/result and commit watermarks, projection/publication lag, late distribution, duplicates/gaps/conflicts, rebuild state, schema/codec failures, authorization suppression, backlog, and source health. Telemetry never automatically becomes public status.

## 14. Fixture, Conformance, Performance, and Interoperability Implications

### 14.1 Fixture corpus

| Family | Positive and negative fixtures |
|---|---|
| DataStream | empty/null derived fields; both types; every result type; common/varying SF/FOI/Procedure; formats; bad System/output/link; populated schema mutation; successor |
| Observation | scalar/vector/record/complex/coverage/external link; parameters; defaulted phenomenon time; missing result time; both/neither result branches |
| SWE/semantics | Boolean/Category/Quantity/Text/Vector/DataRecord; duplicate fields; optional missing/null; nil reasons; constraints; code spaces; unit/frame/property mismatch |
| temporal | past/future phenomenon; future-result reject; offsets; late/out-of-order; equal result; correction/deletion extrema |
| spatial/FOI | common FOI; per-observation SF; proximate/ultimate FOI; moving location; wrong CRS/axis; hidden geometry; linked coverage |
| status/dynamic | independent dimensions; stale last-known; unknown/unavailable; source conflict; dynamic snapshot/no-value; source/service/task non-collapse |
| identity/replay | duplicate; key/payload conflict; epoch reset; gap; retry ambiguity; mixed batch; ETag correction race |
| query/policy | each filter; filter-before-latest; all ties; paging during ingest; unauthorized newest; hidden count/extent; cursor mismatch |
| lifecycle/DDIL | close/archive/live false; reconnect backlog; correction after publish; tombstone/resurrection; federation conflict; rebuild |
| events | genuine SystemEvent versus CRUD/data/health/audit/delivery; correction/type filter; occurrence versus commit |

Golden artifacts pin approved schema commit, media/profile, semantic/unit/vocabulary registries, canonical form, validation/query/projection result, and intentional defect/extension.

### 14.2 Conformance/regression

1. run every advertised Part 2 DataStream/Observation, filter, write/update, JSON, and selected SWE ATS requirement;
2. supplement `resultTime=latest`, which approved ATS omits;
3. test canonical/nested scopes, links, schemas, empty fields, all ties, and no implicit latest;
4. validate writes against exact parent contract and schema-change conflicts;
5. round-trip fields/components/associations through create, retrieve, restart, correction, query, and delete;
6. test filter-before-latest and OSH issue #331 pattern;
7. verify authorization before extent/latest/count/page/cache;
8. prove duplicate, retry, late, correction, tombstone, and rebuild; and
9. enforce status/source health/SystemEvent/CommandStatus non-collapse.

### 14.3 Performance/resilience

Measure sustained/burst ingest by shape/media; contract-cache and validation cost; temporal/FOI query; latest/current/extents under late/correction load; paging during ingest; projection/outbox lag; policy cost; restart/rebuild; DDIL backlog; hot-stream skew; linked-result metadata; batches; and backpressure. Report receipt, durable commit, query visibility, and publication latency/throughput separately.

### 14.4 Interoperability

Exercise CSAPI Explorer, OS4CSAPI, OSH-compatible clients, CS-Go comparison fixtures, web/mobile clients, and an independent generated client for discovery/navigation, schema, JSON/SWE shapes, field/link preservation, filters/ties/order/paging/latest, status/dynamic snapshots, validation/replay/correction/conflict, and post-IDR-035 publication/history consistency.

CS-Go demonstrates persistence, schema checks, time ranges, paging, status/events, MQTT, and CloudEvents, but its broad `latest`, query names, cursors, and incomplete SWE support are not standards behavior. **[I]**

## 15. Downstream Topic Handoff Matrix

| Topic | Fixed input from 034 | Still owned downstream |
|---|---|---|
| 035 Streaming/publication | committed facts vs projection changes; domain/commit order; flags/watermarks/policy-first | Part 3, channels, envelopes, QoS, catch-up, ack, broker/backpressure |
| 036 Command lifecycle | CommandStatus separate; referenced Observation remains typed | transitions/cancel/update/timeout/results |
| 037 Feasibility | no status substitution; typed result/provenance | lifecycle, validity, assumptions, alternatives |
| 038 Command safety | readiness/live independent and not authority | gates, interlocks, dispatch, audit |
| 039–041 Security/policy/audit | authorize before derivation; evidence separation | auth architecture, release, audit integrity |
| 042 DDIL | late history, domain selectors, gaps/watermarks | degraded read/write/publication |
| 043 Synchronization | epoch/sequence, revisions, conflict, tombstone | merge/authority/anti-entropy |
| 048 Observability | semantic outcomes and watermarks/lag | concrete telemetry/health/SLO |
| 050 Conformance | exact classes and supplemental tests | harness implementation |
| 053 Fixtures | fixture families and pins | packaging/generators/golden governance |
| 054 Performance | workloads and distinct outcome clocks | tooling/environments/targets |
| 055 Security tests | hidden latest/count/extent/cache/non-collapse | adversarial matrix/thresholds |
| 056 Interoperability | client/resource/encoding/query/status scenarios | versions/topology/pass policy |

Immediate next step after plan-owner acceptance is IDR-SRV-035 research by the Glaux research workflow. Until acceptance, the report remains a proposal, IDR-SRV-035 remains unauthorized, and no streaming/Part 3 or server implementation begins.

## 16. Decision Analysis, Recommendations, and Implementation Implications

### 16.1 Decision analysis

| Decision | Option | Benefit | Risk/cost | Disposition |
|---|---|---|---|---|
| Stream meaning | generic channel/table | simple | breaks semantics/interoperability | Reject |
| Stream meaning | versioned semantic contract | aligned/reproducible | registry/lifecycle | Adopt |
| Status | mutable untyped System JSON | easy UI | collapsed history/meaning | Reject |
| Status | typed facts + projection | provenance/history | selector/profile | Adopt |
| Latest | arrival LWW | cheap | wrong under replay/DDIL | Reject |
| Latest | named domain selector | deterministic | indexes/projections | Adopt |
| Evolution | mutate populated schema | stable ID | reinterprets history | Reject |
| Evolution | successor stream | immutable meaning | navigation work | Adopt |
| Projection | materialized-only | fast | stale/opaque authority | Reject |
| Projection | rebuildable materialization | fast/correct | invalidation/watermark | Adopt |
| Units | implicit overwrite conversion | convenient | lost provenance | Reject |
| Units | source + explicit derivation | explainable | extra derived stream | Adopt |
| Spatial | guess result geometry | quick | ambiguous/policy unsafe | Reject |
| Spatial | FOI/SF + explicit extension | predictable | advanced feature deferred | Adopt |

### 16.2 Recommendations

| ID | Recommendation | Priority |
|---|---|---|
| R-034-01 | Model DataStream as versioned homogeneous semantic contract/resource, never transport/storage alias. | Critical |
| R-034-02 | Bind Observation to exact parent revision and preserve logical ID, transaction revisions, evidence, provenance. | Critical |
| R-034-03 | Require explicit stream `type` in Glaux profiles and model status as typed Observations. | Critical |
| R-034-04 | Keep status, dynamic property, SystemEvent, source health, CommandStatus, audit, and delivery separate. | Critical |
| R-034-05 | Apply filters/authorization before `resultTime=latest` and return all max-time ties. | Critical |
| R-034-06 | Use phenomenon-time applicability and explicit authority/correction for current status, never arrival overwrite. | Critical |
| R-034-07 | Make current/latest/extents/freshness rebuildable with facts, rules, policy, snapshots, watermarks. | Critical |
| R-034-08 | Create successor stream for incompatible shape/property/unit/frame/nil/optional/order/encoding changes. | Critical |
| R-034-09 | Preserve optional, nil, invalid, stale, unknown, and withheld as distinct. | Critical |
| R-034-10 | Preserve source units; conversion is explicit provenance-bearing derivation. | High |
| R-034-11 | Authorize before filtering, selectors, aggregates, extents, counts, paging, cache, publication. | Critical |
| R-034-12 | Implement required routes/filters first; gate source/unit/quality/stale/spatial extensions. | High |
| R-034-13 | Use item-atomic batches; keep nonaccepted outcomes out of canonical projections. | High |
| R-034-14 | Publish only after commit and distinguish fact revisions, projection changes, and delivery attempts. | Critical |
| R-034-15 | Preserve epoch/sequence/domain clocks/gaps/conflicts/tombstones without global-order claims. | Critical |
| R-034-16 | Build §14 verification corpus before production write/publication readiness. | High |
| R-034-17 | Recheck schemas, ATS errata, registries, implementations, and Part 3 at implementation freeze. | High |

### 16.3 Implementation implications and estimates

| Work package | Complexity | Estimate | Assumptions |
|---|---|---|---|
| stream contract/registry/fingerprint/schema/successor | High | 3–5 developer-weeks | JSON/SWE JSON first |
| Observation model/admission/revision/temporal indexes | High | 5–8 weeks | PostgreSQL baseline |
| SWE validation/bounded codecs | High | 4–7 weeks | reuse IDR-022/023 |
| selectors/extents/status/dynamic snapshots/rebuild | High | 4–7 weeks | policy interface |
| filters/sort/snapshot paging/policy-first derivation | High | 4–6 weeks | IDR-011/027 |
| provenance/idempotency/outbox/telemetry integration | Medium–High | 3–5 weeks | shared services |
| fixtures/conformance/DDIL/policy/interoperability tests | High | 5–8 weeks | parallelizable |

Ranges are planning guidance, not commitments; shared foundations overlap. Text/Binary, direct result-spatial query, aggregates, Part 3, and hardening are separate increments.

## 17. Risks, Constraints, Assumptions, and Open Questions

### 17.1 Risks and controls

| Risk | Consequence | Control |
|---|---|---|
| stream as transport/storage | semantic drift | contract/fingerprint invariants |
| status/event/health collapse | unsafe inference | taxonomy/non-collapse tests |
| arrival latest | wrong current after replay | named selectors/ties |
| schema mutation | reinterpreted history | conflict + successor |
| implicit conversion | unverifiable result | source + explicit derivation |
| null/nil/invalid/stale collapse | false values | component-aware fixtures |
| projection-only authority | unrecoverable state | facts/rules/rebuild |
| post-aggregation auth | data leakage | policy-first evaluation |
| shifting pages | duplicates/omissions | snapshot cursors |
| publication=commit/domain order | loss/wrong subscribers | outbox + both clocks |
| copied schema/ATS defects | interop failure | overlays/fixtures/recheck |
| high-cardinality telemetry | cost/leakage | bounded dimensions |

### 17.2 Constraints/assumptions

- Advertise only implemented classes/media/profiles within the accepted all-class direction.
- Controlled AEP remains non-redistributable.
- PostgreSQL is initial authority; TimescaleDB remains conditional.
- GPC-v1/common-pipeline decisions remain accepted.
- Redaction is profile-valid or a separate derived stream.
- No Part 3 or broker authorization occurs here.

### 17.3 Open questions

1. First Glaux/AEP status-property and event vocabulary: semantic/profile work with 040/053.
2. Which projections merit synchronous materialization: benchmark in 054.
3. Initial SWE JSON versus Text/Binary scope: implementation freeze using 022/023.
4. Need for direct result-spatial query beyond SF/FOI: validate in 056.
5. Authorized Observation revision-history representation: lifecycle/security/API design.
6. Whole-record suppression versus a separate redacted multi-component profile: 040; silent partial objects prohibited.

None blocks this baseline or IDR-SRV-035 research.

## 18. Validation Against This Plan's Success Criteria

| Criterion | Status | Evidence |
|---|---|---|
| Distinguish all dynamic concepts | Met | §§5,8–10 |
| DataStream relationships/SensorML/SWE/encoding/source | Met | §6 |
| Observation time/spatial/semantic/unit/nil/quality/provenance/query | Met | §§7,11–12 |
| Status/dynamic/event/latest/freshness/stale | Met | §§8–10 |
| Query/policy/DDIL/streaming/commands/testing/interoperability | Met | §§12–15 |
| Implementation lessons remain informative | Met | §§3.2,14.4 |
| Bounded decision-usable recommendations | Met | §16 |
| Explicit handoffs | Met | §15 |
| Reproducible references | Met | §§3,19 |

All six research phases, drafting, and internal review are complete. Plan-owner acceptance remains unchecked while In Review.

## 19. References

### 19.1 Standards and authoritative sources

- [CSAPI Part 1, OGC 23-001 1.0](https://docs.ogc.org/is/23-001/23-001.html)
- [CSAPI Part 2, OGC 23-002 1.0](https://docs.ogc.org/is/23-002/23-002.html)
- [Approved CSAPI source pin](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [Pinned DataStream schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/dataStream.json)
- [Pinned Observation schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/observation.json)
- [Pinned Observation result schema model](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/observationSchemaJson.json)
- [SensorML 3.0, OGC 23-000](https://docs.ogc.org/is/23-000/23-000.html)
- [SWE Common 3.0, OGC 24-014](https://docs.ogc.org/is/24-014/24-014.html)
- [OGC API Features Part 1, OGC 17-069r4](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [W3C SSN](https://www.w3.org/TR/vocab-ssn/)
- [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339)
- Controlled AEP `AC/224(JCGISR)D(2026)0005`, 27 Apr 2026, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`

### 19.2 Accepted project sources

- [IDR-SRV-007](idr-srv-007-csapi-part-2-requirement-baseline-report.md), [011](idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md), [015](idr-srv-015-canonical-glaux-server-resource-model-report.md), [017](idr-srv-017-relationship-and-linkage-model-report.md), [018](idr-srv-018-temporal-validity-and-freshness-model-report.md), [019](idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md), [020](idr-srv-020-status-availability-and-system-event-model-report.md), [021](idr-srv-021-sensorml-representation-strategy-report.md), [022](idr-srv-022-swe-common-data-component-strategy-report.md), [023](idr-srv-023-schema-and-encoding-validation-strategy-report.md), [024](idr-srv-024-units-observed-properties-and-semantic-binding-strategy-report.md), [027](idr-srv-027-time-series-observation-storage-strategy-report.md), [029](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md), [030](idr-srv-030-data-lifecycle-retention-archival-and-deletion-strategy-report.md), [031](idr-srv-031-server-write-and-ingestion-model-report.md), [032](idr-srv-032-publisher-to-server-contract-boundary-report.md), and [033](idr-srv-033-simulator-to-server-contract-boundary-report.md)

### 19.3 Implementation/test evidence

- [OSH v2.0.2](https://github.com/opensensorhub/osh-core/tree/235c0eabf24b6d6137b499b4402943d2794b70e6)
- [Connected Systems Go v1.0.4](https://github.com/SomethingCreativeStudios/connected-systems-go/tree/244f4dd586da685d4d9b75e43f73001028b5bd0e)
- [OS4CSAPI phase-9](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230)
- [SECD evidence](https://github.com/Sam-Bolling/csapi-server-interop-secd/tree/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0)
- [CSAPI Explorer](https://ogc-csapi-explorer.pages.dev/)

### 19.4 Footnotes

[^1]: CSAPI Part 2 §§9.1–9.2, Tables 2–5: one System, shared properties/schema, derived properties, empty `null`, and `status`/`observation` types. Accessed 2026-09-14.
[^2]: CSAPI Part 2 §9.6, Requirement 11: every DataStream exposes `/schema` with required `obsFormat`. Accessed 2026-09-14.
[^3]: SensorML 3.0 §§8.2.9–8.2.10, Requirements 11–12: ObservableProperty/SWE inputs, outputs, parameters, and aggregates. Accessed 2026-09-14.
[^4]: SWE Common 3.0 §8.3.1, Requirement 37 and encoding clauses: unique DataRecord field names and declared positional order. Accessed 2026-09-14.
[^5]: CSAPI Part 2 §§14.2,15.2, Requirements 64/80: populated schema protection; update/ATS status conflict is recorded by IDR-SRV-007. Accessed 2026-09-14.
[^6]: CSAPI Part 2 §9.7, Tables 6–7: Observation properties and associations. Accessed 2026-09-14.
[^7]: Approved `observation.json` at commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`: phenomenon default, result time, and inline/link branch; reconciled by IDR-SRV-018. Accessed 2026-09-14.
[^8]: CSAPI Part 2 §§16.1.4–16.1.5, Requirements 96–98: parent result/parameter schemas and UTC time. Accessed 2026-09-14.
[^9]: SWE Common 3.0 §§4.7–4.8,7.1–7.4: sensor-related status/ancillary data and component context. Accessed 2026-09-14.
[^10]: CSAPI Part 1 §14.7: dynamic properties as Observations and requested-time snapshots. Accessed 2026-09-14.
[^11]: CSAPI Part 2 §13.3.2, Requirement 50: Observation `resultTime=latest`; approved ATS omission recorded by IDR-007/018. Accessed 2026-09-14.
[^12]: SWE Common 3.0 §§7.2–7.3, Requirements 4/7/8: units/scales and clear/resolvable semantics. Accessed 2026-09-14.
[^13]: SWE Common 3.0 §§7.4.2,8.2.16, Requirements 11/33: nil values map to defined reasons; optionality is separate. Accessed 2026-09-14.
[^14]: SWE Common 3.0 §§7.4.1,8.2.15: typed static/dynamic quality. Accessed 2026-09-14.
[^15]: CSAPI Part 2 §§13.2–13.3, Requirements 45–51: DataStream/Observation temporal, property, and FOI filters. Accessed 2026-09-14.

---

## Report Completion Checklist

- [x] Topic ID matches overall research plan index
- [x] Topic plan linked/aligned and questions covered
- [x] Findings have reproducible, authority-classified references
- [x] Mutable sources have exact pins and controlled/gap limitations
- [x] Findings, accepted baseline, proposals, and implementation evidence are distinct
- [x] Prior-report conflicts are reconciled
- [x] Executive summary and recommendations are decision-usable
- [x] Risks, success criteria, handoffs, and owners are documented
- [ ] Plan-owner acceptance and date remain pending

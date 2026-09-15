# Section 027: Time-Series Observation Storage Strategy - Research Report

**Topic ID:** IDR-SRV-027<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-027 Time-Series Observation Storage Strategy](../IDR%20Plans/idr-srv-027-time-series-observation-storage-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions, all detailed question groups, all 6 methodology phases, all 10 success criteria, and the required 15-field time-series matrix<br>
**Methodology Used:** Authority-ranked extraction from approved CSAPI, OGC API - Features, SensorML, SWE Common, controlled-source findings, and accepted IDR-SRV-001 through IDR-SRV-026; direct review of current primary technology documentation; implementation-evidence comparison; temporal-category/workload classification; and bounded synthesis into an authoritative-core strategy with explicit downstream gates<br>
**Research Time:** Approximately 16 hours of AI-assisted execution on September 14, 2026<br>
**Official Standards Source Pin:** [`opengeospatial/ogcapi-connected-systems` `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Shared Register Baseline:** [OGC API - Connected Systems upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), Version 1.10; no approved-standard time-series change required a register update<br>
**Technology Documentation Snapshot:** PostgreSQL 18; TimescaleDB 2.27.1; current InfluxDB 3 Enterprise, QuestDB, ClickHouse, DuckDB, SQLite, Apache Kafka 4.1, NATS JetStream, and Rust SQLx documentation; checked September 14, 2026 and treated as mutable implementation evidence<br>
**Controlled AEP Source:** `AC/224(JCGISR)D(2026)0005`, April 27, 2026, SHA-256 `56dc757b6e677b3584e3152a957849f21a24b22854f562613ff283a8b599da8c`; used only through accepted project findings and not redistributed<br>
**Document Purpose:** Establish the Glaux Server time-series authority, temporal-record, query/index, partition, latest-value, retention, replay, DDIL, policy, and test baseline without defining final DDL, implementing draft Part 3, or implementing the server<br>
**Author:** OpenAI Codex<br>
**Accepted By:** TBD pending Glaux Project Lead review<br>
**Acceptance Date:** TBD<br>
**Date:** September 14, 2026<br>
**Last Updated:** September 14, 2026

---

## Reading Guide and Evidence Labels

| Label | Meaning |
|---|---|
| **N** | Normative or standards-derived behavior |
| **A** | Project-controlling AEP/STANAG adoption or operational-context finding |
| **P** | Glaux architecture direction or recommendation proposed for acceptance here |
| **I** | Informative technology, implementation, test, or community evidence |
| **D** | Detailed design deferred to the named downstream topic |
| **X** | Evidence gap or unresolved decision requiring profile input, prototype, or benchmark |

“Time-series” does not mean one table, one timestamp, or one retention policy. Observation facts, current-status projections, System Events, CommandStatus reports, feasibility progress, ingestion evidence, synchronization operations, audit events, and aggregates have different authorities, time axes, correction rules, sensitivities, and lifecycles.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Time-Series Requirement Extraction Methodology
5. Time-Series Data-Category Inventory
6. Resource-Family Temporal Storage Matrix
7. Time Field and Temporal Semantics Findings
8. Query, Filtering, Sorting, Pagination, Latest-Value, Replay, and Backfill Findings
9. Indexing, Partitioning, Retention, Archival, Compression, and Summarization Findings
10. Time-Series Storage Option Evaluation
11. Observation, Status, Event, Command, Feasibility, and Ingestion Storage Implications
12. Spatial-Temporal, Semantic-Temporal, and Dynamic-Location Implications
13. DDIL, Cache, Synchronization, and Federation Implications
14. Security, Policy, Releasability, and Audit Implications
15. Fixture, Conformance, Performance, and Interoperability Test Implications
16. Downstream Topic Handoff Matrix
17. Recommendations
18. Risks, Constraints, and Open Questions
19. Validation Against Plan Success Criteria
20. References

---

## 1. Executive Summary

Glaux should implement its authoritative time-series baseline inside the **PostgreSQL 18 relational-hybrid core selected by IDR-SRV-025**, with PostGIS for spatial assertions. Native declarative range partitions, B-tree/BRIN/GiST indexes, immutable append histories, transactionally maintained projections, and an outbox are sufficient for the initial full profile. A broker, object store, analytical database, or specialist time-series database must not become a second source of CSAPI truth. **[P]**

The canonical design is a **family of typed temporal stores**, not a universal event table. Observation records, System Events, CommandStatus reports, feasibility status/results, ingestion attempts, synchronization operations, publication/outbox entries, and security audit events retain their own semantics. Current status, latest-known position, stream extents, summaries, and subscription catch-up indexes are derived products with source record IDs, derivation/version, transaction watermark, and policy scope. **[P]**

Observations require separate `phenomenonTime` and `resultTime`; the former may be far past or future, while the latter cannot be future. Glaux additionally preserves receive, ingest, commit, publication, evaluation, and synchronization clocks without relabeling them as domain time. Accepted IDR-SRV-018 rules continue to govern exact source lexeme, offset/scale, precision, uncertainty, one request evaluation time, bitemporal history, out-of-order arrival, and deterministic tie handling. **[N/P]**

For the Observation authority, **range partitioning by normalized `result_time` is the leading physical candidate** because it is required, filterable, used by normative `resultTime=latest`, and cannot legally be future. `phenomenonTime` is unsuitable as the sole partition axis because forecasts can be future and observations may concern the far past. The candidate must be benchmarked against server-controlled commit-time partitioning with realistic late/backfill workloads before DDL is frozen. Retention eligibility is independent of the partition key: old-domain-time data received today is not automatically disposable. **[N/P/D]**

Partitioned uniqueness creates an architectural seam: PostgreSQL unique constraints and TimescaleDB unique indexes must include partition columns, but Glaux UUID identity and source idempotency are global. The recommended conceptual solution is a compact, unpartitioned **identity/admission ledger** inserted atomically with the partitioned payload record. It owns UUID/source-key uniqueness, content hash, contract revision, transaction sequence, correction/supersession relation, and payload locator. IDR-SRV-029 must prove locking, atomicity, and failure recovery. **[I/P/D]**

Query-critical time, identity, relationship, policy, quality, and spatial keys belong in typed columns. Exact SWE/SensorML/source material remains content-addressed evidence; canonical result structure remains bound to an immutable SWE contract revision; selective contract-aware projections may expose approved scalar/property keys. Neither opaque JSON-only storage nor unconstrained entity-attribute-value expansion is acceptable. **[P]**

The accepted Observation order remains `(resultTime, ResourceId)` ascending. `resultTime=latest` is evaluated after authorization, route scope, and all other predicates; it returns every equal-time tie and then applies deterministic ordering/paging. Latest-value caches are optional accelerators, never semantic substitutes. Late, corrected, newly authorized, or deleted inputs must invalidate or recompute affected extents, latest projections, aggregates, and caches. **[P]**

Retention is policy-driven by record class, mission/profile, tenant, releasability, audit/legal need, synchronization acknowledgement, provenance dependency, and the appropriate clock—not a database TTL alone. Raw numerical observations may be summarized only under an explicit, versioned semantic rule; status, event, command, feasibility, quality, nil, and categorical histories must not be averaged. A summary never silently replaces lossless evidence. Archive manifests must preserve identity, hashes, schema/contract versions, policy, time bounds, and restore/query status. **[P/D]**

TimescaleDB 2.27.1 remains a **conditional benchmark candidate**, especially for compression/columnstore, chunk management, and continuous aggregates. Adoption requires measured benefit over native PostgreSQL plus acceptable licensing, upgrade, PostGIS, backup, late-data, retention/aggregate, and edge-operability results. InfluxDB, QuestDB, and ClickHouse are not selected as authoritative stores; DuckDB/Parquet may support reproducible analytics/archive verification; SQLite is reduced-profile only; Kafka and NATS are delivery transports after the transactional outbox. **[I/P]**

The remaining decisions are deliberately bounded: exact DDL and partition intervals, retention durations, archive format/tiering, ingest and correction transactions, public query/API behavior, subscription protocol, tasking state machines, DDIL merge rules, security policy, and performance thresholds belong to their named downstream topics. **[D]**

---

## 2. Scope and Plan Alignment

This report executes `IDR-SRV-027`, the third Category E topic. It identifies time-indexed categories, authority and storage patterns, temporal/query/index requirements, partition and retention direction, option tradeoffs, and downstream gates.

It does **not** define final SQL DDL; set mission retention periods; choose backup/archive products; define ingestion endpoints; finalize Observation API semantics; adopt or implement draft CSAPI Part 3 Publish/Subscribe; define command authorization/state machines; implement DDIL reconciliation; or implement the server. `IDR-SRV-028` and later indexed topics remain unauthorized during this iteration.

### 2.1 Research Question Coverage Matrix

| Plan question | Short form | Status | Evidence location |
|---|---|---|---|
| Q1 | What time-indexed categories must be stored, indexed, cached, summarized, or derived? | Complete | Sections 5-6 |
| Q2 | Which families produce facts, histories, snapshots, caches, and views? | Complete | Sections 5-6, 11 |
| Q3 | Which clocks, indexes, retention rules, queries, and consistency assumptions apply? | Complete with exact DDL/policy deferred | Sections 7-9 |
| Q4 | Which storage options fit the accepted architecture and deployments? | Complete with Timescale benchmark gate | Section 10 |
| Q5 | What downstream implications follow? | Complete | Sections 12-16 |

### 2.2 Decision Boundary

Decided here: typed family-specific append stores; PostgreSQL authoritative baseline; candidate Observation partition axis and benchmark alternative; identity/admission ledger need; temporal column/projection rules; latest/extents semantics; index families; retention and summary invariants; replay/backfill persistence requirements; specialist/embedded/analytical/broker roles; DDIL, security, and test constraints.

Deferred: physical schema and metadata documents (`028`); transaction/idempotency mechanics (`029`); lifecycle schedules/archive/delete (`030`); ingestion and normalization (`031`); endpoint/query implementation (`033`/`034`/`051`); publish/subscribe (`035` and any separately authorized Part 3 work); command/feasibility/security state machines (`036`-`039A`); DDIL conflict algorithms (`042`/`043`); deployment/operations (`044`-`049`); conformance/fixtures/performance/interoperability (`050`-`056`).

---

## 3. Evidence Base and Authority Classification

### 3.1 Controlling and Approved Sources

| Source | Pin/status | Authority | Time-series anchors | Availability/limit |
|---|---|---|---|---|
| OGC API - Connected Systems Part 2, OGC 23-002 | Approved 1.0; source tag `v1.0.0`, commit `8e03b236...` | Normative | DataStreams/Observations §8; ControlStreams/Commands/CommandStatus/feasibility §9-10; System Events §11; System History §12; Advanced Filtering §14; JSON/SWE encodings §20-21 | Public and locally inspected |
| OGC API - Connected Systems Part 1, OGC 23-001 | Approved 1.0; same source pin | Normative | resource relations; valid time; System latest-known location; nested routes; feature behavior | Public and locally inspected |
| OGC API - Features Part 1, OGC 17-069r4 | Approved corrigendum | Normative dependency | `datetime`, `bbox`, `limit`, collections, links, conformance | Public |
| SensorML 3.0, OGC 23-000 | Approved | Normative representation | process descriptions, valid time, events, outputs, provenance | Public; IDR-SRV-021 layering controls |
| SWE Common 3.0, OGC 24-014 | Approved | Normative data model | Time/TimeRange, DataRecord/DataArray, nil/quality/units, text/binary encodings | Public; IDR-SRV-022 contract model controls |
| RFC 3339 | Internet Standard | Normative timestamp syntax dependency | offsets, fractional seconds, unknown offset, leap-second syntax | Public |
| Controlled project AEP source | `AC/224(JCGISR)D(2026)0005`, 2026-04-27, recorded hash | Project-controlling within handling bounds | operational adoption, dynamic-data, DDIL, security context through accepted reports | Controlled; no redistribution; no unaccepted duration inferred |
| Accepted IDR-SRV-001 through 026 | Accepted by 2026-09-14 | Project baseline | obligations, routes, query, identity, graph, time, provenance, status/events, representations, validation, semantics, persistence, geospatial | Local and reproducible |

### 3.2 Current Technology Primary Sources

| Source | Snapshot | Evidence class | Decision use / limitation |
|---|---|---|---|
| PostgreSQL documentation | 18 | Mutable primary technical | partitioning/pruning, B-tree/BRIN/multicolumn indexes, timestamp/range types, materialized views; not a workload benchmark |
| TimescaleDB documentation/release | 2.27.1, released 2026-05-19 | Mutable primary technical | hypertables, time/space indexes, uniqueness, retention, Hypercore/columnstore, continuous aggregates; edition/license and policy interactions require validation |
| InfluxDB documentation | InfluxDB 3 Enterprise current docs | Mutable primary technical | append/duplicate behavior, last-value cache, object storage, enterprise boundary; different identity/transaction model |
| QuestDB documentation | Current docs checked 2026-09-14 | Mutable primary technical | designated timestamp, partitions, `LATEST ON`, ASOF, TTL, dedup; one designated axis and row-delete limits constrain fit |
| ClickHouse documentation | Current 2026 documentation | Mutable primary technical | MergeTree analytics, materialized views, TTL/rollups; analytical rather than relational transactional authority |
| DuckDB documentation | Current docs checked 2026-09-14 | Mutable primary technical | Parquet pushdown/partitioned export and local analytics; process-concurrency limits exclude full authority |
| SQLite documentation | Current docs checked 2026-09-14 | Mutable primary technical | WAL/read concurrency and one-writer/local-host boundary; reduced profile only |
| Kafka 4.1 and NATS JetStream documentation | Current docs checked 2026-09-14 | Mutable primary technical | partition/stream order, retention, compaction, consumers, acknowledgements; delivery evidence, not domain truth |
| Rust SQLx documentation | Current crate docs checked 2026-09-14 | Mutable primary technical | PostgreSQL time/JSON mappings; application-library choice remains downstream |

### 3.3 Implementation and Community Evidence

- IDR-SRV-014A through 014G remain informative. OSH implementation/regression evidence, including issue #331, shows why phenomenon-time filtering plus `resultTime=latest` needs explicit operator order and tie fixtures.
- Connected Systems Go demonstrates implementable PostgreSQL-oriented CSAPI resource patterns but does not establish normative behavior or Glaux scale thresholds.
- pygeoapi/SECD/OS4CSAPI evidence shows that endpoints may appear interoperable while filters, pagination, or representation populations diverge; every time filter needs positive, negative, combined, malformed, and continuation controls.
- Draft CSAPI Part 3 implementations may inform IDR-SRV-035 or a separately authorized plan, but they do not alter the approved Part 1/2 storage baseline here.

### 3.4 Evidence Quality, Conflicts, and Limits

Standards and accepted project decisions control; AEP findings apply within handling constraints; official technology documentation proves capability, not performance; code/tests/discussions provide informative evidence only. The approved CSAPI source pin remains stable, and the shared history register remains Version 1.10.

The published standard/OpenAPI family contains known temporal seams already reconciled by IDR-SRV-011/018: broad shared schemas expose `now`/`latest` more widely than approved family rules, and CommandStatus inherited filtering needed a `reportTime` project mapping. This report does not broaden special tokens: only Observation `resultTime=latest` is normative in the accepted baseline.

No source supplies Glaux deployment volumes, retention durations, security markings, or acceptable latency. Those remain profile/workload inputs and benchmark gates. The controlled AEP is not quoted or redistributed, and this report does not invent missing retention policy.

---

## 4. Time-Series Requirement Extraction Methodology

Each temporal category was decomposed across the plan’s fifteen required fields. Requirements were then tested against seven non-collapse questions:

1. **What is the fact?** Observation, state report, event, request, result, ingest attempt, delivery record, audit evidence, or summary.
2. **Whose clock?** Phenomenon, result, source issue/report/event, receive, ingest, commit, publication, synchronization, or request evaluation.
3. **What is authoritative?** Supplied domain fact, server transaction fact, exact evidence, derived projection, cache, transport log, or audit record.
4. **Can it change?** Append correction/supersession, lifecycle revision, derived recomputation, acknowledged delivery, or immutable audit entry.
5. **How is it queried?** Route scope, time interval, latest, semantic key, geometry, relationship, policy, order, page, replay cursor, or aggregate.
6. **What owns identity and idempotency?** Resource UUID, source key/sequence, content hash, contract revision, transaction sequence, or transport offset.
7. **When may it leave primary storage?** Only after record-class policy, legal/audit/provenance needs, dependent products, and DDIL peers/cursors are evaluated.

The option rubric used twelve criteria: standards fidelity; multi-axis temporal semantics; relational/spatial/semantic joins; atomicity and global identity; ingest throughput; range/latest/replay performance; partition/retention behavior; compression/summarization; Rust ecosystem; connected/edge deployment; reproducibility/testability; and licensing/operational complexity.

The synthesis retained source obligations as **N/A/I**, proposed bounded project decisions as **P**, and assigned physical/configuration work to explicit **D/X** gates.

---

## 5. Time-Series Data-Category Inventory

### 5.1 Canonical Storage Classes

| Class | Purpose | Mutation model | Examples |
|---|---|---|---|
| Authoritative domain fact | Accepted externally visible fact | append; correction/supersession rather than silent rewrite | Observation, SystemEvent, CommandStatus |
| Authoritative operational fact | Server/source workflow fact | append with explicit outcome | ingest attempt, feasibility result, sync operation |
| Current resource state | Transactional lifecycle/configuration | versioned/bitemporal revision | DataStream definition, Command resource, system description |
| Exact evidence | Reproducible source material | immutable/content-addressed | encoded batch, SWE block, validation input |
| Derived projection | Recomputable view over authorized facts | replace/version atomically | stream extents, latest status/location, summaries |
| Cache | Disposable acceleration | invalidate/rebuild | authorized response fragment, hot latest lookup |
| Delivery log | Publish/ack/cursor evidence | append plus ack state | outbox, subscription delivery attempt |
| Audit trail | Security/administrative evidence | append/tamper-evident policy | access decision, retention action, export |

### 5.2 Required Fifteen-Field Time-Series Data-Category Matrix

| Data category | Related resource family | Source topic / source anchor | Authoritative/derived/cache/log classification | Append-only/current-state classification | Expected volume/velocity | Required time fields | Spatial/semantic links | Query/index needs | Latest-value/backfill/replay needs | Retention/archival needs | Security/policy needs | Candidate storage pattern(s) | Downstream topic handoff | Notes/unresolved issues |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Observation envelope/result | Observation, DataStream | CS2 §8; 018,022,024-026 | authoritative fact plus exact evidence | immutable append; explicit correction/supersession | potentially highest/high | phenomenon, result, receive, ingest, commit, publication | datastream, FOI/SF, procedure, SWE contract, property/unit, optional spatial assertion | stream + result/phenomenon + RID; policy; optional GiST | normative latest by result; late/backfill/replay required | profile-specific; lossless baseline until 030 | result/location/behavior sensitivity; row and derived-policy scope | PG partitioned fact + identity ledger + artifact/JSONB/typed projections | 029-031,034-035,039,043,053-056 | benchmark partition axis/interval and result projection |
| Status observation | status DataStream, System | CS2 §8; 018,020 | authoritative Observation; derived current status | append facts; replaceable projection | medium/high bursts | observation clocks plus freshness evaluation | status property/vocabulary, system, provenance | system/stream + result; status code; policy | last-known may stale; replay history | do not collapse to current; profile policy | operational pattern and state may be restricted | Observation store + current-status projection | 029,031,034-035,040,042 | “latest” is not “healthy” |
| Dynamic property sample | Sampling Feature/System projection | CS1/CS2 dynamic properties; 018,026 | authoritative Observation; derived snapshot | append facts; computed as-of/latest | medium/high | phenomenon/result plus transaction/evaluation | property role, SF/system, optional geometry | subject/property/time; spatial/semantic joins | recompute on late/corrected facts | retain source facts per policy | historical property may be sensitive | Observation store + scoped projection | 031,034,040,042-043 | public snapshot semantics downstream |
| Position/pose sample | System/SF, location DataStream | CS1 location; 026 | authoritative timed sample; derived latest/trajectory | append facts; derived products versioned | high for mobile systems | phenomenon/result/receive/commit; derivation time | spatial assertion, frame, accuracy, system/SF | stream/time + GiST + policy | latest-known, as-of, trajectory rebuild | profile retention; trajectory dependency | precise history often high sensitivity | Observation store + PostGIS assertion/projection | 029,031,034,039,042-043,054 | pose/frame not reducible to point geometry |
| DataStream temporal extents | DataStream | CS2 §8.2.2; 018 | derived aggregate | recomputed/materialized current projection | low writes driven by observations | min/max phenomenon and result; watermark | datastream and visible observation scope | stream lookup; policy-aware variant | affected by late/delete/correction/auth changes | rebuildable; not independent archive truth | extents can leak hidden facts | transactional aggregate or query/materialized view | 029,034,039,054 | null when no linked observations; caller-visible may differ |
| System Event | SystemEvent, System | CS2 §11; 020 | authoritative historical event | immutable append; correction linked | medium/bursty | event extent, receive/commit/publication | system/resource, event type, provenance | system/type/event start/RID; policy | ordered history and replay | often audit/mission-sensitive; no generic TTL | event details/timing may be restricted | PG append event table + exact artifact if supplied | 029,035,039,042-043,053 | not status observation or audit log |
| Command request/resource | Command, ControlStream | CS2 §9; 018,020 | authoritative resource/workflow state plus exact request | versioned resource; immutable request evidence | low/medium, high consequence | issue, requested/actual execution extent, receive/commit | control stream, FOI, controlled properties, contract | stream/issue/execution/status/policy | reconstruct lifecycle; never broker-only | long/audited profile policy | operationally sensitive; integrity critical | PG relational resource + artifact/evidence | 028-029,036,038 | final state machine downstream |
| CommandStatus report | CommandStatus, Command | CS2 §9.7; 018,020 | authoritative status report; derived currentStatus | append-only reports; deterministic projection | low/medium bursts | report, estimated/actual execution, receive/commit | command, result links, status vocabulary | command/report/sequence/RID; status code | full history/replay; ties preserved | no averaging; audit/legal policy | command progress/results are operationally sensitive | PG append table + current-status projection | 029,035-038,039 | legal transition validation downstream |
| Feasibility request/status/result | Feasibility, ControlStream | CS2 §10; 018,020 | authoritative operational records | request/result append; progress history | low/medium | issue/evaluation/report/execution/expiry as defined downstream | control stream, parameters, result/status | request/report/state/time/policy | asynchronous catch-up required | mission/audit profile policy | sensitive tasking evidence | PG workflow tables + exact inputs/results | 029,035,037-039 | do not infer final clock vocabulary here |
| Ingestion batch/attempt | source, adapter, publisher | 019,023,025; plan | authoritative operational evidence | append attempts/outcomes | high correlated with batches | source-produced, receive, ingest start/end, commit | source, adapter version, contract, hashes | source/key/time/outcome/diagnostic | retry/replay/idempotency evidence | raw/quarantine policy distinct | source payloads/diagnostics may be restricted | PG admission/attempt ledger + artifact store | 028-031,048 | no public Observation unless accepted atomically |
| Validation result/quarantine | all writes/imports | 019,022-023 | evidence/log; quarantine record | append result; resolution event | high at boundary | evaluation and commit | artifact, contract/package, diagnostic code | artifact/contract/outcome/time | reproduce and reprocess | policy/size bounded but provenance-safe | safe diagnostics, hostile payload controls | PG evidence metadata + content artifact | 028-031,039,048,053 | quarantine is not domain truth |
| Synchronization operation/conflict | node/federated records | 016,019,025; plan | authoritative sync evidence | append operation/conflict; cursor state | medium/bursty | source domain, receive, sync, commit | node, resource, source ID/hash, policy | node/sequence/resource/time/gap | core DDIL resume and dedup | retain through acknowledgement/conflict/audit policy | cross-domain releasability | PG sync log + per-peer cursor projection | 029,042-043 | merge algorithm deferred |
| Transactional outbox/publication | all publishable changes | 025 | authoritative delivery intent/log | append intent; mutable delivery acknowledgement | tracks accepted mutations | commit, available-after, publication/ack attempt | resource/event type, transaction sequence, payload ref | undispatched/sequence/topic/policy | replay from stable app cursor | bounded after subscribers/audit rules | prevent unauthorized payload/topic leakage | PG outbox + broker projection | 029,035,042-043,048 | store-before-publish; broker offset not truth |
| Security/audit event | all protected operations | 019,025-026 | authoritative audit evidence | append/tamper-evidence | medium/high | event/decision/commit | actor/purpose/resource/policy/version | actor/resource/action/time/outcome | investigation/export replay only | legal/mission policy; deletion controls | highest sensitivity; separation of duties | PG partitioned audit + protected archive | 030,038-039A,048 | not CSAPI SystemEvent |
| Latest/current cache | observation/status/location/command | 018,020,026 | derived cache/projection | replace/recompute with provenance | small, write-amplified | selected domain time, evaluation, watermark | winning fact(s), subject, policy scope | direct subject lookup | invalidate for late/tie/correction/policy | disposable; rebuild required | per-policy; cannot leak existence | transactional table/materialized view/cache | 029,034-035,039,045,054 | equal-time ties may require set, not scalar |
| Numerical summary/downsample | Observation | plan; 022,024-025 | derived artifact | append/version by algorithm/window | lower than raw | bucket bounds, source bounds, generated/commit | contract/property/unit/quality/source set | stream/property/bucket/version/policy | backfill recomputes affected buckets | may outlive raw only by explicit policy | aggregate inference/policy joins | PG materialized table; conditional Timescale; Parquet | 030,034-035,039,054 | count/min/max/nil/quality and method required |
| Archive segment/manifest | all eligible histories | 025; plan | derived package plus authoritative manifest | immutable package; lifecycle manifest | batch | covered clocks, export/archive/verify time | source partitions, hashes, contracts, policy | manifest/time/class/hash/restore state | restore before standards query unless advertised tier | policy-owned lifecycle | encryption/key/releasability/integrity | object/blob/Parquet plus PG manifest | 028,030,039,043-044,049 | no transparent query claim until proven |

### 5.3 Classification Findings

- Observation, status Observation, SystemEvent, CommandStatus, feasibility progress/result, ingestion evidence, synchronization evidence, outbox intent, and audit evidence are distinct append-oriented categories. **[P]**
- DataStream/ControlStream extents, System status, Command `currentStatus`, latest-known location/property, summaries, and cache entries are derived. They identify their input set or reproducible watermark and never erase the underlying distinction. **[N/P]**
- A resource revision is not an event record, and an event record is not a security audit entry. A transport record proves delivery behavior, not that its payload is the authoritative domain fact. **[P]**
- Expected volume/velocity is intentionally qualitative until workload profiles exist. The benchmark corpus must convert “high/medium/low” into explicit rates, cardinalities, payload sizes, retention windows, and concurrency. **[X/D]**

---

## 6. Resource-Family Temporal Storage Matrix

| Resource family | Authoritative temporal records | Derived/current products | Primary public time behavior | Storage/index implication | Non-collapse rule |
|---|---|---|---|---|---|
| System | description revisions and linked facts | current/as-of description, latest-known location/status | `datetime`/history and latest-known projections | bitemporal revisions; join projections by system/policy | creation/update time is not location/status time |
| Procedure | description revisions | current/as-of description | validity/history as applicable | low-volume revision store | no dynamic samples invented |
| Deployment | description/association revisions | current/as-of discovery view | valid time and feature temporal query | revision/range indexes | deployment period is not every observation time |
| Sampling Feature | revisions plus dynamic-property/location observations | snapshot/latest properties/geometry | `datetime` and dynamic projection | revision plus Observation/PostGIS joins | feature geometry and observed location remain distinct |
| Property | vocabulary/binding revisions | effective semantic lookup | validity/version selection | immutable versioned semantic package | property revision is not measurement time |
| DataStream | stream/contract revisions; linked Observations | phenomenon/result extents, latest projections | extent filters; nested observations | metadata authority plus Observation aggregate/index | extents are derived, not mutable source claims |
| Observation | immutable accepted fact and result | wire representation/cache/summary | phenomenon/result filters; result latest | partitioned facts + identity ledger + contract | domain time is not arrival/commit time |
| ControlStream | stream/contract revisions; linked Commands | issue/execution extents, availability view | issue/execution filters | metadata plus command aggregate indexes | control stream `live` is not command state |
| Command | request/resource and lifecycle links | currentStatus projection | issue/execution/status filters | relational workflow plus status append history | mutable currentStatus is not the status history |
| CommandStatus | immutable report | command current state | report-time history via approved/project mapping | `(command, reportTime, transaction sequence, RID)` | transport ack is not CommandStatus |
| Feasibility | request, progress, terminal result | current evaluation view | asynchronous status/history | workflow facts plus report order | feasibility is not command execution |
| SystemEvent | immutable event record | event collection/page/cache | event-time range/history | system/type/time append indexes | not status, resource revision, or audit |
| Ingestion source/adapter | attempts, batches, source cursors | health/lag metrics | internal/admin | source/key/outcome/time indexes | rejected input is not an Observation |
| Publisher/simulator | source identity/sequence evidence | connection/lag state | internal/admin; later streaming | per-source cursor and outbox | broker delivery is not domain commit |

Normative CSAPI requires DataStream phenomenon/result extents to be generated from linked Observations and set to null when none exist. ControlStream issue/execution extents similarly summarize Commands. These aggregates must be transactionally correct internally; caller-visible versions must be computed or constrained after authorization so hidden records do not leak through min/max values. **[N/P]**

---

## 7. Time Field and Temporal Semantics Findings

### 7.1 Canonical Clock Taxonomy

| Clock | Owner/source | Required treatment |
|---|---|---|
| `phenomenonTime` | observation/source domain | preserve instant and source evidence; may be far past or future; query axis |
| `resultTime` | observation/source domain | required; cannot be future under CSAPI; leading Observation partition/query axis |
| `validTime` | resource description/stream applicability | interval with explicit bounds; version/as-of semantics, not sample time |
| `eventTime` | SystemEvent domain | instant/extent as approved representation; event history axis |
| `issueTime` | Command/server-reported receipt by controlled system | command/control-stream query and extent axis |
| `executionTime` | command domain | optional/planned/actual interval whose meaning depends on lifecycle/status |
| `reportTime` | CommandStatus producer | status-history order; ties require transaction sequence/RID |
| source production time | source/publisher | evidence only unless mapped by a contract to a standard field |
| receive time | Glaux boundary | server observation of arrival; useful for lag, future checks, and audit |
| ingest start/end | validation/normalization pipeline | operational performance/evidence, not domain time |
| commit time + sequence | database transaction | authoritative system-time/watermark and total order |
| publication/delivery time | outbox/broker/subscriber | delivery evidence; may occur repeatedly |
| synchronization time | peer/node operation | DDIL transfer evidence; never overwrites source/domain time |
| evaluation time | one captured request instant | current/as-of/freshness consistency across a page traversal |
| freshness/staleness threshold/result | policy evaluation | derived assessment with policy/version, not a stored replacement timestamp |

### 7.2 Representation, Precision, and Intervals

Glaux retains the accepted `TemporalInstant { instant_utc, source_lexeme, source_scale, precision, uncertainty, derivation }` and `TemporalExtent { start?, end?, bound_kind, precision, derivation }` concepts. Accepted RFC 3339 offsets are normalized to comparable UTC for ordinary CSAPI JSON, while the original lexeme, offset/scale, precision, and derivation remain evidence where required. Unqualified local time is rejected. `-00:00` does not silently become known UTC, leap seconds require a proven path, and no timestamp is silently rounded into another instant. **[N/P]**

PostgreSQL `timestamp with time zone` is a strong normalized instant projection, but its finite range and microsecond storage precision do not prove lossless support for every source lexeme, arbitrary precision, geological/deep-space time, leap seconds, or uncertainty. Exact lexical/structured evidence and explicit overflow/precision handling remain mandatory. A profile demanding lossless canonical precision outside the proven database mapping must reject/quarantine or adopt a tested auxiliary representation; it must not truncate silently. **[I/P/D]**

Intervals use explicit lower/upper bounds and bound kinds, not sentinel dates. PostgreSQL range types can support canonical valid/system intervals; Observation phenomenon/result values remain instant-oriented in the approved model. Open query bounds and source-property bounds are not interchangeable. **[N/I/P]**

### 7.3 Arrival, Duplication, Correction, and Clock Skew

- Out-of-order and replayed arrival is valid when domain values and contract pass validation. Derived latest/extents/summaries are driven by their named domain axis, not arrival order. **[N/P]**
- A retry with the same accepted source idempotency key/content is not a new fact. Equal timestamps do not prove duplication; multiple observations at the same `resultTime` remain distinct and `latest` retains ties. **[P]**
- A correction is a new revision/fact linked to the superseded record with reason, actor/source, transaction sequence, and retained evidence. Public visibility rules are downstream; destructive in-place correction is not the baseline. **[P/D]**
- Server wall time cannot total-order concurrent records. Commit sequence/watermark plus immutable RID resolves ties; source clocks remain evidence with quality/uncertainty. **[P]**
- Future `phenomenonTime` can represent forecast; future `resultTime` is invalid under CSAPI and is rejected or quarantined according to the ingestion mode. Future receive/commit time indicates infrastructure failure and must not be normalized away. **[N/P]**

---

## 8. Query, Filtering, Sorting, Pagination, Latest-Value, Replay, and Backfill Findings

### 8.1 Standards Query Baseline

| Query family | Required/accepted behavior | Storage consequence |
|---|---|---|
| DataStreams | `phenomenonTime`/`resultTime` intersect derived extents | maintain or efficiently calculate both extents; policy-safe representation |
| Observations | filter by phenomenon/result instant against instant/period; `resultTime=latest` special case | both typed axes indexed; latest plan preserves ties |
| ControlStreams | filter by issue/execution extents | derived command extents and range indexes |
| Commands | issue/execution filters plus accepted current-status mapping | command fields plus status projection/join |
| CommandStatus | report-time history through accepted project mapping | report-time ordered append store |
| SystemEvent | event-time query on global/nested collections | system/type/event-time indexes |
| System history | current/latest valid description by default; `datetime` history selection | bitemporal revision indexes, not Observation table |

Different query parameters combine with AND; list values within one allowed parameter use that parameter’s accepted OR semantics. Authorization/lifecycle scope precedes predicates. The initial profile exposes no client sort. Family-specific deterministic total orders, opaque keyset continuation, stable query normalization, and snapshot/watermark binding follow IDR-SRV-011/018. **[N/P]**

### 8.2 Normative Latest Operator

For Observation `resultTime=latest`, Glaux must:

1. authenticate/authorize and establish the canonical/nested route scope;
2. capture one evaluation time and visibility/snapshot context;
3. apply all other filters, including phenomenon time, relationships, semantics, geometry, lifecycle, and policy;
4. select the maximum visible `resultTime` in that resulting set;
5. retain every equal-time tie; and
6. apply deterministic `(resultTime, ResourceId)` ordering and keyset paging.

This is endpoint-wide within the already-filtered scope—not implicitly one row per DataStream, property, or feature. A Glaux “latest value per component” view, if later exposed, is a separately named and advertised projection with its own grouping, nil/quality, tie, freshness, and policy rules. **[N/P]**

### 8.3 Default Orders and Index Alignment

- Observations: `(resultTime, ResourceId)` ascending for public traversal; inverse index scan may serve recent access. **[P]**
- CommandStatus: `(reportTime, transaction_sequence, ResourceId)`. **[P]**
- SystemEvent: `(event_time_start, ResourceId)`, retaining transaction sequence for correction history. **[P]**
- Ingestion/sync/outbox/audit: stable application sequence plus RID; database physical tuple position and broker offset are not public cursors. **[P]**

Cursor state binds normalized filters, order, representation/profile, policy/security context or safe pseudonym, evaluation time, and snapshot/watermark. Late inserts must not cause loops; minimum-consistency mode must disclose its mutation caveat, while repeatable traversal requires a retained snapshot or logical watermark. **[P/D]**

### 8.4 Replay and Backfill

“Replay” has three different meanings and must be named:

| Meaning | Source | Cursor/order | Authority effect |
|---|---|---|---|
| API historical traversal | authoritative records | family total order + snapshot | read only |
| publication/subscription catch-up | transactional outbox/change journal | durable application sequence | republishes committed intent, creates delivery evidence |
| re-ingestion/backfill | exact source artifact/external source | source key/sequence + admission transaction | may create newly accepted facts after validation/idempotency |

Every publishable record is committed before publication intent becomes visible. Resume state uses an opaque Glaux cursor derived from durable application sequence and policy/snapshot context, not a transient broker offset. Backfill records can enter old `result_time` partitions, invalidate latest/extents/summaries, and require subscriber-visible correction/update behavior defined later. **[P/D]**

### 8.5 Query Guardrails

Default/no-range collection behavior, maximum windows, limits, query budgets, timeouts, and asynchronous export remain endpoint/profile decisions. The storage layer must expose enough plan/cost information to reject unsafe broad spatial-semantic-temporal scans before data access. Counts, extents, aggregates, and timing are policy-filtered. No index is evidence that an endpoint filter was semantically applied; black-box tests must prove misses and combinations. **[P/D]**

---

## 9. Indexing, Partitioning, Retention, Archival, Compression, and Summarization Findings

### 9.1 Observation Partition Direction

| Candidate axis | Benefits | Risks | Disposition |
|---|---|---|---|
| `result_time` range | normative required filter/latest axis; cannot be future; partition pruning for common history/latest ranges | late/backfill inserts old partitions; global uniqueness seam; retention cannot blindly equal result age | **Leading baseline; benchmark** |
| server commit time range | monotonic local lifecycle; bounded active writes; simple operational retention/audit | standards result/phenomenon queries scan many partitions; latest needs broad index/summary | **Benchmark alternative for late-heavy workloads** |
| `phenomenon_time` range | direct phenomenon queries | forecast/far-past values, uncertain/extreme domains, poor lifecycle behavior | **Reject as sole primary axis** |
| per-DataStream table/partition | local hot stream operations | unbounded objects/catalog/planning/migrations; awkward global query | **Reject by default** |
| hash/list tenant/source first | isolation/pruning for some deployments | skew, cross-tenant/global query, policy ≠ physical isolation | **Conditional subpartition only after evidence** |

Native PostgreSQL range partitioning is the initial mechanism. Partition interval (day/week/month), precreation, default/quarantine partition, late-partition management, subpartitioning, and detach/archive procedures are workload/policy results—not fixed here. PostgreSQL warns that partition count, pruning keys, and common `WHERE` patterns matter; detached/dropped partitions enable fast lifecycle operations only when every contained row is eligible. **[I/P/D]**

### 9.2 Global Identity and Partitioned Uniqueness

PostgreSQL unique/primary constraints on a partitioned table must include all partition-key columns; Timescale hypertable unique indexes similarly include partition columns. Glaux UUIDv7 resource identity, canonical addressing, source idempotency, and replacement/tombstone rules cannot be weakened to a `(result_time, id)` promise.

The bounded architecture direction is an atomically written unpartitioned identity/admission ledger containing global RID, resource family, parent stream, source key/sequence where applicable, content hash, contract revision, transaction sequence, correction/supersession link, policy/evidence references, and partition locator. The partitioned fact owns query/result payload columns. IDR-SRV-029 must prove that ledger/fact/outbox success is all-or-nothing, concurrent duplicate admission is deterministic, orphan repair is safe, and query joins do not defeat performance. **[I/P/D]**

### 9.3 Candidate Index Families

| Workload | Candidate logical index | Notes |
|---|---|---|
| Observation history/latest | `(datastream_id, result_time, resource_id)` B-tree | supports nested range/order; reverse scan for recent; latest ties retained |
| Phenomenon query | `(datastream_id, phenomenon_time, resource_id)` B-tree | separate because axes are not interchangeable |
| Global family order | `(result_time, resource_id)` B-tree | needed only if global route/workload justifies write cost |
| Append-correlated time | BRIN on result/commit within large partitions | compact candidate; benefit depends on physical correlation |
| Spatial-temporal | GiST canonical geometry plus B-tree time/stream, or benchmark multicolumn strategy | exact predicate after candidate filter; follow IDR-SRV-026 |
| Semantic-temporal | contract/property/stream lookup joined to time index | do not duplicate uncontrolled semantic metadata per sample |
| SystemEvent | `(system_id, event_time_start, resource_id)` and conditional type key | global type/time index only if route workload proves need |
| CommandStatus | `(command_id, report_time, transaction_sequence, resource_id)` | deterministic legal projection/history |
| Outbox/sync | `(delivery_state, application_sequence)` / `(peer_id, sequence)` | partial pending index candidate |
| Retention/audit | policy cohort + eligibility clock/sequence | supports lifecycle selection; not an automatic delete rule |

Multicolumn indexes should follow frequent predicates and leftmost B-tree behavior; adding every possible property/time combination would amplify writes and storage. Partial indexes for current/pending/active states and materialized projection tables are benchmark candidates. Each index needs measured hit rate, write cost, size, vacuum/maintenance, plan stability, and policy selectivity. **[I/P/D]**

### 9.4 Result Storage and Contract-Aware Projection

Each accepted Observation binds an immutable DataStream/SWE contract revision. Query/order/identity/policy/provenance clocks use typed relational columns. Exact encoded blocks or source documents remain content-addressed artifacts when preservation policy requires. Canonical structured values may use controlled relational structures and JSONB, but their component path, nil state, quality, unit/binding, array order/shape, and encoding evidence remain reconstructable. **[P]**

High-value scalar projections may be generated only from versioned approved component paths/contracts. They record source Observation, contract revision, projection version, type/unit semantics, and policy. Unbounded EAV expansion creates join/write/cardinality cost; opaque JSON-only results defeat type-safe range, latest, semantic, and validation behavior. Both are rejected as universal defaults. **[P]**

### 9.5 Retention and Archival

Retention eligibility is a conjunction over record class, deployment/profile, tenant/mission, releasability/classification, legal/audit hold, provenance dependency, correction/tombstone need, synchronization/subscriber acknowledgement, archive verification, and an explicitly named clock. It is not `now - partition_time > duration` alone. **[P]**

Before a partition is detached/dropped, Glaux verifies every row is eligible, its archive manifest/hash is committed if required, dependent aggregates/projections are consistent, DDIL/subscription cursors are not stranded, tombstones/gap semantics exist, and the action is audited. Mixed-policy partitions may require row migration or smaller/cohort partitions; policy is not inferred from physical placement. **[P/D]**

Archives remain discoverable through authoritative manifests containing resource class, covered IDs/time ranges, source partition/watermark, hashes, object locations, encryption/key reference, contract/vocabulary/software versions, policy, creation/verification state, and restore/query capability. “Archived” must not produce a normal online query promise unless transparent tier access is implemented and tested. **[P/D]**

### 9.6 Compression, Downsampling, and Summaries

- Lossless database/column compression does not change semantics and is a benchmark concern. **[I/P]**
- Lossy summarization is permitted only by an explicit profile/retention decision over compatible numerical components, units, nil/quality rules, window/time axis, and authorization scope. **[P/D]**
- A useful numerical summary retains bucket bounds, input coverage/watermark, count, nil/missing/rejected counts, min/max and their times where meaningful, aggregate function, unit/conversion evidence, quality policy, algorithm/version, source contract revisions, policy join, and derivation hash. **[P]**
- Status categories, System Events, CommandStatus, feasibility state, audit, arbitrary geometry/pose, and categorical/code values are not averaged. Event counts or state-duration products are new derived artifacts with explicit semantics. **[P]**
- Late/corrected/deleted/reclassified inputs invalidate affected buckets. A summary may outlive raw records only under an explicit policy and cannot claim lossless reconstruction. **[P]**

PostgreSQL materialized views require explicit refresh; transactionally maintained tables may better serve correctness-critical extents/latest projections. Timescale continuous aggregates can incrementally materialize buckets but late historical writes require refresh coverage, and refresh windows overlapping removed raw data can remove aggregate data. These behaviors require fixture and operational tests before adoption. **[I/P]**

---

## 10. Time-Series Storage Option Evaluation

### 10.1 Evaluation Matrix

| Option | Multi-axis/relational/spatial fit | Ingest/query/lifecycle capability | Edge/reproducibility/operations | Decision |
|---|---|---|---|---|
| Native PostgreSQL 18 partitions | strongest fit with identity, graph, PostGIS, transactions, policy, JSONB/artifacts | range/list/hash partitioning; B-tree/BRIN/GiST; manual projections/maintenance | one accepted authority; mature open tooling; moderate tuning | **Select baseline** |
| TimescaleDB 2.27.1 | PostgreSQL-compatible but time-partition/unique/license/extension constraints | hypertables, chunk lifecycle, columnstore/compression, continuous aggregates | added extension, edition/license, upgrade and edge validation | **Conditional benchmark gate** |
| InfluxDB 3 Enterprise | tag/timestamp model weak for CSAPI graph, global RID, exact contracts and atomic cross-family writes | high-rate time series, in-memory last-value cache, object storage; same timestamp/tag duplicates can overwrite nondeterministically | enterprise boundary and separate authority/operations | **Reject as authority; optional future projection only** |
| QuestDB | SQL time-series features but one designated timestamp conflicts with multiple equal clocks | partitions, `LATEST ON`, ASOF, TTL/dedup; designated timestamp constraints | extra service; TTL/domain-time semantics risky for late/future data | **Reject as authority** |
| ClickHouse | strong analytical/columnar aggregation; weak fit for transactional resource graph/workflow truth | MergeTree, high-rate analytics, materialized views, TTL/rollups | separate cluster/service and eventual merge semantics | **Optional analytical projection after evidence** |
| DuckDB + Parquet | good local columnar queries and reproducible exports | predicate/filter pushdown, partitioned files; not concurrent server-write authority | simple analytical tool/sidecar; archive portability benefits | **Select as optional analytics/archive verifier** |
| SQLite WAL | relational/embedded but no PostGIS parity and one writer | adequate reduced local datasets; limited concurrent ingest | simplest edge packaging; same-host WAL constraints | **Conditional reduced-profile store only** |
| Kafka 4.1 / NATS JetStream | delivery ordering/cursor/ack concepts, not relational domain semantics | durable streams/consumers, retention/replay; broker ordering scoped, compaction/expiry possible | additional service; useful connected deployments, optional edge | **Transport projection after transactional outbox** |
| Object/blob archive | exact/batch economics and portability; no online joins/policy by itself | large immutable segments/manifests; restore or separate engine | local filesystem/S3-compatible possibilities; consistency/encryption required | **Select for eligible exact/archive artifacts, not query truth** |

### 10.2 TimescaleDB Adoption Gate

TimescaleDB is adopted only if a pinned supported edition/version shows material benefit over native PostgreSQL on Glaux fixtures and satisfies all of the following:

1. authoritative transactions, identity ledger, PostGIS, migrations, backup/restore, replication, security/policy, and observability work end to end;
2. late/backfill/correction writes and `latest` ties remain correct;
3. unique-key and chunk/partition constraints do not weaken global identity/idempotency;
4. retention, continuous aggregate refresh, and raw-data deletion do not silently lose required summaries;
5. compression/columnstore transitions do not violate active correction/query windows;
6. licensing permits the intended self-hosted product/deployment/service model;
7. disconnected/containerized installs, offline upgrades, and rollback are reproducible; and
8. measured ingest, p95/p99 queries, write amplification, disk, maintenance, and recovery exceed agreed thresholds enough to justify complexity.

Absent that evidence, native PostgreSQL remains the answer. The gate is reversible: SQL/repository boundaries should avoid leaking extension-specific constructs into domain semantics. **[P]**

### 10.3 Why One Authoritative Core Wins

Glaux queries combine identity, typed relationships, lifecycle, bitemporal state, observations, spatial assertions, semantic contracts, provenance, and policy. Splitting dynamic facts into an independent TSDB introduces dual-write, cross-store snapshot, authorization, correction, backup, DDIL, and conformance failure modes before evidence shows need. Analytical or delivery projections are legitimate only through committed outbox/change records, with rebuild and lag visibility. **[P]**

---

## 11. Observation, Status, Event, Command, Feasibility, and Ingestion Storage Implications

### 11.1 Observation Admission and Correction

A successful admission conceptually commits: global identity/idempotency decision; parent DataStream and immutable contract revision; normalized temporal/relationship/policy/provenance columns; canonical result structure; exact artifact reference where required; validation evidence; transaction sequence; initial projection invalidations/updates; and outbox intent. Failure commits none of the public fact, although a separately authorized safe rejection/quarantine record may be retained. Exact transaction steps belong to IDR-SRV-029/031. **[P/D]**

Corrections/supersessions never reuse arrival time as result time, never silently replace exact source evidence, and never erase the reason or prior fact. Deletion/tombstone and caller-visible correction behavior remain IDR-SRV-016/029/030/034 concerns.

### 11.2 Status and Dynamic State

Status is stored as Observation history in a status DataStream when that is the source model. Current System status is a derived projection with supporting Observation, contract/property binding, domain time, transaction watermark, freshness assessment, provenance, and policy scope. `live` remains a narrow stream capability and does not assert current health/freshness. **[N/P]**

Dynamic Sampling Feature/System properties and latest-known geometry follow the same source-fact/derived-view split. A current projection that cannot be recomputed exactly from retained online facts must carry an archive/source manifest and explicit derivation evidence; otherwise it is not an authoritative substitute.

### 11.3 System Events

System Events form an immutable historical resource collection ordered by event time and RID. They may refer to lifecycle, configuration, mission, anomaly, or other event concepts accepted by IDR-SRV-020, but they are not automatically generated from every database update and do not replace status, resource revision, provenance, or security audit histories. Corrections link records explicitly. **[N/P]**

### 11.4 Commands and Feasibility

Command request evidence, Command resource state, CommandStatus reports, result links, feasibility progress/results, delivery acknowledgements, and audit events are separate. Current status is a deterministic legal projection of ordered reports, not a mutable report row. Estimated and actual execution intervals retain status/context. Terminal-state and post-terminal behavior, authorization, safety, cancellation, timeout, and tamper evidence remain IDR-SRV-036 through 039A. **[N/P/D]**

### 11.5 Ingestion, Validation, and Publication

Ingestion batches/attempts record source identity, source cursor/key, artifact hash, adapter/contract/package versions, receive/process/commit times, outcome, safe diagnostic code, accepted/rejected counts, and resulting RIDs/watermark where allowed. Raw hostile input is isolated from public truth. Metrics may summarize operations but cannot replace attempt evidence required for idempotency/audit. **[P/D]**

Publication follows store-before-publish: the same authoritative transaction creates outbox intent. Kafka/NATS retention, compaction, consumer offsets, or acknowledgements never determine whether the CSAPI fact exists. A delivery failure changes delivery evidence, not domain time or committed authority. **[P]**

---

## 12. Spatial-Temporal, Semantic-Temporal, and Dynamic-Location Implications

### 12.1 Spatial-Temporal

IDR-SRV-026’s canonical PostGIS spatial assertion remains joined to the timed fact. Typical plans combine selective stream/relationship/time B-tree predicates with GiST candidate geometry and an exact spatial predicate; actual index order/composition is benchmarked. A bounding-box index hit is not a semantic intersection result. Policy filters precede counts/extents/pages. **[P]**

Moving-platform samples are authoritative observations; latest location and trajectory/footprint are derived. Trajectory generation records input sample IDs/watermark, selected time axis, interpolation/gap policy, coordinate transform/grid versions, uncertainty/quality filters, algorithm/version, policy, and generated time. A pose additionally needs orientation/reference frame and cannot be reduced to geometry. **[P]**

### 12.2 Semantic-Temporal

Observed/controlled property, unit, result component path, nil/quality state, and immutable SWE contract revision are semantic query anchors. They are resolved through accepted IDR-SRV-022/024 bindings rather than copied as mutable labels onto every fact. Controlled projections may denormalize stable IDs/component paths for performance, but preserve derivation and invalidate on mapping/version changes. **[P]**

Aggregating across contract revisions requires compatible value type, property meaning, unit conversion, nil/quality treatment, spatial/feature scope, and time axis. Glaux performs no silent command conversion and no observation conversion that discards provenance or uncertainty. Semantic expansion/subsumption is explainable and policy bounded. **[P/D]**

### 12.3 Combined Queries

Spatial, semantic, time, relationship, lifecycle, and authorization predicates combine as one logical visible relation before ordering/paging/latest. Plans must avoid both false negatives (unsafe prefilter) and covert leakage (policy after aggregation). The benchmark corpus includes high- and low-selectivity combinations, skewed streams, dateline geometry, future phenomenon time, late result time, contract revisions, and hidden records. **[P]**

---

## 13. DDIL, Cache, Synchronization, and Federation Implications

### 13.1 Durable DDIL Evidence

Each synchronizable admission preserves origin node/source, global RID and source identity/key, content hash, source sequence when meaningful, contract/vocabulary version, domain clocks, origin receive/commit evidence, local receive/sync/commit sequence, correction/replacement relations, policy, and provenance. Local transaction order does not rewrite origin order. **[P]**

Per-peer cursors/watermarks, acknowledged ranges, gaps, conflicts, rejected/quarantined items, resend attempts, and policy-denied items are durable operational records or derived cursor state. A broker cursor alone is insufficient. IDR-SRV-043 must define conflict resolution and tombstone/gap convergence. **[P/D]**

### 13.2 Late Merge and Derived State

A reconnect/backfill can change extent minima/maxima, normative latest, current status/location, summary buckets, freshness, cache validators, and subscription results. Recalculation is keyed by affected subject/stream/time bucket/policy scope and tied to the new transaction watermark. A last-known value remains last-known and may be stale; absence of new data never fabricates an “offline” observation. **[P]**

### 13.3 Federation and Cache Classes

Federated authoritative remote facts, locally mirrored facts, transient query caches, and locally derived projections carry distinct authority/origin/lease/freshness/policy metadata. A cached external record is not silently promoted to locally authoritative truth. Cursor tokens and cached pages bind federation/source snapshot and policy. Expired cache never deletes retained domain facts. **[P/D]**

### 13.4 Bandwidth and Archive Behavior

DDIL exchange should support content hashes, immutable contract/vocabulary packages, range/watermark negotiation, gap detection, resumable batches, deduplication, and policy-aware selective replication. Compression changes transfer bytes, not record identity. Summary-only replication is an explicit lossy product; it cannot later claim full observation equivalence. **[P/D]**

---

## 14. Security, Policy, Releasability, and Audit Implications

Time-series data amplifies inference: even hidden values may be exposed by timestamps, cadence, gaps, latest availability, counts, extents, aggregates, location tracks, command timing, query latency, cursor behavior, cache validators, partition names, logs, or archive manifests. Exact, generalized, summarized, concealed, and denied outputs are distinct disclosure products. **[A/P]**

Policy applies before latest selection, min/max extents, counts, summaries, ordering, paging, streaming, backfill, export, and cache lookup. Derived products record input-policy joins, policy/version, purpose/audience scope, algorithm/version, and watermark. Policy change invalidates or rebuilds affected projections/caches; a global latest row is unsafe when audiences see different facts. **[P]**

At rest/in transit encryption, key rotation, row/tenant separation, database roles, audit immutability/tamper evidence, privileged-operation controls, legal holds, and secure deletion are required design inputs but exact mechanisms belong to Category G and operations topics. Database row-level security may be defense in depth, not the only place semantics live; background aggregate/retention jobs require the same policy discipline. **[P/D]**

Audit events should cover admission/rejection, correction/deletion, policy decision/version, sensitive historical/latest/aggregate query, export/archive/restore, retention action, subscription/backfill, administrative override, and command/feasibility actions. Logs and metrics use stable pseudonymous IDs and safe diagnostics; they do not copy unrestricted payloads or precise tracks by default. **[P/D]**

Retention and deletion can conflict with audit, synchronization, derived products, and legal/mission policy. The lifecycle topic must specify precedence, approval, tombstone/residual metadata, cryptographic erasure, archive handling, and proof. This report establishes no universal “keep forever” or TTL. **[D/X]**

---

## 15. Fixture, Conformance, Performance, and Interoperability Test Implications

### 15.1 Canonical Fixture Corpus

| Fixture family | Minimum cases | Main proof |
|---|---|---|
| Observation clocks | phenomenon=result; delayed result; future phenomenon; invalid future result; offsets; fractional precision; leap/unknown-offset quarantine | non-collapse and validation |
| Latest/ties | multiple streams/FOIs, equal maximum result times, hidden winner, nil/quality, phenomenon filter plus latest | operator scope, ties, policy |
| Ordering/pages | same time many RIDs; inserts before/after cursor; policy/evaluation change; snapshot expiry | deterministic no-loop traversal |
| Late/backfill/correction | old result received now; replay duplicate; same time distinct values; correction/supersession; partition absent | admission, identity, recomputation |
| Status/dynamic state | stale last-known, no value, conflicting sources, late winner, vocabulary revision | current vs latest/fresh/available |
| System Event | instant/extent, same-time ties, types, correction, hidden event | history/filter separation |
| Command/feasibility | accepted-to-terminal, incremental progress, estimated/actual execution, invalid transition, result link | ordered append and projection |
| SWE result | scalar, record, array, nil, quality, unit, contract revision, JSON/Text/Binary exact evidence | contract-aware storage/reproduction |
| Spatial-temporal | moving positions, antimeridian, altitude/reference gap, trajectory gap, policy-generalized track | PostGIS/time interaction |
| Semantic-temporal | property aliases, incompatible units/types, contract revisions, controlled conversion | safe semantic join/summary |
| DDIL/sync | gaps, resend, duplicate, conflict, tombstone, summary-only peer, policy denial | convergence/cursor evidence |
| Retention/archive | mixed policy, legal hold, dependent aggregate, unacked peer, detach/restore/hash failure | no premature/lossy deletion |

### 15.2 Conformance and Interoperability

Conformance tests cover DataStream and Observation phenomenon/result filters; Observation latest ties and scope; ControlStream/Command issue/execution filters; CommandStatus report history; SystemEvent time; System history/as-of; inherited route/relationship filters; limits, errors, content negotiation, and links. Every filter has positive, negative, malformed, boundary, open-interval where allowed, combined, authorization, and multi-page cases. **[N/P]**

Interoperability runs the same golden populations through CSAPI Explorer, OS4CSAPI clients, web/mobile clients, and external version-pinned clients. It verifies identity/population equivalence across JSON and SWE encodings, time normalization, ties, nested/global routes, late data, pagination, errors, and policy-safe absence. Implementation quirks are recorded as versioned evidence, not adopted silently. **[P/D]**

### 15.3 Performance and Recovery

The workload suite varies stream count/cardinality, records/second, concurrent writers/readers, batch sizes, result widths, contract complexity, late-data percentage/age, equal-time ties, spatial/semantic selectivity, policy selectivity, partitions/chunks, retention windows, backfill subscribers, and archive restore.

Measure sustained and burst ingest, p50/p95/p99 commit/query latency, latest/range/spatial-semantic query plans, page stability, deadlocks/retries, WAL, index/heap size, write amplification, vacuum/analyze, partition create/detach, projection lag/rebuild, aggregate refresh, compression transition, backup/restore, crash recovery, and edge resource use. Compare native PostgreSQL and Timescale on identical semantics and pinned versions. Thresholds belong to IDR-SRV-054. **[P/D]**

### 15.4 Required Failure Tests

- process/database/broker crash at every admission/outbox boundary;
- duplicate concurrent source keys and UUIDs across different partitions;
- missing/corrupt artifact, stale contract/vocabulary, bad archive hash/key;
- clock regression, result time in future, extreme source time, precision overflow;
- retention racing query/backfill/aggregate/backup;
- policy change during page/subscription/latest rebuild;
- partition/default-partition exhaustion and disk pressure;
- Timescale/native upgrade/rollback if the extension gate advances; and
- SQLite reduced-profile concurrent-writer and WAL recovery constraints.

---

## 16. Downstream Topic Handoff Matrix

| Topic(s) | Required handoff from IDR-SRV-027 | Acceptance/proof gate |
|---|---|---|
| 028 Metadata/document storage | exact artifacts, canonical result/document boundary, archive manifests, contract references | one authority; deterministic reconstruction; no opaque JSON-only truth |
| 029 Transactions/idempotency/concurrency | identity/admission ledger, fact/outbox atomicity, duplicate/correction locks, projection invalidation | concurrency/crash proof across partition seam |
| 030 Lifecycle/retention/archive/delete | multidimensional eligibility, partition cohort rules, aggregates/archive/DDIL/legal holds | no duration inferred; verified restore and audited delete |
| 031 Write/ingestion | clocks, source keys, strict/quarantine, late/backfill, batch evidence | exact admission outcomes and replay safety |
| 033/034 Query and dynamic-data semantics | time predicates/orders, latest tie scope, defaults, extents, results/projections | black-box standards and policy correctness |
| 035 Streaming/event publication | store-before-publish, durable app sequence, backfill/resume, recomputation events | no broker-as-truth; draft Part 3 only if separately authorized |
| 036-038 Commands/feasibility/security | typed request/status/result histories, ordered projection, sensitive audit | legal state machine and tamper/policy controls |
| 039/039A/040 Security/status | policy-before-derived/query, inference controls, freshness/availability | audience-specific latest/extents/cache proof |
| 042/043 DDIL/sync | origin evidence, gaps/cursors, late recompute, source-vs-local order | deterministic convergence and tombstone/gap behavior |
| 044-049 Deployment/operations | PostgreSQL baseline, optional services, partition jobs, backup/archive/monitoring | reproducible connected/edge install and recovery |
| 050-052 Conformance/API tests | temporal families, malformed/boundary/latest/page cases | versioned requirements-to-test traceability |
| 053 Fixtures | corpus in §15.1 and exact SWE/source evidence | deterministic licensed/sanitized golden assets |
| 054 Performance | workload dimensions, PG/Timescale comparison, recovery measures | agreed thresholds on representative hardware |
| 055 Security tests | time-series inference, policy changes, archives/logs/cursors | no unauthorized fact/aggregate/timing disclosure |
| 056 Interoperability | external clients, nested/global populations, encodings, late/ties/errors | bounded versioned compatibility claims |

IDR-SRV-026 receives confirmation that position samples remain authoritative timed observations while latest location/trajectory are derived. IDR-SRV-022/024 contracts and semantic bindings remain immutable inputs rather than duplicated mutable measurement labels. **[P]**

---

## 17. Recommendations

| ID | Recommendation | Rationale | Priority / precondition |
|---|---|---|---|
| P-027-01 | Use native PostgreSQL 18 partitioned relational tables as the initial full-profile time-series authority, with PostGIS spatial joins. | Preserves accepted transactional graph/policy architecture and avoids premature dual truth. | High; accepted 025/026 |
| P-027-02 | Use typed family-specific append stores; never collapse observations, status, events, command/feasibility history, ingestion, sync, outbox, and audit into one semantic table. | Their clocks, authority, retention, correction, and disclosure differ. | High |
| P-027-03 | Carry all accepted temporal axes and exact precision/source evidence; never substitute arrival/commit for phenomenon/result/event/command time. | Required for standards, replay, DDIL, provenance, and audit. | High; 018 controls |
| P-027-04 | Advance `result_time` range partitioning as the Observation leading candidate, benchmarked against commit-time partitioning under late-heavy workloads. | Aligns with required query/latest and avoids future phenomenon partition pathologies. | High; DDL waits for benchmark/029 |
| P-027-05 | Add an atomic unpartitioned identity/admission ledger concept for global RID/source-key uniqueness across partitioned facts. | Partition-key uniqueness rules must not weaken IDR-SRV-016 identity/idempotency. | High; 029 must prove |
| P-027-06 | Put query-critical clocks/IDs/policy/quality/spatial keys in typed columns; preserve exact artifacts and immutable SWE-contract-bound structured results; allow only versioned selective projections. | Avoids both opaque JSON truth and uncontrolled EAV expansion. | High; 028/031 detail |
| P-027-07 | Implement accepted latest/filter/order/paging semantics from logical authorized records; treat extents/latest/current/summaries as recomputable policy-scoped projections. | Prevents tie errors and derived-data leakage. | High |
| P-027-08 | Use B-tree stream/time/RID indexes as the baseline, BRIN and partial indexes conditionally, and GiST through IDR-SRV-026; justify every additional index with workload evidence. | Balances range/latest behavior with ingest/write cost. | High; 054 measures |
| P-027-09 | Separate retention eligibility from partition axis and database TTL; gate detach/delete on policy, holds, provenance, derived products, archive verification, and DDIL/subscriber state. | Prevents immediate loss of late data and stranded dependencies. | High; 030 owns policy |
| P-027-10 | Permit summaries only as explicit versioned derived artifacts with semantic/unit/quality/policy rules; never average status/event/command/audit categories or claim lossless replacement. | Prevents semantic corruption and inference. | High |
| P-027-11 | Keep TimescaleDB 2.27.1 conditional on the eight-part measured gate; use DuckDB/Parquet only for analytics/archive verification, SQLite only for a declared reduced profile, and Kafka/NATS only after outbox. | Retains portability and one authority while leaving evidence-driven optimization paths. | Medium |
| P-027-12 | Persist durable application sequences, source keys/hashes, gaps, and per-peer/subscriber cursor evidence for replay/backfill/DDIL; do not expose physical offsets as truth. | Required for stable resume, late data, and transport replacement. | High; 035/043 detail |
| P-027-13 | Build the §15 fixture, conformance, workload, failure, recovery, and interoperability corpus before freezing partitions or selecting extensions. | Capability documentation is not workload evidence. | High |
| P-027-14 | Apply policy before latest/extents/counts/aggregates/pages/streams/archives and scope derived products/caches to policy/version. | Time-series metadata and absence patterns are sensitive. | High; Category G details |

---

## 18. Risks, Constraints, and Open Questions

### 18.1 Risks and Mitigations

| Risk/constraint | Impact | Mitigation / owner |
|---|---|---|
| Wrong partition axis/interval | broad scans, hot partitions, late-write pain | dual-candidate benchmark with real workload; 029/054 |
| Partitioned uniqueness seam | duplicate RID/source fact across time partitions | atomic identity/admission ledger and concurrency tests; 029 |
| Late/corrected data leaves projections stale | incorrect latest/extents/summary/subscription | keyed invalidation, watermarks, rebuild/failure tests; 029/034/035 |
| Timestamp precision/range loss | changed ordering/identity or unverifiable source | exact evidence, explicit supported mapping, reject/quarantine; 023/028/031 |
| Universal TTL follows result time | immediate deletion of backfill or policy breach | multidimensional eligibility and cohort verification; 030 |
| Aggregation corrupts semantics | invalid cross-unit/type/quality conclusions | contract-aware versioned aggregation only; 024/030/034 |
| Policy after derivation | hidden values leaked by latest/count/time | policy-scoped products and inference tests; 039/055 |
| Specialist TSDB becomes second truth | split transactions, authorization and recovery | PostgreSQL authority; outbox-only projections; 025/029 |
| Broker retention/offset treated as history | lost or inconsistent replay | application outbox/change sequence; 035/043 |
| Too many indexes/partitions | write amplification and planning cost | workload-driven index budget/plan regression; 054 |
| Controlled/profile input unavailable | unjustified retention/security behavior | record gap; no invented duration/classification; 030/039 |

### 18.2 Open Questions and Resolution Owners

1. What representative high-rate, stream-cardinality, late-data, retention, and edge hardware profiles control the benchmark? — `IDR-SRV-054` with project deployment input.
2. Does `result_time` remain the best Observation partition key after late/backfill and global-query testing, and what interval/subpartition is justified? — `IDR-SRV-029/054`.
3. What exact identity-ledger/fact/outbox transaction and foreign-key pattern meets atomicity without unacceptable join cost? — `IDR-SRV-029`.
4. Which result component paths merit typed projections, and how are arrays/records/high-rate binary blocks chunked? — `IDR-SRV-028/031/034`.
5. What retention durations, legal holds, archive tiers, deletion/tombstone semantics, and summary permissions apply per deployment/profile? — `IDR-SRV-030` plus authorized profile input.
6. Should extents/latest/current projections be trigger-maintained, application-maintained, asynchronously materialized, or query-computed per workload/policy class? — `IDR-SRV-029/034/054`.
7. Does TimescaleDB clear the performance, licensing, edge, backup, PostGIS, late-data, and continuous-aggregate gates? — `IDR-SRV-044/049/054`.
8. What public replay/backfill/resume contract is required, including any future draft CSAPI Part 3 adoption? — `IDR-SRV-035` or a separately authorized research plan.
9. How do correction, deletion, redaction, and policy change propagate to subscribers/federated peers and archived summaries? — `IDR-SRV-030/035/039/043`.
10. Which client versions correctly handle equal latest ties, high-precision times, SWE blocks, and nested/global pagination? — `IDR-SRV-056`.

None of these gaps prevents choosing the authoritative PostgreSQL and semantic-storage baseline; each prevents premature DDL, duration, extension, or public-protocol claims.

---

## 19. Validation Against Plan Success Criteria

| Topic plan success criterion | Validation status | Evidence |
|---|---|---|
| Time-series/time-indexed categories identified with source anchors | Met | §§3, 5-6 |
| Observations, status, dynamic properties, events, command/feasibility, ingestion, latest, audit, summaries distinguished | Met | §§5-6, 11 |
| Phenomenon, result, ingestion, publication, transaction, event, command, valid, sync, freshness implications documented | Met | §§7, 11, 13 |
| Filtering/latest/spatial-semantic/order/page/retention/archive/replay/backfill documented | Met | §§8-9, 12-13 |
| Candidate storage options evaluated against explicit criteria | Met | §§4, 10 |
| Security, policy, DDIL, fixture, performance, conformance, interoperability documented | Met | §§13-15 |
| Implementation/community lessons incorporated as non-normative evidence | Met | §3.3 and test/risk synthesis |
| Recommendations decision-usable and bounded to Glaux Server | Met | §§2.2, 17-18 |
| Downstream handoffs explicit | Met | §16 and recommendation owners |
| References explicit and reproducible | Met | §20; official source pin and technology snapshot in header/§3 |

### 19.1 Methodology Completion

| Phase | Status | Output location |
|---|---|---|
| 1. Source collection/framework | Complete | §§3-4 |
| 2. Category/resource inventory | Complete | §§5-6 |
| 3. Semantics/query/index/retention | Complete | §§7-9 |
| 4. Storage option analysis | Complete | §10 |
| 5. Security/DDIL/test/interoperability | Complete | §§12-16 |
| 6. Synthesis | Complete | §§1, 17-18 |

The required fifteen matrix fields appear verbatim in §5.2. All planned categories and source classes are represented. The report is complete for review; acceptance fields remain intentionally open.

---

## 20. References

### 20.1 Project Plans and Accepted Reports

- [IDR-SRV-027 topic plan](../IDR%20Plans/idr-srv-027-time-series-observation-storage-strategy.md)
- [Glaux Server overall IDR research plan](../IDR%20Plans/overall-idr-research-plan.md)
- [Glaux Server goal and definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research report template](../../../../../Governance/research-report-template.md)
- [IDR-SRV-011 query/filter/sort/page report](idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md)
- [IDR-SRV-016 identity report](idr-srv-016-identifier-uri-and-resource-lifecycle-strategy-report.md)
- [IDR-SRV-018 temporal report](idr-srv-018-temporal-validity-and-freshness-model-report.md)
- [IDR-SRV-019 provenance/quality/trust report](idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md)
- [IDR-SRV-020 status/event report](idr-srv-020-status-availability-and-system-event-model-report.md)
- [IDR-SRV-022 SWE Common report](idr-srv-022-swe-common-data-component-strategy-report.md)
- [IDR-SRV-023 validation report](idr-srv-023-schema-and-encoding-validation-strategy-report.md)
- [IDR-SRV-024 semantic/unit report](idr-srv-024-units-observed-properties-and-semantic-binding-strategy-report.md)
- [IDR-SRV-025 persistence report](idr-srv-025-database-and-persistence-architecture-options-report.md)
- [IDR-SRV-026 geospatial report](idr-srv-026-geospatial-storage-and-query-strategy-report.md)
- [OGC API - Connected Systems upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)

### 20.2 Standards and Official Artifacts

- [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html)
- [Official CSAPI `v1.0.0` source pin](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [CSAPI Part 2 DataStreams/Observations source clause](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/clause_8_requirements_class_datastreams.adoc)
- [CSAPI Part 2 ControlStreams/Commands/CommandStatus source clause](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/clause_9_requirements_class_controlstreams.adoc)
- [CSAPI Part 2 System Events source clause](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/clause_11_requirements_class_system_events.adoc)
- [CSAPI Part 2 System History source clause](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/clause_12_requirements_class_system_history.adoc)
- [CSAPI Part 2 Advanced Filtering source clause](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/clause_14_requirements_class_advanced_filtering.adoc)
- [OGC API - Features - Part 1: Core, OGC 17-069r4](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC SensorML 3.0, OGC 23-000](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model 3.0, OGC 24-014](https://docs.ogc.org/is/24-014/24-014.html)
- [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339)

### 20.3 Database, Time-Series, Analytics, and Transport Sources

- [PostgreSQL 18 declarative partitioning](https://www.postgresql.org/docs/18/ddl-partitioning.html)
- [PostgreSQL 18 index types](https://www.postgresql.org/docs/18/indexes-types.html)
- [PostgreSQL 18 multicolumn indexes](https://www.postgresql.org/docs/18/indexes-multicolumn.html)
- [PostgreSQL 18 date/time types](https://www.postgresql.org/docs/18/datatype-datetime.html)
- [PostgreSQL 18 range types](https://www.postgresql.org/docs/18/rangetypes.html)
- [PostgreSQL 18 materialized views](https://www.postgresql.org/docs/18/rules-materializedviews.html)
- [TimescaleDB 2.27.1 release](https://github.com/timescale/timescaledb/releases/tag/2.27.1)
- [Timescale hypertables and indexes](https://docs.timescale.com/use-timescale/latest/hypertables/)
- [Timescale retention policy](https://docs.timescale.com/use-timescale/latest/data-retention/create-a-retention-policy/)
- [Timescale continuous aggregates](https://docs.timescale.com/use-timescale/latest/continuous-aggregates/about-continuous-aggregates/)
- [Timescale continuous-aggregate refresh policies](https://docs.timescale.com/use-timescale/latest/continuous-aggregates/refresh-policies/)
- [Timescale real-time aggregates](https://docs.timescale.com/use-timescale/latest/continuous-aggregates/real-time-aggregates/)
- [Timescale editions and licensing](https://docs.timescale.com/about/latest/timescaledb-editions/)
- [InfluxDB 3 Enterprise line protocol and duplicate behavior](https://docs.influxdata.com/influxdb3/enterprise/reference/line-protocol/)
- [InfluxDB 3 last-value cache](https://docs.influxdata.com/influxdb3/enterprise/admin/last-value-cache/)
- [InfluxDB 3 Enterprise overview](https://docs.influxdata.com/influxdb3/enterprise/)
- [InfluxDB 3 object storage](https://docs.influxdata.com/influxdb3/enterprise/admin/object-storage/)
- [QuestDB designated timestamp](https://questdb.com/docs/concepts/designated-timestamp/)
- [QuestDB TTL](https://questdb.com/docs/concepts/ttl/)
- [QuestDB schema design essentials](https://questdb.com/docs/schema-design-essentials/)
- [QuestDB time-series optimizations](https://questdb.com/docs/architecture/time-series-optimizations/)
- [ClickHouse time-series database guide](https://clickhouse.com/resources/engineering/what-is-time-series-database)
- [ClickHouse materialized-view and TTL pattern](https://clickhouse.com/blog/using-materialized-views-in-clickhouse)
- [DuckDB concurrency](https://duckdb.org/docs/current/connect/concurrency)
- [DuckDB Parquet overview](https://duckdb.org/docs/stable/data/parquet/overview)
- [DuckDB partitioned writes](https://duckdb.org/docs/current/data/partitioning/partitioned_writes)
- [SQLite write-ahead logging](https://www.sqlite.org/wal.html)
- [SQLite appropriate uses](https://www.sqlite.org/whentouse.html)
- [Apache Kafka 4.1 design](https://kafka.apache.org/41/design/design/)
- [Apache Kafka documentation](https://kafka.apache.org/documentation/)
- [NATS JetStream consumers](https://docs.nats.io/nats-concepts/jetstream/consumers)
- [NATS JetStream streams](https://docs.nats.io/nats-concepts/jetstream/streams)
- [Rust SQLx PostgreSQL type mappings](https://docs.rs/sqlx/latest/sqlx/postgres/types/)

### 20.4 Controlled and Informative Sources

- Controlled project source `AC/224(JCGISR)D(2026)0005`, April 27, 2026, SHA-256 `56dc757b6e677b3584e3152a957849f21a24b22854f562613ff283a8b599da8c`; access and handling remain governed by the project environment; no controlled text is reproduced here.
- [OpenSensorHub core repository](https://github.com/opensensorhub/osh-core)
- [OSH issue #331: phenomenon-time filter with latest result](https://github.com/opensensorhub/osh-core/issues/331)
- [Connected Systems Go](https://github.com/OS4CSAPI/connected-systems-go)
- [pygeoapi](https://github.com/geopython/pygeoapi)
- [OS4CSAPI client research/testing corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing)
- [SECD interoperability repository](https://github.com/Sam-Bolling/csapi-server-interop-secd)
- [CSAPI Explorer](https://ogc-csapi-explorer.pages.dev/)
- [OS4CSAPI discussions](https://github.com/orgs/OS4CSAPI/discussions)

---

**Report status:** In Review. Research is complete; acceptance and all downstream work remain subject to Glaux Project Lead action.

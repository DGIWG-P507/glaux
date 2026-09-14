# Section 025: Database and Persistence Architecture Options - Research Report

**Topic ID:** IDR-SRV-025<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-025 Database and Persistence Architecture Options](../IDR%20Plans/idr-srv-025-database-and-persistence-architecture-options.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions, all detailed-question groups, all 6 methodology phases, and all 9 success criteria<br>
**Methodology Used:** Authority-ranked persistence extraction from accepted IDR-SRV-001 through IDR-SRV-024 and approved CSAPI/SensorML/SWE sources; direct review of current official PostgreSQL, PostGIS, TimescaleDB, SQLite, DuckDB, Kafka, NATS, and Rust ecosystem documentation; implementation-study comparison; data-category and workload classification; option scoring; and bounded synthesis into an architecture direction plus downstream decision gates<br>
**Research Time:** Approximately 15 hours of AI-assisted execution on September 14, 2026<br>
**Official Standards Source Pin:** [`opengeospatial/ogcapi-connected-systems` `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Shared Register Baseline:** [OGC API - Connected Systems upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), Version 1.10; no persistence-specific standards-history change was required<br>
**Technology Documentation Snapshot:** PostgreSQL 18, released PostGIS 3.6 documentation, TimescaleDB 2.27.1, SQLite documentation current through the March 2026 WAL correction, DuckDB 1.5-era current documentation, SQLx 0.9, SeaORM 2.0, and Rust `object_store` 0.14.1; all checked September 14, 2026 and treated as mutable implementation evidence<br>
**Controlled AEP Source:** `AC/224(JCGISR)D(2026)0005`, April 27, 2026, SHA-256 `56dc757b6e677b3584e3152a957849f21a24b22854f562613ff283a8b599da8c`; used only through accepted project findings and not redistributed<br>
**Document Purpose:** Select a bounded persistence architecture direction and option gates for later detailed research without designing schemas, choosing all products, implementing the server, or starting IDR-SRV-026<br>
**Author:** OpenAI Codex<br>
**Accepted By:** TBD until Glaux Project Lead acceptance<br>
**Acceptance Date:** TBD until accepted<br>
**Date:** September 14, 2026<br>
**Last Updated:** September 14, 2026

---

## Reading Guide and Evidence Labels

| Label | Meaning |
|---|---|
| **N** | Normative or standards-derived requirement affecting persisted behavior |
| **A** | Project-controlling AEP/STANAG adoption or operational-context finding |
| **P** | Glaux architecture direction or recommendation proposed for acceptance here |
| **I** | Informative technology, implementation, benchmark, or community evidence |
| **D** | Detailed design or product decision deferred to its owning later topic |
| **X** | Evidence gap, workload uncertainty, or unresolved decision requiring prototype/benchmark work |

“Durable,” “authoritative,” “append-only,” “immutable,” “tamper-evident,” “replicated,” and “cached” are different properties. A broker retention log is not automatically the domain source of truth; a database WAL is not the Glaux event model; a JSON column is not exact source preservation; and a replica is not a DDIL conflict-resolution policy.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Persistence Requirement Extraction Methodology
5. Glaux Server Data-Category Inventory
6. Persistence Responsibility and Retention Matrix
7. Storage Pattern Evaluation Criteria
8. Relational/Hybrid Persistence Option Findings
9. Geospatial Persistence Implications
10. Time-Series Persistence Implications
11. Metadata/Document Storage Implications
12. Event-Log, Audit, and Relationship Persistence Implications
13. DDIL, Synchronization, Cache, and Federation Implications
14. Transaction, Consistency, Migration, and Concurrency Implications
15. Security, Policy, Provenance, and Audit Implications
16. Rust Ecosystem, Deployment, CI, Fixture, and Conformance-Test Implications
17. Implementation Lessons and Risk Analysis
18. Downstream Topic Handoff Matrix
19. Recommendations
20. Risks, Constraints, and Open Questions
21. Validation Against Plan Success Criteria
22. References

---

## 1. Executive Summary

Glaux should use a **relational-hybrid authoritative core centered on PostgreSQL, with PostGIS required for the full CSAPI server profile**. Canonical resource identity, relationships, lifecycle, policy scope, stream contracts, commands, events, provenance, semantic bindings, and coordination metadata belong in one transactional authority. Structured columns carry identities, invariants, join/filter keys, temporal axes, ownership, and policy facts; JSONB carries standards-shaped canonical trees and heterogeneous metadata where relational decomposition would erase structure or make evolution brittle; exact source bytes remain a distinct content-addressed artifact. **[P]**

This direction is evidence-led without pretending the whole implementation is selected. PostgreSQL 18 supplies mature transactions, constraints, multiple index families, JSON/JSONB, range types, partitioning, materialized views, row security, backup/replication tooling, and Rust support. PostGIS 3.6 keeps geometry, spatial reference, index, attribute, relationship, and transaction behavior in the same query/consistency boundary. This directly fits compound CSAPI spatial, temporal, semantic, lifecycle, and policy queries better than splitting authoritative feature metadata into a separate document or search store. **[I/P]**

The architecture should begin with ordinary PostgreSQL range partitioning plus B-tree/BRIN indexes for high-rate Observation and append-oriented records. TimescaleDB 2.27.1 is a serious optional accelerator because hypertables, lifecycle-aware row/column storage, retention policies, and continuous aggregates address the workload. It is **not** selected here: unique keys must include partition dimensions; hypertables constrain some updates and foreign-key patterns; operational/licensing/upgrade effects and late-data behavior need IDR-SRV-027 benchmarks. The baseline must remain correct without TimescaleDB, and its use must not leak into the domain contract. **[I/P/D]**

Relationship traversal should start with typed relational edge tables, reverse indexes, and selectively materialized closure tables. Accepted IDR-SRV-017 workloads are bounded and provenance-rich; they do not justify a graph database as another authority. Exact SensorML/SWE, schema, vocabulary, and validation artifacts should use a transactionally indexed content-addressed artifact interface backed initially by PostgreSQL `bytea` for small artifacts and a durable local-filesystem or S3-compatible object adapter for large artifacts, with thresholds and atomicity finalized by IDR-SRV-028. **[P/D]**

System Events, provenance activities, command transitions, synchronization operations, and audit records require append-oriented durable rows. A **transactional outbox** couples domain state and publishable records in one commit. Kafka or NATS JetStream may later distribute/replay events, but a broker is a transport projection—not the authoritative resource/event/audit store—and must be safely rebuildable or reconciled. Likewise, materialized views and caches are disposable projections with versioned builders, provenance, policy scope, watermarks, and repair procedures. **[P]**

SQLite is a useful bounded option for single-process test fixtures, portable evidence bundles, and a reduced single-node edge profile, but it cannot silently stand in for the full production database: WAL allows concurrent readers but only one writer, requires same-host shared memory, and SQLite’s March 2026 WAL race correction demonstrates why exact minimum versions matter. DuckDB is appropriate for offline analytical exports and benchmark exploration, not concurrent transactional authority. These optional paths need explicit capability profiles and production-equivalence tests. **[I/P]**

PostgreSQL logical replication can support connected one-way read models and controlled dissemination, but it does not solve disconnected multi-writer synchronization: DDL is not replicated and conflicting local writes can stop or be applied/skipped under engine rules that are not Glaux’s authority model. DDIL synchronization therefore needs an application-level, identity-stable, provenance-bearing operation/change log, per-peer cursors, acknowledgement/retry state, tombstones, conflict records, and policy-filtered bundles. IDR-SRV-043 owns that design. **[I/P/D]**

The result is intentionally a **modular monolith in data authority, not a polyglot persistence default**. PostgreSQL/PostGIS is the minimum full-profile operational dependency. Object storage, TimescaleDB, a broker, an embedded edge store, an analytical engine, and external caches are optional adapters admitted only by measured workload and operational benefit. This keeps open-source installation, tactical-edge deployment, transactions, backup/restore, CI, and conformance reproducible while leaving specialist decisions to IDR-SRV-026 through 030.

---

## 2. Scope and Plan Alignment

This report executes `IDR-SRV-025`, the first Category E topic. It inventories what accepted research says must be persisted, classifies authority and lifecycle, compares storage patterns, recommends a cross-cutting architecture direction, and defines decision gates for specialized persistence research.

It does **not** define physical tables, partitions, chunk sizes, geometry encodings, retention periods, encryption products, backup schedules, conflict algorithms, Rust data-access crates, broker bindings, or production deployment topology. It does not implement draft CSAPI Part 3 or the server, and it does not start IDR-SRV-026.

### 2.1 Research Question Coverage Matrix

| Plan question | Short form | Status | Evidence location |
|---|---|---|---|
| Q1 | Data to persist, cache, index, or derive | Complete | Sections 5-6 |
| Q2 | Suitable persistence patterns | Complete | Sections 7-13 |
| Q3 | Storage options across workload families | Complete | Sections 7-12 and 17 |
| Q4 | CSAPI fidelity, dynamic data, DDIL, security, and testing support | Complete with later mechanisms deferred | Sections 8-16 |
| Q5 | Decisions here versus specialized topics | Complete | Sections 18-20 |

### 2.2 Decision Boundary

This report chooses an **architecture direction**, not a frozen stack:

- **Chosen direction:** PostgreSQL relational/JSONB core with PostGIS for the full profile; content-addressed artifact abstraction; transactional outbox; rebuildable projections; typed repository ports.
- **Baseline for later proof:** native PostgreSQL partitioning/indexing for append-heavy data.
- **Shortlisted conditional extensions:** TimescaleDB, durable object storage, Kafka/NATS, SQLite edge bundle, DuckDB analytical path.
- **Rejected as defaults:** independent document/search authority, graph database, broker-as-truth, cloud-only managed service, in-memory authoritative state, SQLite or DuckDB as an unqualified full-profile substitute.
- **Deferred:** detailed schemas, benchmarks, transactions, retention, migration, security, synchronization, and final Rust libraries.

---

## 3. Evidence Base and Authority Classification

### 3.1 Controlling Sources

| Source | Version/status | Authority | Persistence anchors | Access date | Limitation |
|---|---|---|---|---|---|
| [CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) | OGC 23-001, Version 1.0 | N | Feature resources, relationships, CRUD, filters, sorting/paging inputs, history context | 2026-09-14 | Defines API behavior, not database design |
| [CSAPI Part 2](https://docs.ogc.org/is/23-002/23-002.html) | OGC 23-002, Version 1.0 | N | Streams, observations, commands, events, temporal queries, dynamic schema links | 2026-09-14 | Defines resources/interactions, not retention/partitioning |
| [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) | OGC 23-000 | N | Exact descriptions, identifiers, relationships, modes, SWE components | 2026-09-14 | No physical persistence mandate |
| [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html) | OGC 24-014 | N | Ordered component trees, schemas, values, nils, constraints, encodings | 2026-09-14 | No database mandate |
| Accepted IDR-SRV-015 through 024 | Final project baselines | Project-controlling | Resource, identity, relation, temporal, provenance, status/event, document, contract, validation, semantics | 2026-09-14 | Detailed physical designs expressly deferred here/later |
| Controlled AEP baseline | `AC/224(JCGISR)D(2026)0005` | A | Full-scope server and operational/DDIL context carried through accepted reports | 2026-09-14 | No controlled text redistributed; no invented retention rule |

### 3.2 Technology Primary Sources

| Source | Snapshot | Evidence used | Authority class and limitation |
|---|---|---|---|
| [PostgreSQL 18 documentation](https://www.postgresql.org/docs/18/) | Current release documentation, checked 2026-09-14 | SQL transactions/constraints; UUID, JSONB, ranges; indexes; partitioning; materialized views; RLS; backup/logical replication | Product capability **[I]**; benchmark/configuration needed |
| [PostGIS released manuals](https://postgis.net/documentation/manual/) | Released 3.6 branch | Geometry/geography, CRS, validation, functions, GiST/SP-GiST/BRIN spatial indexes | Extension capability **[I]**; exact Glaux spatial design deferred |
| [TimescaleDB 2.27.1 release](https://github.com/timescale/timescaledb/releases/tag/2.27.1) and [hypertable docs](https://docs.timescale.com/use-timescale/latest/hypertables/) | 2026-05-19 release/current docs | Hypertables/chunks, row-column lifecycle, continuous aggregates, retention, indexing/constraint limitations | Candidate extension **[I/X]**; no selection here |
| [SQLite WAL](https://www.sqlite.org/wal.html) and [appropriate uses](https://www.sqlite.org/whentouse.html) | Current docs incl. 2026 WAL-reset advisory | Same-host WAL, concurrent readers, one writer, embedded/edge strengths and limits | Candidate reduced profile **[I]** |
| [DuckDB concurrency](https://duckdb.org/docs/current/connect/concurrency) | Current 1.5-era docs | Single-process write model and analytics-optimized concurrency | Candidate analytics tool **[I]**, not OLTP authority |
| [Apache Kafka design](https://kafka.apache.org/41/design/design/) | Kafka 4.1 documentation | Partitioned replicated log, retention, compaction, transactional transport concepts | Candidate broker **[I]**, not domain truth |
| [NATS JetStream documentation](https://docs.nats.io/nats-concepts/jetstream) | Current docs | Persistent streams, retention, replication, acknowledgement and deduplication behavior | Candidate broker **[I]**, not domain truth |
| [SQLx](https://docs.rs/sqlx/0.9.0/sqlx/) | 0.9.0 | Async PostgreSQL/SQLite support, pools, migrations, typed query macros/offline metadata | Rust candidate **[I]**, final choice deferred |
| [Diesel](https://diesel.rs/) | Current docs | Typed query builder, PostgreSQL/SQLite support, migrations | Rust candidate **[I]** |
| [SeaORM](https://www.sea-ql.org/SeaORM/docs/) | 2.0 docs | Async ORM over SQLx, PostgreSQL/SQLite support, migrations/testing aids | Rust candidate **[I]** |
| [`object_store` crate](https://docs.rs/object_store/0.14.1/object_store/) | 0.14.1 | Local, memory, S3, GCS, Azure, HTTP adapters and portable interface | Artifact-adapter feasibility **[I]**; backend semantics differ |

### 3.3 Implementation Evidence

- OSH demonstrates typed domain-store/filter ports, separate feature/dynamic/command stores, UID/spatial/full-text indexes, temporal feature validity, filtered federation, and failure risks at independent read/write-store boundaries. **[I]**
- CS-Go demonstrates PostgreSQL/PostGIS, JSONB, GiST/cursor indexes, relational associations, transactions, closure triggers, and real migration pressure; its startup ORM auto-migration is a pattern to avoid. **[I]**
- The pygeoapi proof of concept demonstrates a TimescaleDB/PostGIS observation store plus Elasticsearch metadata store, but also shows cross-store transaction/recovery, startup mutation, and rebuild gaps. **[I]**
- SECD demonstrates durable-looking scale—over eleven million observations at the dated study snapshot—but its internal persistence architecture is unavailable and cannot select a Glaux product. **[I]**

### 3.4 Evidence Confidence

Confidence is **high** that Glaux needs a transactional relational/geospatial authority, exact document preservation, append-oriented dynamic/audit evidence, rebuildable projections, and explicit DDIL state. Confidence is **moderate** that native PostgreSQL alone meets production observation throughput because no Glaux workload benchmark exists. Confidence is **low** for any exact partition scheme, object threshold, broker, TimescaleDB requirement, embedded full-profile store, or synchronization implementation; these are routed to measured later decisions.

---

## 4. Persistence Requirement Extraction Methodology

Each accepted finding was converted into a persistence tuple:

`(data category, authority class, identity, mutation model, volume/velocity, temporal axes, spatial shape, exact-document need, relationship/query paths, consistency boundary, retention, policy scope, recovery requirement, candidate pattern, downstream owner)`.

The analysis then separated five persistence roles:

1. **Authoritative state:** current domain truth whose invariants require transactions and ownership.
2. **Immutable or append-oriented evidence:** observations, transitions, events, provenance, audit, and synchronization facts whose history must remain attributable.
3. **Exact artifacts:** byte-preserved documents, schemas, vocabularies, fixture packages, and large results addressed by digest.
4. **Derived projections:** time extents, current status, relationship closures, property summaries, search documents, aggregates, and API representations that can be deterministically rebuilt.
5. **Ephemeral delivery state:** queues, broker offsets, leases, connection sessions, request caches, and work buffers that do not define domain truth.

Candidate patterns were scored against the twelve criteria in Section 7. A product earned no authority merely because an implementation used it. A feature earned no requirement merely because current documentation advertises it. The recommended direction minimizes distributed transactions and operational dependencies while preserving clear adapter seams for measured specialization.

---

## 5. Glaux Server Data-Category Inventory

### 5.1 Controlling Inventory Matrix

| Data category | Related resource family | Source topic / anchor | Classification | Volume / velocity | Temporal characteristics | Geospatial characteristics | Document fidelity | Relationship / index needs | Transaction / consistency | Retention / archival | Security / policy | Candidate storage patterns | Downstream | Notes / unresolved |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| System revisions | System/subsystem | 015-019, CSAPI P1 | Authoritative bitemporal state + history | Medium / low-moderate | valid/system time, revisions, tombstone | current/history locations, bbox, trajectory refs | Exact SensorML plus canonical tree | ID/UID, parent, procedure, deployment, property | identity/relationship atomicity | Long-lived; tombstone/archive policy | ownership, location, capability | PostgreSQL structured + JSONB; PostGIS; artifact store | 026, 028-030 | Never overwrite historical meaning |
| Procedure revisions | Procedure | 015-019, 021-024 | Authoritative versioned state | Medium / low | valid/system time | occasional geometry/position metadata | Exact SensorML | system/stream/property references | revision and dependency consistency | Long-lived | IP/capability sensitivity | PostgreSQL + JSONB + artifacts | 028-030 | Shared references need stable identity |
| Deployment revisions | Deployment/subdeployment | 015-018, CSAPI P1 | Authoritative temporal feature state | Medium / moderate | interval and hierarchy validity | geometry/bbox | GeoJSON/source fidelity | deployed systems, parent/child closure | temporal relationship atomicity | Long-lived/history | operational deployment sensitivity | PostgreSQL/PostGIS | 026, 029-030 | Bitemporal overlap rules deferred |
| SamplingFeature revisions | SamplingFeature/FOI | 015-018, 024 | Authoritative feature state | Medium-high / moderate | validity and association scope | point/curve/surface/solid, CRS | GeoJSON/SensorML source | sampled feature, system, stream, property | relationship integrity | Profile-specific | location/target sensitivity | PostgreSQL/PostGIS + JSONB | 026, 028-030 | External references retain resolution state |
| Property concepts | Property | 015-017, 024 | Authoritative semantic resource | Low-medium / low | version/deprecation/effective time | None normally | Source lexical semantics | base-property graph, role bindings | cycle/identity atomicity | Long-lived/versioned | property/capability sensitivity | PostgreSQL relational + JSONB | 028-030 | Closure is derived |
| Relationship facts | All | 017, 019, 024 | Authoritative typed edge history | Medium-high / moderate | validity/system time | May connect spatial resources | Exact source link evidence | forward/reverse/closure/traversal | endpoint and lifecycle atomicity | Match endpoint history | edge itself may be restricted | PostgreSQL edge tables + indexes | 026, 029-030 | No graph DB required initially |
| DataStream definitions | DataStream | 015-016, 018, 020, 022-024 | Authoritative versioned state | Medium / moderate | validity, schema revision, live/current projection | System/FOI relations | Exact CSAPI/SWE schema | system, procedure, observed property, schema | contract activation atomicity | Long-lived | schema/capability sensitivity | PostgreSQL + JSONB + artifacts | 027-030, 034 | Summary/extents derived |
| ControlStream definitions | ControlStream | 015-016, 020, 022-024 | Authoritative versioned state | Low-medium / low | validity, contract revision | target relationships | Exact SWE command schema | system/procedure/controlled property | activation and authorization prerequisites | Long-lived | highly sensitive command affordance | PostgreSQL + JSONB + artifacts | 028-030, 036-038 | No executable contract if unresolved |
| Observations | Observation | 015, 018-019, 022-024 | Immutable append-oriented record | Very high / high-bursty | phenomenon/result/ingest/system time; late data | FOI geometry ref; sometimes result geometry | Exact payload/encoding where required | stream, FOI, property paths, idempotency key | atomic record+provenance+outbox | High-volume tiered policy | mission data/policy partitions | PostgreSQL partitioned baseline; Timescale candidate; artifact for large result | 027, 029-031, 034 | Benchmark cardinality/payload mix |
| Status data and current state | Dynamic property/status | 018, 020, 022-024 | Append facts + derived current projection | High / bursty | event time, ingest time, freshness | related System location | Exact input contract | stream/system/property/freshness | append + projection/outbox atomicity | Raw/summary policy | operational sensitivity | PostgreSQL partitioned + materialized projection | 027, 029-030, 034 | Current state is rebuildable |
| System Events | SystemEvent | 015, 019-020 | Immutable append-oriented resource/event | High / bursty | event/record/system time, sequence | affected resource refs | Exact event representation | subject/type/correlation/causation | state change + outbox consistency | Durable, policy-defined | event existence may be sensitive | PostgreSQL append tables + outbox | 029-030, 035 | Broker copy not authority |
| Commands | Command | 015-016, 018-020, 022-024 | Authoritative intent + append transition history | Medium / bursty | issue/schedule/transition/result times | target may be spatial | Exact submitted values/schema revision | control stream, target, property, correlation | idempotency, auth evidence, transition atomicity | Long-lived audit-driven | critical command/control | PostgreSQL relational + JSONB + append history | 029-030, 036-041 | Never overwrite state history |
| Feasibility requests/results | Feasibility | 019-020, 022-024 | Immutable request/result evidence; optional current status | Medium / bursty | request/evaluation/expiry | target context | Exact inputs/results | command schema, policy, model version | request/result correlation | Policy/audit-defined | sensitive capability and rationale | PostgreSQL + JSONB/artifact | 029-030, 037-041 | Result may expire; evidence retained separately |
| SensorML source documents | System/Procedure metadata | 021 | Exact immutable artifact + canonical projection | Medium / low | source/effective/system time | embedded geometry/position | Byte-exact mandatory per fidelity class | digest, resource revision, representation | document + revision registration atomicity | Long-lived/versioned | content-level policy | PostgreSQL bytea or object store + metadata | 028-030 | Object threshold deferred |
| SWE schemas/encodings | Stream contracts | 022-023 | Exact immutable artifact + compiled plan | Medium / low-moderate | activation/revision/system time | None | Byte/order-exact as applicable | fingerprint, stream, component paths | immutable parent binding | At least dependent-record life | command/schema sensitivity | PostgreSQL JSONB/bytea + artifacts | 027-030 | Compiled plan derived/rebuildable |
| OpenAPI/schema/profile artifacts | Service/validation | 009, 014, 023 | Immutable versioned artifact | Low / release-time | release/effective time | Schema may describe geometry | Exact bytes and dependency closure | URI/digest/capability set | release bundle consistency | Release/support lifetime | may expose routes/security | Artifact store + PostgreSQL catalog | 028-030, 047, 050 | Never mutable remote dependency |
| Vocabulary packages/mappings | Semantic registry | 024 | Immutable package + authoritative activation state | Low-medium / release-time | version/effective/activation time | Usually none | Exact package/digest | term, scheme, mapping, closure indexes | atomic activation/invalidation | Historical pins retained | mapping disclosure/supply chain | Artifact store + PostgreSQL graph/index | 028-030, 039-040, 047 | Closures derived by profile |
| Validation evidence | All writes/contracts | 019, 023-024 | Outcome/evidence; detailed diagnostics by risk | High / follows writes | validation/system time | references geometry validators | Exact artifact/version refs | subject, validator, diagnostic code | same decision boundary as accepted/rejected item | Tier by audit/replay need | diagnostics may leak content | PostgreSQL rows + optional artifact detail | 028-030, 041, 050-051 | Do not retain every success detail forever by default |
| Provenance/quality/trust | All | 019, 023-024 | Authoritative append evidence/assertions | High / follows transformations | activity/entity validity/system time | May describe spatial derivation | Source evidence references | entity/activity/agent graph | atomically link transformation outcome | Long-lived proportional to evidence | source/trust sensitivity | PostgreSQL relational graph + JSONB detail | 028-030, 041 | Assertions are scoped and versioned |
| Audit records | Security/operations | 019-020, 038-041 | Append-only accountability log | High / every material action | trusted event/system time, sequence | resource refs only | Canonical signed/hashable form | actor/action/subject/correlation | commit or durable handoff with action | Policy/legal/profile-defined | highest protection; separation of duty | PostgreSQL append log + export/WORM option | 029-030, 041 | Append-only is not automatically tamper-evident |
| Publisher/source state | Ingestion | 016, 019, 023 | Authoritative operational state | Medium / high updates | session, watermark, last-seen, freshness | source scope | Small structured metadata | publisher/stream/idempotency | ingestion coordination | Bounded plus audit summary | credentials excluded; source identity sensitive | PostgreSQL | 029-032, 042 | Avoid hot-row bottleneck |
| Synchronization state | Federation/DDIL | 016-019, 043 handoff | Authoritative per-peer state + operation log | High-variable / bursty | origin, sequence, seen/ack/effective time | scoped resource refs | Signed bundle/artifact fidelity | origin/resource/revision/conflict | atomic apply/idempotency | Until convergence plus audit | cross-boundary policy | PostgreSQL operation log + artifact bundles | 029-030, 042-043 | Engine replication alone insufficient |
| Configuration metadata | Service | 023-025, 047 handoff | Authoritative non-secret config version | Low / low | activation/version time | deployment endpoints | Exact source + normalized projection | component/profile/package refs | atomic validated activation | Versioned rollback history | sensitive topology; no secret values | PostgreSQL or signed files + catalog | 029-030, 047 | Secrets remain external references |
| Derived API/search projections | Collections/query | 010-012, 017-018, 020, 024 | Rebuildable materialized view/cache | High read / build-variable | `asOf`, watermark, build time | bbox/spatial search | Generated, not source | sort/cursor/filter/policy keys | versioned rebuild/swap | Disposable with rebuild evidence | must be policy partitioned | PostgreSQL materialized tables/views; optional search cache later | 026-030 | Never sole truth |
| Queue/broker delivery state | Events/streaming | 014H, 020, 035 handoff | Ephemeral/durable delivery projection | Very high / bursty | enqueue/ack/expiry/offset | None | Message bytes | topic/subscriber/correlation | outbox-to-broker reconciliation | Bounded replay policy | ACL/tenant/topic sensitive | DB outbox; Kafka/NATS optional | 029-030, 035, 039 | Not domain/event authority |
| Fixtures/golden files | Test corpus | 014A-G, 021-024, 053 | Immutable versioned test artifact | Low-medium / release-time | scenario clock/version | geometry corpus | Exact bytes essential | expected outcome/source/license | release/test bundle integrity | Project lifetime | sanitize controlled/sensitive data | Git/LFS or artifact store + manifests | 052-056 | Production data not copied casually |

---

## 6. Persistence Responsibility and Retention Matrix

### 6.1 Authority and Mutability Rules

| Persistence role | Write rule | Read/rebuild rule | Retention principle | Failure behavior |
|---|---|---|---|---|
| Current authoritative resource | Changed only through validated domain transaction with revision/ETag and evidence | Read from committed revision/current selector | Keep history/tombstone per IDR-SRV-030 | Roll back entire state transition |
| Immutable dynamic record | Insert once under stable identity/idempotency key; corrections are new evidence | Read exact contract-bound record | Tier by profile while retaining required provenance | Reject duplicate conflict or return prior result deterministically |
| Transition/event/provenance/audit row | Append; never in-place rewrite of historical fact | Project current state separately | Retain with governed deletion/export rules | No successful business action without durable evidence/outbox rule |
| Exact artifact | Content-addressed immutable put; catalog reference commits only after durable verification | Verify digest on read/restore | Retain while referenced plus policy hold | Orphan cleanup and missing-object repair are explicit |
| Materialized projection | Built from named source watermark, builder version, policy scope | May be discarded/rebuilt atomically | Disposable once superseded and verified | Mark stale/degraded; never fabricate authority |
| Cache | Best-effort population after authorization-safe keying | Miss falls back to authority | Short bounded TTL/size | Failure reduces performance, not correctness |
| Broker/queue state | Derived from committed outbox and reconciled | Replay from outbox/event authority within retention | Delivery-specific bounded retention | Retry/deduplicate; no phantom domain commit |
| Temporary buffer | Bounded memory/disk with backpressure | Not queryable as truth | Delete after commit/failure/time limit | Explicit overload/retry result |

Retention is not one database-wide TTL. IDR-SRV-030 must set per-category policies across exact source, revisions, data records, audit, tombstones, artifacts, projections, backups, exports, and broker copies. Referential and evidentiary holds can outlive the API resource’s visible lifecycle.

### 6.2 Atomicity Groups Visible Now

- Resource revision + identity aliases + relationship edges + provenance + validation outcome + System Event/outbox.
- Stream contract revision + exact schema artifacts + compiled plan fingerprint + semantic/unit role map + activation state.
- Observation/Status/Command record + exact parent-contract reference + provenance + idempotency receipt + outbox event.
- Command transition + authorization/feasibility/safety decision references + audit row + publishable status event.
- Vocabulary/profile activation + package digest + validation result + affected projection invalidation/rebuild marker.
- Synchronization apply + origin/sequence receipt + domain mutations + conflicts/tombstones + acknowledgement state.

These are logical requirements for IDR-SRV-029, not physical transaction implementations selected here.

---

## 7. Storage Pattern Evaluation Criteria

### 7.1 Weighted Rubric

| Criterion | Weight | What acceptable evidence must show |
|---|---:|---|
| Transaction and integrity support | 5 | Atomic multi-entity writes, constraints, isolation, conflict/retry behavior, recovery |
| Compound query capability | 5 | Identity, relationship, temporal, spatial, semantic, status, policy, sort, cursor plans |
| Geospatial maturity | 5 | CRS-aware geometry/geography, validation, spatial predicates/indexes, GeoJSON interop |
| Append/time-series behavior | 4 | High-rate ingest, late data, partitioning, latest/range queries, retention, aggregates |
| Source/document fidelity | 4 | Exact bytes plus structured canonical trees, versioning, digest, large-artifact path |
| Security/policy/audit | 5 | Least privilege, policy scoping, encryption integration, immutable/tamper-evident options |
| DDIL/recovery/portability | 5 | Self-hosting, offline operation, backup/restore, export, explicit synchronization state |
| Open-source/reproducible deployment | 5 | Source availability, local/container install, no mandatory managed service |
| Rust ecosystem and testability | 4 | Maintained driver/tooling, async behavior, migrations, deterministic CI/test support |
| Operational simplicity | 4 | Dependency count, monitoring, upgrades, capacity, failure domains, operator skills |
| Evolution/portability | 3 | Explicit migrations, standards-shaped boundaries, replaceable accelerators, export |
| Evidence maturity | 3 | Stable documentation/releases and representative benchmarks, not marketing alone |

Security, transactionality, full-profile geospatial capability, offline self-hosting, and reproducible tests are gates, not benefits that lower-scoring convenience can offset.

### 7.2 Option Comparison

| Option | Fit | Benefits | Costs/risks | Disposition |
|---|---|---|---|---|
| PostgreSQL relational only | Strong core, incomplete full spatial profile | One authority, transactions, JSONB, ranges, partitioning, indexes, ecosystem | Native geometric types do not replace CRS-aware OGC spatial behavior | Keep as core; add PostGIS for full profile |
| PostgreSQL + PostGIS relational-hybrid | Strongest cross-cutting fit | Same transaction/query boundary for structured, JSON, temporal, spatial, policy data | Server/extension operations and migrations; still needs high-rate benchmarks | **Recommended authoritative baseline** |
| Add TimescaleDB for dynamic tables | Potential high-value extension | Hypertables, chunk management, lifecycle row/column storage, aggregates/retention | Extension/license/upgrade coupling; unique/FK/update constraints; late-data tests | Conditional after IDR-SRV-027 benchmark |
| Separate document/search store | Narrow read-model fit | Flexible documents, full-text/search scaling | Dual authority, distributed transactions, projection lag, policy duplication | Reject as default; reconsider only as rebuildable projection |
| Object/blob store | Strong for large immutable artifacts | Content-addressing, streaming, cheap large-object storage, local/S3 adapters | Cross-store commit, backup, orphan/missing objects, backend semantic differences | Conditional artifact tier; catalog remains transactional authority |
| Kafka/NATS durable broker | Strong delivery/replay fit | Fan-out, buffering, subscriber state, transport decoupling | New failure domain; retention/compaction differs from domain history | Optional delivery adapter behind outbox |
| Graph database | Weak present need | Native traversal/query language | Extra authority/ops/policy sync; bounded typed graph fits relational model | Reject by default; require measured relational failure |
| SQLite | Strong reduced single-node/portable role | Embedded, reliable, simple, offline, single-file | One writer, same-host WAL, spatial/feature gaps, semantic drift from PostgreSQL | Conditional explicit edge/test profile only |
| DuckDB | Strong analytical sidecar | Embedded columnar analytics, Parquet/large scan strengths | Not optimized for concurrent small OLTP; single writer process model | Analytical export/benchmark tool only |
| File-backed authoritative store | Weak full-server fit | Minimal dependencies and easy inspection | Concurrency, indexes, transaction scope, migration, crash consistency | Fixtures/bootstrap only |
| Polyglot-by-default | Poor initial fit | Specialized engine per workload | Distributed consistency, security, backup, CI, DDIL, operator burden | Reject |

---

## 8. Relational/Hybrid Persistence Option Findings

### 8.1 Recommended Authoritative Core

PostgreSQL/PostGIS should be the full-profile baseline subject to later prototype confirmation. It allows local IDs/UUIDs, UIDs, revisions, typed relationships, policy facts, temporal ranges, semantic identifiers, contract fingerprints, and outbox rows to share foreign keys and transactions. PostGIS adds standard spatial types, CRS metadata, validity functions, transformations, and index-aware predicates without crossing a service boundary. JSONB supplies indexable canonical subtrees, but exact source bytes remain separate because JSONB decomposes/reorders representation and cannot prove byte fidelity. **[I/P]**

Use structured columns for:

- stable identity, resource/revision type, parent/owner/tenant, lifecycle and optimistic-concurrency token;
- temporal axes and ranges used for selection or exclusion constraints;
- geometry/geography and explicit CRS-related metadata;
- foreign-key endpoints and typed relationship predicates;
- stream/contract fingerprints, component paths, property/unit identity, sequence and idempotency keys;
- policy partitions, provenance references, audit correlation, and materialization watermarks; and
- every field required for predictable filtering, sorting, pagination, integrity, or policy enforcement.

Use JSONB for standards-shaped canonical trees, heterogeneous extensions, validation detail, command/observation result structures not promoted to high-value typed columns, and generated-document input. JSONB must have versioned schemas, size/depth limits, indexes driven by real access paths, and generated-column/extraction parity tests. Do not encode primary relationships, ownership, temporal truth, or access policy solely inside opaque JSON.

### 8.2 Port and Adapter Boundary

Domain services should call typed repository/unit-of-work interfaces shaped by Glaux invariants, not generic CRUD or an “any database” abstraction. The PostgreSQL adapter may use PostGIS, range, JSONB, generated column, GIN/GiST/BRIN, locking, and `RETURNING` behavior deliberately. Portability comes from stable domain contracts, exports, fixtures, and a second adapter’s contract tests—not from avoiding the capabilities that satisfy the full profile.

### 8.3 Rejected Authority Splits

A metadata-document store plus separate time-series database can scale, as the pygeoapi study shows, but it makes stream registration plus first Observation, derived extents, relationships, policy, and failure recovery cross-store operations. Glaux should not incur that consistency burden until one PostgreSQL authority is proven inadequate. Search indexes, analytical extracts, and object tiers can be added as projections because their loss does not erase domain truth.

---

## 9. Geospatial Persistence Implications

Full-scope CSAPI requires geometry-aware storage for Systems, Deployments, SamplingFeatures/Features of Interest, positions, bounding boxes, histories/trajectories, and compound spatial-temporal filters. PostGIS is the leading baseline because `geometry` and `geography`, spatial reference systems, validity checks, transformations, and GiST/SP-GiST/BRIN indexes live beside relational facts. **[I/P]**

IDR-SRV-026 must decide:

- canonical geometry versus exact GeoJSON/SensorML source and dimensionality;
- CRS storage, axis order, transformations, antimeridian/polar behavior, empty/null/invalid geometry;
- geometry versus geography per predicate;
- current versus historical geometry and trajectory representation;
- index families and compound attribute/temporal/spatial query plans;
- bbox derivation/materialization and update rules;
- precision, validity repair versus rejection, and provenance;
- feature collections, policy filtering, cursor stability, and explain-plan thresholds.

The high-level constraint is fixed here: the authoritative spatial representation and the resource/relationship/policy transaction must share a consistent commit boundary. PostGIS raster and topology modules are not selected; CSAPI evidence does not yet require them.

---

## 10. Time-Series Persistence Implications

Observations dominate expected volume and velocity, followed by status, events, ingestion receipts, and audit. Their access patterns include append, idempotent retry, time range, latest, stream/property/FOI filters, ascending/descending traversal, retention, replay, derived extents, and occasionally corrected/late data.

### 10.1 Native PostgreSQL Baseline

Begin benchmarking with declarative range partitioning, immutable contract/resource foreign keys, composite B-tree indexes for selective stream/time paths, BRIN indexes for large physically time-correlated partitions, and narrow current/latest projection tables. PostgreSQL documentation notes partitioning can make bulk archival/detach faster and BRIN is designed for very large naturally correlated tables. These are capability facts, not proof of Glaux throughput. **[I/X]**

The baseline must test:

- narrow scalar, multi-property record, array/vector, categorical, nil, quality, and large binary-result mixes;
- ordered and out-of-order ingest, duplicate/retry storms, backfill into cold partitions, clock skew, and schema revisions;
- latest-per-stream/property/System; wide and narrow time ranges; spatial/FOI/semantic joins; stable keyset pagination;
- partition creation/rotation, index build, vacuum/analyze, retention detach/export/drop, restore, and replication lag;
- transaction cost of provenance, validation receipts, extents, status projection, and outbox;
- hot-key/sequence contention and policy partitioning; and
- restart/crash recovery and no-loss acknowledgement boundaries.

### 10.2 TimescaleDB Gate

TimescaleDB 2.27.1 remains a serious candidate for the dynamic tables. Current documentation shows time-partitioned hypertables, chunks, row-to-column lifecycle storage, continuous aggregates, retention, and PostgreSQL compatibility. It also requires all unique indexes to include partition dimensions and documents limitations involving partition-key updates and hypertable-to-hypertable foreign keys. Continuous-aggregate retention must coordinate refresh windows or summaries can be removed after raw data is dropped. **[I]**

IDR-SRV-027 should select TimescaleDB only if a reproducible self-hosted benchmark shows material wins after accounting for:

- exact idempotency/identity constraints and foreign-key model;
- late/backfilled/corrected records and immutable semantics;
- JSON/SWE payload access and PostGIS joins;
- compression/columnstore behavior, continuous aggregates, and raw/summary retention interaction;
- licensing, extension availability, upgrade/backup/restore, ARM/tactical packaging, and CI matrix;
- standard PostgreSQL escape/migration path; and
- operational complexity versus native partitions.

### 10.3 Analytical Sidecar

DuckDB can query exported Parquet or snapshot data efficiently and is useful for offline analysis, benchmark adjudication, and conformance-result exploration. Its current concurrency documentation emphasizes single-process writers and large analytical rather than many small concurrent transactions. It must not receive live authoritative writes in the baseline. **[I/P]**

---

## 11. Metadata/Document Storage Implications

Accepted IDR-SRV-021 through 024 require exact source, parsed/canonical structures, generated representations, validation evidence, immutable schemas/contracts, semantic packages, and provenance. No single document value satisfies every layer.

### 11.1 Content-Addressed Artifact Contract

Each exact artifact should have:

`artifact_id, digest_algorithm, digest, length, media_type, encoding, source/provenance, policy label, created time, storage backend/key, encryption/integrity metadata, lifecycle state`.

Writes are immutable and verified by length/digest. Resource revisions reference artifacts; replacing a document creates a new artifact/reference. Deduplication by digest is permitted only within compatible tenant/policy/encryption domains so digest existence does not become a cross-domain oracle.

Small artifacts may remain in PostgreSQL `bytea` for transactional simplicity. Large artifacts may use a durable local filesystem or S3-compatible object adapter behind one interface. The database catalog remains authoritative for identity, policy, references, and expected digest. IDR-SRV-028 must benchmark thresholds and choose the default/backends.

### 11.2 Cross-Store Atomicity

A safe candidate sequence is stage object under a non-public temporary key, flush/verify digest, commit catalog/resource references, then promote or mark committed idempotently. Failures need orphan collection and missing-object detection. Deletes use tombstone/retention state before physical removal. Backups must coordinate catalog and objects to a common recovery point or carry repair manifests.

The Rust `object_store` crate demonstrates one feasible adapter spanning local and cloud-style backends, but its documentation exposes backend differences such as missing-delete behavior and local durability settings. The abstraction cannot erase consistency, fsync, conditional-write, listing, multipart, encryption, and error semantic differences. **[I/D]**

### 11.3 Search and Exactness

Full-text search should initially use PostgreSQL facilities/materialized columns where required. A separate search engine can be reconsidered only if measured discovery workloads cannot meet objectives and only as a rebuildable policy-partitioned projection. JSONB or a search document never replaces the exact artifact.

---

## 12. Event-Log, Audit, and Relationship Persistence Implications

### 12.1 Domain Events and Transactional Outbox

The authoritative System Event is a persisted Glaux resource/fact under IDR-SRV-020. A broker message is one delivery attempt or projection of that fact. A database WAL records engine changes, not the stable domain event contract. These layers must remain separate.

State-changing transactions should append a publishable outbox record with stable event identity, aggregate/resource identity, revision, event type, event/record time, correlation/causation, policy scope, payload/artifact reference, and publication state. A relay publishes only committed rows and records attempt/acknowledgement evidence. Consumers deduplicate on stable event/message identity. This avoids the database-commit/broker-publish dual-write gap while permitting NATS, Kafka, HTTP callbacks, or draft Part 3 adapters later. **[P]**

Kafka 4.1 and NATS JetStream offer durable retained streams, acknowledgement/replay, replication, and deduplication/transaction-related features, but their retention, compaction, partition, redelivery, and administrative semantics do not equal Glaux resource history. IDR-SRV-035 must choose bindings and delivery guarantees after Part 3/adoption decisions; IDR-SRV-030 must coordinate broker retention with authoritative history.

### 12.2 Audit and Tamper Evidence

Append-only tables prevent ordinary update paths but do not by themselves prevent privileged tampering, storage rollback, log omission, or administrator access. IDR-SRV-041 should combine least-privilege append paths, immutable identifiers/sequences, canonical record encoding, hash/signature/checkpoint evidence, protected export, external anchoring where required, monitored privilege use, and restore verification. Audit records reference domain/provenance facts rather than duplicating unbounded sensitive payloads.

An action acknowledged as successful must have either its required audit record committed in the same transaction or a formally proven durable handoff. “Log eventually if available” is inadequate for commands, security changes, policy decisions, synchronization conflicts, and destructive lifecycle operations.

### 12.3 Relationship Graph

The accepted relationship model consists of typed, directional, provenance-bearing, temporally and policy scoped edges. PostgreSQL edge tables with endpoint foreign keys, predicate/type columns, validity ranges, forward/reverse indexes, and selected closure/materialized path structures fit this workload. Recursive SQL supports bounded traversal; closure tables accelerate stable hierarchies; derived links remain rebuildable.

A graph database is not justified unless IDR-SRV-026/054 measurements show relational plans cannot meet defined traversal objectives. Adding one would require projection consistency, policy equivalence, backup/restore, synchronization, migration, and test duplication. It must never become a hidden second source of relationship truth.

---

## 13. DDIL, Synchronization, Cache, and Federation Implications

### 13.1 Autonomous Node State

A DDIL node must retain enough local authority to validate and serve its authorized profile without network access: resources/revisions, exact contracts and vocabulary packages, local observations/commands/events, policy decisions, durable outbox, peer configuration, sync manifests, per-origin sequence/watermark state, acknowledged ranges, missing dependencies, conflicts, tombstones, and quarantine/retry state.

Network reconnection does not grant another node’s row updates automatic authority. Stable IDs, source authority, origin, revision, validity, provenance, and policy are evaluated by an application-level synchronization protocol. Conflicts remain explicit facts until resolved; arrival order or database LSN cannot silently determine semantic truth. **[P/D]**

### 13.2 PostgreSQL Replication Boundary

PostgreSQL logical replication can publish subsets and preserve transaction order within a subscription, making it useful for controlled connected read replicas or analytical projections. Current PostgreSQL 18 documentation also states that DDL is not replicated, schema mismatch can stop apply, sequence and large-object handling have restrictions, and locally modified subscriber data can conflict, stop, skip, or be overwritten according to engine behavior. Row-security and replication-role interactions require deliberate configuration. **[I]**

Therefore logical replication is not the Glaux DDIL multi-writer conflict protocol. It may be an implementation tool behind a deployment profile only after IDR-SRV-043 maps engine behavior to source authority, policy, schema versions, conflict evidence, and replay safety.

### 13.3 Cache and Materialized-View Rules

Every cache/materialization declares:

- authoritative inputs and watermark;
- builder/query/profile/schema version and vocabulary/policy scope;
- key composition including tenant/principal-policy context where required;
- completeness/staleness state and last successful build;
- invalidation triggers or bounded expiry;
- deterministic rebuild and atomic swap procedure; and
- health behavior when stale, missing, or rebuilding.

Current status, DataStream extents/property summaries, relationship closures, search text, spatial envelopes, conformance/OAD projections, aggregates, and result counts are candidate materializations. A cache hit must not bypass authorization; a shared key must not mix policy domains. External Redis-like cache infrastructure is not selected because no accepted requirement yet outweighs its operational and invalidation cost.

### 13.4 Federation

Federated query results need source identity, authority, watermark/as-of, partial-result indicators, stable ordering, duplicate/collision handling, policy filtering, and provenance. Remote data may be cached, mirrored, referenced, or imported, but those states must be visible in the canonical model. A live fan-out query is not a durable local fact and must not claim complete results under disconnection.

---

## 14. Transaction, Consistency, Migration, and Concurrency Implications

### 14.1 Transaction Baseline

IDR-SRV-029 should define isolation and retry per operation. This report fixes the following minimums:

- no externally acknowledged resource mutation before canonical state, required relationships, provenance/validation, and outbox/audit obligations are durable;
- idempotency keys bind principal/source, operation, target scope, request digest, and result; reuse with different content is a conflict;
- optimistic concurrency uses an explicit resource revision/ETag rather than last-writer-wins;
- immutable Observation/Command/Event identities detect duplicates independently of arrival order;
- projectors are idempotent and can resume from durable watermarks;
- deadlock/serialization retries are bounded and must not duplicate side effects; and
- cross-object artifacts or broker writes use staged/reconciled protocols, never an unexamined distributed transaction assumption.

PostgreSQL supports Read Committed, Repeatable Read, Serializable, explicit locks, advisory mechanisms, deferrable constraints, and transaction-scoped outbox changes. Which operations use which mechanisms remains IDR-SRV-029’s decision and test responsibility.

### 14.2 Migration Rules

Schema evolution must use explicit, ordered, immutable migration artifacts with checksums and a compatibility ledger. Startup may verify schema level and optionally run explicitly enabled safe migrations, but must not auto-mutate production based on ORM model drift. Migrations identify application-version windows, extension/version prerequisites, lock/disk/time risk, backfill/checkpoint behavior, rollback or forward-recovery method, and backup/restore prerequisites.

Every migration class needs tests from the oldest supported source state, representative large partitions/artifacts, interrupted execution, restart/resume, mixed-version application window, restore, and derived-view rebuild. PostgreSQL, PostGIS, TimescaleDB, object catalogs, vocabulary packages, and brokers each have distinct upgrade state; the release manifest must pin and check their compatibility.

### 14.3 Pagination and Snapshot Consistency

Persistent ordering requires a total stable key reflecting the approved query semantics. Keyset cursors should bind query/profile/policy/principal context, sort direction, last key, and expiry/integrity. Whether a cursor also pins an `asOf` snapshot or tolerates concurrent inserts/deletes must be explicit per collection. Offset pagination alone is unsuitable for deep high-rate dynamic collections, but exact API behavior remains tied to accepted IDR-SRV-011 and later implementation research.

---

## 15. Security, Policy, Provenance, and Audit Implications

### 15.1 Protection and Isolation

Later security topics must classify and protect resource rows, exact artifacts, dynamic values, semantic mappings, derived indexes, caches, backups, WAL/replication streams, broker topics, telemetry, and test exports. Encryption at rest is a deployment system consisting of database/object/broker volumes, key management, backup encryption, rotation, restore, and operator access—not a boolean database property.

PostgreSQL row-level security can provide defense in depth, but current documentation notes owners normally bypass it, `TRUNCATE`/`REFERENCES` are outside ordinary row policies, and replication roles/policies interact in ways that can halt or bypass expected behavior. Glaux must enforce authorization in its domain/query layer and validate database roles/RLS as an independent barrier, not depend on RLS alone. **[I/P]**

### 15.2 Policy-Correct Derivations

- Policy attributes used by filters must be structured and indexed, not buried only in JSON.
- A derived row inherits or computes policy from every contributing fact using a versioned policy decision.
- Counts, sort position, cursors, summaries, closure edges, spatial envelopes, aggregates, cache keys, and query plans cannot leak denied facts.
- Search/graph/broker projections must apply the same policy snapshot and record their watermark; stale-policy projections fail closed.
- Content-addressed deduplication and “already exists” responses cannot reveal a restricted object across tenants.
- Database statistics, diagnostics, slow-query samples, replication conflict logs, and dead-letter records may contain protected identifiers/values.

### 15.3 Provenance and Restore

Backups, exports, migrations, repairs, compaction, conversions, downsampling, reindexing, and synchronization are provenance-producing activities. A successful restore must prove catalog/artifact consistency, schema/package compatibility, row counts/digests, policy configuration, audit-chain continuity, and derived-view freshness. Restore testing is a release requirement, not only an operator document.

---

## 16. Rust Ecosystem, Deployment, CI, Fixture, and Conformance-Test Implications

### 16.1 Rust Data Access

The leading shortlist is:

| Candidate | Strength for Glaux | Risk/question | Disposition |
|---|---|---|---|
| SQLx 0.9 | Async pools, PostgreSQL/SQLite drivers, explicit SQL, typed query macros, offline query metadata, embedded migrations | PostGIS/custom/range mappings, compile metadata workflow, dynamic query composition need proof | Leading prototype candidate; not yet selected |
| Diesel | Mature typed query DSL and migration ecosystem, PostgreSQL/SQLite | Async/dynamic compound CSAPI query ergonomics and PostGIS integration need proof | Prototype alternative |
| SeaORM 2.0 | Async ORM over SQLx, dynamic query building, migration and mock support | ORM abstraction may obscure database-native features or encourage model-driven schema mutation | Evaluate for administrative CRUD, not assumed core |
| Direct driver + query layer | Maximum PostgreSQL/PostGIS control | More mapping, migration, instrumentation, and test infrastructure | Fallback/targeted hot paths |

Repository ports should accept domain query/value objects and transactions, not expose crate types across the core. Final selection belongs to later implementation-platform/service-architecture and TDD topics after a vertical prototype exercises spatial-temporal-semantic filters, JSON/SWE values, migrations, transactions, cancellation, streaming reads, custom types, tracing, and failure injection.

SQLx documentation requires a live schema or committed offline metadata for checked queries and notes migration hash sensitivity to platform line endings. If selected, CI must regenerate/verify offline metadata deterministically, fix migration files to LF, and run queries against the pinned real database/extensions. **[I]**

### 16.2 Deployment Profiles

- **Full profile:** self-hosted PostgreSQL/PostGIS as required services; optional object/broker/Timescale adapters explicitly advertised.
- **Developer/CI profile:** real pinned PostgreSQL/PostGIS containers for repository, migration, query, conformance, and integration tests; lightweight in-memory fakes only for domain unit tests.
- **Reduced edge profile candidate:** SQLite or a constrained local PostgreSQL packaging, explicitly declaring unsupported/different capabilities. SQLite WAL requires one host and one writer at a time; minimum safe versions must include the March 2026 WAL-reset fix (3.51.3, or documented fixed backports 3.50.7/3.44.6). **[I/X]**
- **Analytical profile:** exported Parquet/artifacts queried with DuckDB; read-only with respect to Glaux authority.

SQLite tests cannot claim PostgreSQL/PostGIS query, isolation, RLS, migration, extension, or concurrency conformance. Contract suites may be shared, but database-specific suites and full-profile CI remain mandatory.

### 16.3 Test Architecture

1. Pure domain tests for authority, lifecycle, retention classification, idempotency, and projection invariants.
2. Reusable repository contract tests, plus adapter-specific capability tests.
3. Real PostgreSQL/PostGIS integration tests for migrations, custom types, constraints, isolation, spatial queries, partitioning, and explain plans.
4. Artifact-store contract tests across memory, local filesystem, and selected S3-compatible implementation, including crash/orphan/missing-object cases.
5. Outbox/broker tests for duplicate, ordering, retry, replay, acknowledgement, outage, recovery, and policy partitioning.
6. Migration/upgrade/rollback-or-forward-repair and backup/restore drills using versioned fixtures.
7. DDIL bundle/apply/conflict/replay tests with independent node clocks and policy scopes.
8. Performance/load/soak tests using representative rates, payloads, cardinalities, late data, retention, and slow clients.
9. API/conformance tests proving persistence restart, pagination stability, cross-representation identity, query equivalence, and policy non-interference.

Testcontainers or equivalent reproducible orchestration should pin image digests/extensions and collect plans, versions, configuration, seeds, timing, hardware limits, and result artifacts. A mocked repository passing cannot establish persistent behavior.

---

## 17. Implementation Lessons and Risk Analysis

### 17.1 Lessons to Adopt

- OSH: typed store/filter ports, store-specific indexes, separate workload modules, and explicit restart/consistency tests.
- CS-Go: PostgreSQL/PostGIS, native spatial indexes, structured associations, JSONB for heterogeneous data, cursor indexes, and transactional cascades.
- pygeoapi: pinned/offline schema artifacts and explicit provider boundaries; its cross-store split is a caution requiring repair/rebuild contracts.
- SECD: realistic high-volume observation corpus and restart/query tests, without inferring an unavailable internal design.
- All studies: deployed behavior, OAD/conformance, storage migrations, fixtures, and live endpoints can drift unless release-derived and continuously tested.

### 17.2 Lessons to Avoid

- startup ORM auto-migration as production evolution control;
- all-standard payloads as opaque JSON/documents with no relational invariants;
- denormalized format-specific documents as the only truth;
- independent writable feature/time-series/search stores without a defined commit/reconciliation model;
- read/write database routing that can acknowledge invisible or silently lost updates;
- fixed development credentials and cloud-only services in the required profile;
- offset-only pagination and unindexed compound filters at scale;
- broker or cache availability determining correctness;
- treating durable-looking live data or database error text as proof of architecture; and
- using implementation fixtures as normative golden responses without provenance/license/schema adjudication.

### 17.3 Option Risk Register

| Risk | Trigger | Impact | Mitigation/gate |
|---|---|---|---|
| PostgreSQL becomes an unbounded monolith | Every artifact/value/index placed in hot tables | Vacuum, backup, contention, operator burden | Category-specific tables/artifacts, partitions, benchmarks, retention |
| Premature Timescale dependency | Selected before idempotency/late-data/FK tests | Lock-in or broken invariants | Native baseline plus 027 benchmark/exit test |
| JSONB escape hatch | Relationships/policy/keys hidden in documents | Weak integrity/query/security | Structured promotion rules and schema/extraction parity |
| Polyglot consistency loss | Separate writable stores | Partial commits and stale policy | One authority, outbox, rebuildable projections |
| Object catalog/blob divergence | Crash between object and DB commit | Orphans or missing source | Stage/verify/catalog/reconcile protocol and common restore manifest |
| Broker-as-truth | Domain history aged/compacted by broker | Lost audit/event semantics | Authoritative event rows and outbox replay |
| Graph projection drift | Separate relationship engine | Wrong traversal/policy | Relational authority; versioned rebuildable graph only if justified |
| SQLite parity assumption | Edge/test result generalized to full profile | Concurrency/spatial/SQL errors | Explicit capability profile and production DB CI |
| DDIL via raw replication | Multi-writer conflict under disconnection | Silent overwrite/stopped replication | Application operation log and 043 conflict model |
| Policy leakage through indexes/cache | Derivation ignores denied inputs | Sensitive inference | Policy-aware materialization and non-interference tests |
| Migration outage/corruption | Unbounded DDL/backfill/extension upgrade | Service/data loss | Versioned rehearsed migrations, backup/restore, compatibility window |
| Benchmark theatre | Synthetic narrow happy path | Wrong product/partition decision | Representative corpus, published harness/config/plans, multiple workloads |

No implementation precedent creates a standards obligation. These studies establish useful patterns and failure hypotheses only.

---

## 18. Downstream Topic Handoff Matrix

| Topic | Required handoff | Decision/test owner |
|---|---|---|
| IDR-SRV-026 | PostGIS baseline; canonical/exact geometry split; spatial/temporal/policy plans; CRS/index/bbox/trajectory questions | Final geospatial storage/query strategy |
| IDR-SRV-027 | Native PostgreSQL partition/BRIN baseline; TimescaleDB 2.27.1 gate; representative ingest/query/late-data/retention benchmark | Final time-series storage strategy |
| IDR-SRV-028 | Content-addressed artifacts, JSONB/structured split, bytea/object thresholds, source/canonical/generated layers, orphan/restore protocol | Final metadata/document storage strategy |
| IDR-SRV-029 | Atomicity groups, idempotency, optimistic concurrency, isolation/retry, outbox, projector watermarks, cross-store staging | Final transaction/concurrency strategy |
| IDR-SRV-030 | Per-category retention, archive/tier/export/delete, tombstones, evidence holds, backup/object/broker coordination | Final lifecycle/retention strategy |
| IDR-SRV-031-034 | Publisher receipts, transaction acknowledgement, exact contract binding, dynamic record identity, summaries/extents/current projections | Final ingestion/dynamic-data behavior |
| IDR-SRV-035 | Transactional outbox and broker-as-projection boundary | Streaming/event architecture and Part 3 profile |
| IDR-SRV-036-038 | Command state/transition rows, exact values/schema, atomic audit/outbox and no silent conversion | Command, feasibility, authorization/safety behavior |
| IDR-SRV-039/039A/040 | Database/artifact/broker threat model, RLS limits, policy-aware indexes/caches/dedup/replication | Security and releasability architecture |
| IDR-SRV-041 | Append-only versus tamper-evident audit, durable action boundary, external anchoring/export | Audit/accountability strategy |
| IDR-SRV-042-043 | Autonomous node inventory, operation logs, peer watermarks, bundles, tombstones, explicit conflicts; logical-replication limit | DDIL and synchronization semantics |
| IDR-SRV-045 | One authoritative core with adapter seams and unit-of-work boundary | Service modularization |
| IDR-SRV-046 | Full/developer/reduced-edge/analytical deployment profiles and pinned dependencies | Reference deployment |
| IDR-SRV-047 | DB/object/broker URLs, extension pins, package/profile config, trust roots, secret references | Configuration/secrets |
| IDR-SRV-048 | Pool, transaction, partition, outbox, replication, object, broker, cache, migration, backup health/metrics | Observability |
| IDR-SRV-049 | Coordinated schema/extension/object/package/broker migration, backup, restore, and compatibility | Upgrade/backup/restore |
| IDR-SRV-052 | SQLx/Diesel/SeaORM vertical prototypes, repository contract suites, real-database TDD | Rust test-driven architecture |
| IDR-SRV-053 | Versioned database states, migrations, spatial/dynamic/artifact/DDIL fixtures with provenance/license | Fixture corpus |
| IDR-SRV-054 | Representative ingest/query/paging/retention/replay/restore benchmarks and thresholds | Performance strategy |
| IDR-SRV-050/051/055/056 | Persistence restart and policy non-interference in conformance, security, and interoperability tests | Verification matrices |
| IDR-SRV-057 | Carry the selected direction, conditional extensions, and unresolved gates | Final synthesis after acceptance |

---

## 19. Recommendations

1. **Adopt PostgreSQL/PostGIS as the full-profile authoritative persistence direction.** Keep canonical state, relationships, temporal/policy facts, contracts, events, commands, provenance, and coordination in one transactional boundary. Priority: Critical.
2. **Use a relational/JSONB/artifact hybrid.** Structure invariant/query/security keys; retain heterogeneous canonical trees in validated JSONB; preserve exact bytes in immutable content-addressed artifacts. Priority: Critical.
3. **Start time-series proof with native PostgreSQL partitioning and indexes.** Establish a portable correctness/performance baseline before enabling extensions. Priority: High.
4. **Benchmark TimescaleDB in IDR-SRV-027, but do not require it now.** Test idempotency, foreign keys, late data, compression, retention, PostGIS/JSON joins, upgrades, licensing, edge packaging, and exit path. Priority: High.
5. **Use typed domain repository and unit-of-work ports.** Do not pretend lowest-common-denominator SQL portability; contain database-specific details in adapters and prove contracts. Priority: High.
6. **Use relational typed edges and selective closure materialization.** Do not add a graph database without a measured traversal failure. Priority: High.
7. **Use a transactional outbox.** Persist domain events/audit obligations with state changes, then publish idempotently; brokers remain derived delivery systems. Priority: Critical.
8. **Make caches and materialized views disposable and evidence-bearing.** Record source watermark, builder/profile/policy version, staleness, and deterministic rebuild. Priority: High.
9. **Define a content-addressed artifact adapter.** Keep the catalog transactional; benchmark PostgreSQL `bytea` versus local/S3-compatible storage and implement staged commit/repair. Priority: High.
10. **Treat PostgreSQL logical replication as a tool, not DDIL semantics.** Use application-level authority, operation, acknowledgement, tombstone, conflict, and policy evidence. Priority: Critical.
11. **Permit SQLite only under an explicit reduced profile.** Pin a fixed version and never use SQLite-only success as full PostgreSQL/PostGIS conformance evidence. Priority: Medium.
12. **Use DuckDB only for analytical exports/benchmarks unless later evidence expands its role.** Priority: Medium.
13. **Require explicit numbered migrations and coordinated restore tests.** Reject ungoverned startup ORM mutation. Priority: Critical.
14. **Prototype SQLx first while retaining Diesel and SeaORM comparisons.** Exercise actual Glaux compound queries and migrations before final crate selection. Priority: Medium.
15. **Make security and policy part of every authoritative and derived persistence contract.** RLS is defense in depth; protect backups, replication, caches, indexes, broker state, diagnostics, and deduplication side channels. Priority: Critical.
16. **Gate specialization with published, reproducible workload evidence.** Every added store/extension must beat the baseline enough to justify its failure domain, migration, CI, security, backup, and operator cost. Priority: High.

---

## 20. Risks, Constraints, and Open Questions

### 20.1 Constraints

- Glaux must be open-source, reproducible, self-hostable, and suitable for disconnected/tactical contexts; no mandatory cloud-managed dependency is acceptable.
- Approved standards specify observable behavior but do not prescribe database products or retention.
- Exact source fidelity, canonical query behavior, append evidence, and current projections require separate persisted layers.
- Security, authorization, policy, and audit designs are not complete; physical schemas cannot safely freeze their fields yet.
- No representative Glaux load corpus or service-level objective exists, so performance-based product claims remain provisional.
- Category E topics remain sequential; this report must not preempt their detailed decisions.

### 20.2 Open Questions

| Question | Why open | Resolution path |
|---|---|---|
| Native PostgreSQL versus TimescaleDB for each dynamic table | No representative benchmark; constraint/late-data tradeoffs | IDR-SRV-027 + 054 benchmark |
| Exact partition keys and intervals | Depend on stream cardinality, policy, retention, query mix, memory/storage | IDR-SRV-027/030/054 |
| Geometry/geography and historical position representation | CRS/predicate/trajectory semantics not finalized | IDR-SRV-026 |
| Artifact inline/object threshold and default backend | Payload distribution and backup/atomicity costs unknown | IDR-SRV-028/049 prototype |
| Whether full-text search needs a separate engine | No workload demonstrates PostgreSQL shortfall | IDR-SRV-028/054 |
| Exact transaction isolation/locking per mutation | Operation state machines not fully defined | IDR-SRV-029 |
| Retention and legal/mission holds | Profile/policy inputs unavailable here | IDR-SRV-030/040/041 |
| Broker technology and durability/QoS | Transport architecture/Part 3 profile not final | IDR-SRV-035 |
| SQLite reduced-profile scope | Required edge rate, spatial capability, sync model unknown | IDR-SRV-042/043/046/054 |
| SQLx, Diesel, SeaORM, or mixed access | Compound-query/custom-type prototype absent | IDR-SRV-045/052 |
| RLS/partitioning/replication policy design | Security and dissemination model pending | IDR-SRV-039A/040/043 |
| Tamper-evident audit mechanism | Threat/legal/external-anchor needs pending | IDR-SRV-041 |
| Recovery point/time objectives | Deployment/profile operations research pending | IDR-SRV-046/049 |

These questions do not block the architecture direction. Each optional dependency and irreversible detail remains behind a named evidence gate.

---

## 21. Validation Against Plan Success Criteria

| Topic plan success criterion | Status | Evidence |
|---|---|---|
| Data categories and persistence responsibilities identified with anchors | Met | Sections 3-6; 26-row controlling inventory |
| Authority, history, logs, documents, relationships, caches, views, synchronization, audit, and fixtures distinguished | Met | Sections 4-6 and 12-13 |
| Candidate patterns/options evaluated against explicit criteria | Met | Section 7 weighted rubric and comparison |
| Geospatial, time-series, document, event, relationship, transaction, DDIL, security, and testing implications documented | Met | Sections 8-16 |
| Rust, deployment, local development, CI, and conformance implications documented | Met | Section 16 |
| Implementation/community evidence incorporated as non-normative | Met | Sections 3.3 and 17 |
| Recommendations decision-usable and bounded to Glaux | Met | Sections 2.2 and 19 |
| Downstream handoffs explicit | Met | Section 18 |
| References explicit and reproducible | Met | Sections 3 and 22; mutable snapshots/version pins recorded |

### 21.1 Report Completion Checklist

- [x] Topic ID matches the overall plan index.
- [x] Topic plan is linked and aligned.
- [x] All core and detailed question groups are covered or routed.
- [x] Normative, project, product, implementation, deferred, and unresolved evidence are separated.
- [x] Mutable technology evidence records versions and access date.
- [x] Controlled-source limitation is explicit; no controlled text is redistributed.
- [x] The minimum 15-field data-category matrix is complete.
- [x] Architecture options use explicit criteria and decision gates.
- [x] Recommendations, risks, open questions, and handoffs are explicit.
- [x] Plan-owner acceptance remains pending review.

---

## 22. References

### Project, Governance, and Accepted Research

- [IDR-SRV-025 research plan](../IDR%20Plans/idr-srv-025-database-and-persistence-architecture-options.md).
- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md), Version 3.22 working baseline.
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md).
- [Research Planning Approach](../../../../../Governance/research-planning-approach.md).
- [Research Report Template](../../../../../Governance/research-report-template.md).
- [OGC API - Connected Systems upstream-history evidence register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), Version 1.10.
- Accepted IDR-SRV-001 through IDR-SRV-024 reports in this directory, especially IDR-SRV-014A through 014G and IDR-SRV-015 through 024.
- Project-controlled `AC/224(JCGISR)D(2026)0005`, April 27, 2026, SHA-256 `56dc757b6e677b3584e3152a957849f21a24b22854f562613ff283a8b599da8c`; accepted findings only, source not redistributed.

### Approved Standards

- Open Geospatial Consortium, [OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html), published 2025-07-16.
- Open Geospatial Consortium, [OGC 23-002, *OGC API - Connected Systems - Part 2: Dynamic Data*, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html), published 2025-07-16.
- Open Geospatial Consortium, [tagged CSAPI Version 1.0 source and API artifacts, commit `8e03b236`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2).
- Open Geospatial Consortium, [OGC 23-000, *OGC SensorML Encoding Standard*, Version 3.0](https://docs.ogc.org/is/23-000/23-000.html), published 2025-07-16.
- Open Geospatial Consortium, [OGC 24-014, *OGC SWE Common Data Model Encoding Standard*, Version 3.0.0](https://docs.ogc.org/is/24-014/24-014.html), published 2025-07-16.
- Open Geospatial Consortium, [OGC API - Features - Part 1: Core](https://docs.ogc.org/is/17-069r4/17-069r4.html).
- W3C and OGC, [*Semantic Sensor Network Ontology*, 2017 Recommendation](https://www.w3.org/TR/2017/REC-vocab-ssn-20171019/).

### Persistence and Rust Primary Sources

- PostgreSQL Global Development Group, [PostgreSQL 18 documentation](https://www.postgresql.org/docs/18/): [data types/JSONB](https://www.postgresql.org/docs/18/datatype.html), [partitioning](https://www.postgresql.org/docs/18/ddl-partitioning.html), [BRIN](https://www.postgresql.org/docs/18/brin.html), [row security](https://www.postgresql.org/docs/18/ddl-rowsecurity.html), [concurrency control](https://www.postgresql.org/docs/18/mvcc.html), and [logical replication](https://www.postgresql.org/docs/18/logical-replication.html). Accessed 2026-09-14.
- PostGIS Project, [released PostGIS manuals](https://postgis.net/documentation/manual/) and [PostGIS 3.6 manual](https://postgis.net/docs/manual-3.6/en/). Accessed 2026-09-14.
- Timescale, [TimescaleDB 2.27.1 release](https://github.com/timescale/timescaledb/releases/tag/2.27.1), [hypertables](https://docs.timescale.com/use-timescale/latest/hypertables/), [limitations](https://docs.timescale.com/use-timescale/latest/limitations/), [unique indexes](https://docs.timescale.com/use-timescale/latest/hypertables/hypertables-and-unique-indexes/), and [retention with continuous aggregates](https://docs.timescale.com/use-timescale/latest/data-retention/data-retention-with-continuous-aggregates/). Accessed 2026-09-14.
- SQLite Project, [Write-Ahead Logging](https://www.sqlite.org/wal.html), [Isolation](https://www.sqlite.org/isolation.html), and [Appropriate Uses for SQLite](https://www.sqlite.org/whentouse.html). Accessed 2026-09-14.
- DuckDB Foundation, [Concurrency](https://duckdb.org/docs/current/connect/concurrency) and [workload tuning](https://duckdb.org/docs/current/guides/performance/how_to_tune_workloads). Accessed 2026-09-14.
- Apache Software Foundation, [Apache Kafka 4.1 Design](https://kafka.apache.org/41/design/design/). Accessed 2026-09-14.
- Synadia/NATS Authors, [NATS JetStream concepts](https://docs.nats.io/nats-concepts/jetstream). Accessed 2026-09-14.
- LaunchBadge, [SQLx 0.9.0 documentation](https://docs.rs/sqlx/0.9.0/sqlx/), including [`query!`](https://docs.rs/sqlx/0.9.0/sqlx/macro.query.html) and [`migrate!`](https://docs.rs/sqlx/0.9.0/sqlx/macro.migrate.html). Accessed 2026-09-14.
- Diesel project, [Diesel documentation](https://diesel.rs/). Accessed 2026-09-14.
- SeaQL, [SeaORM 2.0 documentation](https://www.sea-ql.org/SeaORM/docs/). Accessed 2026-09-14.
- Apache Arrow Rust project, [`object_store` 0.14.1 documentation](https://docs.rs/object_store/0.14.1/object_store/). Accessed 2026-09-14.
- OpenTelemetry Authors, [OpenTelemetry documentation](https://opentelemetry.io/docs/), used only for later telemetry correlation context. Accessed 2026-09-14.

---

**Review gate:** This report is complete and in review. It becomes an accepted downstream baseline only after the Glaux Project Lead records acceptance in this report, the topic plan, and the overall plan.

# Section 029: Transaction, Consistency, Idempotency, and Concurrency Strategy - Research Report

**Topic ID:** IDR-SRV-029<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-029 Transaction, Consistency, Idempotency, and Concurrency Strategy](../IDR%20Plans/idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions, all detailed question groups, all 6 methodology phases, all 10 success criteria, and the required 15-field operation-family transaction matrix<br>
**Methodology Used:** Authority-ranked extraction from HTTP semantics, approved OGC standards, controlled-source findings, and accepted IDR-SRV-001 through IDR-SRV-028; direct review of current PostgreSQL, SQLx, Kafka, and NATS primary documentation; operation/consistency/failure classification; and bounded synthesis into transaction, idempotency, concurrency, outbox/inbox, and retry contracts<br>
**Research Time:** Approximately 17 hours of AI-assisted execution on September 14, 2026<br>
**Official Standards Source Pin:** [`opengeospatial/ogcapi-connected-systems` `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Shared Register Baseline:** [OGC API - Connected Systems upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), Version 1.10; upstream `master` remained `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f` on September 14, 2026<br>
**Technology Documentation Snapshot:** PostgreSQL 18.6, SQLx 0.9.0, Apache Kafka 4.2 documentation, and current NATS JetStream documentation; checked September 14, 2026 and treated as mutable implementation evidence<br>
**Controlled AEP Source:** `AC/224(JCGISR)D(2026)0005`, April 27, 2026, SHA-256 `56dc757b6e677b3584e3152a957849f21a24b22854f562613ff283a8b599da8c`; used only through accepted project findings and not redistributed<br>
**Document Purpose:** Establish the Glaux Server atomicity, consistency, idempotency, concurrency, conflict, retry, replay, outbox/inbox, command-safety, DDIL, and verification baseline without defining final DDL, implementing draft Part 3, or implementing the server<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 14, 2026<br>
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

“Exactly once” is not used as a marketing synonym for deduplication. This report distinguishes database atomicity, request idempotency, effectively-once committed effects, at-least-once transport, message deduplication, and real-world device action. A broker acknowledgement cannot prove that a command was executed once, and a unique database row cannot prove that a message was delivered once.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Transaction Requirement Extraction Methodology
5. Operation-Family Transaction Inventory
6. Consistency Model Strategy
7. Transaction Boundary Findings
8. Idempotency and Duplicate-Detection Findings
9. Concurrency-Control and Conflict-Resolution Findings
10. Ingestion, Replay, and Source-Offset Findings
11. Status/Event, Outbox/Inbox, and Durable-Publication Findings
12. Command, Feasibility, and Command-Status Transaction Findings
13. DDIL, Federation, Synchronization, and Last-Known-State Findings
14. Error, Retry, Stale-State, and Problem-Detail Behavior Findings
15. Security, Policy, Provenance, and Audit Implications
16. Rust Implementation, Test, Fixture, Conformance, Performance, and Interoperability Implications
17. Downstream Topic Handoff Matrix
18. Recommendations
19. Risks, Constraints, and Open Questions
20. Validation Against This Plan's Success Criteria
21. References

---

## 1. Executive Summary

Glaux should implement a **PostgreSQL-centered unit-of-work model**. One authoritative mutation transaction commits the affected canonical rows, immutable revision/fact records, identity/admission or idempotency record, provenance activity, validation summary, current projection changes, lifecycle/domain-event fact, transactional outbox entry, and audit reference required to describe the successful change. Intermediate state is never externally visible. External object finalization, broker publication, device dispatch, remote synchronization, derived-cache refresh, and large report generation occur outside that transaction through durable state machines. **[P]**

The default database isolation should remain **Read Committed with declarative constraints and atomic compare-and-set statements** for simple mutations. Use row locks for state-machine aggregates and targeted **Serializable** transactions when correctness depends on a predicate or multi-row invariant that constraints and deterministic locks cannot express safely. PostgreSQL Serializable detects dangerous read/write patterns by aborting a transaction; the application must retry the complete unit of work.[^1] A global Serializable default is not selected before workload evidence. **[I/P]**

Mutable resource replacement, patch, and delete should require a strong representation `ETag` and `If-Match`. HTTP defines `If-Match` as a lost-update precondition and requires strong comparison; a false condition normally yields `412 Precondition Failed`.[^2] Glaux should return `428 Precondition Required` when a protected operation lacks the condition.[^3] Domain revision and `ETag` remain distinct: the revision identifies canonical history, while the `ETag` identifies the selected, policy- and representation-specific response state. **[N/P]**

HTTP method idempotency is necessary but insufficient. GET/HEAD are safe; PUT and DELETE have idempotent intended effects; POST and PATCH are not inherently idempotent.[^2] Every unsafe replay-prone Glaux operation—especially create-by-POST, Observation/status/event ingestion, command/feasibility submission, import, and synchronization—should accept or require a scoped idempotency key or a source-native immutable message identity. The server atomically stores scope, key, normalized operation fingerprint, state, resulting resource/revision, and replayable response metadata. Same key/same intent returns the original committed result; same key/different intent is a conflict. **[P]**

“Effectively once” is the supported local outcome: a unique inbox/idempotency/admission record and the domain effect commit in one PostgreSQL transaction. Delivery remains at least once where acknowledgement loss can cause redelivery. No end-to-end exactly-once claim is made across PostgreSQL, a broker, an object store, a remote peer, or a physical command target. Kafka's own documentation warns that delivery guarantees split into publication and consumption and scopes exactly-once features to coordinated Kafka transactions/Streams.[^4] **[I/P]**

Every publishable change writes an immutable domain/lifecycle event and an outbox row in the same transaction as the authority change. Workers claim committed outbox rows with short transactions, bounded leases/attempts, and `FOR UPDATE SKIP LOCKED`; PostgreSQL explicitly describes `SKIP LOCKED` as suitable for queue-like tables but not a consistent general-purpose view.[^5] Broker message IDs derive from immutable event/outbox IDs. A publish acknowledgement updates delivery state, but duplicate publication remains possible and consumers must deduplicate. **[P]**

Commands receive the strongest boundary. Command intent, exact contract/profile binding, authorization/safety decision, idempotency record, immutable accepted/rejected status, dispatch intent/outbox, provenance, and audit correlation commit before external dispatch. A retry never silently creates a second command. Device dispatch is not repeated unless the target protocol has a proven idempotency/correlation contract or evidence proves the prior attempt was not applied. CommandStatus changes are append-only, state-machine checked, and serialized per command; cancellation, completion, and timeout races produce explicit retained outcomes. **[P/D]**

Observations, status reports, and events are append-oriented. Source identity/partition/sequence/message ID and exact contract fingerprint provide duplicate detection; source offsets advance in the same transaction as the accepted inbox/domain record. Late and out-of-order facts remain valid historical input and never overwrite a latest projection merely because they arrived later. Corrections and same-time conflicts use explicit revision/source policy, not arrival-order last-write-wins. **[P]**

DDIL synchronization cannot be one distributed transaction. Each node commits locally, exchanges immutable manifests/messages at least once, deduplicates in an inbox, and records branch, authority, causal/transaction clocks, tombstones, policy, and conflicts. Safe automatic merges are limited to commutative set additions or byte-identical/idempotent facts under the same authority. Identity, schema/contract, temporal-overlap, command, policy, and divergent-authority conflicts are retained for deterministic policy or operator resolution. **[P/D]**

Retries apply to the entire side-effect-free database unit, never to an arbitrary statement after external action. PostgreSQL `40001` serialization failures and usually `40P01` deadlocks are retry candidates; unique/exclusion failures are interpreted using operation semantics, not blindly retried.[^6] Transactions are short, have bounded lock/statement timeouts, acquire multiple locks in a documented order, and never await user input, broker, remote HTTP, device, or large object I/O while holding locks. **[I/P]**

Detailed DDL, retention windows, ingestion batch semantics, command state machines, Part 3 transport binding, DDIL merge rules, authorization/audit schemas, pool sizing, retry budgets, and performance thresholds remain with the named downstream topics. **[D]**

---

## 2. Scope and Plan Alignment

This report executes `IDR-SRV-029`, the fifth Category E topic. It defines transaction units, visibility guarantees, request/message idempotency, conditional mutation, locks, conflicts, retry/replay, offsets, outbox/inbox, high-risk command and DDIL seams, public error behavior, and implementation/test gates.

It does **not** define final SQL DDL, retention durations, public write payloads, batch partial-success formats, command transition vocabulary, broker or draft Part 3 binding, DDIL merge algorithms, security policy, audit event schema, deployment topology, or production tuning. It does not implement the server. `IDR-SRV-030` and later topics remain unauthorized during this iteration.

### 2.1 Research Question Coverage Matrix

| Plan question | Short form | Status | Evidence location |
|---|---|---|---|
| Q1 | Which transaction and consistency guarantees are required? | Complete; physical DDL deferred | Sections 5-7 |
| Q2 | Which operations need idempotency, duplicate/replay handling, conditions, locks, or conflicts? | Complete | Sections 5, 8-10 |
| Q3 | How do concurrent API, ingest, command, sync, and workers interact safely? | Complete; detailed workflows delegated | Sections 7-13 |
| Q4 | How are identity, relationships, status/events, provenance, audit, and evidence preserved? | Complete | Sections 5-15 |
| Q5 | What implementation and verification implications follow? | Complete | Sections 16-19 |

### 2.2 Accepted-Baseline Reconciliation

- IDR-SRV-015's typed aggregate graph and event/outbox seam define mutation aggregates; representations are not transaction authorities.
- IDR-SRV-016 keeps ResourceId, revision, ETag, idempotency key, domain event, message, storage, and delivery identities distinct.
- IDR-SRV-017 typed edges and cycle/cardinality rules require atomic relationship validation with resource revisions.
- IDR-SRV-018/020 require bitemporal revisions, immutable facts, separate current projections, deterministic latest/tie rules, and non-collapse of domain event, status, audit, outbox, and message identities.
- IDR-SRV-019 requires provenance and trust evaluation in the mutation boundary, while denied/failed attempts need separately durable audit handling.
- IDR-SRV-021 through 024 require exact-source preservation, strict-write/quarantine imports, immutable contract/package binding, and new validation evidence on revalidation.
- IDR-SRV-025 through 028 establish PostgreSQL/PostGIS authority, typed temporal families, identity/admission ledger, immutable artifacts, staging/repair, and disposable derived caches.

---

## 3. Evidence Base and Authority Classification

### 3.1 Controlling Sources

| Source | Pin/status | Authority | Relevant anchors | Limit |
|---|---|---|---|---|
| RFC 9110 HTTP Semantics | Standards Track, June 2022 | Normative HTTP | §§9.2.2, 13.1-13.2, 15 status codes | Defines wire semantics, not Glaux domain transactions |
| RFC 6585 | Standards Track, April 2012 | Normative optional HTTP code | §3 `428 Precondition Required` | Server adoption is project decision |
| RFC 9457 | Standards Track, July 2023 | Normative error format | §§3-5 | Does not define domain conflict taxonomy |
| OGC API - Connected Systems Parts 1/2 | Approved 1.0; tag `v1.0.0`, `8e03b236...` | Normative | CRUD/update classes; cascade/collection behavior; DataStream/ControlStream schema immutability; command/status/feasibility resources | Write classes depend on OGC API - Features Part 4; some artifact/editorial conflicts retained |
| OGC API - Features Part 1 | Approved corrigendum | Normative dependency | HTTP status guidance and ETag recommendation | Read/core focus; transactional detail delegated |
| SensorML 3.0 / SWE Common 3.0 | Approved | Normative representation/contracts | document/data validity and immutable interpretation | Do not define database isolation/idempotency |
| Accepted IDR-SRV-001 through 028 | Accepted through 2026-09-14 | Project-controlling | domain, identity, relation, time, provenance, status, representations, persistence | Detailed downstream owners preserved |
| Controlled AEP source | `AC/224(JCGISR)D(2026)0005`, 2026-04-27, recorded digest | Project-controlled | accepted robustness, profile, DDIL, command/audit context only | Content not reproduced or extended by inference |

### 3.2 Mutable Primary Technology Evidence

| Source | Version/retrieval | Finding used | Authority limit |
|---|---|---|---|
| PostgreSQL documentation | 18.6, checked 2026-09-14 | transaction atomicity/visibility, MVCC/isolation, row/advisory locks, `ON CONFLICT`, `SKIP LOCKED`, serialization/deadlock retries | Capability, not API obligation |
| SQLx | 0.9.0, checked 2026-09-14 | pool-held transaction, commit/rollback/drop behavior, savepoints | Candidate Rust stack behavior |
| Apache Kafka docs | 4.2, checked 2026-09-14 | separation of delivery guarantees; broker-scoped idempotent/transactional processing | Comparative transport evidence only |
| NATS JetStream docs | current, checked 2026-09-14 | durable consumers, acknowledgements, redelivery/deduplication concepts | Comparative transport evidence only |

### 3.3 Standards Findings and Gaps

CSAPI Part 1 requires cascade deletion of a System and nested resources when requested and removal of an associated Deployment link; it also requires a successfully replaced resource to be reflected in every collection containing it. Those are multi-row atomicity requirements for Glaux. Part 2 requires `409` when changing DataStream/ControlStream schemas after child Observation/Command records exist and for default deletion of non-empty streams; it requires Observation and Command representations to validate against the exact parent schema before create/replace/update. Command cancellation is a new `CANCELED` CommandStatus, not deletion of the Command. **[N]**

The approved standards do not prescribe database isolation, idempotency-key headers, deduplication windows, broker guarantees, source-offset schemas, or DDIL conflict algorithms. HTTP provides conditional and method semantics, while Glaux must define the stronger project contract. An unpublished repository-level batch draft discusses atomic versus partial processing; because it is not treated as an approved controlling requirement here, batch behavior remains downstream/profile-defined. **[N/X]**

### 3.4 Evidence Quality

Standards obligations outrank project recommendations; accepted reports constrain this topic; controlled findings refine operational context; technology and implementation evidence establish feasibility only. No current upstream change altered the accepted CSAPI pin or shared register. Exact AEP transaction/command package requirements, external target guarantees, and retention windows remain unavailable or profile-owned and are explicitly deferred.

---

## 4. Transaction Requirement Extraction Methodology

Each operation was classified along five axes:

1. **Authority set:** every canonical, append, projection, artifact, provenance, validation, policy, event/outbox, idempotency/inbox, and audit reference affected.
2. **Atomic boundary:** what must either all commit or all remain absent, and which external effects require staged state machines.
3. **Visibility/ordering:** strong current-state, stable snapshot, read-your-writes, per-aggregate monotonic, causal, eventual, or best-effort derived behavior.
4. **Replay/concurrency:** HTTP precondition, client key, source identity/offset, unique constraint, compare-and-set, row/advisory lock, Serializable predicate, conflict record, retry rule.
5. **Failure evidence:** stable response/problem, validation/provenance/audit record, outbox/inbox status, reconciliation and downstream owner.

### 4.1 Evaluation Rules

- Prefer declarative uniqueness, foreign keys, checks, exclusion constraints, and one atomic SQL statement before application locks.
- Default to Read Committed for bounded row/set mutations; escalate isolation or locks only for a documented invariant.
- Use strong optimistic HTTP concurrency for human/client-managed mutable resources; serialize safety-critical state machines internally.
- Make unsafe retries recognizable before performing the effect.
- Commit no broker/device/network side effect while a database transaction is open.
- Promise effectively-once local effects and at-least-once delivery with deduplication; never infer end-to-end exactly once.
- Keep transactions short, ordered, timeout-bounded, observable, and completely retryable.
- Preserve every rejected/conflicting branch as validation/provenance/audit evidence where policy requires, without partially creating authority state.

---

## 5. Operation-Family Transaction Inventory

| Operation family | Related resource family | Affected data categories | Source topic / source anchor | Transaction boundary | Consistency requirement | Idempotency requirement | Concurrency-control pattern | Conflict type(s) | Retry/replay behavior | Event/audit behavior | Security/policy implication | Test implication | Downstream topic handoff | Notes / unresolved issues |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Feature registration by POST | System, Procedure, Deployment, Sampling Feature, Property | identity, canonical revision, relationships, source/validation/provenance, collection membership | CSAPI P1 CRUD; IDR-015-019/021 | one PG transaction including idempotency and outbox | strong committed state; read-your-write response | client key recommended/required by profile; source UID/authority duplicate check | unique identity indexes; atomic insert; Serializable/locks for cross-resource invariants | duplicate UID/source ID, missing/hidden parent, relation/cardinality | same key+fingerprint returns original; ambiguous failure resolved by key | lifecycle event/outbox and success audit reference atomic | authorize before revealing duplicate; policy snapshot bound | parallel same/different-key registration; lost response | 030-033, 039-041, 052-056 | Final key requirement/profile deferred |
| PUT/PATCH metadata revision | all mutable descriptive resources | new immutable revision, current pointer, normalized graph, relationships, generated-view invalidation | RFC 9110; RFC 6585; CSAPI P1/P2 update | one aggregate transaction | strong optimistic concurrency | HTTP intended-effect idempotency plus optional request key; PATCH key recommended | mandatory strong `If-Match`; revision CAS; row/edge locks; targeted Serializable | stale representation, semantic relation, temporal overlap, policy | `412` stale; safe full-unit retry only | revision/domain event/outbox/provenance/audit reference atomic | ETag is policy/representation scoped; conceal hidden facts | lost-update/write-skew/patch replay/property tests | 031-033, 039-041 | PATCH media/merge semantics deferred |
| Resource retirement/delete/cascade | System/streams and nested resources | lifecycle/tombstone, current membership, relations, retained evidence | CSAPI P1 R61/R68-70; P2 CRUD | whole normative cascade/member change in one transaction or durable admin job not exposed half-applied | strong for visible authority; archives/caches eventual | DELETE intended effect; operation ID for async cascade | `If-Match`; deterministic lock order; Serializable for predicate/cascade set | non-empty stream, new child during delete, retention/hold, association | repeated committed delete returns governed tombstone/success; job resumes by ID | delete/retire event/outbox/audit atomic; destruction later | authorization/policy/hold before existence detail | child-insert race, cascade rollback, huge cascade job | 030-033, 039-043 | Physical deletion never in API transaction by default |
| Relationship add/remove/custom collection membership | all feature families | typed edge, reciprocal/derived links, collection membership, revision/outbox | CSAPI P1; IDR-017 | subject/target/edge and affected collection revisions atomic | strong graph invariants | natural edge identity or operation key | FK/unique/exclusion; lock endpoints in stable ID order; Serializable for cycle predicates | missing/hidden endpoint, cycle, cardinality, duplicate edge | duplicate exact edge succeeds/no-op; divergent edge conflicts | edge lifecycle/outbox/provenance/audit reference atomic | authorize both endpoints and result visibility | concurrent inverse/cycle/cardinality cases | 031-033, 039-040, 052-056 | Cross-node edges become sync conflicts |
| SensorML/SWE import and promotion | feature/stream resources | exact artifact, parsed/canonical state, package binding, validation, provenance | IDR-021-024/028 | staging outside; promotion catalog/resource/evidence/outbox in one PG transaction | no visible partial normalization | source acquisition ID/digest plus workflow key | unique acquisition; package snapshot; CAS current revision | invalid/profile-divergent, identity conflict, package change | strict reject or quarantine; promotion retry by immutable activity | validation/provenance plus publish event on promotion; rejection audit separate | untrusted input and controlled source restrictions | crash at every stage; same bytes/different authority/policy | 030-031, 039-041, 047-053 | External object finalization uses repair state |
| Schema/profile/vocabulary activation | all validation/generation | immutable package, resolver map, activation snapshot, compiled caches | IDR-023/024/028 | package validation first; atomic active-snapshot pointer plus outbox | strong activation; in-flight work keeps captured version | package digest/version and activation request key | lock package namespace; CAS active version; Serializable for dependency predicate | URI/digest collision, missing closure, incompatible active use | failed activation no change; replay returns activity result | activation/deprecation event and audit atomic | privileged, supply-chain and handling policy | concurrent activation, rollback, in-flight validation | 039-040, 045, 047-053 | Never mutate existing contract interpretation |
| Observation ingestion | DataStream/Observation | admission ledger, typed fact, exact payload optional, validation/provenance, latest/extents, outbox | CSAPI P2 R observation schema; IDR-022/027 | accepted item or explicitly documented atomic batch in one transaction | strong admission/fact; projections same transaction where correctness-critical | source/message ID and/or scoped key; contract-aware content duplicate | unique admission key; parent contract fingerprint; conditional projection update | duplicate, same key/different payload, missing parent, late/corrected/equal-time | at-least-once input; duplicate returns original; late is not duplicate | ingest outbox/provenance atomic; no automatic System Event | authorize stream; policy before projection/publication | duplicate storm, out-of-order, equal-time, partition races | 030-035, 039-043, 052-056 | Batch partial success contract deferred |
| Status sample/update | System/DataStream status | immutable sample/history, current projection, freshness inputs, outbox | IDR-020/027 | fact+conditional latest projection+inbox/outbox atomic | append strong; current per accepted order | source sequence/message identity | unique fact; compare `(domain time, authority rank, revision, stable tie)` | stale/late/conflicting source/equal time | late history accepted; stale does not replace current | status-data event/outbox; System Event only for separate occurrence | policy can change visible latest; projection scoped | reordering, correction, policy-change replay | 034-035, 039-043 | No arrival-order LWW |
| System Event creation/correction | System Event | immutable occurrence/revision, relations, provenance, outbox | CSAPI P2; IDR-020 | event identity/revision/evidence/outbox atomic | strong append and deterministic revision | source occurrence/message ID or client key | unique domain event ID/source tuple; revision CAS | duplicate occurrence, correction race, false CRUD-to-event mapping | replay exact event idempotent; correction explicit | domain event distinct from audit/outbox/message | sensitive event type/time/links | duplicate/correction/publication replay | 031, 034-035, 039-043 | Ordinary CRUD does not invent System Event |
| Command submission | ControlStream/Command | immutable intent, contract, authorization/safety, initial status, idempotency, dispatch outbox, provenance/audit | CSAPI P2 command schema; IDR-020/022 | acceptance/rejection decision state atomic; external dispatch after commit | strongest local consistency; no partial accepted command | mandatory scoped key or client command ID; exact intent fingerprint | unique key; lock target/reservation scope; Serializable for schedule predicate | duplicate intent, same key/different command, stale schema/feasibility, target conflict | never blind redispatch; resolve by command status/id | intent/status/dispatch outbox/audit correlation atomic | command authorization and safe diagnostics | simultaneous submit, lost response, dispatch ack loss | 036-041, 052-056 | Device exactly-once is unclaimable absent target contract |
| Feasibility evaluation | ControlStream/Feasibility | request, context fingerprint, status/result history, expiry, provenance/outbox | CSAPI P2 §10; IDR-020 | request acceptance atomic; evaluation async; result transaction separate | snapshot-consistent result with explicit staleness | request key; result source/attempt ID | revision/attempt CAS; optional target lock only if feasibility reserves | stale context, concurrent command, expired result | replay returns same evaluation; new context creates new attempt | state events/audit correlated | result may reveal constraints/other missions | stale-result submit and concurrent scheduling | 037-040 | Feasibility is not reservation unless explicit profile |
| CommandStatus/result/cancel | Command | append status/result, current projection, dispatch/compensation intent, audit/outbox | CSAPI P2 CRUD/update; cancellation note | one transition transaction per command; external cancel after commit | serialized state-machine consistency | source status ID/sequence; cancel request key | row lock command; expected transition/revision; stable tie | terminal-state race, duplicate status, cancel vs completion, contradictory target | duplicate report no-op; invalid transition retained/rejected per policy | transition/outbox/audit atomic | only authorized actor/source may transition | exhaustive state-machine race/model tests | 036-038, 041, 052-056 | Final state vocabulary deferred |
| Inbox consumption/source offset | ingest/broker/sync adapters | message envelope, inbox receipt, domain effect, source cursor | reliability evidence; IDR-025/027 | inbox unique insert + domain effect + durable processed marker/offset atomic in PG | effectively-once local effect | immutable `(source, partition, epoch, message/sequence)` | unique constraint/`ON CONFLICT`; partition-ordered worker if required | duplicate, offset gap/reset, same ID/different bytes | at-least-once receive; redelivery replays stored result | consume/provenance/audit as needed; output via outbox | authenticated source and anti-replay window | crash before/after commit/ack; rebalance; reset | 031, 035, 042-043, 048 | Broker offset commit external: ack only after PG commit |
| Outbox publication | all publishable changes | domain event, outbox, delivery attempts/receipts | IDR-014H/015/020/025 | outbox row atomic with source mutation; claim/publish/ack are separate short steps | at-least-once publication; per-aggregate order where required | immutable event/message ID consumed downstream | `FOR UPDATE SKIP LOCKED`, lease/attempt CAS, partition key | ack loss, duplicate publish, poison event, order gap | bounded retry/backoff; dead-letter/quarantine without losing row | publication attempts observable/auditable but not domain event | policy snapshot/envelope at publish and consume | worker crash every boundary; duplicates/order | 035, 041, 048, 052-056 | No global commit-order claim from sequence alone |
| Artifact catalog plus external object | documents/packages | staging bytes, digest, object, catalog/revision, provenance | IDR-028 | stage/hash/verify object outside; catalog authority commit; repair janitor | catalog strong, object readiness state explicit | digest/acquisition/workflow key | conditional object put + catalog unique keys/state CAS | orphan/missing/corrupt/different provenance | retry staged workflow; retrieval verifies digest | promotion/repair/audit events | policy/encryption domain may prevent dedup | object/catalog crash matrix and backend suite | 030-031, 044-049, 052-055 | No distributed transaction assumed |
| DDIL/federation synchronization | all replicated resources | manifests, inbox, branch revisions, conflicts, tombstones, policy/provenance | IDR-019/028; DDIL context | each received unit locally atomic; session not globally atomic | local strong, cross-node causal/eventual with explicit uncertainty | origin/message/revision/digest IDs and sync session | inbox unique; branch CAS; conflict records; per-origin ordering | divergent identity/state/schema/time/policy/command | replay safe; gaps retained; never blind LWW | sync/conflict/resolution/audit events | policy before transfer/merge/disclosure | partitions, fork/rejoin, duplicate/reordered manifests | 030, 039-043, 052-056 | Exact merge rules deferred |
| Policy-derived view/cache refresh | all query surfaces | policy projection, search/cache/index, generation manifest | IDR-028 | authoritative mutation commits invalidation/outbox; rebuild async | eventual bounded staleness unless response computes directly | build key including policy/resource/package/version | compare watermark/build key; atomic swap | stale cache, revoked access, partial rebuild | safe miss/rebuild; never serve wrong-policy stale entry | invalidation and privileged override audit | fail closed on uncertain policy freshness | revocation race/cache poisoning/leakage | 032-033, 039-040, 048, 052-055 | Strong read paths bypass stale cache |
| Audit of denied/failed attempt | all | security decision, correlation, minimized request metadata | IDR-019/020; later 041 | separate durable audit write after/beside rolled-back domain tx | durable according to audit policy; no domain partial state | request/correlation ID prevents duplicate amplification | append-only sink; independent failure policy | audit sink unavailable, duplicate attempt | bounded retry/spool; response policy defined | audit event only; never counterfeit domain success | minimize sensitive payload/existence | domain rollback plus audit outage/flood | 039-041, 044, 048, 052, 055 | Cannot be atomic with a transaction that rolls back |

---

## 6. Consistency Model Strategy

### 6.1 Consistency Classes

| Class | Use | External promise |
|---|---|---|
| Atomic authoritative | one resource aggregate, relationships/cascade, admission, command transition, package activation | success means all named authoritative effects committed; failure means none |
| Strong optimistic | mutable resource revision through `ETag`/`If-Match` | no silent lost update; stale client gets `412` |
| Snapshot-consistent | multi-query export, validation, feasibility, conformance run | every result names the captured database/package/policy snapshot |
| Read-your-writes | successful write response and immediate primary read | response and canonical `Location`/ETag reflect committed revision |
| Per-aggregate monotonic | command status, resource revisions, per-source streams | older state never replaces a newer accepted state; history remains accessible |
| Effectively-once local | inbox/idempotency plus domain effect | replay yields one committed local effect, not one delivery |
| At-least-once delivery | outbox to broker/peer and inbound adapters | duplicates possible; stable IDs and consumer dedup required |
| Causal/eventual | DDIL/federation and derived caches | origin/causal context, watermark and conflicts are visible; no global instant truth |
| Best-effort derived | previews, non-critical metrics | loss/rebuild allowed; never authority or safety input |

PostgreSQL commits expose all transaction changes together and hide rolled-back/intermediate changes.[^7] Read Committed gives a new statement snapshot and is sufficient when a single statement plus constraints determines correctness. Repeatable Read gives a stable transaction snapshot but does not by itself protect every multi-row business rule. Serializable emulates a serial order for committed transactions and aborts dangerous executions; Glaux must retry the whole decision using the same immutable request inputs. **[I/P]**

Read replicas, search indexes, object mirrors, and caches are not valid sources for mutation preconditions, safety authorization, idempotency lookup, or current command transition. If Glaux exposes replica reads, responses should identify a consistency class/watermark where clients could observe lag; security revocation and command state use primary/strong paths.

### 6.2 Ordering

Ordering is scoped: resource revision order, per-command transition order, per-source partition sequence, domain time order, and outbox delivery attempt order are different. PostgreSQL sequence values can have gaps and do not alone prove commit order. Global total ordering is not promised. Stable replay cursors must be designed to avoid permanently skipping a lower sequence whose transaction committed later; IDR-SRV-035 owns the final subscription cursor.

---

## 7. Transaction Boundary Findings

### 7.1 Canonical Mutation Unit

A successful mutation normally commits: idempotency/inbox admission; immutable input/artifact references; validation outcome summary; resource/fact and revision; typed relationships; lifecycle/current projection changes; provenance activity; domain/lifecycle event; outbox row; and an audit correlation/reference. Not every operation creates every record, but any required record is part of the same unit.

Generated representations, full validation reports, external object writes, notifications, broker sends, device calls, remote synchronization, analytics, and caches are outside. The transaction stores durable intent and enough state for recovery. No external call or unbounded parse occurs while database locks are held.

### 7.2 Isolation and Lock Selection

1. Encode invariant in constraint/index where possible.
2. Use one conditional `INSERT`/`UPDATE ... WHERE revision = expected RETURNING` or `INSERT ... ON CONFLICT` where possible. PostgreSQL guarantees an atomic insert-or-update outcome for `ON CONFLICT DO UPDATE` under high concurrency.[^8]
3. Lock existing aggregate rows with `FOR UPDATE` for command/status/lifecycle transitions.
4. Use transaction-level advisory locks only for stable application-defined keys when no row exists yet or a cross-table namespace must serialize; never use session locks in request code.
5. Use Serializable for predicate/cycle/schedule/cascade invariants that otherwise permit write skew; retry the complete unit.

Acquire multiple locks in canonical resource-type/UUID order and take the strongest needed mode first. PostgreSQL can deadlock even on row locks and recommends consistent lock order; long-lived transactions are explicitly hazardous.[^9] Configure bounded connection-acquire, statement, lock, idle-in-transaction, and overall request deadlines, with distinct public retry behavior.

### 7.3 Cascade and Large Work

Small normative cascades commit atomically. If an authorized cascade is too large for a bounded request transaction, create a durable administrative job before returning `202`; hide or mark the root pending only under an explicitly specified state model, and perform bounded resumable subtransactions with a final atomic visibility transition. Never expose a partly deleted tree as if the operation succeeded. Physical retention/destruction is separate from API retirement.

Savepoints may isolate optional internal work, but they do not justify public “partial success” unless the API contract explicitly defines per-item outcomes. SQLx models transactions/savepoints and rolls back an in-progress transaction on drop, but Glaux should explicitly commit or roll back and surface cancellation/ambiguous commit states rather than depending on destructor timing.[^10]

---

## 8. Idempotency and Duplicate-Detection Findings

### 8.1 Idempotency Record

Conceptually store: tenant/security domain, authenticated principal/client/source, operation and target/route scope, key, request media/profile, normalized semantic fingerprint plus exact-byte digest where material, creation/expiry, state (`in_progress`, `committed`, `rejected_final`, `retryable_failed`, `unknown/reconcile`), resulting IDs/revisions, HTTP status, safe replay headers/body reference, and correlation/provenance/audit IDs.

Scope is part of uniqueness. Keys are opaque and never reused as ResourceIds. Authentication and authorization occur before revealing an existing result. Same key with a different operation fingerprint returns `409` with a stable `idempotency-key-reuse` problem. Concurrent same-key attempts serialize on the unique record; one performs the work and the other returns the committed result or a bounded “in progress” response with `Retry-After`.

### 8.2 Which Operations Require Keys

- Require for command submission/cancellation and any action that may cause a physical effect.
- Require or source-map for broker/DDIL ingestion and synchronization.
- Strongly recommend/require by deployment profile for POST create, bulk import, asynchronous feasibility, and client-generated System Events.
- Optional for PUT/DELETE because HTTP intended effect is idempotent, but useful to replay exact response and resolve ambiguous commit.
- Recommend for PATCH unless the patch format/operation is independently proven idempotent.

Validation failures before a durable effect may be retried after correction with a new key. A deterministic failure may be stored under the original key to prevent command ambiguity; transient failures are not frozen forever. Retention must cover the maximum transport replay, client retry, offline synchronization, command lifetime/safety, and audit horizon. IDR-SRV-030 sets durations; key expiry must be externally documented because replay after expiry can create a new effect.

### 8.3 Duplicate versus Late/Corrected

Duplicate equivalence requires the same identity scope and compatible fingerprint, not merely equal timestamps or payload hashes. Identical Observation bytes from different sources are separate provenance. A late record is not a duplicate; a corrected record is a new revision/relation; a same message ID with different bytes is a conflict/security signal. Content deduplication never substitutes for operation idempotency.

---

## 9. Concurrency-Control and Conflict-Resolution Findings

### 9.1 HTTP Concurrency Contract

Strong ETags are generated from the selected canonical revision plus all material representation/profile/policy inputs. They are not database row versions exposed directly. Glaux requires `If-Match` for PUT/PATCH/DELETE on existing mutable resources and evaluates authorization/normal request checks before preconditions as required by HTTP ordering. `If-None-Match: *` may support create-if-absent on a client-selected target when the endpoint contract allows it.[^2]

Use:

- `412` when a supplied HTTP precondition is false;
- `428` when a required precondition is absent;
- `409` when current state semantically conflicts despite a satisfied precondition—identity collision, non-empty stream schema change/delete, relationship/cardinality/cycle, idempotency key reuse, or unresolved authority conflict;
- CSAPI-required `400` for Observation/Command data invalid against its parent contract.

### 9.2 Conflict Policy

| Conflict | Default action |
|---|---|
| Same scoped key and same intent | replay original result/no new effect |
| Same key/message ID and different intent/bytes | reject `409`, security/audit signal |
| Stale ETag/revision | reject `412`, return current safe ETag/link if authorized |
| UID/source identity collision | reject or quarantine; never auto-merge based only on value/hash |
| Relationship cycle/cardinality violation | atomic reject; explicit problem pointers |
| DataStream/ControlStream contract change with children | reject `409` as CSAPI requires |
| Late/out-of-order append fact | accept history if valid; conditional current projection |
| Same-time/source disagreement | retain conflict/candidates; apply named source policy, not arrival order |
| Command terminal/cancel race | serialize; one valid transition wins, other becomes explicit stale/invalid result |
| DDIL divergent authority/schema/policy | branch/conflict record; deterministic policy/operator resolution |

Last-write-wins is allowed only for explicitly designated, low-risk, single-authority scalar configuration where the write still has a revision, actor, and audit history. It is prohibited for identity, relationships, contracts, temporal validity, provenance/trust, command/control, policy/releasability, and cross-authority synchronization.

---

## 10. Ingestion, Replay, and Source-Offset Findings

Inbound envelopes should carry source/adapter ID, partition/channel, source epoch/session, sequence/offset, message ID, batch/item ID, source and receipt clocks, payload digest/media, contract/package fingerprint, authentication/policy context, and trace correlation. Not every protocol supplies every field; the adapter documents its equivalence and gap behavior.

The inbox/admission ledger unique key and domain effect commit together. For an external broker whose offset acknowledgement cannot join the PostgreSQL transaction, acknowledge only after commit. A crash after commit but before acknowledgement causes redelivery, which the inbox resolves to the stored result. A crash before commit leaves no effect and permits retry. Advancing the durable local source cursor without the domain effect is prohibited.

Offsets are source progress, not domain time or resource identity. Partition reassignment, epoch reset, offset gaps, truncation, and reused sequence numbers are explicit states. An adapter must not infer loss-free continuity from a larger number alone. Backfill/replay identifies its run and reason; it reuses original source identities and adds ingestion activity rather than minting duplicate domain facts.

For batches, Glaux must advertise one of: atomic entire batch; independent item transactions with per-item results; or durable asynchronous job with resumable items. It must never return success while silently dropping failed items. Exact limits, wire format, and whether CSAPI endpoints support batches belong to IDR-SRV-031/034.

Latest/extents/current projections update by a conditional comparison against accepted semantic order and source policy in the same transaction where strong freshness is required. Expensive aggregates and caches update through outbox-driven recomputation with watermarks; they expose staleness rather than masquerading as current authority.

---

## 11. Status/Event, Outbox/Inbox, and Durable-Publication Findings

Domain state, domain/lifecycle event, outbox record, broker message, delivery attempt, and consumer inbox record have separate immutable IDs. A System Event is created only for an actual modeled system occurrence; ordinary CRUD, ingestion, publication, retry, or conflict does not invent one.

### 11.1 Outbox State Machine

The source mutation inserts an outbox item with event ID, aggregate/revision, event type, policy/profile, payload build inputs or immutable payload digest, destination class/partition key, availability time, and correlation. Workers claim committed pending rows in small batches using row locks/`SKIP LOCKED`, mark a bounded lease/attempt, publish outside the claim transaction, then record acknowledgement or retry/dead-letter state in another transaction. `SKIP LOCKED` is used only for work claiming, never to answer authoritative client queries.[^5]

If the worker crashes after publication but before acknowledgement, publication repeats with the same message ID. Consumers therefore use inbox uniqueness and commit acknowledgement only after local effect. Poison events remain retained with safe diagnostics and operator workflow; they are not skipped by advancing a global cursor irreversibly.

Per-aggregate event order is preserved by aggregate revision/partition routing and consumer checks. Cross-aggregate global order is not promised. A notification/listener may wake workers but is not the durable queue; the outbox table is recoverable truth. Broker adoption and Part 3 messages remain IDR-SRV-035 decisions.

### 11.2 Delivery Terminology

- **At-most-once:** acceptable only for non-authoritative telemetry/metrics explicitly designated lossy.
- **At-least-once:** default transport contract for authoritative messages.
- **Effectively-once local effect:** inbox uniqueness plus domain commit.
- **Exactly-once broker processing:** may exist inside one vendor's transactional boundary but is not a Glaux end-to-end claim.
- **Exactly-once physical action:** unclaimable without a target protocol and device behavior that supplies durable operation identity/result semantics.

---

## 12. Command, Feasibility, and Command-Status Transaction Findings

### 12.1 Command Acceptance and Dispatch

Command acceptance captures an immutable target, parameters, SWE contract fingerprint, requested execution window, requester, authority/policy/safety decision, idempotency fingerprint, feasibility reference if any, and current target context. The transaction creates the Command, initial status/evidence, dispatch intent/outbox, provenance, and audit correlation. Only after commit may a dispatcher call the target.

Dispatch attempt, target acknowledgement, execution status, result, cancellation intent, compensation attempt, timeout assessment, and operator override are separate append events. The current Command projection derives under a later explicit state machine. A database rollback cannot retract a physical action, so external dispatch is never made before durable intent. If outcome is uncertain, Glaux records `unknown/pending reconciliation` under the later vocabulary rather than retrying blindly.

### 12.2 Idempotency and Serialization

Command idempotency scope includes security domain, authenticated requester/client, target/ControlStream, operation, and key. Same key/same canonical command returns the original Command and status. Different parameters, target, contract, execution window, or materially changed policy context conflict. A new operator intent uses a new key even if parameters are equal.

Transitions serialize on the Command row or a stable command lock and validate expected current revision, allowed predecessor, source authority, source sequence/time, terminality, and policy. Multiple target resources or scheduling reservations acquire stable ordered locks or use a Serializable schedule predicate. Dispatchers partition by target when target protocols require strict order.

### 12.3 Feasibility

A feasibility result binds request digest, target/contract, resource and policy snapshot, evaluated constraints, evaluator/version, evaluation time, expiry/validity, and provenance. It is advisory unless a profile explicitly creates a reservation. Command submission using a stale result must re-evaluate or reject; it never assumes feasibility guarantees capacity under concurrency. IDR-SRV-037 defines the asynchronous lifecycle and IDR-SRV-038 defines authorization/safety/audit rules.

Cancellation is a posted `CANCELED` status under CSAPI, not Command deletion. A cancel/completion race is serialized: the committed first valid transition determines whether cancellation was accepted, too late, or requires compensation; both attempts remain auditable.

---

## 13. DDIL, Federation, Synchronization, and Last-Known-State Findings

Disconnected nodes maintain independent local transaction histories. Synchronization transfers immutable revisions/events/manifests with origin node, authority, branch/base revision, source/domain/transaction clocks, digest, package/contract fingerprint, policy marking, tombstone, and causal dependencies. Receiving uses an inbox and locally atomic apply/conflict transaction.

Automatic resolution is limited to proven cases: exact replay; same immutable fact/digest under compatible authority/policy; commutative set addition with no cardinality conflict; or a profile-defined single-authority monotonic source sequence. Divergent identity, parentage, contract, valid-time intervals, source authority, command state, deletion versus update, or releasability creates an explicit conflict. Resolution is a new provenance-bearing activity and never rewrites either branch.

Last-known-state stores the selected evidence, source/domain time, local receipt/commit, source reachability, freshness policy/version, synchronization watermark/gaps, and unresolved conflicts. After delayed synchronization, older domain evidence may enrich history without replacing newer current state. A conflict or policy-suppressed newest fact can make visible state `unknown` rather than silently selecting arrival order.

Offline idempotency and inbox records must survive longer than the maximum reconnect/replay horizon. Tombstones must survive long enough to prevent deleted resources from being resurrected by stale peers. Exact durations and vector/causal representation belong to IDR-SRV-030/042/043.

---

## 14. Error, Retry, Stale-State, and Problem-Detail Behavior Findings

| Condition | HTTP behavior | Retry guidance |
|---|---|---|
| malformed request/media/query | `400` or applicable standard code; stable validation problem | correct request; not automatic |
| unauthenticated/unauthorized/concealed | `401`/`403` or policy-selected `404` | reauthenticate/obtain authority; never infer existence |
| required `If-Match` absent | `428` with safe instruction | fetch authorized current representation/ETag, then retry |
| supplied precondition false | `412` | reconcile with current representation; do not blind retry |
| semantic/current-state conflict | `409` with stable conflict type | change intent or invoke governed resolution |
| CSAPI Observation/Command violates parent schema | `400` as required | correct payload/contract |
| accepted asynchronous work | `202`, durable status `Location`, operation ID, optional `Retry-After` | poll/subscribe using operation ID; POST retry uses same key |
| rate/lock/backpressure limit | `429` or `503` with `Retry-After` when safe | bounded exponential backoff plus jitter and same key |
| transaction serialization/deadlock | normally retried internally within budget; otherwise `503`/stable retryable problem | retry whole request with same key/precondition after backoff |
| response lost/commit uncertain | status/result lookup via idempotency key or canonical operation ID | never create a new command/key merely because response was lost |

RFC 9457 problem objects use a stable type URI as the primary problem identifier and permit extensions; problem details are not debugging output and security considerations require careful disclosure control.[^11] Glaux problem extensions may include safe `correlationId`, `retryable`, `retryAfter`, expected/current safe ETag, idempotency status, conflict class, and validation finding codes. They must not expose hidden resource existence, policy rules, competing principals, command affordances, payloads, SQL/locks, schema internals, object keys, or stack traces.

Retry middleware captures immutable request inputs, authorization/policy snapshot rules, current precondition, and deterministic operation context. It rolls back and re-executes the **entire transaction** on allowed SQLSTATEs within attempt/time budgets. It never repeats after an external side effect, never changes idempotency key, and never retries `412`, semantic `409`, authorization, validation, or arbitrary uniqueness errors. Metrics distinguish contention, deadlock, serialization, timeout, pool starvation, and external delivery failures.

---

## 15. Security, Policy, Provenance, and Audit Implications

Authorization and policy evaluation must be transactionally bound to the write context. When policy depends on mutable database facts, read/lock or serialize those facts in the same unit; record policy version/decision reference. A later revocation invalidates derived views and future actions but does not falsify the historical decision.

Idempotency keys, ETags, conflict details, current revisions, lock timing, queue depth, offset gaps, and duplicate responses can reveal resource existence or activity. Scope keys by security domain/principal, compare fingerprints without returning protected content, rate-limit key probing, and use constant/minimized conflict surfaces where policy requires concealment. Do not accept caller-provided actor, authority, audit, or source-trust fields as authoritative.

Successful domain change, provenance, lifecycle/domain event, outbox, and audit correlation should commit together. A denied request or a transaction that rolls back cannot write its audit record inside that rolled-back unit; it needs a separate durable security-audit path, bounded local spool, or independently committed post-failure record. The public response policy when the audit sink is unavailable is safety-class dependent and deferred to IDR-SRV-041.

Database roles enforce least privilege between API transactions, migration/admin, outbox workers, ingestion adapters, audit writers/readers, and repair jobs. Outbox consumers reauthorize or consume a policy-bound envelope as defined downstream; publication does not bypass policy merely because the originating transaction was authorized. Tamper evidence, audit retention, cryptographic chaining/signing, and cross-boundary export remain IDR-SRV-041 decisions.

---

## 16. Rust Implementation, Test, Fixture, Conformance, Performance, and Interoperability Implications

### 16.1 Rust Architecture

Use typed application commands and a unit-of-work service boundary. HTTP handlers authenticate, parse/bound, and create an immutable command; application services open the transaction and call repositories with an explicit transaction handle; domain policy returns typed outcomes/events; the service writes all authoritative effects and commits; adapters publish or dispatch afterward. Repository methods must not silently open nested independent transactions or perform network I/O.

SQLx 0.9.0 `Pool::begin` obtains a pooled connection and starts a transaction; `Transaction` supports commit/rollback/savepoints and rolls back on drop.[^10] Implementation should classify SQLSTATEs, preserve request cancellation semantics, instrument acquire/transaction/lock times, and avoid holding `Transaction` across uncontrolled `.await` points. Background workers use bounded pools/concurrency so outbox pressure cannot starve command/API transactions.

### 16.2 Required Verification Matrix

| Lane | Required cases |
|---|---|
| Unit/property/model | fingerprint stability, key scope, state transitions, conflict taxonomy, ordering/ties, backoff budgets, policy-safe problems |
| Repository integration | constraints, FK/exclusion, revision CAS, `If-Match` mapping, Serializable write skew, row/advisory locks, deadlocks, transaction rollback |
| Idempotency/inbox | simultaneous same key, same key/different body, lost response, expiry boundary, replay after crash, offset reset/gap, duplicate storms |
| Outbox | crash before/after commit/claim/publish/ack, duplicate publish, poison event, lease expiry, per-aggregate ordering, worker competition |
| Command | double submit, target dispatch ambiguity, cancel/complete/timeout races, stale feasibility, invalid transition, target reorder/disconnect |
| DDIL | branch/fork/rejoin, delayed/reordered/duplicate messages, tombstone resurrection, authority/policy/contract conflicts, partial package |
| Security | unauthorized duplicate probe, hidden ETag/conflict, policy revocation/cache race, audit outage, key flooding, cross-tenant collision |
| Conformance/interoperability | CSAPI CRUD/schema/cascade conflicts, 400/409 behavior, ETag/412/428 contract, RFC 9457, client retry/idempotency |
| Performance/stress | hot aggregate/key, Observation throughput, lock waits/deadlocks/retries, long readers, pool starvation, outbox lag/recovery |

Use deterministic barriers/failpoints rather than timing sleeps to force interleavings. Model-based and property tests should enumerate command and lifecycle transitions. Integration tests need a real PostgreSQL 18 instance; SQLite cannot establish PostgreSQL isolation/locking parity. Fault injection must terminate processes/connections at every transaction/external-effect boundary and then assert authority, inbox/outbox, projections, and audit correlations.

Performance gates measure p50/p95/p99 transaction/lock/pool time, serialization/deadlock/retry rate, hot-key throughput, outbox age/redelivery, inbox duplicate rate, source lag/gaps, command-dispatch latency, WAL/replica lag, and recovery time. Tuning may alter indexes, batching, partitioning, or isolation per operation only while invariants and externally advertised behavior remain unchanged.

Implementation-study observations from OSH, Connected Systems Go, pygeoapi, SECD, smoke tests, and OS4CSAPI discussions remain informative. They justify robust persistent identifiers, deterministic paging/status, replay tests, and transaction/outbox seams, but no implementation's implicit overwrite, in-memory state, broker QoS, or error behavior becomes Glaux normative behavior.

---

## 17. Downstream Topic Handoff Matrix

| Topic(s) | Required handoff | Acceptance/evidence gate |
|---|---|---|
| IDR-SRV-030 | idempotency/inbox/tombstone/outbox/conflict retention; cascade jobs; audit/command holds | Replay and resurrection horizons precede deletion schedules |
| IDR-SRV-031 | unit-of-work writes, batch contract, strict/quarantine import, source offsets, response/status lookup | Crash matrix proves no partial authority or lost offset |
| IDR-SRV-032/033 | ETag/If-Match/428/412/409 contract, async 202/status, cache/replica consistency | OAS/runtime/client parity for every precondition/problem |
| IDR-SRV-034 | item/batch idempotency, contract-bound admission, late/correction/latest projection | Duplicate/out-of-order/equal-time workload and conformance tests |
| IDR-SRV-035 | outbox envelope, broker semantics, replay cursor, per-aggregate order, inbox consumer contract | No end-to-end exactly-once claim; duplicate/ack-loss tests |
| IDR-SRV-036-038 | command state machine, feasibility freshness/reservation, dispatch/cancel ambiguity, safety/audit | Exhaustive transition/race/fault model before device integration |
| IDR-SRV-039-041 | transactional policy snapshot, hidden conflicts/ETags/keys, audit success/failure paths | Security test covers duplicate probes, rollback and audit outage |
| IDR-SRV-042/043 | local-first branches, manifests/inbox, tombstones, causal gaps, conflict resolution | No LWW for protected classes; fork/rejoin corpus passes |
| IDR-SRV-044-046 | PG topology, pool/isolation settings, migrations, backup/recovery and replica-read claims | Failure/recovery drills preserve inbox/outbox/idempotency state |
| IDR-SRV-047-049 | retry/timeout/pool/outbox settings, secrets, observability and health | Unsafe setting combinations rejected and never leaked |
| IDR-SRV-050/051 | CRUD/schema/cascade and HTTP conditional/error traceability | Every normative/project guarantee has executable requirement ID |
| IDR-SRV-052/053 | deterministic concurrency harness, failpoints, operation/conflict fixtures | All Section 16 lanes and matrix rows represented |
| IDR-SRV-054 | contention, throughput, retry, pool and outbox stress gates | Tuning cannot weaken atomicity/idempotency/safety |
| IDR-SRV-055/056 | authorization/command race attacks and external client retry interoperability | Verify safe diagnostics and exact retry behavior with pinned clients |

---

## 18. Recommendations

1. **Adopt one PostgreSQL unit of work for every authoritative mutation. [Critical]** Include required revision/fact, relationships, admission/idempotency, validation/provenance, projection, event/outbox, and audit correlation.
2. **Use Read Committed plus constraints/CAS by default and targeted Serializable or row locks by invariant. [High]** Do not choose one isolation level by slogan; document and test each escalation.
3. **Require strong `If-Match` for mutable PUT/PATCH/DELETE. [High]** Return `428` when missing and `412` when false; keep representation ETag separate from domain revision.
4. **Implement a first-class scoped idempotency service. [Critical]** Require keys for commands and replay-prone adapters; bind key to intent fingerprint and replay the original result.
5. **Guarantee effectively-once local effects, not end-to-end exactly once. [Critical]** Use inbox uniqueness and atomic domain commit; document at-least-once delivery and duplicate handling.
6. **Write domain events and outbox entries atomically with authority changes. [Critical]** Publish after commit with stable message IDs, leases, retries, acknowledgements, and consumer inboxes.
7. **Never perform external I/O while holding an authority transaction. [Critical]** Represent object finalization, publication, dispatch, synchronization, and compensation as durable state machines.
8. **Advance source offsets only with the accepted inbox/domain effect. [High]** Treat gaps, epochs, late records, corrections, and replay runs explicitly.
9. **Serialize Command transitions and make dispatch ambiguity visible. [Critical]** Never blind-retry a physical action; same intent/key returns the original Command.
10. **Use constraints and deterministic lock order before broad locks. [High]** Bound transaction/lock/pool time and retry the complete unit on classified failures.
11. **Retain explicit conflicts rather than arrival-order last-write-wins. [High]** Protect identity, relations, contracts, time, provenance, command, policy, and DDIL branches.
12. **Use RFC 9457 stable, policy-safe transaction problems. [High]** Separate 400 validation, 409 semantic conflict, 412 false condition, 428 missing condition, and retryable 429/503.
13. **Keep failed-attempt audit outside rolled-back domain transactions. [High]** Define an independent durable/spooled path and safety-class outage policy.
14. **Test interleavings with deterministic barriers and failpoints. [Critical]** Cover every crash boundary, duplicate, race, fork/rejoin, and hidden-data conflict.
15. **Keep broker, Part 3, DDL, retention, and command state-machine choices downstream. [Medium]** This baseline constrains them without preselecting implementation details.

---

## 19. Risks, Constraints, and Open Questions

### 19.1 Risks

| Risk | Consequence | Mitigation/owner |
|---|---|---|
| “Exactly once” overclaim | duplicate/lost effects and unsafe command retries | precise guarantees, inbox/outbox, 035/036 tests |
| Idempotency key not bound to intent/principal | replay confusion or cross-user data leak | scoped unique record, fingerprint and auth-first lookup |
| Long transactions/external awaits | pool exhaustion, lock contention, deadlocks | short unit, durable external state machines, 044/054 |
| Blanket Serializable or coarse locks | avoidable aborts and throughput collapse | per-invariant selection and benchmarks |
| Read Committed write skew | broken graph/schedule/policy invariant | constraint/locks/targeted Serializable |
| Sequence mistaken for commit/global order | skipped or reordered publication | per-aggregate order and gap-safe replay cursor |
| Outbox ack loss | duplicate publication | stable message ID and consumer inbox |
| Offset advanced separately | silent data loss | atomic inbox/effect/offset; ack after commit |
| Command redispatch after ambiguous response | duplicated physical effect | mandatory key, durable dispatch state, reconciliation |
| Hidden conflict/ETag leakage | existence and operational disclosure | policy-safe responses/key scoping/rate limits |
| Failed audit shares rollback | missing denial/failure evidence | independent durable audit path |
| DDIL last-write-wins | lost authority/provenance or resurrection | branch/conflict/tombstone/causal records |

### 19.2 Open Questions and Owners

- Which POST/PATCH endpoints mandate the project idempotency header and what is its final name/schema? **Owners:** 031-033/045/050.
- What are idempotency, inbox, outbox, tombstone, conflict, and command-safety retention windows? **Owner:** 030.
- Which graph/cascade/schedule predicates need Serializable versus explicit/advisory locks after benchmark? **Owners:** 031/036/044/054.
- Are large cascades synchronous atomic requests or durable jobs, and what pending visibility is legal? **Owners:** 030-033.
- What batch atomic/partial contract is exposed for observations and imports? **Owners:** 031/034/050.
- What exact Command states, transition authority, target correlation, timeout, retry and compensation rules apply? **Owners:** 036-038.
- Which feasibility profiles create a reservation rather than an advisory result? **Owner:** 037/profile authority.
- What broker and Part 3 replay cursor can provide gap-safe per-subscription catch-up? **Owner:** 035.
- Which DDIL causal structure and automatic merge classes are approved? **Owners:** 042/043.
- Does audit-sink failure fail closed for command/admin writes and fail open with spool for lower-risk telemetry? **Owners:** 038-041.
- What retry attempt/time budgets, lock timeouts, pool partitions, and outbox worker concurrency meet workload goals? **Owners:** 044/047/048/054.

---

## 20. Validation Against This Plan's Success Criteria

| Topic Plan Success Criterion | Validation Status | Evidence |
|---|---|---|
| Operation families are identified with source anchors | Met | Section 5 required-field matrix |
| Transaction boundaries and consistency expectations are mapped by operation/resource family | Met | Sections 5-7 |
| Idempotency, duplicates, offsets, replay, conditions, ETags/revisions, locks, and conflicts are evaluated | Met | Sections 8-10 |
| Ingestion, status/event, command, DDIL, validation/document, and audit workflows are analyzed | Met | Sections 5 and 10-15 |
| Error, retry, conflict, and problem-detail behavior is documented | Met | Section 14 |
| Rust, test, fixture, conformance, performance, and interoperability implications are documented | Met | Section 16 |
| Implementation/community evidence remains non-normative | Met | Sections 3.2-3.4 and 16 |
| Recommendations are decision-usable and bounded | Met | Sections 2.2 and 18 |
| Downstream handoffs are explicit | Met | Section 17 and Section 19 owners |
| References are explicit and reproducible | Met | Header pins, footnotes, and Section 21 |

The research is complete and **In Review**. It is not accepted for downstream use until the Glaux Project Lead records acceptance in this report, its topic plan, and the overall plan.

---

## 21. References

### 21.1 Standards and Project Sources

- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html), June 2022
- [RFC 6585: Additional HTTP Status Codes](https://www.rfc-editor.org/rfc/rfc6585.html), April 2012
- [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html), July 2023
- [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html)
- [Official CSAPI approved `v1.0.0` source pin](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [OGC API - Features - Part 1: Core corrigendum](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC SensorML Encoding Standard 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model Encoding Standard 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-029 Research Plan](../IDR%20Plans/idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy.md)
- [OGC API - Connected Systems Upstream-History Register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)
- Accepted [IDR-SRV-015](idr-srv-015-canonical-glaux-server-resource-model-report.md), [IDR-SRV-016](idr-srv-016-identifier-uri-and-resource-lifecycle-strategy-report.md), [IDR-SRV-017](idr-srv-017-relationship-and-linkage-model-report.md), [IDR-SRV-018](idr-srv-018-temporal-validity-and-freshness-model-report.md), [IDR-SRV-019](idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md), and [IDR-SRV-020](idr-srv-020-status-availability-and-system-event-model-report.md)
- Accepted [IDR-SRV-021](idr-srv-021-sensorml-representation-strategy-report.md), [IDR-SRV-022](idr-srv-022-swe-common-data-component-strategy-report.md), [IDR-SRV-023](idr-srv-023-schema-and-encoding-validation-strategy-report.md), and [IDR-SRV-024](idr-srv-024-units-observed-properties-and-semantic-binding-strategy-report.md)
- Accepted [IDR-SRV-025](idr-srv-025-database-and-persistence-architecture-options-report.md), [IDR-SRV-026](idr-srv-026-geospatial-storage-and-query-strategy-report.md), [IDR-SRV-027](idr-srv-027-time-series-observation-storage-strategy-report.md), and [IDR-SRV-028](idr-srv-028-metadata-and-document-storage-strategy-report.md)
- Controlled project source: `AC/224(JCGISR)D(2026)0005`, April 27, 2026, SHA-256 `56dc757b6e677b3584e3152a957849f21a24b22854f562613ff283a8b599da8c` (not redistributed)

### 21.2 Primary Technology Sources

- [PostgreSQL 18: Transactions](https://www.postgresql.org/docs/18/tutorial-transactions.html)
- [PostgreSQL 18: Transaction Isolation](https://www.postgresql.org/docs/18/transaction-iso.html)
- [PostgreSQL 18: Explicit Locking](https://www.postgresql.org/docs/18/explicit-locking.html)
- [PostgreSQL 18: Application-Level Consistency](https://www.postgresql.org/docs/18/applevel-consistency.html)
- [PostgreSQL 18: Serialization Failure Handling](https://www.postgresql.org/docs/18/mvcc-serialization-failure-handling.html)
- [PostgreSQL 18: `INSERT ... ON CONFLICT`](https://www.postgresql.org/docs/18/sql-insert.html)
- [PostgreSQL 18: `SELECT` locking and `SKIP LOCKED`](https://www.postgresql.org/docs/18/sql-select.html)
- [PostgreSQL 18: Advisory Lock Functions](https://www.postgresql.org/docs/18/functions-admin.html#FUNCTIONS-ADVISORY-LOCKS)
- [SQLx 0.9.0 `Transaction`](https://docs.rs/sqlx/0.9.0/sqlx/struct.Transaction.html)
- [SQLx 0.9.0 `Pool`](https://docs.rs/sqlx/0.9.0/sqlx/struct.Pool.html)
- [Apache Kafka 4.2 documentation](https://kafka.apache.org/documentation/)
- [Apache Kafka design: message delivery semantics](https://kafka.apache.org/42/design/design/)
- [NATS JetStream consumers](https://docs.nats.io/nats-concepts/jetstream/consumers)
- [NATS JetStream model deep dive](https://docs.nats.io/using-nats/developer/develop_jetstream/model_deep_dive)

### 21.3 Footnotes

[^1]: PostgreSQL Global Development Group, [Transaction Isolation](https://www.postgresql.org/docs/18/transaction-iso.html) and [Application-Level Consistency](https://www.postgresql.org/docs/18/applevel-consistency.html), PostgreSQL 18.6, checked September 14, 2026.
[^2]: IETF, [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html), §§9.2.2, 13.1-13.2 and 15.5.13, June 2022.
[^3]: IETF, [RFC 6585](https://www.rfc-editor.org/rfc/rfc6585.html), §3, April 2012.
[^4]: Apache Kafka, [Design: Message Delivery Semantics](https://kafka.apache.org/42/design/design/), Kafka 4.2 documentation, checked September 14, 2026.
[^5]: PostgreSQL Global Development Group, [`SELECT` locking clause](https://www.postgresql.org/docs/18/sql-select.html), PostgreSQL 18.6, checked September 14, 2026.
[^6]: PostgreSQL Global Development Group, [Serialization Failure Handling](https://www.postgresql.org/docs/18/mvcc-serialization-failure-handling.html), PostgreSQL 18.6, checked September 14, 2026. `40001` identifies serialization failure and `40P01` deadlock; uniqueness/exclusion retry depends on application semantics.
[^7]: PostgreSQL Global Development Group, [Transactions](https://www.postgresql.org/docs/18/tutorial-transactions.html), PostgreSQL 18.6, checked September 14, 2026.
[^8]: PostgreSQL Global Development Group, [`INSERT`](https://www.postgresql.org/docs/18/sql-insert.html), PostgreSQL 18.6, checked September 14, 2026.
[^9]: PostgreSQL Global Development Group, [Explicit Locking](https://www.postgresql.org/docs/18/explicit-locking.html), PostgreSQL 18.6, checked September 14, 2026.
[^10]: SQLx maintainers, [`Transaction`](https://docs.rs/sqlx/0.9.0/sqlx/struct.Transaction.html) and [`Pool`](https://docs.rs/sqlx/0.9.0/sqlx/struct.Pool.html), SQLx 0.9.0, checked September 14, 2026.
[^11]: IETF, [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html), §§3-5, July 2023.

---

## Report Completion Checklist

- [x] Topic ID matches overall research plan index
- [x] Topic research plan is linked and aligned
- [x] Core research questions are covered or explicitly unresolved
- [x] Findings are evidence-backed with reproducible references
- [x] Normative and informative evidence are classified and not conflated
- [x] Mutable sources identify a version, release, tag, commit, or dated retrieval
- [x] Controlled, inaccessible, missing, or ambiguous evidence limitations are explicit
- [x] Source-backed findings, analyst inference, and project recommendations are distinguishable
- [x] Conflicts with accepted prior reports are reconciled or explicitly escalated
- [x] Executive summary is independently readable
- [x] Recommendations are explicit and actionable
- [x] Risks and open questions are documented
- [x] Success criteria validation is complete
- [x] Plan-owner acceptance and acceptance date are recorded
- [x] Next steps and owners are identified

---

**Acceptance record:** Accepted by the Glaux Project Lead on September 14, 2026. IDR-SRV-030 was authorized as the next bounded single-topic iteration; no later topic, draft Part 3 implementation, or server implementation was authorized.

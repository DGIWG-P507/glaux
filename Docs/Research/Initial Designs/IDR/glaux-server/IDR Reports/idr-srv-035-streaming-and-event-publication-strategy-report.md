# Section 035: Streaming and Event Publication Strategy - Research Report

**Topic ID:** IDR-SRV-035<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-035 Streaming and Event Publication Strategy](../IDR%20Plans/idr-srv-035-streaming-and-event-publication-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 6 core questions and detailed questions concerning taxonomy, triggers, durability, ordering, replay, protocols, subscription policy, DDIL, tasking, testing, interoperability, and draft Part 3 disposition<br>
**Methodology Used:** Authority-ranked standards and accepted-baseline synthesis; immutable repository/release pins; bounded upstream-delta review; protocol and Rust feasibility comparison; failure-mode, policy, replay, and interoperability analysis; decision matrices and traceability validation<br>
**Research Time:** Approximately 19 hours of AI-assisted execution on September 15, 2026<br>
**Official Part 3 Baseline:** `part3-working-draft` at [`6f529a15bfa63259febc3620378d3e5a06305333`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/6f529a15bfa63259febc3620378d3e5a06305333), retrieved September 15, 2026<br>
**CS-Go Baseline:** release `v1.0.4` at [`244f4dd586da685d4d9b75e43f73001028b5bd0e`](https://github.com/SomethingCreativeStudios/connected-systems-go/commit/244f4dd586da685d4d9b75e43f73001028b5bd0e); `main` checked at `b1fd2e0e9bd69e222d05258d659a842ca24502cb`<br>
**OpenSensorHub Baseline:** open draft `osh-addons` PR [#194](https://github.com/opensensorhub/osh-addons/pull/194), head [`50774ec7e9c98f6ab8da827171e5c5abb9923a49`](https://github.com/opensensorhub/osh-addons/commit/50774ec7e9c98f6ab8da827171e5c5abb9923a49)<br>
**Primary Sources:** OGC 23-001 and 23-002 Version 1.0; draft OGC 23-003 at the exact pin above; draft OGC 25-030 Version 1.0; CloudEvents 1.0.2; MQTT 5.0; AsyncAPI 3.0.0; WHATWG SSE; RFC 6455<br>
**Supporting Resources:** Accepted IDR-SRV-014H, IDR-SRV-025, IDR-SRV-027, IDR-SRV-029, IDR-SRV-031 through IDR-SRV-034, and upstream-history register Version 1.12<br>
**Document Purpose:** Establish the server-side durable streaming/publication architecture, protocol sequence, subscription/replay contract, and final draft Part 3 experimental-profile decision without implementing the server or claiming unapproved conformance<br>
**Author:** OpenAI Codex<br>
**Accepted By:** TBD pending Glaux Project Lead review<br>
**Acceptance Date:** TBD pending acceptance<br>
**Date:** September 15, 2026<br>
**Last Updated:** September 15, 2026

---

## Reading Guide

| Label | Meaning |
|---|---|
| N | Requirement or rule from an approved normative source |
| D | Exact pre-publication OGC draft text or draft artifact |
| A | Accepted Glaux project decision |
| I | Version-pinned implementation evidence |
| X | Analyst inference or comparison |
| P | Recommendation or planning decision proposed by this report |

Precedence is **approved normative source, accepted Glaux baseline, exact draft for compatibility analysis, version-pinned implementation evidence, then inference and project recommendation**. “Part 3” below means the exact draft snapshot unless “approved” is stated explicitly.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority
4. Research Methodology
5. Event and Record Taxonomy
6. Publication Triggers and Eligibility
7. Durable Event Log, Outbox, Inbox, and Delivery Model
8. Ordering, Cursors, Replay, Backfill, Retention, and Deduplication
9. Payload, Envelope, and Content Strategy
10. Publication Protocol Evaluation and Selection
11. Topics, Channels, Subscriptions, Filters, Delivery, and Backpressure
12. Security, Authorization, Releasability, and Audit Boundaries
13. DDIL, Reconnect, Snapshot, Catch-up, and Federation
14. Command, Control, Feasibility, Status, and Health Publication
15. Observability, Fixtures, Conformance, Performance, and Interoperability
16. Draft Part 3 Decision, Profile, Compatibility, and Migration
17. Downstream Handoff
18. Recommendations and Implementation Sequence
19. Risks, Constraints, and Open Questions
20. Validation Against Plan Success Criteria
21. References and Sources
Appendix A. Publication Decision Matrix
Appendix B. Experimental Profile Contract
Appendix C. Reproducible Refresh and Validation Record

---

## 1. Executive Summary

Glaux should build one **durable, transport-neutral publication core** and expose it through multiple adapters. A successful domain mutation commits its authoritative record, immutable revision/effect, policy/provenance evidence, and outbox event in one PostgreSQL transaction. Publication workers then append or project that event into a replay-retained change log and deliver it at least once. Protocol acknowledgements update delivery evidence but never redefine whether the domain mutation committed. No supported transport provides end-to-end exactly-once delivery; consumers must be idempotent and use stable event identity, resource revision, and opaque resume cursors. **[A/P]**

The first no-broker live interface should be **authenticated HTTP Server-Sent Events (SSE)** over the same authorized change-feed/replay service. It is server-to-client only, fits browser and command-line clients, inherits the HTTP security boundary, supports named event IDs and reconnect through `Last-Event-ID`, and keeps the first streaming slice operationally small.[^9] Ordinary CSAPI HTTP query remains the authoritative state and polling fallback. SSE transport reconnect is not sufficient recovery: the event ID is an opaque Glaux cursor, and an expired cursor requires a new policy-authorized snapshot plus catch-up. **[P]**

The first broker and draft Part 3 transport should be **MQTT 5.0**, delivered as an optional experimental adapter after the durable log/replay seam exists. MQTT 3.1.1, WebSocket, NATS JetStream, and Kafka remain prepared adapter targets, not co-equal initial requirements. MQTT 5 is selected for active CS-Go/OSH precedent, IoT fit, content/user properties, session and expiry controls, and the current Part 3 direction; its QoS and broker acknowledgement do not substitute for application replay or consumer processing acknowledgement.[^7] NATS/Kafka may later serve deployment-internal distribution where measured scale or integration justifies them, but neither becomes the public CSAPI contract. **[I/X/P]**

The final Part 3 decision is **experimentally profile**. Glaux should plan and later implement outbound support for exact snapshot `6f529a15...` as `glaux-csapi-part3-exp/0.1`, disabled by default and never described as approved OGC Part 3 conformance. The profile uses Resource Events and Resource Data Messages; Batch Resource Events remain independently disabled pending measured need. It generates an AsyncAPI 3.0 description from deployed configuration, uses a versioned MQTT adapter, lists every deviation and capability, and exposes no inbound resource/command publication until the owning tasking and security topics authorize it. **[D/P]**

The September 15 refresh found one material delta from accepted IDR-SRV-014H. Official commit `6f529a15` nests Batch Resource Events under Resource Events, changes its requirement identifiers to `/req/resource-events/batch-events/...`, makes the class optional, and removes the checked-in HTML artifact. It does not supply the missing MQTT binding, ATS, Part 3 AsyncAPI, discovery rule, lowercase parent attribute, or resolutions for issues #14, #68, and #187–#195.[^4] CS-Go has one post-release main change unrelated to Pub/Sub; OSH PR #194 remains an unchanged open draft. This strengthens the need for a version adapter and updates the shared register to Version 1.12; it does not justify an approved conformance claim. **[D/I/X]**

Events are deliberately separated. Observations—including status Observations—SystemEvents, CommandStatus, and source-health evidence are persisted domain records with different meanings. Resource Events are lifecycle notifications; Resource Data Messages carry complete native records; projection invalidations say a rebuildable current/latest view may have changed; delivery records say an adapter attempted or completed a handoff; operational diagnostics and audit records remain private unless a separately authorized projection exists. A public event never becomes the authoritative fact merely because it arrived first. **[A/P]**

The recovery contract is snapshot-plus-catch-up: authorize and bind a subscription, select a policy-consistent snapshot watermark, transfer state as of that watermark, replay authorized events after it, then follow the live tail. Cursors are opaque, integrity-protected, bound to profile/filter/policy context, and advanced across filtered records without exposing global counts. Cursor expiry produces a safe resnapshot instruction rather than silent truncation. This design supports intermittent connectivity without claiming that MQTT, retained messages, or persistent sessions alone provide DDIL synchronization. **[P]**

## 2. Scope and Plan Alignment

### 2.1 Included scope

- Dynamic-resource, lifecycle, projection, delivery, source-health, command/status, diagnostic, and audit event taxonomy.
- Publication eligibility and triggers after commit, including correction, deletion, backfill, replay, projection changes, and aggregation.
- Durable event/outbox/inbox, delivery evidence, ordering, cursor, replay, backfill, retention, deduplication, acknowledgement, and failure behavior.
- HTTP polling, replay/change feed, SSE, WebSocket, MQTT 5/3.1.1, NATS JetStream, Kafka, and internal in-process distribution.
- Topic/channel syntax, subscriptions, wildcard/filter rules, content, discovery, policy, backpressure, DDIL, testing, observability, and interoperability.
- Exact Part 3 refresh, final disposition, profile pin, capability statement, feature gates, conformance boundary, adapter boundary, generated contract, and migration rule.

### 2.2 Excluded scope

- Server code, schema migrations, broker deployment, library selection lockfile, implementation tickets, or an approved implementation roadmap.
- Final Command/Feasibility state machines, safety interlocks, or inbound broker authority; these remain IDR-SRV-036 through 038 work.
- Final identity/policy/releasability model, DDIL conflict model, service decomposition, or deployment topology.
- An OGC Part 3 conformance claim, since no approved Part 3, complete binding, or applicable ATS exists.
- Treating the controlled AEP as permission to disclose or invent content not available in project evidence.

### 2.3 Research question coverage

| Plan question | Result | Principal coverage |
|---|---|---|
| Q1: What should be published? | Complete | §§5–6; Appendix A |
| Q2: How do event categories differ? | Complete | §5 |
| Q3: Which patterns/protocols? | Complete | §§9–11 |
| Q4: How do durability, replay, ordering, filtering, and authorization work? | Complete | §§7–13 |
| Q5: What downstream implications follow? | Complete | §§14–15, 17 |
| Q6: What is the Part 3 disposition? | Complete | §16; Appendix B |

### 2.4 Accepted-baseline reconciliation

This report applies, rather than reopens, the accepted decisions that PostgreSQL is authoritative; local effects are effectively once under scoped idempotency; delivery is at least once; committed mutations and outbox records are atomic; publication never precedes commit; derived projections are rebuildable; an Observation is a versioned typed fact; status is an Observation on a status DataStream; SystemEvent, source health, CommandStatus, audit, and delivery evidence are distinct; policy filters precede latest/aggregation/publication; and simulator/publisher traffic traverses the ordinary write boundary. The Part 3 delta changes draft class organization only and does not contradict those accepted semantics. **[A]**

## 3. Evidence Base and Authority

### 3.1 Primary sources

| Source | Exact state | Authority | Stable anchor/use | Limitation |
|---|---|---|---|---|
| CSAPI Parts 1/2 | OGC 23-001/23-002, Version 1.0 | N | HTTP resource and dynamic-data contract; Part 2 publication boundary | No approved Part 3 binding/replay contract |
| Part 2 AsyncAPI package | `v1.0.0` commit `8e03b236`, AsyncAPI 2.6, info version `0.0.1` | Informative artifact | Four SystemEvent/Observation/Command/CommandStatus channel families | No server/protocol binding; not normative; incomplete/outdated paths |
| CSAPI Part 3 | `part3-working-draft` `6f529a15`, `swg-draft` | D | Resource Events, optional Batch subclass, Resource Data, format-name rules | MQTT unwired; ATS/AsyncAPI missing; open defects |
| OGC API Pub/Sub Part 1 | OGC 25-030 Version 1.0, Draft, retrieved 2026-09-15 | D | discovery, AsyncAPI 3.0, CloudEvents/GeoJSON payload direction | Placeholder submission/approval/publication dates; not a Standard[^5] |
| OGC API EDR Part 2 | OGC 23-057r1 Version 1.0 | N | approved OGC Pub/Sub/AsyncAPI 3 precedent | EDR resource model, not CSAPI binding |
| CloudEvents | CNCF 1.0.2 core/JSON/MQTT | External specification | event identity, attributes, structured mapping | Does not define CSAPI topics, QoS, replay, or authorization |
| MQTT | OASIS 5.0 and 3.1.1 | External standard | transport/QoS/session/expiry properties | No domain transaction or replay guarantee |
| SSE | WHATWG HTML Living Standard, retrieved 2026-09-15 | Web standard | EventSource, `id`, reconnect, `Last-Event-ID` | Server-to-client only; no durable store defined |
| WebSocket | RFC 6455 | Internet standard | full-duplex framed connection | No application message, replay, or authorization contract |
| AsyncAPI | 3.0.0 and 2.6.0 | External specification | generated asynchronous API description | Description is not runtime evidence |

### 3.2 Accepted project and implementation evidence

| Source | Pin/status | Use | Limitation |
|---|---|---|---|
| IDR-SRV-014H | Accepted 2026-08-31 | Part 3 completeness/CS-Go/OSH divergence baseline | Superseded only for recorded `6f529a15` delta |
| IDR-SRV-025/027/029 | Accepted | persistence, time-series, transaction, outbox/inbox, idempotency | Exact DDL deferred |
| IDR-SRV-031–034 | Accepted | write pipeline, publisher/simulator, dynamic semantics, publication triggers | Publication mechanics owned here |
| CS-Go | release `v1.0.4` `244f4dd`; main `b1fd2e0` | working MQTT/Part 3 precedent | Local binding; volatile post-commit delivery; AsyncAPI 2.6 |
| OSH | draft PR #194 `50774ec7` | alternate MQTT hierarchy/security/format precedent | Open, unreleased, tests skipped in CI |
| Rust ecosystem | axum 0.8.9, rumqttc 0.25.1, async-nats 0.50.0, rdkafka 0.39.0 as retrieved | feasibility evidence only | Dependencies must be repinned during implementation |

### 3.3 Evidence quality and gaps

The approved Parts 1/2 and accepted Glaux reports control server meaning. Draft Part 3 is used only to define a named compatibility target. Implementation code proves feasibility, not obligation. The current official branch was fetched and mechanically compared to the accepted `c95c1d60` baseline. All tracked Part 3 issues remained open; CS-Go Pub/Sub paths were unchanged after `v1.0.4`; OSH PR state/head were unchanged. The official branch deleted its generated HTML, so the exact AsciiDoc source and retained PDF are the reproducible draft artifacts. **[D/I]**

No live broker interoperability or load test was executed in this research-only topic. Exact throughput, backlog, retention, Rust-client behavior, TLS/ACL integration, and edge sizing remain implementation/prototype measurements. The controlled AEP may inform mission need but cannot be redistributed or used to invent a public wire contract. **[X]**

## 4. Research Methodology

1. Establish approved Parts 1/2 and accepted IDR invariants before considering protocols.
2. Refresh only the material Part 3, issue/PR, CS-Go, and OSH evidence routed by IDR-SRV-014H.
3. Classify every record/message by authority, durability, subject, trigger, audience, sensitivity, replay need, and external mapping.
4. Trace mutation-to-outbox-to-log-to-adapter-to-consumer failure windows; reject semantics that lose committed events or imply exactly once.
5. Compare protocols against directionality, recovery, filtering, policy, broker burden, DDIL, client reach, Rust fit, and Part 3/interop value.
6. Design snapshot/catch-up, cursor, dedupe, backpressure, security, and discovery independently of one transport.
7. Test the strategy against late arrival, correction, deletion, policy changes, gaps, duplicate delivery, broker outage, restart, retained data, slow consumers, and draft migration.
8. Make the explicit Part 3 decision and validate every plan success criterion and downstream handoff.

## 5. Event and Record Taxonomy

### 5.1 Canonical distinctions

| Concept | Meaning and authority | Durable? | Public by default? | Part 3 relation |
|---|---|---:|---:|---|
| Domain fact/resource revision | Authoritative accepted System, stream, Observation, Command-family, or SystemEvent state | Yes | Through authorized HTTP/data messages | Resource Data payload or Resource Event subject |
| Status Observation | Typed Observation evidence on a `type=status` DataStream | Yes | If its resource is authorized | Observation Resource Data, not health/event metadata |
| SystemEvent | Discrete represented-System occurrence such as maintenance or calibration | Yes | If authorized | Native SystemEvent Resource Data |
| Source-health evidence | Publisher/adapter/connectivity state, not represented-System status | Yes or sampled/rolled up by policy | No; private operational projection | Glaux extension only if later authorized |
| CommandStatus/result | Task progress/result for a Command | Yes | Not until tasking/policy rules authorize | Eligible Resource Data family in draft |
| Domain change event | Internal immutable description of one committed semantic effect | Yes | No direct wire contract | Source for adapters |
| Resource Event | External lifecycle notification for create/update/delete | Replay-retained projection | Optional, authorized | Draft class `/req/resource-events` |
| Batch Resource Event | Count/time-window summary for one nested collection/type/operation | Durable aggregate if enabled | Disabled initially | Optional subclass `/req/resource-events/batch-events` |
| Projection invalidation | Signals current/latest/extents/cache may have changed | Durable only when external recovery needs it | No initial public profile | Glaux internal event |
| Delivery attempt/outcome | Adapter/broker handoff evidence | Yes within operations retention | No | Not a Resource Event |
| Operational diagnostic | Queue depth, connection, worker, broker, rate, or failure signal | Metrics/log/incident retention | No | Not a domain event |
| Security audit record | Actor, decision, policy, resource/event, result, and correlation evidence | Yes, protected | Never ordinary stream | Separate audit plane |

The same mutation may create several records without conflation: one Observation fact; one domain change event; one outbox work item; perhaps one Resource Data delivery and one Resource Event; zero or more attempts; and a projection update. Their identifiers and retention rules differ. **[A/P]**

### 5.2 Event identity

- `event_id`: stable identifier of one semantic event; retransmission/fan-out preserves it.
- `resource_id` plus `resource_revision`: identity/version of the affected fact or resource.
- `causation_id` and `correlation_id`: link source message, command, status/result, transaction, and derived work without making them the same event.
- `source_epoch` plus `source_sequence`: publisher-origin evidence where supplied; not the server cursor.
- `transaction_id` plus transaction-local ordinal: groups atomic effects without asserting cross-transaction order.
- `delivery_id` plus attempt number: identifies transport work; never substitutes for `event_id`.
- Part 3 Resource Events use CloudEvents `source`+`id`; Glaux deduplication keys include profile and tenant/API scope to avoid cross-boundary collision. **[D/A/P]**

## 6. Publication Triggers and Eligibility

### 6.1 Trigger rule

Only a **committed, authorized, publishable semantic effect** creates ordinary external publication work. Rejected, rolled-back, quarantined, duplicate-suppressed, or pending-dependency submissions do not emit an ordinary resource/data event. A duplicate request returns the original outcome and does not create a second semantic event. Publication retries reuse event identity. **[A/P]**

Corrections and accepted late/backfill facts are new revisions/events with flags and causal links; they never masquerade as initial live arrival. Deletion produces a lifecycle/tombstone event with enough authorized context to interpret a now-undereferenceable subject. Projection rebuilds do not replay domain creation events unless the source log proves previously undelivered work. **[P]**

### 6.2 Trigger classes

| Trigger | Internal event | External default | Notes |
|---|---|---|---|
| Create/replace/patch/delete committed | Domain change | Resource Event; data only when complete current representation exists | Delete never uses Resource Data |
| Observation/status accepted | Observation revision | Resource Data live stream; optional lifecycle event | Backfill/correction flagged; domain time preserved |
| SystemEvent accepted | SystemEvent revision | Resource Data; optional lifecycle event | Do not synthesize for diagnostics |
| Current/latest projection changes | Projection change | No separate initial public event | Subscribers infer/refetch; later extension may advertise |
| Source connects/disconnects/degrades | Source-health evidence | Private operations stream only | Not status Observation/SystemEvent automatically |
| Command/Feasibility/Status/Result changes | Task-domain event | Withheld until §§036–038 acceptance | Architecture seam retained |
| Outbox delivery/expiry/dead letter | Delivery event | Operations/audit only | Never presented as resource mutation |
| Policy changes | Policy event and subscription re-evaluation | Revoke/close/redact as allowed | Never disclose newly denied resources |

## 7. Durable Event Log, Outbox, Inbox, and Delivery Model

### 7.1 Required persistence layers

| Layer | Purpose | Transaction boundary | Retention |
|---|---|---|---|
| Authoritative domain store | Facts/resources/revisions and accepted effects | Same unit of work as admission/outbox | Domain lifecycle policy |
| Admission/idempotency inbox | Caller/source intent, fingerprint, decision, original result | Same write boundary | Retry/replay policy |
| Transactional outbox | One row per publishable semantic effect/adapter work seed | Atomic with domain commit | Until projected/delivered plus audit window |
| Durable change/event log | Replayable ordered partitions and immutable event metadata | Derived idempotently from outbox | Advertised replay horizon/holds |
| Subscription/checkpoint store | Managed subscriber/filter/profile/policy binding and acknowledged cursor | Separate conditional updates | Subscription lifecycle |
| Delivery ledger | Adapter, destination, attempt, broker result, expiry/dead-letter | Each attempt/result | Operations/audit policy |
| Aggregate ledger | Window/key/count/range/source offsets for optional batch events | Transactional/idempotent fold | At least source replay horizon |

An in-process broadcast channel may wake SSE/MQTT workers but is never the recovery source. A broker may retain messages, yet remains a delivery system rather than the only authoritative event log. **[P]**

### 7.2 Publication state machine

`committed -> outbox-pending -> log-appended -> adapter-pending -> attempted -> broker-accepted/stream-written -> retry-pending | expired | dead-lettered`

`broker-accepted` means the configured broker acknowledged the selected MQTT exchange level. `stream-written` means bytes were accepted by the HTTP connection path. Neither means the application consumer processed the event. Managed consumer acknowledgement, when later supported, is a separate checkpoint state. Domain commit is never rolled back because a later state fails. **[P]**

### 7.3 Delivery guarantee

Glaux claims:

- effectively-once local domain effects under accepted idempotency rules;
- atomic domain-effect plus outbox creation;
- at-least-once projection and transport delivery while within retention/expiry and authorized;
- possible duplicates across retries, reconnects, topic fan-out, gateway failover, and resnapshot boundaries;
- no global exactly-once or cross-transport atomic-delivery claim;
- explicit terminal expiry/dead-letter evidence rather than silent loss. **[A/P]**

## 8. Ordering, Cursors, Replay, Backfill, Retention, and Deduplication

### 8.1 Ordering model

Glaux does not promise one total order across all tenants, resources, adapters, or database transactions. It preserves:

1. resource revision/precondition order for one logical resource;
2. source epoch/sequence evidence without assuming it equals domain or commit order;
3. transaction-local effect order;
4. durable log order inside a declared partition;
5. broker ordering only within the documented protocol/broker scope.

Domain selection still uses phenomenon/result/applicability and accepted authority/correction rules, not arrival or event-log order. A later event may describe an earlier phenomenon time; consumers must not overwrite current state by arrival. **[A/P]**

### 8.2 Cursor contract

A cursor is opaque, integrity protected, scoped to API/tenant, profile version, filter hash, representation, partition set, and authorization context. It encodes or references a durable high-water position but exposes no global row count. The server reauthorizes every resume; it may narrow results, revoke the stream, or require resnapshot after policy/profile/filter changes. Cursors are not resource versions or event IDs. **[P]**

SSE places the opaque cursor in the SSE `id` field. A reconnect may submit `Last-Event-ID`; explicit replay may submit the same cursor through the advertised change-feed interface. MQTT 5 Resource Events carry a lowercase `glauxcursor` CloudEvents extension; native Resource Data carries `glaux-cursor` as an MQTT 5 User Property so the native payload remains unchanged. Clients store the last durably processed cursor, not merely the last received packet. **[P]**

### 8.3 Snapshot and catch-up

1. Authenticate and authorize the requested scope/filter/representation.
2. Establish a policy-consistent snapshot watermark `W` and issue a bound cursor.
3. Return/query authoritative state as of `W`, with stable paging.
4. Replay authorized log entries strictly after `W`.
5. Atomically transition to the live tail without a gap.
6. On reconnect, continue after the last processed cursor; duplicates at the boundary remain valid.

If the cursor is outside the retained horizon, the server returns an RFC 9457 problem with a stable `cursor-expired` type, no hidden-resource detail, and an authorized resnapshot link. Filter/profile mismatch is a safe `409`; malformed cursor is `400`; unavailable backlog capacity is `429` or `503` with retry guidance. **[P]**

### 8.4 Replay and deduplication

- Resource Events dedupe by profile/API scope plus CloudEvents `source`+`id`.
- Resource Data effects dedupe by resource identity/revision or accepted source/idempotency key; byte equality alone is insufficient.
- A repeated delivery uses the same event and resource revision identity but a new delivery attempt.
- Backfill/correction is new semantic work and must not be suppressed as transport replay.
- Consumer dedupe retention must cover advertised maximum replay/retry duration; after expiry, resnapshot is safer than assuming uniqueness memory.
- Batch Events are hints/summaries and cannot reconstruct individual IDs; recovery always uses authoritative query/change feed.

## 9. Payload, Envelope, and Content Strategy

### 9.1 Message families

| Family | Envelope | Payload | Use |
|---|---|---|---|
| Resource Event | CloudEvents 1.0.2 structured JSON | Omitted by default; bounded authorized summary only when profile enables | Lifecycle notification/reference |
| Batch Resource Event | CloudEvents 1.0.2 structured JSON | Required count and declared visibility-time range | Optional traffic summary, not recovery |
| Resource Data | None | One complete native CSAPI representation | Low-latency data delivery |
| Glaux control/diagnostic | Versioned private schema | Cursor/health/backpressure/revocation metadata | Non-Part 3 operational plane |

CloudEvents context is metadata, not a substitute for the native Observation/SystemEvent/Command schema. `source` is the configured public CSAPI root, `subject` is the canonical authorized resource or collection URL, `time` is declared commit visibility time for lifecycle events, and domain times remain in the resource.[^6] **[D/P]**

### 9.2 Encoding and negotiation

Initial Resource Data publication supports canonical JSON encodings already validated by the write/read boundary. SWE Text/Binary publication remains capability-gated until size, schema/version, broker, and client tests pass. The profile uses Part 3's deterministic media-type-to-format token algorithm for channel selection, but every enabled media type/token pair is generated into AsyncAPI. Content negotiation never means “publish every representation.” **[D/P]**

Payloads carry no secrets, bearer tokens, internal URLs, raw policy labels not authorized for the subscriber, or unrestricted names/descriptions merely because the Resource Event summary permits them. Large snapshots remain HTTP retrievals by reference unless a bounded profile explicitly advertises them. Compression is transport/deployment specific and subject to decompression limits. **[P]**

## 10. Publication Protocol Evaluation and Selection

### 10.1 Evaluation matrix

| Pattern | Direction | Durable replay native? | Security/policy fit | Operational burden | Part 3/interop value | Glaux decision |
|---|---|---:|---|---|---|---|
| CSAPI HTTP polling/query | Request/response | Authoritative history/query | Strong existing boundary | Low | Approved baseline | Required fallback and snapshot source |
| HTTP change feed | Server response pages | Yes, Glaux log/cursor | Same HTTP policy | Low–medium | Profile extension | Required recovery interface; exact route later generated |
| SSE | Server -> client | Reconnect token only; Glaux log supplies replay | Strong HTTP fit; per-request filters | Low | Possible future Part 3 extension | First live adapter |
| WebSocket | Bidirectional | No application replay | Session policy/reauth complex | Medium | No current binding | Prepared; defer until bidirectional need proven |
| MQTT 5 | Bidirectional broker | Session/QoS features, not domain replay | Requires broker+gateway ACL parity | Medium | Intended draft binding; CS-Go/OSH precedent | First broker and Part 3 experimental adapter |
| MQTT 3.1.1 | Bidirectional broker | More limited metadata/session controls | Same ACL burden | Medium | Existing-device reach | Optional compatibility adapter after MQTT 5 |
| NATS Core | Bidirectional broker | No durable Core replay | Strong internal service fit | Medium | No Part 3 binding | Internal adapter candidate only |
| NATS JetStream | Bidirectional durable broker | Yes with consumers/streams | Strong but separate policy plane | Medium | No Part 3 binding | Scale/deployment candidate after benchmark |
| Kafka | Partitioned durable log | Yes, offsets/retention | Mature but heavy edge footprint | High | No Part 3 binding | Enterprise integration adapter, not baseline |
| PostgreSQL notifications/in-process bus | Internal wake-up | No | Process trust boundary | Low | None | Optimization only; never authoritative |

### 10.2 Selection rationale

SSE is selected first because the initial requirement is authorized server-to-client notification, not arbitrary duplex messaging. It reuses HTTP TLS/authentication/origin/rate controls and axum has a direct async SSE response facility.[^14] The server's own cursor/log provides the durable semantics that EventSource does not. **[X/P]**

MQTT 5 is the first broker adapter because Part 3 names MQTT 3.1.1/5, both studied implementations use MQTT, and Version 5 offers User Properties, Message Expiry Interval, Session Expiry Interval, reason codes, and flow-control facilities useful to a precise profile.[^7] The experimental profile requires MQTT 5; a separately advertised 3.1.1 compatibility mode may use structured CloudEvents but cannot silently omit required cursor/recovery metadata. **[D/I/P]**

WebSocket is deferred because duplex frames do not define subscription discovery, replay, ordering, or idempotency, while command ingress is not yet authorized.[^10] NATS and Kafka are deliberately internal/deployment adapters: current Rust clients are feasible, but making either the public standard-facing contract would add an invented binding and operational dependency without demonstrated need.[^11][^12] **[X/P]**

### 10.3 Rust feasibility

The current Rust ecosystem makes all selected adapters practical: axum 0.8.9 supplies SSE and WebSocket response/extract support; rumqttc 0.25.1 supplies Tokio-based MQTT v3/v5 clients; async-nats 0.50.0 supplies Core/JetStream APIs; rdkafka 0.39.0 supplies async Kafka clients but adds `librdkafka` build/runtime considerations.[^14] These are feasibility snapshots, not dependency selections. Implementation must repin, security-audit, license-check, cross-compile, and exercise reconnect/backpressure before adoption. **[I/P]**

## 11. Topics, Channels, Subscriptions, Filters, Delivery, and Backpressure

### 11.1 Canonical subscription model

All adapters compile one logical subscription:

`principal + policy context + resource family + canonical scope + direction + event class + representation + query filter + starting cursor + delivery limits`.

The compiler validates capability and policy before creating transport state. Protocol topics are adapter output, never domain identity. Unsupported filter/protocol combinations fail explicitly rather than broadening access. **[P]**

### 11.2 MQTT experimental topic layout

Profile `0.1` uses one configured, non-secret namespace and exactly these UTF-8 topic families:

```text
{prefix}/glaux-csapi-part3-exp/0.1/events/{canonical-relative-resource-path}
{prefix}/glaux-csapi-part3-exp/0.1/batch/{canonical-relative-collection-path}
{prefix}/glaux-csapi-part3-exp/0.1/data/{canonical-relative-collection-path}/{format-name}
```

`prefix` identifies deployment/tenant routing and is generated in discovery; it is never inferred from the public URL. Canonical path segments use the approved HTTP spelling and percent-encoding rules, without scheme/authority/query/fragment. One Resource Event is published once on its individual resource path; collection subscribers use broker wildcards authorized by the gateway, avoiding duplicate collection+individual fan-out. Resource Data uses its parent collection path. Batch uses the summarized collection path and is disabled in the initial capability set. **[P]**

This is a Glaux binding deviation, not a draft requirement. Adapters may translate CS-Go `:events/:batch-events/:data` and OSH `api/.../:data/<format>` channels, but Glaux does not advertise those aliases as profile `0.1`. **[I/P]**

### 11.3 MQTT delivery settings

- MQTT 5.0 only for declared profile `0.1`; TLS required outside an explicitly isolated test fixture.
- QoS 1 default for outbound Resource Events/Data; QoS 0 may be a separately named lossy telemetry capability; QoS 2 is not an end-to-end exactly-once claim and is not initial scope.
- Retain is false for ordinary events/data. A separately authorized bootstrap pointer may be retained later, never a sensitive full payload by default.
- Clean Start false with bounded Session Expiry may support managed clients; session state does not replace Glaux replay cursors.
- Message Expiry reflects delivery usefulness/retention policy, while durable log retention governs catch-up.
- Packet, inflight, rate, queue, wildcard, subscription, and connection limits are mandatory configuration and AsyncAPI/deployment documentation inputs.

### 11.4 Filters and backpressure

SSE/change-feed filters reuse authorized server query primitives and may narrow by resource family, canonical parent/scope, event class/operation, time window, and supported semantic properties. Payload-content filtering that cannot be enforced before disclosure is rejected. MQTT topic wildcards provide coarse routing only; policy-aware gateways/ACLs must narrow actual resources and directions. **[P]**

Every adapter uses bounded memory. Slow SSE clients are disconnected with their last completed cursor; no durable event is discarded. Broker workers stop claiming or extend leases when inflight limits are reached. Persistent backlog degrades health, rejects new optional subscriptions, and may return `429/503`; it never causes unrecorded domain rollback or silent “success.” Load shedding prioritizes security revocation/control, command-status safety evidence, lifecycle deletion/correction, then ordinary telemetry according to later policy. **[P]**

## 12. Security, Authorization, Releasability, and Audit Boundaries

### 12.1 Enforcement model

| Layer | Responsibility |
|---|---|
| API/policy service | Authenticate, authorize logical subscription/publication, conceal existence, compile filter, reauthorize replay |
| Adapter/gateway | Bind protocol identity to principal, enforce direction/schema/size/rate, prevent topic escape, attach trusted cursor metadata |
| Broker | Mutual TLS or approved authentication, least-privilege publish/subscribe ACLs, quotas, isolation, session/retain/expiry controls |
| Domain write service | Revalidate inbound resource/command semantics and authority; never trust broker routing alone |
| Audit/operations | Record identity, policy version, scope, event/delivery IDs, decision, attempt/result, drops/expiry/dead letter without secrets |

The broker and service are one security surface. Application rejection is insufficient if clients can publish directly to reserved topics; broker ACLs are insufficient if a broad topic reveals resources denied by HTTP. Long-lived subscriptions must be re-evaluated or bounded when identity/policy changes. **[A/P]**

### 12.2 Disclosure controls

- Topic names themselves are disclosure; a principal receives no subscription/discovery template that reveals denied IDs or tenant topology.
- Global offsets, counts, batch summaries, lag, and heartbeats may leak activity; cursors are opaque and batch publication is policy-filtered or disabled.
- Deletion, correction, status, command, source-health, and retained/session data receive explicit sensitivity treatment.
- AsyncAPI is generated from active configuration and caller-visible capabilities, never credentials; broker endpoints expose public names only.
- Replay applies current access policy and cannot be used to recover data that is now denied, subject to protected audit/legal-hold rules outside the stream.
- Resource Event summaries default omitted; canonical references may still disclose existence and require authorization.

### 12.3 Inbound publication gate

Profile `0.1` is outbound only. Non-server principals are denied `events` and `batch` publication at broker and gateway. Observation or status ingestion continues through accepted HTTP/GPC paths until IDR-SRV-036–040 define command/tasking and cross-boundary authority, and an explicit later profile version authorizes exact Resource Data types/directions. This avoids turning a transport experiment into a write-policy bypass. **[A/P]**

## 13. DDIL, Reconnect, Snapshot, Catch-up, and Federation

### 13.1 DDIL behavior

Glaux supports intermittent links by durable server log, bounded client/server checkpoints, explicit snapshot watermark, replay, expiry, and resnapshot—not by asserting “MQTT is DDIL.” A subscriber may disconnect, retain its last processed cursor, reconnect through any advertised adapter, reauthorize, catch up, and then follow live data. Format/profile changes may require a new snapshot. **[P]**

Priority, offline queue sizes, compression, expiry, stale/unknown presentation, conflict reconciliation, and command authorization while disconnected remain IDR-SRV-042/043 decisions. This report requires those topics to preserve the cursor/log semantics and forbids silent newest-arrival overwrite. **[A/P]**

### 13.2 Cross-adapter resume

A cursor names a logical event-log position rather than an SSE connection or MQTT packet ID. Cross-adapter resume is allowed only when profile, scope, representation, filter, and policy binding are equivalent; otherwise the server issues a new snapshot/cursor. MQTT packet IDs, sessions, and retained messages are not portable resume cursors. **[P]**

### 13.3 Federation

Federated Glaux nodes preserve original event identity/source, add auditable relay identity/delivery hops outside the domain payload, enforce local policy before onward publication, and dedupe loops. They do not rewrite origin to appear authoritative or expose upstream topics directly. Partition/cursor translation and conflicts are deferred to IDR-SRV-043, but the event log must retain origin, causation, policy, and hop evidence needed for that work. **[P]**

## 14. Command, Control, Feasibility, Status, and Health Publication

### 14.1 Status and health

Status Observations publish through the Observation Resource Data family and retain their DataStream contract, applicability time, authority, and correction semantics. A “current status changed” signal may be an internal projection invalidation; it is not a replacement fact. Source-health, broker-health, worker-health, and API health remain private operational evidence and never become status Observations or SystemEvents automatically. **[A/P]**

### 14.2 SystemEvents

A committed, authorized SystemEvent may publish as complete native Resource Data and may generate its own lifecycle Resource Event. The native event says what happened in the represented System; the lifecycle event says the SystemEvent resource was created/updated/deleted. Consumers must not conflate the two. **[A/P]**

### 14.3 Commands and feasibility

The architecture reserves outbound Resource Data channels for Command, CommandStatus, CommandResult, and future Feasibility records, with correlation/causation and replay. It does not decide legal states, dispatch, cancellation, reservation, feasibility, safety, or terminality. Initial profile `0.1` advertises none of these directions. IDR-SRV-036–038 must decide which server-originated records become publishable and whether any inbound broker operation can pass the same authorization, precondition, idempotency, feasibility, and safety boundary as HTTP/GPC. **[P]**

Broker acknowledgement can never mean command acceptance, feasibility, execution, or completion. Each of those meanings requires an authoritative domain record and later lifecycle rule. **[P]**

## 15. Observability, Fixtures, Conformance, Performance, and Interoperability

### 15.1 Required observability

Metrics and traces cover outbox age/count, log append lag, per-adapter backlog/inflight/retry/expiry/dead-letter, connected/subscribed clients, filtered/denied counts at safe aggregation, bytes/messages, serialization time, broker acknowledgement latency, SSE disconnect/resume, cursor expiry/resnapshot, replay lag, dedupe hits, aggregate windows, and policy revocations. Trace correlation spans domain transaction, event, outbox, adapter, delivery, and acknowledged checkpoint without putting secrets or high-cardinality payloads in metrics. OpenTelemetry semantic choices are finalized in IDR-SRV-048.[^16] **[P]**

### 15.2 Fixture and test minimum

| Suite | Required cases |
|---|---|
| Domain/outbox | commit/rollback crash windows, duplicate request, correction/backfill/delete, multi-effect transaction |
| Cursor/replay | snapshot race, reconnect boundary duplicate, expiry, malformed/tampered, filter/profile/policy change, cross-adapter resume |
| Ordering | late phenomenon time, concurrent revisions, source epoch reset, partition reorder, transaction-local order |
| SSE | `id`/`Last-Event-ID`, keepalive, slow client, disconnect, auth expiry, bounded memory |
| MQTT | QoS duplicate, reconnect/session expiry, broker outage/restart, ACL denial, topic escape, retain false, message expiry, oversized packet |
| Content | CloudEvents 1.0.2, native schemas, parent/event tokens, media/format mapping, deletion, unknown extensions |
| Security | cross-tenant wildcard, concealed ID, policy revoke, forged cursor, client event publish, credential/discovery leakage |
| Interop | Glaux profile plus CS-Go/OSH adapters, exact topic/field divergences, AsyncAPI-runtime agreement |
| DDIL/load | long outage/backlog, burst catch-up, resnapshot, expiry, fan-out, hot stream, slow subscribers, quota/load shedding |

### 15.3 Conformance boundary

Approved CSAPI Parts 1/2 conformance is tested independently of optional streaming. The experimental profile receives project contract tests and a requirement/deviation matrix, not an OGC Part 3 conformance badge. When an approved Part 3 and ATS exist, IDR-SRV-050/051 must delta-map every implemented capability, run applicable tests, and keep experimental and approved conformance declarations separate until migration completes. **[N/D/P]**

### 15.4 Performance gates

Before enabling each adapter in production-like deployments, measure sustainable writes/events per second, p50/p95/p99 commit-to-visible latency, log/outbox growth, replay throughput, reconnect storm, fan-out, serialization/encoding cost, broker memory/disk/network, SSE connection count, wildcard/filter cost, payload maxima, retention storage, and recovery time. Batch Events may be enabled only when measured benefit exceeds aggregation/recovery/policy complexity and no client treats them as lossless detail. **[P]**

## 16. Draft Part 3 Decision, Profile, Compatibility, and Migration

### 16.1 Option analysis

| Option | Benefit | Cost/risk | Decision |
|---|---|---|---|
| Adopt as approved contract | Strongest apparent commitment | False maturity/conformance; missing binding/ATS/discovery | Reject |
| Experimental version-pinned profile | Gains implementation/interoperability evidence while isolating churn | Local binding/migration/test burden | **Select** |
| Monitor only | No draft coupling | Delays valuable event seams and user-requested capability | Reject as too passive |
| Defer all external work | Small near-term scope | Loses timely prototype/interop feedback | Reject |
| Reject Part 3 | No draft risk | Discards valuable model and ecosystem direction | Reject |

### 16.2 Final decision

**Decision P-035-01: experimentally profile draft Part 3.** Plan an optional implementation named `glaux-csapi-part3-exp/0.1`, based on exact official commit `6f529a15bfa63259febc3620378d3e5a06305333`. It is disabled by default, outbound only, and not part of approved Parts 1/2 conformance. Initial declared classes are `/req/resource-events` and `/req/resource-data-messages`; `/req/resource-events/batch-events` and content-format variants are independently feature-gated. MQTT 5 is its broker binding; SSE is a separate Glaux change-feed adapter, not falsely labelled an OGC Part 3 binding. **[D/P]**

### 16.3 Exact deviations and capability wording

The implementation and generated AsyncAPI must say substantially:

> Experimental Glaux compatibility profile `glaux-csapi-part3-exp/0.1`, based on OGC API - Connected Systems Part 3 working-draft commit `6f529a15...`. This capability is not an approved OGC Standard conformance claim. It implements the advertised subset and Glaux MQTT 5 binding; deviations and delivery/replay semantics are listed in the service description.

Profile deviations are:

1. Glaux supplies the missing MQTT binding and AsyncAPI 3.0 discovery locally.
2. External parent context is lowercase `parentid` for CloudEvents compliance; internal state remains a semantic parent reference. The draft's mixed-case `parentId` is not emitted.
3. Draft event type namespace `org.ogc.api.consys` is retained in `0.1` for CS-Go/OSH compatibility; a future `csapi` change requires a profile version, never a silent alias.
4. Resource Event `data` is omitted by default; optional summaries are an advertised policy-safe capability.
5. One versioned topic family replaces the incompatible CS-Go/OSH layouts.
6. AsyncAPI 3.0 replaces the draft bibliography/Part 2 legacy 2.6 artifact for Glaux discovery.
7. Replay/cursors/outbox guarantees are Glaux extensions; MQTT QoS is described separately.
8. Batch Events are disabled initially and, when enabled, use the new nested draft class identifiers and visibility-time windows.

### 16.4 Feature gates

Required independent gates are: master experimental profile; SSE; MQTT; Resource Events; Resource Data; Batch Events; each resource family; server-to-client/client-to-server direction; each encoding/format; summary/snapshot content; wildcard scope; managed subscription; retain/session/expiry behavior; compatibility adapter; and production enablement. Discovery describes only active, authorized capabilities. **[P]**

### 16.5 Generated contract

Generate AsyncAPI 3.0 JSON/YAML from the same typed capability/topic/message/security registry used by runtime routing. It includes exact profile/upstream pins, server URLs without secrets, protocol version, channels/operations/messages, address parameters, payload schemas/media types, directions, QoS/retain/expiry/session constraints, security schemes, cursor header/extension, replay/snapshot links, limits, and deviations. Golden tests compare generated channels and negative/positive runtime routing. The legacy Part 2 file remains evidence, never Glaux's deployed contract.[^2][^13] **[P]**

### 16.6 Migration policy

Every upstream material trigger—binding/ATS/AsyncAPI addition, Part 3 public review/publication, common Pub/Sub publication, issue #187–#195 resolution, event/parent/topic/format change, CS-Go profile change, or OSH merge/release—causes a documented delta review. Breaking changes create `0.2`/`1.0` side-by-side adapters and AsyncAPI documents for a bounded compatibility window. Cursors identify profile version; incompatible cursors require migration or resnapshot. Removal requires telemetry, notice, tested translation/resnapshot, and project-lead approval. An eventual approved profile never silently inherits the experimental conformance claim. **[P]**

## 17. Downstream Handoff

| Topic | Binding decision from this report | Remaining ownership |
|---|---|---|
| 036 Control/Command lifecycle | Durable events/correlation; no broker ACK semantic | states, transitions, dispatch, cancellation, status/result publication |
| 037 Feasibility/async tasking | Replayable operation/status seam | feasibility workflow, polling/stream completion |
| 038 Command safety/audit | Inbound profile disabled; broker ACK not acceptance | interlocks, authority, emergency behavior |
| 039/039A/040 Security/policy | Logical subscription compiler; broker+gateway parity; opaque cursor | identity, ABAC/releasability, cross-boundary controls |
| 041 Audit | Event/outbox/delivery/cursor correlations | audit schema, retention, review/export |
| 042 DDIL semantics | snapshot+catch-up and expiry/resnapshot seam | priority, stale/unknown, offline limits |
| 043 Synchronization/conflict | origin identity, cursor/log, loop dedupe | federation, merge/conflict/reconciliation |
| 045 Architecture | transport-neutral publication core/adapters | module/process boundaries and ports |
| 046/047 Deployment/config | SSE first, MQTT 5 optional, no mandatory Kafka/NATS | topology, broker product, TLS/secrets, configuration schema |
| 048 Observability | required metrics/traces and health separation | final telemetry model/SLOs |
| 049 Backup/migration | log/checkpoint/profile state and migration | operational procedures and RPO/RTO |
| 050/051 Conformance | experimental contract tests separate from OGC claims | harness and requirement traceability |
| 052/053 Testing/fixtures | matrix in §15 and CS-Go/OSH adapter corpus | Rust seams, golden data, scenario packaging |
| 054 Performance | benchmark gates and Batch Event adoption gate | workloads, thresholds, capacity plans |
| 055 Security tests | negative ACL/wildcard/cursor/replay/retain tests | adversarial suite |
| 056 Interoperability | explicit Glaux/CS-Go/OSH translation boundary | executable cross-server/client matrix |
| 057 Final synthesis | Part 3 experimental decision and refresh triggers | final upstream refresh and reconciliation |

## 18. Recommendations and Implementation Sequence

1. **R-035-01 — Build the transport-neutral publication core first.** Atomic outbox, durable log, delivery ledger, cursor, replay, and idempotent adapters precede public streaming. Priority: Critical.
2. **R-035-02 — Implement HTTP snapshot/change-feed and SSE as the first vertical slice.** Prove authorization, cursor, catch-up, slow-consumer, and restart behavior without a broker. Priority: High.
3. **R-035-03 — Implement `glaux-csapi-part3-exp/0.1` over MQTT 5 next.** Keep it default-off, outbound-only, version-pinned, and generated from configuration. Priority: High.
4. **R-035-04 — Publish no exactly-once claim.** State local atomicity, broker handoff, application acknowledgement, replay, and duplicate behavior separately. Priority: Critical.
5. **R-035-05 — Generate and verify AsyncAPI 3.0.** The typed capability registry drives runtime and contract; tests detect drift. Priority: High.
6. **R-035-06 — Keep Batch Events off until benchmarked.** If enabled, persist aggregate state, use visibility time, flush/recover deterministically, and retain detail recovery. Priority: Medium.
7. **R-035-07 — Keep inbound MQTT publication off until tasking/security acceptance.** Later versions must traverse the normal write/safety boundary. Priority: Critical.
8. **R-035-08 — Treat topics, CloudEvents spelling, and vocabulary as adapter data.** Never leak draft strings into domain identity. Priority: High.
9. **R-035-09 — Make policy part of subscription and cursor identity.** Reauthorize replay and conceal topic/activity metadata. Priority: Critical.
10. **R-035-10 — Prototype against CS-Go and OSH fixtures.** Translate their divergent topics/parent spelling/format/batch behavior and record what is actually interoperable. Priority: High.
11. **R-035-11 — Add NATS/Kafka/WebSocket only from measured requirements.** Preserve ports; avoid baseline deployment burden. Priority: Medium.
12. **R-035-12 — Reopen the profile on material upstream triggers.** Use the register and explicit version migration, not silent drift. Priority: High.

### 18.1 Planning estimate

| Work package | Complexity | Planning estimate | Preconditions |
|---|---|---:|---|
| Publication domain/outbox/log/delivery schema and workers | High | 4–6 developer-weeks | Architecture, persistence, policy seams |
| Authorized change feed, snapshot/catch-up, cursor service | High | 3–5 developer-weeks | Query snapshots, retention, security |
| SSE adapter and tests | Medium | 1–2 developer-weeks | Change feed/cursor available |
| MQTT 5 experimental adapter, broker integration, AsyncAPI | High | 4–6 developer-weeks | Core, security/config, test broker |
| Part 3 messages/topics/golden corpus and CS-Go/OSH adapters | High | 3–5 developer-weeks | Profile accepted; fixtures available |
| Load, DDIL, failure, security, and operational hardening | High | 4–8 developer-weeks | Deployment and test strategies |

Estimates are non-additive ranges for roadmap planning, not commitments. Broker product, team size, security accreditation, cross-compilation targets, and required throughput may dominate them. **[X/P]**

## 19. Risks, Constraints, and Open Questions

### 19.1 Risk register

| Risk | Likelihood/impact | Control/owner |
|---|---|---|
| Draft changes break clients | High/High | exact pin, versioned adapter, side-by-side migration; 049/057 |
| Commit succeeds but publication is lost | Medium/Critical | atomic outbox/log and recovery; 045/052 |
| Exactly-once inferred from QoS 2 | High/High | layered guarantee wording/tests; 050/056 |
| Topic/filter leaks protected activity | High/Critical | subscription compiler, opaque cursors, broker+gateway ACL; 039–040/055 |
| Slow/reconnecting clients exhaust memory | High/High | bounded queues, durable catch-up, disconnect/load shed; 047/054 |
| Cursor retention too short for DDIL | Medium/High | advertised horizon, capacity measurement, resnapshot; 042/049/054 |
| Batch summaries conceal loss or leak counts | Medium/High | disabled initially; policy-aware measured gate; 040/054 |
| Broker session/retain preserves revoked data | Medium/Critical | retain false, bounded expiry, revocation tests; 040/055 |
| Command ingress bypasses safety | Medium/Critical | outbound-only initial profile; 036–040/055 |
| Generated AsyncAPI diverges from runtime | Medium/High | single registry and golden runtime tests; 050–053 |
| Edge deployment burden grows | Medium/High | SSE/no-broker baseline; optional adapters; 046/054 |
| CS-Go/OSH incompatibility persists | High/Medium | explicit compatibility adapters/corpus; 056 |

### 19.2 Open questions routed, not blocking this decision

- What exact external route/link relation represents the Glaux change-feed and snapshot contract? IDR-SRV-045/050.
- Which broker product/topology and certificate/identity bridge are selected? IDR-SRV-046/047/039.
- What retention horizon and partition strategy meet classified/tactical workload needs? IDR-SRV-030/042/049/054 implementation synthesis.
- Which command/feasibility/status/result directions become publishable? IDR-SRV-036–038.
- Can a policy-safe batch profile be useful after filtered counts and recovery costs are measured? IDR-SRV-040/054.
- Will Part 3 resolve MQTT, AsyncAPI, parent naming, namespace, and ATS before implementation begins? Monitor through IDR-SRV-057 and implementation intake.
- What exact throughput/SLO thresholds select PostgreSQL-only, NATS JetStream, Kafka, or another internal distributor? IDR-SRV-046/048/054.

## 20. Validation Against Plan Success Criteria

| Success criterion | Result | Evidence |
|---|---|---|
| Identify publishable records/changes and distinguish semantic categories | Met | §§5–6; Appendix A |
| Define durable persistence, event, outbox/inbox, delivery, and transient boundaries | Met | §7 |
| Define ordering, identity, cursor, replay, backfill, retention, dedupe | Met | §8 |
| Define payload/envelope/content/encoding strategy | Met | §9 |
| Evaluate SSE, WebSocket, MQTT, broker, log, polling patterns | Met | §10 |
| Select first implementation and full-scope candidates | Met | §§1, 10, 18 |
| Define topics/channels/subscriptions/filters/QoS/backpressure | Met | §11; Appendix B |
| Define security/policy/audit boundaries | Met | §12 |
| Define DDIL reconnect/snapshot/catch-up/federation handoff | Met | §13 |
| Bound command/status/health publication without preempting later topics | Met | §14 |
| Define observability, fixtures, conformance, performance, interop | Met | §15 |
| Refresh Part 3/CS-Go/OSH/issues and update material register delta | Met | §§1, 3, 16; register 1.12 |
| Make adopt/profile/monitor/defer/reject decision | Met: experimental profile | §16 |
| Pin version, capability, gates, claim, adapter, contract, migration, sequence | Met | §§16, 18; Appendix B |
| Provide downstream decision-usable handoffs and risks | Met | §§17–19 |

Research phases 1 through 6, report drafting, and author review are complete. Plan-owner acceptance remains deliberately unchecked while this report is **In Review**. IDR-SRV-036 is not authorized by this report.

## 21. References and Sources

### 21.1 Standards and draft sources

1. [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html).
2. [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html).
3. [Tagged Part 2 AsyncAPI 2.6 artifact at `v1.0.0`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/v1.0.0/api/part2/asyncapi/asyncapi-connectedsystems-2.yaml).
4. [Draft Part 3 tree at `6f529a15`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/6f529a15bfa63259febc3620378d3e5a06305333/api/part3) and [material Batch subclass commit](https://github.com/opengeospatial/ogcapi-connected-systems/commit/6f529a15bfa63259febc3620378d3e5a06305333).
5. [Draft OGC API - Publish-Subscribe Workflow - Part 1: Core, OGC 25-030](https://docs.ogc.org/DRAFTS/25-030.html) and [official overview](https://ogcapi.ogc.org/pubsub/overview.html).
6. [OGC API - EDR - Part 2: Publish-Subscribe Workflow, OGC 23-057r1](https://docs.ogc.org/is/23-057r1/23-057r1.html).
7. [CloudEvents 1.0.2 core](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/spec.md), [JSON format](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/formats/json-format.md), and [MQTT binding](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/bindings/mqtt-protocol-binding.md).
8. [OASIS MQTT Version 5.0](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html) and [MQTT Version 3.1.1](https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/mqtt-v3.1.1.html).
9. [AsyncAPI Specification 3.0.0](https://www.asyncapi.com/docs/reference/specification/v3.0.0) and [2.6.0](https://www.asyncapi.com/docs/reference/specification/v2.6.0).
10. [WHATWG HTML: Server-sent events](https://html.spec.whatwg.org/multipage/server-sent-events.html).
11. [RFC 6455: The WebSocket Protocol](https://www.rfc-editor.org/rfc/rfc6455).
12. [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110) and [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457).

### 21.2 Implementation, protocol, and Rust sources

13. [CS-Go release `v1.0.4`](https://github.com/SomethingCreativeStudios/connected-systems-go/releases/tag/v1.0.4), [Pub/Sub source](https://github.com/SomethingCreativeStudios/connected-systems-go/tree/244f4dd586da685d4d9b75e43f73001028b5bd0e/internal/pubsub), and [MQTT source](https://github.com/SomethingCreativeStudios/connected-systems-go/tree/244f4dd586da685d4d9b75e43f73001028b5bd0e/internal/mqtt).
14. [OpenSensorHub `osh-addons` PR #194](https://github.com/opensensorhub/osh-addons/pull/194).
15. [NATS JetStream documentation](https://docs.nats.io/nats-concepts/jetstream) and [Apache Kafka design documentation](https://kafka.apache.org/documentation/#design).
16. [axum 0.8.9 SSE](https://docs.rs/axum/0.8.9/axum/response/sse/), [rumqttc 0.25.1](https://docs.rs/rumqttc/0.25.1/rumqttc/), [async-nats 0.50.0](https://docs.rs/async-nats/0.50.0/async_nats/), and [rdkafka 0.39.0](https://docs.rs/rdkafka/0.39.0/rdkafka/).
17. [OpenTelemetry documentation](https://opentelemetry.io/docs/).

### 21.3 Project sources

18. [Accepted IDR-SRV-014H Part 3 study](idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md).
19. [Accepted IDR-SRV-029 transaction strategy](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md).
20. [Accepted IDR-SRV-031 write and ingestion model](idr-srv-031-server-write-and-ingestion-model-report.md).
21. [Accepted IDR-SRV-032 publisher boundary](idr-srv-032-publisher-to-server-contract-boundary-report.md).
22. [Accepted IDR-SRV-033 simulator boundary](idr-srv-033-simulator-to-server-contract-boundary-report.md).
23. [Accepted IDR-SRV-034 dynamic update semantics](idr-srv-034-datastream-observation-and-status-update-semantics-report.md).
24. [OGC Connected Systems upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md).

### 21.4 Numbered source notes

[^2]: OGC 23-002 assigns publish/subscribe protocol binding to Part 3. The tagged Part 2 support file declares AsyncAPI 2.6 and document version `0.0.1`; it is informative implementation support, not a complete deployed broker contract.
[^4]: Official commit `6f529a15` (September 1, 2026) changes the Batch class identifiers and inheritance, nests it under Resource Events, and removes the generated HTML. The source entry still explicitly identifies `swg-draft`, an unwired MQTT stub, absent ATS, and missing annexes. Retrieved 2026-09-15.
[^5]: OGC 25-030 identifies document number 25-030, Version 1.0, Draft stage, placeholder submission/approval/publication dates, AsyncAPI 3.0 schemas, and CloudEvents/GeoJSON payload classes. Retrieved 2026-09-15.
[^6]: CloudEvents 1.0.2 defines event context and `source`+`id` identity; its MQTT binding leaves topic and application delivery decisions outside the binding.
[^7]: MQTT 5 defines QoS packet flows and features such as User Property, Message Expiry Interval, Session Expiry Interval, Receive Maximum, and reason codes. None makes a database mutation and application consumption one atomic transaction.
[^9]: WHATWG EventSource defines SSE parsing, the `id` field, reconnect behavior, and `Last-Event-ID`; it does not prescribe Glaux storage, authorization, or retention.
[^10]: RFC 6455 defines a bidirectional framed protocol after the opening handshake; application messages, subscription semantics, replay, authorization, and recovery remain application concerns.
[^11]: NATS JetStream supplies persisted streams and stateful consumers; Core NATS and JetStream remain deployment-specific infrastructure rather than an OGC Part 3 binding.
[^12]: Kafka supplies partitioned logs, offsets, retention, and consumer groups; its ordering is partition-scoped and its operational footprint is not justified as the Glaux baseline without measured need.
[^13]: AsyncAPI describes channels, operations, messages, servers, and bindings; only generation from and testing against runtime configuration makes it trustworthy discovery evidence.
[^14]: Versions are dated feasibility observations retrieved 2026-09-15. They are not implementation dependency decisions and must be refreshed, audited, and pinned before code work.
[^16]: OpenTelemetry provides traces, metrics, and logs conventions/tooling; final semantic naming and SLOs remain IDR-SRV-048 work.

## Appendix A. Publication Decision Matrix

| Category | Trigger | Source | Durable/transient | Pattern | Ordering/cursor | Replay/backfill | Payload | Auth/policy | Guarantee | DDIL | Tests | Part 3 mapping/status | Capability/migration | Handoff | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| System/metadata lifecycle | committed create/update/delete | domain revision | durable | Resource Event + HTTP fetch | resource revision + log cursor | log then refetch; deletion tombstone | CE ref/optional summary | scope before emit | at least once | cursor catch-up | CRUD/crash/policy | Resource Events, draft | exp gate/version | 045/050/056 | no pre-commit event |
| Observation | accepted exact-contract fact/revision | DataStream write | durable | native data + optional lifecycle | stream/resource revision + cursor; domain time separate | query/log; backfill flagged | complete OM JSON initially | stream/FOI/property policy | at least once | snapshot/catch-up | late/correct/duplicate/load | Resource Data + Resource Event, draft | family/encoding gate | 042/054/056 | status uses same family |
| Status Observation | accepted status fact | status DataStream | durable | native Observation data | applicability selector independent of cursor | history/current snapshot | complete native Observation | status disclosure policy | at least once | replay/resnapshot | authority/tie/hidden | Observation Resource Data | status-family gate | 040/042 | not source health |
| SystemEvent | accepted System occurrence | SystemEvent store | durable | native data + lifecycle | event resource revision/cursor | history/log | complete SystemEvent | system/event policy | at least once | replay/resnapshot | type/time/delete | Resource Data eligible | gate | 041/042 | not CRUD notification |
| Source health | connect/degrade/recover sample | publisher/gateway | durable rollup + transient metrics | private ops stream/metrics | source epoch/time | bounded ops history | private schema | operations only | documented best effort/logged | ops recovery | flap/redact | no Part 3 mapping | private extension | 041/048 | never auto-status |
| Current/latest change | authoritative fact changes selector | projection worker | rebuildable + optional durable invalidation | internal invalidation/refetch | source fact cursor | rebuild from facts | IDs/watermarks only | policy before projection | no independent fact claim | recompute after catch-up | late/correction/policy | none | public disabled | 034/042 | avoid arrival overwrite |
| Command | later accepted/dispatchable transition | command service | durable | later native/lifecycle | command revision/correlation/cursor | task history/log | complete approved representation | command authority/safety | not defined here | later DDIL gate | race/safety/ACL | Resource Data eligible | profile `0.1` off | 036–038 | broker ACK not acceptance |
| CommandStatus/result | later valid task report | task service | durable | later native/lifecycle | per-command state order/cursor | task history/log | complete native record | source/target/read policy | at least once delivery only | later replay | invalid transition/reorder | Resource Data eligible | profile `0.1` off | 036–038 | no state machine here |
| Feasibility | later request/result transition | feasibility service | durable | later native/private as decided | operation revision/cursor | operation query/log | later schema | task policy | not defined here | later replay | timeout/cancel | unresolved Part 3 | off | 037 | no invented mapping |
| Batch summary | enabled measured aggregate closes | aggregate ledger | durable aggregate | Batch Resource Event | visibility window/end cursor | detail through query/log | CE count/timerange | policy-safe aggregation | hint, at least once | never sole recovery | crash/late/filtered count | optional nested draft subclass | initially off | 040/054 | not atomic batch |
| Delivery outcome | adapter attempt/ack/expire | delivery ledger | durable | private ops/audit | delivery attempt order | retry ledger | metadata only | operations/audit | evidence, not domain delivery | recovery worker | crash/retry/dead letter | none | internal | 041/048/049 | separate broker/app ACK |
| Audit event | security/domain decision | audit service | durable protected | audit query/export | audit sequence/time | protected retention | actor/decision/correlation | audit role only | evidentiary | controlled replication | tamper/redact | none | never ordinary stream | 041/055 | no payload secrets |

## Appendix B. Experimental Profile Contract

| Dimension | `glaux-csapi-part3-exp/0.1` rule |
|---|---|
| Upstream pin | `part3-working-draft` `6f529a15bfa63259febc3620378d3e5a06305333` |
| Claim | Experimental compatibility profile; no approved OGC Part 3 conformance |
| Default | Disabled; production enablement separately controlled |
| Binding | MQTT 5.0 over TLS except isolated tests |
| Initial direction | Server-to-client only |
| Initial classes | Resource Events and selected Resource Data; Batch disabled |
| Initial data families | Observation/status Observation and SystemEvent after exact capability approval |
| Event format | CloudEvents 1.0.2 structured JSON |
| Parent attribute | `parentid`; explicit deviation from draft `parentId` |
| Type namespace | Draft `org.ogc.api.consys` for `0.1`; versioned migration if changed |
| Resource Event data | Omitted by default; bounded summary independently gated |
| Native data | One complete validated CSAPI representation, JSON first |
| Topics | §11.2 exact versioned families |
| QoS/retain | QoS 1 default; retain false; no exactly-once claim |
| Cursor | `glauxcursor` CE extension or MQTT 5 `glaux-cursor` User Property |
| Replay | Authorized Glaux change feed/snapshot, not MQTT session alone |
| Discovery | Generated AsyncAPI 3.0 plus landing-page links when enabled |
| Security | Broker and gateway identity/ACL, topic concealment, quotas, audit |
| Compatibility | Explicit CS-Go/OSH adapters, never silent topic aliases |
| Migration | Side-by-side version/profile, delta report, cursor translate or resnapshot |
| Enablement gates | Core durability, policy, broker security, generated-contract, negative/failure/load/interop tests |

## Appendix C. Reproducible Refresh and Validation Record

### C.1 Upstream pins

```powershell
git ls-remote https://github.com/opengeospatial/ogcapi-connected-systems.git refs/heads/master refs/heads/part3-working-draft refs/tags/v1.0.0
git ls-remote https://github.com/SomethingCreativeStudios/connected-systems-go.git refs/heads/main refs/tags/v1.0.4
git ls-remote https://github.com/opensensorhub/osh-addons.git refs/pull/194/head
```

Observed September 15, 2026:

```text
CSAPI master                3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f
CSAPI part3-working-draft   6f529a15bfa63259febc3620378d3e5a06305333
CSAPI v1.0.0               8e03b236a049849f2ccc24b4fd9fdce5ff69bed2
CS-Go main                  b1fd2e0e9bd69e222d05258d659a842ca24502cb
CS-Go v1.0.4 commit         244f4dd586da685d4d9b75e43f73001028b5bd0e
OSH PR #194 head            50774ec7e9c98f6ab8da827171e5c5abb9923a49
```

### C.2 Part 3 delta and artifact checks

```powershell
git log --oneline c95c1d6003359d0883c4dc759d7a148ab115fdb1..6f529a15bfa63259febc3620378d3e5a06305333
git diff --stat c95c1d6003359d0883c4dc759d7a148ab115fdb1..6f529a15bfa63259febc3620378d3e5a06305333 -- api/part3
rg -n 'TODO|Still missing|parentId|AsyncAPI|annex_ats|clause-protocol-mqtt' api/part3/standard --glob '*.adoc'
```

Result: one commit, 15 Part 3 files affected, 66 additions/2,727 deletions (2,659 deletions are the removed generated HTML); Batch identifiers/inheritance changed; MQTT/ATS/AsyncAPI and named defects remain. The tree has 48 files, one PDF, no HTML, no AsyncAPI, and no ATS. All three event JSON examples parsed. SHA-256 values:

```text
23-003r0.adoc  CAF6A8257ACD4F5DE8B4556FF3929A0BF5A7D4E2F129E20C6519816B72CB0BB7
23-003r0.pdf   BF85A8787D11B8EF6858D8328412266A652D118B6F3651619FC921F68AF50E74
```

GitHub API checks on September 15 found official issues #14, #68, and #187–#195 all open. OSH PR #194 was open, draft, unmerged, and unchanged at the pin. CS-Go diff/log inspection found no changes under `internal/pubsub`, `internal/mqtt`, or `internal/api/asyncapi_handler.go` after release `v1.0.4`; its later main changes do not materially update the accepted Pub/Sub baseline.

### C.3 Report validation targets

```powershell
rg -n '^## ' idr-srv-035-streaming-and-event-publication-strategy-report.md
rg -n '^\[\^[0-9]+\]:' idr-srv-035-streaming-and-event-publication-strategy-report.md
git diff --check
```

## Report Completion Checklist

- [x] Topic ID matches overall research plan index
- [x] Topic research plan is linked and aligned
- [x] Core and detailed research questions are covered or routed explicitly
- [x] Findings are evidence-backed with reproducible references
- [x] Normative, draft, accepted, implementation, inference, and recommendation evidence are distinguished
- [x] Mutable sources identify version, release, tag, commit, and retrieval date
- [x] Controlled, missing, incomplete, and ambiguous evidence limits are explicit
- [x] Material Part 3 delta is recorded in the shared register
- [x] Accepted prior-report decisions are reconciled
- [x] Executive summary is independently decision-usable
- [x] Recommendations and implementation sequence are explicit
- [x] Risks and open questions are documented and routed
- [x] Plan success criteria are validated
- [ ] Plan-owner acceptance and acceptance date recorded
- [x] Next-topic handoff is defined without authorizing IDR-SRV-036

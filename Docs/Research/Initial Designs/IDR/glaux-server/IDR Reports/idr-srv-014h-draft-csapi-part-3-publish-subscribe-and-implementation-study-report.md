# Section 014H: Draft CSAPI Part 3 Publish/Subscribe and Implementation Study - Research Report

**Topic ID:** IDR-SRV-014H<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-014H Draft CSAPI Part 3 Publish/Subscribe and Implementation Study](../IDR%20Plans/idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core question groups, all detailed question groups, all 6 methodology phases, and all 12 success criteria<br>
**Methodology Used:** Immutable draft and implementation pins; source/include, requirement, issue, pull-request, generated-artifact, code, configuration, and test inspection; focused local Go test execution; cross-implementation field/topic mapping; authority and adoption-readiness classification<br>
**Research Time:** Approximately 5 hours of AI-assisted execution on August 31, 2026<br>
**Official Part 3 Baseline:** `part3-working-draft` at [`c95c1d6003359d0883c4dc759d7a148ab115fdb1`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/c95c1d6003359d0883c4dc759d7a148ab115fdb1), retrieved August 31, 2026<br>
**CS-Go Baseline:** `main` / release `v1.0.4` at [`4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952`](https://github.com/SomethingCreativeStudios/connected-systems-go/commit/4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952)<br>
**OpenSensorHub Baseline:** open draft `osh-addons` PR [#194](https://github.com/opensensorhub/osh-addons/pull/194), head [`50774ec7e9c98f6ab8da827171e5c5abb9923a49`](https://github.com/opensensorhub/osh-addons/commit/50774ec7e9c98f6ab8da827171e5c5abb9923a49)<br>
**Shared Register Baseline:** OGC API - Connected Systems upstream-history register version 1.9<br>
**Document Purpose:** Establish whether draft Part 3 is ready to influence Glaux design, distinguish draft requirements from implementation precedent, expose interoperability and operational gaps, and hand a bounded decision backlog to IDR-SRV-035 without selecting the final streaming architecture<br>
**Author:** OpenAI Codex<br>
**Date:** August 31, 2026<br>
**Last Updated:** August 31, 2026

---

## Reading Guide

| Label | Meaning |
|---|---|
| P | Published, approved standard or accepted Glaux baseline |
| D | Pre-publication OGC draft source, generated draft artifact, issue, or working-group proposal |
| I | Implementation code, documentation, configuration, release, or pull request |
| T | Reproduced test, mechanical check, or immutable CI result |
| X | Analyst inference or comparison, not a standards claim |
| R | Glaux recommendation or downstream design constraint, not yet an implementation decision |

The controlling precedence is **published standard and accepted Glaux baseline, then exact draft text for draft-compatibility analysis, then implementation evidence, then inference and recommendation**. Neither code volume nor a passing build converts a draft choice into an OGC requirement.

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence, Authority, and Version Baseline
4. Part 3 Document Structure, Dependencies, and Completeness
5. Protocol-Agnostic Message Model
6. Protocol Binding, Topics, Discovery, AsyncAPI, and Conformance
7. CS-Go Implementation Evidence
8. OpenSensorHub Implementation Evidence
9. Cross-Implementation Comparison and Interoperability
10. Reliability, Transaction, Durability, Ordering, Replay, and DDIL
11. Security, Authorization, Publisher Authority, and Policy
12. Glaux Adoption Readiness
13. Downstream Handoffs and IDR-SRV-035 Decision Backlog
14. Recommendations
15. Risks, Open Questions, and Monitoring Triggers
16. Validation Against the Research Plan
17. References
Appendix A. Draft Requirement and Completeness Matrix
Appendix B. Reproducible Commands and Test Record
Appendix C. Detailed Question Coverage and Completion

---

## 1. Executive Summary

Draft OGC API - Connected Systems - Part 3 is valuable enough to remain an explicit **experimental implementation candidate** for Glaux, but it is not mature enough to define Glaux's external MQTT contract or support an OGC conformance claim. Its protocol-agnostic center is useful now: lifecycle Resource Events, aggregate Batch Resource Events, and complete native Resource Data Messages establish distinct message purposes; canonical CSAPI URLs, CloudEvents identity, create/update/delete vocabulary, and native encodings provide strong early inputs to the resource, lifecycle, transaction, ingestion, security, and fixture topics [D,R].

The external binding is not ready. The pinned source labels itself `swg-draft` and explicitly says that the MQTT binding is an unwired stub with broken include paths, the normative Abstract Test Suite is absent, and examples/relationships annexes are missing. The draft claims five requirements classes, but only Resource Events, Batch Resource Events, Resource Data Messages, and the optional format-name subclass are wired. No AsyncAPI document exists in the Part 3 tree. The generated HTML contains unresolved MQTT and ATS references. Therefore, an implementation cannot satisfy the draft's own expectation of at least one message class plus one protocol binding, nor can conformance be checked through the absent tests [D,T].

The draft also contains material internal or inherited inconsistencies. Resource Events may supposedly be restricted if the exact supported set is discoverable, while another requirement says create/update/delete events shall be published for every exposed resource type; the discovery mechanism is still TODO. The `parentId` extension spelling violates CloudEvents 1.0.2's lowercase-only attribute-name rule. The table calls it recommended while the Resource Event requirement makes it mandatory when a parent exists; Batch Events use `SHOULD`. The allowed-batch-collections requirement mistakenly calls a canonical URL a `topic`. Event type tokens still use legacy `org.ogc.api.consys` while issue #191 proposes `csapi`. AsyncAPI 2.6 remains in the Part 3 bibliography, whereas draft OGC API Publish-Subscribe Workflow Part 1 and approved EDR Part 2 use AsyncAPI 3.0 and issue #68 calls for alignment [D,P].

CS-Go demonstrates that the draft can become working server functionality. Its released `v1.0.4` implementation has independently gated message classes, generated AsyncAPI 2.6 discovery, strict inbound Observation and Command Status data-message validation, HTTP-triggered lifecycle events, aggregate Observation and Command events, reconnecting MQTT transport, echo suppression, and focused tests. Forty-four relevant tests passed locally at the pinned commit. Its documentation correctly refuses formal Part 3 conformance and treats `/asyncapi` as the implementation-specific source of truth because the MQTT binding is unfinished [I,T].

OpenSensorHub demonstrates a second viable integration shape but not a compatible one. Draft PR #194 adds event-bus-driven CloudEvents, node-prefixed hierarchical topics, wildcard validation and permission checks, optional format subtopics, and observation aggregate events. The PR is open, draft, unreviewed, and unreleased. Its green GitHub build explicitly runs Gradle with `-x test`; twelve checked-in methods test only topic/format validation, not publication or broker interoperability [I,T].

The implementations cannot consume one another without an adapter. CS-Go uses unprefixed REST-relative event topics ending `:events` or `:batch-events`; OSH defaults to an `api/` node prefix, nests streams beneath systems, and uses no event-class suffix. CS-Go emits draft-spelled `parentId`; OSH emits CloudEvents-compliant `parentid`. CS-Go offers no format suffix; OSH uses `:data/<format>`, including `swe-csv` for SWE Text where the draft normalization yields `swe-text`. CS-Go aggregates Observation and Command create/update/delete operations in clock-aligned windows and flushes a partial window on shutdown; OSH aggregates Observation creates only, begins its reported window at phenomenon time rather than resource-visibility time, and deliberately drops the pending window on stop [I,X].

Neither implementation establishes production delivery semantics. CS-Go publishes asynchronously after an HTTP/database mutation and skips publication while disconnected; OSH publishes from an in-process event bus and logs failures. Neither shows a transactional outbox, durable inbox, replay cursor, ordering contract, subscriber checkpoint, cross-channel atomicity, or end-to-end duplicate policy. MQTT can reduce polling and native data messages can reduce envelope overhead, but those properties do not create DDIL resilience. Replay, snapshot/backfill, retention, expiry, freshness, priority, and conflict behavior remain decisions for later Glaux topics [I,R].

The resulting recommendation is deliberately two-layered:

1. **Carry the abstract Part 3 model forward now** as a version-pinned design input: transport-neutral domain events, canonical resource references, stable event identity, lifecycle operation, optional summary/snapshot boundaries, native resource-data messages, aggregate summaries, and an outbox-capable transaction seam.
2. **Defer the external profile decision to IDR-SRV-035**. Any later experimental implementation must be disabled by default, isolated behind transport/topic adapters, publish an implementation-specific AsyncAPI artifact, name the exact draft commit and local profile version, make no OGC conformance claim, and define migration/removal behavior.

This is not a recommendation to defer Part 3 indefinitely. It is a recommendation to design Glaux so Part 3 can be added cleanly while refusing to freeze one implementation's invented MQTT hierarchy as the server contract.

---

## 2. Scope and Plan Alignment

### 2.1 Included Scope

- Official Part 3 branch, exact commit history, source entry, requirements, examples, HTML/PDF artifacts, and open Part 3 issues/PRs.
- Approved CSAPI Parts 1 and 2, draft OGC API Publish-Subscribe Workflow Part 1, approved EDR Part 2, CloudEvents 1.0.2 and its MQTT binding, MQTT 3.1.1/5.0, and AsyncAPI 2.6/3.0 roles.
- Resource Events, Batch Resource Events, Resource Data Messages, optional representation-channel naming, discovery, publisher authority, and claimed conformance.
- Released CS-Go and draft OSH implementation code, documentation, configuration, tests, release/merge state, CI state, and reproducibility constraints.
- Field/topic interoperability, durability, DDIL, security, authorization, deployment, observability, performance, migration, and test implications.
- Early constraints for intervening Glaux topics and a precise later-decision backlog for IDR-SRV-035.

### 2.2 Excluded Scope

- Selecting MQTT, NATS, Kafka, AMQP, DDS, SSE, a broker product, or a Rust library.
- Designing the final outbox/inbox, event store, retention policy, replay API, authorization engine, or synchronization protocol.
- Treating the open draft as an AEP-4789 obligation or approved CSAPI Part 3 publication.
- Promoting CS-Go or OSH topic names, suffixes, extra event types, or operational defaults into standards requirements.
- Implementing Part 3, creating engineering tickets, or starting IDR-SRV-015.
- Making the final `adopt / experimental profile / monitor / defer / reject` architecture decision owned by IDR-SRV-035.

### 2.3 Alignment with Accepted Glaux Baselines

Accepted IDR-SRV-003 found that the current adopted standards package contains no approved Part 3 binding and leaves transport, subscription, order, replay, backpressure, and QoS open. IDR-SRV-014A and 014B provide the controlling general OSH and CS-Go implementation baselines; this report narrows only to their current Part 3 work. IDR-SRV-014G correctly routed future Part 3 decisions to their owning topics. Nothing in this report converts draft Pub/Sub into a current NATO or CSAPI Parts 1/2 conformance obligation [G].

All six research phases were completed. The upstream register remained at version 1.9 because the official Part 3 commit, relevant issue states, CS-Go commit/release, and OSH PR head/state did not materially change during execution.

---

## 3. Evidence, Authority, and Version Baseline

### 3.1 Source Ledger

| Source | Exact state | Authority | Material use |
|---|---|---|---|
| CSAPI Part 1, OGC 23-001 | Approved 1.0, published 2025 | P | Resource identities, feature resources, encodings |
| CSAPI Part 2, OGC 23-002 | Approved 1.0, published 2025 | P | Dynamic resources and native encodings; delegates Pub/Sub bindings to Part 3 |
| Part 3 working branch | `part3-working-draft` at `c95c1d60`, `swg-draft` | D | Exact candidate requirements and completeness baseline |
| OGC API Publish-Subscribe Workflow Part 1, OGC 25-030 | Draft; placeholder dates; Draft stage | D | Inherited CloudEvents payload class, AsyncAPI 3.0 and discovery direction |
| OGC API EDR Part 2, OGC 23-057r1 | Approved 1.0, published September 23, 2024 | P | Existing OGC Pub/Sub discovery/channel precedent; issue #68 alignment context |
| CloudEvents core and JSON format | CNCF 1.0.2 | P external specification | Envelope, identity, attribute names, JSON structured format |
| CloudEvents MQTT binding | CNCF 1.0.2 | P external specification | Event-to-MQTT mapping only; deliberately does not define CSAPI topics, QoS, retain, or settlement |
| MQTT | OASIS 3.1.1 and 5.0 | P external standards | Transport capabilities; not a CSAPI topic or policy definition |
| AsyncAPI | 2.6 and 3.0 official specifications | P external specifications | Description syntax; Part 3 version/profile choice remains unsettled |
| Official issues #14, #68, #187-#195 and PR #198 | Issues open; PR #198 merged to draft branch | D | Rationale, questions, and mutable proposed resolution, not approved requirements |
| CS-Go | `main`/`v1.0.4` at `4a00aa6f`; original PR #16 merged | I | Released experimental implementation precedent |
| OSH `osh-addons` PR #194 | Open draft at `50774ec7`; no reviews; no release | I | Active but pre-merge implementation precedent |
| Accepted Glaux IDRs and register 1.9 | Accepted/current | G | Project scope, source precedence, and downstream ownership |

### 3.2 Freshness Results

- Official Part 3 remained at `c95c1d60`; the only change after the initial June artifacts was PR #198's CloudEvents `source`+`id` correction and regenerated review artifacts.
- Issues #14, #68, and #187 through #195 remained open. Issue #192 remained open even though PR #198 was merged August 17, 2026.
- CS-Go remained at `4a00aa6f`, with Part 3 updates in `913c112` contained in release `v1.0.4`.
- OSH PR #194 remained open and draft at `50774ec7`, six commits, eight files, +1,655/-30, with no review or issue comments.
- No state change required an upstream-register version increment.

### 3.3 Important Authority Boundaries

- The Part 3 `external-id` uses an `/IS/` path, but its explicit `swg-draft` metadata and working-branch location control maturity; the identifier does not make it an approved International Standard.
- Generated HTML/PDF demonstrate review output, not approval or completeness.
- A normative-looking `SHALL` in the pinned Part 3 draft is authoritative only for describing that exact draft profile. It is not a current Glaux or approved OGC obligation.
- Open issue recommendations explain likely change pressure. They do not override the checked-in draft until merged, and a merged draft change still does not equal publication.
- CS-Go release status increases implementation confidence, not standards authority. OSH PR status further limits production and interoperability confidence.

---

## 4. Part 3 Document Structure, Dependencies, and Completeness

### 4.1 Repository Inventory

The pinned `api/part3` tree contains 50 entries: 40 AsciiDoc files, three JSON examples, one HTML artifact, one PDF artifact, one Metanorma YAML file, one build script, one `.gitignore`, and two extensionless files. It contains **no AsyncAPI JSON/YAML document and no abstract-test-suite directory** [D,T].

The [source entry](https://github.com/opengeospatial/ogcapi-connected-systems/blob/c95c1d6003359d0883c4dc759d7a148ab115fdb1/api/part3/standard/23-003r0.adoc) wires front matter, scope, conformance, references, terms, conventions, overview, the three message classes, history, and bibliography. Its own maintenance comment says the MQTT file is a level-three stub with broken include paths, the ATS is absent, and examples/relationships annexes are missing. The MQTT include is commented out [D].

### 4.2 Dependency Chain

| Dependency | Draft role | Readiness implication |
|---|---|---|
| CSAPI Parts 1/2 | Resource model, canonical URLs, native encodings | Stable approved base |
| Draft OGC API Pub/Sub Part 1 | Inherited CloudEvents JSON payload class and common discovery direction | Draft-on-draft dependency; version/migration must be pinned |
| CloudEvents 1.0.2 and JSON format | Resource and Batch Event envelope | Stable, but Part 3's `parentId` spelling conflicts with it |
| MQTT 3.1.1/5.0 | Intended first transport | Standards are stable; CSAPI mapping is absent |
| AsyncAPI | Candidate/expected discovery description | Part 3 cites 2.6 informatively; common OGC draft/EDR alignment points to 3.0 |
| SensorML/SWE/OM/CMD/GeoJSON media types | Native Resource Data payloads and format-name examples | Encoding validity remains owned by Parts 1/2 and later Glaux schema work |

Draft OGC API Pub/Sub Part 1 itself identifies Draft stage, uses placeholder submission/approval/publication dates, and supplies the inherited `/req/message-payload-cloudevents-json` class. It also uses AsyncAPI 3.0 and registered discovery patterns. Part 3 therefore cannot safely treat its inherited baseline as immutable [D].

### 4.3 Claimed Versus Available Classes

| Claimed class | Wired? | Requirement substance | ATS | Readiness |
|---|---:|---|---:|---|
| `/req/resource-events` | Yes | Six requirement groups | No | Substantial but internally unsettled |
| `/req/batch-resource-events` | Yes | Six requirement groups | No | Substantial; semantics/name questioned in #193 |
| `/req/resource-data-messages` | Yes | Payload, deletion, allowed-type rules | No | Useful abstract payload boundary |
| `/req/resource-data-messages/content-negotiation` | Yes | One deterministic MIME-to-format-name rule | No | Useful but poorly named and binding-dependent |
| MQTT transport | No | Obsolete Part 2 identifier, broken dependencies, overview bullets, TODO | No | Not implementable as a shared binding |

The conformance clause says there is no Core class and an implementation is expected to implement at least one message category and at least one protocol binding. Because no protocol binding is wired, no implementation can claim compatibility with the complete pinned draft by following only its checked-in normative content [D,X].

### 4.4 Artifact Checks

- All three checked-in event example files parsed as JSON.
- SHA-256 hashes were recorded for the source, HTML, and PDF in Appendix B.
- The HTML visibly renders `[clause-protocol-mqtt]`, links to absent `annex_ats`, and contains the event-token and discovery TODOs.
- Four material TODO/missing locations remain in normative-adjacent source plus an unupdated README.
- The included artifacts were inspected, but the Metanorma build was not rerun because its Ruby toolchain was not present. This does not limit the source/include finding, which is explicit in the immutable entry file.

---

## 5. Protocol-Agnostic Message Model

### 5.1 Resource Events

The Resource Events class inherits draft OGC Pub/Sub CloudEvents JSON and defines server-originated lifecycle notices [D]:

- CloudEvents 1.0 JSON structured envelope.
- Non-empty `id`; `source`+`id` uniquely identifies a distinct event; retransmission may reuse the pair.
- `source` is the CSAPI root URL; `subject` is the canonical URL of the affected resource.
- Event types are `org.ogc.api.consys.<resource>.create|update|delete`.
- `data` is optional. If present, `datacontenttype` is `application/json`, and the object includes at least `name` and `description` where those properties exist.
- Only the hosting CSAPI server may publish; client publication to reserved channels must be rejected.
- Supported event types must be discoverable, but the discovery mechanism remains TODO.

The class has five consequential defects or ambiguities:

1. `/event-types` says every exposed resource type **shall** publish create/update/delete events, while `/all-resource-types` says the server **should** cover every type and **may** restrict types/operations if it declares the exact set. The discoverability mechanism needed to reconcile the restriction is unspecified.
2. The event-token mapping annex is TODO. Thirteen example tokens are listed, but the complete mapping is absent.
3. `parentId` is mixed-case. CloudEvents 1.0.2 requires every standard or extension attribute name to contain only lowercase ASCII letters or digits. Issue #194 proposes a descriptive lowercase replacement such as `resourceparent`.
4. The property table calls `parentId` recommended, while the requirement says it **shall** be present whenever the resource has a parent.
5. `time` may be visibility time, request receipt time, or processing time. That flexibility prevents cross-server latency or ordering comparisons unless an implementation declares its rule.

Issue #191 proposes changing the legacy `consys` event namespace to `csapi`; #195 proposes widening the optional payload from a small summary to omitted, summary, or complete JSON-compatible snapshot patterns. Both remain open [D].

### 5.2 Batch Resource Events

Batch Resource Events are aggregate lifecycle summaries, not the CloudEvents JSON Batch Format and not an atomic HTTP bulk transaction [D]:

- One CloudEvent summarizes exactly one resource type and one operation type.
- `subject` is a canonical nested collection URL.
- `data` is required and contains `timerange`—exactly two chronological RFC 3339 values—and non-negative `count`; unknown additional members must be ignored.
- The window describes when operations became visible to API clients.
- Only nested collections under one parent are eligible: observations, commands, command status, command result, and system events at the paths enumerated by the draft.
- Top-level and multi-parent collections are excluded.
- Only the server may publish; client publications must be rejected.

Material issues are:

- The allowed-collections requirement calls the canonical collection URL a `topic`, mixing protocol-neutral subject semantics with the absent binding.
- Regular Resource Events require a parent when one exists; Batch Events only recommend it.
- The allowed system-event path uses `systemEvents`, while approved CSAPI path/casing and implementation routes require careful reconciliation.
- Issue #193 questions the name and use case. Its proposed clarification—aggregate activity, not atomic batch—is analytically sound but not merged.
- No rule defines late arrivals, clock alignment, empty windows, crash recovery, maximum count, overflow, or whether window time is occurrence, commit, or visibility time beyond the stated visibility meaning.

### 5.3 Resource Data Messages

Resource Data Messages carry one **complete native CSAPI resource representation with no CloudEvents envelope**. Eligible types are Observation, Command, Command Status, Command Result, System Event, System, Datastream, and Controlstream. The class permits server-to-client and client-to-server flow but delegates publish authority to the missing protocol binding. Deletion must be conveyed only by a Resource Event [D].

This separation is one of the draft's strongest reusable choices. Lifecycle notices can remain small and routable; full observation, command, status, result, event, and registration messages retain their approved encoding. It also avoids mandatory envelope overhead for constrained devices. It does not, by itself, define validation, authorization, idempotency, replay, or commit semantics [D,R].

### 5.4 Optional Representation-Channel Naming

The optional subclass defines a deterministic normalized format name:

1. Remove media-type parameters.
2. Remove `application/` if present.
3. Replace `+` with `-`.
4. Lowercase the remainder.

Examples include `json`, `geo-json`, `sml-json`, `om-json`, `cmd-json`, `swe-json`, `swe-text`, and `swe-binary`. A protocol binding decides where that token appears. The class does not require publishing every supported encoding and does not define request-response negotiation; “representation advertisement and channel selection” is a more accurate description, consistent with issue #190 [D,X].

---

## 6. Protocol Binding, Topics, Discovery, AsyncAPI, and Conformance

### 6.1 MQTT Binding State

The MQTT file does not define a CSAPI binding. It contains an obsolete `ogcapi-connectedsystems-2` requirements URI, dependencies on missing clause anchors, six descriptive publish/subscribe bullets, “Use AsyncAPI,” and `TODO`. It defines none of the following [D]:

- MQTT 3.1.1 versus 5.0 required/optional support;
- broker URI discovery or topic namespace/prefix;
- resource, collection, message-class, or representation topic syntax;
- permitted publishers and subscribers by resource/channel;
- CloudEvents structured versus MQTT 5 binary mode;
- MQTT 5 Content Type, User Property, Response Topic, or Correlation Data behavior;
- QoS, retained messages, clean/persistent sessions, expiry, wills, reconnect, resubscription, duplicates, ordering, flow control, backpressure, or maximum packet size;
- authorization/ACL enforcement split;
- conformance tests.

CloudEvents' own MQTT 1.0.2 binding fills only part of this gap. It requires structured JSON mode for MQTT 3.1.1 and permits structured or binary mode for MQTT 5, but explicitly leaves topic choice and transfer/settlement constraints to the application. It contains no QoS or retain rule. It therefore cannot substitute for CSAPI issue #189 [P].

### 6.2 Discovery and AsyncAPI

The draft requires event-type discoverability but lists AsyncAPI, conformance tokens, or root/`/events` links only as candidates. There is no Part 3 AsyncAPI file. The bibliography cites AsyncAPI 2.6; draft common OGC Pub/Sub Part 1 and approved EDR Part 2 use AsyncAPI 3.0. Issue #14 recommends a validated Part 3 AsyncAPI 3.0 example and disposition of the legacy Part 2 file. Issue #68 records the shift toward the common OGC Pub/Sub baseline [D,P].

For Glaux, discovery must ultimately describe the **deployed** broker endpoints, channels, directions, payload schemas/media types, security schemes, supported event tokens, message-class profile, and version. A generated description should be tested against configured behavior and linked from the landing page. It must not expose credentials or claim channels disabled by feature or policy [R].

### 6.3 Conformance Boundary

Formal OGC conformance is unavailable for the pinned draft because:

- the document is not approved;
- the required protocol-binding half of its implementation expectation is absent;
- the normative ATS it invokes is absent;
- discovery and several event details remain unresolved;
- no executable OGC test suite exists for this branch.

A project may truthfully claim **compatibility with a named Glaux experimental profile based on exact draft commit `c95c1d60`**, with enumerated deviations. It may not call that profile approved OGC Part 3 conformance. Later publication and an applicable ATS would require a fresh delta review and migration/conformance run [R].

---

## 7. CS-Go Implementation Evidence

### 7.1 State and Architecture

CS-Go's initial experimental Pub/Sub work merged through PR [#16](https://github.com/SomethingCreativeStudios/connected-systems-go/pull/16) and tag `v1.0.2`; commit `913c112` substantially realigned it with the current Part 3 draft and is included in release `v1.0.4`. The pinned main commit is clean [I].

The implementation uses Eclipse Paho MQTT. MQTT is a master feature flag disabled by default. Resource Data, Resource Events, and Batch Resource Events have independent switches that default true but have no effect unless MQTT is enabled. Broker URL, client ID, username, password, QoS (default 1), retained flag (default false), and one-minute aggregate window are configurable [I].

### 7.2 Topics, Directions, and Discovery

CS-Go defines its own binding because the draft does not [I]:

| Message class | Topic pattern | Direction implemented |
|---|---|---|
| Observation data | `datastreams/{id}/observations:data` | Server publish and client ingest |
| Command data | `controlstreams/{id}/commands:data` | Server publish only |
| Command Status data | `commands/{id}/status:data` | Server publish and client ingest |
| System Event data | `systems/{id}/events:data`, `systemEvents:data` | Server publish only |
| Resource Event | collection and individual REST-relative paths plus `:events` | Server publish |
| Batch Event | observation/command collection path plus `:batch-events` | Server publish |

`GET /asyncapi` generates AsyncAPI 2.6 JSON with broker data, configured channels, operation direction, message schema, and `x-ogc-*` supported-token arrays. The landing page adds a `service-desc` link only when MQTT and at least one message class are active. Tests ensure discovery follows configuration and does not add a Part 3 conformance URI [I,T].

### 7.3 Publication and Ingestion

- Successful POST/PUT/DELETE operations are resolved after the response, except delete metadata is resolved before deletion. Non-2xx mutations do not emit events.
- Supported lifecycle types are system/subsystem, deployment, procedure, property, sampling feature, datastream, observation, controlstream, command, command status/result, and system event.
- Resource Events are emitted to collection and individual topics with `org.ogc.api.consys` types, absolute source/subject/parent URLs, and optional descriptive summary.
- When batching is enabled, Observation and Command lifecycle changes use aggregate events instead of individual events. Windows are UTC clock-aligned, keyed by collection/type/operation, and partial non-empty windows flush on graceful shutdown.
- Inbound Observation and Command Status data messages require exact topics and complete JSON resources, validate IDs/parent association and relevant schema, create or update persisted state, and then emit the corresponding lifecycle event.
- A 30-second topic+payload hash/count cache suppresses the local echo created when the server subscribes to a channel on which it also publishes [I].

### 7.4 Reliability and Security Limits

Paho is configured for clean sessions, automatic reconnect, retry, maximum reconnect interval, and configured QoS/retain. Publication is fire-and-forget: it is skipped when disconnected, and asynchronous failures are logged. There is no persistent event/outbox record tying the successful database operation to broker acknowledgement. A process or broker failure can therefore leave committed HTTP state without its event. The local echo cache is not a general duplicate detector and expires after 30 seconds [I,X].

Broker credentials are supported, but code does not establish broker ACLs, client-certificate identity, HTTP-to-MQTT subject mapping, resource-level subscription policy, or proof that clients cannot publish reserved event topics. The README explicitly says broker-side publisher restrictions cannot be verified by the API process [I].

### 7.5 Test Evidence

At the exact pin, 44 focused tests passed locally using checksum-verified Go 1.25.14 [T]:

- 21 `internal/pubsub` tests;
- 10 `internal/mqtt` tests;
- 9 focused `internal/api` discovery/system-event tests;
- 4 focused `internal/config` tests.

The tests cover event shape/topics, summaries, batch keys/windows/concurrency/shutdown, HTTP success/failure/delete timing, topic parsing, strict ingestion, echo suppression, configuration, discovery gating, and fixture agreement. No live broker probe ran because Docker and a broker executable were unavailable. The successful unit/integration-style results do not prove broker ACLs, reconnect loss behavior, QoS delivery, cross-process durability, or interoperability with OSH.

---

## 8. OpenSensorHub Implementation Evidence

### 8.1 State and Architecture

OSH `osh-addons` PR [#194](https://github.com/opensensorhub/osh-addons/pull/194), titled `[CSAPI] MQTT Module update to CS API Part 3 Draft`, is open, draft, cleanly mergeable, and unreleased. It contains six commits and no review. It extends an existing CSAPI MQTT-to-servlet connector rather than introducing the broker/server foundation from scratch [I].

The PR adds:

- CloudEvents type constants;
- a node/prefix-aware connector;
- topic and wildcard validation;
- subscription permission checks and client event-publication rejection;
- event-bus-driven system, datastream, controlstream, and observation publication;
- optional Resource Data format subtopics;
- observation aggregate windows.

### 8.2 Topics, Directions, and Formats

With the default `nodeId=api`, topics begin `api/`. Resource Event topics have no class suffix and usually follow nested OSH/CSAPI resource paths, for example `api/systems/{sys}/datastreams/{ds}` or `.../observations/{obs}`. Batch Observation Events use the observations collection topic without a distinct batch suffix. Resource Data topics end `:data` and may append `/json`, `/swe-json`, `/swe-binary`, `/swe-csv`, `/om-json`, or `/sml-json` [I].

The connector routes Resource Data subscription/publication through the existing CSAPI HTTP servlet and its request security checks. Wildcard Resource Data subscriptions are rejected. Event subscriptions permit `+` and `#` only in structurally valid resource-ID positions and map the shallowest matching resource type to a stream permission. Client publication to a topic without `:data` is rejected by the connector [I].

The format work follows the draft's general normalization concept, but `swe-csv` maps to SWE Text while the draft's media-type algorithm yields `swe-text`. That token is an implementation deviation, not an equivalent spelling under the current draft [D,I].

### 8.3 Event Publication and Batching

The publisher listens to OSH event-bus events. It emits create/update/delete for systems, datastreams, and controlstreams, plus non-draft `enable` and `disable` types. It publishes stream events to direct and ancestor-system topics. Observation creation events are not published individually; they are counted per datastream and periodically emitted as Batch Events [I].

The default aggregate interval is 30 seconds. The batch is Observation create only. Its `timerange` starts at the earliest observation phenomenon time and ends at the periodic flush wall time. The draft instead describes the interval during which resource operations became visible to API clients, so this is a semantic mismatch when phenomenon time differs from ingestion/visibility time. Pending aggregates are deliberately dropped on service stop [D,I,X].

Regular and Batch Events use CloudEvents 1.0 JSON and `org.ogc.api.consys` types. OSH correctly serializes the extension as lowercase `parentid`; that complies with CloudEvents naming but differs byte-for-byte from the Part 3 draft and CS-Go. Regular events carry no `data`, which the draft permits. Batch Events carry the required JSON summary [I].

### 8.4 Discovery, Reliability, and Test Evidence

No AsyncAPI generator or other Pub/Sub description artifact appears in the PR. Without an out-of-band topic contract, clients cannot discover the exact node prefix, hierarchy, formats, directions, event tokens, or permissions from this work [I].

Publication calls the in-process MQTT server directly and logs exceptions. The PR does not add a durable outbox, event replay, publisher acknowledgement contract, or failed-message recovery. Stop cancels subscriptions and drops the pending observation window. Lower-level MQTT QoS, retain, session, and authentication behavior is inherited from the existing OSH stack and is not newly specified or proven by this PR [I].

The branch contains twelve JUnit methods, all in `TestConSysTopicValidator`, covering exact/wildcard/invalid topics and format parsing. The GitHub `Gradle Build` check passed at the head commit, but the workflow command is `build -x test` with a TODO saying tests are broken. No publisher, batching, permission, broker, discovery, or cross-server test is checked in for this PR. Local execution was not attempted after the environment check found no Java runtime and the Windows checkout had unrelated long-path deletions; immutable code inspection and the no-test CI result bound the claims [I,T].

---

## 9. Cross-Implementation Comparison and Interoperability

### 9.1 Behavior Matrix

| Dimension | Pinned Part 3 | CS-Go `v1.0.4` | OSH PR #194 | Interoperability effect |
|---|---|---|---|---|
| Maturity | SWG draft | Merged/released experimental | Open draft PR | Neither proves OGC conformance |
| MQTT topic binding | Missing | Project-defined | Project-defined | No shared contract |
| Namespace prefix | Undefined | None | `nodeId/`, default `api/` | Exact subscriptions miss |
| Resource Event marker | Undefined | `:events` | None | Exact subscriptions miss |
| Batch marker | Undefined | `:batch-events` | None; collection topic | Message class cannot be selected the same way |
| Data marker | Example hints only | `:data` | `:data[/format]` | Bare JSON sometimes adaptable; format topics differ |
| Resource path shape | Canonical URL semantics, no topic syntax | Mostly flattened REST-relative paths | System-nested hierarchy for streams | Identical resource IDs yield different channels |
| Event namespace | `org.ogc.api.consys` | Same | Same, plus enable/disable | Core three agree; OSH extras require extension handling |
| Parent extension | Draft `parentId` | `parentId` | `parentid` | JSON field mismatch; draft spelling violates CloudEvents |
| Event `data` | Optional summary | Summary where metadata exists | Omitted | Both payload choices allowed, but consumers differ in available context |
| Event resource coverage | Ambiguous all/restricted | 13 tokens advertised | Publisher covers four families; validator recognizes more | Subscription expectations differ |
| Resource Data coverage | Any subset of 8 eligible types | 4 outbound; 2 inbound | Existing servlet paths, broader but not enumerated | Direction/resource capability discovery differs |
| Format selection | Normalization rule only | No format suffix | Six format suffixes | `swe-csv` conflicts with draft `swe-text` |
| AsyncAPI | Missing/TODO | Generated 2.6 | None | No common machine-readable contract |
| Batch resources | Five nested collection kinds eligible | Observation and Command | Observation only | Partial overlap |
| Batch operations | One of create/update/delete | All three | Create only | Partial overlap |
| Batch time | Visibility interval | Event-record clock window | Phenomenon time to flush | Counts may describe different intervals |
| Shutdown | Undefined | Flush partial non-empty window | Drop partial window | Different loss/summary behavior |
| Event publisher restriction | Server-only; reject clients | API cannot prove broker ACL | Connector rejects event publications | End-to-end authority not equivalent |
| Delivery/replay | Undefined | QoS configurable; volatile fire-and-forget | Inherited MQTT; volatile publisher | No durable common guarantee |
| Tests | ATS absent | 44 focused tests reproduced | 12 validator methods; CI skips tests | Unequal evidence depth |

### 9.2 Minimal Reproducible Incompatibilities

No broker is needed to demonstrate the primary mismatch; string equality and parser behavior are sufficient [T,X].

| Scenario | CS-Go channel | OSH channel | Result |
|---|---|---|---|
| Datastream Resource Event | `datastreams/ds-1:events` | `api/systems/sys-1/datastreams/ds-1` | Neither exact subscription receives the other |
| Observation Batch Event | `datastreams/ds-1/observations:batch-events` | `api/systems/sys-1/datastreams/ds-1/observations` | Prefix, hierarchy, and class marker differ |
| Observation data | `datastreams/ds-1/observations:data` | `api/systems/sys-1/datastreams/ds-1/observations:data` | Prefix/hierarchy differ |
| SWE JSON data | No described format-specific channel | `api/.../observations:data/swe-json` | CS-Go neither advertises nor parses OSH's format channel |
| Parent context | JSON `parentId` | JSON `parentid` | Case-sensitive consumers see different extension members |

Once an adapter rewrites topics and the parent attribute, the common create/update/delete CloudEvents payload core is close: both use structured JSON, version `1.0`, CSAPI root source, canonical resource/collection subject, UUID-compatible IDs, RFC 3339 time, and the legacy `consys` namespace. This makes an adapter/profile feasible, not optional [I,X].

### 9.3 Candidate Patterns and Anti-Patterns

**Safe candidates for later Glaux decisions [R]:**

- independent feature flags and disabled-by-default master switch;
- transport-neutral event construction before topic mapping;
- generated discovery from active configuration;
- exact resource-data validation through the same domain rules as HTTP;
- event identity distinct from resource identity;
- collection and individual subscription scopes represented as profile choices;
- aggregate state keyed by parent collection, resource type, operation, and declared window;
- broker adapter boundary and deterministic profile-versioned topics;
- explicit documentation that draft compatibility is experimental.

**Patterns to avoid [R]:**

- emitting after commit without a durable outbox or a documented best-effort contract;
- treating a local echo cache as idempotency/replay protection;
- making topic paths part of the domain model;
- using observation phenomenon time as the transaction visibility window;
- silently dropping a partial aggregate without a declared shutdown/failure policy;
- using event tokens outside the profile without advertising them;
- generating discovery that is not tested against enabled behavior;
- enforcing reserved publishers only in application code while leaving broker ACLs permissive;
- embedding credentials in advertised broker URLs;
- claiming conformance to an unwired draft/ATS.

---

## 10. Reliability, Transaction, Durability, Ordering, Replay, and DDIL

### 10.1 What the Draft Actually Guarantees

The draft defines event identity and permits the same `source`+`id` on retransmission. That supports duplicate recognition if a consumer retains identifiers. It does **not** define delivery level, ordering, replay, persistence, acknowledgement, subscriber checkpoints, expiry, backpressure, or how retransmission is requested. MQTT QoS does not repair application/database transaction gaps and does not create a replay contract [D,P].

### 10.2 Transaction Boundary

The critical server invariant is:

> A committed resource mutation and its publishable event must be derived from one authoritative state transition, with recovery able to detect and deliver committed-but-unpublished events without publishing rolled-back changes [R].

CS-Go gets one half right by publishing only after successful HTTP mutation and resolving delete context before deletion. It remains vulnerable between database commit and broker publication. OSH gets useful domain-event decoupling from its event bus but likewise shows no durable handoff. IDR-SRV-025 and 029 must evaluate a transactional outbox or equivalent atomic record; IDR-SRV-031/032 must define inbound data-message idempotency and publisher contracts; IDR-SRV-035 must decide delivery/retry architecture [I,R].

### 10.3 Ordering, Duplicates, and Causation

CloudEvents `id` identifies an event, not sequence. MQTT ordering is scoped and conditioned by connection, topic, QoS, and implementation behavior. Multi-topic publication of the same event—as CS-Go does for collection and individual scopes and OSH does for ancestor scopes—can legitimately create duplicates for wildcard consumers. A consumer needs a bounded deduplication policy based on `source`+`id`, and Glaux needs later decisions on [R]:

- per-resource version or sequence;
- causal/correlation identity for command, status, result, and derived events;
- whether identical event IDs are reused across fan-out topics;
- reordering tolerance and stale-update rejection;
- deduplication retention and collision behavior;
- observability/audit fields without inventing non-profile CloudEvents attributes.

### 10.4 Replay, Backfill, and Deletion

The HTTP API may recover current resource state after missed create/update notices, but it cannot reconstruct every transition and cannot dereference a deleted subject. Batch Events deliberately omit individual IDs. Production recovery therefore needs an explicit choice among durable event log, queryable notification collection, resource/version change feed, snapshot plus high-water mark, or a documented best-effort non-replay profile. Part 3 supplies none of those choices [D,R].

### 10.5 DDIL Implications

Native Resource Data Messages can reduce overhead, and aggregate events can reduce notification volume. Those are bandwidth optimizations, not DDIL semantics. Clean sessions, volatile aggregation, fire-and-forget publication, and no replay can make an intermittent subscriber permanently miss changes. Later DDIL work must define [R]:

- persistent versus ephemeral sessions and subscription restoration;
- message expiry, retained bootstrap state, and sensitive-data retention limits;
- snapshot/backfill and resume cursors;
- bounded offline queues, eviction, priority, compression, and load shedding;
- stale/fresh/unknown state and last-success metadata;
- clock uncertainty and late data versus transaction visibility;
- command authorization validity while disconnected;
- reconciliation and conflict handling after reconnect.

Part 3 should be one transport-facing view over these server semantics, not their sole storage or synchronization mechanism.

---

## 11. Security, Authorization, Publisher Authority, and Policy

### 11.1 Enforcement Layers

| Layer | Required responsibility | Evidence/current gap |
|---|---|---|
| API/domain process | Validate resource payload, lifecycle authority, command rights, parent ownership, and policy | CS-Go validates inbound Observation/Status but lacks a general MQTT actor-policy mapping |
| MQTT connector/gateway | Authenticate connection, map identity/claims, authorize channel/direction, reject malformed/oversized messages | OSH checks stream permissions and rejects event publication in connector; CS-Go does not expose equivalent enforcement |
| Broker | Enforce publish/subscribe ACLs, TLS, quotas, topic isolation, session policy, retain/expiry limits | Draft is silent; neither implementation proves deployed ACLs |
| Shared policy decision point | Apply resource/tenant/releasability/coalition rules consistently to HTTP and Pub/Sub | Future Glaux IDR-SRV-039/039A/040 decision |
| Audit/observability | Record authenticated actor, decision, event/resource ID, broker result, retry, drop, and policy version | Neither studied path shows complete end-to-end audit |

Application rejection is insufficient if a client can connect directly to a broker and publish to the same topic. Broker ACLs are insufficient if topic scope does not express the same resource visibility and command authority as the HTTP API. Glaux must test both layers as one policy surface [R].

### 11.2 Principal Threats

- Wildcard subscriptions can cross tenant, system, releasability, or need-to-know boundaries.
- Guessable canonical paths and CloudEvents `source`/`subject`/parent context can disclose resource existence even when payloads are concealed.
- Unauthorized Resource Data publication can create observations, commands, statuses, systems, or streams unless direction and actor authority are explicit.
- Retained messages and durable sessions can preserve sensitive data after policy changes or deletion.
- Broker compromise can forge, suppress, replay, reorder, or inspect events.
- AsyncAPI/broker discovery can leak internal hostnames, topic topology, security scheme details, or credentials.
- Ambiguous external origin can produce internal URLs in `source`/`subject`, breaking traversal and authorization scope.
- Oversized native, SensorML, SWE, or embedded snapshot payloads can exhaust broker, server, or constrained clients.
- Event summaries can leak names/descriptions even when the underlying resource would be denied.
- A command accepted over MQTT without the same feasibility/safety/authority checks as HTTP creates a control bypass.

CloudEvents recommends compact events, protocol-level security, and care with sensitive context attributes, but it does not define Glaux authorization or encryption. TLS, client identity, per-direction ACLs, quotas, and payload/schema validation must be explicit profile requirements, then exercised by IDR-SRV-055 [P,R].

### 11.3 Publisher and Subscription Policy Requirements for a Future Experiment

A Glaux experiment must not start unless it can [R]:

- bind broker identity to an auditable principal;
- deny all reserved server event topics to non-server publishers at the broker and gateway;
- authorize subscription at least as narrowly as the corresponding HTTP resource/query;
- authorize client Resource Data publication by resource type, parent, operation, and command role;
- re-evaluate long-lived subscriptions when policy changes, or bound their lifetime;
- avoid credentials in discovery and logs;
- disable retained sensitive payloads by default;
- impose packet, rate, wildcard, inflight, and queue limits;
- preserve concealed-existence behavior and label filtered/withheld states consistently;
- emit security/audit evidence without putting secrets into CloudEvents context.

---

## 12. Glaux Adoption Readiness

### 12.1 Decision Matrix

Scores are analytical, not standards measurements: 1 is weak and 5 is strong.

| Criterion | Score | Evidence | Effect |
|---|---:|---|---|
| Operational value | 5 | Reduces polling; supports lifecycle, live data, command/status, and aggregate notification | Strong reason to retain candidate |
| Abstract message-model clarity | 4 | Three distinct message purposes and useful canonical/CloudEvents rules | Safe to influence internal seams |
| Draft authority/stability | 2 | SWG draft with open issues and draft dependency | Exact pin and migration required |
| MQTT binding completeness | 1 | Unwired TODO stub | Cannot freeze external contract |
| Discovery/conformance readiness | 1 | No Part 3 AsyncAPI or ATS; unresolved discovery | No OGC claim; local artifact required |
| Implementation feasibility | 4 | Released CS-Go plus substantial OSH draft | Technically feasible |
| Independent interoperability | 1 | Material topic/field/time divergence; no broker cross-test | Adapter/profile required |
| Reliability/durability readiness | 1 | No outbox/replay/order/backpressure rules | Later architecture gate |
| Security/policy readiness | 2 | Some OSH checks; no complete broker/API policy model | Security gate required |
| Migration manageability | 3 | Adapter boundary can isolate churn; direct topic coupling cannot | Design boundary now |
| Testability | 3 | CS-Go unit evidence strong; standards ATS and cross-server tests absent | Build fixtures now; conformance later |

### 12.2 Recommendation

**Classification: Experimental candidate.**

This means [R]:

- Part 3 remains explicitly in Glaux plans and IDR-SRV-035's decision set.
- Intervening design topics must preserve the semantics and seams needed to implement it without topic-specific coupling.
- External Pub/Sub remains unimplemented until the project lead accepts later implementation planning after IDR-SRV-035.
- The final architecture may still choose `adopt`, `experimental profile`, `monitor`, `defer`, or `reject` based on the refreshed evidence.

“Monitor only” is too passive because event identity, transaction seams, data-message validation, policy, and fixtures affect the core design now. “Adopt now” is too strong because the binding, discovery, ATS, security, and interoperability are not ready. “Defer” would unnecessarily risk rework; “reject” is contradicted by two viable implementations and clear operational value.

### 12.3 Safeguards for Any Later Experimental Profile

| Safeguard | Minimum rule |
|---|---|
| Identifier | `glaux-part3-experimental` plus profile version and exact upstream commit |
| Default | Master feature disabled; each message class/direction separately gated |
| Authority statement | “Experimental compatibility profile”; no approved OGC Part 3 conformance claim |
| Architecture | Domain event/outbox record independent of MQTT and topic strings |
| Binding | Versioned transport adapter; all topics/directions/formats enumerated |
| Discovery | Generated AsyncAPI, preferably 3.0 after IDR-SRV-035 review, linked only when enabled |
| Deviations | Machine- and human-readable list for parent attribute, token namespace, topics, formats, batching, and delivery |
| Reliability | Declared best-effort or durable delivery; never imply guarantees from QoS alone |
| Security | Broker plus gateway authorization, no credential disclosure, quotas/limits, audit |
| Migration | Compatibility window, adapter/version negotiation, deprecation and removal rule |
| Tests | Golden messages/topics, broker contract, cross-version, negative ACL, reconnect/loss, and cross-implementation adapter tests |

### 12.4 Evidence Gates Before Stronger Claims

1. A complete binding is published or Glaux freezes an explicit local binding with no standards overstatement.
2. Parent extension naming, event namespace, discovery, AsyncAPI version, and content-channel semantics are resolved or listed as profile deviations.
3. Mutation/event atomicity and failed-publication recovery are designed and tested.
4. HTTP and Pub/Sub identity, authorization, releasability, and command authority are equivalent and broker-enforced.
5. Reconnect, duplication, ordering, replay/backfill, retention, expiry, and shutdown behavior are documented and tested.
6. CS-Go/OSH compatibility is either demonstrated or bounded through a tested adapter matrix.
7. Formal OGC conformance is claimed only after an approved standard, applicable ATS, and passing conformance evidence exist.

---

## 13. Downstream Handoffs and IDR-SRV-035 Decision Backlog

### 13.1 Early Handoff Matrix

| Owner | Required input from IDR-SRV-014H |
|---|---|
| 015 Canonical Resource Model | Keep transport-neutral entity/event types; distinguish resource, collection, parent, and message identity |
| 016 Identifier/URI/Lifecycle | Canonical public URL, event `source`+`id`, resource version, parent semantics, deletion context, token naming |
| 018 Temporal/Freshness | Separate occurrence, phenomenon, request, commit, visibility, publication, delivery, and observation times |
| 020 Status/System Event | Distinguish domain System Events from lifecycle Resource Events and non-draft enable/disable extensions |
| 023 Schema/Encoding | Complete native data payload validation, CloudEvents schema, media/format normalization, size limits, golden corpus |
| 025 Persistence | Durable event/outbox, aggregate state, retention, replay/snapshot storage options |
| 029 Transaction/Consistency | Mutation+event atomicity, idempotency, duplicate/reorder handling, aggregate concurrency |
| 031 Write/Ingestion | Direction/type authorization, complete versus partial resource data, validation parity with HTTP |
| 032 Publisher Contract | Broker actor identity, parent/topic agreement, publish acknowledgement, retry/error contract |
| 034 Dynamic Updates | Observation/command/status/result event triggers, batch timing, late arrival, status causation |
| 039/039A/040 | Broker/API policy equivalence, wildcard scope, tenant/releasability, command authority, concealed existence |
| 041 | Actor, decision, source/id, resource/version, broker result, retry/drop, and correlation audit fields |
| 042/043 | Offline queues, resume/replay, snapshots, freshness, expiry, reconciliation, conflicts |
| 045-049 | Adapter modularity, broker deployment, config/secrets, metrics/health, migration/backup of durable event state |
| 050/051 | Draft-profile versus approved-conformance harness and requirement-to-test traceability |
| 052/053 | Multi-layer test seams; golden topics/messages and incompatibility fixtures |
| 054 | Rate, fan-out, wildcard, batch, backpressure, queue, payload-size, reconnect, and DDIL load scenarios |
| 055 | Negative publish/subscribe ACL, cross-tenant, retained-data, spoof/replay, oversized-message, and command tests |
| 056 | CS-Go/OSH/Glaux profile and adapter interoperability matrix |

### 13.2 Questions Reserved for IDR-SRV-035

IDR-SRV-035 must refresh the draft and implementation pins, then decide [R]:

- whether Glaux implements Part 3 at all and at what lifecycle stage;
- approved-only versus draft experimental support;
- transport-neutral event architecture and durable publication mechanism;
- MQTT and/or other transport selection and Rust ecosystem fit;
- exact topic hierarchy, namespace, collection/individual fan-out, wildcard, and representation-channel syntax;
- AsyncAPI version, generation source, media types, landing/broker links, and security schemes;
- message classes and per-resource publish/subscribe directions;
- CloudEvents structured/binary modes and final parent/event-token names;
- summary versus full snapshot policy and payload confidentiality;
- aggregate event inclusion, name, resources, operations, windows, and crash behavior;
- QoS, retain, session, expiry, replay, ordering, deduplication, backpressure, and delivery claim;
- broker topology, ACL enforcement, identity federation, and public origin;
- experimental profile identifier, deviations, version negotiation, migration, deprecation, and conformance transition;
- prototype and evidence required before implementation authorization.

IDR-SRV-035 must not merely choose between CS-Go and OSH. Their divergence is evidence that Glaux needs an explicit profile decision.

---

## 14. Recommendations

1. **R-014H-01 — Retain Part 3 as an experimental candidate.** Priority: High. Carry it into IDR-SRV-035 and the later implementation guide; do not call it adopted yet.
2. **R-014H-02 — Shape internal semantics now, external topics later.** Priority: High. Preserve canonical resource/parent references, lifecycle operations, event identity, native data messages, aggregate summaries, and a transport adapter boundary.
3. **R-014H-03 — Require an outbox-capable transaction seam.** Priority: High. IDR-SRV-025/029 must prevent committed mutations from silently losing their corresponding publishable event.
4. **R-014H-04 — Keep every experimental capability disabled by default and independently gated.** Priority: High. Gate transport, message class, resource type, direction, encoding, and batch behavior.
5. **R-014H-05 — Use generated, configuration-true discovery.** Priority: High. Prefer AsyncAPI 3.0 only after the later alignment review; test its channels, operations, messages, schemas, broker URLs, and security against runtime configuration.
6. **R-014H-06 — Never copy draft `parentId` into Glaux's internal model.** Priority: High. Use a semantic internal parent reference; let a profile adapter emit the selected external spelling after issue #194/draft refresh.
7. **R-014H-07 — Treat `consys`, `csapi`, enable/disable, and format tokens as versioned vocabulary.** Priority: Medium. No unadvertised extra event tokens or aliases.
8. **R-014H-08 — Define time explicitly.** Priority: High. Aggregate visibility windows must not be inferred from phenomenon time; later profiles must state every timestamp's meaning.
9. **R-014H-09 — Enforce publisher/subscriber authority in both broker and service.** Priority: High. Make Pub/Sub policy no weaker than HTTP and include command safety/authority.
10. **R-014H-10 — Build a golden interoperability corpus before a broker prototype.** Priority: High. Include draft examples, CS-Go and OSH topics, both parent spellings, format tokens, batch timing, unknown extensions, duplicates, deletion, and negative ACL cases.
11. **R-014H-11 — Separate delivery claims from MQTT QoS.** Priority: High. Document database-to-broker and broker-to-consumer guarantees, replay, expiry, and failure behavior end-to-end.
12. **R-014H-12 — Monitor material upstream changes, not every commit.** Priority: Medium. Reopen the decision baseline on binding/ATS/AsyncAPI publication, resolution of #187-#195, OSH merge/release, CS-Go profile change, or Part 3 publication.

---

## 15. Risks, Open Questions, and Monitoring Triggers

### 15.1 Risk Register

| Risk | Likelihood / impact | Current control | Later owner |
|---|---|---|---|
| Mutable draft breaks a frozen topic contract | High / High | Exact pin; adapter; versioned experimental profile | 035, 049 |
| Implementation precedent mistaken for normative text | High / High | Evidence labels and deviation matrix | 035, 051 |
| Mutation commits but event is lost | High / High | Require transaction/outbox research | 025, 029, 035 |
| Wildcard or topic guessing leaks protected resources | High / High | Broker+service policy gate | 039-040, 055 |
| Unauthorized data publication bypasses command/write controls | Medium / Critical | Direction and actor authorization | 031-033, 038-040, 055 |
| Retained/queued data outlives policy or deletion | Medium / High | Retain off by default; expiry/retention design | 040, 042, 046-049 |
| Duplicate fan-out or reconnect causes repeated effects | High / High | `source`+`id`, idempotent consumer contract | 029, 035, 043 |
| Batch interval misrepresents resource visibility | High / Medium | Explicit time model and golden fixtures | 018, 034, 053 |
| Discovery overstates enabled or authorized channels | Medium / High | Generated configuration-true AsyncAPI tests | 035, 047, 050-052 |
| Draft `parentId` produces invalid CloudEvents | High / Medium | Semantic internal field; adapter spelling | 016, 035 |
| AsyncAPI 2.6/3.0 drift breaks tooling | Medium / Medium | Defer version selection; validate artifact | 023, 035, 050 |
| “DDIL support” inferred from MQTT or aggregation | High / High | Explicitly separate bandwidth from recovery | 042-043, 054 |
| Large native/snapshot messages exhaust constrained nodes | Medium / High | Limits, representation policy, load/security tests | 023, 047, 054-055 |

### 15.2 Open Questions

- Will Part 3 adopt common OGC Pub/Sub Part 1 as its controlling draft baseline and AsyncAPI 3.0?
- What exact lowercase name and semantics will replace `parentId`, if retained?
- Will the event namespace become `org.ogc.api.csapi`?
- Will aggregate events remain, be renamed, or move to an optional class?
- Will complete JSON resource snapshots be permitted inside Resource Events?
- Which resource types/directions will the first MQTT binding authorize?
- What topic hierarchy, prefix, suffix, wildcard, format, and discovery rules will issue #189 produce?
- Will QoS, retain, session, expiry, replay, and broker ACL requirements enter Part 3 or remain deployment-profile concerns?
- Will the normative ATS and a validated AsyncAPI artifact arrive before Glaux reaches IDR-SRV-035?
- Will OSH PR #194 merge and release, and will CS-Go migrate from its current 2.6/topic profile?

### 15.3 Material Refresh Triggers

Refresh this baseline, rather than merely noting a commit, when any of the following occurs:

- MQTT is wired into the official Part 3 source or its normative requirements materially change;
- a Part 3 ATS or AsyncAPI artifact is added;
- draft OGC API Pub/Sub Part 1 changes the inherited class or is approved;
- #14, #68, or #187-#195 receives an SWG decision/closure with merged text;
- `part3-working-draft` changes `parentId`, event tokens, discovery, batch semantics, payloads, formats, or conformance;
- OSH PR #194 merges/closes or a release contains it;
- CS-Go changes its topics, AsyncAPI version, delivery, authorization, or conformance posture;
- an independent CS-Go/OSH/other Part 3 interoperability result is published;
- Part 3 enters public review, adoption vote, or final publication.

---

## 16. Validation Against the Research Plan

| Success criterion | Result | Evidence |
|---|---|---|
| Exact draft snapshot, branch, status, artifacts, dependencies, date | Pass | Metadata; §§3-4; Appendix B |
| Every in-scope class and material TODO/stub/missing artifact explicit | Pass | §§4-6; Appendix A |
| Event, batch, data, format, discovery, binding, security, conformance separate | Pass | §§5-6, 10-11 |
| Official issue/PR state refreshed with authority preserved | Pass | §§3, 15; register 1.9 unchanged because state did not change |
| CS-Go and OSH exact pins, code/docs/tests/config/merge/release state | Pass | §§7-8 |
| Field/behavior comparison with reproducible anchors | Pass | §9; Appendix B |
| Tests/probes executed where practical; limits labelled | Pass | §§7.5, 8.4; Appendix B |
| Draft, implementation, test, inference, recommendation distinct | Pass | Reading Guide and labels throughout |
| Security, durability, transaction, DDIL, operations bounded/handed off | Pass | §§10-11, 13 |
| Explicit experimental/monitor/defer recommendation with gates | Pass | §12 |
| IDR-SRV-035 backlog precise without preemption | Pass | §13.2 |
| References, commands, commits, states, and limitations reproducible | Pass | §17; Appendix B |

The plan's phases 1 through 6, deliverable drafting, and report review are complete. Plan-owner acceptance remains pending.

---

## 17. References

### 17.1 Controlling and Draft Standards

- [OGC API - Connected Systems - Part 1: Feature Resources](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data](https://docs.ogc.org/is/23-002/23-002.html)
- [Official Part 3 tree at `c95c1d60`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/c95c1d6003359d0883c4dc759d7a148ab115fdb1/api/part3)
- [Part 3 source entry](https://github.com/opengeospatial/ogcapi-connected-systems/blob/c95c1d6003359d0883c4dc759d7a148ab115fdb1/api/part3/standard/23-003r0.adoc)
- [Part 3 conformance clause](https://github.com/opengeospatial/ogcapi-connected-systems/blob/c95c1d6003359d0883c4dc759d7a148ab115fdb1/api/part3/standard/sections/clause_2_conformance.adoc)
- [Draft OGC API - Publish-Subscribe Workflow - Part 1: Core, OGC 25-030](https://docs.ogc.org/DRAFTS/25-030.html)
- [OGC API - EDR - Part 2: Publish-Subscribe Workflow](https://docs.ogc.org/is/23-057r1/23-057r1.html)
- [OGC Discussion Paper 23-013](https://docs.ogc.org/dp/23-013.html)
- [CloudEvents 1.0.2](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/spec.md)
- [CloudEvents JSON Event Format 1.0.2](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/formats/json-format.md)
- [CloudEvents MQTT Protocol Binding 1.0.2](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/bindings/mqtt-protocol-binding.md)
- [MQTT 3.1.1](https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/mqtt-v3.1.1.html)
- [MQTT 5.0](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html)
- [AsyncAPI 2.6.0](https://www.asyncapi.com/docs/reference/specification/v2.6.0)
- [AsyncAPI 3.0.0](https://www.asyncapi.com/docs/reference/specification/v3.0.0)

### 17.2 Official Maintenance Evidence

- [Issue #14: AsyncAPI example](https://github.com/opengeospatial/ogcapi-connected-systems/issues/14)
- [Issue #68: common OGC Pub/Sub alignment](https://github.com/opengeospatial/ogcapi-connected-systems/issues/68)
- Part 3 issue cluster: [#187](https://github.com/opengeospatial/ogcapi-connected-systems/issues/187), [#188](https://github.com/opengeospatial/ogcapi-connected-systems/issues/188), [#189](https://github.com/opengeospatial/ogcapi-connected-systems/issues/189), [#190](https://github.com/opengeospatial/ogcapi-connected-systems/issues/190), [#191](https://github.com/opengeospatial/ogcapi-connected-systems/issues/191), [#192](https://github.com/opengeospatial/ogcapi-connected-systems/issues/192), [#193](https://github.com/opengeospatial/ogcapi-connected-systems/issues/193), [#194](https://github.com/opengeospatial/ogcapi-connected-systems/issues/194), and [#195](https://github.com/opengeospatial/ogcapi-connected-systems/issues/195)
- [PR #198: CloudEvent id alignment](https://github.com/opengeospatial/ogcapi-connected-systems/pull/198)
- [Glaux upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)

### 17.3 Implementation Evidence

- [CS-Go repository at `4a00aa6f`](https://github.com/SomethingCreativeStudios/connected-systems-go/tree/4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952)
- [CS-Go Pub/Sub documentation](https://github.com/SomethingCreativeStudios/connected-systems-go/blob/4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952/README.md#mqtt-publishsubscribe)
- [CS-Go publisher](https://github.com/SomethingCreativeStudios/connected-systems-go/blob/4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952/internal/pubsub/publisher.go)
- [CS-Go MQTT manager](https://github.com/SomethingCreativeStudios/connected-systems-go/blob/4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952/internal/mqtt/client.go)
- [CS-Go ingestion](https://github.com/SomethingCreativeStudios/connected-systems-go/blob/4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952/internal/mqtt/ingestion.go)
- [CS-Go AsyncAPI generator](https://github.com/SomethingCreativeStudios/connected-systems-go/blob/4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952/internal/api/asyncapi_handler.go)
- [OSH PR #194](https://github.com/opensensorhub/osh-addons/pull/194)
- [OSH topic validator](https://github.com/opensensorhub/osh-addons/blob/50774ec7e9c98f6ab8da827171e5c5abb9923a49/services/sensorhub-service-consys-mqtt/src/main/java/org/sensorhub/impl/service/consys/mqtt/ConSysTopicValidator.java)
- [OSH connector](https://github.com/opensensorhub/osh-addons/blob/50774ec7e9c98f6ab8da827171e5c5abb9923a49/services/sensorhub-service-consys-mqtt/src/main/java/org/sensorhub/impl/service/consys/mqtt/ConSysApiMqttConnector.java)
- [OSH event publisher](https://github.com/opensensorhub/osh-addons/blob/50774ec7e9c98f6ab8da827171e5c5abb9923a49/services/sensorhub-service-consys-mqtt/src/main/java/org/sensorhub/impl/service/consys/mqtt/ResourceEventPublisher.java)
- [OSH validator tests](https://github.com/opensensorhub/osh-addons/blob/50774ec7e9c98f6ab8da827171e5c5abb9923a49/services/sensorhub-service-consys-mqtt/src/test/java/org/sensorhub/impl/service/consys/mqtt/TestConSysTopicValidator.java)
- [OSH successful build job](https://github.com/opensensorhub/osh-addons/actions/runs/26847598379/job/79171162119)

---

## Appendix A. Draft Requirement and Completeness Matrix

| Requirement / artifact | Draft statement | Completeness or conflict | Candidate Glaux treatment | Later owner |
|---|---|---|---|---|
| `/req/resource-events/cloudevents-format` | CloudEvents 1.0 JSON | Complete at envelope level | Golden schema/message fixtures | 023, 053 |
| `/event-properties` identity | Non-empty `id`; unique with source; retransmit may reuse | Aligned by PR #198 | Durable event ID and duplicate policy | 016, 029 |
| `source` | CSAPI root URL | Public-origin behavior unstated | Canonical external origin | 016, 046-047 |
| `subject` | Canonical affected resource URL | Deletion dereference/replay unresolved | Preserve tombstone context | 016, 025, 029 |
| `parentId` | Parent canonical URL | Invalid CloudEvents casing; table/requirement strength conflict | Internal semantic parent; adapter later | 016, 035 |
| `time` | Operation/visibility/request/processing timestamp | Multiple meanings allowed | Explicit timestamp model | 018 |
| optional `data` | JSON summary with name/description where defined | Issue #195 proposes snapshots; confidentiality unresolved | Profile-select summary/snapshot/none | 023, 035, 040 |
| event types | `consys` + resource + create/update/delete | Mapping annex TODO; naming #191 | Versioned vocabulary | 020, 035 |
| publisher restriction | Server only; reject clients | Enforcement layer unspecified | Broker and gateway deny | 039-040, 055 |
| all resource types | SHALL versus SHOULD/MAY restriction | Internal tension | Enumerated deployed capability | 035, 050-051 |
| event discoverability | Exact supported set discoverable | Mechanism TODO | Generated AsyncAPI/profile document | 035, 047, 050 |
| Batch envelope | CloudEvents 1.0 JSON | Substantial | Optional aggregate type | 020, 035 |
| Batch subject | One nested canonical collection | Allowed rule calls it `topic` | Domain collection ref; adapter topic | 015-016, 035 |
| Batch parent | SHOULD parent canonical URL | Casing and strength unsettled | Same semantic parent model | 016, 035 |
| Batch data | Required timerange/count | Late/crash/overflow rules absent | Precise aggregate contract | 018, 025, 034-035 |
| Batch collections | Five enumerated nested paths | Casing/path review needed | Profile enumeration only | 015, 034-035 |
| Batch operation/type | Exactly one resource and operation | Clear | Key aggregates accordingly | 029, 034 |
| Resource Data payload | Complete native single resource | Clear abstract boundary | Same validation as HTTP | 023, 031 |
| Resource Data eligible types | Eight types, implementations may expose subset | Direction authority missing | Explicit per-type/direction matrix | 031-035 |
| deletion | Never Resource Data; use Resource Event | Clear | Preserve lifecycle event/tombstone | 016, 029 |
| format-name | Deterministic MIME normalization | Channel placement absent | Reuse only after binding decision | 023, 035 |
| MQTT class | Intended 3.1.1/5.0 transport | Unwired broken stub/TODO | Do not treat implementation topics as requirement | 035 |
| AsyncAPI | Mentioned/candidate; 2.6 bibliography | No artifact; 3.0 alignment pressure | Later generated validated artifact | 035, 050 |
| ATS | Conformance says normative Annex A | Absent | No conformance claim | 050-051 |
| Examples | Three JSON event examples | Parse; no data-message/topic/security examples | Expand golden corpus | 053 |

---

## Appendix B. Reproducible Commands and Test Record

### B.1 Pins and Inventory

```powershell
git -C $official fetch origin part3-working-draft
git -C $official rev-parse origin/part3-working-draft
git -C $official ls-tree -r --name-only c95c1d6003359d0883c4dc759d7a148ab115fdb1 api/part3
git -C $csgo rev-parse HEAD
git -C $osh rev-parse HEAD
```

Expected pins:

```text
official c95c1d6003359d0883c4dc759d7a148ab115fdb1
CS-Go   4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952
OSH     50774ec7e9c98f6ab8da827171e5c5abb9923a49
```

### B.2 Official Draft Mechanical Checks

```powershell
rg -n '#TODO#|TODO|Still missing|missing' "$official/api/part3/standard" --glob '*.adoc'
rg -n 'clause-protocol-mqtt|annex_ats|TODO' "$official/api/part3/standard/23-003r0.html"
Get-ChildItem "$official/api/part3/examples/events/*.json" |
  ForEach-Object { Get-Content $_.FullName -Raw | ConvertFrom-Json | Out-Null }
Get-FileHash -Algorithm SHA256 `
  "$official/api/part3/standard/23-003r0.adoc", `
  "$official/api/part3/standard/23-003r0.html", `
  "$official/api/part3/standard/23-003r0.pdf"
```

Results: three JSON examples passed. Artifact hashes:

```text
c76dc642728802287003f178b2525456743c89ff9ba3753f4bbed196f099db69  23-003r0.adoc
5cee2587a0a8f55e781c54c2c0c61698d917ea89e44e231bfc9d49ae8cc5e602  23-003r0.html
bf85a8787d11b8ef6858d8328412266a652d118b6f3651619fc921f68af50e74  23-003r0.pdf
```

### B.3 CS-Go Test Execution

The official Windows Go 1.25.14 archive was downloaded from `go.dev`, and its SHA-256 matched the published value `119044a92b3987c341cd6aebb256676dd4780d292f7b4e72a3e9976677841697` before extraction to a temporary evidence directory.

```powershell
$env:GOCACHE = '<temporary-evidence-cache>'
$env:GOMODCACHE = '<temporary-evidence-module-cache>'
go test ./internal/pubsub ./internal/mqtt -count=1
go test ./internal/api `
  -run 'Test(AsyncAPI|PubSub|LandingPageLinks|CheckedInAsyncAPI|SystemEventLifecycle)' `
  -count=1
go test ./internal/config -run 'Test(PubSub|BatchResourceEvent)' -count=1
```

Results:

```text
PASS internal/pubsub  21 tests
PASS internal/mqtt   10 tests
PASS internal/api     9 focused tests
PASS internal/config  4 focused tests
TOTAL                 44 tests
```

No live broker probe ran: `docker`, a broker executable, and a system Go installation were initially absent. The portable Go toolchain enabled source tests; it did not supply a broker or prove network policy/delivery behavior.

### B.4 OSH Evidence Limits

```powershell
git -C $osh diff --stat 18a2d858...50774ec7
rg -c '@Test' TestConSysTopicValidator.java
```

Results: 8 files, +1,655/-30; 12 validator test methods. GitHub check run `79171162119` passed, but the workflow's exact Gradle argument is `build -x test`. No test-pass claim is made. Java was unavailable locally, and the checkout had unrelated Windows long-path deletions; no branch modification occurred.

### B.5 Interoperability Fixture Minimum

A future machine-readable fixture should contain at least:

```yaml
resource_event:
  csgo_topic: datastreams/ds-1:events
  osh_topic: api/systems/sys-1/datastreams/ds-1
  csgo_parent_attribute: parentId
  osh_parent_attribute: parentid
batch_observation:
  csgo_topic: datastreams/ds-1/observations:batch-events
  osh_topic: api/systems/sys-1/datastreams/ds-1/observations
resource_data_swe_json:
  csgo_topic: datastreams/ds-1/observations:data
  osh_topic: api/systems/sys-1/datastreams/ds-1/observations:data/swe-json
```

The fixture records observed incompatibility; it does not define Glaux's future binding.

---

## Appendix C. Detailed Question Coverage and Completion

| Plan question group | Answer location | Completion |
|---|---|---|
| Authority, version, branch, commit, history, artifacts, status | §§3-4; Appendix B | Complete |
| Approved/draft/informative dependencies and precedence | §§3.1, 4.2 | Complete |
| Resource Event version, mode, fields, identity, source/subject/type/time/data/parent | §5.1; Appendix A | Complete |
| Resource coverage, operation vocabulary, publisher authority, discovery | §§5.1, 6.2 | Complete |
| Batch semantics, collections, window/count, purpose, differences | §5.2 | Complete |
| Resource Data types, complete payload, direction, deletion, encodings | §5.3 | Complete |
| Format-name derivation and channel responsibility | §5.4 | Complete |
| MQTT 3.1.1/5, topics, QoS, retain, session, correlation, ordering/backpressure | §§6.1, 10 | Complete; missing draft rules explicit |
| AsyncAPI, broker/channel/message/security discovery | §6.2 | Complete |
| Requirements/conformance classes and ATS availability | §§4.3, 6.3; Appendix A | Complete |
| CS-Go connectivity, flags, publication, ingestion, batching, discovery, shutdown | §7 | Complete |
| CS-Go durability, error/retry, echo, QoS/retain, authorization, tests | §§7.4-7.5, 10-11 | Complete |
| OSH PR state, event bus, topics, wildcards, data routing, formats, batching | §8 | Complete |
| OSH publisher authority, permissions, discovery, inherited behavior, tests | §§8.2-8.4 | Complete within available evidence |
| Field-by-field agreement/divergence and causes | §9.1 | Complete |
| Direct interoperability and minimal incompatibility | §9.2; Appendix B.5 | Complete |
| Safe patterns, anti-patterns, prototype needs | §§9.3, 12 | Complete |
| Delivery, durability, transaction, duplication, ordering, replay, DDIL | §10 | Complete; later decisions bounded |
| Authorization split, policy/releasability, audit, wildcard/tenant threats | §11 | Complete; later decisions bounded |
| Internal model stability and feature/profile safeguards | §12 | Complete |
| Compatibility/conformance evidence threshold | §§6.3, 12.4 | Complete |
| Downstream handoffs | §13.1 | Complete |
| Questions retained for IDR-SRV-035 | §13.2 | Complete |
| Upstream monitoring and local decision boundary | §15 | Complete |

**Completion state:** Research and report review complete; report in review; plan-owner acceptance and authorization of the next topic remain separate future actions.

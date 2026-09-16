# Section 043: Server Synchronization and Conflict Handling Boundary - Research Report

**Topic ID:** IDR-SRV-043<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-043 Server Synchronization and Conflict Handling Boundary](../IDR%20Plans/idr-srv-043-server-synchronization-and-conflict-handling-boundary.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All questions concerning server synchronization scope, scenarios, resource classes, authority and identity, replay, duplicate and conflict detection, state and gap handling, tombstones, resolution, quarantine, policy, trust, commands, audit, API behavior, events, observability, and downstream verification<br>
**Methodology Used:** Current primary-source freeze; accepted-baseline reconciliation; scenario/resource inventory; authority and invariant analysis; receive-pipeline and state-machine design; conflict/resolution threat analysis; implementation-lesson comparison; fixture and handoff traceability<br>
**Research Time:** Approximately 34 hours of AI-assisted execution on September 15, 2026<br>
**Approved Standards Baseline:** OGC 23-001 and OGC 23-002 Version 1.0; SensorML 3.0; SWE Common 3.0; accepted Glaux IDR-SRV-001 through IDR-SRV-042; controlled AEP-4789 package `AC/224(JCGISR)D(2026)0005` subject to its recorded status and handling limits<br>
**Protocol Guidance Baseline:** RFC 9110, RFC 9111, RFC 6585, RFC 9457, RFC 3339, MQTT 5.0, CloudEvents 1.0.2, and the experimental Part 3 profile decision accepted in IDR-SRV-035, checked September 15, 2026<br>
**Implementation Evidence:** Official CSAPI tag `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2` and repository head `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`; CloudEvents stable tag at `fc1f6f31f5f011a72183f1bcea20c987cb683ade`; accepted OSH, CS-Go, pygeoapi, SECD, client-smoke, interoperability, discussion, and Part 3 studies used only as non-normative evidence<br>
**Supporting Resources:** PostgreSQL 18 current documentation; the primary CRDT paper; accepted identity, temporal, provenance, transaction, lifecycle, ingestion, dynamic-data, streaming, command, security, policy, audit, and DDIL reports; upstream-history register Version 1.12<br>
**Document Purpose:** Define the server-owned synchronization and conflict-handling contract without selecting enterprise replication topology, cross-domain transfer architecture, database replication layout, broker product, or deployment shape<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 15, 2026<br>
**Date:** September 15, 2026<br>
**Last Updated:** September 15, 2026

---

## Evidence and Decision Legend

| Mark | Meaning |
|---|---|
| N | Normative published specification requirement within its scope |
| C | Controlled AEP/project-source finding, limited to accessible evidence and handling rules |
| A | Accepted Glaux design baseline |
| I | Pinned implementation, test, or community evidence; not a requirement |
| E | Engineering, safety, security, or interoperability inference |
| P | Proposed Glaux decision requiring this report's acceptance |
| X | Open parameter, limitation, or downstream decision |

The published CSAPI standards define resources, representations and API behavior but do not define a general multi-node synchronization protocol, causal merge scheme, conflict service, or exactly-once exchange. Consequently, the synchronization envelope, receive pipeline and conflict records below are Glaux profile decisions. Database replication, MQTT delivery and CloudEvents identity are useful mechanisms only inside their stated scopes. **[N/A/E/P]**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Synchronization/Conflict Extraction Methodology
5. Server Synchronization Scope Boundary
6. Synchronization Scenario Taxonomy
7. Synchronizable Resource and Record Inventory
8. Authority, Identifier, Versioning, Tombstone, Supersession, and Idempotency Findings
9. Synchronization State Model Findings
10. Replay, Duplicate Detection, Delayed Update, and Event-Order Findings
11. Conflict Taxonomy and Conflict Detection Findings
12. Conflict Record, Quarantine, Operator Review, and Resolution Strategy Findings
13. Source Trust, Provenance, Policy/Releasability, and Federation Findings
14. Command Lifecycle, Command-Status Reconciliation, Unknown Outcome, and Command Audit Findings
15. Audit Synchronization, Audit Gap, and Accountability Findings
16. API Response, Error, Event, and Observability Implications
17. Fixture, Conformance, Security Testing, Performance, Deployment, and Interoperability Test Implications
18. Downstream Topic Handoff Matrix
19. Recommendations
20. Risks, Constraints, and Open Questions
21. Validation Against This Plan's Success Criteria
22. References

---

## 1. Executive Summary

Glaux should synchronize **immutable evidence and explicit intentions**, then derive local authorized projections; it should not copy a peer's mutable “current row” and call that convergence. Each candidate arrives in a versioned `SyncEnvelopeV1` with stable identity, origin and source authority, revision/causality, all relevant clocks, schema/profile pins, canonical digest, provenance, policy/trust evidence, lifecycle/tombstone state and range position. A staged receiver authenticates, authorizes, validates, deduplicates, classifies, commits and publishes the result. Every stage is locally atomic and auditable. **[A/E/P]**

The server owns domain-aware exchange admission, inbox/idempotency, validation, conflict detection, quarantine, resolution evidence, local projections, safe API results and audit. It does not own WAN routing, broker clustering, PostgreSQL physical/logical replication, cross-domain transfer, peer discovery, PKI issuance, enterprise federation governance, or complete deployment topology. Those services may transport or replicate bytes, but they cannot decide source authority, policy, command safety, canonical identity, tombstone precedence or which evidence is current. **[A/E/P]**

The contract is **at-least-once transport with effectively-once local application**, not exactly once. An accepted candidate and its inbox outcome commit together. Re-delivery of the same scoped identity and digest returns the stored outcome without another domain effect. Reuse of an identity with different canonical bytes is an integrity conflict. A message hash alone cannot prove domain duplication, and a transport acknowledgement cannot prove validation, commit, command execution or peer convergence. **[N/A/E/P]**

Automatic action is an allowlist: exact replay deduplication; compatible immutable-fact addition; profile-defined monotonic update from one authoritative source; and a mathematically specified commutative merge whose invariant, policy and removal semantics have been proven for that exact field. CRDT theory provides sufficient convergence conditions for suitable data types; it is not permission to apply a generic last-writer or set union to identity, relationships, schemas, validity, policy, commands, audit, deletion or authority. All other material divergence is rejected, quarantined, retained as parallel evidence, or placed in operator review. **[E/P]**

Conflict is not a synonym for every non-acceptance. Invalid, unauthorized, untrusted, incomplete and temporarily unverifiable candidates have separate outcomes. A conflict exists when individually interpretable claims cannot jointly satisfy a protected invariant or named selection rule. The server records both sides and the detection evidence without changing canonical state. Resolution is a new authorized, provenance-bearing and auditable activity; it never rewrites the contenders or pretends that arrival order, node clock, UUID magnitude, central-node preference, broker order or guessed “most restrictive” vocabulary established truth. **[A/P]**

Commands and audit use stricter subcontracts. A delayed command-status report may enrich history only after stable command/attempt identity, authorized reporter, legal transition, fence, sequence and terminality checks. Possible physical effect remains `unknown` or `reconciliation_required` until verified; synchronization never retries a non-idempotent action merely to obtain certainty. Audit records retain original node/epoch/sequence/time/integrity evidence, and accepted transfer creates separate receipt/apply audit evidence. Gaps remain explicit. **[A/P]**

Normal CSAPI clients do not receive the internal synchronization protocol. Existing resource APIs expose only authorized canonical or qualified views. Admin-only synchronization operations may return durable status resources and protected conflict references. HTTP `412` remains a failed conditional request, `428` a required missing precondition, and `409` a resolvable current-state or synchronization conflict. RFC 9457 details are stable and corrective but conceal topology, hidden sources, policy labels, command capability and competing protected values. **[N/A/P]**

Acceptance of this report completes Category G. It does not select deployment topology, replication product, conflict UI, retention numbers, cross-domain solution, or Category H architecture. Those decisions remain downstream and unauthorized until this report is accepted. **[P]**

---

## 2. Scope and Plan Alignment

### 2.1 Included scope

This report defines server behavior for source/adapter replay, local-node reconnect, edge-to-reference exchange, reference-to-edge distribution, peer/federated import, delayed Observations and status, metadata revision exchange, event backfill, command-status reconciliation, audit upload, policy/trust/schema package delivery, gaps, tombstones, conflicts and operator decisions. It covers the authoritative application boundary from candidate receipt through durable outcome and local projection. **[P]**

### 2.2 Excluded scope

It does not design:

- physical or logical database topology, multi-primary SQL replication or disaster-recovery layout;
- broker clustering, WAN routing, radio protocols, store-and-forward network architecture or peer discovery;
- cross-domain guards, release authority workflows, PKI/identity issuance or enterprise federation agreements;
- exact deployment roles, regions, node counts, bandwidth plans, retention numbers or conflict user interface;
- a new public CSAPI conformance class or an assertion that draft Part 3 defines synchronization; or
- server implementation, schema migration, production key management or live command enablement.

### 2.3 Plan-question coverage

| Plan concern | Report location |
|---|---|
| standards and synchronization baseline | Sections 3–5 |
| scenarios and server ownership | Sections 5–6 |
| resource/record classification | Section 7 |
| authority, identity, versions, tombstones and idempotency | Section 8 |
| states, replay, offsets, gaps and delayed input | Sections 9–10 |
| conflict detection, records, quarantine and resolution | Sections 11–12 |
| trust, policy, federation and provenance | Section 13 |
| command and audit reconciliation | Sections 14–15 |
| API, events, observability and verification | Sections 16–17 |
| decisions, risks and downstream ownership | Sections 18–21 |

### 2.4 Accepted-baseline reconciliation

This report preserves the IDR-SRV-042 dimensions and outcomes rather than creating one global “synced” flag. It adopts IDR-SRV-029 inbox/outbox and idempotency rules; IDR-SRV-016 identity/revision rules; IDR-SRV-018 time separation; IDR-SRV-019 provenance; IDR-SRV-020 status/event boundaries; IDR-SRV-023 validation/quarantine; IDR-SRV-030 lifecycle/disposition; IDR-SRV-035 durable publication; IDR-SRV-036 through 038 command rules; IDR-SRV-039/039A security and enforcement; IDR-SRV-040 authorized-view policy; and IDR-SRV-041 audit continuity. Where an infrastructure mechanism is weaker, the accepted application contract controls. **[A/P]**

---

## 3. Evidence Base and Authority Classification

### 3.1 Source inventory

| Source | Frozen version/status | Authority and use | Limitation |
|---|---|---|---|
| CSAPI Part 1, OGC 23-001 | Version 1.0; official tag `v1.0.0` | normative resource/API baseline | no generic node synchronization protocol |
| CSAPI Part 2, OGC 23-002 | Version 1.0 | normative dynamic-data and command baseline | no causal merge/conflict service |
| SensorML 3.0 / SWE Common 3.0 | published OGC standards | normative document/data-contract meaning | not replication protocols |
| RFC 9110 / 9111 / 6585 / 9457 / 3339 | published RFCs | HTTP validators, status, caching, problems and timestamps | not domain conflict arbitration |
| MQTT 5.0 | OASIS Standard | transport/session/QoS evidence | packet identity and QoS are not domain identity/effect |
| CloudEvents | 1.0.2 stable tag `fc1f6f31...` | event envelope identity guidance | no domain order, merge or authorization rule |
| PostgreSQL 18 current docs | checked 2026-09-15 | MVCC, transactions and logical-replication behavior | storage/deployment mechanism, not Glaux semantics |
| Shapiro et al., CRDTs | 2011 primary paper | sufficient convergence conditions | model assumptions and per-type proof required |
| Controlled AEP package | `AC/224(JCGISR)D(2026)0005`, 2026-04-27 | controlled profile context | pre-promulgation; handling limits; no public reproduction |
| accepted IDR-SRV-001–042 | accepted project baseline | controlling project decisions | subordinate to published standards in their scope |
| implementation studies 014A–014H | pinned, accepted studies | non-normative lessons and fixtures | implementations do not create requirements |

### 3.2 Primary-source findings

RFC 9110 defines `If-Match` to prevent lost updates and maps a false precondition to `412`; RFC 6585 defines `428` when the server requires a conditional request. RFC 9110 `409` describes a conflict with current target state that the user might resolve. These status codes support online concurrency and administrative sync operations, but do not decide the winning domain claim. **[N/E]**

CloudEvents requires uniqueness of `source` plus `id` for distinct events and permits the same pair for retransmission. That pair is useful event-level duplicate evidence, not proof that two payloads, domain resources, operations or physical effects are equivalent. Glaux therefore binds it to digest, type, schema and domain identity before deduplication. **[N/E/P]**

MQTT QoS 1 is at least once; overlapping subscriptions may yield further copies; packet identifiers can be reused after acknowledgement; and application meaning remains outside the protocol. MQTT QoS 2's transport phrase “exactly once” is not an end-to-end Glaux effect claim. Logical application identifiers and inbox commits remain mandatory. **[N/A/E]**

PostgreSQL logical replication documents concrete storage conflicts and restrictions, including manually resolved uniqueness conflicts and table-oriented replication limits. It can support deployment replication but cannot preserve Glaux policy, provenance, validation, command and audit decisions by itself. PostgreSQL transactions remain the local atomicity mechanism for inbox outcome plus domain effect. **[E/P]**

The CRDT paper proves convergence only under explicit algebraic and delivery conditions such as monotonic semilattices or commuting operations. Glaux may use such a type only after an invariant-specific proof and threat review. Generic object merge, timestamp LWW and unqualified union are rejected defaults. **[E/P]**

### 3.3 Evidence gaps and controlled material

No approved source reviewed establishes a universal NATO/OGC server-to-server synchronization wire protocol for these resources. The controlled AEP source is recorded at the document/package level and informs bounded responsibilities, but its non-public contents are neither reproduced nor treated as a public interoperability contract. Exact exchange protocol, topology and partner agreements remain profile/deployment work. **[C/X]**

---

## 4. Synchronization/Conflict Extraction Methodology

The analysis used six passes:

1. **Freeze and classify evidence.** Separate normative specifications, controlled sources, accepted Glaux decisions, implementation observations and engineering proposals.
2. **Enumerate exchange scenarios.** Identify producer, receiver, direction, outage/replay behavior, transported object and expected authority.
3. **Classify every object.** Record mutability, identity, revision, ordering, provenance, policy, validation, deletion and projection behavior.
4. **Model the receive pipeline.** Define checks and durable outcomes without assuming transport reliability or arrival order.
5. **Attack protected invariants.** Test divergent identity, parentage, contract, authority, policy, command, audit, tombstone and time cases; allow automation only when safety is proven.
6. **Trace verification and ownership.** Convert every decision into fixture, metric, downstream topic or explicit open parameter.

The decision rubric asks, in order: can the peer and package be authenticated; is transfer and use authorized; are schema/profile/encoding known; is identity stable; are origin and authority credible; is the candidate exact replay, causal successor, concurrent branch or gap; can both claims coexist; is a defined merge mathematically and semantically safe; can the local effect commit atomically with its outcome; and what may be exposed to this caller? A failure at one gate cannot be repaired by a later timestamp or preference. **[A/E/P]**

---

## 5. Server Synchronization Scope Boundary

### 5.1 Server-owned responsibilities

Glaux Server owns:

- authenticated and authorized synchronization endpoints or adapter boundary;
- package/envelope size, media, signature/integrity, schema and profile checks;
- stable peer/source mapping and bounded synchronization grants;
- durable receive session, manifest/range and candidate records;
- inbox/idempotency and canonical-digest comparison;
- resource-specific validation, authority, causality and conflict classification;
- quarantine, protected conflict record and review/resolution workflow;
- locally atomic application and projection update;
- tombstone/supersession enforcement and anti-resurrection checks;
- durable outcome receipt, watermarks/gaps and safe retry/resume;
- local audit, provenance, outbox and observability evidence; and
- policy-filtered public/admin representation of state and problems.

### 5.2 Infrastructure/external responsibilities

| Concern | Primary owner | Glaux boundary |
|---|---|---|
| network routing, link scheduling, compression | deployment/network | accept bounded transport; do not infer domain order |
| broker durability, cluster replication, sessions | broker operator | retain logical IDs and application replay contract |
| PostgreSQL physical/logical replication | database/deployment | never use it as domain conflict authority |
| cross-domain inspection/release | approved CDS/guard | require an authorized released package; do not implement guard |
| PKI issuance and enterprise identity lifecycle | identity/PKI | validate mapped credentials and record evidence |
| federation agreements and source ownership | governance/deployment | enforce installed versioned mappings |
| peer discovery and route selection | deployment/federation service | accept only configured trusted peer identities |
| long-haul bulk transfer/object delivery | transfer service | verify manifest, digest, policy and availability locally |
| conflict user experience | Category H/product | expose protected workflow/status contract only |

### 5.3 Non-bypass invariant

Synchronization is another write path. It passes the same or stricter authentication, authorization, source trust, policy/releasability, validation, command-safety, transaction, provenance and audit gates as an online request. A trusted network, signed bundle, central sender, broker ACL, database replica or administrator transport role does not automatically confer resource authority or release permission. **[A/P]**

---

## 6. Synchronization Scenario Taxonomy

| Scenario | Direction / trigger | Server outcome and boundary |
|---|---|---|
| adapter redelivery | source to same node after timeout/crash | inbox dedupe or conflict; no second effect |
| source offset replay/backfill | source to node by range | validate original identities; fill history/gaps; do not relabel time |
| edge reconnect upload | local/edge to reference | manifest negotiation, policy/trust checks, candidate outcomes per item/range |
| reference distribution | reference to edge | versioned packages/resources; local authority and queued effects re-evaluated |
| bidirectional metadata exchange | node to node | causal/revision comparison; safe successors apply, branches conflict |
| delayed Observation batch | source/edge to receiver | valid historical append; current changes only under accepted selector |
| delayed status/latest evidence | source/edge to receiver | append evidence; derive current by authority, sequence and domain time |
| event replay/backfill | durable log to consumer/node | preserve event ID/position; tolerate boundary duplicate; gap remains explicit |
| local resource creation | edge to reference | accept only scoped local authority; detect canonical identity/parent conflicts |
| tombstone/disposition exchange | either direction | preserve deletion authority/hold/effective revision; prevent stale resurrection |
| schema/profile/vocabulary package | authority to node | verify issuer, version, dependencies, activation and anti-rollback |
| source/trust/policy bundle update | authority to node | stage and validate; tightening/revocation triggers re-evaluation; no silent widening |
| command-status reconciliation | target/gateway/edge to node | verify command attempt/fence/reporter and legal transition; never reissue effect |
| command intent/queue handoff | node to authorized dispatcher | durable intent and exact grant; pre-effect recheck; not generic resource merge |
| audit range upload | edge to audit receiver | verify manifest/checkpoints/gaps; retain original sequence and separate receipt |
| federated CSAPI import | external server to node | profile-mapped import; remote canonical URL is evidence, not automatic local authority |
| experimental Part 3 delivery | publisher/broker to subscriber | delivery projection only; version-pinned adapter; inbox and native resource semantics control |
| snapshot/resnapshot | server to client/peer | authorized snapshot plus watermark; does not erase history or hidden gaps |

“Push,” “pull,” “bidirectional,” “replay,” “backfill,” “distribution,” and “federation” describe different contracts. A deployment must name the scenario and authority model; enabling a generic `/sync` route with unspecified semantics is prohibited. **[P]**

---

## 7. Synchronizable Resource and Record Inventory

| Resource/record family | Canonical behavior | Authority/mutability | Synchronization rule |
|---|---|---|---|
| Systems, Deployments, Procedures, Sampling Features, Properties | stable resource plus immutable revisions | named owner(s); mutable by revision | exchange revisions/relations; concurrent protected-field edits conflict |
| DataStreams and ControlStreams | versioned semantic contract | contract authority; constrained mutation | never rewrite contract beneath children; new version/supersession |
| SensorML documents and SWE structures | exact artifact plus parsed/canonical views | issuer/source and schema pinned | content-address exact bytes; validate local derived view |
| relationships | typed assertion/retraction history | relation-specific authority | additive only if cardinality/cycle/policy invariants remain true |
| Observations | append-oriented facts/corrections | source/publisher authority | preserve IDs, domain time and contract; delayed facts can enrich history |
| status values | append evidence plus current projection | capability-specific reporter authority | never arrival-wins; terminal/monotonic rules where defined |
| System Events | immutable modeled occurrence | event source authority | dedupe by scoped event identity/digest; not every sync action is System Event |
| source registrations/trust | versioned security evidence | security administrator/issuer | stage, anti-rollback, effective-time and revocation checks |
| policy/binding records | immutable versioned evidence | policy authority | no semantic merge; explicit activation/supersession and fail-closed conflict |
| Commands | immutable identity and intent revisions | requester plus command authority | no generic bidirectional merge; idempotency and attempt identity mandatory |
| feasibility results | advisory immutable evidence | named evaluator | retain input/evidence/expiry; never refresh merely by transfer |
| CommandStatus/results | append authoritative evidence | target/gateway/reporter scoped | legal transition/fence/terminal checks; unknown outcome preserved |
| audit records/checkpoints | append-only evidence | originating audit authority | preserve bytes/order/integrity/gaps; no overwrite or renumbering |
| validation artifacts | immutable evidence tied to input/rules | validator identity/version | transfer or recompute distinctly; never claim same result under new rules |
| raw payload/artifact references | content-addressed immutable object | source plus custody | verify digest, availability, policy and reference safety |
| schema/profile/vocabulary cache | immutable signed/versioned package | designated issuer | install/activate separately; protect from rollback/substitution |
| OpenAPI/conformance state | generated/versioned service declaration | local deployment authority | distribute build artifact only; local advertised capability remains local truth |
| inbox/outbox/delivery attempts | local control evidence | local node only | not federated domain truth; export only for diagnostics/audit |
| projections, extents, latest, search indexes, caches | rebuildable derived state | local computation | normally recompute; exchange only as explicitly non-authoritative optimization |

Cacheability does not imply synchronizability, and synchronizability does not imply write authority. Local database rows, queue leases, HTTP cache entries and metrics normally remain node-local even when the underlying immutable domain evidence is exchanged. **[A/P]**

---

## 8. Authority, Identifier, Versioning, Tombstone, Supersession, and Idempotency Findings

### 8.1 `SyncEnvelopeV1`

Every candidate or range item must bind, directly or through an integrity-protected manifest:

| Field group | Minimum content |
|---|---|
| envelope | envelope ID/version, scenario/profile, direction, sender peer, receive session |
| domain identity | resource/record type, stable ID/UID/canonical URL where applicable, parent/stream/command identity |
| revision/causality | immutable revision ID, base/predecessor, origin node, node/boot/source epoch, source sequence/offset, dependencies |
| content | media/encoding, schema/profile/vocabulary versions, exact-byte digest where required, canonical semantic digest |
| time | occurrence/valid/phenomenon/result/report times as applicable, origin receipt/commit, transfer and local receipt; confidence |
| authority | source/publisher/executor/reporter, authority grant/class, security domain, delegation and federation mapping |
| policy/trust | policy binding/decision, releasability transform, credential/trust/bundle epochs, activation/expiry |
| lifecycle | active/tentative/tombstoned/superseded/disposition state, effective time, hold and authority |
| operation | message/event/operation/idempotency key, command attempt/fence, correlation/causation |
| range | manifest/range ID, partition, start/end watermark, known gaps, count/size, checkpoint/root/digest |
| provenance/audit | source artifact/activity references, original signature/integrity evidence, audit correlation |

The envelope is not automatically a public representation and does not alter native CSAPI content. It is a private/profile exchange contract whose sensitive fields are policy protected. **[P]**

### 8.2 Identity and revision rules

- Resource ID identifies the continuing resource; revision ID identifies immutable state; event/message/operation IDs identify different objects.
- External canonical URLs and UIDs are aliases/evidence until accepted mapping proves their local identity scope.
- Source offset and transport packet ID are progress evidence, never ResourceIds.
- UUID generation time and lexical order do not determine domain precedence.
- Same scoped message/event/operation ID plus same canonical fingerprint can deduplicate; the same ID plus different bytes is a collision/conflict.
- Two distinct sources producing equal bytes remain distinct provenance unless a profile proves they report the same immutable occurrence.

### 8.3 Authority and versioning

Authority is action- and field-specific: create, revise, relate, report, correct, tombstone, release, dispatch and resolve can have different principals. A server may accept a remote fact into history without granting it authority over the local current projection. Revision ancestry is a partial order; source sequence is meaningful only inside its declared source/epoch/domain. **[A/P]**

### 8.4 Tombstones and supersession

A tombstone is an authoritative lifecycle assertion, not physical absence. It records target identity/revision, action, issuer/authority, effective/decision/commit time, reason class, policy/hold, predecessor and provenance. It remains until every supported replay horizon and retention/legal constraint proves stale resurrection impossible; Category H/I must choose numbers. Concurrent update versus tombstone is always a protected conflict unless one causally follows the other under the same recognized authority and profile. Supersession preserves both records and their relation; it does not overwrite history. **[A/P/X]**

### 8.5 Idempotency

The inbox key includes security domain, sender/peer or source, scenario/profile, operation/message identity and applicable epoch. The record stores fingerprint, state, result, local IDs/revisions, policy/trust evidence, timestamps and retry disposition. The inbox row and accepted/quarantined/rejected/conflict effect commit atomically. Retention spans the maximum authorized offline, redelivery, replay, command and audit horizon. Reuse after expiry is not safely assumed to be a retry and requires profile-defined treatment. **[A/P/X]**

---

## 9. Synchronization State Model Findings

### 9.1 Session, candidate and local-record states

One state flag cannot represent synchronization. Glaux uses orthogonal state machines:

| Layer | States |
|---|---|
| session/range | proposed, negotiating, authorized, transferring, applying, paused, completed-with-watermark, completed-with-gap, failed-retryable, failed-terminal, expired |
| candidate processing | received, integrity-verified, authenticated, authorized, schema-validated, semantically-validated, classified, applying, applied, rejected, quarantined, conflicted, pending-review, resolved |
| durable outcome | identical-deduplicated, accepted-history, accepted-current, tentative, quarantined, conflict, gap, rejected-policy, rejected-trust, unverifiable |
| local record | unsynchronized, queued, transmitted, acknowledged-by-transport, received-by-peer, accepted-by-peer, rejected-by-peer, quarantined-by-peer, conflicted, superseded, tombstoned |
| continuity | complete-to-watermark, known-gap, unknown-gap, compacted/resnapshot-required, policy-filtered, unverifiable |

Transport acknowledgement is deliberately not `accepted-by-peer`. A session can be complete while individual items are rejected or hidden, provided the manifest and per-item/range outcome truthfully record that fact. “Synchronized” is used only with a named peer, security/policy scope, profile, direction and watermark. **[P]**

### 9.2 Receive/apply pipeline

1. Bound size/rate and create a correlation/session record without parsing untrusted bulk content into logs.
2. Authenticate transport peer and envelope signer; map to configured peer/source identity.
3. Authorize scenario, direction, resource scope, action, range and policy transfer.
4. Verify manifest/signature/digests, package completeness and anti-rollback evidence.
5. Resolve pinned schema/profile/vocabulary/encoding without unapproved network retrieval.
6. Check inbox identity; exact prior result replays, mismatched fingerprint conflicts.
7. Validate syntax, semantics, references, source authority, policy and trust.
8. Compare identity, revision/causality, tombstone, command fence, sequence and local invariant state.
9. Classify as exact replay, compatible history/current, tentative, gap, reject, quarantine or conflict.
10. Atomically persist candidate evidence, inbox outcome, domain effect/projection change, provenance, audit and outbox.
11. Return/store a policy-safe durable outcome and advance only the exact eligible watermark.

Crashes before commit permit retry; crashes after commit cause redelivery and stored-result replay. External transport acknowledgement occurs only after the durable local outcome or according to a protocol mapping that cannot lose the candidate. **[A/P]**

### 9.3 Watermarks and gaps

A watermark is scoped to peer/source, epoch, partition/range, profile and authorization view. It says what position was processed under that scope, not that every conceivable domain record exists or was accepted-current. Gaps carry bounded/unknown status, reason class, start/end when safe, detection time, affected profile and recovery action. Policy-hidden records advance only an opaque authorized cursor; their count or identity is not leaked. **[A/P]**

---

## 10. Replay, Duplicate Detection, Delayed Update, and Event-Order Findings

### 10.1 Classification rules

| Input condition | Required result |
|---|---|
| same scoped identity, digest and contract | `identical-deduplicated`; stored result; no new effect |
| same identity, different bytes/semantic digest | integrity/identity conflict; quarantine; protected alert |
| different identity, equal payload | retain separately unless occurrence equivalence is proven |
| valid older fact | `accepted-history`; current projection normally unchanged |
| valid eligible successor from authoritative source | `accepted-current`; atomic projection/event update |
| causally concurrent compatible immutable additions | accept both only if invariants and policy permit |
| missing predecessor/offset | hold/quarantine or accept bounded history with explicit gap per profile |
| epoch rollback/reused sequence | reject or quarantine; never order solely by sequence |
| schema/profile unavailable | `unverifiable`/quarantine; no best-effort reinterpretation |
| candidate outside authority or policy | `rejected-trust`/`rejected-policy`; conceal safely |
| tombstoned target and stale update | conflict/anti-resurrection; no recreation |

### 10.2 Ordering

Glaux promises order only where the domain defines it: immutable revision ancestry; a trusted source sequence within source and epoch; CommandStatus transition rules; audit node/epoch sequence; and durable event-log position within a declared partition. Occurrence, phenomenon, result, report, receipt, commit and synchronization times remain distinct. Cross-source and cross-partition total order is not promised. **[A/P]**

Broker delivery order, MQTT packet order, database commit sequence, UUID order and wall-clock timestamp do not provide universal causal order. Lamport/vector/dotted version metadata or Merkle/range structures may be adopted downstream, but their representation must preserve the same authority and policy rules. A vector clock can identify concurrency; it cannot decide which claim is authorized or safe. **[E/P/X]**

### 10.3 Replay and event publication

Backfill reuses the original domain/event identity and adds a new synchronization activity/receipt. Accepted historical enrichment may emit a distinct `history-added` or projection-change delivery event under the accepted streaming profile; it must not impersonate the original occurrence. Exact replay produces no second logical change event. Conflict/quarantine administrative notices are protected operational events, not automatically public CSAPI System Events. **[A/P]**

---

## 11. Conflict Taxonomy and Conflict Detection Findings

### 11.1 Conflict definition

A conflict is a durable finding that two or more interpretable claims cannot jointly satisfy a protected identity, authority, relationship, contract, temporal, lifecycle, policy, command, audit or projection invariant. Invalid syntax, unknown schema, missing dependency, unauthorized source and temporary processing failure are distinct findings even when they also require quarantine. **[P]**

### 11.2 Taxonomy

| Code family | Examples | Detection evidence | Default |
|---|---|---|---|
| identity | same ID/different content; UID maps to two resources; alias collision | scoped IDs, digest, identity map | quarantine/review |
| revision/causality | two successors to same base; missing predecessor; epoch rollback | base/revision graph, source epoch/sequence | branch/gap; review if material |
| authority | competing owners/reporters; action outside grant | authority mapping/grant/effective time | reject or quarantine; no preference by node |
| relationship | parent divergence, cycle, cardinality violation | typed graph and constraints | conflict; preserve contenders |
| contract/schema | stream contract changed with children; incompatible profile/version | contract pins, child existence, validator | reject/conflict; new version required |
| temporal/current | same source/order position differs; tied candidates under undefined rule | domain times, sequence, selector policy | conflict/unknown current |
| observation/correction | same occurrence identity differs; correction lineage diverges | observation IDs, source, contract, correction relation | retain branches/quarantine |
| tombstone/update | stale update resurrects; concurrent delete and edit | tombstone revision/authority/causality | conflict; tombstone not erased |
| policy/releasability | incompatible labels/bindings/transforms; stale policy | policy package/binding/decision | fail closed; authority review |
| trust/security | signer/source mismatch; revoked or rollback package | trust epoch, credential, signature, revocation | reject/quarantine/incident |
| command lifecycle | illegal transition; attempt/fence mismatch; terminal divergence | command ID/revision, status sequence, reporter, fence | quarantine/reconcile; no redispatch |
| audit integrity | same audit ID different bytes; sequence/checkpoint mismatch | event digest, node/epoch/sequence, chain/checkpoint | quarantine and explicit gap |
| federation mapping | same remote URL maps inconsistently; profile semantics disagree | mapping version and peer contract | quarantine/review |

### 11.3 Detection invariants

Detection compares canonical semantic content only after exact source bytes and interpretation version are retained. It evaluates authorization and policy before revealing local competitors. A candidate cannot overwrite the evidence used to classify it. If required identity, authority, schema, predecessor, policy or trust evidence is absent, the receiver records `unverifiable` or a gap—not a fabricated conflict winner. **[A/P]**

### 11.4 Synchronization and conflict matrix

| Scenario | Resource/record type | Authority | Identifier/idempotency | State | Conflict type | Detection evidence | Resolution | Quarantine/review | Policy/security | Command/control | Audit/event | Test/conformance | Downstream | Notes/open issue |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| adapter redelivery | any ingest fact | mapped source | source+epoch+message ID+digest | duplicate/apply | ID reuse | inbox/fingerprint | replay stored outcome | quarantine if bytes differ | auth before disclosure | no effect replay | dedupe audit only | crash-window test | 045/050 | retention numeric later |
| source backfill | Observations/events | source authority | original fact/event ID; run ID separate | history/gap/current | sequence/contract | offset, epoch, contract | fill history; selector recompute | gap/invalid quarantine | range and content policy | none | backfill activity | reordered/gap fixtures | 048/053/054 | no arrival-wins |
| edge reconnect | mixed manifest | scoped local authority | manifest/range/item IDs | receive/classify | mixed | manifest, grants, revisions | per-item atomic outcomes | protected review | reauthorize every range | queued commands isolated | receipt/apply audit | partial/resume test | 045/046/049 | transport design later |
| metadata exchange | System/etc. revisions | field/action owner | resource+revision+base | successor/branch | concurrent edit | ancestry, ETag/base, authority | apply successor or explicit resolution revision | review protected fields | authorized view only | ControlStream stricter | revision event after commit | branch fixtures | 045/050/056 | no generic merge |
| relationship exchange | association facts | relation authority | assertion/retraction ID | accepted/conflict | cycle/cardinality | graph snapshot, causal refs | accept compatible addition or review | review on invariant | hidden endpoints concealed | target binding relevant | provenance/audit | graph invariant tests | 045/053 | CRDT union not default |
| delayed Observation | Observation | publisher/source | Observation ID + occurrence/source | history/current | identity/correction | times, contract, digest | retain; named selector | differing same ID quarantined | row/field policy first | may affect feasibility only after reevaluation | projection-change event | temporal corpus | 053/054/056 | latest != arrival |
| delayed status | status evidence | named reporter | status record/source sequence | history/current | latest/terminal | reporter, report time, sequence | transition/selector algorithm | illegal transition quarantine | message redaction | command rules if CommandStatus | status audit/event | tie/order tests | 050/053 | no terminal overwrite |
| event replay | delivery event | event source | CloudEvents source+id+digest; log cursor | replay/gap | event ID reuse | log position, CE fields, digest | dedupe/replay/resnapshot | collision quarantine | cursor policy bound | command event not command effect | protected gap notice | boundary duplicate tests | 048/050 | no global order |
| local create upload | metadata resource | explicit local grant | client/resource/idempotency IDs | tentative/accepted | canonical ID/parent | authority grant, identity map | accept, remap only by explicit process, or conflict | canonical conflict review | local policy evidence retained | no implicit target authority | creation/sync audit | collision fixtures | 045/046 | remap relation must persist |
| tombstone exchange | lifecycle record | disposition authority | tombstone ID+target revision | tombstoned/conflict | delete/update | causality, holds, issuer | causal apply or review | always review concurrent material update | conceal existence/reason | command deletion prohibited | surviving marker | resurrection tests | 049/053 | horizon later |
| schema package | schema/profile | configured issuer | package/version/digest | staged/active | rollback/substitution | signature, deps, epoch | activate explicit compatible version | quarantine invalid/older | protect packages/keys | command contract pinned | install audit | offline package tests | 045/049/050 | no network ref fetch |
| policy/trust update | policy/trust records | policy/security authority | package/version/epoch | staged/active/blocked | divergent/rollback | issuer, anti-rollback, effective time | named authority decision only | mandatory review/incident | fail closed | queued effects rechecked | mandatory audit | rollback/revocation tests | 045/055 | never field-merge |
| command status sync | status/result evidence | authorized target/gateway | command+attempt+fence+status ID | reconcile/unknown | illegal/terminal divergence | lifecycle, reporter, sequence | query target; accept legal evidence; operator disposition | mandatory on ambiguity | capability concealed | never blind redispatch | E4 audit correlation | race/unknown tests | 045/055/056 | broker ACK != effect |
| command queue handoff | dispatch intent | exact executor grant | command revision+dispatch ticket | queued/dispatched | fence/expiry/authority | ticket, lease, policy/safety epoch | reauthorize or expire | safety review as configured | least privilege | single-use fenced action | pre-effect audit | duplicate dispatch tests | 045/046/055 | not ordinary sync |
| audit upload | AuditEventV1/ranges | origin audit authority | node+epoch+sequence+event ID+digest | accepted/gap/conflict | integrity/order | checkpoints, chain, manifest | append original; annotate resolution | mandatory collision review | separate audit access | command audit retained | receipt event separate | gap/tamper corpus | 048/049/055 | never renumber |
| federated import | CSAPI resources | contract-mapped remote | remote identity + local mapping | tentative/accepted | mapping/authority | peer profile, mapping version | import history or explicit promotion | promotion review | federation policy first | tasking disabled unless exact grant | federation audit | cross-server matrix | 045/046/056 | remote current not local truth |
| Part 3 delivery | Resource Event/Data | profile publisher | profile event + native resource identity | receive/replay | profile/topic/version | pinned profile, payload, digest | adapter to normal pipeline | incompatibility quarantine | topic is not authorization | inbound command channels disabled | delivery evidence | pinned experimental tests | 050/056 | no approved conformance claim |
| resnapshot | authorized view | serving node | snapshot/watermark/cursor | complete/new baseline | stale/hidden gap | cursor scope, retention, policy | new authorized snapshot | none unless integrity gap | no hidden counts | no command outcome inference | resnapshot audit | expiry/compaction tests | 048/050 | snapshot not history rewrite |

---

## 12. Conflict Record, Quarantine, Operator Review, and Resolution Strategy Findings

### 12.1 `SyncConflictV1`

A protected conflict record contains: conflict ID/version; state and severity; scenario/profile; detected-at node/time/correlation; safe type/reason code; affected resource/field/invariant references; contender references, immutable digests and revision/causal summaries; source/authority/policy/trust evidence references; gap and time-confidence state; canonical projection impact; quarantine location references; permitted resolution actions; reviewer/approval requirements; resolution record; audit/provenance/outbox references; retention/hold; and redacted public status. Sensitive values stay in access-controlled evidence, not free-text logs. **[P]**

### 12.2 Quarantine

Quarantine is durable isolation, not a directory of arbitrary files or an implicit reject. Objects are content-addressed or otherwise immutably referenced, encrypted/protected as required, malware/format bounded, tenant/domain separated, retention governed and excluded from canonical queries, projections, event publication and command execution. Every transition into, within or out of quarantine is authorized and audited. Revalidation after a schema/trust update creates new evidence; it does not alter the original failed finding. **[A/P]**

### 12.3 Resolution allowlist

| Resolution | Automatic? | Preconditions |
|---|---|---|
| exact replay deduplication | yes | identical scoped ID, digest, contract and prior durable result |
| append compatible immutable fact | yes | source authorized; no identity/contract/policy/invariant collision |
| single-authority monotonic successor | yes | profile names authority/order; epoch valid; predecessor/gap rules satisfied |
| proven commutative merge | conditional | exact data type, algebra, removal semantics, policy and invariants proven/tested |
| preserve both branches/candidates | yes as safe holding action | no canonical winner implied; clients qualified/blocked as required |
| authority-selected successor | operator/governed service | current grant, evidence review, separation of duties where required |
| explicit composite/corrective revision | operator/domain workflow | new revision with provenance linking all contenders |
| reject/revoke one contender | operator/security/policy workflow | authority and reason evidence; original retained per policy |
| map/remap external identity | operator/federation workflow | uniqueness, relationship and downstream reference analysis |
| command outcome disposition | specialized operator process | command authority, target evidence, safety and audit; no invented execution |
| audit annotation | specialized auditor | original immutable; annotation linked and separately authorized |

### 12.4 Prohibited silent resolvers

Arrival time, synchronization time, local database commit time, highest wall clock, latest UUID, longest vector, central/reference-node preference, largest sequence across epochs, broker order, transport QoS, database replication origin, generic last-write-wins and guessed “most restrictive” policy are prohibited as universal conflict resolvers. They may be used only where a narrowly approved profile proves the relevant single-authority semantic rule. **[A/P]**

### 12.5 Review workflow

Review is `open -> claimed -> evidence-reviewed -> decision-proposed -> approval-required/approved -> applying -> resolved` or `deferred/closed-unresolved`. Claim leases prevent concurrent operators but do not hide the conflict. High-risk identity, policy, trust, command, audit and disposition decisions require role separation or configured second approval. Applying a decision uses an expected conflict revision and rechecks policy/current state; stale decisions fail rather than overwrite newly arrived evidence. **[P]**

---

## 13. Source Trust, Provenance, Policy/Releasability, and Federation Findings

### 13.1 Source trust and provenance

Transport peer, envelope signer, original publisher, resource authority, validator, synchronization service and resolver are separate actors. Glaux records each and any delegation. A federation peer can truthfully relay a claim without becoming its original source or authority. Provenance retains source bytes/digest, transformations, schema/profile, validation, policy decision, receive/apply activities and resolver action. **[A/P]**

Unknown source, revoked/suspended source, expired grant, epoch rollback, signature mismatch and source-ID collision do not become ordinary metadata conflicts. They reject or quarantine under the accepted source-trust posture and can trigger a protected security incident. A stale but explicitly bounded offline bundle may support only the exact offline class it authorizes. **[A/P]**

### 13.2 Policy and releasability

Policy applies independently to transfer, storage, validation, use in a projection, onward publication, conflict review and diagnostic exposure. A receiver must not infer that a releasable payload authorizes release of its synchronization metadata, topology, competing source, policy label or command capability. Authorized view construction precedes counts, ranges, watermarks, latest selection and errors. **[A/P]**

Policy conflict fails closed. Glaux does not invent a semantic lattice across unknown marking regimes or union obligations. A recognized policy authority may install an explicit combination rule. Tightening/revocation can hide or block future use and queued work; it does not falsify the historical evidence and decision used when a local action occurred. **[A/P]**

### 13.3 Federation

Federation uses versioned partner profiles: peer identities; security domains; resource/source mappings; allowed scenarios/directions; authority scopes; schemas/media; identity/canonical-URL rules; policy transforms; conflict behavior; cursor/range rules; limits; audit evidence; and revocation/expiry. A public CSAPI read endpoint alone is not such a profile. Remote conformance claims are verified evidence, not permission to import every resource or task a remote System. **[P]**

Cross-domain transfer remains external. Glaux accepts only the released artifact and verifiable release/binding evidence exposed by an approved solution, rechecks local authorization and records the transform/custody chain. It makes no CDS, guard, certification or accreditation claim. **[A/P]**

---

## 14. Command Lifecycle, Command-Status Reconciliation, Unknown Outcome, and Command Audit Findings

Commands are not general replicated mutable objects. The canonical intent, each attempt, dispatch ticket, target acknowledgement, status report, result, cancellation, timeout assessment and operator disposition are separate durable records. Synchronization cannot broaden command authority or bypass the IDR-SRV-038 pre-effect gate. **[A/P]**

### 14.1 Reconciliation algorithm

1. Resolve exact Command identity, revision, ControlStream contract and target.
2. Verify reporter/executor/gateway authority, delegation, attempt ID and fence.
3. Validate status/result schema, report time, source epoch/sequence and evidence digest.
4. Check predecessor, legal public transition, private delivery state and terminality.
5. Append valid evidence even when delayed; advance current projection only under the lifecycle algorithm.
6. If claims disagree or a possible effect lacks verifiable outcome, retain `delivery_uncertain`/`reconciliation_required` and the last defensible public status.
7. Resolve by target query using stable operation identity, verified buffered evidence, proven effect-idempotent retry, authorized operator disposition, or persistent uncertainty.

A broker ACK proves broker/transport receipt only. An accepted synchronization receipt proves the receiving server stored/classified evidence only. Neither proves target execution. A late nonterminal status cannot replace a terminal status; two authorized divergent terminal outcomes require protected reconciliation. **[A/P]**

### 14.2 Queue handoff and duplicate prevention

Command handoff requires the same Command ID/revision, scoped idempotency key, single-use dispatch ticket, attempt/fence, target binding, execution window and policy/safety/trust epochs. The receiver performs immediate pre-dispatch re-evaluation. Loss of acknowledgement never causes a new Command ID or blind new physical attempt. Any adapter-specific retry must prove the target treats the stable operation identity idempotently. **[A/P]**

### 14.3 Command audit

Admission, local queue, sync transmit/receive, dispatch decision, attempted/acknowledged delivery, uncertainty, status evidence, conflict/quarantine, target query and resolution are correlated but distinct audit events. Required pre-effect E4 evidence remains local and durable before dispatch; remote export may lag but cannot substitute for it. Sensitive parameters, capabilities and denial reasons use protected references/digests. **[A/P]**

---

## 15. Audit Synchronization, Audit Gap, and Accountability Findings

Audit upload transfers immutable `AuditEventV1` records or integrity-protected ranges with origin node, chain epoch, sequence, event ID/digest, clocks, schema, checkpoints and known gaps. The receiver verifies but does not renumber, retime, rewrite or splice them into a false single global sequence. A separate local receipt/verification/apply event records the synchronization activity. **[A/P]**

| Condition | Required behavior |
|---|---|
| same audit ID and digest | deduplicate; preserve prior outcome |
| same ID, different canonical bytes | quarantine both references, protected collision incident, no arrival-wins |
| valid contiguous range | append/import under origin stream; advance scoped watermark |
| known missing sequence | retain surrounding records and explicit gap; request range if authorized |
| unknown/failed checkpoint | stop integrity claim at last verified boundary; quarantine affected range |
| delayed range | preserve original times/order; add receipt time separately |
| policy-hidden records | maintain authorized opaque progress without leaking count/identity |
| retention/disposition mismatch | do not silently delete or resurrect; invoke governed lifecycle process |

Audit conflict resolution is an append-only annotation/decision referencing immutable inputs. An operator cannot “repair” the original signer, timestamp, digest or sequence. Accountability queries must distinguish occurrence, local capture, remote receipt, verification, import and later annotation. **[A/P]**

---

## 16. API Response, Error, Event, and Observability Implications

### 16.1 API and problem behavior

Ordinary CSAPI routes continue to expose authorized resource semantics, not internal candidate state. A deployment may add protected profile operations for sync sessions, manifests, outcomes, conflicts and reviews. Bulk work is either atomic, explicit per-item, or a durable asynchronous operation with a status URI; it never reports blanket success while silently dropping items. **[A/P]**

| Condition | HTTP/profile behavior |
|---|---|
| durable sync operation accepted, incomplete | `202` plus durable operation `Location` and optional honest `Retry-After` |
| resource created through synchronous accepted import | applicable CSAPI `201` and canonical `Location` |
| missing required conditional revision | `428` with safe instruction |
| supplied `If-Match`/condition false | `412`; fetch/reconcile authorized current state |
| semantic/current-state or identity conflict | `409` with safe stable conflict type/status link if authorized |
| invalid CSAPI content | specification-appropriate `400`/media error with stable validation findings |
| payload/range too large | `413`; advertise documented bounds where safe |
| rate/backpressure | `429` or `503`; same idempotency identity and bounded retry |
| unavailable dependency/verification | `503` or durable `unverifiable` outcome, according to whether work was accepted |
| gone cursor/range | stable resnapshot-required problem; `410` only when semantics fit |
| denied/concealed | `401`/`403` or policy-selected `404`; no existence oracle |

Problem details use RFC 9457 with stable `type`, `title`, `status`, safe `detail`, opaque `instance`/correlation, `retryable`, `statusLink`, `resnapshotRequired` and policy-safe conflict/outcome code. They exclude peer topology, hidden IDs/counts, source names, policy labels/rules, trust failure internals, command capability/parameters, competing bytes, SQL and stack details. **[N/A/P]**

### 16.2 Events

Commit precedes publication through the accepted outbox. Logical event IDs remain stable across retries; consumers use inboxes. Event payloads identify occurrence/revision and profile, while delivery attempt and broker metadata remain separate. CloudEvents `source`+`id` may carry the logical event identity but does not replace resource revision, causality, domain sequence or policy binding. Part 3 remains outbound, experimental, version-pinned and disabled by default as accepted in IDR-SRV-035. **[N/A/P]**

### 16.3 Observability

Protected metrics include sessions/outcomes by safe class; received/applied/quarantined/conflicted/rejected counts; bytes and item rates; backlog age; range lag; gap age; dedupe ratio; validation/policy/trust failures; conflict age and review queue; retries; apply latency; projection lag; tombstone blocks; command uncertainty; audit continuity; outbox lag; and capacity. High-cardinality resource, peer, tenant and command IDs do not become unrestricted metric labels. **[A/P]**

Traces propagate opaque correlation across receive/validate/classify/apply/publish but do not carry payloads, tokens, markings or command parameters. Logs use stable codes and protected references. Health distinguishes HTTP process, authoritative store, mandatory audit path, sync admission, backlog/degradation and optional peer/broker dependencies; one global healthy/unhealthy flag is insufficient. **[A/P]**

---

## 17. Fixture, Conformance, Security Testing, Performance, Deployment, and Interoperability Test Implications

### 17.1 Required fixture and verification corpus

| Group | Required cases and expected invariant |
|---|---|
| replay/crash | crash before commit, after commit/before ACK, concurrent duplicate, expired key; at most one local effect |
| identity | same ID/same bytes, same ID/different bytes, equal bytes/different source, UID/canonical URL collision |
| revisions | causal successor, concurrent branch, missing base, epoch reset, offset reuse, sequence gap |
| time/latest | late older fact, tied result time, skewed node clock, reconnect order reversal; no arrival-wins |
| lifecycle | update after tombstone, concurrent delete/update, supersession chain, hold conflict; no resurrection |
| schema | known/unknown version, invalid encoding, malicious archive, dependency cycle, rollback package |
| policy/trust | unauthorized range, hidden competitor, revoked signer, stale bounded bundle, mapping conflict, downgrade attempt |
| Observations/status | valid history, correction divergence, terminal status regression, source-authority disagreement |
| commands | duplicate handoff, lost ACK, stale fence, illegal transition, conflicting terminal reports, unknown physical effect |
| audit | contiguous range, known gap, checkpoint failure, same event ID/different bytes, delayed upload, disposition conflict |
| streaming | boundary duplicate, expired cursor, compacted range, hidden event, broker replay, resnapshot |
| conflict review | stale operator decision, dual approval, reopen on new evidence, failed apply, redacted public view |
| failure/capacity | disk/audit full, quarantine full, schema service down, slow peer, cancellation, retry storm |
| interoperability | edge/reference version skew, external CSAPI identity map, experimental Part 3 adapter, CloudEvents duplicate |

Golden fixtures must preserve exact bytes plus canonical interpretation/digest and expected inbox, candidate, conflict, projection, audit and event outcomes. Synthetic policy/command/security data is required; controlled material does not enter public fixtures. **[A/P]**

### 17.2 Conformance boundary

Published Parts 1/2 conformance tests validate their API requirements. Glaux profile tests separately validate synchronization envelopes, admin operations and conflict behavior. Experimental Part 3 tests identify the exact pinned draft/profile and cannot claim approved OGC conformance. Infrastructure failover tests do not substitute for application reconciliation tests. **[N/A/P]**

### 17.3 Security testing

Test signature wrapping/substitution, decompression bombs, parser differentials, path/URI injection, SSRF through references, cross-tenant key collisions, replay floods, timing/existence oracles, unauthorized watermark probing, policy-label leakage, malicious schema/profile packages, trust rollback, forged authority, quarantine escape, reviewer privilege escalation, audit tampering and command redispatch. Fuzz every envelope/manifest/codec boundary with resource budgets. **[E/P]**

### 17.4 Performance and resilience testing

Measure ingest/apply throughput and tail latency by object class; dedupe/conflict overhead; range negotiation; restart/recovery; backlog catch-up without starving reads/commands/audit; projection rebuild; quarantine pressure; tombstone lookup; policy evaluation; and slow/hostile peer isolation. Results must state hardware, dataset, object sizes, conflict ratio, policy profile, durability settings and watermark definition. No universal numeric target is selected here. **[P/X]**

### 17.5 Deployment and interoperability implications

Category H must place authoritative store, artifact store, audit journal/spool, quarantine, workers, broker adapters and admin plane while preserving transaction and trust boundaries. It must decide which roles can be co-located, how keys/secrets and offline packages are managed, and how a node is restored without losing epochs, inbox history or tombstones. Interoperability tests must exchange a declared profile, not assume two CSAPI servers synchronize merely because both serve Parts 1/2. **[P/X]**

---

## 18. Downstream Topic Handoff Matrix

| Downstream topic | Fixed handoff from IDR-SRV-043 | Still open downstream |
|---|---|---|
| IDR-SRV-044 platform/toolchain | need durable transactions, canonical digests, validation, crypto, bounded codecs and test support | exact crates/tool versions |
| IDR-SRV-045 architecture/modularization | transport adapters separated from receive/classify/apply; inbox, conflict, quarantine, policy and audit ports | module/process boundaries and APIs |
| IDR-SRV-046 reference deployment | server vs broker/database/CDS ownership; explicit node roles and trusted profiles | topology, HA, regions and products |
| IDR-SRV-047 configuration | versioned federation/sync profiles, limits, grants and feature gates | file/env/secret model and reload rules |
| IDR-SRV-048 observability | protected metrics/traces/logs and health dimensions | telemetry backend, SLOs and alert thresholds |
| IDR-SRV-049 migration/backup/restore | preserve IDs, revisions, inboxes, conflicts, tombstones, audit epochs and cursors | backup topology, RPO/RTO and upgrade choreography |
| IDR-SRV-050 conformance harness | separate published conformance from Glaux profile/experimental Part 3 tests | harness implementation and evidence format |
| IDR-SRV-051 developer workflow | deterministic fixture/golden and fault-injection needs | CI lanes and local services |
| IDR-SRV-052 documentation | publish profile boundaries, retry/status semantics and operational runbooks | information architecture and generated docs |
| IDR-SRV-053 fixtures | scenario corpus in Section 17; exact bytes and expected outcomes | fixture packaging and generators |
| IDR-SRV-054 performance | workload dimensions, backlog/catch-up and contention cases | numeric budgets/benchmarks |
| IDR-SRV-055 security testing | attack corpus, non-bypass gates and command/audit invariants | tools, environments and accreditation evidence |
| IDR-SRV-056 interoperability | explicit partner profile, identity mapping and version skew | server/client matrix and partner endpoints |
| IDR-SRV-057 final synthesis | Category G boundary and unresolved parameters | final sequencing and cross-topic reconciliation |

No Category H topic is authorized by this handoff. **[P]**

---

## 19. Recommendations

1. Accept `SyncEnvelopeV1`, `SyncConflictV1` and the orthogonal session/candidate/outcome/continuity state model as conceptual contracts.
2. Implement at-least-once receipt with an inbox atomically committed with every local outcome; make no end-to-end exactly-once claim.
3. Exchange immutable revisions/evidence and recompute local authorized projections; do not replicate mutable “current rows” as domain truth.
4. Require scenario-specific, versioned peer profiles and least-privilege synchronization grants; do not expose a semantics-free generic sync endpoint.
5. Permit automatic resolution only through the explicit allowlist and invariant proof; reject generic LWW, central preference and guessed policy merges.
6. Treat tombstones, policy/trust packages, commands and audit as specialized protected records with stricter rules.
7. Preserve original bytes/digest, interpretation versions, provenance, authority, policy and all relevant clocks for every consequential candidate.
8. Make gaps, uncertainty, quarantine and unresolved conflict first-class durable states that can block or qualify projections.
9. Keep normal CSAPI responses standards-valid and expose internal detail only through separately authorized profile/admin resources.
10. Build the Section 17 fixture corpus before production synchronization, federation or inbound Part 3 channels are enabled.
11. Keep database replication, broker delivery and cross-domain transfer subordinate to this application contract.
12. Carry topology, products, numeric horizons, CRDT/vector/Merkle representation and UI choices into the named downstream topics.

---

## 20. Risks, Constraints, and Open Questions

### 20.1 Risks and controls

| Risk | Control |
|---|---|
| transport/database mechanisms mistaken for domain convergence | explicit ownership table and application receive pipeline |
| duplicate physical command effect | stable operation identity, fence, target query and no blind retry |
| silent canonical corruption | immutable contenders, explicit conflict, allowlisted resolution |
| stale resurrection after deletion | authoritative tombstones and horizon/restore tests |
| policy or topology disclosure | policy-first selection, opaque cursors and protected diagnostics |
| unbounded quarantine/backlog DoS | quotas, admission, isolation, backpressure and alerting |
| source/trust rollback | signed versioned packages, epochs and anti-rollback state |
| false completeness after gaps/compaction | scoped watermarks, explicit gaps and resnapshot |
| audit evidence rewritten during aggregation | preserve origin sequence/bytes; separate receipt/annotation |
| interoperability drift | version-pinned partner profiles and golden exchanges |
| CRDT/LWW overapplication | per-type proof and prohibited-default rules |

### 20.2 Open parameters and owners

| Open question | Owner/topic |
|---|---|
| first-implementation synchronization scenarios | IDR-SRV-045/046 and project lead |
| concrete wire protocol, manifest and range negotiation | IDR-SRV-045/046 |
| vector/dotted version, Merkle tree or alternative mechanics | IDR-SRV-045 after prototype evidence |
| tombstone, inbox, replay, conflict and quarantine durations | IDR-SRV-049 plus policy/legal owner |
| node/federation topology, discovery and routing | IDR-SRV-046 |
| exact review UI, roles and approval thresholds | product/security design |
| broker/database products and HA | IDR-SRV-044/046 |
| numeric backlog, size, rate and latency thresholds | IDR-SRV-047/048/054 |
| operational policy/marking adapters and CDS integration | deployment authority; outside Glaux CDS scope |
| whether any field earns a CRDT merge | architecture prototype plus domain/security review |
| inbound experimental Part 3 scope | later implementation decision; disabled by default now |

---

## 21. Validation Against This Plan's Success Criteria

| Success criterion | Evidence | Result |
|---|---|---|
| scenarios and server-scope boundaries with anchors | Sections 3, 5 and 6 | Met |
| resources classified by authority, mutability, cache, append, identity and sync | Section 7 | Met |
| states, replay/idempotency, offsets, duplicates and delay documented | Sections 8–10 | Met |
| conflicts, detection, records, quarantine and resolution documented | Sections 11–12 and 15-column matrix | Met |
| trust, policy, provenance, commands, audit and events documented | Sections 13–16 | Met |
| API, errors, observability and verification implications documented | Sections 16–17 | Met |
| implementation/community lessons incorporated non-normatively | Sections 3 and 16; accepted 014A–H baseline | Met |
| decision-usable and server-bounded recommendations | Sections 5, 19 and 20 | Met |
| downstream handoffs explicit | Section 18 | Met |
| references explicit and reproducible | Section 22 and immutable pins | Met |

The report satisfies the plan while intentionally leaving deployment topology, products, numeric thresholds, wire details, retention horizons and UI to their assigned topics. The deliverable should remain **In Review** until project-lead acceptance. **[P]**

---

## 22. References

### 22.1 Published standards and primary technical sources

1. OGC, [OGC API - Connected Systems - Part 1: Feature Resources](https://docs.ogc.org/is/23-001/23-001.html), OGC 23-001, Version 1.0.
2. OGC, [OGC API - Connected Systems - Part 2: Dynamic Data](https://docs.ogc.org/is/23-002/23-002.html), OGC 23-002, Version 1.0.
3. OGC, [OGC API - Connected Systems repository](https://github.com/opengeospatial/ogcapi-connected-systems), tag `v1.0.0` `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`; HEAD `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`, checked 2026-09-15.
4. OGC, [Sensor Model Language (SensorML) 3.0](https://docs.ogc.org/is/23-000/23-000.html).
5. OGC, [SWE Common Data Model 3.0](https://docs.ogc.org/is/24-014/24-014.html).
6. IETF, [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110), especially Sections 13 and 15.5.10/13.
7. IETF, [RFC 9111: HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111).
8. IETF, [RFC 6585: Additional HTTP Status Codes](https://www.rfc-editor.org/rfc/rfc6585), Section 3.
9. IETF, [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457).
10. IETF, [RFC 3339: Date and Time on the Internet](https://www.rfc-editor.org/rfc/rfc3339).
11. OASIS, [MQTT Version 5.0](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html).
12. CNCF, [CloudEvents Specification](https://github.com/cloudevents/spec), release 1.0.2 stable tag `fc1f6f31f5f011a72183f1bcea20c987cb683ade`, checked 2026-09-15.
13. PostgreSQL Global Development Group, [PostgreSQL 18 Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html), checked 2026-09-15.
14. PostgreSQL Global Development Group, [PostgreSQL 18 Logical Replication Conflicts](https://www.postgresql.org/docs/current/logical-replication-conflicts.html), checked 2026-09-15.
15. PostgreSQL Global Development Group, [PostgreSQL 18 Logical Replication Restrictions](https://www.postgresql.org/docs/current/logical-replication-restrictions.html), checked 2026-09-15.
16. Marc Shapiro, Nuno Preguica, Carlos Baquero and Marek Zawirski, [Conflict-Free Replicated Data Types](https://perso.lip6.fr/Marc.Shapiro/papers/2011/CRDTs_SSS-2011.pdf), 2011.

### 22.2 Accepted project evidence

17. [IDR-SRV-016 Identifier, URI, and Resource Lifecycle Strategy](idr-srv-016-identifier-uri-and-resource-lifecycle-strategy-report.md).
18. [IDR-SRV-018 Temporal Validity and Freshness Model](idr-srv-018-temporal-validity-and-freshness-model-report.md).
19. [IDR-SRV-019 Provenance, Lineage, Quality, and Trust Metadata Model](idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md).
20. [IDR-SRV-023 Schema and Encoding Validation Strategy](idr-srv-023-schema-and-encoding-validation-strategy-report.md).
21. [IDR-SRV-029 Transaction, Consistency, Idempotency, and Concurrency Strategy](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md).
22. [IDR-SRV-030 Data Lifecycle, Retention, Archival, and Deletion Strategy](idr-srv-030-data-lifecycle-retention-archival-and-deletion-strategy-report.md).
23. [IDR-SRV-035 Streaming and Event Publication Strategy](idr-srv-035-streaming-and-event-publication-strategy-report.md).
24. [IDR-SRV-036 Control Stream and Command Lifecycle Model](idr-srv-036-control-stream-and-command-lifecycle-model-report.md).
25. [IDR-SRV-038 Command Authorization, Safety, and Audit Strategy](idr-srv-038-command-authorization-safety-and-audit-strategy-report.md).
26. [IDR-SRV-039 Authentication, Authorization, and API Security Threat Model](idr-srv-039-authentication-authorization-and-api-security-threat-model-report.md).
27. [IDR-SRV-039A Zero-Trust Architecture Alignment and Enforcement Model](idr-srv-039a-zero-trust-architecture-alignment-and-enforcement-model-report.md).
28. [IDR-SRV-040 Policy, Releasability, and Cross-Boundary Access Constraints](idr-srv-040-policy-releasability-and-cross-boundary-access-constraints-report.md).
29. [IDR-SRV-041 Audit Logging and Accountability Strategy](idr-srv-041-audit-logging-and-accountability-strategy-report.md).
30. [IDR-SRV-042 DDIL-Informed Server Semantics](idr-srv-042-ddil-informed-server-semantics-report.md).
31. Controlled AEP-4789 package `AC/224(JCGISR)D(2026)0005`, dated 2026-04-27, pre-promulgation/controlled; referenced without reproducing or publicly linking controlled content.

### 22.3 Reproducibility record

- Official repository pins were checked with `git ls-remote` on 2026-09-15.
- PostgreSQL `current` resolved to Version 18 on 2026-09-15.
- The upstream-history register remains Version 1.12; this research found no material Part 1/2 published-history change requiring an update.
- The report uses accepted implementation-study results instead of silently reinterpreting mutable live systems.

---

## Report Completion Checklist

- [x] All required report sections are present.
- [x] Required synchronization/conflict matrix contains all minimum columns.
- [x] Normative, controlled, accepted, implementation and proposed claims are distinguished.
- [x] Server responsibility is separated from deployment, broker, database and cross-domain infrastructure.
- [x] No exactly-once, generic LWW, central-preference or silent-policy-merge claim is made.
- [x] Commands, policy/trust, tombstones and audit receive stricter conflict treatment.
- [x] Fixtures, security, performance, deployment and interoperability handoffs are explicit.
- [x] Category H and implementation remain unauthorized pending acceptance.

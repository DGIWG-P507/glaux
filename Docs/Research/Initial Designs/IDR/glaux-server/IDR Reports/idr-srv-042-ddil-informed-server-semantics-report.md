# Section 042: DDIL-Informed Server Semantics - Research Report

**Topic ID:** IDR-SRV-042<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-042 DDIL-Informed Server Semantics](../IDR%20Plans/idr-srv-042-ddil-informed-server-semantics.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All questions concerning DDIL modes, dependency and service posture, freshness/validity/last-known/cached/tentative/delayed/unknown/unavailable state, resource and operation behavior, dynamic data, streams, commands, identity/policy/source trust, schemas, responses, audit, client expectations, synchronization handoff, and verification<br>
**Methodology Used:** Current primary-source freeze; accepted-baseline reconciliation; multi-dimensional state modeling; resource/operation inventory; safety and disclosure analysis; standards-versus-profile classification; implementation-lesson comparison; scenario and verification traceability<br>
**Research Time:** Approximately 32 hours of AI-assisted execution on September 15, 2026<br>
**Approved Standards Baseline:** OGC 23-001 and OGC 23-002 Version 1.0; SensorML 3.0; SWE Common 3.0; accepted Glaux IDR-SRV-001 through IDR-SRV-041; controlled AEP-4789 package `AC/224(JCGISR)D(2026)0005` subject to its recorded status and handling limits<br>
**Protocol Guidance Baseline:** RFC 9110, RFC 9111, RFC 9457, RFC 3339, MQTT 5.0, and the experimental Part 3 profile decision accepted in IDR-SRV-035, checked September 15, 2026<br>
**Implementation Evidence:** Official CSAPI repository tag `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`, current repository head `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`, and Part 3 working-draft head `6f529a15bfa63259febc3620378d3e5a06305333`; accepted OSH, CS-Go, pygeoapi, SECD, client-smoke, interoperability, discussion, and Part 3 studies used only as non-normative evidence<br>
**Supporting Resources:** Accepted temporal, status, validation, persistence, transaction, lifecycle, ingestion, dynamic-data, streaming, command, security, ZTA, policy, and audit reports; upstream-history register Version 1.12<br>
**Document Purpose:** Define a safe, interoperable and falsifiable server contract for connected, constrained, intermittent, disconnected, local-only and recovering conditions without designing synchronization/conflict mechanics, deployment topology, or numeric operational policy<br>
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
| C | Controlled AEP/project-source finding, limited to the cited accessible evidence |
| A | Accepted Glaux design baseline |
| I | Pinned implementation, test, or community evidence; not a requirement |
| E | Engineering, safety, or interoperability inference |
| P | Proposed Glaux decision requiring this report's acceptance |
| X | Open parameter, limitation, or downstream decision |

“Must” in a proposed invariant describes the minimum needed for a truthful or safe Glaux profile. It does not convert DDIL behavior into an OGC Parts 1/2 requirement. The approved CSAPI standards define resource and time semantics, but they do not define offline queues, a generic stale-data envelope, resynchronization, or conflict resolution. Those additions are explicit Glaux profile behavior. **[N/A/P]**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. DDIL Semantics Extraction Methodology
5. DDIL Operating Mode Taxonomy
6. Freshness, Validity, Last-Known, Cached, Tentative, Delayed, Unavailable, Unknown, and Stale State Model
7. Resource Family Behavior Findings
8. Operation Behavior Findings
9. Observation, Status, Latest-Value, Source-Health, and Delayed-Update Findings
10. Streaming, Event, Cursor, Reconnect, and Latest-State Snapshot Findings
11. Command, Feasibility, Command-Status, and Unknown-Outcome Findings
12. Authentication, Authorization, Source Trust, Policy, Credential, and Redaction Findings
13. Error, Warning, Problem-Detail, Response Metadata, Diagnostic, and Audit Findings
14. Synchronization/Conflict Handoff Findings
15. Fixture, Conformance, Security Testing, Performance, Deployment, Observability, and Interoperability Test Implications
16. Downstream Topic Handoff Matrix
17. Recommendations
18. Risks, Constraints, and Open Questions
19. Validation Against This Plan's Success Criteria
20. References

---

## 1. Executive Summary

Glaux should treat DDIL as a set of **orthogonal evidence conditions**, not a single global server mode. Link state, each dependency's reachability, local authority, data freshness, time confidence, storage/audit capacity and synchronization progress can differ simultaneously. The server derives an internal operation decision and a policy-safe client posture from those facts. A normal caller learns only what is needed to use the authorized resource; administrators receive protected dependency detail. **[A/E/P]**

The central rule is: **serve facts with their honest time, authority, completeness and finality; never relabel old evidence as current because the source is unreachable.** A current projection may be last-known and stale. A cached representation may be fresh or stale. A late Observation can be valid history. A locally committed resource can be authoritative for its local domain yet not synchronized. “Unavailable” describes a capability; “unknown” describes insufficient knowledge. These dimensions remain separate. **[N/A/P]**

Static, version-pinned service artifacts—landing, OpenAPI, conformance declarations, schemas and installed vocabularies—should remain locally available whenever the HTTP service and authorization path can safely serve them. Historical authorized resources remain queryable from the local authoritative store. Latest/status views may return last-known evidence with a typed assessment. Metadata writes and ingestion continue only where the node has explicit local authority and the accepted transaction, policy, source and audit gates remain satisfied. Administration is normally connected-authority-required. **[A/P]**

Commands are stricter. New command admission or dispatch is disabled unless an accepted offline operating class explicitly permits it. Queueing requires a bounded validity window, stable idempotency identity, local durable intent/audit, policy and safety evidence that remains valid, and honest `PENDING` semantics. Every queued command is re-authorized and revalidated before dispatch. Stale feasibility, stale target status, broker acknowledgement, or a last-known allow can never prove safety or effect. Lost contact after possible dispatch produces private reconciliation and the last defensible public status—not invented failure, cancellation, or completion. **[A/P]**

Streaming follows the accepted snapshot-watermark, durable logical cursor, replay and resnapshot contract. A transport session, MQTT packet identifier, retained message or broker QoS is not a cross-adapter resume cursor. When continuity cannot be proven, the server reports an authorized gap or requires resnapshot; it never implies lossless continuity. Broker outage does not remove committed domain records, and reconnect arrival order never becomes domain order. **[A/P]**

Successful degraded reads normally remain `200` with standards-valid content plus an opt-in Glaux `RepresentationAssessmentV1` or protected service-status link. HTTP cache `Age` and `Cache-Control` describe representation caching, not domain freshness. `206` is reserved for range semantics and is not used for missing sources. `202` means accepted but incomplete processing and requires durable status; it does not replace the accepted CSAPI Command/Feasibility `201` plus `PENDING` contract. Temporary inability uses `503` and honest `Retry-After`; conflicts and failed preconditions retain `409`/`412`/`428`. Problem details expose stable, corrective, non-topological information only. **[N/A/P]**

Acceptance of this report would establish the semantic contract, not numeric thresholds, topology, products, or synchronization algorithms. IDR-SRV-043 must define manifests, range exchange, duplicate/conflict/tombstone mechanics and reconciliation protocol without weakening the state meanings fixed here.

### 1.1 Decisions requested

Accept:

1. the orthogonal DDIL state model and derived service postures in Section 5;
2. the freshness/validity/authority/finality vocabulary and `RepresentationAssessmentV1` in Section 6;
3. the resource and operation behavior matrices in Sections 7–8;
4. the dynamic-data, stream and command safeguards in Sections 9–11;
5. the locally verifiable, no-authority-expansion security posture in Section 12;
6. the HTTP, diagnostic and audit contract in Section 13;
7. the semantic inputs and boundaries handed to IDR-SRV-043 in Section 14; and
8. the verification implications, recommendations, residual risks and downstream ownership in Sections 15–18.

## 2. Scope and Plan Alignment

### 2.1 In scope

- client- and administrator-visible behavior during connected, constrained, intermittent, disconnected, local-only and recovering conditions;
- service posture, dependency and mode vocabulary;
- domain freshness, validity, last-known, cached, tentative, delayed, unknown and unavailable semantics;
- behavior for every resource and operation family named in the plan;
- safe response, problem-detail, status, stream, command, source-trust, policy and audit behavior;
- implementation lessons and testable downstream obligations; and
- the semantic boundary that IDR-SRV-043 must consume.

### 2.2 Out of scope

- replication protocol, delta format, vector/causal algorithm, conflict winner, tombstone horizon or anti-entropy implementation;
- final topology, broker, SIEM, WORM, cache, database or policy-engine product;
- production TTL, maximum-offline age, bandwidth, queue, storage, clock, replay or recovery numbers;
- an OGC Part 3 conformance claim, real operational policy/marking scheme, or live command authorization; and
- client-device offline storage/UI design except for server-facing expectations.

### 2.3 Research-question coverage

| Plan question | Status | Primary evidence |
|---|---|---|
| Q1: operating modes and degraded service states | Complete | Sections 5 and 13 |
| Q2: freshness, validity and state meanings | Complete | Sections 6 and 9 |
| Q3: resource and operation behavior | Complete | Sections 7–8 |
| Q4: dynamic data, streams, commands, security, policy, audit, OpenAPI and conformance | Complete | Sections 9–13 |
| Q5: synchronization, deployment, observability, testing and interoperability handoffs | Complete | Sections 14–16 |

No core question remains unanswered. Numeric/profile parameters and exact synchronization mechanics are intentionally unresolved with named downstream owners.

## 3. Evidence Base and Authority Classification

### 3.1 Source inventory

| Source | Version/pin/status | Class | Anchor used | Access | Limitation |
|---|---|---|---|---|---|
| OGC API - Connected Systems Part 1 | OGC 23-001, 1.0, approved | N | §§7–18; Req. 3; resource `validTime`; service artifacts | 2026-09-15 | No DDIL/offline state model |
| OGC API - Connected Systems Part 2 | OGC 23-002, 1.0, approved | N | §§9–13; Tables 2, 6, 8, 13–14, 19; Reqs. 25, 31, 46, 49–50 | 2026-09-15 | No generic freshness, queue, replay, or sync contract |
| SensorML | OGC 23-000, 3.0, approved | N | §§8.2, 8.5, 9.1.4.6; valid time, constraints, interfaces | 2026-09-15 | Describes systems/processes; not reachability truth |
| SWE Common | OGC 24-014, 3.0, approved | N | §§7–10; Time, quality, nil and encoding semantics | 2026-09-15 | Data model, not transport continuity |
| RFC 9110 | June 2022 Internet Standard | N | §§8.8, 10.2.3, 13, 15.3.3, 15.5.10/13, 15.6.4 | 2026-09-15 | HTTP status does not encode domain freshness |
| RFC 9111 | June 2022 Internet Standard | N | §§4.2–4.2.4, 5.1–5.2 | 2026-09-15 | Cache freshness is representation freshness only |
| RFC 9457 | July 2023 Proposed Standard | N | §§3 and 5 | 2026-09-15 | Extension members need a Glaux profile |
| MQTT 5.0 | OASIS Standard | N within MQTT binding | §§3.1.2, 3.3.2, 4.3; session/message expiry, Receive Maximum, QoS | 2026-09-15 | Transport guarantees do not prove domain commit/effect |
| CSAPI repository | `v1.0.0` `8e03b236...`; head `3fd86c73...` | N for tag; I for head | tagged standards/OpenAPI and mutable development history | 2026-09-15 | Head is not adopted baseline |
| Part 3 working draft | head `6f529a15...` | D/I | experimental events/data-message model; accepted IDR-SRV-014H/035 analysis | 2026-09-15 | Draft; binding/ATS and DDIL mechanics incomplete |
| Controlled AEP package | `AC/224(JCGISR)D(2026)0005`, Apr. 27, 2026; recorded digest | C | accepted IDR-SRV-001/002 DDIL, last-known, status, exchange findings | 2026-09-15 | Controlled pre-promulgation draft; not reproduced or publicly linked |
| Accepted Glaux reports | IDR-SRV-001 through 041 | A | temporal/status/policy/audit/transaction/command handoffs cited below | 2026-09-15 | Design baseline, not implemented proof |
| OSH study | core `v2.0.2` `235c0eab...` | I | historical/live split, event bus, backpressure; reconnect gaps | accepted 2026-08-31 | Java implementation precedent only |
| CS-Go and Part 3 studies | `v1.0.4`; accepted pins `244f4dd...` and `4a00aa6...` | I | MQTT reconnect/config, volatile publish, missing replay/outbox | accepted 2026-08-31 | Current repository head is mutable; no production guarantee |
| pygeoapi study | accepted implementation pin in IDR-SRV-014C | I | offline schema bundle; unlogged derived-cache recovery failure | accepted 2026-08-31 | Narrow PoC; no CSAPI streaming/tasking |
| SECD/interoperability studies | repository `f018fd12...` | I | SSE declaration, temporal/filter and lifecycle negative cases | accepted 2026-08-31 | Prototype/deployment behavior, not standard |
| OS4CSAPI client/discussion studies | phase-9 `75441189...`; 14 discussions through 2026-08-31 | I | client assumptions; typed stale/failed/hidden states | accepted 2026-08-31 | Discussion proposals are not implementation evidence |

### 3.2 Standards-derived facts versus Glaux decisions

The standards supply resource identity, validity and domain timestamps; Observation `resultTime=latest`; DataStream `live`; CommandStatus states and report time; SystemEvent time; HTTP validators, cache controls and error semantics; and transport-level session/expiry mechanisms. **[N]**

They do not require the mode names in Section 5, `RepresentationAssessmentV1`, offline authorization classes, queued-write policy, cursor/log retention, source-connectivity assessments, or conflict behavior. Those are Glaux decisions derived from AEP objectives, accepted reports, safety analysis and interoperability evidence. **[C/A/E/P]**

### 3.3 Evidence limits

No public NATO DDIL implementation profile with numeric freshness, authority or queue thresholds was established. The controlled AEP package supports useful local operation, temporal/validity context, last-known state, constrained exchange and later synchronization, but not the algorithms or thresholds. Implementation studies uniformly demonstrate partial behavior or gaps rather than a complete DDIL contract. Consequently, this report defines explicit semantic seams and safe defaults while leaving deployments to supply bounded values.

### 3.4 Implementation and community lessons

| Evidence | Observed lesson | Adopt/investigate | Avoid/limit |
|---|---|---|---|
| OSH | Separate historical replay and live event paths; demand-aware subscriber flow provides a backpressure seam | retain distinct query/replay/live state machines and exercise disconnect/slow-consumer cases | do not infer cursor, authorization-expiry, durability or reconnect guarantees that were not established |
| CS-Go | Capability-derived AsyncAPI and optional MQTT show deployable draft-Part-3 feasibility | derive advertised transport from active capability and keep it optional/versioned | volatile fire-and-forget publication, clean sessions and a short echo cache are not outbox, replay or idempotency |
| pygeoapi PoC | Pinned offline schema bundles are useful; discovery can otherwise depend on live providers | package schemas/reference graphs locally and cache capability metadata with explicit provenance | unlogged derived public-state caches and unfinished recovery cannot be authoritative |
| SECD | Persisted history and declared SSE provide realistic interoperability inputs | use its temporal, live and lifecycle shapes as adversarial fixtures | no resume, ordering, replay, backpressure or authorization guarantee can be inferred from an OAS declaration |
| Client smoke tests | clients can mishandle `latest`, asynchronous responses, links, complex SWE and incomplete API descriptions | test exact values, IDs, clocks, links and generated-client behavior | status-only smoke tests and narrow-client behavior are not standards or server semantics |
| OS4CSAPI discussions | stale, denied, unavailable, hidden and empty need distinct treatment; source failures should isolate | preserve typed state and source-qualified identity as design prompts | synthetic architecture proposals are not proof of implementation or normative authority |
| Draft Part 3 study | resource/data messages can reduce polling and overhead; aggregate events can reduce volume | retain the accepted experimental profile and logical-cursor boundary | MQTT/QoS/retain/session state does not create DDIL, database atomicity, replay or domain ordering |

NATS, Kafka and alternative brokers remain deployment candidates, not semantic authorities. Selecting one cannot change the resource, policy, audit, cursor, idempotency, freshness or command invariants in this report. **[I/E/P]**

## 4. DDIL Semantics Extraction Methodology

The analysis applied five steps:

1. Extract normative resource/time/status/HTTP/transport meanings without adding offline implications the source does not state.
2. Reconcile accepted Glaux invariants for temporal evidence, current projection, transactionality, command safety, policy, zero trust and audit.
3. Decompose “offline” into independently observable link, dependency, authority, data, clock, capacity and synchronization dimensions.
4. Evaluate each resource and operation against standards alignment, usefulness, safety, policy, interoperability, diagnostic leakage, testability and deployment feasibility.
5. Convert the result into falsifiable profiles, response behavior, fixtures and downstream handoffs.

### 4.1 Decision rubric

| Question | Continue | Degrade/qualify | Queue/stage | Disable/deny |
|---|---|---|---|---|
| Is required local data and exact schema available? | yes | incomplete but honest read | candidate can be durably isolated | no safe interpretation |
| Is identity/policy/source authority locally verifiable? | yes | narrower pre-authorized view | only explicit offline class | absent, expired, rollback, indeterminate |
| Can accepted transaction/audit invariants hold? | yes | safe read with stated limits | durable bounded spool/outbox | would silently lose required evidence |
| Can external effect be fenced and rechecked? | yes | no effect; retain status | explicit valid queued intent | uncertain unsafe effect or no pre-effect audit |
| Can the response avoid protected-state leakage? | yes | generalized assessment | admin-only detail | conceal/deny |

### 4.2 Non-collapse rules

- Network reachability is not System operational status.
- Server service health is not source health.
- HTTP cache freshness is not domain freshness.
- Latest by a standard selector is not necessarily fresh.
- `live=true` is not command readiness, data freshness or guaranteed streaming continuity.
- Valid history is not invalid because it arrived late.
- Authenticated is not currently authorized; previously authorized is not indefinitely authorized.
- Broker delivery is not database commit, command acceptance or physical effect.
- Recovery is not connected-normal until authority, gaps, queues and projections are reconciled.

## 5. DDIL Operating Mode Taxonomy

### 5.1 Connectivity condition

| Code | Internal condition | Detection | Permitted claim |
|---|---|---|---|
| `C0_CONNECTED` | Required configured paths meet current health policy | authenticated probes, successful exchanges, bounded lag | Connected for named path at assessment time only |
| `C1_LIMITED` | Reachable but bandwidth/latency/loss/quotas constrain service | measured thresholds or admin profile | Limited; selected compression/batching/priority may apply |
| `C2_INTERMITTENT` | Path repeatedly transitions or misses bounded exchanges | hysteretic observation | Intermittent; continuity cannot be assumed |
| `C3_DISCONNECTED` | Named external path unavailable | bounded failed checks or explicit isolation | Disconnected from named dependency, not “all systems down” |
| `C4_LOCAL_ONLY` | Deployment is intentionally isolated and locally authoritative for declared classes | signed configuration/profile | Local-only within declared authority; not a fault |
| `C5_RECOVERING` | Connectivity returned but security state, queues, gaps or projections remain unreconciled | recovery state machine | Recovering/resynchronizing; normal authority not yet restored |
| `C6_UNKNOWN` | Evidence cannot classify the path safely | missing/conflicting/time-uncertain evidence | Unknown; never inferred healthy |

These are scoped to `(node, dependency/path, direction, assessment time)`. The server must not store one global enum that erases simultaneous states. Hysteresis prevents mode flapping; exact thresholds are deployment policy. **[E/P/X]**

### 5.2 Orthogonal internal dimensions

| Dimension | Example values | Why separate |
|---|---|---|
| dependency | local store, source, broker, IdP, policy/trust, schema/vocabulary, command gateway, time, audit/export | one may fail while others work |
| service posture | full, degraded, read-only, locally writable, queue-only, command-disabled, protected-unavailable | derived per operation/profile |
| authority | source, local, delegated offline, cached-bounded, tentative, none/indeterminate | connectivity never creates authority |
| data assessment | fresh, stale, unknown, not-applicable; complete/partial/unknown | content state differs from service state |
| time confidence | synchronized, bounded-uncertain, untrusted, unavailable | affects expiry/order/authorization |
| synchronization | current-to-watermark, backlog, gap-known, gap-unknown, conflict, quarantined, recovering | reconnect is not reconciliation |
| capacity | normal, warning, reserved-only, exhausted | determines safe admission/audit behavior |

### 5.3 Dependency-to-operation posture

| Dependency state | Safe reads | Writes/ingest | Streams | Commands/effects | Administration |
|---|---|---|---|---|---|
| source unavailable; local store healthy | serve authorized history/last-known with assessment | other registered sources may continue | replay/local events; no claim of source-live | normally disable target-dependent effect | protected diagnostic only |
| broker unavailable | HTTP history unaffected | commit plus durable outbox if capacity | adapter unavailable; cursor retained | broker-dispatch stays queued only if valid | broker config changes follow admin policy |
| IdP/introspection unavailable | locally verifiable bounded class only | explicit offline class only | existing lease until bounded expiry | only accepted offline command class | deny by default |
| policy/trust service unavailable | fresh protected local bundle only | explicit local scope | re-evaluate on bundle/lease expiry | no last-known allow; dispatch gates remain | deny by default |
| schema/vocabulary origin unavailable | installed reference-closed packages continue | exact installed contract only | encoded payloads keep contract pin | exact installed command contract only | importing new material disabled/quarantined |
| audit primary unavailable | class-specific local spool per E1–E5 | E3/E4 only if required durable path exists | protected sessions within E1 capacity | no effect without E4 durability | deny sensitive audit/admin changes |
| local authoritative store unavailable | no authoritative query; optional separately identified static artifacts | deny | close/resnapshot later | deny | recovery-only protected path |

### 5.4 Exposure policy

Normal clients receive a coarse, resource-relevant assessment: `normal`, `degraded`, `last-known`, `partial`, `queued`, `temporarily-unavailable`, or `unknown`, plus safe time/next-action information. They do not receive dependency names, peer identities, policy epochs, queue depths, routes, source health, command gateways, audit topology or synchronization ranges unless separately authorized. Administrators receive the detailed matrix through a protected operational interface and audit trail. **[A/P]**

Mode transitions are hysteretic, append evidence, update protected health, and can emit a policy-approved service event. They do not automatically create a CSAPI `SystemEvent`, because a server dependency transition is not necessarily an occurrence on the represented System. **[A/P]**

## 6. Freshness, Validity, Last-Known, Cached, Tentative, Delayed, Unavailable, Unknown, and Stale State Model

### 6.1 Canonical definitions

| Term | Glaux meaning | Not equivalent to |
|---|---|---|
| valid | assertion/resource applies under its domain validity interval and rules | fresh, trusted, reachable |
| current projection | fact selected by the accepted projection algorithm at an evaluation snapshot | newest arrival, fresh, complete |
| fresh | named evidence age/conditions satisfy a versioned policy at `assessedAt` | recent HTTP response |
| stale | valid evidence exists but exceeds/violates the named freshness rule | invalid or deleted |
| last-known | most recent eligible evidence known to this authorized view; no newer eligible evidence is known | current-at-source, fresh, globally latest |
| cached | representation or evidence came from a local/intermediate retained copy | stale or untrusted |
| delayed | source/domain occurrence precedes receipt/commit beyond a declared rule | stale at query time or invalid |
| tentative | locally durable candidate lacks required final authority, synchronization or conflict disposition | malformed, uncommitted, publicly authoritative |
| locally authoritative | node may decide within a signed/configured scope | globally/source authoritative |
| source authoritative | accepted source is authoritative for a named assertion/domain under current trust policy | infallible or current |
| unavailable | a named capability cannot presently be provided with required assurances | resource absent or denied |
| unknown | evidence is absent, conflicting, concealed, stale beyond decision use, or otherwise insufficient | unavailable, false, zero, empty |
| partial | result is complete only for an explicitly described authorized scope/watermark/source set | HTTP byte-range response |

### 6.2 Independent assessment axes

Every derived presentation assesses, as applicable: semantic validity; domain time; result/report/event time; receipt and commit time; freshness; completeness; authority; trust/quality; finality/tentativeness; source reachability; synchronization watermark/gaps; policy view; and evaluation time. No single `status` field may collapse them. **[A/P]**

### 6.3 Time fields

| Time | Meaning/use |
|---|---|
| `validTime` | description/assertion applicability; never cache expiry |
| `phenomenonTime` | when Observation value applies; may be future |
| `resultTime` | when result was obtained/generated; cannot be future under Part 2 |
| `issueTime` | when receiving System received a Command |
| `executionTime` | estimated/actual command interval by status context |
| `reportTime` | when CommandStatus report was generated |
| `eventTime` / JSON `time` | SystemEvent occurrence time |
| `sourceTime` | authenticated source assertion where distinct |
| `receivedAt` | first receipt at Glaux boundary |
| `committedAt` | local authoritative transaction time/order |
| `cachedAt` / `validatedAt` | representation or dependency-cache evidence |
| `lastContactAt` | observed communication fact, not System status |
| `lastSynchronizedAt` | completed sync fact; does not prove no later remote changes |
| `assessedAt` | one captured evaluation instant for the response/view |

Clock source, uncertainty/confidence, precision and node/epoch accompany decisions where time matters. An untrusted clock yields `unknown` or denies time-sensitive authority; it does not silently extend expiry. **[A/P]**

### 6.4 `RepresentationAssessmentV1`

Glaux should define an opt-in profile object or linked sidecar, not mutate mandatory CSAPI semantics:

```text
RepresentationAssessmentV1 {
  assessedAt,
  contentState: current | last-known | historical | tentative | partial,
  freshness: fresh | stale | unknown | not-applicable,
  latestEvidenceTime?, ageSeconds?, freshnessRuleRef?,
  completeness: complete-for-view | partial | unknown,
  authority: source | local | delegated-local | mixed | unknown,
  synchronization: current-to-watermark | backlog | gap | conflict | unknown,
  watermark?, warningCodes[], retryAfter?, statusLink?
}
```

`ageSeconds` is domain-evidence age under the named rule, not HTTP `Age`. `watermark` is opaque and policy-bound for normal clients. Omitted fields mean not disclosed/not applicable, never a favorable default. The assessment is computed after authorization and policy filtering so hidden facts cannot alter visible freshness, counts or watermarks. **[A/P]**

Standards-only representations remain valid CSAPI. Deployments advertise the assessment profile through media-type/profile negotiation and a URI-valued link relation; clients that do not opt in rely on standard timestamps and `live` without receiving invented standard fields. **[N/E/P]**

## 7. Resource Family Behavior Findings

| Resource family | Connected behavior | DDIL behavior | Freshness/validity signal | Client-visible posture | Security/command/stream implication | Audit/test/handoff |
|---|---|---|---|---|---|---|
| landing page | generated from enabled local capabilities | serve local build artifact if HTTP/auth safe | build/version validators | normal or coarse service-status link | never enumerate hidden adapters | parity/offline boot; 046/050 |
| conformance | declare implemented approved classes | unchanged unless capability truly disabled by deployment, not transient dependency | immutable release/config identity | no “offline conformance” claim | optional experimental profiles separate | declaration/runtime tests; 050–051 |
| OpenAPI/service docs | local immutable artifact matching routes/security | remain available; remote renderer/reference not required | ETag/build digest | cached/local is normal for pinned artifact | no credentials/internal topology | reference-closed tests; 046/050 |
| schemas/vocabularies | installed exact versions | use verified reference-closed package; new unknown package inactive | version/digest/trust/activation | cached package version may be exposed safely | command/data validation stays exact | rollback/corrupt/missing tests; 047/053 |
| Systems | authorized current logical revision/history | serve local authorized revisions; qualify dynamic projections | `validTime`, revision, assessment | current description may coexist with stale status | never infer reachability/command readiness | local/history tests; 043 |
| Deployments | valid-time and relationships | same local facts; delayed changes append as revisions | validity, commit/sync evidence | historical/current per standard time | physical presence not inferred from planned interval | late relocation test; 043 |
| Procedures | installed descriptions and history | safe to serve verified local version | `validTime`, version/digest | normal unless requested version unavailable | stale procedure cannot silently validate new write | package/version tests; 047 |
| Sampling Features/properties | authorized local resources | serve pinned semantics and historical snapshots | validity/revision/Observation time | unknown semantic package is explicit | hidden targets/properties stay concealed | offline-resolution tests; 043/053 |
| DataStreams | metadata, derived extents, `live` | history available; `live` reevaluated for named delivery, not inferred from last data | valid/phenomenon/result extents, assessment | archive may work while live unavailable | source and broker states separate | extent/live fault tests; 048/054 |
| Observations | committed authorized history/query/latest | retain valid late history; latest may be last-known/stale | phenomenon/result/receive/commit, assessment | 200 with honest assessment when usable | no hidden newer-fact leakage | ordering/gap fixtures; 043/053 |
| status values | derived current assessment plus history | last-known can be stale/unknown; source loss is separate | evidence/evaluation/freshness policy | never fabricate “unavailable” as sensor claim | stale readiness cannot authorize command | state-algebra tests; 048/055 |
| SystemEvents | durable occurrences on represented System | delayed events remain history; server link failure is not auto-event | event/source/receipt time | history with delayed flag/profile assessment | publish only authorized genuine event | event-versus-telemetry test; 043 |
| source registrations/trust | protected authoritative registry | local changes only in explicit authority class; incoming unknown staged/quarantined | trust/bundle epoch, expiry, sync state | normal clients see only authorized source assertion | stale/revoked/unknown affects admission | anti-rollback tests; 043/055 |
| policy/security records | protected PAP/PIP state | verified local bundle; no live fetch dependency at request time | issuer/audience/epoch/expiry/max age | no public policy internals | indeterminate narrows/denies | stale bundle tests; 047/055 |
| ControlStreams | schema/capability metadata | serve local definition but set/evaluate `live` honestly | validTime, schema version, assessment | definition availability does not mean dispatch | command-disabled posture may be concealed | command-profile tests; 055 |
| Commands/status/results | authoritative lifecycle history | accepted queued/local classes only; delayed reports append; ambiguity retained | issue/execution/report/receipt times | last valid public state plus safe assessment | no arrival-wins or invented terminal state | reconnect/effect tests; 043/055 |
| Feasibility | context-bound analysis and history | stale result remains history but is unusable as current safety evidence | evaluation/context digest/expiry | stale/indeterminate explicit in Glaux profile | rerun before command when required | context-change tests; 055 |
| audit records | protected journal/search/export | local E1–E5 durability and later verified export | node/epoch/sequence/time/gap/checkpoint | ordinary clients receive none | audit availability can gate effects | 041 contract; 043/049 |
| validation/raw artifacts | protected evidence by digest | exact installed contract; candidate staged if dependency missing | schema/profile/version/digest/outcome | stable safe error only | no uncontrolled reference fetch | offline/corrupt tests; 049/053 |

“Cache-safe” means policy-partitioned, integrity/version checked and accompanied by correct validators; it does not mean publicly cacheable. Protected or principal-specific responses use appropriate private/no-store controls. **[N/A/P]**

## 8. Operation Behavior Findings

### 8.1 Operation classification

| Operation | Nominal | DDIL default | May queue/stage when | Must disable/deny when | Response/audit |
|---|---|---|---|---|---|
| landing/conformance/OpenAPI/schema GET | local serve | continue | not needed | local artifact/integrity/auth unavailable | 200/304 or 503; artifact health |
| discovery/list/query | local authorized snapshot | continue with local scope/assessment | no implicit remote query queue | completeness required but unknowable and no partial profile | 200 qualified; never 206 for semantic partial |
| item retrieve | local authoritative revision | continue/history/last-known | no | item/view unavailable or policy indeterminate | 200/304, 404/conceal, or 503 |
| create/update metadata | transactional write | local-authority-only | durable tentative/staged profile with ID/status | no authority, schema, policy, audit or precondition | 201/200 when committed; 202 only durable operation; audit E3 |
| delete/retire/disposition | governed lifecycle | online-required by default | only explicit local disposition authority | hold/authority/sync uncertainty | 409/412/428 or durable result; audit E3/E4 |
| Observation/status ingest | authenticated source admission | bounded local ingest | quarantine when source/trust/schema state explicitly permits | unknown/expired source, contract or capacity beyond profile | commit response or stable reject; audit/outbox |
| source registration/trust update | protected admin | online-required | signed candidate quarantine only | cannot verify authority/epoch/rollback | never active on 202 alone; audit E3 |
| policy/config/admin | protected admin | deny by default | signed package import to inactive quarantine | freshness/approval/audit uncertain | stable problem; strong audit |
| stream subscribe | authorized snapshot/cursor | local snapshot/replay/live as available | checkpoint retained, not request queue | policy/lease/log range unavailable | session assessment/gap/resnapshot |
| outbound publish | outbox after commit | retain within bounded durable queue | policy/version/recipient bound and unexpired | expired, revoked, no capacity, policy indeterminate | delivery ledger; source state unchanged |
| command submit | accepted command profile | disabled unless explicit offline class | valid durable `PENDING` intent with bounds | no fresh authority/safety/schema/audit or no valid window | accepted CSAPI `201`; safe 409/503 before creation |
| command cancel/update | lifecycle/precondition | local only if exact authority and cutoff provable | cancellation request may remain private pending confirmation | terminal/cutoff/authority unknown | no forged `CANCELED`; audit decision |
| command dispatch | durable fenced effect | recheck after reconnect | already-approved valid queue only | any mandatory input stale/unknown; audit E4 unavailable | no network call; private reason |
| feasibility | context-bound evaluator | continue only with sufficient local inputs | async local work if profile permits | required live inputs absent | `INDETERMINATE`, `FAILED`, or pre-create 503 by actual outcome |

### 8.2 Required DDIL semantics matrix

| DDIL mode | Resource/operation | Normal behavior | Degraded behavior | Freshness/validity indicator | Client response | Policy/security | Command/control | Event/stream | Audit/event | Test/conformance | Handoff | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| limited | collection query | complete authorized page | bounded page/projection; optional partial authorized source set | assessed time/completeness/watermark | 200 + assessment | filter before completeness/count | none | smaller batches | operation summary if protected | no 206 misuse; latency/load | 046/054 | transport tuning is deployment-specific |
| intermittent | latest Observation | newest eligible result | local last-known, possibly stale | result/commit time + freshness rule | 200 + `last-known`/stale | hidden newer data excluded before selection | not readiness | cursor may lag | assessment decision | disconnect/reconnect boundary | 043/053 | `latest` remains max resultTime among visible candidates |
| disconnected | metadata read | local canonical revision | same, marked local/sync state if profile needs | revision/validTime/sync watermark | 200/304 | no broadened view | capability metadata not permission | local events only | protected read rule | offline corpus | 043/050 | cached transport copy separate |
| local-only | authorized ingest | commit local fact | locally authoritative within bundle | source/node/epoch/commit and bundle | normal commit response | exact source/action scope | no implied tasking | local outbox | E3 | source/bundle expiry | 043/055 | later sync cannot erase historic local decision |
| disconnected | external-source query requiring completeness | federated complete result | local partial view only if advertised | completeness partial/unknown | 200 qualified or 503 | do not reveal missing/hidden source | none | gap may be protected | dependency outcome | partial-vs-hidden test | 043/056 | normal caller sees no topology |
| broker down | stream/publication | live delivery | HTTP truth available; outbox queues within limits | outbox watermark/expiry protected | stream unavailable/resnapshot later | recipient policy rechecked at send | broker ACK not effect | explicit discontinuity | delivery attempts | outage/restart/expiry | 043/048/054 | domain commit remains authoritative |
| IdP/PDP down | protected read | fresh decision | bounded local verification only | credential/bundle expiry/max age | normal or generalized deny | no last-known allow | effects stricter | lease closes at bound | offline decision | stale/revoked/rollback | 046/055 | detail admin-only |
| schema origin down | validate ingest | exact active schema | installed package only | schema digest/version | accept exact or stable error | no runtime fetch/SSRF | exact ControlStream schema | payload retains pin | validation outcome | network-blocked test | 047/053 | unknown new schema quarantined |
| command gateway down | Command submit/dispatch | accept/dispatch by profile | reject or bounded queue; never claim target acceptance | validity, last contact, target evidence | pre-create 409/503 or `PENDING` | fresh authorization required at dispatch | no duplicate effect | status remains last defensible | E4/unknown outcome | crash/gateway reconnect | 043/055 | source/system/gateway state separate |
| recovering | all queued effects | current policies and paths | tighten/revoke first; verify; re-evaluate; release selectively | recovery phase/gaps/conflicts | coarse recovering/queued state | no authority expansion before refresh | revalidate each effect | catch-up then live | receipt/conflict events | reconnect storm/order | 043/054 | connectivity alone is insufficient |

### 8.3 Partial results

A semantically partial collection returns `200` only when the advertised Glaux profile lets the client understand the exact authorized completeness boundary. HTTP `206 Partial Content` is for range transfer and must not mean “some upstreams answered.” If the operation contract requires completeness and Glaux cannot establish it, return `503` or a stable profile-specific problem rather than a misleading empty/complete page. **[N/E/P]**

## 9. Observation, Status, Latest-Value, Source-Health, and Delayed-Update Findings

### 9.1 Observations and latest

Part 2 Observation `phenomenonTime` says when a value applies; `resultTime` says when obtained/generated. Requirement 50's `resultTime=latest` selects all Observations tied at the greatest result time after route scope, authorization and filters. It does not mean latest arrival, latest phenomenon time, one item per DataStream, current source state, or fresh. **[N/A]**

DDIL rules:

- preserve source/message/epoch/sequence, domain time, receipt, commit, digest and contract pin;
- accept valid delayed facts into history even when they do not change the current projection;
- apply corrections and current selection by accepted source/temporal rules, never reconnect arrival order;
- compute extents/latest/current against one authorized snapshot/watermark;
- surface last-known/stale/partial through the opt-in assessment, not by altering Observation time; and
- return an empty authorized set only when it is truly empty for the described view, not when source completeness is unknown.

### 9.2 Status and availability

Status is an Observation-like fact. Availability is a capability-specific derived assessment with `available`, `degraded`, `unavailable`, `unknown`, or `not_applicable`. When contact is lost, Glaux changes **source reachability** and recomputes the assessment; it does not write a synthetic sensor status saying the represented System is unavailable. A System may still operate while disconnected, and a reachable source may report a failed System. **[A/P]**

`DataStream.live` reports whether live data is available from that DataStream under the implementation/profile. It is independent of archive availability, latest freshness, broker health and command acceptability. The derivation rule and assessment time must be stable and tested. **[N/A/P]**

### 9.3 Delayed, replayed and out-of-order input

| Input condition | Historical record | Current projection | Publication | Diagnostic |
|---|---|---|---|---|
| identical replay ID/digest | deduplicate | unchanged | no duplicate logical change | replay/dedupe evidence |
| same ID, different bytes | quarantine conflict | unchanged/unknown if material | integrity notice only if authorized | protected incident |
| older domain time, valid | retain | normally unchanged | late-history event if profile | delayed=true with clocks |
| newer eligible domain fact | retain | update transactionally | current-change after commit | source/projection evidence |
| sequence gap | retain surrounding facts | completeness/gap may become unknown | gap/resnapshot semantics | exact/bounded gap |
| invalid/unauthorized | reject/quarantine | unchanged | none | protected reason |

No automatic backdating, time substitution or “now” stamping may make delayed evidence appear current. **[A/P]**

## 10. Streaming, Event, Cursor, Reconnect, and Latest-State Snapshot Findings

### 10.1 Continuity model

The accepted IDR-SRV-035 contract remains controlling: authorize a point-in-time snapshot, bind it to a logical watermark and opaque policy-bound cursor, replay committed events after that watermark, then follow live delivery. The cursor binds route/scope, filter, representation/profile, policy context and log position. Reauthorization occurs on reconnect and at lease/policy changes. **[A]**

### 10.2 Transport versus logical resume

MQTT 5 can retain session state when Session Expiry Interval is greater than zero, expires messages by Message Expiry Interval, constrains in-flight QoS 1/2 with Receive Maximum, and provides at-least-once delivery at QoS 1. Those are useful adapter mechanisms. They do not define Glaux domain identity, committed-log retention, cross-adapter position, source ordering, authorization validity or conflict behavior. MQTT packet/session state is therefore never the canonical cursor. **[N/E/P]**

The experimental Part 3 profile remains optional, version-pinned, disabled by default and not an approved conformance claim. Its Resource Events are delivery projections; Resource Data retains native CSAPI semantics. Broker disconnect must not cause the server to discard a committed resource change merely because publication failed. **[A/P]**

### 10.3 Gaps, reconnect and resnapshot

| Condition | Required behavior |
|---|---|
| cursor valid and retained range available | reauthorize; replay after last acknowledged logical position; tolerate boundary duplicate |
| cursor expired or range compacted | return typed resnapshot-required outcome; issue new authorized snapshot/watermark |
| known hidden events inside range | advance opaque authorized cursor without leaking hidden count/ID |
| unknown integrity/log gap | disclose a policy-safe gap/continuity-unknown state; do not claim complete replay |
| profile/filter/policy changed | re-evaluate; new snapshot unless equivalence is proven |
| broker session survived but Glaux cursor did not | resnapshot; transport state cannot override application truth |
| reconnect storm/slow consumer | bounded admission/backoff/load shedding; preserve mandatory log/outbox evidence |

Latest-state snapshots are derived authorized views, not replacements for history. They state evaluation time, watermark, visible completeness, freshness and policy/profile. If a newer hidden fact exists, it must not change the visible count, latest selection, gap wording or timing in a way that leaks existence. **[A/P]**

## 11. Command, Feasibility, Command-Status, and Unknown-Outcome Findings

### 11.1 DDIL command classes

| Class | Admission | Queue/dispatch | Required evidence |
|---|---|---|---|
| online-required | reject/temporarily unavailable without current remote inputs | none | current identity/policy/trust/safety/target/gateway |
| bounded local authority | accept only exact pre-authorized action/target/parameters/window | durable queue; full pre-dispatch recheck | signed local grant/bundle, time confidence, local safety, E4 audit |
| pre-authorized emergency | narrow independently approved protective action only | single-use/fenced, strongly audited | explicit non-generic grant, approvals, non-overrideable interlocks |
| simulation/test-only | isolated synthetic namespace | simulator adapter only | cryptographic/config separation from real targets |

The deployment selects the class; callers cannot request a weaker class. **[A/P]**

### 11.2 Admission, queue and dispatch

A Command may be accepted while a target is unreachable only if the ControlStream profile advertises bounded deferred execution, the command has an enforceable deadline/validity, idempotency and target identity are stable, local policy/safety authority covers the exact request, durable Command/`PENDING`/work/audit records commit, and the client is told processing—not effect—was accepted. The accepted CSAPI contract returns `201 Created` with the canonical resource and initial status; `202` is not used merely because processing is asynchronous. **[N/A/P]**

Before any dispatch, including after a brief outage, Glaux rechecks command revision, expiration, cancellation/supersession, actor authority, policy/trust/source/gateway epochs, target binding, required feasibility, current safety/interlock evidence, lease/fence and E4 audit capacity. Any mandatory unknown/stale input denies dispatch. **[A/P]**

### 11.3 Feasibility

Feasibility is an analysis bound to input digest, target/ControlStream version, policy/context, evaluator, evidence set and evaluation/expiry time. A stale result remains historical evidence but cannot become fresh through a new authorization decision. If inputs are insufficient, a successful analysis may return `COMPLETED` plus an advertised `INDETERMINATE` result; a technical evaluation failure is `FAILED`; work never accepted is `REJECTED` or a pre-create HTTP error as appropriate. **[A/P]**

### 11.4 Status and unknown outcome

CommandStatus `reportTime` and legal lifecycle remain authoritative. Delayed valid nonterminal reports may append and advance only through the transition algorithm. No report may overwrite a terminal state. If contact is lost after possible dispatch/effect, Glaux retains the last valid public status and private `delivery_uncertain`/`reconciliation_required` evidence. It must not retry a non-idempotent effect under a new Command ID or invent `FAILED`, `CANCELED` or `COMPLETED` for convenience. **[N/A/P]**

Resolution order is target query by stable identity; verified buffered status/result evidence; adapter-specific retry only when effect-idempotence is proven; authorized operator disposition; or persistent uncertainty. Broker ACK proves transport receipt only. **[A/P]**

## 12. Authentication, Authorization, Source Trust, Policy, Credential, and Redaction Findings

### 12.1 Local security bundle

An offline-capable node uses integrity-protected, anti-rollback bundles containing issuer/trust chain, audience/node/domain, credential-validation keys and algorithms, subject/client/source mappings, permitted resource/action/parameter scopes, policy/safety/schema versions, sequence/epoch, activation/expiry, maximum offline age, revocation state, time-confidence requirement, offline operation classes, dependencies and fail-safe behavior. Sensitive bundles are encrypted/protected at rest as deployment policy requires. **[A/P]**

### 12.2 Decision rules

- Locally verifiable tokens/certificates never outlive their own expiry and are accepted only for the bundle's audience and offline class.
- Cached decisions are keyed to subject, credential, client/workload, resource revision, action, context, policy/trust epochs and time; they are not generic “allows.”
- Stale, expired, rollback, unverifiable or conflicting policy/trust/revocation evidence produces `INDETERMINATE`, then deny, conceal or quarantine under the accepted safe-failure contract.
- Authority never broadens because remote services are unreachable.
- Reconnection imports tightening/revocation/anti-rollback state and re-evaluates sessions, streams, queues and effects before any expansion.
- Historic decisions retain the bundle/version actually used; later central disagreement does not falsify that local act.

### 12.3 Source trust and publishers

| Source state | Admission behavior |
|---|---|
| active, mapped, bundle fresh | accept within exact resource/schema/rate/time/replay scope |
| stale registry but within explicit offline bound | accept only declared bounded-local class and mark decision evidence |
| suspended/revoked | reject; quarantine only if policy requires evidence preservation |
| unknown/unmapped | reject or privileged quarantine; never promote to authoritative |
| simulated/test | accept only isolated namespace/profile; never merge with operational source |
| epoch/sequence rollback or identity collision | quarantine and incident; current projection unchanged/unknown as applicable |

Source lag and last contact are protected operational evidence. Public source authority claims name only the accepted source/provenance safe for that view; they do not expose registry internals or imply truth/currency. **[A/P]**

### 12.4 Redaction under degradation

Policy filtering precedes selection, count, completeness, freshness, watermarks, errors and timing-sensitive work. If a required policy input is unavailable, Glaux uses the safe deny/conceal result; it never returns unredacted data because the transform/PDP is down. A pre-authorized offline transform must be deterministic, locally installed, versioned and bound to the active policy package. Cached transformed views retain their input/binding/version and are re-evaluated before release. **[A/P]**

## 13. Error, Warning, Problem-Detail, Response Metadata, Diagnostic, and Audit Findings

### 13.1 HTTP mapping

| Condition | HTTP/status behavior | Required context |
|---|---|---|
| usable complete or honestly qualified local result | `200 OK` (or `304`) | standard timestamps/validators; optional assessment/profile |
| newly committed resource including async Command/Feasibility | `201 Created` | `Location`; accepted lifecycle body/links |
| durable non-resource operation accepted but incomplete | `202 Accepted` | operation/status URI; no completion promise |
| semantically partial authorized result | `200`, not `206` | assessment describes exact visible completeness boundary |
| resource/current-state conflict | `409 Conflict` | safe stable conflict class |
| write precondition missing/false | `428` / `412` | safe reconciliation instruction |
| temporarily unable to satisfy required assurance | `503 Service Unavailable` | `Retry-After` only when an honest estimate exists |
| upstream attempt timed out and distinction is safe | `504 Gateway Timeout` optionally | do not name/probe hidden dependency |
| denied/concealed | `401`/`403` or policy-selected `404` | same concealment posture online/offline |
| cursor/log continuity unavailable | stable problem or `409`/`410` per profile | resnapshot link/instruction without hidden range detail |

`203 Non-Authoritative Information` is not a general stale-data status. `206` is not a multi-source completeness signal. `Retry-After` is guidance, not proof of recovery time. **[N/E/P]**

### 13.2 Warnings and response metadata

RFC 9111 obsoletes the generic HTTP `Warning` header. Glaux therefore uses standard `Date`, `Age`, `Cache-Control`, `Expires`, `ETag` and `Last-Modified` only for their HTTP meanings, plus an advertised profile representation/sidecar for domain state. A successful standards-only payload is never silently modified with ambiguous fields. **[N/P]**

Stable warning codes include `last-known`, `stale-evidence`, `freshness-unknown`, `partial-visible-scope`, `sync-backlog`, `continuity-gap`, `tentative-local`, `command-disabled`, `resnapshot-required`, and `time-confidence-insufficient`. Each code is policy-filtered and documented; free text is optional and non-parseable. **[P]**

### 13.3 Safe problem details and diagnostics

RFC 9457 problems use stable `type`, `title`, HTTP `status`, corrective `detail`, opaque `instance`/correlation and bounded extensions such as `retryable`, `retryAfter`, `stateCode`, `statusLink` and `resnapshotRequired`. They never expose dependency host/type, internal route, peer/source identity, queue depth, hidden count/ID, policy rule, credential validation step, command capability, safety reason, synchronization range, stack, SQL or filesystem detail. Protected admin diagnostics hold those details under separate authorization and audit. **[N/A/P]**

### 13.4 Audit and accountability

Audit DDIL mode/profile changes; dependency transitions that affect policy or protected service; offline bundle install/activate/reject/rollback; locally authorized decisions; stale/indeterminate blocks; queued/staged/expired/released work; command pre-effect/outcome uncertainty; stream gap/resnapshot; source epoch/gap/collision; reconnect verification; sync accept/quarantine/conflict; and audit capacity degradation. Use the accepted E0–E5 class and no-secret schema. **[A]**

Local E3/E4 actions require durable local audit before state/effect. Remote audit export may wait; the mandatory local floor may not. If all approved durable paths are exhausted, class-specific admission stops before silent loss. Public responses disclose only the operation outcome and safe correlation; audit topology/capacity is protected. **[A/P]**

## 14. Synchronization/Conflict Handoff Findings

### 14.1 Semantic inputs fixed for IDR-SRV-043

Every synchronized candidate must preserve:

- stable resource/event/message/operation identity and resource revision;
- origin node/security domain, source/publisher/executor authority and node/boot/source epochs;
- source/domain sequence plus occurrence, result/report, receipt, commit and synchronization times;
- exact schema/profile/vocabulary/encoding versions and canonical digest;
- provenance, policy binding/decision, offline authority class, credential/trust/bundle epochs and time confidence;
- lifecycle state, tombstone/disposition/hold, command attempt/fence, idempotency and causal dependencies;
- local current-selection/freshness evidence without treating that selection as universal truth;
- range watermark, known/unknown gaps, checkpoints and integrity state; and
- tentative/quarantine/conflict status and protected reason.

### 14.2 Required receiving outcomes

IDR-SRV-043 must produce explicit `identical-deduplicated`, `accepted-history`, `accepted-current`, `tentative`, `quarantined`, `conflict`, `gap`, `rejected-policy`, `rejected-trust`, or `unverifiable` results. It may automate only proven cases such as identical identity/digest, compatible immutable fact, profile-defined monotonic single-authority sequence, or demonstrably commutative addition. **[A/P]**

Arrival time, highest node clock, newest UUID, central-node preference, last writer, broker order or “most restrictive” guessed vocabulary may not silently decide divergent identity, parentage, authority, contract, validity, command state, delete/update or policy conflicts. Resolution is a new authorized provenance/audit activity that preserves both inputs. **[A/P]**

### 14.3 Boundary retained

This report does not choose vector clocks, Merkle trees, range negotiation, delta encoding, compression, partition ownership, conflict UI, tombstone duration, anti-entropy schedule or transport. IDR-SRV-043 selects those mechanics while preserving the semantic distinctions above. Numeric horizons also coordinate with lifecycle/deployment research.

## 15. Fixture, Conformance, Security Testing, Performance, Deployment, Observability, and Interoperability Test Implications

### 15.1 Verification matrix

| Fixture/test | Layer | Expected falsifiable outcome |
|---|---|---|
| nominal-to-limited hysteresis | unit/fault | no flap; protected transition; correct coarse assessment |
| simultaneous source-down/broker-up | integration | history works; no source-live claim; broker health not substituted |
| intentional local-only profile | configuration | not reported as fault; authority remains exact and bounded |
| reconnect before bundle refresh | security | service remains recovering; no authority expansion |
| static artifacts with internet blocked | deployment | landing/OAS/conformance/schemas serve from reference-closed package |
| corrupted/rollback schema package | supply chain | package rejected/quarantined; prior active version retained |
| HTTP cache fresh/domain stale | API | `Age` and domain assessment remain independent |
| HTTP cache stale/domain fresh | API | cache revalidation behavior does not rewrite domain time |
| latest Observation stale | API | correct max visible resultTime returned with stale/last-known assessment |
| equal latest result times | API | all tied visible observations returned |
| hidden newer Observation | security | visible latest/count/timing does not reveal hidden fact |
| delayed valid Observation | ingestion | retained in history; projection changes only by accepted selector |
| replay identical message | ingestion | idempotent dedupe; no duplicate logical change |
| same ID different bytes | integrity | quarantine/conflict; no overwrite |
| known/unknown sequence gap | DDIL | exact/bounded or unknown gap retained; no completeness claim |
| source unreachable vs System unavailable | semantics | distinct assessments; no fabricated System status |
| `live=true` with stale latest | API | both values represented independently |
| semantically partial query | HTTP | 200 plus advertised assessment; never 206 |
| complete-required query with source loss | HTTP | safe 503, not misleading empty success |
| RFC 9457 leakage corpus | security | no dependency/policy/source/command/topology internals |
| IdP down, locally valid bounded token | security | only declared offline class proceeds until bound |
| token/bundle expired or clock unknown | security | time-sensitive protected operation denied |
| revocation epoch rollback | security | bundle rejected; incident/audit appended |
| stale policy transform | policy | no unredacted fallback; deny/conceal/quarantine |
| stale source registry | ingestion | only explicit bounded class or quarantine; no assumed trust |
| broker outage after domain commit | streaming | domain record persists; outbox retains/ages/expires visibly |
| MQTT session present, logical cursor expired | streaming | resnapshot required; session cannot claim continuity |
| reconnect boundary duplicate | streaming | client/logical ID dedupes; no missing event |
| policy/filter change during disconnect | streaming | reauthorize; new snapshot unless equivalence proven |
| hidden events across cursor | security | cursor advances without hidden count/ID leakage |
| command admission without offline class | command | no Command/effect; stable concealed 409/503 as configured |
| bounded queued command | command | durable `PENDING`, exact expiry/authority/audit; no dispatch yet |
| policy/revocation change before dispatch | command | queued command blocked and audited |
| stale feasibility/target status | command | cannot satisfy mandatory dispatch gate |
| gateway loss after possible effect | command | last valid public status retained; private unknown; no new-ID retry |
| delayed command completion after nonterminal | command | accepted only if legal/authorized and correlated |
| delayed completion after terminal cancel | command | conflict/incident; public terminal unchanged |
| local audit primary failure | audit | accepted E1–E5 spool/fail behavior; no mandatory silent loss |
| storage reserve exhaustion | resilience | low-priority work sheds; protected writes/effects stop by class |
| recovery import duplicate/conflict | synchronization | identical dedupe; divergent bytes quarantine; arrival never wins |
| reconnect backlog burst | performance | measured catch-up, p50/p95/p99, bounded memory and priority fairness |
| long outage beyond retention/expiry | performance/recovery | explicit loss/resnapshot/expiry outcomes; no false recovery |
| CSAPI Explorer/generic client | interoperability | standards payload remains parseable; optional profile safely ignorable/negotiated |
| CS-Go/OSH transport adapter | interoperability | field/topic differences mapped; no imported durability claim |

### 15.2 Conformance boundary

Approved Parts 1/2 tests run independently of DDIL extensions. A conformance fixture may select a locally self-contained profile with authorized principals and deterministic dependencies, but transient outages cannot excuse malformed standard resources. Glaux DDIL assertions are project contract tests with requirement IDs, not new OGC conformance URIs. Experimental Part 3 remains separately declared and pinned. **[N/A/P]**

### 15.3 Performance and observability

Measure local query latency and cache-hit correctness; ingestion and audit capacity; queue/spool growth; oldest age and expiry; projection/outbox/sync lag; reconnect catch-up throughput; cursor resnapshot rate; replay duplicate/gap/conflict rate; bundle/policy age; time confidence; stream fan-out and slow consumers; command queue expiry/recheck latency; and recovery RPO/RTO. Report p50/p95/p99, workload/data/profile, outage duration, payload sizes, policy cost, storage/network and failure injection. No numeric pass threshold is invented here. **[P/X]**

Metrics use bounded mode/dependency-class/outcome labels, never resource, source, principal, command, peer, policy term or hidden topology IDs. Alerts cover required dependency transitions, stale/rollback security material, gap/collision, queue/spool reserve, audit-path loss, command uncertainty and recovery stalls. Telemetry is not public source/System status and is not authoritative audit. **[A/P]**

### 15.4 Client expectations

- Generic CSAPI clients continue to receive standards-valid resources and standard time/status semantics.
- Glaux-aware web/mobile clients can negotiate/read assessments, render “last known as of…,” distinguish server/source/System states, and require user confirmation for qualified data.
- Publishers/adapters persist stable IDs, source epoch/sequence, domain time and contract pin and treat retries as idempotent replay.
- Command gateways persist attempt/status evidence, never map transport ACK to domain acceptance, and reconcile stable command identity.
- CSAPI Explorer/OS4CSAPI clients are interoperability probes, not reasons to weaken normative output.

## 16. Downstream Topic Handoff Matrix

| Topic | Required handoff |
|---|---|
| IDR-SRV-043 | modes/states, `RepresentationAssessmentV1`, origin/source/node epochs, clocks, watermarks/gaps, tentative/quarantine/conflict outcomes, no-arrival-wins and queued-effect recheck |
| IDR-SRV-044–045 | encoding-neutral DDIL types, ports/state machines, local bundle/status/queue interfaces, profile negotiation, separation of domain/transport/telemetry |
| IDR-SRV-046 | topology-specific dependency graph, offline-capable profiles, capacity/RPO/RTO, numeric thresholds and protected admin endpoints |
| IDR-SRV-047 | mode/profile config, bundle/package/signature/epoch/expiry management, safe defaults and anti-rollback |
| IDR-SRV-048 | metrics, traces, health, alerts, hysteresis, lag/capacity/time-confidence and public-versus-admin diagnostic separation |
| IDR-SRV-049 | reference-closed backup/restore, queue/spool/log/package preservation, tombstone/gap/verification continuity |
| IDR-SRV-050–051 | separate OGC conformance from Glaux DDIL/profile assertions; requirement IDs, deterministic outage lanes and evidence manifests |
| IDR-SRV-052–053 | reusable fault clock/network/dependency harness and every Section 15 fixture/golden assessment/problem |
| IDR-SRV-054 | constrained bandwidth, long outage, queue growth, reconnect storm, catch-up/replay/resnapshot and capacity benchmarks |
| IDR-SRV-055 | stale/rollback credentials/policy/trust, hidden-state leakage, offline source/command authority, audit failure and double-effect tests |
| IDR-SRV-056 | generic and Glaux-aware client behavior; CS-Go/OSH/SECD adapters; Part 3 profile and fallback behavior |
| IDR-SRV-057 | accepted DDIL claim, explicit residual uncertainty and numeric/product/protocol decisions still owned downstream |

## 17. Recommendations

1. Model DDIL as scoped orthogonal facts and derive per-operation service posture; never persist one global offline boolean.
2. Implement `FreshnessAssessment` and `RepresentationAssessmentV1` as encoding-neutral domain/profile types with explicit evaluation time, rule/version and protected evidence references.
3. Preserve standard timestamps and selectors exactly; never overload `validTime`, `resultTime`, `live`, HTTP `Age`, `203`, `206` or CommandStatus to carry unrelated DDIL meaning.
4. Keep landing, OpenAPI, conformance, schemas and vocabularies reference-closed and locally available, with integrity/version checks and no runtime public dereference.
5. Serve authorized local history through outages; qualify last-known/current/partial views after policy filtering and fail when a required completeness claim cannot be supported.
6. Separate source reachability, represented-System status, API health, broker state and command readiness in storage, API and UI contracts.
7. Permit local writes/ingestion only under an explicit bounded authority class with exact schema, source, policy, transaction, audit, idempotency, capacity and time rules.
8. Keep administration connected-authority-required by default; allow only narrow signed recovery/import actions offline.
9. Use the accepted snapshot-watermark/logical-cursor/replay/resnapshot streaming contract; treat MQTT session/QoS/retain/expiry as adapter mechanisms only.
10. Disable real command dispatch by default under DDIL. Any bounded local class requires pre-positioned authority, valid time, safety evidence, fenced identity, durable E4 audit and complete pre-dispatch re-evaluation.
11. Preserve unknown command outcomes and conflicting delayed reports; never retry uncertain non-idempotent effects under a new identity.
12. Apply authorization/redaction before selection, freshness, completeness, counts, cursors and diagnostics, and expose only coarse non-topological public state.
13. Make reconnect a recovery state that processes tightening/revocation first, verifies evidence, re-evaluates queues/sessions/effects and exits only when declared invariants hold.
14. Implement deterministic fault/time/dependency injection and the Section 15 matrix before claiming any offline-capable deployment profile.

## 18. Risks, Constraints, and Open Questions

### 18.1 Risks and controls

| Risk | Consequence | Control/owner |
|---|---|---|
| one global degraded flag | unsafe broad behavior and misleading UI | orthogonal scoped state; 044–048 |
| stale shown as current | operational/safety error | assessment + timestamps + fixtures |
| assessment leaks hidden newest/source topology | inference/cross-boundary disclosure | filter first; coarse codes; 040/055 |
| cached authorization extends authority | unauthorized read/write/effect | hard expiry/epochs/context keys/no last-known allow |
| queue implies eventual success | duplicate/expired/unsafe work | durable status, validity and send-time recheck |
| transport guarantee mistaken for domain guarantee | missing/duplicate data or effects | logical cursor/outbox/idempotency/fence |
| reconnect releases backlog before revocation | disclosure/effect after tightening | tightening-first recovery state |
| clock uncertainty extends expiry | invalid authority/freshness | time-confidence gates and conservative unknown |
| local storage/audit exhaustion | silent evidence loss or denial | reserves, admission classes, alert/fault tests |
| custom profile breaks generic clients | interoperability failure | explicit negotiation/sidecar; standard payload unchanged |
| partial success uses 206 | incorrect HTTP/client behavior | 200 assessment or 503; contract test |
| sync mechanics erase semantic distinctions | arrival-wins/anti-resurrection failure | mandatory IDR-SRV-043 input/outcome contract |

### 18.2 Open parameters and owners

| Open question | Why unresolved | Owner/decision trigger |
|---|---|---|
| numeric freshness per resource/capability | mission/deployment-specific risk | 046/047 with operational authority |
| maximum offline credential/policy/trust age | assurance and clock depend on deployment | 046/055 security assessment |
| public assessment wire shape/media profile | needs client prototype/schema review | 044/050/056 |
| which metadata writes are locally authoritative | depends on partition/ownership model | 043/046 |
| partial-result source-set disclosure | policy/topology sensitivity varies | 040/046/055 |
| queue/spool sizes and expiry/priority | workload/storage/bandwidth evidence absent | 046/054 benchmarks |
| exact recovery exit criteria | depends on topology and sync protocol | 043/046 |
| command consequence classes/offline eligibility | requires operational safety authority | 046/047/055; disabled until supplied |
| tombstone/idempotency/log retention horizons | depends on maximum replay/outage and lifecycle | 030/043/049 |
| Part 3 migration from experimental profile | draft remains mutable | adopted standard/ATS or material draft change |

No open parameter prevents the semantic baseline. Safe defaults are narrower service, no authority expansion, no unqualified currency claim and no real DDIL command dispatch.

## 19. Validation Against This Plan's Success Criteria

| Success criterion | Result and evidence |
|---|---|
| DDIL modes and degraded service states identified with anchors | Met: Sections 3–5 distinguish source-backed primitives and proposed multi-dimensional modes. |
| Freshness, validity, last-known, cached, tentative, delayed, unknown, unavailable and stale defined | Met: Section 6 supplies definitions, axes, time fields and profile object. |
| Resource and operation behavior documented | Met: Sections 7–8 cover every named family and operation class. |
| Observation/status/latest, streams/events, commands/feasibility, source trust, policy, credential and audit implications | Met: Sections 9–13 provide explicit rules and failure behavior. |
| Response metadata, warnings/problems, diagnostics and leakage avoidance | Met: Sections 6.4, 8.3, 12.4 and 13 define negotiation, HTTP mapping and safe detail. |
| IDR-SRV-043 synchronization/conflict handoff explicit | Met: Section 14 fixes mandatory semantic inputs/outcomes and preserves the mechanics boundary. |
| Fixture, conformance, security, performance, deployment, observability and interoperability implications | Met: Sections 15–16 define 44 falsifiable cases and topic handoffs. |
| Implementation/community lessons incorporated as non-normative evidence | Met: Sections 3.1, 10 and 15 retain OSH/CS-Go/pygeoapi/SECD/client lessons without promoting them. |
| Recommendations decision-usable and server-bounded | Met: Sections 1, 17 and 18 select behavior while deferring topology/products/client-device design. |
| References explicit and reproducible | Met: Section 20 pins mutable sources and links accepted reports and primary standards. |

All internal completion gates are satisfied. Project-lead acceptance remains open. This report does not authorize IDR-SRV-043, implementation, production numeric thresholds, deployment topology, an operational policy/credential package, real command effects, or an OGC Part 3 conformance claim.

## 20. References

### 20.1 Primary standards and protocol sources

- OGC API - Connected Systems Part 1, OGC 23-001 Version 1.0: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems Part 2, OGC 23-002 Version 1.0: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems repository, approved tag `v1.0.0`: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0
- OGC API - Connected Systems repository head checked at `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`: https://github.com/opengeospatial/ogcapi-connected-systems/commit/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f
- Part 3 working-draft head checked at `6f529a15bfa63259febc3620378d3e5a06305333`: https://github.com/opengeospatial/ogcapi-connected-systems/commit/6f529a15bfa63259febc3620378d3e5a06305333
- OGC SensorML 3.0, OGC 23-000: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model 3.0, OGC 24-014: https://docs.ogc.org/is/24-014/24-014.html
- RFC 9110, HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110.html
- RFC 9111, HTTP Caching: https://www.rfc-editor.org/rfc/rfc9111.html
- RFC 9457, Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457.html
- RFC 3339, Date and Time on the Internet: https://www.rfc-editor.org/rfc/rfc3339.html
- MQTT Version 5.0, OASIS Standard: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html

### 20.2 Accepted project evidence

- [IDR-SRV-001 — STANAG 4789 / AEP-4789 Server Obligation Baseline](idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md)
- [IDR-SRV-002 — AEP-4789 Volume I Functional Mapping](idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md)
- [IDR-SRV-014A — OSH Implementation Study](idr-srv-014a-osh-csapi-server-implementation-study-report.md)
- [IDR-SRV-014B — Connected Systems Go Implementation Study](idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md)
- [IDR-SRV-014C — pygeoapi Implementation Study](idr-srv-014c-pygeoapi-csapi-server-implementation-study-report.md)
- [IDR-SRV-014D — SECD Implementation Study](idr-srv-014d-secd-csapi-server-implementation-study-report.md)
- [IDR-SRV-014E — OS4CSAPI Client Smoke-Test Study](idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md)
- [IDR-SRV-014F — SECD Interoperability Study](idr-srv-014f-secd-interoperability-findings-study-report.md)
- [IDR-SRV-014G — OS4CSAPI Discussions Study](idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md)
- [IDR-SRV-014H — Draft Part 3 and Implementation Study](idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md)
- [IDR-SRV-018 — Temporal Validity and Freshness Model](idr-srv-018-temporal-validity-and-freshness-model-report.md)
- [IDR-SRV-020 — Status, Availability, and System Event Model](idr-srv-020-status-availability-and-system-event-model-report.md)
- [IDR-SRV-023 — Schema and Encoding Validation Strategy](idr-srv-023-schema-and-encoding-validation-strategy-report.md)
- [IDR-SRV-029 — Transaction, Consistency, Idempotency, and Concurrency Strategy](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md)
- [IDR-SRV-030 — Data Lifecycle, Retention, Archival, and Deletion Strategy](idr-srv-030-data-lifecycle-retention-archival-and-deletion-strategy-report.md)
- [IDR-SRV-034 — DataStream, Observation, and Status Update Semantics](idr-srv-034-datastream-observation-and-status-update-semantics-report.md)
- [IDR-SRV-035 — Streaming and Event Publication Strategy](idr-srv-035-streaming-and-event-publication-strategy-report.md)
- [IDR-SRV-036 — Control Stream and Command Lifecycle Model](idr-srv-036-control-stream-and-command-lifecycle-model-report.md)
- [IDR-SRV-037 — Feasibility and Asynchronous Tasking Strategy](idr-srv-037-feasibility-and-asynchronous-tasking-strategy-report.md)
- [IDR-SRV-038 — Command Authorization, Safety, and Audit Strategy](idr-srv-038-command-authorization-safety-and-audit-strategy-report.md)
- [IDR-SRV-039 — Authentication, Authorization, and API Security Threat Model](idr-srv-039-authentication-authorization-and-api-security-threat-model-report.md)
- [IDR-SRV-039A — Zero-Trust Architecture Alignment and Enforcement Model](idr-srv-039a-zero-trust-architecture-alignment-and-enforcement-model-report.md)
- [IDR-SRV-040 — Policy, Releasability, and Cross-Boundary Access Constraints](idr-srv-040-policy-releasability-and-cross-boundary-access-constraints-report.md)
- [IDR-SRV-041 — Audit Logging and Accountability Strategy](idr-srv-041-audit-logging-and-accountability-strategy-report.md)

The controlled NATO package is cited by identifier, date, digest and accepted source reports and is intentionally neither linked nor reproduced. Mutable sources were checked September 15, 2026. Their current behavior is informative unless an approved, pinned standard or accepted Glaux decision gives it greater authority.

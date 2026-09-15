# Section 036: Control Stream and Command Lifecycle Model - Research Report

**Topic ID:** IDR-SRV-036<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-036 Control Stream and Command Lifecycle Model](../IDR%20Plans/idr-srv-036-control-stream-and-command-lifecycle-model.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions and detailed questions concerning ControlStream and Command meaning, payloads, lifecycle, feasibility, authorization/safety hooks, dispatch, status/result reporting, cancellation, timeout, DDIL, persistence, events, and verification<br>
**Methodology Used:** Authority-ranked standards and accepted-baseline synthesis; immutable source and implementation pins; resource/actor/authority taxonomy; external-versus-internal state-machine analysis; failure-window, DDIL, security, and interoperability analysis; lifecycle and handoff matrices<br>
**Research Time:** Approximately 17 hours of AI-assisted execution on September 15, 2026<br>
**Approved Standards Baseline:** OGC 23-001 and OGC 23-002 Version 1.0, repository tag `v1.0.0` at [`8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Implementation Evidence:** CS-Go `v1.0.4` at [`244f4dd586da685d4d9b75e43f73001028b5bd0e`](https://github.com/SomethingCreativeStudios/connected-systems-go/commit/244f4dd586da685d4d9b75e43f73001028b5bd0e); OpenSensorHub `v2.0.2` at [`235c0eabf24b6d6137b499b4402943d2794b70e6`](https://github.com/opensensorhub/osh-core/commit/235c0eabf24b6d6137b499b4402943d2794b70e6); OS4CSAPI phase-9 at `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`; SECD at `f018fd129bf0d0d1ce75e68198e3ab4d99d937a0`<br>
**Primary Sources:** OGC API - Connected Systems Parts 1 and 2 Version 1.0; SensorML 3.0; SWE Common 3.0; SOSA/SSN; RFC 9110; RFC 9457<br>
**Supporting Resources:** Accepted IDR-SRV-014A/B/E/F/G, IDR-SRV-016 through 035, controlled-AEP findings, and the upstream-history register Version 1.12<br>
**Document Purpose:** Establish a standards-aligned, safety-ready server planning baseline for ControlStreams, Commands, status/results, orchestration, dispatch, and client-visible lifecycle without implementing the server or preempting feasibility and command-safety topics<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 15, 2026<br>
**Date:** September 15, 2026<br>
**Last Updated:** September 15, 2026

---

## Reading Guide

| Label | Meaning |
|---|---|
| N | Requirement or meaning from an approved normative source |
| A | Accepted Glaux project decision |
| I | Version-pinned implementation or interoperability evidence |
| X | Analyst inference, reconciliation, or identified ambiguity |
| P | Recommendation or planning decision proposed by this report |

Precedence is **approved normative source, accepted Glaux baseline, version-pinned implementation evidence, then inference and recommendation**. Internal lifecycle terms written in `monospace` are Glaux concepts unless they are one of the nine uppercase CSAPI status codes.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Command Lifecycle Extraction Methodology
5. Control Stream and Command Concept Taxonomy
6. ControlStream Semantic Model Findings
7. Command Resource Semantic Model Findings
8. Command Lifecycle State-Transition Findings
9. Submission, Validation, Acceptance, Dispatch, Execution, Cancellation, Timeout, Failure, and Unknown Outcome
10. Command Payload, SWE Common, SensorML, Units, Constraints, and Encoding
11. Command Status, Result, Query, Persistence, Event Publication, and Replay
12. Feasibility Relationship Findings
13. DDIL, Gateway, Brokered Dispatch, and Asynchronous Operation
14. Security, Authorization, Safety, Policy, Releasability, Provenance, and Audit
15. Fixture, Conformance, Performance, and Interoperability Implications
16. Downstream Topic Handoff Matrix
17. Recommendations
18. Risks, Constraints, and Open Questions
19. Validation Against Plan Success Criteria
20. References and Sources
Appendix A. Full Lifecycle Matrix
Appendix B. Reproducible Evidence Record

---

## 1. Executive Summary

Glaux should implement Commands as **durable effect requests with append-oriented execution evidence**, not as Observation-like messages and not as remote procedure calls whose HTTP response proves a physical effect. A successful asynchronous submission means that the server durably created one Command, its initial `PENDING` evidence, idempotency/provenance/audit material, and dispatch outbox work in one transaction. It does not mean the receiving System accepted, scheduled, acknowledged, executed, or completed the Command. Dispatch occurs only after commit. **[A/P]**

The core design is a **two-layer lifecycle**. The public standards layer exposes only the approved CSAPI `CommandStatus.statusCode` vocabulary: `PENDING`, `ACCEPTED`, `REJECTED`, `SCHEDULED`, `UPDATED`, `CANCELED`, `EXECUTING`, `COMPLETED`, and `FAILED`.[^1] The private orchestration layer represents validation, feasibility, authorization, safety, queueing, dispatch attempts, transport acknowledgement, target acknowledgement, cancellation requests, expiry, timeout, reconciliation, and uncertain outcome. Glaux must never serialize private values such as `DISPATCHED`, `ACKNOWLEDGED`, `TIMED_OUT`, `EXPIRED`, or `UNKNOWN` into the standard `statusCode` field. **[N/P]**

`Command.currentStatus` is a derived projection of valid, authorized status evidence. Status history is authoritative; arrival order is not. Glaux validates the reporting principal, command revision, legal transition, event/report time, source sequence where available, terminal-state rule, execution-time constraint, and result contract before a report can affect current state. Delayed evidence may change the projection when it is causally valid. Invalid, duplicate, stale, or conflicting reports are retained in restricted evidence/quarantine as policy permits, generate diagnostics and audit, and do not silently overwrite current state. **[A/P]**

A ControlStream is the versioned command contract for one receiving System and a homogeneous set of controlled properties. It binds the target, command format, parameter schema, optional result and feasibility-result schemas, related Procedure/Deployment/features, and derived issue/execution extents. `live` describes whether live command receipt is supported; `async` describes processing mode. Neither field establishes reachability, feasibility, authority, safety, freshness, or successful dispatch. A schema version that has accepted Commands is immutable; an incompatible change creates a new ControlStream/version rather than reinterpreting history. **[N/A/P]**

Command parameters are validated against the exact schema and media type selected from the parent ControlStream. JSON Commands use the parent `parametersSchema`; SWE encodings use the parent record schema and encoding rules. SensorML inputs/parameters and SWE Common components provide semantic definitions, units, constraints, records, arrays, and choices, but a ControlStream remains the authoritative wire contract. Glaux may normalize only registered, semantics-preserving forms and may never silently convert command units, infer missing controlled properties, or repair an invalid command. **[N/A]**

The standard separates Command, CommandStatus, and CommandResult resources. Results may be inline or references to Observation, observation-set, DataStream/time range, or external content. Results and statuses remain independently identified, authorized, retained, queried, and policy-filtered. A referenced Observation remains an Observation; completion does not prove the commanded real-world state, which may need corroborating Observations. **[N/A]**

Cancellation is not HTTP DELETE. Approved Part 2 represents cancellation by creating a `CANCELED` status report, while deletion removes the resource.[^2] Glaux must expose cancellation only through an authorized mediator. It may commit `CANCELED` only when its configured execution contract makes that terminal decision authoritative; otherwise it records a private `cancel_requested` action and awaits authoritative resolution. If the server cannot guarantee this distinction for a dispatch profile, cancellation must be disabled or rejected rather than falsely reported. **[N/X/P]**

Expiration and timeout are Glaux control concepts, not CSAPI status codes. Expiration before execution maps to a reasoned `REJECTED` only when the receiving system definitively will not execute; a known execution timeout maps to `FAILED` only when the executor authoritatively ends the attempt. Loss of connectivity after dispatch produces a private `reconciliation_required/unknown_outcome` condition while the public Command retains its last valid nonterminal status plus explicit freshness/uncertainty metadata outside `statusCode`. A late authoritative terminal report may resolve it. **[P]**

Direct, gateway-mediated, brokered, simulated, and manual dispatch all use one orchestration port and one durable `DispatchAttempt` evidence model. Broker delivery acknowledgement proves only a transport handoff. It becomes `ACCEPTED` only if an authenticated receiving-system protocol explicitly defines that application acknowledgement with the CSAPI meaning. Retries preserve the Command/revision and dispatch identity; they must not create a new canonical Command or double effect. **[A/P]**

Feasibility remains a separate, durable Command-shaped analysis request using the same parameter schema. It is advisory, time- and context-bound, and neither reserves capacity nor authorizes or dispatches a Command. IDR-SRV-037 owns exact feasibility lifecycle and cache/staleness behavior. IDR-SRV-038 owns final authorization, safety, cancellation authority, and audit controls. Therefore this report does **not** enable inbound command families in `glaux-csapi-part3-exp/0.1`, authorize server implementation, or advertise tasking conformance. **[A/P]**

## 2. Scope and Plan Alignment

### 2.1 Included scope

- Standards meaning and resource relationships for ControlStream, Command, CommandStatus, CommandResult, and Feasibility.
- Command-capable System, controlled-property, Procedure, Deployment, Sampling Feature/feature-of-interest, SensorML, and SWE Common relationships.
- Public and internal states, allowed transitions, terminality, updates, cancellation, expiry, timeout, failure, and uncertain outcome.
- Submission admission, durable creation, dispatch, direct/gateway/broker/simulator/manual adapters, status/result ingestion, persistence, queries, events, replay, and DDIL behavior.
- Required hook points for identity, feasibility, authorization, safety, policy, releasability, provenance, audit, observability, fixtures, conformance, performance, and interoperability.

### 2.2 Excluded scope

- Final feasibility algorithms, result profile, caching horizon, or reservation behavior; IDR-SRV-037 owns these.
- Final authentication, authorization, safety/interlock, command-priority, approval-chain, releasability, or audit schema; IDR-SRV-038 through 041 own these.
- Final DDIL queue policy, synchronization protocol, conflict resolution, service architecture, database DDL, Rust framework/library selection, numeric limits/SLOs, or deployment topology.
- Server code, tasking-channel enablement, draft Part 3 inbound publication, or any conformance claim.

### 2.3 Core-question coverage

| Plan question | Result | Principal coverage |
|---|---|---|
| Q1. Standards meanings | Complete | §§3, 5–7, 10–12 |
| Q2. Lifecycle states and transitions | Complete | §§8–9; Appendix A |
| Q3. Feasibility, trust, policy, transaction, dispatch, event, audit, DDIL interaction | Complete | §§9, 12–14 |
| Q4. Persistence, query, latest/history, events, and client behavior | Complete | §11 |
| Q5. Downstream implications | Complete | §§15–18 |

### 2.4 Accepted-baseline reconciliation

This report applies rather than reopens the accepted findings that PostgreSQL is authoritative; replay-prone unsafe operations use scoped idempotency; one write pipeline performs identity, contract, validation, policy, transaction, provenance, persistence, outbox, audit, and response work; publication is post-commit and at least once; consumers are idempotent; source identities and epochs are explicit; simulator traffic cannot bypass safety; status Observation, SystemEvent, source health, CommandStatus, audit, and delivery evidence remain distinct; and latest/current projections are policy-first and not arrival-wins. **[A]**

## 3. Evidence Base and Authority Classification

### 3.1 Controlling and supporting sources

| Source | Exact state | Class | Use | Limitation |
|---|---|---|---|---|
| CSAPI Part 1 | OGC 23-001, Version 1.0 | N | System, Procedure, Deployment, Sampling Feature, links, security inheritance | Does not define command lifecycle |
| CSAPI Part 2 | OGC 23-002, Version 1.0; tag `v1.0.0`/`8e03b236` | N | ControlStream, Command, statuses, results, feasibility, query and mutation obligations | Does not define internal dispatch/safety state machine |
| Tagged OpenAPI and schemas | same immutable tag | Normatively referenced/supporting artifact | plural routes, request/response shapes, exact status/result schemas | Feasibility paths absent from tagged root OAS; prose contains route/copy defects |
| SensorML | OGC 23-000, Version 3.0 | N | process inputs, outputs, parameters, modes, configuration, semantics | Description model, not execution authority |
| SWE Common | OGC 24-014, Version 3.0 | N | components, units, constraints, records/arrays/choices, encodings | Does not authorize or dispatch Commands |
| SOSA/SSN | W3C/OGC 2017 Recommendation | N dependency | Actuator, Actuation, actuatable property, Procedure, feature, result semantics | CSAPI Command is an API message/resource, not identical to every ontology assertion |
| RFC 9110 / RFC 9457 | current RFCs | External normative | HTTP method/status semantics and problem details | Do not define domain acceptance or task safety |
| Controlled AEP baseline | `AC/224(JCGISR)D(2026)0005`, 27 Apr 2026, SHA-256 `56DC...DA8C` | A/controlled | accepted server/tasking context and secure-handling constraints | Not redistributed; no text is reproduced here |
| Prior Glaux IDRs | accepted through IDR-SRV-035 | A | resource, schema, unit, persistence, transaction, source, ingestion, status, event baseline | Later owners retain detailed security/DDIL decisions |
| CS-Go / OSH | pins in front matter | I | concrete separated resource/store/adapter patterns | Neither is a conformance oracle or safety architecture |
| OS4CSAPI/SECD studies | fixed pins and captured evidence | I | client lifecycle, route, schema, response, and traversal risks | Historical/deployment-specific; no live command was executed |

Approved Part 2 was published July 16, 2025 after June 2 approval and identifies `23-002`, Version 1.0.[^3] The exact tag is used because current repository `master` can change after publication. The upstream-history register was date-checked at Version 1.12; no open-history item changes the approved nine-code lifecycle or resolves the identified published artifact defects. **[N/A]**

### 3.2 Normative baseline

Part 2 defines a ControlStream as Commands targeted at the same System and sharing controlled properties; a Command is a message sent to a System to trigger an Actuation with parameters; and CommandStatus reports execution status.[^3] A ControlStream is a SOSA ActuationCollection specialization and a Command generalizes Actuation. SOSA describes an Actuation as carrying out a Procedure to change the world through an Actuator, acting on at least one actuatable property of a feature of interest.[^4] **[N]**

For asynchronous processing, multiple status reports may cover acceptance/rejection, schedule, execution, progress, failure, and cancellation. For synchronous processing the standard describes one returned status of `COMPLETED`, `REJECTED`, or `FAILED`. A completed report requires actual execution time; a scheduled report requires scheduled execution time; terminal `REJECTED`, `CANCELED`, `COMPLETED`, and `FAILED` admit no later status updates. **[N]**

The Create/Replace/Delete class requires Command creation under `/controlstreams/{csId}/commands`, and status/result creation under `/commands/{cmdId}/status` and `/commands/{cmdId}/result`. The Update class adds partial update when claimed. These mutation classes are optional claims, but if advertised their applicable requirements and ATS must be satisfied. **[N]**

### 3.3 Artifact contradictions and limitations

The approved prose includes singular paths such as `/controlstream/{csId}/commands` and `/command/{cmdId}/status`, and several ControlStream clauses retain DataStream/Observation/`dsId` wording. The tagged root OpenAPI and CRUD clauses consistently use plural `/controlstreams` and `/commands`; Glaux should treat the singular/copy text as errata, record the profile choice, use plural canonical routes, and test both links and OpenAPI against runtime. The tagged root OpenAPI omits feasibility even though the normative feasibility class defines it. **[N/X/P]**

The approved standard permits CREATE/REPLACE/DELETE/UPDATE of status and result resources when those optional mutation classes are claimed. Operationally, Glaux still needs append-oriented evidentiary history. A correction therefore creates a protected revision/tombstone and recomputes projections; it does not erase audit or make history arrival-wins. Whether Glaux initially claims mutation classes for status/result must be decided by conformance planning, not inferred from storage mutability. **[N/A/P]**

No source defines a complete safety policy, executor acknowledgement protocol, dispatch retry identity, unknown-outcome representation, or DDIL reconciliation algorithm. These are explicit Glaux planning choices and downstream handoffs, not attributed standard requirements. **[X/P]**

## 4. Command Lifecycle Extraction Methodology

1. Pin approved standards, schemas, OpenAPI, implementation releases, and accepted reports; classify normative, accepted, implementation, inferred, and proposed statements.
2. Extract every resource property, association, route, status-code meaning, time constraint, schema obligation, filter, mutation, and terminality rule.
3. Separate actors: requesting principal, API client, Glaux admission service, policy/safety evaluator, dispatcher, adapter/gateway/broker, receiving System/executor, status/result reporter, operator, and subscriber.
4. Separate records: request bytes, canonical Command, Command revision, status fact, current projection, result, feasibility analysis, dispatch attempt, transport acknowledgement, event, audit, and operational diagnostic.
5. Model the exact public status vocabulary independently from richer orchestration state; map only where the public meaning is satisfied.
6. Trace synchronous, asynchronous, direct, gateway, broker, simulated, manual, disconnected, duplicate, delayed, cancellation-race, and uncertain-outcome paths.
7. Apply the accepted transaction/idempotency/outbox and source-authority model to each transition and failure window.
8. Compare implementation evidence for feasibility and hazards; never elevate implementation behavior to normative meaning.
9. Derive fixtures, conformance assertions, performance measures, and downstream ownership for every decision.

The acceptance test for a proposed state is: can a client tell what Glaux knows, what the target has asserted, what remains uncertain, and what action is still permitted without confusing transport success with domain success? **[P]**

## 5. Control Stream and Command Concept Taxonomy

| Concept | Definition and authority | Identity/lifecycle | Must not be confused with |
|---|---|---|---|
| Command-capable System | Part 1 System that can receive Commands through at least one advertised ControlStream | stable System ID plus valid description revisions | current reachability or blanket authority |
| Actuator/Procedure | device/process and method that cause a world-state change | referenced semantic/description resources | dispatch adapter or API endpoint |
| Controlled/actuatable property | property expected to be affected | stable definition URI within stream contract | proof of actual post-command state |
| ControlStream | homogeneous, versioned input contract for one System and controlled-property set | canonical resource plus immutable schema versions and derived extents | DataStream, queue, broker topic, or authorization grant |
| Command definition | reusable semantic operation shape embodied by ControlStream/SensorML/SWE metadata | versioned contract | submitted Command instance |
| Command request | client bytes, headers, identity, requested timing, and correlation presented for admission | ephemeral/staged until decision | canonical resource or execution |
| Command resource | durable canonical effect request belonging to one ControlStream | stable ID, immutable receipt identity, controlled revisions | dispatch attempt or Actuation proof |
| Command revision | accepted update to writable intent before the applicable cutoff | monotonic internal revision; may yield `UPDATED` report | a new independent Command unless semantics require one |
| Command execution | target-side processing/effect attempt | one or more private execution/attempt identities | HTTP transaction or transport delivery |
| CommandStatus | time-stamped public status evidence for one Command | independent ID; append-oriented; exact nine-code vocabulary | status Observation, health, audit, or dispatch ACK |
| Current status | deterministic projection of valid status evidence | rebuildable and policy-aware | mutable source of truth or last arrival |
| CommandResult | inline or linked result associated with Command/status | independent ID and policy/retention | proof every controlled property reached desired state |
| Feasibility | separate Command-shaped analysis request on feasibility channel | independent ID/status/result | authorization, safety approval, reservation, or Command |
| Dispatch attempt | private durable handoff attempt through one adapter | attempt ID, command revision, destination, retry lineage | Command or `ACCEPTED` status |
| Transport/application acknowledgement | evidence that broker/gateway/device accepted a message/protocol operation | delivery record with source/time | execution start or completion |
| Cancellation request | private authorized request to stop/not execute | action ID and decision lifecycle | final CSAPI `CANCELED` until authoritative |
| Expiry/timeout | private validity/deadline condition | reason and timer evidence | CSAPI status code |
| Unknown outcome | private reconciliation condition after ambiguous dispatch/execution | since time, last evidence, attempts, owner | invented `UNKNOWN` public status |
| Command event | durable publication projection of accepted lifecycle change | event ID, command/status IDs, revision/cursor | status source of truth |
| Audit record | protected evidence of actor, decision, transition, and effect | immutable/tamper-evident policy retention | public CommandStatus or event |

## 6. ControlStream Semantic Model Findings

### 6.1 Standards-facing model

A ControlStream identifies one receiving System, common controlled properties, and a command schema. Its standard representation includes descriptive fields and links, `controlledProperties`, derived `issueTime` and `executionTime` extents, `live`, `async`, and per-format schema access. Associations may identify Procedure, Deployment, Sampling Feature/features of interest, System, and Commands. The schema endpoint is selected with `cmdFormat`; the returned representation is not necessarily JSON. **[N]**

`controlledProperties` describes what Commands in the stream may affect. `issueTime` and `executionTime` summarize contained Commands; they are not operator-supplied scheduling policies. `live` says the stream can accept live Commands and `async` selects synchronous/asynchronous behavior; neither establishes that the target is connected now. Clients must not infer authorization from discovery or feasibility from visibility. **[N/X]**

### 6.2 Glaux internal binding

Each active ControlStream version should bind:

- canonical System ID and optional Procedure, Deployment, and sampling/ultimate-feature references;
- exact controlled-property URIs and display metadata;
- supported command media types and immutable parameter schema digests;
- optional result and feasibility-result schemas with independent digests;
- dispatch profile ID and target addressing held privately where sensitive;
- accepted adapter types and reporter/source identities;
- operational mode (`sync` or `async`), live capability, and enable/disable lifecycle;
- policy/safety profile references, classification/releasability labels, and tenant/mission scope;
- schema/profile version, validity interval, provenance, and replacement relationship; and
- derived issue/execution extents and command counts as rebuildable projections.

The public response contains only fields authorized by the standard/profile and caller policy. Dispatch addresses, credentials, interlock details, hidden modes, and sensitive capability metadata stay private. **[P]**

### 6.3 Lifecycle and evolution

A ControlStream may be `draft`, `active`, `suspended`, `retired`, or `archived` internally. Only `active` accepts ordinary Commands; `suspended` may retain read/history while rejecting or deferring new Commands according to an advertised profile; `retired/archived` are read-only. These are Glaux management states, not new Part 2 fields. **[P]**

Once a Command is accepted against a schema version, incompatible mutation cannot reinterpret its bytes. A compatible descriptive correction may create a new resource revision; a parameter/result schema or controlled-property change creates a new ControlStream or explicit version with predecessor link. Existing Commands retain the exact contract pin. Parent deletion with Commands follows the advertised CRUD/cascade rules, but production policy should normally archive rather than erase tasking evidence. **[N/A/P]**

### 6.4 ControlStream versus DataStream

| Dimension | ControlStream | DataStream |
|---|---|---|
| Direction | data/Commands into a System | Observations/data out of a System |
| Member | Command | Observation |
| Semantic role | desired effect/actuation request | evidence about observed property |
| Contract | command parameters/results | observation result/parameters |
| State | CommandStatus/Result lifecycle | fact revisions and latest/current projection |
| Safety | effectful and authorization/safety gated | disclosure/integrity policy; no actuation authority |
| Success evidence | target status plus optional corroborating Observation | Observation is evidence, not command acceptance |

## 7. Command Resource Semantic Model Findings

### 7.1 Standards representation

A server-reported Command has a server-assigned ID and ControlStream reference, server receipt `issueTime`, optional Sampling Feature and Procedure references, derived `executionTime`, `sender`, `currentStatus`, and required `parameters`. Client-provided `issueTime`, `executionTime`, and `currentStatus` are ignored on creation/update; parameters must validate against the parent ControlStream schema.[^5] **[N]**

The canonical API resource is not merely the request bytes. Glaux assigns identity and receipt time, derives authenticated sender rather than trusting it, pins the ControlStream/schema version, and records the accepted canonical representation. If the media profile permits client IDs, their scope and collision rules remain the IDR-SRV-016/044 contract; an idempotency key is not itself the Command ID. **[A/P]**

### 7.2 Field ownership

| Field/evidence | Client supplied | Server assigned/derived | Gateway/executor supplied | Public standard field? |
|---|---:|---:|---:|---:|
| parameters and optional FOI/procedure selection | yes, within schema/policy | canonicalized/validated | no | yes |
| requested execution time | only where command schema/profile allows | scheduling interpretation | proposed/actual time in status | Command field is read-only; status field yes |
| `id`, `issueTime`, `sender`, `currentStatus` | ignored if presented as read-only values | yes | reporter may influence status only | yes |
| validity/expiration/deadline | profile extension/request metadata | authoritative accepted value | executor receives bounded value | no unless profile-defined extension |
| feasibility reference | optional profile metadata | validated link/fingerprint | evaluator creates feasibility evidence | no direct base field |
| policy/safety decisions and labels | no trusted self-assertion | yes | inputs/attestations only | private or separately profiled |
| dispatch destination/attempt/retry | no | yes | acknowledgement/status evidence | no |
| `executionTime`, progress, status, result | no on Command | projected from valid reports | authorized reporter | yes through status/result |
| correlation/causation/idempotency | request metadata | scoped, persisted | propagated where safe | Glaux contract, not base Command |

### 7.3 Identity, updates, and supersession

A duplicate retry with the same scoped idempotency key, canonical intent digest, principal, and ControlStream version returns the original outcome and never dispatches twice. Reuse with different intent is a conflict. A Command update creates an immutable revision only while the dispatch profile permits modification; the server emits `UPDATED` after the receiving system accepts the update. Because `UPDATED` is an event-like code rather than a stable phase, Glaux retains `effective_phase` internally and should emit a following stable status when necessary for portable clients. **[N/A/P]**

An update that materially changes target, controlled-property set, safety class, or cannot be atomically applied should be rejected; the client cancels the original where possible and creates a new linked Command. `superseded` is an internal revision relationship, not a public status code. Every dispatch attempt identifies the exact Command revision. **[P]**

## 8. Command Lifecycle State-Transition Findings

### 8.1 Public CSAPI lifecycle

The safe public transition envelope is:

```text
PENDING -> ACCEPTED -> SCHEDULED -> EXECUTING -> COMPLETED
   |          |            |            |
   +----------+------------+------------+--> REJECTED / CANCELED / FAILED
              +---------- UPDATED ----------> prior/effective nonterminal phase
```

This diagram is deliberately an envelope rather than a claim that every path is legal. `REJECTED` means the Command will not execute at all, so it is invalid after known execution began. `FAILED` means execution failed, so it is inappropriate for pre-admission schema or authorization denial. `CANCELED` is authorized-user cancellation and may occur before or during execution if the execution contract makes cancellation authoritative. Repeated `EXECUTING` reports may refine progress. Terminal statuses admit no later status reports. **[N/X]**

Allowed external transitions for the Glaux profile are:

| From | Allowed next status | Rule |
|---|---|---|
| none | `PENDING` or synchronous terminal | Async durable receipt creates `PENDING`; a registered synchronous operation may return one terminal report |
| `PENDING` | `ACCEPTED`, `REJECTED`, `CANCELED` | `FAILED` is reserved for processing/execution failure after acceptance; cancellation requires authority |
| `ACCEPTED` | `SCHEDULED`, `EXECUTING`, `UPDATED`, `REJECTED`, `CANCELED`, `FAILED`, `COMPLETED` | direct completion is valid for short/sync execution; rejection means it never executed |
| `SCHEDULED` | `UPDATED`, `EXECUTING`, `REJECTED`, `CANCELED`, `FAILED`, `COMPLETED` | scheduled execution time required; direct completion allowed when start report omitted |
| `UPDATED` | prior/effective valid nonterminal phase or a valid outcome | internal phase controls; no unlimited self-loop |
| `EXECUTING` | `EXECUTING`, `UPDATED`, `CANCELED`, `FAILED`, `COMPLETED` | no `REJECTED` after known execution; progress/time rules apply |
| terminal | none | correction is protected resource revision/tombstone, not a later lifecycle report |

### 8.2 Private orchestration state

The internal state machine should expose separate dimensions rather than one oversized enum:

- `admission_state`: `received`, `validating`, `rejected`, `committed`;
- `gate_state`: feasibility/authorization/safety `not_required`, `pending`, `passed`, `failed`, `stale`;
- `schedule_state`: `unscheduled`, `queued`, `scheduled`, `expired`;
- `dispatch_state`: `not_ready`, `ready`, `attempting`, `delivered`, `acknowledged`, `retry_wait`, `exhausted`;
- `execution_state`: `not_started`, `accepted`, `executing`, `completed`, `failed`, `canceled`;
- `cancel_state`: `none`, `requested`, `authorized`, `sent`, `confirmed`, `denied`, `too_late`;
- `certainty_state`: `current`, `stale`, `reconciliation_required`, `conflicting`;
- `retention_state`: `active`, `archived`, `legal_hold`, `tombstoned`.

The public mapper uses only authoritative facts and the exact CSAPI meaning. For example, `dispatch_state=delivered` with no target application decision remains public `PENDING` or the last valid status; `certainty_state=reconciliation_required` never becomes `UNKNOWN`. **[P]**

### 8.3 Transition acceptance algorithm

For each candidate status/result, Glaux atomically:

1. authenticates the reporting principal and resolves represented executor/source;
2. authorizes the exact Command, revision, operation, status/result kind, and policy scope;
3. validates media type, static schema, parent result schema, time, percentages, references, and size;
4. deduplicates reporter message/status identity and compares source epoch/sequence;
5. locks or compare-and-swaps the Command projection revision;
6. checks allowed transition, terminality, causal predecessor, cancellation/update authority, and execution-time requirements;
7. appends raw/canonical evidence and source/provenance data;
8. recomputes the authoritative public and internal projections using report time plus causal ordering, not arrival alone;
9. commits audit and outbox events in the same transaction; and
10. returns the committed resource or an exact conflict/problem response.

## 9. Submission, Validation, Acceptance, Dispatch, Execution, Cancellation, Timeout, Failure, and Unknown Outcome

### 9.1 Submission and admission boundary

Before creating a Command, Glaux performs low-cost authentication, route/method/media/profile checks, body and batch limits, idempotency lookup, ControlStream existence/visibility/activity, structural schema validation, parameter/SWE validation, reference integrity, requested-time/validity checks, and authorization to request the capability. Failures here are HTTP problem responses and create no ordinary Command or lifecycle event. Non-oracular policy may conceal existence with `404`; malformed contract is generally `400`/`415`; conflict/precondition/idempotency mismatch is `409`/`412`; rate/capacity controls use the registered HTTP contract. **[A/P]**

Checks that may be asynchronous—detailed feasibility, approval chain, safety interlock, target admission, schedule negotiation—occur only after durable creation when the profile permits pending work. One transaction stores the Command, initial `PENDING` status, request/contract/provenance digest, gate work, audit, and outbox. A `201 Created` identifies the canonical Command and `Location`; a separate `202` workflow is allowed only under the accepted write contract with a durable monitor URI. For a registered synchronous ControlStream, the response contains the terminal status representation but transport success still does not alter its domain meaning. **[N/A/P]**

### 9.2 Receipt, acceptance, and acknowledgement

| Term | Required meaning |
|---|---|
| HTTP request received | bytes reached an API listener; no durable claim |
| Command created | canonical resource and initial evidence committed |
| `PENDING` | receiving system has the Command but no accept/reject decision |
| `ACCEPTED` | receiving system passed initial validation and accepted processing; later reject/failure remains possible |
| Dispatch delivered | adapter/broker/gateway accepted delivery under its protocol |
| Target acknowledged | target protocol acknowledged receipt; maps to `ACCEPTED` only if its registered semantics satisfy that code |
| `SCHEDULED` | receiving system validated and effectively scheduled it; execution time supplied |
| `EXECUTING` | receiving system reports execution in progress |
| `COMPLETED` | receiving system reports successful execution with actual execution time |

### 9.3 Dispatch model

All dispatch types implement a common port such as `dispatch(command_id, revision, attempt_id, deadline, target_binding)` and return transport/application evidence, never a direct mutation of `currentStatus`.

| Pattern | Dispatch behavior | Acceptance authority | Key risk/control |
|---|---|---|---|
| Direct | adapter calls target API after commit | authenticated target response/status protocol | ambiguous network timeout; query by stable ID before retry |
| Gateway-mediated | registered gateway translates and forwards | represented executor only within explicit grant | gateway cannot self-expand targets/capabilities |
| Brokered | outbox publishes stable command/revision identity | application-level target report, not broker ACK | duplicate delivery; inbox/dedupe and expiry required |
| Simulated | simulator adapter runs deterministic scenario | simulator execution service in isolated test scope | synthetic marking; no production safety bypass |
| Manual/operator | work item presented to authorized operator | recorded operator/execution service decision | human latency, dual control, and nonrepudiation |

Dispatch failure before confirmed delivery remains retryable private state while validity and policy allow. Exhaustion before target acceptance normally yields a reasoned `REJECTED` because the receiving system will not execute. Failure after known acceptance/execution yields `FAILED` only when the authoritative executor declares failure. An ambiguous send never authorizes blind replay under a new identity. **[P]**

### 9.4 Cancellation and update

HTTP DELETE deletes a Command resource and is never cancellation.[^2] The Part 2 cancellation representation is a created `CANCELED` status. Glaux mediates it as follows:

1. authorize the actor for this target, Command revision, current state, and cancellation class;
2. serialize with update/terminal transitions using a projection precondition;
3. if the configured executor contract makes server cancellation immediately authoritative, append `CANCELED`, audit, and dispatch abort work atomically;
4. otherwise append private `cancel_requested` evidence and dispatch the request without exposing `CANCELED` yet;
5. append public `CANCELED` only on authoritative confirmation; record `denied`/`too_late` privately and retain/advance the valid public state; and
6. resolve completion/cancellation races from causal executor evidence, never arrival timing.

After public `CANCELED`, later status is prohibited. Any later evidence of physical effect becomes a protected anomaly/incident and, where appropriate, a separate Observation/SystemEvent—not a rewrite of cancellation semantics. **[N/P]**

An update is accepted only before the adapter-specific cutoff and must preserve Command identity semantics. It creates a revision, re-runs affected validation/gates, and dispatches with the same canonical Command ID plus new revision. The `UPDATED` report means the receiving system accepted the update; it does not itself say whether the Command is queued, scheduled, or executing. **[N/P]**

### 9.5 Expiration, timeout, failure, and unknown outcome

| Condition | Internal representation | Public mapping |
|---|---|---|
| validity expires before any execution and target definitively will not execute | `expired`, reason `GLAUX.EXPIRED_BEFORE_EXECUTION` | `REJECTED` with safe message |
| server queue/dispatch deadline exceeds before target acceptance | `dispatch_exhausted` | `REJECTED` if nonexecution is certain; otherwise retain last status + unknown condition |
| known execution exceeds deadline and executor aborts/fails | `execution_timeout` | `FAILED` with safe reason and execution time if known |
| connection lost after possible dispatch | `reconciliation_required` | last valid standard status; separate freshness/uncertainty metadata |
| target rejects after initial acceptance but before execution | target rejection evidence | `REJECTED` |
| target/system stops accepted processing because of internal problem | execution/processing failure | `FAILED` when semantics constitute execution failure |
| safety/policy revocation before execution | gate revocation | `REJECTED`; cancellation only when initiated by authorized user |
| safety stop during known execution | execution failure/emergency stop evidence | normally `FAILED`; exact safety reason owned by IDR-SRV-038 |

Unknown outcome must be visible to authorized operators and clients through a Glaux extension/operation-status view containing `certainty`, `lastAuthoritativeReportTime`, `lastContactTime`, `reconciliationSince`, and safe next action. It must not falsely convert uncertainty to failure or completion. The standard Command remains queryable and preserves its last valid status. **[P]**

## 10. Command Payload, SWE Common, SensorML, Units, Constraints, and Encoding

### 10.1 Contract composition

SensorML AbstractProcess supports inputs, outputs, and parameters expressed as observable properties, SWE Common data components, or streams.[^6] SWE Common supplies scalar components (`Quantity`, `Count`, `Category`, `Boolean`, `Text`, `Time`), ranges and aggregates (`DataRecord`, `DataArray`, `DataChoice`), semantic `definition` URIs, units, nil values, and allowed-value/interval constraints.[^7] These make command meaning machine-readable, but the selected ControlStream schema is the actual command wire contract. **[N]**

Glaux should resolve the chain:

```text
System -> ControlStream version -> command media type -> parameters schema
       -> controlled-property definitions -> optional Procedure/SensorML description
       -> result schema / feasibility-result schema
```

Every stored Command pins all mutable links by resource revision/digest sufficient to reproduce validation. A later description update cannot change the meaning of a historical Command. **[A/P]**

### 10.2 Validation rules

- Require the advertised media type and exact `cmdFormat`; do not sniff or silently fall back.
- Validate the static Command envelope and then the parent ControlStream parameter schema.
- Validate every component recursively, including `DataChoice` member types, `DataRecord` field names, arrays/counts, ranges, categories/codespaces, nil reasons, and required/optional cardinality.
- Require explicit units where the schema does. Convert only at a separately registered adapter boundary with exact source/canonical values, algorithm/version, rounding, and policy evidence; the public Command remains in the accepted contract unit.
- Validate semantic definition URIs and controlled-property bindings against the pinned offline vocabulary/profile; do not dereference arbitrary client URLs in the request path.
- Validate requested time/reference frame and FOI/procedure relationships without inventing missing meaning.
- Preserve unknown extensions only where the negotiated profile allows them; otherwise reject.
- Validate inline results against `resultSchema` and feasibility inline results against `feasibilityResultSchema`.

SECD evidence demonstrates why envelope success is insufficient: a ControlStream used `inputSchema` and nested `DataChoice` members without the component `type` expected by the pinned parser. Glaux fixtures must independently check canonical schema location, recursive validity, parsing, semantic preservation, and round trip. **[I/P]**

### 10.3 Encoding boundary

Part 2 defines ordinary JSON and SWE Common JSON/text/binary Command encodings. Ordinary JSON places parameters in the Command representation according to the parent schema. SWE encodings serialize the Command according to the ControlStream record schema/encoding rules and may map issue time or Sampling Feature using defined semantic identifiers. Glaux should implement JSON first, retain an encoding-neutral canonical component/value tree, and add SWE text/binary only with golden vectors and lossless round-trip evidence. **[N/P]**

Schema validation proves structural and declared semantic conformity. It does not prove feasibility, authorization, safety, target reachability, current configuration, or that the effect occurred. Those remain distinct gates/evidence. **[A/P]**

## 11. Command Status, Result, Query, Persistence, Event Publication, and Replay

### 11.1 Status and result records

`CommandStatus` requires Command association, report time, and a valid status code; it may include progress, execution time, message, and results. Progress must be finite and 0–100. Scheduled/completed execution-time constraints are enforced. Messages are human-readable diagnostics, not machine reason codes; Glaux stores a private stable reason taxonomy and exposes only policy-safe text. **[N/P]**

`CommandResult` supports exactly one result variant per result item: inline data, Observation reference, observation-set reference, DataStream reference with optional result time, or external link. Glaux validates referenced-resource existence/visibility at acceptance where possible, retains broken/deferred resolution state privately, prevents SSRF when resolving external references, and applies policy to both link and target. Partial results may accompany progress reports; completion does not require inline duplication of separately stored Observations. **[N/P]**

### 11.2 Persistence and projection

Recommended logical stores are:

| Store | Mutability and purpose |
|---|---|
| `control_stream_revision` / `command_schema` | immutable versions/digests and public representations |
| `command` / `command_revision` | stable identity plus append-only accepted intent revisions |
| `command_status_evidence` | append-oriented canonical reports, raw digest, reporter/sequence/causality |
| `command_result` / revision | independent typed results and references |
| `command_projection` | rebuildable current public/internal state and certainty |
| `gate_decision` | feasibility/auth/safety decision references and versions |
| `dispatch_attempt` / `delivery_evidence` | adapter, destination, revision, attempt, ACK, retry, deadline |
| `idempotency_record` | scoped request digest and committed response identity |
| `outbox` / event log | atomic post-commit work and replay publication |
| `audit_record` | protected actor/decision/transition evidence |

Status resource replacement/update/deletion, if claimed, creates revision or tombstone evidence and recomputes `command_projection`. Ordinary reporters cannot erase terminal or adverse evidence. Retention, legal hold, and redaction apply by record class and never silently leave `currentStatus` inconsistent with visible authorized history. **[A/P]**

### 11.3 Ordering, latest, and conflict

The current selector is based on accepted transition causality, authoritative reporter scope, report time, source epoch/sequence, and stable ID tie-break—not database insertion order. A later-arriving earlier report may enrich history without regressing state. A causally valid terminal report resolves a private unknown condition. Same-time or cross-source conflicts are quarantined or marked `conflicting`, preserve last uncontested public status, and require configured reconciliation/operator action. **[A/P]**

CS-Go `v1.0.4` demonstrates useful separated Command/status/result persistence, canonical and nested routes, initial `PENDING`, schema checks, pagination, and MQTT status ingestion. Its repository updates `current_status` to each newly created/updated status directly, so last write can win without transition/terminal/event-time enforcement. It is useful feasibility evidence and a specific pattern Glaux must strengthen. OSH likewise demonstrates separated typed stores/handlers but does not complete feasibility. **[I/X/P]**

### 11.4 Query and client behavior

The approved advanced-filtering class includes Command `issueTime`, `executionTime`, `statusCode`, `sender`, and `foi`, plus CommandStatus `statusCode`; base resource filters/pagination also apply where claimed. Glaux should additionally offer only profile-declared filters for System, ControlStream, reporter, report time, certainty, and correlation, with policy applied before counts, sorting, pagination, latest, or aggregation. Cursors are opaque and policy-bound under IDR-SRV-035. **[N/A/P]**

Every Command response links its canonical resource, parent ControlStream, status history, and available results. Clients should:

1. preserve the idempotency key through ambiguous submission;
2. follow `Location`/links rather than construct singular fallback routes;
3. interpret `201`/`202` as the registered resource/operation outcome, not execution;
4. monitor status until terminal or explicit uncertainty/action guidance;
5. tolerate repeated `EXECUTING` and `UPDATED` reports;
6. deduplicate status/events by resource/event identity;
7. refetch authoritative HTTP state after stream gaps or cursor expiry; and
8. treat Command results and corroborating Observations as distinct evidence.

OS4CSAPI smoke evidence observed both `201` and `202` patterns and differing nested/top-level command routes. Glaux must publish one truthful runtime OpenAPI/capability contract and test client tracking, rather than use fallback on arbitrary `400`/`403` responses. **[I/P]**

### 11.5 Events and replay

Accepted Command creation/revision, accepted public status/result, cancellation decision, internal dispatch/reconciliation change, and correction/tombstone each commit an outbox event when eligible. Public tasking publication is policy filtered and disabled until IDR-SRV-038 authorizes it. Transport event order does not redefine status order; replay preserves stable event/status identity and clients refetch on ambiguity. Broker ACK remains delivery evidence only. **[A/P]**

The draft Part 3 experimental profile adopted by IDR-SRV-035 remains outbound-only and its initial command families remain disabled. IDR-SRV-036 supplies lifecycle semantics but does not satisfy authorization/safety or enable those gates. **[A]**

## 12. Feasibility Relationship Findings

Approved Part 2 models Feasibility as a Command created on the parent ControlStream feasibility channel. It uses the same parameter schema, has its own canonical identity, nested status/results, and synchronous/asynchronous behavior. Its status vocabulary reuses CommandStatus but `SCHEDULED` and `UPDATED` are unused; inline output uses the separate feasibility-result schema. **[N]**

Glaux should treat feasibility as:

- **separate:** a feasibility ID is not a Command ID and completion creates no Command;
- **advisory:** `COMPLETED` analysis is not necessarily `feasible=yes`; the structured result carries the decision/details;
- **context-bound:** evaluator/version, target/stream/schema revision, parameter fingerprint, policy context, environmental inputs, issue/completion time, and validity horizon are recorded;
- **non-reserving:** success does not reserve target time/capacity unless a future explicit reservation profile says so;
- **non-authorizing:** it does not replace identity, authorization, releasability, or safety checks;
- **revalidated:** a linked result may be reused only inside its accepted freshness/context rules, with material changes causing re-evaluation; and
- **durable:** status/result, idempotency, audit, and outbox follow the same evidence principles as Commands.

A Command may reference feasibility privately/profile-wise by feasibility ID and input fingerprint. Negative feasibility before Command creation is a valid analysis outcome, not an HTTP schema failure. If policy requires feasibility during asynchronous Command admission, the Command remains `PENDING` while evaluation proceeds, then advances or is reasonedly `REJECTED`. Exact mandatory/optional modes, result vocabulary, staleness, cancellation, caching, reservation boundary, and client contract are handed to IDR-SRV-037. **[N/P]**

## 13. DDIL, Gateway, Brokered Dispatch, and Asynchronous Operation

### 13.1 Queueing and validity

Glaux may accept Commands while a target is disconnected only when the ControlStream/dispatch profile explicitly supports queueing, the actor is authorized for deferred effect, a bounded validity/deadline exists, safety/policy can remain valid, and the client is told the Command is only `PENDING`/accepted for processing. Otherwise the server rejects or returns the registered unavailable outcome before creating a dispatchable Command. **[P]**

Queued Commands retain priority assigned by trusted policy—not client self-assertion—plus target, revision, validity, dependency, gate-decision versions, and retry identity. Before dispatch after reconnect, Glaux rechecks expiration, supersession/cancellation, authorization/safety policy freshness, target binding, and any required feasibility. **[P]**

### 13.2 Reconnect and reconciliation

- Gateways spool status/results with stable message IDs, source epoch/sequence, report time, Command/revision, and schema/profile pin.
- Server inbox uniqueness makes replay idempotent; gaps and epoch changes are explicit.
- Delayed valid reports enter history and may update projection only through the transition algorithm.
- A server sends the same dispatch identity after ambiguous delivery and queries target state when supported before retrying.
- Expired messages are not delivered merely because a broker session survived; broker expiry is a safety backstop, not canonical validity.
- Policy revocation during disconnection blocks new dispatch and is audited; detailed conflict precedence belongs to IDR-SRV-043.
- Snapshot/catch-up and opaque policy-bound cursors from IDR-SRV-035 recover subscriber views; they do not reconcile executor state by themselves.

### 13.3 Unknown and conflicting outcomes

If a gateway disappears after possible effect, Glaux records last authoritative public status, dispatch evidence, and `reconciliation_required`. The system must not retry a potentially non-idempotent physical action under a fresh Command ID merely to obtain certainty. Resolution order is: query target by stable identity; reconcile signed/authorized buffered evidence; apply adapter-specific idempotent retry if proven safe; seek operator decision; or retain the unknown condition. **[P]**

A delayed `COMPLETED` after a private unknown condition is acceptable when it is a valid continuation of a nonterminal public status. A `COMPLETED` after public `CANCELED` conflicts with the terminal rule and becomes incident evidence. Detailed offline operating modes, synchronization, and conflicts remain IDR-SRV-042/043 work. **[N/P]**

## 14. Security, Authorization, Safety, Policy, Releasability, Provenance, and Audit

### 14.1 Required lifecycle hooks

| Hook | Minimum input | Required effect |
|---|---|---|
| discovery/read | principal, target/stream/status/result, tenant/mission, labels | conceal/filter before link, count, latest, history, event |
| submit | principal/source, target capability, parameters, time, purpose/context, policy versions | authenticate, authorize intent, validate, rate-limit, audit before durable admission |
| feasibility | requester, target, inputs/context, evaluator | authorize analysis and its disclosure separately from Command |
| pre-dispatch | latest actor authority, policy/safety/interlock inputs, target state, validity, revision | durable allow/deny decision before outbox becomes dispatchable |
| update/cancel | actor, current revision/phase, cutoff, target contract | compare-and-swap, re-evaluate gates, record decision/race |
| status/result ingest | reporter credential, represented executor, Command/revision, source sequence | exact reporter grant and transition/result validation |
| event publication | subscriber, resource/event, labels, current policy | filter before payload/topic/count and audit sensitive access |
| reconcile/retry | operator/service, uncertain evidence, validity/policy | prevent double effect and record rationale |

Authentication to a broker, possession of a Command ID, gateway network location, or status-shaped JSON does not grant reporting or cancellation authority. Credentials bind to registered principals and represented executors; source claims in payloads remain untrusted until that binding is established. **[A/P]**

### 14.2 Sensitive information and non-oracular behavior

ControlStream existence, supported command types, parameter bounds, target identity/location, timing, issuer, feasibility, schedule, progress, failure reason, result, dispatch topology, and executor health may all be sensitive. Policy must cover metadata and links as well as bodies. Unauthorized callers receive concealed or bounded errors without confirming hidden capabilities; logs/metrics avoid parameters and secrets by default. External links and result dereferencing require allowlists, size/type controls, and SSRF defenses. **[P]**

### 14.3 Audit and provenance

Audit every submission outcome, accepted revision, gate decision, dispatch attempt/ack/retry, status/result acceptance/rejection, cancellation request/decision, timeout/expiry, uncertain/conflicting outcome, correction/tombstone, operator override, and publication/security decision. Record actor, credential/session class, represented source, target/stream/Command/revision, action, input/output digests, policy/evaluator/interlock versions, decision/reason, times, correlation/causation, and affected evidence IDs without copying sensitive parameters unnecessarily. **[P]**

This is the required hook contract, not the final authorization/safety/audit design. IDR-SRV-038 must decide roles, approvals, safety classes/interlocks, abort authority, emergency behavior, and immutable audit schema. IDR-SRV-039–041 own AuthN/Z, policy/releasability, and accountability mechanisms. Until then, command execution and public tasking events stay disabled. **[P]**

## 15. Fixture, Conformance, Performance, and Interoperability Implications

### 15.1 Fixture corpus

Build version-pinned fixtures for:

- valid ControlStreams for scalar, record, array, choice, unit-constrained, synchronous, asynchronous, direct, gateway, broker, simulator, and manual execution;
- invalid/missing parent, controlled property, media type, schema location, nested `DataChoice`, units, allowed values, nil, relationship, time, and result variants;
- every legal public status and transition, repeated progress, direct completion, rejection after acceptance but before execution, and all illegal/terminal regressions;
- Command update before/after cutoff, revision dispatch, duplicate request, idempotency conflict, duplicate status, late/out-of-order status, same-time conflict, and reporter epoch change;
- cancellation before dispatch, while scheduled/executing, denial, too-late race, and unauthorized status POST;
- dispatch crash before/after commit, broker ACK without target acceptance, retry, expired queue item, execution timeout, ambiguous delivery, late resolution, and conflicting terminal evidence;
- inline, Observation, observation-set, DataStream/time-range, external, partial, invalid-multiple-variant, hidden, and broken-reference results;
- feasibility positive/negative/indeterminate/stale/context-changed flows; and
- DDIL spool/reconnect/gap/revocation and snapshot/catch-up event replay.

### 15.2 Conformance and contract tests

Conformance tests should exercise exact plural canonical/nested routes, links, collection envelopes, schemas, read-only fields, schema validation, UTC times, status enum/constraints, result variants, filters, CRUD/update claims, cancellation-versus-delete, and ATS requirements for every advertised class. Runtime routes, landing links, conformance declaration, OpenAPI, examples, and authorization behavior must agree. Published singular/copy defects require explicit negative/compatibility tests and documented disposition. **[N/P]**

Safety/contract tests additionally prove: no dispatch before Command/audit/outbox commit; no duplicate effect across retry/crash; no broker ACK-to-`ACCEPTED` shortcut; only authorized reporters affect state; no status after terminal; no arrival-wins regression; no stale feasibility as authority; no private `UNKNOWN` in the standard enum; and no public tasking publication while gates are disabled. **[A/P]**

### 15.3 Performance and reliability measures

Measure separately: submission validation/commit latency; gate latency; queue age; dispatch latency and attempts; target acceptance/start/completion latency; status/result ingestion and projection contention; per-Command history pagination; concurrent status/cancel/update races; outbox/event lag; retry/dedupe cost; gateway spool catch-up; and policy/audit overhead. Test hot ControlStreams, long-running Commands with many progress reports/results, large SWE payloads, target outage, broker backpressure, restart, and database failover. Numeric SLOs and limits remain IDR-SRV-054/deployment decisions. **[P]**

### 15.4 Interoperability

Test CSAPI Explorer, OS4CSAPI, Glaux web/mobile clients, CS-Go, OSH, SECD where safe/read-only, and deterministic gateway/simulator doubles. Assert semantic completeness, not parser success: parent/reference retention, schema discoverability and recursive parsing, `201/202`/Location/monitor behavior, initial and terminal statuses, update/cancel, result traversal, top-level/nested routes, current/history consistency, and policy filtering. Never execute commands against uncontrolled public systems. **[I/P]**

## 16. Downstream Topic Handoff Matrix

| Topic | Findings handed forward | Decision retained by downstream owner |
|---|---|---|
| IDR-SRV-037 Feasibility | separate durable advisory resource; same parameter schema; context/fingerprint/freshness; no reservation/auth | exact modes, result vocabulary, staleness/cache, synchronous/asynchronous contract |
| IDR-SRV-038 Command safety/audit | two-layer lifecycle; pre-dispatch gate; cancellation authority; reporter roles; uncertain outcome hooks | roles, approvals, safety classes/interlocks, abort/override, final audit fields |
| IDR-SRV-039 AuthN/Z | actor/source/executor separation and route/resource/action checks | protocols, credentials, token claims, PDP/PEP architecture |
| IDR-SRV-040 Policy/releasability | capability/status/result/event sensitivity; policy before link/count/latest/publication | label vocabulary, cross-boundary rules, redaction/concealment |
| IDR-SRV-041 Audit | complete transition/dispatch/reconciliation audit hook inventory | tamper evidence, schema, retention, export, legal hold |
| IDR-SRV-042 DDIL | explicit deferred queue constraints, validity recheck, unknown outcome, stable replay identity | operating modes, availability contract, offline enable/disable rules |
| IDR-SRV-043 Sync/conflict | causal status ordering, epochs/sequences, duplicate/conflicting/terminal evidence | cross-node authority and reconciliation protocol |
| IDR-SRV-044–049 Architecture/API/ops | logical stores, projection, common dispatcher port, adapter/evidence boundaries | Rust components, exact endpoints/extensions, jobs, telemetry/SLOs |
| IDR-SRV-050/052 Conformance/tests | lifecycle invariants, route/artifact contradictions, race/failure scenarios | executable harness and layered Rust test architecture |
| IDR-SRV-053 Fixtures | complete state/payload/dispatch/DDIL corpus | fixture packaging, generators, golden-file governance |
| IDR-SRV-054 Performance | separated phase metrics and stress dimensions | numeric targets, load models, capacity gates |
| IDR-SRV-055 Security tests | unauthorized command/status/cancel/result, non-oracular and double-effect tests | adversarial matrix/tooling and release gates |
| IDR-SRV-056 Interoperability | route/schema/status/result/client matrix | exact partner versions/environments and execution policy |
| Final synthesis/implementation guide | ControlStream contract, two-layer lifecycle, persistence/dispatch/public mapping | integrated architecture and implementation sequence |

## 17. Recommendations

1. **Adopt the two-layer lifecycle.** Keep the exact nine CSAPI codes at the public boundary and model orchestration dimensions privately.
2. **Commit before dispatch.** Atomically persist Command, initial evidence, identity/provenance, gates, audit, and outbox; all external effects are post-commit.
3. **Make history authoritative and projection rebuildable.** Validate causal transitions and reporter authority; never use last arrival as current state.
4. **Version ControlStream contracts immutably.** Pin schema/media/semantic revisions on every Command and create a new stream/version for incompatible change.
5. **Use one dispatch port and durable attempt model.** Direct, gateway, broker, simulator, and manual patterns differ only through registered adapters and authority bindings.
6. **Never promote delivery acknowledgement.** Broker/gateway transport ACK is not `ACCEPTED`, `EXECUTING`, or `COMPLETED`.
7. **Mediate cancellation.** DELETE remains deletion; commit `CANCELED` only where cancellation is authoritative, otherwise retain a private request until resolution.
8. **Represent expiry, timeout, and uncertainty without invented status codes.** Use reasoned standard outcomes only when their meanings are certain; otherwise preserve last status plus an authorized certainty view.
9. **Keep feasibility separate and advisory.** Link by stable identity/input fingerprint; re-evaluate stale context; never treat it as authorization or reservation.
10. **Validate exact payload contracts.** JSON first; recursive SWE semantics, units, constraints, choices, and results; no silent command conversion/repair.
11. **Separate result, effect, and observation evidence.** Completion is executor evidence; corroborating actual state remains an Observation when needed.
12. **Keep tasking disabled until safety work completes.** Do not enable inbound draft Part 3 command publication or claim applicable tasking mutation support before IDR-SRV-037/038 and security gates are accepted and tested.
13. **Turn artifact defects into profile tests.** Use plural routes from tagged OpenAPI/CRUD clauses, document errata, and make runtime discovery/OAS/links agree.
14. **Build the deterministic lifecycle simulator early.** It is the safest way to exercise every transition, race, retry, DDIL, and client path before real actuators.

Suggested implementation sequence after all owning research is accepted: immutable schema/Command/status/result stores and projection; idempotency/audit/outbox; lifecycle validator; simulator/manual adapter; direct/gateway adapter; feasibility and safety gates; read/query/client flows; event publication; brokered/DDIL dispatch; then controlled interoperability and performance testing. **[P]**

## 18. Risks, Constraints, and Open Questions

| Risk/constraint/open question | Disposition |
|---|---|
| Published singular/plural and copy-text defects | profile plural tagged routes; register/test; monitor corrigendum |
| `UPDATED` lacks stable phase semantics | preserve internal effective phase; emit follow-up stable status where needed; test clients |
| Standard cancellation representation can imply stop before target confirmation | enable only with authoritative cancellation contract; otherwise private request/reject capability |
| Optional CRUD/update permits status/result mutation while evidence should persist | immutable revisions/tombstones plus rebuildable projection/audit; conformance claim scoped truthfully |
| `sender` is a string and not sufficient identity | derive from authenticated principal; bind internal immutable identity/provenance |
| Report time can be late, skewed, duplicated, or malicious | reporter authority, sequence/epoch, bounds, causal validation, conflict state |
| Exactly-once physical actuation generally cannot be guaranteed | stable command/attempt identity, target dedupe/query, no fresh-ID retry, explicit uncertainty |
| Completion may not equal desired real-world state | use CommandResult and corroborating Observations separately |
| Feasibility may become stale or be mistaken for reservation | IDR-SRV-037 context/freshness contract; explicit non-reservation default |
| Capability/status disclosure may expose mission intent | policy before discovery/query/link/event; non-oracular errors; protected audit |
| DDIL queue can execute stale or revoked intent | bounded validity; reconnect rechecks; safe default disable per profile |
| Multiple authoritative reporters may conflict | exact per-status grants and precedence/reconciliation owned by 038/043 |
| Exact synchronous response body/status remains implementation-profile sensitive | reconcile OAF Part 4/current OpenAPI in API topic and test exact contract |
| Whether `CANCELED` denotes decision or confirmed physical cessation | IDR-SRV-038 must define per dispatch profile and operator presentation |
| Whether terminal evidence correction may alter public current status | conformance/legal/audit policy decision in 041/050; never erase original evidence |
| Numeric timeouts, retention, quotas, retry limits, and SLOs | deferred to architecture/deployment/performance topics |

None of these prevents acceptance of the lifecycle model. They prevent premature implementation claims and are assigned to explicit downstream owners. **[P]**

## 19. Validation Against This Plan's Success Criteria

| Success criterion | Result | Evidence |
|---|---|---|
| Distinguish ControlStream, definition, request, resource, execution, status, result, feasibility, event, audit | Met | §§5–7 |
| Document System/property/SensorML/SWE/gateway/feasibility/safety/event relationships | Met | §§6, 10, 12–14 |
| Define lifecycle, transitions, terminality, async, cancellation, timeout, failure, unknown | Met | §§8–9; Appendix A |
| Define payload validation, dispatch, reporting, persistence, query, events, DDIL, provenance | Met | §§9–14 |
| Define authorization, safety, audit, policy, fixtures, conformance, performance, interoperability implications | Met | §§14–16 |
| Incorporate implementation/community evidence non-normatively | Met | §§3, 10–11, 15 |
| Produce bounded, decision-usable recommendations | Met | §§16–18 |
| Make downstream handoffs explicit | Met | §16 |
| Provide explicit reproducible references | Met | §20; Appendix B |

Research phases 1 through 6, deliverable drafting, and author review are complete. Plan-owner acceptance remains deliberately unchecked while this report is **In Review**. IDR-SRV-037 is not authorized by this report.

## 20. References and Sources

### 20.1 Standards and official artifacts

1. [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html).
2. [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html).
3. [Official CSAPI repository tag `v1.0.0`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0), including [Part 2 OpenAPI](https://github.com/opengeospatial/ogcapi-connected-systems/blob/v1.0.0/api/part2/openapi/openapi-connectedsystems-2.yaml) and [JSON schemas](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api/part2/json).
4. [OGC SensorML Encoding Standard, OGC 23-000, Version 3.0](https://docs.ogc.org/is/23-000/23-000.html).
5. [OGC SWE Common Data Model Encoding Standard, OGC 24-014, Version 3.0](https://docs.ogc.org/is/24-014/24-014.html).
6. [W3C/OGC Semantic Sensor Network Ontology](https://www.w3.org/TR/vocab-ssn/).
7. [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110) and [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457).

### 20.2 Implementation and interoperability evidence

8. [Connected Systems Go `v1.0.4`](https://github.com/SomethingCreativeStudios/connected-systems-go/tree/244f4dd586da685d4d9b75e43f73001028b5bd0e).
9. [OpenSensorHub core `v2.0.2`](https://github.com/opensensorhub/osh-core/tree/235c0eabf24b6d6137b499b4402943d2794b70e6).
10. [OS4CSAPI client phase-9](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230).
11. [SECD interoperability repository](https://github.com/Sam-Bolling/csapi-server-interop-secd/tree/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0).

### 20.3 Project sources

12. [Accepted IDR-SRV-014A OSH study](idr-srv-014a-osh-csapi-server-implementation-study-report.md).
13. [Accepted IDR-SRV-014B CS-Go study](idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md).
14. [Accepted IDR-SRV-014E client smoke findings](idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md).
15. [Accepted IDR-SRV-014F SECD interoperability findings](idr-srv-014f-secd-interoperability-findings-study-report.md).
16. [Accepted IDR-SRV-014G community lessons](idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md).
17. [Accepted IDR-SRV-021 SensorML strategy](idr-srv-021-sensorml-representation-strategy-report.md), [IDR-SRV-022 SWE strategy](idr-srv-022-swe-common-data-component-strategy-report.md), [IDR-SRV-023 validation strategy](idr-srv-023-schema-and-encoding-validation-strategy-report.md), and [IDR-SRV-024 unit/semantic strategy](idr-srv-024-units-observed-properties-and-semantic-binding-strategy-report.md).
18. [Accepted IDR-SRV-025 persistence options](idr-srv-025-database-and-persistence-architecture-options-report.md), [IDR-SRV-029 transaction strategy](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md), and [IDR-SRV-030 retention/lifecycle strategy](idr-srv-030-data-lifecycle-retention-archival-and-deletion-strategy-report.md).
19. [Accepted IDR-SRV-031 write model](idr-srv-031-server-write-and-ingestion-model-report.md), [IDR-SRV-032 publisher boundary](idr-srv-032-publisher-to-server-contract-boundary-report.md), [IDR-SRV-033 simulator boundary](idr-srv-033-simulator-to-server-contract-boundary-report.md), [IDR-SRV-034 dynamic semantics](idr-srv-034-datastream-observation-and-status-update-semantics-report.md), and [IDR-SRV-035 publication strategy](idr-srv-035-streaming-and-event-publication-strategy-report.md).
20. [OGC Connected Systems upstream-history register, Version 1.12](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md).
21. Controlled AEP baseline `AC/224(JCGISR)D(2026)0005`, 27 April 2026, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`; available only through authorized project custody and accepted releasable findings; not redistributed.

### 20.4 Numbered source notes

[^1]: OGC 23-002 §10.11/Table 14 defines the nine codes, their meanings, execution-time rules, progress behavior, and final states. Retrieved and checked against repository tag `v1.0.0` on 2026-09-15.
[^2]: OGC 23-002 §14.5 explicitly distinguishes cancellation from HTTP DELETE and describes cancellation through creation of a `CANCELED` status report. The additional request-versus-confirmation mediator is a Glaux safety recommendation, not quoted standard behavior.
[^3]: OGC 23-002 identifies approval on 2025-06-02, publication on 2025-07-16, document 23-002, and Version 1.0. Its §§10–11 define ControlStreams/Commands and Feasibility.
[^4]: SOSA/SSN §4.4.2 defines Actuation, Actuator, actuatable property, Procedure, feature of interest, and result relationships. CSAPI Part 2 maps ControlStream/Command to this conceptual basis.
[^5]: OGC 23-002 §10.7 states that server-reported `issueTime` and `currentStatus` are required, parameters are required, and server-generated/read-only values supplied on create/update should be ignored. Its JSON constraint requires parameter encoding against the parent schema.
[^6]: SensorML 3.0 §8.2.9.1 permits process inputs, outputs, and parameters to use ObservableProperty, SWE Common AbstractDataComponent, or DataStream forms.
[^7]: SWE Common 3.0 defines scalar and aggregate data components, units, nil values, constraints such as AllowedValues, and JSON objects including recursively typed DataChoice members.

## Appendix A. Full Lifecycle Matrix

| Lifecycle state | Transition trigger | Source/client/gateway actor | Required validation | Feasibility implication | Authorization/safety implication | Persistence requirement | Event/publication requirement | Audit requirement | Client-visible status | Retry/cancel/timeout behavior | DDIL implication | Security/policy implication | Test/conformance implication | Downstream handoff | Notes / unresolved issues |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Prepared (pre-resource) | client constructs intent | client | local schema hints only; server remains authoritative | may reference prior analysis | no authority yet | none/server may stage bounded bytes | none | request telemetry only | none | retry has no server identity yet | client may hold locally | avoid capability leakage | invalid/missing fields | 044/053 | not a Command |
| Received/validating | authenticated request enters admission | API/client | route, media, size, identity, parent, static and parent schema, references, time | decide required/optional | authenticate and authorize request; cheap policy first | staged evidence only until commit | none before commit | admission attempt/result | HTTP in progress | same idempotency key on ambiguity | bounded listener behavior | non-oracular errors/rate limits | malformed/unauthorized/oversize | 038–040/055 | no public status until durable creation |
| Pending | atomic Command + initial status commit | Glaux admission service | all pre-create rules passed | absent, pending, or accepted reusable evidence | submission grant passed; later gates may remain | Command, revision, `PENDING`, provenance, idempotency, audit, outbox | durable created/status event eligible but tasking channel stays gated | mandatory | `PENDING` | safe retry returns same Command; cancellation per profile; validity active | may queue only if declared | policy filters all reads/events | `201/202`, crash, duplicate | 037–043/050 | received but no accept/reject decision |
| Gate pending (internal) | async feasibility/auth/safety work starts | evaluators/services | evaluator inputs/version/freshness | active analysis | no dispatch until all required gates pass | gate work/decision evidence | private progress only unless profiled | each decision/timeout | `PENDING` | retry evaluator idempotently; cancel request allowed | recheck stale context after reconnect | least privilege/separation of duties | denial, timeout, stale evidence | 037–041 | never serialize as statusCode |
| Accepted | receiving system accepts initial processing | target/gateway or authoritative local executor | legal transition, reporter, revision, initial target validation | does not imply feasibility remains current | required gates passed at accepted scope; still may later reject/fail | append status and projection atomically | durable status event after policy/gate | mandatory | `ACCEPTED` | dispatch/schedule retries use same identity; cancel allowed | may wait connected/schedule | target acceptance must be authenticated | false broker-ACK mapping | 038/043/055 | not execution |
| Queued (internal) | ready work awaits destination/capacity | dispatcher | validity, revision, priority/dependencies | recheck at configured horizon | recheck before dispatch | queue/outbox and scheduling evidence | private operational event | enqueue/dequeue/reprioritize | last valid `PENDING`/`ACCEPTED` | bounded retry; cancel; expiry timer | primary DDIL state | client cannot self-assert priority | starvation/restart/revocation | 042/045/054 | no invented code |
| Scheduled | receiving system effectively schedules | executor/scheduler | reporter, execution time, revision, state | may use but is not created by feasibility | schedule and policy valid | append status/project execution estimate | durable status event | mandatory | `SCHEDULED` | update/cancel before cutoff; timeout tracked | delayed execution permitted within validity | schedule disclosure policy | required time; reschedule race | 037/038/042 | scheduled time required |
| Updated | receiving system accepts Command revision | executor after client update | writable fields, cutoff, new schema/gates, CAS | possibly rerun | rerun affected gates; exact update authority | immutable Command revision + `UPDATED` | revision/status event | old/new digest and decision | `UPDATED`, then effective stable phase | same Command ID/new revision; cancel/time bounds updated safely | offline update reconciliation | no target/safety-class swap by stealth | before/after cutoff, duplicate revision | 038/043/050 | event-like, not stable execution phase |
| Dispatching (internal) | outbox worker claims ready revision | dispatcher/adapter | current revision, gates, validity, destination | recheck if required | final pre-dispatch safety/policy decision | attempt and lease/CAS | private attempt event | mandatory | last valid status | retry same attempt/identity; cancellation serialized | connection failure may produce retry/unknown | credentials isolated per adapter | crash before/after send | 038/042/045/055 | no public dispatch code |
| Delivered/transport ACK (internal) | adapter/broker confirms handoff | gateway/broker | attempt correlation, authentic ACK | none | no promotion to domain authority | delivery evidence | private delivery event | mandatory | unchanged | query target before ambiguous retry | session/QoS not canonical recovery | broker ACL not command authorization | ACK without target acceptance | 035/038/042 | never automatically `ACCEPTED` |
| Target acknowledged (internal) | application protocol confirms receipt | target/gateway | semantic mapping and reporter identity | none | must satisfy registered target contract | target acknowledgement | private or maps via next status | mandatory | unchanged or `ACCEPTED` only if exact semantics met | stable-ID retry/query | delayed ACK reconciled | represented executor grant | ambiguous ACK semantics | 038/043/056 | transport and application ACK distinct |
| Executing | receiving system begins/performs action | target/executor reporter | legal transition, reporter, progress/time | feasibility no longer outcome authority | safety monitoring/abort hooks remain | append status/progress; projection | each material progress status eligible | mandatory | `EXECUTING` | repeated progress; cancel if supported; execution timeout | disconnect causes uncertainty, not failure | sensitive progress/results | progress monotonicity and race | 038/042/054 | repeated reports allowed |
| Cancel requested (internal) | authorized actor asks stop/not execute | client/operator/service | actor, state, revision, cutoff, idempotency | none | cancellation-specific policy/dual control | cancellation action/decision/outbox | private until authoritative outcome | mandatory | prior status | retry same action; deny/too-late possible | request may queue only if still safe/valid | capability-sensitive; exact actor evidence | cancel/complete race | 038/041/043/055 | not yet `CANCELED` |
| Canceled | authoritative cancellation decision/confirmation | configured authoritative mediator/executor | terminal transition and cancellation authority | none | user-driven cancellation semantics | append terminal status/projection | durable terminal event | mandatory | `CANCELED` | no further status; abort effort/anomalies separate | late conflicting evidence is incident | restrict reason/actor | terminal and late completion | 038/041/043 | DELETE is different |
| Rejected | receiving system decides it will not execute | admission/target/policy mediator | pre-execution certainty and safe reason | infeasible may cause reasoned rejection | validation/policy/safety/expiry reason classified | append terminal status/projection | durable terminal event | mandatory | `REJECTED` | no further status; new intent needs new/revised allowed request | queued expiry may map only if nonexecution certain | safe diagnostic/concealment | never after known execution | 037–040/050 | final; “won't execute at all” |
| Completed | successful execution reported | target/executor | authoritative reporter, legal state, actual execution time, result validation | no effect | terminal safety/audit | append status/results/projection | durable terminal/result event | mandatory | `COMPLETED` | no status retry except idempotent duplicate | late valid completion can resolve private unknown | result disclosure separate | required time, duplicate/result link | 038/041/043/056 | not proof of observed final world state |
| Failed | execution/processing failure reported | target/executor | authoritative reporter, post-acceptance semantics, reason/time | no effect | incident/safety hooks | append terminal status/projection | durable terminal event | mandatory | `FAILED` | no further status; retry physical intent requires explicit new decision | disconnect alone insufficient | diagnostic redaction | pre-admission misuse, timeout | 038/041/042/055 | not schema/auth rejection |
| Expired (internal) | validity/deadline passes | timer/orchestrator | clock, state, certainty, target nonexecution | stale analysis invalid | policy determines safe outcome | timer evidence and reason | private plus mapped terminal event if certain | mandatory | `REJECTED` only when nonexecution certain; otherwise unchanged | no dispatch; cancel/reconcile as applicable | common deferred-queue case | trusted time and policy version | expiry before/after send | 037/038/042 | never public `EXPIRED` code |
| Timed out (internal) | processing/execution deadline passes | timer/executor | phase and outcome certainty | none | abort/escalation decision | timeout/attempt evidence | private plus `FAILED` only when authoritative | mandatory | `FAILED` only for known ended execution; otherwise unchanged | query/reconcile; no blind new-ID retry | common lost-contact case | operator escalation | dispatch vs execution timeout | 038/042/048 | never public `TIMED_OUT` code |
| Unknown/reconciliation required (internal) | possible delivery/effect with insufficient evidence | orchestrator/gateway/operator | attempt history, target query, source sequence, conflict rules | analysis irrelevant to actual outcome | restrict further effects until safe decision | certainty projection and all evidence | private authorized event; public freshness metadata only | mandatory | last valid CSAPI status | stable-ID query/retry only if proven idempotent; operator path | central DDIL behavior | highly sensitive incident state | late resolution/conflicting terminal | 041–043/055 | never public `UNKNOWN` statusCode |
| Archived/tombstoned | retention/lifecycle action | records manager/admin | retention, legal hold, relationships, projection rebuild | none | privileged policy | preserve required evidence/revision/tombstone | lifecycle event if authorized | mandatory | historical status subject to policy | no execution action | replication/retention policy | conceal/redact lawfully | delete/correction/current consistency | 030/040/041/050 | management state, not CommandStatus |

## Appendix B. Reproducible Evidence Record

### B.1 Immutable source checks

```powershell
git ls-remote https://github.com/opengeospatial/ogcapi-connected-systems.git refs/tags/v1.0.0 refs/heads/master
git ls-remote https://github.com/SomethingCreativeStudios/connected-systems-go.git refs/tags/v1.0.4
git show v1.0.0:api/part2/standard/sections/clause_9_requirements_class_controlstreams.adoc
git show v1.0.0:api/part2/standard/sections/clause_10_requirements_class_command_feasibility.adoc
git show v1.0.0:api/part2/standard/sections/clause_15_requirements_class_create_replace_delete.adoc
git show v1.0.0:api/part2/standard/sections/clause_16_requirements_class_update.adoc
git show v1.0.0:api/part2/standard/sections/clause_20_requirements_class_json_encoding.adoc
git show v1.0.0:api/part2/standard/sections/clause_21_requirements_class_swecommon_json_encoding.adoc
```

Observed pins on September 15, 2026:

```text
CSAPI Parts 1/2 v1.0.0   8e03b236a049849f2ccc24b4fd9fdce5ff69bed2
CSAPI master              3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f
CS-Go v1.0.4              244f4dd586da685d4d9b75e43f73001028b5bd0e
OSH v2.0.2                235c0eabf24b6d6137b499b4402943d2794b70e6
OS4CSAPI phase-9          754411897173c2ec4debaa9bcf4ed9e0f8a9e230
SECD evidence             f018fd129bf0d0d1ce75e68198e3ab4d99d937a0
```

### B.2 Artifact findings reproduced

- `command.json`, `commandStatus.json`, `commandResult.json`, and `controlStream.json` were inspected at the tag.
- The status enum contains exactly the nine codes in §8; result schema requires exactly one data/reference alternative.
- The tagged OpenAPI uses plural `/controlstreams` and `/commands` routes; normative prose contains singular/copy defects; the root OAS does not list feasibility paths.
- The CRUD clause distinguishes status-based cancellation from DELETE and requires `400` for a Command that violates its parent schema.
- The advanced-filtering clause defines Command time/status/sender/FOI and status-code filters.
- CS-Go `v1.0.4` source and E2E tests confirm separated resources, initial `PENDING`, status-derived `currentStatus`, schema checks, pagination, and MQTT status handling; repository inspection confirms direct last-write projection behavior.
- Accepted OSH, OS4CSAPI, and SECD study evidence was re-read for feasibility absence, route/response divergence, incomplete lifecycle traversal, and nested SWE schema hazards.

### B.3 Report validation targets

```powershell
rg -n '^## ' idr-srv-036-control-stream-and-command-lifecycle-model-report.md
rg -n '^\[\^[0-9]+\]:' idr-srv-036-control-stream-and-command-lifecycle-model-report.md
git diff --check
```

## Report Completion Checklist

- [x] Topic ID matches the overall research plan index
- [x] Topic research plan is linked and aligned
- [x] All core and detailed research questions are answered or explicitly handed off
- [x] Normative, accepted, implementation, inference, and recommendation evidence are distinguished
- [x] Mutable implementation sources identify exact versions/commits and retrieval date
- [x] Controlled-source and artifact limitations are explicit
- [x] ControlStream and Command concepts and ownership are distinguished
- [x] Public and private lifecycle states and allowed transitions are defined
- [x] The required 16-column lifecycle matrix is complete
- [x] Submission, dispatch, cancellation, timeout, unknown, status, result, query, persistence, event, replay, and DDIL behavior are covered
- [x] Feasibility, safety, security, policy, audit, fixture, conformance, performance, and interoperability handoffs are explicit
- [x] Recommendations are decision-usable and bounded to Glaux Server
- [x] References and evidence checks are reproducible
- [x] Report is ready for plan-owner review
- [x] Report accepted by plan owner

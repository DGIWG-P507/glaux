# Section 037: Feasibility and Asynchronous Tasking Strategy - Research Report

**Topic ID:** IDR-SRV-037<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-037 Feasibility and Asynchronous Tasking Strategy](../IDR%20Plans/idr-srv-037-feasibility-and-asynchronous-tasking-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All seven question groups and 37 detailed questions concerning the normative baseline, public resource/API contract, task state model, validation and feasibility, durable Rust implementation, security/DDIL, and conformance/interoperability<br>
**Methodology Used:** Authority-ranked standards extraction; immutable artifact and implementation pins; accepted-baseline reconciliation; public/private state modeling; HTTP/resource-contract analysis; feasibility evidence and invalidation analysis; failure-window and recovery analysis; implementation-option comparison; scenario traceability<br>
**Research Time:** Approximately 23 hours of AI-assisted execution on September 15, 2026<br>
**Approved Standards Baseline:** OGC 23-001 and OGC 23-002 Version 1.0, repository tag `v1.0.0` at [`8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Draft Dependency Check:** OGC API - Features - Part 4 draft `20-002r2`, `1.0.0-SNAPSHOT`, repository commit [`4e30324a14b682ff4a26ee43aad1eb6428c846a3`](https://github.com/opengeospatial/ogcapi-features/commit/4e30324a14b682ff4a26ee43aad1eb6428c846a3), checked September 15, 2026<br>
**Implementation Evidence:** CS-Go `v1.0.4` at [`244f4dd586da685d4d9b75e43f73001028b5bd0e`](https://github.com/SomethingCreativeStudios/connected-systems-go/commit/244f4dd586da685d4d9b75e43f73001028b5bd0e); OpenSensorHub `v2.0.2` at [`235c0eabf24b6d6137b499b4402943d2794b70e6`](https://github.com/opensensorhub/osh-core/commit/235c0eabf24b6d6137b499b4402943d2794b70e6); OS4CSAPI phase-9 at `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`; SECD at `f018fd129bf0d0d1ce75e68198e3ab4d99d937a0`<br>
**Supporting Resources:** Accepted IDR-SRV-007/008/013/015-024/029/031-036, controlled-AEP findings, and upstream-history register Version 1.12<br>
**Document Purpose:** Establish a standards-aligned and implementable feasibility and synchronous/asynchronous tasking baseline without implementing the server, inventing a jobs API, defining final command authorization/safety policy, or treating feasibility as a reservation<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 15, 2026<br>
**Date:** September 15, 2026<br>
**Last Updated:** September 15, 2026

---

## Evidence and Decision Legend

| Mark | Meaning |
|---|---|
| N | Normative or controlling published-standard evidence |
| A | Accepted Glaux design baseline |
| I | Pinned implementation or interoperability observation |
| E | Engineering analysis or inference from cited evidence |
| P | Proposed Glaux decision requiring report acceptance |
| X | Known defect, ambiguity, or unresolved upstream proposal |

The marks classify claims; they are not conformance levels. “Must” and “shall” are used for actual obligations or consequences of an accepted/project-profile decision. “Should” identifies a recommendation awaiting acceptance.

---

## Contents

1. Executive Decision Summary
2. Scope, Terminology, and Evidence Method
3. Published CSAPI Behavior Baseline
4. Feasibility Semantics and Result Model
5. Synchronous and Asynchronous Tasking Lifecycle
6. Validation and Decision-Gate Boundaries
7. Recommended Glaux Public API Behavior
8. Recommended Rust Domain and Component Architecture
9. Persistence, Concurrency, Idempotency, and Recovery
10. Gateway, DDIL, Security, and Information-Exposure Boundaries
11. Conformance and Test Strategy
12. Phased Implementation Recommendation
13. Risks, Deferred Decisions, and Open Questions
14. Traceability and References
Appendices A-D

---

## 1. Executive Decision Summary

Glaux should implement feasibility as a **durable, advisory, Command-shaped analysis exchange** and implement asynchronous tasking as a **processing mode of the standard Command or Feasibility resource**, never as a second public job model. `ControlStream.async` selects the client-visible lifecycle for all submissions on that ControlStream. It does not select a Rust language feature, broker, worker deployment, or HTTP `202` response.[^1] **[N/A/P]**

For the first Glaux profile, a successful POST to either `{api_root}/controlstreams/{csId}/commands` or `{api_root}/controlstreams/{csId}/feasibility` returns `201 Created` and `Location` naming the new canonical Command or Feasibility resource. An asynchronous submission atomically creates the resource, its initial `PENDING` status, idempotency/provenance/audit evidence, and durable work/outbox row; the body is that initial status report. A synchronous submission completes before the response and returns one persisted terminal `CommandStatus` body—only `COMPLETED`, `REJECTED`, or `FAILED`—with the same `201` and canonical-resource `Location`. `Content-Location` identifies the returned status resource. HTTP `202` is not used merely because `ControlStream.async=true`; it is reserved for a future case in which the HTTP mutation itself is queued and has a useful operation-monitor contract.[^2] **[N/A/P]**

The safest synchronous implementation is not an unrecorded inline device call. Glaux should durably stage a private synchronous admission/work record and idempotency identity, dispatch only after commit, await its terminal result, and atomically publish the standard Command/Feasibility, one terminal status, result, audit, and outbox evidence before replying. The private record is excluded from CSAPI queries until publication. A disconnect does not erase the work; a same-key retry rejoins or returns the completed outcome. This preserves the one-status synchronous contract and commit-before-effect rule without exposing invented status codes. Synchronous ControlStreams should be enabled only where their configured end-to-end deadline is realistically bounded. **[A/E/P]**

Feasibility `COMPLETED` means that analysis completed, not that the requested action is feasible. The result carries the decision. Glaux should advertise an exact SWE Common `feasibilityResultSchema` per ControlStream with a required `decision` category (`YES`, `NO`, `CONDITIONAL`, or `INDETERMINATE`), required assessment time and opaque context fingerprint, and profile-selected constraints, alternatives, confidence, validity, and estimated timing fields. `REJECTED` means the analysis will not run; `FAILED` means accepted analysis failed. A successful analysis concluding uncertainty is `COMPLETED + INDETERMINATE`, not `FAILED`. **[N/E/P]**

Every result is bound to the submitted parameter digest, target and ControlStream versions, evaluator version, assessed time, state inputs/watermarks, gateway observation time, and validity horizon. It is advisory and creates no reservation, authority, safety approval, target acceptance, or execution guarantee. A later Command may reference the result in Glaux provenance, but submission must revalidate the command and re-evaluate every invalidated or expired predicate. A cache hit creates a new Feasibility resource unless the request is an idempotent replay; it records the source result and the freshness decision. Upstream issue #59 still proposes reservation/confirmation behavior but remains open and unadopted.[^3] **[N/A/E/P/X]**

The first implementation should use the Rust API process plus bounded in-process workers over PostgreSQL-backed durable work, status, result, idempotency, and outbox tables. Workers claim short-lived leases with row locking, perform external work outside the transaction, and append outcomes through revision/fencing checks. PostgreSQL explicitly identifies `SKIP LOCKED` as suitable for multiple consumers of a queue-like table.[^4] A separate worker process is a prepared deployment option; an external broker is a delivery adapter, not authority. Adopt either only when measured isolation, scaling, reachability, or integration requirements justify it. Rust `async fn` remains an execution mechanism that yields a `Future`; it is not persistence or recovery.[^5] **[E/P]**

This report does not authorize tasking implementation or conformance claims. IDR-SRV-038 still owns command authorization, safety/interlocks, approval, cancellation authority, sensitive diagnostics, and command-specific audit decisions. IDR-SRV-042/043 own final DDIL synchronization/conflict behavior, and IDR-SRV-044/045 own final crate/framework and service decomposition choices. **[A]**

### 1.1 Recommended baseline at a glance

| Decision | Rationale | Implementation consequence | Important alternative | Confidence |
|---|---|---|---|---|
| Standard resources, no public jobs API | Part 2 already supplies resource, status, and result endpoints | All client tracking uses Command/Feasibility links | Separate jobs endpoint | High |
| `201 + Location` for accepted creation in both modes | The resource is created; mode concerns later domain processing | One response contract; body differs by lifecycle point | `202` for every async stream | High |
| One terminal status body for sync; `PENDING` status body for async | Matches Part 2 lifecycle while keeping creation semantics explicit | `Content-Location` names status item | Empty 201 or Command body | Medium-high; Part 2 does not state exact HTTP status/body combination |
| Durable private staging for sync | Reconciles one public status with commit-before-effect/recovery | Hidden work record and wait/rejoin path | Inline unrecorded dispatch | High |
| Advisory, evidence-bound feasibility | The standard defines analysis, not reservation | Freshness manifest and recheck at command time | Treat YES as capacity hold | High |
| PostgreSQL queue plus in-process workers first | Uses accepted authority and minimum operational surface | Lease/fence/retry worker module in API deployment | Broker or separate worker immediately | High |
| Broker and separate worker behind measured triggers | Avoids unnecessary authority split while retaining growth path | Stable task/gateway ports and outbox | General workflow engine | High |

---

## 2. Scope, Terminology, and Evidence Method

### 2.1 Included and excluded scope

Included scope is the Part 2 command/feasibility resource contract, sync/async response and status behavior, result forms, validation-to-execution boundaries, feasibility evidence/freshness, durable work, concurrency, retries, restart recovery, cancellation/update implications, gateway delegation, DDIL effects, diagnostics, and verification.

Excluded scope is the policy content of authorization and safety decisions; control ownership and transfer; general audit schema; production SLO numbers; final Rust libraries; final service/deployment topology; a general workflow engine; draft Part 3 inbound tasking; and server code. Those exclusions are deliberate boundaries, not omissions.

### 2.2 Terms that must remain distinct

| Term | Meaning in this report | Not equivalent to |
|---|---|---|
| Synchronous | One terminal status report is returned after processing on a ControlStream with `async=false` | Blocking a thread; absence of durable work |
| Asynchronous | A resource can progress through several status reports on a ControlStream with `async=true` | Rust `async`; HTTP `202`; broker delivery |
| Admission | Decision to create/stage durable work after request-level checks | Target acceptance or execution |
| Validation | Syntax, schema, semantics, references, version, liveness, and temporal checks | Feasibility, authorization, or safety approval |
| Feasibility | Context-bound prediction/analysis of whether and how a valid intent could be performed | Reservation, authority, acceptance, dispatch, or success |
| Acceptance | Receiving-system decision to process a request | HTTP acceptance or successful completion |
| Dispatch | Attempt to deliver an accepted intent to an execution boundary | Transport acknowledgement or physical effect |
| Result | Data produced by command execution or feasibility analysis | Status, proof of final real-world state, or audit |
| Durable task | Restart-recoverable internal work with identity, state, and evidence | A spawned future |

### 2.3 Evidence method and authority

The published OGC 23-002 Version 1.0 text and tag `v1.0.0` control. Part 1, SensorML 3.0, SWE Common 3.0, RFC 9110, and the referenced Features transaction draft constrain their delegated subjects. Accepted Glaux reports constrain project design. Pinned implementation sources demonstrate possibilities and defects but cannot amend the standard.

The upstream-history review was bounded to issues [#1](https://github.com/opengeospatial/ogcapi-connected-systems/issues/1), [#58](https://github.com/opengeospatial/ogcapi-connected-systems/issues/58), and [#59](https://github.com/opengeospatial/ogcapi-connected-systems/issues/59). All remained open on September 15, 2026, labeled for pre-SWG review with a working recommendation to defer or re-scope, and had no linked branch or pull request. Issue #1's cross-domain Action proposal therefore does not change Command semantics; #59 creates no reservation feature; and #58 remains a safety/authority handoff to IDR-SRV-038. The shared register remains Version 1.12 because no material disposition changed. **[I/X]**

The current Features Part 4 source remained draft `20-002r2`/`1.0.0-SNAPSHOT` at `4e30324a...`. It allows `201`, or `202` when a creation operation itself is queued, and explicitly says the draft does not specify a mechanism for determining a queued operation's outcome.[^6] This is dependency evidence, not a final OGC Standard and not permission to turn CSAPI status into an HTTP-operation monitor. **[I/X]**

### 2.4 Evidence limitations

- The controlled AEP package was not redistributed. Only accepted, releasable project findings were used; classified or unreleasable operational policy is not inferred.
- No scoped CS-Go `v1.0.4` or OSH `v2.0.2` feasibility route/lifecycle was found. Their absence is implementation evidence, not proof that feasibility is optional when its conformance class is claimed.
- The OS4CSAPI corpus observed both `201` and `202` command behavior but did not establish one universal server contract. It reinforces the need for explicit response fixtures.
- Part 2 states the synchronous status body and valid terminal codes but does not explicitly choose the HTTP status or reconcile that body with inherited transaction response representations. The `201`/`Location`/`Content-Location` choice is a Glaux profile decision grounded in RFC 9110 and the draft dependency.
- No prototype was necessary to resolve the architecture choice. The concurrency strategy uses already accepted PostgreSQL transaction/idempotency/outbox rules and official locking semantics; implementation benchmarks remain future work.

---

## 3. Published CSAPI Behavior Baseline

### 3.1 Resource and processing model

A ControlStream groups Commands for the same receiving System with common controlled properties and a common parameter schema. Its required `async` Boolean states whether commands are processed asynchronously; `live` states whether commands are currently accepted. A Command contains required parameters and server-reported issue time/current status, plus timing, sender, target, Procedure, status, and result relationships as applicable. Each supported command format has a schema operation, and the payload must conform to that parent contract. **[N]**

Part 2 defines separate Command, CommandStatus, and CommandResult resources. Synchronous processing returns a single status report with `COMPLETED`, `REJECTED`, or `FAILED`. Asynchronous processing may emit several reports for acceptance/rejection, scheduling, updates, cancellation, execution, progress, completion, or failure. `reportTime` and `statusCode` are required; progress, message, execution time, and partial/final results are conditional or optional. `SCHEDULED` requires scheduled execution time; `COMPLETED` requires actual execution time.[^7] **[N]**

A Feasibility resource is a Command created on a ControlStream's feasibility channel. It uses the same parameter schema and has its own canonical path plus nested status and result collections. It can be processed synchronously or asynchronously. `SCHEDULED` and `UPDATED` are unused for feasibility. Results normally appear inline and conform to the parent `feasibilityResultSchema`, which is usually distinct from `resultSchema`. **[N]**

### 3.2 Standards traceability table

| Clause / requirement | Published obligation or behavior | Direct Glaux implication | Verification method |
|---|---|---|---|
| §10.6; Req. 25 | Expose schema for requested supported command format | Resolve exact versioned parent schema before admission | Format/schema discovery tests |
| §10.7-10.10; Req. 26-30 | Canonical Command `/commands/{id}`; root/nested collections and common collection rules | One identity across root and nested access; emit `canonical` link from alternate URL | Link graph and collection ATS plus supplemental plural-path tests |
| §10.11; Req. 31-32 | Status collection and per-Command nested endpoint | Status history is the public monitor; no job resource | Status navigation, pagination, time and status-code filters |
| §10.12; Req. 33-34 | Result collections and per-capable-Command endpoint | Store results independently; expose only when command can have result | Inline/reference result fixtures and link checks |
| §11; Req. 35 | Canonical Feasibility `/feasibility/{id}` | Stable resource identity distinct from later Command | Canonical/alternate retrieval and link test |
| §11; Req. 36 | Nested feasibility endpoint for each ControlStream | Implement plural `/controlstreams/{csId}/feasibility` as reconciled path | Supplemental path test; record singular prose defect |
| §11; Req. 37-38 | Status and result endpoints for every Feasibility | Empty result collection is retrievable before result; status is always retrievable | Pending/terminal polling tests |
| §11; Req. 39 | Feasibility collections optional; common rules if exposed | Do not claim optional custom collections initially | Conformance declaration and negative route test |
| §13; Req. 56-61 | Filter Commands by issue/execution time, status, sender, FOI and statuses by code when advanced filtering is claimed | Shared typed query path over durable projections/history | Exact-match, interval, combination, pagination and policy tests |
| §14; Req. 71-72 | CRD class creates/replaces/deletes Commands; invalid parent-schema payload gets `400` | POST creates Command; schema failure precedes creation | Mutation ATS plus no-side-effect negative test |
| §14; Req. 73-74 | CRD on Command status/result resources | Restrict writers by role but implement claimed operations; retain revisions internally | Authorized writer, ETag, revision, deletion-retention tests |
| §14; Req. 75 | CRD on Feasibility; normative nested replace/delete path omits `{id}` | Use ATS/update-consistent `/controlstreams/{csId}/feasibility/{id}`; document defect | Canonical and nested path fixtures |
| §14; Req. 76-77 | CRD on Feasibility status/result | Same controlled-writer and revision model as Command evidence | Role, ownership, schema, history tests |
| §15; Req. 85-91 | UPDATE class applies to Command/Feasibility/status/result | Support PATCH only when class is claimed; enforce accepted merge-patch/precondition profile | Draft dependency ATS plus supplemental CAS tests |
| §20; Req. 99-105 | JSON schemas and command/result constraints | Validate parameters and inline ordinary/feasibility data against distinct parent schemas | JSON Schema plus parent-contract tests; supplement A.105 feasibility omission |
| §21-23 | Supported SWE JSON/Text/Binary schemas and encodings control payloads | One logical validation pipeline with format-specific decoding | Golden and negative encoding corpus |

### 3.3 Path and ATS defect disposition

Glaux should implement the plural routes consistently used by the overview, neighboring requirements, transaction clauses, and OpenAPI: `/controlstreams/{csId}/commands` and `/controlstreams/{csId}/feasibility`; `/commands/{cmdId}/status`; `/commands/{cmdId}/result`; `/feasibility/{feasId}/status`; and `/feasibility/{feasId}/result`. Singular `/controlstream` in Requirement 36 and `/command` in Requirements 32/34 are published defects, not aliases to expose by default. Requirement 75's nested replace/delete route is completed with `/{id}`, as its ATS and Update requirement show. The official-versus-supplemental harness ledger must retain each divergence. **[N/A/E/P/X]**

The Annex A feasibility reference test incorrectly exercises `/commands`, and A.105 omits its feasibility-result-schema branch. Passing the official ATS therefore cannot prove those obligations. Supplemental tests are mandatory for the actual feasibility route, resource ownership, and `feasibilityResultSchema`. **[N/A/X]**

### 3.4 Mutation-class boundary

The base ControlStreams/Commands and Feasibility classes define retrieval and relationships; CRD and UPDATE are separate classes. Glaux's target profile remains the full mutation surface accepted in IDR-SRV-031, but implementation and conformance declarations must state each class independently. A user cancellation is a new `CANCELED` status, not DELETE. Replace/update changes the resource representation under the claimed transaction class; it does not erase prior internal revision, decision, result, or audit evidence. **[N/A]**

---

## 4. Feasibility Semantics and Result Model

### 4.1 What feasibility decides

Feasibility answers: “Given this exact valid intent and the identified, sufficiently fresh operational context, what does this evaluator conclude about potential execution?” It may consider target state, Deployment/configuration, requested time, queue/schedule conflicts, resource limits, gateway knowledge, environment inputs, and authorization-independent operational constraints. It does not decide that the caller may act, that the act is safe, that the target accepts it, or that execution will succeed. **[N/A/E/P]**

The public terminal pair must be interpreted as follows:

| Status and result | Meaning | Incorrect interpretation |
|---|---|---|
| `COMPLETED + YES` | Analysis succeeded and found the intent feasible within stated evidence/validity | Reservation or guarantee |
| `COMPLETED + NO` | Analysis succeeded and found the intent infeasible | Request rejected or evaluator failed |
| `COMPLETED + CONDITIONAL` | Analysis succeeded; feasibility depends on explicit conditions | Generic success without conditions |
| `COMPLETED + INDETERMINATE` | Analysis succeeded but evidence cannot support yes/no | Processing failure |
| `REJECTED` | Receiving system decided the feasibility analysis will not be processed | Infeasible command |
| `FAILED` | Accepted analysis failed during processing | Negative domain decision |
| `CANCELED` | Authorized user cancellation became authoritative | HTTP deletion |

### 4.2 Recommended logical result profile

Every feasibility-capable ControlStream should advertise an exact `feasibilityResultSchema`. The first Glaux logical profile should be a SWE Common `DataRecord`; its actual JSON/Text/Binary encoding follows the advertised schema and selected conformance class.

| Field/component | Cardinality | Purpose and rule |
|---|---:|---|
| `decision` Category | 1 | `YES`, `NO`, `CONDITIONAL`, or `INDETERMINATE`; a binary profile may restrict to YES/NO |
| `assessedAt` Time | 1 | Server/evaluator assessment time in the declared reference frame |
| `contextFingerprint` Text | 1 | Opaque digest of the versioned input manifest; not raw sensitive context |
| `validUntil` Time | 0..1 | Upper trust bound; omission means no reusable validity is asserted |
| `confidence` Quantity | 0..1 | Defined scale/meaning, normally `[0,1]`; never fabricate if evaluator cannot support it |
| `constraints` DataArray of DataRecords | 0..1 | Stable safe codes, outcome, severity, affected parameter/interval, and redacted detail |
| `alternatives` DataArray | 0..1 | Explicit schema-valid parameter/time alternatives; disclosure is policy-filtered |
| `estimatedExecutionTime` Time or TimeRange | 0..1 | Predicted timing, not a schedule or reservation |
| `steps` DataArray | 0..1 | Profile-defined analysis/execution steps when releasable |

This is a Glaux profile recommendation, not a claim that Part 2 standardizes these field names. Part 2 standardizes the placement and validation rule: inline data must match the exact `feasibilityResultSchema`. Profiles may use a simpler binary schema or richer mission-specific record, but clients never infer fields absent from discovery. **[N/E/P]**

### 4.3 Evaluation input manifest and provenance

The private evaluation manifest should bind:

- Feasibility ID, request digest, idempotency identity, submitter context, and requested format;
- System, Deployment, ControlStream, parameter-schema, feasibility-result-schema, Procedure, and target-feature identifiers and immutable versions/digests;
- normalized parameter values plus preserved source representation reference;
- current System/status/availability facts with source, event/report time, ingest time, sequence/epoch, confidence, and staleness;
- queue and schedule snapshot/version, relevant resource-availability version, and existing conflict set digest;
- gateway/adapter identity and version, connectivity observation time, remote state token, and clock-uncertainty estimate;
- evaluator algorithm/model/rule-set identifier and version;
- authorization-independent constraint package version; and
- assessment time, validity derivation, redaction policy/version, and provenance links.

The public result exposes only the safe subset or opaque digest. Protected audit/evidence retains the full manifest according to IDR-SRV-030 and later policy. This makes two apparently identical answers explainably different without leaking schedules, capabilities, or policy rules. **[A/E/P]**

### 4.4 Freshness, caching, and invalidation

A result is reusable only when the request digest and every declared dependency remain equivalent and the current time is before `validUntil`. Material changes to parameters, target, ControlStream or schemas, Procedure/Deployment, queue/schedule, resource state, System status/availability, gateway reachability or remote token, evaluator version, constraint package, or relevant source quality invalidate it. Clock uncertainty shortens—not extends—the validity horizon. **[A/E/P]**

A new non-idempotent feasibility submission always creates a new Feasibility identity even when the evaluator uses cached evidence. The new result cites the source result and records the cache/freshness decision. An idempotent replay with the same scope/key/digest returns the original resource and outcome. A same key with a different digest is a conflict. Cache lookup never bypasses caller authorization to request/view feasibility. **[A/P]**

A later Command may carry a private or profile-approved provenance link to a prior Feasibility result. Command admission rechecks schema/version identity and every volatile predicate; expired or invalidated evidence is recomputed or yields a reasoned `REJECTED` outcome after resource creation, according to mode. No result creates a hold on capacity. If a future operational profile needs reservation, it requires explicit resource/link/expiry/confirmation semantics and separate approval; open issue #59 is not enough. **[A/P/X]**

### 4.5 Gateway-delegated feasibility

When only a gateway/receiving System can evaluate feasibility, Glaux persists the same request and delegates through the task adapter. A verified remote result is normalized only into the advertised schema, retaining the remote correlation, principal, version, timestamps, and raw protected evidence. Gateway unreachability is not automatically `NO`:

- if analysis successfully determines that current evidence is insufficient, use `COMPLETED + INDETERMINATE`;
- if an accepted evaluation attempt fails technically, use `FAILED`;
- if the request will not be processed at all, use `REJECTED`; and
- if it is still legitimately pending under an asynchronous deadline, retain the last valid nonterminal public status and private retry state.

---

## 5. Synchronous and Asynchronous Tasking Lifecycle

### 5.1 State/response matrix

| Dimension | Synchronous Command | Asynchronous Command | Synchronous Feasibility | Asynchronous Feasibility |
|---|---|---|---|---|
| `ControlStream.async` | `false` | `true` | `false` | `true` |
| Immediate durable item | Private staged admission/work identity | Public Command + `PENDING` + work/outbox | Private staged admission/work identity | Public Feasibility + `PENDING` + work/outbox |
| Success HTTP | `201 Created` when terminal resource/status committed | `201 Created` when initial resource/status committed | `201 Created` when terminal resource/status/result committed | `201 Created` when initial resource/status committed |
| `Location` | `/commands/{id}` | `/commands/{id}` | `/feasibility/{id}` | `/feasibility/{id}` |
| Response body | One terminal CommandStatus | Initial `PENDING` CommandStatus | One terminal CommandStatus, normally with result | Initial `PENDING` CommandStatus |
| `Content-Location` | Terminal status item URI | Initial status item URI | Terminal status item URI | Initial status item URI |
| Public status sequence | exactly one of COMPLETED/REJECTED/FAILED | PENDING; then legal subset of all nine codes | exactly one of COMPLETED/REJECTED/FAILED | PENDING; ACCEPTED/EXECUTING as applicable; terminal COMPLETED/REJECTED/CANCELED/FAILED |
| SCHEDULED / UPDATED | not used in one-status response | available for Commands under exact meanings | unused | unused by Part 2 |
| Result | inline or linked in terminal status; endpoint when applicable | zero or more partial/final results | inline result normally; result endpoint always | empty until produced; then one or more results; endpoint always |
| Client monitoring | response, then standard status/result links | standard status/result endpoints and authorized event projection | response, then feasibility links | feasibility status/result endpoints and authorized event projection |
| Client disconnect | durable work continues/reconciles; same-key retry rejoins | no effect on durable resource | same | same |
| HTTP `202` | no | no merely because async | no | no merely because async |

### 5.2 Public transition rules

IDR-SRV-036's exact nine-code boundary remains controlling. For this topic:

- `PENDING` is committed only for asynchronous resources; it means received with no accept/reject decision.
- `ACCEPTED` does not promise completion. It means initial processing was accepted and can still end `REJECTED` or `FAILED`.
- `SCHEDULED` applies only to asynchronous Commands and includes required scheduled execution time.
- `UPDATED` applies only when a Command update is accepted; it is an event-like report, not a stable worker phase.
- `EXECUTING` may repeat with newer report times, progress, estimated end, and partial results.
- `REJECTED`, `CANCELED`, `COMPLETED`, and `FAILED` are terminal and receive no later public status.
- A late or conflicting callback after terminal is retained as restricted evidence/anomaly and cannot overwrite the public projection.

Asynchronous Feasibility normally follows `PENDING -> ACCEPTED -> EXECUTING -> COMPLETED`. It may instead transition from a nonterminal state to `REJECTED`, authorized `CANCELED`, or `FAILED`. `SCHEDULED` and `UPDATED` are rejected at the feasibility boundary even if a remote adapter sends them. Synchronous processing skips all public intermediate states. **[N/A/P]**

### 5.3 Private task phases

The private state machine may include `staged`, `ready`, `claimed`, `evaluating`, `gate_wait`, `dispatching`, `delivery_uncertain`, `reconciling`, `retry_wait`, `dead_letter`, and `completed`. These are operational facts, never serialized into `CommandStatus.statusCode`. A mapping function considers task kind, authoritative actor, certainty, public history, and finality; it is not a string conversion. **[A/P]**

### 5.4 Update, cancellation, timeout, and deletion

An update uses the same canonical Command/Feasibility ID, requires a strong precondition under the accepted Glaux profile, creates an immutable internal revision, reruns affected validation and decision gates, and is serialized against claim/dispatch/cancel/terminal commits. Only Commands can emit `UPDATED`; Feasibility update may replace/patch the request under the claimed class but the next feasibility status cannot use `UPDATED`. If the analysis already started, Glaux should cancel/supersede its private work and reevaluate the new revision without losing history. **[N/A/P]**

Cancellation is requested by POSTing a `CANCELED` status to the nested status endpoint by an authorized actor. The API service mediates the request; it does not let an ordinary requester forge terminal evidence. It commits public `CANCELED` only when the configured authority can guarantee the Part 2 meaning. Otherwise it retains a private cancellation request and waits for authoritative confirmation. IDR-SRV-038 decides who may cancel and when. **[N/A]**

Timeout is private. A known processing failure can become `FAILED`; a known decision not to execute can become `REJECTED`; mere lost contact cannot invent either. Ambiguous delivery/effect remains in private reconciliation with the last valid public status. Deletion invokes resource lifecycle/retention rules and never means cancel. **[A]**

---

## 6. Validation and Decision-Gate Boundaries

### 6.1 Responsibility matrix

| Stage | Primary owner | Inputs/checks | Failure before public creation | Failure after creation | Evidence/result |
|---|---|---|---|---|---|
| Transport/representation | HTTP boundary | method, path, media, size, decompression, syntax | 400/405/406/413/415 Problem Details | not applicable | request correlation and safe parse detail |
| Authentication/request permission | security boundary | credential, principal, endpoint/action, concealment | 401/403 or concealed 404 | revocation triggers later governed decision | protected decision hook; IDR-038/039 owns policy |
| Envelope/resource | command admission | required fields, IDs, times, read-only stripping, parent existence | 400/404/409 as mapped | REJECTED only when valid resource already exists | normalized request plus source reference |
| Parent schema/encoding | schema service | exact ControlStream version, record/choice/array, types, units, ranges, encoding | 400 required for invalid Command/Feasibility parameters | REJECTED if a later immutable dependency becomes incompatible | validation report and schema digest |
| Semantic/relationship | domain validator | definitions, target, Procedure, FOI, temporal window, geometry, reference integrity | 400 or 409 by stable taxonomy | REJECTED for changed current-state conflict | typed facts and rule codes |
| Stream availability | domain admission | `live`, version active, mode supported, deadline support | 409 for non-live/incompatible contract; 503 for transient server admission inability | REJECTED/FAILED according to accepted work outcome | availability snapshot |
| Feasibility | evaluator/adapter | operational constraints and versioned current context | only for a pre-create server policy that makes admission impossible | result decision; negative normally yields Command `REJECTED`, Feasibility `COMPLETED + NO` | result and full input manifest |
| Authorization/safety | policy/safety mediator | command authority, control ownership, approvals, interlocks, releasability | 403/concealed 404 where decision precedes creation | normally REJECTED with safe reason | protected decision evidence; IDR-038 owns rules |
| Receiving-system acceptance | target or authoritative local executor | target-specific first validation and willingness | unavailable for async after creation | ACCEPTED or REJECTED | signed/correlated status evidence |
| Dispatch/execution | worker + adapter + target | claim, fence, delivery, execution, result | not applicable | EXECUTING/COMPLETED/FAILED or private uncertainty | attempts, callbacks, status/results |

### 6.2 Ordering and side-effect rule

Cheap, non-oracular checks run before sensitive or expensive checks, but no caller learns whether a hidden target, schedule, or capability exists without authorization. All deterministic request/schema failures precede public resource creation when possible. Once a resource is created, a valid negative operational decision belongs in status/result rather than an HTTP error. No external dispatch or irreversible side effect occurs before durable intent, idempotency, policy-decision references, and recovery work are committed. **[A/P]**

For a synchronous request, the private staged record satisfies the durability boundary. If an error occurs before staging, return a Problem Details response and no resource. If processing reaches a terminal domain outcome, publish the standard resource and return `201` even when the terminal status is `REJECTED` or `FAILED`: HTTP creation succeeded; domain processing did not. If Glaux cannot stage work safely, return a retryable HTTP error with no physical effect. **[A/E/P]**

### 6.3 Liveness and dependency failures

`live=false` states that the ControlStream is not currently accepting Commands. Glaux should reject a new Command or Feasibility request before creation with stable `409 Conflict`, because the request conflicts with the exposed resource state. A transient overload or unavailable server admission dependency uses `503` and `Retry-After` where honest. A gateway becoming unreachable after asynchronous creation does not retroactively turn creation into an HTTP failure; the worker applies deadline/retry/indeterminacy rules. Exact concealment and error detail remain policy-controlled. **[N/E/P]**

---

## 7. Recommended Glaux Public API Behavior

### 7.1 POST response contract

For both Command and Feasibility POST:

1. Authenticate, authorize request access, decode, validate the exact parent contract, resolve idempotency, and perform configured pre-create checks.
2. Allocate a stable canonical ID and status ID.
3. For async, atomically commit the public resource, `PENDING` status, work/outbox, idempotency outcome, provenance, and audit reference.
4. For sync, atomically commit a private staged record and work/outbox, await a terminal outcome, then atomically publish the resource, exactly one terminal status, result(s), final idempotency response, and outbox evidence.
5. Return `201`, `Location: {canonical-resource}`, `Content-Location: {nested-status-item}`, appropriate `Content-Type`, validators for the retrievable representation where valid, and the CommandStatus body.

The body is status, not a representation of the Command/Feasibility named by `Location`; `Content-Location` removes that ambiguity under RFC 9110. Hypermedia in the status and parent resource leads to canonical identity, full status history, and result collection. The generated OAD must document this exact shape and examples. **[E/P]**

### 7.2 Retrieval and polling

Clients retrieve the canonical resource and follow standard nested links. `currentStatus` is a policy-filtered projection of the authoritative status history; status collection order is deterministic and paged. Async clients poll with validators/backoff or use the separately authorized publication projection from IDR-SRV-035. Events are hints: a client recovers truth from HTTP status/result resources and cursors. No event subscription is required to use tasking. **[A/P]**

The Feasibility result endpoint exists for every Feasibility resource. Before a result is produced it returns an empty collection, not 404. The Command result endpoint is exposed when the parent ControlStream/Command can produce results. Inline, Observation, observation-set, DataStream/time-range, and external references remain distinct result alternatives; each link is authorized independently. **[N/A/P]**

### 7.3 Idempotent retry and timeout recovery

The unsafe POST profile requires a scoped idempotency key. Same principal/tenant/operation/parent/key plus same canonical request digest returns the same ID and semantically equivalent response. Same key plus different digest returns the accepted conflict problem. While synchronous work is in progress, a retry rejoins the wait within its own request budget or returns a retryable problem that preserves the same key; it never creates a second effect. Once complete, the saved response points to the published resource. **[A/P]**

A client timeout is not evidence of failure. The client retries with the same key, then follows `Location`/resource links when returned. Glaux must not advise creating a new key unless the caller intentionally creates a new command intent and accepts the double-effect risk. **[A]**

### 7.4 HTTP `202` and future deferred mutation

The first profile does not advertise deferred HTTP mutation, so it does not return `202`. If later enabled, `202` means the HTTP create/replace/update/delete operation itself is incomplete. Its operation monitor is distinct from CommandStatus and must be designed/documented before use. `ControlStream.async`, a queued dispatcher, slow actuation, or a broker hop alone never triggers `202`. This deliberately resolves the earlier broad “accepted asynchronous work” language in IDR-SRV-029 in favor of the more precise IDR-SRV-013 distinction. **[A/E/P]**

### 7.5 Mutation and writer authority

When CRD/UPDATE classes are claimed, Glaux implements the standard methods and paths but authorizes writers by role. Ordinary command requesters may submit and request authorized update/cancellation; trusted API/worker/gateway/receiving-system principals append or amend status/result evidence under registered authority. Public replacement/update changes the current standards representation with ETag/If-Match checks while internal revisions remain immutable. Deletion follows retention, holds, relationships, and tombstones. **[N/A/P]**

### 7.6 Limits and overload contract

The capability registry must hold per-profile limits for request/result bytes, decompressed size, record/array depth, task deadline, queue depth, per-principal and per-System concurrency, retry budget, callback/status rate, status/result count, and retention. Exact numbers require later performance/deployment evidence. Payload excess returns `413`; principal rate limits use `429`; temporary service/queue saturation uses `503` with truthful `Retry-After`. Material statuses/results are never silently dropped; redundant progress may be coalesced internally only when the externally required history remains truthful. **[A/P]**

---

## 8. Recommended Rust Domain and Component Architecture

### 8.1 Component boundary

```text
HTTP handler -> admission/application service -> PostgreSQL transaction
                         |                         |
                         |                         +-- resources/status/results
                         |                         +-- idempotency + task/outbox
                         v
                 task coordinator/worker -> feasibility or command port
                                                 |
                                    local evaluator / gateway adapter
                                                 |
                                authenticated callback/reconciliation
```

Handlers translate HTTP and never own lifecycle transitions. The application service owns admission and unit-of-work orchestration. Domain transition functions accept typed current state, actor authority, task kind, revision, event time/sequence, and proposed evidence; they return an append decision or typed rejection. Repositories persist but do not decide semantics. Adapters translate registered device/gateway protocols and cannot mint standard status merely from transport acknowledgements. **[A/E/P]**

### 8.2 Domain types and invariants

Use exhaustive enums and newtypes such as `TaskKind::{Command, Feasibility}`, `ProcessingMode::{Synchronous, Asynchronous}`, `PublicStatusCode`, `PrivateTaskPhase`, `FeasibilityDecision`, `TaskId`, `ResourceId`, `ControlStreamVersion`, `ResourceRevision`, `AttemptId`, `FenceToken`, `IdempotencyKey`, `RequestDigest`, and `ContextFingerprint`.

The transition API should make these invalid cases explicit errors: `SCHEDULED`/`UPDATED` for Feasibility; nonterminal status for sync publication; status after terminal; wrong parent/revision; untrusted reporter; `COMPLETED` without required actual execution time; ordinary result validated against feasibility schema or vice versa; decreasing same-attempt progress without correction semantics; and result ownership mismatch. Compile-time types reduce accidental mixing, while database constraints and transactional checks remain authoritative against concurrency. **[N/E/P]**

Conceptual ports are:

```rust
trait FeasibilityAdapter {
    async fn evaluate(&self, task: FeasibilityEnvelope) -> EvaluationOutcome;
    async fn reconcile(&self, remote: RemoteTaskRef) -> ReconciliationOutcome;
    async fn cancel(&self, remote: RemoteTaskRef) -> CancellationOutcome;
}

trait CommandAdapter {
    async fn dispatch(&self, task: CommandEnvelope) -> DispatchOutcome;
    async fn reconcile(&self, remote: RemoteTaskRef) -> ReconciliationOutcome;
    async fn cancel(&self, remote: RemoteTaskRef) -> CancellationOutcome;
}
```

These are design sketches, not final Rust signatures or crate choices. Return types preserve accepted, rejected, retryable, permanent failure, and uncertain outcomes rather than collapsing them into strings or generic errors. **[E/P]**

### 8.3 Architecture option table

| Option | Durability/recovery | Operational cost | Fit now | Adoption or rejection trigger |
|---|---|---:|---|---|
| Handler-spawned future only | Lost/ambiguous on process failure; no durable claim | Low | Reject | Never sufficient for authoritative tasking |
| In-process worker + PostgreSQL durable work | Restart-safe; transactionally joins resources/outbox; horizontally claimable | Low-moderate | **Recommended first** | Retain while measured latency/throughput/isolation targets are met |
| Independently deployed worker using same DB | Same durable model; better process/fault/resource isolation | Moderate | Prepared option | Adopt for independent scaling, credential isolation, blocking drivers, or API blast-radius reduction |
| External broker + DB authority/outbox | Good delivery integration; duplicates/ordering/partition complexity remain | Moderate-high | Conditional adapter | Adopt for remote topology, measured polling pressure, cross-service fanout, or required gateway protocol |
| General workflow engine | Durable multi-step orchestration but adds new authority/model | High | Reject/defer | Reconsider only for proven long-running multi-party/human workflows beyond this task model |

### 8.4 Why the first option is sufficient

The accepted architecture already makes PostgreSQL authoritative and requires atomic outbox/idempotency. A queue-like task table provides work identity, readiness, leases, retries, and recovery without a second consistency system. Multiple workers can claim independent rows with `FOR UPDATE SKIP LOCKED`; a short transaction marks the lease and returns the payload, then external work occurs after commit. Tokio efficiently waits on I/O and coordinates graceful shutdown, but correctness resides in persisted state, leases, fencing, and idempotent adapters—not in a future remaining alive.[^4][^5] **[A/E/P]**

---

## 9. Persistence, Concurrency, Idempotency, and Recovery

### 9.1 Transaction boundaries

Asynchronous admission transaction:

- insert canonical Command/Feasibility and immutable request revision;
- insert initial `PENDING` CommandStatus and current projection;
- insert scoped idempotency key/digest and replayable response metadata;
- insert task row and dispatch/publication outbox rows;
- bind provenance, policy/safety decision references, and audit correlation; and
- commit before any gateway/evaluator call or publication.

Synchronous staging transaction creates the private admission/work, idempotency, outbox, and evidence records. Its terminal publication transaction creates the public resource, sole terminal status, result(s), projection, final response record, and publication outbox atomically. This avoids holding a database transaction across evaluation, network I/O, or physical execution. **[A/E/P]**

### 9.2 Durable task record

At minimum persist task/resource identity, kind, mode, parent/version/revision, request/contract digests, work phase, priority class, not-before/deadline, attempt counter, lease owner/until, fence token, adapter and remote correlation, retry class/backoff, cancellation flag, last error class, result/evidence references, created/updated times, and terminal disposition. Payload bytes remain in the accepted exact-source/artifact store rather than being duplicated without bounds. **[A/P]**

### 9.3 Claim, lease, and fence protocol

1. In a short transaction, select ready nonterminal rows in deterministic priority/order with `FOR UPDATE SKIP LOCKED`.
2. Set `claimed`, a unique attempt ID, incremented fence token, lease owner/expiry, and heartbeat deadline; commit and return claimed rows.
3. Process outside the transaction with bounded concurrency and cancellation awareness.
4. Heartbeat by compare-and-swap on task/attempt/fence while work is legitimately active.
5. Append an outcome only if the claim/revision/fence and public transition are still current; commit outcome, projection, next task state, audit, and outbox together.
6. An expired lease is eligible for recovery, not proof that the prior worker did nothing. Reconcile remote state before any possibly duplicating side effect.

`SKIP LOCKED` is correct for distributing queue work, not for reading authoritative client state; its intentionally inconsistent view is unacceptable for ordinary resource queries. Per-System concurrency and serial command invariants use database constraints, CAS, row/transaction locks, or targeted advisory locks as established by IDR-SRV-029. **[A/E/P]**

### 9.4 Retry and dead-letter semantics

Classify errors as local transient, remote transient, rate-limited, permanent request/contract, permanent adapter, authorization/safety change, or uncertain side effect. Apply bounded exponential backoff with jitter and deadline. Retrying evaluation is safe only with the same input manifest or an explicit new evaluation revision. Retrying dispatch uses the same Command/revision/attempt correlation and requires an idempotent target operation or authoritative reconciliation. **[A/P]**

“Dead letter” is a private intervention state, not a public `FAILED` synonym. The mapper emits `FAILED` only when processing/execution failure is authoritative, `REJECTED` only when non-processing/non-execution is authoritative, or retains the prior public status when outcome is uncertain. Operator repair creates evidence and a new private attempt; it never edits history invisibly. **[A/P]**

### 9.5 Races

| Race | Serialization rule | Public outcome |
|---|---|---|
| duplicate submit | unique scoped key plus digest | same resource/response; conflict on different digest |
| two workers claim | row lock + attempt/fence CAS | one authoritative attempt; duplicate evidence quarantined |
| update vs claim | resource revision CAS; worker validates revision before effect | accepted update yields `UPDATED` only for Command; stale worker cannot commit |
| cancel vs dispatch | cancellation intent and dispatch claim serialized; adapter contract determines certainty | `CANCELED` only with authority; otherwise reconcile |
| terminal vs terminal | first causally valid terminal commit wins | later conflict is restricted evidence/incident |
| lease expiry vs late callback | remote correlation + attempt/fence + transition validation | valid causal callback may resolve; cannot overwrite terminal blindly |
| cache reuse vs state change | context fingerprint/dependency version check in admission transaction | recompute or new result; no stale YES reuse |

### 9.6 Restart and graceful shutdown

On startup, workers recover expired/abandoned claims, verify task/resource/revision state, and reconcile ambiguous external attempts before retry. Non-expired leases are not stolen merely because a new instance starts. Graceful shutdown stops new claims, signals in-flight adapters, continues heartbeats during the grace window, commits completed outcomes, and releases or lets leases expire safely. Tokio cancellation and task tracking can coordinate that process, but a forced stop remains safe because the database is authoritative.[^8] **[E/P]**

---

## 10. Gateway, DDIL, Security, and Information-Exposure Boundaries

### 10.1 Gateway trust and correlation

Every adapter registration binds a gateway identity, represented System scope, allowed operations/statuses/result forms, protocol/version, idempotency/correlation method, acknowledgement meaning, clock model, credential reference, and reconciliation/cancellation capability. A broker or gateway transport ACK proves delivery only. It maps to `ACCEPTED` only when the authenticated receiving-system contract explicitly gives that acknowledgement the Part 2 meaning. **[A/P]**

Callbacks pass through the common write boundary: authenticate principal, resolve adapter/target/remote correlation, validate schema and authority, preserve raw evidence, check revision/fence/sequence/time/transition, append idempotently, update projections, and publish after commit. Out-of-order or duplicate evidence is normal input, not permission for arrival-wins state. **[A]**

### 10.2 DDIL and stale knowledge

Disconnected operation makes freshness visible. A feasibility evaluator records last-contact time, source event time, ingest time, remote state token, clock uncertainty, and missing dependencies. It may return `INDETERMINATE` or a result with reduced confidence/validity only if the advertised schema expresses that honestly. It may not convert “gateway unreachable” into “NO” unless the operational definition itself says connectivity is a feasibility constraint. **[A/E/P]**

An async Command may remain queued for a disconnected gateway only if its execution/validity/policy horizon permits it. Before reconnect dispatch, Glaux rechecks current revision, feasibility predicates, authorization/safety hooks, deadline, target epoch, and conflicting actions. Delayed callbacks reconcile by identity/sequence and cannot resurrect canceled/terminal work. Final fork/rejoin and cross-node conflict rules remain IDR-SRV-042/043 work. **[A/P]**

### 10.3 Sensitive feasibility information

Feasibility can reveal existence, capabilities, readiness, schedules, competing users, resource scarcity, target location, planned activity, gateway health, rule thresholds, and potential alternatives. Public messages should use stable bounded codes and safe parameter locations; raw schedule conflicts, exclusive controller identity, safety rules, topology, model internals, and protected alternatives belong in authorized result fields or audit evidence. Status/result reads and event publication repeat authorization at retrieval/delivery time. **[E/P]**

This topic establishes hook locations but not policy. IDR-SRV-038 decides command-specific grants, control ownership, safety/interlock and approval rules, cancellation authority, fail-open/closed behavior, and command-audit contents. IDR-SRV-039/039A/040 decide authentication, zero-trust enforcement, releasability, and cross-boundary disclosure. **[A]**

### 10.4 Audit/provenance hooks retained

Retain correlation for request and response, principal and represented actor, idempotency decision, source/canonical digests, parent/version, validation rules and outcomes, feasibility manifest/result/redaction, authorization/safety decision references, task claims/attempts/leases, adapter credentials by reference, dispatch and acknowledgements, callbacks, status/result transitions, retries/timeouts/cancellation, manual intervention, cache reuse/invalidation, and publication events. This is a required seam, not the final audit schema or retention policy. **[A/P]**

---

## 11. Conformance and Test Strategy

### 11.1 Test layers

| Layer | Required proof |
|---|---|
| Domain unit/property | exhaustive allowed/forbidden transitions by task kind/mode; terminality; status time/progress/result invariants; freshness dependency mutation |
| Schema/encoding | JSON and each claimed SWE encoding; records/choices/arrays, units, nils, times, geometries, constraints; ordinary vs feasibility result schema |
| Repository/transaction | atomic admission/publication, idempotency race, CAS/fence, lease recovery, outbox, rollback, status/result ownership, ETag preconditions |
| Adapter contract | ACK meaning, stable correlation, duplicate/out-of-order callback, retry classification, cancel/reconcile, clock and stale-state behavior |
| HTTP integration | exact plural paths, 201/Location/Content-Location/body, canonical/nested navigation, filters, errors, empty results, retry/rejoin |
| Fault/restart | crash before/after stage, claim, dispatch, ACK, callback, terminal commit, response, publication; gateway partition and late callback |
| Security | concealed targets, cross-principal key collision, status forgery, result leakage, policy revocation, redaction, abusive queue/status/result rates |
| Interoperability | generic clients, official ATS, supplemental defect tests, OS4CSAPI response variants, CS-Go/OSH behavior comparison |
| Performance | queue claim contention, per-System serialization, sync deadline, worker saturation, large result, status storm, restart backlog |

### 11.2 Executable scenario matrix

| ID | Scenario | Expected evidence and assertion | Requirement/owner |
|---|---|---|---|
| T01 | async valid Command POST | atomic Command + PENDING + task/outbox; `201`; both location headers; no precommit dispatch | 26-32, 71-72; 029/036/037 |
| T02 | sync valid Command completes | no public intermediate status; one COMPLETED with actual execution time/result; `201` | §10.11-12; 037 |
| T03 | sync valid Command rejected | created Command and sole REJECTED status; HTTP still `201` | §10.11; 013/037/038 |
| T04 | invalid parameters every claimed encoding | `400`; no resource/task/outbox/effect | 25, 72, 86, 102/114/etc.; 023 |
| T05 | async lifecycle | PENDING→ACCEPTED→SCHEDULED→EXECUTING(progress/partial)→COMPLETED; required times | §10.11; 036/037 |
| T06 | async late rejection/failure | ACCEPTED may lead REJECTED or FAILED under correct meaning | §10.11; 036/037 |
| T07 | sync Feasibility YES | one COMPLETED plus inline schema-valid YES result; canonical/result links | 35-38, 75-77, 105; 037 |
| T08 | sync Feasibility NO | one COMPLETED plus NO; never REJECTED merely because infeasible | §11; 037 |
| T09 | async Feasibility indeterminate | PENDING→ACCEPTED→EXECUTING→COMPLETED + INDETERMINATE | §11; 037 |
| T10 | invalid feasibility status | reject SCHEDULED/UPDATED without altering projection | §11 status table; 037 |
| T11 | empty feasibility result collection | `200` empty collection before result, then result visible and valid | 38; 037/050 |
| T12 | same/different idempotency replay | same digest returns same ID/outcome; different digest conflicts; one effect | 029/031/037 |
| T13 | client disconnect/lost response | work survives; same-key retry rejoins/recovers canonical outcome | 029/037 |
| T14 | crash window matrix | no dispatch before commit; expired lease reconciles; no double effect | 029/036/037 |
| T15 | cancellation races | request vs claim/dispatch/completion; CANCELED only when authoritative | 73/76; 036-038 |
| T16 | update races | strong precondition, new revision, stale worker fenced; no UPDATED for feasibility | 85-91; 029/037 |
| T17 | stale cached YES | mutate every dependency; recompute/reject; no reservation behavior | 018/029/037 |
| T18 | gateway unavailable variants | pre-admission 503, accepted FAILED, or COMPLETED+INDETERMINATE according to exact case | 013/037/042 |
| T19 | callback disorder/duplicate | causal projection stable; duplicate idempotent; terminal conflict quarantined | 029/034/036/037 |
| T20 | result forms | inline, Observation(s), collection, DataStream/time range, external; policy and referential checks | §10.12/Req. 33-34, 74/88, 104-105 |
| T21 | exact path graph | plural root/nested routes; canonical links; singular defect paths not silently primary | 26-38, 71-77 |
| T22 | filters | issue/execution/status/sender/FOI/statusCode individually and combined under stable paging | 56-61; 011 |
| T23 | status/result writer roles | ordinary requester cannot forge evidence; authorized adapter can append within scope | 73-77; 038/039 |
| T24 | overload/limits | 413/429/503 and Retry-After; no partial commit or silent status loss | 013/031/037/054 |

### 11.3 Official and supplemental harness rules

Every normative record stores document/version, conformance class, requirement URI, official ATS URI, request fixture, expected response/state, and supplemental rationale. Official ATS results remain separately reportable. Supplemental tests cover the singular/plural defects, missing `{id}`, copied `/commands` feasibility ATS, A.105 feasibility branch omission, UTC checks, exact sync/async response profile, transition legality, and non-normative safety/recovery properties. Glaux must not modify an official test silently and call it official. **[A/P]**

### 11.4 Interoperability disposition

CS-Go `v1.0.4` demonstrates persistent Command/status/result resources, plural paths, `201 + Location`, initial `currentStatus=PENDING`, and MQTT status handling, but no feasibility route and no complete transition/recovery model were found. OSH `v2.0.2` demonstrates separated command/status/result stores and synchronous/timeout infrastructure, but the accepted study likewise found no feasibility implementation evidence. OS4CSAPI observed both `201` and `202`, immediate and async responses, and client parsing failures on empty bodies. Glaux should therefore publish one exact response contract, tolerate conforming representation choices only in clients, and use these implementations as comparative fixtures rather than behavioral authority. **[I/E/P]**

---

## 12. Phased Implementation Recommendation

### Phase 0 — Contract and executable model

- Freeze path, response, schema, media, status/result, error, and capability-registry records.
- Implement pure transition and feasibility-freshness models with exhaustive/property tests.
- Add official requirement/ATS ledger and supplemental defect tests.
- Define adapter conformance contract and safe fake evaluator/executor.

Exit: no handler or repository can invent a state/path/response outside the model.

### Phase 1 — Local synchronous Feasibility

- Implement exact JSON first and the common validation pipeline.
- Add private durable sync staging, local deterministic evaluator, one terminal status, inline result schema, idempotent rejoin, and canonical/nested reads.
- Limit enablement to bounded evaluators and explicitly configured ControlStreams.

Exit: T07-T13 and applicable ATS pass across restart/fault injection.

### Phase 2 — Asynchronous Feasibility

- Add public PENDING creation, PostgreSQL worker claims/leases/fences, progress, cancellation, cache/freshness/invalidation, gateway adapter, and result polling/publication.
- Add overload controls and DDIL/stale-evidence fixtures.

Exit: T09-T19/T24 pass and status/result history remains causal under concurrency.

### Phase 3 — Command execution modes

- Reuse the same durable kernel for synchronous and asynchronous Command execution.
- Enable only after IDR-SRV-038 authorization/safety/audit decisions are accepted and implemented.
- Add schedule/update/partial-result/dispatch/reconciliation paths and physical-effect harnesses.

Exit: all Command scenarios pass with no precommit or duplicate effect.

### Phase 4 — Scale and interoperability gates

- Benchmark queue claims, status volume, sync deadlines, per-System fairness, result size, restart backlog, and gateway partitions.
- Separate worker process if isolation/scaling/credential evidence warrants.
- Add broker adapter only if topology/integration/throughput evidence warrants; keep DB authoritative.
- Run official ATS, supplemental suite, OS4CSAPI, and independent clients; publish exact conformance/OAD claims.

Exit: release evidence justifies each enabled mode, adapter, encoding, and conformance class.

---

## 13. Risks, Deferred Decisions, and Open Questions

### 13.1 Principal risks and mitigations

| Risk | Consequence | Mitigation / owner |
|---|---|---|
| Treat `async` as HTTP 202 or Rust async | ambiguous tracking and lost work | fixed 201 profile; durable-state tests; 037 |
| Treat feasibility YES as reservation | unsafe oversubscription/race | explicit evidence/expiry/recheck; future reservation needs profile authority |
| Return REJECTED for infeasible analysis | clients cannot distinguish analysis outcome | COMPLETED + result-decision tests; 037 |
| Expose sensitive constraints/schedules | operational intelligence leak | authorization, redaction, stable safe codes; 038-040 |
| Inline sync dispatch before commit | effect without recoverable identity | private staging and postcommit worker; 029/037 |
| Blind retry after lost ACK | duplicate physical effect | stable correlation, fence, idempotent adapter, reconciliation; 036-038 |
| DB queue contention/starvation | latency and unfairness | deterministic claim order, bounded batches, per-System limits, benchmarks; 045/054 |
| Broker becomes authority | split-brain task/status state | outbox/inbox and DB authority invariant; 035/045 |
| Official ATS defects hide gaps | false conformance claim | separate supplemental ledger; 050/051 |
| Mutable draft dependency changes | response/profile drift | pin, compatibility seam, refresh at implementation/final synthesis; 010A/057 |

### 13.2 Deferred decisions and owners

- Exact authorization grants, control ownership/transfer, safety/interlocks, approval chains, cancellation authority, audit fail-open/closed behavior, and diagnostic disclosure: **IDR-SRV-038**.
- Authentication mechanisms, zero-trust service identity, releasability/cross-boundary policy, and general audit schema: **IDR-SRV-039/039A/040/041**.
- Offline queue permission, reconciliation across stores, target epochs, branch/fork/rejoin, and conflict resolution: **IDR-SRV-042/043**.
- Final Rust web/async/database libraries, process decomposition, deployment, configuration, observability, migrations/backups: **IDR-SRV-044-049**.
- Exact thresholds/SLOs, harness packaging, fixtures, performance, security, and client certification: **IDR-SRV-050-056**.
- Current upstream and draft refresh, including issues #1/#58/#59 and Features Part 4: **IDR-SRV-057 and implementation release gate**.

### 13.3 Remaining open questions

1. Which operational ControlStreams can truthfully meet the synchronous deadline and recovery contract? This is configuration/integration evidence, not a global assumption.
2. Which AEP-approved mission profiles require strictly binary feasibility, and which permit conditional/indeterminate results? The advertised schema decides per ControlStream.
3. Which target protocols provide stable idempotency/correlation and authoritative query-after-timeout? Adapters lacking them may be feasibility-only or require manual reconciliation.
4. Does a future approved standard/profile define reservation and confirmation? Until then Glaux exposes none.
5. Will Features Part 4's final publication change deferred-mutation, representation-return, or PATCH details? Pin and refresh before implementation claims.

None blocks this baseline. Each has an explicit conservative default and owner.

---

## 14. Traceability and References

### 14.1 Research-question coverage

| Plan group | Coverage |
|---|---|
| A. Normative/profile baseline | §§2-3, 11.3-11.4, Appendix B |
| B. Public resource/API contract | §§5, 7, 9.1; Appendix A |
| C. Task state model | §5, §9.4-9.6; Appendix A |
| D. Validation/feasibility pipeline | §§4, 6, 10 |
| E. Durable Rust implementation | §§8-9, 12 |
| F. Security/DDIL/exposure | §10, §13.2 |
| G. Conformance/interoperability | §11, Appendix B |

### 14.2 Controlling standards and drafts

1. Open Geospatial Consortium, [OGC 23-002, *OGC API - Connected Systems - Part 2: Dynamic Data*, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html), approved June 2 and published July 16, 2025; immutable repository tag [`v1.0.0`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api/part2).
2. Open Geospatial Consortium, [OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html).
3. Open Geospatial Consortium, [OGC 23-000, *SensorML Encoding Standard*, Version 3.0](https://docs.ogc.org/is/23-000/23-000.html).
4. Open Geospatial Consortium, [OGC 24-014, *SWE Common Data Model Encoding Standard*, Version 3.0](https://docs.ogc.org/is/24-014/24-014.html).
5. Open Geospatial Consortium, [draft OGC API - Features - Part 4 source](https://github.com/opengeospatial/ogcapi-features/tree/4e30324a14b682ff4a26ee43aad1eb6428c846a3/extensions/transactions/create-replace-update-delete), `20-002r2`, `1.0.0-SNAPSHOT`, checked September 15, 2026.
6. IETF, [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html), June 2022.
7. IETF, [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html), July 2023.

### 14.3 Engineering sources

8. PostgreSQL Global Development Group, [PostgreSQL 18 `SELECT` locking clause](https://www.postgresql.org/docs/18/sql-select.html) and [Concurrency Control](https://www.postgresql.org/docs/18/mvcc.html), checked September 15, 2026.
9. Rust Project, [Rust Reference: async functions](https://doc.rust-lang.org/stable/reference/items/functions.html#async-functions), checked September 15, 2026.
10. Tokio Project, [Graceful Shutdown](https://tokio.rs/tokio/topics/shutdown), checked September 15, 2026.

### 14.4 Official history and implementation evidence

11. OGC Connected Systems issues [#1 Cross-domain Action/Task Definition](https://github.com/opengeospatial/ogcapi-connected-systems/issues/1), [#58 transfer of control](https://github.com/opengeospatial/ogcapi-connected-systems/issues/58), and [#59 task reservation/confirmation](https://github.com/opengeospatial/ogcapi-connected-systems/issues/59), states checked September 15, 2026.
12. [CS-Go `v1.0.4`](https://github.com/SomethingCreativeStudios/connected-systems-go/tree/v1.0.4), [OSH `v2.0.2`](https://github.com/opensensorhub/osh-core/tree/v2.0.2), accepted OS4CSAPI phase-9 evidence, and accepted SECD evidence at the report-header pins.

### 14.5 Project sources

13. [Accepted IDR-SRV-007 Part 2 requirement baseline](idr-srv-007-csapi-part-2-requirement-baseline-report.md), [IDR-SRV-008 conformance mapping](idr-srv-008-conformance-class-and-requirement-mapping-report.md), and [IDR-SRV-013 error strategy](idr-srv-013-error-model-http-status-codes-and-failure-semantics-report.md).
14. [Accepted IDR-SRV-015 through IDR-SRV-020](./) for resource, identity/lifecycle, relationships, temporal/freshness, provenance, and status/event foundations.
15. [Accepted IDR-SRV-022 SWE strategy](idr-srv-022-swe-common-data-component-strategy-report.md), [IDR-SRV-023 validation strategy](idr-srv-023-schema-and-encoding-validation-strategy-report.md), and [IDR-SRV-024 semantics/unit strategy](idr-srv-024-units-observed-properties-and-semantic-binding-strategy-report.md).
16. [Accepted IDR-SRV-029 transaction strategy](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md), [IDR-SRV-031 write model](idr-srv-031-server-write-and-ingestion-model-report.md), [IDR-SRV-032 publisher boundary](idr-srv-032-publisher-to-server-contract-boundary-report.md), and [IDR-SRV-033 simulator boundary](idr-srv-033-simulator-to-server-contract-boundary-report.md).
17. [Accepted IDR-SRV-034 dynamic semantics](idr-srv-034-datastream-observation-and-status-update-semantics-report.md), [IDR-SRV-035 streaming/publication strategy](idr-srv-035-streaming-and-event-publication-strategy-report.md), and [IDR-SRV-036 command lifecycle](idr-srv-036-control-stream-and-command-lifecycle-model-report.md).
18. [OGC Connected Systems upstream-history register, Version 1.12](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md).
19. Controlled AEP baseline `AC/224(JCGISR)D(2026)0005`, April 27, 2026, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`; available only through authorized project custody and accepted releasable findings; not redistributed.

### 14.6 Numbered source notes

[^1]: OGC 23-002 §10.2 makes `async` a required ControlStream Boolean indicating asynchronous command processing; §§10.11 and 11 define the resulting status patterns. It does not prescribe a queue, worker, broker, Rust construct, or HTTP `202`.
[^2]: RFC 9110 §§15.3.2-15.3.3 says `201` identifies created resources, normally through `Location`, while `202` means request processing itself has not completed and is intentionally noncommittal. §8.7 permits `Content-Location` to identify enclosed action-status content.
[^3]: OGC 23-002 §11 describes feasibility as advance analysis using the same parameters, with binary and detailed information. Open issue #59 remained an unadopted reservation/confirmation proposal on September 15, 2026.
[^4]: PostgreSQL 18 `SELECT` documentation warns that `SKIP LOCKED` gives an inconsistent view unsuitable for general queries but can avoid contention among consumers of a queue-like table.
[^5]: The Rust Reference says calling an `async fn` returns a future and the function body runs when that future is polled. Persistence, leasing, idempotency, restart recovery, and external-effect certainty are outside that language semantic.
[^6]: Features Part 4 draft ATS at commit `4e30324a...` accepts `201`, or `202` when creation itself is queued, and says the draft specifies no mechanism for determining queued-operation outcome. It remains draft evidence.
[^7]: OGC 23-002 §10.11 defines the nine codes and exact synchronous/asynchronous, execution-time, progress, and terminal-state meanings. §11 redefines their applicability for Feasibility and makes `SCHEDULED`/`UPDATED` unused.
[^8]: Tokio's official graceful-shutdown guidance separates detecting shutdown, signaling tasks, and waiting for completion. These mechanisms help orderly operation but do not replace durable recovery.

---

## Appendix A. Detailed Lifecycle and Ownership Matrix

| Task/mode | Private phase | Public status emitted | Authoritative initiator/reporter | Required evidence | Retry/cancel/recovery rule |
|---|---|---|---|---|---|
| any | received/validating | none | API boundary | request/principal/digest/contract | no effect before durable stage |
| async any | admitted | PENDING | Glaux admission | atomic resource/status/task/idempotency/outbox | same-key retry returns same resource |
| sync any | staged | none | Glaux admission | private durable work/idempotency | retry rejoins; excluded from CSAPI queries |
| feasibility | accepted | ACCEPTED (async only) | configured evaluator/gateway | initial validation/acceptance correlation | may later reject/fail |
| feasibility | evaluating | EXECUTING (async only) | evaluator/gateway | manifest, attempt/fence, progress/time | repeated progress allowed |
| feasibility | analyzed yes/no/conditional/indeterminate | COMPLETED | evaluator through Glaux validator | actual analysis time and schema-valid result | terminal; cache under validity rules |
| feasibility | not processed | REJECTED | receiving evaluator/authorized mediator | certain non-processing reason | terminal; no result required |
| feasibility | technical processing failure | FAILED | evaluator/worker | failure evidence and actual processing time as applicable | terminal; new request to retry intent |
| command | accepted | ACCEPTED (async only) | receiving System or authoritative local executor | target acknowledgement semantics | not a completion promise |
| command | scheduled | SCHEDULED (async only) | receiving System/scheduler | scheduled execution time | update/cancel before governed cutoff |
| command | revision accepted | UPDATED (async only) | receiving System/mediator | new revision and rerun gate evidence | old worker fenced |
| command | executing | EXECUTING (async only) | receiving System/executor | execution/progress/partial-result evidence | repeat; reconcile on lost contact |
| command | successful effect | COMPLETED | receiving System/executor | actual execution time and valid result | terminal; observations may corroborate world state |
| command | certain nonexecution | REJECTED | receiving System/authorized mediator | reason and certainty | terminal |
| any | user cancellation authoritative | CANCELED | cancellation mediator | actor, authority, race disposition | terminal; DELETE remains distinct |
| any | uncertain delivery/effect | unchanged | task coordinator | attempts, ACK/callback, time/epoch | reconcile; never invent public UNKNOWN |
| any | worker retry/dead letter | unchanged unless authoritative mapping | task coordinator/operator | error class, attempt/fence, intervention | bounded retry or governed repair |

## Appendix B. Reproducibility Record

### B.1 Immutable source checks

```powershell
git ls-remote https://github.com/opengeospatial/ogcapi-connected-systems.git refs/tags/v1.0.0 refs/heads/master
git ls-remote https://github.com/opengeospatial/ogcapi-features.git refs/heads/master
git ls-remote https://github.com/SomethingCreativeStudios/connected-systems-go.git refs/tags/v1.0.4
git show v1.0.0:api/part2/standard/sections/clause_9_requirements_class_controlstreams.adoc
git show v1.0.0:api/part2/standard/sections/clause_10_requirements_class_command_feasibility.adoc
git show v1.0.0:api/part2/standard/sections/clause_15_requirements_class_create_replace_delete.adoc
git show v1.0.0:api/part2/standard/sections/clause_16_requirements_class_update.adoc
git show v1.0.0:api/part2/standard/sections/clause_20_requirements_class_json_encoding.adoc
```

Observed September 15, 2026:

```text
CSAPI Parts 1/2 v1.0.0   8e03b236a049849f2ccc24b4fd9fdce5ff69bed2
CSAPI master              3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f
Features Part 4 source    4e30324a14b682ff4a26ee43aad1eb6428c846a3
CS-Go v1.0.4              244f4dd586da685d4d9b75e43f73001028b5bd0e
OSH v2.0.2                235c0eabf24b6d6137b499b4402943d2794b70e6
OS4CSAPI phase-9          754411897173c2ec4debaa9bcf4ed9e0f8a9e230
SECD evidence             f018fd129bf0d0d1ce75e68198e3ab4d99d937a0
```

### B.2 Findings reproduced

- Part 2 source defines sync one-status and async multi-status behavior and the exact code tables.
- Feasibility reuses Command/CommandStatus/CommandResult, the parameter schema, and distinct `feasibilityResultSchema`.
- Normative prose contains the documented singular/plural and missing-ID defects; ATS contains the copied feasibility path and A.105 omission.
- Current Features Part 4 draft differentiates immediate `201` from queued HTTP-operation `202` and supplies no universal operation monitor.
- Issues #1/#58/#59 remained open with no linked development.
- CS-Go `v1.0.4` contains Command/status/result routes and models but no feasibility route; OSH/OS4CSAPI/SECD conclusions are consumed from accepted pinned studies.
- PostgreSQL documents `SKIP LOCKED` as a queue-consumer mechanism; Rust documents async functions as futures rather than durable jobs.

## Appendix C. Requirement-by-Requirement Disposition

This compact ledger makes the grouped table in §3.2 mechanically complete. “Official + supplemental” means Glaux runs the published Annex A test unchanged and separately runs the corrective/project-profile test identified in §§3.3 and 11.

| Req. | Requirement URI | Concrete Glaux disposition | Verification |
|---:|---|---|---|
| 25 | `/req/controlstream/schema-op` | serve exact schema for each supported command format/version | official ATS + format negatives |
| 26 | `/req/controlstream/cmd-canonical-url` | canonical Command is `/commands/{id}`; alternate retrieval links canonical | official ATS + link graph |
| 27 | `/req/controlstream/cmd-resources-endpoint` | expose typed, paged Command collections | official ATS + schema/ownership test |
| 28 | `/req/controlstream/cmd-canonical-endpoint` | expose authorized Commands at `/commands` | official ATS + policy paging |
| 29 | `/req/controlstream/cmd-ref-from-controlstream` | use plural `/controlstreams/{csId}/commands`; only children of parent | official + supplemental plural-path test |
| 30 | `/req/controlstream/cmd-collections` | apply common collection representation rules | official ATS |
| 31 | `/req/controlstream/status-resources-endpoint` | typed, paged status collection with required query behavior | official ATS + ordering test |
| 32 | `/req/controlstream/command-status-endpoint` | use plural `/commands/{cmdId}/status`; only related statuses | official + supplemental singular-defect test |
| 33 | `/req/controlstream/result-resources-endpoint` | typed, paged result collection | official ATS + result alternatives |
| 34 | `/req/controlstream/command-result-endpoint` | use plural `/commands/{cmdId}/result` when result-capable; enforce ownership | official + supplemental singular-defect test |
| 35 | `/req/feasibility/canonical-url` | canonical Feasibility is `/feasibility/{id}` | official ATS + corrected resource assertions |
| 36 | `/req/feasibility/ref-from-controlstream` | use `/controlstreams/{csId}/feasibility`; only children of parent | official + corrected feasibility/plural test |
| 37 | `/req/feasibility/status-endpoint` | always expose `/feasibility/{feasId}/status` | official ATS + lifecycle fixtures |
| 38 | `/req/feasibility/result-endpoint` | always expose `/feasibility/{feasId}/result`, empty until populated | official ATS + empty/terminal tests |
| 39 | `/req/feasibility/collections` | do not expose optional custom collections initially; apply common rules if later enabled | conformance declaration + route test |
| 56 | `/req/advanced-filtering/cmd-by-issuetime` | exact issue-time filtering over authorized Commands | official ATS + boundaries/paging |
| 57 | `/req/advanced-filtering/cmd-by-exectime` | exact execution-time filtering | official ATS + interval cases |
| 58 | `/req/advanced-filtering/cmd-by-status` | filter using public current-status projection | official ATS + transition cases |
| 59 | `/req/advanced-filtering/cmd-by-sender` | filter by authorized sender identifier without leakage | official ATS + concealment test |
| 60 | `/req/advanced-filtering/cmd-by-foi` | filter by target FOI relationship | official ATS + relationship fixtures |
| 61 | `/req/advanced-filtering/status-by-statuscode` | filter status history by exact nine-code vocabulary | official + corrected resource-type test |
| 71 | `/req/create-replace-delete/command` | support POST/PUT/DELETE at specified nested/canonical paths under claimed CRD class | official ATS + revision/lifecycle tests |
| 72 | `/req/create-replace-delete/command-schema` | reject parent-schema-invalid create/replace with `400` and no effect | official ATS + all encodings |
| 73 | `/req/create-replace-delete/command-status` | support authorized status POST/PUT/DELETE; retain immutable internal revisions | official ATS + actor/terminal tests |
| 74 | `/req/create-replace-delete/command-result` | support authorized result POST/PUT/DELETE with parent ownership/schema | official ATS + result alternatives |
| 75 | `/req/create-replace-delete/feasibility` | support POST and item-specific PUT/DELETE; repair missing nested `{id}` in profile/OAD | official + supplemental path test |
| 76 | `/req/create-replace-delete/feasibility-status` | support controlled status mutations and reject invalid feasibility codes | official ATS + transition test |
| 77 | `/req/create-replace-delete/feasibility-result` | support controlled result mutations against feasibility result schema | official ATS + schema/ownership test |
| 85 | `/req/update/command` | PATCH current Command revision only with accepted media/precondition profile | official draft-dependency ATS + CAS race |
| 86 | `/req/update/command-schema` | reject schema-invalid Command update with `400` and no partial effect | official + all-encoding negative tests |
| 87 | `/req/update/command-status` | PATCH only authorized mutable projection; preserve prior revision | official + terminal/history tests |
| 88 | `/req/update/command-result` | PATCH result under exact parent contract and precondition | official + result-schema tests |
| 89 | `/req/update/feasibility` | PATCH Feasibility request/revision; rerun affected analysis without `UPDATED` code | official + supplemental lifecycle test |
| 90 | `/req/update/feasibility-status` | PATCH only authorized status representation; preserve evidence history | official + actor/history tests |
| 91 | `/req/update/feasibility-result` | PATCH result against `feasibilityResultSchema` with precondition | official + schema/revision tests |
| 99 | `/req/json/controlstream-schema` | serialize/validate ControlStream including required `async` | official JSON Schema test |
| 100 | `/req/json/commandschema-schema` | serialize command schema resource using published schema | official JSON Schema test |
| 101 | `/req/json/command-schema` | serialize Command and Feasibility Command-shaped resource using published schema | official JSON Schema + feasibility fixture |
| 102 | `/req/json/command-constraints` | UTC-check times and validate parameters against exact parent schema | official + supplemental UTC test |
| 103 | `/req/json/commandstatus-schema` | serialize every status item/collection using published schema | official JSON Schema + lifecycle corpus |
| 104 | `/req/json/commandresult-schema` | serialize every result item/collection using published schema | official JSON Schema + all alternatives |
| 105 | `/req/json/commandresult-constraints` | validate inline ordinary data against `resultSchema` and feasibility data against `feasibilityResultSchema` | official + supplemental feasibility-branch test |

## Appendix D. Report Completion Checklist

- [x] Topic ID, plan, scope, and prerequisites aligned
- [x] Applicable requirements, conformance classes, ATS, and defects traced
- [x] Relevant upstream issues date-checked without overriding the published baseline
- [x] Synchronous/asynchronous Command and Feasibility lifecycles modeled
- [x] Required state/response and responsibility matrices complete
- [x] HTTP status, Location, Content-Location, body, navigation, polling, and errors resolved
- [x] Feasibility result, evidence, provenance, freshness, cache, invalidation, and non-reservation semantics defined
- [x] Rust domain/component and architecture options evaluated
- [x] Persistence, transaction, claim, lease, fence, idempotency, retry, race, and restart rules recommended
- [x] Gateway, DDIL, security, information-exposure, provenance, and audit handoffs bounded
- [x] Official/supplemental conformance and executable scenarios traced
- [x] Phased implementation, risks, alternatives, confidence, and downstream owners stated
- [x] Sources are primary/pinned where available and limitations are explicit
- [x] Report edited and ready for plan-owner review
- [x] Report accepted by plan owner

**Research execution completed:** September 15, 2026<br>
**Current disposition:** Final and accepted by the Glaux Project Lead<br>
**Next authorized topic:** IDR-SRV-038 — Command Authorization, Safety, and Audit Strategy. No later topic is authorized.
